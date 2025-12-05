---
layout: post
title: 📬 메시지 큐 완벽 가이드 (RabbitMQ, Kafka, 비동기 처리)
date: 2025-11-13
categories:
  ["SamBeak", "Architecture", "MessageQueue", "RabbitMQ", "Kafka", "Backend"]
---

# 메시지 큐란 무엇인가

<br>
편의점 택배를 생각해보자. <br>
보내는 사람이 택배를 맡기면, <br>
받는 사람이 나중에 찾아간다. <br><br>

보내는 사람은 받는 사람을 기다릴 필요 없다. <br>
받는 사람도 보내는 사람을 기다릴 필요 없다. <br><br>

**메시지 큐**는 이런 택배 보관함 역할을 한다. <br>
시스템 간 메시지를 임시 저장하고, <br>
받는 쪽이 준비되면 처리한다. <br><br>

> ## 왜 메시지 큐를 배워야 할까?

<br>

**이유 1: 비동기 처리** <br>
오래 걸리는 작업을 나중에 처리 <br><br>

**이유 2: 시스템 분리** <br>
서비스 간 느슨한 결합 <br><br>

**이유 3: 부하 분산** <br>
트래픽 급증 시 버퍼 역할 <br><br>

**이유 4: 면접 필수** <br>
MSA, 이벤트 기반 아키텍처 단골 질문 <br><br>

# 기본 개념 요약

<br>

## 🏷️ 메시지 큐 구조

<br>

```
[Producer]  ──→  [Message Queue]  ──→  [Consumer]
 (발신자)          (메시지 저장소)        (수신자)
```

<br>

### Producer (생산자)

<br>
**개념**: 메시지를 보내는 쪽 <br><br>

**예시**:

- 주문 서비스가 "주문 생성됨" 메시지 발행
- 결제 서비스가 "결제 완료" 메시지 발행

<br>

### Queue (큐)

<br>
**개념**: 메시지를 임시 저장하는 공간 <br><br>

**특징**:

- FIFO (First In First Out)
- 메시지 영속성 보장
- 여러 Consumer가 구독 가능

<br>

### Consumer (소비자)

<br>
**개념**: 메시지를 받아서 처리하는 쪽 <br><br>

**예시**:

- 알림 서비스가 "주문 생성됨" 메시지 수신
- 배송 서비스가 "결제 완료" 메시지 수신

<br>

## 🏷️ 메시징 패턴

<br>

### 1. Point-to-Point (P2P)

<br>

```
[Producer] ──→ [Queue] ──→ [Consumer]
                  │
                  └──→ 하나의 Consumer만 처리
```

<br>

**특징**: 메시지당 하나의 Consumer만 처리 <br>
**사용 예**: 작업 분배, 태스크 큐 <br><br>

### 2. Publish/Subscribe (Pub/Sub)

<br>

```
                    ┌──→ [Consumer 1]
[Producer] ──→ [Topic] ──→ [Consumer 2]
                    └──→ [Consumer 3]
```

<br>

**특징**: 모든 구독자가 메시지 수신 <br>
**사용 예**: 이벤트 브로드캐스트, 알림 <br><br>

## 🏷️ RabbitMQ vs Kafka

<br>

| 구분            | RabbitMQ      | Kafka             |
| --------------- | ------------- | ----------------- |
| **모델**        | 메시지 브로커 | 이벤트 스트리밍   |
| **메시지 보관** | 소비 후 삭제  | 영구 보관         |
| **처리량**      | 중간          | 매우 높음         |
| **순서 보장**   | 큐 단위       | 파티션 단위       |
| **재처리**      | 어려움        | 쉬움              |
| **적합한 경우** | 작업 큐, RPC  | 로그, 이벤트 소싱 |

<br>

## 🏷️ 핵심 용어

<br>

**Exchange (RabbitMQ)**: 메시지 라우팅 규칙 <br>
**Topic (Kafka)**: 메시지 카테고리 <br>
**Partition (Kafka)**: 토픽 내 병렬 처리 단위 <br>
**Consumer Group**: 같은 메시지를 나눠 처리하는 그룹 <br>
**Offset (Kafka)**: 메시지 위치 (재처리 가능) <br>
**ACK**: 메시지 처리 완료 확인 <br><br>

# 실전 예시

<br>

## 🏷️ RabbitMQ + NestJS

<br>

### 1. 설치 및 설정

<br>

```bash
# RabbitMQ Docker 실행
docker run -d --name rabbitmq \
  -p 5672:5672 -p 15672:15672 \
  rabbitmq:3-management

# NestJS 패키지 설치
npm install @nestjs/microservices amqplib amqp-connection-manager
```

<br>

### 2. Producer 구현

<br>

```typescript
// src/orders/orders.module.ts
import { Module } from "@nestjs/common";
import { ClientsModule, Transport } from "@nestjs/microservices";

@Module({
  imports: [
    ClientsModule.register([
      {
        name: "NOTIFICATION_SERVICE",
        transport: Transport.RMQ,
        options: {
          urls: ["amqp://localhost:5672"],
          queue: "notifications_queue",
          queueOptions: { durable: true },
        },
      },
    ]),
  ],
})
export class OrdersModule {}

// src/orders/orders.service.ts
import { Inject, Injectable } from "@nestjs/common";
import { ClientProxy } from "@nestjs/microservices";

@Injectable()
export class OrdersService {
  constructor(
    @Inject("NOTIFICATION_SERVICE")
    private readonly notificationClient: ClientProxy
  ) {}

  async createOrder(dto: CreateOrderDto): Promise<Order> {
    // 1. 주문 생성
    const order = await this.orderRepository.save({
      userId: dto.userId,
      items: dto.items,
      status: "PENDING",
    });

    // 2. 메시지 발행 (비동기)
    this.notificationClient.emit("order_created", {
      orderId: order.id,
      userId: order.userId,
      totalAmount: order.totalAmount,
    });

    // 3. 즉시 응답 (알림은 나중에 처리됨)
    return order;
  }

  async cancelOrder(orderId: string): Promise<void> {
    const order = await this.orderRepository.findById(orderId);
    order.status = "CANCELLED";
    await this.orderRepository.save(order);

    // 취소 이벤트 발행
    this.notificationClient.emit("order_cancelled", {
      orderId: order.id,
      userId: order.userId,
    });
  }
}
```

<br>

### 3. Consumer 구현

<br>

```typescript
// src/notifications/notifications.controller.ts
import { Controller } from "@nestjs/common";
import { EventPattern, Payload } from "@nestjs/microservices";

@Controller()
export class NotificationsController {
  constructor(
    private readonly emailService: EmailService,
    private readonly pushService: PushService
  ) {}

  @EventPattern("order_created")
  async handleOrderCreated(
    @Payload() data: { orderId: string; userId: string; totalAmount: number }
  ): Promise<void> {
    console.log("주문 생성 이벤트 수신:", data);

    // 이메일 발송
    await this.emailService.sendOrderConfirmation(
      data.userId,
      data.orderId,
      data.totalAmount
    );

    // 푸시 알림
    await this.pushService.sendNotification(
      data.userId,
      `주문이 완료되었습니다. 주문번호: ${data.orderId}`
    );
  }

  @EventPattern("order_cancelled")
  async handleOrderCancelled(
    @Payload() data: { orderId: string; userId: string }
  ): Promise<void> {
    console.log("주문 취소 이벤트 수신:", data);

    await this.emailService.sendOrderCancellation(data.userId, data.orderId);
  }
}

// main.ts (Consumer 서비스)
import { NestFactory } from "@nestjs/core";
import { Transport, MicroserviceOptions } from "@nestjs/microservices";

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    NotificationsModule,
    {
      transport: Transport.RMQ,
      options: {
        urls: ["amqp://localhost:5672"],
        queue: "notifications_queue",
        queueOptions: { durable: true },
        noAck: false, // 수동 ACK
      },
    }
  );

  await app.listen();
  console.log("Notification 서비스 시작");
}
bootstrap();
```

<br>

## 🏷️ Kafka + NestJS

<br>

### 1. 설치 및 설정

<br>

```bash
# Kafka Docker Compose
# docker-compose.yml
version: '3'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
    ports:
      - "2181:2181"

  kafka:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
```

```bash
# NestJS Kafka 패키지
npm install @nestjs/microservices kafkajs
```

<br>

### 2. Producer 구현

<br>

```typescript
// src/orders/orders.module.ts
import { Module } from "@nestjs/common";
import { ClientsModule, Transport } from "@nestjs/microservices";

@Module({
  imports: [
    ClientsModule.register([
      {
        name: "KAFKA_SERVICE",
        transport: Transport.KAFKA,
        options: {
          client: {
            clientId: "orders",
            brokers: ["localhost:9092"],
          },
          producer: {
            allowAutoTopicCreation: true,
          },
        },
      },
    ]),
  ],
})
export class OrdersModule {}

// src/orders/orders.service.ts
@Injectable()
export class OrdersService {
  constructor(
    @Inject("KAFKA_SERVICE")
    private readonly kafkaClient: ClientKafka
  ) {}

  async onModuleInit() {
    // 토픽 구독 (응답 받을 경우)
    this.kafkaClient.subscribeToResponseOf("order.created");
    await this.kafkaClient.connect();
  }

  async createOrder(dto: CreateOrderDto): Promise<Order> {
    const order = await this.orderRepository.save({
      userId: dto.userId,
      items: dto.items,
      status: "PENDING",
    });

    // Kafka 메시지 발행
    this.kafkaClient.emit("order.created", {
      key: order.id, // 파티션 키
      value: {
        orderId: order.id,
        userId: order.userId,
        items: order.items,
        totalAmount: order.totalAmount,
        createdAt: new Date().toISOString(),
      },
    });

    return order;
  }
}
```

<br>

### 3. Consumer 구현

<br>

```typescript
// src/inventory/inventory.controller.ts
import { Controller } from "@nestjs/common";
import { MessagePattern, Payload } from "@nestjs/microservices";

@Controller()
export class InventoryController {
  constructor(private readonly inventoryService: InventoryService) {}

  @MessagePattern("order.created")
  async handleOrderCreated(
    @Payload()
    message: {
      orderId: string;
      items: Array<{ productId: string; quantity: number }>;
    }
  ): Promise<void> {
    console.log("재고 차감 이벤트 수신:", message.orderId);

    // 재고 차감
    for (const item of message.items) {
      await this.inventoryService.decreaseStock(item.productId, item.quantity);
    }
  }
}

// main.ts (Consumer 서비스)
async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    InventoryModule,
    {
      transport: Transport.KAFKA,
      options: {
        client: {
          brokers: ["localhost:9092"],
          clientId: "inventory-consumer",
        },
        consumer: {
          groupId: "inventory-group",
        },
      },
    }
  );

  await app.listen();
}
```

<br>

## 🏷️ 에러 처리와 재시도

<br>

```typescript
// Dead Letter Queue (DLQ) 패턴
@Injectable()
export class OrderEventHandler {
  private readonly MAX_RETRIES = 3;

  @EventPattern("order.created")
  async handleOrderCreated(@Payload() data: OrderCreatedEvent): Promise<void> {
    try {
      await this.processOrder(data);
    } catch (error) {
      // 재시도 횟수 확인
      const retryCount = data.retryCount || 0;

      if (retryCount < this.MAX_RETRIES) {
        // 재시도
        this.kafkaClient.emit("order.created", {
          ...data,
          retryCount: retryCount + 1,
        });
      } else {
        // DLQ로 이동
        this.kafkaClient.emit("order.created.dlq", {
          ...data,
          error: error.message,
          failedAt: new Date().toISOString(),
        });
      }
    }
  }
}

// 멱등성 보장
@Injectable()
export class IdempotentHandler {
  constructor(private readonly redis: Redis) {}

  async handleMessage(messageId: string, handler: () => Promise<void>) {
    // 이미 처리된 메시지인지 확인
    const processed = await this.redis.get(`processed:${messageId}`);
    if (processed) {
      console.log("이미 처리된 메시지:", messageId);
      return;
    }

    // 처리
    await handler();

    // 처리 완료 표시 (24시간 유지)
    await this.redis.set(`processed:${messageId}`, "1", "EX", 86400);
  }
}
```

<br>

## 🏷️ 실전 아키텍처

<br>

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   주문 API   │────→│   Kafka     │────→│  재고 서비스  │
└─────────────┘     │             │     └─────────────┘
                    │  order.     │
                    │  created    │────→┌─────────────┐
                    │             │     │  알림 서비스  │
                    └─────────────┘     └─────────────┘
                           │
                           ↓
                    ┌─────────────┐
                    │  분석 서비스  │
                    └─────────────┘
```

<br>

```typescript
// 이벤트 기반 주문 처리 흐름
// 1. 주문 생성 → order.created 발행
// 2. 재고 서비스 → 재고 차감 → inventory.updated 발행
// 3. 결제 서비스 → 결제 처리 → payment.completed 발행
// 4. 알림 서비스 → 이메일/푸시 발송
// 5. 분석 서비스 → 데이터 저장
```

<br>

# 실전 체크리스트

<br>

## ✅ 메시지 설계

<br>

- [ ] 메시지 스키마 정의
- [ ] 버전 관리 고려
- [ ] 필요한 정보만 포함
- [ ] 멱등성 키 포함

<br>

## ✅ 신뢰성

<br>

- [ ] 메시지 영속성 설정
- [ ] ACK 전략 결정
- [ ] 재시도 로직 구현
- [ ] DLQ 설정

<br>

## ✅ 성능

<br>

- [ ] 파티션 수 결정 (Kafka)
- [ ] Consumer Group 설계
- [ ] 배치 처리 고려
- [ ] 모니터링 설정

<br>

## ✅ 운영

<br>

- [ ] 메시지 추적 가능
- [ ] 알림 설정
- [ ] 백업 전략
- [ ] 장애 대응 계획

<br>

# 요약

<br>
메시지 큐는 시스템 간 비동기 통신을 가능하게 하는 핵심 인프라다. <br><br>

**💎 핵심 포인트**:

1. **Producer**: 메시지 발행
2. **Queue/Topic**: 메시지 저장
3. **Consumer**: 메시지 처리
4. **P2P**: 하나의 Consumer만 처리
5. **Pub/Sub**: 모든 구독자가 수신
6. **ACK**: 처리 완료 확인

<br>

**📌 선택 기준**:

| 상황                | 권장       |
| ------------------- | ---------- |
| **작업 큐**         | RabbitMQ   |
| **이벤트 스트리밍** | Kafka      |
| **실시간 알림**     | RabbitMQ   |
| **로그 수집**       | Kafka      |
| **마이크로서비스**  | 둘 다 가능 |

<br>

**🚀 Best Practices**:

- 멱등성 보장 필수
- DLQ로 실패 메시지 관리
- 메시지 스키마 버전 관리
- Consumer Group으로 확장
- 모니터링과 알림 설정
- 재시도 전략 수립

<br>

메시지 큐를 잘 활용하면, <br>
시스템 간 결합도를 낮추고, <br>
확장성과 안정성을 높일 수 있다. <br><br>
