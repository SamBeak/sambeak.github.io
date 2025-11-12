---
layout: post
title: ⚡ 캐싱 전략 완벽 가이드
date: 2025-11-03
categories:
  ["SamBeak", "Cache", "Redis", "Performance", "Architecture", "Optimization"]
---

# 캐싱 전략이란 무엇인가

<br>
웹사이트를 이용하다 보면 같은 페이지를 다시 열 때 <br>
처음보다 훨씬 빠르게 로딩되는 경험을 한 적이 있을 것이다. <br><br>

이는 마치 냉장고에 자주 먹는 음식을 미리 꺼내놓는 것과 같다. <br>
매번 냉장고를 열어 찾는 대신, <br>
테이블 위에 꺼내두면 바로 먹을 수 있다. <br><br>

캐싱(Caching)은 바로 <br>
자주 사용되는 데이터를 빠른 저장소에 미리 저장해두고 <br>
필요할 때 즉시 가져다 쓰는 성능 최적화의 핵심 기술이다. <br><br>

> ## 왜 캐싱 전략이 필요할까?

<br>

**문제 1: 느린 응답 속도** <br>
DB 조회는 10~100ms 걸리지만, 캐시는 1ms 이내 (10~100배 빠름) <br><br>

**문제 2: 데이터베이스 과부하** <br>
같은 데이터를 수천 명이 조회하면 DB가 병목이 된다. <br>
캐시 사용 시 DB 부하 80% 이상 감소 <br><br>

**문제 3: 비용 증가** <br>
DB 서버 확장은 비용이 많이 든다. <br><br>

**문제 4: 외부 API 의존성** <br>
외부 API 호출은 네트워크 지연과 비용 발생 <br><br>

# 기본 개념 요약

<br>

## 🏷️ 캐싱 계층

<br>

### 1. 브라우저 캐시

이미지, CSS, JavaScript를 브라우저에 저장 <br><br>

### 2. CDN 캐시

정적 파일을 전 세계에 분산 저장 <br><br>

### 3. 애플리케이션 캐시 (Redis/Memcached)

API 응답, 계산 결과, 세션 데이터 저장 <br><br>

### 4. 데이터베이스 캐시

쿼리 결과를 메모리에 저장 <br><br>

## 🏷️ 주요 캐싱 패턴

<br>

### 1. Cache-Aside (가장 일반적)

<br>

```
1. 캐시 확인
2. 캐시 히트 → 반환
3. 캐시 미스 → DB 조회 → 캐시 저장 → 반환
```

<br>
**장점**: 필요한 데이터만 캐싱 <br>
**단점**: 첫 요청은 느림 <br><br>

### 2. Write-Through

데이터 쓰기 시 캐시와 DB에 동시 저장 <br>
**장점**: 데이터 일관성 보장 <br>
**단점**: 쓰기 성능 저하 <br><br>

### 3. Write-Behind

캐시에만 먼저 쓰고, DB는 비동기 저장 <br>
**장점**: 쓰기 성능 매우 빠름 <br>
**단점**: 캐시 장애 시 데이터 손실 위험 <br><br>

## 🏷️ TTL (Time To Live)

<br>
일정 시간 후 자동으로 캐시 삭제 <br><br>

**권장 TTL**:

- 정적 데이터: 24시간
- 동적 데이터: 1시간
- 실시간 데이터: 1분

<br>

# Redis 캐싱 실전 예시

<br>

> ## Cache-Aside 패턴

<br>

```python
import redis
import json

cache = redis.Redis(host='localhost', port=6379, decode_responses=True)

def get_user(user_id):
    cache_key = f"user:{user_id}"

    # 1. 캐시 확인
    cached = cache.get(cache_key)
    if cached:
        return json.loads(cached)

    # 2. DB 조회
    user_data = db.query(f"SELECT * FROM users WHERE id = {user_id}")

    # 3. 캐시 저장 (1시간)
    cache.setex(cache_key, 3600, json.dumps(user_data))

    return user_data
```

<br>

> ## 캐시 무효화

<br>

```python
def update_user(user_id, data):
    # DB 업데이트
    db.update(user_id, data)

    # 캐시 삭제 (중요!)
    cache.delete(f"user:{user_id}")
```

<br>

> ## Cache Stampede 방지

<br>

**문제**: 인기 데이터의 캐시 만료 시 동시 요청이 DB로 몰림 <br><br>

```python
def get_with_lock(key):
    cached = cache.get(key)
    if cached:
        return json.loads(cached)

    # Lock 획득 (10초 타임아웃)
    lock_key = f"lock:{key}"
    if cache.set(lock_key, "1", nx=True, ex=10):
        try:
            data = db.query("SELECT ...")
            cache.setex(key, 300, json.dumps(data))
            return data
        finally:
            cache.delete(lock_key)
    else:
        # Lock 대기
        time.sleep(0.1)
        return get_with_lock(key)
```

<br>

> ## 캐시 Warming

<br>

```python
def warm_up_cache():
    """서버 시작 시 인기 데이터 미리 캐싱"""

    # 인기 게시글
    posts = db.query("SELECT * FROM posts ORDER BY views DESC LIMIT 100")
    for post in posts:
        cache.setex(f"post:{post['id']}", 3600, json.dumps(post))

    # 활성 사용자
    users = db.query("SELECT * FROM users WHERE last_login > NOW() - INTERVAL 1 DAY")
    for user in users:
        cache.setex(f"user:{user['id']}", 7200, json.dumps(user))
```

<br>

> ## 다중 조회 최적화

<br>

```python
def get_multiple_users(user_ids):
    """여러 사용자를 한 번에 조회"""

    keys = [f"user:{uid}" for uid in user_ids]
    cached = cache.mget(keys)  # 한 번에 조회

    result = []
    missing_ids = []

    for i, data in enumerate(cached):
        if data:
            result.append(json.loads(data))
        else:
            missing_ids.append(user_ids[i])

    # 캐시 미스만 DB 조회
    if missing_ids:
        users = db.query(f"SELECT * FROM users WHERE id IN ({','.join(map(str, missing_ids))})")
        for user in users:
            cache.setex(f"user:{user['id']}", 3600, json.dumps(user))
            result.append(user)

    return result
```

<br>

# 캐시 모니터링

<br>

> ## 캐시 히트율 측정

<br>

```python
def calculate_hit_rate():
    info = cache.info('stats')
    hits = info['keyspace_hits']
    misses = info['keyspace_misses']

    hit_rate = (hits / (hits + misses)) * 100
    print(f"캐시 히트율: {hit_rate:.2f}%")

    # 목표: 80% 이상
```

<br>

**목표 히트율**:

- 80% 이상: 우수
- 60~80%: 양호
- 60% 이하: 개선 필요

<br>

> ## 메모리 사용 모니터링

<br>

```python
def check_memory():
    info = cache.info('memory')
    used = info['used_memory_human']
    max_mem = info['maxmemory_human']

    print(f"메모리: {used} / {max_mem}")
```

<br>

# 실전 최적화 기법

<br>

> ## 1. 적절한 TTL 설정

<br>

```python
TTL_CONFIG = {
    'static': 86400,    # 24시간
    'user': 3600,       # 1시간
    'session': 1800,    # 30분
    'trending': 300,    # 5분
    'realtime': 60,     # 1분
}

def cache_with_smart_ttl(key, data, type='static'):
    ttl = TTL_CONFIG.get(type, 3600)
    cache.setex(key, ttl, json.dumps(data))
```

<br>

> ## 2. 명확한 캐시 키 설계

<br>

```python
# ✅ 좋은 예
cache.set("user:123", data)
cache.set("post:456", data)
cache.set("user:123:posts", data)

# ❌ 나쁜 예
cache.set("u123", data)
cache.set("p456", data)
```

<br>

> ## 3. 데이터 압축

<br>

```python
import zlib

def set_compressed(key, data, ttl=3600):
    json_str = json.dumps(data)
    compressed = zlib.compress(json_str.encode())
    cache.setex(key, ttl, compressed)

def get_compressed(key):
    compressed = cache.get(key)
    if not compressed:
        return None
    return json.loads(zlib.decompress(compressed).decode())
```

<br>

> ## 4. 캐시 제거 정책

<br>

```conf
# redis.conf

# LRU: 가장 오래 사용 안 된 키 삭제
maxmemory-policy allkeys-lru

# LFU: 가장 적게 사용된 키 삭제
maxmemory-policy allkeys-lfu
```

<br>
**권장**: `allkeys-lru` <br><br>

# 캐싱 안티패턴

<br>

## ❌ 안티패턴 1: 모든 데이터 캐싱

<br>

```python
# 잘못된 예
for user in all_users:  # 100만 명
    cache.set(f"user:{user['id']}", user)

# ✅ 올바른 예: 자주 조회되는 데이터만
if access_count > 10:
    cache.setex(key, 3600, data)
```

<br>

## ❌ 안티패턴 2: 캐시 무효화 누락

<br>

```python
# 잘못된 예
def update_user(user_id, data):
    db.update(user_id, data)
    # 캐시 무효화 안 함!

# ✅ 올바른 예
def update_user(user_id, data):
    db.update(user_id, data)
    cache.delete(f"user:{user_id}")
```

<br>

## ❌ 안티패턴 3: 너무 큰 데이터

<br>

```python
# 잘못된 예: 10MB 데이터 캐싱
cache.set("huge_data", big_object)

# ✅ 올바른 예: 필요한 필드만
cache.set("summary", {'id': 1, 'name': 'test'})
```

<br>

# 실전 체크리스트

<br>

## ✅ 캐시 설계

<br>

- [ ] 자주 조회되는 데이터만 캐싱
- [ ] 적절한 TTL 설정
- [ ] 명확한 키 네이밍 규칙
- [ ] 데이터 타입별 전략 수립

<br>

## ✅ 캐시 무효화

<br>

- [ ] 데이터 변경 시 캐시 삭제
- [ ] 연관 캐시도 함께 무효화
- [ ] TTL로 자동 만료 보장

<br>

## ✅ 모니터링

<br>

- [ ] 캐시 히트율 80% 이상 유지
- [ ] 메모리 사용률 모니터링
- [ ] 응답 시간 측정

<br>

## ✅ 안정성

<br>

- [ ] Cache Stampede 방지
- [ ] 캐시 장애 시 대응 방안
- [ ] 캐시 Warming 전략

<br>

# 요약

<br>
캐싱은 성능 최적화의 가장 효과적인 방법이다. <br><br>

**💎 핵심 포인트**:

1. **Cache-Aside 패턴**이 가장 일반적
2. **TTL 설정**으로 자동 만료 관리
3. **캐시 무효화**는 반드시 구현
4. **히트율 80% 이상** 목표
5. **Cache Stampede** 반드시 방지
6. **모니터링**으로 지속적 개선

<br>

**🚀 적용 순서**:

1단계: 자주 조회되는 데이터 식별 <br>
2단계: Redis로 Cache-Aside 구현 <br>
3단계: 적절한 TTL 설정 <br>
4단계: 캐시 무효화 로직 추가 <br>
5단계: 히트율 모니터링 <br>
6단계: 최적화 (압축, 다중 조회 등) <br><br>

**⚠️ 주의사항**:

- 모든 데이터를 캐싱하지 말 것
- 캐시 무효화를 잊지 말 것
- 너무 큰 데이터는 캐싱 지양
- 캐시 장애 대비 필수

<br>

**📊 기대 효과**:

좋은 캐싱 전략은 **숫자로 증명**된다. <br>
DB 부하 80% 감소, 응답 시간 10~100배 빠름, <br>
서버 비용 50% 절감 등의 효과를 기대할 수 있다. <br><br>

캐싱은 한 번 구현하면 지속적으로 성능 향상 효과를 제공한다. <br>
데이터 특성에 맞는 전략을 선택하고, <br>
모니터링을 통해 지속적으로 최적화하는 것이 중요하다. <br><br>
