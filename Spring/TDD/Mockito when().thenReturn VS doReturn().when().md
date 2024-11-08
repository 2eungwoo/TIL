## Mockito에서 `when().thenReturn()` 과 `doReturn().when()` 차이

간혹 when().thenReturn()을 쓰면 테스트에 통과하지 못하고, doReturn().when()을 쓰면 통과하는 그런 경우가 있어 그 차이를 알아보고 기록한다. 

두 방법 모두 메서드가 예상대로 동작할 때 반환 값을 설정하는 두 가지 주요 방법이다.

주로 서비스 레이어 테스트코드를 작성할 때(extract method 테스트) 사용하고 있는데, 두 방법은 서로 비슷해 보이지만 실제로 동작 방식(특히 순서)과 용도에 중요한 차이가 있다. 이에 유의하여 목적에 따라 알맞게 사용해야 한다.

### 1. `when(...).thenReturn(...)`

가장 일반적인 Mockito 설정 방식으로 메서드가 호출될 때 반환할 값을 설정한다. 이 방식은 메서드가 정상적으로 실행되고 결과를 반환해야 할 때 사용한다. 즉, 해당 메서드가 호출되고 실제로 값을 반환하는 상황에서 사용된다.

### 예시 코드 (기본 사용법)

```java
// SampleService.java
public class SampleService {
    public String getData() {
        return "hello";
    }
}

// SampleServiceTest.java
@Test
void testUsingWhenThenReturn() {
    // given
    SampleService sampleService = mock(SampleService.class);
    when(myService.getData()).thenReturn("hello");

    // when
    String result = sampleService.getData();

    // then
    assertEquals("hello", result);
}

```

- `when(myService.getData()).thenReturn("Data")`는 `getData()` 메서드가 호출될 때마다 `"hello"`를 반환하도록 설정한다.
- `when(...).thenReturn(...)`은 메서드가 **정상적으로 호출될 때만** 값을 반환한다.

### 2. `doReturn(...).when(...)`

메서드 호출을 **실행하지 않고** 반환 값을 설정하는 방식이다. 이 방법은 메서드가 부작용을 발생시키거나 예외를 던질 가능성이 있을 때 사용한다. `doReturn(...).when(...)`은 메서드 호출을 건너뛰고 결과만 반환하므로, 호출된 메서드가 예외를 던지거나 상태를 변경하는 경우에 유용하다.

### 예시 코드 (부작용이 있는 메서드에서 사용)

```java
// SampleService.java
public class SampleService {
    public String getData() {
        if (someCondition()) {
            throw new IllegalStateException("Error");
        }
        return "hello";
    }

    private boolean someCondition() {
        return true;
    }
}

// SampleServiceTest.java
@Test
void testUsingDoReturnWhen() {
    // given
    SampleService sampleService = mock(SampleService.class);

    // 예외를 피하기 위해 doReturn 사용
    doReturn("hello").when(sampleService).getData();

    // when
    String result = sampleService.getData();

    // then
    assertEquals("hello", result);
}

```

- `doReturn("Data").when(myService).getData()`는 `getData()`가 호출될 때 예외를 피하고 `"Data"`를 반환한다.
- `doReturn(...).when(...)`은 **메서드가 호출되지 않고**, 설정된 반환값을 대신 전달한다.

### 3. `when(...).thenReturn(...)`과 `doReturn(...).when(...)`의 차이점

| 특징 | `when(...).thenReturn(...)` | `doReturn(...).when(...)` |
| --- | --- | --- |
| **용도** | 메서드가 **정상적으로 실행**될 때 반환값 설정 | 메서드의 **실행을 막고** 반환값 설정 |
| **메서드 호출** | 메서드 호출 후 **직접 실행**되고 값을 반환 | 메서드 호출을 **건너뛰고** 반환값만 설정 |
| **부작용이 있는 메서드** | 부작용이 있는 메서드에서 사용할 수 없다 | 부작용이 있는 메서드에서 사용할 수 있다 |

### 4. 언제 `when(...).thenReturn(...)` or `doReturn(...).when(...)` ??

### 예시 1: `when(...).thenReturn(...)`을 사용할 수 있을 때

```java
@Test
@DisplayName("정상적으로 메서드가 호출되고 반환값이 설정되는 경우")
void testUsingWhenThenReturn() {
    // given
    SampleService sampleService = mock(SampleService.class);
    when(sampleService.getData()).thenReturn("hello");

    // when
    String result = SampleService.getData();

    // then
    assertEquals("hello", result);
}

```

### 예시 2: `doReturn(...).when(...)`을 사용해야 하는 경우

부작용이 있는 메서드에서 예외를 피하고 싶을 때 `doReturn(...).when(...)`을 사용한다.

```java
@Test
@DisplayName("메서드 호출 시 부작용을 피하려는 경우")
void testUsingDoReturnWhen() {
    // given
    SampleService sampleService = mock(SampleService.class);

    // 부작용을 피하고, 반환값만 설정
    doReturn("hello").when(sampleService).getData();

    // when
    String result = sampleService.getData();

    // then
    assertEquals("hello", result);
}

```

### 5. `doReturn(...).when(...)`을 사용하는 이유

`doReturn(...).when(...)`을 사용하는 이유는 다음과 같다:

1. **메서드가 예외를 던지는 경우**:
    - `when(...).thenReturn(...)`은 메서드가 예외를 던지지 않을 때만 사용해야 한다. 예외를 던지는 메서드에서는 `doReturn(...).when(...)`을 사용하는 것이 좋다.
2. **부작용이 있는 메서드**:
    - 메서드가 부작용을 발생시키는 경우, `doReturn(...).when(...)`을 사용하여 부작용 없이 값을 설정할 수 있다.

### 6. 결론

- 일반적인 반환값 설정에는 `when(...).thenReturn(...)`
- 메서드 호출을 건너뛰고 부작용을 피하려면 `doReturn(...).when(...)`
