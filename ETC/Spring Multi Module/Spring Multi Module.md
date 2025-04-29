# 🔘 Spring Multimodule
---

### 🪴 root 프로젝트 생성

- 🪴 root 프로젝트 생성
    
    
    multimodule에서 root가 되는 프로젝트는 보통 dependency가 따로 필요 없다. 
    
    필요하게 된다면 그때 알아서 코드로 추가하면 됨
  
    ![image](https://github.com/user-attachments/assets/d0c04ba2-906d-4ba4-9cf9-9b22d8526c42)

    
    > 필요 없는 파일 삭제한다.
    > 
    
    > /src 는 필요 없으니 삭제, [help.md](http://help.md) 이런거도 걍 삭제
    > 
    
    ---
    
    ### 🥘module-api, module-common
    
    *module-api*

    ![image](https://github.com/user-attachments/assets/6a66cbca-a92d-44c4-bddb-34d5ba7bf0e3)
    
    > root 프로젝트에 우클릭하고 Module 생성하면 됨
    > 
    
    > lombok, spring web 같은 의존성 추가해서 `module-api` 모듈 생성
    > 
    
    동일한 방식으로 `module-common` 생성 (lomok만 추가)
  
    ![image](https://github.com/user-attachments/assets/a1fb7bcf-94bd-4272-837e-a5d13784bfc5)

    
    > `module-common` 같은 경우는 서버를 여기서 띄우는게 아니기 때문에 Application.Main이 필요가 없음 → 삭제
    > 
    
    ---
    
    ## 의존성
    
    root프로젝트한테 모듈에 대해 알려줘야함
    
    *root/settings.gradle*
    
    ```
    rootProject.name = 'multimodule'
    
    include 'module-api'
    include 'module-common'
    ```
    
    > 모듈 이름을 적어서 include
    > 

---

### 🍥 모듈 간 의존성

- 🍥 모듈 간 의존성
    
    api모듈과 common 모듈 사이의 의존성 설정을 해주자.
    
    api가 common을 참조하기 때문에 module-api의 build.gradle을 수정해줘야 한다.
    
    *module-api/build.gradle*
    
    ```java
    dependencies {
    	//	...
    	**implementation project(':module-common')**
    }
    ```
    
    > root의 settings.gradle에 적어줬던 모듈 이름과 텍스트가 정확히 일치해야 함
    > 
    
    여기까지 하고 gradle reload하면 모듈 찾을 수 없다고 에러뜬다.
    
    이게 왜냐면 root에서 settings.gradle을 관리해줄건데, 서버를 띄웠을 때 모듈 내부에 settings.gradle의 우선순위를 더 높게 보기 때문에 이걸 삭제해줘야한다.
    
    그리고 이거 추가
    
    *module-api/build.gradle*
    
    ```java
    tasks.register("prepareKotlinBuildScriptModel") {}
    ```
    

---

### ⛱️ 모듈 간 참조

- ⛱️ 모듈 간 참조
    
    > 이제 api 모듈에서 common 모듈의 코드를 참조해서 쓸 수 있을까?
    > 
    1. java class
    2. spring bean
    
    코드로 예시를 작성하여 학습
    
    ### 🌲 java class
    
    *module-common/src/…/response/response.class*
    
    ```java
    @Getter
    @AllArgsConstructor
    public enum Response {
        SUCCESS("000", "success"),
        FAIL("999", "fail");
    
        private String code;
        private String message;
    }
    ```
    
    > `module-common` 의 response enum
    > 
    
    *module-api/src/…/DemoController.class*
    
    ```java
    @RestController
    @RequiredArgsConstructor
    public class DemoController {
    
        private final DemoService demoService;
    
        @GetMapping("/save")
        public String save(){
            return demoService.save();
        }
    
        @GetMapping("/find")
        public String find(){
            return demoService.find();
        }
    }
    ```
    
    *module-api/src/…/DemoService.class*
    
    ```java
    @Service
    public class DemoService {
    
        public String save() {
            System.out.println(Response.SUCCESS.getCode());
            return "save";
        }
    
        public String find() {
            return "find";
        }
    }
    ```
    
    `curl [localhost:8080/save](http://localhost:8080/save)` 를 날려보면, s-out으로 common 모듈의 response 클래스 값을 참조해서 잘 출력이 된다.
    
    ### 🌳 spring bean
    
    *module-common/…/ExcamService.class*
    
    ```java
    @Service
    public class ExamService {
    
        public String examService(){
            return "examService";
        }
    }
    ```
    
    *module-api/src/…/DemoController.class*
    
    ```java
    @RestController
    @RequiredArgsConstructor
    public class DemoController {
    
    		// ...
        private final ExamService examService;
    
        // ...
        
        @GetMapping("/example")
        public String example(){
            return examService.print();
        }
    }
    ```
    
    > 이렇게 그냥 주입하면 컴포넌트 스캔 범위에 없어서 Bean으로 등록이 안된다.
    > 
    
    그럼 `ModuleApiApplication` 을 `module-common` 쪽에서도 스캔할 수 있게 path를 맞춰주면 된다.
    
    지금 `ModuleApiApplication` 은 package path 다음과 같다
    
    ```java
    package dev.backend.moduleapi;
    ```
    
    package 기준으로 컴포넌트 스캔이 일어나니까 범위를 맞춰주려면
    
    ```java
    package dev.backend; 
    ```
    
    > 해당 path로 `ModuleApiApplication.class` 파일 자체를 옮겨주면 된다.
    > 
    
    ### 📌 요약
    
     순수 자바 클래스는 그냥 된다.
    
     스프링 빈은 컴포넌트 스캔에 유의해야 한다.
    
    ### 🤔 근데 이렇게 application 파일을 옮겨주는게 맞나?
    
    다른 방법도 있다.
    
    - 다음 토글 확인
    

---

### 🌀 모듈 간 컴포넌트 스캔 방법

- 🌀모듈 간 컴포넌트 스캔 방법
    
    `@SpringBootApplication` 의 `String[] scanBasePackages() default {};` 옵션을 사용해준다.
    
    사용법
    
    ```java
    @SpringBootApplication (
    		scanBasePackages = {"dev.backend.moduleapi", "dev.backend.modulecommon"}
    )
    public class ModuleApiApplication {
    
    	public static void main(String[] args) {
    		SpringApplication.run(ModuleApiApplication.class, args);
    	}
    
    }
    ```
    
    > 컴포넌트 스캔이 필요한 경로만 골라서 저렇게 넣어주면 됨
    > 

---

### 📡 멀티 모듈에서 DB 연동

- 📡 멀티 모듈에서 DB 연동
    
    프로젝트에 배치모듈, API모듈, Common 모듈 이렇게 3개 있다고 할 때, DB를 보통 common 한 곳에서 관리하는게 리스크도 적고 운영 측면에서도 편리하다.
    
    module-common 에서 Entity랑 Repository 생성/관리해주고, module-api에서 사용할 것이기 때문에 module-api 쪽에 application.yml을 작성해주고 jpa의존성도 추가해준다.
    
    ### Entity, Repository를 컴포넌트 스캔 범위에 넣어주기
    
    *module-api*
    
    ```java
    @SpringBootApplication (
    		scanBasePackages = {"dev.backend.moduleapi", "dev.backend.modulecommon"}
    )
    @EntityScan("dev.backend.modulecommon.domain")
    @EnableJpaRepositories(basePackages = "dev.backend.modulecommon.domain")
    public class ModuleApiApplication {
    
    	public static void main(String[] args) {
    		SpringApplication.run(ModuleApiApplication.class, args);
    	}
    
    }
    ```
    
    > 해당 어노테이션으로 역시 스캔 범위를 지정해줘야 entity랑 repository가 빈으로 등록된다.
    > 

---

### 🍚 멀티 모듈 gradle 빌드

- 🍚 멀티 모듈 gradle 빌드
    
    멀티모듈에서는 좀 해줘야하는게 있다.
    
    ### 세팅
    
    common 쪽에서 다음을 추가
    
    ```java
    tasks.bootJar { enabled = false }
    tasks.jar { enabled = true }
    ```
    
    > tasks.bootJar { enabled = false }
    > 
    
    이거 기본값이 true임. 이게 동작하면 **.jar 파일이 생성되는데, 어플리케이션 구동에 필요한 의존성이나 클래스, 리소스 이런것들이 포함되기 때문에 .jar로 실행이 가능하게 된다.
    
    (common 모듈은 application main 클래스가 없기 때문에 에러발생함)
    
    근데 어쨋든 common 모듈은 다른 모듈에서 참조하게 하는 목적으로 만드는 모듈이기 때문에 실행 가능한 .jar 파일을 만들 필요가 없다.
    
    > tasks.jar { enabled = true }
    > 
    
    이거 하면 xxx-plane.jar 가 만들어지는데, plane이 붙은거는 그냥 xxx.jar랑 다르게 dependency가 붙어있지 않다. 즉, 클래스와 리소스만을 포함하고 있어서 서버를 실행시키지 않게 할 수 있다.
    
    요약 : plane.jar는 필요하지만 .jar는 필요가 없어서 한다는거임
    
    ---
    
    ### 빌드
    
    ```java
    ./gradlew clean :module-api:buildNeeded --stacktrace --info --refresh-dependencies -x test
    ```
    
    > `clean` : 기존 빌드되어 있는 폴더는 날려버리고
    > 
    
    > `:module-api:buildNeeded` : api 모듈을 빌드할 것이다.
    > 
    
    > `—stacktrace` : 로그 보여줘라
    > 
    
    > `—info` : info 레벨 로그 출력해줘라. (debug → info → warn → error)
    > 
    
    root에서 명령어 찍어서 빌드 하면, 각 모듈마다 /build가 생기고 아까 설정했던대로 common 모듈은 plane.jar만 생겨있다.
    
    그래서 실행을 하고자 할때는 api모듈은 java -jar 파일명.jar 하면 됨
    

---

### 🥥 profile 설정

- 🥥 profile 설정
    
    ### 1. 환경 별 profile 값 사용
    
    application.yml 수정하기
    
    ### 2. IntelliJ 실행 시 Profile 적용
    
    application-local.yml로 이름 변경 후, run configuration에서 수정해줘야 intellij에서 default가 아니라 내가 만들어준 yml을 바라보고 실행할 수 있음

  ![image](https://github.com/user-attachments/assets/7d665171-2169-4c46-91e2-d774d1e196ad)
    
    ### 3. jar 실행 시 JVM 옵션으로 Profile 적용
    
    ```java
    java -jar -Dspring.profiles.active={profilename} {빌드파일이름}.jar
    ```
    
    > 이렇게 하면 됨
    >
