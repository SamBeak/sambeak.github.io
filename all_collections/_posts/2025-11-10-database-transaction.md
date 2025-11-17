---
layout: post
title: 🔒 데이터베이스 트랜잭션과 동시성 제어 완벽 가이드 (ACID, 격리 수준, 락)
date: 2025-11-10
categories:
  [
    "SamBeak",
    "Database",
    "Transaction",
    "Concurrency",
    "ACID",
    "Lock",
  ]
---

# 트랜잭션이란 무엇인가

<br>
은행에서 계좌 이체를 한다고 상상해보자. <br>
A의 계좌에서 돈이 빠져나가고, <br>
B의 계좌로 돈이 들어가야 한다. <br><br>

만약 중간에 오류가 발생하면? <br>
A의 돈만 빠지고 B는 못 받는 상황 발생! <br><br>

**트랜잭션**은 이런 문제를 방지한다. <br>
"모두 성공하거나, 모두 실패하거나" <br>
중간 상태는 절대 허용하지 않는다. <br><br>

> ## 왜 트랜잭션을 배워야 할까?

<br>

**이유 1: 데이터 무결성** <br>
돈이 증발하거나 복제되는 것을 방지 <br><br>

**이유 2: 동시성 문제 해결** <br>
여러 사용자가 동시에 접근할 때 충돌 방지 <br><br>

**이유 3: 실무 필수** <br>
금융, 커머스 등 모든 서비스에서 사용 <br><br>

**이유 4: 면접 단골** <br>
ACID, 격리 수준, 락은 면접 단골 질문 <br><br>

# 기본 개념 요약

<br>

## 🏷️ ACID 속성

<br>

### 1. Atomicity (원자성)

<br>
**개념**: All or Nothing <br><br>

```sql
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 100000 WHERE id = 1;
UPDATE accounts SET balance = balance + 100000 WHERE id = 2;
COMMIT; -- 모두 성공
-- 또는 ROLLBACK; -- 모두 취소
```

<br>

### 2. Consistency (일관성)

<br>
**개념**: 트랜잭션 전후 데이터베이스는 일관된 상태 유지 <br><br>

```
이체 전: A=100만원, B=50만원 (총 150만원)
이체 후: A=90만원, B=60만원 (총 150만원) ✅
```

<br>

### 3. Isolation (격리성)

<br>
**개념**: 동시 실행 트랜잭션은 서로 영향 안 줌 <br><br>

### 4. Durability (지속성)

<br>
**개념**: 커밋된 트랜잭션은 영구 저장 <br><br>

## 🏷️ 트랜잭션 격리 수준

<br>

### 동시성 문제 3가지

<br>

**1. Dirty Read**: 커밋 안 된 데이터 읽기 <br>
**2. Non-repeatable Read**: 같은 데이터 조회 시 다른 값 <br>
**3. Phantom Read**: 같은 조건 조회 시 다른 결과 개수 <br><br>

### 격리 수준 비교

<br>

| 격리 수준 | Dirty Read | Non-repeatable | Phantom Read |
|-----------|------------|----------------|--------------|
| **READ UNCOMMITTED** | ⭕ | ⭕ | ⭕ |
| **READ COMMITTED** | ❌ | ⭕ | ⭕ |
| **REPEATABLE READ** | ❌ | ❌ | ⭕ |
| **SERIALIZABLE** | ❌ | ❌ | ❌ |

<br>

**권장**: READ COMMITTED (성능과 일관성 균형)

<br>

## 🏷️ 락(Lock) 메커니즘

<br>

### 락 종류

<br>

**Shared Lock (S-Lock)**: 읽기 전용, 여러 트랜잭션 공유 가능 <br>
**Exclusive Lock (X-Lock)**: 읽기/쓰기, 단독 점유 <br><br>

### 비관적 락 vs 낙관적 락

<br>

| 구분 | 비관적 락 | 낙관적 락 |
|------|-----------|-----------|
| **락 시점** | 읽기 시작 | 커밋 시점 |
| **충돌 전략** | 대기 | 재시도 |
| **동시성** | 낮음 | 높음 |
| **사용 예시** | 재고 관리 | 게시글 수정 |

<br>

# 실전 예시

<br>

## 🏷️ Spring @Transactional

<br>

```java
@Service
public class AccountService {
    
    @Transactional
    public void transfer(Long fromId, Long toId, BigDecimal amount) {
        Account from = accountRepository.findById(fromId).orElseThrow();
        Account to = accountRepository.findById(toId).orElseThrow();
        
        if (from.getBalance().compareTo(amount) < 0) {
            throw new IllegalArgumentException("잔액 부족");
        }
        
        from.setBalance(from.getBalance().subtract(amount));
        to.setBalance(to.getBalance().add(amount));
        
        accountRepository.save(from);
        accountRepository.save(to);
        // 정상 종료 → COMMIT, 예외 발생 → ROLLBACK
    }
    
    // 격리 수준 설정
    @Transactional(isolation = Isolation.REPEATABLE_READ)
    public void calculateTotal(Long orderId) {
        // 트랜잭션 내에서 데이터 일관성 보장
    }
    
    // 읽기 전용 최적화
    @Transactional(readOnly = true)
    public List<Order> getOrders() {
        return orderRepository.findAll();
    }
}
```

<br>

## 🏷️ 비관적 락 구현

<br>

```java
@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
    
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("SELECT p FROM Product p WHERE p.id = :id")
    Optional<Product> findByIdWithLock(@Param("id") Long id);
}

@Service
public class OrderService {
    
    @Transactional
    public void purchaseProduct(Long productId, int quantity) {
        // SELECT ... FOR UPDATE
        Product product = productRepository.findByIdWithLock(productId)
            .orElseThrow();
        
        if (product.getStock() < quantity) {
            throw new IllegalArgumentException("재고 부족");
        }
        
        product.setStock(product.getStock() - quantity);
        productRepository.save(product);
    }
}
```

<br>

## 🏷️ 낙관적 락 구현

<br>

```java
@Entity
public class Product {
    @Id
    private Long id;
    private int stock;
    
    @Version // 낙관적 락 버전 컬럼
    private Long version;
}

@Service
public class OrderService {
    
    @Transactional
    public void purchaseProduct(Long productId, int quantity) {
        try {
            Product product = productRepository.findById(productId).orElseThrow();
            
            if (product.getStock() < quantity) {
                throw new IllegalArgumentException("재고 부족");
            }
            
            product.setStock(product.getStock() - quantity);
            productRepository.save(product);
            // UPDATE ... SET version = version + 1 WHERE id = ? AND version = ?
            
        } catch (OptimisticLockException e) {
            throw new IllegalStateException("다른 사용자가 먼저 구매했습니다.");
        }
    }
    
    @Retryable(
        value = OptimisticLockException.class,
        maxAttempts = 3,
        backoff = @Backoff(delay = 100)
    )
    public void purchaseWithRetry(Long productId, int quantity) {
        purchaseProduct(productId, quantity);
    }
}
```

<br>

## 🏷️ 데드락 방지

<br>

```java
@Service
public class TransferService {
    
    @Transactional
    public void transfer(Long fromId, Long toId, BigDecimal amount) {
        // 락 순서 통일로 데드락 방지
        Long firstId = Math.min(fromId, toId);
        Long secondId = Math.max(fromId, toId);
        
        Account first = accountRepository.findByIdWithLock(firstId);
        Account second = accountRepository.findByIdWithLock(secondId);
        
        Account from = (fromId.equals(firstId)) ? first : second;
        Account to = (fromId.equals(firstId)) ? second : first;
        
        from.withdraw(amount);
        to.deposit(amount);
    }
}
```

<br>

# 실전 체크리스트

<br>

## ✅ 트랜잭션 설계

<br>

- [ ] @Transactional 적절히 사용
- [ ] 트랜잭션 범위 최소화
- [ ] 읽기 전용은 readOnly = true
- [ ] 적절한 격리 수준 선택

<br>

## ✅ 동시성 제어

<br>

- [ ] 충돌 빈도에 따라 락 선택
- [ ] 비관적 락: 재고, 결제
- [ ] 낙관적 락: 게시글, 프로필
- [ ] 낙관적 락 재시도 로직

<br>

## ✅ 데드락 방지

<br>

- [ ] 락 순서 통일
- [ ] 타임아웃 설정
- [ ] 데드락 재시도 로직
- [ ] 트랜잭션 시간 최소화

<br>

## ✅ 성능 최적화

<br>

- [ ] 인덱스 활용
- [ ] 불필요한 락 최소화
- [ ] 배치 처리 고려
- [ ] 커넥션 풀 관리

<br>

# 요약

<br>
트랜잭션은 데이터 무결성과 동시성을 보장하는 핵심 메커니즘이다. <br><br>

**💎 핵심 포인트**:

1. **ACID**: 원자성, 일관성, 격리성, 지속성
2. **격리 수준**: READ COMMITTED 권장
3. **비관적 락**: 충돌 빈번할 때
4. **낙관적 락**: 충돌 드물 때
5. **데드락 방지**: 락 순서 통일
6. **@Transactional**: Spring 트랜잭션 관리

<br>

**📌 선택 기준**:

| 상황 | 권장 방법 |
|------|-----------|
| **재고 관리** | 비관적 락 |
| **결제 처리** | 비관적 락 + SERIALIZABLE |
| **게시글 수정** | 낙관적 락 |
| **통계 조회** | 읽기 전용 트랜잭션 |
| **대량 업데이트** | 배치 처리 |

<br>

**🚀 Best Practices**:

- 트랜잭션은 짧게 유지
- 격리 수준은 필요한 만큼만
- 데드락은 예방이 최선
- 재시도 로직은 필수
- 모니터링과 로깅 중요

<br>

트랜잭션을 잘 이해하고 적용하면, <br>
데이터 정합성을 보장하면서도 <br>
높은 동시성을 달성할 수 있다. <br><br>
