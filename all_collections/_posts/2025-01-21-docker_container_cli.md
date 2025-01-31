---
layout: post
title: 🆑 자주 쓰는 Docker Container CLI
date: 2025-01-21
categories: ["SamBeak", "Docker", "devops"]
---

# 도커 CLI 명령어 <br>

> ## 컨테이너 관련 명령어

🏷️컨테이너 조회 <br>

```bash
# docker image ls
docker image ls
```

🏷️컨테이너 생성 <br>

```bash
docker ps # 실행 중인 컨테이너 목록 조회
docker ps -a # 전체 컨테이너 목록 조회
```

🏷️컨테이너 실행 <br>

```bash
# docker start [containerId]
docker start d32 # 3자리 이상 기입
```

🏷️컨테이너 중지 <br>

```bash
# docker stop [containerId]
docker stop d32
```

🏷️컨테이너 강제 종료 <br>

```bash
# docker kill [containerId]
docker kill d32
```

🏷️중지 된 컨테이너 삭제 <br>

```bash
#  docker rm [containerId]
docker rm d32
```

🏷️중지 된 모든 컨테이너 삭제 <br>

```bash
# docker rm $(docker ps -qa)
docker rm $(docker ps -qa)
```

🏷️ 중지 후 컨테이너 삭제 <br>

```bash
#  docker rm -f [containerId]
docker rm -f d32
```

🏷️ 컨테이너 통합 실행 <br>

```bash
# docker run [imageName]
docker run nginx

# --name : 컨테이너 이름 명명
# docker run --name [containerName] [imageName]
dokcer run --name nginxContainer nginx

# -d : 백그라운드 실행
# docker run -d [imageName]
docker run -d nginx

# -p : 포트 지정 실행
# docker run -d -p [hostPort]:[containerPort] [imageName]
docker run -d -p 4000:80 nginx
# 4000번포트로 접근하면 container 포트가 80인 곳으로 연결

```

🏷️ 컨테이너 내부 접속 <br>

```bash
# docker exec -it containerId bash
# exit 나가기
docker exec -it d32 bash
exit
```

🏷️ 컨테이너 환경 변수 설정 실행 <br>

```bash
# docker run -e [eName]=[eValue] [imageName]
docker run -e MYSQL_ROOT_PASSWORD=password123 -d -p 3306:3306 mysql
```

🏷️ 컨테이너 볼륨 설정 실행 <br>

```bash
# docker run -v [hostPath]:[containerPath] [imageName]
docker run -e MYSQL_ROOT_PASSWORD=[password] -d -p 3306:3306 -v [host pathName]:/var/lib/mysql
```
