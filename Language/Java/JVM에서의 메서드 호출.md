## 📘 INTRO

정처기 실기 시험 직후 자식 클래스 생성자 호출 시 이뤄지는 일련의 과정에 대해 알아보다가 동적 디스패치(Dynamic Dispatch) 개념에 대해 알게 되었다.

처음에는 오버라이딩된 메서드가 왜 자식 클래스 기준으로 호출되는 걸까? 하는 단순한 궁금증에서 출발했고 JVM 내부에서 실제로 어떻게 메서드가 호출되는지, 특히 런타임에 어떤 방식으로 실제 메서드 구현을 결정하는지에 대해 궁금증이 생겼다.

JVM의 내부 구조, 특히 메서드 호출 시 어떤 방식으로 실제 구현체를 찾아가는지에 대한 개념을 정리하면서 정적 디스패치 vs 동적 디스패치, 그리고 이를 가능하게 하는 vtable(가상 메서드 테이블) 구조까지 알아보고 정리해보자

---

## ✅ JVM에서 메서드는 어떻게 호출되나?

자바에서 코드가 실행될 때, JVM은 클래스를 메모리에 로드하고 메서드를 실행한다.

이때 **어떤 메서드를 호출할지** JVM이 결정하는 방식에는 두 가지가 있다:

- 참고
    
    [Static/Dynamic Dispatch](https://www.notion.so/Static-Dynamic-Dispatch-1db1f679d4cb80a4ab47ccbe6fb1b948?pvs=21) 
    

### **🌀 메서드 호출 방식 두 가지**

| **정적 디스패치** (Static Dispatch) | 컴파일 타임에 메서드 결정 | 컴파일 타임 |
| --- | --- | --- |
| **동적 디스패치** (Dynamic Dispatch) | 런타임에 실제 객체 타입을 기준으로 메서드 결정 | 런타임 |

## **⚙️ 정적 디스패치 (Static Dispatch)**

> 오버로딩, static 메서드 등은
> 
> 
> **`정적(Static)`**
> 

```java
class Printer {
    void print(int x) {
        System.out.println("int: " + x);
    }
    void print(String x) {
        System.out.println("String: " + x);
    }

    static void hello() {
        System.out.println("Hello from static");
    }
}
```

```java
Printer p = new Printer();
p.print(10);       // 컴파일 타임에 결정 → int 버전
p.print("hello");  // 컴파일 타임에 결정 → String 버전
Printer.hello();   // 클래스 기준으로 호출
```

> static 메서드는 클래스 타입 기준, 오버로딩은 인자의 타입을 기준으로 컴파일 타임에 결정
> 

---

## **⚙️ 동적 디스패치 (Dynamic Dispatch)**

> 오버라이딩된 메서드는
> 
> 
> **실제 객체 타입 기준으로 런타임에 결정**
> 

```java
class Parent {
    void show() {
        System.out.println("Parent show");
    }
}

class Child extends Parent {
    @Override
    void show() {
        System.out.println("Child show");
    }
}

Parent p = new Child();
p.show(); // 🚨 컴파일 시점에는 Parent 기준이지만, 런타임에는 Child.show() 호출됨!
```

> 실제 객체의 타입이 Child이므로 `show()`는 Child의 것이 호출
> 

---

### **🧩** 자바에서는 이 런타임 결정(동적 디스패치)을 효율적으로 처리하기 위해 클래스마다 `vtable` 이라는 구조를 사용한다

## **🗂️ vtable?**

> Virtual Method Table
> 

- JVM이 오버라이딩된 인스턴스 메서드를 효율적으로 호출하기 위해 사용하는 클래스 단위의 메서드 주소 테이블

- 런타임에 어떤 메서드를 호출할지 결정

### 👽 **동작 흐름**

1. 부모 클래스의 메서드를 자식 클래스가 오버라이딩 
    
    → 자식 클래스의 vtable에서 해당 메서드의 주소로 대체
    
2. 객체가 인스턴스 메서드를 호출 
    
    → 자신의 클래스의 vtable을 참조해서 실제 메서드 주소를 찾아 호출
    

오버라이딩 메서드의 실제 주소를 매번 찾아서 실행시키는 방법은 성능 관점에서 비효율적일 수 있다.

`vtable` 을 이용하면 바로 점프해서 찾을 수 있으므로 메모리 최적화 효과를 볼 수 있다.

### 🎃 메서드 호출 방식과의 관계

| **메서드 종류** | **디스패치 방식** | **vtable 사용 여부** | **설명** |
| --- | --- | --- | --- |
| static 메서드 | 정적 디스패치 | ❌  | 클래스 단위로 고정되어 있음 |
| final 메서드 | 정적 디스패치 | ❌ | 오버라이딩이 불가능하므로 고정 |
| 오버로딩된 메서드 | 정적 디스패치 | ❌ | 인자 타입 기준으로 컴파일 타임에 결정 |
| 오버라이딩된 메서드 | **동적 디스패치** | ⭕ | vtable을 통해 런타임에 메서드 결정 |

---

## 💣 **예제: 오버라이딩 시 vtable 갱신**

> 부모 클래스의 메서드를 자식 클래스가 오버라이딩 → 자식 클래스의 vtable에서 해당 메서드 주소로 대체
> 

```java
class Parent {
    void hello() {
        System.out.println("Hello from Parent");
    }
}

class Child extends Parent {
    @Override
    void hello() {
        System.out.println("Hello from Child");
    }
}
```

### **🔍 vtable 관점:**

- Parent 클래스의 vtable:

```java
hello() → Parent.hello()
```

• Child 클래스의 vtable (오버라이딩됨):

```java
hello() → Child.hello()   ← 부모의 것을 덮어씀
```

## 💣 **예제: 객체가 메서드를 호출할 때 vtable 참조**

> 객체가 인스턴스 메서드를 호출 → 자신의 클래스의 vtable을 참조해서 실제 메서드 주소를 찾아 호출
> 

---

```java
Parent obj = new Child();
obj.hello(); // Hello from Child
```

### **🔍 vtable 기반 호출 흐름:**

1. 변수 타입은 `Parent` 이지만 `new Child()`로 생성했으므로
    
    → JVM은 `Child`의 vtable을 참조
    
2. `hello()` 메서드를 호출 하면
    
    → `Child`의 vtable에 있는 `hello() → Child.hello()` 호출
