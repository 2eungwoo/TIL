## 1️⃣ 트랜잭션이란?

`트랜잭션(Transaction)`이란 데이터베이스에서 수행되는 하나 이상의 작업을 하나의 논리적인 단위로 묶은 것을 말한다.

> 즉, "모든 작업이 성공해야 저장하고, 하나라도 실패하면 전부 되돌린다"는 개념
> 

### ✅ 핵심 연산

- `Commit`: 모든 작업이 정상 처리 → DB에 반영
- `Rollback`: 중간에 실패 → 이전 상태로 되돌림

*💰 예시: A → B 계좌 송금*

```sql
@Transactional
public void transfer(Account from, Account to, int amount) {
    from.withdraw(amount);   // A 계좌에서 출금
    to.deposit(amount);      // B 계좌로 입금
}
```

> 입금 중 오류가 나면 출금도 되돌려야 한다. 이것이 `트랜잭션`
> 

## 2️⃣ 트랜잭션의 4대 특성: **ACID**

트랜잭션은 아래 네 가지 특성을 만족해야 한다.

| 특성 | 설명 |
| --- | --- |
| **Atomicity (원자성)** | 모두 성공하거나, 모두 실패해야 한다. |
| **Consistency (일관성)** | 트랜잭션 전후 DB 상태가 무결성을 만족해야 한다. |
| **Isolation (고립성)** | 다른 트랜잭션과 상호 간섭이 없어야 한다. |
| **Durability (지속성)** | 커밋된 데이터는 영구적으로 저장되어야 한다. |

### 고립성(Isolation)이 중요한 이유

고립성이 없으면 다음과 같은 문제가 발생할 수 있다.

| 문제 유형 | 설명 |
| --- | --- |
| Dirty Read | 아직 커밋되지 않은 데이터를 읽음 |
| Non-repeatable Read | 같은 row를 두 번 읽었는데 값이 바뀜 |
| Phantom Read | 같은 조건으로 쿼리했는데 행의 수가 바뀜 |

## 3️⃣ 격리 수준 (Isolation Level)

스프링에서는 `@Transactional(isolation = ...)` 와 같은 방식으로 격리 수준을 설정한다.

격리 수준은 ACID의 "Isolation"을 어떻게 구현할지를 결정한다.

```java
@Transactional(isolation = Isolation.READ_COMMITTED)

```

### 격리 수준별 정리

| 수준 | 읽기 허용 범위 | 방지하는 문제 | 설명 |
| --- | --- | --- | --- |
| `READ_UNCOMMITTED` | 커밋되지 않은 데이터 | 없음 ❌ | Dirty Read 허용 (거의 안 씀) |
| `READ_COMMITTED` | 커밋된 데이터만 | Dirty Read | 대부분 RDB의 기본값 |
| `REPEATABLE_READ` | 첫 조회 기준 고정 | Dirty, Non-repeatable Read | MySQL 기본값 |
| `SERIALIZABLE` | 완전 직렬화 | 모든 문제 방지 | 가장 안전하지만 성능 저하 큼 |

### 💵 격리 수준 예제

*🧪 `READ_UNCOMMITTED` - Dirty Read 가능*

```java
@Transactional(isolation = Isolation.READ_UNCOMMITTED)
public Account getAccount(Long id) {
    return accountRepository.findById(id).orElseThrow();
}
```

> 다른 트랜잭션이 아직 커밋하지 않은 변경 데이터도 읽을 수 있음 → 데이터 신뢰도 낮음
> 

*🧪 `READ_COMMITTED` - Dirty Read 방지*

```java
@Transactional(isolation = Isolation.READ_COMMITTED)
public void process() {
    User u1 = userRepository.findById(1L).get();
    // 중간에 다른 트랜잭션이 변경 후 커밋
    User u2 = userRepository.findById(1L).get();
    // u1 != u2 일 수 있음 (Non-repeatable Read)
}
```

> 커밋된 데이터만 읽지만 같은 row를 다시 읽을 때 값이 달라질 수 있음
> 

🧪 `REPEATABLE_READ` - Non-repeatable Read 방지

```java
@Transactional(isolation = Isolation.REPEATABLE_READ)
public void checkConsistency() {
    Product p1 = productRepository.findById(1L).get();
    // 중간에 다른 트랜잭션이 수정했어도,
    Product p2 = productRepository.findById(1L).get();
    // p1 == p2 → 동일한 값 보장
}
```

> 새로운 row가 나타나는 "Phantom Read"는 허용될 수 있음
> 

*🧪 `SERIALIZABLE` - 완전 직렬화*

```java
@Transactional(isolation = Isolation.SERIALIZABLE)
public void checkOrders() {
    int before = orderRepository.countByUserId(1L);
    // 다른 트랜잭션에서 INSERT 발생
    int after = orderRepository.countByUserId(1L);
    // before == after → 새로 삽입된 row 안 보임
}
```

> 가장 안전한 동시성 처리 방법이지만 락이 많이 걸리므로 속도 저하 커서 사용 시 주의해야함
> 

### 📒 정리 - 언제 어떤 격리 수준을 쓸까?

| 상황 | 추천 격리 수준 |
| --- | --- |
| 단순 조회 | `READ_COMMITTED` |
| 정확한 금액 처리, 중복 방지 | `REPEATABLE_READ` |
| 높은 무결성 보장, 금융/회계 | `SERIALIZABLE` |
| 테스트용, 분석용 빠른 쿼리 | `READ_UNCOMMITTED` (주의) |

*reference*
- [공식 Spring Framework 문서 - Transaction Management](https://docs.spring.io/spring-framework/reference/data-access/transaction.html)
- Baeldung - Spring @Transactional
- [위키백과 - ACID](https://en.wikipedia.org/wiki/ACID)
- [MySQL 문서 - Transaction Isolation Levels](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html)
- [테코톡](https://www.youtube.com/watch?v=taUeIi6a6hk)

