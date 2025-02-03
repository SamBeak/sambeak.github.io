---
layout: post
title: ✒️ 도커 컴포즈 작성법
date: 2025-02-03
categories: ["SamBeak", "Docker", "devops"]
---

# Docker Compose 기초 <br>

> ## 나만의 도커 컴포즈 만들기

### 🏷️ 도커 컴포즈란 ? <br>

한번에 하나의 컨테이너를 띄우지 않고<br>
동시에 여러 개의 컨테이너를 띄워야할 때도 있다.<br>
이런 경우, <br>
하나씩 일일이 컨테이너를 띄워야할까? <br><br>

결로만 두고 말하자면, 그렇지 않다. <br>
물론, 어느정도 할 수 있다. <br>
그러나 엄청난 비효율과 <br>
유지보수의 상당한 어려움이 동시에 <br>
수반될 것이다 <br><br>

그래서 도커는 컴포즈를 통해 <br>
여러 개의 컨테이너를 <br>
각각 서비스로 정의하고 구성하여 <br>
하나의 묶으로 관리할 수 있도록 한다 <br><br>

쉽게 말해, 여러 개의 컨테이너를 <br>
동시에 띄울 수 있게 해주는 것이다 <br><br>

### 🏷️ 도커 컴포즈 주요 실수<br>

각 컨테이너는 자신만의 네트워크망과 <br>
IP 주소를 보유한다. <br><br>

즉, localhost라고 표기해버리면 <br>
호스트 컴퓨터를 가리키는 것이 아니라 <br>
각각의 컨테이너 자신을 가리키게 된다는 것이다 <br><br>

따라서, DB 커넥션 등 다른 서버랑 커넥션을 붙히려면 <br>
localhost가 아닌 compose.yml에 정의한 <br>
service 이름으로 작성해야 한다. <br><br>

아래는 Spring의 DB connection 예제다 <br>

```XML
<!--context-datasource.xml -->
<property name="url" value="jdbc:mysql://my-db-server:3306/testDB" />
```

### 🏷️ Services <br>

```YAML
services:
	[serviceName]:
```

docker compose에서 하나의 컨테이너를 <br>
서비스라 부른다. <br><br>

### 🏷️ Container_name <br>

```YAML
# container_name: [containerName]
container_name: webserver
```

### 🏷️ Image <br>

컨테이너에 어떤 이미지를 사용할 것인지 설정 <br>

```YAML
# image: [imageName]
image: nginx
```

### 🏷️ Ports <br>

호스트 포트와 컨테이너 포트를 설정 <br>

```YAML
# ports
#   - [hostPort]:[containerPort]
ports:
	- 80:80
```

### 🏷️ Environment <br>

환경변수를 설정 <br>

```YAML
# environment:
#    [environmentName]: [customValue]
environment:
	MYSQL_ROOT_PASSWORD: [password]
	MYSQL_DATABASE: [dbName]
```

### 🏷️ Volumes <br>

해당 서비스의 볼륨 설정 <br>

```YAML
# volumes:
#      - [hostPath]:[containerPath]
volumes:
      - ./mysql_data:/var/lib/mysql
```

### 🏷️ Healthcheck <br>

컨테이너의 상태를 체크하기 위한 설정 <br>

- test : 명령어를 실행하여 결과 확인 <br>
- interval: [시간]마다 체크 <br>
- timeout: [시간] 내 결과 나오지 않으면 실패로 간주 <br>
- retries: [횟수]만큼 재시도 <br>
- start_period: [시간] 후에 체크 시작 <br>

```YAML
# healthcheck:
#    test: [command]
#    interval: [time]
#    timeout: [time]
#    retries: [time]
#    start_period: [time]
healthcheck:
      test: ["CMD", "mysqladmin", "ping"]
      interval: 5s
      timeout: 3s
      retries: 3
      start_period: 10s
```

### 🏷️ Build <br>

Dockerfile을 기반으로 빌드한 이미지를 <br>
활용하겠다는 설정 <br><br>

```YAML
# build [Dockerfile경로]
build .
```

### 🏷️ Depends_on <br>

의존 관계를 설정하여 <br>
조건이 성립할 때, 해당 서비스를 실행시킴 <br>

- condition: [조건]이 성립할 때, 실행 <br>

```YAML
# depends_on:
#     [serverName]:
#         condition: service_healthy
depends_on:
      mysql-server:
        condition: service_healthy
```

### 🏷️ compose.yml 작성 예제 <br>

```YAML
services:
  my-db:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORLD: [password]
      MYSQL_DATABASE: [dbName]
    volumes:
      - ./mysql_data:/var/lib/mysql
    ports:
      - 3306:3306
    healthcheck:
      test: ["CMD", "mysqladmin", "ping"]
      interval: 5s
      retries: 10

  my-cache-server:
    image: redis
    ports:
      - 6379:6379
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      retries: 10

  my-servedr:
    build: .
    ports:
      - 8080:8080
    depends_on:
      my-db:
        condition: service_healthy
      my-cache-server:
        condition: service_healthy
```
