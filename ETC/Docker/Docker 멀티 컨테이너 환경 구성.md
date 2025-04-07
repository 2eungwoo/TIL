## 🌀 프로젝트 환경

- Spring Boot 3.4.4
- MySQL 8.0.33
- Redis 7.2.4
- Docker 27.5.1

### source
https://github.com/2eungwoo/docker-prac

## 🐳 Docker I : Single-Cotainer

프로젝트 빌드 파일을 dockerfile로 이미지 생성, 이 후 docker로 빌드/실행

*spring-boot project build*

```jsx
$ ./gradlew clean build --exclude-task test
```

- 💡 jar 파일 이름 변경하는 방법
    
    *build.gradle*
    
    ```jsx
    tasks.named('bootJar') {
        archiveFileName = 'myapp.jar'
    }
    
    //
    
    bootJar {
        archiveFileName = 'myapp.jar'
    }
    ```
    
    > jar 파일 이름 변경
    > 
    > - 차이
    >     
    >     
    >     |  | bootJar {} | tasks.named("bootJar") {} |
    >     | --- | --- | --- |
    >     | 구성 시점 | 즉시 구성 (eager) | 지연 구성 (lazy) |
    >     | Gradle 버전 호환 | Gradle 4.x 이상 | Gradle 5.1 이상 권장 |
    >     | 유연성 | 낮음 | 높음 (특히 조건부/멀티모듈에서 유리) |
    >     | 코드 가독성 | 간단하고 직관적 | 약간 복잡하지만 구조화에 유리 |

Dockerfile

```jsx
FROM openjdk:17
ARG JAR_FILE=build/libs/myapp.jar
COPY ${JAR_FILE} ./myapp.jar
ENV TZ=Asia/Seoul
ENTRYPOINT ["java","-jar","./myapp.jar"]
```

> project root에 생성
> 
> - 🌐 도커 파일 주요 명령어
>     
>     ```jsx
>     FROM : 새로운 이미지를 생성할 때 기반으로 사용할 이미지를 지정. {이미지_이름}:{태그}
>     ex) FROM openjdk:17
>     
>     ARG : 이미지 빌드 시점에서 사용할 변수 지정. 빌드 파일의 경로
>     ex) ARG JAR_FILE=build/libs/app.jar
>     
>     COPY : 호스트에 있는 파일이나 디렉토리를 도커 이미지의 파일 시스템으로 복사
>     ex) COPY ${JAR_FILE} ./app.jar
>     
>     ENV : 컨테이너에서 사용할 환경 변수 지정. 
>     ex) ENV TZ=Asia/Seoul // TimeZone 환경 변수
>     
>     ENTRYPOINT : 컨테이너가 실행되었을 때 항상 실행되어야 하는 커맨드 지정
>     ex) ENTRYPOINT ["java","-jar","./app.jar"]
>     ```
>     

*docker build, create image 생성*

```jsx
$ docker build -t {docker_hub_username}/{my_image_name}:{tag} {dockerfile's location}
```

> root 에서 cli 작업하고 있으면 dockerfile’s location 은 . (dot) 으로 쓰면 됨
> 

*run docker image*

```jsx
$ docker run -p {host_port}:{container_port} {image_name}:{tag}
```

*container shell* 

```jsx
$ docker exec -it {container_id} bash
```

> bash로 컨테이너 내부 진입
> 

### 🌈 Env Setting : Dev/Prod Environment

### Spring Profile

- 어플리케이션 설정을 특정 환경에서만 적용되게 하거나 환경 별(로컬,개발,배포 환경 등)로 다르게 적용할 때 사용
- Spring boot는 어플리케이션이 실행될 때 자동으로 application.yml (or properties) 찾음

*application.yml*

```yaml
spring:
  profiles:
    active: local # default active status -> local

---

spring:
  config:
    activate:
      on-profile: local # application-local.yml 과 동일

---

spring:
  config:
    activate:
      on-profile: prod # application-prod.yml 과 동일

```

> `YAML`은 하나의 profile에 `---`로 구분을 하면, 논리적으로 구분이 되어 파일을 나누어서 사용하는 효과를 볼 수 있음
> 

*application.yml*

```yaml
spring:
  profiles:
    active: local # default active status -> local
    group:
      local:
        - common # (local,common) 한 그룹으로 하여 함께 구동
      prod:
        - common # (prod,common) 한 그룹으로 하여 함께 구동

---

spring:
  config:
    activate:
      on-profile: common

---

spring:
  config:
    activate:
      on-profile: local

---

spring:
  config:
    activate:
      on-profile: prod

```

> 위와 같이 `group` 을 이용해서 여러 profile들을 한꺼번에 그룹화 하여 하나의 profile로 만들 수 있음
> 

> `spring.profiles.active=prod` 로 실행하면 prod와 common 두 개의 profile들을 한꺼번에 실행한다.
> 

*cli*

```yaml
$ java -Dspring.profiles.active={option} -jar myapp.jar

$ docker run -e "SPRING_PROFILES_ACTIVE={option}" -p 8080:8080 myapp:latest
```

*IntelliJ IDEA*

```yaml
-Dspring.profiles.active={option}
```

> `Edit Configuration` → `Modify Options` →`Add VM Options` 에 작성
> 

### .env file

```yaml
import: optional:file:.env[.properties]
```

> Spring Boot는 **기본적으로 .env 파일을 읽지 않음**
> 
> 
> 환경 변수는 운영체제나 Docker 등의 외부 환경에서 주입되는 걸로 간주하고
> 
> .env 파일은 **내부적으로 로드하지 않는다.**
> 
> ---
> 
> 💡 **따라서 .env 파일을 직접 Spring Config로 등록해줘야 함** 
> 
> - 예시
>     
>     ```yaml
>     spring:
>       config:
>         import: optional:file:.env[.properties]
>       profiles:
>         active: local # default active status -> local
>         group:
>           local:
>             - common
>           prod:
>             - common
>     
>     ---
>     
>     ...
>     ```
>     

## 🐳 Docker II : Multi-Cotainer

### ❄️ Docker Compose

- `Docker Compose`  : 멀티 컨테이너 도커 어플리케이션을 정의하고 실행하는 도구
- Application, DB, Redis, Nginx 등을 각각 독립적인 컨테이너로 관리한다고 했을 때 멀티 컨테이너 라이프 사이클을 어떻게 관리해야 할까?
- 여러 개의 도커 컨테이너로부터 이루어진 서비스를 구축 및 네트워크 연결, 실행 순서를 자동으로 관리
- `docker-compose.yml` 파일을 작성하여 1회 실행하는 것으로 설정된 모든 컨테이너를 실행

- *application.yml*
    
    ```yaml
    spring:
      config:
        import: optional:file:.env[.properties]
      profiles:
        active: ${PROFILES_ACTIVE} # Active Profile 동적으로 바꿔줄거임
    #    group:
    #      local:
    #        - common
    #      prod:
    #        - common
    
    ---
    
    spring:
      config:
        activate:
          on-profile: common
      datasource:
        # 여기서 localhost가 아니라 docker container의 호스트로 조져줘야됨....!!!!!!!
        url: jdbc:mysql://docker-prac-mysql:${DB_PORT}/${DB_NAME}?serverTimezone=Asia/Seoul&characterEncoding=UTF-8
        username: ${DB_USERNAME}
        password: ${DB_PASSWORD}
        driver-class-name: com.mysql.cj.jdbc.Driver
    
      jpa:
        hibernate:
          ddl-auto: update
        properties:
          hibernate:
            dialect: org.hibernate.dialect.MySQL8Dialect
            format_sql: true
        show-sql: true
      data:
        redis:
          host: ${REDIS_HOST}
          port: 6379
    
    #server:
    #  port: 8081
    
    ---
    
    spring:
      config:
        activate:
          on-profile: local
    
      datasource:
        url: jdbc:mysql://docker-prac-mysql:${DB_PORT}/${DB_NAME}?serverTimezone=Asia/Seoul&characterEncoding=UTF-8
        username: ${DB_USERNAME}
        password: ${DB_PASSWORD}
        driver-class-name: com.mysql.cj.jdbc.Driver
    
      jpa:
        hibernate:
          ddl-auto: update
        properties:
          hibernate:
            dialect: org.hibernate.dialect.MySQL8Dialect
            format_sql: true
        show-sql: true
      data:
        redis:
          host: ${REDIS_HOST}
          port: 6379
    
    #server:
    #  port: 8082
    
    ---
    
    spring:
      config:
        activate:
          on-profile: prod
    
      datasource:
        url: jdbc:mysql://docker-prac-mysql:${DB_PORT}/${DB_NAME}?serverTimezone=Asia/Seoul&characterEncoding=UTF-8
        username: ${DB_USERNAME}
        password: ${DB_PASSWORD}
        driver-class-name: com.mysql.cj.jdbc.Driver
    
      jpa:
        hibernate:
          ddl-auto: validate
        properties:
          hibernate:
            dialect: org.hibernate.dialect.MySQL8Dialect
            format_sql: true
        show-sql: true
      data:
        redis:
          host: ${REDIS_HOST}
          port: 6379
    
    #server:
    #  port: 8083
    
    ```
    
    > 주의할 점은 datasource의 db url host를 container 입장에서 바라보고 써줘야한다.
    > 
- *docker-compose-local.yml*
    
    ```yaml
    version: "3.8"
    
    services:
      docker-prac-redis:
        container_name: myapp-redis
        build:
          dockerfile: Dockerfile
          context: ./redis
        image: swo98/docker-prac-redis
        environment:
          - REDIS_HOST=${REDIS_HOST}
          - REDIS_PORT=${REDIS_PORT}
        ports:
          - "6379:6379"
    
      docker-prac-mysql:
        container_name: myapp_mysql
        build:
          dockerfile: Dockerfile
          context: ./mysql
        image: swo98/docker-prac-mysql
        environment:
          - MYSQL_DATABASE=${DB_NAME}
          - MYSQL_ROOT_PASSWORD=${DB_PASSWORD}
    #    volumes:
    #      - ./mysql/config:/etc/mysql/conf.d
        ports:
          - "3306:3306"
    
      docker-prac-app:
        container_name: myapp-application
        build:
          dockerfile: Dockerfile
          context: .
        depends_on: # redis, mysql container run -> app container run
          - docker-prac-redis
          - docker-prac-mysql
        image: swo98/docker-prac-application
        environment:
          - DB_USERNAME=${DB_USERNAME}
          - DB_PASSWORD=${DB_PASSWORD}
          - DB_HOST=${DB_HOST}
          - DB_PORT=${DB_PORT}
          - DB_NAME=${DB_NAME}
          - PROFILES_ACTIVE=local
        ports:
          - "8080:8080"
        # restart: always
    
    # ports : {컨테이너_외부}:{컨테이너_내부}
    
    ```
    
    > `build`  : ㅇㄷ에 있는 뭐로 빌드 할거냐
    > 
    
    > `depends-on` : 컨테이너 간 의존 설정, 실행 순서를 컨트롤
    > 
    
    > `ports` : {컨테이너_외부}:{컨테이너_내부}
    > 
    
    > `restart` : 컨테이너 안의 서비스가 실행 가능한 상태인지 까지는 확인하지 않기 때문에 의존성 걸어준 애들이 아직 실행 안됐을 경우 실패할텐데 그 때 그냥 재시작 하라는 설정
    > 

*command*

```java
 // .env 파일 자동 반영
$ docker-compose config

// 이미지 없으면 빌드 후 컨테이너 실행, 있으면 해당 이미지 사용
$ docker-compose up

// 무조건 재빌드 후 컨테이너 실행
$ docker-compose up --build
```

*run with .env*

```bash
$ docker-compose -f {compose.yml_파일명} config
$ docker-compose -f {compose.yml_파일명} up --build
```

```bash
$ docker-compose --env-file .env -f docker-compose-local.yml up --build
$ docker-compose --env-file .env -f docker-compose-prod.yml up --build
```

*container db-cli*

```bash
$ docker exec -it {container_id} mysql -u root -p
$ docker exec -it {container_id} redis-cli
```

*reference*

https://velog.io/@cheesechoux/Docker-docker-container-run-%ED%99%98%EA%B2%BD-%EB%B3%80%EC%88%98%EA%B0%80-%EC%84%A4%EC%A0%95%EB%90%98%EC%A7%80-%EC%95%8A%EC%95%98%EC%8A%B5%EB%8B%88%EB%8B%A4

https://da2uns2.tistory.com/entry/Docker-Docker-Compose-%EC%82%AC%EC%9A%A9%ED%95%B4-web-db-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88-%EC%97%B0%EA%B2%B0%ED%95%98%EA%B8%B0-springboot-mariadb

https://docs.docker.com/reference/compose-file/build/
