grinder 오픈소스를 활용한 네이버의 부하 테스트 도구

## 1. **nGrinder controller 설치**

https://github.com/naver/ngrinder/releases 

ngrinder용 디렉토리 생성 후 관리

```bash
$ mkdir ngrinder
$ cd ngrinder
```

## 2. **nGrinder controller 실행**

```bash
$ java -jar ngrinder-controller-3.5.8.war --port= 7070  
```

오류 발생 시

```bash
$ java -Djava.io.tmpdir=/현재유저패스/.ngrinder/lib -jar ngrinder-controller-{버전}.war
```

기본 포트 8080인데, 마지막에 —poart={num} 으로 포트 설정 가능

1. **localhost:{port}/login 접속**
<img width="428" alt="image" src="https://github.com/2eungwoo/TIL/assets/89715722/6ee36224-e59e-4a92-98a8-f4597627ad0d">


초기 id/password = admin/admin

## 3. **Agent 설치 및 실행**

<img width="287" alt="image" src="https://github.com/2eungwoo/TIL/assets/89715722/8e7f70e9-cf27-472a-955f-263731284296">



```bash
# 압축 해제
tar -xvf ngrinder-agent-{version}-localhost.tar

# 폴더로 이동
cd ngrinder-agent
```

```bash
# Agent쉘스크립트 파일 실행
./run_agent.sh

# Agent 백그라운드 실행
./run_agent_bg.sh
```

```
admin -> AgentManagementagent 가 정상적으로 실행되고 잘 등록되었는지admin > Agent Management에서 확인 가능합니다
```

admin → agent management 에서 정상적으로 agent가 실행되고 등록되었는지 확인 가능

<img width="706" alt="image" src="https://github.com/2eungwoo/TIL/assets/89715722/6bfaa341-3ed3-47de-81a4-df10f58b9086">



## 4. **Script 작성**

<img width="705" alt="image" src="https://github.com/2eungwoo/TIL/assets/89715722/eca8069c-7670-4939-8447-a11030353777">



Script → Create a Script

<img width="705" alt="image" src="https://github.com/2eungwoo/TIL/assets/89715722/1b664f78-02ea-43a8-a924-bb03a11afd59">


<img width="705" alt="image" src="https://github.com/2eungwoo/TIL/assets/89715722/55e1ff73-321f-476d-97f8-6f8f240ee541">



Validate 버튼으로 스크립트 검증, 아래 로그와 같이 성공로그 확인

참고로 java —version 11로 버전 변경 후에 에러 잡혔음

## 5. Test 작성 및 결과 확인

상단 Peerformance Test 버튼

<img width="706" alt="image" src="https://github.com/2eungwoo/TIL/assets/89715722/3568cc2e-614a-459f-8089-a57cb6a4fb20">


<img width="700" alt="image" src="https://github.com/2eungwoo/TIL/assets/89715722/79516d05-89d3-4d3e-8b4d-eefedd3c2212">


- 테스트 옵션
    - **Agent** : 성능 측정에 사용할 Agent
        - Agent를 여러개로 구성하고 싶은 경우 Docker 나 cloud service 를 고려해 볼 수 있다.
        - 일반적인 로컬에서 테스트를 실행할 경우 1이 고정값
    - **Vuser per agent** : 프로세스 수 * 스레드 수가 각 agent당 **가상 사용자의 수**가 된다.
        - Agent 당 설정할 가상 사용자 수
        - 동시에 요청을 날리는 사용자
    - **Process / Thread**
        - 하나의 Agent에서 생성할 프로세스와 스레드 개수
        - 한 Agent에 얼마나 많은 worker process를 쓸지이다. 그리고 이 process들은 또 스레드들을 가지게 된다
    - **Script**
        - 성능 측정 시 각 Agent 에서 실행할 스크립트
        - 방금 우리가 생성한 Script를 추가한다.
    - **Duration : 얼마나 오래 테스트가 지속될지 정하는 것**
        - 성능 측정 수행 시간 (0:00:10 (시:분:초로 10초 의미))
        - 어느 블로그의 말을 따르면 어느정도 길게 확보해줘야 의미있는 평균치가 나온다고 한다.
    - **Run Count**
        - 스레드 당 테스트 코드를 수행하는 횟수
        - Run Count와 Duration의 경우 둘 중 하나만 선택해서 기간 동안 실행하거나, Run Count 만큼 실행하게 한다.
    - **Enable Ramp-up**
        - 성능 측정 과정에서 가상 사용자를 점진적으로 늘리도록 활성화
    - **Initial Count**
        - 처음 시작 시 가상 사용자 수
    - **Initial Sleep Time**
        - 테스트 시작 시간
    - **Incremental Step**
        - Process 또는 thread 를 증가시키는 개수
    - **Interval**
        - Process 또는 Thread를 증가시키는 시간 간격
    - **Ramp-up**
        - Ramp-up을 체크하면 프로세스 수가 점진적으로 늘어난다. **점진적인 부하를 테스트**하고 싶을 때 쓰면 될 것 같다.
    
- 결과 옵션
    - MTT 는 평균적인 1회 수행 시간으로 중요한 지표이다.
    - TPS 는 초당 트랜잭션 개수를 말한다.
    - Total Vusers: Vusers 수 (버츄얼 유저)
    - TPS: 평균 TPS
    - Peak TPS: 최고 TPS
    - Mean Test Time: 평균 테스트시간
    - Executed Tests: 테스트 실행 횟수
    - Successful Tests: 테스트 성공 횟수
    - Errors: 에러 횟수
    - Run time: 테스트 실행시간

- 출처: https://0soo.tistory.com/223#https://naver.github.io/ngrinder/ [Lifealong:티스토리]
