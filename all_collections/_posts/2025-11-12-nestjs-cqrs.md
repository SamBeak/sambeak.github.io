---
layout: post
title: 🚀 NestJS + CQRS 완벽 가이드 (Command, Query, Event Sourcing)
date: 2025-11-12
categories:
  ["SamBeak", "NestJS", "CQRS", "Architecture", "TypeScript", "Backend"]
---

# CQRS란 무엇인가

<br>
식당을 상상해보자. <br>
주문받는 직원과 음식 서빙하는 직원이 따로 있다. <br><br>

왜 분리할까? <br>
주문은 빠르게, 서빙은 정확하게 해야 하기 때문이다. <br><br>

**CQRS(Command Query Responsibility Segregation)**는 <br>
데이터를 **변경하는 작업(Command)**과 <br>
데이터를 **조회하는 작업(Query)**을 분리하는 패턴이다. <br><br>

마치 식당에서 역할을 나누듯, <br>
시스템도 쓰기와 읽기를 분리하면 <br>
각각 최적화할 수 있다. <br><br>

> ## 왜 CQRS를 배워야 할까?

<br>

**이유 1: 성능 최적화** <br>
읽기와 쓰기를 각각 최적화 가능 <br><br>

**이유 2: 확장성** <br>
읽기 서버와 쓰기 서버를 독립적으로 확장 <br><br>

**이유 3: 복잡한 도메인 처리** <br>
DDD와 함께 사용하면 강력함 <br><br>

**이유 4: 면접 필수** <br>
CQRS, Event Sourcing은 시니어 면접 단골 <br><br>

# 기본 개념 요약

<br>

## 🏷️ CQRS 핵심 구조

<br>

```
[클라이언트]
    │
    ├── Command (쓰기) ──→ [Command Handler] ──→ [Write DB]
    │                              │
    │                              ↓ (이벤트 발행)
    │                         [Event Bus]
    │                              │
    │                              ↓
    └── Query (읽기) ───→ [Query Handler] ───→ [Read DB]
```

<br>

### Command (명령)

<br>
**개념**: 데이터를 변경하는 요청 <br><br>

**특징**:

- 상태를 변경함
- 반환값 없거나 최소화
- 예: 주문 생성, 결제 처리

<br>

### Query (조회)

<br>
**개념**: 데이터를 조회하는 요청 <br><br>

**특징**:

- 상태를 변경하지 않음
- 데이터를 반환
- 예: 주문 목록 조회, 상품 검색

<br>

### Event (이벤트)

<br>
**개념**: 시스템에서 발생한 사실 <br><br>

**특징**:

- 과거형으로 명명 (OrderCreated)
- 불변
- 다른 핸들러가 반응

<br>

## 🏷️ 전통적 CRUD vs CQRS

<br>

| 구분            | CRUD      | CQRS           |
| --------------- | --------- | -------------- |
| **모델**        | 단일 모델 | 읽기/쓰기 분리 |
| **DB**          | 하나의 DB | 분리 가능      |
| **복잡도**      | 단순      | 복잡           |
| **확장성**      | 제한적    | 높음           |
| **적합한 경우** | 단순 CRUD | 복잡한 도메인  |

<br>

## 🏷️ NestJS CQRS 모듈 구조

<br>

```
src/
├── orders/
│   ├── commands/
│   │   ├── impl/
│   │   │   └── create-order.command.ts
│   │   └── handlers/
│   │       └── create-order.handler.ts
│   ├── queries/
│   │   ├── impl/
│   │   │   └── get-order.query.ts
│   │   └── handlers/
│   │       └── get-order.handler.ts
│   ├── events/
│   │   ├── impl/
│   │   │   └── order-created.event.ts
│   │   └── handlers/
│   │       └── order-created.handler.ts
│   ├── dto/
│   ├── entities/
│   └── orders.module.ts
```

<br>

# 실전 예시

<br>

## 🏷️ 프로젝트 설정

<br>

```bash
# NestJS 프로젝트 생성
npm i -g @nestjs/cli
nest new nestjs-cqrs-demo

# CQRS 패키지 설치
npm install @nestjs/cqrs

# TypeORM 설치 (선택)
npm install @nestjs/typeorm typeorm pg
```

<br>

## 🏷️ Command 구현

<br>

### 1. Command 정의

<br>

```typescript
// src/orders/commands/impl/create-order.command.ts
export class CreateOrderCommand {
  constructor(
    public readonly userId: string,
    public readonly items: OrderItemDto[],
    public readonly shippingAddress: string
  ) {}
}

// src/orders/commands/impl/cancel-order.command.ts
export class CancelOrderCommand {
  constructor(
    public readonly orderId: string,
    public readonly reason: string
  ) {}
}
```

<br>

### 2. Command Handler 구현

<br>

```typescript
// src/orders/commands/handlers/create-order.handler.ts
import { CommandHandler, ICommandHandler, EventBus } from "@nestjs/cqrs";
import { CreateOrderCommand } from "../impl/create-order.command";
import { OrderCreatedEvent } from "../../events/impl/order-created.event";
import { OrderRepository } from "../../repositories/order.repository";
import { Order } from "../../entities/order.entity";

@CommandHandler(CreateOrderCommand)
export class CreateOrderHandler implements ICommandHandler<CreateOrderCommand> {
  constructor(
    private readonly orderRepository: OrderRepository,
    private readonly eventBus: EventBus
  ) {}

  async execute(command: CreateOrderCommand): Promise<string> {
    const { userId, items, shippingAddress } = command;

    // 1. 주문 생성
    const order = Order.create({
      userId,
      items,
      shippingAddress,
      status: "PENDING",
    });

    // 2. 저장
    await this.orderRepository.save(order);

    // 3. 이벤트 발행
    this.eventBus.publish(
      new OrderCreatedEvent(order.id, order.userId, order.totalAmount)
    );

    return order.id;
  }
}

// src/orders/commands/handlers/cancel-order.handler.ts
@CommandHandler(CancelOrderCommand)
export class CancelOrderHandler implements ICommandHandler<CancelOrderCommand> {
  constructor(
    private readonly orderRepository: OrderRepository,
    private readonly eventBus: EventBus
  ) {}

  async execute(command: CancelOrderCommand): Promise<void> {
    const { orderId, reason } = command;

    // 1. 주문 조회
    const order = await this.orderRepository.findById(orderId);
    if (!order) {
      throw new NotFoundException("주문을 찾을 수 없습니다");
    }

    // 2. 취소 가능 여부 확인
    if (order.status === "SHIPPED") {
      throw new BadRequestException("배송된 주문은 취소할 수 없습니다");
    }

    // 3. 주문 취소
    order.cancel(reason);
    await this.orderRepository.save(order);

    // 4. 이벤트 발행
    this.eventBus.publish(new OrderCancelledEvent(orderId, reason));
  }
}
```

<br>

## 🏷️ Query 구현

<br>

### 1. Query 정의

<br>

```typescript
// src/orders/queries/impl/get-order.query.ts
export class GetOrderQuery {
  constructor(public readonly orderId: string) {}
}

// src/orders/queries/impl/get-orders-by-user.query.ts
export class GetOrdersByUserQuery {
  constructor(
    public readonly userId: string,
    public readonly page: number = 1,
    public readonly limit: number = 10
  ) {}
}
```

<br>

### 2. Query Handler 구현

<br>

```typescript
// src/orders/queries/handlers/get-order.handler.ts
import { IQueryHandler, QueryHandler } from "@nestjs/cqrs";
import { GetOrderQuery } from "../impl/get-order.query";
import { OrderReadRepository } from "../../repositories/order-read.repository";

@QueryHandler(GetOrderQuery)
export class GetOrderHandler implements IQueryHandler<GetOrderQuery> {
  constructor(private readonly orderReadRepository: OrderReadRepository) {}

  async execute(query: GetOrderQuery): Promise<OrderDto> {
    const { orderId } = query;

    const order = await this.orderReadRepository.findById(orderId);
    if (!order) {
      throw new NotFoundException("주문을 찾을 수 없습니다");
    }

    return OrderDto.from(order);
  }
}

// src/orders/queries/handlers/get-orders-by-user.handler.ts
@QueryHandler(GetOrdersByUserQuery)
export class GetOrdersByUserHandler
  implements IQueryHandler<GetOrdersByUserQuery>
{
  constructor(private readonly orderReadRepository: OrderReadRepository) {}

  async execute(query: GetOrdersByUserQuery): Promise<PaginatedOrdersDto> {
    const { userId, page, limit } = query;

    const [orders, total] = await this.orderReadRepository.findByUserId(
      userId,
      page,
      limit
    );

    return {
      data: orders.map(OrderDto.from),
      total,
      page,
      limit,
      totalPages: Math.ceil(total / limit),
    };
  }
}
```

<br>

## 🏷️ Event 구현

<br>

### 1. Event 정의

<br>

```typescript
// src/orders/events/impl/order-created.event.ts
export class OrderCreatedEvent {
  constructor(
    public readonly orderId: string,
    public readonly userId: string,
    public readonly totalAmount: number
  ) {}
}

// src/orders/events/impl/order-cancelled.event.ts
export class OrderCancelledEvent {
  constructor(
    public readonly orderId: string,
    public readonly reason: string
  ) {}
}
```

<br>

### 2. Event Handler 구현

<br>

```typescript
// src/orders/events/handlers/order-created.handler.ts
import { EventsHandler, IEventHandler } from "@nestjs/cqrs";
import { OrderCreatedEvent } from "../impl/order-created.event";

@EventsHandler(OrderCreatedEvent)
export class OrderCreatedHandler implements IEventHandler<OrderCreatedEvent> {
  constructor(
    private readonly notificationService: NotificationService,
    private readonly analyticsService: AnalyticsService
  ) {}

  async handle(event: OrderCreatedEvent): Promise<void> {
    const { orderId, userId, totalAmount } = event;

    // 1. 알림 전송
    await this.notificationService.sendOrderConfirmation(userId, orderId);

    // 2. 분석 데이터 저장
    await this.analyticsService.trackOrder(orderId, totalAmount);

    console.log(`주문 생성됨: ${orderId}, 금액: ${totalAmount}`);
  }
}

// src/orders/events/handlers/order-cancelled.handler.ts
@EventsHandler(OrderCancelledEvent)
export class OrderCancelledHandler
  implements IEventHandler<OrderCancelledEvent>
{
  constructor(
    private readonly inventoryService: InventoryService,
    private readonly paymentService: PaymentService
  ) {}

  async handle(event: OrderCancelledEvent): Promise<void> {
    const { orderId, reason } = event;

    // 1. 재고 복구
    await this.inventoryService.restoreStock(orderId);

    // 2. 환불 처리
    await this.paymentService.refund(orderId);

    console.log(`주문 취소됨: ${orderId}, 사유: ${reason}`);
  }
}
```

<br>

## 🏷️ Controller 구현

<br>

```typescript
// src/orders/orders.controller.ts
import { Controller, Post, Get, Body, Param, Query } from "@nestjs/common";
import { CommandBus, QueryBus } from "@nestjs/cqrs";
import { CreateOrderCommand } from "./commands/impl/create-order.command";
import { CancelOrderCommand } from "./commands/impl/cancel-order.command";
import { GetOrderQuery } from "./queries/impl/get-order.query";
import { GetOrdersByUserQuery } from "./queries/impl/get-orders-by-user.query";

@Controller("orders")
export class OrdersController {
  constructor(
    private readonly commandBus: CommandBus,
    private readonly queryBus: QueryBus
  ) {}

  // Command: 주문 생성
  @Post()
  async createOrder(@Body() dto: CreateOrderDto): Promise<{ orderId: string }> {
    const orderId = await this.commandBus.execute(
      new CreateOrderCommand(dto.userId, dto.items, dto.shippingAddress)
    );
    return { orderId };
  }

  // Command: 주문 취소
  @Post(":id/cancel")
  async cancelOrder(
    @Param("id") orderId: string,
    @Body() dto: CancelOrderDto
  ): Promise<void> {
    await this.commandBus.execute(new CancelOrderCommand(orderId, dto.reason));
  }

  // Query: 주문 조회
  @Get(":id")
  async getOrder(@Param("id") orderId: string): Promise<OrderDto> {
    return this.queryBus.execute(new GetOrderQuery(orderId));
  }

  // Query: 사용자 주문 목록
  @Get("user/:userId")
  async getOrdersByUser(
    @Param("userId") userId: string,
    @Query("page") page: number = 1,
    @Query("limit") limit: number = 10
  ): Promise<PaginatedOrdersDto> {
    return this.queryBus.execute(new GetOrdersByUserQuery(userId, page, limit));
  }
}
```

<br>

## 🏷️ Module 설정

<br>

```typescript
// src/orders/orders.module.ts
import { Module } from "@nestjs/common";
import { CqrsModule } from "@nestjs/cqrs";
import { TypeOrmModule } from "@nestjs/typeorm";
import { OrdersController } from "./orders.controller";
import { Order } from "./entities/order.entity";

// Command Handlers
import { CreateOrderHandler } from "./commands/handlers/create-order.handler";
import { CancelOrderHandler } from "./commands/handlers/cancel-order.handler";

// Query Handlers
import { GetOrderHandler } from "./queries/handlers/get-order.handler";
import { GetOrdersByUserHandler } from "./queries/handlers/get-orders-by-user.handler";

// Event Handlers
import { OrderCreatedHandler } from "./events/handlers/order-created.handler";
import { OrderCancelledHandler } from "./events/handlers/order-cancelled.handler";

// Repositories
import { OrderRepository } from "./repositories/order.repository";
import { OrderReadRepository } from "./repositories/order-read.repository";

const CommandHandlers = [CreateOrderHandler, CancelOrderHandler];
const QueryHandlers = [GetOrderHandler, GetOrdersByUserHandler];
const EventHandlers = [OrderCreatedHandler, OrderCancelledHandler];

@Module({
  imports: [CqrsModule, TypeOrmModule.forFeature([Order])],
  controllers: [OrdersController],
  providers: [
    ...CommandHandlers,
    ...QueryHandlers,
    ...EventHandlers,
    OrderRepository,
    OrderReadRepository,
  ],
})
export class OrdersModule {}
```

<br>

## 🏷️ Entity와 Repository

<br>

```typescript
// src/orders/entities/order.entity.ts
import { AggregateRoot } from "@nestjs/cqrs";

@Entity()
export class Order extends AggregateRoot {
  @PrimaryGeneratedColumn("uuid")
  id: string;

  @Column()
  userId: string;

  @Column("jsonb")
  items: OrderItem[];

  @Column()
  shippingAddress: string;

  @Column()
  status: OrderStatus;

  @Column("decimal")
  totalAmount: number;

  @CreateDateColumn()
  createdAt: Date;

  static create(props: CreateOrderProps): Order {
    const order = new Order();
    order.userId = props.userId;
    order.items = props.items;
    order.shippingAddress = props.shippingAddress;
    order.status = "PENDING";
    order.totalAmount = props.items.reduce(
      (sum, item) => sum + item.price * item.quantity,
      0
    );
    return order;
  }

  cancel(reason: string): void {
    if (this.status === "SHIPPED") {
      throw new Error("배송된 주문은 취소 불가");
    }
    this.status = "CANCELLED";
  }
}

// src/orders/repositories/order.repository.ts
@Injectable()
export class OrderRepository {
  constructor(
    @InjectRepository(Order)
    private readonly repository: Repository<Order>
  ) {}

  async save(order: Order): Promise<Order> {
    return this.repository.save(order);
  }

  async findById(id: string): Promise<Order | null> {
    return this.repository.findOne({ where: { id } });
  }
}

// src/orders/repositories/order-read.repository.ts
// 읽기 전용 - 최적화된 쿼리
@Injectable()
export class OrderReadRepository {
  constructor(
    @InjectRepository(Order)
    private readonly repository: Repository<Order>
  ) {}

  async findById(id: string): Promise<Order | null> {
    return this.repository
      .createQueryBuilder("order")
      .select(["order.id", "order.status", "order.totalAmount"])
      .where("order.id = :id", { id })
      .getOne();
  }

  async findByUserId(
    userId: string,
    page: number,
    limit: number
  ): Promise<[Order[], number]> {
    return this.repository.findAndCount({
      where: { userId },
      skip: (page - 1) * limit,
      take: limit,
      order: { createdAt: "DESC" },
    });
  }
}
```

<br>

## 🏷️ Event Sourcing (선택)

<br>

```typescript
// src/orders/events/order-event-store.ts
@Injectable()
export class OrderEventStore {
  constructor(
    @InjectRepository(OrderEvent)
    private readonly repository: Repository<OrderEvent>
  ) {}

  async save(event: DomainEvent): Promise<void> {
    const orderEvent = new OrderEvent();
    orderEvent.aggregateId = event.aggregateId;
    orderEvent.eventType = event.constructor.name;
    orderEvent.payload = JSON.stringify(event);
    orderEvent.occurredAt = new Date();

    await this.repository.save(orderEvent);
  }

  async getEvents(aggregateId: string): Promise<DomainEvent[]> {
    const events = await this.repository.find({
      where: { aggregateId },
      order: { occurredAt: "ASC" },
    });

    return events.map((e) => JSON.parse(e.payload));
  }

  // 이벤트로부터 상태 복원
  async rehydrate(aggregateId: string): Promise<Order> {
    const events = await this.getEvents(aggregateId);
    const order = new Order();

    events.forEach((event) => {
      order.apply(event);
    });

    return order;
  }
}
```

<br>

# 실전 체크리스트

<br>

## ✅ Command 설계

<br>

- [ ] Command는 의도를 명확히 표현
- [ ] 필요한 데이터만 포함
- [ ] 반환값 최소화
- [ ] 유효성 검사 포함

<br>

## ✅ Query 설계

<br>

- [ ] 읽기 전용 Repository 분리
- [ ] 필요한 필드만 조회
- [ ] 페이지네이션 적용
- [ ] 캐싱 고려

<br>

## ✅ Event 설계

<br>

- [ ] 과거형 이름 사용
- [ ] 불변 객체로 설계
- [ ] 필요한 정보만 포함
- [ ] 비동기 처리 고려

<br>

## ✅ 아키텍처

<br>

- [ ] 모듈별 분리
- [ ] Handler 단일 책임
- [ ] 의존성 주입 활용
- [ ] 테스트 용이성 확보

<br>

# 요약

<br>
CQRS는 읽기와 쓰기를 분리하여 각각 최적화하는 패턴이다. <br><br>

**💎 핵심 포인트**:

1. **Command**: 상태 변경 요청
2. **Query**: 데이터 조회 요청
3. **Event**: 발생한 사실 알림
4. **Handler**: 각 요청 처리
5. **EventBus**: 이벤트 전달
6. **분리된 Repository**: 읽기/쓰기 최적화

<br>

**📌 NestJS CQRS 흐름**:

| 단계                       | 설명                  |
| -------------------------- | --------------------- |
| **1. Controller**          | 요청 수신             |
| **2. CommandBus/QueryBus** | 적절한 Handler로 전달 |
| **3. Handler**             | 비즈니스 로직 실행    |
| **4. EventBus**            | 이벤트 발행           |
| **5. EventHandler**        | 부가 작업 처리        |

<br>

**🚀 Best Practices**:

- 단순 CRUD는 CQRS 불필요
- 복잡한 도메인에 적용
- Event Sourcing은 선택
- 읽기 모델 캐싱 활용
- 비동기 이벤트 처리
- 테스트 코드 필수

<br>

CQRS는 복잡한 도메인에서 빛을 발한다. <br>
단순한 CRUD 앱에는 과할 수 있지만, <br>
확장성과 유지보수성이 중요하다면 고려하자. <br><br>
