레퍼런스 : https://quantrader.tistory.com/157

### build

1. 프로젝트 디렉토리 이동 
2. ./gradlew build
3. 빌드된 파일(*jar) : build/libs 에 있음
4. 해당 디렉토리 이동 후 실행 java -jar {빌드파일명}

### **docker 이미지 생성**

이제 gradle Tasks 에서 bootJar 더블클릭을 통해 새로운 jar 파일을 만들어 준다. 이 파일을 docker 로 올리고 tar 파일로 저장해서 운영서버에 올리는 작업까지 해보자.

jar 파일이 생성된 build/libs 폴더에 Dockerfile 을 작성

openjdk 17 버전과 jar 파일을 app.jar 파일로 변환, 9201 포트를 EXPOSE 해주는 내용으로 작성

```java
FROM openjdk:17
ARG JAR_FILE=*.jar
COPY ${JAR_FILE} app.jar
EXPOSE 9201
ENTRYPOINT ["java","-jar","app.jar"]
```

해당 폴더 경로의 터미널을 열어 docker build 명령어로 도커에 이미지를 올려주기

```java
docker build -t {image_name} ./
```

> `docker build -t {name} ./` 명령어에서 `./`는 Dockerfile이 있는 **현재 디렉토리를 나타냄**
> 

빌드 과정을 거쳐 정상적으로 docker desktop 에 이미지가 만들어졌다.

name:tag ⇒ docker-practice:latest
<img width="1301" alt="스크린샷 2024-09-20 오후 6 26 03" src="https://github.com/user-attachments/assets/45e08802-9222-46fc-b268-fbd341160c27">


생성된 이미지를 docker-practice.tar 파일로 저장하여 운영서버로 복사한 뒤 다시 load 하는 과정을 거친다

```java
#{image_name}.tar로 이미지 저장
$ docker save -o ~/경로/{image_name}.tar {image_name}:latest

# 예시
$ docker save -o ~/Desktop/projects/docker-practice/{image_name}.tar {image_name}:latest
$ docker save -o ~/Desktop/projects/docker-practice/docker-practice.tar docker-practice:latest

```

```java
#{image_name} 이미지 로드
$ docker load -i ~/경로/{image_name}.tar

# 예시
$ docker load -i ~/Desktop/projects/docker-practice/{image_name}.tar
$ docker load -i ~/Desktop/projects/docker-practice/docker-practice.tar

```

로드된 이미지는 docker desktop 이미지 탭에서 확인이 가능합니다. 이제 docker run 명령어를 통해 컨테이너로 서버를 띄워준다.

데몬으로 실행되며 9201 포트 맵핑 시켜서 진행

```java
docker run -d -p 9201:9201 {image_name}
```

운영서버 docker desktop에서 정상적으로 서버가 실행되는 것을 확인 가능하다.

<img width="1263" alt="스크린샷 2024-09-20 오후 6 30 52" src="https://github.com/user-attachments/assets/284b1b90-0fc3-49c9-a893-d1609baadde8">


### 로컬로 접속해보기

프로젝트 포트를 9201로 바꾼 후

```java
server.port = 9201
```

[localhost:9201/docker-test로](http://localhost:9201/docker-test로) 접속해보자

<img width="435" alt="스크린샷 2024-09-20 오후 6 49 33" src="https://github.com/user-attachments/assets/60f73229-5982-4481-a020-663811e8f1f7">


잘 나옴
