# NestJS Logging & Monitoring - Top Interview Questions

## Logging Fundamentals

<details>
<summary><strong>1. Why is logging important in applications?</strong></summary>

**Answer:**

Logging is a critical practice in production applications that enables developers and operations teams to understand application behavior, troubleshoot issues, and maintain system health.

**Key Reasons for Logging:**

1. **Debugging & Troubleshooting**
   - Identify root causes of errors and bugs
   - Trace execution flow through complex systems
   - Reproduce issues that occur in production

2. **Monitoring & Observability**
   - Track application health and performance
   - Detect anomalies and unusual patterns
   - Monitor system resource usage

3. **Security & Compliance**
   - Audit user actions and access patterns
   - Detect security breaches or suspicious activities
   - Meet regulatory requirements (GDPR, HIPAA, SOX)

4. **Performance Analysis**
   - Identify bottlenecks and slow operations
   - Track API response times and database queries
   - Optimize system performance based on real data

5. **Business Intelligence**
   - Understand user behavior and usage patterns
   - Track feature adoption and user engagement
   - Make data-driven business decisions

**Production Example:**

```typescript
import { Injectable, Logger } from '@nestjs/common';

@Injectable()
export class PaymentService {
  private readonly logger = new Logger(PaymentService.name);

  async processPayment(orderId: string, amount: number) {
    // Log business-critical operations
    this.logger.log(`Processing payment for order ${orderId}, amount: $${amount}`);
    
    try {
      const result = await this.chargeCard(amount);
      
      // Log successful operations with relevant details
      this.logger.log(
        `Payment successful for order ${orderId}. Transaction ID: ${result.transactionId}`
      );
      
      return result;
    } catch (error) {
      // Log errors with full context for troubleshooting
      this.logger.error(
        `Payment failed for order ${orderId}, amount: $${amount}`,
        error.stack
      );
      
      // In production, this helps quickly identify payment issues
      throw error;
    }
  }
}
```

**Benefits in Production:**

```typescript
// Without logs: "Payment failed" - No context!
// With logs: 
// [ERROR] Payment failed for order ORD-12345, amount: $99.99
// Stack trace: CardDeclinedException at PaymentGateway.charge...
// Timestamp: 2025-12-30T10:30:45.123Z
// Request ID: req-abc-123
// User ID: user-456

// This enables:
// - Quick identification of the problem
// - Contact customer with specific details
// - Track patterns (multiple declined cards?)
// - Business metrics (failure rate, revenue impact)
```

**What NOT to Log:**

```typescript
// ❌ BAD - Sensitive information
this.logger.log(`Processing payment with card: ${cardNumber}`); // PCI violation!
this.logger.log(`User password: ${password}`); // Security risk!
this.logger.log(`API Key: ${apiKey}`); // Credential exposure!

// ✅ GOOD - Masked or omitted sensitive data
this.logger.log(`Processing payment with card ending: ****${cardNumber.slice(-4)}`);
this.logger.log(`User authenticated successfully`); // No password logged
this.logger.log(`External API call initiated`); // No API key logged
```

**Real-World Impact:**

- **Downtime Reduction**: Logs help identify issues 10x faster than without
- **Cost Savings**: Quick issue resolution reduces revenue loss
- **Customer Satisfaction**: Proactive monitoring prevents user-facing errors
- **Compliance**: Audit logs protect against legal issues ($millions in fines)

</details>

<details>
<summary><strong>2. What are the different log levels (error, warn, info, debug, verbose)?</strong></summary>

**Answer:**

Log levels categorize log messages by severity and importance, helping filter and prioritize information. NestJS supports 5 standard log levels aligned with industry practices.

**Log Levels (Highest to Lowest Severity):**

### 1. **ERROR** - Critical Failures
Application errors that require immediate attention.

```typescript
import { Injectable, Logger } from '@nestjs/common';

@Injectable()
export class OrderService {
  private readonly logger = new Logger(OrderService.name);

  async createOrder(orderData: CreateOrderDto) {
    try {
      // Business logic
      return await this.orderRepository.save(orderData);
    } catch (error) {
      // Use ERROR for exceptions and failures
      this.logger.error(
        `Failed to create order for user ${orderData.userId}`,
        error.stack, // Include stack trace
        'OrderService' // Context
      );
      throw error;
    }
  }
}
```

**When to use ERROR:**
- Exceptions and unhandled errors
- Failed database operations
- External API failures
- Critical business logic failures

---

### 2. **WARN** - Potential Issues
Unusual situations that don't stop execution but need attention.

```typescript
@Injectable()
export class CacheService {
  private readonly logger = new Logger(CacheService.name);

  async get(key: string) {
    try {
      return await this.redis.get(key);
    } catch (error) {
      // Use WARN for degraded functionality
      this.logger.warn(
        `Cache unavailable, falling back to database. Key: ${key}`
      );
      // Application continues but with reduced performance
      return this.database.get(key);
    }
  }

  checkCacheSize() {
    const size = this.getCacheSize();
    if (size > this.THRESHOLD) {
      // Use WARN for approaching limits
      this.logger.warn(
        `Cache size approaching limit: ${size}MB / ${this.MAX_SIZE}MB`
      );
    }
  }
}
```

**When to use WARN:**
- Deprecated API usage
- Approaching resource limits (memory, disk space)
- Fallback mechanisms triggered
- Configuration issues (non-blocking)
- Unusual user behavior

---

### 3. **LOG / INFO** - Important Events
Normal but significant events in application flow.

```typescript
@Injectable()
export class AuthService {
  private readonly logger = new Logger(AuthService.name);

  async login(email: string, password: string) {
    // Use LOG for important business events
    this.logger.log(`User login attempt: ${email}`);
    
    const user = await this.validateCredentials(email, password);
    
    if (user) {
      // Track successful operations
      this.logger.log(`User logged in successfully: ${user.id}`);
      return this.generateToken(user);
    }
    
    // Log security-relevant failures
    this.logger.log(`Failed login attempt for: ${email}`);
    throw new UnauthorizedException();
  }
}

@Injectable()
export class AppService {
  private readonly logger = new Logger(AppService.name);

  onModuleInit() {
    // Use LOG for application lifecycle events
    this.logger.log('Application initialized successfully');
    this.logger.log(`Environment: ${process.env.NODE_ENV}`);
    this.logger.log(`Database connected: ${this.dbConfig.host}`);
  }
}
```

**When to use LOG/INFO:**
- Application startup/shutdown
- User authentication events
- Important state changes
- Completed background jobs
- External service calls

---

### 4. **DEBUG** - Detailed Diagnostic Information
Detailed information useful for debugging during development.

```typescript
@Injectable()
export class ProductService {
  private readonly logger = new Logger(ProductService.name);

  async getProduct(id: string) {
    // Use DEBUG for detailed execution flow
    this.logger.debug(`Fetching product with ID: ${id}`);
    
    const cached = await this.cache.get(id);
    if (cached) {
      this.logger.debug(`Product found in cache: ${id}`);
      return cached;
    }
    
    this.logger.debug(`Cache miss, querying database for product: ${id}`);
    const product = await this.repository.findOne(id);
    
    this.logger.debug(`Product retrieved, updating cache: ${id}`);
    await this.cache.set(id, product);
    
    return product;
  }
}
```

**When to use DEBUG:**
- Function entry/exit points
- Variable states and values
- Conditional branch execution
- Loop iterations (be careful with volume!)
- Cache hits/misses
- Query parameters

---

### 5. **VERBOSE** - Everything
Most detailed logging, rarely used in production.

```typescript
@Injectable()
export class DataSyncService {
  private readonly logger = new Logger(DataSyncService.name);

  async syncData(items: Item[]) {
    // Use VERBOSE for extremely detailed logs
    this.logger.verbose(`Starting sync for ${items.length} items`);
    
    for (const item of items) {
      this.logger.verbose(`Processing item: ${JSON.stringify(item)}`);
      this.logger.verbose(`Validating item ${item.id}`);
      // ... validation logic
      this.logger.verbose(`Validation passed for item ${item.id}`);
      this.logger.verbose(`Saving item ${item.id} to database`);
      // ... save logic
      this.logger.verbose(`Item ${item.id} saved successfully`);
    }
    
    this.logger.verbose('Sync completed');
  }
}
```

**When to use VERBOSE:**
- Deep debugging of complex issues
- Tracing every step of execution
- Development and testing environments
- Rarely in production (high volume!)

---

**Comparison Table:**

| Level | Severity | Production? | Use Case | Example |
|-------|----------|-------------|----------|---------|
| **ERROR** | Highest | ✅ Always | Failures requiring action | Payment failed, DB connection lost |
| **WARN** | High | ✅ Always | Potential problems | Cache miss, deprecated API used |
| **LOG/INFO** | Normal | ✅ Always | Important events | User login, job completed |
| **DEBUG** | Low | ⚠️ Selective | Development debugging | Function calls, variable values |
| **VERBOSE** | Lowest | ❌ Rarely | Deep troubleshooting | Every step traced |

---

**Production Configuration Example:**

```typescript
// main.ts
import { Logger } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, {
    // Production: Only ERROR, WARN, LOG
    logger: process.env.NODE_ENV === 'production' 
      ? ['error', 'warn', 'log']
      : ['error', 'warn', 'log', 'debug', 'verbose'], // Development: All levels
  });

  const logger = new Logger('Bootstrap');
  
  await app.listen(3000);
  logger.log(`Application running on port 3000`);
  logger.log(`Log levels: ${process.env.NODE_ENV === 'production' ? 'ERROR, WARN, LOG' : 'ALL'}`);
}
bootstrap();
```

**Best Practices:**

```typescript
// ✅ GOOD - Appropriate level usage
this.logger.error('Payment processing failed', error.stack);     // Critical failure
this.logger.warn('Rate limit approaching for user 123');         // Potential issue
this.logger.log('Order #456 placed successfully');               // Important event
this.logger.debug('Cache lookup for key: product-789');          // Debugging info
this.logger.verbose('Request body: ' + JSON.stringify(body));    // Detailed trace

// ❌ BAD - Inappropriate level usage
this.logger.error('User clicked button');              // Not an error!
this.logger.log(error.stack);                          // Should be ERROR
this.logger.debug('Payment failed!');                  // Should be ERROR
this.logger.verbose('User login');                     // Should be LOG
```

**Performance Impact:**

```typescript
// Each log level adds overhead
// In high-traffic production:
// - ERROR, WARN, LOG: ~0.1ms each (acceptable)
// - DEBUG: 10-100x more logs (impacts performance)
// - VERBOSE: 100-1000x more logs (can crash application!)

// Monitor log volume:
// Production: ~1000 logs/second = OK
// Debug mode: ~100,000 logs/second = Problem!
```

</details>
<details>
<summary><strong>3. When should you use each log level?</strong></summary>

**Answer:**

Choosing the correct log level is crucial for effective logging. Here's a comprehensive guide with production examples for when to use each level.

---

### **ERROR Level - Use When:**

**1. Exceptions and Unhandled Errors**
```typescript
@Injectable()
export class PaymentService {
  private readonly logger = new Logger(PaymentService.name);

  async processPayment(order: Order) {
    try {
      return await this.paymentGateway.charge(order);
    } catch (error) {
      // ✅ ERROR: Transaction failed - requires immediate action
      this.logger.error(
        `Payment processing failed for order ${order.id}`,
        error.stack
      );
      throw error;
    }
  }
}
```

**2. Critical System Failures**
```typescript
async onModuleInit() {
  try {
    await this.database.connect();
  } catch (error) {
    // ✅ ERROR: Cannot operate without database
    this.logger.error('Failed to connect to database', error.stack);
    process.exit(1); // Critical failure, cannot continue
  }
}
```

**3. Data Integrity Issues**
```typescript
async validateOrder(order: Order) {
  const inventory = await this.checkInventory(order.productId);
  
  if (inventory < order.quantity) {
    // ✅ ERROR: Business rule violation - overselling
    this.logger.error(
      `Inventory insufficient for order ${order.id}. ` +
      `Requested: ${order.quantity}, Available: ${inventory}`
    );
    throw new InsufficientInventoryException();
  }
}
```

---

### **WARN Level - Use When:**

**1. Degraded Performance / Fallback**
```typescript
async getProductFromCache(id: string) {
  try {
    return await this.redisCache.get(id);
  } catch (error) {
    // ✅ WARN: Cache failure, but application continues
    this.logger.warn(
      `Redis cache unavailable, falling back to database for product ${id}`
    );
    return await this.database.getProduct(id);
  }
}
```

**2. Approaching Limits**
```typescript
async checkSystemHealth() {
  const memoryUsage = process.memoryUsage().heapUsed / 1024 / 1024;
  
  if (memoryUsage > 800) { // 800 MB threshold
    // ✅ WARN: Memory usage high but not critical yet
    this.logger.warn(
      `High memory usage detected: ${memoryUsage.toFixed(2)} MB`
    );
  }
}
```

**3. Deprecated Features**
```typescript
@Get('/old-endpoint')
getOldData() {
  // ✅ WARN: Deprecated API usage
  this.logger.warn(
    'Deprecated endpoint /old-endpoint called. ' +
    'Please use /v2/data instead'
  );
  return this.legacyService.getData();
}
```

**4. Configuration Issues (Non-Fatal)**
```typescript
constructor(private configService: ConfigService) {
  const timeout = this.configService.get('API_TIMEOUT');
  
  if (!timeout) {
    // ✅ WARN: Using default, but should be configured
    this.logger.warn(
      'API_TIMEOUT not configured, using default: 5000ms'
    );
  }
}
```

**5. Suspicious Activity**
```typescript
async login(credentials: LoginDto) {
  const failedAttempts = await this.getFailedAttempts(credentials.email);
  
  if (failedAttempts >= 3) {
    // ✅ WARN: Potential security issue
    this.logger.warn(
      `Multiple failed login attempts detected for ${credentials.email}. ` +
      `Count: ${failedAttempts}`
    );
  }
}
```

---

### **LOG (INFO) Level - Use When:**

**1. Important Business Events**
```typescript
async placeOrder(order: CreateOrderDto) {
  const savedOrder = await this.orderRepository.save(order);
  
  // ✅ LOG: Significant business event
  this.logger.log(
    `Order placed successfully. ` +
    `OrderID: ${savedOrder.id}, User: ${order.userId}, ` +
    `Amount: $${order.totalAmount}`
  );
  
  return savedOrder;
}
```

**2. Authentication & Authorization Events**
```typescript
async login(email: string, password: string) {
  const user = await this.validateUser(email, password);
  
  // ✅ LOG: Security-relevant event
  this.logger.log(`User logged in: ${user.id} (${email})`);
  
  return this.generateToken(user);
}

async logout(userId: string) {
  // ✅ LOG: Session ended
  this.logger.log(`User logged out: ${userId}`);
}
```

**3. Application Lifecycle Events**
```typescript
async onModuleInit() {
  // ✅ LOG: Application state changes
  this.logger.log('PaymentService initialized');
  this.logger.log(`Connected to payment gateway: ${this.gateway.name}`);
}

async onModuleDestroy() {
  // ✅ LOG: Graceful shutdown
  this.logger.log('PaymentService shutting down');
}
```

**4. External Service Calls**
```typescript
async sendEmail(to: string, subject: string) {
  // ✅ LOG: External integration event
  this.logger.log(`Sending email to ${to}: "${subject}"`);
  
  await this.emailService.send({ to, subject });
  
  this.logger.log(`Email sent successfully to ${to}`);
}
```

**5. Background Job Completion**
```typescript
@Cron('0 0 * * *') // Daily at midnight
async generateDailyReport() {
  // ✅ LOG: Job started
  this.logger.log('Starting daily report generation');
  
  const report = await this.createReport();
  
  // ✅ LOG: Job completed
  this.logger.log(
    `Daily report generated successfully. ` +
    `Orders: ${report.orderCount}, Revenue: $${report.revenue}`
  );
}
```

---

### **DEBUG Level - Use When:**

**1. Function Entry/Exit**
```typescript
async getUser(id: string) {
  // ✅ DEBUG: Trace execution flow
  this.logger.debug(`getUser() called with id: ${id}`);
  
  const user = await this.userRepository.findOne(id);
  
  this.logger.debug(`getUser() returning user: ${user ? 'found' : 'not found'}`);
  return user;
}
```

**2. Conditional Logic Branches**
```typescript
async calculateDiscount(order: Order) {
  this.logger.debug(`Calculating discount for order ${order.id}`);
  
  if (order.totalAmount > 1000) {
    this.logger.debug('Applying bulk order discount');
    return order.totalAmount * 0.1;
  } else if (order.isFirstOrder) {
    this.logger.debug('Applying first-time customer discount');
    return order.totalAmount * 0.05;
  } else {
    this.logger.debug('No discount applicable');
    return 0;
  }
}
```

**3. Cache Operations**
```typescript
async getProduct(id: string) {
  this.logger.debug(`Checking cache for product ${id}`);
  
  const cached = await this.cache.get(id);
  if (cached) {
    this.logger.debug(`Cache HIT for product ${id}`);
    return cached;
  }
  
  this.logger.debug(`Cache MISS for product ${id}, fetching from DB`);
  const product = await this.repository.findOne(id);
  
  this.logger.debug(`Storing product ${id} in cache`);
  await this.cache.set(id, product);
  
  return product;
}
```

**4. Query Execution Details**
```typescript
async searchProducts(query: string) {
  this.logger.debug(`Building search query for: "${query}"`);
  
  const queryBuilder = this.repository
    .createQueryBuilder('product')
    .where('product.name LIKE :query', { query: `%${query}%` });
  
  this.logger.debug(`Executing query: ${queryBuilder.getQuery()}`);
  
  const results = await queryBuilder.getMany();
  
  this.logger.debug(`Query returned ${results.length} results`);
  return results;
}
```

**5. State Changes**
```typescript
async processOrderStatus(orderId: string, newStatus: OrderStatus) {
  const order = await this.getOrder(orderId);
  
  // ✅ DEBUG: State transition details
  this.logger.debug(
    `Changing order ${orderId} status: ` +
    `${order.status} -> ${newStatus}`
  );
  
  order.status = newStatus;
  await this.orderRepository.save(order);
}
```

---

### **VERBOSE Level - Use When:**

**1. Deep Debugging Complex Issues**
```typescript
async syncInventory(products: Product[]) {
  this.logger.verbose(`Starting inventory sync for ${products.length} products`);
  
  for (const product of products) {
    this.logger.verbose(`Processing product: ${JSON.stringify(product)}`);
    
    this.logger.verbose(`Validating product ${product.id}`);
    const isValid = await this.validateProduct(product);
    this.logger.verbose(`Validation result for ${product.id}: ${isValid}`);
    
    if (isValid) {
      this.logger.verbose(`Updating inventory for ${product.id}`);
      await this.updateInventory(product);
      this.logger.verbose(`Inventory updated for ${product.id}`);
    } else {
      this.logger.verbose(`Skipping invalid product ${product.id}`);
    }
  }
  
  this.logger.verbose('Inventory sync completed');
}
```

**2. Request/Response Payloads (Development Only)**
```typescript
@Post('/orders')
async createOrder(@Body() dto: CreateOrderDto) {
  // ✅ VERBOSE: Full request details (dev only!)
  this.logger.verbose(`Received order request: ${JSON.stringify(dto)}`);
  
  const order = await this.orderService.create(dto);
  
  this.logger.verbose(`Order response: ${JSON.stringify(order)}`);
  return order;
}
```

**3. Loop Iterations (Sparingly!)**
```typescript
async batchProcess(items: Item[]) {
  this.logger.verbose(`Batch processing ${items.length} items`);
  
  for (let i = 0; i < items.length; i++) {
    // ⚠️ VERBOSE: Can generate HUGE logs!
    this.logger.verbose(`Processing item ${i + 1}/${items.length}: ${items[i].id}`);
    await this.processItem(items[i]);
  }
}
```

---

### **Decision Matrix:**

| Scenario | Level | Reason |
|----------|-------|--------|
| Database connection failed | ERROR | Critical, cannot operate |
| Payment declined by gateway | ERROR | Transaction failed |
| Cache unavailable, using DB | WARN | Degraded but operational |
| Memory usage at 85% | WARN | Approaching threshold |
| User logged in | LOG | Important audit event |
| Order placed | LOG | Business event |
| Daily report completed | LOG | Background job finished |
| Function called with params | DEBUG | Development debugging |
| Cache hit/miss | DEBUG | Performance analysis |
| Validating each field | VERBOSE | Deep troubleshooting |
| Full request body | VERBOSE | Detailed diagnostics |

---

### **Environment-Specific Configuration:**

```typescript
// main.ts
const logLevels = {
  production: ['error', 'warn', 'log'],           // Minimal in prod
  staging: ['error', 'warn', 'log', 'debug'],     // More detail in staging
  development: ['error', 'warn', 'log', 'debug', 'verbose'], // Everything in dev
};

const app = await NestFactory.create(AppModule, {
  logger: logLevels[process.env.NODE_ENV] || logLevels.development,
});
```

---

### **Anti-Patterns to Avoid:**

```typescript
// ❌ BAD: Wrong level usage
this.logger.error('User viewed product page');     // Not an error!
this.logger.log(exception.stack);                  // Should be ERROR
this.logger.debug('Payment processing failed!');   // Should be ERROR
this.logger.warn('Application started');           // Should be LOG
this.logger.verbose('User login');                 // Should be LOG

// ✅ GOOD: Appropriate levels
this.logger.log('User viewed product page');
this.logger.error('Payment processing failed', exception.stack);
this.logger.error('Payment processing failed', exception.stack);
this.logger.log('Application started');
this.logger.log('User logged in');
```

---

### **Production Best Practice:**

```typescript
@Injectable()
export class UserService {
  private readonly logger = new Logger(UserService.name);

  async createUser(dto: CreateUserDto) {
    try {
      // Use DEBUG for detailed flow (dev only)
      this.logger.debug(`Creating user: ${dto.email}`);
      
      const user = await this.userRepository.save(dto);
      
      // Use LOG for important business event
      this.logger.log(`User created successfully: ${user.id}`);
      
      return user;
    } catch (error) {
      // Check error type for appropriate level
      if (error.code === 'DUPLICATE_EMAIL') {
        // WARN: Expected failure, not critical
        this.logger.warn(`Duplicate email attempted: ${dto.email}`);
        throw new ConflictException('Email already exists');
      } else {
        // ERROR: Unexpected failure
        this.logger.error(
          `Failed to create user for ${dto.email}`,
          error.stack
        );
        throw error;
      }
    }
  }
}
```

**Key Takeaway**: Choose log levels based on **severity** and **action required**, not just for categorization!

</details>

<details>
<summary><strong>4. What is the built-in Logger in NestJS?</strong></summary>

**Answer:**

NestJS provides a built-in `Logger` class that offers structured, context-aware logging out of the box. It's production-ready and follows best practices for application logging.

---

### **Core Features:**

1. **Multiple log levels** (error, warn, log, debug, verbose)
2. **Contextual logging** (identify source of logs)
3. **Timestamp support** (track when events occur)
4. **Colorized output** (visual distinction in console)
5. **Stacktrace support** (for errors)
6. **Global configuration** (control log levels across app)

---

### **Basic Usage:**

```typescript
import { Logger, Injectable } from '@nestjs/common';

@Injectable()
export class UserService {
  // Create logger instance with context
  private readonly logger = new Logger(UserService.name);

  async findUser(id: string) {
    // Use various log levels
    this.logger.log(`Searching for user with ID: ${id}`);
    this.logger.debug(`Database query parameters: ${JSON.stringify({ id })}`);
    
    try {
      const user = await this.userRepository.findOne(id);
      
      if (!user) {
        this.logger.warn(`User not found: ${id}`);
        return null;
      }
      
      this.logger.log(`User found: ${user.email}`);
      return user;
    } catch (error) {
      this.logger.error(
        `Error finding user ${id}`,
        error.stack,  // Stack trace
        'UserService' // Optional context override
      );
      throw error;
    }
  }
}
```

**Output Example:**
```
[Nest] 12345  - 12/30/2025, 10:30:45 AM     LOG [UserService] Searching for user with ID: 123
[Nest] 12345  - 12/30/2025, 10:30:45 AM   DEBUG [UserService] Database query parameters: {"id":"123"}
[Nest] 12345  - 12/30/2025, 10:30:45 AM     LOG [UserService] User found: john@example.com
```

---

### **Logger Methods:**

```typescript
import { Logger } from '@nestjs/common';

export class ExampleService {
  private readonly logger = new Logger(ExampleService.name);

  demonstrateAllMethods() {
    // 1. LOG (general information)
    this.logger.log('This is a general log message');
    
    // 2. ERROR (failures and exceptions)
    this.logger.error('Something failed!', stackTrace, 'Context');
    
    // 3. WARN (warnings and potential issues)
    this.logger.warn('This is a warning');
    
    // 4. DEBUG (detailed debug information)
    this.logger.debug('Debug information here');
    
    // 5. VERBOSE (extremely detailed logs)
    this.logger.verbose('Verbose logging details');
  }
}
```

---

### **Static vs Instance Usage:**

```typescript
// Static usage (no context)
Logger.log('Application starting...');
Logger.error('Fatal error!', 'Main');

// Instance usage (with context) - RECOMMENDED
@Injectable()
export class OrderService {
  private readonly logger = new Logger(OrderService.name);
  //                                    ^^^^^^^^^^^^^^^^
  //                               Context: "OrderService"
  
  processOrder() {
    this.logger.log('Processing order');
    // Output: [Nest] ... LOG [OrderService] Processing order
  }
}
```

---

### **Production Example - Complete Service:**

```typescript
import { Injectable, Logger, NotFoundException } from '@nestjs/common';

@Injectable()
export class PaymentService {
  private readonly logger = new Logger(PaymentService.name);

  constructor(
    private paymentGateway: PaymentGateway,
    private orderRepository: OrderRepository,
  ) {
    // Log service initialization
    this.logger.log('PaymentService initialized');
  }

  async processPayment(orderId: string, amount: number) {
    // Log important business operation
    this.logger.log(
      `Processing payment - Order: ${orderId}, Amount: $${amount}`
    );

    try {
      // Log detailed execution (debug level)
      this.logger.debug(`Validating order ${orderId}`);
      
      const order = await this.orderRepository.findOne(orderId);
      if (!order) {
        // Log business validation failure
        this.logger.warn(`Order not found during payment: ${orderId}`);
        throw new NotFoundException('Order not found');
      }

      this.logger.debug(`Charging payment gateway for $${amount}`);
      const result = await this.paymentGateway.charge({
        amount,
        orderId,
        currency: 'USD',
      });

      // Log successful operation
      this.logger.log(
        `Payment successful - Order: ${orderId}, ` +
        `Transaction: ${result.transactionId}`
      );

      return result;
    } catch (error) {
      // Log error with full context and stack trace
      this.logger.error(
        `Payment failed - Order: ${orderId}, Amount: $${amount}`,
        error.stack,
        'PaymentService'
      );

      // Re-throw for controller to handle
      throw error;
    }
  }

  async refundPayment(transactionId: string) {
    this.logger.log(`Initiating refund for transaction ${transactionId}`);

    try {
      const result = await this.paymentGateway.refund(transactionId);
      
      this.logger.log(`Refund successful for transaction ${transactionId}`);
      return result;
    } catch (error) {
      if (error.code === 'ALREADY_REFUNDED') {
        // Use WARN for expected errors
        this.logger.warn(
          `Refund attempted on already refunded transaction: ${transactionId}`
        );
      } else {
        // Use ERROR for unexpected failures
        this.logger.error(
          `Refund failed for transaction ${transactionId}`,
          error.stack
        );
      }
      throw error;
    }
  }
}
```

---

### **Global Logger Configuration:**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { Logger } from '@nestjs/common';
import { AppModule } from './app.module';

async function bootstrap() {
  // Configure logger during app creation
  const app = await NestFactory.create(AppModule, {
    logger: ['error', 'warn', 'log'], // Only these levels
    // OR logger: false, // Disable all logging
    // OR logger: new MyCustomLogger(), // Custom logger
  });

  // Use logger for bootstrap messages
  const logger = new Logger('Bootstrap');
  
  await app.listen(3000);
  
  logger.log('Application is running on http://localhost:3000');
  logger.log(`Environment: ${process.env.NODE_ENV}`);
  logger.log(`Log levels: ERROR, WARN, LOG`);
}

bootstrap();
```

---

### **Environment-Based Configuration:**

```typescript
// main.ts
async function bootstrap() {
  const logLevels = {
    production: ['error', 'warn', 'log'],
    staging: ['error', 'warn', 'log', 'debug'],
    development: ['error', 'warn', 'log', 'debug', 'verbose'],
  };

  const currentEnv = process.env.NODE_ENV || 'development';

  const app = await NestFactory.create(AppModule, {
    logger: logLevels[currentEnv],
  });

  const logger = new Logger('Bootstrap');
  logger.log(`Starting in ${currentEnv} mode`);
  logger.log(`Active log levels: ${logLevels[currentEnv].join(', ')}`);

  await app.listen(3000);
}

bootstrap();
```

---

### **Interceptor with Logger Example:**

```typescript
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
  Logger,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  private readonly logger = new Logger(LoggingInterceptor.name);

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { method, url } = request;
    const startTime = Date.now();

    // Log incoming request
    this.logger.log(`Incoming ${method} ${url}`);

    return next.handle().pipe(
      tap({
        next: (data) => {
          const duration = Date.now() - startTime;
          // Log successful response
          this.logger.log(
            `Completed ${method} ${url} - ${duration}ms`
          );
        },
        error: (error) => {
          const duration = Date.now() - startTime;
          // Log failed response
          this.logger.error(
            `Failed ${method} ${url} - ${duration}ms`,
            error.stack
          );
        },
      }),
    );
  }
}
```

---

### **Exception Filter with Logger:**

```typescript
import {
  ExceptionFilter,
  Catch,
  ArgumentsHost,
  HttpException,
  Logger,
} from '@nestjs/common';

@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  private readonly logger = new Logger(AllExceptionsFilter.name);

  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const request = ctx.getRequest();
    const response = ctx.getResponse();

    const status =
      exception instanceof HttpException
        ? exception.getStatus()
        : 500;

    const message =
      exception instanceof HttpException
        ? exception.message
        : 'Internal server error';

    // Log all exceptions with context
    this.logger.error(
      `${request.method} ${request.url} - Status: ${status}`,
      exception instanceof Error ? exception.stack : 'Unknown error'
    );

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      message,
    });
  }
}
```

---

### **Benefits of Built-in Logger:**

```typescript
// ✅ Zero configuration needed
// ✅ Consistent format across application
// ✅ Context tracking (know where logs come from)
// ✅ Timestamp and process ID included
// ✅ Colorized console output
// ✅ Easy to test and mock
// ✅ Can be replaced with custom implementation

// Example benefits in production:
// [Nest] 12345 - 12/30/2025, 10:30:45 AM   LOG [PaymentService] Payment processed
// ^^^^^ ^^^^^^   ^^^^^^^^^^^^^^^^^^^^^^^^   ^^^ ^^^^^^^^^^^^^^^^ ^^^^^^^^^^^^^^^^^
// Brand PID      Timestamp                  Level Context       Message
```

---

### **Limitations:**

```typescript
// ❌ Console-only by default (no file output)
// ❌ No log rotation
// ❌ No external service integration (e.g., CloudWatch, Datadog)
// ❌ Limited formatting options
// ❌ Not optimized for high-performance scenarios

// Solution: Use third-party loggers (Winston, Pino) for production
// (We'll cover this in later questions)
```

---

### **Testing with Logger:**

```typescript
import { Test } from '@nestjs/testing';
import { Logger } from '@nestjs/common';
import { UserService } from './user.service';

describe('UserService', () => {
  let service: UserService;
  let logger: Logger;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [UserService],
    }).compile();

    service = module.get<UserService>(UserService);
    logger = service['logger']; // Access private logger

    // Mock logger methods
    jest.spyOn(logger, 'log').mockImplementation();
    jest.spyOn(logger, 'error').mockImplementation();
  });

  it('should log when finding user', async () => {
    await service.findUser('123');
    
    expect(logger.log).toHaveBeenCalledWith(
      'Searching for user with ID: 123'
    );
  });
});
```

---

### **When to Use Built-in Logger:**

```typescript
// ✅ GOOD for:
// - Development and testing
// - Small applications
// - Microservices with centralized log collection
// - Quick prototypes
// - Learning NestJS

// ⚠️ Consider alternatives for:
// - High-traffic production apps (use Pino)
// - Complex log formatting needs (use Winston)
// - Multiple log destinations (use Winston transports)
// - Performance-critical applications
// - Compliance requirements (structured logs)
```

**Key Takeaway**: The built-in Logger is perfect for getting started and small applications, but production apps often benefit from third-party loggers like Winston or Pino.

</details>

## Built-in Logger

<details>
<summary><strong>5. How do you use the built-in `Logger` class?</strong></summary>

**Answer:**

The NestJS `Logger` class provides multiple ways to log messages with different severity levels. You can use it as a static class, create instances with context, or inject it as a dependency.

---

### **Method 1: Static Usage (Quick & Simple)**

```typescript
import { Logger } from '@nestjs/common';

// Direct static calls - no instantiation needed
Logger.log('Application starting...');
Logger.error('Critical failure!', 'Bootstrap');
Logger.warn('Configuration missing');
Logger.debug('Debug information');
Logger.verbose('Detailed trace');
```

**Use Case:**
```typescript
// main.ts - Application bootstrap
import { NestFactory } from '@nestjs/core';
import { Logger } from '@nestjs/common';
import { AppModule } from './app.module';

async function bootstrap() {
  // Static logger for bootstrap process
  Logger.log('Starting application...', 'Bootstrap');
  
  const app = await NestFactory.create(AppModule);
  
  Logger.log('Application created successfully', 'Bootstrap');
  Logger.log(`Environment: ${process.env.NODE_ENV}`, 'Bootstrap');
  
  await app.listen(3000);
  
  Logger.log('Application listening on port 3000', 'Bootstrap');
}

bootstrap().catch((error) => {
  Logger.error('Application failed to start', error.stack, 'Bootstrap');
  process.exit(1);
});
```

---

### **Method 2: Instance with Context (Recommended)**

```typescript
import { Injectable, Logger } from '@nestjs/common';

@Injectable()
export class UserService {
  // Create logger instance with context
  private readonly logger = new Logger(UserService.name);
  //                                    ^^^^^^^^^^^^^^^^
  //                              Context identifies log source

  async createUser(userData: CreateUserDto) {
    // All logs will show [UserService] context
    this.logger.log(`Creating user: ${userData.email}`);
    
    try {
      const user = await this.userRepository.save(userData);
      
      this.logger.log(`User created successfully: ${user.id}`);
      return user;
    } catch (error) {
      this.logger.error(
        `Failed to create user: ${userData.email}`,
        error.stack
      );
      throw error;
    }
  }
}
```

**Output:**
```
[Nest] 12345  - 12/30/2025, 10:30:45 AM     LOG [UserService] Creating user: john@example.com
[Nest] 12345  - 12/30/2025, 10:30:46 AM     LOG [UserService] User created successfully: usr-123
```

---

### **Method 3: Custom Context Override**

```typescript
import { Logger } from '@nestjs/common';

export class PaymentService {
  private readonly logger = new Logger(PaymentService.name);

  async processPayment(orderId: string) {
    // Default context: PaymentService
    this.logger.log(`Processing payment for order ${orderId}`);
    
    try {
      await this.chargeCard();
    } catch (error) {
      // Override context for this specific log
      this.logger.error(
        'Card charge failed',
        error.stack,
        'PaymentGateway' // Custom context override
      );
    }
  }
}
```

**Output:**
```
[Nest] 12345  - 12/30/2025, 10:30:45 AM     LOG [PaymentService] Processing payment for order 123
[Nest] 12345  - 12/30/2025, 10:30:46 AM   ERROR [PaymentGateway] Card charge failed
```

---

### **All Logger Methods:**

```typescript
import { Logger } from '@nestjs/common';

export class ExampleService {
  private readonly logger = new Logger(ExampleService.name);

  demonstrateAllMethods() {
    // 1. LOG - General information
    this.logger.log('User logged in');
    
    // 2. ERROR - Errors with stack trace
    this.logger.error(
      'Operation failed',
      error.stack,        // Stack trace (optional)
      'CustomContext'     // Context override (optional)
    );
    
    // 3. WARN - Warnings
    this.logger.warn('Deprecated API called');
    
    // 4. DEBUG - Debug information
    this.logger.debug('Cache miss for key: user-123');
    
    // 5. VERBOSE - Detailed trace
    this.logger.verbose('Full request payload: ' + JSON.stringify(data));
  }
}
```

---

### **Production Example - Order Service:**

```typescript
import { Injectable, Logger, NotFoundException } from '@nestjs/common';

@Injectable()
export class OrderService {
  private readonly logger = new Logger(OrderService.name);

  constructor(
    private orderRepository: OrderRepository,
    private inventoryService: InventoryService,
    private paymentService: PaymentService,
  ) {
    // Log service initialization
    this.logger.log('OrderService initialized');
  }

  async createOrder(orderDto: CreateOrderDto) {
    // Log important business operation start
    this.logger.log(
      `Creating order for user ${orderDto.userId}, ` +
      `items: ${orderDto.items.length}, ` +
      `total: $${orderDto.totalAmount}`
    );

    // Debug-level logs for detailed flow
    this.logger.debug(`Validating order data: ${JSON.stringify(orderDto)}`);

    try {
      // Check inventory
      this.logger.debug('Checking inventory availability');
      const available = await this.inventoryService.checkAvailability(
        orderDto.items
      );

      if (!available) {
        // Warning for business validation failure
        this.logger.warn(
          `Insufficient inventory for order. User: ${orderDto.userId}`
        );
        throw new BadRequestException('Insufficient inventory');
      }

      // Process payment
      this.logger.log(`Processing payment: $${orderDto.totalAmount}`);
      const payment = await this.paymentService.charge(orderDto.totalAmount);

      // Create order
      const order = await this.orderRepository.save({
        ...orderDto,
        paymentId: payment.id,
        status: 'CONFIRMED',
      });

      // Log successful completion
      this.logger.log(
        `Order created successfully. ` +
        `OrderID: ${order.id}, ` +
        `PaymentID: ${payment.id}`
      );

      return order;
    } catch (error) {
      // Log error with full context
      this.logger.error(
        `Failed to create order for user ${orderDto.userId}`,
        error.stack,
        'OrderService'
      );
      throw error;
    }
  }

  async cancelOrder(orderId: string) {
    this.logger.log(`Cancelling order: ${orderId}`);

    const order = await this.orderRepository.findOne(orderId);
    
    if (!order) {
      this.logger.warn(`Order not found for cancellation: ${orderId}`);
      throw new NotFoundException('Order not found');
    }

    if (order.status === 'CANCELLED') {
      this.logger.warn(`Order already cancelled: ${orderId}`);
      return order;
    }

    // Refund payment
    this.logger.log(`Initiating refund for order ${orderId}`);
    await this.paymentService.refund(order.paymentId);

    order.status = 'CANCELLED';
    await this.orderRepository.save(order);

    this.logger.log(`Order cancelled successfully: ${orderId}`);
    return order;
  }
}
```

---

### **Best Practices:**

```typescript
// ✅ GOOD - Contextual logging
this.logger.log(`User ${userId} logged in successfully`);
this.logger.error('Payment failed', error.stack);

// ❌ BAD - No context
this.logger.log('Success');
this.logger.error('Error');

// ✅ GOOD - Structured information
this.logger.log(
  `Order created: ID=${order.id}, User=${order.userId}, Amount=$${order.total}`
);

// ❌ BAD - Unstructured
this.logger.log('Order created');

// ✅ GOOD - Instance with context
private readonly logger = new Logger(ServiceName.name);

// ⚠️ ACCEPTABLE - Static (for bootstrap only)
Logger.log('Starting app', 'Bootstrap');

// ✅ GOOD - Include error stack
this.logger.error('Operation failed', error.stack);

// ❌ BAD - No stack trace
this.logger.error('Operation failed');
```

---

### **Performance Considerations:**

```typescript
// Avoid expensive operations in logs that won't be shown
if (this.logger.isLevelEnabled('debug')) {
  // Only stringify if debug is enabled
  this.logger.debug(`Data: ${JSON.stringify(largeObject)}`);
}

// Or use lazy evaluation
this.logger.debug(() => `Data: ${JSON.stringify(largeObject)}`);
```

---

### **Testing with Logger:**

```typescript
import { Test } from '@nestjs/testing';
import { Logger } from '@nestjs/common';
import { OrderService } from './order.service';

describe('OrderService', () => {
  let service: OrderService;
  let loggerSpy: jest.SpyInstance;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [OrderService],
    }).compile();

    service = module.get<OrderService>(OrderService);
    
    // Spy on logger methods
    loggerSpy = jest.spyOn(Logger.prototype, 'log').mockImplementation();
  });

  it('should log order creation', async () => {
    await service.createOrder(mockOrderDto);
    
    expect(loggerSpy).toHaveBeenCalledWith(
      expect.stringContaining('Creating order')
    );
  });
});
```

</details>

<details>
<summary><strong>6. How do you inject Logger into services?</strong></summary>

**Answer:**

While you typically create a Logger instance directly in services, NestJS also supports dependency injection for Logger, which is useful for custom logger implementations and testing.

---

### **Method 1: Direct Instantiation (Most Common)**

```typescript
import { Injectable, Logger } from '@nestjs/common';

@Injectable()
export class UserService {
  // Create logger instance directly (no injection)
  private readonly logger = new Logger(UserService.name);

  async findUser(id: string) {
    this.logger.log(`Finding user: ${id}`);
    return await this.userRepository.findOne(id);
  }
}
```

**Pros:**
- Simple and straightforward
- No additional setup needed
- Context automatically set
- Most common pattern in NestJS

---

### **Method 2: Constructor Injection**

```typescript
import { Injectable, Logger } from '@nestjs/common';

@Injectable()
export class ProductService {
  // Inject Logger through constructor
  constructor(private readonly logger: Logger) {}

  async getProduct(id: string) {
    this.logger.log(`Fetching product: ${id}`);
    return await this.productRepository.findOne(id);
  }
}

// Module configuration
@Module({
  providers: [
    ProductService,
    Logger, // Add Logger to providers
  ],
})
export class ProductModule {}
```

**Issue:** No automatic context! Logs show generic "Logger" context.

**Output:**
```
[Nest] 12345  - 12/30/2025, 10:30:45 AM     LOG [Logger] Fetching product: 123
                                                    ^^^^^^^^ Not helpful!
```

---

### **Method 3: Custom Logger Provider with Context**

```typescript
import { Injectable, Logger, Scope } from '@nestjs/common';

@Injectable({ scope: Scope.TRANSIENT })
export class MyLogger extends Logger {
  constructor() {
    super();
  }
}

// Service using injected logger
@Injectable()
export class OrderService {
  constructor(private readonly logger: MyLogger) {
    // Set context after injection
    this.logger.setContext(OrderService.name);
  }

  async createOrder(data: CreateOrderDto) {
    this.logger.log('Creating order');
    // Output: [Nest] ... LOG [OrderService] Creating order
  }
}

// Module
@Module({
  providers: [
    OrderService,
    MyLogger, // Provide custom logger
  ],
})
export class OrderModule {}
```

---

### **Method 4: Factory Provider for Context-Aware Logger**

```typescript
import { Logger, Module, Provider } from '@nestjs/common';

// Factory function to create context-aware logger
function createLoggerProvider(context: string): Provider {
  return {
    provide: `${context}Logger`,
    useFactory: () => {
      const logger = new Logger();
      logger.setContext(context);
      return logger;
    },
  };
}

@Injectable()
export class PaymentService {
  constructor(
    @Inject('PaymentServiceLogger')
    private readonly logger: Logger,
  ) {}

  async processPayment(amount: number) {
    this.logger.log(`Processing payment: $${amount}`);
    // Output: [Nest] ... LOG [PaymentService] Processing payment: $100
  }
}

@Module({
  providers: [
    PaymentService,
    createLoggerProvider('PaymentService'),
  ],
})
export class PaymentModule {}
```

---

### **Method 5: Global Custom Logger Module**

```typescript
// logger.service.ts
import { Injectable, LoggerService, Scope } from '@nestjs/common';

@Injectable({ scope: Scope.TRANSIENT })
export class CustomLoggerService implements LoggerService {
  private context?: string;

  setContext(context: string) {
    this.context = context;
  }

  log(message: string) {
    console.log(`[${this.context}] LOG: ${message}`);
  }

  error(message: string, trace?: string) {
    console.error(`[${this.context}] ERROR: ${message}`, trace);
  }

  warn(message: string) {
    console.warn(`[${this.context}] WARN: ${message}`);
  }

  debug(message: string) {
    console.debug(`[${this.context}] DEBUG: ${message}`);
  }

  verbose(message: string) {
    console.log(`[${this.context}] VERBOSE: ${message}`);
  }
}

// logger.module.ts
import { Module, Global } from '@nestjs/common';
import { CustomLoggerService } from './logger.service';

@Global() // Make it available everywhere
@Module({
  providers: [CustomLoggerService],
  exports: [CustomLoggerService],
})
export class LoggerModule {}

// Usage in any service
@Injectable()
export class UserService {
  constructor(private readonly logger: CustomLoggerService) {
    this.logger.setContext(UserService.name);
  }

  async findUser(id: string) {
    this.logger.log(`Finding user: ${id}`);
  }
}

// app.module.ts
@Module({
  imports: [LoggerModule], // Import once
  // Now CustomLoggerService is available everywhere
})
export class AppModule {}
```

---

### **Method 6: Testing-Friendly Injection Pattern**

```typescript
import { Injectable, Logger } from '@nestjs/common';

// Create a token for logger injection
export const LOGGER = 'LOGGER';

@Injectable()
export class ProductService {
  constructor(
    @Inject(LOGGER)
    private readonly logger: Logger,
  ) {}

  async getProduct(id: string) {
    this.logger.log(`Fetching product: ${id}`);
    return await this.productRepository.findOne(id);
  }
}

// production.module.ts
@Module({
  providers: [
    ProductService,
    {
      provide: LOGGER,
      useFactory: () => {
        const logger = new Logger();
        logger.setContext('ProductService');
        return logger;
      },
    },
  ],
})
export class ProductModule {}

// Testing
describe('ProductService', () => {
  let service: ProductService;
  let mockLogger: jest.Mocked<Logger>;

  beforeEach(async () => {
    mockLogger = {
      log: jest.fn(),
      error: jest.fn(),
      warn: jest.fn(),
      debug: jest.fn(),
      verbose: jest.fn(),
    } as any;

    const module = await Test.createTestingModule({
      providers: [
        ProductService,
        {
          provide: LOGGER,
          useValue: mockLogger, // Mock logger for testing
        },
      ],
    }).compile();

    service = module.get<ProductService>(ProductService);
  });

  it('should log when fetching product', async () => {
    await service.getProduct('123');
    
    expect(mockLogger.log).toHaveBeenCalledWith('Fetching product: 123');
  });
});
```

---

### **Comparison Table:**

| Method | Pros | Cons | Use Case |
|--------|------|------|----------|
| **Direct Instantiation** | Simple, auto-context | Not testable without mocking | Most common, production code |
| **Constructor Injection** | Testable | No auto-context | When testing is critical |
| **Custom Provider** | Flexible, testable | More setup | Custom logger implementations |
| **Global Module** | Available everywhere | Adds complexity | Large applications |
| **Factory Provider** | Full control | Verbose | Advanced scenarios |

---

### **Recommended Pattern for Production:**

```typescript
// Simple services - Direct instantiation
@Injectable()
export class SimpleService {
  private readonly logger = new Logger(SimpleService.name);
  
  doSomething() {
    this.logger.log('Doing something');
  }
}

// Complex services with testing - Injection with context
@Injectable()
export class ComplexService {
  constructor(
    private readonly customLogger: CustomLoggerService,
  ) {
    this.customLogger.setContext(ComplexService.name);
  }
  
  async doComplexThing() {
    this.customLogger.log('Doing complex thing');
  }
}
```

---

### **Best Practices:**

```typescript
// ✅ GOOD - Direct instantiation for most cases
private readonly logger = new Logger(ServiceName.name);

// ✅ GOOD - Injection for custom logger
constructor(private readonly logger: CustomLoggerService) {
  this.logger.setContext(ServiceName.name);
}

// ❌ BAD - Injection without context
constructor(private readonly logger: Logger) {
  // Logs will show [Logger] instead of [ServiceName]
}

// ✅ GOOD - Set context after injection
constructor(private readonly logger: Logger) {
  this.logger.setContext(ServiceName.name);
}

// ✅ GOOD - Use class name for context
Logger(ServiceName.name) // Refactor-safe

// ❌ BAD - Hard-coded string
Logger('ServiceName') // Breaks on rename
```

---

### **Key Takeaways:**

1. **Default Pattern**: Direct instantiation (`new Logger(ClassName.name)`)
2. **Testing**: Use injection with custom providers
3. **Always Set Context**: Either in constructor or via `setContext()`
4. **Global Logger**: Use `@Global()` module for app-wide custom logger
5. **Keep It Simple**: Don't over-engineer unless you need custom behavior

</details>
<details>
<summary><strong>7. How do you create a logger instance with context?</strong></summary>

**Answer:**

Context in logging identifies the source of log messages, making it easier to trace issues in production. NestJS Logger provides multiple ways to add context to your logs.

---

### **Method 1: Constructor with Context (Recommended)**

```typescript
import { Injectable, Logger } from '@nestjs/common';

@Injectable()
export class UserService {
  // Pass context as constructor argument
  private readonly logger = new Logger(UserService.name);
  //                                    ^^^^^^^^^^^^^^^^
  //                              Context: "UserService"

  async createUser(email: string) {
    this.logger.log(`Creating user: ${email}`);
    // Output: [Nest] 12345  - 12/30/2025, 10:30:45 AM     LOG [UserService] Creating user: john@example.com
  }
}
```

**Benefits:**
- Auto-completion and type safety
- Refactor-safe (rename class = context updates)
- Most common and recommended pattern

---

### **Method 2: String Context**

```typescript
import { Logger } from '@nestjs/common';

export class PaymentService {
  // Use string literal as context
  private readonly logger = new Logger('PaymentService');
  
  processPayment() {
    this.logger.log('Processing payment');
    // Output: [Nest] ... LOG [PaymentService] Processing payment
  }
}
```

**Warning:** Not refactor-safe! If you rename the class, the context string won't update automatically.

---

### **Method 3: Dynamic Context with setContext()**

```typescript
import { Injectable, Logger } from '@nestjs/common';

@Injectable()
export class OrderService {
  private readonly logger = new Logger();

  constructor() {
    // Set context after instantiation
    this.logger.setContext(OrderService.name);
  }

  async createOrder() {
    this.logger.log('Creating order');
    // Output: [Nest] ... LOG [OrderService] Creating order
  }

  async processSpecialOrder() {
    // Temporarily change context for specific operation
    this.logger.setContext('SpecialOrderProcessor');
    this.logger.log('Processing special order');
    // Output: [Nest] ... LOG [SpecialOrderProcessor] Processing special order
    
    // Reset context
    this.logger.setContext(OrderService.name);
  }
}
```

---

### **Method 4: Per-Log Context Override**

```typescript
import { Logger } from '@nestjs/common';

export class ProductService {
  private readonly logger = new Logger(ProductService.name);

  async getProduct(id: string) {
    // Default context: ProductService
    this.logger.log(`Fetching product ${id}`);
    // Output: [Nest] ... LOG [ProductService] Fetching product 123

    try {
      const cached = await this.cache.get(id);
      
      if (cached) {
        // Override context for this specific log
        this.logger.log(
          `Cache hit for product ${id}`,
          'CacheManager' // Context override as 3rd argument
        );
        // Output: [Nest] ... LOG [CacheManager] Cache hit for product 123
        return cached;
      }
    } catch (error) {
      // Error logs support context override too
      this.logger.error(
        'Cache lookup failed',
        error.stack,
        'CacheManager' // Context override
      );
      // Output: [Nest] ... ERROR [CacheManager] Cache lookup failed
    }
  }
}
```

---

### **Method 5: Hierarchical Context (Custom Pattern)**

```typescript
import { Injectable, Logger } from '@nestjs/common';

@Injectable()
export class AuthService {
  private readonly logger: Logger;

  constructor() {
    // Create hierarchical context
    this.logger = new Logger(`${AuthService.name}`);
  }

  private createContextualLogger(method: string): Logger {
    const logger = new Logger();
    logger.setContext(`${AuthService.name}:${method}`);
    return logger;
  }

  async login(email: string) {
    const logger = this.createContextualLogger('login');
    
    logger.log(`Login attempt: ${email}`);
    // Output: [Nest] ... LOG [AuthService:login] Login attempt: user@example.com
    
    try {
      const user = await this.validateCredentials(email);
      logger.log('Login successful');
      return user;
    } catch (error) {
      logger.error('Login failed', error.stack);
      throw error;
    }
  }

  async register(email: string) {
    const logger = this.createContextualLogger('register');
    
    logger.log(`Registration attempt: ${email}`);
    // Output: [Nest] ... LOG [AuthService:register] Registration attempt: user@example.com
  }
}
```

---

### **Production Example - E-Commerce Application:**

```typescript
import { Injectable, Logger } from '@nestjs/common';

@Injectable()
export class OrderProcessingService {
  // Main logger with service context
  private readonly logger = new Logger(OrderProcessingService.name);

  async processOrder(order: Order) {
    // Use service context for general operations
    this.logger.log(
      `Processing order ${order.id} for user ${order.userId}`
    );
    // Output: [OrderProcessingService] Processing order...

    try {
      // Step 1: Validate inventory
      this.logger.debug('Validating inventory', 'InventoryCheck');
      await this.validateInventory(order);
      // Output: [InventoryCheck] Validating inventory

      // Step 2: Process payment
      this.logger.log(
        `Processing payment: $${order.totalAmount}`,
        'PaymentProcessor'
      );
      const payment = await this.processPayment(order);
      // Output: [PaymentProcessor] Processing payment: $99.99

      // Step 3: Update order status
      this.logger.log('Updating order status', 'OrderStatusManager');
      await this.updateOrderStatus(order.id, 'CONFIRMED');
      // Output: [OrderStatusManager] Updating order status

      // Success log with default context
      this.logger.log(
        `Order processed successfully: ${order.id}`
      );
      // Output: [OrderProcessingService] Order processed successfully...

      return { success: true, orderId: order.id };
    } catch (error) {
      // Error with context showing which step failed
      this.logger.error(
        `Order processing failed at step: ${error.step}`,
        error.stack,
        error.component // Dynamic context based on failure point
      );
      throw error;
    }
  }

  private async validateInventory(order: Order) {
    // Create scoped logger for this operation
    const logger = new Logger('InventoryValidator');
    
    logger.debug(`Checking ${order.items.length} items`);
    // All logs in this method use [InventoryValidator] context
    
    for (const item of order.items) {
      logger.verbose(`Checking stock for item ${item.productId}`);
      const stock = await this.inventoryService.getStock(item.productId);
      
      if (stock < item.quantity) {
        logger.warn(
          `Insufficient stock for ${item.productId}. ` +
          `Required: ${item.quantity}, Available: ${stock}`
        );
        throw new Error('Insufficient inventory');
      }
    }
    
    logger.log('Inventory validation passed');
  }
}
```

**Output Example:**
```
[Nest] 12345  - 12/30/2025, 10:30:45 AM     LOG [OrderProcessingService] Processing order ORD-123 for user USR-456
[Nest] 12345  - 12/30/2025, 10:30:45 AM   DEBUG [InventoryCheck] Validating inventory
[Nest] 12345  - 12/30/2025, 10:30:46 AM     LOG [InventoryValidator] Checking 3 items
[Nest] 12345  - 12/30/2025, 10:30:46 AM     LOG [InventoryValidator] Inventory validation passed
[Nest] 12345  - 12/30/2025, 10:30:47 AM     LOG [PaymentProcessor] Processing payment: $99.99
[Nest] 12345  - 12/30/2025, 10:30:48 AM     LOG [OrderStatusManager] Updating order status
[Nest] 12345  - 12/30/2025, 10:30:48 AM     LOG [OrderProcessingService] Order processed successfully: ORD-123
```

---

### **Context Naming Conventions:**

```typescript
// ✅ GOOD - Service name as context
private readonly logger = new Logger(UserService.name);

// ✅ GOOD - Module:Service pattern
private readonly logger = new Logger('AuthModule:AuthService');

// ✅ GOOD - Feature-based context
private readonly logger = new Logger('PaymentProcessing');

// ✅ GOOD - Domain-driven context
private readonly logger = new Logger('OrderManagement:Inventory');

// ❌ BAD - Too generic
private readonly logger = new Logger('Service');

// ❌ BAD - Too verbose
private readonly logger = new Logger(
  'ApplicationOrderProcessingServiceBusinessLogicHandler'
);

// ❌ BAD - Hard-coded, not refactor-safe
private readonly logger = new Logger('UserService'); // Use UserService.name instead
```

---

### **Static Logger with Context:**

```typescript
import { Logger } from '@nestjs/common';

// In main.ts or bootstrap functions
Logger.log('Application starting', 'Bootstrap');
Logger.log('Database connected', 'Database');
Logger.log('Listening on port 3000', 'Application');

// Output:
// [Nest] ... LOG [Bootstrap] Application starting
// [Nest] ... LOG [Database] Database connected
// [Nest] ... LOG [Application] Listening on port 3000
```

---

### **Testing Context Behavior:**

```typescript
import { Test } from '@nestjs/testing';
import { Logger } from '@nestjs/common';
import { UserService } from './user.service';

describe('UserService Logging', () => {
  let service: UserService;
  let logSpy: jest.SpyInstance;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [UserService],
    }).compile();

    service = module.get<UserService>(UserService);
    
    // Spy on logger
    logSpy = jest.spyOn(Logger.prototype, 'log');
  });

  it('should log with correct context', async () => {
    await service.createUser('test@example.com');
    
    // Verify log was called
    expect(logSpy).toHaveBeenCalled();
    
    // Verify context (check the logger instance)
    const loggerInstance = service['logger'];
    expect(loggerInstance['context']).toBe('UserService');
  });
});
```

---

### **Best Practices:**

```typescript
// ✅ DO: Use class name for context
private readonly logger = new Logger(ClassName.name);

// ✅ DO: Use meaningful, specific contexts
private readonly logger = new Logger('PaymentGateway');

// ✅ DO: Override context for sub-operations
this.logger.log('Cache miss', 'CacheManager');

// ✅ DO: Create scoped loggers for complex operations
const operationLogger = new Logger('SpecificOperation');

// ❌ DON'T: Use generic contexts
private readonly logger = new Logger('Logger');

// ❌ DON'T: Hard-code strings (not refactor-safe)
private readonly logger = new Logger('MyService');

// ❌ DON'T: Make contexts too long
private readonly logger = new Logger(
  'VeryLongApplicationUserManagementServiceImplementation'
);
```

**Key Takeaway:** Context is crucial for debugging production issues. Use descriptive, consistent contexts that clearly identify the log source.

</details>

<details>
<summary><strong>8. How do you disable logging?</strong></summary>

**Answer:**

NestJS provides multiple ways to disable logging globally, selectively by level, or completely. This is useful for testing, performance optimization, or compliance requirements.

---

### **Method 1: Disable All Logging**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, {
    logger: false, // Completely disable logging
  });

  await app.listen(3000);
  // No logs will be output at all
}

bootstrap();
```

**Use Case:** Silent testing, performance benchmarks, or specific security requirements.

---

### **Method 2: Disable Specific Log Levels**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, {
    // Only enable ERROR and WARN, disable LOG, DEBUG, VERBOSE
    logger: ['error', 'warn'],
  });

  await app.listen(3000);
  // Only error and warn logs will appear
}

bootstrap();
```

---

### **Method 3: Environment-Based Configuration**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { Logger } from '@nestjs/common';
import { AppModule } from './app.module';

async function bootstrap() {
  // Define log levels per environment
  const logLevels = {
    production: ['error', 'warn', 'log'],
    staging: ['error', 'warn', 'log', 'debug'],
    development: ['error', 'warn', 'log', 'debug', 'verbose'],
    test: false, // Completely disable in tests
  };

  const env = process.env.NODE_ENV || 'development';
  
  const app = await NestFactory.create(AppModule, {
    logger: logLevels[env],
  });

  // Only log if not in test mode
  if (env !== 'test') {
    const logger = new Logger('Bootstrap');
    logger.log(`Application running in ${env} mode`);
    logger.log(`Log levels: ${JSON.stringify(logLevels[env])}`);
  }

  await app.listen(3000);
}

bootstrap();
```

---

### **Method 4: Environment Variables**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  // Read from environment variable
  const loggingEnabled = process.env.ENABLE_LOGGING !== 'false';
  
  const app = await NestFactory.create(AppModule, {
    logger: loggingEnabled ? ['error', 'warn', 'log'] : false,
  });

  await app.listen(3000);
}

bootstrap();
```

```bash
# .env for production
ENABLE_LOGGING=true

# .env for testing
ENABLE_LOGGING=false
```

---

### **Method 5: Custom Logger with Disable Flag**

```typescript
// custom-logger.service.ts
import { Injectable, LoggerService } from '@nestjs/common';

@Injectable()
export class CustomLogger implements LoggerService {
  private enabled: boolean;

  constructor() {
    this.enabled = process.env.LOGGING_ENABLED === 'true';
  }

  log(message: string) {
    if (this.enabled) {
      console.log(`LOG: ${message}`);
    }
  }

  error(message: string, trace?: string) {
    if (this.enabled) {
      console.error(`ERROR: ${message}`, trace);
    }
  }

  warn(message: string) {
    if (this.enabled) {
      console.warn(`WARN: ${message}`);
    }
  }

  debug(message: string) {
    if (this.enabled) {
      console.debug(`DEBUG: ${message}`);
    }
  }

  verbose(message: string) {
    if (this.enabled) {
      console.log(`VERBOSE: ${message}`);
    }
  }

  // Method to toggle logging at runtime
  setEnabled(enabled: boolean) {
    this.enabled = enabled;
  }
}

// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { CustomLogger } from './custom-logger.service';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, {
    logger: new CustomLogger(),
  });

  await app.listen(3000);
}

bootstrap();
```

---

### **Method 6: Disable in Tests**

```typescript
// user.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { Logger } from '@nestjs/common';
import { UserService } from './user.service';

describe('UserService', () => {
  let service: UserService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [UserService],
    })
      // Disable logging during tests
      .setLogger(false)
      .compile();

    service = module.get<UserService>(UserService);
  });

  it('should create user without logs', async () => {
    // Test runs silently, no log output
    const user = await service.createUser({ email: 'test@example.com' });
    expect(user).toBeDefined();
  });
});
```

**Alternative - Mock Logger:**
```typescript
beforeEach(async () => {
  // Mock all logger methods
  jest.spyOn(Logger.prototype, 'log').mockImplementation();
  jest.spyOn(Logger.prototype, 'error').mockImplementation();
  jest.spyOn(Logger.prototype, 'warn').mockImplementation();
  jest.spyOn(Logger.prototype, 'debug').mockImplementation();
  jest.spyOn(Logger.prototype, 'verbose').mockImplementation();

  const module = await Test.createTestingModule({
    providers: [UserService],
  }).compile();

  service = module.get<UserService>(UserService);
});
```

---

### **Method 7: Silent Logger Class**

```typescript
// silent-logger.ts
import { LoggerService } from '@nestjs/common';

export class SilentLogger implements LoggerService {
  log(message: string) {}
  error(message: string, trace?: string) {}
  warn(message: string) {}
  debug(message: string) {}
  verbose(message: string) {}
}

// main.ts (testing/CI environment)
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { SilentLogger } from './silent-logger';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, {
    logger: new SilentLogger(), // No output
  });

  await app.listen(3000);
}

bootstrap();
```

---

### **Method 8: Conditional Service Logging**

```typescript
import { Injectable, Logger } from '@nestjs/common';

@Injectable()
export class UserService {
  private readonly logger = new Logger(UserService.name);
  private readonly loggingEnabled: boolean;

  constructor() {
    // Check environment variable
    this.loggingEnabled = process.env.SERVICE_LOGGING !== 'false';
  }

  async createUser(email: string) {
    // Conditional logging
    if (this.loggingEnabled) {
      this.logger.log(`Creating user: ${email}`);
    }

    const user = await this.userRepository.save({ email });

    if (this.loggingEnabled) {
      this.logger.log(`User created: ${user.id}`);
    }

    return user;
  }
}
```

---

### **Production Configuration Example:**

```typescript
// config/logger.config.ts
import { LogLevel } from '@nestjs/common';

export interface LoggerConfig {
  enabled: boolean;
  levels: LogLevel[];
}

export const getLoggerConfig = (): LoggerConfig => {
  const env = process.env.NODE_ENV;

  switch (env) {
    case 'production':
      return {
        enabled: true,
        levels: ['error', 'warn', 'log'],
      };
    
    case 'staging':
      return {
        enabled: true,
        levels: ['error', 'warn', 'log', 'debug'],
      };
    
    case 'development':
      return {
        enabled: true,
        levels: ['error', 'warn', 'log', 'debug', 'verbose'],
      };
    
    case 'test':
      return {
        enabled: false,
        levels: [],
      };
    
    default:
      return {
        enabled: true,
        levels: ['error', 'warn', 'log'],
      };
  }
};

// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { getLoggerConfig } from './config/logger.config';

async function bootstrap() {
  const loggerConfig = getLoggerConfig();

  const app = await NestFactory.create(AppModule, {
    logger: loggerConfig.enabled ? loggerConfig.levels : false,
  });

  await app.listen(3000);
}

bootstrap();
```

---

### **Method 9: Dynamic Runtime Control**

```typescript
// logger-control.service.ts
import { Injectable } from '@nestjs/common';
import { Logger } from '@nestjs/common';

@Injectable()
export class LoggerControlService {
  private static globalLoggingEnabled = true;

  static disableLogging() {
    this.globalLoggingEnabled = false;
  }

  static enableLogging() {
    this.globalLoggingEnabled = true;
  }

  static isEnabled(): boolean {
    return this.globalLoggingEnabled;
  }
}

// Wrapper logger
export class ControlledLogger extends Logger {
  log(message: string, context?: string) {
    if (LoggerControlService.isEnabled()) {
      super.log(message, context);
    }
  }

  error(message: string, trace?: string, context?: string) {
    if (LoggerControlService.isEnabled()) {
      super.error(message, trace, context);
    }
  }

  // ... implement other methods
}

// Usage
LoggerControlService.disableLogging(); // Turn off all logs
LoggerControlService.enableLogging();  // Turn back on
```

---

### **Comparison Table:**

| Method | Scope | Use Case | Pros | Cons |
|--------|-------|----------|------|------|
| `logger: false` | Global | Complete silence | Simple, clean | All or nothing |
| `logger: ['error']` | Global | Specific levels | Flexible | Need to restart |
| Environment-based | Global | Multi-environment | Best practice | Requires config |
| Custom Logger | Global | Full control | Very flexible | More code |
| Test mocking | Per-test | Unit tests | Precise | Test-only |
| Conditional in service | Per-service | Selective | Fine-grained | Repetitive |

---

### **Best Practices:**

```typescript
// ✅ GOOD - Environment-based
const app = await NestFactory.create(AppModule, {
  logger: process.env.NODE_ENV === 'test' 
    ? false 
    : ['error', 'warn', 'log'],
});

// ✅ GOOD - Disable in tests
const module = await Test.createTestingModule({
  providers: [UserService],
}).setLogger(false).compile();

// ✅ GOOD - Production-only critical logs
logger: ['error', 'warn']

// ❌ BAD - Always disabled
logger: false // Lost all observability!

// ❌ BAD - Hard-coded in code
if (true) { this.logger.log(...) } // Use config instead
```

---

### **Package.json Scripts:**

```json
{
  "scripts": {
    "start:dev": "NODE_ENV=development nest start --watch",
    "start:prod": "NODE_ENV=production node dist/main",
    "start:silent": "ENABLE_LOGGING=false node dist/main",
    "test": "NODE_ENV=test jest",
    "test:silent": "NODE_ENV=test jest --silent"
  }
}
```

**Key Takeaway:** Always disable logging in tests for clean output, but keep ERROR/WARN logs in production for monitoring and debugging.

</details>
<details>
<summary><strong>9. Can you customize log format?</strong></summary>

**Answer:**

Yes, you can customize the log format in NestJS by creating a custom logger that implements the `LoggerService` interface. This allows you to control how logs are formatted, add timestamps, include metadata, or integrate with external logging services.

---

### **Method 1: Extend Built-in Logger**

```typescript
// custom-logger.ts
import { Logger } from '@nestjs/common';

export class CustomFormattedLogger extends Logger {
  log(message: string, context?: string) {
    // Custom format: Add emoji and custom timestamp
    const timestamp = new Date().toISOString();
    console.log(`✅ [${timestamp}] [${context || 'Application'}] ${message}`);
  }

  error(message: string, trace?: string, context?: string) {
    const timestamp = new Date().toISOString();
    console.error(`❌ [${timestamp}] [${context || 'Application'}] ERROR: ${message}`);
    if (trace) {
      console.error(`Stack trace: ${trace}`);
    }
  }

  warn(message: string, context?: string) {
    const timestamp = new Date().toISOString();
    console.warn(`⚠️  [${timestamp}] [${context || 'Application'}] WARN: ${message}`);
  }

  debug(message: string, context?: string) {
    const timestamp = new Date().toISOString();
    console.debug(`🐞 [${timestamp}] [${context || 'Application'}] DEBUG: ${message}`);
  }

  verbose(message: string, context?: string) {
    const timestamp = new Date().toISOString();
    console.log(`💬 [${timestamp}] [${context || 'Application'}] VERBOSE: ${message}`);
  }
}

// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { CustomFormattedLogger } from './custom-logger';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, {
    logger: new CustomFormattedLogger(),
  });

  await app.listen(3000);
}
bootstrap();
```

**Output:**
```
✅ [2025-12-30T10:30:45.123Z] [UserService] User created successfully
❌ [2025-12-30T10:30:46.456Z] [PaymentService] ERROR: Payment processing failed
⚠️  [2025-12-30T10:30:47.789Z] [CacheService] WARN: Cache unavailable
```

---

### **Method 2: JSON Format for Production**

```typescript
// json-logger.ts
import { LoggerService } from '@nestjs/common';

export class JsonLogger implements LoggerService {
  private formatLog(level: string, message: string, context?: string, trace?: string) {
    const log = {
      timestamp: new Date().toISOString(),
      level: level.toUpperCase(),
      context: context || 'Application',
      message,
      pid: process.pid,
      ...(trace && { trace }),
    };
    return JSON.stringify(log);
  }

  log(message: string, context?: string) {
    console.log(this.formatLog('info', message, context));
  }

  error(message: string, trace?: string, context?: string) {
    console.error(this.formatLog('error', message, context, trace));
  }

  warn(message: string, context?: string) {
    console.warn(this.formatLog('warn', message, context));
  }

  debug(message: string, context?: string) {
    console.debug(this.formatLog('debug', message, context));
  }

  verbose(message: string, context?: string) {
    console.log(this.formatLog('verbose', message, context));
  }
}

// main.ts
const app = await NestFactory.create(AppModule, {
  logger: new JsonLogger(),
});
```

**Output:**
```json
{"timestamp":"2025-12-30T10:30:45.123Z","level":"INFO","context":"UserService","message":"User created successfully","pid":12345}
{"timestamp":"2025-12-30T10:30:46.456Z","level":"ERROR","context":"PaymentService","message":"Payment failed","pid":12345,"trace":"Error: Payment failed\n    at PaymentService.process..."}
```

---

### **Method 3: Color-Coded Console Logger**

```typescript
// Install: npm install chalk
import { LoggerService } from '@nestjs/common';
import * as chalk from 'chalk';

export class ColoredLogger implements LoggerService {
  private formatMessage(level: string, message: string, context?: string): string {
    const timestamp = new Date().toLocaleString();
    const ctx = context || 'App';
    return `[${timestamp}] [${ctx}] ${level}: ${message}`;
  }

  log(message: string, context?: string) {
    console.log(
      chalk.green(this.formatMessage('LOG', message, context))
    );
  }

  error(message: string, trace?: string, context?: string) {
    console.error(
      chalk.red(this.formatMessage('ERROR', message, context))
    );
    if (trace) {
      console.error(chalk.red(trace));
    }
  }

  warn(message: string, context?: string) {
    console.warn(
      chalk.yellow(this.formatMessage('WARN', message, context))
    );
  }

  debug(message: string, context?: string) {
    console.debug(
      chalk.blue(this.formatMessage('DEBUG', message, context))
    );
  }

  verbose(message: string, context?: string) {
    console.log(
      chalk.gray(this.formatMessage('VERBOSE', message, context))
    );
  }
}
```

---

### **Method 4: Production Logger with Metadata**

```typescript
// production-logger.ts
import { LoggerService } from '@nestjs/common';

interface LogMetadata {
  timestamp: string;
  level: string;
  context: string;
  message: string;
  pid: number;
  hostname: string;
  environment: string;
  requestId?: string;
  userId?: string;
  [key: string]: any;
}

export class ProductionLogger implements LoggerService {
  private metadata: Partial<LogMetadata> = {};

  constructor() {
    // Set default metadata
    this.metadata = {
      pid: process.pid,
      hostname: require('os').hostname(),
      environment: process.env.NODE_ENV || 'development',
    };
  }

  // Method to add request-specific metadata
  setRequestMetadata(requestId: string, userId?: string) {
    this.metadata.requestId = requestId;
    this.metadata.userId = userId;
  }

  private formatLog(
    level: string,
    message: string,
    context?: string,
    trace?: string,
  ): string {
    const log: LogMetadata = {
      timestamp: new Date().toISOString(),
      level: level.toUpperCase(),
      context: context || 'Application',
      message,
      ...this.metadata,
      ...(trace && { trace }),
    };

    return JSON.stringify(log);
  }

  log(message: string, context?: string) {
    console.log(this.formatLog('info', message, context));
  }

  error(message: string, trace?: string, context?: string) {
    console.error(this.formatLog('error', message, context, trace));
  }

  warn(message: string, context?: string) {
    console.warn(this.formatLog('warn', message, context));
  }

  debug(message: string, context?: string) {
    if (process.env.NODE_ENV !== 'production') {
      console.debug(this.formatLog('debug', message, context));
    }
  }

  verbose(message: string, context?: string) {
    if (process.env.NODE_ENV === 'development') {
      console.log(this.formatLog('verbose', message, context));
    }
  }
}

// Usage in interceptor
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';
import { ProductionLogger } from './production-logger';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  constructor(private logger: ProductionLogger) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const requestId = request.headers['x-request-id'] || this.generateRequestId();
    const userId = request.user?.id;

    // Set metadata for this request
    this.logger.setRequestMetadata(requestId, userId);

    return next.handle();
  }

  private generateRequestId(): string {
    return `req-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }
}
```

**Output:**
```json
{
  "timestamp": "2025-12-30T10:30:45.123Z",
  "level": "INFO",
  "context": "OrderService",
  "message": "Order created",
  "pid": 12345,
  "hostname": "api-server-01",
  "environment": "production",
  "requestId": "req-1703937045-abc123xyz",
  "userId": "user-456"
}
```

---

### **Method 5: Multi-Line Pretty Format (Development)**

```typescript
// pretty-logger.ts
import { LoggerService } from '@nestjs/common';

export class PrettyLogger implements LoggerService {
  private formatPretty(
    level: string,
    message: string,
    context?: string,
    trace?: string,
  ): string {
    const timestamp = new Date().toLocaleString();
    const border = '='.repeat(80);
    
    let output = `\n${border}\n`;
    output += `[${level.toUpperCase()}] ${timestamp}\n`;
    output += `Context: ${context || 'Application'}\n`;
    output += `Message: ${message}\n`;
    
    if (trace) {
      output += `\nStack Trace:\n${trace}\n`;
    }
    
    output += border;
    
    return output;
  }

  log(message: string, context?: string) {
    console.log(this.formatPretty('LOG', message, context));
  }

  error(message: string, trace?: string, context?: string) {
    console.error(this.formatPretty('ERROR', message, context, trace));
  }

  warn(message: string, context?: string) {
    console.warn(this.formatPretty('WARN', message, context));
  }

  debug(message: string, context?: string) {
    console.debug(this.formatPretty('DEBUG', message, context));
  }

  verbose(message: string, context?: string) {
    console.log(this.formatPretty('VERBOSE', message, context));
  }
}
```

**Output:**
```
================================================================================
[ERROR] 12/30/2025, 10:30:45 AM
Context: PaymentService
Message: Payment processing failed

Stack Trace:
Error: Card declined
    at PaymentGateway.charge (payment-gateway.ts:45:11)
    at PaymentService.process (payment.service.ts:23:5)
================================================================================
```

---

### **Method 6: Environment-Specific Formatting**

```typescript
// adaptive-logger.ts
import { LoggerService } from '@nestjs/common';
import { JsonLogger } from './json-logger';
import { PrettyLogger } from './pretty-logger';

export class AdaptiveLogger implements LoggerService {
  private logger: LoggerService;

  constructor() {
    // JSON for production, Pretty for development
    this.logger =
      process.env.NODE_ENV === 'production'
        ? new JsonLogger()
        : new PrettyLogger();
  }

  log(message: string, context?: string) {
    this.logger.log(message, context);
  }

  error(message: string, trace?: string, context?: string) {
    this.logger.error(message, trace, context);
  }

  warn(message: string, context?: string) {
    this.logger.warn(message, context);
  }

  debug(message: string, context?: string) {
    this.logger.debug(message, context);
  }

  verbose(message: string, context?: string) {
    this.logger.verbose(message, context);
  }
}

// main.ts
const app = await NestFactory.create(AppModule, {
  logger: new AdaptiveLogger(),
});
```

---

### **Best Practices:**

```typescript
// ✅ GOOD - JSON format for production (parseable by log aggregators)
export class ProductionLogger implements LoggerService {
  log(message: string) {
    console.log(JSON.stringify({
      level: 'info',
      message,
      timestamp: new Date().toISOString(),
    }));
  }
}

// ✅ GOOD - Pretty format for development (human-readable)
export class DevelopmentLogger implements LoggerService {
  log(message: string, context?: string) {
    console.log(`[${context}] ${message}`);
  }
}

// ✅ GOOD - Include essential metadata
const log = {
  timestamp: new Date().toISOString(),
  level: 'info',
  message,
  context,
  requestId,
};

// ❌ BAD - No timestamp
console.log(message); // When did this happen?

// ❌ BAD - Unstructured in production
console.log(`User ${userId} did something`); // Hard to parse

// ❌ BAD - Too much formatting overhead
const formatted = this.applyColors(
  this.addBorders(
    this.addEmojis(message)
  )
); // Slow!
```

---

### **Performance Consideration:**

```typescript
// ⚠️ Avoid expensive formatting for disabled log levels
export class OptimizedLogger implements LoggerService {
  private shouldLog(level: string): boolean {
    const enabledLevels = process.env.LOG_LEVELS?.split(',') || ['error', 'warn', 'log'];
    return enabledLevels.includes(level);
  }

  debug(message: string, context?: string) {
    // Skip formatting if debug is disabled
    if (!this.shouldLog('debug')) {
      return;
    }
    
    // Only format if needed
    const formatted = this.expensiveFormatting(message, context);
    console.debug(formatted);
  }
}
```

**Key Takeaway:** Use JSON format for production (parseable), pretty format for development (readable), and always include timestamp, level, and context.

</details>

## Custom Loggers

<details>
<summary><strong>10. How do you create a custom logger?</strong></summary>

**Answer:**

Creating a custom logger in NestJS involves implementing the `LoggerService` interface. This allows you to fully control logging behavior, integrate with external services, add custom features, and format logs according to your needs.

---

### **Step 1: Implement LoggerService Interface**

```typescript
// custom-logger.service.ts
import { LoggerService, Injectable } from '@nestjs/common';

@Injectable()
export class CustomLogger implements LoggerService {
  /**
   * Write a 'log' level log.
   */
  log(message: any, ...optionalParams: any[]) {
    console.log('[CUSTOM LOG]', message, ...optionalParams);
  }

  /**
   * Write an 'error' level log.
   */
  error(message: any, ...optionalParams: any[]) {
    console.error('[CUSTOM ERROR]', message, ...optionalParams);
  }

  /**
   * Write a 'warn' level log.
   */
  warn(message: any, ...optionalParams: any[]) {
    console.warn('[CUSTOM WARN]', message, ...optionalParams);
  }

  /**
   * Write a 'debug' level log.
   */
  debug?(message: any, ...optionalParams: any[]) {
    console.debug('[CUSTOM DEBUG]', message, ...optionalParams);
  }

  /**
   * Write a 'verbose' level log.
   */
  verbose?(message: any, ...optionalParams: any[]) {
    console.log('[CUSTOM VERBOSE]', message, ...optionalParams);
  }
}
```

---

### **Step 2: Use Custom Logger in Application**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { CustomLogger } from './custom-logger.service';

async function bootstrap() {
  // Method 1: Provide at application creation
  const app = await NestFactory.create(AppModule, {
    logger: new CustomLogger(),
  });

  await app.listen(3000);
}

bootstrap();
```

---

### **Step 3: Advanced Custom Logger with Features**

```typescript
// advanced-logger.service.ts
import { LoggerService, Injectable } from '@nestjs/common';
import * as fs from 'fs';
import * as path from 'path';

interface LogEntry {
  timestamp: string;
  level: string;
  context?: string;
  message: string;
  trace?: string;
  metadata?: Record<string, any>;
}

@Injectable()
export class AdvancedLogger implements LoggerService {
  private logFile: string;
  private context?: string;

  constructor() {
    // Create logs directory if it doesn't exist
    const logsDir = path.join(process.cwd(), 'logs');
    if (!fs.existsSync(logsDir)) {
      fs.mkdirSync(logsDir, { recursive: true });
    }

    // Create log file with date
    const date = new Date().toISOString().split('T')[0];
    this.logFile = path.join(logsDir, `app-${date}.log`);
  }

  setContext(context: string) {
    this.context = context;
  }

  private writeToFile(logEntry: LogEntry) {
    const logLine = JSON.stringify(logEntry) + '\n';
    fs.appendFileSync(this.logFile, logLine);
  }

  private createLogEntry(
    level: string,
    message: any,
    trace?: string,
  ): LogEntry {
    return {
      timestamp: new Date().toISOString(),
      level: level.toUpperCase(),
      context: this.context,
      message: typeof message === 'object' ? JSON.stringify(message) : message,
      ...(trace && { trace }),
    };
  }

  log(message: any, context?: string) {
    const logEntry = this.createLogEntry('log', message);
    if (context) logEntry.context = context;
    
    // Write to console
    console.log(
      `[✅ ${logEntry.timestamp}] [${logEntry.context || 'App'}] ${logEntry.message}`
    );
    
    // Write to file
    this.writeToFile(logEntry);
  }

  error(message: any, trace?: string, context?: string) {
    const logEntry = this.createLogEntry('error', message, trace);
    if (context) logEntry.context = context;
    
    console.error(
      `[❌ ${logEntry.timestamp}] [${logEntry.context || 'App'}] ${logEntry.message}`
    );
    if (trace) console.error(trace);
    
    this.writeToFile(logEntry);
  }

  warn(message: any, context?: string) {
    const logEntry = this.createLogEntry('warn', message);
    if (context) logEntry.context = context;
    
    console.warn(
      `[⚠️  ${logEntry.timestamp}] [${logEntry.context || 'App'}] ${logEntry.message}`
    );
    
    this.writeToFile(logEntry);
  }

  debug(message: any, context?: string) {
    if (process.env.NODE_ENV === 'production') return;
    
    const logEntry = this.createLogEntry('debug', message);
    if (context) logEntry.context = context;
    
    console.debug(
      `[🐞 ${logEntry.timestamp}] [${logEntry.context || 'App'}] ${logEntry.message}`
    );
    
    this.writeToFile(logEntry);
  }

  verbose(message: any, context?: string) {
    if (process.env.NODE_ENV !== 'development') return;
    
    const logEntry = this.createLogEntry('verbose', message);
    if (context) logEntry.context = context;
    
    console.log(
      `[💬 ${logEntry.timestamp}] [${logEntry.context || 'App'}] ${logEntry.message}`
    );
    
    this.writeToFile(logEntry);
  }
}
```

---

### **Step 4: Create Logger Module (Recommended)**

```typescript
// logger.module.ts
import { Module, Global } from '@nestjs/common';
import { AdvancedLogger } from './advanced-logger.service';

@Global() // Make logger available everywhere
@Module({
  providers: [AdvancedLogger],
  exports: [AdvancedLogger],
})
export class LoggerModule {}

// app.module.ts
import { Module } from '@nestjs/common';
import { LoggerModule } from './logger/logger.module';

@Module({
  imports: [LoggerModule],
  // Now AdvancedLogger is available in all modules
})
export class AppModule {}

// Usage in any service
import { Injectable } from '@nestjs/common';
import { AdvancedLogger } from './logger/advanced-logger.service';

@Injectable()
export class UserService {
  constructor(private readonly logger: AdvancedLogger) {
    this.logger.setContext(UserService.name);
  }

  async createUser(email: string) {
    this.logger.log(`Creating user: ${email}`);
    // Custom logger handles console + file logging automatically
  }
}
```

---

### **Step 5: Production-Grade Logger with External Service**

```typescript
// Install: npm install axios
import { LoggerService, Injectable } from '@nestjs/common';
import axios from 'axios';

interface ExternalLogService {
  apiKey: string;
  endpoint: string;
}

@Injectable()
export class ProductionLogger implements LoggerService {
  private externalService: ExternalLogService;
  private buffer: any[] = [];
  private flushInterval: NodeJS.Timeout;

  constructor() {
    this.externalService = {
      apiKey: process.env.LOG_API_KEY || '',
      endpoint: process.env.LOG_ENDPOINT || 'https://logs.example.com/api/logs',
    };

    // Flush logs every 5 seconds
    this.flushInterval = setInterval(() => this.flushLogs(), 5000);
  }

  private async flushLogs() {
    if (this.buffer.length === 0) return;

    const logsToSend = [...this.buffer];
    this.buffer = [];

    try {
      await axios.post(this.externalService.endpoint, {
        logs: logsToSend,
        apiKey: this.externalService.apiKey,
      });
    } catch (error) {
      // Fallback: write to console if external service fails
      console.error('Failed to send logs to external service:', error.message);
      // Re-add failed logs to buffer
      this.buffer.push(...logsToSend);
    }
  }

  private addToBuffer(level: string, message: any, context?: string, trace?: string) {
    this.buffer.push({
      timestamp: new Date().toISOString(),
      level,
      context,
      message,
      trace,
      pid: process.pid,
      hostname: require('os').hostname(),
    });
  }

  log(message: any, context?: string) {
    console.log(`[${context || 'App'}]`, message);
    this.addToBuffer('log', message, context);
  }

  error(message: any, trace?: string, context?: string) {
    console.error(`[${context || 'App'}]`, message, trace);
    this.addToBuffer('error', message, context, trace);
    
    // Immediately flush errors (don't wait for interval)
    this.flushLogs();
  }

  warn(message: any, context?: string) {
    console.warn(`[${context || 'App'}]`, message);
    this.addToBuffer('warn', message, context);
  }

  debug(message: any, context?: string) {
    if (process.env.NODE_ENV !== 'production') {
      console.debug(`[${context || 'App'}]`, message);
    }
    this.addToBuffer('debug', message, context);
  }

  verbose(message: any, context?: string) {
    if (process.env.NODE_ENV === 'development') {
      console.log(`[${context || 'App'}]`, message);
    }
    this.addToBuffer('verbose', message, context);
  }

  // Clean up interval on shutdown
  onModuleDestroy() {
    clearInterval(this.flushInterval);
    this.flushLogs(); // Final flush
  }
}
```

---

### **Step 6: Testing Custom Logger**

```typescript
// custom-logger.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { CustomLogger } from './custom-logger.service';

describe('CustomLogger', () => {
  let logger: CustomLogger;
  let consoleLogSpy: jest.SpyInstance;
  let consoleErrorSpy: jest.SpyInstance;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [CustomLogger],
    }).compile();

    logger = module.get<CustomLogger>(CustomLogger);
    
    // Spy on console methods
    consoleLogSpy = jest.spyOn(console, 'log').mockImplementation();
    consoleErrorSpy = jest.spyOn(console, 'error').mockImplementation();
  });

  afterEach(() => {
    jest.clearAllMocks();
  });

  it('should be defined', () => {
    expect(logger).toBeDefined();
  });

  it('should log messages', () => {
    logger.log('Test message', 'TestContext');
    
    expect(consoleLogSpy).toHaveBeenCalledWith(
      expect.stringContaining('Test message')
    );
  });

  it('should log errors with stack trace', () => {
    const error = new Error('Test error');
    logger.error('Error occurred', error.stack, 'TestContext');
    
    expect(consoleErrorSpy).toHaveBeenCalled();
  });

  it('should set and use context', () => {
    logger.setContext('UserService');
    logger.log('User created');
    
    expect(consoleLogSpy).toHaveBeenCalledWith(
      expect.stringContaining('UserService')
    );
  });
});
```

---

### **Complete Example - E-Commerce Logger:**

```typescript
// ecommerce-logger.service.ts
import { LoggerService, Injectable } from '@nestjs/common';
import * as fs from 'fs';
import * as path from 'path';

enum LogLevel {
  ERROR = 0,
  WARN = 1,
  LOG = 2,
  DEBUG = 3,
  VERBOSE = 4,
}

@Injectable()
export class EcommerceLogger implements LoggerService {
  private context?: string;
  private minLevel: LogLevel;
  private logDir: string;

  constructor() {
    // Set minimum log level based on environment
    this.minLevel = this.getMinLogLevel();
    
    // Setup log directory
    this.logDir = path.join(process.cwd(), 'logs');
    this.ensureLogDirectory();
  }

  private getMinLogLevel(): LogLevel {
    const env = process.env.NODE_ENV;
    switch (env) {
      case 'production':
        return LogLevel.WARN;
      case 'staging':
        return LogLevel.LOG;
      default:
        return LogLevel.VERBOSE;
    }
  }

  private ensureLogDirectory() {
    if (!fs.existsSync(this.logDir)) {
      fs.mkdirSync(this.logDir, { recursive: true });
    }
  }

  private shouldLog(level: LogLevel): boolean {
    return level <= this.minLevel;
  }

  private writeToFile(level: string, message: string, context?: string) {
    const date = new Date().toISOString().split('T')[0];
    const filename = path.join(this.logDir, `${level}-${date}.log`);
    
    const logEntry = {
      timestamp: new Date().toISOString(),
      level,
      context: context || this.context,
      message,
    };

    fs.appendFileSync(filename, JSON.stringify(logEntry) + '\n');
  }

  setContext(context: string) {
    this.context = context;
  }

  log(message: any, context?: string) {
    if (!this.shouldLog(LogLevel.LOG)) return;
    
    const ctx = context || this.context || 'App';
    console.log(`[✅ LOG] [${ctx}] ${message}`);
    this.writeToFile('info', message, ctx);
  }

  error(message: any, trace?: string, context?: string) {
    if (!this.shouldLog(LogLevel.ERROR)) return;
    
    const ctx = context || this.context || 'App';
    console.error(`[❌ ERROR] [${ctx}] ${message}`);
    if (trace) console.error(trace);
    
    this.writeToFile('error', `${message}${trace ? ` | ${trace}` : ''}`, ctx);
  }

  warn(message: any, context?: string) {
    if (!this.shouldLog(LogLevel.WARN)) return;
    
    const ctx = context || this.context || 'App';
    console.warn(`[⚠️  WARN] [${ctx}] ${message}`);
    this.writeToFile('warn', message, ctx);
  }

  debug(message: any, context?: string) {
    if (!this.shouldLog(LogLevel.DEBUG)) return;
    
    const ctx = context || this.context || 'App';
    console.debug(`[🐞 DEBUG] [${ctx}] ${message}`);
    this.writeToFile('debug', message, ctx);
  }

  verbose(message: any, context?: string) {
    if (!this.shouldLog(LogLevel.VERBOSE)) return;
    
    const ctx = context || this.context || 'App';
    console.log(`[💬 VERBOSE] [${ctx}] ${message}`);
    this.writeToFile('verbose', message, ctx);
  }
}

// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { EcommerceLogger } from './logger/ecommerce-logger.service';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, {
    logger: new EcommerceLogger(),
  });

  await app.listen(3000);
}

bootstrap();
```

---

### **Key Takeaways:**

1. **Implement `LoggerService`** interface with all 5 methods
2. **Add features** like file logging, external services, buffering
3. **Environment-aware** behavior (verbose in dev, minimal in prod)
4. **Use `@Global()` module** to make logger available everywhere
5. **Test thoroughly** to ensure reliability
6. **Handle failures** gracefully (fallback to console if external service fails)

**When to Create Custom Logger:**
- Need file logging
- Integrate with external service (Datadog, Sentry)
- Custom formatting requirements
- Special compliance needs
- Performance optimization

</details>

<details>
<summary><strong>11. What is the `LoggerService` interface?</strong></summary>

**Answer:**

The `LoggerService` interface is a contract in NestJS that defines the structure for creating custom loggers. Any class implementing this interface can be used as a logger in NestJS applications, ensuring compatibility with the framework's logging system.

---

### **Interface Definition:**

```typescript
// From @nestjs/common
export interface LoggerService {
  /**
   * Write a 'log' level log.
   */
  log(message: any, ...optionalParams: any[]): any;

  /**
   * Write an 'error' level log.
   */
  error(message: any, ...optionalParams: any[]): any;

  /**
   * Write a 'warn' level log.
   */
  warn(message: any, ...optionalParams: any[]): any;

  /**
   * Write a 'debug' level log (optional).
   */
  debug?(message: any, ...optionalParams: any[]): any;

  /**
   * Write a 'verbose' level log (optional).
   */
  verbose?(message: any, ...optionalParams: any[]): any;
}
```

**Key Points:**
- **Required methods**: `log()`, `error()`, `warn()`
- **Optional methods**: `debug()`, `verbose()`
- **Flexible parameters**: Accept any type of message and additional params
- **No return type restriction**: Can return anything (usually void)

---

### **Basic Implementation:**

```typescript
import { LoggerService } from '@nestjs/common';

export class BasicLogger implements LoggerService {
  log(message: any, ...optionalParams: any[]) {
    console.log('[LOG]', message, ...optionalParams);
  }

  error(message: any, ...optionalParams: any[]) {
    console.error('[ERROR]', message, ...optionalParams);
  }

  warn(message: any, ...optionalParams: any[]) {
    console.warn('[WARN]', message, ...optionalParams);
  }

  debug(message: any, ...optionalParams: any[]) {
    console.debug('[DEBUG]', message, ...optionalParams);
  }

  verbose(message: any, ...optionalParams: any[]) {
    console.log('[VERBOSE]', message, ...optionalParams);
  }
}
```

---

### **Production Implementation with Features:**

```typescript
import { LoggerService, Injectable } from '@nestjs/common';

interface LogContext {
  requestId?: string;
  userId?: string;
  service?: string;
}

@Injectable()
export class ProductionLogger implements LoggerService {
  private context: LogContext = {};

  /**
   * Set contextual information for logs
   */
  setContext(context: LogContext) {
    this.context = { ...this.context, ...context };
  }

  /**
   * Clear context (useful for request cleanup)
   */
  clearContext() {
    this.context = {};
  }

  /**
   * Format log entry with timestamp and context
   */
  private formatMessage(level: string, message: any, ...optionalParams: any[]): string {
    const timestamp = new Date().toISOString();
    const contextStr = Object.keys(this.context).length 
      ? JSON.stringify(this.context) 
      : '';
    
    const params = optionalParams.length 
      ? ' ' + optionalParams.map(p => 
          typeof p === 'object' ? JSON.stringify(p) : p
        ).join(' ')
      : '';

    return `[${timestamp}] [${level}] ${contextStr} ${message}${params}`;
  }

  log(message: any, ...optionalParams: any[]) {
    console.log(this.formatMessage('LOG', message, ...optionalParams));
  }

  error(message: any, ...optionalParams: any[]) {
    console.error(this.formatMessage('ERROR', message, ...optionalParams));
  }

  warn(message: any, ...optionalParams: any[]) {
    console.warn(this.formatMessage('WARN', message, ...optionalParams));
  }

  debug(message: any, ...optionalParams: any[]) {
    if (process.env.NODE_ENV !== 'production') {
      console.debug(this.formatMessage('DEBUG', message, ...optionalParams));
    }
  }

  verbose(message: any, ...optionalParams: any[]) {
    if (process.env.NODE_ENV === 'development') {
      console.log(this.formatMessage('VERBOSE', message, ...optionalParams));
    }
  }
}

// Usage
const logger = new ProductionLogger();
logger.setContext({ service: 'UserService', requestId: 'req-123' });
logger.log('User created');
// Output: [2025-12-30T10:30:45.123Z] [LOG] {"service":"UserService","requestId":"req-123"} User created
```

---

### **Integration with External Services:**

```typescript
// Install: npm install @sentry/node
import { LoggerService } from '@nestjs/common';
import * as Sentry from '@sentry/node';

export class SentryLogger implements LoggerService {
  constructor() {
    // Initialize Sentry
    Sentry.init({
      dsn: process.env.SENTRY_DSN,
      environment: process.env.NODE_ENV,
    });
  }

  log(message: any, ...optionalParams: any[]) {
    console.log(message, ...optionalParams);
    // Optionally send info to Sentry
    Sentry.captureMessage(String(message), 'info');
  }

  error(message: any, ...optionalParams: any[]) {
    console.error(message, ...optionalParams);
    
    // Send errors to Sentry with full context
    if (message instanceof Error) {
      Sentry.captureException(message);
    } else {
      Sentry.captureMessage(String(message), 'error');
    }
  }

  warn(message: any, ...optionalParams: any[]) {
    console.warn(message, ...optionalParams);
    Sentry.captureMessage(String(message), 'warning');
  }

  debug(message: any, ...optionalParams: any[]) {
    if (process.env.NODE_ENV !== 'production') {
      console.debug(message, ...optionalParams);
    }
  }

  verbose(message: any, ...optionalParams: any[]) {
    if (process.env.NODE_ENV === 'development') {
      console.log(message, ...optionalParams);
    }
  }
}

// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { SentryLogger } from './sentry-logger';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, {
    logger: new SentryLogger(),
  });

  await app.listen(3000);
}
bootstrap();
```

---

### **File Logging Implementation:**

```typescript
import { LoggerService } from '@nestjs/common';
import * as fs from 'fs';
import * as path from 'path';

export class FileLogger implements LoggerService {
  private logStream: fs.WriteStream;
  private errorStream: fs.WriteStream;

  constructor() {
    const logsDir = path.join(process.cwd(), 'logs');
    
    // Create logs directory
    if (!fs.existsSync(logsDir)) {
      fs.mkdirSync(logsDir, { recursive: true });
    }

    const date = new Date().toISOString().split('T')[0];
    
    // Create separate streams for general and error logs
    this.logStream = fs.createWriteStream(
      path.join(logsDir, `app-${date}.log`),
      { flags: 'a' } // Append mode
    );
    
    this.errorStream = fs.createWriteStream(
      path.join(logsDir, `error-${date}.log`),
      { flags: 'a' }
    );
  }

  private writeLog(stream: fs.WriteStream, level: string, message: any, ...optionalParams: any[]) {
    const timestamp = new Date().toISOString();
    const logEntry = {
      timestamp,
      level,
      message: String(message),
      ...(optionalParams.length && { metadata: optionalParams }),
    };

    stream.write(JSON.stringify(logEntry) + '\n');
  }

  log(message: any, ...optionalParams: any[]) {
    console.log(message, ...optionalParams);
    this.writeLog(this.logStream, 'LOG', message, ...optionalParams);
  }

  error(message: any, ...optionalParams: any[]) {
    console.error(message, ...optionalParams);
    // Write to both general and error-specific log
    this.writeLog(this.logStream, 'ERROR', message, ...optionalParams);
    this.writeLog(this.errorStream, 'ERROR', message, ...optionalParams);
  }

  warn(message: any, ...optionalParams: any[]) {
    console.warn(message, ...optionalParams);
    this.writeLog(this.logStream, 'WARN', message, ...optionalParams);
  }

  debug(message: any, ...optionalParams: any[]) {
    console.debug(message, ...optionalParams);
    this.writeLog(this.logStream, 'DEBUG', message, ...optionalParams);
  }

  verbose(message: any, ...optionalParams: any[]) {
    console.log(message, ...optionalParams);
    this.writeLog(this.logStream, 'VERBOSE', message, ...optionalParams);
  }

  // Cleanup method
  close() {
    this.logStream.end();
    this.errorStream.end();
  }
}
```

---

### **Multi-Transport Logger:**

```typescript
import { LoggerService } from '@nestjs/common';

interface Transport {
  log(level: string, message: any, ...optionalParams: any[]): void;
}

class ConsoleTransport implements Transport {
  log(level: string, message: any, ...optionalParams: any[]) {
    const method = level === 'error' ? console.error : 
                   level === 'warn' ? console.warn : 
                   level === 'debug' ? console.debug : console.log;
    method(`[${level.toUpperCase()}]`, message, ...optionalParams);
  }
}

class FileTransport implements Transport {
  private fs = require('fs');
  private path = require('path');
  
  log(level: string, message: any, ...optionalParams: any[]) {
    const logFile = this.path.join(process.cwd(), 'logs', 'app.log');
    const logEntry = JSON.stringify({
      timestamp: new Date().toISOString(),
      level,
      message,
      metadata: optionalParams,
    }) + '\n';
    
    this.fs.appendFileSync(logFile, logEntry);
  }
}

export class MultiTransportLogger implements LoggerService {
  private transports: Transport[] = [];

  constructor(transports?: Transport[]) {
    this.transports = transports || [new ConsoleTransport()];
  }

  addTransport(transport: Transport) {
    this.transports.push(transport);
  }

  private broadcast(level: string, message: any, ...optionalParams: any[]) {
    this.transports.forEach(transport => {
      transport.log(level, message, ...optionalParams);
    });
  }

  log(message: any, ...optionalParams: any[]) {
    this.broadcast('log', message, ...optionalParams);
  }

  error(message: any, ...optionalParams: any[]) {
    this.broadcast('error', message, ...optionalParams);
  }

  warn(message: any, ...optionalParams: any[]) {
    this.broadcast('warn', message, ...optionalParams);
  }

  debug(message: any, ...optionalParams: any[]) {
    this.broadcast('debug', message, ...optionalParams);
  }

  verbose(message: any, ...optionalParams: any[]) {
    this.broadcast('verbose', message, ...optionalParams);
  }
}

// Usage
const logger = new MultiTransportLogger([
  new ConsoleTransport(),
  new FileTransport(),
]);

logger.log('Application started'); // Logs to both console and file
```

---

### **Async Logger (Non-Blocking):**

```typescript
import { LoggerService } from '@nestjs/common';

export class AsyncLogger implements LoggerService {
  private queue: Array<{ level: string; message: any; params: any[] }> = [];
  private processing = false;

  private async processQueue() {
    if (this.processing || this.queue.length === 0) return;
    
    this.processing = true;

    while (this.queue.length > 0) {
      const log = this.queue.shift();
      if (!log) break;

      // Simulate async operation (e.g., network call)
      await this.sendToExternalService(log);
    }

    this.processing = false;
  }

  private async sendToExternalService(log: any) {
    // Simulate external logging service
    return new Promise((resolve) => {
      setTimeout(() => {
        console.log('[ASYNC]', log.level, log.message);
        resolve(true);
      }, 10);
    });
  }

  private enqueue(level: string, message: any, ...params: any[]) {
    this.queue.push({ level, message, params });
    this.processQueue(); // Non-blocking
  }

  log(message: any, ...optionalParams: any[]) {
    this.enqueue('LOG', message, ...optionalParams);
  }

  error(message: any, ...optionalParams: any[]) {
    this.enqueue('ERROR', message, ...optionalParams);
  }

  warn(message: any, ...optionalParams: any[]) {
    this.enqueue('WARN', message, ...optionalParams);
  }

  debug(message: any, ...optionalParams: any[]) {
    this.enqueue('DEBUG', message, ...optionalParams);
  }

  verbose(message: any, ...optionalParams: any[]) {
    this.enqueue('VERBOSE', message, ...optionalParams);
  }
}
```

---

### **Testing LoggerService Implementation:**

```typescript
import { Test } from '@nestjs/testing';
import { ProductionLogger } from './production-logger';

describe('ProductionLogger', () => {
  let logger: ProductionLogger;
  let consoleLogSpy: jest.SpyInstance;
  let consoleErrorSpy: jest.SpyInstance;

  beforeEach(() => {
    logger = new ProductionLogger();
    consoleLogSpy = jest.spyOn(console, 'log').mockImplementation();
    consoleErrorSpy = jest.spyOn(console, 'error').mockImplementation();
  });

  afterEach(() => {
    jest.restoreAllMocks();
  });

  it('should implement LoggerService interface', () => {
    expect(logger.log).toBeDefined();
    expect(logger.error).toBeDefined();
    expect(logger.warn).toBeDefined();
    expect(typeof logger.log).toBe('function');
    expect(typeof logger.error).toBe('function');
  });

  it('should log messages with context', () => {
    logger.setContext({ service: 'TestService' });
    logger.log('Test message');

    expect(consoleLogSpy).toHaveBeenCalledWith(
      expect.stringContaining('TestService')
    );
  });

  it('should handle multiple parameters', () => {
    logger.log('Message', 'param1', { key: 'value' });

    expect(consoleLogSpy).toHaveBeenCalled();
  });

  it('should log errors', () => {
    const error = new Error('Test error');
    logger.error('Error occurred', error.stack);

    expect(consoleErrorSpy).toHaveBeenCalled();
  });
});
```

---

### **Key Benefits:**

```typescript
// ✅ Flexibility - Use any logging library
export class WinstonLogger implements LoggerService {
  // Wrap Winston with LoggerService interface
}

// ✅ Testability - Easy to mock
const mockLogger: LoggerService = {
  log: jest.fn(),
  error: jest.fn(),
  warn: jest.fn(),
};

// ✅ Consistency - Same interface everywhere
class ServiceA {
  constructor(private logger: LoggerService) {}
}

class ServiceB {
  constructor(private logger: LoggerService) {}
}

// ✅ Swappability - Change implementation without changing code
const app = await NestFactory.create(AppModule, {
  logger: isDev ? new DevLogger() : new ProdLogger(),
});
```

---

### **When to Implement LoggerService:**

| Scenario | Should Implement? | Reason |
|----------|-------------------|--------|
| Custom log format | ✅ Yes | Full control over output |
| External service integration | ✅ Yes | Datadog, Sentry, CloudWatch |
| File logging | ✅ Yes | Persistent logs |
| Performance optimization | ✅ Yes | Async, batching, buffering |
| Multiple transports | ✅ Yes | Console + File + External |
| Simple app | ❌ No | Built-in Logger is sufficient |

**Key Takeaway:** The `LoggerService` interface provides a contract for building custom loggers while maintaining compatibility with NestJS's dependency injection and framework features.

</details>

<details>
<summary><strong>12. How do you implement custom log formatting?</strong></summary>

**Answer:**

Custom log formatting allows you to control exactly how logs appear, add metadata, structure data for parsing, and integrate with external systems. You can implement custom formatting by creating a logger that implements `LoggerService` or extends the built-in `Logger` class.

---

### **Method 1: Simple String Formatting**

```typescript
import { LoggerService } from '@nestjs/common';

export class SimpleFormattedLogger implements LoggerService {
  private format(
    level: string,
    message: string,
    context?: string
  ): string {
    const timestamp = new Date().toISOString();
    const ctx = context ? `[${context}]` : '';
    return `${timestamp} | ${level.toUpperCase().padEnd(7)} | ${ctx} ${message}`;
  }

  log(message: any, context?: string) {
    console.log(this.format('log', message, context));
  }

  error(message: any, trace?: string, context?: string) {
    console.error(this.format('error', message, context));
    if (trace) {
      console.error(trace);
    }
  }

  warn(message: any, context?: string) {
    console.warn(this.format('warn', message, context));
  }

  debug(message: any, context?: string) {
    console.debug(this.format('debug', message, context));
  }

  verbose(message: any, context?: string) {
    console.log(this.format('verbose', message, context));
  }
}
```

**Output:**
```
2025-12-30T10:30:45.123Z | LOG     | [UserService] User created successfully
2025-12-30T10:30:46.456Z | ERROR   | [PaymentService] Payment processing failed
2025-12-30T10:30:47.789Z | WARN    | [CacheService] Cache unavailable
```

---

### **Method 2: JSON Structured Format (Production)**

```typescript
import { LoggerService } from '@nestjs/common';

interface StructuredLog {
  timestamp: string;
  level: string;
  message: string;
  context?: string;
  metadata?: Record<string, any>;
  trace?: string;
  environment: string;
  hostname: string;
  pid: number;
}

export class JsonFormattedLogger implements LoggerService {
  private readonly hostname: string;
  private readonly environment: string;

  constructor() {
    this.hostname = require('os').hostname();
    this.environment = process.env.NODE_ENV || 'development';
  }

  private formatJson(
    level: string,
    message: any,
    context?: string,
    trace?: string,
    metadata?: Record<string, any>
  ): string {
    const log: StructuredLog = {
      timestamp: new Date().toISOString(),
      level: level.toUpperCase(),
      message: this.stringifyMessage(message),
      environment: this.environment,
      hostname: this.hostname,
      pid: process.pid,
      ...(context && { context }),
      ...(trace && { trace }),
      ...(metadata && Object.keys(metadata).length > 0 && { metadata }),
    };

    return JSON.stringify(log);
  }

  private stringifyMessage(message: any): string {
    if (typeof message === 'string') return message;
    if (message instanceof Error) return message.message;
    try {
      return JSON.stringify(message);
    } catch {
      return String(message);
    }
  }

  log(message: any, context?: string) {
    console.log(this.formatJson('info', message, context));
  }

  error(message: any, trace?: string, context?: string) {
    console.error(this.formatJson('error', message, context, trace));
  }

  warn(message: any, context?: string) {
    console.warn(this.formatJson('warn', message, context));
  }

  debug(message: any, context?: string) {
    if (this.environment !== 'production') {
      console.debug(this.formatJson('debug', message, context));
    }
  }

  verbose(message: any, context?: string) {
    if (this.environment === 'development') {
      console.log(this.formatJson('verbose', message, context));
    }
  }
}
```

**Output:**
```json
{"timestamp":"2025-12-30T10:30:45.123Z","level":"INFO","message":"User created successfully","environment":"production","hostname":"api-server-01","pid":12345,"context":"UserService"}
{"timestamp":"2025-12-30T10:30:46.456Z","level":"ERROR","message":"Payment processing failed","environment":"production","hostname":"api-server-01","pid":12345,"context":"PaymentService","trace":"Error: Payment failed\n    at..."}
```

---

### **Method 3: Colorized Console Format (Development)**

```typescript
// Install: npm install chalk
import { LoggerService } from '@nestjs/common';
import chalk from 'chalk';

export class ColorizedLogger implements LoggerService {
  private formatWithColor(
    level: string,
    message: string,
    context?: string
  ): string {
    const timestamp = chalk.gray(new Date().toLocaleTimeString());
    const ctx = context ? chalk.cyan(`[${context}]`) : '';
    
    let levelStr: string;
    switch (level.toLowerCase()) {
      case 'error':
        levelStr = chalk.red.bold('ERROR  ');
        break;
      case 'warn':
        levelStr = chalk.yellow.bold('WARN   ');
        break;
      case 'log':
        levelStr = chalk.green.bold('LOG    ');
        break;
      case 'debug':
        levelStr = chalk.blue.bold('DEBUG  ');
        break;
      case 'verbose':
        levelStr = chalk.magenta.bold('VERBOSE');
        break;
      default:
        levelStr = level.toUpperCase();
    }

    return `${timestamp} ${levelStr} ${ctx} ${message}`;
  }

  log(message: any, context?: string) {
    console.log(this.formatWithColor('log', String(message), context));
  }

  error(message: any, trace?: string, context?: string) {
    console.error(this.formatWithColor('error', String(message), context));
    if (trace) {
      console.error(chalk.red(trace));
    }
  }

  warn(message: any, context?: string) {
    console.warn(this.formatWithColor('warn', String(message), context));
  }

  debug(message: any, context?: string) {
    console.debug(this.formatWithColor('debug', String(message), context));
  }

  verbose(message: any, context?: string) {
    console.log(this.formatWithColor('verbose', String(message), context));
  }
}
```

**Output (with colors):**
```
10:30:45 AM LOG     [UserService] User created successfully
10:30:46 AM ERROR   [PaymentService] Payment processing failed
10:30:47 AM WARN    [CacheService] Cache unavailable
```

---

### **Method 4: Multi-Line Pretty Format**

```typescript
import { LoggerService } from '@nestjs/common';

export class PrettyFormattedLogger implements LoggerService {
  private formatPretty(
    level: string,
    message: any,
    context?: string,
    trace?: string
  ): string {
    const lines: string[] = [];
    const border = '═'.repeat(80);
    const timestamp = new Date().toLocaleString();

    lines.push(`\n╔${border}╗`);
    lines.push(`║ ${level.toUpperCase().padEnd(78)} ║`);
    lines.push(`║ ${timestamp.padEnd(78)} ║`);
    
    if (context) {
      lines.push(`║ Context: ${context.padEnd(69)} ║`);
    }
    
    lines.push(`╠${border}╣`);
    
    // Handle message (wrap if too long)
    const messageStr = String(message);
    const messageLines = this.wrapText(messageStr, 76);
    messageLines.forEach(line => {
      lines.push(`║ ${line.padEnd(78)} ║`);
    });
    
    if (trace) {
      lines.push(`╠${border}╣`);
      lines.push(`║ Stack Trace:${' '.repeat(66)} ║`);
      const traceLines = trace.split('\n').slice(0, 5); // First 5 lines
      traceLines.forEach(line => {
        const truncated = line.substring(0, 76);
        lines.push(`║ ${truncated.padEnd(78)} ║`);
      });
    }
    
    lines.push(`╚${border}╝`);
    
    return lines.join('\n');
  }

  private wrapText(text: string, maxWidth: number): string[] {
    const words = text.split(' ');
    const lines: string[] = [];
    let currentLine = '';

    words.forEach(word => {
      if ((currentLine + word).length > maxWidth) {
        lines.push(currentLine.trim());
        currentLine = word + ' ';
      } else {
        currentLine += word + ' ';
      }
    });

    if (currentLine.trim()) {
      lines.push(currentLine.trim());
    }

    return lines;
  }

  log(message: any, context?: string) {
    console.log(this.formatPretty('log', message, context));
  }

  error(message: any, trace?: string, context?: string) {
    console.error(this.formatPretty('error', message, context, trace));
  }

  warn(message: any, context?: string) {
    console.warn(this.formatPretty('warn', message, context));
  }

  debug(message: any, context?: string) {
    console.debug(this.formatPretty('debug', message, context));
  }

  verbose(message: any, context?: string) {
    console.log(this.formatPretty('verbose', message, context));
  }
}
```

**Output:**
```
╔════════════════════════════════════════════════════════════════════════════════╗
║ ERROR                                                                          ║
║ 12/30/2025, 10:30:45 AM                                                        ║
║ Context: PaymentService                                                        ║
╠════════════════════════════════════════════════════════════════════════════════╣
║ Payment processing failed for order ORD-123                                    ║
╠════════════════════════════════════════════════════════════════════════════════╣
║ Stack Trace:                                                                   ║
║ Error: Card declined                                                           ║
║     at PaymentGateway.charge (payment-gateway.ts:45:11)                        ║
║     at PaymentService.process (payment.service.ts:23:5)                        ║
╚════════════════════════════════════════════════════════════════════════════════╝
```

---

### **Method 5: Custom Format with Template**

```typescript
import { LoggerService } from '@nestjs/common';

type LogTemplate = (data: {
  timestamp: Date;
  level: string;
  message: string;
  context?: string;
  trace?: string;
}) => string;

export class TemplateFormattedLogger implements LoggerService {
  constructor(private template: LogTemplate) {}

  private format(
    level: string,
    message: any,
    context?: string,
    trace?: string
  ): string {
    return this.template({
      timestamp: new Date(),
      level,
      message: String(message),
      context,
      trace,
    });
  }

  log(message: any, context?: string) {
    console.log(this.format('log', message, context));
  }

  error(message: any, trace?: string, context?: string) {
    console.error(this.format('error', message, context, trace));
  }

  warn(message: any, context?: string) {
    console.warn(this.format('warn', message, context));
  }

  debug(message: any, context?: string) {
    console.debug(this.format('debug', message, context));
  }

  verbose(message: any, context?: string) {
    console.log(this.format('verbose', message, context));
  }
}

// Usage with different templates

// Template 1: Compact format
const compactTemplate: LogTemplate = ({ timestamp, level, message, context }) => {
  return `${timestamp.toISOString()} [${level.toUpperCase()}] ${context || 'App'}: ${message}`;
};

// Template 2: Verbose format
const verboseTemplate: LogTemplate = ({ timestamp, level, message, context, trace }) => {
  let output = `Time: ${timestamp.toLocaleString()}\n`;
  output += `Level: ${level.toUpperCase()}\n`;
  output += `Context: ${context || 'Application'}\n`;
  output += `Message: ${message}\n`;
  if (trace) output += `Trace: ${trace}\n`;
  return output;
};

// Template 3: Syslog format
const syslogTemplate: LogTemplate = ({ timestamp, level, message, context }) => {
  const priority = level === 'error' ? 3 : level === 'warn' ? 4 : 6;
  return `<${priority}>${timestamp.toISOString()} ${require('os').hostname()} ${context || 'app'}[${process.pid}]: ${message}`;
};

// Create loggers with different templates
const compactLogger = new TemplateFormattedLogger(compactTemplate);
const verboseLogger = new TemplateFormattedLogger(verboseTemplate);
const syslogLogger = new TemplateFormattedLogger(syslogTemplate);
```

---

### **Method 6: Format with Metadata Enrichment**

```typescript
import { LoggerService } from '@nestjs/common';

interface LogMetadata {
  requestId?: string;
  userId?: string;
  sessionId?: string;
  ipAddress?: string;
  userAgent?: string;
  [key: string]: any;
}

export class MetadataEnrichedLogger implements LoggerService {
  private metadata: LogMetadata = {};

  setMetadata(metadata: LogMetadata) {
    this.metadata = { ...this.metadata, ...metadata };
  }

  clearMetadata() {
    this.metadata = {};
  }

  private format(
    level: string,
    message: any,
    context?: string,
    trace?: string
  ): string {
    const log = {
      '@timestamp': new Date().toISOString(),
      level: level.toUpperCase(),
      message: String(message),
      context: context || 'Application',
      ...this.metadata, // Add all metadata
      ...(trace && { trace }),
      // Add system info
      system: {
        hostname: require('os').hostname(),
        pid: process.pid,
        platform: process.platform,
        nodeVersion: process.version,
      },
    };

    return JSON.stringify(log);
  }

  log(message: any, context?: string) {
    console.log(this.format('log', message, context));
  }

  error(message: any, trace?: string, context?: string) {
    console.error(this.format('error', message, context, trace));
  }

  warn(message: any, context?: string) {
    console.warn(this.format('warn', message, context));
  }

  debug(message: any, context?: string) {
    console.debug(this.format('debug', message, context));
  }

  verbose(message: any, context?: string) {
    console.log(this.format('verbose', message, context));
  }
}

// Usage in interceptor
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  constructor(private logger: MetadataEnrichedLogger) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    
    // Enrich logs with request metadata
    this.logger.setMetadata({
      requestId: request.id || this.generateId(),
      userId: request.user?.id,
      ipAddress: request.ip,
      userAgent: request.headers['user-agent'],
      method: request.method,
      url: request.url,
    });

    return next.handle();
  }

  private generateId(): string {
    return `req-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }
}
```

**Output:**
```json
{
  "@timestamp": "2025-12-30T10:30:45.123Z",
  "level": "LOG",
  "message": "User created successfully",
  "context": "UserService",
  "requestId": "req-1703937045-abc123xyz",
  "userId": "user-456",
  "ipAddress": "192.168.1.100",
  "userAgent": "Mozilla/5.0...",
  "method": "POST",
  "url": "/api/users",
  "system": {
    "hostname": "api-server-01",
    "pid": 12345,
    "platform": "linux",
    "nodeVersion": "v18.17.0"
  }
}
```

---

### **Method 7: Environment-Aware Formatter**

```typescript
import { LoggerService } from '@nestjs/common';

export class AdaptiveFormattedLogger implements LoggerService {
  private formatter: (level: string, message: any, context?: string) => string;

  constructor() {
    // Choose formatter based on environment
    this.formatter = this.selectFormatter();
  }

  private selectFormatter() {
    const env = process.env.NODE_ENV;

    switch (env) {
      case 'production':
        // JSON format for production (parseable by log aggregators)
        return (level: string, message: any, context?: string) => {
          return JSON.stringify({
            timestamp: new Date().toISOString(),
            level: level.toUpperCase(),
            message: String(message),
            context,
            pid: process.pid,
          });
        };

      case 'development':
        // Pretty format for development (human-readable)
        return (level: string, message: any, context?: string) => {
          const timestamp = new Date().toLocaleTimeString();
          return `[${timestamp}] [${level.toUpperCase()}] [${context || 'App'}] ${message}`;
        };

      case 'test':
        // Minimal format for tests (less noise)
        return (level: string, message: any) => {
          return `[${level.toUpperCase()}] ${message}`;
        };

      default:
        return (level: string, message: any) => `${level}: ${message}`;
    }
  }

  log(message: any, context?: string) {
    console.log(this.formatter('log', message, context));
  }

  error(message: any, trace?: string, context?: string) {
    console.error(this.formatter('error', message, context));
    if (trace) console.error(trace);
  }

  warn(message: any, context?: string) {
    console.warn(this.formatter('warn', message, context));
  }

  debug(message: any, context?: string) {
    console.debug(this.formatter('debug', message, context));
  }

  verbose(message: any, context?: string) {
    console.log(this.formatter('verbose', message, context));
  }
}
```

---

### **Best Practices:**

```typescript
// ✅ GOOD - JSON for production (machine-parseable)
const prodFormat = (log) => JSON.stringify(log);

// ✅ GOOD - Pretty for development (human-readable)
const devFormat = (log) => `[${log.level}] ${log.message}`;

// ✅ GOOD - Include timestamp always
{ timestamp: new Date().toISOString(), ...log }

// ✅ GOOD - Include context for traceability
{ context: 'UserService', ...log }

// ✅ GOOD - Handle different message types
const message = typeof msg === 'string' ? msg : JSON.stringify(msg);

// ❌ BAD - No timestamp
const log = { level, message }; // When did this happen?

// ❌ BAD - Expensive formatting always executed
logger.debug(expensiveOperation()); // Runs even if debug disabled!

// ❌ BAD - Unstructured in production
console.log(`User ${userId} action ${action}`); // Hard to parse

// ❌ BAD - Synchronous I/O in formatter
fs.writeFileSync('log.txt', message); // Blocks event loop!
```

---

### **Performance Tips:**

```typescript
// Use lazy evaluation for expensive formatting
class OptimizedLogger implements LoggerService {
  debug(message: any, context?: string) {
    if (!this.isDebugEnabled()) return; // Skip if disabled
    
    // Only format if needed
    const formatted = this.format('debug', message, context);
    console.debug(formatted);
  }

  private isDebugEnabled(): boolean {
    return process.env.LOG_LEVEL === 'debug';
  }
}
```

**Key Takeaway:** Choose JSON format for production (parseable by tools), pretty format for development (readable by humans), and always include timestamp, level, context, and relevant metadata.

</details>

## Third-Party Logging Libraries

<details>
<summary><strong>13. What popular logging libraries work with NestJS (Winston, Pino)?</strong></summary>

**Answer:**

NestJS works seamlessly with popular Node.js logging libraries like **Winston** and **Pino**. These libraries offer advanced features like multiple transports, structured logging, high performance, and integration with external logging services.

---

### **Popular Logging Libraries Overview:**

| Library | Strengths | Best For | Performance | Community |
|---------|-----------|----------|-------------|------------|
| **Winston** | Transports, flexibility, rich ecosystem | Complex apps, multiple outputs | Medium | Very large |
| **Pino** | Speed, low overhead, JSON output | High-traffic apps, microservices | Very high | Large |
| **Bunyan** | JSON streaming, CLI tools | Structured logging, debugging | High | Medium |
| **Log4js** | Java-like API, appenders | Teams familiar with Log4j | Medium | Medium |

---

### **1. Winston - Most Popular**

**Features:**
- Multiple transports (console, file, HTTP, databases)
- Custom log levels and colors
- Exception and rejection handling
- Query logs
- Profiling support
- Rich ecosystem of transports

**Installation:**
```bash
npm install winston nest-winston
```

**Basic Setup:**
```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { WinstonModule } from 'nest-winston';
import * as winston from 'winston';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, {
    logger: WinstonModule.createLogger({
      transports: [
        // Console transport with colors
        new winston.transports.Console({
          format: winston.format.combine(
            winston.format.timestamp(),
            winston.format.colorize(),
            winston.format.printf(({ timestamp, level, message, context }) => {
              return `${timestamp} [${context || 'App'}] ${level}: ${message}`;
            }),
          ),
        }),
        
        // File transport for errors
        new winston.transports.File({
          filename: 'logs/error.log',
          level: 'error',
          format: winston.format.combine(
            winston.format.timestamp(),
            winston.format.json(),
          ),
        }),
        
        // File transport for all logs
        new winston.transports.File({
          filename: 'logs/combined.log',
          format: winston.format.combine(
            winston.format.timestamp(),
            winston.format.json(),
          ),
        }),
      ],
    }),
  });

  await app.listen(3000);
}
bootstrap();
```

**Usage in Services:**
```typescript
import { Injectable, Inject } from '@nestjs/common';
import { WINSTON_MODULE_PROVIDER } from 'nest-winston';
import { Logger } from 'winston';

@Injectable()
export class UserService {
  constructor(
    @Inject(WINSTON_MODULE_PROVIDER) private readonly logger: Logger,
  ) {}

  async createUser(userData: any) {
    try {
      this.logger.info('Creating user', {
        context: 'UserService',
        userData: { email: userData.email },
      });

      // Business logic
      const user = await this.userRepository.save(userData);

      this.logger.info('User created successfully', {
        context: 'UserService',
        userId: user.id,
      });

      return user;
    } catch (error) {
      this.logger.error('Failed to create user', {
        context: 'UserService',
        error: error.message,
        stack: error.stack,
      });
      throw error;
    }
  }
}
```

---

### **2. Pino - Fastest Logger**

**Features:**
- Extremely fast (5x-10x faster than Winston)
- Low overhead
- JSON output by default
- Child loggers for context
- Asynchronous logging
- Stream-based architecture

**Installation:**
```bash
npm install nestjs-pino pino-http pino-pretty
```

**Basic Setup:**
```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { LoggerModule } from 'nestjs-pino';

@Module({
  imports: [
    LoggerModule.forRoot({
      pinoHttp: {
        transport: {
          target: 'pino-pretty', // Pretty print in development
          options: {
            colorize: true,
            translateTime: 'SYS:standard',
            ignore: 'pid,hostname',
          },
        },
        level: process.env.LOG_LEVEL || 'info',
        // Custom serializers
        serializers: {
          req: (req) => ({
            method: req.method,
            url: req.url,
            headers: req.headers,
          }),
          res: (res) => ({
            statusCode: res.statusCode,
          }),
        },
      },
    }),
  ],
})
export class AppModule {}
```

**Production Configuration:**
```typescript
// app.module.ts (production)
import { Module } from '@nestjs/common';
import { LoggerModule } from 'nestjs-pino';

@Module({
  imports: [
    LoggerModule.forRoot({
      pinoHttp: {
        level: process.env.NODE_ENV === 'production' ? 'info' : 'debug',
        // Production: JSON output (no pretty print)
        ...(process.env.NODE_ENV === 'production'
          ? {
              formatters: {
                level: (label) => ({ level: label }),
              },
            }
          : {
              transport: {
                target: 'pino-pretty',
                options: { colorize: true },
              },
            }),
        // Add custom context
        customProps: (req, res) => ({
          requestId: req.id,
          userId: req.user?.id,
        }),
        // Redact sensitive data
        redact: ['req.headers.authorization', 'req.body.password'],
      },
    }),
  ],
})
export class AppModule {}
```

**Usage in Services:**
```typescript
import { Injectable } from '@nestjs/common';
import { PinoLogger } from 'nestjs-pino';

@Injectable()
export class OrderService {
  constructor(private readonly logger: PinoLogger) {
    // Set context for this service
    this.logger.setContext(OrderService.name);
  }

  async createOrder(orderData: any) {
    this.logger.info({ orderData }, 'Creating order');

    try {
      const order = await this.orderRepository.save(orderData);
      
      this.logger.info(
        { orderId: order.id, total: order.total },
        'Order created successfully',
      );

      return order;
    } catch (error) {
      this.logger.error(
        { error: error.message, stack: error.stack },
        'Failed to create order',
      );
      throw error;
    }
  }

  async processOrder(orderId: string) {
    // Create child logger with orderId context
    const childLogger = this.logger.logger.child({ orderId });

    childLogger.info('Starting order processing');
    childLogger.info('Validating order');
    childLogger.info('Processing payment');
    childLogger.info('Order processed successfully');
  }
}
```

---

### **3. Bunyan - Structured JSON Logger**

**Features:**
- JSON output for machine parsing
- CLI tool for viewing logs
- Serializers for common objects
- Multiple streams

**Installation:**
```bash
npm install bunyan
```

**Setup:**
```typescript
import { LoggerService } from '@nestjs/common';
import * as bunyan from 'bunyan';

export class BunyanLogger implements LoggerService {
  private logger: bunyan;

  constructor() {
    this.logger = bunyan.createLogger({
      name: 'my-app',
      streams: [
        {
          level: 'info',
          stream: process.stdout,
        },
        {
          level: 'error',
          path: 'logs/error.log',
        },
      ],
      serializers: bunyan.stdSerializers,
    });
  }

  log(message: string, context?: string) {
    this.logger.info({ context }, message);
  }

  error(message: string, trace?: string, context?: string) {
    this.logger.error({ context, trace }, message);
  }

  warn(message: string, context?: string) {
    this.logger.warn({ context }, message);
  }

  debug(message: string, context?: string) {
    this.logger.debug({ context }, message);
  }

  verbose(message: string, context?: string) {
    this.logger.trace({ context }, message);
  }
}
```

---

### **4. Log4js - Java-Style Logger**

**Features:**
- Familiar to Java developers
- Multiple appenders
- Log levels
- Categories

**Installation:**
```bash
npm install log4js
```

**Setup:**
```typescript
import { LoggerService } from '@nestjs/common';
import * as log4js from 'log4js';

export class Log4jsLogger implements LoggerService {
  private logger: log4js.Logger;

  constructor() {
    log4js.configure({
      appenders: {
        console: { type: 'console' },
        file: { type: 'file', filename: 'logs/app.log' },
      },
      categories: {
        default: { appenders: ['console', 'file'], level: 'info' },
      },
    });

    this.logger = log4js.getLogger();
  }

  log(message: string) {
    this.logger.info(message);
  }

  error(message: string, trace?: string) {
    this.logger.error(message, trace);
  }

  warn(message: string) {
    this.logger.warn(message);
  }

  debug(message: string) {
    this.logger.debug(message);
  }

  verbose(message: string) {
    this.logger.trace(message);
  }
}
```

---

### **Comparison: Winston vs Pino**

```typescript
// WINSTON - Rich features, multiple transports
import * as winston from 'winston';

const winstonLogger = winston.createLogger({
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: 'app.log' }),
    new winston.transports.Http({ host: 'log-server.com' }),
  ],
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json(),
  ),
});

// ✅ Pros: Multiple transports, flexible, rich ecosystem
// ❌ Cons: Slower, higher memory usage

// PINO - High performance, simple
import pino from 'pino';

const pinoLogger = pino({
  level: 'info',
  transport: {
    target: 'pino-pretty',
  },
});

// ✅ Pros: Very fast, low overhead, async by default
// ❌ Cons: Fewer built-in transports, JSON-focused
```

---

### **Performance Benchmarks:**

```typescript
// Benchmark results (logs per second)
// 
// Pino:        ~50,000 ops/sec
// Winston:     ~10,000 ops/sec
// Bunyan:      ~30,000 ops/sec
// Console:     ~15,000 ops/sec
//
// Conclusion: Pino is 5x faster than Winston
```

---

### **Production Example with Winston:**

```typescript
// logger.config.ts
import * as winston from 'winston';
import 'winston-daily-rotate-file';

const errorFilter = winston.format((info) => {
  return info.level === 'error' ? info : false;
});

const infoFilter = winston.format((info) => {
  return info.level === 'info' ? info : false;
});

export const winstonConfig = {
  transports: [
    // Console - all levels with colors (dev/staging)
    new winston.transports.Console({
      level: 'debug',
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.timestamp({ format: 'YYYY-MM-DD HH:mm:ss' }),
        winston.format.printf(
          ({ timestamp, level, message, context, ...meta }) => {
            return `${timestamp} [${context || 'App'}] ${level}: ${message} ${Object.keys(meta).length ? JSON.stringify(meta) : ''}`;
          },
        ),
      ),
    }),

    // Error logs - rotate daily
    new winston.transports.DailyRotateFile({
      filename: 'logs/error-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      level: 'error',
      format: winston.format.combine(
        errorFilter(),
        winston.format.timestamp(),
        winston.format.json(),
      ),
      maxSize: '20m',
      maxFiles: '14d', // Keep for 14 days
    }),

    // Info logs - rotate daily
    new winston.transports.DailyRotateFile({
      filename: 'logs/app-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      level: 'info',
      format: winston.format.combine(
        infoFilter(),
        winston.format.timestamp(),
        winston.format.json(),
      ),
      maxSize: '20m',
      maxFiles: '14d',
    }),

    // All logs combined
    new winston.transports.DailyRotateFile({
      filename: 'logs/combined-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.json(),
      ),
      maxSize: '20m',
      maxFiles: '30d', // Keep for 30 days
    }),
  ],
  // Handle uncaught exceptions
  exceptionHandlers: [
    new winston.transports.File({ filename: 'logs/exceptions.log' }),
  ],
  // Handle unhandled promise rejections
  rejectionHandlers: [
    new winston.transports.File({ filename: 'logs/rejections.log' }),
  ],
};

// main.ts
import { NestFactory } from '@nestjs/core';
import { WinstonModule } from 'nest-winston';
import { winstonConfig } from './logger.config';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, {
    logger: WinstonModule.createLogger(winstonConfig),
  });
  await app.listen(3000);
}
```

---

### **Production Example with Pino:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { LoggerModule } from 'nestjs-pino';
import * as path from 'path';

@Module({
  imports: [
    LoggerModule.forRoot({
      pinoHttp: {
        level: process.env.LOG_LEVEL || 'info',
        
        // Production: output JSON to stdout (captured by Docker/K8s)
        ...(process.env.NODE_ENV === 'production'
          ? {
              // No transport in production (raw JSON)
              formatters: {
                level: (label) => ({ level: label }),
                bindings: (bindings) => ({
                  pid: bindings.pid,
                  hostname: bindings.hostname,
                  node_version: process.version,
                }),
              },
            }
          : {
              // Development: pretty print
              transport: {
                target: 'pino-pretty',
                options: {
                  colorize: true,
                  translateTime: 'SYS:standard',
                  ignore: 'pid,hostname',
                },
              },
            }),

        // Add request context
        customProps: (req) => ({
          requestId: req.id,
          userId: req.user?.id,
          ip: req.ip,
        }),

        // Redact sensitive data
        redact: {
          paths: [
            'req.headers.authorization',
            'req.headers.cookie',
            'req.body.password',
            'req.body.creditCard',
            'res.headers["set-cookie"]',
          ],
          remove: true,
        },

        // Custom log level mapping
        customLogLevel: (req, res, err) => {
          if (res.statusCode >= 400 && res.statusCode < 500) {
            return 'warn';
          } else if (res.statusCode >= 500 || err) {
            return 'error';
          }
          return 'info';
        },
      },
    }),
  ],
})
export class AppModule {}
```

---

### **When to Use Which:**

| Use Case | Recommended | Reason |
|----------|-------------|--------|
| High-traffic API | Pino | Performance critical |
| Microservices | Pino | Low overhead, JSON |
| Enterprise app | Winston | Rich features, transports |
| Multiple outputs | Winston | Built-in transports |
| Debugging | Winston | Better formatting options |
| Production logs | Pino | JSON, fast, structured |
| Development | Either | Personal preference |
| Legacy migration | Log4js | Familiar to Java devs |

**Key Takeaway:** Use **Pino** for high-performance applications requiring speed and structured JSON logging. Use **Winston** for complex applications needing multiple transports, custom formatting, and a rich ecosystem.

</details>

<details>
<summary><strong>14. How do you integrate Winston logger?</strong></summary>

**Answer:**

Integrating Winston with NestJS involves installing the necessary packages, configuring transports and formats, and injecting the logger into your services. Winston provides powerful features like multiple transports, custom formats, and error handling.

---

### **Step 1: Installation**

```bash
# Core packages
npm install winston nest-winston

# Optional: Daily rotate file transport
npm install winston-daily-rotate-file

# Optional: Transport for external services
npm install winston-cloudwatch winston-elasticsearch
```

---

### **Step 2: Basic Configuration in main.ts**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { WinstonModule } from 'nest-winston';
import * as winston from 'winston';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, {
    // Replace default logger with Winston
    logger: WinstonModule.createLogger({
      transports: [
        // Console transport
        new winston.transports.Console({
          format: winston.format.combine(
            winston.format.timestamp(),
            winston.format.colorize(),
            winston.format.printf(({ timestamp, level, message, context }) => {
              return `${timestamp} [${context || 'Application'}] ${level}: ${message}`;
            }),
          ),
        }),
        
        // File transport
        new winston.transports.File({
          filename: 'logs/error.log',
          level: 'error',
        }),
      ],
    }),
  });

  await app.listen(3000);
}
bootstrap();
```

---

### **Step 3: Module-Level Configuration**

```typescript
// logger/logger.module.ts
import { Module } from '@nestjs/common';
import { WinstonModule } from 'nest-winston';
import * as winston from 'winston';

const transports: winston.transport[] = [
  // Console with pretty formatting
  new winston.transports.Console({
    format: winston.format.combine(
      winston.format.timestamp({ format: 'YYYY-MM-DD HH:mm:ss' }),
      winston.format.errors({ stack: true }),
      winston.format.splat(),
      winston.format.colorize(),
      winston.format.printf(({ timestamp, level, message, context, ...meta }) => {
        const metaStr = Object.keys(meta).length ? JSON.stringify(meta, null, 2) : '';
        return `${timestamp} [${context || 'App'}] ${level}: ${message} ${metaStr}`;
      }),
    ),
  }),
];

// Add file transports in production
if (process.env.NODE_ENV === 'production') {
  transports.push(
    new winston.transports.File({
      filename: 'logs/error.log',
      level: 'error',
      format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.json(),
      ),
    }),
    new winston.transports.File({
      filename: 'logs/combined.log',
      format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.json(),
      ),
    }),
  );
}

@Module({
  imports: [
    WinstonModule.forRoot({
      transports,
      // Exit on error?
      exitOnError: false,
      // Handle uncaught exceptions
      exceptionHandlers: [
        new winston.transports.File({ filename: 'logs/exceptions.log' }),
      ],
      // Handle unhandled rejections
      rejectionHandlers: [
        new winston.transports.File({ filename: 'logs/rejections.log' }),
      ],
    }),
  ],
  exports: [WinstonModule],
})
export class LoggerModule {}

// app.module.ts
import { Module } from '@nestjs/common';
import { LoggerModule } from './logger/logger.module';

@Module({
  imports: [LoggerModule],
})
export class AppModule {}
```

---

### **Step 4: Inject and Use in Services**

```typescript
import { Injectable, Inject } from '@nestjs/common';
import { WINSTON_MODULE_PROVIDER } from 'nest-winston';
import { Logger } from 'winston';

@Injectable()
export class UserService {
  constructor(
    @Inject(WINSTON_MODULE_PROVIDER)
    private readonly logger: Logger,
  ) {}

  async createUser(createUserDto: any) {
    // Log with context
    this.logger.info('Creating new user', {
      context: UserService.name,
      email: createUserDto.email,
    });

    try {
      const user = await this.userRepository.save(createUserDto);

      this.logger.info('User created successfully', {
        context: UserService.name,
        userId: user.id,
        email: user.email,
      });

      return user;
    } catch (error) {
      this.logger.error('Failed to create user', {
        context: UserService.name,
        error: error.message,
        stack: error.stack,
        dto: createUserDto,
      });
      throw error;
    }
  }

  async deleteUser(userId: string) {
    this.logger.warn('Deleting user', {
      context: UserService.name,
      userId,
    });

    await this.userRepository.delete(userId);
  }
}
```

---

### **Step 5: Custom Winston Configuration File**

```typescript
// config/winston.config.ts
import * as winston from 'winston';
import 'winston-daily-rotate-file';

// Custom log format
const customFormat = winston.format.printf(
  ({ timestamp, level, message, context, ...meta }) => {
    const metaString = Object.keys(meta).length ? JSON.stringify(meta) : '';
    return `[${timestamp}] [${level.toUpperCase().padEnd(7)}] [${context || 'Application'}] ${message} ${metaString}`;
  },
);

// Environment-specific settings
const isDevelopment = process.env.NODE_ENV === 'development';
const isProduction = process.env.NODE_ENV === 'production';

// Console transport configuration
const consoleTransport = new winston.transports.Console({
  format: winston.format.combine(
    winston.format.timestamp({ format: 'YYYY-MM-DD HH:mm:ss' }),
    winston.format.errors({ stack: true }),
    ...(isDevelopment ? [winston.format.colorize()] : []),
    customFormat,
  ),
});

// Daily rotate file transport for errors
const errorFileTransport = new winston.transports.DailyRotateFile({
  filename: 'logs/error-%DATE%.log',
  datePattern: 'YYYY-MM-DD',
  level: 'error',
  maxSize: '20m',
  maxFiles: '14d',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json(),
  ),
});

// Daily rotate file transport for all logs
const combinedFileTransport = new winston.transports.DailyRotateFile({
  filename: 'logs/app-%DATE%.log',
  datePattern: 'YYYY-MM-DD',
  maxSize: '20m',
  maxFiles: '30d',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json(),
  ),
});

// Build transports array based on environment
const transports: winston.transport[] = [consoleTransport];

if (isProduction) {
  transports.push(errorFileTransport, combinedFileTransport);
}

export const winstonConfig: winston.LoggerOptions = {
  level: process.env.LOG_LEVEL || (isDevelopment ? 'debug' : 'info'),
  transports,
  exceptionHandlers: [
    new winston.transports.File({ filename: 'logs/exceptions.log' }),
  ],
  rejectionHandlers: [
    new winston.transports.File({ filename: 'logs/rejections.log' }),
  ],
  exitOnError: false,
};

// main.ts
import { NestFactory } from '@nestjs/core';
import { WinstonModule } from 'nest-winston';
import { winstonConfig } from './config/winston.config';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, {
    logger: WinstonModule.createLogger(winstonConfig),
  });
  await app.listen(3000);
}
bootstrap();
```

---

### **Step 6: Production Configuration with Multiple Transports**

```typescript
// config/winston-production.config.ts
import * as winston from 'winston';
import 'winston-daily-rotate-file';

// Filter for specific log levels
const errorFilter = winston.format((info) => {
  return info.level === 'error' ? info : false;
});

const warnFilter = winston.format((info) => {
  return info.level === 'warn' ? info : false;
});

const infoFilter = winston.format((info) => {
  return info.level === 'info' ? info : false;
});

export const productionWinstonConfig: winston.LoggerOptions = {
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.metadata(),
  ),
  defaultMeta: {
    service: 'my-nestjs-app',
    environment: process.env.NODE_ENV,
    version: process.env.APP_VERSION || '1.0.0',
  },
  transports: [
    // Console - JSON format for container logs
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.json(),
      ),
    }),

    // Error logs - separate file with rotation
    new winston.transports.DailyRotateFile({
      filename: 'logs/error-%DATE%.log',
      datePattern: 'YYYY-MM-DD-HH',
      zippedArchive: true,
      maxSize: '20m',
      maxFiles: '14d',
      level: 'error',
      format: winston.format.combine(
        errorFilter(),
        winston.format.timestamp(),
        winston.format.json(),
      ),
    }),

    // Warning logs
    new winston.transports.DailyRotateFile({
      filename: 'logs/warn-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      maxSize: '20m',
      maxFiles: '7d',
      level: 'warn',
      format: winston.format.combine(
        warnFilter(),
        winston.format.timestamp(),
        winston.format.json(),
      ),
    }),

    // Info logs
    new winston.transports.DailyRotateFile({
      filename: 'logs/info-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      maxSize: '50m',
      maxFiles: '30d',
      level: 'info',
      format: winston.format.combine(
        infoFilter(),
        winston.format.timestamp(),
        winston.format.json(),
      ),
    }),

    // Combined logs (all levels)
    new winston.transports.DailyRotateFile({
      filename: 'logs/combined-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      maxSize: '100m',
      maxFiles: '30d',
      format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.json(),
      ),
    }),
  ],
  exceptionHandlers: [
    new winston.transports.File({
      filename: 'logs/exceptions.log',
      maxsize: 5242880, // 5MB
    }),
  ],
  rejectionHandlers: [
    new winston.transports.File({
      filename: 'logs/rejections.log',
      maxsize: 5242880, // 5MB
    }),
  ],
};
```

---

### **Step 7: Create Logger Service Wrapper**

```typescript
// logger/winston-logger.service.ts
import { Injectable, Inject, LoggerService } from '@nestjs/common';
import { WINSTON_MODULE_PROVIDER } from 'nest-winston';
import { Logger } from 'winston';

@Injectable()
export class WinstonLoggerService implements LoggerService {
  private context?: string;

  constructor(
    @Inject(WINSTON_MODULE_PROVIDER)
    private readonly logger: Logger,
  ) {}

  setContext(context: string) {
    this.context = context;
  }

  log(message: any, context?: string) {
    const ctx = context || this.context;
    this.logger.info(message, { context: ctx });
  }

  error(message: any, trace?: string, context?: string) {
    const ctx = context || this.context;
    this.logger.error(message, { context: ctx, trace });
  }

  warn(message: any, context?: string) {
    const ctx = context || this.context;
    this.logger.warn(message, { context: ctx });
  }

  debug(message: any, context?: string) {
    const ctx = context || this.context;
    this.logger.debug(message, { context: ctx });
  }

  verbose(message: any, context?: string) {
    const ctx = context || this.context;
    this.logger.verbose(message, { context: ctx });
  }

  // Additional Winston-specific methods
  logWithMeta(level: string, message: string, meta: any) {
    this.logger.log(level, message, {
      context: this.context,
      ...meta,
    });
  }

  // Profile a function
  async profile<T>(name: string, fn: () => Promise<T>): Promise<T> {
    this.logger.profile(name);
    try {
      return await fn();
    } finally {
      this.logger.profile(name);
    }
  }
}

// Usage in service
@Injectable()
export class ProductService {
  constructor(private readonly logger: WinstonLoggerService) {
    this.logger.setContext(ProductService.name);
  }

  async getProduct(id: string) {
    return await this.logger.profile('getProduct', async () => {
      this.logger.log(`Fetching product ${id}`);
      const product = await this.productRepository.findOne(id);
      return product;
    });
  }
}
```

---

### **Step 8: HTTP Request Logging with Interceptor**

```typescript
// interceptors/winston-logger.interceptor.ts
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
  Inject,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';
import { WINSTON_MODULE_PROVIDER } from 'nest-winston';
import { Logger } from 'winston';

@Injectable()
export class WinstonLoggerInterceptor implements NestInterceptor {
  constructor(
    @Inject(WINSTON_MODULE_PROVIDER)
    private readonly logger: Logger,
  ) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { method, url, body, headers } = request;
    const userAgent = headers['user-agent'] || '';
    const ip = headers['x-forwarded-for'] || request.ip;

    const now = Date.now();
    const requestId = this.generateRequestId();

    // Log incoming request
    this.logger.info('Incoming request', {
      context: 'HTTP',
      requestId,
      method,
      url,
      body: this.sanitizeBody(body),
      userAgent,
      ip,
    });

    return next.handle().pipe(
      tap({
        next: (data) => {
          const response = context.switchToHttp().getResponse();
          const { statusCode } = response;
          const duration = Date.now() - now;

          // Log successful response
          this.logger.info('Outgoing response', {
            context: 'HTTP',
            requestId,
            method,
            url,
            statusCode,
            duration: `${duration}ms`,
          });
        },
        error: (error) => {
          const response = context.switchToHttp().getResponse();
          const { statusCode } = response;
          const duration = Date.now() - now;

          // Log error response
          this.logger.error('Request failed', {
            context: 'HTTP',
            requestId,
            method,
            url,
            statusCode,
            duration: `${duration}ms`,
            error: error.message,
            stack: error.stack,
          });
        },
      }),
    );
  }

  private generateRequestId(): string {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }

  private sanitizeBody(body: any): any {
    if (!body) return null;
    const sanitized = { ...body };
    // Remove sensitive fields
    delete sanitized.password;
    delete sanitized.token;
    delete sanitized.creditCard;
    return sanitized;
  }
}

// app.controller.ts
import { Controller, Get, UseInterceptors } from '@nestjs/common';
import { WinstonLoggerInterceptor } from './interceptors/winston-logger.interceptor';

@Controller()
@UseInterceptors(WinstonLoggerInterceptor)
export class AppController {
  @Get()
  getHello(): string {
    return 'Hello World!';
  }
}
```

---

### **Testing Winston Integration:**

```typescript
// user.service.spec.ts
import { Test } from '@nestjs/testing';
import { WINSTON_MODULE_PROVIDER } from 'nest-winston';
import { UserService } from './user.service';
import { Logger } from 'winston';

describe('UserService', () => {
  let service: UserService;
  let logger: Logger;

  const mockLogger = {
    info: jest.fn(),
    error: jest.fn(),
    warn: jest.fn(),
    debug: jest.fn(),
  };

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UserService,
        {
          provide: WINSTON_MODULE_PROVIDER,
          useValue: mockLogger,
        },
      ],
    }).compile();

    service = module.get<UserService>(UserService);
    logger = module.get<Logger>(WINSTON_MODULE_PROVIDER);
  });

  it('should log user creation', async () => {
    const createUserDto = { email: 'test@example.com', name: 'Test User' };
    
    await service.createUser(createUserDto);

    expect(mockLogger.info).toHaveBeenCalledWith(
      'Creating new user',
      expect.objectContaining({
        context: 'UserService',
        email: 'test@example.com',
      }),
    );
  });

  it('should log errors', async () => {
    const error = new Error('Database error');
    jest.spyOn(service['userRepository'], 'save').mockRejectedValue(error);

    await expect(service.createUser({})).rejects.toThrow();

    expect(mockLogger.error).toHaveBeenCalledWith(
      'Failed to create user',
      expect.objectContaining({
        context: 'UserService',
        error: 'Database error',
      }),
    );
  });
});
```

---

### **Key Configuration Options:**

```typescript
const config: winston.LoggerOptions = {
  // Log level
  level: 'info', // 'error' | 'warn' | 'info' | 'debug' | 'verbose'

  // Default metadata for all logs
  defaultMeta: { service: 'my-app', version: '1.0.0' },

  // Log format
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json(),
  ),

  // Transports (where logs go)
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: 'app.log' }),
  ],

  // Exit on error?
  exitOnError: false,

  // Silent mode (disable all logging)
  silent: false,
};
```

**Key Takeaway:** Winston integration involves installing `nest-winston`, configuring transports and formats, injecting the logger using `WINSTON_MODULE_PROVIDER`, and using it in services with proper context.

</details>

<details>
<summary><strong>15. What are the advantages of Winston (transports, formatting)?</strong></summary>

**Answer:**

Winston is one of the most popular logging libraries for Node.js, offering extensive features like multiple transports, flexible formatting, exception handling, and a rich ecosystem of plugins. It's designed for versatility and production-grade logging needs.

---

### **Key Advantages:**

#### **1. Multiple Transports (Log to Different Destinations)**

Winston supports logging to multiple destinations simultaneously:

```typescript
import * as winston from 'winston';

const logger = winston.createLogger({
  transports: [
    // Console
    new winston.transports.Console(),
    
    // File
    new winston.transports.File({ filename: 'app.log' }),
    
    // HTTP endpoint
    new winston.transports.Http({
      host: 'logs.example.com',
      port: 8080,
      path: '/logs',
    }),
    
    // Stream
    new winston.transports.Stream({
      stream: fs.createWriteStream('stream.log'),
    }),
  ],
});

// Logs go to ALL transports simultaneously
logger.info('User logged in');
```

**Built-in Transports:**
- `Console` - Output to console
- `File` - Write to files
- `Http` - Send to HTTP endpoints
- `Stream` - Write to Node.js streams

**Third-Party Transports (via npm):**
```bash
# MongoDB
npm install winston-mongodb

# Redis
npm install winston-redis

# Elasticsearch
npm install winston-elasticsearch

# AWS CloudWatch
npm install winston-cloudwatch

# Syslog
npm install winston-syslog

# Loggly
npm install winston-loggly-bulk

# Slack
npm install winston-slack-webhook-transport
```

**Example with Multiple Transports:**
```typescript
import * as winston from 'winston';
import 'winston-daily-rotate-file';
import 'winston-mongodb';

const logger = winston.createLogger({
  transports: [
    // Console for development
    new winston.transports.Console({
      format: winston.format.simple(),
    }),
    
    // Rotating files for production
    new winston.transports.DailyRotateFile({
      filename: 'logs/app-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      maxSize: '20m',
      maxFiles: '14d',
    }),
    
    // MongoDB for searchable logs
    new winston.transports.MongoDB({
      db: process.env.MONGODB_URI,
      collection: 'logs',
      level: 'info',
    }),
    
    // CloudWatch for AWS infrastructure
    new winston.transports.CloudWatch({
      logGroupName: 'my-app',
      logStreamName: 'production',
      awsRegion: 'us-east-1',
    }),
  ],
});
```

---

#### **2. Flexible Formatting**

Winston provides powerful formatting options:

**Pre-built Formats:**
```typescript
import * as winston from 'winston';

// JSON format
winston.format.json();
// Output: {"level":"info","message":"User login"}

// Simple string format
winston.format.simple();
// Output: info: User login

// Colorized output
winston.format.colorize();

// Pretty print JSON
winston.format.prettyPrint();

// Add timestamp
winston.format.timestamp({ format: 'YYYY-MM-DD HH:mm:ss' });

// Log errors with stack traces
winston.format.errors({ stack: true });

// String interpolation
winston.format.splat();

// Metadata extraction
winston.format.metadata();

// Padding levels for alignment
winston.format.padLevels();

// Align output
winston.format.align();
```

**Combine Multiple Formats:**
```typescript
const logger = winston.createLogger({
  format: winston.format.combine(
    winston.format.timestamp({ format: 'YYYY-MM-DD HH:mm:ss' }),
    winston.format.errors({ stack: true }),
    winston.format.splat(),
    winston.format.json(),
  ),
});

logger.info('User %s logged in', 'john@example.com');
// Output: {"level":"info","message":"User john@example.com logged in","timestamp":"2025-12-30 10:30:45"}
```

**Custom Format with printf:**
```typescript
const customFormat = winston.format.printf(
  ({ level, message, timestamp, ...metadata }) => {
    let msg = `${timestamp} [${level.toUpperCase()}]: ${message}`;
    
    if (Object.keys(metadata).length > 0) {
      msg += ` ${JSON.stringify(metadata)}`;
    }
    
    return msg;
  },
);

const logger = winston.createLogger({
  format: winston.format.combine(
    winston.format.timestamp(),
    customFormat,
  ),
});

logger.info('Order placed', { orderId: 'ORD-123', total: 99.99 });
// Output: 2025-12-30T10:30:45.123Z [INFO]: Order placed {"orderId":"ORD-123","total":99.99}
```

**Format Filters:**
```typescript
// Only log errors
const errorFilter = winston.format((info) => {
  return info.level === 'error' ? info : false;
});

// Only log in production
const prodFilter = winston.format((info) => {
  return process.env.NODE_ENV === 'production' ? info : false;
});

const logger = winston.createLogger({
  format: winston.format.combine(
    errorFilter(),
    winston.format.json(),
  ),
  transports: [new winston.transports.File({ filename: 'errors.log' })],
});
```

---

#### **3. Log Levels with Priorities**

```typescript
// Default npm log levels
const levels = {
  error: 0,   // Highest priority
  warn: 1,
  info: 2,
  http: 3,
  verbose: 4,
  debug: 5,
  silly: 6,   // Lowest priority
};

const logger = winston.createLogger({
  levels,
  level: 'info', // Only log 'info' and higher (error, warn, info)
});

logger.error('Critical error');   // ✅ Logged
logger.warn('Warning message');   // ✅ Logged
logger.info('Info message');      // ✅ Logged
logger.http('HTTP request');      // ❌ Not logged (below threshold)
logger.debug('Debug info');       // ❌ Not logged

// Custom log levels
const customLevels = {
  levels: {
    critical: 0,
    error: 1,
    warning: 2,
    info: 3,
    debug: 4,
  },
  colors: {
    critical: 'red',
    error: 'red',
    warning: 'yellow',
    info: 'green',
    debug: 'blue',
  },
};

winston.addColors(customLevels.colors);

const customLogger = winston.createLogger({
  levels: customLevels.levels,
  level: 'info',
  format: winston.format.combine(
    winston.format.colorize(),
    winston.format.simple(),
  ),
  transports: [new winston.transports.Console()],
});
```

---

#### **4. Exception and Rejection Handling**

```typescript
const logger = winston.createLogger({
  transports: [
    new winston.transports.File({ filename: 'app.log' }),
  ],
  
  // Catch uncaught exceptions
  exceptionHandlers: [
    new winston.transports.File({ filename: 'exceptions.log' }),
  ],
  
  // Catch unhandled promise rejections
  rejectionHandlers: [
    new winston.transports.File({ filename: 'rejections.log' }),
  ],
  
  // Don't exit on error
  exitOnError: false,
});

// This will be logged to exceptions.log
throw new Error('Uncaught exception');

// This will be logged to rejections.log
Promise.reject(new Error('Unhandled rejection'));
```

---

#### **5. Query and Stream Logs**

```typescript
import * as winston from 'winston';

const logger = winston.createLogger({
  transports: [
    new winston.transports.File({ filename: 'app.log' }),
  ],
});

// Query logs
const options = {
  from: new Date() - 24 * 60 * 60 * 1000, // Last 24 hours
  until: new Date(),
  limit: 10,
  start: 0,
  order: 'desc',
  fields: ['message', 'level'],
};

logger.query(options, (err, results) => {
  if (err) {
    console.error('Query error:', err);
  } else {
    console.log('Query results:', results);
  }
});

// Stream logs in real-time
const stream = logger.stream({ start: -1 });

stream.on('log', (log) => {
  console.log('New log:', log);
});
```

---

#### **6. Profiling**

```typescript
const logger = winston.createLogger({
  transports: [new winston.transports.Console()],
});

// Start profiling
logger.profile('test-operation');

// ... perform operation ...
setTimeout(() => {
  // End profiling (automatically logs duration)
  logger.profile('test-operation');
  // Output: info: test-operation {"durationMs":1000}
}, 1000);

// Usage in async function
async function fetchData() {
  logger.profile('fetchData');
  
  const data = await fetch('https://api.example.com/data');
  
  logger.profile('fetchData');
  // Output: info: fetchData {"durationMs":234}
  
  return data;
}
```

---

#### **7. Child Loggers with Context**

```typescript
const logger = winston.createLogger({
  transports: [new winston.transports.Console()],
  format: winston.format.json(),
});

// Create child logger with default metadata
const userLogger = logger.child({ service: 'UserService' });
const orderLogger = logger.child({ service: 'OrderService' });

userLogger.info('User created');
// Output: {"level":"info","message":"User created","service":"UserService"}

orderLogger.info('Order placed');
// Output: {"level":"info","message":"Order placed","service":"OrderService"}

// Nested child loggers
const requestLogger = logger.child({ requestId: 'req-123' });
const userRequestLogger = requestLogger.child({ userId: 'user-456' });

userRequestLogger.info('Processing request');
// Output: {"level":"info","message":"Processing request","requestId":"req-123","userId":"user-456"}
```

---

#### **8. Different Formats for Different Transports**

```typescript
const logger = winston.createLogger({
  transports: [
    // Console: Pretty, colorized format
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.timestamp({ format: 'HH:mm:ss' }),
        winston.format.printf(
          ({ timestamp, level, message }) =>
            `${timestamp} ${level}: ${message}`,
        ),
      ),
    }),
    
    // File: JSON format for parsing
    new winston.transports.File({
      filename: 'app.log',
      format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.json(),
      ),
    }),
    
    // HTTP: Compact format
    new winston.transports.Http({
      host: 'logs.example.com',
      format: winston.format.json(),
    }),
  ],
});
```

---

#### **9. Metadata and Default Fields**

```typescript
const logger = winston.createLogger({
  defaultMeta: {
    service: 'my-nestjs-app',
    version: '1.0.0',
    environment: process.env.NODE_ENV,
    hostname: require('os').hostname(),
  },
  transports: [new winston.transports.Console()],
  format: winston.format.json(),
});

logger.info('Application started');
// Output: {"level":"info","message":"Application started","service":"my-nestjs-app","version":"1.0.0","environment":"production","hostname":"server-01"}

// Add extra metadata per log
logger.info('User login', { userId: 'user-123', ip: '192.168.1.1' });
// Output includes both defaultMeta and extra metadata
```

---

#### **10. Log Rotation**

```typescript
import * as winston from 'winston';
import 'winston-daily-rotate-file';

const rotateTransport = new winston.transports.DailyRotateFile({
  filename: 'logs/app-%DATE%.log',
  datePattern: 'YYYY-MM-DD',
  zippedArchive: true,  // Compress old files
  maxSize: '20m',       // Rotate when file reaches 20MB
  maxFiles: '14d',      // Keep logs for 14 days
});

rotateTransport.on('rotate', (oldFilename, newFilename) => {
  console.log('Log rotated:', { oldFilename, newFilename });
});

const logger = winston.createLogger({
  transports: [rotateTransport],
});
```

---

### **Production Example with All Features:**

```typescript
import * as winston from 'winston';
import 'winston-daily-rotate-file';

// Custom format
const productionFormat = winston.format.combine(
  winston.format.timestamp(),
  winston.format.errors({ stack: true }),
  winston.format.metadata(),
  winston.format.json(),
);

// Create logger
const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  
  defaultMeta: {
    service: 'ecommerce-api',
    version: process.env.APP_VERSION || '1.0.0',
    environment: process.env.NODE_ENV,
  },
  
  format: productionFormat,
  
  transports: [
    // Console (JSON for container logs)
    new winston.transports.Console({
      format: winston.format.json(),
    }),
    
    // Error logs with rotation
    new winston.transports.DailyRotateFile({
      filename: 'logs/error-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      level: 'error',
      maxSize: '20m',
      maxFiles: '30d',
      zippedArchive: true,
    }),
    
    // All logs with rotation
    new winston.transports.DailyRotateFile({
      filename: 'logs/combined-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      maxSize: '50m',
      maxFiles: '30d',
      zippedArchive: true,
    }),
    
    // HTTP logs only
    new winston.transports.DailyRotateFile({
      filename: 'logs/http-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      level: 'http',
      maxSize: '20m',
      maxFiles: '7d',
    }),
  ],
  
  exceptionHandlers: [
    new winston.transports.File({ filename: 'logs/exceptions.log' }),
  ],
  
  rejectionHandlers: [
    new winston.transports.File({ filename: 'logs/rejections.log' }),
  ],
  
  exitOnError: false,
});

// NestJS integration
import { NestFactory } from '@nestjs/core';
import { WinstonModule } from 'nest-winston';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, {
    logger: WinstonModule.createLogger({
      instance: logger,
    }),
  });
  
  await app.listen(3000);
}
```

---

### **Comparison with Other Loggers:**

| Feature | Winston | Pino | Bunyan | Console |
|---------|---------|------|--------|----------|
| Multiple transports | ✅ Built-in | ❌ External | ✅ Streams | ❌ |
| Custom formats | ✅ Extensive | ⚠️ Limited | ⚠️ Limited | ❌ |
| Performance | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ |
| Ecosystem | ✅ Very large | ⭐⭐⭐ | ⭐⭐ | ❌ |
| Log rotation | ✅ Plugin | ✅ Plugin | ✅ Built-in | ❌ |
| Exception handling | ✅ Built-in | ❌ | ❌ | ❌ |
| Query logs | ✅ Yes | ❌ | ❌ | ❌ |
| Profiling | ✅ Yes | ❌ | ❌ | ✅ console.time |
| Learning curve | Medium | Low | Medium | None |

---

### **When to Use Winston:**

✅ **Use Winston when you need:**
- Multiple log destinations (console + file + database + external service)
- Custom log formats for different environments
- Exception and rejection handling
- Log rotation and archival
- Rich ecosystem of transports
- Query and stream logs
- Profiling capabilities

❌ **Don't use Winston if:**
- You need maximum performance (use Pino)
- You only need simple console logging
- You want minimal dependencies

**Key Takeaway:** Winston's main advantages are **multiple transports** (log to many destinations), **flexible formatting** (customize output), **exception handling**, **log rotation**, and a **rich ecosystem** of plugins and transports.

</details>

<details>
<summary><strong>16. How do you integrate Pino for high-performance logging?</strong></summary>

**Answer:**

Pino is an ultra-fast logging library for Node.js, designed for high-performance applications. It's 5-10x faster than Winston due to its asynchronous architecture and minimal overhead. Integrating Pino with NestJS is straightforward using the `nestjs-pino` package.

---

### **Step 1: Installation**

```bash
# Core packages
npm install nestjs-pino pino-http

# Pretty printing for development
npm install --save-dev pino-pretty

# Optional: Transports for external services
npm install pino-elasticsearch pino-cloudwatch
```

---

### **Step 2: Basic Integration in AppModule**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { LoggerModule } from 'nestjs-pino';

@Module({
  imports: [
    LoggerModule.forRoot({
      pinoHttp: {
        // Log level
        level: process.env.LOG_LEVEL || 'info',
        
        // Pretty print in development
        transport: process.env.NODE_ENV !== 'production' ? {
          target: 'pino-pretty',
          options: {
            colorize: true,
            translateTime: 'SYS:standard',
            ignore: 'pid,hostname',
          },
        } : undefined,
      },
    }),
  ],
})
export class AppModule {}
```

**Output (Development with pino-pretty):**
```
[2025-12-30 10:30:45] INFO: User created successfully
[2025-12-30 10:30:46] ERROR: Payment failed
```

**Output (Production - JSON):**
```json
{"level":30,"time":1703937045123,"msg":"User created successfully"}
{"level":50,"time":1703937046456,"msg":"Payment failed"}
```

---

### **Step 3: Production Configuration**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { LoggerModule } from 'nestjs-pino';

@Module({
  imports: [
    LoggerModule.forRoot({
      pinoHttp: {
        level: process.env.NODE_ENV === 'production' ? 'info' : 'debug',
        
        // Production: Raw JSON output
        ...(process.env.NODE_ENV === 'production'
          ? {
              // Customize log format
              formatters: {
                level: (label) => ({ level: label }),
                bindings: (bindings) => ({
                  pid: bindings.pid,
                  hostname: bindings.hostname,
                  node_version: process.version,
                }),
              },
            }
          : {
              // Development: Pretty print
              transport: {
                target: 'pino-pretty',
                options: {
                  colorize: true,
                  translateTime: 'SYS:dd-mm-yyyy HH:MM:ss',
                  ignore: 'pid,hostname',
                  messageFormat: '{levelLabel} - {msg}',
                },
              },
            }),
        
        // Add custom properties to all logs
        customProps: (req, res) => ({
          requestId: req.id,
          userId: req.user?.id,
          userAgent: req.headers['user-agent'],
        }),
        
        // Redact sensitive information
        redact: {
          paths: [
            'req.headers.authorization',
            'req.headers.cookie',
            'req.body.password',
            'req.body.creditCard',
            'req.body.ssn',
            'res.headers["set-cookie"]',
          ],
          remove: true, // Remove redacted fields instead of showing [Redacted]
        },
        
        // Custom log level based on status code
        customLogLevel: (req, res, err) => {
          if (res.statusCode >= 400 && res.statusCode < 500) {
            return 'warn';
          } else if (res.statusCode >= 500 || err) {
            return 'error';
          } else if (res.statusCode >= 300 && res.statusCode < 400) {
            return 'silent'; // Don't log redirects
          }
          return 'info';
        },
        
        // Serialize request and response
        serializers: {
          req: (req) => ({
            method: req.method,
            url: req.url,
            query: req.query,
            params: req.params,
            headers: req.headers,
          }),
          res: (res) => ({
            statusCode: res.statusCode,
          }),
        },
        
        // Auto-log all HTTP requests
        autoLogging: true,
      },
    }),
  ],
})
export class AppModule {}
```

---

### **Step 4: Use in Services**

```typescript
import { Injectable } from '@nestjs/common';
import { PinoLogger } from 'nestjs-pino';

@Injectable()
export class UserService {
  constructor(private readonly logger: PinoLogger) {
    // Set context for this service
    this.logger.setContext(UserService.name);
  }

  async createUser(createUserDto: any) {
    // Log with metadata
    this.logger.info({ email: createUserDto.email }, 'Creating user');

    try {
      const user = await this.userRepository.save(createUserDto);

      this.logger.info(
        { userId: user.id, email: user.email },
        'User created successfully',
      );

      return user;
    } catch (error) {
      this.logger.error(
        { error: error.message, stack: error.stack },
        'Failed to create user',
      );
      throw error;
    }
  }

  async getUser(userId: string) {
    this.logger.debug({ userId }, 'Fetching user');
    return this.userRepository.findOne(userId);
  }
}
```

**Output:**
```json
{"level":30,"time":1703937045123,"context":"UserService","email":"john@example.com","msg":"Creating user"}
{"level":30,"time":1703937045456,"context":"UserService","userId":"user-123","email":"john@example.com","msg":"User created successfully"}
```

---

### **Step 5: Child Loggers for Request Context**

```typescript
import { Injectable } from '@nestjs/common';
import { PinoLogger } from 'nestjs-pino';

@Injectable()
export class OrderService {
  constructor(private readonly logger: PinoLogger) {
    this.logger.setContext(OrderService.name);
  }

  async processOrder(orderId: string, userId: string) {
    // Create child logger with order context
    const orderLogger = this.logger.logger.child({
      orderId,
      userId,
    });

    orderLogger.info('Starting order processing');

    try {
      orderLogger.info('Validating order');
      await this.validateOrder(orderId);

      orderLogger.info('Processing payment');
      await this.processPayment(orderId);

      orderLogger.info('Updating inventory');
      await this.updateInventory(orderId);

      orderLogger.info('Order processed successfully');
    } catch (error) {
      orderLogger.error(
        { error: error.message },
        'Order processing failed',
      );
      throw error;
    }
  }
}
```

**Output (all logs include orderId and userId):**
```json
{"level":30,"time":1703937045123,"orderId":"ORD-123","userId":"user-456","msg":"Starting order processing"}
{"level":30,"time":1703937045234,"orderId":"ORD-123","userId":"user-456","msg":"Validating order"}
{"level":30,"time":1703937045345,"orderId":"ORD-123","userId":"user-456","msg":"Processing payment"}
```

---

### **Step 6: Custom Pino Logger Configuration**

```typescript
// config/pino.config.ts
import { Params } from 'nestjs-pino';
import * as pino from 'pino';

export const pinoConfig: Params = {
  pinoHttp: {
    level: process.env.LOG_LEVEL || 'info',
    
    // Async logging for better performance
    stream: pino.destination({
      dest: process.stdout.fd,
      sync: false, // Asynchronous logging
      minLength: 4096, // Buffer size
    }),
    
    // Custom log levels
    customLevels: {
      critical: 60,
    },
    
    // Use custom levels
    useOnlyCustomLevels: false,
    
    // Timestamp format
    timestamp: () => `,"time":"${new Date().toISOString()}"`,
    
    // Message key
    messageKey: 'msg',
    
    // Error serialization
    serializers: {
      err: pino.stdSerializers.err,
      req: (req) => ({
        id: req.id,
        method: req.method,
        url: req.url,
      }),
      res: (res) => ({
        statusCode: res.statusCode,
      }),
    },
    
    // Custom request ID generator
    genReqId: (req) => req.headers['x-request-id'] || generateId(),
    
    // Redact sensitive data
    redact: {
      paths: [
        'req.headers.authorization',
        'req.body.password',
      ],
      censor: '[REDACTED]',
    },
  },
};

function generateId(): string {
  return `req-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
}

// app.module.ts
import { Module } from '@nestjs/common';
import { LoggerModule } from 'nestjs-pino';
import { pinoConfig } from './config/pino.config';

@Module({
  imports: [LoggerModule.forRoot(pinoConfig)],
})
export class AppModule {}
```

---

### **Step 7: Multiple Transports (Files, External Services)**

```typescript
// Install transport package
// npm install pino-multi-stream

import { Module } from '@nestjs/common';
import { LoggerModule } from 'nestjs-pino';
import * as pino from 'pino';
import * as fs from 'fs';

const streams = [
  // Console
  { stream: process.stdout },
  
  // File for all logs
  { stream: fs.createWriteStream('logs/app.log', { flags: 'a' }) },
  
  // File for errors only
  {
    level: 'error' as pino.Level,
    stream: fs.createWriteStream('logs/error.log', { flags: 'a' }),
  },
];

@Module({
  imports: [
    LoggerModule.forRoot({
      pinoHttp: {
        stream: pino.multistream(streams),
      },
    }),
  ],
})
export class AppModule {}
```

---

### **Step 8: Integration with External Services**

**Elasticsearch Transport:**
```typescript
// Install: npm install pino-elasticsearch
import { Module } from '@nestjs/common';
import { LoggerModule } from 'nestjs-pino';
import { createWriteStream } from 'pino-elasticsearch';

const streamToElastic = createWriteStream({
  node: process.env.ELASTICSEARCH_URL || 'http://localhost:9200',
  index: 'app-logs',
  consistency: 'one',
  flush-bytes: 1000,
});

@Module({
  imports: [
    LoggerModule.forRoot({
      pinoHttp: {
        stream: streamToElastic,
      },
    }),
  ],
})
export class AppModule {}
```

**CloudWatch Transport:**
```typescript
// Install: npm install pino-cloudwatch
import { Module } from '@nestjs/common';
import { LoggerModule } from 'nestjs-pino';
import { createWriteStream } from 'pino-cloudwatch';

const streamToCloudWatch = createWriteStream({
  logGroupName: '/aws/app/my-nestjs-app',
  logStreamName: `${process.env.NODE_ENV}-${new Date().toISOString().split('T')[0]}`,
  awsRegion: 'us-east-1',
  awsAccessKeyId: process.env.AWS_ACCESS_KEY,
  awsSecretAccessKey: process.env.AWS_SECRET_KEY,
});

@Module({
  imports: [
    LoggerModule.forRoot({
      pinoHttp: {
        stream: streamToCloudWatch,
      },
    }),
  ],
})
export class AppModule {}
```

---

### **Step 9: HTTP Request Logging Middleware**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { Logger } from 'nestjs-pino';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, { bufferLogs: true });
  
  // Use Pino logger
  app.useLogger(app.get(Logger));
  
  await app.listen(3000);
}
bootstrap();
```

**Automatic HTTP Logging:**
Pino automatically logs all HTTP requests with:
- Request ID
- Method and URL
- Status code
- Response time
- User agent
- Custom metadata

**Output:**
```json
{"level":30,"time":1703937045123,"req":{"id":"req-123","method":"GET","url":"/users/123"},"res":{"statusCode":200},"responseTime":45,"msg":"request completed"}
```

---

### **Step 10: Testing with Pino**

```typescript
// user.service.spec.ts
import { Test } from '@nestjs/testing';
import { PinoLogger } from 'nestjs-pino';
import { UserService } from './user.service';

describe('UserService', () => {
  let service: UserService;
  let logger: PinoLogger;

  const mockLogger = {
    info: jest.fn(),
    error: jest.fn(),
    warn: jest.fn(),
    debug: jest.fn(),
    setContext: jest.fn(),
    logger: {
      child: jest.fn().mockReturnThis(),
    },
  };

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UserService,
        {
          provide: PinoLogger,
          useValue: mockLogger,
        },
      ],
    }).compile();

    service = module.get<UserService>(UserService);
    logger = module.get<PinoLogger>(PinoLogger);
  });

  it('should log user creation', async () => {
    const createUserDto = { email: 'test@example.com' };
    
    await service.createUser(createUserDto);

    expect(mockLogger.info).toHaveBeenCalledWith(
      { email: 'test@example.com' },
      'Creating user',
    );
  });
});
```

---

### **Performance Comparison:**

```typescript
// Benchmark: 10,000 log writes

// Pino
const pino = require('pino')();
console.time('Pino');
for (let i = 0; i < 10000; i++) {
  pino.info({ iteration: i }, 'Test message');
}
console.timeEnd('Pino');
// Output: Pino: ~150ms

// Winston
const winston = require('winston');
const winstonLogger = winston.createLogger({
  transports: [new winston.transports.Console()],
});
console.time('Winston');
for (let i = 0; i < 10000; i++) {
  winstonLogger.info('Test message', { iteration: i });
}
console.timeEnd('Winston');
// Output: Winston: ~800ms

// Result: Pino is 5-6x faster
```

---

### **Advanced Configuration Example:**

```typescript
// logger/pino-production.config.ts
import { Params } from 'nestjs-pino';
import * as pino from 'pino';

export const pinoProductionConfig: Params = {
  pinoHttp: {
    level: 'info',
    
    // Performance: Async destination
    stream: pino.destination({
      dest: process.stdout.fd,
      sync: false,
      minLength: 4096,
    }),
    
    // Format
    formatters: {
      level: (label) => ({ level: label }),
      bindings: (bindings) => ({
        pid: bindings.pid,
        hostname: bindings.hostname,
      }),
    },
    
    // Base metadata
    base: {
      env: process.env.NODE_ENV,
      app: 'my-nestjs-app',
      version: process.env.APP_VERSION || '1.0.0',
    },
    
    // Request metadata
    customProps: (req) => ({
      requestId: req.id,
      userId: req.user?.id,
      ip: req.headers['x-forwarded-for'] || req.ip,
    }),
    
    // Redact sensitive data
    redact: {
      paths: [
        'req.headers.authorization',
        'req.headers.cookie',
        'req.body.password',
        'req.body.creditCard',
        'res.headers["set-cookie"]',
      ],
      remove: true,
    },
    
    // Serializers
    serializers: {
      req: (req) => ({
        id: req.id,
        method: req.method,
        url: req.url,
        query: req.query,
        params: req.params,
      }),
      res: (res) => ({
        statusCode: res.statusCode,
      }),
      err: pino.stdSerializers.err,
    },
    
    // Auto logging
    autoLogging: {
      ignore: (req) => req.url === '/health', // Don't log health checks
    },
    
    // Custom log level
    customLogLevel: (req, res, err) => {
      if (res.statusCode >= 500 || err) return 'error';
      if (res.statusCode >= 400) return 'warn';
      return 'info';
    },
  },
};
```

---

### **When to Use Pino:**

✅ **Use Pino for:**
- High-traffic applications
- Microservices
- Performance-critical systems
- Container-based deployments (K8s, Docker)
- Structured JSON logging
- Low-latency requirements

❌ **Don't use Pino if:**
- You need many built-in transports (use Winston)
- You prefer human-readable logs in production
- You need synchronous logging guarantees

**Key Takeaway:** Pino offers **ultra-fast performance** (5-10x faster than Winston), **asynchronous logging**, **automatic HTTP request tracking**, and **JSON-first output**, making it ideal for high-performance NestJS applications.

</details>

<details>
<summary><strong>17. What is the difference between Winston and Pino?</strong></summary>

**Answer:**

Winston and Pino are both popular logging libraries for Node.js, but they have different philosophies and strengths. Winston focuses on **versatility and features**, while Pino prioritizes **performance and simplicity**.

---

### **Quick Comparison:**

| Feature | Winston | Pino |
|---------|---------|------|
| **Performance** | ~10,000 logs/sec | ~50,000 logs/sec (5x faster) |
| **Philosophy** | Feature-rich, flexible | Fast, minimal overhead |
| **Output** | Multiple formats | JSON-first |
| **Transports** | Built-in (console, file, HTTP, stream) | External (via pino-transport) |
| **Formatting** | Extensive customization | Limited, JSON-focused |
| **Async logging** | Optional | Default (better performance) |
| **HTTP logging** | Manual setup | Built-in with nestjs-pino |
| **Learning curve** | Medium | Low |
| **Ecosystem** | Very large | Growing |
| **Memory usage** | Higher | Lower |
| **Best for** | Complex apps, multiple outputs | High-traffic, microservices |

---

### **1. Performance Comparison**

```typescript
// BENCHMARK: 10,000 log operations

// Pino - Asynchronous by default
import pino from 'pino';
const pinoLogger = pino();

console.time('Pino');
for (let i = 0; i < 10000; i++) {
  pinoLogger.info({ iteration: i, data: 'test' }, 'Log message');
}
console.timeEnd('Pino');
// Result: ~150ms (66,666 ops/sec)

// Winston - Synchronous by default
import * as winston from 'winston';
const winstonLogger = winston.createLogger({
  transports: [new winston.transports.Console()],
});

console.time('Winston');
for (let i = 0; i < 10000; i++) {
  winstonLogger.info('Log message', { iteration: i, data: 'test' });
}
console.timeEnd('Winston');
// Result: ~800ms (12,500 ops/sec)

// WINNER: Pino is 5-6x faster
```

**Why Pino is Faster:**
1. **Asynchronous by default** - Non-blocking I/O
2. **Minimal overhead** - Less processing per log
3. **Optimized JSON serialization** - Fast-json-stringify
4. **No string concatenation** - Direct object logging

---

### **2. Transports Comparison**

**Winston - Built-in Transports:**
```typescript
import * as winston from 'winston';

const logger = winston.createLogger({
  transports: [
    // ✅ Built-in: Console
    new winston.transports.Console(),
    
    // ✅ Built-in: File
    new winston.transports.File({ filename: 'app.log' }),
    
    // ✅ Built-in: HTTP
    new winston.transports.Http({
      host: 'logs.example.com',
      port: 8080,
    }),
    
    // ✅ Built-in: Stream
    new winston.transports.Stream({
      stream: fs.createWriteStream('stream.log'),
    }),
  ],
});

// Easy to add multiple destinations
logger.add(new winston.transports.File({ filename: 'error.log', level: 'error' }));
```

**Pino - External Transports:**
```typescript
import pino from 'pino';

// ⚠️ Transports are external (separate process)
const logger = pino(
  {
    // Main logger config
  },
  pino.transport({
    target: 'pino/file',
    options: { destination: 'app.log' },
  }),
);

// Or use pino-multi-stream for multiple outputs
import { multistream } from 'pino';

const streams = [
  { stream: process.stdout },
  { stream: fs.createWriteStream('app.log') },
];

const multiLogger = pino(multistream(streams));

// ❌ More complex setup for multiple destinations
```

**Winner:** Winston (easier multiple transports)

---

### **3. Format Flexibility**

**Winston - Extensive Formatting:**
```typescript
import * as winston from 'winston';

const logger = winston.createLogger({
  format: winston.format.combine(
    // ✅ Timestamp
    winston.format.timestamp({ format: 'YYYY-MM-DD HH:mm:ss' }),
    
    // ✅ Colors
    winston.format.colorize(),
    
    // ✅ Pretty print objects
    winston.format.prettyPrint(),
    
    // ✅ Custom format
    winston.format.printf(({ level, message, timestamp }) => {
      return `[${timestamp}] ${level}: ${message}`;
    }),
  ),
});

// Output: [2025-12-30 10:30:45] info: User created
```

**Pino - JSON-First:**
```typescript
import pino from 'pino';

// Production: Raw JSON (fast)
const logger = pino();
logger.info('User created');
// Output: {"level":30,"time":1703937045123,"msg":"User created"}

// Development: Pretty print (slower, requires transport)
const prettyLogger = pino({
  transport: {
    target: 'pino-pretty',
    options: { colorize: true },
  },
});

// ⚠️ Limited custom formatting (JSON-focused)
```

**Winner:** Winston (more formatting options)

---

### **4. Configuration Complexity**

**Winston Configuration:**
```typescript
import * as winston from 'winston';
import 'winston-daily-rotate-file';

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json(),
  ),
  defaultMeta: { service: 'my-app' },
  transports: [
    new winston.transports.Console(),
    new winston.transports.DailyRotateFile({
      filename: 'logs/app-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      maxSize: '20m',
      maxFiles: '14d',
    }),
  ],
  exceptionHandlers: [
    new winston.transports.File({ filename: 'exceptions.log' }),
  ],
});

// ⚠️ More configuration options = more complexity
```

**Pino Configuration:**
```typescript
import pino from 'pino';

const logger = pino({
  level: 'info',
  // Simple, minimal config
});

// ✅ Less configuration = simpler setup
```

**Winner:** Pino (simpler configuration)

---

### **5. Error Handling**

**Winston - Built-in Exception Handling:**
```typescript
const logger = winston.createLogger({
  transports: [new winston.transports.Console()],
  
  // ✅ Automatic exception handling
  exceptionHandlers: [
    new winston.transports.File({ filename: 'exceptions.log' }),
  ],
  
  // ✅ Automatic rejection handling
  rejectionHandlers: [
    new winston.transports.File({ filename: 'rejections.log' }),
  ],
});

// Uncaught exceptions automatically logged
throw new Error('Uncaught error');
```

**Pino - Manual Exception Handling:**
```typescript
const logger = pino();

// ⚠️ Manual setup required
process.on('uncaughtException', (error) => {
  logger.fatal({ err: error }, 'Uncaught exception');
  process.exit(1);
});

process.on('unhandledRejection', (reason) => {
  logger.fatal({ reason }, 'Unhandled rejection');
  process.exit(1);
});
```

**Winner:** Winston (built-in exception handling)

---

### **6. NestJS Integration**

**Winston with NestJS:**
```typescript
// Install: npm install winston nest-winston
import { NestFactory } from '@nestjs/core';
import { WinstonModule } from 'nest-winston';
import * as winston from 'winston';

const app = await NestFactory.create(AppModule, {
  logger: WinstonModule.createLogger({
    transports: [
      new winston.transports.Console(),
      new winston.transports.File({ filename: 'app.log' }),
    ],
  }),
});

// Inject in services
import { WINSTON_MODULE_PROVIDER } from 'nest-winston';
import { Logger } from 'winston';

@Injectable()
export class UserService {
  constructor(
    @Inject(WINSTON_MODULE_PROVIDER) private logger: Logger,
  ) {}
}
```

**Pino with NestJS:**
```typescript
// Install: npm install nestjs-pino pino-http
import { LoggerModule } from 'nestjs-pino';

@Module({
  imports: [
    LoggerModule.forRoot({
      pinoHttp: {
        level: 'info',
        transport: {
          target: 'pino-pretty',
        },
      },
    }),
  ],
})
export class AppModule {}

// Inject in services
import { PinoLogger } from 'nestjs-pino';

@Injectable()
export class UserService {
  constructor(private logger: PinoLogger) {
    this.logger.setContext(UserService.name);
  }
}
```

**Winner:** Pino (better NestJS integration with auto HTTP logging)

---

### **7. Memory and CPU Usage**

```typescript
// MEMORY BENCHMARK: 100,000 logs

// Pino
const pinoLogger = pino();
const pinoMemBefore = process.memoryUsage().heapUsed;
for (let i = 0; i < 100000; i++) {
  pinoLogger.info({ i }, 'Message');
}
const pinoMemAfter = process.memoryUsage().heapUsed;
console.log('Pino memory:', (pinoMemAfter - pinoMemBefore) / 1024 / 1024, 'MB');
// Result: ~15 MB

// Winston
const winstonLogger = winston.createLogger({
  transports: [new winston.transports.Console()],
});
const winstonMemBefore = process.memoryUsage().heapUsed;
for (let i = 0; i < 100000; i++) {
  winstonLogger.info('Message', { i });
}
const winstonMemAfter = process.memoryUsage().heapUsed;
console.log('Winston memory:', (winstonMemAfter - winstonMemBefore) / 1024 / 1024, 'MB');
// Result: ~45 MB

// WINNER: Pino (3x less memory usage)
```

---

### **8. Child Loggers**

**Winston Child Loggers:**
```typescript
const logger = winston.createLogger({ /* ... */ });

// Create child with default metadata
const childLogger = logger.child({ requestId: 'req-123' });

childLogger.info('Processing request');
// Output includes requestId in all logs
```

**Pino Child Loggers:**
```typescript
const logger = pino();

// Create child (fast, lightweight)
const childLogger = logger.child({ requestId: 'req-123' });

childLogger.info('Processing request');
// Output: {"level":30,"requestId":"req-123","msg":"Processing request"}

// Pino's child loggers are more performant
```

**Winner:** Pino (faster child logger creation)

---

### **9. Production Use Cases**

**Use Winston When:**
```typescript
// ✅ Multiple output destinations
const logger = winston.createLogger({
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: 'app.log' }),
    new winston.transports.MongoDB({ db: mongoUri }),
    new winston.transports.CloudWatch({ /* ... */ }),
  ],
});

// ✅ Complex formatting requirements
const customFormat = winston.format.printf(({ level, message, timestamp }) => {
  return `Custom format: [${timestamp}] ${level} - ${message}`;
});

// ✅ Human-readable logs in production
// ✅ Built-in exception handling
// ✅ Log querying and streaming
```

**Use Pino When:**
```typescript
// ✅ High-traffic applications
// - Handles 50,000+ requests/sec
// - Minimal CPU overhead
// - Low memory footprint

// ✅ Microservices architecture
// - Fast JSON logging
// - Structured output for log aggregators
// - Container-friendly (stdout JSON)

// ✅ Performance is critical
const logger = pino({
  level: 'info',
  // Fast, minimal config
});

// JSON output perfect for ELK, Datadog, CloudWatch
```

---

### **10. Code Comparison: Same Task**

**Task: Log HTTP requests with context**

**Winston Implementation:**
```typescript
import * as winston from 'winston';
import { Injectable, NestMiddleware } from '@nestjs/common';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  private logger = winston.createLogger({
    transports: [new winston.transports.Console()],
  });

  use(req: Request, res: Response, next: NextFunction) {
    const start = Date.now();
    
    res.on('finish', () => {
      const duration = Date.now() - start;
      this.logger.info('HTTP Request', {
        method: req.method,
        url: req.url,
        statusCode: res.statusCode,
        duration: `${duration}ms`,
      });
    });
    
    next();
  }
}

// ⚠️ Manual setup required
```

**Pino Implementation:**
```typescript
import { LoggerModule } from 'nestjs-pino';

@Module({
  imports: [
    LoggerModule.forRoot({
      pinoHttp: {
        autoLogging: true, // ✅ Automatic HTTP logging!
      },
    }),
  ],
})
export class AppModule {}

// ✅ Automatic, no middleware needed
// Output: {"level":30,"req":{"method":"GET","url":"/users"},"res":{"statusCode":200},"responseTime":45,"msg":"request completed"}
```

---

### **Decision Matrix:**

```typescript
// CHOOSE WINSTON IF:
const shouldUseWinston = (
  multipleTransports === true ||        // Log to console + file + database + external
  complexFormatting === true ||         // Need custom, human-readable formats
  builtInExceptionHandling === true ||  // Want automatic error catching
  logQuerying === true ||               // Need to query logs programmatically
  largeEcosystem === true               // Want many third-party transports
);

// CHOOSE PINO IF:
const shouldUsePino = (
  highTraffic === true ||               // >10,000 requests/sec
  performanceCritical === true ||       // Need minimal overhead
  microservices === true ||             // Container/K8s deployment
  structuredLogging === true ||         // JSON output for aggregators
  lowMemory === true                    // Memory-constrained environment
);
```

---

### **Side-by-Side Example:**

**Winston:**
```typescript
import * as winston from 'winston';

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json(),
  ),
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: 'app.log' }),
  ],
});

logger.info('User created', { userId: 'user-123' });
// Flexibility, features, multiple outputs
```

**Pino:**
```typescript
import pino from 'pino';

const logger = pino({ level: 'info' });

logger.info({ userId: 'user-123' }, 'User created');
// Speed, simplicity, JSON output
```

---

### **Summary Table:**

| Criterion | Winston | Pino | Winner |
|-----------|---------|------|--------|
| Performance | 10K ops/sec | 50K ops/sec | 🏆 Pino |
| Memory usage | Higher | Lower | 🏆 Pino |
| Built-in transports | Many | Few | 🏆 Winston |
| Formatting options | Extensive | Limited | 🏆 Winston |
| Configuration | Complex | Simple | 🏆 Pino |
| Exception handling | Built-in | Manual | 🏆 Winston |
| NestJS integration | Good | Excellent | 🏆 Pino |
| HTTP auto-logging | Manual | Automatic | 🏆 Pino |
| Learning curve | Medium | Low | 🏆 Pino |
| Ecosystem | Very large | Growing | 🏆 Winston |
| Production readiness | ✅ | ✅ | Tie |

**Key Takeaway:** Use **Winston** for complex applications needing multiple transports and custom formatting. Use **Pino** for high-performance applications prioritizing speed, low overhead, and structured JSON logging.

</details>

## Log Transports

<details>
<summary><strong>18. What are log transports?</strong></summary>

**Answer:**

Log transports are **destinations where log messages are sent**. They determine where and how logs are stored, displayed, or transmitted. A single logger can have multiple transports, allowing logs to be sent to different destinations simultaneously (e.g., console, files, databases, external services).

---

### **Concept:**

```typescript
// Transport = Destination for logs

Logger -----> [Transport 1: Console]    // Display in terminal
      |
      +-----> [Transport 2: File]       // Write to file
      |
      +-----> [Transport 3: Database]   // Store in MongoDB
      |
      +-----> [Transport 4: HTTP]       // Send to external service

// Same log message goes to ALL transports
```

---

### **Winston Transports:**

#### **1. Console Transport**
```typescript
import * as winston from 'winston';

const logger = winston.createLogger({
  transports: [
    new winston.transports.Console({
      level: 'debug',  // Log all levels
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.simple(),
      ),
    }),
  ],
});

logger.info('This appears in console');
// Output: info: This appears in console
```

---

#### **2. File Transport**
```typescript
const logger = winston.createLogger({
  transports: [
    new winston.transports.File({
      filename: 'logs/app.log',
      level: 'info',
      format: winston.format.json(),
    }),
  ],
});

logger.info('This is written to file');
// Writes to logs/app.log: {"level":"info","message":"This is written to file","timestamp":"..."}
```

---

#### **3. HTTP Transport**
```typescript
const logger = winston.createLogger({
  transports: [
    new winston.transports.Http({
      host: 'logs.example.com',
      port: 8080,
      path: '/logs',
      ssl: true,
      level: 'error',  // Only send errors
      format: winston.format.json(),
    }),
  ],
});

logger.error('This is sent to HTTP endpoint');
// POST https://logs.example.com:8080/logs
// Body: {"level":"error","message":"..."}
```

---

#### **4. Stream Transport**
```typescript
import * as fs from 'fs';

const logStream = fs.createWriteStream('logs/stream.log', { flags: 'a' });

const logger = winston.createLogger({
  transports: [
    new winston.transports.Stream({
      stream: logStream,
      format: winston.format.json(),
    }),
  ],
});

logger.info('This is written to stream');
```

---

### **Multiple Transports Example:**

```typescript
import * as winston from 'winston';

const logger = winston.createLogger({
  level: 'debug',
  
  transports: [
    // Transport 1: Console (all levels, colorized)
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.simple(),
      ),
    }),
    
    // Transport 2: File for all logs (JSON format)
    new winston.transports.File({
      filename: 'logs/combined.log',
      format: winston.format.json(),
    }),
    
    // Transport 3: Separate file for errors only
    new winston.transports.File({
      filename: 'logs/error.log',
      level: 'error',  // Only errors
      format: winston.format.json(),
    }),
  ],
});

// This log goes to Console + combined.log
logger.info('User logged in');

// This log goes to Console + combined.log + error.log
logger.error('Database connection failed');
```

---

### **Third-Party Transports:**

#### **MongoDB Transport**
```bash
npm install winston-mongodb
```

```typescript
import 'winston-mongodb';

const logger = winston.createLogger({
  transports: [
    new winston.transports.MongoDB({
      db: 'mongodb://localhost:27017/logs',
      collection: 'application_logs',
      level: 'info',
      options: {
        useUnifiedTopology: true,
      },
    }),
  ],
});

logger.info('Stored in MongoDB', { userId: 'user-123' });
// Stored as document in MongoDB collection
```

---

#### **Redis Transport**
```bash
npm install winston-redis
```

```typescript
import 'winston-redis';

const logger = winston.createLogger({
  transports: [
    new winston.transports.Redis({
      host: 'localhost',
      port: 6379,
      key: 'app-logs',  // Redis key
      length: 1000,     // Max logs to store
    }),
  ],
});

logger.info('Stored in Redis');
```

---

#### **Elasticsearch Transport**
```bash
npm install winston-elasticsearch
```

```typescript
import { ElasticsearchTransport } from 'winston-elasticsearch';

const esTransport = new ElasticsearchTransport({
  level: 'info',
  clientOpts: {
    node: 'http://localhost:9200',
  },
  index: 'app-logs',
});

const logger = winston.createLogger({
  transports: [esTransport],
});

logger.info('Indexed in Elasticsearch');
```

---

#### **CloudWatch Transport**
```bash
npm install winston-cloudwatch
```

```typescript
import * as CloudWatchTransport from 'winston-cloudwatch';

const logger = winston.createLogger({
  transports: [
    new CloudWatchTransport({
      logGroupName: '/aws/my-app',
      logStreamName: 'production',
      awsRegion: 'us-east-1',
      awsAccessKeyId: process.env.AWS_ACCESS_KEY,
      awsSecretKey: process.env.AWS_SECRET_KEY,
    }),
  ],
});

logger.info('Sent to AWS CloudWatch');
```

---

#### **Slack Transport**
```bash
npm install winston-slack-webhook-transport
```

```typescript
import SlackHook from 'winston-slack-webhook-transport';

const logger = winston.createLogger({
  transports: [
    new SlackHook({
      webhookUrl: process.env.SLACK_WEBHOOK_URL,
      level: 'error',  // Only send errors to Slack
      formatter: (info) => {
        return {
          text: `⚠️ Error: ${info.message}`,
          attachments: [
            {
              color: 'danger',
              fields: [
                { title: 'Level', value: info.level },
                { title: 'Timestamp', value: new Date().toISOString() },
              ],
            },
          ],
        };
      },
    }),
  ],
});

logger.error('Critical error!'); // Sends notification to Slack
```

---

### **Daily Rotating File Transport:**

```bash
npm install winston-daily-rotate-file
```

```typescript
import 'winston-daily-rotate-file';

const logger = winston.createLogger({
  transports: [
    new winston.transports.DailyRotateFile({
      filename: 'logs/app-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      zippedArchive: true,  // Compress old files
      maxSize: '20m',       // Rotate when file reaches 20MB
      maxFiles: '14d',      // Keep logs for 14 days
      level: 'info',
    }),
  ],
});

// Creates files like:
// logs/app-2025-12-30.log
// logs/app-2025-12-29.log.gz
// logs/app-2025-12-28.log.gz
```

---

### **Custom Transport:**

```typescript
import Transport from 'winston-transport';

// Create custom transport
class CustomTransport extends Transport {
  log(info, callback) {
    setImmediate(() => {
      this.emit('logged', info);
    });

    // Custom logic: Send to external API
    fetch('https://api.example.com/logs', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        level: info.level,
        message: info.message,
        timestamp: new Date().toISOString(),
      }),
    }).catch((error) => {
      console.error('Failed to send log:', error);
    });

    callback();
  }
}

// Use custom transport
const logger = winston.createLogger({
  transports: [
    new CustomTransport(),
  ],
});

logger.info('Sent via custom transport');
```

---

### **Transport-Specific Formatting:**

```typescript
const logger = winston.createLogger({
  transports: [
    // Console: Colorized, human-readable
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.simple(),
      ),
    }),
    
    // File: JSON for parsing
    new winston.transports.File({
      filename: 'logs/app.log',
      format: winston.format.json(),
    }),
    
    // HTTP: Compact JSON
    new winston.transports.Http({
      host: 'logs.example.com',
      format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.json(),
      ),
    }),
  ],
});

// Same log, different formats per transport
logger.info('User logged in', { userId: 'user-123' });

// Console: info: User logged in {"userId":"user-123"}
// File: {"level":"info","message":"User logged in","userId":"user-123"}
// HTTP: {"level":"info","message":"User logged in","userId":"user-123","timestamp":"..."}
```

---

### **Production Multi-Transport Setup:**

```typescript
import * as winston from 'winston';
import 'winston-daily-rotate-file';
import 'winston-mongodb';
import CloudWatchTransport from 'winston-cloudwatch';

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json(),
  ),
  defaultMeta: {
    service: 'my-app',
    environment: process.env.NODE_ENV,
  },
  transports: [
    // 1. Console (for container logs)
    new winston.transports.Console({
      format: winston.format.json(),
    }),
    
    // 2. Rotating files (for local storage)
    new winston.transports.DailyRotateFile({
      filename: 'logs/app-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      maxSize: '20m',
      maxFiles: '30d',
    }),
    
    // 3. Error file (separate error logs)
    new winston.transports.DailyRotateFile({
      filename: 'logs/error-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      level: 'error',
      maxSize: '20m',
      maxFiles: '30d',
    }),
    
    // 4. MongoDB (for querying and analysis)
    new winston.transports.MongoDB({
      db: process.env.MONGODB_URI,
      collection: 'logs',
      level: 'info',
    }),
    
    // 5. CloudWatch (for AWS monitoring)
    new CloudWatchTransport({
      logGroupName: '/aws/my-app',
      logStreamName: process.env.NODE_ENV,
      awsRegion: 'us-east-1',
    }),
  ],
  
  // Exception transport
  exceptionHandlers: [
    new winston.transports.File({ filename: 'logs/exceptions.log' }),
  ],
});

// This single log goes to 5 different destinations!
logger.info('Application started');
```

---

### **Conditional Transports:**

```typescript
const transports = [
  // Always log to console
  new winston.transports.Console(),
];

// Add file transport in production
if (process.env.NODE_ENV === 'production') {
  transports.push(
    new winston.transports.DailyRotateFile({
      filename: 'logs/app-%DATE%.log',
      maxSize: '20m',
      maxFiles: '30d',
    }),
  );
}

// Add CloudWatch in AWS environment
if (process.env.AWS_REGION) {
  transports.push(
    new CloudWatchTransport({
      logGroupName: '/aws/my-app',
      logStreamName: process.env.NODE_ENV,
      awsRegion: process.env.AWS_REGION,
    }),
  );
}

const logger = winston.createLogger({ transports });
```

---

### **Transport Events:**

```typescript
const fileTransport = new winston.transports.File({
  filename: 'logs/app.log',
});

// Listen to transport events
fileTransport.on('logged', (info) => {
  console.log('Log written to file:', info.message);
});

fileTransport.on('error', (error) => {
  console.error('Transport error:', error);
});

const logger = winston.createLogger({
  transports: [fileTransport],
});
```

---

### **Benefits of Multiple Transports:**

```typescript
// ✅ Redundancy - If one transport fails, others still work
const logger = winston.createLogger({
  transports: [
    new winston.transports.Console(),     // Local debugging
    new winston.transports.File({ filename: 'app.log' }),  // Backup
    new CloudWatchTransport({ /* ... */ }),  // Cloud monitoring
  ],
});

// ✅ Different levels per transport
const logger2 = winston.createLogger({
  transports: [
    new winston.transports.Console({ level: 'debug' }),  // All logs locally
    new winston.transports.File({ 
      filename: 'error.log', 
      level: 'error'  // Only errors in file
    }),
  ],
});

// ✅ Different formats per destination
const logger3 = winston.createLogger({
  transports: [
    new winston.transports.Console({
      format: winston.format.simple(),  // Human-readable
    }),
    new winston.transports.File({
      filename: 'app.log',
      format: winston.format.json(),    // Machine-parseable
    }),
  ],
});
```

**Key Takeaway:** Transports are **destinations for logs** (console, files, databases, external services). Multiple transports allow sending logs to different destinations simultaneously with different levels and formats.

</details>

<details>
<summary><strong>19. How do you log to files using Winston?</strong></summary>

**Answer:**

Winston provides multiple ways to log to files: basic file transport, rotating files, separate files by log level, and advanced configurations for production environments.

---

### **Method 1: Basic File Logging**

```typescript
import * as winston from 'winston';

const logger = winston.createLogger({
  transports: [
    new winston.transports.File({
      filename: 'logs/app.log',
      level: 'info',
      format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.json(),
      ),
    }),
  ],
});

logger.info('This is logged to file');
logger.error('This is an error');

// Creates logs/app.log:
// {"level":"info","message":"This is logged to file","timestamp":"2025-12-30T10:30:45.123Z"}
// {"level":"error","message":"This is an error","timestamp":"2025-12-30T10:30:46.456Z"}
```

---

### **Method 2: Multiple Files by Log Level**

```typescript
import * as winston from 'winston';

const logger = winston.createLogger({
  transports: [
    // All logs
    new winston.transports.File({
      filename: 'logs/combined.log',
      format: winston.format.json(),
    }),
    
    // Only errors
    new winston.transports.File({
      filename: 'logs/error.log',
      level: 'error',
      format: winston.format.json(),
    }),
    
    // Only warnings
    new winston.transports.File({
      filename: 'logs/warn.log',
      level: 'warn',
      format: winston.format.json(),
    }),
  ],
});

logger.info('Info message');     // → combined.log only
logger.warn('Warning message');  // → combined.log + warn.log
logger.error('Error message');   // → combined.log + error.log
```

---

### **Method 3: Daily Rotating Files**

```bash
npm install winston-daily-rotate-file
```

```typescript
import * as winston from 'winston';
import 'winston-daily-rotate-file';

const transport = new winston.transports.DailyRotateFile({
  filename: 'logs/app-%DATE%.log',
  datePattern: 'YYYY-MM-DD',
  zippedArchive: true,  // Compress old logs
  maxSize: '20m',       // Rotate at 20MB
  maxFiles: '14d',      // Keep for 14 days
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json(),
  ),
});

// Listen to rotation events
transport.on('rotate', (oldFilename, newFilename) => {
  console.log('Log file rotated:', { oldFilename, newFilename });
});

const logger = winston.createLogger({
  transports: [transport],
});

logger.info('This creates daily files');

// Creates files:
// logs/app-2025-12-30.log
// logs/app-2025-12-29.log.gz (compressed)
// logs/app-2025-12-28.log.gz
```

---

### **Method 4: Size-Based Rotation**

```typescript
import * as winston from 'winston';
import 'winston-daily-rotate-file';

const logger = winston.createLogger({
  transports: [
    new winston.transports.DailyRotateFile({
      filename: 'logs/app-%DATE%.log',
      datePattern: 'YYYY-MM-DD-HH',  // Hourly rotation
      maxSize: '10m',                 // Rotate at 10MB
      maxFiles: '7d',                 // Keep for 7 days
    }),
  ],
});

// Creates files:
// logs/app-2025-12-30-10.log (10:00 AM)
// logs/app-2025-12-30-11.log (11:00 AM)
// If file reaches 10MB, creates:
// logs/app-2025-12-30-11-1.log
// logs/app-2025-12-30-11-2.log
```

---

### **Method 5: Separate Files for Each Log Level**

```typescript
import * as winston from 'winston';
import 'winston-daily-rotate-file';

// Create filter for specific level
const levelFilter = (level: string) => {
  return winston.format((info) => {
    return info.level === level ? info : false;
  })();
};

const logger = winston.createLogger({
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json(),
  ),
  transports: [
    // Error logs
    new winston.transports.DailyRotateFile({
      filename: 'logs/error-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      level: 'error',
      maxSize: '20m',
      maxFiles: '30d',
      format: levelFilter('error'),
    }),
    
    // Warn logs
    new winston.transports.DailyRotateFile({
      filename: 'logs/warn-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      level: 'warn',
      maxSize: '20m',
      maxFiles: '14d',
      format: levelFilter('warn'),
    }),
    
    // Info logs
    new winston.transports.DailyRotateFile({
      filename: 'logs/info-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      level: 'info',
      maxSize: '50m',
      maxFiles: '7d',
      format: levelFilter('info'),
    }),
    
    // Debug logs (development only)
    ...(process.env.NODE_ENV !== 'production' ? [
      new winston.transports.DailyRotateFile({
        filename: 'logs/debug-%DATE%.log',
        datePattern: 'YYYY-MM-DD',
        level: 'debug',
        maxSize: '100m',
        maxFiles: '3d',
      }),
    ] : []),
  ],
});
```

---

### **Method 6: Production Configuration**

```typescript
import * as winston from 'winston';
import 'winston-daily-rotate-file';
import * as path from 'path';
import * as fs from 'fs';

// Ensure logs directory exists
const logsDir = path.join(process.cwd(), 'logs');
if (!fs.existsSync(logsDir)) {
  fs.mkdirSync(logsDir, { recursive: true });
}

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.metadata(),
    winston.format.json(),
  ),
  
  defaultMeta: {
    service: 'my-app',
    environment: process.env.NODE_ENV,
    hostname: require('os').hostname(),
  },
  
  transports: [
    // Combined logs (all levels)
    new winston.transports.DailyRotateFile({
      filename: path.join(logsDir, 'combined-%DATE%.log'),
      datePattern: 'YYYY-MM-DD',
      maxSize: '50m',
      maxFiles: '30d',
      zippedArchive: true,
    }),
    
    // Error logs (errors only)
    new winston.transports.DailyRotateFile({
      filename: path.join(logsDir, 'error-%DATE%.log'),
      datePattern: 'YYYY-MM-DD',
      level: 'error',
      maxSize: '20m',
      maxFiles: '30d',
      zippedArchive: true,
    }),
    
    // HTTP logs (for API requests)
    new winston.transports.DailyRotateFile({
      filename: path.join(logsDir, 'http-%DATE%.log'),
      datePattern: 'YYYY-MM-DD',
      level: 'http',
      maxSize: '20m',
      maxFiles: '14d',
    }),
  ],
  
  // Exception logging
  exceptionHandlers: [
    new winston.transports.File({
      filename: path.join(logsDir, 'exceptions.log'),
      maxsize: 5242880, // 5MB
    }),
  ],
  
  // Rejection logging
  rejectionHandlers: [
    new winston.transports.File({
      filename: path.join(logsDir, 'rejections.log'),
      maxsize: 5242880, // 5MB
    }),
  ],
});

export default logger;
```

---

### **Method 7: File Configuration Options**

```typescript
import * as winston from 'winston';

const logger = winston.createLogger({
  transports: [
    new winston.transports.File({
      // Required
      filename: 'logs/app.log',
      
      // Optional configurations
      level: 'info',              // Minimum level
      maxsize: 5242880,           // 5MB max file size
      maxFiles: 5,                // Keep max 5 files
      tailable: true,             // Create numbered files (app1.log, app2.log)
      zippedArchive: false,       // Don't compress
      
      // File writing options
      options: {
        flags: 'a',               // Append mode
        mode: 0o666,              // File permissions
      },
      
      // Formatting
      format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.json(),
      ),
      
      // Stream options
      stream: null,               // Custom stream
      
      // Delay opening the file
      lazy: false,                // Open immediately
    }),
  ],
});
```

---

### **Method 8: Custom File Structure**

```typescript
import * as winston from 'winston';
import * as path from 'path';

// Organize logs by service/module
const createModuleLogger = (moduleName: string) => {
  const logsDir = path.join('logs', moduleName);
  
  return winston.createLogger({
    transports: [
      new winston.transports.DailyRotateFile({
        filename: path.join(logsDir, 'app-%DATE%.log'),
        datePattern: 'YYYY-MM-DD',
        maxSize: '20m',
        maxFiles: '14d',
      }),
      new winston.transports.DailyRotateFile({
        filename: path.join(logsDir, 'error-%DATE%.log'),
        datePattern: 'YYYY-MM-DD',
        level: 'error',
        maxSize: '10m',
        maxFiles: '30d',
      }),
    ],
  });
};

// Usage
const userLogger = createModuleLogger('users');
const orderLogger = createModuleLogger('orders');
const paymentLogger = createModuleLogger('payments');

userLogger.info('User created');
// → logs/users/app-2025-12-30.log

orderLogger.error('Order failed');
// → logs/orders/error-2025-12-30.log

// Directory structure:
// logs/
//   users/
//     app-2025-12-30.log
//     error-2025-12-30.log
//   orders/
//     app-2025-12-30.log
//     error-2025-12-30.log
//   payments/
//     app-2025-12-30.log
```

---

### **Method 9: NestJS Integration**

```typescript
// logger.config.ts
import * as winston from 'winston';
import 'winston-daily-rotate-file';
import * as path from 'path';

const logsDir = path.join(process.cwd(), 'logs');

export const winstonConfig = {
  transports: [
    // Console for development
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.simple(),
      ),
    }),
    
    // File for production
    new winston.transports.DailyRotateFile({
      filename: path.join(logsDir, 'app-%DATE%.log'),
      datePattern: 'YYYY-MM-DD',
      maxSize: '20m',
      maxFiles: '14d',
      format: winston.format.json(),
    }),
  ],
};

// main.ts
import { NestFactory } from '@nestjs/core';
import { WinstonModule } from 'nest-winston';
import { winstonConfig } from './logger.config';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, {
    logger: WinstonModule.createLogger(winstonConfig),
  });
  
  await app.listen(3000);
}
bootstrap();

// Usage in service
import { Injectable, Inject } from '@nestjs/common';
import { WINSTON_MODULE_PROVIDER } from 'nest-winston';
import { Logger } from 'winston';

@Injectable()
export class UserService {
  constructor(
    @Inject(WINSTON_MODULE_PROVIDER) private logger: Logger,
  ) {}

  createUser(userData: any) {
    this.logger.info('Creating user', {
      context: 'UserService',
      email: userData.email,
    });
    // Logged to logs/app-2025-12-30.log
  }
}
```

---

### **Method 10: JSON File with Metadata**

```typescript
import * as winston from 'winston';

const logger = winston.createLogger({
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.metadata(),
    winston.format.json(),
  ),
  defaultMeta: {
    service: 'ecommerce-api',
    version: '1.0.0',
    environment: process.env.NODE_ENV,
    pid: process.pid,
  },
  transports: [
    new winston.transports.File({
      filename: 'logs/app.log',
    }),
  ],
});

logger.info('Order placed', {
  orderId: 'ORD-123',
  userId: 'user-456',
  total: 99.99,
});

// logs/app.log:
// {
//   "level": "info",
//   "message": "Order placed",
//   "timestamp": "2025-12-30T10:30:45.123Z",
//   "service": "ecommerce-api",
//   "version": "1.0.0",
//   "environment": "production",
//   "pid": 12345,
//   "metadata": {
//     "orderId": "ORD-123",
//     "userId": "user-456",
//     "total": 99.99
//   }
// }
```

---

### **Reading Log Files:**

```typescript
import * as fs from 'fs';
import * as readline from 'readline';

// Read JSON log file line by line
async function readLogFile(filename: string) {
  const fileStream = fs.createReadStream(filename);
  const rl = readline.createInterface({
    input: fileStream,
    crlfDelay: Infinity,
  });

  for await (const line of rl) {
    try {
      const log = JSON.parse(line);
      console.log(`[${log.level}] ${log.message}`, log.metadata);
    } catch (error) {
      console.error('Failed to parse log line:', line);
    }
  }
}

// Find errors in log file
async function findErrors(filename: string) {
  const fileStream = fs.createReadStream(filename);
  const rl = readline.createInterface({
    input: fileStream,
    crlfDelay: Infinity,
  });

  const errors = [];
  for await (const line of rl) {
    const log = JSON.parse(line);
    if (log.level === 'error') {
      errors.push(log);
    }
  }
  
  return errors;
}

// Usage
readLogFile('logs/app-2025-12-30.log');
const errors = await findErrors('logs/error-2025-12-30.log');
console.log(`Found ${errors.length} errors`);
```

---

### **Best Practices:**

```typescript
// ✅ GOOD - Separate files by level
const logger = winston.createLogger({
  transports: [
    new winston.transports.File({ filename: 'combined.log' }),
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
  ],
});

// ✅ GOOD - Use daily rotation
new winston.transports.DailyRotateFile({
  filename: 'app-%DATE%.log',
  maxFiles: '14d',  // Auto-delete old logs
});

// ✅ GOOD - JSON format for parsing
format: winston.format.json()

// ✅ GOOD - Include timestamp
format: winston.format.timestamp()

// ✅ GOOD - Create logs directory
if (!fs.existsSync('logs')) {
  fs.mkdirSync('logs', { recursive: true });
}

// ❌ BAD - No rotation (disk fills up)
new winston.transports.File({ filename: 'app.log' }) // Grows forever!

// ❌ BAD - Logging to root directory
filename: 'app.log'  // Should be in logs/ subdirectory

// ❌ BAD - No size limit
// File can grow indefinitely

// ❌ BAD - Mixed levels in same file without separation
// Hard to find errors among info logs
```

**Key Takeaway:** Use **daily rotating file transports** with **maxSize** and **maxFiles** limits, separate files by log level (error, combined), JSON format for parsing, and ensure logs directory exists before writing.

</details>

<details>
<summary><strong>20. How do you log to external services (Loggly, Papertrail)?</strong></summary>

**Answer:**

Logging to external services allows centralized log management, analysis, alerting, and long-term storage. Popular services include Loggly, Papertrail, Datadog, New Relic, and Splunk. Integration typically involves installing transport packages and configuring API keys.

---

### **1. Loggly Integration**

**Installation:**
```bash
npm install winston-loggly-bulk
```

**Basic Setup:**
```typescript
import * as winston from 'winston';
import { Loggly } from 'winston-loggly-bulk';

const logger = winston.createLogger({
  transports: [
    new Loggly({
      token: process.env.LOGGLY_TOKEN,
      subdomain: 'your-subdomain',
      tags: ['nestjs', 'production'],
      json: true,
      level: 'info',
    }),
  ],
});

logger.info('User logged in', { userId: 'user-123', ip: '192.168.1.1' });
// Sent to Loggly with tags and metadata
```

**Production Configuration:**
```typescript
import * as winston from 'winston';
import { Loggly } from 'winston-loggly-bulk';

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json(),
  ),
  defaultMeta: {
    service: 'my-app',
    environment: process.env.NODE_ENV,
    hostname: require('os').hostname(),
  },
  transports: [
    // Console for local debugging
    new winston.transports.Console(),
    
    // Loggly for centralized logging
    new Loggly({
      token: process.env.LOGGLY_TOKEN,
      subdomain: process.env.LOGGLY_SUBDOMAIN,
      tags: [
        'nodejs',
        'nestjs',
        process.env.NODE_ENV,
        require('os').hostname(),
      ],
      json: true,
      level: 'info',
      
      // Buffer logs for better performance
      bufferOptions: {
        size: 100,
        retriesInMilliSeconds: 30000,
      },
      
      // Network options
      networkErrorsOnConsole: true,
      isBulk: true,
    }),
  ],
});

export default logger;
```

**NestJS Integration:**
```typescript
// logger.module.ts
import { Module } from '@nestjs/common';
import { WinstonModule } from 'nest-winston';
import * as winston from 'winston';
import { Loggly } from 'winston-loggly-bulk';

@Module({
  imports: [
    WinstonModule.forRoot({
      transports: [
        new winston.transports.Console(),
        new Loggly({
          token: process.env.LOGGLY_TOKEN,
          subdomain: process.env.LOGGLY_SUBDOMAIN,
          tags: ['nestjs', process.env.NODE_ENV],
          json: true,
        }),
      ],
    }),
  ],
})
export class LoggerModule {}
```

---

### **2. Papertrail Integration**

**Installation:**
```bash
npm install winston-papertrail
```

**Basic Setup:**
```typescript
import * as winston from 'winston';
import * as WinstonPapertrail from 'winston-papertrail';

const papertrailTransport = new WinstonPapertrail({
  host: 'logs.papertrailapp.com',
  port: 12345,  // Your Papertrail port
  hostname: require('os').hostname(),
  program: 'my-nestjs-app',
  level: 'info',
  colorize: true,
});

const logger = winston.createLogger({
  transports: [papertrailTransport],
});

logger.info('Application started');
// Sent to Papertrail
```

**Production Configuration:**
```typescript
import * as winston from 'winston';
import * as WinstonPapertrail from 'winston-papertrail';

const papertrailTransport = new WinstonPapertrail({
  host: process.env.PAPERTRAIL_HOST || 'logs.papertrailapp.com',
  port: parseInt(process.env.PAPERTRAIL_PORT || '12345'),
  hostname: process.env.HOSTNAME || require('os').hostname(),
  program: process.env.APP_NAME || 'my-app',
  facility: 'local0',
  logFormat: (level, message) => `[${level}] ${message}`,
  level: 'info',
  colorize: false,  // Disable colors for production
  
  // Handle connection errors
  handleExceptions: true,
  inlineMeta: true,
});

// Wait for connection before logging
papertrailTransport.on('connect', () => {
  console.log('Connected to Papertrail');
});

papertrailTransport.on('error', (err) => {
  console.error('Papertrail error:', err);
});

const logger = winston.createLogger({
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json(),
  ),
  transports: [
    new winston.transports.Console(),
    papertrailTransport,
  ],
});

export default logger;
```

---

### **3. Datadog Integration**

**Installation:**
```bash
npm install winston-datadog-logs
```

**Configuration:**
```typescript
import * as winston from 'winston';
import DatadogWinston from 'winston-datadog-logs';

const logger = winston.createLogger({
  transports: [
    new winston.transports.Console(),
    new DatadogWinston({
      apiKey: process.env.DATADOG_API_KEY,
      hostname: require('os').hostname(),
      service: 'my-nestjs-app',
      ddsource: 'nodejs',
      ddtags: `env:${process.env.NODE_ENV},version:1.0.0`,
      level: 'info',
    }),
  ],
});

logger.info('User action', {
  userId: 'user-123',
  action: 'login',
  timestamp: new Date().toISOString(),
});
// Sent to Datadog Logs
```

---

### **4. Splunk Integration**

**Installation:**
```bash
npm install winston-splunk-httplogger
```

**Configuration:**
```typescript
import * as winston from 'winston';
import * as SplunkLogger from 'winston-splunk-httplogger';

const splunkTransport = new SplunkLogger({
  token: process.env.SPLUNK_TOKEN,
  host: 'splunk.example.com',
  port: 8088,
  protocol: 'https',
  path: '/services/collector/event',
  maxBatchCount: 100,
  maxBatchSize: 10000,
  level: 'info',
});

const logger = winston.createLogger({
  transports: [splunkTransport],
});

logger.info('Event logged to Splunk', {
  eventType: 'user_action',
  userId: 'user-123',
});
```

---

### **5. New Relic Integration**

**Installation:**
```bash
npm install newrelic
```

**Configuration:**
```typescript
// newrelic.js (in project root)
exports.config = {
  app_name: ['My NestJS App'],
  license_key: process.env.NEW_RELIC_LICENSE_KEY,
  logging: {
    level: 'info',
    enabled: true,
  },
};

// main.ts
import 'newrelic';  // Must be first import!
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();

// Custom logging to New Relic
import * as newrelic from 'newrelic';

export class UserService {
  createUser(userData: any) {
    // Log custom event
    newrelic.recordCustomEvent('UserCreated', {
      userId: userData.id,
      email: userData.email,
      timestamp: Date.now(),
    });
    
    // Add custom attributes to transaction
    newrelic.addCustomAttributes({
      userId: userData.id,
      action: 'create_user',
    });
  }
}
```

---

### **6. AWS CloudWatch Logs**

**Installation:**
```bash
npm install winston-cloudwatch
```

**Configuration:**
```typescript
import * as winston from 'winston';
import * as CloudWatchTransport from 'winston-cloudwatch';

const cloudWatchTransport = new CloudWatchTransport({
  logGroupName: '/aws/my-app',
  logStreamName: `${process.env.NODE_ENV}-${new Date().toISOString().split('T')[0]}`,
  awsRegion: process.env.AWS_REGION || 'us-east-1',
  awsAccessKeyId: process.env.AWS_ACCESS_KEY_ID,
  awsSecretKey: process.env.AWS_SECRET_ACCESS_KEY,
  messageFormatter: (log) => {
    return JSON.stringify({
      level: log.level,
      message: log.message,
      timestamp: new Date().toISOString(),
      metadata: log.meta,
    });
  },
  uploadRate: 2000,  // Send logs every 2 seconds
  errorHandler: (err) => {
    console.error('CloudWatch error:', err);
  },
});

const logger = winston.createLogger({
  transports: [cloudWatchTransport],
});

logger.info('Application started');
// Sent to CloudWatch Logs
```

---

### **7. Google Cloud Logging**

**Installation:**
```bash
npm install @google-cloud/logging-winston
```

**Configuration:**
```typescript
import * as winston from 'winston';
import { LoggingWinston } from '@google-cloud/logging-winston';

const loggingWinston = new LoggingWinston({
  projectId: process.env.GCP_PROJECT_ID,
  keyFilename: process.env.GCP_KEY_FILE,
  logName: 'my-app-log',
  resource: {
    type: 'global',
  },
});

const logger = winston.createLogger({
  level: 'info',
  transports: [
    new winston.transports.Console(),
    loggingWinston,
  ],
});

logger.info('Logged to Google Cloud', {
  userId: 'user-123',
  action: 'login',
});
```

---

### **8. Generic HTTP/HTTPS Endpoint**

```typescript
import * as winston from 'winston';
import * as https from 'https';

// Custom transport for any HTTP endpoint
class HttpTransport extends winston.transports.Http {
  constructor(options: any) {
    super(options);
  }
}

const logger = winston.createLogger({
  transports: [
    new winston.transports.Http({
      host: 'logs.example.com',
      port: 443,
      path: '/api/logs',
      ssl: true,
      auth: {
        username: process.env.LOG_USERNAME,
        password: process.env.LOG_PASSWORD,
      },
      headers: {
        'Content-Type': 'application/json',
        'X-API-Key': process.env.LOG_API_KEY,
      },
      format: winston.format.json(),
    }),
  ],
});

logger.info('Sent to custom HTTP endpoint');
```

---

### **9. Multiple External Services**

```typescript
import * as winston from 'winston';
import { Loggly } from 'winston-loggly-bulk';
import DatadogWinston from 'winston-datadog-logs';
import * as CloudWatchTransport from 'winston-cloudwatch';

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json(),
  ),
  defaultMeta: {
    service: 'my-app',
    environment: process.env.NODE_ENV,
  },
  transports: [
    // Console (local debugging)
    new winston.transports.Console(),
    
    // Loggly (general logging)
    new Loggly({
      token: process.env.LOGGLY_TOKEN,
      subdomain: process.env.LOGGLY_SUBDOMAIN,
      tags: ['nestjs', 'production'],
      json: true,
    }),
    
    // Datadog (monitoring and APM)
    new DatadogWinston({
      apiKey: process.env.DATADOG_API_KEY,
      service: 'my-nestjs-app',
    }),
    
    // CloudWatch (AWS infrastructure)
    new CloudWatchTransport({
      logGroupName: '/aws/my-app',
      logStreamName: 'production',
      awsRegion: 'us-east-1',
    }),
  ],
});

// Single log sent to 4 destinations!
logger.info('Application started');
```

---

### **10. NestJS Production Setup**

```typescript
// config/logger.config.ts
import * as winston from 'winston';
import { Loggly } from 'winston-loggly-bulk';
import DatadogWinston from 'winston-datadog-logs';
import 'winston-daily-rotate-file';

const transports: winston.transport[] = [
  // Always log to console
  new winston.transports.Console({
    format: winston.format.combine(
      winston.format.colorize(),
      winston.format.simple(),
    ),
  }),
];

// Add file transport in non-production
if (process.env.NODE_ENV !== 'production') {
  transports.push(
    new winston.transports.DailyRotateFile({
      filename: 'logs/app-%DATE%.log',
      maxFiles: '7d',
    }),
  );
}

// Add external services in production
if (process.env.NODE_ENV === 'production') {
  // Loggly
  if (process.env.LOGGLY_TOKEN) {
    transports.push(
      new Loggly({
        token: process.env.LOGGLY_TOKEN,
        subdomain: process.env.LOGGLY_SUBDOMAIN,
        tags: ['production', 'nestjs'],
        json: true,
      }),
    );
  }
  
  // Datadog
  if (process.env.DATADOG_API_KEY) {
    transports.push(
      new DatadogWinston({
        apiKey: process.env.DATADOG_API_KEY,
        service: 'my-app',
        ddtags: 'env:production',
      }),
    );
  }
}

export const loggerConfig = {
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json(),
  ),
  defaultMeta: {
    service: 'my-nestjs-app',
    environment: process.env.NODE_ENV,
  },
  transports,
};

// main.ts
import { NestFactory } from '@nestjs/core';
import { WinstonModule } from 'nest-winston';
import { loggerConfig } from './config/logger.config';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, {
    logger: WinstonModule.createLogger(loggerConfig),
  });
  
  await app.listen(3000);
}
bootstrap();
```

---

### **Error Handling:**

```typescript
import * as winston from 'winston';
import { Loggly } from 'winston-loggly-bulk';

const logglyTransport = new Loggly({
  token: process.env.LOGGLY_TOKEN,
  subdomain: process.env.LOGGLY_SUBDOMAIN,
  tags: ['nestjs'],
  json: true,
});

// Handle transport errors
logglyTransport.on('error', (err) => {
  console.error('Loggly transport error:', err);
  // Fallback to local file
  fs.appendFileSync('logs/transport-errors.log', 
    JSON.stringify({ error: err.message, timestamp: new Date() }) + '\n'
  );
});

logglyTransport.on('warn', (warn) => {
  console.warn('Loggly transport warning:', warn);
});

const logger = winston.createLogger({
  transports: [
    new winston.transports.Console(),
    logglyTransport,
    // Fallback file transport
    new winston.transports.File({ filename: 'logs/fallback.log' }),
  ],
});
```

---

### **Best Practices:**

```typescript
// ✅ GOOD - Environment-based configuration
const externalTransports = process.env.NODE_ENV === 'production' ? [
  new Loggly({ token: process.env.LOGGLY_TOKEN }),
] : [];

// ✅ GOOD - Error handling for transports
transport.on('error', (err) => console.error('Transport error:', err));

// ✅ GOOD - Include metadata
logger.info('User action', { userId, action, timestamp });

// ✅ GOOD - Use environment variables for secrets
token: process.env.LOGGLY_TOKEN

// ✅ GOOD - Batch logs for performance
bufferOptions: { size: 100 }

// ❌ BAD - Hardcoded credentials
token: 'abc123'  // Security risk!

// ❌ BAD - No error handling
// Transport fails silently

// ❌ BAD - Sending all logs in development
// Wastes quota and costs money

// ❌ BAD - No fallback
// If external service is down, logs are lost
```

**Key Takeaway:** Use external logging services like **Loggly**, **Papertrail**, **Datadog**, or **CloudWatch** for centralized log management, configure with environment variables, handle transport errors with fallbacks, and batch logs for better performance.

</details>

<details>
<summary><strong>21. How do you send logs to Elasticsearch/Kibana (ELK Stack)?</strong></summary>

**Answer:**

The ELK Stack (Elasticsearch, Logstash, Kibana) is a powerful log management solution. Logs are sent to Elasticsearch for storage and indexing, then visualized in Kibana. You can send logs directly from NestJS or use Logstash/Filebeat as intermediaries.

---

### **Architecture:**

```
NestJS App → [Method 1] → Elasticsearch → Kibana (Visualization)
            ↓ [Method 2]
            Logstash → Elasticsearch → Kibana
            ↓ [Method 3]
            Files → Filebeat → Elasticsearch → Kibana
```

---

### **Method 1: Direct to Elasticsearch (Winston)**

**Installation:**
```bash
npm install winston winston-elasticsearch
```

**Basic Configuration:**
```typescript
import * as winston from 'winston';
import { ElasticsearchTransport } from 'winston-elasticsearch';

const esTransportOpts = {
  level: 'info',
  clientOpts: {
    node: process.env.ELASTICSEARCH_URL || 'http://localhost:9200',
    auth: {
      username: process.env.ES_USERNAME || 'elastic',
      password: process.env.ES_PASSWORD || 'changeme',
    },
  },
  index: 'nestjs-logs',
};

const esTransport = new ElasticsearchTransport(esTransportOpts);

const logger = winston.createLogger({
  transports: [
    new winston.transports.Console(),
    esTransport,
  ],
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json(),
  ),
});

logger.info('User logged in', { userId: 'user-123', ip: '192.168.1.1' });
// Indexed in Elasticsearch under 'nestjs-logs' index
```

---

### **Method 2: Production Configuration**

```typescript
import * as winston from 'winston';
import { ElasticsearchTransport } from 'winston-elasticsearch';

// Elasticsearch transport configuration
const esTransportOpts = {
  level: 'info',
  
  // Elasticsearch client options
  clientOpts: {
    node: process.env.ELASTICSEARCH_URL,
    auth: {
      username: process.env.ES_USERNAME,
      password: process.env.ES_PASSWORD,
    },
    // SSL/TLS options
    ssl: {
      rejectUnauthorized: false, // For self-signed certs
    },
  },
  
  // Index configuration
  index: 'nestjs-logs',
  indexPrefix: 'nestjs',
  indexSuffixPattern: 'YYYY.MM.DD', // Daily indices: nestjs-logs-2025.12.30
  
  // Document type (deprecated in ES 7+)
  // type: '_doc',
  
  // Message formatting
  transformer: (logData) => {
    return {
      '@timestamp': logData.timestamp,
      severity: logData.level,
      message: logData.message,
      fields: {
        ...logData.meta,
        environment: process.env.NODE_ENV,
        service: 'my-nestjs-app',
        hostname: require('os').hostname(),
      },
    };
  },
  
  // Buffering for performance
  bufferLimit: 100,
  flushInterval: 2000, // Flush every 2 seconds
  
  // Error handling
  handleExceptions: true,
};

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.metadata(),
    winston.format.json(),
  ),
  defaultMeta: {
    service: 'my-nestjs-app',
    environment: process.env.NODE_ENV,
  },
  transports: [
    new winston.transports.Console(),
    new ElasticsearchTransport(esTransportOpts),
  ],
});

export default logger;
```

---

### **Method 3: NestJS Integration**

```typescript
// logger/elasticsearch-logger.config.ts
import * as winston from 'winston';
import { ElasticsearchTransport } from 'winston-elasticsearch';

const esTransport = new ElasticsearchTransport({
  level: 'info',
  clientOpts: {
    node: process.env.ELASTICSEARCH_URL,
    auth: {
      username: process.env.ES_USERNAME,
      password: process.env.ES_PASSWORD,
    },
  },
  index: 'nestjs-logs',
  indexPrefix: 'nestjs',
  indexSuffixPattern: 'YYYY.MM.DD',
});

// Handle Elasticsearch errors
esTransport.on('error', (error) => {
  console.error('Elasticsearch logging error:', error);
});

export const elasticsearchLoggerConfig = {
  transports: [
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.simple(),
      ),
    }),
    esTransport,
  ],
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json(),
  ),
};

// main.ts
import { NestFactory } from '@nestjs/core';
import { WinstonModule } from 'nest-winston';
import { elasticsearchLoggerConfig } from './logger/elasticsearch-logger.config';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, {
    logger: WinstonModule.createLogger(elasticsearchLoggerConfig),
  });
  
  await app.listen(3000);
}
bootstrap();
```

---

### **Method 4: Using Pino with Elasticsearch**

```bash
npm install pino pino-elasticsearch
```

```typescript
import pino from 'pino';
import { createWriteStream } from 'pino-elasticsearch';

const streamToElastic = createWriteStream({
  node: process.env.ELASTICSEARCH_URL || 'http://localhost:9200',
  auth: {
    username: process.env.ES_USERNAME || 'elastic',
    password: process.env.ES_PASSWORD || 'changeme',
  },
  index: 'nestjs-logs',
  consistency: 'one',
  'flush-bytes': 1000,
});

const logger = pino(
  {
    level: 'info',
  },
  streamToElastic,
);

logger.info({ userId: 'user-123' }, 'User logged in');
// Sent to Elasticsearch
```

**NestJS Pino + Elasticsearch:**
```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { LoggerModule } from 'nestjs-pino';
import { createWriteStream } from 'pino-elasticsearch';

const streamToElastic = createWriteStream({
  node: process.env.ELASTICSEARCH_URL,
  auth: {
    username: process.env.ES_USERNAME,
    password: process.env.ES_PASSWORD,
  },
  index: 'nestjs-logs',
});

@Module({
  imports: [
    LoggerModule.forRoot({
      pinoHttp: {
        level: 'info',
        stream: streamToElastic,
      },
    }),
  ],
})
export class AppModule {}
```

---

### **Method 5: Filebeat → Elasticsearch**

**Step 1: Log to Files**
```typescript
import * as winston from 'winston';
import 'winston-daily-rotate-file';

const logger = winston.createLogger({
  transports: [
    new winston.transports.DailyRotateFile({
      filename: 'logs/app-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      maxSize: '20m',
      maxFiles: '14d',
      format: winston.format.json(), // JSON for Filebeat
    }),
  ],
});

logger.info('User logged in', { userId: 'user-123' });
// Writes to logs/app-2025-12-30.log
```

**Step 2: Configure Filebeat (filebeat.yml)**
```yaml
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /app/logs/*.log
    json.keys_under_root: true
    json.add_error_key: true
    fields:
      service: my-nestjs-app
      environment: production

output.elasticsearch:
  hosts: ["localhost:9200"]
  username: "elastic"
  password: "changeme"
  index: "nestjs-logs-%{+yyyy.MM.dd}"

setup.kibana:
  host: "localhost:5601"
```

---

### **Method 6: Logstash → Elasticsearch**

**Step 1: Send Logs to Logstash**
```typescript
import * as winston from 'winston';

// Send logs to Logstash via TCP
const logger = winston.createLogger({
  transports: [
    new winston.transports.Http({
      host: 'localhost',
      port: 5000,
      path: '/',
      format: winston.format.json(),
    }),
  ],
});
```

**Step 2: Configure Logstash (logstash.conf)**
```conf
input {
  http {
    host => "0.0.0.0"
    port => 5000
    codec => json
  }
}

filter {
  # Add fields
  mutate {
    add_field => { "[@metadata][index]" => "nestjs-logs" }
  }
  
  # Parse timestamp
  date {
    match => [ "timestamp", "ISO8601" ]
    target => "@timestamp"
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    user => "elastic"
    password => "changeme"
    index => "%{[@metadata][index]}-%{+YYYY.MM.dd}"
  }
  
  # Debug output
  stdout { codec => rubydebug }
}
```

---

### **Method 7: Structured Logging for ELK**

```typescript
import * as winston from 'winston';
import { ElasticsearchTransport } from 'winston-elasticsearch';

const logger = winston.createLogger({
  transports: [
    new ElasticsearchTransport({
      clientOpts: { node: 'http://localhost:9200' },
      index: 'nestjs-logs',
      transformer: (logData) => {
        // Structure for Kibana analysis
        return {
          '@timestamp': new Date().toISOString(),
          severity: logData.level,
          message: logData.message,
          
          // Application context
          application: {
            name: 'my-nestjs-app',
            version: '1.0.0',
            environment: process.env.NODE_ENV,
          },
          
          // Server info
          server: {
            hostname: require('os').hostname(),
            pid: process.pid,
          },
          
          // User context
          user: logData.meta?.userId ? {
            id: logData.meta.userId,
          } : undefined,
          
          // Request context
          request: logData.meta?.requestId ? {
            id: logData.meta.requestId,
            method: logData.meta.method,
            url: logData.meta.url,
          } : undefined,
          
          // Error details
          error: logData.meta?.error ? {
            message: logData.meta.error,
            stack: logData.meta.stack,
          } : undefined,
          
          // Custom fields
          custom: logData.meta,
        };
      },
    }),
  ],
});

// Usage with structured data
logger.info('Order placed', {
  userId: 'user-123',
  requestId: 'req-456',
  method: 'POST',
  url: '/api/orders',
  orderId: 'order-789',
  total: 99.99,
});

// In Elasticsearch/Kibana, you can filter by:
// - application.environment: "production"
// - user.id: "user-123"
// - request.method: "POST"
// - custom.orderId: "order-789"
```

---

### **Method 8: Kibana Index Pattern**

**Create Index Pattern in Kibana:**

1. Open Kibana (http://localhost:5601)
2. Go to Management → Index Patterns
3. Create index pattern: `nestjs-logs-*`
4. Select time field: `@timestamp`
5. Save

**Search in Kibana:**
```
# Find all errors
severity: "error"

# Find logs for specific user
user.id: "user-123"

# Find logs in time range
@timestamp: [now-1h TO now]

# Combine filters
severity: "error" AND application.environment: "production"

# Full-text search
message: "payment failed"
```

---

### **Method 9: Complete NestJS + ELK Setup**

```typescript
// config/elk-logger.config.ts
import * as winston from 'winston';
import { ElasticsearchTransport } from 'winston-elasticsearch';
import 'winston-daily-rotate-file';

const transports: winston.transport[] = [
  // Console (always)
  new winston.transports.Console({
    format: winston.format.combine(
      winston.format.colorize(),
      winston.format.simple(),
    ),
  }),
];

// File backup (in case Elasticsearch is down)
transports.push(
  new winston.transports.DailyRotateFile({
    filename: 'logs/app-%DATE%.log',
    datePattern: 'YYYY-MM-DD',
    maxSize: '20m',
    maxFiles: '7d',
    format: winston.format.json(),
  }),
);

// Elasticsearch (production only)
if (process.env.ELASTICSEARCH_URL) {
  const esTransport = new ElasticsearchTransport({
    level: 'info',
    clientOpts: {
      node: process.env.ELASTICSEARCH_URL,
      auth: {
        username: process.env.ES_USERNAME,
        password: process.env.ES_PASSWORD,
      },
      ssl: {
        rejectUnauthorized: process.env.NODE_ENV === 'production',
      },
    },
    index: 'nestjs-logs',
    indexPrefix: 'nestjs',
    indexSuffixPattern: 'YYYY.MM.DD',
    bufferLimit: 100,
    flushInterval: 2000,
  });

  esTransport.on('error', (error) => {
    console.error('Elasticsearch transport error:', error);
  });

  transports.push(esTransport);
}

export const elkLoggerConfig = {
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.metadata(),
    winston.format.json(),
  ),
  defaultMeta: {
    service: 'my-nestjs-app',
    version: process.env.APP_VERSION || '1.0.0',
    environment: process.env.NODE_ENV,
    hostname: require('os').hostname(),
  },
  transports,
};

// main.ts
import { NestFactory } from '@nestjs/core';
import { WinstonModule } from 'nest-winston';
import { elkLoggerConfig } from './config/elk-logger.config';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, {
    logger: WinstonModule.createLogger(elkLoggerConfig),
  });
  
  await app.listen(3000);
  
  // Log startup
  const logger = app.get(WinstonModule.createLogger(elkLoggerConfig));
  logger.log('Application started successfully', 'Bootstrap');
}
bootstrap();

// Usage in service
import { Injectable, Inject } from '@nestjs/common';
import { WINSTON_MODULE_PROVIDER } from 'nest-winston';
import { Logger } from 'winston';

@Injectable()
export class OrderService {
  constructor(
    @Inject(WINSTON_MODULE_PROVIDER) private logger: Logger,
  ) {}

  async createOrder(orderData: any) {
    this.logger.info('Creating order', {
      context: 'OrderService',
      userId: orderData.userId,
      items: orderData.items.length,
      total: orderData.total,
    });
    
    try {
      const order = await this.orderRepository.save(orderData);
      
      this.logger.info('Order created successfully', {
        context: 'OrderService',
        orderId: order.id,
        userId: orderData.userId,
      });
      
      return order;
    } catch (error) {
      this.logger.error('Failed to create order', {
        context: 'OrderService',
        error: error.message,
        stack: error.stack,
        userId: orderData.userId,
      });
      throw error;
    }
  }
}
```

---

### **Method 10: Docker Compose Setup**

```yaml
# docker-compose.yml
version: '3.8'

services:
  # NestJS Application
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - ELASTICSEARCH_URL=http://elasticsearch:9200
      - ES_USERNAME=elastic
      - ES_PASSWORD=changeme
    depends_on:
      - elasticsearch
      - kibana
    volumes:
      - ./logs:/app/logs

  # Elasticsearch
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data

  # Kibana
  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch

  # Filebeat (optional)
  filebeat:
    image: docker.elastic.co/beats/filebeat:8.11.0
    volumes:
      - ./logs:/logs:ro
      - ./filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
    depends_on:
      - elasticsearch

volumes:
  elasticsearch-data:
```

**Run:**
```bash
docker-compose up -d

# Access Kibana
open http://localhost:5601

# View logs in Kibana
# 1. Create index pattern: nestjs-logs-*
# 2. Go to Discover
# 3. Search and filter logs
```

---

### **Best Practices:**

```typescript
// ✅ GOOD - Daily indices for easier management
indexSuffixPattern: 'YYYY.MM.DD'
// Creates: nestjs-logs-2025.12.30, nestjs-logs-2025.12.31

// ✅ GOOD - Buffer logs for performance
bufferLimit: 100,
flushInterval: 2000,

// ✅ GOOD - Structured logging
logger.info('Order placed', {
  orderId: 'order-123',
  userId: 'user-456',
  total: 99.99,
});

// ✅ GOOD - Include @timestamp
transformer: (logData) => ({
  '@timestamp': new Date().toISOString(),
  ...logData,
})

// ✅ GOOD - Fallback to files
transports: [
  esTransport,
  fileTransport, // Backup if ES is down
]

// ❌ BAD - Unstructured logs
logger.info(`Order ${orderId} placed by ${userId}`);
// Hard to query in Kibana

// ❌ BAD - No buffering
// Each log = separate HTTP request to Elasticsearch

// ❌ BAD - No error handling
// ES errors go unnoticed

// ❌ BAD - Sensitive data in logs
logger.info('Payment', { creditCard: '1234-5678-9012-3456' });
// Stored forever in Elasticsearch!
```

**Key Takeaway:** Send logs to Elasticsearch using **winston-elasticsearch** or **pino-elasticsearch**, configure with daily indices, use structured logging with proper fields, buffer logs for performance, and visualize/search in Kibana.

</details>

## Structured Logging

<details>
<summary><strong>22. What is structured logging?</strong></summary>

**Answer:**

Structured logging is the practice of logging data in a **consistent, machine-readable format** (typically JSON) with key-value pairs instead of plain text strings. This makes logs easier to parse, query, analyze, and monitor using log management tools.

---

### **Unstructured vs Structured Logging:**

**❌ Unstructured (String-based):**
```typescript
logger.info('User john@example.com logged in from IP 192.168.1.1 at 2025-12-30T10:30:45Z');
// Output: User john@example.com logged in from IP 192.168.1.1 at 2025-12-30T10:30:45Z

// Problems:
// - Hard to parse
// - Can't query by email or IP
// - No consistent format
// - Requires regex to extract data
```

**✅ Structured (JSON):**
```typescript
logger.info('User logged in', {
  email: 'john@example.com',
  ip: '192.168.1.1',
  timestamp: '2025-12-30T10:30:45Z',
});
// Output: {"level":"info","message":"User logged in","email":"john@example.com","ip":"192.168.1.1","timestamp":"2025-12-30T10:30:45Z"}

// Benefits:
// - Easy to parse
// - Queryable: "find all logs where email='john@example.com'"
// - Consistent structure
// - No regex needed
```

---

### **Key Concepts:**

**1. Key-Value Pairs:**
```typescript
// Each piece of data has a clear key
logger.info('Payment processed', {
  orderId: 'ORD-123',
  userId: 'user-456',
  amount: 99.99,
  currency: 'USD',
  paymentMethod: 'credit_card',
  status: 'success',
});

// JSON output:
{
  "level": "info",
  "message": "Payment processed",
  "orderId": "ORD-123",
  "userId": "user-456",
  "amount": 99.99,
  "currency": "USD",
  "paymentMethod": "credit_card",
  "status": "success"
}
```

**2. Consistent Schema:**
```typescript
// Define log structure
interface LogContext {
  userId?: string;
  requestId?: string;
  action: string;
  resource?: string;
  duration?: number;
  statusCode?: number;
}

function logAction(message: string, context: LogContext) {
  logger.info(message, context);
}

// All logs follow same structure
logAction('User created', {
  action: 'create',
  resource: 'user',
  userId: 'user-123',
});

logAction('Product updated', {
  action: 'update',
  resource: 'product',
  userId: 'user-456',
});
```

---

### **Implementation with Winston:**

```typescript
import * as winston from 'winston';

const logger = winston.createLogger({
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json(), // ✅ JSON format
  ),
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: 'app.log' }),
  ],
});

// Structured logging
logger.info('User login', {
  userId: 'user-123',
  email: 'john@example.com',
  ip: '192.168.1.1',
  userAgent: 'Mozilla/5.0...',
  loginMethod: 'password',
});

// Output:
{
  "level": "info",
  "message": "User login",
  "timestamp": "2025-12-30T10:30:45.123Z",
  "userId": "user-123",
  "email": "john@example.com",
  "ip": "192.168.1.1",
  "userAgent": "Mozilla/5.0...",
  "loginMethod": "password"
}
```

---

### **Implementation with Pino:**

```typescript
import pino from 'pino';

const logger = pino({
  // Pino uses JSON by default
});

// Structured logging (metadata first, message second)
logger.info(
  {
    userId: 'user-123',
    orderId: 'order-456',
    total: 99.99,
    items: 3,
  },
  'Order placed',
);

// Output:
{
  "level": 30,
  "time": 1703937045123,
  "userId": "user-123",
  "orderId": "order-456",
  "total": 99.99,
  "items": 3,
  "msg": "Order placed"
}
```

---

### **NestJS Implementation:**

```typescript
// logger/structured-logger.service.ts
import { Injectable, Inject } from '@nestjs/common';
import { WINSTON_MODULE_PROVIDER } from 'nest-winston';
import { Logger } from 'winston';

interface LogMetadata {
  userId?: string;
  requestId?: string;
  context?: string;
  [key: string]: any;
}

@Injectable()
export class StructuredLoggerService {
  constructor(
    @Inject(WINSTON_MODULE_PROVIDER)
    private readonly logger: Logger,
  ) {}

  log(message: string, metadata: LogMetadata = {}) {
    this.logger.info(message, metadata);
  }

  error(message: string, error: Error, metadata: LogMetadata = {}) {
    this.logger.error(message, {
      ...metadata,
      error: {
        message: error.message,
        stack: error.stack,
        name: error.name,
      },
    });
  }

  warn(message: string, metadata: LogMetadata = {}) {
    this.logger.warn(message, metadata);
  }
}

// Usage in service
import { Injectable } from '@nestjs/common';
import { StructuredLoggerService } from './logger/structured-logger.service';

@Injectable()
export class OrderService {
  constructor(private readonly logger: StructuredLoggerService) {}

  async createOrder(orderData: any) {
    this.logger.log('Creating order', {
      context: 'OrderService',
      userId: orderData.userId,
      items: orderData.items.length,
      total: orderData.total,
      currency: orderData.currency,
    });

    try {
      const order = await this.orderRepository.save(orderData);

      this.logger.log('Order created', {
        context: 'OrderService',
        orderId: order.id,
        userId: orderData.userId,
        status: order.status,
      });

      return order;
    } catch (error) {
      this.logger.error('Order creation failed', error, {
        context: 'OrderService',
        userId: orderData.userId,
      });
      throw error;
    }
  }
}
```

---

### **Common Fields for Structured Logs:**

```typescript
interface StructuredLog {
  // Core fields
  timestamp: string;        // ISO 8601 timestamp
  level: string;            // error, warn, info, debug
  message: string;          // Human-readable message
  
  // Application context
  service: string;          // my-nestjs-app
  version: string;          // 1.0.0
  environment: string;      // production, staging, development
  
  // Server context
  hostname: string;         // server-01
  pid: number;              // Process ID
  
  // Request context
  requestId?: string;       // Correlation ID
  userId?: string;          // Authenticated user
  sessionId?: string;       // Session identifier
  
  // HTTP context
  method?: string;          // GET, POST, PUT, DELETE
  url?: string;             // /api/users/123
  statusCode?: number;      // 200, 404, 500
  duration?: number;        // Request duration (ms)
  ip?: string;              // Client IP
  userAgent?: string;       // Browser/client info
  
  // Business context
  action?: string;          // create, update, delete
  resource?: string;        // user, order, product
  
  // Error context (if applicable)
  error?: {
    message: string;
    stack?: string;
    code?: string;
  };
  
  // Custom metadata
  [key: string]: any;
}

// Usage
const log: StructuredLog = {
  timestamp: new Date().toISOString(),
  level: 'info',
  message: 'User created',
  service: 'my-nestjs-app',
  version: '1.0.0',
  environment: 'production',
  hostname: 'server-01',
  pid: 12345,
  requestId: 'req-789',
  userId: 'user-123',
  method: 'POST',
  url: '/api/users',
  statusCode: 201,
  duration: 45,
  action: 'create',
  resource: 'user',
};

logger.info(log.message, log);
```

---

### **Querying Structured Logs:**

**With JSON logs, you can easily query:**

```bash
# Find all errors for specific user
jq 'select(.level=="error" and .userId=="user-123")' app.log

# Find slow requests (>1000ms)
jq 'select(.duration > 1000)' app.log

# Count requests by status code
jq -r '.statusCode' app.log | sort | uniq -c

# Find all payment-related logs
jq 'select(.resource=="payment")' app.log

# Group by action
jq -r '.action' app.log | sort | uniq -c
```

**In Elasticsearch/Kibana:**
```
# Find errors for specific user
userId:"user-123" AND level:"error"

# Find slow requests
duration:>1000

# Find specific action
action:"payment" AND status:"failed"

# Time-based query
timestamp:[now-1h TO now] AND level:"error"
```

---

### **Benefits:**

```typescript
// ✅ 1. Easy Parsing
const log = JSON.parse(logLine);
console.log(log.userId); // Direct access

// ✅ 2. Queryable
// SELECT * FROM logs WHERE userId='user-123' AND level='error'

// ✅ 3. Aggregation
// Count orders by status:
// SELECT status, COUNT(*) FROM logs GROUP BY status

// ✅ 4. Alerting
// Alert if: count(level='error' AND action='payment') > 10 in last 5min

// ✅ 5. Debugging
// Find all logs for a specific request:
// SELECT * FROM logs WHERE requestId='req-123' ORDER BY timestamp

// ✅ 6. Analytics
// Average request duration by endpoint:
// SELECT url, AVG(duration) FROM logs GROUP BY url
```

---

### **Anti-Patterns:**

```typescript
// ❌ BAD - Unstructured string
logger.info(`User ${userId} placed order ${orderId} for $${total}`);
// Can't query by userId or orderId

// ❌ BAD - Nested strings in JSON
logger.info('Event', {
  data: `User ${userId} did something`,  // String interpolation inside JSON
});

// ❌ BAD - Inconsistent field names
logger.info('Event 1', { user_id: '123' });
logger.info('Event 2', { userId: '456' });  // Different key!
logger.info('Event 3', { uid: '789' });     // Another different key!

// ❌ BAD - Mixed types
logger.info('Event 1', { amount: 99.99 });    // Number
logger.info('Event 2', { amount: '99.99' });  // String - inconsistent!

// ✅ GOOD - Structured with consistent schema
logger.info('User action', {
  userId: 'user-123',      // Always camelCase
  orderId: 'order-456',    // Consistent naming
  total: 99.99,            // Always number
  currency: 'USD',         // Always string
  action: 'order_placed',  // Consistent format
});
```

---

### **Production Example:**

```typescript
// logger/structured-logger.ts
import * as winston from 'winston';

interface BaseLogContext {
  service: string;
  version: string;
  environment: string;
  hostname: string;
  pid: number;
}

const baseContext: BaseLogContext = {
  service: 'my-nestjs-app',
  version: process.env.APP_VERSION || '1.0.0',
  environment: process.env.NODE_ENV || 'development',
  hostname: require('os').hostname(),
  pid: process.pid,
};

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json(),
  ),
  defaultMeta: baseContext,
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: 'logs/app.log' }),
  ],
});

export default logger;

// Usage
logger.info('Order placed', {
  userId: 'user-123',
  orderId: 'order-456',
  total: 99.99,
  items: 3,
  paymentMethod: 'credit_card',
});

// Output includes both baseContext and custom fields:
{
  "level": "info",
  "message": "Order placed",
  "timestamp": "2025-12-30T10:30:45.123Z",
  "service": "my-nestjs-app",
  "version": "1.0.0",
  "environment": "production",
  "hostname": "server-01",
  "pid": 12345,
  "userId": "user-123",
  "orderId": "order-456",
  "total": 99.99,
  "items": 3,
  "paymentMethod": "credit_card"
}
```

---

### **Comparison:**

| Aspect | Unstructured | Structured |
|--------|--------------|------------|
| **Format** | Plain text | JSON/Key-value |
| **Parsing** | Regex required | JSON.parse() |
| **Querying** | grep, text search | SQL-like, field queries |
| **Consistency** | Variable | Schema-based |
| **Aggregation** | Difficult | Easy |
| **Alerting** | Pattern matching | Field-based rules |
| **Debugging** | Manual search | Trace requests |
| **Analytics** | Limited | Full support |
| **Tool Support** | Basic | Excellent |

**Key Takeaway:** Structured logging uses **JSON format with key-value pairs** for consistent, machine-readable logs that are easy to parse, query, analyze, and monitor. Always log with structured metadata instead of string interpolation.

</details>

<details>
<summary><strong>23. Why is JSON format preferred for logs?</strong></summary>

**Answer:**

JSON (JavaScript Object Notation) is the preferred format for logs in modern applications because it's **machine-readable**, **parseable**, **queryable**, and **standardized**. It enables efficient log aggregation, analysis, and monitoring across distributed systems.

---

### **Key Reasons:**

#### **1. Machine-Readable**

```typescript
// ❌ Text log (hard to parse)
"2025-12-30 10:30:45 INFO User john@example.com logged in from 192.168.1.1"

// ✅ JSON log (easy to parse)
{
  "timestamp": "2025-12-30T10:30:45.123Z",
  "level": "info",
  "message": "User logged in",
  "email": "john@example.com",
  "ip": "192.168.1.1"
}

// Parsing JSON is simple:
const log = JSON.parse(logLine);
console.log(log.email);  // "john@example.com"
console.log(log.ip);     // "192.168.1.1"

// Parsing text requires complex regex:
const regex = /User (\S+) logged in from ([\d.]+)/;
const match = textLog.match(regex);
```

---

#### **2. Easy Querying**

```typescript
// JSON logs can be queried with field names

// Find all errors for specific user:
jq 'select(.level=="error" and .userId=="user-123")' logs.json

// Find slow requests (>1000ms):
jq 'select(.duration > 1000)' logs.json

// Count by status code:
jq -r '.statusCode' logs.json | sort | uniq -c

// In Elasticsearch:
userId:"user-123" AND level:"error"

// In Splunk:
source="app.log" userId="user-123" level="error"

// In SQL (if logs are in database):
SELECT * FROM logs WHERE userId='user-123' AND level='error'
```

---

#### **3. Type Safety**

```typescript
// JSON preserves data types
{
  "timestamp": "2025-12-30T10:30:45.123Z",  // String
  "level": "info",                          // String
  "userId": "user-123",                     // String
  "age": 25,                                // Number
  "total": 99.99,                           // Number (float)
  "isPremium": true,                        // Boolean
  "items": ["item1", "item2"],             // Array
  "metadata": {                             // Object
    "source": "web",
    "device": "mobile"
  }
}

// Text logs lose type information:
"age: 25"  // Is this a number or string "25"?
"isPremium: true"  // Is this boolean true or string "true"?
```

---

#### **4. Tool Compatibility**

```typescript
// JSON works with all modern log tools:

// ✅ Elasticsearch/Kibana
// ✅ Splunk
// ✅ Datadog
// ✅ New Relic
// ✅ CloudWatch Insights
// ✅ Grafana Loki
// ✅ LogDNA
// ✅ Papertrail

// Winston with JSON
import * as winston from 'winston';

const logger = winston.createLogger({
  format: winston.format.json(),  // ✅ JSON output
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: 'app.log' }),
  ],
});

logger.info('Order placed', { orderId: 'order-123', total: 99.99 });
// Output: {"level":"info","message":"Order placed","orderId":"order-123","total":99.99}
```

---

#### **5. Nested Data Support**

```typescript
// JSON handles complex nested structures
logger.info('Order placed', {
  order: {
    id: 'order-123',
    total: 99.99,
    items: [
      { id: 'item-1', name: 'Product A', price: 49.99 },
      { id: 'item-2', name: 'Product B', price: 50.00 },
    ],
    shipping: {
      address: '123 Main St',
      city: 'New York',
      country: 'USA',
    },
  },
  user: {
    id: 'user-456',
    email: 'john@example.com',
    isPremium: true,
  },
});

// Can query nested fields:
// order.total > 50
// order.items[0].price
// user.isPremium = true
```

---

#### **6. Aggregation and Analytics**

```typescript
// JSON enables complex analytics

// Average response time by endpoint:
SELECT 
  url, 
  AVG(duration) as avg_duration,
  COUNT(*) as request_count
FROM logs
WHERE level='info'
GROUP BY url
ORDER BY avg_duration DESC

// Error rate by service:
SELECT 
  service,
  COUNT(CASE WHEN level='error' THEN 1 END) * 100.0 / COUNT(*) as error_rate
FROM logs
GROUP BY service

// User activity over time:
SELECT 
  DATE(timestamp) as date,
  COUNT(DISTINCT userId) as active_users
FROM logs
GROUP BY DATE(timestamp)
```

---

#### **7. Standardization**

```typescript
// JSON provides a standard format across services

// Service A (NestJS)
logger.info('User created', {
  service: 'api',
  userId: 'user-123',
  timestamp: new Date().toISOString(),
});

// Service B (Python)
logger.info('Email sent', {
  service: 'email-service',
  userId: 'user-123',
  timestamp: datetime.now().isoformat(),
})

// Service C (Go)
log.Info("Payment processed", map[string]interface{}{
  "service": "payment-service",
  "userId": "user-123",
  "timestamp": time.Now().Format(time.RFC3339),
})

// All services produce JSON logs that can be aggregated:
// SELECT * FROM logs WHERE userId='user-123' ORDER BY timestamp
```

---

#### **8. No Parsing Errors**

```typescript
// ❌ Text logs are fragile
"User john@example.com logged in"  // ✅ Works
"User john@exam ple.com logged in"  // ❌ Space breaks parsing
"User logged in"  // ❌ Missing email

// ✅ JSON is robust
{ "email": "john@example.com" }  // ✅ Works
{ "email": "john@exam ple.com" }  // ✅ Still works (space in value)
{ "email": null }  // ✅ Explicit null
{ }  // ✅ Empty object (no email)

// JSON parsing is binary: it works or throws error
try {
  const log = JSON.parse(logLine);
} catch (error) {
  console.error('Invalid JSON log');
}
```

---

#### **9. Performance**

```typescript
// JSON parsing is fast (native in JavaScript)
const log = JSON.parse(jsonString);  // Native, optimized

// Text parsing is slow (regex)
const regex = /User (\S+) from IP ([\d.]+) at (\S+)/;
const match = text.match(regex);

// Benchmark:
// JSON.parse: ~100,000 ops/sec
// Regex parsing: ~10,000 ops/sec
// JSON is 10x faster!
```

---

#### **10. Stream Processing**

```typescript
import { createReadStream } from 'fs';
import { createInterface } from 'readline';

// Process large JSON log files line by line
const fileStream = createReadStream('logs/app.log');
const rl = createInterface({
  input: fileStream,
  crlfDelay: Infinity,
});

for await (const line of rl) {
  const log = JSON.parse(line);
  
  // Process each log
  if (log.level === 'error') {
    console.error('Error found:', log);
  }
  
  // Aggregate metrics
  if (log.duration > 1000) {
    console.warn('Slow request:', log.url, log.duration);
  }
}
```

---

### **Production Example:**

```typescript
import * as winston from 'winston';

const logger = winston.createLogger({
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json(),  // ✅ JSON format
  ),
  defaultMeta: {
    service: 'my-nestjs-app',
    version: '1.0.0',
    environment: process.env.NODE_ENV,
  },
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: 'logs/app.log' }),
  ],
});

// Log with structured data
logger.info('Order placed', {
  orderId: 'order-123',
  userId: 'user-456',
  total: 99.99,
  currency: 'USD',
  items: 3,
  paymentMethod: 'credit_card',
  shippingCountry: 'USA',
});

// Output (one line, formatted here for readability):
{
  "level": "info",
  "message": "Order placed",
  "timestamp": "2025-12-30T10:30:45.123Z",
  "service": "my-nestjs-app",
  "version": "1.0.0",
  "environment": "production",
  "orderId": "order-123",
  "userId": "user-456",
  "total": 99.99,
  "currency": "USD",
  "items": 3,
  "paymentMethod": "credit_card",
  "shippingCountry": "USA"
}
```

---

### **Comparison Table:**

| Feature | Text Logs | JSON Logs |
|---------|-----------|----------|
| **Parsing** | Regex (complex) | JSON.parse() (simple) |
| **Querying** | grep, text search | Field-based queries |
| **Type Safety** | All strings | Preserves types |
| **Nested Data** | Difficult | Native support |
| **Tool Support** | Limited | Excellent |
| **Analytics** | Manual | SQL/aggregations |
| **Performance** | Slow regex | Fast parsing |
| **Standardization** | Variable | JSON standard |
| **Error Handling** | Partial matches | Binary (works/fails) |
| **Human Readable** | ✅ Easy | ❌ Harder (but tools help) |

---

### **Human Readability Solution:**

```typescript
// Problem: JSON is hard to read during development
{"level":"info","message":"User logged in","userId":"user-123","timestamp":"2025-12-30T10:30:45.123Z"}

// Solution: Use pino-pretty or winston formatting in development
import * as winston from 'winston';

const logger = winston.createLogger({
  format: process.env.NODE_ENV === 'production'
    ? winston.format.json()  // ✅ JSON in production
    : winston.format.combine(  // 👁️ Pretty in development
        winston.format.colorize(),
        winston.format.simple(),
      ),
  transports: [new winston.transports.Console()],
});

// Development output (colorized):
// info: User logged in {"userId":"user-123"}

// Production output (JSON):
// {"level":"info","message":"User logged in","userId":"user-123"}
```

**With Pino:**
```typescript
import pino from 'pino';

const logger = pino(
  process.env.NODE_ENV === 'production'
    ? {}  // Raw JSON in production
    : {
        transport: {
          target: 'pino-pretty',  // Pretty in development
          options: { colorize: true },
        },
      },
);
```

---

### **Best Practices:**

```typescript
// ✅ GOOD - JSON in production
const logger = winston.createLogger({
  format: winston.format.json(),
});

// ✅ GOOD - Consistent field names
logger.info('Event', {
  userId: 'user-123',  // Always camelCase
  orderId: 'order-456',
  timestamp: new Date().toISOString(),
});

// ✅ GOOD - Include metadata
logger.info('Payment processed', {
  orderId: 'order-123',
  amount: 99.99,
  currency: 'USD',
  gateway: 'stripe',
});

// ✅ GOOD - One log per line (JSONL format)
{"level":"info","message":"Log 1"}
{"level":"info","message":"Log 2"}
{"level":"info","message":"Log 3"}

// ❌ BAD - String interpolation
logger.info(`User ${userId} placed order ${orderId}`);
// Can't query by userId or orderId

// ❌ BAD - Mixed format
logger.info('Event', { data: `String ${value}` });
// Mixing string interpolation with JSON

// ❌ BAD - Array of logs (not JSONL)
[
  {"level":"info","message":"Log 1"},
  {"level":"info","message":"Log 2"}
]
// Tools expect one JSON object per line
```

---

### **Tool Integration:**

```typescript
// Elasticsearch: Automatically indexes JSON fields
POST /logs/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "level": "error" } },
        { "match": { "userId": "user-123" } }
      ]
    }
  }
}

// CloudWatch Insights: Query JSON fields
fields @timestamp, level, message, userId
| filter level = "error" and userId = "user-123"
| sort @timestamp desc

// Datadog: Faceted search on JSON fields
level:error service:my-app @userId:user-123

// Grafana Loki: LogQL queries
{service="my-app"} | json | level="error" and userId="user-123"
```

**Key Takeaway:** JSON format is preferred for logs because it's **machine-readable**, **parseable**, **queryable**, supports **nested data** and **type safety**, works with all modern tools, enables **analytics**, and provides **standardization** across distributed systems.

</details>

<details>
<summary><strong>24. How do you include metadata in logs (request ID, user ID)?</strong></summary>

**Answer:**

Including metadata (context information like request ID, user ID, session ID) in logs enables **tracing requests**, **debugging issues**, **analyzing user behavior**, and **correlating logs** across services. Metadata is added as key-value pairs in structured logs.

---

### **Method 1: Direct Metadata in Log Calls**

```typescript
import * as winston from 'winston';

const logger = winston.createLogger({
  format: winston.format.json(),
  transports: [new winston.transports.Console()],
});

// Include metadata directly
logger.info('User logged in', {
  userId: 'user-123',
  requestId: 'req-456',
  sessionId: 'sess-789',
  ip: '192.168.1.1',
  userAgent: 'Mozilla/5.0...',
});

// Output:
{
  "level": "info",
  "message": "User logged in",
  "userId": "user-123",
  "requestId": "req-456",
  "sessionId": "sess-789",
  "ip": "192.168.1.1",
  "userAgent": "Mozilla/5.0..."
}
```

---

### **Method 2: Default Metadata (winston.defaultMeta)**

```typescript
import * as winston from 'winston';

const logger = winston.createLogger({
  format: winston.format.json(),
  defaultMeta: {
    service: 'my-nestjs-app',
    version: '1.0.0',
    environment: process.env.NODE_ENV,
    hostname: require('os').hostname(),
    pid: process.pid,
  },
  transports: [new winston.transports.Console()],
});

logger.info('Application started');
// Output includes defaultMeta:
{
  "level": "info",
  "message": "Application started",
  "service": "my-nestjs-app",
  "version": "1.0.0",
  "environment": "production",
  "hostname": "server-01",
  "pid": 12345
}
```

---

### **Method 3: Child Loggers**

```typescript
import * as winston from 'winston';

const logger = winston.createLogger({
  format: winston.format.json(),
  transports: [new winston.transports.Console()],
});

// Create child logger with request context
const requestLogger = logger.child({
  requestId: 'req-123',
  userId: 'user-456',
});

// All logs from this child include requestId and userId
requestLogger.info('Fetching user data');
requestLogger.info('Data fetched successfully');
requestLogger.info('Response sent');

// Each log includes:
{
  "level": "info",
  "message": "...",
  "requestId": "req-123",
  "userId": "user-456"
}
```

---

### **Method 4: Middleware for Request Context (NestJS)**

```typescript
// middleware/logger.middleware.ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';
import { v4 as uuidv4 } from 'uuid';

// Extend Express Request to include custom properties
declare global {
  namespace Express {
    interface Request {
      id?: string;
      logger?: any;
    }
  }
}

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  constructor(private readonly logger: Logger) {}

  use(req: Request, res: Response, next: NextFunction) {
    // Generate request ID
    req.id = req.headers['x-request-id'] as string || uuidv4();

    // Create child logger with request metadata
    req.logger = this.logger.child({
      requestId: req.id,
      method: req.method,
      url: req.url,
      ip: req.ip,
      userAgent: req.headers['user-agent'],
      userId: req.user?.id,  // If authenticated
    });

    // Log incoming request
    req.logger.info('Incoming request');

    // Log response
    const start = Date.now();
    res.on('finish', () => {
      const duration = Date.now() - start;
      req.logger.info('Request completed', {
        statusCode: res.statusCode,
        duration: `${duration}ms`,
      });
    });

    next();
  }
}

// app.module.ts
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './middleware/logger.middleware';

@Module({})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(LoggerMiddleware).forRoutes('*');
  }
}

// Usage in controller
import { Controller, Get, Req } from '@nestjs/common';
import { Request } from 'express';

@Controller('users')
export class UsersController {
  @Get()
  getUsers(@Req() req: Request) {
    // Use logger from request
    req.logger.info('Fetching all users');
    
    const users = this.usersService.findAll();
    
    req.logger.info('Users fetched', { count: users.length });
    
    return users;
  }
}
```

---

### **Method 5: AsyncLocalStorage (Node.js Context)**

```typescript
import { AsyncLocalStorage } from 'async_hooks';
import * as winston from 'winston';
import { v4 as uuidv4 } from 'uuid';

// Create async context
const asyncLocalStorage = new AsyncLocalStorage();

// Custom format to include context
const contextFormat = winston.format((info) => {
  const context = asyncLocalStorage.getStore();
  if (context) {
    return { ...info, ...context };
  }
  return info;
})();

const logger = winston.createLogger({
  format: winston.format.combine(contextFormat, winston.format.json()),
  transports: [new winston.transports.Console()],
});

// Middleware to set context
import { Injectable, NestMiddleware } from '@nestjs/common';

@Injectable()
export class ContextMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    const context = {
      requestId: req.headers['x-request-id'] || uuidv4(),
      userId: req.user?.id,
      sessionId: req.session?.id,
    };

    asyncLocalStorage.run(context, () => {
      next();
    });
  }
}

// Usage - no need to pass context explicitly!
import { Injectable } from '@nestjs/common';

@Injectable()
export class UserService {
  getUser(id: string) {
    logger.info('Fetching user', { userId: id });
    // Automatically includes requestId, userId from context
  }
}
```

---

### **Method 6: Interceptor for Request Metadata**

```typescript
// interceptors/logging.interceptor.ts
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
  Logger,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';
import { v4 as uuidv4 } from 'uuid';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  private readonly logger = new Logger('HTTP');

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { method, url, body, headers } = request;
    
    // Generate request ID
    const requestId = headers['x-request-id'] || uuidv4();
    
    // Extract user info
    const userId = request.user?.id;
    const userEmail = request.user?.email;

    const metadata = {
      requestId,
      method,
      url,
      userId,
      userEmail,
      ip: request.ip,
      userAgent: headers['user-agent'],
    };

    // Log request
    this.logger.log('Incoming request', JSON.stringify(metadata));

    const now = Date.now();

    return next.handle().pipe(
      tap({
        next: (data) => {
          const response = context.switchToHttp().getResponse();
          this.logger.log('Request completed', JSON.stringify({
            ...metadata,
            statusCode: response.statusCode,
            duration: `${Date.now() - now}ms`,
          }));
        },
        error: (error) => {
          this.logger.error('Request failed', JSON.stringify({
            ...metadata,
            error: error.message,
            duration: `${Date.now() - now}ms`,
          }));
        },
      }),
    );
  }
}

// app.controller.ts
import { Controller, UseInterceptors } from '@nestjs/common';
import { LoggingInterceptor } from './interceptors/logging.interceptor';

@Controller()
@UseInterceptors(LoggingInterceptor)
export class AppController {
  // All routes automatically get request metadata in logs
}
```

---

### **Method 7: Custom Logger Service**

```typescript
// logger/metadata-logger.service.ts
import { Injectable, Scope } from '@nestjs/common';
import { Logger } from 'winston';

interface LogMetadata {
  requestId?: string;
  userId?: string;
  sessionId?: string;
  correlationId?: string;
  [key: string]: any;
}

@Injectable({ scope: Scope.REQUEST })
export class MetadataLoggerService {
  private metadata: LogMetadata = {};

  constructor(private readonly logger: Logger) {}

  setMetadata(metadata: LogMetadata) {
    this.metadata = { ...this.metadata, ...metadata };
  }

  private formatLog(message: string, data?: any) {
    return {
      message,
      ...this.metadata,  // Always include metadata
      ...data,
    };
  }

  info(message: string, data?: any) {
    this.logger.info(this.formatLog(message, data));
  }

  error(message: string, error: Error, data?: any) {
    this.logger.error(this.formatLog(message, {
      ...data,
      error: {
        message: error.message,
        stack: error.stack,
      },
    }));
  }

  warn(message: string, data?: any) {
    this.logger.warn(this.formatLog(message, data));
  }
}

// Usage in service
import { Injectable } from '@nestjs/common';
import { MetadataLoggerService } from './logger/metadata-logger.service';

@Injectable()
export class OrderService {
  constructor(private readonly logger: MetadataLoggerService) {}

  createOrder(orderData: any, requestContext: any) {
    // Set metadata once
    this.logger.setMetadata({
      requestId: requestContext.requestId,
      userId: requestContext.userId,
    });

    // All subsequent logs include metadata automatically
    this.logger.info('Creating order', { orderId: orderData.id });
    this.logger.info('Validating order');
    this.logger.info('Processing payment');
    this.logger.info('Order created successfully');
  }
}
```

---

### **Method 8: Pino with Request Context**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { LoggerModule } from 'nestjs-pino';

@Module({
  imports: [
    LoggerModule.forRoot({
      pinoHttp: {
        // Automatically adds request metadata
        customProps: (req, res) => ({
          requestId: req.id,
          userId: req.user?.id,
          userEmail: req.user?.email,
          sessionId: req.session?.id,
          ip: req.headers['x-forwarded-for'] || req.ip,
        }),
        
        // Automatically generates request ID
        genReqId: (req) => req.headers['x-request-id'] || require('uuid').v4(),
      },
    }),
  ],
})
export class AppModule {}

// Usage in service
import { Injectable } from '@nestjs/common';
import { PinoLogger } from 'nestjs-pino';

@Injectable()
export class UserService {
  constructor(private readonly logger: PinoLogger) {
    this.logger.setContext(UserService.name);
  }

  getUser(id: string) {
    // Automatically includes requestId, userId from request
    this.logger.info({ targetUserId: id }, 'Fetching user');
  }
}
```

---

### **Method 9: Complete Production Example**

```typescript
// config/logger.config.ts
import * as winston from 'winston';

interface RequestMetadata {
  requestId: string;
  userId?: string;
  sessionId?: string;
  ip: string;
  method: string;
  url: string;
  userAgent: string;
}

export const createLogger = () => {
  return winston.createLogger({
    level: 'info',
    format: winston.format.combine(
      winston.format.timestamp(),
      winston.format.errors({ stack: true }),
      winston.format.json(),
    ),
    defaultMeta: {
      service: 'my-nestjs-app',
      version: process.env.APP_VERSION || '1.0.0',
      environment: process.env.NODE_ENV,
      hostname: require('os').hostname(),
      pid: process.pid,
    },
    transports: [
      new winston.transports.Console(),
      new winston.transports.File({ filename: 'logs/app.log' }),
    ],
  });
};

// middleware/request-logger.middleware.ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';
import { v4 as uuidv4 } from 'uuid';

@Injectable()
export class RequestLoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    // Generate or extract request ID
    const requestId = req.headers['x-request-id'] as string || uuidv4();
    req.id = requestId;

    // Add request ID to response headers
    res.setHeader('X-Request-ID', requestId);

    // Create metadata
    const metadata = {
      requestId,
      userId: req.user?.id,
      sessionId: req.session?.id,
      ip: req.headers['x-forwarded-for'] || req.ip,
      method: req.method,
      url: req.url,
      userAgent: req.headers['user-agent'],
    };

    // Attach metadata to request
    req.metadata = metadata;

    next();
  }
}

// Usage in service
import { Injectable, Inject } from '@nestjs/common';
import { REQUEST } from '@nestjs/core';
import { Request } from 'express';
import { Logger } from 'winston';

@Injectable({ scope: Scope.REQUEST })
export class OrderService {
  constructor(
    @Inject(REQUEST) private readonly request: Request,
    private readonly logger: Logger,
  ) {}

  createOrder(orderData: any) {
    // Log with request metadata
    this.logger.info('Creating order', {
      ...this.request.metadata,  // Include all request metadata
      orderId: orderData.id,
      total: orderData.total,
    });
  }
}
```

---

### **Common Metadata Fields:**

```typescript
interface LogMetadata {
  // Request context
  requestId: string;        // Unique request identifier
  correlationId?: string;   // Cross-service request tracking
  
  // User context
  userId?: string;          // Authenticated user ID
  userEmail?: string;       // User email
  sessionId?: string;       // Session identifier
  
  // HTTP context
  method?: string;          // GET, POST, PUT, DELETE
  url?: string;             // Request URL
  statusCode?: number;      // Response status
  duration?: number;        // Request duration (ms)
  
  // Client context
  ip?: string;              // Client IP address
  userAgent?: string;       // Browser/client info
  country?: string;         // Geo location
  
  // Application context
  service: string;          // Service name
  version: string;          // Application version
  environment: string;      // prod, staging, dev
  hostname: string;         // Server hostname
  pid: number;              // Process ID
  
  // Business context
  action?: string;          // Business action
  resource?: string;        // Resource type
  resourceId?: string;      // Resource ID
}
```

---

### **Best Practices:**

```typescript
// ✅ GOOD - Include request ID
logger.info('User created', {
  requestId: 'req-123',
  userId: 'user-456',
});

// ✅ GOOD - Use child loggers
const requestLogger = logger.child({ requestId: 'req-123' });
requestLogger.info('Processing');

// ✅ GOOD - Consistent field names
userId (always camelCase)

// ✅ GOOD - Include correlation ID for distributed tracing
logger.info('API call', {
  requestId: 'req-123',
  correlationId: 'corr-456',  // Same across services
});

// ❌ BAD - No request ID
logger.info('User created');  // Can't trace this log

// ❌ BAD - Inconsistent naming
logger.info('Event 1', { user_id: '123' });
logger.info('Event 2', { userId: '456' });

// ❌ BAD - Sensitive data
logger.info('Login', {
  password: 'secret123',  // Never log passwords!
});
```

**Key Takeaway:** Include metadata like **requestId**, **userId**, **sessionId** using **child loggers**, **middleware**, **interceptors**, or **AsyncLocalStorage** to enable request tracing, debugging, and log correlation across distributed systems.

</details>

<details>
<summary><strong>25. How do you implement correlation IDs for request tracking?</strong></summary>

**Answer:**

Correlation IDs (also called trace IDs or transaction IDs) are **unique identifiers** that track a single request as it flows through multiple services in a distributed system. They enable **end-to-end tracing**, **debugging**, and **monitoring** across microservices.

---

### **Why Correlation IDs?**

```typescript
// Without correlation ID:
// Service A logs:
"User login request received"

// Service B logs:
"User authentication successful"

// Service C logs:
"User profile fetched"

// Problem: Can't connect these logs to the same user request!

// With correlation ID:
// Service A: { correlationId: "corr-123", message: "User login request" }
// Service B: { correlationId: "corr-123", message: "Authentication successful" }
// Service C: { correlationId: "corr-123", message: "Profile fetched" }

// ✅ Now you can trace the entire request flow!
```

---

### **Method 1: Generate Correlation ID in Gateway/Entry Point**

```typescript
// middleware/correlation-id.middleware.ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';
import { v4 as uuidv4 } from 'uuid';

declare global {
  namespace Express {
    interface Request {
      correlationId?: string;
    }
  }
}

@Injectable()
export class CorrelationIdMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    // Check if correlation ID exists in headers (from upstream service)
    const correlationId = 
      req.headers['x-correlation-id'] as string ||
      req.headers['x-request-id'] as string ||
      uuidv4();  // Generate new if not present

    // Store in request
    req.correlationId = correlationId;

    // Add to response headers for client/downstream services
    res.setHeader('X-Correlation-ID', correlationId);

    next();
  }
}

// app.module.ts
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { CorrelationIdMiddleware } from './middleware/correlation-id.middleware';

@Module({})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(CorrelationIdMiddleware).forRoutes('*');
  }
}
```

---

### **Method 2: Log Correlation ID in Every Log**

```typescript
// logger/correlation-logger.service.ts
import { Injectable, Scope, Inject } from '@nestjs/common';
import { REQUEST } from '@nestjs/core';
import { Request } from 'express';
import * as winston from 'winston';

@Injectable({ scope: Scope.REQUEST })
export class CorrelationLoggerService {
  private logger: winston.Logger;

  constructor(
    @Inject(REQUEST) private readonly request: Request,
    @Inject('WINSTON_LOGGER') private readonly baseLogger: winston.Logger,
  ) {
    // Create child logger with correlation ID
    this.logger = this.baseLogger.child({
      correlationId: this.request.correlationId,
      service: 'user-service',
    });
  }

  log(message: string, metadata?: any) {
    this.logger.info(message, metadata);
  }

  error(message: string, error: Error, metadata?: any) {
    this.logger.error(message, {
      ...metadata,
      error: {
        message: error.message,
        stack: error.stack,
      },
    });
  }
}

// Usage in service
import { Injectable } from '@nestjs/common';
import { CorrelationLoggerService } from './logger/correlation-logger.service';

@Injectable()
export class UserService {
  constructor(private readonly logger: CorrelationLoggerService) {}

  async getUser(id: string) {
    // All logs automatically include correlationId
    this.logger.log('Fetching user', { userId: id });
    
    const user = await this.userRepository.findOne(id);
    
    this.logger.log('User fetched', { userId: id });
    
    return user;
  }
}

// Log output:
{
  "level": "info",
  "message": "Fetching user",
  "correlationId": "corr-123-456",  // ✅ Automatically included
  "service": "user-service",
  "userId": "user-789"
}
```

---

### **Method 3: AsyncLocalStorage (Node.js Context)**

```typescript
// context/correlation-context.ts
import { AsyncLocalStorage } from 'async_hooks';

interface CorrelationContext {
  correlationId: string;
  userId?: string;
  sessionId?: string;
}

export class CorrelationContextManager {
  private static asyncLocalStorage = new AsyncLocalStorage<CorrelationContext>();

  static run<T>(context: CorrelationContext, callback: () => T): T {
    return this.asyncLocalStorage.run(context, callback);
  }

  static getContext(): CorrelationContext | undefined {
    return this.asyncLocalStorage.getStore();
  }

  static getCorrelationId(): string | undefined {
    return this.getContext()?.correlationId;
  }
}

// middleware/correlation-context.middleware.ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';
import { v4 as uuidv4 } from 'uuid';
import { CorrelationContextManager } from '../context/correlation-context';

@Injectable()
export class CorrelationContextMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    const correlationId = 
      req.headers['x-correlation-id'] as string || uuidv4();

    const context = {
      correlationId,
      userId: req.user?.id,
      sessionId: req.session?.id,
    };

    // Store context in AsyncLocalStorage
    CorrelationContextManager.run(context, () => {
      res.setHeader('X-Correlation-ID', correlationId);
      next();
    });
  }
}

// logger/context-aware-logger.ts
import * as winston from 'winston';
import { CorrelationContextManager } from '../context/correlation-context';

// Custom format that automatically adds correlation ID
const correlationFormat = winston.format((info) => {
  const context = CorrelationContextManager.getContext();
  if (context) {
    return { ...info, ...context };
  }
  return info;
})();

export const logger = winston.createLogger({
  format: winston.format.combine(
    winston.format.timestamp(),
    correlationFormat,  // ✅ Auto-injects correlation ID
    winston.format.json(),
  ),
  transports: [new winston.transports.Console()],
});

// Usage - no need to pass correlation ID!
import { Injectable } from '@nestjs/common';
import { logger } from './logger/context-aware-logger';

@Injectable()
export class OrderService {
  createOrder(orderData: any) {
    // Correlation ID automatically included from context
    logger.info('Creating order', { orderId: orderData.id });
    logger.info('Validating order');
    logger.info('Processing payment');
  }
}
```

---

### **Method 4: Propagate to External Services (HTTP Calls)**

```typescript
// http/correlation-http.service.ts
import { Injectable, Inject } from '@nestjs/common';
import { HttpService } from '@nestjs/axios';
import { REQUEST } from '@nestjs/core';
import { Request } from 'express';
import { firstValueFrom } from 'rxjs';

@Injectable({ scope: Scope.REQUEST })
export class CorrelationHttpService {
  constructor(
    @Inject(REQUEST) private readonly request: Request,
    private readonly httpService: HttpService,
  ) {}

  async get<T>(url: string): Promise<T> {
    // Propagate correlation ID to downstream service
    const response = await firstValueFrom(
      this.httpService.get<T>(url, {
        headers: {
          'X-Correlation-ID': this.request.correlationId,  // ✅ Propagate!
        },
      }),
    );
    return response.data;
  }

  async post<T>(url: string, data: any): Promise<T> {
    const response = await firstValueFrom(
      this.httpService.post<T>(url, data, {
        headers: {
          'X-Correlation-ID': this.request.correlationId,
        },
      }),
    );
    return response.data;
  }
}

// Usage
import { Injectable } from '@nestjs/common';
import { CorrelationHttpService } from './http/correlation-http.service';

@Injectable()
export class PaymentService {
  constructor(private readonly http: CorrelationHttpService) {}

  async processPayment(paymentData: any) {
    // Call external payment service with correlation ID
    const result = await this.http.post('https://payment-api.com/charge', paymentData);
    // Payment service will log with same correlation ID!
    return result;
  }
}
```

---

### **Method 5: Axios Interceptor**

```typescript
// interceptors/correlation-axios.interceptor.ts
import axios from 'axios';
import { CorrelationContextManager } from '../context/correlation-context';

// Add correlation ID to all outgoing requests
axios.interceptors.request.use((config) => {
  const correlationId = CorrelationContextManager.getCorrelationId();
  
  if (correlationId) {
    config.headers['X-Correlation-ID'] = correlationId;
  }
  
  return config;
});

// Log correlation ID in responses
axios.interceptors.response.use(
  (response) => {
    const correlationId = response.headers['x-correlation-id'];
    console.log(`Response from ${response.config.url} [${correlationId}]`);
    return response;
  },
  (error) => {
    const correlationId = error.config?.headers['X-Correlation-ID'];
    console.error(`Error in request [${correlationId}]:`, error.message);
    return Promise.reject(error);
  },
);
```

---

### **Method 6: Database Queries with Correlation ID**

```typescript
// database/correlation-repository.ts
import { Injectable, Inject } from '@nestjs/common';
import { REQUEST } from '@nestjs/core';
import { Request } from 'express';
import { DataSource } from 'typeorm';
import { logger } from './logger/context-aware-logger';

@Injectable({ scope: Scope.REQUEST })
export class CorrelationRepository {
  constructor(
    @Inject(REQUEST) private readonly request: Request,
    private readonly dataSource: DataSource,
  ) {}

  async executeQuery(query: string, params: any[]) {
    const correlationId = this.request.correlationId;
    
    logger.info('Executing database query', {
      correlationId,
      query,
      params,
    });

    const start = Date.now();
    
    try {
      const result = await this.dataSource.query(query, params);
      
      logger.info('Query executed successfully', {
        correlationId,
        duration: `${Date.now() - start}ms`,
        rowCount: result.length,
      });
      
      return result;
    } catch (error) {
      logger.error('Query failed', error, {
        correlationId,
        query,
        duration: `${Date.now() - start}ms`,
      });
      throw error;
    }
  }
}
```

---

### **Method 7: Message Queue (RabbitMQ, Kafka)**

```typescript
// messaging/correlation-message.service.ts
import { Injectable, Inject } from '@nestjs/common';
import { REQUEST } from '@nestjs/core';
import { Request } from 'express';
import * as amqp from 'amqplib';

@Injectable({ scope: Scope.REQUEST })
export class CorrelationMessageService {
  constructor(
    @Inject(REQUEST) private readonly request: Request,
    private readonly channel: amqp.Channel,
  ) {}

  async publishMessage(queue: string, message: any) {
    const correlationId = this.request.correlationId;

    // Include correlation ID in message
    const messageWithCorrelation = {
      ...message,
      correlationId,  // ✅ Propagate to message consumer
      timestamp: new Date().toISOString(),
    };

    await this.channel.sendToQueue(
      queue,
      Buffer.from(JSON.stringify(messageWithCorrelation)),
      {
        correlationId,  // Also in AMQP properties
      },
    );

    logger.info('Message published', {
      correlationId,
      queue,
      messageId: message.id,
    });
  }
}

// Consumer extracts correlation ID
export class MessageConsumer {
  async consumeMessage(msg: amqp.ConsumeMessage) {
    const correlationId = msg.properties.correlationId;
    const messageData = JSON.parse(msg.content.toString());

    // Set correlation ID for this processing context
    CorrelationContextManager.run({ correlationId }, () => {
      this.processMessage(messageData);
    });
  }

  processMessage(data: any) {
    // All logs will include the correlation ID
    logger.info('Processing message', { messageId: data.id });
  }
}
```

---

### **Method 8: Complete Production Example**

```typescript
// correlation/correlation.module.ts
import { Module, Global } from '@nestjs/common';
import { CorrelationService } from './correlation.service';
import { CorrelationIdMiddleware } from './correlation-id.middleware';

@Global()
@Module({
  providers: [CorrelationService],
  exports: [CorrelationService],
})
export class CorrelationModule {}

// correlation/correlation.service.ts
import { Injectable } from '@nestjs/common';
import { AsyncLocalStorage } from 'async_hooks';
import { v4 as uuidv4 } from 'uuid';

interface CorrelationContext {
  correlationId: string;
  requestId: string;
  userId?: string;
  sessionId?: string;
  parentSpanId?: string;  // For distributed tracing
}

@Injectable()
export class CorrelationService {
  private asyncLocalStorage = new AsyncLocalStorage<CorrelationContext>();

  run<T>(context: CorrelationContext, callback: () => T): T {
    return this.asyncLocalStorage.run(context, callback);
  }

  getContext(): CorrelationContext | undefined {
    return this.asyncLocalStorage.getStore();
  }

  getCorrelationId(): string {
    return this.getContext()?.correlationId || 'unknown';
  }

  createContext(headers: Record<string, any>, user?: any): CorrelationContext {
    return {
      correlationId: headers['x-correlation-id'] || uuidv4(),
      requestId: headers['x-request-id'] || uuidv4(),
      userId: user?.id,
      sessionId: headers['x-session-id'],
      parentSpanId: headers['x-parent-span-id'],
    };
  }
}

// correlation/correlation-id.middleware.ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';
import { CorrelationService } from './correlation.service';
import { logger } from '../logger/logger';

@Injectable()
export class CorrelationIdMiddleware implements NestMiddleware {
  constructor(private readonly correlationService: CorrelationService) {}

  use(req: Request, res: Response, next: NextFunction) {
    const context = this.correlationService.createContext(req.headers, req.user);

    // Add to response headers
    res.setHeader('X-Correlation-ID', context.correlationId);
    res.setHeader('X-Request-ID', context.requestId);

    // Run in correlation context
    this.correlationService.run(context, () => {
      logger.info('Incoming request', {
        method: req.method,
        url: req.url,
      });

      const start = Date.now();

      res.on('finish', () => {
        logger.info('Request completed', {
          method: req.method,
          url: req.url,
          statusCode: res.statusCode,
          duration: `${Date.now() - start}ms`,
        });
      });

      next();
    });
  }
}

// logger/logger.ts
import * as winston from 'winston';
import { CorrelationService } from '../correlation/correlation.service';

const correlationService = new CorrelationService();

const correlationFormat = winston.format((info) => {
  const context = correlationService.getContext();
  if (context) {
    return { ...info, ...context };
  }
  return info;
})();

export const logger = winston.createLogger({
  format: winston.format.combine(
    winston.format.timestamp(),
    correlationFormat,
    winston.format.errors({ stack: true }),
    winston.format.json(),
  ),
  defaultMeta: {
    service: 'my-nestjs-app',
    version: '1.0.0',
  },
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: 'logs/app.log' }),
  ],
});

// app.module.ts
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { CorrelationModule } from './correlation/correlation.module';
import { CorrelationIdMiddleware } from './correlation/correlation-id.middleware';

@Module({
  imports: [CorrelationModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(CorrelationIdMiddleware).forRoutes('*');
  }
}
```

---

### **Method 9: Distributed Tracing (OpenTelemetry)**

```typescript
// Install: npm install @opentelemetry/sdk-node @opentelemetry/auto-instrumentations-node

// tracing.ts
import { NodeSDK } from '@opentelemetry/sdk-node';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';
import { JaegerExporter } from '@opentelemetry/exporter-jaeger';

const sdk = new NodeSDK({
  traceExporter: new JaegerExporter({
    endpoint: 'http://localhost:14268/api/traces',
  }),
  instrumentations: [getNodeAutoInstrumentations()],
});

sdk.start();

// Automatically generates trace IDs and span IDs
// Propagates through HTTP calls, database queries, etc.

// Usage with logger
import { trace, context } from '@opentelemetry/api';
import { logger } from './logger/logger';

export function logWithTrace(message: string, metadata?: any) {
  const span = trace.getSpan(context.active());
  const traceId = span?.spanContext().traceId;
  const spanId = span?.spanContext().spanId;

  logger.info(message, {
    ...metadata,
    traceId,  // ✅ OpenTelemetry trace ID
    spanId,
  });
}
```

---

### **Querying Logs by Correlation ID:**

```bash
# Elasticsearch
GET /logs/_search
{
  "query": {
    "term": { "correlationId": "corr-123-456" }
  },
  "sort": [{ "timestamp": "asc" }]
}

# CloudWatch Insights
fields @timestamp, message, service
| filter correlationId = "corr-123-456"
| sort @timestamp asc

# Splunk
source="app.log" correlationId="corr-123-456" | sort _time

# jq (command line)
cat logs/app.log | jq 'select(.correlationId=="corr-123-456")'

# grep
grep 'correlationId":"corr-123-456' logs/app.log
```

---

### **Correlation ID Flow Example:**

```
Client Request
    ↓
    [correlationId: corr-123]
    ↓
API Gateway
    ↓ (propagate in header)
    [X-Correlation-ID: corr-123]
    ↓
User Service
    ↓ (log with correlationId)
    { correlationId: "corr-123", message: "Fetching user" }
    ↓
    [X-Correlation-ID: corr-123]
    ↓
Auth Service
    ↓ (log with correlationId)
    { correlationId: "corr-123", message: "Validating token" }
    ↓
    [X-Correlation-ID: corr-123]
    ↓
Database
    ↓ (log with correlationId)
    { correlationId: "corr-123", message: "Query executed" }
    ↓
Response to Client
    [X-Correlation-ID: corr-123]
```

---

### **Best Practices:**

```typescript
// ✅ GOOD - Generate at entry point
const correlationId = req.headers['x-correlation-id'] || uuidv4();

// ✅ GOOD - Propagate to all downstream services
axios.get(url, {
  headers: { 'X-Correlation-ID': correlationId },
});

// ✅ GOOD - Include in all logs
logger.info('Event', { correlationId, ...data });

// ✅ GOOD - Return in response headers
res.setHeader('X-Correlation-ID', correlationId);

// ✅ GOOD - Use standard header names
'X-Correlation-ID'  // Most common
'X-Request-ID'      // Alternative
'traceparent'       // W3C Trace Context standard

// ✅ GOOD - Store in AsyncLocalStorage for automatic inclusion
CorrelationContextManager.run({ correlationId }, () => {
  // All logs automatically include correlationId
});

// ❌ BAD - Different IDs per service
// Service A: correlationId: "abc"
// Service B: correlationId: "xyz"  // Can't trace!

// ❌ BAD - Not propagating to external services
await axios.get(url);  // Missing X-Correlation-ID header

// ❌ BAD - Inconsistent header names
'X-Correlation-ID' vs 'X-CorrelationId' vs 'correlationId'
```

**Key Takeaway:** Implement correlation IDs using **middleware** to generate/extract IDs, **AsyncLocalStorage** for automatic context propagation, and **HTTP headers** (X-Correlation-ID) to propagate IDs across distributed services, enabling end-to-end request tracing.

</details>

## Application Monitoring

<details>
<summary><strong>26. What is Application Performance Monitoring (APM)?</strong></summary>

**Answer:**

Application Performance Monitoring (APM) is the practice of **monitoring and managing** the performance and availability of software applications. It involves tracking **response times**, **error rates**, **throughput**, **resource utilization**, and **user experience** to identify bottlenecks, diagnose issues, and optimize application performance.

---

### **Key Concepts:**

#### **1. What APM Tracks:**

```typescript
// APM monitors:

✅ Response Time / Latency
   - API endpoint response times
   - Database query duration
   - External service call latency
   - Page load times

✅ Throughput
   - Requests per second (RPS)
   - Transactions per minute (TPM)
   - Data processed per hour

✅ Error Rate
   - HTTP error codes (4xx, 5xx)
   - Exception counts
   - Failed transactions
   - Error percentage

✅ Resource Utilization
   - CPU usage
   - Memory consumption
   - Disk I/O
   - Network bandwidth

✅ User Experience
   - Apdex score (Application Performance Index)
   - User satisfaction metrics
   - Geographic performance

✅ Dependencies
   - Database performance
   - External API calls
   - Cache hit rates
   - Message queue processing
```

---

#### **2. APM Architecture:**

```
┌─────────────────────────────────────────────────────┐
│              Your NestJS Application                │
│  ┌──────────────────────────────────────────────┐  │
│  │         APM Agent (Instrumentation)          │  │
│  │  • Automatic transaction tracking             │  │
│  │  • Error capture                              │  │
│  │  • Performance metrics collection             │  │
│  │  • Distributed tracing                        │  │
│  └──────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
                      ↓
            Sends metrics & traces
                      ↓
┌─────────────────────────────────────────────────────┐
│              APM Server / Platform                  │
│  • New Relic                                        │
│  • Datadog                                          │
│  • Elastic APM                                      │
│  • Dynatrace                                        │
│  • AppDynamics                                      │
└─────────────────────────────────────────────────────┘
                      ↓
          Data aggregation & analysis
                      ↓
┌─────────────────────────────────────────────────────┐
│              APM Dashboard / UI                     │
│  • Performance charts                               │
│  • Error tracking                                   │
│  • Distributed traces                               │
│  • Alerting                                         │
└─────────────────────────────────────────────────────┘
```

---

### **Method 1: New Relic APM**

```typescript
// Install: npm install newrelic

// newrelic.js (at project root)
'use strict';

exports.config = {
  app_name: ['My NestJS App'],
  license_key: process.env.NEW_RELIC_LICENSE_KEY,
  logging: {
    level: 'info',
  },
  distributed_tracing: {
    enabled: true,
  },
  transaction_tracer: {
    enabled: true,
    transaction_threshold: 'apdex_f',  // Slow transactions
    record_sql: 'obfuscated',
  },
  error_collector: {
    enabled: true,
    capture_events: true,
  },
};

// main.ts - Load newrelic FIRST (before any other imports)
require('newrelic');  // ✅ Must be first!

import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
  console.log('New Relic APM enabled');
}
bootstrap();

// Custom instrumentation
import * as newrelic from 'newrelic';

export class OrderService {
  async createOrder(orderData: any) {
    // Start custom transaction
    return newrelic.startBackgroundTransaction('createOrder', async () => {
      const order = await this.orderRepository.save(orderData);
      
      // Add custom attributes
      newrelic.addCustomAttributes({
        orderId: order.id,
        total: order.total,
        userId: order.userId,
      });
      
      return order;
    });
  }

  async processPayment(orderId: string) {
    // Record custom events
    newrelic.recordCustomEvent('PaymentProcessed', {
      orderId,
      timestamp: Date.now(),
    });
  }
}
```

---

### **Method 2: Datadog APM**

```typescript
// Install: npm install dd-trace

// tracer.ts
import tracer from 'dd-trace';

tracer.init({
  service: 'my-nestjs-app',
  env: process.env.NODE_ENV,
  version: '1.0.0',
  logInjection: true,  // Inject trace IDs into logs
  runtimeMetrics: true,  // Collect runtime metrics
  profiling: true,  // Enable profiling
});

export default tracer;

// main.ts - Import tracer FIRST
import './tracer';  // ✅ Must be first!

import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
  console.log('Datadog APM enabled');
}
bootstrap();

// Custom instrumentation
import tracer from 'dd-trace';

export class UserService {
  async getUser(id: string) {
    const span = tracer.scope().active();
    
    // Add tags to current span
    span?.setTag('user.id', id);
    span?.setTag('operation', 'getUser');

    const user = await this.userRepository.findOne(id);
    
    span?.setTag('user.found', !!user);
    
    return user;
  }

  async createUser(userData: any) {
    // Create custom span
    const span = tracer.startSpan('user.create');
    
    try {
      span.setTag('user.email', userData.email);
      
      const user = await this.userRepository.save(userData);
      
      span.setTag('user.id', user.id);
      
      return user;
    } catch (error) {
      span.setTag('error', true);
      span.log({ event: 'error', message: error.message });
      throw error;
    } finally {
      span.finish();
    }
  }
}
```

---

### **Method 3: Elastic APM**

```typescript
// Install: npm install elastic-apm-node

// apm.ts
import apm from 'elastic-apm-node';

apm.start({
  serviceName: 'my-nestjs-app',
  serverUrl: process.env.ELASTIC_APM_SERVER_URL,
  secretToken: process.env.ELASTIC_APM_SECRET_TOKEN,
  environment: process.env.NODE_ENV,
  logLevel: 'info',
  captureBody: 'all',  // Capture request/response bodies
  captureHeaders: true,
});

export default apm;

// main.ts - Import apm FIRST
import './apm';  // ✅ Must be first!

import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
  console.log('Elastic APM enabled');
}
bootstrap();

// Custom instrumentation
import apm from './apm';

export class OrderService {
  async createOrder(orderData: any) {
    // Start custom transaction
    const transaction = apm.startTransaction('Create Order', 'custom');
    
    try {
      // Add labels (tags)
      transaction?.addLabels({
        orderId: orderData.id,
        userId: orderData.userId,
      });

      const order = await this.orderRepository.save(orderData);
      
      // Set result
      transaction?.setOutcome('success');
      transaction?.result = 'success';
      
      return order;
    } catch (error) {
      // Capture error
      apm.captureError(error);
      transaction?.setOutcome('failure');
      throw error;
    } finally {
      transaction?.end();
    }
  }

  async processPayment(orderId: string) {
    // Create custom span
    const span = apm.startSpan('Payment Processing');
    
    try {
      const result = await this.paymentService.charge(orderId);
      span?.setOutcome('success');
      return result;
    } catch (error) {
      span?.setOutcome('failure');
      throw error;
    } finally {
      span?.end();
    }
  }
}
```

---

### **Method 4: Custom APM Metrics**

```typescript
// metrics/metrics.service.ts
import { Injectable } from '@nestjs/common';
import * as client from 'prom-client';

@Injectable()
export class MetricsService {
  private httpRequestDuration: client.Histogram;
  private httpRequestTotal: client.Counter;
  private errorTotal: client.Counter;

  constructor() {
    // Response time histogram
    this.httpRequestDuration = new client.Histogram({
      name: 'http_request_duration_ms',
      help: 'Duration of HTTP requests in milliseconds',
      labelNames: ['method', 'route', 'status_code'],
      buckets: [10, 50, 100, 200, 500, 1000, 2000, 5000],
    });

    // Request counter
    this.httpRequestTotal = new client.Counter({
      name: 'http_requests_total',
      help: 'Total number of HTTP requests',
      labelNames: ['method', 'route', 'status_code'],
    });

    // Error counter
    this.errorTotal = new client.Counter({
      name: 'errors_total',
      help: 'Total number of errors',
      labelNames: ['type', 'route'],
    });
  }

  recordHttpRequest(method: string, route: string, statusCode: number, duration: number) {
    this.httpRequestDuration.observe(
      { method, route, status_code: statusCode },
      duration,
    );
    this.httpRequestTotal.inc({ method, route, status_code: statusCode });
  }

  recordError(type: string, route: string) {
    this.errorTotal.inc({ type, route });
  }

  getMetrics(): string {
    return client.register.metrics();
  }
}

// interceptors/metrics.interceptor.ts
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';
import { MetricsService } from '../metrics/metrics.service';

@Injectable()
export class MetricsInterceptor implements NestInterceptor {
  constructor(private readonly metricsService: MetricsService) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const response = context.switchToHttp().getResponse();
    const start = Date.now();

    return next.handle().pipe(
      tap({
        next: () => {
          const duration = Date.now() - start;
          this.metricsService.recordHttpRequest(
            request.method,
            request.route?.path || request.url,
            response.statusCode,
            duration,
          );
        },
        error: (error) => {
          const duration = Date.now() - start;
          this.metricsService.recordHttpRequest(
            request.method,
            request.route?.path || request.url,
            error.status || 500,
            duration,
          );
          this.metricsService.recordError(
            error.constructor.name,
            request.route?.path || request.url,
          );
        },
      }),
    );
  }
}

// metrics.controller.ts
import { Controller, Get } from '@nestjs/common';
import { MetricsService } from './metrics/metrics.service';

@Controller('metrics')
export class MetricsController {
  constructor(private readonly metricsService: MetricsService) {}

  @Get()
  getMetrics(): string {
    return this.metricsService.getMetrics();
  }
}
```

---

### **Key APM Metrics (The Golden Signals):**

```typescript
// 1. Latency (Response Time)
//    How long requests take
average_response_time = total_response_time / request_count

// 2. Traffic (Throughput)
//    Requests per second
requests_per_second = request_count / time_window

// 3. Errors (Error Rate)
//    Percentage of failed requests
error_rate = (error_count / request_count) * 100

// 4. Saturation (Resource Utilization)
//    System resource usage
cpu_usage_percent = (cpu_used / cpu_total) * 100
memory_usage_percent = (memory_used / memory_total) * 100

// RED Method (Rate, Errors, Duration)
Rate:     requests_per_second
Errors:   error_rate_percent
Duration: average_response_time

// USE Method (Utilization, Saturation, Errors)
Utilization: cpu_usage, memory_usage
Saturation:  queue_depth, thread_pool_usage
Errors:      error_count
```

---

### **APM Dashboard Example:**

```
┌─────────────────────────────────────────────────────┐
│           Application Performance Overview          │
├─────────────────────────────────────────────────────┤
│  Throughput: 1,234 requests/min        ↑ 12%       │
│  Avg Response Time: 145ms              ↓ 8%        │
│  Error Rate: 0.23%                     ↑ 0.05%     │
│  Apdex Score: 0.95                     → Same      │
├─────────────────────────────────────────────────────┤
│  Slowest Endpoints:                                 │
│  • POST /api/orders/create        2,345ms  (p99)   │
│  • GET  /api/users/:id/profile   1,123ms  (p99)   │
│  • POST /api/payments/process      987ms  (p99)   │
├─────────────────────────────────────────────────────┤
│  Most Errors:                                       │
│  • GET  /api/products/:id         23 errors        │
│  • POST /api/checkout             12 errors        │
├─────────────────────────────────────────────────────┤
│  Service Map:                                       │
│    API Gateway → User Service → Database            │
│              ↘ Auth Service ↗                       │
│              ↘ Payment Service                      │
└─────────────────────────────────────────────────────┘
```

---

### **Benefits of APM:**

```typescript
✅ Real-time Performance Monitoring
   - Identify slow endpoints immediately
   - Detect performance degradation

✅ Error Tracking
   - Capture exceptions with stack traces
   - Track error rates and patterns

✅ Distributed Tracing
   - See request flow across microservices
   - Identify bottlenecks in service chains

✅ Resource Optimization
   - Identify memory leaks
   - Optimize database queries
   - Reduce CPU usage

✅ User Experience Insights
   - Apdex scores
   - Geographic performance
   - Device-specific issues

✅ Proactive Alerting
   - Alert on high error rates
   - Notify on slow response times
   - Detect anomalies

✅ Root Cause Analysis
   - Detailed transaction traces
   - Database query analysis
   - External service performance
```

---

### **Best Practices:**

```typescript
// ✅ GOOD - Load APM agent first
import './apm';  // Before everything else
import { NestFactory } from '@nestjs/core';

// ✅ GOOD - Add custom attributes
apm.addCustomAttributes({
  userId: user.id,
  orderId: order.id,
});

// ✅ GOOD - Track custom transactions
const transaction = apm.startTransaction('Background Job');
// ... work ...
transaction.end();

// ✅ GOOD - Monitor external dependencies
const span = apm.startSpan('External API Call');
await axios.get(url);
span?.end();

// ✅ GOOD - Set meaningful transaction names
apm.setTransactionName('POST /api/orders/:id/checkout');

// ❌ BAD - Load APM agent after other imports
import { NestFactory } from '@nestjs/core';
import './apm';  // Too late!

// ❌ BAD - No custom instrumentation for critical paths
// APM may not capture important business logic

// ❌ BAD - Ignoring APM alerts
// High error rate for 2 hours → no investigation
```

**Key Takeaway:** APM monitors application **performance** (response times, throughput), **errors**, and **resource utilization** using tools like **New Relic**, **Datadog**, or **Elastic APM**, providing insights through **distributed tracing**, **metrics**, and **alerts** to optimize application health.

</details>

<details>
<summary><strong>27. What monitoring tools can you use (New Relic, Datadog, Sentry)?</strong></summary>

**Answer:**

Modern applications use multiple monitoring tools for different purposes: **APM** (New Relic, Datadog), **error tracking** (Sentry), **infrastructure monitoring** (Datadog, Prometheus), and **log aggregation** (ELK, Splunk). Each tool specializes in specific monitoring aspects.

---

### **Comparison Table:**

| Tool | Primary Purpose | Best For | Pricing | NestJS Support |
|------|----------------|----------|---------|----------------|
| **New Relic** | APM, Full-stack monitoring | Enterprise, Full observability | $99-$349/host/month | ✅ Excellent |
| **Datadog** | Infrastructure & APM | Cloud-native, Multi-cloud | $15-$23/host/month | ✅ Excellent |
| **Sentry** | Error tracking & monitoring | Error management, Performance | Free tier, $26+/month | ✅ Excellent |
| **Elastic APM** | APM, Logs, Traces | Open-source, Self-hosted | Free (self-hosted) | ✅ Good |
| **Prometheus** | Metrics collection | Time-series metrics | Free (open-source) | ✅ Good |
| **Grafana** | Visualization | Dashboards, Alerting | Free (open-source) | ✅ Via Prometheus |
| **Dynatrace** | Full-stack monitoring | Enterprise, AI-powered | $69+/host/month | ✅ Good |
| **AppDynamics** | APM, Business monitoring | Enterprise Java apps | Custom pricing | ✅ Limited |

---

### **Tool 1: New Relic**

```typescript
// Install: npm install newrelic

// newrelic.js
exports.config = {
  app_name: ['My NestJS App'],
  license_key: process.env.NEW_RELIC_LICENSE_KEY,
  distributed_tracing: { enabled: true },
  logging: { level: 'info' },
};

// main.ts
require('newrelic');  // Must be first!
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();

// Features:
✅ Automatic instrumentation (HTTP, DB, external calls)
✅ Real User Monitoring (RUM)
✅ Infrastructure monitoring
✅ Custom dashboards
✅ Alert policies
✅ Deployment tracking
✅ Service maps
✅ Transaction traces

// Use cases:
• Enterprise applications
• Full-stack observability
• Business intelligence
• SLA monitoring
```

---

### **Tool 2: Datadog**

```typescript
// Install: npm install dd-trace

// tracer.ts
import tracer from 'dd-trace';

tracer.init({
  service: 'my-nestjs-app',
  env: process.env.NODE_ENV,
  version: '1.0.0',
  logInjection: true,
  runtimeMetrics: true,
  profiling: true,
});

export default tracer;

// main.ts
import './tracer';  // Must be first!
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();

// Features:
✅ APM (Application Performance Monitoring)
✅ Infrastructure monitoring (servers, containers, K8s)
✅ Log management
✅ Real-time metrics
✅ Distributed tracing
✅ Security monitoring
✅ Synthetic monitoring
✅ Network performance monitoring

// Use cases:
• Cloud-native applications
• Microservices architecture
• DevOps teams
• Multi-cloud environments
• Kubernetes monitoring
```

---

### **Tool 3: Sentry**

```typescript
// Install: npm install @sentry/node @sentry/profiling-node

// sentry.ts
import * as Sentry from '@sentry/node';
import { ProfilingIntegration } from '@sentry/profiling-node';

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 1.0,  // 100% of transactions
  profilesSampleRate: 1.0,
  integrations: [
    new ProfilingIntegration(),
  ],
});

// main.ts
import './sentry';  // Initialize Sentry
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();

// Features:
✅ Error tracking with stack traces
✅ Performance monitoring
✅ Release tracking
✅ User feedback
✅ Breadcrumbs (events leading to error)
✅ Source map support
✅ Issue grouping
✅ Alerting

// Use cases:
• Error tracking (primary)
• Frontend + Backend monitoring
• Release quality tracking
• User-reported issues
• Performance regression detection
```

---

### **Tool 4: Elastic APM**

```typescript
// Install: npm install elastic-apm-node

// apm.ts
import apm from 'elastic-apm-node';

apm.start({
  serviceName: 'my-nestjs-app',
  serverUrl: process.env.ELASTIC_APM_SERVER_URL,
  secretToken: process.env.ELASTIC_APM_SECRET_TOKEN,
  environment: process.env.NODE_ENV,
});

export default apm;

// main.ts
import './apm';  // Must be first!
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();

// Features:
✅ APM (performance monitoring)
✅ Distributed tracing
✅ Integrates with ELK Stack
✅ Real-time metrics
✅ Service maps
✅ Open-source
✅ Self-hosted option

// Use cases:
• Already using ELK Stack
• Open-source preference
• Self-hosted requirements
• Cost-sensitive projects
```

---

### **Tool 5: Prometheus + Grafana**

```typescript
// Install: npm install prom-client

// metrics/prometheus.service.ts
import { Injectable } from '@nestjs/common';
import * as client from 'prom-client';

@Injectable()
export class PrometheusService {
  private register: client.Registry;
  private httpRequestDuration: client.Histogram;
  private httpRequestTotal: client.Counter;

  constructor() {
    this.register = new client.Registry();
    
    // Default metrics (CPU, memory, etc.)
    client.collectDefaultMetrics({ register: this.register });

    // Custom metrics
    this.httpRequestDuration = new client.Histogram({
      name: 'http_request_duration_seconds',
      help: 'Duration of HTTP requests in seconds',
      labelNames: ['method', 'route', 'status_code'],
      buckets: [0.01, 0.05, 0.1, 0.5, 1, 2, 5],
      registers: [this.register],
    });

    this.httpRequestTotal = new client.Counter({
      name: 'http_requests_total',
      help: 'Total number of HTTP requests',
      labelNames: ['method', 'route', 'status_code'],
      registers: [this.register],
    });
  }

  recordRequest(method: string, route: string, statusCode: number, duration: number) {
    this.httpRequestDuration.observe(
      { method, route, status_code: statusCode },
      duration / 1000,  // Convert to seconds
    );
    this.httpRequestTotal.inc({ method, route, status_code: statusCode });
  }

  getMetrics(): Promise<string> {
    return this.register.metrics();
  }
}

// metrics.controller.ts
import { Controller, Get, Header } from '@nestjs/common';
import { PrometheusService } from './prometheus.service';

@Controller('metrics')
export class MetricsController {
  constructor(private readonly prometheus: PrometheusService) {}

  @Get()
  @Header('Content-Type', 'text/plain')
  async getMetrics(): Promise<string> {
    return this.prometheus.getMetrics();
  }
}

// Features:
✅ Time-series metrics database
✅ Powerful query language (PromQL)
✅ Alerting (Alertmanager)
✅ Service discovery
✅ Open-source
✅ Grafana integration for dashboards

// Use cases:
• Kubernetes monitoring
• Infrastructure metrics
• Custom metrics
• Cost-free solution
• Open-source preference
```

---

### **Multi-Tool Strategy (Recommended):**

```typescript
// Combine tools for comprehensive monitoring

// 1. Sentry - Error tracking
import * as Sentry from '@sentry/node';
Sentry.init({ dsn: process.env.SENTRY_DSN });

// 2. Datadog - APM & Infrastructure
import tracer from 'dd-trace';
tracer.init({ service: 'my-app' });

// 3. ELK Stack - Log aggregation
import * as winston from 'winston';
import { ElasticsearchTransport } from 'winston-elasticsearch';

const logger = winston.createLogger({
  transports: [
    new ElasticsearchTransport({
      clientOpts: { node: 'http://localhost:9200' },
    }),
  ],
});

// 4. Prometheus - Custom metrics
import * as client from 'prom-client';
const register = new client.Registry();
client.collectDefaultMetrics({ register });

// Why multi-tool?
✅ Sentry: Best-in-class error tracking
✅ Datadog: Infrastructure + APM
✅ ELK: Log aggregation and analysis
✅ Prometheus: Custom business metrics

// Each tool specializes in different aspects
```

---

### **Decision Matrix:**

```typescript
// Choose based on your needs:

if (budget === 'unlimited' && team === 'large') {
  return 'New Relic';  // Full-stack, enterprise features
}

if (kubernetes || microservices || multiCloud) {
  return 'Datadog';  // Best for cloud-native
}

if (primaryConcern === 'errors') {
  return 'Sentry';  // Best error tracking
}

if (openSource && selfHosted) {
  return 'Elastic APM + Prometheus + Grafana';
}

if (startup && budget === 'limited') {
  return 'Sentry (free tier) + Prometheus (free)';
}

if (compliance || dataPrivacy) {
  return 'Self-hosted: Elastic APM + Prometheus';
}
```

---

### **Production Example: Multi-Tool Setup**

```typescript
// config/monitoring.config.ts
export const monitoringConfig = {
  sentry: {
    enabled: process.env.NODE_ENV === 'production',
    dsn: process.env.SENTRY_DSN,
    environment: process.env.NODE_ENV,
    tracesSampleRate: 0.1,  // 10% of transactions
  },
  datadog: {
    enabled: process.env.DATADOG_ENABLED === 'true',
    service: 'my-nestjs-app',
    env: process.env.NODE_ENV,
  },
  prometheus: {
    enabled: true,
    endpoint: '/metrics',
  },
  newRelic: {
    enabled: process.env.NEW_RELIC_ENABLED === 'true',
    licenseKey: process.env.NEW_RELIC_LICENSE_KEY,
  },
};

// monitoring/monitoring.module.ts
import { Module, Global, DynamicModule } from '@nestjs/common';
import { monitoringConfig } from '../config/monitoring.config';
import * as Sentry from '@sentry/node';
import tracer from 'dd-trace';

@Global()
@Module({})
export class MonitoringModule {
  static forRoot(): DynamicModule {
    // Initialize Sentry
    if (monitoringConfig.sentry.enabled) {
      Sentry.init({
        dsn: monitoringConfig.sentry.dsn,
        environment: monitoringConfig.sentry.environment,
        tracesSampleRate: monitoringConfig.sentry.tracesSampleRate,
      });
      console.log('Sentry initialized');
    }

    // Initialize Datadog
    if (monitoringConfig.datadog.enabled) {
      tracer.init({
        service: monitoringConfig.datadog.service,
        env: monitoringConfig.datadog.env,
      });
      console.log('Datadog initialized');
    }

    // Initialize New Relic
    if (monitoringConfig.newRelic.enabled) {
      require('newrelic');
      console.log('New Relic initialized');
    }

    return {
      module: MonitoringModule,
      providers: [],
      exports: [],
    };
  }
}

// app.module.ts
import { Module } from '@nestjs/common';
import { MonitoringModule } from './monitoring/monitoring.module';

@Module({
  imports: [MonitoringModule.forRoot()],
})
export class AppModule {}
```

---

### **Cost Comparison (Estimated Monthly):**

```
// For a typical 10-host deployment:

New Relic:      $990 - $3,490/month
Datadog:        $150 - $230/month
Sentry:         Free - $80/month (varies by events)
Dynatrace:      $690+/month
Elastic APM:    Free (self-hosted) or ~$95/month (Elastic Cloud)
Prometheus:     Free (self-hosted)
Grafana:        Free (self-hosted) or $49/month (Grafana Cloud)

// Recommended starter combo:
Sentry (free tier) + Prometheus (free) + Grafana (free)
= $0/month

// Recommended production combo:
Datadog ($150) + Sentry ($80)
= $230/month

// Enterprise combo:
New Relic ($3,490)
= $3,490/month
```

---

### **Feature Comparison:**

| Feature | New Relic | Datadog | Sentry | Elastic APM | Prometheus |
|---------|-----------|---------|--------|-------------|------------|
| **APM** | ✅ Excellent | ✅ Excellent | ⚠️ Basic | ✅ Good | ❌ No |
| **Error Tracking** | ✅ Good | ✅ Good | ✅ Excellent | ✅ Good | ❌ No |
| **Infrastructure** | ✅ Excellent | ✅ Excellent | ❌ No | ⚠️ Limited | ✅ Good |
| **Logs** | ✅ Good | ✅ Excellent | ❌ No | ✅ Excellent | ❌ No |
| **Distributed Tracing** | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | ❌ No |
| **Custom Metrics** | ✅ Yes | ✅ Yes | ⚠️ Limited | ✅ Yes | ✅ Excellent |
| **Alerting** | ✅ Excellent | ✅ Excellent | ✅ Good | ✅ Good | ✅ Good |
| **Dashboards** | ✅ Excellent | ✅ Excellent | ✅ Good | ✅ Good | ⚠️ Need Grafana |
| **Self-Hosted** | ❌ No | ❌ No | ✅ Yes | ✅ Yes | ✅ Yes |
| **Open Source** | ❌ No | ❌ No | ✅ Yes | ✅ Yes | ✅ Yes |
| **Free Tier** | ✅ Limited | ✅ Limited | ✅ Generous | ✅ Yes | ✅ Yes |

---

### **Integration Example: All Tools**

```typescript
// main.ts - Initialize all monitoring tools

// 1. New Relic (if enabled)
if (process.env.NEW_RELIC_ENABLED === 'true') {
  require('newrelic');
}

// 2. Datadog (if enabled)
if (process.env.DATADOG_ENABLED === 'true') {
  require('./tracer');
}

// 3. Elastic APM (if enabled)
if (process.env.ELASTIC_APM_ENABLED === 'true') {
  require('./apm');
}

// 4. Sentry (if enabled)
import * as Sentry from '@sentry/node';
if (process.env.SENTRY_ENABLED === 'true') {
  Sentry.init({ dsn: process.env.SENTRY_DSN });
}

import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Prometheus metrics endpoint
  // Available at GET /metrics
  
  await app.listen(3000);
  console.log('Monitoring tools initialized');
}
bootstrap();
```

---

### **Best Practices:**

```typescript
// ✅ GOOD - Environment-based monitoring
const monitoringEnabled = process.env.NODE_ENV === 'production';

// ✅ GOOD - Sample large-scale apps
tracesSampleRate: 0.1,  // 10% of requests

// ✅ GOOD - Use multiple tools for different purposes
Sentry (errors) + Datadog (APM) + Prometheus (metrics)

// ✅ GOOD - Initialize monitoring agents FIRST
require('./monitoring');  // Before any other imports

// ✅ GOOD - Set meaningful service names
service: 'api-gateway'  // Not just 'my-app'

// ✅ GOOD - Tag with environment and version
env: process.env.NODE_ENV,
version: process.env.APP_VERSION,

// ❌ BAD - Initialize monitoring late
import { NestFactory } from '@nestjs/core';
require('newrelic');  // Too late!

// ❌ BAD - 100% sampling in high-traffic apps
tracesSampleRate: 1.0,  // Expensive!

// ❌ BAD - No monitoring in production
if (process.env.NODE_ENV !== 'production') {
  // Only monitor in dev? Wrong!
}
```

**Key Takeaway:** Use **New Relic** or **Datadog** for comprehensive APM, **Sentry** for error tracking, **Prometheus + Grafana** for custom metrics, and **Elastic APM** for open-source solutions. Combine tools based on needs: errors (Sentry), infrastructure (Datadog), metrics (Prometheus).

</details>

<details>
<summary><strong>28. How do you integrate Sentry for error tracking?</strong></summary>

**Answer:**

Sentry is a **real-time error tracking** platform that captures exceptions, tracks releases, monitors performance, and provides detailed stack traces. Integration with NestJS involves installing the SDK, initializing Sentry, and optionally adding filters, breadcrumbs, and custom context.

---

### **Method 1: Basic Sentry Setup**

```typescript
// Install
npm install @sentry/node @sentry/profiling-node

// sentry.config.ts
import * as Sentry from '@sentry/node';
import { ProfilingIntegration } from '@sentry/profiling-node';

export function initSentry() {
  Sentry.init({
    dsn: process.env.SENTRY_DSN,
    environment: process.env.NODE_ENV || 'development',
    
    // Percentage of transactions to trace
    tracesSampleRate: 1.0,  // 100% in dev, reduce in prod
    
    // Percentage of profiles to capture
    profilesSampleRate: 1.0,
    
    integrations: [
      new ProfilingIntegration(),
    ],
  });
}

// main.ts
import { initSentry } from './sentry.config';

// Initialize Sentry FIRST!
initSentry();

import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
  console.log('Application started with Sentry error tracking');
}
bootstrap();

// Test error capture
import * as Sentry from '@sentry/node';

try {
  throw new Error('Test error');
} catch (error) {
  Sentry.captureException(error);
}
```

---

### **Method 2: Global Exception Filter**

```typescript
// filters/sentry-exception.filter.ts
import {
  ExceptionFilter,
  Catch,
  ArgumentsHost,
  HttpException,
  HttpStatus,
} from '@nestjs/common';
import { Request, Response } from 'express';
import * as Sentry from '@sentry/node';

@Catch()
export class SentryExceptionFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    // Determine status code
    const status =
      exception instanceof HttpException
        ? exception.getStatus()
        : HttpStatus.INTERNAL_SERVER_ERROR;

    // Capture exception in Sentry
    Sentry.captureException(exception, {
      contexts: {
        http: {
          method: request.method,
          url: request.url,
          status_code: status,
        },
      },
      user: {
        id: request.user?.id,
        email: request.user?.email,
      },
      tags: {
        endpoint: request.url,
        method: request.method,
      },
      extra: {
        body: request.body,
        query: request.query,
        params: request.params,
      },
    });

    // Send response
    const message =
      exception instanceof HttpException
        ? exception.getResponse()
        : 'Internal server error';

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      message,
    });
  }
}

// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { SentryExceptionFilter } from './filters/sentry-exception.filter';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Apply global filter
  app.useGlobalFilters(new SentryExceptionFilter());
  
  await app.listen(3000);
}
bootstrap();
```

---

### **Method 3: Sentry Interceptor for Performance**

```typescript
// interceptors/sentry-performance.interceptor.ts
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';
import * as Sentry from '@sentry/node';

@Injectable()
export class SentryPerformanceInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { method, url } = request;

    // Start transaction
    const transaction = Sentry.startTransaction({
      op: 'http.server',
      name: `${method} ${url}`,
      data: {
        method,
        url,
      },
    });

    // Set transaction on scope
    Sentry.getCurrentHub().configureScope((scope) => {
      scope.setSpan(transaction);
    });

    const start = Date.now();

    return next.handle().pipe(
      tap({
        next: () => {
          const response = context.switchToHttp().getResponse();
          transaction.setHttpStatus(response.statusCode);
          transaction.setData('duration', Date.now() - start);
          transaction.finish();
        },
        error: (error) => {
          transaction.setHttpStatus(500);
          transaction.setStatus('internal_error');
          transaction.finish();
        },
      }),
    );
  }
}

// Apply to specific controllers
import { Controller, Get, UseInterceptors } from '@nestjs/common';
import { SentryPerformanceInterceptor } from './interceptors/sentry-performance.interceptor';

@Controller('users')
@UseInterceptors(SentryPerformanceInterceptor)
export class UsersController {
  @Get()
  getUsers() {
    // Performance tracked by Sentry
    return this.usersService.findAll();
  }
}
```

---

### **Method 4: Custom Sentry Context**

```typescript
// services/user.service.ts
import { Injectable } from '@nestjs/common';
import * as Sentry from '@sentry/node';

@Injectable()
export class UserService {
  async createUser(userData: any) {
    // Set user context
    Sentry.setUser({
      id: userData.id,
      email: userData.email,
      username: userData.username,
    });

    try {
      const user = await this.userRepository.save(userData);
      
      // Add breadcrumb (event trail)
      Sentry.addBreadcrumb({
        category: 'user',
        message: 'User created successfully',
        level: 'info',
        data: {
          userId: user.id,
          email: user.email,
        },
      });
      
      return user;
    } catch (error) {
      // Add tags for filtering
      Sentry.setTag('operation', 'createUser');
      Sentry.setTag('table', 'users');
      
      // Add extra context
      Sentry.setExtra('userData', userData);
      Sentry.setExtra('timestamp', new Date().toISOString());
      
      // Capture exception
      Sentry.captureException(error);
      
      throw error;
    }
  }

  async processOrder(orderId: string) {
    // Set context for this operation
    Sentry.setContext('order', {
      orderId,
      timestamp: Date.now(),
    });

    // Add breadcrumbs to track flow
    Sentry.addBreadcrumb({
      message: 'Starting order processing',
      data: { orderId },
    });

    try {
      await this.validateOrder(orderId);
      Sentry.addBreadcrumb({ message: 'Order validated' });

      await this.processPayment(orderId);
      Sentry.addBreadcrumb({ message: 'Payment processed' });

      await this.fulfillOrder(orderId);
      Sentry.addBreadcrumb({ message: 'Order fulfilled' });

      return { success: true };
    } catch (error) {
      // Sentry will show breadcrumb trail leading to error
      Sentry.captureException(error);
      throw error;
    }
  }
}
```

---

### **Method 5: Release Tracking**

```typescript
// sentry.config.ts
import * as Sentry from '@sentry/node';

export function initSentry() {
  Sentry.init({
    dsn: process.env.SENTRY_DSN,
    environment: process.env.NODE_ENV,
    
    // Track releases
    release: process.env.APP_VERSION || '1.0.0',
    
    // Attach commits (requires Sentry CLI setup)
    // This enables:
    // - Suspect commits
    // - Suggested assignees
    // - Release health tracking
    
    tracesSampleRate: 1.0,
  });
}

// Deploy script (deploy.sh)
#!/bin/bash

# Create Sentry release
sentry-cli releases new "$APP_VERSION"

# Associate commits with release
sentry-cli releases set-commits "$APP_VERSION" --auto

# Deploy application
npm run build
npm run start:prod

# Finalize release
sentry-cli releases finalize "$APP_VERSION"

# Create deploy
sentry-cli releases deploys "$APP_VERSION" new -e production
```

---

### **Method 6: Filtering Errors**

```typescript
// sentry.config.ts
import * as Sentry from '@sentry/node';

export function initSentry() {
  Sentry.init({
    dsn: process.env.SENTRY_DSN,
    environment: process.env.NODE_ENV,
    
    // Filter before sending to Sentry
    beforeSend(event, hint) {
      const error = hint.originalException;
      
      // Don't send 404 errors
      if (event.exception?.values?.[0]?.type === 'NotFoundException') {
        return null;
      }
      
      // Don't send validation errors
      if (event.exception?.values?.[0]?.type === 'ValidationError') {
        return null;
      }
      
      // Filter by HTTP status
      if (event.contexts?.http?.status_code === 404) {
        return null;
      }
      
      // Sanitize sensitive data
      if (event.request?.data) {
        delete event.request.data.password;
        delete event.request.data.creditCard;
      }
      
      return event;
    },
    
    // Ignore specific errors
    ignoreErrors: [
      'Non-Error exception captured',
      'NotFoundException',
      'ValidationError',
    ],
    
    // Sample rate (0.0 to 1.0)
    tracesSampleRate: process.env.NODE_ENV === 'production' ? 0.1 : 1.0,
  });
}
```

---

### **Method 7: Sentry Cron Monitoring**

```typescript
// jobs/backup.job.ts
import { Injectable } from '@nestjs/common';
import { Cron } from '@nestjs/schedule';
import * as Sentry from '@sentry/node';

@Injectable()
export class BackupJob {
  @Cron('0 2 * * *')  // Every day at 2 AM
  async handleBackup() {
    // Create check-in
    const checkInId = Sentry.captureCheckIn({
      monitorSlug: 'daily-backup',
      status: 'in_progress',
    });

    try {
      await this.performBackup();
      
      // Mark as successful
      Sentry.captureCheckIn({
        checkInId,
        monitorSlug: 'daily-backup',
        status: 'ok',
      });
    } catch (error) {
      // Mark as failed
      Sentry.captureCheckIn({
        checkInId,
        monitorSlug: 'daily-backup',
        status: 'error',
      });
      
      Sentry.captureException(error);
    }
  }
}
```

---

### **Method 8: User Feedback**

```typescript
// controllers/feedback.controller.ts
import { Controller, Post, Body } from '@nestjs/common';
import * as Sentry from '@sentry/node';

@Controller('feedback')
export class FeedbackController {
  @Post()
  submitFeedback(@Body() feedbackData: any) {
    const eventId = Sentry.lastEventId();
    
    // Send user feedback to Sentry
    Sentry.captureUserFeedback({
      event_id: eventId,
      name: feedbackData.name,
      email: feedbackData.email,
      comments: feedbackData.comments,
    });
    
    return { success: true };
  }
}

// Frontend example (when error occurs)
fetch('/api/feedback', {
  method: 'POST',
  body: JSON.stringify({
    name: 'John Doe',
    email: 'john@example.com',
    comments: 'The checkout page crashed when I tried to pay',
  }),
});
```

---

### **Method 9: Source Maps**

```typescript
// Install Sentry CLI
npm install @sentry/cli --save-dev

// .sentryclirc
[defaults]
url=https://sentry.io/
org=your-org
project=your-project

[auth]
token=your-auth-token

// package.json
{
  "scripts": {
    "build": "nest build",
    "build:prod": "nest build && npm run sentry:sourcemaps",
    "sentry:sourcemaps": "sentry-cli sourcemaps upload --release=$APP_VERSION dist/"
  }
}

// This enables:
// ✅ Original source code in stack traces
// ✅ Exact line numbers
// ✅ Variable names (not minified)
```

---

### **Method 10: Complete Production Setup**

```typescript
// config/sentry.config.ts
import * as Sentry from '@sentry/node';
import { ProfilingIntegration } from '@sentry/profiling-node';

export function initSentry() {
  const isProd = process.env.NODE_ENV === 'production';
  
  Sentry.init({
    dsn: process.env.SENTRY_DSN,
    environment: process.env.NODE_ENV,
    release: process.env.APP_VERSION,
    
    // Sample rates (lower in production to reduce costs)
    tracesSampleRate: isProd ? 0.1 : 1.0,  // 10% in prod
    profilesSampleRate: isProd ? 0.1 : 1.0,
    
    integrations: [
      new ProfilingIntegration(),
    ],
    
    // Filter errors
    beforeSend(event, hint) {
      // Don't send client errors (4xx)
      const statusCode = event.contexts?.http?.status_code;
      if (statusCode && statusCode >= 400 && statusCode < 500) {
        return null;
      }
      
      // Sanitize sensitive data
      if (event.request?.data) {
        delete event.request.data.password;
        delete event.request.data.token;
        delete event.request.data.creditCard;
      }
      
      if (event.request?.cookies) {
        delete event.request.cookies.sessionId;
      }
      
      return event;
    },
    
    // Ignore common errors
    ignoreErrors: [
      'NotFoundException',
      'UnauthorizedException',
      'BadRequestException',
    ],
    
    // Debug in development
    debug: !isProd,
  });
}

// filters/all-exceptions.filter.ts
import { ExceptionFilter, Catch, ArgumentsHost } from '@nestjs/common';
import * as Sentry from '@sentry/node';

@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const request = ctx.getRequest();
    const response = ctx.getResponse();

    // Capture in Sentry with context
    Sentry.withScope((scope) => {
      // Set user
      if (request.user) {
        scope.setUser({
          id: request.user.id,
          email: request.user.email,
          username: request.user.username,
        });
      }
      
      // Set tags
      scope.setTag('endpoint', request.url);
      scope.setTag('method', request.method);
      
      // Set context
      scope.setContext('http', {
        method: request.method,
        url: request.url,
        headers: request.headers,
      });
      
      scope.setContext('request', {
        body: request.body,
        query: request.query,
        params: request.params,
      });
      
      // Capture
      Sentry.captureException(exception);
    });

    // Send response
    const status = exception instanceof HttpException ? exception.getStatus() : 500;
    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
    });
  }
}

// main.ts
import { initSentry } from './config/sentry.config';
initSentry();

import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { AllExceptionsFilter } from './filters/all-exceptions.filter';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalFilters(new AllExceptionsFilter());
  await app.listen(3000);
}
bootstrap();
```

---

### **Sentry Dashboard Features:**

```
✅ Issues:
   - Grouped errors with stack traces
   - Frequency and impact
   - First/last seen timestamps
   - Affected users count

✅ Performance:
   - Transaction traces
   - Slow endpoints identification
   - Database query performance
   - External service latency

✅ Releases:
   - New errors per release
   - Regression detection
   - Commit tracking
   - Deploy health

✅ Alerts:
   - Spike in errors
   - New issue detection
   - Performance degradation
   - Custom metric thresholds

✅ User Feedback:
   - Reports from users
   - Context around errors
```

---

### **Best Practices:**

```typescript
// ✅ GOOD - Initialize Sentry first
initSentry();
import { NestFactory } from '@nestjs/core';

// ✅ GOOD - Add context to errors
Sentry.setUser({ id: user.id, email: user.email });
Sentry.setTag('operation', 'checkout');
Sentry.captureException(error);

// ✅ GOOD - Use breadcrumbs for tracing
Sentry.addBreadcrumb({ message: 'Payment started' });

// ✅ GOOD - Filter sensitive data
beforeSend(event) {
  delete event.request?.data?.password;
  return event;
}

// ✅ GOOD - Reduce sample rate in production
tracesSampleRate: isProd ? 0.1 : 1.0,

// ✅ GOOD - Track releases
release: process.env.APP_VERSION,

// ❌ BAD - Initialize Sentry late
import { NestFactory } from '@nestjs/core';
initSentry();  // Too late!

// ❌ BAD - Send all errors (including validation)
// This creates noise and increases costs

// ❌ BAD - No user context
Sentry.captureException(error);  // Who was affected?

// ❌ BAD - 100% sampling in high-traffic production
tracesSampleRate: 1.0,  // Expensive!
```

**Key Takeaway:** Integrate Sentry by **initializing first** in main.ts, using **global exception filters** to capture errors, adding **user context** and **breadcrumbs** for debugging, implementing **beforeSend** to filter sensitive data, and tracking **releases** for regression detection.

</details>

<details>
<summary><strong>29. How do you monitor API response times?</strong></summary>

**Answer:**

Monitoring API response times helps identify **performance bottlenecks**, **slow endpoints**, and **degraded user experience**. Use **interceptors**, **middleware**, **APM tools**, or **custom metrics** to track request duration, percentiles (p50, p95, p99), and throughput.

---

### **Method 1: Interceptor with Logging**

```typescript
// interceptors/response-time.interceptor.ts
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
  Logger,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class ResponseTimeInterceptor implements NestInterceptor {
  private readonly logger = new Logger('ResponseTime');

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { method, url } = request;
    const start = Date.now();

    return next.handle().pipe(
      tap({
        next: () => {
          const response = context.switchToHttp().getResponse();
          const duration = Date.now() - start;
          
          this.logger.log(
            JSON.stringify({
              method,
              url,
              statusCode: response.statusCode,
              duration: `${duration}ms`,
              timestamp: new Date().toISOString(),
            }),
          );
          
          // Warn on slow requests
          if (duration > 1000) {
            this.logger.warn(`Slow request: ${method} ${url} took ${duration}ms`);
          }
        },
        error: (error) => {
          const duration = Date.now() - start;
          this.logger.error(
            `Request failed: ${method} ${url} after ${duration}ms`,
            error.stack,
          );
        },
      }),
    );
  }
}

// app.module.ts
import { Module } from '@nestjs/common';
import { APP_INTERCEPTOR } from '@nestjs/core';
import { ResponseTimeInterceptor } from './interceptors/response-time.interceptor';

@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: ResponseTimeInterceptor,
    },
  ],
})
export class AppModule {}

// Log output:
{
  "method": "GET",
  "url": "/api/users/123",
  "statusCode": 200,
  "duration": "145ms",
  "timestamp": "2025-12-30T10:30:45.123Z"
}
```

---

### **Method 2: Middleware with Response Headers**

```typescript
// middleware/response-time.middleware.ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class ResponseTimeMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    const start = Date.now();

    // Add response time header
    res.on('finish', () => {
      const duration = Date.now() - start;
      res.setHeader('X-Response-Time', `${duration}ms`);
    });

    next();
  }
}

// app.module.ts
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { ResponseTimeMiddleware } from './middleware/response-time.middleware';

@Module({})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(ResponseTimeMiddleware).forRoutes('*');
  }
}

// Client can see response time in headers:
// X-Response-Time: 145ms
```

---

### **Method 3: Prometheus Metrics (Recommended)**

```typescript
// Install: npm install prom-client

// metrics/prometheus.service.ts
import { Injectable } from '@nestjs/common';
import * as client from 'prom-client';

@Injectable()
export class PrometheusService {
  private register: client.Registry;
  private httpRequestDuration: client.Histogram;
  private httpRequestsTotal: client.Counter;

  constructor() {
    this.register = new client.Registry();
    
    // Response time histogram
    this.httpRequestDuration = new client.Histogram({
      name: 'http_request_duration_seconds',
      help: 'Duration of HTTP requests in seconds',
      labelNames: ['method', 'route', 'status_code'],
      // Buckets for percentile calculation
      buckets: [0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10],
      registers: [this.register],
    });

    // Request counter
    this.httpRequestsTotal = new client.Counter({
      name: 'http_requests_total',
      help: 'Total number of HTTP requests',
      labelNames: ['method', 'route', 'status_code'],
      registers: [this.register],
    });

    // Default metrics (CPU, memory)
    client.collectDefaultMetrics({ register: this.register });
  }

  recordRequest(method: string, route: string, statusCode: number, duration: number) {
    // Record duration in seconds
    this.httpRequestDuration.observe(
      { method, route, status_code: statusCode },
      duration / 1000,
    );
    
    // Increment counter
    this.httpRequestsTotal.inc({ method, route, status_code: statusCode });
  }

  async getMetrics(): Promise<string> {
    return this.register.metrics();
  }
}

// interceptors/prometheus.interceptor.ts
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';
import { PrometheusService } from '../metrics/prometheus.service';

@Injectable()
export class PrometheusInterceptor implements NestInterceptor {
  constructor(private readonly prometheus: PrometheusService) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const start = Date.now();

    return next.handle().pipe(
      tap({
        next: () => {
          const response = context.switchToHttp().getResponse();
          const duration = Date.now() - start;
          
          this.prometheus.recordRequest(
            request.method,
            request.route?.path || request.url,
            response.statusCode,
            duration,
          );
        },
        error: (error) => {
          const duration = Date.now() - start;
          this.prometheus.recordRequest(
            request.method,
            request.route?.path || request.url,
            error.status || 500,
            duration,
          );
        },
      }),
    );
  }
}

// metrics.controller.ts
import { Controller, Get, Header } from '@nestjs/common';
import { PrometheusService } from './prometheus.service';

@Controller('metrics')
export class MetricsController {
  constructor(private readonly prometheus: PrometheusService) {}

  @Get()
  @Header('Content-Type', 'text/plain; version=0.0.4')
  async getMetrics(): Promise<string> {
    return this.prometheus.getMetrics();
  }
}

// Query Prometheus:
// Average response time:
rate(http_request_duration_seconds_sum[5m]) / rate(http_request_duration_seconds_count[5m])

// 95th percentile:
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

// 99th percentile:
histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))
```

---

### **Method 4: Custom Metrics Service**

```typescript
// metrics/metrics.service.ts
import { Injectable } from '@nestjs/common';

interface RequestMetric {
  method: string;
  url: string;
  statusCode: number;
  duration: number;
  timestamp: Date;
}

@Injectable()
export class MetricsService {
  private metrics: RequestMetric[] = [];
  private readonly MAX_METRICS = 10000;  // Keep last 10k requests

  recordRequest(method: string, url: string, statusCode: number, duration: number) {
    this.metrics.push({
      method,
      url,
      statusCode,
      duration,
      timestamp: new Date(),
    });

    // Keep only recent metrics
    if (this.metrics.length > this.MAX_METRICS) {
      this.metrics.shift();
    }
  }

  getStats(route?: string) {
    const filtered = route
      ? this.metrics.filter((m) => m.url === route)
      : this.metrics;

    if (filtered.length === 0) {
      return null;
    }

    const durations = filtered.map((m) => m.duration).sort((a, b) => a - b);
    const totalRequests = filtered.length;
    const errorCount = filtered.filter((m) => m.statusCode >= 500).length;

    return {
      totalRequests,
      errorCount,
      errorRate: ((errorCount / totalRequests) * 100).toFixed(2) + '%',
      avg: this.average(durations),
      min: Math.min(...durations),
      max: Math.max(...durations),
      p50: this.percentile(durations, 0.5),
      p95: this.percentile(durations, 0.95),
      p99: this.percentile(durations, 0.99),
    };
  }

  getSlowRequests(threshold = 1000) {
    return this.metrics
      .filter((m) => m.duration > threshold)
      .sort((a, b) => b.duration - a.duration)
      .slice(0, 100);  // Top 100 slowest
  }

  private average(arr: number[]): number {
    return Math.round(arr.reduce((a, b) => a + b, 0) / arr.length);
  }

  private percentile(arr: number[], p: number): number {
    const index = Math.ceil(arr.length * p) - 1;
    return arr[index];
  }
}

// metrics.controller.ts
import { Controller, Get, Query } from '@nestjs/common';
import { MetricsService } from './metrics.service';

@Controller('metrics')
export class MetricsController {
  constructor(private readonly metricsService: MetricsService) {}

  @Get('stats')
  getStats(@Query('route') route?: string) {
    return this.metricsService.getStats(route);
  }

  @Get('slow')
  getSlowRequests(@Query('threshold') threshold?: string) {
    return this.metricsService.getSlowRequests(Number(threshold) || 1000);
  }
}

// GET /metrics/stats
{
  "totalRequests": 15234,
  "errorCount": 23,
  "errorRate": "0.15%",
  "avg": 145,
  "min": 12,
  "max": 3456,
  "p50": 120,
  "p95": 450,
  "p99": 1200
}

// GET /metrics/slow?threshold=1000
[
  {
    "method": "POST",
    "url": "/api/orders/create",
    "statusCode": 200,
    "duration": 3456,
    "timestamp": "2025-12-30T10:30:45.123Z"
  }
]
```

---

### **Method 5: Database Query Monitoring**

```typescript
// database/query-logger.ts
import { Logger } from '@nestjs/common';
import { QueryRunner } from 'typeorm';

export class QueryLogger {
  private logger = new Logger('DatabaseQuery');

  logQuery(query: string, parameters?: any[], queryRunner?: QueryRunner) {
    // This is called by TypeORM for every query
  }

  logQuerySlow(time: number, query: string, parameters?: any[]) {
    this.logger.warn(
      JSON.stringify({
        type: 'slow_query',
        duration: `${time}ms`,
        query,
        parameters,
      }),
    );
  }
}

// ormconfig.ts
export default {
  type: 'postgres',
  host: 'localhost',
  port: 5432,
  username: 'user',
  password: 'password',
  database: 'mydb',
  logging: true,
  logger: new QueryLogger(),
  maxQueryExecutionTime: 1000,  // Log queries taking >1000ms
};
```

---

### **Method 6: APM Tool Integration (Datadog)**

```typescript
// tracer.ts
import tracer from 'dd-trace';

tracer.init({
  service: 'my-nestjs-app',
  env: process.env.NODE_ENV,
  logInjection: true,
  runtimeMetrics: true,
});

export default tracer;

// main.ts
import './tracer';  // Must be first!
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();

// Datadog automatically tracks:
✅ Request response times
✅ Database query durations
✅ External API call latency
✅ p50, p75, p90, p95, p99 percentiles
✅ Slow trace analysis
✅ Resource (endpoint) performance
```

---

### **Method 7: Response Time Buckets**

```typescript
// metrics/response-time-buckets.service.ts
import { Injectable } from '@nestjs/common';

interface BucketCounts {
  fast: number;      // < 100ms
  normal: number;    // 100-500ms
  slow: number;      // 500-1000ms
  verySlow: number;  // > 1000ms
}

@Injectable()
export class ResponseTimeBucketsService {
  private buckets: Map<string, BucketCounts> = new Map();

  recordRequest(route: string, duration: number) {
    if (!this.buckets.has(route)) {
      this.buckets.set(route, { fast: 0, normal: 0, slow: 0, verySlow: 0 });
    }

    const bucket = this.buckets.get(route)!;

    if (duration < 100) {
      bucket.fast++;
    } else if (duration < 500) {
      bucket.normal++;
    } else if (duration < 1000) {
      bucket.slow++;
    } else {
      bucket.verySlow++;
    }
  }

  getStats() {
    const stats: any = {};
    
    this.buckets.forEach((counts, route) => {
      const total = counts.fast + counts.normal + counts.slow + counts.verySlow;
      
      stats[route] = {
        total,
        distribution: {
          fast: `${((counts.fast / total) * 100).toFixed(1)}%`,
          normal: `${((counts.normal / total) * 100).toFixed(1)}%`,
          slow: `${((counts.slow / total) * 100).toFixed(1)}%`,
          verySlow: `${((counts.verySlow / total) * 100).toFixed(1)}%`,
        },
      };
    });
    
    return stats;
  }
}

// GET /metrics/buckets
{
  "/api/users": {
    "total": 10000,
    "distribution": {
      "fast": "85.0%",      // < 100ms
      "normal": "12.5%",    // 100-500ms
      "slow": "2.0%",       // 500-1000ms
      "verySlow": "0.5%"    // > 1000ms
    }
  }
}
```

---

### **Method 8: Real-Time Monitoring Dashboard**

```typescript
// metrics/real-time-metrics.gateway.ts
import {
  WebSocketGateway,
  WebSocketServer,
  OnGatewayConnection,
} from '@nestjs/websockets';
import { Server } from 'socket.io';

@WebSocketGateway({ cors: true })
export class RealTimeMetricsGateway implements OnGatewayConnection {
  @WebSocketServer()
  server: Server;

  handleConnection(client: any) {
    console.log('Client connected for real-time metrics');
  }

  emitMetric(data: any) {
    this.server.emit('metric', data);
  }
}

// interceptors/real-time-metrics.interceptor.ts
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';
import { RealTimeMetricsGateway } from '../metrics/real-time-metrics.gateway';

@Injectable()
export class RealTimeMetricsInterceptor implements NestInterceptor {
  constructor(private readonly gateway: RealTimeMetricsGateway) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const start = Date.now();

    return next.handle().pipe(
      tap(() => {
        const response = context.switchToHttp().getResponse();
        const duration = Date.now() - start;
        
        // Emit to WebSocket clients
        this.gateway.emitMetric({
          method: request.method,
          url: request.url,
          statusCode: response.statusCode,
          duration,
          timestamp: new Date().toISOString(),
        });
      }),
    );
  }
}

// Frontend (React/Vue/Angular)
const socket = io('http://localhost:3000');

socket.on('metric', (data) => {
  console.log('New request:', data);
  // Update real-time dashboard
  updateChart(data.duration);
});
```

---

### **Method 9: Performance Budget Alerts**

```typescript
// performance/performance-monitor.service.ts
import { Injectable, Logger } from '@nestjs/common';
import * as Sentry from '@sentry/node';

interface PerformanceBudget {
  route: string;
  maxAvgDuration: number;  // Average shouldn't exceed this
  maxP95Duration: number;  // 95th percentile shouldn't exceed this
}

@Injectable()
export class PerformanceMonitorService {
  private readonly logger = new Logger('PerformanceMonitor');
  private budgets: PerformanceBudget[] = [
    { route: '/api/users', maxAvgDuration: 100, maxP95Duration: 200 },
    { route: '/api/orders', maxAvgDuration: 150, maxP95Duration: 300 },
    { route: '/api/products', maxAvgDuration: 50, maxP95Duration: 100 },
  ];
  private metrics: Map<string, number[]> = new Map();

  recordRequest(route: string, duration: number) {
    if (!this.metrics.has(route)) {
      this.metrics.set(route, []);
    }
    
    this.metrics.get(route)!.push(duration);
    
    // Check budget every 100 requests
    if (this.metrics.get(route)!.length % 100 === 0) {
      this.checkBudget(route);
    }
  }

  private checkBudget(route: string) {
    const budget = this.budgets.find((b) => b.route === route);
    if (!budget) return;

    const durations = this.metrics.get(route)!.slice(-1000);  // Last 1000 requests
    const avg = durations.reduce((a, b) => a + b) / durations.length;
    const p95 = this.percentile(durations, 0.95);

    if (avg > budget.maxAvgDuration) {
      this.logger.error(
        `Performance budget exceeded for ${route}: avg=${avg}ms (budget: ${budget.maxAvgDuration}ms)`,
      );
      
      // Alert via Sentry
      Sentry.captureMessage(`Performance budget exceeded: ${route}`, {
        level: 'warning',
        tags: { route, metric: 'avg_duration' },
        extra: { avg, budget: budget.maxAvgDuration },
      });
    }

    if (p95 > budget.maxP95Duration) {
      this.logger.error(
        `Performance budget exceeded for ${route}: p95=${p95}ms (budget: ${budget.maxP95Duration}ms)`,
      );
    }
  }

  private percentile(arr: number[], p: number): number {
    const sorted = arr.sort((a, b) => a - b);
    const index = Math.ceil(sorted.length * p) - 1;
    return sorted[index];
  }
}
```

---

### **Method 10: Production Complete Setup**

```typescript
// monitoring/response-time.module.ts
import { Module, Global } from '@nestjs/common';
import { APP_INTERCEPTOR } from '@nestjs/core';
import { PrometheusService } from './prometheus.service';
import { PrometheusInterceptor } from './prometheus.interceptor';
import { MetricsController } from './metrics.controller';
import { PerformanceMonitorService } from './performance-monitor.service';

@Global()
@Module({
  providers: [
    PrometheusService,
    PerformanceMonitorService,
    {
      provide: APP_INTERCEPTOR,
      useClass: PrometheusInterceptor,
    },
  ],
  controllers: [MetricsController],
  exports: [PrometheusService, PerformanceMonitorService],
})
export class ResponseTimeModule {}

// app.module.ts
import { Module } from '@nestjs/common';
import { ResponseTimeModule } from './monitoring/response-time.module';

@Module({
  imports: [ResponseTimeModule],
})
export class AppModule {}
```

---

### **Grafana Dashboard Query Examples:**

```promql
# Average response time by endpoint
rate(http_request_duration_seconds_sum[5m]) / rate(http_request_duration_seconds_count[5m])

# 95th percentile response time
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# Requests per second
rate(http_requests_total[1m])

# Slow requests (>1s)
http_request_duration_seconds_bucket{le="1"}

# Error rate
rate(http_requests_total{status_code=~"5.."}[5m]) / rate(http_requests_total[5m])
```

---

### **Best Practices:**

```typescript
// ✅ GOOD - Use histograms for percentiles
const histogram = new client.Histogram({
  name: 'http_request_duration_seconds',
  buckets: [0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5],
});

// ✅ GOOD - Track by route (not full URL)
route: request.route?.path || '/unknown'  // '/api/users/:id'

// ✅ GOOD - Alert on p95/p99, not average
if (p99 > 1000) {
  alert('p99 response time exceeded 1s');
}

// ✅ GOOD - Separate endpoints in metrics
labelNames: ['method', 'route', 'status_code']

// ✅ GOOD - Monitor database queries separately
this.logger.warn('Slow query', { duration, query });

// ✅ GOOD - Set performance budgets
maxAvgDuration: 100ms per endpoint

// ❌ BAD - Only tracking averages
avg_response_time  // Hides outliers!

// ❌ BAD - Full URL in metrics (high cardinality)
url: request.url  // '/api/users/123456' creates too many metrics

// ❌ BAD - No thresholds
// No alerts when performance degrades

// ❌ BAD - Not tracking database queries
// API might be fast, but DB is slow
```

**Key Takeaway:** Monitor API response times using **interceptors** to measure duration, **Prometheus histograms** for percentiles (p50, p95, p99), **APM tools** for automatic tracking, and **performance budgets** with alerts on threshold violations to detect degraded performance.

</details>

<details>
<summary><strong>30. How do you set up alerts for errors?</strong></summary>

**Answer:**

Setting up **error alerts** ensures immediate notification when critical issues occur in production. Use **APM tools** (Sentry, Datadog, New Relic), **custom alerting services** (PagerDuty, Slack), or **log-based alerts** (CloudWatch, Elasticsearch) to notify teams of errors, spikes, or anomalies.

---

### **Method 1: Sentry Alerts**

```typescript
// Install: npm install @sentry/node

// sentry.config.ts
import * as Sentry from '@sentry/node';

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 1.0,
});

// Sentry Dashboard → Alerts → Create Alert Rule:

// Alert Rule 1: New Error
{
  name: 'New Error Alert',
  conditions: [
    'A new issue is created',
  ],
  actions: [
    'Send notification to #engineering Slack channel',
    'Send email to team@company.com',
  ],
}

// Alert Rule 2: Error Spike
{
  name: 'Error Spike Alert',
  conditions: [
    'Number of events is greater than 100 in 1 minute',
    'Compared to the same time yesterday',
  ],
  actions: [
    'Send notification to #critical-alerts Slack',
    'Create PagerDuty incident',
  ],
}

// Alert Rule 3: High Error Rate
{
  name: 'High Error Rate',
  conditions: [
    'Error rate is above 5%',
    'In the last 5 minutes',
  ],
  actions: [
    'Send notification to #engineering',
    'Email on-call engineer',
  ],
}

// Alert Rule 4: Critical Endpoint Errors
{
  name: 'Payment Endpoint Errors',
  conditions: [
    'The event is tagged with endpoint:/api/payments',
    'AND level:error',
  ],
  actions: [
    'Create PagerDuty incident (high priority)',
    'Send SMS to on-call engineer',
  ],
}
```

---

### **Method 2: Custom Alert Service (Slack)**

```typescript
// Install: npm install @slack/webhook

// alerts/slack-alert.service.ts
import { Injectable, Logger } from '@nestjs/common';
import { IncomingWebhook } from '@slack/webhook';

@Injectable()
export class SlackAlertService {
  private webhook: IncomingWebhook;
  private readonly logger = new Logger('SlackAlert');

  constructor() {
    this.webhook = new IncomingWebhook(process.env.SLACK_WEBHOOK_URL);
  }

  async sendErrorAlert(error: Error, context: any) {
    try {
      await this.webhook.send({
        text: '⚠️ *Error Alert*',
        attachments: [
          {
            color: 'danger',
            title: error.name,
            text: error.message,
            fields: [
              {
                title: 'Environment',
                value: process.env.NODE_ENV,
                short: true,
              },
              {
                title: 'Service',
                value: 'my-nestjs-app',
                short: true,
              },
              {
                title: 'Endpoint',
                value: context.url,
                short: true,
              },
              {
                title: 'Method',
                value: context.method,
                short: true,
              },
              {
                title: 'User ID',
                value: context.userId || 'N/A',
                short: true,
              },
              {
                title: 'Timestamp',
                value: new Date().toISOString(),
                short: true,
              },
            ],
            footer: 'NestJS Error Monitor',
          },
        ],
      });
    } catch (err) {
      this.logger.error('Failed to send Slack alert', err);
    }
  }

  async sendCriticalAlert(message: string, details: any) {
    await this.webhook.send({
      text: '🔥 *CRITICAL ALERT* 🔥',
      attachments: [
        {
          color: '#ff0000',
          title: message,
          fields: Object.entries(details).map(([key, value]) => ({
            title: key,
            value: String(value),
            short: true,
          })),
          footer: 'Immediate action required',
        },
      ],
    });
  }
}

// filters/alert-exception.filter.ts
import { ExceptionFilter, Catch, ArgumentsHost } from '@nestjs/common';
import { SlackAlertService } from '../alerts/slack-alert.service';

@Catch()
export class AlertExceptionFilter implements ExceptionFilter {
  constructor(private readonly slackAlert: SlackAlertService) {}

  catch(exception: any, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const request = ctx.getRequest();
    const response = ctx.getResponse();

    // Send alert for server errors (5xx)
    if (!exception.status || exception.status >= 500) {
      this.slackAlert.sendErrorAlert(exception, {
        url: request.url,
        method: request.method,
        userId: request.user?.id,
      });
    }

    response.status(exception.status || 500).json({
      statusCode: exception.status || 500,
      message: exception.message,
    });
  }
}
```

---

### **Method 3: Email Alerts (NodeMailer)**

```typescript
// Install: npm install nodemailer

// alerts/email-alert.service.ts
import { Injectable, Logger } from '@nestjs/common';
import * as nodemailer from 'nodemailer';

@Injectable()
export class EmailAlertService {
  private transporter: nodemailer.Transporter;
  private readonly logger = new Logger('EmailAlert');

  constructor() {
    this.transporter = nodemailer.createTransport({
      host: process.env.SMTP_HOST,
      port: Number(process.env.SMTP_PORT),
      secure: true,
      auth: {
        user: process.env.SMTP_USER,
        pass: process.env.SMTP_PASS,
      },
    });
  }

  async sendErrorAlert(error: Error, context: any) {
    const html = `
      <h2>⚠️ Error Alert</h2>
      <table>
        <tr><td><strong>Error:</strong></td><td>${error.name}</td></tr>
        <tr><td><strong>Message:</strong></td><td>${error.message}</td></tr>
        <tr><td><strong>Endpoint:</strong></td><td>${context.url}</td></tr>
        <tr><td><strong>Method:</strong></td><td>${context.method}</td></tr>
        <tr><td><strong>User ID:</strong></td><td>${context.userId || 'N/A'}</td></tr>
        <tr><td><strong>Environment:</strong></td><td>${process.env.NODE_ENV}</td></tr>
        <tr><td><strong>Timestamp:</strong></td><td>${new Date().toISOString()}</td></tr>
      </table>
      <h3>Stack Trace:</h3>
      <pre>${error.stack}</pre>
    `;

    try {
      await this.transporter.sendMail({
        from: process.env.ALERT_EMAIL_FROM,
        to: process.env.ALERT_EMAIL_TO,
        subject: `[🚨 ERROR] ${error.name} in ${process.env.NODE_ENV}`,
        html,
      });
    } catch (err) {
      this.logger.error('Failed to send email alert', err);
    }
  }
}
```

---

### **Method 4: PagerDuty Integration**

```typescript
// Install: npm install @pagerduty/pdjs

// alerts/pagerduty-alert.service.ts
import { Injectable, Logger } from '@nestjs/common';
import { event } from '@pagerduty/pdjs';

@Injectable()
export class PagerDutyAlertService {
  private readonly logger = new Logger('PagerDutyAlert');

  async triggerIncident(error: Error, context: any) {
    try {
      await event({
        data: {
          routing_key: process.env.PAGERDUTY_ROUTING_KEY,
          event_action: 'trigger',
          payload: {
            summary: `Error: ${error.message}`,
            severity: 'error',
            source: 'my-nestjs-app',
            custom_details: {
              error_name: error.name,
              endpoint: context.url,
              method: context.method,
              user_id: context.userId,
              environment: process.env.NODE_ENV,
              stack_trace: error.stack,
            },
          },
        },
      });
    } catch (err) {
      this.logger.error('Failed to trigger PagerDuty incident', err);
    }
  }

  async resolveIncident(dedupKey: string) {
    await event({
      data: {
        routing_key: process.env.PAGERDUTY_ROUTING_KEY,
        event_action: 'resolve',
        dedup_key: dedupKey,
      },
    });
  }
}

// Usage in exception filter
@Catch()
export class CriticalExceptionFilter implements ExceptionFilter {
  constructor(private readonly pagerDuty: PagerDutyAlertService) {}

  catch(exception: any, host: ArgumentsHost) {
    // Trigger PagerDuty for critical errors
    if (this.isCritical(exception)) {
      this.pagerDuty.triggerIncident(exception, {
        url: request.url,
        method: request.method,
        userId: request.user?.id,
      });
    }
  }

  private isCritical(exception: any): boolean {
    // Database connection errors
    if (exception.message?.includes('ECONNREFUSED')) return true;
    
    // Payment errors
    if (exception.endpoint?.includes('/payment')) return true;
    
    // Authentication service down
    if (exception.service === 'auth') return true;
    
    return false;
  }
}
```

---

### **Method 5: CloudWatch Alarms**

```typescript
// AWS CloudWatch metric filter + alarm

// 1. Create metric filter for errors
aws logs put-metric-filter \
  --log-group-name "/aws/lambda/my-nestjs-app" \
  --filter-name ErrorCount \
  --filter-pattern "[timestamp, level=ERROR, ...]" \
  --metric-transformations \
    metricName=ErrorCount,metricNamespace=MyApp,metricValue=1

// 2. Create alarm
aws cloudwatch put-metric-alarm \
  --alarm-name "High-Error-Rate" \
  --alarm-description "Alert when error rate exceeds 10 in 5 minutes" \
  --metric-name ErrorCount \
  --namespace MyApp \
  --statistic Sum \
  --period 300 \
  --threshold 10 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:AlertTopic

// winston logger with CloudWatch
import * as winston from 'winston';
import WinstonCloudWatch from 'winston-cloudwatch';

const logger = winston.createLogger({
  transports: [
    new WinstonCloudWatch({
      logGroupName: '/aws/lambda/my-nestjs-app',
      logStreamName: 'errors',
      awsRegion: 'us-east-1',
    }),
  ],
});

logger.error('Payment processing failed', { orderId: '123' });
// Triggers CloudWatch alarm if threshold exceeded
```

---

### **Method 6: Datadog Monitors**

```typescript
// Datadog Dashboard → Monitors → New Monitor

// Monitor 1: Error Rate
{
  name: 'High Error Rate',
  type: 'metric alert',
  query: 'avg(last_5m):sum:my_app.errors{env:production} > 50',
  message: `
    🚨 High error rate detected!
    
    Error count: {{value}}
    Threshold: 50 errors in 5 minutes
    
    @slack-engineering @pagerduty-oncall
  `,
  tags: ['env:production', 'team:backend'],
}

// Monitor 2: Database Connection Errors
{
  name: 'Database Connection Errors',
  type: 'log alert',
  query: 'logs("error.type:DatabaseConnectionError").index("main").rollup("count").last("5m") > 5',
  message: '@pagerduty-oncall Database connection failing!',
}

// Monitor 3: Service Health Check Failing
{
  name: 'Health Check Failing',
  type: 'service check',
  query: '"http.check".over("url:https://api.myapp.com/health").by("*").last(2).count_by_status()',
  message: '@slack-critical Service health check failing for 2 consecutive checks',
}
```

---

### **Method 7: Alert Aggregation (Prevent Alert Fatigue)**

```typescript
// alerts/alert-aggregator.service.ts
import { Injectable } from '@nestjs/common';

interface AlertBucket {
  errorType: string;
  count: number;
  firstOccurrence: Date;
  lastOccurrence: Date;
}

@Injectable()
export class AlertAggregatorService {
  private buckets: Map<string, AlertBucket> = new Map();
  private readonly AGGREGATION_WINDOW = 5 * 60 * 1000;  // 5 minutes
  private readonly ALERT_THRESHOLD = 10;  // Alert after 10 similar errors

  recordError(errorType: string) {
    if (!this.buckets.has(errorType)) {
      this.buckets.set(errorType, {
        errorType,
        count: 0,
        firstOccurrence: new Date(),
        lastOccurrence: new Date(),
      });
    }

    const bucket = this.buckets.get(errorType)!;
    bucket.count++;
    bucket.lastOccurrence = new Date();

    // Alert if threshold exceeded
    if (bucket.count === this.ALERT_THRESHOLD) {
      this.sendAggregatedAlert(bucket);
    }

    // Reset buckets older than aggregation window
    this.cleanupOldBuckets();
  }

  private sendAggregatedAlert(bucket: AlertBucket) {
    // Send single alert for multiple occurrences
    console.log(`Alert: ${bucket.errorType} occurred ${bucket.count} times`);
    // Send to Slack, PagerDuty, etc.
  }

  private cleanupOldBuckets() {
    const now = Date.now();
    this.buckets.forEach((bucket, key) => {
      if (now - bucket.lastOccurrence.getTime() > this.AGGREGATION_WINDOW) {
        this.buckets.delete(key);
      }
    });
  }
}

// ✅ Instead of 100 individual alerts, send 1 aggregated alert
```

---

### **Method 8: Alert Routing by Severity**

```typescript
// alerts/alert-router.service.ts
import { Injectable } from '@nestjs/common';
import { SlackAlertService } from './slack-alert.service';
import { EmailAlertService } from './email-alert.service';
import { PagerDutyAlertService } from './pagerduty-alert.service';

enum AlertSeverity {
  INFO = 'info',
  WARNING = 'warning',
  ERROR = 'error',
  CRITICAL = 'critical',
}

@Injectable()
export class AlertRouterService {
  constructor(
    private readonly slack: SlackAlertService,
    private readonly email: EmailAlertService,
    private readonly pagerDuty: PagerDutyAlertService,
  ) {}

  async routeAlert(error: Error, severity: AlertSeverity, context: any) {
    switch (severity) {
      case AlertSeverity.INFO:
        // Just log, no alert
        console.log('Info:', error.message);
        break;

      case AlertSeverity.WARNING:
        // Slack notification
        await this.slack.sendErrorAlert(error, context);
        break;

      case AlertSeverity.ERROR:
        // Slack + Email
        await Promise.all([
          this.slack.sendErrorAlert(error, context),
          this.email.sendErrorAlert(error, context),
        ]);
        break;

      case AlertSeverity.CRITICAL:
        // All channels + PagerDuty (wake up on-call)
        await Promise.all([
          this.slack.sendCriticalAlert(error.message, context),
          this.email.sendErrorAlert(error, context),
          this.pagerDuty.triggerIncident(error, context),
        ]);
        break;
    }
  }

  determineSeverity(error: Error, context: any): AlertSeverity {
    // Database connection error
    if (error.message.includes('ECONNREFUSED')) {
      return AlertSeverity.CRITICAL;
    }

    // Payment endpoint
    if (context.url.includes('/payment')) {
      return AlertSeverity.CRITICAL;
    }

    // Authentication error
    if (error.name === 'UnauthorizedException') {
      return AlertSeverity.INFO;  // Expected, don't alert
    }

    // Default to error
    return AlertSeverity.ERROR;
  }
}
```

---

### **Method 9: Alert Silencing (Maintenance Windows)**

```typescript
// alerts/alert-silencer.service.ts
import { Injectable } from '@nestjs/common';

interface MaintenanceWindow {
  start: Date;
  end: Date;
  reason: string;
}

@Injectable()
export class AlertSilencerService {
  private maintenanceWindows: MaintenanceWindow[] = [];

  addMaintenanceWindow(start: Date, end: Date, reason: string) {
    this.maintenanceWindows.push({ start, end, reason });
  }

  shouldSilenceAlert(): boolean {
    const now = new Date();
    return this.maintenanceWindows.some(
      (window) => now >= window.start && now <= window.end,
    );
  }

  async sendAlertIfNotSilenced(alert: () => Promise<void>) {
    if (this.shouldSilenceAlert()) {
      console.log('Alert silenced due to maintenance window');
      return;
    }
    await alert();
  }
}

// Usage
if (!alertSilencer.shouldSilenceAlert()) {
  await slackAlert.sendErrorAlert(error, context);
}
```

---

### **Method 10: Complete Production Alert System**

```typescript
// alerts/alert.module.ts
import { Module, Global } from '@nestjs/common';
import { SlackAlertService } from './slack-alert.service';
import { EmailAlertService } from './email-alert.service';
import { PagerDutyAlertService } from './pagerduty-alert.service';
import { AlertRouterService } from './alert-router.service';
import { AlertAggregatorService } from './alert-aggregator.service';
import { AlertSilencerService } from './alert-silencer.service';

@Global()
@Module({
  providers: [
    SlackAlertService,
    EmailAlertService,
    PagerDutyAlertService,
    AlertRouterService,
    AlertAggregatorService,
    AlertSilencerService,
  ],
  exports: [AlertRouterService],
})
export class AlertModule {}

// filters/production-exception.filter.ts
import { ExceptionFilter, Catch, ArgumentsHost } from '@nestjs/common';
import { AlertRouterService } from '../alerts/alert-router.service';

@Catch()
export class ProductionExceptionFilter implements ExceptionFilter {
  constructor(private readonly alertRouter: AlertRouterService) {}

  async catch(exception: any, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const request = ctx.getRequest();
    const response = ctx.getResponse();

    const context = {
      url: request.url,
      method: request.method,
      userId: request.user?.id,
      ip: request.ip,
    };

    // Determine severity and route alert
    const severity = this.alertRouter.determineSeverity(exception, context);
    await this.alertRouter.routeAlert(exception, severity, context);

    // Send response
    response.status(exception.status || 500).json({
      statusCode: exception.status || 500,
      message: exception.message,
    });
  }
}
```

---

### **Alert Configuration Best Practices:**

```typescript
// ✅ GOOD - Set meaningful thresholds
threshold: 50 errors in 5 minutes  // Not too sensitive

// ✅ GOOD - Use alert aggregation
// Send 1 alert for 100 similar errors

// ✅ GOOD - Route by severity
CRITICAL → PagerDuty (wake up on-call)
ERROR → Slack + Email
WARNING → Slack only

// ✅ GOOD - Include context in alerts
{
  error: 'Payment failed',
  orderId: '123',
  userId: '456',
  amount: 99.99,
}

// ✅ GOOD - Silence during maintenance
if (maintenanceWindow) {
  silenceAlerts();
}

// ✅ GOOD - Alert on trends, not individual errors
if (errorRate > 5% for 5 minutes) {
  alert();
}

// ❌ BAD - Alert on every error
// Creates alert fatigue

// ❌ BAD - No context in alerts
alert('An error occurred');  // What error? Where?

// ❌ BAD - Same channel for all severities
// Info logs mixed with critical alerts

// ❌ BAD - No alert aggregation
// 1000 similar errors = 1000 alerts
```

**Key Takeaway:** Set up error alerts using **Sentry** for automatic error detection, **Slack/Email** for notifications, **PagerDuty** for critical incidents, **alert aggregation** to prevent fatigue, and **severity-based routing** (INFO → log, WARNING → Slack, ERROR → Slack+Email, CRITICAL → PagerDuty).

</details>

## Health Checks

<details>
<summary><strong>31. What are health checks and why are they important?</strong></summary>

**Answer:**

Health checks are **HTTP endpoints** that report the **status** and **availability** of your application and its dependencies (database, cache, external APIs). They're essential for **orchestrators** (Kubernetes, Docker Swarm), **load balancers**, and **monitoring systems** to determine if a service is healthy and ready to handle traffic.

---

### **Why Health Checks Are Important:**

```typescript
// Without health checks:
// ❌ Load balancer sends traffic to crashed instances
// ❌ Kubernetes doesn't know when to restart pods
// ❌ No automatic recovery from failures
// ❌ Manual intervention required

// With health checks:
// ✅ Automatic traffic routing to healthy instances
// ✅ Automatic pod restarts in Kubernetes
// ✅ Self-healing infrastructure
// ✅ Zero-downtime deployments
```

---

### **Types of Health Checks:**

#### **1. Liveness Probe**
```typescript
// "Is the application alive?"
// If liveness fails → Restart the container

GET /health/live

// Success (200):
{
  "status": "ok",
  "info": {
    "app": { "status": "up" }
  }
}

// Failure (503):
{
  "status": "error",
  "error": {
    "app": { "status": "down", "message": "Deadlock detected" }
  }
}

// Use cases:
// - Application deadlock
// - Memory leak causing OOM
// - Infinite loop
```

#### **2. Readiness Probe**
```typescript
// "Is the application ready to receive traffic?"
// If readiness fails → Stop sending traffic (don't restart)

GET /health/ready

// Success (200):
{
  "status": "ok",
  "info": {
    "database": { "status": "up" },
    "redis": { "status": "up" },
    "api": { "status": "up" }
  }
}

// Failure (503):
{
  "status": "error",
  "error": {
    "database": { "status": "down", "message": "Connection timeout" }
  }
}

// Use cases:
// - Database connection down
// - Cache warming in progress
// - Dependency service unavailable
```

#### **3. Startup Probe**
```typescript
// "Has the application finished starting?"
// Used for slow-starting applications

GET /health/startup

// Success (200): Application fully initialized
// Failure (503): Still starting

// Use cases:
// - Large data loading on startup
// - Migration scripts running
// - Cache preloading
```

---

### **Health Check Architecture:**

```
┌─────────────────────────────────────────────────────────────┐
│                    Load Balancer / Ingress                  │
│  Checks: GET /health/ready every 5 seconds                  │
└─────────────────────────────────────────────────────────────┘
                            ↓
              ┌─────────────┼─────────────┐
              ↓             ↓             ↓
    ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
    │  Pod 1      │ │  Pod 2      │ │  Pod 3      │
    │  ✅ Healthy │ │  ❌ Unhealthy│ │  ✅ Healthy │
    │  Get traffic│ │  No traffic │ │  Get traffic│
    └─────────────┘ └─────────────┘ └─────────────┘
            │               │               │
            └───────────────┴───────────────┘
                            ↓
                  Health Check Endpoint
                      /health/ready
                            ↓
              ┌─────────────┴─────────────┐
              ↓                           ↓
    ┌─────────────────┐         ┌─────────────────┐
    │  Database       │         │  Redis Cache    │
    │  Check: ✅ UP   │         │  Check: ✅ UP   │
    └─────────────────┘         └─────────────────┘
```

---

### **Basic Health Check Example:**

```typescript
// health/health.controller.ts
import { Controller, Get } from '@nestjs/common';

@Controller('health')
export class HealthController {
  @Get()
  check() {
    return {
      status: 'ok',
      timestamp: new Date().toISOString(),
      uptime: process.uptime(),
      memory: process.memoryUsage(),
    };
  }
}

// GET /health
{
  "status": "ok",
  "timestamp": "2025-12-30T10:30:45.123Z",
  "uptime": 3600.5,  // seconds
  "memory": {
    "rss": 52428800,        // Resident Set Size
    "heapTotal": 20971520,  // Total heap allocated
    "heapUsed": 10485760,   // Heap actually used
    "external": 1024000     // C++ objects bound to JS
  }
}
```

---

### **Benefits of Health Checks:**

```typescript
✅ 1. Automatic Failover
   // Load balancer detects unhealthy instance
   // Traffic automatically routed to healthy instances
   
✅ 2. Self-Healing
   // Kubernetes detects failed liveness probe
   // Pod automatically restarted
   
✅ 3. Zero-Downtime Deployments
   // New pods not marked ready until dependencies available
   // Old pods kept running until new pods ready
   
✅ 4. Dependency Monitoring
   // Single endpoint shows status of all dependencies
   // Quick identification of bottlenecks
   
✅ 5. Proactive Monitoring
   // Alert before complete failure
   // Degraded performance detected early
   
✅ 6. Chaos Engineering
   // Intentionally fail health checks
   // Test auto-recovery mechanisms
```

---

### **Real-World Scenario:**

```typescript
// Scenario: Database connection pool exhausted

// Without health check:
// 1. Users get 500 errors
// 2. Load balancer keeps sending traffic
// 3. All pods failing
// 4. Manual restart required
// 5. 15 minutes downtime

// With health check:
// 1. Pod detects database connection issue
// 2. Health check returns 503
// 3. Load balancer stops sending traffic to this pod
// 4. Other healthy pods handle requests
// 5. Kubernetes restarts unhealthy pod
// 6. Pod recovers, health check returns 200
// 7. Traffic resumes
// 8. 0 downtime for users
```

---

### **Kubernetes Health Check Configuration:**

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nestjs-app
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: app
        image: my-nestjs-app:latest
        ports:
        - containerPort: 3000
        
        # Liveness probe: Is the app alive?
        livenessProbe:
          httpGet:
            path: /health/live
            port: 3000
          initialDelaySeconds: 30  # Wait 30s before first check
          periodSeconds: 10        # Check every 10s
          timeoutSeconds: 5        # Timeout after 5s
          failureThreshold: 3      # Restart after 3 failures
        
        # Readiness probe: Is the app ready for traffic?
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 2      # Remove from service after 2 failures
        
        # Startup probe: Has the app finished starting?
        startupProbe:
          httpGet:
            path: /health/startup
            port: 3000
          initialDelaySeconds: 0
          periodSeconds: 5
          failureThreshold: 30     # Allow 150s (30 * 5) for startup
```

---

### **AWS Load Balancer Health Check:**

```typescript
// Target Group Health Check Settings:
{
  "HealthCheckProtocol": "HTTP",
  "HealthCheckPath": "/health",
  "HealthCheckPort": "3000",
  "HealthCheckIntervalSeconds": 30,
  "HealthCheckTimeoutSeconds": 5,
  "HealthyThresholdCount": 2,      // Healthy after 2 successes
  "UnhealthyThresholdCount": 3,    // Unhealthy after 3 failures
  "Matcher": {
    "HttpCode": "200"  // Success status code
  }
}

// AWS will:
// ✅ Route traffic only to healthy targets
// ✅ Automatically deregister unhealthy targets
// ✅ Re-register when health check passes
```

---

### **Docker Swarm Health Check:**

```dockerfile
# Dockerfile
FROM node:18-alpine

WORKDIR /app
COPY package*.json ./
RUN npm ci --production
COPY . .

# Health check configuration
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD node healthcheck.js || exit 1

EXPOSE 3000
CMD ["node", "dist/main.js"]
```

```typescript
// healthcheck.js
const http = require('http');

const options = {
  host: 'localhost',
  port: 3000,
  path: '/health',
  timeout: 2000,
};

const request = http.request(options, (res) => {
  if (res.statusCode === 200) {
    process.exit(0);  // Healthy
  } else {
    process.exit(1);  // Unhealthy
  }
});

request.on('error', () => {
  process.exit(1);  // Unhealthy
});

request.end();
```

---

### **Monitoring Health Checks:**

```typescript
// Prometheus metrics for health checks
import * as client from 'prom-client';

const healthCheckGauge = new client.Gauge({
  name: 'health_check_status',
  help: 'Health check status (1 = healthy, 0 = unhealthy)',
  labelNames: ['check_name'],
});

// Update metrics
healthCheckGauge.set({ check_name: 'database' }, 1);  // Healthy
healthCheckGauge.set({ check_name: 'redis' }, 0);     // Unhealthy

// Grafana dashboard shows:
// - Database: ✅ Healthy
// - Redis: ❌ Unhealthy
```

---

### **Common Health Check Mistakes:**

```typescript
// ❌ BAD - Too frequent checks
periodSeconds: 1  // Checking every second creates load

// ✅ GOOD - Reasonable interval
periodSeconds: 10  // Every 10 seconds is sufficient

// ❌ BAD - No timeout
// Health check hangs forever

// ✅ GOOD - Set timeout
timeoutSeconds: 5  // Fail after 5 seconds

// ❌ BAD - Same endpoint for liveness and readiness
livenessProbe: /health
readinessProbe: /health  // Both check same thing

// ✅ GOOD - Separate endpoints
livenessProbe: /health/live   // Checks app is alive
readinessProbe: /health/ready // Checks dependencies

// ❌ BAD - Health check depends on external service
@Get('health')
async check() {
  await externalAPI.call();  // External failure = health check fails
}

// ✅ GOOD - Liveness only checks app itself
@Get('health/live')
check() {
  return { status: 'ok' };  // Just check app is running
}

// ❌ BAD - No initial delay
initialDelaySeconds: 0  // Check before app ready

// ✅ GOOD - Allow time to start
initialDelaySeconds: 30  // Wait for app initialization
```

---

### **Health Check Best Practices:**

```typescript
✅ Use separate endpoints for liveness and readiness
✅ Liveness: Only check app health (fast, simple)
✅ Readiness: Check dependencies (database, cache, APIs)
✅ Return proper HTTP status codes (200 = healthy, 503 = unhealthy)
✅ Include meaningful error messages
✅ Set reasonable timeouts (3-5 seconds)
✅ Don't make health checks too expensive
✅ Cache dependency checks (don't check DB on every request)
✅ Include version info in health response
✅ Monitor health check failures in metrics
```

**Key Takeaway:** Health checks are **HTTP endpoints** that report application status for **orchestrators** (Kubernetes) and **load balancers**. They enable **automatic failover**, **self-healing**, **zero-downtime deployments**, and **dependency monitoring**. Use **liveness** (is app alive?) and **readiness** (ready for traffic?) probes.

</details>

<details>
<summary><strong>32. How do you implement health check endpoints using `@nestjs/terminus`?</strong></summary>

**Answer:**

`@nestjs/terminus` is NestJS's official library for implementing health checks. It provides decorators, health indicators (database, HTTP, disk, memory), and integrations with orchestrators like Kubernetes. It follows industry standards and supports custom health indicators.

---

### **Method 1: Basic Setup**

```typescript
// Install
npm install @nestjs/terminus @nestjs/axios axios

// health/health.module.ts
import { Module } from '@nestjs/common';
import { TerminusModule } from '@nestjs/terminus';
import { HttpModule } from '@nestjs/axios';
import { HealthController } from './health.controller';

@Module({
  imports: [TerminusModule, HttpModule],
  controllers: [HealthController],
})
export class HealthModule {}

// health/health.controller.ts
import { Controller, Get } from '@nestjs/common';
import { HealthCheck, HealthCheckService, HttpHealthIndicator } from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private http: HttpHealthIndicator,
  ) {}

  @Get()
  @HealthCheck()
  check() {
    return this.health.check([
      () => this.http.pingCheck('nestjs-docs', 'https://docs.nestjs.com'),
    ]);
  }
}

// GET /health
{
  "status": "ok",
  "info": {
    "nestjs-docs": {
      "status": "up"
    }
  },
  "error": {},
  "details": {
    "nestjs-docs": {
      "status": "up"
    }
  }
}

// app.module.ts
import { Module } from '@nestjs/common';
import { HealthModule } from './health/health.module';

@Module({
  imports: [HealthModule],
})
export class AppModule {}
```

---

### **Method 2: Database Health Check**

```typescript
// Install TypeORM health indicator
npm install @nestjs/typeorm typeorm

// health/health.controller.ts
import { Controller, Get } from '@nestjs/common';
import {
  HealthCheck,
  HealthCheckService,
  TypeOrmHealthIndicator,
} from '@nestjs/terminus';

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

// Success response (200):
{
  "status": "ok",
  "info": {
    "database": {
      "status": "up"
    }
  },
  "error": {},
  "details": {
    "database": {
      "status": "up"
    }
  }
}

// Failure response (503):
{
  "status": "error",
  "info": {},
  "error": {
    "database": {
      "status": "down",
      "message": "Connection timeout"
    }
  },
  "details": {
    "database": {
      "status": "down",
      "message": "Connection timeout"
    }
  }
}
```

---

### **Method 3: Multiple Health Indicators**

```typescript
// health/health.controller.ts
import { Controller, Get } from '@nestjs/common';
import {
  HealthCheck,
  HealthCheckService,
  TypeOrmHealthIndicator,
  HttpHealthIndicator,
  DiskHealthIndicator,
  MemoryHealthIndicator,
} from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private db: TypeOrmHealthIndicator,
    private http: HttpHealthIndicator,
    private disk: DiskHealthIndicator,
    private memory: MemoryHealthIndicator,
  ) {}

  @Get()
  @HealthCheck()
  check() {
    return this.health.check([
      // Database check
      () => this.db.pingCheck('database'),
      
      // External API check
      () => this.http.pingCheck('payment-api', 'https://api.stripe.com'),
      
      // Disk space check (must have >50% free space)
      () => this.disk.checkStorage('storage', {
        path: '/',
        thresholdPercent: 0.5,  // 50%
      }),
      
      // Memory check (heap must be <300MB)
      () => this.memory.checkHeap('memory_heap', 300 * 1024 * 1024),  // 300MB
      
      // RSS memory check (must be <500MB)
      () => this.memory.checkRSS('memory_rss', 500 * 1024 * 1024),  // 500MB
    ]);
  }
}

// Response:
{
  "status": "ok",
  "info": {
    "database": { "status": "up" },
    "payment-api": { "status": "up" },
    "storage": { "status": "up" },
    "memory_heap": { "status": "up" },
    "memory_rss": { "status": "up" }
  },
  "error": {},
  "details": { /* ... */ }
}
```

---

### **Method 4: Liveness vs Readiness Probes**

```typescript
// health/health.controller.ts
import { Controller, Get } from '@nestjs/common';
import {
  HealthCheck,
  HealthCheckService,
  TypeOrmHealthIndicator,
  HttpHealthIndicator,
} from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private db: TypeOrmHealthIndicator,
    private http: HttpHealthIndicator,
  ) {}

  // Liveness probe: Is the app alive?
  // Used by Kubernetes to determine if pod should be restarted
  @Get('live')
  @HealthCheck()
  checkLive() {
    // Only check if app is running (no external dependencies)
    return this.health.check([
      // Simple check that always passes if app is running
      () => Promise.resolve({ app: { status: 'up' } }),
    ]);
  }

  // Readiness probe: Is the app ready to receive traffic?
  // Used by Kubernetes to determine if pod should receive traffic
  @Get('ready')
  @HealthCheck()
  checkReady() {
    // Check all dependencies
    return this.health.check([
      () => this.db.pingCheck('database'),
      () => this.http.pingCheck('auth-service', 'http://auth:3001/health'),
      () => this.http.pingCheck('payment-api', 'http://payment:3002/health'),
    ]);
  }

  // Startup probe: Has the app finished starting?
  @Get('startup')
  @HealthCheck()
  checkStartup() {
    // Check if initialization is complete
    return this.health.check([
      () => this.db.pingCheck('database'),
      // Check if cache is warmed up
      () => this.checkCacheWarming(),
    ]);
  }

  private async checkCacheWarming() {
    const isWarmed = await this.cacheService.isWarmed();
    if (isWarmed) {
      return { cache: { status: 'up' } };
    }
    throw new Error('Cache warming in progress');
  }
}
```

---

### **Method 5: Custom Health Indicator**

```typescript
// health/indicators/redis.health.ts
import { Injectable } from '@nestjs/common';
import { HealthIndicator, HealthIndicatorResult, HealthCheckError } from '@nestjs/terminus';
import { InjectRedis } from '@liaoliaots/nestjs-redis';
import { Redis } from 'ioredis';

@Injectable()
export class RedisHealthIndicator extends HealthIndicator {
  constructor(@InjectRedis() private readonly redis: Redis) {
    super();
  }

  async isHealthy(key: string): Promise<HealthIndicatorResult> {
    try {
      // Ping Redis
      const result = await this.redis.ping();
      
      if (result === 'PONG') {
        return this.getStatus(key, true, { message: 'Redis is healthy' });
      }
      
      throw new Error('Redis ping failed');
    } catch (error) {
      throw new HealthCheckError(
        'Redis check failed',
        this.getStatus(key, false, { message: error.message }),
      );
    }
  }
}

// health/indicators/queue.health.ts
import { Injectable } from '@nestjs/common';
import { HealthIndicator, HealthIndicatorResult, HealthCheckError } from '@nestjs/terminus';
import { InjectQueue } from '@nestjs/bull';
import { Queue } from 'bull';

@Injectable()
export class QueueHealthIndicator extends HealthIndicator {
  constructor(@InjectQueue('default') private readonly queue: Queue) {
    super();
  }

  async isHealthy(key: string): Promise<HealthIndicatorResult> {
    try {
      // Check queue connection
      const jobCounts = await this.queue.getJobCounts();
      
      // Alert if too many failed jobs
      if (jobCounts.failed > 100) {
        throw new Error(`Too many failed jobs: ${jobCounts.failed}`);
      }
      
      return this.getStatus(key, true, { jobCounts });
    } catch (error) {
      throw new HealthCheckError(
        'Queue check failed',
        this.getStatus(key, false, { message: error.message }),
      );
    }
  }
}

// health/health.controller.ts
import { Controller, Get } from '@nestjs/common';
import { HealthCheck, HealthCheckService } from '@nestjs/terminus';
import { RedisHealthIndicator } from './indicators/redis.health';
import { QueueHealthIndicator } from './indicators/queue.health';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private redisHealth: RedisHealthIndicator,
    private queueHealth: QueueHealthIndicator,
  ) {}

  @Get()
  @HealthCheck()
  check() {
    return this.health.check([
      () => this.redisHealth.isHealthy('redis'),
      () => this.queueHealth.isHealthy('queue'),
    ]);
  }
}
```

---

### **Method 6: Microservices Health Check**

```typescript
// health/indicators/microservice.health.ts
import { Injectable } from '@nestjs/common';
import { HealthIndicator, HealthIndicatorResult, HealthCheckError } from '@nestjs/terminus';
import { HttpService } from '@nestjs/axios';
import { firstValueFrom } from 'rxjs';

@Injectable()
export class MicroserviceHealthIndicator extends HealthIndicator {
  constructor(private readonly http: HttpService) {
    super();
  }

  async isHealthy(key: string, url: string): Promise<HealthIndicatorResult> {
    try {
      const response = await firstValueFrom(
        this.http.get(url, { timeout: 3000 }),
      );
      
      return this.getStatus(key, true, {
        statusCode: response.status,
        responseTime: response.headers['x-response-time'],
      });
    } catch (error) {
      throw new HealthCheckError(
        `${key} health check failed`,
        this.getStatus(key, false, { message: error.message }),
      );
    }
  }
}

// health/health.controller.ts
@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private microserviceHealth: MicroserviceHealthIndicator,
  ) {}

  @Get('ready')
  @HealthCheck()
  checkReady() {
    return this.health.check([
      () => this.microserviceHealth.isHealthy('user-service', 'http://user-service:3001/health'),
      () => this.microserviceHealth.isHealthy('order-service', 'http://order-service:3002/health'),
      () => this.microserviceHealth.isHealthy('payment-service', 'http://payment-service:3003/health'),
    ]);
  }
}
```

---

### **Method 7: Graceful Shutdown**

```typescript
// health/health.service.ts
import { Injectable, OnModuleDestroy } from '@nestjs/common';

@Injectable()
export class HealthService implements OnModuleDestroy {
  private isShuttingDown = false;

  onModuleDestroy() {
    this.isShuttingDown = true;
  }

  isHealthy(): boolean {
    return !this.isShuttingDown;
  }
}

// health/health.controller.ts
import { Controller, Get, ServiceUnavailableException } from '@nestjs/common';
import { HealthCheck, HealthCheckService } from '@nestjs/terminus';
import { HealthService } from './health.service';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private healthService: HealthService,
  ) {}

  @Get('ready')
  @HealthCheck()
  checkReady() {
    // Return unhealthy during shutdown
    if (!this.healthService.isHealthy()) {
      throw new ServiceUnavailableException('Application is shutting down');
    }
    
    return this.health.check([
      // ... other checks
    ]);
  }
}

// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Enable graceful shutdown
  app.enableShutdownHooks();
  
  // Handle SIGTERM (Kubernetes sends this before killing pod)
  process.on('SIGTERM', async () => {
    console.log('SIGTERM received, starting graceful shutdown');
    
    // Wait for in-flight requests to complete
    setTimeout(async () => {
      await app.close();
      process.exit(0);
    }, 10000);  // 10 second grace period
  });
  
  await app.listen(3000);
}
bootstrap();
```

---

### **Method 8: Cached Health Checks**

```typescript
// health/cached-health.service.ts
import { Injectable } from '@nestjs/common';

interface CachedResult {
  status: string;
  timestamp: Date;
}

@Injectable()
export class CachedHealthService {
  private cache: Map<string, CachedResult> = new Map();
  private readonly CACHE_TTL = 10000;  // 10 seconds

  async checkWithCache(
    key: string,
    checkFn: () => Promise<any>,
  ): Promise<any> {
    const cached = this.cache.get(key);
    const now = Date.now();

    // Return cached result if still valid
    if (cached && now - cached.timestamp.getTime() < this.CACHE_TTL) {
      return cached.status;
    }

    // Perform actual check
    try {
      const result = await checkFn();
      this.cache.set(key, { status: result, timestamp: new Date() });
      return result;
    } catch (error) {
      // Return last known good state if check fails
      if (cached) {
        return cached.status;
      }
      throw error;
    }
  }
}

// health/health.controller.ts
@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private cachedHealth: CachedHealthService,
    private db: TypeOrmHealthIndicator,
  ) {}

  @Get()
  @HealthCheck()
  check() {
    return this.health.check([
      // Cache database check for 10 seconds
      () => this.cachedHealth.checkWithCache('database', () =>
        this.db.pingCheck('database'),
      ),
    ]);
  }
}
```

---

### **Method 9: Detailed Health Response**

```typescript
// health/health.controller.ts
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
  async check() {
    const healthCheck = await this.health.check([
      () => this.db.pingCheck('database'),
    ]);

    // Add custom metadata
    return {
      ...healthCheck,
      timestamp: new Date().toISOString(),
      uptime: process.uptime(),
      version: process.env.APP_VERSION || '1.0.0',
      environment: process.env.NODE_ENV,
      hostname: require('os').hostname(),
      memory: {
        heapUsed: process.memoryUsage().heapUsed,
        heapTotal: process.memoryUsage().heapTotal,
        rss: process.memoryUsage().rss,
      },
    };
  }
}

// Response:
{
  "status": "ok",
  "info": { "database": { "status": "up" } },
  "error": {},
  "details": { "database": { "status": "up" } },
  "timestamp": "2025-12-30T10:30:45.123Z",
  "uptime": 3600.5,
  "version": "1.0.0",
  "environment": "production",
  "hostname": "api-pod-abc123",
  "memory": {
    "heapUsed": 52428800,
    "heapTotal": 104857600,
    "rss": 157286400
  }
}
```

---

### **Method 10: Complete Production Setup**

```typescript
// health/health.module.ts
import { Module } from '@nestjs/common';
import { TerminusModule } from '@nestjs/terminus';
import { HttpModule } from '@nestjs/axios';
import { TypeOrmModule } from '@nestjs/typeorm';
import { HealthController } from './health.controller';
import { RedisHealthIndicator } from './indicators/redis.health';
import { QueueHealthIndicator } from './indicators/queue.health';
import { MicroserviceHealthIndicator } from './indicators/microservice.health';
import { CachedHealthService } from './cached-health.service';

@Module({
  imports: [
    TerminusModule,
    HttpModule,
    TypeOrmModule,
  ],
  controllers: [HealthController],
  providers: [
    RedisHealthIndicator,
    QueueHealthIndicator,
    MicroserviceHealthIndicator,
    CachedHealthService,
  ],
})
export class HealthModule {}
```

---

### **Best Practices:**

```typescript
// ✅ GOOD - Separate liveness and readiness
@Get('live')  // Simple, always returns 200 if app running
@Get('ready')  // Checks dependencies

// ✅ GOOD - Cache expensive checks
this.cachedHealth.checkWithCache('database', checkFn)

// ✅ GOOD - Set timeouts
this.http.get(url, { timeout: 3000 })

// ✅ GOOD - Return structured response
{ status: 'ok', info: {...}, error: {...} }

// ✅ GOOD - Include metadata
version, uptime, hostname, memory

// ❌ BAD - Same endpoint for liveness and readiness
@Get('health')  // Can't distinguish purposes

// ❌ BAD - No timeout on checks
await externalAPI.call();  // Could hang forever

// ❌ BAD - Too many dependencies in liveness
// Liveness should only check app, not dependencies

// ❌ BAD - No caching on frequent checks
// Database pinged every 5 seconds creates load
```

**Key Takeaway:** Use `@nestjs/terminus` to implement health checks with **built-in indicators** (database, HTTP, disk, memory), **custom indicators** for Redis/queues, **separate endpoints** for liveness (/health/live) and readiness (/health/ready), and **caching** to reduce check frequency and load.

</details>

<details>
<summary><strong>33. What is liveness vs readiness probe?</strong></summary>

**Answer:**

**Liveness probes** check if the application is **alive** (if not, restart it). **Readiness probes** check if the application is **ready to serve traffic** (if not, stop sending traffic but don't restart). They serve different purposes in Kubernetes and have different failure actions.

---

### **Comparison Table:**

| Aspect | Liveness Probe | Readiness Probe |
|--------|----------------|------------------|
| **Purpose** | Is the app alive? | Is the app ready for traffic? |
| **Failure Action** | Restart the pod | Stop sending traffic (don't restart) |
| **Checks** | App is not deadlocked/hung | Dependencies are available |
| **Scope** | Application health only | Application + dependencies |
| **Speed** | Fast (<1s preferred) | Can be slower (1-3s) |
| **Endpoint** | `/health/live` | `/health/ready` |
| **Example Failures** | Deadlock, infinite loop, OOM | Database down, cache unavailable |
| **Kubernetes Action** | `kubelet` restarts container | Removes pod from Service endpoints |
| **During Startup** | Should pass quickly | Can fail until dependencies ready |
| **During Shutdown** | Should still pass | Should fail immediately |

---

### **Liveness Probe (Is the app alive?)**

```typescript
// health/health.controller.ts
import { Controller, Get } from '@nestjs/common';
import { HealthCheck, HealthCheckService } from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(private health: HealthCheckService) {}

  // Liveness: Only checks if the app itself is running
  @Get('live')
  @HealthCheck()
  checkLive() {
    return this.health.check([
      // Simple check: if this endpoint responds, app is alive
      () => Promise.resolve({ app: { status: 'up' } }),
    ]);
  }
}

// Kubernetes configuration
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: app
    image: my-app:latest
    livenessProbe:
      httpGet:
        path: /health/live
        port: 3000
      initialDelaySeconds: 30    # Wait 30s before first check
      periodSeconds: 10           # Check every 10s
      timeoutSeconds: 5           # Timeout after 5s
      failureThreshold: 3         # Restart after 3 consecutive failures

// What liveness checks:
✅ Application is running
✅ No deadlock
✅ No infinite loop
✅ Event loop not blocked

// What liveness DOES NOT check:
❌ Database connection
❌ External API availability
❌ Cache availability
❌ Disk space

// When liveness fails → Kubernetes restarts the pod
```

---

### **Readiness Probe (Ready for traffic?)**

```typescript
// health/health.controller.ts
import { Controller, Get } from '@nestjs/common';
import {
  HealthCheck,
  HealthCheckService,
  TypeOrmHealthIndicator,
  HttpHealthIndicator,
} from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private db: TypeOrmHealthIndicator,
    private http: HttpHealthIndicator,
  ) {}

  // Readiness: Checks if app + dependencies are ready
  @Get('ready')
  @HealthCheck()
  checkReady() {
    return this.health.check([
      // Check database
      () => this.db.pingCheck('database'),
      
      // Check Redis
      () => this.http.pingCheck('redis', 'http://redis:6379/ping'),
      
      // Check external auth service
      () => this.http.pingCheck('auth-service', 'http://auth:3001/health'),
    ]);
  }
}

// Kubernetes configuration
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: app
    image: my-app:latest
    readinessProbe:
      httpGet:
        path: /health/ready
        port: 3000
      initialDelaySeconds: 10    # Wait 10s before first check
      periodSeconds: 5            # Check every 5s
      timeoutSeconds: 3           # Timeout after 3s
      failureThreshold: 2         # Mark unready after 2 failures
      successThreshold: 1         # Mark ready after 1 success

// What readiness checks:
✅ Database is reachable
✅ Cache is available
✅ External APIs are responding
✅ Required services are up

// When readiness fails → Pod removed from Service (no restart)
```

---

### **Real-World Scenario:**

```typescript
// Scenario: Database connection pool exhausted

// Without separate probes:
// 1. Readiness check includes database
// 2. Database fails
// 3. Pod marked unhealthy
// 4. Kubernetes RESTARTS the pod
// 5. Problem: Restarting won't fix database issue
// 6. Pod keeps restarting (CrashLoopBackOff)

// With separate probes:
// 1. Liveness: App is alive ✅ (no restart)
// 2. Readiness: Database down ❌ (stop traffic)
// 3. Kubernetes stops sending traffic to this pod
// 4. Other healthy pods handle requests
// 5. Database recovers
// 6. Readiness check passes ✅
// 7. Traffic resumes to this pod
// 8. Zero downtime!
```

---

### **Complete Example:**

```typescript
// health/health.controller.ts
import { Controller, Get } from '@nestjs/common';
import {
  HealthCheck,
  HealthCheckService,
  TypeOrmHealthIndicator,
  MemoryHealthIndicator,
} from '@nestjs/terminus';
import { RedisHealthIndicator } from './indicators/redis.health';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private db: TypeOrmHealthIndicator,
    private memory: MemoryHealthIndicator,
    private redis: RedisHealthIndicator,
  ) {}

  // Liveness: Simple, fast check
  @Get('live')
  @HealthCheck()
  checkLive() {
    return this.health.check([
      // Only check memory (indicator of potential OOM)
      () => this.memory.checkHeap('memory_heap', 1024 * 1024 * 1024), // 1GB
    ]);
  }

  // Readiness: Comprehensive dependency check
  @Get('ready')
  @HealthCheck()
  checkReady() {
    return this.health.check([
      () => this.db.pingCheck('database'),
      () => this.redis.isHealthy('redis'),
    ]);
  }
}
```

---

### **Kubernetes Deployment Example:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nestjs-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nestjs-app
  template:
    metadata:
      labels:
        app: nestjs-app
    spec:
      containers:
      - name: app
        image: nestjs-app:1.0.0
        ports:
        - containerPort: 3000
        
        # Startup probe: Has the app finished starting?
        # Disabled liveness/readiness until this passes
        startupProbe:
          httpGet:
            path: /health/startup
            port: 3000
          initialDelaySeconds: 0
          periodSeconds: 5
          failureThreshold: 30       # 30 * 5 = 150s startup time allowed
        
        # Liveness probe: Is the app alive?
        livenessProbe:
          httpGet:
            path: /health/live
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3        # Restart after 3 failures (30s)
        
        # Readiness probe: Ready for traffic?
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 2        # Unready after 2 failures (10s)
          successThreshold: 1        # Ready after 1 success
```

---

### **Failure Actions:**

```typescript
// Liveness Probe Failure:
// Time: 0s - Pod starts
// Time: 30s - First liveness check (initialDelaySeconds: 30)
// Time: 40s - Second liveness check
// Time: 50s - Third liveness check FAILS
// Time: 60s - Fourth liveness check FAILS
// Time: 70s - Fifth liveness check FAILS (3 consecutive failures)
// Action: Kubernetes RESTARTS the container

// Readiness Probe Failure:
// Time: 0s - Pod starts
// Time: 10s - First readiness check (initialDelaySeconds: 10)
// Time: 15s - Second readiness check PASSES
// Pod is now receiving traffic ✅
// Time: 20s - Readiness check FAILS (database down)
// Time: 25s - Readiness check FAILS (2 consecutive failures)
// Action: Pod REMOVED from Service endpoints (no traffic)
// Container keeps running (no restart)
// Time: 30s - Database recovers
// Time: 35s - Readiness check PASSES
// Action: Pod ADDED back to Service endpoints
// Traffic resumes ✅
```

---

### **Use Cases:**

#### **Liveness Probe Use Cases:**
```typescript
// 1. Deadlock detection
if (applicationDeadlocked) {
  // Liveness fails → Restart needed
}

// 2. Memory leak causing OOM
if (memoryUsage > 1GB) {
  // Liveness fails → Restart to free memory
}

// 3. Infinite loop
if (eventLoopBlocked) {
  // Liveness fails → Restart to recover
}

// 4. Unrecoverable error
try {
  criticalOperation();
} catch (error) {
  if (error.unrecoverable) {
    // Liveness fails → Restart
  }
}
```

#### **Readiness Probe Use Cases:**
```typescript
// 1. Database migration in progress
if (migrationRunning) {
  // Readiness fails → Don't send traffic
  // Don't restart (migration needs to complete)
}

// 2. Cache warming
if (!cacheWarmed) {
  // Readiness fails → Wait until cache ready
}

// 3. Dependency service down
if (!authServiceAvailable) {
  // Readiness fails → Can't process requests
  // Don't restart (won't help)
}

// 4. Graceful shutdown
process.on('SIGTERM', () => {
  // Readiness fails immediately → Stop new traffic
  // Finish existing requests
  // Then shutdown
});
```

---

### **Common Mistakes:**

```typescript
// ❌ BAD - Same endpoint for both probes
livenessProbe:
  httpGet:
    path: /health
readinessProbe:
  httpGet:
    path: /health  // Same as liveness!

// Problem: Can't distinguish purposes
// If database fails, pod restarts (wrong action)

// ✅ GOOD - Separate endpoints
livenessProbe:
  httpGet:
    path: /health/live   // Only checks app
readinessProbe:
  httpGet:
    path: /health/ready  // Checks dependencies

// ❌ BAD - Liveness checks database
@Get('live')
checkLive() {
  return this.health.check([
    () => this.db.pingCheck('database'),  // ❌ Wrong!
  ]);
}

// Problem: Database down → Pod restarts (won't fix database)

// ✅ GOOD - Liveness only checks app
@Get('live')
checkLive() {
  return this.health.check([
    () => Promise.resolve({ app: { status: 'up' } }),
  ]);
}

// ❌ BAD - No initial delay
livenessProbe:
  initialDelaySeconds: 0  // Check immediately

// Problem: App not ready yet → Immediate failure → Restart loop

// ✅ GOOD - Allow time to start
livenessProbe:
  initialDelaySeconds: 30  // Wait for app to start

// ❌ BAD - Too aggressive failure threshold
livenessProbe:
  failureThreshold: 1  // Restart after 1 failure

// Problem: Temporary network glitch → Unnecessary restart

// ✅ GOOD - Allow some failures
livenessProbe:
  failureThreshold: 3  // Restart after 3 consecutive failures
```

---

### **Decision Tree:**

```
Should this check be in liveness or readiness?

"If this check fails, would restarting the pod fix it?"
    |
    ├─ YES → Liveness probe
    |   Examples:
    |   • Memory leak (restart frees memory)
    |   • Deadlock (restart breaks deadlock)
    |   • Corrupted state (restart resets state)
    |
    └─ NO → Readiness probe
        Examples:
        • Database down (restart won't help)
        • External API down (restart won't help)
        • Dependency service unavailable (restart won't help)
```

---

### **Best Practices:**

```typescript
// ✅ GOOD - Liveness: Fast, simple
@Get('live')
checkLive() {
  return { status: 'ok' };
}

// ✅ GOOD - Readiness: Comprehensive
@Get('ready')
checkReady() {
  return this.health.check([
    () => this.db.pingCheck('database'),
    () => this.redis.isHealthy('redis'),
  ]);
}

// ✅ GOOD - Different failure thresholds
livenessProbe:
  failureThreshold: 3  // More tolerant (restart is expensive)
readinessProbe:
  failureThreshold: 2  // Less tolerant (stop traffic quickly)

// ✅ GOOD - Different check intervals
livenessProbe:
  periodSeconds: 10  // Less frequent (restart is rare)
readinessProbe:
  periodSeconds: 5   // More frequent (traffic control is common)

// ✅ GOOD - Graceful shutdown
process.on('SIGTERM', () => {
  // Mark as unready immediately
  this.isReady = false;
  
  // Allow time for in-flight requests
  setTimeout(() => {
    process.exit(0);
  }, 10000);
});
```

---

### **Monitoring Probe Failures:**

```typescript
// Log probe failures for debugging
import { Logger } from '@nestjs/common';

@Controller('health')
export class HealthController {
  private readonly logger = new Logger('HealthCheck');

  @Get('ready')
  @HealthCheck()
  async checkReady() {
    try {
      const result = await this.health.check([
        () => this.db.pingCheck('database'),
      ]);
      return result;
    } catch (error) {
      this.logger.error('Readiness check failed', error);
      throw error;
    }
  }
}

// Prometheus metrics
const healthCheckFailures = new client.Counter({
  name: 'health_check_failures_total',
  help: 'Total number of health check failures',
  labelNames: ['probe_type', 'check_name'],
});

healthCheckFailures.inc({ probe_type: 'readiness', check_name: 'database' });
```

**Key Takeaway:** **Liveness probes** check if the app is alive (restart if fails), **readiness probes** check if ready for traffic (stop traffic if fails, no restart). Use **separate endpoints**: liveness for app health only (fast), readiness for dependencies (comprehensive). Liveness failure → restart pod, readiness failure → remove from service.

</details>

<details>
<summary><strong>34. How do you check database health?</strong></summary>

**Answer:**

Database health checks verify that the application can **connect** to the database, **execute queries**, and **maintain connection pools**. Use `@nestjs/terminus` with TypeORM/Sequelize health indicators, custom queries, connection pool monitoring, and timeout handling.

---

### **Method 1: TypeORM Health Indicator**

```typescript
// Install
npm install @nestjs/terminus @nestjs/typeorm typeorm

// health/health.controller.ts
import { Controller, Get } from '@nestjs/common';
import {
  HealthCheck,
  HealthCheckService,
  TypeOrmHealthIndicator,
} from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private db: TypeOrmHealthIndicator,
  ) {}

  @Get('db')
  @HealthCheck()
  checkDatabase() {
    return this.health.check([
      () => this.db.pingCheck('database'),
    ]);
  }
}

// Success (200):
{
  "status": "ok",
  "info": {
    "database": {
      "status": "up"
    }
  },
  "error": {},
  "details": {
    "database": {
      "status": "up"
    }
  }
}

// Failure (503):
{
  "status": "error",
  "info": {},
  "error": {
    "database": {
      "status": "down",
      "message": "Connection timeout"
    }
  },
  "details": {
    "database": {
      "status": "down",
      "message": "Connection timeout"
    }
  }
}
```

---

### **Method 2: Multiple Database Connections**

```typescript
// app.module.ts - Multiple databases
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    // Primary database
    TypeOrmModule.forRoot({
      name: 'default',
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      database: 'main_db',
    }),
    
    // Analytics database
    TypeOrmModule.forRoot({
      name: 'analytics',
      type: 'postgres',
      host: 'analytics-db',
      port: 5432,
      database: 'analytics_db',
    }),
  ],
})
export class AppModule {}

// health/health.controller.ts
import { Controller, Get } from '@nestjs/common';
import {
  HealthCheck,
  HealthCheckService,
  TypeOrmHealthIndicator,
} from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private db: TypeOrmHealthIndicator,
  ) {}

  @Get('db')
  @HealthCheck()
  checkDatabase() {
    return this.health.check([
      // Check primary database
      () => this.db.pingCheck('database', { connection: 'default' }),
      
      // Check analytics database
      () => this.db.pingCheck('analytics', { connection: 'analytics' }),
    ]);
  }
}

// Response:
{
  "status": "ok",
  "info": {
    "database": { "status": "up" },
    "analytics": { "status": "up" }
  }
}
```

---

### **Method 3: Custom Database Health Check**

```typescript
// health/indicators/database.health.ts
import { Injectable } from '@nestjs/common';
import {
  HealthIndicator,
  HealthIndicatorResult,
  HealthCheckError,
} from '@nestjs/terminus';
import { InjectDataSource } from '@nestjs/typeorm';
import { DataSource } from 'typeorm';

@Injectable()
export class DatabaseHealthIndicator extends HealthIndicator {
  constructor(@InjectDataSource() private dataSource: DataSource) {
    super();
  }

  async isHealthy(key: string): Promise<HealthIndicatorResult> {
    try {
      // Execute simple query
      const result = await this.dataSource.query('SELECT 1');
      
      if (result) {
        return this.getStatus(key, true, {
          message: 'Database is healthy',
        });
      }
      
      throw new Error('Query failed');
    } catch (error) {
      throw new HealthCheckError(
        'Database check failed',
        this.getStatus(key, false, {
          message: error.message,
        }),
      );
    }
  }

  async checkWithQuery(key: string, query: string): Promise<HealthIndicatorResult> {
    const start = Date.now();
    
    try {
      await this.dataSource.query(query);
      const duration = Date.now() - start;
      
      return this.getStatus(key, true, {
        duration: `${duration}ms`,
      });
    } catch (error) {
      throw new HealthCheckError(
        'Database query failed',
        this.getStatus(key, false, {
          message: error.message,
        }),
      );
    }
  }
}

// health/health.controller.ts
import { Controller, Get } from '@nestjs/common';
import { HealthCheck, HealthCheckService } from '@nestjs/terminus';
import { DatabaseHealthIndicator } from './indicators/database.health';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private dbHealth: DatabaseHealthIndicator,
  ) {}

  @Get('db')
  @HealthCheck()
  checkDatabase() {
    return this.health.check([
      () => this.dbHealth.isHealthy('database'),
    ]);
  }

  @Get('db/detailed')
  @HealthCheck()
  checkDatabaseDetailed() {
    return this.health.check([
      // Check with custom query
      () => this.dbHealth.checkWithQuery('users_table', 'SELECT COUNT(*) FROM users'),
    ]);
  }
}
```

---

### **Method 4: Connection Pool Health**

```typescript
// health/indicators/connection-pool.health.ts
import { Injectable } from '@nestjs/common';
import {
  HealthIndicator,
  HealthIndicatorResult,
  HealthCheckError,
} from '@nestjs/terminus';
import { InjectDataSource } from '@nestjs/typeorm';
import { DataSource } from 'typeorm';

@Injectable()
export class ConnectionPoolHealthIndicator extends HealthIndicator {
  constructor(@InjectDataSource() private dataSource: DataSource) {
    super();
  }

  async isHealthy(key: string): Promise<HealthIndicatorResult> {
    try {
      // Check if dataSource is initialized
      if (!this.dataSource.isInitialized) {
        throw new Error('DataSource not initialized');
      }

      // Get connection pool stats
      const driver = this.dataSource.driver as any;
      const pool = driver.master;  // Connection pool
      
      const stats = {
        totalCount: pool.totalCount || 0,
        idleCount: pool.idleCount || 0,
        waitingCount: pool.waitingCount || 0,
      };

      // Alert if pool is exhausted
      if (stats.waitingCount > 10) {
        throw new Error(`Too many waiting connections: ${stats.waitingCount}`);
      }

      return this.getStatus(key, true, stats);
    } catch (error) {
      throw new HealthCheckError(
        'Connection pool check failed',
        this.getStatus(key, false, {
          message: error.message,
        }),
      );
    }
  }
}

// Response:
{
  "status": "ok",
  "info": {
    "connection_pool": {
      "status": "up",
      "totalCount": 10,
      "idleCount": 8,
      "waitingCount": 0
    }
  }
}
```

---

### **Method 5: Sequelize Health Indicator**

```typescript
// Install
npm install @nestjs/sequelize sequelize sequelize-typescript

// health/indicators/sequelize.health.ts
import { Injectable } from '@nestjs/common';
import {
  HealthIndicator,
  HealthIndicatorResult,
  HealthCheckError,
} from '@nestjs/terminus';
import { InjectConnection } from '@nestjs/sequelize';
import { Sequelize } from 'sequelize-typescript';

@Injectable()
export class SequelizeHealthIndicator extends HealthIndicator {
  constructor(@InjectConnection() private sequelize: Sequelize) {
    super();
  }

  async isHealthy(key: string): Promise<HealthIndicatorResult> {
    try {
      // Authenticate connection
      await this.sequelize.authenticate();
      
      return this.getStatus(key, true, {
        message: 'Sequelize connection is healthy',
      });
    } catch (error) {
      throw new HealthCheckError(
        'Sequelize check failed',
        this.getStatus(key, false, {
          message: error.message,
        }),
      );
    }
  }
}
```

---

### **Method 6: Prisma Health Indicator**

```typescript
// health/indicators/prisma.health.ts
import { Injectable } from '@nestjs/common';
import {
  HealthIndicator,
  HealthIndicatorResult,
  HealthCheckError,
} from '@nestjs/terminus';
import { PrismaService } from '../prisma.service';

@Injectable()
export class PrismaHealthIndicator extends HealthIndicator {
  constructor(private prisma: PrismaService) {
    super();
  }

  async isHealthy(key: string): Promise<HealthIndicatorResult> {
    try {
      // Execute raw query
      await this.prisma.$queryRaw`SELECT 1`;
      
      return this.getStatus(key, true, {
        message: 'Prisma connection is healthy',
      });
    } catch (error) {
      throw new HealthCheckError(
        'Prisma check failed',
        this.getStatus(key, false, {
          message: error.message,
        }),
      );
    }
  }
}
```

---

### **Method 7: MongoDB Health Indicator**

```typescript
// health/indicators/mongodb.health.ts
import { Injectable } from '@nestjs/common';
import {
  HealthIndicator,
  HealthIndicatorResult,
  HealthCheckError,
} from '@nestjs/terminus';
import { InjectConnection } from '@nestjs/mongoose';
import { Connection } from 'mongoose';

@Injectable()
export class MongoHealthIndicator extends HealthIndicator {
  constructor(@InjectConnection() private connection: Connection) {
    super();
  }

  async isHealthy(key: string): Promise<HealthIndicatorResult> {
    try {
      // Check connection state
      const state = this.connection.readyState;
      
      // 0 = disconnected, 1 = connected, 2 = connecting, 3 = disconnecting
      if (state !== 1) {
        throw new Error(`MongoDB state: ${this.getStateName(state)}`);
      }
      
      // Ping database
      await this.connection.db.admin().ping();
      
      return this.getStatus(key, true, {
        state: 'connected',
        host: this.connection.host,
        port: this.connection.port,
      });
    } catch (error) {
      throw new HealthCheckError(
        'MongoDB check failed',
        this.getStatus(key, false, {
          message: error.message,
        }),
      );
    }
  }

  private getStateName(state: number): string {
    const states = ['disconnected', 'connected', 'connecting', 'disconnecting'];
    return states[state] || 'unknown';
  }
}
```

---

### **Method 8: Query Performance Check**

```typescript
// health/indicators/query-performance.health.ts
import { Injectable } from '@nestjs/common';
import {
  HealthIndicator,
  HealthIndicatorResult,
  HealthCheckError,
} from '@nestjs/terminus';
import { InjectDataSource } from '@nestjs/typeorm';
import { DataSource } from 'typeorm';

@Injectable()
export class QueryPerformanceHealthIndicator extends HealthIndicator {
  constructor(@InjectDataSource() private dataSource: DataSource) {
    super();
  }

  async isHealthy(
    key: string,
    options: { maxDuration: number } = { maxDuration: 1000 },
  ): Promise<HealthIndicatorResult> {
    const start = Date.now();
    
    try {
      // Execute test query
      await this.dataSource.query('SELECT 1');
      const duration = Date.now() - start;
      
      // Check if query is too slow
      if (duration > options.maxDuration) {
        throw new Error(
          `Query took ${duration}ms (threshold: ${options.maxDuration}ms)`,
        );
      }
      
      return this.getStatus(key, true, {
        duration: `${duration}ms`,
        threshold: `${options.maxDuration}ms`,
      });
    } catch (error) {
      throw new HealthCheckError(
        'Query performance check failed',
        this.getStatus(key, false, {
          message: error.message,
        }),
      );
    }
  }
}

// Usage
@Get('db')
@HealthCheck()
checkDatabase() {
  return this.health.check([
    // Query must complete within 500ms
    () => this.queryPerf.isHealthy('query_performance', { maxDuration: 500 }),
  ]);
}
```

---

### **Method 9: Read/Write Check**

```typescript
// health/indicators/read-write.health.ts
import { Injectable } from '@nestjs/common';
import {
  HealthIndicator,
  HealthIndicatorResult,
  HealthCheckError,
} from '@nestjs/terminus';
import { InjectDataSource } from '@nestjs/typeorm';
import { DataSource } from 'typeorm';

@Injectable()
export class ReadWriteHealthIndicator extends HealthIndicator {
  constructor(@InjectDataSource() private dataSource: DataSource) {
    super();
  }

  async isHealthy(key: string): Promise<HealthIndicatorResult> {
    try {
      const testId = `health_check_${Date.now()}`;
      
      // Create test table if not exists
      await this.dataSource.query(`
        CREATE TABLE IF NOT EXISTS health_checks (
          id VARCHAR(255) PRIMARY KEY,
          created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        )
      `);
      
      // Write test
      await this.dataSource.query(
        'INSERT INTO health_checks (id) VALUES ($1)',
        [testId],
      );
      
      // Read test
      const result = await this.dataSource.query(
        'SELECT * FROM health_checks WHERE id = $1',
        [testId],
      );
      
      if (!result || result.length === 0) {
        throw new Error('Read test failed');
      }
      
      // Cleanup
      await this.dataSource.query(
        'DELETE FROM health_checks WHERE id = $1',
        [testId],
      );
      
      return this.getStatus(key, true, {
        read: 'ok',
        write: 'ok',
      });
    } catch (error) {
      throw new HealthCheckError(
        'Read/Write check failed',
        this.getStatus(key, false, {
          message: error.message,
        }),
      );
    }
  }
}
```

---

### **Method 10: Complete Production Setup**

```typescript
// health/health.module.ts
import { Module } from '@nestjs/common';
import { TerminusModule } from '@nestjs/terminus';
import { TypeOrmModule } from '@nestjs/typeorm';
import { HealthController } from './health.controller';
import { DatabaseHealthIndicator } from './indicators/database.health';
import { ConnectionPoolHealthIndicator } from './indicators/connection-pool.health';
import { QueryPerformanceHealthIndicator } from './indicators/query-performance.health';

@Module({
  imports: [TerminusModule, TypeOrmModule],
  controllers: [HealthController],
  providers: [
    DatabaseHealthIndicator,
    ConnectionPoolHealthIndicator,
    QueryPerformanceHealthIndicator,
  ],
})
export class HealthModule {}

// health/health.controller.ts
import { Controller, Get } from '@nestjs/common';
import { HealthCheck, HealthCheckService } from '@nestjs/terminus';
import { DatabaseHealthIndicator } from './indicators/database.health';
import { ConnectionPoolHealthIndicator } from './indicators/connection-pool.health';
import { QueryPerformanceHealthIndicator } from './indicators/query-performance.health';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private dbHealth: DatabaseHealthIndicator,
    private poolHealth: ConnectionPoolHealthIndicator,
    private queryPerfHealth: QueryPerformanceHealthIndicator,
  ) {}

  @Get('db')
  @HealthCheck()
  checkDatabase() {
    return this.health.check([
      // Basic connection check
      () => this.dbHealth.isHealthy('database'),
      
      // Connection pool health
      () => this.poolHealth.isHealthy('connection_pool'),
      
      // Query performance
      () => this.queryPerfHealth.isHealthy('query_performance', {
        maxDuration: 500,
      }),
    ]);
  }
}
```

---

### **Best Practices:**

```typescript
// ✅ GOOD - Use timeout on database checks
const timeout = new Promise((_, reject) =>
  setTimeout(() => reject(new Error('Timeout')), 5000),
);

Promise.race([this.dataSource.query('SELECT 1'), timeout]);

// ✅ GOOD - Check connection pool health
if (pool.waitingCount > 10) {
  throw new Error('Pool exhausted');
}

// ✅ GOOD - Cache health check results
private cache: { result: any; timestamp: number };

if (Date.now() - this.cache.timestamp < 10000) {
  return this.cache.result;  // Return cached result (10s TTL)
}

// ✅ GOOD - Monitor query performance
const start = Date.now();
await this.dataSource.query('SELECT 1');
const duration = Date.now() - start;

if (duration > 1000) {
  logger.warn(`Slow database query: ${duration}ms`);
}

// ❌ BAD - No timeout
await this.dataSource.query('SELECT 1');  // Could hang forever

// ❌ BAD - Complex queries in health check
await this.dataSource.query('SELECT * FROM users JOIN orders...');
// Use simple queries (SELECT 1)

// ❌ BAD - No error handling
return this.dataSource.query('SELECT 1');  // Throws unhandled error

// ❌ BAD - Checking on every request
// Cache results for 5-10 seconds
```

**Key Takeaway:** Check database health using `@nestjs/terminus` with **TypeOrmHealthIndicator** for simple ping checks, **custom indicators** for connection pool monitoring, **query performance checks** with timeouts, **read/write tests** for comprehensive validation, and **caching** to avoid checking on every request.

</details>

<details>
<summary><strong>35. How do you check external API health?</strong></summary>

**Answer:**

External API health checks verify that **third-party services** (payment gateways, authentication providers, email services) are **reachable** and **responding correctly**. Use `@nestjs/terminus` HttpHealthIndicator, custom health indicators with timeouts, circuit breakers, and fallback strategies.

---

### **Method 1: HttpHealthIndicator (Built-in)**

```typescript
// Install
npm install @nestjs/terminus @nestjs/axios axios

// health/health.controller.ts
import { Controller, Get } from '@nestjs/common';
import {
  HealthCheck,
  HealthCheckService,
  HttpHealthIndicator,
} from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private http: HttpHealthIndicator,
  ) {}

  @Get('external')
  @HealthCheck()
  checkExternalAPIs() {
    return this.health.check([
      // Ping external API
      () => this.http.pingCheck('stripe-api', 'https://api.stripe.com'),
      () => this.http.pingCheck('sendgrid-api', 'https://api.sendgrid.com'),
    ]);
  }
}

// Success (200):
{
  "status": "ok",
  "info": {
    "stripe-api": { "status": "up" },
    "sendgrid-api": { "status": "up" }
  },
  "error": {},
  "details": {
    "stripe-api": { "status": "up" },
    "sendgrid-api": { "status": "up" }
  }
}

// Failure (503):
{
  "status": "error",
  "info": {
    "stripe-api": { "status": "up" }
  },
  "error": {
    "sendgrid-api": {
      "status": "down",
      "message": "Request timeout"
    }
  },
  "details": { /* ... */ }
}
```

---

### **Method 2: Custom Health Indicator with Timeout**

```typescript
// health/indicators/external-api.health.ts
import { Injectable } from '@nestjs/common';
import {
  HealthIndicator,
  HealthIndicatorResult,
  HealthCheckError,
} from '@nestjs/terminus';
import { HttpService } from '@nestjs/axios';
import { firstValueFrom, timeout, catchError } from 'rxjs';

@Injectable()
export class ExternalApiHealthIndicator extends HealthIndicator {
  constructor(private readonly http: HttpService) {
    super();
  }

  async isHealthy(
    key: string,
    url: string,
    options: { timeout?: number; expectedStatus?: number } = {},
  ): Promise<HealthIndicatorResult> {
    const { timeout: timeoutMs = 5000, expectedStatus = 200 } = options;
    const start = Date.now();

    try {
      const response = await firstValueFrom(
        this.http.get(url).pipe(
          timeout(timeoutMs),
          catchError((error) => {
            throw error;
          }),
        ),
      );

      const duration = Date.now() - start;

      if (response.status !== expectedStatus) {
        throw new Error(`Expected status ${expectedStatus}, got ${response.status}`);
      }

      return this.getStatus(key, true, {
        statusCode: response.status,
        responseTime: `${duration}ms`,
      });
    } catch (error) {
      const duration = Date.now() - start;

      throw new HealthCheckError(
        `${key} health check failed`,
        this.getStatus(key, false, {
          message: error.message,
          responseTime: `${duration}ms`,
        }),
      );
    }
  }
}

// health/health.controller.ts
@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private apiHealth: ExternalApiHealthIndicator,
  ) {}

  @Get('external')
  @HealthCheck()
  checkExternalAPIs() {
    return this.health.check([
      // Check with 3 second timeout
      () => this.apiHealth.isHealthy(
        'payment-api',
        'https://api.stripe.com/v1/ping',
        { timeout: 3000 },
      ),
    ]);
  }
}
```

---

### **Method 3: Multiple External APIs**

```typescript
// health/health.controller.ts
import { Controller, Get } from '@nestjs/common';
import { HealthCheck, HealthCheckService } from '@nestjs/terminus';
import { ExternalApiHealthIndicator } from './indicators/external-api.health';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private apiHealth: ExternalApiHealthIndicator,
  ) {}

  @Get('external')
  @HealthCheck()
  checkExternalAPIs() {
    return this.health.check([
      // Payment gateway
      () => this.apiHealth.isHealthy(
        'stripe',
        'https://api.stripe.com',
        { timeout: 3000 },
      ),

      // Email service
      () => this.apiHealth.isHealthy(
        'sendgrid',
        'https://api.sendgrid.com',
        { timeout: 3000 },
      ),

      // SMS service
      () => this.apiHealth.isHealthy(
        'twilio',
        'https://api.twilio.com',
        { timeout: 3000 },
      ),

      // Auth provider
      () => this.apiHealth.isHealthy(
        'auth0',
        'https://YOUR_DOMAIN.auth0.com/test',
        { timeout: 3000 },
      ),
    ]);
  }
}

// Response:
{
  "status": "ok",
  "info": {
    "stripe": { "status": "up", "responseTime": "123ms" },
    "sendgrid": { "status": "up", "responseTime": "89ms" },
    "twilio": { "status": "up", "responseTime": "156ms" },
    "auth0": { "status": "up", "responseTime": "234ms" }
  },
  "error": {},
  "details": { /* ... */ }
}
```

---

### **Method 4: Authenticated API Health Check**

```typescript
// health/indicators/authenticated-api.health.ts
import { Injectable } from '@nestjs/common';
import {
  HealthIndicator,
  HealthIndicatorResult,
  HealthCheckError,
} from '@nestjs/terminus';
import { HttpService } from '@nestjs/axios';
import { firstValueFrom, timeout } from 'rxjs';

@Injectable()
export class AuthenticatedApiHealthIndicator extends HealthIndicator {
  constructor(private readonly http: HttpService) {
    super();
  }

  async isHealthy(
    key: string,
    url: string,
    headers: Record<string, string>,
  ): Promise<HealthIndicatorResult> {
    const start = Date.now();

    try {
      const response = await firstValueFrom(
        this.http.get(url, { headers }).pipe(timeout(5000)),
      );

      const duration = Date.now() - start;

      return this.getStatus(key, true, {
        statusCode: response.status,
        responseTime: `${duration}ms`,
      });
    } catch (error) {
      throw new HealthCheckError(
        `${key} health check failed`,
        this.getStatus(key, false, {
          message: error.message,
        }),
      );
    }
  }
}

// Usage
@Get('external')
@HealthCheck()
checkExternalAPIs() {
  return this.health.check([
    // Check with API key
    () => this.authApiHealth.isHealthy(
      'stripe',
      'https://api.stripe.com/v1/balance',
      {
        'Authorization': `Bearer ${process.env.STRIPE_SECRET_KEY}`,
      },
    ),
  ]);
}
```

---

### **Method 5: Response Validation**

```typescript
// health/indicators/validated-api.health.ts
import { Injectable } from '@nestjs/common';
import {
  HealthIndicator,
  HealthIndicatorResult,
  HealthCheckError,
} from '@nestjs/terminus';
import { HttpService } from '@nestjs/axios';
import { firstValueFrom, timeout } from 'rxjs';

@Injectable()
export class ValidatedApiHealthIndicator extends HealthIndicator {
  constructor(private readonly http: HttpService) {
    super();
  }

  async isHealthy(
    key: string,
    url: string,
    validator: (data: any) => boolean,
  ): Promise<HealthIndicatorResult> {
    try {
      const response = await firstValueFrom(
        this.http.get(url).pipe(timeout(5000)),
      );

      // Validate response data
      if (!validator(response.data)) {
        throw new Error('Response validation failed');
      }

      return this.getStatus(key, true, {
        statusCode: response.status,
      });
    } catch (error) {
      throw new HealthCheckError(
        `${key} health check failed`,
        this.getStatus(key, false, {
          message: error.message,
        }),
      );
    }
  }
}

// Usage
@Get('external')
@HealthCheck()
checkExternalAPIs() {
  return this.health.check([
    // Validate response structure
    () => this.validatedApiHealth.isHealthy(
      'weather-api',
      'https://api.weather.com/status',
      (data) => data.status === 'operational',
    ),
  ]);
}
```

---

### **Method 6: Circuit Breaker Pattern**

```typescript
// Install: npm install opossum

// health/indicators/circuit-breaker-api.health.ts
import { Injectable } from '@nestjs/common';
import {
  HealthIndicator,
  HealthIndicatorResult,
  HealthCheckError,
} from '@nestjs/terminus';
import CircuitBreaker from 'opossum';
import axios from 'axios';

@Injectable()
export class CircuitBreakerApiHealthIndicator extends HealthIndicator {
  private breakers: Map<string, CircuitBreaker> = new Map();

  getBreaker(key: string, url: string): CircuitBreaker {
    if (!this.breakers.has(key)) {
      const breaker = new CircuitBreaker(
        async () => axios.get(url, { timeout: 3000 }),
        {
          timeout: 3000,           // Function timeout
          errorThresholdPercentage: 50,  // Open after 50% failures
          resetTimeout: 30000,     // Try again after 30s
        },
      );

      breaker.on('open', () => {
        console.log(`Circuit breaker opened for ${key}`);
      });

      breaker.on('halfOpen', () => {
        console.log(`Circuit breaker half-open for ${key}`);
      });

      breaker.on('close', () => {
        console.log(`Circuit breaker closed for ${key}`);
      });

      this.breakers.set(key, breaker);
    }

    return this.breakers.get(key)!;
  }

  async isHealthy(key: string, url: string): Promise<HealthIndicatorResult> {
    const breaker = this.getBreaker(key, url);

    try {
      const response = await breaker.fire();

      return this.getStatus(key, true, {
        statusCode: response.status,
        circuitState: breaker.opened ? 'open' : 'closed',
      });
    } catch (error) {
      throw new HealthCheckError(
        `${key} health check failed`,
        this.getStatus(key, false, {
          message: error.message,
          circuitState: breaker.opened ? 'open' : 'closed',
        }),
      );
    }
  }
}

// Circuit breaker states:
// CLOSED: Normal operation, requests pass through
// OPEN: Too many failures, requests immediately fail
// HALF_OPEN: Testing if service recovered
```

---

### **Method 7: Cached Health Checks**

```typescript
// health/indicators/cached-api.health.ts
import { Injectable } from '@nestjs/common';
import {
  HealthIndicator,
  HealthIndicatorResult,
  HealthCheckError,
} from '@nestjs/terminus';
import { HttpService } from '@nestjs/axios';
import { firstValueFrom, timeout } from 'rxjs';

interface CachedResult {
  result: HealthIndicatorResult;
  timestamp: number;
  isHealthy: boolean;
}

@Injectable()
export class CachedApiHealthIndicator extends HealthIndicator {
  private cache: Map<string, CachedResult> = new Map();
  private readonly CACHE_TTL = 30000;  // 30 seconds

  constructor(private readonly http: HttpService) {
    super();
  }

  async isHealthy(key: string, url: string): Promise<HealthIndicatorResult> {
    const cached = this.cache.get(key);
    const now = Date.now();

    // Return cached result if still valid
    if (cached && now - cached.timestamp < this.CACHE_TTL) {
      return cached.result;
    }

    try {
      const response = await firstValueFrom(
        this.http.get(url).pipe(timeout(3000)),
      );

      const result = this.getStatus(key, true, {
        statusCode: response.status,
        cached: false,
      });

      // Cache successful result
      this.cache.set(key, {
        result,
        timestamp: now,
        isHealthy: true,
      });

      return result;
    } catch (error) {
      // Return last known good state if available
      if (cached && cached.isHealthy) {
        return cached.result;
      }

      throw new HealthCheckError(
        `${key} health check failed`,
        this.getStatus(key, false, {
          message: error.message,
        }),
      );
    }
  }
}
```

---

### **Method 8: Parallel Health Checks**

```typescript
// health/health.controller.ts
import { Controller, Get } from '@nestjs/common';
import { HealthCheck, HealthCheckService } from '@nestjs/terminus';
import { ExternalApiHealthIndicator } from './indicators/external-api.health';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private apiHealth: ExternalApiHealthIndicator,
  ) {}

  @Get('external')
  @HealthCheck()
  async checkExternalAPIs() {
    // Parallel health checks (faster than sequential)
    return this.health.check([
      () => this.apiHealth.isHealthy('stripe', 'https://api.stripe.com'),
      () => this.apiHealth.isHealthy('sendgrid', 'https://api.sendgrid.com'),
      () => this.apiHealth.isHealthy('twilio', 'https://api.twilio.com'),
    ]);
    // All checks run in parallel, returns when all complete or any fails
  }
}
```

---

### **Method 9: Graceful Degradation**

```typescript
// health/health.controller.ts
import { Controller, Get } from '@nestjs/common';
import { HealthCheck, HealthCheckService } from '@nestjs/terminus';
import { ExternalApiHealthIndicator } from './indicators/external-api.health';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private apiHealth: ExternalApiHealthIndicator,
  ) {}

  // Critical dependencies (must be healthy)
  @Get('ready')
  @HealthCheck()
  checkReady() {
    return this.health.check([
      () => this.apiHealth.isHealthy('payment-gateway', 'https://api.stripe.com'),
      // If payment gateway down, don't accept traffic
    ]);
  }

  // Non-critical dependencies (informational only)
  @Get('external/optional')
  async checkOptionalAPIs() {
    const results: any = {};

    // Email service (optional - can queue emails if down)
    try {
      await this.apiHealth.isHealthy('email', 'https://api.sendgrid.com');
      results.email = { status: 'up' };
    } catch (error) {
      results.email = { status: 'down', message: 'Emails will be queued' };
    }

    // SMS service (optional - can skip notifications if down)
    try {
      await this.apiHealth.isHealthy('sms', 'https://api.twilio.com');
      results.sms = { status: 'up' };
    } catch (error) {
      results.sms = { status: 'down', message: 'SMS disabled temporarily' };
    }

    // Always return 200 for optional dependencies
    return { status: 'ok', ...results };
  }
}
```

---

### **Method 10: Complete Production Setup**

```typescript
// health/health.module.ts
import { Module } from '@nestjs/common';
import { TerminusModule } from '@nestjs/terminus';
import { HttpModule } from '@nestjs/axios';
import { HealthController } from './health.controller';
import { ExternalApiHealthIndicator } from './indicators/external-api.health';
import { CircuitBreakerApiHealthIndicator } from './indicators/circuit-breaker-api.health';
import { CachedApiHealthIndicator } from './indicators/cached-api.health';

@Module({
  imports: [
    TerminusModule,
    HttpModule.register({
      timeout: 5000,
      maxRedirects: 5,
    }),
  ],
  controllers: [HealthController],
  providers: [
    ExternalApiHealthIndicator,
    CircuitBreakerApiHealthIndicator,
    CachedApiHealthIndicator,
  ],
})
export class HealthModule {}

// health/health.controller.ts
@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private apiHealth: ExternalApiHealthIndicator,
    private circuitApiHealth: CircuitBreakerApiHealthIndicator,
    private cachedApiHealth: CachedApiHealthIndicator,
  ) {}

  @Get('external')
  @HealthCheck()
  checkExternalAPIs() {
    return this.health.check([
      // Critical API with circuit breaker
      () => this.circuitApiHealth.isHealthy(
        'payment-gateway',
        'https://api.stripe.com',
      ),

      // Non-critical API with caching
      () => this.cachedApiHealth.isHealthy(
        'email-service',
        'https://api.sendgrid.com',
      ),

      // Standard health check
      () => this.apiHealth.isHealthy(
        'auth-provider',
        'https://auth.example.com/health',
        { timeout: 3000 },
      ),
    ]);
  }
}
```

---

### **Best Practices:**

```typescript
// ✅ GOOD - Set reasonable timeouts
this.http.get(url).pipe(timeout(3000))  // 3 second timeout

// ✅ GOOD - Use circuit breakers for flaky APIs
const breaker = new CircuitBreaker(apiCall, {
  errorThresholdPercentage: 50,
  resetTimeout: 30000,
});

// ✅ GOOD - Cache health check results
if (cached && Date.now() - cached.timestamp < 30000) {
  return cached.result;  // Return cached (30s TTL)
}

// ✅ GOOD - Distinguish critical vs non-critical
readiness: [payment-gateway]  // Critical
optional: [email-service, sms-service]  // Non-critical

// ✅ GOOD - Run checks in parallel
Promise.all([
  checkStripe(),
  checkSendgrid(),
  checkTwilio(),
]);

// ✅ GOOD - Monitor API health metrics
const apiHealthGauge = new client.Gauge({
  name: 'external_api_health',
  labelNames: ['api_name'],
});

// ❌ BAD - No timeout
await axios.get(url);  // Could hang forever

// ❌ BAD - Checking on every request
// Use caching (30-60 seconds)

// ❌ BAD - Same priority for all APIs
// Critical APIs should fail readiness check
// Optional APIs should not

// ❌ BAD - Sequential checks
await checkStripe();
await checkSendgrid();  // Slow!
// Use Promise.all() for parallel

// ❌ BAD - No circuit breaker for flaky APIs
// API with 50% failure rate will keep failing
```

**Key Takeaway:** Check external API health using `@nestjs/terminus` **HttpHealthIndicator** for simple pings, **custom indicators** with timeouts and response validation, **circuit breakers** for flaky APIs, **caching** (30s TTL) to reduce check frequency, and **distinguish critical vs non-critical** APIs (critical failures affect readiness, optional failures don't).

</details>

## Metrics & Observability

<details>
<summary><strong>36. What metrics should you track (latency, throughput, error rate)?</strong></summary>

**Answer:**

Track **performance metrics** (latency, throughput), **reliability metrics** (error rate, availability), **saturation metrics** (CPU, memory, connections), and **business metrics** (user signups, revenue). Use the **RED method** (Rate, Errors, Duration) or **USE method** (Utilization, Saturation, Errors) for comprehensive monitoring.

---

### **Key Metrics Categories:**

#### **1. Performance Metrics (Speed)**
```typescript
✅ Latency / Response Time
   - Average response time
   - p50 (median)
   - p95 (95th percentile)
   - p99 (99th percentile)
   - p99.9 (99.9th percentile)

✅ Throughput
   - Requests per second (RPS)
   - Transactions per minute (TPM)
   - Messages processed per second

✅ Duration
   - Database query time
   - External API call time
   - Cache lookup time
   - Function execution time
```

#### **2. Reliability Metrics (Errors)**
```typescript
✅ Error Rate
   - Percentage of failed requests
   - 4xx errors (client errors)
   - 5xx errors (server errors)
   - Error count by endpoint

✅ Availability
   - Uptime percentage (99.9%, 99.99%)
   - Downtime minutes per month
   - MTBF (Mean Time Between Failures)
   - MTTR (Mean Time To Recovery)

✅ Success Rate
   - Percentage of successful requests
   - Successful transactions
```

#### **3. Saturation Metrics (Resource Usage)**
```typescript
✅ CPU Usage
   - CPU utilization percentage
   - Load average (1m, 5m, 15m)

✅ Memory Usage
   - Heap used vs total
   - RSS (Resident Set Size)
   - Memory leak detection

✅ Connections
   - Active connections
   - Connection pool usage
   - Queue depth

✅ Disk I/O
   - Disk usage percentage
   - I/O operations per second
```

#### **4. Business Metrics**
```typescript
✅ User Actions
   - User signups
   - Login attempts
   - Feature usage

✅ Transactions
   - Orders placed
   - Payments processed
   - Revenue generated

✅ Engagement
   - Active users
   - Session duration
   - Page views
```

---

### **The RED Method (Rate, Errors, Duration):**

```typescript
// For REQUEST-DRIVEN services (APIs, web servers)

import * as client from 'prom-client';

// 1. RATE: Requests per second
const httpRequestsTotal = new client.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code'],
});

httpRequestsTotal.inc({ method: 'GET', route: '/api/users', status_code: 200 });

// Query: rate(http_requests_total[5m])
// Result: 1,234 requests/sec

// 2. ERRORS: Error rate
const httpRequestErrorsTotal = new client.Counter({
  name: 'http_request_errors_total',
  help: 'Total number of HTTP request errors',
  labelNames: ['method', 'route', 'error_type'],
});

httpRequestErrorsTotal.inc({ method: 'POST', route: '/api/orders', error_type: 'DatabaseError' });

// Query: rate(http_request_errors_total[5m]) / rate(http_requests_total[5m])
// Result: 0.23% error rate

// 3. DURATION: Response time
const httpRequestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10],
});

httpRequestDuration.observe(
  { method: 'GET', route: '/api/users', status_code: 200 },
  0.145,  // 145ms in seconds
);

// Queries:
// Average: rate(http_request_duration_seconds_sum[5m]) / rate(http_request_duration_seconds_count[5m])
// p95: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
// p99: histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))
```

---

### **The USE Method (Utilization, Saturation, Errors):**

```typescript
// For RESOURCE-ORIENTED monitoring (CPU, memory, disk, network)

import * as client from 'prom-client';

// 1. UTILIZATION: How busy is the resource?
const cpuUsage = new client.Gauge({
  name: 'cpu_usage_percent',
  help: 'CPU usage percentage',
});

setInterval(() => {
  const usage = process.cpuUsage();
  cpuUsage.set((usage.user + usage.system) / 1000000);  // Convert to seconds
}, 5000);

// 2. SATURATION: How much work is queued?
const connectionPoolWaiting = new client.Gauge({
  name: 'connection_pool_waiting_count',
  help: 'Number of connections waiting in pool',
});

connectionPoolWaiting.set(pool.waitingCount);

// 3. ERRORS: Resource errors
const diskErrors = new client.Counter({
  name: 'disk_errors_total',
  help: 'Total disk errors',
  labelNames: ['operation'],
});

diskErrors.inc({ operation: 'write' });
```

---

### **Implementation Example:**

```typescript
// metrics/metrics.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import * as client from 'prom-client';

@Injectable()
export class MetricsService implements OnModuleInit {
  private register: client.Registry;

  // Performance metrics
  private httpRequestsTotal: client.Counter;
  private httpRequestDuration: client.Histogram;

  // Error metrics
  private httpRequestErrorsTotal: client.Counter;

  // Resource metrics
  private cpuUsage: client.Gauge;
  private memoryUsage: client.Gauge;
  private eventLoopLag: client.Gauge;

  // Business metrics
  private userSignups: client.Counter;
  private ordersTotal: client.Counter;

  constructor() {
    this.register = new client.Registry();
    this.initializeMetrics();
  }

  onModuleInit() {
    // Collect default metrics (CPU, memory, etc.)
    client.collectDefaultMetrics({ register: this.register });

    // Start monitoring event loop lag
    this.monitorEventLoop();
  }

  private initializeMetrics() {
    // Performance metrics
    this.httpRequestsTotal = new client.Counter({
      name: 'http_requests_total',
      help: 'Total HTTP requests',
      labelNames: ['method', 'route', 'status_code'],
      registers: [this.register],
    });

    this.httpRequestDuration = new client.Histogram({
      name: 'http_request_duration_seconds',
      help: 'HTTP request duration',
      labelNames: ['method', 'route', 'status_code'],
      buckets: [0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5],
      registers: [this.register],
    });

    // Error metrics
    this.httpRequestErrorsTotal = new client.Counter({
      name: 'http_request_errors_total',
      help: 'Total HTTP request errors',
      labelNames: ['method', 'route', 'error_type'],
      registers: [this.register],
    });

    // Resource metrics
    this.cpuUsage = new client.Gauge({
      name: 'process_cpu_usage_percent',
      help: 'Process CPU usage percentage',
      registers: [this.register],
    });

    this.memoryUsage = new client.Gauge({
      name: 'process_memory_usage_bytes',
      help: 'Process memory usage in bytes',
      labelNames: ['type'],
      registers: [this.register],
    });

    this.eventLoopLag = new client.Gauge({
      name: 'nodejs_eventloop_lag_seconds',
      help: 'Event loop lag in seconds',
      registers: [this.register],
    });

    // Business metrics
    this.userSignups = new client.Counter({
      name: 'user_signups_total',
      help: 'Total user signups',
      registers: [this.register],
    });

    this.ordersTotal = new client.Counter({
      name: 'orders_total',
      help: 'Total orders placed',
      labelNames: ['status'],
      registers: [this.register],
    });
  }

  private monitorEventLoop() {
    let lastCheck = Date.now();

    setInterval(() => {
      const now = Date.now();
      const lag = (now - lastCheck - 100) / 1000;  // Expected 100ms interval
      this.eventLoopLag.set(Math.max(0, lag));
      lastCheck = now;
    }, 100);
  }

  // Record HTTP request
  recordRequest(method: string, route: string, statusCode: number, duration: number) {
    this.httpRequestsTotal.inc({ method, route, status_code: statusCode });
    this.httpRequestDuration.observe(
      { method, route, status_code: statusCode },
      duration / 1000,  // Convert ms to seconds
    );
  }

  // Record error
  recordError(method: string, route: string, errorType: string) {
    this.httpRequestErrorsTotal.inc({ method, route, error_type: errorType });
  }

  // Record business event
  recordUserSignup() {
    this.userSignups.inc();
  }

  recordOrder(status: 'completed' | 'failed') {
    this.ordersTotal.inc({ status });
  }

  // Get metrics
  async getMetrics(): Promise<string> {
    return this.register.metrics();
  }
}
```

---

### **Metrics Priority (Golden Signals):**

```typescript
// The 4 Golden Signals (Google SRE)

1. LATENCY
   - How long requests take
   p50: 50ms  // Half of requests faster than this
   p95: 200ms  // 95% of requests faster than this
   p99: 500ms  // 99% of requests faster than this

2. TRAFFIC
   - Volume of requests
   1,234 requests/second
   5,000 transactions/minute

3. ERRORS
   - Rate of failing requests
   0.1% error rate (1 in 1000 requests)
   99.9% success rate

4. SATURATION
   - How full the service is
   CPU: 65%
   Memory: 80%
   Connections: 70/100

// Alert on:
✅ p99 latency > 1000ms
✅ Error rate > 1%
✅ CPU usage > 80%
✅ Memory usage > 90%
```

---

### **Alerting Thresholds:**

```typescript
// metrics/alerts.ts
export const ALERT_THRESHOLDS = {
  // Performance
  maxP99Latency: 1000,        // 1 second
  maxP95Latency: 500,         // 500ms
  maxAvgLatency: 200,         // 200ms

  // Reliability
  maxErrorRate: 0.01,         // 1%
  minAvailability: 0.999,     // 99.9%

  // Saturation
  maxCpuUsage: 0.80,          // 80%
  maxMemoryUsage: 0.90,       // 90%
  maxEventLoopLag: 0.1,       // 100ms

  // Throughput
  minRequestsPerSec: 100,     // Alert if traffic too low (possible issue)
};

// Check and alert
if (p99Latency > ALERT_THRESHOLDS.maxP99Latency) {
  alert('High latency detected!');
}

if (errorRate > ALERT_THRESHOLDS.maxErrorRate) {
  alert('High error rate!');
}
```

---

### **Dashboard Visualization:**

```
Grafana Dashboard Example:

┌─────────────────────────────────────────────────────────────┐
│  Performance Overview                                       │
├─────────────────────────────────────────────────────────────┤
│  📊 Response Time (last 1h)                                │
│     Average: 145ms                                          │
│     p95: 340ms                                              │
│     p99: 890ms                                              │
│     [Line chart showing trend]                              │
├─────────────────────────────────────────────────────────────┤
│  📈 Throughput                                              │
│     1,234 requests/sec   ↑ 12%                            │
│     [Bar chart by endpoint]                                 │
├─────────────────────────────────────────────────────────────┤
│  ⚠️  Error Rate                                             │
│     0.23%   ↓ 0.05%                                        │
│     [Pie chart: 4xx vs 5xx]                                │
├─────────────────────────────────────────────────────────────┤
│  💾 Resource Usage                                          │
│     CPU: 65%   Memory: 1.2GB/2GB   Connections: 45/100    │
│     [Gauge charts]                                          │
└─────────────────────────────────────────────────────────────┘
```

---

### **Best Practices:**

```typescript
// ✅ GOOD - Track percentiles, not just averages
p50: 100ms  // Median
p95: 300ms  // 95th percentile
p99: 800ms  // 99th percentile
// Averages hide outliers!

// ✅ GOOD - Use histograms for response times
const histogram = new client.Histogram({
  name: 'http_request_duration_seconds',
  buckets: [0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5],
});

// ✅ GOOD - Label metrics for filtering
labelNames: ['method', 'route', 'status_code']
// Can query: rate(http_requests_total{route="/api/users"}[5m])

// ✅ GOOD - Track both technical and business metrics
http_requests_total  // Technical
orders_total         // Business

// ✅ GOOD - Set up alerts on metrics
if (errorRate > 0.01) {
  sendAlert('High error rate');
}

// ❌ BAD - Only tracking averages
avg_response_time = 150ms
// Hides the fact that p99 = 5000ms!

// ❌ BAD - No labels on metrics
http_requests_total
// Can't distinguish /api/users from /api/orders

// ❌ BAD - Not tracking error rate
// Only tracking response time

// ❌ BAD - No resource monitoring
// CPU/memory at 100% but you don't know
```

**Key Takeaway:** Track **performance metrics** (latency p50/p95/p99, throughput), **reliability metrics** (error rate, availability), **saturation metrics** (CPU, memory, connections), and **business metrics** (signups, orders). Use **RED method** (Rate, Errors, Duration) for services and **USE method** (Utilization, Saturation, Errors) for resources.

</details>

<details>
<summary><strong>37. How do you integrate Prometheus for metrics?</strong></summary>

**Answer:**

Integrate Prometheus using the `prom-client` library to **collect metrics**, expose a `/metrics` endpoint for Prometheus to scrape, and configure Prometheus server to poll your NestJS app. Use **counters** for totals, **gauges** for current values, **histograms** for distributions, and **summaries** for percentiles.

---

### **Method 1: Basic Setup with prom-client**

```typescript
// Install
npm install prom-client

// metrics/metrics.module.ts
import { Module, Global } from '@nestjs/common';
import { MetricsService } from './metrics.service';
import { MetricsController } from './metrics.controller';

@Global()  // Make available throughout app
@Module({
  providers: [MetricsService],
  controllers: [MetricsController],
  exports: [MetricsService],
})
export class MetricsModule {}

// metrics/metrics.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import * as client from 'prom-client';

@Injectable()
export class MetricsService implements OnModuleInit {
  private readonly register: client.Registry;

  constructor() {
    this.register = new client.Registry();
  }

  onModuleInit() {
    // Collect default metrics (CPU, memory, event loop, etc.)
    client.collectDefaultMetrics({
      register: this.register,
      prefix: 'nestjs_',  // Prefix all default metrics
    });
  }

  getMetrics(): Promise<string> {
    return this.register.metrics();
  }

  getRegister(): client.Registry {
    return this.register;
  }
}

// metrics/metrics.controller.ts
import { Controller, Get, Header } from '@nestjs/common';
import { MetricsService } from './metrics.service';

@Controller('metrics')
export class MetricsController {
  constructor(private readonly metricsService: MetricsService) {}

  @Get()
  @Header('Content-Type', client.register.contentType)
  async getMetrics() {
    return this.metricsService.getMetrics();
  }
}

// Output at GET http://localhost:3000/metrics:
# HELP nodejs_heap_size_total_bytes Process heap size from Node.js in bytes.
# TYPE nodejs_heap_size_total_bytes gauge
nodejs_heap_size_total_bytes 18874368

# HELP nodejs_heap_size_used_bytes Process heap size used from Node.js in bytes.
# TYPE nodejs_heap_size_used_bytes gauge
nodejs_heap_size_used_bytes 15234560

# HELP process_cpu_user_seconds_total Total user CPU time spent in seconds.
# TYPE process_cpu_user_seconds_total counter
process_cpu_user_seconds_total 0.234
```

---

### **Method 2: Custom Counter (for totals)**

```typescript
// metrics/metrics.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import * as client from 'prom-client';

@Injectable()
export class MetricsService implements OnModuleInit {
  private readonly register: client.Registry;
  private httpRequestsTotal: client.Counter;

  constructor() {
    this.register = new client.Registry();
    this.initializeMetrics();
  }

  onModuleInit() {
    client.collectDefaultMetrics({ register: this.register });
  }

  private initializeMetrics() {
    // Counter: Only goes up (total requests, total errors, etc.)
    this.httpRequestsTotal = new client.Counter({
      name: 'http_requests_total',
      help: 'Total number of HTTP requests',
      labelNames: ['method', 'route', 'status_code'],
      registers: [this.register],
    });
  }

  // Increment counter
  incrementRequestCount(method: string, route: string, statusCode: number) {
    this.httpRequestsTotal.inc({
      method,
      route,
      status_code: statusCode,
    });
  }

  getMetrics(): Promise<string> {
    return this.register.metrics();
  }
}

// Usage in interceptor
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';
import { MetricsService } from '../metrics/metrics.service';

@Injectable()
export class MetricsInterceptor implements NestInterceptor {
  constructor(private metricsService: MetricsService) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const response = context.switchToHttp().getResponse();

    return next.handle().pipe(
      tap(() => {
        this.metricsService.incrementRequestCount(
          request.method,
          request.route.path,
          response.statusCode,
        );
      }),
    );
  }
}

// Output at /metrics:
# HELP http_requests_total Total number of HTTP requests
# TYPE http_requests_total counter
http_requests_total{method="GET",route="/api/users",status_code="200"} 1234
http_requests_total{method="POST",route="/api/orders",status_code="201"} 567
http_requests_total{method="GET",route="/api/products",status_code="200"} 8901
```

---

### **Method 3: Gauge (for current values)**

```typescript
// metrics/metrics.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import * as client from 'prom-client';

@Injectable()
export class MetricsService implements OnModuleInit {
  private readonly register: client.Registry;
  private activeConnections: client.Gauge;
  private queueSize: client.Gauge;

  constructor() {
    this.register = new client.Registry();
    this.initializeMetrics();
  }

  private initializeMetrics() {
    // Gauge: Can go up or down (active connections, queue size, temperature)
    this.activeConnections = new client.Gauge({
      name: 'active_connections',
      help: 'Number of active connections',
      registers: [this.register],
    });

    this.queueSize = new client.Gauge({
      name: 'queue_size',
      help: 'Current size of the processing queue',
      labelNames: ['queue_name'],
      registers: [this.register],
    });
  }

  // Set gauge value
  setActiveConnections(count: number) {
    this.activeConnections.set(count);
  }

  // Increment/decrement
  incrementConnections() {
    this.activeConnections.inc();
  }

  decrementConnections() {
    this.activeConnections.dec();
  }

  setQueueSize(queueName: string, size: number) {
    this.queueSize.set({ queue_name: queueName }, size);
  }
}

// Output at /metrics:
# HELP active_connections Number of active connections
# TYPE active_connections gauge
active_connections 45

# HELP queue_size Current size of the processing queue
# TYPE queue_size gauge
queue_size{queue_name="email"} 12
queue_size{queue_name="notifications"} 5
```

---

### **Method 4: Histogram (for distributions)**

```typescript
// metrics/metrics.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import * as client from 'prom-client';

@Injectable()
export class MetricsService implements OnModuleInit {
  private readonly register: client.Registry;
  private httpRequestDuration: client.Histogram;

  constructor() {
    this.register = new client.Registry();
    this.initializeMetrics();
  }

  private initializeMetrics() {
    // Histogram: Counts observations in buckets (for percentiles)
    this.httpRequestDuration = new client.Histogram({
      name: 'http_request_duration_seconds',
      help: 'HTTP request duration in seconds',
      labelNames: ['method', 'route', 'status_code'],
      buckets: [0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10],
      registers: [this.register],
    });
  }

  // Record observation
  recordRequestDuration(method: string, route: string, statusCode: number, durationMs: number) {
    this.httpRequestDuration.observe(
      { method, route, status_code: statusCode },
      durationMs / 1000,  // Convert milliseconds to seconds
    );
  }
}

// Usage in interceptor
@Injectable()
export class MetricsInterceptor implements NestInterceptor {
  constructor(private metricsService: MetricsService) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const start = Date.now();
    const request = context.switchToHttp().getRequest();
    const response = context.switchToHttp().getResponse();

    return next.handle().pipe(
      tap(() => {
        const duration = Date.now() - start;
        this.metricsService.recordRequestDuration(
          request.method,
          request.route.path,
          response.statusCode,
          duration,
        );
      }),
    );
  }
}

// Output at /metrics:
# HELP http_request_duration_seconds HTTP request duration in seconds
# TYPE http_request_duration_seconds histogram
http_request_duration_seconds_bucket{le="0.005",method="GET",route="/api/users",status_code="200"} 123
http_request_duration_seconds_bucket{le="0.01",method="GET",route="/api/users",status_code="200"} 456
http_request_duration_seconds_bucket{le="0.025",method="GET",route="/api/users",status_code="200"} 789
http_request_duration_seconds_bucket{le="+Inf",method="GET",route="/api/users",status_code="200"} 1234
http_request_duration_seconds_sum{method="GET",route="/api/users",status_code="200"} 123.45
http_request_duration_seconds_count{method="GET",route="/api/users",status_code="200"} 1234

// Calculate percentiles in Prometheus:
// p50: histogram_quantile(0.50, rate(http_request_duration_seconds_bucket[5m]))
// p95: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
// p99: histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))
```

---

### **Method 5: Summary (for pre-calculated percentiles)**

```typescript
// metrics/metrics.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import * as client from 'prom-client';

@Injectable()
export class MetricsService implements OnModuleInit {
  private readonly register: client.Registry;
  private requestDurationSummary: client.Summary;

  constructor() {
    this.register = new client.Registry();
    this.initializeMetrics();
  }

  private initializeMetrics() {
    // Summary: Pre-calculates percentiles on client side
    this.requestDurationSummary = new client.Summary({
      name: 'http_request_duration_summary_seconds',
      help: 'HTTP request duration summary',
      labelNames: ['method', 'route'],
      percentiles: [0.5, 0.9, 0.95, 0.99],  // Calculate p50, p90, p95, p99
      registers: [this.register],
    });
  }

  recordRequestDurationSummary(method: string, route: string, durationMs: number) {
    this.requestDurationSummary.observe(
      { method, route },
      durationMs / 1000,
    );
  }
}

// Output at /metrics:
# HELP http_request_duration_summary_seconds HTTP request duration summary
# TYPE http_request_duration_summary_seconds summary
http_request_duration_summary_seconds{quantile="0.5",method="GET",route="/api/users"} 0.145
http_request_duration_summary_seconds{quantile="0.9",method="GET",route="/api/users"} 0.340
http_request_duration_summary_seconds{quantile="0.95",method="GET",route="/api/users"} 0.520
http_request_duration_summary_seconds{quantile="0.99",method="GET",route="/api/users"} 0.890
http_request_duration_summary_seconds_sum{method="GET",route="/api/users"} 123.45
http_request_duration_summary_seconds_count{method="GET",route="/api/users"} 1234

// Histogram vs Summary:
// Histogram: Prometheus calculates percentiles (more flexible, can aggregate across instances)
// Summary: Client calculates percentiles (less server load, but can't aggregate)
```

---

### **Method 6: Prometheus Configuration**

```yaml
# prometheus.yml
global:
  scrape_interval: 15s  # Scrape every 15 seconds
  evaluation_interval: 15s  # Evaluate rules every 15 seconds

scrape_configs:
  - job_name: 'nestjs-api'
    static_configs:
      - targets: ['localhost:3000']  # Your NestJS app
    metrics_path: '/metrics'  # Endpoint to scrape
    scrape_interval: 10s  # Override global interval

  - job_name: 'nestjs-workers'
    static_configs:
      - targets:
        - 'worker-1:3000'
        - 'worker-2:3000'
        - 'worker-3:3000'
    metrics_path: '/metrics'

# Run Prometheus
docker run -p 9090:9090 -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus

# Access Prometheus UI: http://localhost:9090
```

---

### **Method 7: PromQL Queries**

```promql
# Request rate (requests per second)
rate(http_requests_total[5m])

# Request rate by endpoint
sum by (route) (rate(http_requests_total[5m]))

# Error rate (percentage)
sum(rate(http_requests_total{status_code=~"5.."}[5m])) 
/ 
sum(rate(http_requests_total[5m])) 
* 100

# Average response time
rate(http_request_duration_seconds_sum[5m]) 
/ 
rate(http_request_duration_seconds_count[5m])

# p95 latency
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# p99 latency by endpoint
histogram_quantile(
  0.99, 
  sum by (route, le) (rate(http_request_duration_seconds_bucket[5m]))
)

# CPU usage
rate(process_cpu_user_seconds_total[5m]) * 100

# Memory usage (MB)
process_resident_memory_bytes / 1024 / 1024

# Active connections
active_connections

# Top 5 slowest endpoints (p99)
topk(5, 
  histogram_quantile(
    0.99, 
    sum by (route, le) (rate(http_request_duration_seconds_bucket[5m]))
  )
)
```

---

### **Method 8: Grafana Dashboard**

```typescript
// Grafana dashboard panels (import JSON or create manually)

{
  "dashboard": {
    "title": "NestJS Performance",
    "panels": [
      {
        "title": "Request Rate",
        "targets": [{
          "expr": "sum(rate(http_requests_total[5m]))"
        }]
      },
      {
        "title": "Error Rate",
        "targets": [{
          "expr": "sum(rate(http_requests_total{status_code=~\"5..\"}[5m])) / sum(rate(http_requests_total[5m])) * 100"
        }]
      },
      {
        "title": "Response Time Percentiles",
        "targets": [
          {
            "expr": "histogram_quantile(0.50, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "p50"
          },
          {
            "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "p95"
          },
          {
            "expr": "histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "p99"
          }
        ]
      },
      {
        "title": "Memory Usage",
        "targets": [{
          "expr": "process_resident_memory_bytes / 1024 / 1024"
        }]
      }
    ]
  }
}

// Run Grafana
docker run -d -p 3001:3000 grafana/grafana

// Access Grafana: http://localhost:3001
// Default credentials: admin / admin

// Add Prometheus data source:
// Configuration → Data Sources → Add data source → Prometheus
// URL: http://prometheus:9090
```

---

### **Method 9: Alerting Rules**

```yaml
# prometheus-alerts.yml
groups:
  - name: api_alerts
    interval: 30s
    rules:
      # High error rate
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status_code=~"5.."}[5m])) 
          / 
          sum(rate(http_requests_total[5m])) 
          > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value | humanizePercentage }}"

      # High latency (p99 > 1s)
      - alert: HighLatency
        expr: |
          histogram_quantile(
            0.99, 
            rate(http_request_duration_seconds_bucket[5m])
          ) > 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High latency detected"
          description: "p99 latency is {{ $value }}s"

      # High CPU usage
      - alert: HighCpuUsage
        expr: rate(process_cpu_user_seconds_total[5m]) * 100 > 80
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage"
          description: "CPU usage is {{ $value }}%"

      # High memory usage
      - alert: HighMemoryUsage
        expr: process_resident_memory_bytes / 1024 / 1024 > 1500
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage"
          description: "Memory usage is {{ $value }}MB"

      # Low request rate (possible downtime)
      - alert: LowRequestRate
        expr: sum(rate(http_requests_total[5m])) < 10
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Low request rate"
          description: "Only {{ $value }} requests/sec"

# Load alerts in prometheus.yml
rule_files:
  - 'prometheus-alerts.yml'

# Configure Alertmanager for notifications
alertmanagers:
  - static_configs:
      - targets: ['alertmanager:9093']
```

---

### **Method 10: Complete Production Setup**

```typescript
// metrics/metrics.module.ts
import { Module, Global } from '@nestjs/common';
import { MetricsService } from './metrics.service';
import { MetricsController } from './metrics.controller';
import { APP_INTERCEPTOR } from '@nestjs/core';
import { MetricsInterceptor } from './metrics.interceptor';

@Global()
@Module({
  providers: [
    MetricsService,
    {
      provide: APP_INTERCEPTOR,
      useClass: MetricsInterceptor,
    },
  ],
  controllers: [MetricsController],
  exports: [MetricsService],
})
export class MetricsModule {}

// metrics/metrics.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import * as client from 'prom-client';

@Injectable()
export class MetricsService implements OnModuleInit {
  private readonly register: client.Registry;

  // HTTP metrics
  private httpRequestsTotal: client.Counter;
  private httpRequestDuration: client.Histogram;
  private httpRequestSize: client.Histogram;
  private httpResponseSize: client.Histogram;

  // Business metrics
  private userSignups: client.Counter;
  private ordersTotal: client.Counter;
  private paymentAmount: client.Histogram;

  // Resource metrics
  private activeConnections: client.Gauge;
  private databaseConnections: client.Gauge;
  private cacheHitRate: client.Gauge;

  constructor() {
    this.register = new client.Registry();
    this.initializeMetrics();
  }

  onModuleInit() {
    // Default metrics
    client.collectDefaultMetrics({
      register: this.register,
      prefix: 'nestjs_',
    });
  }

  private initializeMetrics() {
    // HTTP metrics
    this.httpRequestsTotal = new client.Counter({
      name: 'http_requests_total',
      help: 'Total HTTP requests',
      labelNames: ['method', 'route', 'status_code'],
      registers: [this.register],
    });

    this.httpRequestDuration = new client.Histogram({
      name: 'http_request_duration_seconds',
      help: 'HTTP request duration',
      labelNames: ['method', 'route', 'status_code'],
      buckets: [0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5],
      registers: [this.register],
    });

    this.httpRequestSize = new client.Histogram({
      name: 'http_request_size_bytes',
      help: 'HTTP request size in bytes',
      labelNames: ['method', 'route'],
      buckets: [100, 1000, 5000, 10000, 50000, 100000, 500000, 1000000],
      registers: [this.register],
    });

    this.httpResponseSize = new client.Histogram({
      name: 'http_response_size_bytes',
      help: 'HTTP response size in bytes',
      labelNames: ['method', 'route'],
      buckets: [100, 1000, 5000, 10000, 50000, 100000, 500000, 1000000],
      registers: [this.register],
    });

    // Business metrics
    this.userSignups = new client.Counter({
      name: 'user_signups_total',
      help: 'Total user signups',
      labelNames: ['method'],  // email, google, facebook
      registers: [this.register],
    });

    this.ordersTotal = new client.Counter({
      name: 'orders_total',
      help: 'Total orders',
      labelNames: ['status'],  // completed, cancelled, refunded
      registers: [this.register],
    });

    this.paymentAmount = new client.Histogram({
      name: 'payment_amount_dollars',
      help: 'Payment amount in dollars',
      buckets: [10, 50, 100, 250, 500, 1000, 5000],
      registers: [this.register],
    });

    // Resource metrics
    this.activeConnections = new client.Gauge({
      name: 'active_connections',
      help: 'Active connections',
      registers: [this.register],
    });

    this.databaseConnections = new client.Gauge({
      name: 'database_connections',
      help: 'Database connection pool usage',
      labelNames: ['state'],  // idle, active, waiting
      registers: [this.register],
    });

    this.cacheHitRate = new client.Gauge({
      name: 'cache_hit_rate',
      help: 'Cache hit rate percentage',
      registers: [this.register],
    });
  }

  // HTTP metrics
  recordRequest(method: string, route: string, statusCode: number, duration: number, requestSize: number, responseSize: number) {
    this.httpRequestsTotal.inc({ method, route, status_code: statusCode });
    this.httpRequestDuration.observe({ method, route, status_code: statusCode }, duration / 1000);
    this.httpRequestSize.observe({ method, route }, requestSize);
    this.httpResponseSize.observe({ method, route }, responseSize);
  }

  // Business metrics
  recordUserSignup(method: string) {
    this.userSignups.inc({ method });
  }

  recordOrder(status: string, amount: number) {
    this.ordersTotal.inc({ status });
    this.paymentAmount.observe(amount);
  }

  // Resource metrics
  setActiveConnections(count: number) {
    this.activeConnections.set(count);
  }

  setDatabaseConnections(idle: number, active: number, waiting: number) {
    this.databaseConnections.set({ state: 'idle' }, idle);
    this.databaseConnections.set({ state: 'active' }, active);
    this.databaseConnections.set({ state: 'waiting' }, waiting);
  }

  setCacheHitRate(rate: number) {
    this.cacheHitRate.set(rate);
  }

  getMetrics(): Promise<string> {
    return this.register.metrics();
  }
}

// metrics/metrics.interceptor.ts
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';
import { MetricsService } from './metrics.service';

@Injectable()
export class MetricsInterceptor implements NestInterceptor {
  constructor(private metricsService: MetricsService) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const start = Date.now();
    const request = context.switchToHttp().getRequest();
    const response = context.switchToHttp().getResponse();

    const requestSize = JSON.stringify(request.body || {}).length;

    return next.handle().pipe(
      tap((data) => {
        const duration = Date.now() - start;
        const responseSize = JSON.stringify(data || {}).length;

        this.metricsService.recordRequest(
          request.method,
          request.route?.path || request.path,
          response.statusCode,
          duration,
          requestSize,
          responseSize,
        );
      }),
    );
  }
}

// docker-compose.yml
version: '3.8'
services:
  nestjs-app:
    build: .
    ports:
      - '3000:3000'

  prometheus:
    image: prom/prometheus:latest
    ports:
      - '9090:9090'
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./prometheus-alerts.yml:/etc/prometheus/prometheus-alerts.yml
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'

  grafana:
    image: grafana/grafana:latest
    ports:
      - '3001:3000'
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana-storage:/var/lib/grafana

volumes:
  grafana-storage:
```

---

### **Best Practices:**

```typescript
// ✅ GOOD - Use histogram for latency (not summary)
const histogram = new client.Histogram({
  name: 'http_request_duration_seconds',
  buckets: [0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5],
});
// Can aggregate across instances, calculate percentiles in Prometheus

// ✅ GOOD - Choose appropriate bucket ranges
buckets: [0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5]
// Covers 5ms to 5s with good granularity

// ✅ GOOD - Use labels for dimensions
labelNames: ['method', 'route', 'status_code']
// Can filter: http_requests_total{route="/api/users"}

// ✅ GOOD - Set reasonable scrape interval
scrape_interval: 10s  # 10-15 seconds is good

// ✅ GOOD - Expose /metrics endpoint (not /api/metrics)
@Get('metrics')  // Standard Prometheus convention

// ✅ GOOD - Include default metrics
client.collectDefaultMetrics({ register: this.register });
// CPU, memory, event loop, GC, etc.

// ❌ BAD - High cardinality labels
labelNames: ['user_id']  // Could have millions of values!
// Prometheus will struggle with too many unique label combinations

// ❌ BAD - Using gauge for totals
const totalRequests = new client.Gauge({
  name: 'http_requests_total',  // Should be Counter!
});
// Gauges can go up and down, counters only go up

// ❌ BAD - Too many buckets
buckets: [0.001, 0.002, 0.003, 0.004, ...]  // 100 buckets
// Creates huge cardinality, slows down queries

// ❌ BAD - Not setting Content-Type header
@Get('metrics')
getMetrics() {
  return this.metricsService.getMetrics();
}
// Prometheus expects: Content-Type: text/plain; version=0.0.4

// ❌ BAD - Blocking operations in metrics collection
await database.query('SELECT * FROM users');  // Don't query DB for metrics!
// Metrics should be fast (<10ms), not trigger slow operations
```

**Key Takeaway:** Integrate Prometheus using `prom-client` with **counters** for totals (http_requests_total), **gauges** for current values (active_connections), **histograms** for distributions (request_duration_seconds with buckets), expose a **/metrics** endpoint, configure Prometheus to scrape every 10-15 seconds, and visualize in **Grafana** with PromQL queries.

</details>

<details>
<summary><strong>38. What is the RED method (Rate, Errors, Duration)?</strong></summary>

**Answer:**

The **RED method** (Rate, Errors, Duration) is a monitoring framework for **request-driven services** (APIs, web servers, microservices). Track **Rate** (requests per second), **Errors** (error rate percentage), and **Duration** (response time distribution) to understand service health and performance.

---

### **RED Method Components:**

```typescript
// RED Method for Request-Driven Services

1. RATE (R)
   - How many requests per second?
   - Throughput, traffic volume
   - Metric: requests/second, transactions/minute
   - Why: Understand load and capacity

2. ERRORS (E)
   - What percentage of requests fail?
   - Error rate, success rate
   - Metric: errors/second, error percentage
   - Why: Reliability and user experience

3. DURATION (D)
   - How long do requests take?
   - Latency, response time
   - Metric: p50, p95, p99 percentiles
   - Why: Performance and user satisfaction
```

---

### **Method 1: Rate (Requests per Second)**

```typescript
// metrics/metrics.service.ts
import { Injectable } from '@nestjs/common';
import * as client from 'prom-client';

@Injectable()
export class MetricsService {
  private readonly register: client.Registry;
  private httpRequestsTotal: client.Counter;

  constructor() {
    this.register = new client.Registry();

    // Counter for total requests (RATE)
    this.httpRequestsTotal = new client.Counter({
      name: 'http_requests_total',
      help: 'Total HTTP requests',
      labelNames: ['method', 'route', 'status_code'],
      registers: [this.register],
    });
  }

  incrementRequestCount(method: string, route: string, statusCode: number) {
    this.httpRequestsTotal.inc({ method, route, status_code: statusCode });
  }
}

// Prometheus Query (PromQL):
// Overall request rate (last 5 minutes)
rate(http_requests_total[5m])
// Result: 234.5 requests/second

// Request rate by endpoint
sum by (route) (rate(http_requests_total[5m]))
// Result:
// /api/users: 120 req/s
// /api/orders: 80 req/s
// /api/products: 34.5 req/s

// Request rate by status code
sum by (status_code) (rate(http_requests_total[5m]))
// Result:
// 200: 220 req/s
// 201: 10 req/s
// 404: 3 req/s
// 500: 1.5 req/s

// Total requests in last hour
increase(http_requests_total[1h])
// Result: 843,000 requests
```

---

### **Method 2: Errors (Error Rate)**

```typescript
// metrics/metrics.service.ts
import { Injectable } from '@nestjs/common';
import * as client from 'prom-client';

@Injectable()
export class MetricsService {
  private readonly register: client.Registry;
  private httpRequestsTotal: client.Counter;

  constructor() {
    this.register = new client.Registry();

    this.httpRequestsTotal = new client.Counter({
      name: 'http_requests_total',
      help: 'Total HTTP requests',
      labelNames: ['method', 'route', 'status_code'],
      registers: [this.register],
    });
  }

  recordRequest(method: string, route: string, statusCode: number) {
    this.httpRequestsTotal.inc({ method, route, status_code: statusCode });
  }
}

// Prometheus Query (PromQL):
// Overall error rate (5xx errors)
sum(rate(http_requests_total{status_code=~"5.."}[5m])) 
/ 
sum(rate(http_requests_total[5m])) 
* 100
// Result: 0.64% (0.64% of requests are 5xx errors)

// Success rate (2xx + 3xx)
(
  sum(rate(http_requests_total{status_code=~"2..|3.."}[5m]))
  /
  sum(rate(http_requests_total[5m]))
) * 100
// Result: 99.36% success rate

// Error rate by endpoint
sum by (route) (
  rate(http_requests_total{status_code=~"5.."}[5m])
) 
/ 
sum by (route) (
  rate(http_requests_total[5m])
) * 100
// Result:
// /api/users: 0.1%
// /api/orders: 2.5%  ⚠️ High error rate!
// /api/products: 0.3%

// Count of 4xx errors (client errors)
sum(rate(http_requests_total{status_code=~"4.."}[5m]))
// Result: 12.3 req/s

// Count of 5xx errors (server errors)
sum(rate(http_requests_total{status_code=~"5.."}[5m]))
// Result: 1.5 req/s
```

---

### **Method 3: Duration (Response Time)**

```typescript
// metrics/metrics.service.ts
import { Injectable } from '@nestjs/common';
import * as client from 'prom-client';

@Injectable()
export class MetricsService {
  private readonly register: client.Registry;
  private httpRequestDuration: client.Histogram;

  constructor() {
    this.register = new client.Registry();

    // Histogram for request duration (DURATION)
    this.httpRequestDuration = new client.Histogram({
      name: 'http_request_duration_seconds',
      help: 'HTTP request duration',
      labelNames: ['method', 'route', 'status_code'],
      buckets: [0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5],
      registers: [this.register],
    });
  }

  recordDuration(method: string, route: string, statusCode: number, durationMs: number) {
    this.httpRequestDuration.observe(
      { method, route, status_code: statusCode },
      durationMs / 1000,
    );
  }
}

// Prometheus Query (PromQL):
// Average response time
rate(http_request_duration_seconds_sum[5m]) 
/ 
rate(http_request_duration_seconds_count[5m])
// Result: 0.145 seconds (145ms average)

// p50 (median) - 50% of requests faster than this
histogram_quantile(0.50, rate(http_request_duration_seconds_bucket[5m]))
// Result: 0.089 seconds (89ms)

// p95 - 95% of requests faster than this
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
// Result: 0.340 seconds (340ms)

// p99 - 99% of requests faster than this
histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))
// Result: 0.890 seconds (890ms)

// p99.9 - 99.9% of requests faster than this
histogram_quantile(0.999, rate(http_request_duration_seconds_bucket[5m]))
// Result: 2.345 seconds (2.3s)

// Duration by endpoint (p95)
histogram_quantile(
  0.95,
  sum by (route, le) (rate(http_request_duration_seconds_bucket[5m]))
)
// Result:
// /api/users: 0.200s
// /api/orders: 0.450s
// /api/products: 0.150s
```

---

### **Method 4: Complete RED Implementation**

```typescript
// metrics/metrics.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import * as client from 'prom-client';

@Injectable()
export class MetricsService implements OnModuleInit {
  private readonly register: client.Registry;

  // R - Rate
  private httpRequestsTotal: client.Counter;

  // E - Errors
  // (Uses same counter, filtered by status_code)

  // D - Duration
  private httpRequestDuration: client.Histogram;

  constructor() {
    this.register = new client.Registry();
    this.initializeMetrics();
  }

  onModuleInit() {
    client.collectDefaultMetrics({ register: this.register });
  }

  private initializeMetrics() {
    // RATE: Total requests
    this.httpRequestsTotal = new client.Counter({
      name: 'http_requests_total',
      help: 'Total HTTP requests',
      labelNames: ['method', 'route', 'status_code'],
      registers: [this.register],
    });

    // DURATION: Response time distribution
    this.httpRequestDuration = new client.Histogram({
      name: 'http_request_duration_seconds',
      help: 'HTTP request duration',
      labelNames: ['method', 'route', 'status_code'],
      buckets: [0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5],
      registers: [this.register],
    });
  }

  // Record request (Rate + Errors + Duration)
  recordRequest(
    method: string,
    route: string,
    statusCode: number,
    durationMs: number,
  ) {
    // RATE
    this.httpRequestsTotal.inc({ method, route, status_code: statusCode });

    // DURATION
    this.httpRequestDuration.observe(
      { method, route, status_code: statusCode },
      durationMs / 1000,
    );

    // ERRORS (implicit - tracked by status_code)
  }

  getMetrics(): Promise<string> {
    return this.register.metrics();
  }
}

// metrics/metrics.interceptor.ts
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';
import { MetricsService } from './metrics.service';

@Injectable()
export class MetricsInterceptor implements NestInterceptor {
  constructor(private metricsService: MetricsService) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const start = Date.now();
    const request = context.switchToHttp().getRequest();
    const response = context.switchToHttp().getResponse();

    return next.handle().pipe(
      tap(() => {
        const duration = Date.now() - start;

        this.metricsService.recordRequest(
          request.method,
          request.route?.path || request.path,
          response.statusCode,
          duration,
        );
      }),
    );
  }
}
```

---

### **Method 5: Grafana Dashboard (RED)**

```json
// Grafana Dashboard JSON
{
  "dashboard": {
    "title": "RED Method - API Performance",
    "panels": [
      {
        "title": "RATE - Requests per Second",
        "type": "graph",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total[5m]))",
            "legendFormat": "Total Requests/sec"
          },
          {
            "expr": "sum by (route) (rate(http_requests_total[5m]))",
            "legendFormat": "{{route}}"
          }
        ]
      },
      {
        "title": "ERRORS - Error Rate (%)",
        "type": "graph",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total{status_code=~\"5..\"}[5m])) / sum(rate(http_requests_total[5m])) * 100",
            "legendFormat": "5xx Error Rate"
          },
          {
            "expr": "sum(rate(http_requests_total{status_code=~\"4..\"}[5m])) / sum(rate(http_requests_total[5m])) * 100",
            "legendFormat": "4xx Error Rate"
          }
        ],
        "yaxes": [
          { "format": "percent" }
        ]
      },
      {
        "title": "DURATION - Response Time Percentiles",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.50, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "p50 (median)"
          },
          {
            "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "p95"
          },
          {
            "expr": "histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "p99"
          }
        ],
        "yaxes": [
          { "format": "s" }
        ]
      },
      {
        "title": "Request Distribution by Status Code",
        "type": "piechart",
        "targets": [
          {
            "expr": "sum by (status_code) (rate(http_requests_total[5m]))"
          }
        ]
      }
    ]
  }
}
```

---

### **Method 6: Alert Rules (RED)**

```yaml
# prometheus-alerts.yml
groups:
  - name: red_method_alerts
    interval: 30s
    rules:
      # RATE - Traffic drop (possible outage)
      - alert: LowRequestRate
        expr: sum(rate(http_requests_total[5m])) < 10
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Low request rate detected"
          description: "Only {{ $value }} requests/sec (expected >100)"

      # RATE - Traffic spike
      - alert: HighRequestRate
        expr: sum(rate(http_requests_total[5m])) > 1000
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High request rate"
          description: "{{ $value }} requests/sec (consider scaling)"

      # ERRORS - High error rate
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status_code=~"5.."}[5m])) 
          / 
          sum(rate(http_requests_total[5m])) 
          > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate"
          description: "{{ $value | humanizePercentage }} error rate (expected <1%)"

      # ERRORS - High error rate on specific endpoint
      - alert: EndpointHighErrorRate
        expr: |
          sum by (route) (
            rate(http_requests_total{status_code=~"5.."}[5m])
          ) 
          / 
          sum by (route) (
            rate(http_requests_total[5m])
          ) > 0.1
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate on {{$labels.route}}"
          description: "{{ $value | humanizePercentage }} error rate"

      # DURATION - High p99 latency
      - alert: HighLatencyP99
        expr: |
          histogram_quantile(
            0.99, 
            rate(http_request_duration_seconds_bucket[5m])
          ) > 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High p99 latency"
          description: "p99 latency is {{ $value }}s (expected <1s)"

      # DURATION - High p95 latency
      - alert: HighLatencyP95
        expr: |
          histogram_quantile(
            0.95, 
            rate(http_request_duration_seconds_bucket[5m])
          ) > 0.5
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High p95 latency"
          description: "p95 latency is {{ $value }}s (expected <500ms)"

      # DURATION - Slow endpoint
      - alert: SlowEndpoint
        expr: |
          histogram_quantile(
            0.95,
            sum by (route, le) (rate(http_request_duration_seconds_bucket[5m]))
          ) > 2
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Slow endpoint: {{$labels.route}}"
          description: "p95 latency is {{ $value }}s"
```

---

### **Method 7: RED Dashboard Example**

```
RED Method Dashboard
┌────────────────────────────────────────────────────────────┐
│  RATE - Requests per Second (last 1h)                     │
├────────────────────────────────────────────────────────────┤
│  Current: 234.5 req/s   ↑ 12% from 1h ago                 │
│  [Line chart showing trend]                                │
│  Peak: 456 req/s (at 14:23)                                │
│  By endpoint:                                              │
│    /api/users: 120 req/s (51%)                             │
│    /api/orders: 80 req/s (34%)                             │
│    /api/products: 34.5 req/s (15%)                         │
├────────────────────────────────────────────────────────────┤
│  ERRORS - Error Rate (last 1h)                             │
├────────────────────────────────────────────────────────────┤
│  5xx Error Rate: 0.64%   ⚠️ (expected <1%)                │
│  4xx Error Rate: 5.2%    ✅ (client errors)                │
│  Success Rate: 94.16%    ⚠️ (expected >99%)                │
│  [Line chart showing error rate over time]                 │
│  Errors by endpoint:                                       │
│    /api/orders: 2.5% ❌ (investigate!)                     │
│    /api/users: 0.1% ✅                                      │
│    /api/products: 0.3% ✅                                   │
├────────────────────────────────────────────────────────────┤
│  DURATION - Response Time Percentiles (last 1h)            │
├────────────────────────────────────────────────────────────┤
│  p50 (median): 89ms      ✅                                 │
│  p95: 340ms              ✅                                 │
│  p99: 890ms              ⚠️ (expected <500ms)              │
│  p99.9: 2.3s             ❌ (expected <1s)                 │
│  [Line chart showing percentiles over time]                │
│  Slowest endpoints (p95):                                  │
│    /api/orders: 450ms                                      │
│    /api/users: 200ms                                       │
│    /api/products: 150ms                                    │
└────────────────────────────────────────────────────────────┘
```

---

### **RED vs USE vs Four Golden Signals:**

```typescript
// RED Method - For REQUEST-DRIVEN services (APIs, web servers)
1. Rate - Requests per second
2. Errors - Error rate
3. Duration - Response time

Use for: REST APIs, GraphQL APIs, gRPC services, web servers

// USE Method - For RESOURCES (CPU, memory, disk, network)
1. Utilization - How busy (CPU %)
2. Saturation - How full (queue length)
3. Errors - Error count

Use for: Infrastructure monitoring, capacity planning

// Four Golden Signals (Google SRE)
1. Latency - Response time
2. Traffic - Requests per second
3. Errors - Error rate
4. Saturation - Resource usage

Use for: Comprehensive monitoring (combines RED + USE)

// Comparison:
RED Method ≈ First 3 Golden Signals
USE Method ≈ Resource-focused monitoring
Golden Signals = RED + Saturation
```

---

### **Best Practices:**

```typescript
// ✅ GOOD - Track all three RED metrics
rate(http_requests_total[5m])  // Rate
sum(rate(http_requests_total{status_code=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))  // Errors
histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))  // Duration

// ✅ GOOD - Use percentiles for duration (not averages)
p50, p95, p99  // Shows distribution
// Average hides outliers!

// ✅ GOOD - Alert on error rate, not error count
error_rate = errors / total_requests
// 10 errors/sec is fine at 10,000 req/sec (0.1%)
// 10 errors/sec is terrible at 100 req/sec (10%)

// ✅ GOOD - Monitor by endpoint
sum by (route) (rate(http_requests_total[5m]))
// Identify problematic endpoints

// ✅ GOOD - Use time windows (5m, 15m, 1h)
rate(http_requests_total[5m])  // Recent rate
rate(http_requests_total[1h])  // Trend

// ❌ BAD - Only tracking rate
// Missing errors and duration!

// ❌ BAD - Using average for duration
avg(http_request_duration_seconds)
// Hides slow requests! Use percentiles.

// ❌ BAD - Alert on error count (not rate)
sum(rate(http_request_errors_total[5m])) > 10
// Should be: error_rate > 0.01 (1%)

// ❌ BAD - No labels
http_requests_total  // Can't distinguish endpoints
// Use: http_requests_total{route="/api/users"}
```

**Key Takeaway:** The **RED method** tracks **Rate** (requests/sec with `rate(http_requests_total[5m])`), **Errors** (error rate % with `sum(rate({status_code=~"5.."})) / sum(rate(total))`), and **Duration** (response time percentiles with `histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))`) to monitor request-driven services. Use for APIs, web servers, and microservices.

</details>

<details>
<summary><strong>39. What is distributed tracing?</strong></summary>

**Answer:**

Distributed tracing tracks a **request's journey** across multiple services in a microservices architecture. It creates a **trace** (end-to-end flow) composed of **spans** (individual operations) to identify bottlenecks, understand dependencies, and debug issues across service boundaries.

---

### **Distributed Tracing Concepts:**

```typescript
// Trace: Complete request flow from start to finish
Trace ID: abc123  // Unique identifier for entire request

// Span: Individual operation within the trace
Span 1: API Gateway        [======] 250ms
Span 2: └─ Auth Service     [==] 50ms
Span 3: └─ Order Service    [========] 150ms
  Span 4: └─ Database Query [====] 80ms
  Span 5: └─ Payment API    [===] 60ms
Span 6: └─ Email Service    [=] 30ms

// Each span contains:
- Trace ID (abc123)
- Span ID (unique for this span)
- Parent Span ID (creates hierarchy)
- Start time & duration
- Service name
- Operation name
- Tags (metadata)
- Logs (events within span)
```

---

### **Method 1: OpenTelemetry Setup (Industry Standard)**

```typescript
// Install OpenTelemetry packages
npm install @opentelemetry/api \
  @opentelemetry/sdk-node \
  @opentelemetry/auto-instrumentations-node \
  @opentelemetry/exporter-jaeger

// tracing.ts - Initialize before app starts
import { NodeSDK } from '@opentelemetry/sdk-node';
import { JaegerExporter } from '@opentelemetry/exporter-jaeger';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';
import { Resource } from '@opentelemetry/resources';
import { SemanticResourceAttributes } from '@opentelemetry/semantic-conventions';

const jaegerExporter = new JaegerExporter({
  endpoint: 'http://localhost:14268/api/traces',
});

const sdk = new NodeSDK({
  resource: new Resource({
    [SemanticResourceAttributes.SERVICE_NAME]: 'nestjs-api',
    [SemanticResourceAttributes.SERVICE_VERSION]: '1.0.0',
  }),
  traceExporter: jaegerExporter,
  instrumentations: [
    getNodeAutoInstrumentations({
      // Automatically instruments HTTP, Express, TypeORM, etc.
      '@opentelemetry/instrumentation-http': { enabled: true },
      '@opentelemetry/instrumentation-express': { enabled: true },
      '@opentelemetry/instrumentation-pg': { enabled: true },  // PostgreSQL
    }),
  ],
});

sdk.start();

// Graceful shutdown
process.on('SIGTERM', () => {
  sdk.shutdown()
    .then(() => console.log('Tracing terminated'))
    .catch((error) => console.log('Error terminating tracing', error));
});

// main.ts - Import tracing before NestFactory
import './tracing';  // Must be first!
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();

// HTTP requests are now automatically traced!
```

---

### **Method 2: Manual Span Creation**

```typescript
// users/users.service.ts
import { Injectable } from '@nestjs/common';
import { trace, context, SpanStatusCode } from '@opentelemetry/api';

@Injectable()
export class UsersService {
  private readonly tracer = trace.getTracer('users-service');

  async findUserById(userId: string) {
    // Create a custom span
    return await this.tracer.startActiveSpan(
      'findUserById',
      async (span) => {
        try {
          // Add attributes (tags)
          span.setAttribute('user.id', userId);
          span.setAttribute('db.system', 'postgresql');

          // Add event (log within span)
          span.addEvent('Starting database query');

          const user = await this.database.query(
            'SELECT * FROM users WHERE id = $1',
            [userId],
          );

          span.addEvent('Database query completed', {
            'db.rows_returned': user ? 1 : 0,
          });

          // Set status
          if (!user) {
            span.setStatus({
              code: SpanStatusCode.ERROR,
              message: 'User not found',
            });
          } else {
            span.setStatus({ code: SpanStatusCode.OK });
          }

          return user;
        } catch (error) {
          // Record error
          span.recordException(error);
          span.setStatus({
            code: SpanStatusCode.ERROR,
            message: error.message,
          });
          throw error;
        } finally {
          // Always end span
          span.end();
        }
      },
    );
  }
}
```

---

### **Method 3: Trace Context Propagation**

```typescript
// Propagate trace context to downstream services

// http.service.ts
import { Injectable } from '@nestjs/common';
import { HttpService } from '@nestjs/axios';
import { trace, context, propagation } from '@opentelemetry/api';
import { firstValueFrom } from 'rxjs';

@Injectable()
export class CustomHttpService {
  constructor(private readonly httpService: HttpService) {}

  async get(url: string) {
    const tracer = trace.getTracer('http-client');

    return await tracer.startActiveSpan('http.get', async (span) => {
      try {
        span.setAttribute('http.method', 'GET');
        span.setAttribute('http.url', url);

        // Inject trace context into HTTP headers
        const headers: Record<string, string> = {};
        propagation.inject(context.active(), headers);

        const response = await firstValueFrom(
          this.httpService.get(url, { headers }),
        );

        span.setAttribute('http.status_code', response.status);
        span.setStatus({ code: SpanStatusCode.OK });

        return response.data;
      } catch (error) {
        span.recordException(error);
        span.setStatus({ code: SpanStatusCode.ERROR });
        throw error;
      } finally {
        span.end();
      }
    });
  }
}

// Headers injected:
// traceparent: 00-abc123def456-0123456789abcdef-01
// tracestate: key1=value1,key2=value2
```

---

### **Method 4: Database Query Tracing**

```typescript
// TypeORM automatic instrumentation (via OpenTelemetry)
// queries are automatically traced when using:
// @opentelemetry/instrumentation-typeorm

// Manual tracing for complex queries
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { trace } from '@opentelemetry/api';
import { User } from './user.entity';

@Injectable()
export class UsersService {
  private readonly tracer = trace.getTracer('users-service');

  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>,
  ) {}

  async findActiveUsers() {
    return await this.tracer.startActiveSpan(
      'database.findActiveUsers',
      async (span) => {
        try {
          span.setAttribute('db.system', 'postgresql');
          span.setAttribute('db.operation', 'SELECT');
          span.setAttribute('db.table', 'users');

          const start = Date.now();

          const users = await this.usersRepository
            .createQueryBuilder('user')
            .where('user.isActive = :isActive', { isActive: true })
            .orderBy('user.createdAt', 'DESC')
            .limit(100)
            .getMany();

          const duration = Date.now() - start;

          span.setAttribute('db.rows_returned', users.length);
          span.setAttribute('db.duration_ms', duration);

          if (duration > 1000) {
            span.addEvent('Slow query detected', { duration_ms: duration });
          }

          span.setStatus({ code: SpanStatusCode.OK });
          return users;
        } catch (error) {
          span.recordException(error);
          span.setStatus({ code: SpanStatusCode.ERROR });
          throw error;
        } finally {
          span.end();
        }
      },
    );
  }
}
```

---

### **Method 5: Custom Interceptor for Tracing**

```typescript
// tracing/tracing.interceptor.ts
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';
import { trace, context, SpanStatusCode } from '@opentelemetry/api';

@Injectable()
export class TracingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const tracer = trace.getTracer('http-server');
    const request = context.switchToHttp().getRequest();
    const response = context.switchToHttp().getResponse();

    const spanName = `${request.method} ${request.route?.path || request.path}`;

    return new Observable((observer) => {
      tracer.startActiveSpan(spanName, (span) => {
        // Add request attributes
        span.setAttribute('http.method', request.method);
        span.setAttribute('http.url', request.url);
        span.setAttribute('http.route', request.route?.path);
        span.setAttribute('http.user_agent', request.headers['user-agent']);

        if (request.user?.id) {
          span.setAttribute('user.id', request.user.id);
        }

        next
          .handle()
          .pipe(
            tap({
              next: (data) => {
                span.setAttribute('http.status_code', response.statusCode);
                span.setStatus({ code: SpanStatusCode.OK });
                observer.next(data);
                observer.complete();
              },
              error: (error) => {
                span.recordException(error);
                span.setAttribute('http.status_code', response.statusCode || 500);
                span.setStatus({
                  code: SpanStatusCode.ERROR,
                  message: error.message,
                });
                observer.error(error);
              },
              complete: () => {
                span.end();
              },
            }),
          )
          .subscribe();
      });
    });
  }
}

// app.module.ts
import { Module } from '@nestjs/common';
import { APP_INTERCEPTOR } from '@nestjs/core';
import { TracingInterceptor } from './tracing/tracing.interceptor';

@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: TracingInterceptor,
    },
  ],
})
export class AppModule {}
```

---

### **Method 6: Jaeger Integration**

```yaml
# docker-compose.yml
version: '3.8'
services:
  jaeger:
    image: jaegertracing/all-in-one:latest
    ports:
      - '5775:5775/udp'     # Zipkin compact
      - '6831:6831/udp'     # Jaeger compact
      - '6832:6832/udp'     # Jaeger binary
      - '5778:5778'         # Config
      - '16686:16686'       # Web UI
      - '14268:14268'       # HTTP collector
      - '14250:14250'       # gRPC
      - '9411:9411'         # Zipkin
    environment:
      - COLLECTOR_ZIPKIN_HOST_PORT=:9411

  nestjs-api:
    build: .
    ports:
      - '3000:3000'
    environment:
      - JAEGER_ENDPOINT=http://jaeger:14268/api/traces
    depends_on:
      - jaeger

# Access Jaeger UI: http://localhost:16686
```

---

### **Method 7: Trace Visualization**

```
Jaeger UI - Trace View

┌────────────────────────────────────────────────────────────┐
│ Trace: abc123                         Duration: 487ms    │
├────────────────────────────────────────────────────────────┤
│                                                           │
│ [api-gateway] POST /api/orders                            │
│ ╔═════════════════════════════════════════════╗ 487ms │
│                                                           │
│   [auth-service] POST /verify-token                       │
│   ╠═══════════╣ 45ms                                     │
│                                                           │
│   [order-service] POST /orders                            │
│   ╠══════════════════════════════════════════╗ 378ms     │
│                                                           │
│     [database] INSERT INTO orders                         │
│     ╠═════════════════╣ 89ms                            │
│                                                           │
│     [payment-service] POST /charge                        │
│     ╠════════════════════════════╗ 234ms  ⚠️ Slow!     │
│                                                           │
│       [stripe-api] POST /v1/charges                       │
│       ╠═════════════════════╗ 198ms                  │
│                                                           │
│     [inventory-service] PUT /reserve                      │
│     ╠═════════╣ 34ms                                   │
│                                                           │
│   [notification-service] POST /send-email                 │
│   ╠═══════════╣ 52ms                                     │
│                                                           │
└────────────────────────────────────────────────────────────┘

// Click on any span to see details:
- Span ID
- Trace ID
- Start time & duration
- Tags (attributes)
- Logs (events)
- Stack trace (if error)
```

---

### **Method 8: Correlation with Logs**

```typescript
// Inject trace context into logs

// logger/logger.service.ts
import { Injectable, LoggerService as NestLoggerService } from '@nestjs/common';
import { trace, context } from '@opentelemetry/api';
import * as winston from 'winston';

@Injectable()
export class LoggerService implements NestLoggerService {
  private logger: winston.Logger;

  constructor() {
    this.logger = winston.createLogger({
      format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.json(),
      ),
      transports: [new winston.transports.Console()],
    });
  }

  log(message: string, context?: string) {
    const traceInfo = this.getTraceInfo();
    this.logger.info(message, {
      context,
      ...traceInfo,
    });
  }

  error(message: string, trace?: string, context?: string) {
    const traceInfo = this.getTraceInfo();
    this.logger.error(message, {
      context,
      trace,
      ...traceInfo,
    });
  }

  private getTraceInfo() {
    const span = trace.getSpan(context.active());
    if (!span) return {};

    const spanContext = span.spanContext();
    return {
      traceId: spanContext.traceId,
      spanId: spanContext.spanId,
      traceFlags: spanContext.traceFlags,
    };
  }
}

// Log output:
{
  "timestamp": "2025-12-30T10:30:45.123Z",
  "level": "info",
  "message": "Order created successfully",
  "context": "OrdersService",
  "traceId": "abc123def456789",  // Can search logs by trace ID!
  "spanId": "0123456789abcdef"
}

// In Jaeger, click "Logs" tab on span to see correlated logs
```

---

### **Method 9: Sampling Strategy**

```typescript
// tracing.ts - Configure sampling
import { NodeSDK } from '@opentelemetry/sdk-node';
import { TraceIdRatioBasedSampler, ParentBasedSampler } from '@opentelemetry/sdk-trace-base';

const sdk = new NodeSDK({
  // Sampling strategies:

  // 1. Sample 10% of traces (reduce overhead)
  sampler: new TraceIdRatioBasedSampler(0.1),

  // 2. Parent-based sampling (follow parent decision)
  sampler: new ParentBasedSampler({
    root: new TraceIdRatioBasedSampler(0.1),
  }),

  // 3. Always sample (development)
  sampler: new TraceIdRatioBasedSampler(1.0),

  // 4. Never sample (disable tracing)
  sampler: new TraceIdRatioBasedSampler(0.0),

  // Production recommendation:
  sampler: new ParentBasedSampler({
    root: new TraceIdRatioBasedSampler(0.1),  // Sample 10% of root traces
  }),
});

// Custom sampler based on route
import { Sampler, SamplingDecision } from '@opentelemetry/sdk-trace-base';

class RouteBasedSampler implements Sampler {
  shouldSample(context, traceId, spanName) {
    // Always sample errors
    if (spanName.includes('error')) {
      return { decision: SamplingDecision.RECORD_AND_SAMPLED };
    }

    // Always sample critical endpoints
    if (spanName.includes('/api/orders') || spanName.includes('/api/payments')) {
      return { decision: SamplingDecision.RECORD_AND_SAMPLED };
    }

    // Sample 10% of other requests
    return Math.random() < 0.1
      ? { decision: SamplingDecision.RECORD_AND_SAMPLED }
      : { decision: SamplingDecision.NOT_RECORD };
  }
}
```

---

### **Method 10: Complete Production Setup**

```typescript
// tracing/tracing.module.ts
import { Module, Global } from '@nestjs/common';
import { APP_INTERCEPTOR } from '@nestjs/core';
import { TracingInterceptor } from './tracing.interceptor';
import { TracingService } from './tracing.service';

@Global()
@Module({
  providers: [
    TracingService,
    {
      provide: APP_INTERCEPTOR,
      useClass: TracingInterceptor,
    },
  ],
  exports: [TracingService],
})
export class TracingModule {}

// tracing/tracing.service.ts
import { Injectable } from '@nestjs/common';
import { trace, context, Span, SpanStatusCode } from '@opentelemetry/api';

@Injectable()
export class TracingService {
  private readonly tracer = trace.getTracer('app-tracer');

  async traceAsync<T>(
    spanName: string,
    fn: (span: Span) => Promise<T>,
    attributes?: Record<string, any>,
  ): Promise<T> {
    return await this.tracer.startActiveSpan(spanName, async (span) => {
      try {
        if (attributes) {
          Object.entries(attributes).forEach(([key, value]) => {
            span.setAttribute(key, value);
          });
        }

        const result = await fn(span);
        span.setStatus({ code: SpanStatusCode.OK });
        return result;
      } catch (error) {
        span.recordException(error);
        span.setStatus({
          code: SpanStatusCode.ERROR,
          message: error.message,
        });
        throw error;
      } finally {
        span.end();
      }
    });
  }

  getCurrentSpan(): Span | undefined {
    return trace.getSpan(context.active());
  }

  addEvent(name: string, attributes?: Record<string, any>) {
    const span = this.getCurrentSpan();
    if (span) {
      span.addEvent(name, attributes);
    }
  }

  setAttribute(key: string, value: any) {
    const span = this.getCurrentSpan();
    if (span) {
      span.setAttribute(key, value);
    }
  }
}

// Usage in service
import { Injectable } from '@nestjs/common';
import { TracingService } from '../tracing/tracing.service';

@Injectable()
export class OrdersService {
  constructor(private tracingService: TracingService) {}

  async createOrder(orderData: any) {
    return this.tracingService.traceAsync(
      'createOrder',
      async (span) => {
        span.setAttribute('order.amount', orderData.amount);
        span.setAttribute('order.items_count', orderData.items.length);

        // Validate order
        this.tracingService.addEvent('Validating order');
        await this.validateOrder(orderData);

        // Save to database
        this.tracingService.addEvent('Saving to database');
        const order = await this.ordersRepository.save(orderData);

        // Process payment
        this.tracingService.addEvent('Processing payment');
        await this.paymentsService.charge(order.id, order.amount);

        return order;
      },
      { 'service.name': 'orders' },
    );
  }
}
```

---

### **Best Practices:**

```typescript
// ✅ GOOD - Use OpenTelemetry (industry standard)
import { NodeSDK } from '@opentelemetry/sdk-node';
// Works with Jaeger, Zipkin, Tempo, Datadog, New Relic, etc.

// ✅ GOOD - Propagate trace context across services
const headers = {};
propagation.inject(context.active(), headers);
// Maintains trace ID across service boundaries

// ✅ GOOD - Add meaningful attributes
span.setAttribute('user.id', userId);
span.setAttribute('order.total', total);
span.setAttribute('db.query_duration_ms', duration);

// ✅ GOOD - Sample traces in production (10-20%)
sampler: new TraceIdRatioBasedSampler(0.1)
// Reduces overhead and storage costs

// ✅ GOOD - Always sample errors
if (error) {
  return { decision: SamplingDecision.RECORD_AND_SAMPLED };
}

// ✅ GOOD - Correlate traces with logs
logger.info('Order created', {
  traceId: span.spanContext().traceId,
  spanId: span.spanContext().spanId,
});

// ❌ BAD - Not propagating trace context
axios.get(url);  // No traceparent header!
// Trace breaks at service boundary

// ❌ BAD - Sampling 100% in production
sampler: new TraceIdRatioBasedSampler(1.0)
// Huge performance overhead and storage costs

// ❌ BAD - High cardinality attributes
span.setAttribute('email', user.email);  // Millions of values!
// Use: span.setAttribute('user.id', user.id);

// ❌ BAD - Forgetting to end spans
const span = tracer.startSpan('operation');
// ... operation ...
// span.end() never called! Memory leak!

// ❌ BAD - Not using auto-instrumentation
// Manually instrumenting every HTTP call, DB query
// Use: getNodeAutoInstrumentations()
```

**Key Takeaway:** Distributed tracing tracks requests across microservices using **OpenTelemetry** (industry standard). Initialize with `NodeSDK`, use **auto-instrumentation** for HTTP/DB, create **custom spans** for business logic, **propagate trace context** via HTTP headers (`traceparent`), **sample 10-20%** in production, visualize in **Jaeger**, and **correlate with logs** using trace IDs.

</details>

## Best Practices

<details>
<summary><strong>40. Should you log sensitive information (passwords, tokens)?</strong></summary>

**Answer:**

**NO, never log sensitive information** like passwords, tokens, credit cards, SSNs, or PII (Personally Identifiable Information). Use **redaction**, **sanitization**, **masking**, and **filtering** to prevent sensitive data from appearing in logs.

---

### **What NOT to Log:**

```typescript
// ❌ NEVER LOG:

1. Authentication Credentials
   - Passwords (plaintext or hashed)
   - API keys / tokens
   - OAuth tokens (access/refresh)
   - Session IDs
   - JWT tokens
   - Private keys / certificates

2. Financial Information
   - Credit card numbers
   - CVV codes
   - Bank account numbers
   - Routing numbers
   - Payment tokens

3. Personal Identifiable Information (PII)
   - Social Security Numbers (SSN)
   - Driver's license numbers
   - Passport numbers
   - Date of birth
   - Full addresses
   - Phone numbers (sometimes)
   - Email addresses (sometimes, depends on policy)

4. Health Information (PHI - Protected Health Information)
   - Medical records
   - Diagnoses
   - Prescriptions
   - Insurance information

5. Legal/Compliance Data
   - Trade secrets
   - Confidential business data
   - Encryption keys
   - Private customer data
```

---

### **Method 1: Redaction with Winston**

```typescript
// Install
npm install winston-logstash-format

// logger/logger.service.ts
import * as winston from 'winston';

const sensitiveFields = [
  'password',
  'token',
  'apiKey',
  'api_key',
  'accessToken',
  'access_token',
  'refreshToken',
  'refresh_token',
  'creditCard',
  'credit_card',
  'ssn',
  'cvv',
];

const redactSensitiveData = winston.format((info) => {
  const redacted = JSON.parse(JSON.stringify(info));

  const redactObject = (obj: any) => {
    for (const key in obj) {
      if (typeof obj[key] === 'object' && obj[key] !== null) {
        redactObject(obj[key]);  // Recursive
      } else if (sensitiveFields.some(field => key.toLowerCase().includes(field.toLowerCase()))) {
        obj[key] = '[REDACTED]';
      }
    }
  };

  redactObject(redacted);
  return redacted;
});

export const logger = winston.createLogger({
  format: winston.format.combine(
    redactSensitiveData(),  // Apply redaction
    winston.format.timestamp(),
    winston.format.json(),
  ),
  transports: [new winston.transports.Console()],
});

// Usage
logger.info('User login', {
  email: 'user@example.com',
  password: 'secret123',  // Will be redacted
  token: 'abc123',        // Will be redacted
});

// Output:
{
  "timestamp": "2025-12-30T10:30:45.123Z",
  "level": "info",
  "message": "User login",
  "email": "user@example.com",
  "password": "[REDACTED]",
  "token": "[REDACTED]"
}
```

---

### **Method 2: Sanitization Utility**

```typescript
// logger/sanitizer.ts
export class LogSanitizer {
  private static readonly SENSITIVE_KEYS = [
    'password',
    'token',
    'apiKey',
    'api_key',
    'accessToken',
    'access_token',
    'refreshToken',
    'refresh_token',
    'creditCard',
    'credit_card',
    'cardNumber',
    'card_number',
    'cvv',
    'ssn',
    'socialSecurity',
    'privateKey',
    'private_key',
  ];

  static sanitize(data: any): any {
    if (data === null || data === undefined) {
      return data;
    }

    if (typeof data !== 'object') {
      return data;
    }

    if (Array.isArray(data)) {
      return data.map(item => this.sanitize(item));
    }

    const sanitized: any = {};

    for (const [key, value] of Object.entries(data)) {
      if (this.isSensitiveKey(key)) {
        sanitized[key] = '[REDACTED]';
      } else if (typeof value === 'object' && value !== null) {
        sanitized[key] = this.sanitize(value);  // Recursive
      } else {
        sanitized[key] = value;
      }
    }

    return sanitized;
  }

  private static isSensitiveKey(key: string): boolean {
    const lowerKey = key.toLowerCase();
    return this.SENSITIVE_KEYS.some(sensitive =>
      lowerKey.includes(sensitive.toLowerCase()),
    );
  }

  // Mask credit card (show last 4 digits)
  static maskCreditCard(cardNumber: string): string {
    if (!cardNumber || cardNumber.length < 4) {
      return '[REDACTED]';
    }
    return '**** **** **** ' + cardNumber.slice(-4);
  }

  // Mask email (show first char and domain)
  static maskEmail(email: string): string {
    const [local, domain] = email.split('@');
    return `${local[0]}***@${domain}`;
  }

  // Mask phone number
  static maskPhone(phone: string): string {
    if (phone.length < 4) return '[REDACTED]';
    return '***-***-' + phone.slice(-4);
  }
}

// Usage
import { LogSanitizer } from './logger/sanitizer';

const userData = {
  email: 'user@example.com',
  password: 'secret123',
  creditCard: '4111111111111111',
  name: 'John Doe',
};

logger.info('User data', LogSanitizer.sanitize(userData));

// Output:
{
  "email": "user@example.com",
  "password": "[REDACTED]",
  "creditCard": "[REDACTED]",
  "name": "John Doe"
}
```

---

### **Method 3: Custom Winston Format**

```typescript
// logger/formats/redact.format.ts
import { format } from 'winston';
import { LogSanitizer } from '../sanitizer';

export const redactFormat = format((info) => {
  // Sanitize the entire log entry
  return LogSanitizer.sanitize(info);
});

// logger/logger.module.ts
import { Module } from '@nestjs/common';
import { WinstonModule } from 'nest-winston';
import * as winston from 'winston';
import { redactFormat } from './formats/redact.format';

@Module({
  imports: [
    WinstonModule.forRoot({
      format: winston.format.combine(
        redactFormat(),  // Apply redaction first
        winston.format.timestamp(),
        winston.format.json(),
      ),
      transports: [
        new winston.transports.Console(),
        new winston.transports.File({ filename: 'app.log' }),
      ],
    }),
  ],
})
export class LoggerModule {}
```

---

### **Method 4: Interceptor for Request/Response Sanitization**

```typescript
// logger/logging.interceptor.ts
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';
import { LoggerService } from './logger.service';
import { LogSanitizer } from './sanitizer';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  constructor(private logger: LoggerService) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { method, url, body, headers } = request;

    // Sanitize request body (remove passwords, tokens, etc.)
    const sanitizedBody = LogSanitizer.sanitize(body);

    // Sanitize headers (remove Authorization, etc.)
    const sanitizedHeaders = this.sanitizeHeaders(headers);

    this.logger.log('Incoming request', {
      method,
      url,
      body: sanitizedBody,
      headers: sanitizedHeaders,
    });

    return next.handle().pipe(
      tap((data) => {
        // Sanitize response data
        const sanitizedData = LogSanitizer.sanitize(data);

        this.logger.log('Outgoing response', {
          method,
          url,
          statusCode: context.switchToHttp().getResponse().statusCode,
          data: sanitizedData,
        });
      }),
    );
  }

  private sanitizeHeaders(headers: any): any {
    const sanitized = { ...headers };

    // Remove sensitive headers
    const sensitiveHeaders = [
      'authorization',
      'cookie',
      'x-api-key',
      'x-auth-token',
    ];

    sensitiveHeaders.forEach(header => {
      if (sanitized[header]) {
        sanitized[header] = '[REDACTED]';
      }
    });

    return sanitized;
  }
}
```

---

### **Method 5: Masking Specific Fields**

```typescript
// logger/masker.ts
export class DataMasker {
  // Mask credit card (show last 4 digits)
  static maskCreditCard(card: string): string {
    if (!card || card.length < 4) return '[REDACTED]';
    return '**** **** **** ' + card.slice(-4);
  }

  // Mask email (show first letter and domain)
  static maskEmail(email: string): string {
    if (!email || !email.includes('@')) return '[REDACTED]';
    const [local, domain] = email.split('@');
    return `${local[0]}***@${domain}`;
  }

  // Mask SSN (show last 4 digits)
  static maskSSN(ssn: string): string {
    if (!ssn || ssn.length < 4) return '[REDACTED]';
    return '***-**-' + ssn.slice(-4);
  }

  // Mask phone number
  static maskPhone(phone: string): string {
    if (!phone || phone.length < 4) return '[REDACTED]';
    return '***-***-' + phone.slice(-4);
  }

  // Partial token (show first/last few characters)
  static maskToken(token: string): string {
    if (!token || token.length < 8) return '[REDACTED]';
    return `${token.slice(0, 4)}...${token.slice(-4)}`;
  }

  // Generic masking
  static mask(value: string, visible: number = 4): string {
    if (!value || value.length <= visible) return '[REDACTED]';
    return '*'.repeat(value.length - visible) + value.slice(-visible);
  }
}

// Usage
import { DataMasker } from './logger/masker';

const user = {
  email: 'john.doe@example.com',
  creditCard: '4111111111111111',
  ssn: '123-45-6789',
  phone: '555-123-4567',
};

logger.info('User info', {
  email: DataMasker.maskEmail(user.email),           // j***@example.com
  creditCard: DataMasker.maskCreditCard(user.creditCard),  // **** **** **** 1111
  ssn: DataMasker.maskSSN(user.ssn),                 // ***-**-6789
  phone: DataMasker.maskPhone(user.phone),           // ***-***-4567
});
```

---

### **Method 6: Safe Logging Helper**

```typescript
// logger/safe-logger.ts
import { Injectable } from '@nestjs/common';
import { LoggerService } from '@nestjs/common';
import { LogSanitizer } from './sanitizer';
import { DataMasker } from './masker';

@Injectable()
export class SafeLogger implements LoggerService {
  constructor(private baseLogger: LoggerService) {}

  log(message: string, context?: any) {
    const sanitized = LogSanitizer.sanitize(context);
    this.baseLogger.log(message, sanitized);
  }

  error(message: string, trace?: string, context?: any) {
    const sanitized = LogSanitizer.sanitize(context);
    this.baseLogger.error(message, trace, sanitized);
  }

  warn(message: string, context?: any) {
    const sanitized = LogSanitizer.sanitize(context);
    this.baseLogger.warn(message, sanitized);
  }

  debug(message: string, context?: any) {
    const sanitized = LogSanitizer.sanitize(context);
    this.baseLogger.debug(message, sanitized);
  }

  verbose(message: string, context?: any) {
    const sanitized = LogSanitizer.sanitize(context);
    this.baseLogger.verbose(message, sanitized);
  }

  // Safe user logging (masks PII)
  logUser(message: string, user: any) {
    this.log(message, {
      userId: user.id,  // ID is OK
      email: DataMasker.maskEmail(user.email),
      // Don't log password, token, etc.
    });
  }

  // Safe payment logging
  logPayment(message: string, payment: any) {
    this.log(message, {
      paymentId: payment.id,
      amount: payment.amount,
      currency: payment.currency,
      card: DataMasker.maskCreditCard(payment.cardNumber),
      // Don't log CVV, full card number
    });
  }
}
```

---

### **Method 7: Environment-Based Logging**

```typescript
// logger/logger.config.ts
import { ConfigService } from '@nestjs/config';

export const getLoggerConfig = (configService: ConfigService) => {
  const isProduction = configService.get('NODE_ENV') === 'production';

  return {
    level: isProduction ? 'info' : 'debug',
    format: winston.format.combine(
      // In production, always redact sensitive data
      isProduction ? redactFormat() : winston.format.simple(),
      winston.format.timestamp(),
      winston.format.json(),
    ),
    transports: [
      new winston.transports.Console(),
      isProduction
        ? new winston.transports.File({
            filename: 'app.log',
            maxsize: 5242880,  // 5MB
            maxFiles: 5,
          })
        : null,
    ].filter(Boolean),
  };
};

// Development: More verbose, less redaction (for debugging)
// Production: Less verbose, strict redaction (for security)
```

---

### **Method 8: Compliance Checklist**

```typescript
// logger/compliance.ts
export const COMPLIANCE_GUIDELINES = {
  // GDPR (EU)
  gdpr: {
    description: 'General Data Protection Regulation',
    rules: [
      'No PII in logs without explicit consent',
      'Right to be forgotten - can delete user logs',
      'Data minimization - log only necessary data',
      'Pseudonymization - use user IDs instead of names/emails',
    ],
  },

  // PCI-DSS (Payment Card Industry)
  pciDss: {
    description: 'Payment Card Industry Data Security Standard',
    rules: [
      'Never log full credit card numbers (PAN)',
      'Never log CVV/CVV2',
      'Never log PIN/PIN block',
      'Mask card numbers (show last 4 digits only)',
      'Never log magnetic stripe data',
    ],
  },

  // HIPAA (US Healthcare)
  hipaa: {
    description: 'Health Insurance Portability and Accountability Act',
    rules: [
      'No PHI (Protected Health Information) in logs',
      'No patient names, addresses, SSNs',
      'No diagnoses, treatments, prescriptions',
      'Use de-identified data only',
    ],
  },

  // SOC 2 (Security)
  soc2: {
    description: 'Service Organization Control 2',
    rules: [
      'Encrypt logs at rest and in transit',
      'Access control for log viewing',
      'Audit trail of who accessed logs',
      'Retention policy (typically 90-365 days)',
    ],
  },
};

// Checklist before logging:
const isSafeToLog = (data: any): boolean => {
  // 1. Does it contain passwords/tokens? ❌ NO
  // 2. Does it contain PII (name, email, phone)? ❌ NO (or masked)
  // 3. Does it contain payment info (card, CVV)? ❌ NO (or masked)
  // 4. Does it contain health info? ❌ NO
  // 5. Is it necessary for debugging? ✅ YES
  // 6. Is it anonymized/pseudonymized? ✅ YES

  return true;  // Only log if all checks pass
};
```

---

### **Method 9: Testing Sanitization**

```typescript
// logger/sanitizer.spec.ts
import { LogSanitizer } from './sanitizer';

describe('LogSanitizer', () => {
  it('should redact passwords', () => {
    const input = { username: 'john', password: 'secret123' };
    const output = LogSanitizer.sanitize(input);

    expect(output.username).toBe('john');
    expect(output.password).toBe('[REDACTED]');
  });

  it('should redact tokens', () => {
    const input = { userId: 1, accessToken: 'abc123' };
    const output = LogSanitizer.sanitize(input);

    expect(output.userId).toBe(1);
    expect(output.accessToken).toBe('[REDACTED]');
  });

  it('should handle nested objects', () => {
    const input = {
      user: {
        id: 1,
        credentials: {
          password: 'secret',
          token: 'abc',
        },
      },
    };

    const output = LogSanitizer.sanitize(input);

    expect(output.user.id).toBe(1);
    expect(output.user.credentials.password).toBe('[REDACTED]');
    expect(output.user.credentials.token).toBe('[REDACTED]');
  });

  it('should handle arrays', () => {
    const input = {
      users: [
        { id: 1, password: 'secret1' },
        { id: 2, password: 'secret2' },
      ],
    };

    const output = LogSanitizer.sanitize(input);

    expect(output.users[0].password).toBe('[REDACTED]');
    expect(output.users[1].password).toBe('[REDACTED]');
  });
});
```

---

### **Method 10: Complete Production Setup**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { APP_INTERCEPTOR } from '@nestjs/core';
import { LoggingInterceptor } from './logger/logging.interceptor';
import { LoggerModule } from './logger/logger.module';

@Module({
  imports: [LoggerModule],
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: LoggingInterceptor,  // Sanitizes all requests/responses
    },
  ],
})
export class AppModule {}

// auth/auth.service.ts
import { Injectable } from '@nestjs/common';
import { SafeLogger } from '../logger/safe-logger';

@Injectable()
export class AuthService {
  constructor(private logger: SafeLogger) {}

  async login(email: string, password: string) {
    // ✅ GOOD - Don't log password
    this.logger.log('Login attempt', { email });

    // ❌ BAD - Never do this!
    // this.logger.log('Login attempt', { email, password });

    const user = await this.validateUser(email, password);

    if (user) {
      // ✅ GOOD - Log user ID, not full user object (may contain sensitive data)
      this.logger.log('Login successful', { userId: user.id });
    } else {
      this.logger.warn('Login failed', { email });
    }

    return user;
  }

  async createUser(userData: any) {
    // ✅ GOOD - Sanitize before logging
    this.logger.log('Creating user', LogSanitizer.sanitize(userData));

    // ❌ BAD - Never log raw user data
    // this.logger.log('Creating user', userData);  // May contain password!

    const user = await this.usersRepository.save(userData);

    // ✅ GOOD - Only log non-sensitive fields
    this.logger.log('User created', {
      userId: user.id,
      email: DataMasker.maskEmail(user.email),
    });

    return user;
  }
}
```

---

### **Best Practices:**

```typescript
// ✅ GOOD - Use sanitization for all logs
logger.log('User data', LogSanitizer.sanitize(user));

// ✅ GOOD - Mask PII when necessary
email: DataMasker.maskEmail(user.email)  // j***@example.com

// ✅ GOOD - Log IDs instead of sensitive data
logger.log('User login', { userId: user.id })  // Not full user object

// ✅ GOOD - Redact at the format level
winston.format.combine(
  redactFormat(),  // Automatic redaction
  winston.format.json(),
)

// ✅ GOOD - Test your sanitization
expect(sanitized.password).toBe('[REDACTED]');

// ✅ GOOD - Have a compliance checklist
// Does this log contain PII? Payment data? Health info?

// ❌ BAD - Logging passwords
logger.log('Login', { email, password })  // NEVER!

// ❌ BAD - Logging full request/response without sanitization
logger.log('Request', req.body)  // May contain sensitive data

// ❌ BAD - Logging tokens
logger.log('Auth', { token: req.headers.authorization })  // NEVER!

// ❌ BAD - Logging credit cards
logger.log('Payment', { cardNumber: '4111...' })  // NEVER!

// ❌ BAD - Assuming data is safe
logger.log('Data', data)  // Always sanitize first!

// ❌ BAD - No masking for PII
logger.log('User', { email: 'john@example.com' })  // Should mask
```

**Key Takeaway:** **Never log sensitive information** (passwords, tokens, credit cards, SSNs, PII). Use **automatic redaction** with Winston formats, **sanitization utilities** to remove sensitive fields, **masking** for partial data (last 4 digits), **environment-based** logging (stricter in production), and **test** your sanitization. Follow compliance guidelines (GDPR, PCI-DSS, HIPAA).

</details>

<details>
<summary><strong>41. How do you handle log rotation?</strong></summary>

**Answer:**

Log rotation **archives old logs** and **creates new log files** to prevent disk space exhaustion. Use **winston-daily-rotate-file** for time-based rotation, **maxsize** for size-based rotation, **maxFiles** for retention, and **compression** to save space.

---

### **Why Log Rotation is Important:**

```typescript
// Without rotation:
app.log (50GB!)  // Eventually fills disk, crashes app

// With rotation:
app-2025-12-30.log (100MB)
app-2025-12-29.log.gz (20MB compressed)
app-2025-12-28.log.gz (20MB compressed)
app-2025-12-27.log.gz (20MB compressed)
// Older logs deleted after 30 days

// Benefits:
1. Prevents disk space exhaustion
2. Improves log search performance (smaller files)
3. Easier to manage and backup
4. Compliance with retention policies
5. Compressed archives save space
```

---

### **Method 1: Winston Daily Rotate File**

```typescript
// Install
npm install winston-daily-rotate-file

// logger/logger.module.ts
import { Module } from '@nestjs/common';
import { WinstonModule } from 'nest-winston';
import * as winston from 'winston';
import * as DailyRotateFile from 'winston-daily-rotate-file';

@Module({
  imports: [
    WinstonModule.forRoot({
      transports: [
        // Console (no rotation needed)
        new winston.transports.Console({
          format: winston.format.combine(
            winston.format.colorize(),
            winston.format.simple(),
          ),
        }),

        // Daily rotate file for all logs
        new DailyRotateFile({
          filename: 'logs/app-%DATE%.log',
          datePattern: 'YYYY-MM-DD',
          zippedArchive: true,  // Compress old logs
          maxSize: '20m',       // Rotate when file reaches 20MB
          maxFiles: '14d',      // Keep logs for 14 days
          format: winston.format.combine(
            winston.format.timestamp(),
            winston.format.json(),
          ),
        }),

        // Separate file for errors
        new DailyRotateFile({
          filename: 'logs/error-%DATE%.log',
          datePattern: 'YYYY-MM-DD',
          zippedArchive: true,
          maxSize: '20m',
          maxFiles: '30d',      // Keep error logs longer
          level: 'error',       // Only errors
          format: winston.format.combine(
            winston.format.timestamp(),
            winston.format.json(),
          ),
        }),
      ],
    }),
  ],
})
export class LoggerModule {}

// Generated files:
// logs/app-2025-12-30.log          (current)
// logs/app-2025-12-29.log.gz       (compressed)
// logs/app-2025-12-28.log.gz
// logs/error-2025-12-30.log        (current)
// logs/error-2025-12-29.log.gz
```

---

### **Method 2: Size-Based Rotation**

```typescript
// logger/logger.config.ts
import * as winston from 'winston';
import * as DailyRotateFile from 'winston-daily-rotate-file';

export const loggerConfig = {
  transports: [
    // Rotate by size (20MB)
    new DailyRotateFile({
      filename: 'logs/app-%DATE%.log',
      datePattern: 'YYYY-MM-DD-HH',  // Hourly rotation
      maxSize: '20m',                 // Rotate at 20MB
      maxFiles: '7d',                 // Keep 7 days
      zippedArchive: true,
    }),

    // Rotate by size and time
    new DailyRotateFile({
      filename: 'logs/combined-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      maxSize: '50m',      // Rotate at 50MB OR daily (whichever comes first)
      maxFiles: 10,        // Keep 10 files (not time-based)
      zippedArchive: true,
    }),
  ],
};

// Example: If logs reach 50MB before midnight, rotate immediately
// logs/combined-2025-12-30-01.log  (reached 50MB at 1am)
// logs/combined-2025-12-30-02.log  (reached 50MB at 2am)
// logs/combined-2025-12-30.log     (daily rotation at midnight)
```

---

### **Method 3: Multiple Log Levels (Separate Rotation)**

```typescript
// logger/logger.module.ts
import * as DailyRotateFile from 'winston-daily-rotate-file';

@Module({
  imports: [
    WinstonModule.forRoot({
      transports: [
        // INFO logs (shorter retention)
        new DailyRotateFile({
          filename: 'logs/info-%DATE%.log',
          level: 'info',
          datePattern: 'YYYY-MM-DD',
          maxFiles: '7d',        // Keep 7 days
          maxSize: '20m',
          zippedArchive: true,
        }),

        // WARN logs (medium retention)
        new DailyRotateFile({
          filename: 'logs/warn-%DATE%.log',
          level: 'warn',
          datePattern: 'YYYY-MM-DD',
          maxFiles: '14d',       // Keep 14 days
          maxSize: '10m',
          zippedArchive: true,
        }),

        // ERROR logs (long retention)
        new DailyRotateFile({
          filename: 'logs/error-%DATE%.log',
          level: 'error',
          datePattern: 'YYYY-MM-DD',
          maxFiles: '30d',       // Keep 30 days
          maxSize: '20m',
          zippedArchive: true,
        }),

        // DEBUG logs (very short retention, development only)
        new DailyRotateFile({
          filename: 'logs/debug-%DATE%.log',
          level: 'debug',
          datePattern: 'YYYY-MM-DD-HH',  // Hourly
          maxFiles: '24h',       // Keep 24 hours only
          maxSize: '50m',
          zippedArchive: true,
        }),
      ],
    }),
  ],
})
export class LoggerModule {}
```

---

### **Method 4: Rotation Events**

```typescript
// logger/logger.service.ts
import { Injectable } from '@nestjs/common';
import * as winston from 'winston';
import * as DailyRotateFile from 'winston-daily-rotate-file';

@Injectable()
export class LoggerService {
  private logger: winston.Logger;

  constructor() {
    const transport = new DailyRotateFile({
      filename: 'logs/app-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      maxFiles: '14d',
      maxSize: '20m',
      zippedArchive: true,
    });

    // Listen to rotation events
    transport.on('rotate', (oldFilename, newFilename) => {
      console.log(`Log rotated: ${oldFilename} -> ${newFilename}`);
      // Optional: Upload old file to S3, send notification, etc.
    });

    transport.on('archive', (zipFilename) => {
      console.log(`Log archived: ${zipFilename}`);
      // Optional: Upload to long-term storage
    });

    transport.on('logRemoved', (removedFilename) => {
      console.log(`Log removed: ${removedFilename}`);
      // Triggered when maxFiles policy deletes old logs
    });

    this.logger = winston.createLogger({
      transports: [transport],
    });
  }
}
```

---

### **Method 5: Custom Rotation with logrotate (Linux)**

```bash
# /etc/logrotate.d/nestjs-app
/var/log/nestjs-app/*.log {
    daily                    # Rotate daily
    rotate 14                # Keep 14 days
    missingok                # Don't error if log file missing
    notifempty               # Don't rotate empty files
    compress                 # Compress old logs
    delaycompress            # Compress after 1 day (so current log not compressed)
    create 0640 nodejs nodejs  # Create new log with permissions
    sharedscripts            # Run scripts once for all logs
    postrotate
        # Reload app to use new log file
        systemctl reload nestjs-app
    endscript
}

# Test configuration
sudo logrotate -d /etc/logrotate.d/nestjs-app

# Force rotation (for testing)
sudo logrotate -f /etc/logrotate.d/nestjs-app
```

---

### **Method 6: PM2 Log Rotation**

```bash
# Install PM2 log rotate module
pm2 install pm2-logrotate

# Configure
pm2 set pm2-logrotate:max_size 10M        # Rotate at 10MB
pm2 set pm2-logrotate:retain 30           # Keep 30 files
pm2 set pm2-logrotate:compress true       # Compress old logs
pm2 set pm2-logrotate:dateFormat YYYY-MM-DD  # Date format
pm2 set pm2-logrotate:workerInterval 30   # Check every 30 seconds
pm2 set pm2-logrotate:rotateInterval '0 0 * * *'  # Daily at midnight

# View configuration
pm2 conf pm2-logrotate

# ecosystem.config.js
module.exports = {
  apps: [{
    name: 'nestjs-app',
    script: 'dist/main.js',
    error_file: 'logs/error.log',
    out_file: 'logs/out.log',
    log_date_format: 'YYYY-MM-DD HH:mm:ss Z',
  }],
};

# PM2 will automatically rotate logs based on pm2-logrotate config
```

---

### **Method 7: Docker Log Rotation**

```yaml
# docker-compose.yml
version: '3.8'
services:
  nestjs-app:
    image: nestjs-app:latest
    logging:
      driver: "json-file"
      options:
        max-size: "10m"      # Rotate at 10MB
        max-file: "3"        # Keep 3 files
        compress: "true"     # Compress old logs

# Docker daemon config (/etc/docker/daemon.json)
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "5"
  }
}

# Restart Docker daemon
sudo systemctl restart docker

# View logs
docker logs nestjs-app

# Log files location
# /var/lib/docker/containers/<container-id>/<container-id>-json.log
```

---

### **Method 8: Kubernetes Log Rotation**

```yaml
# Kubernetes automatically rotates logs
# Default: 10MB per file, 5 files max

# Custom rotation via kubelet config
# /var/lib/kubelet/config.yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
containerLogMaxSize: 10Mi      # Max log file size
containerLogMaxFiles: 5        # Max number of log files

# Pod definition
apiVersion: v1
kind: Pod
metadata:
  name: nestjs-app
spec:
  containers:
  - name: app
    image: nestjs-app:latest
    # Logs written to stdout/stderr are automatically rotated by kubelet

# View logs
kubectl logs nestjs-app
kubectl logs nestjs-app --previous  # View previous container logs

# For long-term storage, use logging solution:
# - Fluentd/Fluent Bit (forward to Elasticsearch)
# - Promtail (forward to Loki)
# - CloudWatch Logs (AWS)
# - Cloud Logging (GCP)
```

---

### **Method 9: Archive to S3 (Long-term Storage)**

```typescript
// logger/s3-archiver.ts
import { Injectable } from '@nestjs/common';
import { S3 } from 'aws-sdk';
import * as fs from 'fs';
import * as path from 'path';

@Injectable()
export class S3LogArchiver {
  private s3: S3;

  constructor() {
    this.s3 = new S3({
      region: process.env.AWS_REGION,
      credentials: {
        accessKeyId: process.env.AWS_ACCESS_KEY_ID,
        secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
      },
    });
  }

  async archiveLog(logFilePath: string) {
    const fileName = path.basename(logFilePath);
    const fileContent = fs.readFileSync(logFilePath);

    await this.s3.putObject({
      Bucket: 'my-app-logs',
      Key: `logs/${new Date().getFullYear()}/${fileName}`,
      Body: fileContent,
      StorageClass: 'GLACIER',  // Cheaper long-term storage
    }).promise();

    console.log(`Archived ${fileName} to S3`);

    // Delete local file after successful upload
    fs.unlinkSync(logFilePath);
  }
}

// Schedule archiving with cron
import { Injectable } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';

@Injectable()
export class LogArchiveScheduler {
  constructor(private s3Archiver: S3LogArchiver) {}

  @Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT)
  async archiveOldLogs() {
    const logsDir = 'logs';
    const files = fs.readdirSync(logsDir);

    for (const file of files) {
      if (file.endsWith('.gz')) {  // Only archive compressed files
        const filePath = path.join(logsDir, file);
        const stats = fs.statSync(filePath);

        // Archive files older than 7 days
        if (Date.now() - stats.mtime.getTime() > 7 * 24 * 60 * 60 * 1000) {
          await this.s3Archiver.archiveLog(filePath);
        }
      }
    }
  }
}
```

---

### **Method 10: Complete Production Setup**

```typescript
// logger/logger.config.ts
import * as winston from 'winston';
import * as DailyRotateFile from 'winston-daily-rotate-file';
import * as path from 'path';

export const createLogger = () => {
  const logDir = process.env.LOG_DIR || 'logs';

  // Ensure log directory exists
  if (!fs.existsSync(logDir)) {
    fs.mkdirSync(logDir, { recursive: true });
  }

  return winston.createLogger({
    level: process.env.LOG_LEVEL || 'info',
    format: winston.format.combine(
      winston.format.timestamp({ format: 'YYYY-MM-DD HH:mm:ss' }),
      winston.format.errors({ stack: true }),
      winston.format.json(),
    ),
    defaultMeta: {
      service: process.env.SERVICE_NAME || 'nestjs-app',
      environment: process.env.NODE_ENV,
      hostname: require('os').hostname(),
    },
    transports: [
      // Console (no rotation)
      new winston.transports.Console({
        format: winston.format.combine(
          winston.format.colorize(),
          winston.format.simple(),
        ),
      }),

      // Combined logs (info and above)
      new DailyRotateFile({
        filename: path.join(logDir, 'combined-%DATE%.log'),
        datePattern: 'YYYY-MM-DD',
        maxSize: '20m',
        maxFiles: '14d',
        zippedArchive: true,
        level: 'info',
      }),

      // Error logs (longer retention)
      new DailyRotateFile({
        filename: path.join(logDir, 'error-%DATE%.log'),
        datePattern: 'YYYY-MM-DD',
        maxSize: '20m',
        maxFiles: '30d',
        zippedArchive: true,
        level: 'error',
      }),

      // HTTP access logs
      new DailyRotateFile({
        filename: path.join(logDir, 'access-%DATE%.log'),
        datePattern: 'YYYY-MM-DD',
        maxSize: '50m',
        maxFiles: '7d',
        zippedArchive: true,
      }),
    ],
    exceptionHandlers: [
      new DailyRotateFile({
        filename: path.join(logDir, 'exceptions-%DATE%.log'),
        datePattern: 'YYYY-MM-DD',
        maxSize: '20m',
        maxFiles: '30d',
        zippedArchive: true,
      }),
    ],
    rejectionHandlers: [
      new DailyRotateFile({
        filename: path.join(logDir, 'rejections-%DATE%.log'),
        datePattern: 'YYYY-MM-DD',
        maxSize: '20m',
        maxFiles: '30d',
        zippedArchive: true,
      }),
    ],
  });
};
```

---

### **Rotation Strategies Comparison:**

```typescript
// Strategy 1: Time-Based (Daily)
datePattern: 'YYYY-MM-DD'
maxFiles: '14d'
// Pros: Predictable, easy to manage
// Cons: Files can grow large if high traffic

// Strategy 2: Size-Based (20MB)
maxSize: '20m'
maxFiles: 10
// Pros: Consistent file sizes, prevents huge files
// Cons: Unpredictable rotation times

// Strategy 3: Hybrid (Size AND Time)
maxSize: '20m'
datePattern: 'YYYY-MM-DD'
maxFiles: '14d'
// Pros: Best of both worlds
// Cons: Slightly more complex

// Recommendation: Hybrid (size + time)
```

---

### **Best Practices:**

```typescript
// ✅ GOOD - Use daily rotation
datePattern: 'YYYY-MM-DD'
// Easy to manage, predictable

// ✅ GOOD - Set max file size
maxSize: '20m'
// Prevents huge files that are hard to process

// ✅ GOOD - Compress old logs
zippedArchive: true
// Saves 80-90% disk space

// ✅ GOOD - Different retention for different levels
info: '7d', warn: '14d', error: '30d'
// Keep errors longer for investigation

// ✅ GOOD - Archive to S3/Glacier
// Move old logs to cheap long-term storage

// ✅ GOOD - Monitor disk space
if (diskUsage > 80%) {
  alert('Low disk space!');
}

// ❌ BAD - No rotation
filename: 'app.log'  // Will grow forever!

// ❌ BAD - No size limit
maxSize: undefined  // File can reach gigabytes

// ❌ BAD - Keeping logs forever
maxFiles: undefined  // Disk will fill up

// ❌ BAD - Not compressing
zippedArchive: false  // Wastes disk space

// ❌ BAD - Same retention for all levels
// Error logs should be kept longer than info logs
```

**Key Takeaway:** Handle log rotation using **winston-daily-rotate-file** with **daily rotation** (`datePattern: 'YYYY-MM-DD'`), **size limits** (`maxSize: '20m'`), **retention policies** (`maxFiles: '14d'`), **compression** (`zippedArchive: true`), different retention for different levels (errors longer), and **archive old logs** to S3/Glacier for long-term storage.

</details>

<details>
<summary><strong>42. What log retention policies should you follow?</strong></summary>

**Answer:**

Log retention policies define **how long to keep logs** before deletion. Follow **compliance requirements** (GDPR, SOC2, HIPAA), **business needs** (debugging, auditing), and **storage costs**. Typical retention: **7-30 days** for info, **30-90 days** for errors, **1-7 years** for audit logs.

---

### **Retention Guidelines by Log Type:**

```typescript
// General Application Logs
Info/Debug:     7-14 days     // Recent debugging
Warnings:       14-30 days    // Investigate patterns
Errors:         30-90 days    // Root cause analysis
Critical:       90-180 days   // Incident review

// Access/HTTP Logs
Access logs:    7-30 days     // Traffic analysis

// Security Logs
Authentication: 90-365 days   // Security audits
Authorization:  90-365 days   // Access control review
Failed logins:  90-365 days   // Brute force detection

// Audit Logs (Compliance)
Financial:      7 years       // SOX, IRS requirements
Healthcare:     6 years       // HIPAA
Payment:        1 year        // PCI-DSS
GDPR:           As needed     // Can be deleted on request

// Performance Metrics
Metrics:        30-90 days    // Performance analysis
Traces:         7-30 days     // Distributed tracing
```

---

### **Method 1: Retention by Log Level**

```typescript
// logger/logger.config.ts
import * as DailyRotateFile from 'winston-daily-rotate-file';

export const loggerConfig = {
  transports: [
    // Debug logs - shortest retention
    new DailyRotateFile({
      filename: 'logs/debug-%DATE%.log',
      level: 'debug',
      datePattern: 'YYYY-MM-DD',
      maxFiles: '7d',        // 7 days
      maxSize: '20m',
      zippedArchive: true,
    }),

    // Info logs - short retention
    new DailyRotateFile({
      filename: 'logs/info-%DATE%.log',
      level: 'info',
      datePattern: 'YYYY-MM-DD',
      maxFiles: '14d',       // 14 days
      maxSize: '20m',
      zippedArchive: true,
    }),

    // Warning logs - medium retention
    new DailyRotateFile({
      filename: 'logs/warn-%DATE%.log',
      level: 'warn',
      datePattern: 'YYYY-MM-DD',
      maxFiles: '30d',       // 30 days
      maxSize: '20m',
      zippedArchive: true,
    }),

    // Error logs - long retention
    new DailyRotateFile({
      filename: 'logs/error-%DATE%.log',
      level: 'error',
      datePattern: 'YYYY-MM-DD',
      maxFiles: '90d',       // 90 days
      maxSize: '20m',
      zippedArchive: true,
    }),
  ],
};
```

---

### **Method 2: Compliance-Based Retention**

```typescript
// logger/retention-policies.ts
export const RETENTION_POLICIES = {
  // GDPR (EU) - Right to be forgotten
  gdpr: {
    general: '30d',          // General logs
    audit: 'user-request',   // Delete on user request
    maxRetention: '2y',      // Maximum 2 years
  },

  // PCI-DSS (Payment Card Industry)
  pciDss: {
    audit: '1y',             // 1 year minimum
    access: '90d',           // 90 days
  },

  // HIPAA (Healthcare)
  hipaa: {
    audit: '6y',             // 6 years
    access: '6y',
  },

  // SOX (Sarbanes-Oxley)
  sox: {
    financial: '7y',         // 7 years
    audit: '7y',
  },

  // SOC 2 (Security)
  soc2: {
    security: '1y',          // 1 year
    access: '90d',
    incidents: '2y',         // 2 years
  },
};

// logger/logger.config.ts
import { RETENTION_POLICIES } from './retention-policies';

const isCompliant = (logType: string) => {
  const compliance = process.env.COMPLIANCE;  // 'pci-dss', 'hipaa', etc.

  if (compliance === 'pci-dss') {
    return RETENTION_POLICIES.pciDss[logType] || '30d';
  }

  if (compliance === 'hipaa') {
    return RETENTION_POLICIES.hipaa[logType] || '6y';
  }

  // Default
  return '30d';
};

export const loggerConfig = {
  transports: [
    // Audit logs (compliance-based retention)
    new DailyRotateFile({
      filename: 'logs/audit-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      maxFiles: isCompliant('audit'),  // Varies by compliance
      maxSize: '20m',
      zippedArchive: true,
    }),
  ],
};
```

---

### **Method 3: Environment-Based Retention**

```typescript
// logger/logger.config.ts
const getRetentionPolicy = () => {
  const env = process.env.NODE_ENV;

  if (env === 'development') {
    return {
      debug: '3d',     // 3 days
      info: '7d',      // 7 days
      error: '14d',    // 14 days
    };
  }

  if (env === 'staging') {
    return {
      debug: '7d',     // 7 days
      info: '14d',     // 14 days
      error: '30d',    // 30 days
    };
  }

  if (env === 'production') {
    return {
      debug: '7d',     // 7 days (shouldn't have many debug logs)
      info: '30d',     // 30 days
      error: '90d',    // 90 days
      audit: '1y',     // 1 year
    };
  }

  return { info: '7d', error: '30d' };
};

const retention = getRetentionPolicy();

export const loggerConfig = {
  transports: [
    new DailyRotateFile({
      filename: 'logs/info-%DATE%.log',
      maxFiles: retention.info,
    }),
    new DailyRotateFile({
      filename: 'logs/error-%DATE%.log',
      maxFiles: retention.error,
    }),
  ],
};
```

---

### **Method 4: Cost-Optimized Retention (S3 Lifecycle)**

```typescript
// Archive logs to S3 with lifecycle policies

// S3 Lifecycle Policy (AWS Console or CLI)
{
  "Rules": [
    {
      "Id": "Archive-logs-to-glacier",
      "Status": "Enabled",
      "Filter": {
        "Prefix": "logs/"
      },
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "STANDARD_IA"  // Infrequent Access (cheaper)
        },
        {
          "Days": 90,
          "StorageClass": "GLACIER"      // Very cheap long-term storage
        },
        {
          "Days": 365,
          "StorageClass": "DEEP_ARCHIVE"  // Cheapest (retrieval takes hours)
        }
      ],
      "Expiration": {
        "Days": 2555  // 7 years (for compliance)
      }
    }
  ]
}

// Cost comparison:
// Standard:      $0.023/GB/month
// Standard-IA:   $0.0125/GB/month (after 30 days)
// Glacier:       $0.004/GB/month (after 90 days)
// Deep Archive:  $0.00099/GB/month (after 1 year)

// Example: 1TB of logs for 7 years
// Standard:      $1,932/year * 7 = $13,524
// With lifecycle: ~$500 total (saves $13,000!)
```

---

### **Method 5: Tiered Storage Strategy**

```typescript
// logger/tiered-storage.ts
import { Injectable } from '@nestjs/common';
import { Cron } from '@nestjs/schedule';
import { S3 } from 'aws-sdk';
import * as fs from 'fs';
import * as path from 'path';

@Injectable()
export class TieredStorageService {
  private s3: S3;

  constructor() {
    this.s3 = new S3();
  }

  @Cron('0 0 * * *')  // Daily at midnight
  async manageLogs() {
    const logsDir = 'logs';
    const files = fs.readdirSync(logsDir);

    for (const file of files) {
      const filePath = path.join(logsDir, file);
      const stats = fs.statSync(filePath);
      const ageInDays = (Date.now() - stats.mtime.getTime()) / (1000 * 60 * 60 * 24);

      // Tier 1: Local disk (0-7 days) - Fast access
      if (ageInDays <= 7) {
        continue;  // Keep on local disk
      }

      // Tier 2: S3 Standard (7-30 days) - Quick retrieval
      if (ageInDays <= 30) {
        await this.uploadToS3(filePath, 'STANDARD');
        fs.unlinkSync(filePath);  // Delete local copy
      }

      // Tier 3: S3 Glacier (30-365 days) - Cheap storage
      if (ageInDays <= 365) {
        await this.uploadToS3(filePath, 'GLACIER');
      }

      // Tier 4: Delete (>365 days)
      if (ageInDays > 365) {
        await this.deleteFromS3(file);
      }
    }
  }

  private async uploadToS3(filePath: string, storageClass: string) {
    const fileContent = fs.readFileSync(filePath);
    await this.s3.putObject({
      Bucket: 'my-app-logs',
      Key: `logs/${path.basename(filePath)}`,
      Body: fileContent,
      StorageClass: storageClass,
    }).promise();
  }

  private async deleteFromS3(filename: string) {
    await this.s3.deleteObject({
      Bucket: 'my-app-logs',
      Key: `logs/${filename}`,
    }).promise();
  }
}
```

---

### **Method 6: User Data Deletion (GDPR)**

```typescript
// logger/gdpr.service.ts
import { Injectable } from '@nestjs/common';
import * as fs from 'fs';
import * as readline from 'readline';

@Injectable()
export class GDPRService {
  // Delete user data from logs (GDPR right to be forgotten)
  async deleteUserData(userId: string) {
    const logsDir = 'logs';
    const files = fs.readdirSync(logsDir);

    for (const file of files) {
      if (file.endsWith('.log')) {
        await this.removeUserFromLog(
          path.join(logsDir, file),
          userId,
        );
      }
    }

    console.log(`Deleted data for user ${userId}`);
  }

  private async removeUserFromLog(filePath: string, userId: string) {
    const tempPath = `${filePath}.temp`;
    const fileStream = fs.createReadStream(filePath);
    const rl = readline.createInterface({ input: fileStream });
    const writeStream = fs.createWriteStream(tempPath);

    for await (const line of rl) {
      try {
        const log = JSON.parse(line);

        // Skip lines containing user data
        if (log.userId === userId || log.user?.id === userId) {
          continue;  // Don't write this line
        }

        writeStream.write(line + '\n');
      } catch (error) {
        // Not JSON, write as-is
        writeStream.write(line + '\n');
      }
    }

    writeStream.end();
    fs.renameSync(tempPath, filePath);  // Replace original
  }
}

// Usage
const gdprService = app.get(GDPRService);
await gdprService.deleteUserData('user-123');
```

---

### **Method 7: Retention Policy Documentation**

```typescript
// RETENTION_POLICY.md

# Log Retention Policy

## Overview
This document defines log retention periods for all log types.

## Retention Periods

### Application Logs
| Log Type | Retention | Reason |
|----------|-----------|--------|
| Debug    | 7 days    | Development debugging |
| Info     | 30 days   | Operational analysis |
| Warning  | 30 days   | Pattern investigation |
| Error    | 90 days   | Root cause analysis |

### Security Logs
| Log Type          | Retention | Reason |
|-------------------|-----------|--------|
| Authentication    | 1 year    | SOC 2 compliance |
| Failed logins     | 1 year    | Security audits |
| Authorization     | 1 year    | Access control |

### Audit Logs
| Log Type     | Retention | Reason |
|--------------|-----------|--------|
| Financial    | 7 years   | SOX compliance |
| User actions | 1 year    | Compliance |

### Access Logs
| Log Type  | Retention | Reason |
|-----------|-----------|--------|
| HTTP      | 30 days   | Traffic analysis |
| API calls | 30 days   | Usage patterns |

## Storage Tiers
- 0-7 days: Local disk (fast access)
- 7-30 days: S3 Standard (quick retrieval)
- 30-365 days: S3 Glacier (cheap storage)
- >365 days: Deleted (or Deep Archive for compliance)

## Compliance
- PCI-DSS: 1 year audit logs
- HIPAA: 6 years audit logs
- GDPR: User data deleted on request
- SOX: 7 years financial logs

## Review
This policy is reviewed annually and updated as needed.

Last updated: 2025-12-30
```

---

### **Method 8: Automated Retention Enforcement**

```typescript
// logger/retention-enforcer.ts
import { Injectable } from '@nestjs/common';
import { Cron } from '@nestjs/schedule';
import * as fs from 'fs';
import * as path from 'path';

interface RetentionPolicy {
  pattern: RegExp;
  maxAgeDays: number;
}

@Injectable()
export class RetentionEnforcer {
  private policies: RetentionPolicy[] = [
    { pattern: /debug-.*\.log/, maxAgeDays: 7 },
    { pattern: /info-.*\.log/, maxAgeDays: 30 },
    { pattern: /warn-.*\.log/, maxAgeDays: 30 },
    { pattern: /error-.*\.log/, maxAgeDays: 90 },
    { pattern: /audit-.*\.log/, maxAgeDays: 365 },
  ];

  @Cron('0 0 * * *')  // Daily
  async enforceRetention() {
    const logsDir = 'logs';
    const files = fs.readdirSync(logsDir);

    for (const file of files) {
      const filePath = path.join(logsDir, file);
      const policy = this.getPolicy(file);

      if (!policy) continue;

      const stats = fs.statSync(filePath);
      const ageInDays = (Date.now() - stats.mtime.getTime()) / (1000 * 60 * 60 * 24);

      if (ageInDays > policy.maxAgeDays) {
        fs.unlinkSync(filePath);
        console.log(`Deleted ${file} (age: ${ageInDays.toFixed(1)} days)`);
      }
    }
  }

  private getPolicy(filename: string): RetentionPolicy | undefined {
    return this.policies.find(policy => policy.pattern.test(filename));
  }
}
```

---

### **Method 9: Retention Monitoring**

```typescript
// logger/retention-monitor.ts
import { Injectable } from '@nestjs/common';
import { Cron } from '@nestjs/schedule';
import * as fs from 'fs';
import * as path from 'path';

@Injectable()
export class RetentionMonitor {
  @Cron('0 0 * * 0')  // Weekly
  async reportRetention() {
    const logsDir = 'logs';
    const files = fs.readdirSync(logsDir);

    let totalSize = 0;
    const fileStats: any[] = [];

    for (const file of files) {
      const filePath = path.join(logsDir, file);
      const stats = fs.statSync(filePath);
      const ageInDays = (Date.now() - stats.mtime.getTime()) / (1000 * 60 * 60 * 24);

      totalSize += stats.size;
      fileStats.push({
        name: file,
        size: (stats.size / 1024 / 1024).toFixed(2) + ' MB',
        age: ageInDays.toFixed(1) + ' days',
      });
    }

    console.log('Log Retention Report');
    console.log('====================');
    console.log(`Total files: ${files.length}`);
    console.log(`Total size: ${(totalSize / 1024 / 1024).toFixed(2)} MB`);
    console.log('\nFiles:');
    console.table(fileStats);

    // Alert if disk usage high
    if (totalSize > 10 * 1024 * 1024 * 1024) {  // 10GB
      console.warn('WARNING: Log disk usage exceeds 10GB!');
    }
  }
}
```

---

### **Method 10: Complete Retention Policy Setup**

```typescript
// logger/logger.module.ts
import { Module } from '@nestjs/common';
import { ScheduleModule } from '@nestjs/schedule';
import { WinstonModule } from 'nest-winston';
import { RetentionEnforcer } from './retention-enforcer';
import { RetentionMonitor } from './retention-monitor';
import { TieredStorageService } from './tiered-storage';
import { GDPRService } from './gdpr.service';

@Module({
  imports: [
    ScheduleModule.forRoot(),
    WinstonModule.forRoot({
      // Logger config with retention
    }),
  ],
  providers: [
    RetentionEnforcer,   // Automatically delete old logs
    RetentionMonitor,    // Report on retention status
    TieredStorageService,  // Move to S3/Glacier
    GDPRService,         // Handle user data deletion
  ],
  exports: [GDPRService],
})
export class LoggerModule {}
```

---

### **Best Practices:**

```typescript
// ✅ GOOD - Different retention for different levels
info: '30d', error: '90d', audit: '1y'
// Errors need longer retention for investigation

// ✅ GOOD - Follow compliance requirements
PCI-DSS: 1 year
HIPAA: 6 years
SOX: 7 years

// ✅ GOOD - Use tiered storage
0-7d: Local disk
7-30d: S3 Standard
30d+: Glacier
// Balances cost and access speed

// ✅ GOOD - Document retention policy
// Clear documentation for auditors and team

// ✅ GOOD - Automate retention enforcement
@Cron('0 0 * * *')
enforceRetention()
// Don't rely on manual cleanup

// ✅ GOOD - Monitor retention compliance
weekly report of log sizes and ages

// ❌ BAD - Same retention for all logs
maxFiles: '30d'  // Everything deleted after 30 days
// Audit logs may need longer retention!

// ❌ BAD - No retention policy
maxFiles: undefined  // Disk fills up

// ❌ BAD - Not considering compliance
// Delete logs after 7 days
// But PCI-DSS requires 1 year!

// ❌ BAD - No cost optimization
// Keep everything on expensive SSD storage
// Use S3 Glacier for old logs

// ❌ BAD - No GDPR compliance
// Can't delete user data from logs
// Must be able to remove on request
```

**Key Takeaway:** Follow retention policies based on **log level** (info: 7-30d, errors: 30-90d), **compliance requirements** (PCI-DSS: 1y, HIPAA: 6y, SOX: 7y), **environment** (production longer than dev), **tiered storage** (local → S3 → Glacier → delete), **automate enforcement** with cron jobs, and **document policies** for audits. Use **GDPR-compliant** deletion for user data.

</details>

<details>
<summary><strong>43. Should you log in production vs development differently?</strong></summary>

**Answer:**

**YES, absolutely**. Production logging should be **structured**, **sanitized**, **minimal**, and **performance-optimized** with centralized collection. Development logging should be **verbose**, **human-readable**, **detailed**, and include debug information for rapid iteration.

---

### **Key Differences:**

```typescript
// Development
- Log Level: DEBUG
- Format: Human-readable (colorized console)
- Destination: Console, local files
- Sensitive Data: Sometimes logged for debugging
- Performance: Not critical
- Sampling: 100% (all requests)
- Stack Traces: Full stack traces
- Retention: Short (3-7 days)

// Production
- Log Level: INFO (or WARN)
- Format: JSON (structured)
- Destination: Centralized logging (ELK, CloudWatch)
- Sensitive Data: NEVER logged (sanitized)
- Performance: Critical (async, buffered)
- Sampling: 10-20% for high traffic
- Stack Traces: Error boundary only
- Retention: Long (30-90 days)
```

---

### **Method 1: Environment-Based Configuration**

```typescript
// logger/logger.config.ts
import * as winston from 'winston';
import * as DailyRotateFile from 'winston-daily-rotate-file';

export const getLoggerConfig = () => {
  const isDevelopment = process.env.NODE_ENV === 'development';
  const isProduction = process.env.NODE_ENV === 'production';

  // Development configuration
  if (isDevelopment) {
    return {
      level: 'debug',  // Verbose
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.timestamp({ format: 'HH:mm:ss' }),
        winston.format.printf(({ timestamp, level, message, ...meta }) => {
          return `${timestamp} [${level}] ${message} ${
            Object.keys(meta).length ? JSON.stringify(meta, null, 2) : ''
          }`;
        }),
      ),
      transports: [
        new winston.transports.Console(),  // Console only
        // Optional: Simple file for debugging
        new winston.transports.File({
          filename: 'debug.log',
          maxsize: 10485760,  // 10MB
        }),
      ],
    };
  }

  // Production configuration
  if (isProduction) {
    return {
      level: 'info',  // Less verbose
      format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.errors({ stack: true }),
        winston.format.json(),  // Structured
      ),
      transports: [
        // Console (captured by Docker/K8s)
        new winston.transports.Console({
          format: winston.format.json(),
        }),
        // Rotate files with compression
        new DailyRotateFile({
          filename: 'logs/app-%DATE%.log',
          datePattern: 'YYYY-MM-DD',
          maxFiles: '30d',
          maxSize: '20m',
          zippedArchive: true,
        }),
        new DailyRotateFile({
          filename: 'logs/error-%DATE%.log',
          datePattern: 'YYYY-MM-DD',
          level: 'error',
          maxFiles: '90d',
          maxSize: '20m',
          zippedArchive: true,
        }),
      ],
      exceptionHandlers: [
        new DailyRotateFile({
          filename: 'logs/exceptions-%DATE%.log',
          datePattern: 'YYYY-MM-DD',
          maxFiles: '90d',
        }),
      ],
    };
  }

  // Staging (hybrid)
  return {
    level: 'debug',
    format: winston.format.combine(
      winston.format.timestamp(),
      winston.format.json(),
    ),
    transports: [
      new winston.transports.Console(),
      new DailyRotateFile({
        filename: 'logs/staging-%DATE%.log',
        datePattern: 'YYYY-MM-DD',
        maxFiles: '14d',
      }),
    ],
  };
};

// logger/logger.module.ts
@Module({
  imports: [
    WinstonModule.forRoot(getLoggerConfig()),
  ],
})
export class LoggerModule {}
```

---

### **Method 2: Development Logger (Verbose & Colorized)**

```typescript
// logger/development-logger.ts
import * as winston from 'winston';

export const developmentLogger = winston.createLogger({
  level: 'debug',
  format: winston.format.combine(
    winston.format.colorize(),
    winston.format.timestamp({ format: 'HH:mm:ss.SSS' }),
    winston.format.printf(({ timestamp, level, message, context, ...meta }) => {
      let log = `${timestamp} ${level}`;
      
      if (context) {
        log += ` [${context}]`;
      }
      
      log += ` ${message}`;

      // Pretty print metadata
      if (Object.keys(meta).length) {
        log += `\n${JSON.stringify(meta, null, 2)}`;
      }

      return log;
    }),
  ),
  transports: [
    new winston.transports.Console(),
  ],
});

// Example output:
// 10:30:45.123 info [UsersService] User created
// {
//   "userId": 123,
//   "email": "user@example.com",
//   "duration": "45ms"
// }

// 10:30:46.789 debug [OrdersController] Processing order
// {
//   "orderId": 456,
//   "items": [
//     { "id": 1, "quantity": 2 },
//     { "id": 2, "quantity": 1 }
//   ]
// }
```

---

### **Method 3: Production Logger (Structured & Sanitized)**

```typescript
// logger/production-logger.ts
import * as winston from 'winston';
import { LogSanitizer } from './sanitizer';

export const productionLogger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    // Sanitize sensitive data
    winston.format((info) => LogSanitizer.sanitize(info))(),
    winston.format.json(),
  ),
  defaultMeta: {
    service: process.env.SERVICE_NAME,
    environment: 'production',
    version: process.env.APP_VERSION,
    hostname: require('os').hostname(),
  },
  transports: [
    new winston.transports.Console({
      format: winston.format.json(),
    }),
    new DailyRotateFile({
      filename: 'logs/app-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      maxFiles: '30d',
      maxSize: '20m',
      zippedArchive: true,
    }),
  ],
});

// Example output:
{
  "timestamp": "2025-12-30T10:30:45.123Z",
  "level": "info",
  "message": "User created",
  "service": "nestjs-api",
  "environment": "production",
  "version": "1.2.3",
  "hostname": "pod-abc-123",
  "userId": 123,
  "duration": 45,
  "traceId": "abc123def456"
}
```

---

### **Method 4: Conditional Logging**

```typescript
// logger/conditional-logger.service.ts
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class ConditionalLogger {
  private isDevelopment: boolean;
  private isProduction: boolean;

  constructor(private configService: ConfigService) {
    const env = this.configService.get('NODE_ENV');
    this.isDevelopment = env === 'development';
    this.isProduction = env === 'production';
  }

  // Only log in development
  debug(message: string, context?: any) {
    if (this.isDevelopment) {
      console.log(`[DEBUG] ${message}`, context);
    }
  }

  // Log in all environments
  info(message: string, context?: any) {
    console.log(`[INFO] ${message}`, context);
  }

  // Log in all environments
  error(message: string, error?: Error, context?: any) {
    console.error(`[ERROR] ${message}`, { error, ...context });
  }

  // Only log performance in development
  logPerformance(operation: string, duration: number) {
    if (this.isDevelopment && duration > 100) {
      console.warn(`[PERF] ${operation} took ${duration}ms`);
    } else if (this.isProduction && duration > 1000) {
      // Only log slow operations in production
      console.warn(`[PERF] Slow operation: ${operation} took ${duration}ms`);
    }
  }

  // Log request/response in development, minimal in production
  logRequest(method: string, url: string, body?: any) {
    if (this.isDevelopment) {
      console.log(`[REQUEST] ${method} ${url}`, { body });
    } else {
      // Production: only log method and URL
      console.log(`[REQUEST] ${method} ${url}`);
    }
  }
}
```

---

### **Method 5: Sampling (Production Only)**

```typescript
// logger/sampling-logger.ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class SamplingLogger {
  private readonly sampleRate: number;

  constructor() {
    // Sample 10% in production, 100% in development
    this.sampleRate = process.env.NODE_ENV === 'production' ? 0.1 : 1.0;
  }

  log(message: string, context?: any) {
    if (this.shouldLog()) {
      console.log(message, context);
    }
  }

  // Always log errors
  error(message: string, error?: Error) {
    console.error(message, error);
  }

  // Always log critical
  critical(message: string, context?: any) {
    console.error('[CRITICAL]', message, context);
  }

  private shouldLog(): boolean {
    return Math.random() < this.sampleRate;
  }
}

// Production: Only 10% of info/debug logs recorded
// Development: All logs recorded
```

---

### **Method 6: Feature Flags for Debug Logging**

```typescript
// logger/feature-flag-logger.ts
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class FeatureFlagLogger {
  constructor(private configService: ConfigService) {}

  log(message: string, context?: any) {
    console.log(message, context);
  }

  // Debug logging controlled by feature flag
  debug(message: string, context?: any) {
    const debugEnabled = this.configService.get('DEBUG_LOGGING_ENABLED');
    
    if (debugEnabled === 'true') {
      console.debug('[DEBUG]', message, context);
    }
  }

  // Conditional stack traces
  error(message: string, error?: Error) {
    const fullStackTraces = this.configService.get('FULL_STACK_TRACES');
    
    if (fullStackTraces === 'true') {
      console.error(message, error);
    } else {
      // Production: Only log error message and first line of stack
      console.error(message, {
        name: error?.name,
        message: error?.message,
        stack: error?.stack?.split('\n')[0],
      });
    }
  }
}

// .env.development
DEBUG_LOGGING_ENABLED=true
FULL_STACK_TRACES=true

// .env.production
DEBUG_LOGGING_ENABLED=false
FULL_STACK_TRACES=false
```

---

### **Method 7: Performance Optimization (Production)**

```typescript
// logger/optimized-logger.ts
import { Injectable } from '@nestjs/common';
import { EventEmitter } from 'events';

@Injectable()
export class OptimizedLogger {
  private logBuffer: any[] = [];
  private readonly bufferSize = 100;
  private readonly flushInterval = 5000;  // 5 seconds
  private eventEmitter: EventEmitter;

  constructor() {
    this.eventEmitter = new EventEmitter();
    
    // Only buffer in production
    if (process.env.NODE_ENV === 'production') {
      this.startBuffering();
    }
  }

  log(message: string, context?: any) {
    const logEntry = {
      timestamp: new Date().toISOString(),
      level: 'info',
      message,
      ...context,
    };

    if (process.env.NODE_ENV === 'production') {
      // Buffer logs in production
      this.logBuffer.push(logEntry);
      
      if (this.logBuffer.length >= this.bufferSize) {
        this.flush();
      }
    } else {
      // Immediate logging in development
      console.log(message, context);
    }
  }

  private startBuffering() {
    setInterval(() => this.flush(), this.flushInterval);
  }

  private flush() {
    if (this.logBuffer.length === 0) return;

    // Batch write to stdout (captured by log collector)
    console.log(JSON.stringify(this.logBuffer));
    this.logBuffer = [];
  }
}
```

---

### **Method 8: Complete Environment Configuration**

```typescript
// config/logger.config.ts
export const getLoggerConfig = () => {
  const env = process.env.NODE_ENV;

  const configs = {
    development: {
      level: 'debug',
      format: 'pretty',
      console: {
        enabled: true,
        colorize: true,
      },
      file: {
        enabled: true,
        path: 'debug.log',
        maxSize: '10m',
      },
      sanitization: false,  // Allow sensitive data for debugging
      sampling: 1.0,        // 100%
      performance: true,    // Log all performance metrics
      stackTraces: 'full',  // Full stack traces
      requestLogging: 'verbose',  // Log full request/response
      retention: '7d',
    },

    staging: {
      level: 'debug',
      format: 'json',
      console: {
        enabled: true,
        colorize: false,
      },
      file: {
        enabled: true,
        path: 'logs/staging.log',
        maxSize: '20m',
      },
      sanitization: true,   // Sanitize sensitive data
      sampling: 0.5,        // 50%
      performance: true,
      stackTraces: 'full',
      requestLogging: 'minimal',  // Only method and URL
      retention: '14d',
    },

    production: {
      level: 'info',
      format: 'json',
      console: {
        enabled: true,
        colorize: false,
      },
      file: {
        enabled: true,
        path: 'logs/app.log',
        maxSize: '20m',
        rotate: true,
      },
      sanitization: true,   // Always sanitize
      sampling: 0.1,        // 10%
      performance: false,   // Don't log every performance metric
      stackTraces: 'error-only',  // Only for errors
      requestLogging: 'minimal',  // Minimal request logging
      retention: '30d',
      compression: true,
      centralizedLogging: {
        enabled: true,
        destination: 'cloudwatch',  // or 'elasticsearch', 'datadog'
      },
    },
  };

  return configs[env] || configs.production;
};
```

---

### **Method 9: Testing Differences**

```typescript
// logger/logger.spec.ts
import { developmentLogger } from './development-logger';
import { productionLogger } from './production-logger';

describe('Logger', () => {
  describe('Development', () => {
    it('should log debug messages', () => {
      const spy = jest.spyOn(console, 'log');
      
      developmentLogger.debug('Test message');
      
      expect(spy).toHaveBeenCalled();
    });

    it('should include full context', () => {
      const spy = jest.spyOn(console, 'log');
      
      const context = { userId: 123, email: 'user@example.com', password: 'secret' };
      developmentLogger.log('User created', context);
      
      // Development: password may be logged (for debugging)
      expect(spy).toHaveBeenCalledWith(
        expect.stringContaining('password'),
        expect.anything(),
      );
    });
  });

  describe('Production', () => {
    it('should NOT log debug messages', () => {
      process.env.NODE_ENV = 'production';
      const spy = jest.spyOn(console, 'log');
      
      productionLogger.debug('Test message');
      
      expect(spy).not.toHaveBeenCalled();
    });

    it('should sanitize sensitive data', () => {
      const spy = jest.spyOn(console, 'log');
      
      const context = { userId: 123, email: 'user@example.com', password: 'secret' };
      productionLogger.log('User created', context);
      
      // Production: password should be redacted
      expect(spy).toHaveBeenCalledWith(
        expect.not.stringContaining('secret'),
        expect.anything(),
      );
    });

    it('should output JSON format', () => {
      const spy = jest.spyOn(console, 'log');
      
      productionLogger.log('Test message');
      
      const output = spy.mock.calls[0][0];
      expect(() => JSON.parse(output)).not.toThrow();
    });
  });
});
```

---

### **Method 10: Comparison Table**

```typescript
// Comprehensive comparison

| Feature                | Development | Staging | Production |
|------------------------|-------------|---------|------------|
| Log Level              | debug       | debug   | info       |
| Format                 | Pretty      | JSON    | JSON       |
| Colorized              | ✅ Yes      | ❌ No   | ❌ No      |
| Console Output         | ✅ Yes      | ✅ Yes  | ✅ Yes     |
| File Output            | Optional    | ✅ Yes  | ✅ Yes     |
| Rotation               | ❌ No       | ✅ Yes  | ✅ Yes     |
| Compression            | ❌ No       | ✅ Yes  | ✅ Yes     |
| Sanitization           | ❌ No       | ✅ Yes  | ✅ Yes     |
| Sampling               | 100%        | 50%     | 10%        |
| Stack Traces           | Full        | Full    | Error only |
| Performance Logging    | All         | All     | Critical   |
| Request Body Logging   | Full        | Minimal | Minimal    |
| Sensitive Data         | Sometimes   | ❌ Never| ❌ Never   |
| Retention              | 7 days      | 14 days | 30-90 days |
| Centralized Logging    | ❌ No       | Optional| ✅ Yes     |
| Alerting               | ❌ No       | Optional| ✅ Yes     |
| Cost Optimization      | ❌ No       | Optional| ✅ Yes     |
```

---

### **Best Practices:**

```typescript
// ✅ GOOD - Environment-specific configuration
const logger = getLoggerConfig(process.env.NODE_ENV);

// ✅ GOOD - Development: Verbose and readable
if (isDevelopment) {
  logger.debug('Processing order', { orderId, items });
}

// ✅ GOOD - Production: Structured and sanitized
if (isProduction) {
  logger.info('Order processed', { orderId });  // No sensitive data
}

// ✅ GOOD - Different log levels
development: 'debug'
staging: 'debug'
production: 'info'  // Less verbose

// ✅ GOOD - Sanitize in production only
if (isProduction) {
  context = LogSanitizer.sanitize(context);
}

// ✅ GOOD - Sample high-volume logs in production
if (isProduction && Math.random() < 0.1) {
  logger.log('Request processed');
}

// ✅ GOOD - Always log errors in all environments
logger.error('Payment failed', { orderId, error });

// ❌ BAD - Same configuration for all environments
const logger = winston.createLogger({ level: 'debug' });  // Too verbose for production!

// ❌ BAD - Logging sensitive data in production
logger.info('User login', { email, password });  // NEVER in production!

// ❌ BAD - Not using structured logging in production
console.log('User ' + userId + ' created order ' + orderId);  // Hard to query

// ❌ BAD - No sampling in high-traffic production
logger.info('Request received');  // Millions of logs/day!

// ❌ BAD - Full stack traces in production
console.error(error);  // Exposes internal structure
// Use error.message only

// ❌ BAD - Colorized logs in production
winston.format.colorize()  // ANSI codes waste space, not parseable
```

---

### **Migration Strategy:**

```typescript
// 1. Start with environment detection
const env = process.env.NODE_ENV || 'development';

// 2. Create separate configs
const devConfig = { /* verbose */ };
const prodConfig = { /* structured */ };

// 3. Gradually add sanitization
if (env === 'production') {
  format: sanitizeFormat()
}

// 4. Test both configurations
npm run test:dev
npm run test:prod

// 5. Monitor production logs
// Ensure no sensitive data leaks
// Verify performance impact
```

**Key Takeaway:** **Yes, log differently** in production vs development. Development: **verbose** (debug level), **colorized** console, **full context**, allow debugging data. Production: **structured JSON**, **info level**, **sanitized** (no sensitive data), **sampled** (10-20%), **compressed rotation**, **centralized collection** (ELK/CloudWatch), shorter stack traces, minimal request logging.

</details>

<details>
<summary><strong>44. How do you aggregate logs from multiple instances?</strong></summary>

**Answer:**

Aggregate logs from multiple instances using **centralized logging** with tools like **Elasticsearch + Kibana (ELK Stack)**, **AWS CloudWatch**, **Datadog**, **Splunk**, or **Grafana Loki**. Use **log shippers** (Fluentd, Filebeat, Promtail) to collect logs, **correlation IDs** to trace requests, and **metadata** (hostname, pod name) to identify sources.

---

### **Why Aggregate Logs:**

```typescript
// Problem: Without aggregation
server-1: "User 123 placed order"
server-2: "Payment processed for order 456"
server-3: "Email sent to user 123"
// Can't correlate across servers, hard to debug distributed issues

// Solution: With aggregation
Centralized log store:
  [server-1] User 123 placed order 456 [traceId: abc]
  [server-2] Payment processed for order 456 [traceId: abc]
  [server-3] Email sent to user 123 [traceId: abc]
// All logs in one place, searchable, correlated by traceId

Benefits:
1. Single source of truth
2. Search across all instances
3. Correlation with trace IDs
4. Performance analysis
5. Alerting and monitoring
6. Troubleshooting distributed issues
```

---

### **Method 1: ELK Stack (Elasticsearch + Logstash + Kibana)**

```yaml
# docker-compose.yml
version: '3.8'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
    ports:
      - '9200:9200'
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data

  logstash:
    image: docker.elastic.co/logstash/logstash:8.11.0
    ports:
      - '5000:5000'
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    ports:
      - '5601:5601'
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch

  nestjs-app-1:
    build: .
    environment:
      - LOGSTASH_HOST=logstash
      - LOGSTASH_PORT=5000
      - INSTANCE_ID=server-1

  nestjs-app-2:
    build: .
    environment:
      - LOGSTASH_HOST=logstash
      - LOGSTASH_PORT=5000
      - INSTANCE_ID=server-2

volumes:
  elasticsearch-data:

# logstash.conf
input {
  tcp {
    port => 5000
    codec => json
  }
}

filter {
  # Add timestamp if missing
  if ![timestamp] {
    mutate {
      add_field => { "timestamp" => "%{@timestamp}" }
    }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "nestjs-logs-%{+YYYY.MM.dd}"
  }
  stdout { codec => rubydebug }
}
```

```typescript
// logger/elk-logger.ts
import * as winston from 'winston';
import * as LogstashTransport from 'winston-logstash-transport';

export const elkLogger = winston.createLogger({
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json(),
  ),
  defaultMeta: {
    service: 'nestjs-api',
    instance: process.env.INSTANCE_ID || require('os').hostname(),
    version: process.env.APP_VERSION,
  },
  transports: [
    // Console
    new winston.transports.Console(),
    
    // Logstash (sends to Elasticsearch)
    new LogstashTransport({
      host: process.env.LOGSTASH_HOST || 'localhost',
      port: parseInt(process.env.LOGSTASH_PORT) || 5000,
    }),
  ],
});

// Logs automatically aggregated in Elasticsearch
// View in Kibana: http://localhost:5601
```

---

### **Method 2: Filebeat + Elasticsearch**

```yaml
# filebeat.yml
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/nestjs/*.log
    json.keys_under_root: true
    json.add_error_key: true
    fields:
      service: nestjs-api
      instance: ${HOSTNAME}
    fields_under_root: true

output.elasticsearch:
  hosts: ["elasticsearch:9200"]
  index: "nestjs-logs-%{+yyyy.MM.dd}"

# docker-compose.yml
version: '3.8'
services:
  nestjs-app:
    build: .
    volumes:
      - ./logs:/var/log/nestjs

  filebeat:
    image: docker.elastic.co/beats/filebeat:8.11.0
    user: root
    volumes:
      - ./filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
      - ./logs:/var/log/nestjs:ro
      - filebeat-data:/usr/share/filebeat/data
    depends_on:
      - elasticsearch

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    ports:
      - '9200:9200'

  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    ports:
      - '5601:5601'

volumes:
  filebeat-data:
```

---

### **Method 3: AWS CloudWatch Logs**

```typescript
// logger/cloudwatch-logger.ts
import * as winston from 'winston';
import WinstonCloudWatch from 'winston-cloudwatch';

export const cloudwatchLogger = winston.createLogger({
  transports: [
    new winston.transports.Console(),
    
    new WinstonCloudWatch({
      logGroupName: '/aws/nestjs-api',
      logStreamName: () => {
        // Unique stream per instance
        return `${process.env.INSTANCE_ID || require('os').hostname()}-${new Date().toISOString().split('T')[0]}`;
      },
      awsRegion: process.env.AWS_REGION,
      jsonMessage: true,
      messageFormatter: (log) => JSON.stringify({
        timestamp: new Date().toISOString(),
        level: log.level,
        message: log.message,
        service: 'nestjs-api',
        instance: process.env.INSTANCE_ID,
        ...log.meta,
      }),
    }),
  ],
});

// Query logs with CloudWatch Insights
// fields @timestamp, message, instance, level
// | filter level = "error"
// | sort @timestamp desc
// | limit 100
```

```typescript
// infrastructure/cloudwatch-setup.ts
import { CloudWatchLogs } from 'aws-sdk';

const cloudwatch = new CloudWatchLogs({ region: 'us-east-1' });

// Create log group
await cloudwatch.createLogGroup({
  logGroupName: '/aws/nestjs-api',
}).promise();

// Set retention (30 days)
await cloudwatch.putRetentionPolicy({
  logGroupName: '/aws/nestjs-api',
  retentionInDays: 30,
}).promise();

// Create metric filter (error count)
await cloudwatch.putMetricFilter({
  logGroupName: '/aws/nestjs-api',
  filterName: 'ErrorCount',
  filterPattern: '[level=ERROR]',
  metricTransformations: [{
    metricName: 'ErrorCount',
    metricNamespace: 'NestJS/API',
    metricValue: '1',
  }],
}).promise();
```

---

### **Method 4: Fluentd (Kubernetes)**

```yaml
# fluentd-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluentd-config
data:
  fluent.conf: |
    <source>
      @type tail
      path /var/log/containers/*nestjs*.log
      pos_file /var/log/fluentd-containers.log.pos
      tag kubernetes.*
      read_from_head true
      <parse>
        @type json
        time_key timestamp
        time_format %Y-%m-%dT%H:%M:%S.%NZ
      </parse>
    </source>

    <filter kubernetes.**>
      @type kubernetes_metadata
      @id filter_kube_metadata
    </filter>

    <match kubernetes.**>
      @type elasticsearch
      host elasticsearch
      port 9200
      index_name nestjs-logs
      logstash_format true
      logstash_prefix nestjs-logs
      <buffer>
        @type file
        path /var/log/fluentd-buffers/kubernetes.system.buffer
        flush_mode interval
        retry_type exponential_backoff
        flush_interval 5s
      </buffer>
    </match>

---
# fluentd-daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd-kubernetes-daemonset:v1-debian-elasticsearch
        env:
        - name: FLUENT_ELASTICSEARCH_HOST
          value: "elasticsearch"
        - name: FLUENT_ELASTICSEARCH_PORT
          value: "9200"
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
        - name: config
          mountPath: /fluentd/etc
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
      - name: config
        configMap:
          name: fluentd-config
```

---

### **Method 5: Grafana Loki + Promtail**

```yaml
# docker-compose.yml
version: '3.8'
services:
  loki:
    image: grafana/loki:latest
    ports:
      - "3100:3100"
    command: -config.file=/etc/loki/local-config.yaml
    volumes:
      - loki-data:/loki

  promtail:
    image: grafana/promtail:latest
    volumes:
      - ./logs:/var/log/nestjs
      - ./promtail-config.yml:/etc/promtail/config.yml
    command: -config.file=/etc/promtail/config.yml

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_AUTH_ANONYMOUS_ENABLED=true
      - GF_AUTH_ANONYMOUS_ORG_ROLE=Admin
    volumes:
      - grafana-data:/var/lib/grafana

  nestjs-app-1:
    build: .
    volumes:
      - ./logs:/app/logs
    environment:
      - INSTANCE_ID=server-1

  nestjs-app-2:
    build: .
    volumes:
      - ./logs:/app/logs
    environment:
      - INSTANCE_ID=server-2

volumes:
  loki-data:
  grafana-data:

# promtail-config.yml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: nestjs
    static_configs:
      - targets:
          - localhost
        labels:
          job: nestjs-api
          __path__: /var/log/nestjs/*.log
    pipeline_stages:
      - json:
          expressions:
            level: level
            message: message
            instance: instance
            traceId: traceId
      - labels:
          level:
          instance:
      - timestamp:
          source: timestamp
          format: RFC3339
```

```typescript
// Query logs in Grafana
{job="nestjs-api"} |= "error"
{job="nestjs-api", instance="server-1"} |= "order"
{job="nestjs-api"} | json | level="error" | line_format "{{.message}}"
```

---

### **Method 6: Datadog Logs**

```typescript
// logger/datadog-logger.ts
import * as winston from 'winston';

const datadogTransport = new winston.transports.Http({
  host: 'http-intake.logs.datadoghq.com',
  path: `/api/v2/logs?dd-api-key=${process.env.DD_API_KEY}&ddsource=nodejs&service=nestjs-api`,
  ssl: true,
});

export const datadogLogger = winston.createLogger({
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json(),
  ),
  defaultMeta: {
    service: 'nestjs-api',
    env: process.env.NODE_ENV,
    version: process.env.APP_VERSION,
    hostname: require('os').hostname(),
  },
  transports: [
    new winston.transports.Console(),
    datadogTransport,
  ],
});

// Or use Datadog Agent
// docker run -d --name datadog-agent \
//   -e DD_API_KEY=<your-api-key> \
//   -e DD_LOGS_ENABLED=true \
//   -e DD_LOGS_CONFIG_CONTAINER_COLLECT_ALL=true \
//   -v /var/run/docker.sock:/var/run/docker.sock:ro \
//   -v /proc/:/host/proc/:ro \
//   -v /sys/fs/cgroup/:/host/sys/fs/cgroup:ro \
//   datadog/agent:latest
```

---

### **Method 7: Correlation with Trace IDs**

```typescript
// logger/correlation.service.ts
import { Injectable } from '@nestjs/common';
import { AsyncLocalStorage } from 'async_hooks';
import { v4 as uuidv4 } from 'uuid';

@Injectable()
export class CorrelationService {
  private readonly asyncLocalStorage = new AsyncLocalStorage<Map<string, any>>();

  run(callback: () => void) {
    const store = new Map();
    store.set('traceId', uuidv4());
    store.set('timestamp', new Date().toISOString());
    this.asyncLocalStorage.run(store, callback);
  }

  getTraceId(): string {
    return this.asyncLocalStorage.getStore()?.get('traceId');
  }

  setMetadata(key: string, value: any) {
    this.asyncLocalStorage.getStore()?.set(key, value);
  }

  getMetadata(): Record<string, any> {
    const store = this.asyncLocalStorage.getStore();
    return store ? Object.fromEntries(store) : {};
  }
}

// middleware/correlation.middleware.ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { CorrelationService } from '../logger/correlation.service';

@Injectable()
export class CorrelationMiddleware implements NestMiddleware {
  constructor(private correlationService: CorrelationService) {}

  use(req: any, res: any, next: () => void) {
    this.correlationService.run(() => {
      // Extract trace ID from header or generate new one
      const traceId = req.headers['x-trace-id'] || this.correlationService.getTraceId();
      
      this.correlationService.setMetadata('traceId', traceId);
      this.correlationService.setMetadata('requestId', req.id);
      this.correlationService.setMetadata('userId', req.user?.id);
      this.correlationService.setMetadata('ip', req.ip);

      // Add to response header
      res.setHeader('X-Trace-ID', traceId);

      next();
    });
  }
}

// logger/logger.service.ts
import { Injectable } from '@nestjs/common';
import { CorrelationService } from './correlation.service';

@Injectable()
export class LoggerService {
  constructor(private correlationService: CorrelationService) {}

  log(message: string, context?: any) {
    const metadata = this.correlationService.getMetadata();
    
    console.log(JSON.stringify({
      timestamp: new Date().toISOString(),
      level: 'info',
      message,
      ...metadata,  // Includes traceId, requestId, userId
      ...context,
    }));
  }
}

// Now all logs from same request have same traceId
// Can search: traceId="abc-123" across all instances
```

---

### **Method 8: Log Aggregation with Metadata**

```typescript
// logger/aggregation-logger.ts
import * as winston from 'winston';

export const aggregationLogger = winston.createLogger({
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json(),
  ),
  defaultMeta: {
    // Service identification
    service: process.env.SERVICE_NAME || 'nestjs-api',
    version: process.env.APP_VERSION || '1.0.0',
    
    // Instance identification
    instance: process.env.INSTANCE_ID || require('os').hostname(),
    container: process.env.HOSTNAME,  // Docker/K8s container ID
    pod: process.env.POD_NAME,        // Kubernetes pod name
    node: process.env.NODE_NAME,      // Kubernetes node name
    
    // Environment
    environment: process.env.NODE_ENV,
    region: process.env.AWS_REGION,
    availabilityZone: process.env.AWS_AZ,
    
    // Application context
    processId: process.pid,
    platform: process.platform,
    nodeVersion: process.version,
  },
  transports: [
    new winston.transports.Console(),
  ],
});

// Log output:
{
  "timestamp": "2025-12-30T10:30:45.123Z",
  "level": "info",
  "message": "Order created",
  "service": "nestjs-api",
  "version": "1.2.3",
  "instance": "server-1",
  "container": "abc123def456",
  "pod": "nestjs-api-deployment-abc-123",
  "node": "ip-10-0-1-123.ec2.internal",
  "environment": "production",
  "region": "us-east-1",
  "availabilityZone": "us-east-1a",
  "traceId": "abc123",
  "orderId": 456
}

// Query in Elasticsearch:
// service="nestjs-api" AND pod="nestjs-api-deployment-abc-123"
// service="nestjs-api" AND traceId="abc123"
// service="nestjs-api" AND availabilityZone="us-east-1a" AND level="error"
```

---

### **Method 9: Multi-Instance Search Example**

```typescript
// Elasticsearch query to search across all instances

// 1. Find all errors in last hour
GET /nestjs-logs-*/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "level": "error" } },
        { "range": { "@timestamp": { "gte": "now-1h" } } }
      ]
    }
  },
  "sort": [{ "@timestamp": "desc" }],
  "size": 100
}

// 2. Trace request across all instances
GET /nestjs-logs-*/_search
{
  "query": {
    "match": { "traceId": "abc-123" }
  },
  "sort": [{ "@timestamp": "asc" }]
}

// 3. Errors by instance
GET /nestjs-logs-*/_search
{
  "size": 0,
  "query": {
    "match": { "level": "error" }
  },
  "aggs": {
    "by_instance": {
      "terms": { "field": "instance" }
    }
  }
}

// 4. Performance by endpoint
GET /nestjs-logs-*/_search
{
  "size": 0,
  "aggs": {
    "by_endpoint": {
      "terms": { "field": "route" },
      "aggs": {
        "avg_duration": {
          "avg": { "field": "duration" }
        }
      }
    }
  }
}
```

---

### **Method 10: Complete Production Setup**

```typescript
// logger/logger.module.ts
import { Module, Global } from '@nestjs/common';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { WinstonModule } from 'nest-winston';
import * as winston from 'winston';
import WinstonCloudWatch from 'winston-cloudwatch';
import * as LogstashTransport from 'winston-logstash-transport';

@Global()
@Module({
  imports: [
    WinstonModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (configService: ConfigService) => {
        const transports: any[] = [
          // Always log to console (Docker/K8s captures this)
          new winston.transports.Console({
            format: winston.format.json(),
          }),
        ];

        // Add CloudWatch in AWS
        if (configService.get('AWS_REGION')) {
          transports.push(
            new WinstonCloudWatch({
              logGroupName: '/aws/nestjs-api',
              logStreamName: `${configService.get('INSTANCE_ID')}-${new Date().toISOString().split('T')[0]}`,
              awsRegion: configService.get('AWS_REGION'),
              jsonMessage: true,
            }),
          );
        }

        // Add Logstash for ELK Stack
        if (configService.get('LOGSTASH_HOST')) {
          transports.push(
            new LogstashTransport({
              host: configService.get('LOGSTASH_HOST'),
              port: parseInt(configService.get('LOGSTASH_PORT')),
            }),
          );
        }

        return {
          level: configService.get('LOG_LEVEL') || 'info',
          format: winston.format.combine(
            winston.format.timestamp(),
            winston.format.errors({ stack: true }),
            winston.format.json(),
          ),
          defaultMeta: {
            service: configService.get('SERVICE_NAME'),
            version: configService.get('APP_VERSION'),
            instance: configService.get('INSTANCE_ID'),
            environment: configService.get('NODE_ENV'),
            region: configService.get('AWS_REGION'),
            pod: configService.get('POD_NAME'),
          },
          transports,
        };
      },
    }),
  ],
})
export class LoggerModule {}
```

---

### **Best Practices:**

```typescript
// ✅ GOOD - Use centralized logging
// All instances → ELK/CloudWatch/Datadog
// Single source of truth

// ✅ GOOD - Include instance metadata
defaultMeta: {
  instance: process.env.INSTANCE_ID,
  pod: process.env.POD_NAME,
}

// ✅ GOOD - Use correlation IDs
traceId: "abc-123"  // Same across all instances for one request

// ✅ GOOD - Structured logging (JSON)
{ "timestamp": "...", "level": "info", "message": "...", ... }
// Easy to parse and search

// ✅ GOOD - Use log shippers (Filebeat, Fluentd)
// Automatically collect from all instances

// ✅ GOOD - Index by date
nestjs-logs-2025-12-30
nestjs-logs-2025-12-29
// Better performance, easier to delete old logs

// ❌ BAD - Log to local files only
// Can't search across instances!

// ❌ BAD - No instance identification
// Can't tell which instance logged what

// ❌ BAD - No correlation IDs
// Can't trace requests across services

// ❌ BAD - Unstructured logs
"User 123 placed order"
// Hard to parse and query

// ❌ BAD - Different log formats per instance
// Inconsistent, hard to aggregate
```

**Key Takeaway:** Aggregate logs from multiple instances using **centralized logging** (ELK Stack, CloudWatch, Datadog, Loki), **log shippers** (Filebeat, Fluentd, Promtail) to collect logs, **correlation IDs** to trace requests across instances, **metadata** (instance, pod, service) to identify sources, **structured JSON** for parsing, and **search/query** capabilities to analyze logs across all instances.

</details>

<details>
<summary><strong>45. What information should every log entry include?</strong></summary>

**Answer:**

Every log entry should include: **timestamp** (ISO 8601), **log level** (DEBUG/INFO/WARN/ERROR), **message** (clear, actionable), **service metadata** (name, version, environment, instance), **correlation IDs** (requestId, traceId, spanId), **user context** (userId, sessionId), **HTTP context** (method, URL, statusCode, duration), **error details** (stack, errorCode), and **business context** (orderId, customerId).

---

### **Essential Log Fields:**

```typescript
// Minimum required fields
interface LogEntry {
  // 1. Temporal (when)
  timestamp: string;              // ISO 8601: "2025-12-30T10:30:45.123Z"
  
  // 2. Severity (what level)
  level: string;                  // "debug" | "info" | "warn" | "error" | "critical"
  
  // 3. Content (what happened)
  message: string;                // Clear, actionable description
  
  // 4. Source (where)
  service: string;                // "nestjs-api"
  version: string;                // "1.2.3"
  environment: string;            // "production" | "staging" | "development"
  instance: string;               // "server-1" or hostname
  
  // 5. Context (who/why)
  traceId?: string;               // Correlate across services
  requestId?: string;             // Unique per request
  userId?: string;                // Who triggered this
  
  // 6. Details (additional info)
  [key: string]: any;             // Custom fields
}

// Example
{
  "timestamp": "2025-12-30T10:30:45.123Z",
  "level": "error",
  "message": "Payment processing failed",
  "service": "nestjs-api",
  "version": "1.2.3",
  "environment": "production",
  "instance": "server-1",
  "traceId": "abc-123-xyz",
  "requestId": "req-456",
  "userId": "user-789",
  "orderId": 12345,
  "errorCode": "PAYMENT_GATEWAY_TIMEOUT",
  "duration": 5000
}
```

---

### **Method 1: Complete Log Entry Schema**

```typescript
// logger/log-entry.interface.ts
export interface CompleteLogEntry {
  // ========== REQUIRED FIELDS ==========
  
  // Temporal
  timestamp: string;              // ISO 8601 with timezone
  
  // Severity
  level: 'debug' | 'info' | 'warn' | 'error' | 'critical';
  
  // Message
  message: string;                // Human-readable description
  
  // Service Identity
  service: string;                // Service name
  version: string;                // Application version
  environment: 'production' | 'staging' | 'development' | 'test';
  
  // Instance Identity
  instance: string;               // Hostname, container ID, or instance ID
  hostname: string;               // Server hostname
  
  // ========== RECOMMENDED FIELDS ==========
  
  // Correlation (for distributed tracing)
  traceId?: string;               // OpenTelemetry trace ID
  spanId?: string;                // OpenTelemetry span ID
  parentSpanId?: string;          // Parent span ID
  requestId?: string;             // Unique request identifier
  correlationId?: string;         // Business correlation ID
  
  // User Context
  userId?: string;                // User identifier
  sessionId?: string;             // Session identifier
  username?: string;              // Username (if applicable)
  userAgent?: string;             // Client user agent
  ipAddress?: string;             // Client IP address
  
  // HTTP Context (for API logs)
  method?: string;                // HTTP method: GET, POST, etc.
  url?: string;                   // Request URL
  route?: string;                 // Route pattern: /api/orders/:id
  statusCode?: number;            // HTTP status code: 200, 404, 500
  duration?: number;              // Request duration in milliseconds
  
  // Business Context
  orderId?: string;               // Order ID
  customerId?: string;            // Customer ID
  transactionId?: string;         // Transaction ID
  resource?: string;              // Resource being accessed
  action?: string;                // Action being performed
  
  // Error Details (for error logs)
  error?: {
    name: string;                 // Error name: "ValidationError"
    message: string;              // Error message
    code?: string;                // Error code: "E1001"
    stack?: string;               // Stack trace
    cause?: any;                  // Error cause
  };
  
  // Performance
  memory?: {
    heapUsed: number;
    heapTotal: number;
    external: number;
  };
  cpuUsage?: {
    user: number;
    system: number;
  };
  
  // Additional Metadata
  tags?: string[];                // Tags for categorization
  metadata?: Record<string, any>; // Additional context
  
  // Infrastructure (Kubernetes/Docker)
  pod?: string;                   // Kubernetes pod name
  container?: string;             // Container ID
  namespace?: string;             // Kubernetes namespace
  node?: string;                  // Kubernetes node name
  cluster?: string;               // Cluster name
  region?: string;                // AWS region, GCP zone, etc.
}
```

---

### **Method 2: Winston Format with All Fields**

```typescript
// logger/logger.service.ts
import { Injectable } from '@nestjs/common';
import * as winston from 'winston';
import { v4 as uuidv4 } from 'uuid';
import * as os from 'os';

@Injectable()
export class LoggerService {
  private logger: winston.Logger;

  constructor() {
    this.logger = winston.createLogger({
      level: process.env.LOG_LEVEL || 'info',
      format: winston.format.combine(
        winston.format.timestamp({ format: 'YYYY-MM-DDTHH:mm:ss.SSSZ' }),
        winston.format.errors({ stack: true }),
        winston.format.json(),
      ),
      defaultMeta: this.getDefaultMeta(),
      transports: [
        new winston.transports.Console(),
        new winston.transports.File({ filename: 'logs/combined.log' }),
      ],
    });
  }

  private getDefaultMeta() {
    return {
      // Service Identity
      service: process.env.SERVICE_NAME || 'nestjs-api',
      version: process.env.APP_VERSION || '1.0.0',
      environment: process.env.NODE_ENV || 'development',
      
      // Instance Identity
      instance: process.env.INSTANCE_ID || os.hostname(),
      hostname: os.hostname(),
      pid: process.pid,
      
      // Infrastructure
      pod: process.env.POD_NAME,
      container: process.env.HOSTNAME,
      namespace: process.env.NAMESPACE,
      node: process.env.NODE_NAME,
      region: process.env.AWS_REGION || process.env.GCP_REGION,
      
      // Platform
      platform: os.platform(),
      nodeVersion: process.version,
    };
  }

  log(
    level: string,
    message: string,
    context?: {
      // Correlation
      traceId?: string;
      spanId?: string;
      requestId?: string;
      
      // User Context
      userId?: string;
      sessionId?: string;
      
      // HTTP Context
      method?: string;
      url?: string;
      statusCode?: number;
      duration?: number;
      
      // Business Context
      orderId?: string;
      customerId?: string;
      
      // Error
      error?: Error;
      
      // Additional
      [key: string]: any;
    },
  ) {
    const logEntry: any = {
      message,
      ...context,
    };

    // Add error details
    if (context?.error) {
      logEntry.error = {
        name: context.error.name,
        message: context.error.message,
        stack: context.error.stack,
        code: (context.error as any).code,
      };
    }

    // Add memory usage for performance logs
    if (level === 'debug' || context?.includePerformance) {
      const memUsage = process.memoryUsage();
      logEntry.memory = {
        heapUsed: memUsage.heapUsed,
        heapTotal: memUsage.heapTotal,
        external: memUsage.external,
      };
    }

    this.logger.log(level, logEntry);
  }

  info(message: string, context?: any) {
    this.log('info', message, context);
  }

  error(message: string, error?: Error, context?: any) {
    this.log('error', message, { ...context, error });
  }

  warn(message: string, context?: any) {
    this.log('warn', message, context);
  }

  debug(message: string, context?: any) {
    this.log('debug', message, context);
  }
}

// Usage
logger.info('Order created', {
  traceId: 'abc-123',
  requestId: 'req-456',
  userId: 'user-789',
  orderId: 12345,
  customerId: 'cust-999',
  method: 'POST',
  url: '/api/orders',
  statusCode: 201,
  duration: 150,
});

// Output:
{
  "timestamp": "2025-12-30T10:30:45.123Z",
  "level": "info",
  "message": "Order created",
  "service": "nestjs-api",
  "version": "1.2.3",
  "environment": "production",
  "instance": "server-1",
  "hostname": "ip-10-0-1-123",
  "pid": 1234,
  "pod": "nestjs-api-deployment-abc-123",
  "traceId": "abc-123",
  "requestId": "req-456",
  "userId": "user-789",
  "orderId": 12345,
  "customerId": "cust-999",
  "method": "POST",
  "url": "/api/orders",
  "statusCode": 201,
  "duration": 150
}
```

---

### **Method 3: Interceptor with Complete Context**

```typescript
// interceptors/logging.interceptor.ts
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';
import { LoggerService } from '../logger/logger.service';
import { v4 as uuidv4 } from 'uuid';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  constructor(private logger: LoggerService) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const response = context.switchToHttp().getResponse();
    const startTime = Date.now();

    // Generate IDs if not present
    request.id = request.id || uuidv4();
    request.traceId = request.headers['x-trace-id'] || uuidv4();

    // Extract all context
    const logContext = {
      // Correlation
      requestId: request.id,
      traceId: request.traceId,
      
      // User Context
      userId: request.user?.id,
      username: request.user?.username,
      sessionId: request.session?.id,
      ipAddress: request.ip || request.connection.remoteAddress,
      userAgent: request.headers['user-agent'],
      
      // HTTP Context
      method: request.method,
      url: request.url,
      route: request.route?.path,
      protocol: request.protocol,
      
      // Request Details
      query: request.query,
      params: request.params,
      headers: this.sanitizeHeaders(request.headers),
    };

    // Log request
    this.logger.info('Incoming request', logContext);

    return next.handle().pipe(
      tap({
        next: (data) => {
          const duration = Date.now() - startTime;
          
          this.logger.info('Request completed', {
            ...logContext,
            statusCode: response.statusCode,
            duration,
            responseSize: JSON.stringify(data || {}).length,
          });
        },
        error: (error) => {
          const duration = Date.now() - startTime;
          
          this.logger.error('Request failed', error, {
            ...logContext,
            statusCode: response.statusCode || 500,
            duration,
            errorCode: error.code,
            errorName: error.name,
          });
        },
      }),
    );
  }

  private sanitizeHeaders(headers: any): any {
    const sanitized = { ...headers };
    // Remove sensitive headers
    delete sanitized.authorization;
    delete sanitized.cookie;
    delete sanitized['x-api-key'];
    return sanitized;
  }
}
```

---

### **Method 4: Structured Log Builder**

```typescript
// logger/log-builder.ts
export class LogBuilder {
  private entry: any = {};

  // Required fields
  setMessage(message: string): this {
    this.entry.message = message;
    return this;
  }

  setLevel(level: 'debug' | 'info' | 'warn' | 'error' | 'critical'): this {
    this.entry.level = level;
    return this;
  }

  // Correlation
  setTraceId(traceId: string): this {
    this.entry.traceId = traceId;
    return this;
  }

  setRequestId(requestId: string): this {
    this.entry.requestId = requestId;
    return this;
  }

  setCorrelationId(correlationId: string): this {
    this.entry.correlationId = correlationId;
    return this;
  }

  // User context
  setUser(userId: string, username?: string): this {
    this.entry.userId = userId;
    if (username) this.entry.username = username;
    return this;
  }

  setSession(sessionId: string): this {
    this.entry.sessionId = sessionId;
    return this;
  }

  // HTTP context
  setRequest(method: string, url: string, statusCode?: number): this {
    this.entry.method = method;
    this.entry.url = url;
    if (statusCode) this.entry.statusCode = statusCode;
    return this;
  }

  setDuration(duration: number): this {
    this.entry.duration = duration;
    return this;
  }

  // Business context
  setOrder(orderId: string): this {
    this.entry.orderId = orderId;
    return this;
  }

  setCustomer(customerId: string): this {
    this.entry.customerId = customerId;
    return this;
  }

  setTransaction(transactionId: string): this {
    this.entry.transactionId = transactionId;
    return this;
  }

  // Error context
  setError(error: Error): this {
    this.entry.error = {
      name: error.name,
      message: error.message,
      stack: error.stack,
      code: (error as any).code,
    };
    return this;
  }

  // Custom fields
  setCustom(key: string, value: any): this {
    this.entry[key] = value;
    return this;
  }

  setMetadata(metadata: Record<string, any>): this {
    this.entry.metadata = metadata;
    return this;
  }

  // Build
  build(): any {
    return {
      timestamp: new Date().toISOString(),
      ...this.entry,
    };
  }

  // Log directly
  log(logger: any): void {
    const entry = this.build();
    logger[entry.level || 'info'](entry.message, entry);
  }
}

// Usage
new LogBuilder()
  .setLevel('info')
  .setMessage('Order created successfully')
  .setTraceId('abc-123')
  .setRequestId('req-456')
  .setUser('user-789', 'john@example.com')
  .setRequest('POST', '/api/orders', 201)
  .setDuration(150)
  .setOrder('order-12345')
  .setCustomer('cust-999')
  .setCustom('paymentMethod', 'credit_card')
  .log(logger);
```

---

### **Method 5: Field Priority by Log Level**

```typescript
// logger/field-priority.ts
export class LogFieldPriority {
  // Required for ALL logs
  static getRequiredFields() {
    return {
      timestamp: new Date().toISOString(),
      level: 'info',
      message: '',
      service: process.env.SERVICE_NAME,
      version: process.env.APP_VERSION,
      environment: process.env.NODE_ENV,
      instance: process.env.INSTANCE_ID || require('os').hostname(),
    };
  }

  // Required for DEBUG logs
  static getDebugFields(context: any) {
    return {
      ...this.getRequiredFields(),
      // Include everything for debugging
      traceId: context.traceId,
      requestId: context.requestId,
      userId: context.userId,
      method: context.method,
      url: context.url,
      query: context.query,
      params: context.params,
      body: context.body,  // Full body in debug
      headers: context.headers,
      memory: process.memoryUsage(),
      cpu: process.cpuUsage(),
    };
  }

  // Required for INFO logs
  static getInfoFields(context: any) {
    return {
      ...this.getRequiredFields(),
      traceId: context.traceId,
      requestId: context.requestId,
      userId: context.userId,
      method: context.method,
      url: context.url,
      statusCode: context.statusCode,
      duration: context.duration,
      // Business context
      orderId: context.orderId,
      customerId: context.customerId,
    };
  }

  // Required for WARN logs
  static getWarnFields(context: any) {
    return {
      ...this.getRequiredFields(),
      traceId: context.traceId,
      requestId: context.requestId,
      userId: context.userId,
      method: context.method,
      url: context.url,
      statusCode: context.statusCode,
      // Warning details
      warningCode: context.warningCode,
      warningMessage: context.warningMessage,
    };
  }

  // Required for ERROR logs
  static getErrorFields(error: Error, context: any) {
    return {
      ...this.getRequiredFields(),
      level: 'error',
      // Correlation (critical for debugging)
      traceId: context.traceId,
      requestId: context.requestId,
      userId: context.userId,
      // HTTP context
      method: context.method,
      url: context.url,
      statusCode: context.statusCode || 500,
      // Error details (MUST include)
      error: {
        name: error.name,
        message: error.message,
        stack: error.stack,
        code: (error as any).code,
        cause: (error as any).cause,
      },
      // Business context (for root cause)
      orderId: context.orderId,
      customerId: context.customerId,
      transactionId: context.transactionId,
    };
  }
}

// Usage
logger.info('Order created', LogFieldPriority.getInfoFields(context));
logger.error('Payment failed', LogFieldPriority.getErrorFields(error, context));
```

---

### **Method 6: Field Validation**

```typescript
// logger/log-validator.ts
import { IsString, IsNotEmpty, IsEnum, IsOptional, IsNumber, validate } from 'class-validator';

enum LogLevel {
  DEBUG = 'debug',
  INFO = 'info',
  WARN = 'warn',
  ERROR = 'error',
  CRITICAL = 'critical',
}

export class LogEntryDto {
  @IsString()
  @IsNotEmpty()
  timestamp!: string;

  @IsEnum(LogLevel)
  level!: LogLevel;

  @IsString()
  @IsNotEmpty()
  message!: string;

  @IsString()
  @IsNotEmpty()
  service!: string;

  @IsString()
  @IsNotEmpty()
  version!: string;

  @IsString()
  @IsNotEmpty()
  environment!: string;

  @IsString()
  @IsNotEmpty()
  instance!: string;

  @IsString()
  @IsOptional()
  traceId?: string;

  @IsString()
  @IsOptional()
  requestId?: string;

  @IsString()
  @IsOptional()
  userId?: string;

  @IsString()
  @IsOptional()
  method?: string;

  @IsString()
  @IsOptional()
  url?: string;

  @IsNumber()
  @IsOptional()
  statusCode?: number;

  @IsNumber()
  @IsOptional()
  duration?: number;
}

export class LogValidator {
  static async validateLogEntry(entry: any): Promise<{ valid: boolean; errors: string[] }> {
    const logEntry = Object.assign(new LogEntryDto(), entry);
    const errors = await validate(logEntry);

    if (errors.length > 0) {
      const errorMessages = errors.map(err => 
        Object.values(err.constraints || {}).join(', ')
      );
      return { valid: false, errors: errorMessages };
    }

    return { valid: true, errors: [] };
  }

  static ensureRequiredFields(entry: any): any {
    const required = {
      timestamp: entry.timestamp || new Date().toISOString(),
      level: entry.level || 'info',
      message: entry.message || 'No message provided',
      service: entry.service || process.env.SERVICE_NAME || 'unknown',
      version: entry.version || process.env.APP_VERSION || '0.0.0',
      environment: entry.environment || process.env.NODE_ENV || 'development',
      instance: entry.instance || require('os').hostname(),
    };

    return { ...required, ...entry };
  }
}

// Usage
const logEntry = {
  message: 'Order created',
  level: 'info',
  orderId: 12345,
};

const validated = LogValidator.ensureRequiredFields(logEntry);
const { valid, errors } = await LogValidator.validateLogEntry(validated);

if (!valid) {
  console.error('Invalid log entry:', errors);
} else {
  logger.info(validated);
}
```

---

### **Method 7: OpenTelemetry Semantic Conventions**

```typescript
// logger/otel-fields.ts
import { SemanticAttributes } from '@opentelemetry/semantic-conventions';

export class OtelLogFields {
  static httpRequest(req: any) {
    return {
      // HTTP
      [SemanticAttributes.HTTP_METHOD]: req.method,
      [SemanticAttributes.HTTP_URL]: req.url,
      [SemanticAttributes.HTTP_TARGET]: req.path,
      [SemanticAttributes.HTTP_HOST]: req.hostname,
      [SemanticAttributes.HTTP_SCHEME]: req.protocol,
      [SemanticAttributes.HTTP_STATUS_CODE]: req.statusCode,
      [SemanticAttributes.HTTP_USER_AGENT]: req.headers['user-agent'],
      [SemanticAttributes.HTTP_REQUEST_CONTENT_LENGTH]: req.headers['content-length'],
      
      // Network
      [SemanticAttributes.NET_PEER_IP]: req.ip,
      [SemanticAttributes.NET_PEER_PORT]: req.socket?.remotePort,
      
      // User
      'enduser.id': req.user?.id,
      'enduser.username': req.user?.username,
    };
  }

  static database(operation: string, table: string, query?: string) {
    return {
      [SemanticAttributes.DB_SYSTEM]: 'postgresql',
      [SemanticAttributes.DB_NAME]: process.env.DB_NAME,
      [SemanticAttributes.DB_OPERATION]: operation,
      [SemanticAttributes.DB_SQL_TABLE]: table,
      [SemanticAttributes.DB_STATEMENT]: query,
    };
  }

  static messaging(system: string, destination: string) {
    return {
      [SemanticAttributes.MESSAGING_SYSTEM]: system,
      [SemanticAttributes.MESSAGING_DESTINATION]: destination,
      [SemanticAttributes.MESSAGING_OPERATION]: 'send',
    };
  }

  static error(error: Error) {
    return {
      'exception.type': error.name,
      'exception.message': error.message,
      'exception.stacktrace': error.stack,
      'exception.escaped': false,
    };
  }
}

// Usage (standardized field names)
logger.info('HTTP request', {
  ...OtelLogFields.httpRequest(request),
  traceId: 'abc-123',
  spanId: 'span-456',
});

logger.info('Database query', {
  ...OtelLogFields.database('SELECT', 'orders', 'SELECT * FROM orders WHERE id = $1'),
  traceId: 'abc-123',
});
```

---

### **Method 8: Environment-Specific Fields**

```typescript
// logger/environment-fields.ts
export class EnvironmentFields {
  static development() {
    return {
      // Include more debug info
      environment: 'development',
      includeStackTrace: true,
      includeRequestBody: true,
      includeResponseBody: true,
      includeHeaders: true,
      includeQuery: true,
      includeMemoryUsage: true,
    };
  }

  static staging() {
    return {
      environment: 'staging',
      includeStackTrace: true,
      includeRequestBody: false,    // Sanitized
      includeResponseBody: false,
      includeHeaders: false,         // Sanitized
      includeQuery: true,
      includeMemoryUsage: false,
    };
  }

  static production() {
    return {
      environment: 'production',
      includeStackTrace: false,      // Only for errors
      includeRequestBody: false,     // Never log in production
      includeResponseBody: false,
      includeHeaders: false,
      includeQuery: false,
      includeMemoryUsage: false,
    };
  }

  static getCurrent() {
    const env = process.env.NODE_ENV || 'development';
    
    switch (env) {
      case 'production':
        return this.production();
      case 'staging':
        return this.staging();
      default:
        return this.development();
    }
  }
}

// Usage
const config = EnvironmentFields.getCurrent();

logger.info('Request', {
  message: 'User login',
  ...(config.includeRequestBody && { body: req.body }),
  ...(config.includeHeaders && { headers: req.headers }),
  ...(config.includeMemoryUsage && { memory: process.memoryUsage() }),
});
```

---

### **Method 9: Complete Example with All Fields**

```typescript
// Example: E-commerce order creation log

// Complete log entry
{
  // ========== REQUIRED ==========
  "timestamp": "2025-12-30T10:30:45.123Z",
  "level": "info",
  "message": "Order created successfully",
  
  // Service Identity
  "service": "nestjs-api",
  "version": "1.2.3",
  "environment": "production",
  
  // Instance Identity
  "instance": "server-1",
  "hostname": "ip-10-0-1-123.ec2.internal",
  "pid": 1234,
  
  // ========== CORRELATION ==========
  "traceId": "abc-123-xyz-789",
  "spanId": "span-456",
  "parentSpanId": "span-123",
  "requestId": "req-789-abc",
  
  // ========== USER CONTEXT ==========
  "userId": "user-123",
  "username": "john@example.com",
  "sessionId": "sess-456",
  "ipAddress": "192.168.1.100",
  "userAgent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)...",
  
  // ========== HTTP CONTEXT ==========
  "method": "POST",
  "url": "/api/orders",
  "route": "/api/orders",
  "protocol": "https",
  "statusCode": 201,
  "duration": 150,
  
  // ========== BUSINESS CONTEXT ==========
  "orderId": "order-12345",
  "customerId": "cust-999",
  "transactionId": "txn-888",
  "paymentMethod": "credit_card",
  "totalAmount": 99.99,
  "currency": "USD",
  "items": [
    { "productId": "prod-111", "quantity": 2, "price": 49.99 }
  ],
  
  // ========== INFRASTRUCTURE ==========
  "pod": "nestjs-api-deployment-abc-123",
  "container": "abc123def456",
  "namespace": "production",
  "node": "node-1",
  "cluster": "prod-cluster-us-east-1",
  "region": "us-east-1",
  "availabilityZone": "us-east-1a",
  
  // ========== PERFORMANCE ==========
  "memory": {
    "heapUsed": 45678912,
    "heapTotal": 67108864,
    "external": 1234567
  },
  "cpu": {
    "user": 123456,
    "system": 78901
  },
  
  // ========== METADATA ==========
  "tags": ["order", "payment", "success"],
  "metadata": {
    "promotionCode": "SUMMER2025",
    "referralSource": "google",
    "deviceType": "desktop"
  }
}

// For ERROR log, add:
{
  // ... all above fields ...
  "level": "error",
  "message": "Order creation failed",
  
  // Error details
  "error": {
    "name": "PaymentGatewayError",
    "message": "Payment gateway timeout",
    "code": "GATEWAY_TIMEOUT",
    "stack": "Error: Payment gateway timeout\n    at PaymentService.process...",
    "cause": {
      "gatewayResponse": "504 Gateway Timeout",
      "attemptNumber": 3
    }
  },
  
  // Additional error context
  "errorCode": "E1001",
  "errorCategory": "payment",
  "severity": "high",
  "retryable": true
}
```

---

### **Method 10: Log Schema Documentation**

```typescript
// docs/log-schema.md
/**
 * Standard Log Entry Schema
 * 
 * This document defines the standard fields that MUST and SHOULD be included
 * in every log entry across all services.
 * 
 * Field Categories:
 * - REQUIRED: Must be present in every log
 * - RECOMMENDED: Should be present when applicable
 * - OPTIONAL: May be present for additional context
 */

export interface LogSchema {
  // ========== REQUIRED (MUST) ==========
  timestamp: string;              // ISO 8601 format with timezone
  level: LogLevel;                // Standardized log level
  message: string;                // Clear, actionable message
  service: string;                // Service name
  version: string;                // Application version
  environment: Environment;       // Deployment environment
  instance: string;               // Instance/hostname identifier

  // ========== RECOMMENDED (SHOULD) ==========
  traceId?: string;               // Distributed tracing ID
  requestId?: string;             // Unique request identifier
  userId?: string;                // User identifier (when authenticated)
  
  // ========== HTTP (when applicable) ==========
  method?: HttpMethod;            // HTTP method
  url?: string;                   // Request URL
  statusCode?: number;            // HTTP status code
  duration?: number;              // Request duration (ms)
  
  // ========== ERROR (when level=error) ==========
  error?: {
    name: string;                 // Error name/type
    message: string;              // Error message
    code?: string;                // Error code
    stack?: string;               // Stack trace
  };
  
  // ========== OPTIONAL (additional context) ==========
  [key: string]: any;             // Custom business fields
}

// Field naming conventions
const conventions = {
  // Use camelCase for field names
  goodExample: 'userId',
  badExample: 'user_id',
  
  // Use descriptive names
  goodExample2: 'customerId',
  badExample2: 'cid',
  
  // Include units in name
  goodExample3: 'durationMs',
  badExample3: 'duration',  // Unclear units
  
  // Use standard prefixes
  errorCode: 'For error codes',
  isEnabled: 'For boolean flags',
  totalAmount: 'For calculated values',
};

// Validation
export function validateLogEntry(entry: any): boolean {
  const required = ['timestamp', 'level', 'message', 'service', 'version', 'environment', 'instance'];
  return required.every(field => entry[field] !== undefined);
}

// Testing
import { validateLogEntry } from './log-schema';

describe('Log Schema', () => {
  it('should validate required fields', () => {
    const validEntry = {
      timestamp: '2025-12-30T10:30:45.123Z',
      level: 'info',
      message: 'Test message',
      service: 'nestjs-api',
      version: '1.0.0',
      environment: 'test',
      instance: 'test-instance',
    };
    
    expect(validateLogEntry(validEntry)).toBe(true);
  });

  it('should reject missing required fields', () => {
    const invalidEntry = {
      message: 'Test message',
    };
    
    expect(validateLogEntry(invalidEntry)).toBe(false);
  });
});
```

---

### **Comparison: Field Priority by Use Case**

| Use Case | Required Fields | Recommended Fields | Optional Fields |
|----------|----------------|-------------------|-----------------|
| **API Request** | timestamp, level, message, service, version, instance | traceId, requestId, userId, method, url, statusCode, duration | query, params, headers |
| **Database Query** | timestamp, level, message, service, version, instance | traceId, dbOperation, table, duration | query, rowsAffected |
| **Error Log** | timestamp, level, message, service, version, instance, error (name, message, stack) | traceId, requestId, userId, errorCode | context, cause, retryable |
| **Business Event** | timestamp, level, message, service, version, instance | traceId, userId, entityId (orderId, customerId) | businessContext, metadata |
| **Performance** | timestamp, level, message, service, version, instance | duration, memory, cpu | cache, database, external |
| **Security Event** | timestamp, level, message, service, version, instance, userId, ipAddress, action | success, reason | metadata, geolocation |

---

### **Best Practices:**

```typescript
// ✅ GOOD - Include all required fields
{
  "timestamp": "2025-12-30T10:30:45.123Z",
  "level": "info",
  "message": "Order created",
  "service": "nestjs-api",
  "version": "1.2.3",
  "environment": "production",
  "instance": "server-1"
}

// ✅ GOOD - Use ISO 8601 for timestamp
"timestamp": "2025-12-30T10:30:45.123Z"

// ✅ GOOD - Include correlation IDs
"traceId": "abc-123", "requestId": "req-456"

// ✅ GOOD - Include user context
"userId": "user-789", "sessionId": "sess-abc"

// ✅ GOOD - Include HTTP context
"method": "POST", "url": "/api/orders", "statusCode": 201, "duration": 150

// ✅ GOOD - Include business context
"orderId": "12345", "customerId": "cust-999"

// ✅ GOOD - Include full error details
"error": {
  "name": "ValidationError",
  "message": "Invalid email",
  "code": "E1001",
  "stack": "Error: ..."
}

// ❌ BAD - Missing required fields
{ "message": "Something happened" }
// Can't tell when, where, or severity

// ❌ BAD - Non-standard timestamp
"timestamp": "12/30/2025 10:30:45"
// Use ISO 8601

// ❌ BAD - No correlation IDs
// Can't trace across services

// ❌ BAD - No user context
// Can't identify who triggered the action

// ❌ BAD - Vague message
"message": "Error"
// Be specific!

// ❌ BAD - Missing error details
"error": "Something went wrong"
// Include name, message, stack, code
```

**Key Takeaway:** Every log entry should include **required fields** (timestamp, level, message, service, version, environment, instance), **correlation IDs** (traceId, requestId) for distributed tracing, **user context** (userId, sessionId) when applicable, **HTTP context** (method, URL, statusCode, duration) for API logs, **business context** (orderId, customerId) for domain events, and **error details** (name, message, stack, code) for error logs. Use **structured JSON**, **ISO 8601 timestamps**, **standardized field names**, and **validate schema** to ensure consistency across all services.

</details>
