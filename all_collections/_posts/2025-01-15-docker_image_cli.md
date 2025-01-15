---
layout: post
title: 🆑 자주 쓰는 Docker Image CLI
date: 2025-01-15
categories: ["SamBeak", "Docker", "devops"]
---

# 도커 CLI 명령어 <br>

> ## 이미지 관련 명령어

🏷️이미지 조회 <br>

```bash
# docker pull [imageName]:[tagName]
docker pull nginx
docker pull nginx:lastest
docker pull nginx:alpine # 안정판
```

🏷️ (컨테이너 등록 미포함) 이미지 삭제 <br>

```bash
# docker image rm [imageId]
docker image rm d12
# imageId는 3자리 이상만 기입해도 된다
```

🏷️ (컨테이너 등록 포함) 이미지 삭제 <br>

```bash
# docker image rm -f [imageId]
docker image rm -f d12
```

컨테이너에서 실행 중인 이미지는 삭제가 불가하다 <br><br>

🏷️ 이미지 삭제 (확장판) <br>

```bash
# docker image rm -f $(docker image -q)
docker image rm -f $(docker image -q)
```

컨테이너에서 실행 중이지 않은 <br>
모든 이미지를 삭제한다<br><br>

🏷️ 이미지 조회 <br>

```bash
# docker image ls
docker image ls
```
