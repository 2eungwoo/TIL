# Common API Response, Error Response practice
*repository* : [Common API Response, Error Response practice](https://github.com/2eungwoo/API-Response-Error-Response-practice)

## INTRODUCE

api 요청 응답 시 일관성 있는 `response body`를 얻기 위한 작업

에러 발생 시 의도한 에러와 그렇지 않은 에러 모두 일관성 있는 응답

## OUTLINE

- `global exception handling`
- `api result respons`

## PACKAGE STRUCTURE

<img width="435" alt="image" src="https://github.com/breadman98/API-Response-Error-Response-practice/assets/89715722/6eeb1998-1139-4630-bd99-8c88c0a4ae23">


## GLOBAL EXCEPTION HANDLING

- `key` : 전체 로직과 `exception handling` 책임을 분리

=> `@RestControllerAdvice` , `@ExceptionHandler`

- 목표 : 다음과 같은 일관성 있는 에러 응답
    
    ```json
    {
      "status": 400,
      "code": "C002",
      "message": "invalid input type",
      "errors": [
          {
              "field": "description",
              "value": "",
              "reason": "비어 있을 수 없습니다"
          }
      ]
    }
    ```
    
    ```json
    {
      "status": 404,
      "code": "M001",
      "message": "member not exist",
      "errors": []
    }
    ```
    

### ErrorResponse.class

```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class ErrorResponse {

    private int status;
    private String code;
    private String message;
    private List<FieldError> errors;

    private ErrorResponse(final ErrorCode code, final List<FieldError> errors) {
        this.status = code.getStatus();
        this.code = code.getCode();
        this.message = code.getMessage();
        this.errors = errors;
    }

    private ErrorResponse(final ErrorCode code) {
        this.status = code.getStatus();
        this.code = code.getCode();
        this.message = code.getMessage();
        this.errors = new ArrayList<>();
    }

    public static ErrorResponse of(final ErrorCode code, final BindingResult bindingResult) {
        return new ErrorResponse(code, FieldError.of(bindingResult));
    }

    public static ErrorResponse of(final ErrorCode code, final Set<ConstraintViolation<?>> constraintViolations) {
        return new ErrorResponse(code, FieldError.of(constraintViolations));
    }

    public static ErrorResponse of(final ErrorCode code, final String missingParameterName) {
        return new ErrorResponse(code, FieldError.of(missingParameterName, "", "parameter must required"));
    }

    public static ErrorResponse of(final ErrorCode code) {
        return new ErrorResponse(code);
    }

    public static ErrorResponse of(final ErrorCode code, final List<FieldError> errors) {
        return new ErrorResponse(code, errors);
    }

    public static ErrorResponse of(MethodArgumentTypeMismatchException e) {
        final String value = e.getValue() == null ? "" : e.getValue().toString();
        final List<FieldError> errors = FieldError.of(e.getName(), value, e.getErrorCode());
        return new ErrorResponse(ErrorCode.INVALID_TYPE_VALUE, errors);
    }

    @Getter
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    public static class FieldError {
        private String field;
        private String value;
        private String reason;

        public FieldError(String field, String value, String reason) {
            this.field = field;
            this.value = value;
            this.reason = reason;
        }

        public static List<FieldError> of(final String field, final String value, final String reason) {
            List<FieldError> fieldErrors = new ArrayList<>();
            fieldErrors.add(new FieldError(field, value, reason));
            return fieldErrors;
        }

        public static List<FieldError> of(final BindingResult bindingResult) {
            final List<org.springframework.validation.FieldError> fieldErrors = bindingResult.getFieldErrors();
            return fieldErrors.stream()
                    .map(error -> new FieldError(
                            error.getField(),
                            error.getRejectedValue() == null ? "" : error.getRejectedValue().toString(),
                            error.getDefaultMessage()
                    ))
                    .collect(Collectors.toList());
        }

        public static List<FieldError> of(final Set<ConstraintViolation<?>> constraintViolations) {
            List<ConstraintViolation<?>> lists = new ArrayList<>(constraintViolations);
            return lists.stream()
                    .map(error -> new FieldError(
                            error.getPropertyPath().toString(),
                            "",
                            error.getMessageTemplate()
                    ))
                    .collect(Collectors.toList());
        }
    }

}
```

`ErrorResponse.class`에서 원하는 json 응답 형태를 필드로 갖는다.

기본적인 생성자의 사용 대신, `정적 팩토리 메서드`를 사용하여 파라미터에 따라 유연한 사용이 가능하다.

*reference : [Effective Java 아이템 1. 생성자 대신 정적 팩토리 메서드를 고려하라](https://github.com/alanhakhyeonsong/LetsReadBooks/blob/master/Effective%20Java%203E/contents/chapter02/item01.md)*

널리 알려진 규약에 따라 `of` 라는 메소드 명을 갖게 한다.

이 메소드들은 다음과 같이 여러 에러 상황에 대응할 수 있다.

- javax.validation의 `@Valid` 나 `@Validated`를 통해 유효성 검증에서 걸린 `Binding Error` 처리
- 데이터 유효성 검사 실패 시 발생하는 예외에서 실패 정보를 담고 있는  `ConstraintViolation` 처리
- 프로그래머가 일반적으로 사용하기 위해 만드는 `CustomError` 에 대한 처리

### ErrorCode.enum

```java
@Getter
@AllArgsConstructor
public enum ErrorCode {

    // Common
    INTERNAL_SERVER_ERROR(500, "C001", "internal server error"),
    INVALID_INPUT_VALUE(400, "C002", "invalid input type"),
    METHOD_NOT_ALLOWED(405, "C003", "method not allowed"),
    INVALID_TYPE_VALUE(400, "C004", "invalid type value"),
    BAD_CREDENTIALS(400, "C005", "bad credentials"),

    // Member
    MEMBER_NOT_EXIST(404, "M001", "member not exist"),
    USER_EMAIL_ALREADY_EXISTS(400, "M002", "user email already exists"),
    NO_AUTHORITY(403, "M003", "no authority"),
    NEED_LOGIN(401, "M004", "need login"),
    AUTHENTICATION_NOT_FOUND(401, "M005", "no authenticated data in security context "),
    MEMBER_ALREADY_LOGOUT(400, "M006", "member already logout"),

    // Auth
    REFRESH_TOKEN_INVALID(400, "A001", "refresh token invalid"),

		// etc
		// ...
		
    private final int status;
    private final String code;
    private final String message;
}
```

`Enum` 타입으로 한 곳에서 에러 코드를 관리한다.

원하는 에러를 터뜨려줄 때, statu및 message의 중복을 해결해주기 위한 방법이다.

### GlobalExceptionHandler.class

```java
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {

		// spring security
    @ExceptionHandler
    protected ResponseEntity<ErrorResponse> handleBadCredentialException(BadCredentialsException e) {
        final ErrorResponse response = ErrorResponse.of(BAD_CREDENTIALS);
        return new ResponseEntity<>(response, HttpStatus.BAD_REQUEST);
    }

    @ExceptionHandler
    protected ResponseEntity<ErrorResponse> handleMethodArgumentTypeMismatchException(MethodArgumentTypeMismatchException e) {
        final ErrorResponse response = ErrorResponse.of(e);
        return new ResponseEntity<>(response, HttpStatus.BAD_REQUEST);
    }

    @ExceptionHandler
    protected ResponseEntity<ErrorResponse> handleHttpRequestMethodNotSupportedException(HttpRequestMethodNotSupportedException e) {
        final ErrorResponse response = ErrorResponse.of(METHOD_NOT_ALLOWED);
        return new ResponseEntity<>(response, HttpStatus.METHOD_NOT_ALLOWED);
    }

    // @Valid, @Validated 에서 binding error 발생 시 (@RequestBody)
    @ExceptionHandler
    protected ResponseEntity<ErrorResponse> handleBindException(BindException e) {
        final ErrorResponse response = ErrorResponse.of(INVALID_INPUT_VALUE, e.getBindingResult());
        System.out.println("================== 메소드 콜");
        return new ResponseEntity<>(response, HttpStatus.BAD_REQUEST);
    }

    // 비즈니스 요구사항에 따른 Exception
    @ExceptionHandler
    protected ResponseEntity<ErrorResponse> handleBusinessException(CustomException e) {
        final ErrorCode errorCode = e.getErrorCode();
        final ErrorResponse response = ErrorResponse.of(errorCode, e.getErrors());
        return new ResponseEntity<>(response, HttpStatus.valueOf(errorCode.getStatus()));
    }

    // 그 밖에 발생하는 모든 예외처리
    @ExceptionHandler(Exception.class)
    protected ResponseEntity<ErrorResponse> handleException(Exception e) {
        log.error("Exception: ", e);
        final ErrorResponse response = ErrorResponse.of(INTERNAL_SERVER_ERROR);
        return new ResponseEntity<>(response, HttpStatus.INTERNAL_SERVER_ERROR);
    }

}
```

Spring에서 제공하는 `@RestControllerAdvice`, `@ExceptionHandler`를 사용하여 전역으로 에러처리를 하도록 해준다. 

@ControllerAdvice는 `@ExceptionHandler`, `@ModelAttribute`, `@InitBinder`가 적용된 메소드들을 AOP를 적용해 컨트롤러 단에 적용하기 위해 고안된 어노테이션이다.

### CustomException.class

```java

@Getter
public class CustomException extends RuntimeException {

    private final ErrorCode errorCode;
    private List<ErrorResponse.FieldError> errors = new ArrayList<>();

    public CustomException(ErrorCode errorCode) {
        super(errorCode.getMessage());
        this.errorCode = errorCode;
    }

    public CustomException(String message, ErrorCode errorCode) {
        super(message);
        this.errorCode = errorCode;
    }

    public CustomException(ErrorCode errorCode, List<ErrorResponse.FieldError> errors) {
        super(errorCode.getMessage());
        this.errorCode = errorCode;
        this.errors = errors;
    }
}
```

`RuntimeException`을 상속받는 `CustomException`을 정의해두어 다양한 Exception들에 대해 구체적으로 정의하고 사용할 수 있다.

<img width="289" alt="image" src="https://github.com/breadman98/API-Response-Error-Response-practice/assets/89715722/551781d4-e3e9-472e-9964-59c247c04395">


`CustomException`을 상속받는 구체적인 Exception들은 도메인 내에서 예외가 발생할만한 부분마다 디테일하게 처리 가능하나, 케이스마다 직접 정의해주어야 하므로 클래스가 많아진다는 단점이 있다.

다음과 같이 구체적인 Exception들을 정의하여 사용하면 된다.
<img width="862" alt="image" src="https://github.com/breadman98/API-Response-Error-Response-practice/assets/89715722/da8d1267-5be0-4c52-a60b-ca56cbb5f698">

### MemberNotExistExecption.class

```java
public class MemberNotExistException extends CustomException{
    public MemberNotExistException(){
        super(ErrorCode.MEMBER_NOT_EXIST);
    }
}

```

### NeedLoginException.class

```java
public class NeedLoginException extends CustomException{
    public NeedLoginException() {
        super(ErrorCode.NEED_LOGIN);
    }
}
```

### ETC.class

```java
public class 기타등등Exception extends CustomException{
    public 기타등등Exception() {
        super("custom msg",ErrorCode.CUSTOM_ERR);
    }
}
```

## Common API Response

- `key` : 모든 api 응답에 대해 응답 형식을 통일
- 목표 : 다음과 같은 일관성 있는 api 결과 응답
    
    ```json
    {
        "status": 200,
        "code": "M001",
        "message": "회원가입 되었습니다.",
        "data": {
            "email": "test@gmail.com",
            "nickname": "test"
        }
    }
    ```
    
    ErrorResponse와 마찬가지로, ResultCode를 따로 정의한 후 정적 스태틱 메소드로 생성하여 사용한다.
    
    공통 응답의 경우 에러와는 다르게, 케이스가 정해져있지 않기 때문에 간결한 모습을 가질 수 있다.
    
    ### ResponseCode.Enum
    
    ```java
    @Getter
    @AllArgsConstructor
    public enum ResponseCode {
    
        // Member
        REGISTER_SUCCESS(200, "M001", "회원가입 되었습니다."),
        LOGIN_SUCCESS(200, "M002", "로그인 되었습니다."),
        REISSUE_SUCCESS(200, "M003", "재발급 되었습니다."),
        LOGOUT_SUCCESS(200, "M004", "로그아웃 되었습니다."),
        GET_MY_INFO_SUCCESS(200, "M005", "내 정보 조회 완료");
        
        // etc
        // ...
    
        private final int status;
        private final String code;
        private final String message;
    }
    ```
    
    ## ResultResponse.class
    
    ```java
    @Getter
    public class ResultResponse {
    
        private final int status;
        private final String code;
        private final String message;
        private final Object data;
    
        public static ResultResponse of(ResponseCode responseCode, Object data) {
            return new ResultResponse(responseCode, data);
        }
    
        public ResultResponse(ResponseCode resultCode, Object data) {
            this.status = resultCode.getStatus();
            this.code = resultCode.getCode();
            this.message = resultCode.getMessage();
            this.data = data;
        }
    }
    ```
    
    이제 controller layer에서 직접 정의한 `ResultResponse` 타입으로 result를 생성해준 뒤, ResponseEntity에 담아 리턴하면 된다.
    
    ## MemberController.Class
    
    ```java
    @RestController
    @RequiredArgsConstructor
    public class MemberController {
    
        private final MemberService memberService;
    
        @PostMapping("/v2/member")
        public ResponseEntity<ResultResponse> signup(@Valid @RequestBody MemberDto.Request memberRequestDto) {
            MemberDto.Response memberResponseDto = memberService.saveMember(memberRequestDto);
            ResultResponse response = ResultResponse.of(ResponseCode.REGISTER_SUCCESS, memberResponseDto);
            return new ResponseEntity<>(response, HttpStatus.valueOf(response.getStatus()));
        }
    
        @GetMapping("/v2/member/{memberId}")
        public ResponseEntity<ResultResponse> getMyInfo(@PathVariable Long memberId) {
            MemberDto.Response memberResponseDto = memberService.getMyInfo(memberId);
            ResultResponse response = ResultResponse.of(ResponseCode.GET_MY_INFO_SUCCESS, memberResponseDto);
            return new ResponseEntity<>(response, HttpStatus.valueOf(response.getStatus()));
        }
    }
    
    ```
    
    ## MY LEGACY
    
    이전에 사용하던 방법은 `ResponseEntity` 에 응답 결과를 담아 반환하는 것이 아닌,
    
    직접 ResponseEntity 자체를 커스텀 정의하여, Builder를 사용해 생성 후 리턴했다.
    
    ## Controller : MyLegacy
    
    ```java
    @RestController
    @RequiredArgsConstructor
    public class MemberController {
    
        private final MemberService memberService;
    
        // my legacy
        @PostMapping("/v1/member")
        public ResultResponseMyLegacy<?> saveMember(@Valid @RequestBody MemberDto.Request memberRequestDto){
            MemberDto.Response savedMemberDto = memberService.saveMember(memberRequestDto);
            return ResultResponseMyLegacy.builder()
                    .status(ResponseCodeMyLegacy.REGISTER_SUCCESS.getStatus())
                    .code(ResponseCodeMyLegacy.REGISTER_SUCCESS.getCode())
                    .message(ResponseCodeMyLegacy.REGISTER_SUCCESS.getMessage())
                    .data(savedMemberDto)
                    .build();
        }
    }
    ```
    
    각 컨트롤러마다, 빌더 패턴을 사용한 리턴 방식이, 코드를 봤을 때 예측 가능하며 가독성이 더 있다고 생각한 이유였다.
    
    ## ResultResponseMyLegacy.Class
    
    ```java
    @Builder
    @AllArgsConstructor
    @Getter
    // ResultResponse<T> : don't use raw type
    public class ResultResponseMyLegacy<T> {
    
        private final HttpStatus status;
        private final String code;
        private final String message;
        private final T data;
    
    }
    ```
    
    직접 정의한 커스텀 ResponseEntity는 위와 같이 생겼으며, 응답으로 어떠한 경우도 처리 가능하게끔 제네릭 타입으로 정의한다.
    
    이 경우, raw type을 사용하지 말 것을 권장하는 notice 문구가 나오기도 하며 실제로 권장되는 사항이 아니다.
    
    *reference* : [Effective Java Item 26 `Raw type은 사용하지말라`](https://github.com/alanhakhyeonsong/LetsReadBooks/blob/master/Effective%20Java%203E/contents/chapter05/item26.md)<br/>
    *reference* : [Spring Guide - Exception Guide](https://github.com/cheese10yun/spring-guide/blob/master/docs/exception-guide.md)
