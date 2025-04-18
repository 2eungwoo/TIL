## INTRO

프로젝트에서 쿼리 결과나 서비스 레이어의 메소드가 반환하는 dto 객체 등 stream을 활용했었다.

잘 알고 사용한 것이 아니라, ai 자문도 구해보고 다른 사람의 소스코드를 살펴보면서 익힌 일종의 야매? 였다.

작성하려는 코드에 대해 잘 알고 능동적으로 사용하기 위해 stream에 대해서 알아보자

## Stream API?

Java Stream API를 사용하기 전에는 많은 양의 데이터를 배열이나 컬렉션으로 처리했는데, 이 방법들은 데이터에 접근하기 위해 반복문을 사용해서 새로운 코드를 작성해야 하기 때문에 코드가 길어지고 복잡해지며 재상용성 또한 좋지 않았다.

Java 8 부터 등장하는 `Stream API` 는 데이터를 다양한 방식으로 읽고 쓰기 위한 공통된 방법이다.

마치 SQL 쿼리 처럼 정형화된 처리 패턴을 가지게 하여 데이터를 다룰 수 있도록 하였다.
가

## Stream API 구조

데이터를 처리하기 위한 API이므로 데이터를 각각의 단계 별로 처리하는 구조로 되어있다.

- `생성` → `중간 연산` → `최종 연산`

### 🔹생성

`생성` 은 데이터의 컬렉션을 `stream`으로 변환하는 과정으로, `stream`을 생성하는 단계이다.

이 단계의 특징은 데이터가 필요할 때만 load된다는 점이다. → `지연 평가(Lazy Evaluation)`

- **`지연 평가(Lazy Evaluation)`**
    
    중간 연산 단계에서는 연산을 호출할 때 즉시 수행되지 않고 최종 연산이 호출될 때까지 지연된다.
    
    데이터의 연속적인 흐름에 대해 중간 연산은 실행되지 않고 있다가 최종 연산을 만나면 그 때 중간 연산이 실행된다는 뜻이다.
    
    대부분의 포스트에서 하수처리장에 비유하는데, 오염물(데이터)이 침전, 정화, 소독(중간연산)의 과정을 거치는 것 처럼 물이 순차적으로 다음 연산에 도달하여 처리되는 과정을 떠올리면 쉽다.
    

대량의 데이터 셋에서 메모리 사용량을 최적화하고, 불필요한 데이터를 가져오지 않아도 되기 때문에 매우 효율적이게 된다.

### 🔹중간 연산

`중간 연산` 은 데이터 컬렉션을 원하는 형태로 만들어주는 단계이다.(filter, map, sort 등의 연산)

중간 연산의 `입력값`과 `결과값` 모두 `stream`이며, 이러한 특징에 의해 `chaining`이 가능하다.

### 🔹최종 연산

데이터 컬렉션이나 하나의 값으로 변환된 결과 값을 얻을 수 있는 단계이다.

`최종 연산` 은 결과 값을 얻고 나면(연산이 끝나면) `stream` 이 닫혀 더 이상의 컨트롤이 불가능하기 때문에 추가적인 조작이 필요하다면 새롭게 stream을 생성 후 작업해야 한다.

닫힌 `Stream` 에 대해 연산을 시도하면 `IlleagalStateException` 이 발생한다.

## Stream API 특징

### ✅ 가독성 향상

앞서 `Lazy Evalution` 의 특징에 의해 `chaining` 이 가능하기 때문에 가독성 측면에서 우수하다.

- 기존 방식 예제(반복문)
    
    ```java
    // fruits : {"Apple", "Banana", "Cherry", "Orange", "Mango"}
    // e가 포함된 과일 이름 오름차순 정렬하여 출력
    
    List<String> filteredFruits = new ArrayList<>();
    
    for(String f : fruits) {
        if(f.contains('e')) {
            filteredFruits.add(f);
        }
    }
    
    Collections.sort(filteredFruits);
    
    for(String f : filteredFruits) {
        System.out.println(f);
    }
    ```
    
- Stream API 예제
    
    ```java
    // fruits : {"Apple", "Banana", "Cherry", "Orange", "Mango"}
    // e가 포함된 과일 이름 오름차순 정렬하여 출력
    
    List<String> filteredFruits = fruits.stream()
                                        .filter(f -> f,contains('e))
                                        .sorted()
                                        .forEach(System.out::println)
    ```
    

### ✅ 병렬 처리 지원

`Stream API` 에서는 `parallel()` 또는 `parallelStream()` 이라는 연산을 추가하는 것으로 병렬 처리가 가능하다.

병렬 처리는 스트림을 여러 스레드로 분할하여 데이터를 동시에 처리하기 때문에 대규모 데이터를 처리하기 용이하며, 데이터의 순서를 보장하려면 정렬이 필요하다.

- 사용 예제
    
    ```java
    // parallel()
    fruits.stream()
             .filter(fruit -> fruit.contains("e"))
             .sorted()  
             .parallel() 
             .forEach(fruit -> System.out.println(Thread.currentThread().getName() + ": " + fruit));
             
    // parallelStream()   
    fruits.parallelStream()
             .filter(fruit -> fruit.contains("e"))
             .sorted()  
             .forEach(fruit -> System.out.println(Thread.currentThread().getName() + ": " + fruit));  
    ```
    
    > `currentThread().getName()` 으로 처리하고 있는 스레드의 이름을 알 수 있음
    > 

## Java Stream 사용 방법

`생성` → `중간 연산` → `최종 연산` 각 단계의 사용법을 구분해서 이해해보자

### 🔸생성

`Stream` 은 배열, ArrayList, Set, Map 등 컬렉션으로 생성할 수 있다. 

`stream()` 메소드를 사용한다.

```java
// ArrayList 생성
List<String> fruits = new ArrayList<>();
fruits.add("Apple");
fruits.add("Banana");

// Stream 생성
Stream<String> stream = fruits.stream();
```

```java
// Array 생성
String[] animals = {"Dog","Cat","Cow"}

// Stream 생성
Stream<String> stream1 = Arrays.stream(animals);
Stream<String> stream2 = Stream.of(animals);
```

`generate()` 혹은 `iterate()` 메소드를 이용하여 다양한 데이터 형태의 Stream을 만들 수 있다.

```java
// 0부터 시작하는 무한 스트림 생성
Stream<Integer> infiniteNumberStream = Stream.generate(() -> 0);
// 난수 생성
Stream<Integer> infiniteRandomStream = Stream.generate(() -> (int)(Math.random()*100));

// 5개만 출력
infiniteNumberStream.limit(5).forEach(System.out::println);
infiniteRandomStream .limit(5).forEach(System.out::println);
```

> `generate()`: 주어지는 `공급자(Provider)` 를 사용하여 단순한 무한 스트림을 만들 때 쓰며 `limit()` 으로 크기를 제한할 수 있다.
> 
- 참고
    
    ```java
    // 1부터 100까지 난수 생성
    Random rand = new Random();
    Stream<Integer> example = Stream.generate(() -> rand.nextInt(100)+1)
    ```
    

```java
// 1부터 시작하여 +3 씩 증가하는 무한 스트림 생성
Stream<Integer> infiniteNumberStream = Stream.iterate(1, n -> n + 3);
// 조건 추가, 1부터 시작 +3 씩 증가하지만 100 이하 까지만 생성
Stream<Integer> infiniteEvenNumberStream = Stream.iterate(1, n -> n<=100, n -> n+3);

// 5개만 출력
infiniteNumberStream.limit(5).forEach(System.out::println);
infiniteEvenNumberStream .limit(5).forEach(System.out::println);
```

> `iterate()` : 주어지는 초기값에서 시작하여 이전 요소를 사용해서 다음 요소를 연속적으로 생성하는 스트림을 만들 때 쓰며 `limit()` 으로 크기를 제한할 수 있다.
> 
- 참고
    
    ```java
    // 관계 연산이 필요한 조건은 filtering을 해야됨 (ex 짝수만)
    Stream<Integer> example = Stream.iterate(1, n -> n + 3)
    																.filter(n -> n % 2 == 0);
    ```
    
### 🔸중간 연산

생성한 스트림에 대해 데이터를 입맛대로 컨트롤하는 단계이다.

*filter*

```sql
exampleStream.filter(n -> n % 2 == 0)
```

> `filter`는 각 요소에 대해 조건을 평가하고, 조건이 `true`인 요소들만 남긴 새로운 스트림을 반환한다.
> 

*map*

```sql
exampleStream.map(n -> -n)
```

> `map`은 스트림의 각 요소를 주어진 함수에 따라 변환한 새로운 스트림을 반환한다.


*flatMap*

```sql
List<List<String>> nestedList = List.of(
    List.of("a", "b"),
    List.of("c", "d"),
    List.of("e")
);

List<String> flatList = nestedList.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());

System.out.println(flatList); // [a, b, c, d, e]

```

> 여러 개의 스트림(Stream<Stream<T>>)을 하나로 평탄하게 합쳐준다. (+중첩 제거)
>

*sorted*

```sql
exampleStream.sorted()
```

> `sorted`는 스트림의 요소들을 오름차순(기본 정렬 기준) 또는 지정된 비교자(comparator)에 따라 정렬한 새로운 스트림을 반환한다.
> 

*peak*

```sql
exampleStream.map(n -> -n).peek(n -> System.out.println(n))
```

> `peek`는 각 요소를 소비하지 않고 중간에 엿보며(주로 디버깅용) 작업을 수행할 수 있게 해주는 중간 연산이다.
> 

*distinct*

```sql
exampleStream.map(n -> n%10).distinct()
```

> `distinct`는 중복된 요소를 제거하고 고유한 요소만 포함한 새로운 스트림을 반환한다.
> 

*limit*

```sql
exampleStream.limit(10)
```

> `limit(n)`은 스트림의 앞에서부터 최대 n개의 요소만 남긴 새로운 스트림을 반환한다.
> 

### 🔸최종 연산

중간 연산을 통해 만들어진 데이터는 또 다른 데이터 컬렉션으로 변환된다.

이러한 결과를 처리해주는 연산들이다.

*출력*

```sql
forEach()
```

> 각 요소를 순회하며 연산한다
> 

*소모*

```sql
reduce()
```

> 스트림의 요소를 줄여나가면서 연산을 수행한다. 첫 요소로 연산하고 그 결과를 다음 요소와 연산해서 최종 값을 만든다.
> 

*검색*

```sql
findFirst() : 시퀀셜 처리
findAny() : 병렬 처리
```

> 특정 조건에 맞는 요소를 찾는다.
> 

*검사*

```sql
anyMatch(), allMatch(), noneMatch()

# anyMatch() : 하나라도 조건 만족하면 true
List<Integer> list = List.of(1, 2, 3, 4, 5);
boolean hasEven = list.stream().anyMatch(n -> n % 2 == 0);
System.out.println(hasEven ); // true 

# allMatch() : 모두 조건 만족하면 true
List<Integer> list = List.of(2, 4, 6);
boolean allEven = list.stream().allMatch(n -> n % 2 == 0);
System.out.println(allEven); // true 

# 조건을 만족하는 게 하나도 없으면 true
List<Integer> list = List.of(1, 3, 5);
boolean noneEven = list.stream().noneMatch(n -> n % 2 == 0);
System.out.println(noneEven); // true (짝수가 하나도 없음)

```

> 조건에 맞는지 확인을 위해 사용한다.
> 

*계산*

```sql
count(), min(), max(), sum(), average()
```

*수집*

```sql
collect()

/*

Collectors 제공 메소드
	- toList()
	- toSet()
	- toMap()
	- toArray()
	- toCollection()
			 ex)Collectors.toCollection(ArrayList:new)	
*/
```

> 스트림의 요소를 수집하여 원하는 형태로 변환한다.
> 

> `Java Collector` 인터페이스를 매개변수로 호출하는데,  Java에서 제공하는 `Collectors 클래스`에서 이미 만들어둔 메소드를 이용해서 요소를 변환한다.

- 참고
    
    https://sigridjin.medium.com/java-stream-api%EB%8A%94-%EC%99%9C-for-loop%EB%B3%B4%EB%8B%A4-%EB%8A%90%EB%A6%B4%EA%B9%8C-50dec4b9974b
    
*reference:*
https://p829911.tistory.com/20<br/>
https://velog.io/@chamominedev/%EC%8A%A4%ED%8A%B8%EB%A6%BCstream%EC%9D%B4%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80<br/>
