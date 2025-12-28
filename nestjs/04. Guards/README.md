# NestJS Guards - Top Interview Questions

## Guard Fundamentals

### 1. What is a Guard in NestJS and what is its purpose?

<details>
<summary>Answer</summary>

A Guard determines if a request should be **handled by the route handler** or rejected.

**Purpose:**
- **Authorization** - Check if user has permission
- **Authentication** - Verify user identity
- **Conditional access** - Allow/deny based on metadata
- **Route protection** - Secure endpoints

**Key Characteristics:**
- Implements `CanActivate` interface
- Returns `boolean` or `Promise<boolean>` or `Observable<boolean>`
- `true` = allow request to proceed
- `false` or exception = deny request
- Executes after middleware, before interceptors
- Has access to `ExecutionContext` (metadata, request, etc.)

**Simple Example:**
```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    return !!request.user;  // true if user exists, false otherwise
  }
}

// Usage
@Controller('users')
export class UsersController {
  @Get()
  @UseGuards(AuthGuard)  // Protected route
  findAll() {
    return [];
  }
}
```

**Request Lifecycle:**
```
Client Request
    ↓
Middleware (authenticate, attach user)
    ↓
Guards (authorize, check permissions) ← Guards run here
    ↓
Interceptors (before)
    ↓
Pipes (validate)
    ↓
Route Handler
    ↓
Interceptors (after)
    ↓
Response
```

**Common Use Cases:**
```typescript
// Authentication - Is user logged in?
@UseGuards(AuthGuard)

// Authorization - Does user have permission?
@UseGuards(RolesGuard)

// Combined - Both authentication and authorization
@UseGuards(AuthGuard, RolesGuard)
```

**Interview Tip**: Guards control access to routes based on conditions (authentication, authorization, roles). They return `true` to allow or `false`/throw to deny. Execute after middleware, before interceptors. Perfect for implementing authentication and authorization logic.

</details>

### 2. When are Guards executed in the request lifecycle?

<details>
<summary>Answer</summary>

Guards execute **after middleware** but **before interceptors, pipes, and handlers**.

**Execution Order:**
```
1. Middleware(s)           // Authenticate, set req.user
2. Guards                  // ← Guards execute HERE (authorize)
3. Interceptors (before)   // Transform request
4. Pipes                   // Validate/transform parameters
5. Route Handler           // Execute business logic
6. Interceptors (after)    // Transform response
7. Exception Filters       // Handle errors
```

**Visual Flow:**
```typescript
HTTP Request
    ↓
[Middleware] - Authenticate user, set req.user
    ↓
[Guards] - Check if user has permission ← GUARDS HERE
    ↓
[Interceptors (before)] - Log, transform
    ↓
[Pipes] - Validate DTOs
    ↓
[Route Handler] - Business logic
    ↓
[Interceptors (after)] - Transform response
    ↓
[Exception Filters] - Handle errors
    ↓
HTTP Response
```

**Example:**
```typescript
// Middleware - Runs FIRST
@Injectable()
export class AuthMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log('1. Middleware: Authenticating user');
    req['user'] = { id: 1, role: 'admin' };
    next();
  }
}

// Guard - Runs SECOND
@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    console.log('2. Guard: Checking roles');
    const request = context.switchToHttp().getRequest();
    return request.user?.role === 'admin';
  }
}

// Interceptor - Runs THIRD
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler) {
    console.log('3. Interceptor: Before handler');
    return next.handle().pipe(
      tap(() => console.log('5. Interceptor: After handler'))
    );
  }
}

// Handler - Runs FOURTH
@Controller('admin')
export class AdminController {
  @Get()
  @UseGuards(RolesGuard)
  @UseInterceptors(LoggingInterceptor)
  getData() {
    console.log('4. Handler: Processing request');
    return { data: 'Admin data' };
  }
}

// Output:
// 1. Middleware: Authenticating user
// 2. Guard: Checking roles
// 3. Interceptor: Before handler
// 4. Handler: Processing request
// 5. Interceptor: After handler
```

**Interview Tip**: Guards run after middleware (which authenticates) but before interceptors/pipes/handlers. Order: Middleware → **Guards** → Interceptors → Pipes → Handler. Use middleware to authenticate, guards to authorize.

</details>

### 3. What is the `CanActivate` interface?

<details>
<summary>Answer</summary>

`CanActivate` is the interface that Guards must implement with a `canActivate()` method.

**Interface Definition:**
```typescript
interface CanActivate {
  canActivate(context: ExecutionContext): boolean | Promise<boolean> | Observable<boolean>;
}
```

**Basic Implementation:**
```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    return !!request.user;  // true = allow, false = deny
  }
}
```

**Async Implementation:**
```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private authService: AuthService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = request.headers.authorization?.split(' ')[1];
    
    if (!token) {
      return false;
    }
    
    const isValid = await this.authService.validateToken(token);
    return isValid;
  }
}
```

**Observable Implementation:**
```typescript
import { Observable, of } from 'rxjs';

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): Observable<boolean> {
    const request = context.switchToHttp().getRequest();
    return of(!!request.user);
  }
}
```

**With Exception:**
```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    
    if (!request.user) {
      throw new UnauthorizedException('User not authenticated');
    }
    
    return true;
  }
}
```

**Interview Tip**: `CanActivate` is the interface for Guards with a `canActivate(context)` method. Return `true` to allow, `false` to deny, or throw an exception. Can return synchronous boolean, Promise, or Observable.

</details>

### 4. What should a Guard return (boolean or Observable/Promise)?

<details>
<summary>Answer</summary>

Guards can return: `boolean`, `Promise<boolean>`, or `Observable<boolean>`.

**Synchronous (boolean):**
```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    return !!request.user;  // Immediate boolean
  }
}
```

**Asynchronous (Promise<boolean>):**
```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private authService: AuthService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = request.headers.authorization?.split(' ')[1];
    
    // Async operation
    const isValid = await this.authService.validateToken(token);
    return isValid;  // Promise<boolean>
  }
}
```

**Observable (Observable<boolean>):**
```typescript
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private authService: AuthService) {}

  canActivate(context: ExecutionContext): Observable<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = request.headers.authorization?.split(' ')[1];
    
    return this.authService.validateToken$(token).pipe(
      map(isValid => isValid)
    );
  }
}
```

**Return Values:**
- `true` → Request proceeds to handler
- `false` → Request denied (403 Forbidden)
- Throw exception → Custom error (401 Unauthorized, etc.)

**Best Practices:**
```typescript
// ✅ Return true to allow
return true;

// ✅ Return false to deny (403 Forbidden)
return false;

// ✅ Throw for custom error message
throw new UnauthorizedException('Invalid token');

// ✅ Use async for database/API calls
async canActivate(context: ExecutionContext): Promise<boolean> {
  const user = await this.userService.findById(userId);
  return user?.isActive ?? false;
}
```

**Interview Tip**: Guards return `boolean` (sync), `Promise<boolean>` (async), or `Observable<boolean>` (RxJS). `true` = allow, `false` = deny with 403. Throw exceptions for custom error messages like 401 Unauthorized.

</details>

### 5. Can Guards be async?

<details>
<summary>Answer</summary>

Yes, Guards can be **async** and perform asynchronous operations.

**Async Guard:**
```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  constructor(
    private authService: AuthService,
    private userService: UserService,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = request.headers.authorization?.split(' ')[1];
    
    if (!token) {
      return false;
    }
    
    // Async operations
    const payload = await this.authService.validateToken(token);
    const user = await this.userService.findById(payload.sub);
    
    if (!user || !user.isActive) {
      return false;
    }
    
    request.user = user;
    return true;
  }
}
```

**Database Query:**
```typescript
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(
    private userService: UserService,
    private reflector: Reflector,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    const request = context.switchToHttp().getRequest();
    const userId = request.user?.id;
    
    // Async database query
    const user = await this.userService.findById(userId);
    
    return roles.includes(user.role);
  }
}
```

**With Caching:**
```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  constructor(
    private authService: AuthService,
    private cacheService: CacheService,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = request.headers.authorization?.split(' ')[1];
    
    // Check cache
    const cacheKey = `auth:${token}`;
    let user = await this.cacheService.get(cacheKey);
    
    if (!user) {
      // Validate token
      const payload = await this.authService.validateToken(token);
      user = payload;
      
      // Cache for 5 minutes
      await this.cacheService.set(cacheKey, user, 300);
    }
    
    request.user = user;
    return true;
  }
}
```

**Multiple Async Operations:**
```typescript
@Injectable()
export class PermissionGuard implements CanActivate {
  constructor(
    private userService: UserService,
    private permissionService: PermissionService,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const userId = request.user?.id;
    const resource = context.getClass().name;
    const action = context.getHandler().name;
    
    // Multiple async operations
    const [user, permissions] = await Promise.all([
      this.userService.findById(userId),
      this.permissionService.getUserPermissions(userId),
    ]);
    
    return permissions.includes(`${resource}:${action}`);
  }
}
```

**Interview Tip**: Guards can be async using `async/await` with `Promise<boolean>`. Use for database queries, API calls, cache lookups, and token validation. NestJS waits for the Promise to resolve before proceeding.

</details>

## Creating Guards

### 6. How do you create a Guard implementing `CanActivate`?

<details>
<summary>Answer</summary>

Create a class with `@Injectable()` that implements `CanActivate` interface.

**Basic Guard:**
```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    return !!request.user;  // Check if user exists
  }
}
```

**With Dependency Injection:**
```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  constructor(
    private authService: AuthService,
    private logger: LoggerService,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = request.headers.authorization?.split(' ')[1];
    
    if (!token) {
      this.logger.warn('No token provided');
      return false;
    }
    
    try {
      const user = await this.authService.validateToken(token);
      request.user = user;
      return true;
    } catch (error) {
      this.logger.error('Token validation failed', error);
      return false;
    }
  }
}
```

**Complete Example:**
```typescript
import {
  Injectable,
  CanActivate,
  ExecutionContext,
  UnauthorizedException,
} from '@nestjs/common';

@Injectable()
export class JwtAuthGuard implements CanActivate {
  constructor(
    private jwtService: JwtService,
    private userService: UserService,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = this.extractToken(request);
    
    if (!token) {
      throw new UnauthorizedException('No token provided');
    }
    
    try {
      // Verify token
      const payload = await this.jwtService.verifyAsync(token);
      
      // Load user
      const user = await this.userService.findById(payload.sub);
      
      if (!user || !user.isActive) {
        throw new UnauthorizedException('User not found or inactive');
      }
      
      // Attach user to request
      request.user = user;
      return true;
    } catch (error) {
      throw new UnauthorizedException('Invalid token');
    }
  }
  
  private extractToken(request: any): string | null {
    const authHeader = request.headers.authorization;
    if (authHeader && authHeader.startsWith('Bearer ')) {
      return authHeader.substring(7);
    }
    return null;
  }
}

// Apply to route
@Controller('users')
export class UsersController {
  @Get()
  @UseGuards(JwtAuthGuard)
  findAll() {
    return [];
  }
}
```

**Interview Tip**: Create Guard with `@Injectable()` implementing `CanActivate`. Use dependency injection for services. Return `true` to allow, `false`/throw to deny. Apply with `@UseGuards()`.

</details>

### 7. What is the `ExecutionContext` in Guards?

<details>
<summary>Answer</summary>

`ExecutionContext` provides details about the current request and execution environment.

**What It Provides:**
- Access to request/response objects
- Route handler metadata
- Controller class information
- HTTP method, URL, headers
- Custom metadata set by decorators

**Common Methods:**
```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    // Get HTTP context
    const request = context.switchToHttp().getRequest();
    const response = context.switchToHttp().getResponse();
    
    // Get handler (method) and class (controller)
    const handler = context.getHandler();
    const controller = context.getClass();
    
    // Get handler/controller names
    const handlerName = handler.name;
    const controllerName = controller.name;
    
    return true;
  }
}
```

**Accessing Request Data:**
```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    
    // Access request properties
    const method = request.method;          // GET, POST, etc.
    const url = request.url;                // /users/123
    const headers = request.headers;        // All headers
    const body = request.body;              // Request body
    const params = request.params;          // Route params
    const query = request.query;            // Query params
    const user = request.user;              // User (if set by middleware)
    
    return !!user;
  }
}
```

**With Reflector for Metadata:**
```typescript
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // Get metadata from handler or class
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    
    if (!roles) {
      return true;  // No roles required
    }
    
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    
    return roles.includes(user.role);
  }
}
```

**Complete Example:**
```typescript
@Injectable()
export class AdvancedGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // HTTP context
    const request = context.switchToHttp().getRequest();
    const response = context.switchToHttp().getResponse();
    
    // Handler and class info
    const handler = context.getHandler();
    const controller = context.getClass();
    
    // Metadata
    const isPublic = this.reflector.get<boolean>('isPublic', handler);
    const roles = this.reflector.get<string[]>('roles', handler);
    
    // Request data
    const { method, url, user } = request;
    
    console.log(`${method} ${url} by ${user?.email}`);
    console.log(`Handler: ${handler.name}, Controller: ${controller.name}`);
    console.log(`Public: ${isPublic}, Roles: ${roles}`);
    
    if (isPublic) {
      return true;
    }
    
    if (!user) {
      return false;
    }
    
    if (roles && !roles.includes(user.role)) {
      return false;
    }
    
    return true;
  }
}
```

**Interview Tip**: `ExecutionContext` provides access to request/response via `switchToHttp()`, handler/class info via `getHandler()`/`getClass()`, and metadata via Reflector. Use to inspect request and make authorization decisions.

</details>

### 8. How do you access the request object in a Guard?

<details>
<summary>Answer</summary>

Use `context.switchToHttp().getRequest()` to access the request object.

**Basic Access:**
```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    // Get request object
    const request = context.switchToHttp().getRequest();
    
    return !!request.user;
  }
}
```

**Accessing Request Properties:**
```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    
    // All request properties
    const method = request.method;              // 'GET'
    const url = request.url;                    // '/users/123'
    const originalUrl = request.originalUrl;    // Full URL
    const path = request.path;                  // Path only
    const headers = request.headers;            // Headers object
    const body = request.body;                  // Request body
    const params = request.params;              // Route params { id: '123' }
    const query = request.query;                // Query params { page: '1' }
    const ip = request.ip;                      // Client IP
    const user = request.user;                  // User (if set)
    
    return !!user;
  }
}
```

**Extract Token from Headers:**
```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    
    // Extract Authorization header
    const authHeader = request.headers.authorization;
    const token = authHeader?.split(' ')[1];
    
    return !!token;
  }
}
```

**Attach Data to Request:**
```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private authService: AuthService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = request.headers.authorization?.split(' ')[1];
    
    if (!token) {
      return false;
    }
    
    const user = await this.authService.validateToken(token);
    
    // Attach user to request for later use
    request.user = user;
    request.token = token;
    
    return true;
  }
}
```

**Typed Request:**
```typescript
interface RequestWithUser extends Request {
  user: {
    id: string;
    email: string;
    role: string;
  };
}

@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest<RequestWithUser>();
    
    // Now request.user is typed
    const userRole = request.user.role;
    
    return userRole === 'admin';
  }
}
```

**Interview Tip**: Access request with `context.switchToHttp().getRequest()`. Get headers, body, params, query, user, etc. Use to extract tokens, check user info, and make authorization decisions. Can attach data to request for downstream use.

</details>

## Applying Guards

### 9. How do you apply a Guard to a route handler using `@UseGuards()`?

<details>
<summary>Answer</summary>

Use `@UseGuards()` decorator on a route handler to apply Guards.

**Basic Usage:**
```typescript
@Controller('users')
export class UsersController {
  @Get()
  @UseGuards(AuthGuard)  // Apply to this route
  findAll() {
    return [];
  }
}
```

**Multiple Guards:**
```typescript
@Controller('users')
export class UsersController {
  @Get()
  @UseGuards(AuthGuard, RolesGuard)  // Execute in order
  findAll() {
    return [];
  }
}
```

**With Custom Decorators:**
```typescript
@Controller('admin')
export class AdminController {
  @Get('users')
  @UseGuards(AuthGuard, RolesGuard)
  @Roles('admin', 'superadmin')
  getUsers() {
    return [];
  }
}
```

**Different Guards for Different Routes:**
```typescript
@Controller('posts')
export class PostsController {
  @Get()
  findAll() {
    return [];  // Public - no guard
  }
  
  @Get(':id')
  @UseGuards(AuthGuard)  // Auth required
  findOne(@Param('id') id: string) {
    return {};
  }
  
  @Post()
  @UseGuards(AuthGuard, RolesGuard)  // Auth + Role check
  @Roles('admin', 'editor')
  create(@Body() dto: CreatePostDto) {
    return {};
  }
  
  @Delete(':id')
  @UseGuards(AuthGuard, RolesGuard)
  @Roles('admin')
  remove(@Param('id') id: string) {
    return {};
  }
}
```

**Instantiate Guard (Pass Options):**
```typescript
@Controller('users')
export class UsersController {
  @Get()
  @UseGuards(new RoleGuard(['admin', 'user']))  // Instantiate with options
  findAll() {
    return [];
  }
}
```

**Interview Tip**: Apply Guards with `@UseGuards(GuardClass)` on route handlers. Can apply multiple Guards - they execute in order. Use for protecting specific routes with authentication and authorization.

</details>

### 10. How do you apply a Guard to an entire controller?

<details>
<summary>Answer</summary>

Use `@UseGuards()` decorator on the controller class to protect all routes.

**Controller-Level Guard:**
```typescript
@Controller('users')
@UseGuards(AuthGuard)  // All routes require authentication
export class UsersController {
  @Get()
  findAll() {}  // Protected
  
  @Get(':id')
  findOne(@Param('id') id: string) {}  // Protected
  
  @Post()
  create(@Body() dto: CreateUserDto) {}  // Protected
}
```

**Multiple Guards on Controller:**
```typescript
@Controller('admin')
@UseGuards(AuthGuard, RolesGuard)
@Roles('admin')  // All routes require admin role
export class AdminController {
  @Get('users')
  getUsers() {}  // Admin only
  
  @Delete('users/:id')
  deleteUser(@Param('id') id: string) {}  // Admin only
}
```

**Override at Route Level:**
```typescript
@Controller('posts')
@UseGuards(AuthGuard)  // Default: auth required
export class PostsController {
  @Get()
  @Public()  // Override - make this route public
  findAll() {}
  
  @Get(':id')
  findOne(@Param('id') id: string) {}  // Uses controller guard (auth required)
  
  @Post()
  @UseGuards(RolesGuard)  // Additional guard for this route
  @Roles('admin')
  create(@Body() dto: CreatePostDto) {}  // Auth (controller) + Role (route)
}
```

**Mixed Approach:**
```typescript
@Controller('products')
@UseGuards(AuthGuard)  // Auth for all routes
export class ProductsController {
  @Get()
  @Public()  // Public route
  findAll() {}
  
  @Get(':id')
  findOne(@Param('id') id: string) {}  // Auth required
  
  @Post()
  @UseGuards(RolesGuard)  // Auth + Roles
  @Roles('admin', 'editor')
  create(@Body() dto: CreateProductDto) {}
  
  @Delete(':id')
  @UseGuards(RolesGuard)  // Auth + Roles
  @Roles('admin')
  remove(@Param('id') id: string) {}
}
```

**Interview Tip**: Apply `@UseGuards()` on controller class to protect all routes. Can override at route level with additional guards or `@Public()` decorator. Guards on both controller and route are combined.

</details>
### 11. How do you apply a Guard globally using `APP_GUARD`?

<details>
<summary>Answer</summary>

Use `APP_GUARD` provider to apply Guards globally to all routes.

**Global Guard Setup:**
```typescript
import { Module } from '@nestjs/common';
import { APP_GUARD } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: AuthGuard,  // Apply to ALL routes
    },
  ],
})
export class AppModule {}
```

**Multiple Global Guards:**
```typescript
@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: AuthGuard,  // Executes first
    },
    {
      provide: APP_GUARD,
      useClass: RolesGuard,  // Executes second
    },
  ],
})
export class AppModule {}
```

**With Public Routes:**
```typescript
// auth.guard.ts
@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // Check if route is public
    const isPublic = this.reflector.get<boolean>(
      'isPublic',
      context.getHandler(),
    );
    
    if (isPublic) {
      return true;  // Skip authentication
    }
    
    // Check authentication
    const request = context.switchToHttp().getRequest();
    return !!request.user;
  }
}

// public.decorator.ts
export const Public = () => SetMetadata('isPublic', true);

// app.module.ts
@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: AuthGuard,
    },
  ],
})
export class AppModule {}

// Usage
@Controller('posts')
export class PostsController {
  @Get()
  @Public()  // Bypass global AuthGuard
  findAll() {}
  
  @Post()
  create(@Body() dto: CreatePostDto) {}  // Protected by global guard
}
```

**Interview Tip**: Use `APP_GUARD` provider in module to apply Guards globally. Access via dependency injection. Use `@Public()` decorator with Reflector to skip authentication for specific routes.

</details>

### 12. Can you apply multiple Guards to a single route?

<details>
<summary>Answer</summary>

Yes, apply multiple Guards - they execute in order from left to right.

**Multiple Guards:**
```typescript
@Controller('users')
export class UsersController {
  @Get()
  @UseGuards(AuthGuard, RolesGuard, ThrottleGuard)  // Execute in order
  findAll() {
    return [];
  }
}
```

**Execution Order:**
```typescript
// AuthGuard executes FIRST
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    console.log('1. AuthGuard');
    const request = context.switchToHttp().getRequest();
    return !!request.user;
  }
}

// RolesGuard executes SECOND
@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    console.log('2. RolesGuard');
    const request = context.switchToHttp().getRequest();
    return request.user?.role === 'admin';
  }
}

// ThrottleGuard executes THIRD
@Injectable()
export class ThrottleGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    console.log('3. ThrottleGuard');
    return true;
  }
}

@Controller('admin')
export class AdminController {
  @Get()
  @UseGuards(AuthGuard, RolesGuard, ThrottleGuard)
  getData() {
    console.log('4. Handler');
    return {};
  }
}

// Output:
// 1. AuthGuard
// 2. RolesGuard
// 3. ThrottleGuard
// 4. Handler
```

**Combined with Controller Guards:**
```typescript
@Controller('posts')
@UseGuards(AuthGuard)  // Controller-level
export class PostsController {
  @Get()
  @UseGuards(RolesGuard)  // Route-level
  @Roles('admin')
  findAll() {}
  
  // Execution order:
  // 1. AuthGuard (controller)
  // 2. RolesGuard (route)
}
```

**Interview Tip**: Apply multiple Guards with `@UseGuards(Guard1, Guard2, ...)`. They execute left to right. Controller guards execute before route guards. If any guard returns `false` or throws, request is denied.

</details>

## Reflector & Metadata

### 13. What is the `Reflector` class and how do you use it in Guards?

<details>
<summary>Answer</summary>

`Reflector` reads metadata attached to routes by decorators like `@SetMetadata()`.

**Basic Usage:**
```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}  // Inject Reflector

  canActivate(context: ExecutionContext): boolean {
    // Read 'roles' metadata from handler
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    
    if (!roles) {
      return true;  // No roles required
    }
    
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    
    return roles.includes(user?.role);
  }
}
```

**Read from Handler or Class:**
```typescript
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // Read from handler (method)
    const handlerRoles = this.reflector.get<string[]>(
      'roles',
      context.getHandler(),
    );
    
    // Read from class (controller)
    const classRoles = this.reflector.get<string[]>(
      'roles',
      context.getClass(),
    );
    
    // Combine both
    const roles = [...(classRoles || []), ...(handlerRoles || [])];
    
    if (!roles.length) {
      return true;
    }
    
    const request = context.switchToHttp().getRequest();
    return roles.includes(request.user?.role);
  }
}
```

**getAllAndOverride (Handler Overrides Class):**
```typescript
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // Handler metadata overrides class metadata
    const roles = this.reflector.getAllAndOverride<string[]>('roles', [
      context.getHandler(),  // Check handler first
      context.getClass(),    // Then class
    ]);
    
    if (!roles) {
      return true;
    }
    
    const request = context.switchToHttp().getRequest();
    return roles.includes(request.user?.role);
  }
}
```

**getAllAndMerge (Combine Handler and Class):**
```typescript
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // Merge handler and class metadata
    const roles = this.reflector.getAllAndMerge<string[]>('roles', [
      context.getHandler(),
      context.getClass(),
    ]);
    
    if (!roles.length) {
      return true;
    }
    
    const request = context.switchToHttp().getRequest();
    return roles.some(role => request.user?.role === role);
  }
}
```

**Interview Tip**: Inject `Reflector` to read metadata from decorators. Use `reflector.get(key, target)` for single source, `getAllAndOverride()` for override behavior, `getAllAndMerge()` to combine metadata.

</details>

### 14. What is `@SetMetadata()` decorator?

<details>
<summary>Answer</summary>

`@SetMetadata()` attaches custom metadata to routes that Guards can read.

**Basic Usage:**
```typescript
import { SetMetadata } from '@nestjs/common';

@Controller('users')
export class UsersController {
  @Get()
  @SetMetadata('roles', ['admin', 'user'])  // Attach metadata
  findAll() {
    return [];
  }
}
```

**Read in Guard:**
```typescript
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // Read metadata
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    
    if (!roles) {
      return true;
    }
    
    const request = context.switchToHttp().getRequest();
    return roles.includes(request.user?.role);
  }
}
```

**Multiple Metadata:**
```typescript
@Controller('posts')
export class PostsController {
  @Get()
  @SetMetadata('roles', ['admin', 'editor'])
  @SetMetadata('permissions', ['read:posts'])
  @SetMetadata('isPublic', false)
  findAll() {
    return [];
  }
}
```

**Interview Tip**: `@SetMetadata(key, value)` attaches metadata to routes. Guards read it with `Reflector`. Better to create custom decorators instead of using `@SetMetadata()` directly.

</details>

### 15. How do you create custom decorators like `@Roles()` or `@Public()`?

<details>
<summary>Answer</summary>

Wrap `@SetMetadata()` in custom decorators for better DX.

**@Roles() Decorator:**
```typescript
import { SetMetadata } from '@nestjs/common';

export const Roles = (...roles: string[]) => SetMetadata('roles', roles);

// Usage
@Controller('users')
export class UsersController {
  @Get()
  @Roles('admin', 'user')  // Cleaner than @SetMetadata
  findAll() {
    return [];
  }
}
```

**@Public() Decorator:**
```typescript
export const Public = () => SetMetadata('isPublic', true);

// Usage
@Controller('auth')
export class AuthController {
  @Post('login')
  @Public()  // Skip authentication
  login(@Body() dto: LoginDto) {
    return {};
  }
}
```

**@Permissions() Decorator:**
```typescript
export const Permissions = (...permissions: string[]) =>
  SetMetadata('permissions', permissions);

// Usage
@Controller('posts')
export class PostsController {
  @Get()
  @Permissions('read:posts')
  findAll() {}
  
  @Post()
  @Permissions('create:posts')
  create(@Body() dto: CreatePostDto) {}
  
  @Delete(':id')
  @Permissions('delete:posts')
  remove(@Param('id') id: string) {}
}
```

**Composite Decorators:**
```typescript
import { applyDecorators } from '@nestjs/common';

export const AdminOnly = () =>
  applyDecorators(
    Roles('admin'),
    Permissions('admin:all'),
  );

// Usage
@Controller('admin')
export class AdminController {
  @Get()
  @AdminOnly()  // Applies both Roles and Permissions
  getData() {}
}
```

**Interview Tip**: Create custom decorators wrapping `SetMetadata()` for cleaner code. Use `applyDecorators()` to combine multiple decorators into one.

</details>

### 16. How do you retrieve metadata in Guards using `Reflector.get()`?

<details>
<summary>Answer</summary>

Use `reflector.get(key, target)` to read metadata in Guards.

**Basic Retrieval:**
```typescript
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // Get metadata from handler
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    
    if (!roles) {
      return true;
    }
    
    const request = context.switchToHttp().getRequest();
    return roles.includes(request.user?.role);
  }
}
```

**From Handler or Class:**
```typescript
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // Try handler first, then class
    const roles = 
      this.reflector.get<string[]>('roles', context.getHandler()) ||
      this.reflector.get<string[]>('roles', context.getClass()) ||
      [];
    
    if (!roles.length) {
      return true;
    }
    
    const request = context.switchToHttp().getRequest();
    return roles.includes(request.user?.role);
  }
}
```

**Check if Public:**
```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // Check if route is public
    const isPublic = this.reflector.get<boolean>(
      'isPublic',
      context.getHandler(),
    );
    
    if (isPublic) {
      return true;  // Skip authentication
    }
    
    const request = context.switchToHttp().getRequest();
    return !!request.user;
  }
}
```

**Multiple Metadata Keys:**
```typescript
@Injectable()
export class PermissionsGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // Get multiple metadata keys
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    const permissions = this.reflector.get<string[]>(
      'permissions',
      context.getHandler(),
    );
    const isPublic = this.reflector.get<boolean>(
      'isPublic',
      context.getHandler(),
    );
    
    if (isPublic) {
      return true;
    }
    
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    
    // Check roles
    if (roles && !roles.includes(user?.role)) {
      return false;
    }
    
    // Check permissions
    if (permissions && !permissions.every(p => user?.permissions.includes(p))) {
      return false;
    }
    
    return true;
  }
}
```

**Interview Tip**: Use `reflector.get(key, target)` to read metadata. Check handler with `context.getHandler()`, class with `context.getClass()`. Return default value if metadata doesn't exist.

</details>

## Authentication Guards

### 17. How do you implement a JWT Authentication Guard?

<details>
<summary>Answer</summary>

Create Guard that validates JWT tokens from request headers.

**JWT Auth Guard:**
```typescript
import {
  Injectable,
  CanActivate,
  ExecutionContext,
  UnauthorizedException,
} from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';

@Injectable()
export class JwtAuthGuard implements CanActivate {
  constructor(private jwtService: JwtService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = this.extractToken(request);
    
    if (!token) {
      throw new UnauthorizedException('No token provided');
    }
    
    try {
      // Verify token
      const payload = await this.jwtService.verifyAsync(token, {
        secret: process.env.JWT_SECRET,
      });
      
      // Attach payload to request
      request.user = payload;
      return true;
    } catch (error) {
      throw new UnauthorizedException('Invalid token');
    }
  }
  
  private extractToken(request: any): string | null {
    const authHeader = request.headers.authorization;
    if (authHeader && authHeader.startsWith('Bearer ')) {
      return authHeader.substring(7);
    }
    return null;
  }
}
```

**With User Loading:**
```typescript
@Injectable()
export class JwtAuthGuard implements CanActivate {
  constructor(
    private jwtService: JwtService,
    private userService: UserService,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = this.extractToken(request);
    
    if (!token) {
      throw new UnauthorizedException('No token provided');
    }
    
    try {
      // Verify token
      const payload = await this.jwtService.verifyAsync(token);
      
      // Load full user from database
      const user = await this.userService.findById(payload.sub);
      
      if (!user || !user.isActive) {
        throw new UnauthorizedException('User not found or inactive');
      }
      
      // Attach user to request
      request.user = user;
      return true;
    } catch (error) {
      throw new UnauthorizedException('Invalid token');
    }
  }
  
  private extractToken(request: any): string | null {
    const authHeader = request.headers.authorization;
    return authHeader?.startsWith('Bearer ')
      ? authHeader.substring(7)
      : null;
  }
}

// Usage
@Controller('users')
export class UsersController {
  @Get()
  @UseGuards(JwtAuthGuard)
  findAll() {
    return [];
  }
}
```

**With Public Routes:**
```typescript
@Injectable()
export class JwtAuthGuard implements CanActivate {
  constructor(
    private jwtService: JwtService,
    private reflector: Reflector,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    // Check if route is public
    const isPublic = this.reflector.get<boolean>(
      'isPublic',
      context.getHandler(),
    );
    
    if (isPublic) {
      return true;
    }
    
    const request = context.switchToHttp().getRequest();
    const token = this.extractToken(request);
    
    if (!token) {
      throw new UnauthorizedException('No token provided');
    }
    
    try {
      const payload = await this.jwtService.verifyAsync(token);
      request.user = payload;
      return true;
    } catch (error) {
      throw new UnauthorizedException('Invalid token');
    }
  }
  
  private extractToken(request: any): string | null {
    const authHeader = request.headers.authorization;
    return authHeader?.startsWith('Bearer ')
      ? authHeader.substring(7)
      : null;
  }
}

// Apply globally
@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: JwtAuthGuard,
    },
  ],
})
export class AppModule {}

// Mark routes as public
@Controller('auth')
export class AuthController {
  @Post('login')
  @Public()
  login(@Body() dto: LoginDto) {}
}
```

**Interview Tip**: JWT Guard extracts token from `Authorization` header, verifies with `JwtService`, and attaches user to request. Throw `UnauthorizedException` for invalid tokens. Support public routes with `@Public()` decorator.

</details>

### 18. How do you validate JWT tokens in a Guard?

<details>
<summary>Answer</summary>

Use `JwtService.verifyAsync()` to validate JWT tokens.

**Basic Validation:**
```typescript
@Injectable()
export class JwtAuthGuard implements CanActivate {
  constructor(private jwtService: JwtService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = request.headers.authorization?.split(' ')[1];
    
    if (!token) {
      throw new UnauthorizedException('No token');
    }
    
    try {
      // Validate token
      const payload = await this.jwtService.verifyAsync(token);
      request.user = payload;
      return true;
    } catch (error) {
      throw new UnauthorizedException('Invalid token');
    }
  }
}
```

**With Secret:**
```typescript
async canActivate(context: ExecutionContext): Promise<boolean> {
  const token = this.extractToken(request);
  
  try {
    const payload = await this.jwtService.verifyAsync(token, {
      secret: process.env.JWT_SECRET,  // Specify secret
    });
    request.user = payload;
    return true;
  } catch (error) {
    throw new UnauthorizedException('Invalid token');
  }
}
```

**Check Expiration:**
```typescript
async canActivate(context: ExecutionContext): Promise<boolean> {
  const token = this.extractToken(request);
  
  try {
    const payload = await this.jwtService.verifyAsync(token);
    
    // Check if token is expired
    const now = Math.floor(Date.now() / 1000);
    if (payload.exp && payload.exp < now) {
      throw new UnauthorizedException('Token expired');
    }
    
    request.user = payload;
    return true;
  } catch (error) {
    if (error.name === 'TokenExpiredError') {
      throw new UnauthorizedException('Token expired');
    }
    throw new UnauthorizedException('Invalid token');
  }
}
```

**Interview Tip**: Use `jwtService.verifyAsync(token)` to validate JWT. Wrap in try/catch to handle invalid tokens. Check expiration with `payload.exp`. Throw `UnauthorizedException` for invalid/expired tokens.

</details>

### 19. How do you extract JWT from request headers?

<details>
<summary>Answer</summary>

Extract JWT from `Authorization: Bearer <token>` header.

**Extract Token:**
```typescript
private extractToken(request: any): string | null {
  const authHeader = request.headers.authorization;
  
  if (authHeader && authHeader.startsWith('Bearer ')) {
    return authHeader.substring(7);  // Remove 'Bearer '
  }
  
  return null;
}
```

**In Guard:**
```typescript
@Injectable()
export class JwtAuthGuard implements CanActivate {
  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = this.extractToken(request);
    
    if (!token) {
      throw new UnauthorizedException('No token provided');
    }
    
    // Validate token...
    return true;
  }
  
  private extractToken(request: any): string | null {
    const authHeader = request.headers.authorization;
    return authHeader?.startsWith('Bearer ')
      ? authHeader.substring(7)
      : null;
  }
}
```

**Multiple Sources (Header or Cookie):**
```typescript
private extractToken(request: any): string | null {
  // Try Authorization header first
  const authHeader = request.headers.authorization;
  if (authHeader && authHeader.startsWith('Bearer ')) {
    return authHeader.substring(7);
  }
  
  // Try cookie
  if (request.cookies && request.cookies['access_token']) {
    return request.cookies['access_token'];
  }
  
  // Try query param (not recommended for production)
  if (request.query && request.query.token) {
    return request.query.token;
  }
  
  return null;
}
```

**Interview Tip**: Extract JWT from `Authorization` header with `request.headers.authorization`. Check if it starts with `Bearer ` and extract the token part. Can also check cookies or query params as fallback.

</details>

### 20. How do you attach user information to the request?

<details>
<summary>Answer</summary>

Attach user to `request.user` after validating token.

**Basic Attach:**
```typescript
@Injectable()
export class JwtAuthGuard implements CanActivate {
  constructor(private jwtService: JwtService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = this.extractToken(request);
    
    const payload = await this.jwtService.verifyAsync(token);
    
    // Attach payload to request
    request.user = payload;  // { sub: '123', email: 'user@example.com' }
    
    return true;
  }
}
```

**Attach Full User Object:**
```typescript
@Injectable()
export class JwtAuthGuard implements CanActivate {
  constructor(
    private jwtService: JwtService,
    private userService: UserService,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = this.extractToken(request);
    
    const payload = await this.jwtService.verifyAsync(token);
    
    // Load full user from database
    const user = await this.userService.findById(payload.sub);
    
    // Attach full user object
    request.user = user;  // { id, email, role, permissions, ... }
    
    return true;
  }
}
```

**Access in Handler:**
```typescript
@Controller('profile')
export class ProfileController {
  @Get()
  @UseGuards(JwtAuthGuard)
  getProfile(@Req() request: Request) {
    // Access user attached by guard
    const user = request['user'];
    return user;
  }
  
  // Or use custom decorator
  @Get('me')
  @UseGuards(JwtAuthGuard)
  getMe(@CurrentUser() user: User) {
    return user;
  }
}
```

**Interview Tip**: After validating token, attach user to `request.user`. Can attach JWT payload or load full user from database. Access in handlers via `@Req()` or custom `@CurrentUser()` decorator.

</details>

### 21. How do you skip authentication for public routes?

<details>
<summary>Answer</summary>

Use `@Public()` decorator with Reflector to skip authentication.

**Public Decorator:**
```typescript
import { SetMetadata } from '@nestjs/common';

export const Public = () => SetMetadata('isPublic', true);
```

**Guard with Public Check:**
```typescript
@Injectable()
export class JwtAuthGuard implements CanActivate {
  constructor(
    private jwtService: JwtService,
    private reflector: Reflector,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    // Check if route is public
    const isPublic = this.reflector.get<boolean>(
      'isPublic',
      context.getHandler(),
    );
    
    if (isPublic) {
      return true;  // Skip authentication
    }
    
    // Normal authentication flow
    const request = context.switchToHttp().getRequest();
    const token = this.extractToken(request);
    
    if (!token) {
      throw new UnauthorizedException('No token');
    }
    
    const payload = await this.jwtService.verifyAsync(token);
    request.user = payload;
    return true;
  }
}
```

**Usage:**
```typescript
@Controller('auth')
export class AuthController {
  @Post('login')
  @Public()  // No authentication required
  login(@Body() dto: LoginDto) {
    return this.authService.login(dto);
  }
  
  @Post('register')
  @Public()
  register(@Body() dto: RegisterDto) {
    return this.authService.register(dto);
  }
}

@Controller('users')
export class UsersController {
  @Get()
  @Public()  // Public endpoint
  findAll() {
    return [];
  }
  
  @Get('profile')
  getProfile() {  // Protected (no @Public decorator)
    return {};
  }
}
```

**Check Handler or Class:**
```typescript
async canActivate(context: ExecutionContext): Promise<boolean> {
  // Check handler first, then class
  const isPublic = this.reflector.getAllAndOverride<boolean>('isPublic', [
    context.getHandler(),
    context.getClass(),
  ]);
  
  if (isPublic) {
    return true;
  }
  
  // Authentication logic...
}
```

**Interview Tip**: Create `@Public()` decorator with `SetMetadata('isPublic', true)`. In Guard, check with `reflector.get('isPublic', context.getHandler())`. If true, skip authentication and return true.

</details>

## Authorization Guards

### 22. How do you implement a Role-Based Authorization Guard?

<details>
<summary>Answer</summary>

Create Guard that checks if user has required roles from metadata.

**Roles Decorator:**
```typescript
import { SetMetadata } from '@nestjs/common';

export const Roles = (...roles: string[]) => SetMetadata('roles', roles);
```

**Roles Guard:**
```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // Get required roles from metadata
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    
    if (!roles) {
      return true;  // No roles required
    }
    
    // Get user from request
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    
    if (!user) {
      return false;  // No user attached
    }
    
    // Check if user has any required role
    return roles.includes(user.role);
  }
}
```

**Usage:**
```typescript
@Controller('admin')
@UseGuards(JwtAuthGuard, RolesGuard)  // Auth first, then roles
export class AdminController {
  @Get('users')
  @Roles('admin', 'superadmin')  // Only admin or superadmin
  getUsers() {
    return [];
  }
  
  @Delete('users/:id')
  @Roles('superadmin')  // Only superadmin
  deleteUser(@Param('id') id: string) {
    return {};
  }
}
```

**With Error Messages:**
```typescript
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    
    if (!roles) {
      return true;
    }
    
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    
    if (!user) {
      throw new ForbiddenException('User not authenticated');
    }
    
    if (!roles.includes(user.role)) {
      throw new ForbiddenException(
        `User role '${user.role}' is not authorized. Required: ${roles.join(', ')}`,
      );
    }
    
    return true;
  }
}
```

**Interview Tip**: Create `@Roles()` decorator, RolesGuard reads roles from metadata with Reflector, checks if `user.role` is in required roles. Use after AuthGuard to ensure user is attached to request.

</details>

### 23. How do you check if a user has required roles?

<details>
<summary>Answer</summary>

Compare `user.role` or `user.roles` with required roles from metadata.

**Single Role:**
```typescript
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    
    // Check if user has any of the required roles
    return roles.includes(user.role);  // user.role = 'admin'
  }
}
```

**Multiple Roles (Array):**
```typescript
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.get<string[]>(
      'roles',
      context.getHandler(),
    );
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    
    // User has array of roles
    const userRoles = user.roles;  // ['user', 'editor']
    
    // Check if user has ANY required role
    return requiredRoles.some(role => userRoles.includes(role));
  }
}
```

**Must Have ALL Roles:**
```typescript
@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.get<string[]>('roles', context.getHandler());
    const user = request.user;
    
    // User must have ALL required roles
    return requiredRoles.every(role => user.roles.includes(role));
  }
}
```

**Interview Tip**: Use `Array.includes()` to check single role, `Array.some()` for "any role", `Array.every()` for "all roles". Compare against `user.role` (single) or `user.roles` (array).

</details>

### 24. What is RBAC (Role-Based Access Control)?

<details>
<summary>Answer</summary>

RBAC restricts system access based on **user roles** (admin, user, editor, etc.).

**Concept:**
```
User → Has Role → Role Has Permissions → Access Resource
```

**Example Roles:**
- **Admin** - Full access to everything
- **Editor** - Can create/edit content
- **Viewer** - Read-only access
- **Guest** - Limited public access

**Implementation:**
```typescript
// User entity
class User {
  id: string;
  email: string;
  role: 'admin' | 'editor' | 'viewer' | 'guest';
}

// Roles decorator
export const Roles = (...roles: string[]) => SetMetadata('roles', roles);

// Roles guard
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.get<string[]>(
      'roles',
      context.getHandler(),
    );
    
    if (!requiredRoles) {
      return true;
    }
    
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    
    return requiredRoles.includes(user.role);
  }
}

// Usage
@Controller('posts')
@UseGuards(JwtAuthGuard, RolesGuard)
export class PostsController {
  @Get()
  @Roles('admin', 'editor', 'viewer')  // All authenticated users
  findAll() {}
  
  @Post()
  @Roles('admin', 'editor')  // Only admin and editor
  create(@Body() dto: CreatePostDto) {}
  
  @Delete(':id')
  @Roles('admin')  // Only admin
  remove(@Param('id') id: string) {}
}
```

**Interview Tip**: RBAC assigns roles to users, roles have permissions. Implement with `@Roles()` decorator and RolesGuard that checks `user.role`. Simple and effective for most applications.

</details>

### 25. What is the difference between Authentication and Authorization Guards?

<details>
<summary>Answer</summary>

**Authentication** verifies identity (who you are); **Authorization** checks permissions (what you can do).

**Comparison:**
| Feature | Authentication | Authorization |
|---------|---------------|---------------|
| Purpose | Verify identity | Check permissions |
| Question | "Who are you?" | "What can you do?" |
| Checks | Token validity, user exists | Roles, permissions |
| Example | JWT validation | Role check (admin/user) |
| Order | First | Second |

**Authentication Guard:**
```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = this.extractToken(request);
    
    // Verify identity
    const payload = await this.jwtService.verifyAsync(token);
    request.user = payload;
    
    return true;  // User is authenticated
  }
}
```

**Authorization Guard:**
```typescript
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    const request = context.switchToHttp().getRequest();
    const user = request.user;  // Already authenticated
    
    // Check permissions
    return roles.includes(user.role);  // User is authorized
  }
}
```

**Used Together:**
```typescript
@Controller('admin')
@UseGuards(AuthGuard, RolesGuard)  // Auth THEN authorization
export class AdminController {
  @Get()
  @Roles('admin')
  getData() {}
}

// Flow:
// 1. AuthGuard - Validates JWT, attaches user
// 2. RolesGuard - Checks if user.role === 'admin'
```

**Interview Tip**: Authentication = verify identity (JWT validation). Authorization = check permissions (role/permission check). Always authenticate first, then authorize. Use AuthGuard for authentication, RolesGuard for authorization.

</details>

## Error Handling

### 26. What exceptions should Guards throw?

<details>
<summary>Answer</summary>

Guards should throw `UnauthorizedException` or `ForbiddenException`.

**UnauthorizedException (401):**
```typescript
import { UnauthorizedException } from '@nestjs/common';

// When user is not authenticated
throw new UnauthorizedException('Invalid token');
throw new UnauthorizedException('Token expired');
throw new UnauthorizedException('User not found');
```

**ForbiddenException (403):**
```typescript
import { ForbiddenException } from '@nestjs/common';

// When user is authenticated but not authorized
throw new ForbiddenException('Insufficient permissions');
throw new ForbiddenException('Admin role required');
throw new ForbiddenException('You cannot access this resource');
```

**In Guards:**
```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = this.extractToken(request);
    
    if (!token) {
      // User not authenticated
      throw new UnauthorizedException('No token provided');
    }
    
    try {
      const payload = await this.jwtService.verifyAsync(token);
      request.user = payload;
      return true;
    } catch (error) {
      // Invalid token
      throw new UnauthorizedException('Invalid or expired token');
    }
  }
}

@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    
    if (!roles.includes(user.role)) {
      // User authenticated but not authorized
      throw new ForbiddenException(
        `Role '${user.role}' is not authorized. Required: ${roles.join(', ')}`,
      );
    }
    
    return true;
  }
}
```

**Interview Tip**: Throw `UnauthorizedException` (401) for authentication failures (invalid token, not logged in). Throw `ForbiddenException` (403) for authorization failures (wrong role, insufficient permissions).

</details>

### 27. What is `UnauthorizedException` vs `ForbiddenException`?

<details>
<summary>Answer</summary>

**UnauthorizedException (401)** = not authenticated; **ForbiddenException (403)** = authenticated but not authorized.

**Comparison:**
| Exception | Status Code | Meaning | When to Use |
|-----------|------------|---------|-------------|
| UnauthorizedException | 401 | Not authenticated | No/invalid token, user not logged in |
| ForbiddenException | 403 | Not authorized | Valid user but insufficient permissions |

**UnauthorizedException (401):**
```typescript
// Authentication failures
throw new UnauthorizedException('No token provided');
throw new UnauthorizedException('Invalid token');
throw new UnauthorizedException('Token expired');
throw new UnauthorizedException('User not found');
throw new UnauthorizedException('Invalid credentials');
```

**ForbiddenException (403):**
```typescript
// Authorization failures
throw new ForbiddenException('Insufficient permissions');
throw new ForbiddenException('Admin role required');
throw new ForbiddenException('You cannot delete this resource');
throw new ForbiddenException('Access denied to this endpoint');
```

**Real-World Example:**
```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  async canActivate(context: ExecutionContext): Promise<boolean> {
    const token = this.extractToken(request);
    
    // 401 - User not authenticated
    if (!token) {
      throw new UnauthorizedException('Please log in');
    }
    
    try {
      const user = await this.authService.validateToken(token);
      request.user = user;
      return true;
    } catch (error) {
      // 401 - Invalid token
      throw new UnauthorizedException('Invalid or expired token');
    }
  }
}

@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    const user = request.user;
    
    // User is authenticated (has valid token)
    // But doesn't have required role
    if (!roles.includes(user.role)) {
      // 403 - User authenticated but not authorized
      throw new ForbiddenException(
        `This action requires ${roles.join(' or ')} role`,
      );
    }
    
    return true;
  }
}
```

**Interview Tip**: 401 (UnauthorizedException) = "Who are you?" - authentication failed. 403 (ForbiddenException) = "What can you do?" - authorization failed. Use 401 for AuthGuard, 403 for RolesGuard.

</details>

### 28. When should you throw each exception?

<details>
<summary>Answer</summary>

**UnauthorizedException (401)** for authentication issues; **ForbiddenException (403)** for authorization issues.

**Throw UnauthorizedException (401) when:**
- No token provided
- Invalid token format
- Token expired
- Token signature invalid
- User not found
- Invalid credentials (login)

```typescript
// Authentication Guard
throw new UnauthorizedException('No token provided');
throw new UnauthorizedException('Invalid token');
throw new UnauthorizedException('Token expired');
throw new UnauthorizedException('User not found');
```

**Throw ForbiddenException (403) when:**
- User lacks required role
- User lacks required permission
- Resource access denied
- Action not allowed for user

```typescript
// Authorization Guard
throw new ForbiddenException('Admin role required');
throw new ForbiddenException('Insufficient permissions');
throw new ForbiddenException('You cannot delete this resource');
throw new ForbiddenException('Access denied');
```

**Decision Tree:**
```
Is user authenticated (valid token)?
├─ NO → UnauthorizedException (401)
└─ YES → Is user authorized (has permissions)?
    ├─ NO → ForbiddenException (403)
    └─ YES → Allow request
```

**Interview Tip**: 401 for "not authenticated" (AuthGuard), 403 for "authenticated but not authorized" (RolesGuard). Client should re-authenticate for 401, cannot retry for 403.

</details>

## Guards vs Other Features

### 29. What is the difference between Guards and Middleware?

<details>
<summary>Answer</summary>

**Middleware** runs before routing; **Guards** run after routing and have access to ExecutionContext.

**Comparison:**
| Feature | Middleware | Guards |
|---------|-----------|--------|
| Execution | Before routing | After routing |
| Access to | req, res, next | ExecutionContext |
| Purpose | Auth, logging, CORS | Authorization |
| Return | void (calls next) | boolean |
| Metadata | No | Yes (via Reflector) |

**Middleware:**
```typescript
// Runs before routing - doesn't know which handler
@Injectable()
export class AuthMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    // Authenticate user, attach to request
    const token = req.headers.authorization?.split(' ')[1];
    const user = await this.authService.validateToken(token);
    req['user'] = user;
    next();  // Always proceeds
  }
}
```

**Guard:**
```typescript
// Runs after routing - knows handler and metadata
@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    // Can access metadata
    const roles = this.reflector.get('roles', context.getHandler());
    const request = context.switchToHttp().getRequest();
    
    // Decide to allow or deny
    return roles.includes(request.user.role);  // true/false
  }
}
```

**When to Use:**
```typescript
// ✅ Middleware for:
// - Authentication (validate token, attach user)
// - Logging requests
// - CORS, helmet, compression
// - Request parsing

// ✅ Guards for:
// - Authorization (check roles/permissions)
// - Route protection based on metadata
// - Conditional access
```

**Interview Tip**: Middleware = before routing, no metadata access, authenticate. Guards = after routing, has metadata access, authorize. Use middleware to authenticate, guards to authorize.

</details>

### 30. What is the difference between Guards and Interceptors?

<details>
<summary>Answer</summary>

**Guards** decide if request proceeds; **Interceptors** transform request/response with RxJS.

**Comparison:**
| Feature | Guards | Interceptors |
|---------|--------|--------------|
| Purpose | Authorization | Transform req/res |
| Return | boolean | Observable |
| When | Before handler | Before & after handler |
| Use Case | Check permissions | Transform data, cache, timeout |

**Guard:**
```typescript
@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    // Decide if request can proceed
    const user = context.switchToHttp().getRequest().user;
    return user.role === 'admin';  // true = allow, false = deny
  }
}
```

**Interceptor:**
```typescript
@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler) {
    console.log('Before handler');
    
    // Transform response
    return next.handle().pipe(
      tap(() => console.log('After handler')),
      map(data => ({ data, timestamp: Date.now() })),
    );
  }
}
```

**When to Use:**
```typescript
// ✅ Guards for:
// - Check if user has permission
// - Authentication/authorization
// - Conditional route access

// ✅ Interceptors for:
// - Transform response format
// - Add metadata to response
// - Caching
// - Timeouts
// - Logging before/after handler
```

**Interview Tip**: Guards authorize access (return boolean). Interceptors transform request/response (return Observable). Use guards to protect routes, interceptors to transform data.

</details>

### 31. When should you use Guards vs Middleware for authentication?

<details>
<summary>Answer</summary>

Use **Middleware** to authenticate; use **Guards** to authorize.

**Best Practice:**
```typescript
// Middleware - Authenticate (validate token, attach user)
@Injectable()
export class AuthMiddleware implements NestMiddleware {
  async use(req: Request, res: Response, next: NextFunction) {
    const token = req.headers.authorization?.split(' ')[1];
    
    if (token) {
      const user = await this.authService.validateToken(token);
      req['user'] = user;  // Attach user
    }
    
    next();  // Always proceed
  }
}

// Guard - Authorize (check if user has permission)
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    return !!request.user;  // Check if user exists
  }
}

@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const roles = this.reflector.get('roles', context.getHandler());
    const user = context.switchToHttp().getRequest().user;
    return roles.includes(user.role);  // Check role
  }
}
```

**Why This Approach:**
```typescript
// Middleware can't access route metadata
// Can't know if route requires specific roles
consumer.apply(AuthMiddleware).forRoutes('*');

// Guards can access metadata
@Get()
@Roles('admin')  // Guard reads this metadata
@UseGuards(RolesGuard)
findAll() {}
```

**Alternative - Guards Only:**
```typescript
// Can also do authentication in Guard
@Injectable()
export class JwtAuthGuard implements CanActivate {
  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    
    // Check if public
    const isPublic = this.reflector.get('isPublic', context.getHandler());
    if (isPublic) return true;
    
    // Authenticate
    const token = this.extractToken(request);
    const user = await this.authService.validateToken(token);
    request['user'] = user;
    
    return true;
  }
}

// Apply globally
@Module({
  providers: [{ provide: APP_GUARD, useClass: JwtAuthGuard }],
})
export class AppModule {}
```

**Interview Tip**: Middleware for authentication (validate token, attach user), Guards for authorization (check roles/permissions). Guards have access to route metadata, middleware doesn't. Both approaches work - choose based on your needs.

</details>