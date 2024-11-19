### 값 타입 컬렉션

값 타입을 컬렉션에 담아서 사용할 때, 해당 컬렉션을 값 타입 컬렉션이라 한다.

@OneToMany 처럼 엔티티를 컬렉션으로 사용하는 것이 아닌, Integer, String, Embedded와 같은 값 타입을 컬렉션으로 사용하는 것이다.

RDB는 컬렉션을 담을 수 있는 구조가 없기 때문에 별도의 테이블을 만들어서 저장해야 한다.

이 때 값 타입 컬렉션은 개념적으로는 일대다 관계이다.

또한 값 타입을 저장하는 테이블은 값 타입을 소유한 엔티티의 기본 키와 모든 값 타입 필드를 묶어서 PK로 사용하며, 엔티티의 기본 키를 PK겸 FK로 사용한다.

### @ElementCollection

값 타입 컬렉션을 매핑할때 사용하는 어노테이션.

컬렉션인 해당 필드가 컬렉션 객체임을 JPA에게 알려주는 역할이다. 

@Entity가 아닌 Basic Type 또는 Embeddable 클래스로 정의된 컬렉션을 테이블로 생성하여 일대다 관계로 다룬다.

### @CollectionTable

@ElementCollection과 함께 사용한다.

@CollectionTable은 값 타입 컬렉션을 매핑할 테이블에 대한 정보를 지정하는 역할이다.

이를 생략하면 기본 값을 이용하여 매핑한다.

- JPA는 **기본값으로 엔티티 이름과 필드 이름을 조합**하여 컬렉션을 매핑할 테이블 이름을 설정.
- 기본적으로 테이블 이름은 `<엔티티명>_<필드명>` 형식, 외래 키는 `<엔티티명>_id` 형식

```tsx
@Entity
public class Member {
    @Id
    @GeneratedValue
    private Long id;

    @ElementCollection
    private List<Nickname> nickname; 
}

```

> 위와 같이 `@CollectionTable`을 생략하면, JPA는 기본값을 바탕으로 테이블과 외래 키를 생성하여 매핑을 처리한다.
> 
> - 컬렉션을 저장할 테이블 이름: `Member_nickname`
> - 테이블의 외래 키 이름: `member_id`

### 참고 및 주의사항

- 값 타입 컬렉션은 조회 시 Lazy loading 전략을 사용하며, 타입의 생명 주기는 부모 엔티티에 의해 관리된다.

즉 Cascade ALL + orphanRemoval true 를 필수로 갖게 되는 것

- 값 타입은 엔티티와 다르게 식별자 개념이 없기 때문에 값을 변경하면 추적이 어렵다.

예를 들어 값 타입 컬렉션에 삭제가 발생하면 소유하는 엔티티와 연관된 모든 데이터를 삭제하고 현재 남아있는 값을 모두 다시 저장한다.

값 타입 컬렉션을 매핑하는 테이블은 모두 컬럼을 묶어서 기본 키를 구성해야 한다.(null X,중복 X)

### @ElementCollection, @OneToMany 차이

- @ElementCollection은 연관된 부모 엔티티 하나에만 연관되어 관리하기 때문에 부모 엔티티와 독립적 사용이 안된다. 항상 부모와 함께 저장, 삭제가 되므로 cascade 옵션은 제공되지 않는다. (그냥 ALL임)
- 식별자 개념이 없으므로 컬렉션 값 변경 시 전체 삭제 후 새로 추가된다.

- @OneToMany는 다른 엔티티에 의해 관리될 수 있는 엔티티이므로 id로 연관을 맺는다.

### 결론

값 타입은 정말 값 타입이라 판단되고, 정말 단순할 때 사용하는 것이 좋다.

식별자가 필요하고, 지속해서 값을 추적, 변경해야 한다면 값 타입이 아닌 엔티티로 사용해야 한다.

*reference*

https://developer-hm.tistory.com/48

https://prohannah.tistory.com/133
