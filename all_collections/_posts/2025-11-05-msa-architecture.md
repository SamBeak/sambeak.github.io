---
layout: post
title: 🏗️ MSA 아키텍처 설계와 전환 완벽 가이드
date: 2025-11-05
categories:
  [
    "SamBeak",
    "MSA",
    "Microservices",
    "Architecture",
    "Docker",
    "Kubernetes",
  ]
---

# MSA 아키텍처란 무엇인가

<br>
서비스가 커질수록 코드 수정이 점점 어려워진다. <br>
한 부분을 고치면 다른 부분이 망가지고, <br>
배포 한 번에 전체 서비스가 멈춘다. <br><br>

이는 마치 거대한 레고 블록 한 덩어리와 같다. <br>
하나의 블록을 바꾸려면 전체를 분해해야 하고, <br>
실수로 잘못 건드리면 전체가 무너진다. <br><br>

MSA(Microservices Architecture)는 <br>
이 거대한 덩어리를 작은 조각들로 나누어 <br>
각각 독립적으로 개발, 배포, 운영하는 <br>
현대적인 소프트웨어 아키텍처 패턴이다. <br><br>

> ## 왜 MSA로 전환해야 할까?

<br>

**문제 1: 배포 리스크** <br>
모놀리스는 한 줄 수정에도 전체를 재배포해야 한다. <br><br>

**문제 2: 기술 선택의 제약** <br>
전체가 하나의 언어/프레임워크에 종속된다. <br><br>

**문제 3: 확장성 한계** <br>
특정 기능만 확장하고 싶어도 전체를 확장해야 한다. <br><br>

**문제 4: 팀 협업 어려움** <br>
여러 팀이 하나의 코드베이스에서 충돌한다. <br><br>

# 기본 개념 요약

<br>

## 🏷️ 모놀리스 vs MSA

<br>

### 모놀리스 (Monolithic)

<br>
**개념**: 모든 기능이 하나의 애플리케이션 <br><br>

**식당 비유**: <br>
하나의 거대한 주방에서 <br>
전채, 메인, 디저트를 모두 만든다. <br>
셰프 한 명이 아프면 전체 주방이 멈춘다. <br><br>

**장점**: 
- 간단한 구조
- 쉬운 디버깅
- 트랜잭션 관리 용이

<br>

**단점**:
- 부분 배포 불가
- 확장성 제한
- 기술 스택 고정
- 복잡도 증가

<br>

### MSA (Microservices)

<br>
**개념**: 기능별로 독립된 작은 서비스들 <br><br>

**식당 비유**: <br>
전채 전문점, 메인 전문점, 디저트 전문점으로 분리. <br>
각 매장은 독립적으로 운영되며, <br>
한 매장에 문제가 생겨도 다른 매장은 정상 운영. <br><br>

**장점**:
- 독립적 배포
- 기술 다양성
- 선택적 확장
- 팀 자율성

<br>

**단점**:
- 복잡한 운영
- 네트워크 오버헤드
- 분산 트랜잭션
- 디버깅 어려움

<br>

## 🏷️ MSA 핵심 원칙

<br>

### 1. Single Responsibility (단일 책임)

<br>
각 서비스는 하나의 비즈니스 기능만 담당 <br><br>

**예시**:
- 사용자 서비스: 회원가입, 로그인
- 주문 서비스: 주문 생성, 조회
- 결제 서비스: 결제 처리
- 알림 서비스: 이메일, SMS 발송

<br>

### 2. Decentralized Data (분산 데이터)

<br>
각 서비스는 독립된 데이터베이스 소유 <br><br>

```
[사용자 서비스] → [MySQL: users]
[주문 서비스]   → [PostgreSQL: orders]
[결제 서비스]   → [MongoDB: payments]
```

<br>

### 3. API를 통한 통신

<br>
서비스 간 통신은 오직 API를 통해서만 <br><br>

**통신 방식**:
- REST API (동기)
- gRPC (동기, 고성능)
- Message Queue (비동기)

<br>

### 4. 독립적 배포

<br>
각 서비스를 다른 서비스와 독립적으로 배포 <br><br>

### 5. 장애 격리 (Fault Isolation)

<br>
한 서비스의 장애가 다른 서비스에 영향 최소화 <br><br>

## 🏷️ MSA 핵심 컴포넌트

<br>

### 1. API Gateway

<br>
**역할**: 클라이언트 요청을 적절한 서비스로 라우팅 <br><br>

**기능**:
- 인증/인가
- 요청 라우팅
- 로드 밸런싱
- Rate Limiting
- 로깅/모니터링

<br>

**도구**: Kong, Spring Cloud Gateway, AWS API Gateway <br><br>

### 2. Service Discovery

<br>
**역할**: 서비스 위치 자동 탐색 <br><br>

**필요성**: <br>
MSA에서는 서비스가 동적으로 생성/삭제되므로 <br>
IP와 포트를 자동으로 찾아야 한다. <br><br>

**도구**: Eureka, Consul, Kubernetes Service <br><br>

### 3. Configuration Management

<br>
**역할**: 중앙화된 설정 관리 <br><br>

**도구**: Spring Cloud Config, Consul, etcd <br><br>

### 4. Message Broker

<br>
**역할**: 비동기 통신 및 이벤트 처리 <br><br>

**도구**: RabbitMQ, Kafka, AWS SQS <br><br>

### 5. Container Orchestration

<br>
**역할**: 컨테이너 배포 및 관리 자동화 <br><br>

**도구**: Kubernetes, Docker Swarm, ECS <br><br>

# MSA로 전환하기

<br>

> ## 전환 전략

<br>

### 1. Strangler Fig Pattern (교살자 무화과 패턴)

<br>
**개념**: 점진적으로 모놀리스를 MSA로 교체 <br><br>

**단계**:
1. 새 기능은 MSA로 개발
2. 기존 기능을 하나씩 MSA로 이동
3. 모놀리스를 점진적으로 축소
4. 최종적으로 모놀리스 제거

<br>

```
[모놀리스 100%]
↓
[모놀리스 80% + MSA 20%]
↓
[모놀리스 50% + MSA 50%]
↓
[모놀리스 20% + MSA 80%]
↓
[MSA 100%]
```

<br>

### 2. Database-per-Service

<br>
**원칙**: 각 서비스가 독립된 DB 소유 <br><br>

**전환 방법**:
1. 스키마 분리
2. 데이터 복제 (동기화)
3. API를 통한 데이터 조회

<br>

### 3. 서비스 분리 기준

<br>

**Domain-Driven Design (DDD) 기반**: <br>

- **Bounded Context** 단위로 분리
- **Aggregate** 단위로 데이터 관리

<br>

**분리 순서**:
1. 가장 독립적인 기능부터
2. 변경이 잦은 기능
3. 확장이 필요한 기능
4. 기술 스택 변경이 필요한 기능

<br>

# 실전 예시

<br>

> ## Spring Boot MSA 구성

<br>

### API Gateway (Spring Cloud Gateway)

<br>

```java
@Configuration
public class GatewayConfig {
    
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            // 사용자 서비스
            .route("user-service", r -> r
                .path("/api/users/**")
                .filters(f -> f
                    .rewritePath("/api/users/(?<segment>.*)", "/users/${segment}")
                    .addRequestHeader("X-Gateway", "true"))
                .uri("lb://USER-SERVICE"))
            
            // 주문 서비스
            .route("order-service", r -> r
                .path("/api/orders/**")
                .uri("lb://ORDER-SERVICE"))
            
            // 결제 서비스
            .route("payment-service", r -> r
                .path("/api/payments/**")
                .uri("lb://PAYMENT-SERVICE"))
            
            .build();
    }
}
```

<br>

### Service Discovery (Eureka)

<br>

**Eureka Server**:

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

<br>

**Eureka Client (각 서비스)**:

```java
@SpringBootApplication
@EnableDiscoveryClient
public class UserServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}
```

<br>

```yaml
# application.yml
spring:
  application:
    name: user-service

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
```

<br>

> ## 서비스 간 통신

<br>

### 1. REST API (동기)

<br>

```java
@Service
public class OrderService {
    
    @Autowired
    private RestTemplate restTemplate;
    
    public OrderResponse createOrder(OrderRequest request) {
        // 1. 사용자 정보 확인
        String userUrl = "http://USER-SERVICE/users/" + request.getUserId();
        User user = restTemplate.getForObject(userUrl, User.class);
        
        if (user == null) {
            throw new UserNotFoundException();
        }
        
        // 2. 주문 생성
        Order order = new Order();
        order.setUserId(user.getId());
        order.setItems(request.getItems());
        orderRepository.save(order);
        
        // 3. 결제 요청
        PaymentRequest paymentRequest = new PaymentRequest();
        paymentRequest.setOrderId(order.getId());
        paymentRequest.setAmount(order.getTotalAmount());
        
        String paymentUrl = "http://PAYMENT-SERVICE/payments";
        PaymentResponse payment = restTemplate.postForObject(
            paymentUrl, 
            paymentRequest, 
            PaymentResponse.class
        );
        
        return new OrderResponse(order, payment);
    }
}
```

<br>

### 2. Message Queue (비동기)

<br>

**주문 서비스 → 이벤트 발행**:

```java
@Service
public class OrderService {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    public void createOrder(OrderRequest request) {
        Order order = new Order();
        order.setUserId(request.getUserId());
        orderRepository.save(order);
        
        // 주문 생성 이벤트 발행
        OrderCreatedEvent event = new OrderCreatedEvent(
            order.getId(),
            order.getUserId(),
            order.getTotalAmount()
        );
        
        rabbitTemplate.convertAndSend(
            "order.exchange",
            "order.created",
            event
        );
    }
}
```

<br>

**알림 서비스 → 이벤트 구독**:

```java
@Service
public class NotificationService {
    
    @RabbitListener(queues = "notification.queue")
    public void handleOrderCreated(OrderCreatedEvent event) {
        // 주문 완료 알림 발송
        sendEmail(event.getUserId(), "주문이 완료되었습니다.");
        sendSMS(event.getUserId(), "주문 번호: " + event.getOrderId());
    }
}
```

<br>

> ## Docker & Kubernetes 배포

<br>

### Dockerfile (각 서비스)

<br>

```dockerfile
FROM openjdk:17-jdk-slim

WORKDIR /app

COPY target/user-service.jar app.jar

EXPOSE 8080

ENV JAVA_OPTS="-Xmx512m -Xms256m"

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

<br>

### Kubernetes Deployment

<br>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: user-service
  template:
    metadata:
      labels:
        app: user-service
    spec:
      containers:
      - name: user-service
        image: myregistry/user-service:1.0.0
        ports:
        - containerPort: 8080
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "prod"
        - name: EUREKA_SERVER
          value: "http://eureka-server:8761/eureka"
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 20
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: user-service
spec:
  selector:
    app: user-service
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
```

<br>

> ## Circuit Breaker (장애 격리)

<br>

**Resilience4j 적용**:

```java
@Service
public class OrderService {
    
    @Autowired
    private RestTemplate restTemplate;
    
    @CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
    @Retry(name = "paymentService", fallbackMethod = "paymentFallback")
    @TimeLimiter(name = "paymentService")
    public CompletableFuture<PaymentResponse> processPayment(PaymentRequest request) {
        return CompletableFuture.supplyAsync(() -> {
            String url = "http://PAYMENT-SERVICE/payments";
            return restTemplate.postForObject(url, request, PaymentResponse.class);
        });
    }
    
    // Fallback 메서드
    public CompletableFuture<PaymentResponse> paymentFallback(
        PaymentRequest request, 
        Exception ex
    ) {
        log.error("결제 서비스 호출 실패. Fallback 실행", ex);
        
        // 대체 로직
        PaymentResponse response = new PaymentResponse();
        response.setStatus("PENDING");
        response.setMessage("결제 처리 중입니다. 잠시 후 확인해주세요.");
        
        return CompletableFuture.completedFuture(response);
    }
}
```

<br>

**application.yml**:

```yaml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        registerHealthIndicator: true
        slidingWindowSize: 10
        minimumNumberOfCalls: 5
        permittedNumberOfCallsInHalfOpenState: 3
        automaticTransitionFromOpenToHalfOpenEnabled: true
        waitDurationInOpenState: 5s
        failureRateThreshold: 50
        eventConsumerBufferSize: 10
  
  retry:
    instances:
      paymentService:
        maxAttempts: 3
        waitDuration: 1s
  
  timelimiter:
    instances:
      paymentService:
        timeoutDuration: 3s
```

<br>

# 분산 트랜잭션 처리

<br>

## 🏷️ Saga Pattern

<br>
**개념**: 분산 환경에서 트랜잭션을 여러 단계로 나누어 처리 <br><br>

### Choreography 방식 (이벤트 기반)

<br>

```java
// 1. 주문 서비스
@Service
public class OrderService {
    
    public void createOrder(OrderRequest request) {
        Order order = orderRepository.save(new Order(request));
        
        // 이벤트 발행
        eventPublisher.publish(new OrderCreatedEvent(order));
    }
    
    @EventListener
    public void handlePaymentFailed(PaymentFailedEvent event) {
        // 보상 트랜잭션: 주문 취소
        Order order = orderRepository.findById(event.getOrderId());
        order.setStatus(OrderStatus.CANCELLED);
        orderRepository.save(order);
    }
}

// 2. 결제 서비스
@Service
public class PaymentService {
    
    @EventListener
    public void handleOrderCreated(OrderCreatedEvent event) {
        try {
            Payment payment = processPayment(event);
            eventPublisher.publish(new PaymentCompletedEvent(payment));
        } catch (Exception ex) {
            eventPublisher.publish(new PaymentFailedEvent(event.getOrderId()));
        }
    }
}

// 3. 배송 서비스
@Service
public class DeliveryService {
    
    @EventListener
    public void handlePaymentCompleted(PaymentCompletedEvent event) {
        Delivery delivery = createDelivery(event);
        eventPublisher.publish(new DeliveryStartedEvent(delivery));
    }
}
```

<br>

### Orchestration 방식 (중앙 조율)

<br>

```java
@Service
public class OrderOrchestrator {
    
    public void processOrder(OrderRequest request) {
        String sagaId = UUID.randomUUID().toString();
        
        try {
            // 1. 주문 생성
            Order order = orderService.createOrder(request);
            
            // 2. 결제 처리
            Payment payment = paymentService.processPayment(order);
            
            // 3. 재고 차감
            inventoryService.decreaseStock(order.getItems());
            
            // 4. 배송 시작
            deliveryService.startDelivery(order);
            
            // 모두 성공
            order.setStatus(OrderStatus.COMPLETED);
            
        } catch (PaymentException ex) {
            // 보상 트랜잭션
            orderService.cancelOrder(order.getId());
            throw ex;
            
        } catch (InventoryException ex) {
            // 보상 트랜잭션
            paymentService.refund(payment.getId());
            orderService.cancelOrder(order.getId());
            throw ex;
        }
    }
}
```

<br>

# MSA 모니터링

<br>

## 🏷️ 분산 추적 (Distributed Tracing)

<br>

**Zipkin/Jaeger 연동**:

```java
@Configuration
public class TracingConfig {
    
    @Bean
    public Tracer jaegerTracer() {
        return Configuration.fromEnv("user-service")
            .withSampler(new ConstSampler(true))
            .withReporter(new LoggingReporter())
            .getTracer();
    }
}
```

<br>

**사용 예시**:

```java
@Service
public class UserService {
    
    @Autowired
    private Tracer tracer;
    
    public User getUser(Long id) {
        Span span = tracer.buildSpan("getUser").start();
        
        try {
            span.setTag("userId", id);
            User user = userRepository.findById(id);
            span.log("User found: " + user.getName());
            return user;
        } catch (Exception ex) {
            span.setTag("error", true);
            span.log(ex.getMessage());
            throw ex;
        } finally {
            span.finish();
        }
    }
}
```

<br>

## 🏷️ 중앙 로깅

<br>

**ELK Stack (Elasticsearch, Logstash, Kibana)**:

```yaml
# logstash.conf
input {
  file {
    path => "/var/log/services/*.log"
    start_position => "beginning"
  }
}

filter {
  json {
    source => "message"
  }
  
  grok {
    match => { "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} %{GREEDYDATA:message}" }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "services-%{+YYYY.MM.dd}"
  }
}
```

<br>

# MSA 안티패턴

<br>

## ❌ 안티패턴 1: 너무 작은 서비스

<br>

### 문제

<br>

```
❌ 잘못된 분리:
- UserProfileService
- UserAddressService
- UserPhoneService
- UserEmailService
```

<br>

### 해결

<br>

```
✅ 적절한 분리:
- UserService (프로필, 주소, 연락처 통합)
```

<br>

## ❌ 안티패턴 2: 공유 데이터베이스

<br>

### 문제

<br>

```
❌ 여러 서비스가 하나의 DB 공유
[주문 서비스] ─┐
[결제 서비스] ─┼─→ [공유 DB]
[배송 서비스] ─┘
```

<br>

### 해결

<br>

```
✅ 서비스별 독립 DB
[주문 서비스] → [주문 DB]
[결제 서비스] → [결제 DB]
[배송 서비스] → [배송 DB]
```

<br>

## ❌ 안티패턴 3: 동기 호출 체인

<br>

### 문제

<br>

```java
// ❌ 동기 호출의 연쇄
API Gateway → Service A (100ms)
  → Service B (100ms)
    → Service C (100ms)
      → Service D (100ms)
        
총 응답 시간: 400ms + 네트워크 지연
```

<br>

### 해결

<br>

```java
// ✅ 비동기 이벤트 기반
API Gateway → Service A (이벤트 발행, 즉시 응답)
  ↓ 이벤트
Service B, C, D (병렬 처리)

응답 시간: 100ms
```

<br>

# 실전 체크리스트

<br>

## ✅ MSA 도입 전

<br>

- [ ] 팀 규모가 충분한가? (최소 3개 이상 팀)
- [ ] DevOps 역량이 있는가?
- [ ] 모놀리스의 문제가 명확한가?
- [ ] 복잡도 증가를 감당할 수 있는가?

<br>

## ✅ 서비스 설계

<br>

- [ ] 단일 책임 원칙 준수
- [ ] 독립된 데이터베이스
- [ ] API 우선 설계
- [ ] 버전 관리 전략
- [ ] 에러 처리 표준화

<br>

## ✅ 운영 인프라

<br>

- [ ] API Gateway 구축
- [ ] Service Discovery 구현
- [ ] 중앙 로깅 시스템
- [ ] 분산 추적 도구
- [ ] 모니터링 대시보드

<br>

## ✅ 안정성

<br>

- [ ] Circuit Breaker 적용
- [ ] Retry 정책
- [ ] Timeout 설정
- [ ] Health Check
- [ ] Graceful Shutdown

<br>

## ✅ 보안

<br>

- [ ] API Gateway 인증
- [ ] 서비스 간 인증 (mTLS)
- [ ] 비밀 정보 관리 (Vault)
- [ ] 네트워크 격리

<br>

# 요약

<br>
MSA는 복잡도와 자율성의 트레이드오프다. <br><br>

**💎 핵심 포인트**:

1. **단일 책임**: 서비스는 하나의 기능만
2. **독립 배포**: 다른 서비스와 무관하게 배포
3. **데이터 분리**: 각 서비스가 독립 DB 소유
4. **API 통신**: 서비스 간 통신은 API로만
5. **장애 격리**: Circuit Breaker로 장애 전파 차단
6. **점진적 전환**: Strangler Fig 패턴 활용

<br>

**🚀 전환 순서**:

1단계: 모놀리스 분석 및 서비스 경계 정의 <br>
2단계: API Gateway, Service Discovery 구축 <br>
3단계: 독립성 높은 기능부터 분리 <br>
4단계: 데이터베이스 분리 <br>
5단계: 모니터링 및 로깅 구축 <br>
6단계: 점진적으로 확대 <br><br>

**⚠️ 주의사항**:

- 작은 서비스가 좋은 것은 아님
- 공유 DB는 MSA가 아님
- 동기 호출 체인은 피할 것
- 분산 트랜잭션은 Saga 패턴으로
- 모니터링 없는 MSA는 위험

<br>

**📊 도입 시기**:

MSA는 만능이 아니다. 다음 조건에 해당할 때 고려: <br>

- 팀이 3개 이상으로 분리 가능
- 배포 빈도가 주 1회 이상
- 특정 기능의 확장 필요
- 기술 스택 다양화 필요
- DevOps 문화가 정착됨

<br>

MSA는 기술적 선택이 아니라 <br>
조직과 비즈니스의 문제를 해결하는 수단이다. <br>
무조건 도입보다는 현재 문제를 정확히 파악하고, <br>
MSA가 진짜 해결책인지 먼저 고민해야 한다. <br><br>
