### record?
java 16에서 공개된 새로운 클래스 타입으로 다음과 같은 특징을 갖는다.
- 불변 객체(final 필드로 생성자 초기화 후 변경 불가. 즉, set 불가)
- 생성자, getter, hashCode, equals, toString 자동 제공
- 상속 불가

### 클래스 dto
```java
@Getter
//@RequiredArgsConstructor
public class UserResponseDTO {

    private final String username;
    private final String age;
    private final String gender;
    private final String email;

    public UserResponseDTO(String username, String age, String gender, String email) {

        this.username = username;
        this.age = age;
        this.gender = gender;
        this.email = email;
    }
}
```
### 레코드 dto
```java
public record UserResponseDTO(
        String username, 
        String age;
        String gender;
        String email;
        ) {}
```
> setter가 없는 dto, 즉 response 전용 dto에서 사용

### 성능 차이?
#### 컴파일 단계
- “record”와 “클래스 + @Lombok” 모두 컴파일 단계에서 “생성자, getter, hashCode, equals, toString”들이 생성된다.
- 이때 record는 컴파일러가 가장 최적화된 방식으로 해당하는 메소드를 생성한다고 한다.
- 아래 StackOverFlow 게시글을 통해 Byte 코드를 확인하면 record Byte 코드의 규모가 훨씬 작은 것을 볼 수 있다.
- https://stackoverflow.com/questions/61208493/do-java-records-actually-save-memory-over-a-similar-class-declaration-or-are-the/61209698#61209698
- 따라서 컴파일시점에서는 훨씬 좋은 성능을 가진다.

#### 런타임 단계
- 런타임에서는 대부분의 자료들이 큰 차이가 없다고 한다. 
- JVM이 record에 대해서 어느 정도 최적화된 동작을 할 수 있지만 아주 미미한 차이라고 한다.

#### 기타 : 데이터 저장
- 데이터 저장 필드 같은 경우에는 동일. record라고 해서 더 작은 저장 필드를 가지지는 않는다.


  
