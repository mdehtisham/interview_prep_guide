# NestJS Core Concepts - Top Interview Questions

## Architecture & Fundamentals

### 1. What is NestJS and why should you use it?

<details>
<summary>Answer</summary>

**NestJS** is a progressive Node.js framework for building efficient, scalable, and maintainable server-side applications. It's built with TypeScript (also supports JavaScript) and combines elements of OOP (Object-Oriented Programming), FP (Functional Programming), and FRP (Functional Reactive Programming).

**Why use NestJS:**

1. **Architecture & Structure**: Provides opinionated, well-structured architecture out of the box (unlike Express which is minimal)
2. **TypeScript First**: Full TypeScript support with decorators, type safety, and interfaces
3. **Modularity**: Built-in modular architecture makes code organization and scalability easier
4. **Dependency Injection**: Powerful DI container for loose coupling and testability
5. **Enterprise-Ready**: Follows SOLID principles, design patterns, and best practices
6. **Ecosystem**: Rich ecosystem with official packages for GraphQL, WebSockets, Microservices, CQRS, etc.
7. **Testing**: First-class testing support with utilities for unit and e2e tests
8. **Platform Agnostic**: Can use Express or Fastify under the hood
9. **Decorator-Based**: Clean, declarative code using decorators (@Controller, @Get, etc.)
10. **Production-Ready**: Used by major companies; stable, well-documented, and actively maintained

**Example:**
```typescript
// Clean, declarative code with decorators
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {} // DI

  @Get()
  @UseGuards(AuthGuard) // Built-in features
  async findAll(): Promise<User[]> {
    return this.usersService.findAll();
  }
}
```

**Best for**: Enterprise applications, microservices, APIs requiring structure, maintainability, and scalability.

</details>

### 2. What problem does NestJS solve in the Node.js ecosystem?

<details>
<summary>Answer</summary>

NestJS solves several critical problems in the Node.js ecosystem:

**1. Lack of Structure & Architecture**
- **Problem**: Express/Koa are minimal and unopinionated; developers must architect everything themselves
- **Solution**: NestJS provides a well-defined, modular architecture out of the box

**2. Scalability Challenges**
- **Problem**: As applications grow, maintaining code organization becomes difficult
- **Solution**: Module-based architecture keeps code organized and maintainable at scale

**3. No Built-in Dependency Injection**
- **Problem**: Manually managing dependencies leads to tight coupling and hard-to-test code
- **Solution**: Powerful DI container for loose coupling and easy testing

**4. TypeScript Integration Issues**
- **Problem**: Setting up TypeScript with decorators in Express requires significant configuration
- **Solution**: TypeScript-first with decorators working seamlessly

**5. Inconsistent Code Patterns**
- **Problem**: Different developers use different patterns, making collaboration difficult
- **Solution**: Enforces consistent patterns and conventions across the codebase

**6. Testing Complexity**
- **Problem**: Testing Node.js apps often requires significant boilerplate and mocking setup
- **Solution**: Built-in testing utilities and DI make unit/integration testing straightforward

**7. Missing Enterprise Features**
- **Problem**: Need to integrate multiple libraries for auth, validation, logging, etc.
- **Solution**: Official packages for common needs (Guards, Pipes, Interceptors, Exception Filters)

**8. Boilerplate Code**
- **Problem**: Lots of repetitive code for common tasks (validation, serialization, error handling)
- **Solution**: Decorators and built-in features reduce boilerplate significantly

**Real-World Impact:**
```typescript
// Without NestJS (Express) - Manual setup, no structure
const express = require('express');
const app = express();
// Manual routing, validation, error handling, DI...

// With NestJS - Structured, declarative, maintainable
@Module({
  imports: [TypeOrmModule, ConfigModule],
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}
```

**Bottom Line**: NestJS brings the structure, patterns, and tooling from frameworks like Angular/Spring to Node.js, making it suitable for large-scale, production applications.

</details>
3. Explain the main architectural pattern NestJS follows (Modules, Controllers, Providers).
4. What is the difference between NestJS and Express?
5. Why is NestJS built on top of Express/Fastify?

### 3. Explain the main architectural pattern NestJS follows (Modules, Controllers, Providers)

<details>
<summary>Answer</summary>

NestJS follows a **modular, layered architecture** inspired by Angular, consisting of three core building blocks:

**1. Modules (`@Module`)**
- **Purpose**: Organization unit that groups related components
- **Responsibility**: Encapsulate features and manage dependencies
- **Structure**: Contains controllers, providers, imports, and exports

```typescript
@Module({
  imports: [TypeOrmModule.forFeature([User])],    // Dependencies from other modules
  controllers: [UsersController],                   // HTTP layer
  providers: [UsersService, UsersRepository],       // Business logic & data access
  exports: [UsersService],                          // Make service available to other modules
})
export class UsersModule {}
```

**2. Controllers (`@Controller`)**
- **Purpose**: Handle HTTP requests and responses (Presentation Layer)
- **Responsibility**: Route handling, request/response transformation, delegation to services
- **Should NOT**: Contain business logic or database access

```typescript
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {} // DI

  @Get(':id')
  @UseGuards(AuthGuard) // Apply guards
  async findOne(@Param('id') id: string): Promise<User> {
    return this.usersService.findOne(id); // Delegate to service
  }

  @Post()
  @UsePipes(ValidationPipe) // Apply validation
  async create(@Body() createUserDto: CreateUserDto): Promise<User> {
    return this.usersService.create(createUserDto);
  }
}
```

**3. Providers (`@Injectable`)**
- **Purpose**: Business logic, data access, external API calls (Business Logic Layer)
- **Includes**: Services, Repositories, Factories, Helpers, Guards, Interceptors, etc.
- **Responsibility**: Core application logic, reusable across controllers

```typescript
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}

  async findOne(id: string): Promise<User> {
    const user = await this.userRepository.findOne({ where: { id } });
    if (!user) {
      throw new NotFoundException(`User #${id} not found`);
    }
    return user;
  }

  async create(createUserDto: CreateUserDto): Promise<User> {
    const user = this.userRepository.create(createUserDto);
    return this.userRepository.save(user);
  }
}
```

**Architecture Flow:**
```
Client Request
    ↓
Controller (HTTP layer - routing, validation)
    ↓
Service (Business logic)
    ↓
Repository (Data access)
    ↓
Database
```

**Key Principles:**
1. **Separation of Concerns**: Each layer has a distinct responsibility
2. **Dependency Injection**: Controllers depend on Services, Services depend on Repositories
3. **Modularity**: Features are encapsulated in modules
4. **Testability**: Each layer can be tested independently with mocks

**Production Best Practice:**
```typescript
// ❌ Bad: Business logic in controller
@Controller('users')
export class UsersController {
  @Post()
  async create(@Body() dto: CreateUserDto) {
    // Direct database access - WRONG!
    const user = await this.userRepository.save(dto);
    return user;
  }
}

// ✅ Good: Layered architecture
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}
  
  @Post()
  async create(@Body() dto: CreateUserDto) {
    return this.usersService.create(dto); // Delegate to service
  }
}

@Injectable()
export class UsersService {
  constructor(private readonly userRepository: UsersRepository) {}
  
  async create(dto: CreateUserDto): Promise<User> {
    // Business logic, validation, error handling here
    return this.userRepository.save(dto);
  }
}
```

</details>

### 4. What is the difference between NestJS and Express?

<details>
<summary>Answer</summary>

**Key Differences:**

| Aspect | Express | NestJS |
|--------|---------|--------|
| **Architecture** | Minimal, unopinionated | Opinionated, modular architecture |
| **TypeScript** | Requires manual setup | Built-in, first-class support |
| **Structure** | No structure enforced | Module-based structure required |
| **Dependency Injection** | Manual (DIY) | Built-in DI container |
| **Decorators** | Not supported natively | Core feature (@Controller, @Get, etc.) |
| **Learning Curve** | Easier to start | Steeper, but scales better |
| **Boilerplate** | Less initial code | More initial setup, less long-term |
| **Testing** | Manual setup | Built-in testing utilities |
| **Scalability** | Requires discipline | Built for scale |

**1. Architecture & Organization**

```typescript
// Express - Minimal, you decide everything
const express = require('express');
const app = express();

app.get('/users', (req, res) => {
  // Everything manually: validation, error handling, business logic
  res.json({ users: [] });
});

app.listen(3000);
```

```typescript
// NestJS - Structured, opinionated
@Module({
  imports: [TypeOrmModule],
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}

@Controller('users')
export class UsersController {
  constructor(private usersService: UsersService) {} // DI
  
  @Get()
  findAll(): Promise<User[]> {
    return this.usersService.findAll();
  }
}
```

**2. Dependency Injection**

```typescript
// Express - Manual dependency management
const usersService = new UsersService();
const usersController = new UsersController(usersService);

// NestJS - Automatic DI
@Injectable()
export class UsersService {}

@Controller('users')
export class UsersController {
  constructor(private usersService: UsersService) {} // Auto-injected
}
```

**3. Built-in Features**

```typescript
// Express - Install & configure everything manually
const express = require('express');
const helmet = require('helmet');
const cors = require('cors');
const rateLimit = require('express-rate-limit');

app.use(helmet());
app.use(cors());
app.use(rateLimit({...}));

// NestJS - Built-in or official packages
@Module({
  imports: [
    ThrottlerModule.forRoot({...}), // Rate limiting
    ConfigModule.forRoot(),          // Configuration
  ],
})
export class AppModule {}

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.enableCors();                  // CORS
  app.use(helmet());                 // Security
  await app.listen(3000);
}
```

**4. Error Handling**

```typescript
// Express - Manual error middleware
app.use((err, req, res, next) => {
  res.status(500).json({ message: err.message });
});

// NestJS - Exception Filters (automatic)
@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    // Centralized error handling
  }
}
```

**5. Validation**

```typescript
// Express - Manual validation
app.post('/users', (req, res) => {
  if (!req.body.email) {
    return res.status(400).json({ error: 'Email required' });
  }
  // More validation...
});

// NestJS - Declarative validation with Pipes
export class CreateUserDto {
  @IsEmail()
  email: string;

  @MinLength(8)
  password: string;
}

@Post()
create(@Body() createUserDto: CreateUserDto) {
  // Automatically validated
}
```

**When to Use Each:**

**Use Express when:**
- Building small, simple APIs
- Need maximum flexibility
- Team prefers minimal frameworks
- Prototyping quickly

**Use NestJS when:**
- Building enterprise applications
- Need structure and maintainability
- Working in large teams
- Require TypeScript and DI
- Building microservices or complex systems

**Production Reality:**
- Express: Great for small projects, but requires discipline for large apps
- NestJS: Higher initial investment, but pays off in maintainability and scalability

**Note**: NestJS actually uses Express (or Fastify) under the hood, so you get Express performance with NestJS structure!

</details>

### 5. Why is NestJS built on top of Express/Fastify?

<details>
<summary>Answer</summary>

NestJS is built on top of Express (default) or Fastify by design, leveraging proven HTTP server platforms rather than building from scratch. Here's why:

**1. Leverage Battle-Tested Technology**
- Express/Fastify are mature, production-proven HTTP servers
- No need to reinvent the wheel for low-level HTTP handling
- Inherits years of bug fixes, security patches, and optimization

**2. Performance & Ecosystem**
- Express: Largest ecosystem, most middleware available
- Fastify: ~65% faster than Express, schema-based validation
- Both handle millions of requests in production environments

**3. Middleware Compatibility**
- Access to vast Express/Fastify middleware ecosystem
- Use existing middleware: helmet, cors, compression, passport, etc.

```typescript
// Use any Express middleware
import * as helmet from 'helmet';
import * as compression from 'compression';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.use(helmet());      // Express middleware works
  app.use(compression()); // Express middleware works
  await app.listen(3000);
}
```

**4. Platform Agnostic Design**
- NestJS abstracts HTTP platform details
- Can switch between Express and Fastify with minimal changes
- Your business logic stays independent of the HTTP server

```typescript
// Using Express (default)
const app = await NestFactory.create(AppModule);

// Switch to Fastify - just change adapter
import { FastifyAdapter } from '@nestjs/platform-fastify';
const app = await NestFactory.create(
  AppModule,
  new FastifyAdapter()
);
```

**5. Separation of Concerns**
- NestJS focuses on architecture, DI, modularity
- Express/Fastify handle HTTP specifics
- Clean separation of responsibilities

**6. Developer Familiarity**
- Most Node.js developers know Express
- Lower barrier to adoption
- Easy migration from Express projects

**Production Benefits:**

```typescript
// ✅ Get best of both worlds
@Module({
  // NestJS features: DI, Guards, Interceptors
  imports: [TypeOrmModule, ConfigModule],
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}

// + Express/Fastify: HTTP handling, middleware, routing
```

**Choosing Between Express and Fastify:**

**Use Express when:**
- Need maximum middleware compatibility
- Team familiar with Express
- Default choice for most apps

**Use Fastify when:**
- Need maximum performance (2x faster)
- Built-in JSON schema validation
- Handling high-throughput APIs

```typescript
// Fastify example with performance benefits
import { NestFactory } from '@nestjs/core';
import { FastifyAdapter, NestFastifyApplication } from '@nestjs/platform-fastify';

async function bootstrap() {
  const app = await NestFactory.create<NestFastifyApplication>(
    AppModule,
    new FastifyAdapter({
      logger: true,
      // Fastify-specific options
    })
  );
  await app.listen(3000, '0.0.0.0');
}
```

**Key Takeaway**: NestJS adds structure and features on top of proven HTTP servers, giving you architectural benefits without sacrificing performance or ecosystem compatibility.

</details>

## Modules

### 6. What is a Module in NestJS?

<details>
<summary>Answer</summary>

A **Module** is a class annotated with `@Module()` decorator that organizes application structure by grouping related components (controllers, providers, imports). It's the fundamental building block for organizing code in NestJS.

**Purpose:**
1. **Organization**: Group related features together
2. **Encapsulation**: Keep internal implementation details private
3. **Dependency Management**: Control what's shared between modules
4. **Scalability**: Break large apps into manageable pieces

**Module Anatomy:**

```typescript
@Module({
  imports: [],      // Modules whose providers you need
  controllers: [],  // Controllers owned by this module
  providers: [],    // Services/providers available in this module
  exports: [],      // Providers to make available to other modules
})
export class MyModule {}
```

**Real-World Example:**

```typescript
// users.module.ts
@Module({
  imports: [
    TypeOrmModule.forFeature([User]),        // Database entities
    ConfigModule,                             // Configuration
  ],
  controllers: [UsersController],             // HTTP endpoints
  providers: [
    UsersService,                             // Business logic
    UsersRepository,                          // Data access
    PasswordHashingService,                   // Utility
  ],
  exports: [UsersService],                    // Other modules can use this
})
export class UsersModule {}
```

**Key Properties:**

**1. `imports`**: Modules whose exported providers you need

```typescript
@Module({
  imports: [
    DatabaseModule,    // Need database connection
    AuthModule,        // Need auth services
  ],
})
export class UsersModule {}
```

**2. `controllers`**: Route handlers for this module

```typescript
@Module({
  controllers: [
    UsersController,   // Handles /users routes
    ProfileController, // Handles /profile routes
  ],
})
export class UsersModule {}
```

**3. `providers`**: Services, repositories, factories available in this module

```typescript
@Module({
  providers: [
    UsersService,           // Available for injection
    UsersRepository,
    {
      provide: 'CONFIG',    // Custom provider
      useValue: { apiKey: 'xxx' },
    },
  ],
})
export class UsersModule {}
```

**4. `exports`**: Providers to share with other modules

```typescript
@Module({
  providers: [UsersService, InternalHelper],
  exports: [UsersService],  // Only UsersService available outside
                            // InternalHelper stays private
})
export class UsersModule {}
```

**Module Types:**

**1. Feature Module** (most common)
```typescript
// Organizes a specific feature
@Module({
  imports: [TypeOrmModule.forFeature([Product])],
  controllers: [ProductsController],
  providers: [ProductsService],
  exports: [ProductsService],
})
export class ProductsModule {}
```

**2. Shared Module** (utilities, common services)
```typescript
@Module({
  providers: [LoggerService, ConfigService],
  exports: [LoggerService, ConfigService], // Shared across app
})
export class SharedModule {}
```

**3. Global Module** (available everywhere)
```typescript
@Global() // Don't need to import in other modules
@Module({
  providers: [DatabaseService],
  exports: [DatabaseService],
})
export class DatabaseModule {}
```

**4. Dynamic Module** (configurable modules)
```typescript
@Module({})
export class ConfigModule {
  static forRoot(options: ConfigOptions): DynamicModule {
    return {
      module: ConfigModule,
      providers: [
        {
          provide: 'CONFIG_OPTIONS',
          useValue: options,
        },
      ],
      exports: ['CONFIG_OPTIONS'],
    };
  }
}
```

**Module Composition Example:**

```typescript
// Root module imports feature modules
@Module({
  imports: [
    ConfigModule.forRoot(),   // Global config
    DatabaseModule,           // Database connection
    UsersModule,              // Users feature
    ProductsModule,           // Products feature
    OrdersModule,             // Orders feature
  ],
})
export class AppModule {}
```

**Best Practices:**

1. **One Feature, One Module**: Each module should represent a single domain/feature
2. **Minimize Exports**: Only export what's necessary
3. **Avoid Circular Dependencies**: Use forwardRef() if unavoidable
4. **Keep Modules Focused**: If a module has too many providers, split it

```typescript
// ✅ Good: Focused module
@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}

// ❌ Bad: Too much responsibility
@Module({
  controllers: [UsersController, OrdersController, ProductsController],
  providers: [UsersService, OrdersService, ProductsService, ...],
})
export class EverythingModule {} // Split this!
```

**Production Tip**: Organize modules by business domain (Users, Orders, Products) rather than technical layers (Controllers, Services, Repositories) for better maintainability.

</details>

### 7. What is the Root Module and why is it required?

<details>
<summary>Answer</summary>

**Root Module** (typically `AppModule`) is the starting point of every NestJS application. It's required because NestJS uses it to build the application graph - the internal data structure that resolves module and provider relationships.

**Why It's Required:**

1. **Application Entry Point**: NestJS needs a starting point to bootstrap the application
2. **Dependency Resolution**: Root module is where dependency injection container begins resolving dependencies
3. **Module Graph**: NestJS traverses from root module to build the complete application structure
4. **Imports Other Modules**: Acts as the orchestrator that brings all feature modules together

**Basic Root Module:**

```typescript
// app.module.ts - Root Module
import { Module } from '@nestjs/common';
import { UsersModule } from './users/users.module';
import { ProductsModule } from './products/products.module';
import { AuthModule } from './auth/auth.module';

@Module({
  imports: [
    // Import all feature modules
    UsersModule,
    ProductsModule,
    AuthModule,
  ],
  controllers: [],  // Usually empty in root module
  providers: [],    // Global providers can go here
})
export class AppModule {}
```

**Bootstrapping with Root Module:**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';  // Root module

async function bootstrap() {
  // NestFactory.create() requires the root module
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

**Production Root Module Example:**

```typescript
// app.module.ts - Production-ready
@Module({
  imports: [
    // Configuration (loaded first)
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: '.env',
    }),
    
    // Database
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: (config: ConfigService) => ({
        type: 'postgres',
        host: config.get('DB_HOST'),
        port: config.get('DB_PORT'),
        database: config.get('DB_NAME'),
        autoLoadEntities: true,
      }),
      inject: [ConfigService],
    }),
    
    // Feature Modules
    AuthModule,
    UsersModule,
    ProductsModule,
    OrdersModule,
    
    // Global utilities
    LoggerModule,
    CacheModule.register({
      isGlobal: true,
      ttl: 300,
    }),
  ],
  controllers: [AppController],  // Health check endpoint
  providers: [
    AppService,
    // Global exception filter
    {
      provide: APP_FILTER,
      useClass: GlobalExceptionFilter,
    },
    // Global validation pipe
    {
      provide: APP_PIPE,
      useClass: ValidationPipe,
    },
  ],
})
export class AppModule {}
```

**What NOT to Put in Root Module:**

```typescript
// ❌ Bad: Feature-specific controllers/providers
@Module({
  imports: [UsersModule, ProductsModule],
  controllers: [
    UsersController,     // Should be in UsersModule
    ProductsController,  // Should be in ProductsModule
  ],
  providers: [
    UsersService,        // Should be in UsersModule
    ProductsService,     // Should be in ProductsModule
  ],
})
export class AppModule {}

// ✅ Good: Only imports and global config
@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),
    DatabaseModule,
    UsersModule,      // Controllers/Providers inside
    ProductsModule,   // Controllers/Providers inside
  ],
  providers: [
    // Only global providers (filters, pipes, guards)
    { provide: APP_FILTER, useClass: GlobalExceptionFilter },
  ],
})
export class AppModule {}
```

**Key Points:**

1. **Single Root Module**: Every app has exactly one root module
2. **Naming Convention**: Usually named `AppModule`
3. **Kept Minimal**: Should mostly just import other modules
4. **Global Providers**: Good place for APP_FILTER, APP_PIPE, APP_GUARD
5. **No Business Logic**: Keep business logic in feature modules

**Interview Tip**: The root module is like the main() function in other languages - it's where everything starts, but the real work happens in other modules.

</details>

### 8. What are the properties of the `@Module()` decorator (imports, providers, exports, controllers)?

<details>
<summary>Answer</summary>

The `@Module()` decorator accepts an object with four main properties that define the module's structure and dependencies:

**1. `imports`**: Array of modules whose exported providers are needed

```typescript
@Module({
  imports: [
    TypeOrmModule.forFeature([User]),  // Need User repository
    ConfigModule,                       // Need ConfigService
    AuthModule,                         // Need AuthService
  ],
})
export class UsersModule {}
```

**Purpose**: Import functionality from other modules
**Rules**: 
- Can only use providers that other modules have exported
- Creates dependencies between modules
- Order doesn't matter (NestJS resolves dependencies)

**2. `providers`**: Array of providers (services, repositories, factories) available in this module

```typescript
@Module({
  providers: [
    UsersService,              // Standard provider
    UsersRepository,           // Custom repository
    {
      provide: 'CONFIG',       // Custom token
      useValue: { apiKey: 'xxx' },
    },
    {
      provide: 'ASYNC_DATA',   // Factory provider
      useFactory: async () => {
        return await fetchData();
      },
    },
  ],
})
export class UsersModule {}
```

**Purpose**: Define services/providers that can be injected
**Rules**:
- Providers are available for injection within this module
- Not available outside unless exported
- Can use class, value, or factory providers

**3. `controllers`**: Array of controllers that handle HTTP requests

```typescript
@Module({
  controllers: [
    UsersController,      // Handles /users routes
    ProfileController,    // Handles /profile routes
  ],
})
export class UsersModule {}
```

**Purpose**: Define HTTP route handlers
**Rules**:
- Controllers are instantiated by NestJS
- Can inject providers from this module or imported modules
- Cannot be exported (controllers are module-specific)

**4. `exports`**: Array of providers to make available to other modules

```typescript
@Module({
  providers: [
    UsersService,      // Available internally
    InternalHelper,    // Available internally
    CacheService,      // Available internally
  ],
  exports: [
    UsersService,      // Available to other modules
    CacheService,      // Available to other modules
    // InternalHelper NOT exported - stays private
  ],
})
export class UsersModule {}
```

**Purpose**: Share providers with other modules
**Rules**:
- Only exported providers can be used by other modules
- Can export own providers or re-export imported modules
- Promotes encapsulation

**Complete Real-World Example:**

```typescript
// users.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';
import { User } from './entities/user.entity';
import { AuthModule } from '../auth/auth.module';
import { EmailModule } from '../email/email.module';

@Module({
  // 1. IMPORTS: Modules we depend on
  imports: [
    TypeOrmModule.forFeature([User]),  // Database access
    AuthModule,                         // Need AuthService
    EmailModule,                        // Need EmailService
  ],
  
  // 2. CONTROLLERS: HTTP endpoints
  controllers: [
    UsersController,                    // Handles /users routes
  ],
  
  // 3. PROVIDERS: Business logic & services
  providers: [
    UsersService,                       // Main service
    UsersRepository,                    // Data access
    UserValidationService,              // Internal validation
    {
      provide: 'USER_CONFIG',           // Configuration
      useValue: {
        maxLoginAttempts: 5,
        sessionTimeout: 3600,
      },
    },
  ],
  
  // 4. EXPORTS: What other modules can use
  exports: [
    UsersService,                       // Other modules need this
    // UsersRepository NOT exported - internal only
    // UserValidationService NOT exported - internal only
  ],
})
export class UsersModule {}
```

**Module Interaction Example:**

```typescript
// orders.module.ts - Uses UsersModule
@Module({
  imports: [
    UsersModule,  // Import to access exported UsersService
  ],
  controllers: [OrdersController],
  providers: [OrdersService],
})
export class OrdersModule {}

// orders.service.ts - Can inject UsersService
@Injectable()
export class OrdersService {
  constructor(
    private readonly usersService: UsersService,  // Available because UsersModule exported it
  ) {}
  
  async createOrder(userId: string) {
    const user = await this.usersService.findOne(userId);
    // Create order logic...
  }
}
```

**Re-exporting Modules:**

```typescript
// common.module.ts - Re-export shared modules
@Module({
  imports: [
    ConfigModule,
    LoggerModule,
    CacheModule,
  ],
  exports: [
    ConfigModule,   // Re-export so others don't need to import directly
    LoggerModule,   // Re-export
    CacheModule,    // Re-export
  ],
})
export class CommonModule {}

// Now other modules only need to import CommonModule
@Module({
  imports: [CommonModule],  // Gets Config, Logger, and Cache
})
export class UsersModule {}
```

**Best Practices:**

1. **Minimal Exports**: Only export what's necessary
2. **Explicit Imports**: Import exactly what you need
3. **No Circular Dependencies**: Avoid Module A importing B, B importing A
4. **Controllers Stay Private**: Never export controllers
5. **Group Related Items**: Keep related providers in same module

**Common Mistakes:**

```typescript
// ❌ Trying to inject non-exported provider
@Module({
  providers: [PrivateService],
  exports: [],  // Not exported!
})
export class ModuleA {}

@Module({
  imports: [ModuleA],
})
export class ModuleB {
  constructor(private service: PrivateService) {}  // ERROR!
}

// ✅ Correct: Export what's needed
@Module({
  providers: [PublicService],
  exports: [PublicService],  // Exported!
})
export class ModuleA {}
```

**Interview Tip**: Think of imports as "dependencies I need", providers as "services I have", controllers as "routes I handle", and exports as "services I share".

</details>
### 9. What is the difference between `imports` and `providers` in a module?

<details>
<summary>Answer</summary>

**`imports`** and **`providers`** serve completely different purposes in module configuration:

**`imports`**: External Dependencies (what you NEED from other modules)

```typescript
@Module({
  imports: [
    TypeOrmModule.forFeature([User]),  // Need database access
    AuthModule,                         // Need AuthService from AuthModule
    ConfigModule,                       // Need ConfigService
  ],
})
export class UsersModule {}
```

**Purpose**: Bring in functionality from other modules
**Contains**: Other modules
**Effect**: Makes exported providers from those modules available for injection
**Analogy**: Like `import` statements in JavaScript/TypeScript

**`providers`**: Internal Services (what you PROVIDE/OWN)

```typescript
@Module({
  providers: [
    UsersService,              // Service you implement
    UsersRepository,           // Repository you implement  
    PasswordHasher,            // Utility you implement
  ],
})
export class UsersModule {}
```

**Purpose**: Define services/classes that belong to this module
**Contains**: Classes decorated with @Injectable()
**Effect**: Makes these available for injection within this module
**Analogy**: Like class definitions in your file

**Side-by-Side Comparison:**

| Aspect | `imports` | `providers` |
|--------|-----------|-------------|
| **What** | Other modules | Your services |
| **Purpose** | Get dependencies | Define services |
| **Contains** | Modules | Injectable classes |
| **Scope** | Cross-module | Within module |
| **Example** | `AuthModule` | `UsersService` |

**Real-World Example:**

```typescript
// users.module.ts
@Module({
  // IMPORTS: External modules I depend on
  imports: [
    TypeOrmModule.forFeature([User]),   // From @nestjs/typeorm
    AuthModule,                          // My custom module (provides AuthService)
    EmailModule,                         // My custom module (provides EmailService)
  ],
  
  // PROVIDERS: Services I implement in this module
  providers: [
    UsersService,           // I implement this
    UsersRepository,        // I implement this
    UserValidationService,  // I implement this
  ],
  
  controllers: [UsersController],
  exports: [UsersService],  // Make UsersService available to others
})
export class UsersModule {}
```

**Using Both Together:**

```typescript
// users.service.ts
@Injectable()
export class UsersService {
  constructor(
    // From imports - AuthModule exports AuthService
    private readonly authService: AuthService,
    
    // From imports - TypeOrmModule provides repository
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
    
    // From providers - defined in same module
    private readonly userValidationService: UserValidationService,
  ) {}
  
  async createUser(dto: CreateUserDto) {
    // Use imported AuthService
    const hashedPassword = await this.authService.hashPassword(dto.password);
    
    // Use local provider
    await this.userValidationService.validateEmail(dto.email);
    
    // Use imported repository
    return this.userRepository.save({ ...dto, password: hashedPassword });
  }
}
```

**Common Scenarios:**

**Scenario 1: Using a service from another module**
```typescript
// auth.module.ts
@Module({
  providers: [AuthService],
  exports: [AuthService],  // Export so others can use it
})
export class AuthModule {}

// users.module.ts
@Module({
  imports: [AuthModule],  // Import the module
  providers: [UsersService],
})
export class UsersModule {}

// users.service.ts
@Injectable()
export class UsersService {
  constructor(private authService: AuthService) {}  // Can inject!
}
```

**Scenario 2: Self-contained module**
```typescript
@Module({
  imports: [],  // No external dependencies
  providers: [
    UtilityService,     // All internal
    HelperService,      // All internal
  ],
})
export class UtilityModule {}
```

**Key Rules:**

1. **imports = Modules**: Only put modules in imports
2. **providers = Classes**: Only put injectable classes in providers
3. **imports Makes Available**: Imported modules' exports become injectable
4. **providers Defines**: Providers define what this module offers
5. **Can't Cross-Inject**: Can't inject from another module without importing it

**Common Mistakes:**

```typescript
// ❌ WRONG: Putting a service in imports
@Module({
  imports: [UsersService],  // ERROR! Service, not a module
})

// ✅ CORRECT: Service goes in providers
@Module({
  providers: [UsersService],
})

// ❌ WRONG: Putting a module in providers
@Module({
  providers: [AuthModule],  // ERROR! Module, not a service
})

// ✅ CORRECT: Module goes in imports
@Module({
  imports: [AuthModule],
})

// ❌ WRONG: Forgetting to import before injecting
@Module({
  imports: [],  // AuthModule NOT imported
  providers: [UsersService],
})
export class UsersModule {}

@Injectable()
export class UsersService {
  constructor(private authService: AuthService) {}  // ERROR!
}

// ✅ CORRECT: Import first
@Module({
  imports: [AuthModule],  // Import the module
  providers: [UsersService],
})
export class UsersModule {}
```

**Mental Model:**

```
imports = "I need these modules" (dependencies IN)
providers = "I offer these services" (services I own)
exports = "Others can use these" (dependencies OUT)
```

**Interview Tip**: imports brings in external modules (like npm packages), while providers defines your own services (like your source code).

</details>

### 10. What is a Global Module and when would you use `@Global()`?

<details>
<summary>Answer</summary>

**Global Module** is a module decorated with `@Global()` that makes its exports available to all modules without explicitly importing them. It's NestJS's way of providing singleton services across the entire application.

**Basic Example:**

```typescript
import { Global, Module } from '@nestjs/common';

@Global()  // Makes this module's exports available everywhere
@Module({
  providers: [ConfigService, DatabaseService],
  exports: [ConfigService, DatabaseService],
})
export class CoreModule {}
```

**How It Works:**

```typescript
// Without @Global - Must import everywhere
@Module({
  imports: [ConfigModule],  // Must import
  providers: [UsersService],
})
export class UsersModule {}

@Module({
  imports: [ConfigModule],  // Must import again
  providers: [ProductsService],
})
export class ProductsModule {}

// With @Global - No imports needed
@Global()
@Module({
  providers: [ConfigService],
  exports: [ConfigService],
})
export class ConfigModule {}

@Module({
  // No import needed! Can still inject ConfigService
  providers: [UsersService],
})
export class UsersModule {}

@Module({
  // No import needed! Can still inject ConfigService
  providers: [ProductsService],
})
export class ProductsModule {}
```

**When to Use Global Modules:**

**1. Configuration Services**
```typescript
@Global()
@Module({
  providers: [
    {
      provide: ConfigService,
      useFactory: () => {
        return new ConfigService(`.env.${process.env.NODE_ENV}`);
      },
    },
  ],
  exports: [ConfigService],
})
export class ConfigModule {}

// Every module can now inject ConfigService without importing
@Injectable()
export class AnyService {
  constructor(private config: ConfigService) {}  // Works!
}
```

**2. Database Connections**
```typescript
@Global()
@Module({
  providers: [
    {
      provide: 'DATABASE_CONNECTION',
      useFactory: async () => {
        return await createConnection({
          type: 'postgres',
          // ... config
        });
      },
    },
  ],
  exports: ['DATABASE_CONNECTION'],
})
export class DatabaseModule {}
```

**3. Logger Services**
```typescript
@Global()
@Module({
  providers: [LoggerService],
  exports: [LoggerService],
})
export class LoggerModule {}

// Available everywhere without import
@Injectable()
export class UsersService {
  constructor(private logger: LoggerService) {
    this.logger.log('UsersService initialized');
  }
}
```

**4. Utility Services (Cache, HTTP Client)**
```typescript
@Global()
@Module({
  providers: [CacheService, HttpService],
  exports: [CacheService, HttpService],
})
export class SharedModule {}
```

**Real-World Production Example:**

```typescript
// core.module.ts - Global module with common services
import { Global, Module } from '@nestjs/common';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { LoggerService } from './logger/logger.service';
import { CacheService } from './cache/cache.service';

@Global()
@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,  // ConfigModule itself is global
      envFilePath: '.env',
    }),
  ],
  providers: [
    LoggerService,
    CacheService,
    {
      provide: 'APP_CONFIG',
      useFactory: (config: ConfigService) => ({
        apiUrl: config.get('API_URL'),
        apiKey: config.get('API_KEY'),
        environment: config.get('NODE_ENV'),
      }),
      inject: [ConfigService],
    },
  ],
  exports: [
    LoggerService,
    CacheService,
    'APP_CONFIG',
  ],
})
export class CoreModule {}

// app.module.ts - Import once in root
@Module({
  imports: [
    CoreModule,        // Import once
    UsersModule,
    ProductsModule,
    OrdersModule,
  ],
})
export class AppModule {}

// Any service can now use global providers
@Injectable()
export class UsersService {
  constructor(
    private logger: LoggerService,      // Available!
    private cache: CacheService,         // Available!
    @Inject('APP_CONFIG') private config: any,  // Available!
  ) {}
}
```

**Important Rules:**

1. **Import Once**: Global modules should be imported once (usually in root module)
2. **Use Sparingly**: Only for truly global services
3. **Registered Once**: Global module registration is once per application
4. **Still Need Export**: Must export providers even if global

**Best Practices:**

```typescript
// ✅ Good: Truly global services
@Global()
@Module({
  providers: [
    ConfigService,    // Used everywhere
    LoggerService,    // Used everywhere
    CacheService,     // Used everywhere
  ],
  exports: [ConfigService, LoggerService, CacheService],
})
export class CoreModule {}

// ❌ Bad: Feature-specific services as global
@Global()  // DON'T do this!
@Module({
  providers: [
    UsersService,      // Only needed in user feature
    ProductsService,   // Only needed in product feature
  ],
  exports: [UsersService, ProductsService],
})
export class BadGlobalModule {}  // This defeats modularity!
```

**Global vs Regular Module:**

```typescript
// Regular Module - Explicit imports required
@Module({
  providers: [FeatureService],
  exports: [FeatureService],
})
export class FeatureModule {}

// Every module that needs it must import
@Module({
  imports: [FeatureModule],  // Explicit
})
export class Module1 {}

@Module({
  imports: [FeatureModule],  // Explicit
})
export class Module2 {}

// Global Module - Available everywhere
@Global()
@Module({
  providers: [GlobalService],
  exports: [GlobalService],
})
export class GlobalModule {}

// No imports needed
@Module({})
export class Module1 {}  // Can inject GlobalService

@Module({})
export class Module2 {}  // Can inject GlobalService
```

**When NOT to Use Global:**

1. **Feature-specific services**: Keep in feature modules
2. **Domain logic**: Should be explicit dependencies
3. **Testing isolation**: Global makes unit testing harder
4. **Tight coupling**: Overuse creates hidden dependencies

**Using @nestjs/config as Global:**

```typescript
// Most common global module pattern
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,  // Makes ConfigService available everywhere
      envFilePath: '.env',
      validationSchema: configValidationSchema,
    }),
    UsersModule,
    ProductsModule,
  ],
})
export class AppModule {}

// Now ConfigService available in any module without import
@Injectable()
export class UsersService {
  constructor(private config: ConfigService) {
    const dbHost = this.config.get('DB_HOST');
  }
}
```

**Performance Consideration:**

Global modules don't affect performance - they're just about import convenience. The services are still singletons either way.

**Interview Tip**: Use `@Global()` for infrastructure services (config, logging, caching) that are truly needed everywhere, but keep domain/feature services in regular modules to maintain clear dependencies and modularity.

</details>
### 11. What is a Dynamic Module and when would you use it?

<details>
<summary>Answer</summary>

**Dynamic Module** is a module that is configured at runtime with specific options, rather than being statically defined. It returns a `DynamicModule` object instead of using the @Module() decorator statically.

**Purpose:**
- Configure modules with custom options at runtime
- Create reusable, configurable modules
- Support different configurations for different environments
- Enable flexible module initialization

**Static vs Dynamic Module:**

```typescript
// STATIC Module - Fixed configuration
@Module({
  providers: [ConfigService],
  exports: [ConfigService],
})
export class ConfigModule {}  // No customization

// DYNAMIC Module - Runtime configuration
@Module({})
export class ConfigModule {
  static forRoot(options: ConfigOptions): DynamicModule {
    return {
      module: ConfigModule,
      providers: [
        {
          provide: 'CONFIG_OPTIONS',
          useValue: options,  // Custom options passed at runtime
        },
        ConfigService,
      ],
      exports: [ConfigService],
    };
  }
}

// Usage: Configure with custom options
@Module({
  imports: [
    ConfigModule.forRoot({        // Pass options!
      envFilePath: '.env.production',
      isGlobal: true,
    }),
  ],
})
export class AppModule {}
```

**Real-World Example: Database Module**

```typescript
// database.module.ts
import { DynamicModule, Module } from '@nestjs/common';

export interface DatabaseOptions {
  type: 'postgres' | 'mysql';
  host: string;
  port: number;
  username: string;
  password: string;
  database: string;
}

@Module({})
export class DatabaseModule {
  // Static method that returns DynamicModule
  static forRoot(options: DatabaseOptions): DynamicModule {
    return {
      module: DatabaseModule,
      providers: [
        {
          provide: 'DATABASE_OPTIONS',
          useValue: options,  // Options available for injection
        },
        {
          provide: 'DATABASE_CONNECTION',
          useFactory: async (options: DatabaseOptions) => {
            // Create connection with provided options
            return await createConnection(options);
          },
          inject: ['DATABASE_OPTIONS'],
        },
        DatabaseService,
      ],
      exports: ['DATABASE_CONNECTION', DatabaseService],
      global: true,  // Optional: make it global
    };
  }
}

// Usage in AppModule
@Module({
  imports: [
    DatabaseModule.forRoot({
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'admin',
      password: 'password',
      database: 'myapp',
    }),
  ],
})
export class AppModule {}
```

**Common Patterns: `forRoot()` and `forFeature()`**

**1. `forRoot()` Pattern**: Configure module once (usually in AppModule)

```typescript
@Module({})
export class TypeOrmModule {
  static forRoot(options: TypeOrmModuleOptions): DynamicModule {
    return {
      module: TypeOrmModule,
      providers: [
        {
          provide: 'TYPEORM_MODULE_OPTIONS',
          useValue: options,
        },
        // Create database connection
      ],
      global: true,
    };
  }
}

// Usage: Configure once in root
@Module({
  imports: [
    TypeOrmModule.forRoot({  // Global configuration
      type: 'postgres',
      host: 'localhost',
      // ...
    }),
  ],
})
export class AppModule {}
```

**2. `forFeature()` Pattern**: Register feature-specific resources

```typescript
@Module({})
export class TypeOrmModule {
  static forFeature(entities: any[]): DynamicModule {
    return {
      module: TypeOrmModule,
      providers: [
        ...entities.map(entity => ({
          provide: `${entity.name}Repository`,
          useFactory: (connection) => connection.getRepository(entity),
          inject: ['DATABASE_CONNECTION'],
        })),
      ],
      exports: [...entities.map(e => `${e.name}Repository`)],
    };
  }
}

// Usage: Register entities per feature module
@Module({
  imports: [
    TypeOrmModule.forFeature([User, Profile]),  // Feature-specific
  ],
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}
```

**Complete Production Example:**

```typescript
// config.module.ts - Reusable configuration module
import { DynamicModule, Global, Module } from '@nestjs/common';
import { ConfigService } from './config.service';

export interface ConfigModuleOptions {
  envFilePath?: string;
  isGlobal?: boolean;
  validationSchema?: any;
  ignoreEnvFile?: boolean;
}

@Module({})
export class ConfigModule {
  // Synchronous configuration
  static forRoot(options: ConfigModuleOptions = {}): DynamicModule {
    const providers = [
      {
        provide: 'CONFIG_OPTIONS',
        useValue: options,
      },
      ConfigService,
    ];

    return {
      module: ConfigModule,
      providers,
      exports: [ConfigService],
      global: options.isGlobal ?? false,  // Make global if specified
    };
  }

  // Asynchronous configuration (when you need async setup)
  static forRootAsync(options: {
    useFactory: (...args: any[]) => Promise<ConfigModuleOptions>;
    inject?: any[];
  }): DynamicModule {
    return {
      module: ConfigModule,
      providers: [
        {
          provide: 'CONFIG_OPTIONS',
          useFactory: options.useFactory,
          inject: options.inject || [],
        },
        ConfigService,
      ],
      exports: [ConfigService],
    };
  }
}

// Usage Example 1: Simple configuration
@Module({
  imports: [
    ConfigModule.forRoot({
      envFilePath: '.env',
      isGlobal: true,
    }),
  ],
})
export class AppModule {}

// Usage Example 2: Async configuration with dependencies
@Module({
  imports: [
    ConfigModule.forRootAsync({
      useFactory: async (httpService: HttpService) => {
        // Fetch config from remote service
        const config = await httpService.get('https://api.config.com/settings');
        return {
          envFilePath: config.data.envPath,
          isGlobal: true,
        };
      },
      inject: [HttpService],
    }),
  ],
})
export class AppModule {}
```

**When to Use Dynamic Modules:**

**1. Reusable Libraries/Packages**
```typescript
// You're creating an NPM package that needs configuration
@Module({})
export class MyLibraryModule {
  static forRoot(options: LibraryOptions): DynamicModule {
    return { /* ... */ };
  }
}
```

**2. Environment-Specific Configuration**
```typescript
const config = process.env.NODE_ENV === 'production'
  ? { timeout: 5000, retries: 3 }
  : { timeout: 30000, retries: 1 };

CacheModule.forRoot(config);
```

**3. Feature Flags**
```typescript
LoggingModule.forRoot({
  enableDebug: process.env.DEBUG === 'true',
  logLevel: process.env.LOG_LEVEL,
});
```

**4. Multi-Tenant Applications**
```typescript
DatabaseModule.forRoot({
  database: `tenant_${tenantId}`,
  // Different DB per tenant
});
```

**DynamicModule Interface:**

```typescript
interface DynamicModule {
  module: Type<any>;              // The module class itself
  imports?: any[];                // Modules to import
  providers?: Provider[];         // Providers to register
  controllers?: Type<any>[];      // Controllers to register
  exports?: any[];                // Providers/modules to export
  global?: boolean;               // Make it a global module?
}
```

**Best Practices:**

```typescript
// ✅ Good: Type-safe options
export interface MyModuleOptions {
  apiKey: string;
  timeout?: number;
}

static forRoot(options: MyModuleOptions): DynamicModule {
  // Validate options
  if (!options.apiKey) {
    throw new Error('apiKey is required');
  }
  
  return { /* ... */ };
}

// ✅ Good: Provide defaults
static forRoot(options: Partial<MyModuleOptions> = {}): DynamicModule {
  const finalOptions = {
    timeout: 5000,      // Default
    retries: 3,         // Default
    ...options,         // User overrides
  };
  
  return { /* ... */ };
}

// ✅ Good: Support both sync and async
static forRoot(options: MyModuleOptions): DynamicModule { /* ... */ }
static forRootAsync(options: MyModuleAsyncOptions): DynamicModule { /* ... */ }
```

**Common Mistake:**

```typescript
// ❌ Wrong: Trying to use @Module decorator with forRoot
@Module({  // Don't do this!
  imports: [ConfigModule],
})
export class ConfigModule {
  static forRoot() { /* ... */ }
}

// ✅ Correct: Empty @Module decorator
@Module({})  // Keep it empty
export class ConfigModule {
  static forRoot() { /* ... */ }
}
```

**Interview Tip**: Dynamic modules are like functions that return modules with custom configuration - think of `forRoot()` as a factory method that creates a configured module instance.

</details>

### 12. What is the difference between `forRoot()` and `forFeature()` methods?

<details>
<summary>Answer</summary>

**`forRoot()`** and **`forFeature()`** are naming conventions for dynamic module methods that serve different purposes in module configuration:

**`forRoot()`**: Global, One-Time Configuration
- Called once in the root module (AppModule)
- Sets up global/shared resources (database connection, configuration)
- Typically creates singletons
- Usually marked as global

**`forFeature()`**: Feature-Specific Configuration  
- Called in each feature module
- Registers feature-specific resources (entities, repositories)
- Depends on forRoot() being called first
- Not global, scoped to the importing module

**Side-by-Side Comparison:**

| Aspect | `forRoot()` | `forFeature()` |
|--------|-------------|----------------|
| **Called** | Once (in AppModule) | Multiple times (per feature) |
| **Purpose** | Global setup | Feature-specific setup |
| **Example** | Database connection | Register entities |
| **Scope** | Global/Singleton | Per module |
| **Depends On** | Nothing | Requires forRoot() |

**Real-World Example: TypeORM**

```typescript
// app.module.ts - Root Module
@Module({
  imports: [
    // forRoot() - Called ONCE for global database connection
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'admin',
      password: 'password',
      database: 'myapp',
      synchronize: false,
      // This creates the DATABASE CONNECTION (singleton)
    }),
    
    UsersModule,
    ProductsModule,
    OrdersModule,
  ],
})
export class AppModule {}
```

```typescript
// users.module.ts - Feature Module
@Module({
  imports: [
    // forFeature() - Register User-specific entities
    TypeOrmModule.forFeature([User, Profile]),
    // This gives us User and Profile repositories
  ],
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}

// products.module.ts - Another Feature Module
@Module({
  imports: [
    // forFeature() - Register Product-specific entities
    TypeOrmModule.forFeature([Product, Category]),
    // This gives us Product and Category repositories
  ],
  controllers: [ProductsController],
  providers: [ProductsService],
})
export class ProductsModule {}
```

**Implementation Example:**

```typescript
// typeorm.module.ts
import { DynamicModule, Module } from '@nestjs/common';

@Module({})
export class TypeOrmModule {
  // forRoot() - Create the database connection (called once)
  static forRoot(options: ConnectionOptions): DynamicModule {
    return {
      module: TypeOrmModule,
      providers: [
        {
          provide: 'DATABASE_CONNECTION',
          useFactory: async () => {
            // Create ONE database connection for entire app
            return await createConnection(options);
          },
        },
      ],
      exports: ['DATABASE_CONNECTION'],
      global: true,  // Make connection available everywhere
    };
  }

  // forFeature() - Register repositories (called per feature module)
  static forFeature(entities: any[]): DynamicModule {
    return {
      module: TypeOrmModule,
      providers: [
        // Create repository for each entity
        ...entities.map(entity => ({
          provide: `${entity.name}Repository`,
          useFactory: (connection: Connection) => {
            // Use the global connection created by forRoot()
            return connection.getRepository(entity);
          },
          inject: ['DATABASE_CONNECTION'],
        })),
      ],
      exports: [
        // Export repositories so services can inject them
        ...entities.map(e => `${e.name}Repository`),
      ],
    };
  }
}
```

**Usage Flow:**

```typescript
// Step 1: forRoot() in AppModule (creates connection)
TypeOrmModule.forRoot({
  type: 'postgres',
  // ... connection options
})  // → Creates global database connection

// Step 2: forFeature() in UsersModule (registers entities)
TypeOrmModule.forFeature([User, Profile])
// → Creates UserRepository and ProfileRepository
// → Uses the connection from forRoot()

// Step 3: Inject and use in service
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)  // Provided by forFeature()
    private userRepository: Repository<User>,
  ) {}
}
```

**Another Example: ConfigModule**

```typescript
// config.module.ts
@Module({})
export class ConfigModule {
  // forRoot() - Load configuration once
  static forRoot(options: ConfigOptions): DynamicModule {
    return {
      module: ConfigModule,
      providers: [
        {
          provide: 'CONFIG',
          useFactory: () => {
            // Load .env file, validate, etc.
            return loadConfig(options);
          },
        },
        ConfigService,
      ],
      exports: [ConfigService],
      global: true,
    };
  }

  // forFeature() - Load feature-specific config
  static forFeature(configKey: string): DynamicModule {
    return {
      module: ConfigModule,
      providers: [
        {
          provide: `${configKey}_CONFIG`,
          useFactory: (config: ConfigService) => {
            // Get feature-specific configuration
            return config.get(configKey);
          },
          inject: [ConfigService],
        },
      ],
      exports: [`${configKey}_CONFIG`],
    };
  }
}

// Usage
@Module({
  imports: [
    ConfigModule.forRoot({ envFilePath: '.env' }),  // Once
  ],
})
export class AppModule {}

@Module({
  imports: [
    ConfigModule.forFeature('database'),  // Get database config
  ],
})
export class DatabaseModule {}

@Module({
  imports: [
    ConfigModule.forFeature('auth'),  // Get auth config
  ],
})
export class AuthModule {}
```

**Key Differences Illustrated:**

```typescript
// forRoot() creates SHARED resources
TypeOrmModule.forRoot({ /* ... */ })
→ Creates 1 database connection
→ Used by ALL modules
→ Singleton

// forFeature() creates MODULE-SPECIFIC resources  
UsersModule:
  TypeOrmModule.forFeature([User]) → UserRepository
ProductsModule:
  TypeOrmModule.forFeature([Product]) → ProductRepository
OrdersModule:
  TypeOrmModule.forFeature([Order]) → OrderRepository

→ Each module gets its own repositories
→ All repositories use the same connection from forRoot()
```

**Common Patterns:**

**Pattern 1: Database Module**
```typescript
forRoot()    → Database connection
forFeature() → Entity repositories
```

**Pattern 2: Config Module**
```typescript
forRoot()    → Load all configuration
forFeature() → Access specific config sections
```

**Pattern 3: HTTP Module**
```typescript
forRoot()    → Global HTTP client config
forFeature() → Service-specific HTTP config
```

**Best Practices:**

```typescript
// ✅ Good: forRoot() in AppModule only
@Module({
  imports: [
    TypeOrmModule.forRoot({ /* ... */ }),  // Once
    UsersModule,
    ProductsModule,
  ],
})
export class AppModule {}

// ❌ Bad: Calling forRoot() multiple times
@Module({
  imports: [
    TypeOrmModule.forRoot({ /* ... */ }),  // Creates connection 1
  ],
})
export class AppModule {}

@Module({
  imports: [
    TypeOrmModule.forRoot({ /* ... */ }),  // Creates connection 2 (WRONG!)
  ],
})
export class UsersModule {}  // Don't do this!

// ✅ Good: forFeature() in feature modules
@Module({
  imports: [
    TypeOrmModule.forFeature([User]),  // ✓
  ],
})
export class UsersModule {}

@Module({
  imports: [
    TypeOrmModule.forFeature([Product]),  // ✓
  ],
})
export class ProductsModule {}
```

**What If You Forget forRoot()?**

```typescript
// If you call forFeature() without calling forRoot() first:
@Module({
  imports: [
    // TypeOrmModule.forRoot() NOT called!
    TypeOrmModule.forFeature([User]),  // ERROR!
  ],
})
// → Error: No database connection available
```

**Mental Model:**

```
forRoot()    = "Set up the foundation" (once)
forFeature() = "Add rooms to the house" (many times)

forRoot()    = "Create the database connection" (once) 
forFeature() = "Register entities that use that connection" (per module)

forRoot()    = "Global configuration" (singleton)
forFeature() = "Use that configuration" (per feature)
```

**Interview Tip**: 
- **forRoot() = Infrastructure** (call once in AppModule)
- **forFeature() = Features** (call in each feature module)
- forFeature() depends on forRoot() being called first

</details>
### 13. How do you resolve circular dependencies between modules?

<details>
<summary>Answer</summary>

**Circular dependency** occurs when Module A imports Module B, and Module B imports Module A, creating a cycle. NestJS provides `forwardRef()` to resolve this, but it's better to refactor to avoid them entirely.

**What is a Circular Dependency?**

```typescript
// users.module.ts
@Module({
  imports: [OrdersModule],  // Imports OrdersModule
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}

// orders.module.ts
@Module({
  imports: [UsersModule],  // Imports UsersModule
  providers: [OrdersService],
  exports: [OrdersService],
})
export class OrdersModule {}

// → CIRCULAR DEPENDENCY: Users → Orders → Users
```

**Solution 1: Using `forwardRef()` (Quick Fix)**

```typescript
import { forwardRef } from '@nestjs/common';

// users.module.ts
@Module({
  imports: [forwardRef(() => OrdersModule)],  // Lazy reference
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}

// orders.module.ts
@Module({
  imports: [forwardRef(() => UsersModule)],  // Lazy reference
  providers: [OrdersService],
  exports: [OrdersService],
})
export class OrdersModule {}

// → forwardRef() delays resolution until runtime
```

**How `forwardRef()` Works:**

```typescript
// Without forwardRef - Immediate evaluation
imports: [UsersModule]  // → UsersModule must be defined NOW

// With forwardRef - Lazy evaluation
imports: [forwardRef(() => UsersModule)]  // → Evaluated later
```

**Complete Example with Services:**

```typescript
// users.module.ts
import { Module, forwardRef } from '@nestjs/common';
import { UsersService } from './users.service';
import { OrdersModule } from '../orders/orders.module';

@Module({
  imports: [forwardRef(() => OrdersModule)],
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}

// users.service.ts
import { Injectable, Inject, forwardRef } from '@nestjs/common';
import { OrdersService } from '../orders/orders.service';

@Injectable()
export class UsersService {
  constructor(
    @Inject(forwardRef(() => OrdersService))  // forwardRef in service too!
    private ordersService: OrdersService,
  ) {}

  async getUserOrders(userId: string) {
    return this.ordersService.findByUserId(userId);
  }
}

// orders.module.ts
import { Module, forwardRef } from '@nestjs/common';
import { OrdersService } from './orders.service';
import { UsersModule } from '../users/users.module';

@Module({
  imports: [forwardRef(() => UsersModule)],
  providers: [OrdersService],
  exports: [OrdersService],
})
export class OrdersModule {}

// orders.service.ts
import { Injectable, Inject, forwardRef } from '@nestjs/common';
import { UsersService } from '../users/users.service';

@Injectable()
export class OrdersService {
  constructor(
    @Inject(forwardRef(() => UsersService))  // forwardRef in service too!
    private usersService: UsersService,
  ) {}

  async getOrderWithUser(orderId: string) {
    const order = await this.findOne(orderId);
    const user = await this.usersService.findOne(order.userId);
    return { ...order, user };
  }
}
```

**Solution 2: Extract Shared Module (RECOMMENDED)**

```typescript
// Instead of circular dependency, create a shared module

// shared.module.ts - Contains shared services
@Module({
  providers: [UserHelperService, OrderHelperService],
  exports: [UserHelperService, OrderHelperService],
})
export class SharedModule {}

// users.module.ts
@Module({
  imports: [SharedModule],  // No circular dependency
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}

// orders.module.ts
@Module({
  imports: [SharedModule],  // No circular dependency
  providers: [OrdersService],
  exports: [OrdersService],
})
export class OrdersModule {}
```

**Solution 3: Use Events (Decoupling)**

```typescript
// users.service.ts - Emit event instead of calling OrdersService
import { Injectable } from '@nestjs/common';
import { EventEmitter2 } from '@nestjs/event-emitter';

@Injectable()
export class UsersService {
  constructor(private eventEmitter: EventEmitter2) {}

  async createUser(userData: any) {
    const user = await this.userRepository.save(userData);
    
    // Emit event instead of calling OrdersService
    this.eventEmitter.emit('user.created', { userId: user.id });
    
    return user;
  }
}

// orders.service.ts - Listen to event
import { Injectable } from '@nestjs/common';
import { OnEvent } from '@nestjs/event-emitter';

@Injectable()
export class OrdersService {
  @OnEvent('user.created')
  handleUserCreated(payload: { userId: string }) {
    // React to user creation without importing UsersModule
    console.log('User created:', payload.userId);
  }
}
```

**Solution 4: Dependency Injection at Controller Level**

```typescript
// Instead of services depending on each other,
// let controllers orchestrate both services

// users.controller.ts
@Controller('users')
export class UsersController {
  constructor(
    private usersService: UsersService,
    private ordersService: OrdersService,  // Inject both here
  ) {}

  @Get(':id/orders')
  async getUserOrders(@Param('id') id: string) {
    const user = await this.usersService.findOne(id);
    const orders = await this.ordersService.findByUserId(id);
    return { user, orders };
  }
}

// No circular dependency between services!
```

**Solution 5: Refactor to Single Module**

```typescript
// If Users and Orders are tightly coupled, maybe they should be one module

@Module({
  imports: [TypeOrmModule.forFeature([User, Order])],
  controllers: [UsersController, OrdersController],
  providers: [UsersService, OrdersService],
  exports: [UsersService, OrdersService],
})
export class UserOrderModule {}  // Combined module
```

**Best Practices:**

**1. Avoid Circular Dependencies**
```typescript
// ❌ Bad: Circular dependency
UsersModule ↔ OrdersModule

// ✅ Good: One-way dependency
UsersModule → OrdersModule
// OrdersModule doesn't import UsersModule
```

**2. Use Shared Module**
```typescript
// ✅ Good: Extract common logic
UsersModule → SharedModule ← OrdersModule
```

**3. Use Events for Loose Coupling**
```typescript
// ✅ Good: Event-driven architecture
UsersService emits 'user.created'
OrdersService listens to 'user.created'
// No direct dependency
```

**4. Rethink Architecture**
```typescript
// Question: Do these modules really need each other?
// Maybe the design needs refactoring
```

**When `forwardRef()` is Acceptable:**

1. **Quick fix during development** (but refactor later)
2. **Legacy code migration** (temporary solution)
3. **Genuinely bidirectional relationship** (rare, usually means design issue)

**When to Avoid `forwardRef()`:**

1. **New projects** - Design properly from the start
2. **Production code** - Refactor instead
3. **Code smells** - Usually indicates poor architecture

**Common Circular Dependency Scenarios:**

**Scenario 1: Parent-Child Relationship**
```typescript
// ❌ Bad
UsersModule → ProfileModule
ProfileModule → UsersModule

// ✅ Good: Profile depends on User, not vice versa
ProfileModule → UsersModule
// User doesn't need to know about Profile
```

**Scenario 2: Aggregation**
```typescript
// ❌ Bad
UsersModule ↔ OrdersModule

// ✅ Good: Create aggregation service
UserOrdersModule → UsersModule
UserOrdersModule → OrdersModule
```

**Testing with Circular Dependencies:**

```typescript
// Testing with forwardRef
describe('UsersService', () => {
  let service: UsersService;
  let ordersService: OrdersService;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: OrdersService,
          useValue: {
            findByUserId: jest.fn(),  // Mock it
          },
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
  });
});
```

**Error Messages:**

```typescript
// Common error without forwardRef:
// "Nest can't resolve dependencies of the UsersService (?). 
//  Please make sure that the argument dependency at index [0] 
//  is available in the UsersModule context."

// This often means circular dependency!
```

**Interview Tip**: While `forwardRef()` solves circular dependencies technically, it's usually a sign of poor architecture. In interviews, mention forwardRef() but emphasize refactoring strategies like shared modules, events, or better module boundaries.

</details>

## Controllers

### 14. What is a Controller in NestJS and what is its responsibility?

<details>
<summary>Answer</summary>

**Controller** is a class decorated with `@Controller()` that handles incoming HTTP requests and returns responses to the client. It's responsible for routing, request handling, and delegating business logic to services.

**Key Responsibilities:**

1. **Route Handling**: Map HTTP requests to handler methods
2. **Request Processing**: Extract parameters, query strings, body, headers
3. **Delegation**: Call services to execute business logic
4. **Response Formatting**: Return data to client
5. **HTTP Layer Only**: Should NOT contain business logic

**Basic Controller Example:**

```typescript
import { Controller, Get, Post, Body, Param } from '@nestjs/common';
import { UsersService } from './users.service';

@Controller('users')  // Route prefix: /users
export class UsersController {
  constructor(private readonly usersService: UsersService) {}  // Inject service

  @Get()  // GET /users
  findAll() {
    return this.usersService.findAll();  // Delegate to service
  }

  @Get(':id')  // GET /users/:id
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(id);
  }

  @Post()  // POST /users
  create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }
}
```

**Controller Architecture:**

```
HTTP Request
    ↓
Controller (Handles routing, extracts data)
    ↓
Service (Business logic)
    ↓
Repository (Database access)
    ↓
Database
```

**Production-Grade Controller:**

```typescript
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
  UsePipes,
  ValidationPipe,
} from '@nestjs/common';
import { UsersService } from './users.service';
import { CreateUserDto, UpdateUserDto } from './dto';
import { JwtAuthGuard } from '../auth/guards/jwt-auth.guard';
import { RolesGuard } from '../auth/guards/roles.guard';
import { Roles } from '../auth/decorators/roles.decorator';

@Controller('users')  // Base route: /users
@UseGuards(JwtAuthGuard)  // Apply authentication to all routes
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  // GET /users?page=1&limit=10
  @Get()
  @HttpCode(HttpStatus.OK)
  async findAll(
    @Query('page') page: number = 1,
    @Query('limit') limit: number = 10,
  ) {
    return this.usersService.findAll({ page, limit });
  }

  // GET /users/:id
  @Get(':id')
  async findOne(@Param('id') id: string) {
    return this.usersService.findOne(id);
  }

  // POST /users
  @Post()
  @HttpCode(HttpStatus.CREATED)
  @UsePipes(new ValidationPipe({ whitelist: true }))  // Validate DTO
  async create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }

  // PUT /users/:id
  @Put(':id')
  @UseGuards(RolesGuard)
  @Roles('admin')  // Only admins can update
  async update(
    @Param('id') id: string,
    @Body() updateUserDto: UpdateUserDto,
  ) {
    return this.usersService.update(id, updateUserDto);
  }

  // DELETE /users/:id
  @Delete(':id')
  @UseGuards(RolesGuard)
  @Roles('admin')  // Only admins can delete
  @HttpCode(HttpStatus.NO_CONTENT)
  async remove(@Param('id') id: string) {
    await this.usersService.remove(id);
  }
}
```

**What Controllers Should Do:**

**✅ DO:**
```typescript
@Controller('users')
export class UsersController {
  constructor(private usersService: UsersService) {}

  @Get(':id')
  async findOne(@Param('id') id: string) {
    // ✅ Extract parameters
    // ✅ Call service
    // ✅ Return response
    return this.usersService.findOne(id);
  }

  @Post()
  @UsePipes(ValidationPipe)  // ✅ Apply validation
  async create(@Body() dto: CreateUserDto) {
    // ✅ Delegate to service
    return this.usersService.create(dto);
  }
}
```

**What Controllers Should NOT Do:**

**❌ DON'T:**
```typescript
@Controller('users')
export class UsersController {
  constructor(private userRepository: Repository<User>) {}

  @Post()
  async create(@Body() dto: CreateUserDto) {
    // ❌ Business logic in controller
    if (dto.age < 18) {
      throw new BadRequestException('Must be 18+');
    }

    // ❌ Direct database access in controller
    const user = this.userRepository.create(dto);
    await this.userRepository.save(user);

    // ❌ Complex data transformation in controller
    const userData = {
      fullName: `${dto.firstName} ${dto.lastName}`,
      email: dto.email.toLowerCase(),
      // ...
    };

    return userData;
  }
}

// ✅ CORRECT: All logic in service
@Controller('users')
export class UsersController {
  constructor(private usersService: UsersService) {}

  @Post()
  async create(@Body() dto: CreateUserDto) {
    return this.usersService.create(dto);  // Service handles everything
  }
}

@Injectable()
export class UsersService {
  constructor(private userRepository: Repository<User>) {}

  async create(dto: CreateUserDto) {
    // ✅ Business logic in service
    if (dto.age < 18) {
      throw new BadRequestException('Must be 18+');
    }

    // ✅ Database access in service
    const user = this.userRepository.create(dto);
    return this.userRepository.save(user);
  }
}
```

**Controller Scope:**

**Responsibilities:**
- HTTP request/response handling
- Route definitions
- Parameter extraction
- Applying guards, pipes, interceptors
- Delegating to services

**NOT Responsibilities:**
- Business logic
- Data validation logic
- Database operations
- Complex calculations
- External API calls

**Controller Lifecycle:**

```typescript
1. Request arrives → /users/123
2. NestJS routes to controller → UsersController.findOne()
3. Decorators execute → Guards, Pipes, Interceptors
4. Controller extracts parameters → id = '123'
5. Controller calls service → usersService.findOne('123')
6. Service returns data → User object
7. Controller returns response → HTTP 200 + JSON
```

**Route Prefix:**

```typescript
// Controller prefix
@Controller('users')  // Base: /users
export class UsersController {
  @Get()           // Route: GET /users
  @Get(':id')      // Route: GET /users/:id
  @Post()          // Route: POST /users
  @Put(':id')      // Route: PUT /users/:id
  @Delete(':id')   // Route: DELETE /users/:id
}

// Nested routes
@Controller('users/:userId/orders')  // Base: /users/:userId/orders
export class UserOrdersController {
  @Get()           // Route: GET /users/:userId/orders
  @Get(':orderId') // Route: GET /users/:userId/orders/:orderId
}
```

**Multiple Controllers:**

```typescript
// One controller per resource
@Controller('users')
export class UsersController {}  // Handles /users

@Controller('products')
export class ProductsController {}  // Handles /products

@Controller('orders')
export class OrdersController {}  // Handles /orders
```

**Best Practices:**

1. **Thin Controllers**: Keep controllers lean, logic in services
2. **One Resource Per Controller**: UsersController for users, ProductsController for products
3. **Consistent Naming**: `{Resource}Controller` (e.g., UsersController)
4. **Use DTOs**: Type-safe request/response objects
5. **Apply Guards/Pipes**: Use decorators for cross-cutting concerns
6. **Async/Await**: Always use async for database operations

**Testing Controllers:**

```typescript
// Unit test - Mock the service
describe('UsersController', () => {
  let controller: UsersController;
  let service: UsersService;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      controllers: [UsersController],
      providers: [
        {
          provide: UsersService,
          useValue: {
            findAll: jest.fn().mockResolvedValue([]),
            findOne: jest.fn(),
            create: jest.fn(),
          },
        },
      ],
    }).compile();

    controller = module.get<UsersController>(UsersController);
    service = module.get<UsersService>(UsersService);
  });

  it('should return all users', async () => {
    const result = await controller.findAll();
    expect(service.findAll).toHaveBeenCalled();
  });
});
```

**Interview Tip**: Controllers are like receptionists - they greet requests, understand what's needed, delegate work to the right department (service), and deliver the response. They don't do the actual work themselves!

</details>
### 15. What is the `@Controller()` decorator and how do you define route prefixes?

<details>
<summary>Answer</summary>

**`@Controller()`** decorator marks a class as a controller and optionally defines a route prefix for all routes in that controller. It tells NestJS that this class handles HTTP requests.

**Basic Usage:**

```typescript
import { Controller } from '@nestjs/common';

// Without route prefix
@Controller()
export class AppController {
  @Get('home')  // Route: GET /home
  home() {
    return 'Home page';
  }
}

// With route prefix
@Controller('users')  // Prefix all routes with /users
export class UsersController {
  @Get()        // Route: GET /users
  @Get(':id')   // Route: GET /users/:id
  @Post()       // Route: POST /users
}
```

**Route Prefix Benefits:**

```typescript
// WITHOUT prefix - Repetitive
@Controller()
export class UsersController {
  @Get('users')           // GET /users
  @Get('users/:id')       // GET /users/:id
  @Post('users')          // POST /users
  @Put('users/:id')       // PUT /users/:id
  @Delete('users/:id')    // DELETE /users/:id
}

// WITH prefix - DRY (Don't Repeat Yourself)
@Controller('users')  // Prefix: /users
export class UsersController {
  @Get()          // GET /users
  @Get(':id')     // GET /users/:id
  @Post()         // POST /users
  @Put(':id')     // PUT /users/:id
  @Delete(':id')  // DELETE /users/:id
}
```

**Multiple Route Segments:**

```typescript
// Single segment
@Controller('users')  // /users/*

// Multiple segments
@Controller('api/v1/users')  // /api/v1/users/*

// With parameters
@Controller('users/:userId/orders')  // /users/:userId/orders/*
export class UserOrdersController {
  @Get()  // GET /users/:userId/orders
  findAll(@Param('userId') userId: string) {
    return `Orders for user ${userId}`;
  }
  
  @Get(':orderId')  // GET /users/:userId/orders/:orderId
  findOne(
    @Param('userId') userId: string,
    @Param('orderId') orderId: string,
  ) {
    return `Order ${orderId} for user ${userId}`;
  }
}
```

**Production Example:**

```typescript
// users.controller.ts
import { Controller, Get, Post, Put, Delete, Param, Body } from '@nestjs/common';

@Controller('api/users')  // Prefix: /api/users
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()              // GET /api/users
  findAll() {
    return this.usersService.findAll();
  }

  @Get(':id')         // GET /api/users/:id
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(id);
  }

  @Post()             // POST /api/users
  create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }

  @Put(':id')         // PUT /api/users/:id
  update(@Param('id') id: string, @Body() updateUserDto: UpdateUserDto) {
    return this.usersService.update(id, updateUserDto);
  }

  @Delete(':id')      // DELETE /api/users/:id
  remove(@Param('id') id: string) {
    return this.usersService.remove(id);
  }
}
```

**Versioning with Route Prefix:**

```typescript
// API versioning using controller prefix

// Version 1
@Controller('api/v1/users')
export class UsersV1Controller {
  @Get()  // GET /api/v1/users
  findAll() {
    return this.usersService.findAllV1();
  }
}

// Version 2  
@Controller('api/v2/users')
export class UsersV2Controller {
  @Get()  // GET /api/v2/users
  findAll() {
    return this.usersService.findAllV2();  // Different implementation
  }
}
```

**Dynamic Route Prefix:**

```typescript
// Using environment variable or config
const apiPrefix = process.env.API_PREFIX || 'api';

@Controller(`${apiPrefix}/users`)  // /api/users or custom prefix
export class UsersController {}

// Or use global prefix in main.ts instead
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.setGlobalPrefix('api');  // All routes prefixed with /api
  await app.listen(3000);
}
```

**Nested Resources:**

```typescript
// Parent-child relationship

// Users controller
@Controller('users')
export class UsersController {
  @Get(':id')  // GET /users/:id
}

// User posts (nested resource)
@Controller('users/:userId/posts')
export class UserPostsController {
  @Get()  // GET /users/:userId/posts
  findUserPosts(@Param('userId') userId: string) {
    return this.postsService.findByUserId(userId);
  }
  
  @Get(':postId')  // GET /users/:userId/posts/:postId
  findOne(
    @Param('userId') userId: string,
    @Param('postId') postId: string,
  ) {
    return this.postsService.findOne(userId, postId);
  }
}
```

**Controller Configuration Options:**

```typescript
// Basic
@Controller('users')

// With options object (advanced)
@Controller({
  path: 'users',
  host: 'admin.example.com',  // Subdomain routing
})
export class AdminUsersController {}

// Host-based routing
@Controller({
  host: ':tenant.example.com',  // Multi-tenant
  path: 'users',
})
export class TenantUsersController {
  @Get()
  findAll(@HostParam('tenant') tenant: string) {
    return `Users for tenant: ${tenant}`;
  }
}
```

**Best Practices:**

```typescript
// ✅ Good: Resource-based naming
@Controller('users')      // Handles user resources
@Controller('products')   // Handles product resources
@Controller('orders')     // Handles order resources

// ✅ Good: Include API versioning
@Controller('api/v1/users')

// ✅ Good: RESTful naming (plural nouns)
@Controller('users')      // Plural
@Controller('products')   // Plural

// ❌ Bad: Verb-based naming
@Controller('getUsers')   // Don't use verbs
@Controller('createUser') // Don't use verbs

// ❌ Bad: Mixing conventions
@Controller('user')       // Singular - be consistent
@Controller('products')   // Plural
```

**Empty Controller (No Prefix):**

```typescript
// Sometimes used for root-level routes
@Controller()
export class AppController {
  @Get()         // GET /
  root() {
    return 'API Root';
  }
  
  @Get('health') // GET /health
  health() {
    return { status: 'ok' };
  }
}
```

**Module Registration:**

```typescript
// Controllers must be registered in a module
@Module({
  controllers: [UsersController],  // Register here
  providers: [UsersService],
})
export class UsersModule {}
```

**Route Conflicts:**

```typescript
// ❌ Avoid conflicting routes
@Controller('users')
export class UsersController {
  @Get(':id')       // GET /users/:id
  @Get('active')    // GET /users/active - Will never match!
                    // 'active' will be interpreted as :id
}

// ✅ Correct: Specific routes before parameterized routes
@Controller('users')
export class UsersController {
  @Get('active')    // GET /users/active - Matches first
  @Get(':id')       // GET /users/:id - Matches if not 'active'
}
```

**Interview Tip**: `@Controller()` is like the building address - it tells NestJS where this controller "lives" in your API's URL structure. All routes inside inherit this prefix.

</details>

### 16. What is the difference between `@Param()`, `@Query()`, and `@Body()` decorators?

<details>
<summary>Answer</summary>

These decorators extract different parts of the HTTP request:

**Quick Comparison:**

| Decorator | Extracts From | Example | Use Case |
|-----------|---------------|---------|----------|
| `@Param()` | URL path | `/users/:id` | Resource identification |
| `@Query()` | Query string | `/users?page=1` | Filtering, pagination, sorting |
| `@Body()` | Request body | POST/PUT data | Creating/updating resources |

**1. `@Param()` - URL Path Parameters**

Extracts dynamic segments from the URL path.

```typescript
@Controller('users')
export class UsersController {
  // GET /users/123
  @Get(':id')
  findOne(@Param('id') id: string) {
    return `User ID: ${id}`;  // id = '123'
  }

  // GET /users/123/posts/456
  @Get(':userId/posts/:postId')
  findUserPost(
    @Param('userId') userId: string,
    @Param('postId') postId: string,
  ) {
    return `User ${userId}, Post ${postId}`;
  }

  // Get all params as object
  @Get(':userId/posts/:postId')
  findUserPost2(@Param() params: any) {
    console.log(params);  // { userId: '123', postId: '456' }
    return params;
  }
}
```

**2. `@Query()` - Query String Parameters**

Extracts parameters from the query string (after `?`).

```typescript
@Controller('users')
export class UsersController {
  // GET /users?page=1&limit=10
  @Get()
  findAll(
    @Query('page') page: number,
    @Query('limit') limit: number,
  ) {
    return `Page ${page}, Limit ${limit}`;  // page=1, limit=10
  }

  // GET /users?page=1&limit=10&sort=name&order=asc
  @Get()
  findAllAdvanced(
    @Query('page') page: number = 1,      // Default values
    @Query('limit') limit: number = 10,
    @Query('sort') sort: string = 'id',
    @Query('order') order: 'asc' | 'desc' = 'asc',
  ) {
    return { page, limit, sort, order };
  }

  // Get all query params as object
  @Get()
  findAll2(@Query() query: any) {
    console.log(query);  // { page: '1', limit: '10', sort: 'name' }
    return query;
  }

  // With DTO for type safety
  @Get()
  findAll3(@Query() query: PaginationDto) {
    return this.usersService.findAll(query);
  }
}

// pagination.dto.ts
export class PaginationDto {
  @IsOptional()
  @IsInt()
  @Type(() => Number)
  page?: number = 1;

  @IsOptional()
  @IsInt()
  @Min(1)
  @Max(100)
  @Type(() => Number)
  limit?: number = 10;
}
```

**3. `@Body()` - Request Body**

Extracts data from the request body (typically POST, PUT, PATCH requests).

```typescript
@Controller('users')
export class UsersController {
  // POST /users with body: { "name": "John", "email": "john@example.com" }
  @Post()
  create(@Body() createUserDto: CreateUserDto) {
    console.log(createUserDto);  // { name: 'John', email: 'john@example.com' }
    return this.usersService.create(createUserDto);
  }

  // Extract specific field from body
  @Post()
  create2(
    @Body('name') name: string,
    @Body('email') email: string,
  ) {
    return { name, email };
  }

  // PUT /users/123 with body: { "name": "Jane" }
  @Put(':id')
  update(
    @Param('id') id: string,           // From URL
    @Body() updateUserDto: UpdateUserDto,  // From body
  ) {
    return this.usersService.update(id, updateUserDto);
  }
}

// create-user.dto.ts
export class CreateUserDto {
  @IsString()
  @MinLength(2)
  name: string;

  @IsEmail()
  email: string;

  @IsInt()
  @Min(18)
  age: number;
}
```

**Complete Real-World Example:**

```typescript
@Controller('products')
export class ProductsController {
  constructor(private productsService: ProductsService) {}

  // GET /products?category=electronics&minPrice=100&maxPrice=1000&page=1&limit=20
  @Get()
  findAll(
    @Query('category') category?: string,         // Query params
    @Query('minPrice') minPrice?: number,
    @Query('maxPrice') maxPrice?: number,
    @Query('page') page: number = 1,
    @Query('limit') limit: number = 20,
  ) {
    return this.productsService.findAll({
      category,
      minPrice,
      maxPrice,
      page,
      limit,
    });
  }

  // GET /products/123
  @Get(':id')
  findOne(@Param('id') id: string) {             // URL parameter
    return this.productsService.findOne(id);
  }

  // POST /products with body: { "name": "Laptop", "price": 999 }
  @Post()
  create(@Body() createProductDto: CreateProductDto) {  // Request body
    return this.productsService.create(createProductDto);
  }

  // PUT /products/123 with body: { "price": 899 }
  @Put(':id')
  update(
    @Param('id') id: string,                   // URL parameter
    @Body() updateProductDto: UpdateProductDto,  // Request body
  ) {
    return this.productsService.update(id, updateProductDto);
  }

  // GET /products/123/reviews?rating=5&page=1
  @Get(':productId/reviews')
  findReviews(
    @Param('productId') productId: string,      // URL parameter
    @Query('rating') rating?: number,            // Query parameter
    @Query('page') page: number = 1,
  ) {
    return this.productsService.findReviews(productId, { rating, page });
  }
}
```

**When to Use Each:**

**Use `@Param()`:**
- Resource identification (IDs)
- Required path segments
- RESTful resource hierarchy
```typescript
GET /users/:id
GET /users/:userId/posts/:postId
DELETE /products/:id
```

**Use `@Query()`:**
- Optional filtering
- Pagination (page, limit)
- Sorting (sort, order)
- Search terms
- Boolean flags
```typescript
GET /products?category=electronics&page=1&limit=10
GET /users?search=john&status=active&sort=createdAt
GET /orders?startDate=2024-01-01&endDate=2024-12-31
```

**Use `@Body()`:**
- Creating resources (POST)
- Updating resources (PUT/PATCH)
- Complex data structures
- File metadata
```typescript
POST /users { "name": "John", "email": "john@example.com" }
PUT /users/123 { "name": "Jane" }
PATCH /users/123 { "status": "inactive" }
```

**Type Conversion:**

```typescript
// Query and Param come as strings by default
@Get(':id')
findOne(@Param('id') id: string) {  // id is string
  const numericId = +id;  // Convert to number
}

// Use ParseIntPipe for automatic conversion
@Get(':id')
findOne(@Param('id', ParseIntPipe) id: number) {  // id is number
  // id is already a number
}

// Query params
@Get()
findAll(
  @Query('page', ParseIntPipe) page: number,    // Converted to number
  @Query('active', ParseBoolPipe) active: boolean,  // Converted to boolean
) {}
```

**DTO Validation:**

```typescript
// With ValidationPipe, all decorators support DTO validation

// Query DTO
export class FilterDto {
  @IsOptional()
  @IsString()
  category?: string;

  @IsOptional()
  @IsInt()
  @Min(0)
  minPrice?: number;
}

@Get()
findAll(@Query() filters: FilterDto) {  // Automatically validated
  return this.service.findAll(filters);
}

// Body DTO
export class CreateUserDto {
  @IsString()
  @MinLength(2)
  name: string;

  @IsEmail()
  email: string;
}

@Post()
create(@Body() dto: CreateUserDto) {  // Automatically validated
  return this.service.create(dto);
}
```

**All Three Together:**

```typescript
// PATCH /users/123/posts/456?notify=true
// Body: { "title": "Updated Title" }
@Patch(':userId/posts/:postId')
updatePost(
  @Param('userId') userId: string,              // URL parameter
  @Param('postId') postId: string,              // URL parameter
  @Query('notify') notify: boolean = false,      // Query parameter
  @Body() updatePostDto: UpdatePostDto,          // Request body
) {
  return this.postsService.update(
    userId,
    postId,
    updatePostDto,
    notify,
  );
}
```

**Common Mistakes:**

```typescript
// ❌ Wrong: Using @Body() for GET requests
@Get()
findAll(@Body() filters: FilterDto) {  // GET doesn't have body!
  // Use @Query() instead
}

// ✅ Correct
@Get()
findAll(@Query() filters: FilterDto) {
  return this.service.findAll(filters);
}

// ❌ Wrong: Using @Query() for POST data
@Post()
create(@Query() createUserDto: CreateUserDto) {  // Wrong!
  // Use @Body() instead
}

// ✅ Correct
@Post()
create(@Body() createUserDto: CreateUserDto) {
  return this.service.create(createUserDto);
}
```

**Interview Tip**: 
- `@Param()` = URL path (identifying **what** resource)
- `@Query()` = Query string (filtering **which** resources)
- `@Body()` = Request body (sending **data** to create/update)

</details>
### 17. What HTTP method decorators are available (`@Get()`, `@Post()`, etc.)?

<details>
<summary>Answer</summary>

NestJS provides decorators for all standard HTTP methods. Each decorator maps a handler method to a specific HTTP verb.

**Available HTTP Method Decorators:**

```typescript
import {
  Get,
  Post,
  Put,
  Patch,
  Delete,
  Options,
  Head,
  All,
} from '@nestjs/common';
```

**1. `@Get()` - Retrieve Resources**
```typescript
@Controller('users')
export class UsersController {
  @Get()              // GET /users - Get all users
  findAll() {
    return this.usersService.findAll();
  }

  @Get(':id')         // GET /users/:id - Get one user
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(id);
  }
}
```

**2. `@Post()` - Create Resources**
```typescript
@Post()               // POST /users - Create new user
create(@Body() createUserDto: CreateUserDto) {
  return this.usersService.create(createUserDto);
}

@Post('bulk')         // POST /users/bulk - Bulk create
createBulk(@Body() users: CreateUserDto[]) {
  return this.usersService.createBulk(users);
}
```

**3. `@Put()` - Replace/Update Resource (Full Update)**
```typescript
@Put(':id')           // PUT /users/:id - Replace entire user
replace(
  @Param('id') id: string,
  @Body() updateUserDto: UpdateUserDto,
) {
  return this.usersService.replace(id, updateUserDto);
}
```

**4. `@Patch()` - Partial Update**
```typescript
@Patch(':id')         // PATCH /users/:id - Update specific fields
update(
  @Param('id') id: string,
  @Body() updateUserDto: Partial<UpdateUserDto>,
) {
  return this.usersService.update(id, updateUserDto);
}
```

**5. `@Delete()` - Remove Resources**
```typescript
@Delete(':id')        // DELETE /users/:id - Delete user
remove(@Param('id') id: string) {
  return this.usersService.remove(id);
}

@Delete()             // DELETE /users - Delete all (be careful!)
removeAll() {
  return this.usersService.removeAll();
}
```

**6. `@Options()` - Get Communication Options**
```typescript
@Options('*')         // OPTIONS /users/* - CORS preflight
getOptions() {
  return {            // Return allowed methods
    methods: ['GET', 'POST', 'PUT', 'DELETE'],
    headers: ['Content-Type', 'Authorization'],
  };
}
```

**7. `@Head()` - Get Headers Only (No Body)**
```typescript
@Head(':id')          // HEAD /users/:id - Check if exists
checkExists(@Param('id') id: string) {
  // Returns only headers, no body
  const exists = await this.usersService.exists(id);
  if (!exists) {
    throw new NotFoundException();
  }
}
```

**8. `@All()` - Handle All HTTP Methods**
```typescript
@All()                // Matches GET, POST, PUT, DELETE, etc.
handleAll(@Req() req: Request) {
  return `Method: ${req.method}`;
}

@All('*')             // Catch-all route
catchAll() {
  return 'This handles any HTTP method';
}
```

**Complete CRUD Example:**

```typescript
import {
  Controller,
  Get,
  Post,
  Put,
  Patch,
  Delete,
  Body,
  Param,
  HttpCode,
  HttpStatus,
} from '@nestjs/common';

@Controller('products')
export class ProductsController {
  constructor(private productsService: ProductsService) {}

  // CREATE
  @Post()
  @HttpCode(HttpStatus.CREATED)  // 201
  create(@Body() createProductDto: CreateProductDto) {
    return this.productsService.create(createProductDto);
  }

  // READ - All
  @Get()
  @HttpCode(HttpStatus.OK)  // 200 (default)
  findAll() {
    return this.productsService.findAll();
  }

  // READ - One
  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.productsService.findOne(id);
  }

  // UPDATE - Full (PUT)
  @Put(':id')
  replace(
    @Param('id') id: string,
    @Body() replaceProductDto: ReplaceProductDto,
  ) {
    // PUT replaces entire resource
    return this.productsService.replace(id, replaceProductDto);
  }

  // UPDATE - Partial (PATCH)
  @Patch(':id')
  update(
    @Param('id') id: string,
    @Body() updateProductDto: UpdateProductDto,
  ) {
    // PATCH updates only provided fields
    return this.productsService.update(id, updateProductDto);
  }

  // DELETE
  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)  // 204
  remove(@Param('id') id: string) {
    return this.productsService.remove(id);
  }
}
```

**PUT vs PATCH:**

```typescript
// PUT - Replace entire resource (all fields required)
@Put(':id')
replace(@Param('id') id: string, @Body() data: CompleteUserDto) {
  // Must provide ALL fields
  return this.usersService.replace(id, data);
}
// PUT /users/1 { "name": "John", "email": "john@example.com", "age": 30 }
// All fields must be present

// PATCH - Update specific fields (partial)
@Patch(':id')
update(@Param('id') id: string, @Body() data: Partial<UserDto>) {
  // Only update provided fields
  return this.usersService.update(id, data);
}
// PATCH /users/1 { "name": "Jane" }
// Only updates name, other fields unchanged
```

**Custom Route Paths:**

```typescript
@Controller('users')
export class UsersController {
  @Get()                    // GET /users
  @Get('active')            // GET /users/active
  @Get(':id')               // GET /users/:id
  @Get(':id/profile')       // GET /users/:id/profile
  @Post()                   // POST /users
  @Post('register')         // POST /users/register
  @Post('login')            // POST /users/login
  @Patch(':id/activate')    // PATCH /users/:id/activate
  @Delete(':id')            // DELETE /users/:id
}
```

**RESTful Best Practices:**

```typescript
// ✅ Good: RESTful routes
@Get()            // List resources
@Get(':id')       // Get single resource
@Post()           // Create resource
@Put(':id')       // Replace resource
@Patch(':id')     // Update resource
@Delete(':id')    // Delete resource

// ❌ Bad: Non-RESTful routes
@Get('getUsers')        // Don't use verbs
@Post('createUser')     // Don't use verbs
@Get('deleteUser/:id')  // Wrong method for delete
```

**HTTP Status Codes:**

```typescript
@Post()
@HttpCode(HttpStatus.CREATED)  // 201 for creation
create() {}

@Delete(':id')
@HttpCode(HttpStatus.NO_CONTENT)  // 204 for successful deletion
remove() {}

@Patch(':id')
@HttpCode(HttpStatus.OK)  // 200 for successful update (default)
update() {}
```

**Interview Tip**: Match HTTP methods to their semantic meaning - GET for reading, POST for creating, PUT/PATCH for updating, DELETE for removing. Use PATCH for partial updates and PUT for complete replacements.

</details>

### 18. Why should you avoid using `@Res()` directly?

<details>
<summary>Answer</summary>

**Problem with `@Res()`:** When you inject the response object using `@Res()` or `@Response()`, you lose NestJS's built-in response handling features and must manually send the response.

**Why Avoid It:**

**1. Breaks NestJS Interceptors**
```typescript
// ❌ Bad: @Res() bypasses interceptors
@Get()
findAll(@Res() res: Response) {
  const users = this.usersService.findAll();
  res.json(users);  // Interceptors won't run!
}

// ✅ Good: Return value, interceptors work
@Get()
findAll() {
  return this.usersService.findAll();  // Interceptors run automatically
}
```

**2. Breaks Automatic Response Transformation**
```typescript
// ❌ Bad: Manual serialization needed
@Get(':id')
findOne(@Param('id') id: string, @Res() res: Response) {
  const user = this.usersService.findOne(id);
  // Must manually exclude sensitive data
  const { password, ...safeUser } = user;
  res.json(safeUser);
}

// ✅ Good: Automatic serialization with @Exclude()
@Get(':id')
findOne(@Param('id') id: string): Promise<User> {
  return this.usersService.findOne(id);
  // ClassSerializerInterceptor automatically excludes @Exclude() fields
}

export class User {
  id: string;
  email: string;
  
  @Exclude()  // Automatically removed from response
  password: string;
}
```

**3. Breaks Exception Filters**
```typescript
// ❌ Bad: Exception filters don't catch errors
@Get(':id')
async findOne(@Param('id') id: string, @Res() res: Response) {
  try {
    const user = await this.usersService.findOne(id);
    res.json(user);
  } catch (error) {
    res.status(500).json({ message: 'Error' });  // Manual error handling
  }
}

// ✅ Good: Exception filters handle errors automatically
@Get(':id')
findOne(@Param('id') id: string): Promise<User> {
  return this.usersService.findOne(id);
  // Throws NotFoundException if not found
  // Global exception filter handles it automatically
}
```

**4. Manual Status Code Management**
```typescript
// ❌ Bad: Manual status codes
@Post()
create(@Body() dto: CreateUserDto, @Res() res: Response) {
  const user = this.usersService.create(dto);
  res.status(201).json(user);  // Must manually set 201
}

// ✅ Good: Declarative status codes
@Post()
@HttpCode(HttpStatus.CREATED)  // Declarative
create(@Body() dto: CreateUserDto) {
  return this.usersService.create(dto);
}
```

**5. Testing Becomes Harder**
```typescript
// ❌ Bad: Hard to test
@Get()
findAll(@Res() res: Response) {
  res.json(this.usersService.findAll());  // Must mock res.json()
}

// Test
it('should return users', async () => {
  const mockResponse = {
    json: jest.fn(),  // Must mock response methods
  };
  await controller.findAll(mockResponse as any);
  expect(mockResponse.json).toHaveBeenCalled();
});

// ✅ Good: Easy to test
@Get()
findAll() {
  return this.usersService.findAll();  // Just return data
}

// Test
it('should return users', async () => {
  const result = await controller.findAll();  // Simple!
  expect(result).toEqual([...]);
});
```

**When It's Okay to Use `@Res()`:**

**1. File Downloads/Streaming**
```typescript
@Get(':id/download')
async downloadFile(
  @Param('id') id: string,
  @Res() res: Response,  // Needed for streaming
) {
  const file = await this.filesService.getFile(id);
  res.set({
    'Content-Type': file.mimetype,
    'Content-Disposition': `attachment; filename="${file.filename}"`,
  });
  file.stream.pipe(res);  // Stream file to response
}
```

**2. Custom Headers**
```typescript
@Get()
findAll(@Res({ passthrough: true }) res: Response) {
  res.set('X-Custom-Header', 'value');  // Set custom header
  return this.usersService.findAll();   // Still return value!
}
// Note: passthrough: true keeps NestJS handling
```

**3. Redirects**
```typescript
// ❌ Not ideal, but works
@Get('old-route')
redirect(@Res() res: Response) {
  res.redirect('/new-route');
}

// ✅ Better: Use @Redirect decorator
@Get('old-route')
@Redirect('/new-route', 301)
redirect() {}
```

**4. Server-Sent Events (SSE)**
```typescript
@Sse('events')
events(@Res() res: Response) {  // SSE requires response object
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');
  
  const interval = setInterval(() => {
    res.write(`data: ${JSON.stringify({ time: new Date() })}\n\n`);
  }, 1000);
  
  res.on('close', () => clearInterval(interval));
}
```

**Using `passthrough: true`:**

```typescript
// Allows using @Res() while keeping NestJS features
@Get()
findAll(@Res({ passthrough: true }) res: Response) {
  res.set('X-Total-Count', '100');  // Custom header
  res.cookie('session', 'abc123');  // Set cookie
  
  return this.usersService.findAll();  // Still return value!
  // Interceptors and serialization still work!
}
```

**Best Practices:**

```typescript
// ✅ Prefer this (standard approach)
@Get()
findAll() {
  return this.usersService.findAll();
}

// ✅ Use decorators for status codes
@Post()
@HttpCode(HttpStatus.CREATED)
create(@Body() dto: CreateUserDto) {
  return this.usersService.create(dto);
}

// ✅ Use decorators for headers
@Get()
@Header('Cache-Control', 'max-age=3600')
findAll() {
  return this.usersService.findAll();
}

// ✅ Use passthrough if you must access response
@Get()
findAll(@Res({ passthrough: true }) res: Response) {
  res.set('X-Custom', 'value');
  return this.usersService.findAll();
}

// ❌ Avoid this (bypasses NestJS features)
@Get()
findAll(@Res() res: Response) {
  res.json(this.usersService.findAll());
}
```

**What You Lose Without Passthrough:**

```
❌ Interceptors don't run
❌ Exception filters don't work  
❌ @UseInterceptors() ignored
❌ ClassSerializerInterceptor ignored
❌ Automatic error handling disabled
❌ Response transformation disabled
❌ Must manually handle everything
```

**Interview Tip**: Avoid `@Res()` unless you have a specific reason (file streaming, SSE). If you must use it, add `{ passthrough: true }` to keep NestJS features. Let NestJS handle responses by returning values from your handlers.

</details>

### 19. How do you return different HTTP status codes using `@HttpCode()`?

<details>
<summary>Answer</summary>

The `@HttpCode()` decorator explicitly sets the HTTP status code for a route handler, overriding NestJS's default status codes.

**Default Status Codes:**
- `@Get()`, `@Put()`, `@Patch()`, `@Delete()` → **200 OK**
- `@Post()` → **201 Created**

**Basic Usage:**

```typescript
import { Post, HttpCode, HttpStatus, Delete } from '@nestjs/common';

@Controller('users')
export class UsersController {
  // POST defaults to 201 Created
  @Post()
  create(@Body() dto: CreateUserDto) {
    return this.usersService.create(dto);
  }

  // Override to 200 OK for non-creating POST
  @Post('login')
  @HttpCode(HttpStatus.OK)  // 200 instead of 201
  login(@Body() credentials: LoginDto) {
    return this.authService.login(credentials);
  }

  // DELETE with no content
  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)  // 204
  remove(@Param('id') id: string): Promise<void> {
    return this.usersService.remove(id);
  }
}
```

**Common Status Codes:**

```typescript
// 2xx Success
@Post()
@HttpCode(HttpStatus.OK)         // 200
@HttpCode(HttpStatus.CREATED)    // 201 (POST default)
@HttpCode(HttpStatus.ACCEPTED)   // 202 - Async processing
@HttpCode(HttpStatus.NO_CONTENT) // 204 - Success, no body

// Real examples
@Post('search')
@HttpCode(HttpStatus.OK)  // Search doesn't create
search(@Body() dto: SearchDto) {}

@Post('bulk')
@HttpCode(HttpStatus.ACCEPTED)  // Queued for processing
bulkProcess(@Body() data: any[]) {
  this.queue.add('bulk-job', data);
  return { status: 'queued' };
}

@Delete(':id')
@HttpCode(HttpStatus.NO_CONTENT)  // No response body
remove(@Param('id') id: string): void {}
```

**Dynamic Status Codes:**

```typescript
// Use @Res() with passthrough for dynamic codes
@Post()
create(
  @Body() dto: CreateUserDto,
  @Res({ passthrough: true }) res: Response,
) {
  const result = this.usersService.create(dto);
  
  if (result.isNew) {
    res.status(HttpStatus.CREATED);  // 201
  } else {
    res.status(HttpStatus.OK);       // 200
  }
  
  return result;
}
```

**Real-World Examples:**

```typescript
// Authentication
@Post('login')
@HttpCode(HttpStatus.OK)  // 200, not creating
login() {}

@Post('logout')
@HttpCode(HttpStatus.NO_CONTENT)  // 204, no return
logout() {}

// Async operations
@Post('reports/generate')
@HttpCode(HttpStatus.ACCEPTED)  // 202
generateReport() {
  // Queue job, return jobId
}

// Bulk operations
@Delete('bulk')
@HttpCode(HttpStatus.NO_CONTENT)  // 204
bulkDelete(@Body() ids: string[]) {}
```

**Interview Tip**: `@HttpCode()` overrides NestJS defaults. Use 200 for POST that doesn't create (login/search), 204 for deletes with no body, and 202 for async operations.

</details>

## Providers & Dependency Injection

### 20. What is Dependency Injection and why is it important?

<details>
<summary>Answer</summary>

**Dependency Injection (DI)** is a design pattern where a class receives its dependencies from external sources rather than creating them itself. NestJS has a built-in IoC (Inversion of Control) container that manages DI automatically.

**Without DI:**
```typescript
// ❌ Bad: Class creates dependencies (tight coupling)
export class UsersController {
  private usersService: UsersService;
  
  constructor() {
    this.usersService = new UsersService();  // Hard-coded
  }
}
// Problems: Can't test, can't swap implementations
```

**With DI:**
```typescript
// ✅ Good: Dependencies injected
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}
  // NestJS injects UsersService automatically
}
```

**Benefits:**
1. **Loose Coupling** - Easy to swap implementations
2. **Testability** - Can inject mocks for testing
3. **Lifecycle Management** - NestJS manages creation/destruction
4. **Reusability** - Share instances across modules

**Complete Example:**

```typescript
// Repository
@Injectable()
export class UsersRepository {
  findAll() { return ['users']; }
}

// Service with injected dependencies
@Injectable()
**With DI:**
```typescript
// ✅ Good: Dependencies injected
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}
  // NestJS injects UsersService automatically
}
```

**Benefits:**
1. **Loose Coupling** - Easy to swap implementations
2. **Testability** - Can inject mocks for testing
3. **Lifecycle Management** - NestJS manages creation/destruction
4. **Reusability** - Share instances across modules

**Complete Example:**

```typescript
// Repository
@Injectable()
export class UsersRepository {
  findAll() { return ['users']; }
}

// Service with injected dependencies
@Injectable()
export class UsersService {
  constructor(
    private readonly repository: UsersRepository,
    private readonly emailService: EmailService,
  ) {}
  
  async create(data: any) {
    const user = await this.repository.create(data);
    await this.emailService.sendWelcome(user.email);
    return user;
  }
}

// Module - register providers
@Module({
  providers: [UsersRepository, UsersService, EmailService],
  controllers: [UsersController],
})
export class UsersModule {}
```

**Interview Tip**: DI is a pattern where classes receive dependencies from external sources (NestJS IoC container) instead of creating them. It enables loose coupling, easy testing, and automatic lifecycle management.

</details>

### 21. What is the `@Injectable()` decorator?

<details>
<summary>Answer</summary>

`@Injectable()` marks a class as a **provider** that can be managed by NestJS's DI system.

**Basic Usage:**
```typescript
@Injectable()
export class UsersService {
  findAll() {
    return ['users'];
  }
}

// Now injectable anywhere
@Controller('users')
export class UsersController {
  constructor(private usersService: UsersService) {}
}
```

**What Needs `@Injectable()`:**
- ✅ Services
- ✅ Repositories  
- ✅ Guards, Interceptors, Pipes, Filters
- ❌ Controllers (use `@Controller`)
- ❌ DTOs (plain classes)
- ❌ Entities (use `@Entity`)

**With Scopes:**
```typescript
@Injectable({ scope: Scope.REQUEST })  // New per request
export class RequestService {}

@Injectable({ scope: Scope.TRANSIENT })  // New per injection
export class TransientService {}
```

**Interview Tip**: `@Injectable()` marks a class as a provider for DI. Use it on services, repositories, guards, interceptors, pipes, and filters - but NOT on controllers, DTOs, or entities.

</details>

### 22. What is Constructor-based Injection?

<details>
<summary>Answer</summary>

**Constructor-based injection** is the primary way to inject dependencies in NestJS - dependencies are provided as constructor parameters.

**Basic Example:**
```typescript
@Controller('users')
export class UsersController {
  // TypeScript shorthand: private readonly declares property
  constructor(private readonly usersService: UsersService) {}
  
  @Get()
  findAll() {
    return this.usersService.findAll();
  }
}
```

**Multiple Dependencies:**
```typescript
@Injectable()
export class UsersService {
  constructor(
    private readonly repository: UsersRepository,
    private readonly emailService: EmailService,
    private readonly logger: LoggerService,
  ) {}
}
```

**Custom Tokens:**
```typescript
@Injectable()
export class UsersService {
  constructor(
    @Inject('DATABASE_CONNECTION') private connection: Connection,
    @Inject('CONFIG') private config: any,
  ) {}
}
```

**Optional Dependencies:**
```typescript
@Injectable()
export class UsersService {
  constructor(
    private readonly repository: UsersRepository,  // Required
    @Optional() private cacheService?: CacheService,  // Optional
  ) {}
}
```

**Interview Tip**: Constructor-based injection is NestJS's primary DI mechanism. Dependencies are declared as constructor parameters with `private readonly`. NestJS reads TypeScript metadata to determine what to inject.

</details>

### 23. What are Provider Scopes (DEFAULT, REQUEST, TRANSIENT)?

<details>
<summary>Answer</summary>

Provider scopes control the lifetime and sharing of provider instances.

**DEFAULT Scope (Singleton):**
```typescript
@Injectable()  // Default scope
export class UsersService {}
// One instance shared across entire application
```

**REQUEST Scope:**
```typescript
@Injectable({ scope: Scope.REQUEST })
export class RequestService {}
// New instance per HTTP request
// Use for request-specific data
```

**TRANSIENT Scope:**
```typescript
@Injectable({ scope: Scope.TRANSIENT })
export class TransientService {}
// New instance every time it's injected
```

**Comparison:**
| Scope | Lifetime | Use Case |
|-------|----------|----------|
| DEFAULT | App lifetime | Stateless services, shared logic |
| REQUEST | Per request | Request-specific context, tracking |
| TRANSIENT | Per injection | Unique instances, no sharing |

**REQUEST Scope Example:**
```typescript
@Injectable({ scope: Scope.REQUEST })
export class RequestContext {
  userId: string;
  requestId: string;
}

@Injectable({ scope: Scope.REQUEST })
export class UsersService {
  constructor(private context: RequestContext) {}
  
  findAll() {
    console.log(`Request by user: ${this.context.userId}`);
  }
}
```

**Interview Tip**: DEFAULT (singleton) for stateless services, REQUEST for request-specific data, TRANSIENT for unique instances. REQUEST scope has performance implications - use sparingly.

</details>

### 24. What is the difference between DEFAULT and REQUEST scope?

<details>
<summary>Answer</summary>

**DEFAULT Scope (Singleton):**
- ✅ One instance for entire app
- ✅ Best performance
- ✅ Shared state across all requests
- ❌ Can't store request-specific data

**REQUEST Scope:**
- ✅ New instance per request
- ✅ Can store request-specific data
- ❌ Performance overhead (creates new instances)
- ❌ Entire dependency tree becomes REQUEST-scoped

**Performance Impact:**
```typescript
// DEFAULT - Created once
@Injectable()
export class UsersService {}
// App startup: 1 instance created
// 1000 requests: Same instance reused

// REQUEST - Created per request  
@Injectable({ scope: Scope.REQUEST })
export class RequestService {}
// App startup: 0 instances
// 1000 requests: 1000 instances created
```

**When to Use REQUEST:**
```typescript
// ✅ Good: Request-specific tracking
@Injectable({ scope: Scope.REQUEST })
export class RequestLogger {
  requestId: string;
  userId: string;
  startTime: number;
}

// ❌ Bad: Stateless service doesn't need REQUEST scope
@Injectable({ scope: Scope.REQUEST })  // Unnecessary!
export class MathService {
  add(a: number, b: number) {
    return a + b;  // No request-specific state
  }
}
```

**Interview Tip**: Use DEFAULT (singleton) for stateless services (best performance). Use REQUEST only when you need request-specific data (logger, context, tracking). REQUEST scope creates new instances per request, impacting performance.

</details>

### 25. What is a Custom Provider?

<details>
<summary>Answer</summary>

A **Custom Provider** allows you to control how NestJS creates and provides dependencies beyond standard class providers.

**Types:**
1. **useClass** - Provide alternate implementation
2. **useValue** - Provide static value  
3. **useFactory** - Provide via factory function
4. **useExisting** - Create alias

**1. useClass:**
```typescript
@Module({
  providers: [
    {
      provide: 'LoggerService',
      useClass: process.env.NODE_ENV === 'production'
        ? ProductionLogger
        : DevLogger,
    },
  ],
})
```

**2. useValue:**
```typescript
@Module({
  providers: [
    {
      provide: 'CONFIG',
      useValue: {
        apiKey: process.env.API_KEY,
        apiUrl: process.env.API_URL,
      },
    },
  ],
})
```

**3. useFactory:**
```typescript
@Module({
  providers: [
    {
      provide: 'DATABASE_CONNECTION',
      useFactory: async (config: ConfigService) => {
        return await createConnection(config.get('DB_URL'));
      },
      inject: [ConfigService],
    },
  ],
})
```

**4. useExisting:**
```typescript
@Module({
  providers: [
    LoggerService,
    {
      provide: 'ILogger',
      useExisting: LoggerService,  // Alias
    },
  ],
})
```

**Interview Tip**: Custom providers let you control dependency creation. Use `useClass` for conditional implementations, `useValue` for static config, `useFactory` for async initialization, and `useExisting` for aliases.

</details>

### 26. What is the difference between `useClass`, `useValue`, and `useFactory`?

<details>
<summary>Answer</summary>

These are different ways to provide custom implementations to the DI container.

**Comparison:**
| Type | When Created | Use Case | Async Support |
|------|-------------|----------|---------------|
| useClass | On injection | Swap implementations | No |
| useValue | Immediately | Static values, mocks | No |
| useFactory | On injection | Dynamic/async creation | Yes |

**useClass - Swap Implementations:**
```typescript
// Provide different class based on environment
{
  provide: EmailService,
  useClass: process.env.NODE_ENV === 'test'
    ? MockEmailService
    : SendGridEmailService,
}
```

**useValue - Static Values:**
```typescript
// Provide configuration object
{
  provide: 'CONFIG',
  useValue: {
    apiKey: 'abc123',
    timeout: 5000,
  },
}

// Provide mock for testing
{
  provide: UsersService,
  useValue: {
    findAll: jest.fn().mockResolvedValue([]),
  },
}
```

**useFactory - Dynamic Creation:**
```typescript
// Async initialization
{
  provide: 'DATABASE_CONNECTION',
  useFactory: async (config: ConfigService) => {
    const connection = await createConnection({
      host: config.get('DB_HOST'),
      port: config.get('DB_PORT'),
    });
    return connection;
  },
  inject: [ConfigService],  // Dependencies
}

// Conditional logic
{
  provide: 'CACHE_SERVICE',
  useFactory: (config: ConfigService) => {
    return config.get('REDIS_URL')
      ? new RedisCache()
      : new MemoryCache();
  },
  inject: [ConfigService],
}
```

**Interview Tip**: `useClass` for alternate implementations, `useValue` for static config/mocks, `useFactory` for async initialization or dynamic creation with dependencies.

</details>

### 27. When would you use `useFactory` provider?

<details>
<summary>Answer</summary>

Use `useFactory` when you need **dynamic provider creation**, **async initialization**, or **complex logic** during instantiation.

**Use Cases:**

**1. Async Initialization:**
```typescript
{
  provide: 'DATABASE_CONNECTION',
  useFactory: async () => {
    const connection = await createConnection();
    await connection.connect();
    return connection;
  },
}
```

**2. With Injected Dependencies:**
```typescript
{
  provide: 'CACHE_SERVICE',
  useFactory: (config: ConfigService, logger: Logger) => {
    const cacheConfig = {
      host: config.get('REDIS_HOST'),
      port: config.get('REDIS_PORT'),
    };
    logger.log('Initializing cache');
    return new RedisClient(cacheConfig);
  },
  inject: [ConfigService, Logger],
}
```

**3. Conditional Logic:**
```typescript
{
  provide: 'PAYMENT_SERVICE',
  useFactory: (config: ConfigService) => {
    const provider = config.get('PAYMENT_PROVIDER');
    switch (provider) {
      case 'stripe':
        return new StripeService(config.get('STRIPE_KEY'));
      case 'paypal':
        return new PayPalService(config.get('PAYPAL_KEY'));
      default:
        return new MockPaymentService();
    }
  },
  inject: [ConfigService],
}
```

**4. External Library Setup:**
```typescript
{
  provide: 'REDIS_CLIENT',
  useFactory: async (config: ConfigService) => {
    const client = createClient({
      url: config.get('REDIS_URL'),
    });
    await client.connect();
    return client;
  },
  inject: [ConfigService],
}
```

**Interview Tip**: Use `useFactory` for async initialization (database connections), dynamic creation based on config, or when provider creation requires injected dependencies and complex logic.

</details>

### 28. How do you handle circular dependencies using `forwardRef()`?

<details>
<summary>Answer</summary>

**Circular Dependency Problem:**
```typescript
// ❌ Error: A needs B, B needs A
@Injectable()
export class UsersService {
  constructor(private postsService: PostsService) {}
}

@Injectable()
export class PostsService {
  constructor(private usersService: UsersService) {}
}
// Error: Circular dependency!
```

**Solution - forwardRef():**
```typescript
import { Injectable, Inject, forwardRef } from '@nestjs/common';

@Injectable()
export class UsersService {
  constructor(
    @Inject(forwardRef(() => PostsService))
    private postsService: PostsService,
  ) {}
}

@Injectable()
export class PostsService {
  constructor(
    @Inject(forwardRef(() => UsersService))
    private usersService: UsersService,
  ) {}
}
```

**In Modules:**
```typescript
@Module({
  imports: [forwardRef(() => PostsModule)],
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}

@Module({
  imports: [forwardRef(() => UsersModule)],
  providers: [PostsService],
  exports: [PostsService],
})
export class PostsModule {}
```

**Better Solutions (Avoid Circular Dependencies):**

**1. Extract Shared Logic:**
```typescript
// Create shared service
@Injectable()
export class SharedService {
  // Common logic used by both
}

@Injectable()
export class UsersService {
  constructor(private shared: SharedService) {}
}

@Injectable()
export class PostsService {
  constructor(private shared: SharedService) {}
}
```

**2. Event-Driven:**
```typescript
@Injectable()
export class UsersService {
  constructor(private eventEmitter: EventEmitter2) {}
  
  create(data: any) {
    const user = this.repository.create(data);
    this.eventEmitter.emit('user.created', user);
    return user;
  }
}

@Injectable()
export class PostsService {
  @OnEvent('user.created')
  handleUserCreated(user: User) {
    // React to user creation
  }
}
```

**Interview Tip**: Use `forwardRef(() => Service)` for circular dependencies, but it's better to refactor - extract shared logic to a separate service or use event-driven communication to decouple services.

</details>

## Services

### 29. What is a Service in NestJS?

<details>
<summary>Answer</summary>

A **Service** is a class marked with `@Injectable()` that contains business logic, separate from controllers and data access layers.

**Basic Service:**
```typescript
@Injectable()
export class UsersService {
  private users = [];
  
  findAll() {
    return this.users;
  }
  
  findOne(id: string) {
    return this.users.find(u => u.id === id);
  }
  
  create(data: CreateUserDto) {
    const user = { id: Date.now().toString(), ...data };
    this.users.push(user);
    return user;
  }
}
```

**Three-Layer Architecture:**
```typescript
// 1. Controller - HTTP layer
@Controller('users')
export class UsersController {
  constructor(private usersService: UsersService) {}
  
  @Post()
  create(@Body() dto: CreateUserDto) {
    return this.usersService.create(dto);
  }
}

// 2. Service - Business logic
@Injectable()
export class UsersService {
  constructor(
    private repository: UsersRepository,
    private emailService: EmailService,
  ) {}
  
  async create(data: CreateUserDto) {
    // Business logic
    const user = await this.repository.create(data);
    await this.emailService.sendWelcome(user.email);
    return user;
  }
}

// 3. Repository - Data access
@Injectable()
export class UsersRepository {
  constructor(@InjectRepository(User) private repo: Repository<User>) {}
  
  create(data: Partial<User>) {
    return this.repo.save(data);
  }
}
```

**Interview Tip**: Services contain business logic, separate from HTTP concerns (controllers) and data access (repositories). They're injectable classes that implement the application's core functionality.

</details>

### 30. What is the difference between a Service and a Provider?

<details>
<summary>Answer</summary>

**Provider** is the generic term; **Service** is a specific type of provider.

**Provider (Generic Term):**
Any class that can be injected:
- Services (business logic)
- Repositories (data access)
- Factories, Helpers, Utils
- Guards, Interceptors, Pipes

**Service (Specific Type):**
A provider that contains business logic.

```typescript
// All are providers, but different types:

// Service - business logic
@Injectable()
export class UsersService {
  createUser() { /* business logic */ }
}

// Repository - data access
@Injectable()
export class UsersRepository {
  save() { /* database operations */ }
}

// Guard - authorization
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate() { /* auth logic */ }
}

// All registered as providers:
@Module({
  providers: [UsersService, UsersRepository, AuthGuard],
})
```

**Key Points:**
- Every Service is a Provider
- Not every Provider is a Service
- "Provider" = anything injectable
- "Service" = business logic layer

**Interview Tip**: Provider is the generic term for any injectable class in NestJS. Service is a specific type of provider that contains business logic, distinguished from repositories (data), guards (auth), etc.

</details>

### 31. How do you share a service across multiple modules?

<details>
<summary>Answer</summary>

Use the `exports` array in the module to make a service available to other modules.

**Providing Module:**
```typescript
@Module({
  providers: [UsersService],
  exports: [UsersService],  // Export for other modules
})
export class UsersModule {}
```

**Consuming Module:**
```typescript
@Module({
  imports: [UsersModule],  // Import the module
  controllers: [OrdersController],
})
export class OrdersModule {}

// Now OrdersController can inject UsersService
@Controller('orders')
export class OrdersController {
  constructor(private usersService: UsersService) {}
}
```

**Global Module:**
```typescript
@Global()  // Available everywhere
@Module({
  providers: [LoggerService, ConfigService],
  exports: [LoggerService, ConfigService],
})
export class CoreModule {}

// Now any module can inject without importing
@Module({
  // No need to import CoreModule
  controllers: [UsersController],
})
export class UsersModule {}
```

**Module Re-Export:**
```typescript
@Module({
  imports: [UsersModule, PostsModule],
  exports: [UsersModule, PostsModule],  // Re-export
})
export class AppModule {}
```

**Interview Tip**: Export services from their module using the `exports` array, then import that module where needed. Use `@Global()` for truly global services like logging or configuration.

</details>

## Application Lifecycle

### 32. What are the main Lifecycle Hooks in NestJS?

<details>
<summary>Answer</summary>

NestJS provides lifecycle hooks to execute code at specific points during application startup and shutdown.

**Main Hooks:**
1. `onModuleInit()` - After module dependencies resolved
2. `onApplicationBootstrap()` - After all modules initialized
3. `onModuleDestroy()` - Before module destroyed
4. `beforeApplicationShutdown()` - Before app shutdown starts
5. `onApplicationShutdown()` - During shutdown

**Example:**
```typescript
import {
  Injectable,
  OnModuleInit,
  OnModuleDestroy,
  OnApplicationBootstrap,
  OnApplicationShutdown,
  BeforeApplicationShutdown,
} from '@nestjs/common';

@Injectable()
export class DatabaseService
  implements
    OnModuleInit,
    OnApplicationBootstrap,
    OnModuleDestroy,
    BeforeApplicationShutdown,
    OnApplicationShutdown
{
  private connection;

  async onModuleInit() {
    console.log('Module initialized');
    this.connection = await createConnection();
  }

  onApplicationBootstrap() {
    console.log('App fully bootstrapped');
  }

  beforeApplicationShutdown(signal?: string) {
    console.log(`Before shutdown: ${signal}`);
  }

  async onModuleDestroy() {
    console.log('Module being destroyed');
    await this.connection.close();
  }

  onApplicationShutdown(signal?: string) {
    console.log(`App shutdown: ${signal}`);
  }
}
```

**Execution Order:**
```
Startup:
1. onModuleInit
2. onApplicationBootstrap

Shutdown:
1. beforeApplicationShutdown
2. onModuleDestroy
3. onApplicationShutdown
```

**Interview Tip**: Lifecycle hooks let you execute code during app startup (`onModuleInit`, `onApplicationBootstrap`) and shutdown (`onModuleDestroy`, `onApplicationShutdown`). Use them for connections, cleanup, and graceful shutdown.

</details>

### 33. What is `OnModuleInit` and when is it called?

<details>
<summary>Answer</summary>

`OnModuleInit` is a lifecycle hook called **after all module dependencies are resolved** but before the application starts listening.

**Usage:**
```typescript
import { Injectable, OnModuleInit } from '@nestjs/common';

@Injectable()
export class DatabaseService implements OnModuleInit {
  private connection;

  async onModuleInit() {
    // Called after dependencies injected
    this.connection = await createConnection();
    console.log('Database connected');
  }
}
```

**Common Use Cases:**

**1. Initialize Connections:**
```typescript
@Injectable()
export class RedisService implements OnModuleInit {
  private client: Redis;

  async onModuleInit() {
    this.client = createClient();
    await this.client.connect();
  }
}
```

**2. Load Data:**
```typescript
@Injectable()
export class CacheService implements OnModuleInit {
  async onModuleInit() {
    await this.loadCacheFromDatabase();
  }
}
```

**3. Register Event Listeners:**
```typescript
@Injectable()
export class EventsService implements OnModuleInit {
  onModuleInit() {
    this.eventEmitter.on('user.created', this.handleUserCreated);
  }
}
```

**Interview Tip**: `OnModuleInit` is called after module dependencies are resolved. Use it for async initialization like establishing connections, loading data, or registering listeners before the app starts.

</details>

### 34. What is the order of lifecycle hooks execution?

<details>
<summary>Answer</summary>

**Startup Order:**
```
1. Constructor called
2. Dependencies injected
3. onModuleInit() - Module dependencies resolved
4. onApplicationBootstrap() - All modules initialized
5. Application starts listening
```

**Shutdown Order:**
```
1. Shutdown signal received (SIGTERM/SIGINT)
2. beforeApplicationShutdown(signal) - Before shutdown
3. onModuleDestroy() - Cleanup
4. onApplicationShutdown(signal) - Final cleanup
5. Process exits
```

**Complete Example:**
```typescript
@Injectable()
export class LifecycleService
  implements
    OnModuleInit,
    OnApplicationBootstrap,
    BeforeApplicationShutdown,
    OnModuleDestroy,
    OnApplicationShutdown
{
  constructor() {
    console.log('1. Constructor');
  }

  onModuleInit() {
    console.log('2. onModuleInit');
  }

  onApplicationBootstrap() {
    console.log('3. onApplicationBootstrap');
  }

  beforeApplicationShutdown(signal?: string) {
    console.log(`4. beforeApplicationShutdown: ${signal}`);
  }

  onModuleDestroy() {
    console.log('5. onModuleDestroy');
  }

  onApplicationShutdown(signal?: string) {
    console.log(`6. onApplicationShutdown: ${signal}`);
  }
}

// Output on startup:
// 1. Constructor
// 2. onModuleInit
// 3. onApplicationBootstrap

// Output on SIGTERM:
// 4. beforeApplicationShutdown: SIGTERM
// 5. onModuleDestroy
// 6. onApplicationShutdown: SIGTERM
```

**Interview Tip**: Startup: Constructor → onModuleInit → onApplicationBootstrap. Shutdown: beforeApplicationShutdown → onModuleDestroy → onApplicationShutdown.

</details>

### 35. How do you enable graceful shutdown?

<details>
<summary>Answer</summary>

Enable graceful shutdown to properly clean up resources when the app receives termination signals.

**Enable in main.ts:**
```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Enable shutdown hooks
  app.enableShutdownHooks();
  
  await app.listen(3000);
}
```

**Implement Cleanup:**
```typescript
@Injectable()
export class DatabaseService implements OnApplicationShutdown {
  private connection;

  async onApplicationShutdown(signal?: string) {
    console.log(`Received ${signal}, closing database`);
    await this.connection.close();
  }
}

@Injectable()
export class QueueService implements OnModuleDestroy {
  private queue;

  async onModuleDestroy() {
    console.log('Closing queue connections');
    await this.queue.close();
  }
}
```

**Handle Signals:**
```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.enableShutdownHooks();

  // Custom signal handling
  process.on('SIGTERM', async () => {
    console.log('SIGTERM received');
    await app.close();
  });

  process.on('SIGINT', async () => {
    console.log('SIGINT received');
    await app.close();
  });

  await app.listen(3000);
}
```

**Complete Example:**
```typescript
// main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.enableShutdownHooks();

  const gracefulShutdown = async (signal: string) => {
    console.log(`\\n${signal} received, starting graceful shutdown`);
    await app.close();
    console.log('App closed gracefully');
    process.exit(0);
  };

  process.on('SIGTERM', () => gracefulShutdown('SIGTERM'));
  process.on('SIGINT', () => gracefulShutdown('SIGINT'));

  await app.listen(3000);
}

// Services with cleanup
@Injectable()
export class AppService
  implements OnModuleDestroy, OnApplicationShutdown
{
  async onModuleDestroy() {
    // Close connections, save state
  }

  async onApplicationShutdown(signal?: string) {
    console.log(`Shutdown complete: ${signal}`);
  }
}
```

**Interview Tip**: Call `app.enableShutdownHooks()` to enable graceful shutdown. Implement `onModuleDestroy` and `onApplicationShutdown` hooks to clean up resources like database connections, queues, and file handles.

</details>

## Application Bootstrap

### 36. What is `NestFactory` and what does `NestFactory.create()` do?

<details>
<summary>Answer</summary>

`NestFactory` is the entry point for creating and configuring a NestJS application instance.

**Basic Usage:**
```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

**What `create()` Does:**
1. Creates application instance
2. Resolves all modules and dependencies
3. Initializes DI container
4. Creates HTTP adapter (Express/Fastify)
5. Returns INestApplication instance

**With Options:**
```typescript
const app = await NestFactory.create(AppModule, {
  logger: ['error', 'warn'],  // Log levels
  cors: true,  // Enable CORS
  bodyParser: true,  // Parse request bodies
  httpsOptions: {  // HTTPS config
    key: fs.readFileSync('key.pem'),
    cert: fs.readFileSync('cert.pem'),
  },
});
```

**With Fastify:**
```typescript
import { FastifyAdapter } from '@nestjs/platform-fastify';

const app = await NestFactory.create(
  AppModule,
  new FastifyAdapter(),
);
```

**Create Microservice:**
```typescript
const app = await NestFactory.createMicroservice(AppModule, {
  transport: Transport.TCP,
  options: { port: 3001 },
});
```

**Interview Tip**: `NestFactory.create(AppModule)` creates the app instance, resolves dependencies, and initializes the DI container. It returns an INestApplication that you configure and start with `listen()`.

</details>

### 37. How do you enable CORS in NestJS?

<details>
<summary>Answer</summary>

Enable CORS (Cross-Origin Resource Sharing) to allow requests from different origins.

**Method 1: In create():**
```typescript
const app = await NestFactory.create(AppModule, {
  cors: true,  // Enable with defaults
});
```

**Method 2: Using enableCors():**
```typescript
const app = await NestFactory.create(AppModule);
app.enableCors();  // Enable with defaults
await app.listen(3000);
```

**With Options:**
```typescript
app.enableCors({
  origin: 'https://example.com',  // Specific origin
  methods: 'GET,HEAD,PUT,PATCH,POST,DELETE',
  credentials: true,  // Allow cookies
  allowedHeaders: ['Content-Type', 'Authorization'],
});
```

**Multiple Origins:**
```typescript
app.enableCors({
  origin: ['https://example.com', 'https://app.example.com'],
  credentials: true,
});
```

**Dynamic Origin:**
```typescript
app.enableCors({
  origin: (origin, callback) => {
    const whitelist = ['https://example.com', 'https://app.example.com'];
    if (whitelist.includes(origin) || !origin) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
});
```

**Production Example:**
```typescript
const app = await NestFactory.create(AppModule);

app.enableCors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') || '*',
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  credentials: true,
  allowedHeaders: ['Content-Type', 'Authorization', 'X-Requested-With'],
  exposedHeaders: ['X-Total-Count'],
  maxAge: 3600,
});
```

**Interview Tip**: Enable CORS with `app.enableCors()` or in `NestFactory.create()` options. Configure `origin`, `methods`, and `credentials` for security. Use dynamic origin checking for multiple allowed domains.

</details>

### 38. How do you set a global prefix for all routes?

<details>
<summary>Answer</summary>

Use `setGlobalPrefix()` to add a prefix to all routes (useful for API versioning).

**Basic Usage:**
```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  app.setGlobalPrefix('api');  // All routes prefixed with /api
  
  await app.listen(3000);
}

// Routes become:
// /api/users
// /api/products
// /api/orders
```

**With Version:**
```typescript
app.setGlobalPrefix('api/v1');

// Routes:
// /api/v1/users
// /api/v1/products
```

**Exclude Specific Routes:**
```typescript
app.setGlobalPrefix('api', {
  exclude: [
    'health',  // /health (no prefix)
    'metrics',  // /metrics (no prefix)
    { path: 'auth/login', method: RequestMethod.POST },
  ],
});

// /api/users -> prefixed
// /health -> NOT prefixed
// /metrics -> NOT prefixed
```

**With Versioning:**
```typescript
app.setGlobalPrefix('api');
app.enableVersioning({
  type: VersioningType.URI,
});

@Controller({ version: '1' })
export class UsersV1Controller {}
// /api/v1/users

@Controller({ version: '2' })
export class UsersV2Controller {}
// /api/v2/users
```

**Production Example:**
```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  app.setGlobalPrefix('api/v1', {
    exclude: [
      'health',  // Health check
      'metrics',  // Prometheus metrics
      'docs',  // Swagger docs
    ],
  });
  
  await app.listen(3000);
}
```

**Interview Tip**: Use `app.setGlobalPrefix('api')` to prefix all routes. Exclude specific routes like health checks with the `exclude` option. Combine with versioning for API version management.

</details>

### 39. What is the difference between `app.listen()` and `app.init()`?

<details>
<summary>Answer</summary>

**`app.listen(port)`:**
- Initializes the app AND starts HTTP server
- Returns HTTP server instance
- For standalone apps

```typescript
const app = await NestFactory.create(AppModule);
await app.listen(3000);  // App running on port 3000
```

**`app.init()`:**
- Only initializes the app
- Does NOT start HTTP server
- For serverless (Lambda, Cloud Functions)

```typescript
const app = await NestFactory.create(AppModule);
await app.init();  // Initialized but not listening
```

**When to Use Each:**

**Use `listen()` - Standard Apps:**
```typescript
// Traditional server
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

**Use `init()` - Serverless:**
```typescript
// AWS Lambda
let app: INestApplication;

export const handler = async (event, context) => {
  if (!app) {
    app = await NestFactory.create(AppModule);
    await app.init();  // Initialize once
  }
  
  // Handle request without listening
  return handleRequest(app, event);
};
```

**`listen()` with Custom Server:**
```typescript
const app = await NestFactory.create(AppModule);
const server = await app.listen(3000);

// Access underlying server
server.setTimeout(30000);
server.on('close', () => console.log('Server closed'));
```

**Production Examples:**

**Standard Server:**
```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  const port = process.env.PORT || 3000;
  await app.listen(port);
  console.log(`App listening on port ${port}`);
}
```

**Serverless (Lambda):**
```typescript
import { NestFactory } from '@nestjs/core';
import { ExpressAdapter } from '@nestjs/platform-express';
import * as express from 'express';
import * as awsServerlessExpress from 'aws-serverless-express';

let cachedServer;

async function bootstrapServer() {
  if (!cachedServer) {
    const expressApp = express();
    const app = await NestFactory.create(
      AppModule,
      new ExpressAdapter(expressApp),
    );
    await app.init();  // Don't listen, just init
    cachedServer = awsServerlessExpress.createServer(expressApp);
  }
  return cachedServer;
}

export const handler = async (event, context) => {
  const server = await bootstrapServer();
  return awsServerlessExpress.proxy(server, event, context, 'PROMISE').promise;
};
```

**Interview Tip**: `listen(port)` initializes AND starts the HTTP server for traditional apps. `init()` only initializes without listening, used for serverless environments where the platform handles HTTP.

</details>

### 40. How do you configure Express/Fastify adapter?

<details>
<summary>Answer</summary>

NestJS supports both Express and Fastify as HTTP adapters. Configure them for performance, middleware, and platform-specific features.

**Express (Default):**
```typescript
import { NestFactory } from '@nestjs/core';
import { NestExpressApplication } from '@nestjs/platform-express';

async function bootstrap() {
  const app = await NestFactory.create<NestExpressApplication>(AppModule);
  
  // Express-specific config
  app.set('trust proxy', 1);
  app.use('/static', express.static('public'));
  app.useStaticAssets('public');
  app.setBaseViewsDir('views');
  app.setViewEngine('hbs');
  
  await app.listen(3000);
}
```

**Fastify (Faster):**
```typescript
import { NestFactory } from '@nestjs/core';
import { FastifyAdapter, NestFastifyApplication } from '@nestjs/platform-fastify';

async function bootstrap() {
  const app = await NestFactory.create<NestFastifyApplication>(
    AppModule,
    new FastifyAdapter({ logger: true }),
  );
  
  // Fastify-specific config
  await app.register(require('@fastify/compress'), {
    encodings: ['gzip', 'deflate'],
  });
  
  await app.listen(3000, '0.0.0.0');
}
```

**With Options:**

**Express:**
```typescript
import * as express from 'express';

const expressApp = express();
const app = await NestFactory.create(
  AppModule,
  new ExpressAdapter(expressApp),
);

// Configure Express
expressApp.set('trust proxy', true);
expressApp.use(express.json({ limit: '10mb' }));
expressApp.use(express.urlencoded({ extended: true }));
```

**Fastify:**
```typescript
const fastifyAdapter = new FastifyAdapter({
  logger: true,
  bodyLimit: 10485760,  // 10MB
  trustProxy: true,
  caseSensitive: true,
});

const app = await NestFactory.create<NestFastifyApplication>(
  AppModule,
  fastifyAdapter,
);
```

**Performance Comparison:**
```typescript
// Express - More middleware, slower
const app = await NestFactory.create(AppModule);

// Fastify - 2x faster, less middleware
const app = await NestFactory.create(
  AppModule,
  new FastifyAdapter(),
);
```

**Production Example:**

**Express:**
```typescript
import { NestFactory } from '@nestjs/core';
import { NestExpressApplication } from '@nestjs/platform-express';
import * as compression from 'compression';
import helmet from 'helmet';

async function bootstrap() {
  const app = await NestFactory.create<NestExpressApplication>(AppModule);
  
  // Security
  app.use(helmet());
  
  // Compression
  app.use(compression());
  
  // Trust proxy (behind load balancer)
  app.set('trust proxy', 1);
  
  // Body parsing limits
  app.use(express.json({ limit: '10mb' }));
  
  await app.listen(3000);
}
```

**Fastify:**
```typescript
import { NestFactory } from '@nestjs/core';
import { FastifyAdapter, NestFastifyApplication } from '@nestjs/platform-fastify';

async function bootstrap() {
  const adapter = new FastifyAdapter({
    logger: process.env.NODE_ENV !== 'production',
    trustProxy: true,
    bodyLimit: 10 * 1024 * 1024,  // 10MB
  });

  const app = await NestFactory.create<NestFastifyApplication>(
    AppModule,
    adapter,
  );

  // Fastify plugins
  await app.register(require('@fastify/helmet'));
  await app.register(require('@fastify/compress'));
  await app.register(require('@fastify/rate-limit'), {
    max: 100,
    timeWindow: '1 minute',
  });

  await app.listen(3000, '0.0.0.0');
}
```

**Switching Platforms:**
```typescript
// Install Fastify
// npm install @nestjs/platform-fastify fastify

// Change from Express to Fastify
// Before:
const app = await NestFactory.create(AppModule);

// After:
import { FastifyAdapter } from '@nestjs/platform-fastify';
const app = await NestFactory.create(AppModule, new FastifyAdapter());
```

**Interview Tip**: Express is default, Fastify is 2x faster. Configure adapters for performance (body limits, compression), security (helmet, trust proxy), and platform-specific features (plugins, middleware). Use `NestExpressApplication` or `NestFastifyApplication` types for platform-specific APIs.

</details>
