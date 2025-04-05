
## 🌀 스프링부트 `@Transactional` ?

스프링을 사용하면서 데이터베이스와 관련된 작업을 할 때 빠지지 않고 등장하는 것이 바로 `@Transactional` 어노테이션이다. 하지만 무지성으로 붙여서 사용하면 위험. 

트랜잭션이 진짜로 작동하고 있는지, 기대한 대로 롤백되는지, 성능에 영향을 주는 건 아닌지 확실하게 이해하고 있어야 한다.

---

### ⚙️ `@Transactional`의 동작 원리

스프링에서 `@Transactional`은 AOP 기반의 프록시 패턴을 이용해서 동작한다. 

스프링이 우리가 만든 클래스의 프록시 객체를 생성해서 그 프록시를 통해 트랜잭션을 자동으로 처리해주는 방식이다.

### ✔️ 프록시 기반 구조

*UserService.class*

```java
@Service
public class UserService {

    @Transactional
    public void registerUser(User user) {
        userRepository.save(user);
        // ...
    }
}
```

```
Client → UserServiceProxy.registerUser()
                        ㄴ 트랜잭션 시작
                        ㄴ 실제 메서드 실행
                        ㄴ 예외 확인 후 커밋 또는 롤백

```

> 실제로 `UserService`가 호출되는 게 아니라, **`UserService`의 프록시 객체**가 호출된다.
> 

> 이 프록시는 메서드 호출 전/후에 트랜잭션을 열고 닫는 로직을 자동으로 감싼다.
> 

---

## ✅ 기본 사용법

```java
@Transactional
public void createOrder(Order order) {
    orderRepository.save(order);
}
```

트랜잭션은 메서드가 성공적으로 완료되면 커밋되고, `RuntimeException`이 발생하면 자동으로 롤백된다.

---

## ⚠️ 주의

### 1. 예외가 발생해도 롤백되지 않는 경우

기본적으로 `RuntimeException` (또는 Error)만 롤백 대상임

```java
@Transactional
public void fail() throws Exception {
    throw new Exception("롤백되지 않음 ❌");
}
```

- 😵 왜 `Exception`을 던졌는데 롤백이 안 될까?
    
    [Java 오류(Error)와 예외(Exception) 클래스 이해하기](https://www.notion.so/Java-Error-Exception-f8676a1512c34236ae3dab0d46bee61b?pvs=21) 
    
    ### 스프링의 기본 트랜잭션 롤백 정책:
    
    - ✅ **롤백**: `RuntimeException` (혹은 `Error`) 발생 시
    - ❌ **롤백 안 함**: `Checked Exception` (예: `Exception`, `IOException`) 발생 시
        
        ---
        
        | 종류 | 예시 | 특징 |
        | --- | --- | --- |
        | **Checked Exception** | `IOException`, `SQLException`, `Exception` | 반드시 `throws` 처리해야 함 |
        | **Unchecked (Runtime) Exception** | `NullPointerException`, `IllegalArgumentException`, `RuntimeException` | 컴파일러가 강제하지 않음 |
        
        스프링은 예외가 `'예상 가능한 흐름'인지`, `진짜 '오류'인지` 구분해서 처리한다.
        
        그래서 `RuntimeException`은 ’에러’니까 롤백하고, `Checked Exception`은 ‘업무 흐름 상 예외일 수 있으니 롤백하지 말자’라는 전략이다.
        
        ---
        

### 해결 방법:

`@Transactional(rollbackFor = Exception.class)` 이런 식으로 직접 명시하면 된다.

```java
@Transactional(rollbackFor = Exception.class)
public void fail() throws Exception {
    throw new Exception("이제 롤백됨 ✅");
}
```

> `Checked Exception`을 써야 하는 상황이라면 `rollbackFor`를 반드시 명시
> 

## 💡TIP

- `Unchecked (Runtime) Excepion`을 상속하는 커스텀 예외를 만들어서 사용한다.

```java
public class MyCustomException extends RuntimeException {
    public MyCustomException (String msg) {
        super(msg);
    }
}
```

> 그러면 일일이 명시 안해줘도 됨
> 

---

### 2. 자기 자신 호출 시 트랜잭션이 작동하지 않음

프록시는 외부에서 호출될 때만 동작하기 때문에 클래스 내부에서 `this.메서드()`로 호출하면 트랜잭션이 적용되지 않음

```java
@Service
public class OrderService {

    public void outer() {
        this.inner(); // ❌ 트랜잭션 안 걸림
    }

    @Transactional
    public void inner() {
        // 트랜잭션 적용 안 됨
    }
}
```

### 해결 방법

- 클래스 분리

```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final InnerService innerService;

    public void outer() {
        System.out.println("🚀 outer 실행");
        innerService.inner(); // ✅ 프록시 거침 → 트랜잭션 정상 작동
    }
}

@Service
public class InnerService {

    @Transactional
    public void inner() {
        System.out.println("✅ 트랜잭션 적용된 inner 실행");
        // 여기서 DB 작업 등 수행 → 트랜잭션 적용됨
    }
}
```

> 클래스 구조를 나눠 외부에서 호출되게 하자
> 

---

## 🧊 격리 수준 (Isolation Level)

트랜잭션 간의 동시성 문제를 제어하는 설정. 기본 값은 데이터베이스 벤더에 따라 다르며 보통 `READ_COMMITTED`가 기본임

```java
@Transactional(isolation = Isolation.READ_COMMITTED)
```

| 옵션 | 설명 |
| --- | --- |
| `READ_UNCOMMITTED` | 다른 트랜잭션의 변경 내용을 커밋 전에 읽을 수 있음 (Dirty Read 허용) |
| `READ_COMMITTED` | 커밋된 데이터만 읽을 수 있음 (기본값) |
| `REPEATABLE_READ` | 트랜잭션 동안 읽은 데이터가 항상 동일함 (Phantom Read 허용 X) |
| `SERIALIZABLE` | 가장 엄격한 수준, 완전한 직렬화. 성능 저하 가능 |

각 설정은 동시성 이슈(Dirty Read, Non-repeatable Read, Phantom Read)를 얼마나 허용할지에 따라 선택된다.

---

## 🧪 readOnly 설정

조회만 수행하는 메서드에 `readOnly = true`를 설정하면 스프링이 dirty checking 같은 작업을 생략한다.

```java
@Transactional(readOnly = true)
public User getUser(Long id) {
    return userRepository.findById(id).orElseThrow();
}
```

---

*reference*

- [공식 문서: Spring Transaction Management](https://docs.spring.io/spring-framework/reference/data-access/transaction.html)
- [Baeldung: Guide to Spring Transaction Management](https://www.baeldung.com/transaction-configuration-with-jpa-and-spring)
- https://velog.io/@betterfuture4/Spring-Transactional-%EC%B4%9D%EC%A0%95%EB%A6%AC
