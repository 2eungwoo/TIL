## JDK 변경

현재 jdk 버전 확인

```jsx
$ java -version
```

설치된 모든 jdk 버전들 확인

```jsx
$ /usr/libexec/java_home -V
```

그 다음 버전을 보고 변경 후 저장

```jsx
$ export JAVA_HOME=$(/usr/libexec/java_home -v {버전})
$ source ~/.zshrc
```

버전 확인

```jsx
$ java -version
```

## JAVA_HOME변경 ( java 17 설치 )

현재 자바 버전 확인

```bash
# 현재 자바 버전 확인
$ java --version

# 로컬에 설치되어 있는 자바 버전 확인
$ /usr/libexec/java_home -V
```

자바 버전 변경

```bash

# 버전 변경
$ export JAVA_HOME=$(/usr/libexec/java_home -v {버전})

# 자바 버전 확인
$ java --version
```

자바 기본 버전 설정

```bash

# zsh 사용자
$ vi ~/.zhrc

# bash 사용자
$ vi ~/.bash_profile
```

```bash
 # 설정 추가
export JAVA_HOME=$(/usr/libexec/java_home -v {버전})

# 변경사항 적용
source ~/.zshrc
```

참고 : https://llighter.github.io/install-java-on-mac/

## **<!> command not found: vi**

터미널에서 다음과 같이 임시적으로 환경변수 설정

```bash
$ export PATH=%PATH:/bin:/usr/local/bin:/usr/bin
```

```bash
$ vi ~/.zhsrc

# 환경변수 올바르게 설정 후 적용
$ source ~/.zhsrc
```
