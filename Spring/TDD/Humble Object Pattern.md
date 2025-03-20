

테스트 코드와 프로덕트 코드를 병행 작성하다보니 생긴 몇 가지 생각들이 있다.

첫 번째로 테스트 코드 작성에 생각보다 많은 노력을 기울이게 된다.

두 번째로 테스트 코드를 깔끔하게 작성하기 위해 프로덕트 코드의 구조를 생각해보게 된다.

테스트 코드를 작성하면서 자연스레 코드의 품질을 생각하게 되는 현상이라 생각하면서 험블 객체 패턴에 대해 정리해본다.

<br/>

## 험블 객체 패턴(Humble Object  Pattern)

```java
테스트하기 어려운 행위와 테스트하기 쉬운 행위를 분리하는 디자인 패턴
단위 테스트 작성자가 테스트하기 어려운 행위를 쉽게 할 수 있도록 한다.
```

> 테스트가 어려운 행위들을 단위 테스트가 쉬운 행위와 어려운 행위 두 가지 모듈 또는 클래스로 나누는 것.
> 
> 테스트가 어려운 모듈이 험블 객체 부분이다.


<br/>

## 예제(미적용)
- 시나리오 : 사용자가 상품을 주문하면 DB에 저장하고 로깅

### Order
```java
@Entity
@Getter
@NoArgsConstructor
@AllArgsConstructor
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String productName;
    private int quantity;
}
```


### OrderService
```java
@Service
@Slf4j
@RequiredArgsConstructor
public class OrderService {
    private final OrderRepository orderRepository;

    @Transactional
    public void placeOrder(String productName, int quantity) {
        // 비즈니스 로직 (유효성 검사)
        if (quantity <= 0) {
            log.warn("주문 실패: 수량이 0 이하입니다.");
            return;
        }

        Order order = new Order(productName, quantity);
        orderRepository.save(order);

        log.info("주문 성공: {}, 수량: {}", productName, quantity);
    }
}
```

> 비즈니스 로직과 DB 처리, 로그 처리 모두 한 클래스에 섞여있는 모습
> 

> 유효성 검사 로직을 테스트하려면, DB와 로깅을 모킹해야하므로 단위 테스트가 어렵다.
> 

> 새로운 비즈니스 로직(새로운 검증 로직 등) 추가되면 OrderService를 직접 수정해야함
> 

<br/>

## 예제(적용)
- 시나리오 : 사용자가 상품을 주문하면 DB에 저장하고 로깅
- 목표 : Humble Object Pattern 적용 → 비즈니스 로직과 DB+로그 처리를 분리

### OrderProcessor
```java
@Component
public class OrderProcessor {
    public boolean validateOrder(int quantity) {
        return quantity > 0; // 주문 수량이 0보다 커야 유효
    }
}
```

> 비즈니스 로직(주문 유효성 검증)만 담당 → 단위 테스트 가능
> 

### OrderService
```java
@Service
@Slf4j
@RequiredArgsConstructor
public class OrderService {
    private final OrderProcessor orderProcessor;
    private final OrderRepository orderRepository;

    @Transactional
    public void placeOrder(String productName, int quantity) {
        if (!orderProcessor.validateOrder(quantity)) {
            log.warn("주문 실패: 수량이 0 이하입니다.");
            return;
        }

        Order order = new Order(productName, quantity);
        orderRepository.save(order);
        log.info("주문 성공: {}, 수량: {}", productName, quantity);
    }
}
```

> 험블 객체.  DB 처리, 로그 처리만 담당 → 테스트하기 어려운 작업만 포함
> 

<br/>

## 테스트 코드
### OrderProcessorTest
```java
public class OrderProcessorTest {
    private final OrderProcessor orderProcessor = new OrderProcessor();

    @Test
    void testValidOrder() {
        assertTrue(orderProcessor.validateOrder(5));
    }

    @Test
    void testInvalidOrder() {
        assertFalse(orderProcessor.validateOrder(0)); 
    }
}
```

> 순수한 비즈니스 로직만 쉽게 단위 테스트 가능
> 

### OrderServiceTest
```java
public class OrderServiceTest {
    private OrderProcessor mockProcessor;
    private OrderRepository mockRepository;
    private OrderService orderService;

    @BeforeEach
    void setUp() {
        mockProcessor = Mockito.mock(OrderProcessor.class);
        mockRepository = Mockito.mock(OrderRepository.class);
        orderService = new OrderService(mockProcessor, mockRepository);
    }

    @Test
    void testPlaceOrder_Success() {
        when(mockProcessor.validateOrder(5)).thenReturn(true);

        orderService.placeOrder("Laptop", 5);

        verify(mockProcessor, times(1)).validateOrder(5);
        verify(mockRepository, times(1)).save(any(Order.class));
    }

    @Test
    void testPlaceOrder_Fail() {
        when(mockProcessor.validateOrder(0)).thenReturn(false);

        orderService.placeOrder("Laptop", 0);

        verify(mockProcessor, times(1)).validateOrder(0);
        verify(mockRepository, times(0)).save(any(Order.class));
    }
}
```

> Mocking 사용으로 DB 연동 없이 `OrderService.Class` 의 동작만 테스트 가능
> 

<br/>

## 객체지향 원칙
SRP, OCP가 자연스럽게 지켜지는 느낌

### 1. 단일 책임 원칙 

- 하나의 클래스가 여러 역할을 수행하지 않도록 분리

| 역할 | 책임 | 적용된 클래스 |
| --- | --- | --- |
| **비즈니스 로직 처리** | 주문의 유효성 검증 | `OrderProcessor` |
| **DB 저장 & 로깅** | 데이터 저장, 로그 기록 | `OrderService` |

### 2. 개방-폐쇄 원칙

- 새로운 비즈니스 로직(주문 검증 규칙 등) 추가하려면? → `OrderProcessor` 내부 로직만 수정하면 됨

<br/>

## 결론

**Hubmle Object Pattern?**

- 비즈니스 로직을 테스트하기 쉬운 객체로 분리하여 테스트하기 어려운 부분(DB 연동 등)은 최소한의 역할만 수행하는 객체에 배치한다.
- 비즈니스 로직을 쉽게 단위 테스트로 뺼 수 있다.
- DB, 로깅 등 분리하여 코드 유지보수 용이함
- 책임 분리로 코드가 깔끔해짐
- 이전에 알아봤었던 when().Return이니 doReturn().when()이니 남발했던 거 다 코드가 구려서 복잡하게 테스트 해야했던거임
