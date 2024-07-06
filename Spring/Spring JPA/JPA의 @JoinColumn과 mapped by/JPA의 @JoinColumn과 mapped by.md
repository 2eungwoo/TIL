
## @JoinColumn

FK 매핑에 사용된다. 

name 속성으로 매핑할 FK 컬럼명을 지정한다.

| 속성 | 기능 | 기본값 |
| --- | --- | --- |
| name | 매핑할 FK 컬럼 명 | 필드명_[참조하는 테이블의 PK 컬럼명] |
| referencedColumnName | FK가 참조하는 대상 테이블의 컬럼 명 | 참조하는 테이블의 PK 컬럼 명 |
| foreignKey (DDL) | 테이블 생성 시 FK 제약조건을 직접 지정할 때 사용 |  |
| uniquenullableinsertableupdatablecolumnDefinitiontable | @Column의 속성과 동일 |  |

name과 referencedColumnName **착각하지 말자**

name : FK의 이름 지어주기

referencedColumnName **:** 해당 FK가 대상 테이블의 어떤 컬럼을 참조하는지 지정하기

## 연관관계 주인

테이블은 FK 하나로 두 테이블의 연관관계를 관리하지만, 객체는 양방향으로 사용하려면 각 객체가 서로를 참조해야 하므로 두 번의 참조가 필요하다.

따라서 객체를 테이블에 매핑할 때 두 객체 중 어떤 객체로 FK를 관리해야 할지 지정해줘야 한다.

여기서 **FK를 관리하게 되는 객체**가 **연관관계의 주인**이다.

### 방법

테이블의 FK가 있는 곳이 연관관계의 주인이 된다.

DB 테이블의 1:N 관계에서 N쪽이 항상 FK를 갖는다.


### 주의

양방향 매핑 시 연관관계의 주인을 지정해줘야 한다.

연관관게의 주인만이 FK를 생성,수정,삭제 가능하며 주인이 아닌 쪽은 읽기만 가능하다.

따라서 연관관계의 주인에 값을 세팅해줘야하고, 가급적 두 객체 모두에게 값을 세팅해주는 것이 좋다.

이 과정이 생략되면 연관관계 매핑이 들어가지 않는 문제가 발생한다.

## mappedBy

연관관계 주인을 지정해주는 속성

```java
mappedBy = "{name}"
# 나는 내 연관관계에서 주인의 name 필드에 해당한다!
```

### 방법

- 연관관게의 주인은 mappedBy 속성을 사용하지 않는다.
- 주인이 아니라면 mappedBy 속성을 사용해서 주인이 아님을 설정한다. 속성 값으로는 연관관게의 주인을 지정해준다.
- mappedBy 속성 값에 들어올 이름은 연관관계의 주인의 해당 속성 필드명과 일치시킨다.
- N side가 항상 연관관계의 주인이 되므로 @ManyToOne에는 mappedBy 속성이 없다.

```java
@Entity
public class Member {
	...
	@ManyToOne // N side 이므로 연관관계 주인
	@JoinColumn(name = "team_id")
	private Team team;
}

@Entity
public class Team {
	...
	@OneToMany(mappedBy = "team") // 나(Team)은 연관관계 주인이 아니고, 내 주인은 Member의 team 필드이다!
	private List<Member> members = new ArrayList<>();
}
```

이제 Team 의 members는 Member 가 주인이고 생성,수정,삭제 권한은 Member의 team이 갖게 되었다.

Team에서는 members를 조회만 가능하다.

### mappedBy 미사용 시 발생하는 문제

```java
@Entity
public class Member {
	...
	@ManyToOne
	@JoinColumn(name = "team_id")
	private Team team;
}

@Entity
public class Team {
	...
	@OneToMany
	private List<Member> members = new ArrayList<>();
}
```

위 에시에서 `mappedBy`를 사용하지 않고 양방향 관계를 설정하면 `Hibernate`는 `Team`과 `Member`를 연결하기 위해 중간 테이블을 생성한다.

mappedBy 미사용 시 발생하는 문제는 다음과 같다.

### 1. 중간 테이블 생성

Hibernate는 `Team`과 `Member`를 연결하기 위해 `team_members`라는 중간 테이블을 생성한다.

이 테이블은 두 엔티티의 관게를 관리한다. 이는 불필요한 테이블을 생성하게 되어 데이터베이스의 복잡성 증가의 요인이 될 수 있다.

### 2. 데이터 정합성 문제

`mappedBy`를 사용하지 않으면 두 엔티티의 연관관계를 각각 독립적으로 관리하게 된다. 이는 데이터 정합성 문제가 발생될 수 있다.

예를 들어 멤버 X가 팀을 이적한다(A→B)

1. Member X를 Team B로 이적시키기 위해 `Member` 엔티티의 `team` 필드를 Team B로 변경
    
    ```java
    member.setTeam(teamB);
    ```
    
2. Team A에서 Member X를 제거하고, Team B에 Member X를 추가
    
    ```java
    teamA.getMembers().remove(member);
    teamB.getMembers().add(member);
    ```
    

이 때, 두 단계가 각각 독립적으로 이루어지기 때문에 중간 테이블을 통해 관계를 관리하는 경우 데이터 정합성 문제가 발생할 수 있다. 예를 들어 `Team`엔티티에서 `Member`를 추가/제거 하는 로직이 누락되거나 일관성이 없으면, DB에 잘못된 관계가 저장될 수 있다.

### 해결 방법 : mappedBy

`mappedBy`를 사용하여 연관관계의 주인을 지정해주고, 한 쪽에서만 관계를 관리하도록 한다. 

```java
@Entity
public class Member {
	...
	@ManyToOne 
	@JoinColumn(name = "team_id")
	private Team team;
}

@Entity
public class Team {
	...
	@OneToMany(mappedBy = "team") 
	private List<Member> members = new ArrayList<>();
}
```

이제 Member 엔티티의 team 필드를 연관관계 주인으로 설정했고, Team 엔티티에서 members 필드는 읽기 전용이 된다. 따라서 Team과 Member의 관계를 Member 엔티티의 team 필드를 통해서만 관리한다. (데이터 정합성 문제 방지)

### 결론

mappedBy를 사용하지 않으면 Hibernate가 중간 테이블을 생성하고, 관계를 독립적으로 관리하게 되어 데이터 정합성 문제가 발생할 수 있다. 

따라서 mappedBy를 사용하여 연관관계의 주인을 지정하고, 한 쪽에서만 관계를 관리할 수 있도록 해야한다.
