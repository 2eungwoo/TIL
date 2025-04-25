## 📘 INTRO

25년 1회차 정처기 실기 시험에 나온 문제… 풀 때는 확신하고 풀었는데 가채점 복원 시트에서 사람들이 열불나게 토론하고 있길래 정확하게 이해하기 위해 알아보았다.

자바에서 생성자 호출 시 오버라이딩 메서드가 어떻게 작동하는지에 대해서는 알고 있었지만 처음 듣는 용어인 동적 디스패치 개념을 알아보자

---

## 🤯 사람들이 막 싸운다.

- 문제의 문제
    
    ### 🎯 시험 문제 복원 코드
    
    ```java
    class parent {
        static int total = 0;
        int v = 1;
    
        public parent() {
            total += (++v);
            show();  // 동적 디스패치
        }
    
        public void show() {
            total += total;
        }
    }
    
    class child extends parent {
        int v = 10;
    
        public child() {
            v += 2;           // 12
            total += v++;     // total += 12 → total = 18
            show();           // child의 show() 호출됨 → total += total * 2 = 54
        }
    
        @Override
        public void show() {
            total += total * 2;
        }
    }
    
    class Main {
        public static void main(String[] args) {
            new child();
            System.out.println(parent.total);
        }
    }
    
    ```
    
    ### ✅ 출력:
    
    ```
    54
    ```
    
    - 💡 계산 흐름
        
        
        | 단계 | 설명 | total |
        | --- | --- | --- |
        | 1 | parent 생성자: ++v → 2, total += 2 | 2 |
        | 2 | show() → child의 show() 호출됨 | 2 + 2×2 = 6 |
        | 3 | child 생성자: v=12, total += 12 | 18 |
        | 4 | show() → child의 show() | 18 + 18×2 = 54 |

## 💬 논점

> super()가 암시적으로 실행 된 건 알겠는데, 자식 클래스 생성자가 완료되기 이전 시점이니 부모 클래스의 메소드에 접근하는게 맞지 않느냐?
> 

알아보자.

---

## ✅ Dynamic Dispatch?

> "오버라이딩된 인스턴스 메서드를 호출할 때, 실제 객체의 타입을 기준으로 런타임에 결정하는 것
> 

```java
class Parent {
    Parent() {
        System.out.println("Parent constructor");
        show(); // ← 동적 디스패치로 Child.show() 호출됨
    }

    void show() {
        System.out.println("Parent show");
    }
}

class Child extends Parent {
    int x = 10;

    Child() {
        System.out.println("Child constructor");
    }

    @Override
    void show() {
        System.out.println("Child show, x = " + x); // x는 아직 0일 수도 있음!
    }
}

```

```java
Parent p = new Child();
p.show();  // 이건 런타임에 Child의 show()가 호출됨
```

이런 게 바로 **동적 디스패치(Dynamic Dispatch)**

---

## 🤔 근데 그럼 생성자 안에서도 동적 디스패치임?

### ✔️ *Yes, but...!*

- 자바는 생성자 내부에서도 **동적 디스패치를 적용한다.**
- 그래서 `Parent` 생성자에서 `show()`를 호출하면 오버라이드된 Child의 `show()` **가 호출된다.

**하지만 함정이 있음:**

- 자식 생성자 아직 실행 전 이므로 자식 필드가 초기화되지 않았을 수 있음
- → 이 때문에 정상적인 오버라이딩처럼 보이지만 예상 못한 결과가 나올 수 있음

---

### 💣 예시:

```java
class Parent {
    Parent() {
        System.out.println("Parent constructor");
        show(); // ← 동적 디스패치로 Child.show() 호출됨
    }

    void show() {
        System.out.println("Parent show");
    }
}

class Child extends Parent {
    int x = 10;

    Child() {
        System.out.println("Child constructor");
    }

    @Override
    void show() {
        System.out.println("Child show, x = " + x);
        // 이 때 x는...
    }
}

```

### ✅ 출력:

```
Parent constructor
Child show, x = 0  // ← 동적 디스패치지만 x 초기화 전
Child constructor
```

> 자식 생성자는 아직 실행 전이라 `x = 10`이 아직 초기화되지 않음 → `x = 0`
> 

## 📌 요약

| 항목 | 설명 |
| --- | --- |
| 동적 디스패치 | 자바는 오버라이딩 메서드 호출 시 항상 인스턴스 메서드를 실제 객체 타입 기준으로 호출한다. |
| 생성자 안에서도 동작함 | 부모 생성자에서 자식의 오버라이드 메서드 호출 가능 |
| 주의점 | 자식 필드는 아직 초기화 전일 수 있음 (예상치 못한 결과 발생 가능) |

> "생성자 안에서는 오버라이딩된 메서드 호출을 피하라."
> 

---

## 🤔 그럼 정적 디스패치(Static Dispatch)도 있겠네

**정적 디스패치(Static Dispatch)**는 **컴파일 타임**에 어떤 메서드가 호출될지를 결정하는 방식이다.

자바에서는 다음과 같은 상황에서 발생한다:

- **`오버로딩`된 메서드**: 같은 이름을 가진 여러 메서드가 인자 타입이나 개수에 따라 구분되는 경우
- **`static` 메서드**: 객체의 타입이 아닌 **클래스 타입**에 따라 호출되는 메서드들의 경우

### 💣 예시 1: 오버로딩된 메서드

```java
class Example {
    void print(int a) {
        System.out.println("int: " + a);
    }

    void print(String a) {
        System.out.println("String: " + a);
    }
}

public class Main {
    public static void main(String[] args) {
        Example ex = new Example();
        ex.print(5);     // 컴파일 타임에 결정, int를 받는 메서드 호출
        ex.print("Hello"); // 컴파일 타임에 결정, String을 받는 메서드 호출
    }
}

```

> 컴파일 타임에 `ex.print(5)`와 `ex.print("Hello")` 이 코드에 의해 어떤 메서드가 호출될지 결정되기 때문에 이 호출들은 **정적 디스패치(Static Dispatch)** 방식으로 처리된다.
> 

### 💣 예시 2: `static` 메서드

```java
java
복사편집
class Example {
    static void display() {
        System.out.println("Static method in Example");
    }
}

public class Main {
    public static void main(String[] args) {
        Example.display();  // 컴파일 타임에 결정된 클래스의 static 메서드 호출
    }
}

```

> `static` 메서드는 클래스 이름으로 직접 호출된다. **정적 디스패치(Static Dispatch)**에 의해 `컴파일 타임`에 호출될 메서드가 결정되는 것이다.
>
> 
---
[JVM에서의 Dispatch](https://github.com/2eungwoo/TIL/blob/main/Language/Java/JVM%EC%97%90%EC%84%9C%EC%9D%98%20Dispatch.md)
