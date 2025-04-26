## ✅ Garbage Collection?

> 프로그램에서 사용되지 않는 객체(가비지)를 찾아 메모리를 회수하는 작업/모듈
> 

이 때 가비지란 더 이상 사용되지 않는 객체를 가리키며`Garbage Collection` 은 제한된 메모리 공간을 효율적으로 정리해 새로운 객체가 원활히 할당되도록 한다.

---

## 📒 Minor GC, Major GC

JVM의 Heap 영역은 다음의 두 가지 전제를 바탕으로 설계되었다.

- 대부분의 객체는 금방 접근 불가능한 상태가 된다.
- 오래된 객체에서 새로운 객체로의 참조는 매우 적게 존재한다.

대부분의 객체는 일회성 성질이 강하고 메모리에 남아있는 시간이 짧기 때문에, 객체의 생존 시간에 따라 아래와 같이 두 가지로 Heap 영역을 분리한다.

- 🌳 **Young Generation**
    
    ### 🌳 Young Generation
    
    - 새롭게 생성된 객체가 할당되는 영역
    - 대부분의 객체가 **금방 사라지므로** 많은 객체가 이곳에서 생성되었다가 곧 GC 대상이 됨
    - 이 영역에 대한 GC → `Minor GC`
    
- 🌲 **Old Generation**
    
    ### 🌲 Old Generation
    
    - Young 영역에서 **생존**한 객체들이 **승격(Promotion)** 되는 영역
    - 상대적으로 큰 메모리 공간을 가지며, 가비지 발생 빈도는 낮음
    - Old 영역에 대한 GC → `Major GC`
    

---

## ⚡Garbage Collection 동작 방식

`Young Generation` , `Old Generation` 은 서로 다른 메모리 구조로 나뉜 영역이기 때문에 세부적인 동작 방식은 다르지만 기본적으로 GC가 실행된다고 하면 다음의 두 가지 공통적인 단계를 거친다.

- 🍨 **Stop the World**
    
    ### 🍨 Stop The World (STW)
    
    > GC를 실행하기 위해 JVM이 모든 쓰레드의 실행을 멈추고 GC를 수행할 때까지 기다리는 상태
    > 
    
    GC가 실행될 때는GC를 수행하는 쓰레드를 제외한 모든 애플리케이션 쓰레드의 작업이 일시 중단된다.
    
    이러한 현상을 `Stop-The-World (STW)` 라고 부른다.
    
    **❗ STW의 영향**
    
    - 모든 쓰레드가 멈추기 때문에 애플리케이션 전체가 일시적으로 멈춘 것처럼 보임
    - STW가 발생하는 동안 응답 속도 지연이나 장시간 멈춤이 사용자에게 체감될 수 있음
    - 특히 Major GC가 발생할 때는 STW 시간이 길어질 수 있어 성능 저하로 이어질 수 있음
    
- 🍪 **Mark and Sweep**
    
    ### 🍪 Mark and Sweep
    
    > 애플리케이션의 사용되지 않는 객체를 정리하는 단순한 GC 방식
    > 
    
    `Mark and Sweep`는 가비지 컬렉션(GC)에서 가장 기본적이고 전통적인 알고리즘 중 하나이다. 
    
    이 방식은 크게 두 단계로 이루어집니다:
    
    1️⃣ **Mark (표시 단계) :** 
    
    - 모든 객체를 탐색하면서 애플리케이션에서 사용 중인 객체를 마크한다.
    - 애플리케이션에서 접근 가능한 객체만 마크로 표시
    
    2️⃣ **Sweep (정리 단계) :** 
    
    - 마크되지 않은 객체는 사용되지 않는 객체로 간주되어 메모리에서 제거됨
    - 이렇게 가비지가 제거되면 메모리 공간이 비게 됩니다.
    

---

 *reference:*

https://d2.naver.com/helloworld/1329<br/>
https://asfirstalways.tistory.com/159<br/>
https://asfirstalways.tistory.com/158<br/>
https://lob-dev.tistory.com/entry/Presentation-JVM-GC-%EA%B8%B0%EB%B3%B8-%EA%B0%9C%EB%85%90%EA%B3%BC-%EA%B8%B0%EB%B3%B8-GC-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98<br/>
https://mangkyu.tistory.com/118<br/>
https://mangkyu.tistory.com/119
