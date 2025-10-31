---
layout: post
title: ⏰ 시계열 데이터 파티셔닝 완벽 가이드
date: 2025-10-31
categories:
  ["SamBeak", "Database", "Partitioning", "Time Series", "PostgreSQL", "MySQL"]
---

# 시계열 데이터 파티셔닝이란 무엇인가

<br>
데이터베이스에 데이터가 쌓이다 보면 어느 순간 느려지는 경험을 한 적이 있을 것이다. <br>
특히 시간 순서대로 계속 쌓이는 데이터, 즉 시계열 데이터는 <br>
로그, 센서 데이터, 거래 내역처럼 끊임없이 증가한다. <br><br>

이를 비유하자면 하나의 큰 서랍장에 계속 물건을 넣는 것과 같다. <br>
처음엔 괜찮지만 물건이 많아지면 <br>
원하는 물건을 찾기 위해 서랍 전체를 뒤져야 한다. <br>
하지만 물건을 "날짜별로" 여러 서랍에 나눠 담으면 <br>
"지난주 것"만 찾으면 되니 훨씬 빠르다. <br><br>

시계열 데이터 파티셔닝은 바로 이렇게 <br>
큰 테이블을 시간 단위로 나누어 관리하는 기법이다. <br><br>

> ## 왜 파티셔닝이 필요할까?

<br>
데이터가 많아지면 다음과 같은 문제가 발생한다. <br><br>

**문제 1: 느린 조회 속도** <br>
최근 1주일 데이터만 보고 싶은데 <br>
5년치 데이터를 모두 스캔해야 한다. <br><br>

**문제 2: 비효율적인 백업** <br>
오래된 데이터까지 매번 백업하느라 시간이 오래 걸린다. <br><br>

**문제 3: 삭제의 어려움** <br>
오래된 데이터를 삭제하려면 <br>
DELETE 쿼리로 한 건씩 지워야 해서 느리다. <br><br>

**문제 4: 인덱스 비대화** <br>
인덱스도 커져서 메모리에 올리기 어렵다. <br><br>

# 기본 개념 요약

<br>

> ## 파티셔닝이란?

<br>
파티셔닝은 **하나의 큰 테이블을 여러 개의 작은 테이블로 나누는 것**이다. <br>
물리적으로는 여러 테이블이지만, 논리적으로는 하나의 테이블처럼 사용한다. <br><br>

## 🏷️ 파티셔닝의 종류

<br>

### 1. 범위 파티셔닝 (Range Partitioning)

<br>
특정 값의 범위로 나눈다. <br>
시계열 데이터에 가장 많이 사용된다. <br><br>

**예시:**

- 2024년 1월 데이터 → partition_2024_01
- 2024년 2월 데이터 → partition_2024_02
- 2024년 3월 데이터 → partition_2024_03

<br>

### 2. 리스트 파티셔닝 (List Partitioning)

<br>
특정 값의 목록으로 나눈다. <br><br>

**예시:**

- 서울 지역 데이터 → partition_seoul
- 부산 지역 데이터 → partition_busan

<br>

### 3. 해시 파티셔닝 (Hash Partitioning)

<br>
해시 함수를 사용하여 균등하게 분산한다. <br>
데이터 분포가 고르지 않을 때 유용하다. <br><br>

## 🏷️ 파티셔닝의 장점

<br>

**🟢 조회 성능 향상** <br>
필요한 파티션만 스캔하므로 속도가 빠르다. (Partition Pruning) <br><br>

**🟢 효율적인 데이터 관리** <br>
오래된 파티션만 삭제하거나 아카이빙할 수 있다. <br><br>

**🟢 병렬 처리** <br>
여러 파티션을 동시에 처리할 수 있다. <br><br>

**🟢 유지보수 용이** <br>
파티션별로 백업, 복구, 인덱스 재구성이 가능하다. <br><br>

## 🏷️ 파티셔닝의 단점

<br>

**🔴 복잡도 증가** <br>
파티션 관리를 위한 추가 작업이 필요하다. <br><br>

**🔴 파티션 키 변경 어려움** <br>
한 번 정한 파티션 키는 변경하기 어렵다. <br><br>

**🔴 조인 성능 저하 가능** <br>
파티션 키가 아닌 조건으로 조인하면 느려질 수 있다. <br><br>

# PostgreSQL 파티셔닝 예시

<br>
PostgreSQL은 10 버전부터 네이티브 파티셔닝을 지원한다. <br><br>

> ## 월별 파티셔닝 예시

<br>

### 1. 부모 테이블 생성

<br>

```sql
-- 부모 테이블 생성 (실제 데이터는 저장되지 않음)
CREATE TABLE sensor_data (
    id BIGSERIAL,
    sensor_id INT NOT NULL,
    temperature DECIMAL(5,2),
    humidity DECIMAL(5,2),
    measured_at TIMESTAMP NOT NULL,
    PRIMARY KEY (id, measured_at)
) PARTITION BY RANGE (measured_at);
```

<br>
중요한 점은 **PRIMARY KEY에 파티션 키(measured_at)가 포함**되어야 한다는 것이다. <br><br>

### 2. 파티션 테이블 생성

<br>

```sql
-- 2024년 1월 파티션
CREATE TABLE sensor_data_2024_01 PARTITION OF sensor_data
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

-- 2024년 2월 파티션
CREATE TABLE sensor_data_2024_02 PARTITION OF sensor_data
    FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');

-- 2024년 3월 파티션
CREATE TABLE sensor_data_2024_03 PARTITION OF sensor_data
    FOR VALUES FROM ('2024-03-01') TO ('2024-04-01');

-- 기본 파티션 (범위를 벗어난 데이터용)
CREATE TABLE sensor_data_default PARTITION OF sensor_data DEFAULT;
```

<br>

### 3. 인덱스 생성

<br>

```sql
-- 부모 테이블에 인덱스를 생성하면 모든 파티션에 자동 적용
CREATE INDEX idx_sensor_data_sensor_id ON sensor_data (sensor_id);
CREATE INDEX idx_sensor_data_measured_at ON sensor_data (measured_at);
```

<br>

### 4. 데이터 삽입

<br>

```sql
-- 부모 테이블에 INSERT하면 자동으로 적절한 파티션에 들어감
INSERT INTO sensor_data (sensor_id, temperature, humidity, measured_at)
VALUES
    (1, 23.5, 65.2, '2024-01-15 10:30:00'),
    (2, 24.1, 62.8, '2024-02-20 14:45:00'),
    (3, 22.8, 68.5, '2024-03-10 09:15:00');
```

<br>

### 5. 데이터 조회

<br>

```sql
-- 특정 기간 조회 (자동으로 해당 파티션만 스캔)
SELECT * FROM sensor_data
WHERE measured_at BETWEEN '2024-02-01' AND '2024-02-28';

-- 실행 계획 확인
EXPLAIN SELECT * FROM sensor_data
WHERE measured_at BETWEEN '2024-02-01' AND '2024-02-28';
```

<br>

### 6. 오래된 파티션 삭제

<br>

```sql
-- 2024년 1월 데이터 전체 삭제 (매우 빠름)
DROP TABLE sensor_data_2024_01;

-- 또는 파티션을 분리하여 아카이빙
ALTER TABLE sensor_data DETACH PARTITION sensor_data_2024_01;
-- 이제 sensor_data_2024_01은 독립된 일반 테이블이 됨
```

<br>

### 7. 자동 파티션 생성 함수

<br>

```sql
-- 자동으로 다음 달 파티션을 생성하는 함수
CREATE OR REPLACE FUNCTION create_monthly_partition()
RETURNS void AS $$
DECLARE
    partition_date DATE;
    partition_name TEXT;
    start_date TEXT;
    end_date TEXT;
BEGIN
    -- 다음 달 파티션 생성
    partition_date := DATE_TRUNC('month', CURRENT_DATE + INTERVAL '1 month');
    partition_name := 'sensor_data_' || TO_CHAR(partition_date, 'YYYY_MM');
    start_date := TO_CHAR(partition_date, 'YYYY-MM-DD');
    end_date := TO_CHAR(partition_date + INTERVAL '1 month', 'YYYY-MM-DD');

    -- 파티션이 없으면 생성
    IF NOT EXISTS (
        SELECT 1 FROM pg_tables WHERE tablename = partition_name
    ) THEN
        EXECUTE format(
            'CREATE TABLE %I PARTITION OF sensor_data FOR VALUES FROM (%L) TO (%L)',
            partition_name, start_date, end_date
        );
        RAISE NOTICE '파티션 생성됨: %', partition_name;
    END IF;
END;
$$ LANGUAGE plpgsql;

-- 매달 1일에 자동 실행되도록 스케줄링 (pg_cron 확장 필요)
-- SELECT cron.schedule('0 0 1 * *', 'SELECT create_monthly_partition()');
```

<br>

# MySQL 파티셔닝 예시

<br>
MySQL은 5.1 버전부터 파티셔닝을 지원한다. <br><br>

> ## 월별 파티셔닝 예시

<br>

### 1. 테이블 생성과 동시에 파티셔닝

<br>

```sql
-- 범위 파티셔닝으로 월별 분할
CREATE TABLE sensor_data (
    id BIGINT AUTO_INCREMENT,
    sensor_id INT NOT NULL,
    temperature DECIMAL(5,2),
    humidity DECIMAL(5,2),
    measured_at DATETIME NOT NULL,
    PRIMARY KEY (id, measured_at),
    KEY idx_sensor_id (sensor_id)
)
PARTITION BY RANGE (YEAR(measured_at) * 100 + MONTH(measured_at)) (
    PARTITION p_2024_01 VALUES LESS THAN (202402),
    PARTITION p_2024_02 VALUES LESS THAN (202403),
    PARTITION p_2024_03 VALUES LESS THAN (202404),
    PARTITION p_2024_04 VALUES LESS THAN (202405),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);
```

<br>
MySQL에서는 PRIMARY KEY에 파티션 키가 포함되어야 한다. <br><br>

### 2. 기존 테이블을 파티셔닝 테이블로 변경

<br>

```sql
-- 기존 테이블에 파티셔닝 추가
ALTER TABLE sensor_data
PARTITION BY RANGE (YEAR(measured_at) * 100 + MONTH(measured_at)) (
    PARTITION p_2024_01 VALUES LESS THAN (202402),
    PARTITION p_2024_02 VALUES LESS THAN (202403),
    PARTITION p_2024_03 VALUES LESS THAN (202404)
);
```

<br>

### 3. 데이터 삽입

<br>

```sql
-- 자동으로 적절한 파티션에 삽입됨
INSERT INTO sensor_data (sensor_id, temperature, humidity, measured_at)
VALUES
    (1, 23.5, 65.2, '2024-01-15 10:30:00'),
    (2, 24.1, 62.8, '2024-02-20 14:45:00'),
    (3, 22.8, 68.5, '2024-03-10 09:15:00');
```

<br>

### 4. 파티션 정보 조회

<br>

```sql
-- 파티션 목록 확인
SELECT
    PARTITION_NAME,
    PARTITION_METHOD,
    PARTITION_EXPRESSION,
    TABLE_ROWS
FROM INFORMATION_SCHEMA.PARTITIONS
WHERE TABLE_NAME = 'sensor_data'
ORDER BY PARTITION_ORDINAL_POSITION;
```

<br>

### 5. 파티션 추가

<br>

```sql
-- 2024년 5월 파티션 추가
ALTER TABLE sensor_data
ADD PARTITION (
    PARTITION p_2024_05 VALUES LESS THAN (202406)
);
```

<br>

### 6. 파티션 재구성 (MAXVALUE 파티션이 있을 때)

<br>

```sql
-- MAXVALUE 파티션을 분할하여 새 파티션 추가
ALTER TABLE sensor_data
REORGANIZE PARTITION p_future INTO (
    PARTITION p_2024_05 VALUES LESS THAN (202406),
    PARTITION p_2024_06 VALUES LESS THAN (202407),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);
```

<br>

### 7. 오래된 파티션 삭제

<br>

```sql
-- 2024년 1월 파티션 삭제 (매우 빠름)
ALTER TABLE sensor_data DROP PARTITION p_2024_01;
```

<br>

### 8. 파티션 자동 관리 프로시저

<br>

```sql
DELIMITER $$

CREATE PROCEDURE manage_partitions()
BEGIN
    DECLARE next_month INT;
    DECLARE partition_name VARCHAR(20);

    -- 다음 달 계산
    SET next_month = (YEAR(DATE_ADD(CURRENT_DATE, INTERVAL 2 MONTH)) * 100) +
                     MONTH(DATE_ADD(CURRENT_DATE, INTERVAL 2 MONTH));
    SET partition_name = CONCAT('p_', YEAR(DATE_ADD(CURRENT_DATE, INTERVAL 2 MONTH)),
                                '_', LPAD(MONTH(DATE_ADD(CURRENT_DATE, INTERVAL 2 MONTH)), 2, '0'));

    -- 파티션 추가 (p_future 재구성)
    SET @sql = CONCAT('ALTER TABLE sensor_data REORGANIZE PARTITION p_future INTO (',
                      'PARTITION ', partition_name, ' VALUES LESS THAN (', next_month + 1, '),',
                      'PARTITION p_future VALUES LESS THAN MAXVALUE)');

    PREPARE stmt FROM @sql;
    EXECUTE stmt;
    DEALLOCATE PREPARE stmt;

    -- 3개월 이전 파티션 삭제
    SET @old_partition = CONCAT('p_', YEAR(DATE_SUB(CURRENT_DATE, INTERVAL 3 MONTH)),
                                '_', LPAD(MONTH(DATE_SUB(CURRENT_DATE, INTERVAL 3 MONTH)), 2, '0'));

    -- 파티션 존재 여부 확인 후 삭제
    IF EXISTS (SELECT 1 FROM INFORMATION_SCHEMA.PARTITIONS
               WHERE TABLE_NAME = 'sensor_data' AND PARTITION_NAME = @old_partition) THEN
        SET @sql = CONCAT('ALTER TABLE sensor_data DROP PARTITION ', @old_partition);
        PREPARE stmt FROM @sql;
        EXECUTE stmt;
        DEALLOCATE PREPARE stmt;
    END IF;
END$$

DELIMITER ;

-- 매달 1일 자정에 실행 (MySQL Event Scheduler 사용)
CREATE EVENT monthly_partition_management
ON SCHEDULE EVERY 1 MONTH
STARTS CONCAT(DATE_FORMAT(CURRENT_DATE + INTERVAL 1 MONTH, '%Y-%m-'), '01 00:00:00')
DO CALL manage_partitions();

-- Event Scheduler 활성화
SET GLOBAL event_scheduler = ON;
```

<br>

# 실전 팁: 일별 파티셔닝

<br>
데이터가 매우 많거나 매일 삭제해야 하는 경우 일별 파티셔닝을 사용한다. <br><br>

## PostgreSQL 일별 파티셔닝

<br>

```sql
-- 일별 파티션 예시
CREATE TABLE access_log (
    id BIGSERIAL,
    user_id INT NOT NULL,
    url TEXT,
    accessed_at TIMESTAMP NOT NULL,
    PRIMARY KEY (id, accessed_at)
) PARTITION BY RANGE (accessed_at);

-- 2024년 10월 30일 파티션
CREATE TABLE access_log_2024_10_30 PARTITION OF access_log
    FOR VALUES FROM ('2024-10-30') TO ('2024-10-31');

-- 2024년 10월 31일 파티션
CREATE TABLE access_log_2024_10_31 PARTITION OF access_log
    FOR VALUES FROM ('2024-10-31') TO ('2024-11-01');
```

<br>

## MySQL 일별 파티셔닝

<br>

```sql
-- 일별 파티션 예시 (TO_DAYS 함수 사용)
CREATE TABLE access_log (
    id BIGINT AUTO_INCREMENT,
    user_id INT NOT NULL,
    url TEXT,
    accessed_at DATETIME NOT NULL,
    PRIMARY KEY (id, accessed_at)
)
PARTITION BY RANGE (TO_DAYS(accessed_at)) (
    PARTITION p_20241030 VALUES LESS THAN (TO_DAYS('2024-10-31')),
    PARTITION p_20241031 VALUES LESS THAN (TO_DAYS('2024-11-01')),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);
```

<br>

# 운영 시 주의할 점

<br>

> ## 1. 파티션 키 선택이 가장 중요하다

<br>
**시계열 데이터는 시간 컬럼을 파티션 키로 사용**해야 한다. <br>
가장 많이 사용하는 조회 패턴에 맞춰 선택하자. <br><br>

- 로그 데이터: 일별 또는 시간별 파티셔닝 <br>
- 거래 데이터: 월별 또는 분기별 파티셔닝 <br>
- 센서 데이터: 일별 또는 주별 파티셔닝 <br><br>

> ## 2. 파티션 크기를 적절하게 유지하라

<br>
**너무 많은 파티션은 오히려 성능을 저하**시킨다. <br><br>

❌ **나쁜 예**: 시간별 파티셔닝으로 1년에 8,760개 파티션 생성 <br>
✅ **좋은 예**: 일별 또는 월별 파티셔닝으로 12~365개 파티션 유지 <br><br>

**권장 파티션 수**: 100개 이하 <br><br>

> ## 3. 자동 파티션 관리를 구축하라

<br>
매달 수동으로 파티션을 만들면 실수할 수 있다. <br>
**자동화 스크립트나 프로시저**를 만들어 두자. <br><br>

- 미래 파티션 미리 생성 (최소 1~2달 치) <br>
- 오래된 파티션 자동 삭제 또는 아카이빙 <br>
- 파티션 생성 실패 시 알림 설정 <br><br>

> ## 4. PRIMARY KEY에 파티션 키를 포함하라

<br>
PostgreSQL과 MySQL 모두 **PRIMARY KEY에 파티션 키가 포함**되어야 한다. <br><br>

```sql
-- ❌ 잘못된 예
PRIMARY KEY (id)  -- 파티션 키 없음

-- ✅ 올바른 예
PRIMARY KEY (id, measured_at)  -- 파티션 키 포함
```

<br>

> ## 5. 파티션 프루닝을 확인하라

<br>
쿼리가 실제로 필요한 파티션만 스캔하는지 **실행 계획(EXPLAIN)**을 확인하자. <br><br>

```sql
-- PostgreSQL
EXPLAIN SELECT * FROM sensor_data
WHERE measured_at >= '2024-02-01' AND measured_at < '2024-03-01';

-- MySQL
EXPLAIN PARTITIONS SELECT * FROM sensor_data
WHERE measured_at >= '2024-02-01' AND measured_at < '2024-03-01';
```

<br>
결과에서 어떤 파티션을 스캔하는지 확인해야 한다. <br><br>

> ## 6. 백업 전략을 수립하라

<br>
파티션별로 백업하면 더 효율적이다. <br><br>

- 오래된 파티션: 한 번만 백업 후 보관 <br>
- 최근 파티션: 자주 백업 <br>
- 아카이빙: 오래된 파티션은 별도 저장소로 이동 <br><br>

> ## 7. 파티션 테이블의 제약사항을 이해하라

<br>

**PostgreSQL 제약사항:**

- UNIQUE 제약조건에도 파티션 키 포함 필요 <br>
- FOREIGN KEY 제약조건 제한적 지원 <br><br>

**MySQL 제약사항:**

- 외래 키(Foreign Key) 지원 안 됨 <br>
- 전문 검색(Fulltext) 인덱스 지원 안 됨 <br>
- 파티션 키는 INTEGER나 함수 반환값이어야 함 <br><br>

> ## 8. 모니터링과 알림을 설정하라

<br>
다음 항목을 모니터링해야 한다. <br><br>

- 파티션별 데이터 크기 <br>
- 파티션 생성 실패 여부 <br>
- 디스크 공간 부족 <br>
- 느린 쿼리 (파티션 프루닝 실패) <br><br>

# 요약

<br>
시계열 데이터 파티셔닝은 대용량 시계열 데이터를 효율적으로 관리하는 핵심 기법이다. <br><br>

## 💎 핵심 포인트

<br>

**1. 파티셔닝은 큰 테이블을 시간 단위로 나누는 것** <br>
마치 서랍장을 날짜별로 나누는 것과 같다. <br><br>

**2. 조회 성능이 획기적으로 향상된다** <br>
전체가 아닌 필요한 파티션만 스캔한다. <br><br>

**3. PostgreSQL과 MySQL 모두 네이티브 파티셔닝 지원** <br>
PostgreSQL 10+, MySQL 5.1+ 버전부터 사용 가능하다. <br><br>

**4. 자동화가 필수다** <br>
파티션 생성과 삭제를 자동화하지 않으면 운영이 어렵다. <br><br>

**5. 적절한 파티션 크기 유지** <br>
너무 많지도, 적지도 않게 (권장: 100개 이하) <br><br>

## 🚀 실전 체크리스트

<br>

✅ 파티션 키는 가장 많이 조회하는 시간 컬럼으로 선택 <br>
✅ PRIMARY KEY에 파티션 키 포함 <br>
✅ 미래 파티션을 미리 생성하는 자동화 구축 <br>
✅ 오래된 파티션 삭제/아카이빙 자동화 <br>
✅ EXPLAIN으로 파티션 프루닝 확인 <br>
✅ 파티션별 백업 전략 수립 <br>
✅ 모니터링과 알림 설정 <br><br>

시계열 데이터를 다루는 시스템을 구축한다면 <br>
파티셔닝은 선택이 아닌 필수다. <br>
처음엔 복잡해 보이지만, 한 번 구축해두면 <br>
장기적으로 엄청난 성능 향상과 관리 효율을 얻을 수 있다. <br><br>
