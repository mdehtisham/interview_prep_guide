# NestJS Architecture & Design Patterns - Top Interview Questions

## Architecture Fundamentals

<details>
<summary><strong>1. What is the recommended architecture in NestJS (layered architecture)?</strong></summary>

**Answer:**

- NestJS recommends a **layered (n-tier) architecture** based on **separation of concerns**
- Each layer has a specific responsibility
- The typical layers are:
  - **Presentation (Controllers)**
  - **Business Logic (Services)**
  - **Data Access (Repositories)**
  - **Domain (Entities/Models)**
- This architecture promotes **maintainability**, **testability**, and **scalability**

---

### **Layered Architecture in NestJS:**

```
┌─────────────────────────────────────────┐
│         Presentation Layer              │
│         (Controllers)                   │
│  - Handle HTTP requests/responses       │
│  - Route requests                       │
│  - Validate input (DTOs)                │
│  - Return responses                     │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│      Business Logic Layer               │
│         (Services)                      │
│  - Business rules                       │
│  - Orchestration                        │
│  - Data transformation                  │
│  - Call repositories                    │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│      Data Access Layer                  │
│      (Repositories)                     │
│  - Database operations                  │
│  - Query building                       │
│  - Data persistence                     │
│  - ORM interaction                      │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│         Domain Layer                    │
│      (Entities/Models)                  │
│  - Business entities                    │
│  - Domain logic                         │
│  - Data structures                      │
└─────────────────────────────────────────┘
```

---

### **Method 1: Basic Layered Architecture Example**

```typescript
// ========== DOMAIN LAYER ==========
// entities/user.entity.ts
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  email: string;

  @Column()
  name: string;

  @Column()
  password: string;

  @Column({ default: true })
  isActive: boolean;

  @Column({ type: 'timestamp', default: () => 'CURRENT_TIMESTAMP' })
  createdAt: Date;

  // Domain method
  activate(): void {
    this.isActive = true;
  }

  deactivate(): void {
    this.isActive = false;
  }
}

// ========== DATA ACCESS LAYER ==========
// repositories/user.repository.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from '../entities/user.entity';

@Injectable()
export class UserRepository {
  constructor(
    @InjectRepository(User)
    private readonly repository: Repository<User>,
  ) {}

  // Data access methods only
  async findById(id: string): Promise<User | null> {
    return this.repository.findOne({ where: { id } });
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.repository.findOne({ where: { email } });
  }

  async save(user: User): Promise<User> {
    return this.repository.save(user);
  }

  async remove(user: User): Promise<void> {
    await this.repository.remove(user);
  }

  async findActiveUsers(): Promise<User[]> {
    return this.repository.find({ where: { isActive: true } });
  }
}

// ========== BUSINESS LOGIC LAYER ==========
// services/user.service.ts
import { Injectable, NotFoundException, ConflictException } from '@nestjs/common';
import { UserRepository } from '../repositories/user.repository';
import { User } from '../entities/user.entity';
import { CreateUserDto } from '../dto/create-user.dto';
import { UpdateUserDto } from '../dto/update-user.dto';
import * as bcrypt from 'bcrypt';

@Injectable()
export class UserService {
  constructor(private readonly userRepository: UserRepository) {}

  // Business logic: user registration
  async registerUser(createUserDto: CreateUserDto): Promise<User> {
    // Check if user already exists
    const existingUser = await this.userRepository.findByEmail(createUserDto.email);
    if (existingUser) {
      throw new ConflictException('User with this email already exists');
    }

    // Hash password (business rule)
    const hashedPassword = await bcrypt.hash(createUserDto.password, 10);

    // Create user entity
    const user = new User();
    user.email = createUserDto.email;
    user.name = createUserDto.name;
    user.password = hashedPassword;

    // Save to database
    return this.userRepository.save(user);
  }

  // Business logic: get user profile
  async getUserProfile(userId: string): Promise<User> {
    const user = await this.userRepository.findById(userId);
    if (!user) {
      throw new NotFoundException(`User with ID ${userId} not found`);
    }
    return user;
  }

  // Business logic: update user
  async updateUser(userId: string, updateUserDto: UpdateUserDto): Promise<User> {
    const user = await this.getUserProfile(userId);
    
    // Apply updates
    if (updateUserDto.name) {
      user.name = updateUserDto.name;
    }
    if (updateUserDto.email) {
      user.email = updateUserDto.email;
    }

    return this.userRepository.save(user);
  }

  // Business logic: deactivate user
  async deactivateUser(userId: string): Promise<User> {
    const user = await this.getUserProfile(userId);
    user.deactivate();  // Use domain method
    return this.userRepository.save(user);
  }

  // Business logic: get all active users
  async getActiveUsers(): Promise<User[]> {
    return this.userRepository.findActiveUsers();
  }
}

// ========== PRESENTATION LAYER ==========
// controllers/user.controller.ts
import { Controller, Get, Post, Put, Delete, Body, Param, HttpCode, HttpStatus } from '@nestjs/common';
import { UserService } from '../services/user.service';
import { CreateUserDto } from '../dto/create-user.dto';
import { UpdateUserDto } from '../dto/update-user.dto';
import { UserResponseDto } from '../dto/user-response.dto';

@Controller('users')
export class UserController {
  constructor(private readonly userService: UserService) {}

  // Handle HTTP POST /users
  @Post()
  @HttpCode(HttpStatus.CREATED)
  async register(@Body() createUserDto: CreateUserDto): Promise<UserResponseDto> {
    const user = await this.userService.registerUser(createUserDto);
    return this.mapToResponse(user);
  }

  // Handle HTTP GET /users/:id
  @Get(':id')
  async getProfile(@Param('id') id: string): Promise<UserResponseDto> {
    const user = await this.userService.getUserProfile(id);
    return this.mapToResponse(user);
  }

  // Handle HTTP PUT /users/:id
  @Put(':id')
  async update(
    @Param('id') id: string,
    @Body() updateUserDto: UpdateUserDto,
  ): Promise<UserResponseDto> {
    const user = await this.userService.updateUser(id, updateUserDto);
    return this.mapToResponse(user);
  }

  // Handle HTTP POST /users/:id/deactivate
  @Post(':id/deactivate')
  async deactivate(@Param('id') id: string): Promise<UserResponseDto> {
    const user = await this.userService.deactivateUser(id);
    return this.mapToResponse(user);
  }

  // Handle HTTP GET /users/active
  @Get('active/list')
  async getActiveUsers(): Promise<UserResponseDto[]> {
    const users = await this.userService.getActiveUsers();
    return users.map(user => this.mapToResponse(user));
  }

  // Map entity to response DTO (don't expose password)
  private mapToResponse(user: any): UserResponseDto {
    return {
      id: user.id,
      email: user.email,
      name: user.name,
      isActive: user.isActive,
      createdAt: user.createdAt,
    };
  }
}

// ========== DTOs (Data Transfer Objects) ==========
// dto/create-user.dto.ts
import { IsEmail, IsString, MinLength, MaxLength } from 'class-validator';

export class CreateUserDto {
  @IsEmail()
  email: string;

  @IsString()
  @MinLength(2)
  @MaxLength(50)
  name: string;

  @IsString()
  @MinLength(8)
  password: string;
}

// dto/user-response.dto.ts
export class UserResponseDto {
  id: string;
  email: string;
  name: string;
  isActive: boolean;
  createdAt: Date;
}
```

---

### **Method 2: Module Organization**

```typescript
// user.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UserController } from './controllers/user.controller';
import { UserService } from './services/user.service';
import { UserRepository } from './repositories/user.repository';
import { User } from './entities/user.entity';

@Module({
  imports: [
    TypeOrmModule.forFeature([User]),  // Register entity
  ],
  controllers: [UserController],        // Presentation layer
  providers: [
    UserService,                        // Business logic layer
    UserRepository,                     // Data access layer
  ],
  exports: [UserService],               // Export service for other modules
})
export class UserModule {}
```

---

### **Method 3: Benefits of Layered Architecture**

```typescript
// ✅ Separation of Concerns
// Each layer has a single responsibility

// Controller: ONLY handles HTTP
@Controller('orders')
export class OrderController {
  // No business logic here!
  // No database queries here!
  // Only routing and delegation
}

// Service: ONLY contains business logic
@Injectable()
export class OrderService {
  // No HTTP handling here!
  // Business rules and orchestration only
  async createOrder(userId: string, items: any[]): Promise<Order> {
    // Validate business rules
    if (items.length === 0) {
      throw new BadRequestException('Order must have at least one item');
    }
    // Calculate total
    const total = items.reduce((sum, item) => sum + item.price * item.quantity, 0);
    // Apply discount (business rule)
    const discount = this.calculateDiscount(total);
    // Save order
    return this.orderRepository.save({ userId, items, total, discount });
  }
}

// Repository: ONLY handles data access
@Injectable()
export class OrderRepository {
  // No business logic here!
  // Only database operations
  async save(order: Order): Promise<Order> {
    return this.repository.save(order);
  }
}

// ✅ Testability
// Each layer can be tested independently

// Test controller without database
describe('OrderController', () => {
  it('should call service', async () => {
    const mockService = { createOrder: jest.fn() };
    const controller = new OrderController(mockService as any);
    await controller.create({});
    expect(mockService.createOrder).toHaveBeenCalled();
  });
});

// Test service without HTTP
describe('OrderService', () => {
  it('should calculate discount', async () => {
    const mockRepo = { save: jest.fn() };
    const service = new OrderService(mockRepo as any);
    // Test business logic in isolation
  });
});

// ✅ Reusability
// Service can be used by multiple controllers
@Controller('admin/orders')
export class AdminOrderController {
  constructor(private orderService: OrderService) {}
  // Reuse same service
}

@Controller('api/orders')
export class ApiOrderController {
  constructor(private orderService: OrderService) {}
  // Reuse same service
}

// ✅ Maintainability
// Changes to one layer don't affect others

// Change database from PostgreSQL to MongoDB
// Only need to update Repository layer
// Service and Controller remain unchanged

@Injectable()
export class OrderRepository {
  // Changed from TypeORM to Mongoose
  constructor(@InjectModel(Order.name) private model: Model<Order>) {}
  
  async save(order: Order): Promise<Order> {
    return this.model.create(order);  // Changed implementation
  }
  // Service and Controller code stays the same!
}
```

---

### **Method 4: Layer Communication Rules**

```typescript
// ✅ CORRECT: Top-down dependency flow
Controller → Service → Repository → Database

// Controller depends on Service
export class UserController {
  constructor(private userService: UserService) {}
}

// Service depends on Repository
export class UserService {
  constructor(private userRepository: UserRepository) {}
}

// ❌ WRONG: Skip layers
export class UserController {
  // Don't inject repository directly in controller!
  constructor(private userRepository: UserRepository) {}
  
  async getUser(id: string) {
    // Controller should NOT call repository directly
    return this.userRepository.findById(id);
  }
}

// ❌ WRONG: Bottom-up dependency
export class UserRepository {
  // Repository should NOT depend on Service!
  constructor(private userService: UserService) {}
}

// ✅ CORRECT: Cross-cutting concerns (shared services)
export class UserService {
  constructor(
    private userRepository: UserRepository,
    private emailService: EmailService,      // OK: shared service
    private loggerService: LoggerService,    // OK: shared service
  ) {}
}
```

---

### **Method 5: Advanced Layered Architecture (with Application Layer)**

```typescript
// Add Application Layer for complex use cases

// ========== DOMAIN LAYER ==========
// domain/order.entity.ts
export class Order {
  id: string;
  userId: string;
  items: OrderItem[];
  status: OrderStatus;
  total: number;

  // Domain logic
  canBeCancelled(): boolean {
    return this.status === OrderStatus.PENDING;
  }

  cancel(): void {
    if (!this.canBeCancelled()) {
      throw new Error('Order cannot be cancelled');
    }
    this.status = OrderStatus.CANCELLED;
  }
}

// ========== DATA ACCESS LAYER ==========
// repositories/order.repository.ts
@Injectable()
export class OrderRepository {
  async findById(id: string): Promise<Order> { /*...*/ }
  async save(order: Order): Promise<Order> { /*...*/ }
}

// ========== DOMAIN SERVICE LAYER ==========
// domain/services/order-domain.service.ts
@Injectable()
export class OrderDomainService {
  // Domain logic: calculate discount based on business rules
  calculateDiscount(total: number, userLevel: string): number {
    if (userLevel === 'VIP') return total * 0.2;
    if (userLevel === 'PREMIUM') return total * 0.1;
    if (total > 100) return total * 0.05;
    return 0;
  }

  // Domain logic: validate order
  validateOrder(order: Order): void {
    if (order.items.length === 0) {
      throw new Error('Order must have items');
    }
    if (order.total < 0) {
      throw new Error('Invalid order total');
    }
  }
}

// ========== APPLICATION SERVICE LAYER ==========
// services/order.service.ts
@Injectable()
export class OrderService {
  constructor(
    private orderRepository: OrderRepository,
    private orderDomainService: OrderDomainService,
    private userService: UserService,           // Orchestration
    private paymentService: PaymentService,     // Orchestration
    private emailService: EmailService,         // Orchestration
  ) {}

  // Application logic: orchestrate use case
  async createOrder(userId: string, items: OrderItem[]): Promise<Order> {
    // 1. Get user (orchestration)
    const user = await this.userService.findById(userId);

    // 2. Create order entity
    const order = new Order();
    order.userId = userId;
    order.items = items;
    order.total = items.reduce((sum, item) => sum + item.price, 0);

    // 3. Apply domain logic
    const discount = this.orderDomainService.calculateDiscount(order.total, user.level);
    order.total -= discount;

    // 4. Validate (domain logic)
    this.orderDomainService.validateOrder(order);

    // 5. Save order
    const savedOrder = await this.orderRepository.save(order);

    // 6. Process payment (orchestration)
    await this.paymentService.processPayment(savedOrder.id, order.total);

    // 7. Send confirmation email (orchestration)
    await this.emailService.sendOrderConfirmation(user.email, savedOrder);

    return savedOrder;
  }
}

// ========== PRESENTATION LAYER ==========
// controllers/order.controller.ts
@Controller('orders')
export class OrderController {
  constructor(private orderService: OrderService) {}

  @Post()
  async create(@Body() dto: CreateOrderDto) {
    return this.orderService.createOrder(dto.userId, dto.items);
  }
}
```

---

### **Best Practices:**

```typescript
// ✅ GOOD: Clear layer separation
// Controller: HTTP only
@Controller('users')
export class UserController {
  constructor(private userService: UserService) {}
  
  @Post()
  async create(@Body() dto: CreateUserDto) {
    return this.userService.createUser(dto);
  }
}

// Service: Business logic only
@Injectable()
export class UserService {
  constructor(private userRepository: UserRepository) {}
  
  async createUser(dto: CreateUserDto): Promise<User> {
    // Business logic here
    const hashedPassword = await bcrypt.hash(dto.password, 10);
    return this.userRepository.save({ ...dto, password: hashedPassword });
  }
}

// Repository: Data access only
@Injectable()
export class UserRepository {
  async save(user: User): Promise<User> {
    return this.repository.save(user);
  }
}

// ❌ BAD: Mixed responsibilities
@Controller('users')
export class UserController {
  @Post()
  async create(@Body() dto: CreateUserDto) {
    // Don't put business logic in controller!
    const hashedPassword = await bcrypt.hash(dto.password, 10);
    // Don't access database directly from controller!
    return this.repository.save({ ...dto, password: hashedPassword });
  }
}

// ✅ GOOD: Dependency direction (top-down)
Controller → Service → Repository

// ❌ BAD: Circular dependencies
Service ↔ Controller  // Never do this!

// ✅ GOOD: One responsibility per layer
// Controller: Routing, validation, response formatting
// Service: Business logic, orchestration
// Repository: Data persistence, queries
// Entity: Data structure, domain logic

// ❌ BAD: Everything in one place
@Controller('users')
export class UserController {
  // Controller doing everything - hard to test, maintain, reuse
}
```

**Key Takeaway:**

- NestJS recommends **layered architecture** with clear separation:
  - **Controllers** handle HTTP (routing, validation, responses)
  - **Services** contain business logic (rules, orchestration, calculations)
  - **Repositories** manage data access (queries, persistence)
  - **Entities** represent domain models (data structure, domain logic)
- Each layer has **single responsibility**, depends only on layers below, and can be tested independently
- This promotes **maintainability**, **testability**, **scalability**, and **reusability**

</details>

<details>
<summary><strong>2. What are the typical layers (Controller, Service, Repository)?</strong></summary>

**Answer:**

The typical layers in NestJS are: **Controller Layer** (handles HTTP requests/responses, routing), **Service Layer** (contains business logic, orchestration), **Repository Layer** (manages data access, database operations), and **Entity/Domain Layer** (data models, domain logic). Each layer has a specific responsibility following **Separation of Concerns** principle.

---

### **Layer Breakdown:**

```typescript
┌──────────────────────────────────────────────────────────────┐
│                     CONTROLLER LAYER                         │
│  Responsibility: Handle HTTP communication                    │
│  - Route HTTP requests to appropriate methods                │
│  - Validate request data (DTOs, pipes)                       │
│  - Transform responses                                        │
│  - Handle authentication/authorization decorators            │
│  - Return HTTP status codes                                   │
│                                                              │
│  What NOT to do:                                             │
│  ❌ Business logic                                           │
│  ❌ Direct database access                                   │
│  ❌ Complex calculations                                     │
└──────────────────────────────────────────────────────────────┘
                            ↓
┌──────────────────────────────────────────────────────────────┐
│                      SERVICE LAYER                           │
│  Responsibility: Business logic and orchestration            │
│  - Implement business rules                                  │
│  - Orchestrate multiple operations                           │
│  - Call repositories                                         │
│  - Transform data                                            │
│  - Coordinate with other services                            │
│                                                              │
│  What NOT to do:                                             │
│  ❌ HTTP-specific code                                       │
│  ❌ Direct database queries                                  │
│  ❌ Response formatting                                      │
└──────────────────────────────────────────────────────────────┘
                            ↓
┌──────────────────────────────────────────────────────────────┐
│                    REPOSITORY LAYER                          │
│  Responsibility: Data access and persistence                 │
│  - Execute database queries                                  │
│  - Handle ORM operations                                     │
│  - Manage transactions                                       │
│  - Abstract database details                                 │
│                                                              │
│  What NOT to do:                                             │
│  ❌ Business logic                                           │
│  ❌ HTTP handling                                            │
│  ❌ Complex calculations                                     │
└──────────────────────────────────────────────────────────────┘
                            ↓
┌──────────────────────────────────────────────────────────────┐
│                   ENTITY/DOMAIN LAYER                        │
│  Responsibility: Data structure and domain logic             │
│  - Define data models                                        │
│  - Domain-specific methods                                   │
│  - Value objects                                             │
│  - Business entities                                         │
└──────────────────────────────────────────────────────────────┘
```

---

### **Method 1: Controller Layer - HTTP Handling**

```typescript
// controllers/product.controller.ts
import {
  Controller,
  Get,
  Post,
  Put,
  Delete,
  Body,
  Param,
  Query,
  HttpCode,
  HttpStatus,
  UseGuards,
  ParseUUIDPipe,
} from '@nestjs/common';
import { ProductService } from '../services/product.service';
import { CreateProductDto } from '../dto/create-product.dto';
import { UpdateProductDto } from '../dto/update-product.dto';
import { JwtAuthGuard } from '../guards/jwt-auth.guard';

@Controller('products')
@UseGuards(JwtAuthGuard)  // Controller handles authentication
export class ProductController {
  constructor(private readonly productService: ProductService) {}

  // ========== CONTROLLER RESPONSIBILITIES ==========

  // 1. Route HTTP requests
  @Post()
  @HttpCode(HttpStatus.CREATED)
  async create(@Body() createProductDto: CreateProductDto) {
    // Delegate to service
    return this.productService.createProduct(createProductDto);
  }

  // 2. Handle path parameters with validation
  @Get(':id')
  async findOne(@Param('id', ParseUUIDPipe) id: string) {
    // Validate UUID format, delegate to service
    return this.productService.getProductById(id);
  }

  // 3. Handle query parameters
  @Get()
  async findAll(
    @Query('page') page: number = 1,
    @Query('limit') limit: number = 10,
    @Query('category') category?: string,
  ) {
    // Extract query params, delegate to service
    return this.productService.getProducts({ page, limit, category });
  }

  // 4. Update operations
  @Put(':id')
  async update(
    @Param('id', ParseUUIDPipe) id: string,
    @Body() updateProductDto: UpdateProductDto,
  ) {
    return this.productService.updateProduct(id, updateProductDto);
  }

  // 5. Delete operations
  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  async remove(@Param('id', ParseUUIDPipe) id: string) {
    await this.productService.deleteProduct(id);
  }

  // 6. Response transformation (if needed)
  @Get(':id/details')
  async getDetails(@Param('id', ParseUUIDPipe) id: string) {
    const product = await this.productService.getProductWithDetails(id);
    
    // Transform response for API
    return {
      id: product.id,
      name: product.name,
      price: product.price,
      available: product.stock > 0,
      category: product.category.name,
    };
  }
}

// ❌ WRONG: Business logic in controller
@Controller('products')
export class BadProductController {
  @Post()
  async create(@Body() dto: CreateProductDto) {
    // Don't do calculations in controller!
    const discountedPrice = dto.price * 0.9;
    
    // Don't access database directly from controller!
    const product = await this.repository.save({ ...dto, price: discountedPrice });
    
    return product;
  }
}
```

---

### **Method 2: Service Layer - Business Logic**

```typescript
// services/product.service.ts
import { Injectable, NotFoundException, BadRequestException } from '@nestjs/common';
import { ProductRepository } from '../repositories/product.repository';
import { Product } from '../entities/product.entity';
import { CreateProductDto } from '../dto/create-product.dto';
import { UpdateProductDto } from '../dto/update-product.dto';

@Injectable()
export class ProductService {
  constructor(
    private readonly productRepository: ProductRepository,
    private readonly categoryService: CategoryService,  // Orchestration
    private readonly inventoryService: InventoryService,  // Orchestration
  ) {}

  // ========== SERVICE RESPONSIBILITIES ==========

  // 1. Business logic: Create product with validation
  async createProduct(dto: CreateProductDto): Promise<Product> {
    // Business rule: Check if product name already exists
    const existing = await this.productRepository.findByName(dto.name);
    if (existing) {
      throw new BadRequestException('Product with this name already exists');
    }

    // Business rule: Validate category exists
    const category = await this.categoryService.findById(dto.categoryId);
    if (!category) {
      throw new BadRequestException('Invalid category');
    }

    // Business rule: Calculate discounted price
    const finalPrice = this.calculatePrice(dto.price, dto.discount);

    // Business rule: Validate price
    if (finalPrice < 0) {
      throw new BadRequestException('Price cannot be negative');
    }

    // Create product entity
    const product = new Product();
    product.name = dto.name;
    product.price = finalPrice;
    product.category = category;
    product.stock = dto.initialStock || 0;

    // Save through repository
    const savedProduct = await this.productRepository.save(product);

    // Orchestration: Update inventory
    await this.inventoryService.createInventoryRecord(savedProduct.id, savedProduct.stock);

    return savedProduct;
  }

  // 2. Business logic: Get product with validation
  async getProductById(id: string): Promise<Product> {
    const product = await this.productRepository.findById(id);
    
    if (!product) {
      throw new NotFoundException(`Product with ID ${id} not found`);
    }

    return product;
  }

  // 3. Business logic: Complex query with filtering
  async getProducts(options: {
    page: number;
    limit: number;
    category?: string;
  }): Promise<{ data: Product[]; total: number }> {
    // Business rule: Validate pagination
    if (options.page < 1) {
      throw new BadRequestException('Page must be >= 1');
    }
    if (options.limit < 1 || options.limit > 100) {
      throw new BadRequestException('Limit must be between 1 and 100');
    }

    // Call repository with calculated offset
    const offset = (options.page - 1) * options.limit;
    
    return this.productRepository.findWithPagination({
      offset,
      limit: options.limit,
      category: options.category,
    });
  }

  // 4. Business logic: Update with validation
  async updateProduct(id: string, dto: UpdateProductDto): Promise<Product> {
    const product = await this.getProductById(id);

    // Business rule: Can't update if product is discontinued
    if (product.isDiscontinued) {
      throw new BadRequestException('Cannot update discontinued product');
    }

    // Apply updates
    if (dto.name) product.name = dto.name;
    if (dto.price) {
      // Business rule: Price can't be reduced by more than 50%
      if (dto.price < product.price * 0.5) {
        throw new BadRequestException('Price reduction too large');
      }
      product.price = dto.price;
    }

    return this.productRepository.save(product);
  }

  // 5. Business logic: Delete with checks
  async deleteProduct(id: string): Promise<void> {
    const product = await this.getProductById(id);

    // Business rule: Can't delete if there are pending orders
    const hasPendingOrders = await this.productRepository.hasPendingOrders(product.id);
    if (hasPendingOrders) {
      throw new BadRequestException('Cannot delete product with pending orders');
    }

    await this.productRepository.remove(product);
  }

  // Private helper: Business calculation
  private calculatePrice(basePrice: number, discount: number = 0): number {
    if (discount < 0 || discount > 100) {
      throw new BadRequestException('Discount must be between 0 and 100');
    }
    return basePrice * (1 - discount / 100);
  }

  // Orchestration: Multiple operations
  async getProductWithDetails(id: string): Promise<any> {
    const product = await this.getProductById(id);
    
    // Orchestrate calls to multiple services
    const [category, inventory, reviews] = await Promise.all([
      this.categoryService.findById(product.categoryId),
      this.inventoryService.getByProductId(product.id),
      this.reviewService.getByProductId(product.id),
    ]);

    return {
      ...product,
      category,
      inventory,
      averageRating: this.calculateAverageRating(reviews),
      reviewCount: reviews.length,
    };
  }

  private calculateAverageRating(reviews: any[]): number {
    if (reviews.length === 0) return 0;
    const sum = reviews.reduce((acc, review) => acc + review.rating, 0);
    return sum / reviews.length;
  }
}
```

---

### **Method 3: Repository Layer - Data Access**

```typescript
// repositories/product.repository.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Product } from '../entities/product.entity';

@Injectable()
export class ProductRepository {
  constructor(
    @InjectRepository(Product)
    private readonly repository: Repository<Product>,
  ) {}

  // ========== REPOSITORY RESPONSIBILITIES ==========

  // 1. Basic CRUD operations
  async save(product: Product): Promise<Product> {
    return this.repository.save(product);
  }

  async findById(id: string): Promise<Product | null> {
    return this.repository.findOne({
      where: { id },
      relations: ['category'],  // Load relations
    });
  }

  async findByName(name: string): Promise<Product | null> {
    return this.repository.findOne({ where: { name } });
  }

  async remove(product: Product): Promise<void> {
    await this.repository.remove(product);
  }

  // 2. Complex queries
  async findWithPagination(options: {
    offset: number;
    limit: number;
    category?: string;
  }): Promise<{ data: Product[]; total: number }> {
    const queryBuilder = this.repository.createQueryBuilder('product')
      .leftJoinAndSelect('product.category', 'category')
      .skip(options.offset)
      .take(options.limit);

    // Add category filter if provided
    if (options.category) {
      queryBuilder.where('category.name = :category', { category: options.category });
    }

    const [data, total] = await queryBuilder.getManyAndCount();

    return { data, total };
  }

  // 3. Specialized queries
  async findByPriceRange(min: number, max: number): Promise<Product[]> {
    return this.repository.createQueryBuilder('product')
      .where('product.price BETWEEN :min AND :max', { min, max })
      .orderBy('product.price', 'ASC')
      .getMany();
  }

  async findLowStock(threshold: number): Promise<Product[]> {
    return this.repository.createQueryBuilder('product')
      .where('product.stock < :threshold', { threshold })
      .andWhere('product.isDiscontinued = false')
      .getMany();
  }

  async hasPendingOrders(productId: string): Promise<boolean> {
    const count = await this.repository.createQueryBuilder('product')
      .innerJoin('product.orderItems', 'orderItem')
      .innerJoin('orderItem.order', 'order')
      .where('product.id = :productId', { productId })
      .andWhere('order.status IN (:...statuses)', { statuses: ['pending', 'processing'] })
      .getCount();

    return count > 0;
  }

  // 4. Aggregation queries
  async getTotalValue(): Promise<number> {
    const result = await this.repository.createQueryBuilder('product')
      .select('SUM(product.price * product.stock)', 'total')
      .getRawOne();

    return result?.total || 0;
  }

  async getCountByCategory(): Promise<{ category: string; count: number }[]> {
    return this.repository.createQueryBuilder('product')
      .select('category.name', 'category')
      .addSelect('COUNT(product.id)', 'count')
      .innerJoin('product.category', 'category')
      .groupBy('category.name')
      .getRawMany();
  }

  // 5. Batch operations
  async saveMany(products: Product[]): Promise<Product[]> {
    return this.repository.save(products);
  }

  async updateStock(productId: string, quantity: number): Promise<void> {
    await this.repository.update(productId, { stock: quantity });
  }

  // 6. Transaction support
  async updateWithTransaction(id: string, updates: Partial<Product>): Promise<Product> {
    return this.repository.manager.transaction(async (manager) => {
      const product = await manager.findOne(Product, { where: { id } });
      if (!product) throw new Error('Product not found');
      
      Object.assign(product, updates);
      return manager.save(product);
    });
  }
}

// ❌ WRONG: Business logic in repository
@Injectable()
export class BadProductRepository {
  async save(product: Product): Promise<Product> {
    // Don't put business logic in repository!
    if (product.price < 0) {
      throw new Error('Invalid price');  // This belongs in service!
    }
    
    // Don't orchestrate multiple services from repository!
    await this.emailService.notifyNewProduct(product);  // Wrong layer!
    
    return this.repository.save(product);
  }
}
```

---

### **Method 4: Entity/Domain Layer - Data Models**

```typescript
// entities/product.entity.ts
import { Entity, Column, PrimaryGeneratedColumn, ManyToOne, OneToMany, CreateDateColumn, UpdateDateColumn } from 'typeorm';
import { Category } from './category.entity';
import { OrderItem } from './order-item.entity';

@Entity('products')
export class Product {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ length: 255 })
  name: string;

  @Column('text', { nullable: true })
  description: string;

  @Column('decimal', { precision: 10, scale: 2 })
  price: number;

  @Column('int', { default: 0 })
  stock: number;

  @Column({ default: false })
  isDiscontinued: boolean;

  @ManyToOne(() => Category, category => category.products)
  category: Category;

  @Column()
  categoryId: string;

  @OneToMany(() => OrderItem, orderItem => orderItem.product)
  orderItems: OrderItem[];

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  // ========== DOMAIN METHODS ==========
  // Domain logic: Check if product is available
  isAvailable(): boolean {
    return !this.isDiscontinued && this.stock > 0;
  }

  // Domain logic: Check if can be ordered
  canOrder(quantity: number): boolean {
    return this.isAvailable() && this.stock >= quantity;
  }

  // Domain logic: Reduce stock
  reduceStock(quantity: number): void {
    if (!this.canOrder(quantity)) {
      throw new Error('Insufficient stock');
    }
    this.stock -= quantity;
  }

  // Domain logic: Restock
  restock(quantity: number): void {
    if (quantity < 0) {
      throw new Error('Quantity must be positive');
    }
    this.stock += quantity;
  }

  // Domain logic: Discontinue product
  discontinue(): void {
    this.isDiscontinued = true;
  }

  // Domain logic: Calculate discounted price
  getPriceWithDiscount(discountPercent: number): number {
    if (discountPercent < 0 || discountPercent > 100) {
      throw new Error('Invalid discount percentage');
    }
    return this.price * (1 - discountPercent / 100);
  }
}

// Value Object example
export class Money {
  constructor(
    public readonly amount: number,
    public readonly currency: string,
  ) {
    if (amount < 0) {
      throw new Error('Amount cannot be negative');
    }
  }

  add(other: Money): Money {
    if (this.currency !== other.currency) {
      throw new Error('Cannot add different currencies');
    }
    return new Money(this.amount + other.amount, this.currency);
  }

  multiply(factor: number): Money {
    return new Money(this.amount * factor, this.currency);
  }

  toString(): string {
    return `${this.amount} ${this.currency}`;
  }
}
```

---

### **Method 5: Complete Example - All Layers Working Together**

```typescript
// ========== REQUEST FLOW ==========

// 1. HTTP Request arrives
POST /products
{
  "name": "Laptop",
  "price": 999.99,
  "categoryId": "cat-123",
  "discount": 10,
  "initialStock": 50
}

// 2. CONTROLLER receives request
@Controller('products')
export class ProductController {
  @Post()
  async create(@Body() dto: CreateProductDto) {
    // ✅ Controller: Validate DTO, delegate to service
    return this.productService.createProduct(dto);
  }
}

// 3. SERVICE applies business logic
@Injectable()
export class ProductService {
  async createProduct(dto: CreateProductDto): Promise<Product> {
    // ✅ Service: Check business rules
    const existing = await this.productRepository.findByName(dto.name);
    if (existing) throw new ConflictException();

    // ✅ Service: Validate category
    const category = await this.categoryService.findById(dto.categoryId);

    // ✅ Service: Calculate price
    const finalPrice = dto.price * (1 - dto.discount / 100);

    // ✅ Service: Create entity
    const product = new Product();
    product.name = dto.name;
    product.price = finalPrice;
    product.stock = dto.initialStock;
    product.category = category;

    // ✅ Service: Save via repository
    return this.productRepository.save(product);
  }
}

// 4. REPOSITORY accesses database
@Injectable()
export class ProductRepository {
  async save(product: Product): Promise<Product> {
    // ✅ Repository: Execute database operation
    return this.repository.save(product);
  }
}

// 5. ENTITY represents data
@Entity('products')
export class Product {
  // ✅ Entity: Data structure + domain methods
  id: string;
  name: string;
  price: number;
  stock: number;

  isAvailable(): boolean {
    return this.stock > 0;
  }
}

// 6. Response returns to client
{
  "id": "prod-456",
  "name": "Laptop",
  "price": 899.99,
  "stock": 50,
  "createdAt": "2025-12-30T10:00:00Z"
}
```

---

### **Layer Comparison Table:**

| Layer | Responsibility | Can Access | Cannot Access | Example |
|-------|---------------|-----------|---------------|---------|
| **Controller** | HTTP handling, routing, validation | Services, Guards, Pipes | Repositories, Database | `@Get()`, `@Post()`, `@Body()` |
| **Service** | Business logic, orchestration | Repositories, Other Services | HTTP objects (req, res) | Business rules, calculations |
| **Repository** | Data access, queries | Database, ORM | Services, Controllers | `save()`, `findById()`, queries |
| **Entity** | Data model, domain logic | Nothing (pure data) | Services, Repositories | Data fields, domain methods |

---

### **Best Practices:**

```typescript
// ✅ GOOD: Each layer has single responsibility
// Controller: HTTP only
@Controller('users')
export class UserController {
  @Post()
  create(@Body() dto: CreateUserDto) {
    return this.userService.create(dto);
  }
}

// Service: Business logic only
@Injectable()
export class UserService {
  async create(dto: CreateUserDto): Promise<User> {
    const hashedPassword = await bcrypt.hash(dto.password, 10);
    return this.userRepository.save({ ...dto, password: hashedPassword });
  }
}

// Repository: Data access only
@Injectable()
export class UserRepository {
  async save(user: User): Promise<User> {
    return this.repository.save(user);
  }
}

// ❌ BAD: Mixed responsibilities
@Controller('users')
export class BadUserController {
  @Post()
  async create(@Body() dto: CreateUserDto) {
    // Don't hash password in controller!
    const hashedPassword = await bcrypt.hash(dto.password, 10);
    // Don't access database from controller!
    return this.repository.save({ ...dto, password: hashedPassword });
  }
}

// ✅ GOOD: Dependency direction
Controller → Service → Repository → Database

// ❌ BAD: Skip layers
Controller → Repository  // Don't skip service layer!

// ✅ GOOD: Testability
// Test service without database
const mockRepo = { save: jest.fn() };
const service = new UserService(mockRepo as any);

// Test controller without service implementation
const mockService = { create: jest.fn() };
const controller = new UserController(mockService as any);
```

**Key Takeaway:**

- The typical layers in NestJS are:
  - **Controller** (HTTP routing, validation, response formatting)
  - **Service** (business logic, rules, orchestration)
  - **Repository** (data access, queries, persistence)
  - **Entity** (data models, domain logic)
- Each layer has **single responsibility**, depends only on layers below, communicates through well-defined interfaces, and can be tested independently
- **Controllers** handle HTTP, **Services** handle business logic, **Repositories** handle data, and **Entities** represent domain models

</details>

<details>
<summary><strong>3. What is Separation of Concerns (SoC)?</strong></summary>

**Answer:**

- **Separation of Concerns (SoC)** is a design principle where different parts of an application are divided based on their specific functionality or concern
- In NestJS, this means:
  - **Controllers handle HTTP**
  - **Services handle business logic**
  - **Repositories handle data access**
  - **Entities represent domain models**
- Each component has a **single, well-defined responsibility** and doesn't mix concerns from other layers

---

### **Why Separation of Concerns Matters:**

```typescript
// Without SoC (BAD) - Everything mixed together
@Controller('users')
export class BadUserController {
  @Post()
  async create(@Body() body: any) {
    // ❌ Mixing HTTP, validation, business logic, and database access
    
    // HTTP concern
    const { email, password, name } = body;
    
    // Validation concern
    if (!email || !email.includes('@')) {
      throw new Error('Invalid email');
    }
    
    // Business logic concern
    const hashedPassword = await bcrypt.hash(password, 10);
    
    // Database concern
    const user = await this.connection.query(
      'INSERT INTO users (email, password, name) VALUES ($1, $2, $3)',
      [email, hashedPassword, name]
    );
    
    // Email concern
    await this.smtp.send({
      to: email,
      subject: 'Welcome',
      body: 'Welcome to our app!'
    });
    
    return user;
  }
}
// Problems:
// - Hard to test (need database, SMTP, HTTP mocks)
// - Hard to maintain (one class does everything)
// - Hard to reuse (can't use logic outside HTTP context)
// - Hard to change (changing database affects everything)

// With SoC (GOOD) - Each concern separated
@Controller('users')
export class UserController {
  constructor(private userService: UserService) {}
  
  @Post()
  async create(@Body() dto: CreateUserDto) {
    // ✅ Only HTTP concern: routing and delegation
    return this.userService.createUser(dto);
  }
}

@Injectable()
export class UserService {
  constructor(
    private userRepository: UserRepository,
    private emailService: EmailService,
  ) {}
  
  async createUser(dto: CreateUserDto): Promise<User> {
    // ✅ Only business logic concern
    const hashedPassword = await bcrypt.hash(dto.password, 10);
    const user = await this.userRepository.save({ ...dto, password: hashedPassword });
    await this.emailService.sendWelcomeEmail(user.email);
    return user;
  }
}

@Injectable()
export class UserRepository {
  // ✅ Only data access concern
  async save(user: User): Promise<User> {
    return this.repository.save(user);
  }
}

// Benefits:
// ✅ Easy to test (mock each dependency)
// ✅ Easy to maintain (find and fix issues in specific layer)
// ✅ Easy to reuse (service can be used by CLI, cron jobs, etc.)
// ✅ Easy to change (swap database without touching business logic)
```

---

### **Method 1: Concerns by Layer**

```typescript
// ========== CONCERN 1: HTTP Communication ==========
// controllers/order.controller.ts
@Controller('orders')
export class OrderController {
  constructor(private orderService: OrderService) {}

  // Concern: Handle HTTP request/response
  @Post()
  @HttpCode(HttpStatus.CREATED)
  @UseGuards(JwtAuthGuard)
  async create(
    @Body() createOrderDto: CreateOrderDto,
    @Req() req: Request,
  ) {
    // Only HTTP concerns:
    // - Extract user from request
    // - Validate DTO (done by ValidationPipe)
    // - Set HTTP status code
    // - Return response
    const userId = req.user.id;
    return this.orderService.createOrder(userId, createOrderDto);
  }

  @Get(':id')
  async findOne(@Param('id', ParseUUIDPipe) id: string) {
    // Only HTTP concerns:
    // - Parse and validate path parameter
    // - Route to appropriate method
    return this.orderService.getOrder(id);
  }
}

// ========== CONCERN 2: Business Logic ==========
// services/order.service.ts
@Injectable()
export class OrderService {
  constructor(
    private orderRepository: OrderRepository,
    private productService: ProductService,
    private paymentService: PaymentService,
    private inventoryService: InventoryService,
    private notificationService: NotificationService,
  ) {}

  // Concern: Business rules and orchestration
  async createOrder(userId: string, dto: CreateOrderDto): Promise<Order> {
    // Business rule: Validate products exist
    const products = await this.productService.findByIds(dto.productIds);
    if (products.length !== dto.productIds.length) {
      throw new BadRequestException('Some products not found');
    }

    // Business rule: Check inventory
    for (const item of dto.items) {
      const available = await this.inventoryService.checkStock(item.productId, item.quantity);
      if (!available) {
        throw new BadRequestException(`Insufficient stock for product ${item.productId}`);
      }
    }

    // Business calculation: Calculate total
    const total = dto.items.reduce((sum, item) => {
      const product = products.find(p => p.id === item.productId);
      return sum + (product.price * item.quantity);
    }, 0);

    // Business rule: Apply discount
    const discount = this.calculateDiscount(total, userId);
    const finalTotal = total - discount;

    // Create order
    const order = await this.orderRepository.create({
      userId,
      items: dto.items,
      total: finalTotal,
      discount,
      status: OrderStatus.PENDING,
    });

    // Orchestration: Process payment
    await this.paymentService.processPayment(order.id, finalTotal);

    // Orchestration: Reserve inventory
    await this.inventoryService.reserveItems(dto.items);

    // Orchestration: Send notification
    await this.notificationService.sendOrderConfirmation(userId, order.id);

    return order;
  }

  private calculateDiscount(total: number, userId: string): number {
    // Business logic for discount calculation
    if (total > 1000) return total * 0.1;
    if (total > 500) return total * 0.05;
    return 0;
  }
}

// ========== CONCERN 3: Data Access ==========
// repositories/order.repository.ts
@Injectable()
export class OrderRepository {
  constructor(
    @InjectRepository(Order)
    private repository: Repository<Order>,
  ) {}

  // Concern: Database operations only
  async create(data: Partial<Order>): Promise<Order> {
    const order = this.repository.create(data);
    return this.repository.save(order);
  }

  async findById(id: string): Promise<Order | null> {
    return this.repository.findOne({
      where: { id },
      relations: ['items', 'user'],
    });
  }

  async findByUserId(userId: string): Promise<Order[]> {
    return this.repository.find({
      where: { userId },
      order: { createdAt: 'DESC' },
    });
  }

  async updateStatus(id: string, status: OrderStatus): Promise<void> {
    await this.repository.update(id, { status });
  }
}

// ========== CONCERN 4: Domain Logic ==========
// entities/order.entity.ts
@Entity('orders')
export class Order {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  userId: string;

  @Column('decimal')
  total: number;

  @Column({ type: 'enum', enum: OrderStatus })
  status: OrderStatus;

  @OneToMany(() => OrderItem, item => item.order)
  items: OrderItem[];

  // Concern: Domain behavior
  canBeCancelled(): boolean {
    return this.status === OrderStatus.PENDING || this.status === OrderStatus.PROCESSING;
  }

  cancel(): void {
    if (!this.canBeCancelled()) {
      throw new Error('Order cannot be cancelled');
    }
    this.status = OrderStatus.CANCELLED;
  }

  markAsShipped(): void {
    if (this.status !== OrderStatus.PROCESSING) {
      throw new Error('Only processing orders can be shipped');
    }
    this.status = OrderStatus.SHIPPED;
  }
}

// ========== CONCERN 5: Validation ==========
// dto/create-order.dto.ts
export class CreateOrderDto {
  @IsArray()
  @ValidateNested({ each: true })
  @Type(() => OrderItemDto)
  items: OrderItemDto[];

  @IsUUID('4', { each: true })
  productIds: string[];
}

// ========== CONCERN 6: External Communication ==========
// services/notification.service.ts
@Injectable()
export class NotificationService {
  // Concern: Email/SMS/Push notifications
  async sendOrderConfirmation(userId: string, orderId: string): Promise<void> {
    // Send email
    // Send SMS
    // Send push notification
  }
}
```

---

### **Method 2: Vertical Slicing (Feature-based SoC)**

```typescript
// Organize by feature, each feature has its own concerns separated

src/
├── features/
│   ├── users/
│   │   ├── controllers/          # HTTP concern
│   │   │   └── user.controller.ts
│   │   ├── services/             # Business logic concern
│   │   │   └── user.service.ts
│   │   ├── repositories/         # Data access concern
│   │   │   └── user.repository.ts
│   │   ├── entities/             # Domain concern
│   │   │   └── user.entity.ts
│   │   ├── dto/                  # Validation concern
│   │   │   ├── create-user.dto.ts
│   │   │   └── update-user.dto.ts
│   │   └── user.module.ts
│   │
│   ├── orders/
│   │   ├── controllers/
│   │   ├── services/
│   │   ├── repositories/
│   │   ├── entities/
│   │   ├── dto/
│   │   └── order.module.ts
│   │
│   └── products/
│       ├── controllers/
│       ├── services/
│       ├── repositories/
│       ├── entities/
│       ├── dto/
│       └── product.module.ts
│
└── shared/                       # Shared concerns
    ├── database/                 # Database concern
    ├── config/                   # Configuration concern
    ├── logging/                  # Logging concern
    ├── auth/                     # Authentication concern
    └── validation/               # Validation concern
```

---

### **Method 3: Cross-cutting Concerns**

```typescript
// Some concerns span multiple features (cross-cutting concerns)

// ========== CONCERN: Logging ==========
// shared/logging/logger.service.ts
@Injectable()
export class LoggerService {
  // Concern: Logging only
  log(message: string, context?: string): void {
    console.log(`[${context}] ${message}`);
  }

  error(message: string, trace?: string, context?: string): void {
    console.error(`[${context}] ${message}`, trace);
  }
}

// ========== CONCERN: Authentication ==========
// shared/auth/jwt-auth.guard.ts
@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  // Concern: Authentication only
  canActivate(context: ExecutionContext): boolean | Promise<boolean> {
    return super.canActivate(context) as Promise<boolean>;
  }
}

// ========== CONCERN: Error Handling ==========
// shared/filters/http-exception.filter.ts
@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  // Concern: Error handling and response formatting
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const status = exception.getStatus();

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      message: exception.message,
    });
  }
}

// ========== CONCERN: Data Transformation ==========
// shared/interceptors/transform.interceptor.ts
@Injectable()
export class TransformInterceptor implements NestInterceptor {
  // Concern: Response transformation only
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => ({
        success: true,
        data,
        timestamp: new Date().toISOString(),
      })),
    );
  }
}

// Usage: Apply cross-cutting concerns declaratively
@Controller('users')
@UseGuards(JwtAuthGuard)          // Authentication concern
@UseFilters(HttpExceptionFilter)  // Error handling concern
@UseInterceptors(TransformInterceptor)  // Transformation concern
export class UserController {
  constructor(
    private userService: UserService,
    private logger: LoggerService,  // Logging concern
  ) {}

  @Post()
  async create(@Body() dto: CreateUserDto) {
    this.logger.log('Creating user', 'UserController');
    return this.userService.createUser(dto);
  }
}
```

---

### **Method 4: Testing with SoC**

```typescript
// SoC makes testing much easier - test each concern independently

// Test HTTP concern (Controller)
describe('UserController', () => {
  let controller: UserController;
  let service: UserService;

  beforeEach(() => {
    // Mock service (don't need real business logic)
    service = {
      createUser: jest.fn(),
      findById: jest.fn(),
    } as any;

    controller = new UserController(service);
  });

  it('should call service.createUser', async () => {
    const dto = { email: 'test@test.com', name: 'Test' };
    await controller.create(dto);

    // Test only HTTP concern: controller delegates to service
    expect(service.createUser).toHaveBeenCalledWith(dto);
  });
});

// Test Business Logic concern (Service)
describe('UserService', () => {
  let service: UserService;
  let repository: UserRepository;
  let emailService: EmailService;

  beforeEach(() => {
    // Mock dependencies (don't need real database or email)
    repository = {
      save: jest.fn(),
      findByEmail: jest.fn(),
    } as any;

    emailService = {
      sendWelcomeEmail: jest.fn(),
    } as any;

    service = new UserService(repository, emailService);
  });

  it('should hash password before saving', async () => {
    const dto = { email: 'test@test.com', password: 'plain123', name: 'Test' };
    repository.findByEmail = jest.fn().mockResolvedValue(null);
    repository.save = jest.fn().mockImplementation(user => user);

    await service.createUser(dto);

    // Test only business logic concern: password hashing
    expect(repository.save).toHaveBeenCalledWith(
      expect.objectContaining({
        email: dto.email,
        password: expect.not.stringContaining('plain123'), // Password should be hashed
      })
    );
  });

  it('should send welcome email after user creation', async () => {
    const dto = { email: 'test@test.com', password: 'pass123', name: 'Test' };
    repository.findByEmail = jest.fn().mockResolvedValue(null);
    repository.save = jest.fn().mockResolvedValue({ id: '1', ...dto });

    await service.createUser(dto);

    // Test only business logic concern: email orchestration
    expect(emailService.sendWelcomeEmail).toHaveBeenCalledWith(dto.email);
  });
});

// Test Data Access concern (Repository)
describe('UserRepository', () => {
  let repository: UserRepository;
  let typeormRepository: Repository<User>;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UserRepository,
        {
          provide: getRepositoryToken(User),
          useValue: {
            save: jest.fn(),
            findOne: jest.fn(),
          },
        },
      ],
    }).compile();

    repository = module.get<UserRepository>(UserRepository);
    typeormRepository = module.get(getRepositoryToken(User));
  });

  it('should save user to database', async () => {
    const user = { email: 'test@test.com', name: 'Test' };
    typeormRepository.save = jest.fn().mockResolvedValue(user);

    const result = await repository.save(user as User);

    // Test only data access concern: database operation
    expect(typeormRepository.save).toHaveBeenCalledWith(user);
    expect(result).toEqual(user);
  });
});

// Test Domain concern (Entity)
describe('Order Entity', () => {
  it('should allow cancellation of pending orders', () => {
    const order = new Order();
    order.status = OrderStatus.PENDING;

    // Test only domain logic concern: business rules
    expect(order.canBeCancelled()).toBe(true);
    
    order.cancel();
    expect(order.status).toBe(OrderStatus.CANCELLED);
  });

  it('should not allow cancellation of shipped orders', () => {
    const order = new Order();
    order.status = OrderStatus.SHIPPED;

    // Test only domain logic concern: business rules
    expect(order.canBeCancelled()).toBe(false);
    expect(() => order.cancel()).toThrow('Order cannot be cancelled');
  });
});
```

---

### **Method 5: Refactoring to SoC**

```typescript
// Before: Mixed concerns (BAD)
@Controller('products')
export class ProductController {
  constructor(@InjectRepository(Product) private repo: Repository<Product>) {}

  @Post()
  async create(@Body() body: any) {
    // Mixed: validation, business logic, database access all in controller
    if (!body.name || body.name.length < 3) {
      throw new Error('Name too short');
    }
    
    if (body.price < 0) {
      throw new Error('Invalid price');
    }

    const discountedPrice = body.price * 0.9;
    
    const product = await this.repo.save({
      name: body.name,
      price: discountedPrice,
    });

    await this.emailAdmin(`New product: ${product.name}`);
    
    return product;
  }
}

// After: Separated concerns (GOOD)

// 1. DTO for validation concern
export class CreateProductDto {
  @IsString()
  @MinLength(3)
  name: string;

  @IsNumber()
  @Min(0)
  price: number;
}

// 2. Controller for HTTP concern
@Controller('products')
export class ProductController {
  constructor(private productService: ProductService) {}

  @Post()
  async create(@Body() dto: CreateProductDto) {
    return this.productService.createProduct(dto);
  }
}

// 3. Service for business logic concern
@Injectable()
export class ProductService {
  constructor(
    private productRepository: ProductRepository,
    private notificationService: NotificationService,
  ) {}

  async createProduct(dto: CreateProductDto): Promise<Product> {
    // Business logic: apply discount
    const discountedPrice = dto.price * 0.9;

    // Save through repository
    const product = await this.productRepository.save({
      name: dto.name,
      price: discountedPrice,
    });

    // Orchestration: notify admin
    await this.notificationService.notifyAdmin(`New product: ${product.name}`);

    return product;
  }
}

// 4. Repository for data access concern
@Injectable()
export class ProductRepository {
  constructor(@InjectRepository(Product) private repo: Repository<Product>) {}

  async save(data: Partial<Product>): Promise<Product> {
    return this.repo.save(data);
  }
}

// 5. NotificationService for email concern
@Injectable()
export class NotificationService {
  async notifyAdmin(message: string): Promise<void> {
    // Email logic here
  }
}
```

---

### **Benefits of SoC:**

```typescript
// ✅ Testability: Test each concern independently
// ✅ Maintainability: Easy to find and fix issues
// ✅ Reusability: Use services in multiple contexts (HTTP, CLI, cron jobs)
// ✅ Scalability: Add new features without affecting existing code
// ✅ Team collaboration: Different developers work on different concerns
// ✅ Flexibility: Change implementation of one concern without affecting others

// Example: Switching database
// Without SoC: Need to change controller, service, and database code
// With SoC: Only change repository implementation

// Before (TypeORM)
@Injectable()
export class UserRepository {
  constructor(@InjectRepository(User) private repo: Repository<User>) {}
  
  async save(user: User): Promise<User> {
    return this.repo.save(user);
  }
}

// After (Mongoose) - Service code unchanged!
@Injectable()
export class UserRepository {
  constructor(@InjectModel(User.name) private model: Model<User>) {}
  
  async save(user: User): Promise<User> {
    return this.model.create(user);
  }
}
```

---

### **Best Practices:**

```typescript
// ✅ GOOD: Each file has one concern
user.controller.ts   // HTTP concern only
user.service.ts      // Business logic concern only
user.repository.ts   // Data access concern only
user.entity.ts       // Domain concern only

// ❌ BAD: One file does everything
user.ts  // HTTP + business logic + database + domain

// ✅ GOOD: Clear naming shows concern
@Controller('users')      // HTTP layer
@Injectable() UserService // Business layer
@Injectable() UserRepository // Data layer

// ❌ BAD: Unclear responsibilities
@Injectable() UserManager  // What does this do?

// ✅ GOOD: Dependency direction follows concerns
Controller → Service → Repository
(HTTP) → (Business) → (Data)

// ❌ BAD: Circular dependencies
Service ↔ Controller

// ✅ GOOD: Single Responsibility Principle
class UserService {
  // Only user business logic
}

class EmailService {
  // Only email sending
}

// ❌ BAD: Multiple responsibilities
class UserService {
  // User logic + email sending + file upload + payment processing
}
```

**Key Takeaway:**

- **Separation of Concerns (SoC)** means dividing code by specific responsibilities:
  - **Controllers** handle HTTP requests/responses
  - **Services** contain business logic and orchestration
  - **Repositories** manage data access
  - **Entities** represent domain models
  - **DTOs** handle validation
  - **Shared services** handle cross-cutting concerns (logging, auth, email)
- Each component has **one clear responsibility**, making the code **testable**, **maintainable**, **reusable**, and **scalable**
- Changes to one concern don't affect others

</details>

<details>
<summary><strong>4. What is Dependency Injection and why is it important?</strong></summary>

**Answer:**

- **Dependency Injection (DI)**: Design pattern where a class receives its dependencies from external sources rather than creating them itself
- **NestJS Implementation**: **IoC container** automatically creates and injects dependencies  
- **Importance**: Promotes **loose coupling**, **testability**, **reusability**, and makes code **easier to maintain** and **change**

---

### **Without DI vs With DI:**

```typescript
// ❌ WITHOUT Dependency Injection (tightly coupled)
export class UserService {
  private userRepository: UserRepository;
  private emailService: EmailService;

  constructor() {
    // Service creates its own dependencies - BAD!
    this.userRepository = new UserRepository();
    this.emailService = new EmailService();
  }

  async createUser(data: any) {
    const user = await this.userRepository.save(data);
    await this.emailService.sendWelcome(user.email);
    return user;
  }
}

// Problems:
// 1. Hard to test (can't mock dependencies)
// 2. Tightly coupled (UserService knows how to create dependencies)
// 3. Hard to change (can't swap implementations)
// 4. Hard to reuse (dependencies are hardcoded)

// Testing is difficult:
const service = new UserService();
// Can't mock UserRepository or EmailService!
// Tests will hit real database and send real emails!

// ✅ WITH Dependency Injection (loosely coupled)
@Injectable()
export class UserService {
  constructor(
    private readonly userRepository: UserRepository,    // Injected
    private readonly emailService: EmailService,        // Injected
  ) {}

  async createUser(data: any) {
    const user = await this.userRepository.save(data);
    await this.emailService.sendWelcome(user.email);
    return user;
  }
}

// Benefits:
// 1. Easy to test (can inject mocks)
// 2. Loosely coupled (UserService doesn't know how dependencies are created)
// 3. Easy to change (swap implementations without changing UserService)
// 4. Reusable (NestJS container manages instances)

// Testing is easy:
const mockRepo = { save: jest.fn() };
const mockEmail = { sendWelcome: jest.fn() };
const service = new UserService(mockRepo, mockEmail);
// Full control over dependencies!
```

---

### **Method 1: Basic Dependency Injection in NestJS**

```typescript
// Step 1: Mark class as injectable
@Injectable()
export class UserRepository {
  async findById(id: string): Promise<User> {
    // Database logic
  }
}

@Injectable()
export class EmailService {
  async sendEmail(to: string, subject: string): Promise<void> {
    // Email logic
  }
}

// Step 2: Inject dependencies via constructor
@Injectable()
export class UserService {
  constructor(
    private readonly userRepository: UserRepository,  // DI here
    private readonly emailService: EmailService,      // DI here
  ) {}

  async createUser(dto: CreateUserDto): Promise<User> {
    const user = await this.userRepository.save(dto);
    await this.emailService.sendEmail(user.email, 'Welcome');
    return user;
  }
}

// Step 3: Controller also uses DI
@Controller('users')
export class UserController {
  constructor(private readonly userService: UserService) {}  // DI here

  @Post()
  async create(@Body() dto: CreateUserDto) {
    return this.userService.createUser(dto);
  }
}

// Step 4: Register providers in module
@Module({
  controllers: [UserController],
  providers: [
    UserService,      // NestJS will create and inject this
    UserRepository,   // NestJS will create and inject this
    EmailService,     // NestJS will create and inject this
  ],
})
export class UserModule {}

// NestJS IoC container automatically:
// 1. Creates instances of all providers
// 2. Resolves dependencies (sees UserService needs UserRepository and EmailService)
// 3. Injects dependencies into constructors
// 4. Manages lifecycle (singleton by default)
```

---

### **Method 2: Different Types of Dependency Injection**

```typescript
// ========== CONSTRUCTOR INJECTION (Recommended) ==========
@Injectable()
export class OrderService {
  constructor(
    private readonly orderRepository: OrderRepository,    // Injected
    private readonly paymentService: PaymentService,      // Injected
  ) {}
  // ✅ Dependencies are immutable (readonly)
  // ✅ Clear what dependencies are needed
  // ✅ Easy to test
}

// ========== PROPERTY INJECTION (Not recommended in NestJS) ==========
// NestJS doesn't support property injection by default
// Use constructor injection instead

// ========== SETTER INJECTION (Rarely used) ==========
// Also not common in NestJS
// Use constructor injection

// ========== METHOD INJECTION (Special cases) ==========
@Injectable()
export class TaskService {
  // Inject per method (useful for REQUEST scoped providers)
  async processTask(@Inject(REQUEST) request: Request) {
    // Use request-specific data
  }
}
```

---

### **Method 3: Interface-based Dependency Injection**

```typescript
// Define interface for dependency
export interface IEmailService {
  sendEmail(to: string, subject: string, body: string): Promise<void>;
}

// Implementation 1: Real email service
@Injectable()
export class SendGridEmailService implements IEmailService {
  async sendEmail(to: string, subject: string, body: string): Promise<void> {
    // SendGrid implementation
    await sendgrid.send({ to, subject, text: body });
  }
}

// Implementation 2: Mock email service
@Injectable()
export class MockEmailService implements IEmailService {
  async sendEmail(to: string, subject: string, body: string): Promise<void> {
    // Mock implementation (just log)
    console.log(`Email to ${to}: ${subject}`);
  }
}

// Use token for injection
export const EMAIL_SERVICE = 'EMAIL_SERVICE';

// Service depends on interface, not implementation
@Injectable()
export class UserService {
  constructor(
    @Inject(EMAIL_SERVICE) private readonly emailService: IEmailService,
  ) {}

  async createUser(dto: CreateUserDto): Promise<User> {
    // Uses interface, doesn't know which implementation
    await this.emailService.sendEmail(dto.email, 'Welcome', 'Welcome!');
  }
}

// Module configuration - swap implementations easily
@Module({
  providers: [
    UserService,
    {
      provide: EMAIL_SERVICE,
      useClass: process.env.NODE_ENV === 'production' 
        ? SendGridEmailService   // Production: real emails
        : MockEmailService,      // Development: mock emails
    },
  ],
})
export class UserModule {}

// Benefits:
// ✅ Swap implementations without changing UserService
// ✅ Use different implementations per environment
// ✅ Easy to test (inject mock implementation)
```

---

### **Method 4: Testing with Dependency Injection**

```typescript
// Service to test
@Injectable()
export class OrderService {
  constructor(
    private readonly orderRepository: OrderRepository,
    private readonly paymentService: PaymentService,
    private readonly emailService: EmailService,
  ) {}

  async createOrder(userId: string, items: any[]): Promise<Order> {
    // Calculate total
    const total = items.reduce((sum, item) => sum + item.price, 0);

    // Save order
    const order = await this.orderRepository.save({ userId, items, total });

    // Process payment
    await this.paymentService.charge(userId, total);

    // Send confirmation
    await this.emailService.sendOrderConfirmation(order.id);

    return order;
  }
}

// Unit test with mocked dependencies
describe('OrderService', () => {
  let service: OrderService;
  let orderRepository: jest.Mocked<OrderRepository>;
  let paymentService: jest.Mocked<PaymentService>;
  let emailService: jest.Mocked<EmailService>;

  beforeEach(async () => {
    // Create mock implementations
    const mockOrderRepository = {
      save: jest.fn(),
      findById: jest.fn(),
    };

    const mockPaymentService = {
      charge: jest.fn(),
    };

    const mockEmailService = {
      sendOrderConfirmation: jest.fn(),
    };

    // Create testing module with mocks
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        OrderService,
        {
          provide: OrderRepository,
          useValue: mockOrderRepository,  // Inject mock
        },
        {
          provide: PaymentService,
          useValue: mockPaymentService,   // Inject mock
        },
        {
          provide: EmailService,
          useValue: mockEmailService,     // Inject mock
        },
      ],
    }).compile();

    service = module.get<OrderService>(OrderService);
    orderRepository = module.get(OrderRepository);
    paymentService = module.get(PaymentService);
    emailService = module.get(EmailService);
  });

  it('should create order and process payment', async () => {
    // Arrange
    const userId = 'user-123';
    const items = [{ price: 10 }, { price: 20 }];
    const savedOrder = { id: 'order-456', userId, items, total: 30 };
    
    orderRepository.save.mockResolvedValue(savedOrder);
    paymentService.charge.mockResolvedValue(undefined);
    emailService.sendOrderConfirmation.mockResolvedValue(undefined);

    // Act
    const result = await service.createOrder(userId, items);

    // Assert
    expect(orderRepository.save).toHaveBeenCalledWith({
      userId,
      items,
      total: 30,
    });
    expect(paymentService.charge).toHaveBeenCalledWith(userId, 30);
    expect(emailService.sendOrderConfirmation).toHaveBeenCalledWith('order-456');
    expect(result).toEqual(savedOrder);
  });

  it('should not send email if payment fails', async () => {
    // Arrange
    const userId = 'user-123';
    const items = [{ price: 10 }];
    
    orderRepository.save.mockResolvedValue({ id: 'order-456' } as Order);
    paymentService.charge.mockRejectedValue(new Error('Payment failed'));

    // Act & Assert
    await expect(service.createOrder(userId, items)).rejects.toThrow('Payment failed');
    expect(emailService.sendOrderConfirmation).not.toHaveBeenCalled();
  });
});

// Without DI, this test would be impossible or require complex mocking!
```

---

### **Method 5: Advanced DI Patterns**

```typescript
// ========== OPTIONAL DEPENDENCIES ==========
@Injectable()
export class UserService {
  constructor(
    private readonly userRepository: UserRepository,
    @Optional() private readonly cacheService?: CacheService,  // Optional
  ) {}

  async findById(id: string): Promise<User> {
    // Use cache if available
    if (this.cacheService) {
      const cached = await this.cacheService.get(`user:${id}`);
      if (cached) return cached;
    }

    const user = await this.userRepository.findById(id);

    if (this.cacheService) {
      await this.cacheService.set(`user:${id}`, user);
    }

    return user;
  }
}

// ========== FACTORY PROVIDERS ==========
@Module({
  providers: [
    {
      provide: 'DATABASE_CONNECTION',
      useFactory: async (configService: ConfigService) => {
        // Create connection based on config
        const connection = await createConnection({
          host: configService.get('DB_HOST'),
          port: configService.get('DB_PORT'),
        });
        return connection;
      },
      inject: [ConfigService],  // Inject ConfigService into factory
    },
  ],
})
export class DatabaseModule {}

// ========== ASYNC PROVIDERS ==========
@Module({
  providers: [
    {
      provide: 'ASYNC_CONNECTION',
      useFactory: async (): Promise<Connection> => {
        // Async initialization
        const connection = await connectToDatabase();
        await connection.migrate();
        return connection;
      },
    },
  ],
})
export class AppModule {}

// ========== MULTIPLE IMPLEMENTATIONS (Array Injection) ==========
// Define strategy interface
export interface PaymentStrategy {
  processPayment(amount: number): Promise<void>;
}

@Injectable()
export class CreditCardStrategy implements PaymentStrategy {
  async processPayment(amount: number): Promise<void> {
    // Credit card payment
  }
}

@Injectable()
export class PayPalStrategy implements PaymentStrategy {
  async processPayment(amount: number): Promise<void> {
    // PayPal payment
  }
}

// Service that uses all strategies
export const PAYMENT_STRATEGIES = 'PAYMENT_STRATEGIES';

@Injectable()
export class PaymentService {
  constructor(
    @Inject(PAYMENT_STRATEGIES) private strategies: PaymentStrategy[],
  ) {}

  async pay(method: string, amount: number): Promise<void> {
    const strategy = this.strategies.find(s => s.constructor.name.includes(method));
    if (!strategy) throw new Error('Payment method not found');
    await strategy.processPayment(amount);
  }
}

// Module registration
@Module({
  providers: [
    CreditCardStrategy,
    PayPalStrategy,
    {
      provide: PAYMENT_STRATEGIES,
      useFactory: (cc: CreditCardStrategy, pp: PayPalStrategy) => [cc, pp],
      inject: [CreditCardStrategy, PayPalStrategy],
    },
    PaymentService,
  ],
})
export class PaymentModule {}
```

---

### **Why DI is Important:**

```typescript
// 1. TESTABILITY
// Without DI: Hard to test
class UserService {
  constructor() {
    this.repo = new UserRepository();  // Can't mock!
  }
}

// With DI: Easy to test
class UserService {
  constructor(private repo: UserRepository) {}  // Can inject mock!
}

// 2. LOOSE COUPLING
// Without DI: Tightly coupled
class OrderService {
  private paymentService = new StripePaymentService();  // Hardcoded!
  // If we want to use PayPal, we need to change OrderService
}

// With DI: Loosely coupled
class OrderService {
  constructor(private paymentService: IPaymentService) {}
  // Can inject StripePaymentService or PayPalPaymentService without changing OrderService
}

// 3. REUSABILITY
// Without DI: Can't reuse
class EmailService {
  private config = { host: 'smtp.gmail.com' };  // Hardcoded!
}

// With DI: Highly reusable
class EmailService {
  constructor(private config: EmailConfig) {}  // Injected config
}

// 4. FLEXIBILITY
// Without DI: Hard to change
class CacheService {
  private redis = new Redis('localhost:6379');  // Hardcoded!
}

// With DI: Easy to change
class CacheService {
  constructor(@Inject('CACHE_CLIENT') private client: any) {}
  // Can inject Redis, Memcached, or in-memory cache
}

// 5. LIFECYCLE MANAGEMENT
// Without DI: Manual lifecycle management
const instance1 = new UserService(new UserRepository());
const instance2 = new UserService(new UserRepository());
// Created two instances and two repository instances

// With DI: Automatic lifecycle management
@Injectable()  // Singleton by default
class UserService {
  constructor(private repo: UserRepository) {}
}
// NestJS creates one instance and reuses it
```

---

### **Best Practices:**

```typescript
// ✅ GOOD: Use constructor injection
@Injectable()
export class UserService {
  constructor(private readonly userRepository: UserRepository) {}
}

// ❌ BAD: Create dependencies manually
@Injectable()
export class UserService {
  private userRepository = new UserRepository();
}

// ✅ GOOD: Depend on abstractions (interfaces)
constructor(@Inject('IEmailService') private emailService: IEmailService) {}

// ❌ BAD: Depend on concrete implementations
constructor(private emailService: SendGridEmailService) {}

// ✅ GOOD: Use readonly for injected dependencies
constructor(private readonly userService: UserService) {}

// ❌ BAD: Mutable dependencies
constructor(private userService: UserService) {}
// Someone could do: this.userService = null;

// ✅ GOOD: Keep dependencies minimal
@Injectable()
export class UserService {
  constructor(
    private userRepository: UserRepository,
    private emailService: EmailService,
  ) {}
}

// ❌ BAD: Too many dependencies (code smell)
@Injectable()
export class UserService {
  constructor(
    private dep1: any, private dep2: any, private dep3: any,
    private dep4: any, private dep5: any, private dep6: any,
    private dep7: any, private dep8: any,  // Too many!
  ) {}
}
// If you need 8+ dependencies, consider refactoring

// ✅ GOOD: Use tokens for interface injection
export const EMAIL_SERVICE = 'EMAIL_SERVICE';
constructor(@Inject(EMAIL_SERVICE) private emailService: IEmailService) {}

// ✅ GOOD: Test with injected mocks
const mockRepo = { save: jest.fn() };
const service = new UserService(mockRepo);
```

**Key Takeaway:**

- **Dependency Injection (DI)**: Pattern where classes receive dependencies from external sources (NestJS IoC container) rather than creating them
- **Benefits**: Enables **loose coupling**, **testability**, **reusability**, **flexibility**, and **maintainability**
- **Best Practices**: Use **constructor injection** with **readonly** modifiers, depend on **abstractions** (interfaces) not concrete classes, leverage NestJS's IoC container for automatic dependency resolution and lifecycle management

</details>

## Modular Architecture

<details>
<summary><strong>5. What is a Module in NestJS?</strong></summary>

**Answer:**

- **Module Definition**: Class decorated with `@Module()` that organizes application structure by grouping related components (controllers, services, providers)
- **Metadata**: Uses metadata to define **providers** (services to instantiate), **controllers** (routes to register), **imports** (other modules to use), and **exports** (providers to share)
- **Benefits**: Enables **encapsulation**, **reusability**, and **lazy loading**

---

### **Module Anatomy:**

```typescript
import { Module } from '@nestjs/common';
import { UserController } from './user.controller';
import { UserService } from './user.service';
import { UserRepository } from './user.repository';
import { TypeOrmModule } from '@nestjs/typeorm';
import { User } from './entities/user.entity';

@Module({
  imports: [
    // Import other modules to use their exported providers
    TypeOrmModule.forFeature([User]),
    // Can import SharedModule, AuthModule, etc.
  ],
  controllers: [
    // HTTP controllers in this module
    UserController,
  ],
  providers: [
    // Services, repositories, and other providers
    UserService,
    UserRepository,
  ],
  exports: [
    // Export providers for other modules to use
    UserService,
  ],
})
export class UserModule {}

// Module metadata properties:
// - imports: Other modules whose exports this module needs
// - controllers: Controllers belonging to this module
// - providers: Injectable providers (services, repositories, factories, etc.)
// - exports: Subset of providers to make available to other modules
```

---

### **Method 1: Basic Feature Module**

```typescript
// ========== USER MODULE ==========
// user/user.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UserController } from './controllers/user.controller';
import { UserService } from './services/user.service';
import { UserRepository } from './repositories/user.repository';
import { User } from './entities/user.entity';

@Module({
  imports: [
    // Import TypeOrmModule to use User entity
    TypeOrmModule.forFeature([User]),
  ],
  controllers: [
    // Register controllers
    UserController,
  ],
  providers: [
    // Register providers (services, repositories)
    UserService,
    UserRepository,
  ],
  exports: [
    // Export UserService so other modules can use it
    UserService,
  ],
})
export class UserModule {}

// user/controllers/user.controller.ts
@Controller('users')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get()
  async findAll() {
    return this.userService.findAll();
  }

  @Post()
  async create(@Body() dto: CreateUserDto) {
    return this.userService.create(dto);
  }
}

// user/services/user.service.ts
@Injectable()
export class UserService {
  constructor(private readonly userRepository: UserRepository) {}

  async findAll(): Promise<User[]> {
    return this.userRepository.findAll();
  }

  async create(dto: CreateUserDto): Promise<User> {
    return this.userRepository.save(dto);
  }
}

// user/repositories/user.repository.ts
@Injectable()
export class UserRepository {
  constructor(
    @InjectRepository(User)
    private readonly repository: Repository<User>,
  ) {}

  async findAll(): Promise<User[]> {
    return this.repository.find();
  }

  async save(data: Partial<User>): Promise<User> {
    return this.repository.save(data);
  }
}

// ========== APP MODULE (Root Module) ==========
// app.module.ts
import { Module } from '@nestjs/common';
import { UserModule } from './user/user.module';

@Module({
  imports: [
    UserModule,  // Import feature module
  ],
})
export class AppModule {}

// Module benefits:
// ✅ Encapsulation: User-related code is grouped together
// ✅ Reusability: UserModule can be imported by other modules
// ✅ Maintainability: Easy to find user-related code
// ✅ Testability: Can test UserModule independently
```

---

### **Method 2: Module with Dependencies (Imports)**

```typescript
// ========== ORDER MODULE DEPENDS ON USER MODULE ==========
// order/order.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { OrderController } from './controllers/order.controller';
import { OrderService } from './services/order.service';
import { Order } from './entities/order.entity';
import { UserModule } from '../user/user.module';  // Import UserModule

@Module({
  imports: [
    TypeOrmModule.forFeature([Order]),
    UserModule,  // Import UserModule to use UserService
  ],
  controllers: [OrderController],
  providers: [OrderService],
  exports: [OrderService],
})
export class OrderModule {}

// order/services/order.service.ts
@Injectable()
export class OrderService {
  constructor(
    private readonly orderRepository: OrderRepository,
    private readonly userService: UserService,  // From UserModule
  ) {}

  async createOrder(userId: string, items: any[]): Promise<Order> {
    // Can use UserService because UserModule exports it
    const user = await this.userService.findById(userId);
    
    if (!user) {
      throw new NotFoundException('User not found');
    }

    const order = await this.orderRepository.save({
      userId,
      items,
      total: this.calculateTotal(items),
    });

    return order;
  }

  private calculateTotal(items: any[]): number {
    return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  }
}

// Dependency flow:
// OrderModule imports UserModule
// OrderService uses UserService (exported by UserModule)
// NestJS IoC container resolves dependencies automatically
```

---

### **Method 3: Shared Module (Used by Multiple Modules)**

```typescript
// ========== SHARED MODULE ==========
// shared/shared.module.ts
import { Module, Global } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { LoggerService } from './services/logger.service';
import { EmailService } from './services/email.service';
import { CacheService } from './services/cache.service';

@Global()  // Makes module globally available (optional)
@Module({
  imports: [ConfigModule],
  providers: [
    LoggerService,
    EmailService,
    CacheService,
  ],
  exports: [
    // Export all shared services
    LoggerService,
    EmailService,
    CacheService,
  ],
})
export class SharedModule {}

// shared/services/logger.service.ts
@Injectable()
export class LoggerService {
  log(message: string, context?: string): void {
    console.log(`[${context || 'App'}] ${message}`);
  }

  error(message: string, trace?: string, context?: string): void {
    console.error(`[${context || 'App'}] ${message}`, trace);
  }
}

// ========== FEATURE MODULES USE SHARED MODULE ==========
// user/user.module.ts
@Module({
  imports: [SharedModule],  // Import shared services
  controllers: [UserController],
  providers: [UserService],
})
export class UserModule {}

// user/services/user.service.ts
@Injectable()
export class UserService {
  constructor(
    private readonly userRepository: UserRepository,
    private readonly logger: LoggerService,      // From SharedModule
    private readonly emailService: EmailService,  // From SharedModule
  ) {}

  async createUser(dto: CreateUserDto): Promise<User> {
    this.logger.log('Creating user', 'UserService');
    
    const user = await this.userRepository.save(dto);
    
    await this.emailService.sendWelcomeEmail(user.email);
    
    this.logger.log(`User created: ${user.id}`, 'UserService');
    
    return user;
  }
}

// ========== APP MODULE ==========
@Module({
  imports: [
    SharedModule,  // Import once in root module
    UserModule,
    OrderModule,
    ProductModule,
  ],
})
export class AppModule {}

// Benefits:
// ✅ Avoid duplication: Logger, Email, Cache used by all modules
// ✅ Single source of truth: Shared config in one place
// ✅ Easy updates: Change shared service in one place
```

---

### **Method 4: Dynamic Modules (Configurable Modules)**

```typescript
// ========== DYNAMIC MODULE WITH CONFIGURATION ==========
// database/database.module.ts
import { Module, DynamicModule } from '@nestjs/common';
import { DatabaseService } from './database.service';

export interface DatabaseConfig {
  host: string;
  port: number;
  username: string;
  password: string;
  database: string;
}

@Module({})
export class DatabaseModule {
  // Static method to create configured module
  static forRoot(config: DatabaseConfig): DynamicModule {
    return {
      module: DatabaseModule,
      providers: [
        {
          provide: 'DATABASE_CONFIG',
          useValue: config,
        },
        DatabaseService,
      ],
      exports: [DatabaseService],
      global: true,  // Make globally available
    };
  }

  // Async configuration
  static forRootAsync(options: {
    useFactory: (...args: any[]) => Promise<DatabaseConfig> | DatabaseConfig;
    inject?: any[];
  }): DynamicModule {
    return {
      module: DatabaseModule,
      imports: options.imports || [],
      providers: [
        {
          provide: 'DATABASE_CONFIG',
          useFactory: options.useFactory,
          inject: options.inject || [],
        },
        DatabaseService,
      ],
      exports: [DatabaseService],
      global: true,
    };
  }
}

// database/database.service.ts
@Injectable()
export class DatabaseService {
  constructor(@Inject('DATABASE_CONFIG') private config: DatabaseConfig) {}

  async connect(): Promise<void> {
    console.log(`Connecting to ${this.config.host}:${this.config.port}`);
    // Connection logic
  }
}

// ========== USAGE ==========
// app.module.ts
@Module({
  imports: [
    // Static configuration
    DatabaseModule.forRoot({
      host: 'localhost',
      port: 5432,
      username: 'admin',
      password: 'secret',
      database: 'mydb',
    }),

    // OR Async configuration with ConfigService
    DatabaseModule.forRootAsync({
      useFactory: (configService: ConfigService) => ({
        host: configService.get('DB_HOST'),
        port: configService.get('DB_PORT'),
        username: configService.get('DB_USERNAME'),
        password: configService.get('DB_PASSWORD'),
        database: configService.get('DB_NAME'),
      }),
      inject: [ConfigService],
    }),

    UserModule,
  ],
})
export class AppModule {}

// Real-world examples:
// - TypeOrmModule.forRoot(config)
// - ConfigModule.forRoot()
// - JwtModule.register(config)
```

---

### **Method 5: Module Re-exporting**

```typescript
// ========== CORE MODULE (Re-exports multiple modules) ==========
// core/core.module.ts
import { Module } from '@nestjs/common';
import { DatabaseModule } from '../database/database.module';
import { ConfigModule } from '../config/config.module';
import { LoggerModule } from '../logger/logger.module';
import { CacheModule } from '../cache/cache.module';

@Module({
  imports: [
    DatabaseModule,
    ConfigModule,
    LoggerModule,
    CacheModule,
  ],
  exports: [
    // Re-export imported modules
    DatabaseModule,
    ConfigModule,
    LoggerModule,
    CacheModule,
  ],
})
export class CoreModule {}

// ========== FEATURE MODULES IMPORT CORE MODULE ==========
// user/user.module.ts
@Module({
  imports: [
    CoreModule,  // Gets DatabaseModule, ConfigModule, LoggerModule, CacheModule
  ],
  controllers: [UserController],
  providers: [UserService],
})
export class UserModule {}

// user/services/user.service.ts
@Injectable()
export class UserService {
  constructor(
    private readonly databaseService: DatabaseService,  // From CoreModule
    private readonly configService: ConfigService,      // From CoreModule
    private readonly logger: LoggerService,             // From CoreModule
    private readonly cache: CacheService,               // From CoreModule
  ) {}
}

// Benefits:
// ✅ Single import: Import CoreModule instead of 4 separate modules
// ✅ Consistency: All features get same core services
// ✅ Easy to update: Add new core service in one place
```

---

### **Method 6: Lazy Loading Modules**

```typescript
// ========== LAZY LOADED ADMIN MODULE ==========
// admin/admin.module.ts
@Module({
  imports: [
    RouterModule.register([
      {
        path: 'admin',
        module: AdminModule,
      },
    ]),
  ],
  controllers: [AdminController],
  providers: [AdminService],
})
export class AdminModule {}

// ========== APP MODULE WITH LAZY LOADING ==========
// app.module.ts
import { Module } from '@nestjs/common';
import { RouterModule } from '@nestjs/core';

@Module({
  imports: [
    UserModule,
    OrderModule,
    // AdminModule is NOT imported here - loaded lazily
  ],
})
export class AppModule {}

// Lazy loading configuration
const routes: Routes = [
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule),
  },
];

// Benefits:
// ✅ Faster startup: Don't load admin module until accessed
// ✅ Reduced memory: Admin code not in memory until needed
// ✅ Code splitting: Separate bundle for admin features
```

---

### **Module Comparison:**

| Feature | Feature Module | Shared Module | Dynamic Module | Global Module |
|---------|---------------|---------------|----------------|---------------|
| Purpose | Organize related features | Provide common services | Configurable functionality | Available everywhere |
| Example | UserModule, OrderModule | LoggerModule, CacheModule | DatabaseModule.forRoot() | ConfigModule |
| Decorator | `@Module()` | `@Module()` | `@Module()` + static method | `@Global() @Module()` |
| Import | By other modules | By all modules | With configuration | Automatically available |
| Exports | Feature-specific providers | Common utilities | Configured providers | All providers |

---

### **Best Practices:**

```typescript
// ✅ GOOD: One module per feature
UserModule      // All user-related code
OrderModule     // All order-related code
ProductModule   // All product-related code

// ❌ BAD: Everything in one module
AppModule       // 50 controllers, 100 services - too big!

// ✅ GOOD: Export only what's needed
@Module({
  providers: [UserService, UserRepository, UserHelper],
  exports: [UserService],  // Only export public API
})
export class UserModule {}

// ❌ BAD: Export everything
@Module({
  providers: [UserService, UserRepository, UserHelper],
  exports: [UserService, UserRepository, UserHelper],  // Exposes internals!
})

// ✅ GOOD: Use @Global() sparingly
@Global()
@Module({
  providers: [ConfigService, LoggerService],  // Truly global services
})
export class CoreModule {}

// ❌ BAD: Overuse @Global()
@Global()
@Module({
  providers: [UserService],  // Feature-specific service shouldn't be global
})

// ✅ GOOD: Clear module dependencies
@Module({
  imports: [UserModule, ProductModule],  // Clear what this module needs
})
export class OrderModule {}

// ❌ BAD: Hidden dependencies
// OrderService uses UserService but OrderModule doesn't import UserModule

// ✅ GOOD: Organize modules by feature
src/
├── user/
│   ├── user.module.ts
│   ├── controllers/
│   ├── services/
│   └── entities/
├── order/
│   ├── order.module.ts
│   ├── controllers/
│   ├── services/
│   └── entities/
└── product/
    ├── product.module.ts
    ├── controllers/
    ├── services/
    └── entities/

// ❌ BAD: Organize modules by layer
src/
├── controllers/
│   ├── user.controller.ts
│   ├── order.controller.ts
│   └── product.controller.ts
├── services/
│   ├── user.service.ts
│   ├── order.service.ts
│   └── product.service.ts
└── all-features.module.ts  // One huge module
```

**Key Takeaway:**

- **Module Definition**: Class with `@Module()` decorator that groups related components (controllers, services, providers)
- **Metadata Properties**: Define **imports** (dependencies from other modules), **providers** (services to instantiate), **controllers** (HTTP handlers), and **exports** (providers to share)
- **Benefits**: Enable **encapsulation** (group related code), **reusability** (import modules), **dependency management** (clear imports), and **lazy loading**
- **Best Practices**: Use **feature modules** for domains, **shared modules** for common utilities, **dynamic modules** for configuration, and **@Global()** sparingly for truly global services

</details>

<details>
<summary><strong>6. How do you organize code into modules?</strong></summary>

**Answer:**

Organize code into modules using **feature-based modularization**:

- **Purpose**: Each module represents a **business domain** or **bounded context**
- **Structure**: One module per feature (e.g., UserModule, OrderModule, ProductModule)
- **Shared modules**: Place cross-cutting concerns in shared modules (e.g., AuthModule, LoggerModule, DatabaseModule)
- **Core module**: Keep application-wide infrastructure (config, database, cache) in a core module
- **Guidelines**: Use clear naming, enforce encapsulation, and follow the Single Responsibility Principle

---

### **Module Organization Strategies:**

```
Strategy 1: Feature-based (Recommended)
==========================================
src/
├── app.module.ts                    # Root module
├── users/                           # User feature
│   ├── users.module.ts
│   ├── controllers/
│   │   └── users.controller.ts
│   ├── services/
│   │   └── users.service.ts
│   ├── repositories/
│   │   └── users.repository.ts
│   ├── entities/
│   │   └── user.entity.ts
│   └── dto/
│       ├── create-user.dto.ts
│       └── update-user.dto.ts
├── orders/                          # Order feature
│   ├── orders.module.ts
│   ├── controllers/
│   ├── services/
│   ├── repositories/
│   ├── entities/
│   └── dto/
├── products/                        # Product feature
│   ├── products.module.ts
│   ├── controllers/
│   ├── services/
│   ├── repositories/
│   ├── entities/
│   └── dto/
├── shared/                          # Shared utilities
│   ├── shared.module.ts
│   ├── services/
│   │   ├── logger.service.ts
│   │   ├── email.service.ts
│   │   └── cache.service.ts
│   └── guards/
│       └── auth.guard.ts
└── core/                            # Core/infrastructure
    ├── core.module.ts
    ├── database/
    │   └── database.module.ts
    └── config/
        └── config.module.ts

Benefits:
✅ High cohesion: Related code stays together
✅ Easy to find: All user code in users/ folder
✅ Easy to test: Test entire feature in isolation
✅ Easy to scale: Add new features without touching existing ones
✅ Clear boundaries: Each feature is self-contained
```

---

### **Method 1: Feature-based Organization**

```typescript
// ========== ROOT MODULE ==========
// app.module.ts
import { Module } from '@nestjs/common';
import { CoreModule } from './core/core.module';
import { SharedModule } from './shared/shared.module';
import { UsersModule } from './users/users.module';
import { OrdersModule } from './orders/orders.module';
import { ProductsModule } from './products/products.module';
import { PaymentsModule } from './payments/payments.module';

@Module({
  imports: [
    // Core infrastructure modules
    CoreModule,
    
    // Shared utilities
    SharedModule,
    
    // Feature modules
    UsersModule,
    OrdersModule,
    ProductsModule,
    PaymentsModule,
  ],
})
export class AppModule {}

// ========== FEATURE MODULE: USERS ==========
// users/users.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UsersController } from './controllers/users.controller';
import { UsersService } from './services/users.service';
import { UsersRepository } from './repositories/users.repository';
import { User } from './entities/user.entity';
import { SharedModule } from '../shared/shared.module';

@Module({
  imports: [
    TypeOrmModule.forFeature([User]),
    SharedModule,  // For logger, email, etc.
  ],
  controllers: [UsersController],
  providers: [UsersService, UsersRepository],
  exports: [UsersService],  // Export for other modules
})
export class UsersModule {}

// ========== FEATURE MODULE: ORDERS ==========
// orders/orders.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { OrdersController } from './controllers/orders.controller';
import { OrdersService } from './services/orders.service';
import { OrdersRepository } from './repositories/orders.repository';
import { Order } from './entities/order.entity';
import { UsersModule } from '../users/users.module';  // Depends on Users
import { ProductsModule } from '../products/products.module';  // Depends on Products
import { PaymentsModule } from '../payments/payments.module';  // Depends on Payments

@Module({
  imports: [
    TypeOrmModule.forFeature([Order]),
    UsersModule,      // Import to use UsersService
    ProductsModule,   // Import to use ProductsService
    PaymentsModule,   // Import to use PaymentsService
  ],
  controllers: [OrdersController],
  providers: [OrdersService, OrdersRepository],
  exports: [OrdersService],
})
export class OrdersModule {}

// orders/services/orders.service.ts
@Injectable()
export class OrdersService {
  constructor(
    private readonly ordersRepository: OrdersRepository,
    private readonly usersService: UsersService,        // From UsersModule
    private readonly productsService: ProductsService,  // From ProductsModule
    private readonly paymentsService: PaymentsService,  // From PaymentsModule
    private readonly logger: LoggerService,             // From SharedModule
  ) {}

  async createOrder(userId: string, productIds: string[]): Promise<Order> {
    // Validate user exists
    const user = await this.usersService.findById(userId);
    if (!user) {
      throw new NotFoundException('User not found');
    }

    // Validate products exist
    const products = await this.productsService.findByIds(productIds);
    if (products.length !== productIds.length) {
      throw new BadRequestException('Some products not found');
    }

    // Calculate total
    const total = products.reduce((sum, p) => sum + p.price, 0);

    // Create order
    const order = await this.ordersRepository.create({
      userId,
      productIds,
      total,
      status: 'pending',
    });

    // Process payment
    await this.paymentsService.processPayment(order.id, total);

    // Log
    this.logger.log(`Order created: ${order.id}`, 'OrdersService');

    return order;
  }
}

// Module dependency graph:
// OrdersModule → UsersModule, ProductsModule, PaymentsModule, SharedModule
// UsersModule → SharedModule
// ProductsModule → SharedModule
// PaymentsModule → SharedModule
```

---

### **Method 2: Shared Module for Cross-cutting Concerns**

```typescript
// ========== SHARED MODULE ==========
// shared/shared.module.ts
import { Module, Global } from '@nestjs/common';
import { LoggerService } from './services/logger.service';
import { EmailService } from './services/email.service';
import { CacheService } from './services/cache.service';
import { ValidationService } from './services/validation.service';
import { JwtAuthGuard } from './guards/jwt-auth.guard';
import { RolesGuard } from './guards/roles.guard';
import { LoggingInterceptor } from './interceptors/logging.interceptor';
import { TransformInterceptor } from './interceptors/transform.interceptor';

@Global()  // Make globally available (optional but convenient)
@Module({
  providers: [
    // Services
    LoggerService,
    EmailService,
    CacheService,
    ValidationService,
    
    // Guards
    JwtAuthGuard,
    RolesGuard,
    
    // Interceptors
    LoggingInterceptor,
    TransformInterceptor,
  ],
  exports: [
    // Export all shared utilities
    LoggerService,
    EmailService,
    CacheService,
    ValidationService,
    JwtAuthGuard,
    RolesGuard,
    LoggingInterceptor,
    TransformInterceptor,
  ],
})
export class SharedModule {}

// shared/services/logger.service.ts
@Injectable()
export class LoggerService {
  log(message: string, context?: string): void {
    console.log(`[${context || 'App'}] ${message}`);
  }

  error(message: string, trace?: string, context?: string): void {
    console.error(`[${context || 'App'}] ${message}`, trace);
  }

  warn(message: string, context?: string): void {
    console.warn(`[${context || 'App'}] ${message}`);
  }
}

// shared/services/email.service.ts
@Injectable()
export class EmailService {
  async send(to: string, subject: string, body: string): Promise<void> {
    // Email sending logic
  }

  async sendWelcomeEmail(email: string): Promise<void> {
    await this.send(email, 'Welcome!', 'Welcome to our platform');
  }

  async sendOrderConfirmation(email: string, orderId: string): Promise<void> {
    await this.send(email, 'Order Confirmation', `Order ${orderId} confirmed`);
  }
}

// Usage in any module:
@Module({
  imports: [SharedModule],  // Or no import if @Global()
  providers: [SomeService],
})
export class SomeModule {}

@Injectable()
export class SomeService {
  constructor(
    private readonly logger: LoggerService,  // From SharedModule
    private readonly email: EmailService,    // From SharedModule
  ) {}
}
```

---

### **Method 3: Core Module for Infrastructure**

```typescript
// ========== CORE MODULE ==========
// core/core.module.ts
import { Module, Global } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { TypeOrmModule } from '@nestjs/typeorm';
import { DatabaseModule } from './database/database.module';
import { RedisModule } from './redis/redis.module';
import { EventModule } from './events/event.module';

@Global()
@Module({
  imports: [
    // Configuration
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: '.env',
    }),
    
    // Database
    TypeOrmModule.forRootAsync({
      useFactory: (configService: ConfigService) => ({
        type: 'postgres',
        host: configService.get('DB_HOST'),
        port: configService.get('DB_PORT'),
        username: configService.get('DB_USERNAME'),
        password: configService.get('DB_PASSWORD'),
        database: configService.get('DB_NAME'),
        autoLoadEntities: true,
        synchronize: process.env.NODE_ENV === 'development',
      }),
      inject: [ConfigService],
    }),
    
    // Redis cache
    RedisModule.forRootAsync({
      useFactory: (configService: ConfigService) => ({
        host: configService.get('REDIS_HOST'),
        port: configService.get('REDIS_PORT'),
      }),
      inject: [ConfigService],
    }),
    
    // Event system
    EventModule,
  ],
  exports: [
    ConfigModule,
    DatabaseModule,
    RedisModule,
    EventModule,
  ],
})
export class CoreModule {}

// Core module provides:
// ✅ Configuration (environment variables)
// ✅ Database connection
// ✅ Cache connection
// ✅ Event system
// ✅ Other infrastructure concerns

// Feature modules don't need to worry about infrastructure:
@Module({
  imports: [
    CoreModule,  // Gets all infrastructure
  ],
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}
```

---

### **Method 4: Domain-Driven Design (DDD) Organization**

```typescript
// ========== DDD-STYLE ORGANIZATION ==========
src/
├── app.module.ts
├── core/                           # Core/Infrastructure layer
│   ├── database/
│   ├── config/
│   └── events/
├── shared/                         # Shared kernel
│   ├── domain/
│   │   ├── value-objects/
│   │   └── base-entity.ts
│   ├── application/
│   │   └── base.service.ts
│   └── infrastructure/
│       └── base.repository.ts
└── modules/                        # Bounded contexts
    ├── users/                      # User bounded context
    │   ├── users.module.ts
    │   ├── domain/                 # Domain layer
    │   │   ├── entities/
    │   │   │   └── user.entity.ts
    │   │   ├── value-objects/
    │   │   │   └── email.vo.ts
    │   │   └── repositories/
    │   │       └── user.repository.interface.ts
    │   ├── application/            # Application layer
    │   │   ├── services/
    │   │   │   └── users.service.ts
    │   │   ├── dto/
    │   │   └── use-cases/
    │   │       ├── create-user.use-case.ts
    │   │       └── update-user.use-case.ts
    │   ├── infrastructure/         # Infrastructure layer
    │   │   └── repositories/
    │   │       └── user.repository.impl.ts
    │   └── presentation/           # Presentation layer
    │       └── controllers/
    │           └── users.controller.ts
    ├── orders/                     # Order bounded context
    │   ├── orders.module.ts
    │   ├── domain/
    │   ├── application/
    │   ├── infrastructure/
    │   └── presentation/
    └── products/                   # Product bounded context
        ├── products.module.ts
        ├── domain/
        ├── application/
        ├── infrastructure/
        └── presentation/

// users/users.module.ts
@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UsersController],  // Presentation
  providers: [
    UsersService,                  // Application
    CreateUserUseCase,             // Application
    UpdateUserUseCase,             // Application
    {
      provide: 'IUserRepository',  // Domain interface
      useClass: UserRepositoryImpl, // Infrastructure implementation
    },
  ],
  exports: [UsersService],
})
export class UsersModule {}
```

---

### **Method 5: Monorepo Organization (Multiple Apps)**

```typescript
// ========== MONOREPO STRUCTURE ==========
apps/
├── api/                           # Main API application
│   ├── src/
│   │   ├── app.module.ts
│   │   ├── users/
│   │   ├── orders/
│   │   └── products/
│   └── main.ts
├── admin/                         # Admin application
│   ├── src/
│   │   ├── app.module.ts
│   │   ├── dashboard/
│   │   └── reports/
│   └── main.ts
├── worker/                        # Background worker
│   ├── src/
│   │   ├── app.module.ts
│   │   ├── jobs/
│   │   └── processors/
│   └── main.ts
└── websocket/                     # WebSocket server
    ├── src/
    │   ├── app.module.ts
    │   ├── chat/
    │   └── notifications/
    └── main.ts

libs/                              # Shared libraries
├── common/                        # Common utilities
│   ├── src/
│   │   ├── decorators/
│   │   ├── pipes/
│   │   └── guards/
│   └── index.ts
├── database/                      # Database entities
│   ├── src/
│   │   ├── entities/
│   │   └── repositories/
│   └── index.ts
└── events/                        # Event definitions
    ├── src/
    │   └── events/
    └── index.ts

// api/src/app.module.ts
import { Module } from '@nestjs/common';
import { CommonModule } from '@app/common';      // Shared lib
import { DatabaseModule } from '@app/database';  // Shared lib
import { UsersModule } from './users/users.module';
import { OrdersModule } from './orders/orders.module';

@Module({
  imports: [
    CommonModule,
    DatabaseModule,
    UsersModule,
    OrdersModule,
  ],
})
export class AppModule {}

// admin/src/app.module.ts
import { Module } from '@nestjs/common';
import { CommonModule } from '@app/common';      // Same shared lib
import { DatabaseModule } from '@app/database';  // Same shared lib
import { DashboardModule } from './dashboard/dashboard.module';
import { ReportsModule } from './reports/reports.module';

@Module({
  imports: [
    CommonModule,
    DatabaseModule,
    DashboardModule,
    ReportsModule,
  ],
})
export class AppModule {}

// Benefits:
// ✅ Code reuse: Shared libs used by multiple apps
// ✅ Separation: Different apps for different concerns
// ✅ Scalability: Scale apps independently
// ✅ Team autonomy: Different teams own different apps
```

---

### **Module Organization Patterns:**

| Pattern | Structure | Use Case | Example |
|---------|-----------|----------|---------|
| **Feature-based** | One module per feature | Small to medium apps | UserModule, OrderModule |
| **Layer-based** | Modules by layer | Legacy apps | ControllersModule, ServicesModule |
| **DDD-based** | Modules by bounded context | Complex domains | UserContextModule, OrderContextModule |
| **Monorepo** | Multiple apps + shared libs | Microservices, multiple frontends | API app, Admin app, Worker app |

---

### **Best Practices:**

```typescript
// ✅ GOOD: Feature-based organization
src/
├── users/users.module.ts
├── orders/orders.module.ts
└── products/products.module.ts

// ❌ BAD: Layer-based organization
src/
├── controllers.module.ts
├── services.module.ts
└── repositories.module.ts

// ✅ GOOD: Clear module boundaries
@Module({
  imports: [SharedModule],
  controllers: [UsersController],
  providers: [UsersService, UsersRepository],
  exports: [UsersService],  // Only export public API
})
export class UsersModule {}

// ❌ BAD: Circular dependencies
// UsersModule imports OrdersModule
// OrdersModule imports UsersModule
// Solution: Create a SharedModule or refactor

// ✅ GOOD: Single Responsibility per module
@Module({
  // Only user-related code
})
export class UsersModule {}

// ❌ BAD: Module doing too much
@Module({
  // User + Order + Product + Payment code all in one module
})
export class EverythingModule {}

// ✅ GOOD: Explicit dependencies
@Module({
  imports: [UsersModule, ProductsModule],  // Clear what we need
})
export class OrdersModule {}

// ❌ BAD: Hidden dependencies
// OrdersService uses UsersService but doesn't import UsersModule

// ✅ GOOD: Consistent naming
users/users.module.ts
orders/orders.module.ts
products/products.module.ts

// ❌ BAD: Inconsistent naming
user/user-module.ts
order/orders.mod.ts
product/prod.module.ts
```

**Key Takeaway:**

- **Modularization Approach:** Use **feature-based modularization** — one module per business domain (e.g., `UserModule`, `OrderModule`)
- **Shared & Core Modules:** Put cross-cutting concerns in **shared modules** (e.g., LoggerService, EmailService) and infrastructure in a **core module** (database, config, cache)
- **Module Structure:** Each feature module should contain `controllers/` (HTTP), `services/` (business logic), `repositories/` (data access), `entities/` (domain models), and `dto/` (validation)
- **Guidelines:** Use clear imports to express dependencies, export only public APIs, avoid circular dependencies, and follow the Single Responsibility Principle
- **Scale Options:** For large systems, consider **DDD-style** bounded contexts or a **monorepo** (multiple apps + shared libs)

</details>

<details>
<summary><strong>7. What is feature-based modularization?</strong></summary>

**Answer:**

**Feature-based modularization**: organize code by **business features** or **domains** (instead of technical layers).

- **Structure:** each feature/module groups its controllers, services, repositories, entities, and DTOs (e.g., `UserModule` → user controller, user service, user repository).
- **Benefits:** **high cohesion**, **low coupling**, easier to locate code, supports independent deployment, and enables team autonomy.
- **When to use:** preferred for most applications — simplifies testing, scaling, and later refactor to microservices.

---

### **Feature-based vs Layer-based:**

```typescript
// ❌ LAYER-BASED (Technical layers - NOT recommended)
src/
├── controllers/
│   ├── user.controller.ts
│   ├── order.controller.ts
│   ├── product.controller.ts
│   └── payment.controller.ts
├── services/
│   ├── user.service.ts
│   ├── order.service.ts
│   ├── product.service.ts
│   └── payment.service.ts
├── repositories/
│   ├── user.repository.ts
│   ├── order.repository.ts
│   ├── product.repository.ts
│   └── payment.repository.ts
└── entities/
    ├── user.entity.ts
    ├── order.entity.ts
    ├── product.entity.ts
    └── payment.entity.ts

Problems:
❌ Low cohesion: User code scattered across 4 folders
❌ Hard to find: Need to open 4 files to understand user feature
❌ Hard to test: User test requires files from multiple folders
❌ Hard to delete: Removing user feature requires changes in 4 places
❌ Team conflicts: Multiple teams editing same folders
❌ Unclear boundaries: No clear separation between features

// ✅ FEATURE-BASED (Business features - Recommended)
src/
├── users/                          # User feature
│   ├── users.module.ts
│   ├── controllers/
│   │   └── users.controller.ts
│   ├── services/
│   │   └── users.service.ts
│   ├── repositories/
│   │   └── users.repository.ts
│   ├── entities/
│   │   └── user.entity.ts
│   └── dto/
│       ├── create-user.dto.ts
│       └── update-user.dto.ts
├── orders/                         # Order feature
│   ├── orders.module.ts
│   ├── controllers/
│   ├── services/
│   ├── repositories/
│   ├── entities/
│   └── dto/
├── products/                       # Product feature
│   ├── products.module.ts
│   ├── controllers/
│   ├── services/
│   ├── repositories/
│   ├── entities/
│   └── dto/
└── payments/                       # Payment feature
    ├── payments.module.ts
    ├── controllers/
    ├── services/
    ├── repositories/
    ├── entities/
    └── dto/

Benefits:
✅ High cohesion: All user code in users/ folder
✅ Easy to find: Everything about users in one place
✅ Easy to test: Test entire user feature in isolation
✅ Easy to delete: Delete users/ folder removes entire feature
✅ Team autonomy: User team owns users/ folder
✅ Clear boundaries: Clear separation between features
✅ Scalability: Add new features without touching existing ones
```

---

### **Method 1: Complete Feature Module Example**

```typescript
// ========== USER FEATURE MODULE ==========
// users/users.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UsersController } from './controllers/users.controller';
import { UserProfileController } from './controllers/user-profile.controller';
import { UsersService } from './services/users.service';
import { UserAuthService } from './services/user-auth.service';
import { UsersRepository } from './repositories/users.repository';
import { User } from './entities/user.entity';
import { UserProfile } from './entities/user-profile.entity';
import { SharedModule } from '../shared/shared.module';

@Module({
  imports: [
    TypeOrmModule.forFeature([User, UserProfile]),
    SharedModule,  // For logger, email, etc.
  ],
  controllers: [
    UsersController,
    UserProfileController,
  ],
  providers: [
    UsersService,
    UserAuthService,
    UsersRepository,
  ],
  exports: [
    UsersService,  // Other modules can use this
  ],
})
export class UsersModule {}

// users/controllers/users.controller.ts
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  async findAll(): Promise<User[]> {
    return this.usersService.findAll();
  }

  @Post()
  async create(@Body() dto: CreateUserDto): Promise<User> {
    return this.usersService.create(dto);
  }

  @Get(':id')
  async findOne(@Param('id') id: string): Promise<User> {
    return this.usersService.findById(id);
  }

  @Put(':id')
  async update(
    @Param('id') id: string,
    @Body() dto: UpdateUserDto,
  ): Promise<User> {
    return this.usersService.update(id, dto);
  }

  @Delete(':id')
  async remove(@Param('id') id: string): Promise<void> {
    return this.usersService.remove(id);
  }
}

// users/services/users.service.ts
@Injectable()
export class UsersService {
  constructor(
    private readonly usersRepository: UsersRepository,
    private readonly logger: LoggerService,  // From SharedModule
    private readonly email: EmailService,    // From SharedModule
  ) {}

  async findAll(): Promise<User[]> {
    return this.usersRepository.findAll();
  }

  async findById(id: string): Promise<User> {
    const user = await this.usersRepository.findById(id);
    if (!user) {
      throw new NotFoundException(`User ${id} not found`);
    }
    return user;
  }

  async create(dto: CreateUserDto): Promise<User> {
    this.logger.log('Creating user', 'UsersService');
    
    const existingUser = await this.usersRepository.findByEmail(dto.email);
    if (existingUser) {
      throw new ConflictException('Email already exists');
    }

    const hashedPassword = await bcrypt.hash(dto.password, 10);
    const user = await this.usersRepository.save({
      ...dto,
      password: hashedPassword,
    });

    await this.email.sendWelcomeEmail(user.email);
    
    return user;
  }

  async update(id: string, dto: UpdateUserDto): Promise<User> {
    const user = await this.findById(id);
    Object.assign(user, dto);
    return this.usersRepository.save(user);
  }

  async remove(id: string): Promise<void> {
    const user = await this.findById(id);
    await this.usersRepository.remove(user);
  }
}

// users/repositories/users.repository.ts
@Injectable()
export class UsersRepository {
  constructor(
    @InjectRepository(User)
    private readonly repository: Repository<User>,
  ) {}

  async findAll(): Promise<User[]> {
    return this.repository.find();
  }

  async findById(id: string): Promise<User | null> {
    return this.repository.findOne({ where: { id } });
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.repository.findOne({ where: { email } });
  }

  async save(user: Partial<User>): Promise<User> {
    return this.repository.save(user);
  }

  async remove(user: User): Promise<void> {
    await this.repository.remove(user);
  }
}

// users/entities/user.entity.ts
@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  email: string;

  @Column()
  name: string;

  @Column()
  password: string;

  @Column({ default: true })
  isActive: boolean;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}

// users/dto/create-user.dto.ts
export class CreateUserDto {
  @IsEmail()
  email: string;

  @IsString()
  @MinLength(3)
  name: string;

  @IsString()
  @MinLength(8)
  password: string;
}

// Everything related to users is in users/ folder!
// Easy to find, maintain, test, and delete.
```

---

### **Method 2: Feature Dependencies**

```typescript
// ========== ORDERS MODULE DEPENDS ON USERS MODULE ==========
// orders/orders.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { OrdersController } from './controllers/orders.controller';
import { OrdersService } from './services/orders.service';
import { OrdersRepository } from './repositories/orders.repository';
import { Order } from './entities/order.entity';
import { UsersModule } from '../users/users.module';        // Import feature
import { ProductsModule } from '../products/products.module'; // Import feature
import { PaymentsModule } from '../payments/payments.module'; // Import feature

@Module({
  imports: [
    TypeOrmModule.forFeature([Order]),
    UsersModule,      // Use user services
    ProductsModule,   // Use product services
    PaymentsModule,   // Use payment services
  ],
  controllers: [OrdersController],
  providers: [OrdersService, OrdersRepository],
  exports: [OrdersService],
})
export class OrdersModule {}

// orders/services/orders.service.ts
@Injectable()
export class OrdersService {
  constructor(
    private readonly ordersRepository: OrdersRepository,
    private readonly usersService: UsersService,        // From UsersModule
    private readonly productsService: ProductsService,  // From ProductsModule
    private readonly paymentsService: PaymentsService,  // From PaymentsModule
    private readonly logger: LoggerService,
  ) {}

  async createOrder(dto: CreateOrderDto): Promise<Order> {
    this.logger.log('Creating order', 'OrdersService');

    // Use UserService from UsersModule
    const user = await this.usersService.findById(dto.userId);
    if (!user) {
      throw new NotFoundException('User not found');
    }

    // Use ProductsService from ProductsModule
    const products = await this.productsService.findByIds(dto.productIds);
    if (products.length !== dto.productIds.length) {
      throw new BadRequestException('Some products not found');
    }

    // Calculate total
    const total = products.reduce((sum, p) => sum + p.price, 0);

    // Create order
    const order = await this.ordersRepository.save({
      userId: dto.userId,
      productIds: dto.productIds,
      total,
      status: 'pending',
    });

    // Use PaymentsService from PaymentsModule
    await this.paymentsService.processPayment(order.id, total);

    this.logger.log(`Order created: ${order.id}`, 'OrdersService');

    return order;
  }
}

// Feature dependency graph:
// OrdersModule → UsersModule, ProductsModule, PaymentsModule
// Each feature is independent but can use other features via modules
```

---

### **Method 3: Feature Folder Structure**

```typescript
// ========== STANDARD FEATURE STRUCTURE ==========
users/
├── users.module.ts                 # Module definition
├── controllers/                    # HTTP layer
│   ├── users.controller.ts         # Main user endpoints
│   ├── user-auth.controller.ts     # Auth endpoints
│   └── user-profile.controller.ts  # Profile endpoints
├── services/                       # Business logic layer
│   ├── users.service.ts            # Main user service
│   ├── user-auth.service.ts        # Auth service
│   └── user-profile.service.ts     # Profile service
├── repositories/                   # Data access layer
│   ├── users.repository.ts         # User data access
│   └── user-profile.repository.ts  # Profile data access
├── entities/                       # Domain models
│   ├── user.entity.ts              # User entity
│   └── user-profile.entity.ts      # Profile entity
├── dto/                           # Data transfer objects
│   ├── create-user.dto.ts
│   ├── update-user.dto.ts
│   ├── login.dto.ts
│   └── user-response.dto.ts
├── guards/                        # Feature-specific guards
│   └── user-ownership.guard.ts
├── interfaces/                    # TypeScript interfaces
│   └── user.interface.ts
├── types/                         # Type definitions
│   └── user-role.type.ts
├── constants/                     # Feature constants
│   └── user.constants.ts
├── events/                        # Domain events
│   ├── user-created.event.ts
│   └── user-updated.event.ts
└── tests/                         # Feature tests
    ├── users.service.spec.ts
    ├── users.controller.spec.ts
    └── users.e2e.spec.ts

// Benefits:
// ✅ Self-contained: Everything users need is in users/ folder
// ✅ Independent: Can develop, test, deploy users feature independently
// ✅ Clear ownership: User team owns entire users/ folder
// ✅ Easy navigation: All user code in one place
```

---

### **Method 4: Bounded Context (DDD-style Features)**

```typescript
// ========== E-COMMERCE BOUNDED CONTEXTS ==========
src/
├── app.module.ts
├── catalog/                        # Catalog context
│   ├── catalog.module.ts
│   ├── products/
│   │   ├── controllers/
│   │   ├── services/
│   │   └── entities/
│   ├── categories/
│   │   ├── controllers/
│   │   ├── services/
│   │   └── entities/
│   └── reviews/
│       ├── controllers/
│       ├── services/
│       └── entities/
├── checkout/                       # Checkout context
│   ├── checkout.module.ts
│   ├── cart/
│   │   ├── controllers/
│   │   ├── services/
│   │   └── entities/
│   ├── orders/
│   │   ├── controllers/
│   │   ├── services/
│   │   └── entities/
│   └── payments/
│       ├── controllers/
│       ├── services/
│       └── entities/
├── identity/                       # Identity context
│   ├── identity.module.ts
│   ├── users/
│   │   ├── controllers/
│   │   ├── services/
│   │   └── entities/
│   ├── auth/
│   │   ├── controllers/
│   │   ├── services/
│   │   └── strategies/
│   └── roles/
│       ├── controllers/
│       ├── services/
│       └── entities/
└── fulfillment/                    # Fulfillment context
    ├── fulfillment.module.ts
    ├── shipping/
    │   ├── controllers/
    │   ├── services/
    │   └── entities/
    ├── inventory/
    │   ├── controllers/
    │   ├── services/
    │   └── entities/
    └── warehouses/
        ├── controllers/
        ├── services/
        └── entities/

// catalog/catalog.module.ts
@Module({
  imports: [
    ProductsModule,
    CategoriesModule,
    ReviewsModule,
  ],
  exports: [
    ProductsModule,   // Other contexts can use products
  ],
})
export class CatalogModule {}

// checkout/checkout.module.ts
@Module({
  imports: [
    CartModule,
    OrdersModule,
    PaymentsModule,
    CatalogModule,    // Import catalog to use products
    IdentityModule,   // Import identity to use users
  ],
  exports: [
    OrdersModule,     // Other contexts can use orders
  ],
})
export class CheckoutModule {}

// Benefits:
// ✅ Domain-driven: Organized by business domains
// ✅ Clear boundaries: Each context is independent
// ✅ Scalability: Contexts can become microservices later
// ✅ Team autonomy: Different teams own different contexts
```

---

### **Method 5: Feature Flags and Modularity**

```typescript
// ========== ENABLE/DISABLE FEATURES VIA MODULES ==========
// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { UsersModule } from './users/users.module';
import { OrdersModule } from './orders/orders.module';
import { ProductsModule } from './products/products.module';
import { RecommendationsModule } from './recommendations/recommendations.module';
import { AnalyticsModule } from './analytics/analytics.module';

@Module({
  imports: [
    ConfigModule.forRoot(),
    
    // Core features (always enabled)
    UsersModule,
    OrdersModule,
    ProductsModule,
    
    // Optional features (feature flags)
    ...(process.env.ENABLE_RECOMMENDATIONS === 'true' ? [RecommendationsModule] : []),
    ...(process.env.ENABLE_ANALYTICS === 'true' ? [AnalyticsModule] : []),
  ],
})
export class AppModule {}

// OR use dynamic imports:
@Module({})
export class AppModule {
  static async forRoot(config: ConfigService): Promise<DynamicModule> {
    const imports = [
      UsersModule,
      OrdersModule,
      ProductsModule,
    ];

    // Conditionally add features
    if (config.get('ENABLE_RECOMMENDATIONS')) {
      imports.push(RecommendationsModule);
    }

    if (config.get('ENABLE_ANALYTICS')) {
      imports.push(AnalyticsModule);
    }

    return {
      module: AppModule,
      imports,
    };
  }
}

// Benefits:
// ✅ Feature toggles: Enable/disable features without code changes
// ✅ A/B testing: Different features for different users
// ✅ Gradual rollout: Enable features gradually
// ✅ Reduced bundle size: Don't load disabled features
```

---

### **Benefits Summary:**

| Aspect | Feature-based | Layer-based |
|--------|--------------|-------------|
| **Cohesion** | High (all related code together) | Low (code scattered) |
| **Finding code** | Easy (one folder) | Hard (multiple folders) |
| **Testing** | Easy (isolated feature) | Hard (dependencies across layers) |
| **Team ownership** | Clear (one team per feature) | Unclear (shared folders) |
| **Scaling** | Easy (add features independently) | Hard (all in same layers) |
| **Deletion** | Easy (delete one folder) | Hard (delete from multiple places) |
| **Microservices** | Easy (feature → service) | Hard (need to refactor) |
| **Deployment** | Independent | Coupled |

---

### **Best Practices:**

```typescript
// ✅ GOOD: One feature per module
users/users.module.ts     // User feature
orders/orders.module.ts   // Order feature
products/products.module.ts // Product feature

// ❌ BAD: Mixed features in one module
@Module({
  controllers: [UsersController, OrdersController, ProductsController],
  providers: [UsersService, OrdersService, ProductsService],
})
export class EverythingModule {}

// ✅ GOOD: Feature has everything it needs
users/
├── controllers/
├── services/
├── repositories/
├── entities/
└── dto/

// ❌ BAD: Feature scattered across folders
src/
├── controllers/users.controller.ts
├── services/users.service.ts
└── repositories/users.repository.ts

// ✅ GOOD: Clear feature dependencies
@Module({
  imports: [UsersModule, ProductsModule],  // Explicit dependencies
})
export class OrdersModule {}

// ❌ BAD: Hidden dependencies
// OrdersService uses UsersService but doesn't import UsersModule

// ✅ GOOD: Feature exports public API only
@Module({
  providers: [UsersService, UsersRepository, UserHelper],
  exports: [UsersService],  // Only public API
})
export class UsersModule {}

// ❌ BAD: Feature exports everything
@Module({
  providers: [UsersService, UsersRepository, UserHelper],
  exports: [UsersService, UsersRepository, UserHelper],  // Too much!
})

// ✅ GOOD: Feature can be removed easily
// Delete users/ folder → users feature removed

// ❌ BAD: Feature code everywhere
// Need to delete from controllers/, services/, repositories/, entities/
```

**Key Takeaway:** Feature-based modularization (by business feature/domain)

- **What:** Organize code by features (e.g., users, orders, products) instead of by technical layer.
- **Module contents:** each feature/module holds controllers, services, repositories, entities, and DTOs.
- **Benefits:** **high cohesion**, **low coupling**, easier navigation, team autonomy, independent testing & deployment, and simpler refactor to microservices.
- **Rules of thumb:** use one module per feature, declare explicit imports for dependencies, and export only public APIs.

</details>

<details>
<summary><strong>8. When should you create a new module?</strong></summary>

**Answer:**

When to create a new module

- **Create a module when:** you have a distinct business feature/domain (e.g., users, orders), shared functionality used by multiple modules, a DDD bounded context, or a cross-cutting concern (auth, logging).
- **Guidelines:** prefer single responsibility, ~5–10+ related files, clear boundaries, and measurable potential for reuse or independent deployment.
- **Avoid premature modularization:** start simple and refactor into modules when features grow; don’t create modules for tiny utilities or unclear responsibilities.

---

### **When to Create a Module:**

```typescript
// ✅ CREATE MODULE: Distinct business feature
// Users is a clear business domain
@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UsersController],
  providers: [UsersService, UsersRepository],
  exports: [UsersService],
})
export class UsersModule {}

// ✅ CREATE MODULE: Shared functionality
// Logger used by all features
@Module({
  providers: [LoggerService],
  exports: [LoggerService],
})
export class LoggerModule {}

// ✅ CREATE MODULE: Cross-cutting concern
// Authentication used across app
@Module({
  imports: [JwtModule.register({ secret: 'secret' })],
  providers: [AuthService, JwtStrategy],
  exports: [AuthService],
})
export class AuthModule {}

// ❌ DON'T CREATE MODULE: Too small
// Single utility function doesn't need module
@Module({
  providers: [{ provide: 'UTILS', useValue: { formatDate: () => {} } }],
})
export class UtilsModule {}  // Overkill!

// ❌ DON'T CREATE MODULE: No clear boundary
// Mixed responsibilities
@Module({
  controllers: [UsersController, OrdersController, ProductsController],
  providers: [UsersService, OrdersService, ProductsService],
})
export class RandomStuffModule {}  // What is this module for?
```

---

### **Method 1: Feature Size Rule of Thumb**

```typescript
// ========== RULE: Create module when feature has 5+ related files ==========

// TOO SMALL for a module (2-3 files)
utils/
├── date.util.ts
└── string.util.ts
// Solution: Put in shared/ folder, not separate module

// GOOD SIZE for a module (5-10 files)
users/
├── users.module.ts
├── controllers/
│   └── users.controller.ts
├── services/
│   └── users.service.ts
├── repositories/
│   └── users.repository.ts
├── entities/
│   └── user.entity.ts
└── dto/
    ├── create-user.dto.ts
    └── update-user.dto.ts
// ✅ Clear feature with multiple components

// LARGE feature (15+ files) - might need sub-modules
users/
├── users.module.ts
├── authentication/
│   ├── auth.controller.ts
│   ├── auth.service.ts
│   └── strategies/
│       ├── jwt.strategy.ts
│       └── local.strategy.ts
├── profile/
│   ├── profile.controller.ts
│   ├── profile.service.ts
│   └── profile.entity.ts
├── permissions/
│   ├── permissions.controller.ts
│   ├── permissions.service.ts
│   └── permissions.entity.ts
└── notifications/
    ├── notifications.service.ts
    └── notifications.entity.ts
// Consider splitting into sub-modules:
// - AuthModule (authentication concern)
// - ProfileModule (profile management)
// - PermissionsModule (authorization concern)
```

---

### **Method 2: Business Domain Checklist**

```typescript
// ========== CHECKLIST: Should this be a module? ==========

// ✅ YES - Create UserModule
// □ Distinct business domain? ✓ (User management)
// □ Has own entities? ✓ (User entity)
// □ Has own business logic? ✓ (Registration, authentication)
// □ Used by other modules? ✓ (Orders need user info)
// □ Can be tested independently? ✓
// □ Could become microservice? ✓
@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UsersController],
  providers: [UsersService, UsersRepository],
  exports: [UsersService],
})
export class UsersModule {}

// ✅ YES - Create OrdersModule
// □ Distinct business domain? ✓ (Order management)
// □ Has own entities? ✓ (Order, OrderItem entities)
// □ Has own business logic? ✓ (Order creation, payment, fulfillment)
// □ Used by other modules? ✓ (Reports, Analytics)
// □ Can be tested independently? ✓
// □ Could become microservice? ✓
@Module({
  imports: [
    TypeOrmModule.forFeature([Order, OrderItem]),
    UsersModule,
    ProductsModule,
  ],
  controllers: [OrdersController],
  providers: [OrdersService, OrdersRepository],
  exports: [OrdersService],
})
export class OrdersModule {}

// ❌ NO - Don't create HelperModule
// □ Distinct business domain? ✗ (Just utilities)
// □ Has own entities? ✗
// □ Has own business logic? ✗ (Just helpers)
// □ Used by other modules? ✓
// □ Can be tested independently? ✓ (but simple functions)
// □ Could become microservice? ✗ (too small)
// Solution: Put in shared/utils/ folder
export class DateUtils {
  static format(date: Date): string { /* ... */ }
}

// ❌ NO - Don't create ValidationModule for one validator
// □ Distinct business domain? ✗
// □ Has own entities? ✗
// □ Has own business logic? ✗
// □ Used by other modules? ✓
// □ Can be tested independently? ✓
// □ Could become microservice? ✗
// Solution: Use built-in ValidationPipe or put in shared/
```

---

### **Method 3: Module Lifecycle Triggers**

```typescript
// ========== WHEN TO SPLIT: Module getting too large ==========

// BEFORE: One large UsersModule (20+ files)
users/
├── users.module.ts
├── authentication/        # Auth logic
├── authorization/         # Permission logic
├── profile/              # Profile management
├── preferences/          # User preferences
├── notifications/        # User notifications
└── social/              # Social features

// Problem: UsersModule doing too much!

// AFTER: Split into focused modules
@Module({
  imports: [
    UserAuthModule,          // Authentication (login, register)
    UserAuthorizationModule, // Authorization (permissions, roles)
    UserProfileModule,       // Profile management
    UserPreferencesModule,   // User settings
    UserNotificationsModule, // Notifications
    UserSocialModule,        // Social features
  ],
  exports: [
    UserAuthModule,
    UserProfileModule,
  ],
})
export class UsersModule {}

// Each sub-module has clear responsibility
@Module({
  controllers: [AuthController],
  providers: [AuthService, JwtStrategy, LocalStrategy],
  exports: [AuthService],
})
export class UserAuthModule {}

// ========== WHEN TO SHARE: Multiple modules need same code ==========

// BEFORE: Duplicated email code
users/services/user-email.service.ts
orders/services/order-email.service.ts
notifications/services/notification-email.service.ts
// Problem: Same email logic in 3 places!

// AFTER: Create shared EmailModule
@Module({
  providers: [EmailService],
  exports: [EmailService],
})
export class EmailModule {}

// All modules import EmailModule
@Module({
  imports: [EmailModule],
  providers: [UsersService],
})
export class UsersModule {}

// ========== WHEN TO EXTRACT: Feature can be independent ==========

// BEFORE: Payment logic inside OrdersModule
orders/
├── orders.module.ts
├── services/
│   ├── orders.service.ts
│   └── payment.service.ts    # Payment logic here
└── entities/
    ├── order.entity.ts
    └── payment.entity.ts

// Problem: Payment is independent concern, might need different team

// AFTER: Extract PaymentsModule
@Module({
  controllers: [PaymentsController],
  providers: [PaymentsService, PaymentsRepository],
  exports: [PaymentsService],
})
export class PaymentsModule {}

@Module({
  imports: [PaymentsModule],  // Import instead of embedding
  controllers: [OrdersController],
  providers: [OrdersService],
})
export class OrdersModule {}
```

---

### **Method 4: Module Granularity Examples**

```typescript
// ========== TOO FINE-GRAINED (Over-modularization) ==========
// ❌ DON'T DO THIS: Too many tiny modules
@Module({ providers: [DateFormatter] })
export class DateFormatterModule {}

@Module({ providers: [StringValidator] })
export class StringValidatorModule {}

@Module({ providers: [NumberUtils] })
export class NumberUtilsModule {}

// Problem: Module overhead > benefit

// ✅ BETTER: Group related utilities
@Module({
  providers: [
    DateFormatter,
    StringValidator,
    NumberUtils,
  ],
  exports: [DateFormatter, StringValidator, NumberUtils],
})
export class UtilsModule {}

// ========== TOO COARSE-GRAINED (Under-modularization) ==========
// ❌ DON'T DO THIS: One huge module
@Module({
  controllers: [
    UsersController,
    OrdersController,
    ProductsController,
    PaymentsController,
    ShippingController,
    ReviewsController,
    // ... 20 more controllers
  ],
  providers: [
    UsersService,
    OrdersService,
    ProductsService,
    // ... 50 more services
  ],
})
export class EverythingModule {}

// Problem: Hard to maintain, test, understand

// ✅ BETTER: Split by feature
@Module({ imports: [UsersModule, OrdersModule, ProductsModule] })
export class AppModule {}

// ========== JUST RIGHT (Good granularity) ==========
// ✅ GOOD: One module per business feature
@Module({
  controllers: [UsersController],
  providers: [UsersService, UsersRepository],
  exports: [UsersService],
})
export class UsersModule {}

@Module({
  controllers: [OrdersController],
  providers: [OrdersService, OrdersRepository],
  exports: [OrdersService],
})
export class OrdersModule {}

// ✅ GOOD: Shared module for common utilities
@Module({
  providers: [LoggerService, EmailService, CacheService],
  exports: [LoggerService, EmailService, CacheService],
})
export class SharedModule {}
```

---

### **Method 5: Decision Tree**

```typescript
// ========== MODULE CREATION DECISION TREE ==========

/*
START: Do you need a new module?
│
├─ Is it a distinct business feature? (User, Order, Product)
│  ├─ YES → Create Feature Module
│  │        @Module({ imports, controllers, providers, exports })
│  │        Example: UsersModule, OrdersModule
│  │
│  └─ NO → Continue
│     │
│     ├─ Is it shared by multiple features? (Logger, Email, Cache)
│     │  ├─ YES → Create Shared Module
│     │  │        @Module({ providers, exports })
│     │  │        Example: SharedModule, LoggerModule
│     │  │
│     │  └─ NO → Continue
│     │     │
│     │     ├─ Is it infrastructure concern? (Database, Config, Redis)
│     │     │  ├─ YES → Create Core/Infrastructure Module
│     │     │  │        @Module({ imports, providers, exports })
│     │     │  │        Example: DatabaseModule, ConfigModule
│     │     │  │
│     │     │  └─ NO → Continue
│     │     │     │
│     │     │     ├─ Is it cross-cutting concern? (Auth, Logging, Caching)
│     │     │     │  ├─ YES → Create Cross-cutting Module
│     │     │     │  │        @Global() @Module({ ... })
│     │     │     │  │        Example: AuthModule, LoggingModule
│     │     │     │  │
│     │     │     │  └─ NO → Continue
│     │     │     │     │
│     │     │     │     └─ Is it really small? (1-2 files, simple utilities)
│     │     │     │        ├─ YES → DON'T create module
│     │     │     │        │        Put in shared/utils/ folder
│     │     │     │        │        Example: date.util.ts, string.util.ts
│     │     │     │        │
│     │     │     │        └─ NO → Re-evaluate, might need module
│
└─ Module too large? (20+ files)
   ├─ YES → Split into sub-modules
   │        Example: UsersModule → AuthModule + ProfileModule + NotificationsModule
   │
   └─ NO → Keep as single module
*/

// Implementation examples:

// Feature Module
@Module({ /* ... */ })
export class UsersModule {}

// Shared Module
@Module({ /* ... */ })
export class SharedModule {}

// Infrastructure Module
@Module({ /* ... */ })
export class DatabaseModule {}

// Cross-cutting Module
@Global()
@Module({ /* ... */ })
export class AuthModule {}

// No Module - Just a utility file
// shared/utils/date.util.ts
export class DateUtils {
  static format() { /* ... */ }
}
```

---

### **Best Practices:**

```typescript
// ✅ GOOD: Create module for distinct feature
@Module({ /* ... */ })
export class UsersModule {}

// ❌ BAD: One module for everything
@Module({
  controllers: [/* 50 controllers */],
  providers: [/* 100 providers */],
})
export class AppModule {}

// ✅ GOOD: Create module when used by 3+ features
@Module({ providers: [LoggerService], exports: [LoggerService] })
export class LoggerModule {}
// Used by: UsersModule, OrdersModule, ProductsModule

// ❌ BAD: Create module for one-off utility
@Module({ providers: [{ provide: 'HELPER', useValue: {} }] })
export class SingleHelperModule {}

// ✅ GOOD: Module has clear responsibility
@Module({ /* Authentication only */ })
export class AuthModule {}

// ❌ BAD: Module with mixed responsibilities
@Module({ /* Auth + Email + Cache + Logging + ... */ })
export class MiscModule {}

// ✅ GOOD: Start simple, refactor when needed
// Start: Put everything in AppModule
// Later: Extract UsersModule when user code grows
// Later: Extract SharedModule when utilities grow

// ❌ BAD: Premature modularization
// Creating 50 modules for a simple app

// ✅ GOOD: Module can be tested independently
// UserModule can be tested without OrdersModule

// ❌ BAD: Modules tightly coupled
// UserModule and OrdersModule share internal state
```

**Key Takeaway:** When to create a module

- **When:** distinct business feature/domain (users, orders, products), shared functionality used by 3+ modules (logger, email, cache), cross-cutting concern (auth, logging), DDD bounded context, or a module growing beyond ~15–20 files (split into sub-modules).
- **When NOT to:** single utility functions, 1–2 trivial files, or unclear responsibilities (avoid premature modularization).
- **Guidelines:** aim for single responsibility, clear boundaries, ~5–10+ related files, potential for reuse, and independent testability.

</details>

<details>
<summary><strong>9. What is the difference between shared modules and feature modules?</strong></summary>

**Answer:**

Feature vs. shared modules

- **Feature modules:** organize code by business domain (users, orders, products). They encapsulate feature-specific controllers, services, repositories, and entities (vertical slicing).
- **Shared modules:** provide cross-cutting concerns and utilities used across features (logger, email, database, cache). They offer horizontal services consumed by feature modules.
- **Quick rule:** Feature = vertical feature slice; Shared = horizontal utilities and infrastructure.

---

### **Comparison:**

| Aspect | Feature Module | Shared Module |
|--------|----------------|---------------|
| **Purpose** | Implement business feature | Provide reusable utilities |
| **Scope** | Single domain (Users, Orders) | Cross-cutting concerns |
| **Controllers** | Yes (HTTP endpoints) | No (internal services) |
| **Entities** | Yes (domain entities) | No (or generic entities) |
| **Business Logic** | Yes (feature-specific) | No (generic helpers) |
| **Used By** | Other features (optional) | Many/all features |
| **Examples** | UsersModule, OrdersModule | LoggerModule, EmailModule |
| **Export** | Feature services (optional) | All services (required) |
| **Import** | Feature dependencies | Infrastructure dependencies |
| **Independence** | Can work standalone | Support other modules |

---

### **Method 1: Feature Module Example**

```typescript
// ========== FEATURE MODULE: UsersModule ==========
// users/users.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UsersController } from './controllers/users.controller';
import { UsersService } from './services/users.service';
import { UsersRepository } from './repositories/users.repository';
import { User } from './entities/user.entity';
import { SharedModule } from '../shared/shared.module';  // Import shared utilities

@Module({
  imports: [
    TypeOrmModule.forFeature([User]),  // Feature-specific entity
    SharedModule,                       // Import shared services
  ],
  controllers: [UsersController],      // HTTP endpoints for users
  providers: [
    UsersService,                      // User business logic
    UsersRepository,                   // User data access
  ],
  exports: [UsersService],             // Export if other features need it
})
export class UsersModule {}

// users/controllers/users.controller.ts
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  async findAll(): Promise<User[]> {
    return this.usersService.findAll();
  }

  @Post()
  async create(@Body() dto: CreateUserDto): Promise<User> {
    return this.usersService.create(dto);
  }
}

// users/services/users.service.ts
@Injectable()
export class UsersService {
  constructor(
    private readonly usersRepository: UsersRepository,
    private readonly logger: LoggerService,  // From SharedModule
    private readonly email: EmailService,    // From SharedModule
  ) {}

  async findAll(): Promise<User[]> {
    this.logger.log('Fetching all users', 'UsersService');
    return this.usersRepository.findAll();
  }

  async create(dto: CreateUserDto): Promise<User> {
    this.logger.log('Creating user', 'UsersService');
    
    const user = await this.usersRepository.save(dto);
    
    // Use shared email service
    await this.email.sendWelcomeEmail(user.email);
    
    return user;
  }
}

// users/entities/user.entity.ts
@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  name: string;

  @Column({ unique: true })
  email: string;

  // Feature-specific entity: User domain
}

// Characteristics of Feature Module:
// ✅ Has controllers (HTTP layer)
// ✅ Has services (business logic)
// ✅ Has repositories (data access)
// ✅ Has entities (domain models)
// ✅ Focused on ONE business domain (users)
// ✅ Can import shared modules
// ✅ May export services for other features
```

---

### **Method 2: Shared Module Example**

```typescript
// ========== SHARED MODULE: Utilities for all features ==========
// shared/shared.module.ts
import { Module, Global } from '@nestjs/common';
import { LoggerService } from './services/logger.service';
import { EmailService } from './services/email.service';
import { CacheService } from './services/cache.service';
import { DateUtils } from './utils/date.utils';
import { StringUtils } from './utils/string.utils';

@Global()  // Make available globally without importing
@Module({
  providers: [
    LoggerService,   // Logging utility
    EmailService,    // Email sending utility
    CacheService,    // Caching utility
    DateUtils,       // Date helpers
    StringUtils,     // String helpers
  ],
  exports: [
    // Export EVERYTHING so other modules can use
    LoggerService,
    EmailService,
    CacheService,
    DateUtils,
    StringUtils,
  ],
})
export class SharedModule {}

// shared/services/logger.service.ts
@Injectable()
export class LoggerService {
  log(message: string, context?: string): void {
    console.log(`[${context || 'App'}] ${message}`);
  }

  error(message: string, trace?: string, context?: string): void {
    console.error(`[${context || 'App'}] ERROR: ${message}`, trace);
  }

  warn(message: string, context?: string): void {
    console.warn(`[${context || 'App'}] WARNING: ${message}`);
  }

  // Generic logging service - no business logic
}

// shared/services/email.service.ts
@Injectable()
export class EmailService {
  async sendEmail(to: string, subject: string, body: string): Promise<void> {
    // Generic email sending - used by all features
    console.log(`Sending email to ${to}: ${subject}`);
  }

  async sendWelcomeEmail(email: string): Promise<void> {
    return this.sendEmail(email, 'Welcome!', 'Welcome to our platform');
  }

  async sendOrderConfirmation(email: string, orderId: string): Promise<void> {
    return this.sendEmail(email, 'Order Confirmation', `Order ${orderId} confirmed`);
  }

  // Generic email utilities - no feature-specific logic
}

// shared/services/cache.service.ts
@Injectable()
export class CacheService {
  private cache = new Map<string, any>();

  async get<T>(key: string): Promise<T | null> {
    return this.cache.get(key) || null;
  }

  async set(key: string, value: any, ttl?: number): Promise<void> {
    this.cache.set(key, value);
    if (ttl) {
      setTimeout(() => this.cache.delete(key), ttl * 1000);
    }
  }

  async delete(key: string): Promise<void> {
    this.cache.delete(key);
  }

  // Generic caching - used by all features
}

// shared/utils/date.utils.ts
@Injectable()
export class DateUtils {
  format(date: Date, format: string): string {
    // Date formatting utility
    return date.toISOString();
  }

  addDays(date: Date, days: number): Date {
    const result = new Date(date);
    result.setDate(result.getDate() + days);
    return result;
  }

  // Generic date utilities - no business logic
}

// Characteristics of Shared Module:
// ✅ NO controllers (not HTTP endpoints)
// ✅ NO entities (no domain models)
// ✅ NO business logic (generic utilities only)
// ✅ Used by MANY features (cross-cutting)
// ✅ Exports ALL services
// ✅ Often @Global() for convenience
// ✅ Infrastructure/utilities, not features
```

---

### **Method 3: How They Work Together**

```typescript
// ========== APP STRUCTURE ==========
src/
├── app.module.ts
├── shared/                        # SHARED MODULE
│   ├── shared.module.ts           # Exports utilities
│   ├── services/
│   │   ├── logger.service.ts
│   │   ├── email.service.ts
│   │   └── cache.service.ts
│   └── utils/
│       ├── date.utils.ts
│       └── string.utils.ts
├── users/                         # FEATURE MODULE
│   ├── users.module.ts            # Imports SharedModule
│   ├── controllers/
│   │   └── users.controller.ts
│   ├── services/
│   │   └── users.service.ts
│   └── entities/
│       └── user.entity.ts
├── orders/                        # FEATURE MODULE
│   ├── orders.module.ts           # Imports SharedModule
│   ├── controllers/
│   │   └── orders.controller.ts
│   ├── services/
│   │   └── orders.service.ts
│   └── entities/
│       └── order.entity.ts
└── products/                      # FEATURE MODULE
    ├── products.module.ts         # Imports SharedModule
    ├── controllers/
    │   └── products.controller.ts
    ├── services/
    │   └── products.service.ts
    └── entities/
        └── product.entity.ts

// app.module.ts
@Module({
  imports: [
    SharedModule,    // Shared utilities
    UsersModule,     // Feature: User management
    OrdersModule,    // Feature: Order management
    ProductsModule,  // Feature: Product management
  ],
})
export class AppModule {}

// Dependency flow:
// UsersModule → SharedModule (uses logger, email)
// OrdersModule → SharedModule (uses logger, email)
// OrdersModule → UsersModule (uses user service)
// OrdersModule → ProductsModule (uses product service)

// Feature modules depend on:
// - Shared module (for utilities)
// - Other feature modules (for cross-feature logic)

// Shared module depends on:
// - Infrastructure (database, config)
// - External libraries
```

---

### **Method 4: When to Use Each**

```typescript
// ========== FEATURE MODULE - Use when: ==========
// ✅ Distinct business domain
@Module({ /* ... */ })
export class UsersModule {}  // User management domain

// ✅ Has HTTP endpoints
@Controller('orders')
export class OrdersController {}  // Orders feature

// ✅ Has domain entities
@Entity('products')
export class Product {}  // Product domain model

// ✅ Business logic specific to one feature
@Injectable()
export class OrdersService {
  async createOrder() { /* Order-specific logic */ }
}

// ✅ Can work independently
// UsersModule can be tested without OrdersModule

// ========== SHARED MODULE - Use when: ==========
// ✅ Used by 3+ feature modules
@Module({ exports: [LoggerService] })
export class SharedModule {}  // All features need logging

// ✅ Cross-cutting concern
@Injectable()
export class AuthGuard {}  // Authentication for all endpoints

// ✅ Infrastructure/utilities
@Injectable()
export class DatabaseService {}  // Database connection for all

// ✅ NO business logic
@Injectable()
export class EmailService {
  // Generic email sending, no feature-specific logic
  async sendEmail(to: string, subject: string, body: string) {}
}

// ✅ Generic helpers
export class StringUtils {
  static capitalize(str: string): string { /* ... */ }
}

// ========== ANTI-PATTERNS ==========
// ❌ BAD: Business logic in shared module
@Injectable()
export class SharedModule {
  // DON'T: User-specific logic in shared module
  async createUser(dto: CreateUserDto) { /* ... */ }
}
// Solution: Move to UsersModule

// ❌ BAD: Feature module with no clear domain
@Module({
  controllers: [UsersController, OrdersController],  // Multiple domains
})
export class MixedModule {}
// Solution: Split into UsersModule and OrdersModule

// ❌ BAD: Shared module with domain entity
@Entity('users')
export class User {}  // In SharedModule
// Solution: Move to UsersModule (feature)

// ❌ BAD: Feature module with only utilities
@Module({
  providers: [{ provide: 'UTILS', useValue: { format: () => {} } }],
})
export class HelperModule {}
// Solution: Move to SharedModule or use plain functions
```

---

### **Method 5: Advanced Patterns**

```typescript
// ========== CORE MODULE (Infrastructure Shared) ==========
// core/core.module.ts
@Global()
@Module({
  imports: [
    TypeOrmModule.forRoot(databaseConfig),  // Database
    ConfigModule.forRoot(),                 // Configuration
    RedisModule.forRoot(redisConfig),       // Redis
  ],
  providers: [
    DatabaseService,
    ConfigService,
    RedisService,
  ],
  exports: [
    DatabaseService,
    ConfigService,
    RedisService,
  ],
})
export class CoreModule {}

// ========== COMMON MODULE (Business Shared) ==========
// common/common.module.ts
@Module({
  providers: [
    LoggerService,
    EmailService,
    CacheService,
  ],
  exports: [
    LoggerService,
    EmailService,
    CacheService,
  ],
})
export class CommonModule {}

// ========== AUTH MODULE (Cross-cutting Feature) ==========
// auth/auth.module.ts
@Global()  // Authentication needed everywhere
@Module({
  imports: [
    JwtModule.register({ secret: 'secret' }),
    UsersModule,  // Auth depends on users feature
  ],
  providers: [
    AuthService,
    JwtStrategy,
    LocalStrategy,
  ],
  exports: [AuthService],
})
export class AuthModule {}

// ========== FEATURE MODULE WITH SUB-FEATURES ==========
// users/users.module.ts
@Module({
  imports: [
    UserAuthModule,      // Sub-feature: User authentication
    UserProfileModule,   // Sub-feature: User profiles
    UserSettingsModule,  // Sub-feature: User settings
  ],
  exports: [
    UserAuthModule,
    UserProfileModule,
  ],
})
export class UsersModule {}

// Module hierarchy:
// AppModule
// ├── CoreModule (infrastructure shared - @Global)
// ├── CommonModule (business shared)
// ├── AuthModule (cross-cutting feature - @Global)
// ├── UsersModule (feature)
// │   ├── UserAuthModule (sub-feature)
// │   ├── UserProfileModule (sub-feature)
// │   └── UserSettingsModule (sub-feature)
// ├── OrdersModule (feature)
// └── ProductsModule (feature)
```

---

### **Best Practices:**

```typescript
// ✅ GOOD: Clear separation
// Shared: Generic utilities
@Module({ providers: [LoggerService], exports: [LoggerService] })
export class SharedModule {}

// Feature: Business domain
@Module({ controllers: [UsersController], providers: [UsersService] })
export class UsersModule {}

// ❌ BAD: Mixed concerns
@Module({
  providers: [LoggerService, UsersService, OrdersService],  // Mixed!
})
export class SharedModule {}

// ✅ GOOD: Shared module exports everything
@Module({
  providers: [LoggerService, EmailService],
  exports: [LoggerService, EmailService],  // Export all
})
export class SharedModule {}

// ❌ BAD: Shared module doesn't export
@Module({
  providers: [LoggerService],
  exports: [],  // Can't use it!
})
export class SharedModule {}

// ✅ GOOD: Feature module imports shared
@Module({
  imports: [SharedModule],  // Use shared utilities
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}

// ❌ BAD: Duplicating shared code
@Module({
  providers: [
    UsersService,
    { provide: 'LOGGER', useValue: console.log },  // Duplicate logger!
  ],
})
export class UsersModule {}

// ✅ GOOD: Use @Global() for truly global shared modules
@Global()
@Module({ providers: [LoggerService], exports: [LoggerService] })
export class SharedModule {}
// Now all modules can use LoggerService without importing

// ❌ BAD: @Global() for feature modules
@Global()
@Module({ controllers: [UsersController] })
export class UsersModule {}  // Don't make features global!
```

**Key Takeaway:**

- **Feature modules (vertical):** One module per business domain (e.g., `UserModule`, `OrderModule`) containing controllers, services, repositories, entities — focused on a single domain.
- **Shared modules (horizontal):** Provide cross-cutting utilities (logger, email, cache, date helpers); typically export providers, contain no controllers/entities, and are often `@Global()` for convenience.
- **How they interact:** Feature modules **import** shared modules to reuse utilities and **export** services only when other features need them; keep feature logic inside feature modules and infrastructure/utilities in shared/core modules.

</details>

<details>
<summary><strong>10. What is Dependency Injection pattern?</strong></summary>

**Answer:**

**Dependency Injection (DI)** is a design pattern where objects receive their dependencies from external sources rather than creating them internally.

- **How it works in NestJS:** The **IoC (Inversion of Control) container** reads constructor params and provides instances automatically (constructor injection).
- **Why use DI:** Promotes **loose coupling**, **easy testing** (inject mocks), **better maintainability**, and **flexibility** (swap implementations or provide test doubles).

- **Best practice:** Prefer constructor injection with `@Injectable()` and depend on abstractions (interfaces/tokens) when possible.

---

### **Without vs With DI:**

```typescript
// ❌ WITHOUT DI: Hard-coded dependencies
class UsersService {
  private usersRepository: UsersRepository;
  private logger: LoggerService;
  private email: EmailService;

  constructor() {
    // Service creates its own dependencies (TIGHT COUPLING)
    this.usersRepository = new UsersRepository();
    this.logger = new LoggerService();
    this.email = new EmailService();
  }

  async createUser(dto: CreateUserDto): Promise<User> {
    this.logger.log('Creating user');
    const user = await this.usersRepository.save(dto);
    await this.email.sendWelcomeEmail(user.email);
    return user;
  }
}

// Problems:
// ❌ Hard to test: Can't mock UsersRepository, LoggerService, EmailService
// ❌ Tight coupling: UsersService knows how to create dependencies
// ❌ Hard to change: If LoggerService constructor changes, UsersService breaks
// ❌ No flexibility: Can't use different implementations
// ❌ Multiple instances: Every UsersService creates new dependencies

// ✅ WITH DI: Dependencies injected
@Injectable()
class UsersService {
  constructor(
    // NestJS injects dependencies automatically (LOOSE COUPLING)
    private readonly usersRepository: UsersRepository,
    private readonly logger: LoggerService,
    private readonly email: EmailService,
  ) {}

  async createUser(dto: CreateUserDto): Promise<User> {
    this.logger.log('Creating user');
    const user = await this.usersRepository.save(dto);
    await this.email.sendWelcomeEmail(user.email);
    return user;
  }
}

// Benefits:
// ✅ Easy to test: Inject mock dependencies
// ✅ Loose coupling: UsersService doesn't create dependencies
// ✅ Flexible: Change implementations without changing UsersService
// ✅ Single instance: NestJS manages lifecycle (singleton by default)
// ✅ Maintainable: Dependencies declared in constructor
```

---

### **Method 1: Constructor Injection (Recommended)**

```typescript
// ========== CONSTRUCTOR INJECTION ==========
// users/services/users.service.ts
import { Injectable } from '@nestjs/common';

@Injectable()  // Mark as injectable
export class UsersService {
  constructor(
    // Declare dependencies in constructor
    private readonly usersRepository: UsersRepository,
    private readonly logger: LoggerService,
    private readonly email: EmailService,
  ) {}
  // NestJS automatically injects instances when creating UsersService

  async create(dto: CreateUserDto): Promise<User> {
    // Use injected dependencies
    this.logger.log('Creating user', 'UsersService');
    const user = await this.usersRepository.save(dto);
    await this.email.sendWelcomeEmail(user.email);
    return user;
  }
}

// users/users.module.ts
@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UsersController],
  providers: [
    UsersService,       // Register service
    UsersRepository,    // Register repository
    LoggerService,      // Register logger
    EmailService,       // Register email
  ],
  exports: [UsersService],
})
export class UsersModule {}
// NestJS creates dependency graph:
// UsersService needs: UsersRepository, LoggerService, EmailService
// → NestJS creates these first, then injects into UsersService

// users/controllers/users.controller.ts
@Controller('users')
export class UsersController {
  constructor(
    private readonly usersService: UsersService,  // Inject service
  ) {}

  @Post()
  async create(@Body() dto: CreateUserDto): Promise<User> {
    return this.usersService.create(dto);  // Use injected service
  }
}

// Dependency chain:
// UsersController → UsersService → [UsersRepository, LoggerService, EmailService]
// NestJS resolves all dependencies automatically
```

---

### **Method 2: Interface-based DI (Abstract Classes)**

```typescript
// ========== DI WITH INTERFACES (Abstraction) ==========
// users/interfaces/users-repository.interface.ts
export abstract class IUsersRepository {
  abstract findAll(): Promise<User[]>;
  abstract findById(id: string): Promise<User | null>;
  abstract save(user: Partial<User>): Promise<User>;
  abstract remove(id: string): Promise<void>;
}

// users/repositories/typeorm-users.repository.ts
@Injectable()
export class TypeOrmUsersRepository implements IUsersRepository {
  constructor(
    @InjectRepository(User)
    private readonly repository: Repository<User>,
  ) {}

  async findAll(): Promise<User[]> {
    return this.repository.find();
  }

  async findById(id: string): Promise<User | null> {
    return this.repository.findOne({ where: { id } });
  }

  async save(user: Partial<User>): Promise<User> {
    return this.repository.save(user);
  }

  async remove(id: string): Promise<void> {
    await this.repository.delete(id);
  }
}

// users/repositories/in-memory-users.repository.ts
@Injectable()
export class InMemoryUsersRepository implements IUsersRepository {
  private users: User[] = [];

  async findAll(): Promise<User[]> {
    return this.users;
  }

  async findById(id: string): Promise<User | null> {
    return this.users.find(u => u.id === id) || null;
  }

  async save(user: Partial<User>): Promise<User> {
    const newUser = { ...user, id: Math.random().toString() } as User;
    this.users.push(newUser);
    return newUser;
  }

  async remove(id: string): Promise<void> {
    this.users = this.users.filter(u => u.id !== id);
  }
}

// users/services/users.service.ts
@Injectable()
export class UsersService {
  constructor(
    // Inject interface, not concrete class
    @Inject('USERS_REPOSITORY')
    private readonly usersRepository: IUsersRepository,
  ) {}

  async findAll(): Promise<User[]> {
    return this.usersRepository.findAll();
  }
  // UsersService doesn't know if it's using TypeOrm or InMemory!
}

// users/users.module.ts
@Module({
  providers: [
    UsersService,
    {
      provide: 'USERS_REPOSITORY',
      useClass: TypeOrmUsersRepository,  // Use TypeOrm in production
      // useClass: InMemoryUsersRepository,  // Use InMemory in tests
    },
  ],
})
export class UsersModule {}

// Benefits:
// ✅ Swap implementations: Change useClass without changing UsersService
// ✅ Easy testing: Use InMemoryUsersRepository in tests
// ✅ Loose coupling: UsersService depends on interface, not implementation
// ✅ Multiple implementations: Different repos for different use cases
```

---

### **Method 3: Different DI Patterns**

```typescript
// ========== PATTERN 1: Class Provider (Default) ==========
@Module({
  providers: [
    UsersService,  // Shorthand
    // Equivalent to:
    // { provide: UsersService, useClass: UsersService }
  ],
})
export class UsersModule {}

// ========== PATTERN 2: Value Provider ==========
@Module({
  providers: [
    {
      provide: 'CONFIG',
      useValue: {
        apiUrl: 'https://api.example.com',
        timeout: 5000,
      },
    },
  ],
})
export class AppModule {}

@Injectable()
export class ApiService {
  constructor(@Inject('CONFIG') private readonly config: any) {}

  async fetch(): Promise<any> {
    return fetch(this.config.apiUrl);
  }
}

// ========== PATTERN 3: Factory Provider ==========
@Module({
  providers: [
    {
      provide: 'DATABASE_CONNECTION',
      useFactory: async (configService: ConfigService) => {
        const connectionString = configService.get('DATABASE_URL');
        return createConnection(connectionString);
      },
      inject: [ConfigService],  // Factory dependencies
    },
  ],
})
export class DatabaseModule {}

// ========== PATTERN 4: Existing Provider (Alias) ==========
@Module({
  providers: [
    LoggerService,
    {
      provide: 'LOGGER',
      useExisting: LoggerService,  // Alias for LoggerService
    },
  ],
})
export class AppModule {}

// ========== PATTERN 5: Async Provider ==========
@Module({
  providers: [
    {
      provide: 'ASYNC_CONNECTION',
      useFactory: async (): Promise<Connection> => {
        const connection = await createConnection();
        await connection.synchronize();
        return connection;
      },
    },
  ],
})
export class DatabaseModule {}
```

---

### **Method 4: Custom Providers and Tokens**

```typescript
// ========== CUSTOM INJECTION TOKENS ==========
// constants/injection-tokens.ts
export const INJECTION_TOKENS = {
  USERS_REPOSITORY: 'USERS_REPOSITORY',
  LOGGER: 'LOGGER',
  CACHE: 'CACHE',
  EMAIL: 'EMAIL',
  DATABASE: 'DATABASE',
};

// users/users.module.ts
@Module({
  providers: [
    {
      provide: INJECTION_TOKENS.USERS_REPOSITORY,
      useClass: TypeOrmUsersRepository,
    },
    {
      provide: INJECTION_TOKENS.LOGGER,
      useClass: WinstonLogger,
    },
  ],
})
export class UsersModule {}

// users/services/users.service.ts
@Injectable()
export class UsersService {
  constructor(
    @Inject(INJECTION_TOKENS.USERS_REPOSITORY)
    private readonly usersRepository: IUsersRepository,
    @Inject(INJECTION_TOKENS.LOGGER)
    private readonly logger: ILogger,
  ) {}
}

// ========== OPTIONAL DEPENDENCIES ==========
@Injectable()
export class UsersService {
  constructor(
    private readonly usersRepository: UsersRepository,
    @Optional() private readonly cache?: CacheService,  // Optional
  ) {}

  async findById(id: string): Promise<User> {
    // Use cache if available
    if (this.cache) {
      const cached = await this.cache.get(`user:${id}`);
      if (cached) return cached;
    }

    const user = await this.usersRepository.findById(id);

    if (this.cache) {
      await this.cache.set(`user:${id}`, user);
    }

    return user;
  }
}

// ========== MULTIPLE IMPLEMENTATIONS ==========
// Strategy pattern with DI
@Module({
  providers: [
    {
      provide: 'PAYMENT_STRATEGIES',
      useFactory: (
        stripe: StripePaymentStrategy,
        paypal: PaypalPaymentStrategy,
      ) => {
        return { stripe, paypal };
      },
      inject: [StripePaymentStrategy, PaypalPaymentStrategy],
    },
  ],
})
export class PaymentsModule {}

@Injectable()
export class PaymentsService {
  constructor(
    @Inject('PAYMENT_STRATEGIES')
    private readonly strategies: {
      stripe: StripePaymentStrategy;
      paypal: PaypalPaymentStrategy;
    },
  ) {}

  async processPayment(method: string, amount: number): Promise<void> {
    const strategy = this.strategies[method];
    if (!strategy) {
      throw new Error(`Unknown payment method: ${method}`);
    }
    return strategy.process(amount);
  }
}
```

---

### **Method 5: Testing with DI**

```typescript
// ========== EASY TESTING WITH DI ==========
// users/services/users.service.spec.ts
describe('UsersService', () => {
  let service: UsersService;
  let usersRepository: UsersRepository;
  let logger: LoggerService;
  let email: EmailService;

  beforeEach(async () => {
    // Create testing module with MOCK dependencies
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: UsersRepository,
          useValue: {
            findAll: jest.fn().mockResolvedValue([]),
            save: jest.fn().mockResolvedValue({ id: '1', name: 'John' }),
          },
        },
        {
          provide: LoggerService,
          useValue: {
            log: jest.fn(),
            error: jest.fn(),
          },
        },
        {
          provide: EmailService,
          useValue: {
            sendWelcomeEmail: jest.fn().mockResolvedValue(undefined),
          },
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
    usersRepository = module.get<UsersRepository>(UsersRepository);
    logger = module.get<LoggerService>(LoggerService);
    email = module.get<EmailService>(EmailService);
  });

  it('should create user and send welcome email', async () => {
    const dto = { name: 'John', email: 'john@example.com' };
    const result = await service.create(dto);

    expect(result).toEqual({ id: '1', name: 'John' });
    expect(usersRepository.save).toHaveBeenCalledWith(dto);
    expect(logger.log).toHaveBeenCalledWith('Creating user', 'UsersService');
    expect(email.sendWelcomeEmail).toHaveBeenCalledWith('john@example.com');
  });

  // Easy to test! Just inject mocks instead of real dependencies
});

// WITHOUT DI, testing would require:
// ❌ Modifying UsersService to accept dependencies
// ❌ Complex mocking setup
// ❌ Monkey-patching
// ❌ Hard to isolate behavior
```

---

### **Benefits Summary:**

| Benefit | Description | Example |
|---------|-------------|---------|
| **Loose Coupling** | Classes don't create dependencies | UsersService doesn't `new` dependencies |
| **Testability** | Easy to inject mocks | Replace real DB with mock in tests |
| **Flexibility** | Swap implementations | Use InMemory repo in tests, TypeOrm in prod |
| **Maintainability** | Dependencies explicit in constructor | Clear what each class needs |
| **Reusability** | Services can be reused | Same LoggerService in all modules |
| **Single Responsibility** | Class focuses on its logic, not creation | UsersService focuses on user logic |
| **Lifecycle Management** | NestJS manages instances | Singleton by default, no manual management |

---

### **Best Practices:**

```typescript
// ✅ GOOD: Constructor injection
@Injectable()
export class UsersService {
  constructor(private readonly usersRepository: UsersRepository) {}
}

// ❌ BAD: Creating dependencies manually
export class UsersService {
  private usersRepository = new UsersRepository();
}

// ✅ GOOD: Inject interfaces/abstractions
constructor(
  @Inject('REPOSITORY') private readonly repo: IRepository,
) {}

// ❌ BAD: Inject concrete classes when abstraction possible
constructor(private readonly repo: TypeOrmRepository) {}

// ✅ GOOD: Use @Injectable() decorator
@Injectable()
export class UsersService {}

// ❌ BAD: Forget @Injectable()
export class UsersService {}  // Won't work with DI!

// ✅ GOOD: Register providers in module
@Module({ providers: [UsersService, UsersRepository] })

// ❌ BAD: Use service without registering
// UsersService not in providers → Error!

// ✅ GOOD: Use readonly for dependencies
constructor(private readonly service: UsersService) {}

// ❌ BAD: Mutable dependencies
constructor(private service: UsersService) {}
this.service = new UsersService();  // Don't reassign!

// ✅ GOOD: Optional dependencies with @Optional()
constructor(@Optional() private cache?: CacheService) {}

// ❌ BAD: Required dependency that might not exist
constructor(private cache: CacheService) {}  // Fails if not registered
```

**Key Takeaway:**

- **What:** **Dependency Injection (DI)** lets classes receive dependencies from the NestJS IoC container instead of constructing them.
- **How (best practice):** Prefer **constructor injection** with `@Injectable()` and depend on abstractions when useful (e.g., `@Inject('TOKEN')`, interface-based DI, factory providers).
- **Why:** Enables **loose coupling**, **easy testing** (inject mocks), **flexibility** (swap implementations), **maintainability** (explicit dependencies), and automatic **lifecycle management** (singletons by default). DI underpins SOLID design in NestJS.

</details>

<details>
<summary><strong>11. What is Repository Pattern and when to use it?</strong></summary>

**Answer:**

**Repository Pattern:**

- **What:** Abstracts data-access from business logic via a repository class (collection-like API) that handles queries, mapping, and persistence.
- **Where it lives:** Sits between the **Service layer** and the **Database** — services call repository methods instead of raw ORM queries.
- **When to use:** When you need **database abstraction**, want to **swap implementations** (TypeORM → Prisma), require **testability** (mock repository), or need to centralize **complex queries/transactions**.

---

### **Without vs With Repository:**

```typescript
// ❌ WITHOUT REPOSITORY: Service directly uses TypeORM
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,  // Direct TypeORM dependency
  ) {}

  async findAll(): Promise<User[]> {
    // Business logic mixed with database queries
    return this.userRepository.find({
      where: { isActive: true },
      relations: ['profile', 'orders'],
      order: { createdAt: 'DESC' },
    });
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.userRepository.findOne({
      where: { email },
      relations: ['profile'],
    });
  }
}

// Problems:
// ❌ Tight coupling to TypeORM
// ❌ Hard to test (need real database)
// ❌ Query logic in service (violates SoC)
// ❌ Hard to swap ORM (TypeORM → Prisma requires service changes)
// ❌ Duplicate queries across services

// ✅ WITH REPOSITORY: Abstraction layer
@Injectable()
export class UsersRepository {
  constructor(
    @InjectRepository(User)
    private readonly repository: Repository<User>,
  ) {}

  async findAll(): Promise<User[]> {
    return this.repository.find({
      where: { isActive: true },
      relations: ['profile', 'orders'],
      order: { createdAt: 'DESC' },
    });
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.repository.findOne({
      where: { email },
      relations: ['profile'],
    });
  }
}

@Injectable()
export class UsersService {
  constructor(
    private readonly usersRepository: UsersRepository,  // Repository abstraction
  ) {}

  async findAll(): Promise<User[]> {
    return this.usersRepository.findAll();  // Clean, simple
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.usersRepository.findByEmail(email);
  }
}

// Benefits:
// ✅ Loose coupling (service doesn't know about TypeORM)
// ✅ Easy to test (mock repository)
// ✅ Separation of concerns (queries in repository)
// ✅ Reusable queries
// ✅ Easy to swap ORM (change only repository)
```

---

### **Method 1: Basic Repository Pattern**

```typescript
// ========== BASIC REPOSITORY ==========
// users/repositories/users.repository.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from '../entities/user.entity';

@Injectable()
export class UsersRepository {
  constructor(
    @InjectRepository(User)
    private readonly repository: Repository<User>,
  ) {}

  // CREATE
  async save(user: Partial<User>): Promise<User> {
    return this.repository.save(user);
  }

  async saveMany(users: Partial<User>[]): Promise<User[]> {
    return this.repository.save(users);
  }

  // READ
  async findAll(): Promise<User[]> {
    return this.repository.find({
      where: { isActive: true },
      order: { createdAt: 'DESC' },
    });
  }

  async findById(id: string): Promise<User | null> {
    return this.repository.findOne({ where: { id } });
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.repository.findOne({ where: { email } });
  }

  async findByIds(ids: string[]): Promise<User[]> {
    return this.repository.findByIds(ids);
  }

  // UPDATE
  async update(id: string, data: Partial<User>): Promise<User> {
    await this.repository.update(id, data);
    return this.findById(id);
  }

  // DELETE
  async remove(id: string): Promise<void> {
    await this.repository.delete(id);
  }

  async softDelete(id: string): Promise<void> {
    await this.repository.softDelete(id);
  }

  // COUNT
  async count(): Promise<number> {
    return this.repository.count({ where: { isActive: true } });
  }

  // EXISTS
  async exists(id: string): Promise<boolean> {
    const count = await this.repository.count({ where: { id } });
    return count > 0;
  }
}

// users/services/users.service.ts
@Injectable()
export class UsersService {
  constructor(private readonly usersRepository: UsersRepository) {}

  async findAll(): Promise<User[]> {
    return this.usersRepository.findAll();
  }

  async findById(id: string): Promise<User> {
    const user = await this.usersRepository.findById(id);
    if (!user) {
      throw new NotFoundException(`User ${id} not found`);
    }
    return user;
  }

  async create(dto: CreateUserDto): Promise<User> {
    const existingUser = await this.usersRepository.findByEmail(dto.email);
    if (existingUser) {
      throw new ConflictException('Email already exists');
    }
    return this.usersRepository.save(dto);
  }

  async update(id: string, dto: UpdateUserDto): Promise<User> {
    await this.findById(id);  // Check exists
    return this.usersRepository.update(id, dto);
  }

  async remove(id: string): Promise<void> {
    await this.findById(id);  // Check exists
    return this.usersRepository.remove(id);
  }
}

// Service focuses on business logic, repository handles data access
```

---

### **Method 2: Generic Repository Pattern**

```typescript
// ========== GENERIC REPOSITORY BASE ==========
// common/repositories/base.repository.ts
import { Repository, FindOptionsWhere, FindManyOptions } from 'typeorm';

export abstract class BaseRepository<T> {
  constructor(protected readonly repository: Repository<T>) {}

  async save(entity: Partial<T>): Promise<T> {
    return this.repository.save(entity as any);
  }

  async saveMany(entities: Partial<T>[]): Promise<T[]> {
    return this.repository.save(entities as any);
  }

  async findAll(options?: FindManyOptions<T>): Promise<T[]> {
    return this.repository.find(options);
  }

  async findOne(options: FindOptionsWhere<T>): Promise<T | null> {
    return this.repository.findOne({ where: options });
  }

  async findById(id: string | number): Promise<T | null> {
    return this.repository.findOne({ where: { id } as any });
  }

  async update(id: string | number, data: Partial<T>): Promise<void> {
    await this.repository.update(id, data as any);
  }

  async remove(id: string | number): Promise<void> {
    await this.repository.delete(id);
  }

  async count(options?: FindManyOptions<T>): Promise<number> {
    return this.repository.count(options);
  }

  async exists(id: string | number): Promise<boolean> {
    const count = await this.repository.count({ where: { id } as any });
    return count > 0;
  }
}

// users/repositories/users.repository.ts
@Injectable()
export class UsersRepository extends BaseRepository<User> {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {
    super(userRepository);
  }

  // Add user-specific methods
  async findByEmail(email: string): Promise<User | null> {
    return this.userRepository.findOne({ where: { email } });
  }

  async findActiveUsers(): Promise<User[]> {
    return this.userRepository.find({
      where: { isActive: true },
      order: { createdAt: 'DESC' },
    });
  }

  async findWithOrders(userId: string): Promise<User | null> {
    return this.userRepository.findOne({
      where: { id: userId },
      relations: ['orders'],
    });
  }

  async searchByName(name: string): Promise<User[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .where('user.name ILIKE :name', { name: `%${name}%` })
      .getMany();
  }
}

// orders/repositories/orders.repository.ts
@Injectable()
export class OrdersRepository extends BaseRepository<Order> {
  constructor(
    @InjectRepository(Order)
    private readonly orderRepository: Repository<Order>,
  ) {
    super(orderRepository);
  }

  // Add order-specific methods
  async findByUserId(userId: string): Promise<Order[]> {
    return this.orderRepository.find({
      where: { userId },
      order: { createdAt: 'DESC' },
    });
  }

  async findPendingOrders(): Promise<Order[]> {
    return this.orderRepository.find({
      where: { status: 'pending' },
    });
  }
}

// Benefits:
// ✅ Reusable base methods (save, find, update, delete)
// ✅ Consistent API across all repositories
// ✅ Each repository can add domain-specific methods
// ✅ Less boilerplate
```

---

### **Method 3: Interface-based Repository (Testability)**

```typescript
// ========== INTERFACE-BASED REPOSITORY ==========
// users/interfaces/users-repository.interface.ts
export abstract class IUsersRepository {
  abstract findAll(): Promise<User[]>;
  abstract findById(id: string): Promise<User | null>;
  abstract findByEmail(email: string): Promise<User | null>;
  abstract save(user: Partial<User>): Promise<User>;
  abstract update(id: string, data: Partial<User>): Promise<User>;
  abstract remove(id: string): Promise<void>;
}

// users/repositories/typeorm-users.repository.ts
@Injectable()
export class TypeOrmUsersRepository implements IUsersRepository {
  constructor(
    @InjectRepository(User)
    private readonly repository: Repository<User>,
  ) {}

  async findAll(): Promise<User[]> {
    return this.repository.find();
  }

  async findById(id: string): Promise<User | null> {
    return this.repository.findOne({ where: { id } });
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.repository.findOne({ where: { email } });
  }

  async save(user: Partial<User>): Promise<User> {
    return this.repository.save(user);
  }

  async update(id: string, data: Partial<User>): Promise<User> {
    await this.repository.update(id, data);
    return this.findById(id);
  }

  async remove(id: string): Promise<void> {
    await this.repository.delete(id);
  }
}

// users/repositories/in-memory-users.repository.ts
@Injectable()
export class InMemoryUsersRepository implements IUsersRepository {
  private users: User[] = [];

  async findAll(): Promise<User[]> {
    return this.users;
  }

  async findById(id: string): Promise<User | null> {
    return this.users.find(u => u.id === id) || null;
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.users.find(u => u.email === email) || null;
  }

  async save(user: Partial<User>): Promise<User> {
    const newUser = { ...user, id: Math.random().toString() } as User;
    this.users.push(newUser);
    return newUser;
  }

  async update(id: string, data: Partial<User>): Promise<User> {
    const index = this.users.findIndex(u => u.id === id);
    this.users[index] = { ...this.users[index], ...data };
    return this.users[index];
  }

  async remove(id: string): Promise<void> {
    this.users = this.users.filter(u => u.id !== id);
  }
}

// users/users.module.ts
@Module({
  providers: [
    UsersService,
    {
      provide: IUsersRepository,
      useClass: TypeOrmUsersRepository,  // Production
      // useClass: InMemoryUsersRepository,  // Testing
    },
  ],
})
export class UsersModule {}

// users/services/users.service.ts
@Injectable()
export class UsersService {
  constructor(
    private readonly usersRepository: IUsersRepository,  // Interface!
  ) {}

  async findAll(): Promise<User[]> {
    return this.usersRepository.findAll();
  }
}

// users/services/users.service.spec.ts
describe('UsersService', () => {
  let service: UsersService;
  let repository: IUsersRepository;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: IUsersRepository,
          useClass: InMemoryUsersRepository,  // Easy testing!
        },
      ],
    }).compile();

    service = module.get(UsersService);
    repository = module.get(IUsersRepository);
  });

  it('should find all users', async () => {
    const users = await service.findAll();
    expect(users).toEqual([]);
  });
});

// Benefits:
// ✅ Easy to swap implementations (TypeORM → Prisma)
// ✅ Easy testing (use InMemory in tests)
// ✅ Service depends on interface, not implementation
```

---

### **Method 4: Complex Queries in Repository**

```typescript
// ========== COMPLEX QUERIES ==========
// users/repositories/users.repository.ts
@Injectable()
export class UsersRepository {
  constructor(
    @InjectRepository(User)
    private readonly repository: Repository<User>,
  ) {}

  // Simple query
  async findById(id: string): Promise<User | null> {
    return this.repository.findOne({ where: { id } });
  }

  // Query with relations
  async findByIdWithOrders(id: string): Promise<User | null> {
    return this.repository.findOne({
      where: { id },
      relations: ['orders', 'profile'],
    });
  }

  // Query builder for complex logic
  async findActiveUsersWithRecentOrders(): Promise<User[]> {
    const thirtyDaysAgo = new Date();
    thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);

    return this.repository
      .createQueryBuilder('user')
      .leftJoinAndSelect('user.orders', 'order')
      .where('user.isActive = :isActive', { isActive: true })
      .andWhere('order.createdAt >= :date', { date: thirtyDaysAgo })
      .orderBy('user.createdAt', 'DESC')
      .getMany();
  }

  // Aggregation query
  async getUserStatistics(): Promise<any> {
    return this.repository
      .createQueryBuilder('user')
      .leftJoin('user.orders', 'order')
      .select('user.id', 'userId')
      .addSelect('user.name', 'userName')
      .addSelect('COUNT(order.id)', 'orderCount')
      .addSelect('SUM(order.total)', 'totalSpent')
      .groupBy('user.id')
      .having('COUNT(order.id) > :minOrders', { minOrders: 5 })
      .getRawMany();
  }

  // Search query
  async searchUsers(query: string): Promise<User[]> {
    return this.repository
      .createQueryBuilder('user')
      .where('user.name ILIKE :query', { query: `%${query}%` })
      .orWhere('user.email ILIKE :query', { query: `%${query}%` })
      .getMany();
  }

  // Pagination
  async findWithPagination(page: number, limit: number): Promise<[User[], number]> {
    return this.repository.findAndCount({
      skip: (page - 1) * limit,
      take: limit,
      order: { createdAt: 'DESC' },
    });
  }

  // Raw query for very complex cases
  async findByCustomQuery(params: any): Promise<User[]> {
    return this.repository.query(
      `SELECT * FROM users WHERE custom_condition = $1`,
      [params.condition],
    );
  }
}

// Service stays clean - all queries in repository
@Injectable()
export class UsersService {
  constructor(private readonly usersRepository: UsersRepository) {}

  async getActiveUsersWithRecentOrders(): Promise<User[]> {
    return this.usersRepository.findActiveUsersWithRecentOrders();
  }

  async getUserStats(): Promise<any> {
    return this.usersRepository.getUserStatistics();
  }

  async search(query: string): Promise<User[]> {
    return this.usersRepository.searchUsers(query);
  }

  async getPaginated(page: number, limit: number): Promise<any> {
    const [users, total] = await this.usersRepository.findWithPagination(page, limit);
    return {
      users,
      total,
      page,
      pageCount: Math.ceil(total / limit),
    };
  }
}

// All database logic centralized in repository!
```

---

### **Method 5: Transaction Support in Repository**

```typescript
// ========== TRANSACTIONS IN REPOSITORY ==========
// users/repositories/users.repository.ts
@Injectable()
export class UsersRepository {
  constructor(
    @InjectRepository(User)
    private readonly repository: Repository<User>,
    private readonly connection: Connection,
  ) {}

  // Simple save
  async save(user: Partial<User>): Promise<User> {
    return this.repository.save(user);
  }

  // Transaction: Create user with profile
  async createUserWithProfile(
    userData: Partial<User>,
    profileData: Partial<UserProfile>,
  ): Promise<User> {
    return this.connection.transaction(async (manager) => {
      // Save user
      const user = await manager.save(User, userData);

      // Save profile with user reference
      const profile = await manager.save(UserProfile, {
        ...profileData,
        userId: user.id,
      });

      // Return user with profile
      return manager.findOne(User, {
        where: { id: user.id },
        relations: ['profile'],
      });
    });
  }

  // Transaction: Transfer operation
  async transferUserData(fromId: string, toId: string): Promise<void> {
    await this.connection.transaction(async (manager) => {
      const fromUser = await manager.findOne(User, { where: { id: fromId } });
      const toUser = await manager.findOne(User, { where: { id: toId } });

      if (!fromUser || !toUser) {
        throw new NotFoundException('User not found');
      }

      // Transfer logic here...
      await manager.save(User, fromUser);
      await manager.save(User, toUser);
    });
  }
}

// users/services/users.service.ts
@Injectable()
export class UsersService {
  constructor(private readonly usersRepository: UsersRepository) {}

  async createUserWithProfile(
    userDto: CreateUserDto,
    profileDto: CreateProfileDto,
  ): Promise<User> {
    // Service doesn't know about transactions - repository handles it
    return this.usersRepository.createUserWithProfile(userDto, profileDto);
  }
}

// Repository encapsulates transaction logic
```

---

### **When to Use Repository Pattern:**

| Use Case | Repository | Direct ORM |
|----------|-----------|------------|
| **Simple CRUD** | Optional | ✅ Direct ORM OK |
| **Complex queries** | ✅ Use Repository | ❌ Gets messy |
| **Need testability** | ✅ Use Repository | ❌ Hard to mock |
| **Multiple data sources** | ✅ Use Repository | ❌ Tight coupling |
| **Want ORM flexibility** | ✅ Use Repository | ❌ Locked in |
| **Prototype/MVP** | Optional | ✅ Direct ORM faster |
| **Large app** | ✅ Use Repository | ❌ Maintainability issues |
| **Microservices** | ✅ Use Repository | ❌ Hard to swap |

---

### **Best Practices:**

```typescript
// ✅ GOOD: Repository handles data access only
@Injectable()
export class UsersRepository {
  async findAll(): Promise<User[]> {
    return this.repository.find();
  }
}

// ❌ BAD: Repository contains business logic
@Injectable()
export class UsersRepository {
  async createUserAndSendEmail(dto: CreateUserDto): Promise<User> {
    const user = await this.repository.save(dto);
    await this.emailService.send(user.email);  // Business logic!
    return user;
  }
}

// ✅ GOOD: Service uses repository
@Injectable()
export class UsersService {
  constructor(private readonly usersRepository: UsersRepository) {}
  
  async create(dto: CreateUserDto): Promise<User> {
    const user = await this.usersRepository.save(dto);
    await this.emailService.send(user.email);  // Business logic in service
    return user;
  }
}

// ✅ GOOD: Repository methods are domain-focused
async findActiveUsers(): Promise<User[]>
async findByEmail(email: string): Promise<User | null>

// ❌ BAD: Repository methods expose ORM details
async findWithTypeOrmQueryBuilder(): Promise<User[]>
async executeRawSQL(sql: string): Promise<any>

// ✅ GOOD: One repository per entity/aggregate
UsersRepository → User entity
OrdersRepository → Order entity

// ❌ BAD: Generic repository for everything
GenericRepository<T> → All entities (loses domain specificity)

// ✅ GOOD: Repository returns domain entities
async findById(id: string): Promise<User>

// ❌ BAD: Repository returns raw data
async findById(id: string): Promise<any>
```

**Key Takeaway:**

- **What & where:** Repository Pattern provides a **collection-like interface** for domain entities and sits between the service layer and the database.
- **Benefits:** Enables **database abstraction** (swap TypeORM → Prisma), **testability** (mock repositories), **separation of concerns**, **reusable queries**, and centralized data access for complex queries/transactions.
- **Implementation & when to use:** Implement as an `@Injectable()` repository class (or extend a `BaseRepository`), prefer interface-based providers for flexibility, and use repositories for complex, testable, or multi-ORM apps — avoid for trivial single-entity CRUD prototypes.

</details>

<details>
<summary><strong>12. What is Factory Pattern and how is it used in NestJS?</strong></summary>

**Answer:**

**Factory Pattern** creates objects without specifying exact classes, delegating instantiation logic to factory method/class. In NestJS, use **factory providers** (`useFactory`) to create instances dynamically based on configuration, environment, or runtime conditions. Common uses: **database connections** (different configs per env), **strategy selection** (payment methods), **complex initialization** (async setup), and **conditional providers** (feature flags).

---

### **Factory Pattern Concepts:**

```typescript
// ❌ WITHOUT FACTORY: Hard-coded instantiation
class PaymentService {
  processPayment(method: string, amount: number): void {
    // Tight coupling - must know all payment classes
    if (method === 'stripe') {
      const stripe = new StripePayment();
      stripe.charge(amount);
    } else if (method === 'paypal') {
      const paypal = new PaypalPayment();
      paypal.charge(amount);
    } else if (method === 'square') {
      const square = new SquarePayment();
      square.charge(amount);
    }
    // Adding new method requires changing this code!
  }
}

// ✅ WITH FACTORY: Dynamic instantiation
interface PaymentStrategy {
  charge(amount: number): Promise<void>;
}

class PaymentFactory {
  private strategies: Map<string, PaymentStrategy> = new Map();

  register(method: string, strategy: PaymentStrategy): void {
    this.strategies.set(method, strategy);
  }

  create(method: string): PaymentStrategy {
    const strategy = this.strategies.get(method);
    if (!strategy) {
      throw new Error(`Unknown payment method: ${method}`);
    }
    return strategy;
  }
}

class PaymentService {
  constructor(private readonly factory: PaymentFactory) {}

  processPayment(method: string, amount: number): Promise<void> {
    const strategy = this.factory.create(method);
    return strategy.charge(amount);
  }
}

// Benefits:
// ✅ Loose coupling (PaymentService doesn't know payment classes)
// ✅ Easy to extend (register new payment methods)
// ✅ Single place for object creation
// ✅ Testable (mock factory)
```

---

### **Method 1: Factory Provider in NestJS**

```typescript
// ========== FACTORY PROVIDER: Database Connection ==========
// database/database.module.ts
import { Module } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { createConnection, Connection } from 'typeorm';

@Module({
  providers: [
    {
      provide: 'DATABASE_CONNECTION',
      useFactory: async (configService: ConfigService): Promise<Connection> => {
        // Factory creates connection based on environment
        const env = configService.get('NODE_ENV');
        const config = {
          type: 'postgres',
          host: configService.get('DB_HOST'),
          port: configService.get('DB_PORT'),
          username: configService.get('DB_USERNAME'),
          password: configService.get('DB_PASSWORD'),
          database: configService.get('DB_DATABASE'),
          entities: [__dirname + '/../**/*.entity{.ts,.js}'],
          synchronize: env === 'development',  // Only in dev
          logging: env === 'development',
        };

        return createConnection(config);
      },
      inject: [ConfigService],  // Inject dependencies into factory
    },
  ],
  exports: ['DATABASE_CONNECTION'],
})
export class DatabaseModule {}

// Usage in service
@Injectable()
export class UsersService {
  constructor(
    @Inject('DATABASE_CONNECTION')
    private readonly connection: Connection,
  ) {}

  async findAll(): Promise<User[]> {
    return this.connection.getRepository(User).find();
  }
}

// Factory creates different connections based on config!
```

---

### **Method 2: Strategy Factory Pattern**

```typescript
// ========== STRATEGY FACTORY: Payment Processing ==========
// payments/strategies/payment-strategy.interface.ts
export interface PaymentStrategy {
  charge(amount: number, metadata: any): Promise<PaymentResult>;
  refund(transactionId: string): Promise<void>;
}

// payments/strategies/stripe-payment.strategy.ts
@Injectable()
export class StripePaymentStrategy implements PaymentStrategy {
  async charge(amount: number, metadata: any): Promise<PaymentResult> {
    console.log(`Stripe: Charging $${amount}`);
    // Stripe API integration
    return { transactionId: 'stripe_123', status: 'success' };
  }

  async refund(transactionId: string): Promise<void> {
    console.log(`Stripe: Refunding ${transactionId}`);
  }
}

// payments/strategies/paypal-payment.strategy.ts
@Injectable()
export class PaypalPaymentStrategy implements PaymentStrategy {
  async charge(amount: number, metadata: any): Promise<PaymentResult> {
    console.log(`PayPal: Charging $${amount}`);
    // PayPal API integration
    return { transactionId: 'paypal_456', status: 'success' };
  }

  async refund(transactionId: string): Promise<void> {
    console.log(`PayPal: Refunding ${transactionId}`);
  }
}

// payments/factories/payment-strategy.factory.ts
@Injectable()
export class PaymentStrategyFactory {
  constructor(
    private readonly stripeStrategy: StripePaymentStrategy,
    private readonly paypalStrategy: PaypalPaymentStrategy,
  ) {}

  create(method: string): PaymentStrategy {
    switch (method) {
      case 'stripe':
        return this.stripeStrategy;
      case 'paypal':
        return this.paypalStrategy;
      default:
        throw new BadRequestException(`Unknown payment method: ${method}`);
    }
  }
}

// OR: Use factory provider with Map
@Module({
  providers: [
    StripePaymentStrategy,
    PaypalPaymentStrategy,
    {
      provide: 'PAYMENT_STRATEGIES',
      useFactory: (
        stripe: StripePaymentStrategy,
        paypal: PaypalPaymentStrategy,
      ) => {
        const strategies = new Map<string, PaymentStrategy>();
        strategies.set('stripe', stripe);
        strategies.set('paypal', paypal);
        return strategies;
      },
      inject: [StripePaymentStrategy, PaypalPaymentStrategy],
    },
  ],
})
export class PaymentsModule {}

// payments/services/payments.service.ts
@Injectable()
export class PaymentsService {
  constructor(
    @Inject('PAYMENT_STRATEGIES')
    private readonly strategies: Map<string, PaymentStrategy>,
  ) {}

  async processPayment(
    method: string,
    amount: number,
    metadata: any,
  ): Promise<PaymentResult> {
    const strategy = this.strategies.get(method);
    if (!strategy) {
      throw new BadRequestException(`Unsupported payment method: ${method}`);
    }

    return strategy.charge(amount, metadata);
  }

  async refund(method: string, transactionId: string): Promise<void> {
    const strategy = this.strategies.get(method);
    if (!strategy) {
      throw new BadRequestException(`Unsupported payment method: ${method}`);
    }

    return strategy.refund(transactionId);
  }
}

// payments/controllers/payments.controller.ts
@Controller('payments')
export class PaymentsController {
  constructor(private readonly paymentsService: PaymentsService) {}

  @Post('charge')
  async charge(@Body() dto: ChargeDto): Promise<PaymentResult> {
    return this.paymentsService.processPayment(
      dto.method,  // 'stripe' or 'paypal'
      dto.amount,
      dto.metadata,
    );
  }
}

// Factory dynamically selects strategy!
```

---

### **Method 3: Dynamic Module Factory**

```typescript
// ========== DYNAMIC MODULE FACTORY ==========
// logger/logger.module.ts
import { Module, DynamicModule } from '@nestjs/common';
import { LoggerService } from './logger.service';

export interface LoggerModuleOptions {
  level: 'debug' | 'info' | 'warn' | 'error';
  format: 'json' | 'text';
  transports: string[];
}

@Module({})
export class LoggerModule {
  // Factory method to create dynamic module
  static forRoot(options: LoggerModuleOptions): DynamicModule {
    return {
      module: LoggerModule,
      providers: [
        {
          provide: 'LOGGER_OPTIONS',
          useValue: options,
        },
        LoggerService,
      ],
      exports: [LoggerService],
    };
  }

  // Factory method for async configuration
  static forRootAsync(options: {
    useFactory: (...args: any[]) => Promise<LoggerModuleOptions>;
    inject?: any[];
  }): DynamicModule {
    return {
      module: LoggerModule,
      providers: [
        {
          provide: 'LOGGER_OPTIONS',
          useFactory: options.useFactory,
          inject: options.inject || [],
        },
        LoggerService,
      ],
      exports: [LoggerService],
    };
  }
}

// logger/logger.service.ts
@Injectable()
export class LoggerService {
  constructor(
    @Inject('LOGGER_OPTIONS')
    private readonly options: LoggerModuleOptions,
  ) {
    console.log('Logger initialized with:', options);
  }

  log(message: string): void {
    if (this.options.level === 'info' || this.options.level === 'debug') {
      console.log(`[${this.options.format}] ${message}`);
    }
  }

  error(message: string): void {
    console.error(`[${this.options.format}] ERROR: ${message}`);
  }
}

// app.module.ts - Synchronous factory
@Module({
  imports: [
    LoggerModule.forRoot({
      level: 'debug',
      format: 'json',
      transports: ['console', 'file'],
    }),
  ],
})
export class AppModule {}

// OR: Asynchronous factory with config service
@Module({
  imports: [
    LoggerModule.forRootAsync({
      useFactory: (configService: ConfigService) => ({
        level: configService.get('LOG_LEVEL'),
        format: configService.get('LOG_FORMAT'),
        transports: configService.get('LOG_TRANSPORTS').split(','),
      }),
      inject: [ConfigService],
    }),
  ],
})
export class AppModule {}

// Factory creates module with different configurations!
```

---

### **Method 4: Abstract Factory Pattern**

```typescript
// ========== ABSTRACT FACTORY: Multiple Related Objects ==========
// notifications/factories/notification-factory.interface.ts
export interface NotificationFactory {
  createSender(): NotificationSender;
  createFormatter(): NotificationFormatter;
  createLogger(): NotificationLogger;
}

export interface NotificationSender {
  send(message: string, recipient: string): Promise<void>;
}

export interface NotificationFormatter {
  format(template: string, data: any): string;
}

export interface NotificationLogger {
  log(message: string): void;
}

// notifications/factories/email-notification.factory.ts
@Injectable()
export class EmailNotificationFactory implements NotificationFactory {
  createSender(): NotificationSender {
    return {
      async send(message: string, recipient: string): Promise<void> {
        console.log(`Email to ${recipient}: ${message}`);
        // Email sending logic
      },
    };
  }

  createFormatter(): NotificationFormatter {
    return {
      format(template: string, data: any): string {
        // HTML email formatting
        return `<html><body>${template}</body></html>`;
      },
    };
  }

  createLogger(): NotificationLogger {
    return {
      log(message: string): void {
        console.log(`[Email] ${message}`);
      },
    };
  }
}

// notifications/factories/sms-notification.factory.ts
@Injectable()
export class SmsNotificationFactory implements NotificationFactory {
  createSender(): NotificationSender {
    return {
      async send(message: string, recipient: string): Promise<void> {
        console.log(`SMS to ${recipient}: ${message}`);
        // SMS sending logic
      },
    };
  }

  createFormatter(): NotificationFormatter {
    return {
      format(template: string, data: any): string {
        // Plain text SMS formatting (160 chars max)
        return template.substring(0, 160);
      },
    };
  }

  createLogger(): NotificationLogger {
    return {
      log(message: string): void {
        console.log(`[SMS] ${message}`);
      },
    };
  }
}

// notifications/notifications.module.ts
@Module({
  providers: [
    EmailNotificationFactory,
    SmsNotificationFactory,
    {
      provide: 'NOTIFICATION_FACTORIES',
      useFactory: (
        emailFactory: EmailNotificationFactory,
        smsFactory: SmsNotificationFactory,
      ) => {
        const factories = new Map<string, NotificationFactory>();
        factories.set('email', emailFactory);
        factories.set('sms', smsFactory);
        return factories;
      },
      inject: [EmailNotificationFactory, SmsNotificationFactory],
    },
  ],
})
export class NotificationsModule {}

// notifications/services/notifications.service.ts
@Injectable()
export class NotificationsService {
  constructor(
    @Inject('NOTIFICATION_FACTORIES')
    private readonly factories: Map<string, NotificationFactory>,
  ) {}

  async sendNotification(
    channel: string,
    template: string,
    data: any,
    recipient: string,
  ): Promise<void> {
    const factory = this.factories.get(channel);
    if (!factory) {
      throw new BadRequestException(`Unknown channel: ${channel}`);
    }

    // Factory creates all related objects
    const sender = factory.createSender();
    const formatter = factory.createFormatter();
    const logger = factory.createLogger();

    const message = formatter.format(template, data);
    logger.log(`Sending ${channel} notification to ${recipient}`);
    await sender.send(message, recipient);
  }
}

// Factory creates family of related objects!
```

---

### **Method 5: Conditional Factory (Feature Flags)**

```typescript
// ========== CONDITIONAL FACTORY: Feature Flags ==========
// features/features.module.ts
@Module({
  providers: [
    {
      provide: 'CACHE_SERVICE',
      useFactory: (configService: ConfigService) => {
        const cacheEnabled = configService.get('CACHE_ENABLED');
        
        if (cacheEnabled === 'true') {
          // Use Redis cache
          return new RedisCacheService(
            configService.get('REDIS_HOST'),
            configService.get('REDIS_PORT'),
          );
        } else {
          // Use in-memory cache
          return new InMemoryCacheService();
        }
      },
      inject: [ConfigService],
    },
    {
      provide: 'ANALYTICS_SERVICE',
      useFactory: (configService: ConfigService) => {
        const analyticsProvider = configService.get('ANALYTICS_PROVIDER');
        
        switch (analyticsProvider) {
          case 'google':
            return new GoogleAnalyticsService(configService.get('GA_TRACKING_ID'));
          case 'mixpanel':
            return new MixpanelAnalyticsService(configService.get('MIXPANEL_TOKEN'));
          case 'segment':
            return new SegmentAnalyticsService(configService.get('SEGMENT_KEY'));
          default:
            return new NoOpAnalyticsService();  // Disabled
        }
      },
      inject: [ConfigService],
    },
  ],
})
export class FeaturesModule {}

// Factory selects implementation based on configuration!
```

---

### **Benefits of Factory Pattern:**

| Benefit | Description | Example |
|---------|-------------|---------|
| **Decoupling** | Client doesn't know concrete classes | PaymentService doesn't know Stripe/PayPal |
| **Flexibility** | Easy to add new implementations | Add new payment method without changing service |
| **Testability** | Mock factory in tests | Return mock strategy |
| **Configuration** | Create objects based on config | Different DB per environment |
| **Conditional Logic** | Centralize creation logic | One place for feature flags |
| **Complex Initialization** | Handle async setup | Database connection pooling |

---

### **Best Practices:**

```typescript
// ✅ GOOD: Factory provider for complex initialization
{
  provide: 'DATABASE',
  useFactory: async (config: ConfigService) => {
    return createConnection(config.get('DB_URL'));
  },
  inject: [ConfigService],
}

// ❌ BAD: Hard-coded instantiation
const database = new Database('localhost:5432');

// ✅ GOOD: Factory returns interface
create(method: string): PaymentStrategy

// ❌ BAD: Factory returns concrete class
create(method: string): StripePayment | PaypalPayment

// ✅ GOOD: Factory handles all creation logic
const strategy = factory.create('stripe');
await strategy.charge(100);

// ❌ BAD: Client has conditional logic
if (method === 'stripe') {
  const stripe = new StripePayment();
  await stripe.charge(100);
}

// ✅ GOOD: Use Map for strategy registry
const strategies = new Map<string, Strategy>();
strategies.set('stripe', stripeStrategy);

// ❌ BAD: Switch statement everywhere
switch (method) {
  case 'stripe': return new StripeStrategy();
  case 'paypal': return new PaypalStrategy();
}

// ✅ GOOD: Dynamic module with factory
LoggerModule.forRoot(options)

// ❌ BAD: Hard-coded module
@Module({ providers: [LoggerService] })
```

**Key Takeaway:** **Factory Pattern** creates objects dynamically without specifying exact classes, delegating instantiation to factory method. In NestJS, use **factory providers** (`useFactory`) for: **dynamic configuration** (database connection per env), **strategy selection** (payment methods, notification channels), **complex initialization** (async setup, pooling), **conditional logic** (feature flags), and **abstract factories** (creating families of related objects). Benefits: **loose coupling** (client doesn't know concrete classes), **flexibility** (easy to add implementations), **testability** (mock factory), **centralized creation logic**. Implement with `useFactory` in providers, inject dependencies via `inject: []`, and return different implementations based on configuration or runtime conditions.

</details>

<details>
<summary><strong>13. What is Strategy Pattern (e.g., Passport strategies)?</strong></summary>

**Answer:**

**Strategy Pattern** defines a family of **interchangeable algorithms** encapsulated in separate classes with a common interface. Client selects strategy at runtime without knowing implementation details. In NestJS, common uses: **Passport authentication strategies** (Local, JWT, OAuth), **payment processing** (Stripe, PayPal, Square), **notification channels** (Email, SMS, Push), **file storage** (S3, Google Cloud, Local), and **data validation** (different rules per context).

---

### **Strategy Pattern Concept:**

```typescript
// ❌ WITHOUT STRATEGY: Conditional logic everywhere
class PaymentService {
  processPayment(method: string, amount: number): void {
    if (method === 'stripe') {
      // Stripe-specific logic
      console.log('Processing via Stripe');
      // ... 50 lines of Stripe code
    } else if (method === 'paypal') {
      // PayPal-specific logic
      console.log('Processing via PayPal');
      // ... 50 lines of PayPal code
    } else if (method === 'square') {
      // Square-specific logic
      console.log('Processing via Square');
      // ... 50 lines of Square code
    }
    // Adding new method requires modifying this class (violates Open-Closed Principle)
  }
}

// Problems:
// ❌ Violates Open-Closed Principle (modify for new strategies)
// ❌ Hard to test (need to test all strategies together)
// ❌ Conditional logic scattered
// ❌ Hard to extend

// ✅ WITH STRATEGY: Each algorithm in separate class
interface PaymentStrategy {
  process(amount: number): Promise<PaymentResult>;
}

class StripeStrategy implements PaymentStrategy {
  async process(amount: number): Promise<PaymentResult> {
    console.log('Processing via Stripe');
    // Stripe-specific logic
    return { success: true, transactionId: 'stripe_123' };
  }
}

class PaypalStrategy implements PaymentStrategy {
  async process(amount: number): Promise<PaymentResult> {
    console.log('Processing via PayPal');
    // PayPal-specific logic
    return { success: true, transactionId: 'paypal_456' };
  }
}

class PaymentService {
  constructor(private strategy: PaymentStrategy) {}

  setStrategy(strategy: PaymentStrategy): void {
    this.strategy = strategy;
  }

  async processPayment(amount: number): Promise<PaymentResult> {
    return this.strategy.process(amount);  // Delegate to strategy
  }
}

// Usage:
const service = new PaymentService(new StripeStrategy());
await service.processPayment(100);  // Uses Stripe

service.setStrategy(new PaypalStrategy());
await service.processPayment(200);  // Uses PayPal

// Benefits:
// ✅ Open-Closed Principle (add new strategies without modifying service)
// ✅ Easy to test (test each strategy independently)
// ✅ No conditional logic
// ✅ Easy to extend
```

---

### **Method 1: Passport Authentication Strategies**

```typescript
// ========== PASSPORT STRATEGIES IN NESTJS ==========
// auth/strategies/local.strategy.ts
import { Strategy } from 'passport-local';
import { PassportStrategy } from '@nestjs/passport';
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { AuthService } from '../services/auth.service';

@Injectable()
export class LocalStrategy extends PassportStrategy(Strategy) {
  constructor(private authService: AuthService) {
    super({
      usernameField: 'email',
      passwordField: 'password',
    });
  }

  // Strategy-specific validation logic
  async validate(email: string, password: string): Promise<any> {
    const user = await this.authService.validateUser(email, password);
    if (!user) {
      throw new UnauthorizedException('Invalid credentials');
    }
    return user;
  }
}

// auth/strategies/jwt.strategy.ts
import { ExtractJwt, Strategy } from 'passport-jwt';
import { PassportStrategy } from '@nestjs/passport';
import { Injectable } from '@nestjs/common';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: 'your-secret-key',
    });
  }

  // Strategy-specific validation logic
  async validate(payload: any) {
    return { userId: payload.sub, username: payload.username };
  }
}

// auth/strategies/google.strategy.ts
import { Strategy, VerifyCallback } from 'passport-google-oauth20';
import { PassportStrategy } from '@nestjs/passport';
import { Injectable } from '@nestjs/common';

@Injectable()
export class GoogleStrategy extends PassportStrategy(Strategy, 'google') {
  constructor() {
    super({
      clientID: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
      callbackURL: 'http://localhost:3000/auth/google/callback',
      scope: ['email', 'profile'],
    });
  }

  // Strategy-specific validation logic
  async validate(
    accessToken: string,
    refreshToken: string,
    profile: any,
    done: VerifyCallback,
  ): Promise<any> {
    const { name, emails, photos } = profile;
    const user = {
      email: emails[0].value,
      firstName: name.givenName,
      lastName: name.familyName,
      picture: photos[0].value,
      accessToken,
    };
    done(null, user);
  }
}

// auth/auth.module.ts
@Module({
  imports: [
    PassportModule,
    JwtModule.register({
      secret: 'your-secret-key',
      signOptions: { expiresIn: '1h' },
    }),
  ],
  providers: [
    AuthService,
    LocalStrategy,   // Register strategies
    JwtStrategy,
    GoogleStrategy,
  ],
})
export class AuthModule {}

// auth/controllers/auth.controller.ts
@Controller('auth')
export class AuthController {
  constructor(private authService: AuthService) {}

  // Use LocalStrategy
  @UseGuards(AuthGuard('local'))
  @Post('login')
  async login(@Request() req) {
    return this.authService.login(req.user);
  }

  // Use JwtStrategy
  @UseGuards(AuthGuard('jwt'))
  @Get('profile')
  getProfile(@Request() req) {
    return req.user;
  }

  // Use GoogleStrategy
  @Get('google')
  @UseGuards(AuthGuard('google'))
  async googleAuth(@Request() req) {}

  @Get('google/callback')
  @UseGuards(AuthGuard('google'))
  googleAuthRedirect(@Request() req) {
    return this.authService.googleLogin(req.user);
  }
}

// Different strategies for different authentication methods!
// AuthGuard('local') → LocalStrategy
// AuthGuard('jwt') → JwtStrategy
// AuthGuard('google') → GoogleStrategy
```

---

### **Method 2: Payment Processing Strategies**

```typescript
// ========== PAYMENT STRATEGIES ==========
// payments/strategies/payment-strategy.interface.ts
export interface PaymentStrategy {
  charge(amount: number, currency: string, metadata: any): Promise<PaymentResult>;
  refund(transactionId: string): Promise<void>;
  getTransactionStatus(transactionId: string): Promise<PaymentStatus>;
}

export interface PaymentResult {
  success: boolean;
  transactionId: string;
  message?: string;
}

export enum PaymentStatus {
  PENDING = 'pending',
  COMPLETED = 'completed',
  FAILED = 'failed',
  REFUNDED = 'refunded',
}

// payments/strategies/stripe-payment.strategy.ts
@Injectable()
export class StripePaymentStrategy implements PaymentStrategy {
  private stripe: Stripe;

  constructor(private configService: ConfigService) {
    this.stripe = new Stripe(configService.get('STRIPE_SECRET_KEY'), {
      apiVersion: '2023-10-16',
    });
  }

  async charge(amount: number, currency: string, metadata: any): Promise<PaymentResult> {
    try {
      const paymentIntent = await this.stripe.paymentIntents.create({
        amount: amount * 100, // Stripe uses cents
        currency,
        metadata,
      });

      return {
        success: true,
        transactionId: paymentIntent.id,
        message: 'Payment processed via Stripe',
      };
    } catch (error) {
      return {
        success: false,
        transactionId: null,
        message: error.message,
      };
    }
  }

  async refund(transactionId: string): Promise<void> {
    await this.stripe.refunds.create({
      payment_intent: transactionId,
    });
  }

  async getTransactionStatus(transactionId: string): Promise<PaymentStatus> {
    const paymentIntent = await this.stripe.paymentIntents.retrieve(transactionId);
    return this.mapStripeStatus(paymentIntent.status);
  }

  private mapStripeStatus(status: string): PaymentStatus {
    const mapping = {
      'processing': PaymentStatus.PENDING,
      'succeeded': PaymentStatus.COMPLETED,
      'canceled': PaymentStatus.FAILED,
    };
    return mapping[status] || PaymentStatus.PENDING;
  }
}

// payments/strategies/paypal-payment.strategy.ts
@Injectable()
export class PaypalPaymentStrategy implements PaymentStrategy {
  private paypal: any;

  constructor(private configService: ConfigService) {
    this.paypal = {
      clientId: configService.get('PAYPAL_CLIENT_ID'),
      clientSecret: configService.get('PAYPAL_CLIENT_SECRET'),
    };
  }

  async charge(amount: number, currency: string, metadata: any): Promise<PaymentResult> {
    try {
      // PayPal-specific logic
      console.log(`PayPal: Charging ${amount} ${currency}`);
      
      return {
        success: true,
        transactionId: `paypal_${Date.now()}`,
        message: 'Payment processed via PayPal',
      };
    } catch (error) {
      return {
        success: false,
        transactionId: null,
        message: error.message,
      };
    }
  }

  async refund(transactionId: string): Promise<void> {
    console.log(`PayPal: Refunding ${transactionId}`);
    // PayPal refund logic
  }

  async getTransactionStatus(transactionId: string): Promise<PaymentStatus> {
    // PayPal status check logic
    return PaymentStatus.COMPLETED;
  }
}

// payments/strategies/square-payment.strategy.ts
@Injectable()
export class SquarePaymentStrategy implements PaymentStrategy {
  private squareClient: any;

  constructor(private configService: ConfigService) {
    this.squareClient = {
      accessToken: configService.get('SQUARE_ACCESS_TOKEN'),
    };
  }

  async charge(amount: number, currency: string, metadata: any): Promise<PaymentResult> {
    try {
      console.log(`Square: Charging ${amount} ${currency}`);
      
      return {
        success: true,
        transactionId: `square_${Date.now()}`,
        message: 'Payment processed via Square',
      };
    } catch (error) {
      return {
        success: false,
        transactionId: null,
        message: error.message,
      };
    }
  }

  async refund(transactionId: string): Promise<void> {
    console.log(`Square: Refunding ${transactionId}`);
  }

  async getTransactionStatus(transactionId: string): Promise<PaymentStatus> {
    return PaymentStatus.COMPLETED;
  }
}

// payments/services/payments.service.ts
@Injectable()
export class PaymentsService {
  private strategies: Map<string, PaymentStrategy>;

  constructor(
    private stripeStrategy: StripePaymentStrategy,
    private paypalStrategy: PaypalPaymentStrategy,
    private squareStrategy: SquarePaymentStrategy,
  ) {
    this.strategies = new Map();
    this.strategies.set('stripe', this.stripeStrategy);
    this.strategies.set('paypal', this.paypalStrategy);
    this.strategies.set('square', this.squareStrategy);
  }

  async processPayment(
    method: string,
    amount: number,
    currency: string,
    metadata: any,
  ): Promise<PaymentResult> {
    const strategy = this.strategies.get(method);
    if (!strategy) {
      throw new BadRequestException(`Unsupported payment method: ${method}`);
    }

    // Delegate to strategy
    return strategy.charge(amount, currency, metadata);
  }

  async refundPayment(method: string, transactionId: string): Promise<void> {
    const strategy = this.strategies.get(method);
    if (!strategy) {
      throw new BadRequestException(`Unsupported payment method: ${method}`);
    }

    return strategy.refund(transactionId);
  }

  async getStatus(method: string, transactionId: string): Promise<PaymentStatus> {
    const strategy = this.strategies.get(method);
    if (!strategy) {
      throw new BadRequestException(`Unsupported payment method: ${method}`);
    }

    return strategy.getTransactionStatus(transactionId);
  }
}

// payments/controllers/payments.controller.ts
@Controller('payments')
export class PaymentsController {
  constructor(private paymentsService: PaymentsService) {}

  @Post('charge')
  async charge(@Body() dto: ChargePaymentDto) {
    return this.paymentsService.processPayment(
      dto.method,    // 'stripe', 'paypal', or 'square'
      dto.amount,
      dto.currency,
      dto.metadata,
    );
  }

  @Post('refund')
  async refund(@Body() dto: RefundPaymentDto) {
    return this.paymentsService.refundPayment(dto.method, dto.transactionId);
  }

  @Get('status/:method/:transactionId')
  async getStatus(
    @Param('method') method: string,
    @Param('transactionId') transactionId: string,
  ) {
    return this.paymentsService.getStatus(method, transactionId);
  }
}

// Easy to add new payment method: Create new strategy class, register in map!
```

---

### **Method 3: Notification Channel Strategies**

```typescript
// ========== NOTIFICATION STRATEGIES ==========
// notifications/strategies/notification-strategy.interface.ts
export interface NotificationStrategy {
  send(recipient: string, subject: string, body: string): Promise<void>;
  sendBulk(recipients: string[], subject: string, body: string): Promise<void>;
}

// notifications/strategies/email-notification.strategy.ts
@Injectable()
export class EmailNotificationStrategy implements NotificationStrategy {
  constructor(private configService: ConfigService) {}

  async send(recipient: string, subject: string, body: string): Promise<void> {
    console.log(`Sending email to ${recipient}`);
    console.log(`Subject: ${subject}`);
    console.log(`Body: ${body}`);
    // Email sending logic (e.g., using nodemailer, SendGrid)
  }

  async sendBulk(recipients: string[], subject: string, body: string): Promise<void> {
    for (const recipient of recipients) {
      await this.send(recipient, subject, body);
    }
  }
}

// notifications/strategies/sms-notification.strategy.ts
@Injectable()
export class SmsNotificationStrategy implements NotificationStrategy {
  constructor(private configService: ConfigService) {}

  async send(recipient: string, subject: string, body: string): Promise<void> {
    console.log(`Sending SMS to ${recipient}`);
    console.log(`Message: ${body}`);
    // SMS sending logic (e.g., using Twilio, AWS SNS)
  }

  async sendBulk(recipients: string[], subject: string, body: string): Promise<void> {
    for (const recipient of recipients) {
      await this.send(recipient, subject, body);
    }
  }
}

// notifications/strategies/push-notification.strategy.ts
@Injectable()
export class PushNotificationStrategy implements NotificationStrategy {
  constructor(private configService: ConfigService) {}

  async send(recipient: string, subject: string, body: string): Promise<void> {
    console.log(`Sending push notification to ${recipient}`);
    console.log(`Title: ${subject}`);
    console.log(`Body: ${body}`);
    // Push notification logic (e.g., using Firebase Cloud Messaging)
  }

  async sendBulk(recipients: string[], subject: string, body: string): Promise<void> {
    console.log(`Sending bulk push to ${recipients.length} devices`);
    // Bulk push logic
  }
}

// notifications/services/notifications.service.ts
@Injectable()
export class NotificationsService {
  private strategies: Map<string, NotificationStrategy>;

  constructor(
    private emailStrategy: EmailNotificationStrategy,
    private smsStrategy: SmsNotificationStrategy,
    private pushStrategy: PushNotificationStrategy,
  ) {
    this.strategies = new Map();
    this.strategies.set('email', this.emailStrategy);
    this.strategies.set('sms', this.smsStrategy);
    this.strategies.set('push', this.pushStrategy);
  }

  async sendNotification(
    channel: string,
    recipient: string,
    subject: string,
    body: string,
  ): Promise<void> {
    const strategy = this.strategies.get(channel);
    if (!strategy) {
      throw new BadRequestException(`Unsupported notification channel: ${channel}`);
    }

    return strategy.send(recipient, subject, body);
  }

  async sendMultiChannel(
    channels: string[],
    recipient: string,
    subject: string,
    body: string,
  ): Promise<void> {
    // Send via multiple channels
    await Promise.all(
      channels.map(channel => this.sendNotification(channel, recipient, subject, body)),
    );
  }
}

// Usage:
await notificationsService.sendNotification('email', 'user@example.com', 'Welcome', 'Hello!');
await notificationsService.sendNotification('sms', '+1234567890', 'OTP', 'Your code: 123456');
await notificationsService.sendNotification('push', 'device_token_123', 'New Message', 'You have a message');
```

---

### **Method 4: File Storage Strategies**

```typescript
// ========== FILE STORAGE STRATEGIES ==========
// storage/strategies/storage-strategy.interface.ts
export interface StorageStrategy {
  upload(file: Express.Multer.File, path: string): Promise<string>;
  download(path: string): Promise<Buffer>;
  delete(path: string): Promise<void>;
  getUrl(path: string): string;
}

// storage/strategies/s3-storage.strategy.ts
@Injectable()
export class S3StorageStrategy implements StorageStrategy {
  private s3: AWS.S3;

  constructor(private configService: ConfigService) {
    this.s3 = new AWS.S3({
      accessKeyId: configService.get('AWS_ACCESS_KEY'),
      secretAccessKey: configService.get('AWS_SECRET_KEY'),
      region: configService.get('AWS_REGION'),
    });
  }

  async upload(file: Express.Multer.File, path: string): Promise<string> {
    const params = {
      Bucket: this.configService.get('AWS_S3_BUCKET'),
      Key: path,
      Body: file.buffer,
      ContentType: file.mimetype,
    };

    await this.s3.upload(params).promise();
    return path;
  }

  async download(path: string): Promise<Buffer> {
    const params = {
      Bucket: this.configService.get('AWS_S3_BUCKET'),
      Key: path,
    };

    const result = await this.s3.getObject(params).promise();
    return result.Body as Buffer;
  }

  async delete(path: string): Promise<void> {
    const params = {
      Bucket: this.configService.get('AWS_S3_BUCKET'),
      Key: path,
    };

    await this.s3.deleteObject(params).promise();
  }

  getUrl(path: string): string {
    return `https://${this.configService.get('AWS_S3_BUCKET')}.s3.amazonaws.com/${path}`;
  }
}

// storage/strategies/local-storage.strategy.ts
@Injectable()
export class LocalStorageStrategy implements StorageStrategy {
  private uploadsDir = './uploads';

  async upload(file: Express.Multer.File, path: string): Promise<string> {
    const fullPath = `${this.uploadsDir}/${path}`;
    await fs.promises.mkdir(dirname(fullPath), { recursive: true });
    await fs.promises.writeFile(fullPath, file.buffer);
    return path;
  }

  async download(path: string): Promise<Buffer> {
    const fullPath = `${this.uploadsDir}/${path}`;
    return fs.promises.readFile(fullPath);
  }

  async delete(path: string): Promise<void> {
    const fullPath = `${this.uploadsDir}/${path}`;
    await fs.promises.unlink(fullPath);
  }

  getUrl(path: string): string {
    return `http://localhost:3000/uploads/${path}`;
  }
}

// storage/storage.module.ts
@Module({
  providers: [
    S3StorageStrategy,
    LocalStorageStrategy,
    {
      provide: 'STORAGE_STRATEGY',
      useFactory: (
        s3Strategy: S3StorageStrategy,
        localStrategy: LocalStorageStrategy,
        configService: ConfigService,
      ) => {
        const storageType = configService.get('STORAGE_TYPE');
        return storageType === 's3' ? s3Strategy : localStrategy;
      },
      inject: [S3StorageStrategy, LocalStorageStrategy, ConfigService],
    },
  ],
})
export class StorageModule {}

// storage/services/storage.service.ts
@Injectable()
export class StorageService {
  constructor(
    @Inject('STORAGE_STRATEGY')
    private strategy: StorageStrategy,
  ) {}

  async uploadFile(file: Express.Multer.File, path: string): Promise<string> {
    return this.strategy.upload(file, path);
  }

  async downloadFile(path: string): Promise<Buffer> {
    return this.strategy.download(path);
  }

  async deleteFile(path: string): Promise<void> {
    return this.strategy.delete(path);
  }

  getFileUrl(path: string): string {
    return this.strategy.getUrl(path);
  }
}

// Switch between S3 and Local storage via config!
```

---

### **Method 5: Validation Strategies**

```typescript
// ========== VALIDATION STRATEGIES ==========
// validation/strategies/validation-strategy.interface.ts
export interface ValidationStrategy {
  validate(data: any): ValidationResult;
}

export interface ValidationResult {
  valid: boolean;
  errors: string[];
}

// validation/strategies/strict-validation.strategy.ts
@Injectable()
export class StrictValidationStrategy implements ValidationStrategy {
  validate(data: any): ValidationResult {
    const errors: string[] = [];

    // Strict validation rules
    if (!data.email || !data.email.includes('@')) {
      errors.push('Valid email is required');
    }
    if (!data.password || data.password.length < 12) {
      errors.push('Password must be at least 12 characters');
    }
    if (!data.name || data.name.length < 3) {
      errors.push('Name must be at least 3 characters');
    }

    return { valid: errors.length === 0, errors };
  }
}

// validation/strategies/lenient-validation.strategy.ts
@Injectable()
export class LenientValidationStrategy implements ValidationStrategy {
  validate(data: any): ValidationResult {
    const errors: string[] = [];

    // Lenient validation rules
    if (!data.email) {
      errors.push('Email is required');
    }
    if (!data.password || data.password.length < 6) {
      errors.push('Password must be at least 6 characters');
    }

    return { valid: errors.length === 0, errors };
  }
}

// validation/services/validation.service.ts
@Injectable()
export class ValidationService {
  constructor(
    private strictStrategy: StrictValidationStrategy,
    private lenientStrategy: LenientValidationStrategy,
  ) {}

  validate(data: any, mode: 'strict' | 'lenient' = 'strict'): ValidationResult {
    const strategy = mode === 'strict' ? this.strictStrategy : this.lenientStrategy;
    return strategy.validate(data);
  }
}
```

---

### **Benefits of Strategy Pattern:**

| Benefit | Description | Example |
|---------|-------------|---------|
| **Open-Closed** | Add strategies without modifying client | Add new payment method without changing service |
| **Testability** | Test each strategy independently | Test Stripe strategy separately |
| **Flexibility** | Switch strategies at runtime | Change payment method per user preference |
| **Maintainability** | Each strategy in own file | StripeStrategy.ts, PaypalStrategy.ts |
| **Reusability** | Strategies can be reused | Same Stripe strategy in multiple services |
| **No Conditionals** | No if/else chains | No `if (method === 'stripe')` |

---

### **Best Practices:**

```typescript
// ✅ GOOD: Strategy interface
export interface PaymentStrategy {
  charge(amount: number): Promise<PaymentResult>;
}

// ❌ BAD: No interface
class StripePayment { /* ... */ }
class PaypalPayment { /* ... */ }  // Different APIs

// ✅ GOOD: Store strategies in Map
private strategies = new Map<string, Strategy>();

// ❌ BAD: Switch statement
if (method === 'stripe') return new StripeStrategy();

// ✅ GOOD: Register strategies in constructor
constructor(stripe: StripeStrategy, paypal: PaypalStrategy) {
  this.strategies.set('stripe', stripe);
  this.strategies.set('paypal', paypal);
}

// ❌ BAD: Create strategies on demand
getStrategy(method: string) {
  return new StripeStrategy();  // New instance every time
}

// ✅ GOOD: Delegate to strategy
return this.strategy.charge(amount);

// ❌ BAD: Client has strategy logic
if (this.method === 'stripe') {
  // Stripe logic here
}
```

**Key Takeaway:** **Strategy Pattern** encapsulates **interchangeable algorithms** in separate classes with common interface, allowing runtime selection without conditional logic. In NestJS, use for: **Passport strategies** (Local, JWT, OAuth), **payment methods** (Stripe, PayPal, Square), **notification channels** (Email, SMS, Push), **file storage** (S3, Local, GCS), and **validation rules** (strict, lenient). Benefits: **Open-Closed Principle** (add strategies without modifying client), **no conditionals**, **testable** (test each strategy independently), **maintainable** (one file per strategy), and **flexible** (switch at runtime). Implement with interface, register strategies in Map, and delegate operations to selected strategy.

</details>

<details>
<summary><strong>14. What is Decorator Pattern (used extensively in NestJS)?</strong></summary>

**Answer:**

**Decorator Pattern** adds new functionality to objects **dynamically** without modifying their structure. In TypeScript/NestJS, decorators are **annotations** (using `@` syntax) that modify classes, methods, properties, or parameters at **design time**. NestJS uses decorators extensively: `@Controller()`, `@Get()`, `@Injectable()`, `@UseGuards()`, `@UsePipes()`, etc. Decorators enable **metadata reflection**, **AOP (Aspect-Oriented Programming)**, and **declarative programming**.

---

### **Decorator Concept:**

```typescript
// ❌ WITHOUT DECORATORS: Imperative code
class UsersController {
  constructor() {
    // Manually register routes
    app.get('/users', this.findAll.bind(this));
    app.post('/users', this.create.bind(this));
    app.get('/users/:id', this.findOne.bind(this));
  }

  async findAll() {
    // Manually add validation
    // Manually add authentication
    // Manually add authorization
    // Manually add logging
    // Then actual logic...
  }
}

// Problems:
// ❌ Boilerplate code
// ❌ Cross-cutting concerns mixed with business logic
// ❌ Hard to maintain
// ❌ Repetitive

// ✅ WITH DECORATORS: Declarative code
@Controller('users')  // Declare this is a controller for /users
@UseGuards(AuthGuard)  // Apply authentication to all routes
export class UsersController {
  @Get()  // Declare GET /users route
  @UseInterceptors(LoggingInterceptor)  // Add logging
  async findAll() {
    // Only business logic!
    return [];
  }

  @Post()  // Declare POST /users route
  @UsePipes(ValidationPipe)  // Add validation
  async create(@Body() dto: CreateUserDto) {
    // Only business logic!
    return dto;
  }

  @Get(':id')  // Declare GET /users/:id route
  @UseGuards(RolesGuard)  // Add authorization
  @Roles('admin')  // Declare required role
  async findOne(@Param('id') id: string) {
    // Only business logic!
    return { id };
  }
}

// Benefits:
// ✅ Clean, declarative code
// ✅ Cross-cutting concerns separated
// ✅ Reusable decorators
// ✅ Easy to maintain
```

---

### **Method 1: Built-in NestJS Decorators**

```typescript
// ========== CONTROLLER DECORATORS ==========
import {
  Controller,
  Get,
  Post,
  Put,
  Delete,
  Patch,
  Param,
  Body,
  Query,
  Headers,
  Req,
  Res,
  HttpCode,
  HttpStatus,
  UseGuards,
  UsePipes,
  UseInterceptors,
  UseFilters,
} from '@nestjs/common';

@Controller('users')  // Class decorator: Define controller path
@UseGuards(AuthGuard)  // Apply guard to all routes
export class UsersController {
  // ========== METHOD DECORATORS ==========
  @Get()  // Define HTTP GET method
  @HttpCode(HttpStatus.OK)  // Set response status code
  @UseInterceptors(LoggingInterceptor)  // Apply interceptor
  async findAll(
    @Query('page') page: string,  // Parameter decorator: Extract query param
    @Query('limit') limit: string,
    @Headers('authorization') auth: string,  // Extract header
  ) {
    return { page, limit, auth };
  }

  @Post()  // Define HTTP POST method
  @UsePipes(ValidationPipe)  // Apply validation pipe
  @UseFilters(HttpExceptionFilter)  // Apply exception filter
  async create(
    @Body() dto: CreateUserDto,  // Parameter decorator: Extract body
  ) {
    return dto;
  }

  @Get(':id')  // Route parameter
  async findOne(
    @Param('id') id: string,  // Parameter decorator: Extract route param
  ) {
    return { id };
  }

  @Put(':id')
  async update(
    @Param('id') id: string,
    @Body() dto: UpdateUserDto,
  ) {
    return { id, ...dto };
  }

  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  async remove(@Param('id') id: string) {
    return;
  }
}

// ========== SERVICE DECORATORS ==========
@Injectable()  // Class decorator: Mark as injectable
export class UsersService {
  constructor(private usersRepository: UsersRepository) {}  // DI

  async findAll() {
    return this.usersRepository.findAll();
  }
}

// ========== ENTITY DECORATORS (TypeORM) ==========
import { Entity, Column, PrimaryGeneratedColumn, CreateDateColumn } from 'typeorm';

@Entity('users')  // Class decorator: Mark as database entity
export class User {
  @PrimaryGeneratedColumn('uuid')  // Property decorator: Primary key
  id: string;

  @Column()  // Property decorator: Database column
  name: string;

  @Column({ unique: true })  // Column with options
  email: string;

  @CreateDateColumn()  // Auto-generate creation timestamp
  createdAt: Date;
}

// ========== GUARD DECORATORS ==========
@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    return user?.role === 'admin';
  }
}

@Controller('admin')
@UseGuards(RolesGuard)  // Apply guard
export class AdminController {
  @Get()
  getAdminData() {
    return { message: 'Admin only' };
  }
}
```

---

### **Method 2: Custom Decorators**

```typescript
// ========== CUSTOM PARAMETER DECORATOR: @User() ==========
// common/decorators/user.decorator.ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const User = createParamDecorator(
  (data: string, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const user = request.user;

    // If data specified, return specific property
    return data ? user?.[data] : user;
  },
);

// Usage:
@Controller('profile')
export class ProfileController {
  @Get()
  @UseGuards(JwtAuthGuard)
  getProfile(@User() user: any) {
    // Get entire user object
    return user;
  }

  @Get('email')
  @UseGuards(JwtAuthGuard)
  getEmail(@User('email') email: string) {
    // Get only email property
    return { email };
  }
}

// ========== CUSTOM METHOD DECORATOR: @Roles() ==========
// common/decorators/roles.decorator.ts
import { SetMetadata } from '@nestjs/common';

export const ROLES_KEY = 'roles';
export const Roles = (...roles: string[]) => SetMetadata(ROLES_KEY, roles);

// Usage:
@Controller('admin')
export class AdminController {
  @Get('users')
  @Roles('admin')  // Set metadata
  getUsers() {
    return [];
  }

  @Get('settings')
  @Roles('admin', 'superadmin')  // Multiple roles
  getSettings() {
    return {};
  }
}

// Guard reads metadata:
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<string[]>(ROLES_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);

    if (!requiredRoles) {
      return true;
    }

    const request = context.switchToHttp().getRequest();
    const user = request.user;

    return requiredRoles.some((role) => user?.roles?.includes(role));
  }
}

// ========== CUSTOM METHOD DECORATOR: @Public() ==========
// common/decorators/public.decorator.ts
import { SetMetadata } from '@nestjs/common';

export const IS_PUBLIC_KEY = 'isPublic';
export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);

// Usage:
@Controller('auth')
export class AuthController {
  @Post('login')
  @Public()  // Skip authentication for this route
  login(@Body() dto: LoginDto) {
    return this.authService.login(dto);
  }

  @Post('register')
  @Public()  // Skip authentication
  register(@Body() dto: RegisterDto) {
    return this.authService.register(dto);
  }

  @Get('profile')
  // No @Public() - authentication required
  getProfile(@User() user: any) {
    return user;
  }
}

// Guard checks for @Public():
@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  constructor(private reflector: Reflector) {
    super();
  }

  canActivate(context: ExecutionContext) {
    const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);

    if (isPublic) {
      return true;  // Skip authentication
    }

    return super.canActivate(context);
  }
}

// ========== CUSTOM METHOD DECORATOR: @Timeout() ==========
// common/decorators/timeout.decorator.ts
import { SetMetadata } from '@nestjs/common';

export const TIMEOUT_KEY = 'timeout';
export const Timeout = (ms: number) => SetMetadata(TIMEOUT_KEY, ms);

// Usage:
@Controller('reports')
export class ReportsController {
  @Get('slow')
  @Timeout(30000)  // 30 second timeout
  async generateSlowReport() {
    // Long-running operation
    await new Promise(resolve => setTimeout(resolve, 25000));
    return { report: 'data' };
  }

  @Get('fast')
  @Timeout(5000)  // 5 second timeout
  async generateFastReport() {
    return { report: 'data' };
  }
}

// Interceptor reads timeout metadata:
@Injectable()
export class TimeoutInterceptor implements NestInterceptor {
  constructor(private reflector: Reflector) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const timeout = this.reflector.get<number>(TIMEOUT_KEY, context.getHandler());

    if (!timeout) {
      return next.handle();
    }

    return next.handle().pipe(
      timeoutWith(
        timeout,
        throwError(() => new RequestTimeoutException(`Request timeout after ${timeout}ms`)),
      ),
    );
  }
}
```

---

### **Method 3: Combining Multiple Decorators**

```typescript
// ========== COMBINING DECORATORS ==========
// common/decorators/api-endpoint.decorator.ts
import { applyDecorators } from '@nestjs/common';
import {
  ApiOperation,
  ApiResponse,
  ApiBearerAuth,
  ApiTags,
} from '@nestjs/swagger';
import { UseGuards, UseInterceptors } from '@nestjs/common';
import { JwtAuthGuard } from '../guards/jwt-auth.guard';
import { LoggingInterceptor } from '../interceptors/logging.interceptor';
import { TransformInterceptor } from '../interceptors/transform.interceptor';

// Composite decorator
export function ApiEndpoint(options: {
  summary: string;
  description?: string;
  auth?: boolean;
  roles?: string[];
}) {
  const decorators = [
    ApiOperation({ summary: options.summary, description: options.description }),
    ApiResponse({ status: 200, description: 'Success' }),
    ApiResponse({ status: 400, description: 'Bad Request' }),
    UseInterceptors(LoggingInterceptor, TransformInterceptor),
  ];

  if (options.auth) {
    decorators.push(UseGuards(JwtAuthGuard), ApiBearerAuth());
  }

  if (options.roles) {
    decorators.push(Roles(...options.roles));
  }

  return applyDecorators(...decorators);
}

// Usage: One decorator applies many!
@Controller('users')
@ApiTags('users')
export class UsersController {
  @Get()
  @ApiEndpoint({
    summary: 'Get all users',
    description: 'Retrieve list of all users',
    auth: true,
    roles: ['admin'],
  })
  findAll() {
    return [];
  }

  @Post()
  @ApiEndpoint({
    summary: 'Create user',
    auth: true,
  })
  create(@Body() dto: CreateUserDto) {
    return dto;
  }

  @Get('public')
  @ApiEndpoint({
    summary: 'Get public user data',
    auth: false,  // No authentication
  })
  getPublic() {
    return { data: 'public' };
  }
}
```

---

### **Method 4: Class Decorator Example**

```typescript
// ========== CUSTOM CLASS DECORATOR ==========
// common/decorators/cacheable.decorator.ts
export function Cacheable(ttl: number) {
  return function (target: any) {
    // Add caching metadata to class
    Reflect.defineMetadata('cache:ttl', ttl, target);
    Reflect.defineMetadata('cache:enabled', true, target);

    // Wrap all methods with caching logic
    const methodNames = Object.getOwnPropertyNames(target.prototype);
    
    for (const methodName of methodNames) {
      if (methodName === 'constructor') continue;

      const originalMethod = target.prototype[methodName];
      
      target.prototype[methodName] = async function (...args: any[]) {
        const cacheKey = `${target.name}:${methodName}:${JSON.stringify(args)}`;
        
        // Check cache
        const cached = await cache.get(cacheKey);
        if (cached) {
          console.log(`Cache HIT: ${cacheKey}`);
          return cached;
        }

        // Call original method
        console.log(`Cache MISS: ${cacheKey}`);
        const result = await originalMethod.apply(this, args);

        // Store in cache
        await cache.set(cacheKey, result, ttl);

        return result;
      };
    }

    return target;
  };
}

// Usage:
@Injectable()
@Cacheable(60)  // Cache all methods for 60 seconds
export class UsersService {
  async findAll(): Promise<User[]> {
    console.log('Fetching from database...');
    // Expensive database query
    return [];
  }

  async findById(id: string): Promise<User> {
    console.log('Fetching user from database...');
    // Expensive database query
    return null;
  }
}

// First call: Fetches from database, stores in cache
// Second call (within 60s): Returns from cache
```

---

### **Method 5: Property Decorator Example**

```typescript
// ========== CUSTOM PROPERTY DECORATOR ==========
// common/decorators/inject-logger.decorator.ts
import { Logger } from '@nestjs/common';

export function InjectLogger(context?: string) {
  return function (target: any, propertyKey: string) {
    // Create logger instance
    const logger = new Logger(context || target.constructor.name);

    // Define property
    Object.defineProperty(target, propertyKey, {
      get: () => logger,
      enumerable: true,
      configurable: true,
    });
  };
}

// Usage:
@Injectable()
export class UsersService {
  @InjectLogger('UsersService')
  private readonly logger: Logger;

  async findAll(): Promise<User[]> {
    this.logger.log('Finding all users');
    // Logic...
    return [];
  }

  async create(dto: CreateUserDto): Promise<User> {
    this.logger.log('Creating user');
    // Logic...
    return null;
  }
}

// ========== VALIDATION PROPERTY DECORATOR ==========
// common/decorators/min-value.decorator.ts
export function MinValue(min: number) {
  return function (target: any, propertyKey: string) {
    let value: number;

    const getter = function () {
      return value;
    };

    const setter = function (newValue: number) {
      if (newValue < min) {
        throw new Error(`${propertyKey} must be at least ${min}`);
      }
      value = newValue;
    };

    Object.defineProperty(target, propertyKey, {
      get: getter,
      set: setter,
      enumerable: true,
      configurable: true,
    });
  };
}

// Usage:
export class CreateProductDto {
  @MinValue(0)
  price: number;

  @MinValue(1)
  quantity: number;
}

const dto = new CreateProductDto();
dto.price = 10;  // OK
dto.quantity = 5;  // OK
dto.price = -5;  // Error: price must be at least 0
```

---

### **Common NestJS Decorators:**

| Category | Decorator | Purpose |
|----------|-----------|---------|
| **Controller** | `@Controller()` | Define controller |
| | `@Get()`, `@Post()`, `@Put()`, `@Delete()` | HTTP methods |
| | `@Param()`, `@Body()`, `@Query()`, `@Headers()` | Extract request data |
| **Service** | `@Injectable()` | Mark as injectable service |
| **Guard** | `@UseGuards()` | Apply authentication/authorization |
| **Pipe** | `@UsePipes()` | Apply validation/transformation |
| **Interceptor** | `@UseInterceptors()` | Apply interceptor (logging, transform) |
| **Filter** | `@UseFilters()` | Apply exception filter |
| **Metadata** | `@SetMetadata()` | Attach custom metadata |
| **Swagger** | `@ApiTags()`, `@ApiOperation()`, `@ApiResponse()` | API documentation |
| **Validation** | `@IsString()`, `@IsEmail()`, `@Min()`, `@Max()` | Validation rules (class-validator) |

---

### **Benefits of Decorator Pattern:**

| Benefit | Description | Example |
|---------|-------------|---------|
| **Declarative** | Describe what, not how | `@Get()` instead of `app.get()` |
| **Reusable** | Use same decorator everywhere | `@UseGuards(AuthGuard)` |
| **Composable** | Combine multiple decorators | `@Public()`, `@Roles('admin')` |
| **AOP** | Separate cross-cutting concerns | Authentication, logging, validation |
| **Clean Code** | Business logic separated | No boilerplate in methods |
| **Metadata** | Attach data to classes/methods | Reflector reads metadata |

---

### **Best Practices:**

```typescript
// ✅ GOOD: Use built-in decorators
@Controller('users')
@UseGuards(AuthGuard)

// ❌ BAD: Manual route registration
app.get('/users', handler)

// ✅ GOOD: Combine related decorators
@ApiEndpoint({ summary: 'Get users', auth: true })

// ❌ BAD: Repeat decorators
@UseGuards(AuthGuard)
@UseInterceptors(LoggingInterceptor)
@ApiOperation({ summary: 'Get users' })
// ... repeat for every endpoint

// ✅ GOOD: Custom parameter decorator
@Get()
findAll(@User() user: any) {}

// ❌ BAD: Manual extraction
@Get()
findAll(@Request() req: any) {
  const user = req.user;  // Manual extraction
}

// ✅ GOOD: Metadata-driven behavior
@Roles('admin')
@Get()
adminOnly() {}

// ❌ BAD: Conditional logic everywhere
@Get()
adminOnly(@User() user: any) {
  if (user.role !== 'admin') throw new ForbiddenException();
}

// ✅ GOOD: Apply decorators at class level
@Controller('users')
@UseGuards(AuthGuard)  // All routes protected
export class UsersController {}

// ❌ BAD: Repeat on every method
@Get()
@UseGuards(AuthGuard)
method1() {}

@Post()
@UseGuards(AuthGuard)
method2() {}
```

**Key Takeaway:** **Decorator Pattern** adds functionality dynamically using `@` syntax annotations in TypeScript. NestJS uses decorators extensively for **declarative programming**: `@Controller()`, `@Get()`, `@Injectable()`, `@UseGuards()`, `@Body()`, etc. Decorators enable **metadata reflection** (Reflector reads metadata), **AOP** (separate cross-cutting concerns like auth, logging), and **clean code** (no boilerplate). Create custom decorators with `createParamDecorator()` (parameter), `SetMetadata()` (method/class), or function returning decorator. Combine multiple decorators with `applyDecorators()`. Benefits: **declarative** (what, not how), **reusable**, **composable**, **maintainable**, and **separation of concerns**. Decorators are fundamental to NestJS architecture.

</details>

<details>
<summary><strong>15. What is Singleton Pattern (providers)?</strong></summary>

**Answer:**

**Singleton Pattern** ensures a class has **only one instance** throughout the application lifetime with a global access point. In NestJS, providers use **Singleton scope by default** - the IoC container creates one instance and shares it across all injections. This is the **DEFAULT scope**. Singletons are memory-efficient, share state (cache, config, connections), and are created once at application startup.

---

### **Singleton in NestJS:**

```typescript
// ❌ WITHOUT SINGLETON: Multiple instances
class Logger {
  log(message: string) {
    console.log(message);
  }
}

// Every time you need logger, create new instance
const logger1 = new Logger();
const logger2 = new Logger();
const logger3 = new Logger();
// Problem: 3 separate instances, waste memory, no shared state

// ✅ WITH SINGLETON: One instance shared
@Injectable()
export class Logger {
  log(message: string) {
    console.log(message);
  }
}

@Module({
  providers: [Logger],  // NestJS creates ONE instance
  exports: [Logger],
})
export class SharedModule {}

// Every injection uses SAME instance:
@Injectable()
export class UsersService {
  constructor(private logger: Logger) {}  // Same logger instance
}

@Injectable()
export class OrdersService {
  constructor(private logger: Logger) {}  // Same logger instance
}

@Injectable()
export class ProductsService {
  constructor(private logger: Logger) {}  // Same logger instance
}

// Benefits:
// ✅ Memory efficient (one instance)
// ✅ Shared state (all services use same logger)
// ✅ Created once at startup
// ✅ Automatic by NestJS
```

---

### **Method 1: Default Singleton Scope**

```typescript
// ========== DEFAULT SCOPE (SINGLETON) ==========
// logger/logger.service.ts
import { Injectable, Scope } from '@nestjs/common';

@Injectable()  // Default scope is SINGLETON
export class LoggerService {
  private logs: string[] = [];  // Shared state

  log(message: string, context?: string): void {
    const logEntry = `[${context || 'App'}] ${new Date().toISOString()}: ${message}`;
    this.logs.push(logEntry);
    console.log(logEntry);
  }

  getAllLogs(): string[] {
    return this.logs;  // Returns all logs from all services
  }

  getLogCount(): number {
    return this.logs.length;
  }
}

// logger/logger.module.ts
@Module({
  providers: [LoggerService],  // ONE instance created
  exports: [LoggerService],
})
export class LoggerModule {}

// users/users.service.ts
@Injectable()
export class UsersService {
  constructor(private logger: LoggerService) {}

  async create(dto: CreateUserDto): Promise<User> {
    this.logger.log('Creating user', 'UsersService');  // Uses shared logger
    // ...
    return user;
  }
}

// orders/orders.service.ts
@Injectable()
export class OrdersService {
  constructor(private logger: LoggerService) {}

  async create(dto: CreateOrderDto): Promise<Order> {
    this.logger.log('Creating order', 'OrdersService');  // Uses same logger instance
    // ...
    return order;
  }
}

// app.controller.ts
@Controller()
export class AppController {
  constructor(private logger: LoggerService) {}

  @Get('logs')
  getLogs() {
    // Returns logs from ALL services (shared state)
    return {
      logs: this.logger.getAllLogs(),
      count: this.logger.getLogCount(),
    };
  }
}

// Proof of singleton:
// UsersService.logger === OrdersService.logger === AppController.logger
// All use SAME instance with SHARED state (logs array)
```

---

### **Method 2: Explicit Singleton Scope Declaration**

```typescript
// ========== EXPLICIT SINGLETON SCOPE ==========
import { Injectable, Scope } from '@nestjs/common';

@Injectable({ scope: Scope.DEFAULT })  // Explicitly declare SINGLETON
export class ConfigService {
  private config: any = {};

  constructor() {
    // Load config once at startup
    console.log('ConfigService instantiated (only once)');
    this.config = {
      database: {
        host: process.env.DB_HOST || 'localhost',
        port: parseInt(process.env.DB_PORT) || 5432,
      },
      jwt: {
        secret: process.env.JWT_SECRET || 'secret',
        expiresIn: '1h',
      },
    };
  }

  get(key: string): any {
    return this.config[key];
  }

  set(key: string, value: any): void {
    this.config[key] = value;
  }
}

// This constructor runs ONCE when app starts
// All services share same ConfigService instance
```

---

### **Method 3: Singleton for Database Connection**

```typescript
// ========== SINGLETON: Database Connection Pool ==========
// database/database.service.ts
import { Injectable, OnModuleInit, OnModuleDestroy } from '@nestjs/common';
import { Pool } from 'pg';

@Injectable()  // Singleton by default
export class DatabaseService implements OnModuleInit, OnModuleDestroy {
  private pool: Pool;

  constructor(private configService: ConfigService) {
    console.log('DatabaseService created (singleton)');
  }

  async onModuleInit() {
    // Create connection pool ONCE
    this.pool = new Pool({
      host: this.configService.get('DB_HOST'),
      port: this.configService.get('DB_PORT'),
      database: this.configService.get('DB_NAME'),
      user: this.configService.get('DB_USER'),
      password: this.configService.get('DB_PASSWORD'),
      max: 20,  // Max 20 connections in pool
    });

    console.log('Database connection pool created');
  }

  async query(sql: string, params?: any[]): Promise<any> {
    const client = await this.pool.connect();
    try {
      return await client.query(sql, params);
    } finally {
      client.release();
    }
  }

  async onModuleDestroy() {
    // Clean up connection pool
    await this.pool.end();
    console.log('Database connection pool closed');
  }
}

// All services use SAME connection pool (singleton)
@Injectable()
export class UsersRepository {
  constructor(private db: DatabaseService) {}  // Same pool

  async findAll(): Promise<User[]> {
    const result = await this.db.query('SELECT * FROM users');
    return result.rows;
  }
}

@Injectable()
export class OrdersRepository {
  constructor(private db: DatabaseService) {}  // Same pool

  async findAll(): Promise<Order[]> {
    const result = await this.db.query('SELECT * FROM orders');
    return result.rows;
  }
}

// Benefits:
// ✅ One connection pool shared across all repositories
// ✅ Efficient resource usage
// ✅ Pool managed in one place
```

---

### **Method 4: Singleton for Cache Service**

```typescript
// ========== SINGLETON: In-Memory Cache ==========
// cache/cache.service.ts
import { Injectable } from '@nestjs/common';

@Injectable()  // Singleton - shared cache across app
export class CacheService {
  private cache = new Map<string, { value: any; expiry: number }>();

  constructor() {
    console.log('CacheService created (singleton)');
    
    // Cleanup expired entries every minute
    setInterval(() => {
      this.cleanupExpired();
    }, 60000);
  }

  set(key: string, value: any, ttlSeconds: number = 300): void {
    const expiry = Date.now() + ttlSeconds * 1000;
    this.cache.set(key, { value, expiry });
  }

  get<T>(key: string): T | null {
    const entry = this.cache.get(key);
    
    if (!entry) {
      return null;
    }

    // Check if expired
    if (Date.now() > entry.expiry) {
      this.cache.delete(key);
      return null;
    }

    return entry.value as T;
  }

  delete(key: string): void {
    this.cache.delete(key);
  }

  clear(): void {
    this.cache.clear();
  }

  size(): number {
    return this.cache.size;
  }

  private cleanupExpired(): void {
    const now = Date.now();
    for (const [key, entry] of this.cache.entries()) {
      if (now > entry.expiry) {
        this.cache.delete(key);
      }
    }
  }
}

// All services share SAME cache (singleton)
@Injectable()
export class UsersService {
  constructor(private cache: CacheService) {}

  async findById(id: string): Promise<User> {
    // Check shared cache
    const cached = this.cache.get<User>(`user:${id}`);
    if (cached) {
      return cached;
    }

    // Fetch from database
    const user = await this.usersRepository.findById(id);

    // Store in shared cache
    this.cache.set(`user:${id}`, user, 300);

    return user;
  }
}

@Injectable()
export class OrdersService {
  constructor(private cache: CacheService) {}  // Same cache instance

  async findById(id: string): Promise<Order> {
    const cached = this.cache.get<Order>(`order:${id}`);
    if (cached) {
      return cached;
    }

    const order = await this.ordersRepository.findById(id);
    this.cache.set(`order:${id}`, order, 300);

    return order;
  }
}

// Both services use SAME cache Map (shared state)
```

---

### **Method 5: Singleton vs Other Scopes**

```typescript
// ========== SCOPE COMPARISON ==========

// 1. DEFAULT (SINGLETON) - One instance for entire app
@Injectable({ scope: Scope.DEFAULT })
export class SingletonService {
  private counter = 0;

  increment(): number {
    return ++this.counter;
  }
}

// Call increment() from different services:
// Service A: increment() → 1
// Service B: increment() → 2  (same counter!)
// Service C: increment() → 3  (same counter!)

// 2. REQUEST - New instance per HTTP request
@Injectable({ scope: Scope.REQUEST })
export class RequestScopedService {
  private counter = 0;

  increment(): number {
    return ++this.counter;
  }
}

// Call increment() from different services in SAME request:
// Service A: increment() → 1
// Service B: increment() → 2  (same instance within request)
// New request:
// Service A: increment() → 1  (NEW instance)

// 3. TRANSIENT - New instance every time
@Injectable({ scope: Scope.TRANSIENT })
export class TransientService {
  private counter = 0;

  increment(): number {
    return ++this.counter;
  }
}

// Call increment() from different services:
// Service A: increment() → 1  (new instance)
// Service B: increment() → 1  (NEW instance, separate counter)
// Service C: increment() → 1  (NEW instance, separate counter)

// ========== WHEN TO USE EACH ==========
// SINGLETON (DEFAULT):
// ✅ Shared services (logger, config, cache)
// ✅ Database connections
// ✅ Stateless services
// ✅ Most use cases (95%+)

// REQUEST:
// ❓ Request-specific data (current user, request ID)
// ❓ Audit logging per request
// ⚠️ Performance overhead (new instance per request)

// TRANSIENT:
// ❓ Services that shouldn't share state
// ❓ Temporary workers
// ⚠️ Memory overhead (many instances)
```

---

### **Singleton Pattern Benefits:**

| Benefit | Description | Example |
|---------|-------------|---------|
| **Memory Efficient** | One instance for entire app | LoggerService: 1 instance vs 100+ |
| **Shared State** | All injections share state | CacheService: shared cache Map |
| **Lazy Initialization** | Created once when first needed | Config loaded once at startup |
| **Global Access** | Available via DI everywhere | Same logger in all services |
| **Resource Management** | One connection pool, not many | DatabaseService: single pool |
| **Performance** | No instance creation overhead | No `new` on every injection |

---

### **Common Singleton Use Cases:**

```typescript
// ✅ Configuration Service (singleton)
@Injectable()
export class ConfigService {
  private config: any;  // Loaded once, shared everywhere
}

// ✅ Logger Service (singleton)
@Injectable()
export class LoggerService {
  private logs: string[];  // All services add to same array
}

// ✅ Cache Service (singleton)
@Injectable()
export class CacheService {
  private cache = new Map();  // Shared cache across app
}

// ✅ Database Connection (singleton)
@Injectable()
export class DatabaseService {
  private pool: Pool;  // One pool shared by all
}

// ✅ Event Emitter (singleton)
@Injectable()
export class EventBus {
  private emitter = new EventEmitter();  // Global event bus
}

// ✅ Metrics Collector (singleton)
@Injectable()
export class MetricsService {
  private metrics = {
    requests: 0,
    errors: 0,
  };  // Shared metrics
}

// ❌ DON'T USE SINGLETON for request-specific data
@Injectable({ scope: Scope.REQUEST })  // Use REQUEST scope
export class CurrentUserService {
  user: User;  // Different per request
}
```

---

### **Best Practices:**

```typescript
// ✅ GOOD: Use singleton for shared services
@Injectable()  // Default singleton
export class LoggerService {}

// ❌ BAD: Use REQUEST scope unnecessarily
@Injectable({ scope: Scope.REQUEST })
export class LoggerService {}  // Creates new instance per request!

// ✅ GOOD: Singleton for stateless services
@Injectable()
export class UsersService {
  async findAll() {
    return this.usersRepository.findAll();
  }
}

// ❌ BAD: Singleton with request-specific state
@Injectable()
export class UsersService {
  private currentUser: User;  // DON'T store request data in singleton!
}

// ✅ GOOD: Singleton for connection pools
@Injectable()
export class DatabaseService {
  private pool: Pool;  // Shared connection pool
}

// ❌ BAD: Create new connection every time
@Injectable({ scope: Scope.TRANSIENT })
export class DatabaseService {
  private connection: Connection;  // New connection every injection!
}

// ✅ GOOD: Use constructor for initialization
@Injectable()
export class ConfigService {
  private config: any;

  constructor() {
    this.config = this.loadConfig();  // Runs once
  }
}

// ❌ BAD: Initialize in every method
@Injectable()
export class ConfigService {
  get(key: string) {
    const config = this.loadConfig();  // Loads every call!
    return config[key];
  }
}
```

**Key Takeaway:** **Singleton Pattern** ensures **one instance** throughout application lifetime. In NestJS, providers use **Singleton scope by default** (`scope: Scope.DEFAULT`) - IoC container creates one instance and shares it across all injections. Benefits: **memory efficient** (one instance vs many), **shared state** (cache, config, logs), **lazy initialization** (created once when needed), **global access** (available everywhere via DI), and **resource management** (single connection pool). Use for: **logger**, **config**, **cache**, **database connections**, **event emitters**, and **stateless services** (95%+ of cases). Avoid for: **request-specific data** (use `Scope.REQUEST`), **temporary workers** (use `Scope.TRANSIENT`). Singleton is default and recommended for most services.

</details>

<details>
<summary><strong>16. What is Observer Pattern (event-driven architecture)?</strong></summary>

**Answer:**

**Observer Pattern** defines **one-to-many dependency** where when one object (subject) changes state, all dependents (observers) are **notified automatically**. In NestJS, implement with **EventEmitter2** (`@nestjs/event-emitter`): services emit events, listeners subscribe to events. Use for: **decoupling** (services don't know about each other), **async operations** (send email after user registration), **audit logging**, **notifications**, and **CQRS** (Command Query Responsibility Segregation).

---

### **Observer Pattern Concept:**

```typescript
// ❌ WITHOUT OBSERVER: Tight coupling
@Injectable()
export class UsersService {
  constructor(
    private usersRepository: UsersRepository,
    private emailService: EmailService,
    private analyticsService: AnalyticsService,
    private notificationService: NotificationService,
    private auditService: AuditService,
  ) {}

  async create(dto: CreateUserDto): Promise<User> {
    // Create user
    const user = await this.usersRepository.save(dto);

    // Manually call all related services
    await this.emailService.sendWelcomeEmail(user.email);
    await this.analyticsService.trackUserCreation(user.id);
    await this.notificationService.notifyAdmin('New user created', user);
    await this.auditService.log('USER_CREATED', user.id);

    return user;
  }
}

// Problems:
// ❌ Tight coupling (UsersService knows all services)
// ❌ Hard to extend (add new side effect = modify UsersService)
// ❌ Violates Single Responsibility Principle
// ❌ Hard to test

// ✅ WITH OBSERVER: Loose coupling
@Injectable()
export class UsersService {
  constructor(
    private usersRepository: UsersRepository,
    private eventEmitter: EventEmitter2,  // Only dependency
  ) {}

  async create(dto: CreateUserDto): Promise<User> {
    // Create user
    const user = await this.usersRepository.save(dto);

    // Emit event (don't know who listens)
    this.eventEmitter.emit('user.created', user);

    return user;
  }
}

// Listeners automatically called:
@Injectable()
export class EmailListener {
  @OnEvent('user.created')
  async handleUserCreated(user: User) {
    await this.emailService.sendWelcomeEmail(user.email);
  }
}

@Injectable()
export class AnalyticsListener {
  @OnEvent('user.created')
  async handleUserCreated(user: User) {
    await this.analyticsService.trackUserCreation(user.id);
  }
}

// Benefits:
// ✅ Loose coupling (UsersService doesn't know listeners)
// ✅ Easy to extend (add listener without modifying UsersService)
// ✅ Single Responsibility (each listener has one job)
// ✅ Easy to test
```

---

### **Method 1: Basic Event Emitter Setup**

```typescript
// ========== INSTALL EVENT EMITTER ==========
// npm install @nestjs/event-emitter

// app.module.ts
import { EventEmitterModule } from '@nestjs/event-emitter';

@Module({
  imports: [
    EventEmitterModule.forRoot({
      wildcard: false,        // Support wildcards (e.g., 'user.*')
      delimiter: '.',         // Event name delimiter
      newListener: false,
      removeListener: false,
      maxListeners: 10,       // Max listeners per event
      verboseMemoryLeak: false,
      ignoreErrors: false,
    }),
  ],
})
export class AppModule {}

// ========== EMIT EVENTS ==========
// users/services/users.service.ts
import { Injectable } from '@nestjs/common';
import { EventEmitter2 } from '@nestjs/event-emitter';

@Injectable()
export class UsersService {
  constructor(
    private usersRepository: UsersRepository,
    private eventEmitter: EventEmitter2,
  ) {}

  async create(dto: CreateUserDto): Promise<User> {
    const user = await this.usersRepository.save(dto);

    // Emit event with payload
    this.eventEmitter.emit('user.created', {
      userId: user.id,
      email: user.email,
      name: user.name,
      timestamp: new Date(),
    });

    return user;
  }

  async update(id: string, dto: UpdateUserDto): Promise<User> {
    const user = await this.usersRepository.update(id, dto);

    // Emit update event
    this.eventEmitter.emit('user.updated', {
      userId: user.id,
      changes: dto,
      timestamp: new Date(),
    });

    return user;
  }

  async remove(id: string): Promise<void> {
    await this.usersRepository.remove(id);

    // Emit delete event
    this.eventEmitter.emit('user.deleted', {
      userId: id,
      timestamp: new Date(),
    });
  }
}

// ========== LISTEN TO EVENTS ==========
// users/listeners/user-events.listener.ts
import { Injectable } from '@nestjs/common';
import { OnEvent } from '@nestjs/event-emitter';

@Injectable()
export class UserEventsListener {
  constructor(
    private emailService: EmailService,
    private logger: LoggerService,
  ) {}

  @OnEvent('user.created')
  async handleUserCreated(payload: any) {
    this.logger.log(`User created: ${payload.userId}`, 'UserEventsListener');
    
    // Send welcome email
    await this.emailService.sendWelcomeEmail(payload.email);
  }

  @OnEvent('user.updated')
  async handleUserUpdated(payload: any) {
    this.logger.log(`User updated: ${payload.userId}`, 'UserEventsListener');
    
    // Send update confirmation email
    await this.emailService.sendUpdateConfirmation(payload.userId);
  }

  @OnEvent('user.deleted')
  async handleUserDeleted(payload: any) {
    this.logger.log(`User deleted: ${payload.userId}`, 'UserEventsListener');
    
    // Clean up user data
    await this.cleanupUserData(payload.userId);
  }

  private async cleanupUserData(userId: string): Promise<void> {
    // Delete user files, sessions, etc.
  }
}

// users/users.module.ts
@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UsersController],
  providers: [
    UsersService,
    UsersRepository,
    UserEventsListener,  // Register listener
  ],
})
export class UsersModule {}
```

---

### **Method 2: Multiple Listeners for Same Event**

```typescript
// ========== MULTIPLE LISTENERS ==========
// listeners/email.listener.ts
@Injectable()
export class EmailListener {
  constructor(private emailService: EmailService) {}

  @OnEvent('user.created')
  async sendWelcomeEmail(payload: any) {
    await this.emailService.sendWelcomeEmail(payload.email);
  }
}

// listeners/analytics.listener.ts
@Injectable()
export class AnalyticsListener {
  constructor(private analyticsService: AnalyticsService) {}

  @OnEvent('user.created')
  async trackUserCreation(payload: any) {
    await this.analyticsService.track('user_created', {
      userId: payload.userId,
      timestamp: payload.timestamp,
    });
  }
}

// listeners/notification.listener.ts
@Injectable()
export class NotificationListener {
  constructor(private notificationService: NotificationService) {}

  @OnEvent('user.created')
  async notifyAdmin(payload: any) {
    await this.notificationService.sendToAdmin(
      'New User',
      `User ${payload.name} (${payload.email}) registered`,
    );
  }
}

// listeners/audit.listener.ts
@Injectable()
export class AuditListener {
  constructor(private auditService: AuditService) {}

  @OnEvent('user.created')
  async logUserCreation(payload: any) {
    await this.auditService.log({
      action: 'USER_CREATED',
      entityId: payload.userId,
      timestamp: payload.timestamp,
      metadata: payload,
    });
  }
}

// All 4 listeners called automatically when 'user.created' emitted!
// Add/remove listeners without touching UsersService
```

---

### **Method 3: Async vs Sync Listeners**

```typescript
// ========== SYNC LISTENERS (default) ==========
@Injectable()
export class SyncListener {
  @OnEvent('order.created')
  handleOrderCreated(payload: any) {
    console.log('Sync listener 1');
    // Blocks until complete
  }

  @OnEvent('order.created')
  handleOrderCreated2(payload: any) {
    console.log('Sync listener 2');
    // Runs after listener 1
  }
}

// Order: listener 1 → listener 2 → return to caller

// ========== ASYNC LISTENERS ==========
@Injectable()
export class AsyncListener {
  @OnEvent('order.created', { async: true })
  async handleOrderCreated(payload: any) {
    console.log('Async listener 1');
    await this.processOrder(payload);
    // Doesn't block other listeners
  }

  @OnEvent('order.created', { async: true })
  async handleOrderCreated2(payload: any) {
    console.log('Async listener 2');
    await this.sendEmail(payload);
    // Runs in parallel with listener 1
  }
}

// Order: listener 1 and 2 run in parallel → return to caller immediately

// ========== WHEN TO USE ==========
// Sync listeners:
// ✅ Critical operations that must complete before continuing
// ✅ Need to ensure order of execution
// ⚠️ Blocks caller until all listeners done

// Async listeners:
// ✅ Non-critical side effects (email, analytics)
// ✅ Long-running operations
// ✅ Don't want to block main flow
// ⚠️ No guarantee of completion order
```

---

### **Method 4: Event Classes (Type Safety)**

```typescript
// ========== TYPED EVENTS ==========
// events/user-created.event.ts
export class UserCreatedEvent {
  constructor(
    public readonly userId: string,
    public readonly email: string,
    public readonly name: string,
    public readonly timestamp: Date,
  ) {}
}

// events/order-placed.event.ts
export class OrderPlacedEvent {
  constructor(
    public readonly orderId: string,
    public readonly userId: string,
    public readonly total: number,
    public readonly items: OrderItem[],
    public readonly timestamp: Date,
  ) {}
}

// users/services/users.service.ts
@Injectable()
export class UsersService {
  constructor(private eventEmitter: EventEmitter2) {}

  async create(dto: CreateUserDto): Promise<User> {
    const user = await this.usersRepository.save(dto);

    // Emit typed event
    this.eventEmitter.emit(
      'user.created',
      new UserCreatedEvent(user.id, user.email, user.name, new Date()),
    );

    return user;
  }
}

// listeners/user-events.listener.ts
@Injectable()
export class UserEventsListener {
  @OnEvent('user.created')
  async handleUserCreated(event: UserCreatedEvent) {
    // TypeScript knows event structure!
    console.log(`User created: ${event.userId}`);
    console.log(`Email: ${event.email}`);
    console.log(`Name: ${event.name}`);
    console.log(`Timestamp: ${event.timestamp}`);
  }
}

// Benefits:
// ✅ Type safety
// ✅ Autocomplete in IDE
// ✅ Refactoring support
// ✅ Clear event structure
```

---

### **Method 5: Wildcard Listeners**

```typescript
// ========== WILDCARD LISTENERS ==========
// Enable wildcards in module:
EventEmitterModule.forRoot({
  wildcard: true,
  delimiter: '.',
})

// Emit specific events:
this.eventEmitter.emit('user.created', user);
this.eventEmitter.emit('user.updated', user);
this.eventEmitter.emit('user.deleted', user);
this.eventEmitter.emit('order.created', order);
this.eventEmitter.emit('order.shipped', order);

// Listen to all user events:
@Injectable()
export class UserWildcardListener {
  @OnEvent('user.*')  // Matches: user.created, user.updated, user.deleted
  async handleAnyUserEvent(payload: any) {
    console.log('User event occurred:', payload);
    await this.logEvent('user', payload);
  }
}

// Listen to all events:
@Injectable()
export class GlobalListener {
  @OnEvent('**')  // Matches ALL events
  async handleAllEvents(payload: any) {
    console.log('Event occurred:', payload);
    await this.logToDatabase(payload);
  }
}

// Listen to specific pattern:
@Injectable()
export class PatternListener {
  @OnEvent('*.created')  // Matches: user.created, order.created, product.created
  async handleCreationEvents(payload: any) {
    console.log('Something was created:', payload);
    await this.trackCreation(payload);
  }
}
```

---

### **Observer Pattern Benefits:**

| Benefit | Description | Example |
|---------|-------------|---------|
| **Decoupling** | Subjects don't know observers | UsersService doesn't know listeners |
| **Extensibility** | Add observers without modifying subject | Add new listener, don't touch service |
| **Single Responsibility** | Each observer has one job | EmailListener only sends emails |
| **Async Processing** | Side effects don't block main flow | Email sent asynchronously |
| **Scalability** | Easy to add/remove observers | Add analytics without code changes |
| **Testability** | Test observers independently | Mock event emitter |

---

### **Common Use Cases:**

```typescript
// ✅ User registration flow
this.eventEmitter.emit('user.registered', user);
// Listeners: send welcome email, create profile, track analytics, notify admin

// ✅ Order processing
this.eventEmitter.emit('order.placed', order);
// Listeners: send confirmation email, update inventory, notify warehouse, track sale

// ✅ File upload
this.eventEmitter.emit('file.uploaded', file);
// Listeners: generate thumbnail, scan for viruses, update quota, notify owner

// ✅ Payment processing
this.eventEmitter.emit('payment.received', payment);
// Listeners: send receipt, update order status, trigger fulfillment, log transaction

// ✅ Audit logging
this.eventEmitter.emit('**', action);  // Listen to all events
// Listener: log all actions to audit trail

// ✅ Cache invalidation
this.eventEmitter.emit('user.updated', user);
// Listener: clear user cache
```

---

### **Best Practices:**

```typescript
// ✅ GOOD: Use typed event classes
this.eventEmitter.emit('user.created', new UserCreatedEvent(user));

// ❌ BAD: Use plain objects
this.eventEmitter.emit('user.created', { id: user.id });  // No type safety

// ✅ GOOD: Use meaningful event names
this.eventEmitter.emit('user.created', user);
this.eventEmitter.emit('order.placed', order);
this.eventEmitter.emit('payment.received', payment);

// ❌ BAD: Generic event names
this.eventEmitter.emit('event', data);
this.eventEmitter.emit('update', user);

// ✅ GOOD: One listener class per concern
@Injectable()
export class EmailListener {
  @OnEvent('user.created')
  sendWelcomeEmail() {}
}

@Injectable()
export class AnalyticsListener {
  @OnEvent('user.created')
  trackUserCreation() {}
}

// ❌ BAD: One listener for everything
@Injectable()
export class MegaListener {
  @OnEvent('user.created')
  handleUserCreated() {
    this.sendEmail();
    this.trackAnalytics();
    this.notifyAdmin();
    this.updateCache();
    // Too many responsibilities!
  }
}

// ✅ GOOD: Async for side effects
@OnEvent('user.created', { async: true })
async sendWelcomeEmail() {}

// ❌ BAD: Sync for long-running operations
@OnEvent('user.created')  // Blocks until email sent!
async sendWelcomeEmail() {}

// ✅ GOOD: Handle errors in listeners
@OnEvent('user.created')
async handleUserCreated(event: UserCreatedEvent) {
  try {
    await this.emailService.send(event.email);
  } catch (error) {
    this.logger.error('Failed to send email', error);
    // Don't throw - shouldn't break other listeners
  }
}

// ❌ BAD: Let errors propagate
@OnEvent('user.created')
async handleUserCreated(event: UserCreatedEvent) {
  await this.emailService.send(event.email);  // Throws and breaks other listeners!
}
```

**Key Takeaway:** **Observer Pattern** implements **one-to-many dependency** where subjects notify all observers automatically when state changes. In NestJS, use `@nestjs/event-emitter`: services emit events with `eventEmitter.emit()`, listeners subscribe with `@OnEvent()` decorator. Benefits: **loose coupling** (services don't know listeners), **extensibility** (add listeners without modifying emitters), **Single Responsibility** (one listener per concern), **async processing** (side effects don't block), and **scalability** (easy to add/remove observers). Use for: **user registration** (email, analytics, notifications), **order processing** (inventory, fulfillment), **audit logging**, **cache invalidation**, and **CQRS**. Implement with typed event classes for type safety, use `async: true` for non-critical operations, and handle errors in listeners to avoid breaking other observers.

</details>

## Domain-Driven Design (DDD)

<details>
<summary><strong>17. What is Domain-Driven Design?</strong></summary>

**Answer:**

**Domain-Driven Design (DDD)** is a software development approach focusing on **complex business logic** by modeling software based on the **business domain**. Core concepts: **Ubiquitous Language** (shared vocabulary), **Bounded Contexts** (separate models per domain), **Entities** (objects with identity), **Value Objects** (immutable descriptive objects), **Aggregates** (clusters of entities), **Domain Services** (business logic), and **Repositories** (persistence abstraction). DDD helps manage complexity in large systems with rich business rules.

---

### **DDD Core Concepts:**

```
┌─────────────────────────────────────────────────────────┐
│                  DOMAIN-DRIVEN DESIGN                   │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────────────┐     ┌──────────────────┐        │
│  │ Ubiquitous       │────▶│  Bounded         │        │
│  │ Language         │     │  Context         │        │
│  │ (Shared vocab)   │     │  (Model boundary)│        │
│  └──────────────────┘     └──────────────────┘        │
│           │                         │                  │
│           │                         │                  │
│           ▼                         ▼                  │
│  ┌──────────────────────────────────────────┐         │
│  │         DOMAIN MODEL                     │         │
│  │                                          │         │
│  │  ┌────────┐  ┌────────┐  ┌──────────┐  │         │
│  │  │Entity  │  │Value   │  │Aggregate │  │         │
│  │  │(ID)    │  │Object  │  │(Root)    │  │         │
│  │  └────────┘  └────────┘  └──────────┘  │         │
│  │                                          │         │
│  │  ┌────────┐  ┌────────┐  ┌──────────┐  │         │
│  │  │Domain  │  │Domain  │  │Factory   │  │         │
│  │  │Service │  │Event   │  │          │  │         │
│  │  └────────┘  └────────┘  └──────────┘  │         │
│  └──────────────────────────────────────────┘         │
│           │                                            │
│           ▼                                            │
│  ┌──────────────────────────────────────────┐         │
│  │       INFRASTRUCTURE                     │         │
│  │  ┌────────┐                ┌──────────┐  │         │
│  │  │Reposit │                │Database  │  │         │
│  │  │-ory    │───────────────▶│          │  │         │
│  │  └────────┘                └──────────┘  │         │
│  └──────────────────────────────────────────┘         │
└─────────────────────────────────────────────────────────┘
```

---

### **Method 1: Ubiquitous Language**

```typescript
// ========== UBIQUITOUS LANGUAGE ==========
// Shared vocabulary between developers and domain experts

// ❌ BAD: Technical language
class UserData {
  id: string;
  field1: string;  // What is field1?
  field2: number;  // What is field2?
  status: number;  // 0 = ?, 1 = ?
}

class UserManager {
  processUser(user: UserData): void {
    // Technical terms, not business terms
  }
}

// ✅ GOOD: Ubiquitous language (business terms)
class Customer {
  customerId: string;
  email: string;
  loyaltyPoints: number;
  membershipStatus: MembershipStatus;
}

enum MembershipStatus {
  BRONZE = 'bronze',
  SILVER = 'silver',
  GOLD = 'gold',
  PLATINUM = 'platinum',
}

class CustomerService {
  rewardLoyaltyPoints(customer: Customer, points: number): void {
    // Business language: "reward loyalty points"
    customer.loyaltyPoints += points;
    
    // Business rule: upgrade membership
    if (customer.loyaltyPoints > 1000 && customer.membershipStatus === MembershipStatus.SILVER) {
      customer.upgradeMembership(MembershipStatus.GOLD);
    }
  }

  applyMembershipDiscount(customer: Customer, orderTotal: number): number {
    // Business language: "apply membership discount"
    const discountRate = this.getDiscountRate(customer.membershipStatus);
    return orderTotal * (1 - discountRate);
  }

  private getDiscountRate(status: MembershipStatus): number {
    // Business rules in code
    switch (status) {
      case MembershipStatus.BRONZE: return 0.05;  // 5% discount
      case MembershipStatus.SILVER: return 0.10;  // 10% discount
      case MembershipStatus.GOLD: return 0.15;    // 15% discount
      case MembershipStatus.PLATINUM: return 0.20; // 20% discount
    }
  }
}

// Terms match business domain:
// - Customer (not User)
// - Loyalty Points (not points)
// - Membership Status (not user level)
// - Reward (not add)
// - Apply Discount (not calculate)
```

---

### **Method 2: Bounded Contexts**

```typescript
// ========== BOUNDED CONTEXTS ==========
// Separate models for different business domains

// E-COMMERCE BOUNDED CONTEXTS:

// ┌──────────────────────────────────────────┐
// │     CATALOG CONTEXT                      │
// │  (Product browsing & search)             │
// └──────────────────────────────────────────┘
// catalog/domain/product.entity.ts
export class CatalogProduct {
  productId: string;
  name: string;
  description: string;
  price: number;
  stock: number;
  category: string;
  images: string[];
  
  isAvailable(): boolean {
    return this.stock > 0;
  }
}

// ┌──────────────────────────────────────────┐
// │     ORDERING CONTEXT                     │
// │  (Cart, checkout, order placement)       │
// └──────────────────────────────────────────┘
// ordering/domain/order-line-item.entity.ts
export class OrderLineItem {
  productId: string;         // Reference to product
  productName: string;       // Snapshot at order time
  priceAtOrder: number;      // Price when ordered (not current price!)
  quantity: number;
  
  getSubtotal(): number {
    return this.priceAtOrder * this.quantity;
  }
}

// ordering/domain/order.entity.ts
export class Order {
  orderId: string;
  customerId: string;
  items: OrderLineItem[];
  status: OrderStatus;
  total: number;
  placedAt: Date;
  
  placeOrder(): void {
    if (this.items.length === 0) {
      throw new Error('Cannot place order with no items');
    }
    this.status = OrderStatus.PLACED;
    this.placedAt = new Date();
  }
  
  calculateTotal(): number {
    return this.items.reduce((sum, item) => sum + item.getSubtotal(), 0);
  }
}

// ┌──────────────────────────────────────────┐
// │     INVENTORY CONTEXT                    │
// │  (Stock management, warehouses)          │
// └──────────────────────────────────────────┘
// inventory/domain/inventory-item.entity.ts
export class InventoryItem {
  productId: string;
  warehouseId: string;
  quantityOnHand: number;
  quantityReserved: number;
  reorderPoint: number;
  
  getAvailableQuantity(): number {
    return this.quantityOnHand - this.quantityReserved;
  }
  
  reserve(quantity: number): void {
    if (this.getAvailableQuantity() < quantity) {
      throw new Error('Insufficient stock');
    }
    this.quantityReserved += quantity;
  }
  
  needsReorder(): boolean {
    return this.quantityOnHand <= this.reorderPoint;
  }
}

// ┌──────────────────────────────────────────┐
// │     SHIPPING CONTEXT                     │
// │  (Fulfillment, delivery tracking)        │
// └──────────────────────────────────────────┘
// shipping/domain/shipment.entity.ts
export class Shipment {
  shipmentId: string;
  orderId: string;
  trackingNumber: string;
  carrier: string;
  status: ShipmentStatus;
  estimatedDelivery: Date;
  
  ship(): void {
    this.status = ShipmentStatus.SHIPPED;
  }
  
  markDelivered(): void {
    this.status = ShipmentStatus.DELIVERED;
  }
}

// Different contexts have different Product representations!
// Catalog: full details, images, descriptions
// Ordering: snapshot of product at order time
// Inventory: stock levels, warehouses
// Shipping: weight, dimensions for shipping
```

---

### **Method 3: Layered Architecture in DDD**

```typescript
// ========== DDD LAYERED ARCHITECTURE ==========

/*
┌─────────────────────────────────────────┐
│   PRESENTATION LAYER (API)              │
│   - Controllers                         │
│   - DTOs                                │
│   - Validation                          │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│   APPLICATION LAYER                     │
│   - Use Cases / Application Services    │
│   - Orchestration                       │
│   - Transaction Management              │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│   DOMAIN LAYER                          │
│   - Entities                            │
│   - Value Objects                       │
│   - Domain Services                     │
│   - Domain Events                       │
│   - Aggregates                          │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│   INFRASTRUCTURE LAYER                  │
│   - Repositories (implementations)      │
│   - Database                            │
│   - External Services                   │
└─────────────────────────────────────────┘
*/

// PRESENTATION LAYER
// orders/controllers/orders.controller.ts
@Controller('orders')
export class OrdersController {
  constructor(private placeOrderUseCase: PlaceOrderUseCase) {}

  @Post()
  async placeOrder(@Body() dto: PlaceOrderDto) {
    return this.placeOrderUseCase.execute(dto);
  }
}

// orders/dto/place-order.dto.ts
export class PlaceOrderDto {
  @IsString()
  customerId: string;

  @IsArray()
  @ValidateNested({ each: true })
  items: OrderItemDto[];
}

// APPLICATION LAYER
// orders/application/place-order.use-case.ts
@Injectable()
export class PlaceOrderUseCase {
  constructor(
    private ordersRepository: IOrdersRepository,
    private inventoryService: InventoryDomainService,
    private eventBus: EventBus,
  ) {}

  async execute(dto: PlaceOrderDto): Promise<Order> {
    // 1. Create order (domain logic)
    const order = Order.create(dto.customerId, dto.items);

    // 2. Check inventory (domain service)
    await this.inventoryService.reserveItems(dto.items);

    // 3. Place order (domain method)
    order.placeOrder();

    // 4. Save (infrastructure)
    await this.ordersRepository.save(order);

    // 5. Emit event
    this.eventBus.publish(new OrderPlacedEvent(order));

    return order;
  }
}

// DOMAIN LAYER
// orders/domain/order.entity.ts
export class Order {
  private orderId: string;
  private customerId: string;
  private items: OrderLineItem[];
  private status: OrderStatus;
  private total: number;
  private placedAt: Date;

  private constructor(data: any) {
    this.orderId = data.orderId;
    this.customerId = data.customerId;
    this.items = data.items;
    this.status = OrderStatus.DRAFT;
  }

  // Factory method
  static create(customerId: string, items: OrderItemDto[]): Order {
    const order = new Order({
      orderId: uuid(),
      customerId,
      items: items.map(item => OrderLineItem.create(item)),
    });
    
    order.calculateTotal();
    
    return order;
  }

  // Domain logic
  placeOrder(): void {
    if (this.items.length === 0) {
      throw new DomainException('Cannot place order with no items');
    }
    
    if (this.total <= 0) {
      throw new DomainException('Order total must be positive');
    }

    this.status = OrderStatus.PLACED;
    this.placedAt = new Date();
  }

  addItem(item: OrderLineItem): void {
    this.items.push(item);
    this.calculateTotal();
  }

  removeItem(productId: string): void {
    this.items = this.items.filter(item => item.productId !== productId);
    this.calculateTotal();
  }

  private calculateTotal(): void {
    this.total = this.items.reduce((sum, item) => sum + item.getSubtotal(), 0);
  }

  // Getters (no setters - immutability)
  getTotal(): number {
    return this.total;
  }

  getStatus(): OrderStatus {
    return this.status;
  }
}

// orders/domain/services/inventory-domain.service.ts
@Injectable()
export class InventoryDomainService {
  async reserveItems(items: OrderItemDto[]): Promise<void> {
    // Complex business logic that doesn't belong to a single entity
    for (const item of items) {
      const inventoryItem = await this.inventoryRepository.findByProductId(item.productId);
      
      if (!inventoryItem) {
        throw new DomainException(`Product ${item.productId} not found in inventory`);
      }

      inventoryItem.reserve(item.quantity);
      await this.inventoryRepository.save(inventoryItem);
    }
  }
}

// INFRASTRUCTURE LAYER
// orders/infrastructure/typeorm-orders.repository.ts
@Injectable()
export class TypeOrmOrdersRepository implements IOrdersRepository {
  constructor(
    @InjectRepository(OrderEntity)
    private repository: Repository<OrderEntity>,
  ) {}

  async save(order: Order): Promise<void> {
    const entity = this.toEntity(order);
    await this.repository.save(entity);
  }

  async findById(id: string): Promise<Order | null> {
    const entity = await this.repository.findOne({ where: { id } });
    return entity ? this.toDomain(entity) : null;
  }

  private toEntity(order: Order): OrderEntity {
    // Map domain to persistence
  }

  private toDomain(entity: OrderEntity): Order {
    // Map persistence to domain
  }
}
```

---

### **Method 4: DDD in NestJS Structure**

```typescript
// ========== DDD FOLDER STRUCTURE ==========
src/
├── orders/                              # Bounded Context
│   ├── orders.module.ts
│   ├── controllers/                     # Presentation Layer
│   │   └── orders.controller.ts
│   ├── application/                     # Application Layer
│   │   ├── use-cases/
│   │   │   ├── place-order.use-case.ts
│   │   │   ├── cancel-order.use-case.ts
│   │   │   └── get-order.use-case.ts
│   │   └── dto/
│   │       ├── place-order.dto.ts
│   │       └── order-response.dto.ts
│   ├── domain/                          # Domain Layer
│   │   ├── entities/
│   │   │   ├── order.entity.ts
│   │   │   └── order-line-item.entity.ts
│   │   ├── value-objects/
│   │   │   ├── money.value-object.ts
│   │   │   └── address.value-object.ts
│   │   ├── services/
│   │   │   └── pricing-domain.service.ts
│   │   ├── events/
│   │   │   ├── order-placed.event.ts
│   │   │   └── order-cancelled.event.ts
│   │   ├── repositories/
│   │   │   └── orders-repository.interface.ts
│   │   └── exceptions/
│   │       └── order.exceptions.ts
│   └── infrastructure/                  # Infrastructure Layer
│       ├── repositories/
│       │   └── typeorm-orders.repository.ts
│       └── persistence/
│           └── order.entity.ts          # TypeORM entity
├── customers/                           # Another Bounded Context
│   ├── controllers/
│   ├── application/
│   ├── domain/
│   └── infrastructure/
└── shared/                              # Shared Kernel
    ├── domain/
    │   ├── base-entity.ts
    │   ├── domain-event.ts
    │   └── value-object.ts
    └── infrastructure/
        └── event-bus.ts
```

---

### **Method 5: When to Use DDD**

```typescript
// ========== USE DDD WHEN: ==========

// ✅ Complex business logic
// Example: Insurance policy calculations, loan approvals, pricing engines
export class LoanApplication {
  calculateMonthlyPayment(): Money {
    // Complex calculation based on:
    // - Credit score
    // - Loan amount
    // - Interest rate
    // - Term length
    // - Down payment
    // - Insurance
    // - Taxes
  }

  approve(): void {
    // Complex business rules:
    // - Income verification
    // - Debt-to-income ratio
    // - Credit history
    // - Employment history
    // - Collateral valuation
  }
}

// ✅ Multiple domain experts involved
// Example: Healthcare system (doctors, nurses, billing, insurance)
// Each context has its own model and language

// ✅ Long-term project with evolving requirements
// DDD helps manage complexity as project grows

// ✅ Large team working on same codebase
// Bounded contexts allow teams to work independently

// ❌ DON'T USE DDD WHEN:

// ❌ Simple CRUD application
export class UsersService {
  async create(dto: CreateUserDto): Promise<User> {
    return this.usersRepository.save(dto);
  }
  
  async findAll(): Promise<User[]> {
    return this.usersRepository.findAll();
  }
  
  // No complex business logic - DDD overhead not worth it
}

// ❌ Small project (< 3 months)
// DDD adds complexity - use simple layered architecture

// ❌ Prototype or MVP
// Iterate quickly first, apply DDD later if needed

// ❌ No domain experts available
// DDD requires collaboration with business
```

---

### **DDD Benefits:**

| Benefit | Description | Example |
|---------|-------------|---------|
| **Ubiquitous Language** | Shared vocabulary between devs and business | Order.place() not Order.setStatus(1) |
| **Bounded Contexts** | Separate models per domain | Catalog Product ≠ Inventory Product |
| **Business Logic Centralized** | Domain layer contains all rules | Order.validate() in entity |
| **Testable** | Domain logic isolated from infrastructure | Test entities without database |
| **Maintainable** | Clear structure and boundaries | Easy to find and modify logic |
| **Scalable** | Contexts can become microservices | OrderContext → Order Service |

---

### **Best Practices:**

```typescript
// ✅ GOOD: Rich domain model with behavior
export class Order {
  placeOrder(): void { /* logic */ }
  cancel(): void { /* logic */ }
  addItem(item: OrderLineItem): void { /* logic */ }
}

// ❌ BAD: Anemic domain model (just data)
export class Order {
  status: string;
  items: any[];
  // No methods, just getters/setters
}

// ✅ GOOD: Ubiquitous language
customer.rewardLoyaltyPoints(100);
order.placeOrder();
shipment.markDelivered();

// ❌ BAD: Technical language
customer.addPoints(100);
order.setStatus(1);
shipment.status = 'delivered';

// ✅ GOOD: Bounded contexts
CatalogProduct (browsing)
OrderLineItem (ordering)
InventoryItem (stock)

// ❌ BAD: One product model for everything
Product (tries to serve all contexts)

// ✅ GOOD: Domain services for cross-entity logic
@Injectable()
export class PricingDomainService {
  calculatePrice(product: Product, customer: Customer): Money {}
}

// ❌ BAD: Logic scattered across entities
product.calculatePriceForCustomer(customer);  // Wrong responsibility

// ✅ GOOD: Value objects for concepts
export class Money {
  constructor(readonly amount: number, readonly currency: string) {}
  add(other: Money): Money {}
}

// ❌ BAD: Primitives everywhere
price: number;  // What currency? Can be negative?
```

**Key Takeaway:** **Domain-Driven Design (DDD)** models software based on **business domain** to manage complexity in large systems. Core concepts: **Ubiquitous Language** (shared business vocabulary in code), **Bounded Contexts** (separate models per domain), **Entities** (objects with identity), **Value Objects** (immutable descriptive objects), **Aggregates** (entity clusters), **Domain Services** (cross-entity logic), and **Repositories** (persistence abstraction). Use **layered architecture**: Presentation (controllers, DTOs) → Application (use cases, orchestration) → Domain (entities, business rules) → Infrastructure (database, external services). Use DDD for: **complex business logic**, **multiple domain experts**, **long-term projects**, **large teams**. Don't use for: **simple CRUD**, **small projects**, **prototypes**, **no domain experts**. Benefits: clear structure, testable domain logic, maintainable, scalable to microservices.

</details>

<details>
<summary><strong>18. What are Entities, Value Objects, and Aggregates?</strong></summary>

**Answer:**

**Entities** are objects with **unique identity** that persists over time (User, Order, Product with ID). **Value Objects** are **immutable objects** defined by their attributes, not identity (Money, Address, Email - two $10 are same). **Aggregates** are **clusters of entities** with one root entity enforcing consistency boundaries (Order aggregate contains OrderLineItems, Order is root). Entities have lifecycle, Value Objects don't. Aggregates ensure business rules across multiple objects.

---

### **Entities vs Value Objects vs Aggregates:**

```typescript
// ========== ENTITIES: Objects with Identity ==========
// Same identity = same object, even if attributes change

export class User {
  // ✅ HAS IDENTITY (userId)
  private userId: string;
  private email: string;
  private name: string;
  private age: number;

  constructor(userId: string, email: string, name: string, age: number) {
    this.userId = userId;
    this.email = email;
    this.name = name;
    this.age = age;
  }

  // Identity-based equality
  equals(other: User): boolean {
    return this.userId === other.userId;
  }

  // Can change attributes, identity remains
  changeName(newName: string): void {
    this.name = newName;
  }

  changeEmail(newEmail: string): void {
    this.email = newEmail;
  }
}

// Example:
const user1 = new User('123', 'john@example.com', 'John', 30);
const user2 = new User('123', 'john@example.com', 'John', 30);
user1.equals(user2);  // true (same ID)

user1.changeName('John Smith');
user1.equals(user2);  // still true (same ID, different name doesn't matter)

// ========== VALUE OBJECTS: Defined by attributes, no identity ==========
// Same attributes = same object

export class Money {
  // ❌ NO IDENTITY
  private readonly amount: number;
  private readonly currency: string;

  constructor(amount: number, currency: string) {
    if (amount < 0) {
      throw new Error('Amount cannot be negative');
    }
    this.amount = amount;
    this.currency = currency;
  }

  // Value-based equality
  equals(other: Money): boolean {
    return this.amount === other.amount && this.currency === other.currency;
  }

  // Immutable - returns new instance
  add(other: Money): Money {
    if (this.currency !== other.currency) {
      throw new Error('Cannot add different currencies');
    }
    return new Money(this.amount + other.amount, this.currency);
  }

  multiply(factor: number): Money {
    return new Money(this.amount * factor, this.currency);
  }

  getAmount(): number {
    return this.amount;
  }

  getCurrency(): string {
    return this.currency;
  }
}

// Example:
const money1 = new Money(10, 'USD');
const money2 = new Money(10, 'USD');
money1.equals(money2);  // true (same amount and currency)

const money3 = money1.add(money2);  // New instance: Money(20, 'USD')
money1.equals(money3);  // false (different amounts)

// ========== AGGREGATES: Cluster with one root ==========
// Order (root) + OrderLineItems (children)

export class Order {
  // Order is AGGREGATE ROOT
  private orderId: string;
  private customerId: string;
  private items: OrderLineItem[];  // Children
  private status: OrderStatus;
  private total: Money;

  // Only Order can modify its items
  addItem(productId: string, quantity: number, price: Money): void {
    const item = new OrderLineItem(productId, quantity, price);
    this.items.push(item);
    this.recalculateTotal();
  }

  removeItem(productId: string): void {
    this.items = this.items.filter(item => item.productId !== productId);
    this.recalculateTotal();
  }

  // Aggregate ensures consistency
  private recalculateTotal(): void {
    this.total = this.items.reduce(
      (sum, item) => sum.add(item.getSubtotal()),
      new Money(0, 'USD'),
    );
  }

  // Business rules enforced by aggregate
  placeOrder(): void {
    if (this.items.length === 0) {
      throw new Error('Cannot place order with no items');
    }
    this.status = OrderStatus.PLACED;
  }
}

// OrderLineItem is NOT accessible outside Order
class OrderLineItem {
  constructor(
    readonly productId: string,
    readonly quantity: number,
    readonly price: Money,
  ) {}

  getSubtotal(): Money {
    return this.price.multiply(this.quantity);
  }
}

// ❌ DON'T access children directly:
// orderLineItemRepository.save(item);  // Wrong!

// ✅ DO access through aggregate root:
order.addItem('product-123', 2, new Money(10, 'USD'));  // Right!
orderRepository.save(order);  // Saves order and all items
```

---

### **Method 1: Entity Examples**

```typescript
// ========== ENTITY: USER ==========
// users/domain/user.entity.ts
export class User {
  private userId: string;
  private email: string;
  private name: string;
  private passwordHash: string;
  private registeredAt: Date;
  private lastLoginAt: Date;

  constructor(data: UserData) {
    this.userId = data.userId;
    this.email = data.email;
    this.name = data.name;
    this.passwordHash = data.passwordHash;
    this.registeredAt = data.registeredAt || new Date();
  }

  // Identity
  getId(): string {
    return this.userId;
  }

  equals(other: User): boolean {
    return this.userId === other.userId;
  }

  // Behavior (entity has business logic)
  changeEmail(newEmail: string): void {
    if (!this.isValidEmail(newEmail)) {
      throw new DomainException('Invalid email format');
    }
    this.email = newEmail;
  }

  updatePassword(oldPassword: string, newPassword: string): void {
    if (!this.verifyPassword(oldPassword)) {
      throw new DomainException('Invalid old password');
    }
    this.passwordHash = this.hashPassword(newPassword);
  }

  recordLogin(): void {
    this.lastLoginAt = new Date();
  }

  private isValidEmail(email: string): boolean {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }

  private verifyPassword(password: string): boolean {
    return bcrypt.compareSync(password, this.passwordHash);
  }

  private hashPassword(password: string): string {
    return bcrypt.hashSync(password, 10);
  }
}

// ========== ENTITY: PRODUCT ==========
// products/domain/product.entity.ts
export class Product {
  private productId: string;
  private name: string;
  private description: string;
  private price: Money;
  private stock: number;
  private category: string;

  constructor(data: ProductData) {
    this.productId = data.productId;
    this.name = data.name;
    this.description = data.description;
    this.price = data.price;
    this.stock = data.stock;
    this.category = data.category;
  }

  getId(): string {
    return this.productId;
  }

  // Business logic
  updatePrice(newPrice: Money): void {
    if (newPrice.getAmount() <= 0) {
      throw new DomainException('Price must be positive');
    }
    this.price = newPrice;
  }

  reduceStock(quantity: number): void {
    if (quantity > this.stock) {
      throw new DomainException('Insufficient stock');
    }
    this.stock -= quantity;
  }

  increaseStock(quantity: number): void {
    this.stock += quantity;
  }

  isAvailable(): boolean {
    return this.stock > 0;
  }

  getPrice(): Money {
    return this.price;
  }
}
```

---

### **Method 2: Value Object Examples**

```typescript
// ========== VALUE OBJECT: MONEY ==========
// shared/domain/value-objects/money.value-object.ts
export class Money {
  private readonly amount: number;
  private readonly currency: string;

  constructor(amount: number, currency: string) {
    if (amount < 0) {
      throw new Error('Amount cannot be negative');
    }
    if (!['USD', 'EUR', 'GBP'].includes(currency)) {
      throw new Error('Invalid currency');
    }
    this.amount = amount;
    this.currency = currency;
  }

  // Immutable operations (return new instances)
  add(other: Money): Money {
    this.validateCurrency(other);
    return new Money(this.amount + other.amount, this.currency);
  }

  subtract(other: Money): Money {
    this.validateCurrency(other);
    return new Money(this.amount - other.amount, this.currency);
  }

  multiply(factor: number): Money {
    return new Money(this.amount * factor, this.currency);
  }

  divide(divisor: number): Money {
    if (divisor === 0) {
      throw new Error('Cannot divide by zero');
    }
    return new Money(this.amount / divisor, this.currency);
  }

  // Value equality
  equals(other: Money): boolean {
    return this.amount === other.amount && this.currency === other.currency;
  }

  isGreaterThan(other: Money): boolean {
    this.validateCurrency(other);
    return this.amount > other.amount;
  }

  private validateCurrency(other: Money): void {
    if (this.currency !== other.currency) {
      throw new Error('Cannot operate on different currencies');
    }
  }

  getAmount(): number {
    return this.amount;
  }

  getCurrency(): string {
    return this.currency;
  }

  toString(): string {
    return `${this.amount} ${this.currency}`;
  }
}

// Usage:
const price1 = new Money(10, 'USD');
const price2 = new Money(5, 'USD');
const total = price1.add(price2);  // Money(15, 'USD')
console.log(total.toString());  // "15 USD"

// ========== VALUE OBJECT: ADDRESS ==========
// shared/domain/value-objects/address.value-object.ts
export class Address {
  private readonly street: string;
  private readonly city: string;
  private readonly state: string;
  private readonly zipCode: string;
  private readonly country: string;

  constructor(street: string, city: string, state: string, zipCode: string, country: string) {
    if (!street || !city || !zipCode || !country) {
      throw new Error('Address fields cannot be empty');
    }
    this.street = street;
    this.city = city;
    this.state = state;
    this.zipCode = zipCode;
    this.country = country;
  }

  equals(other: Address): boolean {
    return (
      this.street === other.street &&
      this.city === other.city &&
      this.state === other.state &&
      this.zipCode === other.zipCode &&
      this.country === other.country
    );
  }

  toString(): string {
    return `${this.street}, ${this.city}, ${this.state} ${this.zipCode}, ${this.country}`;
  }

  getStreet(): string {
    return this.street;
  }

  getCity(): string {
    return this.city;
  }

  getZipCode(): string {
    return this.zipCode;
  }
}

// ========== VALUE OBJECT: EMAIL ==========
// shared/domain/value-objects/email.value-object.ts
export class Email {
  private readonly value: string;

  constructor(value: string) {
    if (!this.isValid(value)) {
      throw new Error('Invalid email format');
    }
    this.value = value.toLowerCase();
  }

  private isValid(email: string): boolean {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }

  equals(other: Email): boolean {
    return this.value === other.value;
  }

  getValue(): string {
    return this.value;
  }

  getDomain(): string {
    return this.value.split('@')[1];
  }

  toString(): string {
    return this.value;
  }
}
```

---

### **Method 3: Aggregate Examples**

```typescript
// ========== AGGREGATE: ORDER (Root) + ORDER LINE ITEMS ==========
// orders/domain/order.aggregate.ts
export class Order {
  // Aggregate Root
  private orderId: string;
  private customerId: string;
  private items: OrderLineItem[];  // Aggregate members
  private shippingAddress: Address;
  private billingAddress: Address;
  private status: OrderStatus;
  private total: Money;
  private placedAt: Date;

  private constructor(data: OrderData) {
    this.orderId = data.orderId;
    this.customerId = data.customerId;
    this.items = [];
    this.shippingAddress = data.shippingAddress;
    this.billingAddress = data.billingAddress;
    this.status = OrderStatus.DRAFT;
    this.total = new Money(0, 'USD');
  }

  // Factory method
  static create(customerId: string, shippingAddress: Address, billingAddress: Address): Order {
    return new Order({
      orderId: uuid(),
      customerId,
      shippingAddress,
      billingAddress,
    });
  }

  // Aggregate controls access to children
  addItem(productId: string, productName: string, quantity: number, price: Money): void {
    // Business rule: check if product already in order
    const existingItem = this.items.find(item => item.getProductId() === productId);
    
    if (existingItem) {
      existingItem.increaseQuantity(quantity);
    } else {
      const item = OrderLineItem.create(productId, productName, quantity, price);
      this.items.push(item);
    }

    this.recalculateTotal();
  }

  removeItem(productId: string): void {
    const index = this.items.findIndex(item => item.getProductId() === productId);
    
    if (index === -1) {
      throw new DomainException('Item not found in order');
    }

    this.items.splice(index, 1);
    this.recalculateTotal();
  }

  updateItemQuantity(productId: string, newQuantity: number): void {
    const item = this.items.find(item => item.getProductId() === productId);
    
    if (!item) {
      throw new DomainException('Item not found in order');
    }

    if (newQuantity <= 0) {
      this.removeItem(productId);
    } else {
      item.setQuantity(newQuantity);
      this.recalculateTotal();
    }
  }

  // Business invariant: Order must have items before placing
  placeOrder(): void {
    if (this.items.length === 0) {
      throw new DomainException('Cannot place order with no items');
    }

    if (this.status !== OrderStatus.DRAFT) {
      throw new DomainException('Order already placed');
    }

    this.status = OrderStatus.PLACED;
    this.placedAt = new Date();
  }

  cancel(): void {
    if (this.status === OrderStatus.SHIPPED) {
      throw new DomainException('Cannot cancel shipped order');
    }

    this.status = OrderStatus.CANCELLED;
  }

  // Aggregate ensures consistency
  private recalculateTotal(): void {
    this.total = this.items.reduce(
      (sum, item) => sum.add(item.getSubtotal()),
      new Money(0, 'USD'),
    );
  }

  // Getters
  getId(): string {
    return this.orderId;
  }

  getItems(): ReadonlyArray<OrderLineItem> {
    return this.items;  // Read-only access
  }

  getTotal(): Money {
    return this.total;
  }

  getStatus(): OrderStatus {
    return this.status;
  }
}

// orders/domain/order-line-item.entity.ts
export class OrderLineItem {
  // NOT aggregate root, belongs to Order
  private productId: string;
  private productName: string;
  private quantity: number;
  private price: Money;

  private constructor(productId: string, productName: string, quantity: number, price: Money) {
    if (quantity <= 0) {
      throw new DomainException('Quantity must be positive');
    }
    this.productId = productId;
    this.productName = productName;
    this.quantity = quantity;
    this.price = price;
  }

  static create(productId: string, productName: string, quantity: number, price: Money): OrderLineItem {
    return new OrderLineItem(productId, productName, quantity, price);
  }

  getSubtotal(): Money {
    return this.price.multiply(this.quantity);
  }

  increaseQuantity(amount: number): void {
    this.quantity += amount;
  }

  setQuantity(newQuantity: number): void {
    if (newQuantity <= 0) {
      throw new DomainException('Quantity must be positive');
    }
    this.quantity = newQuantity;
  }

  getProductId(): string {
    return this.productId;
  }

  getQuantity(): number {
    return this.quantity;
  }

  getPrice(): Money {
    return this.price;
  }
}

// Repository operates on AGGREGATE ROOT only
@Injectable()
export class OrdersRepository {
  async save(order: Order): Promise<void> {
    // Save order AND all its items together (consistency boundary)
  }

  async findById(orderId: string): Promise<Order | null> {
    // Load order WITH all its items
  }
}

// ❌ DON'T create separate repository for children:
// OrderLineItemsRepository  // WRONG!

// ✅ DO access children through aggregate root:
order.addItem('product-123', 'Widget', 2, new Money(10, 'USD'));
await ordersRepository.save(order);  // Saves order + items together
```

---

### **Comparison:**

| Aspect | Entity | Value Object | Aggregate |
|--------|--------|--------------|-----------|
| **Identity** | ✅ Has unique ID | ❌ No identity | ✅ Root has ID |
| **Equality** | By ID | By all attributes | By root ID |
| **Mutability** | ✅ Mutable | ❌ Immutable | ✅ Mutable |
| **Lifecycle** | Created, modified, deleted | Created, replaced | Manages children |
| **Examples** | User, Order, Product | Money, Address, Email | Order+Items, User+Profile |
| **Persistence** | Own table | Embedded or separate | Root + children together |
| **Access** | Direct | Direct | Through root only |

---

### **Best Practices:**

```typescript
// ✅ GOOD: Entity with identity and behavior
export class User {
  private userId: string;  // Identity
  
  changeEmail(newEmail: string) {  // Behavior
    // Validation and logic
  }
}

// ❌ BAD: Anemic entity (just data)
export class User {
  userId: string;
  email: string;
  // No methods, just properties
}

// ✅ GOOD: Immutable value object
export class Money {
  private readonly amount: number;
  
  add(other: Money): Money {
    return new Money(this.amount + other.amount, this.currency);  // New instance
  }
}

// ❌ BAD: Mutable value object
export class Money {
  amount: number;
  
  add(other: Money): void {
    this.amount += other.amount;  // Mutates!
  }
}

// ✅ GOOD: Access children through aggregate root
order.addItem(productId, quantity, price);
await orderRepository.save(order);  // Saves order + items

// ❌ BAD: Direct access to children
const item = new OrderLineItem(...);
await orderLineItemRepository.save(item);  // Wrong!

// ✅ GOOD: Value objects in entity
export class User {
  private email: Email;  // Value object
  private address: Address;  // Value object
}

// ❌ BAD: Primitives everywhere
export class User {
  private email: string;  // Primitive, no validation
  private address: string;  // Primitive, no structure
}

// ✅ GOOD: Aggregate enforces consistency
order.addItem(...);  // Order recalculates total
order.placeOrder();  // Order validates items exist

// ❌ BAD: External code manages consistency
order.items.push(item);
order.total += item.subtotal;  // Manual calculation, error-prone
```

**Key Takeaway:** **Entities** have **unique identity** that persists (User with ID, even if name changes). **Value Objects** are **immutable** and defined by attributes (Money, Address - two $10 are identical). **Aggregates** are **clusters with root entity** enforcing consistency (Order manages OrderLineItems, ensures total is correct). Entities: mutable, lifecycle, behavior (methods). Value Objects: immutable, no identity, equality by attributes, operations return new instances. Aggregates: root controls access to children, enforces business rules, saved/loaded as unit, repository operates on root only. Use entities for domain concepts with identity, value objects for descriptive concepts, aggregates for consistency boundaries. Access aggregate children only through root, never directly.

</details>

<details>
<summary><strong>19. How do Aggregates work with Repositories in DDD?</strong></summary>

**Answer:**

In DDD, **Repositories operate on Aggregate Roots only**, not individual entities within the aggregate. Each aggregate has **one repository** that loads/saves the **entire aggregate as a unit**, ensuring **consistency boundary**. Repository methods return domain entities (not database entities), hide persistence details, and support **unit of work pattern**. This ensures aggregate integrity: all changes to aggregate members go through root, all saves are transactional, and domain model stays independent of infrastructure.

---

### **Aggregate-Repository Relationship:**

```
┌─────────────────────────────────────────────────────┐
│              AGGREGATE PATTERN                      │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌─────────────────────────────────┐               │
│  │   Order (Aggregate Root)        │               │
│  │   - orderId                     │               │
│  │   - customerId                  │◄──────────────┼─── Repository operates
│  │   - total                       │               │    on ROOT only
│  │   - status                      │               │
│  │                                 │               │
│  │   + addItem()                   │               │
│  │   + removeItem()                │               │
│  │   + placeOrder()                │               │
│  └─────────────┬───────────────────┘               │
│                │ owns                              │
│                │                                   │
│                ▼                                   │
│  ┌─────────────────────────────────┐               │
│  │   OrderLineItem (child)         │               │
│  │   - productId                   │ ◄─────────────┼─── NO direct repository
│  │   - quantity                    │               │    for children!
│  │   - price                       │               │
│  │                                 │               │
│  │   + getSubtotal()               │               │
│  └─────────────────────────────────┘               │
│                                                     │
└─────────────────────────────────────────────────────┘
                    │
                    │ Repository interface
                    ▼
┌─────────────────────────────────────────────────────┐
│          IOrdersRepository                          │
│  + save(order: Order): Promise<void>                │
│  + findById(id: string): Promise<Order>             │
│  + findByCustomerId(id: string): Promise<Order[]>   │
└─────────────────────────────────────────────────────┘
                    │
                    │ Implementation
                    ▼
┌─────────────────────────────────────────────────────┐
│       TypeOrmOrdersRepository                       │
│  - Maps domain Order ↔ database tables              │
│  - Saves Order + OrderLineItems together            │
│  - Loads complete aggregate                         │
└─────────────────────────────────────────────────────┘
```

---

### **Method 1: Repository Interface (Domain Layer)**

```typescript
// ========== REPOSITORY INTERFACE IN DOMAIN LAYER ==========
// orders/domain/repositories/orders-repository.interface.ts

export interface IOrdersRepository {
  // Repository operates on aggregate root (Order) only
  
  save(order: Order): Promise<void>;
  
  findById(orderId: string): Promise<Order | null>;
  
  findByCustomerId(customerId: string): Promise<Order[]>;
  
  findByStatus(status: OrderStatus): Promise<Order[]>;
  
  delete(order: Order): Promise<void>;
  
  // Returns domain entity, not database entity
  // Loads complete aggregate (Order + all OrderLineItems)
}

// ❌ WRONG: No repository for aggregate children
// export interface IOrderLineItemsRepository {}  // Don't create this!

// Why? Because OrderLineItems are part of Order aggregate
// Access them through Order, not directly

// orders/domain/order.aggregate.ts
export class Order {
  private orderId: string;
  private customerId: string;
  private items: OrderLineItem[];  // Aggregate children
  private status: OrderStatus;
  private total: Money;

  // All modifications go through aggregate root
  addItem(productId: string, name: string, quantity: number, price: Money): void {
    const item = OrderLineItem.create(productId, name, quantity, price);
    this.items.push(item);
    this.recalculateTotal();
  }

  removeItem(productId: string): void {
    this.items = this.items.filter(item => item.getProductId() !== productId);
    this.recalculateTotal();
  }

  placeOrder(): void {
    if (this.items.length === 0) {
      throw new DomainException('Cannot place empty order');
    }
    this.status = OrderStatus.PLACED;
  }

  private recalculateTotal(): void {
    this.total = this.items.reduce(
      (sum, item) => sum.add(item.getSubtotal()),
      new Money(0, 'USD'),
    );
  }

  // Getters provide controlled access
  getId(): string {
    return this.orderId;
  }

  getItems(): ReadonlyArray<OrderLineItem> {
    return this.items;  // Read-only
  }

  getTotal(): Money {
    return this.total;
  }
}

// orders/domain/order-line-item.entity.ts
export class OrderLineItem {
  // Child entity, not aggregate root
  private constructor(
    private productId: string,
    private productName: string,
    private quantity: number,
    private price: Money,
  ) {}

  static create(productId: string, name: string, quantity: number, price: Money): OrderLineItem {
    if (quantity <= 0) {
      throw new DomainException('Quantity must be positive');
    }
    return new OrderLineItem(productId, name, quantity, price);
  }

  getSubtotal(): Money {
    return this.price.multiply(this.quantity);
  }

  getProductId(): string {
    return this.productId;
  }
}
```

---

### **Method 2: Repository Implementation (Infrastructure Layer)**

```typescript
// ========== REPOSITORY IMPLEMENTATION ==========
// orders/infrastructure/repositories/typeorm-orders.repository.ts

import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { IOrdersRepository } from '../../domain/repositories/orders-repository.interface';
import { Order } from '../../domain/order.aggregate';
import { OrderEntity } from '../persistence/order.entity';
import { OrderLineItemEntity } from '../persistence/order-line-item.entity';

@Injectable()
export class TypeOrmOrdersRepository implements IOrdersRepository {
  constructor(
    @InjectRepository(OrderEntity)
    private orderRepo: Repository<OrderEntity>,
    @InjectRepository(OrderLineItemEntity)
    private itemRepo: Repository<OrderLineItemEntity>,
  ) {}

  // Save entire aggregate (Order + all OrderLineItems) together
  async save(order: Order): Promise<void> {
    // Map domain aggregate to database entities
    const orderEntity = this.mapToOrderEntity(order);
    const itemEntities = this.mapToItemEntities(order);

    // Save in transaction (consistency!)
    await this.orderRepo.manager.transaction(async (manager) => {
      // Save order
      await manager.save(OrderEntity, orderEntity);

      // Delete existing items (simplification - could optimize)
      await manager.delete(OrderLineItemEntity, { orderId: order.getId() });

      // Save all items
      if (itemEntities.length > 0) {
        await manager.save(OrderLineItemEntity, itemEntities);
      }
    });
  }

  // Load complete aggregate (Order + all OrderLineItems)
  async findById(orderId: string): Promise<Order | null> {
    const orderEntity = await this.orderRepo.findOne({
      where: { id: orderId },
      relations: ['items'],  // Load children too!
    });

    if (!orderEntity) {
      return null;
    }

    // Map database entities to domain aggregate
    return this.mapToDomainOrder(orderEntity);
  }

  async findByCustomerId(customerId: string): Promise<Order[]> {
    const orderEntities = await this.orderRepo.find({
      where: { customerId },
      relations: ['items'],  // Always load complete aggregate
    });

    return orderEntities.map(entity => this.mapToDomainOrder(entity));
  }

  async delete(order: Order): Promise<void> {
    await this.orderRepo.manager.transaction(async (manager) => {
      // Delete items first (foreign key constraint)
      await manager.delete(OrderLineItemEntity, { orderId: order.getId() });
      
      // Delete order
      await manager.delete(OrderEntity, { id: order.getId() });
    });
  }

  // ===== MAPPING METHODS =====
  // Domain → Database

  private mapToOrderEntity(order: Order): OrderEntity {
    const entity = new OrderEntity();
    entity.id = order.getId();
    entity.customerId = order.getCustomerId();
    entity.status = order.getStatus();
    entity.total = order.getTotal().getAmount();
    entity.currency = order.getTotal().getCurrency();
    entity.placedAt = order.getPlacedAt();
    return entity;
  }

  private mapToItemEntities(order: Order): OrderLineItemEntity[] {
    return order.getItems().map(item => {
      const entity = new OrderLineItemEntity();
      entity.orderId = order.getId();
      entity.productId = item.getProductId();
      entity.productName = item.getProductName();
      entity.quantity = item.getQuantity();
      entity.price = item.getPrice().getAmount();
      entity.currency = item.getPrice().getCurrency();
      return entity;
    });
  }

  // Database → Domain

  private mapToDomainOrder(entity: OrderEntity): Order {
    // Reconstruct aggregate from database entities
    const items = entity.items.map(itemEntity => 
      OrderLineItem.create(
        itemEntity.productId,
        itemEntity.productName,
        itemEntity.quantity,
        new Money(itemEntity.price, itemEntity.currency),
      ),
    );

    return Order.reconstitute({
      orderId: entity.id,
      customerId: entity.customerId,
      items,
      status: entity.status,
      total: new Money(entity.total, entity.currency),
      placedAt: entity.placedAt,
    });
  }
}

// ===== DATABASE ENTITIES (TypeORM) =====
// orders/infrastructure/persistence/order.entity.ts

@Entity('orders')
export class OrderEntity {
  @PrimaryColumn('uuid')
  id: string;

  @Column('uuid')
  customerId: string;

  @Column({ type: 'varchar' })
  status: string;

  @Column({ type: 'decimal', precision: 10, scale: 2 })
  total: number;

  @Column({ type: 'varchar', length: 3 })
  currency: string;

  @Column({ type: 'timestamp', nullable: true })
  placedAt: Date;

  // Relationship to children
  @OneToMany(() => OrderLineItemEntity, item => item.order, { cascade: true })
  items: OrderLineItemEntity[];
}

// orders/infrastructure/persistence/order-line-item.entity.ts

@Entity('order_line_items')
export class OrderLineItemEntity {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column('uuid')
  orderId: string;

  @Column('uuid')
  productId: string;

  @Column()
  productName: string;

  @Column('int')
  quantity: number;

  @Column({ type: 'decimal', precision: 10, scale: 2 })
  price: number;

  @Column({ type: 'varchar', length: 3 })
  currency: string;

  @ManyToOne(() => OrderEntity, order => order.items)
  order: OrderEntity;
}
```

---

### **Method 3: Using Repository in Application Layer**

```typescript
// ========== APPLICATION LAYER USAGE ==========
// orders/application/use-cases/place-order.use-case.ts

@Injectable()
export class PlaceOrderUseCase {
  constructor(
    @Inject('IOrdersRepository')
    private ordersRepository: IOrdersRepository,
    private inventoryService: InventoryDomainService,
    private eventBus: EventBus,
  ) {}

  async execute(dto: PlaceOrderDto): Promise<OrderResponseDto> {
    // 1. Create aggregate (domain logic)
    const order = Order.create(
      dto.customerId,
      dto.shippingAddress,
      dto.billingAddress,
    );

    // 2. Add items through aggregate root
    for (const itemDto of dto.items) {
      const product = await this.productsRepository.findById(itemDto.productId);
      
      if (!product) {
        throw new NotFoundException('Product not found');
      }

      // Reserve inventory (domain service)
      await this.inventoryService.reserve(itemDto.productId, itemDto.quantity);

      // Add item to order
      order.addItem(
        product.getId(),
        product.getName(),
        itemDto.quantity,
        product.getPrice(),
      );
    }

    // 3. Apply business rules
    order.placeOrder();

    // 4. Save entire aggregate (Order + all items)
    await this.ordersRepository.save(order);

    // 5. Publish domain event
    this.eventBus.publish(new OrderPlacedEvent(order));

    return this.mapToDto(order);
  }
}

// orders/application/use-cases/cancel-order.use-case.ts

@Injectable()
export class CancelOrderUseCase {
  constructor(
    @Inject('IOrdersRepository')
    private ordersRepository: IOrdersRepository,
    private inventoryService: InventoryDomainService,
  ) {}

  async execute(orderId: string): Promise<void> {
    // 1. Load complete aggregate
    const order = await this.ordersRepository.findById(orderId);

    if (!order) {
      throw new NotFoundException('Order not found');
    }

    // 2. Release inventory
    for (const item of order.getItems()) {
      await this.inventoryService.release(
        item.getProductId(),
        item.getQuantity(),
      );
    }

    // 3. Apply business rule
    order.cancel();

    // 4. Save aggregate
    await this.ordersRepository.save(order);
  }
}

// orders/application/queries/get-order.query.ts

@Injectable()
export class GetOrderQuery {
  constructor(
    @Inject('IOrdersRepository')
    private ordersRepository: IOrdersRepository,
  ) {}

  async execute(orderId: string): Promise<OrderResponseDto> {
    // Load complete aggregate
    const order = await this.ordersRepository.findById(orderId);

    if (!order) {
      throw new NotFoundException('Order not found');
    }

    // Map to DTO
    return {
      orderId: order.getId(),
      customerId: order.getCustomerId(),
      items: order.getItems().map(item => ({
        productId: item.getProductId(),
        productName: item.getProductName(),
        quantity: item.getQuantity(),
        price: item.getPrice().getAmount(),
        subtotal: item.getSubtotal().getAmount(),
      })),
      total: order.getTotal().getAmount(),
      status: order.getStatus(),
      placedAt: order.getPlacedAt(),
    };
  }
}
```

---

### **Method 4: Repository Registration in Module**

```typescript
// ========== MODULE CONFIGURATION ==========
// orders/orders.module.ts

import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { OrderEntity } from './infrastructure/persistence/order.entity';
import { OrderLineItemEntity } from './infrastructure/persistence/order-line-item.entity';
import { TypeOrmOrdersRepository } from './infrastructure/repositories/typeorm-orders.repository';
import { PlaceOrderUseCase } from './application/use-cases/place-order.use-case';
import { CancelOrderUseCase } from './application/use-cases/cancel-order.use-case';
import { GetOrderQuery } from './application/queries/get-order.query';
import { OrdersController } from './controllers/orders.controller';

@Module({
  imports: [
    TypeOrmModule.forFeature([OrderEntity, OrderLineItemEntity]),
  ],
  controllers: [OrdersController],
  providers: [
    // Repository implementation
    TypeOrmOrdersRepository,
    
    // Provide interface → implementation binding
    {
      provide: 'IOrdersRepository',
      useClass: TypeOrmOrdersRepository,
    },
    
    // Use cases
    PlaceOrderUseCase,
    CancelOrderUseCase,
    GetOrderQuery,
  ],
  exports: ['IOrdersRepository'],  // Export for other modules
})
export class OrdersModule {}

// Domain layer depends on interface (IOrdersRepository)
// Infrastructure provides implementation (TypeOrmOrdersRepository)
// Dependency Inversion Principle!
```

---

### **Method 5: Multiple Aggregates with Separate Repositories**

```typescript
// ========== MULTIPLE AGGREGATES ==========
// Each aggregate has its own repository

// ORDERS AGGREGATE
// orders/domain/repositories/orders-repository.interface.ts
export interface IOrdersRepository {
  save(order: Order): Promise<void>;
  findById(id: string): Promise<Order | null>;
}

// CUSTOMERS AGGREGATE
// customers/domain/repositories/customers-repository.interface.ts
export interface ICustomersRepository {
  save(customer: Customer): Promise<void>;
  findById(id: string): Promise<Customer | null>;
  findByEmail(email: string): Promise<Customer | null>;
}

// PRODUCTS AGGREGATE
// products/domain/repositories/products-repository.interface.ts
export interface IProductsRepository {
  save(product: Product): Promise<void>;
  findById(id: string): Promise<Product | null>;
  findByCategory(category: string): Promise<Product[]>;
}

// Cross-aggregate references use IDs only!
export class Order {
  private customerId: string;  // Reference by ID, not object
  
  // Don't store entire Customer object
  // Load separately if needed
}

// orders/application/use-cases/place-order.use-case.ts
@Injectable()
export class PlaceOrderUseCase {
  constructor(
    @Inject('IOrdersRepository')
    private ordersRepository: IOrdersRepository,
    @Inject('ICustomersRepository')
    private customersRepository: ICustomersRepository,  // Separate repository
    @Inject('IProductsRepository')
    private productsRepository: IProductsRepository,  // Separate repository
  ) {}

  async execute(dto: PlaceOrderDto): Promise<OrderResponseDto> {
    // 1. Verify customer exists (separate aggregate)
    const customer = await this.customersRepository.findById(dto.customerId);
    if (!customer) {
      throw new NotFoundException('Customer not found');
    }

    // 2. Create order aggregate
    const order = Order.create(dto.customerId);

    // 3. Add products (separate aggregates)
    for (const itemDto of dto.items) {
      const product = await this.productsRepository.findById(itemDto.productId);
      
      if (!product) {
        throw new NotFoundException('Product not found');
      }

      // Store product snapshot in order
      order.addItem(
        product.getId(),
        product.getName(),  // Snapshot
        itemDto.quantity,
        product.getPrice(),  // Snapshot at order time
      );
    }

    // 4. Save order aggregate
    await this.ordersRepository.save(order);

    return this.mapToDto(order);
  }
}
```

---

### **Key Rules:**

| Rule | Description | Example |
|------|-------------|---------|
| **One Repository per Aggregate** | Each aggregate root has one repository | OrdersRepository, CustomersRepository |
| **Load Complete Aggregate** | Repository returns root + all children | Order with all OrderLineItems |
| **Save as Unit** | Save root + children in single transaction | order.addItem() → save(order) saves both |
| **No Child Repositories** | Don't create repositories for children | ❌ OrderLineItemsRepository |
| **Interface in Domain** | Repository interface in domain layer | IOrdersRepository in domain/ |
| **Implementation in Infrastructure** | Concrete class in infrastructure | TypeOrmOrdersRepository in infrastructure/ |
| **Return Domain Objects** | Repository returns domain entities, not DB entities | Order (domain), not OrderEntity (DB) |
| **Cross-Aggregate by ID** | Reference other aggregates by ID only | Order stores customerId, not Customer object |

---

### **Best Practices:**

```typescript
// ✅ GOOD: Repository operates on aggregate root
const order = await ordersRepository.findById(orderId);
order.addItem(productId, name, quantity, price);
await ordersRepository.save(order);  // Saves order + items together

// ❌ BAD: Direct access to children
const item = await orderLineItemsRepository.findById(itemId);  // Wrong!
item.quantity = 5;
await orderLineItemsRepository.save(item);  // Breaks aggregate consistency

// ✅ GOOD: Load complete aggregate
const order = await ordersRepository.findById(orderId);
// order.getItems() returns all items

// ❌ BAD: Lazy loading children
const order = await ordersRepository.findById(orderId);
const items = await order.loadItems();  // Wrong! Load together

// ✅ GOOD: Repository interface in domain
export interface IOrdersRepository {
  save(order: Order): Promise<void>;
}

// ❌ BAD: Repository depends on TypeORM in domain
export interface IOrdersRepository {
  save(order: Order): Promise<Repository<OrderEntity>>;  // Leaking infrastructure!
}

// ✅ GOOD: Cross-aggregate by ID
export class Order {
  private customerId: string;  // Reference by ID
}

// ❌ BAD: Storing entire aggregate
export class Order {
  private customer: Customer;  // Wrong! Different aggregate
}

// ✅ GOOD: Transactional save
async save(order: Order): Promise<void> {
  await this.manager.transaction(async (manager) => {
    await manager.save(orderEntity);
    await manager.save(itemEntities);
  });
}

// ❌ BAD: Separate saves (not transactional)
async save(order: Order): Promise<void> {
  await this.orderRepo.save(orderEntity);
  await this.itemRepo.save(itemEntities);  // Not atomic!
}
```

**Key Takeaway:** **Repositories operate on Aggregate Roots only**, loading and saving the **entire aggregate as a unit** (Order + all OrderLineItems together). Each aggregate has **one repository** with interface in domain layer, implementation in infrastructure. Repository returns **domain entities** (not DB entities), always loads **complete aggregate** (root + children), and saves **transactionally** to ensure consistency. **Don't create repositories for children** - access them through root only. Cross-aggregate references use **IDs only**, not object references. Benefits: **consistency boundary** enforced, **testable** domain logic (mock repository interface), **flexible persistence** (swap implementations), **transaction management** simplified, and **domain model independent** of database structure.

</details>

<details>
<summary><strong>20. How to implement DDD in NestJS?</strong></summary>

**Answer:**

Implement DDD in NestJS using **layered architecture**: organize code by bounded contexts (orders/, customers/), separate layers (domain/, application/, infrastructure/, controllers/), define **domain entities** with business logic, create **value objects** for immutable concepts, build **aggregates** with root control, implement **repository interfaces** in domain (implementations in infrastructure), use **domain services** for cross-entity logic, create **application services** (use cases) for orchestration, and publish **domain events** for communication. Structure ensures domain independence from frameworks.

---

### **NestJS DDD Project Structure:**

```
src/
├── orders/                              # Bounded Context (Module)
│   ├── orders.module.ts
│   ├── controllers/                     # Presentation Layer
│   │   ├── orders.controller.ts         # HTTP/GraphQL endpoints
│   │   └── dto/
│   │       ├── place-order.dto.ts
│   │       └── order-response.dto.ts
│   ├── application/                     # Application Layer
│   │   ├── use-cases/                   # Use cases (orchestration)
│   │   │   ├── place-order.use-case.ts
│   │   │   ├── cancel-order.use-case.ts
│   │   │   └── get-order.query.ts
│   │   └── event-handlers/              # Domain event handlers
│   │       └── order-placed.handler.ts
│   ├── domain/                          # Domain Layer (pure business logic)
│   │   ├── entities/
│   │   │   └── order.aggregate.ts       # Aggregate root
│   │   │   └── order-line-item.entity.ts
│   │   ├── value-objects/
│   │   │   ├── money.value-object.ts
│   │   │   └── order-status.value-object.ts
│   │   ├── services/
│   │   │   └── pricing-domain.service.ts # Domain service
│   │   ├── events/
│   │   │   ├── order-placed.event.ts
│   │   │   └── order-cancelled.event.ts
│   │   ├── repositories/
│   │   │   └── orders-repository.interface.ts  # Interface only
│   │   └── exceptions/
│   │       └── order.exceptions.ts
│   └── infrastructure/                  # Infrastructure Layer
│       ├── repositories/
│       │   └── typeorm-orders.repository.ts  # Repository implementation
│       └── persistence/
│           ├── order.entity.ts          # TypeORM entity
│           └── order-line-item.entity.ts
│
├── customers/                           # Another Bounded Context
│   ├── customers.module.ts
│   ├── controllers/
│   ├── application/
│   ├── domain/
│   └── infrastructure/
│
├── products/                            # Another Bounded Context
│   ├── products.module.ts
│   ├── controllers/
│   ├── application/
│   ├── domain/
│   └── infrastructure/
│
└── shared/                              # Shared Kernel
    ├── domain/
    │   ├── base-entity.ts
    │   ├── domain-event.ts
    │   ├── value-object.base.ts
    │   └── aggregate-root.base.ts
    └── infrastructure/
        ├── event-bus.ts
        └── unit-of-work.ts
```

---

### **Method 1: Domain Layer Implementation**

```typescript
// ========== DOMAIN LAYER: AGGREGATE ROOT ==========
// orders/domain/entities/order.aggregate.ts

import { AggregateRoot } from '../../../shared/domain/aggregate-root.base';
import { OrderLineItem } from './order-line-item.entity';
import { Money } from '../value-objects/money.value-object';
import { OrderStatus } from '../value-objects/order-status.value-object';
import { OrderPlacedEvent } from '../events/order-placed.event';
import { OrderCancelledEvent } from '../events/order-cancelled.event';
import { DomainException } from '../exceptions/domain.exception';

export class Order extends AggregateRoot {
  private orderId: string;
  private customerId: string;
  private items: OrderLineItem[] = [];
  private status: OrderStatus;
  private total: Money;
  private placedAt: Date | null = null;

  private constructor(data: any) {
    super();
    this.orderId = data.orderId;
    this.customerId = data.customerId;
    this.status = data.status || OrderStatus.draft();
    this.items = data.items || [];
    this.total = data.total || Money.zero('USD');
  }

  // ===== FACTORY METHODS =====

  static create(customerId: string): Order {
    const order = new Order({
      orderId: this.generateId(),
      customerId,
      status: OrderStatus.draft(),
      items: [],
      total: Money.zero('USD'),
    });

    return order;
  }

  static reconstitute(data: any): Order {
    // Used by repository to recreate from persistence
    return new Order(data);
  }

  // ===== DOMAIN BEHAVIOR =====

  addItem(productId: string, productName: string, quantity: number, price: Money): void {
    // Business rule: Check if item already exists
    const existingItem = this.items.find(item => item.getProductId() === productId);

    if (existingItem) {
      existingItem.increaseQuantity(quantity);
    } else {
      const item = OrderLineItem.create(productId, productName, quantity, price);
      this.items.push(item);
    }

    this.recalculateTotal();
  }

  removeItem(productId: string): void {
    const index = this.items.findIndex(item => item.getProductId() === productId);

    if (index === -1) {
      throw new DomainException('Item not found in order');
    }

    this.items.splice(index, 1);
    this.recalculateTotal();
  }

  updateItemQuantity(productId: string, newQuantity: number): void {
    const item = this.items.find(item => item.getProductId() === productId);

    if (!item) {
      throw new DomainException('Item not found');
    }

    if (newQuantity <= 0) {
      this.removeItem(productId);
    } else {
      item.setQuantity(newQuantity);
      this.recalculateTotal();
    }
  }

  placeOrder(): void {
    // Business invariants
    if (this.items.length === 0) {
      throw new DomainException('Cannot place order with no items');
    }

    if (!this.status.isDraft()) {
      throw new DomainException('Order already placed');
    }

    if (this.total.isZero()) {
      throw new DomainException('Order total must be greater than zero');
    }

    // Change state
    this.status = OrderStatus.placed();
    this.placedAt = new Date();

    // Publish domain event
    this.addDomainEvent(new OrderPlacedEvent(this.orderId, this.customerId, this.total));
  }

  cancel(): void {
    if (this.status.isShipped()) {
      throw new DomainException('Cannot cancel shipped order');
    }

    if (this.status.isCancelled()) {
      throw new DomainException('Order already cancelled');
    }

    this.status = OrderStatus.cancelled();

    this.addDomainEvent(new OrderCancelledEvent(this.orderId));
  }

  private recalculateTotal(): void {
    this.total = this.items.reduce(
      (sum, item) => sum.add(item.getSubtotal()),
      Money.zero('USD'),
    );
  }

  // ===== GETTERS =====

  getId(): string {
    return this.orderId;
  }

  getCustomerId(): string {
    return this.customerId;
  }

  getItems(): ReadonlyArray<OrderLineItem> {
    return this.items;
  }

  getTotal(): Money {
    return this.total;
  }

  getStatus(): OrderStatus {
    return this.status;
  }

  getPlacedAt(): Date | null {
    return this.placedAt;
  }

  private static generateId(): string {
    return `order_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
}

// ========== VALUE OBJECT: MONEY ==========
// orders/domain/value-objects/money.value-object.ts

export class Money {
  private constructor(
    private readonly amount: number,
    private readonly currency: string,
  ) {
    if (amount < 0) {
      throw new Error('Amount cannot be negative');
    }
  }

  static zero(currency: string): Money {
    return new Money(0, currency);
  }

  static of(amount: number, currency: string): Money {
    return new Money(amount, currency);
  }

  add(other: Money): Money {
    this.ensureSameCurrency(other);
    return new Money(this.amount + other.amount, this.currency);
  }

  subtract(other: Money): Money {
    this.ensureSameCurrency(other);
    return new Money(this.amount - other.amount, this.currency);
  }

  multiply(factor: number): Money {
    return new Money(this.amount * factor, this.currency);
  }

  isZero(): boolean {
    return this.amount === 0;
  }

  equals(other: Money): boolean {
    return this.amount === other.amount && this.currency === other.currency;
  }

  private ensureSameCurrency(other: Money): void {
    if (this.currency !== other.currency) {
      throw new Error('Cannot operate on different currencies');
    }
  }

  getAmount(): number {
    return this.amount;
  }

  getCurrency(): string {
    return this.currency;
  }
}

// ========== VALUE OBJECT: ORDER STATUS ==========
// orders/domain/value-objects/order-status.value-object.ts

export class OrderStatus {
  private constructor(private readonly value: string) {}

  static draft(): OrderStatus {
    return new OrderStatus('DRAFT');
  }

  static placed(): OrderStatus {
    return new OrderStatus('PLACED');
  }

  static shipped(): OrderStatus {
    return new OrderStatus('SHIPPED');
  }

  static cancelled(): OrderStatus {
    return new OrderStatus('CANCELLED');
  }

  isDraft(): boolean {
    return this.value === 'DRAFT';
  }

  isPlaced(): boolean {
    return this.value === 'PLACED';
  }

  isShipped(): boolean {
    return this.value === 'SHIPPED';
  }

  isCancelled(): boolean {
    return this.value === 'CANCELLED';
  }

  toString(): string {
    return this.value;
  }

  equals(other: OrderStatus): boolean {
    return this.value === other.value;
  }
}

// ========== REPOSITORY INTERFACE ==========
// orders/domain/repositories/orders-repository.interface.ts

export interface IOrdersRepository {
  save(order: Order): Promise<void>;
  findById(orderId: string): Promise<Order | null>;
  findByCustomerId(customerId: string): Promise<Order[]>;
  delete(order: Order): Promise<void>;
}
```

---

### **Method 2: Application Layer (Use Cases)**

```typescript
// ========== APPLICATION LAYER: USE CASE ==========
// orders/application/use-cases/place-order.use-case.ts

import { Injectable, Inject, NotFoundException } from '@nestjs/common';
import { IOrdersRepository } from '../../domain/repositories/orders-repository.interface';
import { Order } from '../../domain/entities/order.aggregate';
import { Money } from '../../domain/value-objects/money.value-object';
import { EventBus } from '../../../shared/infrastructure/event-bus';
import { PlaceOrderDto } from '../../controllers/dto/place-order.dto';

@Injectable()
export class PlaceOrderUseCase {
  constructor(
    @Inject('IOrdersRepository')
    private readonly ordersRepository: IOrdersRepository,
    @Inject('IProductsRepository')
    private readonly productsRepository: any,
    private readonly eventBus: EventBus,
  ) {}

  async execute(dto: PlaceOrderDto): Promise<OrderResponseDto> {
    // 1. Create aggregate
    const order = Order.create(dto.customerId);

    // 2. Add items
    for (const itemDto of dto.items) {
      // Fetch product (from different aggregate)
      const product = await this.productsRepository.findById(itemDto.productId);

      if (!product) {
        throw new NotFoundException(`Product ${itemDto.productId} not found`);
      }

      // Add item to order
      order.addItem(
        product.getId(),
        product.getName(),
        itemDto.quantity,
        product.getPrice(),
      );
    }

    // 3. Place order (business logic)
    order.placeOrder();

    // 4. Persist
    await this.ordersRepository.save(order);

    // 5. Publish domain events
    const events = order.getDomainEvents();
    for (const event of events) {
      await this.eventBus.publish(event);
    }
    order.clearDomainEvents();

    // 6. Return DTO
    return this.toDto(order);
  }

  private toDto(order: Order): OrderResponseDto {
    return {
      orderId: order.getId(),
      customerId: order.getCustomerId(),
      items: order.getItems().map(item => ({
        productId: item.getProductId(),
        productName: item.getProductName(),
        quantity: item.getQuantity(),
        price: item.getPrice().getAmount(),
        subtotal: item.getSubtotal().getAmount(),
      })),
      total: order.getTotal().getAmount(),
      status: order.getStatus().toString(),
      placedAt: order.getPlacedAt(),
    };
  }
}

// ========== EVENT HANDLER ==========
// orders/application/event-handlers/order-placed.handler.ts

import { Injectable } from '@nestjs/common';
import { OnEvent } from '@nestjs/event-emitter';
import { OrderPlacedEvent } from '../../domain/events/order-placed.event';

@Injectable()
export class OrderPlacedHandler {
  constructor(
    private readonly emailService: any,
    private readonly analyticsService: any,
  ) {}

  @OnEvent('order.placed')
  async handle(event: OrderPlacedEvent): Promise<void> {
    // Side effects (not part of domain logic)
    
    // Send confirmation email
    await this.emailService.sendOrderConfirmation(
      event.customerId,
      event.orderId,
    );

    // Track analytics
    await this.analyticsService.trackOrderPlaced({
      orderId: event.orderId,
      total: event.total.getAmount(),
    });

    console.log(`Order ${event.orderId} placed successfully`);
  }
}
```

---

### **Method 3: Infrastructure Layer (Repository)**

```typescript
// ========== INFRASTRUCTURE: REPOSITORY IMPLEMENTATION ==========
// orders/infrastructure/repositories/typeorm-orders.repository.ts

import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { IOrdersRepository } from '../../domain/repositories/orders-repository.interface';
import { Order } from '../../domain/entities/order.aggregate';
import { OrderEntity } from '../persistence/order.entity';
import { OrderLineItemEntity } from '../persistence/order-line-item.entity';
import { Money } from '../../domain/value-objects/money.value-object';
import { OrderStatus } from '../../domain/value-objects/order-status.value-object';
import { OrderLineItem } from '../../domain/entities/order-line-item.entity';

@Injectable()
export class TypeOrmOrdersRepository implements IOrdersRepository {
  constructor(
    @InjectRepository(OrderEntity)
    private readonly orderRepo: Repository<OrderEntity>,
    @InjectRepository(OrderLineItemEntity)
    private readonly itemRepo: Repository<OrderLineItemEntity>,
  ) {}

  async save(order: Order): Promise<void> {
    await this.orderRepo.manager.transaction(async (manager) => {
      // Map domain to persistence
      const orderEntity = new OrderEntity();
      orderEntity.id = order.getId();
      orderEntity.customerId = order.getCustomerId();
      orderEntity.status = order.getStatus().toString();
      orderEntity.total = order.getTotal().getAmount();
      orderEntity.currency = order.getTotal().getCurrency();
      orderEntity.placedAt = order.getPlacedAt();

      await manager.save(OrderEntity, orderEntity);

      // Delete existing items (simplification)
      await manager.delete(OrderLineItemEntity, { orderId: order.getId() });

      // Save items
      const itemEntities = order.getItems().map(item => {
        const entity = new OrderLineItemEntity();
        entity.orderId = order.getId();
        entity.productId = item.getProductId();
        entity.productName = item.getProductName();
        entity.quantity = item.getQuantity();
        entity.price = item.getPrice().getAmount();
        entity.currency = item.getPrice().getCurrency();
        return entity;
      });

      if (itemEntities.length > 0) {
        await manager.save(OrderLineItemEntity, itemEntities);
      }
    });
  }

  async findById(orderId: string): Promise<Order | null> {
    const entity = await this.orderRepo.findOne({
      where: { id: orderId },
      relations: ['items'],
    });

    if (!entity) {
      return null;
    }

    return this.toDomain(entity);
  }

  async findByCustomerId(customerId: string): Promise<Order[]> {
    const entities = await this.orderRepo.find({
      where: { customerId },
      relations: ['items'],
    });

    return entities.map(entity => this.toDomain(entity));
  }

  async delete(order: Order): Promise<void> {
    await this.orderRepo.delete({ id: order.getId() });
  }

  private toDomain(entity: OrderEntity): Order {
    const items = entity.items.map(itemEntity =>
      OrderLineItem.create(
        itemEntity.productId,
        itemEntity.productName,
        itemEntity.quantity,
        Money.of(itemEntity.price, itemEntity.currency),
      ),
    );

    return Order.reconstitute({
      orderId: entity.id,
      customerId: entity.customerId,
      items,
      status: this.toOrderStatus(entity.status),
      total: Money.of(entity.total, entity.currency),
      placedAt: entity.placedAt,
    });
  }

  private toOrderStatus(status: string): OrderStatus {
    switch (status) {
      case 'DRAFT': return OrderStatus.draft();
      case 'PLACED': return OrderStatus.placed();
      case 'SHIPPED': return OrderStatus.shipped();
      case 'CANCELLED': return OrderStatus.cancelled();
      default: return OrderStatus.draft();
    }
  }
}

// ========== PERSISTENCE ENTITIES ==========
// orders/infrastructure/persistence/order.entity.ts

import { Entity, Column, PrimaryColumn, OneToMany } from 'typeorm';
import { OrderLineItemEntity } from './order-line-item.entity';

@Entity('orders')
export class OrderEntity {
  @PrimaryColumn('uuid')
  id: string;

  @Column('uuid')
  customerId: string;

  @Column()
  status: string;

  @Column('decimal')
  total: number;

  @Column({ length: 3 })
  currency: string;

  @Column({ type: 'timestamp', nullable: true })
  placedAt: Date | null;

  @OneToMany(() => OrderLineItemEntity, item => item.order, { cascade: true })
  items: OrderLineItemEntity[];
}
```

---

### **Method 4: Module Configuration**

```typescript
// ========== MODULE SETUP ==========
// orders/orders.module.ts

import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { EventEmitterModule } from '@nestjs/event-emitter';

// Infrastructure
import { OrderEntity } from './infrastructure/persistence/order.entity';
import { OrderLineItemEntity } from './infrastructure/persistence/order-line-item.entity';
import { TypeOrmOrdersRepository } from './infrastructure/repositories/typeorm-orders.repository';

// Application
import { PlaceOrderUseCase } from './application/use-cases/place-order.use-case';
import { CancelOrderUseCase } from './application/use-cases/cancel-order.use-case';
import { GetOrderQuery } from './application/queries/get-order.query';
import { OrderPlacedHandler } from './application/event-handlers/order-placed.handler';

// Presentation
import { OrdersController } from './controllers/orders.controller';

@Module({
  imports: [
    TypeOrmModule.forFeature([OrderEntity, OrderLineItemEntity]),
    EventEmitterModule.forRoot(),
  ],
  controllers: [OrdersController],
  providers: [
    // Repository
    TypeOrmOrdersRepository,
    {
      provide: 'IOrdersRepository',
      useClass: TypeOrmOrdersRepository,
    },

    // Use cases
    PlaceOrderUseCase,
    CancelOrderUseCase,
    GetOrderQuery,

    // Event handlers
    OrderPlacedHandler,
  ],
  exports: ['IOrdersRepository'],
})
export class OrdersModule {}

// ========== MAIN APP MODULE ==========
// app.module.ts

import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { OrdersModule } from './orders/orders.module';
import { CustomersModule } from './customers/customers.module';
import { ProductsModule } from './products/products.module';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      database: 'ecommerce',
      autoLoadEntities: true,
      synchronize: false,  // Use migrations in production!
    }),
    
    // Bounded contexts as modules
    OrdersModule,
    CustomersModule,
    ProductsModule,
  ],
})
export class AppModule {}
```

---

### **Method 5: Shared Base Classes**

```typescript
// ========== SHARED BASE CLASSES ==========
// shared/domain/aggregate-root.base.ts

export abstract class AggregateRoot {
  private domainEvents: any[] = [];

  protected addDomainEvent(event: any): void {
    this.domainEvents.push(event);
  }

  getDomainEvents(): any[] {
    return this.domainEvents;
  }

  clearDomainEvents(): void {
    this.domainEvents = [];
  }
}

// shared/domain/domain-event.ts

export interface IDomainEvent {
  occurredOn: Date;
  eventName: string;
}

export abstract class DomainEvent implements IDomainEvent {
  public readonly occurredOn: Date;
  public readonly eventName: string;

  constructor(eventName: string) {
    this.occurredOn = new Date();
    this.eventName = eventName;
  }
}

// orders/domain/events/order-placed.event.ts

import { DomainEvent } from '../../../shared/domain/domain-event';
import { Money } from '../value-objects/money.value-object';

export class OrderPlacedEvent extends DomainEvent {
  constructor(
    public readonly orderId: string,
    public readonly customerId: string,
    public readonly total: Money,
  ) {
    super('order.placed');
  }
}
```

---

### **DDD Layers Summary:**

| Layer | Responsibilities | Examples | Dependencies |
|-------|------------------|----------|--------------|
| **Domain** | Business logic, entities, rules | Order, Money, IOrdersRepository | None (pure domain) |
| **Application** | Use cases, orchestration | PlaceOrderUseCase, OrderPlacedHandler | Domain layer only |
| **Infrastructure** | Persistence, external services | TypeOrmOrdersRepository, EmailService | Domain + Application |
| **Presentation** | Controllers, DTOs | OrdersController, PlaceOrderDto | Application layer |

---

### **Best Practices:**

```typescript
// ✅ GOOD: Domain layer independent
export class Order {
  placeOrder(): void {
    // Pure business logic, no framework dependencies
  }
}

// ❌ BAD: Domain depends on framework
import { Injectable } from '@nestjs/common';  // Wrong in domain!
export class Order {}

// ✅ GOOD: Repository interface in domain
export interface IOrdersRepository {
  save(order: Order): Promise<void>;
}

// ❌ BAD: Repository implementation in domain
export class TypeOrmOrdersRepository {}  // Should be in infrastructure

// ✅ GOOD: Aggregate controls children
order.addItem(productId, name, quantity, price);
await ordersRepository.save(order);

// ❌ BAD: Direct child manipulation
const item = new OrderLineItem(...);
order.items.push(item);  // Bypasses aggregate logic

// ✅ GOOD: Use cases orchestrate
class PlaceOrderUseCase {
  async execute(dto) {
    const order = Order.create(...);
    order.placeOrder();
    await this.repository.save(order);
  }
}

// ❌ BAD: Business logic in controller
@Post()
async placeOrder(@Body() dto) {
  const order = new Order();
  order.status = 'PLACED';  // Business logic leaking!
  await this.repository.save(order);
}
```

**Key Takeaway:** Implement **DDD in NestJS** with **layered architecture**: organize by **bounded contexts** (orders/, customers/), separate **domain** (entities, value objects, aggregates, repository interfaces - no framework dependencies), **application** (use cases, orchestration, event handlers), **infrastructure** (repository implementations, TypeORM entities, external services), and **presentation** (controllers, DTOs). Key patterns: **aggregates** control children, **repositories** operate on roots, **value objects** immutable, **domain events** for communication, **dependency injection** via interfaces (DIP), **use cases** orchestrate workflows. Module provides repository interface → implementation binding. Structure ensures **domain independence**, **testability** (mock interfaces), **flexibility** (swap implementations), and **maintainability** (clear boundaries). Follow dependency rule: Domain → Application → Infrastructure/Presentation.

</details>

## SOLID Principles

<details>
<summary><strong>21. What are SOLID principles?</strong></summary>

**Answer:**

**SOLID** is an acronym for **five design principles** that make software more **maintainable**, **flexible**, and **scalable**: **S**ingle Responsibility Principle (class has one reason to change), **O**pen/Closed Principle (open for extension, closed for modification), **L**iskov Substitution Principle (subtypes must be substitutable), **I**nterface Segregation Principle (many specific interfaces better than one general), and **D**ependency Inversion Principle (depend on abstractions, not concretions). These principles reduce coupling, increase cohesion, and make code easier to test and modify.

---

### **SOLID Principles Overview:**

```
┌──────────────────────────────────────────────────────────┐
│                   SOLID PRINCIPLES                       │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  S  Single Responsibility Principle (SRP)               │
│     "A class should have one, and only one,             │
│      reason to change"                                  │
│     Each class does ONE thing                           │
│                                                          │
│  O  Open/Closed Principle (OCP)                         │
│     "Open for extension, closed for modification"       │
│     Add new features without changing existing code     │
│                                                          │
│  L  Liskov Substitution Principle (LSP)                 │
│     "Objects should be replaceable with subtypes        │
│      without breaking the program"                      │
│     Derived classes must be substitutable for base      │
│                                                          │
│  I  Interface Segregation Principle (ISP)               │
│     "Many client-specific interfaces are better         │
│      than one general-purpose interface"                │
│     Don't force clients to depend on unused methods     │
│                                                          │
│  D  Dependency Inversion Principle (DIP)                │
│     "Depend on abstractions, not on concretions"        │
│     High-level modules shouldn't depend on low-level    │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

### **Method 1: Single Responsibility Principle (SRP)**

**Definition:** A class should have **one, and only one, reason to change**. Each class should do one thing well.

```typescript
// ========== SINGLE RESPONSIBILITY PRINCIPLE ==========

// ❌ BAD: Multiple responsibilities in one class
@Injectable()
export class UserService {
  // Responsibility 1: User business logic
  async createUser(dto: CreateUserDto): Promise<User> {
    const user = new User();
    user.email = dto.email;
    user.password = await bcrypt.hash(dto.password, 10);
    return this.userRepository.save(user);
  }

  // Responsibility 2: Email sending
  async sendWelcomeEmail(user: User): Promise<void> {
    const transporter = nodemailer.createTransport({/*...*/});
    await transporter.sendMail({
      to: user.email,
      subject: 'Welcome!',
      html: '<h1>Welcome to our app!</h1>',
    });
  }

  // Responsibility 3: Logging
  async logUserActivity(user: User, action: string): Promise<void> {
    const log = `${new Date()} - User ${user.id} performed ${action}`;
    fs.appendFileSync('user-activity.log', log);
  }

  // Responsibility 4: Data export
  async exportUsersToCsv(): Promise<string> {
    const users = await this.userRepository.findAll();
    return users.map(u => `${u.id},${u.email}`).join('\n');
  }
}

// This class has FOUR reasons to change:
// 1. User business logic changes
// 2. Email provider changes (SMTP → SendGrid)
// 3. Logging strategy changes (file → database)
// 4. Export format changes (CSV → JSON)

// ✅ GOOD: Separate responsibilities
@Injectable()
export class UserService {
  constructor(
    private userRepository: UserRepository,
    private emailService: EmailService,  // Separate service
    private loggerService: LoggerService,  // Separate service
  ) {}

  // Only user business logic
  async createUser(dto: CreateUserDto): Promise<User> {
    const user = new User();
    user.email = dto.email;
    user.password = await bcrypt.hash(dto.password, 10);
    
    const savedUser = await this.userRepository.save(user);
    
    // Delegate to other services
    await this.emailService.sendWelcomeEmail(savedUser);
    this.loggerService.log(`User created: ${savedUser.id}`);
    
    return savedUser;
  }
}

@Injectable()
export class EmailService {
  // Only email sending
  async sendWelcomeEmail(user: User): Promise<void> {
    const transporter = nodemailer.createTransporter({/*...*/});
    await transporter.sendMail({
      to: user.email,
      subject: 'Welcome!',
      html: '<h1>Welcome to our app!</h1>',
    });
  }
}

@Injectable()
export class LoggerService {
  // Only logging
  log(message: string): void {
    const log = `${new Date()} - ${message}`;
    fs.appendFileSync('app.log', log);
  }
}

@Injectable()
export class UserExportService {
  // Only data export
  async exportUsersToCsv(): Promise<string> {
    const users = await this.userRepository.findAll();
    return users.map(u => `${u.id},${u.email}`).join('\n');
  }
}

// Now each class has ONE reason to change
```

---

### **Method 2: Open/Closed Principle (OCP)**

**Definition:** Software entities should be **open for extension** but **closed for modification**. Add new functionality without changing existing code.

```typescript
// ========== OPEN/CLOSED PRINCIPLE ==========

// ❌ BAD: Must modify code to add new payment methods
@Injectable()
export class PaymentService {
  async processPayment(amount: number, method: string): Promise<void> {
    if (method === 'credit-card') {
      // Credit card processing logic
      console.log(`Processing ${amount} via credit card`);
    } else if (method === 'paypal') {
      // PayPal processing logic
      console.log(`Processing ${amount} via PayPal`);
    } else if (method === 'stripe') {
      // Stripe processing logic
      console.log(`Processing ${amount} via Stripe`);
    }
    // To add new method, must modify this class!
  }
}

// ✅ GOOD: Use Strategy pattern - open for extension
export interface IPaymentProcessor {
  process(amount: number): Promise<void>;
}

@Injectable()
export class CreditCardProcessor implements IPaymentProcessor {
  async process(amount: number): Promise<void> {
    console.log(`Processing ${amount} via credit card`);
    // Credit card logic
  }
}

@Injectable()
export class PayPalProcessor implements IPaymentProcessor {
  async process(amount: number): Promise<void> {
    console.log(`Processing ${amount} via PayPal`);
    // PayPal logic
  }
}

@Injectable()
export class StripeProcessor implements IPaymentProcessor {
  async process(amount: number): Promise<void> {
    console.log(`Processing ${amount} via Stripe`);
    // Stripe logic
  }
}

// Add new processor without modifying existing code
@Injectable()
export class BitcoinProcessor implements IPaymentProcessor {
  async process(amount: number): Promise<void> {
    console.log(`Processing ${amount} via Bitcoin`);
    // Bitcoin logic
  }
}

@Injectable()
export class PaymentService {
  private processors = new Map<string, IPaymentProcessor>();

  constructor(
    creditCardProcessor: CreditCardProcessor,
    paypalProcessor: PayPalProcessor,
    stripeProcessor: StripeProcessor,
    bitcoinProcessor: BitcoinProcessor,
  ) {
    this.processors.set('credit-card', creditCardProcessor);
    this.processors.set('paypal', paypalProcessor);
    this.processors.set('stripe', stripeProcessor);
    this.processors.set('bitcoin', bitcoinProcessor);
  }

  async processPayment(amount: number, method: string): Promise<void> {
    const processor = this.processors.get(method);
    
    if (!processor) {
      throw new Error(`Unknown payment method: ${method}`);
    }

    await processor.process(amount);
  }
}

// To add new payment method:
// 1. Create new processor class implementing IPaymentProcessor
// 2. Register in constructor
// No modification of existing logic!
```

---

### **Method 3: Liskov Substitution Principle (LSP)**

**Definition:** Objects of a superclass should be **replaceable with objects of subclasses** without breaking the application. Derived classes must honor the contract of the base class.

```typescript
// ========== LISKOV SUBSTITUTION PRINCIPLE ==========

// ❌ BAD: Violates LSP
abstract class Bird {
  abstract fly(): void;
}

class Sparrow extends Bird {
  fly(): void {
    console.log('Sparrow flying');
  }
}

class Penguin extends Bird {
  fly(): void {
    // Penguins can't fly!
    throw new Error('Penguins cannot fly');
  }
}

function makeBirdFly(bird: Bird): void {
  bird.fly();  // This breaks if bird is Penguin!
}

makeBirdFly(new Sparrow());  // OK
makeBirdFly(new Penguin());  // ERROR! Violates LSP

// ✅ GOOD: Proper abstraction
abstract class Bird {
  abstract move(): void;
}

class FlyingBird extends Bird {
  move(): void {
    this.fly();
  }

  fly(): void {
    console.log('Flying');
  }
}

class Sparrow extends FlyingBird {
  fly(): void {
    console.log('Sparrow flying');
  }
}

class Penguin extends Bird {
  move(): void {
    this.swim();
  }

  swim(): void {
    console.log('Penguin swimming');
  }
}

function makeBirdMove(bird: Bird): void {
  bird.move();  // Works for all birds
}

makeBirdMove(new Sparrow());  // Flies
makeBirdMove(new Penguin());  // Swims

// Real-world NestJS example:

// ❌ BAD: Violates LSP
export interface IRepository<T> {
  findAll(): Promise<T[]>;
  save(entity: T): Promise<T>;
  delete(id: string): Promise<void>;
}

export class InMemoryUserRepository implements IRepository<User> {
  findAll(): Promise<User[]> {
    return Promise.resolve([]);
  }

  save(user: User): Promise<User> {
    return Promise.resolve(user);
  }

  delete(id: string): Promise<void> {
    throw new Error('In-memory repository does not support deletion');
    // Violates contract! Client expects delete to work
  }
}

// ✅ GOOD: Honors contract
export interface IRepository<T> {
  findAll(): Promise<T[]>;
  save(entity: T): Promise<T>;
  delete(id: string): Promise<void>;
}

export class InMemoryUserRepository implements IRepository<User> {
  private users: User[] = [];

  findAll(): Promise<User[]> {
    return Promise.resolve(this.users);
  }

  save(user: User): Promise<User> {
    this.users.push(user);
    return Promise.resolve(user);
  }

  delete(id: string): Promise<void> {
    this.users = this.users.filter(u => u.id !== id);
    return Promise.resolve();
  }
}

// Now any code using IRepository works with any implementation
```

---

### **Method 4: Interface Segregation Principle (ISP)**

**Definition:** Clients should not be forced to depend on **interfaces they don't use**. Many specific interfaces are better than one general-purpose interface.

```typescript
// ========== INTERFACE SEGREGATION PRINCIPLE ==========

// ❌ BAD: Fat interface with many methods
export interface IRepository<T> {
  // Read operations
  findAll(): Promise<T[]>;
  findById(id: string): Promise<T | null>;
  findByQuery(query: any): Promise<T[]>;
  
  // Write operations
  save(entity: T): Promise<T>;
  update(id: string, data: Partial<T>): Promise<T>;
  delete(id: string): Promise<void>;
  
  // Bulk operations
  bulkInsert(entities: T[]): Promise<void>;
  bulkDelete(ids: string[]): Promise<void>;
  
  // Advanced operations
  transaction(callback: () => Promise<void>): Promise<void>;
  cache(key: string, ttl: number): Promise<void>;
  export(): Promise<string>;
}

// Problem: Read-only clients must implement unused methods
export class ReadOnlyUserRepository implements IRepository<User> {
  findAll(): Promise<User[]> { /* implementation */ }
  findById(id: string): Promise<User | null> { /* implementation */ }
  findByQuery(query: any): Promise<User[]> { /* implementation */ }
  
  // Must implement these even though read-only!
  save(entity: User): Promise<User> {
    throw new Error('Read-only repository');
  }
  update(id: string, data: Partial<User>): Promise<User> {
    throw new Error('Read-only repository');
  }
  delete(id: string): Promise<void> {
    throw new Error('Read-only repository');
  }
  bulkInsert(entities: User[]): Promise<void> {
    throw new Error('Read-only repository');
  }
  bulkDelete(ids: string[]): Promise<void> {
    throw new Error('Read-only repository');
  }
  transaction(callback: () => Promise<void>): Promise<void> {
    throw new Error('Read-only repository');
  }
  cache(key: string, ttl: number): Promise<void> {
    throw new Error('Read-only repository');
  }
  export(): Promise<string> {
    throw new Error('Read-only repository');
  }
}

// ✅ GOOD: Segregated interfaces
export interface IReadRepository<T> {
  findAll(): Promise<T[]>;
  findById(id: string): Promise<T | null>;
  findByQuery(query: any): Promise<T[]>;
}

export interface IWriteRepository<T> {
  save(entity: T): Promise<T>;
  update(id: string, data: Partial<T>): Promise<T>;
  delete(id: string): Promise<void>;
}

export interface IBulkOperations<T> {
  bulkInsert(entities: T[]): Promise<void>;
  bulkDelete(ids: string[]): Promise<void>;
}

export interface ICacheable {
  cache(key: string, ttl: number): Promise<void>;
}

// Clients implement only what they need
export class ReadOnlyUserRepository implements IReadRepository<User> {
  findAll(): Promise<User[]> { /* implementation */ }
  findById(id: string): Promise<User | null> { /* implementation */ }
  findByQuery(query: any): Promise<User[]> { /* implementation */ }
  // No need to implement write methods!
}

export class FullUserRepository 
  implements IReadRepository<User>, IWriteRepository<User>, IBulkOperations<User> {
  // Implements all three interfaces
  findAll(): Promise<User[]> { /* implementation */ }
  findById(id: string): Promise<User | null> { /* implementation */ }
  findByQuery(query: any): Promise<User[]> { /* implementation */ }
  save(entity: User): Promise<User> { /* implementation */ }
  update(id: string, data: Partial<User>): Promise<User> { /* implementation */ }
  delete(id: string): Promise<void> { /* implementation */ }
  bulkInsert(entities: User[]): Promise<void> { /* implementation */ }
  bulkDelete(ids: string[]): Promise<void> { /* implementation */ }
}

// Services depend only on what they need
@Injectable()
export class UserQueryService {
  constructor(
    // Only needs read operations
    @Inject('IReadRepository') private repository: IReadRepository<User>,
  ) {}

  async getUser(id: string): Promise<User | null> {
    return this.repository.findById(id);
  }
}
```

---

### **Method 5: Dependency Inversion Principle (DIP)**

**Definition:** High-level modules should **not depend on low-level modules**. Both should depend on **abstractions**. Abstractions should not depend on details; details should depend on abstractions.

```typescript
// ========== DEPENDENCY INVERSION PRINCIPLE ==========

// ❌ BAD: High-level module depends on low-level module
import { TypeOrmUserRepository } from './typeorm-user.repository';  // Concrete class

@Injectable()
export class UserService {
  // Depends on concrete implementation!
  private userRepository: TypeOrmUserRepository;

  constructor() {
    this.userRepository = new TypeOrmUserRepository();
  }

  async getUser(id: string): Promise<User> {
    return this.userRepository.findById(id);
  }
}

// Problems:
// 1. Cannot swap implementations (tied to TypeORM)
// 2. Hard to test (cannot mock)
// 3. Violates DIP (depends on concretion)

// ✅ GOOD: Both depend on abstraction
// 1. Define abstraction (interface)
export interface IUserRepository {
  findById(id: string): Promise<User | null>;
  save(user: User): Promise<User>;
  delete(id: string): Promise<void>;
}

// 2. High-level module depends on abstraction
@Injectable()
export class UserService {
  constructor(
    // Depends on interface, not concrete class
    @Inject('IUserRepository')
    private readonly userRepository: IUserRepository,
  ) {}

  async getUser(id: string): Promise<User> {
    const user = await this.userRepository.findById(id);
    if (!user) {
      throw new NotFoundException('User not found');
    }
    return user;
  }

  async createUser(dto: CreateUserDto): Promise<User> {
    const user = new User();
    user.email = dto.email;
    return this.userRepository.save(user);
  }
}

// 3. Low-level modules implement abstraction
@Injectable()
export class TypeOrmUserRepository implements IUserRepository {
  constructor(
    @InjectRepository(UserEntity)
    private repository: Repository<UserEntity>,
  ) {}

  async findById(id: string): Promise<User | null> {
    const entity = await this.repository.findOne({ where: { id } });
    return entity ? this.toDomain(entity) : null;
  }

  async save(user: User): Promise<User> {
    const entity = this.toEntity(user);
    const saved = await this.repository.save(entity);
    return this.toDomain(saved);
  }

  async delete(id: string): Promise<void> {
    await this.repository.delete(id);
  }

  private toDomain(entity: UserEntity): User { /* mapping */ }
  private toEntity(user: User): UserEntity { /* mapping */ }
}

@Injectable()
export class InMemoryUserRepository implements IUserRepository {
  private users: Map<string, User> = new Map();

  async findById(id: string): Promise<User | null> {
    return this.users.get(id) || null;
  }

  async save(user: User): Promise<User> {
    this.users.set(user.id, user);
    return user;
  }

  async delete(id: string): Promise<void> {
    this.users.delete(id);
  }
}

// 4. Module configuration (dependency injection)
@Module({
  providers: [
    UserService,
    TypeOrmUserRepository,
    {
      provide: 'IUserRepository',
      useClass: TypeOrmUserRepository,  // Can easily swap to InMemoryUserRepository
    },
  ],
})
export class UserModule {}

// Benefits:
// 1. Easy to swap implementations (TypeORM → Prisma)
// 2. Easy to test (mock IUserRepository)
// 3. High-level logic independent of low-level details
```

---

### **SOLID Principles Summary:**

| Principle | Description | Key Benefit | Example |
|-----------|-------------|-------------|---------|
| **SRP** | One class, one responsibility | Easier to maintain | UserService (users), EmailService (emails) |
| **OCP** | Open for extension, closed for modification | Add features without breaking | Strategy pattern for payment methods |
| **LSP** | Subtypes must be substitutable | Polymorphism works correctly | All IRepository implementations work |
| **ISP** | Many specific interfaces > one general | Clients depend only on what they use | IReadRepository, IWriteRepository |
| **DIP** | Depend on abstractions, not concretions | Flexible, testable | Inject IUserRepository, not TypeOrmUserRepository |

---

### **Best Practices:**

```typescript
// ✅ GOOD: Follows all SOLID principles
export interface IUserRepository {  // DIP: abstraction
  findById(id: string): Promise<User | null>;
}

@Injectable()
export class UserService {  // SRP: only user business logic
  constructor(
    @Inject('IUserRepository')  // DIP: depend on abstraction
    private repository: IUserRepository,
  ) {}

  async getUser(id: string): Promise<User> {  // LSP: works with any IUserRepository
    return this.repository.findById(id);
  }
}

export class TypeOrmUserRepository implements IUserRepository {  // OCP: extend by adding new implementations
  async findById(id: string): Promise<User | null> {
    // Implementation
  }
}

// ❌ BAD: Violates multiple SOLID principles
@Injectable()
export class UserService {
  // SRP violation: user logic + email + logging
  async createUser(dto: CreateUserDto): Promise<User> {
    const user = new User();
    
    // DIP violation: depends on concrete TypeORM repository
    const repo = new TypeOrmUserRepository();
    await repo.save(user);
    
    // SRP violation: sending email in user service
    const transporter = nodemailer.createTransport({/*...*/});
    await transporter.sendMail({/*...*/});
    
    // SRP violation: logging in user service
    fs.appendFileSync('log.txt', 'User created');
    
    return user;
  }
}
```

**Key Takeaway:** **SOLID principles** improve code quality through **five guidelines**: **SRP** (Single Responsibility - one class, one job), **OCP** (Open/Closed - extend without modifying), **LSP** (Liskov Substitution - subtypes substitutable for base types), **ISP** (Interface Segregation - many specific interfaces over one general), and **DIP** (Dependency Inversion - depend on abstractions, not implementations). Benefits: **reduced coupling** (classes less dependent), **increased cohesion** (related code together), **better testability** (mock interfaces), **easier maintenance** (change one thing at a time), and **flexible architecture** (swap implementations). NestJS supports SOLID via **dependency injection** (DIP), **decorators** (SRP separation), **interfaces** (ISP, DIP), and **modular structure** (SRP, OCP). Apply by: separating concerns (one service per responsibility), using interfaces for dependencies, following strategy pattern for extensibility, and keeping abstractions clean.

</details>

<details>
<summary><strong>22. What is Single Responsibility Principle (SRP)?</strong></summary>

**Answer:**

**Single Responsibility Principle (SRP)** states that a class should have **one, and only one, reason to change** - meaning it should do **one thing** and do it well. In NestJS: **Controllers** handle HTTP (one endpoint group), **Services** contain business logic (one domain), **Repositories** manage data access (one entity), **Guards** handle authorization (one concern), **Pipes** validate/transform (one purpose). SRP prevents **God Classes** (do everything), improves **testability** (focused tests), enables **reusability** (small, focused classes), and simplifies **maintenance** (change one thing without affecting others).

---

### **Single Responsibility Principle Examples:**

```
┌────────────────────────────────────────────────────┐
│        SINGLE RESPONSIBILITY PRINCIPLE             │
│   "A class should have ONE reason to change"      │
├────────────────────────────────────────────────────┤
│                                                    │
│   ❌ UserService (Multiple responsibilities)      │
│      - User CRUD                                   │
│      - Send emails                                 │
│      - Log activities                              │
│      - Export data                                 │
│      - Validate passwords                          │
│   → 5 reasons to change!                          │
│                                                    │
│   ✅ Separated (Single responsibilities)          │
│      UserService       → User CRUD                 │
│      EmailService      → Send emails               │
│      LoggerService     → Log activities            │
│      ExportService     → Export data               │
│      ValidationService → Validate data             │
│   → 1 reason to change each!                      │
│                                                    │
└────────────────────────────────────────────────────┘
```

---

### **Method 1: Controller Responsibility**

```typescript
// ========== CONTROLLER: HTTP REQUEST HANDLING ONLY ==========

// ❌ BAD: Controller with multiple responsibilities
@Controller('users')
export class UsersController {
  constructor(private userRepository: Repository<User>) {}

  @Post()
  async createUser(@Body() dto: CreateUserDto) {
    // Responsibility 1: Validation
    if (!dto.email || !dto.email.includes('@')) {
      throw new BadRequestException('Invalid email');
    }

    // Responsibility 2: Business logic
    const existingUser = await this.userRepository.findOne({
      where: { email: dto.email },
    });
    if (existingUser) {
      throw new ConflictException('User already exists');
    }

    // Responsibility 3: Password hashing
    const hashedPassword = await bcrypt.hash(dto.password, 10);

    // Responsibility 4: Database access
    const user = this.userRepository.create({
      email: dto.email,
      password: hashedPassword,
    });
    await this.userRepository.save(user);

    // Responsibility 5: Email sending
    const transporter = nodemailer.createTransporter({/*...*/});
    await transporter.sendMail({
      to: user.email,
      subject: 'Welcome!',
    });

    // Responsibility 6: Logging
    console.log(`User created: ${user.id}`);

    return user;
  }
}

// Controller has 6 responsibilities!

// ✅ GOOD: Controller delegates to services
@Controller('users')
export class UsersController {
  constructor(private readonly userService: UserService) {}

  // Only responsibility: Handle HTTP request/response
  @Post()
  @HttpCode(HttpStatus.CREATED)
  async createUser(@Body() dto: CreateUserDto): Promise<UserResponseDto> {
    const user = await this.userService.createUser(dto);
    return this.toDto(user);
  }

  @Get(':id')
  async getUser(@Param('id') id: string): Promise<UserResponseDto> {
    const user = await this.userService.findById(id);
    if (!user) {
      throw new NotFoundException('User not found');
    }
    return this.toDto(user);
  }

  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  async deleteUser(@Param('id') id: string): Promise<void> {
    await this.userService.deleteUser(id);
  }

  private toDto(user: User): UserResponseDto {
    return {
      id: user.id,
      email: user.email,
      createdAt: user.createdAt,
    };
  }
}

// Controller's single responsibility: HTTP request/response handling
```

---

### **Method 2: Service Responsibility**

```typescript
// ========== SERVICE: BUSINESS LOGIC ONLY ==========

// ❌ BAD: Service with multiple responsibilities
@Injectable()
export class UserService {
  constructor(
    private userRepository: Repository<User>,
  ) {}

  async createUser(dto: CreateUserDto): Promise<User> {
    // Responsibility 1: Validation
    if (!dto.email.includes('@')) {
      throw new Error('Invalid email');
    }

    // Responsibility 2: Database access
    const existingUser = await this.userRepository.findOne({
      where: { email: dto.email },
    });

    if (existingUser) {
      throw new Error('User exists');
    }

    // Responsibility 3: Password hashing
    const hashedPassword = await bcrypt.hash(dto.password, 10);

    // Responsibility 4: Database persistence
    const user = this.userRepository.create({
      email: dto.email,
      password: hashedPassword,
    });
    await this.userRepository.save(user);

    // Responsibility 5: Email sending
    const transporter = nodemailer.createTransport({/*...*/});
    await transporter.sendMail({
      to: user.email,
      subject: 'Welcome!',
    });

    // Responsibility 6: Analytics tracking
    await fetch('https://analytics.example.com/track', {
      method: 'POST',
      body: JSON.stringify({ event: 'user_created', userId: user.id }),
    });

    // Responsibility 7: Cache invalidation
    await redis.del('users:list');

    return user;
  }
}

// ✅ GOOD: Service focused on business logic, delegates everything else
@Injectable()
export class UserService {
  constructor(
    private readonly userRepository: UserRepository,  // Data access
    private readonly emailService: EmailService,      // Email
    private readonly eventBus: EventEmitter2,         // Events
  ) {}

  // Single responsibility: User business logic orchestration
  async createUser(dto: CreateUserDto): Promise<User> {
    // Check business rule: email must be unique
    const existingUser = await this.userRepository.findByEmail(dto.email);
    
    if (existingUser) {
      throw new ConflictException('User with this email already exists');
    }

    // Create user (business logic)
    const user = User.create(dto.email, dto.password);

    // Persist
    await this.userRepository.save(user);

    // Emit event for side effects
    this.eventBus.emit('user.created', { userId: user.id, email: user.email });

    return user;
  }

  async findById(id: string): Promise<User | null> {
    return this.userRepository.findById(id);
  }

  async deleteUser(id: string): Promise<void> {
    const user = await this.userRepository.findById(id);
    
    if (!user) {
      throw new NotFoundException('User not found');
    }

    await this.userRepository.delete(id);
    
    this.eventBus.emit('user.deleted', { userId: id });
  }
}

// Separate services for separate concerns
@Injectable()
export class EmailService {
  // Single responsibility: Email sending
  async sendWelcomeEmail(email: string): Promise<void> {
    const transporter = nodemailer.createTransporter({/*...*/});
    await transporter.sendMail({
      to: email,
      subject: 'Welcome!',
      html: '<h1>Welcome to our app!</h1>',
    });
  }
}

@Injectable()
export class UserCreatedHandler {
  constructor(
    private readonly emailService: EmailService,
    private readonly analyticsService: AnalyticsService,
    private readonly cacheService: CacheService,
  ) {}

  // Single responsibility: Handle user creation side effects
  @OnEvent('user.created')
  async handle(payload: { userId: string; email: string }): Promise<void> {
    // Send email
    await this.emailService.sendWelcomeEmail(payload.email);
    
    // Track analytics
    await this.analyticsService.track('user_created', { userId: payload.userId });
    
    // Invalidate cache
    await this.cacheService.del('users:list');
  }
}
```

---

### **Method 3: Repository Responsibility**

```typescript
// ========== REPOSITORY: DATA ACCESS ONLY ==========

// ❌ BAD: Repository with multiple responsibilities
@Injectable()
export class UserRepository {
  constructor(
    @InjectRepository(UserEntity)
    private repository: Repository<UserEntity>,
  ) {}

  async save(user: User): Promise<User> {
    // Responsibility 1: Data access
    const entity = this.toEntity(user);
    await this.repository.save(entity);

    // Responsibility 2: Cache management (wrong place!)
    await redis.set(`user:${user.id}`, JSON.stringify(user));

    // Responsibility 3: Search indexing (wrong place!)
    await elasticsearch.index({
      index: 'users',
      id: user.id,
      body: user,
    });

    // Responsibility 4: Logging (wrong place!)
    console.log(`User saved: ${user.id}`);

    // Responsibility 5: Email notification (VERY wrong place!)
    await this.sendNotification(user);

    return user;
  }

  private async sendNotification(user: User): Promise<void> {
    // Email logic in repository!
  }
}

// ✅ GOOD: Repository only handles data access
@Injectable()
export class UserRepository {
  constructor(
    @InjectRepository(UserEntity)
    private readonly repository: Repository<UserEntity>,
  ) {}

  // Single responsibility: Data persistence and retrieval
  async save(user: User): Promise<User> {
    const entity = this.toEntity(user);
    const saved = await this.repository.save(entity);
    return this.toDomain(saved);
  }

  async findById(id: string): Promise<User | null> {
    const entity = await this.repository.findOne({ where: { id } });
    return entity ? this.toDomain(entity) : null;
  }

  async findByEmail(email: string): Promise<User | null> {
    const entity = await this.repository.findOne({ where: { email } });
    return entity ? this.toDomain(entity) : null;
  }

  async delete(id: string): Promise<void> {
    await this.repository.delete(id);
  }

  private toEntity(user: User): UserEntity {
    // Mapping logic
  }

  private toDomain(entity: UserEntity): User {
    // Mapping logic
  }
}

// Cache management in separate service
@Injectable()
export class UserCacheService {
  // Single responsibility: User cache management
  async set(user: User): Promise<void> {
    await redis.set(`user:${user.id}`, JSON.stringify(user), 'EX', 3600);
  }

  async get(id: string): Promise<User | null> {
    const cached = await redis.get(`user:${id}`);
    return cached ? JSON.parse(cached) : null;
  }

  async invalidate(id: string): Promise<void> {
    await redis.del(`user:${id}`);
  }
}

// Search indexing in separate service
@Injectable()
export class UserSearchService {
  // Single responsibility: User search indexing
  async index(user: User): Promise<void> {
    await elasticsearch.index({
      index: 'users',
      id: user.id,
      body: {
        email: user.email,
        name: user.name,
        createdAt: user.createdAt,
      },
    });
  }

  async remove(id: string): Promise<void> {
    await elasticsearch.delete({
      index: 'users',
      id,
    });
  }
}
```

---

### **Method 4: NestJS Building Blocks Follow SRP**

```typescript
// ========== EACH NESTJS COMPONENT HAS ONE RESPONSIBILITY ==========

// ✅ Guard: Only authentication/authorization
@Injectable()
export class AuthGuard implements CanActivate {
  // Single responsibility: Check if user is authenticated
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    return !!request.user;
  }
}

// ✅ Pipe: Only validation/transformation
@Injectable()
export class ParseIntPipe implements PipeTransform {
  // Single responsibility: Transform string to integer
  transform(value: string): number {
    const val = parseInt(value, 10);
    if (isNaN(val)) {
      throw new BadRequestException('Validation failed');
    }
    return val;
  }
}

// ✅ Interceptor: Only cross-cutting concern (logging)
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  // Single responsibility: Log requests/responses
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const now = Date.now();
    return next.handle().pipe(
      tap(() => console.log(`Request took ${Date.now() - now}ms`)),
    );
  }
}

// ✅ Filter: Only error handling
@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  // Single responsibility: Format HTTP exceptions
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const status = exception.getStatus();

    response.status(status).json({
      statusCode: status,
      message: exception.message,
      timestamp: new Date().toISOString(),
    });
  }
}

// ✅ DTO: Only data transfer structure
export class CreateUserDto {
  // Single responsibility: Define user creation data structure
  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  password: string;
}

// ✅ Entity: Only domain model
@Entity('users')
export class User {
  // Single responsibility: Represent user domain model
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  email: string;

  @Column()
  password: string;

  @Column({ default: true })
  isActive: boolean;
}
```

---

### **Method 5: Recognizing SRP Violations**

```typescript
// ========== HOW TO SPOT SRP VIOLATIONS ==========

// ❌ Signs of SRP violation:

// 1. Class name with "And", "Manager", "Handler", "Utility"
export class UserAndOrderManager {}  // Does 2+ things
export class DataManager {}  // Too generic, does everything
export class UtilityHelper {}  // Grab bag of unrelated functions

// 2. Too many dependencies (>5 injections)
@Injectable()
export class UserService {
  constructor(
    private userRepo: UserRepository,
    private emailService: EmailService,
    private smsService: SmsService,
    private cacheService: CacheService,
    private loggerService: LoggerService,
    private analyticsService: AnalyticsService,
    private notificationService: NotificationService,
    private exportService: ExportService,
  ) {}
  // Too many dependencies = doing too much!
}

// 3. Large classes (>300 lines)
@Injectable()
export class UserService {
  // 50 methods, 500 lines
  // Doing way too much!
}

// 4. Methods unrelated to class name
export class UserService {
  createUser() {}  // OK
  deleteUser() {}  // OK
  sendEmail() {}  // Not user business logic!
  generatePdf() {}  // Not user business logic!
  processPayment() {}  // Not user business logic!
}

// ✅ How to fix SRP violations:

// 1. Extract separate services
@Injectable()
export class UserService {
  // Only user business logic
  createUser() {}
  deleteUser() {}
}

@Injectable()
export class EmailService {
  // Only email
  sendEmail() {}
}

@Injectable()
export class PdfService {
  // Only PDF generation
  generatePdf() {}
}

@Injectable()
export class PaymentService {
  // Only payments
  processPayment() {}
}

// 2. Use events to decouple side effects
@Injectable()
export class UserService {
  constructor(private eventBus: EventEmitter2) {}

  async createUser(dto: CreateUserDto): Promise<User> {
    const user = await this.userRepository.save(dto);
    
    // Don't call email/SMS/analytics directly
    // Emit event instead
    this.eventBus.emit('user.created', user);
    
    return user;
  }
}

@Injectable()
export class UserCreatedHandler {
  @OnEvent('user.created')
  async handle(user: User): Promise<void> {
    // Side effects in separate handler
    await this.emailService.sendWelcomeEmail(user.email);
    await this.analyticsService.track('user_created');
  }
}
```

---

### **SRP Benefits:**

| Benefit | Description | Example |
|---------|-------------|---------|
| **Easier to understand** | Class does one thing | UserService only handles user logic |
| **Easier to test** | Focused unit tests | Test user logic without testing email |
| **Easier to maintain** | Changes isolated | Change email logic without touching user logic |
| **More reusable** | Smaller, focused classes | EmailService used by UserService, OrderService |
| **Less coupling** | Classes don't depend on unrelated code | UserService doesn't depend on PDF generation |
| **Easier to extend** | Add new functionality without modifying existing | Add SMS without modifying EmailService |

---

### **Best Practices:**

```typescript
// ✅ GOOD: Single responsibility per class
@Controller('users')
export class UsersController {
  // HTTP only
}

@Injectable()
export class UserService {
  // Business logic only
}

@Injectable()
export class UserRepository {
  // Data access only
}

@Injectable()
export class EmailService {
  // Email only
}

// ❌ BAD: Multiple responsibilities in one class
@Controller('users')
export class UsersController {
  // HTTP + business logic + data access + email
  @Post()
  async create(@Body() dto: CreateUserDto) {
    const user = await this.db.query('INSERT...');  // Data access in controller!
    await this.sendEmail(user);  // Email in controller!
    return user;
  }
}

// ✅ GOOD: Use events for side effects
@Injectable()
export class UserService {
  async createUser(dto: CreateUserDto): Promise<User> {
    const user = await this.repository.save(dto);
    this.eventBus.emit('user.created', user);  // Decouple side effects
    return user;
  }
}

// ❌ BAD: Call everything directly
@Injectable()
export class UserService {
  async createUser(dto: CreateUserDto): Promise<User> {
    const user = await this.repository.save(dto);
    await this.emailService.send(user);  // Tight coupling
    await this.smsService.send(user);
    await this.analyticsService.track(user);
    return user;
  }
}

// ✅ GOOD: Descriptive, focused class names
export class UserService {}
export class EmailService {}
export class PaymentService {}

// ❌ BAD: Generic, vague names
export class UserManager {}  // What does it manage?
export class DataHandler {}  // What data? What handling?
export class Utils {}  // Everything!
```

**Key Takeaway:** **Single Responsibility Principle (SRP)** means **one class, one job, one reason to change**. In NestJS: **Controllers** handle HTTP only (routing, request/response), **Services** contain business logic only (user operations, not email), **Repositories** manage data access only (database, not caching), **Guards/Pipes/Interceptors** handle one cross-cutting concern each. Benefits: **easier testing** (focused tests), **better maintainability** (change one thing), **increased reusability** (small, composable classes), **reduced coupling** (classes independent). Violations: large classes (>300 lines), too many dependencies (>5 injections), unrelated methods (sendEmail in UserService), vague names (Manager, Handler). Fix by: **extracting services** (EmailService, PdfService), **using events** (decouple side effects), **descriptive names** (UserService not UserManager). Ask: "What is this class's ONE job?" and "How many reasons would I have to change this class?"

</details>

<details>
<summary><strong>23. What is Dependency Inversion Principle (DIP)?</strong></summary>

**Answer:**

**Dependency Inversion Principle (DIP)** states that **high-level modules should not depend on low-level modules**; both should depend on **abstractions** (interfaces). Additionally, **abstractions should not depend on details**; details should depend on abstractions. In NestJS: inject **interfaces** instead of concrete classes, use **providers with tokens** (`@Inject('IUserRepository')`), implement **repository patterns** (interface in domain, implementation in infrastructure), and leverage **NestJS dependency injection** system. DIP enables **testability** (mock interfaces), **flexibility** (swap implementations), and **maintainability** (change implementation without modifying consumers).

---

### **Dependency Inversion Principle Architecture:**

```
┌───────────────────────────────────────────────────────┐
│     DEPENDENCY INVERSION PRINCIPLE (DIP)              │
├───────────────────────────────────────────────────────┤
│                                                       │
│   WITHOUT DIP (❌ BAD):                              │
│                                                       │
│   ┌─────────────────┐                                │
│   │  UserService    │  High-level module             │
│   │  (business      │                                │
│   │   logic)        │                                │
│   └────────┬────────┘                                │
│            │ depends on (↓)                          │
│            ▼                                          │
│   ┌─────────────────┐                                │
│   │ TypeOrmUserRepo │  Low-level module              │
│   │ (concrete       │                                │
│   │  implementation)│                                │
│   └─────────────────┘                                │
│                                                       │
│   Problem: UserService tied to TypeORM!              │
│                                                       │
├───────────────────────────────────────────────────────┤
│                                                       │
│   WITH DIP (✅ GOOD):                                │
│                                                       │
│   ┌─────────────────┐                                │
│   │  UserService    │  High-level module             │
│   │  (business      │                                │
│   │   logic)        │                                │
│   └────────┬────────┘                                │
│            │ depends on (↓)                          │
│            ▼                                          │
│   ┌─────────────────┐                                │
│   │ IUserRepository │  Abstraction (interface)       │
│   │   (interface)   │                                │
│   └────────▲────────┘                                │
│            │ implements (↑)                          │
│            │                                          │
│   ┌────────┴────────┐                                │
│   │ TypeOrmUserRepo │  Low-level module              │
│   │ (implementation)│                                │
│   └─────────────────┘                                │
│                                                       │
│   Both depend on abstraction!                        │
│   Easy to swap implementations                       │
│                                                       │
└───────────────────────────────────────────────────────┘
```

---

### **Method 1: Basic DIP with Interfaces**

```typescript
// ========== DEPENDENCY INVERSION PRINCIPLE ==========

// ❌ BAD: Service depends on concrete repository class
import { TypeOrmUserRepository } from './typeorm-user.repository';

@Injectable()
export class UserService {
  // Direct dependency on concrete implementation!
  constructor(private userRepository: TypeOrmUserRepository) {}

  async getUser(id: string): Promise<User> {
    return this.userRepository.findById(id);
  }
}

// Problems:
// 1. Cannot swap TypeORM for Prisma without changing UserService
// 2. Hard to test (cannot mock TypeOrmUserRepository easily)
// 3. UserService knows about database implementation details
// 4. Violates DIP (high-level depends on low-level)

// ✅ GOOD: Service depends on interface
// Step 1: Define abstraction (interface)
export interface IUserRepository {
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  save(user: User): Promise<User>;
  delete(id: string): Promise<void>;
}

// Step 2: High-level module depends on interface
@Injectable()
export class UserService {
  constructor(
    // Depend on abstraction, not concretion
    @Inject('IUserRepository')
    private readonly userRepository: IUserRepository,
  ) {}

  async getUser(id: string): Promise<User> {
    const user = await this.userRepository.findById(id);
    
    if (!user) {
      throw new NotFoundException('User not found');
    }
    
    return user;
  }

  async createUser(email: string, password: string): Promise<User> {
    // Check business rule
    const existingUser = await this.userRepository.findByEmail(email);
    
    if (existingUser) {
      throw new ConflictException('User already exists');
    }

    // Create and save
    const user = new User();
    user.email = email;
    user.password = await this.hashPassword(password);
    
    return this.userRepository.save(user);
  }

  private async hashPassword(password: string): Promise<string> {
    return bcrypt.hash(password, 10);
  }
}

// Step 3: Low-level module implements interface
@Injectable()
export class TypeOrmUserRepository implements IUserRepository {
  constructor(
    @InjectRepository(UserEntity)
    private repository: Repository<UserEntity>,
  ) {}

  async findById(id: string): Promise<User | null> {
    const entity = await this.repository.findOne({ where: { id } });
    return entity ? this.toDomain(entity) : null;
  }

  async findByEmail(email: string): Promise<User | null> {
    const entity = await this.repository.findOne({ where: { email } });
    return entity ? this.toDomain(entity) : null;
  }

  async save(user: User): Promise<User> {
    const entity = this.toEntity(user);
    const saved = await this.repository.save(entity);
    return this.toDomain(saved);
  }

  async delete(id: string): Promise<void> {
    await this.repository.delete(id);
  }

  private toDomain(entity: UserEntity): User {
    const user = new User();
    user.id = entity.id;
    user.email = entity.email;
    user.password = entity.password;
    return user;
  }

  private toEntity(user: User): UserEntity {
    const entity = new UserEntity();
    entity.id = user.id;
    entity.email = user.email;
    entity.password = user.password;
    return entity;
  }
}

// Step 4: Alternative implementation (Prisma)
@Injectable()
export class PrismaUserRepository implements IUserRepository {
  constructor(private prisma: PrismaService) {}

  async findById(id: string): Promise<User | null> {
    const entity = await this.prisma.user.findUnique({ where: { id } });
    return entity ? this.toDomain(entity) : null;
  }

  async findByEmail(email: string): Promise<User | null> {
    const entity = await this.prisma.user.findUnique({ where: { email } });
    return entity ? this.toDomain(entity) : null;
  }

  async save(user: User): Promise<User> {
    const saved = await this.prisma.user.create({
      data: {
        email: user.email,
        password: user.password,
      },
    });
    return this.toDomain(saved);
  }

  async delete(id: string): Promise<void> {
    await this.prisma.user.delete({ where: { id } });
  }

  private toDomain(entity: any): User {
    const user = new User();
    user.id = entity.id;
    user.email = entity.email;
    user.password = entity.password;
    return user;
  }
}

// Step 5: Module configuration (Dependency Injection)
@Module({
  providers: [
    UserService,
    TypeOrmUserRepository,
    // PrismaUserRepository,
    {
      provide: 'IUserRepository',
      useClass: TypeOrmUserRepository,  // Easy to swap!
      // useClass: PrismaUserRepository,  // Just change this line
    },
  ],
  exports: [UserService],
})
export class UserModule {}

// Benefits:
// 1. Easy to swap TypeORM → Prisma (change one line)
// 2. Easy to test (mock IUserRepository)
// 3. UserService knows nothing about database implementation
// 4. Follows DIP (both depend on IUserRepository interface)
```

---

### **Method 2: Testing with DIP**

```typescript
// ========== TESTING BECOMES EASY WITH DIP ==========

// ❌ BAD: Without DIP, testing is hard
@Injectable()
export class UserService {
  constructor(private userRepository: TypeOrmUserRepository) {}

  async getUser(id: string): Promise<User> {
    return this.userRepository.findById(id);
  }
}

// Test requires actual TypeORM setup!
describe('UserService', () => {
  let service: UserService;
  let repository: TypeOrmUserRepository;

  beforeEach(async () => {
    // Must set up entire TypeORM infrastructure
    const module = await Test.createTestingModule({
      imports: [TypeOrmModule.forRoot({/*...*/})],
      providers: [UserService, TypeOrmUserRepository],
    }).compile();

    service = module.get(UserService);
  });

  it('should get user', async () => {
    // Must use real database or complex mocking
  });
});

// ✅ GOOD: With DIP, testing is simple
@Injectable()
export class UserService {
  constructor(
    @Inject('IUserRepository')
    private userRepository: IUserRepository,
  ) {}

  async getUser(id: string): Promise<User> {
    const user = await this.userRepository.findById(id);
    
    if (!user) {
      throw new NotFoundException('User not found');
    }
    
    return user;
  }
}

// Test with mock implementation
describe('UserService', () => {
  let service: UserService;
  let mockRepository: IUserRepository;

  beforeEach(async () => {
    // Simple mock implementation
    mockRepository = {
      findById: jest.fn(),
      findByEmail: jest.fn(),
      save: jest.fn(),
      delete: jest.fn(),
    };

    const module = await Test.createTestingModule({
      providers: [
        UserService,
        {
          provide: 'IUserRepository',
          useValue: mockRepository,  // Inject mock
        },
      ],
    }).compile();

    service = module.get(UserService);
  });

  it('should get user', async () => {
    const mockUser = new User();
    mockUser.id = '123';
    mockUser.email = 'test@example.com';

    // Mock returns predefined value
    (mockRepository.findById as jest.Mock).mockResolvedValue(mockUser);

    const result = await service.getUser('123');

    expect(result).toEqual(mockUser);
    expect(mockRepository.findById).toHaveBeenCalledWith('123');
  });

  it('should throw NotFoundException when user not found', async () => {
    // Mock returns null
    (mockRepository.findById as jest.Mock).mockResolvedValue(null);

    await expect(service.getUser('123')).rejects.toThrow(NotFoundException);
  });
});

// No database, no TypeORM, just pure logic testing!
```

---

### **Method 3: Multiple Implementations**

```typescript
// ========== MULTIPLE IMPLEMENTATIONS OF SAME INTERFACE ==========

export interface IUserRepository {
  findById(id: string): Promise<User | null>;
  save(user: User): Promise<User>;
}

// Implementation 1: TypeORM
@Injectable()
export class TypeOrmUserRepository implements IUserRepository {
  constructor(
    @InjectRepository(UserEntity)
    private repository: Repository<UserEntity>,
  ) {}

  async findById(id: string): Promise<User | null> {
    const entity = await this.repository.findOne({ where: { id } });
    return entity ? this.toDomain(entity) : null;
  }

  async save(user: User): Promise<User> {
    const entity = this.toEntity(user);
    const saved = await this.repository.save(entity);
    return this.toDomain(saved);
  }

  private toDomain(entity: UserEntity): User { /* mapping */ }
  private toEntity(user: User): UserEntity { /* mapping */ }
}

// Implementation 2: In-Memory (for testing)
@Injectable()
export class InMemoryUserRepository implements IUserRepository {
  private users: Map<string, User> = new Map();

  async findById(id: string): Promise<User | null> {
    return this.users.get(id) || null;
  }

  async save(user: User): Promise<User> {
    this.users.set(user.id, user);
    return user;
  }

  clear(): void {
    this.users.clear();
  }
}

// Implementation 3: Cached Repository
@Injectable()
export class CachedUserRepository implements IUserRepository {
  constructor(
    @Inject('IUserRepository')
    private baseRepository: IUserRepository,
    private cacheService: CacheService,
  ) {}

  async findById(id: string): Promise<User | null> {
    // Check cache first
    const cached = await this.cacheService.get<User>(`user:${id}`);
    
    if (cached) {
      return cached;
    }

    // Fetch from base repository
    const user = await this.baseRepository.findById(id);

    // Cache result
    if (user) {
      await this.cacheService.set(`user:${id}`, user, 3600);
    }

    return user;
  }

  async save(user: User): Promise<User> {
    const saved = await this.baseRepository.save(user);
    
    // Invalidate cache
    await this.cacheService.del(`user:${user.id}`);
    
    return saved;
  }
}

// Module configuration - easy to switch!
@Module({
  providers: [
    UserService,
    TypeOrmUserRepository,
    InMemoryUserRepository,
    {
      provide: 'IUserRepository',
      // Development: In-memory
      // useClass: InMemoryUserRepository,
      
      // Production: TypeORM
      useClass: TypeOrmUserRepository,
      
      // Production with cache: Cached
      // useClass: CachedUserRepository,
    },
  ],
})
export class UserModule {}
```

---

### **Method 4: Repository Pattern with DIP**

```typescript
// ========== REPOSITORY PATTERN FOLLOWS DIP ==========

// Domain Layer: Define interface (abstraction)
// orders/domain/repositories/orders-repository.interface.ts
export interface IOrdersRepository {
  save(order: Order): Promise<void>;
  findById(orderId: string): Promise<Order | null>;
  findByCustomerId(customerId: string): Promise<Order[]>;
}

// Application Layer: Use interface
// orders/application/use-cases/place-order.use-case.ts
@Injectable()
export class PlaceOrderUseCase {
  constructor(
    @Inject('IOrdersRepository')
    private readonly ordersRepository: IOrdersRepository,  // Depend on interface
  ) {}

  async execute(dto: PlaceOrderDto): Promise<Order> {
    const order = Order.create(dto.customerId);
    
    // Add items...
    dto.items.forEach(item => {
      order.addItem(item.productId, item.quantity, item.price);
    });

    order.placeOrder();

    await this.ordersRepository.save(order);

    return order;
  }
}

// Infrastructure Layer: Implement interface
// orders/infrastructure/repositories/typeorm-orders.repository.ts
@Injectable()
export class TypeOrmOrdersRepository implements IOrdersRepository {
  constructor(
    @InjectRepository(OrderEntity)
    private repository: Repository<OrderEntity>,
  ) {}

  async save(order: Order): Promise<void> {
    const entity = this.toEntity(order);
    await this.repository.save(entity);
  }

  async findById(orderId: string): Promise<Order | null> {
    const entity = await this.repository.findOne({
      where: { id: orderId },
      relations: ['items'],
    });
    return entity ? this.toDomain(entity) : null;
  }

  async findByCustomerId(customerId: string): Promise<Order[]> {
    const entities = await this.repository.find({
      where: { customerId },
      relations: ['items'],
    });
    return entities.map(entity => this.toDomain(entity));
  }

  private toDomain(entity: OrderEntity): Order { /* mapping */ }
  private toEntity(order: Order): OrderEntity { /* mapping */ }
}

// Module: Wire dependencies
@Module({
  imports: [TypeOrmModule.forFeature([OrderEntity])],
  providers: [
    PlaceOrderUseCase,
    TypeOrmOrdersRepository,
    {
      provide: 'IOrdersRepository',
      useClass: TypeOrmOrdersRepository,
    },
  ],
})
export class OrdersModule {}

// Layered architecture with DIP:
// Domain (interface) ← Application (uses interface) ← Infrastructure (implements interface)
// No dependencies from domain/application to infrastructure!
```

---

### **Method 5: Custom Providers with Tokens**

```typescript
// ========== CUSTOM PROVIDERS WITH INJECTION TOKENS ==========

// Define token
export const USER_REPOSITORY = 'USER_REPOSITORY';

export interface IUserRepository {
  findById(id: string): Promise<User | null>;
  save(user: User): Promise<User>;
}

// Service uses token
@Injectable()
export class UserService {
  constructor(
    @Inject(USER_REPOSITORY)
    private readonly userRepository: IUserRepository,
  ) {}

  async getUser(id: string): Promise<User> {
    return this.userRepository.findById(id);
  }
}

// Module provides implementation
@Module({
  providers: [
    UserService,
    TypeOrmUserRepository,
    {
      provide: USER_REPOSITORY,  // Token
      useClass: TypeOrmUserRepository,  // Implementation
    },
  ],
})
export class UserModule {}

// ===== FACTORY PROVIDERS =====
// Dynamic implementation selection

@Module({
  providers: [
    UserService,
    TypeOrmUserRepository,
    InMemoryUserRepository,
    {
      provide: USER_REPOSITORY,
      useFactory: (configService: ConfigService) => {
        // Select implementation based on environment
        const env = configService.get('NODE_ENV');
        
        if (env === 'test') {
          return new InMemoryUserRepository();
        } else {
          return new TypeOrmUserRepository(/* inject dependencies */);
        }
      },
      inject: [ConfigService],
    },
  ],
})
export class UserModule {}

// ===== VALUE PROVIDERS =====
// Provide mock for testing

@Module({
  providers: [
    UserService,
    {
      provide: USER_REPOSITORY,
      useValue: {
        findById: jest.fn(),
        save: jest.fn(),
      },
    },
  ],
})
export class TestUserModule {}
```

---

### **DIP Benefits:**

| Benefit | Description | Example |
|---------|-------------|---------|
| **Testability** | Easy to mock dependencies | Mock IUserRepository instead of TypeORM |
| **Flexibility** | Swap implementations easily | TypeORM → Prisma in one line |
| **Maintainability** | Changes isolated to implementations | Update TypeORM version without touching service |
| **Decoupling** | High-level logic independent of details | UserService doesn't know about database |
| **Reusability** | Same interface, multiple implementations | In-memory for tests, TypeORM for prod |
| **Clean Architecture** | Dependencies point inward | Domain/Application don't depend on Infrastructure |

---

### **Best Practices:**

```typescript
// ✅ GOOD: Depend on interface
@Injectable()
export class UserService {
  constructor(
    @Inject('IUserRepository')
    private repository: IUserRepository,  // Interface
  ) {}
}

// ❌ BAD: Depend on concrete class
@Injectable()
export class UserService {
  constructor(
    private repository: TypeOrmUserRepository,  // Concrete class
  ) {}
}

// ✅ GOOD: Interface doesn't depend on implementation details
export interface IUserRepository {
  findById(id: string): Promise<User>;  // Domain entity
}

// ❌ BAD: Interface depends on TypeORM
export interface IUserRepository {
  findById(id: string): Promise<Repository<UserEntity>>;  // TypeORM leak!
}

// ✅ GOOD: Module binds interface to implementation
@Module({
  providers: [
    {
      provide: 'IUserRepository',
      useClass: TypeOrmUserRepository,
    },
  ],
})
export class UserModule {}

// ❌ BAD: Service creates repository directly
@Injectable()
export class UserService {
  private repository = new TypeOrmUserRepository();  // Direct instantiation!
}

// ✅ GOOD: Interface in domain layer, implementation in infrastructure
src/
  users/
    domain/
      repositories/
        user-repository.interface.ts  // Interface here
    infrastructure/
      repositories/
        typeorm-user.repository.ts    // Implementation here

// ❌ BAD: Interface and implementation mixed
src/
  users/
    user.repository.ts  // Interface and implementation together
```

**Key Takeaway:** **Dependency Inversion Principle (DIP)** means **depend on abstractions (interfaces), not concretions (classes)**. High-level modules (services with business logic) shouldn't depend on low-level modules (database repositories); both depend on **interfaces**. In NestJS: define **repository interfaces** (`IUserRepository`), inject with **tokens** (`@Inject('IUserRepository')`), provide **implementations** in module configuration (`useClass: TypeOrmUserRepository`), and keep **interfaces in domain layer**, implementations in infrastructure. Benefits: **testability** (mock interfaces easily), **flexibility** (swap TypeORM for Prisma in one line), **maintainability** (change implementation without touching consumers), **decoupling** (business logic independent of database), and **clean architecture** (dependencies point inward). Follow pattern: interface → high-level uses interface → low-level implements interface → module wires together via DI.

</details>

<details>
<summary><strong>24. How does NestJS enforce SOLID principles?</strong></summary>

**Answer:**

NestJS **enforces SOLID principles** through its **architecture and built-in features**: **Dependency Injection** (DIP - depend on abstractions via `@Inject()`), **Modular structure** (SRP - one module per feature), **Decorators** (SRP - separate concerns with `@Controller`, `@Injectable`, `@Guard`), **Provider system** (OCP, DIP - extensible via custom providers), **Guards/Pipes/Interceptors** (ISP - specific interfaces), and **Interface-based programming** (LSP - subtypes substitutable). NestJS **encourages** SOLID through conventions: services for business logic, controllers for HTTP, repositories for data access, modules for organization, and dependency injection for decoupling.

---

### **How NestJS Enforces SOLID:**

```
┌────────────────────────────────────────────────────────┐
│       NestJS FEATURES → SOLID PRINCIPLES               │
├────────────────────────────────────────────────────────┤
│                                                        │
│  1️⃣  DEPENDENCY INJECTION                             │
│      → Enforces DIP                                    │
│      Constructor injection, @Inject(), providers      │
│                                                        │
│  2️⃣  MODULAR ARCHITECTURE                             │
│      → Enforces SRP, OCP                              │
│      @Module(), feature modules, shared modules       │
│                                                        │
│  3️⃣  DECORATORS                                        │
│      → Enforces SRP                                    │
│      @Controller(), @Injectable(), @Guard()           │
│                                                        │
│  4️⃣  PROVIDER SYSTEM                                   │
│      → Enforces OCP, DIP                              │
│      useClass, useFactory, useValue                   │
│                                                        │
│  5️⃣  GUARDS/PIPES/INTERCEPTORS/FILTERS                │
│      → Enforces ISP, SRP                              │
│      Specific interfaces for specific concerns        │
│                                                        │
│  6️⃣  INTERFACE-BASED PROGRAMMING                      │
│      → Enforces LSP, DIP                              │
│      Repository patterns, strategy patterns           │
│                                                        │
└────────────────────────────────────────────────────────┘
```

---

### **Method 1: Dependency Injection Enforces DIP**

```typescript
// ========== DI ENFORCES DEPENDENCY INVERSION PRINCIPLE ==========

// NestJS forces you to use dependency injection
// Cannot directly instantiate dependencies (unless you explicitly try)

// ✅ NestJS Way: Constructor injection (DIP enforced)
@Injectable()
export class UserService {
  constructor(
    // NestJS injects dependencies
    @Inject('IUserRepository')
    private readonly userRepository: IUserRepository,
    private readonly emailService: EmailService,
  ) {}

  async createUser(dto: CreateUserDto): Promise<User> {
    const user = await this.userRepository.save(dto);
    await this.emailService.sendWelcome(user.email);
    return user;
  }
}

// Module wires dependencies
@Module({
  providers: [
    UserService,
    EmailService,
    TypeOrmUserRepository,
    {
      provide: 'IUserRepository',
      useClass: TypeOrmUserRepository,  // Easy to swap
    },
  ],
})
export class UserModule {}

// ❌ Anti-pattern: Direct instantiation (breaks DIP)
@Injectable()
export class UserService {
  private userRepository = new TypeOrmUserRepository();  // Bad!
  private emailService = new EmailService();  // Bad!

  async createUser(dto: CreateUserDto): Promise<User> {
    // Now tightly coupled to concrete implementations
  }
}

// NestJS DI System Benefits:
// 1. Forces abstraction-based programming
// 2. Makes testing easy (inject mocks)
// 3. Enables swapping implementations
// 4. Manages object lifecycle (singletons, transient, request-scoped)
```

---

### **Method 2: Modular Architecture Enforces SRP & OCP**

```typescript
// ========== MODULES ENFORCE SINGLE RESPONSIBILITY ==========

// NestJS encourages one module per feature (SRP)

// ✅ Each module has single responsibility
@Module({
  imports: [TypeOrmModule.forFeature([UserEntity])],
  controllers: [UsersController],  // HTTP handling
  providers: [
    UserService,  // Business logic
    UserRepository,  // Data access
  ],
  exports: [UserService],  // What to expose
})
export class UsersModule {}

@Module({
  controllers: [OrdersController],
  providers: [OrderService, OrderRepository],
  exports: [OrderService],
})
export class OrdersModule {}

@Module({
  providers: [EmailService],
  exports: [EmailService],
})
export class EmailModule {}

// ✅ Main app module composes features (OCP)
@Module({
  imports: [
    UsersModule,     // Add feature
    OrdersModule,    // Add feature
    EmailModule,     // Add feature
    PaymentsModule,  // Add new feature without modifying existing
  ],
})
export class AppModule {}

// ❌ Anti-pattern: God module (violates SRP)
@Module({
  controllers: [
    UsersController,
    OrdersController,
    ProductsController,
    PaymentsController,
    // Everything in one module!
  ],
  providers: [
    UserService,
    OrderService,
    ProductService,
    PaymentService,
    EmailService,
    // Too many responsibilities!
  ],
})
export class AppModule {}

// NestJS Module System Benefits:
// 1. Forces feature-based organization (SRP)
// 2. Easy to add new features (OCP)
// 3. Clear boundaries between domains
// 4. Encapsulation (exports control what's visible)
```

---

### **Method 3: Decorators Enforce SRP**

```typescript
// ========== DECORATORS SEPARATE CONCERNS (SRP) ==========

// NestJS decorators force separation of concerns

// ✅ Controller: HTTP only
@Controller('users')
export class UsersController {
  constructor(private readonly userService: UserService) {}

  @Post()
  async create(@Body() dto: CreateUserDto): Promise<UserResponseDto> {
    return this.userService.createUser(dto);
  }
}

// ✅ Service: Business logic only
@Injectable()
export class UserService {
  constructor(
    @Inject('IUserRepository')
    private repository: IUserRepository,
  ) {}

  async createUser(dto: CreateUserDto): Promise<User> {
    // Business logic
  }
}

// ✅ Guard: Authentication only
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    // Authentication logic only
    const request = context.switchToHttp().getRequest();
    return !!request.user;
  }
}

// ✅ Pipe: Validation only
@Injectable()
export class ValidationPipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    // Validation logic only
  }
}

// ✅ Interceptor: Cross-cutting concern only
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    // Logging logic only
    console.log('Before...');
    return next.handle().pipe(tap(() => console.log('After...')));
  }
}

// ✅ Filter: Error handling only
@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    // Error handling logic only
  }
}

// Each decorator enforces a specific responsibility!
// Cannot mix controller logic with service logic
// Clear separation of concerns
```

---

### **Method 4: Provider System Enforces OCP & DIP**

```typescript
// ========== PROVIDER SYSTEM ENABLES EXTENSIBILITY (OCP, DIP) ==========

// NestJS providers enable extension without modification (OCP)

// ✅ Strategy pattern with providers
export interface IPaymentProcessor {
  process(amount: number): Promise<void>;
}

@Injectable()
export class CreditCardProcessor implements IPaymentProcessor {
  async process(amount: number): Promise<void> {
    console.log(`Processing ${amount} via credit card`);
  }
}

@Injectable()
export class PayPalProcessor implements IPaymentProcessor {
  async process(amount: number): Promise<void> {
    console.log(`Processing ${amount} via PayPal`);
  }
}

// Add new processor without modifying existing code (OCP)
@Injectable()
export class StripeProcessor implements IPaymentProcessor {
  async process(amount: number): Promise<void> {
    console.log(`Processing ${amount} via Stripe`);
  }
}

@Module({
  providers: [
    // Register all processors
    CreditCardProcessor,
    PayPalProcessor,
    StripeProcessor,
    
    // Factory provider selects implementation (DIP + OCP)
    {
      provide: 'IPaymentProcessor',
      useFactory: (config: ConfigService) => {
        const method = config.get('PAYMENT_METHOD');
        
        switch (method) {
          case 'credit-card':
            return new CreditCardProcessor();
          case 'paypal':
            return new PayPalProcessor();
          case 'stripe':
            return new StripeProcessor();
          default:
            return new CreditCardProcessor();
        }
      },
      inject: [ConfigService],
    },
  ],
})
export class PaymentModule {}

// ✅ Environment-based provider selection
@Module({
  providers: [
    TypeOrmUserRepository,
    InMemoryUserRepository,
    {
      provide: 'IUserRepository',
      useFactory: (config: ConfigService) => {
        // Test: in-memory, Production: database
        return config.get('NODE_ENV') === 'test'
          ? new InMemoryUserRepository()
          : new TypeOrmUserRepository();
      },
      inject: [ConfigService],
    },
  ],
})
export class UserModule {}

// ✅ Conditional providers (OCP)
const databaseProviders = [
  {
    provide: 'DATABASE_CONNECTION',
    useFactory: async (config: ConfigService) => {
      const type = config.get('DB_TYPE');
      
      if (type === 'postgres') {
        return createPostgresConnection();
      } else if (type === 'mysql') {
        return createMySQLConnection();
      }
      
      return createSQLiteConnection();
    },
    inject: [ConfigService],
  },
];

// Add new database type without modifying existing code!
```

---

### **Method 5: Guards/Pipes/Interceptors Enforce ISP**

```typescript
// ========== SPECIFIC INTERFACES (ISP) ==========

// NestJS provides many specific interfaces instead of one giant interface

// ✅ CanActivate: Only for guards
@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    // Only need to implement canActivate
    // Don't need to implement transform, intercept, catch, etc.
    return true;
  }
}

// ✅ PipeTransform: Only for pipes
@Injectable()
export class ParseIntPipe implements PipeTransform {
  transform(value: string): number {
    // Only need to implement transform
    // Don't need to implement canActivate, intercept, etc.
    return parseInt(value, 10);
  }
}

// ✅ NestInterceptor: Only for interceptors
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    // Only need to implement intercept
    return next.handle();
  }
}

// ✅ ExceptionFilter: Only for filters
@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    // Only need to implement catch
  }
}

// ❌ If NestJS had one giant interface (violates ISP):
// export interface INestComponent {
//   canActivate?(context: ExecutionContext): boolean;
//   transform?(value: any): any;
//   intercept?(context: ExecutionContext, next: CallHandler): Observable<any>;
//   catch?(exception: any, host: ArgumentsHost): void;
// }
//
// Problem: Every component must know about all methods!

// NestJS ISP Benefits:
// 1. Each component implements only what it needs
// 2. Clear purpose for each interface
// 3. Smaller, focused interfaces
// 4. Easy to understand and implement
```

---

### **SOLID Principles in NestJS:**

| SOLID Principle | NestJS Feature | How It's Enforced |
|-----------------|----------------|-------------------|
| **SRP** | `@Controller`, `@Injectable`, `@Module` | Decorators separate concerns, modules group related code |
| **OCP** | Provider system, Dynamic modules | Add new providers without modifying existing code |
| **LSP** | Interfaces, Abstract classes | All implementations of `IRepository` work interchangeably |
| **ISP** | `CanActivate`, `PipeTransform`, `NestInterceptor` | Many specific interfaces instead of one general |
| **DIP** | Dependency Injection, `@Inject()` | Constructor injection with tokens, depend on interfaces |

---

### **Complete SOLID Example in NestJS:**

```typescript
// ========== COMPLETE EXAMPLE FOLLOWING ALL SOLID PRINCIPLES ==========

// 1. INTERFACE (DIP, ISP)
export interface IUserRepository {
  findById(id: string): Promise<User | null>;
  save(user: User): Promise<User>;
}

// 2. IMPLEMENTATION (DIP, LSP)
@Injectable()
export class TypeOrmUserRepository implements IUserRepository {
  constructor(
    @InjectRepository(UserEntity)
    private repository: Repository<UserEntity>,
  ) {}

  async findById(id: string): Promise<User | null> {
    // Implementation
  }

  async save(user: User): Promise<User> {
    // Implementation
  }
}

// 3. SERVICE (SRP, DIP)
@Injectable()
export class UserService {
  constructor(
    @Inject('IUserRepository')  // DIP: Depend on interface
    private readonly repository: IUserRepository,
  ) {}

  // SRP: Only user business logic
  async createUser(dto: CreateUserDto): Promise<User> {
    const user = new User();
    user.email = dto.email;
    return this.repository.save(user);
  }
}

// 4. CONTROLLER (SRP)
@Controller('users')
export class UsersController {
  constructor(private readonly userService: UserService) {}

  // SRP: Only HTTP handling
  @Post()
  async create(@Body() dto: CreateUserDto): Promise<UserResponseDto> {
    return this.userService.createUser(dto);
  }
}

// 5. GUARD (SRP, ISP)
@Injectable()
export class AuthGuard implements CanActivate {
  // ISP: Specific interface for guards
  // SRP: Only authentication
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    return !!request.user;
  }
}

// 6. MODULE (SRP, OCP)
@Module({
  imports: [TypeOrmModule.forFeature([UserEntity])],
  controllers: [UsersController],  // SRP: User-related HTTP
  providers: [
    UserService,  // SRP: User business logic
    TypeOrmUserRepository,
    {
      provide: 'IUserRepository',
      useClass: TypeOrmUserRepository,  // DIP: Wire interface to implementation
      // OCP: Easy to add new implementation
    },
  ],
  exports: [UserService],
})
export class UsersModule {}  // SRP: One module per feature

// All SOLID principles enforced by NestJS architecture!
```

---

### **Best Practices:**

```typescript
// ✅ GOOD: Follow NestJS conventions (enforces SOLID)
@Controller('users')  // SRP: HTTP only
export class UsersController {
  constructor(private service: UserService) {}  // DIP: Inject service
}

@Injectable()  // SRP: Business logic only
export class UserService {
  constructor(
    @Inject('IUserRepository')  // DIP: Depend on interface
    private repo: IUserRepository,
  ) {}
}

// ❌ BAD: Fight NestJS conventions (violates SOLID)
@Controller('users')
export class UsersController {
  // SRP violation: Business logic in controller
  @Post()
  async create(@Body() dto: CreateUserDto) {
    const user = new User();
    const repo = new TypeOrmUserRepository();  // DIP violation: Direct instantiation
    return repo.save(user);
  }
}

// ✅ GOOD: Use provider system for extensibility (OCP)
@Module({
  providers: [
    {
      provide: 'IUserRepository',
      useClass: TypeOrmUserRepository,  // Easy to swap
    },
  ],
})
export class UserModule {}

// ❌ BAD: Hardcode dependencies (violates OCP)
@Injectable()
export class UserService {
  private repo = new TypeOrmUserRepository();  // Hardcoded!
}

// ✅ GOOD: Use specific interfaces (ISP)
export class AuthGuard implements CanActivate {
  canActivate() { /* only this method */ }
}

// ❌ BAD: Implement unnecessary methods (violates ISP)
export class AuthGuard implements CanActivate, PipeTransform {
  canActivate() { /* needed */ }
  transform() { throw new Error('Not used'); }  // Unnecessary!
}
```

**Key Takeaway:** NestJS **enforces SOLID principles** through its architecture: **Dependency Injection** enforces DIP (constructor injection, `@Inject()`, provider system), **Decorators** enforce SRP (`@Controller` for HTTP, `@Injectable` for services, `@Guard` for auth), **Modular structure** enforces SRP and OCP (one module per feature, easy to add modules), **Provider system** enables OCP (useClass, useFactory, useValue for extensibility), **Specific interfaces** enforce ISP (`CanActivate`, `PipeTransform`, `NestInterceptor` each have one method), and **Interface-based programming** supports LSP and DIP (repository pattern, strategy pattern). Benefits: **maintainable** code (clear separation), **testable** (mock interfaces), **flexible** (swap implementations), **scalable** (add features without breaking existing), and **clean architecture** (dependencies point inward). Follow NestJS conventions, use dependency injection, organize by modules, depend on interfaces, and leverage decorators.

</details>

## Service Layer Patterns

<details>
<summary><strong>25. What should go in a Service vs Controller?</strong></summary>

**Answer:**

**Controllers** handle **HTTP-specific concerns**: routing (`@Get()`, `@Post()`), request parsing (`@Body()`, `@Param()`), response formatting (status codes, headers), DTOs for input/output, guards/pipes/interceptors, and delegating to services. **Services** contain **business logic**: validation rules, calculations, orchestration (calling multiple repositories/services), state management, transactions, and domain operations. **Rule of thumb**: Controllers are **thin** (HTTP adapters), Services are **thick** (business operations). Controllers should never access databases directly or contain complex logic.

---

### **Controller vs Service Responsibilities:**

```
┌─────────────────────────────────────────────────────┐
│              CONTROLLER (HTTP Layer)                │
│  "What comes in, what goes out"                     │
├─────────────────────────────────────────────────────┤
│  ✅ Routing (@Get, @Post, @Put, @Delete)           │
│  ✅ Request parsing (@Body, @Param, @Query)        │
│  ✅ Response formatting (status codes, DTOs)       │
│  ✅ Guards/Pipes/Interceptors                      │
│  ✅ Delegating to services                         │
│  ❌ Business logic                                  │
│  ❌ Database access                                 │
│  ❌ Complex calculations                            │
└─────────────────────────────────────────────────────┘
                      │
                      │ delegates to
                      ▼
┌─────────────────────────────────────────────────────┐
│              SERVICE (Business Layer)               │
│  "How things work"                                  │
├─────────────────────────────────────────────────────┤
│  ✅ Business logic & validation                     │
│  ✅ Calculations & transformations                  │
│  ✅ Orchestration (multiple repositories)          │
│  ✅ Transactions                                     │
│  ✅ Domain operations                               │
│  ✅ State management                                │
│  ❌ HTTP concerns (status codes, headers)           │
│  ❌ Request/response parsing                        │
└─────────────────────────────────────────────────────┘
```

---

### **Method 1: Controller Responsibilities**

```typescript
// ========== CONTROLLER: HTTP CONCERNS ONLY ==========

// ✅ GOOD: Controller handles HTTP, delegates to service
@Controller('users')
export class UsersController {
  constructor(private readonly userService: UserService) {}

  // Responsibility 1: Routing
  @Get()
  @HttpCode(HttpStatus.OK)
  async findAll(): Promise<UserResponseDto[]> {
    const users = await this.userService.findAll();
    return users.map(user => this.toDto(user));  // Response formatting
  }

  // Responsibility 2: Request parsing
  @Get(':id')
  async findOne(@Param('id') id: string): Promise<UserResponseDto> {
    const user = await this.userService.findById(id);
    
    if (!user) {
      throw new NotFoundException('User not found');  // HTTP error
    }
    
    return this.toDto(user);  // Response formatting
  }

  // Responsibility 3: Request validation (DTOs)
  @Post()
  @HttpCode(HttpStatus.CREATED)
  async create(@Body() dto: CreateUserDto): Promise<UserResponseDto> {
    const user = await this.userService.createUser(dto);
    return this.toDto(user);
  }

  // Responsibility 4: Update with partial data
  @Patch(':id')
  async update(
    @Param('id') id: string,
    @Body() dto: UpdateUserDto,
  ): Promise<UserResponseDto> {
    const user = await this.userService.updateUser(id, dto);
    return this.toDto(user);
  }

  // Responsibility 5: Deletion
  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  async remove(@Param('id') id: string): Promise<void> {
    await this.userService.deleteUser(id);
  }

  // Responsibility 6: Guards (authentication/authorization)
  @UseGuards(AuthGuard, RolesGuard)
  @Roles('admin')
  @Get('admin/stats')
  async getAdminStats(): Promise<AdminStatsDto> {
    return this.userService.getAdminStats();
  }

  // Helper: DTO mapping (presentation concern)
  private toDto(user: User): UserResponseDto {
    return {
      id: user.id,
      email: user.email,
      name: user.name,
      createdAt: user.createdAt,
    };
  }
}

// ❌ BAD: Controller with business logic
@Controller('users')
export class UsersController {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>,  // Direct database access!
  ) {}

  @Post()
  async create(@Body() dto: CreateUserDto) {
    // Business logic in controller! Wrong!
    
    // Validation
    if (!dto.email.includes('@')) {
      throw new BadRequestException('Invalid email');
    }

    // Check existence
    const existing = await this.userRepository.findOne({
      where: { email: dto.email },
    });

    if (existing) {
      throw new ConflictException('User exists');
    }

    // Password hashing
    const hashedPassword = await bcrypt.hash(dto.password, 10);

    // Create user
    const user = this.userRepository.create({
      email: dto.email,
      password: hashedPassword,
    });

    // Save
    await this.userRepository.save(user);

    // Send email
    await this.sendWelcomeEmail(user.email);

    return user;
  }

  // Too much responsibility in controller!
}
```

---

### **Method 2: Service Responsibilities**

```typescript
// ========== SERVICE: BUSINESS LOGIC ==========

// ✅ GOOD: Service handles business logic
@Injectable()
export class UserService {
  constructor(
    private readonly userRepository: UserRepository,
    private readonly emailService: EmailService,
    private readonly eventBus: EventEmitter2,
  ) {}

  // Responsibility 1: Business rules & validation
  async createUser(dto: CreateUserDto): Promise<User> {
    // Business rule: Email must be unique
    const existingUser = await this.userRepository.findByEmail(dto.email);
    
    if (existingUser) {
      throw new ConflictException('User with this email already exists');
    }

    // Business rule: Password requirements
    this.validatePassword(dto.password);

    // Create user (domain logic)
    const user = new User();
    user.email = dto.email;
    user.name = dto.name;
    user.password = await this.hashPassword(dto.password);

    // Save
    await this.userRepository.save(user);

    // Side effects (via events)
    this.eventBus.emit('user.created', { userId: user.id, email: user.email });

    return user;
  }

  // Responsibility 2: Complex queries/calculations
  async getUsersByActivity(minOrders: number): Promise<User[]> {
    // Business logic: Get users with minimum order count
    const users = await this.userRepository.findAll();
    
    return users.filter(user => {
      const orderCount = user.orders?.length || 0;
      return orderCount >= minOrders;
    });
  }

  // Responsibility 3: Orchestration (multiple operations)
  async updateUserProfile(id: string, dto: UpdateUserProfileDto): Promise<User> {
    const user = await this.userRepository.findById(id);
    
    if (!user) {
      throw new NotFoundException('User not found');
    }

    // Business rule: Cannot change email to one already in use
    if (dto.email && dto.email !== user.email) {
      const emailTaken = await this.userRepository.findByEmail(dto.email);
      
      if (emailTaken) {
        throw new ConflictException('Email already in use');
      }
    }

    // Update fields
    if (dto.email) user.email = dto.email;
    if (dto.name) user.name = dto.name;

    // Save
    await this.userRepository.save(user);

    // Emit event
    this.eventBus.emit('user.updated', { userId: user.id });

    return user;
  }

  // Responsibility 4: State management
  async activateUser(id: string): Promise<void> {
    const user = await this.userRepository.findById(id);
    
    if (!user) {
      throw new NotFoundException('User not found');
    }

    // Business rule: Cannot activate already active user
    if (user.isActive) {
      throw new BadRequestException('User is already active');
    }

    user.isActive = true;
    await this.userRepository.save(user);

    this.eventBus.emit('user.activated', { userId: user.id });
  }

  // Responsibility 5: Business calculations
  async calculateUserScore(userId: string): Promise<number> {
    const user = await this.userRepository.findById(userId);
    
    if (!user) {
      throw new NotFoundException('User not found');
    }

    // Business logic: Calculate score based on activity
    let score = 0;
    
    // Points for orders
    score += (user.orders?.length || 0) * 10;
    
    // Points for reviews
    score += (user.reviews?.length || 0) * 5;
    
    // Bonus for active user
    if (user.isActive) {
      score += 50;
    }

    return score;
  }

  // Private helper methods (business logic)
  private validatePassword(password: string): void {
    if (password.length < 8) {
      throw new BadRequestException('Password must be at least 8 characters');
    }
    
    if (!/[A-Z]/.test(password)) {
      throw new BadRequestException('Password must contain uppercase letter');
    }
    
    if (!/[0-9]/.test(password)) {
      throw new BadRequestException('Password must contain number');
    }
  }

  private async hashPassword(password: string): Promise<string> {
    return bcrypt.hash(password, 10);
  }

  async findById(id: string): Promise<User | null> {
    return this.userRepository.findById(id);
  }

  async findAll(): Promise<User[]> {
    return this.userRepository.findAll();
  }

  async deleteUser(id: string): Promise<void> {
    const user = await this.userRepository.findById(id);
    
    if (!user) {
      throw new NotFoundException('User not found');
    }

    await this.userRepository.delete(id);
    
    this.eventBus.emit('user.deleted', { userId: id });
  }
}

// ❌ BAD: Service with HTTP concerns
@Injectable()
export class UserService {
  async createUser(req: Request, res: Response) {  // Wrong! HTTP objects in service
    const dto = req.body;
    
    const user = new User();
    user.email = dto.email;
    
    await this.userRepository.save(user);
    
    // Wrong! HTTP response in service
    res.status(201).json({
      success: true,
      data: user,
    });
  }
}
```

---

### **Method 3: Clear Separation Example**

```typescript
// ========== COMPLETE EXAMPLE: CLEAR SEPARATION ==========

// CONTROLLER: HTTP layer
@Controller('orders')
export class OrdersController {
  constructor(private readonly orderService: OrderService) {}

  // HTTP: Routing, parsing, response
  @Post()
  @HttpCode(HttpStatus.CREATED)
  @UseGuards(AuthGuard)
  async placeOrder(
    @Body() dto: PlaceOrderDto,
    @CurrentUser() user: User,
  ): Promise<OrderResponseDto> {
    // Delegate business logic to service
    const order = await this.orderService.placeOrder(user.id, dto);
    
    // Format response (HTTP concern)
    return {
      orderId: order.id,
      total: order.total,
      status: order.status,
      items: order.items.map(item => ({
        productId: item.productId,
        quantity: item.quantity,
        price: item.price,
      })),
    };
  }

  @Get(':id')
  async getOrder(@Param('id') id: string): Promise<OrderResponseDto> {
    const order = await this.orderService.findById(id);
    
    if (!order) {
      throw new NotFoundException('Order not found');  // HTTP error
    }
    
    return this.toDto(order);
  }

  @Patch(':id/cancel')
  @UseGuards(AuthGuard)
  async cancelOrder(
    @Param('id') id: string,
    @CurrentUser() user: User,
  ): Promise<void> {
    await this.orderService.cancelOrder(id, user.id);
  }

  private toDto(order: Order): OrderResponseDto {
    // DTO mapping (presentation concern)
  }
}

// SERVICE: Business logic layer
@Injectable()
export class OrderService {
  constructor(
    private readonly orderRepository: OrderRepository,
    private readonly productService: ProductService,
    private readonly inventoryService: InventoryService,
    private readonly paymentService: PaymentService,
    private readonly eventBus: EventEmitter2,
  ) {}

  // Business logic: Place order
  async placeOrder(customerId: string, dto: PlaceOrderDto): Promise<Order> {
    // Business rule: Validate products exist and are available
    for (const item of dto.items) {
      const product = await this.productService.findById(item.productId);
      
      if (!product) {
        throw new BadRequestException(`Product ${item.productId} not found`);
      }

      if (!product.isAvailable) {
        throw new BadRequestException(`Product ${item.productId} not available`);
      }
    }

    // Business rule: Check inventory
    for (const item of dto.items) {
      const hasStock = await this.inventoryService.checkStock(
        item.productId,
        item.quantity,
      );
      
      if (!hasStock) {
        throw new BadRequestException(`Insufficient stock for ${item.productId}`);
      }
    }

    // Create order (domain logic)
    const order = new Order();
    order.customerId = customerId;
    order.items = dto.items;
    order.total = this.calculateTotal(dto.items);
    order.status = OrderStatus.PENDING;

    // Reserve inventory
    for (const item of dto.items) {
      await this.inventoryService.reserve(item.productId, item.quantity);
    }

    // Process payment
    await this.paymentService.charge(customerId, order.total);

    // Update order status
    order.status = OrderStatus.PLACED;

    // Save
    await this.orderRepository.save(order);

    // Emit event
    this.eventBus.emit('order.placed', { orderId: order.id, customerId });

    return order;
  }

  // Business logic: Cancel order
  async cancelOrder(orderId: string, customerId: string): Promise<void> {
    const order = await this.orderRepository.findById(orderId);
    
    if (!order) {
      throw new NotFoundException('Order not found');
    }

    // Business rule: Only customer can cancel their order
    if (order.customerId !== customerId) {
      throw new ForbiddenException('Cannot cancel order of another customer');
    }

    // Business rule: Cannot cancel shipped order
    if (order.status === OrderStatus.SHIPPED) {
      throw new BadRequestException('Cannot cancel shipped order');
    }

    // Release inventory
    for (const item of order.items) {
      await this.inventoryService.release(item.productId, item.quantity);
    }

    // Refund payment
    await this.paymentService.refund(order.id);

    // Update status
    order.status = OrderStatus.CANCELLED;
    await this.orderRepository.save(order);

    // Emit event
    this.eventBus.emit('order.cancelled', { orderId: order.id });
  }

  async findById(id: string): Promise<Order | null> {
    return this.orderRepository.findById(id);
  }

  // Business calculation
  private calculateTotal(items: OrderItem[]): number {
    return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  }
}
```

---

### **Controller vs Service Comparison:**

| Aspect | Controller | Service |
|--------|------------|---------|
| **Purpose** | HTTP adapter | Business logic |
| **Knows about** | Requests, responses, DTOs | Domain entities, rules |
| **Dependencies** | Services only | Repositories, other services |
| **Testability** | Integration tests (HTTP) | Unit tests (logic) |
| **Complexity** | Thin, simple | Thick, complex |
| **Changes when** | API contract changes | Business rules change |
| **Examples** | Routing, parsing, status codes | Validation, calculations, orchestration |

---

### **Method 4: What NOT to Put Where**

```typescript
// ========== ANTI-PATTERNS ==========

// ❌ DON'T: Business logic in controller
@Controller('users')
export class UsersController {
  @Post()
  async create(@Body() dto: CreateUserDto) {
    // Validation in controller - wrong!
    if (!dto.email.includes('@')) {
      throw new BadRequestException('Invalid email');
    }

    // Database access in controller - wrong!
    const repo = this.userRepository;
    const existing = await repo.findOne({ where: { email: dto.email } });

    if (existing) {
      throw new ConflictException('User exists');
    }

    // Password hashing in controller - wrong!
    const hashed = await bcrypt.hash(dto.password, 10);

    // All of this should be in service!
  }
}

// ❌ DON'T: HTTP concerns in service
@Injectable()
export class UserService {
  async createUser(req: Request, res: Response) {  // Wrong!
    const dto = req.body;  // Wrong! Service shouldn't know about Request
    
    const user = new User();
    await this.userRepository.save(user);
    
    // Wrong! Service shouldn't send HTTP responses
    res.status(201).json({ success: true, data: user });
  }
}

// ❌ DON'T: Direct repository access in controller
@Controller('users')
export class UsersController {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>,  // Wrong! Use service
  ) {}

  @Get()
  async findAll() {
    return this.userRepository.find();  // Wrong! Should go through service
  }
}

// ❌ DON'T: Status codes in service
@Injectable()
export class UserService {
  async createUser(dto: CreateUserDto) {
    const user = await this.userRepository.save(dto);
    
    // Wrong! Service shouldn't know about HTTP status codes
    return {
      statusCode: 201,
      message: 'User created',
      data: user,
    };
  }
}

// ✅ DO: Proper separation
@Controller('users')
export class UsersController {
  constructor(private readonly userService: UserService) {}

  @Post()
  @HttpCode(HttpStatus.CREATED)  // HTTP concern in controller
  async create(@Body() dto: CreateUserDto): Promise<UserResponseDto> {
    const user = await this.userService.createUser(dto);  // Delegate to service
    return this.toDto(user);  // Format response in controller
  }
}

@Injectable()
export class UserService {
  async createUser(dto: CreateUserDto): Promise<User> {
    // Only business logic, no HTTP concerns
    const user = new User();
    // ... business logic ...
    return this.userRepository.save(user);
  }
}
```

---

### **Best Practices:**

```typescript
// ✅ GOOD: Thin controller, thick service
@Controller('products')
export class ProductsController {
  constructor(private service: ProductService) {}

  @Get()
  async findAll(@Query() filters: ProductFiltersDto) {
    return this.service.findAll(filters);
  }
}

@Injectable()
export class ProductService {
  async findAll(filters: ProductFiltersDto): Promise<Product[]> {
    // Complex filtering logic here
  }
}

// ❌ BAD: Thick controller, thin service
@Controller('products')
export class ProductsController {
  @Get()
  async findAll(@Query() filters: ProductFiltersDto) {
    // Complex filtering logic in controller - wrong!
    const products = await this.repository.find();
    return products.filter(p => p.price > filters.minPrice && ...);
  }
}

// ✅ GOOD: Controller delegates everything to service
@Controller('orders')
export class OrdersController {
  constructor(private service: OrderService) {}

  @Post()
  async create(@Body() dto: PlaceOrderDto, @CurrentUser() user: User) {
    return this.service.placeOrder(user.id, dto);  // Service does all work
  }
}

// ❌ BAD: Controller has business logic
@Controller('orders')
export class OrdersController {
  @Post()
  async create(@Body() dto: PlaceOrderDto, @CurrentUser() user: User) {
    // Validation in controller
    if (dto.items.length === 0) {
      throw new BadRequestException('No items');
    }

    // Calculations in controller
    const total = dto.items.reduce((sum, i) => sum + i.price * i.qty, 0);

    // This should all be in service!
  }
}
```

**Key Takeaway:** **Controllers** are **thin HTTP adapters** handling routing (`@Get()`, `@Post()`), request parsing (`@Body()`, `@Param()`), response formatting (DTOs, status codes), guards/pipes, and **delegating to services**. **Services** are **thick business logic** containing validation rules, calculations, orchestration (multiple repositories/services), transactions, domain operations, and state management. **Never** put business logic in controllers, never access repositories directly from controllers, never handle HTTP concerns in services. Controllers should be **3-10 lines per endpoint** (parse, call service, format response). Services contain **all domain knowledge**. Test controllers with integration tests (HTTP), test services with unit tests (logic). Ask: "If I change from REST to GraphQL, what changes?" (only controllers). "If business rules change?" (only services).

</details>

<details>
<summary><strong>26. Should business logic be in Controllers or Services?</strong></summary>

**Answer:**

**Business logic should ALWAYS be in Services**, never in Controllers. Controllers are **HTTP adapters** that translate HTTP requests to service calls and format responses - they should be **stateless**, **thin** (5-10 lines per method), and contain **zero business logic**. Services contain **all domain knowledge**: validation rules, business calculations, state transitions, orchestration, and complex operations. This separation enables **reusability** (same service for REST, GraphQL, CLI), **testability** (test logic without HTTP), **maintainability** (change business rules without touching HTTP layer), and follows **Single Responsibility Principle**.

---

### **Business Logic Location:**

```
┌──────────────────────────────────────────────────┐
│            NEVER in Controller                   │
│  ❌ Validation rules                             │
│  ❌ Business calculations                        │
│  ❌ Database queries                             │
│  ❌ State transitions                            │
│  ❌ Complex conditions                           │
│  ❌ Orchestration logic                          │
└──────────────────────────────────────────────────┘
                    │
                    │ Delegate to
                    ▼
┌──────────────────────────────────────────────────┐
│            ALWAYS in Service                     │
│  ✅ Validation rules (business)                  │
│  ✅ Calculations & transformations               │
│  ✅ State management                             │
│  ✅ Orchestration (multiple operations)         │
│  ✅ Transaction management                       │
│  ✅ Domain operations                            │
└──────────────────────────────────────────────────┘
```

---

### **Method 1: Why Services (Not Controllers)**

```typescript
// ========== WHY BUSINESS LOGIC GOES IN SERVICES ==========

// ❌ BAD: Business logic in controller
@Controller('orders')
export class OrdersController {
  constructor(
    @InjectRepository(Order) private orderRepo: Repository<Order>,
    @InjectRepository(Product) private productRepo: Repository<Product>,
    private emailService: EmailService,
  ) {}

  @Post()
  async placeOrder(@Body() dto: PlaceOrderDto) {
    // ❌ Validation in controller
    if (dto.items.length === 0) {
      throw new BadRequestException('Order must have items');
    }

    // ❌ Database queries in controller
    for (const item of dto.items) {
      const product = await this.productRepo.findOne({
        where: { id: item.productId },
      });

      if (!product) {
        throw new NotFoundException('Product not found');
      }

      // ❌ Business rule in controller
      if (product.stock < item.quantity) {
        throw new BadRequestException('Insufficient stock');
      }
    }

    // ❌ Calculation in controller
    const total = dto.items.reduce((sum, item) => {
      return sum + item.price * item.quantity;
    }, 0);

    // ❌ Business rule in controller
    if (total > 10000) {
      throw new BadRequestException('Order exceeds maximum');
    }

    // ❌ State management in controller
    const order = new Order();
    order.items = dto.items;
    order.total = total;
    order.status = 'pending';

    // ❌ Database save in controller
    await this.orderRepo.save(order);

    // ❌ Side effect in controller
    await this.emailService.sendOrderConfirmation(order);

    return order;
  }
}

// Problems:
// 1. Cannot reuse logic (tied to HTTP)
// 2. Hard to test (need HTTP infrastructure)
// 3. Cannot call from GraphQL, CLI, or queue workers
// 4. Violates Single Responsibility Principle
// 5. Controller is 40+ lines (should be < 10)

// ✅ GOOD: Business logic in service
@Controller('orders')
export class OrdersController {
  constructor(private readonly orderService: OrderService) {}

  @Post()
  @HttpCode(HttpStatus.CREATED)
  async placeOrder(@Body() dto: PlaceOrderDto): Promise<OrderResponseDto> {
    // Only HTTP concerns: parse, delegate, format
    const order = await this.orderService.placeOrder(dto);
    return this.toDto(order);
  }

  private toDto(order: Order): OrderResponseDto {
    return {
      orderId: order.id,
      total: order.total,
      status: order.status,
    };
  }
}

@Injectable()
export class OrderService {
  constructor(
    private readonly orderRepository: OrderRepository,
    private readonly productService: ProductService,
    private readonly eventBus: EventEmitter2,
  ) {}

  // All business logic here!
  async placeOrder(dto: PlaceOrderDto): Promise<Order> {
    // Validation
    this.validateOrderItems(dto.items);

    // Check products and stock
    await this.validateProductsAndStock(dto.items);

    // Calculate total
    const total = this.calculateTotal(dto.items);

    // Business rule: maximum order amount
    if (total > 10000) {
      throw new BusinessRuleException('Order exceeds maximum amount of $10,000');
    }

    // Create order
    const order = Order.create(dto.customerId, dto.items, total);

    // Save
    await this.orderRepository.save(order);

    // Emit event for side effects
    this.eventBus.emit('order.placed', { orderId: order.id });

    return order;
  }

  private validateOrderItems(items: OrderItemDto[]): void {
    if (items.length === 0) {
      throw new BusinessRuleException('Order must contain at least one item');
    }

    if (items.length > 50) {
      throw new BusinessRuleException('Order cannot exceed 50 items');
    }
  }

  private async validateProductsAndStock(items: OrderItemDto[]): Promise<void> {
    for (const item of items) {
      const product = await this.productService.findById(item.productId);

      if (!product) {
        throw new NotFoundException(`Product ${item.productId} not found`);
      }

      if (product.stock < item.quantity) {
        throw new BusinessRuleException(
          `Insufficient stock for product ${product.name}`,
        );
      }
    }
  }

  private calculateTotal(items: OrderItemDto[]): number {
    return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  }
}

// Benefits:
// 1. Can reuse from REST, GraphQL, CLI, queue workers
// 2. Easy to test (no HTTP needed)
// 3. Clear separation of concerns
// 4. Controller is 8 lines (perfect!)
// 5. All business rules in one place
```

---

### **Method 2: Reusability Example**

```typescript
// ========== BUSINESS LOGIC IN SERVICE = REUSABLE EVERYWHERE ==========

// SERVICE: Business logic (reusable)
@Injectable()
export class UserService {
  async createUser(email: string, password: string): Promise<User> {
    // All business logic here
    const existingUser = await this.userRepository.findByEmail(email);
    
    if (existingUser) {
      throw new ConflictException('User already exists');
    }

    this.validatePassword(password);

    const user = new User();
    user.email = email;
    user.password = await this.hashPassword(password);

    return this.userRepository.save(user);
  }

  private validatePassword(password: string): void {
    if (password.length < 8) {
      throw new BadRequestException('Password too short');
    }
    // More validation...
  }

  private async hashPassword(password: string): Promise<string> {
    return bcrypt.hash(password, 10);
  }
}

// USAGE 1: REST API
@Controller('users')
export class UsersController {
  constructor(private userService: UserService) {}

  @Post()
  async create(@Body() dto: CreateUserDto) {
    return this.userService.createUser(dto.email, dto.password);
  }
}

// USAGE 2: GraphQL
@Resolver()
export class UserResolver {
  constructor(private userService: UserService) {}

  @Mutation(() => User)
  async createUser(@Args('input') input: CreateUserInput) {
    return this.userService.createUser(input.email, input.password);
  }
}

// USAGE 3: CLI Command
@Injectable()
export class CreateUserCommand {
  constructor(private userService: UserService) {}

  async run(email: string, password: string): Promise<void> {
    const user = await this.userService.createUser(email, password);
    console.log(`User created: ${user.id}`);
  }
}

// USAGE 4: Queue Worker
@Processor('user-registration')
export class UserRegistrationProcessor {
  constructor(private userService: UserService) {}

  @Process('create-user')
  async handleCreateUser(job: Job<{ email: string; password: string }>) {
    await this.userService.createUser(job.data.email, job.data.password);
  }
}

// USAGE 5: Scheduled Task
@Injectable()
export class UserTasks {
  constructor(private userService: UserService) {}

  @Cron('0 0 * * *')  // Daily
  async createDemoUsers() {
    await this.userService.createUser('demo@example.com', 'password123');
  }
}

// Same business logic, multiple interfaces!
// If logic was in controller, couldn't reuse!
```

---

### **Method 3: Testability Example**

```typescript
// ========== TESTING BUSINESS LOGIC ==========

// ❌ BAD: Testing business logic in controller
describe('OrdersController', () => {
  let controller: OrdersController;
  let app: INestApplication;

  beforeEach(async () => {
    // Must set up entire HTTP infrastructure
    const module = await Test.createTestingModule({
      imports: [
        TypeOrmModule.forRoot({/* database config */}),
        OrdersModule,
      ],
    }).compile();

    app = module.createNestApplication();
    await app.init();
  });

  it('should validate order total', async () => {
    // Must make HTTP request!
    return request(app.getHttpServer())
      .post('/orders')
      .send({ items: [/* ... */], total: 99999 })
      .expect(400)
      .expect(res => {
        expect(res.body.message).toContain('exceeds maximum');
      });
  });

  // Slow, complex, brittle tests
});

// ✅ GOOD: Testing business logic in service
describe('OrderService', () => {
  let service: OrderService;
  let mockRepository: jest.Mocked<OrderRepository>;
  let mockProductService: jest.Mocked<ProductService>;

  beforeEach(() => {
    // Simple mocks, no HTTP, no database
    mockRepository = {
      save: jest.fn(),
      findById: jest.fn(),
    } as any;

    mockProductService = {
      findById: jest.fn(),
    } as any;

    service = new OrderService(mockRepository, mockProductService);
  });

  it('should reject order over $10,000', async () => {
    const dto = {
      customerId: '123',
      items: [{ productId: 'p1', quantity: 1, price: 15000 }],
    };

    await expect(service.placeOrder(dto)).rejects.toThrow(
      'Order exceeds maximum amount',
    );
  });

  it('should calculate total correctly', async () => {
    const dto = {
      customerId: '123',
      items: [
        { productId: 'p1', quantity: 2, price: 100 },
        { productId: 'p2', quantity: 3, price: 50 },
      ],
    };

    mockProductService.findById.mockResolvedValue({ stock: 100 } as any);

    await service.placeOrder(dto);

    expect(mockRepository.save).toHaveBeenCalledWith(
      expect.objectContaining({ total: 350 }),  // 2*100 + 3*50
    );
  });

  it('should reject order with insufficient stock', async () => {
    const dto = {
      customerId: '123',
      items: [{ productId: 'p1', quantity: 10, price: 100 }],
    };

    mockProductService.findById.mockResolvedValue({ stock: 5 } as any);

    await expect(service.placeOrder(dto)).rejects.toThrow('Insufficient stock');
  });

  // Fast, simple, focused tests
});
```

---

### **Method 4: Decision Guide**

```typescript
// ========== WHERE DOES THIS CODE GO? ==========

// 🤔 Ask yourself: "If I change from REST to GraphQL, does this code change?"

// ✅ CHANGES with interface → Controller
// - Routing (@Get, @Post)
// - HTTP status codes (@HttpCode)
// - Request parsing (@Body, @Param)
// - Response formatting (DTOs)
// - Authentication guards (@UseGuards)

// ✅ DOESN'T CHANGE with interface → Service
// - Validation rules (email format)
// - Business calculations (tax, total)
// - Business rules (max order amount)
// - Database operations
// - State transitions
// - Orchestration

// EXAMPLES:

// Controller (HTTP-specific)
@Controller('products')
export class ProductsController {
  @Get(':id')  // ← HTTP routing
  @HttpCode(HttpStatus.OK)  // ← HTTP status
  async findOne(@Param('id') id: string): Promise<ProductDto> {  // ← HTTP parsing
    const product = await this.service.findById(id);  // ← Delegate
    return this.toDto(product);  // ← HTTP formatting
  }
}

// Service (Business logic)
@Injectable()
export class ProductService {
  async findById(id: string): Promise<Product> {
    const product = await this.repository.findById(id);
    
    if (!product) {
      throw new NotFoundException('Product not found');  // ← Business rule
    }

    // Business logic: Apply discount if applicable
    if (product.onSale) {
      product.price = product.price * 0.9;  // ← Business calculation
    }

    return product;
  }
}
```

---

### **Method 5: Real-World Complete Example**

```typescript
// ========== COMPLETE REAL-WORLD EXAMPLE ==========

// CONTROLLER: Thin HTTP adapter (7 lines)
@Controller('users')
export class UsersController {
  constructor(private readonly userService: UserService) {}

  @Post('register')
  @HttpCode(HttpStatus.CREATED)
  async register(@Body() dto: RegisterUserDto): Promise<UserResponseDto> {
    const user = await this.userService.registerUser(dto);
    return { id: user.id, email: user.email, createdAt: user.createdAt };
  }

  @Post('login')
  async login(@Body() dto: LoginDto): Promise<{ token: string }> {
    const token = await this.userService.login(dto.email, dto.password);
    return { token };
  }

  @Get('profile')
  @UseGuards(AuthGuard)
  async getProfile(@CurrentUser() user: User): Promise<UserResponseDto> {
    const profile = await this.userService.getUserProfile(user.id);
    return this.toDto(profile);
  }

  private toDto(user: User): UserResponseDto {
    return { id: user.id, email: user.email, name: user.name };
  }
}

// SERVICE: Thick business logic (60+ lines)
@Injectable()
export class UserService {
  constructor(
    private readonly userRepository: UserRepository,
    private readonly emailService: EmailService,
    private readonly jwtService: JwtService,
    private readonly eventBus: EventEmitter2,
  ) {}

  async registerUser(dto: RegisterUserDto): Promise<User> {
    // Business rule: Email validation
    if (!this.isValidEmail(dto.email)) {
      throw new BadRequestException('Invalid email format');
    }

    // Business rule: Password strength
    this.validatePasswordStrength(dto.password);

    // Business rule: Unique email
    const existingUser = await this.userRepository.findByEmail(dto.email);
    if (existingUser) {
      throw new ConflictException('Email already registered');
    }

    // Business rule: Name requirements
    if (dto.name.length < 2) {
      throw new BadRequestException('Name must be at least 2 characters');
    }

    // Create user
    const user = new User();
    user.email = dto.email.toLowerCase();
    user.name = dto.name;
    user.password = await this.hashPassword(dto.password);
    user.isActive = false;  // Business rule: Require email verification
    user.createdAt = new Date();

    // Save
    await this.userRepository.save(user);

    // Business process: Send verification email
    this.eventBus.emit('user.registered', {
      userId: user.id,
      email: user.email,
    });

    return user;
  }

  async login(email: string, password: string): Promise<string> {
    // Business rule: Find user
    const user = await this.userRepository.findByEmail(email);
    if (!user) {
      throw new UnauthorizedException('Invalid credentials');
    }

    // Business rule: Verify password
    const isPasswordValid = await this.verifyPassword(password, user.password);
    if (!isPasswordValid) {
      throw new UnauthorizedException('Invalid credentials');
    }

    // Business rule: Check if user is active
    if (!user.isActive) {
      throw new UnauthorizedException('Account not activated');
    }

    // Business process: Update last login
    user.lastLoginAt = new Date();
    await this.userRepository.save(user);

    // Business process: Generate token
    const token = this.jwtService.sign({
      sub: user.id,
      email: user.email,
    });

    // Business process: Log activity
    this.eventBus.emit('user.logged-in', { userId: user.id });

    return token;
  }

  async getUserProfile(userId: string): Promise<User> {
    const user = await this.userRepository.findById(userId);
    
    if (!user) {
      throw new NotFoundException('User not found');
    }

    return user;
  }

  // Private business logic methods
  private isValidEmail(email: string): boolean {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }

  private validatePasswordStrength(password: string): void {
    if (password.length < 8) {
      throw new BadRequestException('Password must be at least 8 characters');
    }
    if (!/[A-Z]/.test(password)) {
      throw new BadRequestException('Password must contain uppercase letter');
    }
    if (!/[a-z]/.test(password)) {
      throw new BadRequestException('Password must contain lowercase letter');
    }
    if (!/[0-9]/.test(password)) {
      throw new BadRequestException('Password must contain number');
    }
  }

  private async hashPassword(password: string): Promise<string> {
    return bcrypt.hash(password, 10);
  }

  private async verifyPassword(plain: string, hashed: string): Promise<boolean> {
    return bcrypt.compare(plain, hashed);
  }
}

// Controller: 15 lines (HTTP only)
// Service: 80+ lines (all business logic)
// Perfect separation!
```

---

### **Best Practices:**

```typescript
// ✅ GOOD: Business logic in service
@Injectable()
export class OrderService {
  async placeOrder(dto: PlaceOrderDto): Promise<Order> {
    // All validation, calculation, orchestration here
  }
}

@Controller('orders')
export class OrdersController {
  @Post()
  async create(@Body() dto: PlaceOrderDto) {
    return this.orderService.placeOrder(dto);  // Just delegate
  }
}

// ❌ BAD: Business logic in controller
@Controller('orders')
export class OrdersController {
  @Post()
  async create(@Body() dto: PlaceOrderDto) {
    // Validation, calculation here - wrong!
    if (dto.total > 10000) throw new BadRequestException();
    // ...
  }
}

// ✅ GOOD: Thin controller (< 10 lines per method)
@Controller('users')
export class UsersController {
  @Post()
  async create(@Body() dto: CreateUserDto) {
    const user = await this.service.createUser(dto);
    return { id: user.id, email: user.email };
  }
}

// ❌ BAD: Thick controller (> 20 lines per method)
@Controller('users')
export class UsersController {
  @Post()
  async create(@Body() dto: CreateUserDto) {
    // 50 lines of business logic - wrong!
  }
}
```

**Key Takeaway:** **Business logic ALWAYS in Services, NEVER in Controllers**. Controllers are **thin HTTP adapters** (5-10 lines per method) that parse requests, delegate to services, and format responses. Services contain **all business knowledge**: validation rules, calculations, state transitions, orchestration, and complex operations. Benefits: **reusability** (same service for REST, GraphQL, CLI, queues), **testability** (test logic without HTTP), **maintainability** (change business rules in one place), and **separation of concerns**. Ask: "If I change from REST to GraphQL, does this change?" → YES = Controller, NO = Service. Controllers should only know about HTTP (`@Get`, `@Body`, status codes, DTOs). Services should never know about HTTP (no Request, Response, status codes). Test services with unit tests (fast, simple), test controllers with integration tests (HTTP endpoints).

</details>

<details>
<summary><strong>27. How do you handle complex business logic?</strong></summary>

**Answer:**

Handle complex business logic by **breaking it into smaller services** (Single Responsibility), using **Domain-Driven Design** (Entities, Value Objects, Aggregates), implementing **Strategy Pattern** (swappable algorithms), leveraging **Service Orchestration** (coordinator service calls multiple services), using **Events** (decouple side effects), applying **Transaction Scripts** (simple flows) vs **Domain Models** (complex domains), and creating **Domain Services** (operations spanning multiple aggregates). Complex logic should be **testable**, **maintainable**, **readable** (clear method names), and **reusable**. Avoid god classes (one service doing everything), keep methods small (< 50 lines), use private helper methods, and document business rules.

---

### **Strategies for Complex Business Logic:**

```
┌────────────────────────────────────────────────────┐
│         HANDLING COMPLEX BUSINESS LOGIC            │
├────────────────────────────────────────────────────┤
│                                                    │
│  1. Service Decomposition                         │
│     Break into smaller, focused services          │
│                                                    │
│  2. Domain-Driven Design (DDD)                    │
│     Entities, Value Objects, Aggregates           │
│                                                    │
│  3. Strategy Pattern                               │
│     Swappable algorithms for different scenarios  │
│                                                    │
│  4. Service Orchestration                         │
│     Coordinator service manages workflow          │
│                                                    │
│  5. Event-Driven Architecture                     │
│     Decouple side effects via events              │
│                                                    │
│  6. Domain Services                                │
│     Operations spanning multiple entities         │
│                                                    │
└────────────────────────────────────────────────────┘
```

---

### **Method 1: Service Decomposition (Break Down Complexity)**

```typescript
// ========== BREAK COMPLEX LOGIC INTO SMALLER SERVICES ==========

// ❌ BAD: God service (one service does everything)
@Injectable()
export class OrderService {
  async placeOrder(dto: PlaceOrderDto): Promise<Order> {
    // 300+ lines of complex logic!
    
    // Validate customer
    // Check inventory
    // Calculate pricing
    // Apply discounts
    // Process payment
    // Reserve inventory
    // Create shipment
    // Send notifications
    // Update analytics
    // ...
    
    // Too complex, hard to test, hard to maintain!
  }
}

// ✅ GOOD: Decomposed services (each does one thing)
@Injectable()
export class OrderService {
  constructor(
    private readonly customerService: CustomerService,
    private readonly inventoryService: InventoryService,
    private readonly pricingService: PricingService,
    private readonly paymentService: PaymentService,
    private readonly shipmentService: ShipmentService,
    private readonly orderRepository: OrderRepository,
    private readonly eventBus: EventEmitter2,
  ) {}

  async placeOrder(customerId: string, dto: PlaceOrderDto): Promise<Order> {
    // Step 1: Validate customer
    await this.customerService.validateCustomer(customerId);

    // Step 2: Check inventory availability
    await this.inventoryService.validateAvailability(dto.items);

    // Step 3: Calculate pricing (delegates to pricing service)
    const pricing = await this.pricingService.calculateOrderPricing(
      customerId,
      dto.items,
    );

    // Step 4: Create order entity
    const order = Order.create({
      customerId,
      items: dto.items,
      subtotal: pricing.subtotal,
      discount: pricing.discount,
      tax: pricing.tax,
      total: pricing.total,
    });

    // Step 5: Process payment
    await this.paymentService.charge(customerId, order.total, order.id);

    // Step 6: Reserve inventory
    await this.inventoryService.reserveItems(dto.items, order.id);

    // Step 7: Save order
    await this.orderRepository.save(order);

    // Step 8: Emit event for side effects (shipment, notifications)
    this.eventBus.emit('order.placed', {
      orderId: order.id,
      customerId,
      items: dto.items,
    });

    return order;
  }
}

// Each service is focused and testable
@Injectable()
export class PricingService {
  constructor(
    private readonly discountService: DiscountService,
    private readonly taxService: TaxService,
  ) {}

  async calculateOrderPricing(
    customerId: string,
    items: OrderItem[],
  ): Promise<OrderPricing> {
    // Calculate subtotal
    const subtotal = items.reduce(
      (sum, item) => sum + item.price * item.quantity,
      0,
    );

    // Apply discounts
    const discount = await this.discountService.calculateDiscount(
      customerId,
      items,
      subtotal,
    );

    // Calculate tax
    const tax = await this.taxService.calculateTax(subtotal - discount);

    // Calculate total
    const total = subtotal - discount + tax;

    return { subtotal, discount, tax, total };
  }
}

@Injectable()
export class InventoryService {
  async validateAvailability(items: OrderItem[]): Promise<void> {
    for (const item of items) {
      const stock = await this.getStock(item.productId);
      
      if (stock < item.quantity) {
        throw new BusinessRuleException(
          `Insufficient stock for product ${item.productId}`,
        );
      }
    }
  }

  async reserveItems(items: OrderItem[], orderId: string): Promise<void> {
    // Reserve inventory for order
  }

  private async getStock(productId: string): Promise<number> {
    // Get current stock level
  }
}

// Benefits:
// - Each service < 100 lines
// - Single responsibility
// - Easy to test in isolation
// - Easy to reuse
// - Clear separation of concerns
```

---

### **Method 2: Domain-Driven Design (Rich Domain Models)**

```typescript
// ========== USE DDD FOR COMPLEX DOMAINS ==========

// DOMAIN LAYER: Rich entities with business logic

// Entity: Order Aggregate Root
export class Order {
  private constructor(
    public readonly id: string,
    public readonly customerId: string,
    private _items: OrderItem[],
    private _status: OrderStatus,
    private _total: number,
    public readonly createdAt: Date,
  ) {}

  // Factory method
  static create(dto: CreateOrderDto): Order {
    const order = new Order(
      uuidv4(),
      dto.customerId,
      dto.items.map(item => OrderItem.create(item)),
      OrderStatus.PENDING,
      0,
      new Date(),
    );

    // Business rule: Calculate total on creation
    order.calculateTotal();

    return order;
  }

  // Business logic: Add item
  addItem(item: OrderItem): void {
    // Business rule: Cannot add items to shipped order
    if (this._status === OrderStatus.SHIPPED) {
      throw new DomainException('Cannot add items to shipped order');
    }

    // Business rule: Maximum 50 items per order
    if (this._items.length >= 50) {
      throw new DomainException('Order cannot exceed 50 items');
    }

    this._items.push(item);
    this.calculateTotal();
  }

  // Business logic: Confirm order
  confirm(): void {
    // Business rule: Can only confirm pending order
    if (this._status !== OrderStatus.PENDING) {
      throw new DomainException('Can only confirm pending order');
    }

    // Business rule: Must have at least one item
    if (this._items.length === 0) {
      throw new DomainException('Order must have at least one item');
    }

    this._status = OrderStatus.CONFIRMED;
  }

  // Business logic: Ship order
  ship(trackingNumber: string): void {
    // Business rule: Can only ship confirmed order
    if (this._status !== OrderStatus.CONFIRMED) {
      throw new DomainException('Can only ship confirmed order');
    }

    this._status = OrderStatus.SHIPPED;
    // Domain event
    this.addDomainEvent(new OrderShippedEvent(this.id, trackingNumber));
  }

  // Business logic: Cancel order
  cancel(): void {
    // Business rule: Cannot cancel shipped order
    if (this._status === OrderStatus.SHIPPED) {
      throw new DomainException('Cannot cancel shipped order');
    }

    this._status = OrderStatus.CANCELLED;
  }

  // Business calculation
  private calculateTotal(): void {
    this._total = this._items.reduce(
      (sum, item) => sum + item.getTotal(),
      0,
    );
  }

  // Getters (encapsulation)
  get items(): ReadonlyArray<OrderItem> {
    return [...this._items];
  }

  get status(): OrderStatus {
    return this._status;
  }

  get total(): number {
    return this._total;
  }
}

// Value Object: OrderItem
export class OrderItem {
  private constructor(
    public readonly productId: string,
    public readonly quantity: number,
    public readonly price: number,
  ) {}

  static create(dto: CreateOrderItemDto): OrderItem {
    // Business rules
    if (dto.quantity <= 0) {
      throw new DomainException('Quantity must be positive');
    }

    if (dto.price < 0) {
      throw new DomainException('Price cannot be negative');
    }

    return new OrderItem(dto.productId, dto.quantity, dto.price);
  }

  getTotal(): number {
    return this.quantity * this.price;
  }

  equals(other: OrderItem): boolean {
    return this.productId === other.productId;
  }
}

// APPLICATION SERVICE: Orchestrates domain logic
@Injectable()
export class OrderApplicationService {
  constructor(
    private readonly orderRepository: OrderRepository,
    private readonly eventBus: EventEmitter2,
  ) {}

  async placeOrder(dto: PlaceOrderDto): Promise<Order> {
    // Create rich domain entity (contains business logic)
    const order = Order.create(dto);

    // Confirm order (business rule validation in entity)
    order.confirm();

    // Persist
    await this.orderRepository.save(order);

    // Publish domain events
    this.publishDomainEvents(order);

    return order;
  }

  async shipOrder(orderId: string, trackingNumber: string): Promise<void> {
    const order = await this.orderRepository.findById(orderId);
    
    if (!order) {
      throw new NotFoundException('Order not found');
    }

    // Business logic in domain entity
    order.ship(trackingNumber);

    await this.orderRepository.save(order);
    this.publishDomainEvents(order);
  }

  private publishDomainEvents(order: Order): void {
    const events = order.getDomainEvents();
    events.forEach(event => this.eventBus.emit(event.name, event));
    order.clearDomainEvents();
  }
}

// Benefits:
// - Business logic lives in domain entities
// - Self-documenting (methods read like business rules)
// - Encapsulation (invariants protected)
// - Testable without infrastructure
```

---

### **Method 3: Strategy Pattern (Swappable Algorithms)**

```typescript
// ========== STRATEGY PATTERN FOR COMPLEX LOGIC VARIATIONS ==========

// PROBLEM: Different pricing strategies based on customer type

// Strategy interface
export interface PricingStrategy {
  calculatePrice(basePrice: number, quantity: number): number;
}

// Concrete strategies
@Injectable()
export class RetailPricingStrategy implements PricingStrategy {
  calculatePrice(basePrice: number, quantity: number): number {
    // No discount for retail customers
    return basePrice * quantity;
  }
}

@Injectable()
export class WholesalePricingStrategy implements PricingStrategy {
  calculatePrice(basePrice: number, quantity: number): number {
    // 10% discount for wholesale
    const subtotal = basePrice * quantity;
    return subtotal * 0.9;
  }
}

@Injectable()
export class VipPricingStrategy implements PricingStrategy {
  calculatePrice(basePrice: number, quantity: number): number {
    // Volume-based discount for VIP
    let discount = 0;
    
    if (quantity >= 100) {
      discount = 0.2;  // 20% off for 100+
    } else if (quantity >= 50) {
      discount = 0.15; // 15% off for 50+
    } else if (quantity >= 10) {
      discount = 0.1;  // 10% off for 10+
    }
    
    const subtotal = basePrice * quantity;
    return subtotal * (1 - discount);
  }
}

// Context: Uses strategy
@Injectable()
export class OrderService {
  constructor(
    private readonly strategyFactory: PricingStrategyFactory,
  ) {}

  async calculateOrderTotal(
    customerId: string,
    items: OrderItem[],
  ): Promise<number> {
    // Get customer type
    const customer = await this.getCustomer(customerId);

    // Get appropriate strategy
    const strategy = this.strategyFactory.getStrategy(customer.type);

    // Calculate total using strategy
    let total = 0;
    for (const item of items) {
      const itemTotal = strategy.calculatePrice(item.price, item.quantity);
      total += itemTotal;
    }

    return total;
  }

  private async getCustomer(id: string): Promise<Customer> {
    // Fetch customer
  }
}

// Factory to get strategy
@Injectable()
export class PricingStrategyFactory {
  constructor(
    private readonly retail: RetailPricingStrategy,
    private readonly wholesale: WholesalePricingStrategy,
    private readonly vip: VipPricingStrategy,
  ) {}

  getStrategy(customerType: CustomerType): PricingStrategy {
    switch (customerType) {
      case CustomerType.RETAIL:
        return this.retail;
      case CustomerType.WHOLESALE:
        return this.wholesale;
      case CustomerType.VIP:
        return this.vip;
      default:
        throw new Error(`Unknown customer type: ${customerType}`);
    }
  }
}

// Benefits:
// - Complex pricing logic isolated in strategies
// - Easy to add new customer types (OCP)
// - Each strategy testable independently
// - Business rules clearly separated
```

---

### **Method 4: Service Orchestration (Coordinator Pattern)**

```typescript
// ========== ORCHESTRATION SERVICE FOR COMPLEX WORKFLOWS ==========

// COORDINATOR: Manages multi-step workflow
@Injectable()
export class CheckoutOrchestrator {
  constructor(
    private readonly cartService: CartService,
    private readonly inventoryService: InventoryService,
    private readonly pricingService: PricingService,
    private readonly paymentService: PaymentService,
    private readonly orderService: OrderService,
    private readonly shipmentService: ShipmentService,
    private readonly notificationService: NotificationService,
    private readonly logger: Logger,
  ) {}

  async processCheckout(
    customerId: string,
    paymentMethod: PaymentMethod,
  ): Promise<CheckoutResult> {
    this.logger.log(`Starting checkout for customer ${customerId}`);

    try {
      // Step 1: Get cart
      const cart = await this.cartService.getCart(customerId);
      
      if (cart.items.length === 0) {
        throw new BusinessRuleException('Cart is empty');
      }

      // Step 2: Validate inventory
      await this.inventoryService.validateAvailability(cart.items);

      // Step 3: Calculate final pricing
      const pricing = await this.pricingService.calculateFinalPricing(
        customerId,
        cart.items,
      );

      // Step 4: Create order (idempotent)
      const order = await this.orderService.createOrder({
        customerId,
        items: cart.items,
        pricing,
      });

      // Step 5: Process payment
      const payment = await this.paymentService.charge({
        customerId,
        amount: pricing.total,
        method: paymentMethod,
        orderId: order.id,
      });

      if (!payment.success) {
        // Payment failed - mark order as failed
        await this.orderService.markAsFailed(order.id, payment.errorMessage);
        throw new PaymentFailedException(payment.errorMessage);
      }

      // Step 6: Reserve inventory (after payment succeeds)
      await this.inventoryService.reserveItems(cart.items, order.id);

      // Step 7: Confirm order
      await this.orderService.confirmOrder(order.id);

      // Step 8: Create shipment
      const shipment = await this.shipmentService.createShipment({
        orderId: order.id,
        customerId,
        items: cart.items,
      });

      // Step 9: Clear cart
      await this.cartService.clearCart(customerId);

      // Step 10: Send confirmation (async via event)
      await this.notificationService.sendOrderConfirmation(order.id);

      this.logger.log(`Checkout completed for order ${order.id}`);

      return {
        success: true,
        orderId: order.id,
        total: pricing.total,
        estimatedDelivery: shipment.estimatedDelivery,
      };

    } catch (error) {
      this.logger.error(`Checkout failed for customer ${customerId}`, error);
      
      // Handle rollback if needed
      await this.handleCheckoutFailure(error);
      
      throw error;
    }
  }

  private async handleCheckoutFailure(error: Error): Promise<void> {
    // Implement compensation logic
    // - Refund payment if charged
    // - Release inventory if reserved
    // - Notify customer of failure
  }
}

// Benefits:
// - Complex workflow in one place
// - Clear sequence of operations
// - Easy to understand flow
// - Centralized error handling
// - Can add logging/monitoring
```

---

### **Method 5: Event-Driven Architecture (Decouple Side Effects)**

```typescript
// ========== EVENTS TO HANDLE COMPLEX SIDE EFFECTS ==========

// SERVICE: Emits events instead of handling everything
@Injectable()
export class OrderService {
  constructor(
    private readonly orderRepository: OrderRepository,
    private readonly eventBus: EventEmitter2,
  ) {}

  async placeOrder(dto: PlaceOrderDto): Promise<Order> {
    // Core business logic only
    const order = Order.create(dto);
    await this.orderRepository.save(order);

    // Emit event (decouple side effects)
    this.eventBus.emit('order.placed', {
      orderId: order.id,
      customerId: dto.customerId,
      items: dto.items,
      total: order.total,
    });

    return order;
  }
}

// EVENT HANDLERS: Handle side effects independently
@Injectable()
export class OrderEventHandlers {
  constructor(
    private readonly emailService: EmailService,
    private readonly inventoryService: InventoryService,
    private readonly analyticsService: AnalyticsService,
    private readonly loyaltyService: LoyaltyService,
  ) {}

  @OnEvent('order.placed')
  async handleOrderPlaced(event: OrderPlacedEvent): Promise<void> {
    // Send confirmation email
    await this.emailService.sendOrderConfirmation(
      event.customerId,
      event.orderId,
    );
  }

  @OnEvent('order.placed')
  async updateInventory(event: OrderPlacedEvent): Promise<void> {
    // Update inventory levels
    await this.inventoryService.decrementStock(event.items);
  }

  @OnEvent('order.placed')
  async trackAnalytics(event: OrderPlacedEvent): Promise<void> {
    // Track order in analytics
    await this.analyticsService.trackOrder(event);
  }

  @OnEvent('order.placed')
  async awardLoyaltyPoints(event: OrderPlacedEvent): Promise<void> {
    // Award loyalty points
    const points = Math.floor(event.total / 10);
    await this.loyaltyService.awardPoints(event.customerId, points);
  }
}

// Benefits:
// - Core logic stays simple
// - Side effects decoupled
// - Easy to add new handlers
// - Each handler independently testable
// - Failures in one handler don't affect others
```

---

### **Comparison of Approaches:**

| Approach | Use When | Benefits | Complexity |
|----------|----------|----------|------------|
| **Service Decomposition** | Logic spans multiple concerns | Clear separation, reusable | Low-Medium |
| **Domain-Driven Design** | Rich domain with complex rules | Business logic in entities | High |
| **Strategy Pattern** | Multiple algorithms/variations | Swappable, extensible | Medium |
| **Service Orchestration** | Multi-step workflows | Clear flow, centralized | Medium |
| **Event-Driven** | Many side effects/integrations | Decoupled, scalable | Medium-High |

---

### **Best Practices:**

```typescript
// ✅ GOOD: Break down complex logic
@Injectable()
export class OrderService {
  async placeOrder(dto: PlaceOrderDto): Promise<Order> {
    await this.validateOrder(dto);
    await this.checkInventory(dto.items);
    const pricing = await this.calculatePricing(dto);
    const order = await this.createOrder(dto, pricing);
    return order;
  }

  private async validateOrder(dto: PlaceOrderDto): Promise<void> {
    // Validation logic
  }

  private async checkInventory(items: OrderItem[]): Promise<void> {
    // Inventory logic
  }

  private async calculatePricing(dto: PlaceOrderDto): Promise<Pricing> {
    // Pricing logic
  }

  private async createOrder(dto: PlaceOrderDto, pricing: Pricing): Promise<Order> {
    // Creation logic
  }
}

// ❌ BAD: One giant method
@Injectable()
export class OrderService {
  async placeOrder(dto: PlaceOrderDto): Promise<Order> {
    // 200 lines of complex logic in one method!
  }
}

// ✅ GOOD: Extract to separate service
@Injectable()
export class PricingService {
  calculatePrice(items: OrderItem[]): number {
    // Complex pricing logic isolated
  }
}

// ❌ BAD: Mix concerns
@Injectable()
export class OrderService {
  async placeOrder(dto: PlaceOrderDto): Promise<Order> {
    // Validate
    // Calculate price
    // Process payment
    // Send email
    // Update analytics
    // All in one service!
  }
}
```

**Key Takeaway:** Handle complex business logic by **decomposing into smaller services** (each < 100 lines, single responsibility), using **Domain-Driven Design** for rich domains (entities contain business rules, value objects for immutability, aggregates for consistency boundaries), applying **Strategy Pattern** for algorithm variations (swappable pricing, payment, notification strategies), leveraging **Service Orchestration** for multi-step workflows (coordinator service manages sequence), using **Events** to decouple side effects (emit events, separate handlers), and creating **Domain Services** for operations spanning multiple entities. Keep methods small (< 50 lines), use descriptive names (`validateCustomerEligibility`, `calculateOrderTotal`), extract private helper methods, document complex business rules with comments, prefer composition over inheritance, test each service independently, and avoid god classes (one service doing everything). Complex logic should be **readable** (clear intent), **testable** (isolated units), **maintainable** (easy to change), and **reusable** (shared across features).

</details>

<details>
<summary><strong>28. What is the difference between Service and Repository?</strong></summary>

**Answer:**

**Services** contain **business logic** (validation, calculations, orchestration, state transitions), **orchestrate multiple repositories/services**, implement **use cases** (user stories), and **know about business rules**. **Repositories** provide **data access abstraction** (CRUD operations), **hide database details** (ORM, SQL), operate on **single entity/aggregate**, and **know nothing about business logic**. **Rule**: Services call repositories, repositories never call services. Services are **thick** (complex business logic), repositories are **thin** (simple data operations). Services are **application layer**, repositories are **infrastructure layer**. Never put business logic in repositories (validation, calculations, orchestration). Repositories should only have methods like `find`, `save`, `delete`, `findByX`.

---

### **Service vs Repository:**

```
┌─────────────────────────────────────────────────┐
│            SERVICE (Application Layer)          │
│  "What to do" - Business Logic                  │
├─────────────────────────────────────────────────┤
│  ✅ Business validation & rules                 │
│  ✅ Complex calculations                        │
│  ✅ Orchestration (multiple repos/services)    │
│  ✅ Transaction management                      │
│  ✅ State transitions                           │
│  ✅ Use case implementation                     │
│  ❌ SQL queries / ORM operations                │
│  ❌ Database-specific code                      │
└─────────────────────────────────────────────────┘
                    │
                    │ uses
                    ▼
┌─────────────────────────────────────────────────┐
│         REPOSITORY (Infrastructure Layer)       │
│  "How to persist" - Data Access                 │
├─────────────────────────────────────────────────┤
│  ✅ CRUD operations (save, find, delete)        │
│  ✅ Database queries (TypeORM, SQL)             │
│  ✅ Data mapping (entity ↔ database)           │
│  ✅ Single entity/aggregate operations          │
│  ❌ Business validation                          │
│  ❌ Business calculations                        │
│  ❌ Orchestration                                │
│  ❌ Calling other services                       │
└─────────────────────────────────────────────────┘
```

---

### **Method 1: Clear Separation Example**

```typescript
// ========== SERVICE VS REPOSITORY RESPONSIBILITIES ==========

// REPOSITORY: Data access only (thin)
@Injectable()
export class UserRepository {
  constructor(
    @InjectRepository(User)
    private readonly repo: Repository<User>,
  ) {}

  // Simple CRUD operations only
  async save(user: User): Promise<User> {
    return this.repo.save(user);
  }

  async findById(id: string): Promise<User | null> {
    return this.repo.findOne({ where: { id } });
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.repo.findOne({ where: { email } });
  }

  async findAll(): Promise<User[]> {
    return this.repo.find();
  }

  async delete(id: string): Promise<void> {
    await this.repo.delete(id);
  }

  // Data access query (no business logic)
  async findActiveUsers(): Promise<User[]> {
    return this.repo.find({ where: { isActive: true } });
  }

  // Complex query (still just data access)
  async findUsersWithOrders(minOrders: number): Promise<User[]> {
    return this.repo
      .createQueryBuilder('user')
      .leftJoinAndSelect('user.orders', 'order')
      .having('COUNT(order.id) >= :minOrders', { minOrders })
      .groupBy('user.id')
      .getMany();
  }

  // ❌ NEVER in repository: Business logic!
  // async createUser(dto: CreateUserDto): Promise<User> {
  //   // Validation - NO!
  //   if (!dto.email.includes('@')) throw new Error();
  //   
  //   // Password hashing - NO!
  //   dto.password = await bcrypt.hash(dto.password, 10);
  //   
  //   // Side effects - NO!
  //   await this.emailService.send();
  //   
  //   // This belongs in SERVICE!
  // }
}

// SERVICE: Business logic (thick)
@Injectable()
export class UserService {
  constructor(
    private readonly userRepository: UserRepository,
    private readonly emailService: EmailService,
    private readonly eventBus: EventEmitter2,
  ) {}

  // Use case: Create user (business logic)
  async createUser(dto: CreateUserDto): Promise<User> {
    // Business validation
    this.validateEmail(dto.email);
    this.validatePassword(dto.password);

    // Business rule: Unique email
    const existingUser = await this.userRepository.findByEmail(dto.email);
    if (existingUser) {
      throw new ConflictException('Email already exists');
    }

    // Business logic: Hash password
    const hashedPassword = await this.hashPassword(dto.password);

    // Create entity
    const user = new User();
    user.email = dto.email;
    user.name = dto.name;
    user.password = hashedPassword;
    user.isActive = false;  // Business rule: require verification

    // Persist (delegate to repository)
    await this.userRepository.save(user);

    // Business process: Send verification email
    this.eventBus.emit('user.created', { userId: user.id, email: user.email });

    return user;
  }

  // Use case: Activate user
  async activateUser(userId: string): Promise<void> {
    // Load entity
    const user = await this.userRepository.findById(userId);
    
    if (!user) {
      throw new NotFoundException('User not found');
    }

    // Business rule: Cannot activate already active user
    if (user.isActive) {
      throw new BadRequestException('User already active');
    }

    // State transition
    user.isActive = true;
    user.activatedAt = new Date();

    // Persist
    await this.userRepository.save(user);

    // Side effect
    this.eventBus.emit('user.activated', { userId: user.id });
  }

  // Orchestration: Multiple operations
  async getUserWithStats(userId: string): Promise<UserWithStatsDto> {
    // Get user from repository
    const user = await this.userRepository.findById(userId);
    
    if (!user) {
      throw new NotFoundException('User not found');
    }

    // Calculate business metrics (not in repository!)
    const stats = await this.calculateUserStats(user);

    // Combine data
    return {
      id: user.id,
      email: user.email,
      name: user.name,
      stats,
    };
  }

  // Private business logic
  private validateEmail(email: string): void {
    if (!email.includes('@')) {
      throw new BadRequestException('Invalid email');
    }
  }

  private validatePassword(password: string): void {
    if (password.length < 8) {
      throw new BadRequestException('Password too short');
    }
  }

  private async hashPassword(password: string): Promise<string> {
    return bcrypt.hash(password, 10);
  }

  private async calculateUserStats(user: User): Promise<UserStats> {
    // Business calculation
    return {
      orderCount: user.orders?.length || 0,
      totalSpent: user.orders?.reduce((sum, o) => sum + o.total, 0) || 0,
    };
  }
}

// Summary:
// - Repository: 8 methods, all data access, ~60 lines
// - Service: 4 use cases + helpers, all business logic, ~120 lines
```

---

### **Method 2: What Goes Where**

```typescript
// ========== DECISION GUIDE: SERVICE VS REPOSITORY ==========

// REPOSITORY: Only data operations

@Injectable()
export class OrderRepository {
  // ✅ GOOD: Simple CRUD
  async save(order: Order): Promise<Order> {
    return this.repo.save(order);
  }

  async findById(id: string): Promise<Order | null> {
    return this.repo.findOne({ where: { id }, relations: ['items'] });
  }

  // ✅ GOOD: Data filtering/queries
  async findByCustomer(customerId: string): Promise<Order[]> {
    return this.repo.find({ where: { customerId } });
  }

  // ✅ GOOD: Complex query (still just data access)
  async findOrdersWithStatus(
    customerId: string,
    status: OrderStatus,
  ): Promise<Order[]> {
    return this.repo
      .createQueryBuilder('order')
      .where('order.customerId = :customerId', { customerId })
      .andWhere('order.status = :status', { status })
      .leftJoinAndSelect('order.items', 'items')
      .getMany();
  }

  // ✅ GOOD: Batch operations
  async saveMany(orders: Order[]): Promise<Order[]> {
    return this.repo.save(orders);
  }

  // ❌ BAD: Business validation in repository
  async createOrder(dto: CreateOrderDto): Promise<Order> {
    // Validation - belongs in service!
    if (dto.items.length === 0) {
      throw new BadRequestException('Order must have items');
    }

    // Calculation - belongs in service!
    const total = dto.items.reduce((sum, i) => sum + i.price * i.quantity, 0);

    // This is business logic, not data access!
  }

  // ❌ BAD: Orchestration in repository
  async placeOrder(order: Order): Promise<Order> {
    // Calling other services - belongs in service!
    await this.inventoryService.reserve(order.items);
    await this.paymentService.charge(order.total);
    return this.repo.save(order);
  }

  // ❌ BAD: State transitions in repository
  async cancelOrder(orderId: string): Promise<void> {
    const order = await this.findById(orderId);
    
    // Business rule - belongs in service!
    if (order.status === OrderStatus.SHIPPED) {
      throw new BadRequestException('Cannot cancel shipped order');
    }
    
    order.status = OrderStatus.CANCELLED;
    await this.repo.save(order);
  }
}

// SERVICE: Business logic and orchestration

@Injectable()
export class OrderService {
  constructor(
    private readonly orderRepository: OrderRepository,
    private readonly inventoryService: InventoryService,
    private readonly paymentService: PaymentService,
  ) {}

  // ✅ GOOD: Business logic in service
  async createOrder(dto: CreateOrderDto): Promise<Order> {
    // Validation (business rule)
    this.validateOrderItems(dto.items);

    // Calculation (business logic)
    const total = this.calculateTotal(dto.items);

    // Create entity
    const order = new Order();
    order.customerId = dto.customerId;
    order.items = dto.items;
    order.total = total;
    order.status = OrderStatus.PENDING;

    // Persist (delegate to repository)
    return this.orderRepository.save(order);
  }

  // ✅ GOOD: Orchestration in service
  async placeOrder(orderId: string): Promise<Order> {
    // Load entity
    const order = await this.orderRepository.findById(orderId);
    
    if (!order) {
      throw new NotFoundException('Order not found');
    }

    // Orchestrate multiple operations
    await this.inventoryService.reserveItems(order.items);
    await this.paymentService.charge(order.customerId, order.total);

    // Update state
    order.status = OrderStatus.PLACED;

    // Persist
    return this.orderRepository.save(order);
  }

  // ✅ GOOD: State transition in service
  async cancelOrder(orderId: string): Promise<void> {
    const order = await this.orderRepository.findById(orderId);
    
    if (!order) {
      throw new NotFoundException('Order not found');
    }

    // Business rule
    if (order.status === OrderStatus.SHIPPED) {
      throw new BadRequestException('Cannot cancel shipped order');
    }

    // State transition
    order.status = OrderStatus.CANCELLED;

    // Orchestrate side effects
    await this.inventoryService.releaseItems(order.items);
    await this.paymentService.refund(order.id);

    // Persist
    await this.orderRepository.save(order);
  }

  // Private business logic
  private validateOrderItems(items: OrderItem[]): void {
    if (items.length === 0) {
      throw new BadRequestException('Order must have items');
    }
  }

  private calculateTotal(items: OrderItem[]): number {
    return items.reduce((sum, i) => sum + i.price * i.quantity, 0);
  }
}
```

---

### **Method 3: Repository Pattern Example**

```typescript
// ========== REPOSITORY PATTERN: ABSTRACT DATA ACCESS ==========

// Interface: Repository contract
export interface IUserRepository {
  save(user: User): Promise<User>;
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  findAll(): Promise<User[]>;
  delete(id: string): Promise<void>;
}

// Implementation: TypeORM repository
@Injectable()
export class TypeOrmUserRepository implements IUserRepository {
  constructor(
    @InjectRepository(User)
    private readonly repo: Repository<User>,
  ) {}

  async save(user: User): Promise<User> {
    return this.repo.save(user);
  }

  async findById(id: string): Promise<User | null> {
    return this.repo.findOne({ where: { id } });
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.repo.findOne({ where: { email } });
  }

  async findAll(): Promise<User[]> {
    return this.repo.find();
  }

  async delete(id: string): Promise<void> {
    await this.repo.delete(id);
  }
}

// Alternative implementation: In-memory (for testing)
export class InMemoryUserRepository implements IUserRepository {
  private users: Map<string, User> = new Map();

  async save(user: User): Promise<User> {
    this.users.set(user.id, user);
    return user;
  }

  async findById(id: string): Promise<User | null> {
    return this.users.get(id) || null;
  }

  async findByEmail(email: string): Promise<User | null> {
    return Array.from(this.users.values()).find(u => u.email === email) || null;
  }

  async findAll(): Promise<User[]> {
    return Array.from(this.users.values());
  }

  async delete(id: string): Promise<void> {
    this.users.delete(id);
  }
}

// Service: Depends on interface, not implementation
@Injectable()
export class UserService {
  constructor(
    @Inject('IUserRepository')
    private readonly userRepository: IUserRepository,  // Interface!
  ) {}

  async createUser(dto: CreateUserDto): Promise<User> {
    // Business logic...
    const user = new User();
    // ...
    
    // Repository implementation hidden
    return this.userRepository.save(user);
  }
}

// Module: Provide implementation
@Module({
  providers: [
    UserService,
    {
      provide: 'IUserRepository',
      useClass: TypeOrmUserRepository,  // Can swap to InMemoryUserRepository for tests
    },
  ],
})
export class UserModule {}

// Benefits:
// - Service doesn't know about TypeORM
// - Can swap implementations (in-memory for tests, different DB)
// - Repository focuses only on data access
```

---

### **Method 4: Anti-Patterns to Avoid**

```typescript
// ========== COMMON MISTAKES ==========

// ❌ ANTI-PATTERN 1: Business logic in repository
@Injectable()
export class UserRepository {
  async registerUser(dto: CreateUserDto): Promise<User> {
    // Validation - NO!
    if (!dto.email.includes('@')) {
      throw new BadRequestException('Invalid email');
    }

    // Business rule - NO!
    const existing = await this.repo.findOne({ where: { email: dto.email } });
    if (existing) {
      throw new ConflictException('Email exists');
    }

    // Password hashing - NO!
    const hashed = await bcrypt.hash(dto.password, 10);

    // Send email - NO!
    await this.emailService.sendWelcome(dto.email);

    // All of this belongs in SERVICE!
  }
}

// ❌ ANTI-PATTERN 2: Repository calling other services
@Injectable()
export class OrderRepository {
  constructor(
    private readonly paymentService: PaymentService,  // NO!
    private readonly inventoryService: InventoryService,  // NO!
  ) {}

  async createOrder(order: Order): Promise<Order> {
    // Orchestration - belongs in service!
    await this.inventoryService.reserve(order.items);
    await this.paymentService.charge(order.total);
    return this.repo.save(order);
  }
}

// ❌ ANTI-PATTERN 3: Service with database queries
@Injectable()
export class UserService {
  async findActiveUsers(): Promise<User[]> {
    // Direct database query - should be in repository!
    return this.entityManager
      .createQueryBuilder(User, 'user')
      .where('user.isActive = :active', { active: true })
      .getMany();
  }
}

// ❌ ANTI-PATTERN 4: Repository returning DTOs
@Injectable()
export class UserRepository {
  async findUserProfile(id: string): Promise<UserProfileDto> {
    const user = await this.repo.findOne({ where: { id } });
    
    // DTO transformation - belongs in service or controller!
    return {
      name: user.name,
      email: user.email,
      // ...
    };
  }
}

// ✅ CORRECT: Clear separation
@Injectable()
export class UserRepository {
  // Only data access
  async findById(id: string): Promise<User | null> {
    return this.repo.findOne({ where: { id } });
  }

  async findActiveUsers(): Promise<User[]> {
    return this.repo.find({ where: { isActive: true } });
  }
}

@Injectable()
export class UserService {
  // All business logic
  async registerUser(dto: CreateUserDto): Promise<User> {
    this.validateEmail(dto.email);
    const existing = await this.userRepository.findByEmail(dto.email);
    if (existing) throw new ConflictException();
    // ...
    return this.userRepository.save(user);
  }

  async getUserProfile(id: string): Promise<UserProfileDto> {
    const user = await this.userRepository.findById(id);
    return this.toProfileDto(user);  // DTO mapping in service
  }
}
```

---

### **Service vs Repository Comparison:**

| Aspect | Service | Repository |
|--------|---------|------------|
| **Purpose** | Business logic | Data access |
| **Layer** | Application layer | Infrastructure layer |
| **Knows about** | Business rules, use cases | Database, ORM, queries |
| **Methods** | `createUser`, `placeOrder`, `calculateTotal` | `save`, `findById`, `delete` |
| **Dependencies** | Multiple repositories, other services | Only database/ORM |
| **Complexity** | Thick (complex logic) | Thin (simple CRUD) |
| **Calls** | Calls repositories | Never calls services |
| **Testing** | Unit tests with mocked repos | Integration tests with database |
| **Changes when** | Business rules change | Database schema changes |

---

### **Best Practices:**

```typescript
// ✅ GOOD: Thin repository
@Injectable()
export class ProductRepository {
  async save(product: Product): Promise<Product> {
    return this.repo.save(product);
  }

  async findById(id: string): Promise<Product | null> {
    return this.repo.findOne({ where: { id } });
  }

  async findByCategory(category: string): Promise<Product[]> {
    return this.repo.find({ where: { category } });
  }
}

// ✅ GOOD: Thick service
@Injectable()
export class ProductService {
  async createProduct(dto: CreateProductDto): Promise<Product> {
    this.validateProduct(dto);
    const product = new Product();
    // Business logic...
    return this.productRepository.save(product);
  }

  async updatePrice(id: string, newPrice: number): Promise<Product> {
    const product = await this.productRepository.findById(id);
    // Business validation...
    product.price = newPrice;
    return this.productRepository.save(product);
  }

  private validateProduct(dto: CreateProductDto): void {
    // Validation logic
  }
}

// ✅ GOOD: Repository methods naming
// - save, findById, findAll, delete
// - findByEmail, findByStatus, findActiveUsers
// - NOT: createUser, updateUserProfile, validateOrder

// ✅ GOOD: Service methods naming
// - createUser, updateUserProfile, activateUser
// - placeOrder, cancelOrder, calculateTotal
// - NOT: save, findById (these are repository methods)
```

**Key Takeaway:** **Services** contain **business logic** (validation, calculations, orchestration, state transitions, use cases) and **call multiple repositories/services**. **Repositories** provide **data access abstraction** (CRUD: save, find, delete), **hide database implementation** (TypeORM, SQL), operate on **single entity/aggregate**, and **contain zero business logic**. Services are **application layer** (use cases), repositories are **infrastructure layer** (persistence). **Rule**: Services call repositories, repositories NEVER call services or other repositories. Repository methods: `save`, `findById`, `findByEmail`, `delete` (data operations only). Service methods: `createUser`, `placeOrder`, `activateUser`, `calculateTotal` (business operations). Repositories should be **thin** (simple CRUD, ~50 lines), services should be **thick** (complex logic, ~100-200 lines). Never put business validation, calculations, or orchestration in repositories. Test repositories with integration tests (real database), test services with unit tests (mocked repositories).

</details>

## Error Handling Patterns

<details>
<summary><strong>29. How should you structure error handling in large applications?</strong></summary>

**Answer:**

Structure error handling with **custom exception hierarchies** (domain-specific errors extending base exceptions), **exception filters** (global and specific), **error codes/enums** (machine-readable identifiers), **consistent error responses** (standard format), **logging strategy** (context, correlation IDs), **graceful degradation** (fallbacks), and **domain exceptions** (business rule violations). Use **global exception filter** for common errors, **specific filters** for domain errors, **throw exceptions early** (fail fast), **handle at appropriate layer** (business logic in services, HTTP errors in filters), and **never swallow errors** silently. Provide **actionable error messages** for clients, **detailed logs** for developers.

---

### **Error Handling Architecture:**

```
┌─────────────────────────────────────────────────────┐
│              ERROR HANDLING LAYERS                  │
├─────────────────────────────────────────────────────┤
│                                                     │
│  1. Domain Exceptions (Business Rules)             │
│     - InvalidOrderException                        │
│     - InsufficientFundsException                   │
│     - BusinessRuleViolationException               │
│                                                     │
│  2. Application Exceptions (Use Cases)             │
│     - NotFoundException                            │
│     - UnauthorizedException                        │
│     - ValidationException                          │
│                                                     │
│  3. Infrastructure Exceptions (Technical)          │
│     - DatabaseConnectionException                  │
│     - ExternalServiceException                     │
│     - TimeoutException                             │
│                                                     │
│  4. Exception Filters (Transform to HTTP)          │
│     - Global exception filter (catch all)          │
│     - Domain exception filter (business errors)    │
│     - HTTP exception filter (NestJS built-in)      │
│                                                     │
│  5. Error Response Format (Consistent)             │
│     - status, message, code, timestamp, path       │
│     - Optional: details, stackTrace (dev only)     │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

### **Method 1: Custom Exception Hierarchy**

```typescript
// ========== CUSTOM EXCEPTION HIERARCHY ==========

// BASE: Domain exception
export class DomainException extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly statusCode: number = 400,
  ) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

// BUSINESS RULE EXCEPTIONS
export class BusinessRuleViolationException extends DomainException {
  constructor(message: string, code: string) {
    super(message, code, 400);
  }
}

export class InvalidOrderException extends BusinessRuleViolationException {
  constructor(reason: string) {
    super(`Invalid order: ${reason}`, 'ORDER_INVALID');
  }
}

export class InsufficientStockException extends BusinessRuleViolationException {
  constructor(productId: string, requested: number, available: number) {
    super(
      `Insufficient stock for product ${productId}. Requested: ${requested}, Available: ${available}`,
      'INSUFFICIENT_STOCK',
    );
  }
}

export class OrderCancellationException extends BusinessRuleViolationException {
  constructor(reason: string) {
    super(`Cannot cancel order: ${reason}`, 'ORDER_CANCELLATION_FAILED');
  }
}

// APPLICATION EXCEPTIONS
export class NotFoundException extends DomainException {
  constructor(entity: string, id: string) {
    super(`${entity} with id ${id} not found`, 'NOT_FOUND', 404);
  }
}

export class UnauthorizedException extends DomainException {
  constructor(message: string = 'Unauthorized') {
    super(message, 'UNAUTHORIZED', 401);
  }
}

export class ForbiddenException extends DomainException {
  constructor(message: string = 'Forbidden') {
    super(message, 'FORBIDDEN', 403);
  }
}

// INFRASTRUCTURE EXCEPTIONS
export class ExternalServiceException extends DomainException {
  constructor(
    public readonly serviceName: string,
    message: string,
  ) {
    super(`External service ${serviceName} error: ${message}`, 'EXTERNAL_SERVICE_ERROR', 502);
  }
}

export class DatabaseException extends DomainException {
  constructor(message: string) {
    super(`Database error: ${message}`, 'DATABASE_ERROR', 500);
  }
}

// USAGE IN SERVICE
@Injectable()
export class OrderService {
  async placeOrder(dto: PlaceOrderDto): Promise<Order> {
    // Throw domain-specific exceptions
    if (dto.items.length === 0) {
      throw new InvalidOrderException('Order must contain at least one item');
    }

    const product = await this.productService.findById(dto.items[0].productId);
    
    if (!product) {
      throw new NotFoundException('Product', dto.items[0].productId);
    }

    if (product.stock < dto.items[0].quantity) {
      throw new InsufficientStockException(
        product.id,
        dto.items[0].quantity,
        product.stock,
      );
    }

    // Business logic...
  }

  async cancelOrder(orderId: string): Promise<void> {
    const order = await this.orderRepository.findById(orderId);
    
    if (!order) {
      throw new NotFoundException('Order', orderId);
    }

    if (order.status === OrderStatus.SHIPPED) {
      throw new OrderCancellationException('order has already been shipped');
    }

    if (order.status === OrderStatus.CANCELLED) {
      throw new OrderCancellationException('order is already cancelled');
    }

    // Cancel logic...
  }
}
```

---

### **Method 2: Exception Filters (Transform Exceptions to HTTP)**

```typescript
// ========== EXCEPTION FILTERS ==========

// GLOBAL EXCEPTION FILTER: Catches all exceptions
@Catch()
export class GlobalExceptionFilter implements ExceptionFilter {
  constructor(private readonly logger: Logger) {}

  catch(exception: unknown, host: ArgumentsHost): void {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    let status = HttpStatus.INTERNAL_SERVER_ERROR;
    let message = 'Internal server error';
    let code = 'INTERNAL_ERROR';
    let details: any;

    // Handle different exception types
    if (exception instanceof DomainException) {
      // Domain exceptions (business rules)
      status = exception.statusCode;
      message = exception.message;
      code = exception.code;
    } else if (exception instanceof HttpException) {
      // NestJS HTTP exceptions
      status = exception.getStatus();
      const exceptionResponse = exception.getResponse();
      
      if (typeof exceptionResponse === 'object') {
        message = (exceptionResponse as any).message || exception.message;
        code = (exceptionResponse as any).error || exception.name;
        details = (exceptionResponse as any).details;
      } else {
        message = exceptionResponse as string;
      }
    } else if (exception instanceof Error) {
      // Generic errors
      message = exception.message;
      code = exception.name;
    }

    // Build error response
    const errorResponse = {
      statusCode: status,
      message,
      code,
      timestamp: new Date().toISOString(),
      path: request.url,
      method: request.method,
      ...(details && { details }),
      // Include stack trace in development only
      ...(process.env.NODE_ENV === 'development' && {
        stack: exception instanceof Error ? exception.stack : undefined,
      }),
    };

    // Log error
    this.logger.error(
      `${request.method} ${request.url} - ${status} ${code}: ${message}`,
      exception instanceof Error ? exception.stack : undefined,
    );

    // Send response
    response.status(status).json(errorResponse);
  }
}

// DOMAIN EXCEPTION FILTER: Handles business rule violations
@Catch(DomainException)
export class DomainExceptionFilter implements ExceptionFilter {
  constructor(private readonly logger: Logger) {}

  catch(exception: DomainException, host: ArgumentsHost): void {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    // Log business rule violations at warn level (not errors)
    this.logger.warn(
      `Business rule violation: ${exception.code} - ${exception.message}`,
      { path: request.url, method: request.method },
    );

    // Send consistent error response
    response.status(exception.statusCode).json({
      statusCode: exception.statusCode,
      message: exception.message,
      code: exception.code,
      timestamp: new Date().toISOString(),
      path: request.url,
    });
  }
}

// HTTP EXCEPTION FILTER: Handles NestJS HTTP exceptions
@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost): void {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const status = exception.getStatus();

    const errorResponse = {
      statusCode: status,
      message: exception.message,
      timestamp: new Date().toISOString(),
      path: request.url,
    };

    response.status(status).json(errorResponse);
  }
}

// REGISTER IN MAIN.TS
async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Register global exception filter
  app.useGlobalFilters(
    new GlobalExceptionFilter(new Logger('ExceptionFilter')),
  );

  await app.listen(3000);
}

// OR REGISTER IN MODULE
@Module({
  providers: [
    {
      provide: APP_FILTER,
      useClass: GlobalExceptionFilter,
    },
    {
      provide: APP_FILTER,
      useClass: DomainExceptionFilter,
    },
  ],
})
export class AppModule {}
```

---

### **Method 3: Consistent Error Response Format**

```typescript
// ========== STANDARD ERROR RESPONSE FORMAT ==========

// ERROR RESPONSE DTO
export class ErrorResponseDto {
  statusCode: number;
  message: string;
  code: string;
  timestamp: string;
  path: string;
  method?: string;
  correlationId?: string;
  details?: any;
  stack?: string;  // Development only
}

// ERROR BUILDER
export class ErrorResponseBuilder {
  private error: Partial<ErrorResponseDto> = {};

  static create(): ErrorResponseBuilder {
    return new ErrorResponseBuilder();
  }

  withStatus(statusCode: number): this {
    this.error.statusCode = statusCode;
    return this;
  }

  withMessage(message: string): this {
    this.error.message = message;
    return this;
  }

  withCode(code: string): this {
    this.error.code = code;
    return this;
  }

  withPath(path: string): this {
    this.error.path = path;
    return this;
  }

  withMethod(method: string): this {
    this.error.method = method;
    return this;
  }

  withCorrelationId(id: string): this {
    this.error.correlationId = id;
    return this;
  }

  withDetails(details: any): this {
    this.error.details = details;
    return this;
  }

  withStack(stack: string): this {
    if (process.env.NODE_ENV === 'development') {
      this.error.stack = stack;
    }
    return this;
  }

  build(): ErrorResponseDto {
    this.error.timestamp = new Date().toISOString();
    return this.error as ErrorResponseDto;
  }
}

// USAGE IN FILTER
@Catch()
export class GlobalExceptionFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost): void {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    const errorResponse = ErrorResponseBuilder.create()
      .withStatus(this.getStatus(exception))
      .withMessage(this.getMessage(exception))
      .withCode(this.getCode(exception))
      .withPath(request.url)
      .withMethod(request.method)
      .withCorrelationId(request.headers['x-correlation-id'] as string)
      .withStack(exception instanceof Error ? exception.stack : undefined)
      .build();

    response.status(errorResponse.statusCode).json(errorResponse);
  }

  private getStatus(exception: unknown): number {
    if (exception instanceof DomainException) return exception.statusCode;
    if (exception instanceof HttpException) return exception.getStatus();
    return 500;
  }

  private getMessage(exception: unknown): string {
    if (exception instanceof Error) return exception.message;
    return 'Internal server error';
  }

  private getCode(exception: unknown): string {
    if (exception instanceof DomainException) return exception.code;
    if (exception instanceof Error) return exception.name;
    return 'INTERNAL_ERROR';
  }
}

// EXAMPLE ERROR RESPONSES

// 400 Bad Request - Business rule violation
{
  "statusCode": 400,
  "message": "Insufficient stock for product ABC123. Requested: 10, Available: 5",
  "code": "INSUFFICIENT_STOCK",
  "timestamp": "2025-12-30T10:30:00.000Z",
  "path": "/api/orders",
  "method": "POST",
  "correlationId": "req-123-456"
}

// 404 Not Found
{
  "statusCode": 404,
  "message": "Order with id 12345 not found",
  "code": "NOT_FOUND",
  "timestamp": "2025-12-30T10:30:00.000Z",
  "path": "/api/orders/12345",
  "method": "GET"
}

// 500 Internal Server Error (with details in development)
{
  "statusCode": 500,
  "message": "Database connection failed",
  "code": "DATABASE_ERROR",
  "timestamp": "2025-12-30T10:30:00.000Z",
  "path": "/api/users",
  "method": "GET",
  "stack": "Error: Connection timeout\n    at Connection.connect (...)"
}
```

---

### **Method 4: Logging Strategy**

```typescript
// ========== COMPREHENSIVE ERROR LOGGING ==========

@Injectable()
export class ErrorLogger {
  constructor(private readonly logger: Logger) {}

  logBusinessRuleViolation(
    exception: DomainException,
    context: Record<string, any>,
  ): void {
    // Business rules violations are WARNINGS, not errors
    this.logger.warn({
      type: 'BUSINESS_RULE_VIOLATION',
      code: exception.code,
      message: exception.message,
      ...context,
    });
  }

  logApplicationError(
    exception: Error,
    context: Record<string, any>,
  ): void {
    // Application errors are ERRORS
    this.logger.error({
      type: 'APPLICATION_ERROR',
      name: exception.name,
      message: exception.message,
      stack: exception.stack,
      ...context,
    });
  }

  logInfrastructureError(
    exception: Error,
    serviceName: string,
    context: Record<string, any>,
  ): void {
    // Infrastructure errors are CRITICAL
    this.logger.error({
      type: 'INFRASTRUCTURE_ERROR',
      service: serviceName,
      name: exception.name,
      message: exception.message,
      stack: exception.stack,
      ...context,
    }, 'CRITICAL');
  }
}

// ENHANCED EXCEPTION FILTER WITH LOGGING
@Catch()
export class GlobalExceptionFilter implements ExceptionFilter {
  constructor(
    private readonly logger: Logger,
    private readonly errorLogger: ErrorLogger,
  ) {}

  catch(exception: unknown, host: ArgumentsHost): void {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    // Build context for logging
    const context = {
      path: request.url,
      method: request.method,
      correlationId: request.headers['x-correlation-id'],
      userId: (request as any).user?.id,
      ip: request.ip,
      userAgent: request.headers['user-agent'],
    };

    // Log based on exception type
    if (exception instanceof DomainException) {
      this.errorLogger.logBusinessRuleViolation(exception, context);
    } else if (exception instanceof ExternalServiceException) {
      this.errorLogger.logInfrastructureError(
        exception,
        exception.serviceName,
        context,
      );
    } else if (exception instanceof Error) {
      this.errorLogger.logApplicationError(exception, context);
    }

    // Send response...
  }
}
```

---

### **Method 5: Graceful Degradation & Fallbacks**

```typescript
// ========== GRACEFUL DEGRADATION ==========

@Injectable()
export class OrderService {
  constructor(
    private readonly orderRepository: OrderRepository,
    private readonly paymentService: PaymentService,
    private readonly inventoryService: InventoryService,
    private readonly logger: Logger,
  ) {}

  async placeOrder(dto: PlaceOrderDto): Promise<Order> {
    // Critical operations - fail if they fail
    const order = await this.createOrder(dto);

    try {
      // Try to process payment
      await this.paymentService.charge(dto.customerId, order.total);
    } catch (error) {
      // Payment failed - mark order as failed
      await this.orderRepository.updateStatus(order.id, OrderStatus.FAILED);
      throw new PaymentFailedException('Payment processing failed');
    }

    // Non-critical operations - degrade gracefully
    try {
      // Try to reserve inventory (best effort)
      await this.inventoryService.reserveItems(dto.items);
    } catch (error) {
      // Inventory reservation failed - log and continue
      this.logger.warn('Inventory reservation failed, continuing anyway', {
        orderId: order.id,
        error: error.message,
      });
      // Order still succeeds, inventory will be handled by background job
    }

    return order;
  }

  // WITH FALLBACK
  async getProductRecommendations(userId: string): Promise<Product[]> {
    try {
      // Try to get personalized recommendations from ML service
      return await this.mlService.getRecommendations(userId);
    } catch (error) {
      this.logger.warn('ML service unavailable, using fallback', {
        userId,
        error: error.message,
      });
      
      // Fallback: return popular products
      return this.productService.getPopularProducts();
    }
  }

  // WITH RETRY
  async sendOrderConfirmation(orderId: string): Promise<void> {
    const maxRetries = 3;
    let attempt = 0;

    while (attempt < maxRetries) {
      try {
        await this.emailService.sendOrderConfirmation(orderId);
        return; // Success
      } catch (error) {
        attempt++;
        
        if (attempt >= maxRetries) {
          // Max retries reached - log and fail gracefully
          this.logger.error('Failed to send order confirmation after retries', {
            orderId,
            attempts: attempt,
          });
          
          // Queue for later retry instead of failing the order
          await this.queueService.add('send-email', { orderId });
          return;
        }

        // Wait before retry (exponential backoff)
        await this.delay(Math.pow(2, attempt) * 1000);
      }
    }
  }

  private delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}
```

---

### **Error Handling Structure Comparison:**

| Aspect | Poor Structure | Good Structure |
|--------|----------------|----------------|
| **Exception Types** | Generic `throw new Error()` | Custom domain exceptions |
| **Error Codes** | None or inconsistent | Enum-based, consistent codes |
| **Response Format** | Varies by endpoint | Standardized across app |
| **Logging** | Scattered console.logs | Centralized, contextual |
| **Filters** | None or basic | Global + domain-specific |
| **Business Rules** | Mixed with technical errors | Separate domain exceptions |
| **Development** | No stack traces | Stack traces in dev only |

---

### **Best Practices:**

```typescript
// ✅ GOOD: Structured error handling
@Injectable()
export class OrderService {
  async placeOrder(dto: PlaceOrderDto): Promise<Order> {
    // Throw specific domain exceptions
    if (dto.items.length === 0) {
      throw new InvalidOrderException('Order must have items');
    }

    // Let exceptions bubble up
    const order = await this.createOrder(dto);
    return order;
  }
}

// ❌ BAD: Generic errors
async placeOrder(dto: PlaceOrderDto): Promise<Order> {
  if (dto.items.length === 0) {
    throw new Error('Bad order');  // Too generic!
  }
}

// ✅ GOOD: Consistent error format
{
  "statusCode": 400,
  "code": "INSUFFICIENT_STOCK",
  "message": "Clear, actionable message",
  "timestamp": "2025-12-30T10:30:00.000Z"
}

// ❌ BAD: Inconsistent errors
{ "error": "Something went wrong" }  // No structure
throw "Order failed";  // String instead of Error
```

**Key Takeaway:** Structure error handling with **custom exception hierarchies** (DomainException base class, business exceptions like `InvalidOrderException`, application exceptions like `NotFoundException`, infrastructure exceptions like `ExternalServiceException`), **exception filters** (global filter catches all, domain filter for business rules, HTTP filter for NestJS exceptions), **consistent error response format** (statusCode, message, code, timestamp, path, correlationId), **error codes enum** (machine-readable like `INSUFFICIENT_STOCK`, `ORDER_INVALID`), **contextual logging** (business rules = WARN level, application errors = ERROR, infrastructure = CRITICAL), and **graceful degradation** (critical operations fail, non-critical degrade with fallbacks). Use **specific exceptions** (throw domain exceptions early), **never swallow errors** silently, **log with context** (correlationId, userId, path), **separate business rule violations** (400) from technical errors (500), **include stack traces in development only**, and provide **actionable error messages** to clients. Test error scenarios, monitor error rates, and implement circuit breakers for external services.

</details>

<details>
<summary><strong>30. Should you use try-catch in controllers or services?</strong></summary>

**Answer:**

**Prefer try-catch in services**, not controllers. **Controllers** should let exceptions bubble up to **exception filters** (global error handling). **Services** use try-catch for **resource cleanup** (transactions, file handles), **wrapping external errors** (third-party APIs), **retry logic**, **partial failure handling**, and **converting technical exceptions to domain exceptions**. **Never** catch exceptions just to log and re-throw. **Rule**: Only use try-catch if you're going to **do something meaningful** (cleanup, retry, wrap, degrade gracefully). Let NestJS exception filters handle final HTTP error responses. Controllers should be **exception-transparent** (no try-catch unless necessary).

---

### **Try-Catch Placement Strategy:**

```
┌─────────────────────────────────────────────────────┐
│              CONTROLLERS (No try-catch)             │
│  Let exceptions bubble up                          │
├─────────────────────────────────────────────────────┤
│  @Get()                                            │
│  async findUser(@Param('id') id: string) {        │
│    return this.userService.findById(id);          │
│  }                                                 │
│  // No try-catch! Let filter handle it            │
└─────────────────────────────────────────────────────┘
                    │
                    │ Exception bubbles up
                    ▼
┌─────────────────────────────────────────────────────┐
│         SERVICES (try-catch when needed)           │
│  Cleanup, retry, wrap, degrade                     │
├─────────────────────────────────────────────────────┤
│  ✅ Transaction rollback                           │
│  ✅ Resource cleanup (files, connections)          │
│  ✅ Wrap external service errors                   │
│  ✅ Retry logic                                     │
│  ✅ Partial failure handling                       │
│  ✅ Convert technical → domain exceptions          │
│  ❌ Just logging and re-throwing                   │
└─────────────────────────────────────────────────────┘
                    │
                    │ If exception not handled
                    ▼
┌─────────────────────────────────────────────────────┐
│         EXCEPTION FILTERS (Final handler)          │
│  Transform to HTTP response                        │
└─────────────────────────────────────────────────────┘
```

---

### **Method 1: Controllers - Let Exceptions Bubble**

```typescript
// ========== CONTROLLERS: DON'T CATCH EXCEPTIONS ==========

// ✅ GOOD: No try-catch in controller
@Controller('users')
export class UsersController {
  constructor(private readonly userService: UserService) {}

  @Get(':id')
  async findOne(@Param('id') id: string): Promise<UserResponseDto> {
    // Just call service - let exceptions bubble up
    const user = await this.userService.findById(id);
    return this.toDto(user);
  }

  @Post()
  async create(@Body() dto: CreateUserDto): Promise<UserResponseDto> {
    // Service might throw exceptions - that's fine
    const user = await this.userService.createUser(dto);
    return this.toDto(user);
  }

  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  async remove(@Param('id') id: string): Promise<void> {
    // Let service exceptions propagate to exception filter
    await this.userService.deleteUser(id);
  }

  private toDto(user: User): UserResponseDto {
    return { id: user.id, email: user.email, name: user.name };
  }
}

// ❌ BAD: Unnecessary try-catch in controller
@Controller('users')
export class UsersController {
  @Get(':id')
  async findOne(@Param('id') id: string): Promise<UserResponseDto> {
    try {
      const user = await this.userService.findById(id);
      return this.toDto(user);
    } catch (error) {
      // Pointless! Just re-throwing
      throw error;
    }
  }

  @Post()
  async create(@Body() dto: CreateUserDto): Promise<UserResponseDto> {
    try {
      const user = await this.userService.createUser(dto);
      return this.toDto(user);
    } catch (error) {
      // Converting to HTTP exception - let filter do this!
      if (error instanceof ConflictException) {
        throw new HttpException('User exists', HttpStatus.CONFLICT);
      }
      throw error;
    }
  }
}

// ⚠️ RARE CASE: try-catch in controller (edge case)
@Controller('files')
export class FilesController {
  @Post('upload')
  @UseInterceptors(FileInterceptor('file'))
  async uploadFile(@UploadedFile() file: Express.Multer.File) {
    try {
      return await this.filesService.upload(file);
    } catch (error) {
      // Cleanup uploaded file if processing fails
      await fs.promises.unlink(file.path);
      throw error;
    }
  }
}
```

---

### **Method 2: Services - Try-Catch for Specific Purposes**

```typescript
// ========== SERVICES: USE TRY-CATCH STRATEGICALLY ==========

// ✅ USE CASE 1: Transaction Rollback
@Injectable()
export class OrderService {
  constructor(
    private readonly dataSource: DataSource,
    private readonly orderRepository: OrderRepository,
    private readonly inventoryService: InventoryService,
  ) {}

  async placeOrder(dto: PlaceOrderDto): Promise<Order> {
    const queryRunner = this.dataSource.createQueryRunner();
    await queryRunner.connect();
    await queryRunner.startTransaction();

    try {
      // Create order
      const order = await this.orderRepository.create(dto);
      await queryRunner.manager.save(order);

      // Reserve inventory
      await this.inventoryService.reserveItems(dto.items, queryRunner.manager);

      // Commit transaction
      await queryRunner.commitTransaction();
      
      return order;
    } catch (error) {
      // Rollback transaction on any error
      await queryRunner.rollbackTransaction();
      throw error;  // Re-throw so filter can handle it
    } finally {
      // Always release connection
      await queryRunner.release();
    }
  }
}

// ✅ USE CASE 2: Wrap External Service Errors
@Injectable()
export class PaymentService {
  constructor(
    private readonly stripeClient: Stripe,
    private readonly logger: Logger,
  ) {}

  async charge(customerId: string, amount: number): Promise<PaymentResult> {
    try {
      // Call external payment API
      const charge = await this.stripeClient.charges.create({
        customer: customerId,
        amount: amount * 100,  // Cents
        currency: 'usd',
      });

      return {
        success: true,
        transactionId: charge.id,
      };
    } catch (error) {
      // Wrap Stripe errors into domain exceptions
      this.logger.error('Payment failed', { customerId, amount, error });
      
      if (error.code === 'card_declined') {
        throw new PaymentDeclinedException('Card was declined');
      }
      
      if (error.code === 'insufficient_funds') {
        throw new InsufficientFundsException('Insufficient funds');
      }
      
      // Generic payment error
      throw new PaymentFailedException(`Payment failed: ${error.message}`);
    }
  }
}

// ✅ USE CASE 3: Retry Logic
@Injectable()
export class EmailService {
  async sendOrderConfirmation(orderId: string): Promise<void> {
    const maxRetries = 3;
    let lastError: Error;

    for (let attempt = 1; attempt <= maxRetries; attempt++) {
      try {
        await this.emailProvider.send({
          to: await this.getOrderEmail(orderId),
          subject: 'Order Confirmation',
          template: 'order-confirmation',
          data: { orderId },
        });
        
        return;  // Success!
      } catch (error) {
        lastError = error;
        
        this.logger.warn(`Email send failed (attempt ${attempt}/${maxRetries})`, {
          orderId,
          error: error.message,
        });

        if (attempt < maxRetries) {
          // Wait before retry (exponential backoff)
          await this.delay(Math.pow(2, attempt) * 1000);
        }
      }
    }

    // All retries failed
    throw new EmailSendFailedException(
      `Failed to send email after ${maxRetries} attempts: ${lastError.message}`,
    );
  }

  private delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

// ✅ USE CASE 4: Partial Failure Handling
@Injectable()
export class OrderService {
  async processOrder(orderId: string): Promise<ProcessingResult> {
    const order = await this.orderRepository.findById(orderId);
    const results: ProcessingResult = {
      orderId,
      success: true,
      warnings: [],
    };

    // Critical operation - must succeed
    await this.paymentService.charge(order.customerId, order.total);

    // Non-critical operation - degrade gracefully
    try {
      await this.inventoryService.updateStock(order.items);
    } catch (error) {
      // Log warning but don't fail the order
      this.logger.warn('Inventory update failed', { orderId, error });
      results.warnings.push('Inventory update deferred');
      // Queue for background processing
      await this.queueService.add('update-inventory', { orderId });
    }

    // Another non-critical operation
    try {
      await this.analyticsService.trackOrder(order);
    } catch (error) {
      // Analytics failure shouldn't affect order
      this.logger.warn('Analytics tracking failed', { orderId, error });
      results.warnings.push('Analytics tracking failed');
    }

    return results;
  }
}

// ✅ USE CASE 5: Resource Cleanup
@Injectable()
export class FileService {
  async processUpload(file: Express.Multer.File): Promise<ProcessedFile> {
    const tempPath = file.path;

    try {
      // Process file
      const processed = await this.imageProcessor.resize(tempPath, 800, 600);
      
      // Upload to S3
      const url = await this.s3Service.upload(processed);
      
      return { url, size: processed.size };
    } catch (error) {
      // Cleanup temp file on error
      this.logger.error('File processing failed', { file: file.filename, error });
      throw new FileProcessingException(`Failed to process file: ${error.message}`);
    } finally {
      // Always cleanup temp file
      try {
        await fs.promises.unlink(tempPath);
      } catch (unlinkError) {
        this.logger.warn('Failed to cleanup temp file', { path: tempPath });
      }
    }
  }
}

// ❌ ANTI-PATTERN: Catch just to log and re-throw
@Injectable()
export class UserService {
  async createUser(dto: CreateUserDto): Promise<User> {
    try {
      const user = await this.userRepository.save(dto);
      return user;
    } catch (error) {
      // Pointless! Exception filter will log anyway
      this.logger.error('User creation failed', error);
      throw error;  // Just re-throwing
    }
  }
}

// ❌ ANTI-PATTERN: Swallowing exceptions
@Injectable()
export class NotificationService {
  async notify(userId: string, message: string): Promise<void> {
    try {
      await this.pushService.send(userId, message);
    } catch (error) {
      // Silently swallowing error - bad!
      // At minimum, log it or queue for retry
    }
  }
}
```

---

### **Method 3: When to Use Try-Catch (Decision Tree)**

```typescript
// ========== DECISION TREE: SHOULD I USE TRY-CATCH? ==========

// ❓ Question 1: Do I need to CLEANUP resources?
//    ✅ YES → Use try-catch with finally
async processWithTransaction() {
  const queryRunner = this.dataSource.createQueryRunner();
  try {
    await queryRunner.startTransaction();
    // ... work ...
    await queryRunner.commitTransaction();
  } catch (error) {
    await queryRunner.rollbackTransaction();
    throw error;
  } finally {
    await queryRunner.release();  // Always cleanup
  }
}

// ❓ Question 2: Do I need to WRAP external errors?
//    ✅ YES → Use try-catch to convert to domain exceptions
async callExternalApi() {
  try {
    return await this.httpClient.get('https://api.example.com');
  } catch (error) {
    throw new ExternalServiceException('API', error.message);
  }
}

// ❓ Question 3: Do I need to RETRY on failure?
//    ✅ YES → Use try-catch in retry loop
async sendWithRetry() {
  for (let i = 0; i < 3; i++) {
    try {
      return await this.send();
    } catch (error) {
      if (i === 2) throw error;
      await this.delay(1000);
    }
  }
}

// ❓ Question 4: Can I DEGRADE GRACEFULLY on failure?
//    ✅ YES → Use try-catch to provide fallback
async getRecommendations() {
  try {
    return await this.mlService.getPersonalized();
  } catch (error) {
    return this.getPopular();  // Fallback
  }
}

// ❓ Question 5: Is this a NON-CRITICAL operation?
//    ✅ YES → Use try-catch to prevent cascade failure
async processOrder() {
  await this.payment.charge();  // Critical - let fail

  try {
    await this.analytics.track();  // Non-critical
  } catch (error) {
    this.logger.warn('Analytics failed');
    // Continue processing
  }
}

// ❓ Question 6: Just want to log and re-throw?
//    ❌ NO → Don't use try-catch, let filter handle it
async badExample() {
  try {
    return await this.service.doSomething();
  } catch (error) {
    this.logger.error(error);  // Filter will log anyway
    throw error;  // Just re-throwing - pointless!
  }
}
```

---

### **Method 4: Controller vs Service Error Handling**

```typescript
// ========== CONTROLLER VS SERVICE COMPARISON ==========

// SCENARIO 1: Simple CRUD

// Controller (no try-catch)
@Controller('products')
export class ProductsController {
  @Get(':id')
  async findOne(@Param('id') id: string): Promise<ProductDto> {
    return this.productService.findById(id);  // No try-catch
  }
}

// Service (throw exceptions)
@Injectable()
export class ProductService {
  async findById(id: string): Promise<Product> {
    const product = await this.productRepository.findById(id);
    
    if (!product) {
      throw new NotFoundException('Product', id);  // Exception filter handles
    }
    
    return product;
  }
}

// SCENARIO 2: Complex operation with external service

// Controller (still no try-catch)
@Controller('orders')
export class OrdersController {
  @Post()
  async placeOrder(@Body() dto: PlaceOrderDto): Promise<OrderDto> {
    return this.orderService.placeOrder(dto);  // Let service handle errors
  }
}

// Service (try-catch for external service)
@Injectable()
export class OrderService {
  async placeOrder(dto: PlaceOrderDto): Promise<Order> {
    const order = await this.createOrder(dto);

    // Try-catch for external payment service
    try {
      await this.paymentGateway.charge(order.total);
    } catch (error) {
      // Wrap external error
      throw new PaymentFailedException(error.message);
    }

    return order;
  }
}

// SCENARIO 3: Non-critical side effect

// Controller (no try-catch)
@Controller('users')
export class UsersController {
  @Post('register')
  async register(@Body() dto: RegisterDto): Promise<UserDto> {
    return this.userService.register(dto);
  }
}

// Service (try-catch for non-critical operations)
@Injectable()
export class UserService {
  async register(dto: RegisterDto): Promise<User> {
    // Critical: Create user (let fail if it fails)
    const user = await this.userRepository.save(dto);

    // Non-critical: Send welcome email (don't fail registration)
    try {
      await this.emailService.sendWelcome(user.email);
    } catch (error) {
      // Log but don't throw
      this.logger.warn('Welcome email failed', { userId: user.id, error });
      // Queue for retry
      await this.queueService.add('send-welcome', { userId: user.id });
    }

    return user;
  }
}
```

---

### **Try-Catch Placement Comparison:**

| Scenario | Controller | Service | Reason |
|----------|------------|---------|--------|
| **Simple CRUD** | No try-catch | No try-catch | Let filter handle |
| **External API call** | No try-catch | try-catch (wrap) | Convert to domain exception |
| **Transaction** | No try-catch | try-catch (rollback) | Cleanup on failure |
| **Retry logic** | No try-catch | try-catch (retry) | Handle transient failures |
| **Non-critical operation** | No try-catch | try-catch (degrade) | Prevent cascade failure |
| **Resource cleanup** | No try-catch | try-catch (finally) | Release connections/files |
| **Logging only** | No try-catch | No try-catch | Filter logs anyway |

---

### **Best Practices:**

```typescript
// ✅ GOOD: Let exceptions bubble in controller
@Controller('users')
export class UsersController {
  @Post()
  async create(@Body() dto: CreateUserDto) {
    return this.userService.createUser(dto);  // No try-catch
  }
}

// ✅ GOOD: Try-catch in service for cleanup
@Injectable()
export class OrderService {
  async placeOrder(dto: PlaceOrderDto): Promise<Order> {
    const queryRunner = this.dataSource.createQueryRunner();
    try {
      await queryRunner.startTransaction();
      const order = await this.createOrder(dto, queryRunner.manager);
      await queryRunner.commitTransaction();
      return order;
    } catch (error) {
      await queryRunner.rollbackTransaction();
      throw error;
    } finally {
      await queryRunner.release();
    }
  }
}

// ✅ GOOD: Try-catch for wrapping external errors
@Injectable()
export class PaymentService {
  async charge(amount: number): Promise<void> {
    try {
      await this.stripe.charges.create({ amount });
    } catch (error) {
      throw new PaymentFailedException(error.message);
    }
  }
}

// ❌ BAD: Try-catch in controller just to re-throw
@Controller('users')
export class UsersController {
  @Post()
  async create(@Body() dto: CreateUserDto) {
    try {
      return this.userService.createUser(dto);
    } catch (error) {
      throw error;  // Pointless!
    }
  }
}

// ❌ BAD: Swallowing exceptions
@Injectable()
export class UserService {
  async createUser(dto: CreateUserDto): Promise<User> {
    try {
      return await this.userRepository.save(dto);
    } catch (error) {
      // Silent failure - bad!
      return null;
    }
  }
}
```

**Key Takeaway:** **Controllers should NOT use try-catch** - let exceptions bubble up to exception filters for consistent error handling. **Services use try-catch ONLY for specific purposes**: **resource cleanup** (transactions, database connections, file handles with `finally` block), **wrapping external errors** (convert third-party API errors to domain exceptions), **retry logic** (transient failures with exponential backoff), **partial failure handling** (non-critical operations that shouldn't fail the entire flow), and **graceful degradation** (fallback strategies). **Never** use try-catch just to log and re-throw (exception filters log automatically). **Rule**: Only catch if you're going to **DO something meaningful** (cleanup, retry, wrap, degrade). Use `finally` for guaranteed cleanup, re-throw after handling, and log at appropriate levels (critical operations = ERROR, non-critical = WARN). Controllers should be **exception-transparent** (no error handling logic), services should be **exception-aware** (handle only when necessary).

</details>

<details>
<summary><strong>31. What is the global error handling strategy?</strong></summary>

**Answer:**

Global error handling strategy uses **Exception Filters** (`@Catch()` decorator), registered **globally** in `main.ts` or via `APP_FILTER` provider. Implement **layered exception filters**: **global filter** (catches all uncaught exceptions, final safety net), **HTTP exception filter** (handles `HttpException` from NestJS), **domain exception filter** (handles custom business exceptions), and **specific filters** (validation errors, TypeORM errors). Filters transform exceptions into **consistent HTTP responses** (statusCode, message, code, timestamp, path). Use **exception filter order** (specific → general). Enable **correlation IDs** (track requests across services), **structured logging** (context, user, error details), and **error monitoring** (Sentry, DataDog). Never expose **internal errors** to clients in production.

---

### **Global Error Handling Architecture:**

```
┌─────────────────────────────────────────────────────┐
│           INCOMING REQUEST                          │
└─────────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────┐
│        CONTROLLERS & SERVICES                       │
│        (throw exceptions)                           │
└─────────────────────────────────────────────────────┘
                    │
                    │ Exception thrown
                    ▼
┌─────────────────────────────────────────────────────┐
│        EXCEPTION FILTER CHAIN                       │
│        (specific → general)                         │
├─────────────────────────────────────────────────────┤
│                                                     │
│  1. Specific Exception Filters                     │
│     @Catch(ValidationException)                    │
│     @Catch(TypeORMError)                           │
│                                                     │
│  2. Domain Exception Filter                        │
│     @Catch(DomainException)                        │
│                                                     │
│  3. HTTP Exception Filter                          │
│     @Catch(HttpException)                          │
│                                                     │
│  4. Global Exception Filter (catch-all)            │
│     @Catch()                                        │
│                                                     │
└─────────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────┐
│        CONSISTENT ERROR RESPONSE                    │
│  {                                                  │
│    statusCode: 400,                                │
│    message: "Clear error message",                 │
│    code: "BUSINESS_RULE_VIOLATION",                │
│    timestamp: "2025-12-30T10:00:00.000Z",          │
│    path: "/api/orders",                            │
│    correlationId: "req-123-456"                    │
│  }                                                  │
└─────────────────────────────────────────────────────┘
```

---

### **Method 1: Global Exception Filter Setup**

```typescript
// ========== GLOBAL EXCEPTION FILTER ==========

// Global filter (catch-all, final safety net)
@Catch()
export class GlobalExceptionFilter implements ExceptionFilter {
  constructor(
    private readonly logger: Logger,
    private readonly configService: ConfigService,
  ) {}

  catch(exception: unknown, host: ArgumentsHost): void {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    const isDevelopment = this.configService.get('NODE_ENV') === 'development';

    // Determine status code
    let status = HttpStatus.INTERNAL_SERVER_ERROR;
    let message = 'Internal server error';
    let code = 'INTERNAL_ERROR';

    if (exception instanceof HttpException) {
      status = exception.getStatus();
      const exceptionResponse = exception.getResponse();
      
      if (typeof exceptionResponse === 'object') {
        message = (exceptionResponse as any).message || exception.message;
        code = (exceptionResponse as any).error || 'HTTP_EXCEPTION';
      } else {
        message = exceptionResponse as string;
      }
    } else if (exception instanceof Error) {
      message = exception.message;
      code = exception.name;
    }

    // Get correlation ID from request headers or generate one
    const correlationId = 
      (request.headers['x-correlation-id'] as string) || 
      this.generateCorrelationId();

    // Build error response
    const errorResponse: any = {
      statusCode: status,
      message: isDevelopment ? message : this.sanitizeMessage(message, status),
      code,
      timestamp: new Date().toISOString(),
      path: request.url,
      method: request.method,
      correlationId,
    };

    // Include stack trace only in development
    if (isDevelopment && exception instanceof Error) {
      errorResponse.stack = exception.stack;
    }

    // Log error with context
    this.logError(exception, request, correlationId);

    // Send response
    response.status(status).json(errorResponse);
  }

  private logError(
    exception: unknown,
    request: Request,
    correlationId: string,
  ): void {
    const context = {
      correlationId,
      path: request.url,
      method: request.method,
      userId: (request as any).user?.id,
      ip: request.ip,
      userAgent: request.headers['user-agent'],
    };

    if (exception instanceof HttpException) {
      // 4xx errors are client errors (warn level)
      if (exception.getStatus() < 500) {
        this.logger.warn({
          message: exception.message,
          statusCode: exception.getStatus(),
          ...context,
        });
      } else {
        // 5xx errors are server errors (error level)
        this.logger.error({
          message: exception.message,
          statusCode: exception.getStatus(),
          stack: exception.stack,
          ...context,
        });
      }
    } else if (exception instanceof Error) {
      this.logger.error({
        message: exception.message,
        name: exception.name,
        stack: exception.stack,
        ...context,
      });
    } else {
      this.logger.error({
        message: 'Unknown exception',
        exception: String(exception),
        ...context,
      });
    }
  }

  private sanitizeMessage(message: string, status: number): string {
    // Don't expose internal errors in production
    if (status >= 500) {
      return 'An unexpected error occurred. Please try again later.';
    }
    return message;
  }

  private generateCorrelationId(): string {
    return `req-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }
}

// REGISTER IN MAIN.TS (Option 1: Direct registration)
async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Register global exception filter
  const logger = app.get(Logger);
  const configService = app.get(ConfigService);
  
  app.useGlobalFilters(
    new GlobalExceptionFilter(logger, configService),
  );

  await app.listen(3000);
}

// REGISTER IN APP MODULE (Option 2: Dependency injection)
@Module({
  providers: [
    Logger,
    {
      provide: APP_FILTER,
      useClass: GlobalExceptionFilter,
    },
  ],
})
export class AppModule {}
```

---

### **Method 2: Layered Exception Filters**

```typescript
// ========== LAYERED EXCEPTION FILTER STRATEGY ==========

// LAYER 1: Domain Exception Filter (business rules)
@Catch(DomainException)
export class DomainExceptionFilter implements ExceptionFilter {
  constructor(private readonly logger: Logger) {}

  catch(exception: DomainException, host: ArgumentsHost): void {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    // Business rule violations are warnings (expected errors)
    this.logger.warn({
      type: 'BUSINESS_RULE_VIOLATION',
      code: exception.code,
      message: exception.message,
      path: request.url,
      userId: (request as any).user?.id,
    });

    response.status(exception.statusCode).json({
      statusCode: exception.statusCode,
      message: exception.message,
      code: exception.code,
      timestamp: new Date().toISOString(),
      path: request.url,
    });
  }
}

// LAYER 2: Validation Exception Filter (DTO validation)
@Catch(ValidationException)
export class ValidationExceptionFilter implements ExceptionFilter {
  catch(exception: ValidationException, host: ArgumentsHost): void {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    response.status(HttpStatus.BAD_REQUEST).json({
      statusCode: HttpStatus.BAD_REQUEST,
      message: 'Validation failed',
      code: 'VALIDATION_ERROR',
      errors: exception.errors,  // Detailed validation errors
      timestamp: new Date().toISOString(),
      path: request.url,
    });
  }
}

// LAYER 3: TypeORM Exception Filter (database errors)
@Catch(QueryFailedError)
export class TypeORMExceptionFilter implements ExceptionFilter {
  constructor(private readonly logger: Logger) {}

  catch(exception: QueryFailedError, host: ArgumentsHost): void {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    this.logger.error({
      type: 'DATABASE_ERROR',
      message: exception.message,
      query: exception.query,
      parameters: exception.parameters,
      path: request.url,
    });

    // Handle specific database errors
    let statusCode = HttpStatus.INTERNAL_SERVER_ERROR;
    let message = 'Database error occurred';
    let code = 'DATABASE_ERROR';

    // Unique constraint violation
    if ((exception as any).code === '23505') {
      statusCode = HttpStatus.CONFLICT;
      message = 'Record already exists';
      code = 'DUPLICATE_ENTRY';
    }

    // Foreign key constraint violation
    if ((exception as any).code === '23503') {
      statusCode = HttpStatus.BAD_REQUEST;
      message = 'Referenced record does not exist';
      code = 'FOREIGN_KEY_VIOLATION';
    }

    response.status(statusCode).json({
      statusCode,
      message,
      code,
      timestamp: new Date().toISOString(),
      path: request.url,
    });
  }
}

// LAYER 4: HTTP Exception Filter (NestJS exceptions)
@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  constructor(private readonly logger: Logger) {}

  catch(exception: HttpException, host: ArgumentsHost): void {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const status = exception.getStatus();

    const exceptionResponse = exception.getResponse();
    let message = exception.message;
    let code = 'HTTP_EXCEPTION';

    if (typeof exceptionResponse === 'object') {
      message = (exceptionResponse as any).message || message;
      code = (exceptionResponse as any).error || code;
    }

    // Log 4xx as warnings, 5xx as errors
    const logLevel = status < 500 ? 'warn' : 'error';
    this.logger[logLevel]({
      statusCode: status,
      message,
      path: request.url,
      method: request.method,
    });

    response.status(status).json({
      statusCode: status,
      message,
      code,
      timestamp: new Date().toISOString(),
      path: request.url,
    });
  }
}

// LAYER 5: Global catch-all (see Method 1)

// REGISTER ALL FILTERS IN ORDER (specific → general)
@Module({
  providers: [
    Logger,
    {
      provide: APP_FILTER,
      useClass: ValidationExceptionFilter,  // Most specific
    },
    {
      provide: APP_FILTER,
      useClass: TypeORMExceptionFilter,
    },
    {
      provide: APP_FILTER,
      useClass: DomainExceptionFilter,
    },
    {
      provide: APP_FILTER,
      useClass: HttpExceptionFilter,
    },
    {
      provide: APP_FILTER,
      useClass: GlobalExceptionFilter,  // Most general (catch-all)
    },
  ],
})
export class AppModule {}
```

---

### **Method 3: Correlation ID & Request Tracking**

```typescript
// ========== CORRELATION ID MIDDLEWARE ==========

// Middleware: Add correlation ID to all requests
@Injectable()
export class CorrelationIdMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction): void {
    // Get correlation ID from header or generate new one
    const correlationId = 
      (req.headers['x-correlation-id'] as string) || 
      this.generateCorrelationId();

    // Store in request for later use
    (req as any).correlationId = correlationId;

    // Set response header
    res.setHeader('X-Correlation-Id', correlationId);

    next();
  }

  private generateCorrelationId(): string {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }
}

// Register middleware
@Module({})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer): void {
    consumer.apply(CorrelationIdMiddleware).forRoutes('*');
  }
}

// ENHANCED LOGGER WITH CORRELATION ID
@Injectable()
export class RequestLogger {
  constructor(private readonly logger: Logger) {}

  log(message: string, context?: Record<string, any>): void {
    this.logger.log({
      message,
      correlationId: context?.correlationId,
      ...context,
    });
  }

  error(
    message: string,
    error: Error,
    context?: Record<string, any>,
  ): void {
    this.logger.error({
      message,
      error: error.message,
      stack: error.stack,
      correlationId: context?.correlationId,
      ...context,
    });
  }
}

// USAGE IN EXCEPTION FILTER
@Catch()
export class GlobalExceptionFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost): void {
    const ctx = host.switchToHttp();
    const request = ctx.getRequest<Request>();
    const correlationId = (request as any).correlationId;

    // Include correlation ID in error response
    const errorResponse = {
      statusCode: this.getStatus(exception),
      message: this.getMessage(exception),
      correlationId,  // Client can use this to track their request
      timestamp: new Date().toISOString(),
      path: request.url,
    };

    // Log with correlation ID
    this.logger.error({
      message: 'Request failed',
      correlationId,
      exception: exception instanceof Error ? exception.message : String(exception),
      path: request.url,
    });

    // ...
  }
}
```

---

### **Method 4: Error Monitoring Integration**

```typescript
// ========== ERROR MONITORING (SENTRY, DATADOG, ETC.) ==========

// Sentry integration
import * as Sentry from '@sentry/node';

@Catch()
export class GlobalExceptionFilter implements ExceptionFilter {
  constructor(private readonly logger: Logger) {}

  catch(exception: unknown, host: ArgumentsHost): void {
    const ctx = host.switchToHttp();
    const request = ctx.getRequest<Request>();
    const response = ctx.getResponse<Response>();

    // Send to Sentry (only for 5xx errors)
    if (this.is5xxError(exception)) {
      Sentry.captureException(exception, {
        tags: {
          path: request.url,
          method: request.method,
        },
        user: {
          id: (request as any).user?.id,
          ip_address: request.ip,
        },
        extra: {
          correlationId: (request as any).correlationId,
          body: request.body,
          query: request.query,
          params: request.params,
        },
      });
    }

    // Regular error handling
    const errorResponse = this.buildErrorResponse(exception, request);
    this.logger.error(errorResponse);
    response.status(errorResponse.statusCode).json(errorResponse);
  }

  private is5xxError(exception: unknown): boolean {
    if (exception instanceof HttpException) {
      return exception.getStatus() >= 500;
    }
    // Unknown exceptions are treated as 5xx
    return true;
  }

  private buildErrorResponse(exception: unknown, request: Request): any {
    // Build error response...
  }
}

// Initialize Sentry in main.ts
async function bootstrap() {
  // Initialize Sentry
  Sentry.init({
    dsn: process.env.SENTRY_DSN,
    environment: process.env.NODE_ENV,
    tracesSampleRate: 1.0,
  });

  const app = await NestFactory.create(AppModule);
  
  // Add Sentry request handler
  app.use(Sentry.Handlers.requestHandler());
  
  // Add global exception filter
  app.useGlobalFilters(new GlobalExceptionFilter(new Logger()));
  
  // Add Sentry error handler (must be after exception filter)
  app.use(Sentry.Handlers.errorHandler());

  await app.listen(3000);
}
```

---

### **Method 5: Environment-Specific Error Responses**

```typescript
// ========== ENVIRONMENT-SPECIFIC ERROR HANDLING ==========

@Catch()
export class GlobalExceptionFilter implements ExceptionFilter {
  constructor(
    private readonly logger: Logger,
    private readonly configService: ConfigService,
  ) {}

  catch(exception: unknown, host: ArgumentsHost): void {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    const isDevelopment = this.configService.get('NODE_ENV') === 'development';
    const isProduction = this.configService.get('NODE_ENV') === 'production';

    const status = this.getStatus(exception);
    const message = this.getMessage(exception);

    // DEVELOPMENT: Include detailed information
    if (isDevelopment) {
      response.status(status).json({
        statusCode: status,
        message,
        code: this.getCode(exception),
        timestamp: new Date().toISOString(),
        path: request.url,
        method: request.method,
        // Additional dev info
        stack: exception instanceof Error ? exception.stack : undefined,
        request: {
          body: request.body,
          query: request.query,
          params: request.params,
          headers: request.headers,
        },
      });
      return;
    }

    // PRODUCTION: Sanitized error messages
    if (isProduction) {
      response.status(status).json({
        statusCode: status,
        // Sanitize 5xx error messages
        message: status >= 500 
          ? 'An unexpected error occurred. Please try again later.' 
          : message,
        code: this.getCode(exception),
        timestamp: new Date().toISOString(),
        correlationId: (request as any).correlationId,
        // No stack trace, no request details in production
      });
      return;
    }

    // STAGING/TEST: Moderate detail
    response.status(status).json({
      statusCode: status,
      message,
      code: this.getCode(exception),
      timestamp: new Date().toISOString(),
      path: request.url,
      correlationId: (request as any).correlationId,
    });
  }

  private getStatus(exception: unknown): number {
    if (exception instanceof HttpException) {
      return exception.getStatus();
    }
    return HttpStatus.INTERNAL_SERVER_ERROR;
  }

  private getMessage(exception: unknown): string {
    if (exception instanceof Error) {
      return exception.message;
    }
    return 'Internal server error';
  }

  private getCode(exception: unknown): string {
    if (exception instanceof DomainException) {
      return exception.code;
    }
    if (exception instanceof Error) {
      return exception.name;
    }
    return 'INTERNAL_ERROR';
  }
}
```

---

### **Global Error Handling Strategy Checklist:**

| Component | Implementation | Status |
|-----------|----------------|--------|
| **Global Filter** | `@Catch()` with `APP_FILTER` | ✅ Required |
| **Layered Filters** | Domain, HTTP, Validation, Database | ✅ Recommended |
| **Correlation IDs** | Middleware + response header | ✅ Required |
| **Structured Logging** | Context, correlationId, userId | ✅ Required |
| **Error Monitoring** | Sentry, DataDog, CloudWatch | ✅ Production |
| **Environment Handling** | Different responses dev/prod | ✅ Required |
| **Sanitized Messages** | Hide internal errors in production | ✅ Required |
| **Consistent Format** | Standard error response DTO | ✅ Required |
| **Error Codes** | Machine-readable identifiers | ✅ Recommended |

---

### **Best Practices:**

```typescript
// ✅ GOOD: Layered exception filters
@Module({
  providers: [
    { provide: APP_FILTER, useClass: ValidationExceptionFilter },
    { provide: APP_FILTER, useClass: DomainExceptionFilter },
    { provide: APP_FILTER, useClass: GlobalExceptionFilter },
  ],
})
export class AppModule {}

// ✅ GOOD: Correlation ID tracking
@Injectable()
export class CorrelationIdMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction): void {
    const correlationId = req.headers['x-correlation-id'] || uuidv4();
    (req as any).correlationId = correlationId;
    res.setHeader('X-Correlation-Id', correlationId);
    next();
  }
}

// ✅ GOOD: Environment-specific error details
const errorResponse = {
  statusCode: status,
  message: isProduction && status >= 500 
    ? 'Internal server error' 
    : exception.message,
  ...(isDevelopment && { stack: exception.stack }),
};

// ❌ BAD: Exposing internal errors in production
response.json({
  message: exception.message,  // May contain sensitive info!
  stack: exception.stack,  // Never expose in production!
});

// ❌ BAD: No correlation ID
// Can't track request across services or logs

// ❌ BAD: Inconsistent error format
// Different endpoints return different error structures
```

**Key Takeaway:** Global error handling uses **Exception Filters** registered globally via `APP_FILTER` provider or `app.useGlobalFilters()` in `main.ts`. Implement **layered strategy**: **specific filters** (ValidationException, TypeORMError) → **domain filter** (business rules) → **HTTP filter** (NestJS exceptions) → **global catch-all** (safety net). All filters transform exceptions to **consistent format** (statusCode, message, code, timestamp, path, correlationId). Use **correlation IDs** (middleware generates, stores in request, includes in responses and logs) to track requests across services. Implement **environment-specific handling**: **development** (detailed stack traces, request details), **production** (sanitized messages, no internal details), **staging** (moderate detail). Integrate **error monitoring** (Sentry, DataDog) for 5xx errors. Use **structured logging** (JSON format with context: correlationId, userId, path, method). Never expose **database errors**, **stack traces**, or **sensitive data** in production. Log **business rule violations as WARNINGS** (expected), **application errors as ERRORS**, **infrastructure failures as CRITICAL**. Test exception filters, monitor error rates, and set up alerts for critical errors.

</details>

## Code Organization

<details>
<summary><strong>32. How do you structure a large NestJS project?</strong></summary>

**Answer:**

Structure large NestJS projects using **feature-based organization** (modules grouped by business domain), **layered architecture** (controllers, services, repositories, entities per feature), **shared/common modules** (utilities, guards, interceptors), **core module** (singleton services like config, database), and **DDD principles** (domain, application, infrastructure layers). Use **monorepo** for microservices (Nx, Turborepo), **barrel exports** for clean imports, **clear naming conventions** (`.controller.ts`, `.service.ts`, `.module.ts`), and **separate concerns** (DTOs, interfaces, types, constants). Organize by **vertical slices** (feature modules are self-contained) not **horizontal slices** (all controllers together). Keep **modules focused** (single responsibility), use **dependency inversion** (interfaces in domain, implementations in infrastructure).

---

### **Large Project Structure:**

```
src/
├── main.ts                          # Application entry point
├── app.module.ts                    # Root module
│
├── core/                            # Singleton services, shared across app
│   ├── core.module.ts
│   ├── database/
│   │   ├── database.module.ts
│   │   └── database.providers.ts
│   ├── config/
│   │   ├── config.module.ts
│   │   ├── app.config.ts
│   │   └── database.config.ts
│   ├── logging/
│   │   ├── logging.module.ts
│   │   └── logger.service.ts
│   └── health/
│       ├── health.module.ts
│       └── health.controller.ts
│
├── shared/                          # Reusable across features
│   ├── shared.module.ts
│   ├── guards/
│   │   ├── auth.guard.ts
│   │   └── roles.guard.ts
│   ├── interceptors/
│   │   ├── logging.interceptor.ts
│   │   └── transform.interceptor.ts
│   ├── pipes/
│   │   └── validation.pipe.ts
│   ├── decorators/
│   │   ├── current-user.decorator.ts
│   │   └── roles.decorator.ts
│   ├── filters/
│   │   ├── http-exception.filter.ts
│   │   └── domain-exception.filter.ts
│   ├── utils/
│   │   ├── pagination.util.ts
│   │   └── date.util.ts
│   └── interfaces/
│       └── pagination.interface.ts
│
├── features/                        # Feature modules (business domains)
│   ├── users/
│   │   ├── users.module.ts
│   │   ├── controllers/
│   │   │   └── users.controller.ts
│   │   ├── services/
│   │   │   └── users.service.ts
│   │   ├── repositories/
│   │   │   └── users.repository.ts
│   │   ├── entities/
│   │   │   └── user.entity.ts
│   │   ├── dto/
│   │   │   ├── create-user.dto.ts
│   │   │   └── update-user.dto.ts
│   │   ├── interfaces/
│   │   │   └── user.interface.ts
│   │   └── tests/
│   │       └── users.service.spec.ts
│   │
│   ├── orders/
│   │   ├── orders.module.ts
│   │   ├── controllers/
│   │   │   └── orders.controller.ts
│   │   ├── services/
│   │   │   ├── orders.service.ts
│   │   │   └── order-processing.service.ts
│   │   ├── repositories/
│   │   │   └── orders.repository.ts
│   │   ├── entities/
│   │   │   ├── order.entity.ts
│   │   │   └── order-item.entity.ts
│   │   ├── dto/
│   │   │   ├── create-order.dto.ts
│   │   │   └── order-response.dto.ts
│   │   ├── events/
│   │   │   ├── order-placed.event.ts
│   │   │   └── order-event.handler.ts
│   │   └── tests/
│   │       └── orders.service.spec.ts
│   │
│   ├── products/
│   │   ├── products.module.ts
│   │   ├── controllers/
│   │   ├── services/
│   │   ├── repositories/
│   │   ├── entities/
│   │   └── dto/
│   │
│   └── payments/
│       ├── payments.module.ts
│       ├── controllers/
│       ├── services/
│       ├── repositories/
│       └── dto/
│
└── infrastructure/                  # External concerns
    ├── database/
    │   └── migrations/
    ├── cache/
    │   └── redis.service.ts
    ├── email/
    │   └── email.service.ts
    ├── storage/
    │   └── s3.service.ts
    └── external-apis/
        ├── stripe.service.ts
        └── sendgrid.service.ts
```

---

### **Method 1: Feature-Based Organization**

```typescript
// ========== FEATURE-BASED STRUCTURE ==========

// FEATURE MODULE: Users
// src/features/users/users.module.ts
@Module({
  imports: [
    TypeOrmModule.forFeature([User]),
    SharedModule,  // Import shared utilities
  ],
  controllers: [UsersController],
  providers: [
    UsersService,
    UsersRepository,
  ],
  exports: [UsersService],  // Export for other modules
})
export class UsersModule {}

// CONTROLLER: src/features/users/controllers/users.controller.ts
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  async findAll(): Promise<UserResponseDto[]> {
    return this.usersService.findAll();
  }

  @Post()
  async create(@Body() dto: CreateUserDto): Promise<UserResponseDto> {
    return this.usersService.create(dto);
  }
}

// SERVICE: src/features/users/services/users.service.ts
@Injectable()
export class UsersService {
  constructor(private readonly usersRepository: UsersRepository) {}

  async findAll(): Promise<User[]> {
    return this.usersRepository.findAll();
  }

  async create(dto: CreateUserDto): Promise<User> {
    // Business logic
    return this.usersRepository.save(dto);
  }
}

// REPOSITORY: src/features/users/repositories/users.repository.ts
@Injectable()
export class UsersRepository {
  constructor(
    @InjectRepository(User)
    private readonly repo: Repository<User>,
  ) {}

  async findAll(): Promise<User[]> {
    return this.repo.find();
  }

  async save(user: Partial<User>): Promise<User> {
    return this.repo.save(user);
  }
}

// ENTITY: src/features/users/entities/user.entity.ts
@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  email: string;

  @Column()
  name: string;
}

// DTO: src/features/users/dto/create-user.dto.ts
export class CreateUserDto {
  @IsEmail()
  email: string;

  @IsString()
  @MinLength(2)
  name: string;
}

// Benefits:
// - All user-related code in one place
// - Easy to find and navigate
// - Can delete entire feature by removing folder
// - Clear dependencies between features
```

---

### **Method 2: Core Module (Singleton Services)**

```typescript
// ========== CORE MODULE: SINGLETON SERVICES ==========

// src/core/core.module.ts
@Global()  // Make available everywhere without importing
@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      load: [appConfig, databaseConfig],
    }),
    DatabaseModule,
    LoggingModule,
  ],
  providers: [
    Logger,
    {
      provide: APP_FILTER,
      useClass: GlobalExceptionFilter,
    },
    {
      provide: APP_INTERCEPTOR,
      useClass: LoggingInterceptor,
    },
  ],
  exports: [Logger, DatabaseModule],
})
export class CoreModule {
  // Prevent re-importing core module
  constructor(@Optional() @SkipSelf() parentModule?: CoreModule) {
    if (parentModule) {
      throw new Error('CoreModule is already loaded. Import it in AppModule only.');
    }
  }
}

// src/core/config/app.config.ts
export default registerAs('app', () => ({
  port: parseInt(process.env.PORT, 10) || 3000,
  environment: process.env.NODE_ENV || 'development',
  apiPrefix: process.env.API_PREFIX || 'api',
}));

// src/core/database/database.module.ts
@Module({
  imports: [
    TypeOrmModule.forRootAsync({
      inject: [ConfigService],
      useFactory: (configService: ConfigService) => ({
        type: 'postgres',
        host: configService.get('database.host'),
        port: configService.get('database.port'),
        username: configService.get('database.username'),
        password: configService.get('database.password'),
        database: configService.get('database.name'),
        entities: [__dirname + '/../../**/*.entity{.ts,.js}'],
        synchronize: false,
        migrations: [__dirname + '/migrations/*{.ts,.js}'],
      }),
    }),
  ],
})
export class DatabaseModule {}

// src/core/logging/logger.service.ts
@Injectable()
export class LoggerService {
  private logger = new Logger('Application');

  log(message: string, context?: string): void {
    this.logger.log(message, context);
  }

  error(message: string, trace?: string, context?: string): void {
    this.logger.error(message, trace, context);
  }

  warn(message: string, context?: string): void {
    this.logger.warn(message, context);
  }
}
```

---

### **Method 3: Shared Module (Reusable Components)**

```typescript
// ========== SHARED MODULE: REUSABLE ACROSS FEATURES ==========

// src/shared/shared.module.ts
@Module({
  imports: [CommonModule],
  providers: [
    // Guards
    AuthGuard,
    RolesGuard,
    // Pipes
    ValidationPipe,
    // Utils
    PaginationUtil,
    DateUtil,
  ],
  exports: [
    CommonModule,
    // Export everything so features can use them
    AuthGuard,
    RolesGuard,
    ValidationPipe,
    PaginationUtil,
    DateUtil,
  ],
})
export class SharedModule {}

// src/shared/decorators/current-user.decorator.ts
export const CurrentUser = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return request.user;
  },
);

// src/shared/guards/roles.guard.ts
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<string[]>('roles', [
      context.getHandler(),
      context.getClass(),
    ]);

    if (!requiredRoles) {
      return true;
    }

    const request = context.switchToHttp().getRequest();
    const user = request.user;

    return requiredRoles.some(role => user.roles?.includes(role));
  }
}

// src/shared/interceptors/transform.interceptor.ts
@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, Response<T>> {
  intercept(context: ExecutionContext, next: CallHandler): Observable<Response<T>> {
    return next.handle().pipe(
      map(data => ({
        statusCode: context.switchToHttp().getResponse().statusCode,
        data,
        timestamp: new Date().toISOString(),
      })),
    );
  }
}

interface Response<T> {
  statusCode: number;
  data: T;
  timestamp: string;
}

// src/shared/utils/pagination.util.ts
export class PaginationUtil {
  static paginate<T>(
    items: T[],
    page: number,
    limit: number,
  ): PaginatedResult<T> {
    const startIndex = (page - 1) * limit;
    const endIndex = page * limit;

    return {
      data: items.slice(startIndex, endIndex),
      meta: {
        total: items.length,
        page,
        limit,
        totalPages: Math.ceil(items.length / limit),
      },
    };
  }
}

export interface PaginatedResult<T> {
  data: T[];
  meta: {
    total: number;
    page: number;
    limit: number;
    totalPages: number;
  };
}
```

---

### **Method 4: DDD-Inspired Structure (Advanced)**

```typescript
// ========== DOMAIN-DRIVEN DESIGN STRUCTURE ==========

// src/features/orders/
orders/
├── orders.module.ts
├── domain/                          # Domain layer (business logic)
│   ├── entities/
│   │   ├── order.entity.ts
│   │   └── order-item.entity.ts
│   ├── value-objects/
│   │   ├── money.vo.ts
│   │   └── order-status.vo.ts
│   ├── aggregates/
│   │   └── order.aggregate.ts
│   ├── services/
│   │   └── order-domain.service.ts  # Domain services
│   ├── events/
│   │   ├── order-placed.event.ts
│   │   └── order-cancelled.event.ts
│   └── interfaces/
│       └── order-repository.interface.ts  # Repository contract
│
├── application/                     # Application layer (use cases)
│   ├── commands/
│   │   ├── place-order.command.ts
│   │   └── cancel-order.command.ts
│   ├── handlers/
│   │   ├── place-order.handler.ts
│   │   └── cancel-order.handler.ts
│   ├── queries/
│   │   └── get-order.query.ts
│   ├── dto/
│   │   ├── place-order.dto.ts
│   │   └── order-response.dto.ts
│   └── services/
│       └── order-application.service.ts
│
├── infrastructure/                  # Infrastructure layer (technical details)
│   ├── repositories/
│   │   └── typeorm-order.repository.ts  # TypeORM implementation
│   ├── adapters/
│   │   ├── payment-gateway.adapter.ts
│   │   └── email.adapter.ts
│   └── persistence/
│       └── order.schema.ts
│
└── presentation/                    # Presentation layer (HTTP)
    ├── controllers/
    │   └── orders.controller.ts
    ├── dto/                         # HTTP-specific DTOs
    │   └── create-order-request.dto.ts
    └── mappers/
        └── order.mapper.ts

// DOMAIN ENTITY
// src/features/orders/domain/entities/order.entity.ts
export class Order {
  private constructor(
    public readonly id: string,
    public readonly customerId: string,
    private _items: OrderItem[],
    private _status: OrderStatus,
    private _total: Money,
  ) {}

  static create(customerId: string, items: OrderItem[]): Order {
    const order = new Order(
      uuidv4(),
      customerId,
      items,
      OrderStatus.PENDING,
      Money.zero(),
    );
    order.calculateTotal();
    return order;
  }

  place(): void {
    if (this._status !== OrderStatus.PENDING) {
      throw new DomainException('Can only place pending orders');
    }
    this._status = OrderStatus.PLACED;
  }

  cancel(): void {
    if (this._status === OrderStatus.SHIPPED) {
      throw new DomainException('Cannot cancel shipped order');
    }
    this._status = OrderStatus.CANCELLED;
  }

  private calculateTotal(): void {
    this._total = this._items.reduce(
      (sum, item) => sum.add(item.getTotal()),
      Money.zero(),
    );
  }

  get items(): ReadonlyArray<OrderItem> {
    return [...this._items];
  }

  get status(): OrderStatus {
    return this._status;
  }

  get total(): Money {
    return this._total;
  }
}

// REPOSITORY INTERFACE (in domain layer)
// src/features/orders/domain/interfaces/order-repository.interface.ts
export interface IOrderRepository {
  save(order: Order): Promise<Order>;
  findById(id: string): Promise<Order | null>;
  findByCustomerId(customerId: string): Promise<Order[]>;
}

// REPOSITORY IMPLEMENTATION (in infrastructure layer)
// src/features/orders/infrastructure/repositories/typeorm-order.repository.ts
@Injectable()
export class TypeOrmOrderRepository implements IOrderRepository {
  constructor(
    @InjectRepository(OrderEntity)
    private readonly repo: Repository<OrderEntity>,
  ) {}

  async save(order: Order): Promise<Order> {
    const entity = this.toEntity(order);
    const saved = await this.repo.save(entity);
    return this.toDomain(saved);
  }

  async findById(id: string): Promise<Order | null> {
    const entity = await this.repo.findOne({ where: { id } });
    return entity ? this.toDomain(entity) : null;
  }

  async findByCustomerId(customerId: string): Promise<Order[]> {
    const entities = await this.repo.find({ where: { customerId } });
    return entities.map(e => this.toDomain(e));
  }

  private toEntity(order: Order): OrderEntity {
    // Map domain model to database entity
  }

  private toDomain(entity: OrderEntity): Order {
    // Map database entity to domain model
  }
}

// APPLICATION SERVICE (use case)
// src/features/orders/application/services/order-application.service.ts
@Injectable()
export class OrderApplicationService {
  constructor(
    @Inject('IOrderRepository')
    private readonly orderRepository: IOrderRepository,
    private readonly eventBus: EventEmitter2,
  ) {}

  async placeOrder(dto: PlaceOrderDto): Promise<Order> {
    // Create domain entity
    const order = Order.create(dto.customerId, dto.items);

    // Place order (domain logic)
    order.place();

    // Persist
    await this.orderRepository.save(order);

    // Emit domain event
    this.eventBus.emit('order.placed', new OrderPlacedEvent(order.id));

    return order;
  }
}

// CONTROLLER (presentation layer)
// src/features/orders/presentation/controllers/orders.controller.ts
@Controller('orders')
export class OrdersController {
  constructor(
    private readonly orderService: OrderApplicationService,
  ) {}

  @Post()
  async placeOrder(@Body() dto: PlaceOrderDto): Promise<OrderResponseDto> {
    const order = await this.orderService.placeOrder(dto);
    return OrderMapper.toResponseDto(order);
  }
}
```

---

### **Project Structure Comparison:**

| Approach | When to Use | Benefits | Complexity |
|----------|-------------|----------|------------|
| **Feature-based** | Most projects | Clear boundaries, easy navigation | Low |
| **Layered** | Simple CRUD apps | Clear separation by technical concern | Low |
| **DDD-inspired** | Complex domains | Rich domain models, clear business logic | High |
| **Monorepo** | Microservices | Shared code, multiple apps | High |

---

### **Best Practices:**

```typescript
// ✅ GOOD: Feature-based structure
src/
├── features/
│   ├── users/
│   ├── orders/
│   └── products/

// ❌ BAD: Horizontal slicing
src/
├── controllers/
│   ├── users.controller.ts
│   ├── orders.controller.ts
│   └── products.controller.ts
├── services/
│   ├── users.service.ts
│   ├── orders.service.ts
│   └── products.service.ts

// ✅ GOOD: Clear module boundaries
@Module({
  imports: [SharedModule],  // Only what's needed
  exports: [UsersService],  // Explicit exports
})

// ❌ BAD: Tight coupling
// Importing everything from everywhere

// ✅ GOOD: Naming conventions
users.controller.ts
users.service.ts
users.repository.ts
user.entity.ts
create-user.dto.ts

// ❌ BAD: Inconsistent naming
userController.ts
UserService.ts
user-repo.ts
```

**Key Takeaway:** Structure large NestJS projects using **feature-based organization** (group by business domain: users/, orders/, products/), **layered architecture within features** (controllers/, services/, repositories/, entities/, dto/), **core module** (@Global, singleton services like config, database, logging), **shared module** (reusable guards, interceptors, pipes, decorators, utilities), and **clear separation** (domain logic in entities, use cases in services, HTTP in controllers). Use **vertical slices** (self-contained feature modules) over horizontal slices (all controllers together). Implement **DDD principles** for complex domains (domain/, application/, infrastructure/, presentation/ layers). Use **barrel exports** (index.ts) for clean imports, **naming conventions** (`.controller.ts`, `.service.ts`, `.module.ts`), and **dependency inversion** (interfaces in domain, implementations in infrastructure). Keep **modules focused** (single responsibility), **minimize coupling** (explicit imports/exports), and **maximize cohesion** (related code together). For microservices use **monorepo** (Nx, Turborepo) with shared libraries. Test structure with `jest`, organize tests alongside code (`users.service.spec.ts`), and document module dependencies.

</details>

<details>
<summary><strong>33. What is the recommended folder structure?</strong></summary>

**Answer:**

Recommended folder structure uses **feature-based organization** (group by domain, not by technical role), **consistent naming conventions** (`.controller.ts`, `.service.ts`, `.entity.ts`), **separation of concerns** (src/features/, src/core/, src/shared/), **co-located tests** (next to source files), and **clear hierarchy** (module → controller → service → repository → entity). Structure: `src/main.ts` (entry), `src/app.module.ts` (root), `src/core/` (singleton services), `src/shared/` (reusable utilities), `src/features/` (business domains), each feature contains controllers/, services/, repositories/, entities/, dto/, tests/. Avoid **deep nesting** (max 3-4 levels), use **barrel exports** (index.ts) sparingly, keep **related code together**, and follow **NestJS conventions**.

---

### **Recommended Folder Structure:**

```
nestjs-app/
├── src/
│   ├── main.ts                      # Application entry point
│   ├── app.module.ts                # Root module
│   │
│   ├── core/                        # Core module (singleton, global)
│   │   ├── core.module.ts
│   │   ├── config/
│   │   │   ├── config.module.ts
│   │   │   ├── app.config.ts
│   │   │   ├── database.config.ts
│   │   │   └── jwt.config.ts
│   │   ├── database/
│   │   │   ├── database.module.ts
│   │   │   ├── database.providers.ts
│   │   │   └── migrations/
│   │   │       ├── 001-create-users.ts
│   │   │       └── 002-create-orders.ts
│   │   ├── logging/
│   │   │   ├── logging.module.ts
│   │   │   └── logger.service.ts
│   │   └── health/
│   │       ├── health.module.ts
│   │       └── health.controller.ts
│   │
│   ├── shared/                      # Shared module (reusable)
│   │   ├── shared.module.ts
│   │   ├── guards/
│   │   │   ├── auth.guard.ts
│   │   │   ├── roles.guard.ts
│   │   │   └── throttle.guard.ts
│   │   ├── interceptors/
│   │   │   ├── logging.interceptor.ts
│   │   │   ├── timeout.interceptor.ts
│   │   │   └── transform.interceptor.ts
│   │   ├── pipes/
│   │   │   ├── validation.pipe.ts
│   │   │   └── parse-int.pipe.ts
│   │   ├── decorators/
│   │   │   ├── current-user.decorator.ts
│   │   │   ├── roles.decorator.ts
│   │   │   └── api-paginated-response.decorator.ts
│   │   ├── filters/
│   │   │   ├── http-exception.filter.ts
│   │   │   ├── domain-exception.filter.ts
│   │   │   └── all-exceptions.filter.ts
│   │   ├── dto/
│   │   │   ├── pagination.dto.ts
│   │   │   └── response.dto.ts
│   │   ├── interfaces/
│   │   │   ├── pagination.interface.ts
│   │   │   └── response.interface.ts
│   │   ├── utils/
│   │   │   ├── pagination.util.ts
│   │   │   ├── date.util.ts
│   │   │   └── string.util.ts
│   │   ├── constants/
│   │   │   ├── error-codes.constant.ts
│   │   │   └── roles.constant.ts
│   │   └── exceptions/
│   │       ├── domain.exception.ts
│   │       ├── business-rule.exception.ts
│   │       └── not-found.exception.ts
│   │
│   ├── features/                    # Feature modules (business domains)
│   │   │
│   │   ├── auth/
│   │   │   ├── auth.module.ts
│   │   │   ├── controllers/
│   │   │   │   └── auth.controller.ts
│   │   │   ├── services/
│   │   │   │   ├── auth.service.ts
│   │   │   │   └── jwt.service.ts
│   │   │   ├── strategies/
│   │   │   │   ├── jwt.strategy.ts
│   │   │   │   └── local.strategy.ts
│   │   │   ├── dto/
│   │   │   │   ├── login.dto.ts
│   │   │   │   ├── register.dto.ts
│   │   │   │   └── token-response.dto.ts
│   │   │   └── tests/
│   │   │       ├── auth.service.spec.ts
│   │   │       └── auth.controller.spec.ts
│   │   │
│   │   ├── users/
│   │   │   ├── users.module.ts
│   │   │   ├── controllers/
│   │   │   │   └── users.controller.ts
│   │   │   ├── services/
│   │   │   │   └── users.service.ts
│   │   │   ├── repositories/
│   │   │   │   ├── users.repository.ts
│   │   │   │   └── users.repository.interface.ts
│   │   │   ├── entities/
│   │   │   │   └── user.entity.ts
│   │   │   ├── dto/
│   │   │   │   ├── create-user.dto.ts
│   │   │   │   ├── update-user.dto.ts
│   │   │   │   └── user-response.dto.ts
│   │   │   ├── interfaces/
│   │   │   │   └── user.interface.ts
│   │   │   └── tests/
│   │   │       ├── users.service.spec.ts
│   │   │       ├── users.controller.spec.ts
│   │   │       └── users.repository.spec.ts
│   │   │
│   │   ├── orders/
│   │   │   ├── orders.module.ts
│   │   │   ├── controllers/
│   │   │   │   ├── orders.controller.ts
│   │   │   │   └── order-items.controller.ts
│   │   │   ├── services/
│   │   │   │   ├── orders.service.ts
│   │   │   │   ├── order-processing.service.ts
│   │   │   │   └── order-validation.service.ts
│   │   │   ├── repositories/
│   │   │   │   ├── orders.repository.ts
│   │   │   │   └── order-items.repository.ts
│   │   │   ├── entities/
│   │   │   │   ├── order.entity.ts
│   │   │   │   └── order-item.entity.ts
│   │   │   ├── dto/
│   │   │   │   ├── create-order.dto.ts
│   │   │   │   ├── update-order.dto.ts
│   │   │   │   ├── order-response.dto.ts
│   │   │   │   └── order-item.dto.ts
│   │   │   ├── enums/
│   │   │   │   └── order-status.enum.ts
│   │   │   ├── events/
│   │   │   │   ├── order-placed.event.ts
│   │   │   │   ├── order-cancelled.event.ts
│   │   │   │   └── order-events.handler.ts
│   │   │   └── tests/
│   │   │       └── orders.service.spec.ts
│   │   │
│   │   └── products/
│   │       ├── products.module.ts
│   │       ├── controllers/
│   │       │   ├── products.controller.ts
│   │       │   └── categories.controller.ts
│   │       ├── services/
│   │       │   ├── products.service.ts
│   │       │   └── categories.service.ts
│   │       ├── repositories/
│   │       │   ├── products.repository.ts
│   │       │   └── categories.repository.ts
│   │       ├── entities/
│   │       │   ├── product.entity.ts
│   │       │   └── category.entity.ts
│   │       ├── dto/
│   │       │   ├── create-product.dto.ts
│   │       │   └── product-response.dto.ts
│   │       └── tests/
│   │           └── products.service.spec.ts
│   │
│   └── infrastructure/               # Infrastructure concerns
│       ├── cache/
│       │   ├── redis.module.ts
│       │   └── redis.service.ts
│       ├── email/
│       │   ├── email.module.ts
│       │   ├── email.service.ts
│       │   └── templates/
│       ├── storage/
│       │   ├── storage.module.ts
│       │   └── s3.service.ts
│       ├── queue/
│       │   ├── queue.module.ts
│       │   └── processors/
│       └── external-apis/
│           ├── stripe/
│           │   ├── stripe.module.ts
│           │   └── stripe.service.ts
│           └── sendgrid/
│               ├── sendgrid.module.ts
│               └── sendgrid.service.ts
│
├── test/                            # E2E tests
│   ├── app.e2e-spec.ts
│   ├── users.e2e-spec.ts
│   └── jest-e2e.json
│
├── .env                             # Environment variables
├── .env.example
├── .gitignore
├── nest-cli.json                    # NestJS CLI config
├── package.json
├── tsconfig.json                    # TypeScript config
├── tsconfig.build.json
└── README.md
```

---

### **Method 1: Feature Module Structure (Detailed)**

```typescript
// ========== DETAILED FEATURE MODULE STRUCTURE ==========

// FEATURE: Users (Complete example)
users/
├── users.module.ts              # Module definition
├── index.ts                     # Barrel export (optional)
│
├── controllers/                 # HTTP layer
│   ├── users.controller.ts      # Main CRUD controller
│   └── users.controller.spec.ts # Controller tests
│
├── services/                    # Business logic layer
│   ├── users.service.ts         # Main business logic
│   ├── users.service.spec.ts    # Service tests
│   ├── user-validation.service.ts  # Specific concern
│   └── user-avatar.service.ts   # Another specific concern
│
├── repositories/                # Data access layer
│   ├── users.repository.ts      # TypeORM repository implementation
│   ├── users.repository.interface.ts  # Repository contract
│   └── users.repository.spec.ts # Repository tests
│
├── entities/                    # Database entities
│   ├── user.entity.ts           # Main entity
│   └── user-profile.entity.ts   # Related entity
│
├── dto/                         # Data Transfer Objects
│   ├── create-user.dto.ts       # Input DTO
│   ├── update-user.dto.ts       # Input DTO
│   ├── user-response.dto.ts     # Output DTO
│   └── user-list-response.dto.ts # Output DTO
│
├── interfaces/                  # TypeScript interfaces
│   ├── user.interface.ts        # Domain interface
│   └── user-filter.interface.ts # Filter interface
│
├── enums/                       # Enums
│   └── user-role.enum.ts        # Role enum
│
├── events/                      # Domain events (optional)
│   ├── user-created.event.ts
│   └── user-events.handler.ts
│
└── tests/                       # Additional tests
    ├── integration/
    │   └── users.integration.spec.ts
    └── fixtures/
        └── user.fixture.ts

// MODULE FILE
// users/users.module.ts
@Module({
  imports: [
    TypeOrmModule.forFeature([User, UserProfile]),
    SharedModule,  // Import shared utilities
  ],
  controllers: [UsersController],
  providers: [
    UsersService,
    UserValidationService,
    UserAvatarService,
    {
      provide: 'IUserRepository',
      useClass: UsersRepository,
    },
  ],
  exports: [
    UsersService,  // Export for other modules
  ],
})
export class UsersModule {}

// CONTROLLER FILE
// users/controllers/users.controller.ts
@Controller('users')
@UseGuards(AuthGuard)
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  async findAll(@Query() query: PaginationDto): Promise<UserListResponseDto> {
    return this.usersService.findAll(query);
  }

  @Get(':id')
  async findOne(@Param('id') id: string): Promise<UserResponseDto> {
    return this.usersService.findById(id);
  }

  @Post()
  async create(@Body() dto: CreateUserDto): Promise<UserResponseDto> {
    return this.usersService.create(dto);
  }

  @Patch(':id')
  async update(
    @Param('id') id: string,
    @Body() dto: UpdateUserDto,
  ): Promise<UserResponseDto> {
    return this.usersService.update(id, dto);
  }

  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  async remove(@Param('id') id: string): Promise<void> {
    return this.usersService.remove(id);
  }
}

// SERVICE FILE
// users/services/users.service.ts
@Injectable()
export class UsersService {
  constructor(
    @Inject('IUserRepository')
    private readonly usersRepository: IUserRepository,
    private readonly userValidationService: UserValidationService,
    private readonly eventEmitter: EventEmitter2,
  ) {}

  async findAll(query: PaginationDto): Promise<UserListResponseDto> {
    const users = await this.usersRepository.findAll(query);
    return { data: users, total: users.length };
  }

  async findById(id: string): Promise<User> {
    const user = await this.usersRepository.findById(id);
    
    if (!user) {
      throw new NotFoundException(`User with id ${id} not found`);
    }
    
    return user;
  }

  async create(dto: CreateUserDto): Promise<User> {
    await this.userValidationService.validateEmail(dto.email);
    
    const user = await this.usersRepository.save(dto);
    
    this.eventEmitter.emit('user.created', { userId: user.id });
    
    return user;
  }

  async update(id: string, dto: UpdateUserDto): Promise<User> {
    const user = await this.findById(id);
    Object.assign(user, dto);
    return this.usersRepository.save(user);
  }

  async remove(id: string): Promise<void> {
    await this.findById(id);  // Ensure exists
    await this.usersRepository.delete(id);
    this.eventEmitter.emit('user.deleted', { userId: id });
  }
}
```

---

### **Method 2: Core vs Shared vs Features**

```typescript
// ========== CORE VS SHARED VS FEATURES ==========

// CORE: Singleton, global services (loaded once)
core/
├── core.module.ts               # @Global() decorator
├── config/
│   ├── app.config.ts            # App-wide config
│   ├── database.config.ts       # Database config
│   └── jwt.config.ts            # JWT config
├── database/
│   ├── database.module.ts       # Database connection
│   └── migrations/              # Database migrations
└── logging/
    └── logger.service.ts        # Custom logger

// core/core.module.ts
@Global()  // Available everywhere without importing
@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      load: [appConfig, databaseConfig, jwtConfig],
    }),
    TypeOrmModule.forRootAsync({
      inject: [ConfigService],
      useFactory: (config: ConfigService) => ({
        type: 'postgres',
        host: config.get('database.host'),
        // ...
      }),
    }),
  ],
  providers: [LoggerService],
  exports: [LoggerService],
})
export class CoreModule {
  // Prevent re-importing
  constructor(@Optional() @SkipSelf() parentModule?: CoreModule) {
    if (parentModule) {
      throw new Error('CoreModule is already loaded. Import once in AppModule.');
    }
  }
}

// SHARED: Reusable utilities (imported where needed)
shared/
├── shared.module.ts
├── guards/
│   ├── auth.guard.ts            # Authentication
│   └── roles.guard.ts           # Authorization
├── interceptors/
│   ├── logging.interceptor.ts   # Request logging
│   └── timeout.interceptor.ts   # Request timeout
├── pipes/
│   └── validation.pipe.ts       # Custom validation
├── decorators/
│   ├── current-user.decorator.ts  # Extract current user
│   └── roles.decorator.ts       # Set required roles
├── dto/
│   └── pagination.dto.ts        # Pagination input
├── utils/
│   ├── pagination.util.ts       # Pagination helper
│   └── date.util.ts             # Date utilities
└── exceptions/
    ├── domain.exception.ts      # Base domain exception
    └── business-rule.exception.ts  # Business rule violation

// shared/shared.module.ts
@Module({
  providers: [
    AuthGuard,
    RolesGuard,
    LoggingInterceptor,
    ValidationPipe,
    PaginationUtil,
  ],
  exports: [
    AuthGuard,
    RolesGuard,
    LoggingInterceptor,
    ValidationPipe,
    PaginationUtil,
  ],
})
export class SharedModule {}

// FEATURES: Business domains (imported in AppModule)
features/
├── auth/
│   └── auth.module.ts           # Authentication feature
├── users/
│   └── users.module.ts          # User management
├── orders/
│   └── orders.module.ts         # Order management
└── products/
    └── products.module.ts       # Product catalog

// app.module.ts
@Module({
  imports: [
    CoreModule,      // Global services (once)
    SharedModule,    // Reusable utilities (once)
    AuthModule,      // Feature modules
    UsersModule,
    OrdersModule,
    ProductsModule,
  ],
})
export class AppModule {}
```

---

### **Method 3: Naming Conventions**

```typescript
// ========== NAMING CONVENTIONS ==========

// FILE NAMES (kebab-case with suffix)
users.controller.ts              // Controller
users.service.ts                 // Service
users.repository.ts              // Repository
user.entity.ts                   // Entity (singular)
create-user.dto.ts               // DTO
user-response.dto.ts             // Response DTO
user.interface.ts                // Interface
user-role.enum.ts                // Enum
users.module.ts                  // Module
users.controller.spec.ts         // Test file

// CLASS NAMES (PascalCase with suffix)
export class UsersController { }
export class UsersService { }
export class UsersRepository { }
export class User { }              // Entity (no suffix)
export class CreateUserDto { }
export class UserResponseDto { }
export interface IUserRepository { }  // Interface (I prefix)
export enum UserRole { }

// FOLDER NAMES (plural, kebab-case)
users/                           // Feature module
controllers/                     // Layer
services/
repositories/
dto/
entities/

// CONSTANTS (UPPER_SNAKE_CASE)
export const MAX_LOGIN_ATTEMPTS = 5;
export const DEFAULT_PAGE_SIZE = 20;

// ENUMS (PascalCase for enum, UPPER_SNAKE_CASE for values)
export enum UserRole {
  ADMIN = 'ADMIN',
  USER = 'USER',
  GUEST = 'GUEST',
}

export enum OrderStatus {
  PENDING = 'PENDING',
  CONFIRMED = 'CONFIRMED',
  SHIPPED = 'SHIPPED',
  CANCELLED = 'CANCELLED',
}

// EXAMPLE COMPLETE STRUCTURE
users/
├── users.module.ts              # UsersModule
├── controllers/
│   └── users.controller.ts      # UsersController
├── services/
│   └── users.service.ts         # UsersService
├── repositories/
│   ├── users.repository.ts      # UsersRepository
│   └── users.repository.interface.ts  # IUserRepository
├── entities/
│   └── user.entity.ts           # User
├── dto/
│   ├── create-user.dto.ts       # CreateUserDto
│   ├── update-user.dto.ts       # UpdateUserDto
│   └── user-response.dto.ts     # UserResponseDto
├── enums/
│   └── user-role.enum.ts        # UserRole
└── tests/
    └── users.service.spec.ts    # UsersService test
```

---

### **Method 4: Test Organization**

```typescript
// ========== TEST ORGANIZATION ==========

// OPTION 1: Co-located tests (recommended for unit tests)
users/
├── controllers/
│   ├── users.controller.ts
│   └── users.controller.spec.ts     # Next to source
├── services/
│   ├── users.service.ts
│   └── users.service.spec.ts        # Next to source
└── repositories/
    ├── users.repository.ts
    └── users.repository.spec.ts     # Next to source

// OPTION 2: Separate tests folder (for integration tests)
users/
├── controllers/
│   └── users.controller.ts
├── services/
│   └── users.service.ts
└── tests/
    ├── unit/
    │   ├── users.service.spec.ts
    │   └── users.controller.spec.ts
    ├── integration/
    │   └── users.integration.spec.ts
    └── fixtures/
        └── user.fixture.ts

// OPTION 3: Root-level test folder (for E2E tests)
project-root/
├── src/
│   └── features/
│       └── users/
└── test/                            # E2E tests
    ├── users.e2e-spec.ts
    ├── orders.e2e-spec.ts
    └── jest-e2e.json

// TEST FILE NAMING
users.service.spec.ts            # Unit test
users.integration.spec.ts        # Integration test
users.e2e-spec.ts                # E2E test
```

---

### **Folder Structure Best Practices:**

| Principle | Good | Bad |
|-----------|------|-----|
| **Organization** | Feature-based (users/) | Layer-based (controllers/) |
| **Depth** | Max 3-4 levels | 6+ levels deep |
| **Co-location** | Tests next to source | Tests far away |
| **Naming** | Consistent suffixes (.service.ts) | Inconsistent (service.ts, svc.ts) |
| **Exports** | Explicit exports in module | Export everything |
| **Coupling** | Minimal cross-feature imports | Features importing from each other |

---

### **Best Practices:**

```typescript
// ✅ GOOD: Feature-based structure
src/
├── features/
│   ├── users/
│   │   ├── controllers/
│   │   ├── services/
│   │   └── entities/
│   └── orders/
│       ├── controllers/
│       └── services/

// ❌ BAD: Layer-based structure
src/
├── controllers/
│   ├── users.controller.ts
│   └── orders.controller.ts
└── services/
    ├── users.service.ts
    └── orders.service.ts

// ✅ GOOD: Consistent naming
users.controller.ts
users.service.ts
user.entity.ts
create-user.dto.ts

// ❌ BAD: Inconsistent naming
UserController.ts
user-service.ts
User_Entity.ts
createUser.dto.ts

// ✅ GOOD: Co-located tests
users/
├── users.service.ts
└── users.service.spec.ts

// ❌ BAD: Separated tests
users/
└── users.service.ts
test/
└── users.service.spec.ts

// ✅ GOOD: Shallow hierarchy
features/users/services/users.service.ts  // 4 levels

// ❌ BAD: Deep hierarchy
features/domain/users/application/services/user-management/users.service.ts  // 7 levels
```

**Key Takeaway:** Recommended folder structure uses **feature-based organization** (src/features/ with users/, orders/, products/ by business domain), **consistent naming conventions** (kebab-case files, PascalCase classes, `.controller.ts`, `.service.ts`, `.entity.ts` suffixes), **clear separation** (src/core/ for singleton services, src/shared/ for reusable utilities, src/features/ for business logic, src/infrastructure/ for external concerns), **co-located tests** (unit tests next to source files, E2E tests in test/ folder), and **shallow hierarchy** (max 3-4 levels deep). Each feature module contains: controllers/ (HTTP layer), services/ (business logic), repositories/ (data access), entities/ (database models), dto/ (data transfer objects), interfaces/ (TypeScript interfaces), enums/ (enums), events/ (domain events), tests/ (additional tests). Use **core module** (@Global for config, database, logging), **shared module** (guards, interceptors, pipes, decorators, utilities), and **barrel exports** (index.ts) sparingly. Follow NestJS conventions, keep related code together, minimize coupling between features, and document module dependencies.

</details>

<details>
<summary><strong>34. How do you organize shared code?</strong></summary>

**Answer:**

Organize shared code in **shared module** (src/shared/), containing **guards** (auth, roles), **interceptors** (logging, transform, timeout), **pipes** (validation, transform), **decorators** (custom parameter decorators), **filters** (exception filters), **dto** (common DTOs like pagination), **interfaces** (shared TypeScript interfaces), **utils** (helper functions), **constants** (app-wide constants), and **exceptions** (custom exception classes). Export from SharedModule, import where needed. For **cross-cutting concerns** use **core module** (@Global for singletons like config, logger). Use **barrel exports** (index.ts) for clean imports. Keep **domain-specific** code in feature modules, only **truly reusable** code in shared. Avoid **circular dependencies** between shared and features.

---

### **Shared Code Organization:**

```
src/shared/
├── shared.module.ts             # Shared module definition
├── index.ts                     # Barrel export (optional)
│
├── guards/                      # Auth & authorization
│   ├── auth.guard.ts
│   ├── roles.guard.ts
│   └── throttle.guard.ts
│
├── interceptors/                # Request/response interceptors
│   ├── logging.interceptor.ts
│   ├── timeout.interceptor.ts
│   ├── transform.interceptor.ts
│   └── cache.interceptor.ts
│
├── pipes/                       # Validation & transformation
│   ├── validation.pipe.ts
│   ├── parse-int.pipe.ts
│   └── trim.pipe.ts
│
├── decorators/                  # Custom decorators
│   ├── current-user.decorator.ts
│   ├── roles.decorator.ts
│   ├── api-paginated-response.decorator.ts
│   └── public.decorator.ts
│
├── filters/                     # Exception filters
│   ├── http-exception.filter.ts
│   ├── domain-exception.filter.ts
│   ├── validation-exception.filter.ts
│   └── all-exceptions.filter.ts
│
├── dto/                         # Common DTOs
│   ├── pagination.dto.ts
│   ├── response.dto.ts
│   └── error-response.dto.ts
│
├── interfaces/                  # Shared interfaces
│   ├── pagination.interface.ts
│   ├── response.interface.ts
│   └── repository.interface.ts
│
├── utils/                       # Helper functions
│   ├── pagination.util.ts
│   ├── date.util.ts
│   ├── string.util.ts
│   └── validation.util.ts
│
├── constants/                   # App-wide constants
│   ├── error-codes.constant.ts
│   ├── roles.constant.ts
│   └── config.constant.ts
│
└── exceptions/                  # Custom exceptions
    ├── domain.exception.ts
    ├── business-rule.exception.ts
    ├── not-found.exception.ts
    └── validation.exception.ts
```

---

### **Method 1: Shared Module Setup**

```typescript
// ========== SHARED MODULE ORGANIZATION ==========

// src/shared/shared.module.ts
@Module({
  imports: [CommonModule],
  providers: [
    // Guards
    AuthGuard,
    RolesGuard,
    
    // Interceptors
    LoggingInterceptor,
    TimeoutInterceptor,
    TransformInterceptor,
    
    // Pipes
    ValidationPipe,
    
    // Utils
    PaginationUtil,
    DateUtil,
    StringUtil,
  ],
  exports: [
    CommonModule,
    
    // Export everything so feature modules can use them
    AuthGuard,
    RolesGuard,
    LoggingInterceptor,
    TimeoutInterceptor,
    TransformInterceptor,
    ValidationPipe,
    PaginationUtil,
    DateUtil,
    StringUtil,
  ],
})
export class SharedModule {}

// USAGE IN FEATURE MODULE
// src/features/users/users.module.ts
@Module({
  imports: [
    TypeOrmModule.forFeature([User]),
    SharedModule,  // Import shared utilities
  ],
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}

// USAGE IN CONTROLLER
// src/features/users/controllers/users.controller.ts
@Controller('users')
@UseGuards(AuthGuard, RolesGuard)  // From SharedModule
export class UsersController {
  @Get()
  @UseInterceptors(CacheInterceptor)  // From SharedModule
  @Roles('admin')  // From SharedModule decorator
  async findAll(
    @Query(ValidationPipe) query: PaginationDto,  // From SharedModule
  ): Promise<UserListResponseDto> {
    return this.usersService.findAll(query);
  }

  @Get('me')
  async getProfile(
    @CurrentUser() user: User,  // From SharedModule decorator
  ): Promise<UserResponseDto> {
    return this.usersService.findById(user.id);
  }
}
```

---

### **Method 2: Guards (Shared Authentication/Authorization)**

```typescript
// ========== SHARED GUARDS ==========

// src/shared/guards/auth.guard.ts
@Injectable()
export class AuthGuard implements CanActivate {
  constructor(
    private readonly jwtService: JwtService,
    private readonly reflector: Reflector,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    // Check if route is public
    const isPublic = this.reflector.getAllAndOverride<boolean>('isPublic', [
      context.getHandler(),
      context.getClass(),
    ]);

    if (isPublic) {
      return true;
    }

    // Get request
    const request = context.switchToHttp().getRequest();
    const token = this.extractTokenFromHeader(request);

    if (!token) {
      throw new UnauthorizedException('No token provided');
    }

    try {
      // Verify token
      const payload = await this.jwtService.verifyAsync(token);
      request.user = payload;
      return true;
    } catch {
      throw new UnauthorizedException('Invalid token');
    }
  }

  private extractTokenFromHeader(request: Request): string | undefined {
    const [type, token] = request.headers.authorization?.split(' ') ?? [];
    return type === 'Bearer' ? token : undefined;
  }
}

// src/shared/guards/roles.guard.ts
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private readonly reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // Get required roles from decorator
    const requiredRoles = this.reflector.getAllAndOverride<string[]>('roles', [
      context.getHandler(),
      context.getClass(),
    ]);

    if (!requiredRoles) {
      return true;
    }

    // Get user from request
    const request = context.switchToHttp().getRequest();
    const user = request.user;

    if (!user) {
      throw new UnauthorizedException('User not authenticated');
    }

    // Check if user has required roles
    const hasRole = requiredRoles.some(role => user.roles?.includes(role));

    if (!hasRole) {
      throw new ForbiddenException(
        `User does not have required roles: ${requiredRoles.join(', ')}`,
      );
    }

    return true;
  }
}

// USAGE
@Controller('admin')
@UseGuards(AuthGuard, RolesGuard)
@Roles('admin')
export class AdminController {
  // Only authenticated admins can access
}
```

---

### **Method 3: Decorators (Shared Custom Decorators)**

```typescript
// ========== SHARED DECORATORS ==========

// src/shared/decorators/current-user.decorator.ts
export const CurrentUser = createParamDecorator(
  (data: keyof User | undefined, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const user = request.user;

    // Return specific property if requested
    return data ? user?.[data] : user;
  },
);

// USAGE
@Get('profile')
async getProfile(@CurrentUser() user: User) {
  return user;
}

@Get('email')
async getEmail(@CurrentUser('email') email: string) {
  return { email };
}

// src/shared/decorators/roles.decorator.ts
export const Roles = (...roles: string[]) => SetMetadata('roles', roles);

// USAGE
@Roles('admin', 'moderator')
@Get('admin')
async adminOnly() {
  return 'Admin content';
}

// src/shared/decorators/public.decorator.ts
export const Public = () => SetMetadata('isPublic', true);

// USAGE
@Public()
@Post('login')
async login(@Body() dto: LoginDto) {
  return this.authService.login(dto);
}

// src/shared/decorators/api-paginated-response.decorator.ts
export const ApiPaginatedResponse = <T>(model: Type<T>) => {
  return applyDecorators(
    ApiOkResponse({
      schema: {
        allOf: [
          {
            properties: {
              data: {
                type: 'array',
                items: { $ref: getSchemaPath(model) },
              },
              meta: {
                type: 'object',
                properties: {
                  total: { type: 'number' },
                  page: { type: 'number' },
                  limit: { type: 'number' },
                  totalPages: { type: 'number' },
                },
              },
            },
          },
        ],
      },
    }),
  );
};

// USAGE
@ApiPaginatedResponse(UserResponseDto)
@Get()
async findAll(@Query() query: PaginationDto) {
  return this.usersService.findAll(query);
}
```

---

### **Method 4: Interceptors (Shared Request/Response Handling)**

```typescript
// ========== SHARED INTERCEPTORS ==========

// src/shared/interceptors/logging.interceptor.ts
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  constructor(private readonly logger: Logger) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { method, url } = request;
    const now = Date.now();

    this.logger.log(`→ ${method} ${url}`);

    return next.handle().pipe(
      tap(() => {
        const duration = Date.now() - now;
        this.logger.log(`← ${method} ${url} ${duration}ms`);
      }),
      catchError(error => {
        const duration = Date.now() - now;
        this.logger.error(`✖ ${method} ${url} ${duration}ms - ${error.message}`);
        throw error;
      }),
    );
  }
}

// src/shared/interceptors/transform.interceptor.ts
@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, ResponseDto<T>> {
  intercept(context: ExecutionContext, next: CallHandler): Observable<ResponseDto<T>> {
    return next.handle().pipe(
      map(data => ({
        statusCode: context.switchToHttp().getResponse().statusCode,
        data,
        timestamp: new Date().toISOString(),
      })),
    );
  }
}

interface ResponseDto<T> {
  statusCode: number;
  data: T;
  timestamp: string;
}

// src/shared/interceptors/timeout.interceptor.ts
@Injectable()
export class TimeoutInterceptor implements NestInterceptor {
  constructor(private readonly timeoutMs: number = 5000) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      timeout(this.timeoutMs),
      catchError(error => {
        if (error.name === 'TimeoutError') {
          throw new RequestTimeoutException('Request took too long');
        }
        throw error;
      }),
    );
  }
}

// GLOBAL USAGE (in main.ts)
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  app.useGlobalInterceptors(
    new LoggingInterceptor(new Logger()),
    new TransformInterceptor(),
    new TimeoutInterceptor(5000),
  );

  await app.listen(3000);
}

// CONTROLLER-LEVEL USAGE
@UseInterceptors(LoggingInterceptor)
@Controller('users')
export class UsersController {}

// ROUTE-LEVEL USAGE
@UseInterceptors(TransformInterceptor)
@Get()
async findAll() {}
```

---

### **Method 5: Utils (Shared Helper Functions)**

```typescript
// ========== SHARED UTILITIES ==========

// src/shared/utils/pagination.util.ts
@Injectable()
export class PaginationUtil {
  static DEFAULT_PAGE = 1;
  static DEFAULT_LIMIT = 20;
  static MAX_LIMIT = 100;

  static paginate<T>(
    items: T[],
    page: number = PaginationUtil.DEFAULT_PAGE,
    limit: number = PaginationUtil.DEFAULT_LIMIT,
  ): PaginatedResult<T> {
    // Validate and sanitize inputs
    page = Math.max(1, page);
    limit = Math.min(Math.max(1, limit), PaginationUtil.MAX_LIMIT);

    const startIndex = (page - 1) * limit;
    const endIndex = page * limit;
    const total = items.length;
    const totalPages = Math.ceil(total / limit);

    return {
      data: items.slice(startIndex, endIndex),
      meta: {
        total,
        page,
        limit,
        totalPages,
        hasNext: page < totalPages,
        hasPrev: page > 1,
      },
    };
  }

  static getPaginationParams(
    page?: number,
    limit?: number,
  ): { skip: number; take: number } {
    const validPage = Math.max(1, page || PaginationUtil.DEFAULT_PAGE);
    const validLimit = Math.min(
      Math.max(1, limit || PaginationUtil.DEFAULT_LIMIT),
      PaginationUtil.MAX_LIMIT,
    );

    return {
      skip: (validPage - 1) * validLimit,
      take: validLimit,
    };
  }
}

export interface PaginatedResult<T> {
  data: T[];
  meta: {
    total: number;
    page: number;
    limit: number;
    totalPages: number;
    hasNext: boolean;
    hasPrev: boolean;
  };
}

// src/shared/utils/date.util.ts
@Injectable()
export class DateUtil {
  static addDays(date: Date, days: number): Date {
    const result = new Date(date);
    result.setDate(result.getDate() + days);
    return result;
  }

  static isExpired(expiryDate: Date): boolean {
    return new Date() > expiryDate;
  }

  static formatDate(date: Date, format: string = 'YYYY-MM-DD'): string {
    // Format date according to format string
    const year = date.getFullYear();
    const month = String(date.getMonth() + 1).padStart(2, '0');
    const day = String(date.getDate()).padStart(2, '0');
    
    return format
      .replace('YYYY', String(year))
      .replace('MM', month)
      .replace('DD', day);
  }

  static parseDate(dateString: string): Date {
    return new Date(dateString);
  }
}

// src/shared/utils/string.util.ts
@Injectable()
export class StringUtil {
  static slugify(text: string): string {
    return text
      .toLowerCase()
      .replace(/[^\w\s-]/g, '')
      .replace(/\s+/g, '-')
      .replace(/-+/g, '-')
      .trim();
  }

  static truncate(text: string, length: number, suffix: string = '...'): string {
    if (text.length <= length) {
      return text;
    }
    return text.substring(0, length - suffix.length) + suffix;
  }

  static capitalize(text: string): string {
    return text.charAt(0).toUpperCase() + text.slice(1).toLowerCase();
  }

  static randomString(length: number): string {
    return Math.random().toString(36).substring(2, 2 + length);
  }
}

// USAGE IN SERVICE
@Injectable()
export class UsersService {
  async findAll(query: PaginationDto): Promise<PaginatedResult<User>> {
    const users = await this.usersRepository.findAll();
    return PaginationUtil.paginate(users, query.page, query.limit);
  }

  async createSlug(name: string): Promise<string> {
    return StringUtil.slugify(name);
  }
}
```

---

### **Shared Code Organization Best Practices:**

| Category | What Goes Here | What Doesn't |
|----------|----------------|--------------|
| **Guards** | Auth, roles, throttling | Feature-specific authorization |
| **Interceptors** | Logging, timeout, transform | Feature-specific transformations |
| **Pipes** | Generic validation, parsing | Domain-specific validation |
| **Decorators** | CurrentUser, Roles, Public | Feature-specific decorators |
| **Utils** | Pagination, date, string helpers | Domain-specific utilities |
| **DTOs** | Pagination, base response | Feature-specific DTOs |
| **Exceptions** | Base exceptions, common errors | Feature-specific business errors |

---

### **Best Practices:**

```typescript
// ✅ GOOD: Truly reusable code in shared
shared/
├── guards/auth.guard.ts         # Used by all features
├── dto/pagination.dto.ts        # Used everywhere
└── utils/date.util.ts           # Generic helper

// ❌ BAD: Feature-specific code in shared
shared/
├── guards/can-edit-user.guard.ts  # User-specific!
├── dto/user-filter.dto.ts         # User-specific!
└── utils/user-validator.util.ts   # User-specific!

// ✅ GOOD: Export from SharedModule
@Module({
  providers: [AuthGuard, RolesGuard],
  exports: [AuthGuard, RolesGuard],  // Explicit exports
})
export class SharedModule {}

// ❌ BAD: No exports
@Module({
  providers: [AuthGuard, RolesGuard],
  // Not exported - can't be used!
})

// ✅ GOOD: Import SharedModule where needed
@Module({
  imports: [SharedModule],  // Get access to shared utilities
  controllers: [UsersController],
})
export class UsersModule {}

// ❌ BAD: Import individual shared components
@Module({
  imports: [],
  controllers: [UsersController],
  providers: [AuthGuard],  // Importing directly - wrong!
})
```

**Key Takeaway:** Organize shared code in **src/shared/** module containing **guards** (AuthGuard, RolesGuard for authentication/authorization), **interceptors** (LoggingInterceptor, TimeoutInterceptor, TransformInterceptor for request/response handling), **pipes** (ValidationPipe for input validation), **decorators** (@CurrentUser, @Roles, @Public for custom parameter decorators), **filters** (exception filters for error handling), **dto** (PaginationDto, ResponseDto for common data structures), **interfaces** (shared TypeScript interfaces), **utils** (PaginationUtil, DateUtil, StringUtil for helper functions), **constants** (ERROR_CODES, ROLES for app-wide constants), and **exceptions** (DomainException, BusinessRuleException base classes). Export everything from SharedModule, import SharedModule in feature modules that need shared utilities. Keep **truly reusable** code in shared (used by 3+ features), move **feature-specific** code to feature modules. Use **barrel exports** (index.ts) for clean imports. Avoid **circular dependencies** between shared and features. For **singleton services** (config, database, logger) use **core module** with @Global decorator. Test shared utilities independently, document public APIs, and version shared code carefully in monorepos.

</details>

<details>
<summary><strong>35. Should you use barrel exports (index.ts)?</strong></summary>

**Answer:**

Use barrel exports **sparingly and strategically**. They provide **clean imports** (import from 'users/' instead of 'users/services/users.service') and **encapsulation** (control what's public), but cause **circular dependency risks**, **build performance issues** (bundlers load entire module), **IDE slowdowns** (autocomplete loads everything), and **testing complications**. Use for **public APIs** (feature module exports, shared utilities) and **complex folder structures** (multiple related exports). **Avoid** for **internal module files**, **large modules** (many services/components), and **frequently changing code**. Best practice: use **direct imports** by default, add barrel exports only when API is stable and benefits are clear.

---

### **Barrel Exports Overview:**

A barrel export is an `index.ts` file that re-exports items from other files to simplify imports:

```typescript
// users/index.ts (barrel export)
export * from './users.module';
export * from './dto/create-user.dto';
export * from './dto/user-response.dto';
export * from './entities/user.entity';

// ALLOWS THIS:
import { UsersModule, CreateUserDto, UserResponseDto } from './users';

// INSTEAD OF THIS:
import { UsersModule } from './users/users.module';
import { CreateUserDto } from './users/dto/create-user.dto';
import { UserResponseDto } from './users/dto/user-response.dto';
```

---

### **Method 1: When to Use Barrel Exports (Good Cases)**

```typescript
// ========== GOOD USE CASE 1: PUBLIC API OF SHARED MODULE ==========

// src/shared/index.ts - Export public utilities
export * from './guards/auth.guard';
export * from './guards/roles.guard';
export * from './decorators/current-user.decorator';
export * from './decorators/roles.decorator';
export * from './dto/pagination.dto';
export * from './dto/response.dto';
export * from './utils/pagination.util';
export * from './utils/date.util';
export * from './constants/roles.constant';
export * from './exceptions/domain.exception';

// USAGE - Clean import from shared module
import { 
  AuthGuard, 
  RolesGuard, 
  CurrentUser, 
  PaginationDto,
  PaginationUtil 
} from '@shared';  // Using path alias

// BENEFIT: Single import path for all shared utilities

// ========== GOOD USE CASE 2: FEATURE MODULE PUBLIC API ==========

// src/features/users/index.ts - Export what other modules need
export { UsersModule } from './users.module';
export { UsersService } from './services/users.service';
export { User } from './entities/user.entity';
export { UserResponseDto } from './dto/user-response.dto';
export { CreateUserDto } from './dto/create-user.dto';
// Note: DO NOT export internal services, repositories, controllers

// USAGE IN ANOTHER MODULE
// src/features/orders/services/orders.service.ts
import { UsersService, User } from '@features/users';  // Clean import

@Injectable()
export class OrdersService {
  constructor(private readonly usersService: UsersService) {}

  async createOrder(userId: string, dto: CreateOrderDto) {
    const user = await this.usersService.findById(userId);
    // ...
  }
}

// BENEFIT: Clear public API, controlled access

// ========== GOOD USE CASE 3: COMPLEX DTO STRUCTURE ==========

// src/features/orders/dto/index.ts
export * from './create-order.dto';
export * from './update-order.dto';
export * from './order-response.dto';
export * from './order-item.dto';
export * from './order-filter.dto';
export * from './order-summary.dto';

// USAGE
import {
  CreateOrderDto,
  UpdateOrderDto,
  OrderResponseDto,
  OrderItemDto,
} from './dto';  // All DTOs from one path

// BENEFIT: Simplified imports for many related DTOs

// ========== GOOD USE CASE 4: DOMAIN MODELS ==========

// src/features/orders/entities/index.ts
export * from './order.entity';
export * from './order-item.entity';
export * from './order-status.enum';

// USAGE
import { Order, OrderItem, OrderStatus } from './entities';

// BENEFIT: Group related domain models
```

---

### **Method 2: When to Avoid Barrel Exports (Bad Cases)**

```typescript
// ========== BAD USE CASE 1: INTERNAL MODULE FILES ==========

// ❌ BAD: src/features/users/index.ts - Exporting everything
export * from './controllers/users.controller';  // Internal!
export * from './services/users.service';
export * from './services/user-validation.service';  // Internal!
export * from './repositories/users.repository';  // Internal!
export * from './entities/user.entity';
export * from './dto/create-user.dto';
// This exposes internal implementation details!

// ✅ GOOD: Only export public API
export { UsersModule } from './users.module';
export { UsersService } from './services/users.service';  // Only if used externally
export { User } from './entities/user.entity';
export { UserResponseDto } from './dto/user-response.dto';
// Internal services, repositories, controllers NOT exported

// ========== BAD USE CASE 2: CAUSES CIRCULAR DEPENDENCIES ==========

// ❌ BAD: Barrel export causing circular dependency
// users/index.ts
export * from './users.service';
export * from './users.module';

// users/users.service.ts
import { OrdersService } from '../orders';  // Imports from orders/index.ts

// orders/index.ts
export * from './orders.service';
export * from './orders.module';

// orders/orders.service.ts
import { UsersService } from '../users';  // Imports from users/index.ts
// CIRCULAR DEPENDENCY! users → orders → users

// ✅ GOOD: Direct imports to avoid circular dependencies
// orders/orders.service.ts
import { UsersService } from '../users/services/users.service';  // Direct path

// ========== BAD USE CASE 3: PERFORMANCE ISSUES ==========

// ❌ BAD: Large barrel export file
// shared/index.ts - Exports 50+ items
export * from './guards/auth.guard';
export * from './guards/roles.guard';
export * from './guards/throttle.guard';
// ... 47 more exports
export * from './utils/complex-calculation.util';  // Heavy module

// users/users.controller.ts
import { AuthGuard } from '@shared';  // Loads ALL 50+ exports!

// PROBLEM: Bundlers load entire shared module even if you only need AuthGuard
// This slows down builds, increases bundle size, and slows IDE autocomplete

// ✅ GOOD: Direct imports for specific needs
import { AuthGuard } from '@shared/guards/auth.guard';  // Only loads what's needed

// ========== BAD USE CASE 4: TESTING COMPLICATIONS ==========

// ❌ BAD: Barrel export complicates mocking
// users/index.ts
export * from './users.service';
export * from './users.repository';

// users.service.spec.ts
import { UsersService, UsersRepository } from './index';  // Loads both + more

// PROBLEM: Cannot easily mock partial imports, loads unnecessary code

// ✅ GOOD: Direct imports for testing
import { UsersService } from './users.service';
import { UsersRepository } from './users.repository';
// Clear, explicit, easy to mock
```

---

### **Method 3: Selective Barrel Exports (Recommended Approach)**

```typescript
// ========== RECOMMENDED: SELECTIVE BARREL EXPORTS ==========

// src/features/users/index.ts - Only public API
export { UsersModule } from './users.module';

// Export services ONLY if used by other modules
export { UsersService } from './services/users.service';

// Export entities ONLY if referenced externally
export { User } from './entities/user.entity';

// Export DTOs for external consumption
export type { CreateUserDto } from './dto/create-user.dto';
export type { UpdateUserDto } from './dto/update-user.dto';
export type { UserResponseDto } from './dto/user-response.dto';

// DO NOT export:
// - Controllers (internal to module)
// - Repositories (internal implementation)
// - Internal services (only used within module)
// - Guards/interceptors (unless shared across modules)

// tsconfig.json - Configure path aliases for clean imports
{
  "compilerOptions": {
    "paths": {
      "@features/*": ["src/features/*"],
      "@shared": ["src/shared/index.ts"],
      "@core": ["src/core/index.ts"]
    }
  }
}

// USAGE
import { UsersModule, UsersService, User } from '@features/users';
import { AuthGuard, RolesGuard } from '@shared';

// ========== SPLIT BARREL EXPORTS BY CATEGORY ==========

// src/shared/guards/index.ts - Guards only
export * from './auth.guard';
export * from './roles.guard';
export * from './throttle.guard';

// src/shared/decorators/index.ts - Decorators only
export * from './current-user.decorator';
export * from './roles.decorator';
export * from './public.decorator';

// src/shared/utils/index.ts - Utils only
export * from './pagination.util';
export * from './date.util';
export * from './string.util';

// src/shared/index.ts - Main barrel (lightweight)
export * from './guards';
export * from './decorators';
export * from './utils';
export * from './dto/pagination.dto';
export * from './dto/response.dto';

// BENEFIT: Smaller, focused barrel exports reduce loading overhead

// USAGE - Can import from specific barrel or main barrel
import { AuthGuard } from '@shared/guards';  // Specific
import { AuthGuard } from '@shared';          // General
```

---

### **Method 4: Alternatives to Barrel Exports**

```typescript
// ========== ALTERNATIVE 1: DIRECT IMPORTS WITH PATH ALIASES ==========

// tsconfig.json
{
  "compilerOptions": {
    "paths": {
      "@features/users/*": ["src/features/users/*"],
      "@shared/guards/*": ["src/shared/guards/*"]
    }
  }
}

// No barrel exports needed
import { UsersService } from '@features/users/services/users.service';
import { AuthGuard } from '@shared/guards/auth.guard';

// BENEFIT: No circular dependencies, explicit imports, better performance

// ========== ALTERNATIVE 2: EXPLICIT NAMED EXPORTS IN MODULE ==========

// users/users.module.ts
@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UsersController],
  providers: [UsersService, UsersRepository],
  exports: [UsersService],  // Explicit in module
})
export class UsersModule {}

// orders/orders.module.ts
@Module({
  imports: [UsersModule],  // Import entire module
  controllers: [OrdersController],
  providers: [OrdersService],
})
export class OrdersModule {}

// orders/orders.service.ts
import { UsersService } from '../users/services/users.service';  // Direct import

// BENEFIT: Module system handles dependencies, no barrel exports needed

// ========== ALTERNATIVE 3: PUBLIC API PATTERN ==========

// users/public-api.ts (instead of index.ts)
export { UsersModule } from './users.module';
export { UsersService } from './services/users.service';
export { User } from './entities/user.entity';

// USAGE
import { UsersService } from '@features/users/public-api';

// BENEFIT: Explicit naming makes intent clear (this is the public API)
```

---

### **Method 5: Fixing Circular Dependencies from Barrel Exports**

```typescript
// ========== FIXING CIRCULAR DEPENDENCIES ==========

// PROBLEM: Circular dependency
// users/index.ts
export * from './users.service';

// orders/index.ts
export * from './orders.service';

// users/users.service.ts
import { OrdersService } from '../orders';  // Via barrel

// orders/orders.service.ts
import { UsersService } from '../users';  // Via barrel
// CIRCULAR!

// ========== FIX 1: DIRECT IMPORTS ==========

// users/users.service.ts
import { OrdersService } from '../orders/services/orders.service';  // Direct path

// orders/orders.service.ts
import { UsersService } from '../users/services/users.service';  // Direct path

// ========== FIX 2: DEPENDENCY INVERSION ==========

// shared/interfaces/user-service.interface.ts
export interface IUserService {
  findById(id: string): Promise<User>;
}

// shared/interfaces/order-service.interface.ts
export interface IOrderService {
  findByUserId(userId: string): Promise<Order[]>;
}

// users/users.service.ts
@Injectable()
export class UsersService implements IUserService {
  constructor(
    @Inject('IOrderService')
    private readonly ordersService: IOrderService,  // Interface, not concrete
  ) {}
}

// orders/orders.service.ts
@Injectable()
export class OrdersService implements IOrderService {
  constructor(
    @Inject('IUserService')
    private readonly usersService: IUserService,  // Interface, not concrete
  ) {}
}

// BENEFIT: Break circular dependency using interfaces

// ========== FIX 3: EVENT-DRIVEN (BEST FOR COMPLEX CASES) ==========

// users/users.service.ts
@Injectable()
export class UsersService {
  constructor(private readonly eventEmitter: EventEmitter2) {}

  async deleteUser(id: string) {
    await this.usersRepository.delete(id);
    this.eventEmitter.emit('user.deleted', { userId: id });  // Fire event
  }
}

// orders/orders.service.ts
@Injectable()
export class OrdersService {
  @OnEvent('user.deleted')
  async handleUserDeleted(payload: { userId: string }) {
    // Cancel orders for deleted user
    await this.ordersRepository.cancelByUserId(payload.userId);
  }
}

// BENEFIT: No direct dependency, decoupled, scalable
```

---

### **Barrel Exports Decision Matrix:**

| Scenario | Use Barrel Export? | Reason |
|----------|-------------------|--------|
| Public API of feature module | ✅ Yes | Clean imports, encapsulation |
| Shared utilities (guards, utils) | ✅ Yes | Convenience, grouped exports |
| Complex DTO structures (10+ files) | ✅ Yes | Simplify imports |
| Internal module files | ❌ No | Exposes implementation details |
| Large modules (50+ exports) | ❌ No | Performance issues |
| Risk of circular dependencies | ❌ No | Hard to debug |
| Testing files | ❌ No | Complicates mocking |
| Frequently changing code | ❌ No | Maintenance burden |

---

### **Best Practices:**

```typescript
// ✅ GOOD: Selective, explicit barrel exports
// users/index.ts
export { UsersModule } from './users.module';
export { UsersService } from './services/users.service';
export type { UserResponseDto } from './dto/user-response.dto';

// ❌ BAD: Export everything with wildcard
// users/index.ts
export * from './controllers/users.controller';
export * from './services/users.service';
export * from './services/user-validation.service';
export * from './repositories/users.repository';
// Too much exposed!

// ✅ GOOD: Split barrel exports by category
shared/
├── guards/index.ts       # Only guards
├── decorators/index.ts   # Only decorators
└── index.ts              # Main barrel

// ❌ BAD: One massive barrel export
shared/
└── index.ts              # All 100+ exports in one file

// ✅ GOOD: Use direct imports by default
import { UsersService } from './users/services/users.service';

// ❌ BAD: Always use barrel exports
import { UsersService } from './users';

// ✅ GOOD: Document exported API
// users/index.ts
/**
 * Public API for Users module
 * Only export what's needed by other modules
 */
export { UsersModule } from './users.module';
export { UsersService } from './services/users.service';

// ❌ BAD: No documentation
export * from './users.module';
export * from './services/users.service';
```

**Key Takeaway:** Use barrel exports **sparingly and strategically** - they're **beneficial** for **public APIs** (feature module exports like UsersModule, UsersService), **shared utilities** (guards, decorators, utils with 10-20 exports), and **complex DTO structures** (10+ related DTOs), providing **clean imports** (import from 'users/' instead of deep paths) and **encapsulation** (control what's public). However, **avoid barrel exports** for **internal module files** (controllers, repositories are implementation details), **large modules** (50+ exports cause build slowdowns), **circular dependency risks** (barrel exports mask dependency chains), and **frequently changing code** (maintenance burden). **Best practice**: use **direct imports by default** (import { UsersService } from './users/services/users.service'), add **selective barrel exports** only for stable public APIs (export { UsersModule, UsersService } not export *), **split barrel exports by category** (guards/index.ts, decorators/index.ts), configure **path aliases** in tsconfig.json (@features/*, @shared/*), and use **type-only exports** for DTOs (export type { CreateUserDto }). Fix circular dependencies with **direct imports**, **dependency inversion** (interfaces), or **event-driven architecture** (EventEmitter). Monitor **bundle size** and **build performance**, document exported APIs clearly, and prioritize **explicit over implicit** exports.

</details>

<details>
<summary><strong>36. What are Provider scopes (DEFAULT, REQUEST, TRANSIENT)?</strong></summary>

**Answer:**

NestJS providers have three scopes: **DEFAULT** (singleton, shared across application, created once, best performance), **REQUEST** (new instance per request, request-specific data, multi-tenancy, higher overhead), and **TRANSIENT** (new instance per injection, not shared, highest overhead). Set scope with `@Injectable({ scope: Scope.REQUEST })` or in module providers array. **DEFAULT** is recommended for **stateless services** (99% of cases), **REQUEST** for **request-scoped context** (current user, tenant ID, correlation ID), **TRANSIENT** for **truly disposable** instances (rare). **Performance**: DEFAULT (fast, singleton) > REQUEST (slower, per-request overhead) > TRANSIENT (slowest, per-injection overhead). Scopes propagate up the dependency chain (if OrdersService is REQUEST-scoped, all its dependents become REQUEST-scoped).

---

### **Provider Scopes Overview:**

```typescript
// Three provider scopes in NestJS
export enum Scope {
  DEFAULT,    // Singleton - shared across entire application
  REQUEST,    // New instance created for each HTTP request
  TRANSIENT,  // New instance created for each injection point
}

// Setting provider scope
@Injectable({ scope: Scope.REQUEST })
export class UsersService {
  // New instance for each HTTP request
}

// Or in module providers array
@Module({
  providers: [
    {
      provide: UsersService,
      useClass: UsersService,
      scope: Scope.REQUEST,
    },
  ],
})
export class UsersModule {}
```

---

### **Method 1: DEFAULT Scope (Singleton - Recommended)**

```typescript
// ========== DEFAULT SCOPE (SINGLETON) ==========

// No scope specified = DEFAULT scope
@Injectable()
export class UsersService {
  private readonly users: Map<string, User> = new Map();

  constructor(
    private readonly usersRepository: UsersRepository,
    private readonly logger: Logger,
  ) {
    this.logger.log('UsersService created');  // Called ONCE at startup
  }

  async findById(id: string): Promise<User> {
    return this.usersRepository.findById(id);
  }
}

// CHARACTERISTICS:
// - Created ONCE when application starts
// - Same instance shared across ALL requests
// - Stored in memory for application lifetime
// - Best performance (no overhead)
// - Cannot access request-specific data (req, res)

// LIFECYCLE:
// Application Start → Create UsersService (once) → Inject everywhere → Reuse forever

// USAGE - Recommended for stateless services
@Injectable()  // DEFAULT scope by default
export class OrdersService {
  constructor(
    private readonly ordersRepository: OrdersRepository,
    private readonly usersService: UsersService,  // Same singleton instance
  ) {}

  async create(dto: CreateOrderDto): Promise<Order> {
    // Stateless logic - no request-specific data
    return this.ordersRepository.save(dto);
  }
}

// ========== WHEN TO USE DEFAULT SCOPE ==========

// ✅ USE DEFAULT for:
// 1. Stateless services (no request-specific state)
@Injectable()
export class ProductsService {
  async findAll(): Promise<Product[]> {
    return this.productsRepository.findAll();
  }
}

// 2. Repositories (data access)
@Injectable()
export class UsersRepository {
  constructor(
    @InjectRepository(User)
    private readonly repository: Repository<User>,
  ) {}

  async findById(id: string): Promise<User> {
    return this.repository.findOne({ where: { id } });
  }
}

// 3. Utilities (helper functions)
@Injectable()
export class DateUtil {
  formatDate(date: Date): string {
    return date.toISOString();
  }
}

// 4. Configuration services
@Injectable()
export class ConfigService {
  get databaseUrl(): string {
    return process.env.DATABASE_URL;
  }
}

// 5. Loggers (unless request-specific)
@Injectable()
export class Logger {
  log(message: string) {
    console.log(`[LOG] ${message}`);
  }
}

// PERFORMANCE: ⚡⚡⚡ BEST (no overhead, singleton)
```

---

### **Method 2: REQUEST Scope (Per-Request)**

```typescript
// ========== REQUEST SCOPE (PER-REQUEST) ==========

@Injectable({ scope: Scope.REQUEST })
export class RequestContextService {
  private userId: string;
  private tenantId: string;
  private correlationId: string;

  constructor(
    @Inject(REQUEST) private readonly request: Request,
  ) {
    // Access request object
    this.userId = request.user?.id;
    this.tenantId = request.headers['x-tenant-id'];
    this.correlationId = request.headers['x-correlation-id'] || uuidv4();
  }

  getUserId(): string {
    return this.userId;
  }

  getTenantId(): string {
    return this.tenantId;
  }

  getCorrelationId(): string {
    return this.correlationId;
  }
}

// CHARACTERISTICS:
// - New instance created for EACH HTTP request
// - Can access request object (req, res)
// - Destroyed after request completes
// - Higher overhead than DEFAULT
// - Scope propagates up dependency chain

// LIFECYCLE:
// Request → Create RequestContextService → Inject into controllers/services → Destroy after response

// USAGE - Access current user/tenant in services
@Injectable({ scope: Scope.REQUEST })
export class AuditService {
  constructor(
    private readonly requestContext: RequestContextService,  // Also REQUEST-scoped
    @Inject(REQUEST) private readonly request: Request,
  ) {}

  async logAction(action: string, resource: string): Promise<void> {
    const userId = this.requestContext.getUserId();
    const tenantId = this.requestContext.getTenantId();
    const correlationId = this.requestContext.getCorrelationId();

    await this.auditRepository.save({
      userId,
      tenantId,
      correlationId,
      action,
      resource,
      timestamp: new Date(),
      ip: this.request.ip,
      userAgent: this.request.headers['user-agent'],
    });
  }
}

// ========== WHEN TO USE REQUEST SCOPE ==========

// ✅ USE REQUEST for:
// 1. Request context (user, tenant, correlation ID)
@Injectable({ scope: Scope.REQUEST })
export class RequestContext {
  constructor(@Inject(REQUEST) private readonly request: Request) {}

  get currentUser(): User {
    return this.request.user;
  }

  get tenantId(): string {
    return this.request.headers['x-tenant-id'];
  }
}

// 2. Multi-tenancy (tenant-specific data)
@Injectable({ scope: Scope.REQUEST })
export class TenantService {
  private tenantId: string;

  constructor(@Inject(REQUEST) request: Request) {
    this.tenantId = request.headers['x-tenant-id'];
  }

  async getTenantConfig(): Promise<TenantConfig> {
    return this.configRepository.findByTenantId(this.tenantId);
  }
}

// 3. Request-specific logging (with correlation ID)
@Injectable({ scope: Scope.REQUEST })
export class RequestLogger {
  private correlationId: string;

  constructor(@Inject(REQUEST) request: Request) {
    this.correlationId = request.headers['x-correlation-id'] || uuidv4();
  }

  log(message: string) {
    console.log(`[${this.correlationId}] ${message}`);
  }
}

// 4. Audit trails (who did what)
@Injectable({ scope: Scope.REQUEST })
export class AuditService {
  constructor(
    @Inject(REQUEST) private readonly request: Request,
    private readonly auditRepository: AuditRepository,
  ) {}

  async logAccess(resource: string): Promise<void> {
    await this.auditRepository.save({
      userId: this.request.user.id,
      resource,
      timestamp: new Date(),
    });
  }
}

// ❌ DO NOT USE REQUEST for:
// - Stateless services (use DEFAULT instead)
// - High-traffic services (performance impact)
// - Services that don't need request data

// PERFORMANCE: ⚡⚡ MODERATE (per-request overhead)

// ========== SCOPE PROPAGATION ==========

// If UsersService is REQUEST-scoped, all its dependents become REQUEST-scoped
@Injectable({ scope: Scope.REQUEST })
export class UsersService {
  // REQUEST-scoped
}

@Injectable()  // DEFAULT scope declared
export class OrdersService {
  constructor(
    private readonly usersService: UsersService,  // REQUEST-scoped dependency
  ) {}
  // OrdersService becomes REQUEST-scoped due to dependency!
}

@Controller('orders')
export class OrdersController {
  constructor(
    private readonly ordersService: OrdersService,  // Now REQUEST-scoped
  ) {}
  // OrdersController becomes REQUEST-scoped too!
}

// IMPACT: Entire dependency chain becomes REQUEST-scoped
// Be careful: One REQUEST-scoped service can affect many others
```

---

### **Method 3: TRANSIENT Scope (Per-Injection)**

```typescript
// ========== TRANSIENT SCOPE (PER-INJECTION) ==========

@Injectable({ scope: Scope.TRANSIENT })
export class UniqueIdGenerator {
  private readonly id: string = uuidv4();

  constructor() {
    console.log('UniqueIdGenerator created:', this.id);
  }

  getId(): string {
    return this.id;
  }
}

// CHARACTERISTICS:
// - New instance created for EACH INJECTION POINT
// - Not shared between injection points
// - Highest overhead
// - Rarely used (most use cases covered by DEFAULT/REQUEST)

// LIFECYCLE:
// Injection Point 1 → Create UniqueIdGenerator (instance A)
// Injection Point 2 → Create UniqueIdGenerator (instance B)
// Both are different instances

// EXAMPLE - Multiple injections create multiple instances
@Injectable()
export class OrdersService {
  constructor(
    private readonly idGenerator1: UniqueIdGenerator,  // Instance A
    private readonly idGenerator2: UniqueIdGenerator,  // Instance B
  ) {
    console.log('ID 1:', this.idGenerator1.getId());  // Different ID
    console.log('ID 2:', this.idGenerator2.getId());  // Different ID
  }
}

// ========== WHEN TO USE TRANSIENT SCOPE ==========

// ✅ USE TRANSIENT for (rare cases):
// 1. Truly disposable instances
@Injectable({ scope: Scope.TRANSIENT })
export class TempFileWriter {
  private readonly tempFile: string = `/tmp/${uuidv4()}.txt`;

  async write(data: string): Promise<void> {
    await fs.writeFile(this.tempFile, data);
  }

  async cleanup(): Promise<void> {
    await fs.unlink(this.tempFile);
  }
}

// 2. Stateful services that shouldn't be shared (rare)
@Injectable({ scope: Scope.TRANSIENT })
export class Calculator {
  private result: number = 0;

  add(value: number): void {
    this.result += value;
  }

  getResult(): number {
    return this.result;
  }
}

// ❌ DO NOT USE TRANSIENT for:
// - Normal services (use DEFAULT)
// - Request-specific services (use REQUEST)
// - Most production use cases (rarely needed)

// PERFORMANCE: ⚡ WORST (per-injection overhead)
```

---

### **Method 4: Scope Comparison**

```typescript
// ========== SCOPE COMPARISON ==========

// DEFAULT SCOPE (Singleton)
@Injectable()
export class DefaultService {
  constructor() {
    console.log('DefaultService created');  // Called ONCE
  }
}

// REQUEST SCOPE (Per-Request)
@Injectable({ scope: Scope.REQUEST })
export class RequestService {
  constructor() {
    console.log('RequestService created');  // Called PER REQUEST
  }
}

// TRANSIENT SCOPE (Per-Injection)
@Injectable({ scope: Scope.TRANSIENT })
export class TransientService {
  constructor() {
    console.log('TransientService created');  // Called PER INJECTION
  }
}

// TEST CONTROLLER
@Controller('test')
export class TestController {
  constructor(
    private readonly defaultService: DefaultService,      // Same instance everywhere
    private readonly requestService: RequestService,      // New per request
    private readonly transientService1: TransientService, // New instance 1
    private readonly transientService2: TransientService, // New instance 2
  ) {}

  @Get()
  async test() {
    return {
      defaultService: this.defaultService,      // Same instance ID
      requestService: this.requestService,      // Changes per request
      transientService1: this.transientService1,// Different from transientService2
      transientService2: this.transientService2,// Different from transientService1
    };
  }
}

// REQUEST 1:
// DefaultService created (ONCE at startup)
// RequestService created (for request 1)
// TransientService created (injection point 1)
// TransientService created (injection point 2)

// REQUEST 2:
// RequestService created (for request 2)
// TransientService created (injection point 1)
// TransientService created (injection point 2)
// Note: DefaultService NOT recreated (singleton)
```

---

### **Provider Scope Comparison Table:**

| Aspect | DEFAULT | REQUEST | TRANSIENT |
|--------|---------|---------|-----------|
| **Created** | Once at startup | Per HTTP request | Per injection point |
| **Shared** | Across entire app | Within single request | Not shared |
| **Lifetime** | Application lifetime | Request lifetime | Injection lifetime |
| **Access to Request** | ❌ No | ✅ Yes | ❌ No |
| **Performance** | ⚡⚡⚡ Best | ⚡⚡ Moderate | ⚡ Worst |
| **Memory** | Low | Medium | High |
| **Use Cases** | Stateless services | Request context, multi-tenancy | Rare, disposable instances |
| **Propagation** | ❌ No | ✅ Yes (up chain) | ✅ Yes (up chain) |
| **Recommended** | ✅ 99% of cases | Use when needed | Rarely use |

---

### **Method 5: Performance Implications**

```typescript
// ========== PERFORMANCE COMPARISON ==========

// SCENARIO: 1000 requests/second, 10 services in dependency chain

// DEFAULT SCOPE:
// - Services created: 10 (once at startup)
// - Memory allocations: 10 total
// - GC pressure: None
// - Performance: ⚡⚡⚡ BEST (0ms overhead)

// REQUEST SCOPE:
// - Services created: 10,000 per second (10 services × 1000 requests)
// - Memory allocations: 10,000 per second
// - GC pressure: High
// - Performance: ⚡⚡ MODERATE (~5-10ms overhead per request)

// TRANSIENT SCOPE:
// - Services created: 30,000 per second (if 3 injection points × 10 services × 1000 requests)
// - Memory allocations: 30,000 per second
// - GC pressure: Very high
// - Performance: ⚡ WORST (~20-30ms overhead per request)

// BENCHMARK EXAMPLE
import { performance } from 'perf_hooks';

// DEFAULT (Singleton)
const start1 = performance.now();
for (let i = 0; i < 1000; i++) {
  const service = defaultService;  // Same instance
}
const end1 = performance.now();
console.log('DEFAULT:', end1 - start1, 'ms');  // ~0.1ms

// REQUEST
const start2 = performance.now();
for (let i = 0; i < 1000; i++) {
  const service = new RequestService();  // New instance per request
}
const end2 = performance.now();
console.log('REQUEST:', end2 - start2, 'ms');  // ~5-10ms

// TRANSIENT
const start3 = performance.now();
for (let i = 0; i < 1000; i++) {
  const service1 = new TransientService();  // New instance
  const service2 = new TransientService();  // New instance
  const service3 = new TransientService();  // New instance
}
const end3 = performance.now();
console.log('TRANSIENT:', end3 - start3, 'ms');  // ~20-30ms
```

---

### **Best Practices:**

```typescript
// ✅ GOOD: Use DEFAULT scope by default
@Injectable()  // DEFAULT scope
export class UsersService {
  // Stateless, no request-specific data
}

// ❌ BAD: Unnecessary REQUEST scope
@Injectable({ scope: Scope.REQUEST })
export class UsersService {
  // No request-specific logic - should be DEFAULT!
}

// ✅ GOOD: REQUEST scope for request context
@Injectable({ scope: Scope.REQUEST })
export class RequestContext {
  constructor(@Inject(REQUEST) private readonly request: Request) {}

  get currentUser(): User {
    return this.request.user;
  }
}

// ❌ BAD: Storing request data in DEFAULT-scoped service
@Injectable()
export class BadService {
  private currentUser: User;  // WRONG! Shared across requests!

  setCurrentUser(user: User) {
    this.currentUser = user;  // Race condition!
  }
}

// ✅ GOOD: Minimize REQUEST-scoped services
// Only RequestContext is REQUEST-scoped
@Injectable({ scope: Scope.REQUEST })
export class RequestContext { }

@Injectable()  // DEFAULT scope
export class UsersService {
  constructor(private readonly requestContext: RequestContext) {}
  // UsersService becomes REQUEST-scoped due to dependency
}

// ✅ GOOD: Document why REQUEST scope is needed
/**
 * RequestContextService
 * 
 * REQUEST-scoped to access current user, tenant, and correlation ID
 * for each HTTP request. Required for multi-tenancy and audit logging.
 */
@Injectable({ scope: Scope.REQUEST })
export class RequestContextService { }
```

**Key Takeaway:** NestJS providers have three scopes: **DEFAULT** (singleton, created once at startup, shared across entire application, best performance with zero overhead, recommended for **stateless services**, repositories, utilities, config - **99% of use cases**), **REQUEST** (new instance per HTTP request, can access request object via @Inject(REQUEST), higher overhead with 5-10ms per request, use for **request-specific context** like current user, tenant ID, correlation ID, audit logging, multi-tenancy - **when you need request data**), and **TRANSIENT** (new instance per injection point, not shared between injections, highest overhead with 20-30ms, rarely used - **only for truly disposable instances**). Set scope with `@Injectable({ scope: Scope.REQUEST })` or in module providers array. **Scope propagation**: if a service is REQUEST/TRANSIENT-scoped, all services that depend on it become REQUEST/TRANSIENT-scoped (entire dependency chain affected). **Performance**: DEFAULT > REQUEST > TRANSIENT. **Best practice**: use DEFAULT by default, only use REQUEST when you genuinely need request-specific data (user, tenant, correlation ID), avoid TRANSIENT in production (rarely needed), document why non-DEFAULT scopes are used, minimize REQUEST-scoped services to reduce performance impact, and never store request-specific state in DEFAULT-scoped services (race conditions).

</details>

<details>
<summary><strong>37. What is REQUEST scope and when to use it?</strong></summary>

**Answer:**

REQUEST scope creates a **new provider instance for each HTTP request** (not singleton), allows **access to request object** via `@Inject(REQUEST)`, and is **destroyed after request completes**. Use for **request-specific context** (current user, tenant ID, correlation ID), **multi-tenancy** (tenant-based data isolation), **audit logging** (track who did what), and **request-scoped caching** (cache per request). Set with `@Injectable({ scope: Scope.REQUEST })`. **Important**: scope **propagates up dependency chain** (if UsersService is REQUEST-scoped, all its dependents become REQUEST-scoped). Has **performance overhead** (~5-10ms per request) due to instance creation/destruction. **Best practice**: minimize REQUEST-scoped services, use for genuinely request-specific data only, pass request context explicitly instead of using REQUEST scope when possible.

---

### **REQUEST Scope Overview:**

```typescript
// REQUEST-scoped service - new instance per HTTP request
@Injectable({ scope: Scope.REQUEST })
export class RequestContextService {
  private userId: string;
  private tenantId: string;
  private correlationId: string;

  constructor(@Inject(REQUEST) private readonly request: Request) {
    // Access request object
    this.userId = request.user?.id;
    this.tenantId = request.headers['x-tenant-id'] as string;
    this.correlationId = request.headers['x-correlation-id'] as string || uuidv4();
  }

  getUserId(): string {
    return this.userId;
  }

  getTenantId(): string {
    return this.tenantId;
  }

  getCorrelationId(): string {
    return this.correlationId;
  }
}

// LIFECYCLE:
// Request 1 → Create RequestContextService (instance A) → Process request → Destroy instance A
// Request 2 → Create RequestContextService (instance B) → Process request → Destroy instance B
// Each request gets its own instance
```

---

### **Method 1: Request Context Pattern (Most Common Use Case)**

```typescript
// ========== REQUEST CONTEXT FOR CURRENT USER/TENANT ==========

// Request context service - stores request-specific data
@Injectable({ scope: Scope.REQUEST })
export class RequestContext {
  private readonly _user: User;
  private readonly _tenantId: string;
  private readonly _correlationId: string;
  private readonly _requestId: string;
  private readonly _ipAddress: string;

  constructor(@Inject(REQUEST) private readonly request: Request) {
    this._user = request.user;
    this._tenantId = request.headers['x-tenant-id'] as string;
    this._correlationId = request.headers['x-correlation-id'] as string || uuidv4();
    this._requestId = uuidv4();
    this._ipAddress = request.ip;
  }

  get user(): User {
    return this._user;
  }

  get userId(): string {
    return this._user?.id;
  }

  get tenantId(): string {
    return this._tenantId;
  }

  get correlationId(): string {
    return this._correlationId;
  }

  get requestId(): string {
    return this._requestId;
  }

  get ipAddress(): string {
    return this._ipAddress;
  }

  isAuthenticated(): boolean {
    return !!this._user;
  }

  hasRole(role: string): boolean {
    return this._user?.roles?.includes(role) ?? false;
  }
}

// USAGE IN SERVICE - Access current user without passing explicitly
@Injectable()  // DEFAULT scope (becomes REQUEST-scoped due to dependency)
export class OrdersService {
  constructor(
    private readonly ordersRepository: OrdersRepository,
    private readonly requestContext: RequestContext,  // REQUEST-scoped
  ) {}

  async create(dto: CreateOrderDto): Promise<Order> {
    // Access current user from request context
    const userId = this.requestContext.userId;
    const tenantId = this.requestContext.tenantId;

    const order = await this.ordersRepository.save({
      ...dto,
      userId,
      tenantId,
      createdAt: new Date(),
    });

    return order;
  }

  async findMyOrders(): Promise<Order[]> {
    // Automatically filter by current user
    const userId = this.requestContext.userId;
    return this.ordersRepository.findByUserId(userId);
  }
}

// USAGE IN CONTROLLER
@Controller('orders')
@UseGuards(AuthGuard)
export class OrdersController {
  constructor(
    private readonly ordersService: OrdersService,
    private readonly requestContext: RequestContext,
  ) {}

  @Post()
  async create(@Body() dto: CreateOrderDto): Promise<OrderResponseDto> {
    // No need to pass userId - service gets it from RequestContext
    return this.ordersService.create(dto);
  }

  @Get('my-orders')
  async getMyOrders(): Promise<OrderResponseDto[]> {
    // Service automatically filters by current user
    return this.ordersService.findMyOrders();
  }

  @Get('current-user')
  async getCurrentUser(): Promise<User> {
    // Access current user anywhere
    return this.requestContext.user;
  }
}
```

---

### **Method 2: Multi-Tenancy Pattern**

```typescript
// ========== MULTI-TENANCY WITH REQUEST SCOPE ==========

// Tenant context - isolate data by tenant
@Injectable({ scope: Scope.REQUEST })
export class TenantContext {
  private readonly _tenantId: string;
  private _tenant: Tenant;

  constructor(
    @Inject(REQUEST) private readonly request: Request,
    private readonly tenantsRepository: TenantsRepository,
  ) {
    // Extract tenant from header, subdomain, or JWT
    this._tenantId = this.extractTenantId(request);
  }

  private extractTenantId(request: Request): string {
    // Option 1: From header
    const headerTenant = request.headers['x-tenant-id'] as string;
    if (headerTenant) return headerTenant;

    // Option 2: From subdomain (tenant1.app.com → tenant1)
    const host = request.headers.host;
    const subdomain = host?.split('.')[0];
    if (subdomain && subdomain !== 'www' && subdomain !== 'api') {
      return subdomain;
    }

    // Option 3: From JWT token
    const user = request.user as any;
    if (user?.tenantId) return user.tenantId;

    throw new UnauthorizedException('Tenant ID not found');
  }

  get tenantId(): string {
    return this._tenantId;
  }

  async getTenant(): Promise<Tenant> {
    if (!this._tenant) {
      this._tenant = await this.tenantsRepository.findById(this._tenantId);
      if (!this._tenant) {
        throw new NotFoundException(`Tenant ${this._tenantId} not found`);
      }
    }
    return this._tenant;
  }
}

// Tenant-aware repository
@Injectable()
export class ProductsRepository {
  constructor(
    @InjectRepository(Product)
    private readonly repository: Repository<Product>,
    private readonly tenantContext: TenantContext,  // REQUEST-scoped
  ) {}

  async findAll(): Promise<Product[]> {
    // Automatically filter by tenant
    const tenantId = this.tenantContext.tenantId;
    return this.repository.find({ where: { tenantId } });
  }

  async findById(id: string): Promise<Product> {
    // Ensure product belongs to current tenant
    const tenantId = this.tenantContext.tenantId;
    const product = await this.repository.findOne({
      where: { id, tenantId },
    });

    if (!product) {
      throw new NotFoundException('Product not found');
    }

    return product;
  }

  async save(product: Partial<Product>): Promise<Product> {
    // Automatically set tenant ID
    const tenantId = this.tenantContext.tenantId;
    return this.repository.save({ ...product, tenantId });
  }
}

// USAGE - Tenant isolation is automatic
@Injectable()
export class ProductsService {
  constructor(private readonly productsRepository: ProductsRepository) {}

  async findAll(): Promise<Product[]> {
    // Only returns products for current tenant
    return this.productsRepository.findAll();
  }

  async create(dto: CreateProductDto): Promise<Product> {
    // Automatically assigned to current tenant
    return this.productsRepository.save(dto);
  }
}
```

---

### **Method 3: Request-Scoped Logging with Correlation ID**

```typescript
// ========== REQUEST-SCOPED LOGGING ==========

// Logger with correlation ID
@Injectable({ scope: Scope.REQUEST })
export class RequestLogger {
  private readonly correlationId: string;
  private readonly requestId: string;
  private readonly userId: string;
  private readonly path: string;
  private readonly method: string;
  private readonly startTime: number;

  constructor(
    @Inject(REQUEST) private readonly request: Request,
    private readonly logger: Logger,
  ) {
    this.correlationId = request.headers['x-correlation-id'] as string || uuidv4();
    this.requestId = uuidv4();
    this.userId = request.user?.id || 'anonymous';
    this.path = request.path;
    this.method = request.method;
    this.startTime = Date.now();

    // Add correlation ID to response headers
    request.res?.setHeader('x-correlation-id', this.correlationId);
    request.res?.setHeader('x-request-id', this.requestId);
  }

  private formatMessage(level: string, message: string, context?: any): string {
    const duration = Date.now() - this.startTime;
    return JSON.stringify({
      timestamp: new Date().toISOString(),
      level,
      correlationId: this.correlationId,
      requestId: this.requestId,
      userId: this.userId,
      method: this.method,
      path: this.path,
      duration,
      message,
      context,
    });
  }

  log(message: string, context?: any): void {
    this.logger.log(this.formatMessage('INFO', message, context));
  }

  error(message: string, error?: Error, context?: any): void {
    this.logger.error(
      this.formatMessage('ERROR', message, {
        ...context,
        error: error?.message,
        stack: error?.stack,
      }),
    );
  }

  warn(message: string, context?: any): void {
    this.logger.warn(this.formatMessage('WARN', message, context));
  }

  debug(message: string, context?: any): void {
    this.logger.debug(this.formatMessage('DEBUG', message, context));
  }
}

// USAGE IN SERVICE
@Injectable()
export class OrdersService {
  constructor(
    private readonly ordersRepository: OrdersRepository,
    private readonly requestLogger: RequestLogger,  // REQUEST-scoped
  ) {}

  async create(dto: CreateOrderDto): Promise<Order> {
    this.requestLogger.log('Creating order', { dto });

    try {
      const order = await this.ordersRepository.save(dto);
      
      this.requestLogger.log('Order created successfully', {
        orderId: order.id,
      });

      return order;
    } catch (error) {
      this.requestLogger.error('Failed to create order', error, { dto });
      throw error;
    }
  }
}

// LOG OUTPUT (all logs from same request have same correlation ID):
// {"timestamp":"2025-01-01T10:00:00.000Z","level":"INFO","correlationId":"abc-123","requestId":"req-456","userId":"user-789","method":"POST","path":"/orders","duration":5,"message":"Creating order","context":{"dto":{...}}}
// {"timestamp":"2025-01-01T10:00:00.005Z","level":"INFO","correlationId":"abc-123","requestId":"req-456","userId":"user-789","method":"POST","path":"/orders","duration":10,"message":"Order created successfully","context":{"orderId":"order-123"}}
```

---

### **Method 4: Audit Logging Pattern**

```typescript
// ========== AUDIT LOGGING WITH REQUEST SCOPE ==========

// Audit service - track who did what
@Injectable({ scope: Scope.REQUEST })
export class AuditService {
  constructor(
    @Inject(REQUEST) private readonly request: Request,
    private readonly auditRepository: AuditRepository,
    private readonly requestContext: RequestContext,
  ) {}

  async logAction(
    action: string,
    resource: string,
    resourceId?: string,
    metadata?: any,
  ): Promise<void> {
    const auditLog = {
      userId: this.requestContext.userId,
      tenantId: this.requestContext.tenantId,
      correlationId: this.requestContext.correlationId,
      action,
      resource,
      resourceId,
      metadata,
      timestamp: new Date(),
      ipAddress: this.request.ip,
      userAgent: this.request.headers['user-agent'],
      method: this.request.method,
      path: this.request.path,
    };

    await this.auditRepository.save(auditLog);
  }

  async logCreate(resource: string, resourceId: string, data: any): Promise<void> {
    await this.logAction('CREATE', resource, resourceId, { data });
  }

  async logUpdate(
    resource: string,
    resourceId: string,
    before: any,
    after: any,
  ): Promise<void> {
    await this.logAction('UPDATE', resource, resourceId, { before, after });
  }

  async logDelete(resource: string, resourceId: string, data: any): Promise<void> {
    await this.logAction('DELETE', resource, resourceId, { data });
  }

  async logAccess(resource: string, resourceId?: string): Promise<void> {
    await this.logAction('ACCESS', resource, resourceId);
  }
}

// USAGE IN SERVICE
@Injectable()
export class UsersService {
  constructor(
    private readonly usersRepository: UsersRepository,
    private readonly auditService: AuditService,  // REQUEST-scoped
  ) {}

  async create(dto: CreateUserDto): Promise<User> {
    const user = await this.usersRepository.save(dto);

    // Log creation with current user context
    await this.auditService.logCreate('users', user.id, dto);

    return user;
  }

  async update(id: string, dto: UpdateUserDto): Promise<User> {
    const before = await this.usersRepository.findById(id);
    const after = await this.usersRepository.save({ ...before, ...dto });

    // Log update with before/after state
    await this.auditService.logUpdate('users', id, before, after);

    return after;
  }

  async delete(id: string): Promise<void> {
    const user = await this.usersRepository.findById(id);
    await this.usersRepository.delete(id);

    // Log deletion
    await this.auditService.logDelete('users', id, user);
  }

  async findById(id: string): Promise<User> {
    const user = await this.usersRepository.findById(id);

    // Log access for sensitive operations
    await this.auditService.logAccess('users', id);

    return user;
  }
}
```

---

### **Method 5: When NOT to Use REQUEST Scope**

```typescript
// ========== WHEN NOT TO USE REQUEST SCOPE ==========

// ❌ BAD: Using REQUEST scope for stateless service
@Injectable({ scope: Scope.REQUEST })  // Unnecessary!
export class ProductsService {
  constructor(private readonly productsRepository: ProductsRepository) {}

  async findAll(): Promise<Product[]> {
    // No request-specific logic - should be DEFAULT scope!
    return this.productsRepository.findAll();
  }
}

// ✅ GOOD: Use DEFAULT scope for stateless services
@Injectable()  // DEFAULT scope
export class ProductsService {
  constructor(private readonly productsRepository: ProductsRepository) {}

  async findAll(): Promise<Product[]> {
    return this.productsRepository.findAll();
  }
}

// ❌ BAD: Storing request data in DEFAULT-scoped service
@Injectable()  // DEFAULT scope (singleton!)
export class BadService {
  private currentUser: User;  // RACE CONDITION!

  setCurrentUser(user: User) {
    this.currentUser = user;  // User from request 1 overwritten by request 2!
  }

  getCurrentUser(): User {
    return this.currentUser;  // Returns wrong user!
  }
}

// ✅ GOOD: Pass request data as parameters
@Injectable()
export class GoodService {
  async doSomething(userId: string): Promise<void> {
    // Accept userId as parameter instead of storing in instance
  }
}

// Or use RequestContext (REQUEST-scoped)
@Injectable()
export class GoodService {
  constructor(private readonly requestContext: RequestContext) {}

  async doSomething(): Promise<void> {
    const userId = this.requestContext.userId;  // Safe - new instance per request
  }
}

// ❌ BAD: Overusing REQUEST scope (performance impact)
@Injectable({ scope: Scope.REQUEST })
export class Service1 { }

@Injectable({ scope: Scope.REQUEST })
export class Service2 { }

@Injectable({ scope: Scope.REQUEST })
export class Service3 { }
// 10+ REQUEST-scoped services = high overhead!

// ✅ GOOD: Minimize REQUEST-scoped services
@Injectable({ scope: Scope.REQUEST })
export class RequestContext {
  // Single REQUEST-scoped service for request data
}

@Injectable()  // DEFAULT scope
export class Service1 {
  constructor(private readonly requestContext: RequestContext) {}
  // Service1 becomes REQUEST-scoped due to dependency, but that's OK
}

@Injectable()  // DEFAULT scope
export class Service2 {
  constructor(private readonly requestContext: RequestContext) {}
}
```

---

### **REQUEST Scope Use Cases:**

| Use Case | Example | Benefit |
|----------|---------|---------|
| **Current User Context** | Access user ID without passing explicitly | Clean service signatures |
| **Multi-Tenancy** | Automatic tenant isolation | Data security |
| **Correlation ID** | Track requests across services | Distributed tracing |
| **Audit Logging** | Track who did what | Compliance |
| **Request-Scoped Cache** | Cache data for single request | Avoid duplicate queries |
| **Request Timing** | Measure request duration | Performance monitoring |

---

### **Best Practices:**

```typescript
// ✅ GOOD: Single REQUEST-scoped context service
@Injectable({ scope: Scope.REQUEST })
export class RequestContext {
  constructor(@Inject(REQUEST) private readonly request: Request) {}
  // All request-specific data in one place
}

// ❌ BAD: Multiple REQUEST-scoped services
@Injectable({ scope: Scope.REQUEST })
export class UserContext { }

@Injectable({ scope: Scope.REQUEST })
export class TenantContext { }

@Injectable({ scope: Scope.REQUEST })
export class LoggingContext { }
// Should be combined into one RequestContext

// ✅ GOOD: Pass request context explicitly when possible
@Injectable()
export class OrdersService {
  async create(userId: string, dto: CreateOrderDto): Promise<Order> {
    // Explicit parameter - no REQUEST scope needed
    return this.ordersRepository.save({ ...dto, userId });
  }
}

// ❌ BAD: Using REQUEST scope just to avoid passing parameters
@Injectable({ scope: Scope.REQUEST })
export class OrdersService {
  constructor(@Inject(REQUEST) private readonly request: Request) {}

  async create(dto: CreateOrderDto): Promise<Order> {
    const userId = this.request.user.id;
    // REQUEST scope just to avoid passing userId
  }
}

// ✅ GOOD: Document why REQUEST scope is needed
/**
 * RequestContext
 * 
 * REQUEST-scoped to provide access to:
 * - Current user (authentication)
 * - Tenant ID (multi-tenancy)
 * - Correlation ID (distributed tracing)
 * 
 * Note: This makes all dependent services REQUEST-scoped
 */
@Injectable({ scope: Scope.REQUEST })
export class RequestContext { }
```

**Key Takeaway:** REQUEST scope creates **new instance per HTTP request** (not singleton), allows **access to request object** via `@Inject(REQUEST)`, provides **request-specific data** (user, tenant, correlation ID), and is **destroyed after request completes**. Use for **request context** (RequestContext service with current user, tenant ID, correlation ID), **multi-tenancy** (automatic tenant isolation in repositories, TenantContext service), **audit logging** (track who did what with user/IP/timestamp), **request-scoped logging** (correlation ID for distributed tracing), and **request-scoped caching** (cache per request, not shared). Set with `@Injectable({ scope: Scope.REQUEST })`. **Scope propagation**: if service is REQUEST-scoped, all dependents become REQUEST-scoped (entire dependency chain affected, OrdersService depends on RequestContext → OrdersService becomes REQUEST-scoped). **Performance overhead**: ~5-10ms per request due to instance creation/destruction, memory allocation, garbage collection. **Best practice**: minimize REQUEST-scoped services (single RequestContext service, not 10+ REQUEST-scoped services), use only when genuinely need request-specific data (user/tenant/correlation ID), pass request context explicitly when possible (avoid REQUEST scope just to avoid parameters), document why REQUEST scope is needed, never store request-specific state in DEFAULT-scoped services (race conditions), and consider alternatives (pass userId as parameter, use middleware to set context, use AsyncLocalStorage).

</details>

<details>
<summary><strong>38. What is TRANSIENT scope?</strong></summary>

**Answer:**

TRANSIENT scope creates a **new provider instance for each injection point** (not shared, even within same request/component), meaning if a service is injected twice in same class, two different instances are created. Use for **truly disposable instances** (temporary calculations, one-time operations), **stateful services that shouldn't be shared** (each consumer needs independent state), and **dynamic configuration** (different instances with different configs). Set with `@Injectable({ scope: Scope.TRANSIENT })`. Has **highest overhead** (~20-30ms impact) due to creating instance for every injection. **Rarely needed in production** - most use cases covered by DEFAULT (stateless services) or REQUEST (request-specific data). **Scope propagates**: if dependency is TRANSIENT, dependent becomes TRANSIENT. **Best practice**: almost never use TRANSIENT scope, prefer DEFAULT for stateless services, use REQUEST for request-specific data, avoid unless you have specific reason for non-shared instances.

---

### **TRANSIENT Scope Overview:**

```typescript
// TRANSIENT-scoped service - new instance per injection point
@Injectable({ scope: Scope.TRANSIENT })
export class UniqueIdGenerator {
  private readonly id: string = uuidv4();

  constructor() {
    console.log('UniqueIdGenerator created with ID:', this.id);
  }

  getId(): string {
    return this.id;
  }
}

// USAGE - Multiple injections create multiple instances
@Injectable()
export class ExampleService {
  constructor(
    private readonly idGenerator1: UniqueIdGenerator,  // Instance A
    private readonly idGenerator2: UniqueIdGenerator,  // Instance B (different!)
  ) {
    console.log('ID 1:', this.idGenerator1.getId());  // e.g., "abc-123"
    console.log('ID 2:', this.idGenerator2.getId());  // e.g., "def-456" (different!)
  }
}

// LIFECYCLE:
// ExampleService created → Create UniqueIdGenerator (A) → Create UniqueIdGenerator (B)
// Each injection point gets its own instance
```

---

### **Method 1: TRANSIENT Scope Behavior**

```typescript
// ========== TRANSIENT SCOPE BEHAVIOR ==========

// TRANSIENT-scoped service
@Injectable({ scope: Scope.TRANSIENT })
export class Counter {
  private count: number = 0;

  constructor() {
    console.log('Counter instance created');
  }

  increment(): void {
    this.count++;
  }

  getCount(): number {
    return this.count;
  }
}

// SERVICE WITH MULTIPLE INJECTIONS
@Injectable()
export class TestService {
  constructor(
    private readonly counter1: Counter,  // Instance A
    private readonly counter2: Counter,  // Instance B (separate instance!)
  ) {
    // Output:
    // Counter instance created (for counter1)
    // Counter instance created (for counter2)
  }

  test(): void {
    this.counter1.increment();  // counter1 count = 1
    this.counter1.increment();  // counter1 count = 2
    
    this.counter2.increment();  // counter2 count = 1 (different instance!)
    
    console.log('Counter 1:', this.counter1.getCount());  // 2
    console.log('Counter 2:', this.counter2.getCount());  // 1 (independent state)
  }
}

// CONTRAST WITH DEFAULT SCOPE (Singleton)
@Injectable()  // DEFAULT scope
export class SingletonCounter {
  private count: number = 0;

  constructor() {
    console.log('SingletonCounter instance created');  // Called ONCE
  }

  increment(): void {
    this.count++;
  }

  getCount(): number {
    return this.count;
  }
}

@Injectable()
export class TestService2 {
  constructor(
    private readonly counter1: SingletonCounter,  // Same instance
    private readonly counter2: SingletonCounter,  // Same instance (singleton!)
  ) {
    // Output:
    // SingletonCounter instance created (ONCE, shared)
  }

  test(): void {
    this.counter1.increment();  // count = 1
    this.counter2.increment();  // count = 2 (same instance!)
    
    console.log('Counter 1:', this.counter1.getCount());  // 2
    console.log('Counter 2:', this.counter2.getCount());  // 2 (shared state)
  }
}
```

---

### **Method 2: Use Cases for TRANSIENT Scope (Rare)**

```typescript
// ========== USE CASE 1: TEMPORARY FILE OPERATIONS ==========

@Injectable({ scope: Scope.TRANSIENT })
export class TempFileWriter {
  private readonly tempFile: string;

  constructor() {
    // Each instance gets unique temp file
    this.tempFile = `/tmp/${uuidv4()}.txt`;
  }

  async write(data: string): Promise<void> {
    await fs.promises.writeFile(this.tempFile, data);
  }

  async read(): Promise<string> {
    return fs.promises.readFile(this.tempFile, 'utf-8');
  }

  async cleanup(): Promise<void> {
    try {
      await fs.promises.unlink(this.tempFile);
    } catch {
      // File already deleted or doesn't exist
    }
  }

  getTempFilePath(): string {
    return this.tempFile;
  }
}

// USAGE - Each consumer gets independent temp file
@Injectable()
export class ReportService {
  async generateReport(data: any): Promise<Buffer> {
    const writer = new TempFileWriter();  // Manually instantiate if TRANSIENT
    
    try {
      await writer.write(JSON.stringify(data));
      const content = await writer.read();
      // Process content...
      return Buffer.from(content);
    } finally {
      await writer.cleanup();  // Clean up temp file
    }
  }
}

// ========== USE CASE 2: STATEFUL CALCULATOR (Academic Example) ==========

@Injectable({ scope: Scope.TRANSIENT })
export class Calculator {
  private result: number = 0;

  add(value: number): Calculator {
    this.result += value;
    return this;  // Fluent API
  }

  subtract(value: number): Calculator {
    this.result -= value;
    return this;
  }

  multiply(value: number): Calculator {
    this.result *= value;
    return this;
  }

  getResult(): number {
    return this.result;
  }

  reset(): void {
    this.result = 0;
  }
}

// USAGE - Each consumer gets independent calculator
@Injectable()
export class MathService {
  constructor(
    private readonly calc1: Calculator,  // Independent instance
    private readonly calc2: Calculator,  // Independent instance
  ) {}

  async calculate1(): Promise<number> {
    return this.calc1.add(5).multiply(2).getResult();  // 10
  }

  async calculate2(): Promise<number> {
    return this.calc2.add(10).subtract(3).getResult();  // 7
  }
  // calc1 and calc2 have independent state
}

// NOTE: In practice, you'd use pure functions instead of TRANSIENT scope
export class BetterMathService {
  calculate1(): number {
    let result = 0;
    result += 5;
    result *= 2;
    return result;  // 10
  }
  // No need for stateful calculator!
}

// ========== USE CASE 3: UNIQUE ID GENERATION ==========

@Injectable({ scope: Scope.TRANSIENT })
export class UniqueIdService {
  private readonly uuid: string = uuidv4();
  private readonly timestamp: number = Date.now();

  getUuid(): string {
    return this.uuid;
  }

  getTimestamp(): number {
    return this.timestamp;
  }

  getCombinedId(): string {
    return `${this.timestamp}-${this.uuid}`;
  }
}

// USAGE - Each injection gets unique ID
@Injectable()
export class OrdersService {
  constructor(
    private readonly orderIdGenerator: UniqueIdService,   // Instance A
    private readonly correlationIdGenerator: UniqueIdService,  // Instance B
  ) {}

  async create(dto: CreateOrderDto): Promise<Order> {
    const orderId = this.orderIdGenerator.getCombinedId();
    const correlationId = this.correlationIdGenerator.getCombinedId();

    // orderId and correlationId are different
    console.log('Order ID:', orderId);           // 1672531200000-abc-123
    console.log('Correlation ID:', correlationId);  // 1672531200000-def-456

    return this.ordersRepository.save({
      ...dto,
      id: orderId,
      correlationId,
    });
  }
}

// NOTE: In practice, just call uuidv4() directly - no need for TRANSIENT scope!
@Injectable()
export class BetterOrdersService {
  async create(dto: CreateOrderDto): Promise<Order> {
    return this.ordersRepository.save({
      ...dto,
      id: uuidv4(),
      correlationId: uuidv4(),
    });
  }
}
```

---

### **Method 3: TRANSIENT vs DEFAULT vs REQUEST**

```typescript
// ========== COMPARISON: DEFAULT VS REQUEST VS TRANSIENT ==========

// DEFAULT SCOPE (Singleton) - Recommended for 99% of cases
@Injectable()
export class DefaultService {
  constructor() {
    console.log('DefaultService created');  // Called ONCE at startup
  }
}

@Injectable()
export class Consumer1 {
  constructor(private readonly service: DefaultService) {}  // Same instance
}

@Injectable()
export class Consumer2 {
  constructor(private readonly service: DefaultService) {}  // Same instance
}
// Consumer1 and Consumer2 share the same DefaultService instance

// REQUEST SCOPE - New instance per HTTP request
@Injectable({ scope: Scope.REQUEST })
export class RequestService {
  constructor() {
    console.log('RequestService created');  // Called per HTTP request
  }
}

@Injectable()
export class Consumer3 {
  constructor(private readonly service: RequestService) {}  // Instance A for request 1
}

@Injectable()
export class Consumer4 {
  constructor(private readonly service: RequestService) {}  // Same instance A for request 1
}
// Consumer3 and Consumer4 share the same instance within same request
// But get different instance in next request

// TRANSIENT SCOPE - New instance per injection
@Injectable({ scope: Scope.TRANSIENT })
export class TransientService {
  constructor() {
    console.log('TransientService created');  // Called per injection point
  }
}

@Injectable()
export class Consumer5 {
  constructor(
    private readonly service1: TransientService,  // Instance A
    private readonly service2: TransientService,  // Instance B (different!)
  ) {}
}
// Consumer5 has TWO different TransientService instances

// SUMMARY TABLE
// | Scope     | Instances Created | Shared?                      |
// |-----------|------------------|------------------------------|
// | DEFAULT   | 1 (at startup)   | Across entire application    |
// | REQUEST   | 1 per request    | Within same request          |
// | TRANSIENT | 1 per injection  | Never shared                 |
```

---

### **Method 4: Performance Impact**

```typescript
// ========== PERFORMANCE IMPACT OF TRANSIENT SCOPE ==========

// SCENARIO: Service with 5 dependencies, 1000 requests/second

// DEFAULT SCOPE:
// - Instances created: 5 (once at startup)
// - Memory allocations: 5 total
// - GC pressure: None
// - Overhead: 0ms

// REQUEST SCOPE:
// - Instances created: 5,000 per second (5 deps × 1000 req/s)
// - Memory allocations: 5,000 per second
// - GC pressure: Medium
// - Overhead: ~5-10ms per request

// TRANSIENT SCOPE (worst case: 3 injection points per service):
// - Instances created: 15,000 per second (3 injections × 5 services × 1000 req/s)
// - Memory allocations: 15,000 per second
// - GC pressure: HIGH
// - Overhead: ~20-30ms per request

// BENCHMARK
import { performance } from 'perf_hooks';

// DEFAULT (Singleton) - Fast
const defaultService = new DefaultService();
const start1 = performance.now();
for (let i = 0; i < 10000; i++) {
  const service = defaultService;  // Reference to existing instance
}
const end1 = performance.now();
console.log('DEFAULT:', end1 - start1, 'ms');  // ~0.1ms

// TRANSIENT - Slow
const start2 = performance.now();
for (let i = 0; i < 10000; i++) {
  const service = new TransientService();  // New instance each time
}
const end2 = performance.now();
console.log('TRANSIENT:', end2 - start2, 'ms');  // ~50-100ms

// CONCLUSION: TRANSIENT scope can be 100-1000x slower than DEFAULT
```

---

### **Method 5: When NOT to Use TRANSIENT Scope**

```typescript
// ========== WHEN NOT TO USE TRANSIENT SCOPE ==========

// ❌ BAD: Using TRANSIENT for stateless service
@Injectable({ scope: Scope.TRANSIENT })  // Unnecessary!
export class ProductsService {
  constructor(private readonly productsRepository: ProductsRepository) {}

  async findAll(): Promise<Product[]> {
    return this.productsRepository.findAll();
  }
}

// ✅ GOOD: Use DEFAULT for stateless services
@Injectable()  // DEFAULT scope
export class ProductsService {
  constructor(private readonly productsRepository: ProductsRepository) {}

  async findAll(): Promise<Product[]> {
    return this.productsRepository.findAll();
  }
}

// ❌ BAD: Using TRANSIENT to generate unique IDs
@Injectable({ scope: Scope.TRANSIENT })
export class IdGenerator {
  private readonly id = uuidv4();
  getId(): string { return this.id; }
}

// ✅ GOOD: Just call uuidv4() directly
@Injectable()
export class OrdersService {
  async create(dto: CreateOrderDto): Promise<Order> {
    return this.ordersRepository.save({
      ...dto,
      id: uuidv4(),  // Direct call - no TRANSIENT scope needed
    });
  }
}

// ❌ BAD: Using TRANSIENT for request-specific data
@Injectable({ scope: Scope.TRANSIENT })
export class UserContext {
  constructor(@Inject(REQUEST) private readonly request: Request) {}
  getCurrentUser(): User { return this.request.user; }
}

// ✅ GOOD: Use REQUEST scope for request-specific data
@Injectable({ scope: Scope.REQUEST })  // REQUEST, not TRANSIENT
export class UserContext {
  constructor(@Inject(REQUEST) private readonly request: Request) {}
  getCurrentUser(): User { return this.request.user; }
}

// ❌ BAD: Using TRANSIENT everywhere "just in case"
@Injectable({ scope: Scope.TRANSIENT })
export class Service1 { }

@Injectable({ scope: Scope.TRANSIENT })
export class Service2 { }

@Injectable({ scope: Scope.TRANSIENT })
export class Service3 { }
// Massive performance overhead for no benefit!

// ✅ GOOD: Use DEFAULT by default
@Injectable()
export class Service1 { }

@Injectable()
export class Service2 { }

@Injectable()
export class Service3 { }
```

---

### **TRANSIENT Scope Decision Matrix:**

| Scenario | Use TRANSIENT? | Better Alternative |
|----------|---------------|-------------------|
| Stateless service | ❌ No | DEFAULT scope |
| Request-specific data | ❌ No | REQUEST scope |
| Generate unique IDs | ❌ No | Call uuidv4() directly |
| Temporary file operations | ⚠️ Maybe | Manual instantiation with `new` |
| Truly disposable instances | ⚠️ Maybe | Consider if really needed |
| Stateful calculator | ❌ No | Use pure functions |
| Independent state per consumer | ⚠️ Maybe | Usually a code smell |

---

### **Best Practices:**

```typescript
// ✅ GOOD: Use DEFAULT scope (99% of cases)
@Injectable()
export class OrdersService {
  // Stateless, no shared state issues
}

// ✅ GOOD: Use REQUEST scope for request context
@Injectable({ scope: Scope.REQUEST })
export class RequestContext {
  // Need access to request object
}

// ⚠️ RARE: Use TRANSIENT only when absolutely necessary
@Injectable({ scope: Scope.TRANSIENT })
export class VerySpecialCase {
  // Document WHY this is TRANSIENT!
}

// ✅ GOOD: Avoid TRANSIENT in production
// Most "TRANSIENT use cases" are better solved with:
// 1. DEFAULT scope + stateless design
// 2. REQUEST scope for request data
// 3. Factory pattern for dynamic creation
// 4. Direct instantiation with `new`

// ✅ GOOD: Document why TRANSIENT is needed (rare cases)
/**
 * TempFileWriter
 * 
 * TRANSIENT-scoped because each consumer needs independent temp file.
 * Each injection creates a new instance with unique temp file path.
 * 
 * WARNING: High memory overhead. Use with caution.
 */
@Injectable({ scope: Scope.TRANSIENT })
export class TempFileWriter { }
```

**Key Takeaway:** TRANSIENT scope creates **new instance per injection point** (not shared, even within same class/request), meaning `constructor(svc1: MyService, svc2: MyService)` creates two different MyService instances. Use **rarely** for **truly disposable instances** (temp file operations where each consumer needs independent file), **stateful services that shouldn't be shared** (calculator with mutable state, though pure functions are better), and **unique ID generation** (though calling uuidv4() directly is simpler). Set with `@Injectable({ scope: Scope.TRANSIENT })`. Has **highest overhead** (~20-30ms per request, 100-1000x slower than DEFAULT) due to creating/destroying instances for every injection point, high memory allocations, and garbage collection pressure. **Scope propagates**: if dependency is TRANSIENT, dependent becomes TRANSIENT (entire chain affected). **Almost never needed in production** - 99.9% of use cases covered by DEFAULT (stateless services, repositories, utilities) or REQUEST (request-specific data like user/tenant). **Best practice**: default to DEFAULT scope, use REQUEST for request context, avoid TRANSIENT unless you have specific reason for non-shared instances per injection, document why TRANSIENT is needed (code smell if undocumented), prefer alternatives (pure functions instead of stateful calculator, call uuidv4() directly instead of TRANSIENT IdGenerator, factory pattern for dynamic creation, manual instantiation with `new`), and never use TRANSIENT just to avoid thinking about shared state.

</details>

<details>
<summary><strong>39. What are the performance implications of REQUEST scope?</strong></summary>

**Answer:**

REQUEST scope has **significant performance overhead** (~5-10ms per request) compared to DEFAULT scope due to: **instance creation/destruction** (new instance per request vs singleton), **memory allocation** (heap allocation for each request), **garbage collection pressure** (frequent object disposal), **dependency chain recreation** (all dependents become REQUEST-scoped), and **context switching** (dependency injection container overhead). Impact multiplies with **dependency chain depth** (5 services in chain = 5 instances per request). At **high traffic** (1000 req/s), creates 5,000 instances/second for 5-service chain. **Best practice**: minimize REQUEST-scoped services (single RequestContext), use DEFAULT for stateless services, profile performance impact, consider alternatives (pass context explicitly, use AsyncLocalStorage), and only use when benefits outweigh costs (multi-tenancy, audit logging, correlation ID tracking).

---

### **Performance Impact Summary:**

| Metric | DEFAULT | REQUEST | Impact |
|--------|---------|---------|--------|
| Instance creation | Once (startup) | Per request | 1000x more |
| Latency overhead | 0ms | 5-10ms | 10-15x slower |
| Memory per request | 0KB | 5-10KB | Continuous allocation |
| GC frequency | Rare | Every few seconds | 10-100x more |
| Throughput | 10,000 req/s | 7,000 req/s | 30% reduction |

**Key Takeaway:** REQUEST scope has **significant performance overhead** (~5-10ms per request, 10-15x slower than DEFAULT) due to **instance creation/destruction** (new objects per request vs singleton), **memory allocation** (heap allocations for each request, 5MB/s at 1000 req/s for 5KB service), **garbage collection pressure** (10-100x more frequent GC, pauses add latency, p99 latency increases 4x), **dependency chain recreation** (scope propagates, if 1 service is REQUEST-scoped, all dependents become REQUEST-scoped, 5-service chain = 5 instances per request), and **DI container overhead** (resolving dependencies repeatedly). Impact **multiplies with traffic** (1000 req/s × 5 services = 5,000 instances/second) and **chain depth** (overhead ≈ number of services × 1ms). **Best practice**: minimize REQUEST-scoped services (single RequestContext, not 10+ REQUEST services), use DEFAULT for stateless services (99% of cases), profile performance impact before deploying (use Artillery/k6 for load testing, clinic.js for profiling), monitor metrics (p50/p95/p99 latency, throughput, memory usage, GC pauses), consider alternatives (pass context explicitly as parameters, use AsyncLocalStorage, middleware to set context), cache expensive operations within request (avoid duplicate DB queries), and only use REQUEST scope when benefits outweigh costs (multi-tenancy with tenant isolation, audit logging with user tracking, correlation ID for distributed tracing).

</details>

<details>
<summary><strong>40. What is a DTO and why use them?</strong></summary>

**Answer:**

DTO (Data Transfer Object) is a **plain TypeScript class that defines structure and validation rules** for data transferred between layers (client ↔ API ↔ service), carries data without business logic, uses **class-validator decorators** for validation (@IsEmail, @IsNotEmpty, @Min), and **class-transformer decorators** for transformation (@Expose, @Exclude, @Transform). Use DTOs to: **validate input** (prevent invalid data from entering system), **transform data** (hide passwords, format dates, expose only needed fields), **ensure type safety** (TypeScript checks at compile time), **document API** (@ApiProperty for Swagger), **version APIs** (V1UserDto vs V2UserDto), **prevent over-posting** (ValidationPipe with whitelist: true strips extra fields), and **decouple API contract from internal models** (DTO ≠ Entity). Place in `dto/` folder, use with @Body(), @Query(), @Param(), apply ValidationPipe globally.

---

### **DTO Example with Validation:**

```typescript
// ========== INPUT DTO (CREATE) ==========

export class CreateUserDto {
  @IsString()
  @IsNotEmpty({ message: 'Name is required' })
  @MinLength(2, { message: 'Name must be at least 2 characters' })
  @MaxLength(50, { message: 'Name cannot exceed 50 characters' })
  name: string;

  @IsEmail({}, { message: 'Invalid email format' })
  email: string;

  @IsString()
  @MinLength(8, { message: 'Password must be at least 8 characters' })
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/, {
    message: 'Password must contain uppercase, lowercase, and number',
  })
  password: string;

  @IsOptional()
  @IsString()
  @MaxLength(500)
  bio?: string;

  @IsOptional()
  @IsEnum(UserRole)
  role?: UserRole = UserRole.USER;
}

// ========== OUTPUT DTO (RESPONSE) ==========

export class UserResponseDto {
  @Expose()
  id: string;

  @Expose()
  name: string;

  @Expose()
  email: string;

  @Expose()
  bio?: string;

  @Expose()
  role: UserRole;

  @Expose()
  @Transform(({ value }) => value?.toISOString())
  createdAt: string;

  // password NOT exposed - security!
  // internal fields NOT exposed
}

// ========== USAGE IN CONTROLLER ==========

@Controller('users')
export class UsersController {
  @Post()
  @ApiOperation({ summary: 'Create new user' })
  @ApiBody({ type: CreateUserDto })
  @ApiResponse({ status: 201, type: UserResponseDto })
  async create(@Body() dto: CreateUserDto): Promise<UserResponseDto> {
    // dto is validated automatically by ValidationPipe
    // If validation fails, throws 400 BadRequestException
    return this.usersService.create(dto);
  }

  @Get(':id')
  async findOne(@Param('id') id: string): Promise<UserResponseDto> {
    const user = await this.usersService.findById(id);
    // Transform entity to DTO (hide password, format dates)
    return plainToClass(UserResponseDto, user, {
      excludeExtraneousValues: true,
    });
  }
}
```

---

### **Why Use DTOs - Key Benefits:**

```typescript
// ========== BENEFIT 1: INPUT VALIDATION ==========

// Without DTO - No validation
@Post()
async create(@Body() data: any) {
  // What if email is invalid? What if name is missing?
  return this.usersService.create(data);  // ❌ Vulnerable!
}

// With DTO - Automatic validation
@Post()
async create(@Body() dto: CreateUserDto) {
  // ValidationPipe validates dto before reaching here
  // If invalid, automatically returns 400 error with details
  return this.usersService.create(dto);  // ✅ Safe!
}

// ========== BENEFIT 2: PREVENT OVER-POSTING ==========

// Without DTO - Security vulnerability
@Post()
async create(@Body() data: any) {
  // Client sends: { "name": "Hacker", "email": "hack@er.com", "isAdmin": true }
  // isAdmin gets saved! 🚨 SECURITY BREACH
  return this.usersService.create(data);
}

// With DTO + ValidationPipe whitelist
export class CreateUserDto {
  name: string;
  email: string;
  password: string;
  // isAdmin NOT in DTO
}

@Post()
@UsePipes(new ValidationPipe({ whitelist: true }))
async create(@Body() dto: CreateUserDto) {
  // ValidationPipe strips isAdmin from request
  // Only name, email, password are processed
  return this.usersService.create(dto);  // ✅ Safe!
}

// ========== BENEFIT 3: HIDE SENSITIVE DATA ==========

// Entity (database model)
@Entity()
export class User {
  id: string;
  name: string;
  email: string;
  password: string;  // Should NEVER be exposed!
  passwordResetToken: string;
  internalNotes: string;
}

// Without DTO - Exposes everything
@Get(':id')
async findOne(@Param('id') id: string): Promise<User> {
  return this.usersRepository.findById(id);
  // Returns: { id, name, email, password, passwordResetToken, internalNotes }
  // 🚨 PASSWORD EXPOSED!
}

// With DTO - Only expose what's needed
export class UserResponseDto {
  @Expose() id: string;
  @Expose() name: string;
  @Expose() email: string;
  // password, passwordResetToken, internalNotes NOT exposed
}

@Get(':id')
async findOne(@Param('id') id: string): Promise<UserResponseDto> {
  const user = await this.usersRepository.findById(id);
  return plainToClass(UserResponseDto, user, { excludeExtraneousValues: true });
  // Returns: { id, name, email }
  // ✅ Password hidden!
}

// ========== BENEFIT 4: API VERSIONING ==========

// V1 DTO - Original API
export class UserResponseDtoV1 {
  @Expose() id: string;
  @Expose() name: string;
  @Expose() email: string;
}

// V2 DTO - New fields added
export class UserResponseDtoV2 {
  @Expose() id: string;
  @Expose() name: string;
  @Expose() email: string;
  @Expose() phoneNumber: string;  // New field
  @Expose() isActive: boolean;    // New field
}

// Both versions work simultaneously
@Controller({ version: '1', path: 'users' })
export class UsersControllerV1 {
  @Get(':id')
  async findOne(@Param('id') id: string): Promise<UserResponseDtoV1> {
    // Returns V1 format
  }
}

@Controller({ version: '2', path: 'users' })
export class UsersControllerV2 {
  @Get(':id')
  async findOne(@Param('id') id: string): Promise<UserResponseDtoV2> {
    // Returns V2 format with new fields
  }
}

// ========== BENEFIT 5: SWAGGER DOCUMENTATION ==========

export class CreateProductDto {
  @ApiProperty({
    description: 'Product name',
    example: 'Wireless Mouse',
    minLength: 2,
    maxLength: 100,
  })
  @IsString()
  @MinLength(2)
  @MaxLength(100)
  name: string;

  @ApiProperty({
    description: 'Product price in USD',
    example: 29.99,
    minimum: 0,
  })
  @IsNumber()
  @Min(0)
  price: number;
}

// Swagger UI automatically generates:
// - Request body schema with validation rules
// - Example values
// - Response schema
// - "Try it out" functionality
```

---

### **DTO Structure and Naming:**

```typescript
// ========== DTO ORGANIZATION ==========

// Folder structure
users/
├── dto/
│   ├── create-user.dto.ts      // Input: POST /users
│   ├── update-user.dto.ts      // Input: PATCH /users/:id
│   ├── user-response.dto.ts    // Output: Single user
│   ├── user-list-response.dto.ts  // Output: List of users
│   └── user-filter.dto.ts      // Input: Query params for filtering
├── entities/
│   └── user.entity.ts          // Database model
├── controllers/
│   └── users.controller.ts
└── services/
    └── users.service.ts

// Naming conventions
CreateUserDto      // Input for creating resource
UpdateUserDto      // Input for updating resource
UserResponseDto    // Output for single resource
UserListResponseDto  // Output for list of resources
UserFilterDto      // Query parameters
UserSortDto        // Sorting parameters
PaginationDto      // Pagination parameters (shared)
```

---

### **Global ValidationPipe Setup:**

```typescript
// ========== MAIN.TS - GLOBAL VALIDATION ==========

import { ValidationPipe } from '@nestjs/common';
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Apply ValidationPipe globally
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,          // Strip properties not in DTO
      forbidNonWhitelisted: true,  // Throw error if unknown properties
      transform: true,          // Auto-transform to DTO type
      transformOptions: {
        enableImplicitConversion: true,  // Convert types (string to number)
      },
      disableErrorMessages: false,  // Include validation error messages
      validationError: {
        target: false,          // Don't include target object in error
        value: false,           // Don't include value in error
      },
    }),
  );

  await app.listen(3000);
}
bootstrap();
```

---

### **Best Practices:**

```typescript
// ✅ GOOD: Separate input and output DTOs
export class CreateUserDto { /* input */ }
export class UserResponseDto { /* output */ }

// ❌ BAD: Single DTO for both
export class UserDto { /* confused: input or output? */ }

// ✅ GOOD: Use validation decorators
export class CreateUserDto {
  @IsEmail()
  email: string;

  @MinLength(8)
  password: string;
}

// ❌ BAD: No validation
export class CreateUserDto {
  email: string;     // What if it's not an email?
  password: string;  // What if it's too short?
}

// ✅ GOOD: Hide sensitive fields in output DTO
export class UserResponseDto {
  @Expose() id: string;
  @Expose() email: string;
  // password NOT exposed
}

// ❌ BAD: Return entity directly
@Get(':id')
async findOne(@Param('id') id: string): Promise<User> {
  return this.usersRepository.findById(id);  // Exposes password!
}

// ✅ GOOD: Use whitelist to prevent over-posting
app.useGlobalPipes(
  new ValidationPipe({ whitelist: true, forbidNonWhitelisted: true }),
);

// ❌ BAD: No whitelist - vulnerable to extra fields
app.useGlobalPipes(new ValidationPipe());
```

**Key Takeaway:** DTO (Data Transfer Object) is a **plain TypeScript class** that defines **structure and validation rules** for data transferred between client and server, using **class-validator** for validation (@IsEmail, @IsNotEmpty, @Min, @Max, @IsEnum, @ValidateNested) and **class-transformer** for transformation (@Expose, @Exclude, @Transform). Use DTOs to: **validate input** (prevent invalid data, return 400 with detailed errors), **prevent over-posting** (ValidationPipe with whitelist: true strips extra fields, forbidNonWhitelisted: true rejects unknown fields like isAdmin), **hide sensitive data** (UserResponseDto exposes only id/name/email, hides password/tokens/internal fields), **transform data** (format dates to ISO strings, compute derived fields, normalize values), **ensure type safety** (TypeScript compile-time checks), **document API** (@ApiProperty generates Swagger docs with examples/validation rules), **version APIs** (V1UserDto vs V2UserDto maintain backward compatibility), and **decouple API from internal models** (Entity for database, DTO for API contract). **Structure**: place DTOs in `dto/` folder (create-user.dto.ts, update-user.dto.ts, user-response.dto.ts), use naming conventions (CreateXDto for input, XResponseDto for output, XFilterDto for query params), apply ValidationPipe globally in main.ts, separate input DTOs (create/update) from output DTOs (response), use plainToClass() with excludeExtraneousValues: true to transform entities to DTOs. **Best practice**: always use DTOs for API endpoints (never expose entities directly), validate all input (apply decorators to every field), whitelist fields (security against injection attacks), document with @ApiProperty (Swagger integration), transform sensitive data (exclude password, format dates), version DTOs when API changes, and test validation rules (unit tests for DTO validation, E2E tests for API contracts).

</details>

<details>
<summary><strong>41. Where should DTOs be defined?</strong></summary>

**Answer:**

DTOs should be defined in a **`dto/` folder** within each feature module (users/dto/, orders/dto/), keeping them **co-located with the feature** they belong to. Use **clear naming conventions** (create-user.dto.ts, update-user.dto.ts, user-response.dto.ts, user-filter.dto.ts). For **shared DTOs** used across multiple modules (PaginationDto, ResponseDto, ErrorDto), place in `shared/dto/`. **Never put DTOs in entity folder** or mix with business logic. **One DTO per file** with filename matching class name. Export from module's barrel export (index.ts) for clean imports. Structure: `src/features/users/dto/create-user.dto.ts` contains `CreateUserDto` class. For **large projects**, further organize by operation type (dto/requests/, dto/responses/) or by subdomain (dto/profile/, dto/settings/).

---

### **DTO Folder Structure:**

```
src/
├── features/
│   ├── users/
│   │   ├── dto/                         # DTOs for users feature
│   │   │   ├── create-user.dto.ts       # Input: POST /users
│   │   │   ├── update-user.dto.ts       # Input: PATCH /users/:id
│   │   │   ├── user-response.dto.ts     # Output: User data
│   │   │   ├── user-list-response.dto.ts # Output: List of users
│   │   │   ├── user-filter.dto.ts       # Input: Query params
│   │   │   ├── change-password.dto.ts   # Input: PATCH /users/:id/password
│   │   │   └── index.ts                 # Barrel export (optional)
│   │   ├── entities/
│   │   │   └── user.entity.ts           # Database model (NOT DTO!)
│   │   ├── controllers/
│   │   │   └── users.controller.ts
│   │   ├── services/
│   │   │   └── users.service.ts
│   │   └── users.module.ts
│   │
│   ├── orders/
│   │   ├── dto/
│   │   │   ├── create-order.dto.ts
│   │   │   ├── update-order.dto.ts
│   │   │   ├── order-response.dto.ts
│   │   │   ├── order-item.dto.ts        # Nested DTO
│   │   │   ├── order-filter.dto.ts
│   │   │   └── cancel-order.dto.ts
│   │   ├── entities/
│   │   │   ├── order.entity.ts
│   │   │   └── order-item.entity.ts
│   │   └── ...
│   │
│   └── products/
│       ├── dto/
│       │   ├── create-product.dto.ts
│       │   ├── update-product.dto.ts
│       │   └── product-response.dto.ts
│       └── ...
│
└── shared/                              # Shared DTOs across features
    ├── dto/
    │   ├── pagination.dto.ts            # Used by all list endpoints
    │   ├── pagination-response.dto.ts   # Paginated response wrapper
    │   ├── error-response.dto.ts        # Error response format
    │   ├── success-response.dto.ts      # Success response wrapper
    │   └── sort.dto.ts                  # Sorting parameters
    └── ...
```

---

### **Method 1: Feature-Based DTO Organization**

```typescript
// ========== FEATURE-BASED STRUCTURE ==========

// src/features/users/dto/create-user.dto.ts
export class CreateUserDto {
  @IsString()
  @IsNotEmpty()
  name: string;

  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  password: string;
}

// src/features/users/dto/update-user.dto.ts
export class UpdateUserDto {
  @IsOptional()
  @IsString()
  @MinLength(2)
  name?: string;

  @IsOptional()
  @IsEmail()
  email?: string;

  @IsOptional()
  @IsString()
  bio?: string;
}

// src/features/users/dto/user-response.dto.ts
export class UserResponseDto {
  @Expose()
  id: string;

  @Expose()
  name: string;

  @Expose()
  email: string;

  @Expose()
  @Transform(({ value }) => value?.toISOString())
  createdAt: string;
}

// src/features/users/dto/user-filter.dto.ts
export class UserFilterDto extends PaginationDto {
  @IsOptional()
  @IsString()
  search?: string;

  @IsOptional()
  @IsEnum(UserRole)
  role?: UserRole;

  @IsOptional()
  @IsBoolean()
  @Transform(({ value }) => value === 'true')
  isActive?: boolean;
}

// src/features/users/dto/index.ts (barrel export)
export * from './create-user.dto';
export * from './update-user.dto';
export * from './user-response.dto';
export * from './user-filter.dto';

// USAGE IN CONTROLLER
import {
  CreateUserDto,
  UpdateUserDto,
  UserResponseDto,
  UserFilterDto,
} from './dto';  // Clean import from barrel

@Controller('users')
export class UsersController {
  @Post()
  async create(@Body() dto: CreateUserDto): Promise<UserResponseDto> {
    return this.usersService.create(dto);
  }

  @Get()
  async findAll(@Query() filter: UserFilterDto): Promise<UserResponseDto[]> {
    return this.usersService.findAll(filter);
  }
}
```

---

### **Method 2: Shared DTOs Organization**

```typescript
// ========== SHARED DTOS (CROSS-FEATURE) ==========

// src/shared/dto/pagination.dto.ts
export class PaginationDto {
  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  @ApiProperty({ description: 'Page number', default: 1, minimum: 1 })
  page?: number = 1;

  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  @Max(100)
  @ApiProperty({ description: 'Items per page', default: 20, minimum: 1, maximum: 100 })
  limit?: number = 20;
}

// src/shared/dto/pagination-response.dto.ts
export class PaginationMetaDto {
  @ApiProperty({ description: 'Current page number' })
  page: number;

  @ApiProperty({ description: 'Items per page' })
  limit: number;

  @ApiProperty({ description: 'Total number of items' })
  total: number;

  @ApiProperty({ description: 'Total number of pages' })
  totalPages: number;

  @ApiProperty({ description: 'Has next page' })
  hasNext: boolean;

  @ApiProperty({ description: 'Has previous page' })
  hasPrev: boolean;
}

export class PaginatedResponseDto<T> {
  @ApiProperty({ description: 'Array of items', isArray: true })
  data: T[];

  @ApiProperty({ description: 'Pagination metadata', type: PaginationMetaDto })
  meta: PaginationMetaDto;
}

// src/shared/dto/sort.dto.ts
export class SortDto {
  @IsOptional()
  @IsString()
  @ApiProperty({ description: 'Sort field', required: false, example: 'createdAt' })
  sortBy?: string;

  @IsOptional()
  @IsEnum(['ASC', 'DESC'])
  @ApiProperty({ description: 'Sort order', enum: ['ASC', 'DESC'], default: 'DESC' })
  sortOrder?: 'ASC' | 'DESC' = 'DESC';
}

// src/shared/dto/error-response.dto.ts
export class ErrorResponseDto {
  @ApiProperty({ description: 'HTTP status code', example: 400 })
  statusCode: number;

  @ApiProperty({ description: 'Error message', example: 'Validation failed' })
  message: string | string[];

  @ApiProperty({ description: 'Error type', example: 'Bad Request' })
  error: string;

  @ApiProperty({ description: 'Timestamp', example: '2025-01-01T00:00:00Z' })
  timestamp: string;

  @ApiProperty({ description: 'Request path', example: '/api/users' })
  path: string;
}

// USAGE - Feature DTOs extend shared DTOs
// src/features/users/dto/user-filter.dto.ts
export class UserFilterDto extends PaginationDto {  // Extends shared DTO
  @IsOptional()
  @IsString()
  search?: string;

  @IsOptional()
  @IsEnum(UserRole)
  role?: UserRole;
}

// src/features/users/dto/user-list-response.dto.ts
export class UserListResponseDto extends PaginatedResponseDto<UserResponseDto> {
  // Inherits data: UserResponseDto[] and meta: PaginationMetaDto
}
```

---

### **Method 3: Large Project DTO Organization**

```typescript
// ========== LARGE PROJECT STRUCTURE ==========

// For large features with many DTOs, organize by operation type
src/features/users/
├── dto/
│   ├── requests/                       # Input DTOs
│   │   ├── create-user.dto.ts
│   │   ├── update-user.dto.ts
│   │   ├── change-password.dto.ts
│   │   ├── update-profile.dto.ts
│   │   └── update-settings.dto.ts
│   ├── responses/                      # Output DTOs
│   │   ├── user-response.dto.ts
│   │   ├── user-list-response.dto.ts
│   │   ├── user-profile-response.dto.ts
│   │   └── user-settings-response.dto.ts
│   ├── filters/                        # Query parameter DTOs
│   │   ├── user-filter.dto.ts
│   │   ├── user-search.dto.ts
│   │   └── user-sort.dto.ts
│   └── index.ts                        # Barrel export
└── ...

// Or organize by subdomain (DDD approach)
src/features/users/
├── dto/
│   ├── authentication/                 # Auth-related DTOs
│   │   ├── login.dto.ts
│   │   ├── register.dto.ts
│   │   ├── refresh-token.dto.ts
│   │   └── auth-response.dto.ts
│   ├── profile/                        # Profile-related DTOs
│   │   ├── update-profile.dto.ts
│   │   ├── profile-response.dto.ts
│   │   └── upload-avatar.dto.ts
│   ├── settings/                       # Settings-related DTOs
│   │   ├── update-settings.dto.ts
│   │   └── settings-response.dto.ts
│   └── index.ts
└── ...

// src/features/users/dto/authentication/login.dto.ts
export class LoginDto {
  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  password: string;
}

// src/features/users/dto/profile/update-profile.dto.ts
export class UpdateProfileDto {
  @IsOptional()
  @IsString()
  bio?: string;

  @IsOptional()
  @IsUrl()
  website?: string;

  @IsOptional()
  @IsString()
  location?: string;
}

// Barrel export for clean imports
// src/features/users/dto/index.ts
export * from './authentication/login.dto';
export * from './authentication/register.dto';
export * from './profile/update-profile.dto';
export * from './profile/profile-response.dto';

// USAGE
import { LoginDto, UpdateProfileDto } from './dto';
```

---

### **Method 4: Naming Conventions**

```typescript
// ========== DTO NAMING CONVENTIONS ==========

// INPUT DTOS (Request body)
CreateUserDto           // POST /users
UpdateUserDto           // PATCH /users/:id
CreateOrderDto          // POST /orders
AddOrderItemDto         // POST /orders/:id/items
ChangePasswordDto       // PATCH /users/:id/password
UploadAvatarDto         // POST /users/:id/avatar
AssignRoleDto           // PATCH /users/:id/role

// OUTPUT DTOS (Response)
UserResponseDto         // Single user
UserListResponseDto     // List of users
OrderResponseDto        // Single order
OrderDetailResponseDto  // Detailed order with relations
UserProfileResponseDto  // User profile view
AuthResponseDto         // Authentication response

// QUERY PARAMETER DTOS
UserFilterDto           // Filter users
ProductSearchDto        // Search products
OrderSortDto            // Sort orders
DateRangeDto            // Date range filter
PaginationDto           // Pagination params

// NESTED/COMPOSITE DTOS
OrderItemDto            // Nested within OrderDto
AddressDto              // Nested within various DTOs
ContactInfoDto          // Nested DTO
PaymentDetailsDto       // Nested DTO

// FILE NAMING (kebab-case)
create-user.dto.ts      // Matches CreateUserDto
update-user.dto.ts      // Matches UpdateUserDto
user-response.dto.ts    // Matches UserResponseDto
user-filter.dto.ts      // Matches UserFilterDto
order-item.dto.ts       // Matches OrderItemDto
```

---

### **Method 5: What NOT to Do**

```typescript
// ========== ANTI-PATTERNS ==========

// ❌ BAD: DTOs in entity folder
src/features/users/
├── entities/
│   ├── user.entity.ts
│   └── user.dto.ts          // WRONG! DTOs don't belong here

// ✅ GOOD: Separate folders
src/features/users/
├── dto/
│   └── user-response.dto.ts  // DTOs here
├── entities/
│   └── user.entity.ts        // Entities here

// ❌ BAD: Mixing DTOs with business logic
src/features/users/
├── services/
│   ├── users.service.ts
│   └── create-user.dto.ts    // WRONG! DTO in services folder

// ✅ GOOD: DTOs in dedicated folder
src/features/users/
├── dto/
│   └── create-user.dto.ts    // DTOs here
├── services/
│   └── users.service.ts      // Services here

// ❌ BAD: Multiple DTOs in one file
// user.dtos.ts
export class CreateUserDto { }
export class UpdateUserDto { }
export class UserResponseDto { }
// Hard to find, violates single responsibility

// ✅ GOOD: One DTO per file
// create-user.dto.ts
export class CreateUserDto { }

// update-user.dto.ts
export class UpdateUserDto { }

// ❌ BAD: Generic DTO names
export class UserDto { }        // Input or output?
export class Data { }           // What data?
export class Request { }        // What request?

// ✅ GOOD: Descriptive DTO names
export class CreateUserDto { }      // Clear: input for creating user
export class UserResponseDto { }    // Clear: output for user data
export class UserFilterDto { }      // Clear: query params for filtering

// ❌ BAD: DTOs in shared folder when feature-specific
src/shared/dto/
└── create-user.dto.ts    // WRONG! User-specific, not shared

// ✅ GOOD: Feature-specific DTOs in feature folder
src/features/users/dto/
└── create-user.dto.ts    // User-specific DTOs here

// ❌ BAD: No folder structure for many DTOs
src/features/users/dto/
├── create-user.dto.ts
├── update-user.dto.ts
├── user-response.dto.ts
├── user-filter.dto.ts
├── login.dto.ts
├── register.dto.ts
├── change-password.dto.ts
├── update-profile.dto.ts
├── profile-response.dto.ts
├── settings-response.dto.ts
└── ... (20+ files in flat structure)  // Hard to navigate

// ✅ GOOD: Organize by subdomain for many DTOs
src/features/users/dto/
├── authentication/
│   ├── login.dto.ts
│   ├── register.dto.ts
│   └── auth-response.dto.ts
├── profile/
│   ├── update-profile.dto.ts
│   └── profile-response.dto.ts
└── settings/
    └── settings-response.dto.ts
```

---

### **DTO Location Decision Tree:**

```
Is the DTO used by multiple features?
├─ YES → Place in src/shared/dto/
│        Examples: PaginationDto, ErrorResponseDto, SortDto
│
└─ NO → Is it feature-specific?
        └─ YES → Place in src/features/{feature}/dto/
                 Examples: CreateUserDto, OrderResponseDto
                 
                 Does the feature have 10+ DTOs?
                 ├─ YES → Organize by subdomain or operation type
                 │        src/features/users/dto/authentication/
                 │        src/features/users/dto/profile/
                 │
                 └─ NO → Flat structure in dto/ folder
                         src/features/users/dto/create-user.dto.ts
                         src/features/users/dto/user-response.dto.ts
```

---

### **Best Practices:**

```typescript
// ✅ GOOD: Co-locate DTOs with feature
src/features/users/
├── dto/
│   ├── create-user.dto.ts
│   └── user-response.dto.ts
├── entities/
├── controllers/
└── services/

// ✅ GOOD: One DTO per file
// create-user.dto.ts
export class CreateUserDto {
  name: string;
  email: string;
}

// ✅ GOOD: Use barrel exports for clean imports
// dto/index.ts
export * from './create-user.dto';
export * from './update-user.dto';

// Usage
import { CreateUserDto, UpdateUserDto } from './dto';

// ✅ GOOD: Shared DTOs in shared folder
src/shared/dto/
├── pagination.dto.ts
├── error-response.dto.ts
└── sort.dto.ts

// ✅ GOOD: Clear naming conventions
CreateUserDto           // Operation + Entity + Dto
UserResponseDto         // Entity + Purpose + Dto
UserFilterDto           // Entity + Purpose + Dto

// ✅ GOOD: Organize large DTO collections
src/features/users/dto/
├── requests/
├── responses/
└── filters/
```

**Key Takeaway:** DTOs should be defined in a **`dto/` folder within each feature module** (src/features/users/dto/, src/features/orders/dto/), keeping them **co-located with the feature** they belong to for better organization and maintainability. Use **clear naming conventions** (create-user.dto.ts for CreateUserDto, update-user.dto.ts for UpdateUserDto, user-response.dto.ts for UserResponseDto, user-filter.dto.ts for UserFilterDto). Place **shared DTOs** used across multiple modules in **src/shared/dto/** (PaginationDto, ErrorResponseDto, SortDto, PaginationMetaDto). **File structure**: one DTO per file, filename in kebab-case matching class name in PascalCase, use barrel exports (index.ts) for clean imports. For **large features** with 10+ DTOs, organize by operation type (dto/requests/, dto/responses/, dto/filters/) or by subdomain (dto/authentication/, dto/profile/, dto/settings/). **Never** put DTOs in entity folder (entities/ for database models only), mix with business logic (services/, controllers/), use generic names (UserDto, Data, Request), or place feature-specific DTOs in shared folder. **Best practice**: separate input DTOs (CreateUserDto, UpdateUserDto) from output DTOs (UserResponseDto, UserListResponseDto), extend shared DTOs when appropriate (UserFilterDto extends PaginationDto), use descriptive names that indicate purpose (ChangePasswordDto, UploadAvatarDto), export from barrel (dto/index.ts), and organize by feature first (feature/dto/), then by subdomain if needed (feature/dto/subdomain/).

</details>

<details>
<summary><strong>42. Should DTOs contain logic?</strong></summary>

**Answer:**

DTOs should **NOT contain business logic** - they are **plain data containers** for transferring data between layers. DTOs can contain: **validation decorators** (@IsEmail, @MinLength), **transformation decorators** (@Transform, @Expose, @Exclude), **default values** (property initializers), and **simple computed getters** (derived from own properties). DTOs should **never** have: **business logic** (calculating order total, applying discounts), **database operations** (queries, saves), **external dependencies** (injected services, repositories), **async operations**, or **complex transformations**. Keep DTOs **dumb** and **stateless**. Business logic belongs in **services**, data access in **repositories**, transformations in **interceptors** or **mapper classes**. Exception: simple data manipulation like `getFullName()` combining firstName + lastName is acceptable if purely derived from DTO's own fields.

---

### **What DTOs Can Contain:**

```typescript
// ========== ACCEPTABLE: VALIDATION DECORATORS ==========

export class CreateUserDto {
  @IsString()
  @IsNotEmpty()
  @MinLength(2)
  @MaxLength(50)
  name: string;  // ✅ Validation decorators OK

  @IsEmail()
  email: string;  // ✅ Validation OK

  @IsString()
  @MinLength(8)
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
  password: string;  // ✅ Complex validation OK

  @IsOptional()
  @IsInt()
  @Min(18)
  @Max(120)
  age?: number;  // ✅ Range validation OK
}

// ========== ACCEPTABLE: TRANSFORMATION DECORATORS ==========

export class UserResponseDto {
  @Expose()
  id: string;  // ✅ Expose decorator OK

  @Expose()
  name: string;

  @Exclude()
  password: string;  // ✅ Exclude decorator OK

  @Expose()
  @Transform(({ value }) => value?.toISOString())
  createdAt: string;  // ✅ Simple transformation OK

  @Expose()
  @Transform(({ value }) => value?.toUpperCase())
  country: string;  // ✅ Format transformation OK
}

// ========== ACCEPTABLE: DEFAULT VALUES ==========

export class CreateOrderDto {
  @IsUUID()
  userId: string;

  @IsArray()
  @ValidateNested({ each: true })
  @Type(() => OrderItemDto)
  items: OrderItemDto[];

  @IsOptional()
  @IsEnum(OrderStatus)
  status: OrderStatus = OrderStatus.PENDING;  // ✅ Default value OK

  @IsOptional()
  @IsEnum(PaymentMethod)
  paymentMethod: PaymentMethod = PaymentMethod.CREDIT_CARD;  // ✅ Default OK

  @IsOptional()
  @IsBoolean()
  notifyUser: boolean = true;  // ✅ Default OK
}

// ========== ACCEPTABLE: SIMPLE COMPUTED GETTERS ==========

export class UserProfileDto {
  @Expose()
  firstName: string;

  @Expose()
  lastName: string;

  @Expose()
  email: string;

  // ✅ OK: Simple getter derived from own properties
  get fullName(): string {
    return `${this.firstName} ${this.lastName}`;
  }

  // ✅ OK: Simple formatting
  get displayEmail(): string {
    return this.email.toLowerCase();
  }
}

export class AddressDto {
  @IsString()
  street: string;

  @IsString()
  city: string;

  @IsString()
  state: string;

  @IsString()
  @Matches(/^\d{5}$/)
  zipCode: string;

  // ✅ OK: Simple concatenation of own properties
  get fullAddress(): string {
    return `${this.street}, ${this.city}, ${this.state} ${this.zipCode}`;
  }
}
```

---

### **What DTOs Should NOT Contain:**

```typescript
// ========== FORBIDDEN: BUSINESS LOGIC ==========

// ❌ BAD: Business logic in DTO
export class CreateOrderDto {
  @IsArray()
  items: OrderItemDto[];

  @IsEnum(PaymentMethod)
  paymentMethod: PaymentMethod;

  // ❌ WRONG! Business logic in DTO
  calculateTotal(): number {
    let total = 0;
    for (const item of this.items) {
      total += item.price * item.quantity;
      
      // Apply discount logic
      if (item.quantity > 10) {
        total *= 0.9;  // 10% discount
      }
      
      // Add tax
      total *= 1.08;  // 8% tax
    }
    return total;
  }

  // ❌ WRONG! Complex business rules
  isEligibleForFreeShipping(): boolean {
    return this.calculateTotal() > 50 && this.paymentMethod === PaymentMethod.CREDIT_CARD;
  }
}

// ✅ GOOD: Business logic in service
@Injectable()
export class OrdersService {
  calculateTotal(items: OrderItemDto[]): number {
    let total = 0;
    for (const item of items) {
      total += item.price * item.quantity;
      
      if (item.quantity > 10) {
        total *= 0.9;
      }
      
      total *= 1.08;
    }
    return total;
  }

  isEligibleForFreeShipping(total: number, paymentMethod: PaymentMethod): boolean {
    return total > 50 && paymentMethod === PaymentMethod.CREDIT_CARD;
  }

  async create(dto: CreateOrderDto): Promise<Order> {
    const total = this.calculateTotal(dto.items);
    const freeShipping = this.isEligibleForFreeShipping(total, dto.paymentMethod);
    
    return this.ordersRepository.save({
      ...dto,
      total,
      shippingCost: freeShipping ? 0 : 10,
    });
  }
}

// ========== FORBIDDEN: DATABASE OPERATIONS ==========

// ❌ BAD: Database operations in DTO
export class CreateUserDto {
  @IsEmail()
  email: string;

  @IsString()
  password: string;

  // ❌ WRONG! Database query in DTO
  async isEmailUnique(): Promise<boolean> {
    const user = await userRepository.findByEmail(this.email);
    return !user;
  }

  // ❌ WRONG! Saving to database
  async save(): Promise<User> {
    return await userRepository.save(this);
  }
}

// ✅ GOOD: Database operations in service/repository
@Injectable()
export class UsersService {
  constructor(private readonly usersRepository: UsersRepository) {}

  async isEmailUnique(email: string): Promise<boolean> {
    const user = await this.usersRepository.findByEmail(email);
    return !user;
  }

  async create(dto: CreateUserDto): Promise<User> {
    const isUnique = await this.isEmailUnique(dto.email);
    if (!isUnique) {
      throw new ConflictException('Email already exists');
    }
    
    return this.usersRepository.save(dto);
  }
}

// ========== FORBIDDEN: EXTERNAL DEPENDENCIES ==========

// ❌ BAD: Injected dependencies in DTO
export class CreateProductDto {
  @IsString()
  name: string;

  @IsNumber()
  price: number;

  // ❌ WRONG! DTOs should not have dependencies
  constructor(
    private readonly productsService: ProductsService,  // NO!
    private readonly logger: Logger,  // NO!
  ) {}

  // ❌ WRONG! Calling external services
  async validatePrice(): Promise<boolean> {
    return this.productsService.isPriceValid(this.price);
  }
}

// ✅ GOOD: DTOs are plain objects, no dependencies
export class CreateProductDto {
  @IsString()
  name: string;

  @IsNumber()
  @Min(0)
  price: number;
  // No constructor, no dependencies
}

// Validation in service
@Injectable()
export class ProductsService {
  isPriceValid(price: number): boolean {
    return price > 0 && price < 10000;
  }

  async create(dto: CreateProductDto): Promise<Product> {
    if (!this.isPriceValid(dto.price)) {
      throw new BadRequestException('Invalid price');
    }
    return this.productsRepository.save(dto);
  }
}

// ========== FORBIDDEN: ASYNC OPERATIONS ==========

// ❌ BAD: Async operations in DTO
export class UploadImageDto {
  @IsString()
  imageBase64: string;

  // ❌ WRONG! Async operation in DTO
  async uploadToS3(): Promise<string> {
    const buffer = Buffer.from(this.imageBase64, 'base64');
    const url = await s3Client.upload(buffer);
    return url;
  }

  // ❌ WRONG! API call
  async validateImage(): Promise<boolean> {
    const response = await fetch('https://api.imagevalidator.com', {
      method: 'POST',
      body: JSON.stringify({ image: this.imageBase64 }),
    });
    return response.ok;
  }
}

// ✅ GOOD: Async operations in service
@Injectable()
export class ImagesService {
  constructor(private readonly s3Service: S3Service) {}

  async uploadToS3(imageBase64: string): Promise<string> {
    const buffer = Buffer.from(imageBase64, 'base64');
    return this.s3Service.upload(buffer);
  }

  async create(dto: UploadImageDto): Promise<Image> {
    const url = await this.uploadToS3(dto.imageBase64);
    return this.imagesRepository.save({ url });
  }
}
```

---

### **Acceptable vs Unacceptable Logic:**

```typescript
// ========== COMPARISON: ACCEPTABLE VS UNACCEPTABLE ==========

// ✅ ACCEPTABLE: Simple data formatting
export class UserDto {
  firstName: string;
  lastName: string;

  get fullName(): string {
    return `${this.firstName} ${this.lastName}`;  // Simple concatenation
  }

  get initials(): string {
    return `${this.firstName[0]}${this.lastName[0]}`.toUpperCase();  // Simple formatting
  }
}

// ❌ UNACCEPTABLE: Complex calculations
export class OrderDto {
  items: OrderItemDto[];

  // ❌ Complex business logic
  calculateTotalWithTaxAndDiscount(): number {
    let total = 0;
    for (const item of this.items) {
      let itemTotal = item.price * item.quantity;
      
      // Volume discount
      if (item.quantity >= 10) itemTotal *= 0.9;
      if (item.quantity >= 50) itemTotal *= 0.8;
      
      // Category discount
      if (item.category === 'electronics') itemTotal *= 0.95;
      
      // Tax calculation
      const tax = this.getTaxRate(item.category);
      itemTotal *= (1 + tax);
      
      total += itemTotal;
    }
    
    // Coupon discount
    if (this.couponCode) {
      total -= this.getCouponDiscount(this.couponCode, total);
    }
    
    return total;
  }
}

// ✅ ACCEPTABLE: Type conversion
export class DateRangeDto {
  @IsString()
  startDate: string;

  @IsString()
  endDate: string;

  // ✅ Simple type conversion
  get startDateObj(): Date {
    return new Date(this.startDate);
  }

  get endDateObj(): Date {
    return new Date(this.endDate);
  }
}

// ❌ UNACCEPTABLE: External API calls
export class WeatherDto {
  @IsString()
  city: string;

  // ❌ External API call
  async getCurrentWeather(): Promise<Weather> {
    const response = await fetch(`https://api.weather.com/${this.city}`);
    return response.json();
  }
}
```

---

### **Use Mapper Classes for Complex Transformations:**

```typescript
// ========== MAPPER PATTERN FOR TRANSFORMATIONS ==========

// Entity (database model)
@Entity()
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column()
  email: string;

  @Column()
  password: string;

  @Column({ nullable: true })
  avatarUrl: string;

  @Column({ type: 'timestamp' })
  createdAt: Date;

  @Column({ default: true })
  isActive: boolean;
}

// DTO (plain data)
export class UserResponseDto {
  id: string;
  fullName: string;
  email: string;
  avatarUrl: string;
  memberSince: string;
  status: string;
}

// ❌ BAD: Transformation logic in DTO
export class UserResponseDto {
  id: string;
  firstName: string;
  lastName: string;
  email: string;

  // ❌ Complex transformation
  static fromEntity(entity: User): UserResponseDto {
    const dto = new UserResponseDto();
    dto.id = entity.id;
    dto.firstName = entity.firstName;
    dto.lastName = entity.lastName;
    dto.email = entity.email;
    
    // Complex logic
    if (entity.avatarUrl) {
      dto.avatarUrl = entity.avatarUrl;
    } else {
      dto.avatarUrl = this.getDefaultAvatar(entity.email);
    }
    
    // Date formatting
    dto.memberSince = this.formatDate(entity.createdAt);
    
    // Status calculation
    dto.status = entity.isActive ? 'active' : 'inactive';
    
    return dto;
  }
}

// ✅ GOOD: Separate mapper class
@Injectable()
export class UserMapper {
  toResponseDto(entity: User): UserResponseDto {
    return {
      id: entity.id,
      fullName: `${entity.firstName} ${entity.lastName}`,
      email: entity.email,
      avatarUrl: entity.avatarUrl || this.getDefaultAvatar(entity.email),
      memberSince: entity.createdAt.toISOString().split('T')[0],
      status: entity.isActive ? 'active' : 'inactive',
    };
  }

  toResponseDtoList(entities: User[]): UserResponseDto[] {
    return entities.map(entity => this.toResponseDto(entity));
  }

  private getDefaultAvatar(email: string): string {
    // Generate default avatar URL
    return `https://api.dicebear.com/7.x/avataaars/svg?seed=${email}`;
  }
}

// USAGE IN SERVICE
@Injectable()
export class UsersService {
  constructor(
    private readonly usersRepository: UsersRepository,
    private readonly userMapper: UserMapper,
  ) {}

  async findById(id: string): Promise<UserResponseDto> {
    const user = await this.usersRepository.findById(id);
    
    if (!user) {
      throw new NotFoundException('User not found');
    }
    
    // Use mapper for transformation
    return this.userMapper.toResponseDto(user);
  }

  async findAll(): Promise<UserResponseDto[]> {
    const users = await this.usersRepository.findAll();
    return this.userMapper.toResponseDtoList(users);
  }
}
```

---

### **Best Practices:**

```typescript
// ✅ GOOD: DTOs are plain data containers
export class CreateUserDto {
  @IsString()
  @IsNotEmpty()
  name: string;

  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  password: string;
  
  // Only validation decorators, no logic
}

// ❌ BAD: DTOs with business logic
export class CreateUserDto {
  name: string;
  email: string;
  password: string;

  // Business logic doesn't belong here
  hashPassword(): string {
    return bcrypt.hashSync(this.password, 10);
  }

  generateUsername(): string {
    return this.email.split('@')[0] + Math.random().toString(36).substr(2, 5);
  }
}

// ✅ GOOD: Business logic in service
@Injectable()
export class UsersService {
  async create(dto: CreateUserDto): Promise<User> {
    // Hash password in service
    const hashedPassword = await bcrypt.hash(dto.password, 10);
    
    // Generate username in service
    const username = this.generateUsername(dto.email);
    
    return this.usersRepository.save({
      ...dto,
      password: hashedPassword,
      username,
    });
  }

  private generateUsername(email: string): string {
    return email.split('@')[0] + Math.random().toString(36).substr(2, 5);
  }
}

// ✅ GOOD: Simple computed getter (acceptable)
export class UserDto {
  firstName: string;
  lastName: string;

  get fullName(): string {
    return `${this.firstName} ${this.lastName}`;
  }
}

// ❌ BAD: Complex getter with external dependencies
export class OrderDto {
  items: OrderItemDto[];

  // Complex calculation with business rules
  get total(): number {
    return OrderCalculator.calculateWithDiscountsAndTax(this.items);
  }
}
```

**Key Takeaway:** DTOs should **NOT contain business logic** - they are **plain data containers** for transferring data between layers (client ↔ API ↔ service). DTOs **can** contain: **validation decorators** (@IsEmail, @IsNotEmpty, @MinLength, @Max, custom validators), **transformation decorators** (@Transform for simple formatting, @Expose/@Exclude for controlling visibility), **default values** (status: OrderStatus = OrderStatus.PENDING, notifyUser: boolean = true), and **simple computed getters** (get fullName() combining firstName + lastName, get displayEmail() lowercasing email - only if derived from DTO's own properties). DTOs should **never** have: **business logic** (calculating totals with discounts/tax, applying business rules, complex validations), **database operations** (queries, saves, updates, isEmailUnique() checking database), **external dependencies** (injected services via constructor, repositories, loggers, API clients), **async operations** (uploadToS3(), external API calls, file I/O), or **complex transformations** (multi-step data processing, conditional logic based on external state). Keep DTOs **dumb** (no intelligence), **stateless** (no mutable state), and **dependency-free** (no constructor injection). Business logic belongs in **services** (calculateTotal, applyDiscounts, business rules), data access in **repositories** (findByEmail, save, update), complex transformations in **mapper classes** (UserMapper.toResponseDto()), and formatting in **interceptors** (TransformInterceptor). **Exception**: simple data manipulation purely from DTO's own fields is acceptable (fullName getter, initials, date formatting), but if it requires external data or complex logic, move to service/mapper.

</details>

<details>
<summary><strong>43. What is the difference between Entity and DTO?</strong></summary>

**Answer:**

**Entity** is a **database model** that represents a **table/collection** in the database, decorated with ORM decorators (@Entity, @Column, @PrimaryGeneratedColumn, @ManyToOne, @OneToMany), contains **all fields** including sensitive data (password, tokens), has **relationships** with other entities, includes **database-specific logic** (hooks, computed columns), and is **never exposed directly** to the client. **DTO** is an **API contract** that defines **data structure** for client-server communication, decorated with validation decorators (@IsEmail, @IsNotEmpty, @Min), contains **only fields needed** for specific operation (create/update/response), has **no relationships** (uses plain objects or nested DTOs), includes **validation/transformation logic** only, and is **exposed to the client**. **Key differences**: Entity = database layer (internal, persistent, has relations, TypeORM/Mongoose decorators), DTO = API layer (external, ephemeral, no relations, class-validator decorators). **Use entities** for database operations (save, update, query, relations), **use DTOs** for API endpoints (request body, response, validation, hiding sensitive data). **Never** expose entities directly to clients (security risk, tight coupling, exposes internal structure, version lock-in).

---

### **Comparison Table:**

| Aspect | Entity | DTO |
|--------|--------|-----|
| **Purpose** | Database model (persistent storage) | Data transfer object (API contract) |
| **Layer** | Database layer (internal) | API layer (external) |
| **Decorators** | ORM decorators (@Entity, @Column, @PrimaryGeneratedColumn, @ManyToOne, @OneToMany) | Validation decorators (@IsEmail, @IsNotEmpty, @MinLength, @Transform) |
| **Fields** | All fields including sensitive data (password, tokens, internal flags) | Only fields needed for operation (filtered, secure) |
| **Relationships** | Has relationships (@ManyToOne, @OneToMany, @ManyToMany) | No relationships (plain objects, nested DTOs) |
| **Database-specific** | Yes (database hooks, computed columns, indexes) | No (database-agnostic) |
| **Exposure** | Never exposed to client (internal only) | Exposed to client (public API) |
| **Mutability** | Mutable (updated, saved to database) | Immutable (created per request, discarded after) |
| **Lifecycle** | Long-lived (persisted in database) | Short-lived (exists only during request) |
| **Location** | `entities/` folder | `dto/` folder |
| **Naming** | `User`, `Order`, `Product` (singular nouns) | `CreateUserDto`, `UserResponseDto`, `UpdateOrderDto` |
| **Validation** | Database-level validation (NOT NULL, UNIQUE, CHECK constraints) | Application-level validation (class-validator) |
| **Transformation** | Database serialization/deserialization | API serialization/deserialization (@Transform, @Expose, @Exclude) |

---

### **Method 1: Entity Example (Database Model)**

```typescript
// ========== ENTITY: DATABASE MODEL ==========

// src/features/users/entities/user.entity.ts
import { Entity, Column, PrimaryGeneratedColumn, CreateDateColumn, UpdateDateColumn, OneToMany, BeforeInsert } from 'typeorm';
import { Order } from '../../orders/entities/order.entity';
import * as bcrypt from 'bcrypt';

@Entity('users')  // Database table name
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ length: 100 })
  firstName: string;

  @Column({ length: 100 })
  lastName: string;

  @Column({ unique: true })
  email: string;

  @Column()
  password: string;  // Sensitive field (NEVER expose to client)

  @Column({ nullable: true })
  phoneNumber: string;

  @Column({ nullable: true })
  avatarUrl: string;

  @Column({ type: 'enum', enum: ['user', 'admin', 'moderator'], default: 'user' })
  role: string;

  @Column({ default: true })
  isActive: boolean;

  @Column({ default: false })
  isEmailVerified: boolean;

  @Column({ nullable: true })
  refreshToken: string;  // Sensitive field (NEVER expose to client)

  @Column({ nullable: true })
  resetPasswordToken: string;  // Sensitive field (NEVER expose to client)

  @Column({ nullable: true })
  resetPasswordExpires: Date;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  @Column({ nullable: true })
  lastLoginAt: Date;

  // RELATIONSHIPS (entities have relations)
  @OneToMany(() => Order, order => order.user)
  orders: Order[];

  // DATABASE HOOKS (entity-specific logic)
  @BeforeInsert()
  async hashPassword() {
    if (this.password) {
      this.password = await bcrypt.hash(this.password, 10);
    }
  }

  // ENTITY METHODS (business logic related to entity)
  async validatePassword(password: string): Promise<boolean> {
    return bcrypt.compare(password, this.password);
  }

  get fullName(): string {
    return `${this.firstName} ${this.lastName}`;
  }
}

// SUMMARY:
// - All fields including sensitive data (password, tokens)
// - Database-specific decorators (@Entity, @Column, @OneToMany)
// - Relationships with other entities (orders)
// - Database hooks (@BeforeInsert for password hashing)
// - Entity methods (validatePassword, fullName)
// - NEVER exposed to client directly
```

---

### **Method 2: DTO Examples (API Contracts)**

```typescript
// ========== DTOs: API CONTRACTS ==========

// INPUT DTO: Create User (POST /users)
// src/features/users/dto/create-user.dto.ts
import { IsEmail, IsString, MinLength, MaxLength, IsOptional, Matches } from 'class-validator';
import { ApiProperty } from '@nestjs/swagger';

export class CreateUserDto {
  @ApiProperty({ description: 'First name', example: 'John' })
  @IsString()
  @MinLength(2)
  @MaxLength(50)
  firstName: string;

  @ApiProperty({ description: 'Last name', example: 'Doe' })
  @IsString()
  @MinLength(2)
  @MaxLength(50)
  lastName: string;

  @ApiProperty({ description: 'Email address', example: 'john@example.com' })
  @IsEmail()
  email: string;

  @ApiProperty({ description: 'Password (min 8 chars, 1 uppercase, 1 number)', example: 'Password123' })
  @IsString()
  @MinLength(8)
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/, {
    message: 'Password must contain at least 1 uppercase letter and 1 number',
  })
  password: string;

  @ApiProperty({ description: 'Phone number', example: '+1234567890', required: false })
  @IsOptional()
  @IsString()
  phoneNumber?: string;
}

// SUMMARY:
// - Only fields needed for creating a user
// - NO sensitive fields (no tokens, no internal flags)
// - Validation decorators (@IsEmail, @MinLength, @Matches)
// - NO relationships (plain object)
// - Exposed to client (request body)

// INPUT DTO: Update User (PATCH /users/:id)
// src/features/users/dto/update-user.dto.ts
export class UpdateUserDto {
  @IsOptional()
  @IsString()
  @MinLength(2)
  firstName?: string;

  @IsOptional()
  @IsString()
  @MinLength(2)
  lastName?: string;

  @IsOptional()
  @IsString()
  phoneNumber?: string;

  @IsOptional()
  @IsString()
  avatarUrl?: string;

  // NOTE: password, email, role, tokens NOT included (separate operations)
}

// OUTPUT DTO: User Response (GET /users/:id)
// src/features/users/dto/user-response.dto.ts
import { Expose, Exclude } from 'class-transformer';

@Exclude()  // Exclude all by default
export class UserResponseDto {
  @Expose()
  id: string;

  @Expose()
  firstName: string;

  @Expose()
  lastName: string;

  @Expose()
  email: string;

  @Expose()
  phoneNumber: string;

  @Expose()
  avatarUrl: string;

  @Expose()
  role: string;

  @Expose()
  isActive: boolean;

  @Expose()
  createdAt: Date;

  // SENSITIVE FIELDS EXCLUDED:
  // password - NEVER expose
  // refreshToken - NEVER expose
  // resetPasswordToken - NEVER expose
  // updatedAt - internal field
  // lastLoginAt - internal field
  // orders - relationships not included (use separate endpoint)
}

// SUMMARY:
// - Only safe fields exposed to client
// - Sensitive fields excluded (password, tokens)
// - Transformation decorators (@Expose, @Exclude)
// - NO relationships (use separate endpoint for orders)
// - Safe to expose to client (response body)
```

---

### **Method 3: Using Entities and DTOs Together**

```typescript
// ========== SERVICE: USING ENTITIES AND DTOs ==========

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private readonly usersRepository: Repository<User>,
  ) {}

  // CREATE: DTO as input, Entity for database, DTO as output
  async create(createUserDto: CreateUserDto): Promise<UserResponseDto> {
    // 1. Receive DTO from controller (validated input)
    // 2. Create entity from DTO
    const user = this.usersRepository.create({
      firstName: createUserDto.firstName,
      lastName: createUserDto.lastName,
      email: createUserDto.email,
      password: createUserDto.password,  // Will be hashed by @BeforeInsert hook
      phoneNumber: createUserDto.phoneNumber,
    });

    // 3. Save entity to database
    const savedUser = await this.usersRepository.save(user);

    // 4. Transform entity to DTO (hide sensitive fields)
    return this.toResponseDto(savedUser);
  }

  // READ: Entity from database, DTO as output
  async findById(id: string): Promise<UserResponseDto> {
    // 1. Query database (returns entity)
    const user = await this.usersRepository.findOne({ where: { id } });

    if (!user) {
      throw new NotFoundException('User not found');
    }

    // 2. Transform entity to DTO (hide sensitive fields)
    return this.toResponseDto(user);
  }

  // UPDATE: DTO as input, Entity for database, DTO as output
  async update(id: string, updateUserDto: UpdateUserDto): Promise<UserResponseDto> {
    // 1. Find entity
    const user = await this.usersRepository.findOne({ where: { id } });

    if (!user) {
      throw new NotFoundException('User not found');
    }

    // 2. Update entity with DTO data
    Object.assign(user, updateUserDto);

    // 3. Save updated entity
    const updatedUser = await this.usersRepository.save(user);

    // 4. Transform entity to DTO
    return this.toResponseDto(updatedUser);
  }

  // HELPER: Transform entity to DTO
  private toResponseDto(user: User): UserResponseDto {
    return plainToInstance(UserResponseDto, user, {
      excludeExtraneousValues: true,  // Only @Expose() fields included
    });
  }
}

// CONTROLLER: DTOs for API, Entities never exposed
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Post()
  @ApiResponse({ type: UserResponseDto })
  async create(@Body() dto: CreateUserDto): Promise<UserResponseDto> {
    return this.usersService.create(dto);  // DTO in, DTO out
  }

  @Get(':id')
  @ApiResponse({ type: UserResponseDto })
  async findOne(@Param('id') id: string): Promise<UserResponseDto> {
    return this.usersService.findById(id);  // DTO out
  }

  @Patch(':id')
  @ApiResponse({ type: UserResponseDto })
  async update(
    @Param('id') id: string,
    @Body() dto: UpdateUserDto,
  ): Promise<UserResponseDto> {
    return this.usersService.update(id, dto);  // DTO in, DTO out
  }
}

// SUMMARY:
// - Controller receives/returns DTOs only (API layer)
// - Service uses entities for database operations (database layer)
// - Service transforms entities to DTOs before returning (security)
// - Entities NEVER leave the service layer
```

---

### **Method 4: Why Never Expose Entities Directly**

```typescript
// ========== ANTI-PATTERN: EXPOSING ENTITY ==========

// ❌ BAD: Exposing entity directly
@Controller('users')
export class UsersController {
  @Get(':id')
  async findOne(@Param('id') id: string): Promise<User> {
    return this.usersService.findById(id);  // Returns entity with ALL fields
  }
}

// PROBLEMS:
// 1. SECURITY RISK: Exposes sensitive fields
//    Response: { id, firstName, lastName, email, password, refreshToken, resetPasswordToken, ... }
//    Client can see password hashes, tokens, internal flags

// 2. TIGHT COUPLING: API tied to database schema
//    If you add a field to entity, it's automatically exposed to API
//    Breaking change for clients

// 3. OVER-FETCHING: Sends unnecessary data
//    Client gets all fields even if they don't need them
//    Increased payload size, slower response

// 4. VERSION LOCK-IN: Can't version API independently
//    Database schema changes force API changes
//    No backward compatibility

// 5. RELATIONSHIP ISSUES: Circular references
//    Entity has @OneToMany orders, which has @ManyToOne user
//    JSON.stringify() fails with circular reference error

// ✅ GOOD: Using DTO
@Controller('users')
export class UsersController {
  @Get(':id')
  async findOne(@Param('id') id: string): Promise<UserResponseDto> {
    return this.usersService.findById(id);  // Returns DTO with safe fields only
  }
}

// BENEFITS:
// 1. SECURITY: Only safe fields exposed
//    Response: { id, firstName, lastName, email, phoneNumber, avatarUrl, role, isActive, createdAt }
//    Password, tokens, internal fields hidden

// 2. DECOUPLING: API independent of database schema
//    Can change entity without breaking API
//    Can version DTOs (V1UserDto, V2UserDto)

// 3. OPTIMIZED: Only necessary data sent
//    Smaller payload, faster response
//    Client gets exactly what they need

// 4. FLEXIBILITY: Different DTOs for different operations
//    UserResponseDto (detailed)
//    UserListItemDto (summary for lists)
//    UserProfileDto (public profile)

// 5. NO CIRCULAR REFERENCES: DTOs are plain objects
//    No relationships, no circular references
//    Safe to serialize to JSON
```

---

### **Method 5: Multiple DTOs for Same Entity**

```typescript
// ========== MULTIPLE DTOs FOR DIFFERENT USE CASES ==========

// Entity: User (database model)
@Entity()
export class User {
  id: string;
  firstName: string;
  lastName: string;
  email: string;
  password: string;
  avatarUrl: string;
  bio: string;
  role: string;
  isActive: boolean;
  createdAt: Date;
  orders: Order[];
}

// DTO 1: Create User (POST /users)
export class CreateUserDto {
  firstName: string;
  lastName: string;
  email: string;
  password: string;
  // Only fields needed for registration
}

// DTO 2: Update User (PATCH /users/:id)
export class UpdateUserDto {
  firstName?: string;
  lastName?: string;
  bio?: string;
  avatarUrl?: string;
  // Only fields user can update
  // NO password, email, role (separate operations)
}

// DTO 3: User Response (GET /users/:id)
export class UserResponseDto {
  id: string;
  firstName: string;
  lastName: string;
  email: string;
  avatarUrl: string;
  role: string;
  isActive: boolean;
  createdAt: Date;
  // Safe fields for authenticated user viewing their own profile
}

// DTO 4: User List Item (GET /users)
export class UserListItemDto {
  id: string;
  firstName: string;
  lastName: string;
  avatarUrl: string;
  // Minimal fields for list view (performance)
}

// DTO 5: Public User Profile (GET /users/:id/profile)
export class PublicUserProfileDto {
  id: string;
  firstName: string;
  lastName: string;
  avatarUrl: string;
  bio: string;
  // Public profile (no email, no role, no admin fields)
}

// DTO 6: Admin User View (GET /admin/users/:id)
export class AdminUserDto {
  id: string;
  firstName: string;
  lastName: string;
  email: string;
  role: string;
  isActive: boolean;
  createdAt: Date;
  lastLoginAt: Date;
  ordersCount: number;
  // Admin view (more fields, still no password/tokens)
}

// SUMMARY:
// - ONE entity (User) in database
// - MULTIPLE DTOs for different API operations
// - Each DTO exposes only necessary fields
// - Flexible, secure, optimized
```

**Key Takeaway:** **Entity** is a **database model** representing a table/collection (User, Order, Product), decorated with **ORM decorators** (@Entity, @Column, @PrimaryGeneratedColumn, @ManyToOne, @OneToMany, @CreateDateColumn), contains **all fields** including sensitive data (password, refreshToken, resetPasswordToken, internal flags, audit fields), has **relationships** with other entities (@ManyToOne user, @OneToMany orders), includes **database-specific logic** (@BeforeInsert hooks for password hashing, computed columns, database constraints), is **mutable** (updated and saved), **long-lived** (persisted in database), located in **entities/ folder**, named with singular nouns (User, Order), and is **NEVER exposed to client** (internal use only in repositories/services). **DTO** is an **API contract** defining data structure for client-server communication (CreateUserDto, UserResponseDto, UpdateOrderDto), decorated with **validation decorators** (@IsEmail, @IsNotEmpty, @MinLength, @Matches, @Transform), contains **only fields needed** for specific operation (create DTO has firstName/lastName/email/password but no tokens, response DTO has id/firstName/lastName/email but no password, update DTO has firstName/lastName/bio but no email/password), has **no relationships** (uses plain objects or nested DTOs, not @ManyToOne), includes **validation/transformation logic** only (class-validator/class-transformer), is **immutable** (created per request, discarded after), **short-lived** (exists only during request), located in **dto/ folder**, named with operation suffix (CreateUserDto, UpdateUserDto, UserResponseDto, UserFilterDto), and is **exposed to client** (request/response bodies). **Key differences**: Entity = database layer (TypeORM/Mongoose decorators, persistent storage, has relations, all fields including sensitive, internal use), DTO = API layer (class-validator decorators, ephemeral, no relations, filtered fields, public exposure). **Never expose entities directly** because: **security risk** (exposes password/tokens/internal fields), **tight coupling** (API tied to database schema, breaking changes), **over-fetching** (sends unnecessary data, slower response), **version lock-in** (can't version API independently), **circular references** (relationships cause JSON serialization errors). **Best practice**: controllers receive/return DTOs only, services use entities for database operations, services transform entities to DTOs before returning (use plainToInstance with excludeExtraneousValues: true), create multiple DTOs for same entity (CreateUserDto, UpdateUserDto, UserResponseDto, UserListItemDto, PublicUserProfileDto, AdminUserDto), keep entity in database layer (never crosses service boundary), keep DTO in API layer (safe to expose).

</details>

<details>
<summary><strong>44. What is event-driven architecture?</strong></summary>

**Answer:**

**Event-driven architecture (EDA)** is a design pattern where components communicate through **events** (notifications of state changes) rather than direct calls. When something happens (user registered, order placed, payment processed), the component **emits an event** (UserRegisteredEvent, OrderPlacedEvent, PaymentProcessedEvent), and **multiple listeners** (event handlers) can react independently (send welcome email, update analytics, trigger notifications, update inventory) without the emitter knowing who's listening. Benefits: **loose coupling** (components don't depend on each other), **scalability** (add/remove listeners without changing emitter), **async processing** (long-running tasks don't block main flow), **audit trail** (all events logged), **flexibility** (easy to add new features by adding listeners). In NestJS, use **EventEmitter2** (`@nestjs/event-emitter`) for in-process events, **@nestjs/cqrs** for CQRS/Event Sourcing, or **message brokers** (RabbitMQ, Kafka, Redis) for distributed events across microservices.

---

### **Event-Driven Architecture Components:**

```
┌──────────────┐   emits     ┌───────────┐   notifies   ┌─────────────┐
│   Emitter    │ ─────────> │   Event   │ ───────────> │  Listener 1 │
│  (Service)   │            │   Bus     │              │  (Handler)  │
└──────────────┘            └───────────┘              └─────────────┘
                                  │                            │
                                  │ notifies                   │
                                  ▼                            │
                            ┌─────────────┐                   │
                            │  Listener 2 │                   │
                            │  (Handler)  │                   │
                            └─────────────┘                   │
                                  │                            │
                                  │ notifies                   │
                                  ▼                            │
                            ┌─────────────┐                   │
                            │  Listener 3 │                   │
                            │  (Handler)  │                   │
                            └─────────────┘                   │

Flow:
1. Emitter: Component performs action (user registers, order placed)
2. Event: Emits event with data (UserRegisteredEvent { userId, email })
3. Event Bus: Dispatches event to all registered listeners
4. Listeners: Multiple handlers react independently (email, analytics, audit log)
5. Decoupled: Emitter doesn't know/care who's listening
```

---

### **Method 1: Basic Event Emitter Setup**

```typescript
// ========== SETUP: INSTALL AND CONFIGURE ==========

// 1. Install EventEmitter2
// npm install @nestjs/event-emitter

// 2. Import in AppModule
// src/app.module.ts
import { Module } from '@nestjs/common';
import { EventEmitterModule } from '@nestjs/event-emitter';

@Module({
  imports: [
    EventEmitterModule.forRoot({
      // Event emitter configuration
      wildcard: true,          // Use wildcards in event names (order.*)
      delimiter: '.',          // Delimiter for namespaced events (order.created)
      newListener: false,      // Don't emit newListener events
      removeListener: false,   // Don't emit removeListener events
      maxListeners: 10,        // Max listeners per event (0 = unlimited)
      verboseMemoryLeak: true, // Show warning when maxListeners exceeded
      ignoreErrors: false,     // Throw errors from event handlers
    }),
  ],
})
export class AppModule {}

// ========== EVENT CLASS ==========

// src/features/users/events/user-registered.event.ts
export class UserRegisteredEvent {
  constructor(
    public readonly userId: string,
    public readonly email: string,
    public readonly name: string,
    public readonly timestamp: Date = new Date(),
  ) {}
}

// ========== EMITTER (SERVICE) ==========

// src/features/users/services/users.service.ts
import { Injectable } from '@nestjs/common';
import { EventEmitter2 } from '@nestjs/event-emitter';
import { UserRegisteredEvent } from '../events/user-registered.event';

@Injectable()
export class UsersService {
  constructor(
    private readonly usersRepository: UsersRepository,
    private readonly eventEmitter: EventEmitter2,  // Inject EventEmitter2
  ) {}

  async register(dto: RegisterDto): Promise<User> {
    // 1. Perform main operation
    const user = await this.usersRepository.create({
      email: dto.email,
      password: await bcrypt.hash(dto.password, 10),
      name: dto.name,
    });

    // 2. Emit event (notify listeners)
    this.eventEmitter.emit(
      'user.registered',  // Event name
      new UserRegisteredEvent(user.id, user.email, user.name),  // Event payload
    );

    // 3. Return immediately (listeners run asynchronously)
    return user;
  }
}

// ========== LISTENER 1: SEND WELCOME EMAIL ==========

// src/features/users/listeners/send-welcome-email.listener.ts
import { Injectable, Logger } from '@nestjs/common';
import { OnEvent } from '@nestjs/event-emitter';
import { UserRegisteredEvent } from '../events/user-registered.event';

@Injectable()
export class SendWelcomeEmailListener {
  private readonly logger = new Logger(SendWelcomeEmailListener.name);

  constructor(private readonly emailService: EmailService) {}

  @OnEvent('user.registered')  // Listen to 'user.registered' event
  async handleUserRegistered(event: UserRegisteredEvent) {
    this.logger.log(`Sending welcome email to ${event.email}`);

    try {
      await this.emailService.sendWelcomeEmail({
        to: event.email,
        name: event.name,
      });

      this.logger.log(`Welcome email sent to ${event.email}`);
    } catch (error) {
      this.logger.error(`Failed to send welcome email: ${error.message}`);
      // Don't throw - other listeners should still run
    }
  }
}

// ========== LISTENER 2: UPDATE ANALYTICS ==========

// src/features/analytics/listeners/track-user-registration.listener.ts
@Injectable()
export class TrackUserRegistrationListener {
  private readonly logger = new Logger(TrackUserRegistrationListener.name);

  constructor(private readonly analyticsService: AnalyticsService) {}

  @OnEvent('user.registered')  // Same event, different handler
  async handleUserRegistered(event: UserRegisteredEvent) {
    this.logger.log(`Tracking user registration: ${event.userId}`);

    await this.analyticsService.track({
      event: 'user_registered',
      userId: event.userId,
      properties: {
        email: event.email,
        timestamp: event.timestamp,
      },
    });
  }
}

// ========== LISTENER 3: AUDIT LOG ==========

// src/features/audit/listeners/log-user-registration.listener.ts
@Injectable()
export class LogUserRegistrationListener {
  constructor(private readonly auditLogRepository: AuditLogRepository) {}

  @OnEvent('user.registered')
  async handleUserRegistered(event: UserRegisteredEvent) {
    await this.auditLogRepository.create({
      action: 'user.registered',
      userId: event.userId,
      data: {
        email: event.email,
        name: event.name,
      },
      timestamp: event.timestamp,
    });
  }
}

// ========== MODULE REGISTRATION ==========

// src/features/users/users.module.ts
@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UsersController],
  providers: [
    UsersService,
    // Register listeners
    SendWelcomeEmailListener,
    TrackUserRegistrationListener,
    LogUserRegistrationListener,
  ],
})
export class UsersModule {}

// SUMMARY:
// - UsersService emits 'user.registered' event after creating user
// - 3 listeners react independently: email, analytics, audit log
// - Emitter doesn't know about listeners (loose coupling)
// - Easy to add new listeners without changing UsersService
```

---

### **Method 2: Multiple Events in Feature Module**

```typescript
// ========== ORDER EVENTS ==========

// src/features/orders/events/order-created.event.ts
export class OrderCreatedEvent {
  constructor(
    public readonly orderId: string,
    public readonly userId: string,
    public readonly total: number,
    public readonly items: OrderItemDto[],
  ) {}
}

// src/features/orders/events/order-paid.event.ts
export class OrderPaidEvent {
  constructor(
    public readonly orderId: string,
    public readonly userId: string,
    public readonly total: number,
    public readonly paymentMethod: string,
  ) {}
}

// src/features/orders/events/order-shipped.event.ts
export class OrderShippedEvent {
  constructor(
    public readonly orderId: string,
    public readonly userId: string,
    public readonly trackingNumber: string,
  ) {}
}

// src/features/orders/events/order-cancelled.event.ts
export class OrderCancelledEvent {
  constructor(
    public readonly orderId: string,
    public readonly userId: string,
    public readonly reason: string,
  ) {}
}

// ========== EMITTER: ORDERS SERVICE ==========

@Injectable()
export class OrdersService {
  constructor(
    private readonly ordersRepository: OrdersRepository,
    private readonly eventEmitter: EventEmitter2,
  ) {}

  async create(dto: CreateOrderDto): Promise<Order> {
    const order = await this.ordersRepository.save({
      userId: dto.userId,
      items: dto.items,
      total: this.calculateTotal(dto.items),
      status: 'pending',
    });

    // Emit order created event
    this.eventEmitter.emit(
      'order.created',
      new OrderCreatedEvent(order.id, order.userId, order.total, order.items),
    );

    return order;
  }

  async markAsPaid(orderId: string, paymentMethod: string): Promise<Order> {
    const order = await this.ordersRepository.findById(orderId);
    order.status = 'paid';
    order.paidAt = new Date();
    await this.ordersRepository.save(order);

    // Emit order paid event
    this.eventEmitter.emit(
      'order.paid',
      new OrderPaidEvent(order.id, order.userId, order.total, paymentMethod),
    );

    return order;
  }

  async ship(orderId: string, trackingNumber: string): Promise<Order> {
    const order = await this.ordersRepository.findById(orderId);
    order.status = 'shipped';
    order.shippedAt = new Date();
    order.trackingNumber = trackingNumber;
    await this.ordersRepository.save(order);

    // Emit order shipped event
    this.eventEmitter.emit(
      'order.shipped',
      new OrderShippedEvent(order.id, order.userId, trackingNumber),
    );

    return order;
  }

  async cancel(orderId: string, reason: string): Promise<Order> {
    const order = await this.ordersRepository.findById(orderId);
    order.status = 'cancelled';
    order.cancelledAt = new Date();
    await this.ordersRepository.save(order);

    // Emit order cancelled event
    this.eventEmitter.emit(
      'order.cancelled',
      new OrderCancelledEvent(order.id, order.userId, reason),
    );

    return order;
  }
}

// ========== LISTENERS FOR ORDER EVENTS ==========

// Listener 1: Send order confirmation email
@Injectable()
export class SendOrderConfirmationListener {
  @OnEvent('order.created')
  async handleOrderCreated(event: OrderCreatedEvent) {
    await this.emailService.sendOrderConfirmation(event.userId, event.orderId);
  }
}

// Listener 2: Update inventory
@Injectable()
export class UpdateInventoryListener {
  @OnEvent('order.created')
  async handleOrderCreated(event: OrderCreatedEvent) {
    for (const item of event.items) {
      await this.inventoryService.decreaseStock(item.productId, item.quantity);
    }
  }

  @OnEvent('order.cancelled')
  async handleOrderCancelled(event: OrderCancelledEvent) {
    const order = await this.ordersRepository.findById(event.orderId);
    for (const item of order.items) {
      await this.inventoryService.increaseStock(item.productId, item.quantity);
    }
  }
}

// Listener 3: Send shipping notification
@Injectable()
export class SendShippingNotificationListener {
  @OnEvent('order.shipped')
  async handleOrderShipped(event: OrderShippedEvent) {
    await this.notificationService.send({
      userId: event.userId,
      title: 'Order Shipped',
      message: `Your order has been shipped. Tracking: ${event.trackingNumber}`,
    });
  }
}

// Listener 4: Analytics tracking
@Injectable()
export class TrackOrderAnalyticsListener {
  @OnEvent('order.*')  // Wildcard: listen to all order events
  async handleOrderEvent(event: any) {
    await this.analyticsService.track({
      event: event.constructor.name,
      data: event,
    });
  }
}
```

---

### **Method 3: Namespaced Events and Wildcards**

```typescript
// ========== NAMESPACED EVENTS ==========

// Events organized by namespace: feature.action
// user.registered
// user.updated
// user.deleted
// order.created
// order.paid
// order.shipped
// order.cancelled
// payment.initiated
// payment.completed
// payment.failed

// ========== WILDCARD LISTENERS ==========

@Injectable()
export class GlobalAuditListener {
  @OnEvent('**')  // Listen to ALL events
  async handleAllEvents(event: any) {
    console.log('Event fired:', event);
    await this.auditLogRepository.create({
      event: event.constructor.name,
      data: event,
      timestamp: new Date(),
    });
  }
}

@Injectable()
export class UserEventsListener {
  @OnEvent('user.*')  // Listen to all user events (user.registered, user.updated, user.deleted)
  async handleUserEvents(event: any) {
    console.log('User event:', event);
    // Handle all user-related events
  }
}

@Injectable()
export class OrderEventsListener {
  @OnEvent('order.*')  // Listen to all order events
  async handleOrderEvents(event: any) {
    console.log('Order event:', event);
    // Handle all order-related events
  }
}

@Injectable()
export class PaymentEventsListener {
  @OnEvent('payment.{completed,failed}')  // Listen to specific events
  async handlePaymentCompletedOrFailed(event: any) {
    // Handle only payment.completed and payment.failed
  }
}
```

---

### **Method 4: Async vs Sync Event Handling**

```typescript
// ========== ASYNC EVENT HANDLING (DEFAULT) ==========

@Injectable()
export class AsyncEventListener {
  @OnEvent('order.created')  // Async by default
  async handleOrderCreated(event: OrderCreatedEvent) {
    // Runs asynchronously (doesn't block emitter)
    await this.sendEmail(event);
    await this.updateAnalytics(event);
    // Emitter doesn't wait for this to finish
  }
}

// ========== SYNC EVENT HANDLING ==========

@Injectable()
export class SyncEventListener {
  @OnEvent('order.created', { async: false })  // Synchronous
  handleOrderCreated(event: OrderCreatedEvent) {
    // Runs synchronously (blocks emitter until finished)
    console.log('Order created:', event.orderId);
    // Must complete before emitter continues
    // Use sparingly - can slow down main flow
  }
}

// ========== EMITTER BEHAVIOR ==========

@Injectable()
export class OrdersService {
  async create(dto: CreateOrderDto): Promise<Order> {
    const order = await this.ordersRepository.save(dto);

    // Emit event
    this.eventEmitter.emit('order.created', new OrderCreatedEvent(order));
    // ^ Returns immediately (async listeners don't block)
    // Sync listeners block here until finished

    console.log('Order created, returning to client');
    return order;  // Client gets response immediately
    // Async listeners still running in background
  }
}
```

---

### **Method 5: Benefits of Event-Driven Architecture**

```typescript
// ========== COMPARISON: WITHOUT EVENTS VS WITH EVENTS ==========

// ❌ WITHOUT EVENTS (Tight Coupling)
@Injectable()
export class UsersService {
  constructor(
    private readonly usersRepository: UsersRepository,
    private readonly emailService: EmailService,           // Dependency
    private readonly analyticsService: AnalyticsService,   // Dependency
    private readonly auditLogService: AuditLogService,     // Dependency
    private readonly notificationService: NotificationService, // Dependency
  ) {}

  async register(dto: RegisterDto): Promise<User> {
    // 1. Create user
    const user = await this.usersRepository.create(dto);

    // 2. Send welcome email (tightly coupled)
    try {
      await this.emailService.sendWelcomeEmail(user.email, user.name);
    } catch (error) {
      // Email fails - what to do? Rollback? Continue?
    }

    // 3. Track analytics (tightly coupled)
    try {
      await this.analyticsService.track('user_registered', { userId: user.id });
    } catch (error) {
      // Analytics fails - what to do?
    }

    // 4. Create audit log (tightly coupled)
    try {
      await this.auditLogService.log('user_registered', user);
    } catch (error) {
      // Audit log fails - what to do?
    }

    // 5. Send notification (tightly coupled)
    try {
      await this.notificationService.send(user.id, 'Welcome!');
    } catch (error) {
      // Notification fails - what to do?
    }

    return user;
  }
}

// PROBLEMS:
// - UsersService depends on 4+ services (tight coupling)
// - Hard to add new features (must modify UsersService)
// - Error handling complex (what if email fails? rollback?)
// - Slow response (waits for all operations)
// - Hard to test (must mock all dependencies)

// ✅ WITH EVENTS (Loose Coupling)
@Injectable()
export class UsersService {
  constructor(
    private readonly usersRepository: UsersRepository,
    private readonly eventEmitter: EventEmitter2,  // Only 1 dependency
  ) {}

  async register(dto: RegisterDto): Promise<User> {
    // 1. Create user
    const user = await this.usersRepository.create(dto);

    // 2. Emit event (fire and forget)
    this.eventEmitter.emit('user.registered', new UserRegisteredEvent(user));

    // 3. Return immediately
    return user;  // Fast response
    // Listeners run asynchronously in background
  }
}

// Listeners handle side effects independently
@Injectable()
export class SendWelcomeEmailListener {
  @OnEvent('user.registered')
  async handleUserRegistered(event: UserRegisteredEvent) {
    await this.emailService.sendWelcomeEmail(event.email, event.name);
    // If fails, doesn't affect other listeners or user registration
  }
}

@Injectable()
export class TrackAnalyticsListener {
  @OnEvent('user.registered')
  async handleUserRegistered(event: UserRegisteredEvent) {
    await this.analyticsService.track('user_registered', { userId: event.userId });
  }
}

// BENEFITS:
// - Loose coupling (UsersService only depends on EventEmitter)
// - Easy to add features (add new listener, don't touch UsersService)
// - Simple error handling (listener errors don't affect main flow)
// - Fast response (listeners run async, don't block)
// - Easy to test (test UsersService without mocking all dependencies)
// - Scalable (add/remove listeners without changing emitter)
```

**Key Takeaway:** **Event-driven architecture (EDA)** is a design pattern where components communicate through **events** (notifications of state changes like UserRegisteredEvent, OrderPlacedEvent, PaymentCompletedEvent) rather than direct method calls, enabling **loose coupling** and **async processing**. When action happens (user registers, order placed, payment processed), component **emits event** using EventEmitter2 (`this.eventEmitter.emit('user.registered', new UserRegisteredEvent(...))`), and **multiple listeners** decorated with @OnEvent('event.name') react independently (SendWelcomeEmailListener, TrackAnalyticsListener, LogAuditListener, UpdateInventoryListener) without emitter knowing who's listening. **Benefits**: **loose coupling** (emitter doesn't depend on listeners, only EventEmitter2), **scalability** (add/remove listeners without changing emitter), **async processing** (listeners run in background, don't block main flow, fast API response), **flexibility** (easy to add features by adding listeners, no code changes to emitter), **audit trail** (all events logged, complete history), **error isolation** (listener error doesn't affect other listeners or main flow). **Setup**: install @nestjs/event-emitter (`npm install @nestjs/event-emitter`), import EventEmitterModule.forRoot() in AppModule, create event classes (UserRegisteredEvent with payload), inject EventEmitter2 in service, emit events (`eventEmitter.emit('event.name', eventObject)`), create listeners with @OnEvent('event.name') decorator. **Event naming**: use namespaced events (user.registered, order.created, payment.completed), wildcards for broad listening (@OnEvent('order.*') for all order events, @OnEvent('**') for all events), specific event names for targeted handling. **Async vs sync**: listeners async by default (don't block emitter, run in background), sync with `@OnEvent('event', { async: false })` (blocks emitter, use sparingly). **Use cases**: send emails/notifications (don't block main flow), update analytics (track events), audit logging (record all actions), update inventory (sync stock after order), trigger workflows (multi-step processes), communicate between modules (decoupled integration). **Comparison to tight coupling**: without events = service depends on multiple services (EmailService, AnalyticsService, AuditLogService, NotificationService), all operations in service method (slow, complex error handling, hard to extend), with events = service depends only on EventEmitter2, listeners handle side effects independently (fast response, simple error handling, easy to extend by adding listeners).

</details>

<details>
<summary><strong>45. How do you implement event emitters in NestJS?</strong></summary>

**Answer:**

Implement event emitters in NestJS using **@nestjs/event-emitter** (built on EventEmitter2): **install** package (`npm install @nestjs/event-emitter`), **configure** EventEmitterModule.forRoot() in AppModule with options (wildcard: true, delimiter: '.'), **create event classes** (UserRegisteredEvent, OrderPlacedEvent with payload), **inject EventEmitter2** in service, **emit events** using `eventEmitter.emit('event.name', eventObject)` or `eventEmitter.emitAsync()` for awaiting all listeners, **create listeners** with @OnEvent('event.name') decorator on async method, **register listeners** as providers in module. Event flow: service emits → EventEmitter2 dispatches → all registered listeners execute (async by default). Use **namespaced events** (user.registered, order.created, payment.completed) for organization, **wildcards** (@OnEvent('order.*') for all order events, @OnEvent('**') for all events) for broad handling, **priorities** (@OnEvent('event', { prependListener: true })) for execution order.

---

### **Method 1: Basic Event Emitter Implementation**

```typescript
// ========== STEP 1: INSTALL AND CONFIGURE ==========

// Install package
// npm install @nestjs/event-emitter

// Configure in AppModule
// src/app.module.ts
import { Module } from '@nestjs/common';
import { EventEmitterModule } from '@nestjs/event-emitter';

@Module({
  imports: [
    EventEmitterModule.forRoot({
      wildcard: true,          // Enable wildcards in event names (order.*)
      delimiter: '.',          // Delimiter for namespaced events
      newListener: false,      // Don't emit events when listeners added
      removeListener: false,   // Don't emit events when listeners removed
      maxListeners: 10,        // Max listeners per event (0 = unlimited)
      verboseMemoryLeak: true, // Warn if maxListeners exceeded
      ignoreErrors: false,     // Throw errors from listeners
    }),
    UsersModule,
    OrdersModule,
    NotificationsModule,
  ],
})
export class AppModule {}

// ========== STEP 2: CREATE EVENT CLASS ==========

// src/features/users/events/user-registered.event.ts
export class UserRegisteredEvent {
  constructor(
    public readonly userId: string,
    public readonly email: string,
    public readonly name: string,
    public readonly registeredAt: Date = new Date(),
  ) {}
}

// ========== STEP 3: EMIT EVENT IN SERVICE ==========

// src/features/users/services/users.service.ts
import { Injectable } from '@nestjs/common';
import { EventEmitter2 } from '@nestjs/event-emitter';
import { UserRegisteredEvent } from '../events/user-registered.event';

@Injectable()
export class UsersService {
  constructor(
    private readonly usersRepository: UsersRepository,
    private readonly eventEmitter: EventEmitter2,  // Inject EventEmitter2
  ) {}

  async register(dto: RegisterDto): Promise<User> {
    // 1. Perform main operation
    const user = await this.usersRepository.create({
      email: dto.email,
      password: await bcrypt.hash(dto.password, 10),
      name: dto.name,
    });

    // 2. Emit event
    this.eventEmitter.emit(
      'user.registered',  // Event name (namespaced)
      new UserRegisteredEvent(user.id, user.email, user.name),  // Event payload
    );
    // emit() is synchronous but listeners run async by default
    // Returns immediately, doesn't wait for listeners

    // 3. Return result
    return user;
  }

  // Alternative: Wait for all listeners to complete
  async registerAndWait(dto: RegisterDto): Promise<User> {
    const user = await this.usersRepository.create(dto);

    // emitAsync() waits for all listeners to complete
    await this.eventEmitter.emitAsync(
      'user.registered',
      new UserRegisteredEvent(user.id, user.email, user.name),
    );

    return user;  // All listeners finished
  }
}

// ========== STEP 4: CREATE LISTENER ==========

// src/features/notifications/listeners/send-welcome-email.listener.ts
import { Injectable, Logger } from '@nestjs/common';
import { OnEvent } from '@nestjs/event-emitter';
import { UserRegisteredEvent } from '../../users/events/user-registered.event';

@Injectable()
export class SendWelcomeEmailListener {
  private readonly logger = new Logger(SendWelcomeEmailListener.name);

  constructor(private readonly emailService: EmailService) {}

  @OnEvent('user.registered')  // Subscribe to event
  async handleUserRegistered(event: UserRegisteredEvent) {
    this.logger.log(`Handling user registration: ${event.userId}`);

    try {
      await this.emailService.sendWelcomeEmail({
        to: event.email,
        name: event.name,
      });

      this.logger.log(`Welcome email sent to ${event.email}`);
    } catch (error) {
      this.logger.error(`Failed to send welcome email: ${error.message}`);
      // Don't throw - other listeners should still run
      // Or throw if you want to stop event propagation
    }
  }
}

// ========== STEP 5: REGISTER LISTENER IN MODULE ==========

// src/features/notifications/notifications.module.ts
@Module({
  providers: [
    NotificationsService,
    SendWelcomeEmailListener,  // Register listener as provider
  ],
})
export class NotificationsModule {}

// SUMMARY:
// 1. Install @nestjs/event-emitter
// 2. Configure EventEmitterModule.forRoot() in AppModule
// 3. Create event class (UserRegisteredEvent)
// 4. Inject EventEmitter2 in service
// 5. Emit event: eventEmitter.emit('event.name', eventObject)
// 6. Create listener with @OnEvent('event.name')
// 7. Register listener as provider in module
```

---

### **Method 2: Multiple Listeners for Same Event**

```typescript
// ========== EVENT CLASS ==========

// src/features/orders/events/order-created.event.ts
export class OrderCreatedEvent {
  constructor(
    public readonly orderId: string,
    public readonly userId: string,
    public readonly total: number,
    public readonly items: OrderItemDto[],
    public readonly createdAt: Date = new Date(),
  ) {}
}

// ========== EMITTER ==========

@Injectable()
export class OrdersService {
  constructor(
    private readonly ordersRepository: OrdersRepository,
    private readonly eventEmitter: EventEmitter2,
  ) {}

  async create(dto: CreateOrderDto): Promise<Order> {
    const order = await this.ordersRepository.save({
      userId: dto.userId,
      items: dto.items,
      total: this.calculateTotal(dto.items),
    });

    // Emit event - all listeners will react
    this.eventEmitter.emit(
      'order.created',
      new OrderCreatedEvent(order.id, order.userId, order.total, order.items),
    );

    return order;
  }
}

// ========== LISTENER 1: SEND EMAIL ==========

@Injectable()
export class SendOrderConfirmationListener {
  private readonly logger = new Logger(SendOrderConfirmationListener.name);

  @OnEvent('order.created')
  async handleOrderCreated(event: OrderCreatedEvent) {
    this.logger.log(`Sending order confirmation for order ${event.orderId}`);
    
    await this.emailService.sendOrderConfirmation({
      orderId: event.orderId,
      userId: event.userId,
      total: event.total,
      items: event.items,
    });
  }
}

// ========== LISTENER 2: UPDATE INVENTORY ==========

@Injectable()
export class UpdateInventoryListener {
  private readonly logger = new Logger(UpdateInventoryListener.name);

  @OnEvent('order.created')
  async handleOrderCreated(event: OrderCreatedEvent) {
    this.logger.log(`Updating inventory for order ${event.orderId}`);

    for (const item of event.items) {
      await this.inventoryService.decreaseStock(item.productId, item.quantity);
    }
  }
}

// ========== LISTENER 3: TRACK ANALYTICS ==========

@Injectable()
export class TrackOrderAnalyticsListener {
  @OnEvent('order.created')
  async handleOrderCreated(event: OrderCreatedEvent) {
    await this.analyticsService.track({
      event: 'order_created',
      properties: {
        orderId: event.orderId,
        userId: event.userId,
        total: event.total,
        itemsCount: event.items.length,
        timestamp: event.createdAt,
      },
    });
  }
}

// ========== LISTENER 4: AUDIT LOG ==========

@Injectable()
export class LogOrderCreationListener {
  @OnEvent('order.created')
  async handleOrderCreated(event: OrderCreatedEvent) {
    await this.auditLogRepository.create({
      action: 'order.created',
      userId: event.userId,
      resourceId: event.orderId,
      data: event,
      timestamp: event.createdAt,
    });
  }
}

// SUMMARY:
// - One event emission
// - Four listeners react independently
// - All run asynchronously
// - If one fails, others still execute
```

---

### **Method 3: Namespaced Events and Wildcards**

```typescript
// ========== NAMESPACED EVENTS ==========

// Events organized by namespace: feature.action
// user.registered
// user.updated
// user.deleted
// order.created
// order.paid
// order.shipped
// order.cancelled
// payment.initiated
// payment.completed
// payment.failed

// ========== WILDCARD LISTENERS ==========

// Listen to all events
@Injectable()
export class GlobalAuditListener {
  @OnEvent('**')  // Listen to ALL events
  async handleAllEvents(event: any) {
    await this.auditLogRepository.create({
      eventName: event.constructor.name,
      data: event,
      timestamp: new Date(),
    });
  }
}

// Listen to all user events
@Injectable()
export class UserEventsListener {
  private readonly logger = new Logger(UserEventsListener.name);

  @OnEvent('user.*')  // Listen to user.registered, user.updated, user.deleted
  async handleUserEvents(event: any) {
    this.logger.log(`User event: ${event.constructor.name}`);
    
    // Handle all user-related events
    await this.analyticsService.track({
      event: event.constructor.name,
      data: event,
    });
  }
}

// Listen to all order events
@Injectable()
export class OrderEventsListener {
  @OnEvent('order.*')  // Listen to order.created, order.paid, order.shipped, etc.
  async handleOrderEvents(event: any) {
    // Update order statistics
    await this.statisticsService.updateOrderStats(event);
  }
}

// Listen to specific payment events
@Injectable()
export class PaymentEventsListener {
  @OnEvent('payment.{completed,failed}')  // Only payment.completed and payment.failed
  async handlePaymentCompletedOrFailed(event: any) {
    if (event.constructor.name === 'PaymentCompletedEvent') {
      await this.notificationService.send('Payment successful');
    } else {
      await this.notificationService.send('Payment failed');
    }
  }
}

// ========== EMIT NAMESPACED EVENTS ==========

@Injectable()
export class UsersService {
  async register(dto: RegisterDto): Promise<User> {
    const user = await this.usersRepository.create(dto);
    this.eventEmitter.emit('user.registered', new UserRegisteredEvent(user));
    return user;
  }

  async update(id: string, dto: UpdateUserDto): Promise<User> {
    const user = await this.usersRepository.update(id, dto);
    this.eventEmitter.emit('user.updated', new UserUpdatedEvent(user));
    return user;
  }

  async delete(id: string): Promise<void> {
    await this.usersRepository.delete(id);
    this.eventEmitter.emit('user.deleted', new UserDeletedEvent(id));
  }
}
```

---

### **Method 4: Async vs Sync Event Handling**

```typescript
// ========== ASYNC EVENT HANDLING (DEFAULT) ==========

@Injectable()
export class AsyncEventListener {
  @OnEvent('order.created')  // Async by default
  async handleOrderCreated(event: OrderCreatedEvent) {
    // Runs asynchronously (doesn't block emitter)
    await this.sendEmail(event);
    await this.updateAnalytics(event);
    console.log('Async handler finished');
    // Emitter doesn't wait for this
  }
}

// ========== SYNC EVENT HANDLING ==========

@Injectable()
export class SyncEventListener {
  @OnEvent('order.created', { async: false })  // Synchronous
  handleOrderCreated(event: OrderCreatedEvent) {
    // Runs synchronously (blocks emitter until finished)
    console.log('Sync handler:', event.orderId);
    // Must complete before emitter continues
    // Use sparingly - slows down main flow
  }
}

// ========== USING emitAsync() TO WAIT FOR LISTENERS ==========

@Injectable()
export class OrdersService {
  // Don't wait for listeners (default)
  async create(dto: CreateOrderDto): Promise<Order> {
    const order = await this.ordersRepository.save(dto);
    
    this.eventEmitter.emit('order.created', new OrderCreatedEvent(order));
    // ^ Returns immediately
    
    console.log('Order created, returning');
    return order;  // Listeners still running in background
  }

  // Wait for all listeners to complete
  async createAndWait(dto: CreateOrderDto): Promise<Order> {
    const order = await this.ordersRepository.save(dto);
    
    await this.eventEmitter.emitAsync('order.created', new OrderCreatedEvent(order));
    // ^ Waits for ALL listeners to complete
    
    console.log('Order created, all listeners finished');
    return order;  // All listeners completed
  }
}

// ========== LISTENER PRIORITY ==========

@Injectable()
export class HighPriorityListener {
  @OnEvent('order.created', { prependListener: true })  // Execute first
  async handleOrderCreated(event: OrderCreatedEvent) {
    console.log('High priority handler - runs first');
  }
}

@Injectable()
export class NormalPriorityListener {
  @OnEvent('order.created')  // Normal priority
  async handleOrderCreated(event: OrderCreatedEvent) {
    console.log('Normal priority handler - runs after high priority');
  }
}
```

---

### **Method 5: Advanced Event Emitter Features**

```typescript
// ========== REMOVING LISTENERS DYNAMICALLY ==========

@Injectable()
export class DynamicListenerService {
  constructor(private readonly eventEmitter: EventEmitter2) {}

  // Add listener programmatically
  addListener() {
    this.eventEmitter.on('order.created', (event: OrderCreatedEvent) => {
      console.log('Dynamic listener:', event.orderId);
    });
  }

  // Remove specific listener
  removeListener() {
    const handler = (event: OrderCreatedEvent) => {
      console.log('Will be removed:', event.orderId);
    };

    this.eventEmitter.on('order.created', handler);
    this.eventEmitter.off('order.created', handler);  // Remove this specific handler
  }

  // Remove all listeners for event
  removeAllListeners() {
    this.eventEmitter.removeAllListeners('order.created');
  }
}

// ========== EVENT METADATA ==========

@Injectable()
export class EventMetadataService {
  constructor(private readonly eventEmitter: EventEmitter2) {}

  // Get listener count
  getListenerCount(): number {
    return this.eventEmitter.listenerCount('order.created');
  }

  // Get all event names
  getAllEventNames(): string[] {
    return this.eventEmitter.eventNames() as string[];
  }

  // Check if event has listeners
  hasListeners(eventName: string): boolean {
    return this.eventEmitter.listenerCount(eventName) > 0;
  }
}

// ========== ERROR HANDLING IN LISTENERS ==========

@Injectable()
export class ErrorHandlingListener {
  private readonly logger = new Logger(ErrorHandlingListener.name);

  @OnEvent('order.created')
  async handleOrderCreated(event: OrderCreatedEvent) {
    try {
      await this.processOrder(event);
    } catch (error) {
      this.logger.error(`Failed to process order: ${error.message}`, error.stack);
      
      // Option 1: Don't throw - let other listeners continue
      // return;
      
      // Option 2: Throw - stop event propagation
      throw error;
    }
  }
}

// ========== TYPED EVENTS WITH GENERICS ==========

// Base event class
export abstract class BaseEvent {
  public readonly timestamp: Date = new Date();
  public readonly eventId: string = uuidv4();
}

// Typed events
export class UserRegisteredEvent extends BaseEvent {
  constructor(
    public readonly userId: string,
    public readonly email: string,
  ) {
    super();
  }
}

export class OrderCreatedEvent extends BaseEvent {
  constructor(
    public readonly orderId: string,
    public readonly userId: string,
    public readonly total: number,
  ) {
    super();
  }
}

// Type-safe listener
@Injectable()
export class TypeSafeListener {
  @OnEvent('user.registered')
  handleUserRegistered(event: UserRegisteredEvent) {
    // TypeScript knows event has userId, email, timestamp, eventId
    console.log(event.userId, event.email, event.timestamp, event.eventId);
  }
}

// ========== CONDITIONAL EVENT EMISSION ==========

@Injectable()
export class ConditionalEmitterService {
  constructor(private readonly eventEmitter: EventEmitter2) {}

  async updateUser(id: string, dto: UpdateUserDto): Promise<User> {
    const oldUser = await this.usersRepository.findById(id);
    const newUser = await this.usersRepository.update(id, dto);

    // Only emit if email changed
    if (oldUser.email !== newUser.email) {
      this.eventEmitter.emit(
        'user.email.changed',
        new UserEmailChangedEvent(newUser.id, oldUser.email, newUser.email),
      );
    }

    // Only emit if role changed
    if (oldUser.role !== newUser.role) {
      this.eventEmitter.emit(
        'user.role.changed',
        new UserRoleChangedEvent(newUser.id, oldUser.role, newUser.role),
      );
    }

    // Always emit generic update event
    this.eventEmitter.emit('user.updated', new UserUpdatedEvent(newUser));

    return newUser;
  }
}
```

**Key Takeaway:** Implement event emitters in NestJS using **@nestjs/event-emitter** package (built on EventEmitter2): **install** (`npm install @nestjs/event-emitter`), **configure** EventEmitterModule.forRoot() in AppModule with options (wildcard: true for wildcards like order.*, delimiter: '.' for namespaced events, maxListeners: 10, ignoreErrors: false), **create event classes** (UserRegisteredEvent, OrderCreatedEvent with payload properties like userId/email/orderId/total), **inject EventEmitter2** in service constructor, **emit events** using `eventEmitter.emit('event.name', eventObject)` (fire and forget, returns immediately) or `eventEmitter.emitAsync('event.name', eventObject)` (waits for all listeners to complete), **create listeners** with @OnEvent('event.name') decorator on async method (method receives event object as parameter), **register listeners** as providers in module. **Event flow**: service emits event → EventEmitter2 dispatches to all registered listeners → listeners execute independently (async by default, don't block emitter). **Event naming**: use namespaced conventions (user.registered, order.created, payment.completed), wildcards for broad handling (@OnEvent('order.*') for all order events, @OnEvent('**') for all events), specific names for targeted handling (@OnEvent('user.registered')). **Listener options**: async by default (run in background, don't block emitter), sync with `@OnEvent('event', { async: false })` (blocks emitter, use sparingly), priority with `prependListener: true` (execute before others). **Multiple listeners**: one event emission triggers all registered listeners (SendEmailListener, UpdateInventoryListener, TrackAnalyticsListener, AuditLogListener), all run independently, if one fails others still execute (unless error thrown). **Advanced**: remove listeners dynamically (`eventEmitter.off()`), get listener count (`eventEmitter.listenerCount('event')`), get all event names (`eventEmitter.eventNames()`), typed events extending BaseEvent class for type safety, conditional emission (only emit if specific field changed).

</details>

<details>
<summary><strong>46. What is CQRS (Command Query Responsibility Segregation)?</strong></summary>

**Answer:**

**CQRS (Command Query Responsibility Segregation)** is a pattern that separates **write operations (commands)** from **read operations (queries)** using different models and potentially different databases. **Commands** (CreateUserCommand, UpdateOrderCommand) **change state** (create/update/delete), are handled by **CommandHandlers** with business logic validation, and return **void or success status**. **Queries** (GetUserQuery, GetOrdersQuery) **read state** without modification, are handled by **QueryHandlers** optimized for fast reads, and return **data/DTOs**. Benefits: **independent optimization** (write model for consistency, read model for performance), **scalability** (scale reads/writes separately, read replicas, caching), **flexibility** (different databases for reads/writes, denormalized read models), **maintainability** (simpler models, clear separation), **Event Sourcing integration** (commands generate events, queries use projections). Drawbacks: **complexity** (more code, two models), **eventual consistency** (read model may lag write model). Use CQRS for **complex domains**, **high-read systems**, or when **reads/writes have different requirements**.

---

### **CQRS Pattern Overview:**

```
┌─────────────────────────────────────────────────────────────┐
│                      CLIENT REQUEST                         │
└─────────────────────────────────────────────────────────────┘
                    │                    │
                    │                    │
        ┌───────────▼─────────┐   ┌─────▼───────────┐
        │      COMMAND        │   │      QUERY      │
        │  (Write Operation)  │   │  (Read Operation│
        │                     │   │                 │
        │ CreateUserCommand   │   │  GetUserQuery   │
        │ UpdateOrderCommand  │   │  GetOrdersQuery │
        │ DeleteProductCommand│   │  SearchQuery    │
        └──────────┬──────────┘   └────────┬────────┘
                   │                       │
                   │                       │
        ┌──────────▼──────────┐   ┌───────▼─────────┐
        │  COMMAND HANDLER    │   │  QUERY HANDLER  │
        │  (Business Logic)   │   │  (Data Retrieval│
        │                     │   │                 │
        │ - Validate          │   │ - Fast reads    │
        │ - Apply rules       │   │ - Optimized     │
        │ - Save to DB        │   │ - No side effects│
        │ - Emit events       │   │                 │
        └──────────┬──────────┘   └───────┬─────────┘
                   │                       │
                   │                       │
        ┌──────────▼──────────┐   ┌───────▼─────────┐
        │   WRITE MODEL       │   │   READ MODEL    │
        │ (Normalized, ACID)  │   │ (Denormalized)  │
        │                     │   │                 │
        │ PostgreSQL          │   │ MongoDB         │
        │ Entity/Aggregate    │   │ Read-optimized  │
        │ Transactional       │   │ Cached views    │
        └─────────────────────┘   └─────────────────┘

Key Concepts:
- Commands: Change state (write)
- Queries: Read state (no changes)
- Separate models for reads and writes
- Different optimization strategies
- Optional: Different databases
```

---

### **Method 1: Basic CQRS Structure**

```typescript
// ========== COMMANDS (WRITE OPERATIONS) ==========

// Command: Create User
// src/features/users/commands/impl/create-user.command.ts
export class CreateUserCommand {
  constructor(
    public readonly email: string,
    public readonly name: string,
    public readonly password: string,
  ) {}
}

// Command Handler: Business logic for creating user
// src/features/users/commands/handlers/create-user.handler.ts
import { CommandHandler, ICommandHandler } from '@nestjs/cqrs';

@CommandHandler(CreateUserCommand)
export class CreateUserHandler implements ICommandHandler<CreateUserCommand> {
  constructor(
    private readonly usersRepository: UsersRepository,
    private readonly eventBus: EventBus,
  ) {}

  async execute(command: CreateUserCommand): Promise<string> {
    // 1. Validate business rules
    const existingUser = await this.usersRepository.findByEmail(command.email);
    if (existingUser) {
      throw new ConflictException('Email already exists');
    }

    // 2. Create entity
    const user = new User(
      uuidv4(),
      command.email,
      command.name,
      await bcrypt.hash(command.password, 10),
    );

    // 3. Save to database (write model)
    await this.usersRepository.save(user);

    // 4. Emit domain event
    this.eventBus.publish(new UserCreatedEvent(user.id, user.email, user.name));

    // 5. Return result
    return user.id;  // Commands return minimal data
  }
}

// ========== QUERIES (READ OPERATIONS) ==========

// Query: Get User by ID
// src/features/users/queries/impl/get-user.query.ts
export class GetUserQuery {
  constructor(public readonly userId: string) {}
}

// Query Handler: Data retrieval without business logic
// src/features/users/queries/handlers/get-user.handler.ts
import { QueryHandler, IQueryHandler } from '@nestjs/cqrs';

@QueryHandler(GetUserQuery)
export class GetUserHandler implements IQueryHandler<GetUserQuery> {
  constructor(private readonly usersReadRepository: UsersReadRepository) {}

  async execute(query: GetUserQuery): Promise<UserDto> {
    // Simple data retrieval from read model
    const user = await this.usersReadRepository.findById(query.userId);

    if (!user) {
      throw new NotFoundException('User not found');
    }

    return {
      id: user.id,
      email: user.email,
      name: user.name,
      createdAt: user.createdAt,
    };  // Queries return data
  }
}

// ========== CONTROLLER ==========

@Controller('users')
export class UsersController {
  constructor(
    private readonly commandBus: CommandBus,  // For commands
    private readonly queryBus: QueryBus,      // For queries
  ) {}

  @Post()
  async create(@Body() dto: CreateUserDto): Promise<{ userId: string }> {
    // Execute command
    const userId = await this.commandBus.execute(
      new CreateUserCommand(dto.email, dto.name, dto.password),
    );

    return { userId };
  }

  @Get(':id')
  async findOne(@Param('id') id: string): Promise<UserDto> {
    // Execute query
    return this.queryBus.execute(new GetUserQuery(id));
  }
}

// ========== MODULE SETUP ==========

// src/features/users/users.module.ts
import { Module } from '@nestjs/common';
import { CqrsModule } from '@nestjs/cqrs';

@Module({
  imports: [
    CqrsModule,  // Import CQRS module
    TypeOrmModule.forFeature([User]),
  ],
  controllers: [UsersController],
  providers: [
    // Command handlers
    CreateUserHandler,
    UpdateUserHandler,
    DeleteUserHandler,
    
    // Query handlers
    GetUserHandler,
    GetUsersHandler,
    SearchUsersHandler,
    
    // Repositories
    UsersRepository,
    UsersReadRepository,
  ],
})
export class UsersModule {}
```

---

### **Method 2: Separate Read and Write Models**

```typescript
// ========== WRITE MODEL (NORMALIZED, TRANSACTIONAL) ==========

// Entity for writes: Normalized, relational
@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  email: string;

  @Column()
  password: string;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  // Relationships for integrity
  @OneToMany(() => Order, order => order.user)
  orders: Order[];

  @OneToMany(() => Post, post => post.author)
  posts: Post[];
}

// Write repository: ACID transactions
@Injectable()
export class UsersWriteRepository {
  constructor(
    @InjectRepository(User)
    private readonly repository: Repository<User>,
  ) {}

  async save(user: User): Promise<User> {
    return this.repository.save(user);  // Transactional
  }

  async update(id: string, data: Partial<User>): Promise<User> {
    await this.repository.update(id, data);
    return this.repository.findOne({ where: { id } });
  }

  async delete(id: string): Promise<void> {
    await this.repository.delete(id);
  }
}

// ========== READ MODEL (DENORMALIZED, OPTIMIZED) ==========

// Read model: Denormalized for fast queries
export class UserReadModel {
  id: string;
  email: string;
  fullName: string;  // Computed field
  postsCount: number;  // Cached count
  ordersCount: number;  // Cached count
  lastOrderDate: Date;  // Cached data
  createdAt: Date;
  
  // Flat structure, no relations
  // Pre-computed, cached, optimized for reads
}

// Read repository: Optimized for queries
@Injectable()
export class UsersReadRepository {
  constructor(
    // Could use different database (MongoDB, Redis, Elasticsearch)
    @InjectRepository(UserReadModel)
    private readonly repository: Repository<UserReadModel>,
  ) {}

  async findById(id: string): Promise<UserReadModel> {
    // Fast lookup, denormalized data
    return this.repository.findOne({ where: { id } });
  }

  async findAll(options: FindOptions): Promise<UserReadModel[]> {
    // Optimized for list queries
    return this.repository.find({
      skip: options.skip,
      take: options.limit,
      order: { [options.sortBy]: options.sortOrder },
    });
  }

  async search(term: string): Promise<UserReadModel[]> {
    // Full-text search on denormalized data
    return this.repository
      .createQueryBuilder('user')
      .where('user.fullName ILIKE :term', { term: `%${term}%` })
      .orWhere('user.email ILIKE :term', { term: `%${term}%` })
      .getMany();
  }
}

// ========== SYNC READ MODEL FROM WRITE MODEL ==========

// Event handler: Update read model when write model changes
@Injectable()
export class UserCreatedEventHandler implements IEventHandler<UserCreatedEvent> {
  constructor(private readonly usersReadRepository: UsersReadRepository) {}

  async handle(event: UserCreatedEvent) {
    // Update read model
    await this.usersReadRepository.save({
      id: event.userId,
      email: event.email,
      fullName: event.name,  // Denormalized
      postsCount: 0,
      ordersCount: 0,
      lastOrderDate: null,
      createdAt: event.createdAt,
    });
  }
}
```

---

### **Method 3: Commands vs Queries Comparison**

```typescript
// ========== COMMANDS (WRITES) ==========

// Command structure
export class UpdateOrderCommand {
  constructor(
    public readonly orderId: string,
    public readonly status: OrderStatus,
  ) {}
}

@CommandHandler(UpdateOrderCommand)
export class UpdateOrderHandler implements ICommandHandler<UpdateOrderCommand> {
  async execute(command: UpdateOrderCommand): Promise<void> {
    // 1. Load aggregate/entity
    const order = await this.ordersRepository.findById(command.orderId);

    // 2. Apply business rules
    if (!order.canTransitionTo(command.status)) {
      throw new BadRequestException('Invalid status transition');
    }

    // 3. Modify state
    order.updateStatus(command.status);

    // 4. Save (write model)
    await this.ordersRepository.save(order);

    // 5. Emit events
    this.eventBus.publish(new OrderUpdatedEvent(order));

    // 6. Return minimal data (void or success indicator)
    // Commands don't return domain data
  }
}

// Command characteristics:
// ✓ Changes state (create, update, delete)
// ✓ Business logic and validation
// ✓ Uses write model (normalized, transactional)
// ✓ Emits domain events
// ✓ Returns void or success status
// ✓ Can fail (throw exceptions)
// ✓ Not cached

// ========== QUERIES (READS) ==========

// Query structure
export class GetOrdersQuery {
  constructor(
    public readonly userId: string,
    public readonly page: number = 1,
    public readonly limit: number = 20,
  ) {}
}

@QueryHandler(GetOrdersQuery)
export class GetOrdersHandler implements IQueryHandler<GetOrdersQuery> {
  async execute(query: GetOrdersQuery): Promise<OrderDto[]> {
    // 1. Fetch from read model (optimized, denormalized)
    const orders = await this.ordersReadRepository.findByUserId(
      query.userId,
      query.page,
      query.limit,
    );

    // 2. Map to DTOs
    return orders.map(order => ({
      id: order.id,
      total: order.total,
      status: order.status,
      itemsCount: order.itemsCount,  // Pre-computed
      createdAt: order.createdAt,
    }));

    // 3. Return data
    // No business logic, just data retrieval
  }
}

// Query characteristics:
// ✓ Reads state (no modifications)
// ✓ No business logic (just data retrieval)
// ✓ Uses read model (denormalized, optimized)
// ✓ No events emitted
// ✓ Returns data/DTOs
// ✓ Never fails (returns empty or null)
// ✓ Can be cached aggressively
```

---

### **Method 4: Benefits and Drawbacks**

```typescript
// ========== BENEFITS ==========

// 1. INDEPENDENT OPTIMIZATION
// Write model: Normalized for consistency
@Entity()
export class Order {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @ManyToOne(() => User)
  user: User;  // Normalized relation

  @OneToMany(() => OrderItem, item => item.order)
  items: OrderItem[];  // Separate table

  @Column({ type: 'decimal' })
  total: number;
}

// Read model: Denormalized for performance
export class OrderReadModel {
  id: string;
  userId: string;
  userEmail: string;  // Denormalized (no join needed)
  userName: string;   // Denormalized
  items: {            // Embedded (no join needed)
    productId: string;
    productName: string;  // Denormalized
    quantity: number;
    price: number;
  }[];
  total: number;
  itemsCount: number;  // Pre-computed
}

// 2. SCALABILITY
// Scale writes and reads independently
// - Write database: Single master (PostgreSQL)
// - Read database: Multiple replicas (PostgreSQL read replicas)
// - Read cache: Redis for hot data
// - Search: Elasticsearch for full-text search

// 3. FLEXIBILITY
// Different databases for different needs
const commandHandlers = {
  writeDatabase: 'PostgreSQL',  // ACID transactions
};

const queryHandlers = {
  readDatabase: 'MongoDB',       // Fast queries, flexible schema
  cache: 'Redis',                // Hot data, sub-millisecond reads
  search: 'Elasticsearch',       // Full-text search
};

// 4. MAINTAINABILITY
// Simpler models (not trying to serve both reads and writes)
// Clear separation of concerns
// Easy to add new query handlers without affecting commands

// ========== DRAWBACKS ==========

// 1. COMPLEXITY
// More code: separate commands, queries, handlers, models
// - 1 operation = 1 command + 1 command handler + 1 query + 1 query handler
// - vs traditional: 1 service method

// 2. EVENTUAL CONSISTENCY
// Read model may lag behind write model
@CommandHandler(CreateOrderCommand)
export class CreateOrderHandler {
  async execute(command: CreateOrderCommand): Promise<string> {
    const order = await this.ordersRepository.save(command);
    
    // Write model updated immediately
    this.eventBus.publish(new OrderCreatedEvent(order));
    
    return order.id;
  }
}

@EventHandler(OrderCreatedEvent)
export class OrderCreatedEventHandler {
  async handle(event: OrderCreatedEvent) {
    // Read model updated after event processed (eventual consistency)
    await this.ordersReadRepository.save(event);
  }
}

// Client may not see order immediately after creation
// GET /orders/:id might return 404 briefly

// 3. LEARNING CURVE
// Team needs to understand CQRS pattern
// More architectural decisions (when to use CQRS? same DB or separate?)
```

---

### **Method 5: When to Use CQRS**

```typescript
// ========== GOOD USE CASES ==========

// ✅ Complex domain with different read/write requirements
// Example: E-commerce system
// - Writes: Complex validation, inventory checks, payment processing
// - Reads: Product listings, search, filtering, recommendations

// ✅ High read-to-write ratio
// Example: Social media (read-heavy)
// - Writes: Post creation (infrequent)
// - Reads: Feed, search, profile views (very frequent)
// - CQRS allows aggressive caching, read replicas, denormalized views

// ✅ Different optimization needs
// Example: Analytics dashboard
// - Writes: Raw event data (append-only, fast writes)
// - Reads: Aggregated stats, charts (complex queries, pre-computed)

// ✅ Event Sourcing
// CQRS pairs well with Event Sourcing
// - Commands generate events
// - Queries use projections (materialized views from events)

// ========== BAD USE CASES ==========

// ❌ Simple CRUD applications
// Example: Basic user management
// - Reads and writes are similar
// - No complex business logic
// - CQRS adds unnecessary complexity

// ❌ Low traffic applications
// Example: Internal admin tool
// - Performance not critical
// - Simplicity more important
// - Traditional service layer sufficient

// ❌ Strong consistency requirements
// Example: Banking transactions
// - Read immediately after write must be consistent
// - Eventual consistency not acceptable
// - CQRS eventual consistency problematic

// DECISION TREE
const shouldUseCQRS = {
  complexDomain: true,              // YES: different read/write needs
  highReadWriteRatio: true,         // YES: 10:1 or higher
  differentOptimization: true,      // YES: reads need denormalization
  eventSourcing: true,              // YES: CQRS natural fit
  team: {
    experienceWithCQRS: true,       // YES: team understands pattern
    willingToManageComplexity: true,// YES: complexity justified
  },
  // If most are true → Use CQRS
  // If most are false → Use traditional service layer
};
```

**Key Takeaway:** **CQRS (Command Query Responsibility Segregation)** is a pattern separating **write operations (commands)** from **read operations (queries)** using different models and handlers. **Commands** (CreateUserCommand, UpdateOrderCommand, DeleteProductCommand) **change state** (create/update/delete), are handled by **CommandHandlers** implementing ICommandHandler with **business logic validation** (check rules, apply constraints), use **write model** (normalized entities, ACID transactions, relational database), **emit domain events** (UserCreatedEvent, OrderUpdatedEvent), and return **void or minimal data** (userId, success status - not domain data). **Queries** (GetUserQuery, GetOrdersQuery, SearchProductsQuery) **read state** without modification, are handled by **QueryHandlers** implementing IQueryHandler with **no business logic** (just data retrieval), use **read model** (denormalized views, pre-computed fields like postsCount/ordersCount, optimized for fast reads, potentially different database like MongoDB/Redis/Elasticsearch), **don't emit events**, and return **data/DTOs** (complete data for client consumption). **Implementation**: install @nestjs/cqrs, import CqrsModule in module, inject CommandBus for commands (`commandBus.execute(new CreateUserCommand(...))`), inject QueryBus for queries (`queryBus.execute(new GetUserQuery(...))`), register command/query handlers as providers. **Benefits**: **independent optimization** (write model normalized for consistency, read model denormalized for performance), **scalability** (scale reads/writes separately, read replicas, aggressive caching on queries, write master + multiple read databases), **flexibility** (different databases for reads/writes, write to PostgreSQL + read from MongoDB/Redis/Elasticsearch), **maintainability** (simpler models not trying to serve both purposes, clear separation of concerns, easy to add query handlers without affecting commands), **Event Sourcing integration** (commands generate events, queries use projections). **Drawbacks**: **complexity** (more code with separate commands/queries/handlers/models, one operation needs command + command handler + query + query handler vs single service method), **eventual consistency** (read model may lag write model, client might not see order immediately after creation), **learning curve** (team needs to understand pattern and architectural decisions). **Use CQRS when**: complex domain with different read/write requirements (e-commerce with complex writes but simple reads), high read-to-write ratio (10:1 or higher like social media feeds), different optimization needs (writes need ACID, reads need denormalization), using Event Sourcing (natural fit with CQRS). **Avoid CQRS when**: simple CRUD (basic user management with similar reads/writes), low traffic (internal tools where simplicity more important), strong consistency requirements (banking where read after write must be immediately consistent, eventual consistency not acceptable).

</details>

<details>
<summary><strong>47. How do you implement CQRS in NestJS using `@nestjs/cqrs`?</strong></summary>

**Answer:**

Implement CQRS in NestJS using **@nestjs/cqrs** package: **install** (`npm install @nestjs/cqrs`), **import CqrsModule** in feature module, **create command classes** (CreateUserCommand with payload), **create CommandHandlers** with @CommandHandler decorator implementing ICommandHandler, **inject CommandBus** in controller/service and execute with `commandBus.execute(command)`, **create query classes** (GetUserQuery with parameters), **create QueryHandlers** with @QueryHandler decorator implementing IQueryHandler, **inject QueryBus** and execute with `queryBus.execute(query)`, **create event classes** for domain events, **create EventHandlers** with @EventsHandler decorator implementing IEventHandler, **inject EventBus** in command handler and publish with `eventBus.publish(event)`, **register all handlers** as providers in module. Pattern: Controller → CommandBus/QueryBus → Handler → Repository/Service → EventBus (for commands only). Commands change state and emit events, queries return data without side effects.

---

### **Method 1: Complete CQRS Setup**

```typescript
// ========== STEP 1: INSTALL AND IMPORT ==========

// Install package
// npm install @nestjs/cqrs

// Import CqrsModule in feature module
// src/features/users/users.module.ts
import { Module } from '@nestjs/common';
import { CqrsModule } from '@nestjs/cqrs';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    CqrsModule,  // Import CQRS module
    TypeOrmModule.forFeature([User]),
  ],
  controllers: [UsersController],
  providers: [
    // Command handlers
    CreateUserHandler,
    UpdateUserHandler,
    DeleteUserHandler,
    
    // Query handlers
    GetUserHandler,
    GetUsersHandler,
    
    // Event handlers
    UserCreatedHandler,
    UserUpdatedHandler,
    
    // Repositories
    UsersRepository,
  ],
})
export class UsersModule {}

// ========== STEP 2: CREATE COMMAND ==========

// src/features/users/commands/impl/create-user.command.ts
export class CreateUserCommand {
  constructor(
    public readonly email: string,
    public readonly name: string,
    public readonly password: string,
  ) {}
}

// ========== STEP 3: CREATE COMMAND HANDLER ==========

// src/features/users/commands/handlers/create-user.handler.ts
import { CommandHandler, ICommandHandler, EventBus } from '@nestjs/cqrs';
import { ConflictException } from '@nestjs/common';

@CommandHandler(CreateUserCommand)  // Decorator links handler to command
export class CreateUserHandler implements ICommandHandler<CreateUserCommand> {
  constructor(
    private readonly usersRepository: UsersRepository,
    private readonly eventBus: EventBus,  // Inject EventBus for domain events
  ) {}

  async execute(command: CreateUserCommand): Promise<string> {
    // 1. Validate business rules
    const existingUser = await this.usersRepository.findByEmail(command.email);
    if (existingUser) {
      throw new ConflictException('Email already exists');
    }

    // 2. Create and save entity
    const user = await this.usersRepository.create({
      email: command.email,
      name: command.name,
      password: await bcrypt.hash(command.password, 10),
    });

    // 3. Publish domain event
    this.eventBus.publish(
      new UserCreatedEvent(user.id, user.email, user.name),
    );

    // 4. Return result (minimal data)
    return user.id;
  }
}

// ========== STEP 4: CREATE QUERY ==========

// src/features/users/queries/impl/get-user.query.ts
export class GetUserQuery {
  constructor(public readonly userId: string) {}
}

// ========== STEP 5: CREATE QUERY HANDLER ==========

// src/features/users/queries/handlers/get-user.handler.ts
import { QueryHandler, IQueryHandler } from '@nestjs/cqrs';
import { NotFoundException } from '@nestjs/common';

@QueryHandler(GetUserQuery)  // Decorator links handler to query
export class GetUserHandler implements IQueryHandler<GetUserQuery> {
  constructor(private readonly usersRepository: UsersRepository) {}

  async execute(query: GetUserQuery): Promise<UserDto> {
    // Simple data retrieval (no business logic)
    const user = await this.usersRepository.findById(query.userId);

    if (!user) {
      throw new NotFoundException('User not found');
    }

    // Return DTO
    return {
      id: user.id,
      email: user.email,
      name: user.name,
      createdAt: user.createdAt,
    };
  }
}

// ========== STEP 6: CREATE DOMAIN EVENT ==========

// src/features/users/events/user-created.event.ts
export class UserCreatedEvent {
  constructor(
    public readonly userId: string,
    public readonly email: string,
    public readonly name: string,
    public readonly timestamp: Date = new Date(),
  ) {}
}

// ========== STEP 7: CREATE EVENT HANDLER ==========

// src/features/notifications/handlers/user-created.handler.ts
import { EventsHandler, IEventHandler } from '@nestjs/cqrs';

@EventsHandler(UserCreatedEvent)  // Decorator links handler to event
export class UserCreatedHandler implements IEventHandler<UserCreatedEvent> {
  constructor(private readonly emailService: EmailService) {}

  async handle(event: UserCreatedEvent) {
    // React to domain event
    await this.emailService.sendWelcomeEmail({
      to: event.email,
      name: event.name,
    });
  }
}

// ========== STEP 8: USE IN CONTROLLER ==========

// src/features/users/controllers/users.controller.ts
import { Controller, Post, Get, Body, Param } from '@nestjs/common';
import { CommandBus, QueryBus } from '@nestjs/cqrs';

@Controller('users')
export class UsersController {
  constructor(
    private readonly commandBus: CommandBus,  // Inject CommandBus
    private readonly queryBus: QueryBus,      // Inject QueryBus
  ) {}

  @Post()
  async create(@Body() dto: CreateUserDto): Promise<{ userId: string }> {
    // Execute command
    const userId = await this.commandBus.execute(
      new CreateUserCommand(dto.email, dto.name, dto.password),
    );

    return { userId };
  }

  @Get(':id')
  async findOne(@Param('id') id: string): Promise<UserDto> {
    // Execute query
    return this.queryBus.execute(new GetUserQuery(id));
  }
}

// SUMMARY:
// 1. Install @nestjs/cqrs
// 2. Import CqrsModule in module
// 3. Create command class (CreateUserCommand)
// 4. Create command handler (@CommandHandler, ICommandHandler)
// 5. Create query class (GetUserQuery)
// 6. Create query handler (@QueryHandler, IQueryHandler)
// 7. Create event class (UserCreatedEvent)
// 8. Create event handler (@EventsHandler, IEventHandler)
// 9. Inject CommandBus/QueryBus in controller
// 10. Execute: commandBus.execute(command), queryBus.execute(query)
```

---

### **Method 2: Complete CRUD with CQRS**

```typescript
// ========== COMMANDS ==========

// Create
export class CreateUserCommand {
  constructor(
    public readonly email: string,
    public readonly name: string,
    public readonly password: string,
  ) {}
}

// Update
export class UpdateUserCommand {
  constructor(
    public readonly userId: string,
    public readonly name?: string,
    public readonly email?: string,
  ) {}
}

// Delete
export class DeleteUserCommand {
  constructor(public readonly userId: string) {}
}

// ========== COMMAND HANDLERS ==========

@CommandHandler(CreateUserCommand)
export class CreateUserHandler implements ICommandHandler<CreateUserCommand> {
  constructor(
    private readonly repository: UsersRepository,
    private readonly eventBus: EventBus,
  ) {}

  async execute(command: CreateUserCommand): Promise<string> {
    const user = await this.repository.create(command);
    this.eventBus.publish(new UserCreatedEvent(user.id, user.email));
    return user.id;
  }
}

@CommandHandler(UpdateUserCommand)
export class UpdateUserHandler implements ICommandHandler<UpdateUserCommand> {
  constructor(
    private readonly repository: UsersRepository,
    private readonly eventBus: EventBus,
  ) {}

  async execute(command: UpdateUserCommand): Promise<void> {
    const user = await this.repository.findById(command.userId);
    
    if (!user) {
      throw new NotFoundException('User not found');
    }

    await this.repository.update(command.userId, {
      name: command.name,
      email: command.email,
    });

    this.eventBus.publish(new UserUpdatedEvent(command.userId));
  }
}

@CommandHandler(DeleteUserCommand)
export class DeleteUserHandler implements ICommandHandler<DeleteUserCommand> {
  constructor(
    private readonly repository: UsersRepository,
    private readonly eventBus: EventBus,
  ) {}

  async execute(command: DeleteUserCommand): Promise<void> {
    await this.repository.delete(command.userId);
    this.eventBus.publish(new UserDeletedEvent(command.userId));
  }
}

// ========== QUERIES ==========

// Get single user
export class GetUserQuery {
  constructor(public readonly userId: string) {}
}

// Get list of users
export class GetUsersQuery {
  constructor(
    public readonly page: number = 1,
    public readonly limit: number = 20,
    public readonly search?: string,
  ) {}
}

// Search users
export class SearchUsersQuery {
  constructor(
    public readonly term: string,
    public readonly filters?: UserFilters,
  ) {}
}

// ========== QUERY HANDLERS ==========

@QueryHandler(GetUserQuery)
export class GetUserHandler implements IQueryHandler<GetUserQuery> {
  constructor(private readonly repository: UsersRepository) {}

  async execute(query: GetUserQuery): Promise<UserDto> {
    const user = await this.repository.findById(query.userId);
    
    if (!user) {
      throw new NotFoundException('User not found');
    }

    return this.toDto(user);
  }

  private toDto(user: User): UserDto {
    return {
      id: user.id,
      email: user.email,
      name: user.name,
      createdAt: user.createdAt,
    };
  }
}

@QueryHandler(GetUsersQuery)
export class GetUsersHandler implements IQueryHandler<GetUsersQuery> {
  constructor(private readonly repository: UsersRepository) {}

  async execute(query: GetUsersQuery): Promise<PaginatedResult<UserDto>> {
    const [users, total] = await this.repository.findAndCount({
      skip: (query.page - 1) * query.limit,
      take: query.limit,
      where: query.search ? { name: Like(`%${query.search}%`) } : {},
    });

    return {
      data: users.map(this.toDto),
      meta: {
        page: query.page,
        limit: query.limit,
        total,
        totalPages: Math.ceil(total / query.limit),
      },
    };
  }

  private toDto(user: User): UserDto {
    return {
      id: user.id,
      email: user.email,
      name: user.name,
      createdAt: user.createdAt,
    };
  }
}

// ========== CONTROLLER WITH FULL CRUD ==========

@Controller('users')
export class UsersController {
  constructor(
    private readonly commandBus: CommandBus,
    private readonly queryBus: QueryBus,
  ) {}

  @Post()
  async create(@Body() dto: CreateUserDto) {
    const userId = await this.commandBus.execute(
      new CreateUserCommand(dto.email, dto.name, dto.password),
    );
    return { userId };
  }

  @Get()
  async findAll(@Query() query: GetUsersDto) {
    return this.queryBus.execute(
      new GetUsersQuery(query.page, query.limit, query.search),
    );
  }

  @Get(':id')
  async findOne(@Param('id') id: string) {
    return this.queryBus.execute(new GetUserQuery(id));
  }

  @Patch(':id')
  async update(@Param('id') id: string, @Body() dto: UpdateUserDto) {
    await this.commandBus.execute(
      new UpdateUserCommand(id, dto.name, dto.email),
    );
    return { message: 'User updated successfully' };
  }

  @Delete(':id')
  async delete(@Param('id') id: string) {
    await this.commandBus.execute(new DeleteUserCommand(id));
    return { message: 'User deleted successfully' };
  }
}
```

---

### **Method 3: Domain Events with Multiple Handlers**

```typescript
// ========== DOMAIN EVENT ==========

export class OrderCreatedEvent {
  constructor(
    public readonly orderId: string,
    public readonly userId: string,
    public readonly total: number,
    public readonly items: OrderItem[],
  ) {}
}

// ========== MULTIPLE EVENT HANDLERS ==========

// Handler 1: Send email
@EventsHandler(OrderCreatedEvent)
export class SendOrderEmailHandler implements IEventHandler<OrderCreatedEvent> {
  constructor(private readonly emailService: EmailService) {}

  async handle(event: OrderCreatedEvent) {
    await this.emailService.sendOrderConfirmation({
      userId: event.userId,
      orderId: event.orderId,
      total: event.total,
    });
  }
}

// Handler 2: Update inventory
@EventsHandler(OrderCreatedEvent)
export class UpdateInventoryHandler implements IEventHandler<OrderCreatedEvent> {
  constructor(private readonly inventoryService: InventoryService) {}

  async handle(event: OrderCreatedEvent) {
    for (const item of event.items) {
      await this.inventoryService.decreaseStock(item.productId, item.quantity);
    }
  }
}

// Handler 3: Track analytics
@EventsHandler(OrderCreatedEvent)
export class TrackOrderAnalyticsHandler implements IEventHandler<OrderCreatedEvent> {
  constructor(private readonly analyticsService: AnalyticsService) {}

  async handle(event: OrderCreatedEvent) {
    await this.analyticsService.track({
      event: 'order_created',
      properties: {
        orderId: event.orderId,
        userId: event.userId,
        total: event.total,
      },
    });
  }
}

// ========== COMMAND EMITS EVENT ==========

@CommandHandler(CreateOrderCommand)
export class CreateOrderHandler implements ICommandHandler<CreateOrderCommand> {
  constructor(
    private readonly repository: OrdersRepository,
    private readonly eventBus: EventBus,
  ) {}

  async execute(command: CreateOrderCommand): Promise<string> {
    const order = await this.repository.create(command);

    // Publish event - all 3 handlers will react
    this.eventBus.publish(
      new OrderCreatedEvent(order.id, order.userId, order.total, order.items),
    );

    return order.id;
  }
}

// ========== REGISTER IN MODULE ==========

@Module({
  imports: [CqrsModule],
  providers: [
    CreateOrderHandler,  // Command handler
    
    // All event handlers
    SendOrderEmailHandler,
    UpdateInventoryHandler,
    TrackOrderAnalyticsHandler,
  ],
})
export class OrdersModule {}
```

---

### **Method 4: Sagas for Complex Workflows**

```typescript
// ========== SAGA: COORDINATE MULTIPLE COMMANDS ==========

// Saga listens to events and dispatches commands
@Injectable()
export class OrderSaga {
  @Saga()
  orderCreated = (events$: Observable<any>): Observable<ICommand> => {
    return events$.pipe(
      ofType(OrderCreatedEvent),  // Listen to OrderCreatedEvent
      map(event => {
        // Dispatch command to process payment
        return new ProcessPaymentCommand(event.orderId, event.total);
      }),
    );
  };

  @Saga()
  paymentProcessed = (events$: Observable<any>): Observable<ICommand> => {
    return events$.pipe(
      ofType(PaymentProcessedEvent),  // Listen to PaymentProcessedEvent
      map(event => {
        // Dispatch command to ship order
        return new ShipOrderCommand(event.orderId);
      }),
    );
  };

  @Saga()
  paymentFailed = (events$: Observable<any>): Observable<ICommand> => {
    return events$.pipe(
      ofType(PaymentFailedEvent),  // Listen to PaymentFailedEvent
      map(event => {
        // Dispatch command to cancel order
        return new CancelOrderCommand(event.orderId, 'Payment failed');
      }),
    );
  };
}

// SAGA WORKFLOW:
// 1. User creates order → OrderCreatedEvent
// 2. Saga dispatches ProcessPaymentCommand
// 3. Payment succeeds → PaymentProcessedEvent → ShipOrderCommand
//    OR Payment fails → PaymentFailedEvent → CancelOrderCommand

// Register saga in module
@Module({
  imports: [CqrsModule],
  providers: [
    OrderSaga,  // Register saga
    CreateOrderHandler,
    ProcessPaymentHandler,
    ShipOrderHandler,
    CancelOrderHandler,
  ],
})
export class OrdersModule {}
```

---

### **Method 5: Advanced CQRS Patterns**

```typescript
// ========== AGGREGATE ROOT ==========

// Aggregate with domain logic
export class User extends AggregateRoot {
  constructor(
    public id: string,
    public email: string,
    public name: string,
    private password: string,
  ) {
    super();
  }

  // Domain methods
  updateProfile(name: string, email: string) {
    this.name = name;
    this.email = email;
    
    // Apply domain event
    this.apply(new UserProfileUpdatedEvent(this.id, name, email));
  }

  changePassword(newPassword: string) {
    this.password = newPassword;
    this.apply(new UserPasswordChangedEvent(this.id));
  }

  // Commit events to EventBus
  commit() {
    this.getUncommittedEvents().forEach(event => {
      this.eventBus.publish(event);
    });
    this.uncommit();
  }
}

// ========== COMMAND HANDLER WITH AGGREGATE ==========

@CommandHandler(UpdateUserProfileCommand)
export class UpdateUserProfileHandler implements ICommandHandler<UpdateUserProfileCommand> {
  constructor(private readonly repository: UsersRepository) {}

  async execute(command: UpdateUserProfileCommand): Promise<void> {
    // Load aggregate
    const user = await this.repository.findById(command.userId);

    // Execute domain logic (applies events internally)
    user.updateProfile(command.name, command.email);

    // Save aggregate
    await this.repository.save(user);

    // Commit events to EventBus
    user.commit();
  }
}

// ========== QUERY WITH CACHING ==========

@QueryHandler(GetUserQuery)
export class GetUserHandler implements IQueryHandler<GetUserQuery> {
  constructor(
    private readonly repository: UsersRepository,
    @Inject(CACHE_MANAGER) private readonly cacheManager: Cache,
  ) {}

  async execute(query: GetUserQuery): Promise<UserDto> {
    // Check cache first
    const cacheKey = `user:${query.userId}`;
    const cached = await this.cacheManager.get<UserDto>(cacheKey);

    if (cached) {
      return cached;
    }

    // Query database
    const user = await this.repository.findById(query.userId);

    if (!user) {
      throw new NotFoundException('User not found');
    }

    const dto = this.toDto(user);

    // Cache result
    await this.cacheManager.set(cacheKey, dto, 300);  // 5 minutes

    return dto;
  }
}

// ========== INVALIDATE CACHE ON COMMAND ==========

@EventsHandler(UserUpdatedEvent)
export class InvalidateUserCacheHandler implements IEventHandler<UserUpdatedEvent> {
  constructor(@Inject(CACHE_MANAGER) private readonly cacheManager: Cache) {}

  async handle(event: UserUpdatedEvent) {
    // Invalidate cache when user updated
    await this.cacheManager.del(`user:${event.userId}`);
  }
}
```

**Key Takeaway:** Implement CQRS in NestJS using **@nestjs/cqrs** package: **install** (`npm install @nestjs/cqrs`), **import CqrsModule** in feature module (UsersModule, OrdersModule), **create command classes** (CreateUserCommand with readonly properties for email/name/password, UpdateOrderCommand, DeleteProductCommand), **create CommandHandlers** decorated with @CommandHandler(CommandClass) implementing ICommandHandler<TCommand> (execute method with business logic validation, repository save, event publishing via EventBus), **inject CommandBus** in controller/service and execute with `await commandBus.execute(new CreateUserCommand(...))`, **create query classes** (GetUserQuery with userId parameter, GetUsersQuery with pagination/search), **create QueryHandlers** decorated with @QueryHandler(QueryClass) implementing IQueryHandler<TQuery> (execute method returns DTOs, no business logic, just data retrieval), **inject QueryBus** and execute with `await queryBus.execute(new GetUserQuery(id))`, **create domain event classes** (UserCreatedEvent, OrderPlacedEvent with payload), **create EventHandlers** decorated with @EventsHandler(EventClass) implementing IEventHandler<TEvent> (handle method reacts to events, multiple handlers can listen to same event), **inject EventBus** in command handler and publish with `eventBus.publish(new UserCreatedEvent(...))`, **register all handlers** as providers in module (command handlers, query handlers, event handlers, sagas). **Pattern flow**: Controller receives request → injects CommandBus/QueryBus → executes command/query → handler processes (CommandHandler validates + saves + publishes event, QueryHandler retrieves data) → EventHandlers react independently (send email, update analytics, audit log) → response returned. **Commands**: change state (create/update/delete), validate business rules (email unique, valid transitions), use write model (normalized, transactional), emit domain events (UserCreatedEvent), return void or minimal data (userId, success status), can throw exceptions. **Queries**: read state without modification, no business logic (just retrieval), use read model (denormalized, cached, optimized), don't emit events, return DTOs with complete data, never throw for "not found" (return null or empty). **Advanced**: use Sagas (@Saga decorator, listens to events and dispatches commands, coordinate workflows like OrderCreatedEvent → ProcessPaymentCommand → PaymentProcessedEvent → ShipOrderCommand), AggregateRoot for domain logic (methods apply events, commit() publishes to EventBus), cache queries (check cache first, invalidate on events), multiple event handlers for same event (all execute independently).

</details>

<details>
<summary><strong>48. What is Event Sourcing?</strong></summary>

**Answer:**

**Event Sourcing** is a pattern where **state is stored as a sequence of events** (append-only log) rather than current state. Instead of updating user record (UPDATE users SET name = 'John'), you store events (UserCreatedEvent, UserNameChangedEvent, UserEmailChangedEvent), and **rebuild current state by replaying events**. Benefits: **complete audit trail** (every change recorded, who/when/why), **time travel** (rebuild state at any point in history by replaying events up to that time), **event replay** (fix bugs by replaying events with corrected logic), **multiple projections** (same events create different read models for different use cases), **debugging** (replay production events in dev to reproduce bugs). Challenges: **complexity** (event versioning, event migration, eventual consistency), **storage** (events never deleted, grow indefinitely, need archiving strategy), **query performance** (rebuilding state expensive, need projections/snapshots). Use with **CQRS** (commands generate events, queries use projections). Implement with **event store** (database storing events), **event handlers** (update projections when events occur), **snapshots** (periodically save current state to avoid replaying all events).

---

### **Event Sourcing Concept:**

```
Traditional Approach:
┌──────────────────┐
│  UPDATE users    │  Current state only
│  SET name='John' │  Previous values lost
└──────────────────┘

Event Sourcing Approach:
┌────────────────────────────────────────────┐
│             EVENT STORE                    │
│  (Append-only log of all changes)         │
├────────────────────────────────────────────┤
│  1. UserCreatedEvent { name: 'Alice' }     │
│  2. UserNameChangedEvent { name: 'Bob' }   │
│  3. UserEmailChangedEvent { email: '...' } │
│  4. UserNameChangedEvent { name: 'John' }  │
└────────────────────────────────────────────┘
         │
         │ Replay events
         ▼
┌────────────────────┐
│  CURRENT STATE     │  Rebuilt from events
│  name: 'John'      │  All history preserved
│  email: '...'      │
└────────────────────┘

Key Concepts:
- Events are immutable (never modified or deleted)
- Current state = sum of all events
- Can rebuild any historical state
- Complete audit trail
```

---

### **Method 1: Basic Event Sourcing Implementation**

```typescript
// ========== DOMAIN EVENTS ==========

// Base event class
export abstract class DomainEvent {
  public readonly timestamp: Date = new Date();
  public readonly eventId: string = uuidv4();
  
  constructor(
    public readonly aggregateId: string,
    public readonly version: number,  // Optimistic locking
  ) {}
}

// User events
export class UserCreatedEvent extends DomainEvent {
  constructor(
    aggregateId: string,
    version: number,
    public readonly email: string,
    public readonly name: string,
  ) {
    super(aggregateId, version);
  }
}

export class UserNameChangedEvent extends DomainEvent {
  constructor(
    aggregateId: string,
    version: number,
    public readonly oldName: string,
    public readonly newName: string,
  ) {
    super(aggregateId, version);
  }
}

export class UserEmailChangedEvent extends DomainEvent {
  constructor(
    aggregateId: string,
    version: number,
    public readonly oldEmail: string,
    public readonly newEmail: string,
  ) {
    super(aggregateId, version);
  }
}

export class UserDeletedEvent extends DomainEvent {
  constructor(
    aggregateId: string,
    version: number,
    public readonly deletedAt: Date,
  ) {
    super(aggregateId, version);
  }
}

// ========== EVENT STORE ==========

// Store events in database
@Entity()
export class StoredEvent {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  aggregateId: string;  // Which aggregate (user ID, order ID)

  @Column()
  eventType: string;  // Event class name

  @Column({ type: 'jsonb' })
  payload: any;  // Event data

  @Column()
  version: number;  // Event sequence number

  @Column({ type: 'timestamp' })
  timestamp: Date;

  @Index()
  @Column()
  userId?: string;  // Who triggered event (optional)
}

// Event store service
@Injectable()
export class EventStore {
  constructor(
    @InjectRepository(StoredEvent)
    private readonly repository: Repository<StoredEvent>,
  ) {}

  // Append event to store
  async append(event: DomainEvent): Promise<void> {
    await this.repository.save({
      aggregateId: event.aggregateId,
      eventType: event.constructor.name,
      payload: event,
      version: event.version,
      timestamp: event.timestamp,
    });
  }

  // Get all events for aggregate
  async getEvents(aggregateId: string): Promise<DomainEvent[]> {
    const storedEvents = await this.repository.find({
      where: { aggregateId },
      order: { version: 'ASC' },  // Replay in order
    });

    return storedEvents.map(stored => this.deserialize(stored));
  }

  // Get events since version (for snapshots)
  async getEventsSince(aggregateId: string, version: number): Promise<DomainEvent[]> {
    const storedEvents = await this.repository.find({
      where: {
        aggregateId,
        version: MoreThan(version),
      },
      order: { version: 'ASC' },
    });

    return storedEvents.map(stored => this.deserialize(stored));
  }

  private deserialize(stored: StoredEvent): DomainEvent {
    // Reconstruct event from stored data
    const EventClass = this.getEventClass(stored.eventType);
    return Object.assign(new EventClass(), stored.payload);
  }

  private getEventClass(eventType: string): any {
    // Map event type to class
    const eventMap = {
      UserCreatedEvent,
      UserNameChangedEvent,
      UserEmailChangedEvent,
      UserDeletedEvent,
    };
    return eventMap[eventType];
  }
}

// ========== AGGREGATE ROOT WITH EVENT SOURCING ==========

export class User extends AggregateRoot {
  private id: string;
  private email: string;
  private name: string;
  private version: number = 0;
  private isDeleted: boolean = false;

  // Rebuild state from events
  static fromEvents(events: DomainEvent[]): User {
    const user = new User();
    
    for (const event of events) {
      user.applyEvent(event);
    }
    
    return user;
  }

  // Apply single event (for replay)
  private applyEvent(event: DomainEvent) {
    if (event instanceof UserCreatedEvent) {
      this.id = event.aggregateId;
      this.email = event.email;
      this.name = event.name;
      this.version = event.version;
    } else if (event instanceof UserNameChangedEvent) {
      this.name = event.newName;
      this.version = event.version;
    } else if (event instanceof UserEmailChangedEvent) {
      this.email = event.newEmail;
      this.version = event.version;
    } else if (event instanceof UserDeletedEvent) {
      this.isDeleted = true;
      this.version = event.version;
    }
  }

  // Domain method: change name
  changeName(newName: string) {
    if (this.name === newName) return;

    const event = new UserNameChangedEvent(
      this.id,
      this.version + 1,
      this.name,
      newName,
    );

    this.applyEvent(event);  // Update local state
    this.apply(event);       // Queue for persistence
  }

  // Domain method: change email
  changeEmail(newEmail: string) {
    if (this.email === newEmail) return;

    const event = new UserEmailChangedEvent(
      this.id,
      this.version + 1,
      this.email,
      newEmail,
    );

    this.applyEvent(event);
    this.apply(event);
  }
}

// ========== REPOSITORY WITH EVENT SOURCING ==========

@Injectable()
export class UsersEventSourcedRepository {
  constructor(private readonly eventStore: EventStore) {}

  // Load aggregate from events
  async findById(id: string): Promise<User> {
    const events = await this.eventStore.getEvents(id);
    
    if (events.length === 0) {
      throw new NotFoundException('User not found');
    }

    return User.fromEvents(events);
  }

  // Save aggregate (persist new events)
  async save(user: User): Promise<void> {
    const events = user.getUncommittedEvents();
    
    for (const event of events) {
      await this.eventStore.append(event as DomainEvent);
    }

    user.uncommit();
  }
}

// ========== COMMAND HANDLER WITH EVENT SOURCING ==========

@CommandHandler(ChangeUserNameCommand)
export class ChangeUserNameHandler implements ICommandHandler<ChangeUserNameCommand> {
  constructor(private readonly repository: UsersEventSourcedRepository) {}

  async execute(command: ChangeUserNameCommand): Promise<void> {
    // 1. Load aggregate from events
    const user = await this.repository.findById(command.userId);

    // 2. Execute domain logic (generates event)
    user.changeName(command.newName);

    // 3. Save (persist events)
    await this.repository.save(user);
  }
}
```

---

### **Method 2: Projections (Read Models)**

```typescript
// ========== READ MODEL (PROJECTION) ==========

// Denormalized view built from events
@Entity()
export class UserReadModel {
  @PrimaryColumn()
  id: string;

  @Column()
  email: string;

  @Column()
  name: string;

  @Column()
  createdAt: Date;

  @Column()
  updatedAt: Date;

  @Column({ default: false })
  isDeleted: boolean;

  @Column()
  version: number;  // Last processed event version
}

// ========== PROJECTION HANDLER (UPDATE READ MODEL) ==========

@Injectable()
export class UserProjectionHandler {
  constructor(
    @InjectRepository(UserReadModel)
    private readonly repository: Repository<UserReadModel>,
  ) {}

  @EventsHandler(UserCreatedEvent)
  async handleUserCreated(event: UserCreatedEvent) {
    // Create read model from event
    await this.repository.save({
      id: event.aggregateId,
      email: event.email,
      name: event.name,
      createdAt: event.timestamp,
      updatedAt: event.timestamp,
      version: event.version,
    });
  }

  @EventsHandler(UserNameChangedEvent)
  async handleUserNameChanged(event: UserNameChangedEvent) {
    // Update read model
    await this.repository.update(event.aggregateId, {
      name: event.newName,
      updatedAt: event.timestamp,
      version: event.version,
    });
  }

  @EventsHandler(UserEmailChangedEvent)
  async handleUserEmailChanged(event: UserEmailChangedEvent) {
    await this.repository.update(event.aggregateId, {
      email: event.newEmail,
      updatedAt: event.timestamp,
      version: event.version,
    });
  }

  @EventsHandler(UserDeletedEvent)
  async handleUserDeleted(event: UserDeletedEvent) {
    await this.repository.update(event.aggregateId, {
      isDeleted: true,
      updatedAt: event.timestamp,
      version: event.version,
    });
  }
}

// ========== QUERY HANDLER USES PROJECTION ==========

@QueryHandler(GetUserQuery)
export class GetUserHandler implements IQueryHandler<GetUserQuery> {
  constructor(
    @InjectRepository(UserReadModel)
    private readonly repository: Repository<UserReadModel>,
  ) {}

  async execute(query: GetUserQuery): Promise<UserDto> {
    // Query fast read model (not replaying events)
    const user = await this.repository.findOne({
      where: { id: query.userId, isDeleted: false },
    });

    if (!user) {
      throw new NotFoundException('User not found');
    }

    return {
      id: user.id,
      email: user.email,
      name: user.name,
      createdAt: user.createdAt,
    };
  }
}
```

---

### **Method 3: Snapshots for Performance**

```typescript
// ========== SNAPSHOT ==========

// Periodic save of current state
@Entity()
export class Snapshot {
  @PrimaryColumn()
  aggregateId: string;

  @Column()
  aggregateType: string;

  @Column({ type: 'jsonb' })
  state: any;  // Current state

  @Column()
  version: number;  // Version at snapshot time

  @Column({ type: 'timestamp' })
  timestamp: Date;
}

// ========== SNAPSHOT STORE ==========

@Injectable()
export class SnapshotStore {
  constructor(
    @InjectRepository(Snapshot)
    private readonly repository: Repository<Snapshot>,
  ) {}

  async save(aggregateId: string, state: any, version: number): Promise<void> {
    await this.repository.save({
      aggregateId,
      aggregateType: state.constructor.name,
      state,
      version,
      timestamp: new Date(),
    });
  }

  async get(aggregateId: string): Promise<Snapshot | null> {
    return this.repository.findOne({ where: { aggregateId } });
  }
}

// ========== REPOSITORY WITH SNAPSHOTS ==========

@Injectable()
export class UsersEventSourcedRepository {
  private readonly SNAPSHOT_INTERVAL = 100;  // Snapshot every 100 events

  constructor(
    private readonly eventStore: EventStore,
    private readonly snapshotStore: SnapshotStore,
  ) {}

  async findById(id: string): Promise<User> {
    // 1. Try to load snapshot
    const snapshot = await this.snapshotStore.get(id);
    let user: User;
    let version = 0;

    if (snapshot) {
      // Restore from snapshot
      user = Object.assign(new User(), snapshot.state);
      version = snapshot.version;
    } else {
      user = new User();
    }

    // 2. Replay events since snapshot
    const events = await this.eventStore.getEventsSince(id, version);
    
    if (events.length === 0 && !snapshot) {
      throw new NotFoundException('User not found');
    }

    for (const event of events) {
      user['applyEvent'](event);
    }

    return user;
  }

  async save(user: User): Promise<void> {
    const events = user.getUncommittedEvents();
    
    for (const event of events) {
      await this.eventStore.append(event as DomainEvent);
    }

    // Create snapshot if threshold reached
    const currentVersion = user['version'];
    if (currentVersion % this.SNAPSHOT_INTERVAL === 0) {
      await this.snapshotStore.save(user['id'], user, currentVersion);
    }

    user.uncommit();
  }
}

// PERFORMANCE COMPARISON:
// Without snapshot: Replay 10,000 events (slow)
// With snapshot: Load snapshot + replay 50 events (fast)
```

---

### **Method 4: Time Travel and Event Replay**

```typescript
// ========== TIME TRAVEL: REBUILD STATE AT SPECIFIC TIME ==========

@Injectable()
export class TimeTravelService {
  constructor(
    @InjectRepository(StoredEvent)
    private readonly eventRepository: Repository<StoredEvent>,
  ) {}

  // Get user state at specific date
  async getUserAtDate(userId: string, date: Date): Promise<User> {
    const events = await this.eventRepository.find({
      where: {
        aggregateId: userId,
        timestamp: LessThanOrEqual(date),
      },
      order: { version: 'ASC' },
    });

    if (events.length === 0) {
      throw new NotFoundException('User did not exist at that time');
    }

    const domainEvents = events.map(e => this.deserializeEvent(e));
    return User.fromEvents(domainEvents);
  }

  // Get all changes to user
  async getUserHistory(userId: string): Promise<UserHistoryEntry[]> {
    const events = await this.eventRepository.find({
      where: { aggregateId: userId },
      order: { version: 'ASC' },
    });

    return events.map(event => ({
      timestamp: event.timestamp,
      eventType: event.eventType,
      changes: this.extractChanges(event),
    }));
  }

  private extractChanges(event: StoredEvent): any {
    const payload = event.payload;
    
    if (event.eventType === 'UserNameChangedEvent') {
      return {
        field: 'name',
        oldValue: payload.oldName,
        newValue: payload.newName,
      };
    }
    
    if (event.eventType === 'UserEmailChangedEvent') {
      return {
        field: 'email',
        oldValue: payload.oldEmail,
        newValue: payload.newEmail,
      };
    }
    
    return payload;
  }
}

// ========== USAGE: TIME TRAVEL ==========

@Controller('users')
export class UsersController {
  constructor(private readonly timeTravelService: TimeTravelService) {}

  @Get(':id/history')
  async getHistory(@Param('id') id: string) {
    return this.timeTravelService.getUserHistory(id);
  }

  @Get(':id/at/:date')
  async getUserAtDate(@Param('id') id: string, @Param('date') date: string) {
    const targetDate = new Date(date);
    return this.timeTravelService.getUserAtDate(id, targetDate);
  }
}

// EXAMPLE RESPONSE:
// GET /users/123/history
// [
//   {
//     timestamp: '2025-01-01T10:00:00Z',
//     eventType: 'UserCreatedEvent',
//     changes: { email: 'alice@example.com', name: 'Alice' }
//   },
//   {
//     timestamp: '2025-01-15T14:30:00Z',
//     eventType: 'UserNameChangedEvent',
//     changes: { field: 'name', oldValue: 'Alice', newValue: 'Alice Smith' }
//   },
//   {
//     timestamp: '2025-02-20T09:15:00Z',
//     eventType: 'UserEmailChangedEvent',
//     changes: { field: 'email', oldValue: 'alice@example.com', newValue: 'alice.smith@example.com' }
//   }
// ]

// GET /users/123/at/2025-01-20
// { email: 'alice@example.com', name: 'Alice Smith' }  // State on Jan 20
```

---

### **Method 5: Benefits and Challenges**

```typescript
// ========== BENEFITS ==========

// 1. COMPLETE AUDIT TRAIL
// Every change recorded with timestamp and user
const auditTrail = await eventStore.getEvents('user-123');
// [
//   UserCreatedEvent { userId: 'admin', timestamp: '2025-01-01' },
//   UserNameChangedEvent { userId: 'admin', timestamp: '2025-01-15' },
//   UserEmailChangedEvent { userId: 'user-123', timestamp: '2025-02-20' }
// ]

// 2. TIME TRAVEL
// Rebuild state at any point
const userIn2024 = await getUserAtDate('user-123', new Date('2024-12-31'));
const userIn2025 = await getUserAtDate('user-123', new Date('2025-01-31'));

// 3. EVENT REPLAY
// Fix bug and replay events
async function fixBugAndReplay() {
  // Load all events
  const events = await eventStore.getAllEvents();
  
  // Clear projections
  await clearAllProjections();
  
  // Replay with fixed logic
  for (const event of events) {
    await projectionHandler.handle(event);  // Fixed handler
  }
}

// 4. MULTIPLE PROJECTIONS
// Same events, different views
@EventsHandler(OrderCreatedEvent)
class OrderSummaryProjection {
  // Projection 1: Order summary
  async handle(event: OrderCreatedEvent) {
    await orderSummaryRepository.save({ orderId: event.orderId, total: event.total });
  }
}

@EventsHandler(OrderCreatedEvent)
class RevenueProjection {
  // Projection 2: Revenue analytics
  async handle(event: OrderCreatedEvent) {
    await revenueRepository.increment(event.timestamp.getMonth(), event.total);
  }
}

@EventsHandler(OrderCreatedEvent)
class InventoryProjection {
  // Projection 3: Inventory tracking
  async handle(event: OrderCreatedEvent) {
    for (const item of event.items) {
      await inventoryRepository.decreaseStock(item.productId, item.quantity);
    }
  }
}

// ========== CHALLENGES ==========

// 1. EVENT VERSIONING
// Events change over time
export class UserCreatedEventV1 {
  constructor(public readonly email: string) {}
}

export class UserCreatedEventV2 {
  constructor(
    public readonly email: string,
    public readonly name: string,  // New field
  ) {}
}

// Need upcasting (convert old events to new format)
function upcastEvent(event: any): UserCreatedEventV2 {
  if (event.version === 1) {
    return new UserCreatedEventV2(event.email, 'Unknown');  // Default name
  }
  return event;
}

// 2. STORAGE GROWTH
// Events never deleted, grow indefinitely
// Solution: Archiving old events, snapshots

// 3. EVENTUAL CONSISTENCY
// Projections updated asynchronously
// Read model may lag behind events

// 4. COMPLEXITY
// More code: event store, projections, snapshots, event versioning
// Harder to understand than CRUD
```

**Key Takeaway:** **Event Sourcing** is a pattern where **state is stored as sequence of immutable events** (append-only log) rather than current state, so instead of UPDATE users SET name='John', you store events (UserCreatedEvent, UserNameChangedEvent, UserEmailChangedEvent) and **rebuild current state by replaying events** (User.fromEvents(events)). **Benefits**: **complete audit trail** (every change recorded with who/when/what, full history forever), **time travel** (rebuild state at any point in history by replaying events up to specific timestamp, getUserAtDate('user-123', '2024-12-31') returns user state on that date), **event replay** (fix bugs by clearing projections and replaying events with corrected logic, reproduce production issues in dev), **multiple projections** (same events create different read models: OrderSummaryProjection for orders list, RevenueProjection for analytics dashboard, InventoryProjection for stock tracking), **debugging** (see exact sequence of events leading to bug, replay in dev environment). **Implementation**: **event store** (database table storing events with aggregateId/eventType/payload/version/timestamp, events never modified or deleted), **aggregate root** (User.fromEvents() rebuilds state by replaying events, domain methods like changeName() generate new events), **repository** (findById loads events and replays, save appends new events to store), **projections** (read models updated by event handlers, UserProjectionHandler listens to events and updates UserReadModel for fast queries), **snapshots** (periodically save current state every N events to avoid replaying thousands of events, load snapshot + replay events since snapshot). **Challenges**: **event versioning** (events change over time, need upcasting to convert V1 to V2, add new fields with defaults, migrate old events), **storage growth** (events never deleted, grow indefinitely, need archiving/retention strategy), **query performance** (replaying all events expensive without snapshots, need projections for fast reads), **eventual consistency** (projections updated asynchronously after events, read model may lag write model), **complexity** (more code than CRUD, event store + projections + snapshots + versioning + aggregate root, steeper learning curve). **Use Event Sourcing when**: need complete audit trail (financial systems, healthcare, compliance), time travel required (undo/redo, historical reporting), multiple projections from same data (different dashboards/views), debugging critical (reproduce production bugs), with CQRS (commands generate events, queries use projections). **Avoid when**: simple CRUD sufficient, audit trail not critical, complexity not justified, team lacks experience, strong consistency required immediately.

</details>

## Scalability Patterns

<details>
<summary><strong>49. How do you design for horizontal scalability?</strong></summary>

**Answer:**

Design for **horizontal scalability** by ensuring your NestJS application can run **multiple identical instances** behind a load balancer without conflicts. Key principles: **stateless architecture** (no session state in memory, use Redis/database for sessions), **share-nothing design** (each instance independent, no shared file system, use S3/cloud storage for uploads), **distributed caching** (Redis cluster, not in-memory cache), **database connection pooling** (limit connections per instance, use read replicas for queries), **message queues** (Bull/RabbitMQ for async jobs, any instance can process), **sticky sessions avoided** (load balancer distributes randomly, not by user), **centralized logging** (Winston → Elasticsearch/CloudWatch, not local files), **health checks** (expose /health endpoint for load balancer), **graceful shutdown** (SIGTERM handler to finish requests before exit). Deploy multiple instances (Docker containers, Kubernetes pods, EC2 instances) and scale by adding/removing instances based on load (CPU, memory, queue depth).

---

### **Horizontal Scalability Architecture:**

```
                    ┌──────────────────┐
                    │  Load Balancer   │
                    │   (ALB, Nginx)   │
                    └────────┬─────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ▼                    ▼                    ▼
┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│  Instance 1   │    │  Instance 2   │    │  Instance 3   │
│   (NestJS)    │    │   (NestJS)    │    │   (NestJS)    │
└───────┬───────┘    └───────┬───────┘    └───────┬───────┘
        │                    │                    │
        └────────────────────┼────────────────────┘
                             │
                    ┌────────▼─────────┐
                    │  Shared Services │
                    ├──────────────────┤
                    │  Redis (Cache,   │
                    │   Sessions)      │
                    ├──────────────────┤
                    │  PostgreSQL      │
                    │   (Database)     │
                    ├──────────────────┤
                    │  RabbitMQ        │
                    │   (Queue)        │
                    ├──────────────────┤
                    │  S3 (Storage)    │
                    └──────────────────┘

Key Principles:
- Multiple identical instances
- Stateless (no local state)
- Shared external services
- Load balanced traffic
- Scale by adding instances
```

---

### **Method 1: Stateless Session Management**

```typescript
// ========== AVOID: IN-MEMORY SESSION (NOT SCALABLE) ==========

// ❌ BAD: Session stored in memory
const sessions = new Map<string, Session>();  // Lost when instance restarts

@Post('login')
async login(@Body() dto: LoginDto, @Res() res: Response) {
  const user = await this.authService.validateUser(dto);
  
  const sessionId = uuidv4();
  sessions.set(sessionId, { userId: user.id, expiresAt: Date.now() + 3600000 });
  
  res.cookie('sessionId', sessionId);
  return { success: true };
}

// PROBLEMS:
// - Session lost if instance restarts
// - Session not shared between instances
// - User logged out if load balancer switches instance

// ========== SOLUTION: REDIS SESSION STORE (SCALABLE) ==========

// ✅ GOOD: Session stored in Redis (shared across instances)

// Install dependencies
// npm install @nestjs/passport passport passport-jwt
// npm install @nestjs/jwt
// npm install ioredis @nestjs/cache-manager cache-manager-ioredis-yet

// Configure Redis session store
// src/app.module.ts
import { CacheModule } from '@nestjs/cache-manager';
import { redisStore } from 'cache-manager-ioredis-yet';

@Module({
  imports: [
    CacheModule.register({
      isGlobal: true,
      store: redisStore,
      host: process.env.REDIS_HOST,
      port: parseInt(process.env.REDIS_PORT),
      ttl: 3600,  // 1 hour
    }),
  ],
})
export class AppModule {}

// Use JWT tokens (stateless)
// src/features/auth/auth.service.ts
@Injectable()
export class AuthService {
  constructor(
    private readonly jwtService: JwtService,
    @Inject(CACHE_MANAGER) private readonly cacheManager: Cache,
  ) {}

  async login(dto: LoginDto): Promise<{ accessToken: string }> {
    const user = await this.validateUser(dto);

    // Generate JWT (stateless, works on any instance)
    const payload = { sub: user.id, email: user.email };
    const accessToken = this.jwtService.sign(payload);

    // Optional: Store refresh token in Redis
    const refreshToken = this.jwtService.sign(payload, { expiresIn: '7d' });
    await this.cacheManager.set(
      `refresh:${user.id}`,
      refreshToken,
      7 * 24 * 60 * 60,  // 7 days
    );

    return { accessToken };
  }

  async logout(userId: string): Promise<void> {
    // Remove refresh token from Redis
    await this.cacheManager.del(`refresh:${userId}`);
  }
}

// JWT Guard (stateless, no session lookup)
@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {}

@Controller('profile')
export class ProfileController {
  @UseGuards(JwtAuthGuard)  // Verifies JWT (stateless)
  @Get()
  async getProfile(@Req() req) {
    return req.user;  // Extracted from JWT
  }
}

// BENEFITS:
// - No session state in instance memory
// - JWT verifiable on any instance
// - Refresh tokens shared in Redis
// - Horizontally scalable
```

---

### **Method 2: Distributed Caching**

```typescript
// ========== AVOID: IN-MEMORY CACHE (NOT SCALABLE) ==========

// ❌ BAD: Cache stored in memory
const cache = new Map<string, any>();  // Not shared between instances

@Get(':id')
async getUser(@Param('id') id: string) {
  // Check local cache
  if (cache.has(id)) {
    return cache.get(id);
  }

  const user = await this.usersRepository.findById(id);
  cache.set(id, user);  // Cached only on THIS instance
  return user;
}

// PROBLEMS:
// - Cache not shared between instances
// - Cache miss when load balancer switches instance
// - Wasted memory (duplicate cache on each instance)

// ========== SOLUTION: REDIS DISTRIBUTED CACHE (SCALABLE) ==========

// ✅ GOOD: Cache stored in Redis (shared across instances)

@Injectable()
export class UsersService {
  constructor(
    private readonly usersRepository: UsersRepository,
    @Inject(CACHE_MANAGER) private readonly cacheManager: Cache,
  ) {}

  async findById(id: string): Promise<UserDto> {
    // Check Redis cache (shared across all instances)
    const cacheKey = `user:${id}`;
    const cached = await this.cacheManager.get<UserDto>(cacheKey);

    if (cached) {
      return cached;
    }

    // Query database
    const user = await this.usersRepository.findById(id);

    if (!user) {
      throw new NotFoundException('User not found');
    }

    const dto = this.toDto(user);

    // Store in Redis (available to all instances)
    await this.cacheManager.set(cacheKey, dto, 300);  // 5 minutes

    return dto;
  }

  async update(id: string, dto: UpdateUserDto): Promise<UserDto> {
    const user = await this.usersRepository.update(id, dto);

    // Invalidate cache (affects all instances)
    await this.cacheManager.del(`user:${id}`);

    return this.toDto(user);
  }
}

// BENEFITS:
// - Cache shared between all instances
// - Consistent cache hits across instances
// - Single source of truth for cached data
```

---

### **Method 3: Database Connection Pooling**

```typescript
// ========== DATABASE CONNECTION POOLING ==========

// Configure connection pool limits per instance
// src/config/database.config.ts
export const databaseConfig = (): TypeOrmModuleOptions => ({
  type: 'postgres',
  host: process.env.DB_HOST,
  port: parseInt(process.env.DB_PORT),
  username: process.env.DB_USERNAME,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  
  // Connection pool configuration
  extra: {
    max: 10,  // Max connections per instance (10 instances × 10 = 100 total)
    min: 2,   // Min connections per instance
    idleTimeoutMillis: 30000,  // Close idle connections after 30s
    connectionTimeoutMillis: 2000,  // Timeout after 2s
    
    // Connection pool settings
    acquireTimeout: 10000,  // Wait max 10s for connection
    evictionRunIntervalMillis: 10000,  // Check for idle connections every 10s
  },
  
  // Auto-load entities
  autoLoadEntities: true,
  synchronize: false,  // Use migrations in production
});

// CALCULATION:
// Database max connections: 200
// Expected instances: 10
// Connections per instance: 200 / 10 = 20 (set to 10 with headroom)

// ========== READ REPLICAS FOR SCALABILITY ==========

// Configure read replicas for queries
// src/config/database.config.ts
export const databaseConfig = (): TypeOrmModuleOptions => ({
  type: 'postgres',
  replication: {
    master: {
      host: process.env.DB_MASTER_HOST,
      port: parseInt(process.env.DB_MASTER_PORT),
      username: process.env.DB_USERNAME,
      password: process.env.DB_PASSWORD,
      database: process.env.DB_NAME,
    },
    slaves: [
      {
        host: process.env.DB_REPLICA_1_HOST,
        port: parseInt(process.env.DB_REPLICA_1_PORT),
        username: process.env.DB_USERNAME,
        password: process.env.DB_PASSWORD,
        database: process.env.DB_NAME,
      },
      {
        host: process.env.DB_REPLICA_2_HOST,
        port: parseInt(process.env.DB_REPLICA_2_PORT),
        username: process.env.DB_USERNAME,
        password: process.env.DB_PASSWORD,
        database: process.env.DB_NAME,
      },
    ],
  },
});

// TypeORM automatically routes:
// - SELECT queries → Read replicas (load balanced)
// - INSERT/UPDATE/DELETE → Master

@Injectable()
export class UsersRepository {
  async findAll(): Promise<User[]> {
    // Automatically routed to read replica
    return this.repository.find();
  }

  async create(data: CreateUserDto): Promise<User> {
    // Automatically routed to master
    return this.repository.save(data);
  }
}

// BENEFITS:
// - Scale reads independently from writes
// - Master handles writes (10% of traffic)
// - Replicas handle reads (90% of traffic)
// - Add more replicas as read traffic grows
```

---

### **Method 4: Message Queues for Async Jobs**

```typescript
// ========== BACKGROUND JOBS WITH BULL QUEUE (SCALABLE) ==========

// Install Bull
// npm install @nestjs/bull bull

// Configure Bull with Redis
// src/app.module.ts
import { BullModule } from '@nestjs/bull';

@Module({
  imports: [
    BullModule.forRoot({
      redis: {
        host: process.env.REDIS_HOST,
        port: parseInt(process.env.REDIS_PORT),
      },
    }),
    BullModule.registerQueue({
      name: 'emails',
    }),
    BullModule.registerQueue({
      name: 'notifications',
    }),
  ],
})
export class AppModule {}

// Producer: Add job to queue
@Injectable()
export class UsersService {
  constructor(
    @InjectQueue('emails') private readonly emailQueue: Queue,
  ) {}

  async register(dto: RegisterDto): Promise<User> {
    const user = await this.usersRepository.create(dto);

    // Add job to queue (any instance can process)
    await this.emailQueue.add('welcome', {
      userId: user.id,
      email: user.email,
      name: user.name,
    });

    return user;
  }
}

// Consumer: Process job (any instance)
@Processor('emails')
export class EmailProcessor {
  constructor(private readonly emailService: EmailService) {}

  @Process('welcome')
  async sendWelcomeEmail(job: Job) {
    const { userId, email, name } = job.data;
    
    await this.emailService.sendWelcomeEmail({
      to: email,
      name,
    });
  }
}

// BENEFITS:
// - Job added to centralized queue (Redis)
// - Any instance can process job (horizontally scalable)
// - Load distributed automatically
// - Retry failed jobs
// - Scale workers independently from API servers

// SCALING:
// - 5 API instances (handle HTTP requests)
// - 10 worker instances (process queue jobs)
// - Scale based on queue depth
```

---

### **Method 5: File Storage and Shared Resources**

```typescript
// ========== AVOID: LOCAL FILE STORAGE (NOT SCALABLE) ==========

// ❌ BAD: File stored on instance disk
@Post('upload')
@UseInterceptors(FileInterceptor('file'))
async uploadFile(@UploadedFile() file: Express.Multer.File) {
  // Save to local disk
  const path = `./uploads/${file.originalname}`;
  await fs.promises.writeFile(path, file.buffer);
  
  return { url: `/uploads/${file.originalname}` };
}

// PROBLEMS:
// - File only on instance that handled upload
// - Other instances can't access file
// - File lost if instance replaced

// ========== SOLUTION: CLOUD STORAGE (SCALABLE) ==========

// ✅ GOOD: File stored in S3 (accessible from all instances)

// Install AWS SDK
// npm install @aws-sdk/client-s3

// Configure S3
// src/services/storage.service.ts
import { S3Client, PutObjectCommand } from '@aws-sdk/client-s3';

@Injectable()
export class StorageService {
  private readonly s3Client = new S3Client({
    region: process.env.AWS_REGION,
    credentials: {
      accessKeyId: process.env.AWS_ACCESS_KEY_ID,
      secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
    },
  });

  async uploadFile(file: Express.Multer.File): Promise<string> {
    const key = `uploads/${Date.now()}-${file.originalname}`;

    await this.s3Client.send(
      new PutObjectCommand({
        Bucket: process.env.S3_BUCKET,
        Key: key,
        Body: file.buffer,
        ContentType: file.mimetype,
      }),
    );

    return `https://${process.env.S3_BUCKET}.s3.amazonaws.com/${key}`;
  }
}

@Controller('files')
export class FilesController {
  constructor(private readonly storageService: StorageService) {}

  @Post('upload')
  @UseInterceptors(FileInterceptor('file'))
  async uploadFile(@UploadedFile() file: Express.Multer.File) {
    const url = await this.storageService.uploadFile(file);
    return { url };
  }
}

// BENEFITS:
// - File accessible from all instances
// - Survives instance replacement
// - Scales independently
// - CDN integration
```

---

### **Method 6: Health Checks and Graceful Shutdown**

```typescript
// ========== HEALTH CHECKS FOR LOAD BALANCER ==========

// Install terminus
// npm install @nestjs/terminus

// src/health/health.controller.ts
import { Controller, Get } from '@nestjs/common';
import { HealthCheck, HealthCheckService, TypeOrmHealthIndicator } from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(
    private readonly health: HealthCheckService,
    private readonly db: TypeOrmHealthIndicator,
  ) {}

  @Get()
  @HealthCheck()
  check() {
    return this.health.check([
      () => this.db.pingCheck('database'),
      // Add more checks: Redis, message queue, etc.
    ]);
  }
}

// Load balancer configuration (ALB, Nginx)
// Health check path: /health
// Interval: 10 seconds
// Unhealthy threshold: 2 consecutive failures
// Healthy threshold: 2 consecutive successes

// ========== GRACEFUL SHUTDOWN ==========

// src/main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Enable graceful shutdown
  app.enableShutdownHooks();

  // Handle SIGTERM (sent by orchestrator when scaling down)
  process.on('SIGTERM', async () => {
    console.log('SIGTERM received, shutting down gracefully');
    
    // Stop accepting new requests
    await app.close();
    
    // Kubernetes waits for terminationGracePeriodSeconds (default 30s)
    // Finish existing requests during this time
  });

  await app.listen(3000);
}

bootstrap();

// Kubernetes deployment with graceful shutdown
// deployment.yaml
// apiVersion: apps/v1
// kind: Deployment
// spec:
//   template:
//     spec:
//       containers:
//       - name: nestjs-app
//         lifecycle:
//           preStop:
//             exec:
//               command: ["/bin/sh", "-c", "sleep 10"]  # Wait for requests to finish
//       terminationGracePeriodSeconds: 30  # Max wait time

// FLOW:
// 1. Scale down triggered (kubectl scale, autoscaler)
// 2. Load balancer stops sending new requests to instance
// 3. SIGTERM sent to instance
// 4. Instance finishes existing requests (up to 30s)
// 5. Instance exits
```

**Key Takeaway:** Design for **horizontal scalability** by ensuring NestJS application can run **multiple identical instances** behind load balancer without conflicts. **Core principles**: **stateless architecture** (no session state in memory, use JWT tokens verified on any instance, store sessions in Redis if needed, no file system dependencies), **share-nothing design** (each instance independent, no shared local disk, use S3/cloud storage for file uploads, no in-memory cache), **distributed caching** (Redis cluster for shared cache across instances, invalidate cache consistently, avoid local Map/memory cache), **database connection pooling** (limit connections per instance: 200 max connections ÷ 10 instances = 20 per instance, use read replicas for queries - master handles writes/replicas handle reads 90% of traffic, scale reads independently), **message queues** (Bull with Redis for async jobs - any instance can process jobs, scale workers independently from API servers, distribute load automatically), **centralized services** (Redis for cache/sessions, PostgreSQL with replicas for data, RabbitMQ/Bull for queues, S3 for file storage), **health checks** (expose /health endpoint checking database/Redis/queue connectivity, load balancer routes traffic only to healthy instances, unhealthy instances removed automatically), **graceful shutdown** (handle SIGTERM signal, stop accepting new requests, finish existing requests within grace period 30s, Kubernetes waits before force kill). **Avoid**: in-memory sessions (lost on restart, not shared), local file storage (other instances can't access), in-memory cache (duplicate cache per instance, inconsistent), sticky sessions (load balancer locks user to instance, can't scale down), hardcoded instance-specific config. **Scale by**: adding/removing instances based on metrics (CPU >70% add instance, <30% remove instance), autoscaling groups (Kubernetes HPA, AWS Auto Scaling), scale API servers and workers independently (5 API instances + 10 worker instances). **Deploy**: Docker containers, Kubernetes pods with HPA, EC2 instances with ALB, Fargate tasks, Cloud Run instances.

</details>

<details>
<summary><strong>50. What is stateless architecture and why is it important?</strong></summary>

**Answer:**

**Stateless architecture** means each request contains **all information needed** to process it (JWT token, request headers, query parameters), and the server **doesn't store session state** between requests. Every request is **independent** - server doesn't remember previous requests, no session variables, no user context in memory. Benefits: **horizontal scalability** (any instance can handle any request, add instances without coordination), **fault tolerance** (if instance crashes, user not affected - next request goes to different instance), **load balancing** (distribute requests randomly, no sticky sessions needed), **simpler deployment** (no session migration, restart instances anytime, zero-downtime deployments). Implement by: **using JWT** for authentication (token contains user ID/role, verified on each request, no session lookup), **storing session data externally** (Redis for cart/preferences, database for user data, no in-memory state), **making requests self-contained** (pagination cursor in request, not in session; filters in query params, not in session), **avoiding global state** (no singleton with mutable state, no static variables with user data). Contrast with stateful: session stored in server memory, requires sticky sessions, hard to scale.

---

### **Stateless vs Stateful Architecture:**

```
STATEFUL ARCHITECTURE:
┌──────────────────────────────────────────────────┐
│  User makes Request 1                            │
│  → Server stores session in memory               │
│  → Server remembers user context                 │
└──────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────┐
│  User makes Request 2                            │
│  → MUST go to same server (sticky session)       │
│  → Server retrieves session from memory          │
│  → Uses previous context                         │
└──────────────────────────────────────────────────┘
Problems: Hard to scale, single point of failure

STATELESS ARCHITECTURE:
┌──────────────────────────────────────────────────┐
│  User makes Request 1 (with JWT token)           │
│  → Can go to any server instance                 │
│  → Server extracts user from JWT                 │
│  → Processes request independently               │
└──────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────┐
│  User makes Request 2 (with JWT token)           │
│  → Can go to different server instance           │
│  → Server extracts user from JWT                 │
│  → No previous context needed                    │
└──────────────────────────────────────────────────┘
Benefits: Easy to scale, fault tolerant, load balanced
```

---

### **Method 1: Stateless Authentication with JWT**

```typescript
// ========== STATEFUL AUTHENTICATION (NOT SCALABLE) ==========

// ❌ BAD: Session stored in server memory
const sessions = new Map<string, { userId: string; expiresAt: number }>();

@Post('login')
async login(@Body() dto: LoginDto, @Res() res: Response) {
  const user = await this.authService.validateUser(dto);
  
  // Create session in memory
  const sessionId = uuidv4();
  sessions.set(sessionId, {
    userId: user.id,
    expiresAt: Date.now() + 3600000,  // 1 hour
  });
  
  // Set cookie
  res.cookie('sessionId', sessionId, { httpOnly: true });
  return { success: true };
}

@Get('profile')
async getProfile(@Req() req: Request) {
  const sessionId = req.cookies.sessionId;
  
  // Look up session in memory
  const session = sessions.get(sessionId);
  
  if (!session || session.expiresAt < Date.now()) {
    throw new UnauthorizedException('Invalid session');
  }
  
  // Get user from database
  const user = await this.usersRepository.findById(session.userId);
  return user;
}

// PROBLEMS:
// - Session lost if instance restarts
// - Session not shared between instances (must use sticky sessions)
// - Hard to scale horizontally
// - Single point of failure

// ========== STATELESS AUTHENTICATION (SCALABLE) ==========

// ✅ GOOD: JWT tokens (no session storage)

// Install JWT
// npm install @nestjs/jwt @nestjs/passport passport passport-jwt

// Configure JWT
// src/features/auth/auth.module.ts
@Module({
  imports: [
    JwtModule.register({
      secret: process.env.JWT_SECRET,
      signOptions: { expiresIn: '1h' },
    }),
  ],
  providers: [AuthService, JwtStrategy],
})
export class AuthModule {}

// Login: Generate JWT token
@Injectable()
export class AuthService {
  constructor(private readonly jwtService: JwtService) {}

  async login(dto: LoginDto): Promise<{ accessToken: string }> {
    const user = await this.validateUser(dto);

    // Create JWT payload (contains all info needed)
    const payload = {
      sub: user.id,
      email: user.email,
      role: user.role,
    };

    // Sign JWT (stateless token)
    const accessToken = this.jwtService.sign(payload);

    return { accessToken };
  }
}

// JWT Strategy: Verify token on each request
@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      secretOrKey: process.env.JWT_SECRET,
    });
  }

  async validate(payload: any) {
    // Token verified, extract user info from payload
    return {
      userId: payload.sub,
      email: payload.email,
      role: payload.role,
    };
  }
}

// Use JWT guard: No session lookup needed
@Controller('profile')
export class ProfileController {
  @UseGuards(JwtAuthGuard)
  @Get()
  async getProfile(@Req() req) {
    // User info extracted from JWT (no database lookup)
    return req.user;  // { userId: '123', email: '...', role: 'user' }
  }
}

// BENEFITS:
// - No session storage (stateless)
// - Token verified on any instance
// - Horizontally scalable
// - No sticky sessions needed
// - Instance restart doesn't affect users
```

---

### **Method 2: Self-Contained Requests**

```typescript
// ========== STATEFUL: SESSION-BASED PAGINATION (NOT SCALABLE) ==========

// ❌ BAD: Pagination state in session
const paginationState = new Map<string, { page: number; filters: any }>();

@Get('products')
async getProducts(@Req() req: Request) {
  const sessionId = req.cookies.sessionId;
  
  // Get pagination state from session
  const state = paginationState.get(sessionId) || { page: 1, filters: {} };
  
  // Query with session state
  const products = await this.productsRepository.find({
    skip: (state.page - 1) * 20,
    take: 20,
    where: state.filters,
  });
  
  return products;
}

@Post('products/next-page')
async nextPage(@Req() req: Request) {
  const sessionId = req.cookies.sessionId;
  
  // Update session state
  const state = paginationState.get(sessionId);
  state.page += 1;
  paginationState.set(sessionId, state);
  
  return { message: 'Page updated' };
}

// PROBLEMS:
// - Pagination state in server memory
// - Lost on instance restart
// - Not shared between instances

// ========== STATELESS: SELF-CONTAINED PAGINATION (SCALABLE) ==========

// ✅ GOOD: All pagination info in request
@Get('products')
async getProducts(
  @Query('page', new DefaultValuePipe(1), ParseIntPipe) page: number,
  @Query('limit', new DefaultValuePipe(20), ParseIntPipe) limit: number,
  @Query('category') category?: string,
  @Query('minPrice', new DefaultValuePipe(0), ParseIntPipe) minPrice?: number,
  @Query('maxPrice') maxPrice?: number,
) {
  // All info in request - no session needed
  const products = await this.productsRepository.find({
    skip: (page - 1) * limit,
    take: limit,
    where: {
      ...(category && { category }),
      ...(minPrice && { price: MoreThanOrEqual(minPrice) }),
      ...(maxPrice && { price: LessThanOrEqual(maxPrice) }),
    },
  });

  return {
    data: products,
    meta: {
      page,
      limit,
      // Client knows current page from query params
      // Next request: GET /products?page=2&limit=20&category=electronics
    },
  };
}

// BENEFITS:
// - No server state
// - Request contains all info
// - Can be processed by any instance
// - Bookmarkable URLs
// - No session dependency
```

---

### **Method 3: External State Storage**

```typescript
// ========== STATEFUL: SHOPPING CART IN MEMORY (NOT SCALABLE) ==========

// ❌ BAD: Cart stored in server memory
const carts = new Map<string, CartItem[]>();

@Post('cart/add')
async addToCart(
  @Req() req: Request,
  @Body() item: CartItem,
) {
  const sessionId = req.cookies.sessionId;
  
  // Get cart from memory
  const cart = carts.get(sessionId) || [];
  cart.push(item);
  
  // Store cart in memory
  carts.set(sessionId, cart);
  
  return { cart };
}

// PROBLEMS:
// - Cart lost on instance restart
// - Not shared between instances
// - Memory leak if not cleaned up

// ========== STATELESS: CART IN REDIS (SCALABLE) ==========

// ✅ GOOD: Cart stored in Redis (external state)
@Injectable()
export class CartService {
  constructor(@Inject(CACHE_MANAGER) private readonly cache: Cache) {}

  async addItem(userId: string, item: CartItem): Promise<CartItem[]> {
    // Get cart from Redis (shared across instances)
    const cacheKey = `cart:${userId}`;
    const cart = await this.cache.get<CartItem[]>(cacheKey) || [];
    
    // Add item
    cart.push(item);
    
    // Store in Redis (available to all instances)
    await this.cache.set(cacheKey, cart, 86400);  // 24 hours
    
    return cart;
  }

  async getCart(userId: string): Promise<CartItem[]> {
    const cacheKey = `cart:${userId}`;
    return await this.cache.get<CartItem[]>(cacheKey) || [];
  }

  async clearCart(userId: string): Promise<void> {
    await this.cache.del(`cart:${userId}`);
  }
}

@Controller('cart')
export class CartController {
  constructor(private readonly cartService: CartService) {}

  @UseGuards(JwtAuthGuard)
  @Post('add')
  async addItem(@Req() req, @Body() item: CartItem) {
    const userId = req.user.userId;
    const cart = await this.cartService.addItem(userId, item);
    return { cart };
  }

  @UseGuards(JwtAuthGuard)
  @Get()
  async getCart(@Req() req) {
    const userId = req.user.userId;
    return this.cartService.getCart(userId);
  }
}

// BENEFITS:
// - Cart in external storage (Redis)
// - Shared across all instances
// - Survives instance restart
// - Stateless from instance perspective
```

---

### **Method 4: Benefits of Stateless Architecture**

```typescript
// ========== HORIZONTAL SCALABILITY ==========

// Stateless: Scale freely
// Any instance can handle any request
// Load balancer distributes randomly
// Add/remove instances without coordination

// Autoscaling configuration (Kubernetes)
// apiVersion: autoscaling/v2
// kind: HorizontalPodAutoscaler
// spec:
//   scaleTargetRef:
//     apiVersion: apps/v1
//     kind: Deployment
//     name: nestjs-app
//   minReplicas: 3
//   maxReplicas: 10
//   metrics:
//   - type: Resource
//     resource:
//       name: cpu
//       target:
//         type: Utilization
//         averageUtilization: 70

// With stateless architecture:
// - Traffic spike → autoscaler adds instances
// - Traffic drops → autoscaler removes instances
// - No session migration needed
// - Zero-downtime scaling

// ========== FAULT TOLERANCE ==========

// Stateless: Instance crash doesn't affect users
@Injectable()
export class OrdersService {
  @UseGuards(JwtAuthGuard)
  @Post()
  async createOrder(@Req() req, @Body() dto: CreateOrderDto) {
    // Extract user from JWT (no session lookup)
    const userId = req.user.userId;
    
    // Create order
    const order = await this.ordersRepository.create({
      ...dto,
      userId,
    });
    
    return order;
  }
}

// SCENARIO: Instance crashes during request
// 1. User retries request
// 2. Load balancer routes to different instance
// 3. JWT verified on new instance
// 4. Order created successfully
// 5. User not affected by crash

// ========== LOAD BALANCING ==========

// Stateless: Round-robin load balancing
// No sticky sessions needed

// Nginx configuration
// upstream nestjs_backend {
//     server 10.0.1.10:3000;
//     server 10.0.1.11:3000;
//     server 10.0.1.12:3000;
//     # Round-robin by default (no ip_hash needed)
// }

// ========== ZERO-DOWNTIME DEPLOYMENTS ==========

// Stateless: Rolling updates without session loss
// 1. Deploy new version to instance 1
// 2. Health check passes
// 3. Load balancer routes traffic to instance 1
// 4. Deploy new version to instance 2
// 5. Repeat for all instances
// 6. No users logged out or lose data
```

---

### **Method 5: When Stateful is Necessary**

```typescript
// ========== STATEFUL USE CASES ==========

// WebSockets: Persistent connections (inherently stateful)
@WebSocketGateway()
export class ChatGateway {
  private connectedUsers = new Map<string, Socket>();

  @SubscribeMessage('join')
  handleJoin(client: Socket, userId: string) {
    // Store socket connection (stateful)
    this.connectedUsers.set(userId, client);
  }

  @SubscribeMessage('message')
  handleMessage(client: Socket, message: string) {
    // Broadcast to connected users
    this.connectedUsers.forEach((socket) => {
      socket.emit('message', message);
    });
  }
}

// SOLUTION FOR SCALING WEBSOCKETS:
// - Use sticky sessions for WebSocket connections
// - Or use Redis pub/sub for message broadcasting
// - Or use dedicated WebSocket servers

// Long-running processes: Streaming, file processing
@Controller('reports')
export class ReportsController {
  @Get('generate')
  async generateReport(@Res() res: Response) {
    // Start streaming (stateful connection)
    res.setHeader('Content-Type', 'text/csv');
    
    const stream = await this.reportsService.generateCSV();
    stream.pipe(res);
  }
}

// SOLUTION:
// - Use sticky sessions for duration of stream
// - Or use S3 pre-signed URLs (stateless)
// - Or use background jobs + polling (stateless)
```

**Key Takeaway:** **Stateless architecture** means each request contains **all information needed to process it** (JWT token with userId/email/role, request headers, query parameters with page/filters/sort), and server **doesn't store session state** between requests (no session variables in memory, no user context stored, no state about previous requests). Every request is **independent** - server processes request using only data in request + external storage (Redis/database), any instance can handle any request, server doesn't need to remember user. **Benefits**: **horizontal scalability** (add instances freely without coordination, any instance handles any request, load balancer distributes randomly without sticky sessions, autoscaling based on CPU/memory), **fault tolerance** (instance crash doesn't affect users - next request goes to healthy instance, no session lost, retry works seamlessly), **load balancing** (round-robin distribution, no sticky sessions needed, better resource utilization), **simpler deployment** (rolling updates without session migration, restart instances anytime without logging out users, zero-downtime deployments, no session replication). **Implementation**: **JWT authentication** (token contains userId/email/role, verified on each request without database lookup, no session storage needed, token self-contained), **external state storage** (Redis for cart/preferences - shared across instances, database for user data, no in-memory state), **self-contained requests** (pagination in query params ?page=2&limit=20, filters in request not session, cursor-based pagination with token in response), **avoid global mutable state** (no singleton with user data, no static variables with session, no in-memory maps with user state). **Comparison to stateful**: stateful stores session in server memory (lost on restart, requires sticky sessions - user locked to instance, hard to scale - can't add instances freely, single point of failure - crash logs out users), stateless stores state externally (survives restart, no sticky sessions - any instance works, easy to scale - add instances freely, fault tolerant - crash doesn't affect users). **When stateful necessary**: WebSocket connections (persistent connection inherently stateful, use sticky sessions or Redis pub/sub for broadcasting), streaming responses (stateful connection during stream, use sticky sessions or S3 pre-signed URLs), long-running processes (use background jobs + polling for stateless alternative).

</details>


