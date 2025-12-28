# NestJS Middleware - Top Interview Questions

## Middleware Fundamentals

### 1. What is Middleware in NestJS?

<details>
<summary>Answer</summary>

Middleware is a function that executes **before** the route handler, with access to request and response objects.

**Definition:**
```typescript
// Middleware signature
function middleware(req: Request, res: Response, next: NextFunction) {
  // Execute code
  // Modify req/res
  next(); // Call next middleware or route handler
}
```

**Key Characteristics:**
- Executes before route handlers
- Has access to `request` and `response` objects
- Can modify request/response
- Can end request-response cycle
- Must call `next()` to pass control

**Simple Example:**
```typescript
// Functional middleware
export function logger(req: Request, res: Response, next: NextFunction) {
  console.log(`${req.method} ${req.url}`);
  next(); // Must call next()
}

// Class-based middleware
@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log(`${req.method} ${req.url}`);
    next();
  }
}
```

**Common Use Cases:**
- Logging requests
- Authentication/Authorization
- Request validation
- CORS handling
- Body parsing
- Compression
- Security headers (helmet)

**Request Lifecycle:**
```
Client Request
    ↓
Middleware(s)
    ↓
Guards
    ↓
Interceptors (before)
    ↓
Pipes
    ↓
Route Handler
    ↓
Interceptors (after)
    ↓
Exception Filters
    ↓
Client Response
```

**Interview Tip**: Middleware executes before guards/interceptors/pipes, has access to raw req/res objects, and must call `next()` to continue processing. Use for cross-cutting concerns like logging, authentication, and request modification.

</details>

### 2. When does middleware execute in the request lifecycle?

<details>
<summary>Answer</summary>

Middleware executes **first** in the request lifecycle, before guards, interceptors, and pipes.

**Execution Order:**
```typescript
1. Middleware(s)           // ← Executes FIRST
2. Guards
3. Interceptors (before)
4. Pipes
5. Route Handler
6. Interceptors (after)
7. Exception Filters
```

**Visual Flow:**
```
HTTP Request
    ↓
[Global Middleware]
    ↓
[Module Middleware]
    ↓
[Route Middleware]
    ↓
Guards → Check authorization
    ↓
Interceptors (before) → Transform request
    ↓
Pipes → Validate/transform params
    ↓
Route Handler → Execute logic
    ↓
Interceptors (after) → Transform response
    ↓
Exception Filters → Handle errors
    ↓
HTTP Response
```

**Example:**
```typescript
// Middleware - Runs FIRST
@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log('1. Middleware executed');
    next();
  }
}

// Guard - Runs SECOND
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    console.log('2. Guard executed');
    return true;
  }
}

// Interceptor - Runs THIRD (before handler)
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler) {
    console.log('3. Interceptor before handler');
    return next.handle().pipe(
      tap(() => console.log('5. Interceptor after handler'))
    );
  }
}

// Route Handler - Runs FOURTH
@Controller('users')
export class UsersController {
  @Get()
  @UseGuards(AuthGuard)
  @UseInterceptors(LoggingInterceptor)
  findAll() {
    console.log('4. Route handler executed');
    return [];
  }
}

// Output:
// 1. Middleware executed
// 2. Guard executed
// 3. Interceptor before handler
// 4. Route handler executed
// 5. Interceptor after handler
```

**Interview Tip**: Middleware runs FIRST, before all other request processors. Order: Middleware → Guards → Interceptors (before) → Pipes → Handler → Interceptors (after) → Filters.

</details>

### 3. Can you use Express middleware in NestJS?

<details>
<summary>Answer</summary>

Yes, NestJS is built on Express (or Fastify), so you can use **any Express middleware**.

**Using Express Middleware:**
```typescript
// Install Express middleware
npm install helmet cors compression body-parser

// Apply in main.ts or module
import { NestFactory } from '@nestjs/core';
import * as helmet from 'helmet';
import * as cors from 'cors';
import * as compression from 'compression';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Use Express middleware directly
  app.use(helmet());
  app.use(cors());
  app.use(compression());
  
  await app.listen(3000);
}
```

**Popular Express Middleware:**
```typescript
import * as helmet from 'helmet';           // Security headers
import * as cors from 'cors';               // CORS
import * as compression from 'compression'; // Gzip compression
import * as morgan from 'morgan';           // HTTP logger
import * as cookieParser from 'cookie-parser';
import * as session from 'express-session';
import * as passport from 'passport';

app.use(helmet());
app.use(cors());
app.use(compression());
app.use(morgan('combined'));
app.use(cookieParser());
app.use(session({ secret: 'secret', resave: false, saveUninitialized: false }));
app.use(passport.initialize());
app.use(passport.session());
```

**In Module:**
```typescript
@Module({})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    // Use Express middleware
    consumer
      .apply(helmet(), cors(), compression())
      .forRoutes('*');
  }
}
```

**Custom Express Middleware:**
```typescript
// Any Express middleware function
function customMiddleware(req, res, next) {
  req.customProperty = 'value';
  next();
}

// Apply in NestJS
consumer.apply(customMiddleware).forRoutes('*');
```

**Interview Tip**: NestJS supports all Express middleware. Use `app.use()` in main.ts for global middleware or `MiddlewareConsumer` in modules for route-specific middleware.

</details>

### 4. What is the difference between functional and class-based middleware?

<details>
<summary>Answer</summary>

**Functional middleware** is a simple function; **class-based middleware** is an injectable class with dependencies.

**Comparison:**
| Feature | Functional | Class-Based |
|---------|-----------|-------------|
| Syntax | Function | Class with `@Injectable()` |
| Dependencies | No DI | Can inject dependencies |
| Complexity | Simple | Complex logic |
| Use Case | Stateless logic | Needs services/repos |

**Functional Middleware:**
```typescript
// Simple function
export function logger(req: Request, res: Response, next: NextFunction) {
  console.log(`${req.method} ${req.url}`);
  next();
}

// Apply
consumer.apply(logger).forRoutes('*');
```

**Pros:** Simple, lightweight, no overhead  
**Cons:** No dependency injection, harder to test

**Class-Based Middleware:**
```typescript
@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  constructor(
    private logger: LoggerService,      // Can inject dependencies!
    private configService: ConfigService,
  ) {}

  use(req: Request, res: Response, next: NextFunction) {
    const isLoggingEnabled = this.configService.get('LOGGING_ENABLED');
    
    if (isLoggingEnabled) {
      this.logger.log(`${req.method} ${req.url}`);
    }
    
    next();
  }
}

// Apply
consumer.apply(LoggerMiddleware).forRoutes('*');
```

**Pros:** Dependency injection, testable, reusable  
**Cons:** More boilerplate, slightly heavier

**When to Use Each:**
```typescript
// ✅ Use Functional for simple logic
function addTimestamp(req, res, next) {
  req.timestamp = Date.now();
  next();
}

// ✅ Use Class-Based when you need services
@Injectable()
export class AuthMiddleware implements NestMiddleware {
  constructor(
    private authService: AuthService,  // Need to inject
    private userService: UserService,
  ) {}

  async use(req: Request, res: Response, next: NextFunction) {
    const token = req.headers.authorization;
    const user = await this.authService.validateToken(token);
    req.user = user;
    next();
  }
}
```

**Interview Tip**: Use functional middleware for simple, stateless logic. Use class-based middleware when you need dependency injection or complex business logic. Both work the same way, but class-based supports DI.

</details>

## Creating Middleware

### 5. How do you create a functional middleware?

<details>
<summary>Answer</summary>

Create a function with `(req, res, next)` signature and call `next()`.

**Basic Functional Middleware:**
```typescript
import { Request, Response, NextFunction } from 'express';

export function logger(req: Request, res: Response, next: NextFunction) {
  console.log(`${req.method} ${req.url}`);
  next(); // Must call next()
}
```

**Apply in Module:**
```typescript
@Module({
  controllers: [UsersController],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(logger).forRoutes('*');
  }
}
```

**Multiple Functional Middleware:**
```typescript
// Add request ID
export function requestId(req: Request, res: Response, next: NextFunction) {
  req['id'] = Math.random().toString(36).substring(7);
  next();
}

// Add timestamp
export function timestamp(req: Request, res: Response, next: NextFunction) {
  req['timestamp'] = new Date().toISOString();
  next();
}

// Log request
export function logger(req: Request, res: Response, next: NextFunction) {
  console.log(`[${req['id']}] ${req.method} ${req.url} at ${req['timestamp']}`);
  next();
}

// Apply all
consumer.apply(requestId, timestamp, logger).forRoutes('*');
```

**Async Functional Middleware:**
```typescript
export async function validateApiKey(
  req: Request,
  res: Response,
  next: NextFunction,
) {
  const apiKey = req.headers['x-api-key'];
  
  if (!apiKey) {
    return res.status(401).json({ message: 'API key required' });
  }
  
  // Async validation
  const isValid = await validateKey(apiKey);
  
  if (!isValid) {
    return res.status(403).json({ message: 'Invalid API key' });
  }
  
  next();
}
```

**With Custom Types:**
```typescript
interface CustomRequest extends Request {
  user?: { id: string; email: string };
  startTime?: number;
}

export function addUser(
  req: CustomRequest,
  res: Response,
  next: NextFunction,
) {
  req.user = { id: '123', email: 'user@example.com' };
  next();
}

export function measureTime(
  req: CustomRequest,
  res: Response,
  next: NextFunction,
) {
  req.startTime = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - req.startTime!;
    console.log(`Request took ${duration}ms`);
  });
  
  next();
}
```

**Interview Tip**: Functional middleware is just a function with `(req, res, next)` signature. Must call `next()` to continue processing. Use for simple, stateless logic without dependency injection.

</details>

### 6. How do you create a class-based middleware using `NestMiddleware` interface?

<details>
<summary>Answer</summary>

Implement `NestMiddleware` interface with a `use()` method.

**Basic Class-Based Middleware:**
```typescript
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log(`${req.method} ${req.url}`);
    next();
  }
}
```

**Apply in Module:**
```typescript
@Module({
  controllers: [UsersController],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(LoggerMiddleware).forRoutes('*');
  }
}
```

**With Dependency Injection:**
```typescript
@Injectable()
export class AuthMiddleware implements NestMiddleware {
  constructor(
    private authService: AuthService,
    private logger: LoggerService,
  ) {}

  async use(req: Request, res: Response, next: NextFunction) {
    try {
      const token = req.headers.authorization?.split(' ')[1];
      
      if (!token) {
        throw new UnauthorizedException('No token provided');
      }
      
      // Use injected service
      const user = await this.authService.validateToken(token);
      req['user'] = user;
      
      this.logger.log(`User ${user.id} authenticated`);
      next();
    } catch (error) {
      this.logger.error('Authentication failed', error);
      return res.status(401).json({ message: 'Unauthorized' });
    }
  }
}
```

**Complex Example:**
```typescript
@Injectable()
export class RequestLoggingMiddleware implements NestMiddleware {
  constructor(
    private logger: LoggerService,
    private metricsService: MetricsService,
    private configService: ConfigService,
  ) {}

  use(req: Request, res: Response, next: NextFunction) {
    const startTime = Date.now();
    const { method, url, ip } = req;
    
    // Log request
    this.logger.log(`→ ${method} ${url} from ${ip}`);
    
    // Listen to response finish
    res.on('finish', () => {
      const duration = Date.now() - startTime;
      const { statusCode } = res;
      
      this.logger.log(
        `← ${method} ${url} ${statusCode} ${duration}ms`,
      );
      
      // Track metrics
      this.metricsService.recordRequest({
        method,
        url,
        statusCode,
        duration,
      });
    });
    
    next();
  }
}
```

**Interview Tip**: Class-based middleware implements `NestMiddleware` with `use(req, res, next)` method. Use `@Injectable()` to enable dependency injection. Apply via `MiddlewareConsumer` in module's `configure()` method.

</details>

### 7. Can middleware inject dependencies?

<details>
<summary>Answer</summary>

**Class-based middleware** can inject dependencies; **functional middleware** cannot.

**Class-Based with DI:**
```typescript
@Injectable()
export class AuthMiddleware implements NestMiddleware {
  constructor(
    private authService: AuthService,      // ✅ Can inject
    private userService: UserService,      // ✅ Can inject
    private logger: LoggerService,         // ✅ Can inject
    private configService: ConfigService,  // ✅ Can inject
  ) {}

  async use(req: Request, res: Response, next: NextFunction) {
    const token = req.headers.authorization;
    
    // Use injected services
    const user = await this.authService.validateToken(token);
    const fullUser = await this.userService.findById(user.id);
    
    this.logger.log(`User ${user.email} authenticated`);
    
    req['user'] = fullUser;
    next();
  }
}
```

**Functional Cannot Use DI:**
```typescript
// ❌ Cannot inject dependencies
export function authMiddleware(req: Request, res: Response, next: NextFunction) {
  // No access to services!
  // Cannot inject AuthService, UserService, etc.
  next();
}
```

**Workaround for Functional:**
```typescript
// Create a factory function
export function createAuthMiddleware(authService: AuthService) {
  return async (req: Request, res: Response, next: NextFunction) => {
    const token = req.headers.authorization;
    const user = await authService.validateToken(token);
    req['user'] = user;
    next();
  };
}

// Apply with injected service (manual setup)
@Module({})
export class AppModule implements NestModule {
  constructor(private authService: AuthService) {}
  
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(createAuthMiddleware(this.authService))
      .forRoutes('*');
  }
}
```

**Complex DI Example:**
```typescript
@Injectable()
export class TenantMiddleware implements NestMiddleware {
  constructor(
    private tenantService: TenantService,
    private cacheService: CacheService,
    private logger: LoggerService,
  ) {}

  async use(req: Request, res: Response, next: NextFunction) {
    const tenantId = req.headers['x-tenant-id'] as string;
    
    if (!tenantId) {
      return res.status(400).json({ message: 'Tenant ID required' });
    }
    
    // Check cache first
    let tenant = await this.cacheService.get(`tenant:${tenantId}`);
    
    if (!tenant) {
      // Load from database
      tenant = await this.tenantService.findById(tenantId);
      
      if (!tenant) {
        this.logger.warn(`Invalid tenant: ${tenantId}`);
        return res.status(404).json({ message: 'Tenant not found' });
      }
      
      // Cache for 5 minutes
      await this.cacheService.set(`tenant:${tenantId}`, tenant, 300);
    }
    
    req['tenant'] = tenant;
    next();
  }
}
```

**Interview Tip**: Only class-based middleware with `@Injectable()` can inject dependencies. Functional middleware cannot use DI. Use class-based middleware when you need services, repositories, or other injectable providers.

</details>

## Applying Middleware

### 8. What is `MiddlewareConsumer` and how do you use it?

<details>
<summary>Answer</summary>

`MiddlewareConsumer` is used to **configure and apply middleware** to routes in a module.

**Basic Usage:**
```typescript
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';

@Module({
  controllers: [UsersController],
  providers: [UsersService],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)  // Apply middleware
      .forRoutes('*');          // To all routes
  }
}
```

**Apply to Specific Routes:**
```typescript
configure(consumer: MiddlewareConsumer) {
  consumer
    .apply(LoggerMiddleware)
    .forRoutes('users');  // Only /users routes
}
```

**Apply to Specific Controllers:**
```typescript
configure(consumer: MiddlewareConsumer) {
  consumer
    .apply(LoggerMiddleware)
    .forRoutes(UsersController);  // All routes in UsersController
}
```

**Apply Multiple Middleware:**
```typescript
configure(consumer: MiddlewareConsumer) {
  consumer
    .apply(LoggerMiddleware, AuthMiddleware, ValidationMiddleware)
    .forRoutes('*');
}
```

**Apply to Specific HTTP Methods:**
```typescript
configure(consumer: MiddlewareConsumer) {
  consumer
    .apply(AuthMiddleware)
    .forRoutes(
      { path: 'users', method: RequestMethod.POST },
      { path: 'users', method: RequestMethod.DELETE },
    );
}
```

**Exclude Routes:**
```typescript
configure(consumer: MiddlewareConsumer) {
  consumer
    .apply(AuthMiddleware)
    .exclude(
      { path: 'users/public', method: RequestMethod.GET },
      'auth/login',
      'auth/register',
    )
    .forRoutes('*');
}
```

**Multiple Consumers:**
```typescript
configure(consumer: MiddlewareConsumer) {
  // Logger for all routes
  consumer
    .apply(LoggerMiddleware)
    .forRoutes('*');
  
  // Auth for specific routes
  consumer
    .apply(AuthMiddleware)
    .exclude('auth/*')
    .forRoutes('*');
  
  // Validation for users only
  consumer
    .apply(ValidationMiddleware)
    .forRoutes(UsersController);
}
```

**Interview Tip**: `MiddlewareConsumer` configures middleware in module's `configure()` method. Use `.apply()` to specify middleware, `.forRoutes()` to specify routes, and `.exclude()` to exclude routes.

</details>
### 9. How do you apply middleware to specific routes using `forRoutes()`?

<details>
<summary>Answer</summary>

Use `forRoutes()` to apply middleware to specific routes, controllers, or HTTP methods.

**By Path:**
```typescript
configure(consumer: MiddlewareConsumer) {
  consumer
    .apply(LoggerMiddleware)
    .forRoutes('users');  // All /users/* routes
}
```

**By Controller:**
```typescript
configure(consumer: MiddlewareConsumer) {
  consumer
    .apply(LoggerMiddleware)
    .forRoutes(UsersController);  // All routes in controller
}
```

**By HTTP Method:**
```typescript
import { RequestMethod } from '@nestjs/common';

configure(consumer: MiddlewareConsumer) {
  consumer
    .apply(AuthMiddleware)
    .forRoutes(
      { path: 'users', method: RequestMethod.POST },
      { path: 'users', method: RequestMethod.PUT },
      { path: 'users', method: RequestMethod.DELETE },
    );
}
```

**Multiple Routes:**
```typescript
configure(consumer: MiddlewareConsumer) {
  consumer
    .apply(LoggerMiddleware)
    .forRoutes('users', 'products', 'orders');
}
```

**Multiple Controllers:**
```typescript
configure(consumer: MiddlewareConsumer) {
  consumer
    .apply(AuthMiddleware)
    .forRoutes(UsersController, ProductsController, OrdersController);
}
```

**Wildcard Routes:**
```typescript
configure(consumer: MiddlewareConsumer) {
  consumer
    .apply(LoggerMiddleware)
    .forRoutes('*');  // All routes
  
  consumer
    .apply(AdminMiddleware)
    .forRoutes('admin/*');  // All /admin/* routes
}
```

**Complex Configuration:**
```typescript
configure(consumer: MiddlewareConsumer) {
  consumer
    .apply(AuthMiddleware)
    .forRoutes(
      // Specific paths and methods
      { path: 'users', method: RequestMethod.POST },
      { path: 'users/:id', method: RequestMethod.PUT },
      { path: 'users/:id', method: RequestMethod.DELETE },
      // Entire controller
      ProductsController,
      // Wildcard
      { path: 'admin/*', method: RequestMethod.ALL },
    );
}
```

**Interview Tip**: `forRoutes()` accepts strings, controllers, or route objects with path and method. Use `RequestMethod` enum for HTTP methods. Supports wildcards like `*` and path patterns.

</details>

### 10. How do you exclude routes from middleware using `exclude()`?

<details>
<summary>Answer</summary>

Use `exclude()` to skip middleware for specific routes.

**Basic Exclusion:**
```typescript
configure(consumer: MiddlewareConsumer) {
  consumer
    .apply(AuthMiddleware)
    .exclude('auth/login', 'auth/register')  // Skip these routes
    .forRoutes('*');  // Apply to all other routes
}
```

**Exclude with HTTP Methods:**
```typescript
configure(consumer: MiddlewareConsumer) {
  consumer
    .apply(AuthMiddleware)
    .exclude(
      { path: 'users/public', method: RequestMethod.GET },
      { path: 'health', method: RequestMethod.GET },
    )
    .forRoutes('*');
}
```

**Exclude Wildcards:**
```typescript
configure(consumer: MiddlewareConsumer) {
  consumer
    .apply(AuthMiddleware)
    .exclude('auth/*', 'public/*')  // Exclude all auth/* and public/* routes
    .forRoutes('*');
}
```

**Real-World Example:**
```typescript
configure(consumer: MiddlewareConsumer) {
  consumer
    .apply(AuthMiddleware)
    .exclude(
      // Public authentication routes
      'auth/login',
      'auth/register',
      'auth/forgot-password',
      'auth/reset-password',
      // Public endpoints
      { path: 'users/public', method: RequestMethod.GET },
      { path: 'products', method: RequestMethod.GET },
      // Health check
      'health',
      // Docs
      'docs/*',
    )
    .forRoutes('*');
}
```

**Multiple Middleware with Different Exclusions:**
```typescript
configure(consumer: MiddlewareConsumer) {
  // Logger - all routes except health
  consumer
    .apply(LoggerMiddleware)
    .exclude('health')
    .forRoutes('*');
  
  // Auth - all routes except auth and public
  consumer
    .apply(AuthMiddleware)
    .exclude('auth/*', 'public/*')
    .forRoutes('*');
  
  // Rate limiter - all routes except health
  consumer
    .apply(RateLimitMiddleware)
    .exclude('health')
    .forRoutes('*');
}
```

**Interview Tip**: `exclude()` skips middleware for specific routes. Place it before `forRoutes()`. Accepts strings, wildcards, or objects with path and method.

</details>

### 11. How do you apply middleware to multiple routes?

<details>
<summary>Answer</summary>

Pass multiple routes, controllers, or configurations to `forRoutes()`.

**Multiple Paths:**
```typescript
configure(consumer: MiddlewareConsumer) {
  consumer
    .apply(LoggerMiddleware)
    .forRoutes('users', 'products', 'orders', 'categories');
}
```

**Multiple Controllers:**
```typescript
configure(consumer: MiddlewareConsumer) {
  consumer
    .apply(AuthMiddleware)
    .forRoutes(
      UsersController,
      ProductsController,
      OrdersController,
      CategoriesController,
    );
}
```

**Multiple Routes with Methods:**
```typescript
configure(consumer: MiddlewareConsumer) {
  consumer
    .apply(AuthMiddleware)
    .forRoutes(
      { path: 'users', method: RequestMethod.POST },
      { path: 'users/:id', method: RequestMethod.PUT },
      { path: 'users/:id', method: RequestMethod.DELETE },
      { path: 'products', method: RequestMethod.POST },
      { path: 'products/:id', method: RequestMethod.PUT },
    );
}
```

**Multiple Middleware to Multiple Routes:**
```typescript
configure(consumer: MiddlewareConsumer) {
  consumer
    .apply(LoggerMiddleware, AuthMiddleware, ValidationMiddleware)
    .forRoutes(UsersController, ProductsController, OrdersController);
}
```

**Separate Configurations:**
```typescript
configure(consumer: MiddlewareConsumer) {
  // Logger for all
  consumer
    .apply(LoggerMiddleware)
    .forRoutes('*');
  
  // Auth for protected routes
  consumer
    .apply(AuthMiddleware)
    .forRoutes(UsersController, ProductsController);
  
  // Admin middleware for admin routes
  consumer
    .apply(AdminMiddleware)
    .forRoutes('admin/*');
  
  // Rate limiting for API routes
  consumer
    .apply(RateLimitMiddleware)
    .forRoutes('api/*');
}
```

**Interview Tip**: Pass multiple routes/controllers to `forRoutes()` as comma-separated arguments. Can also call `consumer.apply()` multiple times for different middleware configurations.

</details>

### 12. What is the order of middleware execution when multiple are applied?

<details>
<summary>Answer</summary>

Middleware executes in the **order they are applied** (top to bottom, left to right).

**Execution Order:**
```typescript
configure(consumer: MiddlewareConsumer) {
  consumer
    .apply(FirstMiddleware, SecondMiddleware, ThirdMiddleware)  // Order matters!
    .forRoutes('*');
}

// Execution:
// 1. FirstMiddleware
// 2. SecondMiddleware
// 3. ThirdMiddleware
// 4. Route Handler
```

**Example:**
```typescript
@Injectable()
export class FirstMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log('1. First middleware');
    next();
  }
}

@Injectable()
export class SecondMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log('2. Second middleware');
    next();
  }
}

@Injectable()
export class ThirdMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log('3. Third middleware');
    next();
  }
}

configure(consumer: MiddlewareConsumer) {
  consumer
    .apply(FirstMiddleware, SecondMiddleware, ThirdMiddleware)
    .forRoutes('*');
}

// Output:
// 1. First middleware
// 2. Second middleware
// 3. Third middleware
```

**Multiple Apply Calls:**
```typescript
configure(consumer: MiddlewareConsumer) {
  consumer.apply(FirstMiddleware).forRoutes('*');   // Executes 1st
  consumer.apply(SecondMiddleware).forRoutes('*');  // Executes 2nd
  consumer.apply(ThirdMiddleware).forRoutes('*');   // Executes 3rd
}
```

**Best Practice Order:**
```typescript
configure(consumer: MiddlewareConsumer) {
  consumer
    .apply(
      RequestIdMiddleware,     // 1. Generate request ID
      LoggerMiddleware,        // 2. Log with request ID
      CorsMiddleware,          // 3. Handle CORS
      HelmetMiddleware,        // 4. Security headers
      CompressionMiddleware,   // 5. Compress responses
      AuthMiddleware,          // 6. Authenticate user
      TenantMiddleware,        // 7. Load tenant context
      RateLimitMiddleware,     // 8. Rate limiting
    )
    .forRoutes('*');
}
```

**Interview Tip**: Middleware executes in the order applied (left to right in `.apply()`, top to bottom for multiple `.apply()` calls). Order matters - put logging first, authentication before authorization, etc.

</details>

## Global Middleware

### 13. How do you apply middleware globally?

<details>
<summary>Answer</summary>

Apply global middleware in `main.ts` using `app.use()` or in root module using `MiddlewareConsumer`.

**In main.ts (Recommended for Simple Middleware):**
```typescript
import { NestFactory } from '@nestjs/core';
import * as helmet from 'helmet';
import * as cors from 'cors';
import * as compression from 'compression';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Global middleware (no DI)
  app.use(helmet());
  app.use(cors());
  app.use(compression());
  
  await app.listen(3000);
}
bootstrap();
```

**In Root Module (For DI Support):**
```typescript
@Module({
  imports: [UsersModule, ProductsModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware, AuthMiddleware)
      .forRoutes('*');  // Apply to all routes
  }
}
```

**Functional Global Middleware:**
```typescript
// logger.middleware.ts
export function logger(req: Request, res: Response, next: NextFunction) {
  console.log(`${req.method} ${req.url}`);
  next();
}

// main.ts
app.use(logger);
```

**Cannot Use DI in main.ts:**
```typescript
// ❌ This won't work - no DI in main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Cannot inject services here!
  app.use(new LoggerMiddleware(/* where to get LoggerService? */));
}
```

**Solution - Use in Module:**
```typescript
@Module({})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)  // Can inject dependencies
      .forRoutes('*');
  }
}
```

**Interview Tip**: Use `app.use()` in main.ts for simple global middleware without DI. Use module's `configure()` with `.forRoutes('*')` for middleware that needs dependency injection.

</details>

### 14. What is the difference between global and module middleware?

<details>
<summary>Answer</summary>

**Global middleware** applies to all routes; **module middleware** applies only to module's routes.

**Comparison:**
| Feature | Global Middleware | Module Middleware |
|---------|------------------|-------------------|
| Scope | All routes | Module routes only |
| Applied In | `main.ts` or root module | Specific module |
| Dependency Injection | No (if in main.ts) | Yes |
| Use Case | Logging, CORS, helmet | Module-specific logic |

**Global Middleware (main.ts):**
```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Applies to ALL routes in ALL modules
  app.use(logger);
  app.use(helmet());
  app.use(cors());
  
  await app.listen(3000);
}
```

**Global Middleware (Root Module):**
```typescript
@Module({
  imports: [UsersModule, ProductsModule, OrdersModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    // Applies to all routes in all modules
    consumer
      .apply(LoggerMiddleware)
      .forRoutes('*');
  }
}
```

**Module-Specific Middleware:**
```typescript
@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    // Only applies to UsersController routes
    consumer
      .apply(UserValidationMiddleware)
      .forRoutes(UsersController);
  }
}
```

**Real-World Example:**
```typescript
// Global in AppModule
@Module({ imports: [UsersModule, AdminModule] })
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    // Logger for everything
    consumer.apply(LoggerMiddleware).forRoutes('*');
    
    // Auth for everything except auth routes
    consumer
      .apply(AuthMiddleware)
      .exclude('auth/*')
      .forRoutes('*');
  }
}

// Module-specific in UsersModule
@Module({})
export class UsersModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    // Only for users routes
    consumer
      .apply(UserCacheMiddleware)
      .forRoutes(UsersController);
  }
}

// Module-specific in AdminModule
@Module({})
export class AdminModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    // Only for admin routes
    consumer
      .apply(AdminAuthMiddleware, AuditMiddleware)
      .forRoutes(AdminController);
  }
}
```

**Interview Tip**: Global middleware in main.ts affects all routes but can't use DI. Module middleware in `configure()` is scoped to that module's routes and supports DI. Use global for cross-cutting concerns, module-specific for domain logic.

</details>

### 15. Can you use dependency injection in global middleware?

<details>
<summary>Answer</summary>

**Not in main.ts**, but **yes in root module** using `configure()`.

**❌ No DI in main.ts:**
```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // ❌ Cannot inject dependencies
  app.use((req, res, next) => {
    // No access to LoggerService, ConfigService, etc.
    next();
  });
  
  await app.listen(3000);
}
```

**✅ Yes DI in Root Module:**
```typescript
@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  constructor(
    private logger: LoggerService,     // ✅ Can inject
    private config: ConfigService,     // ✅ Can inject
  ) {}

  use(req: Request, res: Response, next: NextFunction) {
    const isLoggingEnabled = this.config.get('LOGGING_ENABLED');
    
    if (isLoggingEnabled) {
      this.logger.log(`${req.method} ${req.url}`);
    }
    
    next();
  }
}

@Module({
  imports: [ConfigModule, UsersModule, ProductsModule],
  providers: [LoggerService],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    // ✅ LoggerMiddleware can inject dependencies
    consumer
      .apply(LoggerMiddleware)
      .forRoutes('*');
  }
}
```

**Real-World Example:**
```typescript
@Injectable()
export class AuthMiddleware implements NestMiddleware {
  constructor(
    private authService: AuthService,
    private userService: UserService,
    private cacheService: CacheService,
    private logger: LoggerService,
  ) {}

  async use(req: Request, res: Response, next: NextFunction) {
    const token = req.headers.authorization?.split(' ')[1];
    
    if (!token) {
      return res.status(401).json({ message: 'No token' });
    }
    
    // Check cache
    let user = await this.cacheService.get(`user:${token}`);
    
    if (!user) {
      // Validate token and get user
      const payload = await this.authService.validateToken(token);
      user = await this.userService.findById(payload.sub);
      
      // Cache for 5 minutes
      await this.cacheService.set(`user:${token}`, user, 300);
    }
    
    this.logger.log(`User ${user.email} authenticated`);
    req['user'] = user;
    next();
  }
}

@Module({
  imports: [AuthModule, CacheModule],
  providers: [LoggerService],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(AuthMiddleware)  // Has full DI support
      .exclude('auth/*')
      .forRoutes('*');
  }
}
```

**Interview Tip**: Global middleware in main.ts cannot use DI. To use DI with global scope, apply class-based middleware to `'*'` in root module's `configure()` method.

</details>

## Common Middleware Use Cases

### 16. How do you implement logging middleware?

<details>
<summary>Answer</summary>

Create middleware that logs requests, responses, and timing.

**Simple Logging:**
```typescript
@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log(`${req.method} ${req.url}`);
    next();
  }
}
```

**With Request ID and Timing:**
```typescript
@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  constructor(private logger: LoggerService) {}

  use(req: Request, res: Response, next: NextFunction) {
    const { method, url, ip } = req;
    const userAgent = req.get('user-agent') || '';
    const requestId = Math.random().toString(36).substring(7);
    
    req['requestId'] = requestId;
    const startTime = Date.now();
    
    // Log incoming request
    this.logger.log(
      `→ [${requestId}] ${method} ${url} - ${ip} - ${userAgent}`,
    );
    
    // Listen to response finish
    res.on('finish', () => {
      const { statusCode } = res;
      const duration = Date.now() - startTime;
      const contentLength = res.get('content-length') || 0;
      
      this.logger.log(
        `← [${requestId}] ${method} ${url} ${statusCode} ${duration}ms ${contentLength}bytes`,
      );
    });
    
    next();
  }
}
```

**Production-Grade Logging:**
```typescript
@Injectable()
export class RequestLoggingMiddleware implements NestMiddleware {
  constructor(
    private logger: LoggerService,
    private metricsService: MetricsService,
  ) {}

  use(req: Request, res: Response, next: NextFunction) {
    const startTime = Date.now();
    const { method, originalUrl, ip, headers } = req;
    const userAgent = headers['user-agent'] || '';
    const requestId = headers['x-request-id'] || this.generateId();
    
    req['requestId'] = requestId;
    res.setHeader('X-Request-Id', requestId);
    
    // Log request
    this.logger.log({
      type: 'request',
      requestId,
      method,
      url: originalUrl,
      ip,
      userAgent,
      timestamp: new Date().toISOString(),
    });
    
    // Capture response
    const originalSend = res.send;
    res.send = function (data) {
      res.send = originalSend;
      return res.send(data);
    };
    
    res.on('finish', () => {
      const duration = Date.now() - startTime;
      const { statusCode } = res;
      const contentLength = res.get('content-length') || 0;
      
      // Log response
      this.logger.log({
        type: 'response',
        requestId,
        method,
        url: originalUrl,
        statusCode,
        duration,
        contentLength,
        timestamp: new Date().toISOString(),
      });
      
      // Track metrics
      this.metricsService.recordRequest({
        method,
        url: originalUrl,
        statusCode,
        duration,
      });
    });
    
    next();
  }
  
  private generateId(): string {
    return Math.random().toString(36).substring(2) + Date.now().toString(36);
  }
}
```

**Interview Tip**: Logging middleware captures request/response details and timing. Use `res.on('finish')` to log after response is sent. Include request ID for tracing, measure duration for performance monitoring.

</details>

### 17. How do you implement authentication middleware?

<details>
<summary>Answer</summary>

Create middleware that validates tokens and attaches user to request.

**Basic JWT Authentication:**
```typescript
@Injectable()
export class AuthMiddleware implements NestMiddleware {
  constructor(
    private authService: AuthService,
    private logger: LoggerService,
  ) {}

  async use(req: Request, res: Response, next: NextFunction) {
    try {
      const token = req.headers.authorization?.split(' ')[1];
      
      if (!token) {
        throw new UnauthorizedException('No token provided');
      }
      
      // Validate token
      const payload = await this.authService.validateToken(token);
      
      // Attach user to request
      req['user'] = payload;
      
      next();
    } catch (error) {
      this.logger.error('Authentication failed', error);
      return res.status(401).json({
        statusCode: 401,
        message: 'Unauthorized',
      });
    }
  }
}
```

**With User Loading:**
```typescript
@Injectable()
export class AuthMiddleware implements NestMiddleware {
  constructor(
    private authService: AuthService,
    private userService: UserService,
    private logger: LoggerService,
  ) {}

  async use(req: Request, res: Response, next: NextFunction) {
    const token = this.extractToken(req);
    
    if (!token) {
      return res.status(401).json({ message: 'No token provided' });
    }
    
    try {
      // Validate token
      const payload = await this.authService.validateToken(token);
      
      // Load full user
      const user = await this.userService.findById(payload.sub);
      
      if (!user || !user.isActive) {
        throw new UnauthorizedException('User not found or inactive');
      }
      
      // Attach to request
      req['user'] = user;
      req['token'] = token;
      
      this.logger.log(`User ${user.email} authenticated`);
      
      next();
    } catch (error) {
      this.logger.error(`Auth failed: ${error.message}`);
      return res.status(401).json({
        statusCode: 401,
        message: error.message || 'Unauthorized',
      });
    }
  }
  
  private extractToken(req: Request): string | null {
    const authHeader = req.headers.authorization;
    
    if (authHeader && authHeader.startsWith('Bearer ')) {
      return authHeader.substring(7);
    }
    
    // Check cookie
    return req.cookies?.['access_token'] || null;
  }
}
```

**With Caching:**
```typescript
@Injectable()
export class AuthMiddleware implements NestMiddleware {
  constructor(
    private authService: AuthService,
    private userService: UserService,
    private cacheService: CacheService,
    private logger: LoggerService,
  ) {}

  async use(req: Request, res: Response, next: NextFunction) {
    const token = this.extractToken(req);
    
    if (!token) {
      return res.status(401).json({ message: 'No token provided' });
    }
    
    try {
      // Check cache first
      const cacheKey = `auth:${token}`;
      let user = await this.cacheService.get(cacheKey);
      
      if (!user) {
        // Validate and load user
        const payload = await this.authService.validateToken(token);
        user = await this.userService.findById(payload.sub);
        
        if (!user || !user.isActive) {
          throw new UnauthorizedException('User not found or inactive');
        }
        
        // Cache for 5 minutes
        await this.cacheService.set(cacheKey, user, 300);
      }
      
      req['user'] = user;
      next();
    } catch (error) {
      this.logger.error(`Auth failed: ${error.message}`);
      return res.status(401).json({
        statusCode: 401,
        message: 'Unauthorized',
      });
    }
  }
  
  private extractToken(req: Request): string | null {
    const authHeader = req.headers.authorization;
    return authHeader?.startsWith('Bearer ') ? authHeader.substring(7) : null;
  }
}

// Apply with exclusions
@Module({})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(AuthMiddleware)
      .exclude(
        'auth/login',
        'auth/register',
        'health',
        { path: 'public/*', method: RequestMethod.GET },
      )
      .forRoutes('*');
  }
}
```

**Interview Tip**: Auth middleware validates tokens, loads user data, and attaches user to `req['user']`. Extract token from `Authorization` header or cookies. Cache user data to reduce database queries. Exclude public routes.

</details>

### 18. How do you use `helmet` for security?

<details>
<summary>Answer</summary>

`helmet` sets security-related HTTP headers.

**Installation:**
```bash
npm install helmet
```

**Basic Usage:**
```typescript
import * as helmet from 'helmet';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Apply helmet
  app.use(helmet());
  
  await app.listen(3000);
}
```

**With Options:**
```typescript
app.use(
  helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        styleSrc: ["'self'", "'unsafe-inline'"],
        scriptSrc: ["'self'"],
        imgSrc: ["'self'", 'data:', 'https:'],
      },
    },
    hsts: {
      maxAge: 31536000,
      includeSubDomains: true,
      preload: true,
    },
  }),
);
```

**What Helmet Does:**
```typescript
// Helmet sets these headers:
// X-DNS-Prefetch-Control: off
// X-Frame-Options: SAMEORIGIN
// Strict-Transport-Security: max-age=15552000; includeSubDomains
// X-Download-Options: noopen
// X-Content-Type-Options: nosniff
// X-XSS-Protection: 0
```

**Custom Configuration:**
```typescript
import helmet from 'helmet';

app.use(
  helmet({
    // Enable/disable specific middleware
    contentSecurityPolicy: false,
    crossOriginEmbedderPolicy: false,
    
    // Custom HSTS
    hsts: {
      maxAge: 31536000,
      includeSubDomains: true,
    },
    
    // Frame options
    frameguard: {
      action: 'deny',
    },
  }),
);
```

**Interview Tip**: `helmet` is Express middleware that sets security HTTP headers. Use `app.use(helmet())` in main.ts. Protects against common vulnerabilities like XSS, clickjacking, and MIME sniffing.

</details>

### 19. How do you use `cors` middleware?

<details>
<summary>Answer</summary>

`cors` enables Cross-Origin Resource Sharing.

**Installation:**
```bash
npm install cors
npm install -D @types/cors
```

**Basic Usage:**
```typescript
import * as cors from 'cors';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Enable CORS for all origins
  app.use(cors());
  
  await app.listen(3000);
}
```

**NestJS Built-in CORS:**
```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule, {
    cors: true,  // Enable CORS
  });
  
  await app.listen(3000);
}
```

**With Options:**
```typescript
app.enableCors({
  origin: 'https://example.com',
  methods: 'GET,HEAD,PUT,PATCH,POST,DELETE',
  credentials: true,
  allowedHeaders: 'Content-Type,Authorization',
});
```

**Multiple Origins:**
```typescript
app.enableCors({
  origin: [
    'https://example.com',
    'https://app.example.com',
    'http://localhost:3000',
  ],
  credentials: true,
});
```

**Dynamic Origin:**
```typescript
app.enableCors({
  origin: (origin, callback) => {
    const allowedOrigins = [
      'https://example.com',
      'https://app.example.com',
    ];
    
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true,
});
```

**Production Configuration:**
```typescript
app.enableCors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') || '*',
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'OPTIONS'],
  allowedHeaders: [
    'Content-Type',
    'Authorization',
    'X-Requested-With',
    'X-Request-Id',
  ],
  exposedHeaders: ['X-Request-Id'],
  credentials: true,
  maxAge: 3600, // 1 hour
});
```

**Interview Tip**: Use `app.enableCors()` in main.ts to enable CORS. Configure `origin`, `methods`, `credentials`, and `allowedHeaders`. In production, whitelist specific origins instead of using `'*'`.

</details>

### 20. How do you use `compression` middleware?

<details>
<summary>Answer</summary>

`compression` compresses HTTP responses with gzip.

**Installation:**
```bash
npm install compression
npm install -D @types/compression
```

**Basic Usage:**
```typescript
import * as compression from 'compression';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Enable compression
  app.use(compression());
  
  await app.listen(3000);
}
```

**With Options:**
```typescript
app.use(
  compression({
    level: 6,  // Compression level (0-9)
    threshold: 1024,  // Minimum size to compress (bytes)
    filter: (req, res) => {
      // Don't compress if client doesn't support it
      if (req.headers['x-no-compression']) {
        return false;
      }
      // Use compression defaults
      return compression.filter(req, res);
    },
  }),
);
```

**Custom Filter:**
```typescript
app.use(
  compression({
    filter: (req, res) => {
      // Don't compress images
      if (res.getHeader('Content-Type')?.toString().includes('image')) {
        return false;
      }
      // Compress JSON and text
      return compression.filter(req, res);
    },
  }),
);
```

**Interview Tip**: `compression` middleware compresses responses with gzip to reduce bandwidth. Use `app.use(compression())` in main.ts. Set `threshold` to avoid compressing small responses. Saves bandwidth and improves performance.

</details>

## Advanced Concepts

### 21. Can middleware be async?

<details>
<summary>Answer</summary>

Yes, middleware can be **async** and use `await`.

**Async Middleware:**
```typescript
@Injectable()
export class AuthMiddleware implements NestMiddleware {
  constructor(private authService: AuthService) {}

  async use(req: Request, res: Response, next: NextFunction) {
    const token = req.headers.authorization?.split(' ')[1];
    
    // Async operation
    const user = await this.authService.validateToken(token);
    
    req['user'] = user;
    next();
  }
}
```

**Multiple Async Operations:**
```typescript
@Injectable()
export class TenantMiddleware implements NestMiddleware {
  constructor(
    private tenantService: TenantService,
    private cacheService: CacheService,
  ) {}

  async use(req: Request, res: Response, next: NextFunction) {
    const tenantId = req.headers['x-tenant-id'] as string;
    
    // Async cache lookup
    let tenant = await this.cacheService.get(`tenant:${tenantId}`);
    
    if (!tenant) {
      // Async database query
      tenant = await this.tenantService.findById(tenantId);
      
      // Async cache set
      await this.cacheService.set(`tenant:${tenantId}`, tenant, 300);
    }
    
    req['tenant'] = tenant;
    next();
  }
}
```

**Error Handling:**
```typescript
@Injectable()
export class AuthMiddleware implements NestMiddleware {
  async use(req: Request, res: Response, next: NextFunction) {
    try {
      const token = req.headers.authorization?.split(' ')[1];
      const user = await this.authService.validateToken(token);
      req['user'] = user;
      next();
    } catch (error) {
      // Handle async errors
      return res.status(401).json({ message: error.message });
    }
  }
}
```

**Interview Tip**: Middleware can be async. Use `async use()` method and `await` for async operations. Remember to call `next()` after async operations complete. Handle errors with try/catch.

</details>

### 22. How do you handle errors in middleware?

<details>
<summary>Answer</summary>

Handle errors with try/catch and send appropriate error responses.

**Basic Error Handling:**
```typescript
@Injectable()
export class AuthMiddleware implements NestMiddleware {
  async use(req: Request, res: Response, next: NextFunction) {
    try {
      const token = req.headers.authorization?.split(' ')[1];
      
      if (!token) {
        throw new Error('No token provided');
      }
      
      const user = await this.authService.validateToken(token);
      req['user'] = user;
      next();
    } catch (error) {
      return res.status(401).json({
        statusCode: 401,
        message: error.message,
      });
    }
  }
}
```

**With Logging:**
```typescript
@Injectable()
export class AuthMiddleware implements NestMiddleware {
  constructor(
    private authService: AuthService,
    private logger: LoggerService,
  ) {}

  async use(req: Request, res: Response, next: NextFunction) {
    try {
      const token = this.extractToken(req);
      const user = await this.authService.validateToken(token);
      req['user'] = user;
      next();
    } catch (error) {
      this.logger.error('Authentication failed', {
        error: error.message,
        url: req.url,
        ip: req.ip,
      });
      
      return res.status(401).json({
        statusCode: 401,
        message: 'Unauthorized',
        timestamp: new Date().toISOString(),
      });
    }
  }
  
  private extractToken(req: Request): string {
    const token = req.headers.authorization?.split(' ')[1];
    if (!token) {
      throw new Error('No token provided');
    }
    return token;
  }
}
```

**Pass Errors to Exception Filter:**
```typescript
@Injectable()
export class AuthMiddleware implements NestMiddleware {
  async use(req: Request, res: Response, next: NextFunction) {
    try {
      const token = req.headers.authorization?.split(' ')[1];
      const user = await this.authService.validateToken(token);
      req['user'] = user;
      next();
    } catch (error) {
      // Pass to next error handler
      next(error);
    }
  }
}
```

**Interview Tip**: Use try/catch for error handling in middleware. Send error response with `res.status().json()` or pass error to exception filters with `next(error)`. Log errors for debugging.

</details>

### 23. What is the difference between middleware and interceptors?

<details>
<summary>Answer</summary>

**Middleware** runs before routing; **interceptors** run before/after handler with RxJS support.

**Comparison:**
| Feature | Middleware | Interceptors |
|---------|-----------|--------------|
| Execution | Before routing | Before/after handler |
| Access to | req, res, next | ExecutionContext, CallHandler |
| Transform response | Hard | Easy (RxJS) |
| DI Support | Class-based only | Yes |
| Use Case | Auth, logging, CORS | Transform, cache, timeout |

**Middleware:**
```typescript
@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log('Before routing');
    next();  // Cannot transform response easily
  }
}
```

**Interceptor:**
```typescript
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler) {
    console.log('Before handler');
    
    return next.handle().pipe(
      tap(() => console.log('After handler')),
      map(data => ({ data, timestamp: Date.now() })),  // Easy transform
    );
  }
}
```

**When to Use Each:**
```typescript
// ✅ Use Middleware for:
// - Authentication (before routing)
// - Logging requests
// - CORS, helmet, compression
// - Request parsing
// - Setting headers

// ✅ Use Interceptors for:
// - Transforming responses
// - Caching
// - Timeouts
// - Logging after handler
// - Adding metadata to responses
```

**Interview Tip**: Middleware runs before routing, has access to raw req/res, cannot easily transform responses. Interceptors run around handlers, use RxJS for response transformation, have access to ExecutionContext. Use middleware for pre-routing logic, interceptors for response transformation.

</details>

### 24. What is the difference between middleware and guards?

<details>
<summary>Answer</summary>

**Middleware** runs before routing; **guards** run after routing and determine if handler executes.

**Comparison:**
| Feature | Middleware | Guards |
|---------|-----------|---------|
| Execution | Before routing | After routing, before handler |
| Purpose | Modify req/res | Allow/deny access |
| Return | void (calls next) | boolean/Promise<boolean> |
| Access to | req, res | ExecutionContext |
| Route metadata | No | Yes (via Reflector) |

**Middleware:**
```typescript
@Injectable()
export class AuthMiddleware implements NestMiddleware {
  async use(req: Request, res: Response, next: NextFunction) {
    // Validate and attach user
    const user = await this.authService.validateToken(token);
    req['user'] = user;
    next();  // Always continue
  }
}
```

**Guard:**
```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    // Decide if request can proceed
    return !!request.user;  // true = allow, false = deny
  }
}
```

**When to Use Each:**
```typescript
// ✅ Use Middleware for:
// - Authentication (validate token, attach user)
// - Logging
// - Request parsing
// - Setting up context

// ✅ Use Guards for:
// - Authorization (check roles, permissions)
// - Route protection
// - Conditional access based on metadata
```

**Combined Usage:**
```typescript
// Middleware: Authenticate user
@Injectable()
export class AuthMiddleware implements NestMiddleware {
  async use(req: Request, res: Response, next: NextFunction) {
    const token = req.headers.authorization?.split(' ')[1];
    const user = await this.authService.validateToken(token);
    req['user'] = user;  // Attach user
    next();
  }
}

// Guard: Check if user has admin role
@Injectable()
export class AdminGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    return user?.role === 'admin';  // Authorize
  }
}

// Apply both
@Module({})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(AuthMiddleware).forRoutes('*');
  }
}

@Controller('admin')
export class AdminController {
  @Get()
  @UseGuards(AdminGuard)  // Only admins can access
  getAdminData() {}
}
```

**Interview Tip**: Middleware authenticates and modifies requests before routing. Guards authorize access after routing based on conditions. Use middleware for authentication, guards for authorization. Guards have access to route metadata and ExecutionContext.

</details>

### 25. When should you use middleware vs guards vs interceptors?

<details>
<summary>Answer</summary>

Use based on **when** and **what** you need to do in the request lifecycle.

**Decision Matrix:**
| Use Case | Use | Reason |
|----------|-----|--------|
| Authenticate user | Middleware | Need req/res before routing |
| Check permissions | Guard | Need to block/allow based on metadata |
| Transform response | Interceptor | Need RxJS, runs after handler |
| Logging requests | Middleware | Before routing |
| Logging responses | Interceptor | After handler |
| CORS, helmet | Middleware | Before routing |
| Cache | Interceptor | Transform response |
| Timeout | Interceptor | RxJS timeout operator |
| Rate limiting | Middleware or Guard | Either works |
| Validate roles | Guard | Access to route metadata |

**Execution Order:**
```
Request
  ↓
Middleware (authenticate, log request)
  ↓
Guards (authorize, check permissions)
  ↓
Interceptors (before)
  ↓
Pipes (validate)
  ↓
Handler
  ↓
Interceptors (after - transform response)
  ↓
Response
```

**Real-World Example:**
```typescript
// 1. Middleware - Authenticate user
@Injectable()
export class AuthMiddleware implements NestMiddleware {
  async use(req, res, next) {
    const user = await this.authService.validateToken(token);
    req['user'] = user;  // Attach user
    next();
  }
}

// 2. Guard - Check if user has permission
@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const roles = this.reflector.get('roles', context.getHandler());
    const request = context.switchToHttp().getRequest();
    return roles.includes(request.user.role);  // Authorize
  }
}

// 3. Interceptor - Transform response
@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(context, next: CallHandler) {
    return next.handle().pipe(
      map(data => ({
        data,
        timestamp: Date.now(),
        success: true,
      })),
    );
  }
}

// Apply
@Module({})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(AuthMiddleware).forRoutes('*');  // 1. Auth
  }
}

@Controller('users')
@UseInterceptors(TransformInterceptor)  // 3. Transform
export class UsersController {
  @Get()
  @Roles('admin', 'user')
  @UseGuards(RolesGuard)  // 2. Authorize
  findAll() {
    return [];
  }
}
```

**Quick Reference:**
```typescript
// Middleware: Authentication, logging, CORS, helmet
app.use(helmet());
consumer.apply(AuthMiddleware).forRoutes('*');

// Guards: Authorization, role checks, conditional access
@UseGuards(AuthGuard, RolesGuard)

// Interceptors: Response transformation, caching, logging after
@UseInterceptors(TransformInterceptor, CacheInterceptor)

// Pipes: Validation, transformation of input
@UsePipes(ValidationPipe)
```

**Interview Tip**: Use middleware for authentication and pre-routing tasks. Use guards for authorization and conditional access with metadata. Use interceptors for response transformation with RxJS. Order: Middleware → Guards → Interceptors (before) → Handler → Interceptors (after).

</details>