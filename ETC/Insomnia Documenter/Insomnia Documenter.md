
# Insomnia api document

## api_doc 폴더 생성

프로젝트 루트에 폴더 생성하였음

api_doc / src

## insomnia 설정

doc 생성할 collection 선택 후 

Preferences → Data → Export

![image](https://github.com/user-attachments/assets/5f3b31ee-a8a9-48be-ae13-abf573238366)

![image](https://github.com/user-attachments/assets/7940e8a5-3f6c-4d9b-9074-d4d22f49a92f)

![image](https://github.com/user-attachments/assets/0610facf-f5b1-4697-8a17-ad60992388ce)

JSON 으로 루트 폴더에 생성한 api_doc/src 에 Done

해당 폴더 위치에서 명령어 입력

```jsx
$ npx insomnia-documenter --config {파일명} --output {지정폴더명}
```

![image](https://github.com/user-attachments/assets/ae82f00f-c45f-4878-b7e0-def70af4d044)

![image](https://github.com/user-attachments/assets/5d9d117d-b173-457e-90b8-ba7f3b1e2eb6)

파일 생성 완료

/apidoc_sample 진입 후 명령어 입력

```jsx
$ npx serve
```

![image](https://github.com/user-attachments/assets/de118df0-0350-480f-87d8-a56aa2cc4a70)

완료

![image](https://github.com/user-attachments/assets/2b1ca19c-bc8c-4483-a838-4cc6296e784c)
