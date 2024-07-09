# 1. 연관관계 매핑

### Entity 매핑

- Entity - JPA를 이용해서 데이터베이스 테이블과 매핑할 클래스
- Entity 매핑 - Entity 클래스에 데이터베이스 테이블과 컬럼, PK, FK 등을 설정하는 것

### 연관관계 매핑

- Entity들은 대부분의 경우 다른 Entity들과 연관관계를 가짐
- 데이터베이스 테이블은 FK로 JOIN을 이용해서 관계 테이블을 참조
- Entity는 객체 참조를 이용해서 연관 Entity를 참조
- 즉, 연관관계 매핑 - 데이터베이스 테이블의 FK를 객체의 참조와 매핑하는 것

### 단방향 vs 양방향

- 사실상 단방향 매핑만으로 연관관계 매핑은 이미 완료
- 단방향 매핑에 비해 양방향 매핑은 복잡하고 객체에서 양쪽 방향을 모두 관리해주어야 함
- 양방향 매핑은 단방향 매핑에 비해 반대 방향으로의 객체 그래프 탐색 기능이 추가된 것 뿐
- 대개의 경우 단방향 매핑이면 충분하다.
    - 우선은 단방향 매핑을 사용하고, 반대 방향으로의 개체 그래프 탐색이 필요할 때 양방향을 사용한다.

## 💡 이렇게 이미 잘 알려진 내용들이 모든 경우에 통하는 것이 맞을까?

### 영속성 전이 (persistence cascade)

- Entity의 영속성 상태 변화를 연관된 Entity에도 함께 적용하는 것
- 연관관계의 다중성(Multiplicity) 지정 시 cascade 속성으로 설정

| Cascade Type | 설명 |
| --- | --- |
| PERSIST | Entity를 영속 객체로 추가할 때 연관된 Entity도 함께 영속 객체로 추가한다. |
| REMOVE | Entity를 삭제할 때 연관된 Entity도 함께 삭제한다. |
| DETACH | Entity를 영속성 컨텍스트에서 분리할 때 연관된 Entity도 함께 분리 상태로 만든다. |
| REFRESH | Entity를 데이터베이스에서 다시 읽어올 때 연관된 Entity도 함께 다시 읽어온다. |
| MERGE | Entity를 준영속 상태에서 다시 영속 상태로 변경할 때 연관된 Entity도 함께 변경한다. |
| ALL | 모든 상태 변화에 대해 연관된 Entity에 함께 적용한다. |

### 예시) 다대일(N:1) 단방향 연관관계 매핑에서 영속성 전이(cascade)를 통한 insert 수행

![제목 없는 다이어그램 drawio](https://github.com/2eungwoo/TIL/assets/89715722/5ee602bf-e5c8-468b-b5bf-168252072581)


***Member.class***

```java
@Entity
public class Member {
    @Id
    @Column(name = "member_id")
    private Long memberId;

    private String name;

    @Column(name = "created_at")
    private LocalDateTime createdAt;
}
```

***MemberDetail.class***

```java
@Entity
public class MemberDetail {
    @Id
    @Column(name = "member_detail_id")
    private Long memberDetailId;

    @ManyToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "member_id")
    private Member member;

    private String type;

    private String description;
}
```

***createMemberWithDetailsService.class***

```java
@Transactional
public void createMemberWithDetailsService(){
	Member member = new Member("member1", LocalDateTime.now());
	
	
	MemberDetail memberDetail1 = new MemberDetail();
	memberDetail.setMember(member);
	memberDetail.setType("typeA")
	memberDetail.setDescription("member1-typeA");

	
	MemberDetail memberDetail2 = new MemberDetail();
	memberDetail.setMember(member);
	memberDetail.setType("typeB")
	memberDetail.setDescription("member1-typeB");
	
	
	memberDetailRepository.saveAll(Arrays.asList(memberDetail1,memberDetail2));
	
} 
```

쿼리 결과

```java
insert into members (created_at, name, member_id) values ('2024-07-09T15:20:00.1223', 'member1', 1)
insert into member_details (description, member_id, type) values ('member1-typeA', 1, 'typeA')
insert into member_details (description, member_id, type) values ('member1-typeA', 1, 'typeA')
```

> MemberDetail 엔티티를 저장할 때 Member 엔티티도 저장이 된다.
> 

### 예시) 일대다(1:N) 단방향 연관관계 매핑에서 영속성 전이(cascade)를 통한 insert 수행

```java
08:27
```

---

## N + 1 문제

### N + 1 문제

- 연관된 Entity를 가지고 올 때 의도한 것과 달리 추가적인 쿼리가 발생하는 문제
- Entity에 대해 하나의 쿼리로 N개의 레코드를 가져왔을 때, 연관관계 Entity를 가져오기 위해 쿼리를 N번 추가적으로 수행하는 문제

### 해결방법(JPA)

- Fetch Join
- Entity Graph

### Fetch 전략

- FetchType.EAGER
- FetchType.LAZY

### 기본 Fetch 전략

- ToOne (@OneToOne, @ManyToOne) : FetchType.EAGER)
- ToMany (@OneToMany, @ManyToMany) : FetchType.LAZY)

### N + 1 문제 흔한 오해

- *N+1 문제는 EAGER Fetch 전략 때문에 발생? NO*
    
    ⇒ Fetch 전략을 LAZY로 설정했더라도 연관 Entity를 참조하면 그 순간 추가적인 쿼리가 수행됨
    
    ⇒ Fetch 전략을 뭐로 쓰느냐는, 연관 엔티티를 즉시 로딩할거냐 참조가 일어날때 로딩할거냐에 대한 시점의 차이일 뿐
    
- *findAll() 메서드는 N+1 문제를 발생시키지 않는다? NO*
    
    ⇒ Fetch 전략을 적용해서 연관 Entity를 가져오는 것은 오직 단일 Entity에 대해서만 적용
    
    ⇒ 단일 레코드 조회가 아닌 경우(JPQL 혹은 findAll() 메서드), 해당 메서드를 먼저 수행하고, 그 결과 반환되는 각각의 Entity에 대해서 각각 설정된 Fetch 전략이 적용되어 로딩하는 것
    
    ⇒ 따라서 findAll() 메서드 호출 역시 이 과정에서 N+1 문제가 발생
    

### Fetch JOIN으로 N+1 문제 해결 시 흔히 하는 실수

- **Pagination 쿼리에 Fetch JOIN을 적용하는 경우**
    
    ⇒ Pagination 쿼리에 Fetch JOIN을 적용하면 실제로는 모든 레코드를 가져오는 쿼리가 실행된다(Hibernate)
    
    ⇒ 그 다음 메모리에서 원하는 페이지 만큼만 반환을 하게 되는 것
    
    ⇒ 결과만 봤을 때는 페이지네이션이 잘 적용되는 것 같아 보이지만 데이터베이스 쪽에서는 결국 모든 레코드를 다 가져오는 쿼리를 수행한 것
    
    ⇒ Pagination과 Fetch JOIN 쿼리를 분리하여 사용해야 한다.
    
- **둘 이상의 컬렉션을 Fetch JOIN - MultipleBagFetchException**
    
    ⇒ Java의 java.util.List 타입은 기본적으로 Hibernate의 Bag 타입으로 매핑됨
    
    ⇒ Bag은 Hibernate에서 중복 요소를 허용하는 비순차(unordered) 컬렉션
    
    ⇒ 둘 이상의 컬렉션(Bag)을 Fetch Join하는 경우, 그 결과로 만들어지는 카테시안 곱에서 어느 행이 유효한 중복을 포함하고 있고 어느 행이 그렇지 않은 지 판단할 수 없어 MultipleBagFetchException이 발생
    
    ⇒ List를 Set으로 변경하여 중복을 허용하지 않거나, @OrderColumn을 부여해서 순서를 정해줘야 한다.
    

# 2. Spring Data JPA Repository

reference : [유튜브](https://www.youtube.com/watch?v=rYj8PLIE6-k)
