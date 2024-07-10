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
08:27 이해가 안가요
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

- ***N+1 문제는 EAGER Fetch 전략 때문에 발생? NO***
    
    ⇒ Fetch 전략을 LAZY로 설정했더라도 연관 Entity를 참조하면 그 순간 추가적인 쿼리가 수행됨
    
    ⇒ Fetch 전략을 뭐로 쓰느냐는, 연관 엔티티를 즉시 로딩할거냐 참조가 일어날때 로딩할거냐에 대한 시점의 차이일 뿐
    
- ***findAll() 메서드는 N+1 문제를 발생시키지 않는다? NO***
    
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

### Repository

- 도메인 객체에 접근하기 이ㅜ해서 컬렉션과 유사한 인터페이스를 사용해 도메인과 데이터 매핑 게층 사이를 중재(mediate)
- JPA의 개념이 아니라, Spring Framework에서 제공해주는 것

### Spring Data Repository

- Spring Data Project에서 제공
- data access layer 구현을 위해 반복적으로 작성했던 유사한 코드(boilerplate code)를 줄이기 위한 추상화 제공

### Spring Data JPA Repository

- JpaRepository 인터페이스를 상속받아서 Repository를 선언하는 것만으로 웬만한 CRUD, Paging, Sorting 메서드 사용 가능
- 메서드 이름 규칙을 통한 쿼리 생성

예시)

```java
public interface MemberRepository extends JpaRepository<Member, Long>{
	// select * from Member where name = {Name}
	List<Member> findByName(String Name);

	// select * from Member where name = {Name} and created_at > {createdAt}
	List<Member> findByNameAndCreateDateAfter(String Name, LocalDateTime createdAt);
}
```

## JPA Repsitory 와 JOIN 쿼리

### 오해 - JPA Repository 메서드로는 JOIN 쿼리를 실행할 수 없다?

- JPA는 데이터베이스 테이블 간의 관계를 Entity 클래스 속성으로 모델링
- JPA Repsitory 메서드에서는 underscore를 통해 내부 속성값에 접근할 수 있다

***Person.class***

```java
@Entity
public class Person {
	@Id
	private Long id;
	
	@Embedded
	private Address address;
	
	@Embeddable
	public static class Address {
		String zipcode, city, street;
	}
}
```

***PersonRepository.class***

```java
public interface PersonRepository extends JpaRepository<Person, Long> {
	List<Person> findByAddress_Zipcode(String zipcode);
	// address 안의 zipcode 속성에 접근 가능
}
```

### 사실 - JPA Repository 메서드로 JOIN 쿼리 실행 가능!

```java
@Entity
public class Member {
  @Id
  @Column(name = "member_id")
  private Long memberId;

	@OneToMany
	private List<MemberDetails> details;
}

@Entity
public class MemberDetails {
	@EmbeddedId
	private Pk pk;
	
	@Embeddable
	public static class Pk {
		private String type;
	}
}

public interface MemberRepository extends JpaRepository<Member, Long> {
	// select * from Member m
	// inner join MemberDetail md
	// on m.member_id = md.member_id
	// where md.type = {type}
	List<Member> findByDetails_Pk_Type(String type);
}
```

## 잘 알려지지 않은 사실 - Page vs Slice

### Page interface vs Slice interface

- page 인터페이스가 slice 인터페이스를 상속받고 있다.

***Page.Interface***

```java
public interface Page<T> exnteds Slice<T> {
	int getTotalPages();
	long getTotalElements();
}
```

***Slice.Interface***

```java
public interface Slice<T> extends Streamable<T> { 
	int getNumber();
	int getSize();
	int getNumberOfElements();
	List<T> getContent();
	...
}
```

page 인터페이스에는 추가적으로 getTotalPage와 getTotalElements 메서드가 있다.

따라서 쿼리는 주석과 같은 차이가 난다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
	// select * from Member where name = {name} offset {offset} limit {limit}
	// select count(*) from Member where name = {name}
	Page<Member> getAllByName(String name, Pageable pageable);
	
	// select * from Member where name = {name} offest {offset} limit {limit_plus_1}
	Slice<Member> readAllByName(String name, Pageable pageable);
}
```

> Page : 집계 쿼리가 추가로 나감
> 

count 쿼리가 안나가면 다음 페이지가 있는지 어떻게 알지?

slice 타입으로 선언된 메서드 경우에 limit가 우리가 전달한 limit값보다 레코드를 하나 더 가져온다. → next의 여부를 판단한다.

단, Page 타입으로 선언된 메서드를 사용하는 경우에도 우리가 pageable을 통해 제공한 limit 개수보다 적은 레코드가 반환되게 되면 count쿼리가 안나간다. 즉, 실제 페이징을 더 할 여지가 있는 경우에만 count 쿼리가 나간다.

## JPA Repository 메서드와 DTO

### 오해 - JPA Repository 메서드로는 DTO Projection을 할 수 없다?

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
	List<Member> findByName(String name);
	List<Member> findAllByName(String name);
	List<Member> findMemberIdByName(String name);
	List<Member> findADQDASDADByName(String name);
	
}
```

> 이렇게 projection을 위해서 메서드를 써줘도 결과는 항상 where조건으로 걸러서 엔티티가 반환된다
> 

```sql
select * from Member where name = {name}
```

그러면 원하는 반환 타입을 만들어서 원하는 컬럼만 추출할 수 없나?

### 사실 - JPA Repository 메서드로 DTO Projection 가능!

- Class 기반 (DTO) Projection
- Interface 기반 Projection
- Dynamic Projection

***Class 기반 (DTO) Projection***

```sql
@Value // lomok
public class MemberDto {
	private final String name;
	private final LocalDateTime createdAt;
}

public interface MemberRepository extends JpaRepository<Member, Long> {
	List<MemberDto> findByName(String name);
}

// select name, created_at 
// from Member 
// where name = {name}
```

***Interface 기반 Projection***

```sql
@Entity
public class Member {
	private String name;
}

// projection interface
public interface MemberNameOnlyDto {
	String getName(); // projection 원하는 필드명으로 메서드 선언
}

public interface MemberRepository extends JpaRepository<Member, Long> {
	List<MemberNameOnlyDto> findByCreatedAtAfter(LocalDateTime createdAt);
}

// select name 
// from Member 
// where created_at > {createdAt}
```

> interface로 선언만 해주면 Spring Data JPA가 구현 객체는 프록시로 만들어서 실행해준다.
> 

```sql
@Entity
public class Member {
	private String name;
	
	@OneToMany
	private List<MemberDetail> details;
}

@Entity
public class MemberDetail {
	@EmbeddedId
	private Pk pk;
	
	private String description;
	
	@Embeddable public static class Pk {
		private String type;
	}
}

public interface MemberDto {
 String getName();
 List<MemberDetailDto> getDetails();
 
interface MemberDetailDto { 
	 @Value("#{target.pk.type}")
	 String getTye();
	 String getDescription();
 }
}
```

> interface 중첩 구조 지원, @Value + SpEL (target 변수)
> 

(SpEL = spring expression language)

***Dynamic Projection***

```sql
public interface MemberRepository extends JpaRepository<Member, Long> {
	<T> Collection<T> findByCreatedAtAfter(LocalDateTime createdAt, Class<T> type);
}
// 인자에 DTO 클래스를 넣어준다
```

```sql
Collection<MemberDto> memberDto = memberRepository.findByCreatedAtAfter(LocalDateTime.now(), MemberDto.class);

Collection<MemberNameOnlyDto> memberNameDto = memberRepository.findByCreatedAtAfter(LocalDateTime.now(), MemberNameOnlyDto.class);
```

## Summary

### 연관관계 매핑

- 사실상 단방향 매핑만으로 연관관계 매핑은 이미 완료.
- 대개의 경우 단방향 매핑이면 충분하다.
- 일대다 단방향 연관관계 매핑에서 영속성 전이를 사용할 경우 추가적인 update 쿼리가 발생하므로 이 경우 양방향 매핑이 적합할 수 있다.

### Spring Data JPA Repository

- JpaRepository 상속 - 웬만한 CRUD, Paging, Sorting 메서드 사용 가능
- 메서드 이름 규칙에 따라 interface에 메서드 선언만 하면 쿼리 사용 가능
- JPA Repository 메서드로 JOIN 쿼리 사용가능 - 이름 규칙에 따라 Entity 내 연관관계 필드를 underscore로 탐색
- JPA Repository 메서드에서도 다양한 DTO Projection 지원

reference : [유튜브](https://www.youtube.com/watch?v=rYj8PLIE6-k)
