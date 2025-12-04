---
layout: post
title: 🏗️ DDD(Domain-Driven Design) 완벽 가이드 (Entity, Aggregate, Repository)
date: 2025-11-11
categories:
  [
    "SamBeak",
    "Architecture",
    "DDD",
    "Design Pattern",
    "Domain Model",
  ]
---

# DDD란 무엇인가

<br>
프로그램을 만들 때 가장 중요한 것은? <br>
바로 **비즈니스 로직**이다. <br><br>

하지만 코드가 복잡해지면, <br>
비즈니스 로직이 여기저기 흩어지고, <br>
어디가 핵심인지 알기 어려워진다. <br><br>

**DDD(Domain-Driven Design)**는 <br>
비즈니스 도메인을 중심으로 설계하는 방법이다. <br><br>

마치 레고 블록을 조립하듯, <br>
도메인을 의미 있는 단위로 나누고, <br>
각 블록의 역할을 명확히 정의한다. <br><br>

> ## 왜 DDD를 배워야 할까?

<br>

**이유 1: 복잡도 관리** <br>
큰 프로젝트를 의미 있는 단위로 분해 <br><br>

**이유 2: 도메인 전문가와 소통** <br>
개발자와 비즈니스 담당자가 같은 언어 사용 <br><br>

**이유 3: 유지보수성 향상** <br>
비즈니스 로직이 명확하게 분리됨 <br><br>

**이유 4: 면접 필수** <br>
Entity, Aggregate, Repository는 면접 단골 <br><br>

# 기본 개념 요약

<br>

## 🏷️ DDD 핵심 구성 요소

<br>

### 1. Entity (엔티티)

<br>
**개념**: 고유한 식별자를 가진 객체 <br><br>

**특징**:
- ID로 구분 가능
- 생명주기가 있음
- 속성이 바뀌어도 동일한 객체

<br>

**학생 비유**: <br>
학번(ID)으로 구분하는 학생 <br>
이름이 바뀌어도 같은 학생 <br><br>

```java
@Entity
public class User {
    @Id
    private Long id; // 식별자
    private String name;
    private String email;
    
    // 같은 ID면 같은 User
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof User)) return false;
        User user = (User) o;
        return Objects.equals(id, user.id);
    }
}
```

<br>

### 2. Value Object (값 객체)

<br>
**개념**: 식별자 없이 값으로만 구분되는 객체 <br><br>

**특징**:
- ID가 없음
- 불변(Immutable)
- 값이 같으면 같은 객체

<br>

**돈 비유**: <br>
1만원짜리 지폐는 ID가 없음 <br>
1만원이면 모두 같은 가치 <br><br>

```java
public class Money {
    private final BigDecimal amount;
    private final Currency currency;
    
    public Money(BigDecimal amount, Currency currency) {
        this.amount = amount;
        this.currency = currency;
    }
    
    // 불변: 새 객체 반환
    public Money add(Money other) {
        return new Money(
            this.amount.add(other.amount),
            this.currency
        );
    }
    
    @Override
    public boolean equals(Object o) {
        Money money = (Money) o;
        return amount.equals(money.amount) && 
               currency.equals(money.currency);
    }
}
```

<br>

### 3. Aggregate (애그리거트)

<br>
**개념**: 관련된 객체들의 묶음 <br><br>

**특징**:
- 하나의 Root Entity가 존재
- 외부에서는 Root를 통해서만 접근
- 트랜잭션 단위

<br>

**가족 비유**: <br>
가장(Root)을 통해 가족 구성원에게 접근 <br>
외부에서 직접 자녀에게 연락 안 함 <br><br>

```java
// Order가 Aggregate Root
public class Order {
    @Id
    private Long id;
    
    private List<OrderItem> items = new ArrayList<>();
    private OrderStatus status;
    
    // Root를 통해서만 아이템 추가
    public void addItem(Product product, int quantity) {
        OrderItem item = new OrderItem(product, quantity);
        items.add(item);
    }
    
    // 외부에서 직접 items 수정 불가
    public List<OrderItem> getItems() {
        return Collections.unmodifiableList(items);
    }
}

class OrderItem {
    private Product product;
    private int quantity;
    // OrderItem은 독립적으로 존재 불가
}
```

<br>

### 4. Repository (리포지토리)

<br>
**개념**: Aggregate를 저장/조회하는 인터페이스 <br><br>

**특징**:
- 도메인 계층에 속함
- 컬렉션처럼 동작
- Root만 Repository를 가짐

<br>

**도서관 비유**: <br>
도서관(Repository)에서 책(Aggregate) 대출/반납 <br><br>

```java
public interface OrderRepository {
    Order save(Order order);
    Optional<Order> findById(Long id);
    List<Order> findByUserId(Long userId);
    void delete(Order order);
}
```

<br>

### 5. Domain Service (도메인 서비스)

<br>
**개념**: 특정 Entity에 속하지 않는 비즈니스 로직 <br><br>

```java
@Service
public class TransferService {
    
    public void transfer(Account from, Account to, Money amount) {
        // 여러 Aggregate 간 로직
        from.withdraw(amount);
        to.deposit(amount);
    }
}
```

<br>

## 🏷️ 계층 구조

<br>

```
┌─────────────────────────────┐
│   Presentation Layer        │  ← Controller, View
│   (사용자 인터페이스)        │
└─────────────────────────────┘
            ↓
┌─────────────────────────────┐
│   Application Layer         │  ← Use Case, Service
│   (응용 계층)                │
└─────────────────────────────┘
            ↓
┌─────────────────────────────┐
│   Domain Layer              │  ← Entity, VO, Aggregate
│   (도메인 계층 - 핵심!)     │     Domain Service
└─────────────────────────────┘
            ↓
┌─────────────────────────────┐
│   Infrastructure Layer      │  ← Repository 구현
│   (인프라 계층)              │     DB, 외부 API
└─────────────────────────────┘
```

<br>

## 🏷️ Bounded Context (경계 컨텍스트)

<br>

**개념**: 도메인 모델이 적용되는 경계 <br><br>

**쇼핑몰 예시**:

```
[주문 Context]
- Order (주문)
- OrderItem
- Customer (구매자)

[배송 Context]
- Delivery (배송)
- Customer (수령자)

[결제 Context]
- Payment (결제)
- Customer (결제자)
```

<br>

같은 Customer지만 Context마다 다른 의미! <br><br>

# 실전 예시

<br>

## 🏷️ 주문 시스템 DDD 설계

<br>

### 1. Value Object

<br>

```java
// Money - 불변 값 객체
public class Money {
    private final BigDecimal amount;
    
    public Money(BigDecimal amount) {
        if (amount.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("금액은 0 이상");
        }
        this.amount = amount;
    }
    
    public Money add(Money other) {
        return new Money(this.amount.add(other.amount));
    }
    
    public Money multiply(int quantity) {
        return new Money(this.amount.multiply(new BigDecimal(quantity)));
    }
}

// Address - 불변 값 객체
public class Address {
    private final String city;
    private final String street;
    private final String zipCode;
    
    public Address(String city, String street, String zipCode) {
        this.city = city;
        this.street = street;
        this.zipCode = zipCode;
    }
    
    // getter만 존재 (불변)
}
```

<br>

### 2. Entity

<br>

```java
@Entity
public class Product {
    @Id
    @GeneratedValue
    private Long id;
    
    private String name;
    
    @Embedded
    private Money price;
    
    private int stock;
    
    // 비즈니스 로직
    public void decreaseStock(int quantity) {
        if (this.stock < quantity) {
            throw new OutOfStockException("재고 부족");
        }
        this.stock -= quantity;
    }
}
```

<br>

### 3. Aggregate Root

<br>

```java
@Entity
public class Order {
    @Id
    @GeneratedValue
    private Long id;
    
    @ManyToOne
    private Customer customer;
    
    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();
    
    @Enumerated(EnumType.STRING)
    private OrderStatus status;
    
    @Embedded
    private Address shippingAddress;
    
    private LocalDateTime orderedAt;
    
    // 생성자 - 팩토리 메서드
    public static Order create(Customer customer, Address address) {
        Order order = new Order();
        order.customer = customer;
        order.shippingAddress = address;
        order.status = OrderStatus.PENDING;
        order.orderedAt = LocalDateTime.now();
        return order;
    }
    
    // 비즈니스 로직: Root를 통해서만 아이템 추가
    public void addItem(Product product, int quantity) {
        // 재고 확인
        product.decreaseStock(quantity);
        
        // 아이템 추가
        OrderItem item = new OrderItem(product, quantity);
        items.add(item);
    }
    
    // 총 금액 계산
    public Money calculateTotalPrice() {
        return items.stream()
            .map(OrderItem::getSubtotal)
            .reduce(Money.ZERO, Money::add);
    }
    
    // 주문 확정
    public void confirm() {
        if (items.isEmpty()) {
            throw new IllegalStateException("주문 항목이 없음");
        }
        this.status = OrderStatus.CONFIRMED;
    }
    
    // 주문 취소
    public void cancel() {
        if (this.status == OrderStatus.SHIPPED) {
            throw new IllegalStateException("배송된 주문은 취소 불가");
        }
        
        // 재고 복구
        items.forEach(item -> 
            item.getProduct().increaseStock(item.getQuantity())
        );
        
        this.status = OrderStatus.CANCELLED;
    }
}

@Entity
class OrderItem {
    @Id
    @GeneratedValue
    private Long id;
    
    @ManyToOne
    private Product product;
    
    private int quantity;
    
    @Embedded
    private Money price; // 주문 당시 가격
    
    public OrderItem(Product product, int quantity) {
        this.product = product;
        this.quantity = quantity;
        this.price = product.getPrice();
    }
    
    public Money getSubtotal() {
        return price.multiply(quantity);
    }
}
```

<br>

### 4. Repository

<br>

```java
public interface OrderRepository {
    Order save(Order order);
    Optional<Order> findById(Long id);
    List<Order> findByCustomerId(Long customerId);
    List<Order> findByStatus(OrderStatus status);
}

@Repository
public class JpaOrderRepository implements OrderRepository {
    
    @PersistenceContext
    private EntityManager em;
    
    @Override
    public Order save(Order order) {
        if (order.getId() == null) {
            em.persist(order);
            return order;
        } else {
            return em.merge(order);
        }
    }
    
    @Override
    public Optional<Order> findById(Long id) {
        return Optional.ofNullable(em.find(Order.class, id));
    }
}
```

<br>

### 5. Domain Service

<br>

```java
@Service
public class OrderService {
    
    private final OrderRepository orderRepository;
    private final ProductRepository productRepository;
    
    @Transactional
    public Long createOrder(CreateOrderCommand command) {
        // 1. Customer 조회
        Customer customer = customerRepository.findById(command.getCustomerId())
            .orElseThrow(() -> new IllegalArgumentException("고객 없음"));
        
        // 2. Order 생성 (Aggregate Root)
        Order order = Order.create(customer, command.getShippingAddress());
        
        // 3. 상품 추가
        for (OrderItemRequest item : command.getItems()) {
            Product product = productRepository.findById(item.getProductId())
                .orElseThrow(() -> new IllegalArgumentException("상품 없음"));
            
            order.addItem(product, item.getQuantity());
        }
        
        // 4. 주문 확정
        order.confirm();
        
        // 5. 저장
        Order savedOrder = orderRepository.save(order);
        
        return savedOrder.getId();
    }
    
    @Transactional
    public void cancelOrder(Long orderId) {
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new IllegalArgumentException("주문 없음"));
        
        // 비즈니스 로직은 Aggregate에 위임
        order.cancel();
        
        orderRepository.save(order);
    }
}
```

<br>

### 6. Application Service (Use Case)

<br>

```java
@Service
@RequiredArgsConstructor
public class OrderApplicationService {
    
    private final OrderService orderService;
    private final PaymentService paymentService;
    private final NotificationService notificationService;
    
    @Transactional
    public OrderResponse placeOrder(CreateOrderCommand command) {
        // 1. 주문 생성
        Long orderId = orderService.createOrder(command);
        
        // 2. 결제 처리
        paymentService.processPayment(orderId, command.getPaymentMethod());
        
        // 3. 알림 전송
        notificationService.sendOrderConfirmation(orderId);
        
        // 4. 응답 생성
        return OrderResponse.from(orderId);
    }
}
```

<br>

## 🏷️ 패키지 구조

<br>

```
src/main/java/com/example/shop/
├── domain/
│   ├── order/
│   │   ├── Order.java              (Aggregate Root)
│   │   ├── OrderItem.java          (Entity)
│   │   ├── OrderStatus.java        (Enum)
│   │   ├── OrderRepository.java    (Interface)
│   │   └── OrderService.java       (Domain Service)
│   ├── product/
│   │   ├── Product.java
│   │   └── ProductRepository.java
│   └── common/
│       ├── Money.java              (Value Object)
│       └── Address.java            (Value Object)
├── application/
│   └── OrderApplicationService.java
├── infrastructure/
│   ├── persistence/
│   │   ├── JpaOrderRepository.java
│   │   └── JpaProductRepository.java
│   └── messaging/
│       └── KafkaEventPublisher.java
└── presentation/
    └── OrderController.java
```

<br>

# 실전 체크리스트

<br>

## ✅ 도메인 모델 설계

<br>

- [ ] Aggregate 경계 명확히 정의
- [ ] Value Object 적극 활용
- [ ] Entity는 비즈니스 로직 포함
- [ ] 불변성 최대한 유지

<br>

## ✅ Repository 패턴

<br>

- [ ] Aggregate Root만 Repository
- [ ] 도메인 계층에 인터페이스
- [ ] 인프라 계층에 구현체
- [ ] 컬렉션처럼 사용

<br>

## ✅ 비즈니스 로직 위치

<br>

- [ ] Entity/Aggregate에 최대한
- [ ] Domain Service는 최소화
- [ ] Application Service는 조정만
- [ ] Controller는 변환만

<br>

## ✅ Bounded Context

<br>

- [ ] Context 경계 명확히
- [ ] 같은 용어도 Context마다 다름
- [ ] Context Map 작성
- [ ] 마이크로서비스 경계

<br>

# 요약

<br>
DDD는 비즈니스 도메인을 중심으로 설계하는 방법이다. <br><br>

**💎 핵심 포인트**:

1. **Entity**: 식별자 있는 객체
2. **Value Object**: 불변 값 객체
3. **Aggregate**: 관련 객체 묶음
4. **Repository**: 저장/조회 인터페이스
5. **Domain Service**: Entity 밖 로직
6. **Bounded Context**: 모델 경계

<br>

**📌 설계 원칙**:

| 원칙 | 설명 |
|------|------|
| **도메인 우선** | 기술보다 비즈니스 우선 |
| **Ubiquitous Language** | 개발자-도메인 전문가 동일 용어 |
| **Aggregate 경계** | 트랜잭션 일관성 단위 |
| **불변성** | Value Object는 불변 |

<br>

**🚀 Best Practices**:

- Aggregate는 작게 유지
- Repository는 Root만
- 비즈니스 로직은 Entity에
- Anemic Model 지양
- 과도한 추상화 지양
- 팀과 지속적 소통

<br>

DDD는 복잡한 도메인을 다룰 때 빛을 발한다. <br>
작은 프로젝트에는 과할 수 있지만, <br>
큰 프로젝트에서는 유지보수성을 크게 향상시킨다. <br><br>
