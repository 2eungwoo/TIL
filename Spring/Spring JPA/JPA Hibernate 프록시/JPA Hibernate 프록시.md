## 개요

JPA를 사용할 때 장점 중 하나는 객체 그래프를 통해 연관관게를 탐색할 수 있다는 것이다.

하지만 엔티티들은 데이터베이스에 저장되어 있기 때문에 한 객체 조회 시 연관되어 있는 엔티티를 모두 조회하는 것 보다는 필요한 연관관계만 조회하는 것이 효과적이다. 이러한 상황을 위해 JPA는 `지연 로딩` 이라는 것을 지원하는데, 우리가 일반적으로 가장 많이 사용하는 JPA 구현체인 `하이버네이트(Hibernate)` 는 `프록시 객체`를 통해 지연 로딩을 구현하고 있다.

`JPA 프록시`는 지연로딩이 가능하게 해주지만 잘 알지 못하고 사용하면 예상치 못한 예외들을 마주칠 수 있다. 이러한 예외들에 당황하지 않고 능숙하게 대응하기 위해 `JPA 프록시`에 대해 알아보자

## JPA 프록시(Proxy)

개요에서 말한대로 Hibernate는 지연 로딩을 구현하기 위해 프록시를 사용한다. 지연 로딩을 하려면 연관된 엔티티의 실제 데이터가 필요할 때까지 조회를 미뤄야 한다. 그렇다고 해당 엔티티를 연관관게로 가지고 있는 엔티티 필드에 null 값을 넣어두는 것은 바람직하지 않다. Hibernate는 지연 로딩을 사용하는 연관관계 자리에 프록시 객체를 주입하여 실제 객체가 들어있는 것처럼 동작하도록 한다.

덕분에 사용자는 연관관계 자리에 프록시 객체가 들어있든 실제 객체가 들어있든 신경쓰지 않고 작업할 수 있다.

## 프록시는 실제 객체의 상속본

프록시는 어떻게 실제 객체처럼 동작을 하는가?

이는 프록시가 실제 객체를 상속한 타입을 가지고 있기 때문이다. 그리고 프록시 객체는 실제 객체에 대한 참조를 가지고 있다가, 프록시 객체의 메서드를 호출했을 때 실제 객체의 메서드를 호출한다.

그래서 실제 객체 타입 자리에 들어가도 문제 없이 사용할 수 있는 것이다. 프록시 객체가 실제 객체의 상속본인 것은 JPA 엔티티 생성의 중요한 규칙과 연관이 있다. 이것은  `default 생성자는 접근 제한을 최소 protected로는 가져야 한다` 와`엔티티 클래스는 final로 정의할 수 없다` 이다.

만약 default 생성자가 private이면 프록시 생성 시 `super` 를 호출할 수 없을 것이고, 엔티티를 final로 선언한다면 상속이 불가능하게 되기 때문이다.

## 프록시의 초기화

그런데 최초 지연 로딩 시점에는 당연하게도 참조 값이 없는 상태이다. 그렇기 때문에 실제 객체의 메서드를 호출할 필요가 있을 때 데이터베이스를 조회해서 참조 값을 채우게 된다.

이를 ‘프록시 객체를 초기화한다’ 라고 한다.

실제 객체의 메서드를 호출할 필요가 있을 때 select 쿼리를 실행해서 실제 객체를 데이터베이스에서 조회하여 가져오고 참조 값을 저장하게 된다.

### 예시)

```java
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column
    private String name;

    @JoinColumn(name = "team_id")
    @ManyToOne(fetch = FetchType.LAZY)
    private Team team;

    public Member(String name, Team team) {
        this.name = name;
        this.team = team;
    }
    ...
}

@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
public class Team {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column
    private String name;

    public Team(String name) {
        this.name = name;
    }
    ...
}
```

Member의 연관관계 Team은 지연 로딩으로 설정되어 있기 때문에 Member를 조회하면 team 필드 자리에는 프록시가 들어가 있게 된다. 이 때 프록시 Team의 getName()을 호출하게 되면 select 쿼리가 실행되고 프록시가 초기화된다.

```java
Team team = member.getTeam();
System.out.println(team.getName()); // 이 시점에 프록시 초기화!
```

<img width="373" alt="1" src="https://github.com/user-attachments/assets/c2b276cc-1e77-4daa-b80a-421d7c6198fe">

단, 이 때 프록시가 실제 객체를 참조하는 것이지 프록시가 실제 객체로 바뀌는 것은 아니라는 점을 주의하자.

참고로 프록시를 초기화하지 못하는 경우도 존재한다. 프록시의 초기화는 영속성 컨텍스트의 도움을 받기 때문에 영속성 컨텍스트의 관리를 못하는 상황이라면 그렇다. 이 경우에는 `LazyInitializationException` 을 만나게 된다. (준영속 상태의 프록시를 초기화 한다거나 OSIV 옵션이 꺼져있는 경우, 트랜잭션 바깥에서 프록시를 초기화 하려는 경우)

즉, 프록시는 반드시 `영속 상태` 이어야만 초기화가 된다.

# **id를 조회할 때는 프록시가 초기화되지 않는다**

## id를 조회할 경우 프록시가 초기화 되지 않는다.

getName()이 아닌 getId()를 호출해보자

```java
Team team = member.getTeam();
System.out.println(team.getId());
```

<img width="210" alt="2" src="https://github.com/user-attachments/assets/bc2f1a5d-696d-48ff-922c-17c87cfe019b">

print에 의해 id가 출력되지만 select 쿼리가 발생하지 않았다. 여기서 식별자를 조회할 떄에는 프록시를 초기화하지 않는다는 사실을 유추할 수 있다.

프록시 초기화 과정은 프록시 객체 내부의 `ByteBuddyInterceptor` 라는 클래스가, 정확히는 상위의 추상 클래스인 `AbstractLazyInitializer` 가 담당하게 된다.  해당 클래스의 초기화 로직을 확인해보자

```java
@Override
public final Serializable getIdentifier() {
    if (isUninitialized() && isInitializeProxyWhenAccessingIdentifier() ) {
        initialize();
    }
    return id;
}
```

엔티티의 식별자를 조회하는 메서드(getId()와 같은)를 호출하게 되면 최종적으로 `AbstractLazyInitializer` 의 `getIdentifier()` 를 호출하게 되는데, if 문 바깥쪽에서 그대로 id를 리턴하는 것을 볼 수 있다. 이는 `AbstractLazyInitializer` 가 id 값을 가지고 있기 때문이다. if 문 안쪽은 초기화되지 않았을 것과, 식별자 접근 시 프록시를 초기화하는 옵션을 켰을 때 true가 되는 조건문인데, hibernate.jpa.compliance.proxy 라는 설정을 true로 해줘야만 해당 옵션이 켜진다.

이 설정의 기본값은 false이므로 일반적으로 id를 getter로 호출하게 되면 프록시가 초기화되지 않는 것이다.

그런데 이 때 알아야할 점이 있다. 예제에서 호출한 메서드는 getId()이며, 자바 빈 규약에 맞는 get + 필드명 으로 이루어져있기 때문에 getIdentifier()가 호출된 것이다. 만약 findId()와 같이 getter에 대한 자바 빈 규약을 만족시키지 못하거나, getTeamId()와 같이 식별자 이름과 매칭되지 않을 경우에는 getIdentifier() 메서드를 호출하지 못하고 프록시가 초기화된다.

또한, id값은 프록시가 아니라 프록시 내부의 인터셉터에 들어있고, 프록시 객체가 가진 필드 값들은 모두 null이다. 일반적으로 필드로 꺼내 쓰기 때문에 상관 없겠지만, 만약 필드가 public이어서 바로 접근한다면 null에 접근하게 되는 것이다.

<img width="697" alt="3" src="https://github.com/user-attachments/assets/072faade-9c29-40dd-8017-0906cbeb51fa">

# **프록시의 equals를 재정의 할 때는 instanceof와 getId를 사용할 것**

## 프록시의 equals를 재정의 할 때는 instance of 와 getId 를 사용할 것

대부분 흔히 equals / hashcode 메서드를 만들 때 인텔리제이의 기능을 사용하곤 한다. 그리고 보통 JPA 엔티티의 equals는 id값만 비교해서 재정의한다. 그러면 다음과 같은 메서드가 만들어질 것이다.

```java
@Override
public boolean equals(Object o) {
    if (this == o) {
        return true;
    }
    if (o == null) {
	    return false;
	  }
    if (this.getClass() != o.getClass()) {
        return false;
    }
        
    Team team = (Team) o;
    return Objects.equals(id, team.id);
}
```

이렇게 equals를 재정의하게 된다면, 프록시 객체의 equals를 호출했을 때 문제가 생긴다.

```java
Team team = member.getTeam();
Team sameTeam = member.getTeam();
assertThat(team).isEqualTo(sameTeam);
```

테스트코드로 확인해보면 다음과 같은 결과를 받는다.
<img width="699" alt="4" src="https://github.com/user-attachments/assets/ca46cb33-8954-4683-900a-148250902cbe">


주소 값도 같은 객체인데 equals 메서드를 통해 동등성 비교를 하는 isEqualTo를 통과하지 못한다. 주소값이 같기 때문에 `==` 로 비교하면 같은 객체라고 하는데, 동등성이 만족되지 못하고 있다. 왜지?

바로 `getClass` 에 답이 있다. sameTeam을 인자로 하여 team의 equals를 호출한다. 그러면 getId가 아닌 메서드를 호출했으니 프록시 team이 초기화되고 프록시 team 내부의 실제 객체의 equals를 호출한다.

그렇다면 getClass는 어떻게 될까? equals 안에서 호출되는 getClass의 결과는 Team.Class이고, 인자로 전달받은 sameTeam의 getClass는 프록시 타입이 나오게 되어 equals의 결과가 False가 되는 것이다.

따라서 일반적으로 equals를 비교하던 것처럼 getClass를 비교하는 것으로 equals를 작성해서는 안되며(애초에 리스코프 치환 원칙을 위반하게 됨) `o instance of Team` 과 같은 형태로 코드를 작성해야 한다. 프록시는 원본 객체의 상속본이기 때문에 instance of 를 통과할 수 있다. 또 한가지, 일반적인 객체의 상속관계라면 문제가 없겠지만, 프록시의 필드값은 모두 null이기 때문에 team.id는 사실 null이다. 따라서 `Objects.equals(id, team.id)` 는 id값과 null을 비교하게 되므로 False가 반환된다.

하지만 getId()는 실제 id를 반환하므로 다음과 같이 equals 코드를 수정할 수 있다.

```java
@Override
public boolean equals(Object o) {
    if (this == o) {
        return true;
    }
    if (!(o instanceof Team)) {
        return false;
    }
    Team team = (Team) o;
    return Objects.equals(id, team.getId());
}
```

프록시의 equals 부분은 간과하기 쉬우며 많은 버그를 발생시키기 때문에 유의해야 한다.

## 프록시가 생성되면 영속성 컨텍스트는 프록시를 반환한다.

프록시로 만들어진 엔티티에 대한 조회가 발생하면, 영속성 컨텍스트는 실제 엔티티와 프록시 객체 중 프록시 객체를 반환하게 되어있다.

영속성 컨텍스트의 특징 중 하나인 `동일성 보장` 에 의해 한 번 프록시로 만들어진 객체는 프록시 초기화를 하더라도 실제 엔티티로 변환되지 않고 참조값만 가지게 된다. 따라서 동일성을 보장해주기 위해 한 트랜잭션 내에서 최초 생성이 프록시로 된 엔티티는 이후 초기화 여부에 상관없이 영속성 컨텍스트가 무조건 같은 프록시 객체를 반환한다.

반대로 영속성 컨텍스트에 최초로 저장될 때 실제 엔티티로 저장된 경우, 이후 프록시가 아닌 실제 엔티티가 반환된다. 이는 지연 로딩으로 인해 프록시가 들어갈 자리에도 마찬가지이며, 이미 영속성 컨텍스트에 엔티티가 저장된 상태에서 해당 엔티티를 FetchType.LAZY로 가지고 있는 엔티티를 조회하는 경우라면, 프록시 객체를 반환하는 대신 실제 엔티티를 반환해주게 된다.

*reference* : 

- 자바 ORM 표준 JPA 프로그래밍 (김영한 저)
- https://tecoble.techcourse.co.kr/post/2022-10-17-jpa-hibernate-proxy/
