# NestJS Microservices - Top Interview Questions

## Microservices Fundamentals

### 1. What are Microservices?

<details>
<summary>Answer</summary>

**Microservices** is an architectural pattern where an application is composed of **small, independent services** that communicate over a network. Each service is self-contained, owns its data, and can be deployed independently.

**Key Characteristics:**

```
1. Independence ✅
   - Each service runs independently
   - Can be deployed separately
   - Own database per service
   
2. Communication ✅
   - Services communicate via network (HTTP, TCP, messaging)
   - Use well-defined APIs
   
3. Technology Agnostic ✅
   - Each service can use different tech stack
   - Choose best tool for the job
   
4. Scalability ✅
   - Scale services independently
   - Only scale what needs scaling
```

**Microservices Architecture Example:**

```
┌─────────────────────────────────────────────────────────┐
│                     API Gateway                          │
│            (Routes requests to services)                 │
└─────────┬──────────┬──────────┬──────────┬──────────────┘
          │          │          │          │
    ┌─────▼────┐ ┌──▼──────┐ ┌─▼────────┐ ┌▼──────────┐
    │  Auth    │ │  Users  │ │ Orders   │ │ Payments  │
    │ Service  │ │ Service │ │ Service  │ │ Service   │
    │  :3001   │ │  :3002  │ │  :3003   │ │  :3004    │
    └────┬─────┘ └────┬────┘ └────┬─────┘ └────┬──────┘
         │            │           │             │
    ┌────▼─────┐ ┌───▼─────┐ ┌───▼──────┐ ┌───▼───────┐
    │Auth DB   │ │Users DB │ │Orders DB │ │Payments DB│
    └──────────┘ └─────────┘ └──────────┘ └───────────┘
```

**Simple NestJS Microservice Example:**

```typescript
// main.ts - Auth Microservice
import { NestFactory } from '@nestjs/core';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';
import { AppModule } from './app.module';

async function bootstrap() {
  // Create microservice with TCP transport
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.TCP,
      options: {
        host: 'localhost',
        port: 3001,
      },
    },
  );

  await app.listen();
  console.log('Auth Microservice is listening on port 3001');
}
bootstrap();
```

```typescript
// auth.controller.ts
import { Controller } from '@nestjs/common';
import { MessagePattern } from '@nestjs/microservices';

@Controller()
export class AuthController {
  // Handle 'validate-token' message pattern
  @MessagePattern('validate-token')
  async validateToken(data: { token: string }) {
    console.log('Received token validation request:', data.token);
    
    // Validate token logic
    const isValid = this.validateJWT(data.token);
    
    return {
      valid: isValid,
      userId: isValid ? '123' : null,
    };
  }

  // Handle 'login' message pattern
  @MessagePattern('login')
  async login(data: { email: string; password: string }) {
    console.log('Received login request for:', data.email);
    
    // Authentication logic
    if (data.email === 'user@example.com' && data.password === 'password') {
      return {
        success: true,
        token: 'jwt-token-here',
        userId: '123',
      };
    }
    
    return {
      success: false,
      message: 'Invalid credentials',
    };
  }

  private validateJWT(token: string): boolean {
    // JWT validation logic
    return token.length > 10;
  }
}
```

**Consuming Microservice (API Gateway):**

```typescript
// app.module.ts - API Gateway
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';
import { AuthModule } from './auth/auth.module';

@Module({
  imports: [
    // Register microservice client
    ClientsModule.register([
      {
        name: 'AUTH_SERVICE',
        transport: Transport.TCP,
        options: {
          host: 'localhost',
          port: 3001,
        },
      },
    ]),
    AuthModule,
  ],
})
export class AppModule {}
```

```typescript
// auth.service.ts - API Gateway
import { Injectable, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { firstValueFrom } from 'rxjs';

@Injectable()
export class AuthService {
  constructor(
    @Inject('AUTH_SERVICE') private authClient: ClientProxy,
  ) {}

  async login(email: string, password: string) {
    // Send message to auth microservice
    const response = await firstValueFrom(
      this.authClient.send('login', { email, password }),
    );
    
    return response;
  }

  async validateToken(token: string) {
    // Send message to auth microservice
    const response = await firstValueFrom(
      this.authClient.send('validate-token', { token }),
    );
    
    return response;
  }
}
```

**Real-World Microservices Example (E-commerce):**

```typescript
// Order Service
@Controller()
export class OrderController {
  @MessagePattern('create-order')
  async createOrder(data: CreateOrderDto) {
    // 1. Validate user (call User Service)
    // 2. Check inventory (call Inventory Service)
    // 3. Process payment (call Payment Service)
    // 4. Create order
    // 5. Send confirmation (call Notification Service)
    
    return {
      orderId: 'ORDER-123',
      status: 'confirmed',
      total: 99.99,
    };
  }

  @MessagePattern('get-order')
  async getOrder(data: { orderId: string }) {
    return {
      orderId: data.orderId,
      items: [...],
      total: 99.99,
      status: 'shipped',
    };
  }
}
```

**Benefits of Microservices:**

| Benefit | Description | Example |
|---------|-------------|---------|
| **Independent Deployment** | Deploy services separately | Update payment service without touching orders |
| **Scalability** | Scale only what needs scaling | Scale product search, not admin panel |
| **Technology Flexibility** | Different tech per service | Python for ML, Node.js for API |
| **Fault Isolation** | Failure in one service doesn't crash all | Payment service down, users can still browse |
| **Team Autonomy** | Teams own services end-to-end | Payment team owns payment service completely |
| **Faster Development** | Smaller codebases, faster iteration | Ship features faster |

**Challenges of Microservices:**

| Challenge | Description | Mitigation |
|-----------|-------------|------------|
| **Complexity** | More services = more complexity | Use service mesh, API gateway |
| **Network Latency** | Inter-service calls over network | Cache, optimize calls |
| **Data Consistency** | Distributed data | Eventual consistency, sagas |
| **Debugging** | Harder to trace requests | Distributed tracing, logging |
| **Testing** | Integration testing complex | Contract testing, mocks |
| **DevOps Overhead** | More deployment pipelines | Docker, Kubernetes, CI/CD |

**When to Use Microservices:**

```typescript
// ✅ GOOD: Use microservices when:
// - Large application with clear bounded contexts
// - Need to scale parts independently
// - Multiple teams working on different features
// - Different tech stacks needed for different services
// - High traffic requiring horizontal scaling

// ❌ BAD: Don't use microservices when:
// - Small application (< 5 developers)
// - Unclear requirements
// - Tight deadlines (monolith faster initially)
// - Limited DevOps resources
// - Simple CRUD application
```

**Microservices vs API:**

```typescript
// Microservices are NOT just multiple APIs
// They are autonomous services with:

// 1. Own Database
@Module({
  imports: [
    TypeOrmModule.forRoot({
      database: 'orders_db',  // Own database
    }),
  ],
})
export class OrderServiceModule {}

// 2. Business Logic
export class OrderService {
  // Complete order management logic
  async createOrder() { /* ... */ }
  async cancelOrder() { /* ... */ }
  async calculateTax() { /* ... */ }
}

// 3. Independent Deployment
// Can deploy orders service without touching other services

// 4. Own Data Model
@Entity()
export class Order {
  @PrimaryGeneratedColumn()
  id: number;
  
  // Order-specific fields
}
```

**Interview Tip**: **Microservices** are **small, independent services** that communicate over a network. Each service: **owns its data**, **can be deployed independently**, **handles specific business domain**. Built with **NestJS @nestjs/microservices** package using transports like **TCP, Redis, RabbitMQ, Kafka, gRPC**. Use **@MessagePattern()** to handle requests, **ClientProxy** to send messages. **Benefits**: independent deployment, scalability, fault isolation. **Challenges**: complexity, network latency, debugging. Use for **large apps** with clear domains, **avoid for small apps**.

</details>

### 2. What is the difference between Monolith and Microservices architecture?

<details>
<summary>Answer</summary>

**Monolith** is a single, unified application where all components are tightly coupled. **Microservices** split the application into small, independent services that communicate over a network.

**Architecture Comparison:**

```
MONOLITH                           MICROSERVICES
┌─────────────────────────┐       ┌──────┐ ┌──────┐ ┌──────┐
│   Single Application    │       │ Auth │ │Users │ │Orders│
│                         │       │  :1  │ │  :2  │ │  :3  │
│  ┌──────────────────┐  │       └──┬───┘ └──┬───┘ └──┬───┘
│  │   Auth Module    │  │          │        │        │
│  ├──────────────────┤  │       ┌──▼────┐┌──▼────┐┌──▼────┐
│  │   Users Module   │  │       │Auth DB││User DB││Ord DB │
│  ├──────────────────┤  │       └───────┘└───────┘└───────┘
│  │   Orders Module  │  │
│  ├──────────────────┤  │       Network Calls (HTTP, TCP)
│  │  Payments Module │  │       Independent Deployment
│  └──────────────────┘  │       Own Databases
│          │             │
│    ┌─────▼─────┐      │
│    │  Database │      │
│    └───────────┘      │
└─────────────────────────┘
   In-Process Calls
   Single Deployment
   Shared Database
```

**Detailed Comparison Table:**

| Aspect | Monolith | Microservices |
|--------|----------|---------------|
| **Architecture** | Single unified application | Multiple independent services |
| **Deployment** | Deploy entire application | Deploy services independently |
| **Database** | Single shared database | Database per service (polyglot) |
| **Communication** | In-process (method calls) | Network (HTTP, gRPC, messaging) |
| **Scaling** | Scale entire app | Scale services independently |
| **Technology** | Single tech stack | Different stack per service |
| **Development** | Simple to start | Complex coordination |
| **Testing** | Easier (all in one place) | Harder (integration testing) |
| **Failure** | One bug can crash all | Isolated failures |
| **Team Size** | Single team | Multiple teams per service |
| **Data Consistency** | ACID transactions | Eventual consistency |
| **Performance** | Fast (in-process) | Slower (network calls) |
| **DevOps** | Simple CI/CD | Complex (multiple pipelines) |

**Monolithic Architecture Example:**

```typescript
// main.ts - Single Application
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
  console.log('Monolith running on port 3000');
}
bootstrap();
```

```typescript
// app.module.ts - All modules in one app
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { AuthModule } from './auth/auth.module';
import { UsersModule } from './users/users.module';
import { OrdersModule } from './orders/orders.module';
import { PaymentsModule } from './payments/payments.module';

@Module({
  imports: [
    // Single shared database
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      database: 'monolith_db',  // All data in one DB
      entities: [__dirname + '/**/*.entity{.ts,.js}'],
      synchronize: true,
    }),
    
    // All modules in single app
    AuthModule,
    UsersModule,
    OrdersModule,
    PaymentsModule,
  ],
})
export class AppModule {}
```

```typescript
// orders.service.ts - In-process calls
import { Injectable } from '@nestjs/common';
import { UsersService } from '../users/users.service';
import { PaymentsService } from '../payments/payments.service';

@Injectable()
export class OrdersService {
  constructor(
    // ✅ Direct injection (in-process)
    private usersService: UsersService,
    private paymentsService: PaymentsService,
  ) {}

  async createOrder(userId: string, amount: number) {
    // Direct method calls (fast, no network)
    const user = await this.usersService.findById(userId);
    const payment = await this.paymentsService.charge(user.id, amount);
    
    // All in same database transaction (ACID)
    return this.orderRepository.save({
      userId: user.id,
      paymentId: payment.id,
      amount,
    });
  }
}
```

**Microservices Architecture Example:**

```typescript
// auth-service/main.ts - Auth Microservice
import { NestFactory } from '@nestjs/core';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';
import { AuthModule } from './auth.module';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AuthModule,
    {
      transport: Transport.TCP,
      options: { host: 'localhost', port: 3001 },
    },
  );
  
  await app.listen();
  console.log('Auth Service on port 3001');
}
bootstrap();
```

```typescript
// users-service/main.ts - Users Microservice
async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    UsersModule,
    {
      transport: Transport.TCP,
      options: { host: 'localhost', port: 3002 },
    },
  );
  
  await app.listen();
  console.log('Users Service on port 3002');
}
bootstrap();
```

```typescript
// orders-service/main.ts - Orders Microservice
async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    OrdersModule,
    {
      transport: Transport.TCP,
      options: { host: 'localhost', port: 3003 },
    },
  );
  
  await app.listen();
  console.log('Orders Service on port 3003');
}
bootstrap();
```

```typescript
// orders-service/orders.service.ts - Network calls
import { Injectable, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { firstValueFrom } from 'rxjs';

@Injectable()
export class OrdersService {
  constructor(
    // ✅ Network communication via ClientProxy
    @Inject('USERS_SERVICE') private usersClient: ClientProxy,
    @Inject('PAYMENTS_SERVICE') private paymentsClient: ClientProxy,
  ) {}

  async createOrder(userId: string, amount: number) {
    // Network calls to other services (slower)
    const user = await firstValueFrom(
      this.usersClient.send('get-user', { userId }),
    );
    
    const payment = await firstValueFrom(
      this.paymentsClient.send('charge', { userId, amount }),
    );
    
    // Own database (isolated data)
    return this.orderRepository.save({
      userId: user.id,
      paymentId: payment.id,
      amount,
    });
  }
}
```

**Deployment Differences:**

```bash
# MONOLITH - Single Deployment
npm run build
docker build -t monolith:latest .
docker run -p 3000:3000 monolith:latest
# All features deployed together

# MICROSERVICES - Independent Deployments
# Deploy Auth Service
cd auth-service
docker build -t auth-service:latest .
kubectl apply -f auth-deployment.yaml

# Deploy Users Service
cd users-service
docker build -t users-service:latest .
kubectl apply -f users-deployment.yaml

# Deploy Orders Service (independently)
cd orders-service
docker build -t orders-service:latest .
kubectl apply -f orders-deployment.yaml
```

**Scaling Differences:**

```yaml
# MONOLITH - Scale Everything
# docker-compose.yml
services:
  monolith:
    image: monolith:latest
    deploy:
      replicas: 5  # 5 instances of ENTIRE app
    ports:
      - "3000-3004:3000"
```

```yaml
# MICROSERVICES - Scale Independently
# docker-compose.yml
services:
  auth-service:
    image: auth-service:latest
    deploy:
      replicas: 2  # Low traffic
  
  users-service:
    image: users-service:latest
    deploy:
      replicas: 3  # Medium traffic
  
  orders-service:
    image: orders-service:latest
    deploy:
      replicas: 10  # High traffic - scale only this!
  
  payments-service:
    image: payments-service:latest
    deploy:
      replicas: 5  # Critical service
```

**Data Management:**

```typescript
// MONOLITH - Shared Database
@Module({
  imports: [
    TypeOrmModule.forRoot({
      database: 'monolith_db',  // Single DB
      entities: [User, Order, Payment, Product],  // All entities
    }),
  ],
})

// ✅ ACID Transactions (easy)
@Transactional()
async createOrder() {
  const user = await this.userRepo.save(...);
  const order = await this.orderRepo.save(...);
  const payment = await this.paymentRepo.save(...);
  // All succeed or all fail (atomic)
}
```

```typescript
// MICROSERVICES - Database Per Service
// users-service/app.module.ts
@Module({
  imports: [
    TypeOrmModule.forRoot({
      database: 'users_db',  // Own DB
      entities: [User],  // Only user entities
    }),
  ],
})

// orders-service/app.module.ts
@Module({
  imports: [
    TypeOrmModule.forRoot({
      database: 'orders_db',  // Own DB
      entities: [Order],  // Only order entities
    }),
  ],
})

// ❌ No ACID Transactions (distributed)
// ✅ Use Saga Pattern or Eventual Consistency
async createOrder() {
  // Step 1: Create order
  const order = await this.orderRepo.save(...);
  
  // Step 2: Call payment service (can fail)
  try {
    const payment = await this.paymentsClient.send('charge', ...).toPromise();
  } catch (error) {
    // Compensating transaction
    await this.orderRepo.updateStatus(order.id, 'FAILED');
  }
}
```

**Evolution Path:**

```
Start: Monolith
    │
    ├─ Simple to build
    ├─ Fast development
    └─ Single deployment
    
    ↓ (App grows)
    
Problem: Scaling issues
    │
    ├─ Deploy entire app for small change
    ├─ Can't scale parts independently
    └─ Tight coupling
    
    ↓ (Refactor)
    
Transition: Modular Monolith
    │
    ├─ Modules with clear boundaries
    ├─ Prepare for microservices
    └─ Still single deployment
    
    ↓ (Extract services)
    
Target: Microservices
    │
    ├─ Independent services
    ├─ Scalable
    └─ Flexible
```

**Real-World Example (Netflix Evolution):**

```
2008: Monolith (DVD rental)
  │
  ├─ Single Java application
  └─ Oracle database
  
  ↓ (Growth problems)
  
2012: Transition to Microservices
  │
  ├─ 500+ microservices
  ├─ Independent deployment
  └─ AWS infrastructure
  
Result:
  ├─ Can deploy 1000+ times per day
  ├─ Scale independently
  └─ Global service
```

**When to Choose Which:**

```typescript
// Choose MONOLITH when:
// ✅ Small team (< 10 developers)
// ✅ Early stage startup
// ✅ Unclear requirements
// ✅ Simple application
// ✅ Limited resources
// ✅ Need fast time to market

// Choose MICROSERVICES when:
// ✅ Large team (> 20 developers)
// ✅ Need independent scaling
// ✅ Clear bounded contexts
// ✅ Multiple tech stacks needed
// ✅ High availability requirements
// ✅ Mature DevOps practices
```

**Interview Tip**: **Monolith** is a **single unified application** with shared database, in-process calls, single deployment. **Microservices** are **independent services** with own databases, network calls, independent deployment. **Monolith advantages**: simpler, faster development, ACID transactions, easier testing. **Microservices advantages**: independent scaling/deployment, fault isolation, technology flexibility. **Trade-offs**: microservices add complexity (network latency, distributed transactions, debugging). Start with **modular monolith**, evolve to microservices when needed. Netflix moved from monolith to 500+ microservices for scalability.

</details>

### 3. What transport layers does NestJS support (TCP, Redis, MQTT, NATS, RabbitMQ, Kafka, gRPC)?

<details>
<summary>Answer</summary>

**NestJS supports 7 transport layers** for microservice communication: **TCP**, **Redis**, **MQTT**, **NATS**, **RabbitMQ (AMQP)**, **Kafka**, and **gRPC**. Each has different use cases and characteristics.

**Transport Layers Overview:**

| Transport | Protocol | Use Case | Pattern | Performance |
|-----------|----------|----------|---------|-------------|
| **TCP** | TCP/IP | Default, simple | Request-Response | Fast |
| **Redis** | Pub/Sub | Cache-based messaging | Both | Fast |
| **MQTT** | MQTT | IoT devices, sensors | Pub/Sub | Light |
| **NATS** | NATS | Cloud-native, fast | Both | Very Fast |
| **RabbitMQ** | AMQP | Enterprise messaging | Both | Reliable |
| **Kafka** | Kafka Protocol | Event streaming, logs | Event-driven | High throughput |
| **gRPC** | HTTP/2 | High performance, strict contracts | Request-Response | Very Fast |

**1. TCP Transport (Default):**

```typescript
// microservice/main.ts
import { NestFactory } from '@nestjs/core';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.TCP,  // ✅ Default transport
      options: {
        host: 'localhost',
        port: 3001,
        retryAttempts: 5,
        retryDelay: 3000,
      },
    },
  );
  
  await app.listen();
}
bootstrap();
```

```typescript
// client/app.module.ts
import { ClientsModule, Transport } from '@nestjs/microservices';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'MATH_SERVICE',
        transport: Transport.TCP,
        options: {
          host: 'localhost',
          port: 3001,
        },
      },
    ]),
  ],
})
export class AppModule {}
```

**Use TCP when:**
- Simple point-to-point communication
- Low latency required
- No message broker overhead needed
- Internal services on same network

**2. Redis Transport:**

```bash
npm install redis
```

```typescript
// microservice/main.ts
import { Transport } from '@nestjs/microservices';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.REDIS,  // ✅ Redis Pub/Sub
      options: {
        host: 'localhost',
        port: 6379,
        password: 'redis-password',  // Optional
        db: 0,  // Redis database index
        retryAttempts: 5,
        retryDelay: 3000,
      },
    },
  );
  
  await app.listen();
}
bootstrap();
```

```typescript
// client/app.module.ts
ClientsModule.register([
  {
    name: 'NOTIFICATIONS_SERVICE',
    transport: Transport.REDIS,
    options: {
      host: 'localhost',
      port: 6379,
    },
  },
])
```

**Use Redis when:**
- Need pub/sub messaging
- Already using Redis for caching
- Multiple subscribers per message
- Simple event broadcasting

**3. MQTT Transport:**

```bash
npm install mqtt
```

```typescript
// microservice/main.ts
import { Transport } from '@nestjs/microservices';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.MQTT,  // ✅ MQTT for IoT
      options: {
        url: 'mqtt://localhost:1883',
        username: 'mqtt-user',  // Optional
        password: 'mqtt-password',  // Optional
        
        // MQTT-specific options
        clientId: 'nestjs-mqtt-client',
        clean: true,
        reconnectPeriod: 5000,
        connectTimeout: 30000,
        
        // Topics
        subscribeOptions: {
          qos: 1,  // Quality of Service (0, 1, 2)
        },
      },
    },
  );
  
  await app.listen();
}
bootstrap();
```

**Use MQTT when:**
- IoT applications (sensors, devices)
- Low bandwidth environments
- Need QoS (Quality of Service)
- Publish-subscribe pattern
- Mobile apps

**4. NATS Transport:**

```bash
npm install nats
```

```typescript
// microservice/main.ts
import { Transport } from '@nestjs/microservices';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.NATS,  // ✅ NATS (fast, cloud-native)
      options: {
        servers: ['nats://localhost:4222'],  // NATS servers
        
        // Authentication
        user: 'nats-user',
        pass: 'nats-password',
        
        // Connection options
        name: 'nestjs-nats-client',
        maxReconnectAttempts: -1,  // Infinite
        reconnectTimeWait: 2000,
        timeout: 10000,
        
        // Queue groups (load balancing)
        queue: 'nestjs-queue',
      },
    },
  );
  
  await app.listen();
}
bootstrap();
```

**Use NATS when:**
- Cloud-native applications
- Need very high performance
- Microservices on Kubernetes
- Load balancing (queue groups)
- Lightweight messaging

**5. RabbitMQ (AMQP) Transport:**

```bash
npm install amqplib amqp-connection-manager
```

```typescript
// microservice/main.ts
import { Transport } from '@nestjs/microservices';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.RMQ,  // ✅ RabbitMQ
      options: {
        urls: ['amqp://localhost:5672'],
        
        // Queue configuration
        queue: 'orders_queue',
        queueOptions: {
          durable: true,  // Survive broker restart
          arguments: {
            'x-message-ttl': 60000,  // Message TTL (ms)
          },
        },
        
        // Prefetch count (how many messages to process at once)
        prefetchCount: 1,
        
        // Exchange and routing
        noAck: false,  // Manual acknowledgment
        consumerTag: 'nestjs-consumer',
      },
    },
  );
  
  await app.listen();
}
bootstrap();
```

```typescript
// client/app.module.ts
ClientsModule.register([
  {
    name: 'ORDERS_SERVICE',
    transport: Transport.RMQ,
    options: {
      urls: ['amqp://localhost:5672'],
      queue: 'orders_queue',
      queueOptions: {
        durable: true,
      },
    },
  },
])
```

**Use RabbitMQ when:**
- Enterprise messaging requirements
- Need guaranteed delivery
- Message routing/filtering
- Work queues with load balancing
- Dead letter queues

**6. Kafka Transport:**

```bash
npm install kafkajs
```

```typescript
// microservice/main.ts
import { Transport } from '@nestjs/microservices';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.KAFKA,  // ✅ Apache Kafka
      options: {
        client: {
          clientId: 'orders-service',
          brokers: ['localhost:9092'],  // Kafka brokers
          
          // Authentication (SASL)
          sasl: {
            mechanism: 'plain',
            username: 'kafka-user',
            password: 'kafka-password',
          },
          
          // SSL
          ssl: true,
        },
        
        consumer: {
          groupId: 'orders-consumer-group',  // Consumer group
          
          // Retry configuration
          retry: {
            initialRetryTime: 100,
            retries: 8,
          },
          
          // Session timeout
          sessionTimeout: 30000,
          heartbeatInterval: 3000,
        },
        
        // Consumer run config
        run: {
          autoCommit: true,
          autoCommitInterval: 5000,
          autoCommitThreshold: 100,
        },
      },
    },
  );
  
  await app.listen();
}
bootstrap();
```

```typescript
// Handle Kafka messages
@Controller()
export class OrdersController {
  @MessagePattern('order-created')  // Topic name
  async handleOrderCreated(data: any) {
    console.log('Received order:', data);
    // Process order
  }
  
  @EventPattern('payment-processed')  // Topic name
  async handlePaymentProcessed(data: any) {
    console.log('Payment processed:', data);
    // Update order status
  }
}
```

**Use Kafka when:**
- Event streaming (millions of events)
- Event sourcing
- Log aggregation
- Real-time analytics
- High throughput required
- Need event replay

**7. gRPC Transport:**

```bash
npm install @grpc/grpc-js @grpc/proto-loader
```

```protobuf
// hero.proto
syntax = "proto3";

package hero;

service HeroService {
  rpc FindOne (HeroById) returns (Hero) {}
  rpc FindMany (Empty) returns (HeroList) {}
}

message HeroById {
  int32 id = 1;
}

message Hero {
  int32 id = 1;
  string name = 2;
}

message HeroList {
  repeated Hero heroes = 1;
}

message Empty {}
```

```typescript
// microservice/main.ts
import { Transport } from '@nestjs/microservices';
import { join } from 'path';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.GRPC,  // ✅ gRPC
      options: {
        package: 'hero',  // Package name from .proto
        protoPath: join(__dirname, 'hero.proto'),  // Path to .proto file
        
        // Server URL
        url: 'localhost:5000',
        
        // Loader options
        loader: {
          keepCase: true,
          longs: String,
          enums: String,
          defaults: true,
          oneofs: true,
        },
        
        // gRPC credentials
        credentials: grpc.ServerCredentials.createInsecure(),
      },
    },
  );
  
  await app.listen();
}
bootstrap();
```

```typescript
// hero.controller.ts
import { Controller } from '@nestjs/common';
import { GrpcMethod } from '@nestjs/microservices';

@Controller()
export class HeroController {
  @GrpcMethod('HeroService', 'FindOne')
  findOne(data: { id: number }) {
    return {
      id: data.id,
      name: 'Hero #' + data.id,
    };
  }
  
  @GrpcMethod('HeroService', 'FindMany')
  findMany() {
    return {
      heroes: [
        { id: 1, name: 'Hero 1' },
        { id: 2, name: 'Hero 2' },
      ],
    };
  }
}
```

**Use gRPC when:**
- High performance required
- Type safety (Protocol Buffers)
- Streaming (bidirectional)
- Polyglot (multiple languages)
- Internal microservices

**Choosing the Right Transport:**

```typescript
// Scenario 1: Simple internal services
Transport.TCP  // ✅ Simple, fast, no dependencies

// Scenario 2: Already using Redis
Transport.REDIS  // ✅ Reuse existing infrastructure

// Scenario 3: IoT sensors
Transport.MQTT  // ✅ Lightweight, QoS support

// Scenario 4: Cloud-native on Kubernetes
Transport.NATS  // ✅ Fast, lightweight, resilient

// Scenario 5: Enterprise with complex routing
Transport.RMQ  // ✅ RabbitMQ - reliable, feature-rich

// Scenario 6: Event streaming, analytics
Transport.KAFKA  // ✅ High throughput, event replay

// Scenario 7: High performance, type safety
Transport.GRPC  // ✅ Fast, strongly typed
```

**Interview Tip**: NestJS supports **7 transports**: **TCP** (default, simple), **Redis** (pub/sub, caching), **MQTT** (IoT, lightweight), **NATS** (cloud-native, fast), **RabbitMQ** (enterprise, reliable), **Kafka** (event streaming, high throughput), **gRPC** (high performance, type safety). Choose based on: **TCP** for simple services, **Redis** if already using it, **RabbitMQ** for enterprise, **Kafka** for events, **gRPC** for performance. Configure in **createMicroservice()** with **transport** and **options**. Each transport has specific use cases and trade-offs.

</details>

### 4. What is `@nestjs/microservices` package?

<details>
<summary>Answer</summary>

**@nestjs/microservices** is a NestJS official package that provides microservice architecture support with multiple transport layers (TCP, Redis, RabbitMQ, Kafka, gRPC, MQTT, NATS). It enables building distributed systems with request-response and event-based communication patterns.

**Installation:**

```bash
# Install the microservices package
npm install @nestjs/microservices

# Transport-specific dependencies (install as needed)
npm install redis  # For Redis transport
npm install amqplib amqp-connection-manager  # For RabbitMQ
npm install kafkajs  # For Kafka
npm install mqtt  # For MQTT
npm install nats  # For NATS
npm install @grpc/grpc-js @grpc/proto-loader  # For gRPC
```

**Core Features:**

```
1. Transport Layers ✅
   - TCP, Redis, RabbitMQ, Kafka, gRPC, MQTT, NATS
   - Pluggable transport system
   
2. Communication Patterns ✅
   - Request-Response (@MessagePattern)
   - Event-Based (@EventPattern)
   
3. Client-Server Model ✅
   - ClientProxy for sending messages
   - Controllers with decorators for receiving
   
4. Hybrid Applications ✅
   - Combine HTTP and microservice in one app
   - Multiple transports simultaneously
```

**Key Exports from @nestjs/microservices:**

```typescript
import {
  // Factory methods
  NestFactory,  // createMicroservice()
  
  // Decorators
  MessagePattern,  // Request-response pattern
  EventPattern,    // Event-based pattern
  Payload,         // Extract message payload
  Ctx,             // Access message context
  
  // Client
  ClientProxy,           // Base client interface
  ClientProxyFactory,    // Create clients dynamically
  Client,                // Decorator to inject client
  ClientsModule,         // Module for registering clients
  
  // Transport enum
  Transport,  // TCP, REDIS, RMQ, KAFKA, GRPC, MQTT, NATS
  
  // Options interfaces
  MicroserviceOptions,
  TcpOptions,
  RedisOptions,
  RmqOptions,
  KafkaOptions,
  GrpcOptions,
  MqttOptions,
  NatsOptions,
  
  // Exceptions
  RpcException,  // Throw errors in microservices
  
  // Context
  RmqContext,
  KafkaContext,
  NatsContext,
  MqttContext,
  TcpContext,
  RedisContext,
} from '@nestjs/microservices';
```

**Basic Microservice Setup:**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';
import { AppModule } from './app.module';

async function bootstrap() {
  // Create microservice application
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.TCP,  // Choose transport
      options: {
        host: 'localhost',
        port: 3001,
      },
    },
  );

  await app.listen();
  console.log('Microservice is listening on port 3001');
}
bootstrap();
```

**Message Patterns (Server Side):**

```typescript
// math.controller.ts
import { Controller } from '@nestjs/common';
import { MessagePattern, EventPattern, Payload, Ctx } from '@nestjs/microservices';

@Controller()
export class MathController {
  // Request-response pattern (waits for response)
  @MessagePattern('add')  // Pattern name
  add(@Payload() data: { a: number; b: number }) {
    console.log('Received add request:', data);
    return {
      result: data.a + data.b,
    };
  }

  // Event pattern (fire and forget, no response)
  @EventPattern('user-created')
  handleUserCreated(@Payload() data: { userId: string; email: string }) {
    console.log('User created event:', data);
    // Process event (send email, update cache, etc.)
    // No return value expected
  }

  // Access context for metadata
  @MessagePattern('multiply')
  multiply(
    @Payload() data: { a: number; b: number },
    @Ctx() context: any,
  ) {
    console.log('Context:', context);
    return data.a * data.b;
  }
}
```

**Client Setup (Consumer):**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';
import { AppController } from './app.controller';

@Module({
  imports: [
    // Register microservice clients
    ClientsModule.register([
      {
        name: 'MATH_SERVICE',  // Injection token
        transport: Transport.TCP,
        options: {
          host: 'localhost',
          port: 3001,
        },
      },
    ]),
  ],
  controllers: [AppController],
})
export class AppModule {}
```

```typescript
// app.controller.ts
import { Controller, Get, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { firstValueFrom } from 'rxjs';

@Controller()
export class AppController {
  constructor(
    @Inject('MATH_SERVICE') private mathClient: ClientProxy,
  ) {}

  @Get('add')
  async add() {
    // Send message to microservice
    const response = await firstValueFrom(
      this.mathClient.send('add', { a: 5, b: 3 }),
    );
    
    return response;  // { result: 8 }
  }

  @Get('notify')
  async notify() {
    // Emit event (no response)
    this.mathClient.emit('user-created', {
      userId: '123',
      email: 'user@example.com',
    });
    
    return { message: 'Event emitted' };
  }
}
```

**Multiple Transports:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';

@Module({
  imports: [
    ClientsModule.register([
      // TCP service
      {
        name: 'AUTH_SERVICE',
        transport: Transport.TCP,
        options: { host: 'localhost', port: 3001 },
      },
      
      // Redis service
      {
        name: 'CACHE_SERVICE',
        transport: Transport.REDIS,
        options: { host: 'localhost', port: 6379 },
      },
      
      // RabbitMQ service
      {
        name: 'ORDERS_SERVICE',
        transport: Transport.RMQ,
        options: {
          urls: ['amqp://localhost:5672'],
          queue: 'orders_queue',
        },
      },
      
      // Kafka service
      {
        name: 'EVENTS_SERVICE',
        transport: Transport.KAFKA,
        options: {
          client: {
            brokers: ['localhost:9092'],
          },
          consumer: {
            groupId: 'my-consumer',
          },
        },
      },
    ]),
  ],
})
export class AppModule {}
```

**Using Multiple Clients:**

```typescript
// order.service.ts
import { Injectable, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { firstValueFrom } from 'rxjs';

@Injectable()
export class OrderService {
  constructor(
    @Inject('AUTH_SERVICE') private authClient: ClientProxy,
    @Inject('CACHE_SERVICE') private cacheClient: ClientProxy,
    @Inject('ORDERS_SERVICE') private ordersClient: ClientProxy,
  ) {}

  async createOrder(userId: string, items: any[]) {
    // 1. Validate user via TCP
    const user = await firstValueFrom(
      this.authClient.send('validate-user', { userId }),
    );

    // 2. Check cache via Redis
    const cached = await firstValueFrom(
      this.cacheClient.send('get-user-cart', { userId }),
    );

    // 3. Create order via RabbitMQ
    const order = await firstValueFrom(
      this.ordersClient.send('create-order', {
        userId,
        items: items || cached,
      }),
    );

    return order;
  }
}
```

**Dynamic Client Creation:**

```typescript
// dynamic-client.service.ts
import { Injectable, OnModuleInit, OnModuleDestroy } from '@nestjs/common';
import { ClientProxyFactory, Transport } from '@nestjs/microservices';
import { firstValueFrom } from 'rxjs';

@Injectable()
export class DynamicClientService implements OnModuleInit, OnModuleDestroy {
  private client: ClientProxy;

  onModuleInit() {
    // Create client dynamically
    this.client = ClientProxyFactory.create({
      transport: Transport.TCP,
      options: {
        host: 'localhost',
        port: 3001,
      },
    });

    // Connect to microservice
    this.client.connect();
  }

  async sendMessage(pattern: string, data: any) {
    const response = await firstValueFrom(
      this.client.send(pattern, data),
    );
    return response;
  }

  onModuleDestroy() {
    // Close connection
    this.client.close();
  }
}
```

**Error Handling:**

```typescript
// math.controller.ts
import { Controller } from '@nestjs/common';
import { MessagePattern, RpcException } from '@nestjs/microservices';

@Controller()
export class MathController {
  @MessagePattern('divide')
  divide(data: { a: number; b: number }) {
    if (data.b === 0) {
      // Throw RPC exception
      throw new RpcException('Division by zero');
    }
    
    return {
      result: data.a / data.b,
    };
  }
}
```

```typescript
// app.controller.ts (client)
import { Controller, Get, BadRequestException } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { catchError } from 'rxjs/operators';
import { throwError } from 'rxjs';

@Controller()
export class AppController {
  constructor(
    @Inject('MATH_SERVICE') private mathClient: ClientProxy,
  ) {}

  @Get('divide')
  async divide() {
    try {
      const response = await firstValueFrom(
        this.mathClient.send('divide', { a: 10, b: 0 }).pipe(
          catchError(error => {
            console.error('Microservice error:', error);
            return throwError(() => new BadRequestException(error.message));
          }),
        ),
      );
      
      return response;
    } catch (error) {
      throw new BadRequestException(error.message);
    }
  }
}
```

**Package Structure:**

```
@nestjs/microservices/
├── transporters/
│   ├── tcp/
│   ├── redis/
│   ├── mqtt/
│   ├── nats/
│   ├── rmq/
│   ├── kafka/
│   └── grpc/
├── client/
│   ├── client-proxy.ts
│   ├── client-proxy-factory.ts
│   └── clients-module.ts
├── decorators/
│   ├── message-pattern.decorator.ts
│   ├── event-pattern.decorator.ts
│   ├── payload.decorator.ts
│   └── ctx.decorator.ts
├── exceptions/
│   └── rpc-exception.ts
└── interfaces/
    └── (transport options)
```

**Key Concepts:**

```typescript
// 1. Transport Layer
// Defines HOW services communicate
Transport.TCP      // Direct TCP connection
Transport.REDIS    // Redis pub/sub
Transport.RMQ      // RabbitMQ AMQP
Transport.KAFKA    // Apache Kafka
Transport.GRPC     // gRPC (HTTP/2)
Transport.MQTT     // MQTT protocol
Transport.NATS     // NATS messaging

// 2. Message Patterns
// Defines WHAT messages to handle
@MessagePattern('user.create')  // Request-response
@EventPattern('user.created')   // Event-based

// 3. Client Proxy
// Sends messages to microservices
client.send()   // Request-response (returns Observable)
client.emit()   // Event-based (no response)

// 4. Decorators
@Payload()  // Extract message data
@Ctx()      // Access message context (headers, metadata)
```

**Best Practices:**

```typescript
// ✅ GOOD: Use ClientsModule for static configuration
ClientsModule.register([...])

// ✅ GOOD: Use ClientProxyFactory for dynamic clients
ClientProxyFactory.create({ transport, options })

// ✅ GOOD: Always handle errors
try {
  const result = await firstValueFrom(client.send(...));
} catch (error) {
  // Handle error
}

// ✅ GOOD: Use RpcException for microservice errors
throw new RpcException('Error message');

// ✅ GOOD: Close clients on module destroy
this.client.close();

// ❌ BAD: Not handling connection errors
// Can cause hanging requests

// ❌ BAD: Not converting Observables to Promises
// client.send() returns Observable, use firstValueFrom()

// ❌ BAD: Using same pattern name in multiple controllers
// Can cause conflicts
```

**Interview Tip**: **@nestjs/microservices** is NestJS's **official package** for building microservices. Provides: **7 transport layers** (TCP, Redis, RabbitMQ, Kafka, gRPC, MQTT, NATS), **communication patterns** (@MessagePattern for request-response, @EventPattern for events), **ClientProxy** for sending messages, **ClientsModule** for registering clients. Create microservice with **NestFactory.createMicroservice()**. Use **client.send()** for responses, **client.emit()** for events. Handle errors with **RpcException**. Supports **hybrid apps** (HTTP + microservice).

</details>

## Creating Microservices

### 5. How do you create a microservice application?

<details>
<summary>Answer</summary>

**Create a microservice** using **NestFactory.createMicroservice()** instead of **NestFactory.create()**. Configure the transport layer and options, then start listening for messages.

**Basic Microservice Creation:**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';
import { AppModule } from './app.module';

async function bootstrap() {
  // ✅ Create microservice (NOT regular HTTP app)
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,  // Your application module
    {
      transport: Transport.TCP,  // Transport layer
      options: {
        host: 'localhost',  // Host to bind
        port: 3001,         // Port to listen on
      },
    },
  );

  // Start listening for messages
  await app.listen();
  
  console.log('Microservice is listening on TCP port 3001');
}
bootstrap();
```

**Complete Project Structure:**

```
microservice/
├── src/
│   ├── main.ts                 # Microservice entry point
│   ├── app.module.ts           # Root module
│   ├── app.controller.ts       # Message handlers
│   └── app.service.ts          # Business logic
├── package.json
├── tsconfig.json
└── nest-cli.json
```

**Step-by-Step Setup:**

```bash
# 1. Create new NestJS project
nest new microservice-app
cd microservice-app

# 2. Install microservices package
npm install @nestjs/microservices

# 3. (Optional) Install transport-specific dependencies
npm install redis          # For Redis
npm install amqplib        # For RabbitMQ
npm install kafkajs        # For Kafka
npm install @grpc/grpc-js  # For gRPC
```

**App Module Setup:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  imports: [],  // Import other modules
  controllers: [AppController],  // Controllers with @MessagePattern
  providers: [AppService],       // Services with business logic
})
export class AppModule {}
```

**Controller with Message Patterns:**

```typescript
// app.controller.ts
import { Controller } from '@nestjs/common';
import { MessagePattern, EventPattern, Payload } from '@nestjs/microservices';
import { AppService } from './app.service';

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  // Handle 'get-hello' pattern (request-response)
  @MessagePattern('get-hello')
  getHello(@Payload() data: any) {
    return this.appService.getHello();
  }

  // Handle 'add-numbers' pattern
  @MessagePattern('add-numbers')
  addNumbers(@Payload() data: { a: number; b: number }) {
    return {
      result: this.appService.add(data.a, data.b),
    };
  }

  // Handle 'user-created' event (fire and forget)
  @EventPattern('user-created')
  handleUserCreated(@Payload() data: { userId: string; email: string }) {
    console.log('User created:', data);
    this.appService.processNewUser(data);
    // No return value needed for events
  }
}
```

**Service Implementation:**

```typescript
// app.service.ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class AppService {
  getHello(): string {
    return 'Hello from Microservice!';
  }

  add(a: number, b: number): number {
    return a + b;
  }

  processNewUser(data: { userId: string; email: string }): void {
    // Process new user (send email, update cache, etc.)
    console.log(`Processing new user: ${data.email}`);
  }
}
```

**Different Transport Examples:**

```typescript
// 1. TCP Transport (Default)
async function bootstrapTCP() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.TCP,
      options: {
        host: '0.0.0.0',  // Bind to all interfaces
        port: 3001,
        retryAttempts: 5,
        retryDelay: 3000,
      },
    },
  );
  
  await app.listen();
  console.log('TCP Microservice on port 3001');
}

// 2. Redis Transport
async function bootstrapRedis() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.REDIS,
      options: {
        host: 'localhost',
        port: 6379,
        password: 'redis-password',  // Optional
        db: 0,
      },
    },
  );
  
  await app.listen();
  console.log('Redis Microservice on port 6379');
}

// 3. RabbitMQ Transport
async function bootstrapRabbitMQ() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.RMQ,
      options: {
        urls: ['amqp://localhost:5672'],
        queue: 'my_queue',
        queueOptions: {
          durable: true,
        },
      },
    },
  );
  
  await app.listen();
  console.log('RabbitMQ Microservice listening on queue: my_queue');
}

// 4. Kafka Transport
async function bootstrapKafka() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.KAFKA,
      options: {
        client: {
          clientId: 'my-app',
          brokers: ['localhost:9092'],
        },
        consumer: {
          groupId: 'my-consumer',
        },
      },
    },
  );
  
  await app.listen();
  console.log('Kafka Microservice listening');
}

// 5. gRPC Transport
import { join } from 'path';

async function bootstrapGRPC() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.GRPC,
      options: {
        package: 'hero',
        protoPath: join(__dirname, './hero.proto'),
        url: 'localhost:5000',
      },
    },
  );
  
  await app.listen();
  console.log('gRPC Microservice on port 5000');
}
```

**Environment-Based Configuration:**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';
import { ConfigService } from '@nestjs/config';
import { AppModule } from './app.module';

async function bootstrap() {
  // Create temporary app to access ConfigService
  const tempApp = await NestFactory.create(AppModule);
  const configService = tempApp.get(ConfigService);
  
  // Get configuration from environment
  const transport = configService.get('TRANSPORT', 'TCP');
  const host = configService.get('MICROSERVICE_HOST', 'localhost');
  const port = configService.get('MICROSERVICE_PORT', 3001);
  
  await tempApp.close();

  // Create microservice with environment config
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport[transport],
      options: { host, port },
    },
  );

  await app.listen();
  console.log(`Microservice listening on ${transport} ${host}:${port}`);
}
bootstrap();
```

```bash
# .env
TRANSPORT=TCP
MICROSERVICE_HOST=localhost
MICROSERVICE_PORT=3001
```

**Microservice with Database:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ConfigModule } from '@nestjs/config';
import { UsersModule } from './users/users.module';

@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),
    
    // Database connection
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'postgres',
      password: 'password',
      database: 'microservice_db',
      entities: [__dirname + '/**/*.entity{.ts,.js}'],
      synchronize: true,
    }),
    
    UsersModule,
  ],
})
export class AppModule {}
```

```typescript
// users/users.controller.ts
import { Controller } from '@nestjs/common';
import { MessagePattern, Payload } from '@nestjs/microservices';
import { UsersService } from './users.service';

@Controller()
export class UsersController {
  constructor(private usersService: UsersService) {}

  @MessagePattern('create-user')
  async createUser(@Payload() data: { email: string; name: string }) {
    const user = await this.usersService.create(data);
    return user;
  }

  @MessagePattern('get-user')
  async getUser(@Payload() data: { id: string }) {
    const user = await this.usersService.findOne(data.id);
    return user;
  }
}
```

**Docker Setup:**

```dockerfile
# Dockerfile
FROM node:18-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./
RUN npm ci

# Copy source
COPY . .

# Build
RUN npm run build

# Expose port (for TCP transport)
EXPOSE 3001

# Start microservice
CMD ["node", "dist/main"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  microservice:
    build: .
    ports:
      - "3001:3001"
    environment:
      - TRANSPORT=TCP
      - MICROSERVICE_HOST=0.0.0.0
      - MICROSERVICE_PORT=3001
      - DATABASE_HOST=postgres
    depends_on:
      - postgres
  
  postgres:
    image: postgres:15
    environment:
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=microservice_db
    ports:
      - "5432:5432"
```

**Running the Microservice:**

```bash
# Development
npm run start:dev

# Production build
npm run build
npm run start:prod

# With Docker
docker-compose up
```

**Testing the Microservice:**

```typescript
// Create a test client
import { ClientProxyFactory, Transport } from '@nestjs/microservices';
import { firstValueFrom } from 'rxjs';

async function testMicroservice() {
  // Create client
  const client = ClientProxyFactory.create({
    transport: Transport.TCP,
    options: { host: 'localhost', port: 3001 },
  });

  // Connect
  await client.connect();

  // Send message
  const response = await firstValueFrom(
    client.send('add-numbers', { a: 5, b: 3 }),
  );
  
  console.log('Response:', response);  // { result: 8 }

  // Emit event
  client.emit('user-created', { userId: '123', email: 'user@example.com' });

  // Close connection
  await client.close();
}

testMicroservice();
```

**Best Practices:**

```typescript
// ✅ GOOD: Use environment variables
const port = process.env.MICROSERVICE_PORT || 3001;

// ✅ GOOD: Handle graceful shutdown
process.on('SIGTERM', async () => {
  await app.close();
  process.exit(0);
});

// ✅ GOOD: Add logging
await app.listen();
console.log('Microservice started on port', port);

// ✅ GOOD: Use proper error handling
@MessagePattern('risky-operation')
async riskyOperation() {
  try {
    return await this.service.doSomething();
  } catch (error) {
    throw new RpcException(error.message);
  }
}

// ❌ BAD: Hardcoding configuration
const app = await NestFactory.createMicroservice(AppModule, {
  options: { host: '192.168.1.100', port: 3001 },  // Hardcoded!
});

// ❌ BAD: Not handling errors
@MessagePattern('dangerous')
async dangerous() {
  return await this.service.dangerous();  // Can throw!
}

// ❌ BAD: Mixing HTTP and microservice in main.ts
// Use hybrid application instead
```

**Interview Tip**: Create microservice with **NestFactory.createMicroservice()** instead of create(). Specify **transport** (TCP, Redis, RabbitMQ, Kafka, gRPC) and **options** (host, port). Use **@MessagePattern()** for request-response, **@EventPattern()** for events. Start with **await app.listen()**. Different from HTTP app: **no routes**, uses **message patterns**, **network-based communication**. Can run **multiple microservices** on different ports. Use **Docker** for deployment, **ConfigService** for environment-based config. Test with **ClientProxyFactory.create()**.

</details>

### 6. What is `createMicroservice()` method?

<details>
<summary>Answer</summary>

**`createMicroservice()`** is a **NestFactory method** that creates a microservice application instead of a traditional HTTP server. It returns a microservice instance that listens for messages on a specified transport layer.

**Method Signature:**

```typescript
static async createMicroservice<T extends object>(
  module: any,                           // Application module
  options?: MicroserviceOptions,         // Transport configuration
): Promise<INestMicroservice>
```

**Basic Usage:**

```typescript
import { NestFactory } from '@nestjs/core';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';
import { AppModule } from './app.module';

async function bootstrap() {
  // ✅ Create microservice (not HTTP app)
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,  // Module to bootstrap
    {
      // Transport configuration
      transport: Transport.TCP,
      options: {
        host: 'localhost',
        port: 3001,
      },
    },
  );

  // Start listening for messages
  await app.listen();
  
  console.log('Microservice is listening');
}
bootstrap();
```

**Comparison: create() vs createMicroservice():**

```typescript
// HTTP Application (REST API)
import { NestFactory } from '@nestjs/core';

async function bootstrap() {
  // ✅ Creates HTTP server
  const app = await NestFactory.create(AppModule);
  
  // Listen on HTTP port
  await app.listen(3000);
  
  console.log('HTTP server on http://localhost:3000');
}
```

```typescript
// Microservice Application
import { NestFactory } from '@nestjs/core';
import { Transport } from '@nestjs/microservices';

async function bootstrap() {
  // ✅ Creates microservice
  const app = await NestFactory.createMicroservice(AppModule, {
    transport: Transport.TCP,
    options: { host: 'localhost', port: 3001 },
  });
  
  // Listen for messages
  await app.listen();
  
  console.log('Microservice listening on TCP 3001');
}
```

**Key Differences:**

| Feature | `create()` | `createMicroservice()` |
|---------|------------|------------------------|
| **Type** | HTTP Server | Microservice |
| **Protocol** | HTTP/HTTPS | TCP, Redis, RabbitMQ, Kafka, gRPC, MQTT, NATS |
| **Routing** | @Get(), @Post(), etc. | @MessagePattern(), @EventPattern() |
| **Response** | HTTP response | Message response |
| **Port** | HTTP port (3000) | Transport-specific port |
| **Use Case** | REST API, Web app | Distributed services |

**Return Type:**

```typescript
interface INestMicroservice {
  // Start listening for messages
  listen(): Promise<any>;
  
  // Close microservice
  close(): Promise<void>;
  
  // Get microservice options
  getOptions(): any;
  
  // Initialize microservice
  init(): Promise<this>;
  
  // Use global filters, pipes, interceptors
  useGlobalFilters(...filters: ExceptionFilter[]): this;
  useGlobalPipes(...pipes: PipeTransform[]): this;
  useGlobalInterceptors(...interceptors: NestInterceptor[]): this;
  useGlobalGuards(...guards: CanActivate[]): this;
}
```

**All Transport Configurations:**

```typescript
// 1. TCP Transport
const tcpApp = await NestFactory.createMicroservice<MicroserviceOptions>(
  AppModule,
  {
    transport: Transport.TCP,
    options: {
      host: 'localhost',
      port: 3001,
      retryAttempts: 5,
      retryDelay: 3000,
    },
  },
);

// 2. Redis Transport
const redisApp = await NestFactory.createMicroservice<MicroserviceOptions>(
  AppModule,
  {
    transport: Transport.REDIS,
    options: {
      host: 'localhost',
      port: 6379,
      password: 'redis-password',
      db: 0,
      retryAttempts: 5,
    },
  },
);

// 3. MQTT Transport
const mqttApp = await NestFactory.createMicroservice<MicroserviceOptions>(
  AppModule,
  {
    transport: Transport.MQTT,
    options: {
      url: 'mqtt://localhost:1883',
      clientId: 'nestjs-mqtt',
      username: 'mqtt-user',
      password: 'mqtt-pass',
    },
  },
);

// 4. NATS Transport
const natsApp = await NestFactory.createMicroservice<MicroserviceOptions>(
  AppModule,
  {
    transport: Transport.NATS,
    options: {
      servers: ['nats://localhost:4222'],
      queue: 'nestjs-queue',
    },
  },
);

// 5. RabbitMQ Transport
const rmqApp = await NestFactory.createMicroservice<MicroserviceOptions>(
  AppModule,
  {
    transport: Transport.RMQ,
    options: {
      urls: ['amqp://localhost:5672'],
      queue: 'my_queue',
      queueOptions: {
        durable: true,
      },
    },
  },
);

// 6. Kafka Transport
const kafkaApp = await NestFactory.createMicroservice<MicroserviceOptions>(
  AppModule,
  {
    transport: Transport.KAFKA,
    options: {
      client: {
        clientId: 'my-app',
        brokers: ['localhost:9092'],
      },
      consumer: {
        groupId: 'my-consumer',
      },
    },
  },
);

// 7. gRPC Transport
const grpcApp = await NestFactory.createMicroservice<MicroserviceOptions>(
  AppModule,
  {
    transport: Transport.GRPC,
    options: {
      package: 'hero',
      protoPath: join(__dirname, './hero.proto'),
      url: 'localhost:5000',
    },
  },
);
```

**Using Global Middleware:**

```typescript
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { RpcExceptionFilter } from './filters/rpc-exception.filter';
import { LoggingInterceptor } from './interceptors/logging.interceptor';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.TCP,
      options: { host: 'localhost', port: 3001 },
    },
  );

  // ✅ Use global pipes
  app.useGlobalPipes(
    new ValidationPipe({
      transform: true,
      whitelist: true,
    }),
  );

  // ✅ Use global filters
  app.useGlobalFilters(new RpcExceptionFilter());

  // ✅ Use global interceptors
  app.useGlobalInterceptors(new LoggingInterceptor());

  await app.listen();
}
bootstrap();
```

**Multiple Microservices in One Application:**

```typescript
// Method 1: Separate processes (recommended)
// auth-service/main.ts
async function bootstrapAuthService() {
  const app = await NestFactory.createMicroservice(AuthModule, {
    transport: Transport.TCP,
    options: { port: 3001 },
  });
  await app.listen();
}

// users-service/main.ts
async function bootstrapUsersService() {
  const app = await NestFactory.createMicroservice(UsersModule, {
    transport: Transport.TCP,
    options: { port: 3002 },
  });
  await app.listen();
}

// Method 2: Same process (not recommended)
async function bootstrapMultiple() {
  // Auth microservice
  const authApp = await NestFactory.createMicroservice(AuthModule, {
    transport: Transport.TCP,
    options: { port: 3001 },
  });
  
  // Users microservice
  const usersApp = await NestFactory.createMicroservice(UsersModule, {
    transport: Transport.TCP,
    options: { port: 3002 },
  });
  
  // Start both
  await authApp.listen();
  await usersApp.listen();
}
```

**Graceful Shutdown:**

```typescript
import { NestFactory } from '@nestjs/core';
import { Transport } from '@nestjs/microservices';

async function bootstrap() {
  const app = await NestFactory.createMicroservice(AppModule, {
    transport: Transport.TCP,
    options: { host: 'localhost', port: 3001 },
  });

  // ✅ Handle shutdown signals
  process.on('SIGTERM', async () => {
    console.log('SIGTERM received, shutting down gracefully...');
    await app.close();
    process.exit(0);
  });

  process.on('SIGINT', async () => {
    console.log('SIGINT received, shutting down gracefully...');
    await app.close();
    process.exit(0);
  });

  await app.listen();
  console.log('Microservice is listening');
}
bootstrap();
```

**Lifecycle Hooks:**

```typescript
import { NestFactory } from '@nestjs/core';
import { Transport } from '@nestjs/microservices';

async function bootstrap() {
  const app = await NestFactory.createMicroservice(AppModule, {
    transport: Transport.TCP,
    options: { host: 'localhost', port: 3001 },
  });

  // ✅ Enable shutdown hooks
  app.enableShutdownHooks();

  await app.listen();
}
bootstrap();
```

**With Configuration Service:**

```typescript
import { NestFactory } from '@nestjs/core';
import { ConfigService } from '@nestjs/config';
import { Transport } from '@nestjs/microservices';

async function bootstrap() {
  // Create temporary app to access ConfigService
  const tempApp = await NestFactory.create(AppModule);
  const configService = tempApp.get(ConfigService);
  
  // Get configuration
  const host = configService.get('MICROSERVICE_HOST', 'localhost');
  const port = configService.get('MICROSERVICE_PORT', 3001);
  
  await tempApp.close();

  // Create microservice with config
  const app = await NestFactory.createMicroservice(AppModule, {
    transport: Transport.TCP,
    options: { host, port },
  });

  await app.listen();
  console.log(`Microservice listening on ${host}:${port}`);
}
bootstrap();
```

**Error Handling:**

```typescript
import { NestFactory } from '@nestjs/core';
import { Transport } from '@nestjs/microservices';

async function bootstrap() {
  try {
    const app = await NestFactory.createMicroservice(AppModule, {
      transport: Transport.TCP,
      options: { host: 'localhost', port: 3001 },
    });

    await app.listen();
    console.log('Microservice started successfully');
  } catch (error) {
    console.error('Failed to start microservice:', error);
    process.exit(1);
  }
}
bootstrap();
```

**Testing createMicroservice():**

```typescript
// microservice.spec.ts
import { Test } from '@nestjs/testing';
import { Transport } from '@nestjs/microservices';
import { AppModule } from './app.module';

describe('Microservice', () => {
  it('should create microservice', async () => {
    const module = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    const app = module.createNestMicroservice({
      transport: Transport.TCP,
      options: { host: 'localhost', port: 3001 },
    });

    await app.init();
    
    // Test microservice
    
    await app.close();
  });
});
```

**Best Practices:**

```typescript
// ✅ GOOD: Use TypeScript generic for type safety
const app = await NestFactory.createMicroservice<MicroserviceOptions>(
  AppModule,
  { transport: Transport.TCP, options: { port: 3001 } },
);

// ✅ GOOD: Handle errors
try {
  await app.listen();
} catch (error) {
  console.error('Failed to start:', error);
  process.exit(1);
}

// ✅ GOOD: Use environment variables
const port = parseInt(process.env.PORT) || 3001;

// ✅ GOOD: Enable graceful shutdown
app.enableShutdownHooks();

// ✅ GOOD: Log startup info
await app.listen();
console.log(`Microservice on ${transport} ${host}:${port}`);

// ❌ BAD: Hardcoded configuration
const app = await NestFactory.createMicroservice(AppModule, {
  options: { host: '192.168.1.100', port: 3001 },
});

// ❌ BAD: Not awaiting listen()
app.listen();  // Missing await!

// ❌ BAD: No error handling
await app.listen();  // Can throw!
```

**Interview Tip**: **createMicroservice()** is a **NestFactory method** that creates a **microservice application** instead of HTTP server. Takes **module** and **MicroserviceOptions** (transport, options). Returns **INestMicroservice** with **listen()**, **close()** methods. **Different from create()**: no HTTP routes, uses **@MessagePattern()/@EventPattern()**, listens on **transport-specific port**. Configure with **transport** (TCP, Redis, RabbitMQ, Kafka, gRPC) and **options** (host, port, credentials). Use **useGlobalPipes/Filters/Interceptors** for global middleware. Start with **await app.listen()**. Handle **graceful shutdown** with SIGTERM/SIGINT.

</details>

### 7. How do you configure transport options?

<details>
<summary>Answer</summary>

**Transport options** are configured in the **second parameter** of `createMicroservice()` or in `ClientsModule.register()`. Each transport (TCP, Redis, RabbitMQ, Kafka, gRPC, MQTT, NATS) has specific configuration options.

**Basic Configuration Structure:**

```typescript
const app = await NestFactory.createMicroservice<MicroserviceOptions>(
  AppModule,
  {
    transport: Transport.TCP,  // Transport type
    options: {                  // Transport-specific options
      host: 'localhost',
      port: 3001,
      // ... other options
    },
  },
);
```

**1. TCP Transport Options:**

```typescript
import { NestFactory } from '@nestjs/core';
import { Transport, TcpOptions } from '@nestjs/microservices';

interface TcpOptions {
  host?: string;           // Host to bind (default: 'localhost')
  port?: number;           // Port to listen (default: 3000)
  retryAttempts?: number;  // Connection retry attempts (default: 0)
  retryDelay?: number;     // Delay between retries in ms (default: 0)
}

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.TCP,
      options: {
        host: '0.0.0.0',      // Bind to all interfaces
        port: 3001,            // Custom port
        retryAttempts: 5,      // Retry 5 times on connection failure
        retryDelay: 3000,      // Wait 3 seconds between retries
      },
    },
  );
  
  await app.listen();
}
```

**2. Redis Transport Options:**

```typescript
import { Transport, RedisOptions } from '@nestjs/microservices';

interface RedisOptions {
  host?: string;           // Redis host (default: 'localhost')
  port?: number;           // Redis port (default: 6379)
  password?: string;       // Redis password
  db?: number;             // Redis database (default: 0)
  retryAttempts?: number;  // Retry attempts
  retryDelay?: number;     // Retry delay
  retryStrategy?: (times: number) => number;  // Custom retry strategy
}

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.REDIS,
      options: {
        host: 'localhost',
        port: 6379,
        password: 'my-redis-password',  // Optional
        db: 0,                           // Database index
        retryAttempts: 10,
        retryDelay: 3000,
        
        // Custom retry strategy
        retryStrategy: (times: number) => {
          // Exponential backoff: 1s, 2s, 4s, 8s...
          return Math.min(times * 1000, 3000);
        },
      },
    },
  );
  
  await app.listen();
}
```

**3. RabbitMQ (AMQP) Transport Options:**

```typescript
import { Transport, RmqOptions } from '@nestjs/microservices';

interface RmqOptions {
  urls?: string[];              // AMQP URLs
  queue?: string;               // Queue name
  prefetchCount?: number;       // Prefetch count (default: 0)
  isGlobalPrefetchCount?: boolean;  // Global prefetch (default: false)
  queueOptions?: object;        // Queue options
  socketOptions?: object;       // Socket options
  noAck?: boolean;              // Auto-acknowledge (default: false)
}

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.RMQ,
      options: {
        // Connection URLs (supports clustering)
        urls: [
          'amqp://user:password@localhost:5672',
          'amqp://user:password@rabbitmq-2:5672',  // Fallback
        ],
        
        // Queue configuration
        queue: 'orders_queue',
        prefetchCount: 10,              // Process 10 messages at a time
        isGlobalPrefetchCount: false,
        
        // Queue options
        queueOptions: {
          durable: true,        // Survive broker restart
          exclusive: false,     // Not exclusive to this connection
          autoDelete: false,    // Don't auto-delete when unused
          arguments: {
            'x-message-ttl': 60000,  // Message TTL: 60 seconds
            'x-max-length': 1000,     // Max queue length
          },
        },
        
        // Socket options
        socketOptions: {
          heartbeatIntervalInSeconds: 60,
          reconnectTimeInSeconds: 5,
        },
        
        noAck: false,  // Require manual acknowledgment
      },
    },
  );
  
  await app.listen();
}
```

**4. Kafka Transport Options:**

```typescript
import { Transport, KafkaOptions } from '@nestjs/microservices';
import { Partitioners } from 'kafkajs';

interface KafkaOptions {
  client?: {              // Client configuration
    clientId?: string;
    brokers?: string[];
    ssl?: boolean;
    sasl?: object;
  };
  consumer?: {            // Consumer configuration
    groupId: string;
    sessionTimeout?: number;
    rebalanceTimeout?: number;
    heartbeatInterval?: number;
  };
  producer?: {            // Producer configuration
    allowAutoTopicCreation?: boolean;
    transactionTimeout?: number;
  };
  subscribe?: {           // Subscription options
    fromBeginning?: boolean;
  };
}

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.KAFKA,
      options: {
        // Client configuration
        client: {
          clientId: 'my-app',
          brokers: [
            'localhost:9092',
            'kafka-2:9092',  // Cluster support
          ],
          
          // SSL configuration
          ssl: true,
          
          // SASL authentication
          sasl: {
            mechanism: 'plain',
            username: 'kafka-user',
            password: 'kafka-password',
          },
        },
        
        // Consumer configuration
        consumer: {
          groupId: 'my-consumer-group',
          sessionTimeout: 30000,      // 30 seconds
          rebalanceTimeout: 60000,    // 60 seconds
          heartbeatInterval: 3000,    // 3 seconds
        },
        
        // Producer configuration
        producer: {
          allowAutoTopicCreation: true,
          transactionTimeout: 30000,
          createPartitioner: Partitioners.DefaultPartitioner,
        },
        
        // Subscription options
        subscribe: {
          fromBeginning: false,  // Start from latest offset
        },
      },
    },
  );
  
  await app.listen();
}
```

**5. gRPC Transport Options:**

```typescript
import { Transport, GrpcOptions } from '@nestjs/microservices';
import { join } from 'path';

interface GrpcOptions {
  package: string | string[];       // Package name(s)
  protoPath: string | string[];     // Proto file path(s)
  url?: string;                     // Server URL
  maxSendMessageLength?: number;    // Max send size
  maxReceiveMessageLength?: number; // Max receive size
  credentials?: any;                // Server credentials
  loader?: object;                  // Proto loader options
}

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.GRPC,
      options: {
        // Package and proto file
        package: ['hero', 'user'],  // Multiple packages
        protoPath: [
          join(__dirname, './protos/hero.proto'),
          join(__dirname, './protos/user.proto'),
        ],
        
        // Server URL
        url: '0.0.0.0:5000',
        
        // Message size limits
        maxSendMessageLength: 4 * 1024 * 1024,     // 4MB
        maxReceiveMessageLength: 4 * 1024 * 1024,  // 4MB
        
        // Proto loader options
        loader: {
          keepCase: true,          // Keep field names as-is
          longs: String,           // Convert longs to strings
          enums: String,           // Convert enums to strings
          defaults: true,          // Set default values
          oneofs: true,            // Virtual oneof properties
        },
      },
    },
  );
  
  await app.listen();
}
```

**6. MQTT Transport Options:**

```typescript
import { Transport, MqttOptions } from '@nestjs/microservices';

interface MqttOptions {
  url?: string;           // MQTT broker URL
  clientId?: string;      // Client ID
  username?: string;      // Username
  password?: string;      // Password
  clean?: boolean;        // Clean session (default: true)
  keepalive?: number;     // Keepalive interval (default: 60)
  reconnectPeriod?: number;  // Reconnect period (default: 1000)
  qos?: 0 | 1 | 2;       // Quality of Service
  retain?: boolean;       // Retain messages
}

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.MQTT,
      options: {
        url: 'mqtt://localhost:1883',
        clientId: 'nestjs-mqtt-' + Math.random().toString(16).substr(2, 8),
        username: 'mqtt-user',
        password: 'mqtt-pass',
        clean: true,              // Clean session
        keepalive: 60,            // 60 seconds
        reconnectPeriod: 1000,    // 1 second
        qos: 2,                   // Exactly once delivery
        retain: false,            // Don't retain messages
      },
    },
  );
  
  await app.listen();
}
```

**7. NATS Transport Options:**

```typescript
import { Transport, NatsOptions } from '@nestjs/microservices';

interface NatsOptions {
  servers?: string | string[];  // NATS server URLs
  queue?: string;               // Queue group name
  user?: string;                // Username
  pass?: string;                // Password
  token?: string;               // Token authentication
  maxReconnectAttempts?: number;  // Max reconnect attempts
  reconnectTimeWait?: number;   // Reconnect wait time (ms)
  tls?: object;                 // TLS options
}

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.NATS,
      options: {
        // NATS servers (supports clustering)
        servers: [
          'nats://localhost:4222',
          'nats://nats-2:4222',  // Fallback
        ],
        
        // Queue group (load balancing)
        queue: 'nestjs-queue',
        
        // Authentication
        user: 'nats-user',
        pass: 'nats-password',
        // OR token authentication
        // token: 'my-auth-token',
        
        // Reconnection
        maxReconnectAttempts: -1,   // Infinite reconnects
        reconnectTimeWait: 2000,     // 2 seconds
        
        // TLS configuration
        tls: {
          caFile: './ca.pem',
          certFile: './cert.pem',
          keyFile: './key.pem',
        },
      },
    },
  );
  
  await app.listen();
}
```

**Client-Side Configuration:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'AUTH_SERVICE',
        transport: Transport.TCP,
        options: {
          host: 'auth-service',  // Service name in Docker/K8s
          port: 3001,
          retryAttempts: 5,
          retryDelay: 3000,
        },
      },
      
      {
        name: 'REDIS_SERVICE',
        transport: Transport.REDIS,
        options: {
          host: 'redis',
          port: 6379,
          password: process.env.REDIS_PASSWORD,
        },
      },
      
      {
        name: 'RABBITMQ_SERVICE',
        transport: Transport.RMQ,
        options: {
          urls: [process.env.RABBITMQ_URL],
          queue: 'tasks_queue',
          queueOptions: { durable: true },
        },
      },
    ]),
  ],
})
export class AppModule {}
```

**Environment-Based Configuration:**

```typescript
// config/microservice.config.ts
import { Transport, MicroserviceOptions } from '@nestjs/microservices';

export const getMicroserviceConfig = (): MicroserviceOptions => {
  const transport = process.env.TRANSPORT || 'TCP';
  
  switch (transport) {
    case 'TCP':
      return {
        transport: Transport.TCP,
        options: {
          host: process.env.TCP_HOST || 'localhost',
          port: parseInt(process.env.TCP_PORT) || 3001,
          retryAttempts: parseInt(process.env.RETRY_ATTEMPTS) || 5,
          retryDelay: parseInt(process.env.RETRY_DELAY) || 3000,
        },
      };
      
    case 'REDIS':
      return {
        transport: Transport.REDIS,
        options: {
          host: process.env.REDIS_HOST || 'localhost',
          port: parseInt(process.env.REDIS_PORT) || 6379,
          password: process.env.REDIS_PASSWORD,
        },
      };
      
    case 'RMQ':
      return {
        transport: Transport.RMQ,
        options: {
          urls: [process.env.RABBITMQ_URL],
          queue: process.env.QUEUE_NAME,
          queueOptions: {
            durable: true,
          },
        },
      };
      
    case 'KAFKA':
      return {
        transport: Transport.KAFKA,
        options: {
          client: {
            clientId: process.env.KAFKA_CLIENT_ID,
            brokers: process.env.KAFKA_BROKERS.split(','),
          },
          consumer: {
            groupId: process.env.KAFKA_GROUP_ID,
          },
        },
      };
      
    default:
      throw new Error(`Unsupported transport: ${transport}`);
  }
};
```

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { getMicroserviceConfig } from './config/microservice.config';

async function bootstrap() {
  const app = await NestFactory.createMicroservice(
    AppModule,
    getMicroserviceConfig(),  // Load from environment
  );
  
  await app.listen();
}
bootstrap();
```

```bash
# .env
TRANSPORT=RMQ
RABBITMQ_URL=amqp://user:pass@localhost:5672
QUEUE_NAME=orders_queue
RETRY_ATTEMPTS=10
RETRY_DELAY=3000
```

**Configuration with ConfigService:**

```typescript
// microservice-config.service.ts
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';

@Injectable()
export class MicroserviceConfigService {
  constructor(private configService: ConfigService) {}

  getMicroserviceOptions(): MicroserviceOptions {
    const transport = this.configService.get('TRANSPORT', 'TCP');
    
    return {
      transport: Transport[transport],
      options: this.getTransportOptions(transport),
    };
  }

  private getTransportOptions(transport: string) {
    switch (transport) {
      case 'TCP':
        return {
          host: this.configService.get('TCP_HOST', 'localhost'),
          port: this.configService.get('TCP_PORT', 3001),
          retryAttempts: this.configService.get('RETRY_ATTEMPTS', 5),
        };
        
      case 'REDIS':
        return {
          host: this.configService.get('REDIS_HOST', 'localhost'),
          port: this.configService.get('REDIS_PORT', 6379),
          password: this.configService.get('REDIS_PASSWORD'),
        };
        
      // Add other transports...
    }
  }
}
```

**Best Practices:**

```typescript
// ✅ GOOD: Use environment variables
options: {
  host: process.env.REDIS_HOST || 'localhost',
  port: parseInt(process.env.REDIS_PORT) || 6379,
}

// ✅ GOOD: Set appropriate retry strategies
options: {
  retryAttempts: 10,
  retryDelay: 3000,
  retryStrategy: (times) => Math.min(times * 1000, 5000),
}

// ✅ GOOD: Configure queue durability for RabbitMQ
queueOptions: {
  durable: true,  // Survive broker restart
  autoDelete: false,
}

// ✅ GOOD: Set message size limits for gRPC
options: {
  maxSendMessageLength: 4 * 1024 * 1024,     // 4MB
  maxReceiveMessageLength: 4 * 1024 * 1024,
}

// ✅ GOOD: Use clustering for high availability
urls: [
  'amqp://rabbitmq-1:5672',
  'amqp://rabbitmq-2:5672',
  'amqp://rabbitmq-3:5672',
]

// ❌ BAD: Hardcoding credentials
options: {
  username: 'admin',  // Don't hardcode!
  password: 'password123',
}

// ❌ BAD: Not setting retry strategies
options: {
  host: 'localhost',
  port: 3001,
  // Missing retryAttempts and retryDelay
}

// ❌ BAD: Using non-durable queues in production
queueOptions: {
  durable: false,  // Data loss on restart!
}
```

**Interview Tip**: Transport options are configured in the **options** field of `createMicroservice()` or `ClientsModule.register()`. Each transport has **specific options**: **TCP** (host, port, retryAttempts), **Redis** (host, port, password, db), **RabbitMQ** (urls, queue, queueOptions, prefetchCount), **Kafka** (client, consumer, producer, brokers), **gRPC** (package, protoPath, url, maxMessageLength). Use **environment variables** for configuration, **retryAttempts/retryDelay** for resilience, **durable queues** for RabbitMQ, **clustering** for HA. Access via **process.env** or **ConfigService**.

</details>

### 8. Can an application be both HTTP and Microservice?

<details>
<summary>Answer</summary>

**Yes**, an application can be **both HTTP and Microservice** simultaneously using a **Hybrid Application**. This allows a single NestJS app to serve HTTP requests (REST API) and handle microservice messages at the same time.

**Why Use Hybrid Applications?**

```
✅ Single codebase for HTTP API and microservice handlers
✅ Share services, modules, and business logic
✅ Reduce deployment complexity
✅ Act as API Gateway + Microservice
✅ Cost-effective for small to medium applications
```

**Creating a Hybrid Application:**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { Transport, MicroserviceOptions } from '@nestjs/microservices';
import { AppModule } from './app.module';

async function bootstrap() {
  // 1. Create HTTP application first
  const app = await NestFactory.create(AppModule);

  // 2. Connect microservice to the same app
  app.connectMicroservice<MicroserviceOptions>({
    transport: Transport.TCP,
    options: {
      host: 'localhost',
      port: 3001,
    },
  });

  // 3. Start both HTTP and microservice
  await app.startAllMicroservices();  // Start microservice
  await app.listen(3000);              // Start HTTP server
  
  console.log('HTTP server on http://localhost:3000');
  console.log('Microservice listening on TCP port 3001');
}
bootstrap();
```

**Complete Hybrid Application Example:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';

@Module({
  controllers: [UsersController],  // Handles both HTTP and microservice
  providers: [UsersService],
})
export class AppModule {}
```

```typescript
// users.controller.ts
import { Controller, Get, Post, Body } from '@nestjs/common';
import { MessagePattern, EventPattern, Payload } from '@nestjs/microservices';
import { UsersService } from './users.service';

@Controller('users')  // HTTP routes
export class UsersController {
  constructor(private usersService: UsersService) {}

  // ✅ HTTP Endpoint (REST API)
  @Get()
  async getAllUsers() {
    return this.usersService.findAll();
  }

  // ✅ HTTP Endpoint (REST API)
  @Post()
  async createUser(@Body() data: { name: string; email: string }) {
    return this.usersService.create(data);
  }

  // ✅ Microservice Message Pattern (Request-Response)
  @MessagePattern('get-users')
  async getUsersViaMessage() {
    return this.usersService.findAll();
  }

  // ✅ Microservice Message Pattern (Request-Response)
  @MessagePattern('create-user')
  async createUserViaMessage(@Payload() data: { name: string; email: string }) {
    return this.usersService.create(data);
  }

  // ✅ Microservice Event Pattern (Fire and Forget)
  @EventPattern('user-created')
  handleUserCreated(@Payload() data: { userId: string; email: string }) {
    console.log('User created event received:', data);
    // Process event (send email, update cache, etc.)
  }
}
```

```typescript
// users.service.ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class UsersService {
  private users = [
    { id: 1, name: 'John', email: 'john@example.com' },
    { id: 2, name: 'Jane', email: 'jane@example.com' },
  ];

  findAll() {
    return this.users;
  }

  create(data: { name: string; email: string }) {
    const newUser = {
      id: this.users.length + 1,
      ...data,
    };
    this.users.push(newUser);
    return newUser;
  }
}
```

**Accessing the Hybrid Application:**

```bash
# 1. HTTP REST API
curl http://localhost:3000/users
# Response: [{id: 1, name: 'John', email: 'john@example.com'}, ...]

curl -X POST http://localhost:3000/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice", "email": "alice@example.com"}'
# Response: {id: 3, name: 'Alice', email: 'alice@example.com'}

# 2. Microservice (via client)
# See client code below
```

```typescript
// Test microservice part (client.ts)
import { ClientProxyFactory, Transport } from '@nestjs/microservices';
import { firstValueFrom } from 'rxjs';

async function testMicroservice() {
  // Create client
  const client = ClientProxyFactory.create({
    transport: Transport.TCP,
    options: { host: 'localhost', port: 3001 },
  });

  await client.connect();

  // Call microservice message pattern
  const users = await firstValueFrom(
    client.send('get-users', {}),
  );
  console.log('Users via microservice:', users);

  // Create user via microservice
  const newUser = await firstValueFrom(
    client.send('create-user', {
      name: 'Bob',
      email: 'bob@example.com',
    }),
  );
  console.log('Created user:', newUser);

  // Emit event
  client.emit('user-created', {
    userId: '123',
    email: 'user@example.com',
  });

  await client.close();
}

testMicroservice();
```

**Multiple Microservice Transports:**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { Transport, MicroserviceOptions } from '@nestjs/microservices';
import { AppModule } from './app.module';

async function bootstrap() {
  // Create HTTP application
  const app = await NestFactory.create(AppModule);

  // Connect TCP microservice
  app.connectMicroservice<MicroserviceOptions>({
    transport: Transport.TCP,
    options: {
      host: 'localhost',
      port: 3001,
    },
  });

  // Connect Redis microservice
  app.connectMicroservice<MicroserviceOptions>({
    transport: Transport.REDIS,
    options: {
      host: 'localhost',
      port: 6379,
    },
  });

  // Connect RabbitMQ microservice
  app.connectMicroservice<MicroserviceOptions>({
    transport: Transport.RMQ,
    options: {
      urls: ['amqp://localhost:5672'],
      queue: 'events_queue',
      queueOptions: { durable: true },
    },
  });

  // Start all microservices and HTTP server
  await app.startAllMicroservices();
  await app.listen(3000);
  
  console.log('HTTP API on port 3000');
  console.log('TCP microservice on port 3001');
  console.log('Redis microservice on port 6379');
  console.log('RabbitMQ microservice listening');
}
bootstrap();
```

**API Gateway Pattern (Hybrid Application):**

```typescript
// API Gateway receives HTTP requests and forwards to microservices
// gateway/main.ts
import { NestFactory } from '@nestjs/core';
import { Transport } from '@nestjs/microservices';
import { GatewayModule } from './gateway.module';

async function bootstrap() {
  // HTTP API Gateway
  const app = await NestFactory.create(GatewayModule);
  
  await app.listen(3000);
  console.log('API Gateway on http://localhost:3000');
}
bootstrap();
```

```typescript
// gateway/gateway.module.ts
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';
import { GatewayController } from './gateway.controller';

@Module({
  imports: [
    // Register microservice clients
    ClientsModule.register([
      {
        name: 'AUTH_SERVICE',
        transport: Transport.TCP,
        options: { host: 'auth-service', port: 3001 },
      },
      {
        name: 'USERS_SERVICE',
        transport: Transport.TCP,
        options: { host: 'users-service', port: 3002 },
      },
      {
        name: 'ORDERS_SERVICE',
        transport: Transport.TCP,
        options: { host: 'orders-service', port: 3003 },
      },
    ]),
  ],
  controllers: [GatewayController],
})
export class GatewayModule {}
```

```typescript
// gateway/gateway.controller.ts
import { Controller, Get, Post, Body, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { firstValueFrom } from 'rxjs';

@Controller()
export class GatewayController {
  constructor(
    @Inject('AUTH_SERVICE') private authClient: ClientProxy,
    @Inject('USERS_SERVICE') private usersClient: ClientProxy,
    @Inject('ORDERS_SERVICE') private ordersClient: ClientProxy,
  ) {}

  // HTTP endpoint that calls Auth microservice
  @Post('auth/login')
  async login(@Body() credentials: { email: string; password: string }) {
    const result = await firstValueFrom(
      this.authClient.send('login', credentials),
    );
    return result;
  }

  // HTTP endpoint that calls Users microservice
  @Get('users')
  async getUsers() {
    const users = await firstValueFrom(
      this.usersClient.send('get-users', {}),
    );
    return users;
  }

  // HTTP endpoint that calls Orders microservice
  @Post('orders')
  async createOrder(@Body() orderData: any) {
    const order = await firstValueFrom(
      this.ordersClient.send('create-order', orderData),
    );
    return order;
  }

  // Aggregate data from multiple microservices
  @Get('dashboard')
  async getDashboard() {
    const [users, orders] = await Promise.all([
      firstValueFrom(this.usersClient.send('get-users', {})),
      firstValueFrom(this.ordersClient.send('get-orders', {})),
    ]);
    
    return {
      usersCount: users.length,
      ordersCount: orders.length,
      users,
      orders,
    };
  }
}
```

**Architecture Diagram:**

```
┌─────────────────────────────────────────────────┐
│            Hybrid Application                   │
│                                                 │
│  ┌──────────────┐      ┌──────────────┐       │
│  │ HTTP Server  │      │ Microservice │       │
│  │  Port 3000   │      │   Port 3001  │       │
│  └──────┬───────┘      └──────┬───────┘       │
│         │                     │                │
│         └────────┬────────────┘                │
│                  │                             │
│         ┌────────▼────────┐                   │
│         │  UsersController │                   │
│         │  @Get() /users   │                   │
│         │  @MessagePattern │                   │
│         └────────┬────────┘                   │
│                  │                             │
│         ┌────────▼────────┐                   │
│         │  UsersService    │                   │
│         └──────────────────┘                   │
└─────────────────────────────────────────────────┘

Access:
  - HTTP:  curl http://localhost:3000/users
  - TCP:   client.send('get-users', {})
```

**Use Cases:**

```
1. API Gateway + Microservice
   - Accept HTTP requests from clients
   - Communicate with other services via microservice protocols

2. Monolith to Microservices Migration
   - Start with hybrid app
   - Gradually split into separate microservices

3. Small to Medium Applications
   - Cost-effective (single deployment)
   - Easy to develop and test

4. Internal Tools
   - Web UI via HTTP
   - Background tasks via microservice events
```

**Best Practices:**

```typescript
// ✅ GOOD: Separate HTTP and microservice handlers
@Get('users')  // HTTP
getUsersHttp() { ... }

@MessagePattern('get-users')  // Microservice
getUsersMessage() { ... }

// ✅ GOOD: Share business logic in services
getUsersHttp() {
  return this.usersService.findAll();  // Shared service
}

@MessagePattern('get-users')
getUsersMessage() {
  return this.usersService.findAll();  // Same service
}

// ✅ GOOD: Start microservices before HTTP
await app.startAllMicroservices();  // First
await app.listen(3000);              // Then

// ✅ GOOD: Use different ports for HTTP and microservice
HTTP: 3000
TCP:  3001

// ❌ BAD: Same port for HTTP and microservice
// Not supported - will cause conflicts

// ❌ BAD: Forgetting to start microservices
await app.listen(3000);  // Only HTTP, microservice won't work!

// ❌ BAD: Mixing concerns in one method
@Get('users')
@MessagePattern('get-users')  // Don't combine!
getUsers() { ... }
```

**Interview Tip**: **Yes**, use **hybrid applications** with `app.connectMicroservice()` + `app.startAllMicroservices()`. Create HTTP app with **NestFactory.create()**, then **connect microservices** with `connectMicroservice()`. **Single controller** can handle both **@Get() routes** (HTTP) and **@MessagePattern()** (microservice). Start with **startAllMicroservices()** then **listen()** for HTTP. Use for: **API Gateway** (HTTP frontend + microservice backend), **monolith migration**, **small apps**. Can connect **multiple transports** (TCP, Redis, RabbitMQ) to same app. Share **services and modules** between HTTP and microservice handlers.

</details>

## Message Patterns

### 9. What are Message Patterns?

<details>
<summary>Answer</summary>

**Message Patterns** are **identifiers** that define which handler method should process an incoming message in a microservice. They act like **routes** in HTTP applications but for microservice communication.

**Two Types of Message Patterns:**

```
1. @MessagePattern() ✅
   - Request-Response pattern
   - Waits for and returns a response
   - Synchronous behavior
   - Like REST API call

2. @EventPattern() ✅
   - Event-based pattern
   - Fire-and-forget
   - No response expected
   - Asynchronous behavior
   - Like pub/sub messaging
```

**Message Pattern Syntax:**

```typescript
import { Controller } from '@nestjs/common';
import { MessagePattern, EventPattern, Payload } from '@nestjs/microservices';

@Controller()
export class AppController {
  // String pattern
  @MessagePattern('get-user')
  getUser(@Payload() data: { id: string }) {
    return { id: data.id, name: 'John Doe' };
  }

  // Object pattern (for complex routing)
  @MessagePattern({ cmd: 'create-user', version: '1' })
  createUser(@Payload() data: any) {
    return { success: true };
  }

  // Event pattern (no response)
  @EventPattern('user-created')
  handleUserCreated(@Payload() data: any) {
    console.log('User created:', data);
    // No return needed
  }
}
```

**Pattern Types:**

```typescript
// 1. String Patterns (Simple)
@MessagePattern('add')
add(data: { a: number; b: number }) {
  return { result: data.a + data.b };
}

// 2. Dot-Notation Patterns (Namespaced)
@MessagePattern('user.create')
createUser(data: any) { ... }

@MessagePattern('order.create')
createOrder(data: any) { ... }

@MessagePattern('product.update')
updateProduct(data: any) { ... }

// 3. Object Patterns (Complex Routing)
@MessagePattern({ cmd: 'get-user', version: 'v1' })
getUserV1(data: any) { ... }

@MessagePattern({ cmd: 'get-user', version: 'v2' })
getUserV2(data: any) { ... }

@MessagePattern({ service: 'auth', action: 'login' })
login(data: any) { ... }
```

**Complete Example:**

```typescript
// math.controller.ts
import { Controller } from '@nestjs/common';
import { MessagePattern, EventPattern, Payload, Ctx } from '@nestjs/microservices';

@Controller()
export class MathController {
  // Request-Response: Simple string pattern
  @MessagePattern('add')
  add(@Payload() data: { a: number; b: number }) {
    console.log('Received add request:', data);
    return {
      operation: 'add',
      result: data.a + data.b,
    };
  }

  // Request-Response: Dot-notation pattern
  @MessagePattern('math.multiply')
  multiply(@Payload() data: { a: number; b: number }) {
    return {
      operation: 'multiply',
      result: data.a * data.b,
    };
  }

  // Request-Response: Object pattern
  @MessagePattern({ cmd: 'divide', version: '1' })
  divide(@Payload() data: { a: number; b: number }) {
    if (data.b === 0) {
      throw new Error('Division by zero');
    }
    return {
      operation: 'divide',
      result: data.a / data.b,
    };
  }

  // Event: Fire-and-forget
  @EventPattern('calculation-logged')
  handleCalculationLogged(@Payload() data: any) {
    console.log('Calculation logged:', data);
    // Save to database, send notification, etc.
    // No return value
  }

  // With context
  @MessagePattern('subtract')
  subtract(
    @Payload() data: { a: number; b: number },
    @Ctx() context: any,
  ) {
    console.log('Context:', context);
    return {
      operation: 'subtract',
      result: data.a - data.b,
    };
  }
}
```

**Client-Side Usage:**

```typescript
// app.controller.ts (Client)
import { Controller, Get, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { firstValueFrom } from 'rxjs';

@Controller('calculator')
export class AppController {
  constructor(
    @Inject('MATH_SERVICE') private mathClient: ClientProxy,
  ) {}

  // Call simple string pattern
  @Get('add')
  async add() {
    const response = await firstValueFrom(
      this.mathClient.send('add', { a: 10, b: 5 }),
    );
    return response;  // { operation: 'add', result: 15 }
  }

  // Call dot-notation pattern
  @Get('multiply')
  async multiply() {
    const response = await firstValueFrom(
      this.mathClient.send('math.multiply', { a: 10, b: 5 }),
    );
    return response;  // { operation: 'multiply', result: 50 }
  }

  // Call object pattern
  @Get('divide')
  async divide() {
    const response = await firstValueFrom(
      this.mathClient.send({ cmd: 'divide', version: '1' }, { a: 10, b: 2 }),
    );
    return response;  // { operation: 'divide', result: 5 }
  }

  // Emit event (no response)
  @Get('log')
  emitLog() {
    this.mathClient.emit('calculation-logged', {
      timestamp: new Date(),
      operation: 'add',
      result: 15,
    });
    return { message: 'Event emitted' };
  }
}
```

**Pattern Matching Rules:**

```typescript
// Server
@MessagePattern('user.create')
createUser(data: any) { ... }

// Client - MUST match exactly
client.send('user.create', data)  ✅ Matches
client.send('user.update', data)  ❌ No match
client.send('create', data)       ❌ No match

// Object patterns - ALL properties must match
@MessagePattern({ cmd: 'get', version: '1' })

client.send({ cmd: 'get', version: '1' }, data)  ✅ Matches
client.send({ cmd: 'get' }, data)                 ❌ No match (missing version)
client.send({ cmd: 'get', version: '2' }, data)  ❌ No match (wrong version)
```

**Wildcard Patterns (Transport-Specific):**

```typescript
// NATS supports wildcards
@MessagePattern('user.*')  // Matches: user.create, user.update, user.delete
handleUserEvents(data: any) { ... }

@MessagePattern('order.*.created')  // Matches: order.123.created, order.456.created
handleOrderCreated(data: any) { ... }

// RabbitMQ supports routing keys
@MessagePattern('logs.*.error')  // Matches: logs.auth.error, logs.db.error
handleErrors(data: any) { ... }
```

**Pattern Organization:**

```typescript
// ✅ GOOD: Use namespacing
@MessagePattern('user.create')
@MessagePattern('user.update')
@MessagePattern('user.delete')

@MessagePattern('order.create')
@MessagePattern('order.cancel')

// ✅ GOOD: Use versioning for API evolution
@MessagePattern({ cmd: 'get-user', version: 'v1' })
getUserV1(data: any) { return oldFormat; }

@MessagePattern({ cmd: 'get-user', version: 'v2' })
getUserV2(data: any) { return newFormat; }

// ✅ GOOD: Use descriptive names
@MessagePattern('order.payment.process')
@MessagePattern('user.email.verify')

// ❌ BAD: Generic names
@MessagePattern('process')
@MessagePattern('handle')
@MessagePattern('action')
```

**Real-World Example (E-commerce):**

```typescript
// orders.controller.ts
import { Controller } from '@nestjs/common';
import { MessagePattern, EventPattern, Payload } from '@nestjs/microservices';
import { OrdersService } from './orders.service';

@Controller()
export class OrdersController {
  constructor(private ordersService: OrdersService) {}

  // Request-Response patterns
  @MessagePattern('order.create')
  async createOrder(@Payload() data: any) {
    const order = await this.ordersService.create(data);
    return order;
  }

  @MessagePattern('order.get')
  async getOrder(@Payload() data: { orderId: string }) {
    return await this.ordersService.findById(data.orderId);
  }

  @MessagePattern('order.list')
  async listOrders(@Payload() data: { userId: string }) {
    return await this.ordersService.findByUser(data.userId);
  }

  @MessagePattern('order.cancel')
  async cancelOrder(@Payload() data: { orderId: string }) {
    return await this.ordersService.cancel(data.orderId);
  }

  // Event patterns (fire-and-forget)
  @EventPattern('order.created')
  async handleOrderCreated(@Payload() data: any) {
    // Send confirmation email
    console.log('Order created:', data);
  }

  @EventPattern('order.shipped')
  async handleOrderShipped(@Payload() data: any) {
    // Update tracking, send notification
    console.log('Order shipped:', data);
  }

  @EventPattern('payment.completed')
  async handlePaymentCompleted(@Payload() data: any) {
    // Update order status
    await this.ordersService.markAsPaid(data.orderId);
  }
}
```

```typescript
// Client usage
@Controller('orders')
export class OrdersGatewayController {
  constructor(
    @Inject('ORDERS_SERVICE') private ordersClient: ClientProxy,
  ) {}

  @Post()
  async createOrder(@Body() orderData: any) {
    // Request-response: wait for order creation
    const order = await firstValueFrom(
      this.ordersClient.send('order.create', orderData),
    );
    
    // Fire event: don't wait for response
    this.ordersClient.emit('order.created', order);
    
    return order;
  }

  @Get(':id')
  async getOrder(@Param('id') id: string) {
    return await firstValueFrom(
      this.ordersClient.send('order.get', { orderId: id }),
    );
  }

  @Post(':id/ship')
  async shipOrder(@Param('id') id: string) {
    // Emit event to multiple services
    this.ordersClient.emit('order.shipped', {
      orderId: id,
      timestamp: new Date(),
    });
    
    return { message: 'Order shipping initiated' };
  }
}
```

**Pattern vs Route Comparison:**

| HTTP (REST) | Microservice (Message Pattern) |
|-------------|--------------------------------|
| `@Get('users')` | `@MessagePattern('get-users')` |
| `@Post('users')` | `@MessagePattern('create-user')` |
| `@Put('users/:id')` | `@MessagePattern('update-user')` |
| `@Delete('users/:id')` | `@MessagePattern('delete-user')` |
| `@Get('users/:id')` | `@MessagePattern('find-user')` |
| N/A | `@EventPattern('user-created')` |

**Best Practices:**

```typescript
// ✅ GOOD: Use consistent naming conventions
@MessagePattern('entity.action')  // user.create, order.update

// ✅ GOOD: Separate commands and events
@MessagePattern('order.create')   // Command (request-response)
@EventPattern('order.created')    // Event (fire-and-forget)

// ✅ GOOD: Version your patterns
@MessagePattern({ cmd: 'get-user', v: '1' })
@MessagePattern({ cmd: 'get-user', v: '2' })

// ✅ GOOD: Use typed payloads
@MessagePattern('create-user')
createUser(@Payload() data: CreateUserDto) { ... }

// ✅ GOOD: Document patterns
/**
 * Creates a new user account
 * @pattern user.create
 * @payload { email: string, name: string }
 * @returns { id: string, email: string, name: string }
 */
@MessagePattern('user.create')

// ❌ BAD: Duplicate patterns
@MessagePattern('get-user')  // In UserController
@MessagePattern('get-user')  // In AdminController - CONFLICT!

// ❌ BAD: Unclear patterns
@MessagePattern('process')
@MessagePattern('handle')
@MessagePattern('do')

// ❌ BAD: Too generic
@MessagePattern('action')
handleAction(data: any) { ... }  // What action?
```

**Interview Tip**: **Message Patterns** are **identifiers** for routing messages in microservices. Two types: **@MessagePattern()** (request-response, returns value, synchronous) and **@EventPattern()** (fire-and-forget, no response, asynchronous). Patterns can be **strings** ('user.create'), **dot-notation** ('order.payment.process'), or **objects** ({ cmd: 'get', version: '1' }). Client uses **client.send(pattern, data)** for @MessagePattern and **client.emit(pattern, data)** for @EventPattern. Pattern must **match exactly** on both sides. Use **namespacing** (entity.action), **versioning** for API evolution, **consistent naming**. Like **routes** in HTTP but for microservices.

</details>

### 10. What is `@MessagePattern()` decorator?

<details>
<summary>Answer</summary>

**`@MessagePattern()`** is a **decorator** that marks a method as a **request-response message handler** in a microservice. It waits for a response and returns it to the caller, similar to HTTP request-response.

**Syntax:**

```typescript
import { MessagePattern, Payload } from '@nestjs/microservices';

@MessagePattern(pattern)
methodName(@Payload() data: any) {
  // Process data
  return result;  // Response sent back to caller
}
```

**Basic Usage:**

```typescript
// Server: math.controller.ts
import { Controller } from '@nestjs/common';
import { MessagePattern, Payload } from '@nestjs/microservices';

@Controller()
export class MathController {
  // Simple request-response
  @MessagePattern('add')
  add(@Payload() data: { a: number; b: number }) {
    console.log('Received:', data);
    
    return {
      result: data.a + data.b,
      timestamp: new Date(),
    };
  }
}
```

```typescript
// Client: app.controller.ts
import { Controller, Get, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { firstValueFrom } from 'rxjs';

@Controller()
export class AppController {
  constructor(
    @Inject('MATH_SERVICE') private mathClient: ClientProxy,
  ) {}

  @Get('add')
  async calculate() {
    // Send message and wait for response
    const response = await firstValueFrom(
      this.mathClient.send('add', { a: 5, b: 3 }),
    );
    
    return response;  // { result: 8, timestamp: '2025-12-28...' }
  }
}
```

**Pattern Types:**

```typescript
import { Controller } from '@nestjs/common';
import { MessagePattern, Payload } from '@nestjs/microservices';

@Controller()
export class UsersController {
  // 1. String pattern
  @MessagePattern('get-user')
  getUser(@Payload() data: { id: string }) {
    return { id: data.id, name: 'John', email: 'john@example.com' };
  }

  // 2. Dot-notation pattern (namespaced)
  @MessagePattern('user.create')
  createUser(@Payload() data: { name: string; email: string }) {
    return { id: '123', ...data };
  }

  // 3. Object pattern (complex routing)
  @MessagePattern({ cmd: 'get-user', version: 'v1' })
  getUserV1(@Payload() data: { id: string }) {
    return { id: data.id, name: 'John' };  // Old format
  }

  @MessagePattern({ cmd: 'get-user', version: 'v2' })
  getUserV2(@Payload() data: { id: string }) {
    return {
      user: { id: data.id, name: 'John' },
      metadata: { version: 'v2' },
    };  // New format
  }
}
```

**Using @Payload() and @Ctx():**

```typescript
import { Controller } from '@nestjs/common';
import { MessagePattern, Payload, Ctx } from '@nestjs/microservices';

@Controller()
export class AppController {
  // Extract payload data
  @MessagePattern('process')
  process(@Payload() data: any) {
    console.log('Data:', data);
    return { processed: true };
  }

  // Extract specific payload properties
  @MessagePattern('create-user')
  createUser(
    @Payload('email') email: string,
    @Payload('name') name: string,
  ) {
    return { email, name, id: '123' };
  }

  // Access message context (metadata, headers)
  @MessagePattern('advanced')
  advanced(
    @Payload() data: any,
    @Ctx() context: any,
  ) {
    console.log('Context:', context);
    console.log('Pattern:', context.getPattern());
    
    return { success: true };
  }
}
```

**Async/Promise Support:**

```typescript
import { Controller } from '@nestjs/common';
import { MessagePattern, Payload } from '@nestjs/microservices';

@Controller()
export class UsersController {
  constructor(private usersService: UsersService) {}

  // ✅ Async/await with database
  @MessagePattern('get-user')
  async getUser(@Payload() data: { id: string }) {
    const user = await this.usersService.findById(data.id);
    return user;
  }

  // ✅ Promise
  @MessagePattern('create-user')
  createUser(@Payload() data: CreateUserDto): Promise<User> {
    return this.usersService.create(data);
  }

  // ✅ Observable
  @MessagePattern('list-users')
  listUsers(): Observable<User[]> {
    return from(this.usersService.findAll());
  }

  // ✅ Synchronous
  @MessagePattern('validate')
  validate(@Payload() data: any) {
    return { valid: true };
  }
}
```

**Error Handling:**

```typescript
import { Controller } from '@nestjs/common';
import { MessagePattern, Payload, RpcException } from '@nestjs/microservices';

@Controller()
export class UsersController {
  @MessagePattern('get-user')
  async getUser(@Payload() data: { id: string }) {
    const user = await this.usersService.findById(data.id);
    
    if (!user) {
      // ✅ Throw RpcException (sent to client)
      throw new RpcException('User not found');
    }
    
    return user;
  }

  @MessagePattern('divide')
  divide(@Payload() data: { a: number; b: number }) {
    if (data.b === 0) {
      // ✅ RpcException with custom error
      throw new RpcException({
        statusCode: 400,
        message: 'Division by zero',
        error: 'Bad Request',
      });
    }
    
    return { result: data.a / data.b };
  }

  @MessagePattern('risky-operation')
  async riskyOperation(@Payload() data: any) {
    try {
      const result = await this.service.doSomething(data);
      return result;
    } catch (error) {
      // ✅ Catch and rethrow as RpcException
      throw new RpcException(error.message);
    }
  }
}
```

```typescript
// Client: Error handling
import { catchError } from 'rxjs/operators';
import { throwError } from 'rxjs';

@Controller()
export class AppController {
  constructor(@Inject('USER_SERVICE') private userClient: ClientProxy) {}

  @Get('user/:id')
  async getUser(@Param('id') id: string) {
    try {
      const user = await firstValueFrom(
        this.userClient.send('get-user', { id }).pipe(
          catchError(error => {
            console.error('Microservice error:', error);
            return throwError(() => new NotFoundException(error.message));
          }),
        ),
      );
      
      return user;
    } catch (error) {
      throw new NotFoundException('User not found');
    }
  }
}
```

**Validation with DTOs:**

```typescript
// create-user.dto.ts
import { IsEmail, IsString, MinLength } from 'class-validator';

export class CreateUserDto {
  @IsString()
  @MinLength(2)
  name: string;

  @IsEmail()
  email: string;
}
```

```typescript
// users.controller.ts
import { Controller, UsePipes, ValidationPipe } from '@nestjs/common';
import { MessagePattern, Payload } from '@nestjs/microservices';
import { CreateUserDto } from './dto/create-user.dto';

@Controller()
export class UsersController {
  // ✅ Validation applied automatically
  @MessagePattern('create-user')
  @UsePipes(new ValidationPipe({ transform: true }))
  createUser(@Payload() data: CreateUserDto) {
    // data is validated and transformed
    return { id: '123', ...data };
  }
}
```

```typescript
// main.ts - Global validation
import { ValidationPipe } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.createMicroservice(AppModule, {
    transport: Transport.TCP,
    options: { port: 3001 },
  });

  // ✅ Enable global validation
  app.useGlobalPipes(
    new ValidationPipe({
      transform: true,
      whitelist: true,
      forbidNonWhitelisted: true,
    }),
  );

  await app.listen();
}
```

**Real-World Example (E-commerce):**

```typescript
// orders.controller.ts
import { Controller } from '@nestjs/common';
import { MessagePattern, Payload, RpcException } from '@nestjs/microservices';
import { OrdersService } from './orders.service';
import { CreateOrderDto } from './dto/create-order.dto';

@Controller()
export class OrdersController {
  constructor(
    private ordersService: OrdersService,
    private paymentsService: PaymentsService,
  ) {}

  // Create order
  @MessagePattern('order.create')
  async createOrder(@Payload() data: CreateOrderDto) {
    // Validate user
    const user = await this.usersService.findById(data.userId);
    if (!user) {
      throw new RpcException('User not found');
    }

    // Validate items
    const items = await this.productsService.validateItems(data.items);
    if (!items.allAvailable) {
      throw new RpcException('Some items are out of stock');
    }

    // Create order
    const order = await this.ordersService.create({
      userId: data.userId,
      items: data.items,
      total: items.total,
    });

    return order;
  }

  // Get order details
  @MessagePattern('order.get')
  async getOrder(@Payload() data: { orderId: string }) {
    const order = await this.ordersService.findById(data.orderId);
    
    if (!order) {
      throw new RpcException('Order not found');
    }
    
    return order;
  }

  // Process payment
  @MessagePattern('order.payment.process')
  async processPayment(@Payload() data: { orderId: string; paymentMethod: string }) {
    const order = await this.ordersService.findById(data.orderId);
    
    if (!order) {
      throw new RpcException('Order not found');
    }
    
    if (order.status === 'paid') {
      throw new RpcException('Order already paid');
    }

    // Process payment
    const payment = await this.paymentsService.charge({
      orderId: data.orderId,
      amount: order.total,
      method: data.paymentMethod,
    });

    // Update order status
    await this.ordersService.updateStatus(data.orderId, 'paid');

    return {
      orderId: order.id,
      paymentId: payment.id,
      status: 'paid',
    };
  }

  // List user orders
  @MessagePattern('order.list')
  async listOrders(@Payload() data: { userId: string; limit?: number }) {
    const orders = await this.ordersService.findByUser(
      data.userId,
      data.limit || 10,
    );
    
    return orders;
  }

  // Cancel order
  @MessagePattern('order.cancel')
  async cancelOrder(@Payload() data: { orderId: string; reason: string }) {
    const order = await this.ordersService.findById(data.orderId);
    
    if (!order) {
      throw new RpcException('Order not found');
    }
    
    if (order.status === 'shipped') {
      throw new RpcException('Cannot cancel shipped order');
    }

    // Cancel order
    await this.ordersService.cancel(data.orderId, data.reason);
    
    // Refund if paid
    if (order.status === 'paid') {
      await this.paymentsService.refund(order.paymentId);
    }

    return {
      orderId: order.id,
      status: 'cancelled',
      refunded: order.status === 'paid',
    };
  }
}
```

**Best Practices:**

```typescript
// ✅ GOOD: Use typed DTOs
@MessagePattern('create-user')
createUser(@Payload() data: CreateUserDto) { ... }

// ✅ GOOD: Handle errors with RpcException
@MessagePattern('get-user')
getUser(@Payload() data: { id: string }) {
  if (!user) {
    throw new RpcException('User not found');
  }
  return user;
}

// ✅ GOOD: Use async/await
@MessagePattern('get-user')
async getUser(@Payload() data: { id: string }) {
  const user = await this.usersService.findById(data.id);
  return user;
}

// ✅ GOOD: Return structured responses
@MessagePattern('process')
process(data: any) {
  return {
    success: true,
    data: result,
    timestamp: new Date(),
  };
}

// ✅ GOOD: Use validation
@UsePipes(new ValidationPipe())
@MessagePattern('create-user')
createUser(@Payload() data: CreateUserDto) { ... }

// ❌ BAD: No error handling
@MessagePattern('get-user')
getUser(data: { id: string }) {
  return this.usersService.findById(data.id);  // Can throw!
}

// ❌ BAD: Not using @Payload()
@MessagePattern('process')
process(data: any) {  // Missing @Payload()
  return data;  // data might be undefined
}

// ❌ BAD: No return value
@MessagePattern('create-user')
createUser(data: any) {
  this.usersService.create(data);
  // Missing return!  Client gets undefined
}

// ❌ BAD: Returning undefined
@MessagePattern('get-user')
getUser(data: { id: string }) {
  const user = this.usersService.findById(data.id);
  // No return!  Client gets undefined
}
```

**Interview Tip**: **@MessagePattern()** is a decorator for **request-response** message handling. Method **returns a value** sent back to caller. Use **@Payload()** to extract data, **@Ctx()** for context. Pattern can be **string**, **dot-notation**, or **object**. Supports **async/await**, **Promises**, **Observables**. Throw **RpcException** for errors. Client uses **client.send(pattern, data)** and **waits for response**. Different from **@EventPattern()**: MessagePattern **returns value**, EventPattern **no return**. Use for: **queries** (get-user), **commands** (create-order), **any operation needing response**. Enable **validation** with @UsePipes(ValidationPipe).

</details>

### 11. What is `@EventPattern()` decorator?

<details>
<summary>Answer</summary>

**`@EventPattern()`** is a **decorator** that marks a method as an **event handler** in a microservice. It follows the **fire-and-forget** pattern - the caller emits an event and **doesn't wait for a response**. Used for asynchronous, one-way communication.

**Syntax:**

```typescript
import { EventPattern, Payload } from '@nestjs/microservices';

@EventPattern(event)
methodName(@Payload() data: any) {
  // Process event
  // No return value needed
}
```

**Key Characteristics:**

```
✅ Fire-and-forget (no response)
✅ Asynchronous (non-blocking)
✅ One-way communication
✅ Can have multiple listeners
✅ Doesn't block caller
✅ No acknowledgment needed (by default)
```

**Basic Usage:**

```typescript
// Server: events.controller.ts
import { Controller } from '@nestjs/common';
import { EventPattern, Payload } from '@nestjs/microservices';

@Controller()
export class EventsController {
  // Handle user-created event
  @EventPattern('user-created')
  handleUserCreated(@Payload() data: { userId: string; email: string }) {
    console.log('User created event received:', data);
    
    // Process event:
    // - Send welcome email
    // - Update cache
    // - Notify other services
    // - Log to analytics
    
    // NO RETURN VALUE
  }

  // Handle order-shipped event
  @EventPattern('order-shipped')
  handleOrderShipped(@Payload() data: { orderId: string; trackingNumber: string }) {
    console.log('Order shipped:', data);
    
    // Send shipping notification email
    // Update order tracking
    // Notify customer
  }
}
```

```typescript
// Client: app.controller.ts
import { Controller, Post, Body, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';

@Controller('users')
export class UsersController {
  constructor(
    @Inject('EVENTS_SERVICE') private eventsClient: ClientProxy,
  ) {}

  @Post()
  async createUser(@Body() userData: any) {
    // Create user
    const user = await this.usersService.create(userData);
    
    // ✅ Emit event (doesn't wait for response)
    this.eventsClient.emit('user-created', {
      userId: user.id,
      email: user.email,
      timestamp: new Date(),
    });
    
    // Return immediately (no waiting)
    return user;
  }
}
```

**Event Pattern Types:**

```typescript
import { Controller } from '@nestjs/common';
import { EventPattern, Payload } from '@nestjs/microservices';

@Controller()
export class EventsController {
  // 1. String pattern
  @EventPattern('user-created')
  handleUserCreated(@Payload() data: any) {
    console.log('User created:', data);
  }

  // 2. Dot-notation pattern (namespaced)
  @EventPattern('order.payment.completed')
  handlePaymentCompleted(@Payload() data: any) {
    console.log('Payment completed:', data);
  }

  // 3. Object pattern (complex routing)
  @EventPattern({ event: 'notification', type: 'email' })
  handleEmailNotification(@Payload() data: any) {
    console.log('Email notification:', data);
  }
}
```

**Multiple Listeners for Same Event:**

```typescript
// notifications.controller.ts
@Controller()
export class NotificationsController {
  // Listener 1: Send email
  @EventPattern('user-created')
  async sendWelcomeEmail(@Payload() data: { userId: string; email: string }) {
    console.log('Sending welcome email to:', data.email);
    await this.emailService.sendWelcomeEmail(data.email);
  }
}
```

```typescript
// analytics.controller.ts
@Controller()
export class AnalyticsController {
  // Listener 2: Log to analytics
  @EventPattern('user-created')
  async logUserCreated(@Payload() data: { userId: string; email: string }) {
    console.log('Logging user creation to analytics:', data.userId);
    await this.analyticsService.trackUserCreation(data);
  }
}
```

```typescript
// cache.controller.ts
@Controller()
export class CacheController {
  // Listener 3: Update cache
  @EventPattern('user-created')
  async updateCache(@Payload() data: { userId: string; email: string }) {
    console.log('Updating cache for user:', data.userId);
    await this.cacheService.set(`user:${data.userId}`, data);
  }
}
```

```typescript
// Client emits once, all 3 listeners receive it!
client.emit('user-created', { userId: '123', email: 'user@example.com' });
// ✅ Email sent
// ✅ Analytics logged
// ✅ Cache updated
```

**Real-World E-commerce Example:**

```typescript
// orders.controller.ts
import { Controller } from '@nestjs/common';
import { EventPattern, Payload } from '@nestjs/microservices';

@Controller()
export class OrdersController {
  constructor(
    private ordersService: OrdersService,
    private emailService: EmailService,
    private inventoryService: InventoryService,
    private analyticsService: AnalyticsService,
  ) {}

  // Order placed event
  @EventPattern('order.placed')
  async handleOrderPlaced(@Payload() data: any) {
    console.log('Order placed:', data.orderId);
    
    // Send order confirmation email
    await this.emailService.sendOrderConfirmation({
      email: data.customerEmail,
      orderId: data.orderId,
      items: data.items,
    });
    
    // Update inventory
    await this.inventoryService.reserveItems(data.items);
    
    // Track in analytics
    await this.analyticsService.trackOrder(data);
  }

  // Payment completed event
  @EventPattern('order.payment.completed')
  async handlePaymentCompleted(@Payload() data: any) {
    console.log('Payment completed:', data.orderId);
    
    // Update order status
    await this.ordersService.updateStatus(data.orderId, 'paid');
    
    // Send payment receipt
    await this.emailService.sendPaymentReceipt(data);
    
    // Trigger fulfillment
    // (In real app, would emit another event for warehouse)
  }

  // Order shipped event
  @EventPattern('order.shipped')
  async handleOrderShipped(@Payload() data: any) {
    console.log('Order shipped:', data.orderId);
    
    // Update order status
    await this.ordersService.updateStatus(data.orderId, 'shipped');
    
    // Send shipping notification
    await this.emailService.sendShippingNotification({
      email: data.customerEmail,
      orderId: data.orderId,
      trackingNumber: data.trackingNumber,
    });
  }

  // Order delivered event
  @EventPattern('order.delivered')
  async handleOrderDelivered(@Payload() data: any) {
    console.log('Order delivered:', data.orderId);
    
    // Update order status
    await this.ordersService.updateStatus(data.orderId, 'delivered');
    
    // Request review
    await this.emailService.sendReviewRequest(data);
    
    // Update customer stats
    await this.customersService.incrementOrderCount(data.customerId);
  }

  // Order cancelled event
  @EventPattern('order.cancelled')
  async handleOrderCancelled(@Payload() data: any) {
    console.log('Order cancelled:', data.orderId);
    
    // Update order status
    await this.ordersService.updateStatus(data.orderId, 'cancelled');
    
    // Refund payment
    if (data.paymentId) {
      // Would call payment service
    }
    
    // Restore inventory
    await this.inventoryService.restoreItems(data.items);
    
    // Send cancellation email
    await this.emailService.sendCancellationNotification(data);
  }
}
```

```typescript
// Client: Order creation flow
@Controller('orders')
export class OrdersGatewayController {
  constructor(
    @Inject('ORDERS_SERVICE') private ordersClient: ClientProxy,
  ) {}

  @Post()
  async createOrder(@Body() orderData: any) {
    // 1. Create order (request-response)
    const order = await firstValueFrom(
      this.ordersClient.send('order.create', orderData),
    );
    
    // 2. Emit order.placed event (fire-and-forget)
    this.ordersClient.emit('order.placed', {
      orderId: order.id,
      customerId: orderData.customerId,
      customerEmail: orderData.email,
      items: orderData.items,
      total: order.total,
      timestamp: new Date(),
    });
    
    // Return immediately (event processing happens async)
    return order;
  }

  @Post(':id/ship')
  async shipOrder(@Param('id') id: string, @Body() shipData: any) {
    // Emit shipping event
    this.ordersClient.emit('order.shipped', {
      orderId: id,
      trackingNumber: shipData.trackingNumber,
      carrier: shipData.carrier,
      customerEmail: shipData.email,
      timestamp: new Date(),
    });
    
    return { message: 'Shipping notification sent' };
  }
}
```

**Async Processing:**

```typescript
import { Controller } from '@nestjs/common';
import { EventPattern, Payload } from '@nestjs/microservices';

@Controller()
export class ProcessingController {
  // ✅ Async/await
  @EventPattern('image.uploaded')
  async handleImageUpload(@Payload() data: { imageUrl: string; userId: string }) {
    console.log('Processing image:', data.imageUrl);
    
    // Async image processing
    const thumbnail = await this.imageService.createThumbnail(data.imageUrl);
    const compressed = await this.imageService.compress(data.imageUrl);
    
    // Save results
    await this.imageService.saveProcessedImages({
      original: data.imageUrl,
      thumbnail,
      compressed,
    });
  }

  // ✅ Promise
  @EventPattern('video.uploaded')
  handleVideoUpload(@Payload() data: { videoUrl: string }): Promise<void> {
    return this.videoService.processVideo(data.videoUrl);
  }

  // ✅ Synchronous
  @EventPattern('cache.clear')
  handleCacheClear(@Payload() data: { key: string }) {
    console.log('Clearing cache:', data.key);
    this.cacheService.delete(data.key);
  }
}
```

**Error Handling:**

```typescript
import { Controller } from '@nestjs/common';
import { EventPattern, Payload } from '@nestjs/microservices';

@Controller()
export class EventsController {
  constructor(private logger: Logger) {}

  @EventPattern('risky-event')
  async handleRiskyEvent(@Payload() data: any) {
    try {
      // Process event
      await this.service.processRiskyOperation(data);
      
      this.logger.log('Event processed successfully');
    } catch (error) {
      // ✅ Log error (don't throw - no one is listening!)
      this.logger.error('Failed to process event:', error);
      
      // ✅ Could emit error event
      // this.eventEmitter.emit('event-processing-failed', {
      //   originalEvent: 'risky-event',
      //   data,
      //   error: error.message,
      // });
      
      // Don't throw - event handlers don't return values
    }
  }

  // Dead letter queue pattern
  @EventPattern('process-task')
  async handleTask(@Payload() data: any) {
    const maxRetries = 3;
    let retries = 0;
    
    while (retries < maxRetries) {
      try {
        await this.taskService.process(data);
        return;  // Success
      } catch (error) {
        retries++;
        this.logger.warn(`Retry ${retries}/${maxRetries}`);
        
        if (retries === maxRetries) {
          // Send to dead letter queue
          this.eventsClient.emit('task-failed', {
            task: data,
            error: error.message,
            retries,
          });
        }
        
        // Wait before retry
        await new Promise(resolve => setTimeout(resolve, 1000 * retries));
      }
    }
  }
}
```

**Context Access:**

```typescript
import { Controller } from '@nestjs/common';
import { EventPattern, Payload, Ctx } from '@nestjs/microservices';

@Controller()
export class EventsController {
  // Access event context
  @EventPattern('notification')
  handleNotification(
    @Payload() data: any,
    @Ctx() context: any,
  ) {
    console.log('Event pattern:', context.getPattern());
    console.log('Event data:', data);
    
    // For RabbitMQ: manual acknowledgment
    // const channel = context.getChannelRef();
    // const originalMsg = context.getMessage();
    // channel.ack(originalMsg);
  }
}
```

**Client-Side: Emitting Events:**

```typescript
import { Controller, Post, Body, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';

@Controller()
export class AppController {
  constructor(
    @Inject('EVENTS_SERVICE') private eventsClient: ClientProxy,
  ) {}

  @Post('notify')
  async sendNotification(@Body() data: any) {
    // ✅ Emit event (returns void, doesn't wait)
    this.eventsClient.emit('notification-sent', {
      userId: data.userId,
      message: data.message,
      timestamp: new Date(),
    });
    
    // Response returned immediately
    return { message: 'Notification event emitted' };
  }

  // Emit multiple events
  @Post('user')
  async createUser(@Body() userData: any) {
    const user = await this.usersService.create(userData);
    
    // Emit multiple events
    this.eventsClient.emit('user.created', user);
    this.eventsClient.emit('analytics.track', {
      event: 'user_signup',
      userId: user.id,
    });
    this.eventsClient.emit('email.welcome', {
      email: user.email,
      name: user.name,
    });
    
    return user;
  }
}
```

**Use Cases for @EventPattern():**

```
1. Notifications ✅
   - Send emails, SMS, push notifications
   - User doesn't wait for delivery

2. Analytics & Logging ✅
   - Track events
   - Log activities
   - Non-critical data

3. Cache Updates ✅
   - Invalidate cache
   - Update cached data
   - Background refresh

4. Background Jobs ✅
   - Image processing
   - Video transcoding
   - Report generation

5. Event Sourcing ✅
   - Record state changes
   - Audit trail
   - Event log

6. Fan-out Pattern ✅
   - One event, multiple handlers
   - Parallel processing
   - Decoupled services
```

**Best Practices:**

```typescript
// ✅ GOOD: No return value
@EventPattern('user-created')
handleUserCreated(@Payload() data: any) {
  console.log('Processing event:', data);
  // No return
}

// ✅ GOOD: Async processing
@EventPattern('image-uploaded')
async handleImageUpload(@Payload() data: any) {
  await this.imageService.process(data);
}

// ✅ GOOD: Error handling with logging
@EventPattern('process')
async handleProcess(@Payload() data: any) {
  try {
    await this.service.process(data);
  } catch (error) {
    this.logger.error('Event processing failed:', error);
  }
}

// ✅ GOOD: Use for non-critical operations
@EventPattern('cache-update')
handleCacheUpdate(@Payload() data: any) {
  // If this fails, application continues
  this.cache.set(data.key, data.value);
}

// ❌ BAD: Returning value (ignored!)
@EventPattern('user-created')
handleUserCreated(@Payload() data: any) {
  return { success: true };  // No one receives this!
}

// ❌ BAD: Throwing errors (no error handler)
@EventPattern('process')
handleProcess(@Payload() data: any) {
  throw new Error('Failed');  // Error lost!
}

// ❌ BAD: Using for critical operations
@EventPattern('payment-process')  // ❌ Wrong!
handlePayment(@Payload() data: any) {
  // Should be @MessagePattern() - need response!
}
```

**Interview Tip**: **@EventPattern()** is for **fire-and-forget** event handling. **No response** returned to caller. Used for **asynchronous**, **one-way** communication. Client uses **client.emit(event, data)** - doesn't wait. **Multiple listeners** can handle same event. Use for: **notifications**, **analytics**, **cache updates**, **background jobs**. Different from **@MessagePattern()**: EventPattern **no return**, MessagePattern **returns value**. Handle errors with **try-catch and logging** (don't throw). Supports **async/await**. Good for **non-critical operations** that don't need immediate response.

</details>

### 12. What is the difference between `@MessagePattern()` and `@EventPattern()`?

<details>
<summary>Answer</summary>

**@MessagePattern()** and **@EventPattern()** are both decorators for handling messages in microservices, but they serve **different communication patterns**:

**Quick Comparison:**

| Feature | @MessagePattern() | @EventPattern() |
|---------|-------------------|------------------|
| **Pattern** | Request-Response | Fire-and-Forget |
| **Response** | Returns value | No response |
| **Waiting** | Caller waits | Caller doesn't wait |
| **Communication** | Synchronous | Asynchronous |
| **Use Case** | Queries, Commands | Notifications, Events |
| **Client Method** | `client.send()` | `client.emit()` |
| **Return Value** | Required | Ignored |
| **Error Handling** | RpcException sent to client | Logged locally |
| **Multiple Handlers** | No (one handler) | Yes (many handlers) |
| **Blocking** | Blocks client | Non-blocking |

**1. Request-Response (@MessagePattern):**

```typescript
// Server: math.controller.ts
import { Controller } from '@nestjs/common';
import { MessagePattern, Payload } from '@nestjs/microservices';

@Controller()
export class MathController {
  // ✅ Request-Response: Returns value
  @MessagePattern('add')
  add(@Payload() data: { a: number; b: number }) {
    return {
      result: data.a + data.b,  // ✅ Response sent back
    };
  }
}
```

```typescript
// Client: app.controller.ts
import { Controller, Get, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { firstValueFrom } from 'rxjs';

@Controller()
export class AppController {
  constructor(
    @Inject('MATH_SERVICE') private mathClient: ClientProxy,
  ) {}

  @Get('add')
  async add() {
    // ✅ Send and WAIT for response
    const response = await firstValueFrom(
      this.mathClient.send('add', { a: 5, b: 3 }),
    );
    
    console.log(response);  // { result: 8 }
    return response;
  }
}
```

**2. Fire-and-Forget (@EventPattern):**

```typescript
// Server: notifications.controller.ts
import { Controller } from '@nestjs/common';
import { EventPattern, Payload } from '@nestjs/microservices';

@Controller()
export class NotificationsController {
  // ✅ Fire-and-Forget: No response
  @EventPattern('user-created')
  handleUserCreated(@Payload() data: { userId: string; email: string }) {
    console.log('Sending welcome email to:', data.email);
    // Send email...
    // NO RETURN VALUE
  }
}
```

```typescript
// Client: users.controller.ts
import { Controller, Post, Body, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';

@Controller('users')
export class UsersController {
  constructor(
    @Inject('NOTIFICATIONS_SERVICE') private notificationsClient: ClientProxy,
  ) {}

  @Post()
  async createUser(@Body() userData: any) {
    const user = await this.usersService.create(userData);
    
    // ✅ Emit and DON'T WAIT
    this.notificationsClient.emit('user-created', {
      userId: user.id,
      email: user.email,
    });
    
    // Returns immediately (doesn't wait for email)
    return user;
  }
}
```

**Side-by-Side Comparison:**

```typescript
// ========================================
// @MessagePattern() - Request-Response
// ========================================

// Server
@Controller()
export class UsersController {
  @MessagePattern('get-user')  // ✅ Waits for response
  async getUser(@Payload() data: { id: string }) {
    const user = await this.usersService.findById(data.id);
    return user;  // ✅ MUST return value
  }
}

// Client
const user = await firstValueFrom(
  client.send('get-user', { id: '123' }),  // ✅ Waits for response
);
console.log(user);  // { id: '123', name: 'John', ... }

// ========================================
// @EventPattern() - Fire-and-Forget
// ========================================

// Server
@Controller()
export class NotificationsController {
  @EventPattern('user-updated')  // ✅ Doesn't send response
  handleUserUpdated(@Payload() data: { id: string; changes: any }) {
    console.log('User updated:', data.id);
    // Update cache, send notification, etc.
    // NO RETURN
  }
}

// Client
client.emit('user-updated', { id: '123', changes: { name: 'Jane' } });  // ✅ Returns immediately
console.log('Event emitted, continuing...');
```

**Communication Flow:**

```
@MessagePattern() Flow:
┌────────┐                  ┌──────────┐
│ Client │ ----send----> │ Server   │
│        │                  │          │
│ WAITS  │ <--response-- │ @Message │
│        │                  │ Pattern  │
└────────┘                  └──────────┘
   Blocking                   Returns value

@EventPattern() Flow:
┌────────┐                  ┌──────────┐
│ Client │ ----emit----> │ Server   │
│        │                  │          │
│ GOES   │                  │ @Event   │
│ ON     │                  │ Pattern  │
└────────┘                  └──────────┘
 Non-blocking              No response
```

**Error Handling Differences:**

```typescript
// @MessagePattern() - Errors sent to client
@Controller()
export class UsersController {
  @MessagePattern('get-user')
  async getUser(@Payload() data: { id: string }) {
    const user = await this.usersService.findById(data.id);
    
    if (!user) {
      // ✅ Error sent back to client
      throw new RpcException('User not found');
    }
    
    return user;
  }
}

// Client receives error
try {
  const user = await firstValueFrom(
    client.send('get-user', { id: '999' }),
  );
} catch (error) {
  console.error('Error:', error);  // 'User not found'
}

// ========================================

// @EventPattern() - Errors logged locally
@Controller()
export class NotificationsController {
  @EventPattern('send-email')
  async handleSendEmail(@Payload() data: any) {
    try {
      await this.emailService.send(data.email);
    } catch (error) {
      // ✅ Log error (client doesn't know)
      this.logger.error('Failed to send email:', error);
      // Don't throw - no one is listening
    }
  }
}

// Client doesn't know if email failed
client.emit('send-email', { email: 'user@example.com' });
console.log('Email event emitted (no error feedback)');
```

**Multiple Handlers:**

```typescript
// @MessagePattern() - ONE handler per pattern
@Controller()
export class UsersController {
  @MessagePattern('get-user')  // ✅ Only one handler
  getUser(@Payload() data: { id: string }) {
    return this.usersService.findById(data.id);
  }
}

// ❌ Can't have two handlers for same pattern
// @MessagePattern('get-user')  // ERROR: Duplicate!
// getUser2() { ... }

// ========================================

// @EventPattern() - MULTIPLE handlers allowed
@Controller()
export class NotificationsController {
  @EventPattern('user-created')  // Handler 1
  sendEmail(@Payload() data: any) {
    this.emailService.sendWelcome(data.email);
  }
}

@Controller()
export class AnalyticsController {
  @EventPattern('user-created')  // Handler 2 (same event!)
  trackSignup(@Payload() data: any) {
    this.analyticsService.track('signup', data);
  }
}

@Controller()
export class CacheController {
  @EventPattern('user-created')  // Handler 3 (same event!)
  updateCache(@Payload() data: any) {
    this.cache.set(`user:${data.id}`, data);
  }
}

// All 3 handlers execute!
client.emit('user-created', { id: '123', email: 'user@example.com' });
```

**Use Cases Comparison:**

```typescript
// ✅ Use @MessagePattern() when:

// 1. Need response data
@MessagePattern('calculate-total')
calculateTotal(@Payload() items: any[]) {
  return { total: items.reduce((sum, item) => sum + item.price, 0) };
}

// 2. Querying data
@MessagePattern('get-user')
getUser(@Payload() data: { id: string }) {
  return this.usersService.findById(data.id);
}

// 3. Commands requiring confirmation
@MessagePattern('create-order')
createOrder(@Payload() data: any) {
  const order = this.ordersService.create(data);
  return { orderId: order.id, status: 'created' };
}

// 4. Critical operations
@MessagePattern('process-payment')
processPayment(@Payload() data: any) {
  const result = this.paymentService.charge(data);
  return { success: true, transactionId: result.id };
}

// ========================================

// ✅ Use @EventPattern() when:

// 1. Notifications (don't need confirmation)
@EventPattern('order-shipped')
notifyCustomer(@Payload() data: any) {
  this.emailService.sendShippingNotification(data);
}

// 2. Logging / Analytics (non-critical)
@EventPattern('user-login')
trackLogin(@Payload() data: any) {
  this.analyticsService.track('login', data);
}

// 3. Cache updates (background)
@EventPattern('user-updated')
updateCache(@Payload() data: any) {
  this.cache.set(`user:${data.id}`, data);
}

// 4. Background jobs (async processing)
@EventPattern('image-uploaded')
processImage(@Payload() data: any) {
  this.imageService.createThumbnail(data.imageUrl);
}
```

**Performance Characteristics:**

```typescript
// @MessagePattern() - Slower (waits)
const start = Date.now();
const result = await firstValueFrom(
  client.send('heavy-calculation', data),  // Waits 5 seconds
);
const duration = Date.now() - start;  // ~5000ms
console.log('Duration:', duration);

// @EventPattern() - Faster (doesn't wait)
const start = Date.now();
client.emit('heavy-calculation', data);  // Returns immediately
const duration = Date.now() - start;  // ~1ms
console.log('Duration:', duration);  // Calculation happens in background
```

**When to Use Which:**

```
@MessagePattern() - Use when:
✅ Need response data
✅ Operation is critical
✅ Need error feedback
✅ Querying information
✅ Processing must complete before continuing
✅ Single handler per operation

Examples:
- Get user details
- Create order
- Process payment
- Validate data
- Calculate results

@EventPattern() - Use when:
✅ Don't need response
✅ Operation is non-critical
✅ Can process asynchronously
✅ Broadcasting to multiple services
✅ Background jobs
✅ Multiple handlers allowed

Examples:
- Send notifications
- Log analytics
- Update cache
- Process images
- Trigger workflows
- Audit logging
```

**Combined Example:**

```typescript
// orders.controller.ts
import { Controller } from '@nestjs/common';
import { MessagePattern, EventPattern, Payload } from '@nestjs/microservices';

@Controller()
export class OrdersController {
  constructor(
    private ordersService: OrdersService,
    @Inject('EVENTS_CLIENT') private eventsClient: ClientProxy,
  ) {}

  // ✅ @MessagePattern() - Need response
  @MessagePattern('order.create')
  async createOrder(@Payload() data: any) {
    // Create order (critical - need confirmation)
    const order = await this.ordersService.create(data);
    
    // Emit event for non-critical tasks
    this.eventsClient.emit('order.created', order);
    
    // Return order to client
    return order;  // Client waits for this
  }

  // ✅ @EventPattern() - No response needed
  @EventPattern('order.created')
  async handleOrderCreated(@Payload() data: any) {
    // Non-critical tasks:
    // - Send email
    // - Update analytics
    // - Update cache
    console.log('Order created event:', data.orderId);
    await this.emailService.sendConfirmation(data.customerEmail);
  }
}
```

**Best Practices:**

```typescript
// ✅ GOOD: Use appropriate pattern
@MessagePattern('get-order')  // Need response
getOrder(@Payload() data: { id: string }) {
  return this.ordersService.findById(data.id);
}

@EventPattern('order-shipped')  // No response
handleShipped(@Payload() data: any) {
  this.emailService.sendShippingNotification(data);
}

// ✅ GOOD: Combine both patterns
@MessagePattern('create-order')
async createOrder(@Payload() data: any) {
  const order = await this.ordersService.create(data);
  this.eventsClient.emit('order.created', order);  // Fire event
  return order;  // Return response
}

// ❌ BAD: Using @EventPattern() when response needed
@EventPattern('get-order')  // ❌ Wrong!
getOrder(@Payload() data: { id: string }) {
  return this.ordersService.findById(data.id);  // Return ignored!
}

// ❌ BAD: Using @MessagePattern() for fire-and-forget
@MessagePattern('send-email')  // ❌ Overkill
sendEmail(@Payload() data: any) {
  this.emailService.send(data);
  return { sent: true };  // Client doesn't need this
}
```

**Interview Tip**: **@MessagePattern()** is **request-response** (client waits for response, returns value, use **client.send()**). **@EventPattern()** is **fire-and-forget** (no response, async, use **client.emit()**). Key differences: MessagePattern **blocks caller**, EventPattern **non-blocking**. MessagePattern: **one handler**, EventPattern: **multiple handlers**. Use **@MessagePattern()** for: queries, critical operations, need response. Use **@EventPattern()** for: notifications, analytics, background jobs, no response needed. Errors in MessagePattern: **sent to client**. Errors in EventPattern: **logged locally**.

</details>

## Communication

### 13. How do services communicate in microservices?

<details>
<summary>Answer</summary>

Services in microservices communicate through **message-based protocols** over a **network**. NestJS supports two main communication patterns: **Request-Response** (synchronous) and **Event-Based** (asynchronous).

**Communication Patterns:**

```
1. Request-Response (Synchronous) ✅
   - Client sends message and waits for response
   - Uses @MessagePattern() decorator
   - client.send() method
   - Like HTTP request-response

2. Event-Based (Asynchronous) ✅
   - Client emits event and doesn't wait
   - Uses @EventPattern() decorator
   - client.emit() method
   - Fire-and-forget pattern
```

**Communication Architecture:**

```
┌─────────────────────────────────────────────────┐
│                 Client Service                   │
│  ┌──────────────────────────────────────┐  │
│  │  ClientProxy (TCP/Redis/RMQ...)   │  │
│  └───────────────┬───────────────────────┘  │
│                  │                            │
│     client.send('pattern', data) │            │
│     client.emit('event', data)   │            │
└──────────────────┴──────────────────────────────┘
                   │
        Network (TCP/Redis/RabbitMQ/Kafka/gRPC)
                   │
┌──────────────────┬──────────────────────────────┐
│                  │                            │
│  ┌───────────────▼──────────────────────┐  │
│  │  Controller                      │  │
│  │  @MessagePattern('pattern')      │  │
│  │  @EventPattern('event')          │  │
│  └──────────────────────────────────────┘  │
│                Server Service                   │
└─────────────────────────────────────────────────┘
```

**1. Request-Response Communication:**

```typescript
// ===== Server Service =====
// users-service/users.controller.ts
import { Controller } from '@nestjs/common';
import { MessagePattern, Payload } from '@nestjs/microservices';

@Controller()
export class UsersController {
  constructor(private usersService: UsersService) {}

  // ✅ Request-Response: Returns data
  @MessagePattern('get-user')
  async getUser(@Payload() data: { id: string }) {
    const user = await this.usersService.findById(data.id);
    return user;  // Response sent back
  }

  @MessagePattern('create-user')
  async createUser(@Payload() data: { name: string; email: string }) {
    const user = await this.usersService.create(data);
    return user;
  }
}
```

```typescript
// ===== Client Service =====
// orders-service/orders.service.ts
import { Injectable, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { firstValueFrom } from 'rxjs';

@Injectable()
export class OrdersService {
  constructor(
    @Inject('USERS_SERVICE') private usersClient: ClientProxy,
  ) {}

  async createOrder(orderData: any) {
    // ✅ Call users service and wait for response
    const user = await firstValueFrom(
      this.usersClient.send('get-user', { id: orderData.userId }),
    );

    if (!user) {
      throw new Error('User not found');
    }

    // Create order with user data
    const order = await this.ordersRepository.create({
      userId: user.id,
      userEmail: user.email,
      items: orderData.items,
    });

    return order;
  }
}
```

**2. Event-Based Communication:**

```typescript
// ===== Server Service =====
// notifications-service/notifications.controller.ts
import { Controller } from '@nestjs/common';
import { EventPattern, Payload } from '@nestjs/microservices';

@Controller()
export class NotificationsController {
  constructor(private emailService: EmailService) {}

  // ✅ Event handler: No response
  @EventPattern('order.created')
  async handleOrderCreated(@Payload() data: any) {
    console.log('Order created event:', data);
    
    // Send order confirmation email
    await this.emailService.sendOrderConfirmation({
      email: data.customerEmail,
      orderId: data.orderId,
    });
  }

  @EventPattern('order.shipped')
  async handleOrderShipped(@Payload() data: any) {
    // Send shipping notification
    await this.emailService.sendShippingNotification(data);
  }
}
```

```typescript
// ===== Client Service =====
// orders-service/orders.service.ts
import { Injectable, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';

@Injectable()
export class OrdersService {
  constructor(
    @Inject('NOTIFICATIONS_SERVICE') private notificationsClient: ClientProxy,
  ) {}

  async createOrder(orderData: any) {
    // Create order
    const order = await this.ordersRepository.create(orderData);

    // ✅ Emit event (doesn't wait for response)
    this.notificationsClient.emit('order.created', {
      orderId: order.id,
      customerEmail: orderData.email,
      items: orderData.items,
    });

    // Return immediately
    return order;
  }
}
```

**Transport Layers (Communication Protocols):**

```typescript
// 1. TCP (Default) - Direct connection
ClientsModule.register([
  {
    name: 'USERS_SERVICE',
    transport: Transport.TCP,
    options: {
      host: 'users-service',
      port: 3001,
    },
  },
])

// 2. Redis - Pub/Sub pattern
ClientsModule.register([
  {
    name: 'CACHE_SERVICE',
    transport: Transport.REDIS,
    options: {
      host: 'redis',
      port: 6379,
    },
  },
])

// 3. RabbitMQ - Message queue
ClientsModule.register([
  {
    name: 'ORDERS_SERVICE',
    transport: Transport.RMQ,
    options: {
      urls: ['amqp://rabbitmq:5672'],
      queue: 'orders_queue',
    },
  },
])

// 4. Kafka - Event streaming
ClientsModule.register([
  {
    name: 'EVENTS_SERVICE',
    transport: Transport.KAFKA,
    options: {
      client: {
        brokers: ['kafka:9092'],
      },
      consumer: {
        groupId: 'orders-consumer',
      },
    },
  },
])

// 5. gRPC - High performance RPC
ClientsModule.register([
  {
    name: 'PAYMENTS_SERVICE',
    transport: Transport.GRPC,
    options: {
      package: 'payments',
      protoPath: join(__dirname, './payments.proto'),
      url: 'payments-service:5000',
    },
  },
])
```

**Complete E-commerce Example:**

```typescript
// ===== API Gateway (HTTP Entry Point) =====
// gateway/orders.controller.ts
import { Controller, Post, Body, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { firstValueFrom } from 'rxjs';

@Controller('orders')
export class OrdersController {
  constructor(
    @Inject('USERS_SERVICE') private usersClient: ClientProxy,
    @Inject('PRODUCTS_SERVICE') private productsClient: ClientProxy,
    @Inject('ORDERS_SERVICE') private ordersClient: ClientProxy,
    @Inject('PAYMENTS_SERVICE') private paymentsClient: ClientProxy,
  ) {}

  @Post()
  async createOrder(@Body() orderData: any) {
    // Step 1: Validate user (request-response)
    const user = await firstValueFrom(
      this.usersClient.send('validate-user', { userId: orderData.userId }),
    );

    if (!user.valid) {
      throw new BadRequestException('Invalid user');
    }

    // Step 2: Check product availability (request-response)
    const availability = await firstValueFrom(
      this.productsClient.send('check-availability', {
        items: orderData.items,
      }),
    );

    if (!availability.available) {
      throw new BadRequestException('Some items are out of stock');
    }

    // Step 3: Create order (request-response)
    const order = await firstValueFrom(
      this.ordersClient.send('create-order', {
        userId: orderData.userId,
        items: orderData.items,
        total: availability.total,
      }),
    );

    // Step 4: Process payment (request-response)
    const payment = await firstValueFrom(
      this.paymentsClient.send('process-payment', {
        orderId: order.id,
        amount: order.total,
        method: orderData.paymentMethod,
      }),
    );

    // Step 5: Emit events (fire-and-forget)
    // These happen async in background
    this.ordersClient.emit('order.created', order);
    this.ordersClient.emit('payment.completed', payment);
    this.ordersClient.emit('inventory.reserved', {
      items: orderData.items,
    });

    return {
      orderId: order.id,
      status: 'created',
      paymentId: payment.id,
    };
  }
}
```

```typescript
// ===== Orders Microservice =====
// orders-service/orders.controller.ts
import { Controller } from '@nestjs/common';
import { MessagePattern, EventPattern, Payload } from '@nestjs/microservices';

@Controller()
export class OrdersController {
  constructor(private ordersService: OrdersService) {}

  // Handle request-response
  @MessagePattern('create-order')
  async createOrder(@Payload() data: any) {
    const order = await this.ordersService.create(data);
    return order;
  }

  // Handle events
  @EventPattern('payment.completed')
  async handlePaymentCompleted(@Payload() data: any) {
    await this.ordersService.markAsPaid(data.orderId);
  }
}
```

**Service Discovery Pattern:**

```typescript
// app.module.ts - Register all microservices
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';

@Module({
  imports: [
    ClientsModule.register([
      // Users microservice
      {
        name: 'USERS_SERVICE',
        transport: Transport.TCP,
        options: {
          host: process.env.USERS_SERVICE_HOST || 'localhost',
          port: parseInt(process.env.USERS_SERVICE_PORT) || 3001,
        },
      },
      
      // Products microservice
      {
        name: 'PRODUCTS_SERVICE',
        transport: Transport.TCP,
        options: {
          host: process.env.PRODUCTS_SERVICE_HOST || 'localhost',
          port: parseInt(process.env.PRODUCTS_SERVICE_PORT) || 3002,
        },
      },
      
      // Orders microservice
      {
        name: 'ORDERS_SERVICE',
        transport: Transport.RMQ,
        options: {
          urls: [process.env.RABBITMQ_URL || 'amqp://localhost:5672'],
          queue: 'orders_queue',
        },
      },
      
      // Payments microservice (gRPC)
      {
        name: 'PAYMENTS_SERVICE',
        transport: Transport.GRPC,
        options: {
          package: 'payments',
          protoPath: join(__dirname, './protos/payments.proto'),
          url: process.env.PAYMENTS_SERVICE_URL || 'localhost:5000',
        },
      },
      
      // Notifications microservice (Redis pub/sub)
      {
        name: 'NOTIFICATIONS_SERVICE',
        transport: Transport.REDIS,
        options: {
          host: process.env.REDIS_HOST || 'localhost',
          port: parseInt(process.env.REDIS_PORT) || 6379,
        },
      },
    ]),
  ],
})
export class AppModule {}
```

**Communication Patterns Comparison:**

| Aspect | Request-Response | Event-Based |
|--------|------------------|-------------|
| **Method** | client.send() | client.emit() |
| **Decorator** | @MessagePattern() | @EventPattern() |
| **Response** | Yes, waits | No response |
| **Blocking** | Blocks caller | Non-blocking |
| **Use Case** | Queries, Commands | Notifications, Logging |
| **Example** | Get user data | Send email |
| **Error Handling** | Errors sent to client | Errors logged locally |
| **Performance** | Slower (waits) | Faster (no wait) |

**Communication Flow:**

```
Request-Response Flow:
1. Client calls client.send('pattern', data)
2. Message sent over network (TCP/Redis/RMQ/etc.)
3. Server receives via @MessagePattern('pattern')
4. Server processes and returns result
5. Result sent back over network
6. Client receives response
7. Client continues execution

Event-Based Flow:
1. Client calls client.emit('event', data)
2. Message sent over network
3. Server(s) receive via @EventPattern('event')
4. Server processes event (no response)
5. Client continues immediately (no waiting)
```

**Best Practices:**

```typescript
// ✅ GOOD: Use request-response for critical operations
const user = await firstValueFrom(
  this.usersClient.send('get-user', { id }),
);

// ✅ GOOD: Use events for notifications
this.notificationsClient.emit('order.created', order);

// ✅ GOOD: Handle errors in request-response
try {
  const result = await firstValueFrom(
    this.client.send('pattern', data),
  );
} catch (error) {
  console.error('Service call failed:', error);
}

// ✅ GOOD: Combine both patterns
const order = await firstValueFrom(
  this.ordersClient.send('create-order', data),  // Wait for order
);
this.notificationsClient.emit('order.created', order);  // Fire event

// ✅ GOOD: Use appropriate transport for use case
// TCP: Simple, fast, point-to-point
// RabbitMQ: Reliable, queuing, multiple consumers
// Kafka: Event streaming, high throughput
// Redis: Fast, pub/sub
// gRPC: High performance, type safety

// ❌ BAD: Using events when response needed
this.client.emit('get-user', { id });  // ❌ No response!

// ❌ BAD: Not handling errors
const result = await firstValueFrom(
  this.client.send('pattern', data),  // Can throw!
);

// ❌ BAD: Using request-response for fire-and-forget
const result = await firstValueFrom(
  this.client.send('log-event', data),  // Overkill!
);
```

**Interview Tip**: Services communicate via **network protocols** (TCP, Redis, RabbitMQ, Kafka, gRPC). Two patterns: **Request-Response** (client.send(), @MessagePattern(), waits for response) and **Event-Based** (client.emit(), @EventPattern(), fire-and-forget). Use **ClientProxy** to send messages, register with **ClientsModule**. Request-Response for: **queries**, **critical operations**, **need data back**. Event-Based for: **notifications**, **analytics**, **background jobs**. Can **combine both**: create order (request-response) + emit event (notification). Choose **transport** based on needs: **TCP** (simple), **RabbitMQ** (reliable queuing), **Kafka** (event streaming), **gRPC** (performance).

</details>

### 14. What is request-response pattern?

<details>
<summary>Answer</summary>

**Request-Response pattern** is a **synchronous communication** pattern where the client sends a message and **waits for a response** from the server. Similar to HTTP request-response but works over microservice transports (TCP, Redis, RabbitMQ, etc.).

**Key Characteristics:**

```
✅ Synchronous (client waits)
✅ Bidirectional (request + response)
✅ Returns data to caller
✅ Blocking operation
✅ Error feedback to client
✅ Uses @MessagePattern() decorator
✅ Client uses client.send() method
✅ Observable-based (RxJS)
```

**How It Works:**

```
┌──────────┐                    ┌──────────┐
│  Client  │ ---1. Request---> │  Server  │
│          │                    │          │
│  WAITS   │                    │ Process  │
│          │                    │          │
│          │ <--2. Response--- │          │
│ Continue │                    │          │
└──────────┘                    └──────────┘

Timeline:
t0: Client sends request
t1: Client blocks and waits
t2: Server receives request
t3: Server processes
t4: Server sends response
t5: Client receives response
t6: Client continues execution
```

**Basic Implementation:**

```typescript
// ===== Server Side =====
// users-service/users.controller.ts
import { Controller } from '@nestjs/common';
import { MessagePattern, Payload } from '@nestjs/microservices';

@Controller()
export class UsersController {
  constructor(private usersService: UsersService) {}

  // ✅ Request-Response handler
  @MessagePattern('get-user')  // Pattern identifier
  async getUser(@Payload() data: { id: string }) {
    console.log('Received request for user:', data.id);
    
    // Process request
    const user = await this.usersService.findById(data.id);
    
    if (!user) {
      throw new RpcException('User not found');
    }
    
    // Return response
    return {
      id: user.id,
      name: user.name,
      email: user.email,
      createdAt: user.createdAt,
    };
  }
}
```

```typescript
// ===== Client Side =====
// app.controller.ts
import { Controller, Get, Param, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { firstValueFrom } from 'rxjs';

@Controller('users')
export class UsersController {
  constructor(
    @Inject('USERS_SERVICE') private usersClient: ClientProxy,
  ) {}

  @Get(':id')
  async getUser(@Param('id') id: string) {
    console.log('Sending request for user:', id);
    
    // ✅ Send request and WAIT for response
    const user = await firstValueFrom(
      this.usersClient.send('get-user', { id }),
    );
    
    console.log('Received response:', user);
    return user;
  }
}
```

**Observable-Based Communication:**

```typescript
import { timeout, retry, catchError } from 'rxjs/operators';
import { throwError } from 'rxjs';

@Controller()
export class AppController {
  constructor(
    @Inject('USERS_SERVICE') private usersClient: ClientProxy,
  ) {}

  @Get('user/:id')
  async getUser(@Param('id') id: string) {
    // Convert Observable to Promise
    const user = await firstValueFrom(
      this.usersClient.send('get-user', { id }).pipe(
        timeout(5000),  // 5 second timeout
        retry(3),       // Retry 3 times on failure
        catchError(error => {
          console.error('Service error:', error);
          return throwError(() => new NotFoundException(error.message));
        }),
      ),
    );
    
    return user;
  }
}
```

**Parallel Requests (Performance):**

```typescript
import { forkJoin } from 'rxjs';

@Controller()
export class AppController {
  constructor(
    @Inject('USERS_SERVICE') private usersClient: ClientProxy,
    @Inject('PRODUCTS_SERVICE') private productsClient: ClientProxy,
    @Inject('ORDERS_SERVICE') private ordersClient: ClientProxy,
  ) {}

  @Get('dashboard/:userId')
  async getDashboard(@Param('userId') userId: string) {
    // ✅ Execute requests in parallel
    const [user, products, orders] = await firstValueFrom(
      forkJoin([
        this.usersClient.send('get-user', { id: userId }),
        this.productsClient.send('get-recommended', { userId }),
        this.ordersClient.send('get-orders', { userId }),
      ]),
    );

    return {
      user,
      recommendations: products,
      recentOrders: orders,
    };
  }
}
```

**Best Practices:**

```typescript
// ✅ GOOD: Handle errors
try {
  const result = await firstValueFrom(
    this.client.send('pattern', data),
  );
} catch (error) {
  throw new ServiceUnavailableException();
}

// ✅ GOOD: Set timeouts
this.client.send('pattern', data).pipe(timeout(5000))

// ✅ GOOD: Use parallel requests
const [r1, r2] = await firstValueFrom(
  forkJoin([client1.send(...), client2.send(...)]),
);

// ❌ BAD: No error handling
const result = await firstValueFrom(
  this.client.send('pattern', data),  // Can throw!
);
```

**Interview Tip**: **Request-Response** is **synchronous** communication where client **sends request** and **waits for response**. Uses **@MessagePattern()** on server, **client.send()** on client. Returns **Observable** (convert with **firstValueFrom()**). **Blocks caller** until response. Use for: **queries**, **critical operations**, **operations needing confirmation**. Server throws **RpcException** for errors. Can use **timeout()**, **retry()**, **forkJoin()** for optimization.

</details>

### 15. What is event-based pattern?

<details>
<summary>Answer</summary>

**Event-Based pattern** is an **asynchronous communication** pattern where the client emits an event and **doesn't wait for a response**. Also known as **fire-and-forget** or **pub/sub** pattern.

**Key Characteristics:**

```
✅ Asynchronous (client doesn't wait)
✅ One-way communication
✅ No response expected
✅ Non-blocking operation
✅ Multiple handlers allowed
✅ Uses @EventPattern() decorator
✅ Client uses client.emit() method
✅ Fire-and-forget
```

**How It Works:**

```
┌──────────┐                    ┌──────────┐
│  Client  │ ---1. Emit Event-> │  Server  │
│          │                    │  1       │
│ Continue │                    │ Process  │
│ Immediately                   │          │
└──────────┘                    └──────────┘
                               
                               ┌──────────┐
                               │  Server  │
                               │  2       │
                               │ Process  │
                               │          │
                               └──────────┘

Timeline:
t0: Client emits event
t1: Client continues immediately (no waiting)
t2: Server 1 receives event
t3: Server 2 receives event (parallel)
t4: Servers process independently
```

**Basic Implementation:**

```typescript
// ===== Server Side =====
// notifications-service/notifications.controller.ts
import { Controller } from '@nestjs/common';
import { EventPattern, Payload } from '@nestjs/microservices';

@Controller()
export class NotificationsController {
  constructor(private emailService: EmailService) {}

  // ✅ Event handler (no response)
  @EventPattern('user.created')
  async handleUserCreated(@Payload() data: { userId: string; email: string }) {
    console.log('User created event received:', data);
    
    // Send welcome email
    await this.emailService.sendWelcomeEmail({
      to: data.email,
      userId: data.userId,
    });
    
    // NO RETURN (ignored if present)
  }

  @EventPattern('order.placed')
  async handleOrderPlaced(@Payload() data: any) {
    console.log('Order placed:', data.orderId);
    
    // Send order confirmation
    await this.emailService.sendOrderConfirmation(data);
  }
}
```

```typescript
// ===== Client Side =====
// users-service/users.service.ts
import { Injectable, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';

@Injectable()
export class UsersService {
  constructor(
    @Inject('NOTIFICATIONS_SERVICE') private notificationsClient: ClientProxy,
  ) {}

  async createUser(userData: any) {
    // Create user in database
    const user = await this.usersRepository.create(userData);

    // ✅ Emit event (doesn't wait for response)
    this.notificationsClient.emit('user.created', {
      userId: user.id,
      email: user.email,
      name: user.name,
      timestamp: new Date(),
    });

    // Return immediately (email sent in background)
    return user;
  }
}
```

**Multiple Event Handlers:**

```typescript
// Handler 1: Send Email
// notifications-service/email.controller.ts
@Controller()
export class EmailController {
  @EventPattern('user.created')
  async sendEmail(@Payload() data: any) {
    console.log('Sending welcome email...');
    await this.emailService.send(data.email);
  }
}

// Handler 2: Track Analytics
// analytics-service/analytics.controller.ts
@Controller()
export class AnalyticsController {
  @EventPattern('user.created')
  async trackSignup(@Payload() data: any) {
    console.log('Tracking signup...');
    await this.analyticsService.track('user_signup', data);
  }
}

// Handler 3: Update Cache
// cache-service/cache.controller.ts
@Controller()
export class CacheController {
  @EventPattern('user.created')
  async updateCache(@Payload() data: any) {
    console.log('Updating cache...');
    await this.cacheService.set(`user:${data.userId}`, data);
  }
}

// One emit, all 3 handlers execute in parallel!
client.emit('user.created', { userId: '123', email: 'user@example.com' });
```

**E-commerce Order Flow:**

```typescript
// orders-service/orders.service.ts
import { Injectable, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';

@Injectable()
export class OrdersService {
  constructor(
    @Inject('EVENTS_BUS') private eventsBus: ClientProxy,
  ) {}

  async createOrder(orderData: any) {
    // 1. Create order in database
    const order = await this.ordersRepository.create({
      userId: orderData.userId,
      items: orderData.items,
      total: orderData.total,
      status: 'pending',
    });

    // 2. Emit event (triggers multiple handlers)
    this.eventsBus.emit('order.created', {
      orderId: order.id,
      userId: order.userId,
      customerEmail: orderData.email,
      items: order.items,
      total: order.total,
      timestamp: new Date(),
    });

    // Return immediately
    return order;
  }

  async shipOrder(orderId: string) {
    // Update order status
    await this.ordersRepository.update(orderId, { status: 'shipped' });

    // Emit shipping event
    this.eventsBus.emit('order.shipped', {
      orderId,
      shippedAt: new Date(),
    });
  }
}
```

```typescript
// Event Handlers in Different Services

// 1. Notifications Service
@Controller()
export class OrderNotificationsController {
  @EventPattern('order.created')
  async sendOrderConfirmation(@Payload() data: any) {
    await this.emailService.sendOrderConfirmation({
      email: data.customerEmail,
      orderId: data.orderId,
      items: data.items,
    });
  }

  @EventPattern('order.shipped')
  async sendShippingNotification(@Payload() data: any) {
    await this.emailService.sendShippingNotification(data);
  }
}

// 2. Inventory Service
@Controller()
export class InventoryController {
  @EventPattern('order.created')
  async reserveItems(@Payload() data: any) {
    await this.inventoryService.reserve(data.items);
  }
}

// 3. Analytics Service
@Controller()
export class OrderAnalyticsController {
  @EventPattern('order.created')
  async trackOrder(@Payload() data: any) {
    await this.analyticsService.trackOrderCreated(data);
  }
}

// 4. Loyalty Service
@Controller()
export class LoyaltyController {
  @EventPattern('order.created')
  async addPoints(@Payload() data: any) {
    await this.loyaltyService.addPoints(data.userId, data.total);
  }
}
```

**Error Handling:**

```typescript
import { Controller, Logger } from '@nestjs/common';
import { EventPattern, Payload } from '@nestjs/microservices';

@Controller()
export class EventsController {
  private readonly logger = new Logger(EventsController.name);

  @EventPattern('process.task')
  async handleTask(@Payload() data: any) {
    try {
      // Process event
      await this.taskService.process(data);
      this.logger.log('Task processed successfully');
    } catch (error) {
      // ✅ Log error (don't throw - no one is listening)
      this.logger.error('Failed to process task:', error);
      
      // ✅ Optional: Emit error event for monitoring
      this.eventsClient.emit('task.failed', {
        originalEvent: 'process.task',
        data,
        error: error.message,
        timestamp: new Date(),
      });
    }
  }

  // Dead Letter Queue pattern
  @EventPattern('critical.task')
  async handleCriticalTask(@Payload() data: any) {
    const maxRetries = 3;
    
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
      try {
        await this.taskService.processCritical(data);
        return;  // Success
      } catch (error) {
        this.logger.warn(`Attempt ${attempt}/${maxRetries} failed`);
        
        if (attempt === maxRetries) {
          // Send to dead letter queue
          this.eventsClient.emit('task.dead_letter', {
            task: data,
            error: error.message,
            attempts: maxRetries,
          });
        } else {
          // Wait before retry
          await new Promise(resolve => 
            setTimeout(resolve, 1000 * attempt),
          );
        }
      }
    }
  }
}
```

**Event Sourcing Pattern:**

```typescript
// user-service/user.service.ts
@Injectable()
export class UserService {
  constructor(
    @Inject('EVENTS_BUS') private eventsBus: ClientProxy,
  ) {}

  async updateUser(userId: string, updates: any) {
    // Get current user
    const user = await this.usersRepository.findById(userId);
    
    // Apply updates
    const updatedUser = await this.usersRepository.update(userId, updates);

    // Emit events for each change
    if (updates.email && updates.email !== user.email) {
      this.eventsBus.emit('user.email.changed', {
        userId,
        oldEmail: user.email,
        newEmail: updates.email,
        timestamp: new Date(),
      });
    }

    if (updates.status && updates.status !== user.status) {
      this.eventsBus.emit('user.status.changed', {
        userId,
        oldStatus: user.status,
        newStatus: updates.status,
        timestamp: new Date(),
      });
    }

    return updatedUser;
  }
}
```

**Use Cases:**

```
1. Notifications ✅
   - Email notifications
   - SMS alerts  
   - Push notifications
   - User doesn't wait

2. Analytics & Logging ✅
   - Track user behavior
   - Log events
   - Audit trail
   - Non-critical data

3. Background Jobs ✅
   - Image processing
   - Video transcoding
   - Report generation
   - Async operations

4. Cache Updates ✅
   - Invalidate cache
   - Update cached data
   - Background refresh

5. Fan-Out Pattern ✅
   - Broadcast to multiple services
   - Parallel processing
   - Decoupled services

6. Event Sourcing ✅
   - Record state changes
   - Audit log
   - Event replay
```

**Comparison: Request-Response vs Event-Based:**

| Aspect | Request-Response | Event-Based |
|--------|------------------|-------------|
| **Method** | client.send() | client.emit() |
| **Decorator** | @MessagePattern() | @EventPattern() |
| **Wait** | Yes | No |
| **Response** | Returns data | No response |
| **Blocking** | Blocks | Non-blocking |
| **Handlers** | One | Multiple |
| **Error Feedback** | To client | Logged locally |
| **Use Case** | Get data | Notifications |
| **Performance** | Slower | Faster |

**Best Practices:**

```typescript
// ✅ GOOD: Use for non-critical operations
client.emit('order.created', order);

// ✅ GOOD: Handle errors with logging
@EventPattern('process')
async handle(@Payload() data: any) {
  try {
    await this.service.process(data);
  } catch (error) {
    this.logger.error('Processing failed:', error);
  }
}

// ✅ GOOD: Multiple handlers for same event
@EventPattern('user.created')
sendEmail() { ... }

@EventPattern('user.created')
trackAnalytics() { ... }

// ✅ GOOD: Use for fan-out pattern
client.emit('notification', data);  // All subscribers notified

// ❌ BAD: Using for critical operations needing response
client.emit('process-payment', data);  // ❌ No confirmation!

// ❌ BAD: Throwing errors (no one catches them)
@EventPattern('process')
handle(@Payload() data: any) {
  throw new Error('Failed');  // ❌ Error lost!
}

// ❌ BAD: Expecting return value
const result = client.emit('get-user', { id });  // ❌ Returns void!
```

**Interview Tip**: **Event-Based pattern** is **asynchronous fire-and-forget** communication. Client uses **client.emit()** to send event, **doesn't wait** for response. Server uses **@EventPattern()** to handle. **Multiple handlers** can listen to same event. **Non-blocking** - client continues immediately. Use for: **notifications**, **analytics**, **background jobs**, **cache updates**, **fan-out**. Errors **logged locally** (not sent to client). Different from **request-response**: event-based **no response**, request-response **returns data**. Good for **non-critical operations** that don't need immediate feedback.

</details>

### 16. What is `ClientProxy` and how do you use it?

<details>
<summary>Answer</summary>

**ClientProxy** is an **abstraction** that enables communication with microservices. It provides methods to **send messages** (request-response) and **emit events** (fire-and-forget) to remote services.

**Key Methods:**

```typescript
interface ClientProxy {
  // Request-response (waits for response)
  send<TResult>(pattern: any, data: any): Observable<TResult>;
  
  // Fire-and-forget (no response)
  emit<TResult>(pattern: any, data: any): Observable<TResult>;
  
  // Lifecycle methods
  connect(): Promise<any>;
  close(): void;
}
```

**Registration:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';\nimport { ClientsModule, Transport } from '@nestjs/microservices';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'USERS_SERVICE',  // ✅ Injection token
        transport: Transport.TCP,
        options: { host: 'localhost', port: 3001 },
      },
    ]),
  ],
})
export class AppModule {}
```

**Usage:**

```typescript
@Controller()
export class AppController {
  constructor(
    @Inject('USERS_SERVICE') private usersClient: ClientProxy,
  ) {}

  async getUser(id: string) {
    // ✅ send() - request-response
    const user = await firstValueFrom(
      this.usersClient.send('get-user', { id }),
    );
    
    // ✅ emit() - fire-and-forget
    this.usersClient.emit('user.viewed', { id });
    
    return user;
  }
}
```

**Interview Tip**: **ClientProxy** is abstraction for microservice communication. Use **send()** for request-response (waits), **emit()** for fire-and-forget (no wait). Register with **ClientsModule.register()**, inject with **@Inject('TOKEN')**. Returns **Observable** - convert with **firstValueFrom()**. Has **connect()**/**close()** for lifecycle.

</details>

### 17. How do you inject `ClientProxy` using `@Inject()`?

<details>
<summary>Answer</summary>

Inject **ClientProxy** using **@Inject()** decorator with the **token name** defined in `ClientsModule.register()`.

**Basic Injection:**

```typescript
// 1. Register in module
@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'USERS_SERVICE',  // Token
        transport: Transport.TCP,
        options: { host: 'localhost', port: 3001 },
      },
    ]),
  ],
})

// 2. Inject in controller/service
@Controller()
export class AppController {
  constructor(
    @Inject('USERS_SERVICE') private usersClient: ClientProxy,
  ) {}
}
```

**Multiple Services:**

```typescript
@Injectable()
export class OrdersService {
  constructor(
    @Inject('USERS_SERVICE') private usersClient: ClientProxy,
    @Inject('PRODUCTS_SERVICE') private productsClient: ClientProxy,
    @Inject('PAYMENTS_SERVICE') private paymentsClient: ClientProxy,
  ) {}

  async createOrder(data: any) {
    const user = await firstValueFrom(
      this.usersClient.send('get-user', { id: data.userId }),
    );
    const products = await firstValueFrom(
      this.productsClient.send('get-products', data.productIds),
    );
    const payment = await firstValueFrom(
      this.paymentsClient.send('process-payment', data.payment),
    );
    return { user, products, payment };
  }
}
```

**Interview Tip**: Use **@Inject('TOKEN_NAME')** to inject ClientProxy. Token must match name in **ClientsModule.register()**. Can inject **multiple** ClientProxy instances for different services.

</details>

## Transports

### 18. What is TCP transport and when do you use it?

<details>
<summary>Answer</summary>

**TCP (Transmission Control Protocol)** is the **default transport** in NestJS microservices. It provides direct, **point-to-point** communication between services over TCP sockets.

**Key Characteristics:**

```
✅ Default transport (no extra dependencies)
✅ Point-to-point communication
✅ Direct TCP socket connection
✅ Low latency
✅ Simple setup
✅ Reliable delivery
✅ Connection-oriented
✅ No message broker required
```

**Server Setup:**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { Transport, MicroserviceOptions } from '@nestjs/microservices';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.TCP,  // ✅ TCP transport
      options: {
        host: '0.0.0.0',  // Bind to all interfaces
        port: 3001,        // TCP port
        retryAttempts: 5,  // Retry on connection failure
        retryDelay: 3000,  // Wait 3s between retries
      },
    },
  );
  
  await app.listen();
  console.log('TCP Microservice listening on port 3001');
}
bootstrap();
```

**Client Setup:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'USERS_SERVICE',
        transport: Transport.TCP,
        options: {
          host: 'users-service',  // Service hostname
          port: 3001,
          retryAttempts: 5,
          retryDelay: 3000,
        },
      },
    ]),
  ],
})
export class AppModule {}
```

**Configuration Options:**

```typescript
interface TcpOptions {
  host?: string;           // Host to bind/connect (default: 'localhost')
  port?: number;           // Port number (default: 3000)
  retryAttempts?: number;  // Connection retry attempts (default: 0)
  retryDelay?: number;     // Delay between retries in ms (default: 0)
}
```

**When to Use TCP:**

```
✅ Simple microservices communication
   - Direct service-to-service calls
   - No message broker overhead
   - Small to medium scale

✅ Low latency required
   - Real-time applications
   - High-performance needs
   - Direct connections faster than message brokers

✅ Development and testing
   - No external dependencies
   - Quick setup
   - Easy debugging

✅ Internal services
   - Same network/datacenter
   - Trusted environment
   - No complex routing needed

✅ Request-response patterns
   - Synchronous communication
   - Need immediate responses
   - 1-to-1 communication
```

**When NOT to Use TCP:**

```
❌ High availability requirements
   - No built-in failover
   - Single point of failure
   - Use RabbitMQ/Kafka instead

❌ Load balancing needed
   - No automatic load distribution
   - Manual implementation required
   - Use message brokers

❌ Message persistence required
   - Messages not stored
   - Lost on service restart
   - Use RabbitMQ/Kafka

❌ Pub/Sub patterns
   - One-to-many communication
   - Use Redis/RabbitMQ/Kafka

❌ Complex routing
   - Topic-based routing
   - Message filtering
   - Use RabbitMQ/Kafka
```

**Complete Example:**

```typescript
// ===== Server (Users Service) =====
// users-service/main.ts
import { NestFactory } from '@nestjs/core';
import { Transport } from '@nestjs/microservices';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.createMicroservice(AppModule, {
    transport: Transport.TCP,
    options: {
      host: '0.0.0.0',
      port: 3001,
    },
  });
  
  await app.listen();
}
bootstrap();
```

```typescript
// users-service/users.controller.ts
import { Controller } from '@nestjs/common';
import { MessagePattern, EventPattern, Payload } from '@nestjs/microservices';

@Controller()
export class UsersController {
  @MessagePattern('get-user')
  async getUser(@Payload() data: { id: string }) {
    return {
      id: data.id,
      name: 'John Doe',
      email: 'john@example.com',
    };
  }

  @MessagePattern('create-user')
  async createUser(@Payload() data: any) {
    return {
      id: '123',
      ...data,
      createdAt: new Date(),
    };
  }

  @EventPattern('user.viewed')
  handleUserViewed(@Payload() data: any) {
    console.log('User viewed:', data);
  }
}
```

```typescript
// ===== Client (API Gateway) =====
// gateway/app.module.ts
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';
import { UsersController } from './users.controller';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'USERS_SERVICE',
        transport: Transport.TCP,
        options: {
          host: 'localhost',  // or Docker service name
          port: 3001,
        },
      },
    ]),
  ],
  controllers: [UsersController],
})
export class AppModule {}
```

```typescript
// gateway/users.controller.ts
import { Controller, Get, Post, Body, Param, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { firstValueFrom } from 'rxjs';

@Controller('users')
export class UsersController {
  constructor(
    @Inject('USERS_SERVICE') private usersClient: ClientProxy,
  ) {}

  @Get(':id')
  async getUser(@Param('id') id: string) {
    const user = await firstValueFrom(
      this.usersClient.send('get-user', { id }),
    );
    return user;
  }

  @Post()
  async createUser(@Body() userData: any) {
    const user = await firstValueFrom(
      this.usersClient.send('create-user', userData),
    );
    
    this.usersClient.emit('user.viewed', { id: user.id });
    
    return user;
  }
}
```

**Docker Compose Setup:**

```yaml
# docker-compose.yml
version: '3.8'

services:
  gateway:
    build: ./gateway
    ports:
      - "3000:3000"
    environment:
      - USERS_SERVICE_HOST=users-service
      - USERS_SERVICE_PORT=3001
    depends_on:
      - users-service

  users-service:
    build: ./users-service
    environment:
      - TCP_HOST=0.0.0.0
      - TCP_PORT=3001
    # No external port mapping - internal only
```

**Environment-Based Configuration:**

```typescript
// config/microservice.config.ts
import { Transport, MicroserviceOptions } from '@nestjs/microservices';

export const getTcpConfig = (): MicroserviceOptions => ({
  transport: Transport.TCP,
  options: {
    host: process.env.TCP_HOST || '0.0.0.0',
    port: parseInt(process.env.TCP_PORT) || 3001,
    retryAttempts: parseInt(process.env.RETRY_ATTEMPTS) || 5,
    retryDelay: parseInt(process.env.RETRY_DELAY) || 3000,
  },
});
```

```typescript
// main.ts
import { getTcpConfig } from './config/microservice.config';

async function bootstrap() {
  const app = await NestFactory.createMicroservice(
    AppModule,
    getTcpConfig(),
  );
  await app.listen();
}
```

**Advantages:**

```
1. Simple Setup ✅
   - No external dependencies
   - No message broker required
   - Quick to get started

2. Low Latency ✅
   - Direct TCP connection
   - No broker overhead
   - Fast communication

3. Built-in Support ✅
   - Default in NestJS
   - Well documented
   - Battle-tested

4. Easy Debugging ✅
   - Direct connections
   - Clear error messages
   - Simple troubleshooting
```

**Disadvantages:**

```
1. No Persistence ❌
   - Messages lost on failure
   - No retry mechanism (built-in)
   - Use RabbitMQ/Kafka for persistence

2. No Load Balancing ❌
   - Single connection per client
   - Manual load distribution
   - Use message broker for LB

3. Limited Scalability ❌
   - Point-to-point only
   - No pub/sub
   - Harder to scale horizontally

4. No High Availability ❌
   - Single point of failure
   - No automatic failover
   - Need clustering solution
```

**Best Practices:**

```typescript
// ✅ GOOD: Set retry attempts
options: {
  host: 'service',
  port: 3001,
  retryAttempts: 5,
  retryDelay: 3000,
}

// ✅ GOOD: Bind to 0.0.0.0 in containers
options: {
  host: '0.0.0.0',  // All interfaces
  port: 3001,
}

// ✅ GOOD: Use environment variables
options: {
  host: process.env.SERVICE_HOST,
  port: parseInt(process.env.SERVICE_PORT),
}

// ✅ GOOD: Handle connection errors
try {
  const result = await firstValueFrom(
    this.client.send('pattern', data),
  );
} catch (error) {
  console.error('TCP connection failed:', error);
}

// ❌ BAD: Hardcoded host/port
options: {
  host: '192.168.1.100',  // Don't hardcode!
  port: 3001,
}

// ❌ BAD: No retry configuration
options: {
  host: 'service',
  port: 3001,
  // Missing retryAttempts!
}
```

**Interview Tip**: **TCP transport** is the **default** in NestJS microservices. Provides **direct point-to-point** communication over TCP sockets. **No external dependencies** (no broker needed). Use for: **simple services**, **low latency**, **development**, **internal communication**. Advantages: **simple setup**, **fast**, **easy debugging**. Disadvantages: **no persistence**, **no load balancing**, **limited scalability**. For **production with HA**, consider **RabbitMQ/Kafka**. Configure with **host**, **port**, **retryAttempts**, **retryDelay**.

</details>

### 19. What is Redis transport and when do you use it?

<details>
<summary>Answer</summary>

**Redis transport** uses Redis as a **message broker** for communication between microservices using the **Pub/Sub pattern**.

**Key Characteristics:**

```
✅ Pub/Sub pattern (one-to-many)
✅ Message broadcasting
✅ Multiple subscribers
✅ In-memory speed
✅ Lightweight message broker
✅ Event-based communication
✅ Fire-and-forget
✅ Redis required as dependency
```

**Installation:**

```bash
npm install ioredis
# or
yarn add ioredis
```

**Server Setup:**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { Transport, MicroserviceOptions } from '@nestjs/microservices';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.REDIS,  // ✅ Redis transport
      options: {
        host: 'localhost',
        port: 6379,
        retryAttempts: 5,
        retryDelay: 3000,
        // Optional: Redis auth
        password: process.env.REDIS_PASSWORD,
        db: 0,  // Redis database number
      },
    },
  );
  
  await app.listen();
  console.log('Redis Microservice is listening');
}
bootstrap();
```

**Client Setup:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'NOTIFICATIONS_SERVICE',
        transport: Transport.REDIS,
        options: {
          host: 'localhost',
          port: 6379,
          password: process.env.REDIS_PASSWORD,
        },
      },
    ]),
  ],
})
export class AppModule {}
```

**Configuration Options:**

```typescript
interface RedisOptions {
  host?: string;              // Redis host (default: 'localhost')
  port?: number;              // Redis port (default: 6379)
  db?: number;                // Redis database (default: 0)
  password?: string;          // Redis password (optional)
  retryAttempts?: number;     // Retry attempts (default: 0)
  retryDelay?: number;        // Delay between retries (ms)
  wildcards?: boolean;        // Enable wildcard patterns (default: false)
  
  // Advanced options
  family?: number;            // IP version (4 or 6)
  path?: string;              // Unix socket path
  keepAlive?: number;         // TCP keepAlive (ms)
  connectTimeout?: number;    // Connection timeout (ms)
}
```

**When to Use Redis:**

```
✅ Event broadcasting
   - One-to-many communication
   - Notify multiple services
   - Real-time updates

✅ Pub/Sub patterns
   - Event-driven architecture
   - Decoupled services
   - Async notifications

✅ Cache + messaging
   - Already using Redis for cache
   - Leverage existing infrastructure
   - Reduce dependencies

✅ Real-time applications
   - Fast in-memory operations
   - Low latency
   - Chat, notifications, live updates

✅ Simple event distribution
   - Lightweight message broker
   - No complex routing
   - Fire-and-forget events
```

**When NOT to Use Redis:**

```
❌ Message persistence required
   - Redis is in-memory only
   - Messages lost on restart
   - Use RabbitMQ/Kafka instead

❌ Guaranteed delivery needed
   - No acknowledgments by default
   - No retry mechanisms
   - Use RabbitMQ/Kafka

❌ Complex routing
   - No advanced routing logic
   - No message filtering
   - Use RabbitMQ

❌ Large message volumes
   - Limited by memory
   - No disk persistence
   - Use Kafka for big data

❌ Transaction support
   - No ACID guarantees
   - No distributed transactions
   - Use RabbitMQ/Kafka
```

**Complete Example:**

```typescript
// ===== Server 1 (Notifications Service) =====
// notifications-service/main.ts
import { NestFactory } from '@nestjs/core';
import { Transport } from '@nestjs/microservices';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.createMicroservice(AppModule, {
    transport: Transport.REDIS,
    options: {
      host: process.env.REDIS_HOST || 'localhost',
      port: parseInt(process.env.REDIS_PORT) || 6379,
    },
  });
  
  await app.listen();
  console.log('Notifications Service listening on Redis');
}
bootstrap();
```

```typescript
// notifications-service/notifications.controller.ts
import { Controller } from '@nestjs/common';
import { EventPattern, Payload, Ctx, RmqContext } from '@nestjs/microservices';

@Controller()
export class NotificationsController {
  // ✅ Listen to user.created event
  @EventPattern('user.created')
  async handleUserCreated(@Payload() data: any) {
    console.log('Notifications: User created', data);
    // Send welcome email
    await this.sendWelcomeEmail(data.email);
  }
  
  // ✅ Listen to order.placed event
  @EventPattern('order.placed')
  async handleOrderPlaced(@Payload() data: any) {
    console.log('Notifications: Order placed', data);
    // Send order confirmation
    await this.sendOrderConfirmation(data);
  }
  
  // ✅ Wildcard pattern (requires wildcards: true)
  @EventPattern('user.*')
  async handleUserEvents(@Payload() data: any, @Ctx() context: any) {
    console.log('Wildcard: User event', context.getPattern(), data);
  }
  
  private async sendWelcomeEmail(email: string) {
    // Email logic
  }
  
  private async sendOrderConfirmation(data: any) {
    // Order confirmation logic
  }
}
```

```typescript
// ===== Server 2 (Analytics Service) =====
// analytics-service/main.ts
import { NestFactory } from '@nestjs/core';
import { Transport } from '@nestjs/microservices';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.createMicroservice(AppModule, {
    transport: Transport.REDIS,
    options: {
      host: 'localhost',
      port: 6379,
    },
  });
  
  await app.listen();
  console.log('Analytics Service listening on Redis');
}
bootstrap();
```

```typescript
// analytics-service/analytics.controller.ts
import { Controller } from '@nestjs/common';
import { EventPattern, Payload } from '@nestjs/microservices';

@Controller()
export class AnalyticsController {
  // ✅ Same event, different service
  @EventPattern('user.created')
  async trackUserCreated(@Payload() data: any) {
    console.log('Analytics: User created', data);
    // Track user creation in analytics
    await this.trackEvent('user_created', data);
  }
  
  @EventPattern('order.placed')
  async trackOrderPlaced(@Payload() data: any) {
    console.log('Analytics: Order placed', data);
    // Track order in analytics
    await this.trackEvent('order_placed', data);
  }
  
  private async trackEvent(event: string, data: any) {
    // Analytics logic
  }
}
```

```typescript
// ===== Client (API Gateway) =====
// api-gateway/app.module.ts
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'REDIS_CLIENT',
        transport: Transport.REDIS,
        options: {
          host: 'localhost',
          port: 6379,
        },
      },
    ]),
  ],
})
export class AppModule {}
```

```typescript
// api-gateway/users.controller.ts
import { Controller, Post, Body, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';

@Controller('users')
export class UsersController {
  constructor(
    @Inject('REDIS_CLIENT') private client: ClientProxy,
  ) {}
  
  @Post()
  async createUser(@Body() userData: any) {
    // Create user in database
    const user = await this.usersService.create(userData);
    
    // ✅ Emit event (fire-and-forget)
    // Both Notifications and Analytics services will receive it
    this.client.emit('user.created', {
      id: user.id,
      email: user.email,
      name: user.name,
      createdAt: new Date(),
    });
    
    return user;
  }
  
  @Post('orders')
  async createOrder(@Body() orderData: any) {
    const order = await this.ordersService.create(orderData);
    
    // ✅ Broadcast to all subscribers
    this.client.emit('order.placed', {
      orderId: order.id,
      userId: order.userId,
      total: order.total,
      items: order.items,
    });
    
    return order;
  }
}
```

**Docker Compose Setup:**

```yaml
version: '3.8'

services:
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    command: redis-server --requirepass mypassword
    volumes:
      - redis-data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3
  
  api-gateway:
    build: ./api-gateway
    ports:
      - "3000:3000"
    environment:
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - REDIS_PASSWORD=mypassword
    depends_on:
      - redis
  
  notifications-service:
    build: ./notifications-service
    environment:
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - REDIS_PASSWORD=mypassword
    depends_on:
      - redis
  
  analytics-service:
    build: ./analytics-service
    environment:
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - REDIS_PASSWORD=mypassword
    depends_on:
      - redis

volumes:
  redis-data:
```

**Wildcard Patterns:**

```typescript
// Enable wildcards in server
const app = await NestFactory.createMicroservice(AppModule, {
  transport: Transport.REDIS,
  options: {
    host: 'localhost',
    port: 6379,
    wildcards: true,  // ✅ Enable wildcards
  },
});

// Use wildcards in handlers
@EventPattern('user.*')
handleAllUserEvents(@Payload() data: any) {
  // Receives: user.created, user.updated, user.deleted, etc.
}

@EventPattern('order.*.completed')
handleOrderCompleted(@Payload() data: any) {
  // Receives: order.payment.completed, order.shipping.completed
}

// Emit with pattern
this.client.emit('user.created', data);     // Matches user.*
this.client.emit('user.updated', data);     // Matches user.*
this.client.emit('order.payment.completed', data);  // Matches order.*.completed
```

**Advantages:**

```
✅ Simple Pub/Sub (easy to use)
✅ Multiple subscribers (one-to-many)
✅ Fast (in-memory operations)
✅ Lightweight (minimal overhead)
✅ Real-time (low latency)
✅ Decoupled services
✅ Already using Redis (leverage existing)
```

**Disadvantages:**

```
❌ No message persistence (lost on restart)
❌ No guaranteed delivery
❌ No message acknowledgments
❌ Limited by memory
❌ No complex routing
❌ No transaction support
❌ Single point of failure (without Redis cluster)
```

**Best Practices:**

```typescript
// ✅ GOOD: Use environment variables
options: {
  host: process.env.REDIS_HOST,
  port: parseInt(process.env.REDIS_PORT),
  password: process.env.REDIS_PASSWORD,
}

// ✅ GOOD: Enable retries
options: {
  host: 'redis',
  port: 6379,
  retryAttempts: 5,
  retryDelay: 3000,
}

// ✅ GOOD: Use structured event names
this.client.emit('user.created', data);
this.client.emit('order.payment.completed', data);

// ✅ GOOD: Use wildcards for related events
@EventPattern('user.*')
handleUserEvents(@Payload() data: any) {}

// ❌ BAD: No error handling
@EventPattern('user.created')
async handleUserCreated(@Payload() data: any) {
  // If this fails, event is lost!
  await this.sendEmail(data.email);
}

// ✅ GOOD: Error handling
@EventPattern('user.created')
async handleUserCreated(@Payload() data: any) {
  try {
    await this.sendEmail(data.email);
  } catch (error) {
    console.error('Failed to send email:', error);
    // Log to monitoring system
  }
}
```

**Interview Tip**: **Redis transport** uses **Pub/Sub pattern** for **event broadcasting** to **multiple subscribers**. Fast **in-memory** operations, **lightweight** broker. Use for: **event-driven architecture**, **real-time notifications**, **decoupled services**, **one-to-many communication**. Requires **Redis server** and **ioredis package**. Advantages: **simple**, **fast**, **multiple subscribers**. Disadvantages: **no persistence**, **no guaranteed delivery**, **memory-limited**. For **reliable messaging** with **persistence**, use **RabbitMQ/Kafka**. Redis is best for **fire-and-forget events** and **real-time updates**.

</details>

### 20. What is RabbitMQ (AMQP) and when do you use it?

<details>
<parameter name="summary">Answer</summary>

**RabbitMQ** is a **message broker** that implements the **AMQP** (Advanced Message Queuing Protocol). It provides **reliable**, **persistent**, **queued** communication between microservices.

**Key Characteristics:**

```
✅ Message persistence (disk storage)
✅ Guaranteed delivery
✅ Message acknowledgments
✅ Complex routing (exchanges, bindings)
✅ Dead letter queues
✅ Message TTL (time-to-live)
✅ Priority queues
✅ Request-response + Pub/Sub
✅ High availability (clustering)
✅ Production-ready message broker
```

**Installation:**

```bash
npm install amqplib amqp-connection-manager
# or
yarn add amqplib amqp-connection-manager
```

**Server Setup:**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { Transport, MicroserviceOptions } from '@nestjs/microservices';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.RMQ,  // ✅ RabbitMQ transport
      options: {
        urls: ['amqp://localhost:5672'],
        queue: 'orders_queue',  // Queue name
        noAck: false,           // Require acknowledgments
        persistent: true,        // Persist messages
        
        // Quality of Service (QoS)
        queueOptions: {
          durable: true,         // Queue survives broker restart
          deadLetterExchange: 'dlx',  // Dead letter exchange
          messageTtl: 60000,     // Message TTL (1 minute)
        },
        
        // Connection options
        socketOptions: {
          heartbeatIntervalInSeconds: 60,
          reconnectTimeInSeconds: 5,
        },
      },
    },
  );
  
  await app.listen();
  console.log('RabbitMQ Microservice is listening on orders_queue');
}
bootstrap();
```

**Client Setup:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'ORDERS_SERVICE',
        transport: Transport.RMQ,
        options: {
          urls: ['amqp://localhost:5672'],
          queue: 'orders_queue',
          queueOptions: {
            durable: true,
          },
        },
      },
    ]),
  ],
})
export class AppModule {}
```

**Configuration Options:**

```typescript
interface RmqOptions {
  urls: string[];              // RabbitMQ URLs
  queue: string;               // Queue name
  prefetchCount?: number;      // Messages to prefetch (default: 0)
  noAck?: boolean;            // Auto-acknowledge (default: false)
  persistent?: boolean;        // Persist messages (default: false)
  
  // Queue options
  queueOptions?: {
    durable?: boolean;         // Queue survives restart (default: true)
    exclusive?: boolean;       // Exclusive queue (default: false)
    autoDelete?: boolean;      // Delete when unused (default: false)
    deadLetterExchange?: string;  // DLX name
    deadLetterRoutingKey?: string;  // DLX routing key
    messageTtl?: number;       // Message TTL in ms
    maxLength?: number;        // Max queue length
    maxPriority?: number;      // Priority levels (0-255)
  };
  
  // Connection options
  socketOptions?: {
    heartbeatIntervalInSeconds?: number;
    reconnectTimeInSeconds?: number;
  };
  
  // Channel options
  isGlobalPrefetchCount?: boolean;  // Apply prefetch globally
}
```

**When to Use RabbitMQ:**

```
✅ Message persistence required
   - Messages stored on disk
   - Survive service/broker restarts
   - No data loss

✅ Guaranteed delivery needed
   - Acknowledgments
   - Retry mechanisms
   - Dead letter queues

✅ Complex routing
   - Multiple exchanges
   - Topic routing
   - Header-based routing

✅ Load balancing
   - Multiple consumers
   - Round-robin distribution
   - Work queues

✅ Request-response + Pub/Sub
   - Supports both patterns
   - Flexible messaging
   - RPC patterns

✅ Production systems
   - Battle-tested
   - High availability
   - Monitoring tools
```

**When NOT to Use RabbitMQ:**

```
❌ Ultra-high throughput
   - Limited to ~50k msgs/sec
   - Use Kafka for millions/sec
   - More overhead than Redis

❌ Simple use cases
   - Overkill for small apps
   - TCP might be enough
   - Redis for simple Pub/Sub

❌ Real-time streaming
   - Not optimized for streams
   - Use Kafka for event logs
   - No long-term storage

❌ No persistence needed
   - Overhead for in-memory use
   - Use Redis/TCP instead
   - Faster alternatives exist
```

**Complete Example:**

```typescript
// ===== Server (Orders Service) =====
// orders-service/main.ts
import { NestFactory } from '@nestjs/core';
import { Transport } from '@nestjs/microservices';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.createMicroservice(AppModule, {
    transport: Transport.RMQ,
    options: {
      urls: [process.env.RABBITMQ_URL || 'amqp://localhost:5672'],
      queue: 'orders_queue',
      noAck: false,
      queueOptions: {
        durable: true,
        deadLetterExchange: 'orders_dlx',
      },
    },
  });
  
  await app.listen();
}
bootstrap();
```

```typescript
// orders-service/orders.controller.ts
import { Controller } from '@nestjs/common';
import { 
  MessagePattern, 
  EventPattern, 
  Payload, 
  Ctx, 
  RmqContext 
} from '@nestjs/microservices';

@Controller()
export class OrdersController {
  // ✅ Request-Response pattern
  @MessagePattern('order.create')
  async createOrder(@Payload() data: any, @Ctx() context: RmqContext) {
    console.log('Creating order:', data);
    
    try {
      // Process order
      const order = await this.processOrder(data);
      
      // ✅ Manually acknowledge message
      const channel = context.getChannelRef();
      const originalMsg = context.getMessage();
      channel.ack(originalMsg);
      
      return {
        success: true,
        order,
      };
    } catch (error) {
      // ❌ Reject and requeue (or send to DLQ)
      const channel = context.getChannelRef();
      const originalMsg = context.getMessage();
      channel.nack(originalMsg, false, false);  // Don't requeue
      
      throw error;
    }
  }
  
  // ✅ Event pattern
  @EventPattern('order.placed')
  async handleOrderPlaced(@Payload() data: any, @Ctx() context: RmqContext) {
    console.log('Order placed event:', data);
    
    try {
      await this.sendOrderConfirmation(data);
      
      // Acknowledge
      const channel = context.getChannelRef();
      const originalMsg = context.getMessage();
      channel.ack(originalMsg);
    } catch (error) {
      console.error('Failed to handle order.placed:', error);
      
      // Reject and send to DLQ
      const channel = context.getChannelRef();
      const originalMsg = context.getMessage();
      channel.nack(originalMsg, false, false);
    }
  }
  
  private async processOrder(data: any) {
    // Order processing logic
    return { id: 'order-123', ...data };
  }
  
  private async sendOrderConfirmation(data: any) {
    // Send confirmation email
  }
}
```

```typescript
// ===== Client (API Gateway) =====
// api-gateway/app.module.ts
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'ORDERS_SERVICE',
        transport: Transport.RMQ,
        options: {
          urls: ['amqp://localhost:5672'],
          queue: 'orders_queue',
          queueOptions: {
            durable: true,
          },
        },
      },
    ]),
  ],
})
export class AppModule {}
```

```typescript
// api-gateway/orders.controller.ts
import { Controller, Post, Body, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { firstValueFrom, timeout } from 'rxjs';

@Controller('orders')
export class OrdersController {
  constructor(
    @Inject('ORDERS_SERVICE') private ordersClient: ClientProxy,
  ) {}
  
  @Post()
  async createOrder(@Body() orderData: any) {
    try {
      // ✅ Request-Response (wait for response)
      const result = await firstValueFrom(
        this.ordersClient
          .send('order.create', orderData)
          .pipe(timeout(5000)),  // 5s timeout
      );
      
      // ✅ Emit event (fire-and-forget)
      this.ordersClient.emit('order.placed', {
        orderId: result.order.id,
        userId: orderData.userId,
        timestamp: new Date(),
      });
      
      return result;
    } catch (error) {
      console.error('Order creation failed:', error);
      throw error;
    }
  }
}
```

**Docker Compose Setup:**

```yaml
version: '3.8'

services:
  rabbitmq:
    image: rabbitmq:3-management-alpine
    ports:
      - "5672:5672"    # AMQP port
      - "15672:15672"  # Management UI
    environment:
      - RABBITMQ_DEFAULT_USER=admin
      - RABBITMQ_DEFAULT_PASS=password
    volumes:
      - rabbitmq-data:/var/lib/rabbitmq
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
  
  api-gateway:
    build: ./api-gateway
    ports:
      - "3000:3000"
    environment:
      - RABBITMQ_URL=amqp://admin:password@rabbitmq:5672
    depends_on:
      rabbitmq:
        condition: service_healthy
  
  orders-service:
    build: ./orders-service
    environment:
      - RABBITMQ_URL=amqp://admin:password@rabbitmq:5672
    depends_on:
      rabbitmq:
        condition: service_healthy
    deploy:
      replicas: 3  # ✅ Load balancing

volumes:
  rabbitmq-data:
```

**Dead Letter Queue (DLQ):**

```typescript
// Server with DLQ configuration
const app = await NestFactory.createMicroservice(AppModule, {
  transport: Transport.RMQ,
  options: {
    urls: ['amqp://localhost:5672'],
    queue: 'orders_queue',
    queueOptions: {
      durable: true,
      // ✅ Dead Letter Exchange
      deadLetterExchange: 'orders_dlx',
      deadLetterRoutingKey: 'orders_dlq',
      // ✅ Message TTL (1 minute)
      messageTtl: 60000,
    },
  },
});

// Controller with retry logic
@MessagePattern('order.process')
async processOrder(@Payload() data: any, @Ctx() context: RmqContext) {
  const channel = context.getChannelRef();
  const originalMsg = context.getMessage();
  const retryCount = originalMsg.properties.headers['x-retry-count'] || 0;
  
  try {
    await this.doProcessOrder(data);
    channel.ack(originalMsg);
  } catch (error) {
    if (retryCount < 3) {
      // ✅ Retry: Reject and requeue
      channel.nack(originalMsg, false, true);
    } else {
      // ❌ Max retries: Send to DLQ
      channel.nack(originalMsg, false, false);
    }
  }
}
```

**Priority Queue:**

```typescript
// Server with priority queue
const app = await NestFactory.createMicroservice(AppModule, {
  transport: Transport.RMQ,
  options: {
    urls: ['amqp://localhost:5672'],
    queue: 'orders_queue',
    queueOptions: {
      durable: true,
      maxPriority: 10,  // ✅ Enable priorities (0-10)
    },
  },
});

// Client sending priority messages
this.ordersClient.send('order.create', {
  ...orderData,
  priority: 10,  // High priority
});
```

**Load Balancing:**

```typescript
// Multiple consumers (workers) on same queue
// RabbitMQ automatically distributes messages round-robin

// Worker 1
const app1 = await NestFactory.createMicroservice(AppModule, {
  transport: Transport.RMQ,
  options: {
    urls: ['amqp://localhost:5672'],
    queue: 'orders_queue',  // Same queue
    prefetchCount: 1,       // ✅ Fair dispatch
  },
});

// Worker 2 (same queue!)
const app2 = await NestFactory.createMicroservice(AppModule, {
  transport: Transport.RMQ,
  options: {
    urls: ['amqp://localhost:5672'],
    queue: 'orders_queue',  // Same queue
    prefetchCount: 1,
  },
});

// Messages distributed evenly between workers
```

**Advantages:**

```
✅ Message persistence (durable)
✅ Guaranteed delivery (acks)
✅ Complex routing (exchanges)
✅ Dead letter queues (DLQ)
✅ Load balancing (multiple consumers)
✅ High availability (clustering)
✅ Mature ecosystem (monitoring, tools)
✅ Both request-response & Pub/Sub
```

**Disadvantages:**

```
❌ Lower throughput (vs Kafka)
❌ More complex setup
❌ Higher resource usage
❌ Not ideal for streaming
❌ Overkill for simple cases
```

**Best Practices:**

```typescript
// ✅ GOOD: Enable durability
queueOptions: {
  durable: true,         // Queue survives restart
}
options: {
  persistent: true,      // Messages persist
}

// ✅ GOOD: Use acknowledgments
options: {
  noAck: false,         // Require acks
}
channel.ack(originalMsg);  // Manual ack

// ✅ GOOD: Configure DLQ
queueOptions: {
  deadLetterExchange: 'dlx',
  deadLetterRoutingKey: 'dlq',
}

// ✅ GOOD: Set prefetch for fair dispatch
options: {
  prefetchCount: 1,     // Process one at a time
}

// ✅ GOOD: Use environment variables
urls: [process.env.RABBITMQ_URL],

// ❌ BAD: Auto-acknowledge (can lose messages)
options: {
  noAck: true,  // Don't do this!
}

// ❌ BAD: No error handling
@MessagePattern('order.create')
async createOrder(@Payload() data: any) {
  // If this fails, message is lost!
  return await this.processOrder(data);
}

// ✅ GOOD: Proper error handling with acks
@MessagePattern('order.create')
async createOrder(@Payload() data: any, @Ctx() context: RmqContext) {
  try {
    const result = await this.processOrder(data);
    const channel = context.getChannelRef();
    const originalMsg = context.getMessage();
    channel.ack(originalMsg);
    return result;
  } catch (error) {
    const channel = context.getChannelRef();
    const originalMsg = context.getMessage();
    channel.nack(originalMsg, false, false);
    throw error;
  }
}
```

**Interview Tip**: **RabbitMQ** is a **message broker** using **AMQP protocol** for **reliable**, **persistent** communication. Provides **guaranteed delivery**, **message acknowledgments**, **dead letter queues**, **complex routing**, and **load balancing**. Use for: **production systems**, **reliable messaging**, **work queues**, **both request-response and Pub/Sub**. Advantages: **persistence**, **durability**, **high availability**, **mature ecosystem**. Disadvantages: **complex setup**, **lower throughput than Kafka**, **overkill for simple cases**. Requires **amqplib** and **amqp-connection-manager** packages. Configure with **queue**, **queueOptions** (durable, DLX, TTL), **prefetchCount** for fair dispatch. For **ultra-high throughput**, use **Kafka**. For **simple Pub/Sub**, use **Redis**.

</details>

### 21. What is Kafka and when do you use it?

<details>
<summary>Answer</summary>

**Apache Kafka** is a **distributed event streaming platform** designed for **high-throughput**, **fault-tolerant**, **real-time data pipelines** and **event-driven architectures**.

**Key Characteristics:**

```
✅ High throughput (millions of messages/sec)
✅ Distributed & scalable
✅ Message persistence (disk-based)
✅ Event streaming & replay
✅ Fault-tolerant (replication)
✅ Ordered message delivery (per partition)
✅ Consumer groups (load balancing)
✅ Long-term storage
✅ Real-time & batch processing
✅ Event sourcing support
```

**Installation:**

```bash
npm install kafkajs
# or
yarn add kafkajs
```

**Server Setup:**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { Transport, MicroserviceOptions } from '@nestjs/microservices';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.KAFKA,  // ✅ Kafka transport
      options: {
        client: {
          clientId: 'orders-service',  // Service identifier
          brokers: ['localhost:9092'], // Kafka brokers
          retry: {
            retries: 5,
            initialRetryTime: 300,
          },
        },
        consumer: {
          groupId: 'orders-consumer-group',  // Consumer group
          allowAutoTopicCreation: true,
          retry: {
            retries: 3,
          },
        },
      },
    },
  );
  
  await app.listen();
  console.log('Kafka Microservice is listening');
}
bootstrap();
```

**Client Setup:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'ORDERS_SERVICE',
        transport: Transport.KAFKA,
        options: {
          client: {
            clientId: 'api-gateway',
            brokers: ['localhost:9092'],
          },
          consumer: {
            groupId: 'api-gateway-consumer',
          },
        },
      },
    ]),
  ],
})
export class AppModule {}
```

**Configuration Options:**

```typescript
interface KafkaOptions {
  client: {
    clientId: string;           // Client identifier
    brokers: string[];          // Kafka broker URLs
    ssl?: boolean;              // Enable SSL
    sasl?: {                    // SASL authentication
      mechanism: string;
      username: string;
      password: string;
    };
    connectionTimeout?: number;  // Connection timeout (ms)
    requestTimeout?: number;     // Request timeout (ms)
    retry?: {
      retries?: number;
      initialRetryTime?: number;
    };
  };
  
  consumer: {
    groupId: string;                    // Consumer group ID
    sessionTimeout?: number;            // Session timeout (ms)
    heartbeatInterval?: number;         // Heartbeat interval (ms)
    allowAutoTopicCreation?: boolean;   // Auto-create topics
    maxBytesPerPartition?: number;      // Max bytes per partition
    retry?: {
      retries?: number;
    };
  };
  
  producer?: {
    allowAutoTopicCreation?: boolean;
    transactionTimeout?: number;
    retry?: {
      retries?: number;
    };
  };
  
  // Additional options
  subscribe?: {
    fromBeginning?: boolean;  // Read from start of topic
  };
  
  run?: {
    autoCommit?: boolean;     // Auto-commit offsets
    autoCommitInterval?: number;
    autoCommitThreshold?: number;
  };
}
```

**When to Use Kafka:**

```
✅ High throughput requirements
   - Millions of messages/sec
   - Massive data streams
   - Real-time analytics

✅ Event streaming
   - Event sourcing
   - CQRS patterns
   - Audit logs

✅ Data pipelines
   - ETL processes
   - Data integration
   - Stream processing

✅ Event replay needed
   - Replay historical events
   - Debugging & testing
   - Data recovery

✅ Long-term storage
   - Store events indefinitely
   - Compliance requirements
   - Historical analysis

✅ Real-time processing
   - Stream processing
   - Real-time analytics
   - Live dashboards

✅ Decoupled microservices
   - Event-driven architecture
   - Multiple consumers
   - Scalable systems
```

**When NOT to Use Kafka:**

```
❌ Simple request-response
   - Overkill for basic RPC
   - Use TCP or RabbitMQ
   - More complex setup

❌ Small scale applications
   - Resource-intensive
   - Complex infrastructure
   - Use Redis/RabbitMQ

❌ Guaranteed ordering across topics
   - Only guaranteed per partition
   - Complex multi-partition logic
   - Use RabbitMQ for simple ordering

❌ Low latency critical (<10ms)
   - Higher latency than TCP
   - Batching introduces delay
   - Use gRPC/TCP instead

❌ No streaming requirements
   - Just need Pub/Sub
   - Use Redis for simplicity
   - RabbitMQ for reliability
```

**Complete Example:**

```typescript
// ===== Server (Orders Service) =====
// orders-service/main.ts
import { NestFactory } from '@nestjs/core';
import { Transport } from '@nestjs/microservices';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.createMicroservice(AppModule, {
    transport: Transport.KAFKA,
    options: {
      client: {
        clientId: 'orders-service',
        brokers: [process.env.KAFKA_BROKER || 'localhost:9092'],
      },
      consumer: {
        groupId: 'orders-consumer-group',
        allowAutoTopicCreation: true,
      },
    },
  });
  
  await app.listen();
}
bootstrap();
```

```typescript
// orders-service/orders.controller.ts
import { Controller } from '@nestjs/common';
import { 
  MessagePattern, 
  EventPattern, 
  Payload, 
  Ctx, 
  KafkaContext 
} from '@nestjs/microservices';

@Controller()
export class OrdersController {
  // ✅ Request-Response pattern
  @MessagePattern('order.create')
  async createOrder(@Payload() data: any, @Ctx() context: KafkaContext) {
    const { topic, partition, offset } = context.getMessage();
    console.log('Kafka message:', { topic, partition, offset });
    
    try {
      const order = await this.processOrder(data);
      
      // ✅ Commit offset manually (if autoCommit: false)
      // await context.getConsumer().commitOffsets([
      //   { topic, partition, offset: (parseInt(offset) + 1).toString() },
      // ]);
      
      return {
        success: true,
        order,
      };
    } catch (error) {
      console.error('Order creation failed:', error);
      throw error;
    }
  }
  
  // ✅ Event pattern (no response expected)
  @EventPattern('order.placed')
  async handleOrderPlaced(@Payload() data: any, @Ctx() context: KafkaContext) {
    const { topic, partition, offset, timestamp } = context.getMessage();
    
    console.log('Order placed event:', {
      data,
      topic,
      partition,
      offset,
      timestamp,
    });
    
    try {
      await this.sendOrderNotification(data);
      // Auto-committed if autoCommit: true
    } catch (error) {
      console.error('Failed to handle order.placed:', error);
      // Implement retry logic or DLQ
    }
  }
  
  // ✅ Topic with key (for partitioning)
  @EventPattern('order.updated')
  async handleOrderUpdated(@Payload() data: any, @Ctx() context: KafkaContext) {
    const key = context.getMessage().key?.toString();
    console.log('Order updated with key:', key, data);
    
    // Messages with same key go to same partition (ordered)
    await this.updateOrderStatus(data);
  }
  
  private async processOrder(data: any) {
    return { id: 'order-123', ...data };
  }
  
  private async sendOrderNotification(data: any) {
    // Notification logic
  }
  
  private async updateOrderStatus(data: any) {
    // Update logic
  }
}
```

```typescript
// ===== Client (API Gateway) =====
// api-gateway/app.module.ts
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'ORDERS_SERVICE',
        transport: Transport.KAFKA,
        options: {
          client: {
            clientId: 'api-gateway',
            brokers: ['localhost:9092'],
          },
          consumer: {
            groupId: 'api-gateway-consumer',
          },
        },
      },
    ]),
  ],
})
export class AppModule {}
```

```typescript
// api-gateway/orders.controller.ts
import { Controller, Post, Body, Inject, OnModuleInit } from '@nestjs/common';
import { ClientKafka } from '@nestjs/microservices';
import { firstValueFrom, timeout } from 'rxjs';

@Controller('orders')
export class OrdersController implements OnModuleInit {
  constructor(
    @Inject('ORDERS_SERVICE') private kafkaClient: ClientKafka,
  ) {}
  
  // ✅ Subscribe to response topics
  async onModuleInit() {
    // Required for request-response patterns
    this.kafkaClient.subscribeToResponseOf('order.create');
    await this.kafkaClient.connect();
  }
  
  @Post()
  async createOrder(@Body() orderData: any) {
    try {
      // ✅ Request-Response (wait for response)
      const result = await firstValueFrom(
        this.kafkaClient
          .send('order.create', {
            key: orderData.userId,  // ✅ Partition key
            value: orderData,
          })
          .pipe(timeout(10000)),  // 10s timeout
      );
      
      // ✅ Emit event (fire-and-forget)
      this.kafkaClient.emit('order.placed', {
        key: orderData.userId,
        value: {
          orderId: result.order.id,
          userId: orderData.userId,
          timestamp: new Date().toISOString(),
        },
      });
      
      return result;
    } catch (error) {
      console.error('Order creation failed:', error);
      throw error;
    }
  }
  
  // ✅ Batch processing
  @Post('batch')
  async createBatchOrders(@Body() orders: any[]) {
    const messages = orders.map(order => ({
      key: order.userId,
      value: order,
    }));
    
    // Send multiple messages
    for (const message of messages) {
      this.kafkaClient.emit('order.create', message);
    }
    
    return { success: true, count: messages.length };
  }
}
```

**Docker Compose Setup:**

```yaml
version: '3.8'

services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.4.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - "2181:2181"
  
  kafka:
    image: confluentinc/cp-kafka:7.4.0
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
      - "9093:9093"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9093,PLAINTEXT_HOST://localhost:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"
    volumes:
      - kafka-data:/var/lib/kafka/data
  
  kafka-ui:
    image: provectuslabs/kafka-ui:latest
    depends_on:
      - kafka
    ports:
      - "8080:8080"
    environment:
      KAFKA_CLUSTERS_0_NAME: local
      KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka:9093
  
  api-gateway:
    build: ./api-gateway
    ports:
      - "3000:3000"
    environment:
      - KAFKA_BROKER=kafka:9093
    depends_on:
      - kafka
  
  orders-service:
    build: ./orders-service
    environment:
      - KAFKA_BROKER=kafka:9093
    depends_on:
      - kafka
    deploy:
      replicas: 3  # ✅ Multiple consumers for load balancing

volumes:
  kafka-data:
```

**Consumer Groups (Load Balancing):**

```typescript
// Multiple instances of same service with same groupId
// Kafka distributes partitions among consumers in group

// Instance 1
const app1 = await NestFactory.createMicroservice(AppModule, {
  transport: Transport.KAFKA,
  options: {
    client: { clientId: 'orders-1', brokers: ['localhost:9092'] },
    consumer: { 
      groupId: 'orders-consumer-group',  // ✅ Same group
    },
  },
});

// Instance 2 (same groupId!)
const app2 = await NestFactory.createMicroservice(AppModule, {
  transport: Transport.KAFKA,
  options: {
    client: { clientId: 'orders-2', brokers: ['localhost:9092'] },
    consumer: { 
      groupId: 'orders-consumer-group',  // ✅ Same group
    },
  },
});

// Kafka automatically load balances partitions:
// If topic has 3 partitions:
// - Consumer 1 might get partition 0, 1
// - Consumer 2 might get partition 2
```

**Event Replay:**

```typescript
// Read from beginning of topic (replay all events)
const app = await NestFactory.createMicroservice(AppModule, {
  transport: Transport.KAFKA,
  options: {
    client: {
      clientId: 'replay-service',
      brokers: ['localhost:9092'],
    },
    consumer: {
      groupId: 'replay-consumer',  // Different group for replay
    },
    subscribe: {
      fromBeginning: true,  // ✅ Read all historical events
    },
  },
});

// Useful for:
// - Rebuilding read models (CQRS)
// - Data recovery
// - Testing
// - Debugging
```

**Message Ordering:**

```typescript
// ✅ Ordering guaranteed within partition
// Use same key for related events

// All orders from user-123 go to same partition (ordered)
this.kafkaClient.emit('order.created', {
  key: 'user-123',  // ✅ Partition key
  value: { orderId: 'order-1', userId: 'user-123' },
});

this.kafkaClient.emit('order.updated', {
  key: 'user-123',  // ✅ Same key = same partition
  value: { orderId: 'order-1', status: 'shipped' },
});

// Events with different keys may be out of order
this.kafkaClient.emit('order.created', {
  key: 'user-456',  // Different partition
  value: { orderId: 'order-2', userId: 'user-456' },
});
```

**Advantages:**

```
✅ Ultra-high throughput (millions/sec)
✅ Horizontal scalability
✅ Fault tolerance (replication)
✅ Event streaming & replay
✅ Ordered delivery (per partition)
✅ Long-term storage
✅ Consumer groups (load balancing)
✅ Real-time + batch processing
✅ Event sourcing support
✅ Battle-tested (LinkedIn, Netflix, Uber)
```

**Disadvantages:**

```
❌ Complex setup (Zookeeper/KRaft)
❌ Resource-intensive (memory, disk)
❌ Higher latency than TCP/gRPC
❌ Overkill for simple use cases
❌ Ordering only per partition
❌ Steep learning curve
❌ Operational overhead
```

**Best Practices:**

```typescript
// ✅ GOOD: Use partition keys for ordering
this.kafkaClient.emit('order.updated', {
  key: orderData.userId,  // Related events to same partition
  value: orderData,
});

// ✅ GOOD: Different consumer groups for different services
consumer: {
  groupId: 'orders-service-group',  // Unique per service
}

// ✅ GOOD: Subscribe to response topics for request-response
async onModuleInit() {
  this.kafkaClient.subscribeToResponseOf('order.create');
  await this.kafkaClient.connect();
}

// ✅ GOOD: Handle errors gracefully
@EventPattern('order.placed')
async handleOrderPlaced(@Payload() data: any) {
  try {
    await this.processOrder(data);
  } catch (error) {
    // Log and implement retry/DLQ
    console.error('Processing failed:', error);
  }
}

// ✅ GOOD: Use environment variables
client: {
  brokers: [process.env.KAFKA_BROKER],
}

// ❌ BAD: No partition key (random distribution)
this.kafkaClient.emit('order.updated', orderData);

// ❌ BAD: Same consumer group for different services
// Both services will compete for messages!
consumer: {
  groupId: 'shared-group',  // Don't share!
}

// ❌ BAD: Not subscribing to response topics
// Request-response won't work!
```

**Interview Tip**: **Kafka** is a **distributed event streaming platform** for **high-throughput**, **fault-tolerant** data pipelines. Handles **millions of messages/sec**, provides **message persistence**, **event replay**, and **ordered delivery per partition**. Use for: **event streaming**, **event sourcing**, **real-time analytics**, **data pipelines**, **high-volume systems**. Requires **kafkajs** package. Configure with **clientId**, **brokers**, **consumer groupId**. Advantages: **ultra-high throughput**, **scalability**, **fault tolerance**, **long-term storage**. Disadvantages: **complex setup**, **resource-intensive**, **overkill for simple cases**. Use **partition keys** for ordering. For **simple Pub/Sub**, use **Redis**. For **reliable queuing**, use **RabbitMQ**. For **ultra-low latency**, use **gRPC/TCP**.

</details>

### 22. What is gRPC and what are its advantages?

<details>
<summary>Answer</summary>

**gRPC** (gRPC Remote Procedure Call) is a **high-performance**, **language-agnostic** RPC framework developed by Google that uses **HTTP/2** and **Protocol Buffers** (protobuf) for efficient communication.

**Key Characteristics:**

```
✅ HTTP/2 based (multiplexing, streaming)
✅ Protocol Buffers (binary, type-safe)
✅ Bidirectional streaming
✅ Code generation from .proto files
✅ Language-agnostic (polyglot)
✅ High performance (low latency)
✅ Strongly typed contracts
✅ Built-in authentication (TLS)
✅ Load balancing support
✅ Efficient serialization
```

**Installation:**

```bash
npm install @grpc/grpc-js @grpc/proto-loader
# or
yarn add @grpc/grpc-js @grpc/proto-loader
```

**Protocol Buffers (.proto) Definition:**

```protobuf
// orders.proto
syntax = "proto3";

package orders;

service OrdersService {
  // ✅ Unary RPC (request-response)
  rpc CreateOrder (CreateOrderRequest) returns (CreateOrderResponse);
  
  // ✅ Server streaming (one request, stream responses)
  rpc GetOrderUpdates (GetOrderUpdatesRequest) returns (stream OrderUpdate);
  
  // ✅ Client streaming (stream requests, one response)
  rpc CreateBulkOrders (stream CreateOrderRequest) returns (BulkOrderResponse);
  
  // ✅ Bidirectional streaming (stream both ways)
  rpc ProcessOrders (stream OrderRequest) returns (stream OrderResponse);
}

message CreateOrderRequest {
  string user_id = 1;
  repeated OrderItem items = 2;
  double total = 3;
}

message CreateOrderResponse {
  string order_id = 1;
  string status = 2;
  int64 created_at = 3;
}

message OrderItem {
  string product_id = 1;
  int32 quantity = 2;
  double price = 3;
}

message GetOrderUpdatesRequest {
  string order_id = 1;
}

message OrderUpdate {
  string order_id = 1;
  string status = 2;
  int64 timestamp = 3;
}

message BulkOrderResponse {
  int32 success_count = 1;
  int32 failed_count = 2;
}

message OrderRequest {
  string order_id = 1;
  string action = 2;
}

message OrderResponse {
  string order_id = 1;
  bool success = 2;
  string message = 3;
}
```

**Server Setup:**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { Transport, MicroserviceOptions } from '@nestjs/microservices';
import { join } from 'path';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.GRPC,  // ✅ gRPC transport
      options: {
        package: 'orders',  // Package name from .proto
        protoPath: join(__dirname, '../proto/orders.proto'),
        url: '0.0.0.0:5000',  // gRPC server address
        
        // Optional configurations
        loader: {
          keepCase: true,      // Keep field names as-is
          longs: String,       // Convert longs to strings
          enums: String,       // Convert enums to strings
          defaults: true,      // Set default values
          oneofs: true,        // Include oneOf fields
        },
        
        // TLS/SSL (optional)
        credentials: null,  // Use insecure for development
      },
    },
  );
  
  await app.listen();
  console.log('gRPC Microservice listening on 0.0.0.0:5000');
}
bootstrap();
```

**Server Controller:**

```typescript
// orders.controller.ts
import { Controller } from '@nestjs/common';
import { GrpcMethod, GrpcStreamMethod } from '@nestjs/microservices';
import { Observable, Subject } from 'rxjs';

interface CreateOrderRequest {
  userId: string;
  items: OrderItem[];
  total: number;
}

interface CreateOrderResponse {
  orderId: string;
  status: string;
  createdAt: number;
}

interface OrderItem {
  productId: string;
  quantity: number;
  price: number;
}

interface GetOrderUpdatesRequest {
  orderId: string;
}

interface OrderUpdate {
  orderId: string;
  status: string;
  timestamp: number;
}

@Controller()
export class OrdersController {
  // ✅ Unary RPC (request-response)
  @GrpcMethod('OrdersService', 'CreateOrder')
  async createOrder(data: CreateOrderRequest): Promise<CreateOrderResponse> {
    console.log('Creating order:', data);
    
    const order = await this.processOrder(data);
    
    return {
      orderId: order.id,
      status: 'CREATED',
      createdAt: Date.now(),
    };
  }
  
  // ✅ Server streaming (send multiple responses)
  @GrpcStreamMethod('OrdersService', 'GetOrderUpdates')
  getOrderUpdates(request: Observable<GetOrderUpdatesRequest>): Observable<OrderUpdate> {
    const subject = new Subject<OrderUpdate>();
    
    request.subscribe(data => {
      console.log('Streaming updates for order:', data.orderId);
      
      // Simulate order status updates
      const updates = [
        { orderId: data.orderId, status: 'PROCESSING', timestamp: Date.now() },
        { orderId: data.orderId, status: 'SHIPPED', timestamp: Date.now() + 1000 },
        { orderId: data.orderId, status: 'DELIVERED', timestamp: Date.now() + 2000 },
      ];
      
      updates.forEach((update, index) => {
        setTimeout(() => {
          subject.next(update);
          if (index === updates.length - 1) {
            subject.complete();
          }
        }, index * 1000);
      });
    });
    
    return subject.asObservable();
  }
  
  // ✅ Client streaming (receive multiple requests)
  @GrpcStreamMethod('OrdersService', 'CreateBulkOrders')
  createBulkOrders(data: Observable<CreateOrderRequest>): Observable<any> {
    const subject = new Subject();
    let successCount = 0;
    let failedCount = 0;
    
    data.subscribe({
      next: (order) => {
        console.log('Processing bulk order:', order);
        try {
          // Process order
          this.processBulkOrder(order);
          successCount++;
        } catch (error) {
          failedCount++;
        }
      },
      complete: () => {
        // Send final response
        subject.next({
          successCount,
          failedCount,
        });
        subject.complete();
      },
    });
    
    return subject.asObservable();
  }
  
  // ✅ Bidirectional streaming
  @GrpcStreamMethod('OrdersService', 'ProcessOrders')
  processOrders(data: Observable<any>): Observable<any> {
    const subject = new Subject();
    
    data.subscribe(request => {
      console.log('Processing order request:', request);
      
      // Process and immediately respond
      const response = {
        orderId: request.orderId,
        success: true,
        message: `Order ${request.orderId} processed: ${request.action}`,
      };
      
      subject.next(response);
    });
    
    return subject.asObservable();
  }
  
  private async processOrder(data: CreateOrderRequest) {
    return { id: 'order-123', ...data };
  }
  
  private processBulkOrder(data: CreateOrderRequest) {
    // Bulk processing logic
  }
}
```

**Client Setup:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';
import { join } from 'path';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'ORDERS_PACKAGE',
        transport: Transport.GRPC,
        options: {
          package: 'orders',
          protoPath: join(__dirname, '../proto/orders.proto'),
          url: 'localhost:5000',
          loader: {
            keepCase: true,
          },
        },
      },
    ]),
  ],
})
export class AppModule {}
```

**Client Controller:**

```typescript
// api-gateway/orders.controller.ts
import { 
  Controller, 
  Post, 
  Get, 
  Body, 
  Param, 
  Inject, 
  OnModuleInit 
} from '@nestjs/common';
import { ClientGrpc } from '@nestjs/microservices';
import { Observable, firstValueFrom } from 'rxjs';

interface OrdersService {
  createOrder(data: any): Observable<any>;
  getOrderUpdates(data: any): Observable<any>;
  createBulkOrders(data: Observable<any>): Observable<any>;
  processOrders(data: Observable<any>): Observable<any>;
}

@Controller('orders')
export class OrdersController implements OnModuleInit {
  private ordersService: OrdersService;
  
  constructor(
    @Inject('ORDERS_PACKAGE') private client: ClientGrpc,
  ) {}
  
  onModuleInit() {
    // ✅ Get service from gRPC client
    this.ordersService = this.client.getService<OrdersService>('OrdersService');
  }
  
  // ✅ Unary RPC
  @Post()
  async createOrder(@Body() orderData: any) {
    try {
      const result = await firstValueFrom(
        this.ordersService.createOrder({
          userId: orderData.userId,
          items: orderData.items,
          total: orderData.total,
        }),
      );
      
      return result;
    } catch (error) {
      console.error('gRPC call failed:', error);
      throw error;
    }
  }
  
  // ✅ Server streaming
  @Get(':id/updates')
  getOrderUpdates(@Param('id') orderId: string): Observable<any> {
    // Return stream directly to client
    return this.ordersService.getOrderUpdates({ orderId });
  }
  
  // ✅ Client streaming
  @Post('bulk')
  async createBulkOrders(@Body() orders: any[]) {
    const { Observable } = require('rxjs');
    
    // Create stream of orders
    const orders$ = new Observable(observer => {
      orders.forEach(order => {
        observer.next(order);
      });
      observer.complete();
    });
    
    // Send stream and get response
    const result = await firstValueFrom(
      this.ordersService.createBulkOrders(orders$),
    );
    
    return result;
  }
}
```

**Configuration Options:**

```typescript
interface GrpcOptions {
  package: string | string[];     // Package name(s) from .proto
  protoPath: string | string[];   // Path(s) to .proto file(s)
  url: string;                    // Server address (host:port)
  
  // Proto loader options
  loader?: {
    keepCase?: boolean;           // Keep field names as-is
    longs?: any;                  // How to handle longs (String/Number)
    enums?: any;                  // How to handle enums (String/Number)
    defaults?: boolean;           // Set default values
    oneofs?: boolean;             // Include oneOf fields
    arrays?: boolean;             // Parse repeated fields as arrays
  };
  
  // gRPC credentials (TLS/SSL)
  credentials?: any;
  
  // Channel options
  channelOptions?: {
    'grpc.max_send_message_length'?: number;
    'grpc.max_receive_message_length'?: number;
    'grpc.keepalive_time_ms'?: number;
    'grpc.keepalive_timeout_ms'?: number;
  };
  
  // Metadata
  protoLoader?: string;           // Custom proto loader path
  maxSendMessageLength?: number;  // Max send size
  maxReceiveMessageLength?: number;  // Max receive size
}
```

**Advantages of gRPC:**

```
✅ High Performance
   - Binary protocol (Protocol Buffers)
   - HTTP/2 multiplexing
   - Lower latency than REST/JSON
   - Efficient serialization

✅ Type Safety
   - Strongly typed contracts (.proto)
   - Code generation
   - Compile-time validation
   - Interface definition language (IDL)

✅ Streaming Support
   - Server streaming (one-to-many)
   - Client streaming (many-to-one)
   - Bidirectional streaming
   - Real-time communication

✅ Language Agnostic
   - Works across languages
   - Polyglot microservices
   - Consistent contracts
   - Wide language support

✅ HTTP/2 Features
   - Multiplexing (multiple requests on one connection)
   - Header compression
   - Server push
   - Binary framing

✅ Built-in Features
   - Authentication (TLS)
   - Load balancing
   - Timeouts
   - Deadlines
   - Cancellation

✅ Code Generation
   - Auto-generate client/server code
   - Reduced boilerplate
   - Less error-prone
   - Faster development

✅ Smaller Payload
   - Binary format (vs JSON)
   - Compressed data
   - Reduced bandwidth
   - Cost savings
```

**Disadvantages:**

```
❌ Browser Support
   - Limited browser support
   - Requires gRPC-Web proxy
   - Not ideal for web clients
   - REST/GraphQL better for browsers

❌ Human Readability
   - Binary format (not human-readable)
   - Harder to debug
   - Need special tools
   - Less transparent than JSON

❌ Learning Curve
   - Protocol Buffers syntax
   - gRPC concepts
   - Streaming patterns
   - More complex than REST

❌ Tooling
   - Less mature than REST
   - Fewer debugging tools
   - Limited API testing tools
   - Postman support limited

❌ Backward Compatibility
   - Proto changes can break clients
   - Version management needed
   - Careful schema evolution
   - Migration complexity
```

**When to Use gRPC:**

```
✅ Microservice-to-microservice communication
✅ High-performance requirements
✅ Real-time streaming (chat, live data)
✅ Polyglot systems (multiple languages)
✅ Internal APIs (not browser-facing)
✅ Low latency critical
✅ Strong type safety needed
✅ Bidirectional communication
```

**When NOT to Use gRPC:**

```
❌ Browser clients (use REST/GraphQL)
❌ Public APIs (less accessible)
❌ Simple CRUD operations (REST is simpler)
❌ Human-readable APIs (debugging)
❌ Team unfamiliar with protobuf
```

**Docker Compose Example:**

```yaml
version: '3.8'

services:
  orders-grpc-service:
    build: ./orders-service
    ports:
      - "5000:5000"
    environment:
      - GRPC_PORT=5000
    volumes:
      - ./proto:/app/proto
  
  api-gateway:
    build: ./api-gateway
    ports:
      - "3000:3000"
    environment:
      - ORDERS_GRPC_URL=orders-grpc-service:5000
    depends_on:
      - orders-grpc-service
    volumes:
      - ./proto:/app/proto
```

**Best Practices:**

```typescript
// ✅ GOOD: Use type interfaces
interface CreateOrderRequest {
  userId: string;
  items: OrderItem[];
  total: number;
}

// ✅ GOOD: Error handling
@GrpcMethod('OrdersService', 'CreateOrder')
async createOrder(data: CreateOrderRequest) {
  try {
    return await this.processOrder(data);
  } catch (error) {
    throw new RpcException({
      code: status.INTERNAL,
      message: error.message,
    });
  }
}

// ✅ GOOD: Use environment variables
url: process.env.GRPC_URL || 'localhost:5000',

// ✅ GOOD: Initialize service in onModuleInit
onModuleInit() {
  this.ordersService = this.client.getService<OrdersService>('OrdersService');
}

// ✅ GOOD: Use streaming for real-time data
@GrpcStreamMethod('OrdersService', 'GetOrderUpdates')
getOrderUpdates(data: Observable<any>): Observable<any> {
  // Stream updates to client
}

// ❌ BAD: Not handling stream completion
@GrpcStreamMethod('OrdersService', 'GetData')
getData(data: Observable<any>): Observable<any> {
  const subject = new Subject();
  data.subscribe(item => subject.next(item));
  // Missing: subject.complete()!
  return subject.asObservable();
}

// ❌ BAD: Hardcoded proto paths
protoPath: './proto/orders.proto',  // Won't work in prod!

// ✅ GOOD: Use join for cross-platform paths
protoPath: join(__dirname, '../proto/orders.proto'),
```

**Interview Tip**: **gRPC** is a **high-performance RPC framework** using **HTTP/2** and **Protocol Buffers** for efficient, **type-safe** communication. Advantages: **high performance**, **low latency**, **bidirectional streaming**, **language-agnostic**, **type safety**, **code generation**. Use for: **microservice-to-microservice**, **real-time streaming**, **polyglot systems**, **internal APIs**, **high-performance needs**. Disadvantages: **limited browser support**, **binary format** (not human-readable), **steeper learning curve**. Define contracts in **.proto files**, use **@GrpcMethod()** for unary RPC, **@GrpcStreamMethod()** for streaming. Requires **@grpc/grpc-js** and **@grpc/proto-loader**. For **browser clients**, use **REST/GraphQL**. For **simple CRUD**, **REST is simpler**.

</details>

### 23. What is MQTT transport used for?

<details>
<summary>Answer</summary>

**MQTT** (Message Queuing Telemetry Transport) is a **lightweight**, **publish-subscribe** messaging protocol designed for **IoT devices**, **low-bandwidth** networks, and **unreliable connections**.

**Key Characteristics:**

```
✅ Lightweight protocol (minimal overhead)
✅ Pub/Sub pattern (topics)
✅ QoS levels (0, 1, 2)
✅ Low bandwidth usage
✅ Battery-efficient
✅ Unreliable network support
✅ TCP/IP based
✅ Last Will and Testament (LWT)
✅ Retained messages
✅ IoT optimized
```

**Installation:**

```bash
npm install mqtt
# or
yarn add mqtt
```

**Server Setup:**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { Transport, MicroserviceOptions } from '@nestjs/microservices';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.MQTT,  // ✅ MQTT transport
      options: {
        url: 'mqtt://localhost:1883',  // MQTT broker URL
        
        // Client options
        clientId: 'iot-device-service',
        clean: true,              // Clean session
        keepalive: 60,            // Keepalive interval (seconds)
        reconnectPeriod: 5000,    // Reconnect delay (ms)
        connectTimeout: 30000,    // Connection timeout (ms)
        
        // Authentication (optional)
        username: process.env.MQTT_USERNAME,
        password: process.env.MQTT_PASSWORD,
        
        // QoS (Quality of Service)
        queueQosZero: true,       // Queue QoS 0 messages
        
        // Last Will and Testament
        will: {
          topic: 'devices/iot-device/status',
          payload: 'offline',
          qos: 1,
          retain: true,
        },
      },
    },
  );
  
  await app.listen();
  console.log('MQTT Microservice is listening');
}
bootstrap();
```

**Client Setup:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'IOT_SERVICE',
        transport: Transport.MQTT,
        options: {
          url: 'mqtt://localhost:1883',
          clientId: 'api-gateway',
          clean: true,
          keepalive: 60,
          username: process.env.MQTT_USERNAME,
          password: process.env.MQTT_PASSWORD,
        },
      },
    ]),
  ],
})
export class AppModule {}
```

**Configuration Options:**

```typescript
interface MqttOptions {
  url: string;                  // MQTT broker URL (mqtt://host:port)
  
  // Client options
  clientId?: string;            // Unique client identifier
  clean?: boolean;              // Clean session (default: true)
  keepalive?: number;           // Keepalive interval (seconds)
  reconnectPeriod?: number;     // Reconnect delay (ms)
  connectTimeout?: number;      // Connection timeout (ms)
  queueQosZero?: boolean;       // Queue QoS 0 messages
  
  // Authentication
  username?: string;            // MQTT username
  password?: string;            // MQTT password
  
  // TLS/SSL
  protocol?: 'mqtt' | 'mqtts' | 'ws' | 'wss';  // Protocol
  ca?: Buffer;                  // CA certificate
  cert?: Buffer;                // Client certificate
  key?: Buffer;                 // Client key
  rejectUnauthorized?: boolean; // Reject invalid certs
  
  // Last Will and Testament
  will?: {
    topic: string;              // Will topic
    payload: string | Buffer;   // Will message
    qos?: 0 | 1 | 2;           // Will QoS
    retain?: boolean;           // Retain will message
  };
  
  // Subscription options
  subscribeOptions?: {
    qos?: 0 | 1 | 2;           // QoS level
  };
  
  // Serialization
  serializer?: any;             // Custom serializer
  deserializer?: any;           // Custom deserializer
}
```

**QoS Levels:**

```
QoS 0 (At most once)
  - Fire and forget
  - No acknowledgment
  - Fastest but least reliable
  - Use for: sensor data, logs

QoS 1 (At least once)
  - Acknowledged delivery
  - May receive duplicates
  - Balanced reliability
  - Use for: important events

QoS 2 (Exactly once)
  - Guaranteed delivery (no duplicates)
  - Slowest but most reliable
  - 4-way handshake
  - Use for: critical commands
```

**When to Use MQTT:**

```
✅ IoT applications
   - Smart home devices
   - Sensors & actuators
   - Industrial IoT
   - Connected devices

✅ Low bandwidth networks
   - Mobile networks (3G/4G)
   - Satellite connections
   - Remote locations
   - Limited data plans

✅ Unreliable connections
   - Intermittent connectivity
   - High latency
   - Packet loss
   - Network instability

✅ Battery-powered devices
   - Energy efficiency critical
   - Long battery life needed
   - Sleep mode support
   - Minimize network traffic

✅ Real-time telemetry
   - Sensor data streaming
   - Live monitoring
   - Event notifications
   - Device status updates

✅ Pub/Sub patterns
   - Topic-based routing
   - Wildcard subscriptions
   - One-to-many communication
   - Decoupled systems
```

**When NOT to Use MQTT:**

```
❌ Large payloads
   - Limited message size
   - Use HTTP/Kafka instead
   - Not for file transfers

❌ Request-response patterns
   - Primarily Pub/Sub
   - No built-in RPC
   - Use HTTP/gRPC instead

❌ Complex routing
   - Simple topic structure
   - No advanced filtering
   - Use RabbitMQ/Kafka

❌ Message persistence
   - Limited retention
   - Not for long-term storage
   - Use Kafka for event logs

❌ High throughput
   - Not optimized for volume
   - Use Kafka/RabbitMQ
   - Better alternatives exist
```

**Complete Example:**

```typescript
// ===== Server (IoT Device Service) =====
// iot-service/main.ts
import { NestFactory } from '@nestjs/core';
import { Transport } from '@nestjs/microservices';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.createMicroservice(AppModule, {
    transport: Transport.MQTT,
    options: {
      url: process.env.MQTT_URL || 'mqtt://localhost:1883',
      clientId: 'iot-device-service',
      clean: true,
      keepalive: 60,
      will: {
        topic: 'devices/service/status',
        payload: 'offline',
        qos: 1,
        retain: true,
      },
    },
  });
  
  await app.listen();
}
bootstrap();
```

```typescript
// iot-service/iot.controller.ts
import { Controller } from '@nestjs/common';
import { 
  MessagePattern, 
  EventPattern, 
  Payload, 
  Ctx, 
  MqttContext 
} from '@nestjs/microservices';

@Controller()
export class IotController {
  // ✅ Listen to specific topic
  @EventPattern('sensors/temperature')
  handleTemperature(@Payload() data: any, @Ctx() context: MqttContext) {
    const topic = context.getTopic();
    const packet = context.getPacket();
    
    console.log('Temperature reading:', {
      topic,
      qos: packet.qos,
      retain: packet.retain,
      data,
    });
    
    // Process temperature data
    this.processTemperature(data);
  }
  
  // ✅ Wildcard subscription (single level)
  @EventPattern('sensors/+/humidity')
  handleHumidity(@Payload() data: any, @Ctx() context: MqttContext) {
    const topic = context.getTopic();
    console.log('Humidity from topic:', topic, data);
    // Matches: sensors/room1/humidity, sensors/room2/humidity
  }
  
  // ✅ Wildcard subscription (multiple levels)
  @EventPattern('devices/#')
  handleAllDevices(@Payload() data: any, @Ctx() context: MqttContext) {
    const topic = context.getTopic();
    console.log('Device event:', topic, data);
    // Matches: devices/sensor/temp, devices/actuator/status, etc.
  }
  
  // ✅ Command pattern (device control)
  @MessagePattern('devices/actuator/command')
  handleActuatorCommand(@Payload() data: any, @Ctx() context: MqttContext) {
    console.log('Actuator command:', data);
    
    // Execute command
    const result = this.executeCommand(data);
    
    // Return response (published to reply topic)
    return {
      success: true,
      result,
      timestamp: Date.now(),
    };
  }
  
  // ✅ Status updates
  @EventPattern('devices/+/status')
  handleDeviceStatus(@Payload() data: any, @Ctx() context: MqttContext) {
    const topic = context.getTopic();
    const deviceId = topic.split('/')[1];
    
    console.log(`Device ${deviceId} status:`, data);
    this.updateDeviceStatus(deviceId, data);
  }
  
  private processTemperature(data: any) {
    // Temperature processing logic
  }
  
  private executeCommand(data: any) {
    // Command execution logic
    return { action: data.action, status: 'completed' };
  }
  
  private updateDeviceStatus(deviceId: string, status: any) {
    // Status update logic
  }
}
```

```typescript
// ===== Client (API Gateway) =====
// api-gateway/app.module.ts
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'IOT_SERVICE',
        transport: Transport.MQTT,
        options: {
          url: 'mqtt://localhost:1883',
          clientId: 'api-gateway',
          clean: true,
        },
      },
    ]),
  ],
})
export class AppModule {}
```

```typescript
// api-gateway/iot.controller.ts
import { Controller, Post, Get, Body, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';

@Controller('iot')
export class IotController {
  constructor(
    @Inject('IOT_SERVICE') private iotClient: ClientProxy,
  ) {}
  
  // ✅ Publish sensor data (fire-and-forget)
  @Post('sensors/temperature')
  publishTemperature(@Body() data: any) {
    this.iotClient.emit('sensors/temperature', {
      deviceId: data.deviceId,
      temperature: data.temperature,
      unit: 'celsius',
      timestamp: Date.now(),
    });
    
    return { success: true, message: 'Temperature published' };
  }
  
  // ✅ Send command to actuator
  @Post('actuator/command')
  async sendCommand(@Body() command: any) {
    try {
      const result = await this.iotClient
        .send('devices/actuator/command', command)
        .toPromise();
      
      return result;
    } catch (error) {
      console.error('Command failed:', error);
      throw error;
    }
  }
  
  // ✅ Publish device status
  @Post('device/:id/status')
  publishDeviceStatus(@Param('id') deviceId: string, @Body() status: any) {
    this.iotClient.emit(`devices/${deviceId}/status`, {
      deviceId,
      ...status,
      timestamp: Date.now(),
    });
    
    return { success: true };
  }
  
  // ✅ Publish with QoS
  @Post('sensors/critical')
  publishCriticalData(@Body() data: any) {
    // Publish with QoS 2 (exactly once)
    this.iotClient.emit('sensors/critical', {
      value: data,
      qos: 2,  // ✅ QoS level
      retain: true,  // ✅ Retain message
    });
    
    return { success: true };
  }
}
```

**Docker Compose Setup:**

```yaml
version: '3.8'

services:
  mosquitto:
    image: eclipse-mosquitto:2
    ports:
      - "1883:1883"    # MQTT port
      - "9001:9001"    # WebSocket port
    volumes:
      - ./mosquitto.conf:/mosquitto/config/mosquitto.conf
      - mosquitto-data:/mosquitto/data
      - mosquitto-logs:/mosquitto/log
    healthcheck:
      test: ["CMD", "mosquitto_sub", "-t", "$$SYS/#", "-C", "1", "-W", "1"]
      interval: 10s
      timeout: 5s
      retries: 3
  
  api-gateway:
    build: ./api-gateway
    ports:
      - "3000:3000"
    environment:
      - MQTT_URL=mqtt://mosquitto:1883
    depends_on:
      mosquitto:
        condition: service_healthy
  
  iot-service:
    build: ./iot-service
    environment:
      - MQTT_URL=mqtt://mosquitto:1883
    depends_on:
      mosquitto:
        condition: service_healthy

volumes:
  mosquitto-data:
  mosquitto-logs:
```

**Mosquitto Configuration (mosquitto.conf):**

```conf
# mosquitto.conf
listener 1883
allow_anonymous true

# WebSocket support
listener 9001
protocol websockets

# Persistence
persistence true
persistence_location /mosquitto/data/

# Logging
log_dest file /mosquitto/log/mosquitto.log
log_dest stdout
log_type all

# Authentication (optional)
# password_file /mosquitto/config/passwd
```

**Topic Wildcards:**

```
+ (single level wildcard)
  sensors/+/temperature
  - Matches: sensors/room1/temperature
  - Matches: sensors/room2/temperature
  - Not: sensors/room1/floor1/temperature

# (multi-level wildcard)
  devices/#
  - Matches: devices/sensor
  - Matches: devices/sensor/temp
  - Matches: devices/actuator/status
  - Matches: devices/room1/floor2/sensor/temp

Examples:
  sensors/+/temp       → sensors/room1/temp ✅
  sensors/#            → sensors/room1/temp/high ✅
  +/temperature        → room1/temperature ✅
  devices/+/+/status   → devices/floor1/room1/status ✅
```

**Retained Messages:**

```typescript
// Publish retained message (last known state)
this.iotClient.emit('devices/sensor/status', {
  value: { online: true, lastSeen: Date.now() },
  retain: true,  // ✅ Retain on broker
});

// New subscribers immediately receive retained message
@EventPattern('devices/sensor/status')
handleStatus(@Payload() data: any) {
  // Receives last retained message immediately on subscription
  console.log('Sensor status:', data);
}
```

**Last Will and Testament (LWT):**

```typescript
// Server will message
const app = await NestFactory.createMicroservice(AppModule, {
  transport: Transport.MQTT,
  options: {
    url: 'mqtt://localhost:1883',
    clientId: 'iot-device',
    will: {
      topic: 'devices/iot-device/status',
      payload: JSON.stringify({ status: 'offline', timestamp: Date.now() }),
      qos: 1,
      retain: true,
    },
  },
});

// When client disconnects unexpectedly,
// broker publishes will message automatically
```

**Advantages:**

```
✅ Lightweight (minimal overhead)
✅ Low bandwidth usage
✅ Battery efficient
✅ Unreliable network support
✅ QoS levels (reliability options)
✅ Last Will & Testament (offline detection)
✅ Retained messages (last known state)
✅ Wildcard subscriptions
✅ Topic-based routing
✅ IoT optimized
```

**Disadvantages:**

```
❌ Limited message size
❌ No complex routing
❌ Limited message retention
❌ Not for high throughput
❌ No built-in security (add TLS)
❌ Simple topic structure
❌ Not for large payloads
```

**Best Practices:**

```typescript
// ✅ GOOD: Use structured topic names
this.iotClient.emit('sensors/temperature/room1', data);
this.iotClient.emit('devices/actuator/command', data);

// ✅ GOOD: Use appropriate QoS
this.iotClient.emit('sensors/temp', { value: data, qos: 0 });  // Sensor data
this.iotClient.emit('commands/actuator', { value: data, qos: 2 });  // Critical

// ✅ GOOD: Use retained messages for status
this.iotClient.emit('devices/sensor/status', {
  value: { online: true },
  retain: true,
});

// ✅ GOOD: Configure Last Will
will: {
  topic: 'devices/service/status',
  payload: 'offline',
  qos: 1,
  retain: true,
}

// ✅ GOOD: Use wildcards for subscriptions
@EventPattern('sensors/+/temperature')
@EventPattern('devices/#')

// ✅ GOOD: Use environment variables
url: process.env.MQTT_URL,
username: process.env.MQTT_USERNAME,
password: process.env.MQTT_PASSWORD,

// ❌ BAD: Long topic names
this.iotClient.emit('very/long/topic/structure/that/is/unnecessary', data);

// ❌ BAD: No QoS for critical data
this.iotClient.emit('commands/critical', data);  // Defaults to QoS 0!

// ❌ BAD: Hardcoded broker URL
url: 'mqtt://localhost:1883',  // Use env vars!
```

**Interview Tip**: **MQTT** is a **lightweight Pub/Sub protocol** designed for **IoT devices**, **low-bandwidth** networks, and **unreliable connections**. Provides **3 QoS levels** (0: at most once, 1: at least once, 2: exactly once), **Last Will and Testament** (offline detection), **retained messages** (last known state), and **wildcard subscriptions** (+ and #). Use for: **IoT applications**, **sensors/actuators**, **battery-powered devices**, **unreliable networks**, **real-time telemetry**. Requires **mqtt** package and **MQTT broker** (Mosquitto, HiveMQ). Advantages: **lightweight**, **battery-efficient**, **low bandwidth**, **QoS options**. Disadvantages: **limited message size**, **no complex routing**, **not for high throughput**. For **high-volume data**, use **Kafka**. For **reliable queuing**, use **RabbitMQ**.

</details>

## Request-Response

### 24. How do you send messages using `client.send()`?

<details>
<summary>Answer</summary>

**`client.send()`** is used for **request-response** communication in NestJS microservices. It sends a message and **waits for a response**, returning an **Observable**.

**Basic Syntax:**

```typescript
client.send(pattern, data): Observable<any>
```

**Key Characteristics:**

```
✅ Request-response pattern (waits for response)
✅ Returns Observable (RxJS)
✅ Pattern-based routing
✅ Synchronous-like behavior
✅ Error handling via RxJS operators
✅ Timeout support
✅ Retry support
✅ Response transformation
```

**Basic Usage:**

```typescript
import { Controller, Get, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { firstValueFrom, timeout } from 'rxjs';

@Controller('users')
export class UsersController {
  constructor(
    @Inject('USERS_SERVICE') private client: ClientProxy,
  ) {}
  
  @Get()
  async getUsers() {
    // ✅ Basic send (convert Observable to Promise)
    const result = await firstValueFrom(
      this.client.send('users.findAll', {}),
    );
    
    return result;
  }
  
  @Get(':id')
  async getUserById(@Param('id') id: string) {
    // ✅ Send with data
    const result = await firstValueFrom(
      this.client.send('users.findById', { id }),
    );
    
    return result;
  }
  
  @Post()
  async createUser(@Body() userData: any) {
    // ✅ Send with payload
    const result = await firstValueFrom(
      this.client.send('users.create', userData),
    );
    
    return result;
  }
}
```

**Pattern Types:**

```typescript
// ✅ String pattern (simple)
this.client.send('users.create', data);
this.client.send('orders.findAll', {});

// ✅ Dot notation (hierarchical)
this.client.send('users.profile.update', data);
this.client.send('orders.payment.process', data);

// ✅ Object pattern (structured)
this.client.send({ cmd: 'create_user' }, data);
this.client.send({ cmd: 'get_user', version: 'v2' }, data);

// ✅ Multiple fields
this.client.send({
  service: 'users',
  action: 'create',
  version: 'v1',
}, data);
```

**With Timeout:**

```typescript
import { timeout, catchError } from 'rxjs/operators';
import { of } from 'rxjs';

@Get(':id')
async getUser(@Param('id') id: string) {
  try {
    // ✅ 5 second timeout
    const result = await firstValueFrom(
      this.client.send('users.findById', { id }).pipe(
        timeout(5000),  // Timeout after 5s
        catchError(error => {
          console.error('Request timeout:', error);
          throw new RequestTimeoutException('Service unavailable');
        }),
      ),
    );
    
    return result;
  } catch (error) {
    throw error;
  }
}
```

**With Retry:**

```typescript
import { retry, catchError } from 'rxjs/operators';

@Post()
async createOrder(@Body() orderData: any) {
  try {
    // ✅ Retry 3 times on failure
    const result = await firstValueFrom(
      this.client.send('orders.create', orderData).pipe(
        retry(3),  // Retry up to 3 times
        catchError(error => {
          console.error('All retries failed:', error);
          throw new BadGatewayException('Order service unavailable');
        }),
      ),
    );
    
    return result;
  } catch (error) {
    throw error;
  }
}
```

**With Timeout and Retry:**

```typescript
import { timeout, retry, catchError } from 'rxjs/operators';

@Get(':id')
async getOrder(@Param('id') id: string) {
  try {
    const result = await firstValueFrom(
      this.client.send('orders.findById', { id }).pipe(
        timeout(5000),    // 5s timeout
        retry(2),         // Retry 2 times
        catchError(error => {
          if (error.name === 'TimeoutError') {
            throw new RequestTimeoutException('Order service timeout');
          }
          throw new BadGatewayException('Order service error');
        }),
      ),
    );
    
    return result;
  } catch (error) {
    throw error;
  }
}
```

**Error Handling:**

```typescript
import { catchError } from 'rxjs/operators';
import { throwError } from 'rxjs';

@Post()
async createUser(@Body() userData: any) {
  try {
    const result = await firstValueFrom(
      this.client.send('users.create', userData).pipe(
        catchError(error => {
          console.error('Microservice error:', error);
          
          // Log error
          this.logger.error('User creation failed', error);
          
          // Transform error
          if (error.message?.includes('duplicate')) {
            throw new ConflictException('User already exists');
          }
          
          throw new BadGatewayException('User service unavailable');
        }),
      ),
    );
    
    return result;
  } catch (error) {
    throw error;
  }
}
```

**Response Transformation:**

```typescript
import { map } from 'rxjs/operators';

@Get()
async getUsers() {
  // ✅ Transform response
  const result = await firstValueFrom(
    this.client.send('users.findAll', {}).pipe(
      map(response => {
        // Transform response data
        return {
          data: response.users,
          total: response.count,
          page: response.page,
        };
      }),
    ),
  );
  
  return result;
}
```

**Parallel Requests:**

```typescript
import { forkJoin } from 'rxjs';

@Get(':id/profile')
async getUserProfile(@Param('id') userId: string) {
  // ✅ Make parallel requests
  const result = await firstValueFrom(
    forkJoin({
      user: this.client.send('users.findById', { id: userId }),
      orders: this.client.send('orders.findByUser', { userId }),
      settings: this.client.send('settings.findByUser', { userId }),
    }),
  );
  
  return {
    user: result.user,
    orders: result.orders,
    settings: result.settings,
  };
}
```

**Sequential Requests:**

```typescript
@Post('checkout')
async checkout(@Body() checkoutData: any) {
  try {
    // Step 1: Validate inventory
    const inventory = await firstValueFrom(
      this.client.send('inventory.validate', checkoutData.items),
    );
    
    if (!inventory.available) {
      throw new BadRequestException('Items not available');
    }
    
    // Step 2: Process payment
    const payment = await firstValueFrom(
      this.client.send('payment.process', {
        amount: checkoutData.total,
        method: checkoutData.paymentMethod,
      }),
    );
    
    // Step 3: Create order
    const order = await firstValueFrom(
      this.client.send('orders.create', {
        items: checkoutData.items,
        paymentId: payment.id,
        userId: checkoutData.userId,
      }),
    );
    
    return order;
  } catch (error) {
    throw error;
  }
}
```

**Complete Example:**

```typescript
// ===== Client (API Gateway) =====
// api-gateway/app.module.ts
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'USERS_SERVICE',
        transport: Transport.TCP,
        options: {
          host: 'users-service',
          port: 3001,
        },
      },
      {
        name: 'ORDERS_SERVICE',
        transport: Transport.TCP,
        options: {
          host: 'orders-service',
          port: 3002,
        },
      },
    ]),
  ],
})
export class AppModule {}
```

```typescript
// api-gateway/users.controller.ts
import { 
  Controller, 
  Get, 
  Post, 
  Put, 
  Delete, 
  Body, 
  Param, 
  Inject,
  BadGatewayException,
  NotFoundException,
} from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { firstValueFrom, timeout, retry, catchError } from 'rxjs';

@Controller('users')
export class UsersController {
  constructor(
    @Inject('USERS_SERVICE') private usersClient: ClientProxy,
    @Inject('ORDERS_SERVICE') private ordersClient: ClientProxy,
  ) {}
  
  // ✅ GET all users
  @Get()
  async findAll() {
    try {
      return await firstValueFrom(
        this.usersClient.send('users.findAll', {}).pipe(
          timeout(5000),
          retry(2),
        ),
      );
    } catch (error) {
      throw new BadGatewayException('Users service unavailable');
    }
  }
  
  // ✅ GET user by ID
  @Get(':id')
  async findById(@Param('id') id: string) {
    try {
      const user = await firstValueFrom(
        this.usersClient.send('users.findById', { id }).pipe(
          timeout(5000),
        ),
      );
      
      if (!user) {
        throw new NotFoundException('User not found');
      }
      
      return user;
    } catch (error) {
      if (error instanceof NotFoundException) {
        throw error;
      }
      throw new BadGatewayException('Users service error');
    }
  }
  
  // ✅ POST create user
  @Post()
  async create(@Body() userData: any) {
    try {
      return await firstValueFrom(
        this.usersClient.send('users.create', userData).pipe(
          timeout(10000),  // Longer timeout for creation
          retry(1),        // Retry once
        ),
      );
    } catch (error) {
      throw new BadGatewayException('Failed to create user');
    }
  }
  
  // ✅ PUT update user
  @Put(':id')
  async update(@Param('id') id: string, @Body() updateData: any) {
    try {
      return await firstValueFrom(
        this.usersClient.send('users.update', { id, ...updateData }).pipe(
          timeout(5000),
        ),
      );
    } catch (error) {
      throw new BadGatewayException('Failed to update user');
    }
  }
  
  // ✅ DELETE user
  @Delete(':id')
  async delete(@Param('id') id: string) {
    try {
      return await firstValueFrom(
        this.usersClient.send('users.delete', { id }).pipe(
          timeout(5000),
        ),
      );
    } catch (error) {
      throw new BadGatewayException('Failed to delete user');
    }
  }
  
  // ✅ Complex query with parallel requests
  @Get(':id/dashboard')
  async getDashboard(@Param('id') userId: string) {
    try {
      const { forkJoin } = require('rxjs');
      
      const result = await firstValueFrom(
        forkJoin({
          user: this.usersClient.send('users.findById', { id: userId }),
          orders: this.ordersClient.send('orders.findByUser', { userId }),
          stats: this.usersClient.send('users.getStats', { id: userId }),
        }).pipe(
          timeout(10000),  // Timeout for all requests
        ),
      );
      
      return {
        user: result.user,
        recentOrders: result.orders.slice(0, 5),
        statistics: result.stats,
      };
    } catch (error) {
      throw new BadGatewayException('Failed to load dashboard');
    }
  }
}
```

```typescript
// ===== Server (Users Service) =====
// users-service/users.controller.ts
import { Controller } from '@nestjs/common';
import { MessagePattern, Payload } from '@nestjs/microservices';

@Controller()
export class UsersController {
  // ✅ Handle request and return response
  @MessagePattern('users.findAll')
  async findAll(@Payload() data: any) {
    console.log('Finding all users');
    const users = await this.usersService.findAll();
    return users;
  }
  
  @MessagePattern('users.findById')
  async findById(@Payload() data: { id: string }) {
    console.log('Finding user:', data.id);
    const user = await this.usersService.findById(data.id);
    return user || null;
  }
  
  @MessagePattern('users.create')
  async create(@Payload() userData: any) {
    console.log('Creating user:', userData);
    const user = await this.usersService.create(userData);
    return user;
  }
  
  @MessagePattern('users.update')
  async update(@Payload() data: any) {
    console.log('Updating user:', data.id);
    const user = await this.usersService.update(data.id, data);
    return user;
  }
  
  @MessagePattern('users.delete')
  async delete(@Payload() data: { id: string }) {
    console.log('Deleting user:', data.id);
    await this.usersService.delete(data.id);
    return { success: true };
  }
  
  @MessagePattern('users.getStats')
  async getStats(@Payload() data: { id: string }) {
    console.log('Getting stats for user:', data.id);
    const stats = await this.usersService.getStats(data.id);
    return stats;
  }
}
```

**Best Practices:**

```typescript
// ✅ GOOD: Always use timeout
this.client.send('pattern', data).pipe(
  timeout(5000),  // Prevent hanging
)

// ✅ GOOD: Handle errors properly
this.client.send('pattern', data).pipe(
  timeout(5000),
  catchError(error => {
    console.error('Error:', error);
    throw new BadGatewayException();
  }),
)

// ✅ GOOD: Use retry for transient failures
this.client.send('pattern', data).pipe(
  timeout(5000),
  retry(2),  // Retry on network issues
)

// ✅ GOOD: Convert Observable to Promise
const result = await firstValueFrom(
  this.client.send('pattern', data),
);

// ✅ GOOD: Use forkJoin for parallel requests
forkJoin({
  users: this.client.send('users.findAll', {}),
  orders: this.client.send('orders.findAll', {}),
})

// ❌ BAD: No timeout (can hang forever)
const result = await firstValueFrom(
  this.client.send('pattern', data),  // No timeout!
);

// ❌ BAD: No error handling
const result = await firstValueFrom(
  this.client.send('pattern', data),  // No catchError!
);

// ❌ BAD: Not converting Observable
return this.client.send('pattern', data);  // Returns Observable!

// ❌ BAD: Using toPromise() (deprecated)
const result = await this.client.send('pattern', data).toPromise();
// Use firstValueFrom() instead!
```

**Interview Tip**: **`client.send()`** is used for **request-response** communication in microservices. Sends a message with a **pattern** and **data**, **waits for response**, returns **Observable**. Use **`firstValueFrom()`** to convert Observable to Promise. Always add **`timeout()`** to prevent hanging, **`retry()`** for transient failures, **`catchError()`** for error handling. Use **`forkJoin()`** for parallel requests. Pattern can be **string** ('users.create') or **object** ({ cmd: 'create' }). Different from **`emit()`** which is **fire-and-forget**. Common operators: **timeout()**, **retry()**, **catchError()**, **map()**, **forkJoin()**. Best practices: **always timeout**, **handle errors**, **convert to Promise**, **use retry for network issues**.

</details>

### 25. How do you handle responses from microservices?

<details>
<summary>Answer</summary>

Handling responses from microservices involves **converting Observables to Promises**, **error handling**, **response transformation**, and **applying RxJS operators** for timeout, retry, and composition.

**Methods to Handle Responses:**

```
✅ firstValueFrom() - Convert Observable to Promise
✅ RxJS operators (map, catchError, timeout, retry)
✅ Error handling (try-catch, catchError)
✅ Response transformation (map, tap)
✅ Parallel requests (forkJoin)
✅ Sequential composition (switchMap, mergeMap)
```

**1. Basic Response Handling (firstValueFrom):**

```typescript
import { firstValueFrom } from 'rxjs';
import { Controller, Get, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';

@Controller('users')
export class UsersController {
  constructor(
    @Inject('USERS_SERVICE') private client: ClientProxy,
  ) {}
  
  // ✅ Basic: Convert Observable to Promise
  @Get()
  async getUsers() {
    const result = await firstValueFrom(
      this.client.send('users.findAll', {}),
    );
    
    return result;
  }
  
  // ✅ With data extraction
  @Get(':id')
  async getUser(@Param('id') id: string) {
    const user = await firstValueFrom(
      this.client.send('users.findById', { id }),
    );
    
    if (!user) {
      throw new NotFoundException('User not found');
    }
    
    return user;
  }
}
```

**2. With Timeout:**

```typescript
import { firstValueFrom, timeout } from 'rxjs';

@Get(':id')
async getUser(@Param('id') id: string) {
  try {
    // ✅ Add 5 second timeout
    const user = await firstValueFrom(
      this.client.send('users.findById', { id }).pipe(
        timeout(5000),  // Timeout after 5s
      ),
    );
    
    return user;
  } catch (error) {
    if (error.name === 'TimeoutError') {
      throw new RequestTimeoutException('User service timeout');
    }
    throw error;
  }
}
```

**3. With Error Handling:**

```typescript
import { firstValueFrom, timeout, catchError } from 'rxjs';
import { throwError } from 'rxjs';

@Post()
async createUser(@Body() userData: any) {
  try {
    const result = await firstValueFrom(
      this.client.send('users.create', userData).pipe(
        timeout(10000),
        catchError(error => {
          // Log error
          console.error('Microservice error:', error);
          
          // Handle specific errors
          if (error.message?.includes('duplicate')) {
            throw new ConflictException('User already exists');
          }
          
          if (error.name === 'TimeoutError') {
            throw new RequestTimeoutException('Service timeout');
          }
          
          // Generic error
          throw new BadGatewayException('User service unavailable');
        }),
      ),
    );
    
    return result;
  } catch (error) {
    // This catches errors thrown in catchError
    throw error;
  }
}
```

**4. With Retry:**

```typescript
import { firstValueFrom, timeout, retry, catchError } from 'rxjs';

@Get(':id')
async getUser(@Param('id') id: string) {
  try {
    const user = await firstValueFrom(
      this.client.send('users.findById', { id }).pipe(
        timeout(5000),
        retry({
          count: 3,           // Retry 3 times
          delay: 1000,        // Wait 1s between retries
        }),
        catchError(error => {
          console.error('All retries failed:', error);
          throw new BadGatewayException('User service unavailable');
        }),
      ),
    );
    
    return user;
  } catch (error) {
    throw error;
  }
}
```

**5. With Response Transformation:**

```typescript
import { firstValueFrom, map } from 'rxjs';

@Get()
async getUsers(@Query() query: any) {
  // ✅ Transform response
  const result = await firstValueFrom(
    this.client.send('users.findAll', query).pipe(
      map(response => {
        // Transform the response
        return {
          data: response.users,
          meta: {
            total: response.count,
            page: response.page,
            limit: response.limit,
          },
        };
      }),
    ),
  );
  
  return result;
}
```

**6. With Response Validation:**

```typescript
import { firstValueFrom, map, tap } from 'rxjs';

@Get(':id')
async getUser(@Param('id') id: string) {
  const user = await firstValueFrom(
    this.client.send('users.findById', { id }).pipe(
      tap(response => {
        // ✅ Validate response
        if (!response || !response.id) {
          throw new Error('Invalid response format');
        }
      }),
      map(response => {
        // Transform response
        return {
          id: response.id,
          name: response.name,
          email: response.email,
        };
      }),
    ),
  );
  
  return user;
}
```

**7. Parallel Requests (forkJoin):**

```typescript
import { firstValueFrom, forkJoin, timeout } from 'rxjs';

@Get(':id/profile')
async getUserProfile(@Param('id') userId: string) {
  try {
    // ✅ Execute multiple requests in parallel
    const result = await firstValueFrom(
      forkJoin({
        user: this.client.send('users.findById', { id: userId }),
        orders: this.ordersClient.send('orders.findByUser', { userId }),
        preferences: this.settingsClient.send('settings.get', { userId }),
      }).pipe(
        timeout(10000),  // Timeout for all requests
      ),
    );
    
    return {
      user: result.user,
      recentOrders: result.orders.slice(0, 5),
      preferences: result.preferences,
    };
  } catch (error) {
    throw new BadGatewayException('Failed to load profile');
  }
}
```

**8. Sequential Requests (Chaining):**

```typescript
import { firstValueFrom, timeout } from 'rxjs';

@Post('checkout')
async checkout(@Body() checkoutData: any) {
  try {
    // Step 1: Validate inventory
    const inventory = await firstValueFrom(
      this.inventoryClient.send('inventory.validate', checkoutData.items).pipe(
        timeout(5000),
      ),
    );
    
    if (!inventory.available) {
      throw new BadRequestException('Items not available');
    }
    
    // Step 2: Process payment (depends on inventory)
    const payment = await firstValueFrom(
      this.paymentClient.send('payment.process', {
        amount: checkoutData.total,
        method: checkoutData.paymentMethod,
      }).pipe(
        timeout(10000),
      ),
    );
    
    if (!payment.success) {
      throw new BadRequestException('Payment failed');
    }
    
    // Step 3: Create order (depends on payment)
    const order = await firstValueFrom(
      this.ordersClient.send('orders.create', {
        items: checkoutData.items,
        paymentId: payment.id,
        userId: checkoutData.userId,
      }).pipe(
        timeout(5000),
      ),
    );
    
    return order;
  } catch (error) {
    throw error;
  }
}
```

**9. With Fallback Value:**

```typescript
import { firstValueFrom, catchError, of } from 'rxjs';

@Get(':id/recommendations')
async getRecommendations(@Param('id') userId: string) {
  // ✅ Return fallback if service fails
  const recommendations = await firstValueFrom(
    this.recommendationsClient.send('recommendations.get', { userId }).pipe(
      timeout(3000),
      catchError(error => {
        console.warn('Recommendations service failed, using fallback');
        // Return default recommendations
        return of([]);
      }),
    ),
  );
  
  return recommendations;
}
```

**10. Conditional Requests:**

```typescript
@Get(':id/data')
async getUserData(@Param('id') userId: string, @Query('includeOrders') includeOrders: boolean) {
  // Get user data
  const user = await firstValueFrom(
    this.client.send('users.findById', { id: userId }).pipe(
      timeout(5000),
    ),
  );
  
  // Conditionally fetch orders
  if (includeOrders) {
    const orders = await firstValueFrom(
      this.ordersClient.send('orders.findByUser', { userId }).pipe(
        timeout(5000),
        catchError(() => of([])),  // Don't fail if orders fail
      ),
    );
    
    return { ...user, orders };
  }
  
  return user;
}
```

**11. Streaming Responses (Server-Sent Events):**

```typescript
import { Controller, Sse } from '@nestjs/common';
import { Observable, map } from 'rxjs';

@Controller('users')
export class UsersController {
  @Sse('updates')
  streamUpdates(): Observable<any> {
    // ✅ Stream responses directly (no conversion needed)
    return this.client.send('users.updates', {}).pipe(
      map(data => ({
        data,
        id: Date.now(),
      })),
    );
  }
}
```

**12. Complex Error Handling:**

```typescript
import { firstValueFrom, timeout, retry, catchError } from 'rxjs';
import { RpcException } from '@nestjs/microservices';

@Post()
async createOrder(@Body() orderData: any) {
  try {
    const result = await firstValueFrom(
      this.ordersClient.send('orders.create', orderData).pipe(
        timeout(10000),
        retry({
          count: 2,
          delay: (error, retryCount) => {
            // Exponential backoff: 1s, 2s, 4s
            const delay = Math.pow(2, retryCount) * 1000;
            console.log(`Retry ${retryCount} after ${delay}ms`);
            return of(null).pipe(delay(delay));
          },
        }),
        catchError(error => {
          console.error('Order creation failed:', error);
          
          // Handle RpcException from microservice
          if (error instanceof RpcException) {
            const errorData = error.getError();
            throw new BadRequestException(errorData);
          }
          
          // Handle timeout
          if (error.name === 'TimeoutError') {
            throw new RequestTimeoutException('Order service timeout');
          }
          
          // Generic error
          throw new BadGatewayException('Order service unavailable');
        }),
      ),
    );
    
    return result;
  } catch (error) {
    throw error;
  }
}
```

**13. Response Caching:**

```typescript
import { firstValueFrom, shareReplay } from 'rxjs';

@Controller('users')
export class UsersController {
  private cache$ = this.client.send('users.findAll', {}).pipe(
    shareReplay({ bufferSize: 1, refCount: true }),
  );
  
  @Get()
  async getUsers() {
    // ✅ Cached response (shared Observable)
    return await firstValueFrom(this.cache$);
  }
}
```

**Complete Example:**

```typescript
// orders.controller.ts
import { 
  Controller, 
  Get, 
  Post, 
  Body, 
  Param, 
  Inject,
  BadGatewayException,
  BadRequestException,
  RequestTimeoutException,
  Logger,
} from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { 
  firstValueFrom, 
  forkJoin, 
  timeout, 
  retry, 
  catchError, 
  map,
  of,
} from 'rxjs';

@Controller('orders')
export class OrdersController {
  private readonly logger = new Logger(OrdersController.name);
  
  constructor(
    @Inject('ORDERS_SERVICE') private ordersClient: ClientProxy,
    @Inject('INVENTORY_SERVICE') private inventoryClient: ClientProxy,
    @Inject('PAYMENT_SERVICE') private paymentClient: ClientProxy,
  ) {}
  
  // ✅ Basic response handling
  @Get()
  async findAll() {
    try {
      const orders = await firstValueFrom(
        this.ordersClient.send('orders.findAll', {}).pipe(
          timeout(5000),
          retry(2),
          catchError(error => {
            this.logger.error('Failed to fetch orders', error);
            throw new BadGatewayException('Orders service unavailable');
          }),
        ),
      );
      
      return orders;
    } catch (error) {
      throw error;
    }
  }
  
  // ✅ Response transformation
  @Get(':id')
  async findById(@Param('id') id: string) {
    try {
      const order = await firstValueFrom(
        this.ordersClient.send('orders.findById', { id }).pipe(
          timeout(5000),
          map(response => {
            // Transform response
            return {
              id: response.id,
              status: response.status,
              total: response.total,
              items: response.items.map(item => ({
                productId: item.productId,
                quantity: item.quantity,
                price: item.price,
              })),
            };
          }),
          catchError(error => {
            this.logger.error(`Failed to fetch order ${id}`, error);
            throw new BadGatewayException('Failed to fetch order');
          }),
        ),
      );
      
      return order;
    } catch (error) {
      throw error;
    }
  }
  
  // ✅ Parallel requests
  @Get(':id/details')
  async getOrderDetails(@Param('id') orderId: string) {
    try {
      const result = await firstValueFrom(
        forkJoin({
          order: this.ordersClient.send('orders.findById', { id: orderId }),
          payment: this.paymentClient.send('payment.findByOrder', { orderId }),
          shipping: this.ordersClient.send('shipping.findByOrder', { orderId }),
        }).pipe(
          timeout(10000),
          catchError(error => {
            this.logger.error('Failed to fetch order details', error);
            throw new BadGatewayException('Failed to fetch order details');
          }),
        ),
      );
      
      return {
        order: result.order,
        payment: result.payment,
        shipping: result.shipping,
      };
    } catch (error) {
      throw error;
    }
  }
  
  // ✅ Sequential requests with error handling
  @Post()
  async createOrder(@Body() orderData: any) {
    try {
      // Step 1: Validate inventory
      this.logger.log('Validating inventory...');
      const inventory = await firstValueFrom(
        this.inventoryClient.send('inventory.validate', orderData.items).pipe(
          timeout(5000),
          retry(1),
        ),
      );
      
      if (!inventory.available) {
        throw new BadRequestException('Items not available');
      }
      
      // Step 2: Process payment
      this.logger.log('Processing payment...');
      const payment = await firstValueFrom(
        this.paymentClient.send('payment.process', {
          amount: orderData.total,
          method: orderData.paymentMethod,
        }).pipe(
          timeout(15000),  // Longer timeout for payment
          retry(1),
        ),
      );
      
      if (!payment.success) {
        throw new BadRequestException('Payment failed');
      }
      
      // Step 3: Create order
      this.logger.log('Creating order...');
      const order = await firstValueFrom(
        this.ordersClient.send('orders.create', {
          ...orderData,
          paymentId: payment.id,
        }).pipe(
          timeout(5000),
        ),
      );
      
      return order;
    } catch (error) {
      this.logger.error('Order creation failed', error);
      
      if (error instanceof BadRequestException) {
        throw error;
      }
      
      if (error.name === 'TimeoutError') {
        throw new RequestTimeoutException('Service timeout');
      }
      
      throw new BadGatewayException('Order service unavailable');
    }
  }
}
```

**Best Practices:**

```typescript
// ✅ GOOD: Always use timeout
firstValueFrom(
  this.client.send('pattern', data).pipe(
    timeout(5000),
  ),
)

// ✅ GOOD: Handle errors explicitly
firstValueFrom(
  this.client.send('pattern', data).pipe(
    timeout(5000),
    catchError(error => {
      console.error('Error:', error);
      throw new BadGatewayException();
    }),
  ),
)

// ✅ GOOD: Use retry for transient failures
firstValueFrom(
  this.client.send('pattern', data).pipe(
    timeout(5000),
    retry(2),
  ),
)

// ✅ GOOD: Transform responses
firstValueFrom(
  this.client.send('pattern', data).pipe(
    map(response => transformResponse(response)),
  ),
)

// ✅ GOOD: Use forkJoin for parallel requests
firstValueFrom(
  forkJoin({
    data1: this.client1.send('pattern1', {}),
    data2: this.client2.send('pattern2', {}),
  }),
)

// ❌ BAD: No timeout
firstValueFrom(
  this.client.send('pattern', data),  // Can hang forever!
)

// ❌ BAD: No error handling
firstValueFrom(
  this.client.send('pattern', data),  // Unhandled errors!
)

// ❌ BAD: Using deprecated toPromise()
this.client.send('pattern', data).toPromise();  // Use firstValueFrom()!

// ❌ BAD: Not converting Observable
return this.client.send('pattern', data);  // Returns Observable, not data!
```

**Interview Tip**: Handle microservice responses by **converting Observables to Promises** using **`firstValueFrom()`**. Always apply **`timeout()`** to prevent hanging, **`retry()`** for transient failures, **`catchError()`** for error handling, **`map()`** for transformation. Use **`forkJoin()`** for parallel requests, sequential `await` for dependent requests. Common pattern: `firstValueFrom(client.send().pipe(timeout(5000), retry(2), catchError()))`. Never use deprecated **`toPromise()`**. Always handle **TimeoutError**, **RpcException**, and generic errors. Use **`tap()`** for side effects, **`shareReplay()`** for caching. Best practices: **always timeout**, **handle all errors**, **transform responses**, **log failures**.

</details>

### 26. What is the return type of `send()` method (Observable)?

<details>
<summary>Answer</summary>

**`client.send()`** returns an **Observable** from **RxJS**. This allows for powerful composition, transformation, and error handling using **RxJS operators**.

**Return Type:**

```typescript
client.send<TResult>(pattern: any, data: any): Observable<TResult>
```

**Why Observable?**

```
✅ Asynchronous by nature (microservices are async)
✅ Composable (pipe multiple operators)
✅ Lazy evaluation (only executes when subscribed)
✅ Cancellable (unsubscribe to cancel)
✅ Rich operator ecosystem (map, filter, retry, etc.)
✅ Error handling (catchError)
✅ Timeout support
✅ Retry logic
✅ Stream support (multiple values)
```

**Basic Observable Usage:**

```typescript
import { Observable } from 'rxjs';
import { ClientProxy } from '@nestjs/microservices';

@Controller('users')
export class UsersController {
  constructor(
    @Inject('USERS_SERVICE') private client: ClientProxy,
  ) {}
  
  // ❌ BAD: Returns Observable (not the actual data!)
  @Get()
  getUsers(): Observable<any> {
    return this.client.send('users.findAll', {});
    // Client receives Observable, not data!
  }
  
  // ✅ GOOD: Convert Observable to Promise
  @Get()
  async getUsers(): Promise<any> {
    const result = await firstValueFrom(
      this.client.send('users.findAll', {}),
    );
    return result;  // Returns actual data
  }
}
```

**Converting Observable to Promise:**

```typescript
import { firstValueFrom, lastValueFrom } from 'rxjs';

// ✅ firstValueFrom - Get first emitted value
const result = await firstValueFrom(
  this.client.send('pattern', data),
);

// ✅ lastValueFrom - Get last emitted value (when stream completes)
const result = await lastValueFrom(
  this.client.send('pattern', data),
);

// ❌ DEPRECATED: toPromise() (don't use)
const result = await this.client.send('pattern', data).toPromise();
```

**RxJS Operators with Observable:**

```typescript
import { 
  firstValueFrom, 
  map, 
  filter, 
  tap, 
  timeout, 
  retry, 
  catchError,
  switchMap,
  mergeMap,
} from 'rxjs';

// ✅ map - Transform response
const users = await firstValueFrom(
  this.client.send('users.findAll', {}).pipe(
    map(response => response.data),  // Extract data field
  ),
);

// ✅ filter - Filter response
const activeUsers = await firstValueFrom(
  this.client.send('users.findAll', {}).pipe(
    map(response => response.users),
    map(users => users.filter(u => u.active)),
  ),
);

// ✅ tap - Side effects (logging, etc.)
const result = await firstValueFrom(
  this.client.send('users.create', data).pipe(
    tap(response => console.log('User created:', response)),
  ),
);

// ✅ timeout - Timeout after N milliseconds
const result = await firstValueFrom(
  this.client.send('pattern', data).pipe(
    timeout(5000),  // Timeout after 5s
  ),
);

// ✅ retry - Retry on failure
const result = await firstValueFrom(
  this.client.send('pattern', data).pipe(
    retry(3),  // Retry 3 times
  ),
);

// ✅ catchError - Error handling
const result = await firstValueFrom(
  this.client.send('pattern', data).pipe(
    catchError(error => {
      console.error('Error:', error);
      throw new BadGatewayException();
    }),
  ),
);
```

**Chaining Multiple Operators:**

```typescript
import { firstValueFrom, timeout, retry, map, catchError } from 'rxjs';

@Get(':id')
async getUser(@Param('id') id: string) {
  const user = await firstValueFrom(
    this.client.send('users.findById', { id }).pipe(
      // 1. Timeout after 5s
      timeout(5000),
      
      // 2. Retry 2 times on failure
      retry(2),
      
      // 3. Transform response
      map(response => ({
        id: response.id,
        name: response.name,
        email: response.email,
      })),
      
      // 4. Handle errors
      catchError(error => {
        console.error('Failed to fetch user:', error);
        throw new BadGatewayException('User service unavailable');
      }),
    ),
  );
  
  return user;
}
```

**Combining Multiple Observables:**

```typescript
import { forkJoin, combineLatest, merge, concat } from 'rxjs';

// ✅ forkJoin - Execute in parallel, wait for all
const result = await firstValueFrom(
  forkJoin({
    users: this.usersClient.send('users.findAll', {}),
    orders: this.ordersClient.send('orders.findAll', {}),
    products: this.productsClient.send('products.findAll', {}),
  }),
);
// result = { users: [...], orders: [...], products: [...] }

// ✅ combineLatest - Emit whenever any Observable emits
const combined$ = combineLatest([
  this.client1.send('pattern1', {}),
  this.client2.send('pattern2', {}),
]);

// ✅ merge - Merge multiple Observables into one
const merged$ = merge(
  this.client1.send('pattern1', {}),
  this.client2.send('pattern2', {}),
);

// ✅ concat - Execute sequentially
const sequential$ = concat(
  this.client1.send('pattern1', {}),
  this.client2.send('pattern2', {}),
);
```

**Observable Chaining (Dependent Requests):**

```typescript
import { switchMap, mergeMap } from 'rxjs';

// ✅ switchMap - Switch to new Observable (cancels previous)
const orderDetails = await firstValueFrom(
  this.usersClient.send('users.findById', { id: userId }).pipe(
    switchMap(user => {
      // Use user data to fetch orders
      return this.ordersClient.send('orders.findByUser', { userId: user.id });
    }),
  ),
);

// ✅ mergeMap - Map to Observable (doesn't cancel)
const results = await firstValueFrom(
  this.client.send('users.findAll', {}).pipe(
    mergeMap(users => {
      // Fetch orders for each user
      const requests = users.map(user => 
        this.ordersClient.send('orders.findByUser', { userId: user.id }),
      );
      return forkJoin(requests);
    }),
  ),
);
```

**Typed Observables:**

```typescript
interface User {
  id: string;
  name: string;
  email: string;
}

interface Order {
  id: string;
  userId: string;
  total: number;
}

@Controller('users')
export class UsersController {
  constructor(
    @Inject('USERS_SERVICE') private client: ClientProxy,
  ) {}
  
  // ✅ Typed Observable
  @Get(':id')
  async getUser(@Param('id') id: string): Promise<User> {
    const user = await firstValueFrom(
      this.client.send<User>('users.findById', { id }),  // ✅ Type hint
    );
    
    return user;  // TypeScript knows it's a User
  }
  
  // ✅ Typed parallel requests
  @Get(':id/data')
  async getUserData(@Param('id') userId: string) {
    const result = await firstValueFrom(
      forkJoin({
        user: this.client.send<User>('users.findById', { id: userId }),
        orders: this.ordersClient.send<Order[]>('orders.findByUser', { userId }),
      }),
    );
    
    return result;  // { user: User, orders: Order[] }
  }
}
```

**Observable Subscription (Manual):**

```typescript
// ⚠️ Manual subscription (less common in NestJS)
@Get()
getUsers() {
  this.client.send('users.findAll', {}).subscribe({
    next: (data) => {
      console.log('Received:', data);
    },
    error: (error) => {
      console.error('Error:', error);
    },
    complete: () => {
      console.log('Completed');
    },
  });
}

// ⚠️ Remember to unsubscribe!
@OnDestroy()
ngOnDestroy() {
  this.subscription?.unsubscribe();
}
```

**Observable vs Promise:**

```typescript
// Observable (RxJS)
✅ Lazy (only executes when subscribed)
✅ Cancellable (unsubscribe)
✅ Composable (operators)
✅ Can emit multiple values
✅ Rich operator ecosystem
❌ More complex API
❌ Learning curve

// Promise
✅ Simple API (async/await)
✅ Built into JavaScript
✅ Easy to understand
❌ Not cancellable
❌ Single value only
❌ Limited composition
❌ Eager evaluation
```

**Why NestJS Uses Observable:**

```
✅ Microservices are asynchronous
✅ Need timeout/retry capabilities
✅ Error handling requirements
✅ Response transformation
✅ Composable operations
✅ Stream support (gRPC, WebSockets)
✅ Consistent API across transports
✅ Integration with reactive systems
```

**Common Patterns:**

```typescript
// Pattern 1: Basic request-response
const result = await firstValueFrom(
  this.client.send('pattern', data).pipe(
    timeout(5000),
    retry(2),
    catchError(handleError),
  ),
);

// Pattern 2: Transform response
const transformed = await firstValueFrom(
  this.client.send('pattern', data).pipe(
    map(response => response.data),
    timeout(5000),
  ),
);

// Pattern 3: Parallel requests
const parallel = await firstValueFrom(
  forkJoin({
    data1: this.client1.send('pattern1', {}),
    data2: this.client2.send('pattern2', {}),
  }).pipe(
    timeout(10000),
  ),
);

// Pattern 4: Sequential (dependent) requests
const sequential = await firstValueFrom(
  this.client.send('pattern1', data).pipe(
    switchMap(result1 => 
      this.client.send('pattern2', result1),
    ),
    timeout(10000),
  ),
);

// Pattern 5: With fallback
const withFallback = await firstValueFrom(
  this.client.send('pattern', data).pipe(
    timeout(3000),
    catchError(() => of(defaultValue)),
  ),
);
```

**Complete Example:**

```typescript
import { 
  Controller, 
  Get, 
  Post, 
  Body, 
  Param, 
  Inject 
} from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { 
  Observable,
  firstValueFrom, 
  forkJoin,
  timeout, 
  retry, 
  map, 
  catchError,
  switchMap,
  of,
} from 'rxjs';

@Controller('users')
export class UsersController {
  constructor(
    @Inject('USERS_SERVICE') private usersClient: ClientProxy,
    @Inject('ORDERS_SERVICE') private ordersClient: ClientProxy,
  ) {}
  
  // ✅ Returns Observable directly (for SSE or streaming)
  @Sse('updates')
  streamUpdates(): Observable<any> {
    // Don't convert to Promise for streaming
    return this.usersClient.send('users.updates', {}).pipe(
      map(data => ({ data, id: Date.now() })),
    );
  }
  
  // ✅ Convert Observable to Promise
  @Get()
  async getUsers(): Promise<any> {
    return await firstValueFrom(
      this.usersClient.send('users.findAll', {}).pipe(
        timeout(5000),
        retry(2),
        map(response => response.data),
        catchError(error => {
          throw new BadGatewayException('Service unavailable');
        }),
      ),
    );
  }
  
  // ✅ Parallel requests with forkJoin
  @Get(':id/profile')
  async getProfile(@Param('id') userId: string): Promise<any> {
    return await firstValueFrom(
      forkJoin({
        user: this.usersClient.send('users.findById', { id: userId }),
        orders: this.ordersClient.send('orders.findByUser', { userId }),
      }).pipe(
        timeout(10000),
        map(result => ({
          ...result.user,
          recentOrders: result.orders.slice(0, 5),
        })),
      ),
    );
  }
  
  // ✅ Sequential requests with switchMap
  @Post('orders')
  async createOrder(@Body() orderData: any): Promise<any> {
    return await firstValueFrom(
      this.usersClient.send('users.findById', { id: orderData.userId }).pipe(
        switchMap(user => {
          // Create order with user data
          return this.ordersClient.send('orders.create', {
            ...orderData,
            userEmail: user.email,
          });
        }),
        timeout(10000),
        retry(1),
      ),
    );
  }
}
```

**Best Practices:**

```typescript
// ✅ GOOD: Always convert Observable to Promise in controllers
async getUsers() {
  return await firstValueFrom(this.client.send('pattern', {}));
}

// ✅ GOOD: Add timeout
firstValueFrom(
  this.client.send('pattern', {}).pipe(timeout(5000)),
)

// ✅ GOOD: Chain operators
firstValueFrom(
  this.client.send('pattern', {}).pipe(
    timeout(5000),
    retry(2),
    map(transformResponse),
    catchError(handleError),
  ),
)

// ✅ GOOD: Type your Observables
this.client.send<User>('users.findById', { id })

// ❌ BAD: Return Observable from controller
getUsers(): Observable<any> {
  return this.client.send('pattern', {});  // Don't do this!
}

// ❌ BAD: Use deprecated toPromise()
this.client.send('pattern', {}).toPromise();  // Use firstValueFrom()!

// ❌ BAD: Manual subscription without cleanup
this.client.send('pattern', {}).subscribe(data => {
  console.log(data);
});  // Memory leak!
```

**Interview Tip**: **`client.send()`** returns an **Observable** from **RxJS**, not a Promise. Observables are **lazy** (execute on subscription), **cancellable**, **composable** with operators, and can emit **multiple values**. Convert to Promise using **`firstValueFrom()`** (first value) or **`lastValueFrom()`** (last value). Never use deprecated **`toPromise()`**. Use RxJS operators: **`timeout()`**, **`retry()`**, **`catchError()`**, **`map()`**, **`tap()`**. Combine Observables with **`forkJoin()`** (parallel), **`switchMap()`** (sequential). Always **convert to Promise** in controllers unless streaming (SSE). Observable benefits: **composable**, **powerful operators**, **timeout/retry support**, **error handling**, **stream support**. NestJS uses Observables for **flexibility** and **reactive programming** support.

</details>

### 27. How do you handle errors in request-response?

<details>
<summary>Answer</summary>

Error handling in request-response involves **catching RpcExceptions**, using **catchError operator**, implementing **try-catch blocks**, **timeout handling**, **retry logic**, and **proper error transformation**.

**Error Types in Microservices:**

```
✅ RpcException - Microservice-specific errors
✅ TimeoutError - Request timeout
✅ Network errors - Connection failures
✅ Business logic errors - Validation, authorization
✅ Service unavailable - Service down/unreachable
```

**1. Using RpcException (Server Side):**

```typescript
// Server: Throw RpcException
import { Controller } from '@nestjs/common';
import { MessagePattern, RpcException } from '@nestjs/microservices';

@Controller()
export class UsersController {
  @MessagePattern('users.findById')
  async findById(data: { id: string }) {
    try {
      const user = await this.usersService.findById(data.id);
      
      if (!user) {
        // ✅ Throw RpcException
        throw new RpcException({
          statusCode: 404,
          message: 'User not found',
          error: 'NOT_FOUND',
        });
      }
      
      return user;
    } catch (error) {
      // ✅ Transform to RpcException
      throw new RpcException({
        statusCode: 500,
        message: error.message || 'Internal server error',
        error: 'INTERNAL_ERROR',
      });
    }
  }
  
  @MessagePattern('users.create')
  async create(data: any) {
    try {
      // Validate data
      if (!data.email) {
        throw new RpcException({
          statusCode: 400,
          message: 'Email is required',
          error: 'VALIDATION_ERROR',
        });
      }
      
      // Check if user exists
      const existingUser = await this.usersService.findByEmail(data.email);
      if (existingUser) {
        throw new RpcException({
          statusCode: 409,
          message: 'User already exists',
          error: 'DUPLICATE_USER',
        });
      }
      
      return await this.usersService.create(data);
    } catch (error) {
      if (error instanceof RpcException) {
        throw error;
      }
      
      throw new RpcException({
        statusCode: 500,
        message: 'Failed to create user',
        error: 'CREATE_ERROR',
      });
    }
  }
}
```

**2. Catching Errors (Client Side):**

```typescript
// Client: Handle RpcException
import { 
  Controller, 
  Get, 
  Param, 
  Inject,
  NotFoundException,
  BadRequestException,
  ConflictException,
  BadGatewayException,
} from '@nestjs/common';
import { ClientProxy, RpcException } from '@nestjs/microservices';
import { firstValueFrom, catchError } from 'rxjs';

@Controller('users')
export class UsersController {
  constructor(
    @Inject('USERS_SERVICE') private client: ClientProxy,
  ) {}
  
  @Get(':id')
  async getUser(@Param('id') id: string) {
    try {
      const user = await firstValueFrom(
        this.client.send('users.findById', { id }).pipe(
          catchError(error => {
            // ✅ Handle RpcException
            if (error instanceof RpcException) {
              const errorData: any = error.getError();
              
              // Transform to HTTP exceptions
              if (errorData.statusCode === 404) {
                throw new NotFoundException(errorData.message);
              }
              if (errorData.statusCode === 400) {
                throw new BadRequestException(errorData.message);
              }
              if (errorData.statusCode === 409) {
                throw new ConflictException(errorData.message);
              }
            }
            
            // Generic error
            throw new BadGatewayException('User service unavailable');
          }),
        ),
      );
      
      return user;
    } catch (error) {
      throw error;
    }
  }
}
```

**3. Try-Catch Error Handling:**

```typescript
@Post()
async createUser(@Body() userData: any) {
  try {
    const result = await firstValueFrom(
      this.client.send('users.create', userData),
    );
    
    return result;
  } catch (error) {
    // ✅ Handle specific errors
    if (error instanceof RpcException) {
      const errorData: any = error.getError();
      
      switch (errorData.error) {
        case 'VALIDATION_ERROR':
          throw new BadRequestException(errorData.message);
        case 'DUPLICATE_USER':
          throw new ConflictException(errorData.message);
        case 'INTERNAL_ERROR':
          throw new InternalServerErrorException(errorData.message);
        default:
          throw new BadGatewayException('Service error');
      }
    }
    
    // Network/connection errors
    throw new BadGatewayException('User service unavailable');
  }
}
```

**4. Timeout Error Handling:**

```typescript
import { timeout, catchError } from 'rxjs';
import { TimeoutError } from 'rxjs';

@Get(':id')
async getUser(@Param('id') id: string) {
  try {
    const user = await firstValueFrom(
      this.client.send('users.findById', { id }).pipe(
        timeout(5000),  // 5 second timeout
        catchError(error => {
          // ✅ Handle timeout specifically
          if (error instanceof TimeoutError || error.name === 'TimeoutError') {
            console.error('Request timeout for user:', id);
            throw new RequestTimeoutException('User service timeout');
          }
          
          // Handle RpcException
          if (error instanceof RpcException) {
            const errorData: any = error.getError();
            throw new NotFoundException(errorData.message);
          }
          
          // Generic error
          throw new BadGatewayException('User service error');
        }),
      ),
    );
    
    return user;
  } catch (error) {
    throw error;
  }
}
```

**5. Retry with Error Handling:**

```typescript
import { timeout, retry, catchError } from 'rxjs';

@Get(':id')
async getUser(@Param('id') id: string) {
  try {
    const user = await firstValueFrom(
      this.client.send('users.findById', { id }).pipe(
        timeout(5000),
        
        // ✅ Retry only on network errors (not business logic errors)
        retry({
          count: 3,
          delay: (error, retryCount) => {
            // Don't retry on RpcException (business logic errors)
            if (error instanceof RpcException) {
              throw error;
            }
            
            // Retry on network/timeout errors
            console.log(`Retry ${retryCount} for user ${id}`);
            return of(null).pipe(delay(1000 * retryCount));
          },
        }),
        
        catchError(error => {
          if (error instanceof RpcException) {
            const errorData: any = error.getError();
            if (errorData.statusCode === 404) {
              throw new NotFoundException(errorData.message);
            }
          }
          
          if (error instanceof TimeoutError) {
            throw new RequestTimeoutException('Service timeout');
          }
          
          throw new BadGatewayException('Service unavailable');
        }),
      ),
    );
    
    return user;
  } catch (error) {
    throw error;
  }
}
```

**6. Fallback Values:**

```typescript
import { catchError, of } from 'rxjs';

@Get(':id/recommendations')
async getRecommendations(@Param('id') userId: string) {
  try {
    // ✅ Return fallback on error
    const recommendations = await firstValueFrom(
      this.recommendationsClient.send('recommendations.get', { userId }).pipe(
        timeout(3000),
        catchError(error => {
          console.warn('Recommendations failed, using fallback:', error);
          // Return default recommendations
          return of([]);
        }),
      ),
    );
    
    return recommendations;
  } catch (error) {
    // This won't be reached if catchError returns of([])
    return [];
  }
}
```

**7. Logging Errors:**

```typescript
import { Logger } from '@nestjs/common';

@Controller('users')
export class UsersController {
  private readonly logger = new Logger(UsersController.name);
  
  constructor(
    @Inject('USERS_SERVICE') private client: ClientProxy,
  ) {}
  
  @Post()
  async createUser(@Body() userData: any) {
    try {
      const result = await firstValueFrom(
        this.client.send('users.create', userData).pipe(
          timeout(10000),
          catchError(error => {
            // ✅ Log error details
            this.logger.error(
              `Failed to create user: ${error.message}`,
              error.stack,
              {
                userData: userData.email,
                errorType: error.constructor.name,
              },
            );
            
            // Transform error
            if (error instanceof RpcException) {
              const errorData: any = error.getError();
              throw new BadRequestException(errorData.message);
            }
            
            throw new BadGatewayException('User service error');
          }),
        ),
      );
      
      this.logger.log(`User created successfully: ${result.id}`);
      return result;
    } catch (error) {
      throw error;
    }
  }
}
```

**8. Custom Error Filter (Global):**

```typescript
// rpc-exception.filter.ts
import { 
  ExceptionFilter, 
  Catch, 
  ArgumentsHost,
  HttpStatus,
} from '@nestjs/common';
import { RpcException } from '@nestjs/microservices';
import { Response } from 'express';

@Catch(RpcException)
export class RpcExceptionFilter implements ExceptionFilter {
  catch(exception: RpcException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const error: any = exception.getError();
    
    // ✅ Map RPC errors to HTTP status codes
    const statusCode = error.statusCode || HttpStatus.INTERNAL_SERVER_ERROR;
    const message = error.message || 'Internal server error';
    
    response.status(statusCode).json({
      statusCode,
      message,
      error: error.error || 'INTERNAL_ERROR',
      timestamp: new Date().toISOString(),
    });
  }
}

// main.ts - Register globally
app.useGlobalFilters(new RpcExceptionFilter());
```

**9. Circuit Breaker Pattern:**

```typescript
import { Injectable } from '@nestjs/common';
import { firstValueFrom, timeout, catchError, of } from 'rxjs';

@Injectable()
export class CircuitBreakerService {
  private failures = 0;
  private readonly threshold = 5;
  private readonly resetTime = 60000; // 1 minute
  private state: 'CLOSED' | 'OPEN' | 'HALF_OPEN' = 'CLOSED';
  private lastFailureTime: number;
  
  async executeWithCircuitBreaker<T>(
    observable: Observable<T>,
  ): Promise<T> {
    // ✅ Circuit is OPEN - fail fast
    if (this.state === 'OPEN') {
      if (Date.now() - this.lastFailureTime > this.resetTime) {
        this.state = 'HALF_OPEN';
      } else {
        throw new BadGatewayException('Circuit breaker is OPEN');
      }
    }
    
    try {
      const result = await firstValueFrom(
        observable.pipe(
          timeout(5000),
        ),
      );
      
      // ✅ Success - reset failures
      this.failures = 0;
      if (this.state === 'HALF_OPEN') {
        this.state = 'CLOSED';
      }
      
      return result;
    } catch (error) {
      // ✅ Failure - increment counter
      this.failures++;
      this.lastFailureTime = Date.now();
      
      if (this.failures >= this.threshold) {
        this.state = 'OPEN';
        console.error('Circuit breaker opened!');
      }
      
      throw error;
    }
  }
}

// Usage
@Controller('users')
export class UsersController {
  constructor(
    @Inject('USERS_SERVICE') private client: ClientProxy,
    private circuitBreaker: CircuitBreakerService,
  ) {}
  
  @Get(':id')
  async getUser(@Param('id') id: string) {
    try {
      const user = await this.circuitBreaker.executeWithCircuitBreaker(
        this.client.send('users.findById', { id }),
      );
      
      return user;
    } catch (error) {
      throw new BadGatewayException('User service unavailable');
    }
  }
}
```

**10. Error Monitoring/Alerting:**

```typescript
import { Injectable, Logger } from '@nestjs/common';

@Injectable()
export class ErrorMonitoringService {
  private readonly logger = new Logger(ErrorMonitoringService.name);
  private errorCounts = new Map<string, number>();
  
  logMicroserviceError(
    service: string,
    operation: string,
    error: any,
  ) {
    const key = `${service}:${operation}`;
    const count = (this.errorCounts.get(key) || 0) + 1;
    this.errorCounts.set(key, count);
    
    // ✅ Log error
    this.logger.error(
      `Microservice error: ${key}`,
      error.stack,
      {
        service,
        operation,
        errorType: error.constructor.name,
        count,
      },
    );
    
    // ✅ Alert if threshold exceeded
    if (count >= 10) {
      this.sendAlert(service, operation, count);
    }
  }
  
  private sendAlert(service: string, operation: string, count: number) {
    // Send to monitoring service (e.g., Sentry, DataDog)
    console.error(`ALERT: ${service}.${operation} failed ${count} times!`);
  }
}
```

**Complete Example:**

```typescript
// users.controller.ts
import {
  Controller,
  Get,
  Post,
  Put,
  Delete,
  Body,
  Param,
  Inject,
  Logger,
  NotFoundException,
  BadRequestException,
  ConflictException,
  RequestTimeoutException,
  BadGatewayException,
  InternalServerErrorException,
} from '@nestjs/common';
import { ClientProxy, RpcException } from '@nestjs/microservices';
import { firstValueFrom, timeout, retry, catchError, of, delay } from 'rxjs';
import { TimeoutError } from 'rxjs';

@Controller('users')
export class UsersController {
  private readonly logger = new Logger(UsersController.name);
  
  constructor(
    @Inject('USERS_SERVICE') private client: ClientProxy,
  ) {}
  
  // ✅ GET with comprehensive error handling
  @Get(':id')
  async findById(@Param('id') id: string) {
    try {
      const user = await firstValueFrom(
        this.client.send('users.findById', { id }).pipe(
          timeout(5000),
          retry({
            count: 2,
            delay: (error, retryCount) => {
              // Don't retry business logic errors
              if (error instanceof RpcException) {
                throw error;
              }
              this.logger.warn(`Retrying findById (${retryCount})`);
              return of(null).pipe(delay(1000));
            },
          }),
          catchError(error => {
            this.logger.error(
              `Failed to fetch user ${id}`,
              error.stack,
            );
            
            // Handle RpcException
            if (error instanceof RpcException) {
              const errorData: any = error.getError();
              if (errorData.statusCode === 404) {
                throw new NotFoundException(errorData.message);
              }
              if (errorData.statusCode === 400) {
                throw new BadRequestException(errorData.message);
              }
            }
            
            // Handle timeout
            if (error instanceof TimeoutError || error.name === 'TimeoutError') {
              throw new RequestTimeoutException('User service timeout');
            }
            
            // Generic error
            throw new BadGatewayException('User service unavailable');
          }),
        ),
      );
      
      return user;
    } catch (error) {
      throw error;
    }
  }
  
  // ✅ POST with validation and error handling
  @Post()
  async create(@Body() userData: any) {
    try {
      const result = await firstValueFrom(
        this.client.send('users.create', userData).pipe(
          timeout(10000),
          catchError(error => {
            this.logger.error('Failed to create user', error.stack);
            
            if (error instanceof RpcException) {
              const errorData: any = error.getError();
              
              switch (errorData.error) {
                case 'VALIDATION_ERROR':
                  throw new BadRequestException(errorData.message);
                case 'DUPLICATE_USER':
                  throw new ConflictException(errorData.message);
                case 'INTERNAL_ERROR':
                  throw new InternalServerErrorException(errorData.message);
                default:
                  throw new BadGatewayException('Service error');
              }
            }
            
            if (error instanceof TimeoutError) {
              throw new RequestTimeoutException('Create user timeout');
            }
            
            throw new BadGatewayException('User service unavailable');
          }),
        ),
      );
      
      this.logger.log(`User created: ${result.id}`);
      return result;
    } catch (error) {
      throw error;
    }
  }
  
  // ✅ DELETE with fallback
  @Delete(':id')
  async delete(@Param('id') id: string) {
    try {
      const result = await firstValueFrom(
        this.client.send('users.delete', { id }).pipe(
          timeout(5000),
          catchError(error => {
            this.logger.error(`Failed to delete user ${id}`, error.stack);
            
            if (error instanceof RpcException) {
              const errorData: any = error.getError();
              if (errorData.statusCode === 404) {
                // ✅ User already deleted - return success
                return of({ success: true, alreadyDeleted: true });
              }
            }
            
            throw new BadGatewayException('Delete failed');
          }),
        ),
      );
      
      return result;
    } catch (error) {
      throw error;
    }
  }
}
```

**Best Practices:**

```typescript
// ✅ GOOD: Use RpcException on server
throw new RpcException({
  statusCode: 404,
  message: 'User not found',
  error: 'NOT_FOUND',
});

// ✅ GOOD: Catch and transform on client
catchError(error => {
  if (error instanceof RpcException) {
    const errorData: any = error.getError();
    throw new NotFoundException(errorData.message);
  }
  throw new BadGatewayException();
})

// ✅ GOOD: Handle timeout errors
if (error instanceof TimeoutError) {
  throw new RequestTimeoutException();
}

// ✅ GOOD: Don't retry business logic errors
retry({
  delay: (error) => {
    if (error instanceof RpcException) {
      throw error;  // Don't retry
    }
    return of(null).pipe(delay(1000));
  },
})

// ✅ GOOD: Log all errors
catchError(error => {
  this.logger.error('Service error', error.stack);
  throw error;
})

// ❌ BAD: Generic error handling
catchError(error => {
  throw new Error('Something failed');  // Not helpful!
})

// ❌ BAD: Swallow errors
catchError(error => {
  console.log(error);
  return of(null);  // Don't hide errors!
})

// ❌ BAD: No timeout
this.client.send('pattern', data)  // Can hang forever!
```

**Interview Tip**: Handle errors in request-response using **RpcException** (server-side), **catchError operator** (client-side), **try-catch blocks**, and **timeout handling**. Server throws **RpcException** with structured errors (statusCode, message, error code). Client catches with **catchError**, transforms to **HTTP exceptions** (NotFoundException, BadRequestException, etc.), handles **TimeoutError** separately. Always add **timeout()**, use **retry()** only for network errors (not business logic), **log all errors**, implement **circuit breaker** for resilience. Don't retry **RpcException** (business logic errors). Use **fallback values** for non-critical services. Best practices: **structured errors**, **proper logging**, **transform errors**, **handle timeouts**, **don't swallow errors**.

</details>

## Event-Based Communication

### 28. How do you emit events using `client.emit()`?

<details>
<summary>Answer</summary>

**`client.emit()`** is used for **event-based** (fire-and-forget) communication in NestJS microservices. It sends an event **without waiting for a response**.

**Basic Syntax:**

```typescript
client.emit(pattern, data): Observable<any>
```

**Key Characteristics:**

```
✅ Fire-and-forget (no response expected)
✅ Asynchronous (non-blocking)
✅ One-to-many (multiple handlers can listen)
✅ Event-driven architecture
✅ Decoupled services
✅ Returns Observable (but doesn't wait)
✅ Broadcast to all subscribers
✅ No acknowledgment
```

**Basic Usage:**

```typescript
import { Controller, Post, Body, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';

@Controller('users')
export class UsersController {
  constructor(
    @Inject('EVENTS_SERVICE') private client: ClientProxy,
  ) {}
  
  @Post()
  async createUser(@Body() userData: any) {
    // Create user in database
    const user = await this.usersService.create(userData);
    
    // ✅ Emit event (fire-and-forget)
    this.client.emit('user.created', {
      id: user.id,
      email: user.email,
      name: user.name,
      createdAt: new Date(),
    });
    
    // Returns immediately (doesn't wait for event handlers)
    return user;
  }
}
```

**Event Patterns:**

```typescript
// ✅ String patterns (dot notation)
this.client.emit('user.created', data);
this.client.emit('user.updated', data);
this.client.emit('user.deleted', data);

// ✅ Hierarchical patterns
this.client.emit('order.payment.completed', data);
this.client.emit('order.shipping.started', data);

// ✅ Object patterns
this.client.emit({ event: 'user.created' }, data);
this.client.emit({ event: 'order.created', version: 'v1' }, data);
```

**Multiple Event Handlers (Broadcast):**

```typescript
// ===== Client (API Gateway) =====
@Controller('orders')
export class OrdersController {
  constructor(
    @Inject('EVENTS_SERVICE') private client: ClientProxy,
  ) {}
  
  @Post()
  async createOrder(@Body() orderData: any) {
    const order = await this.ordersService.create(orderData);
    
    // ✅ Emit event - ALL services listening will receive it
    this.client.emit('order.created', {
      orderId: order.id,
      userId: order.userId,
      total: order.total,
      items: order.items,
      timestamp: new Date(),
    });
    
    return order;
  }
}

// ===== Server 1 (Notifications Service) =====
@Controller()
export class NotificationsController {
  @EventPattern('order.created')
  async handleOrderCreated(@Payload() data: any) {
    console.log('Notifications: Order created', data);
    // Send order confirmation email
    await this.sendOrderConfirmation(data);
  }
}

// ===== Server 2 (Analytics Service) =====
@Controller()
export class AnalyticsController {
  @EventPattern('order.created')
  async trackOrderCreated(@Payload() data: any) {
    console.log('Analytics: Order created', data);
    // Track order in analytics
    await this.trackEvent('order_created', data);
  }
}

// ===== Server 3 (Inventory Service) =====
@Controller()
export class InventoryController {
  @EventPattern('order.created')
  async updateInventory(@Payload() data: any) {
    console.log('Inventory: Order created', data);
    // Decrease inventory
    await this.decreaseStock(data.items);
  }
}

// All three services receive the event!
```

**Event Emission Examples:**

```typescript
// Example 1: User lifecycle events
@Controller('users')
export class UsersController {
  constructor(
    @Inject('EVENTS_SERVICE') private client: ClientProxy,
  ) {}
  
  @Post()
  async createUser(@Body() userData: any) {
    const user = await this.usersService.create(userData);
    
    // ✅ Emit user.created event
    this.client.emit('user.created', {
      id: user.id,
      email: user.email,
      name: user.name,
    });
    
    return user;
  }
  
  @Put(':id')
  async updateUser(@Param('id') id: string, @Body() updateData: any) {
    const user = await this.usersService.update(id, updateData);
    
    // ✅ Emit user.updated event
    this.client.emit('user.updated', {
      id: user.id,
      changes: updateData,
      updatedAt: new Date(),
    });
    
    return user;
  }
  
  @Delete(':id')
  async deleteUser(@Param('id') id: string) {
    await this.usersService.delete(id);
    
    // ✅ Emit user.deleted event
    this.client.emit('user.deleted', {
      id,
      deletedAt: new Date(),
    });
    
    return { success: true };
  }
}

// Example 2: Order workflow events
@Controller('orders')
export class OrdersController {
  constructor(
    @Inject('EVENTS_SERVICE') private client: ClientProxy,
  ) {}
  
  @Post()
  async createOrder(@Body() orderData: any) {
    const order = await this.ordersService.create(orderData);
    this.client.emit('order.placed', order);
    return order;
  }
  
  @Post(':id/payment')
  async processPayment(@Param('id') orderId: string, @Body() paymentData: any) {
    const payment = await this.paymentService.process(orderId, paymentData);
    this.client.emit('order.payment.completed', { orderId, payment });
    return payment;
  }
  
  @Post(':id/ship')
  async shipOrder(@Param('id') orderId: string) {
    const shipment = await this.shippingService.ship(orderId);
    this.client.emit('order.shipped', { orderId, shipment });
    return shipment;
  }
  
  @Post(':id/deliver')
  async deliverOrder(@Param('id') orderId: string) {
    await this.ordersService.markDelivered(orderId);
    this.client.emit('order.delivered', { orderId, deliveredAt: new Date() });
    return { success: true };
  }
}
```

**Event Handlers (Server Side):**

```typescript
// notifications.controller.ts
import { Controller } from '@nestjs/common';
import { EventPattern, Payload, Ctx, RmqContext } from '@nestjs/microservices';

@Controller()
export class NotificationsController {
  // ✅ Listen to user.created event
  @EventPattern('user.created')
  async handleUserCreated(@Payload() data: any) {
    console.log('User created:', data);
    // Send welcome email
    await this.emailService.sendWelcomeEmail(data.email);
  }
  
  // ✅ Listen to order.placed event
  @EventPattern('order.placed')
  async handleOrderPlaced(@Payload() data: any) {
    console.log('Order placed:', data);
    // Send order confirmation
    await this.emailService.sendOrderConfirmation(data);
  }
  
  // ✅ Listen to order.shipped event
  @EventPattern('order.shipped')
  async handleOrderShipped(@Payload() data: any) {
    console.log('Order shipped:', data);
    // Send shipping notification
    await this.emailService.sendShippingNotification(data);
  }
  
  // ✅ Access context (for message broker specific info)
  @EventPattern('user.updated')
  async handleUserUpdated(@Payload() data: any, @Ctx() context: RmqContext) {
    const message = context.getMessage();
    const channel = context.getChannelRef();
    
    console.log('User updated:', data);
    await this.processUserUpdate(data);
    
    // Acknowledge message (if using RabbitMQ)
    channel.ack(message);
  }
}
```

**Complete Example (E-commerce):**

```typescript
// ===== API Gateway =====
// app.module.ts
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'EVENTS_BUS',
        transport: Transport.REDIS,  // Using Redis for Pub/Sub
        options: {
          host: 'localhost',
          port: 6379,
        },
      },
    ]),
  ],
})
export class AppModule {}

// orders.controller.ts
@Controller('orders')
export class OrdersController {
  constructor(
    @Inject('EVENTS_BUS') private eventBus: ClientProxy,
  ) {}
  
  @Post()
  async createOrder(@Body() orderData: any) {
    // 1. Create order
    const order = await this.ordersService.create(orderData);
    
    // 2. Emit order.placed event
    this.eventBus.emit('order.placed', {
      orderId: order.id,
      userId: order.userId,
      total: order.total,
      items: order.items,
      timestamp: new Date().toISOString(),
    });
    
    return order;
  }
  
  @Post(':id/cancel')
  async cancelOrder(@Param('id') orderId: string) {
    await this.ordersService.cancel(orderId);
    
    // Emit cancellation event
    this.eventBus.emit('order.cancelled', {
      orderId,
      cancelledAt: new Date().toISOString(),
    });
    
    return { success: true };
  }
}

// ===== Notifications Service =====
@Controller()
export class NotificationsController {
  @EventPattern('order.placed')
  async onOrderPlaced(@Payload() data: any) {
    console.log('[Notifications] Order placed:', data.orderId);
    
    // Send confirmation email
    await this.emailService.send({
      to: data.userEmail,
      subject: 'Order Confirmation',
      body: `Your order ${data.orderId} has been placed.`,
    });
    
    // Send SMS
    await this.smsService.send({
      to: data.userPhone,
      message: `Order ${data.orderId} confirmed.`,
    });
  }
  
  @EventPattern('order.cancelled')
  async onOrderCancelled(@Payload() data: any) {
    console.log('[Notifications] Order cancelled:', data.orderId);
    await this.emailService.sendCancellationEmail(data);
  }
}

// ===== Analytics Service =====
@Controller()
export class AnalyticsController {
  @EventPattern('order.placed')
  async trackOrderPlaced(@Payload() data: any) {
    console.log('[Analytics] Tracking order:', data.orderId);
    
    // Track in analytics
    await this.analyticsService.track('order_placed', {
      orderId: data.orderId,
      userId: data.userId,
      total: data.total,
      itemCount: data.items.length,
    });
  }
  
  @EventPattern('order.cancelled')
  async trackOrderCancelled(@Payload() data: any) {
    console.log('[Analytics] Tracking cancellation:', data.orderId);
    await this.analyticsService.track('order_cancelled', data);
  }
}

// ===== Inventory Service =====
@Controller()
export class InventoryController {
  @EventPattern('order.placed')
  async decreaseStock(@Payload() data: any) {
    console.log('[Inventory] Decreasing stock for order:', data.orderId);
    
    // Decrease inventory for each item
    for (const item of data.items) {
      await this.inventoryService.decreaseStock(
        item.productId,
        item.quantity,
      );
    }
  }
  
  @EventPattern('order.cancelled')
  async restoreStock(@Payload() data: any) {
    console.log('[Inventory] Restoring stock for order:', data.orderId);
    
    // Get order items and restore stock
    const order = await this.ordersService.findById(data.orderId);
    for (const item of order.items) {
      await this.inventoryService.increaseStock(
        item.productId,
        item.quantity,
      );
    }
  }
}
```

**Emit vs Subscribe Pattern:**

```typescript
// ✅ Don't wait for Observable to complete
this.client.emit('event', data);  // Fire-and-forget

// ❌ BAD: Trying to get response from emit
await firstValueFrom(this.client.emit('event', data));  // Don't do this!

// ✅ If you need response, use send() instead
const result = await firstValueFrom(this.client.send('pattern', data));
```

**Event Payload Best Practices:**

```typescript
// ✅ GOOD: Include all necessary data
this.client.emit('order.created', {
  orderId: order.id,
  userId: order.userId,
  total: order.total,
  items: order.items,
  timestamp: new Date().toISOString(),
  metadata: {
    ip: req.ip,
    userAgent: req.headers['user-agent'],
  },
});

// ✅ GOOD: Use structured event names
this.client.emit('user.profile.updated', data);
this.client.emit('payment.card.added', data);

// ❌ BAD: Minimal data (requires extra queries)
this.client.emit('order.created', { orderId: order.id });  // Not enough!

// ❌ BAD: Generic event names
this.client.emit('update', data);  // Too vague!
```

**Event Versioning:**

```typescript
// ✅ Version events for backward compatibility
this.client.emit('order.created.v2', {
  orderId: order.id,
  userId: order.userId,
  // ... new fields
});

// Or use object pattern
this.client.emit(
  { event: 'order.created', version: 'v2' },
  orderData,
);

// Handler for different versions
@EventPattern('order.created.v1')
asynchandleV1(@Payload() data: any) {
  // Handle old format
}

@EventPattern('order.created.v2')
async handleV2(@Payload() data: any) {
  // Handle new format
}
```

**Error Handling in Events:**

```typescript
// Server: Event handlers should handle errors gracefully
@EventPattern('order.placed')
async handleOrderPlaced(@Payload() data: any) {
  try {
    await this.processOrder(data);
  } catch (error) {
    // ✅ Log error but don't throw (no one is listening for response)
    console.error('Failed to process order event:', error);
    
    // ✅ Emit error event
    this.client.emit('order.processing.failed', {
      orderId: data.orderId,
      error: error.message,
      timestamp: new Date(),
    });
  }
}
```

**Best Practices:**

```typescript
// ✅ GOOD: Use emit for fire-and-forget
this.client.emit('user.created', userData);

// ✅ GOOD: Include timestamp
this.client.emit('event', {
  ...data,
  timestamp: new Date().toISOString(),
});

// ✅ GOOD: Use dot notation for event names
this.client.emit('user.profile.updated', data);
this.client.emit('order.payment.completed', data);

// ✅ GOOD: Handle errors in event handlers
@EventPattern('event')
async handleEvent(@Payload() data: any) {
  try {
    await this.process(data);
  } catch (error) {
    console.error('Event processing failed:', error);
  }
}

// ✅ GOOD: Don't await emit
this.client.emit('event', data);  // Don't await!

// ❌ BAD: Trying to get response from emit
const result = await firstValueFrom(
  this.client.emit('event', data),  // Don't do this!
);

// ❌ BAD: Throwing errors in event handlers
@EventPattern('event')
async handleEvent(@Payload() data: any) {
  throw new Error('Failed');  // No one is listening!
}

// ❌ BAD: Emitting too frequently (performance)
for (const item of items) {
  this.client.emit('item.processed', item);  // Batch instead!
}
```

**Interview Tip**: **`client.emit()`** is used for **fire-and-forget** event-based communication. Sends events **without waiting for response**, **non-blocking**, **asynchronous**. Multiple services can listen to the same event (**broadcast**). Use for: **event-driven architecture**, **notifications**, **analytics**, **audit logs**, **decoupled services**. Different from **`send()`** which waits for response. Event handlers use **@EventPattern()** decorator. Common patterns: **user.created**, **order.placed**, **payment.completed**. Always include **timestamp** and **necessary data** in events. Don't await emit, don't expect response. Handle errors gracefully in handlers (log, don't throw). Use **dot notation** for event names. Best for **one-to-many** communication.

</details>

### 29. What is the difference between `send()` and `emit()`?

<details>
<summary>Answer</summary>

**`send()`** and **`emit()`** are two different communication patterns in NestJS microservices: **request-response** vs **event-based** (fire-and-forget).

**Key Differences:**

| Feature | `send()` | `emit()` |
|---------|----------|----------|
| **Pattern** | Request-Response | Event-Based (Pub/Sub) |
| **Waits for response** | ✅ Yes | ❌ No |
| **Blocking** | ✅ Yes (waits) | ❌ No (fire-and-forget) |
| **Return value** | Observable with response | Observable (ignored) |
| **Use case** | Need response data | Notify/broadcast events |
| **Handler decorator** | @MessagePattern() | @EventPattern() |
| **Error handling** | Client receives errors | Errors stay on server |
| **Multiple handlers** | One handler | Multiple handlers |
| **Performance** | Slower (round-trip) | Faster (no wait) |
| **Reliability** | Synchronous-like | Fire-and-forget |

**1. `client.send()` - Request-Response:**

```typescript
// ===== Client =====
import { firstValueFrom } from 'rxjs';

@Controller('users')
export class UsersController {
  constructor(
    @Inject('USERS_SERVICE') private client: ClientProxy,
  ) {}
  
  @Get(':id')
  async getUser(@Param('id') id: string) {
    // ✅ send() - WAITS for response
    const user = await firstValueFrom(
      this.client.send('users.findById', { id }),
    );
    
    return user;  // Returns actual user data
  }
}

// ===== Server =====
@Controller()
export class UsersController {
  // ✅ @MessagePattern for send()
  @MessagePattern('users.findById')
  async findById(@Payload() data: { id: string }) {
    const user = await this.usersService.findById(data.id);
    return user;  // Response sent back to client
  }
}
```

**2. `client.emit()` - Event-Based:**

```typescript
// ===== Client =====
@Controller('users')
export class UsersController {
  constructor(
    @Inject('EVENTS_SERVICE') private client: ClientProxy,
  ) {}
  
  @Post()
  async createUser(@Body() userData: any) {
    const user = await this.usersService.create(userData);
    
    // ✅ emit() - DOESN'T wait for response
    this.client.emit('user.created', {
      id: user.id,
      email: user.email,
    });
    
    return user;  // Returns immediately
  }
}

// ===== Server =====
@Controller()
export class NotificationsController {
  // ✅ @EventPattern for emit()
  @EventPattern('user.created')
  async handleUserCreated(@Payload() data: any) {
    // Process event (no response expected)
    await this.sendWelcomeEmail(data.email);
    // Return value is ignored
  }
}
```

**Comparison Examples:**

```typescript
// ===== Example 1: Fetching Data (Use send()) =====

// ✅ GOOD: Use send() when you need the response
@Get(':id')
async getUser(@Param('id') id: string) {
  const user = await firstValueFrom(
    this.client.send('users.findById', { id }),
  );
  return user;  // Need user data to return to client
}

// ❌ BAD: Using emit() when you need response
@Get(':id')
async getUser(@Param('id') id: string) {
  this.client.emit('users.findById', { id });  // No response!
  return {};  // No data to return!
}

// ===== Example 2: Notifications (Use emit()) =====

// ✅ GOOD: Use emit() for notifications
@Post()
async createOrder(@Body() orderData: any) {
  const order = await this.ordersService.create(orderData);
  
  // Fire-and-forget notification
  this.client.emit('order.created', order);
  
  return order;
}

// ❌ BAD: Using send() for notifications
@Post()
async createOrder(@Body() orderData: any) {
  const order = await this.ordersService.create(orderData);
  
  // Unnecessarily waits for response
  await firstValueFrom(
    this.client.send('order.created', order),  // Don't need response!
  );
  
  return order;
}
```

**When to Use `send()`:**

```typescript
// ✅ Getting data from another service
const user = await firstValueFrom(
  this.client.send('users.findById', { id }),
);

// ✅ Executing operations that return results
const result = await firstValueFrom(
  this.client.send('payment.process', paymentData),
);

// ✅ Validation checks
const isValid = await firstValueFrom(
  this.client.send('inventory.validate', items),
);

// ✅ Calculations
const total = await firstValueFrom(
  this.client.send('pricing.calculate', cart),
);

// ✅ Any operation where you need the result
const report = await firstValueFrom(
  this.client.send('analytics.generate', params),
);
```

**When to Use `emit()`:**

```typescript
// ✅ Sending notifications
this.client.emit('user.created', userData);

// ✅ Logging/auditing
this.client.emit('audit.log', {
  action: 'user.created',
  userId: user.id,
  timestamp: new Date(),
});

// ✅ Analytics/tracking
this.client.emit('analytics.track', {
  event: 'order_placed',
  orderId: order.id,
});

// ✅ Broadcasting events to multiple services
this.client.emit('order.placed', orderData);
// Multiple services can listen and react

// ✅ Cache invalidation
this.client.emit('cache.invalidate', { key: 'users' });

// ✅ Background processing
this.client.emit('image.process', {
  imageId: image.id,
  operations: ['resize', 'compress'],
});
```

**Performance Comparison:**

```typescript
// send() - Slower (waits for response)
@Post()
async createUser(@Body() userData: any) {
  const start = Date.now();
  
  const user = await this.usersService.create(userData);
  
  // ❌ send() waits for response (adds latency)
  await firstValueFrom(
    this.client.send('analytics.track', { userId: user.id }),
  );
  
  console.log(`Time: ${Date.now() - start}ms`);  // ~100-500ms
  return user;
}

// emit() - Faster (fire-and-forget)
@Post()
async createUser(@Body() userData: any) {
  const start = Date.now();
  
  const user = await this.usersService.create(userData);
  
  // ✅ emit() doesn't wait (no latency)
  this.client.emit('analytics.track', { userId: user.id });
  
  console.log(`Time: ${Date.now() - start}ms`);  // ~10-50ms
  return user;
}
```

**Error Handling Differences:**

```typescript
// send() - Client receives errors
@Get(':id')
async getUser(@Param('id') id: string) {
  try {
    const user = await firstValueFrom(
      this.client.send('users.findById', { id }),
    );
    return user;
  } catch (error) {
    // ✅ Client can catch and handle errors
    if (error instanceof RpcException) {
      throw new NotFoundException('User not found');
    }
    throw new BadGatewayException('Service unavailable');
  }
}

// emit() - Errors stay on server
@Post()
async createUser(@Body() userData: any) {
  const user = await this.usersService.create(userData);
  
  // ❌ emit() - client doesn't know if it fails
  this.client.emit('send.welcome.email', { email: user.email });
  
  return user;  // Returns even if email service fails
}

// Server handles errors internally
@EventPattern('send.welcome.email')
async sendWelcomeEmail(@Payload() data: any) {
  try {
    await this.emailService.send(data.email);
  } catch (error) {
    // ❌ Error stays here - client doesn't know
    console.error('Failed to send email:', error);
    // Maybe emit an error event
    this.client.emit('email.failed', { email: data.email, error });
  }
}
```

**Multiple Handlers:**

```typescript
// send() - ONE handler only
@MessagePattern('users.findById')  // Service 1
async findById1() { return { id: 1 }; }

@MessagePattern('users.findById')  // Service 2
async findById2() { return { id: 2 }; }
// Only one will handle the request (round-robin or first)

// emit() - MULTIPLE handlers (broadcast)
@EventPattern('user.created')  // Service 1: Notifications
async handleUserCreated1() { await this.sendEmail(); }

@EventPattern('user.created')  // Service 2: Analytics
async handleUserCreated2() { await this.trackEvent(); }

@EventPattern('user.created')  // Service 3: Audit
async handleUserCreated3() { await this.logAudit(); }
// ALL three handlers receive the event!
```

**Complete Comparison Example:**

```typescript
// ===== API Gateway =====
@Controller('orders')
export class OrdersController {
  constructor(
    @Inject('ORDERS_SERVICE') private ordersClient: ClientProxy,
    @Inject('EVENTS_BUS') private eventBus: ClientProxy,
  ) {}
  
  @Post()
  async createOrder(@Body() orderData: any) {
    // Step 1: Validate inventory (NEED RESPONSE - use send())
    const inventory = await firstValueFrom(
      this.ordersClient.send('inventory.validate', orderData.items),
    );
    
    if (!inventory.available) {
      throw new BadRequestException('Items not available');
    }
    
    // Step 2: Process payment (NEED RESPONSE - use send())
    const payment = await firstValueFrom(
      this.ordersClient.send('payment.process', {
        amount: orderData.total,
        method: orderData.paymentMethod,
      }),
    );
    
    if (!payment.success) {
      throw new BadRequestException('Payment failed');
    }
    
    // Step 3: Create order (NEED RESPONSE - use send())
    const order = await firstValueFrom(
      this.ordersClient.send('orders.create', {
        ...orderData,
        paymentId: payment.id,
      }),
    );
    
    // Step 4: Notify services (DON'T NEED RESPONSE - use emit())
    this.eventBus.emit('order.created', {
      orderId: order.id,
      userId: order.userId,
      total: order.total,
    });
    
    // Step 5: Track analytics (DON'T NEED RESPONSE - use emit())
    this.eventBus.emit('analytics.track', {
      event: 'order_placed',
      orderId: order.id,
    });
    
    // Step 6: Send confirmation email (DON'T NEED RESPONSE - use emit())
    this.eventBus.emit('email.send', {
      to: order.userEmail,
      template: 'order-confirmation',
      data: order,
    });
    
    return order;
  }
}
```

**Choosing Between send() and emit():**

```
Use send() when:
✅ You need the response data
✅ Operation must complete successfully
✅ You need to validate the result
✅ Synchronous-like behavior required
✅ Error handling is critical
✅ Getting data (queries)
✅ Calculations, validations

Use emit() when:
✅ You don't need a response
✅ Fire-and-forget operations
✅ Notifications, alerts
✅ Analytics, logging
✅ Broadcasting to multiple services
✅ Background processing
✅ Performance is critical
✅ Decoupled event-driven architecture
```

**Common Mistakes:**

```typescript
// ❌ MISTAKE 1: Using send() for notifications
@Post()
async createUser(@Body() userData: any) {
  const user = await this.usersService.create(userData);
  
  // ❌ Bad: Unnecessary wait
  await firstValueFrom(
    this.client.send('send.email', user),  // Should use emit()!
  );
  
  return user;
}

// ✅ CORRECT: Use emit() for notifications
@Post()
async createUser(@Body() userData: any) {
  const user = await this.usersService.create(userData);
  this.client.emit('send.email', user);  // Fire-and-forget
  return user;
}

// ❌ MISTAKE 2: Using emit() when you need data
@Get(':id')
async getUser(@Param('id') id: string) {
  // ❌ Bad: No response data
  this.client.emit('users.findById', { id });
  return {};  // No data!
}

// ✅ CORRECT: Use send() to get data
@Get(':id')
async getUser(@Param('id') id: string) {
  const user = await firstValueFrom(
    this.client.send('users.findById', { id }),
  );
  return user;
}

// ❌ MISTAKE 3: Awaiting emit()
@Post()
async createOrder(@Body() orderData: any) {
  const order = await this.ordersService.create(orderData);
  
  // ❌ Bad: Awaiting emit (no value returned)
  await firstValueFrom(
    this.client.emit('order.created', order),
  );
  
  return order;
}

// ✅ CORRECT: Don't await emit()
@Post()
async createOrder(@Body() orderData: any) {
  const order = await this.ordersService.create(orderData);
  this.client.emit('order.created', order);  // Fire-and-forget
  return order;
}
```

**Best Practices:**

```typescript
// ✅ GOOD: Use send() for queries/commands that return data
const user = await firstValueFrom(
  this.client.send('users.findById', { id }),
);

// ✅ GOOD: Use emit() for events/notifications
this.client.emit('user.created', userData);

// ✅ GOOD: Combine both patterns
@Post()
async createOrder(@Body() orderData: any) {
  // Use send() for critical operations
  const order = await firstValueFrom(
    this.client.send('orders.create', orderData),
  );
  
  // Use emit() for notifications
  this.client.emit('order.created', order);
  
  return order;
}

// ✅ GOOD: Error handling for send()
try {
  const result = await firstValueFrom(
    this.client.send('pattern', data),
  );
} catch (error) {
  // Handle errors
}

// ✅ GOOD: Error handling for emit() (on server)
@EventPattern('event')
async handleEvent(@Payload() data: any) {
  try {
    await this.process(data);
  } catch (error) {
    console.error('Event processing failed:', error);
  }
}
```

**Interview Tip**: **`send()`** is for **request-response** (waits for response), **`emit()`** is for **event-based** (fire-and-forget). Use **`send()`** when you **need the response data** (queries, validations, calculations). Use **`emit()`** when you **don't need response** (notifications, analytics, logging, broadcasting). **`send()`** uses **@MessagePattern()**, **`emit()`** uses **@EventPattern()**. **`send()`** is **blocking** (waits), **`emit()`** is **non-blocking** (faster). **`send()`** has **one handler**, **`emit()`** can have **multiple handlers** (broadcast). **`emit()`** is **faster** but **no error feedback** to client. Choose based on whether you **need response** or just **notify**.

</details>

### 30. Does `emit()` wait for a response?

<details>
<summary>Answer</summary>

**No, `emit()` does NOT wait for a response.** It is a **fire-and-forget** operation that sends an event and **immediately continues execution** without waiting for any handlers to process it.

**Key Points:**

```
❌ Does NOT wait for response
❌ Does NOT block execution
❌ Does NOT return data from handlers
✅ Fire-and-forget pattern
✅ Non-blocking
✅ Asynchronous
✅ Continues immediately
✅ No acknowledgment
```

**Demonstration:**

```typescript
import { Controller, Post, Body, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';

@Controller('users')
export class UsersController {
  constructor(
    @Inject('EVENTS_SERVICE') private client: ClientProxy,
  ) {}
  
  @Post()
  async createUser(@Body() userData: any) {
    console.log('1. Creating user...');
    const user = await this.usersService.create(userData);
    
    console.log('2. Emitting user.created event...');
    // ✅ emit() returns immediately
    this.client.emit('user.created', {
      id: user.id,
      email: user.email,
    });
    
    console.log('3. Continuing execution...');
    console.log('4. Returning response...');
    
    return user;
    
    // Output:
    // 1. Creating user...
    // 2. Emitting user.created event...
    // 3. Continuing execution...      ← Doesn't wait!
    // 4. Returning response...
  }
}

// Server (handler executes asynchronously)
@Controller()
export class NotificationsController {
  @EventPattern('user.created')
  async handleUserCreated(@Payload() data: any) {
    console.log('5. Processing event...');  // Happens later!
    await this.sendWelcomeEmail(data.email);
    console.log('6. Event processed.');
  }
}
```

**Timing Comparison:**

```typescript
// emit() - No wait (fast)
@Post()
async createUser(@Body() userData: any) {
  const start = Date.now();
  
  const user = await this.usersService.create(userData);
  
  // ✅ emit() - returns immediately
  this.client.emit('send.welcome.email', user);
  
  const duration = Date.now() - start;
  console.log(`Duration: ${duration}ms`);  // ~10-50ms
  
  return user;  // Returns immediately
}

// send() - Waits for response (slow)
@Post()
async createUser(@Body() userData: any) {
  const start = Date.now();
  
  const user = await this.usersService.create(userData);
  
  // ❌ send() - waits for response
  await firstValueFrom(
    this.client.send('send.welcome.email', user),
  );
  
  const duration = Date.now() - start;
  console.log(`Duration: ${duration}ms`);  // ~100-500ms
  
  return user;  // Returns after handler completes
}
```

**No Response Value:**

```typescript
// ❌ WRONG: Trying to get response from emit()
@Post()
async createUser(@Body() userData: any) {
  const user = await this.usersService.create(userData);
  
  // ❌ This doesn't give you the handler's return value!
  const result = await firstValueFrom(
    this.client.emit('user.created', user),
  );
  
  console.log(result);  // undefined or empty
  return user;
}

// Server handler
@EventPattern('user.created')
async handleUserCreated(@Payload() data: any) {
  await this.sendEmail(data.email);
  return { emailSent: true };  // ❌ Client never receives this!
}

// ✅ CORRECT: Don't expect response from emit()
@Post()
async createUser(@Body() userData: any) {
  const user = await this.usersService.create(userData);
  
  // ✅ Fire-and-forget (don't await)
  this.client.emit('user.created', user);
  
  return user;  // Return immediately
}
```

**Execution Flow:**

```typescript
// ===== Client =====
@Post()
async createOrder(@Body() orderData: any) {
  console.log('Step 1: Start');
  
  const order = await this.ordersService.create(orderData);
  console.log('Step 2: Order created');
  
  // ✅ emit() - doesn't wait
  this.client.emit('order.created', order);
  console.log('Step 3: Event emitted (not processed yet!)');
  
  this.client.emit('analytics.track', { orderId: order.id });
  console.log('Step 4: Analytics event emitted');
  
  console.log('Step 5: Returning response');
  return order;
}

// Output:
// Step 1: Start
// Step 2: Order created
// Step 3: Event emitted (not processed yet!)
// Step 4: Analytics event emitted
// Step 5: Returning response
// (Response sent to client)

// ===== Server 1 (Notifications) =====
@EventPattern('order.created')
async sendNotification(@Payload() data: any) {
  console.log('Step 6: Processing notification...');  // Happens async!
  await this.emailService.send(data);
  console.log('Step 7: Notification sent');
}

// ===== Server 2 (Analytics) =====
@EventPattern('analytics.track')
async trackEvent(@Payload() data: any) {
  console.log('Step 8: Tracking event...');  // Happens async!
  await this.analyticsService.track(data);
  console.log('Step 9: Event tracked');
}

// Steps 6-9 happen AFTER response is already sent!
```

**Multiple Handlers (All Execute Independently):**

```typescript
// Client emits once
@Post()
async createUser(@Body() userData: any) {
  const user = await this.usersService.create(userData);
  
  console.log('Emitting user.created...');
  this.client.emit('user.created', user);
  console.log('Returned immediately!');
  
  return user;  // Returns before any handler finishes
}

// Server 1: Notifications (runs independently)
@EventPattern('user.created')
async handleNotification(@Payload() data: any) {
  console.log('[Notifications] Processing...');
  await sleep(2000);  // 2 seconds
  console.log('[Notifications] Done');
}

// Server 2: Analytics (runs independently)
@EventPattern('user.created')
async handleAnalytics(@Payload() data: any) {
  console.log('[Analytics] Processing...');
  await sleep(1000);  // 1 second
  console.log('[Analytics] Done');
}

// Server 3: Audit (runs independently)
@EventPattern('user.created')
async handleAudit(@Payload() data: any) {
  console.log('[Audit] Processing...');
  await sleep(500);  // 0.5 seconds
  console.log('[Audit] Done');
}

// Output:
// Emitting user.created...
// Returned immediately!         ← Client returns first!
// [Audit] Processing...
// [Analytics] Processing...
// [Notifications] Processing...
// [Audit] Done                   ← Handlers finish later
// [Analytics] Done
// [Notifications] Done
```

**No Error Feedback:**

```typescript
// Client doesn't know if handlers fail
@Post()
async createUser(@Body() userData: any) {
  const user = await this.usersService.create(userData);
  
  // ❌ emit() - no error feedback
  this.client.emit('send.welcome.email', user);
  
  return { success: true };  // Always returns success!
  // Even if email service fails!
}

// Server handler fails silently (from client's perspective)
@EventPattern('send.welcome.email')
async sendEmail(@Payload() data: any) {
  try {
    await this.emailService.send(data.email);
  } catch (error) {
    // ❌ Client has no idea this failed!
    console.error('Email failed:', error);
    
    // ✅ Could emit an error event
    this.client.emit('email.failed', {
      userId: data.id,
      error: error.message,
    });
  }
}
```

**When emit() Behavior is Useful:**

```typescript
// ✅ GOOD: Non-blocking operations
@Post()
async uploadImage(@UploadedFile() file: any) {
  // Save image immediately
  const image = await this.imagesService.save(file);
  
  // ✅ Process image in background (don't wait)
  this.client.emit('image.process', {
    imageId: image.id,
    operations: ['resize', 'compress', 'watermark'],
  });
  
  return image;  // Returns immediately (image processing happens later)
}

// ✅ GOOD: Multiple independent notifications
@Post('checkout')
async checkout(@Body() checkoutData: any) {
  const order = await this.ordersService.create(checkoutData);
  
  // ✅ All fire-and-forget (don't wait for any)
  this.client.emit('order.created', order);
  this.client.emit('inventory.decrease', order.items);
  this.client.emit('analytics.track', { event: 'checkout', orderId: order.id });
  this.client.emit('send.confirmation.email', order);
  
  return order;  // Returns immediately
}

// ✅ GOOD: Logging/auditing
@Put(':id')
async updateUser(@Param('id') id: string, @Body() updateData: any) {
  const user = await this.usersService.update(id, updateData);
  
  // ✅ Audit log (don't wait)
  this.client.emit('audit.log', {
    action: 'user.updated',
    userId: id,
    changes: updateData,
    timestamp: new Date(),
  });
  
  return user;
}
```

**When emit() Behavior is NOT Suitable:**

```typescript
// ❌ BAD: Need to know if operation succeeded
@Post('payment')
async processPayment(@Body() paymentData: any) {
  // ❌ emit() - don't know if payment succeeded!
  this.client.emit('payment.process', paymentData);
  
  return { success: true };  // Can't know if true!
}

// ✅ GOOD: Use send() to get result
@Post('payment')
async processPayment(@Body() paymentData: any) {
  const result = await firstValueFrom(
    this.client.send('payment.process', paymentData),
  );
  
  if (!result.success) {
    throw new BadRequestException('Payment failed');
  }
  
  return result;
}

// ❌ BAD: Need data from handler
@Get(':id/recommendations')
async getRecommendations(@Param('id') userId: string) {
  // ❌ emit() - won't get recommendations!
  this.client.emit('recommendations.get', { userId });
  
  return [];  // No data!
}

// ✅ GOOD: Use send() to get data
@Get(':id/recommendations')
async getRecommendations(@Param('id') userId: string) {
  const recommendations = await firstValueFrom(
    this.client.send('recommendations.get', { userId }),
  );
  
  return recommendations;
}
```

**Observable Behavior:**

```typescript
// emit() returns Observable but it's empty/ignored
import { Observable } from 'rxjs';

@Post()
async createUser(@Body() userData: any) {
  const user = await this.usersService.create(userData);
  
  // emit() returns Observable<any>
  const observable: Observable<any> = this.client.emit('user.created', user);
  
  // But subscribing gives no useful data
  observable.subscribe({
    next: (data) => {
      console.log(data);  // undefined or empty
    },
    complete: () => {
      console.log('Completed');  // Completes immediately
    },
  });
  
  return user;
}

// Just emit and forget - don't subscribe
this.client.emit('user.created', user);  // ✅ Correct usage
```

**Best Practices:**

```typescript
// ✅ GOOD: Use emit() for fire-and-forget
this.client.emit('user.created', userData);

// ✅ GOOD: Don't await emit()
this.client.emit('event', data);  // Not: await ...

// ✅ GOOD: Don't expect response
this.client.emit('event', data);
return { success: true };  // Based on own logic, not handler

// ✅ GOOD: Use send() if you need response
const result = await firstValueFrom(
  this.client.send('pattern', data),
);

// ❌ BAD: Awaiting emit() result
const result = await firstValueFrom(
  this.client.emit('event', data),  // No useful result!
);

// ❌ BAD: Expecting response from emit()
this.client.emit('get.user', { id });
const user = ???;  // Can't get user data!

// ❌ BAD: Making decisions based on emit()
this.client.emit('validate.data', data);
if (isValid) {  // Can't know if valid!
  // ...
}
```

**Summary:**

```
emit() Characteristics:
❌ Does NOT wait
❌ Does NOT block
❌ Does NOT return handler results
❌ Does NOT provide error feedback
✅ Fire-and-forget
✅ Non-blocking
✅ Immediate return
✅ Fast execution
✅ Good for notifications
✅ Good for events
✅ Good for broadcasting

When you need response:
✅ Use send() instead
✅ Wait for result
✅ Get error feedback
✅ Synchronous-like behavior
```

**Interview Tip**: **No, `emit()` does NOT wait for a response.** It is **fire-and-forget**, **non-blocking**, returns **immediately** without waiting for handlers. **No response value** from handlers, **no error feedback** to client. Use **`emit()`** for **notifications**, **events**, **broadcasting**, **analytics**, **logging** where you **don't need response**. If you **need response** or **need to know if operation succeeded**, use **`send()`** instead. **`emit()`** is **faster** because it doesn't wait, but provides **no acknowledgment**. Handler return values are **ignored**. Multiple handlers execute **independently** and **asynchronously** after client continues. Perfect for **event-driven architecture** and **decoupled services**.

</details>

## gRPC

### 31. What is gRPC and why use it over HTTP?

<details>
<summary>Answer</summary>

**gRPC** (gRPC Remote Procedure Call) is a modern, high-performance **RPC framework** developed by Google that uses **HTTP/2**, **Protocol Buffers (protobuf)**, and **strongly typed contracts** for efficient communication between microservices.

**Key Features:**

```
✅ HTTP/2 protocol
✅ Protocol Buffers (binary serialization)
✅ Strongly typed contracts
✅ Bidirectional streaming
✅ Built-in code generation
✅ High performance
✅ Language agnostic
✅ Built-in load balancing
✅ Authentication/authorization
✅ Lower latency
```

**gRPC vs HTTP/REST Comparison:**

| Feature | gRPC | HTTP/REST |
|---------|------|----------|
| **Protocol** | HTTP/2 | HTTP/1.1 (mostly) |
| **Data format** | Binary (Protobuf) | JSON/XML (text) |
| **Performance** | 🚀 Very fast | Slower |
| **Payload size** | 🚀 Smaller (binary) | Larger (text) |
| **Streaming** | ✅ Bidirectional | ❌ Limited |
| **Type safety** | ✅ Strong (codegen) | ❌ Weak (manual) |
| **Browser support** | ❌ Limited | ✅ Native |
| **Human readable** | ❌ Binary | ✅ Yes |
| **Learning curve** | Steeper | Easier |
| **Contract** | .proto files | OpenAPI (optional) |
| **Code generation** | ✅ Built-in | Manual or tools |
| **Latency** | Lower | Higher |

**Why Use gRPC:**

**1. Performance:**

```typescript
// gRPC with Protobuf (binary)
// Payload size: ~50-100 bytes
message User {
  int32 id = 1;
  string name = 2;
  string email = 3;
}

// REST with JSON (text)
// Payload size: ~150-200 bytes
{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com"
}

// gRPC is typically:
// - 5-10x smaller payload
// - 2-5x faster serialization/deserialization
// - Lower CPU usage
// - Lower network bandwidth
```

**2. Type Safety:**

```typescript
// ===== gRPC - Strong Type Safety =====

// 1. Define .proto file
syntax = "proto3";

message GetUserRequest {
  int32 id = 1;  // Type enforced!
}

message User {
  int32 id = 1;
  string name = 2;
  string email = 3;
}

service UsersService {
  rpc GetUser (GetUserRequest) returns (User);  // Contract!
}

// 2. Auto-generated TypeScript interfaces
interface GetUserRequest {
  id: number;  // Strongly typed!
}

interface User {
  id: number;
  name: string;
  email: string;
}

// 3. Type-safe implementation
@GrpcMethod('UsersService', 'GetUser')
getUser(data: GetUserRequest): User {  // ✅ Types enforced!
  return {
    id: data.id,
    name: 'John',
    email: 'john@example.com',
  };
}

// ===== REST - Weak Type Safety =====

@Get(':id')
getUser(@Param('id') id: string): any {  // ❌ No type enforcement
  return {
    id: parseInt(id),  // Manual parsing
    name: 'John',
    email: 'john@example.com',
  };
}
```

**3. Streaming Support:**

```typescript
// gRPC supports 4 streaming patterns:

// ===== 1. Unary (Request-Response) =====
rpc GetUser (GetUserRequest) returns (User);

@GrpcMethod('UsersService', 'GetUser')
getUser(data: GetUserRequest): User {
  return user;
}

// ===== 2. Server Streaming =====
rpc StreamUsers (EmptyRequest) returns (stream User);

@GrpcStreamMethod('UsersService', 'StreamUsers')
streamUsers(): Observable<User> {
  return from(this.usersService.findAll());  // Stream of users
}

// ===== 3. Client Streaming =====
rpc CreateUsers (stream CreateUserRequest) returns (CreateUsersResponse);

@GrpcStreamCall('UsersService', 'CreateUsers')
createUsers(stream: Observable<CreateUserRequest>): Observable<CreateUsersResponse> {
  return stream.pipe(
    toArray(),
    switchMap(users => this.usersService.createMany(users)),
  );
}

// ===== 4. Bidirectional Streaming =====
rpc Chat (stream ChatMessage) returns (stream ChatMessage);

@GrpcStreamCall('UsersService', 'Chat')
chat(messages: Observable<ChatMessage>): Observable<ChatMessage> {
  return messages.pipe(
    map(msg => this.processMessage(msg)),
  );
}

// REST streaming is limited (SSE, WebSocket workarounds)
```

**4. Code Generation:**

```bash
# gRPC - Automatic code generation from .proto files
$ npm run proto:generate

# Generated files:
# - TypeScript interfaces
# - Service stubs
# - Client code
# - Server code
# All strongly typed!

// REST - Manual implementation
// - Write interfaces manually
// - No automatic client generation
// - No contract enforcement
```

**5. HTTP/2 Benefits:**

```typescript
// HTTP/2 features gRPC uses:

// ✅ Multiplexing - multiple requests over single connection
const user = await client.getUser({ id: 1 });
const posts = await client.getPosts({ userId: 1 });
const comments = await client.getComments({ userId: 1 });
// All 3 requests use same TCP connection!

// ✅ Header compression
// Reduces overhead from repeated headers

// ✅ Server push (potential)
// Server can push resources before client requests

// ✅ Bidirectional streaming
// Full duplex communication

// HTTP/1.1 REST:
// - New connection per request (or connection pooling)
// - No multiplexing
// - Text headers (larger)
// - No streaming
```

**When to Use gRPC:**

```typescript
// ✅ GOOD use cases for gRPC:

// 1. Microservice-to-microservice communication
@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'USERS_PACKAGE',
        transport: Transport.GRPC,
        options: {
          package: 'users',
          protoPath: join(__dirname, './users.proto'),
        },
      },
    ]),
  ],
})
export class OrdersModule {}

// 2. Real-time streaming
streamOrders(): Observable<Order> {
  return this.ordersService.streamActiveOrders();
}

// 3. High-performance requirements
// - Low latency critical
// - High throughput needed
// - Resource-constrained environments

// 4. Polyglot environments
// - Services in different languages
// - Need consistent contracts
// - Type safety across languages

// 5. Internal APIs
// - Backend services
// - No browser access needed
```

**When NOT to Use gRPC:**

```typescript
// ❌ BAD use cases for gRPC:

// 1. Browser-facing APIs
// gRPC-Web has limitations
// REST/GraphQL better for browsers

@Controller('api')  // Use REST for browser APIs
export class PublicApiController {
  @Get('users')
  getUsers() {
    return this.usersService.findAll();
  }
}

// 2. Simple CRUD APIs
// REST is simpler and well-understood

// 3. Public APIs
// REST is more accessible
// Better documentation tools (Swagger)
// Easier to test (curl, Postman)

// 4. Developer experience priority
// REST is easier to debug
// Human-readable payloads
// Better tooling
```

**Complete gRPC Example:**

```typescript
// ===== 1. Define .proto file =====
// users.proto
syntax = "proto3";

package users;

message GetUserRequest {
  int32 id = 1;
}

message User {
  int32 id = 1;
  string name = 2;
  string email = 3;
  int64 created_at = 4;
}

message ListUsersRequest {
  int32 page = 1;
  int32 limit = 2;
}

message ListUsersResponse {
  repeated User users = 1;
  int32 total = 2;
}

service UsersService {
  rpc GetUser (GetUserRequest) returns (User);
  rpc ListUsers (ListUsersRequest) returns (ListUsersResponse);
  rpc StreamUsers (Empty) returns (stream User);
}

// ===== 2. Server Implementation =====
import { Controller } from '@nestjs/common';
import { GrpcMethod, GrpcStreamMethod } from '@nestjs/microservices';
import { Observable, from } from 'rxjs';

@Controller()
export class UsersController {
  constructor(private usersService: UsersService) {}
  
  // Unary RPC
  @GrpcMethod('UsersService', 'GetUser')
  async getUser(data: GetUserRequest): Promise<User> {
    return await this.usersService.findById(data.id);
  }
  
  // Unary RPC
  @GrpcMethod('UsersService', 'ListUsers')
  async listUsers(data: ListUsersRequest): Promise<ListUsersResponse> {
    const [users, total] = await this.usersService.findAll(
      data.page,
      data.limit,
    );
    return { users, total };
  }
  
  // Server streaming
  @GrpcStreamMethod('UsersService', 'StreamUsers')
  streamUsers(): Observable<User> {
    return from(this.usersService.findAll());
  }
}

// ===== 3. Main.ts Configuration =====
import { NestFactory } from '@nestjs/core';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';
import { join } from 'path';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.GRPC,
      options: {
        package: 'users',
        protoPath: join(__dirname, './users.proto'),
        url: 'localhost:5000',
      },
    },
  );
  await app.listen();
}
bootstrap();

// ===== 4. Client Usage =====
import { ClientGrpc } from '@nestjs/microservices';

@Injectable()
export class OrdersService {
  private usersService: any;
  
  constructor(@Inject('USERS_PACKAGE') private client: ClientGrpc) {}
  
  onModuleInit() {
    this.usersService = this.client.getService<any>('UsersService');
  }
  
  async createOrder(orderData: any) {
    // Call gRPC service
    const user = await firstValueFrom(
      this.usersService.GetUser({ id: orderData.userId }),
    );
    
    console.log(user);  // Strongly typed User object
    
    return this.ordersService.create({
      ...orderData,
      userName: user.name,
      userEmail: user.email,
    });
  }
}
```

**Performance Benchmarks:**

```typescript
// Typical performance differences:

// ===== REST/JSON =====
// Request: 1000 users
// Time: ~500ms
// Payload: ~150KB
// CPU: ~20%

@Get('users')
async getUsers() {
  const start = Date.now();
  const users = await this.usersService.findAll();
  console.log(`REST Time: ${Date.now() - start}ms`);
  return users;  // JSON serialization
}

// ===== gRPC/Protobuf =====
// Request: 1000 users
// Time: ~100ms (5x faster!)
// Payload: ~30KB (5x smaller!)
// CPU: ~5% (4x less!)

@GrpcMethod('UsersService', 'ListUsers')
async listUsers(data: ListUsersRequest) {
  const start = Date.now();
  const users = await this.usersService.findAll();
  console.log(`gRPC Time: ${Date.now() - start}ms`);
  return { users };  // Protobuf serialization
}
```

**Advantages Summary:**

```
gRPC Advantages:

✅ Performance:
  - Binary protocol (smaller payloads)
  - HTTP/2 (multiplexing, compression)
  - Faster serialization
  - Lower latency
  - Lower bandwidth
  - Lower CPU usage

✅ Type Safety:
  - Strongly typed contracts
  - Auto-generated code
  - Compile-time checking
  - Cross-language consistency

✅ Streaming:
  - Server streaming
  - Client streaming
  - Bidirectional streaming
  - Real-time communication

✅ Developer Experience:
  - Auto-generated clients
  - Built-in load balancing
  - Built-in error handling
  - Consistent patterns

✅ Scalability:
  - Efficient resource usage
  - Connection multiplexing
  - Better for microservices
```

**Disadvantages:**

```
gRPC Disadvantages:

❌ Browser Support:
  - Limited native support
  - Needs gRPC-Web (proxy)
  - Not ideal for public APIs

❌ Debugging:
  - Binary format (not human-readable)
  - Harder to inspect with curl/Postman
  - Need special tools (grpcurl, BloomRPC)

❌ Learning Curve:
  - More complex setup
  - Protobuf syntax
  - Code generation process

❌ Ecosystem:
  - Fewer middleware/tools vs REST
  - Less documentation
  - Smaller community
```

**Best Practices:**

```typescript
// ✅ GOOD: Use gRPC for internal microservices
@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'USERS_SERVICE',
        transport: Transport.GRPC,
        options: {
          package: 'users',
          protoPath: 'users.proto',
        },
      },
    ]),
  ],
})

// ✅ GOOD: Use REST for public APIs
@Controller('api/v1')
export class PublicApiController {
  @Get('users')
  getUsers() {
    return this.usersService.findAll();
  }
}

// ✅ GOOD: Hybrid approach
// - gRPC for service-to-service
// - REST for browser/public APIs
// - API Gateway translates between them

// API Gateway
@Controller('api')
export class GatewayController {
  constructor(
    @Inject('USERS_SERVICE') private usersClient: ClientGrpc,
  ) {}
  
  @Get('users/:id')  // REST endpoint
  async getUser(@Param('id') id: string) {
    // Call internal gRPC service
    return await firstValueFrom(
      this.usersService.GetUser({ id: parseInt(id) }),
    );
  }
}
```

**Interview Tip**: **gRPC** is a **high-performance RPC framework** using **HTTP/2** and **Protocol Buffers** (binary). **5-10x smaller payloads**, **2-5x faster** than REST/JSON. **Strongly typed** contracts with **auto-generated code**. Supports **bidirectional streaming** (4 patterns). Best for **microservice-to-microservice** communication, **high performance** needs, **polyglot** environments. **NOT for browsers** (limited support, use gRPC-Web). **HTTP/2 multiplexing** = multiple requests on single connection. **Type safety** across languages. Use gRPC for **internal APIs**, REST for **public/browser APIs**. **Binary format** harder to debug but much **faster** and **smaller**. Perfect for **service mesh**, **real-time streaming**, **resource-constrained** environments.

</details>

**Key Features:**

```
✅ HTTP/2 protocol
✅ Protocol Buffers (binary serialization)
✅ Strongly typed contracts
✅ Bidirectional streaming
✅ Built-in code generation
✅ High performance
✅ Language agnostic
✅ Built-in load balancing
✅ Authentication/authorization
✅ Lower latency
```

**gRPC vs HTTP/REST Comparison:**

| Feature | gRPC | HTTP/REST |
|---------|------|----------|
| **Protocol** | HTTP/2 | HTTP/1.1 (mostly) |
| **Data format** | Binary (Protobuf) | JSON/XML (text) |
| **Performance** | 🚀 Very fast | Slower |
| **Payload size** | 🚀 Smaller (binary) | Larger (text) |
| **Streaming** | ✅ Bidirectional | ❌ Limited |
| **Type safety** | ✅ Strong (codegen) | ❌ Weak (manual) |
| **Browser support** | ❌ Limited | ✅ Native |
| **Human readable** | ❌ Binary | ✅ Yes |
| **Learning curve** | Steeper | Easier |
| **Contract** | .proto files | OpenAPI (optional) |
| **Code generation** | ✅ Built-in | Manual or tools |
| **Latency** | Lower | Higher |

**Why Use gRPC:**

**1. Performance:**

```typescript
// gRPC with Protobuf (binary)
// Payload size: ~50-100 bytes
message User {
  int32 id = 1;
  string name = 2;
  string email = 3;
}

// REST with JSON (text)
// Payload size: ~150-200 bytes
{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com"
}

// gRPC is typically:
// - 5-10x smaller payload
// - 2-5x faster serialization/deserialization
// - Lower CPU usage
// - Lower network bandwidth
```

**2. Type Safety:**

```typescript
// ===== gRPC - Strong Type Safety =====

// 1. Define .proto file
syntax = "proto3";

message GetUserRequest {
  int32 id = 1;  // Type enforced!
}

message User {
  int32 id = 1;
  string name = 2;
  string email = 3;
}

service UsersService {
  rpc GetUser (GetUserRequest) returns (User);  // Contract!
}

// 2. Auto-generated TypeScript interfaces
interface GetUserRequest {
  id: number;  // Strongly typed!
}

interface User {
  id: number;
  name: string;
  email: string;
}

// 3. Type-safe implementation
@GrpcMethod('UsersService', 'GetUser')
getUser(data: GetUserRequest): User {  // ✅ Types enforced!
  return {
    id: data.id,
    name: 'John',
    email: 'john@example.com',
  };
}

// ===== REST - Weak Type Safety =====

@Get(':id')
getUser(@Param('id') id: string): any {  // ❌ No type enforcement
  return {
    id: parseInt(id),  // Manual parsing
    name: 'John',
    email: 'john@example.com',
  };
}
```

**3. Streaming Support:**

```typescript
// gRPC supports 4 streaming patterns:

// ===== 1. Unary (Request-Response) =====
rpc GetUser (GetUserRequest) returns (User);

@GrpcMethod('UsersService', 'GetUser')
getUser(data: GetUserRequest): User {
  return user;
}

// ===== 2. Server Streaming =====
rpc StreamUsers (EmptyRequest) returns (stream User);

@GrpcStreamMethod('UsersService', 'StreamUsers')
streamUsers(): Observable<User> {
  return from(this.usersService.findAll());  // Stream of users
}

// ===== 3. Client Streaming =====
rpc CreateUsers (stream CreateUserRequest) returns (CreateUsersResponse);

@GrpcStreamCall('UsersService', 'CreateUsers')
createUsers(stream: Observable<CreateUserRequest>): Observable<CreateUsersResponse> {
  return stream.pipe(
    toArray(),
    switchMap(users => this.usersService.createMany(users)),
  );
}

// ===== 4. Bidirectional Streaming =====
rpc Chat (stream ChatMessage) returns (stream ChatMessage);

@GrpcStreamCall('UsersService', 'Chat')
chat(messages: Observable<ChatMessage>): Observable<ChatMessage> {
  return messages.pipe(
    map(msg => this.processMessage(msg)),
  );
}

// REST streaming is limited (SSE, WebSocket workarounds)
```

**4. Code Generation:**

```bash
# gRPC - Automatic code generation from .proto files
$ npm run proto:generate

# Generated files:
# - TypeScript interfaces
# - Service stubs
# - Client code
# - Server code
# All strongly typed!

// REST - Manual implementation
// - Write interfaces manually
// - No automatic client generation
// - No contract enforcement
```

**5. HTTP/2 Benefits:**

```typescript
// HTTP/2 features gRPC uses:

// ✅ Multiplexing - multiple requests over single connection
const user = await client.getUser({ id: 1 });
const posts = await client.getPosts({ userId: 1 });
const comments = await client.getComments({ userId: 1 });
// All 3 requests use same TCP connection!

// ✅ Header compression
// Reduces overhead from repeated headers

// ✅ Server push (potential)
// Server can push resources before client requests

// ✅ Bidirectional streaming
// Full duplex communication

// HTTP/1.1 REST:
// - New connection per request (or connection pooling)
// - No multiplexing
// - Text headers (larger)
// - No streaming
```

**When to Use gRPC:**

```typescript
// ✅ GOOD use cases for gRPC:

// 1. Microservice-to-microservice communication
@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'USERS_PACKAGE',
        transport: Transport.GRPC,
        options: {
          package: 'users',
          protoPath: join(__dirname, './users.proto'),
        },
      },
    ]),
  ],
})
export class OrdersModule {}

// 2. Real-time streaming
streamOrders(): Observable<Order> {
  return this.ordersService.streamActiveOrders();
}

// 3. High-performance requirements
// - Low latency critical
// - High throughput needed
// - Resource-constrained environments

// 4. Polyglot environments
// - Services in different languages
// - Need consistent contracts
// - Type safety across languages

// 5. Internal APIs
// - Backend services
// - No browser access needed
```

**When NOT to Use gRPC:**

```typescript
// ❌ BAD use cases for gRPC:

// 1. Browser-facing APIs
// gRPC-Web has limitations
// REST/GraphQL better for browsers

@Controller('api')  // Use REST for browser APIs
export class PublicApiController {
  @Get('users')
  getUsers() {
    return this.usersService.findAll();
  }
}

// 2. Simple CRUD APIs
// REST is simpler and well-understood

// 3. Public APIs
// REST is more accessible
// Better documentation tools (Swagger)
// Easier to test (curl, Postman)

// 4. Developer experience priority
// REST is easier to debug
// Human-readable payloads
// Better tooling
```

**Complete gRPC Example:**

```typescript
// ===== 1. Define .proto file =====
// users.proto
syntax = "proto3";

package users;

message GetUserRequest {
  int32 id = 1;
}

message User {
  int32 id = 1;
  string name = 2;
  string email = 3;
  int64 created_at = 4;
}

message ListUsersRequest {
  int32 page = 1;
  int32 limit = 2;
}

message ListUsersResponse {
  repeated User users = 1;
  int32 total = 2;
}

service UsersService {
  rpc GetUser (GetUserRequest) returns (User);
  rpc ListUsers (ListUsersRequest) returns (ListUsersResponse);
  rpc StreamUsers (Empty) returns (stream User);
}

// ===== 2. Server Implementation =====
import { Controller } from '@nestjs/common';
import { GrpcMethod, GrpcStreamMethod } from '@nestjs/microservices';
import { Observable, from } from 'rxjs';

@Controller()
export class UsersController {
  constructor(private usersService: UsersService) {}
  
  // Unary RPC
  @GrpcMethod('UsersService', 'GetUser')
  async getUser(data: GetUserRequest): Promise<User> {
    return await this.usersService.findById(data.id);
  }
  
  // Unary RPC
  @GrpcMethod('UsersService', 'ListUsers')
  async listUsers(data: ListUsersRequest): Promise<ListUsersResponse> {
    const [users, total] = await this.usersService.findAll(
      data.page,
      data.limit,
    );
    return { users, total };
  }
  
  // Server streaming
  @GrpcStreamMethod('UsersService', 'StreamUsers')
  streamUsers(): Observable<User> {
    return from(this.usersService.findAll());
  }
}

// ===== 3. Main.ts Configuration =====
import { NestFactory } from '@nestjs/core';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';
import { join } from 'path';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.GRPC,
      options: {
        package: 'users',
        protoPath: join(__dirname, './users.proto'),
        url: 'localhost:5000',
      },
    },
  );
  await app.listen();
}
bootstrap();

// ===== 4. Client Usage =====
import { ClientGrpc } from '@nestjs/microservices';

@Injectable()
export class OrdersService {
  private usersService: any;
  
  constructor(@Inject('USERS_PACKAGE') private client: ClientGrpc) {}
  
  onModuleInit() {
    this.usersService = this.client.getService<any>('UsersService');
  }
  
  async createOrder(orderData: any) {
    // Call gRPC service
    const user = await firstValueFrom(
      this.usersService.GetUser({ id: orderData.userId }),
    );
    
    console.log(user);  // Strongly typed User object
    
    return this.ordersService.create({
      ...orderData,
      userName: user.name,
      userEmail: user.email,
    });
  }
}
```

**Performance Benchmarks:**

```typescript
// Typical performance differences:

// ===== REST/JSON =====
// Request: 1000 users
// Time: ~500ms
// Payload: ~150KB
// CPU: ~20%

@Get('users')
async getUsers() {
  const start = Date.now();
  const users = await this.usersService.findAll();
  console.log(`REST Time: ${Date.now() - start}ms`);
  return users;  // JSON serialization
}

// ===== gRPC/Protobuf =====
// Request: 1000 users
// Time: ~100ms (5x faster!)
// Payload: ~30KB (5x smaller!)
// CPU: ~5% (4x less!)

@GrpcMethod('UsersService', 'ListUsers')
async listUsers(data: ListUsersRequest) {
  const start = Date.now();
  const users = await this.usersService.findAll();
  console.log(`gRPC Time: ${Date.now() - start}ms`);
  return { users };  // Protobuf serialization
}
```

**Advantages Summary:**

```
gRPC Advantages:

✅ Performance:
  - Binary protocol (smaller payloads)
  - HTTP/2 (multiplexing, compression)
  - Faster serialization
  - Lower latency
  - Lower bandwidth
  - Lower CPU usage

✅ Type Safety:
  - Strongly typed contracts
  - Auto-generated code
  - Compile-time checking
  - Cross-language consistency

✅ Streaming:
  - Server streaming
  - Client streaming
  - Bidirectional streaming
  - Real-time communication

✅ Developer Experience:
  - Auto-generated clients
  - Built-in load balancing
  - Built-in error handling
  - Consistent patterns

✅ Scalability:
  - Efficient resource usage
  - Connection multiplexing
  - Better for microservices
```

**Disadvantages:**

```
gRPC Disadvantages:

❌ Browser Support:
  - Limited native support
  - Needs gRPC-Web (proxy)
  - Not ideal for public APIs

❌ Debugging:
  - Binary format (not human-readable)
  - Harder to inspect with curl/Postman
  - Need special tools (grpcurl, BloomRPC)

❌ Learning Curve:
  - More complex setup
  - Protobuf syntax
  - Code generation process

❌ Ecosystem:
  - Fewer middleware/tools vs REST
  - Less documentation
  - Smaller community
```

**Best Practices:**

```typescript
// ✅ GOOD: Use gRPC for internal microservices
@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'USERS_SERVICE',
        transport: Transport.GRPC,
        options: {
          package: 'users',
          protoPath: 'users.proto',
        },
      },
    ]),
  ],
})

// ✅ GOOD: Use REST for public APIs
@Controller('api/v1')
export class PublicApiController {
  @Get('users')
  getUsers() {
    return this.usersService.findAll();
  }
}

// ✅ GOOD: Hybrid approach
// - gRPC for service-to-service
// - REST for browser/public APIs
// - API Gateway translates between them

// API Gateway
@Controller('api')
export class GatewayController {
  constructor(
    @Inject('USERS_SERVICE') private usersClient: ClientGrpc,
  ) {}
  
  @Get('users/:id')  // REST endpoint
  async getUser(@Param('id') id: string) {
    // Call internal gRPC service
    return await firstValueFrom(
      this.usersService.GetUser({ id: parseInt(id) }),
    );
  }
}
```

**Interview Tip**: **gRPC** is a **high-performance RPC framework** using **HTTP/2** and **Protocol Buffers** (binary). **5-10x smaller payloads**, **2-5x faster** than REST/JSON. **Strongly typed** contracts with **auto-generated code**. Supports **bidirectional streaming** (4 patterns). Best for **microservice-to-microservice** communication, **high performance** needs, **polyglot** environments. **NOT for browsers** (limited support, use gRPC-Web). **HTTP/2 multiplexing** = multiple requests on single connection. **Type safety** across languages. Use gRPC for **internal APIs**, REST for **public/browser APIs**. **Binary format** harder to debug but much **faster** and **smaller**. Perfect for **service mesh**, **real-time streaming**, **resource-constrained** environments.

</details>

### 32. How do you define `.proto` files?

<details>
<summary>Answer</summary>

**`.proto` files** (Protocol Buffer files) define the **contract**, **data structures**, and **RPC services** for gRPC communication. They are **language-agnostic** and used to **generate code** for multiple languages.

**Basic Structure:**

```protobuf
// users.proto

// 1. Syntax version (proto3 is recommended)
syntax = "proto3";

// 2. Package name (namespace)
package users;

// 3. Message definitions (data structures)
message User {
  int32 id = 1;
  string name = 2;
  string email = 3;
}

// 4. Service definitions (RPC methods)
service UsersService {
  rpc GetUser (GetUserRequest) returns (User);
}

// 5. Request/Response messages
message GetUserRequest {
  int32 id = 1;
}
```

**Key Components:**

**1. Syntax Declaration:**

```protobuf
// proto3 (recommended - simpler, more features)
syntax = "proto3";

// proto2 (legacy)
syntax = "proto2";

// Always specify syntax at the top!
```

**2. Package:**

```protobuf
// Package defines namespace
package users.v1;

// Prevents naming conflicts
// Generated code uses package as namespace
// TypeScript: users.v1.User
```

**3. Message Types (Data Structures):**

```protobuf
// Basic message
message User {
  int32 id = 1;        // Field number (unique, don't reuse!)
  string name = 2;
  string email = 3;
  bool is_active = 4;  // Use snake_case
  int64 created_at = 5;
}

// Field numbers:
// - Must be unique within a message
// - Used for binary encoding (not the field name!)
// - 1-15: Single byte (use for common fields)
// - 16-2047: Two bytes
// - Don't reuse deleted field numbers!
```

**4. Scalar Types:**

```protobuf
message Example {
  // Integers
  int32 age = 1;           // -2^31 to 2^31-1
  int64 big_number = 2;    // -2^63 to 2^63-1
  uint32 positive = 3;     // 0 to 2^32-1
  uint64 big_positive = 4; // 0 to 2^64-1
  sint32 signed_int = 5;   // Signed (zigzag encoding)
  
  // Floating point
  float price = 6;         // 32-bit float
  double precise = 7;      // 64-bit double
  
  // Boolean
  bool is_active = 8;      // true/false
  
  // String
  string name = 9;         // UTF-8 or ASCII
  
  // Bytes
  bytes data = 10;         // Arbitrary byte sequence
}

// TypeScript mapping:
// int32, uint32 → number
// int64, uint64 → string or Long
// float, double → number
// bool → boolean
// string → string
// bytes → Uint8Array
```

**5. Repeated Fields (Arrays):**

```protobuf
message User {
  int32 id = 1;
  string name = 2;
  repeated string tags = 3;        // Array of strings
  repeated int32 scores = 4;       // Array of numbers
  repeated Address addresses = 5;  // Array of messages
}

message Address {
  string street = 1;
  string city = 2;
  string country = 3;
}

// TypeScript:
interface User {
  id: number;
  name: string;
  tags: string[];         // Array!
  scores: number[];       // Array!
  addresses: Address[];   // Array of objects!
}
```

**6. Optional Fields:**

```protobuf
// proto3 - all fields are optional by default
message User {
  int32 id = 1;              // Optional (default: 0)
  string name = 2;           // Optional (default: "")
  string email = 3;          // Optional (default: "")
}

// Explicit optional (proto3)
message User {
  int32 id = 1;
  string name = 2;
  optional string email = 3;  // Explicitly optional
}

// Default values (proto3):
// Numbers: 0
// Strings: ""
// Booleans: false
// Enums: first value (must be 0)
// Messages: null/undefined
```

**7. Enums:**

```protobuf
enum UserRole {
  // First value MUST be 0 (proto3)
  USER_ROLE_UNSPECIFIED = 0;  // Default value
  USER_ROLE_ADMIN = 1;
  USER_ROLE_MODERATOR = 2;
  USER_ROLE_USER = 3;
}

message User {
  int32 id = 1;
  string name = 2;
  UserRole role = 3;  // Use enum
}

// TypeScript:
enum UserRole {
  USER_ROLE_UNSPECIFIED = 0,
  USER_ROLE_ADMIN = 1,
  USER_ROLE_MODERATOR = 2,
  USER_ROLE_USER = 3,
}
```

**8. Nested Messages:**

```protobuf
message User {
  int32 id = 1;
  string name = 2;
  
  // Nested message
  message Address {
    string street = 1;
    string city = 2;
    string country = 3;
  }
  
  Address address = 3;  // Use nested type
  repeated Address addresses = 4;
}

// Or define separately
message Address {
  string street = 1;
  string city = 2;
  string country = 3;
}

message User {
  int32 id = 1;
  string name = 2;
  Address address = 3;
}
```

**9. Maps:**

```protobuf
message User {
  int32 id = 1;
  string name = 2;
  
  // Map fields
  map<string, string> metadata = 3;  // key: string, value: string
  map<int32, string> labels = 4;     // key: int32, value: string
  map<string, Address> addresses = 5; // key: string, value: message
}

// TypeScript:
interface User {
  id: number;
  name: string;
  metadata: { [key: string]: string };  // Object/Map
  labels: { [key: number]: string };
  addresses: { [key: string]: Address };
}
```

**10. Oneof (Union Types):**

```protobuf
message PaymentMethod {
  oneof method {
    CreditCard credit_card = 1;
    PayPal paypal = 2;
    BankTransfer bank_transfer = 3;
  }
}

message CreditCard {
  string number = 1;
  string cvv = 2;
}

message PayPal {
  string email = 1;
}

message BankTransfer {
  string account_number = 1;
}

// Only ONE field can be set at a time!
// Like a union type or discriminated union
```

**11. Service Definitions:**

```protobuf
service UsersService {
  // Unary RPC (request-response)
  rpc GetUser (GetUserRequest) returns (User);
  
  // Server streaming
  rpc StreamUsers (Empty) returns (stream User);
  
  // Client streaming
  rpc CreateUsers (stream CreateUserRequest) returns (CreateUsersResponse);
  
  // Bidirectional streaming
  rpc Chat (stream ChatMessage) returns (stream ChatMessage);
}

message GetUserRequest {
  int32 id = 1;
}

message Empty {}

message CreateUserRequest {
  string name = 1;
  string email = 2;
}

message CreateUsersResponse {
  int32 created_count = 1;
}

message ChatMessage {
  string text = 1;
  int64 timestamp = 2;
}
```

**Complete Real-World Example:**

```protobuf
// users.proto
syntax = "proto3";

package users.v1;

// ===== Enums =====
enum UserRole {
  USER_ROLE_UNSPECIFIED = 0;
  USER_ROLE_ADMIN = 1;
  USER_ROLE_MODERATOR = 2;
  USER_ROLE_USER = 3;
}

enum UserStatus {
  USER_STATUS_UNSPECIFIED = 0;
  USER_STATUS_ACTIVE = 1;
  USER_STATUS_INACTIVE = 2;
  USER_STATUS_BANNED = 3;
}

// ===== Messages =====
message User {
  int32 id = 1;
  string name = 2;
  string email = 3;
  UserRole role = 4;
  UserStatus status = 5;
  repeated string tags = 6;
  map<string, string> metadata = 7;
  Address address = 8;
  int64 created_at = 9;
  int64 updated_at = 10;
}

message Address {
  string street = 1;
  string city = 2;
  string state = 3;
  string country = 4;
  string postal_code = 5;
}

// ===== Request/Response Messages =====
message GetUserRequest {
  int32 id = 1;
}

message GetUserResponse {
  User user = 1;
}

message ListUsersRequest {
  int32 page = 1;
  int32 limit = 2;
  optional string search = 3;
  repeated UserRole roles = 4;
  optional UserStatus status = 5;
}

message ListUsersResponse {
  repeated User users = 1;
  int32 total = 2;
  int32 page = 3;
  int32 limit = 4;
}

message CreateUserRequest {
  string name = 1;
  string email = 2;
  string password = 3;
  UserRole role = 4;
  optional Address address = 5;
}

message CreateUserResponse {
  User user = 1;
}

message UpdateUserRequest {
  int32 id = 1;
  optional string name = 2;
  optional string email = 3;
  optional UserRole role = 4;
  optional UserStatus status = 5;
  optional Address address = 6;
}

message UpdateUserResponse {
  User user = 1;
}

message DeleteUserRequest {
  int32 id = 1;
}

message DeleteUserResponse {
  bool success = 1;
}

message StreamUsersRequest {
  optional string filter = 1;
}

message Empty {}

// ===== Service =====
service UsersService {
  // Unary RPCs
  rpc GetUser (GetUserRequest) returns (GetUserResponse);
  rpc ListUsers (ListUsersRequest) returns (ListUsersResponse);
  rpc CreateUser (CreateUserRequest) returns (CreateUserResponse);
  rpc UpdateUser (UpdateUserRequest) returns (UpdateUserResponse);
  rpc DeleteUser (DeleteUserRequest) returns (DeleteUserResponse);
  
  // Server streaming
  rpc StreamUsers (StreamUsersRequest) returns (stream User);
  
  // Client streaming
  rpc BulkCreateUsers (stream CreateUserRequest) returns (CreateUsersResponse);
}

message CreateUsersResponse {
  int32 count = 1;
  repeated User users = 2;
}
```

**Best Practices:**

```protobuf
// ✅ GOOD: Use proto3
syntax = "proto3";

// ✅ GOOD: Always specify package
package myapp.users.v1;  // Include version!

// ✅ GOOD: Use snake_case for fields
message User {
  int32 user_id = 1;        // ✅ snake_case
  string first_name = 2;    // ✅ snake_case
  bool is_active = 3;       // ✅ snake_case
}

// ❌ BAD: camelCase
message User {
  int32 userId = 1;         // ❌ camelCase
  string firstName = 2;     // ❌ camelCase
}

// ✅ GOOD: Use descriptive message names
message GetUserRequest { ... }    // ✅ Clear
message CreateOrderResponse { ... }

// ❌ BAD: Generic names
message Request { ... }    // ❌ Too generic
message Data { ... }

// ✅ GOOD: Field numbers 1-15 for common fields
message User {
  int32 id = 1;            // ✅ Common field, use 1-15
  string name = 2;         // ✅ Common field
  string email = 3;        // ✅ Common field
  repeated string tags = 16;  // Less common, 16+
}

// ✅ GOOD: Never reuse field numbers
message User {
  int32 id = 1;
  string name = 2;
  // string old_field = 3;  // Deleted
  reserved 3;              // ✅ Reserve deleted numbers!
  string new_field = 4;
}

// ✅ GOOD: Use enums for fixed sets
enum Status {
  STATUS_UNSPECIFIED = 0;  // ✅ Always have 0 value
  STATUS_ACTIVE = 1;
  STATUS_INACTIVE = 2;
}

// ✅ GOOD: Version your APIs
package myapp.users.v1;     // v1
package myapp.users.v2;     // v2 (breaking changes)

// ✅ GOOD: Group related RPCs in services
service UsersService {
  rpc GetUser (...) returns (...);
  rpc CreateUser (...) returns (...);
  rpc UpdateUser (...) returns (...);
}

service OrdersService {
  rpc GetOrder (...) returns (...);
  rpc CreateOrder (...) returns (...);
}

// ❌ BAD: One giant service
service ApiService {
  rpc GetUser (...) returns (...);
  rpc GetOrder (...) returns (...);
  rpc GetProduct (...) returns (...);
  // Too many unrelated methods!
}
```

**Imports:**

```protobuf
// users.proto
syntax = "proto3";

package users;

import "common/timestamp.proto";  // Import other .proto files
import "common/address.proto";

message User {
  int32 id = 1;
  string name = 2;
  common.Address address = 3;      // Use imported type
  common.Timestamp created_at = 4; // Use imported type
}
```

**Comments:**

```protobuf
syntax = "proto3";

package users;

// This is a single-line comment

/*
 * This is a multi-line comment
 * Used for detailed explanations
 */

// User represents a user in the system
message User {
  int32 id = 1;           // Unique identifier
  string name = 2;        // Full name
  string email = 3;       // Email address (must be unique)
  bool is_active = 4;     // Whether user is active
}

// UsersService provides user management operations
service UsersService {
  // GetUser retrieves a user by ID
  rpc GetUser (GetUserRequest) returns (User);
}
```

**Generating Code:**

```bash
# Install dependencies
npm install @nestjs/microservices
npm install @grpc/grpc-js @grpc/proto-loader
npm install -D @types/node

# Generate TypeScript code
npx grpc_tools_node_protoc \
  --plugin=protoc-gen-ts=./node_modules/.bin/protoc-gen-ts \
  --ts_out=grpc_js:./src/generated \
  --js_out=import_style=commonjs,binary:./src/generated \
  --grpc_out=grpc_js:./src/generated \
  -I ./proto \
  ./proto/users.proto

# Or use ts-proto
npx protoc \
  --plugin=./node_modules/.bin/protoc-gen-ts_proto \
  --ts_proto_out=./src/generated \
  ./proto/users.proto
```

**NestJS Configuration:**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';
import { join } from 'path';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.GRPC,
      options: {
        package: 'users',  // Must match .proto package
        protoPath: join(__dirname, './users.proto'),
        url: 'localhost:5000',
      },
    },
  );
  await app.listen();
}
bootstrap();
```

**Common Patterns:**

```protobuf
// Pagination
message ListRequest {
  int32 page = 1;
  int32 limit = 2;
  optional string cursor = 3;  // For cursor-based pagination
}

message ListResponse {
  repeated User items = 1;
  int32 total = 2;
  optional string next_cursor = 3;
}

// Filtering/Sorting
message ListUsersRequest {
  optional string search = 1;
  repeated string tags = 2;
  optional string sort_by = 3;    // "name", "created_at", etc.
  optional string sort_order = 4;  // "asc", "desc"
}

// Timestamps
message User {
  int32 id = 1;
  string name = 2;
  int64 created_at = 3;  // Unix timestamp (seconds or milliseconds)
  int64 updated_at = 4;
}

// Soft delete
message User {
  int32 id = 1;
  string name = 2;
  bool is_deleted = 3;
  optional int64 deleted_at = 4;
}

// Partial updates (field mask)
message UpdateUserRequest {
  User user = 1;
  repeated string update_mask = 2;  // ["name", "email"]
}
```

**Interview Tip**: **`.proto` files** define **gRPC contracts** using **Protocol Buffers**. Start with **`syntax = \"proto3\";`**, define **`package`** for namespace. **Messages** are data structures with **field numbers** (unique, don't reuse). **Scalar types**: int32, int64, string, bool, bytes, float, double. **`repeated`** for arrays, **`map`** for key-value. **Enums** must start with **0**. **Services** define **RPC methods**. **Field numbers 1-15** are 1 byte (use for common fields). **`oneof`** for union types. Use **snake_case** for fields, **PascalCase** for messages. **Version APIs** in package name. **Never reuse field numbers** (use `reserved`). **Code generation** creates strongly-typed interfaces. Best for **type safety**, **performance**, **cross-language** compatibility.

</details>

### 33. How do you implement gRPC services in NestJS?

<details>
<summary>Answer</summary>

Implementing **gRPC services in NestJS** involves: **defining .proto files**, **configuring gRPC transport**, **implementing controllers with decorators**, and **setting up clients** to consume services.

**Step-by-Step Implementation:**

**1. Install Dependencies:**

```bash
# Install required packages
npm install @nestjs/microservices
npm install @grpc/grpc-js @grpc/proto-loader
npm install -D @types/node

# Optional: For TypeScript code generation
npm install -D ts-proto
```

**2. Define .proto File:**

```protobuf
// src/users/users.proto
syntax = "proto3";

package users;

// ===== Messages =====
message User {
  int32 id = 1;
  string name = 2;
  string email = 3;
  string role = 4;
  int64 created_at = 5;
}

message GetUserRequest {
  int32 id = 1;
}

message ListUsersRequest {
  int32 page = 1;
  int32 limit = 2;
  optional string search = 3;
}

message ListUsersResponse {
  repeated User users = 1;
  int32 total = 2;
}

message CreateUserRequest {
  string name = 1;
  string email = 2;
  string password = 3;
}

message UpdateUserRequest {
  int32 id = 1;
  optional string name = 2;
  optional string email = 3;
}

message DeleteUserRequest {
  int32 id = 1;
}

message DeleteUserResponse {
  bool success = 1;
}

message Empty {}

// ===== Service =====
service UsersService {
  // Unary RPCs
  rpc GetUser (GetUserRequest) returns (User);
  rpc ListUsers (ListUsersRequest) returns (ListUsersResponse);
  rpc CreateUser (CreateUserRequest) returns (User);
  rpc UpdateUser (UpdateUserRequest) returns (User);
  rpc DeleteUser (DeleteUserRequest) returns (DeleteUserResponse);
  
  // Server streaming
  rpc StreamUsers (Empty) returns (stream User);
}
```

**3. Create gRPC Microservice (Server):**

```typescript
// ===== main.ts =====
import { NestFactory } from '@nestjs/core';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';
import { AppModule } from './app.module';
import { join } from 'path';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.GRPC,
      options: {
        package: 'users',  // Must match .proto package
        protoPath: join(__dirname, './users/users.proto'),
        url: '0.0.0.0:5000',  // gRPC server address
        loader: {
          keepCase: true,  // Keep field names as-is
          longs: String,   // Convert int64 to string
          enums: String,   // Convert enums to string
          defaults: true,  // Set default values
          oneofs: true,    // Include oneof fields
        },
      },
    },
  );
  
  await app.listen();
  console.log('gRPC microservice listening on port 5000');
}
bootstrap();
```

**4. Implement gRPC Controller:**

```typescript
// ===== users.controller.ts =====
import { Controller } from '@nestjs/common';
import { GrpcMethod, GrpcStreamMethod, Payload } from '@nestjs/microservices';
import { Observable, from } from 'rxjs';
import { UsersService } from './users.service';

// Interface definitions (auto-generated or manual)
interface User {
  id: number;
  name: string;
  email: string;
  role: string;
  created_at: number;
}

interface GetUserRequest {
  id: number;
}

interface ListUsersRequest {
  page: number;
  limit: number;
  search?: string;
}

interface ListUsersResponse {
  users: User[];
  total: number;
}

interface CreateUserRequest {
  name: string;
  email: string;
  password: string;
}

interface UpdateUserRequest {
  id: number;
  name?: string;
  email?: string;
}

interface DeleteUserRequest {
  id: number;
}

interface DeleteUserResponse {
  success: boolean;
}

@Controller()
export class UsersController {
  constructor(private readonly usersService: UsersService) {}
  
  // ===== Unary RPC: GetUser =====
  @GrpcMethod('UsersService', 'GetUser')
  async getUser(@Payload() data: GetUserRequest): Promise<User> {
    console.log('GetUser called with:', data);
    
    const user = await this.usersService.findById(data.id);
    
    if (!user) {
      throw new Error('User not found');
    }
    
    return {
      id: user.id,
      name: user.name,
      email: user.email,
      role: user.role,
      created_at: user.createdAt.getTime(),
    };
  }
  
  // ===== Unary RPC: ListUsers =====
  @GrpcMethod('UsersService', 'ListUsers')
  async listUsers(@Payload() data: ListUsersRequest): Promise<ListUsersResponse> {
    console.log('ListUsers called with:', data);
    
    const { page = 1, limit = 10, search } = data;
    
    const [users, total] = await this.usersService.findAll(
      page,
      limit,
      search,
    );
    
    return {
      users: users.map(user => ({
        id: user.id,
        name: user.name,
        email: user.email,
        role: user.role,
        created_at: user.createdAt.getTime(),
      })),
      total,
    };
  }
  
  // ===== Unary RPC: CreateUser =====
  @GrpcMethod('UsersService', 'CreateUser')
  async createUser(@Payload() data: CreateUserRequest): Promise<User> {
    console.log('CreateUser called with:', data);
    
    const user = await this.usersService.create({
      name: data.name,
      email: data.email,
      password: data.password,
    });
    
    return {
      id: user.id,
      name: user.name,
      email: user.email,
      role: user.role,
      created_at: user.createdAt.getTime(),
    };
  }
  
  // ===== Unary RPC: UpdateUser =====
  @GrpcMethod('UsersService', 'UpdateUser')
  async updateUser(@Payload() data: UpdateUserRequest): Promise<User> {
    console.log('UpdateUser called with:', data);
    
    const user = await this.usersService.update(data.id, {
      name: data.name,
      email: data.email,
    });
    
    return {
      id: user.id,
      name: user.name,
      email: user.email,
      role: user.role,
      created_at: user.createdAt.getTime(),
    };
  }
  
  // ===== Unary RPC: DeleteUser =====
  @GrpcMethod('UsersService', 'DeleteUser')
  async deleteUser(@Payload() data: DeleteUserRequest): Promise<DeleteUserResponse> {
    console.log('DeleteUser called with:', data);
    
    await this.usersService.delete(data.id);
    
    return { success: true };
  }
  
  // ===== Server Streaming: StreamUsers =====
  @GrpcStreamMethod('UsersService', 'StreamUsers')
  streamUsers(): Observable<User> {
    console.log('StreamUsers called');
    
    // Return Observable that emits users over time
    return from(
      this.usersService.findAll(1, 1000).then(([users]) =>
        users.map(user => ({
          id: user.id,
          name: user.name,
          email: user.email,
          role: user.role,
          created_at: user.createdAt.getTime(),
        })),
      ),
    );
  }
}
```

**5. Create gRPC Client:**

```typescript
// ===== app.module.ts (Gateway/Client) =====
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';
import { join } from 'path';
import { GatewayController } from './gateway.controller';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'USERS_PACKAGE',
        transport: Transport.GRPC,
        options: {
          package: 'users',
          protoPath: join(__dirname, './users/users.proto'),
          url: 'localhost:5000',
        },
      },
    ]),
  ],
  controllers: [GatewayController],
})
export class AppModule {}
```

**6. Use gRPC Client:**

```typescript
// ===== gateway.controller.ts =====
import { Controller, Get, Post, Put, Delete, Param, Body, Inject, OnModuleInit } from '@nestjs/common';
import { ClientGrpc } from '@nestjs/microservices';
import { firstValueFrom, Observable } from 'rxjs';

// Define service interface
interface UsersService {
  getUser(data: { id: number }): Observable<any>;
  listUsers(data: { page: number; limit: number; search?: string }): Observable<any>;
  createUser(data: { name: string; email: string; password: string }): Observable<any>;
  updateUser(data: { id: number; name?: string; email?: string }): Observable<any>;
  deleteUser(data: { id: number }): Observable<any>;
  streamUsers(data: {}): Observable<any>;
}

@Controller('users')
export class GatewayController implements OnModuleInit {
  private usersService: UsersService;
  
  constructor(
    @Inject('USERS_PACKAGE') private client: ClientGrpc,
  ) {}
  
  onModuleInit() {
    // Get the service from the client
    this.usersService = this.client.getService<UsersService>('UsersService');
  }
  
  @Get(':id')
  async getUser(@Param('id') id: string) {
    const user = await firstValueFrom(
      this.usersService.getUser({ id: parseInt(id) }),
    );
    return user;
  }
  
  @Get()
  async listUsers(
    @Query('page') page = '1',
    @Query('limit') limit = '10',
    @Query('search') search?: string,
  ) {
    const result = await firstValueFrom(
      this.usersService.listUsers({
        page: parseInt(page),
        limit: parseInt(limit),
        search,
      }),
    );
    return result;
  }
  
  @Post()
  async createUser(@Body() userData: any) {
    const user = await firstValueFrom(
      this.usersService.createUser(userData),
    );
    return user;
  }
  
  @Put(':id')
  async updateUser(@Param('id') id: string, @Body() userData: any) {
    const user = await firstValueFrom(
      this.usersService.updateUser({
        id: parseInt(id),
        ...userData,
      }),
    );
    return user;
  }
  
  @Delete(':id')
  async deleteUser(@Param('id') id: string) {
    const result = await firstValueFrom(
      this.usersService.deleteUser({ id: parseInt(id) }),
    );
    return result;
  }
  
  @Get('stream')
  async streamUsers() {
    const users: any[] = [];
    
    return new Promise((resolve, reject) => {
      this.usersService.streamUsers({}).subscribe({
        next: (user) => {
          console.log('Received user:', user);
          users.push(user);
        },
        error: (err) => reject(err),
        complete: () => resolve(users),
      });
    });
  }
}
```

**7. Hybrid Application (HTTP + gRPC):**

```typescript
// ===== main.ts (Hybrid) =====
import { NestFactory } from '@nestjs/core';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';
import { AppModule } from './app.module';
import { join } from 'path';

async function bootstrap() {
  // Create HTTP application
  const app = await NestFactory.create(AppModule);
  
  // Add gRPC microservice
  app.connectMicroservice<MicroserviceOptions>({
    transport: Transport.GRPC,
    options: {
      package: 'users',
      protoPath: join(__dirname, './users/users.proto'),
      url: '0.0.0.0:5000',
    },
  });
  
  // Start both HTTP and gRPC
  await app.startAllMicroservices();
  await app.listen(3000);
  
  console.log('HTTP server listening on port 3000');
  console.log('gRPC server listening on port 5000');
}
bootstrap();
```

**8. Error Handling:**

```typescript
import { RpcException } from '@nestjs/microservices';
import { status } from '@grpc/grpc-js';

@Controller()
export class UsersController {
  @GrpcMethod('UsersService', 'GetUser')
  async getUser(@Payload() data: GetUserRequest): Promise<User> {
    const user = await this.usersService.findById(data.id);
    
    if (!user) {
      // Throw gRPC error
      throw new RpcException({
        code: status.NOT_FOUND,
        message: 'User not found',
      });
    }
    
    return user;
  }
  
  @GrpcMethod('UsersService', 'CreateUser')
  async createUser(@Payload() data: CreateUserRequest): Promise<User> {
    try {
      return await this.usersService.create(data);
    } catch (error) {
      if (error.code === 'DUPLICATE_EMAIL') {
        throw new RpcException({
          code: status.ALREADY_EXISTS,
          message: 'Email already exists',
        });
      }
      
      throw new RpcException({
        code: status.INTERNAL,
        message: 'Internal server error',
      });
    }
  }
}

// Client error handling
@Get(':id')
async getUser(@Param('id') id: string) {
  try {
    const user = await firstValueFrom(
      this.usersService.getUser({ id: parseInt(id) }),
    );
    return user;
  } catch (error) {
    if (error.code === status.NOT_FOUND) {
      throw new NotFoundException('User not found');
    }
    throw new InternalServerErrorException('Service unavailable');
  }
}
```

**9. Advanced Features - Metadata:**

```typescript
import { Metadata } from '@grpc/grpc-js';

// Server: Access metadata
@GrpcMethod('UsersService', 'GetUser')
async getUser(
  @Payload() data: GetUserRequest,
  @Metadata() metadata: Metadata,
): Promise<User> {
  // Read metadata
  const authToken = metadata.get('authorization')[0] as string;
  const userId = metadata.get('user-id')[0] as string;
  
  console.log('Auth token:', authToken);
  console.log('User ID:', userId);
  
  return this.usersService.findById(data.id);
}

// Client: Send metadata
onModuleInit() {
  this.usersService = this.client.getService<UsersService>('UsersService');
}

async getUser(id: number) {
  const metadata = new Metadata();
  metadata.set('authorization', 'Bearer token123');
  metadata.set('user-id', '123');
  
  const user = await firstValueFrom(
    this.usersService.getUser({ id }, metadata),
  );
  return user;
}
```

**10. Client Streaming:**

```protobuf
// users.proto
service UsersService {
  rpc BulkCreateUsers (stream CreateUserRequest) returns (BulkCreateResponse);
}

message BulkCreateResponse {
  int32 count = 1;
  repeated User users = 2;
}
```

```typescript
// Server
import { GrpcStreamCall } from '@nestjs/microservices';
import { Observable } from 'rxjs';
import { toArray, switchMap } from 'rxjs/operators';

@GrpcStreamCall('UsersService', 'BulkCreateUsers')
bulkCreateUsers(
  stream: Observable<CreateUserRequest>,
): Observable<BulkCreateResponse> {
  return stream.pipe(
    toArray(),
    switchMap(async (users) => {
      const createdUsers = await this.usersService.createMany(users);
      return {
        count: createdUsers.length,
        users: createdUsers,
      };
    }),
  );
}

// Client
import { Subject } from 'rxjs';

async bulkCreateUsers(usersData: any[]) {
  const stream = new Subject<CreateUserRequest>();
  
  const result$ = this.usersService.bulkCreateUsers(stream.asObservable());
  
  // Send data
  usersData.forEach(user => stream.next(user));
  stream.complete();
  
  // Wait for response
  return await firstValueFrom(result$);
}
```

**11. Bidirectional Streaming:**

```protobuf
service ChatService {
  rpc Chat (stream ChatMessage) returns (stream ChatMessage);
}

message ChatMessage {
  string user_id = 1;
  string text = 2;
  int64 timestamp = 3;
}
```

```typescript
// Server
@GrpcStreamCall('ChatService', 'Chat')
chat(messages: Observable<ChatMessage>): Observable<ChatMessage> {
  return messages.pipe(
    map(msg => {
      console.log('Received:', msg);
      
      // Echo back with modification
      return {
        user_id: 'server',
        text: `Echo: ${msg.text}`,
        timestamp: Date.now(),
      };
    }),
  );
}

// Client
import { Subject } from 'rxjs';

async startChat() {
  const stream = new Subject<ChatMessage>();
  
  const response$ = this.chatService.chat(stream.asObservable());
  
  // Listen to responses
  response$.subscribe({
    next: (msg) => console.log('Received:', msg),
    error: (err) => console.error('Error:', err),
    complete: () => console.log('Chat ended'),
  });
  
  // Send messages
  stream.next({ user_id: 'client', text: 'Hello', timestamp: Date.now() });
  stream.next({ user_id: 'client', text: 'World', timestamp: Date.now() });
  stream.complete();
}
```

**Best Practices:**

```typescript
// ✅ GOOD: Use interfaces for type safety
interface User {
  id: number;
  name: string;
  email: string;
}

// ✅ GOOD: Validate input
@GrpcMethod('UsersService', 'CreateUser')
async createUser(@Payload() data: CreateUserRequest): Promise<User> {
  if (!data.email || !data.name) {
    throw new RpcException({
      code: status.INVALID_ARGUMENT,
      message: 'Email and name are required',
    });
  }
  
  return this.usersService.create(data);
}

// ✅ GOOD: Use proper error codes
throw new RpcException({
  code: status.NOT_FOUND,        // 404 equivalent
  code: status.ALREADY_EXISTS,   // 409 equivalent
  code: status.INVALID_ARGUMENT, // 400 equivalent
  code: status.UNAUTHENTICATED,  // 401 equivalent
  code: status.PERMISSION_DENIED,// 403 equivalent
  code: status.INTERNAL,         // 500 equivalent
  message: 'Error message',
});

// ✅ GOOD: Handle connection errors
onModuleInit() {
  try {
    this.usersService = this.client.getService<UsersService>('UsersService');
  } catch (error) {
    console.error('Failed to connect to gRPC service:', error);
  }
}

// ✅ GOOD: Use timeouts
import { timeout, catchError } from 'rxjs/operators';

async getUser(id: number) {
  try {
    const user = await firstValueFrom(
      this.usersService.getUser({ id }).pipe(
        timeout(5000),  // 5 second timeout
        catchError(error => {
          console.error('gRPC call failed:', error);
          throw new ServiceUnavailableException('User service unavailable');
        }),
      ),
    );
    return user;
  } catch (error) {
    // Handle error
  }
}

// ✅ GOOD: Log gRPC calls
@GrpcMethod('UsersService', 'GetUser')
async getUser(@Payload() data: GetUserRequest): Promise<User> {
  const startTime = Date.now();
  
  try {
    const user = await this.usersService.findById(data.id);
    
    console.log(`GetUser(${data.id}) completed in ${Date.now() - startTime}ms`);
    
    return user;
  } catch (error) {
    console.error(`GetUser(${data.id}) failed:`, error);
    throw error;
  }
}
```

**Complete Project Structure:**

```
src/
├── users/
│   ├── users.proto              # Protocol buffer definition
│   ├── users.controller.ts       # gRPC controller
│   ├── users.service.ts          # Business logic
│   └── users.module.ts           # Module
├── gateway/
│   ├── gateway.controller.ts     # HTTP gateway
│   └── gateway.module.ts         # Gateway module
├── app.module.ts                 # Root module
└── main.ts                       # Bootstrap
```

**Interview Tip**: Implement gRPC in NestJS: **1) Define .proto files** (messages, services), **2) Install packages** (@nestjs/microservices, @grpc/grpc-js), **3) Create microservice** with Transport.GRPC, **4) Use @GrpcMethod()** for unary RPCs, **@GrpcStreamMethod()** for server streaming, **@GrpcStreamCall()** for client/bidirectional streaming, **5) Register ClientsModule** for clients, **6) Use ClientGrpc.getService()** to get service, **7) Use firstValueFrom()** to convert Observable to Promise. **Error handling** with RpcException and gRPC status codes. **Metadata** for auth/headers. **Hybrid apps** support both HTTP and gRPC. **Type safety** with interfaces. **Streaming** supports 4 patterns (unary, server, client, bidirectional).

</details>

### 34. What are the advantages of gRPC (type safety, performance)?

<details>
<summary>Answer</summary>

**gRPC advantages** include: **strong type safety**, **high performance**, **efficient serialization**, **bidirectional streaming**, **automatic code generation**, **language interoperability**, and **built-in features** like authentication, load balancing, and error handling.

**Key Advantages:**

**1. Type Safety:**

```typescript
// ===== Strong Type Safety with .proto =====

// .proto definition
syntax = "proto3";

message User {
  int32 id = 1;        // Type enforced!
  string name = 2;
  string email = 3;
  bool is_active = 4;
}

service UsersService {
  rpc GetUser (GetUserRequest) returns (User);  // Contract enforced!
}

// ✅ Auto-generated TypeScript interfaces
interface User {
  id: number;          // Strongly typed
  name: string;
  email: string;
  is_active: boolean;
}

interface GetUserRequest {
  id: number;
}

// ✅ Compile-time type checking
@GrpcMethod('UsersService', 'GetUser')
async getUser(@Payload() data: GetUserRequest): Promise<User> {
  // TypeScript ensures you return correct type!
  return {
    id: data.id,
    name: 'John',
    email: 'john@example.com',
    is_active: true,  // Must match User interface
  };
}

// ❌ REST - No type enforcement
@Get(':id')
getUser(@Param('id') id: string): any {  // any type!
  return {
    id: id,              // Could be wrong type
    name: 'John',
    email: 123,          // Type error not caught!
    isActive: 'yes',     // Wrong field name!
  };
}

// ✅ Benefits:
// - Compile-time errors (not runtime)
// - Auto-completion in IDE
// - Refactoring safety
// - Cross-language consistency
// - Contract-first development
```

**2. High Performance:**

```typescript
// ===== Protocol Buffers vs JSON =====

// User object
const user = {
  id: 12345,
  name: 'John Doe',
  email: 'john.doe@example.com',
  is_active: true,
  created_at: 1640000000000,
};

// JSON serialization
const json = JSON.stringify(user);
console.log('JSON size:', json.length);  // ~120 bytes
console.log('JSON:', json);
// {"id":12345,"name":"John Doe","email":"john.doe@example.com","is_active":true,"created_at":1640000000000}

// Protobuf serialization
const protobuf = serializeUser(user);
console.log('Protobuf size:', protobuf.length);  // ~50 bytes
console.log('Protobuf:', protobuf);
// [Binary data - not human readable]

// ✅ Protobuf advantages:
// - 50-60% smaller payloads
// - 3-10x faster serialization
// - 3-10x faster deserialization
// - Lower CPU usage
// - Lower network bandwidth
// - Lower memory usage

// Performance benchmark:
const iterations = 100000;

// JSON
const jsonStart = Date.now();
for (let i = 0; i < iterations; i++) {
  JSON.stringify(user);
  JSON.parse(json);
}
const jsonTime = Date.now() - jsonStart;
console.log('JSON time:', jsonTime, 'ms');  // ~500ms

// Protobuf
const protobufStart = Date.now();
for (let i = 0; i < iterations; i++) {
  serializeUser(user);
  deserializeUser(protobuf);
}
const protobufTime = Date.now() - protobufStart;
console.log('Protobuf time:', protobufTime, 'ms');  // ~100ms

// ✅ Protobuf is 5x faster!
```

**3. HTTP/2 Benefits:**

```typescript
// ===== HTTP/2 Multiplexing =====

// ❌ HTTP/1.1 REST - Multiple connections
await fetch('http://api.com/users/1');      // Connection 1
await fetch('http://api.com/users/2');      // Connection 2
await fetch('http://api.com/users/3');      // Connection 3
// 3 separate TCP connections!
// More latency, more overhead

// ✅ gRPC HTTP/2 - Single connection
const user1 = client.getUser({ id: 1 });   // Stream 1
const user2 = client.getUser({ id: 2 });   // Stream 2
const user3 = client.getUser({ id: 3 });   // Stream 3
// All use SAME TCP connection!
// Less latency, less overhead

// ✅ HTTP/2 features:
// - Multiplexing (multiple requests on one connection)
// - Header compression (smaller overhead)
// - Server push (proactive resource sending)
// - Binary framing (efficient parsing)
// - Stream prioritization

// Performance comparison:
// HTTP/1.1: 10 requests = ~500ms (sequential + connection overhead)
// HTTP/2:   10 requests = ~150ms (parallel on single connection)
```

**4. Bidirectional Streaming:**

```typescript
// ===== 4 Communication Patterns =====

// 1. Unary (Request-Response)
rpc GetUser (GetUserRequest) returns (User);

@GrpcMethod('UsersService', 'GetUser')
async getUser(data: GetUserRequest): Promise<User> {
  return user;
}

// 2. Server Streaming (one request, stream responses)
rpc StreamUsers (Empty) returns (stream User);

@GrpcStreamMethod('UsersService', 'StreamUsers')
streamUsers(): Observable<User> {
  return from([user1, user2, user3]);  // Stream users over time
}

// Use case: Real-time updates, large datasets
// Example: Stock prices, live feeds, notifications

// 3. Client Streaming (stream requests, one response)
rpc BulkCreate (stream CreateUserRequest) returns (BulkResponse);

@GrpcStreamCall('UsersService', 'BulkCreate')
bulkCreate(stream: Observable<CreateUserRequest>): Observable<BulkResponse> {
  return stream.pipe(
    toArray(),
    switchMap(users => this.createMany(users)),
  );
}

// Use case: File uploads, batch operations

// 4. Bidirectional Streaming (stream both ways)
rpc Chat (stream Message) returns (stream Message);

@GrpcStreamCall('ChatService', 'Chat')
chat(messages: Observable<Message>): Observable<Message> {
  return messages.pipe(
    map(msg => this.processMessage(msg)),
  );
}

// Use case: Real-time chat, gaming, collaborative editing

// ❌ REST limitations:
// - No native streaming
// - Workarounds: Server-Sent Events (SSE), WebSocket
// - More complex, less standardized
```

**5. Automatic Code Generation:**

```typescript
// ===== Code Generation =====

// Define .proto once
syntax = "proto3";

message User {
  int32 id = 1;
  string name = 2;
}

service UsersService {
  rpc GetUser (GetUserRequest) returns (User);
}

// ✅ Generate code for multiple languages
$ protoc --ts_out=./typescript users.proto    // TypeScript
$ protoc --go_out=./golang users.proto        // Go
$ protoc --java_out=./java users.proto        // Java
$ protoc --python_out=./python users.proto    // Python
$ protoc --csharp_out=./csharp users.proto    // C#

// ✅ Generated files include:
// - Message interfaces
// - Service stubs (client)
// - Service base classes (server)
// - Serialization/deserialization code
// - Type definitions

// ✅ Benefits:
// - Consistent contracts across languages
// - No manual interface writing
// - Automatic updates when .proto changes
// - Less human error
// - Faster development

// ❌ REST:
// - Manual interface definition
// - No automatic client generation (or separate tools)
// - Inconsistent implementations
// - More maintenance overhead
```

**6. Language Interoperability:**

```typescript
// ===== Polyglot Microservices =====

// Same .proto file for all services

// TypeScript Service (NestJS)
@GrpcMethod('UsersService', 'GetUser')
async getUser(data: GetUserRequest): Promise<User> {
  return this.usersService.findById(data.id);
}

// Go Service
func (s *server) GetUser(ctx context.Context, req *GetUserRequest) (*User, error) {
  return s.usersService.FindById(req.Id)
}

// Java Service
public User getUser(GetUserRequest request) {
  return usersService.findById(request.getId());
}

// Python Service
def GetUser(self, request, context):
  return self.users_service.find_by_id(request.id)

// ✅ All services communicate seamlessly!
// - Same contract (.proto)
// - Same data format (Protobuf)
// - Same error handling (gRPC status codes)
// - Type safety in each language

// ❌ REST with JSON:
// - Each service defines own interfaces
// - No compile-time checking across services
// - Manual validation needed
// - Type mismatches only found at runtime
```

**7. Built-in Features:**

```typescript
// ===== Authentication & Metadata =====

import { Metadata } from '@grpc/grpc-js';

// Client: Send auth token
const metadata = new Metadata();
metadata.set('authorization', 'Bearer token123');

const user = await client.getUser({ id: 1 }, metadata);

// Server: Validate auth
@GrpcMethod('UsersService', 'GetUser')
async getUser(
  @Payload() data: GetUserRequest,
  @Metadata() metadata: Metadata,
): Promise<User> {
  const token = metadata.get('authorization')[0];
  
  if (!this.authService.validateToken(token)) {
    throw new RpcException({
      code: status.UNAUTHENTICATED,
      message: 'Invalid token',
    });
  }
  
  return this.usersService.findById(data.id);
}

// ===== Load Balancing =====

// gRPC has built-in client-side load balancing
{
  transport: Transport.GRPC,
  options: {
    url: 'dns:///users-service:5000',  // DNS-based load balancing
    // Or multiple addresses
    url: 'ipv4:10.0.0.1:5000,10.0.0.2:5000,10.0.0.3:5000',
  },
}

// ===== Retry & Timeout =====

// Built-in retry configuration
{
  transport: Transport.GRPC,
  options: {
    url: 'localhost:5000',
    maxSendMessageLength: 1024 * 1024 * 10,  // 10MB
    maxReceiveMessageLength: 1024 * 1024 * 10,
    keepalive: {
      keepaliveTimeMs: 30000,
      keepaliveTimeoutMs: 5000,
      keepalivePermitWithoutCalls: 1,
    },
  },
}

// Client-side timeout
import { timeout } from 'rxjs/operators';

const user = await firstValueFrom(
  this.usersService.getUser({ id: 1 }).pipe(
    timeout(5000),  // 5 second timeout
  ),
);

// ===== Compression =====

// Automatic header compression (HTTP/2)
// Optional payload compression
{
  transport: Transport.GRPC,
  options: {
    url: 'localhost:5000',
    channelOptions: {
      'grpc.default_compression_algorithm': 1,  // 1 = gzip
      'grpc.default_compression_level': 2,
    },
  },
}
```

**8. Better Error Handling:**

```typescript
// ===== Structured Error Codes =====

import { status } from '@grpc/grpc-js';
import { RpcException } from '@nestjs/microservices';

// Server: Throw specific errors
@GrpcMethod('UsersService', 'GetUser')
async getUser(data: GetUserRequest): Promise<User> {
  const user = await this.usersService.findById(data.id);
  
  if (!user) {
    throw new RpcException({
      code: status.NOT_FOUND,        // Structured code!
      message: 'User not found',
      details: { userId: data.id },
    });
  }
  
  return user;
}

// Client: Handle specific errors
try {
  const user = await firstValueFrom(
    this.usersService.getUser({ id: 1 }),
  );
} catch (error) {
  switch (error.code) {
    case status.NOT_FOUND:
      throw new NotFoundException('User not found');
    case status.UNAUTHENTICATED:
      throw new UnauthorizedException('Not authenticated');
    case status.PERMISSION_DENIED:
      throw new ForbiddenException('Access denied');
    case status.INVALID_ARGUMENT:
      throw new BadRequestException('Invalid data');
    default:
      throw new InternalServerErrorException('Service error');
  }
}

// ✅ gRPC status codes:
// - OK (0)
// - CANCELLED (1)
// - UNKNOWN (2)
// - INVALID_ARGUMENT (3)
// - DEADLINE_EXCEEDED (4)
// - NOT_FOUND (5)
// - ALREADY_EXISTS (6)
// - PERMISSION_DENIED (7)
// - UNAUTHENTICATED (16)
// - And more...

// ❌ REST:
// - HTTP status codes (less granular)
// - No standard error format
// - Manual error handling
```

**9. Lower Latency:**

```typescript
// ===== Latency Comparison =====

// Factors contributing to gRPC's lower latency:

// 1. Binary format (faster parsing)
// JSON parsing: ~5ms per 1000 objects
// Protobuf parsing: ~1ms per 1000 objects

// 2. HTTP/2 multiplexing (no head-of-line blocking)
// HTTP/1.1: Request 1 → Response 1 → Request 2 → Response 2
// HTTP/2:   Request 1 & 2 → Response 1 & 2 (parallel)

// 3. Connection reuse (no handshake overhead)
// HTTP/1.1: New connection per request = 50-100ms overhead
// gRPC: Single persistent connection = 0ms overhead

// 4. Smaller payloads (less network time)
// 1KB JSON vs 400 bytes Protobuf = 60% less network time

// Real-world latency:
// REST/JSON:  50-200ms per request
// gRPC:       10-50ms per request
// Improvement: 75-80% faster!
```

**10. Better for Microservices:**

```typescript
// ===== Microservices Architecture =====

// ✅ gRPC advantages for microservices:

// 1. Service discovery integration
@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'USERS_SERVICE',
        transport: Transport.GRPC,
        options: {
          url: 'dns:///users-service:5000',  // Service name
          package: 'users',
          protoPath: 'users.proto',
        },
      },
    ]),
  ],
})

// 2. Efficient inter-service communication
// Multiple services talking to each other:
// API Gateway → Users Service → Auth Service → Email Service
// All using gRPC = fast, type-safe, consistent

// 3. Contract-first development
// Define .proto → Generate code → Implement
// All teams use same contract

// 4. Versioning support
package users.v1;  // Version 1
package users.v2;  // Version 2 (breaking changes)

// 5. Observability
// Built-in metrics, tracing, logging
// Compatible with OpenTelemetry, Jaeger, Zipkin
```

**Performance Benchmarks:**

```typescript
// ===== Real-World Performance Test =====

// Test: 1000 users list request

// REST/JSON
const restStart = Date.now();
const restResponse = await fetch('http://localhost:3000/users');
const restUsers = await restResponse.json();
const restTime = Date.now() - restStart;

console.log('REST time:', restTime, 'ms');           // ~500ms
console.log('REST payload:', JSON.stringify(restUsers).length);  // ~150KB

// gRPC/Protobuf
const grpcStart = Date.now();
const grpcUsers = await firstValueFrom(
  this.usersService.listUsers({ page: 1, limit: 1000 }),
);
const grpcTime = Date.now() - grpcStart;

console.log('gRPC time:', grpcTime, 'ms');           // ~100ms
console.log('gRPC payload:', serializeUsers(grpcUsers).length);  // ~30KB

// Results:
// ✅ gRPC is 5x faster
// ✅ gRPC payload is 5x smaller
// ✅ gRPC uses 4x less CPU
// ✅ gRPC uses 5x less bandwidth
```

**Advantages Summary:**

```
✅ Type Safety:
  - Strong typing with .proto
  - Compile-time checking
  - Auto-generated interfaces
  - Cross-language consistency
  - Contract-first development

✅ Performance:
  - Binary protocol (50-60% smaller)
  - 3-10x faster serialization
  - Lower CPU usage
  - Lower memory usage
  - Lower network bandwidth

✅ HTTP/2:
  - Multiplexing (single connection)
  - Header compression
  - Binary framing
  - Stream prioritization
  - Lower latency

✅ Streaming:
  - Server streaming
  - Client streaming
  - Bidirectional streaming
  - Real-time communication

✅ Code Generation:
  - Automatic client/server code
  - Multiple languages
  - Consistent contracts
  - Less manual work

✅ Polyglot:
  - Language agnostic
  - Same .proto for all
  - Cross-language type safety
  - Easy integration

✅ Built-in Features:
  - Authentication (metadata)
  - Load balancing
  - Retry logic
  - Timeout handling
  - Compression
  - Health checks

✅ Error Handling:
  - Structured error codes
  - Better than HTTP status
  - Consistent across services

✅ Latency:
  - 75-80% lower than REST
  - Connection reuse
  - Efficient parsing
  - Smaller payloads

✅ Microservices:
  - Perfect for service-to-service
  - Service discovery integration
  - Contract-first
  - Versioning support
  - Better observability
```

**When gRPC Shines:**

```typescript
// ✅ PERFECT for:
// - Microservice-to-microservice communication
// - High-performance requirements
// - Real-time streaming (stock prices, live feeds)
// - Polyglot environments (multiple languages)
// - Large data transfers (efficient serialization)
// - Internal APIs (not browser-facing)
// - Mobile backends (smaller payloads = less data usage)
// - IoT devices (efficient protocol)

// ❌ NOT ideal for:
// - Browser-facing APIs (limited support, use gRPC-Web)
// - Public APIs (REST is more accessible)
// - Simple CRUD operations (REST is simpler)
// - Human-readable data (Protobuf is binary)
// - Quick prototyping (more setup required)
```

**Interview Tip**: **gRPC advantages**: **1) Type safety** - strongly typed contracts, auto-generated code, compile-time checking, **2) Performance** - binary Protobuf (50-60% smaller), 3-10x faster serialization, lower latency, **3) HTTP/2** - multiplexing (single connection), header compression, **4) Streaming** - bidirectional, server, client streaming, **5) Code generation** - automatic for multiple languages, **6) Polyglot** - language agnostic, same contract, **7) Built-in features** - auth, load balancing, retries, **8) Error handling** - structured status codes, **9) Lower latency** - 75-80% faster than REST, **10) Microservices** - perfect for service-to-service. **Best for internal APIs**, **NOT for browsers** (limited support). **5-10x smaller payloads**, **2-5x faster** than REST/JSON.

</details>

## RabbitMQ

### 35. How do you integrate RabbitMQ with NestJS?

<details>
<summary>Answer</summary>

**Integrating RabbitMQ with NestJS** involves: **installing dependencies**, **configuring RabbitMQ transport**, **creating message patterns**, **setting up producers and consumers**, and **handling queues, exchanges, and routing**.

**Step-by-Step Integration:**

**1. Install Dependencies:**

```bash
# Install required packages
npm install @nestjs/microservices
npm install amqplib amqp-connection-manager

# Install types
npm install -D @types/amqplib
```

**2. Start RabbitMQ (Docker):**

```bash
# Run RabbitMQ with management plugin
docker run -d \
  --name rabbitmq \
  -p 5672:5672 \
  -p 15672:15672 \
  rabbitmq:3-management

# Access management UI: http://localhost:15672
# Default credentials: guest/guest
```

**3. Create RabbitMQ Microservice:**

```typescript
// ===== main.ts (Microservice) =====
import { NestFactory } from '@nestjs/core';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.RMQ,  // RabbitMQ transport
      options: {
        urls: ['amqp://localhost:5672'],  // RabbitMQ URL
        queue: 'users_queue',              // Queue name
        queueOptions: {
          durable: true,  // Survive broker restart
        },
        // Connection options
        socketOptions: {
          heartbeatIntervalInSeconds: 60,
          reconnectTimeInSeconds: 5,
        },
      },
    },
  );
  
  await app.listen();
  console.log('RabbitMQ microservice is listening...');
}
bootstrap();
```

**4. Create Message Handlers (Consumer):**

```typescript
// ===== users.controller.ts =====
import { Controller } from '@nestjs/common';
import { MessagePattern, Payload, Ctx, RmqContext } from '@nestjs/microservices';
import { UsersService } from './users.service';

@Controller()
export class UsersController {
  constructor(private readonly usersService: UsersService) {}
  
  // Handle messages with pattern 'users.create'
  @MessagePattern('users.create')
  async createUser(
    @Payload() data: any,
    @Ctx() context: RmqContext,
  ) {
    console.log('Received message:', data);
    
    // Get RabbitMQ channel and message
    const channel = context.getChannelRef();
    const originalMsg = context.getMessage();
    
    try {
      const user = await this.usersService.create(data);
      
      // Manually acknowledge message
      channel.ack(originalMsg);
      
      return user;
    } catch (error) {
      console.error('Error processing message:', error);
      
      // Reject message (requeue or dead letter)
      channel.nack(originalMsg, false, false);
      
      throw error;
    }
  }
  
  @MessagePattern('users.findById')
  async findUser(@Payload() data: { id: number }) {
    return await this.usersService.findById(data.id);
  }
  
  @MessagePattern('users.list')
  async listUsers(@Payload() data: { page: number; limit: number }) {
    return await this.usersService.findAll(data.page, data.limit);
  }
}
```

**5. Configure RabbitMQ Client (Producer):**

```typescript
// ===== app.module.ts (Client/Gateway) =====
import { Module } from '@nestjs/common';
import { ClientsModule, Transport } from '@nestjs/microservices';
import { GatewayController } from './gateway.controller';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'USERS_SERVICE',
        transport: Transport.RMQ,
        options: {
          urls: ['amqp://localhost:5672'],
          queue: 'users_queue',
          queueOptions: {
            durable: true,
          },
        },
      },
    ]),
  ],
  controllers: [GatewayController],
})
export class AppModule {}
```

**6. Send Messages (Producer):**

```typescript
// ===== gateway.controller.ts =====
import { Controller, Post, Get, Body, Param, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { firstValueFrom } from 'rxjs';

@Controller('users')
export class GatewayController {
  constructor(
    @Inject('USERS_SERVICE') private client: ClientProxy,
  ) {}
  
  @Post()
  async createUser(@Body() userData: any) {
    // Send message and wait for response
    const user = await firstValueFrom(
      this.client.send('users.create', userData),
    );
    return user;
  }
  
  @Get(':id')
  async getUser(@Param('id') id: string) {
    const user = await firstValueFrom(
      this.client.send('users.findById', { id: parseInt(id) }),
    );
    return user;
  }
  
  @Get()
  async listUsers(
    @Query('page') page = '1',
    @Query('limit') limit = '10',
  ) {
    const users = await firstValueFrom(
      this.client.send('users.list', {
        page: parseInt(page),
        limit: parseInt(limit),
      }),
    );
    return users;
  }
}
```

**7. Event-Based Communication:**

```typescript
// ===== Event Handler (Consumer) =====
import { EventPattern, Payload } from '@nestjs/microservices';

@Controller()
export class NotificationsController {
  // Listen to events (fire-and-forget)
  @EventPattern('user.created')
  async handleUserCreated(@Payload() data: any) {
    console.log('User created event:', data);
    await this.sendWelcomeEmail(data.email);
    // No return value needed
  }
  
  @EventPattern('order.placed')
  async handleOrderPlaced(@Payload() data: any) {
    console.log('Order placed event:', data);
    await this.sendOrderConfirmation(data);
  }
}

// ===== Event Publisher (Producer) =====
@Controller('users')
export class UsersController {
  constructor(
    @Inject('EVENTS_SERVICE') private eventsClient: ClientProxy,
  ) {}
  
  @Post()
  async createUser(@Body() userData: any) {
    const user = await this.usersService.create(userData);
    
    // Emit event (fire-and-forget)
    this.eventsClient.emit('user.created', {
      id: user.id,
      email: user.email,
      name: user.name,
    });
    
    return user;
  }
}
```

**8. Advanced Configuration:**

```typescript
// ===== Advanced RabbitMQ Options =====

// Server configuration
const app = await NestFactory.createMicroservice<MicroserviceOptions>(
  AppModule,
  {
    transport: Transport.RMQ,
    options: {
      urls: [
        'amqp://user:password@localhost:5672',  // With auth
        'amqp://backup-host:5672',              // Fallback host
      ],
      queue: 'users_queue',
      
      // Queue options
      queueOptions: {
        durable: true,           // Survive broker restart
        exclusive: false,        // Allow multiple consumers
        autoDelete: false,       // Don't delete when unused
        messageTtl: 60000,       // Message TTL (60 seconds)
        maxLength: 10000,        // Max queue length
        deadLetterExchange: 'dlx',  // Dead letter exchange
        deadLetterRoutingKey: 'dead_letters',
      },
      
      // Prefetch count (Quality of Service)
      prefetchCount: 10,  // Process 10 messages at a time
      
      // Manual acknowledgment
      noAck: false,  // Require manual ack
      
      // Connection options
      socketOptions: {
        heartbeatIntervalInSeconds: 60,
        reconnectTimeInSeconds: 5,
      },
      
      // Serialization
      serializer: {
        serialize: (value) => JSON.stringify(value),
      },
      deserializer: {
        deserialize: (value) => JSON.parse(value.toString()),
      },
    },
  },
);
```

**9. Multiple Queues:**

```typescript
// ===== Multiple Queue Configuration =====

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'USERS_SERVICE',
        transport: Transport.RMQ,
        options: {
          urls: ['amqp://localhost:5672'],
          queue: 'users_queue',
          queueOptions: { durable: true },
        },
      },
      {
        name: 'ORDERS_SERVICE',
        transport: Transport.RMQ,
        options: {
          urls: ['amqp://localhost:5672'],
          queue: 'orders_queue',
          queueOptions: { durable: true },
        },
      },
      {
        name: 'NOTIFICATIONS_SERVICE',
        transport: Transport.RMQ,
        options: {
          urls: ['amqp://localhost:5672'],
          queue: 'notifications_queue',
          queueOptions: { durable: true },
        },
      },
    ]),
  ],
})
export class AppModule {}

// Use multiple services
@Controller('api')
export class ApiController {
  constructor(
    @Inject('USERS_SERVICE') private usersClient: ClientProxy,
    @Inject('ORDERS_SERVICE') private ordersClient: ClientProxy,
    @Inject('NOTIFICATIONS_SERVICE') private notificationsClient: ClientProxy,
  ) {}
  
  @Post('checkout')
  async checkout(@Body() checkoutData: any) {
    // Call multiple services
    const user = await firstValueFrom(
      this.usersClient.send('users.findById', { id: checkoutData.userId }),
    );
    
    const order = await firstValueFrom(
      this.ordersClient.send('orders.create', checkoutData),
    );
    
    // Send notification
    this.notificationsClient.emit('order.placed', {
      userId: user.id,
      orderId: order.id,
    });
    
    return order;
  }
}
```

**10. Error Handling:**

```typescript
// ===== Error Handling in RabbitMQ =====

import { RpcException } from '@nestjs/microservices';

// Server: Throw errors
@MessagePattern('users.create')
async createUser(@Payload() data: any, @Ctx() context: RmqContext) {
  const channel = context.getChannelRef();
  const originalMsg = context.getMessage();
  
  try {
    if (!data.email) {
      throw new RpcException('Email is required');
    }
    
    const user = await this.usersService.create(data);
    
    // Acknowledge successful processing
    channel.ack(originalMsg);
    
    return user;
  } catch (error) {
    console.error('Error:', error);
    
    // Reject and requeue for retry
    // channel.nack(originalMsg, false, true);  // Requeue
    
    // Or send to dead letter queue
    channel.nack(originalMsg, false, false);  // Don't requeue
    
    throw error;
  }
}

// Client: Handle errors
@Post()
async createUser(@Body() userData: any) {
  try {
    const user = await firstValueFrom(
      this.client.send('users.create', userData),
    );
    return user;
  } catch (error) {
    if (error instanceof RpcException) {
      throw new BadRequestException(error.message);
    }
    throw new ServiceUnavailableException('User service unavailable');
  }
}
```

**11. Dead Letter Queue (DLQ):**

```typescript
// ===== Dead Letter Queue Configuration =====

// Main queue with DLQ
const app = await NestFactory.createMicroservice<MicroserviceOptions>(
  AppModule,
  {
    transport: Transport.RMQ,
    options: {
      urls: ['amqp://localhost:5672'],
      queue: 'users_queue',
      queueOptions: {
        durable: true,
        deadLetterExchange: 'dlx',           // Dead letter exchange
        deadLetterRoutingKey: 'users.dead',  // DLQ routing key
        messageTtl: 300000,                  // 5 minutes TTL
      },
    },
  },
);

// Dead letter queue consumer
const dlqApp = await NestFactory.createMicroservice<MicroserviceOptions>(
  DlqModule,
  {
    transport: Transport.RMQ,
    options: {
      urls: ['amqp://localhost:5672'],
      queue: 'users_dead_queue',
      queueOptions: {
        durable: true,
      },
    },
  },
);

// DLQ Handler
@Controller()
export class DeadLetterController {
  @MessagePattern('*')  // Catch all patterns
  async handleDeadLetter(@Payload() data: any) {
    console.error('Dead letter received:', data);
    
    // Log to database, alert admins, etc.
    await this.logDeadLetter(data);
  }
}
```

**12. Priority Queues:**

```typescript
// ===== Priority Queue =====

const app = await NestFactory.createMicroservice<MicroserviceOptions>(
  AppModule,
  {
    transport: Transport.RMQ,
    options: {
      urls: ['amqp://localhost:5672'],
      queue: 'priority_queue',
      queueOptions: {
        durable: true,
        maxPriority: 10,  // Enable priority (0-10)
      },
    },
  },
);

// Send message with priority
this.client.send('task.process', data, {
  priority: 9,  // High priority
});

this.client.send('task.process', data, {
  priority: 1,  // Low priority
});
```

**13. Monitoring & Health Checks:**

```typescript
// ===== Health Check =====

import { HealthCheckService, MicroserviceHealthIndicator } from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private microservice: MicroserviceHealthIndicator,
  ) {}
  
  @Get()
  @HealthCheck()
  check() {
    return this.health.check([
      () =>
        this.microservice.pingCheck('rabbitmq', {
          transport: Transport.RMQ,
          options: {
            urls: ['amqp://localhost:5672'],
            queue: 'health_queue',
          },
        }),
    ]);
  }
}
```

**14. Testing RabbitMQ Integration:**

```typescript
// ===== Testing =====

import { Test } from '@nestjs/testing';
import { ClientsModule, Transport } from '@nestjs/microservices';

describe('RabbitMQ Integration', () => {
  let app: INestApplication;
  let client: ClientProxy;
  
  beforeAll(async () => {
    const moduleRef = await Test.createTestingModule({
      imports: [
        ClientsModule.register([
          {
            name: 'USERS_SERVICE',
            transport: Transport.RMQ,
            options: {
              urls: ['amqp://localhost:5672'],
              queue: 'test_queue',
              queueOptions: { durable: false },
            },
          },
        ]),
      ],
    }).compile();
    
    app = moduleRef.createNestApplication();
    client = app.get('USERS_SERVICE');
    
    await app.init();
    await client.connect();
  });
  
  afterAll(async () => {
    await client.close();
    await app.close();
  });
  
  it('should send and receive message', async () => {
    const result = await firstValueFrom(
      client.send('users.create', { name: 'John', email: 'john@example.com' }),
    );
    
    expect(result).toBeDefined();
    expect(result.name).toBe('John');
  });
});
```

**Best Practices:**

```typescript
// ✅ GOOD: Use durable queues for production
queueOptions: {
  durable: true,  // Survive broker restart
}

// ✅ GOOD: Manual acknowledgment for reliability
noAck: false,  // Require manual ack

// ✅ GOOD: Set prefetch count
prefetchCount: 10,  // Prevent overwhelming consumers

// ✅ GOOD: Use dead letter queues
queueOptions: {
  deadLetterExchange: 'dlx',
  deadLetterRoutingKey: 'dead_letters',
}

// ✅ GOOD: Implement error handling
try {
  // Process message
  channel.ack(originalMsg);
} catch (error) {
  channel.nack(originalMsg, false, false);
}

// ✅ GOOD: Use multiple broker URLs for HA
urls: [
  'amqp://primary:5672',
  'amqp://secondary:5672',
]

// ✅ GOOD: Set connection timeouts
socketOptions: {
  heartbeatIntervalInSeconds: 60,
  reconnectTimeInSeconds: 5,
}

// ❌ BAD: Auto-delete queues in production
queueOptions: {
  autoDelete: true,  // Don't use in production!
}

// ❌ BAD: No error handling
@MessagePattern('pattern')
async handle(data: any) {
  // No try-catch, no ack/nack!
  return this.process(data);
}

// ❌ BAD: Using noAck in production
noAck: true,  // Messages can be lost!
```

**Complete Example:**

```typescript
// ===== Complete RabbitMQ Integration =====

// 1. Microservice (main.ts)
async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.RMQ,
      options: {
        urls: ['amqp://localhost:5672'],
        queue: 'users_queue',
        queueOptions: {
          durable: true,
          deadLetterExchange: 'dlx',
        },
        prefetchCount: 10,
        noAck: false,
      },
    },
  );
  await app.listen();
}

// 2. Controller
@Controller()
export class UsersController {
  @MessagePattern('users.create')
  async createUser(@Payload() data: any, @Ctx() context: RmqContext) {
    const channel = context.getChannelRef();
    const originalMsg = context.getMessage();
    
    try {
      const user = await this.usersService.create(data);
      channel.ack(originalMsg);
      return user;
    } catch (error) {
      channel.nack(originalMsg, false, false);
      throw error;
    }
  }
}

// 3. Client Module
@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'USERS_SERVICE',
        transport: Transport.RMQ,
        options: {
          urls: ['amqp://localhost:5672'],
          queue: 'users_queue',
          queueOptions: { durable: true },
        },
      },
    ]),
  ],
})

// 4. Client Usage
@Controller('users')
export class GatewayController {
  constructor(@Inject('USERS_SERVICE') private client: ClientProxy) {}
  
  @Post()
  async createUser(@Body() userData: any) {
    return await firstValueFrom(
      this.client.send('users.create', userData),
    );
  }
}
```

**Interview Tip**: **RabbitMQ integration** in NestJS: **1) Install** @nestjs/microservices, amqplib, **2) Configure** Transport.RMQ with URLs and queue name, **3) Use @MessagePattern()** for request-response, **@EventPattern()** for fire-and-forget, **4) Manual ack/nack** with context.getChannelRef(), **5) Set durable: true** for production, **6) Use prefetchCount** to control concurrency, **7) Dead letter queues** for failed messages, **8) Multiple queues** for different services, **9) Error handling** with try-catch and nack(), **10) Health checks** for monitoring. **Key options**: urls, queue, queueOptions, prefetchCount, noAck. **Best for** reliable message queuing, async processing, decoupled services.

</details>

### ### 36. What is a queue in RabbitMQ?

<details>
<summary>Answer</summary>

**A queue in RabbitMQ** is a **buffer** that **stores messages** until they are consumed by consumer applications. It acts as a **mailbox** where messages wait to be processed, providing **decoupling**, **load balancing**, and **reliability** in distributed systems.

**Key Concepts:**

```
✅ Message buffer/storage
✅ FIFO (First-In-First-Out) order
✅ Multiple consumers possible
✅ Persistent or transient
✅ Bound to exchanges
✅ Can have properties (durable, exclusive, auto-delete)
✅ Supports dead letter queues
✅ Can prioritize messages
```

**Queue Basics:**

```typescript
// ===== Queue Flow =====

Producer → Exchange → Queue → Consumer

// Producer sends message
this.client.send('users.create', { name: 'John' });

// Message goes to exchange
Exchange: 'amq.topic' or default exchange

// Exchange routes to queue based on routing key
Queue: 'users_queue'

// Consumer processes message
@MessagePattern('users.create')
async createUser(data: any) {
  return this.usersService.create(data);
}
```

**Queue Properties:**

```typescript
// ===== Queue Configuration =====

{
  transport: Transport.RMQ,
  options: {
    urls: ['amqp://localhost:5672'],
    queue: 'users_queue',  // Queue name
    
    queueOptions: {
      // ===== Durability =====
      durable: true,  // Queue survives broker restart
      // false: Queue deleted on broker restart
      
      // ===== Exclusivity =====
      exclusive: false,  // Allow multiple connections
      // true: Only one connection allowed, deleted when closed
      
      // ===== Auto-Delete =====
      autoDelete: false,  // Keep queue even if no consumers
      // true: Delete queue when last consumer disconnects
      
      // ===== Message TTL =====
      messageTtl: 60000,  // Messages expire after 60 seconds
      
      // ===== Max Length =====
      maxLength: 10000,  // Max 10,000 messages in queue
      // Older messages dropped when limit reached
      
      // ===== Max Priority =====
      maxPriority: 10,  // Enable priority (0-10)
      
      // ===== Dead Letter =====
      deadLetterExchange: 'dlx',  // Where failed messages go
      deadLetterRoutingKey: 'dead_letters',
      
      // ===== Other Options =====
      arguments: {
        'x-queue-mode': 'lazy',  // Lazy queue (disk-based)
        'x-max-length-bytes': 1048576,  // 1MB max size
      },
    },
  },
}
```

**Types of Queues:**

**1. Durable Queues:**

```typescript
// ===== Durable Queue (survives restart) =====

queueOptions: {
  durable: true,  // ✅ Queue persists on disk
}

// Use case: Production environments, important messages
// - Queue survives RabbitMQ broker restart
// - Messages marked as persistent also survive
// - Slight performance cost for disk writes

// ✅ GOOD for production:
{
  queue: 'orders_queue',
  queueOptions: {
    durable: true,  // Don't lose queue on restart
  },
}

// ❌ BAD for production:
{
  queue: 'temp_queue',
  queueOptions: {
    durable: false,  // Queue lost on restart!
  },
}
```

**2. Transient Queues:**

```typescript
// ===== Transient Queue (deleted on restart) =====

queueOptions: {
  durable: false,  // Queue lost on broker restart
}

// Use case: Temporary data, testing, development
// - Faster (no disk writes)
// - Lost on broker restart
// - Good for disposable data

// Testing/development
{
  queue: 'test_queue',
  queueOptions: {
    durable: false,  // OK for testing
  },
}
```

**3. Exclusive Queues:**

```typescript
// ===== Exclusive Queue (single connection) =====

queueOptions: {
  exclusive: true,  // Only one connection allowed
}

// Characteristics:
// - Only one consumer can connect
// - Automatically deleted when connection closes
// - Used for temporary, private queues

// Use case: Request-reply pattern, temporary callbacks
const replyQueue = await channel.assertQueue('', {
  exclusive: true,  // Private reply queue
  autoDelete: true,
});
```

**4. Auto-Delete Queues:**

```typescript
// ===== Auto-Delete Queue =====

queueOptions: {
  autoDelete: true,  // Delete when last consumer disconnects
}

// Use case: Temporary queues, dynamic consumers
// - Queue deleted when last consumer disconnects
// - Good for short-lived consumers

// ❌ BAD for production (can lose messages):
{
  queue: 'important_queue',
  queueOptions: {
    autoDelete: true,  // Dangerous! Can delete unexpectedly
  },
}
```

**Queue Operations:**

**1. Creating Queues:**

```typescript
// NestJS automatically creates queue
const app = await NestFactory.createMicroservice<MicroserviceOptions>(
  AppModule,
  {
    transport: Transport.RMQ,
    options: {
      urls: ['amqp://localhost:5672'],
      queue: 'users_queue',  // Auto-created if doesn't exist
      queueOptions: {
        durable: true,
      },
    },
  },
);

// Manual queue creation (advanced)
import * as amqp from 'amqplib';

const connection = await amqp.connect('amqp://localhost:5672');
const channel = await connection.createChannel();

// Assert queue (create if doesn't exist)
await channel.assertQueue('users_queue', {
  durable: true,
  exclusive: false,
  autoDelete: false,
});
```

**2. Publishing to Queues:**

```typescript
// ===== Sending Messages =====

// Request-response
const user = await firstValueFrom(
  this.client.send('users.create', { name: 'John' }),
);

// Fire-and-forget
this.client.emit('user.created', { id: 1, name: 'John' });

// With options
this.client.send('task.process', data, {
  priority: 9,        // Message priority
  expiration: '60000', // TTL (60 seconds)
  persistent: true,   // Survive broker restart
});
```

**3. Consuming from Queues:**

```typescript
// ===== Consuming Messages =====

// Simple consumer
@MessagePattern('users.create')
async createUser(@Payload() data: any) {
  return await this.usersService.create(data);
}

// With manual acknowledgment
@MessagePattern('users.create')
async createUser(@Payload() data: any, @Ctx() context: RmqContext) {
  const channel = context.getChannelRef();
  const originalMsg = context.getMessage();
  
  try {
    const user = await this.usersService.create(data);
    
    // Acknowledge message (remove from queue)
    channel.ack(originalMsg);
    
    return user;
  } catch (error) {
    // Reject message
    // nack(message, allUpTo, requeue)
    channel.nack(originalMsg, false, false);
    
    throw error;
  }
}

// Multiple consumers on same queue (load balancing)
// Consumer 1
@MessagePattern('task.process')
async processTask1(data: any) {
  console.log('Consumer 1 processing:', data);
}

// Consumer 2
@MessagePattern('task.process')
async processTask2(data: any) {
  console.log('Consumer 2 processing:', data);
}

// Messages distributed round-robin between consumers
```

**Message Acknowledgment:**

```typescript
// ===== Acknowledgment Modes =====

// 1. Auto-Ack (automatic acknowledgment)
noAck: true,  // ❌ Not recommended for production
// - Message removed immediately when delivered
// - No guarantee of processing
// - Message lost if consumer crashes

// 2. Manual Ack (explicit acknowledgment)
noAck: false,  // ✅ Recommended for production
// - Message removed only after explicit ack
// - Redelivered if consumer crashes
// - Reliable processing

@MessagePattern('task.process')
async processTask(@Payload() data: any, @Ctx() context: RmqContext) {
  const channel = context.getChannelRef();
  const message = context.getMessage();
  
  try {
    // Process message
    await this.heavyProcessing(data);
    
    // ✅ Acknowledge (remove from queue)
    channel.ack(message);
  } catch (error) {
    // ❌ Negative acknowledge
    
    // Requeue (try again)
    channel.nack(message, false, true);  // requeue = true
    
    // Don't requeue (send to DLQ)
    channel.nack(message, false, false);  // requeue = false
    
    // Reject (older method, single message)
    channel.reject(message, false);  // requeue = false
  }
}
```

**Prefetch Count (QoS):**

```typescript
// ===== Quality of Service =====

prefetchCount: 10,  // Process 10 messages at a time

// Without prefetch:
// Consumer gets ALL messages immediately
// - Can overwhelm consumer
// - Uneven load distribution

// With prefetch:
// Consumer gets N messages at a time
// - Controlled processing
// - Even load distribution
// - Better performance

// Example:
{
  transport: Transport.RMQ,
  options: {
    urls: ['amqp://localhost:5672'],
    queue: 'tasks_queue',
    prefetchCount: 5,  // Process max 5 concurrent messages
    noAck: false,      // Must ack before getting more
  },
}

// Flow:
// 1. Consumer gets 5 messages
// 2. Processes them
// 3. Acks completed messages
// 4. Gets more messages (up to 5 total)
```

**Dead Letter Queue:**

```typescript
// ===== Dead Letter Queue (DLQ) =====

// Main queue with DLQ
{
  queue: 'orders_queue',
  queueOptions: {
    durable: true,
    deadLetterExchange: 'dlx',           // DLX name
    deadLetterRoutingKey: 'orders.dead',  // Routing key
    messageTtl: 300000,                  // 5 min TTL
  },
}

// Messages go to DLQ when:
// 1. Rejected with requeue=false
// 2. Message TTL expires
// 3. Queue length limit exceeded
// 4. Consumer explicitly routes to DLQ

// Main queue consumer
@MessagePattern('orders.process')
async processOrder(@Payload() data: any, @Ctx() context: RmqContext) {
  const channel = context.getChannelRef();
  const message = context.getMessage();
  
  try {
    await this.ordersService.process(data);
    channel.ack(message);
  } catch (error) {
    // Send to DLQ (don't requeue)
    channel.nack(message, false, false);
  }
}

// DLQ consumer (separate microservice)
const dlqApp = await NestFactory.createMicroservice(DlqModule, {
  transport: Transport.RMQ,
  options: {
    queue: 'orders_dead_queue',  // DLQ
    queueOptions: { durable: true },
  },
});

@Controller()
export class DlqController {
  @MessagePattern('*')
  async handleDeadLetter(@Payload() data: any) {
    // Log, alert, retry logic, etc.
    console.error('Dead letter:', data);
    await this.logFailedMessage(data);
  }
}
```

**Priority Queues:**

```typescript
// ===== Priority Queue =====

// Enable priority
{
  queue: 'tasks_queue',
  queueOptions: {
    durable: true,
    maxPriority: 10,  // Priority range: 0-10
  },
}

// Send messages with priority
this.client.send('task.process', criticalTask, {
  priority: 10,  // Highest priority
});

this.client.send('task.process', normalTask, {
  priority: 5,   // Medium priority
});

this.client.send('task.process', lowTask, {
  priority: 1,   // Low priority
});

// Processing order:
// 1. Priority 10 messages
// 2. Priority 5 messages
// 3. Priority 1 messages

// Within same priority: FIFO
```

**Lazy Queues:**

```typescript
// ===== Lazy Queue (disk-based) =====

{
  queue: 'large_queue',
  queueOptions: {
    durable: true,
    arguments: {
      'x-queue-mode': 'lazy',  // Store messages on disk
    },
  },
}

// Benefits:
// - Lower memory usage
// - Handle millions of messages
// - Good for large backlogs

// Drawbacks:
// - Slower (disk I/O)
// - Higher latency

// Use case:
// - Queues with millions of messages
// - Slow consumers
// - Memory-constrained systems
```

**Queue Patterns:**

```typescript
// ===== Common Queue Patterns =====

// 1. Work Queue (Task Distribution)
// Multiple workers process tasks from single queue
// Load balanced automatically

Producer → Queue → [Worker 1, Worker 2, Worker 3]

// 2. Pub/Sub (Fan-out)
// Multiple queues receive same message

Producer → Exchange (fanout) → [Queue 1, Queue 2, Queue 3]

// 3. Routing (Topic)
// Messages routed based on routing key

Producer → Exchange (topic) → Queue 1 (*.error)
                           → Queue 2 (user.*)
                           → Queue 3 (*.critical)

// 4. RPC (Request-Reply)
// Temporary reply queue for responses

Client → Request Queue → Server
      ← Reply Queue   ←
```

**Best Practices:**

```typescript
// ✅ GOOD: Use durable queues in production
queueOptions: {
  durable: true,  // Survive restart
}

// ✅ GOOD: Set prefetch count
prefetchCount: 10,  // Control concurrency

// ✅ GOOD: Manual acknowledgment
noAck: false,  // Reliable processing

// ✅ GOOD: Use dead letter queues
queueOptions: {
  deadLetterExchange: 'dlx',
}

// ✅ GOOD: Set message TTL
queueOptions: {
  messageTtl: 300000,  // 5 minutes
}

// ✅ GOOD: Limit queue length
queueOptions: {
  maxLength: 10000,  // Prevent unbounded growth
}

// ❌ BAD: Auto-delete in production
queueOptions: {
  autoDelete: true,  // Can lose messages!
}

// ❌ BAD: No acknowledgment
noAck: true,  // Messages can be lost!

// ❌ BAD: No error handling
@MessagePattern('task')
async process(data: any) {
  // No try-catch, no ack/nack!
  return await this.heavyWork(data);
}

// ❌ BAD: Unbounded queue
queueOptions: {
  // No maxLength, no TTL - can grow forever!
}
```

**Summary:**

```
Queue = Message Buffer

Key Properties:
✅ Name (identifier)
✅ Durable (persist on disk)
✅ Exclusive (single connection)
✅ Auto-delete (cleanup)
✅ Arguments (TTL, DLQ, priority, etc.)

Message Flow:
Producer → Exchange → Queue → Consumer

Acknowledgment:
✅ ack() - Success, remove message
❌ nack() - Failure, requeue or DLQ
❌ reject() - Failure, single message

Features:
✅ Load balancing (multiple consumers)
✅ Dead letter queues (failed messages)
✅ Priority queues (order by priority)
✅ Lazy queues (disk-based, large datasets)
✅ TTL (message expiration)
✅ Max length (queue size limit)
✅ Prefetch (QoS, concurrency control)

Best Practices:
✅ durable: true (production)
✅ noAck: false (reliability)
✅ prefetchCount (control load)
✅ Dead letter queues (handle failures)
✅ Set TTL and maxLength (prevent growth)
```

**Interview Tip**: **Queue in RabbitMQ** is a **message buffer** that stores messages until consumed. **FIFO order** (first-in-first-out). Key properties: **durable** (survive restart), **exclusive** (single connection), **auto-delete** (cleanup when empty). **Manual ack/nack** for reliability. **Prefetch count** controls concurrency. **Dead letter queue** for failed messages. **Priority queues** for ordering. **Lazy queues** for large datasets. **Multiple consumers** enable load balancing. **Best practices**: durable queues, manual ack, prefetch count, DLQ, message TTL, max length. **Queue flow**: Producer → Exchange → Queue → Consumer.

</details>

### 37. What is an exchange in RabbitMQ?

<details>
<summary>Answer</summary>

**An exchange in RabbitMQ** is a **message router** that **receives messages from producers** and **routes them to queues** based on **routing rules**. Exchanges don't store messages; they only route them to appropriate queues based on **exchange type** and **routing keys**.

**Key Concepts:**

```
✅ Message router (not storage)
✅ Receives messages from producers
✅ Routes to queues based on rules
✅ Uses routing keys and bindings
✅ Four main types: direct, fanout, topic, headers
✅ Can be durable or transient
✅ Can bind to other exchanges
```

**Message Flow:**

```typescript
// ===== Message Routing Flow =====

Producer → Exchange → Queue(s) → Consumer(s)

// 1. Producer publishes message to exchange
this.client.send('users.create', data);

// 2. Exchange receives message with routing key
Exchange: 'amq.topic'
Routing Key: 'users.create'

// 3. Exchange routes to bound queues
Binding: 'users.*' → users_queue
Binding: '*.create' → audit_queue

// 4. Consumers receive from queues
@MessagePattern('users.create')
Async createUser(data) { ... }
```

**Exchange Types:**

**1. Direct Exchange:**

```typescript
// ===== Direct Exchange (Exact Match) =====

// Routes messages where routing key EXACTLY matches binding key

// Configuration
{
  transport: Transport.RMQ,
  options: {
    urls: ['amqp://localhost:5672'],
    queue: 'users_queue',
    queueOptions: { durable: true },
    // Direct exchange is default
  },
}

// Binding example:
// Queue 'orders_queue' bound with key 'orders'
// Queue 'users_queue' bound with key 'users'
// Queue 'payments_queue' bound with key 'payments'

// Message with routing key 'orders' → orders_queue only
this.client.send('orders', orderData);

// Message with routing key 'users' → users_queue only
this.client.send('users', userData);

// Use case:
// - Simple routing
// - One-to-one message delivery
// - Direct service communication
// - Task distribution

// Visual:
// Producer → Direct Exchange → Queue 1 (key: 'error')
//                         ↓
//                         → Queue 2 (key: 'info')
//                         ↓
//                         → Queue 3 (key: 'warning')

// Example:
const directExchange = {
  name: 'logs_direct',
  type: 'direct',
  options: { durable: true },
};

// Bindings:
// error_queue ← 'error'
// info_queue ← 'info'
// warning_queue ← 'warning'

// Send messages
channel.publish('logs_direct', 'error', Buffer.from('Error message'));
// → Goes to error_queue only

channel.publish('logs_direct', 'info', Buffer.from('Info message'));
// → Goes to info_queue only
```

**2. Fanout Exchange:**

```typescript
// ===== Fanout Exchange (Broadcast) =====

// Routes messages to ALL bound queues (ignores routing key)

// Configuration
const fanoutExchange = {
  name: 'notifications',
  type: 'fanout',
  options: { durable: true },
};

// Multiple queues bound to fanout exchange
// All queues receive every message

// Example: Broadcasting user events
// Queue 1: email_queue (send emails)
// Queue 2: sms_queue (send SMS)
// Queue 3: push_queue (send push notifications)
// Queue 4: audit_queue (log events)

// All queues receive the message
channel.publish('notifications', '', Buffer.from(JSON.stringify({
  event: 'user.created',
  userId: 123,
  email: 'user@example.com',
})));

// Use case:
// - Broadcasting to multiple services
// - Pub/Sub pattern
// - Event notifications
// - Cache invalidation
// - Real-time updates

// NestJS example:
@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'EVENTS_BUS',
        transport: Transport.RMQ,
        options: {
          urls: ['amqp://localhost:5672'],
          queue: 'events_fanout',
          queueOptions: {
            durable: false,
            exclusive: true,  // Each consumer gets own queue
          },
        },
      },
    ]),
  ],
})

// Emit event (all consumers receive it)
this.eventsClient.emit('user.created', userData);

// Multiple services listen:
// Service 1: Email Service
@EventPattern('user.created')
async sendEmail(data) { ... }

// Service 2: SMS Service
@EventPattern('user.created')
async sendSMS(data) { ... }

// Service 3: Analytics Service
@EventPattern('user.created')
async trackEvent(data) { ... }

// Visual:
// Producer → Fanout Exchange → Queue 1 (Email)
//                         ↓
//                         → Queue 2 (SMS)
//                         ↓
//                         → Queue 3 (Push)
//                         ↓
//                         → Queue 4 (Audit)
```

**3. Topic Exchange:**

```typescript
// ===== Topic Exchange (Pattern Matching) =====

// Routes based on wildcard pattern matching
// * = matches exactly one word
// # = matches zero or more words

// Configuration
const topicExchange = {
  name: 'logs_topic',
  type: 'topic',
  options: { durable: true },
};

// Binding patterns:
// Queue 1: '*.error' → All error logs
// Queue 2: 'user.*' → All user events
// Queue 3: 'order.#' → All order-related events
// Queue 4: '#' → All messages

// Examples:

// 1. Routing key: 'user.error'
// Matches: '*.error' (Queue 1), 'user.*' (Queue 2), '#' (Queue 4)
channel.publish('logs_topic', 'user.error', data);

// 2. Routing key: 'order.created'
// Matches: 'order.#' (Queue 3), '#' (Queue 4)
channel.publish('logs_topic', 'order.created', data);

// 3. Routing key: 'order.payment.failed'
// Matches: 'order.#' (Queue 3), '#' (Queue 4)
channel.publish('logs_topic', 'order.payment.failed', data);

// 4. Routing key: 'system.info'
// Matches: '#' (Queue 4) only
channel.publish('logs_topic', 'system.info', data);

// NestJS with Topic Exchange:
// Producer
this.client.emit('user.created', userData);      // Pattern: user.created
this.client.emit('user.updated', userData);      // Pattern: user.updated
this.client.emit('order.placed', orderData);     // Pattern: order.placed
this.client.emit('order.cancelled', orderData);  // Pattern: order.cancelled

// Consumers with patterns
// Consumer 1: All user events
@EventPattern('user.*')
async handleUserEvent(data) {
  console.log('User event:', data);
}

// Consumer 2: All order events
@EventPattern('order.*')
async handleOrderEvent(data) {
  console.log('Order event:', data);
}

// Consumer 3: All events
@EventPattern('#')
async handleAllEvents(data) {
  console.log('Any event:', data);
}

// Consumer 4: Specific events
@EventPattern('*.created')
async handleCreatedEvents(data) {
  console.log('Something created:', data);
}

// Use case:
// - Flexible routing
// - Log aggregation
// - Event-driven architecture
// - Multi-criteria routing
// - Microservices communication

// Visual:
// Producer → Topic Exchange
//              ↓ (user.*)
//              → Queue 1 (User Service)
//              ↓ (order.*)
//              → Queue 2 (Order Service)
//              ↓ (*.error)
//              → Queue 3 (Error Service)
//              ↓ (#)
//              → Queue 4 (Audit Service)
```

**4. Headers Exchange:**

```typescript
// ===== Headers Exchange (Header Matching) =====

// Routes based on message headers instead of routing key
// Uses 'x-match' argument: 'all' or 'any'

// Configuration
const headersExchange = {
  name: 'tasks_headers',
  type: 'headers',
  options: { durable: true },
};

// Bindings with header criteria:
// Queue 1: { format: 'pdf', type: 'report', 'x-match': 'all' }
//          → Must match ALL headers

// Queue 2: { priority: 'high', 'x-match': 'any' }
//          → Must match ANY header

// Queue 3: { region: 'US', tier: 'premium', 'x-match': 'all' }
//          → Must match ALL headers

// Send message with headers
const message = Buffer.from(JSON.stringify(data));
const options = {
  headers: {
    format: 'pdf',
    type: 'report',
    priority: 'high',
  },
};
channel.publish('tasks_headers', '', message, options);

// This message goes to:
// - Queue 1 (matches format AND type)
// - Queue 2 (matches priority)

// Use case:
// - Complex routing logic
// - Multiple criteria matching
// - Content-based routing
// - Less common than topic exchange

// Example with 'x-match: all':
queue.bind(exchange, '', {
  format: 'pdf',
  type: 'report',
  'x-match': 'all',  // MUST have both headers
});

// Example with 'x-match: any':
queue.bind(exchange, '', {
  priority: 'high',
  urgent: 'true',
  'x-match': 'any',  // Either header matches
});
```

**Default Exchange:**

```typescript
// ===== Default Exchange (Empty String) =====

// Special direct exchange with empty name ''
// Every queue is automatically bound with routing key = queue name

// NestJS uses default exchange by default
{
  transport: Transport.RMQ,
  options: {
    urls: ['amqp://localhost:5672'],
    queue: 'users_queue',  // Routing key = 'users_queue'
  },
}

// Message routing:
// Routing key must match queue name exactly
channel.sendToQueue('users_queue', Buffer.from(data));
// Equivalent to:
channel.publish('', 'users_queue', Buffer.from(data));

// Simple and direct - good for basic use cases
```

**Exchange Properties:**

```typescript
// ===== Exchange Configuration =====

const exchangeConfig = {
  // Exchange name
  name: 'my_exchange',
  
  // Exchange type
  type: 'topic',  // 'direct', 'fanout', 'topic', 'headers'
  
  // Options
  options: {
    // Durability
    durable: true,  // Survive broker restart
    // false: Transient, deleted on restart
    
    // Auto-delete
    autoDelete: false,  // Keep exchange even if no bindings
    // true: Delete when last queue unbinds
    
    // Internal
    internal: false,  // Can receive messages from producers
    // true: Only exchange-to-exchange routing
    
    // Alternate exchange
    alternateExchange: 'unrouted_exchange',  // For unroutable messages
    
    // Arguments
    arguments: {
      // Custom arguments
    },
  },
};

// Create exchange manually (advanced)
import * as amqp from 'amqplib';

const connection = await amqp.connect('amqp://localhost:5672');
const channel = await connection.createChannel();

// Assert exchange
await channel.assertExchange('logs_topic', 'topic', {
  durable: true,
  autoDelete: false,
});
```

**Bindings:**

```typescript
// ===== Exchange-to-Queue Bindings =====

// Binding connects exchange to queue with routing rules

// 1. Bind queue to exchange
await channel.bindQueue(
  'users_queue',      // Queue name
  'events_exchange',  // Exchange name
  'user.*',          // Routing pattern
);

// 2. Multiple bindings per queue
await channel.bindQueue('audit_queue', 'events_exchange', 'user.*');
await channel.bindQueue('audit_queue', 'events_exchange', 'order.*');
await channel.bindQueue('audit_queue', 'events_exchange', '*.error');
// audit_queue receives: all user events, all order events, all errors

// 3. Unbind queue from exchange
await channel.unbindQueue('users_queue', 'events_exchange', 'user.*');

// 4. Exchange-to-exchange binding
await channel.bindExchange(
  'destination_exchange',
  'source_exchange',
  'pattern',
);
```

**Practical Examples:**

```typescript
// ===== Example 1: Logging System (Topic Exchange) =====

// Exchange
const logsExchange = {
  name: 'logs',
  type: 'topic',
  durable: true,
};

// Queues and bindings:
// error_queue ← '*.error'
// warning_queue ← '*.warning'
// info_queue ← '*.info'
// all_logs_queue ← '#'

// Usage:
this.client.emit('user.error', { msg: 'User error' });     // → error_queue, all_logs_queue
this.client.emit('order.warning', { msg: 'Warning' });     // → warning_queue, all_logs_queue
this.client.emit('system.info', { msg: 'Info' });          // → info_queue, all_logs_queue

// ===== Example 2: Multi-Service Notification (Fanout) =====

// Exchange
const notificationsExchange = {
  name: 'notifications',
  type: 'fanout',
  durable: true,
};

// All services receive all notifications:
// email_queue
// sms_queue
// push_queue
// slack_queue

// Usage:
this.client.emit('notification', {
  type: 'user.created',
  userId: 123,
  email: 'user@example.com',
  phone: '+1234567890',
});
// All services receive and process according to their logic

// ===== Example 3: Task Routing (Direct Exchange) =====

// Exchange
const tasksExchange = {
  name: 'tasks',
  type: 'direct',
  durable: true,
};

// Queues:
// image_processing_queue ← 'image.process'
// video_processing_queue ← 'video.process'
// pdf_generation_queue ← 'pdf.generate'

// Usage:
this.client.send('image.process', imageData);  // → image_processing_queue
this.client.send('video.process', videoData);  // → video_processing_queue
this.client.send('pdf.generate', pdfData);     // → pdf_generation_queue
```

**NestJS Configuration:**

```typescript
// ===== NestJS with Custom Exchange =====

// Server (uses default topic exchange)
const app = await NestFactory.createMicroservice<MicroserviceOptions>(
  AppModule,
  {
    transport: Transport.RMQ,
    options: {
      urls: ['amqp://localhost:5672'],
      queue: 'users_queue',
      queueOptions: {
        durable: true,
      },
      // NestJS uses default exchanges
      // For custom exchanges, use amqplib directly
    },
  },
);

// Client
@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'USERS_SERVICE',
        transport: Transport.RMQ,
        options: {
          urls: ['amqp://localhost:5672'],
          queue: 'users_queue',
          queueOptions: { durable: true },
        },
      },
    ]),
  ],
})

// For advanced exchange configuration, use amqplib
import * as amqp from 'amqplib';

@Injectable()
export class RabbitMQService {
  private connection: amqp.Connection;
  private channel: amqp.Channel;
  
  async onModuleInit() {
    this.connection = await amqp.connect('amqp://localhost:5672');
    this.channel = await this.connection.createChannel();
    
    // Create custom exchange
    await this.channel.assertExchange('logs_topic', 'topic', {
      durable: true,
    });
    
    // Create and bind queue
    await this.channel.assertQueue('error_logs', { durable: true });
    await this.channel.bindQueue('error_logs', 'logs_topic', '*.error');
  }
  
  async publishToExchange(exchange: string, routingKey: string, message: any) {
    this.channel.publish(
      exchange,
      routingKey,
      Buffer.from(JSON.stringify(message)),
      { persistent: true },
    );
  }
}
```

**Best Practices:**

```typescript
// ✅ GOOD: Use durable exchanges in production
exchangeOptions: {
  durable: true,  // Survive restart
}

// ✅ GOOD: Choose appropriate exchange type
// Direct: Simple routing, one-to-one
// Fanout: Broadcasting, pub/sub
// Topic: Flexible routing, patterns
// Headers: Complex criteria (rare)

// ✅ GOOD: Use topic exchange for flexibility
// Most versatile, supports patterns
type: 'topic'

// ✅ GOOD: Use meaningful routing keys
// Good: 'user.created', 'order.payment.failed'
// Bad: 'evt1', 'msg', 'data'

// ✅ GOOD: Use alternate exchange for unroutable messages
alternateExchange: 'unrouted_messages'

// ❌ BAD: Auto-delete exchanges in production
exchangeOptions: {
  autoDelete: true,  // Can lose routing!
}

// ❌ BAD: Overly complex routing
// Too many patterns, hard to maintain

// ❌ BAD: No naming convention
// Inconsistent routing keys
```

**Comparison Table:**

| Exchange Type | Routing | Use Case | Example |
|---------------|---------|----------|----------|
| **Direct** | Exact match | Simple routing, task queues | `'error'` → error_queue |
| **Fanout** | All queues | Broadcasting, pub/sub | All queues get message |
| **Topic** | Pattern match | Flexible routing, events | `'user.*'` matches `'user.created'` |
| **Headers** | Header match | Complex criteria | `{priority: 'high'}` |
| **Default** | Queue name | Simple direct routing | Queue name = routing key |

**Summary:**

```
Exchange = Message Router

Flow:
Producer → Exchange → Binding → Queue → Consumer

Exchange Types:
1. Direct: Exact routing key match
2. Fanout: Broadcast to all queues
3. Topic: Pattern matching (*, #)
4. Headers: Header-based routing
5. Default: Queue name routing

Key Properties:
✅ Name (identifier)
✅ Type (routing logic)
✅ Durable (persist on restart)
✅ Auto-delete (cleanup)
✅ Bindings (connect to queues)

Best Practices:
✅ Use durable exchanges (production)
✅ Choose appropriate type
✅ Topic exchange (most flexible)
✅ Meaningful routing keys
✅ Alternate exchange (unroutable)
✅ Consistent naming convention

Common Patterns:
✅ Direct: Task distribution
✅ Fanout: Event broadcasting
✅ Topic: Log routing, events
✅ Default: Simple messaging
```

**Interview Tip**: **Exchange in RabbitMQ** is a **message router** that receives messages from producers and routes to queues. **Four types**: **1) Direct** - exact routing key match, **2) Fanout** - broadcast to all queues (pub/sub), **3) Topic** - pattern matching with wildcards (* one word, # zero/more words), **4) Headers** - header-based routing. **Default exchange** routes by queue name. **Bindings** connect exchanges to queues with routing rules. **Exchanges don't store messages**, only route them. **Durable exchanges** survive restart. **Topic exchange** most flexible (supports patterns). **Flow**: Producer → Exchange → Queue → Consumer. **Best for**: decoupled routing, flexible message distribution, pub/sub patterns.

</details>

## Service Discovery

### ### 38. What is service discovery and why is it needed?

<details>
<summary>Answer</summary>

**Service discovery** is a mechanism that allows microservices to **automatically find and communicate with each other** without hardcoding network locations (IP addresses and ports). It provides **dynamic registration** and **lookup** of services in a distributed system.

**Why Service Discovery is Needed:**

**1. Dynamic Environments:**

```typescript
// ❌ WITHOUT Service Discovery (Hardcoded)
const USERS_SERVICE_URL = 'http://192.168.1.100:3001';  // Fixed IP
const ORDERS_SERVICE_URL = 'http://192.168.1.101:3002';

// Problems:
// - IP changes require code changes
// - Can't scale horizontally (only one instance)
// - Manual configuration needed
// - No failover if service goes down
// - Deployment nightmare

// ✅ WITH Service Discovery (Dynamic)
const usersService = await discovery.getService('users-service');
// Returns: { host: '10.0.1.5', port: 3001 } (current instance)

// Benefits:
// - Automatic service lookup
// - Multiple instances (load balancing)
// - Auto-updates when instances change
// - Failover to healthy instances
// - Easy scaling
```

**2. Elastic Scaling:**

```typescript
// ===== Horizontal Scaling =====

// Initially: 1 instance
users-service: 192.168.1.10:3001

// Scale up to 3 instances
users-service:
  - 192.168.1.10:3001
  - 192.168.1.11:3001
  - 192.168.1.12:3001

// Service discovery automatically:
// ✅ Registers new instances
// ✅ Load balances between instances
// ✅ Removes unhealthy instances
// ✅ No code changes needed!

// Without service discovery:
// ❌ Manual configuration updates
// ❌ Code changes and redeployment
// ❌ No automatic failover
```

**3. Cloud/Container Environments:**

```typescript
// ===== Container Orchestration (Kubernetes, Docker Swarm) =====

// Containers have dynamic IPs
// Container 1: 172.17.0.3:3000 (restarts → 172.17.0.8:3000)
// Container 2: 172.17.0.5:3000 (scales down → removed)
// Container 3: 172.17.0.9:3000 (new instance)

// Service discovery tracks changes automatically:
service-registry:
  users-service:
    - instance-1: 172.17.0.3:3000  ✅ healthy
    - instance-2: 172.17.0.8:3000  ✅ healthy (updated)
    - instance-3: 172.17.0.9:3000  ✅ healthy (new)

// Without service discovery:
// ❌ Lost connections when IPs change
// ❌ Can't track container lifecycle
// ❌ Manual updates required
```

**Core Concepts:**

**1. Service Registration:**

```typescript
// ===== Service Registration =====

// Each service registers itself on startup
import { Injectable, OnModuleInit } from '@nestjs/common';
import * as Consul from 'consul';

@Injectable()
export class ServiceRegistrationService implements OnModuleInit {
  private consul: Consul;
  
  constructor() {
    this.consul = new Consul({
      host: 'consul-server',
      port: 8500,
    });
  }
  
  async onModuleInit() {
    // Register service
    await this.consul.agent.service.register({
      name: 'users-service',           // Service name
      id: 'users-service-1',           // Unique instance ID
      address: process.env.HOST || 'localhost',
      port: parseInt(process.env.PORT) || 3001,
      tags: ['api', 'users', 'v1'],    // Tags for filtering
      
      // Health check
      check: {
        http: 'http://localhost:3001/health',
        interval: '10s',               // Check every 10 seconds
        timeout: '5s',
        deregistercriticalserviceafter: '1m',
      },
    });
    
    console.log('Service registered with Consul');
  }
  
  async onModuleDestroy() {
    // Deregister on shutdown
    await this.consul.agent.service.deregister('users-service-1');
  }
}
```

**2. Service Discovery (Lookup):**

```typescript
// ===== Service Lookup =====

@Injectable()
export class ServiceDiscoveryService {
  private consul: Consul;
  
  constructor() {
    this.consul = new Consul({ host: 'consul-server', port: 8500 });
  }
  
  // Get healthy service instances
  async getService(serviceName: string) {
    const result = await this.consul.health.service({
      service: serviceName,
      passing: true,  // Only healthy instances
    });
    
    if (result.length === 0) {
      throw new Error(`No healthy instances of ${serviceName}`);
    }
    
    // Load balance (round-robin, random, etc.)
    const instance = this.selectInstance(result);
    
    return {
      host: instance.Service.Address,
      port: instance.Service.Port,
    };
  }
  
  // Simple random selection
  private selectInstance(instances: any[]) {
    return instances[Math.floor(Math.random() * instances.length)];
  }
}

// Usage in HTTP client
@Injectable()
export class OrdersService {
  constructor(
    private discovery: ServiceDiscoveryService,
    private httpService: HttpService,
  ) {}
  
  async createOrder(orderData: any) {
    // Discover users service
    const usersService = await this.discovery.getService('users-service');
    
    // Make request to discovered instance
    const user = await firstValueFrom(
      this.httpService.get(
        `http://${usersService.host}:${usersService.port}/users/${orderData.userId}`,
      ),
    );
    
    return this.ordersRepository.create({
      ...orderData,
      userName: user.name,
    });
  }
}
```

**3. Health Checks:**

```typescript
// ===== Health Check Endpoint =====

import { Controller, Get } from '@nestjs/common';
import { HealthCheck, HealthCheckService, HttpHealthIndicator, TypeOrmHealthIndicator } from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private http: HttpHealthIndicator,
    private db: TypeOrmHealthIndicator,
  ) {}
  
  @Get()
  @HealthCheck()
  check() {
    return this.health.check([
      // Check database
      () => this.db.pingCheck('database'),
      
      // Check external service
      () => this.http.pingCheck('users-service', 'http://users-service/health'),
    ]);
  }
}

// Response:
// ✅ Healthy:
{
  "status": "ok",
  "info": {
    "database": { "status": "up" },
    "users-service": { "status": "up" }
  }
}

// ❌ Unhealthy:
{
  "status": "error",
  "error": {
    "database": { "status": "down" }
  }
}
```

**Service Discovery Patterns:**

**1. Client-Side Discovery:**

```typescript
// ===== Client-Side Discovery =====

// Client queries registry and selects instance

// Flow:
// 1. Service registers with registry
// 2. Client queries registry for service instances
// 3. Client selects instance (load balancing)
// 4. Client makes request directly to service

@Injectable()
export class ClientSideDiscovery {
  async callService() {
    // 1. Query service registry
    const instances = await this.consul.health.service({
      service: 'users-service',
      passing: true,
    });
    
    // 2. Client-side load balancing
    const instance = this.loadBalance(instances);
    
    // 3. Direct call to service
    const response = await axios.get(
      `http://${instance.address}:${instance.port}/users`,
    );
    
    return response.data;
  }
  
  private loadBalance(instances: any[]) {
    // Round-robin, random, least connections, etc.
    return instances[Math.floor(Math.random() * instances.length)];
  }
}

// Pros:
// ✅ No single point of failure
// ✅ Client controls load balancing
// ✅ Direct communication (fast)

// Cons:
// ❌ Each client needs discovery logic
// ❌ More complex clients
// ❌ Language-specific libraries
```

**2. Server-Side Discovery (Load Balancer):**

```typescript
// ===== Server-Side Discovery =====

// Load balancer queries registry and routes requests

// Flow:
// 1. Service registers with registry
// 2. Client calls load balancer (fixed address)
// 3. Load balancer queries registry
// 4. Load balancer selects instance and forwards request

// Client (simple - just calls load balancer)
@Injectable()
export class ServerSideDiscovery {
  async callService() {
    // Call load balancer (fixed address)
    const response = await axios.get(
      'http://load-balancer/users-service/users',
    );
    
    return response.data;
  }
}

// Load balancer handles discovery and routing
// Examples: NGINX, HAProxy, AWS ELB, Kubernetes Service

// Pros:
// ✅ Simple clients
// ✅ Centralized load balancing
// ✅ Language agnostic

// Cons:
// ❌ Load balancer is single point of failure (need HA)
// ❌ Additional network hop (latency)
// ❌ Load balancer becomes bottleneck
```

**Service Discovery Tools:**

**1. Consul:**

```typescript
// ===== Consul Integration =====

import * as Consul from 'consul';

// Service registration
const consul = new Consul({
  host: 'consul.example.com',
  port: 8500,
  secure: true,
});

await consul.agent.service.register({
  name: 'users-service',
  id: `users-service-${process.pid}`,
  address: 'localhost',
  port: 3001,
  tags: ['api', 'v1'],
  check: {
    http: 'http://localhost:3001/health',
    interval: '10s',
  },
});

// Service discovery
const services = await consul.health.service({
  service: 'users-service',
  passing: true,
});

// Features:
// ✅ Service registry
// ✅ Health checking
// ✅ Key-value store
// ✅ Multi-datacenter
// ✅ DNS interface
```

**2. Eureka (Netflix):**

```typescript
// ===== Eureka Integration =====

import { Eureka } from 'eureka-js-client';

const client = new Eureka({
  instance: {
    app: 'USERS-SERVICE',
    instanceId: 'users-service-1',
    hostName: 'localhost',
    ipAddr: '127.0.0.1',
    port: {
      '$': 3001,
      '@enabled': true,
    },
    vipAddress: 'users-service',
    statusPageUrl: 'http://localhost:3001/info',
    healthCheckUrl: 'http://localhost:3001/health',
  },
  eureka: {
    host: 'eureka.example.com',
    port: 8761,
    servicePath: '/eureka/apps/',
  },
});

client.start();

// Features:
// ✅ Netflix OSS ecosystem
// ✅ Self-preservation mode
// ✅ Region/zone awareness
// ✅ REST API
```

**3. Kubernetes DNS:**

```typescript
// ===== Kubernetes Service Discovery =====

// Built-in service discovery via DNS

// Service definition
apiVersion: v1
kind: Service
metadata:
  name: users-service
spec:
  selector:
    app: users
  ports:
    - port: 3001
      targetPort: 3001

// Automatic DNS:
// users-service.default.svc.cluster.local

// Usage in code:
const response = await axios.get(
  'http://users-service:3001/users',  // DNS name
);

// Features:
// ✅ Built-in (no external tool)
// ✅ Automatic DNS
// ✅ Load balancing
// ✅ Health checks (readiness/liveness)
// ✅ Service mesh integration
```

**4. etcd:**

```typescript
// ===== etcd Integration =====

import { Etcd3 } from 'etcd3';

const client = new Etcd3({
  hosts: 'http://etcd.example.com:2379',
});

// Register service
await client.put(
  '/services/users-service/instance-1',
  JSON.stringify({
    host: 'localhost',
    port: 3001,
    version: '1.0.0',
  }),
);

// Service discovery
const services = await client.getAll()
  .prefix('/services/users-service/')
  .strings();

// Features:
// ✅ Distributed key-value store
// ✅ Strong consistency
// ✅ Watch mechanism
// ✅ Used by Kubernetes
```

**Best Practices:**

```typescript
// ✅ GOOD: Implement health checks
@Get('health')
health() {
  return { status: 'ok' };
}

// ✅ GOOD: Graceful shutdown
async onModuleDestroy() {
  await this.deregisterService();
}

// ✅ GOOD: Retry logic
async getService(serviceName: string, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      return await this.discovery.getService(serviceName);
    } catch (error) {
      if (i === retries - 1) throw error;
      await sleep(1000 * Math.pow(2, i));  // Exponential backoff
    }
  }
}

// ✅ GOOD: Cache service locations
private serviceCache = new Map<string, { instances: any[]; expires: number }>();

// ✅ GOOD: Use tags for versioning
tags: ['v1', 'prod', 'us-east-1']

// ✅ GOOD: Monitor service health
check: {
  http: 'http://localhost:3001/health',
  interval: '10s',
  timeout: '5s',
}

// ❌ BAD: Hardcoded service addresses
const url = 'http://192.168.1.10:3001';  // Never do this!

// ❌ BAD: No health checks
// Service stays registered even when unhealthy

// ❌ BAD: No deregistration on shutdown
// Zombie services in registry

// ❌ BAD: No retry logic
// Fails on first error
```

**Summary:**

```
Service Discovery = Dynamic Service Location

Why Needed:
✅ Dynamic IP addresses (cloud/containers)
✅ Elastic scaling (add/remove instances)
✅ Automatic failover
✅ Load balancing
✅ Zero configuration
✅ Simplified deployment

Core Components:
1. Service Registry (Consul, Eureka, etcd)
2. Service Registration (on startup)
3. Service Discovery (lookup)
4. Health Checks (monitor status)
5. Load Balancing (select instance)

Patterns:
1. Client-Side: Client queries and selects
2. Server-Side: Load balancer handles it

Tools:
✅ Consul (HashiCorp)
✅ Eureka (Netflix)
✅ Kubernetes DNS (built-in)
✅ etcd (CoreOS)
✅ Zookeeper (Apache)

Best Practices:
✅ Health checks (liveness/readiness)
✅ Graceful shutdown (deregister)
✅ Retry logic (resilience)
✅ Cache instances (performance)
✅ Use tags (versioning/filtering)
✅ Monitor registry health

Benefits:
✅ No hardcoded IPs
✅ Auto-scaling support
✅ Automatic failover
✅ Easy deployment
✅ Cloud-native ready
```

**Interview Tip**: **Service discovery** enables microservices to **automatically find each other** without hardcoded IPs. **Why needed**: **1) Dynamic environments** (IPs change), **2) Elastic scaling** (add/remove instances), **3) Cloud/containers** (ephemeral IPs), **4) Automatic failover**, **5) Load balancing**. **Two patterns**: **Client-side** (client queries registry), **Server-side** (load balancer queries). **Components**: **registry** (stores locations), **registration** (services register on startup), **discovery** (lookup instances), **health checks** (monitor status). **Tools**: Consul, Eureka, Kubernetes DNS, etcd. **Best practices**: health checks, graceful shutdown, retry logic, caching. **Without it**: hardcoded IPs, manual updates, no scaling, no failover. **Essential for** microservices, cloud-native apps, containers.

</details>

### 39. How do you implement service discovery in microservices?

<details>
<summary>Answer</summary>

**Implementing service discovery** involves setting up a **service registry**, configuring **service registration** (services register themselves), implementing **service lookup** (finding services), and adding **health checks** (monitoring). Multiple approaches exist depending on your infrastructure.

**Implementation Approaches:**

**1. Using Consul (HashiCorp):**

```typescript
// ===== Step 1: Install Dependencies =====

// npm install consul @nestjs/axios

// ===== Step 2: Service Registration Module =====

import { Injectable, OnModuleInit, OnModuleDestroy } from '@nestjs/common';
import * as Consul from 'consul';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class ConsulService implements OnModuleInit, OnModuleDestroy {
  private consul: Consul.Consul;
  private serviceId: string;
  
  constructor(private configService: ConfigService) {
    this.consul = new Consul({
      host: this.configService.get('CONSUL_HOST', 'localhost'),
      port: this.configService.get('CONSUL_PORT', 8500),
      promisify: true,
    });
    
    this.serviceId = `${this.configService.get('SERVICE_NAME')}-${process.pid}`;
  }
  
  // Register service on startup
  async onModuleInit() {
    const serviceName = this.configService.get('SERVICE_NAME');
    const serviceHost = this.configService.get('SERVICE_HOST', 'localhost');
    const servicePort = this.configService.get('SERVICE_PORT', 3000);
    
    try {
      await this.consul.agent.service.register({
        id: this.serviceId,
        name: serviceName,
        address: serviceHost,
        port: parseInt(servicePort),
        tags: [
          `version:${this.configService.get('SERVICE_VERSION', '1.0.0')}`,
          'api',
          this.configService.get('NODE_ENV', 'development'),
        ],
        check: {
          http: `http://${serviceHost}:${servicePort}/health`,
          interval: '10s',
          timeout: '5s',
          deregistercriticalserviceafter: '1m',
        },
        meta: {
          version: this.configService.get('SERVICE_VERSION', '1.0.0'),
          environment: this.configService.get('NODE_ENV'),
        },
      });
      
      console.log(`Service registered: ${this.serviceId}`);
    } catch (error) {
      console.error('Failed to register service:', error);
      throw error;
    }
  }
  
  // Deregister service on shutdown
  async onModuleDestroy() {
    try {
      await this.consul.agent.service.deregister(this.serviceId);
      console.log(`Service deregistered: ${this.serviceId}`);
    } catch (error) {
      console.error('Failed to deregister service:', error);
    }
  }
  
  // Get healthy service instances
  async getService(serviceName: string): Promise<ServiceInstance> {
    try {
      const result = await this.consul.health.service({
        service: serviceName,
        passing: true,  // Only healthy instances
      });
      
      if (!result || result.length === 0) {
        throw new Error(`No healthy instances found for service: ${serviceName}`);
      }
      
      // Load balance using random selection
      const instance = this.selectInstance(result);
      
      return {
        id: instance.Service.ID,
        name: instance.Service.Service,
        address: instance.Service.Address,
        port: instance.Service.Port,
        tags: instance.Service.Tags,
        meta: instance.Service.Meta,
      };
    } catch (error) {
      console.error(`Failed to discover service ${serviceName}:`, error);
      throw error;
    }
  }
  
  // Get all instances (including unhealthy)
  async getAllInstances(serviceName: string): Promise<ServiceInstance[]> {
    const result = await this.consul.health.service({
      service: serviceName,
    });
    
    return result.map(instance => ({
      id: instance.Service.ID,
      name: instance.Service.Service,
      address: instance.Service.Address,
      port: instance.Service.Port,
      tags: instance.Service.Tags,
      meta: instance.Service.Meta,
      healthy: instance.Checks.every(check => check.Status === 'passing'),
    }));
  }
  
  // Load balancing strategies
  private selectInstance(instances: any[]): any {
    // Random selection
    return instances[Math.floor(Math.random() * instances.length)];
    
    // For round-robin, use a counter
    // For weighted, use instance metadata
    // For least connections, track active connections
  }
  
  // Watch for service changes
  watchService(serviceName: string, callback: (instances: any[]) => void) {
    const watcher = this.consul.watch({
      method: this.consul.health.service,
      options: {
        service: serviceName,
        passing: true,
      },
    });
    
    watcher.on('change', callback);
    watcher.on('error', (error) => {
      console.error(`Watch error for ${serviceName}:`, error);
    });
    
    return watcher;
  }
}

interface ServiceInstance {
  id: string;
  name: string;
  address: string;
  port: number;
  tags: string[];
  meta?: Record<string, string>;
  healthy?: boolean;
}

// ===== Step 3: Health Check Controller =====

import { Controller, Get } from '@nestjs/common';
import { HealthCheck, HealthCheckService, TypeOrmHealthIndicator } from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private db: TypeOrmHealthIndicator,
  ) {}
  
  @Get()
  @HealthCheck()
  check() {
    return this.health.check([
      () => this.db.pingCheck('database'),
    ]);
  }
}

// ===== Step 4: Service Discovery Client =====

import { Injectable } from '@nestjs/common';
import { HttpService } from '@nestjs/axios';
import { firstValueFrom } from 'rxjs';

@Injectable()
export class DiscoveryHttpClient {
  // Cache for service instances
  private serviceCache = new Map<string, {
    instances: ServiceInstance[];
    lastUpdated: number;
  }>();
  
  private readonly CACHE_TTL = 30000; // 30 seconds
  
  constructor(
    private consulService: ConsulService,
    private httpService: HttpService,
  ) {}
  
  // Make HTTP request with service discovery
  async get<T>(serviceName: string, path: string): Promise<T> {
    const instance = await this.getServiceInstance(serviceName);
    const url = `http://${instance.address}:${instance.port}${path}`;
    
    try {
      const response = await firstValueFrom(
        this.httpService.get<T>(url),
      );
      return response.data;
    } catch (error) {
      console.error(`Request failed to ${serviceName}:`, error);
      // Implement retry logic or circuit breaker here
      throw error;
    }
  }
  
  async post<T>(serviceName: string, path: string, data: any): Promise<T> {
    const instance = await this.getServiceInstance(serviceName);
    const url = `http://${instance.address}:${instance.port}${path}`;
    
    const response = await firstValueFrom(
      this.httpService.post<T>(url, data),
    );
    return response.data;
  }
  
  // Get service instance with caching
  private async getServiceInstance(serviceName: string): Promise<ServiceInstance> {
    const cached = this.serviceCache.get(serviceName);
    const now = Date.now();
    
    // Use cache if valid
    if (cached && (now - cached.lastUpdated) < this.CACHE_TTL) {
      return this.selectRandomInstance(cached.instances);
    }
    
    // Fetch fresh instances
    const instance = await this.consulService.getService(serviceName);
    
    // Update cache
    this.serviceCache.set(serviceName, {
      instances: [instance],
      lastUpdated: now,
    });
    
    return instance;
  }
  
  private selectRandomInstance(instances: ServiceInstance[]): ServiceInstance {
    return instances[Math.floor(Math.random() * instances.length)];
  }
  
  // Clear cache for a service
  clearCache(serviceName?: string) {
    if (serviceName) {
      this.serviceCache.delete(serviceName);
    } else {
      this.serviceCache.clear();
    }
  }
}

// ===== Step 5: Usage in Service =====

@Injectable()
export class OrdersService {
  constructor(private discoveryClient: DiscoveryHttpClient) {}
  
  async createOrder(orderData: CreateOrderDto) {
    // Discover and call users service
    const user = await this.discoveryClient.get<User>(
      'users-service',
      `/users/${orderData.userId}`,
    );
    
    // Discover and call inventory service
    const inventory = await this.discoveryClient.post<InventoryResponse>(
      'inventory-service',
      '/inventory/reserve',
      { productId: orderData.productId, quantity: orderData.quantity },
    );
    
    // Create order
    return this.ordersRepository.create({
      ...orderData,
      userName: user.name,
      inventoryReservationId: inventory.reservationId,
    });
  }
}

// ===== Step 6: Module Setup =====

import { Module } from '@nestjs/common';
import { HttpModule } from '@nestjs/axios';
import { TerminusModule } from '@nestjs/terminus';

@Module({
  imports: [
    HttpModule,
    TerminusModule,
  ],
  controllers: [HealthController],
  providers: [ConsulService, DiscoveryHttpClient, OrdersService],
  exports: [ConsulService, DiscoveryHttpClient],
})
export class ServiceDiscoveryModule {}

// ===== Step 7: Environment Configuration =====

// .env
SERVICE_NAME=orders-service
SERVICE_HOST=localhost
SERVICE_PORT=3002
SERVICE_VERSION=1.0.0
CONSUL_HOST=localhost
CONSUL_PORT=8500
NODE_ENV=development
```

**2. Using Kubernetes DNS:**

```typescript
// ===== Kubernetes Native Service Discovery =====

// Step 1: Define Kubernetes Service
// users-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: users-service
  namespace: default
spec:
  selector:
    app: users
  ports:
    - name: http
      port: 3001
      targetPort: 3001
  type: ClusterIP

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: users-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: users
  template:
    metadata:
      labels:
        app: users
    spec:
      containers:
      - name: users-service
        image: users-service:1.0.0
        ports:
        - containerPort: 3001
        env:
        - name: SERVICE_NAME
          value: "users-service"
        livenessProbe:
          httpGet:
            path: /health
            port: 3001
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 3001
          initialDelaySeconds: 5
          periodSeconds: 5

// Step 2: Use DNS names in code
@Injectable()
export class OrdersService {
  constructor(private httpService: HttpService) {}
  
  async createOrder(orderData: CreateOrderDto) {
    // Kubernetes automatically resolves DNS
    // Format: <service-name>.<namespace>.svc.cluster.local
    // Short form: <service-name> (same namespace)
    
    const user = await firstValueFrom(
      this.httpService.get(
        'http://users-service:3001/users/' + orderData.userId,
      ),
    );
    
    // Cross-namespace
    const payment = await firstValueFrom(
      this.httpService.post(
        'http://payment-service.payments.svc.cluster.local:3003/payments',
        paymentData,
      ),
    );
    
    return user.data;
  }
}

// Step 3: Environment-based configuration
@Injectable()
export class K8sDiscoveryService {
  constructor(private configService: ConfigService) {}
  
  getServiceUrl(serviceName: string): string {
    const namespace = this.configService.get('K8S_NAMESPACE', 'default');
    const port = this.configService.get(`${serviceName.toUpperCase()}_PORT`, 3000);
    
    // Use short name in same namespace
    if (namespace === 'default') {
      return `http://${serviceName}:${port}`;
    }
    
    // Use FQDN for cross-namespace
    return `http://${serviceName}.${namespace}.svc.cluster.local:${port}`;
  }
}

// Benefits:
// ✅ Built-in (no external dependencies)
// ✅ Automatic load balancing
// ✅ Health checks (readiness/liveness probes)
// ✅ Zero configuration
```

**3. Using Eureka (Netflix OSS):**

```typescript
// ===== Eureka Service Discovery =====

// npm install eureka-js-client

import { Injectable, OnModuleInit, OnModuleDestroy } from '@nestjs/common';
import { Eureka } from 'eureka-js-client';

@Injectable()
export class EurekaService implements OnModuleInit, OnModuleDestroy {
  private client: Eureka;
  
  constructor(private configService: ConfigService) {
    this.client = new Eureka({
      instance: {
        app: this.configService.get('SERVICE_NAME').toUpperCase(),
        instanceId: `${this.configService.get('SERVICE_NAME')}-${process.pid}`,
        hostName: this.configService.get('SERVICE_HOST', 'localhost'),
        ipAddr: this.configService.get('SERVICE_IP', '127.0.0.1'),
        statusPageUrl: `http://${this.configService.get('SERVICE_HOST')}:${this.configService.get('SERVICE_PORT')}/info`,
        healthCheckUrl: `http://${this.configService.get('SERVICE_HOST')}:${this.configService.get('SERVICE_PORT')}/health`,
        port: {
          '$': parseInt(this.configService.get('SERVICE_PORT', '3000')),
          '@enabled': true,
        },
        vipAddress: this.configService.get('SERVICE_NAME'),
        dataCenterInfo: {
          '@class': 'com.netflix.appinfo.InstanceInfo$DefaultDataCenterInfo',
          name: 'MyOwn',
        },
      },
      eureka: {
        host: this.configService.get('EUREKA_HOST', 'localhost'),
        port: parseInt(this.configService.get('EUREKA_PORT', '8761')),
        servicePath: '/eureka/apps/',
        maxRetries: 3,
        requestRetryDelay: 500,
      },
    });
  }
  
  onModuleInit() {
    this.client.start((error) => {
      if (error) {
        console.error('Failed to register with Eureka:', error);
      } else {
        console.log('Successfully registered with Eureka');
      }
    });
  }
  
  onModuleDestroy() {
    this.client.stop();
  }
  
  getService(serviceName: string) {
    const instances = this.client.getInstancesByAppId(serviceName.toUpperCase());
    
    if (!instances || instances.length === 0) {
      throw new Error(`No instances found for ${serviceName}`);
    }
    
    // Random selection
    const instance = instances[Math.floor(Math.random() * instances.length)];
    
    return {
      host: instance.hostName,
      port: instance.port['$'],
      url: `http://${instance.hostName}:${instance.port['$']}`,
    };
  }
}
```

**4. Using etcd:**

```typescript
// ===== etcd Service Discovery =====

// npm install etcd3

import { Injectable, OnModuleInit, OnModuleDestroy } from '@nestjs/common';
import { Etcd3 } from 'etcd3';

@Injectable()
export class EtcdService implements OnModuleInit, OnModuleDestroy {
  private client: Etcd3;
  private serviceKey: string;
  private lease: any;
  
  constructor(private configService: ConfigService) {
    this.client = new Etcd3({
      hosts: this.configService.get('ETCD_HOSTS', 'http://localhost:2379'),
    });
    
    const serviceName = this.configService.get('SERVICE_NAME');
    const instanceId = `${serviceName}-${process.pid}`;
    this.serviceKey = `/services/${serviceName}/${instanceId}`;
  }
  
  async onModuleInit() {
    // Create lease (TTL)
    this.lease = this.client.lease(10); // 10 seconds TTL
    
    // Register service with lease
    await this.client.put(this.serviceKey).value(JSON.stringify({
      host: this.configService.get('SERVICE_HOST'),
      port: this.configService.get('SERVICE_PORT'),
      version: this.configService.get('SERVICE_VERSION'),
      timestamp: Date.now(),
    })).lease(this.lease);
    
    // Keep-alive (heartbeat)
    await this.lease.keepalive();
    
    console.log('Service registered with etcd');
  }
  
  async onModuleDestroy() {
    await this.client.delete().key(this.serviceKey);
    await this.lease.revoke();
  }
  
  async getService(serviceName: string) {
    const prefix = `/services/${serviceName}/`;
    const services = await this.client.getAll().prefix(prefix).strings();
    
    const instances = Object.entries(services).map(([key, value]) => {
      return JSON.parse(value);
    });
    
    if (instances.length === 0) {
      throw new Error(`No instances found for ${serviceName}`);
    }
    
    return instances[Math.floor(Math.random() * instances.length)];
  }
  
  // Watch for changes
  watchService(serviceName: string, callback: (instances: any[]) => void) {
    const prefix = `/services/${serviceName}/`;
    
    const watcher = this.client.watch().prefix(prefix).create();
    
    watcher.on('put', async (kv) => {
      const instances = await this.getAllInstances(serviceName);
      callback(instances);
    });
    
    watcher.on('delete', async (kv) => {
      const instances = await this.getAllInstances(serviceName);
      callback(instances);
    });
    
    return watcher;
  }
  
  private async getAllInstances(serviceName: string) {
    const prefix = `/services/${serviceName}/`;
    const services = await this.client.getAll().prefix(prefix).strings();
    return Object.values(services).map(v => JSON.parse(v));
  }
}
```

**Advanced Patterns:**

**1. Load Balancing Strategies:**

```typescript
// ===== Load Balancing =====

@Injectable()
export class LoadBalancer {
  private roundRobinCounters = new Map<string, number>();
  private connectionCounts = new Map<string, number>();
  
  // 1. Round-robin
  roundRobin(serviceName: string, instances: ServiceInstance[]): ServiceInstance {
    const counter = this.roundRobinCounters.get(serviceName) || 0;
    const instance = instances[counter % instances.length];
    this.roundRobinCounters.set(serviceName, counter + 1);
    return instance;
  }
  
  // 2. Random
  random(instances: ServiceInstance[]): ServiceInstance {
    return instances[Math.floor(Math.random() * instances.length)];
  }
  
  // 3. Weighted (based on tags/metadata)
  weighted(instances: ServiceInstance[]): ServiceInstance {
    const weights = instances.map(i => parseInt(i.meta?.weight || '1'));
    const totalWeight = weights.reduce((sum, w) => sum + w, 0);
    let random = Math.random() * totalWeight;
    
    for (let i = 0; i < instances.length; i++) {
      random -= weights[i];
      if (random <= 0) return instances[i];
    }
    
    return instances[0];
  }
  
  // 4. Least connections
  leastConnections(instances: ServiceInstance[]): ServiceInstance {
    return instances.reduce((min, instance) => {
      const minCount = this.connectionCounts.get(min.id) || 0;
      const instanceCount = this.connectionCounts.get(instance.id) || 0;
      return instanceCount < minCount ? instance : min;
    });
  }
  
  // Track connections
  incrementConnections(instanceId: string) {
    const count = this.connectionCounts.get(instanceId) || 0;
    this.connectionCounts.set(instanceId, count + 1);
  }
  
  decrementConnections(instanceId: string) {
    const count = this.connectionCounts.get(instanceId) || 0;
    this.connectionCounts.set(instanceId, Math.max(0, count - 1));
  }
}
```

**2. Circuit Breaker Pattern:**

```typescript
// ===== Circuit Breaker with Service Discovery =====

import { Injectable } from '@nestjs/common';

enum CircuitState {
  CLOSED = 'CLOSED',     // Normal operation
  OPEN = 'OPEN',         // Failing, reject requests
  HALF_OPEN = 'HALF_OPEN' // Testing if service recovered
}

@Injectable()
export class CircuitBreakerService {
  private circuits = new Map<string, {
    state: CircuitState;
    failureCount: number;
    lastFailureTime: number;
    successCount: number;
  }>();
  
  private readonly FAILURE_THRESHOLD = 5;
  private readonly TIMEOUT = 60000; // 1 minute
  private readonly SUCCESS_THRESHOLD = 3;
  
  async callWithCircuitBreaker(
    serviceName: string,
    operation: () => Promise<any>,
  ): Promise<any> {
    const circuit = this.getCircuit(serviceName);
    
    // Circuit is OPEN, reject immediately
    if (circuit.state === CircuitState.OPEN) {
      const timeSinceLastFailure = Date.now() - circuit.lastFailureTime;
      
      if (timeSinceLastFailure > this.TIMEOUT) {
        // Try again (HALF_OPEN)
        circuit.state = CircuitState.HALF_OPEN;
        circuit.successCount = 0;
      } else {
        throw new Error(`Circuit breaker OPEN for ${serviceName}`);
      }
    }
    
    try {
      const result = await operation();
      this.onSuccess(serviceName);
      return result;
    } catch (error) {
      this.onFailure(serviceName);
      throw error;
    }
  }
  
  private getCircuit(serviceName: string) {
    if (!this.circuits.has(serviceName)) {
      this.circuits.set(serviceName, {
        state: CircuitState.CLOSED,
        failureCount: 0,
        lastFailureTime: 0,
        successCount: 0,
      });
    }
    return this.circuits.get(serviceName)!;
  }
  
  private onSuccess(serviceName: string) {
    const circuit = this.getCircuit(serviceName);
    circuit.failureCount = 0;
    
    if (circuit.state === CircuitState.HALF_OPEN) {
      circuit.successCount++;
      
      if (circuit.successCount >= this.SUCCESS_THRESHOLD) {
        circuit.state = CircuitState.CLOSED;
        console.log(`Circuit breaker CLOSED for ${serviceName}`);
      }
    }
  }
  
  private onFailure(serviceName: string) {
    const circuit = this.getCircuit(serviceName);
    circuit.failureCount++;
    circuit.lastFailureTime = Date.now();
    
    if (circuit.failureCount >= this.FAILURE_THRESHOLD) {
      circuit.state = CircuitState.OPEN;
      console.log(`Circuit breaker OPEN for ${serviceName}`);
    }
  }
}

// Usage
@Injectable()
export class ResilientOrdersService {
  constructor(
    private discoveryClient: DiscoveryHttpClient,
    private circuitBreaker: CircuitBreakerService,
  ) {}
  
  async createOrder(orderData: CreateOrderDto) {
    const user = await this.circuitBreaker.callWithCircuitBreaker(
      'users-service',
      () => this.discoveryClient.get('users-service', `/users/${orderData.userId}`),
    );
    
    return this.ordersRepository.create({ ...orderData, userName: user.name });
  }
}
```

**Best Practices:**

```typescript
// ✅ GOOD: Implement health checks
@Get('health')
@HealthCheck()
check() {
  return this.health.check([
    () => this.db.pingCheck('database'),
    () => this.http.pingCheck('external-api', 'http://api.example.com'),
  ]);
}

// ✅ GOOD: Graceful shutdown
async onModuleDestroy() {
  await this.deregisterService();
  await this.closeConnections();
}

// ✅ GOOD: Retry with exponential backoff
async getServiceWithRetry(serviceName: string, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await this.consulService.getService(serviceName);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await this.sleep(Math.pow(2, i) * 1000);
    }
  }
}

// ✅ GOOD: Cache service instances
private serviceCache = new Map<string, {
  instances: ServiceInstance[];
  expiresAt: number;
}>();

// ✅ GOOD: Use tags for filtering
tags: ['version:1.0.0', 'region:us-east-1', 'env:prod']

// ✅ GOOD: Monitor discovery health
setInterval(() => {
  this.checkRegistryHealth();
}, 30000);

// ❌ BAD: No health checks
// Service stays registered even when unhealthy

// ❌ BAD: No deregistration
// Zombie services in registry

// ❌ BAD: No caching
// Query registry on every request (performance hit)

// ❌ BAD: Hardcoded timeouts
// Use configuration instead
```

**Summary:**

```
Implementation Steps:

1. Choose Discovery Tool:
   • Consul (feature-rich, popular)
   • Kubernetes DNS (k8s native)
   • Eureka (Netflix stack)
   • etcd (distributed key-value)

2. Service Registration:
   • Register on startup (onModuleInit)
   • Deregister on shutdown (onModuleDestroy)
   • Include metadata (version, tags)
   • Set up health check endpoint

3. Service Discovery:
   • Query registry for healthy instances
   • Implement load balancing
   • Cache results (with TTL)
   • Handle errors gracefully

4. Health Monitoring:
   • Create /health endpoint
   • Check dependencies (DB, APIs)
   • Regular health check intervals
   • Auto-deregister if unhealthy

5. Advanced Patterns:
   • Load balancing strategies
   • Circuit breaker pattern
   • Retry with backoff
   • Service mesh (Istio, Linkerd)

Best Practices:
✅ Health checks (critical)
✅ Graceful shutdown (deregister)
✅ Cache with TTL (performance)
✅ Retry logic (resilience)
✅ Circuit breaker (fault tolerance)
✅ Load balancing (distribution)
✅ Monitoring/alerting (observability)
✅ Tags/metadata (filtering)

Benefits:
✅ No hardcoded IPs
✅ Auto-scaling support
✅ Automatic failover
✅ Load distribution
✅ Cloud-native ready
```

**Interview Tip**: **Implement service discovery** by: **1) Choose tool** (Consul, Kubernetes DNS, Eureka, etcd), **2) Service registration** - register on startup with metadata, health check URL, deregister on shutdown, **3) Service discovery** - query registry for healthy instances, implement load balancing (round-robin/random), cache results, **4) Health checks** - create /health endpoint, check dependencies, auto-deregister if unhealthy. **Key components**: registry (stores services), registration (onModuleInit), discovery (lookup), health monitoring. **Best practices**: health checks, graceful shutdown, caching with TTL, retry logic, circuit breaker. **Kubernetes**: use built-in DNS (service-name.namespace.svc.cluster.local), zero config. **Consul**: feature-rich with watches, KV store, multi-DC. **Essential for** production microservices.

</details>

## Error Handling

### 40. How do you handle errors in microservices?

<details>
<summary>Answer</summary>

**Error handling in microservices** is more complex than monoliths due to **distributed nature**, **network failures**, **multiple failure points**, and **cascading failures**. You need to handle errors at multiple levels: **transport layer**, **application layer**, **circuit breakers**, **retries**, and **fallbacks**.

**Error Handling Layers:**

**1. Application-Level Errors (RpcException):**

```typescript
// ===== RpcException for Microservice Errors =====

import { Injectable } from '@nestjs/common';
import { RpcException } from '@nestjs/microservices';

@Injectable()
export class UsersService {
  async getUserById(id: string) {
    const user = await this.usersRepository.findOne({ where: { id } });
    
    if (!user) {
      // Throw RpcException (not HttpException)
      throw new RpcException({
        statusCode: 404,
        message: 'User not found',
        error: 'NOT_FOUND',
      });
    }
    
    return user;
  }
  
  async createUser(userData: CreateUserDto) {
    try {
      // Validate
      if (!userData.email) {
        throw new RpcException({
          statusCode: 400,
          message: 'Email is required',
          error: 'VALIDATION_ERROR',
        });
      }
      
      // Check duplicate
      const existing = await this.usersRepository.findOne({
        where: { email: userData.email },
      });
      
      if (existing) {
        throw new RpcException({
          statusCode: 409,
          message: 'Email already exists',
          error: 'DUPLICATE_EMAIL',
        });
      }
      
      return await this.usersRepository.create(userData);
    } catch (error) {
      // Database error
      if (error.code === '23505') {  // Postgres unique violation
        throw new RpcException({
          statusCode: 409,
          message: 'Email already exists',
          error: 'DUPLICATE_EMAIL',
        });
      }
      
      // Re-throw RpcException
      if (error instanceof RpcException) {
        throw error;
      }
      
      // Unknown error
      console.error('Unexpected error:', error);
      throw new RpcException({
        statusCode: 500,
        message: 'Internal server error',
        error: 'INTERNAL_ERROR',
      });
    }
  }
}

// ===== Message Pattern Handler =====

import { Controller } from '@nestjs/common';
import { MessagePattern } from '@nestjs/microservices';

@Controller()
export class UsersController {
  constructor(private usersService: UsersService) {}
  
  @MessagePattern({ cmd: 'get_user' })
  async getUser(id: string) {
    try {
      return await this.usersService.getUserById(id);
    } catch (error) {
      // Error is automatically serialized and sent back to client
      throw error;
    }
  }
  
  @MessagePattern({ cmd: 'create_user' })
  async createUser(userData: CreateUserDto) {
    return await this.usersService.createUser(userData);
  }
}
```

**2. Client-Side Error Handling:**

```typescript
// ===== Handling Errors in Client =====

import { Injectable } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { catchError, timeout, retry } from 'rxjs/operators';
import { throwError, of, firstValueFrom } from 'rxjs';

@Injectable()
export class OrdersService {
  constructor(
    @Inject('USERS_SERVICE') private usersClient: ClientProxy,
  ) {}
  
  async createOrder(orderData: CreateOrderDto) {
    try {
      // Method 1: Using firstValueFrom with try-catch
      const user = await firstValueFrom(
        this.usersClient.send({ cmd: 'get_user' }, orderData.userId).pipe(
          timeout(5000),  // 5 second timeout
          catchError((error) => {
            console.error('Users service error:', error);
            
            // Check error type
            if (error.message?.includes('timeout')) {
              throw new Error('Users service timeout');
            }
            
            // RpcException from microservice
            if (error.statusCode === 404) {
              throw new Error('User not found');
            }
            
            // Default error
            throw new Error('Failed to get user');
          }),
        ),
      );
      
      return this.ordersRepository.create({
        ...orderData,
        userName: user.name,
      });
    } catch (error) {
      // Handle error at controller level
      throw error;
    }
  }
  
  // Method 2: With retry logic
  async getUserWithRetry(userId: string) {
    return await firstValueFrom(
      this.usersClient.send({ cmd: 'get_user' }, userId).pipe(
        timeout(5000),
        retry({
          count: 3,  // Retry 3 times
          delay: 1000,  // Wait 1 second between retries
        }),
        catchError((error) => {
          console.error('All retries failed:', error);
          throw error;
        }),
      ),
    );
  }
  
  // Method 3: With fallback
  async getUserWithFallback(userId: string) {
    return await firstValueFrom(
      this.usersClient.send({ cmd: 'get_user' }, userId).pipe(
        timeout(5000),
        catchError((error) => {
          console.error('Users service error, using fallback:', error);
          
          // Return fallback/default value
          return of({
            id: userId,
            name: 'Unknown User',
            email: 'unknown@example.com',
          });
        }),
      ),
    );
  }
}
```

**3. Global Exception Filter:**

```typescript
// ===== Global RPC Exception Filter =====

import { Catch, RpcExceptionFilter, ArgumentsHost } from '@nestjs/common';
import { RpcException } from '@nestjs/microservices';
import { Observable, throwError } from 'rxjs';

@Catch(RpcException)
export class GlobalRpcExceptionFilter implements RpcExceptionFilter<RpcException> {
  catch(exception: RpcException, host: ArgumentsHost): Observable<any> {
    const error = exception.getError();
    
    // Log error
    console.error('RPC Exception:', error);
    
    // Send structured error response
    return throwError(() => ({
      statusCode: error['statusCode'] || 500,
      message: error['message'] || 'Internal server error',
      error: error['error'] || 'INTERNAL_ERROR',
      timestamp: new Date().toISOString(),
    }));
  }
}

// ===== Catch All Exceptions =====

@Catch()
export class AllExceptionsFilter implements RpcExceptionFilter {
  catch(exception: any, host: ArgumentsHost): Observable<any> {
    console.error('Unhandled exception:', exception);
    
    // RpcException
    if (exception instanceof RpcException) {
      const error = exception.getError();
      return throwError(() => error);
    }
    
    // Database errors
    if (exception.code) {
      return throwError(() => ({
        statusCode: 500,
        message: 'Database error',
        error: 'DATABASE_ERROR',
        code: exception.code,
      }));
    }
    
    // Unknown errors
    return throwError(() => ({
      statusCode: 500,
      message: exception.message || 'Internal server error',
      error: 'INTERNAL_ERROR',
    }));
  }
}

// ===== Apply Filter =====

// main.ts (microservice)
import { NestFactory } from '@nestjs/core';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.TCP,
      options: { port: 3001 },
    },
  );
  
  // Apply global exception filter
  app.useGlobalFilters(new AllExceptionsFilter());
  
  await app.listen();
}
bootstrap();
```

**4. Circuit Breaker Pattern:**

```typescript
// ===== Circuit Breaker for Fault Tolerance =====

import { Injectable } from '@nestjs/common';

enum CircuitState {
  CLOSED = 'CLOSED',       // Normal operation
  OPEN = 'OPEN',           // Failing, reject requests immediately
  HALF_OPEN = 'HALF_OPEN', // Testing recovery
}

interface CircuitConfig {
  failureThreshold: number;    // Open circuit after N failures
  successThreshold: number;    // Close circuit after N successes
  timeout: number;             // Time before trying HALF_OPEN (ms)
}

@Injectable()
export class CircuitBreakerService {
  private circuits = new Map<string, {
    state: CircuitState;
    failureCount: number;
    successCount: number;
    lastFailureTime: number;
    nextAttemptTime: number;
  }>();
  
  private defaultConfig: CircuitConfig = {
    failureThreshold: 5,
    successThreshold: 2,
    timeout: 60000, // 1 minute
  };
  
  async execute<T>(
    serviceName: string,
    operation: () => Promise<T>,
    config: Partial<CircuitConfig> = {},
  ): Promise<T> {
    const circuit = this.getCircuit(serviceName);
    const conf = { ...this.defaultConfig, ...config };
    
    // Circuit is OPEN
    if (circuit.state === CircuitState.OPEN) {
      const now = Date.now();
      
      if (now >= circuit.nextAttemptTime) {
        // Try HALF_OPEN
        circuit.state = CircuitState.HALF_OPEN;
        circuit.successCount = 0;
        console.log(`Circuit HALF_OPEN for ${serviceName}`);
      } else {
        // Reject immediately (fail fast)
        throw new RpcException({
          statusCode: 503,
          message: `Service ${serviceName} is currently unavailable`,
          error: 'CIRCUIT_OPEN',
        });
      }
    }
    
    try {
      const result = await operation();
      this.onSuccess(serviceName, conf);
      return result;
    } catch (error) {
      this.onFailure(serviceName, conf);
      throw error;
    }
  }
  
  private getCircuit(serviceName: string) {
    if (!this.circuits.has(serviceName)) {
      this.circuits.set(serviceName, {
        state: CircuitState.CLOSED,
        failureCount: 0,
        successCount: 0,
        lastFailureTime: 0,
        nextAttemptTime: 0,
      });
    }
    return this.circuits.get(serviceName)!;
  }
  
  private onSuccess(serviceName: string, config: CircuitConfig) {
    const circuit = this.getCircuit(serviceName);
    circuit.failureCount = 0;
    
    if (circuit.state === CircuitState.HALF_OPEN) {
      circuit.successCount++;
      
      if (circuit.successCount >= config.successThreshold) {
        circuit.state = CircuitState.CLOSED;
        console.log(`Circuit CLOSED for ${serviceName}`);
      }
    }
  }
  
  private onFailure(serviceName: string, config: CircuitConfig) {
    const circuit = this.getCircuit(serviceName);
    circuit.failureCount++;
    circuit.lastFailureTime = Date.now();
    
    if (circuit.failureCount >= config.failureThreshold) {
      circuit.state = CircuitState.OPEN;
      circuit.nextAttemptTime = Date.now() + config.timeout;
      console.log(`Circuit OPEN for ${serviceName}`);
    }
  }
  
  // Get circuit status
  getStatus(serviceName: string) {
    const circuit = this.getCircuit(serviceName);
    return {
      state: circuit.state,
      failureCount: circuit.failureCount,
      successCount: circuit.successCount,
    };
  }
  
  // Reset circuit manually
  reset(serviceName: string) {
    this.circuits.delete(serviceName);
  }
}

// ===== Usage with Circuit Breaker =====

@Injectable()
export class ResilientOrdersService {
  constructor(
    @Inject('USERS_SERVICE') private usersClient: ClientProxy,
    private circuitBreaker: CircuitBreakerService,
  ) {}
  
  async createOrder(orderData: CreateOrderDto) {
    // Wrap microservice call with circuit breaker
    const user = await this.circuitBreaker.execute(
      'users-service',
      async () => {
        return await firstValueFrom(
          this.usersClient.send({ cmd: 'get_user' }, orderData.userId).pipe(
            timeout(5000),
          ),
        );
      },
      {
        failureThreshold: 3,
        successThreshold: 2,
        timeout: 30000,
      },
    );
    
    return this.ordersRepository.create({
      ...orderData,
      userName: user.name,
    });
  }
}
```

**5. Retry with Exponential Backoff:**

```typescript
// ===== Retry with Exponential Backoff =====

import { Injectable } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { firstValueFrom } from 'rxjs';
import { timeout } from 'rxjs/operators';

@Injectable()
export class RetryService {
  async retryWithBackoff<T>(
    operation: () => Promise<T>,
    maxRetries: number = 3,
    baseDelay: number = 1000,
  ): Promise<T> {
    let lastError: any;
    
    for (let attempt = 0; attempt <= maxRetries; attempt++) {
      try {
        return await operation();
      } catch (error) {
        lastError = error;
        
        if (attempt < maxRetries) {
          // Calculate delay with exponential backoff
          const delay = baseDelay * Math.pow(2, attempt);
          
          // Add jitter (randomness) to prevent thundering herd
          const jitter = Math.random() * 1000;
          const totalDelay = delay + jitter;
          
          console.log(`Retry attempt ${attempt + 1}/${maxRetries} after ${totalDelay}ms`);
          
          await this.sleep(totalDelay);
        }
      }
    }
    
    throw new RpcException({
      statusCode: 503,
      message: `Operation failed after ${maxRetries} retries`,
      error: 'MAX_RETRIES_EXCEEDED',
      originalError: lastError,
    });
  }
  
  private sleep(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

// ===== Usage =====

@Injectable()
export class OrdersService {
  constructor(
    @Inject('USERS_SERVICE') private usersClient: ClientProxy,
    private retryService: RetryService,
  ) {}
  
  async createOrder(orderData: CreateOrderDto) {
    const user = await this.retryService.retryWithBackoff(
      async () => {
        return await firstValueFrom(
          this.usersClient.send({ cmd: 'get_user' }, orderData.userId).pipe(
            timeout(5000),
          ),
        );
      },
      3,     // Max 3 retries
      1000,  // Start with 1 second delay
    );
    
    return user;
  }
}
```

**6. Dead Letter Queue (DLQ):**

```typescript
// ===== Dead Letter Queue for Failed Messages =====

import { Injectable } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';

@Injectable()
export class DeadLetterQueueService {
  constructor(
    @Inject('DLQ_SERVICE') private dlqClient: ClientProxy,
  ) {}
  
  async sendToDeadLetterQueue(data: {
    originalMessage: any;
    error: any;
    serviceName: string;
    timestamp: Date;
    retryCount: number;
  }) {
    try {
      await firstValueFrom(
        this.dlqClient.emit('message.failed', {
          ...data,
          timestamp: new Date(),
        }),
      );
      
      console.log('Message sent to DLQ:', data);
    } catch (error) {
      console.error('Failed to send to DLQ:', error);
      // Last resort: log to file/database
    }
  }
}

// ===== DLQ Handler =====

@Controller()
export class DlqController {
  @EventPattern('message.failed')
  async handleFailedMessage(data: any) {
    console.log('Processing DLQ message:', data);
    
    // Store in database for manual inspection
    await this.dlqRepository.create(data);
    
    // Send alert
    await this.alertService.notify({
      type: 'DLQ_MESSAGE',
      service: data.serviceName,
      error: data.error,
    });
  }
}
```

**7. Timeout Handling:**

```typescript
// ===== Timeout Handling =====

import { timeout, catchError } from 'rxjs/operators';
import { TimeoutError } from 'rxjs';

@Injectable()
export class OrdersService {
  async createOrder(orderData: CreateOrderDto) {
    try {
      const user = await firstValueFrom(
        this.usersClient.send({ cmd: 'get_user' }, orderData.userId).pipe(
          timeout(5000),  // 5 second timeout
          catchError((error) => {
            if (error instanceof TimeoutError) {
              throw new RpcException({
                statusCode: 504,
                message: 'Users service timeout',
                error: 'GATEWAY_TIMEOUT',
              });
            }
            throw error;
          }),
        ),
      );
      
      return user;
    } catch (error) {
      throw error;
    }
  }
}
```

**Best Practices:**

```typescript
// ✅ GOOD: Use RpcException for microservice errors
throw new RpcException({
  statusCode: 404,
  message: 'User not found',
  error: 'NOT_FOUND',
});

// ✅ GOOD: Implement circuit breaker
await this.circuitBreaker.execute('users-service', operation);

// ✅ GOOD: Retry with exponential backoff
await this.retryService.retryWithBackoff(operation, 3, 1000);

// ✅ GOOD: Set timeouts
this.client.send('cmd', data).pipe(timeout(5000))

// ✅ GOOD: Use fallbacks
catchError(() => of(defaultValue))

// ✅ GOOD: Log errors for debugging
console.error('Service error:', { error, context });

// ✅ GOOD: Use dead letter queue
await this.dlqService.sendToDeadLetterQueue(failedMessage);

// ✅ GOOD: Structured error responses
{
  statusCode: 400,
  message: 'Validation failed',
  error: 'VALIDATION_ERROR',
  details: ['Email is required'],
}

// ❌ BAD: Using HttpException in microservices
throw new HttpException('Error', 404);  // Won't work!

// ❌ BAD: No timeout
this.client.send('cmd', data)  // Can hang forever

// ❌ BAD: No retry logic
// Fails on first error

// ❌ BAD: Not catching errors
// Crashes the service

// ❌ BAD: Generic error messages
throw new Error('Error');  // Not helpful
```

**Summary:**

```
Error Handling Strategies:

1. Application Level:
   • Use RpcException (not HttpException)
   • Structured error responses
   • Validate input data
   • Handle business logic errors

2. Transport Level:
   • Connection errors
   • Timeout errors
   • Serialization errors
   • Network failures

3. Resilience Patterns:
   • Circuit Breaker (prevent cascading failures)
   • Retry with exponential backoff
   • Timeout handling
   • Fallback values
   • Bulkhead pattern

4. Error Propagation:
   • Catch and re-throw with context
   • Log errors for debugging
   • Send to monitoring/alerting
   • Dead letter queue for failed messages

5. Global Filters:
   • RpcExceptionFilter
   • AllExceptionsFilter
   • Log all errors
   • Transform errors

Best Practices:
✅ Use RpcException
✅ Implement circuit breaker
✅ Retry with backoff
✅ Set timeouts (5-30s)
✅ Use fallbacks
✅ Dead letter queue
✅ Structured errors
✅ Log for debugging
✅ Monitor error rates
✅ Alert on failures

Common Errors:
• Network timeout
• Service unavailable
• Connection refused
• Serialization failed
• Validation error
• Not found
• Circuit open

Tools:
• Circuit breaker library
• RxJS operators (retry, timeout, catchError)
• Monitoring (Prometheus, Grafana)
• Logging (Winston, Pino)
• Tracing (Jaeger, Zipkin)
```

**Interview Tip**: **Handle microservice errors** with: **1) RpcException** - use for app errors (not HttpException), structured response with statusCode/message/error, **2) Circuit breaker** - prevent cascading failures, open after N failures, half-open to test recovery, **3) Retry with exponential backoff** - automatic retries with increasing delays, add jitter, **4) Timeout handling** - set timeouts (5-30s) to prevent hanging, **5) Fallback values** - return default when service unavailable, **6) Global filters** - catch all errors, transform and log, **7) Dead letter queue** - store failed messages for manual processing. **RxJS operators**: timeout(), retry(), catchError(). **Resilience patterns**: circuit breaker, retry, timeout, fallback, bulkhead. **Best practices**: structured errors, monitoring, alerting, logging with context. **Different from monolith**: distributed failures, network errors, cascading failures, partial failures.

</details>

### 41. What is `RpcException`?

<details>
<summary>Answer</summary>

**`RpcException`** is a specialized exception class in NestJS for **microservices communication**. It's designed to throw errors from microservices that get **serialized and transmitted** to the client over the transport layer. Unlike `HttpException` (which is for HTTP/REST APIs), `RpcException` works with all microservice transports (TCP, Redis, RabbitMQ, Kafka, gRPC, etc.).

**Key Characteristics:**

```typescript
// ===== RpcException Overview =====

import { RpcException } from '@nestjs/microservices';

// Key Points:
// ✅ Designed for microservices (not HTTP)
// ✅ Works with all transports (TCP, Redis, RabbitMQ, etc.)
// ✅ Serializable across network
// ✅ Caught by RpcExceptionFilter
// ✅ Can carry any serializable data
// ✅ Automatically propagated to client

// Location:
// @nestjs/microservices package

// Purpose:
// Send structured error responses from microservices to clients
```

**Basic Usage:**

```typescript
// ===== Throwing RpcException =====

import { Injectable } from '@nestjs/common';
import { RpcException } from '@nestjs/microservices';

@Injectable()
export class UsersService {
  async getUserById(id: string) {
    const user = await this.usersRepository.findOne({ where: { id } });
    
    if (!user) {
      // Throw RpcException with error data
      throw new RpcException({
        statusCode: 404,
        message: 'User not found',
        error: 'NOT_FOUND',
      });
    }
    
    return user;
  }
  
  async createUser(userData: CreateUserDto) {
    // Validation error
    if (!userData.email || !userData.name) {
      throw new RpcException({
        statusCode: 400,
        message: 'Email and name are required',
        error: 'VALIDATION_ERROR',
        details: {
          email: !userData.email ? 'Email is required' : null,
          name: !userData.name ? 'Name is required' : null,
        },
      });
    }
    
    // Duplicate check
    const existing = await this.usersRepository.findOne({
      where: { email: userData.email },
    });
    
    if (existing) {
      throw new RpcException({
        statusCode: 409,
        message: 'Email already exists',
        error: 'DUPLICATE_EMAIL',
        email: userData.email,
      });
    }
    
    return await this.usersRepository.create(userData);
  }
}
```

**RpcException Constructor:**

```typescript
// ===== RpcException Constructor Signatures =====

// 1. String error
throw new RpcException('User not found');
// Error received by client: 'User not found'

// 2. Object error (recommended)
throw new RpcException({
  statusCode: 404,
  message: 'User not found',
  error: 'NOT_FOUND',
});
// Error received by client: { statusCode: 404, message: '...', error: '...' }

// 3. Any serializable data
throw new RpcException({
  code: 'USER_NOT_FOUND',
  userId: '123',
  timestamp: new Date().toISOString(),
  details: {
    reason: 'User was deleted',
    deletedAt: '2024-01-01',
  },
});

// 4. Error object
throw new RpcException(new Error('Something went wrong'));
// Error received: 'Something went wrong'

// The argument passed to RpcException is what gets sent to the client
```

**Catching RpcException on Client:**

```typescript
// ===== Client-Side Error Handling =====

import { Injectable, Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { catchError, firstValueFrom } from 'rxjs';

@Injectable()
export class OrdersService {
  constructor(
    @Inject('USERS_SERVICE') private usersClient: ClientProxy,
  ) {}
  
  async createOrder(orderData: CreateOrderDto) {
    try {
      // Call microservice
      const user = await firstValueFrom(
        this.usersClient.send({ cmd: 'get_user' }, orderData.userId),
      );
      
      return this.ordersRepository.create({
        ...orderData,
        userName: user.name,
      });
    } catch (error) {
      // Error from RpcException is in error object
      console.error('Error from users service:', error);
      
      // Check error properties
      if (error.statusCode === 404) {
        throw new HttpException('User not found', HttpStatus.NOT_FOUND);
      }
      
      if (error.error === 'VALIDATION_ERROR') {
        throw new HttpException(
          error.message || 'Validation failed',
          HttpStatus.BAD_REQUEST,
        );
      }
      
      // Generic error
      throw new HttpException(
        'Service temporarily unavailable',
        HttpStatus.SERVICE_UNAVAILABLE,
      );
    }
  }
  
  // Using RxJS catchError operator
  async getUserWithCatchError(userId: string) {
    return await firstValueFrom(
      this.usersClient.send({ cmd: 'get_user' }, userId).pipe(
        catchError((error) => {
          console.error('RpcException caught:', error);
          
          // error is the data passed to RpcException
          if (error.statusCode === 404) {
            // Return fallback value
            return of({
              id: userId,
              name: 'Unknown User',
              email: 'unknown@example.com',
            });
          }
          
          // Re-throw or transform error
          throw new HttpException(
            error.message || 'Service error',
            error.statusCode || 500,
          );
        }),
      ),
    );
  }
}
```

**RpcException vs HttpException:**

```typescript
// ===== Comparison =====

// ❌ WRONG: Using HttpException in microservice
import { HttpException, HttpStatus } from '@nestjs/common';

@Controller()
export class UsersController {
  @MessagePattern({ cmd: 'get_user' })
  async getUser(id: string) {
    const user = await this.usersService.findOne(id);
    
    if (!user) {
      // DON'T use HttpException in microservices!
      throw new HttpException('User not found', HttpStatus.NOT_FOUND);
      // This won't serialize properly over microservice transports
    }
    
    return user;
  }
}

// ✅ CORRECT: Using RpcException in microservice
import { RpcException } from '@nestjs/microservices';

@Controller()
export class UsersController {
  @MessagePattern({ cmd: 'get_user' })
  async getUser(id: string) {
    const user = await this.usersService.findOne(id);
    
    if (!user) {
      // Use RpcException for microservices
      throw new RpcException({
        statusCode: 404,
        message: 'User not found',
        error: 'NOT_FOUND',
      });
    }
    
    return user;
  }
}

// Comparison Table:
// | Feature         | HttpException       | RpcException           |
// |-----------------|---------------------|------------------------|
// | Use case        | HTTP/REST APIs      | Microservices          |
// | Transport       | HTTP only           | All transports         |
// | Serialization   | HTTP response       | Transport-specific     |
// | Status codes    | HTTP status codes   | Custom status codes    |
// | Package         | @nestjs/common      | @nestjs/microservices  |
```

**RpcExceptionFilter:**

```typescript
// ===== Custom RpcExceptionFilter =====

import { Catch, RpcExceptionFilter, ArgumentsHost } from '@nestjs/common';
import { RpcException } from '@nestjs/microservices';
import { Observable, throwError } from 'rxjs';

@Catch(RpcException)
export class CustomRpcExceptionFilter implements RpcExceptionFilter<RpcException> {
  catch(exception: RpcException, host: ArgumentsHost): Observable<any> {
    const error = exception.getError();
    
    // Log error
    console.error('RpcException caught by filter:', error);
    
    // Transform error
    const transformedError = {
      statusCode: typeof error === 'object' && error['statusCode'] 
        ? error['statusCode'] 
        : 500,
      message: typeof error === 'object' && error['message']
        ? error['message']
        : typeof error === 'string'
        ? error
        : 'Internal server error',
      error: typeof error === 'object' && error['error']
        ? error['error']
        : 'INTERNAL_ERROR',
      timestamp: new Date().toISOString(),
      // Add request ID for tracing
      requestId: this.generateRequestId(),
    };
    
    // Return observable with error
    return throwError(() => transformedError);
  }
  
  private generateRequestId(): string {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }
}

// Apply filter globally
import { NestFactory } from '@nestjs/core';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.TCP,
      options: { port: 3001 },
    },
  );
  
  // Apply global RPC exception filter
  app.useGlobalFilters(new CustomRpcExceptionFilter());
  
  await app.listen();
}
bootstrap();
```

**Advanced Usage:**

```typescript
// ===== Advanced RpcException Patterns =====

// 1. Custom error class extending RpcException
export class UserNotFoundException extends RpcException {
  constructor(userId: string) {
    super({
      statusCode: 404,
      message: `User with ID ${userId} not found`,
      error: 'USER_NOT_FOUND',
      userId,
      timestamp: new Date().toISOString(),
    });
  }
}

export class ValidationException extends RpcException {
  constructor(errors: Record<string, string>) {
    super({
      statusCode: 400,
      message: 'Validation failed',
      error: 'VALIDATION_ERROR',
      details: errors,
    });
  }
}

// Usage
if (!user) {
  throw new UserNotFoundException(userId);
}

if (validationErrors) {
  throw new ValidationException({
    email: 'Invalid email format',
    age: 'Must be at least 18',
  });
}

// 2. Error factory
export class RpcErrorFactory {
  static notFound(resource: string, id: string): RpcException {
    return new RpcException({
      statusCode: 404,
      message: `${resource} not found`,
      error: 'NOT_FOUND',
      resource,
      id,
    });
  }
  
  static validation(field: string, message: string): RpcException {
    return new RpcException({
      statusCode: 400,
      message: `Validation error: ${message}`,
      error: 'VALIDATION_ERROR',
      field,
    });
  }
  
  static unauthorized(message: string = 'Unauthorized'): RpcException {
    return new RpcException({
      statusCode: 401,
      message,
      error: 'UNAUTHORIZED',
    });
  }
  
  static forbidden(message: string = 'Forbidden'): RpcException {
    return new RpcException({
      statusCode: 403,
      message,
      error: 'FORBIDDEN',
    });
  }
  
  static conflict(message: string): RpcException {
    return new RpcException({
      statusCode: 409,
      message,
      error: 'CONFLICT',
    });
  }
  
  static internal(message: string = 'Internal server error'): RpcException {
    return new RpcException({
      statusCode: 500,
      message,
      error: 'INTERNAL_ERROR',
    });
  }
}

// Usage
throw RpcErrorFactory.notFound('User', userId);
throw RpcErrorFactory.validation('email', 'Invalid email format');
throw RpcErrorFactory.unauthorized('Token expired');

// 3. Wrapping database errors
try {
  return await this.usersRepository.create(userData);
} catch (error) {
  // PostgreSQL unique violation
  if (error.code === '23505') {
    throw new RpcException({
      statusCode: 409,
      message: 'Email already exists',
      error: 'DUPLICATE_EMAIL',
      field: 'email',
    });
  }
  
  // Foreign key violation
  if (error.code === '23503') {
    throw new RpcException({
      statusCode: 400,
      message: 'Referenced entity does not exist',
      error: 'FOREIGN_KEY_VIOLATION',
    });
  }
  
  // Generic database error
  console.error('Database error:', error);
  throw new RpcException({
    statusCode: 500,
    message: 'Database operation failed',
    error: 'DATABASE_ERROR',
  });
}

// 4. Chaining errors (preserving context)
try {
  const result = await this.externalService.call();
  return result;
} catch (error) {
  throw new RpcException({
    statusCode: 503,
    message: 'External service unavailable',
    error: 'EXTERNAL_SERVICE_ERROR',
    originalError: error.message,
    service: 'payment-service',
  });
}
```

**Error Serialization:**

```typescript
// ===== How RpcException is Serialized =====

// Server side (microservice)
@MessagePattern({ cmd: 'get_user' })
async getUser(id: string) {
  throw new RpcException({
    statusCode: 404,
    message: 'User not found',
    error: 'NOT_FOUND',
    userId: id,
  });
}

// What gets sent over transport:
// {
//   statusCode: 404,
//   message: 'User not found',
//   error: 'NOT_FOUND',
//   userId: '123',
// }

// Client side (receives error)
const user = await firstValueFrom(
  this.client.send({ cmd: 'get_user' }, '123')
).catch(error => {
  console.log(error);
  // Output:
  // {
  //   statusCode: 404,
  //   message: 'User not found',
  //   error: 'NOT_FOUND',
  //   userId: '123',
  // }
});

// Note: The error object passed to RpcException is what the client receives
```

**Best Practices:**

```typescript
// ✅ GOOD: Use RpcException in microservices
throw new RpcException({ statusCode: 404, message: 'Not found' });

// ✅ GOOD: Structured error objects
throw new RpcException({
  statusCode: 400,
  message: 'Validation failed',
  error: 'VALIDATION_ERROR',
  details: validationErrors,
});

// ✅ GOOD: Include error codes for programmatic handling
error: 'USER_NOT_FOUND'  // Client can switch on this

// ✅ GOOD: Add context (IDs, timestamps, etc.)
throw new RpcException({
  statusCode: 404,
  message: 'User not found',
  userId,
  timestamp: new Date().toISOString(),
});

// ✅ GOOD: Use custom exception classes
throw new UserNotFoundException(userId);

// ✅ GOOD: Log errors before throwing
console.error('User not found:', { userId, context });
throw new RpcException(...);

// ✅ GOOD: Catch and transform errors
try {
  await operation();
} catch (error) {
  if (error instanceof RpcException) throw error;
  throw new RpcException({ ...transform(error) });
}

// ❌ BAD: Using HttpException in microservices
throw new HttpException('Error', 404);  // Won't work!

// ❌ BAD: Throwing plain Error objects
throw new Error('User not found');  // Not structured

// ❌ BAD: No error codes
throw new RpcException('Error');  // Client can't handle programmatically

// ❌ BAD: Exposing internal details
throw new RpcException({
  message: error.stack,  // Security risk!
  dbPassword: 'secret',  // Never expose secrets!
});

// ❌ BAD: Not catching RpcException on client
await this.client.send('cmd', data);  // Unhandled errors crash app
```

**Common Error Patterns:**

```typescript
// ===== Standard Error Responses =====

// 1. Not Found (404)
throw new RpcException({
  statusCode: 404,
  message: 'Resource not found',
  error: 'NOT_FOUND',
  resource: 'User',
  id: userId,
});

// 2. Validation Error (400)
throw new RpcException({
  statusCode: 400,
  message: 'Validation failed',
  error: 'VALIDATION_ERROR',
  details: {
    email: 'Invalid email format',
    age: 'Must be at least 18',
  },
});

// 3. Unauthorized (401)
throw new RpcException({
  statusCode: 401,
  message: 'Authentication required',
  error: 'UNAUTHORIZED',
});

// 4. Forbidden (403)
throw new RpcException({
  statusCode: 403,
  message: 'Insufficient permissions',
  error: 'FORBIDDEN',
  requiredRole: 'admin',
});

// 5. Conflict (409)
throw new RpcException({
  statusCode: 409,
  message: 'Resource already exists',
  error: 'CONFLICT',
  field: 'email',
});

// 6. Internal Server Error (500)
throw new RpcException({
  statusCode: 500,
  message: 'Internal server error',
  error: 'INTERNAL_ERROR',
  requestId: generateRequestId(),
});

// 7. Service Unavailable (503)
throw new RpcException({
  statusCode: 503,
  message: 'Service temporarily unavailable',
  error: 'SERVICE_UNAVAILABLE',
  retryAfter: 60,  // seconds
});
```

**Summary:**

```
RpcException:
• Purpose: Throw errors from microservices
• Package: @nestjs/microservices
• Works with: All transports (TCP, Redis, RabbitMQ, Kafka, gRPC, MQTT)
• Replaces: HttpException (which is for HTTP only)

Constructor:
new RpcException(error: string | object)

Common Usage:
new RpcException({
  statusCode: 404,
  message: 'User not found',
  error: 'NOT_FOUND',
})

Key Features:
✅ Serializable across network
✅ Works with all transports
✅ Caught by RpcExceptionFilter
✅ Can carry any JSON data
✅ Automatically propagated to client

Best Practices:
✅ Use in microservices (not HttpException)
✅ Structured error objects
✅ Include error codes (for programmatic handling)
✅ Add context (IDs, timestamps)
✅ Custom exception classes
✅ Log before throwing
✅ Catch and transform on client
✅ Use RpcExceptionFilter

Common Errors:
• 404: NOT_FOUND
• 400: VALIDATION_ERROR
• 401: UNAUTHORIZED
• 403: FORBIDDEN
• 409: CONFLICT
• 500: INTERNAL_ERROR
• 503: SERVICE_UNAVAILABLE

Client Handling:
try {
  await client.send('cmd', data);
} catch (error) {
  // error is the object passed to RpcException
  if (error.statusCode === 404) { ... }
}
```

**Interview Tip**: **`RpcException`** is NestJS's exception class for **microservices** (not HTTP). **Purpose**: throw errors that are **serialized and sent to client** over any transport (TCP, Redis, RabbitMQ, Kafka, gRPC). **Difference from HttpException**: HttpException is HTTP-only, RpcException works with all microservice transports. **Constructor**: `new RpcException(error)` where error can be string or object. **Common pattern**: `new RpcException({ statusCode: 404, message: 'Not found', error: 'NOT_FOUND' })`. **Client receives**: exact object passed to RpcException. **Best practices**: structured errors with statusCode/message/error code, custom exception classes, caught by RpcExceptionFilter. **Why needed**: proper error handling across microservices, transport-agnostic, serializable. **Package**: `@nestjs/microservices`.

</details>

### ### 42. How do you throw exceptions from microservices?

<details>
<summary>Answer</summary>

**Throwing exceptions from microservices** requires using **`RpcException`** (not `HttpException`) and following proper error handling patterns. The exception is **serialized** and **transmitted** to the client through the transport layer (TCP, Redis, RabbitMQ, etc.).

**Basic Exception Throwing:**

```typescript
// ===== Basic Pattern: Throw RpcException =====

import { Injectable } from '@nestjs/common';
import { RpcException } from '@nestjs/microservices';

@Injectable()
export class UsersService {
  async getUserById(id: string) {
    const user = await this.usersRepository.findOne({ where: { id } });
    
    if (!user) {
      // Step 1: Create RpcException with error data
      throw new RpcException({
        statusCode: 404,
        message: 'User not found',
        error: 'USER_NOT_FOUND',
        userId: id,
      });
      // This error object will be sent to the client
    }
    
    return user;
  }
}

// Controller (Message Handler)
import { Controller } from '@nestjs/common';
import { MessagePattern } from '@nestjs/microservices';

@Controller()
export class UsersController {
  constructor(private usersService: UsersService) {}
  
  @MessagePattern({ cmd: 'get_user' })
  async getUser(id: string) {
    // Exceptions thrown in service will propagate to client
    return await this.usersService.getUserById(id);
  }
}
```

**Complete Exception Handling Flow:**

```typescript
// ===== Server Side (Microservice) =====

import { Controller, Injectable } from '@nestjs/common';
import { MessagePattern, RpcException } from '@nestjs/microservices';

@Injectable()
export class UsersService {
  async createUser(userData: CreateUserDto) {
    try {
      // Validation
      if (!userData.email) {
        throw new RpcException({
          statusCode: 400,
          message: 'Email is required',
          error: 'VALIDATION_ERROR',
          field: 'email',
        });
      }
      
      // Business logic check
      const existing = await this.usersRepository.findOne({
        where: { email: userData.email },
      });
      
      if (existing) {
        throw new RpcException({
          statusCode: 409,
          message: 'Email already exists',
          error: 'DUPLICATE_EMAIL',
          email: userData.email,
        });
      }
      
      // Create user
      return await this.usersRepository.create(userData);
      
    } catch (error) {
      // Re-throw RpcException as-is
      if (error instanceof RpcException) {
        throw error;
      }
      
      // Handle database errors
      if (error.code === '23505') {  // Postgres unique violation
        throw new RpcException({
          statusCode: 409,
          message: 'Email already exists',
          error: 'DUPLICATE_EMAIL',
        });
      }
      
      // Unknown error - log and throw generic error
      console.error('Unexpected error creating user:', error);
      throw new RpcException({
        statusCode: 500,
        message: 'Failed to create user',
        error: 'INTERNAL_ERROR',
      });
    }
  }
}

@Controller()
export class UsersController {
  constructor(private usersService: UsersService) {}
  
  @MessagePattern({ cmd: 'create_user' })
  async createUser(userData: CreateUserDto) {
    // Exceptions propagate automatically
    return await this.usersService.createUser(userData);
  }
}

// ===== Client Side (Consumer) =====

import { Injectable, Inject, HttpException, HttpStatus } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { firstValueFrom, catchError } from 'rxjs';

@Injectable()
export class OrdersService {
  constructor(
    @Inject('USERS_SERVICE') private usersClient: ClientProxy,
  ) {}
  
  async createOrder(orderData: CreateOrderDto) {
    try {
      // Call microservice
      const user = await firstValueFrom(
        this.usersClient.send({ cmd: 'get_user' }, orderData.userId),
      );
      
      return this.ordersRepository.create({
        ...orderData,
        userName: user.name,
      });
      
    } catch (error) {
      // Catch RpcException error
      console.error('Error from users service:', error);
      
      // Transform to HTTP exception for REST API
      if (error.statusCode === 404) {
        throw new HttpException(
          error.message || 'User not found',
          HttpStatus.NOT_FOUND,
        );
      }
      
      if (error.error === 'VALIDATION_ERROR') {
        throw new HttpException(
          error.message || 'Validation failed',
          HttpStatus.BAD_REQUEST,
        );
      }
      
      // Generic error
      throw new HttpException(
        'Service temporarily unavailable',
        HttpStatus.SERVICE_UNAVAILABLE,
      );
    }
  }
}
```

**Different Exception Scenarios:**

**1. Validation Errors:**

```typescript
// ===== Validation Errors =====

import { IsEmail, IsNotEmpty, validate } from 'class-validator';

export class CreateUserDto {
  @IsEmail()
  email: string;
  
  @IsNotEmpty()
  name: string;
}

@Injectable()
export class UsersService {
  async createUser(userData: CreateUserDto) {
    // Validate DTO
    const errors = await validate(userData);
    
    if (errors.length > 0) {
      const validationErrors = errors.reduce((acc, error) => {
        acc[error.property] = Object.values(error.constraints).join(', ');
        return acc;
      }, {});
      
      throw new RpcException({
        statusCode: 400,
        message: 'Validation failed',
        error: 'VALIDATION_ERROR',
        details: validationErrors,
      });
    }
    
    return await this.usersRepository.create(userData);
  }
}

// Error received by client:
// {
//   statusCode: 400,
//   message: 'Validation failed',
//   error: 'VALIDATION_ERROR',
//   details: {
//     email: 'email must be an email',
//     name: 'name should not be empty',
//   },
// }
```

**2. Business Logic Errors:**

```typescript
// ===== Business Logic Errors =====

@Injectable()
export class OrdersService {
  async createOrder(orderData: CreateOrderDto) {
    // Check inventory
    const product = await this.productsRepository.findOne({
      where: { id: orderData.productId },
    });
    
    if (!product) {
      throw new RpcException({
        statusCode: 404,
        message: 'Product not found',
        error: 'PRODUCT_NOT_FOUND',
        productId: orderData.productId,
      });
    }
    
    // Check stock
    if (product.stock < orderData.quantity) {
      throw new RpcException({
        statusCode: 400,
        message: 'Insufficient stock',
        error: 'INSUFFICIENT_STOCK',
        available: product.stock,
        requested: orderData.quantity,
      });
    }
    
    // Check user credit
    const user = await this.usersRepository.findOne({
      where: { id: orderData.userId },
    });
    
    if (user.credit < orderData.total) {
      throw new RpcException({
        statusCode: 402,  // Payment Required
        message: 'Insufficient credit',
        error: 'INSUFFICIENT_CREDIT',
        required: orderData.total,
        available: user.credit,
      });
    }
    
    return await this.ordersRepository.create(orderData);
  }
}
```

**3. Authorization Errors:**

```typescript
// ===== Authorization Errors =====

@Injectable()
export class UsersService {
  async deleteUser(userId: string, requestingUserId: string) {
    // Check if user exists
    const user = await this.usersRepository.findOne({ where: { id: userId } });
    
    if (!user) {
      throw new RpcException({
        statusCode: 404,
        message: 'User not found',
        error: 'USER_NOT_FOUND',
      });
    }
    
    // Check if requesting user is admin or owner
    const requestingUser = await this.usersRepository.findOne({
      where: { id: requestingUserId },
    });
    
    if (!requestingUser) {
      throw new RpcException({
        statusCode: 401,
        message: 'Authentication required',
        error: 'UNAUTHORIZED',
      });
    }
    
    const isAdmin = requestingUser.role === 'admin';
    const isOwner = requestingUserId === userId;
    
    if (!isAdmin && !isOwner) {
      throw new RpcException({
        statusCode: 403,
        message: 'Insufficient permissions to delete this user',
        error: 'FORBIDDEN',
        requiredRole: 'admin',
        currentRole: requestingUser.role,
      });
    }
    
    await this.usersRepository.delete({ id: userId });
    return { success: true };
  }
}
```

**4. Database Errors:**

```typescript
// ===== Database Errors =====

@Injectable()
export class UsersService {
  async createUser(userData: CreateUserDto) {
    try {
      return await this.usersRepository.create(userData);
    } catch (error) {
      console.error('Database error:', error);
      
      // PostgreSQL error codes
      switch (error.code) {
        case '23505':  // Unique violation
          throw new RpcException({
            statusCode: 409,
            message: 'Email already exists',
            error: 'DUPLICATE_EMAIL',
            field: this.extractFieldFromError(error),
          });
        
        case '23503':  // Foreign key violation
          throw new RpcException({
            statusCode: 400,
            message: 'Referenced entity does not exist',
            error: 'FOREIGN_KEY_VIOLATION',
            field: this.extractFieldFromError(error),
          });
        
        case '23502':  // Not null violation
          throw new RpcException({
            statusCode: 400,
            message: 'Required field is missing',
            error: 'NOT_NULL_VIOLATION',
            field: this.extractFieldFromError(error),
          });
        
        case '22001':  // String too long
          throw new RpcException({
            statusCode: 400,
            message: 'Field value too long',
            error: 'VALUE_TOO_LONG',
          });
        
        default:
          throw new RpcException({
            statusCode: 500,
            message: 'Database operation failed',
            error: 'DATABASE_ERROR',
          });
      }
    }
  }
  
  private extractFieldFromError(error: any): string {
    // Extract field name from error message
    const match = error.detail?.match(/Key \((\w+)\)/);
    return match ? match[1] : 'unknown';
  }
}
```

**5. External Service Errors:**

```typescript
// ===== External Service Errors =====

import { HttpService } from '@nestjs/axios';
import { firstValueFrom, catchError, timeout } from 'rxjs';

@Injectable()
export class PaymentService {
  constructor(private httpService: HttpService) {}
  
  async processPayment(paymentData: PaymentDto) {
    try {
      // Call external payment API
      const response = await firstValueFrom(
        this.httpService.post('https://payment-api.com/charge', paymentData).pipe(
          timeout(10000),  // 10 second timeout
          catchError((error) => {
            if (error.name === 'TimeoutError') {
              throw new RpcException({
                statusCode: 504,
                message: 'Payment service timeout',
                error: 'PAYMENT_TIMEOUT',
              });
            }
            throw error;
          }),
        ),
      );
      
      return response.data;
      
    } catch (error) {
      // Already RpcException
      if (error instanceof RpcException) {
        throw error;
      }
      
      // HTTP error from external service
      if (error.response) {
        const status = error.response.status;
        const data = error.response.data;
        
        if (status === 402) {
          throw new RpcException({
            statusCode: 402,
            message: 'Payment declined',
            error: 'PAYMENT_DECLINED',
            reason: data.reason,
          });
        }
        
        if (status >= 500) {
          throw new RpcException({
            statusCode: 503,
            message: 'Payment service unavailable',
            error: 'PAYMENT_SERVICE_UNAVAILABLE',
          });
        }
      }
      
      // Network error
      if (error.code === 'ECONNREFUSED') {
        throw new RpcException({
          statusCode: 503,
          message: 'Cannot connect to payment service',
          error: 'CONNECTION_REFUSED',
        });
      }
      
      // Generic error
      console.error('Payment processing error:', error);
      throw new RpcException({
        statusCode: 500,
        message: 'Payment processing failed',
        error: 'PAYMENT_ERROR',
      });
    }
  }
}
```

**Custom Exception Classes:**

```typescript
// ===== Custom Exception Classes =====

// Base custom exception
export class ServiceException extends RpcException {
  constructor(
    statusCode: number,
    error: string,
    message: string,
    data?: Record<string, any>,
  ) {
    super({
      statusCode,
      error,
      message,
      timestamp: new Date().toISOString(),
      ...data,
    });
  }
}

// Specific exceptions
export class UserNotFoundException extends ServiceException {
  constructor(userId: string) {
    super(404, 'USER_NOT_FOUND', `User with ID ${userId} not found`, { userId });
  }
}

export class DuplicateEmailException extends ServiceException {
  constructor(email: string) {
    super(409, 'DUPLICATE_EMAIL', 'Email already exists', { email });
  }
}

export class InsufficientStockException extends ServiceException {
  constructor(productId: string, available: number, requested: number) {
    super(
      400,
      'INSUFFICIENT_STOCK',
      'Not enough items in stock',
      { productId, available, requested },
    );
  }
}

export class UnauthorizedException extends ServiceException {
  constructor(message: string = 'Authentication required') {
    super(401, 'UNAUTHORIZED', message);
  }
}

export class ForbiddenException extends ServiceException {
  constructor(requiredRole?: string) {
    super(
      403,
      'FORBIDDEN',
      'Insufficient permissions',
      requiredRole ? { requiredRole } : undefined,
    );
  }
}

// Usage
if (!user) {
  throw new UserNotFoundException(userId);
}

if (existing) {
  throw new DuplicateEmailException(userData.email);
}

if (product.stock < quantity) {
  throw new InsufficientStockException(productId, product.stock, quantity);
}

if (!authenticated) {
  throw new UnauthorizedException();
}

if (!authorized) {
  throw new ForbiddenException('admin');
}
```

**Global Exception Filter:**

```typescript
// ===== Global Exception Filter =====

import { Catch, RpcExceptionFilter, ArgumentsHost } from '@nestjs/common';
import { RpcException } from '@nestjs/microservices';
import { Observable, throwError } from 'rxjs';

@Catch()
export class AllExceptionsFilter implements RpcExceptionFilter {
  catch(exception: any, host: ArgumentsHost): Observable<any> {
    console.error('Exception caught:', exception);
    
    // RpcException - pass through with formatting
    if (exception instanceof RpcException) {
      const error = exception.getError();
      return throwError(() => ({
        ...error,
        timestamp: error['timestamp'] || new Date().toISOString(),
      }));
    }
    
    // Database errors
    if (exception.code) {
      return throwError(() => ({
        statusCode: 500,
        message: 'Database error',
        error: 'DATABASE_ERROR',
        code: exception.code,
        timestamp: new Date().toISOString(),
      }));
    }
    
    // Validation errors (class-validator)
    if (Array.isArray(exception) && exception[0]?.constraints) {
      const validationErrors = exception.reduce((acc, err) => {
        acc[err.property] = Object.values(err.constraints).join(', ');
        return acc;
      }, {});
      
      return throwError(() => ({
        statusCode: 400,
        message: 'Validation failed',
        error: 'VALIDATION_ERROR',
        details: validationErrors,
        timestamp: new Date().toISOString(),
      }));
    }
    
    // Generic Error
    return throwError(() => ({
      statusCode: 500,
      message: exception.message || 'Internal server error',
      error: 'INTERNAL_ERROR',
      timestamp: new Date().toISOString(),
    }));
  }
}

// Apply in main.ts
import { NestFactory } from '@nestjs/core';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.TCP,
      options: { port: 3001 },
    },
  );
  
  app.useGlobalFilters(new AllExceptionsFilter());
  
  await app.listen();
}
bootstrap();
```

**Best Practices:**

```typescript
// ✅ GOOD: Use RpcException (not HttpException or Error)
throw new RpcException({
  statusCode: 404,
  message: 'User not found',
  error: 'USER_NOT_FOUND',
});

// ✅ GOOD: Include structured error data
throw new RpcException({
  statusCode: 400,
  message: 'Validation failed',
  error: 'VALIDATION_ERROR',
  details: { email: 'Invalid format', age: 'Must be 18+' },
});

// ✅ GOOD: Add error codes for programmatic handling
error: 'USER_NOT_FOUND'  // Client can check this

// ✅ GOOD: Include context (IDs, values)
throw new RpcException({
  statusCode: 400,
  message: 'Insufficient stock',
  error: 'INSUFFICIENT_STOCK',
  productId,
  available: 5,
  requested: 10,
});

// ✅ GOOD: Log errors before throwing
console.error('Failed to create user:', { error, userId });
throw new RpcException(...);

// ✅ GOOD: Catch and re-throw with context
try {
  await operation();
} catch (error) {
  if (error instanceof RpcException) throw error;
  console.error('Unexpected error:', error);
  throw new RpcException({
    statusCode: 500,
    message: 'Operation failed',
    error: 'INTERNAL_ERROR',
  });
}

// ✅ GOOD: Use custom exception classes
throw new UserNotFoundException(userId);

// ✅ GOOD: Apply global exception filter
app.useGlobalFilters(new AllExceptionsFilter());

// ❌ BAD: Using HttpException
throw new HttpException('Error', 404);  // Won't work in microservices!

// ❌ BAD: Throwing plain Error
throw new Error('User not found');  // Not structured

// ❌ BAD: No error codes
throw new RpcException('Error');  // Can't handle programmatically

// ❌ BAD: Exposing sensitive data
throw new RpcException({
  password: user.password,  // Security risk!
  dbConnection: process.env.DB_URL,  // Never expose!
});

// ❌ BAD: Not catching errors
async getUser(id: string) {
  return await this.usersRepository.findOne({ where: { id } });
  // If error occurs, crashes service
}

// ❌ BAD: Generic error messages
throw new RpcException('Error');  // Not helpful
```

**Summary:**

```
How to Throw Exceptions from Microservices:

1. Use RpcException (not HttpException or Error)
   import { RpcException } from '@nestjs/microservices';
   
2. Throw with structured data:
   throw new RpcException({
     statusCode: 404,
     message: 'User not found',
     error: 'USER_NOT_FOUND',
     userId,
   });

3. Handle different error types:
   • Validation errors (400)
   • Not found (404)
   • Unauthorized (401)
   • Forbidden (403)
   • Conflict (409)
   • Internal errors (500)
   • Service unavailable (503)

4. Catch and transform errors:
   try { ... }
   catch (error) {
     if (error instanceof RpcException) throw error;
     // Transform other errors to RpcException
   }

5. Use custom exception classes:
   class UserNotFoundException extends RpcException { ... }

6. Apply global exception filter:
   app.useGlobalFilters(new AllExceptionsFilter());

7. Client catches errors:
   try {
     await client.send('cmd', data);
   } catch (error) {
     // error is object from RpcException
   }

Best Practices:
✅ Use RpcException
✅ Structured error objects
✅ Error codes (for programmatic handling)
✅ Include context (IDs, values)
✅ Log before throwing
✅ Catch and re-throw with context
✅ Custom exception classes
✅ Global exception filter
✅ Don't expose sensitive data

Flow:
Service → throw RpcException → Serialized → Transport → Client catches
```

**Interview Tip**: **Throw exceptions from microservices** using **`RpcException`** (not HttpException). **Pattern**: `throw new RpcException({ statusCode: 404, message: 'Not found', error: 'NOT_FOUND' })`. **Key differences**: HttpException is HTTP-only, RpcException works with all transports (TCP, Redis, RabbitMQ, Kafka, gRPC). **Error handling**: catch try-catch, re-throw RpcException as-is, transform database/external errors to RpcException. **Best practices**: structured errors (statusCode/message/error code), include context (IDs, values), log before throwing, custom exception classes (UserNotFoundException), global exception filter. **Client receives**: exact object passed to RpcException. **Common errors**: 404 NOT_FOUND, 400 VALIDATION_ERROR, 401 UNAUTHORIZED, 403 FORBIDDEN, 409 CONFLICT, 500 INTERNAL_ERROR, 503 SERVICE_UNAVAILABLE. **Why**: proper serialization across transports, structured error responses, programmatic error handling.

</details>

## Testing Microservices

### 43. How do you test microservices?

<details>
<summary>Answer</summary>

**Testing microservices** involves **unit tests** (test individual components), **integration tests** (test service communication), and **end-to-end tests** (test entire system). Testing microservices is more complex than monoliths due to distributed nature, requiring **mocking**, **test doubles**, **contract testing**, and **isolation techniques**.

**Testing Pyramid for Microservices:**

```typescript
// ===== Testing Layers =====

/*
  E2E Tests (Few)
       /\
      /  \
     /    \
    /______\
  Integration Tests (Some)
       /\
      /  \
     /    \
    /______\
  Unit Tests (Many)

1. Unit Tests (70%):
   - Test individual functions/methods
   - Mock all dependencies
   - Fast, isolated, deterministic

2. Integration Tests (20%):
   - Test service communication
   - Test database interactions
   - Mock external services

3. E2E Tests (10%):
   - Test complete user flows
   - Test all services together
   - Slow, complex, realistic
*/
```

**1. Unit Testing Message Handlers:**

```typescript
// ===== Unit Test: Message Handler =====

import { Test, TestingModule } from '@nestjs/testing';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';
import { RpcException } from '@nestjs/microservices';

describe('UsersController', () => {
  let controller: UsersController;
  let service: UsersService;
  
  // Mock service
  const mockUsersService = {
    getUserById: jest.fn(),
    createUser: jest.fn(),
    updateUser: jest.fn(),
    deleteUser: jest.fn(),
  };
  
  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      controllers: [UsersController],
      providers: [
        {
          provide: UsersService,
          useValue: mockUsersService,
        },
      ],
    }).compile();
    
    controller = module.get<UsersController>(UsersController);
    service = module.get<UsersService>(UsersService);
  });
  
  afterEach(() => {
    jest.clearAllMocks();
  });
  
  describe('getUser', () => {
    it('should return user when found', async () => {
      // Arrange
      const userId = '123';
      const expectedUser = {
        id: userId,
        name: 'John Doe',
        email: 'john@example.com',
      };
      mockUsersService.getUserById.mockResolvedValue(expectedUser);
      
      // Act
      const result = await controller.getUser(userId);
      
      // Assert
      expect(result).toEqual(expectedUser);
      expect(service.getUserById).toHaveBeenCalledWith(userId);
      expect(service.getUserById).toHaveBeenCalledTimes(1);
    });
    
    it('should throw RpcException when user not found', async () => {
      // Arrange
      const userId = '999';
      mockUsersService.getUserById.mockRejectedValue(
        new RpcException({
          statusCode: 404,
          message: 'User not found',
          error: 'USER_NOT_FOUND',
        }),
      );
      
      // Act & Assert
      await expect(controller.getUser(userId)).rejects.toThrow(RpcException);
      expect(service.getUserById).toHaveBeenCalledWith(userId);
    });
  });
  
  describe('createUser', () => {
    it('should create and return new user', async () => {
      // Arrange
      const userData = {
        name: 'Jane Doe',
        email: 'jane@example.com',
      };
      const expectedUser = {
        id: '456',
        ...userData,
      };
      mockUsersService.createUser.mockResolvedValue(expectedUser);
      
      // Act
      const result = await controller.createUser(userData);
      
      // Assert
      expect(result).toEqual(expectedUser);
      expect(service.createUser).toHaveBeenCalledWith(userData);
    });
    
    it('should throw RpcException for duplicate email', async () => {
      // Arrange
      const userData = {
        name: 'Jane Doe',
        email: 'existing@example.com',
      };
      mockUsersService.createUser.mockRejectedValue(
        new RpcException({
          statusCode: 409,
          message: 'Email already exists',
          error: 'DUPLICATE_EMAIL',
        }),
      );
      
      // Act & Assert
      await expect(controller.createUser(userData)).rejects.toThrow(RpcException);
    });
  });
});
```

**2. Unit Testing Services:**

```typescript
// ===== Unit Test: Service with Repository =====

import { Test, TestingModule } from '@nestjs/testing';
import { getRepositoryToken } from '@nestjs/typeorm';
import { UsersService } from './users.service';
import { User } from './user.entity';
import { RpcException } from '@nestjs/microservices';
import { Repository } from 'typeorm';

describe('UsersService', () => {
  let service: UsersService;
  let repository: Repository<User>;
  
  // Mock repository
  const mockRepository = {
    findOne: jest.fn(),
    create: jest.fn(),
    save: jest.fn(),
    update: jest.fn(),
    delete: jest.fn(),
  };
  
  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User),
          useValue: mockRepository,
        },
      ],
    }).compile();
    
    service = module.get<UsersService>(UsersService);
    repository = module.get<Repository<User>>(getRepositoryToken(User));
  });
  
  afterEach(() => {
    jest.clearAllMocks();
  });
  
  describe('getUserById', () => {
    it('should return user when found', async () => {
      // Arrange
      const user = { id: '123', name: 'John', email: 'john@example.com' };
      mockRepository.findOne.mockResolvedValue(user);
      
      // Act
      const result = await service.getUserById('123');
      
      // Assert
      expect(result).toEqual(user);
      expect(repository.findOne).toHaveBeenCalledWith({ where: { id: '123' } });
    });
    
    it('should throw RpcException when user not found', async () => {
      // Arrange
      mockRepository.findOne.mockResolvedValue(null);
      
      // Act & Assert
      await expect(service.getUserById('999')).rejects.toThrow(RpcException);
      await expect(service.getUserById('999')).rejects.toMatchObject({
        error: expect.objectContaining({
          statusCode: 404,
          error: 'USER_NOT_FOUND',
        }),
      });
    });
  });
  
  describe('createUser', () => {
    it('should create user successfully', async () => {
      // Arrange
      const userData = { name: 'Jane', email: 'jane@example.com' };
      const createdUser = { id: '456', ...userData };
      
      mockRepository.findOne.mockResolvedValue(null);  // No duplicate
      mockRepository.create.mockReturnValue(createdUser);
      mockRepository.save.mockResolvedValue(createdUser);
      
      // Act
      const result = await service.createUser(userData);
      
      // Assert
      expect(result).toEqual(createdUser);
      expect(repository.findOne).toHaveBeenCalledWith({
        where: { email: userData.email },
      });
      expect(repository.create).toHaveBeenCalledWith(userData);
      expect(repository.save).toHaveBeenCalledWith(createdUser);
    });
    
    it('should throw RpcException for duplicate email', async () => {
      // Arrange
      const userData = { name: 'Jane', email: 'existing@example.com' };
      const existingUser = { id: '789', ...userData };
      mockRepository.findOne.mockResolvedValue(existingUser);
      
      // Act & Assert
      await expect(service.createUser(userData)).rejects.toThrow(RpcException);
      expect(repository.create).not.toHaveBeenCalled();
      expect(repository.save).not.toHaveBeenCalled();
    });
  });
});
```

**3. Integration Testing with ClientProxy:**

```typescript
// ===== Integration Test: Service Communication =====

import { Test, TestingModule } from '@nestjs/testing';
import { ClientsModule, Transport, ClientProxy } from '@nestjs/microservices';
import { OrdersService } from './orders.service';
import { of, throwError } from 'rxjs';

describe('OrdersService Integration', () => {
  let service: OrdersService;
  let usersClient: ClientProxy;
  
  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      imports: [
        ClientsModule.register([
          {
            name: 'USERS_SERVICE',
            transport: Transport.TCP,
            options: {
              host: 'localhost',
              port: 3001,
            },
          },
        ]),
      ],
      providers: [OrdersService],
    }).compile();
    
    service = module.get<OrdersService>(OrdersService);
    usersClient = module.get<ClientProxy>('USERS_SERVICE');
    
    // Connect to microservice
    await usersClient.connect();
  });
  
  afterEach(async () => {
    await usersClient.close();
  });
  
  it('should create order with user data', async () => {
    // This test requires the users microservice to be running
    // Better to use mocks for CI/CD
    
    const orderData = {
      userId: '123',
      productId: '456',
      quantity: 2,
    };
    
    const result = await service.createOrder(orderData);
    
    expect(result).toBeDefined();
    expect(result.userName).toBeDefined();
  }, 10000);  // Increase timeout for integration test
});
```

**4. Testing with Mocked ClientProxy:**

```typescript
// ===== Test with Mocked ClientProxy =====

import { Test, TestingModule } from '@nestjs/testing';
import { ClientProxy } from '@nestjs/microservices';
import { OrdersService } from './orders.service';
import { of, throwError } from 'rxjs';

describe('OrdersService with Mock', () => {
  let service: OrdersService;
  let usersClient: ClientProxy;
  
  // Mock ClientProxy
  const mockClientProxy = {
    send: jest.fn(),
    emit: jest.fn(),
    connect: jest.fn().mockResolvedValue(undefined),
    close: jest.fn().mockResolvedValue(undefined),
  };
  
  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        OrdersService,
        {
          provide: 'USERS_SERVICE',
          useValue: mockClientProxy,
        },
      ],
    }).compile();
    
    service = module.get<OrdersService>(OrdersService);
    usersClient = module.get<ClientProxy>('USERS_SERVICE');
  });
  
  afterEach(() => {
    jest.clearAllMocks();
  });
  
  describe('createOrder', () => {
    it('should create order successfully', async () => {
      // Arrange
      const orderData = {
        userId: '123',
        productId: '456',
        quantity: 2,
      };
      const mockUser = {
        id: '123',
        name: 'John Doe',
        email: 'john@example.com',
      };
      
      // Mock send to return user
      mockClientProxy.send.mockReturnValue(of(mockUser));
      
      // Act
      const result = await service.createOrder(orderData);
      
      // Assert
      expect(result).toBeDefined();
      expect(result.userName).toBe('John Doe');
      expect(usersClient.send).toHaveBeenCalledWith(
        { cmd: 'get_user' },
        orderData.userId,
      );
    });
    
    it('should handle user not found error', async () => {
      // Arrange
      const orderData = {
        userId: '999',
        productId: '456',
        quantity: 2,
      };
      
      // Mock send to throw error
      mockClientProxy.send.mockReturnValue(
        throwError(() => ({
          statusCode: 404,
          message: 'User not found',
          error: 'USER_NOT_FOUND',
        })),
      );
      
      // Act & Assert
      await expect(service.createOrder(orderData)).rejects.toThrow();
    });
  });
});
```

**5. E2E Testing:**

```typescript
// ===== E2E Test: Complete Flow =====

import { Test, TestingModule } from '@nestjs/testing';
import { INestMicroservice } from '@nestjs/common';
import { MicroserviceOptions, Transport, ClientProxy, ClientsModule } from '@nestjs/microservices';
import { AppModule } from '../src/app.module';

describe('Users Microservice E2E', () => {
  let app: INestMicroservice;
  let client: ClientProxy;
  
  beforeAll(async () => {
    // Create microservice
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();
    
    app = moduleFixture.createNestMicroservice<MicroserviceOptions>({
      transport: Transport.TCP,
      options: {
        host: 'localhost',
        port: 3001,
      },
    });
    
    await app.listen();
    
    // Create client
    const clientModule = await Test.createTestingModule({
      imports: [
        ClientsModule.register([
          {
            name: 'USERS_SERVICE',
            transport: Transport.TCP,
            options: {
              host: 'localhost',
              port: 3001,
            },
          },
        ]),
      ],
    }).compile();
    
    client = clientModule.get<ClientProxy>('USERS_SERVICE');
    await client.connect();
  });
  
  afterAll(async () => {
    await client.close();
    await app.close();
  });
  
  describe('User CRUD Operations', () => {
    let createdUserId: string;
    
    it('should create a user', (done) => {
      client
        .send({ cmd: 'create_user' }, {
          name: 'Test User',
          email: 'test@example.com',
        })
        .subscribe({
          next: (result) => {
            expect(result).toBeDefined();
            expect(result.id).toBeDefined();
            expect(result.name).toBe('Test User');
            expect(result.email).toBe('test@example.com');
            createdUserId = result.id;
            done();
          },
          error: done,
        });
    });
    
    it('should get user by id', (done) => {
      client
        .send({ cmd: 'get_user' }, createdUserId)
        .subscribe({
          next: (result) => {
            expect(result).toBeDefined();
            expect(result.id).toBe(createdUserId);
            expect(result.name).toBe('Test User');
            done();
          },
          error: done,
        });
    });
    
    it('should update user', (done) => {
      client
        .send({ cmd: 'update_user' }, {
          id: createdUserId,
          name: 'Updated User',
        })
        .subscribe({
          next: (result) => {
            expect(result).toBeDefined();
            expect(result.name).toBe('Updated User');
            done();
          },
          error: done,
        });
    });
    
    it('should delete user', (done) => {
      client
        .send({ cmd: 'delete_user' }, createdUserId)
        .subscribe({
          next: (result) => {
            expect(result).toBeDefined();
            expect(result.success).toBe(true);
            done();
          },
          error: done,
        });
    });
    
    it('should return error for non-existent user', (done) => {
      client
        .send({ cmd: 'get_user' }, '999')
        .subscribe({
          next: () => done(new Error('Should have thrown error')),
          error: (error) => {
            expect(error.statusCode).toBe(404);
            expect(error.error).toBe('USER_NOT_FOUND');
            done();
          },
        });
    });
  });
});
```

**6. Contract Testing:**

```typescript
// ===== Contract Testing with Pact =====

// npm install --save-dev @pact-foundation/pact

import { Pact } from '@pact-foundation/pact';
import { like, term } from '@pact-foundation/pact/dsl/matchers';
import { ClientProxy } from '@nestjs/microservices';

describe('Users Service Contract', () => {
  const provider = new Pact({
    consumer: 'OrdersService',
    provider: 'UsersService',
    port: 3001,
  });
  
  beforeAll(() => provider.setup());
  afterAll(() => provider.finalize());
  afterEach(() => provider.verify());
  
  describe('get user by id', () => {
    it('should return user when exists', async () => {
      // Define expected interaction
      await provider.addInteraction({
        state: 'user with id 123 exists',
        uponReceiving: 'a request for user 123',
        withRequest: {
          method: 'POST',
          path: '/users/get',
          body: { cmd: 'get_user', data: '123' },
        },
        willRespondWith: {
          status: 200,
          body: {
            id: '123',
            name: like('John Doe'),
            email: term({
              matcher: '.+@.+\\..+',
              generate: 'john@example.com',
            }),
          },
        },
      });
      
      // Test the interaction
      // Make actual request and verify
    });
  });
});
```

**7. Performance Testing:**

```typescript
// ===== Performance/Load Testing =====

import { Test } from '@nestjs/testing';
import { performance } from 'perf_hooks';

describe('Performance Tests', () => {
  let service: UsersService;
  
  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [UsersService],
    }).compile();
    
    service = module.get<UsersService>(UsersService);
  });
  
  it('should handle 1000 concurrent requests', async () => {
    const start = performance.now();
    
    const promises = Array(1000)
      .fill(null)
      .map((_, i) => service.getUserById(`${i}`));
    
    await Promise.all(promises);
    
    const duration = performance.now() - start;
    
    expect(duration).toBeLessThan(5000);  // Should complete in 5 seconds
  }, 10000);
  
  it('should have acceptable response time', async () => {
    const iterations = 100;
    const times: number[] = [];
    
    for (let i = 0; i < iterations; i++) {
      const start = performance.now();
      await service.getUserById('123');
      const duration = performance.now() - start;
      times.push(duration);
    }
    
    const avgTime = times.reduce((a, b) => a + b, 0) / times.length;
    const maxTime = Math.max(...times);
    
    expect(avgTime).toBeLessThan(50);  // Avg < 50ms
    expect(maxTime).toBeLessThan(200);  // Max < 200ms
  });
});
```

**Best Practices:**

```typescript
// ✅ GOOD: Mock all external dependencies
const mockRepository = {
  findOne: jest.fn(),
  create: jest.fn(),
};

// ✅ GOOD: Test both success and error cases
it('should return user when found', async () => { ... });
it('should throw error when not found', async () => { ... });

// ✅ GOOD: Use descriptive test names
it('should throw RpcException when user not found', ...);

// ✅ GOOD: Arrange-Act-Assert pattern
it('should create user', async () => {
  // Arrange
  const userData = { ... };
  mockService.createUser.mockResolvedValue(...);
  
  // Act
  const result = await controller.createUser(userData);
  
  // Assert
  expect(result).toEqual(...);
});

// ✅ GOOD: Clear mocks between tests
afterEach(() => {
  jest.clearAllMocks();
});

// ✅ GOOD: Test error scenarios
it('should handle timeout error', async () => {
  mockClient.send.mockReturnValue(
    throwError(() => new Error('Timeout')),
  );
  await expect(service.callService()).rejects.toThrow();
});

// ✅ GOOD: Use integration tests sparingly
// Only for critical paths
// Mock external services

// ✅ GOOD: Isolate tests
// Each test should be independent
// Don't rely on test execution order

// ❌ BAD: Testing implementation details
expect(service['privateMethod']).toHaveBeenCalled();  // Don't test private methods

// ❌ BAD: Not mocking dependencies
// Real database calls in unit tests

// ❌ BAD: Too many E2E tests
// E2E tests are slow and brittle

// ❌ BAD: Shared state between tests
let sharedUser;  // Don't share state!

// ❌ BAD: No error testing
// Only testing happy path
```

**Test Configuration:**

```typescript
// ===== Jest Configuration =====

// jest.config.js
module.exports = {
  moduleFileExtensions: ['js', 'json', 'ts'],
  rootDir: 'src',
  testRegex: '.*\\.spec\\.ts$',
  transform: {
    '^.+\\.(t|j)s$': 'ts-jest',
  },
  collectCoverageFrom: [
    '**/*.(t|j)s',
    '!**/*.spec.ts',
    '!**/node_modules/**',
  ],
  coverageDirectory: '../coverage',
  testEnvironment: 'node',
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
};

// package.json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:cov": "jest --coverage",
    "test:e2e": "jest --config ./test/jest-e2e.json"
  }
}
```

**Summary:**

```
Testing Microservices:

1. Unit Tests (70%):
   • Test controllers and services in isolation
   • Mock all dependencies (repositories, ClientProxy)
   • Fast, deterministic, many tests
   • Test both success and error cases

2. Integration Tests (20%):
   • Test service communication
   • Test with real database (test DB)
   • Mock external services
   • Test message patterns

3. E2E Tests (10%):
   • Test complete user flows
   • Spin up microservices
   • Test real communication
   • Slow, use sparingly

4. Contract Tests:
   • Verify service contracts
   • Consumer-driven contracts
   • Pact framework

5. Performance Tests:
   • Load testing
   • Response time testing
   • Concurrent request handling

Key Tools:
• @nestjs/testing (Test module)
• Jest (Test runner)
• RxJS testing utilities
• Pact (Contract testing)

Best Practices:
✅ Test pyramid (many unit, some integration, few E2E)
✅ Mock external dependencies
✅ Arrange-Act-Assert pattern
✅ Test error scenarios
✅ Clear mocks between tests
✅ Descriptive test names
✅ Isolated tests (no shared state)
✅ Code coverage (aim for 80%+)
✅ Fast unit tests (<100ms)
✅ CI/CD integration
```

**Interview Tip**: **Test microservices** using **testing pyramid**: **70% unit tests** (mock dependencies, test controllers/services in isolation), **20% integration tests** (test service communication with mocked ClientProxy), **10% E2E tests** (spin up services, test complete flows). **Key techniques**: mock ClientProxy using `jest.fn()`, use `@nestjs/testing` Test module, mock repositories with `getRepositoryToken()`, test RpcException handling. **Patterns**: Arrange-Act-Assert, mock all external dependencies, test both success and error cases, clear mocks between tests. **Tools**: Jest, @nestjs/testing, RxJS testing utilities, Pact for contract testing. **Best practices**: isolated tests, descriptive names, fast unit tests, code coverage 80%+. **Different from monolith**: need to mock service communication, test distributed failures, contract testing.

</details>

### ### 44. How do you mock `ClientProxy` in tests?

<details>
<summary>Answer</summary>

**Mocking `ClientProxy`** is essential for testing services that communicate with other microservices. You create a **mock object** with `send()` and `emit()` methods that return **RxJS Observables**, then provide it in the testing module using the service **injection token**.

**Basic Mock Pattern:**

```typescript
// ===== Basic ClientProxy Mock =====

import { Test, TestingModule } from '@nestjs/testing';
import { ClientProxy } from '@nestjs/microservices';
import { OrdersService } from './orders.service';
import { of, throwError } from 'rxjs';

describe('OrdersService', () => {
  let service: OrdersService;
  let usersClient: ClientProxy;
  
  // Create mock ClientProxy
  const mockClientProxy = {
    send: jest.fn(),
    emit: jest.fn(),
    connect: jest.fn().mockResolvedValue(undefined),
    close: jest.fn().mockResolvedValue(undefined),
  };
  
  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        OrdersService,
        {
          // Provide mock using injection token
          provide: 'USERS_SERVICE',  // Must match @Inject() token
          useValue: mockClientProxy,
        },
      ],
    }).compile();
    
    service = module.get<OrdersService>(OrdersService);
    usersClient = module.get<ClientProxy>('USERS_SERVICE');
  });
  
  afterEach(() => {
    jest.clearAllMocks();
  });
  
  describe('createOrder', () => {
    it('should create order with user data', async () => {
      // Arrange
      const orderData = {
        userId: '123',
        productId: '456',
        quantity: 2,
      };
      
      const mockUser = {
        id: '123',
        name: 'John Doe',
        email: 'john@example.com',
      };
      
      // Mock send() to return Observable
      mockClientProxy.send.mockReturnValue(of(mockUser));
      
      // Act
      const result = await service.createOrder(orderData);
      
      // Assert
      expect(result).toBeDefined();
      expect(result.userName).toBe('John Doe');
      
      // Verify send was called correctly
      expect(usersClient.send).toHaveBeenCalledWith(
        { cmd: 'get_user' },
        '123',
      );
      expect(usersClient.send).toHaveBeenCalledTimes(1);
    });
  });
});
```

**Complete Mock Implementation:**

```typescript
// ===== Complete ClientProxy Mock =====

import { Test, TestingModule } from '@nestjs/testing';
import { ClientProxy } from '@nestjs/microservices';
import { OrdersService } from './orders.service';
import { of, throwError, EMPTY } from 'rxjs';
import { delay } from 'rxjs/operators';

describe('OrdersService with Complete Mock', () => {
  let service: OrdersService;
  let usersClient: ClientProxy;
  let inventoryClient: ClientProxy;
  
  // Mock for USERS_SERVICE
  const mockUsersClient = {
    send: jest.fn(),
    emit: jest.fn().mockReturnValue(EMPTY),  // emit returns void Observable
    connect: jest.fn().mockResolvedValue(undefined),
    close: jest.fn().mockResolvedValue(undefined),
  };
  
  // Mock for INVENTORY_SERVICE
  const mockInventoryClient = {
    send: jest.fn(),
    emit: jest.fn().mockReturnValue(EMPTY),
    connect: jest.fn().mockResolvedValue(undefined),
    close: jest.fn().mockResolvedValue(undefined),
  };
  
  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        OrdersService,
        {
          provide: 'USERS_SERVICE',
          useValue: mockUsersClient,
        },
        {
          provide: 'INVENTORY_SERVICE',
          useValue: mockInventoryClient,
        },
      ],
    }).compile();
    
    service = module.get<OrdersService>(OrdersService);
    usersClient = module.get<ClientProxy>('USERS_SERVICE');
    inventoryClient = module.get<ClientProxy>('INVENTORY_SERVICE');
  });
  
  afterEach(() => {
    jest.clearAllMocks();
  });
  
  describe('createOrder', () => {
    it('should create order successfully', async () => {
      // Arrange
      const orderData = {
        userId: '123',
        productId: '456',
        quantity: 2,
      };
      
      const mockUser = {
        id: '123',
        name: 'John Doe',
        email: 'john@example.com',
      };
      
      const mockInventory = {
        productId: '456',
        available: 10,
        reserved: 2,
      };
      
      // Mock both service calls
      mockUsersClient.send.mockReturnValue(of(mockUser));
      mockInventoryClient.send.mockReturnValue(of(mockInventory));
      
      // Act
      const result = await service.createOrder(orderData);
      
      // Assert
      expect(result).toBeDefined();
      expect(result.userName).toBe('John Doe');
      expect(result.quantity).toBe(2);
      
      // Verify calls
      expect(usersClient.send).toHaveBeenCalledWith(
        { cmd: 'get_user' },
        '123',
      );
      expect(inventoryClient.send).toHaveBeenCalledWith(
        { cmd: 'reserve_inventory' },
        { productId: '456', quantity: 2 },
      );
    });
    
    it('should handle user not found error', async () => {
      // Arrange
      const orderData = {
        userId: '999',
        productId: '456',
        quantity: 2,
      };
      
      // Mock error response
      mockUsersClient.send.mockReturnValue(
        throwError(() => ({
          statusCode: 404,
          message: 'User not found',
          error: 'USER_NOT_FOUND',
        })),
      );
      
      // Act & Assert
      await expect(service.createOrder(orderData)).rejects.toThrow();
      
      // Verify inventory was not called
      expect(inventoryClient.send).not.toHaveBeenCalled();
    });
    
    it('should handle insufficient inventory', async () => {
      // Arrange
      const orderData = {
        userId: '123',
        productId: '456',
        quantity: 100,
      };
      
      const mockUser = { id: '123', name: 'John Doe' };
      
      mockUsersClient.send.mockReturnValue(of(mockUser));
      mockInventoryClient.send.mockReturnValue(
        throwError(() => ({
          statusCode: 400,
          message: 'Insufficient inventory',
          error: 'INSUFFICIENT_INVENTORY',
          available: 10,
          requested: 100,
        })),
      );
      
      // Act & Assert
      await expect(service.createOrder(orderData)).rejects.toMatchObject({
        statusCode: 400,
        error: 'INSUFFICIENT_INVENTORY',
      });
    });
    
    it('should handle timeout', async () => {
      // Arrange
      const orderData = {
        userId: '123',
        productId: '456',
        quantity: 2,
      };
      
      // Mock delayed response (simulating timeout)
      mockUsersClient.send.mockReturnValue(
        of({ id: '123', name: 'John' }).pipe(delay(6000)),  // 6 second delay
      );
      
      // Act & Assert
      // If service has 5 second timeout, this should fail
      await expect(service.createOrder(orderData)).rejects.toThrow();
    });
  });
  
  describe('notifyUser', () => {
    it('should emit user notification', async () => {
      // Arrange
      const notification = {
        userId: '123',
        message: 'Order created',
      };
      
      mockUsersClient.emit.mockReturnValue(EMPTY);
      
      // Act
      await service.notifyUser(notification);
      
      // Assert
      expect(usersClient.emit).toHaveBeenCalledWith(
        'user.notification',
        notification,
      );
    });
  });
});
```

**Mock with Different Response Scenarios:**

```typescript
// ===== Testing Different Scenarios =====

describe('Different Response Scenarios', () => {
  let service: OrdersService;
  const mockClient = {
    send: jest.fn(),
    emit: jest.fn(),
    connect: jest.fn().mockResolvedValue(undefined),
    close: jest.fn().mockResolvedValue(undefined),
  };
  
  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        OrdersService,
        { provide: 'USERS_SERVICE', useValue: mockClient },
      ],
    }).compile();
    
    service = module.get<OrdersService>(OrdersService);
  });
  
  afterEach(() => {
    jest.clearAllMocks();
  });
  
  it('should handle successful response', async () => {
    // Return success data
    mockClient.send.mockReturnValue(
      of({ id: '123', name: 'John' }),
    );
    
    const result = await service.getUser('123');
    expect(result.name).toBe('John');
  });
  
  it('should handle error response', async () => {
    // Return error
    mockClient.send.mockReturnValue(
      throwError(() => new Error('Service unavailable')),
    );
    
    await expect(service.getUser('123')).rejects.toThrow('Service unavailable');
  });
  
  it('should handle empty response', async () => {
    // Return empty observable
    mockClient.send.mockReturnValue(EMPTY);
    
    // This will timeout or hang if not handled properly
  });
  
  it('should handle multiple values', async () => {
    // Return multiple values (streaming)
    import { from } from 'rxjs';
    
    mockClient.send.mockReturnValue(
      from([{ id: '1' }, { id: '2' }, { id: '3' }]),
    );
    
    const results = [];
    service.streamUsers().subscribe(user => results.push(user));
    
    expect(results).toHaveLength(3);
  });
  
  it('should handle delayed response', async () => {
    // Simulate network delay
    mockClient.send.mockReturnValue(
      of({ id: '123' }).pipe(delay(1000)),  // 1 second delay
    );
    
    const start = Date.now();
    await service.getUser('123');
    const duration = Date.now() - start;
    
    expect(duration).toBeGreaterThanOrEqual(1000);
  });
  
  it('should handle conditional responses', async () => {
    // Different responses based on input
    mockClient.send.mockImplementation((pattern, data) => {
      if (data === '123') {
        return of({ id: '123', name: 'John' });
      } else if (data === '456') {
        return of({ id: '456', name: 'Jane' });
      } else {
        return throwError(() => new Error('Not found'));
      }
    });
    
    const user1 = await service.getUser('123');
    expect(user1.name).toBe('John');
    
    const user2 = await service.getUser('456');
    expect(user2.name).toBe('Jane');
    
    await expect(service.getUser('999')).rejects.toThrow('Not found');
  });
});
```

**Mock Factory Pattern:**

```typescript
// ===== Reusable Mock Factory =====

// test/mocks/client-proxy.mock.ts
import { of, throwError, EMPTY } from 'rxjs';

export class MockClientProxy {
  send = jest.fn();
  emit = jest.fn().mockReturnValue(EMPTY);
  connect = jest.fn().mockResolvedValue(undefined);
  close = jest.fn().mockResolvedValue(undefined);
  
  // Helper methods
  mockSendSuccess(data: any) {
    this.send.mockReturnValue(of(data));
    return this;
  }
  
  mockSendError(error: any) {
    this.send.mockReturnValue(throwError(() => error));
    return this;
  }
  
  mockSendEmpty() {
    this.send.mockReturnValue(EMPTY);
    return this;
  }
  
  mockEmitSuccess() {
    this.emit.mockReturnValue(EMPTY);
    return this;
  }
  
  reset() {
    this.send.mockReset();
    this.emit.mockReset();
    this.connect.mockReset();
    this.close.mockReset();
  }
}

// Usage in tests
import { MockClientProxy } from '../test/mocks/client-proxy.mock';

describe('OrdersService', () => {
  let service: OrdersService;
  let mockUsersClient: MockClientProxy;
  
  beforeEach(async () => {
    mockUsersClient = new MockClientProxy();
    
    const module = await Test.createTestingModule({
      providers: [
        OrdersService,
        { provide: 'USERS_SERVICE', useValue: mockUsersClient },
      ],
    }).compile();
    
    service = module.get<OrdersService>(OrdersService);
  });
  
  afterEach(() => {
    mockUsersClient.reset();
  });
  
  it('should create order', async () => {
    // Use helper method
    mockUsersClient.mockSendSuccess({
      id: '123',
      name: 'John Doe',
    });
    
    const result = await service.createOrder({ userId: '123' });
    expect(result).toBeDefined();
  });
  
  it('should handle error', async () => {
    mockUsersClient.mockSendError({
      statusCode: 404,
      message: 'User not found',
    });
    
    await expect(service.createOrder({ userId: '999' })).rejects.toThrow();
  });
});
```

**Testing with Multiple Clients:**

```typescript
// ===== Multiple ClientProxy Mocks =====

describe('OrdersService with Multiple Clients', () => {
  let service: OrdersService;
  
  const mockClients = {
    users: new MockClientProxy(),
    inventory: new MockClientProxy(),
    payments: new MockClientProxy(),
    notifications: new MockClientProxy(),
  };
  
  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        OrdersService,
        { provide: 'USERS_SERVICE', useValue: mockClients.users },
        { provide: 'INVENTORY_SERVICE', useValue: mockClients.inventory },
        { provide: 'PAYMENTS_SERVICE', useValue: mockClients.payments },
        { provide: 'NOTIFICATIONS_SERVICE', useValue: mockClients.notifications },
      ],
    }).compile();
    
    service = module.get<OrdersService>(OrdersService);
  });
  
  afterEach(() => {
    Object.values(mockClients).forEach(mock => mock.reset());
  });
  
  it('should coordinate between multiple services', async () => {
    // Setup all mocks
    mockClients.users.mockSendSuccess({ id: '123', name: 'John' });
    mockClients.inventory.mockSendSuccess({ available: 10 });
    mockClients.payments.mockSendSuccess({ transactionId: 'txn_123' });
    mockClients.notifications.mockEmitSuccess();
    
    // Act
    const result = await service.createOrder({
      userId: '123',
      productId: '456',
      quantity: 2,
    });
    
    // Assert
    expect(result).toBeDefined();
    
    // Verify all services were called
    expect(mockClients.users.send).toHaveBeenCalled();
    expect(mockClients.inventory.send).toHaveBeenCalled();
    expect(mockClients.payments.send).toHaveBeenCalled();
    expect(mockClients.notifications.emit).toHaveBeenCalled();
  });
  
  it('should rollback on payment failure', async () => {
    // Setup: inventory succeeds, payment fails
    mockClients.users.mockSendSuccess({ id: '123', name: 'John' });
    mockClients.inventory.mockSendSuccess({ reservationId: 'res_123' });
    mockClients.payments.mockSendError({
      statusCode: 402,
      message: 'Payment declined',
    });
    
    // Act & Assert
    await expect(service.createOrder({ userId: '123' })).rejects.toThrow();
    
    // Verify rollback was called
    expect(mockClients.inventory.send).toHaveBeenCalledWith(
      { cmd: 'cancel_reservation' },
      expect.any(Object),
    );
  });
});
```

**Spy Pattern:**

```typescript
// ===== Using Jest Spy =====

describe('OrdersService with Spy', () => {
  let service: OrdersService;
  let usersClient: ClientProxy;
  
  const mockClient = {
    send: jest.fn(),
    emit: jest.fn(),
    connect: jest.fn().mockResolvedValue(undefined),
    close: jest.fn().mockResolvedValue(undefined),
  };
  
  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        OrdersService,
        { provide: 'USERS_SERVICE', useValue: mockClient },
      ],
    }).compile();
    
    service = module.get<OrdersService>(OrdersService);
    usersClient = module.get<ClientProxy>('USERS_SERVICE');
  });
  
  it('should verify call order', async () => {
    mockClient.send
      .mockReturnValueOnce(of({ id: '123', name: 'John' }))      // 1st call
      .mockReturnValueOnce(of({ id: '456', email: 'john@...' })) // 2nd call
      .mockReturnValueOnce(of({ id: '789', role: 'admin' }));    // 3rd call
    
    await service.getUserDetails('123');
    
    // Verify call order
    expect(mockClient.send).toHaveBeenNthCalledWith(
      1,
      { cmd: 'get_user' },
      '123',
    );
    expect(mockClient.send).toHaveBeenNthCalledWith(
      2,
      { cmd: 'get_email' },
      '123',
    );
    expect(mockClient.send).toHaveBeenNthCalledWith(
      3,
      { cmd: 'get_role' },
      '123',
    );
  });
  
  it('should verify send was called with correct arguments', async () => {
    mockClient.send.mockReturnValue(of({ id: '123' }));
    
    await service.getUser('123');
    
    // Exact match
    expect(mockClient.send).toHaveBeenCalledWith(
      { cmd: 'get_user' },
      '123',
    );
    
    // Partial match
    expect(mockClient.send).toHaveBeenCalledWith(
      expect.objectContaining({ cmd: 'get_user' }),
      expect.any(String),
    );
  });
});
```

**Best Practices:**

```typescript
// ✅ GOOD: Mock ClientProxy with all methods
const mockClient = {
  send: jest.fn(),
  emit: jest.fn(),
  connect: jest.fn().mockResolvedValue(undefined),
  close: jest.fn().mockResolvedValue(undefined),
};

// ✅ GOOD: Return Observable from send()
mockClient.send.mockReturnValue(of({ id: '123' }));

// ✅ GOOD: Return EMPTY from emit()
mockClient.emit.mockReturnValue(EMPTY);

// ✅ GOOD: Use throwError for errors
mockClient.send.mockReturnValue(
  throwError(() => new Error('Service error')),
);

// ✅ GOOD: Clear mocks between tests
afterEach(() => {
  jest.clearAllMocks();
});

// ✅ GOOD: Verify method calls
expect(mockClient.send).toHaveBeenCalledWith(
  { cmd: 'get_user' },
  '123',
);

// ✅ GOOD: Test both success and error cases
it('should handle success', ...);
it('should handle error', ...);

// ✅ GOOD: Use mock factory for reusability
const mockClient = new MockClientProxy();

// ❌ BAD: Not returning Observable
mockClient.send.mockReturnValue({ id: '123' });  // Wrong!

// ❌ BAD: Not mocking all methods
const mockClient = { send: jest.fn() };  // Missing emit, connect, close

// ❌ BAD: Not clearing mocks
// Mocks persist between tests

// ❌ BAD: Not verifying calls
// Can't verify correct communication
```

**Summary:**

```
Mocking ClientProxy:

1. Create Mock Object:
   const mockClient = {
     send: jest.fn(),
     emit: jest.fn(),
     connect: jest.fn().mockResolvedValue(undefined),
     close: jest.fn().mockResolvedValue(undefined),
   };

2. Provide in Test Module:
   {
     provide: 'USERS_SERVICE',  // Injection token
     useValue: mockClient,
   }

3. Mock send() Returns:
   // Success
   mockClient.send.mockReturnValue(of({ id: '123' }));
   
   // Error
   mockClient.send.mockReturnValue(
     throwError(() => new Error('Error')),
   );
   
   // Empty
   mockClient.send.mockReturnValue(EMPTY);

4. Mock emit() Returns:
   mockClient.emit.mockReturnValue(EMPTY);

5. Verify Calls:
   expect(mockClient.send).toHaveBeenCalledWith(
     { cmd: 'get_user' },
     '123',
   );

Key Points:
✅ Mock all methods (send, emit, connect, close)
✅ Return Observables (of, throwError, EMPTY)
✅ Clear mocks between tests
✅ Verify method calls
✅ Test success and error cases
✅ Use mock factory for reusability
✅ Mock conditionally with mockImplementation

RxJS Helpers:
• of() - Success value
• throwError() - Error
• EMPTY - Empty observable
• from() - Multiple values
• delay() - Delayed response

Best Practices:
✅ Reusable mock factory
✅ Helper methods
✅ Multiple scenarios
✅ Verify call order
✅ Test timeouts
✅ Isolated tests
```

**Interview Tip**: **Mock ClientProxy** by creating mock object with **send()** and **emit()** methods: `{ send: jest.fn(), emit: jest.fn(), connect: jest.fn(), close: jest.fn() }`. **Provide in test module** using injection token: `{ provide: 'USERS_SERVICE', useValue: mockClient }`. **send() returns Observable**: `mockClient.send.mockReturnValue(of(data))` for success, `throwError(() => error)` for errors, `EMPTY` for empty. **emit() returns EMPTY**: `mockClient.emit.mockReturnValue(EMPTY)`. **Verify calls**: `expect(mockClient.send).toHaveBeenCalledWith({ cmd: 'get_user' }, '123')`. **Best practices**: clear mocks between tests with `jest.clearAllMocks()`, test both success/error cases, use mock factory for reusability. **Key**: must return Observables (RxJS), not plain values.

</details>

## Best Practices

### 45. When should you use microservices vs monolith?

<details>
<summary>Answer</summary>

**Choosing between microservices and monolith** depends on **team size**, **project complexity**, **scalability needs**, **deployment requirements**, and **organizational maturity**. **Start with monolith** for small teams/simple projects, **migrate to microservices** when you have clear service boundaries, scaling needs, and team capacity to handle distributed complexity.

**Decision Framework:**

```typescript
// ===== When to Use MONOLITH =====

✅ Use Monolith When:

1. Small Team (< 10 developers)
   - Easier coordination
   - Simpler development workflow
   - Lower overhead

2. Simple/New Project
   - Unclear domain boundaries
   - Rapidly changing requirements
   - MVP/prototype phase

3. Limited Resources
   - Small infrastructure budget
   - Limited DevOps expertise
   - Fewer deployment environments

4. Tight Coupling Acceptable
   - Shared database is fine
   - Synchronous communication preferred
   - ACID transactions needed

5. Simple Deployment
   - Single deployment unit
   - No complex orchestration
   - Easier rollbacks

// ===== When to Use MICROSERVICES =====

✅ Use Microservices When:

1. Large Team (> 20 developers)
   - Multiple teams
   - Independent development
   - Parallel work streams

2. Clear Domain Boundaries
   - Well-defined services
   - Bounded contexts (DDD)
   - Stable business domains

3. Scalability Requirements
   - Different scaling needs per service
   - High traffic/load
   - Geographic distribution

4. Technology Diversity
   - Different languages/frameworks
   - Polyglot persistence
   - Specialized tools per service

5. Independent Deployment
   - Frequent deployments
   - Zero-downtime updates
   - Canary/blue-green deployments

6. Organizational Maturity
   - Strong DevOps culture
   - Automated CI/CD
   - Monitoring/observability tools
   - Experienced with distributed systems
```

**Comparison Table:**

```typescript
// ===== Monolith vs Microservices =====

| Aspect              | Monolith                    | Microservices                 |
|---------------------|----------------------------|-------------------------------|
| **Architecture**    | Single codebase            | Multiple services             |
| **Deployment**      | Deploy entire app          | Deploy services independently |
| **Scaling**         | Scale entire app           | Scale services independently  |
| **Development**     | Single repository          | Multiple repositories         |
| **Team Structure**  | Single team                | Multiple teams                |
| **Communication**   | In-process (fast)          | Network calls (slower)        |
| **Data**            | Single database            | Database per service          |
| **Transactions**    | ACID (simple)              | Eventual consistency (complex)|
| **Testing**         | Easier                     | More complex                  |
| **Debugging**       | Easier                     | Distributed tracing needed    |
| **Initial Cost**    | Lower                      | Higher                        |
| **Maintenance**     | Simpler (initially)        | Complex (long-term)           |
| **Technology**      | Single stack               | Polyglot                      |
| **Onboarding**      | Easier                     | More complex                  |
| **Failures**        | App-wide failure           | Isolated failures             |
```

**Practical Examples:**

**1. Start with Monolith:**

```typescript
// ===== E-commerce Monolith (Good for MVP) =====

// Single NestJS application
import { Module } from '@nestjs/common';
import { UsersModule } from './users/users.module';
import { ProductsModule } from './products/products.module';
import { OrdersModule } from './orders/orders.module';
import { PaymentsModule } from './payments/payments.module';

@Module({
  imports: [
    UsersModule,
    ProductsModule,
    OrdersModule,
    PaymentsModule,
  ],
})
export class AppModule {}

// All modules in same application
// Shared database
// Simple deployment
// Fast development

// Good for:
// - MVP phase
// - Small team (2-5 developers)
// - Testing product-market fit
// - Simple scaling (vertical)

// OrdersService in monolith
@Injectable()
export class OrdersService {
  constructor(
    private usersService: UsersService,      // Direct injection
    private productsService: ProductsService, // In-process call
    private paymentsService: PaymentsService, // No network latency
  ) {}
  
  async createOrder(orderData: CreateOrderDto) {
    // All services in same process
    const user = await this.usersService.findOne(orderData.userId);
    const product = await this.productsService.findOne(orderData.productId);
    
    // ACID transaction (simple)
    return await this.dataSource.transaction(async (manager) => {
      const order = await manager.save(Order, orderData);
      await this.paymentsService.charge(order);
      await this.productsService.decrementStock(product.id);
      return order;
    });
  }
}
```

**2. Migrate to Microservices:**

```typescript
// ===== E-commerce Microservices (Scale Phase) =====

// Separate services
// 1. Users Service (port 3001)
// 2. Products Service (port 3002)
// 3. Orders Service (port 3003)
// 4. Payments Service (port 3004)

// When to migrate:
// - Team grew to 15+ developers
// - Products service needs independent scaling
// - Different teams own different domains
// - Frequent deployments needed

// OrdersService in microservices
@Injectable()
export class OrdersService {
  constructor(
    @Inject('USERS_SERVICE') private usersClient: ClientProxy,
    @Inject('PRODUCTS_SERVICE') private productsClient: ClientProxy,
    @Inject('PAYMENTS_SERVICE') private paymentsClient: ClientProxy,
  ) {}
  
  async createOrder(orderData: CreateOrderDto) {
    // Network calls to other services
    const user = await firstValueFrom(
      this.usersClient.send({ cmd: 'get_user' }, orderData.userId),
    );
    
    const product = await firstValueFrom(
      this.productsClient.send({ cmd: 'get_product' }, orderData.productId),
    );
    
    // Saga pattern for distributed transaction
    const saga = new OrderSaga();
    
    try {
      const order = await this.ordersRepository.save(orderData);
      saga.addStep('order_created', order.id);
      
      const payment = await firstValueFrom(
        this.paymentsClient.send({ cmd: 'charge' }, order),
      );
      saga.addStep('payment_charged', payment.id);
      
      await firstValueFrom(
        this.productsClient.send({ cmd: 'decrement_stock' }, product.id),
      );
      saga.addStep('stock_decremented', product.id);
      
      return order;
    } catch (error) {
      // Compensating transactions
      await saga.rollback();
      throw error;
    }
  }
}
```

**Decision Tree:**

```typescript
// ===== Decision Tree =====

function shouldUseMicroservices() {
  const questions = [
    {
      question: 'Team size > 20 developers?',
      weight: 3,
    },
    {
      question: 'Clear service boundaries defined?',
      weight: 4,
    },
    {
      question: 'Need independent scaling?',
      weight: 3,
    },
    {
      question: 'Have DevOps/infrastructure expertise?',
      weight: 4,
    },
    {
      question: 'Need polyglot architecture?',
      weight: 2,
    },
    {
      question: 'Frequent independent deployments needed?',
      weight: 3,
    },
    {
      question: 'Have monitoring/observability tools?',
      weight: 3,
    },
    {
      question: 'Can handle eventual consistency?',
      weight: 4,
    },
  ];
  
  // If score > 15, consider microservices
  // If score < 10, stick with monolith
  // If score 10-15, hybrid approach
}

// Hybrid Approach: Modular Monolith
// - Single deployment
// - Well-defined module boundaries
// - Easy to extract services later
// - Best of both worlds initially
```

**Migration Strategy:**

```typescript
// ===== Strangler Fig Pattern: Gradual Migration =====

// Phase 1: Modular Monolith
@Module({
  imports: [
    UsersModule,      // Well-defined boundaries
    ProductsModule,   // Clear interfaces
    OrdersModule,     // Loose coupling
    PaymentsModule,   // Separate databases (logical)
  ],
})
export class AppModule {}

// Phase 2: Extract First Service (e.g., Payments)
// - External payment processing
// - High security requirements
// - Independent scaling needs

// Monolith routes to microservice
@Injectable()
export class PaymentsService {
  constructor(
    @Inject('PAYMENTS_MICROSERVICE') private paymentsClient: ClientProxy,
  ) {}
  
  async charge(order: Order) {
    // Route to microservice
    return await firstValueFrom(
      this.paymentsClient.send({ cmd: 'charge' }, order),
    );
  }
}

// Phase 3: Extract More Services
// - Extract based on business needs
// - Gradual migration (low risk)
// - Monolith shrinks over time

// Phase 4: Full Microservices
// - All services extracted
// - API gateway for routing
// - Service mesh for communication
```

**Real-World Scenarios:**

```typescript
// ===== Scenario 1: Startup (Use Monolith) =====

Context:
- Team: 3 developers
- Product: MVP for new SaaS
- Timeline: 3 months to launch
- Budget: Limited
- Users: Unknown

Decision: MONOLITH

Reasons:
- Fast development
- Simple deployment
- Easy to refactor
- Lower infrastructure cost
- Team can manage entire codebase

// ===== Scenario 2: E-commerce Platform (Use Microservices) =====

Context:
- Team: 50 developers across 5 teams
- Product: Mature e-commerce platform
- Timeline: Continuous development
- Budget: Well-funded
- Users: Millions
- Traffic: Variable (spikes during sales)

Decision: MICROSERVICES

Reasons:
- Multiple teams need independence
- Different scaling needs (catalog vs checkout)
- Frequent deployments without downtime
- Technology diversity (ML for recommendations)
- High availability requirements

Services:
1. User Service (authentication, profiles)
2. Catalog Service (products, search) - Scale for read-heavy
3. Cart Service (shopping cart) - Stateful, session-based
4. Order Service (order processing)
5. Payment Service (PCI compliance, isolated)
6. Inventory Service (stock management)
7. Shipping Service (logistics integration)
8. Notification Service (emails, SMS)
9. Analytics Service (real-time reporting)
10. Recommendation Service (ML-based, Python)

// ===== Scenario 3: Corporate Dashboard (Hybrid) =====

Context:
- Team: 10 developers
- Product: Internal business dashboard
- Timeline: Ongoing
- Budget: Moderate
- Users: 1000 employees

Decision: MODULAR MONOLITH

Reasons:
- Team manageable size
- Internal users (predictable load)
- Can maintain quality with monolith
- Well-defined modules for future extraction
- Lower operational complexity

Structure:
// Modular monolith with clear boundaries
@Module({ ... })
export class ReportsModule {}  // Could become microservice

@Module({ ... })
export class AnalyticsModule {}  // Could become microservice

@Module({ ... })
export class UsersModule {}  // Keep in monolith
```

**Common Mistakes:**

```typescript
// ❌ MISTAKE 1: Premature Microservices

// Startup with 3 developers building microservices
// - Massive overhead
// - Slow development
// - Complex deployment
// - Debugging nightmare
// - High infrastructure cost

Solution: Start with monolith, extract later

// ❌ MISTAKE 2: Never Migrating

// Large team (50 devs) stuck in monolith
// - Deployment bottlenecks
// - Merge conflicts
// - Coupling increases
// - Hard to scale
// - Slow CI/CD

Solution: Gradually extract services (Strangler Fig)

// ❌ MISTAKE 3: Too Many Microservices

// 50 microservices for small application
// - Distributed monolith
// - Network overhead
// - Deployment complexity
// - Difficult to maintain

Solution: Coarser-grained services, merge related services

// ❌ MISTAKE 4: Shared Database

// Microservices sharing same database
// - Tight coupling
// - No independent deployment
// - Schema changes affect all
// - Defeats purpose

Solution: Database per service (even if in same DB cluster)

// ❌ MISTAKE 5: Ignoring Complexity

// Moving to microservices without:
// - DevOps expertise
// - Monitoring tools
// - CI/CD automation
// - Distributed tracing

Solution: Build capabilities before migrating
```

**Best Practices:**

```typescript
// ✅ GOOD: Start simple, evolve

// 1. Start with monolith
// 2. Build with clear module boundaries
// 3. Identify bottlenecks/scaling needs
// 4. Extract services gradually
// 5. Mature into microservices

// ✅ GOOD: Modular monolith

@Module({
  imports: [],
  exports: [UsersService],  // Clear interface
})
export class UsersModule {
  // Easy to extract later
}

// ✅ GOOD: Start with 2-3 services max

// Don't start with 20 services
// Start with:
// 1. Core API (monolith)
// 2. Background Jobs (separate service)
// 3. Maybe: Authentication (separate for security)

// ✅ GOOD: Clear criteria for extraction

Extract when:
- Service has different scaling needs
- Different technology requirements
- Owned by separate team
- Security isolation needed
- Independent release cycle needed

// ✅ GOOD: Infrastructure automation

Before microservices, have:
- CI/CD pipelines
- Infrastructure as Code
- Container orchestration (K8s)
- Service mesh (Istio/Linkerd)
- Monitoring (Prometheus, Grafana)
- Distributed tracing (Jaeger)
- Centralized logging (ELK stack)
```

**Summary:**

```
When to Use Monolith:
✅ Small team (< 10 devs)
✅ New/simple project
✅ Unclear domain boundaries
✅ MVP/prototype phase
✅ Limited resources
✅ Simple scaling acceptable
✅ Fast time to market
✅ ACID transactions needed

When to Use Microservices:
✅ Large team (> 20 devs)
✅ Clear domain boundaries
✅ Independent scaling needs
✅ Technology diversity
✅ Frequent deployments
✅ Organizational maturity
✅ DevOps expertise
✅ High availability requirements

Hybrid: Modular Monolith
✅ Medium team (10-20 devs)
✅ Well-defined modules
✅ Clear boundaries
✅ Easy future extraction
✅ Best of both worlds

Migration Strategy:
1. Start with monolith
2. Build modular architecture
3. Extract services gradually (Strangler Fig)
4. Based on business needs
5. When team/infrastructure ready

Key Factors:
• Team size and structure
• Domain complexity
• Scaling requirements
• Deployment frequency
• Technology needs
• Infrastructure maturity
• Operational expertise
• Budget and resources

Common Mistakes:
❌ Premature microservices
❌ Never migrating from monolith
❌ Too many microservices
❌ Shared database
❌ Ignoring operational complexity

Rule of Thumb:
"Start with a monolith, migrate to microservices when pain points emerge"
```

**Interview Tip**: **Use monolith** for: **small teams** (< 10 devs), **simple/new projects**, **MVP phase**, **unclear boundaries**, **limited resources**. **Use microservices** for: **large teams** (> 20 devs), **clear domain boundaries**, **independent scaling needs**, **technology diversity**, **frequent deployments**, **DevOps maturity**. **Best strategy**: **start with modular monolith**, extract services gradually when needed (Strangler Fig pattern). **Key factors**: team size, domain complexity, scaling requirements, infrastructure maturity. **Common mistakes**: premature microservices (overhead kills small teams), never migrating (large teams bottlenecked), too many services (distributed monolith), shared database (defeats purpose). **Hybrid approach**: modular monolith with clear boundaries, easy to extract later. **Decision**: evaluate team capacity to handle distributed systems complexity before migrating.

</details>

### ### 46. How do you handle distributed transactions?

<details>
<summary>Answer</summary>

**Distributed transactions** are transactions that span **multiple microservices** with **separate databases**. Since traditional **ACID transactions** don't work across services, use **Saga pattern** (orchestration or choreography), **two-phase commit** (2PC), or **eventual consistency** with **compensating transactions** to maintain data consistency.

**Why Distributed Transactions are Hard:**

```typescript
// ===== Problem: Can't Use Traditional ACID =====

// In Monolith (Easy):
@Injectable()
export class OrdersService {
  async createOrder(orderData: CreateOrderDto) {
    // Single database, ACID transaction
    return await this.dataSource.transaction(async (manager) => {
      const order = await manager.save(Order, orderData);
      await manager.save(Payment, { orderId: order.id });
      await manager.update(Product, { id: productId }, { stock: stock - 1 });
      // All or nothing - automatic rollback on error
      return order;
    });
  }
}

// In Microservices (Hard):
@Injectable()
export class OrdersService {
  async createOrder(orderData: CreateOrderDto) {
    // Three separate services, three separate databases
    // Can't use single transaction!
    
    // 1. Create order (Orders DB)
    const order = await this.ordersRepo.save(orderData);
    
    // 2. Charge payment (Payments DB) - What if this fails?
    const payment = await firstValueFrom(
      this.paymentsClient.send({ cmd: 'charge' }, order),
    );
    
    // 3. Decrement stock (Products DB) - What if this fails?
    await firstValueFrom(
      this.productsClient.send({ cmd: 'decrement_stock' }, productId),
    );
    
    // Problem: If step 2 or 3 fails, step 1 already committed!
    // Need distributed transaction pattern
  }
}

// Challenges:
// ❌ No atomic commit across services
// ❌ Partial failures (some succeed, some fail)
// ❌ Network failures
// ❌ Service unavailability
// ❌ Data inconsistency
// ❌ Rollback complexity
```

**Solution 1: Saga Pattern (Orchestration):**

```typescript
// ===== Saga Pattern: Orchestration-Based =====

// Central orchestrator coordinates transaction steps

// Saga orchestrator
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';

interface SagaStep {
  serviceName: string;
  action: string;
  data: any;
  compensate: string;  // Rollback action
  status: 'pending' | 'completed' | 'failed' | 'compensated';
}

@Injectable()
export class OrderSagaOrchestrator {
  constructor(
    @Inject('PAYMENTS_SERVICE') private paymentsClient: ClientProxy,
    @Inject('INVENTORY_SERVICE') private inventoryClient: ClientProxy,
    @Inject('SHIPPING_SERVICE') private shippingClient: ClientProxy,
    @InjectRepository(Order) private ordersRepo: Repository<Order>,
    @InjectRepository(SagaState) private sagaStateRepo: Repository<SagaState>,
  ) {}
  
  async createOrder(orderData: CreateOrderDto) {
    const sagaId = uuidv4();
    const steps: SagaStep[] = [
      {
        serviceName: 'orders',
        action: 'create_order',
        data: orderData,
        compensate: 'cancel_order',
        status: 'pending',
      },
      {
        serviceName: 'payments',
        action: 'charge',
        data: null,  // Will be filled with order ID
        compensate: 'refund',
        status: 'pending',
      },
      {
        serviceName: 'inventory',
        action: 'reserve_stock',
        data: { productId: orderData.productId, quantity: orderData.quantity },
        compensate: 'release_stock',
        status: 'pending',
      },
      {
        serviceName: 'shipping',
        action: 'schedule_shipping',
        data: null,  // Will be filled with order ID
        compensate: 'cancel_shipping',
        status: 'pending',
      },
    ];
    
    // Save saga state
    await this.sagaStateRepo.save({
      sagaId,
      steps: JSON.stringify(steps),
      status: 'in_progress',
    });
    
    try {
      // Execute steps sequentially
      let order: Order;
      
      // Step 1: Create order
      console.log('Step 1: Creating order');
      order = await this.ordersRepo.save(orderData);
      steps[0].status = 'completed';
      await this.updateSagaState(sagaId, steps);
      
      // Step 2: Charge payment
      console.log('Step 2: Charging payment');
      steps[1].data = { orderId: order.id, amount: order.total };
      const payment = await firstValueFrom(
        this.paymentsClient.send({ cmd: 'charge' }, steps[1].data).pipe(
          timeout(10000),
        ),
      );
      steps[1].status = 'completed';
      await this.updateSagaState(sagaId, steps);
      
      // Step 3: Reserve inventory
      console.log('Step 3: Reserving inventory');
      await firstValueFrom(
        this.inventoryClient.send({ cmd: 'reserve_stock' }, steps[2].data).pipe(
          timeout(10000),
        ),
      );
      steps[2].status = 'completed';
      await this.updateSagaState(sagaId, steps);
      
      // Step 4: Schedule shipping
      console.log('Step 4: Scheduling shipping');
      steps[3].data = { orderId: order.id, address: orderData.shippingAddress };
      await firstValueFrom(
        this.shippingClient.send({ cmd: 'schedule_shipping' }, steps[3].data).pipe(
          timeout(10000),
        ),
      );
      steps[3].status = 'completed';
      await this.updateSagaState(sagaId, steps, 'completed');
      
      console.log('Saga completed successfully');
      return order;
      
    } catch (error) {
      console.error('Saga failed, executing compensating transactions:', error);
      
      // Rollback: Execute compensating transactions in reverse order
      await this.compensate(sagaId, steps);
      
      throw new Error('Order creation failed: ' + error.message);
    }
  }
  
  private async compensate(sagaId: string, steps: SagaStep[]) {
    // Execute compensating transactions in reverse order
    for (let i = steps.length - 1; i >= 0; i--) {
      const step = steps[i];
      
      if (step.status === 'completed') {
        try {
          console.log(`Compensating step ${i + 1}: ${step.compensate}`);
          
          switch (step.serviceName) {
            case 'orders':
              await this.ordersRepo.update({ id: step.data.id }, { status: 'cancelled' });
              break;
            
            case 'payments':
              await firstValueFrom(
                this.paymentsClient.send({ cmd: 'refund' }, step.data),
              );
              break;
            
            case 'inventory':
              await firstValueFrom(
                this.inventoryClient.send({ cmd: 'release_stock' }, step.data),
              );
              break;
            
            case 'shipping':
              await firstValueFrom(
                this.shippingClient.send({ cmd: 'cancel_shipping' }, step.data),
              );
              break;
          }
          
          step.status = 'compensated';
          await this.updateSagaState(sagaId, steps);
          
        } catch (error) {
          console.error(`Failed to compensate step ${i + 1}:`, error);
          // Log for manual intervention
          await this.logCompensationFailure(sagaId, step, error);
        }
      }
    }
    
    await this.updateSagaState(sagaId, steps, 'compensated');
  }
  
  private async updateSagaState(
    sagaId: string,
    steps: SagaStep[],
    status?: string,
  ) {
    await this.sagaStateRepo.update(
      { sagaId },
      {
        steps: JSON.stringify(steps),
        status: status || 'in_progress',
        updatedAt: new Date(),
      },
    );
  }
  
  private async logCompensationFailure(
    sagaId: string,
    step: SagaStep,
    error: any,
  ) {
    // Log for manual intervention
    await this.sagaStateRepo.save({
      sagaId,
      failedCompensation: JSON.stringify({ step, error: error.message }),
      needsManualIntervention: true,
    });
  }
}

// Saga state entity
@Entity()
export class SagaState {
  @PrimaryGeneratedColumn('uuid')
  sagaId: string;
  
  @Column('text')
  steps: string;
  
  @Column()
  status: string;
  
  @Column({ type: 'text', nullable: true })
  failedCompensation: string;
  
  @Column({ default: false })
  needsManualIntervention: boolean;
  
  @CreateDateColumn()
  createdAt: Date;
  
  @UpdateDateColumn()
  updatedAt: Date;
}
```

**Solution 2: Saga Pattern (Choreography):**

```typescript
// ===== Saga Pattern: Choreography-Based =====

// Services react to events (no central orchestrator)

// Orders Service
@Controller()
export class OrdersController {
  constructor(
    private ordersRepo: Repository<Order>,
    @Inject('EVENT_BUS') private eventBus: ClientProxy,
  ) {}
  
  @EventPattern('order.create_requested')
  async handleCreateOrder(data: CreateOrderDto) {
    try {
      // Create order
      const order = await this.ordersRepo.save({
        ...data,
        status: 'pending',
      });
      
      // Emit event for next service
      this.eventBus.emit('order.created', {
        orderId: order.id,
        userId: order.userId,
        amount: order.total,
        productId: order.productId,
        quantity: order.quantity,
      });
      
    } catch (error) {
      console.error('Failed to create order:', error);
      this.eventBus.emit('order.creation_failed', { error: error.message });
    }
  }
  
  @EventPattern('payment.charged')
  async handlePaymentCharged(data: any) {
    // Update order status
    await this.ordersRepo.update({ id: data.orderId }, { status: 'paid' });
    console.log('Order marked as paid');
  }
  
  @EventPattern('payment.failed')
  async handlePaymentFailed(data: any) {
    // Compensate: Cancel order
    await this.ordersRepo.update(
      { id: data.orderId },
      { status: 'cancelled', cancelReason: 'Payment failed' },
    );
    console.log('Order cancelled due to payment failure');
  }
  
  @EventPattern('inventory.reserved')
  async handleInventoryReserved(data: any) {
    await this.ordersRepo.update({ id: data.orderId }, { status: 'confirmed' });
    console.log('Order confirmed');
  }
  
  @EventPattern('inventory.reservation_failed')
  async handleInventoryFailed(data: any) {
    // Compensate: Cancel order and refund
    await this.ordersRepo.update(
      { id: data.orderId },
      { status: 'cancelled', cancelReason: 'Insufficient inventory' },
    );
    
    // Emit refund event
    this.eventBus.emit('refund.requested', {
      orderId: data.orderId,
      paymentId: data.paymentId,
    });
  }
}

// Payments Service
@Controller()
export class PaymentsController {
  @EventPattern('order.created')
  async handleOrderCreated(data: any) {
    try {
      // Charge payment
      const payment = await this.chargeCard({
        userId: data.userId,
        amount: data.amount,
        orderId: data.orderId,
      });
      
      // Emit success event
      this.eventBus.emit('payment.charged', {
        orderId: data.orderId,
        paymentId: payment.id,
        amount: payment.amount,
      });
      
    } catch (error) {
      console.error('Payment failed:', error);
      
      // Emit failure event
      this.eventBus.emit('payment.failed', {
        orderId: data.orderId,
        error: error.message,
      });
    }
  }
  
  @EventPattern('refund.requested')
  async handleRefundRequested(data: any) {
    try {
      await this.refundPayment(data.paymentId);
      this.eventBus.emit('refund.completed', data);
    } catch (error) {
      console.error('Refund failed:', error);
      this.eventBus.emit('refund.failed', { ...data, error: error.message });
    }
  }
}

// Inventory Service
@Controller()
export class InventoryController {
  @EventPattern('payment.charged')
  async handlePaymentCharged(data: any) {
    try {
      // Reserve stock
      await this.inventoryRepo.update(
        { productId: data.productId },
        { stock: () => 'stock - ' + data.quantity },
      );
      
      // Emit success event
      this.eventBus.emit('inventory.reserved', {
        orderId: data.orderId,
        productId: data.productId,
        quantity: data.quantity,
      });
      
    } catch (error) {
      console.error('Inventory reservation failed:', error);
      
      // Emit failure event (triggers compensation)
      this.eventBus.emit('inventory.reservation_failed', {
        orderId: data.orderId,
        paymentId: data.paymentId,
        error: error.message,
      });
    }
  }
  
  @EventPattern('order.cancelled')
  async handleOrderCancelled(data: any) {
    // Compensate: Release reserved stock
    await this.inventoryRepo.update(
      { productId: data.productId },
      { stock: () => 'stock + ' + data.quantity },
    );
    console.log('Stock released');
  }
}
```

**Solution 3: Two-Phase Commit (2PC):**

```typescript
// ===== Two-Phase Commit (Rarely Used in Microservices) =====

// Phase 1: Prepare (Ask all services if they can commit)
// Phase 2: Commit or Abort (All commit or all abort)

// Coordinator
@Injectable()
export class TwoPhaseCommitCoordinator {
  async executeTransaction(transactionId: string, operations: Operation[]) {
    const participants: Participant[] = [];
    
    try {
      // PHASE 1: PREPARE
      console.log('Phase 1: Prepare');
      
      for (const operation of operations) {
        const response = await firstValueFrom(
          operation.client.send(
            { cmd: 'prepare' },
            { transactionId, data: operation.data },
          ),
        );
        
        participants.push({
          service: operation.serviceName,
          prepared: response.canCommit,
          client: operation.client,
        });
        
        if (!response.canCommit) {
          throw new Error(`${operation.serviceName} cannot commit`);
        }
      }
      
      // All prepared, proceed to commit
      console.log('All services prepared, committing...');
      
      // PHASE 2: COMMIT
      for (const participant of participants) {
        await firstValueFrom(
          participant.client.send(
            { cmd: 'commit' },
            { transactionId },
          ),
        );
      }
      
      console.log('Transaction committed successfully');
      return { success: true };
      
    } catch (error) {
      // PHASE 2: ABORT
      console.error('Transaction failed, aborting:', error);
      
      for (const participant of participants) {
        if (participant.prepared) {
          await firstValueFrom(
            participant.client.send(
              { cmd: 'abort' },
              { transactionId },
            ),
          );
        }
      }
      
      throw error;
    }
  }
}

// Participant Service
@Controller()
export class PaymentsController {
  private pendingTransactions = new Map<string, any>();
  
  @MessagePattern({ cmd: 'prepare' })
  async prepare(data: { transactionId: string; data: any }) {
    try {
      // Check if can commit (e.g., sufficient balance)
      const canCommit = await this.validatePayment(data.data);
      
      if (canCommit) {
        // Lock resources (don't commit yet)
        this.pendingTransactions.set(data.transactionId, {
          data: data.data,
          lockedAt: new Date(),
        });
      }
      
      return { canCommit };
    } catch (error) {
      return { canCommit: false, error: error.message };
    }
  }
  
  @MessagePattern({ cmd: 'commit' })
  async commit(data: { transactionId: string }) {
    const pending = this.pendingTransactions.get(data.transactionId);
    
    if (pending) {
      // Actually commit the transaction
      await this.chargePayment(pending.data);
      this.pendingTransactions.delete(data.transactionId);
      return { success: true };
    }
    
    return { success: false, error: 'Transaction not found' };
  }
  
  @MessagePattern({ cmd: 'abort' })
  async abort(data: { transactionId: string }) {
    // Release locks
    this.pendingTransactions.delete(data.transactionId);
    return { success: true };
  }
}

// Problems with 2PC:
// ❌ Blocking protocol (services wait for coordinator)
// ❌ Single point of failure (coordinator)
// ❌ Poor performance (multiple round trips)
// ❌ Timeout issues
// ❌ Not recommended for microservices
```

**Solution 4: Eventual Consistency:**

```typescript
// ===== Eventual Consistency with Event Sourcing =====

// Accept temporary inconsistency, eventually converge to consistent state

@Injectable()
export class OrdersService {
  async createOrder(orderData: CreateOrderDto) {
    // 1. Create order immediately (optimistic)
    const order = await this.ordersRepo.save({
      ...orderData,
      status: 'pending_payment',
    });
    
    // 2. Emit event for async processing
    this.eventBus.emit('order.created', {
      orderId: order.id,
      ...orderData,
    });
    
    // 3. Return immediately (don't wait for payment/inventory)
    return {
      ...order,
      message: 'Order created, processing payment...',
    };
  }
  
  // Later, when payment succeeds
  @EventPattern('payment.succeeded')
  async handlePaymentSucceeded(data: any) {
    await this.ordersRepo.update(
      { id: data.orderId },
      { status: 'paid' },
    );
  }
  
  // If payment fails
  @EventPattern('payment.failed')
  async handlePaymentFailed(data: any) {
    await this.ordersRepo.update(
      { id: data.orderId },
      { status: 'payment_failed' },
    );
    
    // Notify user
    this.notificationClient.emit('send_email', {
      to: data.userEmail,
      subject: 'Payment Failed',
      body: 'Your order payment failed. Please try again.',
    });
  }
}

// User sees:
// 1. Order created ✓
// 2. Processing payment... (eventual)
// 3. Payment confirmed ✓ (or failed ❌)
```

**Best Practices:**

```typescript
// ✅ GOOD: Use Saga pattern for complex workflows
const saga = new OrderSagaOrchestrator();
await saga.execute(orderData);

// ✅ GOOD: Idempotency (handle duplicate messages)
@MessagePattern({ cmd: 'charge' })
async charge(data: { orderId: string; idempotencyKey: string }) {
  // Check if already processed
  const existing = await this.paymentsRepo.findOne({
    where: { idempotencyKey: data.idempotencyKey },
  });
  
  if (existing) {
    return existing;  // Already processed
  }
  
  // Process payment
  return await this.processPayment(data);
}

// ✅ GOOD: Timeout handling
await firstValueFrom(
  this.client.send({ cmd: 'charge' }, data).pipe(
    timeout(10000),  // 10 second timeout
  ),
);

// ✅ GOOD: Store saga state for recovery
await this.sagaStateRepo.save({ sagaId, steps, status });

// ✅ GOOD: Manual intervention for failed compensations
if (compensationFailed) {
  await this.alertOps({ sagaId, step, error });
}

// ✅ GOOD: Event versioning
interface OrderCreatedEvent_V1 {
  version: 1;
  orderId: string;
  userId: string;
}

interface OrderCreatedEvent_V2 {
  version: 2;
  orderId: string;
  userId: string;
  total: number;  // New field
}

// ❌ BAD: Assuming all steps succeed
// No compensation logic

// ❌ BAD: Not handling partial failures
// Leaving data inconsistent

// ❌ BAD: Using 2PC in microservices
// Performance bottleneck

// ❌ BAD: Synchronous calls without timeout
// Can hang indefinitely
```

**Summary:**

```
Distributed Transaction Patterns:

1. Saga Pattern (Orchestration):
   • Central orchestrator coordinates steps
   • Executes compensating transactions on failure
   • Best for: Complex workflows, clear coordination
   • Pros: Central control, easier to debug
   • Cons: Single point of failure, orchestrator complexity

2. Saga Pattern (Choreography):
   • Services react to events
   • No central coordinator
   • Best for: Simple workflows, loose coupling
   • Pros: Decentralized, scalable
   • Cons: Hard to track, complex debugging

3. Two-Phase Commit (2PC):
   • Prepare → Commit/Abort
   • Best for: Strong consistency required (rare)
   • Pros: ACID-like guarantees
   • Cons: Blocking, poor performance, not recommended

4. Eventual Consistency:
   • Accept temporary inconsistency
   • Eventually converge to consistent state
   • Best for: High performance, availability
   • Pros: Fast, scalable
   • Cons: Complex for users to understand

Key Concepts:
• Compensating transactions (rollback)
• Idempotency (handle duplicates)
• Saga state persistence
• Timeout handling
• Manual intervention (for failures)
• Event versioning

Best Practices:
✅ Use Saga pattern (orchestration for complex, choreography for simple)
✅ Implement idempotency
✅ Handle timeouts
✅ Store saga state
✅ Plan compensating transactions
✅ Monitor saga execution
✅ Alert on compensation failures
✅ Use eventual consistency when possible

Common Mistakes:
❌ Not handling partial failures
❌ No compensation logic
❌ Using 2PC (performance issues)
❌ Synchronous calls without timeout
❌ Not storing saga state
❌ Assuming all steps succeed
```

**Interview Tip**: **Handle distributed transactions** with **Saga pattern**: **Orchestration** (central coordinator executes steps sequentially, compensates on failure) or **Choreography** (services react to events, no coordinator). **Key components**: saga state persistence, compensating transactions (rollback), idempotency keys, timeout handling. **Two-phase commit** (2PC) rarely used (blocking, poor performance). **Eventual consistency** often preferred (fast, scalable, accept temporary inconsistency). **Best pattern**: orchestration for complex workflows with clear compensation logic, choreography for simple event-driven flows. **Critical**: store saga state for recovery, implement idempotent operations, handle partial failures with compensation, monitor saga execution. **Different from monolith**: can't use ACID transactions across services, need explicit coordination and rollback strategies.

</details>

47. What is eventual consistency?

<details>
<summary><strong>Answer</strong></summary>

**Eventual consistency** is a consistency model in distributed systems where updates to a system will eventually propagate to all nodes, but there may be a temporary period where different nodes have different values. Unlike strong consistency (immediate consistency across all nodes), eventual consistency accepts temporary inconsistencies in favor of higher availability and performance.

### **Core Concept**

```typescript
// STRONG CONSISTENCY (ACID - Monolith)
// All reads immediately reflect the latest write
@Injectable()
export class MonolithOrderService {
  async createOrder(orderData: CreateOrderDto) {
    // Single database transaction - all or nothing
    await this.db.transaction(async (trx) => {
      const order = await trx.orders.create(orderData); // Write
      const inventory = await trx.inventory.decrement(orderData.productId); // Write
      const payment = await trx.payments.create({ orderId: order.id }); // Write
      return order; // All writes committed together
    });
    
    // Immediate read reflects all writes
    const order = await this.db.orders.findById(orderId); // Always consistent
  }
}

// EVENTUAL CONSISTENCY (Microservices)
// Reads may not immediately reflect the latest write
@Injectable()
export class OrderMicroservice {
  constructor(
    @Inject('INVENTORY_SERVICE') private inventoryClient: ClientProxy,
    @Inject('PAYMENT_SERVICE') private paymentClient: ClientProxy,
  ) {}

  async createOrder(orderData: CreateOrderDto) {
    // 1. Create order in Orders DB
    const order = await this.orderRepository.save(orderData);
    
    // 2. Send async events to other services
    this.inventoryClient.emit('order.created', { orderId: order.id, productId: orderData.productId });
    this.paymentClient.emit('order.created', { orderId: order.id, amount: orderData.total });
    
    // 3. Return immediately (inventory/payment not updated yet)
    return order;
    
    // At this point:
    // - Order exists in Orders DB
    // - Inventory NOT yet decremented (eventual)
    // - Payment NOT yet processed (eventual)
    // Eventually all services will be consistent
  }
}
```

### **How It Works**

**Timeline of Eventual Consistency:**

```
Time    Order Service      Inventory Service    Payment Service
----    -------------      -----------------    ---------------
t0      Create Order       (no change)          (no change)        ← INCONSISTENT
        OrderId: 123
        Status: Pending
        
t1      (no change)        Receive Event        (no change)        ← INCONSISTENT
                           Process...
                           
t2      (no change)        Decrement Stock      Receive Event      ← INCONSISTENT
                           Stock: 10 → 9        Process...
                           
t3      (no change)        (no change)          Charge Payment     ← INCONSISTENT
                                                Status: Paid
                                                
t4      Update Status      (no change)          (no change)        ← CONSISTENT
        Status: Paid
        
// At t0-t3: System is INCONSISTENT (temporary)
// At t4: System is CONSISTENT (eventual)
// No reads at t0-t3 will see the full picture
```

### **Implementation Patterns**

**1. Event-Driven Propagation**

```typescript
// orders.service.ts
@Injectable()
export class OrdersService {
  constructor(
    @InjectRepository(Order) private orderRepo: Repository<Order>,
    @Inject('EVENT_BUS') private eventBus: ClientProxy,
  ) {}

  async createOrder(createOrderDto: CreateOrderDto): Promise<Order> {
    // 1. Write to local database (immediate consistency locally)
    const order = await this.orderRepo.save({
      ...createOrderDto,
      status: OrderStatus.PENDING,
      version: 1,
    });

    // 2. Publish event to event bus (eventual consistency globally)
    await this.eventBus.emit('order.created', {
      orderId: order.id,
      customerId: order.customerId,
      items: order.items,
      total: order.total,
      timestamp: new Date(),
    });

    return order;
  }

  // Read may return stale data during propagation
  async getOrderWithInventory(orderId: string) {
    const order = await this.orderRepo.findOne(orderId);
    // Warning: Order exists but inventory might not be updated yet
    return order;
  }
}

// inventory.service.ts
@Injectable()
export class InventoryService {
  @EventPattern('order.created')
  async handleOrderCreated(data: OrderCreatedEvent) {
    // Eventually processes the event
    await this.updateInventory(data.items);
    
    // Publish another event for next step
    await this.eventBus.emit('inventory.updated', {
      orderId: data.orderId,
      items: data.items,
      timestamp: new Date(),
    });
  }

  private async updateInventory(items: OrderItem[]) {
    for (const item of items) {
      await this.inventoryRepo.decrement(
        { productId: item.productId },
        'quantity',
        item.quantity,
      );
    }
  }
}
```

**2. Read Your Own Writes (Session Consistency)**

```typescript
// Ensure users see their own changes immediately
@Injectable()
export class OrdersService {
  private localCache = new Map<string, Order>();

  async createOrder(createOrderDto: CreateOrderDto, userId: string): Promise<Order> {
    const order = await this.orderRepo.save(createOrderDto);
    
    // Cache locally for this user's session
    const cacheKey = `${userId}:${order.id}`;
    this.localCache.set(cacheKey, order);
    
    // Set TTL to clear cache after propagation window
    setTimeout(() => this.localCache.delete(cacheKey), 5000);
    
    // Emit event for eventual propagation
    await this.eventBus.emit('order.created', order);
    
    return order;
  }

  async getOrder(orderId: string, userId: string): Promise<Order> {
    const cacheKey = `${userId}:${orderId}`;
    
    // Check if user created this order recently
    if (this.localCache.has(cacheKey)) {
      return this.localCache.get(cacheKey); // Return user's own write immediately
    }
    
    // Otherwise query database (may be stale)
    return await this.orderRepo.findOne(orderId);
  }
}
```

**3. Conflict Resolution**

```typescript
// When multiple updates occur during propagation
@Injectable()
export class InventoryService {
  @EventPattern('inventory.update')
  async handleInventoryUpdate(data: InventoryUpdateEvent) {
    const current = await this.inventoryRepo.findOne(data.productId);
    
    // Last Write Wins (LWW) strategy
    if (data.timestamp > current.lastModified) {
      await this.inventoryRepo.update(data.productId, {
        quantity: data.quantity,
        lastModified: data.timestamp,
      });
    } else {
      // Ignore stale update
      console.log('Ignoring stale inventory update');
    }
  }
}

// Version-based conflict resolution (Optimistic Locking)
@Injectable()
export class ProductService {
  async updateProduct(productId: string, updates: UpdateProductDto) {
    const product = await this.productRepo.findOne(productId);
    
    // Include version in update
    const result = await this.productRepo.update(
      { id: productId, version: product.version }, // WHERE clause includes version
      { ...updates, version: product.version + 1 },
    );
    
    if (result.affected === 0) {
      throw new ConflictException('Product was modified by another process');
    }
  }
}
```

**4. Compensating Actions for Failures**

```typescript
// Handle eventual consistency failures
@Injectable()
export class OrderSagaService {
  @EventPattern('payment.failed')
  async handlePaymentFailed(data: PaymentFailedEvent) {
    // Eventually consistent rollback
    // 1. Update order status
    await this.orderRepo.update(data.orderId, { status: OrderStatus.FAILED });
    
    // 2. Emit compensation events
    await this.eventBus.emit('inventory.restore', {
      orderId: data.orderId,
      items: data.items,
    });
    
    // 3. Eventually all services will reflect the failure
  }

  @EventPattern('inventory.restored')
  async handleInventoryRestored(data: InventoryRestoredEvent) {
    // Update order with compensation status
    await this.orderRepo.update(data.orderId, {
      status: OrderStatus.CANCELLED,
      cancelReason: 'Payment failed - inventory restored',
    });
  }
}
```

**5. Idempotency for Event Replay**

```typescript
// Ensure events can be processed multiple times safely
@Injectable()
export class InventoryService {
  @EventPattern('order.created')
  async handleOrderCreated(data: OrderCreatedEvent) {
    // Check if already processed
    const processed = await this.processedEventsRepo.findOne({
      eventId: data.eventId,
    });
    
    if (processed) {
      console.log('Event already processed, skipping');
      return; // Idempotent - safe to replay
    }
    
    // Process event
    await this.updateInventory(data.items);
    
    // Mark as processed
    await this.processedEventsRepo.save({
      eventId: data.eventId,
      processedAt: new Date(),
    });
  }
}
```

### **Guarantees and Trade-offs**

**CAP Theorem:**

```typescript
/*
CAP Theorem: Can only achieve 2 of 3:
- Consistency (C): All nodes see the same data
- Availability (A): Every request gets a response
- Partition Tolerance (P): System works despite network failures

Eventual Consistency chooses: AP (Availability + Partition Tolerance)
- Sacrifices immediate Consistency
- Gains high Availability and Partition Tolerance

Strong Consistency chooses: CP (Consistency + Partition Tolerance)
- Sacrifices Availability during network partitions
- Gains immediate Consistency
*/

// Example: Bank transfer (requires strong consistency)
class BankTransferService {
  // MUST use strong consistency - money cannot be lost
  async transfer(from: string, to: string, amount: number) {
    await this.db.transaction(async (trx) => {
      await trx.accounts.decrement(from, amount); // Must be atomic
      await trx.accounts.increment(to, amount);   // Must be atomic
    });
    // Strong consistency: both accounts updated atomically or not at all
  }
}

// Example: Social media likes (can use eventual consistency)
class LikesService {
  // Can tolerate temporary inconsistency - UX is fine
  async likePost(postId: string, userId: string) {
    await this.likesRepo.save({ postId, userId }); // Write locally
    await this.eventBus.emit('post.liked', { postId, userId }); // Propagate eventually
    // Eventual consistency: like count may be slightly off temporarily
  }
}
```

**When to Use Eventual Consistency:**

```typescript
// ✅ GOOD: Non-critical data (likes, views, comments)
@EventPattern('post.viewed')
async handlePostViewed(data: { postId: string }) {
  await this.postsRepo.increment({ id: data.postId }, 'views', 1);
  // Eventual consistency OK - view count doesn't need to be exact
}

// ✅ GOOD: Read-heavy systems (caching, replicas)
@Injectable()
export class ProductCatalogService {
  // Read from eventually consistent replica
  async getProducts() {
    return await this.readReplica.products.find(); // May be slightly stale
  }
  
  // Write to primary
  async updateProduct(productId: string, updates: UpdateProductDto) {
    await this.primary.products.update(productId, updates);
    // Replica will eventually receive update
  }
}

// ❌ BAD: Financial transactions (money must be exact)
@Injectable()
export class PaymentService {
  // DO NOT use eventual consistency for money
  async processPayment(amount: number) {
    // MUST use strong consistency / distributed transaction
    await this.db.transaction(async (trx) => {
      await trx.accounts.decrement('customer', amount);
      await trx.accounts.increment('merchant', amount);
    });
  }
}

// ❌ BAD: Inventory (overselling is unacceptable)
@Injectable()
export class InventoryService {
  // DO NOT use eventual consistency for stock quantity
  async reserveStock(productId: string, quantity: number) {
    // MUST use strong consistency / locking
    const result = await this.inventoryRepo.decrement(
      { productId, quantity: { $gte: quantity } }, // Check before decrement
      'quantity',
      quantity,
    );
    
    if (result.affected === 0) {
      throw new Error('Insufficient stock');
    }
  }
}
```

### **Monitoring and Observability**

```typescript
@Injectable()
export class EventualConsistencyMonitor {
  // Track propagation delay
  async trackPropagationDelay(eventId: string, timestamp: Date) {
    const now = new Date();
    const delay = now.getTime() - timestamp.getTime();
    
    // Alert if delay exceeds threshold
    if (delay > 5000) {
      console.warn(`Event ${eventId} took ${delay}ms to propagate`);
      // Send alert to monitoring system
    }
  }

  // Check data consistency across services
  async verifyConsistency(orderId: string) {
    const [order, inventory, payment] = await Promise.all([
      this.orderClient.send('get.order', orderId).toPromise(),
      this.inventoryClient.send('get.inventory', orderId).toPromise(),
      this.paymentClient.send('get.payment', orderId).toPromise(),
    ]);
    
    // Check if data is consistent
    if (order.status === 'paid' && payment.status !== 'completed') {
      // Inconsistency detected
      console.error('Inconsistency detected between order and payment');
      // Trigger reconciliation process
    }
  }
}
```

### **Real-World Example: E-commerce Order**

```typescript
// Complete flow with eventual consistency

// 1. Order Service - Create order
@Controller()
export class OrdersController {
  @Post('orders')
  async createOrder(@Body() createOrderDto: CreateOrderDto) {
    // Create order immediately (consistent locally)
    const order = await this.orderService.create({
      ...createOrderDto,
      status: OrderStatus.PENDING,
      createdAt: new Date(),
    });
    
    // Publish event (eventual consistency starts here)
    await this.eventBus.emit('order.created', {
      eventId: uuid(),
      orderId: order.id,
      items: order.items,
      total: order.total,
      timestamp: new Date(),
    });
    
    // Return to user immediately (other services not updated yet)
    return {
      orderId: order.id,
      status: 'pending',
      message: 'Order is being processed',
    };
  }
}

// 2. Inventory Service - Eventually updates stock
@Controller()
export class InventoryController {
  @EventPattern('order.created')
  async handleOrderCreated(data: OrderCreatedEvent) {
    try {
      // Process after some delay (network, queue, etc.)
      for (const item of data.items) {
        await this.inventoryService.decrementStock(
          item.productId,
          item.quantity,
        );
      }
      
      // Publish success event
      await this.eventBus.emit('inventory.reserved', {
        eventId: uuid(),
        orderId: data.orderId,
        timestamp: new Date(),
      });
    } catch (error) {
      // Publish failure event for compensation
      await this.eventBus.emit('inventory.failed', {
        eventId: uuid(),
        orderId: data.orderId,
        reason: error.message,
        timestamp: new Date(),
      });
    }
  }
}

// 3. Payment Service - Eventually charges payment
@Controller()
export class PaymentController {
  @EventPattern('inventory.reserved')
  async handleInventoryReserved(data: InventoryReservedEvent) {
    try {
      // Process payment after inventory is reserved
      const payment = await this.paymentService.charge(
        data.orderId,
        data.amount,
      );
      
      // Publish success event
      await this.eventBus.emit('payment.completed', {
        eventId: uuid(),
        orderId: data.orderId,
        paymentId: payment.id,
        timestamp: new Date(),
      });
    } catch (error) {
      // Publish failure event for compensation
      await this.eventBus.emit('payment.failed', {
        eventId: uuid(),
        orderId: data.orderId,
        reason: error.message,
        timestamp: new Date(),
      });
    }
  }
}

// 4. Order Service - Eventually updates order status
@Controller()
export class OrdersController {
  @EventPattern('payment.completed')
  async handlePaymentCompleted(data: PaymentCompletedEvent) {
    // Finally mark order as completed
    await this.orderService.update(data.orderId, {
      status: OrderStatus.COMPLETED,
      paidAt: new Date(),
    });
    
    // Send confirmation to customer
    await this.notificationService.sendOrderConfirmation(data.orderId);
  }
  
  @EventPattern('payment.failed')
  async handlePaymentFailed(data: PaymentFailedEvent) {
    // Compensate - restore inventory, cancel order
    await this.orderService.update(data.orderId, {
      status: OrderStatus.FAILED,
      failReason: data.reason,
    });
    
    await this.eventBus.emit('inventory.restore', {
      eventId: uuid(),
      orderId: data.orderId,
      timestamp: new Date(),
    });
  }
}

/*
Flow timeline:
t0: User creates order → Order status: PENDING
t1: Inventory service receives event → Inventory: NOT YET UPDATED
t2: Inventory service processes → Inventory: RESERVED
t3: Payment service receives event → Payment: NOT YET CHARGED
t4: Payment service processes → Payment: CHARGED
t5: Order service receives event → Order status: COMPLETED

Eventual consistency window: t0 to t5 (typically 100ms - 5s)
During this window, different services have different views of the data
*/
```

### **Best Practices**

```typescript
// 1. Idempotency Keys
@Injectable()
export class IdempotentEventHandler {
  @EventPattern('order.created')
  async handleOrderCreated(data: OrderCreatedEvent) {
    // Always check if event already processed
    const existing = await this.processedEvents.findOne({ eventId: data.eventId });
    if (existing) return; // Already processed
    
    await this.processOrder(data);
    await this.processedEvents.save({ eventId: data.eventId });
  }
}

// 2. Versioning for Conflict Resolution
@Entity()
export class Product {
  @PrimaryGeneratedColumn('uuid')
  id: string;
  
  @Column()
  name: string;
  
  @VersionColumn() // Auto-increments on each update
  version: number;
  
  @Column()
  lastModified: Date;
}

// 3. Timeout Handling
@Injectable()
export class EventTimeoutMonitor {
  async trackEvent(eventId: string) {
    // Set timeout for event processing
    setTimeout(async () => {
      const processed = await this.checkIfProcessed(eventId);
      if (!processed) {
        // Event not processed within timeout - trigger alert
        await this.alertUnprocessedEvent(eventId);
      }
    }, 30000); // 30 second timeout
  }
}

// 4. Reconciliation Jobs
@Injectable()
export class ReconciliationService {
  @Cron('0 */1 * * * *') // Every hour
  async reconcileData() {
    // Find inconsistencies between services
    const orders = await this.orderService.findPendingOrders();
    
    for (const order of orders) {
      const [inventory, payment] = await Promise.all([
        this.inventoryService.getReservation(order.id),
        this.paymentService.getPayment(order.id),
      ]);
      
      // Check if data is consistent
      if (!inventory || !payment) {
        // Inconsistency found - fix it
        await this.fixInconsistency(order);
      }
    }
  }
}
```

### **Common Mistakes**

```typescript
// ❌ MISTAKE 1: Not handling duplicate events
@EventPattern('order.created')
async handleOrderCreated(data: OrderCreatedEvent) {
  // BAD: No idempotency check
  await this.decrementInventory(data.items);
  // If event is delivered twice, inventory decremented twice!
}

// ✅ CORRECT: Handle duplicates
@EventPattern('order.created')
async handleOrderCreated(data: OrderCreatedEvent) {
  const processed = await this.processedEvents.findOne({ eventId: data.eventId });
  if (processed) return;
  
  await this.decrementInventory(data.items);
  await this.processedEvents.save({ eventId: data.eventId });
}

// ❌ MISTAKE 2: Assuming immediate consistency
@Get('orders/:id')
async getOrder(@Param('id') id: string) {
  const order = await this.orderService.findOne(id);
  // BAD: Assuming payment/inventory are already updated
  return {
    order,
    paymentStatus: order.paymentStatus, // May be stale!
    inventoryStatus: order.inventoryStatus, // May be stale!
  };
}

// ✅ CORRECT: Indicate data may be eventual
@Get('orders/:id')
async getOrder(@Param('id') id: string) {
  const order = await this.orderService.findOne(id);
  return {
    order,
    status: order.status,
    note: order.status === 'pending' 
      ? 'Order is being processed. Status may take a few moments to update.'
      : null,
  };
}

// ❌ MISTAKE 3: No compensation for failures
@EventPattern('payment.failed')
async handlePaymentFailed(data: PaymentFailedEvent) {
  // BAD: Just update order, don't restore inventory
  await this.orderService.update(data.orderId, { status: 'failed' });
  // Inventory is still reserved - inconsistent!
}

// ✅ CORRECT: Compensate all changes
@EventPattern('payment.failed')
async handlePaymentFailed(data: PaymentFailedEvent) {
  await this.orderService.update(data.orderId, { status: 'failed' });
  await this.eventBus.emit('inventory.restore', { orderId: data.orderId });
  // Inventory will be restored eventually
}
```

**Interview Tip**: **Eventual consistency** means updates **propagate eventually** (not immediately). **Key characteristics**: temporary inconsistency accepted (different nodes have different values), high availability (no blocking), partition tolerance (works during network failures), conflict resolution needed (version vectors, LWW, CRDTs). **Implementation**: event-driven propagation (emit events, services consume asynchronously), idempotency (handle duplicate events safely), compensating actions (rollback on failure), reconciliation jobs (periodic consistency checks). **When to use**: non-critical data (likes, views, comments), read-heavy systems (caching, replicas), distributed systems requiring high availability. **When NOT to use**: financial transactions (money must be exact), inventory (overselling unacceptable), authentication (security critical). **Trade-off**: sacrifice immediate consistency for availability and performance. **Best practice**: design for eventual consistency from the start, implement idempotency, monitor propagation delays, provide user feedback about pending status.

</details>
48. How do you implement inter-service authentication?

<details>
<summary><strong>Answer</strong></summary>

**Inter-service authentication** ensures that only authorized services can communicate with each other in a microservices architecture. Unlike user authentication (users accessing APIs), inter-service authentication verifies the identity of services calling other services. This prevents unauthorized services from accessing sensitive endpoints and protects against security breaches.

### **Why Inter-Service Authentication is Critical**

```typescript
// WITHOUT inter-service authentication (INSECURE)
@Controller('payments')
export class PaymentsController {
  @Post('charge')
  async chargePayment(@Body() data: ChargeDto) {
    // Problem: Anyone can call this endpoint!
    // Malicious service could charge arbitrary amounts
    return await this.paymentService.charge(data.amount);
  }
}

// WITH inter-service authentication (SECURE)
@Controller('payments')
export class PaymentsController {
  @Post('charge')
  @UseGuards(ServiceAuthGuard) // Only authenticated services can call
  async chargePayment(@Body() data: ChargeDto, @Service() service: ServiceIdentity) {
    // Verify the calling service is authorized
    if (service.name !== 'order-service') {
      throw new ForbiddenException('Only order-service can charge payments');
    }
    return await this.paymentService.charge(data.amount);
  }
}
```

### **Approach 1: JWT-Based Service Authentication**

Each service has a unique JWT token that identifies itself to other services.

**Implementation:**

```typescript
// auth/service-jwt.strategy.ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';

@Injectable()
export class ServiceJwtStrategy extends PassportStrategy(Strategy, 'service-jwt') {
  constructor(private configService: ConfigService) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      secretOrKey: configService.get('SERVICE_JWT_SECRET'), // Shared secret
      issuer: 'auth-service', // Must be issued by auth service
      audience: 'microservices', // For inter-service communication
    });
  }

  async validate(payload: any) {
    // Validate JWT payload
    if (!payload.serviceId || !payload.serviceName) {
      throw new UnauthorizedException('Invalid service token');
    }

    return {
      serviceId: payload.serviceId,
      serviceName: payload.serviceName,
      permissions: payload.permissions,
    };
  }
}

// auth/service-auth.guard.ts
import { Injectable, ExecutionContext } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class ServiceAuthGuard extends AuthGuard('service-jwt') {
  canActivate(context: ExecutionContext) {
    return super.canActivate(context);
  }
}

// Custom decorator to extract service info
export const Service = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return request.user; // Contains service identity from JWT
  },
);
```

**Service Client with JWT:**

```typescript
// payment.client.ts
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { ClientProxy, ClientProxyFactory, Transport } from '@nestjs/microservices';
import * as jwt from 'jsonwebtoken';

@Injectable()
export class PaymentClient {
  private client: ClientProxy;
  private serviceToken: string;

  constructor(private configService: ConfigService) {
    // Generate service JWT token
    this.serviceToken = jwt.sign(
      {
        serviceId: 'order-service-001',
        serviceName: 'order-service',
        permissions: ['payment.charge', 'payment.refund'],
      },
      this.configService.get('SERVICE_JWT_SECRET'),
      {
        issuer: 'auth-service',
        audience: 'microservices',
        expiresIn: '1h', // Token expiration
      },
    );

    this.client = ClientProxyFactory.create({
      transport: Transport.TCP,
      options: { host: 'payment-service', port: 3001 },
    });
  }

  async chargePayment(orderId: string, amount: number) {
    // Include service token in metadata
    return this.client
      .send('payment.charge', {
        orderId,
        amount,
        metadata: {
          authorization: `Bearer ${this.serviceToken}`,
        },
      })
      .toPromise();
  }
}

// payment.controller.ts - Receiving service
@Controller()
export class PaymentController {
  @MessagePattern('payment.charge')
  @UseGuards(ServiceAuthGuard) // Validate service JWT
  async chargePayment(
    @Payload() data: ChargePaymentDto,
    @Service() service: ServiceIdentity,
  ) {
    // service contains { serviceId, serviceName, permissions }
    console.log(`Payment request from ${service.serviceName}`);

    // Check if service has permission
    if (!service.permissions.includes('payment.charge')) {
      throw new ForbiddenException('Service lacks payment.charge permission');
    }

    return await this.paymentService.charge(data.orderId, data.amount);
  }
}
```

### **Approach 2: API Keys per Service**

Each service has a unique API key that it includes in requests.

```typescript
// auth/api-key.guard.ts
import { Injectable, CanActivate, ExecutionContext, UnauthorizedException } from '@nestjs/common';

@Injectable()
export class ApiKeyGuard implements CanActivate {
  // Store valid API keys (in production, load from database/vault)
  private readonly validApiKeys = new Map<string, ServiceInfo>([
    ['sk_order_service_abc123', { name: 'order-service', permissions: ['payment.charge'] }],
    ['sk_inventory_service_xyz789', { name: 'inventory-service', permissions: ['product.update'] }],
  ]);

  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const apiKey = request.headers['x-api-key'];

    if (!apiKey) {
      throw new UnauthorizedException('API key missing');
    }

    const serviceInfo = this.validApiKeys.get(apiKey);
    if (!serviceInfo) {
      throw new UnauthorizedException('Invalid API key');
    }

    // Attach service info to request
    request.service = serviceInfo;
    return true;
  }
}

interface ServiceInfo {
  name: string;
  permissions: string[];
}

// Usage in controller
@Controller('payments')
export class PaymentsController {
  @Post('charge')
  @UseGuards(ApiKeyGuard)
  async chargePayment(
    @Body() data: ChargeDto,
    @Req() req: Request & { service: ServiceInfo },
  ) {
    console.log(`Request from service: ${req.service.name}`);
    
    if (!req.service.permissions.includes('payment.charge')) {
      throw new ForbiddenException('Service not authorized for payment.charge');
    }
    
    return await this.paymentService.charge(data.amount);
  }
}

// Service client with API key
@Injectable()
export class PaymentClient {
  constructor(
    private httpService: HttpService,
    private configService: ConfigService,
  ) {}

  async chargePayment(amount: number) {
    return this.httpService
      .post(
        'http://payment-service/payments/charge',
        { amount },
        {
          headers: {
            'X-API-Key': this.configService.get('ORDER_SERVICE_API_KEY'),
          },
        },
      )
      .toPromise();
  }
}
```

### **Approach 3: Mutual TLS (mTLS)**

Services authenticate each other using SSL/TLS certificates.

```typescript
// bootstrap.ts - Server with mTLS
import { NestFactory } from '@nestjs/core';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';
import * as fs from 'fs';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(AppModule, {
    transport: Transport.TCP,
    options: {
      host: '0.0.0.0',
      port: 3001,
      // Enable mTLS
      tlsOptions: {
        key: fs.readFileSync('./certs/server-key.pem'),
        cert: fs.readFileSync('./certs/server-cert.pem'),
        ca: fs.readFileSync('./certs/ca-cert.pem'), // Certificate Authority
        requestCert: true, // Request client certificate
        rejectUnauthorized: true, // Reject invalid certificates
      },
    },
  });

  await app.listen();
}
bootstrap();

// Client with mTLS
@Module({
  providers: [
    {
      provide: 'PAYMENT_SERVICE',
      useFactory: () => {
        return ClientProxyFactory.create({
          transport: Transport.TCP,
          options: {
            host: 'payment-service',
            port: 3001,
            // Client certificate for mTLS
            tlsOptions: {
              key: fs.readFileSync('./certs/client-key.pem'),
              cert: fs.readFileSync('./certs/client-cert.pem'),
              ca: fs.readFileSync('./certs/ca-cert.pem'),
              rejectUnauthorized: true,
            },
          },
        });
      },
    },
  ],
})
export class OrderModule {}

// Extract service identity from certificate
@Injectable()
export class CertificateAuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const cert = request.socket.getPeerCertificate();

    if (!cert || !cert.subject) {
      throw new UnauthorizedException('Client certificate required');
    }

    // Extract service name from certificate CN (Common Name)
    const serviceName = cert.subject.CN;
    request.service = { name: serviceName };

    return true;
  }
}
```

### **Approach 4: Service Mesh (Istio/Linkerd)**

Service mesh handles authentication automatically.

```yaml
# istio-peer-authentication.yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: microservices
spec:
  mtls:
    mode: STRICT # Enforce mTLS between all services

---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: payment-service-authz
  namespace: microservices
spec:
  selector:
    matchLabels:
      app: payment-service
  action: ALLOW
  rules:
  - from:
    - source:
        principals:
        - "cluster.local/ns/microservices/sa/order-service" # Only order-service allowed
    to:
    - operation:
        methods: ["POST"]
        paths: ["/payments/charge"]
```

**NestJS code with Istio (no manual auth needed):**

```typescript
// No authentication guard needed - Istio handles it!
@Controller('payments')
export class PaymentsController {
  @Post('charge')
  async chargePayment(@Body() data: ChargeDto, @Headers() headers: any) {
    // Istio injects service identity in headers
    const callingService = headers['x-forwarded-client-cert'];
    console.log(`Request from: ${callingService}`);
    
    // Istio already verified the service identity
    return await this.paymentService.charge(data.amount);
  }
}

// Client - no certificate configuration needed
@Injectable()
export class PaymentClient {
  constructor(@Inject('PAYMENT_SERVICE') private client: ClientProxy) {}

  async chargePayment(amount: number) {
    // Istio automatically adds mTLS and service identity
    return this.client.send('payment.charge', { amount }).toPromise();
  }
}
```

### **Approach 5: OAuth 2.0 Client Credentials Flow**

Services obtain access tokens from an authorization server.

```typescript
// auth/oauth.service.ts
import { Injectable } from '@nestjs/common';
import { HttpService } from '@nestjs/axios';
import { firstValueFrom } from 'rxjs';

@Injectable()
export class OAuthService {
  private accessToken: string;
  private tokenExpiry: Date;

  constructor(
    private httpService: HttpService,
    private configService: ConfigService,
  ) {}

  async getAccessToken(): Promise<string> {
    // Return cached token if still valid
    if (this.accessToken && this.tokenExpiry > new Date()) {
      return this.accessToken;
    }

    // Request new token using client credentials
    const response = await firstValueFrom(
      this.httpService.post(
        'https://auth-service/oauth/token',
        new URLSearchParams({
          grant_type: 'client_credentials',
          client_id: this.configService.get('CLIENT_ID'),
          client_secret: this.configService.get('CLIENT_SECRET'),
          scope: 'payment.charge payment.refund',
        }),
        {
          headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
        },
      ),
    );

    this.accessToken = response.data.access_token;
    this.tokenExpiry = new Date(Date.now() + response.data.expires_in * 1000);

    return this.accessToken;
  }
}

// payment.client.ts
@Injectable()
export class PaymentClient {
  constructor(
    private httpService: HttpService,
    private oauthService: OAuthService,
  ) {}

  async chargePayment(amount: number) {
    const accessToken = await this.oauthService.getAccessToken();

    return firstValueFrom(
      this.httpService.post(
        'http://payment-service/payments/charge',
        { amount },
        {
          headers: {
            Authorization: `Bearer ${accessToken}`,
          },
        },
      ),
    );
  }
}

// payment.controller.ts
@Controller('payments')
export class PaymentsController {
  @Post('charge')
  @UseGuards(JwtAuthGuard) // Validates OAuth token
  async chargePayment(
    @Body() data: ChargeDto,
    @Req() req: Request,
  ) {
    // Token already validated by guard
    const scope = req.user['scope']; // e.g., ['payment.charge', 'payment.refund']
    
    if (!scope.includes('payment.charge')) {
      throw new ForbiddenException('Insufficient scope');
    }
    
    return await this.paymentService.charge(data.amount);
  }
}
```

### **Approach 6: Shared Secret / HMAC Signatures**

Services sign requests with a shared secret.

```typescript
// auth/hmac.service.ts
import { Injectable } from '@nestjs/common';
import * as crypto from 'crypto';

@Injectable()
export class HmacService {
  generateSignature(payload: any, secret: string): string {
    const message = JSON.stringify(payload);
    return crypto.createHmac('sha256', secret).update(message).digest('hex');
  }

  verifySignature(payload: any, signature: string, secret: string): boolean {
    const expectedSignature = this.generateSignature(payload, secret);
    return crypto.timingSafeEqual(
      Buffer.from(signature),
      Buffer.from(expectedSignature),
    );
  }
}

// auth/hmac.guard.ts
@Injectable()
export class HmacGuard implements CanActivate {
  constructor(
    private hmacService: HmacService,
    private configService: ConfigService,
  ) {}

  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const signature = request.headers['x-signature'];
    const timestamp = request.headers['x-timestamp'];

    if (!signature || !timestamp) {
      throw new UnauthorizedException('Missing signature or timestamp');
    }

    // Prevent replay attacks (reject old requests)
    const requestTime = parseInt(timestamp);
    const now = Date.now();
    if (now - requestTime > 300000) { // 5 minutes
      throw new UnauthorizedException('Request expired');
    }

    // Verify signature
    const payload = { ...request.body, timestamp };
    const secret = this.configService.get('SHARED_SECRET');

    if (!this.hmacService.verifySignature(payload, signature, secret)) {
      throw new UnauthorizedException('Invalid signature');
    }

    return true;
  }
}

// Client with HMAC signing
@Injectable()
export class PaymentClient {
  constructor(
    private httpService: HttpService,
    private hmacService: HmacService,
    private configService: ConfigService,
  ) {}

  async chargePayment(amount: number) {
    const timestamp = Date.now().toString();
    const payload = { amount, timestamp };
    const secret = this.configService.get('SHARED_SECRET');
    const signature = this.hmacService.generateSignature(payload, secret);

    return firstValueFrom(
      this.httpService.post(
        'http://payment-service/payments/charge',
        { amount },
        {
          headers: {
            'X-Signature': signature,
            'X-Timestamp': timestamp,
          },
        },
      ),
    );
  }
}
```

### **Complete Example: JWT + Permission-Based Authorization**

```typescript
// auth/service-auth.module.ts
import { Module, Global } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { ConfigModule, ConfigService } from '@nestjs/config';

@Global()
@Module({
  imports: [
    JwtModule.registerAsync({
      imports: [ConfigModule],
      useFactory: (configService: ConfigService) => ({
        secret: configService.get('SERVICE_JWT_SECRET'),
        signOptions: {
          expiresIn: '1h',
          issuer: 'auth-service',
          audience: 'microservices',
        },
      }),
      inject: [ConfigService],
    }),
  ],
  providers: [ServiceAuthService, ServiceJwtStrategy],
  exports: [ServiceAuthService, JwtModule],
})
export class ServiceAuthModule {}

// auth/service-auth.service.ts
@Injectable()
export class ServiceAuthService {
  constructor(private jwtService: JwtService) {}

  generateServiceToken(serviceName: string, permissions: string[]): string {
    return this.jwtService.sign({
      serviceId: `${serviceName}-${Date.now()}`,
      serviceName,
      permissions,
      type: 'service',
    });
  }

  verifyServiceToken(token: string): ServiceTokenPayload {
    return this.jwtService.verify(token);
  }
}

interface ServiceTokenPayload {
  serviceId: string;
  serviceName: string;
  permissions: string[];
  type: string;
}

// auth/permissions.decorator.ts
import { SetMetadata } from '@nestjs/common';

export const RequirePermissions = (...permissions: string[]) =>
  SetMetadata('permissions', permissions);

// auth/permissions.guard.ts
@Injectable()
export class PermissionsGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredPermissions = this.reflector.get<string[]>(
      'permissions',
      context.getHandler(),
    );

    if (!requiredPermissions) {
      return true; // No permissions required
    }

    const request = context.switchToHttp().getRequest();
    const service = request.user; // From JWT strategy

    if (!service || !service.permissions) {
      return false;
    }

    // Check if service has all required permissions
    return requiredPermissions.every((permission) =>
      service.permissions.includes(permission),
    );
  }
}

// payment.controller.ts - Complete example
@Controller('payments')
export class PaymentsController {
  @Post('charge')
  @UseGuards(ServiceAuthGuard, PermissionsGuard)
  @RequirePermissions('payment.charge') // Declarative permission check
  async chargePayment(
    @Body() data: ChargeDto,
    @Service() service: ServiceTokenPayload,
  ) {
    console.log(`Payment request from ${service.serviceName}`);
    return await this.paymentService.charge(data.orderId, data.amount);
  }

  @Post('refund')
  @UseGuards(ServiceAuthGuard, PermissionsGuard)
  @RequirePermissions('payment.refund') // Different permission
  async refundPayment(
    @Body() data: RefundDto,
    @Service() service: ServiceTokenPayload,
  ) {
    console.log(`Refund request from ${service.serviceName}`);
    return await this.paymentService.refund(data.paymentId);
  }
}

// order.service.ts - Client with automatic token injection
@Injectable()
export class OrderService {
  private paymentClient: ClientProxy;
  private serviceToken: string;

  constructor(
    private serviceAuthService: ServiceAuthService,
    @Inject('PAYMENT_SERVICE') paymentClient: ClientProxy,
  ) {
    this.paymentClient = paymentClient;
    
    // Generate service token on startup
    this.serviceToken = this.serviceAuthService.generateServiceToken(
      'order-service',
      ['payment.charge', 'payment.refund'],
    );
  }

  async createOrder(createOrderDto: CreateOrderDto) {
    const order = await this.orderRepository.save(createOrderDto);

    // Call payment service with authentication
    const payment = await this.paymentClient
      .send('payment.charge', {
        orderId: order.id,
        amount: order.total,
        metadata: {
          authorization: `Bearer ${this.serviceToken}`,
        },
      })
      .toPromise();

    return { order, payment };
  }
}
```

### **Best Practices**

```typescript
// 1. Token Rotation
@Injectable()
export class TokenRotationService {
  private currentToken: string;
  private nextToken: string;
  private rotationInterval = 3600000; // 1 hour

  constructor(private serviceAuthService: ServiceAuthService) {
    this.rotateTokens();
    setInterval(() => this.rotateTokens(), this.rotationInterval);
  }

  private rotateTokens() {
    this.nextToken = this.serviceAuthService.generateServiceToken(
      'order-service',
      ['payment.charge'],
    );
    
    // Keep old token valid during transition
    setTimeout(() => {
      this.currentToken = this.nextToken;
    }, 60000); // 1 minute overlap
  }

  getToken(): string {
    return this.currentToken || this.nextToken;
  }
}

// 2. Rate Limiting per Service
@Injectable()
export class ServiceRateLimitGuard implements CanActivate {
  private rateLimits = new Map<string, RateLimitInfo>();

  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const service = request.user;

    const rateLimitInfo = this.rateLimits.get(service.serviceName) || {
      count: 0,
      resetTime: Date.now() + 60000,
    };

    if (Date.now() > rateLimitInfo.resetTime) {
      rateLimitInfo.count = 0;
      rateLimitInfo.resetTime = Date.now() + 60000;
    }

    if (rateLimitInfo.count >= 100) { // 100 requests per minute
      throw new TooManyRequestsException('Rate limit exceeded');
    }

    rateLimitInfo.count++;
    this.rateLimits.set(service.serviceName, rateLimitInfo);

    return true;
  }
}

// 3. Audit Logging
@Injectable()
export class ServiceAuditInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler) {
    const request = context.switchToHttp().getRequest();
    const service = request.user;

    console.log(`[AUDIT] Service ${service.serviceName} accessed ${request.url}`);

    return next.handle().pipe(
      tap(() => {
        console.log(`[AUDIT] Service ${service.serviceName} request succeeded`);
      }),
      catchError((error) => {
        console.error(
          `[AUDIT] Service ${service.serviceName} request failed: ${error.message}`,
        );
        throw error;
      }),
    );
  }
}

// 4. Service Registry
@Injectable()
export class ServiceRegistry {
  private services = new Map<string, ServiceConfig>([
    [
      'order-service',
      {
        name: 'order-service',
        permissions: ['payment.charge', 'inventory.reserve'],
        ipWhitelist: ['10.0.1.0/24'],
      },
    ],
    [
      'admin-service',
      {
        name: 'admin-service',
        permissions: ['*'], // All permissions
        ipWhitelist: ['10.0.2.0/24'],
      },
    ],
  ]);

  getServiceConfig(serviceName: string): ServiceConfig | undefined {
    return this.services.get(serviceName);
  }

  isServiceAuthorized(serviceName: string, permission: string): boolean {
    const config = this.getServiceConfig(serviceName);
    if (!config) return false;

    return (
      config.permissions.includes('*') ||
      config.permissions.includes(permission)
    );
  }
}
```

### **Comparison of Approaches**

| Approach | Pros | Cons | Best For |
|----------|------|------|----------|
| **JWT** | Stateless, scalable, includes claims | Token can't be revoked easily | Medium-large deployments |
| **API Keys** | Simple, easy to implement | Static, hard to rotate | Small deployments, internal services |
| **mTLS** | Very secure, cryptographic | Complex setup, certificate management | High-security environments |
| **Service Mesh** | Automatic, no code changes | Requires infrastructure (Kubernetes) | Large, cloud-native deployments |
| **OAuth 2.0** | Industry standard, token refresh | More complex, requires auth server | Enterprise, external integrations |
| **HMAC** | Prevents tampering | Shared secret management | Simple, high-trust environments |

### **Common Mistakes**

```typescript
// ❌ MISTAKE 1: Using user JWT for service auth
@Post('charge')
@UseGuards(JwtAuthGuard) // This validates USER tokens!
async chargePayment(@Body() data: ChargeDto) {
  // Services shouldn't use user tokens - they need service tokens
  return await this.paymentService.charge(data.amount);
}

// ✅ CORRECT: Separate guard for service auth
@Post('charge')
@UseGuards(ServiceAuthGuard) // Validates SERVICE tokens
async chargePayment(@Body() data: ChargeDto) {
  return await this.paymentService.charge(data.amount);
}

// ❌ MISTAKE 2: Hardcoded API keys in code
const API_KEY = 'sk_order_service_abc123'; // BAD: Leaked in Git!

// ✅ CORRECT: Use environment variables
const API_KEY = this.configService.get('ORDER_SERVICE_API_KEY');

// ❌ MISTAKE 3: No token expiration
const token = jwt.sign({ serviceName: 'order-service' }, secret); // BAD: Never expires!

// ✅ CORRECT: Always set expiration
const token = jwt.sign(
  { serviceName: 'order-service' },
  secret,
  { expiresIn: '1h' }, // Expires after 1 hour
);

// ❌ MISTAKE 4: Not validating service identity
@Post('charge')
async chargePayment(@Body() data: ChargeDto) {
  // BAD: No check on which service is calling
  return await this.paymentService.charge(data.amount);
}

// ✅ CORRECT: Validate service identity and permissions
@Post('charge')
@UseGuards(ServiceAuthGuard)
async chargePayment(@Body() data: ChargeDto, @Service() service: ServiceIdentity) {
  if (!['order-service', 'admin-service'].includes(service.name)) {
    throw new ForbiddenException('Service not authorized');
  }
  return await this.paymentService.charge(data.amount);
}
```

**Interview Tip**: **Inter-service authentication** verifies service identity in microservices. **Common approaches**: JWT (stateless tokens with service claims), API keys (simple shared secrets), mTLS (certificate-based), service mesh (automatic with Istio/Linkerd), OAuth 2.0 (client credentials flow), HMAC signatures. **Implementation**: generate service token/certificate at startup, include in requests (Authorization header or metadata), validate in receiving service (guard/middleware), check permissions (can service perform this action?). **Best practices**: separate service auth from user auth (different tokens/guards), rotate credentials regularly, use short expiration times, implement rate limiting per service, audit all inter-service calls, store secrets securely (Vault, K8s secrets). **Common mistake**: using user JWT for service auth (services need their own tokens with service-specific permissions). **Production**: use service mesh (Istio) for automatic mTLS + authorization policies in Kubernetes, or JWT with short TTL for simpler deployments.

</details>
49. How do you handle versioning in microservices?

<details>
<summary><strong>Answer</strong></summary>

**API versioning** in microservices allows you to evolve services independently while maintaining backward compatibility for existing clients. As services change over time, versioning ensures that old clients continue to work while new clients can use enhanced features. Unlike monoliths where a single version is deployed, microservices can have multiple versions running simultaneously.

### **Why Versioning is Critical in Microservices**

```typescript
// WITHOUT versioning (BREAKS old clients)
// v1: Original API
@Get('users/:id')
getUser(@Param('id') id: string) {
  return { id, name: 'John' };
}

// Update to v2: BREAKING CHANGE - Added required field
@Get('users/:id')
getUser(@Param('id') id: string) {
  return {
    id,
    profile: { // BREAKING: Wrapped in 'profile' object
      name: 'John',
      email: 'john@example.com', // New field
    },
  };
}
// Old clients expect { id, name } but get { id, profile: {...} } - BREAKS!

// WITH versioning (Old clients continue working)
@Get('v1/users/:id')
getUserV1(@Param('id') id: string) {
  return { id, name: 'John' }; // Old format
}

@Get('v2/users/:id')
getUserV2(@Param('id') id: string) {
  return {
    id,
    profile: { name: 'John', email: 'john@example.com' }, // New format
  };
}
// Both versions work simultaneously!
```

### **Approach 1: URI Path Versioning**

Version in the URL path (most common, explicit).

```typescript
// users.controller.v1.ts
@Controller('v1/users')
export class UsersControllerV1 {
  @Get(':id')
  async getUser(@Param('id') id: string) {
    return {
      id,
      name: 'John Doe',
    };
  }

  @Post()
  async createUser(@Body() createUserDto: CreateUserDtoV1) {
    return await this.usersService.createV1(createUserDto);
  }
}

// users.controller.v2.ts
@Controller('v2/users')
export class UsersControllerV2 {
  @Get(':id')
  async getUser(@Param('id') id: string) {
    // Enhanced response with more fields
    return {
      id,
      profile: {
        name: 'John Doe',
        email: 'john@example.com',
        avatar: 'https://...',
      },
      metadata: {
        createdAt: new Date(),
        updatedAt: new Date(),
      },
    };
  }

  @Post()
  async createUser(@Body() createUserDto: CreateUserDtoV2) {
    return await this.usersService.createV2(createUserDto);
  }
}

// app.module.ts - Register both versions
@Module({
  imports: [UsersModuleV1, UsersModuleV2],
})
export class AppModule {}

// Clients use different URLs
// Old clients: GET /v1/users/123
// New clients: GET /v2/users/123
```

**Pros:** Explicit, easy to route, clear version in logs  
**Cons:** URL changes, need to duplicate controllers

### **Approach 2: Header-Based Versioning**

Version in request header (cleaner URLs).

```typescript
// versioning.interceptor.ts
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';

@Injectable()
export class VersioningInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler) {
    const request = context.switchToHttp().getRequest();
    const version = request.headers['api-version'] || '1'; // Default to v1
    request.apiVersion = version;
    return next.handle();
  }
}

// users.controller.ts
@Controller('users')
@UseInterceptors(VersioningInterceptor)
export class UsersController {
  constructor(private usersService: UsersService) {}

  @Get(':id')
  async getUser(@Param('id') id: string, @Req() req: Request & { apiVersion: string }) {
    if (req.apiVersion === '1') {
      return await this.usersService.getUserV1(id);
    } else if (req.apiVersion === '2') {
      return await this.usersService.getUserV2(id);
    } else {
      throw new BadRequestException('Unsupported API version');
    }
  }
}

// Clients specify version in header
// curl -H "api-version: 1" http://api/users/123
// curl -H "api-version: 2" http://api/users/123
```

**Pros:** Clean URLs, version in metadata  
**Cons:** Less visible, harder to test in browser

### **Approach 3: Content Negotiation (Accept Header)**

Version via media type in Accept header.

```typescript
// main.ts - Enable content negotiation versioning
import { VersioningType } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  app.enableVersioning({
    type: VersioningType.MEDIA_TYPE,
    key: 'v=', // application/json;v=1
  });
  
  await app.listen(3000);
}
bootstrap();

// users.controller.ts
@Controller('users')
export class UsersController {
  @Get(':id')
  @Version('1')
  async getUserV1(@Param('id') id: string) {
    return { id, name: 'John' };
  }

  @Get(':id')
  @Version('2')
  async getUserV2(@Param('id') id: string) {
    return { id, profile: { name: 'John', email: 'john@example.com' } };
  }
}

// Clients specify version in Accept header
// curl -H "Accept: application/json;v=1" http://api/users/123
// curl -H "Accept: application/json;v=2" http://api/users/123
```

**Pros:** RESTful, standard HTTP, clean URLs  
**Cons:** Complex, not beginner-friendly

### **Approach 4: Query Parameter Versioning**

Version as query parameter.

```typescript
// main.ts
import { VersioningType } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  app.enableVersioning({
    type: VersioningType.URI,
    defaultVersion: '1',
  });
  
  await app.listen(3000);
}
bootstrap();

// users.controller.ts
@Controller('users')
export class UsersController {
  @Get(':id')
  @Version('1')
  async getUserV1(@Param('id') id: string) {
    return { id, name: 'John' };
  }

  @Get(':id')
  @Version('2')
  async getUserV2(@Param('id') id: string) {
    return { id, profile: { name: 'John', email: 'john@example.com' } };
  }
}

// Alternative: Manual query parameter handling
@Get(':id')
async getUser(
  @Param('id') id: string,
  @Query('version') version: string = '1',
) {
  if (version === '1') {
    return this.usersService.getUserV1(id);
  } else if (version === '2') {
    return this.usersService.getUserV2(id);
  }
}

// Clients specify version in query
// GET /users/123?version=1
// GET /users/123?version=2
```

**Pros:** Simple, easy to test  
**Cons:** Pollutes query params, can be ignored accidentally

### **Approach 5: Schema Versioning (Contract-Based)**

Version the data contracts, not the endpoints.

```typescript
// DTOs with version support
// user.dto.v1.ts
export class UserDtoV1 {
  id: string;
  name: string;
}

// user.dto.v2.ts
export class UserDtoV2 {
  id: string;
  profile: {
    name: string;
    email: string;
  };
}

// Transformation service
@Injectable()
export class UserTransformService {
  toV1(user: User): UserDtoV1 {
    return {
      id: user.id,
      name: user.name,
    };
  }

  toV2(user: User): UserDtoV2 {
    return {
      id: user.id,
      profile: {
        name: user.name,
        email: user.email,
      },
    };
  }
}

// Single controller with dynamic transformation
@Controller('users')
export class UsersController {
  @Get(':id')
  async getUser(
    @Param('id') id: string,
    @Headers('api-version') version: string = '1',
  ) {
    const user = await this.usersService.findOne(id);
    
    if (version === '1') {
      return this.transformService.toV1(user);
    } else if (version === '2') {
      return this.transformService.toV2(user);
    }
    
    throw new BadRequestException('Unsupported version');
  }
}
```

### **Semantic Versioning in Microservices**

```typescript
// Use semantic versioning: MAJOR.MINOR.PATCH
// MAJOR: Breaking changes (incompatible API changes)
// MINOR: New features (backward compatible)
// PATCH: Bug fixes (backward compatible)

// Example: Service version 2.3.1
// Major: 2 (breaking changes from v1)
// Minor: 3 (3 new features added since v2.0)
// Patch: 1 (1 bug fix since v2.3.0)

// package.json
{
  "name": "users-service",
  "version": "2.3.1",
  "description": "Users microservice"
}

// Health check endpoint exposes version
@Controller('health')
export class HealthController {
  @Get()
  getHealth() {
    return {
      status: 'ok',
      version: process.env.npm_package_version, // 2.3.1
      apiVersions: ['1', '2'], // Supported API versions
      timestamp: new Date(),
    };
  }
}

// Service registry with version info
@Injectable()
export class ServiceDiscovery {
  async registerService() {
    await this.consul.agent.service.register({
      name: 'users-service',
      tags: [
        'v2', // Major version
        'api-v1', // Supports API v1
        'api-v2', // Supports API v2
      ],
      meta: {
        version: '2.3.1',
        apiVersions: '1,2',
      },
    });
  }
}
```

### **Handling Breaking Changes**

```typescript
// Strategy 1: Parallel Run (both versions simultaneously)
@Module({
  imports: [
    // v1 module
    UsersModuleV1,
    // v2 module
    UsersModuleV2,
  ],
})
export class AppModule {}

// Strategy 2: Adapter Pattern (v1 wraps v2)
@Injectable()
export class UsersServiceV1 {
  constructor(private usersServiceV2: UsersServiceV2) {}

  async getUser(id: string): Promise<UserDtoV1> {
    // Call v2 service
    const userV2 = await this.usersServiceV2.getUser(id);
    
    // Transform v2 response to v1 format
    return {
      id: userV2.id,
      name: userV2.profile.name, // Flatten structure
    };
  }
}

// Strategy 3: Deprecation Warnings
@Controller('v1/users')
export class UsersControllerV1 {
  @Get(':id')
  async getUser(@Param('id') id: string, @Res() res: Response) {
    const user = await this.usersService.getUser(id);
    
    // Add deprecation header
    res.setHeader('Warning', '299 - "API v1 is deprecated. Please migrate to v2 by 2025-12-31"');
    res.setHeader('Sunset', 'Sat, 31 Dec 2025 23:59:59 GMT'); // RFC 8594
    
    return res.json(user);
  }
}

// Strategy 4: Version Sunset (remove old versions)
@Controller('v1/users')
export class UsersControllerV1 {
  @Get(':id')
  async getUser(@Param('id') id: string) {
    // Check if v1 is sunset
    const sunsetDate = new Date('2025-12-31');
    if (new Date() > sunsetDate) {
      throw new GoneException(
        'API v1 has been sunset. Please use v2: https://api.example.com/v2/users',
      );
    }
    
    return await this.usersService.getUser(id);
  }
}
```

### **Versioning for Message-Based Communication**

```typescript
// Event versioning (RabbitMQ, Kafka)
// Include version in event payload
interface OrderCreatedEventV1 {
  version: '1';
  orderId: string;
  userId: string;
  total: number;
}

interface OrderCreatedEventV2 {
  version: '2';
  orderId: string;
  customer: {
    id: string;
    email: string;
  };
  items: OrderItem[];
  total: number;
}

// Publisher sends versioned events
@Injectable()
export class OrdersService {
  async createOrder(createOrderDto: CreateOrderDto) {
    const order = await this.orderRepository.save(createOrderDto);
    
    // Publish v2 event
    await this.eventBus.emit('order.created.v2', {
      version: '2',
      orderId: order.id,
      customer: {
        id: order.userId,
        email: order.userEmail,
      },
      items: order.items,
      total: order.total,
    } as OrderCreatedEventV2);
    
    // Also publish v1 event for backward compatibility
    await this.eventBus.emit('order.created.v1', {
      version: '1',
      orderId: order.id,
      userId: order.userId,
      total: order.total,
    } as OrderCreatedEventV1);
    
    return order;
  }
}

// Consumers subscribe to specific versions
@Controller()
export class InventoryController {
  // Old consumer (v1)
  @EventPattern('order.created.v1')
  async handleOrderCreatedV1(data: OrderCreatedEventV1) {
    console.log(`Processing v1 event: ${data.orderId}`);
    await this.inventoryService.reserveForOrder(data.orderId);
  }
  
  // New consumer (v2)
  @EventPattern('order.created.v2')
  async handleOrderCreatedV2(data: OrderCreatedEventV2) {
    console.log(`Processing v2 event: ${data.orderId}`);
    // Can use enhanced data (items array)
    await this.inventoryService.reserveItems(data.items);
  }
}

// Alternative: Single handler with version detection
@Controller()
export class InventoryController {
  @EventPattern('order.created')
  async handleOrderCreated(data: OrderCreatedEventV1 | OrderCreatedEventV2) {
    if (data.version === '1') {
      const eventV1 = data as OrderCreatedEventV1;
      await this.handleV1(eventV1);
    } else if (data.version === '2') {
      const eventV2 = data as OrderCreatedEventV2;
      await this.handleV2(eventV2);
    }
  }
}
```

### **Database Schema Versioning**

```typescript
// Maintain multiple schema versions
// user.entity.v1.ts
@Entity('users')
export class UserV1 {
  @PrimaryGeneratedColumn('uuid')
  id: string;
  
  @Column()
  name: string;
  
  @Column({ default: 1 })
  schemaVersion: number; // Track schema version
}

// Migration to v2: Add email field
// user.entity.v2.ts
@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;
  
  @Column()
  name: string;
  
  @Column({ nullable: true }) // Nullable for backward compatibility
  email: string;
  
  @Column({ default: 2 })
  schemaVersion: number;
}

// Migration script
export class AddEmailToUsers1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.addColumn(
      'users',
      new TableColumn({
        name: 'email',
        type: 'varchar',
        isNullable: true,
      }),
    );
    
    await queryRunner.query(
      `UPDATE users SET schema_version = 2 WHERE schema_version = 1`,
    );
  }
  
  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropColumn('users', 'email');
    await queryRunner.query(
      `UPDATE users SET schema_version = 1 WHERE schema_version = 2`,
    );
  }
}

// Service handles different schema versions
@Injectable()
export class UsersService {
  async getUser(id: string, apiVersion: string): Promise<UserDtoV1 | UserDtoV2> {
    const user = await this.userRepository.findOne(id);
    
    if (apiVersion === '1') {
      // Return v1 format (only id and name)
      return {
        id: user.id,
        name: user.name,
      };
    } else {
      // Return v2 format (includes email)
      return {
        id: user.id,
        profile: {
          name: user.name,
          email: user.email || 'N/A', // Handle missing email
        },
      };
    }
  }
}
```

### **Client-Driven Version Negotiation**

```typescript
// Server advertises supported versions
@Controller('versions')
export class VersionsController {
  @Get()
  getSupportedVersions() {
    return {
      current: '2',
      supported: ['1', '2'],
      deprecated: ['1'],
      sunset: {
        '1': '2025-12-31', // v1 will be removed on this date
      },
      latest: '2',
    };
  }
}

// Client selects version based on capabilities
@Injectable()
export class ApiClient {
  private selectedVersion: string;

  constructor(private httpService: HttpService) {}

  async initialize() {
    // Discover supported versions
    const response = await firstValueFrom(
      this.httpService.get('http://api/versions'),
    );
    
    const versions = response.data;
    
    // Select latest version supported by both client and server
    const clientSupportedVersions = ['1', '2'];
    const compatibleVersions = versions.supported.filter((v) =>
      clientSupportedVersions.includes(v),
    );
    
    this.selectedVersion = compatibleVersions[compatibleVersions.length - 1];
    console.log(`Using API version: ${this.selectedVersion}`);
  }

  async getUser(id: string) {
    return firstValueFrom(
      this.httpService.get(`http://api/v${this.selectedVersion}/users/${id}`),
    );
  }
}
```

### **Best Practices**

```typescript
// 1. Default to Latest Stable Version
@Controller('users')
export class UsersController {
  @Get(':id')
  async getUser(
    @Param('id') id: string,
    @Headers('api-version') version: string = '2', // Default to v2
  ) {
    // ...
  }
}

// 2. Document Breaking Changes
/**
 * Get user by ID
 * 
 * @version 1 - Returns flat object { id, name }
 * @version 2 - Returns nested object { id, profile: { name, email } }
 * @deprecated v1 will be removed on 2025-12-31
 */
@Get(':id')
async getUser(@Param('id') id: string) {
  // ...
}

// 3. Monitor Version Usage
@Injectable()
export class VersionMetricsInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler) {
    const request = context.switchToHttp().getRequest();
    const version = request.headers['api-version'] || 'default';
    
    // Track version usage in metrics
    this.metricsService.incrementCounter(`api_version_${version}`);
    
    return next.handle();
  }
}

// 4. Automated Testing for All Versions
describe('UsersController', () => {
  const versions = ['1', '2'];
  
  versions.forEach((version) => {
    describe(`API v${version}`, () => {
      it('should get user', async () => {
        const response = await request(app.getHttpServer())
          .get('/users/123')
          .set('api-version', version)
          .expect(200);
        
        // Version-specific assertions
        if (version === '1') {
          expect(response.body).toHaveProperty('name');
        } else if (version === '2') {
          expect(response.body).toHaveProperty('profile');
        }
      });
    });
  });
});

// 5. Gradual Rollout
@Injectable()
export class VersionRolloutService {
  canUseV2(userId: string): boolean {
    // Gradual rollout: 10% of users get v2
    const hash = createHash('md5').update(userId).digest('hex');
    const numericHash = parseInt(hash.substring(0, 8), 16);
    return (numericHash % 100) < 10; // 10% rollout
  }
}

@Controller('users')
export class UsersController {
  @Get(':id')
  async getUser(@Param('id') id: string, @User() currentUser: User) {
    const useV2 = this.rolloutService.canUseV2(currentUser.id);
    
    if (useV2) {
      return this.usersService.getUserV2(id);
    } else {
      return this.usersService.getUserV1(id);
    }
  }
}
```

### **Common Mistakes**

```typescript
// ❌ MISTAKE 1: No default version
@Get(':id')
async getUser(@Param('id') id: string, @Headers('api-version') version: string) {
  // BAD: Crashes if version header is missing!
  if (version === '1') { /* ... */ }
}

// ✅ CORRECT: Default version
@Get(':id')
async getUser(
  @Param('id') id: string,
  @Headers('api-version') version: string = '2', // Default
) {
  // ...
}

// ❌ MISTAKE 2: Abrupt version removal
// One day v1 works, next day it doesn't - breaks clients!

// ✅ CORRECT: Gradual deprecation
// 1. Add deprecation warnings (3 months before removal)
// 2. Set sunset date (RFC 8594)
// 3. Monitor usage
// 4. Remove after sunset date

// ❌ MISTAKE 3: Inconsistent versioning across services
// Order Service uses v1, v2
// Payment Service uses 1.0, 2.0
// Inventory Service uses v1.x, v2.x
// BAD: Confusing!

// ✅ CORRECT: Consistent versioning scheme
// All services use: v1, v2, v3 (or 1.0, 2.0, 3.0)

// ❌ MISTAKE 4: Versioning implementation details
// Versioning database schema, internal classes, etc.
// BAD: Implementation should be hidden

// ✅ CORRECT: Version only public API contracts
// External APIs, events, DTOs - user-facing interfaces
```

**Interview Tip**: **API versioning** allows independent service evolution while maintaining backward compatibility. **Common approaches**: URI path (`/v1/users`, `/v2/users` - explicit), header-based (`api-version: 2` - clean URLs), content negotiation (`Accept: application/json;v=2` - RESTful), query parameter (`?version=2` - simple). **Implementation in NestJS**: URI versioning with separate controllers per version (most common), single controller with version detection (header/query), schema versioning with DTOs (transform responses). **For message-based**: include version in event payload (`order.created.v1`, `order.created.v2`), consumers subscribe to specific versions. **Best practices**: use semantic versioning (MAJOR.MINOR.PATCH), default to latest stable, deprecate with warnings (Sunset header), gradual rollout (percentage-based), monitor version usage, test all versions, document breaking changes. **Breaking changes**: parallel run (both versions active), adapter pattern (v1 wraps v2), sunset old versions after grace period. **Common mistake**: no default version (crashes if version missing), abrupt removal (breaks clients), inconsistent versioning across services. **Production**: URI path versioning for HTTP APIs, event versioning for async communication, 6-12 month deprecation cycle.

</details>
50. What are the challenges of microservices (complexity, debugging, data consistency)?

<details>
<summary><strong>Answer</strong></summary>

**Microservices** introduce significant complexity compared to monolithic architectures. While they offer benefits like independent deployment and scaling, they come with challenges in distributed systems, operational overhead, debugging, data consistency, and team coordination. Understanding these challenges is crucial before adopting microservices.

### **Challenge 1: Increased Complexity**

**Problem:** Managing many services is harder than one monolith.

```typescript
// MONOLITH (Simple)
// Single application, single deployment
@Module({
  imports: [
    UsersModule,
    OrdersModule,
    PaymentsModule,
    InventoryModule,
  ],
})
export class AppModule {}

// Deploy once, everything works

// MICROSERVICES (Complex)
// Multiple applications, multiple deployments
// users-service (NestJS) → users-db (PostgreSQL)
// orders-service (NestJS) → orders-db (PostgreSQL)
// payments-service (NestJS) → payments-db (PostgreSQL)
// inventory-service (NestJS) → inventory-db (PostgreSQL)
// api-gateway (NestJS)
// message-broker (RabbitMQ)
// service-discovery (Consul)
// load-balancer (NGINX)

// Deploy 8+ components, ensure they all communicate correctly
// Manage 4 databases, 4 services, 1 gateway, 1 broker, 1 discovery service
```

**Solutions:**

```typescript
// 1. Infrastructure as Code (IaC)
// docker-compose.yml
version: '3.8'
services:
  users-service:
    build: ./services/users
    ports: ['3001:3000']
    environment:
      DATABASE_URL: postgres://users-db:5432/users
      RABBITMQ_URL: amqp://rabbitmq:5672
    depends_on: [users-db, rabbitmq]
  
  orders-service:
    build: ./services/orders
    ports: ['3002:3000']
    environment:
      DATABASE_URL: postgres://orders-db:5432/orders
      RABBITMQ_URL: amqp://rabbitmq:5672
    depends_on: [orders-db, rabbitmq]
  
  api-gateway:
    build: ./gateway
    ports: ['3000:3000']
    depends_on: [users-service, orders-service]

// 2. Service Mesh (Istio, Linkerd)
// Handles: service discovery, load balancing, retry, circuit breaking
// No code changes needed - infrastructure handles it

// 3. Orchestration (Kubernetes)
// Automates: deployment, scaling, monitoring, health checks
// kubectl apply -f k8s/
```

### **Challenge 2: Distributed Debugging**

**Problem:** Tracing errors across multiple services is difficult.

```typescript
// MONOLITH (Easy debugging)
try {
  const user = await this.usersService.getUser(id); // Line 10
  const order = await this.ordersService.createOrder(user); // Line 11
  const payment = await this.paymentsService.charge(order); // Line 12
} catch (error) {
  // Stack trace shows exact line: Line 12
  console.error(error.stack);
}

// MICROSERVICES (Hard debugging)
// Request flow: API Gateway → Users Service → Orders Service → Payments Service
// Error occurs in Payments Service, but where?
// - Network timeout?
// - Orders Service sent wrong data?
// - Payments Service has a bug?
// - Database connection failed?

// Without distributed tracing, impossible to know!
```

**Solutions:**

```typescript
// 1. Distributed Tracing (Jaeger, Zipkin)
import { Injectable } from '@nestjs/common';
import * as opentracing from 'opentracing';

@Injectable()
export class OrdersService {
  constructor(private tracer: opentracing.Tracer) {}

  async createOrder(userId: string) {
    const span = this.tracer.startSpan('create_order'); // Start trace
    span.setTag('user_id', userId);

    try {
      // Trace user service call
      const userSpan = this.tracer.startSpan('get_user', { childOf: span });
      const user = await this.usersClient.send('get.user', userId).toPromise();
      userSpan.finish();

      // Trace payment service call
      const paymentSpan = this.tracer.startSpan('charge_payment', { childOf: span });
      const payment = await this.paymentsClient.send('charge', { userId, amount: 100 }).toPromise();
      paymentSpan.finish();

      span.setTag('status', 'success');
      return { user, payment };
    } catch (error) {
      span.setTag('error', true);
      span.log({ event: 'error', message: error.message });
      throw error;
    } finally {
      span.finish();
    }
  }
}

// Jaeger UI shows:
// API Gateway (10ms) → Users Service (50ms) → Orders Service (30ms) → Payments Service (200ms) ← ERROR HERE!

// 2. Correlation IDs
@Injectable()
export class CorrelationInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler) {
    const request = context.switchToHttp().getRequest();
    const correlationId = request.headers['x-correlation-id'] || uuidv4();
    request.correlationId = correlationId;

    return next.handle().pipe(
      tap(() => {
        console.log(`[${correlationId}] Request completed`);
      }),
      catchError((error) => {
        console.error(`[${correlationId}] Request failed: ${error.message}`);
        throw error;
      }),
    );
  }
}

// All services log with correlation ID
// [abc-123] API Gateway: Received request
// [abc-123] Users Service: Getting user
// [abc-123] Orders Service: Creating order
// [abc-123] Payments Service: Error charging payment ← Find error quickly!

// 3. Centralized Logging (ELK Stack)
@Injectable()
export class LoggerService {
  log(message: string, context: string, metadata?: any) {
    const logEntry = {
      timestamp: new Date().toISOString(),
      level: 'info',
      service: 'orders-service',
      context,
      message,
      correlationId: metadata?.correlationId,
      ...metadata,
    };
    
    // Send to Elasticsearch
    this.elasticsearchClient.index({
      index: 'microservices-logs',
      body: logEntry,
    });
  }
}

// Kibana: Search all logs by correlation ID
// correlationId:"abc-123" → See all logs from all services for this request
```

### **Challenge 3: Data Consistency**

**Problem:** No ACID transactions across services.

```typescript
// MONOLITH (ACID transactions)
@Injectable()
export class OrderService {
  async createOrder(userId: string, amount: number) {
    await this.db.transaction(async (trx) => {
      // All or nothing - atomic
      await trx.orders.create({ userId, amount });
      await trx.inventory.decrement({ productId: 1 }, 'quantity', 1);
      await trx.payments.create({ userId, amount });
      // If any fails, all rollback automatically
    });
  }
}

// MICROSERVICES (No ACID across services)
@Injectable()
export class OrderService {
  async createOrder(userId: string, amount: number) {
    // 1. Create order (succeeds)
    const order = await this.orderRepo.save({ userId, amount });
    
    // 2. Decrement inventory (succeeds)
    await this.inventoryClient.send('decrement', { productId: 1 }).toPromise();
    
    // 3. Charge payment (FAILS!)
    await this.paymentsClient.send('charge', { userId, amount }).toPromise();
    
    // Problem: Order created, inventory decremented, but payment failed!
    // Inconsistent state - no automatic rollback!
  }
}
```

**Solutions:**

```typescript
// 1. Saga Pattern (Orchestration)
@Injectable()
export class OrderSagaOrchestrator {
  async executeOrderSaga(userId: string, amount: number) {
    let order;
    let inventoryReserved = false;
    let paymentCharged = false;

    try {
      // Step 1: Create order
      order = await this.orderRepo.save({ userId, amount, status: 'pending' });

      // Step 2: Reserve inventory
      await this.inventoryClient.send('reserve', { orderId: order.id }).toPromise();
      inventoryReserved = true;

      // Step 3: Charge payment
      await this.paymentsClient.send('charge', { orderId: order.id, amount }).toPromise();
      paymentCharged = true;

      // Success
      await this.orderRepo.update(order.id, { status: 'completed' });
      return order;
    } catch (error) {
      // Compensating transactions (rollback)
      if (paymentCharged) {
        await this.paymentsClient.send('refund', { orderId: order.id }).toPromise();
      }
      if (inventoryReserved) {
        await this.inventoryClient.send('release', { orderId: order.id }).toPromise();
      }
      if (order) {
        await this.orderRepo.update(order.id, { status: 'failed' });
      }
      throw error;
    }
  }
}

// 2. Eventual Consistency
// Accept temporary inconsistency, converge eventually
@Injectable()
export class OrderService {
  async createOrder(userId: string, amount: number) {
    const order = await this.orderRepo.save({ userId, amount, status: 'pending' });
    
    // Emit events asynchronously
    await this.eventBus.emit('order.created', { orderId: order.id });
    
    return order; // Return immediately, don't wait for other services
  }
}

// Other services eventually process events
@Controller()
export class InventoryController {
  @EventPattern('order.created')
  async handleOrderCreated(data: { orderId: string }) {
    await this.inventoryService.reserveForOrder(data.orderId);
    await this.eventBus.emit('inventory.reserved', { orderId: data.orderId });
  }
}

// 3. Two-Phase Commit (2PC) - Rarely used
// Coordinator asks all services to prepare, then commit
// Slow and blocking - not recommended for microservices
```

### **Challenge 4: Network Failures**

**Problem:** Network is unreliable - calls can fail, timeout, or hang.

```typescript
// MONOLITH (No network calls)
const user = this.usersService.getUser(id); // In-memory call, never fails

// MICROSERVICES (Network calls everywhere)
const user = await this.usersClient.send('get.user', id).toPromise();
// Can fail due to:
// - Network partition
// - Service crashed
// - Service overloaded
// - DNS resolution failed
// - Timeout
```

**Solutions:**

```typescript
// 1. Retry with Exponential Backoff
import { retry, catchError } from 'rxjs/operators';

@Injectable()
export class ResilientClient {
  getUser(id: string) {
    return this.usersClient.send('get.user', id).pipe(
      retry({
        count: 3,
        delay: (retryCount) => timer(Math.pow(2, retryCount) * 1000), // 1s, 2s, 4s
      }),
      catchError((error) => {
        console.error('Failed after 3 retries:', error);
        throw new ServiceUnavailableException('Users service unavailable');
      }),
    );
  }
}

// 2. Circuit Breaker
import { CircuitBreaker } from 'opossum';

@Injectable()
export class CircuitBreakerClient {
  private breaker: CircuitBreaker;

  constructor() {
    this.breaker = new CircuitBreaker(this.callService.bind(this), {
      timeout: 3000, // 3 seconds
      errorThresholdPercentage: 50, // Open circuit if 50% fail
      resetTimeout: 30000, // Try again after 30 seconds
    });
  }

  async getUser(id: string) {
    return this.breaker.fire(id); // Automatically opens circuit if failing
  }

  private async callService(id: string) {
    return this.usersClient.send('get.user', id).toPromise();
  }
}

// 3. Timeout Handling
@Injectable()
export class TimeoutClient {
  getUser(id: string) {
    return this.usersClient.send('get.user', id).pipe(
      timeout(5000), // Fail after 5 seconds
      catchError((error) => {
        if (error.name === 'TimeoutError') {
          throw new RequestTimeoutException('Users service timed out');
        }
        throw error;
      }),
    );
  }
}

// 4. Fallback Strategies
@Injectable()
export class FallbackClient {
  async getUser(id: string) {
    try {
      return await this.usersClient.send('get.user', id).toPromise();
    } catch (error) {
      // Fallback to cache
      const cachedUser = await this.cache.get(`user:${id}`);
      if (cachedUser) {
        return cachedUser;
      }
      
      // Fallback to default
      return { id, name: 'Unknown User', email: 'N/A' };
    }
  }
}
```

### **Challenge 5: Testing Complexity**

**Problem:** Testing microservices requires running multiple services.

```typescript
// MONOLITH (Easy testing)
describe('OrderService', () => {
  it('should create order', async () => {
    const order = await orderService.createOrder(userId, 100);
    expect(order).toBeDefined();
    expect(order.status).toBe('completed');
  });
});

// MICROSERVICES (Complex testing)
// Need to mock: users-service, inventory-service, payments-service, message-broker
describe('OrderService', () => {
  let usersClient: ClientProxy;
  let inventoryClient: ClientProxy;
  let paymentsClient: ClientProxy;

  beforeEach(() => {
    // Mock all external services
    usersClient = { send: jest.fn() } as any;
    inventoryClient = { send: jest.fn() } as any;
    paymentsClient = { send: jest.fn() } as any;
    // Complex setup!
  });

  it('should create order', async () => {
    // Mock responses from all services
    (usersClient.send as jest.Mock).mockReturnValue(of({ id: userId, name: 'John' }));
    (inventoryClient.send as jest.Mock).mockReturnValue(of({ reserved: true }));
    (paymentsClient.send as jest.Mock).mockReturnValue(of({ charged: true }));

    const order = await orderService.createOrder(userId, 100);
    expect(order).toBeDefined();
  });
});
```

**Solutions:**

```typescript
// 1. Contract Testing (Pact)
// Define contracts between services
describe('Users Service Contract', () => {
  it('should return user with expected structure', async () => {
    const provider = new Pact({
      consumer: 'orders-service',
      provider: 'users-service',
    });

    await provider.addInteraction({
      state: 'user exists',
      uponReceiving: 'a request for user',
      withRequest: {
        method: 'GET',
        path: '/users/123',
      },
      willRespondWith: {
        status: 200,
        body: {
          id: '123',
          name: 'John',
        },
      },
    });

    const user = await ordersService.getUser('123');
    expect(user).toEqual({ id: '123', name: 'John' });
  });
});

// 2. Test Containers (Docker)
import { GenericContainer } from 'testcontainers';

describe('Integration Tests', () => {
  let rabbitmqContainer: GenericContainer;
  let postgresContainer: GenericContainer;

  beforeAll(async () => {
    // Start real services in Docker
    rabbitmqContainer = await new GenericContainer('rabbitmq:3-management')
      .withExposedPorts(5672)
      .start();

    postgresContainer = await new GenericContainer('postgres:14')
      .withExposedPorts(5432)
      .start();
  });

  it('should create order with real dependencies', async () => {
    // Test against real RabbitMQ and PostgreSQL
  });

  afterAll(async () => {
    await rabbitmqContainer.stop();
    await postgresContainer.stop();
  });
});

// 3. Component Testing (Test one service in isolation)
// Mock only external services, use real database
describe('OrderService Component Test', () => {
  let app: INestApplication;

  beforeAll(async () => {
    const moduleRef = await Test.createTestingModule({
      imports: [OrderModule, DatabaseModule],
      providers: [
        {
          provide: 'USERS_SERVICE',
          useValue: mockUsersClient, // Mock external service
        },
      ],
    }).compile();

    app = moduleRef.createNestApplication();
    await app.init();
  });

  it('should create order', async () => {
    const response = await request(app.getHttpServer())
      .post('/orders')
      .send({ userId: '123', amount: 100 })
      .expect(201);

    expect(response.body.id).toBeDefined();
  });
});
```

### **Challenge 6: Operational Overhead**

**Problem:** Managing deployments, monitoring, logs for many services.

```typescript
// MONOLITH
// 1 deployment
// 1 log file
// 1 monitoring dashboard
// 1 database to backup

// MICROSERVICES
// 10 deployments (rolling updates, canary deployments)
// 10 log files (centralized logging needed)
// 10 monitoring dashboards (aggregated metrics needed)
// 10 databases to backup (backup orchestration needed)
// 10 health checks (uptime monitoring)
// 10 security patches (dependency management)
```

**Solutions:**

```typescript
// 1. CI/CD Pipeline (GitHub Actions)
// .github/workflows/deploy.yml
name: Deploy Microservices
on:
  push:
    branches: [main]

jobs:
  deploy-users-service:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Build Docker image
        run: docker build -t users-service ./services/users
      - name: Deploy to Kubernetes
        run: kubectl apply -f k8s/users-service.yml

  deploy-orders-service:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Build Docker image
        run: docker build -t orders-service ./services/orders
      - name: Deploy to Kubernetes
        run: kubectl apply -f k8s/orders-service.yml

// 2. Centralized Monitoring (Prometheus + Grafana)
@Injectable()
export class MetricsService {
  private requestCounter = new Counter({
    name: 'http_requests_total',
    help: 'Total HTTP requests',
    labelNames: ['service', 'endpoint', 'status'],
  });

  incrementRequest(endpoint: string, status: number) {
    this.requestCounter.inc({
      service: 'orders-service',
      endpoint,
      status: status.toString(),
    });
  }
}

// Grafana dashboard shows metrics from ALL services

// 3. Health Checks
@Controller('health')
export class HealthController {
  @Get()
  async check() {
    const dbHealthy = await this.checkDatabase();
    const rabbitHealthy = await this.checkRabbitMQ();

    if (!dbHealthy || !rabbitHealthy) {
      throw new ServiceUnavailableException('Service unhealthy');
    }

    return { status: 'ok', timestamp: new Date() };
  }
}

// Kubernetes automatically restarts unhealthy services
```

### **Challenge 7: Team Coordination**

**Problem:** Teams need to coordinate API changes, deployments.

```typescript
// Orders team wants to change user data structure
// But users team hasn't deployed the change yet
// Result: Breaking change!

// Orders Service (deployed first) - BREAKS!
interface User {
  id: string;
  profile: { // NEW structure
    name: string;
    email: string;
  };
}

// Users Service (not deployed yet) - OLD structure
interface User {
  id: string;
  name: string; // OLD structure
}
```

**Solutions:**

```typescript
// 1. API Versioning
// Users Service supports both v1 and v2
@Get('v1/users/:id')
getUserV1() {
  return { id, name }; // Old format
}

@Get('v2/users/:id')
getUserV2() {
  return { id, profile: { name, email } }; // New format
}

// Orders Service can upgrade at their own pace

// 2. Backward Compatibility
interface User {
  id: string;
  name: string; // Keep old field
  email?: string; // Add optional new field
  profile?: { // Add optional new structure
    name: string;
    email: string;
  };
}

// 3. Consumer-Driven Contracts
// Orders team defines what they need from Users service
// Users team implements to satisfy the contract
// Automated tests verify contract is met

// 4. Service Ownership
// Clear ownership: Users team owns Users service API
// Any changes must be communicated and versioned
// Documentation is mandatory
```

### **When to Use Microservices**

```typescript
// ✅ USE MICROSERVICES when:
// - Large team (>20 developers)
// - Clear domain boundaries
// - Independent scaling requirements
// - Different technology stacks needed
// - Frequent deployments (multiple times per day)
// - Organizational maturity (DevOps, CI/CD, monitoring)

// ❌ AVOID MICROSERVICES when:
// - Small team (<5 developers)
// - Unclear domain boundaries
// - Simple application
// - Limited resources (no DevOps expertise)
// - Startup (MVP phase)
// - Tight coupling between components

// Consider MODULAR MONOLITH first:
// - Modules can be extracted later
// - Simpler operations
// - Easier debugging
// - Can scale to microservices when needed
```

### **Best Practices to Mitigate Challenges**

```typescript
// 1. Start with Monolith, extract services later
// Build modular monolith → Identify bounded contexts → Extract services

// 2. Invest in Infrastructure
// - Service mesh (Istio)
// - Centralized logging (ELK)
// - Distributed tracing (Jaeger)
// - Monitoring (Prometheus)
// - CI/CD pipelines

// 3. Design for Failure
// - Circuit breakers
// - Retries with backoff
// - Timeouts
// - Fallbacks
// - Health checks

// 4. Establish Standards
// - API versioning strategy
// - Error handling
// - Logging format
// - Authentication/authorization
// - Service naming

// 5. Automate Everything
// - Deployments (CI/CD)
// - Testing (contract tests)
// - Monitoring (alerts)
// - Scaling (auto-scaling)

// 6. Documentation
// - API documentation (OpenAPI/Swagger)
// - Architecture diagrams
// - Runbooks for incidents
// - Decision logs (ADRs)
```

**Interview Tip**: **Microservices challenges**: increased **complexity** (managing many services vs one monolith), difficult **debugging** (errors span multiple services - need distributed tracing/Jaeger), **data consistency** (no ACID across services - need Saga pattern/eventual consistency), **network failures** (unreliable network - need retries/circuit breakers/timeouts), **testing complexity** (need contract testing/test containers), **operational overhead** (many deployments/logs/monitoring dashboards), **team coordination** (API changes require versioning/backward compatibility). **Solutions**: distributed tracing (Jaeger with correlation IDs), Saga pattern for transactions (orchestration with compensating actions), retry with exponential backoff, circuit breaker (close/open/half-open states), centralized logging (ELK), service mesh (Istio for automatic resilience), CI/CD pipelines, contract testing (Pact), API versioning. **Key mistake**: premature microservices (start with modular monolith, extract services later when boundaries are clear). **Production**: invest in infrastructure first (service mesh, monitoring, logging), design for failure (retries, fallbacks), automate everything (CI/CD, scaling, alerts). **When to avoid**: small teams (<5 devs), unclear boundaries, limited DevOps expertise, MVP phase.

</details>
