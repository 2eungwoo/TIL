# Common Response : Result Response
*repository* : [Common API Response, Error Response practice](https://github.com/2eungwoo/API-Response-Error-Response-practice)

## Result Response

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
    
    ResultCode를 따로 정의한 후 정적 스태틱 메소드로 생성하여 사용한다.
    
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
  
