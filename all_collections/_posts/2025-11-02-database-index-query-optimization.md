---
layout: post
title: 🔍 데이터베이스 인덱스와 쿼리 최적화 완벽 가이드
date: 2025-11-02
categories:
  [
    "SamBeak",
    "Database",
    "Index",
    "Query Optimization",
    "Performance",
    "MySQL",
    "PostgreSQL",
  ]
---

# 데이터베이스 인덱스와 쿼리 최적화란 무엇인가

<br>
데이터베이스를 사용하다 보면 점점 느려지는 경험을 하게 된다. <br>
처음엔 빠르던 조회가 데이터가 쌓이면서 <br>
10초, 30초, 심지어 1분 이상 걸리는 상황이 발생한다. <br><br>

이는 마치 책이 정리되지 않은 도서관과 같다. <br>
책이 100권일 때는 하나씩 찾아도 괜찮지만, <br>
10만 권이 되면 원하는 책을 찾기 위해 <br>
모든 책을 일일이 확인해야 한다면 엄청난 시간이 걸린다. <br><br>

데이터베이스 인덱스는 바로 이런 상황에서 <br>
책의 목차나 색인처럼 빠르게 원하는 데이터를 찾을 수 있게 해주는 <br>
핵심 기술이다. <br><br>

> ## 왜 인덱스와 쿼리 최적화가 필요할까?

<br>
최적화되지 않은 데이터베이스는 다음과 같은 문제를 일으킨다. <br><br>

**문제 1: 느린 조회 속도** <br>
인덱스 없이 100만 건의 데이터에서 특정 사용자를 찾으려면 <br>
모든 행을 하나씩 확인해야 한다. (Full Table Scan) <br><br>

**문제 2: 서버 리소스 낭비** <br>
비효율적인 쿼리는 CPU와 메모리를 과도하게 사용한다. <br>
하나의 느린 쿼리가 전체 서버를 느리게 만든다. <br><br>

**문제 3: 동시 처리 능력 저하** <br>
쿼리 하나가 오래 걸리면 다른 요청들도 대기해야 한다. <br>
락(Lock)이 길어지면서 동시성이 떨어진다. <br><br>

**문제 4: 사용자 이탈** <br>
페이지 로딩이 3초 이상 걸리면 사용자 이탈률이 급증한다. <br><br>

# 기본 개념 요약

<br>

## 🏷️ 인덱스(Index)란?

<br>
**개념**: 데이터를 빠르게 찾기 위한 정렬된 자료구조 <br><br>

**책의 색인 비유**: <br>
책 뒤쪽의 색인을 보면 "데이터베이스 - 45쪽, 123쪽"처럼 <br>
단어와 페이지 번호가 정렬되어 있다. <br>
색인이 없다면 처음부터 끝까지 모든 페이지를 뒤져야 하지만, <br>
색인이 있으면 원하는 페이지로 바로 갈 수 있다. <br><br>

### 인덱스의 장단점

<br>

**장점**:

- **조회 속도 대폭 향상**: O(N) → O(log N)
- **정렬과 그룹핑 성능 향상**
- **WHERE, JOIN 조건 빠른 처리**

**단점**:

- **추가 저장 공간 필요**: 테이블 크기의 10-20%
- **INSERT/UPDATE/DELETE 느려짐**: 인덱스도 함께 갱신
- **잘못 설계하면 오히려 성능 저하**

<br>

## 🏷️ 인덱스의 종류

<br>

### 1. B-Tree 인덱스 (가장 일반적)

<br>
**특징**: 균형 잡힌 트리 구조로 정렬된 상태 유지 <br>
**사용**: 대부분의 조회, 정렬, 범위 검색 <br>
**시간 복잡도**: O(log N) <br><br>

**적합한 경우**:

- WHERE id = 100
- WHERE age BETWEEN 20 AND 30
- ORDER BY created_at

**부적합한 경우**:

- WHERE name LIKE '%김%' (앞에 와일드카드)
- 카디널리티가 매우 낮은 컬럼 (성별, 타입 등)

<br>

### 2. Hash 인덱스

<br>
**특징**: 해시 함수로 빠른 검색 (메모리 기반) <br>
**사용**: 동등 비교 (=) <br>
**시간 복잡도**: O(1) <br><br>

**적합한 경우**: WHERE id = 100 <br>
**부적합한 경우**: 범위 검색, 정렬 (BETWEEN, ORDER BY 불가) <br><br>

### 3. Full-Text 인덱스

<br>
**특징**: 텍스트 전문 검색에 특화 <br>
**사용**: 긴 텍스트 검색 <br><br>

**적합한 경우**: 게시글 내용 검색, 키워드 검색 <br><br>

### 4. 복합 인덱스 (Composite Index)

<br>
**특징**: 여러 컬럼을 조합한 인덱스 <br><br>

```sql
CREATE INDEX idx_user_name_age ON users (name, age);
```

<br>
**중요 원칙**: 컬럼 순서가 매우 중요! <br><br>

**활용 가능**:

- WHERE name = '김철수'
- WHERE name = '김철수' AND age = 25

**활용 불가**:

- WHERE age = 25 (첫 번째 컬럼 누락)

<br>

## 🏷️ 쿼리 최적화 핵심 개념

<br>

### 1. 실행 계획 (Execution Plan)

<br>
데이터베이스가 쿼리를 어떻게 실행할지 계획한 것 <br><br>

**확인 방법**: EXPLAIN 명령어 사용 <br><br>

### 2. 카디널리티 (Cardinality)

<br>
컬럼의 고유한 값의 개수 <br><br>

- **높은 카디널리티**: 이메일, 주민번호 (인덱스 효과 ↑)
- **낮은 카디널리티**: 성별, 상태코드 (인덱스 효과 ↓)

<br>

### 3. 선택도 (Selectivity)

<br>
전체 데이터 중 조건에 맞는 비율 <br><br>

- **낮은 선택도**: 전체의 5% 선택 (인덱스 효과 ↑)
- **높은 선택도**: 전체의 80% 선택 (Full Scan이 나을 수도)

<br>

### 4. 커버링 인덱스 (Covering Index)

<br>
쿼리에 필요한 모든 컬럼이 인덱스에 포함되어 <br>
테이블을 조회하지 않아도 되는 상태 <br><br>

```sql
-- 인덱스: (name, age, email)
SELECT name, age, email FROM users WHERE name = '김철수';
-- 테이블 접근 없이 인덱스만으로 조회 완료!
```

<br>

# 인덱스 생성 예시

<br>

> ## MySQL 인덱스 생성

<br>

### 단일 컬럼 인덱스

<br>

```sql
-- 기본 인덱스 생성
CREATE INDEX idx_email ON users (email);

-- 유니크 인덱스
CREATE UNIQUE INDEX idx_unique_email ON users (email);

-- 인덱스 삭제
DROP INDEX idx_email ON users;

-- 인덱스 목록 확인
SHOW INDEX FROM users;
```

<br>

### 복합 인덱스 (컬럼 순서 중요!)

<br>

```sql
-- 이름 + 나이 복합 인덱스
CREATE INDEX idx_name_age ON users (name, age);

-- 활용 가능한 쿼리
SELECT * FROM users WHERE name = '김철수';
SELECT * FROM users WHERE name = '김철수' AND age = 25;
SELECT * FROM users WHERE name = '김철수' AND age > 20;

-- 활용 불가 (첫 번째 컬럼 누락)
SELECT * FROM users WHERE age = 25;
```

<br>

### 인덱스 힌트

<br>

```sql
-- 특정 인덱스 강제 사용
SELECT * FROM users USE INDEX (idx_name_age)
WHERE name = '김철수';

-- 인덱스 무시
SELECT * FROM users IGNORE INDEX (idx_email)
WHERE email = 'test@example.com';
```

<br>

> ## PostgreSQL 인덱스 생성

<br>

### 기본 인덱스

<br>

```sql
-- B-Tree 인덱스 (기본)
CREATE INDEX idx_email ON users (email);

-- 부분 인덱스 (조건부)
CREATE INDEX idx_active_users ON users (email)
WHERE status = 'active';

-- 표현식 인덱스
CREATE INDEX idx_lower_email ON users (LOWER(email));
```

<br>

### 고급 인덱스

<br>

```sql
-- GIN 인덱스 (배열, JSONB 검색)
CREATE INDEX idx_tags ON posts USING GIN (tags);

-- GiST 인덱스 (지리 정보, 범위)
CREATE INDEX idx_location ON stores USING GiST (location);

-- BRIN 인덱스 (대용량 시계열 데이터)
CREATE INDEX idx_created_at ON logs USING BRIN (created_at);
```

<br>

# 쿼리 최적화 실전 예시

<br>

> ## EXPLAIN으로 실행 계획 분석

<br>

### MySQL EXPLAIN

<br>

```sql
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';
```

<br>

**주요 확인 항목**:

| 항목           | 의미                   | 좋은 값         |
| -------------- | ---------------------- | --------------- |
| type           | 조인 타입              | const, eq_ref   |
| possible_keys  | 사용 가능한 인덱스     | -               |
| key            | 실제 사용된 인덱스     | NULL이 아님     |
| rows           | 예상 검색 행 수        | 적을수록 좋음   |
| Extra          | 추가 정보              | Using index     |

<br>

**type 값 해석** (좋은 순서):

- **const**: PRIMARY KEY나 UNIQUE 인덱스로 단일 행 조회 (최고)
- **eq_ref**: 조인 시 PRIMARY KEY 사용
- **ref**: 일반 인덱스 사용
- **range**: 범위 검색 (BETWEEN, >, <)
- **index**: 인덱스 풀 스캔
- **ALL**: 테이블 풀 스캔 (최악)

<br>

### PostgreSQL EXPLAIN ANALYZE

<br>

```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';
```

<br>

**출력 예시**:

```
Index Scan using idx_email on users  (cost=0.42..8.44 rows=1 width=100)
  Index Cond: (email = 'test@example.com'::text)
  Planning Time: 0.123 ms
  Execution Time: 0.045 ms
```

<br>

**주요 확인 항목**:

- **Seq Scan**: 테이블 풀 스캔 (나쁨)
- **Index Scan**: 인덱스 사용 (좋음)
- **Index Only Scan**: 커버링 인덱스 (최고)
- **rows**: 예상 행 수
- **cost**: 예상 비용

<br>

> ## 느린 쿼리 최적화 예시

<br>

### Before: 최적화 전 (느림)

<br>

```sql
-- 인덱스가 없는 상태
SELECT * FROM orders
WHERE customer_id = 12345
AND order_date >= '2024-01-01'
AND status = 'completed';

-- EXPLAIN 결과: type = ALL (테이블 풀 스캔)
-- 실행 시간: 5.2초 (100만 건 기준)
```

<br>

### After: 최적화 후 (빠름)

<br>

```sql
-- 복합 인덱스 생성
CREATE INDEX idx_orders_optimization
ON orders (customer_id, order_date, status);

-- 같은 쿼리 실행
SELECT * FROM orders
WHERE customer_id = 12345
AND order_date >= '2024-01-01'
AND status = 'completed';

-- EXPLAIN 결과: type = ref (인덱스 사용)
-- 실행 시간: 0.05초 (104배 빠름!)
```

<br>

### 커버링 인덱스로 추가 최적화

<br>

```sql
-- 필요한 컬럼만 조회
SELECT order_id, customer_id, order_date, status, total_amount
FROM orders
WHERE customer_id = 12345
AND order_date >= '2024-01-01'
AND status = 'completed';

-- 커버링 인덱스 생성
CREATE INDEX idx_orders_covering
ON orders (customer_id, order_date, status, order_id, total_amount);

-- EXPLAIN 결과: Extra = Using index (테이블 접근 없음)
-- 실행 시간: 0.02초 (추가로 2.5배 빠름!)
```

<br>

> ## N+1 문제 해결

<br>

### Before: N+1 문제 발생

<br>

```python
# 게시글 목록 조회 (1번)
posts = db.query("SELECT * FROM posts LIMIT 10")

# 각 게시글의 작성자 조회 (N번)
for post in posts:
    author = db.query(f"SELECT * FROM users WHERE id = {post.author_id}")
    post.author_name = author.name

# 총 쿼리 수: 1 + 10 = 11번
```

<br>

### After: JOIN으로 해결

<br>

```sql
-- 한 번의 쿼리로 해결
SELECT
    p.*,
    u.name as author_name,
    u.email as author_email
FROM posts p
INNER JOIN users u ON p.author_id = u.id
LIMIT 10;

-- 총 쿼리 수: 1번 (11배 감소!)
```

<br>

### 대안: IN 절 사용

<br>

```python
# 게시글 목록 조회 (1번)
posts = db.query("SELECT * FROM posts LIMIT 10")
author_ids = [post.author_id for post in posts]

# 작성자 일괄 조회 (1번)
authors = db.query(f"SELECT * FROM users WHERE id IN ({','.join(map(str, author_ids))})")
author_map = {author.id: author for author in authors}

# 메모리에서 매핑
for post in posts:
    post.author = author_map[post.author_id]

# 총 쿼리 수: 2번 (5.5배 감소)
```

<br>

> ## 페이징 최적화

<br>

### Before: OFFSET 사용 (느림)

<br>

```sql
-- 10만 번째 페이지 조회 (매우 느림)
SELECT * FROM posts
ORDER BY id
LIMIT 20 OFFSET 100000;

-- 문제: 10만 건을 읽고 버려야 함
-- 실행 시간: 2.3초
```

<br>

### After: Cursor 기반 페이징 (빠름)

<br>

```sql
-- 첫 페이지
SELECT * FROM posts
ORDER BY id
LIMIT 20;
-- 마지막 id: 20

-- 다음 페이지 (WHERE 조건 사용)
SELECT * FROM posts
WHERE id > 20
ORDER BY id
LIMIT 20;

-- 실행 시간: 0.01초 (230배 빠름!)
```

<br>

# 쿼리 최적화 안티패턴과 해결책

<br>

> ## 안티패턴 1: SELECT *

<br>

### 문제

<br>

```sql
-- 필요 없는 컬럼까지 모두 조회
SELECT * FROM users WHERE id = 100;
```

<br>

### 해결

<br>

```sql
-- 필요한 컬럼만 조회
SELECT id, name, email FROM users WHERE id = 100;
```

<br>

**이유**: 네트워크 전송량 감소, 커버링 인덱스 활용 가능 <br><br>

> ## 안티패턴 2: 함수 사용으로 인덱스 무효화

<br>

### 문제

<br>

```sql
-- 인덱스를 사용할 수 없음
SELECT * FROM users WHERE YEAR(created_at) = 2024;
SELECT * FROM users WHERE LOWER(email) = 'test@example.com';
```

<br>

### 해결

<br>

```sql
-- 범위 검색으로 변경 (인덱스 활용)
SELECT * FROM users
WHERE created_at >= '2024-01-01'
AND created_at < '2025-01-01';

-- 데이터를 소문자로 저장하거나 함수 인덱스 생성
CREATE INDEX idx_lower_email ON users (LOWER(email));
```

<br>

> ## 안티패턴 3: OR 조건 남용

<br>

### 문제

<br>

```sql
-- 인덱스 활용 어려움
SELECT * FROM users
WHERE name = '김철수' OR email = 'kim@example.com';
```

<br>

### 해결

<br>

```sql
-- UNION으로 분리 (각각 인덱스 사용)
SELECT * FROM users WHERE name = '김철수'
UNION
SELECT * FROM users WHERE email = 'kim@example.com';
```

<br>

> ## 안티패턴 4: 와일드카드 앞쪽 사용

<br>

### 문제

<br>

```sql
-- 인덱스를 사용할 수 없음
SELECT * FROM users WHERE name LIKE '%철수%';
```

<br>

### 해결

<br>

```sql
-- 와일드카드를 뒤쪽에만 사용
SELECT * FROM users WHERE name LIKE '김철수%';

-- 또는 Full-Text 인덱스 사용
CREATE FULLTEXT INDEX idx_name_fulltext ON users (name);
SELECT * FROM users WHERE MATCH(name) AGAINST('철수');
```

<br>

> ## 안티패턴 5: 서브쿼리 남용

<br>

### 문제

<br>

```sql
-- 서브쿼리가 매 행마다 실행됨
SELECT *,
    (SELECT COUNT(*) FROM orders WHERE customer_id = u.id) as order_count
FROM users u;
```

<br>

### 해결

<br>

```sql
-- JOIN으로 변경
SELECT u.*, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.customer_id
GROUP BY u.id;
```

<br>

# 실전 모니터링과 분석

<br>

> ## 느린 쿼리 로그 활성화

<br>

### MySQL

<br>

```sql
-- 느린 쿼리 로그 활성화
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;  -- 1초 이상 걸리는 쿼리 기록

-- 로그 파일 위치 확인
SHOW VARIABLES LIKE 'slow_query_log_file';
```

<br>

### PostgreSQL

<br>

```sql
-- postgresql.conf 설정
-- log_min_duration_statement = 1000  (1초)

-- 실행 시간이 긴 쿼리 확인
SELECT query, calls, total_time, mean_time
FROM pg_stat_statements
ORDER BY total_time DESC
LIMIT 10;
```

<br>

> ## 인덱스 사용률 확인

<br>

### MySQL

<br>

```sql
-- 사용되지 않는 인덱스 찾기
SELECT
    t.TABLE_SCHEMA,
    t.TABLE_NAME,
    s.INDEX_NAME,
    s.COLUMN_NAME
FROM information_schema.TABLES t
INNER JOIN information_schema.STATISTICS s
    ON t.TABLE_SCHEMA = s.TABLE_SCHEMA
    AND t.TABLE_NAME = s.TABLE_NAME
LEFT JOIN performance_schema.table_io_waits_summary_by_index_usage i
    ON i.OBJECT_SCHEMA = s.TABLE_SCHEMA
    AND i.OBJECT_NAME = s.TABLE_NAME
    AND i.INDEX_NAME = s.INDEX_NAME
WHERE t.TABLE_SCHEMA NOT IN ('mysql', 'information_schema', 'performance_schema')
AND i.INDEX_NAME IS NULL;
```

<br>

### PostgreSQL

<br>

```sql
-- 인덱스 사용 통계
SELECT
    schemaname,
    tablename,
    indexname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch
FROM pg_stat_user_indexes
ORDER BY idx_scan;
```

<br>

# 실전 최적화 체크리스트

<br>

## ✅ 인덱스 설계

<br>

- [ ] WHERE 절에 자주 사용되는 컬럼에 인덱스 생성
- [ ] JOIN 조건 컬럼에 인덱스 생성
- [ ] ORDER BY 컬럼에 인덱스 고려
- [ ] 복합 인덱스 컬럼 순서 최적화 (카디널리티 높은 순)
- [ ] 커버링 인덱스 활용
- [ ] 사용하지 않는 인덱스 제거

<br>

## ✅ 쿼리 작성

<br>

- [ ] SELECT * 대신 필요한 컬럼만 조회
- [ ] N+1 문제 해결 (JOIN 또는 IN 사용)
- [ ] WHERE 절에서 함수 사용 지양
- [ ] OR 대신 UNION 고려
- [ ] LIKE '%keyword%' 지양
- [ ] 서브쿼리 대신 JOIN 고려

<br>

## ✅ 성능 분석

<br>

- [ ] EXPLAIN으로 실행 계획 확인
- [ ] 느린 쿼리 로그 모니터링
- [ ] 인덱스 사용률 확인
- [ ] 쿼리 실행 시간 측정

<br>

## ✅ 데이터베이스 설정

<br>

- [ ] 커넥션 풀 크기 최적화
- [ ] 쿼리 캐시 활용 (MySQL 5.7 이하)
- [ ] 버퍼 풀 크기 조정
- [ ] 통계 정보 주기적 갱신

<br>

# 요약

<br>
데이터베이스 성능의 90%는 인덱스와 쿼리 최적화에서 결정된다. <br><br>

**💎 핵심 포인트**:

1. **인덱스는 책의 색인**과 같다 - 빠른 검색의 핵심
2. **B-Tree 인덱스**가 가장 일반적이고 효과적
3. **복합 인덱스는 컬럼 순서**가 매우 중요
4. **EXPLAIN으로 실행 계획** 반드시 확인
5. **커버링 인덱스**로 테이블 접근 제거
6. **N+1 문제**는 JOIN이나 IN으로 해결

<br>

**🚀 최적화 우선순위**:

1단계: EXPLAIN으로 느린 쿼리 찾기 <br>
2단계: WHERE/JOIN 컬럼에 인덱스 생성 <br>
3단계: 복합 인덱스로 커버링 인덱스 구성 <br>
4단계: N+1 문제 제거 <br>
5단계: 쿼리 리팩토링 (함수 제거, OR 제거) <br><br>

**⚠️ 주의사항**:

- 인덱스는 만능이 아니다 (쓰기 성능 저하)
- 모든 컬럼에 인덱스를 만들지 말 것
- 정기적으로 사용하지 않는 인덱스 제거
- 통계 정보를 주기적으로 갱신

<br>

**📊 성과 측정**:

좋은 최적화는 **숫자로 증명**된다. <br>
Before/After 실행 시간을 반드시 측정하고 <br>
10배 이상의 성능 향상을 목표로 하자. <br><br>

데이터베이스 최적화는 한 번에 끝나는 작업이 아니다. <br>
데이터가 쌓이고 트래픽이 증가하면서 <br>
계속해서 모니터링하고 개선해야 하는 **지속적인 과정**이다. <br><br>
