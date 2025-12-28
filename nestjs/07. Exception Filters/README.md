# NestJS Exception Filters - Top Interview Questions

## Exception Filter Fundamentals

### 1. What is an Exception Filter in NestJS?

<details>
<summary>Answer</summary>

An **Exception Filter** is a class that handles exceptions thrown during request processing. It catches errors and formats error responses sent to clients.

**Key Features:**
1. Catch and handle exceptions
2. Format error responses
3. Log errors
4. Transform exception details
5. Control what clients see

**Basic Structure:**
```typescript
import { ExceptionFilter, Catch, ArgumentsHost, HttpException } from '@nestjs/common';

@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    const status = exception.getStatus();
    
    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      message: exception.message,
    });
  }
}
```

**Request Lifecycle Position:**
```
Request
  ↓
Middleware
  ↓
Guards
  ↓
Interceptors (before)
  ↓
Pipes
  ↓
Route Handler → throws exception
  ↓
EXCEPTION FILTERS ← Catch and handle errors
  ↓
Response (error)
```

**Simple Example:**
```typescript
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    
    const status = exception instanceof HttpException
      ? exception.getStatus()
      : 500;
    
    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      message: exception instanceof HttpException ? exception.message : 'Internal server error',
    });
  }
}
```

**Catch Specific Exception:**
```typescript
@Catch(NotFoundException)
export class NotFoundExceptionFilter implements ExceptionFilter {
  catch(exception: NotFoundException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    
    response.status(404).json({
      statusCode: 404,
      timestamp: new Date().toISOString(),
      path: request.url,
      message: `Resource not found: ${request.url}`,
    });
  }
}
```

**Usage:**
```typescript
// Apply to method
@Controller('users')
export class UsersController {
  @Get(':id')
  @UseFilters(HttpExceptionFilter)
  findOne(@Param('id') id: string) {
    throw new NotFoundException('User not found');
  }
}

// Apply to controller
@Controller('posts')
@UseFilters(HttpExceptionFilter)
export class PostsController {}

// Apply globally
// main.ts
app.useGlobalFilters(new AllExceptionsFilter());
```

**With Logging:**
```typescript
@Catch()
export class LoggingExceptionFilter implements ExceptionFilter {
  constructor(private readonly logger: Logger) {}
  
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    
    const status = exception instanceof HttpException
      ? exception.getStatus()
      : 500;
    
    // Log the error
    this.logger.error(
      `${request.method} ${request.url}`,
      exception instanceof Error ? exception.stack : 'Unknown error',
    );
    
    response.status(status).json({
      statusCode: status,
      message: exception instanceof HttpException ? exception.message : 'Internal server error',
    });
  }
}
```

**Interview Tip**: Exception Filters catch errors thrown during request processing. Run at the end of request lifecycle. Use `@Catch()` to specify exception types to handle. Implement `ExceptionFilter` interface with `catch()` method. Can format error responses, log errors, hide sensitive data. Apply with `@UseFilters()` or globally.

</details>

### 2. When are Exception Filters executed?

<details>
<summary>Answer</summary>

Exception Filters execute **after** an exception is thrown and **before** sending the response to the client. They run at the **end** of the request lifecycle.

**Request Lifecycle with Exception:**
```
1. Client Request
2. Middleware
3. Guards
4. Interceptors (before)
5. Pipes
6. Route Handler → throws exception
7. ❌ Handler execution stops
8. ❌ Interceptors (after) - SKIPPED
9. EXCEPTION FILTERS ← Execute here
10. Error Response to Client
```

**Normal vs Exception Flow:**
```
NORMAL FLOW:
Request → Middleware → Guards → Interceptors → Pipes → Handler 
  → Interceptors (after) → Response

EXCEPTION FLOW:
Request → Middleware → Guards → Interceptors → Pipes → Handler 
  → ❌ Exception thrown
  → Exception Filters → Error Response
```

**Example with Logging:**
```typescript
// Handler throws exception
@Controller('users')
export class UsersController {
  @Get(':id')
  @UseFilters(LoggingFilter)
  @UseInterceptors(LoggingInterceptor)
  findOne(@Param('id') id: string) {
    console.log('1. Handler executing');
    throw new NotFoundException('User not found');
    console.log('2. This never executes');
  }
}

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler) {
    console.log('3. Interceptor BEFORE');
    return next.handle().pipe(
      tap(() => console.log('4. Interceptor AFTER - SKIPPED')),
    );
  }
}

@Catch()
export class LoggingFilter implements ExceptionFilter {
  catch(exception: any, host: ArgumentsHost) {
    console.log('5. Exception Filter executing');
    // Handle exception
  }
}

// Console output:
// 3. Interceptor BEFORE
// 1. Handler executing
// 5. Exception Filter executing
// (Interceptor AFTER is skipped)
```

**Exception in Different Phases:**
```typescript
// Exception in Guard
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    console.log('1. Guard executing');
    throw new UnauthorizedException(); // Exception here
  }
}

// Exception Filter catches it
@Catch(UnauthorizedException)
export class UnauthorizedFilter implements ExceptionFilter {
  catch(exception: UnauthorizedException, host: ArgumentsHost) {
    console.log('2. Exception Filter catches UnauthorizedException');
    // Handler never executes
  }
}
```

**Multiple Exception Filters:**
```typescript
@Controller('posts')
@UseFilters(
  SpecificExceptionFilter,  // More specific - checked first
  GeneralExceptionFilter,    // More general - checked if not caught
)
export class PostsController {
  @Get(':id')
  findOne(@Param('id') id: string) {
    throw new NotFoundException();
  }
}

// Execution order:
// 1. Exception thrown
// 2. SpecificExceptionFilter checks if it handles this exception
// 3. If not handled, GeneralExceptionFilter checks
// 4. If still not handled, default NestJS error handler
```

**Timing Example:**
```typescript
@Catch()
export class TimingExceptionFilter implements ExceptionFilter {
  catch(exception: any, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const request = ctx.getRequest();
    
    // Exception filter runs AFTER exception
    console.log('Exception caught at:', new Date().toISOString());
    console.log('Request started at:', request.startTime);
    console.log('Time to exception:', Date.now() - request.startTime, 'ms');
  }
}
```

**Interview Tip**: Exception Filters run **after** exception is thrown, **before** response sent. Run at end of request lifecycle. Interceptors' "after" logic is **skipped** when exception occurs. More specific filters checked before general ones. Can catch exceptions from any phase (Guards, Pipes, Handlers, Interceptors).

</details>

### 3. What is the `ExceptionFilter` interface?

<details>
<summary>Answer</summary>

`ExceptionFilter` is the interface that all exception filters must implement. It has one method: `catch()`.

**Interface Definition:**
```typescript
export interface ExceptionFilter<T = any> {
  catch(exception: T, host: ArgumentsHost): any;
}
```

**Basic Implementation:**
```typescript
import { ExceptionFilter, Catch, ArgumentsHost, HttpException } from '@nestjs/common';

@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    const status = exception.getStatus();
    
    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
    });
  }
}
```

**Type Parameter:**
```typescript
// Generic - catches any exception
@Catch()
export class AllExceptionsFilter implements ExceptionFilter<any> {
  catch(exception: any, host: ArgumentsHost) {
    // Handle any exception
  }
}

// Specific type - catches only HttpException
@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter<HttpException> {
  catch(exception: HttpException, host: ArgumentsHost) {
    // exception is typed as HttpException
    const status = exception.getStatus();
    const message = exception.message;
  }
}

// Multiple specific types
@Catch(NotFoundException, BadRequestException)
export class ValidationExceptionFilter implements ExceptionFilter {
  catch(exception: NotFoundException | BadRequestException, host: ArgumentsHost) {
    // Handle specific exceptions
  }
}
```

**With Dependency Injection:**
```typescript
@Catch()
export class LoggingExceptionFilter implements ExceptionFilter {
  constructor(
    private readonly logger: Logger,
    private readonly configService: ConfigService,
  ) {}
  
  catch(exception: any, host: ArgumentsHost) {
    const isProduction = this.configService.get('NODE_ENV') === 'production';
    
    this.logger.error('Exception occurred:', exception);
    
    // Use injected services
  }
}
```

**Return Value:**
```typescript
@Catch()
export class CustomExceptionFilter implements ExceptionFilter {
  catch(exception: any, host: ArgumentsHost): void {
    // Method doesn't need to return anything
    // Just send response through host.getResponse()
    
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    
    response.status(500).json({ error: 'Something went wrong' });
  }
}
```

**Async Exception Filter:**
```typescript
@Catch()
export class AsyncExceptionFilter implements ExceptionFilter {
  constructor(private readonly notificationService: NotificationService) {}
  
  async catch(exception: any, host: ArgumentsHost): Promise<void> {
    // Can be async
    await this.notificationService.sendAlert({
      type: 'error',
      message: exception.message,
    });
    
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    response.status(500).json({ error: 'Error occurred' });
  }
}
```

**Multiple Exception Types:**
```typescript
@Catch(HttpException, Error)
export class MultiTypeFilter implements ExceptionFilter {
  catch(exception: HttpException | Error, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    
    if (exception instanceof HttpException) {
      response.status(exception.getStatus()).json({
        message: exception.message,
      });
    } else if (exception instanceof Error) {
      response.status(500).json({
        message: 'Internal server error',
      });
    }
  }
}
```

**Interview Tip**: All exception filters implement `ExceptionFilter<T>` interface with `catch(exception, host)` method. Use `@Catch()` decorator to specify exception types. Type parameter `T` defines exception type. Can inject dependencies via constructor. Method can be async. Return value is ignored - use `host.getResponse()` to send response.

</details>

### 4. What parameters does the `catch()` method receive?

<details>
<summary>Answer</summary>

The `catch()` method receives two parameters: `exception` (the thrown error) and `host` (ArgumentsHost for context).

**Method Signature:**
```typescript
catch(
  exception: T,              // The exception that was thrown
  host: ArgumentsHost,       // Provides access to request/response
): any
```

**1. Exception Parameter:**
```typescript
@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    // Access exception properties
    const status = exception.getStatus();
    const message = exception.message;
    const response = exception.getResponse();
    
    console.log('Status:', status);
    console.log('Message:', message);
    console.log('Response:', response);
  }
}
```

**2. ArgumentsHost Parameter:**
```typescript
@Catch()
export class CustomExceptionFilter implements ExceptionFilter {
  catch(exception: any, host: ArgumentsHost) {
    // Get HTTP context
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    
    // Get execution context info
    const type = host.getType(); // 'http', 'ws', 'rpc'
    
    console.log('Request URL:', request.url);
    console.log('Request Method:', request.method);
    console.log('Context Type:', type);
    
    response.status(500).json({ error: 'Error' });
  }
}
```

**Exception Details:**
```typescript
@Catch()
export class DetailedExceptionFilter implements ExceptionFilter {
  catch(exception: any, host: ArgumentsHost) {
    // Check exception type
    if (exception instanceof HttpException) {
      const status = exception.getStatus();
      const response = exception.getResponse();
      
      console.log('HttpException:', {
        status,
        message: exception.message,
        response: typeof response === 'string' ? response : response,
      });
    } else if (exception instanceof Error) {
      console.log('Standard Error:', {
        name: exception.name,
        message: exception.message,
        stack: exception.stack,
      });
    } else {
      console.log('Unknown exception:', exception);
    }
  }
}
```

**Request Context:**
```typescript
@Catch()
export class RequestContextFilter implements ExceptionFilter {
  catch(exception: any, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const request = ctx.getRequest();
    
    // Access request details
    console.log('URL:', request.url);
    console.log('Method:', request.method);
    console.log('Headers:', request.headers);
    console.log('Query:', request.query);
    console.log('Body:', request.body);
    console.log('Params:', request.params);
    console.log('User:', request.user);
    console.log('IP:', request.ip);
  }
}
```

**Response Manipulation:**
```typescript
@Catch()
export class ResponseFilter implements ExceptionFilter {
  catch(exception: any, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    
    // Set response status
    response.status(500);
    
    // Set response headers
    response.setHeader('X-Error-Message', exception.message);
    response.setHeader('X-Request-Id', request.id);
    
    // Send JSON response
    response.json({
      statusCode: 500,
      message: exception.message,
      timestamp: new Date().toISOString(),
    });
  }
}
```

**Different Context Types:**
```typescript
@Catch()
export class UniversalExceptionFilter implements ExceptionFilter {
  catch(exception: any, host: ArgumentsHost) {
    const type = host.getType();
    
    if (type === 'http') {
      const ctx = host.switchToHttp();
      const response = ctx.getResponse();
      response.status(500).json({ error: 'HTTP Error' });
    } else if (type === 'ws') {
      const client = host.switchToWs().getClient();
      client.emit('error', { message: exception.message });
    } else if (type === 'rpc') {
      const rpcContext = host.switchToRpc();
      // Handle RPC exception
    }
  }
}
```

**Custom Exception with Data:**
```typescript
class CustomException extends HttpException {
  constructor(
    public readonly code: string,
    message: string,
    status: number,
  ) {
    super(message, status);
  }
}

@Catch(CustomException)
export class CustomExceptionFilter implements ExceptionFilter {
  catch(exception: CustomException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    
    // Access custom properties
    const code = exception.code;
    const status = exception.getStatus();
    const message = exception.message;
    
    response.status(status).json({
      errorCode: code,
      message,
      statusCode: status,
    });
  }
}
```

**Interview Tip**: `catch()` receives `exception` (thrown error) and `host` (ArgumentsHost). Use `exception.getStatus()`, `exception.message` for error details. Use `host.switchToHttp()` to get request/response. Access request info (url, method, body, user). Set response status and body. `host.getType()` determines context type (http/ws/rpc).

</details>

## Built-in Exceptions

### 5. What are the most commonly used built-in HTTP exceptions?

<details>
<summary>Answer</summary>

NestJS provides built-in HTTP exception classes for common HTTP status codes.

**Common Built-in Exceptions:**

| Exception | Status Code | Use Case |
|-----------|-------------|----------|
| `BadRequestException` | 400 | Invalid request data |
| `UnauthorizedException` | 401 | Not authenticated |
| `ForbiddenException` | 403 | Not authorized |
| `NotFoundException` | 404 | Resource not found |
| `MethodNotAllowedException` | 405 | HTTP method not allowed |
| `NotAcceptableException` | 406 | Can't produce requested format |
| `RequestTimeoutException` | 408 | Request timeout |
| `ConflictException` | 409 | Resource conflict |
| `GoneException` | 410 | Resource no longer available |
| `PayloadTooLargeException` | 413 | Request body too large |
| `UnsupportedMediaTypeException` | 415 | Unsupported media type |
| `UnprocessableEntityException` | 422 | Validation failed |
| `InternalServerErrorException` | 500 | Server error |
| `NotImplementedException` | 501 | Not implemented |
| `BadGatewayException` | 502 | Bad gateway |
| `ServiceUnavailableException` | 503 | Service unavailable |
| `GatewayTimeoutException` | 504 | Gateway timeout |

**Basic Usage:**
```typescript
import {
  BadRequestException,
  UnauthorizedException,
  ForbiddenException,
  NotFoundException,
  ConflictException,
  InternalServerErrorException,
} from '@nestjs/common';

@Controller('users')
export class UsersController {
  @Get(':id')
  findOne(@Param('id') id: string) {
    const user = this.usersService.findOne(id);
    
    if (!user) {
      throw new NotFoundException('User not found');
    }
    
    return user;
  }
  
  @Post()
  create(@Body() dto: CreateUserDto) {
    if (!dto.email) {
      throw new BadRequestException('Email is required');
    }
    
    if (this.usersService.emailExists(dto.email)) {
      throw new ConflictException('Email already exists');
    }
    
    return this.usersService.create(dto);
  }
  
  @Delete(':id')
  remove(@Param('id') id: string, @Request() req) {
    if (!req.user) {
      throw new UnauthorizedException('Please log in');
    }
    
    if (req.user.role !== 'admin') {
      throw new ForbiddenException('Admin access required');
    }
    
    return this.usersService.remove(id);
  }
}
```

**With Custom Messages:**
```typescript
// Simple message
throw new NotFoundException('User not found');

// Custom response object
throw new BadRequestException({
  statusCode: 400,
  message: 'Validation failed',
  errors: ['Email is required', 'Password too short'],
});

// With error code
throw new ConflictException({
  message: 'Email already exists',
  errorCode: 'EMAIL_EXISTS',
});
```

**Response Formats:**
```typescript
// Default format
throw new NotFoundException('User not found');
// Response:
{
  "statusCode": 404,
  "message": "User not found"
}

// Custom object
throw new BadRequestException({
  message: 'Validation failed',
  errors: ['Email is invalid'],
});
// Response:
{
  "statusCode": 400,
  "message": "Validation failed",
  "errors": ["Email is invalid"]
}

// Array of messages
throw new BadRequestException(['Email is required', 'Password is required']);
// Response:
{
  "statusCode": 400,
  "message": ["Email is required", "Password is required"]
}
```

**Real-World Examples:**
```typescript
@Controller('posts')
export class PostsController {
  // 400 - Bad Request
  @Post()
  create(@Body() dto: CreatePostDto) {
    if (!dto.title || dto.title.length < 3) {
      throw new BadRequestException('Title must be at least 3 characters');
    }
    return this.postsService.create(dto);
  }
  
  // 401 - Unauthorized
  @Get('my-posts')
  getMyPosts(@Request() req) {
    if (!req.user) {
      throw new UnauthorizedException('Authentication required');
    }
    return this.postsService.findByUser(req.user.id);
  }
  
  // 403 - Forbidden
  @Delete(':id')
  remove(@Param('id') id: string, @Request() req) {
    const post = this.postsService.findOne(id);
    if (post.authorId !== req.user.id) {
      throw new ForbiddenException('You can only delete your own posts');
    }
    return this.postsService.remove(id);
  }
  
  // 404 - Not Found
  @Get(':id')
  findOne(@Param('id') id: string) {
    const post = this.postsService.findOne(id);
    if (!post) {
      throw new NotFoundException(`Post with ID ${id} not found`);
    }
    return post;
  }
  
  // 409 - Conflict
  @Post()
  create(@Body() dto: CreatePostDto) {
    if (this.postsService.slugExists(dto.slug)) {
      throw new ConflictException('Post with this slug already exists');
    }
    return this.postsService.create(dto);
  }
  
  // 422 - Unprocessable Entity
  @Post()
  create(@Body() dto: CreatePostDto) {
    const validationErrors = this.validate(dto);
    if (validationErrors.length > 0) {
      throw new UnprocessableEntityException({
        message: 'Validation failed',
        errors: validationErrors,
      });
    }
    return this.postsService.create(dto);
  }
  
  // 500 - Internal Server Error
  @Get(':id')
  async findOne(@Param('id') id: string) {
    try {
      return await this.postsService.findOne(id);
    } catch (error) {
      throw new InternalServerErrorException('Failed to fetch post');
    }
  }
  
  // 503 - Service Unavailable
  @Post()
  create(@Body() dto: CreatePostDto) {
    if (!this.databaseService.isConnected()) {
      throw new ServiceUnavailableException('Database is unavailable');
    }
    return this.postsService.create(dto);
  }
}
```

**Interview Tip**: Most common: `BadRequestException` (400), `UnauthorizedException` (401), `ForbiddenException` (403), `NotFoundException` (404), `ConflictException` (409), `InternalServerErrorException` (500). Pass string message or object with custom data. Default response includes `statusCode` and `message`. Use appropriate status code for each scenario.

</details>

### 6. What is `HttpException` and how is it used?

<details>
<summary>Answer</summary>

`HttpException` is the **base class** for all HTTP exceptions in NestJS. All built-in exceptions extend it.

**Basic Usage:**
```typescript
import { HttpException, HttpStatus } from '@nestjs/common';

throw new HttpException('Forbidden', HttpStatus.FORBIDDEN);

// Response:
{
  "statusCode": 403,
  "message": "Forbidden"
}
```

**Constructor Signature:**
```typescript
new HttpException(
  response: string | Record<string, any>,
  status: number,
  options?: HttpExceptionOptions
)
```

**With String Message:**
```typescript
throw new HttpException('User not found', HttpStatus.NOT_FOUND);

// Response:
{
  "statusCode": 404,
  "message": "User not found"
}
```

**With Custom Object:**
```typescript
throw new HttpException(
  {
    status: HttpStatus.BAD_REQUEST,
    error: 'Invalid input',
    details: 'Email format is incorrect',
  },
  HttpStatus.BAD_REQUEST,
);

// Response:
{
  "status": 400,
  "error": "Invalid input",
  "details": "Email format is incorrect"
}
```

**HttpStatus Constants:**
```typescript
import { HttpStatus } from '@nestjs/common';

// 4xx Client Errors
HttpStatus.BAD_REQUEST           // 400
HttpStatus.UNAUTHORIZED          // 401
HttpStatus.FORBIDDEN             // 403
HttpStatus.NOT_FOUND             // 404
HttpStatus.CONFLICT              // 409
HttpStatus.UNPROCESSABLE_ENTITY  // 422

// 5xx Server Errors
HttpStatus.INTERNAL_SERVER_ERROR // 500
HttpStatus.NOT_IMPLEMENTED       // 501
HttpStatus.BAD_GATEWAY           // 502
HttpStatus.SERVICE_UNAVAILABLE   // 503
```

**Custom Status Codes:**
```typescript
// Custom status code
throw new HttpException('Custom error', 418); // I'm a teapot

// Or use HttpStatus
throw new HttpException(
  'Payment Required',
  HttpStatus.PAYMENT_REQUIRED, // 402
);
```

**With Options:**
```typescript
throw new HttpException(
  'Forbidden',
  HttpStatus.FORBIDDEN,
  {
    cause: new Error('Access denied'),
    description: 'User lacks required permissions',
  },
);
```

**Extending HttpException:**
```typescript
// Custom exception class
export class EmailAlreadyExistsException extends HttpException {
  constructor(email: string) {
    super(
      {
        statusCode: HttpStatus.CONFLICT,
        message: 'Email already exists',
        email,
        errorCode: 'EMAIL_EXISTS',
      },
      HttpStatus.CONFLICT,
    );
  }
}

// Usage
throw new EmailAlreadyExistsException('john@example.com');

// Response:
{
  "statusCode": 409,
  "message": "Email already exists",
  "email": "john@example.com",
  "errorCode": "EMAIL_EXISTS"
}
```

**Methods Available:**
```typescript
const exception = new HttpException('Not Found', HttpStatus.NOT_FOUND);

// Get status code
const status = exception.getStatus(); // 404

// Get response
const response = exception.getResponse();
// { statusCode: 404, message: "Not Found" }

// Get message
const message = exception.message; // "Not Found"
```

**Built-in vs HttpException:**
```typescript
// These are equivalent:
throw new NotFoundException('User not found');
throw new HttpException('User not found', HttpStatus.NOT_FOUND);

throw new BadRequestException('Invalid data');
throw new HttpException('Invalid data', HttpStatus.BAD_REQUEST);

// But built-in exceptions are more semantic
```

**Real-World Example:**
```typescript
@Injectable()
export class UsersService {
  async findOne(id: string): Promise<User> {
    const user = await this.userRepository.findOne(id);
    
    if (!user) {
      throw new HttpException(
        {
          statusCode: HttpStatus.NOT_FOUND,
          message: 'User not found',
          userId: id,
          timestamp: new Date().toISOString(),
        },
        HttpStatus.NOT_FOUND,
      );
    }
    
    return user;
  }
  
  async create(dto: CreateUserDto): Promise<User> {
    const existingUser = await this.userRepository.findByEmail(dto.email);
    
    if (existingUser) {
      throw new HttpException(
        {
          statusCode: HttpStatus.CONFLICT,
          message: 'User with this email already exists',
          email: dto.email,
          errorCode: 'DUPLICATE_EMAIL',
        },
        HttpStatus.CONFLICT,
      );
    }
    
    return this.userRepository.create(dto);
  }
}
```

**Interview Tip**: `HttpException` is base class for all HTTP exceptions. Constructor takes `response` (string or object) and `status` (number). Use `HttpStatus` enum for status codes. Built-in exceptions (`NotFoundException`, `BadRequestException`) extend `HttpException`. Use `getStatus()` and `getResponse()` to access exception data. Prefer built-in exceptions for common cases.

</details>

### 7. When do you use `BadRequestException`?

<details>
<summary>Answer</summary>

Use `BadRequestException` (400) when client sends **invalid or malformed** request data.

**Common Use Cases:**
1. Missing required fields
2. Invalid data format
3. Invalid data values
4. Validation failures
5. Malformed request body

**Basic Example:**
```typescript
import { BadRequestException } from '@nestjs/common';

@Controller('users')
export class UsersController {
  @Post()
  create(@Body() dto: CreateUserDto) {
    if (!dto.email) {
      throw new BadRequestException('Email is required');
    }
    
    if (!dto.password) {
      throw new BadRequestException('Password is required');
    }
    
    return this.usersService.create(dto);
  }
}
```

**Validation Errors:**
```typescript
@Post()
create(@Body() dto: CreateUserDto) {
  // Email format validation
  if (!/^\S+@\S+\.\S+$/.test(dto.email)) {
    throw new BadRequestException('Invalid email format');
  }
  
  // Password length validation
  if (dto.password.length < 8) {
    throw new BadRequestException('Password must be at least 8 characters');
  }
  
  // Age validation
  if (dto.age && (dto.age < 18 || dto.age > 120)) {
    throw new BadRequestException('Age must be between 18 and 120');
  }
  
  return this.usersService.create(dto);
}
```

**Multiple Validation Errors:**
```typescript
@Post()
create(@Body() dto: CreateUserDto) {
  const errors: string[] = [];
  
  if (!dto.email) {
    errors.push('Email is required');
  } else if (!/^\S+@\S+\.\S+$/.test(dto.email)) {
    errors.push('Invalid email format');
  }
  
  if (!dto.password) {
    errors.push('Password is required');
  } else if (dto.password.length < 8) {
    errors.push('Password must be at least 8 characters');
  }
  
  if (errors.length > 0) {
    throw new BadRequestException({
      message: 'Validation failed',
      errors,
    });
  }
  
  return this.usersService.create(dto);
}

// Response:
{
  "statusCode": 400,
  "message": "Validation failed",
  "errors": [
    "Email is required",
    "Password must be at least 8 characters"
  ]
}
```

**Invalid Query Parameters:**
```typescript
@Get()
findAll(@Query() query: any) {
  // Invalid pagination
  if (query.page && (isNaN(query.page) || query.page < 1)) {
    throw new BadRequestException('Invalid page number');
  }
  
  if (query.limit && (isNaN(query.limit) || query.limit < 1 || query.limit > 100)) {
    throw new BadRequestException('Limit must be between 1 and 100');
  }
  
  // Invalid sort parameter
  const validSortFields = ['name', 'createdAt', 'updatedAt'];
  if (query.sortBy && !validSortFields.includes(query.sortBy)) {
    throw new BadRequestException(
      `Invalid sort field. Must be one of: ${validSortFields.join(', ')}`
    );
  }
  
  return this.usersService.findAll(query);
}
```

**Invalid Date Format:**
```typescript
@Get('reports')
getReports(@Query('startDate') startDate: string, @Query('endDate') endDate: string) {
  const start = new Date(startDate);
  const end = new Date(endDate);
  
  if (isNaN(start.getTime())) {
    throw new BadRequestException('Invalid start date format');
  }
  
  if (isNaN(end.getTime())) {
    throw new BadRequestException('Invalid end date format');
  }
  
  if (start > end) {
    throw new BadRequestException('Start date must be before end date');
  }
  
  return this.reportsService.generate(start, end);
}
```

**File Upload Validation:**
```typescript
@Post('upload')
@UseInterceptors(FileInterceptor('file'))
uploadFile(@UploadedFile() file: Express.Multer.File) {
  if (!file) {
    throw new BadRequestException('File is required');
  }
  
  const allowedMimeTypes = ['image/jpeg', 'image/png', 'image/gif'];
  if (!allowedMimeTypes.includes(file.mimetype)) {
    throw new BadRequestException('Only JPEG, PNG, and GIF images are allowed');
  }
  
  const maxSize = 5 * 1024 * 1024; // 5MB
  if (file.size > maxSize) {
    throw new BadRequestException('File size must not exceed 5MB');
  }
  
  return this.filesService.upload(file);
}
```

**Invalid ID Format:**
```typescript
@Get(':id')
findOne(@Param('id') id: string) {
  // UUID validation
  const uuidRegex = /^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i;
  if (!uuidRegex.test(id)) {
    throw new BadRequestException('Invalid UUID format');
  }
  
  return this.usersService.findOne(id);
}

@Get(':id')
findOne(@Param('id') id: string) {
  // MongoDB ObjectId validation
  if (!/^[0-9a-fA-F]{24}$/.test(id)) {
    throw new BadRequestException('Invalid ID format');
  }
  
  return this.usersService.findOne(id);
}
```

**With ValidationPipe:**
```typescript
// DTO
import { IsEmail, IsString, MinLength, IsInt, Min, Max } from 'class-validator';

export class CreateUserDto {
  @IsEmail()
  email: string;
  
  @IsString()
  @MinLength(8)
  password: string;
  
  @IsInt()
  @Min(18)
  @Max(120)
  age: number;
}

// Controller with ValidationPipe
@Post()
@UsePipes(new ValidationPipe())
create(@Body() dto: CreateUserDto) {
  // ValidationPipe automatically throws BadRequestException
  // if validation fails
  return this.usersService.create(dto);
}

// Automatic response:
{
  "statusCode": 400,
  "message": [
    "email must be an email",
    "password must be longer than or equal to 8 characters"
  ],
  "error": "Bad Request"
}
```

**Interview Tip**: Use `BadRequestException` (400) for **client errors**: missing fields, invalid format, validation failures, invalid query params. Indicates **client** needs to fix request. Can pass string message or object with multiple errors. Often thrown automatically by ValidationPipe. Don't use for authorization (401) or not found (404).

</details>

### 8. When do you use `UnauthorizedException` vs `ForbiddenException`?

<details>
<summary>Answer</summary>

**UnauthorizedException (401)**: User is **not authenticated** (not logged in).
**ForbiddenException (403)**: User **is authenticated** but **lacks permission**.

**Key Difference:**
```
401 Unauthorized: "Who are you?" - Identity not proven
403 Forbidden: "I know who you are, but you can't do this" - Insufficient permissions
```

**Comparison Table:**

| Aspect | UnauthorizedException (401) | ForbiddenException (403) |
|--------|----------------------------|--------------------------|
| Meaning | Not authenticated | Not authorized |
| User State | Not logged in | Logged in |
| Fix | Log in / Provide credentials | Request higher permissions |
| Example | No JWT token | User role is 'user', needs 'admin' |

**UnauthorizedException (401) Examples:**
```typescript
import { UnauthorizedException } from '@nestjs/common';

// Missing authentication token
@Get('profile')
getProfile(@Headers('authorization') auth: string) {
  if (!auth) {
    throw new UnauthorizedException('Authentication token is required');
  }
  // ...
}

// Invalid token
@Get('profile')
getProfile(@Headers('authorization') auth: string) {
  const token = auth?.replace('Bearer ', '');
  
  try {
    const payload = this.jwtService.verify(token);
  } catch (error) {
    throw new UnauthorizedException('Invalid or expired token');
  }
  // ...
}

// Expired session
@Get('data')
getData(@Request() req) {
  if (!req.session || !req.session.user) {
    throw new UnauthorizedException('Session expired. Please log in again');
  }
  // ...
}

// Wrong credentials
@Post('login')
async login(@Body() dto: LoginDto) {
  const user = await this.authService.validateUser(dto.email, dto.password);
  
  if (!user) {
    throw new UnauthorizedException('Invalid email or password');
  }
  
  return this.authService.login(user);
}
```

**ForbiddenException (403) Examples:**
```typescript
import { ForbiddenException } from '@nestjs/common';

// Insufficient role
@Delete(':id')
remove(@Param('id') id: string, @Request() req) {
  // User IS authenticated but not an admin
  if (req.user.role !== 'admin') {
    throw new ForbiddenException('Admin access required');
  }
  
  return this.usersService.remove(id);
}

// Not resource owner
@Put('posts/:id')
update(@Param('id') id: string, @Request() req, @Body() dto: UpdatePostDto) {
  const post = this.postsService.findOne(id);
  
  // User IS authenticated but doesn't own this post
  if (post.authorId !== req.user.id) {
    throw new ForbiddenException('You can only edit your own posts');
  }
  
  return this.postsService.update(id, dto);
}

// Missing permissions
@Post('projects/:id/deploy')
deploy(@Param('id') id: string, @Request() req) {
  const project = this.projectsService.findOne(id);
  
  // User IS authenticated but lacks deploy permission
  if (!req.user.permissions.includes('deploy')) {
    throw new ForbiddenException('You do not have permission to deploy');
  }
  
  return this.projectsService.deploy(id);
}

// Account status
@Get('premium-content')
getPremiumContent(@Request() req) {
  // User IS authenticated but account not premium
  if (req.user.accountType !== 'premium') {
    throw new ForbiddenException('Premium subscription required');
  }
  
  return this.contentService.getPremiumContent();
}
```

**Real-World Flow:**
```typescript
@Controller('admin')
export class AdminController {
  @Get('users')
  getAllUsers(@Request() req) {
    // Step 1: Check authentication (401)
    if (!req.user) {
      throw new UnauthorizedException('Please log in to access this resource');
    }
    
    // Step 2: Check authorization (403)
    if (req.user.role !== 'admin') {
      throw new ForbiddenException('Admin access required');
    }
    
    return this.usersService.findAll();
  }
}
```

**Guard Implementation:**
```typescript
// Authentication Guard (401)
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    
    if (!request.user) {
      throw new UnauthorizedException('Authentication required');
    }
    
    return true;
  }
}

// Authorization Guard (403)
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}
  
  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.get<string[]>('roles', context.getHandler());
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    
    // User IS authenticated (checked by AuthGuard)
    // But may not have required role
    if (!requiredRoles.some(role => user.roles?.includes(role))) {
      throw new ForbiddenException('Insufficient permissions');
    }
    
    return true;
  }
}

// Usage
@Controller('posts')
@UseGuards(AuthGuard) // Must be logged in
export class PostsController {
  @Get()
  findAll() {
    return []; // Any authenticated user can view
  }
  
  @Delete(':id')
  @UseGuards(RolesGuard)
  @Roles('admin')
  remove(@Param('id') id: string) {
    // Must be authenticated AND be admin
    return this.postsService.remove(id);
  }
}
```

**Decision Flow:**
```typescript
function checkAccess(request) {
  // First: Check authentication (WHO)
  if (!request.user || !request.token) {
    throw new UnauthorizedException('Please log in');
    // "I don't know who you are"
  }
  
  // Second: Check authorization (CAN DO WHAT)
  if (!hasPermission(request.user, requiredPermission)) {
    throw new ForbiddenException('Insufficient permissions');
    // "I know who you are, but you can't do this"
  }
  
  // All good
}
```

**Response Examples:**
```typescript
// 401 Response
throw new UnauthorizedException('Please log in');
{
  "statusCode": 401,
  "message": "Please log in",
  "error": "Unauthorized"
}

// 403 Response
throw new ForbiddenException('Admin access required');
{
  "statusCode": 403,
  "message": "Admin access required",
  "error": "Forbidden"
}
```

**Interview Tip**: **401 Unauthorized** = not logged in, missing/invalid credentials. **403 Forbidden** = logged in but lacks permission. Think: 401 asks "Who are you?", 403 says "I know you, but no access". Use 401 for authentication failures, 403 for authorization failures. Guards typically throw 401 for auth, 403 for roles.

</details>

### 9. When do you use `NotFoundException`?

<details>
<summary>Answer</summary>

Use `NotFoundException` (404) when a **requested resource doesn't exist**.

**Common Use Cases:**
1. Entity/record not found by ID
2. Route/endpoint doesn't exist
3. File doesn't exist
4. User not found
5. Resource deleted

**Basic Example:**
```typescript
import { NotFoundException } from '@nestjs/common';

@Controller('users')
export class UsersController {
  @Get(':id')
  findOne(@Param('id') id: string) {
    const user = this.usersService.findOne(id);
    
    if (!user) {
      throw new NotFoundException('User not found');
    }
    
    return user;
  }
}

// Response:
{
  "statusCode": 404,
  "message": "User not found",
  "error": "Not Found"
}
```

**With Resource ID:**
```typescript
@Get(':id')
findOne(@Param('id') id: string) {
  const user = this.usersService.findOne(id);
  
  if (!user) {
    throw new NotFoundException(`User with ID ${id} not found`);
  }
  
  return user;
}

// Response:
{
  "statusCode": 404,
  "message": "User with ID 123 not found",
  "error": "Not Found"
}
```

**Service Layer:**
```typescript
@Injectable()
export class UsersService {
  async findOne(id: string): Promise<User> {
    const user = await this.userRepository.findOne({ where: { id } });
    
    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    
    return user;
  }
  
  async update(id: string, dto: UpdateUserDto): Promise<User> {
    const user = await this.findOne(id); // Throws NotFoundException if not found
    
    Object.assign(user, dto);
    return this.userRepository.save(user);
  }
  
  async remove(id: string): Promise<void> {
    const result = await this.userRepository.delete(id);
    
    if (result.affected === 0) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
  }
}
```

**Multiple Related Resources:**
```typescript
@Get('posts/:postId/comments/:commentId')
findComment(
  @Param('postId') postId: string,
  @Param('commentId') commentId: string,
) {
  const post = this.postsService.findOne(postId);
  if (!post) {
    throw new NotFoundException(`Post with ID ${postId} not found`);
  }
  
  const comment = this.commentsService.findOne(commentId);
  if (!comment) {
    throw new NotFoundException(`Comment with ID ${commentId} not found`);
  }
  
  if (comment.postId !== postId) {
    throw new NotFoundException('Comment does not belong to this post');
  }
  
  return comment;
}
```

**File Not Found:**
```typescript
@Get('files/:filename')
async downloadFile(@Param('filename') filename: string, @Res() res: Response) {
  const filePath = path.join('./uploads', filename);
  
  if (!fs.existsSync(filePath)) {
    throw new NotFoundException(`File ${filename} not found`);
  }
  
  return res.sendFile(filePath);
}
```

**Query with Filters:**
```typescript
@Get('search')
search(@Query('email') email: string) {
  const user = this.usersService.findByEmail(email);
  
  if (!user) {
    throw new NotFoundException(`User with email ${email} not found`);
  }
  
  return user;
}
```

**Soft Deleted Resources:**
```typescript
@Get(':id')
async findOne(@Param('id') id: string) {
  const user = await this.userRepository.findOne({
    where: { id },
    withDeleted: false, // Exclude soft-deleted
  });
  
  if (!user) {
    throw new NotFoundException(`User with ID ${id} not found or has been deleted`);
  }
  
  return user;
}
```

**Custom Error Object:**
```typescript
@Get(':id')
findOne(@Param('id') id: string) {
  const user = this.usersService.findOne(id);
  
  if (!user) {
    throw new NotFoundException({
      statusCode: 404,
      message: 'User not found',
      userId: id,
      timestamp: new Date().toISOString(),
      errorCode: 'USER_NOT_FOUND',
    });
  }
  
  return user;
}

// Response:
{
  "statusCode": 404,
  "message": "User not found",
  "userId": "123",
  "timestamp": "2024-01-01T00:00:00.000Z",
  "errorCode": "USER_NOT_FOUND"
}
```

**Nested Resources:**
```typescript
@Injectable()
export class PostsService {
  async findOne(id: string): Promise<Post> {
    const post = await this.postRepository.findOne({
      where: { id },
      relations: ['author', 'comments'],
    });
    
    if (!post) {
      throw new NotFoundException(`Post with ID ${id} not found`);
    }
    
    return post;
  }
}
```

**404 vs 403:**
```typescript
@Get(':id')
findOne(@Param('id') id: string, @Request() req) {
  const post = this.postsService.findOne(id);
  
  // Use 404 if resource doesn't exist
  if (!post) {
    throw new NotFoundException('Post not found');
  }
  
  // Use 403 if resource exists but user can't access it
  if (post.isPrivate && post.authorId !== req.user?.id) {
    throw new ForbiddenException('Access denied to private post');
  }
  
  return post;
}

// Note: Some APIs prefer to return 404 for inaccessible resources
// to avoid leaking information about resource existence
@Get(':id')
findOne(@Param('id') id: string, @Request() req) {
  const post = this.postsService.findOne(id);
  
  // Return 404 for both cases for security
  if (!post || (post.isPrivate && post.authorId !== req.user?.id)) {
    throw new NotFoundException('Post not found');
  }
  
  return post;
}
```

**Interview Tip**: Use `NotFoundException` (404) when **resource doesn't exist**. Common in GET/PUT/DELETE operations with ID. Throw after database query returns null/undefined. Include resource identifier in message. Use in services for reusability. Consider security: returning 404 for unauthorized access prevents information leakage about resource existence.

</details>

### 10. What is `InternalServerErrorException`?

<details>
<summary>Answer</summary>

`InternalServerErrorException` (500) indicates an **unexpected server error** that the client cannot fix.

**When to Use:**
- Unhandled exceptions
- Database connection failures
- External service failures
- Unexpected errors in business logic
- System errors

**Basic Example:**
```typescript
import { InternalServerErrorException } from '@nestjs/common';

@Get(':id')
async findOne(@Param('id') id: string) {
  try {
    return await this.usersService.findOne(id);
  } catch (error) {
    throw new InternalServerErrorException('Failed to fetch user');
  }
}

// Response:
{
  "statusCode": 500,
  "message": "Failed to fetch user",
  "error": "Internal Server Error"
}
```

**Database Errors:**
```typescript
@Injectable()
export class UsersService {
  async create(dto: CreateUserDto): Promise<User> {
    try {
      return await this.userRepository.save(dto);
    } catch (error) {
      // Database error
      throw new InternalServerErrorException(
        'Failed to create user due to database error'
      );
    }
  }
  
  async findAll(): Promise<User[]> {
    try {
      return await this.userRepository.find();
    } catch (error) {
      if (error.code === 'ECONNREFUSED') {
        throw new InternalServerErrorException('Database connection failed');
      }
      throw new InternalServerErrorException('Failed to fetch users');
    }
  }
}
```

**External Service Failures:**
```typescript
@Injectable()
export class PaymentService {
  async processPayment(dto: PaymentDto) {
    try {
      const response = await this.httpService.post(
        'https://payment-gateway.com/charge',
        dto
      ).toPromise();
      
      return response.data;
    } catch (error) {
      if (error.response?.status === 503) {
        throw new InternalServerErrorException('Payment gateway is unavailable');
      }
      
      throw new InternalServerErrorException('Payment processing failed');
    }
  }
}
```

**File System Errors:**
```typescript
@Post('upload')
async uploadFile(@UploadedFile() file: Express.Multer.File) {
  try {
    const filePath = path.join('./uploads', file.filename);
    await fs.promises.writeFile(filePath, file.buffer);
    return { filename: file.filename };
  } catch (error) {
    throw new InternalServerErrorException('Failed to save file');
  }
}
```

**With Error Details (Development):**
```typescript
@Injectable()
export class UsersService {
  constructor(private configService: ConfigService) {}
  
  async findOne(id: string): Promise<User> {
    try {
      return await this.userRepository.findOne({ where: { id } });
    } catch (error) {
      const isDevelopment = this.configService.get('NODE_ENV') === 'development';
      
      if (isDevelopment) {
        // Include error details in development
        throw new InternalServerErrorException({
          message: 'Failed to fetch user',
          error: error.message,
          stack: error.stack,
        });
      } else {
        // Hide error details in production
        throw new InternalServerErrorException('Failed to fetch user');
      }
    }
  }
}
```

**Unexpected State:**
```typescript
@Post('process')
processData(@Body() dto: DataDto) {
  const result = this.dataService.process(dto);
  
  // Unexpected null/undefined
  if (result === null || result === undefined) {
    throw new InternalServerErrorException('Data processing returned unexpected result');
  }
  
  // Unexpected type
  if (typeof result !== 'object') {
    throw new InternalServerErrorException('Data processing returned invalid type');
  }
  
  return result;
}
```

**Global Exception Handler:**
```typescript
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    
    // Check if it's an HTTP exception
    if (exception instanceof HttpException) {
      const status = exception.getStatus();
      response.status(status).json(exception.getResponse());
    } else {
      // Unknown error - return 500
      console.error('Unexpected error:', exception);
      
      response.status(500).json({
        statusCode: 500,
        message: 'Internal server error',
        error: 'Internal Server Error',
      });
    }
  }
}
```

**When NOT to Use:**
```typescript
// ❌ DON'T use for client errors
@Get(':id')
findOne(@Param('id') id: string) {
  if (!id) {
    // Use BadRequestException instead
    throw new BadRequestException('ID is required');
  }
}

// ❌ DON'T use for not found
@Get(':id')
findOne(@Param('id') id: string) {
  const user = this.usersService.findOne(id);
  if (!user) {
    // Use NotFoundException instead
    throw new NotFoundException('User not found');
  }
}

// ❌ DON'T use for unauthorized
@Get('profile')
getProfile(@Request() req) {
  if (!req.user) {
    // Use UnauthorizedException instead
    throw new UnauthorizedException();
  }
}
```

**Logging with 500 Errors:**
```typescript
@Injectable()
export class UsersService {
  constructor(private logger: Logger) {}
  
  async create(dto: CreateUserDto): Promise<User> {
    try {
      return await this.userRepository.save(dto);
    } catch (error) {
      // Log server errors for investigation
      this.logger.error(
        'Failed to create user',
        error.stack,
        'UsersService',
      );
      
      throw new InternalServerErrorException('Failed to create user');
    }
  }
}
```

**Interview Tip**: Use `InternalServerErrorException` (500) for **server errors** client can't fix: database failures, external service errors, unexpected exceptions. Hide error details in production, show in development. Always log 500 errors for debugging. Don't use for client errors (4xx). Indicates server is at fault, not client.

</details>

## Creating Exception Filters

11. How do you create a custom Exception Filter?
12. What is `ArgumentsHost` and how do you use it?
13. How do you access the request and response objects?
14. How do you send custom error responses?

<details>
<summary>Answer for Q11</summary>

Create a class implementing `ExceptionFilter` interface with `@Catch()` decorator.

**Basic Custom Filter:**
```typescript
import { ExceptionFilter, Catch, ArgumentsHost, HttpException } from '@nestjs/common';
import { Response, Request } from 'express';

@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const status = exception.getStatus();
    
    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      message: exception.message,
    });
  }
}
```

**Catch All Exceptions:**
```typescript
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    
    const status = exception instanceof HttpException
      ? exception.getStatus()
      : 500;
    
    const message = exception instanceof HttpException
      ? exception.message
      : 'Internal server error';
    
    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      method: request.method,
      message,
    });
  }
}
```

**With Logging:**
```typescript
@Catch()
export class LoggingExceptionFilter implements ExceptionFilter {
  private readonly logger = new Logger(LoggingExceptionFilter.name);
  
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    
    const status = exception instanceof HttpException
      ? exception.getStatus()
      : 500;
    
    // Log the error
    this.logger.error(
      `${request.method} ${request.url}`,
      exception instanceof Error ? exception.stack : String(exception),
    );
    
    response.status(status).json({
      statusCode: status,
      message: exception instanceof HttpException ? exception.message : 'Internal server error',
    });
  }
}
```

**With Dependency Injection:**
```typescript
@Injectable()
@Catch()
export class DatabaseExceptionFilter implements ExceptionFilter {
  constructor(
    private readonly logger: Logger,
    private readonly configService: ConfigService,
  ) {}
  
  catch(exception: any, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    
    const isDevelopment = this.configService.get('NODE_ENV') === 'development';
    
    this.logger.error(`Database error: ${exception.message}`, exception.stack);
    
    response.status(500).json({
      statusCode: 500,
      message: 'Database error occurred',
      ...(isDevelopment && { details: exception.message }),
      timestamp: new Date().toISOString(),
      path: request.url,
    });
  }
}
```

**Catch Specific Exceptions:**
```typescript
// Catch only NotFoundException
@Catch(NotFoundException)
export class NotFoundExceptionFilter implements ExceptionFilter {
  catch(exception: NotFoundException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    
    response.status(404).json({
      statusCode: 404,
      message: exception.message,
      path: request.url,
      timestamp: new Date().toISOString(),
      suggestion: 'Please check the URL and try again',
    });
  }
}

// Catch multiple specific exceptions
@Catch(BadRequestException, UnprocessableEntityException)
export class ValidationExceptionFilter implements ExceptionFilter {
  catch(exception: BadRequestException | UnprocessableEntityException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const status = exception.getStatus();
    const exceptionResponse = exception.getResponse();
    
    response.status(status).json({
      statusCode: status,
      errors: typeof exceptionResponse === 'object' ? exceptionResponse : { message: exceptionResponse },
      timestamp: new Date().toISOString(),
    });
  }
}
```

**Production-Ready Filter:**
```typescript
@Injectable()
@Catch()
export class ProductionExceptionFilter implements ExceptionFilter {
  constructor(
    private readonly logger: Logger,
    private readonly configService: ConfigService,
  ) {}
  
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    
    const status = exception instanceof HttpException
      ? exception.getStatus()
      : 500;
    
    const isDevelopment = this.configService.get('NODE_ENV') === 'development';
    
    // Build error response
    const errorResponse: any = {
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      method: request.method,
    };
    
    // Handle different exception types
    if (exception instanceof HttpException) {
      const exceptionResponse = exception.getResponse();
      errorResponse.message = typeof exceptionResponse === 'string'
        ? exceptionResponse
        : (exceptionResponse as any).message || exception.message;
      
      // Include additional data if present
      if (typeof exceptionResponse === 'object' && exceptionResponse !== null) {
        Object.assign(errorResponse, exceptionResponse);
      }
    } else if (exception instanceof Error) {
      errorResponse.message = isDevelopment ? exception.message : 'Internal server error';
      
      if (isDevelopment) {
        errorResponse.error = exception.name;
        errorResponse.stack = exception.stack;
      }
    } else {
      errorResponse.message = 'Internal server error';
    }
    
    // Log errors
    if (status >= 500) {
      this.logger.error(
        `${request.method} ${request.url}`,
        exception instanceof Error ? exception.stack : String(exception),
      );
    } else {
      this.logger.warn(
        `${request.method} ${request.url} - ${errorResponse.message}`,
      );
    }
    
    response.status(status).json(errorResponse);
  }
}
```

</details>

<details>
<summary>Answer for Q12</summary>

`ArgumentsHost` provides access to the **execution context** (request, response, etc.) in exception filters.

**Basic Usage:**
```typescript
catch(exception: HttpException, host: ArgumentsHost) {
  // Switch to HTTP context
  const ctx = host.switchToHttp();
  
  // Get request and response
  const response = ctx.getResponse();
  const request = ctx.getRequest();
  
  // Use them
  response.status(404).json({
    path: request.url,
    message: exception.message,
  });
}
```

**ArgumentsHost Methods:**
```typescript
// Get execution context type
const type = host.getType(); // 'http' | 'ws' | 'rpc'

// Get arguments array
const args = host.getArgs(); // [req, res, next]

// Get specific argument by index
const request = host.getArgByIndex(0);
const response = host.getArgByIndex(1);
const next = host.getArgByIndex(2);

// Switch to specific context
const httpContext = host.switchToHttp();
const wsContext = host.switchToWs();
const rpcContext = host.switchToRpc();
```

**HTTP Context:**
```typescript
@Catch()
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: any, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    
    // Get request object
    const request = ctx.getRequest<Request>();
    console.log('URL:', request.url);
    console.log('Method:', request.method);
    console.log('Headers:', request.headers);
    console.log('Body:', request.body);
    console.log('Query:', request.query);
    console.log('Params:', request.params);
    
    // Get response object
    const response = ctx.getResponse<Response>();
    response.status(500).json({ error: 'Error' });
    
    // Get next function (if needed)
    const next = ctx.getNext();
  }
}
```

**WebSocket Context:**
```typescript
@Catch()
export class WsExceptionFilter implements ExceptionFilter {
  catch(exception: any, host: ArgumentsHost) {
    if (host.getType() === 'ws') {
      const ctx = host.switchToWs();
      
      // Get WebSocket client
      const client = ctx.getClient();
      
      // Get data
      const data = ctx.getData();
      
      // Send error to client
      client.emit('error', {
        message: exception.message,
        timestamp: new Date().toISOString(),
      });
    }
  }
}
```

**RPC Context:**
```typescript
@Catch()
export class RpcExceptionFilter implements ExceptionFilter {
  catch(exception: any, host: ArgumentsHost) {
    if (host.getType() === 'rpc') {
      const ctx = host.switchToRpc();
      
      // Get RPC context
      const data = ctx.getData();
      const context = ctx.getContext();
      
      // Handle RPC exception
      console.log('RPC data:', data);
      console.log('RPC context:', context);
    }
  }
}
```

**Universal Filter (All Contexts):**
```typescript
@Catch()
export class UniversalExceptionFilter implements ExceptionFilter {
  catch(exception: any, host: ArgumentsHost) {
    const type = host.getType();
    
    switch (type) {
      case 'http':
        this.handleHttp(exception, host);
        break;
      case 'ws':
        this.handleWs(exception, host);
        break;
      case 'rpc':
        this.handleRpc(exception, host);
        break;
      default:
        console.error('Unknown context type:', type);
    }
  }
  
  private handleHttp(exception: any, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    
    response.status(500).json({
      message: exception.message,
      path: request.url,
    });
  }
  
  private handleWs(exception: any, host: ArgumentsHost) {
    const ctx = host.switchToWs();
    const client = ctx.getClient();
    
    client.emit('error', { message: exception.message });
  }
  
  private handleRpc(exception: any, host: ArgumentsHost) {
    const ctx = host.switchToRpc();
    // Handle RPC exception
  }
}
```

**Accessing Request Details:**
```typescript
@Catch()
export class DetailedExceptionFilter implements ExceptionFilter {
  catch(exception: any, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const request = ctx.getRequest();
    const response = ctx.getResponse();
    
    // Access request properties
    const details = {
      url: request.url,
      method: request.method,
      headers: {
        userAgent: request.headers['user-agent'],
        contentType: request.headers['content-type'],
        authorization: request.headers['authorization'] ? 'Present' : 'Missing',
      },
      query: request.query,
      params: request.params,
      body: request.body,
      ip: request.ip,
      user: request.user, // If authenticated
    };
    
    console.log('Request details:', details);
    
    response.status(500).json({
      message: exception.message,
      timestamp: new Date().toISOString(),
      path: request.url,
    });
  }
}
```

**Type Safety:**
```typescript
import { Request, Response } from 'express';

@Catch()
export class TypedExceptionFilter implements ExceptionFilter {
  catch(exception: any, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    
    // Type the request and response
    const request = ctx.getRequest<Request>();
    const response = ctx.getResponse<Response>();
    
    // Now you get TypeScript autocomplete
    const url: string = request.url;
    const method: string = request.method;
    
    response.status(500).json({ error: 'Error' });
  }
}
```

</details>

<details>
<summary>Answer for Q13 & Q14</summary>

Access request/response via `host.switchToHttp()` and customize responses by building custom JSON objects.

**Access Request/Response:**
```typescript
@Catch()
export class MyExceptionFilter implements ExceptionFilter {
  catch(exception: any, host: ArgumentsHost) {
    // 1. Switch to HTTP context
    const ctx = host.switchToHttp();
    
    // 2. Get response object
    const response = ctx.getResponse();
    
    // 3. Get request object
    const request = ctx.getRequest();
    
    // 4. Use them
    const status = exception instanceof HttpException ? exception.getStatus() : 500;
    
    // 5. Send custom response
    response.status(status).json({
      success: false,
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      method: request.method,
      message: exception.message,
    });
  }
}
```

**Request Properties Available:**
```typescript
const request = ctx.getRequest();

// URL and method
request.url           // '/users/123'
request.method        // 'GET'
request.baseUrl       // Base URL
request.originalUrl   // Original URL with query
request.protocol      // 'http' or 'https'
request.hostname      // 'example.com'

// Parameters
request.query         // Query string params
request.params        // Route params
request.body          // Request body

// Headers
request.headers       // All headers
request.headers['user-agent']
request.headers['content-type']

// Authentication & Session
request.user          // User object (if authenticated)
request.session       // Session data

// Network
request.ip            // Client IP
request.ips           // IPs if behind proxy
```

**Response Methods Available:**
```typescript
const response = ctx.getResponse();

// Set status code
response.status(404);

// Set headers
response.setHeader('X-Error-Code', 'NOT_FOUND');
response.setHeader('X-Request-Id', '12345');

// Send JSON response
response.json({
  error: 'Not found',
  message: exception.message,
});

// Send plain text
response.send('Error occurred');

// Send status with default message
response.sendStatus(404);
```

**Custom Error Format Examples:**
```typescript
// Standard API Format
response.status(status).json({
  success: false,
  statusCode: status,
  timestamp: new Date().toISOString(),
  path: request.url,
  message: exception.message,
});

// Detailed Format with Error Code
response.status(status).json({
  error: {
    code: 'USER_NOT_FOUND',
    statusCode: status,
    message: exception.message,
  },
  meta: {
    timestamp: new Date().toISOString(),
    path: request.url,
    method: request.method,
  },
});

// Validation Error Format
response.status(400).json({
  statusCode: 400,
  error: 'Validation Error',
  message: 'Request validation failed',
  errors: [
    'email must be an email',
    'password must be longer than 8 characters'
  ],
  timestamp: new Date().toISOString(),
  path: request.url,
});

// Production vs Development Format
const isDevelopment = process.env.NODE_ENV === 'development';

if (isDevelopment) {
  // Detailed for dev
  response.status(status).json({
    statusCode: status,
    message: exception.message,
    error: exception.name,
    stack: exception.stack,
    timestamp: new Date().toISOString(),
    path: request.url,
    method: request.method,
    body: request.body,
    query: request.query,
  });
} else {
  // Minimal for production
  response.status(status).json({
    statusCode: status,
    message: status >= 500 ? 'Internal server error' : exception.message,
    timestamp: new Date().toISOString(),
  });
}
```

</details>

## Applying Exception Filters

15. How do you apply an Exception Filter using `@UseFilters()`?
16. How do you apply an Exception Filter globally?
17. What is `APP_FILTER` token?

<details>
<summary>Answer for Q15</summary>

Apply exception filters with `@UseFilters()` decorator at **method**, **controller**, or **global** level.

**Method Level:**
```typescript
@Controller('users')
export class UsersController {
  @Get(':id')
  @UseFilters(HttpExceptionFilter)
  findOne(@Param('id') id: string) {
    throw new NotFoundException('User not found');
    // HttpExceptionFilter will handle this
  }
  
  @Post()
  @UseFilters(HttpExceptionFilter)
  create(@Body() dto: CreateUserDto) {
    // Filter applies only to this method
  }
}
```

**Controller Level:**
```typescript
@Controller('users')
@UseFilters(HttpExceptionFilter)
export class UsersController {
  // Filter applies to all methods in this controller
  
  @Get(':id')
  findOne(@Param('id') id: string) {
    throw new NotFoundException(); // Handled by HttpExceptionFilter
  }
  
  @Post()
  create(@Body() dto: CreateUserDto) {
    throw new BadRequestException(); // Also handled
  }
}
```

**Multiple Filters:**
```typescript
@Controller('posts')
@UseFilters(ValidationFilter, NotFoundFilter, GeneralFilter)
export class PostsController {
  // Filters are checked in order: ValidationFilter → NotFoundFilter → GeneralFilter
  
  @Get(':id')
  findOne(@Param('id') id: string) {
    throw new NotFoundException(); // Caught by NotFoundFilter
  }
}
```

**Filter Instance vs Class:**
```typescript
// Using class (recommended - NestJS manages lifecycle)
@UseFilters(HttpExceptionFilter)
export class UsersController {}

// Using instance (you manage lifecycle)
@UseFilters(new HttpExceptionFilter())
export class PostsController {}

// With dependencies - use class and @Injectable()
@Injectable()
@Catch()
export class LoggingFilter implements ExceptionFilter {
  constructor(private logger: Logger) {}
  // ...
}

@UseFilters(LoggingFilter) // NestJS will inject dependencies
export class ProductsController {}
```

**Override Controller Filter:**
```typescript
@Controller('users')
@UseFilters(ControllerFilter)
export class UsersController {
  @Get()
  @UseFilters(MethodFilter) // This overrides ControllerFilter for this method
  findAll() {}
  
  @Get(':id')
  // Uses ControllerFilter
  findOne(@Param('id') id: string) {}
}
```

**Real-World Example:**
```typescript
// filters/http-exception.filter.ts
@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    const status = exception.getStatus();
    
    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      message: exception.message,
    });
  }
}

// users.controller.ts
@Controller('users')
@UseFilters(HttpExceptionFilter)
export class UsersController {
  constructor(private usersService: UsersService) {}
  
  @Get()
  findAll() {
    return this.usersService.findAll();
  }
  
  @Get(':id')
  findOne(@Param('id') id: string) {
    const user = this.usersService.findOne(id);
    if (!user) {
      throw new NotFoundException(`User ${id} not found`);
    }
    return user;
  }
  
  @Post()
  create(@Body() dto: CreateUserDto) {
    if (!dto.email) {
      throw new BadRequestException('Email is required');
    }
    return this.usersService.create(dto);
  }
}
```

</details>

<details>
<summary>Answer for Q16 & Q17</summary>

Apply exception filters globally in `main.ts` with `app.useGlobalFilters()` or via `APP_FILTER` provider.

**Method 1: main.ts (Simple, No DI):**
```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { AllExceptionsFilter } from './filters/all-exceptions.filter';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Apply global filter
  app.useGlobalFilters(new AllExceptionsFilter());
  
  await app.listen(3000);
}
bootstrap();
```

**Method 2: APP_FILTER Provider (With DI):**
```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { APP_FILTER } from '@nestjs/core';
import { AllExceptionsFilter } from './filters/all-exceptions.filter';

@Module({
  providers: [
    {
      provide: APP_FILTER,
      useClass: AllExceptionsFilter,
    },
  ],
})
export class AppModule {}

// filters/all-exceptions.filter.ts
@Injectable()
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  constructor(
    private readonly logger: Logger, // DI works!
    private readonly configService: ConfigService,
  ) {}
  
  catch(exception: unknown, host: ArgumentsHost) {
    // Implementation
  }
}
```

**What is APP_FILTER?**
`APP_FILTER` is a provider token for registering **global exception filters** with **dependency injection** support.

**Why Use APP_FILTER:**
```typescript
// ❌ Without APP_FILTER (no DI)
// main.ts
app.useGlobalFilters(new GlobalExceptionFilter());
// Can't inject dependencies in constructor

// ✅ With APP_FILTER (with DI)
// app.module.ts
@Module({
  providers: [
    {
      provide: APP_FILTER,
      useClass: GlobalExceptionFilter,
    },
  ],
})
export class AppModule {}

// Filter can now use DI
@Injectable()
@Catch()
export class GlobalExceptionFilter implements ExceptionFilter {
  constructor(
    private logger: Logger,           // ✅ Injected
    private configService: ConfigService, // ✅ Injected
  ) {}
}
```

**Multiple Global Filters:**
```typescript
// main.ts
app.useGlobalFilters(
  new ValidationFilter(),
  new NotFoundFilter(),
  new AllExceptionsFilter(),
);

// Or with APP_FILTER
@Module({
  providers: [
    {
      provide: APP_FILTER,
      useClass: ValidationFilter,
    },
    {
      provide: APP_FILTER,
      useClass: NotFoundFilter,
    },
    {
      provide: APP_FILTER,
      useClass: AllExceptionsFilter,
    },
  ],
})
export class AppModule {}
```

**Production-Ready Global Filter:**
```typescript
// filters/global-exception.filter.ts
@Injectable()
@Catch()
export class GlobalExceptionFilter implements ExceptionFilter {
  constructor(
    private readonly logger: Logger,
    private readonly configService: ConfigService,
  ) {}
  
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    
    const status = exception instanceof HttpException
      ? exception.getStatus()
      : 500;
    
    const isDevelopment = this.configService.get('NODE_ENV') === 'development';
    
    // Log error
    this.logger.error(
      `${request.method} ${request.url}`,
      exception instanceof Error ? exception.stack : String(exception),
    );
    
    // Send response
    const errorResponse: any = {
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      message: exception instanceof HttpException
        ? exception.message
        : 'Internal server error',
    };
    
    // Include stack trace in development
    if (isDevelopment && exception instanceof Error) {
      errorResponse.stack = exception.stack;
    }
    
    response.status(status).json(errorResponse);
  }
}

// app.module.ts
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

**APP_FILTER with useFactory:**
```typescript
@Module({
  providers: [
    {
      provide: APP_FILTER,
      useFactory: (configService: ConfigService, logger: Logger) => {
        const isDevelopment = configService.get('NODE_ENV') === 'development';
        
        if (isDevelopment) {
          return new DevelopmentExceptionFilter(logger);
        } else {
          return new ProductionExceptionFilter(logger, configService);
        }
      },
      inject: [ConfigService, Logger],
    },
  ],
})
export class AppModule {}
```

**Comparison:**

| Method | Dependency Injection | Use Case |
|--------|---------------------|----------|
| `app.useGlobalFilters()` | ❌ No | Simple, no dependencies |
| `APP_FILTER` | ✅ Yes | Needs DI (logger, config) |

</details>

## Custom Exceptions

18. How do you create a custom exception by extending `HttpException`?
19. How do you pass custom data in exceptions?
20. How do you set custom status codes and messages?

<details>
<summary>Answer for Q18, Q19, Q20</summary>

Create custom exceptions by extending `HttpException` and passing custom data in the constructor.

**Basic Custom Exception:**
```typescript
import { HttpException, HttpStatus } from '@nestjs/common';

export class UserNotFoundException extends HttpException {
  constructor(userId: string) {
    super(`User with ID ${userId} not found`, HttpStatus.NOT_FOUND);
  }
}

// Usage
throw new UserNotFoundException('123');
// Response: { "statusCode": 404, "message": "User with ID 123 not found" }
```

**Custom Exception with Custom Data:**
```typescript
export class EmailAlreadyExistsException extends HttpException {
  constructor(email: string) {
    super(
      {
        statusCode: HttpStatus.CONFLICT,
        message: 'Email already exists',
        email,
        errorCode: 'EMAIL_EXISTS',
        timestamp: new Date().toISOString(),
      },
      HttpStatus.CONFLICT,
    );
  }
}

// Usage
throw new EmailAlreadyExistsException('john@example.com');

// Response:
{
  "statusCode": 409,
  "message": "Email already exists",
  "email": "john@example.com",
  "errorCode": "EMAIL_EXISTS",
  "timestamp": "2024-01-01T00:00:00.000Z"
}
```

**Custom Status Codes and Messages:**
```typescript
// Custom status code
export class PaymentRequiredException extends HttpException {
  constructor(resource: string) {
    super(
      {
        statusCode: 402, // Payment Required
        message: `Payment required to access ${resource}`,
        errorCode: 'PAYMENT_REQUIRED',
      },
      402,
    );
  }
}

// Multiple messages
export class ValidationException extends HttpException {
  constructor(errors: string[]) {
    super(
      {
        statusCode: HttpStatus.BAD_REQUEST,
        message: 'Validation failed',
        errors,
      },
      HttpStatus.BAD_REQUEST,
    );
  }
}

// Usage
throw new ValidationException([
  'Email is required',
  'Password must be at least 8 characters',
]);

// Response:
{
  "statusCode": 400,
  "message": "Validation failed",
  "errors": [
    "Email is required",
    "Password must be at least 8 characters"
  ]
}
```

**Business Logic Exceptions:**
```typescript
export class InsufficientBalanceException extends HttpException {
  constructor(balance: number, required: number) {
    super(
      {
        statusCode: HttpStatus.BAD_REQUEST,
        message: 'Insufficient balance',
        errorCode: 'INSUFFICIENT_BALANCE',
        balance,
        required,
        shortfall: required - balance,
      },
      HttpStatus.BAD_REQUEST,
    );
  }
}

// Usage
throw new InsufficientBalanceException(50, 100);

// Response:
{
  "statusCode": 400,
  "message": "Insufficient balance",
  "errorCode": "INSUFFICIENT_BALANCE",
  "balance": 50,
  "required": 100,
  "shortfall": 50
}
```

**With Additional Context:**
```typescript
export class ResourceAccessDeniedException extends HttpException {
  constructor(resourceType: string, resourceId: string, userId: string, reason: string) {
    super(
      {
        statusCode: HttpStatus.FORBIDDEN,
        message: `Access denied to ${resourceType}`,
        errorCode: 'ACCESS_DENIED',
        resource: {
          type: resourceType,
          id: resourceId,
        },
        userId,
        reason,
        timestamp: new Date().toISOString(),
      },
      HttpStatus.FORBIDDEN,
    );
  }
}

// Usage
throw new ResourceAccessDeniedException('Post', '123', 'user-456', 'User is not the owner');

// Response:
{
  "statusCode": 403,
  "message": "Access denied to Post",
  "errorCode": "ACCESS_DENIED",
  "resource": {
    "type": "Post",
    "id": "123"
  },
  "userId": "user-456",
  "reason": "User is not the owner",
  "timestamp": "2024-01-01T00:00:00.000Z"
}
```

**Custom Properties:**
```typescript
export class RateLimitExceededException extends HttpException {
  public readonly retryAfter: number;
  
  constructor(limit: number, retryAfter: number) {
    super(
      {
        statusCode: 429,
        message: 'Rate limit exceeded',
        errorCode: 'RATE_LIMIT_EXCEEDED',
        limit,
        retryAfter,
      },
      429,
    );
    
    this.retryAfter = retryAfter;
  }
}

// Custom filter can access retryAfter property
@Catch(RateLimitExceededException)
export class RateLimitFilter implements ExceptionFilter {
  catch(exception: RateLimitExceededException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    
    // Set Retry-After header
    response.setHeader('Retry-After', exception.retryAfter);
    
    response.status(429).json(exception.getResponse());
  }
}
```

**Reusable Base Exception:**
```typescript
export class AppException extends HttpException {
  constructor(
    message: string,
    status: HttpStatus,
    errorCode: string,
    additionalData?: Record<string, any>,
  ) {
    super(
      {
        statusCode: status,
        message,
        errorCode,
        timestamp: new Date().toISOString(),
        ...additionalData,
      },
      status,
    );
  }
}

// Extend base exception
export class ProductNotFound extends AppException {
  constructor(productId: string) {
    super(
      `Product with ID ${productId} not found`,
      HttpStatus.NOT_FOUND,
      'PRODUCT_NOT_FOUND',
      { productId },
    );
  }
}

export class OrderAlreadyShipped extends AppException {
  constructor(orderId: string, shippedAt: Date) {
    super(
      'Order has already been shipped',
      HttpStatus.BAD_REQUEST,
      'ORDER_ALREADY_SHIPPED',
      { orderId, shippedAt },
    );
  }
}
```

**Real-World Examples:**
```typescript
// Authentication exception
export class InvalidCredentialsException extends HttpException {
  constructor() {
    super(
      {
        statusCode: HttpStatus.UNAUTHORIZED,
        message: 'Invalid email or password',
        errorCode: 'INVALID_CREDENTIALS',
      },
      HttpStatus.UNAUTHORIZED,
    );
  }
}

// Database exception
export class DatabaseConnectionException extends HttpException {
  constructor(databaseName: string, error: string) {
    super(
      {
        statusCode: HttpStatus.SERVICE_UNAVAILABLE,
        message: 'Database connection failed',
        errorCode: 'DATABASE_CONNECTION_FAILED',
        database: databaseName,
        details: error,
      },
      HttpStatus.SERVICE_UNAVAILABLE,
    );
  }
}

// External API exception
export class ExternalApiException extends HttpException {
  constructor(service: string, statusCode: number, error: string) {
    super(
      {
        statusCode: HttpStatus.BAD_GATEWAY,
        message: `External service ${service} returned error`,
        errorCode: 'EXTERNAL_API_ERROR',
        service,
        externalStatusCode: statusCode,
        externalError: error,
      },
      HttpStatus.BAD_GATEWAY,
    );
  }
}

// File exception
export class FileUploadException extends HttpException {
  constructor(filename: string, reason: string) {
    super(
      {
        statusCode: HttpStatus.BAD_REQUEST,
        message: 'File upload failed',
        errorCode: 'FILE_UPLOAD_FAILED',
        filename,
        reason,
      },
      HttpStatus.BAD_REQUEST,
    );
  }
}
```

**Using Custom Exceptions in Services:**
```typescript
@Injectable()
export class UsersService {
  async findOne(id: string): Promise<User> {
    const user = await this.userRepository.findOne({ where: { id } });
    
    if (!user) {
      throw new UserNotFoundException(id);
    }
    
    return user;
  }
  
  async create(dto: CreateUserDto): Promise<User> {
    const existingUser = await this.userRepository.findOne({
      where: { email: dto.email },
    });
    
    if (existingUser) {
      throw new EmailAlreadyExistsException(dto.email);
    }
    
    try {
      return await this.userRepository.save(dto);
    } catch (error) {
      throw new DatabaseConnectionException('users', error.message);
    }
  }
  
  async updateBalance(userId: string, amount: number): Promise<User> {
    const user = await this.findOne(userId);
    
    if (user.balance < amount) {
      throw new InsufficientBalanceException(user.balance, amount);
    }
    
    user.balance -= amount;
    return this.userRepository.save(user);
  }
}
```

</details>

## Error Response Format

21. What is the default error response format in NestJS?
22. How do you customize the error response structure?
23. Should you include stack traces in production?

<details>
<summary>Answer for Q21, Q22, Q23</summary>

**Default Error Response Format:**

NestJS's default error response includes:
```json
{
  "statusCode": 404,
  "message": "Not found",
  "error": "Not Found"
}
```

For validation errors (ValidationPipe):
```json
{
  "statusCode": 400,
  "message": [
    "email must be an email",
    "password must be longer than 8 characters"
  ],
  "error": "Bad Request"
}
```

**Customize Error Response Structure:**

Create a custom exception filter to change the response format:

```typescript
@Catch()
export class CustomErrorFormatFilter implements ExceptionFilter {
  catch(exception: any, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    
    const status = exception instanceof HttpException
      ? exception.getStatus()
      : 500;
    
    // Custom format
    const customResponse = {
      success: false,
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      message: exception.message,
    };
    
    response.status(status).json(customResponse);
  }
}

// Response:
{
  "success": false,
  "statusCode": 404,
  "timestamp": "2024-01-01T00:00:00.000Z",
  "path": "/users/123",
  "message": "User not found"
}
```

**API Standard Formats:**

```typescript
// REST API Standard
@Catch()
export class RestApiFilter implements ExceptionFilter {
  catch(exception: any, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    
    const status = exception instanceof HttpException ? exception.getStatus() : 500;
    
    response.status(status).json({
      error: {
        code: status,
        message: exception.message,
        type: exception.name || 'Error',
      },
      meta: {
        timestamp: new Date().toISOString(),
        path: request.url,
        method: request.method,
      },
    });
  }
}

// JSON:API Format
@Catch()
export class JsonApiFilter implements ExceptionFilter {
  catch(exception: any, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    
    const status = exception instanceof HttpException ? exception.getStatus() : 500;
    
    response.status(status).json({
      errors: [
        {
          status: status.toString(),
          title: exception.name || 'Error',
          detail: exception.message,
          source: {
            pointer: request.url,
          },
          meta: {
            timestamp: new Date().toISOString(),
          },
        },
      ],
    });
  }
}
```

**Stack Traces in Production:**

**❌ DON'T include stack traces in production** - they expose internal implementation details and security vulnerabilities.

**✅ DO include stack traces in development** - helps with debugging.

```typescript
@Injectable()
@Catch()
export class EnvironmentAwareFilter implements ExceptionFilter {
  constructor(private configService: ConfigService) {}
  
  catch(exception: any, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    
    const status = exception instanceof HttpException ? exception.getStatus() : 500;
    const isDevelopment = this.configService.get('NODE_ENV') === 'development';
    
    const errorResponse: any = {
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      message: exception.message,
    };
    
    // Only include stack trace in development
    if (isDevelopment && exception instanceof Error) {
      errorResponse.stack = exception.stack;
      errorResponse.error = exception.name;
    }
    
    // In production, hide sensitive error messages for 5xx errors
    if (!isDevelopment && status >= 500) {
      errorResponse.message = 'Internal server error';
    }
    
    response.status(status).json(errorResponse);
  }
}

// Development response:
{
  "statusCode": 500,
  "timestamp": "2024-01-01T00:00:00.000Z",
  "path": "/users",
  "message": "Cannot read property 'id' of undefined",
  "error": "TypeError",
  "stack": "TypeError: Cannot read property 'id' of undefined\n    at UsersService.findOne..."
}

// Production response:
{
  "statusCode": 500,
  "timestamp": "2024-01-01T00:00:00.000Z",
  "path": "/users",
  "message": "Internal server error"
}
```

**Logging Errors (Production Best Practice):**

```typescript
@Injectable()
@Catch()
export class ProductionFilter implements ExceptionFilter {
  constructor(
    private logger: Logger,
    private configService: ConfigService,
  ) {}
  
  catch(exception: any, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    
    const status = exception instanceof HttpException ? exception.getStatus() : 500;
    const isProduction = this.configService.get('NODE_ENV') === 'production';
    
    // Log full error details (including stack trace)
    this.logger.error(
      `${request.method} ${request.url} - ${status}`,
      {
        message: exception.message,
        stack: exception.stack,
        user: request.user?.id,
        body: request.body,
        query: request.query,
      },
    );
    
    // Send minimal response to client in production
    const errorResponse: any = {
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
    };
    
    if (isProduction && status >= 500) {
      // Hide internal error details
      errorResponse.message = 'Internal server error';
    } else {
      errorResponse.message = exception.message;
    }
    
    response.status(status).json(errorResponse);
  }
}
```

**Security Considerations:**

```typescript
// ❌ BAD - Exposes internals
{
  "statusCode": 500,
  "message": "Connection refused: ECONNREFUSED localhost:5432",
  "stack": "Error: Connection refused...\n    at Database.connect...",
  "query": "SELECT * FROM users WHERE password = '...'"
}

// ✅ GOOD - Generic message, log details server-side
{
  "statusCode": 500,
  "message": "Internal server error",
  "timestamp": "2024-01-01T00:00:00.000Z"
}
// Full details logged server-side for debugging
```

**Summary:**

| Environment | Include Stack Trace | Include Error Details | Log Errors |
|-------------|-------------------|----------------------|-----------|
| Development | ✅ Yes | ✅ Yes (all details) | ✅ Yes |
| Production | ❌ No | ⚠️ Limited (4xx: yes, 5xx: no) | ✅ Yes (full details) |

</details>

## Global Error Handling

24. How do you implement a global Exception Filter to catch all exceptions?
25. How do you handle validation errors from ValidationPipe?
26. How do you handle unknown/unhandled exceptions?

<details>
<summary>Answer for Q24, Q25, Q26</summary>

**Global Exception Filter (Catch All):**

```typescript
import { ExceptionFilter, Catch, ArgumentsHost, HttpException, HttpStatus } from '@nestjs/common';
import { Injectable, Logger } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';

@Injectable()
@Catch() // No arguments = catches ALL exceptions
export class GlobalExceptionFilter implements ExceptionFilter {
  constructor(
    private readonly logger: Logger,
    private readonly configService: ConfigService,
  ) {}
  
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    
    // Determine status code
    const status = exception instanceof HttpException
      ? exception.getStatus()
      : HttpStatus.INTERNAL_SERVER_ERROR;
    
    const isDevelopment = this.configService.get('NODE_ENV') === 'development';
    
    // Build error response
    let message = 'Internal server error';
    let errorResponse: any;
    
    if (exception instanceof HttpException) {
      // Handle HTTP exceptions
      errorResponse = exception.getResponse();
      message = typeof errorResponse === 'string'
        ? errorResponse
        : errorResponse.message || message;
    } else if (exception instanceof Error) {
      // Handle standard errors
      message = isDevelopment ? exception.message : 'Internal server error';
    }
    
    // Log the exception
    this.logException(exception, request, status);
    
    // Build response object
    const responseBody: any = {
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      method: request.method,
      message,
    };
    
    // Include additional data for HTTP exceptions
    if (exception instanceof HttpException && typeof errorResponse === 'object') {
      Object.assign(responseBody, errorResponse);
    }
    
    // Include stack trace in development
    if (isDevelopment && exception instanceof Error) {
      responseBody.stack = exception.stack;
      responseBody.error = exception.name;
    }
    
    response.status(status).json(responseBody);
  }
  
  private logException(exception: unknown, request: any, status: number) {
    const message = `${request.method} ${request.url} - ${status}`;
    
    if (status >= 500) {
      // Log server errors
      this.logger.error(
        message,
        exception instanceof Error ? exception.stack : String(exception),
      );
    } else {
      // Log client errors
      this.logger.warn(message, { exception: String(exception) });
    }
  }
}

// Register globally with APP_FILTER
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

**Handle Validation Errors from ValidationPipe:**

```typescript
// DTO with validation
import { IsEmail, IsString, MinLength, IsInt, Min } from 'class-validator';

export class CreateUserDto {
  @IsEmail()
  email: string;
  
  @IsString()
  @MinLength(8)
  password: string;
  
  @IsInt()
  @Min(18)
  age: number;
}

// ValidationPipe throws BadRequestException with validation errors
@Controller('users')
export class UsersController {
  @Post()
  @UsePipes(new ValidationPipe())
  create(@Body() dto: CreateUserDto) {
    return this.usersService.create(dto);
  }
}

// Custom filter to handle validation errors
@Catch(BadRequestException)
export class ValidationExceptionFilter implements ExceptionFilter {
  catch(exception: BadRequestException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    
    const exceptionResponse: any = exception.getResponse();
    
    // Format validation errors
    response.status(400).json({
      statusCode: 400,
      error: 'Validation Failed',
      message: 'Request validation failed',
      timestamp: new Date().toISOString(),
      path: request.url,
      // Extract validation errors
      errors: Array.isArray(exceptionResponse.message)
        ? exceptionResponse.message
        : [exceptionResponse.message],
    });
  }
}

// Register with global filter
@Module({
  providers: [
    {
      provide: APP_FILTER,
      useClass: ValidationExceptionFilter,
    },
    {
      provide: APP_FILTER,
      useClass: GlobalExceptionFilter, // Fallback for other exceptions
    },
  ],
})
export class AppModule {}

// Response:
{
  "statusCode": 400,
  "error": "Validation Failed",
  "message": "Request validation failed",
  "timestamp": "2024-01-01T00:00:00.000Z",
  "path": "/users",
  "errors": [
    "email must be an email",
    "password must be longer than or equal to 8 characters",
    "age must not be less than 18"
  ]
}
```

**Handle Unknown/Unhandled Exceptions:**

```typescript
@Injectable()
@Catch() // Catches everything
export class UnhandledExceptionFilter implements ExceptionFilter {
  constructor(
    private readonly logger: Logger,
    private readonly configService: ConfigService,
  ) {}
  
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    
    let status = HttpStatus.INTERNAL_SERVER_ERROR;
    let message = 'Internal server error';
    
    // Check if it's an HttpException
    if (exception instanceof HttpException) {
      status = exception.getStatus();
      const exceptionResponse = exception.getResponse();
      message = typeof exceptionResponse === 'string'
        ? exceptionResponse
        : (exceptionResponse as any).message || message;
    }
    // Check if it's a standard Error
    else if (exception instanceof Error) {
      this.logger.error('Unhandled error', exception.stack);
      message = this.configService.get('NODE_ENV') === 'development'
        ? exception.message
        : 'Internal server error';
    }
    // Unknown exception type
    else {
      this.logger.error('Unknown exception', String(exception));
      message = 'An unexpected error occurred';
    }
    
    response.status(status).json({
      statusCode: status,
      message,
      timestamp: new Date().toISOString(),
      path: request.url,
    });
  }
}
```

**Comprehensive Global Error Handler:**

```typescript
@Injectable()
@Catch()
export class ComprehensiveExceptionFilter implements ExceptionFilter {
  constructor(
    private readonly logger: Logger,
    private readonly configService: ConfigService,
  ) {}
  
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    
    const isDevelopment = this.configService.get('NODE_ENV') === 'development';
    
    // Initialize response
    let status = HttpStatus.INTERNAL_SERVER_ERROR;
    let message = 'Internal server error';
    let errorCode = 'INTERNAL_ERROR';
    let additionalData: any = {};
    
    // Handle different exception types
    if (exception instanceof HttpException) {
      // HTTP exceptions from NestJS
      status = exception.getStatus();
      const exceptionResponse = exception.getResponse();
      
      if (typeof exceptionResponse === 'string') {
        message = exceptionResponse;
      } else if (typeof exceptionResponse === 'object') {
        message = (exceptionResponse as any).message || message;
        additionalData = exceptionResponse;
      }
      
      errorCode = this.getErrorCode(status);
      
      this.logger.warn(`${request.method} ${request.url} - ${status}`, {
        exception: exception.message,
      });
    } else if (exception instanceof Error) {
      // Standard JavaScript errors
      message = isDevelopment ? exception.message : 'Internal server error';
      errorCode = exception.name || 'ERROR';
      
      this.logger.error(
        `${request.method} ${request.url} - Unhandled error`,
        exception.stack,
      );
      
      if (isDevelopment) {
        additionalData.stack = exception.stack;
        additionalData.error = exception.name;
      }
    } else {
      // Unknown exceptions
      this.logger.error(
        `${request.method} ${request.url} - Unknown exception`,
        String(exception),
      );
      message = 'An unexpected error occurred';
      errorCode = 'UNKNOWN_ERROR';
    }
    
    // Build response
    const errorResponse = {
      success: false,
      statusCode: status,
      errorCode,
      message,
      timestamp: new Date().toISOString(),
      path: request.url,
      method: request.method,
      ...additionalData,
    };
    
    // Hide sensitive data in production
    if (!isDevelopment && status >= 500) {
      delete errorResponse.stack;
      delete errorResponse.error;
      errorResponse.message = 'Internal server error';
    }
    
    response.status(status).json(errorResponse);
  }
  
  private getErrorCode(status: number): string {
    const errorCodes: Record<number, string> = {
      400: 'BAD_REQUEST',
      401: 'UNAUTHORIZED',
      403: 'FORBIDDEN',
      404: 'NOT_FOUND',
      409: 'CONFLICT',
      422: 'UNPROCESSABLE_ENTITY',
      429: 'TOO_MANY_REQUESTS',
      500: 'INTERNAL_SERVER_ERROR',
      502: 'BAD_GATEWAY',
      503: 'SERVICE_UNAVAILABLE',
    };
    
    return errorCodes[status] || 'UNKNOWN_ERROR';
  }
}
```

**Example Usage:**

```typescript
// app.module.ts
@Module({
  providers: [
    Logger,
    {
      provide: APP_FILTER,
      useClass: ValidationExceptionFilter, // Handle validation errors
    },
    {
      provide: APP_FILTER,
      useClass: ComprehensiveExceptionFilter, // Handle everything else
    },
  ],
})
export class AppModule {}

// Any exception in the app will be caught
@Controller('users')
export class UsersController {
  @Get(':id')
  findOne(@Param('id') id: string) {
    // Caught by global filter
    throw new NotFoundException('User not found');
  }
  
  @Post()
  create(@Body() dto: CreateUserDto) {
    // Validation errors caught by ValidationExceptionFilter
    return this.usersService.create(dto);
  }
  
  @Get('error')
  causeError() {
    // Unknown error caught by ComprehensiveExceptionFilter
    throw new Error('Something went wrong');
  }
}
```

</details>

## Best Practices

27. Should you log errors in Exception Filters?
28. Should you send detailed error information to clients?
29. What is the difference between Exception Filters and Interceptors for error handling?

<details>
<summary>Answer for Q27, Q28, Q29</summary>

**Q27: Should you log errors in Exception Filters?**

**✅ YES**, always log errors in exception filters for debugging and monitoring.

```typescript
@Injectable()
@Catch()
export class LoggingExceptionFilter implements ExceptionFilter {
  constructor(private readonly logger: Logger) {}
  
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    
    const status = exception instanceof HttpException
      ? exception.getStatus()
      : 500;
    
    // ✅ ALWAYS LOG ERRORS
    if (status >= 500) {
      // Server errors - log with full details
      this.logger.error(
        `${request.method} ${request.url} - ${status}`,
        {
          message: exception instanceof Error ? exception.message : String(exception),
          stack: exception instanceof Error ? exception.stack : undefined,
          user: request.user?.id,
          body: request.body,
          query: request.query,
          headers: {
            userAgent: request.headers['user-agent'],
            referer: request.headers['referer'],
          },
        },
      );
    } else {
      // Client errors - log with less detail
      this.logger.warn(
        `${request.method} ${request.url} - ${status}`,
        {
          message: exception instanceof Error ? exception.message : String(exception),
          user: request.user?.id,
        },
      );
    }
    
    response.status(status).json({
      statusCode: status,
      message: exception instanceof HttpException ? exception.message : 'Internal server error',
    });
  }
}
```

**Logging Best Practices:**

```typescript
// ✅ Log different levels based on severity
if (status >= 500) {
  logger.error('Server error', stack);  // Server fault
} else if (status >= 400) {
  logger.warn('Client error', message); // Client fault
} else {
  logger.log('Request', message);       // Info
}

// ✅ Include context
logger.error('Error', {
  user: request.user?.id,
  url: request.url,
  method: request.method,
  body: request.body,
  ip: request.ip,
  userAgent: request.headers['user-agent'],
});

// ✅ Use structured logging
logger.error('User creation failed', {
  operation: 'CREATE_USER',
  userId: request.user?.id,
  email: dto.email,
  error: exception.message,
  stack: exception.stack,
  timestamp: new Date().toISOString(),
});
```

---

**Q28: Should you send detailed error information to clients?**

**It depends on the environment:**

| Environment | Send Details | Reason |
|-------------|-------------|--------|
| Development | ✅ Yes | Helps debugging |
| Production | ❌ No (for 5xx), ⚠️ Limited (for 4xx) | Security & privacy |

```typescript
@Injectable()
@Catch()
export class EnvironmentAwareFilter implements ExceptionFilter {
  constructor(private configService: ConfigService) {}
  
  catch(exception: any, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    
    const status = exception instanceof HttpException ? exception.getStatus() : 500;
    const isDevelopment = this.configService.get('NODE_ENV') === 'development';
    
    if (isDevelopment) {
      // ✅ DEVELOPMENT: Send full details
      response.status(status).json({
        statusCode: status,
        message: exception.message,
        error: exception.name,
        stack: exception.stack,
        path: request.url,
        method: request.method,
        body: request.body,
        query: request.query,
        timestamp: new Date().toISOString(),
      });
    } else {
      // ✅ PRODUCTION: Send minimal info
      const errorResponse: any = {
        statusCode: status,
        timestamp: new Date().toISOString(),
        path: request.url,
      };
      
      if (status < 500) {
        // 4xx errors: can show message (client error)
        errorResponse.message = exception.message;
      } else {
        // 5xx errors: hide details (server error)
        errorResponse.message = 'Internal server error';
      }
      
      response.status(status).json(errorResponse);
    }
  }
}
```

**What NOT to send to clients:**

```typescript
// ❌ DON'T send in production:
{
  "message": "Connection to database failed: postgresql://admin:password123@db.internal:5432",
  "stack": "Error: Connection failed\n    at DatabaseService.connect (/app/src/database.ts:45:12)",
  "query": "SELECT * FROM users WHERE password = 'secret'",
  "env": {
    "DB_PASSWORD": "password123",
    "API_KEY": "secret-key"
  }
}

// ✅ DO send in production:
{
  "statusCode": 500,
  "message": "Internal server error",
  "timestamp": "2024-01-01T00:00:00.000Z"
}
```

---

**Q29: Exception Filters vs Interceptors for Error Handling?**

**Key Differences:**

| Aspect | Exception Filters | Interceptors |
|--------|------------------|--------------|
| **Purpose** | Catch and handle exceptions | Transform responses & add logic |
| **When** | After exception is thrown | Before and after handler |
| **Control Flow** | Stops execution | Continues execution |
| **Use Case** | Error handling | Success response transformation |
| **Can Catch** | Any exception | Only via catchError operator |

**Exception Filters (Recommended for Error Handling):**

```typescript
@Catch()
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    
    // ✅ Dedicated error handling
    response.status(exception.getStatus()).json({
      statusCode: exception.getStatus(),
      message: exception.message,
      timestamp: new Date().toISOString(),
    });
  }
}

// Usage
@UseFilters(HttpExceptionFilter)
@Controller('users')
export class UsersController {}
```

**Interceptors (Can handle errors, but not primary purpose):**

```typescript
@Injectable()
export class ErrorInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      // ⚠️ Can catch errors, but not recommended
      catchError(err => {
        // Transform error
        throw new HttpException('Something went wrong', 500);
      }),
    );
  }
}

// Usage
@UseInterceptors(ErrorInterceptor)
@Controller('posts')
export class PostsController {}
```

**When to use each:**

```typescript
// ✅ Exception Filter: For error handling
@Catch(NotFoundException)
export class NotFoundFilter implements ExceptionFilter {
  catch(exception: NotFoundException, host: ArgumentsHost) {
    // Format 404 errors
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    
    response.status(404).json({
      statusCode: 404,
      message: exception.message,
      timestamp: new Date().toISOString(),
    });
  }
}

// ✅ Interceptor: For response transformation
@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      // Transform successful responses
      map(data => ({
        success: true,
        data,
        timestamp: new Date().toISOString(),
      })),
    );
  }
}
```

**Combined Usage:**

```typescript
@Controller('users')
@UseInterceptors(TransformInterceptor)  // Transform success responses
@UseFilters(HttpExceptionFilter)        // Handle errors
export class UsersController {
  @Get(':id')
  findOne(@Param('id') id: string) {
    const user = this.usersService.findOne(id);
    
    if (!user) {
      // Exception Filter handles this
      throw new NotFoundException('User not found');
    }
    
    // Interceptor transforms this
    return user;
  }
}

// Success response (via Interceptor):
{
  "success": true,
  "data": { "id": "1", "name": "John" },
  "timestamp": "2024-01-01T00:00:00.000Z"
}

// Error response (via Exception Filter):
{
  "statusCode": 404,
  "message": "User not found",
  "timestamp": "2024-01-01T00:00:00.000Z"
}
```

**Best Practices Summary:**

1. **✅ Use Exception Filters** for error handling (primary purpose)
2. **✅ Use Interceptors** for response transformation (not error handling)
3. **✅ Log all errors** in Exception Filters
4. **✅ Hide error details** in production (5xx errors)
5. **✅ Show error messages** for client errors (4xx)
6. **✅ Use structured logging** for debugging
7. **❌ Don't expose** stack traces, queries, or env vars in production

</details>
