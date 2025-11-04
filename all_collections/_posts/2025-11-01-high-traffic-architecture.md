---
layout: post
title: 🚀 대용량 트래픽 처리 아키텍처 완벽 가이드
date: 2025-11-01
categories:
  [
    "SamBeak",
    "Architecture",
    "System Design",
    "Load Balancing",
    "Scalability",
    "Performance",
  ]
---

# 대용량 트래픽 처리 아키텍처란 무엇인가

<br>
서비스를 운영하다 보면 갑자기 사용자가 폭증하는 순간이 온다. <br>
쇼핑몰의 블랙프라이데이, 티켓 예매 오픈, 이벤트 시작 시점처럼 <br>
평소보다 수십 배, 수백 배의 트래픽이 몰리는 상황 말이다. <br><br>

이는 마치 작은 골목길에 갑자기 수천 명이 몰려드는 것과 같다. <br>
아무리 좋은 가게라도 문이 하나밖에 없고, <br>
계산원이 한 명이면 긴 줄이 생기고 <br>
결국 손님들은 기다리다 지쳐 떠나버린다. <br><br>

대용량 트래픽 처리 아키텍처는 바로 <br>
이런 상황에서도 서비스가 멈추지 않고 <br>
모든 사용자에게 빠르고 안정적인 경험을 제공하기 위한 <br>
시스템 설계 방법이다. <br><br>

> ## 왜 대용량 트래픽 대비가 필요할까?

<br>
대용량 트래픽에 대비하지 않으면 다음과 같은 문제가 발생한다. <br><br>

**문제 1: 서버 다운** <br>
한 대의 서버가 감당할 수 있는 요청에는 한계가 있다. <br>
초과하면 서버가 멈추고 모든 사용자가 접속 불가 상태가 된다. <br><br>

**문제 2: 느린 응답 속도** <br>
요청이 몰리면 처리 대기 시간이 길어진다. <br>
사용자는 3초만 기다려도 이탈률이 급증한다. <br><br>

**문제 3: 데이터베이스 과부하** <br>
모든 요청이 DB에 직접 접근하면 <br>
DB가 병목 지점이 되어 전체 시스템이 느려진다. <br><br>

**문제 4: 비즈니스 손실** <br>
서비스 다운은 곧 매출 손실과 신뢰도 하락으로 이어진다. <br><br>

# 기본 개념 요약

<br>
대용량 트래픽을 처리하기 위한 핵심 전략은 <br>
**분산(Distribution)**과 **효율화(Optimization)**이다. <br><br>

## 🏷️ 핵심 구성 요소

<br>

### 1. 로드 밸런싱 (Load Balancing)

<br>
**개념**: 들어오는 요청을 여러 서버에 균등하게 분산시킨다. <br><br>

**비유**: <br>
은행 창구를 생각해보자. <br>
창구가 하나면 줄이 길지만, <br>
창구를 10개로 늘리고 입구에서 번호표를 나눠주면 <br>
대기 시간이 1/10로 줄어든다. <br><br>

**주요 알고리즘**:

- **Round Robin**: 순서대로 돌아가며 분배
- **Least Connections**: 현재 연결이 가장 적은 서버로 분배
- **IP Hash**: 같은 사용자는 항상 같은 서버로 연결
- **Weighted**: 서버 성능에 따라 가중치를 두고 분배

<br>

### 2. 캐싱 (Caching)

<br>
**개념**: 자주 사용되는 데이터를 빠른 저장소에 미리 저장해둔다. <br><br>

**비유**: <br>
도서관에서 책을 찾을 때마다 서고까지 가는 것보다, <br>
인기 도서는 대출 카운터 근처에 미리 쌓아두면 <br>
찾는 시간이 획기적으로 줄어든다. <br><br>

**캐싱 계층**:

- **CDN(Content Delivery Network) 캐싱**: 정적 파일을 전 세계에 분산 저장
- **애플리케이션 캐싱**: Redis, Memcached로 API 응답 저장
- **데이터베이스 캐싱**: 쿼리 결과를 메모리에 저장
- **브라우저 캐싱**: 사용자 브라우저에 저장

<br>

### 3. 데이터베이스 확장

<br>
**Replication (복제)**:

- Master DB: 쓰기 전담
- Slave DB: 읽기 전담
- 읽기 요청이 80% 이상인 서비스에 효과적

**Sharding (샤딩)**:

- 데이터를 여러 DB로 수평 분할
- 예: 사용자 ID 범위별로 분산

<br>

### 4. 비동기 처리

<br>
**개념**: 시간이 오래 걸리는 작업은 나중에 처리한다. <br><br>

**비유**: <br>
음식점에서 주문을 받을 때, <br>
모든 요리가 완성될 때까지 손님을 세워두지 않고 <br>
주문만 받고 자리로 안내한 뒤 나중에 음식을 가져다준다. <br><br>

**활용 사례**: 이메일 발송, 이미지 처리, 데이터 집계, 알림 발송 <br><br>

### 5. CDN (Content Delivery Network)

<br>
**개념**: 정적 콘텐츠를 사용자와 가까운 곳에서 제공한다. <br><br>

**효과**: 응답 속도 향상, 원본 서버 부하 감소, DDoS 방어 <br><br>

## 🏷️ 확장성의 두 가지 방향

<br>

### 수직 확장 (Scale Up)

<br>
**방법**: 서버의 성능을 높인다. (CPU, 메모리 증설) <br>
**장점**: 구현 간단, 코드 수정 불필요 <br>
**단점**: 비용 급증, 물리적 한계 존재, 장애 시 전체 중단 <br><br>

### 수평 확장 (Scale Out)

<br>
**방법**: 서버의 개수를 늘린다. <br>
**장점**: 비용 효율적, 무한 확장 가능, 장애 시 일부만 영향 <br>
**단점**: 설계 복잡, 세션 관리 문제 발생 <br><br>

**결론**: 대부분의 대규모 서비스는 **수평 확장**을 기본으로 한다. <br><br>

# 실전 아키텍처 구성

<br>

> ## 단계 1: 기본 구성

<br>

```
[사용자] → [웹 서버] → [데이터베이스]
```

<br>
**문제점**: 동시 접속자 1,000명 이상이면 버거움 <br><br>

> ## 단계 2: 로드 밸런서 추가

<br>

```
                     ┌─→ [웹 서버 1] ─┐
[사용자] → [로드 밸런서] ─┼─→ [웹 서버 2] ─┼→ [DB]
                     └─→ [웹 서버 3] ─┘
```

<br>
**개선**: 처리량 3배, 무중단 배포 가능 <br><br>

> ## 단계 3: DB 복제 추가

<br>

```
                                   ┌→ [Master DB] (쓰기)
[사용자] → [로드 밸런서] → [웹 서버들] ┼→ [Slave DB 1] (읽기)
                                   └→ [Slave DB 2] (읽기)
```

<br>
**개선**: 읽기 부하 분산, 읽기 성능 대폭 향상 <br><br>

> ## 단계 4: 캐시 추가

<br>

```
[사용자] → [로드 밸런서] → [웹 서버들] → [Redis 캐시] → [DB]
```

<br>
**개선**: DB 부하 80% 감소, 응답 시간 10배 단축 <br><br>

> ## 단계 5: 완성형

<br>

```
[사용자] → [CDN] → [로드 밸런서] → [웹 서버들] → [Redis] → [DB]
                                        ↓
                                   [메시지 큐]
                                        ↓
                                   [Worker 서버들]
```

<br>
**개선**: 정적 파일 CDN 제공, 무거운 작업 비동기 처리 <br><br>

# Nginx 로드 밸런서 설정 예시

<br>

> ## Round Robin 방식

<br>

```nginx
http {
    upstream backend {
        server 10.0.1.101:8080;
        server 10.0.1.102:8080;
        server 10.0.1.103:8080;
    }

    server {
        listen 80;
        location / {
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```

<br>

> ## Least Connections 방식

<br>

```nginx
upstream backend {
    least_conn;
    server 10.0.1.101:8080;
    server 10.0.1.102:8080;
    server 10.0.1.103:8080;
}
```

<br>

> ## 가중치 설정

<br>

```nginx
upstream backend {
    server 10.0.1.101:8080 weight=3;
    server 10.0.1.102:8080 weight=2;
    server 10.0.1.103:8080 weight=1;
}
```

<br>

> ## 헬스 체크

<br>

```nginx
upstream backend {
    server 10.0.1.101:8080 max_fails=3 fail_timeout=30s;
    server 10.0.1.102:8080 max_fails=3 fail_timeout=30s;
    server 10.0.1.103:8080 backup;
}
```

<br>

# Redis 캐싱 예시

<br>

> ## 기본 캐싱 패턴

<br>

```python
import redis
import json

cache = redis.Redis(host='localhost', port=6379)

def get_user_profile(user_id):
    # 1. 캐시 확인
    cache_key = f"user:{user_id}"
    cached = cache.get(cache_key)

    if cached:
        return json.loads(cached)

    # 2. DB 조회
    data = db.query(f"SELECT * FROM users WHERE id={user_id}")

    # 3. 캐시 저장 (1시간)
    cache.setex(cache_key, 3600, json.dumps(data))

    return data
```

<br>

> ## 인기 게시물 캐싱

<br>

```python
def get_popular_posts():
    cached = cache.get("posts:popular")
    if cached:
        return json.loads(cached)

    posts = db.query("SELECT * FROM posts ORDER BY views DESC LIMIT 10")
    cache.setex("posts:popular", 300, json.dumps(posts))

    return posts
```

<br>

> ## 캐시 무효화

<br>

```python
def update_user(user_id, data):
    db.update(f"UPDATE users SET ... WHERE id={user_id}")
    cache.delete(f"user:{user_id}")  # 캐시 삭제
```

<br>

# 메시지 큐 비동기 처리

<br>

> ## Producer (작업 요청)

<br>

```python
import pika
import json

def send_email_async(user_email):
    connection = pika.BlockingConnection()
    channel = connection.channel()
    channel.queue_declare(queue='email_queue')

    message = {'email': user_email, 'type': 'welcome'}
    channel.basic_publish(
        exchange='',
        routing_key='email_queue',
        body=json.dumps(message)
    )

    print("이메일 작업 큐에 추가")
    # 즉시 리턴!
```

<br>

> ## Consumer (작업 처리)

<br>

```python
def process_email(ch, method, properties, body):
    message = json.loads(body)
    send_email(message['email'])  # 실제 이메일 발송
    ch.basic_ack(delivery_tag=method.delivery_tag)

connection = pika.BlockingConnection()
channel = connection.channel()
channel.basic_consume(queue='email_queue', on_message_callback=process_email)
channel.start_consuming()
```

<br>

# 성능 모니터링

<br>

> ## 주요 지표

<br>

**서버 지표**:

- CPU 사용률 (80% 이상 → Scale Out)
- 메모리 사용률 (90% 이상 → 위험)
- 디스크 I/O
- 네트워크 대역폭

**애플리케이션 지표**:

- 응답 시간 (평균, p95, p99)
- 처리량 (RPS)
- 에러율 (5xx)
- 동시 접속자 수

**데이터베이스 지표**:

- 쿼리 실행 시간
- 커넥션 풀 사용률
- Replication Lag
- 락 대기 시간

**캐시 지표**:

- 캐시 히트율 (80% 이상 목표)
- 메모리 사용률

<br>

# 실전 체크리스트

<br>

## ✅ 데이터베이스 최적화

<br>

- [ ] 인덱스 추가
- [ ] N+1 쿼리 제거
- [ ] 커넥션 풀 조정
- [ ] Replica 분리

<br>

## ✅ 캐싱 적용

<br>

- [ ] API 응답 캐싱
- [ ] 정적 파일 CDN
- [ ] 쿼리 결과 캐싱

<br>

## ✅ 로드 밸런싱

<br>

- [ ] Nginx/HAProxy 설정
- [ ] 헬스 체크 구성
- [ ] 세션 관리 전략

<br>

## ✅ 비동기 처리

<br>

- [ ] 메시지 큐 도입
- [ ] Worker 서버 구성
- [ ] 재시도 로직

<br>

## ✅ 모니터링

<br>

- [ ] Prometheus/Grafana 구축
- [ ] 알림 설정
- [ ] 로그 수집 (ELK)

<br>

# 요약

<br>
대용량 트래픽을 처리하는 핵심은 **분산**과 **효율화**다. <br><br>

**💎 핵심 포인트**:

1. **로드 밸런서**로 요청을 여러 서버에 분산
2. **캐싱**으로 DB 부하를 80% 이상 감소
3. **DB Replication**으로 읽기 성능 향상
4. **메시지 큐**로 무거운 작업을 비동기 처리
5. **CDN**으로 정적 파일을 빠르게 제공
6. **모니터링**으로 병목 지점을 실시간 파악

<br>

**🚀 단계별 적용**:

1단계: 로드 밸런서 + 서버 증설 <br>
2단계: Redis 캐싱 적용 <br>
3단계: DB Replication 구성 <br>
4단계: 메시지 큐 도입 <br>
5단계: CDN 적용 <br><br>

처음부터 완벽한 아키텍처를 만들 필요는 없다. <br>
트래픽이 증가하면서 단계적으로 개선해나가는 것이 현실적이다. <br>
중요한 것은 **병목 지점을 파악하고 우선순위를 정해 해결**하는 것이다. <br><br>
