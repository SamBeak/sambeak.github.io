---
layout: post
title: ✒️ 도커 파일 작성법
date: 2025-01-31
categories: ["SamBeak", "Docker", "devops"]
---

# Dockerfile 기초 <br>

> ## 나만의 도커 이미지 만들기

### 🏷️ 도커 파일이란 ? <br>

컨테이너에 이미지를 실행시키다 보면 <br>
나만의 이미지를 생성하고 실행하고 싶다는 <br>
생각이 든다. <br><br>

이러한 생각을 실현시켜주는 것이 바로 <br>
Dockerfile 이다. <br><br>

내가 개발한 코드로 이미지를 생성하여 <br>
이를 활용해 Docker compose에 쓰기도 하고 <br>
쿠버네티스에 활용하기 때문에 <br>
이미지를 생성하는 과정의 초석을 담당하는 <br>
상당히 중요한 과정이 바로 Dockerfile이다. <br><br>

### 🏷️ Base Image 명시 <br>

```bash
# FROM [imageName]:[tagName]
FROM node:alpine
```

컨테이너를 새로 띄워 <br>
가상 환경을 구축할 때 <br>
기본 이미지를 어떤 것을 사용할 것이고, <br>
어떻게 구성할 것인지를 선택하는 단계이다. <br>
예를 들어, Python 환경이나 jdk 환경처럼 말이다. <br><br>

### 🏷️ 컨테이너 실행 시 최초 명령어 설정 <br>

```bash
# ENTRYPOINT
ENTRYPOINT ["/bin/bash", "-c", "sleep 500"] # 컨테이너가 종료되지 않도록하는 명령어
ENTRYPOINT ["/bin/bash", "-c", "echo hello"] # 리눅스 명령어를 실행시킨다
ENTRYPOINT ["java", "-jar", "/app.jar"] # app.jar파일을 바로 실행
ENTRYPOINT ["npm", "run", "start"]
ENTRYPOINT ["node", "dist/main.js"]
```

### 🏷️ 파일 복사 명령어 <br>

> 폴더의 경로는 /로 끝나야 한다 <br>

```bash
# COPY [hostPath] [containerPath]
COPY *.txt /txt_folder/
COPY . .
```

### 🏷️ 작업 디렉토리 지정 명령어 <br>

```bash
# WORKDIR /[customPathName]
WORKDIR /app
```

### 🏷️ 사용 중인 포트 명시 및 문서화 <br>

```bash
# EXPOSE [portNumber]
EXPOSE 3000
```

### 🏷️ 명령어 실행 <br>

```bash
# RUN [명령어]
RUN npm install
RUN npm run build
```

### 🏷️ Dockerfile 기반 이미지 생성 <br>

```bash
# docker build -t [imageName] [dockerFilepath]
docker build -t nextapp .
```

### 🏷️ Next 이미지 생성 예제 <br>

```bash
FROM node:alpine
WORKDIR /app
COPY . .
EXPOSE 3000
RUN npm install
RUN npm run build
ENTRYPOINT["npm", "start"]
```

```bash
docker build -t next-app .
docker run -d -p 3000:3000 next-app
```
