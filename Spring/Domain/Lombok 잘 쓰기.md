# 목차

1. [@Data 사용은 지양하자](#data-사용은-지양하자)
   - [무분별한 Setter 남용](#무분별한-setter-남용)
   - [ToString으로 인한 양방향 연관관계시 순환 참조 문제](#tostring으로-인한-양방향-연관관계시-순환-참조-문제)
2. [바람직한 Lombok 사용법](#바람직한-lombok-사용법)
   - [@NoArgsConstructor 접근 권한을 최소화 하자](#noargsconstructor-접근-권한을-최소화-하자)
   - [Public으로 안하고 Protected로 하는 이유](#public으로-안하고-protected로-하는-이유)
   - [Builder 사용시 매개변수를 최소화 하자](#builder-사용시-매개변수를-최소화-하자)
  

## @Data 사용은 지양하자

```java
@Entity
@Table(name = "member")
@Data
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;

    @Column(name = "email", nullable = false)
    private String email;

    @Column(name = "name", nullable = false)
    private String name;

    @CreationTimestamp
    @Column(name = "create_at", nullable = false, updatable = false)
    private LocalDateTime createAt;

    @UpdateTimestamp
    @Column(name = "update_at", nullable = false)
    private LocalDateTime updateAt;
}
```

@Data는 @ToString, @EqualsAndHashCode, @Getter, @Setter, @RequiredArgsConstructor을 한번에 사용하는 강력한 어노테이션이다. 

@Data를 사용하기 보다는 용도에 맞게 필요한 어노테이션만 적절히 사용하도록 하자

### **무분별한 Setter 남용**

@Data를 사용하면 자동으로 @Setter도 적용된다. 그로 인해서 생기는 문제점들이 있다.

Setter를 사용하면 객체를 언제든지 변경할 수 있는 상태가 되어 객체의 안전성을 보장하기 힘들다. 위의 Member 엔티티에서 email의 변경 기능을 제공하고 싶지 않으면 email필드에 관련한 setter도 제공되지 말아야 한다. 

### **ToString으로 인한 양방향 연관관계시 순환 참조 문제**

```
@Entity
@Table(name = "member")
@Data
public class Member {
    ....
    @OneToMany
    @JoinColumn(name = "coupon_id")
    private List<Coupon> coupons = new ArrayList<>();
}

@Entity
@Table(name = "coupon")
@Data
public class Coupon {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;

    @ManyToOne
    private Member member;

    public Coupon(Member member) {
        this.member = member;
    }
}
```

위의 코드처럼 Member와 Coupon 객체가 서로 양방향 맵핑이 걸려있을 경우 ToString 호출에 의한 무한 순환 참조가 발생할 수 있다. JPA를 사용하다 보면 객체를 Json으로 직렬화 하는 과정에서 발생하는 문제와 동일한 이유이다. @Data를 아무 생각 없이 쓰면 이러한 문제를 만날 수밖에 없다.

쉬운 해결 방법으로는

```
@ToString(exclude = "coupons")
public class Member {...}
```

@ToString 어노테이션을 붙여서 ToString 항목에서 제외시키면 된다.

@Getter역시 모든 멤버필드에 대해 getter를 제공해주면 캡슐화와 관련하여 좋다고 말할 수 없을 것이다. 무분별하게 getter를 제공해주면 객체를 사용하는 곳에서 get메서드를 이용해 로직을 구현하는 경우가 있을 수 있다. 이런 기능들에 대해서는 해당 객체가 캡슐화하여 제공해주는 것이 바람직하지만 이렇게까지 설계하기에는 너무 복잡해지는 등 현실적으로 힘들다. 따라서 @Getter는 사용하되 최대한 객체가 캡슐화하여 자신의 책임을 다하는 기능을 제공하도록 하는 것이 바람직하다.

## **바람직한 Lombok 사용법**

나름대로 공부하며 생각해보는 바람직한 Lombok 사용법이다.

```java
@Entity
@Table(name = "member")
@ToString(exclude = "coupons")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@EqualsAndHashCode(of = {"id", "email"})
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;

    @Column(name = "email", nullable = false)
    private String email;

    @Column(name = "name", nullable = false)
    private String name;

    @CreationTimestamp
    @Column(name = "create_at", nullable = false, updatable = false)
    private LocalDateTime createAt;

    @UpdateTimestamp
    @Column(name = "update_at", nullable = false)
    private LocalDateTime updateAt;

    @OneToMany
    @JoinColumn(name = "coupon_id")
    private List<Coupon> coupons = new ArrayList<>();

    @Builder
    public Member(String email, String name) {
        this.email = email;
        this.name = name;
    }
}
```

@ToString(exclude = "coupons"), @Getter 메서드는 위에서 언급했으므로 넘어간다.

### **@NoArgsConstructor 접근 권한을 최소화 하자**

JPA에서는 프록시 생성을 위해 기본 생성자가 반드시 하나 필요하다. 정확히는 엔티티의 연관 관계에서 지연 로딩의 경우에는 실제 엔티티가 아닌 프록시 객체를 통해서 조회를 한다. 프록시 객체를 사용하기 위해서 JPA 구현체는 실제 엔티티의 기본 생성자를 통해 프록시 객체를 생성한다.

이때 접근 권한이 protected이면 된다. 굳이 외부에서 생성을 열어둘 필요가 없다.

```java
@NoArgsConstructor(access = AccessLevel.PROTECTED)
```

이 어노테이션의 뜻은 **“아무런 매개변수가 없는 생성자를 생성하되 다른 패키지에 소속된 클래스는 접근을 불허한다”**라는 뜻이다.

즉 아래와 같은 생성자 코드를 생성해 준다는 것이다.

```java
protected Entity() {}
```

### Public으로 안하고 Protected로 하는 이유

우리가 일반적으로 객체를 생성하고, 객체에 값을 채워넣는 방법은 아래의 3가지 방법이 있다.

1. 기본 생성자를 통해 객체 생성 후 setter로 필드 값을 주입
2. 매개변수를 가지는 생성자를 통해 객체 생성과 동시에 필드 값을 초기화
3. 정적 팩토리 메서드 또는 빌터 패턴을 통해 객체 생성과 동시에 필드 값 초기화

먼저 첫번째 기본 생성자로 객체 생성 후 setter로 필드 값을 주입하는 방법은 위에서 말했듯이 객체지향적으로 올바르지 못한 방법이라고 할 수 있기에 권장되지 않는다. (나중에 어디서 객체 값이 변경됐는지 추적도 너무 힘듬)

JPA에서 기본적으로 기본 생성자를 요구하기 때문에 @NoArgsConstructor를 작성하는 것인데 위와 같은 이유때문에 JPA의 Entity Class 요구사항 이외에는 기본 생성자를 이용할 일이 없게 된다. 

결국 2,3번의 방법으로 무분별한 객체 생성을 방지하게 되고 접근 권한을 public이 아닌 protected로 제한함으로써 최대한 접근 범위를 작게 가져가는 것이다.

내용을 정리하면 다음과 같다.

```java
@NoArgsConstructor(access = AccessLevel.PUBLIC)
    - 기본 생성자를 이용하여, 값을 주입하는 방식을 최대한 방지하기 위해서 사용을 권장하지 않는다.
    
    
@NoArgsConstructor(access = AccessLevel.PROTECTED)
    - 위, 아래와 같은 프록시 객체의 생성과 객체에 대한 접근 범위 문제를 해결하기 위해서 사용한다.
    
  
@NoArgsConstructor(access = AccessLevel.PRIVATE)
	  - 프록시 객체 생성시 문제가 생기기 때문에 사용을 권장하지 않는다.
```

## **Builder 사용시 매개변수를 최소화 하자**

```java
@Builder
public class Member {...}
```

클래스에 @Builder를 쓰면 @AllArgsConstructor 어노테이션을 붙인 효과를 발생시켜 모든 멤버 필드에 대해 매개변수를 받는 기본생성자를 만든다.

이 때문에 모든 멤버 필드에 대한 매개변수를 허용하게 된다.

Member의 id 생성 전략이 auto_increment를 의존하고 있다고 가정했을 때, id를 넘겨받지 않아야 하며 createdAt, updatedAt과 같은 경우 @CreationTimestamp, @UpdateTimestamp 각각의 어노테이션이 해당 일을 담당하고 있지만 클래스에 @Builder를 사용했기 때문에 객체 생성시 받지 않아야하는 값들이 값을 받을 수 있는 상태로 만들어진다.

```java
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;

    @Column(name = "email", nullable = false)
    private String email;

    @Column(name = "name", nullable = false)
    private String name;

    @CreationTimestamp
    @Column(name = "create_at", nullable = false, updatable = false)
    private LocalDateTime createAt;

    @UpdateTimestamp
    @Column(name = "update_at", nullable = false)
    private LocalDateTime updateAt;

    @Builder
    public Member(String email, String name) {
        this.email = email;
        this.name = name;
    }
}
```

`reference` : [cheese10yun github](https://github.com/cheese10yun/blog-sample/tree/master/lombok#실무에서-lombok-사용법)

이렇게 받아야 하는 생성자를 필요조건에 따라 지정하고 그 위에 @Builder를 붙이는게 바람직하다.

이렇게 받아야 하는 생성자를 필요조건에 따라 지정하고 그 위에 @Builder를 붙이는게 바람직합니다.

이제 Member.build(). 로 접근을 해도 매개변수 name, email만 넘겨받을 수 있으며 이 외에는 값을 받을 수 없는 상태가 된다.
