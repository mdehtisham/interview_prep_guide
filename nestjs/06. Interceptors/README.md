# NestJS Interceptors - Top Interview Questions

## Interceptor Fundamentals

### 1. What is an Interceptor in NestJS?

<details>
<summary>Answer</summary>

An **Interceptor** is a class annotated with `@Injectable()` that implements `NestInterceptor` interface. Interceptors can **transform** the result returned from a function, **transform** the exception thrown from a function, **extend** basic function behavior, or **completely override** a function.

**Key Capabilities:**
1. Bind extra logic before/after method execution
2. Transform the result returned from a function
3. Transform exceptions thrown from a function
4. Extend basic function behavior
5. Completely override a function (e.g., caching)

**Basic Structure:**
```typescript
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';

@Injectable()
export class MyInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('Before handler execution');
    
    return next.handle().pipe(
      tap(() => console.log('After handler execution')),
    );
  }
}
```

**Request Lifecycle Position:**
```
Client Request
    ↓
Middleware
    ↓
Guards
    ↓
INTERCEPTORS (before) ← Execute before handler
    ↓
Pipes
    ↓
Route Handler
    ↓
INTERCEPTORS (after) ← Execute after handler
    ↓
Response
```

**Simple Example:**
```typescript
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const now = Date.now();
    
    return next.handle().pipe(
      tap(() => {
        console.log(`Execution time: ${Date.now() - now}ms`);
      }),
    );
  }
}

// Usage
@Controller('users')
@UseInterceptors(LoggingInterceptor)
export class UsersController {
  @Get()
  findAll() {
    return [];
  }
}
```

**Transform Response:**
```typescript
@Injectable()
export class TransformInterceptor implements NestInterceptor {
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

// Original response: [{ id: 1, name: 'John' }]
// Transformed response:
{
  "success": true,
  "data": [{ "id": 1, "name": "John" }],
  "timestamp": "2024-01-01T00:00:00.000Z"
}
```

**Interview Tip**: Interceptors run before and after route handlers. They wrap handler execution with RxJS observables, allowing transformation of requests/responses. Use for logging, response transformation, caching, timeouts, error handling.

</details>

### 2. When are Interceptors executed in the request lifecycle?

<details>
<summary>Answer</summary>

Interceptors execute **twice**: **before** the route handler (after Guards) and **after** the route handler (before response).

**Request Lifecycle Order:**
```
1. Middleware
2. Guards
3. INTERCEPTORS (before handler) ← First execution
4. Pipes
5. Route Handler
6. INTERCEPTORS (after handler) ← Second execution
7. Exception Filters
8. Response
```

**Visual Flow:**
```typescript
Client Request
    ↓
[Middleware] - CORS, logging, auth setup
    ↓
[Guards] - Authorization checks
    ↓
[INTERCEPTORS - BEFORE] ← Wrap handler execution
    ↓
[Pipes] - Validation & transformation
    ↓
[Route Handler] - Business logic
    ↓
[INTERCEPTORS - AFTER] ← Transform response
    ↓
[Exception Filters] - Error handling
    ↓
Response to Client
```

**Example with Timing:**
```typescript
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('1. BEFORE handler - Interceptor starts');
    const now = Date.now();
    
    return next.handle().pipe(
      tap(() => {
        console.log('3. AFTER handler - Interceptor ends');
        console.log(`Execution time: ${Date.now() - now}ms`);
      }),
    );
  }
}

@Controller('users')
@UseInterceptors(LoggingInterceptor)
export class UsersController {
  @Get()
  findAll() {
    console.log('2. DURING handler execution');
    return ['user1', 'user2'];
  }
}

// Console output:
// 1. BEFORE handler - Interceptor starts
// 2. DURING handler execution
// 3. AFTER handler - Interceptor ends
// Execution time: 5ms
```

**Multiple Interceptors:**
```typescript
@Controller('posts')
@UseInterceptors(LoggingInterceptor, TransformInterceptor, CacheInterceptor)
export class PostsController {
  @Get()
  findAll() {
    return [];
  }
}

// Execution order:
// 1. LoggingInterceptor (before)
// 2. TransformInterceptor (before)
// 3. CacheInterceptor (before)
// 4. Handler executes
// 5. CacheInterceptor (after)
// 6. TransformInterceptor (after)
// 7. LoggingInterceptor (after)
// (Last in, first out - like middleware stack)
```

**Before vs After:**
```typescript
@Injectable()
export class BeforeAfterInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    // BEFORE: Access request, modify context
    const request = context.switchToHttp().getRequest();
    console.log('Request URL:', request.url);
    console.log('Request Method:', request.method);
    
    // Call handler and wait for response
    return next.handle().pipe(
      // AFTER: Transform response, measure time, log
      map(data => {
        console.log('Response data:', data);
        return { ...data, timestamp: Date.now() };
      }),
      tap(() => console.log('Response sent')),
    );
  }
}
```

**Interview Tip**: Interceptors execute **before** (after Guards, before Pipes) and **after** (after handler, before response) route handlers. Multiple interceptors execute in order going in, reverse order coming out (like onion layers). Use RxJS `pipe()` to define after-handler logic.

</details>

### 3. What is the `NestInterceptor` interface?

<details>
<summary>Answer</summary>

`NestInterceptor` is the interface that all Interceptors must implement. It has one method: `intercept()`.

**Interface Definition:**
```typescript
export interface NestInterceptor<T = any, R = any> {
  intercept(context: ExecutionContext, next: CallHandler<T>): Observable<R> | Promise<Observable<R>>;
}
```

**Type Parameters:**
- `T` - Type of handler return value
- `R` - Type of interceptor return value

**Basic Implementation:**
```typescript
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class MyInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('Before...');
    
    return next.handle().pipe(
      tap(() => console.log('After...')),
    );
  }
}
```

**With Type Safety:**
```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

interface UserResponse {
  success: boolean;
  data: User;
}

@Injectable()
export class TypedInterceptor implements NestInterceptor<User, UserResponse> {
  intercept(
    context: ExecutionContext,
    next: CallHandler<User>,
  ): Observable<UserResponse> {
    return next.handle().pipe(
      map((user: User) => ({
        success: true,
        data: user,
      })),
    );
  }
}
```

**Method Signature:**
```typescript
intercept(
  context: ExecutionContext,  // Information about current request
  next: CallHandler,          // Call to invoke route handler
): Observable<any>            // Must return Observable
```

**Generic Interceptor:**
```typescript
@Injectable()
export class ResponseWrapperInterceptor<T> implements NestInterceptor<T, { data: T }> {
  intercept(context: ExecutionContext, next: CallHandler<T>): Observable<{ data: T }> {
    return next.handle().pipe(
      map(data => ({ data })),
    );
  }
}

// Original: { id: 1, name: 'John' }
// Wrapped: { data: { id: 1, name: 'John' } }
```

**Async Interceptor:**
```typescript
@Injectable()
export class AsyncInterceptor implements NestInterceptor {
  async intercept(
    context: ExecutionContext,
    next: CallHandler,
  ): Promise<Observable<any>> {
    // Can do async work before handler
    await this.doSomethingAsync();
    
    return next.handle().pipe(
      tap(() => console.log('After handler')),
    );
  }
  
  private async doSomethingAsync() {
    // Async operations
  }
}
```

**Interview Tip**: All Interceptors implement `NestInterceptor<T, R>` interface with `intercept(context, next)` method. Must return `Observable<R>`. Use `next.handle()` to invoke handler and get its Observable. Apply RxJS operators with `.pipe()` to transform response.

</details>

### 4. What parameters does the `intercept()` method receive?

<details>
<summary>Answer</summary>

The `intercept()` method receives two parameters: `context` and `next`.

**Method Signature:**
```typescript
intercept(
  context: ExecutionContext,  // Current execution context
  next: CallHandler,          // Handler to call next interceptor/route handler
): Observable<any>
```

**1. ExecutionContext - Request Information:**
```typescript
@Injectable()
export class ContextInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    // Get HTTP context
    const http = context.switchToHttp();
    const request = http.getRequest();
    const response = http.getResponse();
    
    console.log('URL:', request.url);
    console.log('Method:', request.method);
    console.log('Headers:', request.headers);
    
    // Get handler and class references
    const handler = context.getHandler();
    const controller = context.getClass();
    
    console.log('Handler:', handler.name);
    console.log('Controller:', controller.name);
    
    return next.handle();
  }
}
```

**2. CallHandler - Invoke Next Handler:**
```typescript
@Injectable()
export class CallHandlerInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('Before calling handler');
    
    // Call handler and get Observable
    const observable = next.handle();
    
    // Transform the Observable
    return observable.pipe(
      tap(() => console.log('After handler')),
      map(data => ({ success: true, data })),
    );
  }
}
```

**Using Both Parameters:**
```typescript
@Injectable()
export class FullInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    // Use context to get request info
    const request = context.switchToHttp().getRequest();
    const { method, url } = request;
    
    console.log(`${method} ${url} - Starting`);
    const start = Date.now();
    
    // Use next to call handler
    return next.handle().pipe(
      tap(() => {
        const duration = Date.now() - start;
        console.log(`${method} ${url} - Completed in ${duration}ms`);
      }),
    );
  }
}
```

**Access Metadata:**
```typescript
import { Reflector } from '@nestjs/core';

@Injectable()
export class MetadataInterceptor implements NestInterceptor {
  constructor(private reflector: Reflector) {}
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    // Get metadata from handler using context
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    const isPublic = this.reflector.get<boolean>('isPublic', context.getHandler());
    
    console.log('Roles:', roles);
    console.log('Is Public:', isPublic);
    
    return next.handle();
  }
}
```

**Conditional Execution:**
```typescript
@Injectable()
export class ConditionalInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    
    // Check condition before calling handler
    if (request.headers['x-skip-interceptor']) {
      // Skip transformation
      return next.handle();
    }
    
    // Apply transformation
    return next.handle().pipe(
      map(data => ({ wrapped: true, data })),
    );
  }
}
```

**Override Handler (Caching):**
```typescript
@Injectable()
export class CacheInterceptor implements NestInterceptor {
  private cache = new Map();
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const cacheKey = request.url;
    
    // Check cache
    if (this.cache.has(cacheKey)) {
      console.log('Cache hit!');
      // Don't call next.handle(), return cached data
      return of(this.cache.get(cacheKey));
    }
    
    // Cache miss, call handler
    return next.handle().pipe(
      tap(data => {
        console.log('Cache miss, storing...');
        this.cache.set(cacheKey, data);
      }),
    );
  }
}
```

**Interview Tip**: `intercept()` receives `context` (ExecutionContext with request info) and `next` (CallHandler to invoke handler). Use `context.switchToHttp().getRequest()` for request data. Call `next.handle()` to execute handler and get Observable. Can skip handler by returning custom Observable (caching).

</details>

### 5. What is RxJS and why is it used in Interceptors?

<details>
<summary>Answer</summary>

**RxJS** (Reactive Extensions for JavaScript) is a library for reactive programming using **Observables**. Interceptors use RxJS because they wrap handler execution and allow powerful stream transformations.

**Why RxJS in Interceptors:**
1. **Async Operations** - Handle promises and async data streams
2. **Composition** - Chain multiple operations with `.pipe()`
3. **Transformation** - Transform responses with operators
4. **Error Handling** - Catch and handle errors elegantly
5. **Timing** - Add timeouts, delays, retries
6. **Cancellation** - Cancel requests if needed

**Basic Observable:**
```typescript
import { Observable } from 'rxjs';
import { map, tap } from 'rxjs/operators';

@Injectable()
export class BasicInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    // next.handle() returns an Observable
    return next.handle().pipe(
      // Use RxJS operators to transform
      tap(() => console.log('After handler')),
      map(data => ({ success: true, data })),
    );
  }
}
```

**Common RxJS Operators:**
```typescript
import { map, tap, catchError, timeout, retry } from 'rxjs/operators';
import { throwError } from 'rxjs';

@Injectable()
export class RxJSInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      // tap: Side effects (logging) without modifying data
      tap(data => console.log('Response:', data)),
      
      // map: Transform the data
      map(data => ({ success: true, data })),
      
      // timeout: Cancel if takes too long
      timeout(5000),
      
      // retry: Retry on failure
      retry(3),
      
      // catchError: Handle errors
      catchError(err => {
        console.error('Error:', err);
        return throwError(() => new Error('Something went wrong'));
      }),
    );
  }
}
```

**Transform Response:**
```typescript
@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => {
        // Transform response
        return {
          statusCode: 200,
          data,
          timestamp: new Date().toISOString(),
        };
      }),
    );
  }
}

// Original: [{ id: 1, name: 'John' }]
// Transformed:
{
  "statusCode": 200,
  "data": [{ "id": 1, "name": "John" }],
  "timestamp": "2024-01-01T00:00:00.000Z"
}
```

**Measure Time:**
```typescript
@Injectable()
export class TimingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const start = Date.now();
    
    return next.handle().pipe(
      tap(() => {
        const duration = Date.now() - start;
        console.log(`Request took ${duration}ms`);
      }),
    );
  }
}
```

**Error Handling:**
```typescript
import { catchError } from 'rxjs/operators';
import { throwError } from 'rxjs';

@Injectable()
export class ErrorInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      catchError(err => {
        console.error('Caught error:', err);
        
        // Transform error
        return throwError(() => ({
          status: 500,
          message: 'Internal server error',
          timestamp: Date.now(),
        }));
      }),
    );
  }
}
```

**Timeout:**
```typescript
import { timeout, catchError } from 'rxjs/operators';
import { throwError, TimeoutError } from 'rxjs';

@Injectable()
export class TimeoutInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      timeout(5000), // 5 seconds
      catchError(err => {
        if (err instanceof TimeoutError) {
          return throwError(() => new RequestTimeoutException('Request timeout'));
        }
        return throwError(() => err);
      }),
    );
  }
}
```

**Interview Tip**: RxJS provides Observables for async stream handling. Interceptors use RxJS because `next.handle()` returns Observable. Use `.pipe()` with operators: `map()` (transform), `tap()` (side effects), `catchError()` (errors), `timeout()` (timing). Enables powerful composition and transformation.

</details>

### 6. Can Interceptors modify request and response?

<details>
<summary>Answer</summary>

Yes, Interceptors can modify both **request** (before handler) and **response** (after handler).

**Modify Request (Before Handler):**
```typescript
@Injectable()
export class RequestModifierInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    
    // Add custom headers
    request.headers['x-custom-header'] = 'custom-value';
    
    // Add timestamp
    request.timestamp = Date.now();
    
    // Add user info (if not exists)
    if (!request.user) {
      request.user = { id: 'anonymous' };
    }
    
    console.log('Modified request:', request.headers);
    
    return next.handle();
  }
}
```

**Modify Response (After Handler):**
```typescript
@Injectable()
export class ResponseModifierInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => {
        // Wrap response
        return {
          success: true,
          statusCode: 200,
          data,
          timestamp: new Date().toISOString(),
        };
      }),
    );
  }
}

// Original response: [{ id: 1, name: 'John' }]
// Modified response:
{
  "success": true,
  "statusCode": 200,
  "data": [{ "id": 1, "name": "John" }],
  "timestamp": "2024-01-01T00:00:00.000Z"
}
```

**Add Response Headers:**
```typescript
@Injectable()
export class HeaderInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const response = context.switchToHttp().getResponse();
    
    return next.handle().pipe(
      tap(() => {
        // Add custom headers to response
        response.setHeader('X-Response-Time', Date.now().toString());
        response.setHeader('X-Custom-Header', 'NestJS');
      }),
    );
  }
}
```

**Transform Data:**
```typescript
@Injectable()
export class DataTransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => {
        // Transform array of users
        if (Array.isArray(data)) {
          return data.map(item => ({
            ...item,
            fullName: `${item.firstName} ${item.lastName}`,
          }));
        }
        return data;
      }),
    );
  }
}
```

**Sanitize Response (Remove Sensitive Data):**
```typescript
@Injectable()
export class SanitizeInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => {
        // Remove password from response
        if (data && typeof data === 'object') {
          const { password, ...sanitized } = data;
          return sanitized;
        }
        return data;
      }),
    );
  }
}

// Original: { id: 1, email: 'john@example.com', password: 'secret123' }
// Sanitized: { id: 1, email: 'john@example.com' }
```

**Add Metadata:**
```typescript
@Injectable()
export class MetadataInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const start = Date.now();
    
    return next.handle().pipe(
      map(data => ({
        data,
        meta: {
          timestamp: new Date().toISOString(),
          path: request.url,
          method: request.method,
          duration: Date.now() - start,
        },
      })),
    );
  }
}

// Response:
{
  "data": [{ "id": 1, "name": "John" }],
  "meta": {
    "timestamp": "2024-01-01T00:00:00.000Z",
    "path": "/users",
    "method": "GET",
    "duration": 15
  }
}
```

**Pagination Wrapper:**
```typescript
@Injectable()
export class PaginationInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { page = 1, limit = 10 } = request.query;
    
    return next.handle().pipe(
      map(data => ({
        data: Array.isArray(data) ? data : [data],
        pagination: {
          page: Number(page),
          limit: Number(limit),
          total: Array.isArray(data) ? data.length : 1,
        },
      })),
    );
  }
}
```

**Interview Tip**: Modify **request** by accessing `context.switchToHttp().getRequest()` before calling `next.handle()`. Modify **response** by using RxJS `map()` operator on `next.handle().pipe()`. Can add headers, transform data, sanitize responses, add metadata. Request changes affect handler, response changes affect client.

</details>

## Creating Interceptors

### 7. How do you create an Interceptor implementing `NestInterceptor`?

<details>
<summary>Answer</summary>

Create a class with `@Injectable()` decorator that implements `NestInterceptor` interface.

**Basic Interceptor:**
```typescript
import { 
  Injectable, 
  NestInterceptor, 
  ExecutionContext, 
  CallHandler 
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('Before handler...');
    const now = Date.now();
    
    return next.handle().pipe(
      tap(() => console.log(`After handler... ${Date.now() - now}ms`)),
    );
  }
}
```

**Transform Response:**
```typescript
import { map } from 'rxjs/operators';

@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => ({
        success: true,
        data,
        timestamp: Date.now(),
      })),
    );
  }
}
```

**With Dependency Injection:**
```typescript
@Injectable()
export class LoggerInterceptor implements NestInterceptor {
  constructor(
    private readonly logger: LoggerService,
    private readonly configService: ConfigService,
  ) {}
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { method, url } = request;
    
    this.logger.log(`${method} ${url} - Start`);
    const start = Date.now();
    
    return next.handle().pipe(
      tap({
        next: () => {
          const duration = Date.now() - start;
          this.logger.log(`${method} ${url} - Complete (${duration}ms)`);
        },
        error: (err) => {
          this.logger.error(`${method} ${url} - Error: ${err.message}`);
        },
      }),
    );
  }
}
```

**Timeout Interceptor:**
```typescript
import { timeout, catchError } from 'rxjs/operators';
import { throwError, TimeoutError } from 'rxjs';
import { RequestTimeoutException } from '@nestjs/common';

@Injectable()
export class TimeoutInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      timeout(5000),
      catchError(err => {
        if (err instanceof TimeoutError) {
          return throwError(() => new RequestTimeoutException());
        }
        return throwError(() => err);
      }),
    );
  }
}
```

**Error Handling Interceptor:**
```typescript
import { catchError } from 'rxjs/operators';
import { throwError } from 'rxjs';

@Injectable()
export class ErrorInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      catchError(err => {
        console.error('Error caught by interceptor:', err);
        
        // Transform error
        return throwError(() => ({
          statusCode: err.status || 500,
          message: err.message || 'Internal server error',
          timestamp: new Date().toISOString(),
        }));
      }),
    );
  }
}
```

**Configurable Interceptor:**
```typescript
@Injectable()
export class CacheInterceptor implements NestInterceptor {
  constructor(private readonly ttl: number = 60000) {} // 60s default
  
  private cache = new Map<string, { data: any; timestamp: number }>();
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const cacheKey = request.url;
    
    // Check cache
    const cached = this.cache.get(cacheKey);
    if (cached && Date.now() - cached.timestamp < this.ttl) {
      return of(cached.data);
    }
    
    // Execute and cache
    return next.handle().pipe(
      tap(data => {
        this.cache.set(cacheKey, { data, timestamp: Date.now() });
      }),
    );
  }
}
```

**Usage:**
```typescript
// Route level
@Controller('users')
export class UsersController {
  @Get()
  @UseInterceptors(LoggingInterceptor)
  findAll() {
    return [];
  }
}

// Controller level
@Controller('posts')
@UseInterceptors(LoggingInterceptor, TransformInterceptor)
export class PostsController {}

// Global level
// main.ts
app.useGlobalInterceptors(new LoggingInterceptor());

// Module level
@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: LoggingInterceptor,
    },
  ],
})
export class AppModule {}
```

**Interview Tip**: Create Interceptor with `@Injectable()` and `implements NestInterceptor`. Implement `intercept(context, next)` returning `Observable`. Use `next.handle().pipe()` with RxJS operators. Can inject services via constructor. Apply with `@UseInterceptors()` or globally.

</details>

### 8. What is `ExecutionContext` in Interceptors?

<details>
<summary>Answer</summary>

`ExecutionContext` provides information about the current request, handler, and controller being executed.

**ExecutionContext Methods:**
```typescript
@Injectable()
export class ContextInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    // Get HTTP request/response
    const http = context.switchToHttp();
    const request = http.getRequest();
    const response = http.getResponse();
    
    // Get handler and class
    const handler = context.getHandler();
    const controller = context.getClass();
    
    // Get execution type
    const type = context.getType(); // 'http', 'ws', 'rpc'
    
    console.log('Handler name:', handler.name);
    console.log('Controller name:', controller.name);
    console.log('Type:', type);
    
    return next.handle();
  }
}
```

**Access Request Information:**
```typescript
@Injectable()
export class RequestInfoInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    
    console.log('URL:', request.url);
    console.log('Method:', request.method);
    console.log('Headers:', request.headers);
    console.log('Query:', request.query);
    console.log('Body:', request.body);
    console.log('Params:', request.params);
    console.log('User:', request.user);
    
    return next.handle();
  }
}
```

**Access Response Object:**
```typescript
@Injectable()
export class ResponseHeaderInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const response = context.switchToHttp().getResponse();
    
    return next.handle().pipe(
      tap(() => {
        // Set custom headers
        response.setHeader('X-Response-Time', Date.now().toString());
        response.setHeader('X-Powered-By', 'NestJS');
      }),
    );
  }
}
```

**Access Metadata:**
```typescript
import { Reflector } from '@nestjs/core';

@Injectable()
export class MetadataInterceptor implements NestInterceptor {
  constructor(private reflector: Reflector) {}
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    // Get metadata from handler
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    const isPublic = this.reflector.get<boolean>('isPublic', context.getHandler());
    
    // Get metadata from controller
    const controllerRoles = this.reflector.get<string[]>('roles', context.getClass());
    
    console.log('Handler roles:', roles);
    console.log('Is public:', isPublic);
    console.log('Controller roles:', controllerRoles);
    
    return next.handle();
  }
}
```

**Get Handler and Controller Names:**
```typescript
@Injectable()
export class NamingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const handlerName = context.getHandler().name;
    const controllerName = context.getClass().name;
    
    console.log(`${controllerName}.${handlerName}()`);
    
    return next.handle();
  }
}
```

**Different Context Types:**
```typescript
@Injectable()
export class UniversalInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const type = context.getType();
    
    if (type === 'http') {
      const request = context.switchToHttp().getRequest();
      console.log('HTTP Request:', request.url);
    } else if (type === 'ws') {
      const client = context.switchToWs().getClient();
      console.log('WebSocket Client:', client);
    } else if (type === 'rpc') {
      const data = context.switchToRpc().getData();
      console.log('RPC Data:', data);
    }
    
    return next.handle();
  }
}
```

**Interview Tip**: `ExecutionContext` provides request/response access via `switchToHttp()`, handler/controller references via `getHandler()`/`getClass()`, and metadata access. Use to inspect request details, add response headers, check metadata, or get handler information. Available in both Guards and Interceptors.

</details>

### 9. What is `CallHandler` and what does `handle()` do?

<details>
<summary>Answer</summary>

`CallHandler` is an interface with a `handle()` method that invokes the route handler and returns an `Observable`.

**CallHandler Interface:**
```typescript
export interface CallHandler<T = any> {
  handle(): Observable<T>;
}
```

**Basic Usage:**
```typescript
@Injectable()
export class BasicInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('Before calling handle()');
    
    // Call handle() to invoke the route handler
    const observable = next.handle();
    
    console.log('After calling handle() (but handler may not have executed yet)');
    
    // Return the observable (with or without transformations)
    return observable;
  }
}
```

**What `handle()` Does:**
1. Invokes the route handler method
2. Returns an Observable of the handler's response
3. Allows you to transform the response with RxJS operators

**Without Transformation:**
```typescript
@Injectable()
export class PassThroughInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    // Just call handle and return - no transformation
    return next.handle();
  }
}
```

**With Transformation:**
```typescript
@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    // Call handle() and transform its result
    return next.handle().pipe(
      map(data => ({
        success: true,
        data,
      })),
    );
  }
}
```

**Skip Handler (Caching):**
```typescript
@Injectable()
export class CacheInterceptor implements NestInterceptor {
  private cache = new Map();
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const cacheKey = request.url;
    
    // Check if cached
    if (this.cache.has(cacheKey)) {
      // Don't call next.handle() - return cached data instead
      return of(this.cache.get(cacheKey));
    }
    
    // Not cached - call handler and cache result
    return next.handle().pipe(
      tap(data => this.cache.set(cacheKey, data)),
    );
  }
}
```

**Timing:**
```typescript
@Injectable()
export class TimingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const start = Date.now();
    
    // handle() is called here
    return next.handle().pipe(
      // This runs after the handler completes
      tap(() => {
        const duration = Date.now() - start;
        console.log(`Handler took ${duration}ms`);
      }),
    );
  }
}
```

**Multiple Operations:**
```typescript
@Injectable()
export class MultiOpInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('Before handler');
    
    return next.handle().pipe(
      // Log response
      tap(data => console.log('Response:', data)),
      
      // Transform response
      map(data => ({ success: true, data })),
      
      // Handle errors
      catchError(err => {
        console.error('Error:', err);
        return throwError(() => err);
      }),
      
      // Add timeout
      timeout(5000),
    );
  }
}
```

**Conditional Execution:**
```typescript
@Injectable()
export class ConditionalInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    
    if (request.headers['x-skip-transform']) {
      // Call handler without transformation
      return next.handle();
    }
    
    // Call handler with transformation
    return next.handle().pipe(
      map(data => ({ wrapped: true, data })),
    );
  }
}
```

**Async Operations Before Handler:**
```typescript
@Injectable()
export class AsyncInterceptor implements NestInterceptor {
  constructor(private authService: AuthService) {}
  
  async intercept(
    context: ExecutionContext,
    next: CallHandler,
  ): Promise<Observable<any>> {
    // Do async work BEFORE calling handler
    const request = context.switchToHttp().getRequest();
    await this.authService.validateToken(request.headers.authorization);
    
    // Now call handler
    return next.handle().pipe(
      tap(() => console.log('Handler completed')),
    );
  }
}
```

**Interview Tip**: `CallHandler.handle()` invokes the route handler and returns Observable. Call it with `next.handle()` to execute handler. Use `.pipe()` with RxJS operators to transform response. Can skip handler by returning different Observable (caching). Handler doesn't execute until Observable is subscribed to.

</details>

## RxJS Operators in Interceptors

### 10. What are some common RxJS operators used in Interceptors?

<details>
<summary>Answer</summary>

Common RxJS operators for Interceptors include `tap()`, `map()`, `catchError()`, `timeout()`, `retry()`, `finalize()`, and `mergeMap()`.

**Common Operators Table:**

| Operator | Purpose | Modifies Data | Use Case |
|----------|---------|---------------|----------|
| `tap()` | Side effects | No | Logging, timing |
| `map()` | Transform data | Yes | Wrap responses |
| `catchError()` | Error handling | Yes | Transform errors |
| `timeout()` | Time limit | No | Request timeout |
| `retry()` | Retry on error | No | Fault tolerance |
| `finalize()` | Cleanup | No | Release resources |
| `mergeMap()` | Flatten observables | Yes | Async operations |

**tap() - Side Effects:**
```typescript
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const start = Date.now();
    
    return next.handle().pipe(
      tap({
        next: (data) => {
          console.log('Response:', data);
          console.log(`Duration: ${Date.now() - start}ms`);
        },
        error: (err) => {
          console.error('Error:', err.message);
        },
        complete: () => {
          console.log('Request completed');
        },
      }),
    );
  }
}
```

**map() - Transform Data:**
```typescript
import { map } from 'rxjs/operators';

@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => ({
        statusCode: 200,
        success: true,
        data,
        timestamp: new Date().toISOString(),
      })),
    );
  }
}
```

**catchError() - Error Handling:**
```typescript
import { catchError } from 'rxjs/operators';
import { throwError } from 'rxjs';

@Injectable()
export class ErrorInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      catchError(err => {
        console.error('Error caught:', err);
        return throwError(() => new HttpException(
          'Something went wrong',
          HttpStatus.INTERNAL_SERVER_ERROR,
        ));
      }),
    );
  }
}
```

**timeout() - Request Timeout:**
```typescript
import { timeout, catchError } from 'rxjs/operators';
import { throwError, TimeoutError } from 'rxjs';

@Injectable()
export class TimeoutInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      timeout(5000), // 5 seconds
      catchError(err => {
        if (err instanceof TimeoutError) {
          return throwError(() => new RequestTimeoutException('Request timeout'));
        }
        return throwError(() => err);
      }),
    );
  }
}
```

**retry() - Retry Failed Requests:**
```typescript
import { retry, catchError } from 'rxjs/operators';
import { throwError } from 'rxjs';

@Injectable()
export class RetryInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      retry(3), // Retry 3 times on error
      catchError(err => {
        console.error('Failed after 3 retries:', err);
        return throwError(() => err);
      }),
    );
  }
}
```

**finalize() - Cleanup:**
```typescript
import { finalize } from 'rxjs/operators';

@Injectable()
export class CleanupInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const start = Date.now();
    
    return next.handle().pipe(
      finalize(() => {
        // Always executed (success or error)
        console.log(`Request completed in ${Date.now() - start}ms`);
      }),
    );
  }
}
```

**Combined Operators:**
```typescript
import { tap, map, catchError, timeout, finalize } from 'rxjs/operators';
import { throwError, TimeoutError } from 'rxjs';

@Injectable()
export class FullInterceptor implements NestInterceptor {
  constructor(private logger: Logger) {}
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { method, url } = request;
    const start = Date.now();
    
    this.logger.log(`${method} ${url} - Start`);
    
    return next.handle().pipe(
      // Add timeout
      timeout(10000),
      
      // Log response
      tap(data => this.logger.log(`Response: ${JSON.stringify(data)}`)),
      
      // Transform response
      map(data => ({
        success: true,
        data,
        timestamp: new Date().toISOString(),
      })),
      
      // Handle errors
      catchError(err => {
        if (err instanceof TimeoutError) {
          this.logger.error('Request timeout');
          return throwError(() => new RequestTimeoutException());
        }
        this.logger.error(`Error: ${err.message}`);
        return throwError(() => err);
      }),
      
      // Cleanup
      finalize(() => {
        const duration = Date.now() - start;
        this.logger.log(`${method} ${url} - Completed (${duration}ms)`);
      }),
    );
  }
}
```

**Interview Tip**: Most common operators are `tap()` (logging without modification), `map()` (transform response), `catchError()` (error handling), `timeout()` (prevent long requests), and `finalize()` (cleanup). Chain operators with `.pipe()`. Order matters - transformations happen sequentially.

</details>

### 11. How do you use the `tap()` operator?

<details>
<summary>Answer</summary>

`tap()` performs side effects **without modifying** the data stream. Use for logging, metrics, debugging.

**Basic tap():**
```typescript
import { tap } from 'rxjs/operators';

@Injectable()
export class TapInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      tap(data => {
        // Side effect - doesn't modify data
        console.log('Response:', data);
      }),
    );
  }
}
```

**Logging with tap():**
```typescript
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  constructor(private logger: Logger) {}
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { method, url } = request;
    const start = Date.now();
    
    this.logger.log(`→ ${method} ${url}`);
    
    return next.handle().pipe(
      tap({
        next: (data) => {
          const duration = Date.now() - start;
          this.logger.log(`← ${method} ${url} - ${duration}ms`);
        },
        error: (err) => {
          const duration = Date.now() - start;
          this.logger.error(`✗ ${method} ${url} - ${duration}ms - ${err.message}`);
        },
      }),
    );
  }
}
```

**tap() with Callbacks:**
```typescript
@Injectable()
export class CallbackInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      tap({
        next: (value) => console.log('Next:', value),
        error: (err) => console.error('Error:', err),
        complete: () => console.log('Complete'),
      }),
    );
  }
}
```

**Metrics Collection:**
```typescript
@Injectable()
export class MetricsInterceptor implements NestInterceptor {
  constructor(private metricsService: MetricsService) {}
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const start = Date.now();
    
    return next.handle().pipe(
      tap({
        next: () => {
          const duration = Date.now() - start;
          this.metricsService.recordRequestDuration(
            request.method,
            request.url,
            duration,
          );
        },
        error: (err) => {
          this.metricsService.recordError(
            request.method,
            request.url,
            err.status || 500,
          );
        },
      }),
    );
  }
}
```

**Audit Logging:**
```typescript
@Injectable()
export class AuditInterceptor implements NestInterceptor {
  constructor(private auditService: AuditService) {}
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { user, method, url, body } = request;
    
    return next.handle().pipe(
      tap({
        next: (response) => {
          // Log successful operation
          this.auditService.log({
            userId: user?.id,
            action: `${method} ${url}`,
            input: body,
            output: response,
            timestamp: new Date(),
            status: 'success',
          });
        },
        error: (err) => {
          // Log failed operation
          this.auditService.log({
            userId: user?.id,
            action: `${method} ${url}`,
            input: body,
            error: err.message,
            timestamp: new Date(),
            status: 'failed',
          });
        },
      }),
    );
  }
}
```

**Debug Logging:**
```typescript
@Injectable()
export class DebugInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    
    console.log('═══════════════════════════════════════');
    console.log('REQUEST:', {
      method: request.method,
      url: request.url,
      headers: request.headers,
      query: request.query,
      body: request.body,
    });
    
    return next.handle().pipe(
      tap({
        next: (data) => {
          console.log('RESPONSE:', data);
          console.log('═══════════════════════════════════════');
        },
        error: (err) => {
          console.error('ERROR:', err.message);
          console.log('═══════════════════════════════════════');
        },
      }),
    );
  }
}
```

**tap() vs map():**
```typescript
@Injectable()
export class ComparisonInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      // tap() - doesn't modify data
      tap(data => {
        console.log('Original data:', data);
        data.modified = true; // DON'T DO THIS - side effects are bad
      }),
      
      // Data is still original (unless you mutated it - bad practice)
      tap(data => console.log('Still original:', data)),
      
      // map() - returns new data
      map(data => ({
        ...data,
        wrapped: true,
      })),
      
      // Data is now modified
      tap(data => console.log('Now modified:', data)),
    );
  }
}
```

**Interview Tip**: Use `tap()` for side effects like logging, metrics, debugging without modifying data. Takes callback or object with `next`, `error`, `complete`. Data passes through unchanged. Common for logging and monitoring. Don't mutate data inside tap() - use map() for transformations.

</details>

### 12. How do you use the `map()` operator?

<details>
<summary>Answer</summary>

`map()` transforms each value emitted by the Observable. Use for wrapping responses, adding fields, formatting data.

**Basic map():**
```typescript
import { map } from 'rxjs/operators';

@Injectable()
export class MapInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => {
        // Transform and return new value
        return {
          success: true,
          data,
        };
      }),
    );
  }
}

// Original: [{ id: 1, name: 'John' }]
// Transformed: { success: true, data: [{ id: 1, name: 'John' }] }
```

**Wrap Response:**
```typescript
@Injectable()
export class ResponseWrapperInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => ({
        statusCode: 200,
        success: true,
        data,
        timestamp: new Date().toISOString(),
        message: 'Request successful',
      })),
    );
  }
}

// Response:
{
  "statusCode": 200,
  "success": true,
  "data": { "id": 1, "name": "John" },
  "timestamp": "2024-01-01T00:00:00.000Z",
  "message": "Request successful"
}
```

**Add Metadata:**
```typescript
@Injectable()
export class MetadataInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const start = Date.now();
    
    return next.handle().pipe(
      map(data => ({
        data,
        meta: {
          timestamp: new Date().toISOString(),
          path: request.url,
          method: request.method,
          duration: Date.now() - start,
          version: '1.0.0',
        },
      })),
    );
  }
}
```

**Pagination Wrapper:**
```typescript
@Injectable()
export class PaginationInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { page = 1, limit = 10 } = request.query;
    
    return next.handle().pipe(
      map(data => {
        const items = Array.isArray(data) ? data : [data];
        
        return {
          data: items,
          pagination: {
            page: Number(page),
            limit: Number(limit),
            total: items.length,
            totalPages: Math.ceil(items.length / Number(limit)),
          },
        };
      }),
    );
  }
}
```

**Transform Array Items:**
```typescript
@Injectable()
export class ItemTransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => {
        if (Array.isArray(data)) {
          return data.map(item => ({
            ...item,
            fullName: `${item.firstName} ${item.lastName}`,
            createdAtFormatted: new Date(item.createdAt).toLocaleDateString(),
          }));
        }
        return data;
      }),
    );
  }
}
```

**Remove Sensitive Data:**
```typescript
@Injectable()
export class SanitizeInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => {
        if (Array.isArray(data)) {
          return data.map(item => this.sanitize(item));
        }
        return this.sanitize(data);
      }),
    );
  }
  
  private sanitize(data: any): any {
    if (!data || typeof data !== 'object') return data;
    
    const { password, secret, apiKey, ...sanitized } = data;
    return sanitized;
  }
}
```

**Conditional Transformation:**
```typescript
@Injectable()
export class ConditionalMapInterceptor implements NestInterceptor {
  constructor(private reflector: Reflector) {}
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const skipTransform = this.reflector.get<boolean>(
      'skipTransform',
      context.getHandler(),
    );
    
    return next.handle().pipe(
      map(data => {
        if (skipTransform) {
          return data; // Return original
        }
        
        return {
          success: true,
          data,
        };
      }),
    );
  }
}
```

**Format Dates:**
```typescript
@Injectable()
export class DateFormatInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => this.formatDates(data)),
    );
  }
  
  private formatDates(data: any): any {
    if (!data) return data;
    
    if (Array.isArray(data)) {
      return data.map(item => this.formatDates(item));
    }
    
    if (typeof data === 'object') {
      const formatted = {};
      for (const key in data) {
        if (data[key] instanceof Date) {
          formatted[key] = data[key].toISOString();
        } else if (typeof data[key] === 'object') {
          formatted[key] = this.formatDates(data[key]);
        } else {
          formatted[key] = data[key];
        }
      }
      return formatted;
    }
    
    return data;
  }
}
```

**Interview Tip**: Use `map()` to transform response data. Takes function that receives data and returns new value. Common uses: wrapping responses, adding metadata, sanitizing sensitive data, formatting fields. Unlike `tap()`, map() returns new data. Chain multiple map() operations for complex transformations.

</details>

### 13. How do you use the `catchError()` operator?

<details>
<summary>Answer</summary>

`catchError()` handles errors in the Observable stream and returns a new Observable or throws a new error.

**Basic catchError():**
```typescript
import { catchError } from 'rxjs/operators';
import { throwError } from 'rxjs';

@Injectable()
export class ErrorInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      catchError(err => {
        console.error('Error caught:', err.message);
        
        // Return new error
        return throwError(() => new HttpException(
          'Something went wrong',
          HttpStatus.INTERNAL_SERVER_ERROR,
        ));
      }),
    );
  }
}
```

**Transform Errors:**
```typescript
@Injectable()
export class ErrorTransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      catchError(err => {
        // Transform error to standard format
        const error = {
          statusCode: err.status || 500,
          message: err.message || 'Internal server error',
          timestamp: new Date().toISOString(),
          path: context.switchToHttp().getRequest().url,
        };
        
        return throwError(() => new HttpException(error, error.statusCode));
      }),
    );
  }
}
```

**Logging Errors:**
```typescript
@Injectable()
export class ErrorLoggingInterceptor implements NestInterceptor {
  constructor(private logger: Logger) {}
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    
    return next.handle().pipe(
      catchError(err => {
        this.logger.error(
          `Error in ${request.method} ${request.url}: ${err.message}`,
          err.stack,
        );
        
        // Re-throw the error
        return throwError(() => err);
      }),
    );
  }
}
```

**Handle Specific Errors:**
```typescript
@Injectable()
export class SpecificErrorInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      catchError(err => {
        if (err instanceof NotFoundException) {
          return throwError(() => new HttpException(
            'Resource not found',
            HttpStatus.NOT_FOUND,
          ));
        }
        
        if (err instanceof UnauthorizedException) {
          return throwError(() => new HttpException(
            'Unauthorized access',
            HttpStatus.UNAUTHORIZED,
          ));
        }
        
        // Default error handling
        return throwError(() => new HttpException(
          'Internal server error',
          HttpStatus.INTERNAL_SERVER_ERROR,
        ));
      }),
    );
  }
}
```

**Return Fallback Value:**
```typescript
import { of } from 'rxjs';

@Injectable()
export class FallbackInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      catchError(err => {
        console.error('Error, returning fallback:', err.message);
        
        // Return fallback value instead of error
        return of({
          success: false,
          message: 'Service unavailable',
          data: [],
        });
      }),
    );
  }
}
```

**Timeout Error Handling:**
```typescript
import { timeout, catchError } from 'rxjs/operators';
import { throwError, TimeoutError } from 'rxjs';

@Injectable()
export class TimeoutErrorInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      timeout(5000),
      catchError(err => {
        if (err instanceof TimeoutError) {
          return throwError(() => new RequestTimeoutException(
            'Request took too long to complete',
          ));
        }
        return throwError(() => err);
      }),
    );
  }
}
```

**Retry with Error Handling:**
```typescript
import { retry, catchError } from 'rxjs/operators';
import { throwError } from 'rxjs';

@Injectable()
export class RetryErrorInterceptor implements NestInterceptor {
  constructor(private logger: Logger) {}
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      retry(3),
      catchError(err => {
        this.logger.error(`Failed after 3 retries: ${err.message}`);
        
        return throwError(() => new HttpException(
          'Service temporarily unavailable',
          HttpStatus.SERVICE_UNAVAILABLE,
        ));
      }),
    );
  }
}
```

**Send Error Notifications:**
```typescript
@Injectable()
export class ErrorNotificationInterceptor implements NestInterceptor {
  constructor(
    private notificationService: NotificationService,
    private logger: Logger,
  ) {}
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    
    return next.handle().pipe(
      catchError(async (err) => {
        // Log error
        this.logger.error(`Error: ${err.message}`, err.stack);
        
        // Send notification for critical errors
        if (err.status >= 500) {
          await this.notificationService.sendAlert({
            type: 'error',
            message: err.message,
            url: request.url,
            method: request.method,
            timestamp: new Date(),
          });
        }
        
        return throwError(() => err);
      }),
    );
  }
}
```

**Interview Tip**: Use `catchError()` to handle errors in Observable stream. Takes callback receiving error and must return Observable (use `throwError()` or `of()`). Can transform errors, log them, return fallback values, or re-throw. Place after operations that might fail. Use with `timeout()` and `retry()` for robust error handling.

</details>

### 14. What's the difference between `tap()` and `map()`?

<details>
<summary>Answer</summary>

**tap()** performs side effects **without modifying** data. **map()** transforms and **returns new** data.

**Comparison Table:**

| Feature | tap() | map() |
|---------|-------|-------|
| Modifies data | ❌ No | ✅ Yes |
| Return value | Ignored | Becomes new value |
| Use case | Logging, side effects | Transforming data |
| Data flow | Pass through | Transformed |

**tap() Example:**
```typescript
@Injectable()
export class TapExample implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      tap(data => {
        console.log('Logging data:', data);
        // Return value is ignored
        return { modified: true }; // This is ignored!
      }),
      // Data is still original
      tap(data => console.log('Still original:', data)),
    );
  }
}
```

**map() Example:**
```typescript
@Injectable()
export class MapExample implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => {
        console.log('Original data:', data);
        // Return value becomes new data
        return { modified: true, data };
      }),
      // Data is now modified
      tap(data => console.log('Modified:', data)),
    );
  }
}
```

**Side-by-Side Comparison:**
```typescript
@Injectable()
export class ComparisonInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      // tap() - side effects only
      tap(data => {
        console.log('1. Original:', data);
        // { id: 1, name: 'John' }
      }),
      
      // map() - transform data
      map(data => ({
        ...data,
        timestamp: Date.now(),
      })),
      
      // tap() - log transformed data
      tap(data => {
        console.log('2. After map:', data);
        // { id: 1, name: 'John', timestamp: 1234567890 }
      }),
      
      // map() - wrap in response object
      map(data => ({
        success: true,
        data,
      })),
      
      // tap() - final logging
      tap(data => {
        console.log('3. Final:', data);
        // { success: true, data: { id: 1, name: 'John', timestamp: 1234567890 } }
      }),
    );
  }
}
```

**When to Use tap():**
```typescript
@Injectable()
export class UseTapInterceptor implements NestInterceptor {
  constructor(
    private logger: Logger,
    private metricsService: MetricsService,
  ) {}
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const start = Date.now();
    
    return next.handle().pipe(
      // Logging - tap()
      tap(data => this.logger.log('Response received')),
      
      // Metrics - tap()
      tap(() => {
        const duration = Date.now() - start;
        this.metricsService.record(duration);
      }),
      
      // Debugging - tap()
      tap(data => console.log('Debug:', data)),
      
      // Analytics - tap()
      tap(data => {
        if (Array.isArray(data)) {
          this.metricsService.recordCount(data.length);
        }
      }),
    );
  }
}
```

**When to Use map():**
```typescript
@Injectable()
export class UseMapInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      // Wrap response - map()
      map(data => ({
        success: true,
        data,
      })),
      
      // Add timestamp - map()
      map(response => ({
        ...response,
        timestamp: new Date().toISOString(),
      })),
      
      // Transform array - map()
      map(response => ({
        ...response,
        data: Array.isArray(response.data)
          ? response.data.map(item => ({ ...item, processed: true }))
          : response.data,
      })),
    );
  }
}
```

**Common Mistake:**
```typescript
@Injectable()
export class MistakeInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      // ❌ WRONG: Using tap() to modify data
      tap(data => {
        data.modified = true; // Mutates original object!
      }),
      
      // ✅ CORRECT: Use map() to transform
      map(data => ({
        ...data,
        modified: true,
      })),
    );
  }
}
```

**Combined Usage:**
```typescript
@Injectable()
export class CombinedInterceptor implements NestInterceptor {
  constructor(private logger: Logger) {}
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const start = Date.now();
    
    return next.handle().pipe(
      // Log original response - tap()
      tap(data => this.logger.log('Original:', data)),
      
      // Transform response - map()
      map(data => ({
        success: true,
        data,
        timestamp: Date.now(),
      })),
      
      // Log transformed response - tap()
      tap(response => this.logger.log('Transformed:', response)),
      
      // Record metrics - tap()
      tap(() => {
        const duration = Date.now() - start;
        this.logger.log(`Duration: ${duration}ms`);
      }),
    );
  }
}
```

**Interview Tip**: **tap()** for side effects (logging, metrics) without changing data. **map()** for transformations (wrapping, formatting) that return new data. tap() return value is ignored, map() return value becomes stream value. Use tap() for "observe", map() for "transform". Can chain both in same pipe().

</details>

## Applying Interceptors

### 15. How do you apply an Interceptor using `@UseInterceptors()`?

<details>
<summary>Answer</summary>

Use `@UseInterceptors()` decorator to apply interceptors at **method**, **controller**, or **global** level.

**Method Level:**
```typescript
import { UseInterceptors } from '@nestjs/common';

@Controller('users')
export class UsersController {
  @Get()
  @UseInterceptors(LoggingInterceptor)
  findAll() {
    return [];
  }
  
  @Post()
  @UseInterceptors(TransformInterceptor, ValidationInterceptor)
  create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }
}
```

**Controller Level:**
```typescript
@Controller('posts')
@UseInterceptors(LoggingInterceptor, TransformInterceptor)
export class PostsController {
  // All methods in this controller use these interceptors
  
  @Get()
  findAll() {
    return [];
  }
  
  @Get(':id')
  findOne(@Param('id') id: string) {
    return {};
  }
}
```

**Multiple Interceptors:**
```typescript
@Controller('products')
@UseInterceptors(
  LoggingInterceptor,
  CacheInterceptor,
  TransformInterceptor,
)
export class ProductsController {
  @Get()
  @UseInterceptors(PaginationInterceptor)
  findAll() {
    // Executes: Logging → Cache → Transform → Pagination
    return [];
  }
}
```

**With Configuration:**
```typescript
@Injectable()
export class CustomInterceptor implements NestInterceptor {
  constructor(private readonly option: string) {}
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('Option:', this.option);
    return next.handle();
  }
}

@Controller('items')
export class ItemsController {
  @Get()
  @UseInterceptors(new CustomInterceptor('value'))
  findAll() {
    return [];
  }
}
```

**Global Application (main.ts):**
```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Apply to all routes
  app.useGlobalInterceptors(
    new LoggingInterceptor(),
    new TransformInterceptor(),
  );
  
  await app.listen(3000);
}
bootstrap();
```

**Module Level with DI:**
```typescript
@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: LoggingInterceptor,
    },
    {
      provide: APP_INTERCEPTOR,
      useClass: TransformInterceptor,
    },
  ],
})
export class AppModule {}
```

**Conditional Interceptor:**
```typescript
@Controller('data')
export class DataController {
  @Get()
  @UseInterceptors(CacheInterceptor)
  @CacheTTL(60) // Cache for 60 seconds
  findAll() {
    return [];
  }
  
  @Get('fresh')
  // No cache interceptor
  getFresh() {
    return [];
  }
}
```

**Class Instance vs Class Reference:**
```typescript
@Controller('examples')
export class ExamplesController {
  // Class reference - NestJS creates instance with DI
  @Get('method1')
  @UseInterceptors(LoggingInterceptor)
  method1() {
    return 'method1';
  }
  
  // Class instance - manual instantiation
  @Get('method2')
  @UseInterceptors(new LoggingInterceptor())
  method2() {
    return 'method2';
  }
}
```

**Custom Decorator:**
```typescript
// custom-interceptors.decorator.ts
export const UseResponseTransform = () => 
  UseInterceptors(TransformInterceptor, LoggingInterceptor);

@Controller('custom')
export class CustomController {
  @Get()
  @UseResponseTransform()
  findAll() {
    return [];
  }
}
```

**Interview Tip**: Apply interceptors with `@UseInterceptors()` at method/controller level or globally with `app.useGlobalInterceptors()` or `APP_INTERCEPTOR` token. Can pass class reference (with DI) or instance. Multiple interceptors execute in order. Controller-level applies to all methods. Method-level runs after controller-level.

</details>

### 16. How do you apply Interceptors globally?

<details>
<summary>Answer</summary>

Apply Interceptors globally using `app.useGlobalInterceptors()` or `APP_INTERCEPTOR` provider token.

**Method 1: useGlobalInterceptors() in main.ts:**
```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Apply global interceptors
  app.useGlobalInterceptors(
    new LoggingInterceptor(),
    new TransformInterceptor(),
    new ErrorInterceptor(),
  );
  
  await app.listen(3000);
}
bootstrap();
```

**Method 2: APP_INTERCEPTOR Provider (Recommended):**
```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { APP_INTERCEPTOR } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: LoggingInterceptor,
    },
    {
      provide: APP_INTERCEPTOR,
      useClass: TransformInterceptor,
    },
    {
      provide: APP_INTERCEPTOR,
      useClass: ErrorInterceptor,
    },
  ],
})
export class AppModule {}
```

**With Dependency Injection:**
```typescript
// logging.interceptor.ts
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  constructor(
    private logger: Logger,
    private configService: ConfigService,
  ) {}
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const isDebug = this.configService.get('DEBUG');
    if (isDebug) {
      this.logger.log('Request received');
    }
    return next.handle();
  }
}

// app.module.ts
@Module({
  providers: [
    Logger,
    ConfigService,
    {
      provide: APP_INTERCEPTOR,
      useClass: LoggingInterceptor, // DI works!
    },
  ],
})
export class AppModule {}
```

**Multiple Global Interceptors:**
```typescript
@Module({
  providers: [
    // Execution order: 1, 2, 3
    {
      provide: APP_INTERCEPTOR,
      useClass: LoggingInterceptor,
    },
    {
      provide: APP_INTERCEPTOR,
      useClass: CacheInterceptor,
    },
    {
      provide: APP_INTERCEPTOR,
      useClass: TransformInterceptor,
    },
  ],
})
export class AppModule {}
```

**Conditional Global Interceptor:**
```typescript
// interceptors.module.ts
@Module({})
export class InterceptorsModule {
  static forRoot(options: { enableLogging: boolean; enableCache: boolean }): DynamicModule {
    const providers = [];
    
    if (options.enableLogging) {
      providers.push({
        provide: APP_INTERCEPTOR,
        useClass: LoggingInterceptor,
      });
    }
    
    if (options.enableCache) {
      providers.push({
        provide: APP_INTERCEPTOR,
        useClass: CacheInterceptor,
      });
    }
    
    return {
      module: InterceptorsModule,
      providers,
    };
  }
}

// app.module.ts
@Module({
  imports: [
    InterceptorsModule.forRoot({
      enableLogging: true,
      enableCache: false,
    }),
  ],
})
export class AppModule {}
```

**Environment-Based:**
```typescript
@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: LoggingInterceptor,
    },
    {
      provide: APP_INTERCEPTOR,
      useFactory: (configService: ConfigService) => {
        const env = configService.get('NODE_ENV');
        if (env === 'production') {
          return new PerformanceInterceptor();
        }
        return new DebugInterceptor();
      },
      inject: [ConfigService],
    },
  ],
})
export class AppModule {}
```

**Separate Module:**
```typescript
// common/common.module.ts
@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: LoggingInterceptor,
    },
    {
      provide: APP_INTERCEPTOR,
      useClass: TransformInterceptor,
    },
  ],
})
export class CommonModule {}

// app.module.ts
@Module({
  imports: [CommonModule],
})
export class AppModule {}
```

**Comparison:**

| Method | Dependency Injection | Use Case |
|--------|---------------------|----------|
| `useGlobalInterceptors()` | ❌ No | Simple, no DI needed |
| `APP_INTERCEPTOR` | ✅ Yes | Complex, needs services |

**Interview Tip**: Two methods for global interceptors: `app.useGlobalInterceptors()` in main.ts (no DI) or `APP_INTERCEPTOR` provider in module (with DI). Prefer `APP_INTERCEPTOR` when interceptor needs dependencies. Global interceptors run before controller/method interceptors. Can register multiple with separate providers.

</details>

### 17. What is the execution order of multiple Interceptors?

<details>
<summary>Answer</summary>

Interceptors execute in **FIFO (First In First Out)** order going **in**, and **LIFO (Last In First Out)** order coming **out** (like onion layers).

**Execution Flow:**
```
Request
  ↓
[Global Interceptor 1] ─────┐ Before
  ↓                          │
[Global Interceptor 2] ─────┤
  ↓                          │
[Controller Interceptor] ───┤
  ↓                          │
[Method Interceptor] ───────┘
  ↓
[Route Handler]
  ↓
[Method Interceptor] ───────┐ After
  ↓                          │
[Controller Interceptor] ───┤
  ↓                          │
[Global Interceptor 2] ─────┤
  ↓                          │
[Global Interceptor 1] ─────┘
  ↓
Response
```

**Example with Logging:**
```typescript
// Global
@Injectable()
export class GlobalInterceptor1 implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('1. Global 1 - Before');
    return next.handle().pipe(
      tap(() => console.log('8. Global 1 - After')),
    );
  }
}

@Injectable()
export class GlobalInterceptor2 implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('2. Global 2 - Before');
    return next.handle().pipe(
      tap(() => console.log('7. Global 2 - After')),
    );
  }
}

// Controller
@Injectable()
export class ControllerInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('3. Controller - Before');
    return next.handle().pipe(
      tap(() => console.log('6. Controller - After')),
    );
  }
}

// Method
@Injectable()
export class MethodInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('4. Method - Before');
    return next.handle().pipe(
      tap(() => console.log('5. Method - After')),
    );
  }
}

// Usage
// main.ts
app.useGlobalInterceptors(
  new GlobalInterceptor1(),
  new GlobalInterceptor2(),
);

// controller
@Controller('test')
@UseInterceptors(ControllerInterceptor)
export class TestController {
  @Get()
  @UseInterceptors(MethodInterceptor)
  test() {
    console.log('Handler executing');
    return 'done';
  }
}

// Console output:
// 1. Global 1 - Before
// 2. Global 2 - Before
// 3. Controller - Before
// 4. Method - Before
// Handler executing
// 5. Method - After
// 6. Controller - After
// 7. Global 2 - After
// 8. Global 1 - After
```

**Multiple Method-Level Interceptors:**
```typescript
@Controller('users')
export class UsersController {
  @Get()
  @UseInterceptors(
    LoggingInterceptor,    // 1st before, 3rd after
    CacheInterceptor,      // 2nd before, 2nd after
    TransformInterceptor,  // 3rd before, 1st after
  )
  findAll() {
    return [];
  }
}

// Execution:
// 1. LoggingInterceptor - Before
// 2. CacheInterceptor - Before
// 3. TransformInterceptor - Before
// Handler
// 3. TransformInterceptor - After
// 2. CacheInterceptor - After
// 1. LoggingInterceptor - After
```

**Visual Diagram:**
```typescript
// Setup
app.useGlobalInterceptors(A, B);

@Controller()
@UseInterceptors(C)
class Controller {
  @Get()
  @UseInterceptors(D, E)
  handler() {}
}

// Execution order:
Request
  → A before
    → B before
      → C before
        → D before
          → E before
            → Handler
          ← E after
        ← D after
      ← C after
    ← B after
  ← A after
Response
```

**Transformation Chain:**
```typescript
@Injectable()
export class Step1Interceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => ({ step1: true, data })),
    );
  }
}

@Injectable()
export class Step2Interceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => ({ step2: true, data })),
    );
  }
}

@Controller('transform')
export class TransformController {
  @Get()
  @UseInterceptors(Step1Interceptor, Step2Interceptor)
  test() {
    return { original: true };
  }
}

// Handler returns: { original: true }
// After Step2: { step2: true, data: { original: true } }
// After Step1: { step1: true, data: { step2: true, data: { original: true } } }
```

**Timing Example:**
```typescript
@Injectable()
export class Interceptor1 implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const start = Date.now();
    console.log('Int1 Start:', start);
    
    return next.handle().pipe(
      tap(() => {
        console.log('Int1 End:', Date.now() - start, 'ms');
      }),
    );
  }
}

@Injectable()
export class Interceptor2 implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const start = Date.now();
    console.log('Int2 Start:', start);
    
    return next.handle().pipe(
      tap(() => {
        console.log('Int2 End:', Date.now() - start, 'ms');
      }),
    );
  }
}

@Get()
@UseInterceptors(Interceptor1, Interceptor2)
test() {
  console.log('Handler');
  return 'done';
}

// Output:
// Int1 Start: 1000
// Int2 Start: 1000
// Handler
// Int2 End: 5 ms
// Int1 End: 5 ms
```

**Interview Tip**: Interceptors execute in order: Global → Controller → Method going **in**, then reverse order coming **out**. Think of it like onion layers or middleware stack. First interceptor wraps all others. Transformations happen in **reverse order** (last interceptor transforms first). Use this for layered operations like logging (outer) and transformation (inner).

</details>

### 18. What is `APP_INTERCEPTOR` token?

<details>
<summary>Answer</summary>

`APP_INTERCEPTOR` is a special provider token for registering **global interceptors** with **dependency injection** support.

**Basic Usage:**
```typescript
import { APP_INTERCEPTOR } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: LoggingInterceptor,
    },
  ],
})
export class AppModule {}
```

**With Dependencies:**
```typescript
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  constructor(
    private logger: Logger,
    private configService: ConfigService,
  ) {}
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const isDebug = this.configService.get('DEBUG');
    if (isDebug) {
      this.logger.log('Request received');
    }
    return next.handle();
  }
}

@Module({
  providers: [
    Logger,
    ConfigService,
    {
      provide: APP_INTERCEPTOR,
      useClass: LoggingInterceptor, // DI works!
    },
  ],
})
export class AppModule {}
```

**Multiple Interceptors:**
```typescript
@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: LoggingInterceptor,
    },
    {
      provide: APP_INTERCEPTOR,
      useClass: TransformInterceptor,
    },
    {
      provide: APP_INTERCEPTOR,
      useClass: ErrorInterceptor,
    },
  ],
})
export class AppModule {}
```

**With useFactory:**
```typescript
@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useFactory: (configService: ConfigService) => {
        const timeout = configService.get('REQUEST_TIMEOUT');
        return new TimeoutInterceptor(timeout);
      },
      inject: [ConfigService],
    },
  ],
})
export class AppModule {}
```

**Conditional Registration:**
```typescript
@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useFactory: (configService: ConfigService) => {
        const env = configService.get('NODE_ENV');
        
        if (env === 'production') {
          return new ProductionInterceptor();
        } else if (env === 'development') {
          return new DevelopmentInterceptor();
        } else {
          return new DefaultInterceptor();
        }
      },
      inject: [ConfigService],
    },
  ],
})
export class AppModule {}
```

**In Shared Module:**
```typescript
// common/common.module.ts
@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: LoggingInterceptor,
    },
    {
      provide: APP_INTERCEPTOR,
      useClass: TransformInterceptor,
    },
  ],
  exports: [], // No need to export APP_INTERCEPTOR
})
export class CommonModule {}

// app.module.ts
@Module({
  imports: [CommonModule], // Automatically registers global interceptors
})
export class AppModule {}
```

**Dynamic Module:**
```typescript
@Module({})
export class InterceptorsModule {
  static register(options: InterceptorsOptions): DynamicModule {
    const providers = [];
    
    if (options.logging) {
      providers.push({
        provide: APP_INTERCEPTOR,
        useClass: LoggingInterceptor,
      });
    }
    
    if (options.transform) {
      providers.push({
        provide: APP_INTERCEPTOR,
        useClass: TransformInterceptor,
      });
    }
    
    if (options.cache) {
      providers.push({
        provide: APP_INTERCEPTOR,
        useFactory: (cacheService: CacheService) => {
          return new CacheInterceptor(cacheService);
        },
        inject: [CacheService],
      });
    }
    
    return {
      module: InterceptorsModule,
      providers,
    };
  }
}

// Usage
@Module({
  imports: [
    InterceptorsModule.register({
      logging: true,
      transform: true,
      cache: false,
    }),
  ],
})
export class AppModule {}
```

**vs useGlobalInterceptors:**

| Feature | APP_INTERCEPTOR | useGlobalInterceptors() |
|---------|-----------------|-------------------------|
| Location | Module providers | main.ts |
| Dependency Injection | ✅ Yes | ❌ No |
| Module scope | Module | Application |
| Testing | Easy to override | Harder to test |

**Example Comparison:**
```typescript
// ❌ useGlobalInterceptors - No DI
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Can't inject dependencies
  app.useGlobalInterceptors(new LoggingInterceptor());
  
  await app.listen(3000);
}

// ✅ APP_INTERCEPTOR - With DI
@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: LoggingInterceptor, // Can inject Logger, ConfigService, etc.
    },
  ],
})
export class AppModule {}
```

**Testing:**
```typescript
describe('AppModule', () => {
  it('should override APP_INTERCEPTOR', async () => {
    const module = await Test.createTestingModule({
      imports: [AppModule],
    })
      .overrideProvider(APP_INTERCEPTOR)
      .useClass(MockInterceptor)
      .compile();
    
    // Test with mock interceptor
  });
});
```

**Interview Tip**: `APP_INTERCEPTOR` is token for registering global interceptors with DI support. Use in module providers array. Supports `useClass`, `useFactory`, `useValue`. Better than `useGlobalInterceptors()` when interceptor needs dependencies. Can register multiple by repeating token. Easier to test and override.

</details>

## RxJS Operators

10. What are the most common RxJS operators used in Interceptors?
11. What is `tap()` operator used for?
12. What is `map()` operator used for?
13. What is `catchError()` operator used for?
14. What is the difference between `tap()` and `map()`?

## Common Use Cases

### 19. How do you implement a logging Interceptor to measure execution time?

<details>
<summary>Answer</summary>

Create an interceptor that logs request details and measures handler execution time using timestamps.

**Basic Timing Interceptor:**
```typescript
import { Injectable, NestInterceptor, ExecutionContext, CallHandler, Logger } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  private readonly logger = new Logger(LoggingInterceptor.name);
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { method, url } = request;
    const start = Date.now();
    
    this.logger.log(`→ ${method} ${url}`);
    
    return next.handle().pipe(
      tap(() => {
        const duration = Date.now() - start;
        this.logger.log(`← ${method} ${url} - ${duration}ms`);
      }),
    );
  }
}
```

**Detailed Logging with Status Code:**
```typescript
@Injectable()
export class DetailedLoggingInterceptor implements NestInterceptor {
  private readonly logger = new Logger(DetailedLoggingInterceptor.name);
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const response = context.switchToHttp().getResponse();
    const { method, url, ip, headers } = request;
    const userAgent = headers['user-agent'] || '';
    const start = Date.now();
    
    this.logger.log(`→ ${method} ${url} from ${ip}`);
    
    return next.handle().pipe(
      tap({
        next: () => {
          const duration = Date.now() - start;
          const statusCode = response.statusCode;
          this.logger.log(
            `← ${method} ${url} ${statusCode} - ${duration}ms`,
          );
        },
        error: (err) => {
          const duration = Date.now() - start;
          this.logger.error(
            `✗ ${method} ${url} ${err.status || 500} - ${duration}ms - ${err.message}`,
          );
        },
      }),
    );
  }
}
```

**With Request ID:**
```typescript
import { v4 as uuidv4 } from 'uuid';

@Injectable()
export class RequestIdLoggingInterceptor implements NestInterceptor {
  private readonly logger = new Logger(RequestIdLoggingInterceptor.name);
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const response = context.switchToHttp().getResponse();
    const requestId = uuidv4();
    const { method, url } = request;
    const start = Date.now();
    
    // Add request ID to request and response
    request.requestId = requestId;
    response.setHeader('X-Request-Id', requestId);
    
    this.logger.log(`[${requestId}] → ${method} ${url}`);
    
    return next.handle().pipe(
      tap({
        next: () => {
          const duration = Date.now() - start;
          this.logger.log(`[${requestId}] ← ${method} ${url} - ${duration}ms`);
        },
        error: (err) => {
          const duration = Date.now() - start;
          this.logger.error(
            `[${requestId}] ✗ ${method} ${url} - ${duration}ms - ${err.message}`,
          );
        },
      }),
    );
  }
}
```

**Color-Coded Logging:**
```typescript
@Injectable()
export class ColorLoggingInterceptor implements NestInterceptor {
  private readonly logger = new Logger(ColorLoggingInterceptor.name);
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { method, url } = request;
    const start = Date.now();
    
    return next.handle().pipe(
      tap({
        next: () => {
          const duration = Date.now() - start;
          const color = this.getColorByDuration(duration);
          this.logger.log(`${method} ${url} - ${duration}ms ${color}`);
        },
        error: (err) => {
          const duration = Date.now() - start;
          this.logger.error(`${method} ${url} - ${duration}ms - ERROR: ${err.message}`);
        },
      }),
    );
  }
  
  private getColorByDuration(duration: number): string {
    if (duration < 100) return '🟢'; // Fast
    if (duration < 500) return '🟡'; // Medium
    if (duration < 1000) return '🟠'; // Slow
    return '🔴'; // Very slow
  }
}
```

**Structured Logging (JSON):**
```typescript
@Injectable()
export class StructuredLoggingInterceptor implements NestInterceptor {
  private readonly logger = new Logger(StructuredLoggingInterceptor.name);
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const response = context.switchToHttp().getResponse();
    const { method, url, ip, body, query, params, user } = request;
    const start = Date.now();
    
    const logData = {
      timestamp: new Date().toISOString(),
      method,
      url,
      ip,
      query,
      params,
      userId: user?.id,
    };
    
    this.logger.log(JSON.stringify({ ...logData, event: 'request_start' }));
    
    return next.handle().pipe(
      tap({
        next: (data) => {
          const duration = Date.now() - start;
          this.logger.log(
            JSON.stringify({
              ...logData,
              event: 'request_complete',
              duration,
              statusCode: response.statusCode,
              responseSize: JSON.stringify(data).length,
            }),
          );
        },
        error: (err) => {
          const duration = Date.now() - start;
          this.logger.error(
            JSON.stringify({
              ...logData,
              event: 'request_error',
              duration,
              statusCode: err.status || 500,
              error: err.message,
            }),
          );
        },
      }),
    );
  }
}
```

**Performance Monitoring:**
```typescript
@Injectable()
export class PerformanceInterceptor implements NestInterceptor {
  private readonly logger = new Logger(PerformanceInterceptor.name);
  private readonly slowThreshold = 1000; // 1 second
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { method, url } = request;
    const start = Date.now();
    
    return next.handle().pipe(
      tap(() => {
        const duration = Date.now() - start;
        
        if (duration > this.slowThreshold) {
          this.logger.warn(
            `⚠️ SLOW REQUEST: ${method} ${url} took ${duration}ms`,
          );
        } else {
          this.logger.log(`${method} ${url} - ${duration}ms`);
        }
      }),
    );
  }
}
```

**With External Service Integration:**
```typescript
@Injectable()
export class MonitoringInterceptor implements NestInterceptor {
  constructor(
    private readonly logger: Logger,
    private readonly metricsService: MetricsService,
  ) {}
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { method, url } = request;
    const start = Date.now();
    
    return next.handle().pipe(
      tap({
        next: () => {
          const duration = Date.now() - start;
          
          // Log to console
          this.logger.log(`${method} ${url} - ${duration}ms`);
          
          // Send to monitoring service
          this.metricsService.recordRequestDuration(method, url, duration);
          this.metricsService.incrementRequestCount(method, url);
        },
        error: (err) => {
          const duration = Date.now() - start;
          
          this.logger.error(`${method} ${url} - ${duration}ms - ${err.message}`);
          
          // Send error metrics
          this.metricsService.recordError(method, url, err.status || 500);
        },
      }),
    );
  }
}
```

**Interview Tip**: Measure execution time by recording timestamp before `next.handle()` and calculating duration in `tap()` operator. Use `tap()` with object parameter to handle both success and error cases. Can add request ID, log structured data, integrate with monitoring services. Common pattern for performance tracking.

</details>

### 20. How do you transform responses using `map()` operator?

<details>
<summary>Answer</summary>

Use `map()` operator to transform responses into desired format, wrap data, or add metadata.

**Basic Response Wrapper:**
```typescript
import { map } from 'rxjs/operators';

@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => ({
        success: true,
        data,
      })),
    );
  }
}

// Original: { id: 1, name: 'John' }
// Transformed: { success: true, data: { id: 1, name: 'John' } }
```

**Add Timestamp:**
```typescript
@Injectable()
export class TimestampInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => ({
        ...data,
        timestamp: new Date().toISOString(),
        serverTime: Date.now(),
      })),
    );
  }
}
```

**Wrap with Status Code:**
```typescript
@Injectable()
export class StatusWrapperInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const response = context.switchToHttp().getResponse();
    
    return next.handle().pipe(
      map(data => ({
        statusCode: response.statusCode,
        success: true,
        data,
        message: 'Request successful',
      })),
    );
  }
}
```

**Add Pagination:**
```typescript
@Injectable()
export class PaginationTransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { page = 1, limit = 10 } = request.query;
    
    return next.handle().pipe(
      map(data => {
        const items = Array.isArray(data) ? data : [];
        const total = items.length;
        
        return {
          data: items,
          pagination: {
            page: Number(page),
            limit: Number(limit),
            total,
            totalPages: Math.ceil(total / Number(limit)),
            hasMore: total > Number(page) * Number(limit),
          },
        };
      }),
    );
  }
}
```

**Array Transformation:**
```typescript
@Injectable()
export class ArrayTransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => {
        if (Array.isArray(data)) {
          return data.map(item => ({
            ...item,
            fullName: `${item.firstName} ${item.lastName}`,
            age: this.calculateAge(item.birthDate),
          }));
        }
        return data;
      }),
    );
  }
  
  private calculateAge(birthDate: Date): number {
    const today = new Date();
    const birth = new Date(birthDate);
    let age = today.getFullYear() - birth.getFullYear();
    const monthDiff = today.getMonth() - birth.getMonth();
    
    if (monthDiff < 0 || (monthDiff === 0 && today.getDate() < birth.getDate())) {
      age--;
    }
    
    return age;
  }
}
```

**Camel Case to Snake Case:**
```typescript
@Injectable()
export class CaseTransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => this.transformKeys(data)),
    );
  }
  
  private transformKeys(data: any): any {
    if (Array.isArray(data)) {
      return data.map(item => this.transformKeys(item));
    }
    
    if (data && typeof data === 'object' && !(data instanceof Date)) {
      return Object.keys(data).reduce((result, key) => {
        const snakeKey = key.replace(/([A-Z])/g, '_$1').toLowerCase();
        result[snakeKey] = this.transformKeys(data[key]);
        return result;
      }, {});
    }
    
    return data;
  }
}
```

**Remove Null/Undefined:**
```typescript
@Injectable()
export class CleanupInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => this.removeEmpty(data)),
    );
  }
  
  private removeEmpty(data: any): any {
    if (Array.isArray(data)) {
      return data.map(item => this.removeEmpty(item));
    }
    
    if (data && typeof data === 'object') {
      return Object.keys(data).reduce((result, key) => {
        const value = data[key];
        if (value !== null && value !== undefined) {
          result[key] = this.removeEmpty(value);
        }
        return result;
      }, {});
    }
    
    return data;
  }
}
```

**Localization:**
```typescript
@Injectable()
export class LocalizationInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const locale = request.headers['accept-language'] || 'en';
    
    return next.handle().pipe(
      map(data => ({
        data,
        locale,
        currency: this.getCurrencyByLocale(locale),
        dateFormat: this.getDateFormatByLocale(locale),
      })),
    );
  }
  
  private getCurrencyByLocale(locale: string): string {
    const currencies = { en: 'USD', fr: 'EUR', ja: 'JPY' };
    return currencies[locale.split('-')[0]] || 'USD';
  }
  
  private getDateFormatByLocale(locale: string): string {
    const formats = { en: 'MM/DD/YYYY', fr: 'DD/MM/YYYY', ja: 'YYYY/MM/DD' };
    return formats[locale.split('-')[0]] || 'MM/DD/YYYY';
  }
}
```

**Interview Tip**: Use `map()` to transform responses. Common transformations: wrapping data, adding metadata, pagination, array mapping, case conversion, removing nulls. Chain multiple `map()` operations for complex transforms. Return value from `map()` becomes new response data.

</details>

### 21. How do you wrap responses in a standard format?

<details>
<summary>Answer</summary>

Create an interceptor that wraps all responses in a consistent structure with status, data, metadata.

**Standard Response Wrapper:**
```typescript
export interface StandardResponse<T> {
  success: boolean;
  statusCode: number;
  data: T;
  timestamp: string;
  path: string;
}

@Injectable()
export class ResponseWrapperInterceptor<T> implements NestInterceptor<T, StandardResponse<T>> {
  intercept(context: ExecutionContext, next: CallHandler<T>): Observable<StandardResponse<T>> {
    const request = context.switchToHttp().getRequest();
    const response = context.switchToHttp().getResponse();
    
    return next.handle().pipe(
      map(data => ({
        success: true,
        statusCode: response.statusCode,
        data,
        timestamp: new Date().toISOString(),
        path: request.url,
      })),
    );
  }
}

// Original: { id: 1, name: 'John' }
// Wrapped:
{
  "success": true,
  "statusCode": 200,
  "data": { "id": 1, "name": "John" },
  "timestamp": "2024-01-01T00:00:00.000Z",
  "path": "/users/1"
}
```

**With Message Field:**
```typescript
@Injectable()
export class MessageWrapperInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    
    return next.handle().pipe(
      map(data => ({
        success: true,
        message: this.getMessageByMethod(request.method),
        data,
        timestamp: new Date().toISOString(),
      })),
    );
  }
  
  private getMessageByMethod(method: string): string {
    const messages = {
      GET: 'Data retrieved successfully',
      POST: 'Resource created successfully',
      PUT: 'Resource updated successfully',
      PATCH: 'Resource updated successfully',
      DELETE: 'Resource deleted successfully',
    };
    return messages[method] || 'Request successful';
  }
}
```

**With Metadata:**
```typescript
@Injectable()
export class MetadataWrapperInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const start = Date.now();
    
    return next.handle().pipe(
      map(data => ({
        success: true,
        data,
        meta: {
          timestamp: new Date().toISOString(),
          path: request.url,
          method: request.method,
          duration: Date.now() - start,
          version: '1.0.0',
          requestId: request.requestId || null,
        },
      })),
    );
  }
}
```

**With Error Handling:**
```typescript
@Injectable()
export class StandardWrapperInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const response = context.switchToHttp().getResponse();
    
    return next.handle().pipe(
      map(data => ({
        success: true,
        statusCode: response.statusCode,
        data,
        timestamp: new Date().toISOString(),
        path: request.url,
      })),
      catchError(err => {
        return throwError(() => ({
          success: false,
          statusCode: err.status || 500,
          error: err.message || 'Internal server error',
          timestamp: new Date().toISOString(),
          path: request.url,
        }));
      }),
    );
  }
}
```

**Pagination Wrapper:**
```typescript
interface PaginatedResponse<T> {
  success: boolean;
  data: T[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
    hasNext: boolean;
    hasPrevious: boolean;
  };
}

@Injectable()
export class PaginationWrapperInterceptor<T> implements NestInterceptor<T[], PaginatedResponse<T>> {
  intercept(context: ExecutionContext, next: CallHandler<T[]>): Observable<PaginatedResponse<T>> {
    const request = context.switchToHttp().getRequest();
    const { page = 1, limit = 10 } = request.query;
    
    return next.handle().pipe(
      map(data => {
        const items = Array.isArray(data) ? data : [];
        const total = items.length;
        const totalPages = Math.ceil(total / Number(limit));
        
        return {
          success: true,
          data: items,
          pagination: {
            page: Number(page),
            limit: Number(limit),
            total,
            totalPages,
            hasNext: Number(page) < totalPages,
            hasPrevious: Number(page) > 1,
          },
        };
      }),
    );
  }
}
```

**API Version Wrapper:**
```typescript
@Injectable()
export class ApiVersionWrapperInterceptor implements NestInterceptor {
  constructor(private readonly configService: ConfigService) {}
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const apiVersion = this.configService.get('API_VERSION');
    const request = context.switchToHttp().getRequest();
    
    return next.handle().pipe(
      map(data => ({
        apiVersion,
        timestamp: new Date().toISOString(),
        path: request.url,
        data,
      })),
    );
  }
}
```

**Conditional Wrapper:**
```typescript
@Injectable()
export class ConditionalWrapperInterceptor implements NestInterceptor {
  constructor(private readonly reflector: Reflector) {}
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const skipWrapper = this.reflector.get<boolean>(
      'skipWrapper',
      context.getHandler(),
    );
    
    if (skipWrapper) {
      return next.handle(); // Return data as-is
    }
    
    return next.handle().pipe(
      map(data => ({
        success: true,
        data,
        timestamp: new Date().toISOString(),
      })),
    );
  }
}

// Usage with custom decorator
export const SkipWrapper = () => SetMetadata('skipWrapper', true);

@Controller('files')
export class FilesController {
  @Get('download')
  @SkipWrapper()
  downloadFile() {
    // Returns raw file data without wrapper
    return fileBuffer;
  }
}
```

**Interview Tip**: Wrap responses consistently with `map()` operator. Common fields: `success`, `statusCode`, `data`, `timestamp`, `path`. Use TypeScript interfaces for type safety. Can add metadata, pagination, versioning. Use custom decorator to skip wrapper for specific routes (file downloads, health checks).

</details>

### 22. How do you implement a timeout Interceptor using `timeout()`?

<details>
<summary>Answer</summary>

Use RxJS `timeout()` operator to cancel requests that take too long.

**Basic Timeout:**
```typescript
import { timeout, catchError } from 'rxjs/operators';
import { throwError, TimeoutError } from 'rxjs';
import { RequestTimeoutException } from '@nestjs/common';

@Injectable()
export class TimeoutInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      timeout(5000), // 5 seconds
      catchError(err => {
        if (err instanceof TimeoutError) {
          return throwError(() => new RequestTimeoutException('Request timeout'));
        }
        return throwError(() => err);
      }),
    );
  }
}
```

**Configurable Timeout:**
```typescript
@Injectable()
export class ConfigurableTimeoutInterceptor implements NestInterceptor {
  constructor(private readonly configService: ConfigService) {}
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const timeoutMs = this.configService.get('REQUEST_TIMEOUT', 10000);
    
    return next.handle().pipe(
      timeout(timeoutMs),
      catchError(err => {
        if (err instanceof TimeoutError) {
          return throwError(() => new RequestTimeoutException(
            `Request timeout after ${timeoutMs}ms`,
          ));
        }
        return throwError(() => err);
      }),
    );
  }
}
```

**Custom Timeout by Route:**
```typescript
@Injectable()
export class CustomTimeoutInterceptor implements NestInterceptor {
  constructor(private readonly reflector: Reflector) {}
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const timeoutMs = this.reflector.get<number>(
      'timeout',
      context.getHandler(),
    ) || 5000; // Default 5 seconds
    
    return next.handle().pipe(
      timeout(timeoutMs),
      catchError(err => {
        if (err instanceof TimeoutError) {
          const request = context.switchToHttp().getRequest();
          return throwError(() => new RequestTimeoutException(
            `${request.method} ${request.url} timeout after ${timeoutMs}ms`,
          ));
        }
        return throwError(() => err);
      }),
    );
  }
}

// Custom decorator
export const Timeout = (ms: number) => SetMetadata('timeout', ms);

// Usage
@Controller('users')
export class UsersController {
  @Get()
  @Timeout(10000) // 10 seconds for this endpoint
  findAll() {
    return this.usersService.findAll();
  }
  
  @Get(':id')
  @Timeout(2000) // 2 seconds for this endpoint
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(id);
  }
}
```

**With Logging:**
```typescript
@Injectable()
export class LoggingTimeoutInterceptor implements NestInterceptor {
  constructor(private readonly logger: Logger) {}
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { method, url } = request;
    
    return next.handle().pipe(
      timeout(5000),
      catchError(err => {
        if (err instanceof TimeoutError) {
          this.logger.error(`⏱️ Timeout: ${method} ${url} exceeded 5000ms`);
          return throwError(() => new RequestTimeoutException(
            'Request processing time exceeded',
          ));
        }
        return throwError(() => err);
      }),
    );
  }
}
```

**Different Timeouts by Method:**
```typescript
@Injectable()
export class MethodBasedTimeoutInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const timeoutMs = this.getTimeoutByMethod(request.method);
    
    return next.handle().pipe(
      timeout(timeoutMs),
      catchError(err => {
        if (err instanceof TimeoutError) {
          return throwError(() => new RequestTimeoutException());
        }
        return throwError(() => err);
      }),
    );
  }
  
  private getTimeoutByMethod(method: string): number {
    const timeouts = {
      GET: 5000,
      POST: 10000,
      PUT: 10000,
      PATCH: 10000,
      DELETE: 5000,
    };
    return timeouts[method] || 5000;
  }
}
```

**With Retry:**
```typescript
import { retry } from 'rxjs/operators';

@Injectable()
export class TimeoutWithRetryInterceptor implements NestInterceptor {
  constructor(private readonly logger: Logger) {}
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    
    return next.handle().pipe(
      timeout(5000),
      retry(2), // Retry 2 times on timeout
      catchError(err => {
        if (err instanceof TimeoutError) {
          this.logger.error(`Timeout after 2 retries: ${request.url}`);
          return throwError(() => new RequestTimeoutException(
            'Request timeout after retries',
          ));
        }
        return throwError(() => err);
      }),
    );
  }
}
```

**Graceful Fallback:**
```typescript
import { of } from 'rxjs';

@Injectable()
export class TimeoutFallbackInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      timeout(5000),
      catchError(err => {
        if (err instanceof TimeoutError) {
          // Return fallback response instead of error
          return of({
            success: false,
            message: 'Service temporarily unavailable',
            data: null,
          });
        }
        return throwError(() => err);
      }),
    );
  }
}
```

**Interview Tip**: Use `timeout()` operator to cancel long-running requests. Always pair with `catchError()` to handle `TimeoutError`. Can configure timeout per route using custom decorators and Reflector. Consider different timeouts for different HTTP methods. Can combine with `retry()` for resilience or return fallback data.

</details>

### 23. How do you implement error handling using `catchError()`?

<details>
<summary>Answer</summary>

Use `catchError()` operator to catch and transform errors in interceptors.

**Basic Error Handler:**
```typescript
import { catchError } from 'rxjs/operators';
import { throwError } from 'rxjs';

@Injectable()
export class ErrorHandlerInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      catchError(err => {
        console.error('Error caught:', err);
        return throwError(() => new HttpException(
          'Something went wrong',
          HttpStatus.INTERNAL_SERVER_ERROR,
        ));
      }),
    );
  }
}
```

**Transform Errors to Standard Format:**
```typescript
interface ErrorResponse {
  statusCode: number;
  message: string;
  error: string;
  timestamp: string;
  path: string;
}

@Injectable()
export class ErrorTransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    
    return next.handle().pipe(
      catchError(err => {
        const errorResponse: ErrorResponse = {
          statusCode: err.status || 500,
          message: err.message || 'Internal server error',
          error: err.name || 'Error',
          timestamp: new Date().toISOString(),
          path: request.url,
        };
        
        return throwError(() => new HttpException(
          errorResponse,
          errorResponse.statusCode,
        ));
      }),
    );
  }
}
```

**Log and Re-throw:**
```typescript
@Injectable()
export class ErrorLoggingInterceptor implements NestInterceptor {
  constructor(private readonly logger: Logger) {}
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { method, url, body, user } = request;
    
    return next.handle().pipe(
      catchError(err => {
        // Log error details
        this.logger.error({
          message: 'Request failed',
          method,
          url,
          body,
          userId: user?.id,
          error: err.message,
          stack: err.stack,
          timestamp: new Date().toISOString(),
        });
        
        // Re-throw the error
        return throwError(() => err);
      }),
    );
  }
}
```

**Handle Specific Errors:**
```typescript
@Injectable()
export class SpecificErrorInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      catchError(err => {
        // Handle different error types
        if (err instanceof NotFoundException) {
          return throwError(() => new HttpException(
            { message: 'Resource not found', statusCode: 404 },
            HttpStatus.NOT_FOUND,
          ));
        }
        
        if (err instanceof UnauthorizedException) {
          return throwError(() => new HttpException(
            { message: 'Access denied', statusCode: 401 },
            HttpStatus.UNAUTHORIZED,
          ));
        }
        
        if (err instanceof BadRequestException) {
          return throwError(() => new HttpException(
            { message: 'Invalid request', statusCode: 400 },
            HttpStatus.BAD_REQUEST,
          ));
        }
        
        // Default error handling
        return throwError(() => new HttpException(
          { message: 'Internal server error', statusCode: 500 },
          HttpStatus.INTERNAL_SERVER_ERROR,
        ));
      }),
    );
  }
}
```

**Send Error Notifications:**
```typescript
@Injectable()
export class ErrorNotificationInterceptor implements NestInterceptor {
  constructor(
    private readonly logger: Logger,
    private readonly notificationService: NotificationService,
  ) {}
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    
    return next.handle().pipe(
      catchError(async (err) => {
        this.logger.error(`Error: ${err.message}`, err.stack);
        
        // Send notification for critical errors (5xx)
        if (!err.status || err.status >= 500) {
          await this.notificationService.sendAlert({
            type: 'critical_error',
            message: err.message,
            method: request.method,
            url: request.url,
            userId: request.user?.id,
            timestamp: new Date(),
          });
        }
        
        return throwError(() => err);
      }),
    );
  }
}
```

**Sanitize Error Messages:**
```typescript
@Injectable()
export class ErrorSanitizerInterceptor implements NestInterceptor {
  constructor(private readonly configService: ConfigService) {}
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const isProduction = this.configService.get('NODE_ENV') === 'production';
    
    return next.handle().pipe(
      catchError(err => {
        if (isProduction) {
          // Hide sensitive error details in production
          const sanitizedError = {
            statusCode: err.status || 500,
            message: err.status < 500 ? err.message : 'Internal server error',
            // Don't expose stack trace
          };
          
          return throwError(() => new HttpException(
            sanitizedError,
            sanitizedError.statusCode,
          ));
        }
        
        // Show full error in development
        return throwError(() => err);
      }),
    );
  }
}
```

**Fallback Response:**
```typescript
import { of } from 'rxjs';

@Injectable()
export class ErrorFallbackInterceptor implements NestInterceptor {
  constructor(private readonly logger: Logger) {}
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      catchError(err => {
        this.logger.error(`Error: ${err.message}`, err.stack);
        
        // Return fallback response instead of throwing error
        return of({
          success: false,
          message: 'Service temporarily unavailable',
          error: err.message,
          timestamp: new Date().toISOString(),
        });
      }),
    );
  }
}
```

**Rate Limit Errors:**
```typescript
@Injectable()
export class RateLimitErrorInterceptor implements NestInterceptor {
  private errorCounts = new Map<string, number>();
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const key = `${request.ip}:${request.url}`;
    
    return next.handle().pipe(
      catchError(err => {
        // Track error count
        const count = (this.errorCounts.get(key) || 0) + 1;
        this.errorCounts.set(key, count);
        
        // If too many errors, apply rate limiting
        if (count > 10) {
          return throwError(() => new HttpException(
            'Too many errors. Please try again later.',
            HttpStatus.TOO_MANY_REQUESTS,
          ));
        }
        
        // Reset count after 1 minute
        setTimeout(() => this.errorCounts.delete(key), 60000);
        
        return throwError(() => err);
      }),
    );
  }
}
```

**Interview Tip**: Use `catchError()` to handle errors in Observable stream. Can transform errors, log them, send notifications, return fallback values, or re-throw. Always return Observable with `throwError()` or `of()`. Handle specific error types differently. Sanitize errors in production. Can combine with logging, monitoring, alerting.

</details>

### 24. What is `ClassSerializerInterceptor` used for?

<details>
<summary>Answer</summary>

`ClassSerializerInterceptor` uses `class-transformer` to serialize responses and exclude sensitive fields using decorators.

**Basic Usage:**
```typescript
import { ClassSerializerInterceptor, UseInterceptors } from '@nestjs/common';

@Controller('users')
@UseInterceptors(ClassSerializerInterceptor)
export class UsersController {
  @Get()
  findAll() {
    return [
      { id: 1, email: 'john@example.com', password: 'secret123' },
    ];
  }
}

// Password will be automatically excluded
```

**Entity with @Exclude:**
```typescript
import { Exclude } from 'class-transformer';

export class User {
  id: number;
  email: string;
  
  @Exclude()
  password: string;
  
  @Exclude()
  refreshToken: string;
  
  firstName: string;
  lastName: string;
}

// Response:
{
  "id": 1,
  "email": "john@example.com",
  "firstName": "John",
  "lastName": "Doe"
}
// password and refreshToken are excluded
```

**Global Registration:**
```typescript
// main.ts
import { ClassSerializerInterceptor } from '@nestjs/common';
import { Reflector } from '@nestjs/core';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  app.useGlobalInterceptors(
    new ClassSerializerInterceptor(app.get(Reflector)),
  );
  
  await app.listen(3000);
}
bootstrap();

// Or in module
@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: ClassSerializerInterceptor,
    },
  ],
})
export class AppModule {}
```

**@Expose() - Include Only Specific Fields:**
```typescript
import { Expose } from 'class-transformer';

export class UserDto {
  @Expose()
  id: number;
  
  @Expose()
  email: string;
  
  // All other fields are excluded by default
  password: string;
  refreshToken: string;
}
```

**@Transform() - Transform Field Values:**
```typescript
import { Exclude, Expose, Transform } from 'class-transformer';

export class User {
  @Expose()
  id: number;
  
  @Expose()
  @Transform(({ value }) => value.toLowerCase())
  email: string;
  
  @Expose()
  @Transform(({ obj }) => `${obj.firstName} ${obj.lastName}`)
  fullName: string;
  
  firstName: string;
  lastName: string;
  
  @Exclude()
  password: string;
}
```

**Groups - Conditional Serialization:**
```typescript
import { Exclude, Expose } from 'class-transformer';

export class User {
  @Expose()
  id: number;
  
  @Expose()
  email: string;
  
  @Expose({ groups: ['admin'] })
  role: string;
  
  @Expose({ groups: ['admin'] })
  createdAt: Date;
  
  @Exclude()
  password: string;
}

// Controller
@Controller('users')
export class UsersController {
  @Get()
  @SerializeOptions({ groups: ['admin'] })
  @UseInterceptors(ClassSerializerInterceptor)
  findAll() {
    return this.usersService.findAll();
  }
}
```

**Nested Objects:**
```typescript
import { Exclude, Type } from 'class-transformer';

export class Profile {
  bio: string;
  
  @Exclude()
  privateNotes: string;
}

export class User {
  id: number;
  email: string;
  
  @Type(() => Profile)
  profile: Profile;
  
  @Exclude()
  password: string;
}
```

**Custom Serializer:**
```typescript
import { SerializeOptions } from '@nestjs/common';

@Controller('users')
export class UsersController {
  @Get(':id')
  @SerializeOptions({
    excludePrefixes: ['_'],
    strategy: 'excludeAll', // Only include @Expose() fields
  })
  @UseInterceptors(ClassSerializerInterceptor)
  findOne(@Param('id') id: string) {
    return {
      id: 1,
      email: 'john@example.com',
      _internalId: 'abc123', // Will be excluded
      password: 'secret', // Will be excluded
    };
  }
}
```

**Date Formatting:**
```typescript
import { Expose, Transform } from 'class-transformer';

export class Post {
  @Expose()
  id: number;
  
  @Expose()
  title: string;
  
  @Expose()
  @Transform(({ value }) => value.toISOString())
  createdAt: Date;
  
  @Expose()
  @Transform(({ value }) => value.toISOString())
  updatedAt: Date;
}
```

**Interview Tip**: `ClassSerializerInterceptor` automatically serializes responses using `class-transformer`. Use `@Exclude()` to hide sensitive fields like passwords. Use `@Expose()` to include only specific fields. Use `@Transform()` for custom transformations. Register globally or per route. Supports groups for conditional serialization. Works with entity classes from TypeORM/Prisma.

</details>

### 25. How do you exclude sensitive data using `@Exclude()` decorator?

<details>
<summary>Answer</summary>

Use `@Exclude()` decorator from `class-transformer` with `ClassSerializerInterceptor` to automatically remove sensitive fields from responses.

**Basic Example:**
```typescript
import { Exclude } from 'class-transformer';
import { ClassSerializerInterceptor, UseInterceptors } from '@nestjs/common';

export class User {
  id: number;
  email: string;
  username: string;
  
  @Exclude()
  password: string;
  
  @Exclude()
  refreshToken: string;
}

@Controller('users')
@UseInterceptors(ClassSerializerInterceptor)
export class UsersController {
  @Get(':id')
  findOne(@Param('id') id: string): User {
    return {
      id: 1,
      email: 'john@example.com',
      username: 'john_doe',
      password: 'secret123',
      refreshToken: 'token_abc',
    };
  }
}

// Response (password and refreshToken excluded):
{
  "id": 1,
  "email": "john@example.com",
  "username": "john_doe"
}
```

**Multiple Sensitive Fields:**
```typescript
export class User {
  id: number;
  email: string;
  
  @Exclude()
  password: string;
  
  @Exclude()
  passwordHash: string;
  
  @Exclude()
  passwordSalt: string;
  
  @Exclude()
  refreshToken: string;
  
  @Exclude()
  twoFactorSecret: string;
  
  @Exclude()
  resetPasswordToken: string;
}
```

**TypeORM Entity:**
```typescript
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';
import { Exclude } from 'class-transformer';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;
  
  @Column()
  email: string;
  
  @Column()
  username: string;
  
  @Column()
  @Exclude()
  password: string;
  
  @Column({ nullable: true })
  @Exclude()
  refreshToken: string;
  
  @Column({ default: 'user' })
  role: string;
}
```

**Conditional Exclusion:**
```typescript
import { Exclude } from 'class-transformer';

export class User {
  id: number;
  email: string;
  
  @Exclude({ toPlainOnly: true }) // Exclude only when serializing to JSON
  password: string;
  
  @Exclude()
  refreshToken: string;
}
```

**Exclude with Groups:**
```typescript
import { Exclude, Expose } from 'class-transformer';

export class User {
  @Expose()
  id: number;
  
  @Expose()
  email: string;
  
  @Expose({ groups: ['admin'] })
  @Exclude({ groups: ['user'] })
  sensitiveData: string;
  
  @Exclude()
  password: string;
}

// For regular users
@Get('profile')
@SerializeOptions({ groups: ['user'] })
@UseInterceptors(ClassSerializerInterceptor)
getProfile() {
  return user;
}

// For admins
@Get('admin/users/:id')
@SerializeOptions({ groups: ['admin'] })
@UseInterceptors(ClassSerializerInterceptor)
getUser(@Param('id') id: string) {
  return user;
}
```

**Exclude Prefixes:**
```typescript
export class User {
  id: number;
  email: string;
  
  // These will be excluded (underscore prefix)
  _internalId: string;
  _metadata: any;
  __typename: string;
  
  // Use SerializeOptions
}

@Controller('users')
export class UsersController {
  @Get()
  @SerializeOptions({ excludePrefixes: ['_'] })
  @UseInterceptors(ClassSerializerInterceptor)
  findAll() {
    return users;
  }
}
```

**Nested Objects:**
```typescript
import { Exclude, Type } from 'class-transformer';

export class Address {
  street: string;
  city: string;
  
  @Exclude()
  coordinates: { lat: number; lng: number };
}

export class User {
  id: number;
  email: string;
  
  @Type(() => Address)
  address: Address;
  
  @Exclude()
  password: string;
}
```

**Custom Exclude Logic:**
```typescript
import { Exclude, Transform } from 'class-transformer';

export class User {
  id: number;
  email: string;
  
  @Exclude()
  password: string;
  
  // Exclude if value is null/empty
  @Transform(({ value }) => value || undefined, { toPlainOnly: true })
  middleName: string;
  
  // Mask sensitive data instead of excluding
  @Transform(({ value }) => value ? '***' : undefined)
  ssn: string;
}
```

**Exclude in Arrays:**
```typescript
export class User {
  id: number;
  email: string;
  
  @Exclude()
  password: string;
}

@Controller('users')
@UseInterceptors(ClassSerializerInterceptor)
export class UsersController {
  @Get()
  findAll(): User[] {
    return [
      { id: 1, email: 'john@example.com', password: 'secret1' },
      { id: 2, email: 'jane@example.com', password: 'secret2' },
    ];
  }
}

// Response (all passwords excluded):
[
  { "id": 1, "email": "john@example.com" },
  { "id": 2, "email": "jane@example.com" }
]
```

**Interview Tip**: Use `@Exclude()` decorator on entity/DTO fields to hide sensitive data like passwords, tokens, secrets. Must use with `ClassSerializerInterceptor`. Works automatically on all responses. Can use groups for conditional exclusion. Supports nested objects with `@Type()`. Consider masking vs excluding for partial sensitivity.

</details>

## Cache Interceptor

### 26. What is the built-in `CacheInterceptor`?

<details>
<summary>Answer</summary>

`CacheInterceptor` is a built-in NestJS interceptor that caches HTTP responses to improve performance.

**Setup with Cache Module:**
```typescript
// app.module.ts
import { Module, CacheModule } from '@nestjs/common';

@Module({
  imports: [
    CacheModule.register({
      ttl: 60, // seconds
      max: 100, // maximum number of items in cache
    }),
  ],
})
export class AppModule {}
```

**Basic Usage:**
```typescript
import { CacheInterceptor, UseInterceptors } from '@nestjs/common';

@Controller('users')
export class UsersController {
  @Get()
  @UseInterceptors(CacheInterceptor)
  findAll() {
    console.log('Executing findAll...'); // Only logs on cache miss
    return this.usersService.findAll();
  }
}

// First request: Cache miss, executes handler
// Subsequent requests (within TTL): Cache hit, returns cached data
```

**Global Cache:**
```typescript
// app.module.ts
import { CacheModule, CacheInterceptor } from '@nestjs/common';
import { APP_INTERCEPTOR } from '@nestjs/core';

@Module({
  imports: [
    CacheModule.register({
      ttl: 60,
      max: 100,
    }),
  ],
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: CacheInterceptor,
    },
  ],
})
export class AppModule {}

// Now all GET endpoints are cached automatically
```

**Cache Configuration:**
```typescript
import { CacheModule } from '@nestjs/common';
import * as redisStore from 'cache-manager-redis-store';

@Module({
  imports: [
    CacheModule.register({
      store: redisStore,
      host: 'localhost',
      port: 6379,
      ttl: 600, // 10 minutes
      max: 1000,
    }),
  ],
})
export class AppModule {}
```

**Custom TTL per Route:**
```typescript
import { CacheInterceptor, CacheTTL, UseInterceptors } from '@nestjs/common';

@Controller('posts')
export class PostsController {
  @Get()
  @UseInterceptors(CacheInterceptor)
  @CacheTTL(30) // Cache for 30 seconds
  findAll() {
    return this.postsService.findAll();
  }
  
  @Get(':id')
  @UseInterceptors(CacheInterceptor)
  @CacheTTL(300) // Cache for 5 minutes
  findOne(@Param('id') id: string) {
    return this.postsService.findOne(id);
  }
}
```

**Disable Cache for Specific Route:**
```typescript
@Controller('data')
@UseInterceptors(CacheInterceptor)
export class DataController {
  @Get('cached')
  getCached() {
    // This is cached
    return { data: 'cached' };
  }
  
  @Get('fresh')
  @CacheTTL(0) // TTL = 0 disables caching
  getFresh() {
    // This is NOT cached
    return { data: 'fresh', timestamp: Date.now() };
  }
}
```

**Manual Cache Operations:**
```typescript
import { CACHE_MANAGER, Inject } from '@nestjs/common';
import { Cache } from 'cache-manager';

@Injectable()
export class UsersService {
  constructor(@Inject(CACHE_MANAGER) private cacheManager: Cache) {}
  
  async findAll() {
    const cacheKey = 'users_all';
    
    // Check cache
    const cached = await this.cacheManager.get(cacheKey);
    if (cached) {
      return cached;
    }
    
    // Fetch data
    const users = await this.userRepository.find();
    
    // Store in cache
    await this.cacheManager.set(cacheKey, users, { ttl: 60 });
    
    return users;
  }
  
  async invalidateCache() {
    await this.cacheManager.del('users_all');
  }
  
  async clearAll() {
    await this.cacheManager.reset();
  }
}
```

**Cache Key Generation:**
```typescript
// Default key generation: based on URL
// GET /users/123 -> key: "users/123"

// Custom key
import { CacheKey } from '@nestjs/common';

@Controller('products')
export class ProductsController {
  @Get(':id')
  @UseInterceptors(CacheInterceptor)
  @CacheKey('custom_product_key')
  findOne(@Param('id') id: string) {
    return this.productsService.findOne(id);
  }
}
```

**Interview Tip**: `CacheInterceptor` caches GET endpoint responses automatically. Use `CacheModule.register()` for setup. Configure TTL (time to live) globally or per route with `@CacheTTL()`. Cache key based on URL by default. Can use Redis for distributed caching. Only caches successful responses (2xx status codes).

</details>

### 27. How do you cache responses?

<details>
<summary>Answer</summary>

Cache responses using `CacheInterceptor` with `CacheModule` or implement custom caching logic.

**Basic Response Caching:**
```typescript
// 1. Import and register CacheModule
import { Module, CacheModule } from '@nestjs/common';

@Module({
  imports: [
    CacheModule.register({
      ttl: 60, // 60 seconds
      max: 100, // max 100 items
    }),
  ],
})
export class AppModule {}

// 2. Apply CacheInterceptor
import { CacheInterceptor, UseInterceptors } from '@nestjs/common';

@Controller('users')
export class UsersController {
  @Get()
  @UseInterceptors(CacheInterceptor)
  findAll() {
    return this.usersService.findAll();
  }
}
```

**Custom Cache Interceptor:**
```typescript
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable, of } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class CustomCacheInterceptor implements NestInterceptor {
  private cache = new Map<string, any>();
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const cacheKey = this.generateKey(request);
    
    // Check cache
    if (this.cache.has(cacheKey)) {
      console.log('Cache HIT:', cacheKey);
      return of(this.cache.get(cacheKey));
    }
    
    console.log('Cache MISS:', cacheKey);
    
    // Execute handler and cache result
    return next.handle().pipe(
      tap(data => {
        this.cache.set(cacheKey, data);
        
        // Auto-expire after 60 seconds
        setTimeout(() => {
          this.cache.delete(cacheKey);
        }, 60000);
      }),
    );
  }
  
  private generateKey(request: any): string {
    const { method, url, query } = request;
    return `${method}:${url}:${JSON.stringify(query)}`;
  }
}
```

**Redis Cache:**
```typescript
// Install: npm install cache-manager cache-manager-redis-store redis

import { CacheModule } from '@nestjs/common';
import * as redisStore from 'cache-manager-redis-store';

@Module({
  imports: [
    CacheModule.register({
      store: redisStore,
      host: process.env.REDIS_HOST || 'localhost',
      port: process.env.REDIS_PORT || 6379,
      password: process.env.REDIS_PASSWORD,
      ttl: 600,
      max: 1000,
    }),
  ],
})
export class AppModule {}
```

**Conditional Caching:**
```typescript
@Injectable()
export class ConditionalCacheInterceptor implements NestInterceptor {
  constructor(@Inject(CACHE_MANAGER) private cacheManager: Cache) {}
  
  async intercept(context: ExecutionContext, next: CallHandler): Promise<Observable<any>> {
    const request = context.switchToHttp().getRequest();
    
    // Only cache GET requests
    if (request.method !== 'GET') {
      return next.handle();
    }
    
    // Skip cache if header is present
    if (request.headers['x-no-cache']) {
      return next.handle();
    }
    
    const cacheKey = request.url;
    const cached = await this.cacheManager.get(cacheKey);
    
    if (cached) {
      return of(cached);
    }
    
    return next.handle().pipe(
      tap(async (data) => {
        await this.cacheManager.set(cacheKey, data, { ttl: 60 });
      }),
    );
  }
}
```

**Query-Aware Caching:**
```typescript
@Injectable()
export class QueryCacheInterceptor implements NestInterceptor {
  private cache = new Map<string, any>();
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const cacheKey = this.buildCacheKey(request);
    
    if (this.cache.has(cacheKey)) {
      return of(this.cache.get(cacheKey));
    }
    
    return next.handle().pipe(
      tap(data => {
        this.cache.set(cacheKey, data);
      }),
    );
  }
  
  private buildCacheKey(request: any): string {
    const { url, query, user } = request;
    // Include user ID in cache key for user-specific data
    return `${url}:${JSON.stringify(query)}:${user?.id || 'anonymous'}`;
  }
}
```

**Cache with Headers:**
```typescript
@Injectable()
export class HeaderCacheInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const response = context.switchToHttp().getResponse();
    const cacheKey = request.url;
    const cached = this.cache.get(cacheKey);
    
    if (cached) {
      // Add cache status header
      response.setHeader('X-Cache', 'HIT');
      response.setHeader('X-Cache-Key', cacheKey);
      return of(cached);
    }
    
    response.setHeader('X-Cache', 'MISS');
    
    return next.handle().pipe(
      tap(data => {
        this.cache.set(cacheKey, data);
      }),
    );
  }
  
  private cache = new Map();
}
```

**LRU Cache (Least Recently Used):**
```typescript
// Install: npm install lru-cache
import LRU from 'lru-cache';

@Injectable()
export class LRUCacheInterceptor implements NestInterceptor {
  private cache: LRU<string, any>;
  
  constructor() {
    this.cache = new LRU({
      max: 100, // max items
      maxAge: 60000, // 60 seconds
    });
  }
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const cacheKey = request.url;
    
    const cached = this.cache.get(cacheKey);
    if (cached) {
      return of(cached);
    }
    
    return next.handle().pipe(
      tap(data => {
        this.cache.set(cacheKey, data);
      }),
    );
  }
}
```

**Cache Invalidation:**
```typescript
@Injectable()
export class InvalidatableCacheInterceptor implements NestInterceptor {
  constructor(@Inject(CACHE_MANAGER) private cacheManager: Cache) {}
  
  async intercept(context: ExecutionContext, next: CallHandler): Promise<Observable<any>> {
    const request = context.switchToHttp().getRequest();
    const { method, url } = request;
    
    // Invalidate cache on write operations
    if (['POST', 'PUT', 'PATCH', 'DELETE'].includes(method)) {
      await this.invalidateRelatedCache(url);
      return next.handle();
    }
    
    // Cache GET requests
    const cacheKey = url;
    const cached = await this.cacheManager.get(cacheKey);
    
    if (cached) {
      return of(cached);
    }
    
    return next.handle().pipe(
      tap(async (data) => {
        await this.cacheManager.set(cacheKey, data, { ttl: 60 });
      }),
    );
  }
  
  private async invalidateRelatedCache(url: string) {
    // Invalidate all cache entries for this resource
    const pattern = url.split('/').slice(0, 2).join('/');
    // Implementation depends on cache store
    await this.cacheManager.reset();
  }
}
```

**Interview Tip**: Cache responses with `CacheInterceptor` + `CacheModule` or custom implementation. Cache key typically based on URL + query params. Consider user context for personalized data. Only cache GET requests. Invalidate cache on mutations (POST/PUT/DELETE). Use Redis for distributed caching. Add cache headers for debugging.

</details>

### 28. How do you use `@CacheKey()` and `@CacheTTL()` decorators?

<details>
<summary>Answer</summary>

`@CacheKey()` customizes cache key and `@CacheTTL()` sets time-to-live for specific routes.

**@CacheTTL() - Custom Time-to-Live:**
```typescript
import { CacheInterceptor, CacheTTL, UseInterceptors } from '@nestjs/common';

@Controller('posts')
export class PostsController {
  // Cache for 30 seconds
  @Get()
  @UseInterceptors(CacheInterceptor)
  @CacheTTL(30)
  findAll() {
    return this.postsService.findAll();
  }
  
  // Cache for 5 minutes
  @Get(':id')
  @UseInterceptors(CacheInterceptor)
  @CacheTTL(300)
  findOne(@Param('id') id: string) {
    return this.postsService.findOne(id);
  }
  
  // Cache for 1 hour
  @Get('popular')
  @UseInterceptors(CacheInterceptor)
  @CacheTTL(3600)
  getPopular() {
    return this.postsService.getPopular();
  }
}
```

**@CacheKey() - Custom Cache Key:**
```typescript
import { CacheKey } from '@nestjs/common';

@Controller('products')
export class ProductsController {
  // Default key: "products/123"
  @Get(':id')
  @UseInterceptors(CacheInterceptor)
  findOne(@Param('id') id: string) {
    return this.productsService.findOne(id);
  }
  
  // Custom key: "featured_products"
  @Get('featured')
  @UseInterceptors(CacheInterceptor)
  @CacheKey('featured_products')
  @CacheTTL(3600)
  getFeatured() {
    return this.productsService.getFeatured();
  }
}
```

**Combined Usage:**
```typescript
@Controller('users')
export class UsersController {
  @Get('profile')
  @UseInterceptors(CacheInterceptor)
  @CacheKey('user_profile')
  @CacheTTL(600) // 10 minutes
  getProfile(@Request() req) {
    return this.usersService.getProfile(req.user.id);
  }
  
  @Get('settings')
  @UseInterceptors(CacheInterceptor)
  @CacheKey('user_settings')
  @CacheTTL(1800) // 30 minutes
  getSettings(@Request() req) {
    return this.usersService.getSettings(req.user.id);
  }
}
```

**Dynamic Cache Key:**
```typescript
// Custom decorator for dynamic cache keys
import { SetMetadata } from '@nestjs/common';

export const DynamicCacheKey = (prefix: string) => SetMetadata('cacheKeyPrefix', prefix);

@Injectable()
export class DynamicCacheInterceptor implements NestInterceptor {
  constructor(
    @Inject(CACHE_MANAGER) private cacheManager: Cache,
    private reflector: Reflector,
  ) {}
  
  async intercept(context: ExecutionContext, next: CallHandler): Promise<Observable<any>> {
    const request = context.switchToHttp().getRequest();
    const keyPrefix = this.reflector.get<string>(
      'cacheKeyPrefix',
      context.getHandler(),
    ) || '';
    
    const cacheKey = `${keyPrefix}:${request.user?.id}:${request.url}`;
    const cached = await this.cacheManager.get(cacheKey);
    
    if (cached) {
      return of(cached);
    }
    
    return next.handle().pipe(
      tap(async (data) => {
        const ttl = this.reflector.get<number>('cacheTTL', context.getHandler()) || 60;
        await this.cacheManager.set(cacheKey, data, { ttl });
      }),
    );
  }
}

// Usage
@Controller('data')
export class DataController {
  @Get('user-specific')
  @UseInterceptors(DynamicCacheInterceptor)
  @DynamicCacheKey('user_data')
  @CacheTTL(300)
  getUserData(@Request() req) {
    return this.dataService.getUserData(req.user.id);
  }
}
```

**Different TTL for Different Data:**
```typescript
@Controller('content')
export class ContentController {
  // Frequently changing data - short TTL
  @Get('trending')
  @UseInterceptors(CacheInterceptor)
  @CacheTTL(60) // 1 minute
  getTrending() {
    return this.contentService.getTrending();
  }
  
  // Rarely changing data - long TTL
  @Get('categories')
  @UseInterceptors(CacheInterceptor)
  @CacheTTL(86400) // 24 hours
  getCategories() {
    return this.contentService.getCategories();
  }
  
  // Static data - very long TTL
  @Get('terms')
  @UseInterceptors(CacheInterceptor)
  @CacheTTL(604800) // 7 days
  getTerms() {
    return this.contentService.getTerms();
  }
}
```

**Disable Caching:**
```typescript
@Controller('api')
@UseInterceptors(CacheInterceptor)
export class ApiController {
  // Cached endpoint
  @Get('data')
  @CacheTTL(300)
  getData() {
    return this.apiService.getData();
  }
  
  // Disable caching with TTL = 0
  @Get('realtime')
  @CacheTTL(0)
  getRealtime() {
    return this.apiService.getRealtime();
  }
}
```

**Environment-Based TTL:**
```typescript
@Controller('reports')
export class ReportsController {
  constructor(private configService: ConfigService) {}
  
  @Get('sales')
  @UseInterceptors(CacheInterceptor)
  getSalesReport() {
    // TTL from environment variable
    const ttl = this.configService.get('CACHE_TTL_REPORTS', 3600);
    // Note: Can't use dynamic TTL with decorator, need custom interceptor
    return this.reportsService.getSales();
  }
}

// Better approach with custom interceptor
@Injectable()
export class ConfigurableCacheInterceptor implements NestInterceptor {
  constructor(
    @Inject(CACHE_MANAGER) private cacheManager: Cache,
    private configService: ConfigService,
  ) {}
  
  async intercept(context: ExecutionContext, next: CallHandler): Promise<Observable<any>> {
    const request = context.switchToHttp().getRequest();
    const cacheKey = request.url;
    const cached = await this.cacheManager.get(cacheKey);
    
    if (cached) {
      return of(cached);
    }
    
    return next.handle().pipe(
      tap(async (data) => {
        const ttl = this.configService.get('CACHE_TTL', 60);
        await this.cacheManager.set(cacheKey, data, { ttl });
      }),
    );
  }
}
```

**Interview Tip**: `@CacheTTL(seconds)` sets cache expiration per route. `@CacheKey(key)` customizes cache key (default is URL). Use longer TTL for static/rarely changing data. Use `@CacheTTL(0)` to disable caching. Combine both decorators for full control. Consider environment-based TTL with custom interceptor for flexibility.

</details>

## Interceptors vs Other Features

### 29. What is the difference between Interceptors and Middleware?

<details>
<summary>Answer</summary>

**Interceptors** and **Middleware** both process requests, but have different capabilities and use cases.

**Comparison Table:**

| Feature | Middleware | Interceptors |
|---------|-----------|--------------|
| **Execution Point** | Before routing | Before & after handler |
| **Access to Handler** | ❌ No | ✅ Yes (ExecutionContext) |
| **Access to Response** | Limited | ✅ Full access |
| **Transform Response** | ❌ No | ✅ Yes (RxJS) |
| **RxJS Support** | ❌ No | ✅ Yes |
| **Route Awareness** | ❌ No | ✅ Yes |
| **Can Skip Handler** | ✅ Yes | ✅ Yes |
| **Scope** | Global, Route | Global, Controller, Method |
| **Typical Use** | Auth, CORS, Logging | Response transform, Caching |

**Request Lifecycle:**
```
Request
  ↓
1. Middleware ← Runs first
  ↓
2. Guards
  ↓
3. Interceptors (before)
  ↓
4. Pipes
  ↓
5. Handler
  ↓
6. Interceptors (after) ← Can transform response
  ↓
7. Exception Filters
  ↓
Response
```

**Middleware Example:**
```typescript
// Middleware - runs before routing
@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log('Request:', req.method, req.url);
    
    // Can't access route handler or transform response easily
    // Can't use RxJS
    
    next(); // Must call next()
  }
}

// Apply middleware
@Module({})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes('*');
  }
}
```

**Interceptor Example:**
```typescript
// Interceptor - runs before and after handler
@Injectable()
export class LoggerInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    console.log('Request:', request.method, request.url);
    
    // Can access ExecutionContext
    // Can use RxJS operators
    // Can transform response
    
    return next.handle().pipe(
      map(data => ({
        success: true,
        data,
      })),
    );
  }
}
```

**Middleware: Cannot Transform Response:**
```typescript
@Injectable()
export class ResponseMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    // ❌ Can't easily wrap response
    // Would need to intercept res.json(), res.send(), etc.
    
    next();
  }
}
```

**Interceptor: Easy Response Transformation:**
```typescript
@Injectable()
export class ResponseInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    // ✅ Easy to wrap response
    return next.handle().pipe(
      map(data => ({
        success: true,
        data,
        timestamp: Date.now(),
      })),
    );
  }
}
```

**Middleware: No Handler Context:**
```typescript
@Injectable()
export class AuthMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    // ❌ Don't know which handler will execute
    // ❌ Can't access handler metadata
    
    const token = req.headers.authorization;
    if (!token) {
      throw new UnauthorizedException();
    }
    next();
  }
}
```

**Interceptor: Full Handler Context:**
```typescript
@Injectable()
export class AuthInterceptor implements NestInterceptor {
  constructor(private reflector: Reflector) {}
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    // ✅ Know which handler will execute
    // ✅ Can access handler metadata
    
    const isPublic = this.reflector.get<boolean>('isPublic', context.getHandler());
    
    if (isPublic) {
      return next.handle();
    }
    
    const request = context.switchToHttp().getRequest();
    if (!request.headers.authorization) {
      throw new UnauthorizedException();
    }
    
    return next.handle();
  }
}
```

**When to Use Middleware:**
```typescript
// ✅ Use Middleware for:
// - CORS configuration
// - Body parsing
// - Cookie parsing
// - Session handling
// - Static file serving
// - Request logging (without response transformation)
// - Authentication (simple)

@Module({})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(
        cors(),
        helmet(),
        morgan('combined'),
      )
      .forRoutes('*');
  }
}
```

**When to Use Interceptors:**
```typescript
// ✅ Use Interceptors for:
// - Response transformation
// - Response wrapping
// - Caching
// - Logging with timing
// - Exception transformation
// - Timeout handling
// - Metrics collection

@Controller('users')
@UseInterceptors(
  LoggingInterceptor,
  CacheInterceptor,
  TransformInterceptor,
)
export class UsersController {}
```

**Interview Tip**: **Middleware** runs before routing, can't transform responses, no RxJS, no handler context. **Interceptors** run before/after handler, can transform responses with RxJS, have ExecutionContext. Use middleware for request preprocessing (CORS, auth). Use interceptors for response transformation, caching, timing.

</details>

### 30. What is the difference between Interceptors and Guards?

<details>
<summary>Answer</summary>

**Interceptors** transform requests/responses while **Guards** determine if requests are allowed to proceed.

**Comparison Table:**

| Feature | Guards | Interceptors |
|---------|--------|--------------|
| **Purpose** | Authorization | Transformation |
| **Return Type** | boolean/Promise/Observable | Observable |
| **Execute** | Before Interceptors | Before & After handler |
| **Can Block Request** | ✅ Yes (return false) | ✅ Yes (throw error) |
| **Transform Response** | ❌ No | ✅ Yes |
| **RxJS Operators** | ❌ No | ✅ Yes |
| **Use Case** | Auth, permissions | Logging, caching, wrapping |

**Request Lifecycle:**
```
Request
  ↓
Middleware
  ↓
1. Guards ← Check if allowed
  ↓
2. Interceptors (before)
  ↓
Pipes
  ↓
Handler
  ↓
3. Interceptors (after) ← Transform response
  ↓
Response
```

**Guard Example (Authorization):**
```typescript
import { CanActivate, ExecutionContext } from '@nestjs/common';

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    
    // Decide if request can proceed
    if (!request.headers.authorization) {
      return false; // Block request
    }
    
    return true; // Allow request
  }
}

// Returns boolean or throws exception
// Cannot transform response
```

**Interceptor Example (Transformation):**
```typescript
@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    // Can't block request based on authorization
    // Transforms response
    
    return next.handle().pipe(
      map(data => ({
        success: true,
        data,
      })),
    );
  }
}

// Returns Observable
// Can transform response with RxJS
```

**Guards: Block Unauthorized Requests:**
```typescript
@Injectable()
export class RoleGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    
    // ✅ Block based on role
    if (user.role !== 'admin') {
      throw new ForbiddenException('Admin only');
    }
    
    return true;
  }
}

@Controller('admin')
@UseGuards(RoleGuard)
export class AdminController {
  // Only admins can access
}
```

**Interceptors: Cannot Block (But Can Throw):**
```typescript
@Injectable()
export class NotForBlockingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    
    // ❌ Bad practice - use Guard instead
    if (!request.headers.authorization) {
      throw new UnauthorizedException();
    }
    
    // ✅ Good - transform response
    return next.handle().pipe(
      map(data => ({ success: true, data })),
    );
  }
}
```

**Guards: Run Before Interceptors:**
```typescript
@Controller('users')
@UseGuards(AuthGuard) // 1. Check auth
@UseInterceptors(LoggingInterceptor) // 2. Log if allowed
export class UsersController {
  @Get()
  findAll() {
    return [];
  }
}

// If Guard returns false, Interceptor never runs
```

**Combined Usage:**
```typescript
@Controller('posts')
@UseGuards(AuthGuard) // 1. Check if authenticated
@UseGuards(RoleGuard) // 2. Check if has required role
@UseInterceptors(LoggingInterceptor) // 3. Log request
@UseInterceptors(CacheInterceptor) // 4. Check cache
@UseInterceptors(TransformInterceptor) // 5. Transform response
export class PostsController {
  @Get()
  findAll() {
    return this.postsService.findAll();
  }
}
```

**When to Use Guards:**
```typescript
// ✅ Use Guards for:
// - Authentication (is user logged in?)
// - Authorization (does user have permission?)
// - Role-based access control
// - IP whitelisting
// - Rate limiting
// - Feature flags

@Injectable()
export class PermissionGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    const requiredPermission = 'read:users';
    
    return user.permissions.includes(requiredPermission);
  }
}
```

**When to Use Interceptors:**
```typescript
// ✅ Use Interceptors for:
// - Logging with timing
// - Response transformation
// - Caching
// - Timeout handling
// - Error transformation
// - Adding metadata to response

@Injectable()
export class MetadataInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    
    return next.handle().pipe(
      map(data => ({
        data,
        meta: {
          timestamp: Date.now(),
          path: request.url,
        },
      })),
    );
  }
}
```

**Guard + Interceptor Together:**
```typescript
// Guard checks authorization
@Injectable()
export class AdminGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    return request.user?.role === 'admin';
  }
}

// Interceptor adds metadata
@Injectable()
export class AdminAuditInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    
    return next.handle().pipe(
      tap(() => {
        console.log(`Admin ${request.user.id} accessed ${request.url}`);
      }),
    );
  }
}

// Usage
@Controller('admin')
@UseGuards(AdminGuard) // Block non-admins
@UseInterceptors(AdminAuditInterceptor) // Log admin actions
export class AdminController {}
```

**Interview Tip**: **Guards** control access (return boolean). **Interceptors** transform data (return Observable). Guards run before Interceptors. Use Guards for auth/authorization. Use Interceptors for logging/caching/transformation. Guards decide "can execute?", Interceptors handle "what to do before/after execution?".

</details>

### 31. When should you use Interceptors vs Middleware vs Guards?

<details>
<summary>Answer</summary>

Choose based on what you need to do and when in the request lifecycle.

**Decision Flow:**
```
Need to control access (auth/permissions)?
  → Use GUARDS

Need to parse/modify request body?
  → Use MIDDLEWARE

Need to transform response?
  → Use INTERCEPTORS

Need simple request logging?
  → Use MIDDLEWARE

Need logging with timing/response?
  → Use INTERCEPTORS

Need CORS, helmet, compression?
  → Use MIDDLEWARE

Need caching?
  → Use INTERCEPTORS
```

**Use Middleware When:**
```typescript
// ✅ Request preprocessing
// ✅ Third-party Express/Fastify middleware
// ✅ No need for response transformation
// ✅ Framework-agnostic logic

@Module({})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(
        // CORS configuration
        cors(),
        
        // Security headers
        helmet(),
        
        // Request logging
        morgan('combined'),
        
        // Body parsing
        json({ limit: '10mb' }),
        
        // Cookie parsing
        cookieParser(),
        
        // Session handling
        session({ secret: 'secret' }),
      )
      .forRoutes('*');
  }
}

// Examples:
// - CORS setup
// - Security (helmet)
// - Body/cookie parsing
// - Session management
// - Static files
// - Request logging (simple)
```

**Use Guards When:**
```typescript
// ✅ Authentication (is user logged in?)
// ✅ Authorization (can user do this?)
// ✅ Role-based access control
// ✅ Blocking requests
// ✅ Need handler metadata

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    return !!request.user;
  }
}

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}
  
  canActivate(context: ExecutionContext): boolean {
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    const user = context.switchToHttp().getRequest().user;
    return roles.some(role => user.roles.includes(role));
  }
}

// Examples:
// - JWT authentication
// - Role checking (admin, user)
// - Permission verification
// - IP whitelisting
// - Rate limiting
// - Feature flags
```

**Use Interceptors When:**
```typescript
// ✅ Response transformation
// ✅ Caching
// ✅ Logging with timing
// ✅ Exception transformation
// ✅ Need RxJS operators
// ✅ Timeout handling

@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => ({
        success: true,
        data,
        timestamp: Date.now(),
      })),
    );
  }
}

@Injectable()
export class CacheInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const cached = this.checkCache();
    if (cached) return of(cached);
    
    return next.handle().pipe(
      tap(data => this.saveToCache(data)),
    );
  }
}

// Examples:
// - Response wrapping
// - Caching responses
// - Logging with duration
// - Timeout handling
// - Error transformation
// - Adding metadata
```

**Complete Example:**
```typescript
// Middleware: CORS, security, parsing
@Module({})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(cors(), helmet(), json())
      .forRoutes('*');
  }
}

// Guards: Auth and authorization
@Controller('posts')
@UseGuards(AuthGuard, RolesGuard)
export class PostsController {
  
  // Interceptors: Logging, caching, transformation
  @Get()
  @UseInterceptors(
    LoggingInterceptor,
    CacheInterceptor,
    TransformInterceptor,
  )
  @Roles('admin', 'editor')
  findAll() {
    return this.postsService.findAll();
  }
  
  @Post()
  @UseInterceptors(LoggingInterceptor, TransformInterceptor)
  @Roles('admin')
  create(@Body() dto: CreatePostDto) {
    return this.postsService.create(dto);
  }
}

// Request flow:
// 1. Middleware: CORS, security, parsing
// 2. Guards: AuthGuard (authenticated?), RolesGuard (has role?)
// 3. Interceptors (before): LoggingInterceptor, CacheInterceptor
// 4. Handler executes
// 5. Interceptors (after): TransformInterceptor
```

**Comparison Matrix:**

| Task | Middleware | Guards | Interceptors |
|------|-----------|--------|--------------|
| CORS | ✅ | ❌ | ❌ |
| Authentication | ⚠️ | ✅ | ❌ |
| Authorization | ❌ | ✅ | ❌ |
| Body Parsing | ✅ | ❌ | ❌ |
| Request Logging | ✅ | ❌ | ⚠️ |
| Response Logging | ❌ | ❌ | ✅ |
| Response Transform | ❌ | ❌ | ✅ |
| Caching | ❌ | ❌ | ✅ |
| Timeout | ❌ | ❌ | ✅ |
| Exception Transform | ❌ | ❌ | ✅ |

**Real-World Scenario:**
```typescript
// Middleware: Global request setup
@Module({})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(
        cors({ origin: 'https://example.com' }),
        helmet(),
        json({ limit: '10mb' }),
        LoggerMiddleware, // Simple request logging
      )
      .forRoutes('*');
  }
}

// Guards: Protect routes
@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {}

@Injectable()
export class AdminGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    return context.switchToHttp().getRequest().user?.role === 'admin';
  }
}

// Interceptors: Transform and enhance
@Injectable()
export class ResponseWrapperInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => ({ success: true, data, timestamp: Date.now() })),
    );
  }
}

// Usage
@Controller('api/users')
@UseGuards(JwtAuthGuard) // Must be authenticated
@UseInterceptors(
  LoggingInterceptor, // Log with timing
  ResponseWrapperInterceptor, // Wrap response
)
export class UsersController {
  @Get()
  findAll() {
    return this.usersService.findAll();
  }
  
  @Post()
  @UseGuards(AdminGuard) // Must be admin
  create(@Body() dto: CreateUserDto) {
    return this.usersService.create(dto);
  }
}
```

**Interview Tip**: Use **Middleware** for request preprocessing (CORS, parsing, simple logging). Use **Guards** for access control (auth, permissions). Use **Interceptors** for response handling (transformation, caching, timing). Order: Middleware → Guards → Interceptors(before) → Handler → Interceptors(after). Choose based on: when to run, what to access, what to modify.

</details>
