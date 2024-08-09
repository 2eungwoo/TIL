자바에서는 runtime에 발생할 수 있는 오류를 오류`(error)`와 `예외(exception)`로 구분하였다 

오류`(error)`: 시스템이 종료되어야할 수준과 같이 수습할 수 없는 심각한 문제. 개발자가 미리 예측하고 방지하기 힘들다.

ex) StackOverflowError, OutOfMemoryError

`예외(exception)`: 개발자가 구현한 로직에서 발생한 실수나 사용자의 영향에 의해 발생하는 문제. 개발자가 미리 예측하여 방지할 수 있으며 여러 상황에 맞는 예외 처리(Exception Handle)가 가능하다.

ex) NullPointerException, IllegalArgumentException

- `예외(exception)`는 `오류(error)`와 다르게 개발자가 임의로 `throw`할 수 있다.

<img width="187" alt="image" src="https://github.com/2eungwoo/TIL/assets/89715722/1edd3876-8cd6-4849-b6ab-8a1d465b6ee1"> <img width="187" alt="image" src="https://github.com/2eungwoo/TIL/assets/89715722/c3b12195-a48c-481f-a63d-1ed45d873539">



오류(error)와 예외(exception) 모두 Object 클래스를 상속 받는 `Throwable` 클래스를 상속 받는다.

Throwable 객체는 오류나 예외에 대한 메시지를 담고, 예외가 연결될 때 연결된 예외의 정보를 기록한다. 이를 위해 `Throwable` 클래스에는 `getMessage()`와 `printStackTrace()`가 구현되어있다.

`예외(exception)`는 `Checked Exception`과 `Unchecked Exception`으로 나뉜다.

Uncheckd Exception 클래스는 RuntimeException을 상속 받고, Checked Exception 클래스는 상속 받지 않는다.

**쉽게 말해서 Unchecked Exception은 Runtime 예외 클래스를, Checked Exception은 Compile 예외 클래스를 가리킨다.**

Runtime Exception은 명시적으로 예외 처리를 하지 않아도 되기 때문에 특별하게 취급된다.

### Spring 에서 예외처리 기본 처리
|  | Unchecked Exception | Checked Exception |
| --- | --- | --- |
| 확인 시점 | Runtime 시점 | Compile 시점 |
| 처리 여부 | 명시적으로 하지 않아도 됨 | 반드시 예외 처리 필요 |
| 트랜잭션 처리 | rollback 진행 | rollback 안됨 |
| 종류 | NullPointerException, ArithmeticException 등 | IOException, SQLException 등 |

Checked Exception은 확인 시점이 compile 시점이므로, 별도의 예외 처리를 하지 않으면 컴파일 자체가 되지 않는다.

따라서 Checked Exception 발생 가능성이 있다면 반드시 `try ~ catch`로 해당 코드를 감싸거나 `throws`로 던져줘야 한다.

Unchecked Exception은 RuntimeException을 상속받는다.

컴파일 시점에 예외를 catch 하는지 확인하지 않으므로, 컴파일 시점에 예외가 발생하는지 여부를 판단할 수 없다.

예외를 처리할 필요는 없지만 처리해도 된다.

즉, 명시적으로 예외처리를 하지 않아도 된다.

---

### Checked Exception의 Rollback

어차피 try - catch로 잡아줘야 하니, rollback은 안해줄거고 너가 알아서 하라 는 의미일까? rollback은 진행해주지 않는다.

기본적으로 Checked Exception은 복구 가능하다는 매커니즘을 가지고 있다고 한다.

예를 들어 특정 이미지 파일을 찾아서 전송해주는 함수에서, 이미지를 찾지 못했을 경우 기본으로 설정된 default 이미지를 전송하는 것처럼, 복구 전략을 가질 수 있다.

```java
public void sendImage(String fileName){
	File file;
	try{
		file = FileService.find(fileName);
	} catch(FileNotFoundException e){
		file = FileNotFoundException.find("default.jpg");
	}
	
	send(file);
}
```

하지만 이런 식의 예외는 복구하는 것이 아니라 코드의 흐름으로 제어해야 한다.

```java
public void sendFile(String fileName){
	if(FileService.exist(fileName)){
		File file = FileService.find(fileName);
		send(file);
	}else{
		send(FileService.find("default.jpg"));
	}
}
```

### 그러나

일반적으로 Checked Exception 예외가 발생했을 경우 복구 전략을 통해 복구할 수 있는 경우는 많지 않다.

예를 들어, 회원가입을 할 때

유니크해야 하는 이메일 값이 중복이 되어 SQLException이 발생하는 경우 분기를 통해 처리해줄 수 있긴 하지만,

사실 RuntimeException을 발생시키고 입력을 다시 하도록 유도해주는 것이 낫다.

게다가, 어떤 예외가 발생했는지 명확하게 전달해주기 위해 위와 같은 경우에는 DuplicateEmailException과 같은 예외를 만들어 Unchecked Exception을 발생시키는 것이 바람직하다.
따라서 예외 복구 전략이 명확하고 복구가 가능하다면 해당 예외를 try-catch로 잡아주는 것이 좋다.

하지만 이런 경우는 흔하지 않기 때문에, Checked Exception이 발생하면 더 명확하고 구체적인 예외를 전달해주기 위해 Unchecked Exception으로 발생시키는 것이 더 효과적이다.

또한 무분별하게 상위 메서드로 throw 해주는 행위는 지양해야 한다. 상위 메서드들의 책임이 그만큼 증가하며 Checked Exception은 기본적으로 rollback을 진행하지 않은 트랜잭션 속성이 있으므로 잘 알아두어 실수를 방지해야 한다.

### 결론
Checked Excepion은 구체적이고 명확하게 잡아주기 위해서 Unchecked Exception으로 발생시켜주자
예외를 처리할 때 로그를 활용하는 것 이외에 checked exception을 unchecked exception으로 변경해서 던져야 하는 경우가 있었다. 이를 자바에서는 RuntimeException 클래스를 상속받아서 할 수 있다.
[Exception Handling](https://github.com/2eungwoo/TIL/blob/main/Spring/Common-Response/Exception%20Response.md)

참고 : https://cheese10yun.github.io/checked-exception/</br>
https://github.com/cheese10yun/spring-guide/blob/master/docs/exception-guide.md</br>
https://github.com/cheese10yun/blog-sample/blob/master/exception/README.md</br>
https://junghyungil.tistory.com/52
