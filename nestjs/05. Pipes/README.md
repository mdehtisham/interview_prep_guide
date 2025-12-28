# NestJS Pipes - Top Interview Questions

## Pipe Fundamentals

### 1. What is a Pipe in NestJS?

<details>
<summary>Answer</summary>

A **Pipe** is a class annotated with `@Injectable()` that implements `PipeTransform` interface. Pipes transform or validate input data before it reaches the route handler.

**Two Main Use Cases:**
1. **Transformation** - Convert input data (string to number, string to date)
2. **Validation** - Validate input data, throw exception if invalid

**Basic Pipe:**
```typescript
import { PipeTransform, Injectable, ArgumentMetadata } from '@nestjs/common';

@Injectable()
export class MyPipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    // Transform or validate value
    return value;
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
Interceptors (before)
    ↓
PIPES ← You are here
    ↓
Route Handler
    ↓
Interceptors (after)
    ↓
Response
```

**Simple Example:**
```typescript
// Pipe that converts string to uppercase
@Injectable()
export class UpperCasePipe implements PipeTransform {
  transform(value: string) {
    return value.toUpperCase();
  }
}

// Usage
@Controller('users')
export class UsersController {
  @Get(':name')
  findOne(@Param('name', UpperCasePipe) name: string) {
    console.log(name); // 'JOHN' (even if you sent 'john')
    return { name };
  }
}
```

**Validation Example:**
```typescript
@Injectable()
export class ParseIntPipe implements PipeTransform<string, number> {
  transform(value: string, metadata: ArgumentMetadata): number {
    const val = parseInt(value, 10);
    if (isNaN(val)) {
      throw new BadRequestException('Validation failed');
    }
    return val;
  }
}

// Usage
@Get(':id')
findOne(@Param('id', ParseIntPipe) id: number) {
  console.log(typeof id); // 'number'
  return { id };
}
```

**Interview Tip**: Pipes run after Guards and before route handlers. Used for **validation** (check if data is valid) and **transformation** (convert data types). Implement `PipeTransform` interface with `transform()` method.

</details>

### 2. What are the two main use cases for Pipes (validation and transformation)?

<details>
<summary>Answer</summary>

Pipes have two main use cases: **Validation** and **Transformation**.

**1. Validation (Check if data is valid):**
```typescript
import { ValidationPipe } from '@nestjs/common';
import { IsEmail, IsString, MinLength } from 'class-validator';

// DTO with validation rules
export class CreateUserDto {
  @IsString()
  @MinLength(3)
  name: string;

  @IsEmail()
  email: string;
}

@Controller('users')
export class UsersController {
  @Post()
  create(@Body(ValidationPipe) dto: CreateUserDto) {
    // dto is validated before reaching here
    return dto;
  }
}

// Request with invalid data:
// POST /users { "name": "ab", "email": "invalid" }
// Response: 400 Bad Request with validation errors
```

**2. Transformation (Convert data types):**
```typescript
import { ParseIntPipe, ParseBoolPipe } from '@nestjs/common';

@Controller('posts')
export class PostsController {
  @Get(':id')
  findOne(
    @Param('id', ParseIntPipe) id: number,  // string → number
    @Query('published', ParseBoolPipe) published: boolean,  // string → boolean
  ) {
    console.log(typeof id);        // 'number'
    console.log(typeof published); // 'boolean'
    return { id, published };
  }
}

// GET /posts/123?published=true
// id: 123 (number)
// published: true (boolean)
```

**Validation vs Transformation:**
| Use Case | Purpose | Example | Throws Error |
|----------|---------|---------|--------------|
| Validation | Check if valid | Email format, min/max length | Yes, if invalid |
| Transformation | Convert type | String to number, trim spaces | No, unless conversion fails |

**Combined Example:**
```typescript
@Controller('products')
export class ProductsController {
  @Post()
  create(
    @Body(ValidationPipe) dto: CreateProductDto,  // Validation
    @Query('price', ParseIntPipe) price: number,  // Transformation
  ) {
    return { dto, price };
  }
}
```

**Custom Validation Pipe:**
```typescript
@Injectable()
export class IsPositivePipe implements PipeTransform {
  transform(value: number) {
    if (value <= 0) {
      throw new BadRequestException('Value must be positive');
    }
    return value;
  }
}
```

**Custom Transformation Pipe:**
```typescript
@Injectable()
export class TrimPipe implements PipeTransform {
  transform(value: string) {
    return value.trim();
  }
}

@Post()
create(@Body('name', TrimPipe) name: string) {
  // '  john  ' → 'john'
}
```

**Interview Tip**: Pipes have two uses - **validation** (check if data is correct, throw error if not) and **transformation** (convert data types like string to number). Use `ValidationPipe` for validation, `ParseIntPipe` for transformation.

</details>

### 3. When are Pipes executed in the request lifecycle?

<details>
<summary>Answer</summary>

Pipes execute **after Guards** and **before the route handler**.

**Request Lifecycle Order:**
```
1. Middleware
2. Guards
3. Interceptors (before)
4. PIPES ← Execute here
5. Route Handler
6. Interceptors (after)
7. Exception Filters
```

**Visual Flow:**
```typescript
Client Request
    ↓
[Middleware] - Logging, CORS, authentication
    ↓
[Guards] - Authorization, role checking
    ↓
[Interceptors (before)] - Transform request
    ↓
[PIPES] - Validate and transform parameters ← HERE
    ↓
[Route Handler] - Business logic
    ↓
[Interceptors (after)] - Transform response
    ↓
[Exception Filters] - Handle errors
    ↓
Response to Client
```

**Example with All Features:**
```typescript
@Controller('users')
@UseInterceptors(LoggingInterceptor)
export class UsersController {
  @Post()
  @UseGuards(AuthGuard)
  create(
    @Body(ValidationPipe) dto: CreateUserDto,  // Pipe executes here
  ) {
    console.log('Handler executed');
    return dto;
  }
}

// Execution order:
// 1. LoggingInterceptor (before)
// 2. AuthGuard checks if user is authenticated
// 3. ValidationPipe validates dto ← PIPES
// 4. Handler executes
// 5. LoggingInterceptor (after)
```

**Why Pipes Come After Guards:**
```typescript
@Post()
@UseGuards(RolesGuard)  // Check if user has role first
create(
  @Body(ValidationPipe) dto: CreateUserDto,  // Then validate data
) {
  return dto;
}

// Makes sense because:
// - No point validating data if user isn't authorized
// - Guards are cheaper than validation
// - Fail fast on authorization
```

**Pipe Execution on Parameters:**
```typescript
@Get(':id')
findOne(
  @Param('id', ParseIntPipe) id: number,  // Pipe 1
  @Query('limit', ParseIntPipe) limit: number,  // Pipe 2
  @Body(ValidationPipe) dto: UpdateDto,  // Pipe 3
) {
  // All pipes execute before handler
  // Order: Param → Query → Body
}
```

**Interview Tip**: Pipes execute **after Guards** and **before route handlers**. Order: Middleware → Guards → Interceptors → **Pipes** → Handler. Pipes validate/transform data right before handler receives it.

</details>

### 4. What is the `PipeTransform` interface?

<details>
<summary>Answer</summary>

`PipeTransform<T, R>` is the interface that all Pipes must implement. It has one method: `transform()`.

**Interface Definition:**
```typescript
export interface PipeTransform<T = any, R = any> {
  transform(value: T, metadata: ArgumentMetadata): R | Promise<R>;
}
```

**Type Parameters:**
- `T` - Type of input value
- `R` - Type of return value

**Basic Implementation:**
```typescript
import { PipeTransform, Injectable, ArgumentMetadata } from '@nestjs/common';

@Injectable()
export class MyPipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    // value: The input value to transform/validate
    // metadata: Information about the argument
    return value;
  }
}
```

**With Type Safety:**
```typescript
@Injectable()
export class ParseIntPipe implements PipeTransform<string, number> {
  transform(value: string, metadata: ArgumentMetadata): number {
    const num = parseInt(value, 10);
    if (isNaN(num)) {
      throw new BadRequestException('Invalid number');
    }
    return num;
  }
}
```

**Async Pipe:**
```typescript
@Injectable()
export class UserExistsPipe implements PipeTransform<string, Promise<User>> {
  constructor(private userService: UserService) {}

  async transform(userId: string, metadata: ArgumentMetadata): Promise<User> {
    const user = await this.userService.findById(userId);
    if (!user) {
      throw new NotFoundException('User not found');
    }
    return user;
  }
}

// Usage
@Get(':id')
findOne(@Param('id', UserExistsPipe) user: User) {
  // user is already loaded from database
  return user;
}
```

**Transform Method Parameters:**
```typescript
transform(
  value: any,              // The parameter value
  metadata: ArgumentMetadata // Metadata about the parameter
) {
  console.log('Value:', value);
  console.log('Type:', metadata.type);     // 'body', 'query', 'param', 'custom'
  console.log('Metatype:', metadata.metatype); // String, Number, CreateUserDto
  console.log('Data:', metadata.data);     // Parameter name
  
  return value;
}
```

**Multiple Transformations:**
```typescript
@Injectable()
export class UpperCasePipe implements PipeTransform<string, string> {
  transform(value: string): string {
    return value.toUpperCase();
  }
}

@Injectable()
export class TrimPipe implements PipeTransform<string, string> {
  transform(value: string): string {
    return value.trim();
  }
}

// Chain pipes
@Get()
search(
  @Query('q', TrimPipe, UpperCasePipe) query: string
) {
  // '  hello  ' → 'hello' → 'HELLO'
  return { query };
}
```

**Interview Tip**: All Pipes implement `PipeTransform<T, R>` interface with `transform(value, metadata)` method. Input type `T`, return type `R`. Can be sync or async. Throw exceptions for validation failures.

</details>

### 5. Can Pipes be async?

<details>
<summary>Answer</summary>

Yes, Pipes can be **async** - return a `Promise` from `transform()` method.

**Async Pipe:**
```typescript
import { PipeTransform, Injectable, NotFoundException } from '@nestjs/common';

@Injectable()
export class UserByIdPipe implements PipeTransform<string, Promise<User>> {
  constructor(private userService: UserService) {}

  async transform(userId: string): Promise<User> {
    const user = await this.userService.findById(userId);
    
    if (!user) {
      throw new NotFoundException(`User ${userId} not found`);
    }
    
    return user;
  }
}

// Usage
@Get(':id')
async findOne(@Param('id', UserByIdPipe) user: User) {
  // user is already loaded from database
  return user;
}
```

**Database Validation:**
```typescript
@Injectable()
export class EmailExistsPipe implements PipeTransform {
  constructor(private userService: UserService) {}

  async transform(email: string): Promise<string> {
    const exists = await this.userService.existsByEmail(email);
    
    if (exists) {
      throw new ConflictException('Email already exists');
    }
    
    return email;
  }
}

@Post('register')
create(
  @Body('email', EmailExistsPipe) email: string,
  @Body() dto: CreateUserDto,
) {
  // email is validated as unique
  return this.userService.create({ ...dto, email });
}
```

**External API Validation:**
```typescript
@Injectable()
export class GithubUserPipe implements PipeTransform {
  constructor(private httpService: HttpService) {}

  async transform(username: string): Promise<any> {
    try {
      const { data } = await this.httpService
        .get(`https://api.github.com/users/${username}`)
        .toPromise();
      
      return data;
    } catch (error) {
      throw new NotFoundException(`GitHub user ${username} not found`);
    }
  }
}

@Get('github/:username')
async getGithubUser(
  @Param('username', GithubUserPipe) githubUser: any,
) {
  return githubUser;
}
```

**Multiple Async Operations:**
```typescript
@Injectable()
export class ValidateRelationshipsPipe implements PipeTransform {
  constructor(
    private userService: UserService,
    private categoryService: CategoryService,
  ) {}

  async transform(dto: CreatePostDto): Promise<CreatePostDto> {
    // Validate author exists
    const author = await this.userService.findById(dto.authorId);
    if (!author) {
      throw new NotFoundException('Author not found');
    }

    // Validate category exists
    const category = await this.categoryService.findById(dto.categoryId);
    if (!category) {
      throw new NotFoundException('Category not found');
    }

    return dto;
  }
}

@Post()
create(@Body(ValidateRelationshipsPipe) dto: CreatePostDto) {
  return this.postService.create(dto);
}
```

**Async with Caching:**
```typescript
@Injectable()
export class CachedUserPipe implements PipeTransform {
  constructor(
    private userService: UserService,
    private cacheManager: Cache,
  ) {}

  async transform(userId: string): Promise<User> {
    // Try cache first
    const cached = await this.cacheManager.get<User>(`user:${userId}`);
    if (cached) {
      return cached;
    }

    // Load from database
    const user = await this.userService.findById(userId);
    if (!user) {
      throw new NotFoundException('User not found');
    }

    // Cache for next time
    await this.cacheManager.set(`user:${userId}`, user, 300);
    
    return user;
  }
}
```

**Interview Tip**: Pipes can be async - just return `Promise<T>` from `transform()`. Useful for database lookups, API calls, validation against external services. NestJS waits for Promise to resolve before calling handler.

</details>

## Built-in Pipes

### 6. What are the most commonly used built-in Pipes?

<details>
<summary>Answer</summary>

NestJS provides several built-in Pipes for common transformations and validations.

**Most Common Built-in Pipes:**

1. **ValidationPipe** - Validates DTOs with class-validator
2. **ParseIntPipe** - Converts string to integer
3. **ParseBoolPipe** - Converts string to boolean
4. **ParseFloatPipe** - Converts string to float
5. **ParseUUIDPipe** - Validates and parses UUID
6. **ParseEnumPipe** - Validates enum values
7. **ParseArrayPipe** - Parses query string arrays
8. **DefaultValuePipe** - Provides default value
9. **ParseFilePipe** - Validates uploaded files

**ValidationPipe:**
```typescript
import { ValidationPipe } from '@nestjs/common';

@Post()
create(@Body(ValidationPipe) dto: CreateUserDto) {
  return dto;
}
```

**ParseIntPipe:**
```typescript
import { ParseIntPipe } from '@nestjs/common';

@Get(':id')
findOne(@Param('id', ParseIntPipe) id: number) {
  console.log(typeof id); // 'number'
  return { id };
}
```

**ParseBoolPipe:**
```typescript
import { ParseBoolPipe } from '@nestjs/common';

@Get()
findAll(@Query('published', ParseBoolPipe) published: boolean) {
  console.log(typeof published); // 'boolean'
  return [];
}
```

**ParseUUIDPipe:**
```typescript
import { ParseUUIDPipe } from '@nestjs/common';

@Get(':id')
findOne(@Param('id', ParseUUIDPipe) id: string) {
  // Validates UUID format
  return { id };
}
```

**ParseEnumPipe:**
```typescript
enum Status {
  ACTIVE = 'active',
  INACTIVE = 'inactive',
}

@Get()
findAll(@Query('status', new ParseEnumPipe(Status)) status: Status) {
  return { status };
}
```

**DefaultValuePipe:**
```typescript
import { DefaultValuePipe } from '@nestjs/common';

@Get()
findAll(
  @Query('page', new DefaultValuePipe(1), ParseIntPipe) page: number,
) {
  // page defaults to 1 if not provided
  return { page };
}
```

**All Built-in Pipes:**
```typescript
@Controller('examples')
export class ExamplesController {
  @Get('int/:id')
  parseInt(@Param('id', ParseIntPipe) id: number) {
    return { id, type: typeof id };
  }

  @Get('float/:price')
  parseFloat(@Param('price', ParseFloatPipe) price: number) {
    return { price };
  }

  @Get('bool')
  parseBool(@Query('active', ParseBoolPipe) active: boolean) {
    return { active };
  }

  @Get('uuid/:id')
  parseUuid(@Param('id', ParseUUIDPipe) id: string) {
    return { id };
  }

  @Get('enum')
  parseEnum(@Query('status', new ParseEnumPipe(Status)) status: Status) {
    return { status };
  }

  @Get('array')
  parseArray(@Query('ids', ParseArrayPipe) ids: string[]) {
    return { ids };
  }

  @Get('default')
  defaultValue(
    @Query('page', new DefaultValuePipe(1), ParseIntPipe) page: number,
  ) {
    return { page };
  }

  @Post('validate')
  validate(@Body(ValidationPipe) dto: CreateDto) {
    return dto;
  }
}
```

**Interview Tip**: Most common built-in Pipes: `ValidationPipe` (DTO validation), `ParseIntPipe` (string to number), `ParseBoolPipe` (string to boolean), `ParseUUIDPipe` (UUID validation), `DefaultValuePipe` (default values).

</details>

### 7. What is `ValidationPipe` and when do you use it?

<details>
<summary>Answer</summary>

`ValidationPipe` validates request data against DTO classes using `class-validator` decorators.

**Basic Usage:**
```typescript
import { ValidationPipe } from '@nestjs/common';
import { IsEmail, IsString, MinLength } from 'class-validator';

// DTO with validation rules
export class CreateUserDto {
  @IsString()
  @MinLength(3)
  name: string;

  @IsEmail()
  email: string;

  @IsString()
  @MinLength(6)
  password: string;
}

@Controller('users')
export class UsersController {
  @Post()
  create(@Body(ValidationPipe) dto: CreateUserDto) {
    // dto is validated before reaching here
    return dto;
  }
}
```

**Global ValidationPipe:**
```typescript
// main.ts
import { ValidationPipe } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  app.useGlobalPipes(new ValidationPipe());
  
  await app.listen(3000);
}
```

**With Options:**
```typescript
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,              // Strip non-whitelisted properties
    forbidNonWhitelisted: true,   // Throw error for non-whitelisted
    transform: true,              // Auto-transform to DTO instance
    disableErrorMessages: false,  // Show error messages
    validationError: {
      target: false,              // Don't include target in errors
      value: false,               // Don't include value in errors
    },
  }),
);
```

**ValidationPipe Options:**
```typescript
new ValidationPipe({
  whitelist: true,
  // Remove properties not in DTO
  // { name: 'John', hacker: 'value' } → { name: 'John' }

  forbidNonWhitelisted: true,
  // Throw error if non-whitelisted property exists
  // { name: 'John', hacker: 'value' } → 400 Bad Request

  transform: true,
  // Transform plain object to DTO instance
  // Also auto-transform primitive types

  transformOptions: {
    enableImplicitConversion: true,
    // '123' → 123 for @IsNumber() fields
  },

  skipMissingProperties: false,
  // Skip validation of properties that are undefined

  skipNullProperties: false,
  // Skip validation of properties that are null

  skipUndefinedProperties: false,
  // Skip validation of properties that are undefined
})
```

**Example with All Options:**
```typescript
export class CreateUserDto {
  @IsString()
  name: string;

  @IsEmail()
  email: string;

  @IsNumber()
  age: number;
}

// Request: POST /users
// Body: { "name": "John", "email": "john@example.com", "age": "25", "hacker": "bad" }

// With whitelist: true
// → { name: 'John', email: 'john@example.com', age: 25 }
// 'hacker' is stripped, 'age' is transformed to number

// With forbidNonWhitelisted: true
// → 400 Bad Request: "property hacker should not exist"
```

**When to Use ValidationPipe:**
1. ✅ Validate POST/PUT/PATCH request bodies
2. ✅ Validate query parameters with DTOs
3. ✅ Validate URL parameters with DTOs
4. ✅ Auto-transform types (string to number)
5. ✅ Strip malicious properties

**Interview Tip**: `ValidationPipe` validates DTOs with `class-validator` decorators. Use globally with `app.useGlobalPipes()` or per-route with `@Body(ValidationPipe)`. Key options: `whitelist` (strip extra properties), `transform` (auto-convert types), `forbidNonWhitelisted` (throw on extra properties).

</details>

### 8. What is `ParseIntPipe` and how do you use it?

<details>
<summary>Answer</summary>

`ParseIntPipe` converts string parameters to integers and validates they are valid numbers.

**Basic Usage:**
```typescript
import { ParseIntPipe } from '@nestjs/common';

@Controller('users')
export class UsersController {
  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number) {
    console.log(typeof id); // 'number'
    return { id };
  }
}

// GET /users/123 → id = 123 (number)
// GET /users/abc → 400 Bad Request: "Validation failed (numeric string is expected)"
```

**Query Parameters:**
```typescript
@Get()
findAll(
  @Query('page', ParseIntPipe) page: number,
  @Query('limit', ParseIntPipe) limit: number,
) {
  console.log(typeof page);  // 'number'
  console.log(typeof limit); // 'number'
  return { page, limit };
}

// GET /users?page=1&limit=10 → page = 1, limit = 10
```

**With Default Value:**
```typescript
import { DefaultValuePipe, ParseIntPipe } from '@nestjs/common';

@Get()
findAll(
  @Query('page', new DefaultValuePipe(1), ParseIntPipe) page: number,
  @Query('limit', new DefaultValuePipe(10), ParseIntPipe) limit: number,
) {
  return { page, limit };
}

// GET /users → page = 1, limit = 10 (defaults)
// GET /users?page=2 → page = 2, limit = 10
```

**Custom Error Message:**
```typescript
@Get(':id')
findOne(
  @Param(
    'id',
    new ParseIntPipe({ errorHttpStatusCode: HttpStatus.NOT_ACCEPTABLE }),
  )
  id: number,
) {
  return { id };
}
```

**Optional Parameter:**
```typescript
@Get()
findAll(
  @Query('limit', new ParseIntPipe({ optional: true })) limit?: number,
) {
  // limit can be undefined
  return { limit: limit ?? 10 };
}

// GET /users → limit = undefined
// GET /users?limit=20 → limit = 20
```

**Multiple Integer Parameters:**
```typescript
@Get(':userId/posts/:postId')
findPost(
  @Param('userId', ParseIntPipe) userId: number,
  @Param('postId', ParseIntPipe) postId: number,
) {
  return { userId, postId };
}

// GET /123/posts/456 → userId = 123, postId = 456
```

**Interview Tip**: `ParseIntPipe` converts string to integer and validates it's a valid number. Use on params, query, or body fields. Throws `BadRequestException` if not a number. Can provide default with `DefaultValuePipe`.

</details>

### 9. What is `ParseUUIDPipe` used for?

<details>
<summary>Answer</summary>

`ParseUUIDPipe` validates that a parameter is a valid UUID format.

**Basic Usage:**
```typescript
import { ParseUUIDPipe } from '@nestjs/common';

@Controller('users')
export class UsersController {
  @Get(':id')
  findOne(@Param('id', ParseUUIDPipe) id: string) {
    return { id };
  }
}

// ✅ GET /users/550e8400-e29b-41d4-a716-446655440000 → Valid
// ❌ GET /users/123 → 400 Bad Request: "Validation failed (uuid is expected)"
```

**UUID Versions:**
```typescript
// Validate specific UUID version
@Get(':id')
findOne(
  @Param('id', new ParseUUIDPipe({ version: '4' })) id: string,
) {
  return { id };
}

// Supports: '3', '4', '5', 'all' (default)
```

**Multiple UUID Parameters:**
```typescript
@Get(':userId/posts/:postId')
findPost(
  @Param('userId', ParseUUIDPipe) userId: string,
  @Param('postId', ParseUUIDPipe) postId: string,
) {
  return { userId, postId };
}
```

**Optional UUID:**
```typescript
@Get()
findAll(
  @Query('authorId', new ParseUUIDPipe({ optional: true })) authorId?: string,
) {
  return { authorId };
}

// GET /posts → authorId = undefined
// GET /posts?authorId=550e8400-e29b-41d4-a716-446655440000 → Valid
// GET /posts?authorId=123 → 400 Bad Request
```

**With Custom Error:**
```typescript
@Get(':id')
findOne(
  @Param(
    'id',
    new ParseUUIDPipe({
      errorHttpStatusCode: HttpStatus.NOT_ACCEPTABLE,
    }),
  )
  id: string,
) {
  return { id };
}
```

**Real-World Usage:**
```typescript
// User entity
@Entity()
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;  // UUID format

  @Column()
  name: string;
}

// Controller
@Controller('users')
export class UsersController {
  constructor(private userService: UserService) {}

  @Get(':id')
  findOne(@Param('id', ParseUUIDPipe) id: string) {
    // id is validated as UUID before querying database
    return this.userService.findOne(id);
  }

  @Delete(':id')
  remove(@Param('id', ParseUUIDPipe) id: string) {
    return this.userService.remove(id);
  }
}
```

**Interview Tip**: `ParseUUIDPipe` validates UUID format before handler execution. Prevents invalid IDs from reaching database. Supports specific UUID versions ('3', '4', '5'). Throws `BadRequestException` for invalid UUIDs.

</details>

### 10. What is `DefaultValuePipe` for?

<details>
<summary>Answer</summary>

`DefaultValuePipe` provides a default value when a parameter is `undefined` or missing.

**Basic Usage:**
```typescript
import { DefaultValuePipe } from '@nestjs/common';

@Get()
findAll(
  @Query('page', new DefaultValuePipe(1)) page: number,
  @Query('limit', new DefaultValuePipe(10)) limit: number,
) {
  return { page, limit };
}

// GET /users → page = 1, limit = 10 (defaults)
// GET /users?page=2 → page = 2, limit = 10
// GET /users?page=2&limit=20 → page = 2, limit = 20
```

**With Type Transformation:**
```typescript
import { DefaultValuePipe, ParseIntPipe } from '@nestjs/common';

@Get()
findAll(
  @Query('page', new DefaultValuePipe(1), ParseIntPipe) page: number,
  @Query('limit', new DefaultValuePipe(10), ParseIntPipe) limit: number,
) {
  console.log(typeof page);  // 'number'
  console.log(typeof limit); // 'number'
  return { page, limit };
}

// Pipe chain: DefaultValuePipe → ParseIntPipe
// 1. DefaultValuePipe sets default if missing
// 2. ParseIntPipe converts to number
```

**Boolean Defaults:**
```typescript
@Get()
findAll(
  @Query('published', new DefaultValuePipe(false), ParseBoolPipe)
  published: boolean,
) {
  return { published };
}

// GET /posts → published = false
// GET /posts?published=true → published = true
```

**Array Defaults:**
```typescript
@Get()
findAll(
  @Query('tags', new DefaultValuePipe([])) tags: string[],
) {
  return { tags };
}

// GET /posts → tags = []
// GET /posts?tags=tech,news → tags = ['tech', 'news']
```

**Object Defaults:**
```typescript
@Get()
findAll(
  @Query('filter', new DefaultValuePipe({})) filter: any,
) {
  return { filter };
}
```

**Pagination Example:**
```typescript
@Controller('posts')
export class PostsController {
  @Get()
  findAll(
    @Query('page', new DefaultValuePipe(1), ParseIntPipe) page: number,
    @Query('limit', new DefaultValuePipe(10), ParseIntPipe) limit: number,
    @Query('sort', new DefaultValuePipe('createdAt')) sort: string,
    @Query('order', new DefaultValuePipe('DESC')) order: 'ASC' | 'DESC',
  ) {
    const skip = (page - 1) * limit;
    return this.postService.findAll({ skip, limit, sort, order });
  }
}

// GET /posts
// → page=1, limit=10, sort='createdAt', order='DESC'

// GET /posts?page=2&limit=20
// → page=2, limit=20, sort='createdAt', order='DESC'
```

**Interview Tip**: `DefaultValuePipe` sets default when parameter is missing or undefined. Chain with other pipes like `ParseIntPipe` for type conversion. Common for pagination (page=1, limit=10) and filters.

</details>

## Creating Custom Pipes

### 11. How do you create a custom Pipe implementing `PipeTransform`?

<details>
<summary>Answer</summary>

Create a class with `@Injectable()` decorator that implements `PipeTransform` interface.

**Basic Custom Pipe:**
```typescript
import { PipeTransform, Injectable, BadRequestException } from '@nestjs/common';

@Injectable()
export class TrimPipe implements PipeTransform<string, string> {
  transform(value: string): string {
    if (typeof value !== 'string') {
      throw new BadRequestException('Value must be a string');
    }
    return value.trim();
  }
}

// Usage
@Post()
create(@Body('name', TrimPipe) name: string) {
  // '  John  ' → 'John'
  return { name };
}
```

**Validation Pipe:**
```typescript
@Injectable()
export class IsPositivePipe implements PipeTransform<number, number> {
  transform(value: number): number {
    if (value <= 0) {
      throw new BadRequestException('Value must be positive');
    }
    return value;
  }
}

@Get(':id')
findOne(@Param('id', ParseIntPipe, IsPositivePipe) id: number) {
  return { id };
}
```

**Transformation Pipe:**
```typescript
@Injectable()
export class UpperCasePipe implements PipeTransform<string, string> {
  transform(value: string): string {
    return value.toUpperCase();
  }
}

@Get()
search(@Query('q', UpperCasePipe) query: string) {
  return { query };
}
```

**Configurable Pipe:**
```typescript
@Injectable()
export class MinLengthPipe implements PipeTransform<string, string> {
  constructor(private readonly minLength: number) {}

  transform(value: string): string {
    if (value.length < this.minLength) {
      throw new BadRequestException(
        `Value must be at least ${this.minLength} characters`,
      );
    }
    return value;
  }
}

// Usage
@Post()
create(
  @Body('name', new MinLengthPipe(3)) name: string,
  @Body('password', new MinLengthPipe(8)) password: string,
) {
  return { name, password };
}
```

**Async Pipe with DI:**
```typescript
@Injectable()
export class UserExistsPipe implements PipeTransform<string, Promise<User>> {
  constructor(private userService: UserService) {}

  async transform(userId: string): Promise<User> {
    const user = await this.userService.findById(userId);
    
    if (!user) {
      throw new NotFoundException(`User ${userId} not found`);
    }
    
    return user;
  }
}

@Get(':id')
findOne(@Param('id', UserExistsPipe) user: User) {
  // user is already loaded
  return user;
}
```

**With Metadata:**
```typescript
@Injectable()
export class LoggingPipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    console.log('Type:', metadata.type);       // 'body', 'query', 'param'
    console.log('Metatype:', metadata.metatype); // CreateUserDto
    console.log('Data:', metadata.data);       // 'email'
    console.log('Value:', value);
    
    return value;
  }
}
```

**Interview Tip**: Create custom Pipe with `@Injectable()` and `implements PipeTransform`. Implement `transform(value, metadata)` method. Return transformed value or throw exception for validation. Can inject services via constructor for async operations.

</details>

### 12. What parameters does the `transform()` method receive?

<details>
<summary>Answer</summary>

The `transform()` method receives two parameters: `value` and `metadata`.

**Method Signature:**
```typescript
transform(value: T, metadata: ArgumentMetadata): R | Promise<R>
```

**Parameters:**
1. **value** - The actual parameter value to transform/validate
2. **metadata** - Information about the parameter (type, metatype, data)

**Example:**
```typescript
@Injectable()
export class MyPipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    console.log('Value:', value);
    console.log('Metadata:', metadata);
    
    return value;
  }
}
```

**Value Parameter:**
```typescript
@Get(':id')
findOne(@Param('id', MyPipe) id: string) {
  return { id };
}

// GET /users/123
// value = '123' (the actual parameter value)
```

**Metadata Parameter:**
```typescript
export interface ArgumentMetadata {
  type: 'body' | 'query' | 'param' | 'custom';  // Parameter source
  metatype?: Type<unknown>;                       // Parameter type
  data?: string;                                  // Parameter name
}
```

**Type Property:**
```typescript
@Injectable()
export class LogTypePipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    console.log('Parameter source:', metadata.type);
    return value;
  }
}

@Post()
create(
  @Body('name', LogTypePipe) name: string,      // type: 'body'
  @Query('page', LogTypePipe) page: string,     // type: 'query'
  @Param('id', LogTypePipe) id: string,         // type: 'param'
) {}
```

**Metatype Property:**
```typescript
@Injectable()
export class LogMetatypePipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    console.log('Metatype:', metadata.metatype?.name);
    return value;
  }
}

@Post()
create(
  @Body(LogMetatypePipe) dto: CreateUserDto,  // metatype: CreateUserDto
) {}

@Get(':id')
findOne(
  @Param('id', LogMetatypePipe) id: number,   // metatype: Number
) {}
```

**Data Property:**
```typescript
@Injectable()
export class LogDataPipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    console.log('Parameter name:', metadata.data);
    return value;
  }
}

@Post()
create(
  @Body('name', LogDataPipe) name: string,     // data: 'name'
  @Body('email', LogDataPipe) email: string,   // data: 'email'
) {}
```

**All Together:**
```typescript
@Injectable()
export class DebugPipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    console.log({
      value,                        // Actual value
      type: metadata.type,          // 'body', 'query', 'param', 'custom'
      metatype: metadata.metatype,  // CreateUserDto, String, Number, etc.
      data: metadata.data,          // Parameter name
    });
    
    return value;
  }
}

@Post()
create(@Body('email', DebugPipe) email: string) {
  return { email };
}

// POST /users { "email": "john@example.com" }
// Logs:
// {
//   value: 'john@example.com',
//   type: 'body',
//   metatype: [Function: String],
//   data: 'email'
// }
```

**Using Metadata for Validation:**
```typescript
@Injectable()
export class SmartValidationPipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    // Different validation based on source
    if (metadata.type === 'param') {
      // Validate route params
      if (!/^\d+$/.test(value)) {
        throw new BadRequestException('Param must be numeric');
      }
    }
    
    if (metadata.type === 'query') {
      // Validate query params
      if (value && value.length > 100) {
        throw new BadRequestException('Query too long');
      }
    }
    
    return value;
  }
}
```

**Interview Tip**: `transform(value, metadata)` receives two params: `value` (actual parameter value) and `metadata` (parameter info). Metadata has `type` (body/query/param), `metatype` (class type), `data` (parameter name). Use metadata to customize validation logic.

</details>

### 13. What is `ArgumentMetadata` and what properties does it contain?

<details>
<summary>Answer</summary>

`ArgumentMetadata` provides information about the parameter being processed by a Pipe.

**Interface Definition:**
```typescript
export interface ArgumentMetadata {
  type: 'body' | 'query' | 'param' | 'custom';
  metatype?: Type<unknown>;
  data?: string;
}
```

**Properties:**
1. **type** - Where the parameter comes from (`'body'`, `'query'`, `'param'`, `'custom'`)
2. **metatype** - TypeScript type of the parameter (class constructor)
3. **data** - Parameter name or decorator argument

**Type Property:**
```typescript
@Post()
create(
  @Body('name') name: string,     // type: 'body'
  @Query('page') page: string,    // type: 'query'
  @Param('id') id: string,        // type: 'param'
) {}
```

**Metatype Property:**
```typescript
@Post()
create(
  @Body() dto: CreateUserDto,     // metatype: CreateUserDto
  @Param('id') id: number,        // metatype: Number
  @Query('published') pub: boolean, // metatype: Boolean
) {}
```

**Data Property:**
```typescript
@Post()
create(
  @Body('name') name: string,     // data: 'name'
  @Body('email') email: string,   // data: 'email'
  @Param('userId') userId: string, // data: 'userId'
) {}
```

**Using in Pipe:**
```typescript
@Injectable()
export class ValidationPipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    console.log('Source:', metadata.type);
    console.log('Type:', metadata.metatype?.name);
    console.log('Name:', metadata.data);
    
    // Different logic based on metadata
    if (metadata.type === 'body' && metadata.metatype) {
      // Validate DTO
      return this.validateDto(value, metadata.metatype);
    }
    
    if (metadata.type === 'param') {
      // Validate route param
      return this.validateParam(value);
    }
    
    return value;
  }
}
```

**Real Example:**
```typescript
@Injectable()
export class SmartPipe implements PipeTransform {
  transform(value: any, { type, metatype, data }: ArgumentMetadata) {
    // Check parameter source
    if (type === 'param') {
      // Route params are always strings, convert to number
      const num = parseInt(value, 10);
      if (isNaN(num)) {
        throw new BadRequestException(`${data} must be a number`);
      }
      return num;
    }
    
    // Check parameter type
    if (metatype === Number) {
      return Number(value);
    }
    
    if (metatype === Boolean) {
      return value === 'true';
    }
    
    // Check parameter name
    if (data === 'email') {
      // Validate email format
      if (!value.includes('@')) {
        throw new BadRequestException('Invalid email');
      }
    }
    
    return value;
  }
}
```

**Metadata Examples:**
```typescript
@Controller('users')
export class UsersController {
  @Post()
  create(@Body() dto: CreateUserDto) {
    // metadata = {
    //   type: 'body',
    //   metatype: CreateUserDto,
    //   data: undefined
    // }
  }

  @Post('register')
  register(@Body('email') email: string) {
    // metadata = {
    //   type: 'body',
    //   metatype: String,
    //   data: 'email'
    // }
  }

  @Get(':id')
  findOne(@Param('id') id: number) {
    // metadata = {
    //   type: 'param',
    //   metatype: Number,
    //   data: 'id'
    // }
  }

  @Get()
  findAll(@Query('page') page: number) {
    // metadata = {
    //   type: 'query',
    //   metatype: Number,
    //   data: 'page'
    // }
  }
}
```

**Interview Tip**: `ArgumentMetadata` has three properties: `type` (body/query/param/custom), `metatype` (TypeScript type like String, Number, DTO class), `data` (parameter name). Use to customize Pipe behavior based on parameter source and type.

</details>
2. What are the two main use cases for Pipes (validation and transformation)?
3. When are Pipes executed in the request lifecycle?
4. What is the `PipeTransform` interface?
5. Can Pipes be async?

## Built-in Pipes

6. What are the most commonly used built-in Pipes?
7. What is `ValidationPipe` and when do you use it?
8. What is `ParseIntPipe` and how do you use it?
9. What is `ParseUUIDPipe` used for?
10. What is `DefaultValuePipe` for?

## Creating Custom Pipes

11. How do you create a custom Pipe implementing `PipeTransform`?
12. What parameters does the `transform()` method receive?
13. What is `ArgumentMetadata` and what properties does it contain?

## Applying Pipes

### 14. How do you apply a Pipe to a route handler parameter?

<details>
<summary>Answer</summary>

Apply Pipes directly to parameters using decorators like `@Param()`, `@Body()`, `@Query()`.

**Apply to Single Parameter:**
```typescript
import { ParseIntPipe } from '@nestjs/common';

@Controller('users')
export class UsersController {
  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number) {
    return { id };
  }
}
```

**Apply Multiple Pipes:**
```typescript
@Get(':id')
findOne(
  @Param('id', ParseIntPipe, IsPositivePipe) id: number,
) {
  // Pipes execute left to right: ParseIntPipe → IsPositivePipe
  return { id };
}
```

**Apply to Body:**
```typescript
@Post()
create(@Body(ValidationPipe) dto: CreateUserDto) {
  return dto;
}
```

**Apply to Specific Body Field:**
```typescript
@Post()
create(
  @Body('email', new EmailValidationPipe()) email: string,
  @Body('age', ParseIntPipe) age: number,
) {
  return { email, age };
}
```

**Apply to Query Parameters:**
```typescript
@Get()
findAll(
  @Query('page', new DefaultValuePipe(1), ParseIntPipe) page: number,
  @Query('limit', new DefaultValuePipe(10), ParseIntPipe) limit: number,
) {
  return { page, limit };
}
```

**Apply to Multiple Parameters:**
```typescript
@Get(':userId/posts/:postId')
findPost(
  @Param('userId', ParseIntPipe) userId: number,
  @Param('postId', ParseIntPipe) postId: number,
  @Query('include', new DefaultValuePipe('all')) include: string,
) {
  return { userId, postId, include };
}
```

**Method-Level Pipes (All Parameters):**
```typescript
@Post()
@UsePipes(ValidationPipe)  // Applies to all parameters
create(
  @Body() dto: CreateUserDto,
  @Query('notify') notify: boolean,
) {
  return dto;
}
```

**Controller-Level Pipes:**
```typescript
@Controller('users')
@UsePipes(ValidationPipe)  // Applies to all routes in controller
export class UsersController {
  @Post()
  create(@Body() dto: CreateUserDto) {
    return dto;
  }

  @Put(':id')
  update(@Param('id') id: string, @Body() dto: UpdateUserDto) {
    return dto;
  }
}
```

**Interview Tip**: Apply Pipes to parameters with `@Param(name, Pipe)`, `@Body(Pipe)`, `@Query(name, Pipe)`. Chain multiple pipes (execute left to right). Use `@UsePipes()` for method/controller level application.

</details>

### 15. How do you apply ValidationPipe globally?

<details>
<summary>Answer</summary>

Apply `ValidationPipe` globally using `app.useGlobalPipes()` in `main.ts`.

**Global Setup:**
```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Apply globally to all routes
  app.useGlobalPipes(new ValidationPipe());

  await app.listen(3000);
}
bootstrap();
```

**With Options:**
```typescript
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,              // Strip non-whitelisted properties
    forbidNonWhitelisted: true,   // Throw error for non-whitelisted
    transform: true,              // Auto-transform to DTO types
    disableErrorMessages: false,  // Show validation errors
    transformOptions: {
      enableImplicitConversion: true,  // Auto-convert types
    },
  }),
);
```

**Using APP_PIPE Provider:**
```typescript
// app.module.ts
import { Module, ValidationPipe } from '@nestjs/common';
import { APP_PIPE } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_PIPE,
      useClass: ValidationPipe,
    },
  ],
})
export class AppModule {}
```

**With Custom Configuration:**
```typescript
@Module({
  providers: [
    {
      provide: APP_PIPE,
      useValue: new ValidationPipe({
        whitelist: true,
        transform: true,
      }),
    },
  ],
})
export class AppModule {}
```

**Using Factory:**
```typescript
@Module({
  providers: [
    {
      provide: APP_PIPE,
      useFactory: () => {
        return new ValidationPipe({
          whitelist: true,
          forbidNonWhitelisted: true,
          transform: true,
          transformOptions: {
            enableImplicitConversion: true,
          },
        });
      },
    },
  ],
})
export class AppModule {}
```

**Benefits of Global Pipes:**
```typescript
// No need to add ValidationPipe to every route
@Controller('users')
export class UsersController {
  @Post()
  create(@Body() dto: CreateUserDto) {
    // Automatically validated by global ValidationPipe
    return dto;
  }

  @Put(':id')
  update(@Param('id') id: string, @Body() dto: UpdateUserDto) {
    // Also validated
    return dto;
  }
}
```

**Combining Global and Route-Level:**
```typescript
// Global ValidationPipe in main.ts
app.useGlobalPipes(new ValidationPipe({ whitelist: true }));

// Additional pipe on specific route
@Controller('users')
export class UsersController {
  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number) {
    // Both global ValidationPipe and ParseIntPipe apply
    return { id };
  }
}
```

**Interview Tip**: Apply `ValidationPipe` globally in `main.ts` with `app.useGlobalPipes()` or in module with `APP_PIPE` provider. Global pipes apply to all routes automatically. Set common options like `whitelist`, `transform`, `forbidNonWhitelisted`.

</details>

### 16. What is the `APP_PIPE` token?

<details>
<summary>Answer</summary>

`APP_PIPE` is a token for registering global Pipes in a module (with DI support).

**Basic Usage:**
```typescript
import { Module } from '@nestjs/common';
import { APP_PIPE } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';

@Module({
  providers: [
    {
      provide: APP_PIPE,
      useClass: ValidationPipe,
    },
  ],
})
export class AppModule {}
```

**Why Use APP_PIPE vs app.useGlobalPipes():**
| Method | Dependency Injection | Use Case |
|--------|---------------------|----------|
| `app.useGlobalPipes()` | ❌ No DI | Simple pipes without dependencies |
| `APP_PIPE` | ✅ DI Support | Pipes that need services injected |

**With Dependency Injection:**
```typescript
// Custom pipe that needs a service
@Injectable()
export class CustomValidationPipe implements PipeTransform {
  constructor(
    private configService: ConfigService,  // Injected
    private loggerService: LoggerService,  // Injected
  ) {}

  transform(value: any, metadata: ArgumentMetadata) {
    const maxLength = this.configService.get('MAX_STRING_LENGTH');
    this.loggerService.log(`Validating: ${value}`);
    
    // Validation logic...
    return value;
  }
}

// Register with APP_PIPE (supports DI)
@Module({
  providers: [
    {
      provide: APP_PIPE,
      useClass: CustomValidationPipe,
    },
  ],
})
export class AppModule {}
```

**With useValue:**
```typescript
@Module({
  providers: [
    {
      provide: APP_PIPE,
      useValue: new ValidationPipe({
        whitelist: true,
        transform: true,
      }),
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
      provide: APP_PIPE,
      useFactory: (configService: ConfigService) => {
        return new ValidationPipe({
          whitelist: configService.get('VALIDATION_WHITELIST'),
          transform: configService.get('VALIDATION_TRANSFORM'),
        });
      },
      inject: [ConfigService],
    },
  ],
})
export class AppModule {}
```

**Multiple Global Pipes:**
```typescript
@Module({
  providers: [
    {
      provide: APP_PIPE,
      useClass: ValidationPipe,
    },
    {
      provide: APP_PIPE,
      useClass: CustomTransformPipe,
    },
    {
      provide: APP_PIPE,
      useClass: LoggingPipe,
    },
  ],
})
export class AppModule {}
```

**Interview Tip**: `APP_PIPE` token registers global Pipes in modules with **dependency injection support**. Use when Pipe needs injected services. Use `app.useGlobalPipes()` for simple pipes without dependencies.

</details>

## Validation with ValidationPipe

### 17. How do you use `ValidationPipe` for DTO validation?

<details>
<summary>Answer</summary>

Use `ValidationPipe` with DTOs decorated with `class-validator` decorators.

**Install Dependencies:**
```bash
npm install class-validator class-transformer
```

**Create DTO with Validation:**
```typescript
import { IsEmail, IsString, MinLength, IsNumber, Min, Max } from 'class-validator';

export class CreateUserDto {
  @IsString()
  @MinLength(3)
  name: string;

  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  password: string;

  @IsNumber()
  @Min(18)
  @Max(100)
  age: number;
}
```

**Apply ValidationPipe:**
```typescript
import { ValidationPipe } from '@nestjs/common';

@Controller('users')
export class UsersController {
  @Post()
  create(@Body(ValidationPipe) dto: CreateUserDto) {
    // dto is validated before reaching here
    return dto;
  }
}
```

**Global ValidationPipe:**
```typescript
// main.ts
app.useGlobalPipes(new ValidationPipe());

// Now all DTOs are validated automatically
@Controller('users')
export class UsersController {
  @Post()
  create(@Body() dto: CreateUserDto) {
    // Validated by global pipe
    return dto;
  }
}
```

**Validation Errors Response:**
```typescript
// Request: POST /users
// Body: { "name": "Jo", "email": "invalid", "password": "123", "age": 15 }

// Response: 400 Bad Request
{
  "statusCode": 400,
  "message": [
    "name must be longer than or equal to 3 characters",
    "email must be an email",
    "password must be longer than or equal to 8 characters",
    "age must not be less than 18"
  ],
  "error": "Bad Request"
}
```

**Complex DTO:**
```typescript
export class CreatePostDto {
  @IsString()
  @MinLength(10)
  @MaxLength(100)
  title: string;

  @IsString()
  @MinLength(100)
  content: string;

  @IsArray()
  @IsString({ each: true })
  tags: string[];

  @IsBoolean()
  published: boolean;

  @IsOptional()
  @IsDateString()
  publishedAt?: string;
}

@Post()
create(@Body(ValidationPipe) dto: CreatePostDto) {
  return this.postService.create(dto);
}
```

**Nested Validation:**
```typescript
export class AddressDto {
  @IsString()
  street: string;

  @IsString()
  city: string;

  @IsString()
  @Length(5, 5)
  zipCode: string;
}

export class CreateUserDto {
  @IsString()
  name: string;

  @IsEmail()
  email: string;

  @ValidateNested()
  @Type(() => AddressDto)
  address: AddressDto;
}

// Request: POST /users
{
  "name": "John",
  "email": "john@example.com",
  "address": {
    "street": "123 Main St",
    "city": "New York",
    "zipCode": "10001"
  }
}
```

**Interview Tip**: Install `class-validator` and `class-transformer`. Decorate DTO properties with validation decorators. Apply `ValidationPipe` globally or per-route. Pipe automatically validates and returns 400 with error messages on validation failure.

</details>

### 18. What is a DTO (Data Transfer Object)?

<details>
<summary>Answer</summary>

A **DTO** is a class that defines the shape and validation rules for data transferred between client and server.

**Purpose of DTOs:**
1. **Type Safety** - Define expected data structure
2. **Validation** - Add validation rules with decorators
3. **Documentation** - Self-documenting API contracts
4. **Transformation** - Convert and sanitize data

**Basic DTO:**
```typescript
export class CreateUserDto {
  name: string;
  email: string;
  password: string;
}

@Post()
create(@Body() dto: CreateUserDto) {
  return dto;
}
```

**DTO with Validation:**
```typescript
import { IsEmail, IsString, MinLength } from 'class-validator';

export class CreateUserDto {
  @IsString()
  @MinLength(3)
  name: string;

  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  password: string;
}
```

**Why Not Use Interfaces?**
```typescript
// ❌ Interfaces don't exist at runtime
interface CreateUserDto {
  name: string;
  email: string;
}

// ✅ Classes exist at runtime (can use decorators)
class CreateUserDto {
  @IsString()
  name: string;

  @IsEmail()
  email: string;
}
```

**Multiple DTOs:**
```typescript
// Create DTO
export class CreateUserDto {
  @IsString()
  name: string;

  @IsEmail()
  email: string;

  @IsString()
  password: string;
}

// Update DTO
export class UpdateUserDto {
  @IsOptional()
  @IsString()
  name?: string;

  @IsOptional()
  @IsEmail()
  email?: string;
}

// Response DTO
export class UserResponseDto {
  id: string;
  name: string;
  email: string;
  createdAt: Date;
  // No password!
}
```

**Reusing DTOs with PartialType:**
```typescript
import { PartialType } from '@nestjs/mapped-types';

export class UpdateUserDto extends PartialType(CreateUserDto) {
  // All CreateUserDto properties become optional
}
```

**Nested DTOs:**
```typescript
export class AddressDto {
  @IsString()
  street: string;

  @IsString()
  city: string;
}

export class CreateUserDto {
  @IsString()
  name: string;

  @ValidateNested()
  @Type(() => AddressDto)
  address: AddressDto;
}
```

**Interview Tip**: DTOs are **classes** (not interfaces) that define data structure and validation rules. Use `class-validator` decorators for validation. DTOs provide type safety, validation, and documentation. Use separate DTOs for create, update, and response.

</details>

### 19. What is `class-validator` library?

<details>
<summary>Answer</summary>

`class-validator` is a library that provides **validation decorators** for class properties.

**Installation:**
```bash
npm install class-validator class-transformer
```

**Basic Usage:**
```typescript
import { IsEmail, IsString, MinLength, MaxLength } from 'class-validator';

export class CreateUserDto {
  @IsString()
  @MinLength(3)
  @MaxLength(50)
  name: string;

  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  password: string;
}
```

**Common Decorators:**
```typescript
import {
  IsString,
  IsNumber,
  IsBoolean,
  IsEmail,
  IsUrl,
  IsUUID,
  IsDate,
  IsArray,
  IsEnum,
  IsOptional,
  MinLength,
  MaxLength,
  Min,
  Max,
  Length,
  Matches,
  ValidateNested,
} from 'class-validator';

export class ExampleDto {
  // String validations
  @IsString()
  name: string;

  @IsEmail()
  email: string;

  @IsUrl()
  website: string;

  // Number validations
  @IsNumber()
  @Min(0)
  @Max(100)
  age: number;

  // Boolean
  @IsBoolean()
  active: boolean;

  // Array
  @IsArray()
  @IsString({ each: true })
  tags: string[];

  // Enum
  @IsEnum(['ACTIVE', 'INACTIVE'])
  status: string;

  // Optional
  @IsOptional()
  @IsString()
  middleName?: string;

  // Custom regex
  @Matches(/^[0-9]{10}$/)
  phone: string;

  // Length
  @Length(5, 10)
  username: string;

  // Date
  @IsDate()
  @Type(() => Date)
  birthDate: Date;

  // Nested validation
  @ValidateNested()
  @Type(() => AddressDto)
  address: AddressDto;
}
```

**String Validators:**
```typescript
@IsString()           // Must be string
@MinLength(3)         // Min 3 characters
@MaxLength(50)        // Max 50 characters
@Length(5, 10)        // Between 5-10 characters
@IsEmail()            // Valid email
@IsUrl()              // Valid URL
@IsUUID()             // Valid UUID
@Matches(/regex/)     // Match regex pattern
@Contains('hello')    // Contains substring
@IsAlpha()            // Only letters
@IsAlphanumeric()     // Letters and numbers
```

**Number Validators:**
```typescript
@IsNumber()           // Must be number
@IsInt()              // Must be integer
@Min(0)               // Minimum value
@Max(100)             // Maximum value
@IsPositive()         // Must be positive
@IsNegative()         // Must be negative
@IsDivisibleBy(5)     // Divisible by 5
```

**Array Validators:**
```typescript
@IsArray()                    // Must be array
@IsString({ each: true })     // Each element is string
@ArrayMinSize(1)              // Min 1 element
@ArrayMaxSize(10)             // Max 10 elements
@ArrayNotEmpty()              // Array not empty
@ArrayUnique()                // No duplicates
```

**Used with ValidationPipe:**
```typescript
// Install
npm install class-validator class-transformer

// DTO
export class CreateUserDto {
  @IsString()
  @MinLength(3)
  name: string;

  @IsEmail()
  email: string;
}

// Controller
@Post()
create(@Body(ValidationPipe) dto: CreateUserDto) {
  return dto;
}

// Invalid request returns 400 with error messages
```

**Interview Tip**: `class-validator` provides validation decorators (`@IsString()`, `@IsEmail()`, `@MinLength()`, etc.). Install with `class-transformer`. Use decorators on DTO properties, validate with `ValidationPipe`. Supports strings, numbers, arrays, nested objects, custom validation.

</details>

### 20. What is `class-transformer` library?

<details>
<summary>Answer</summary>

`class-transformer` transforms plain JavaScript objects to class instances (and vice versa).

**Installation:**
```bash
npm install class-transformer class-validator
```

**Basic Usage:**
```typescript
import { plainToClass, classToPlain } from 'class-transformer';

class User {
  id: number;
  name: string;
  email: string;
}

// Plain object to class instance
const plain = { id: 1, name: 'John', email: 'john@example.com' };
const user = plainToClass(User, plain);
console.log(user instanceof User); // true

// Class instance to plain object
const plainAgain = classToPlain(user);
```

**Type Transformation:**
```typescript
import { Type } from 'class-transformer';

export class CreatePostDto {
  @IsString()
  title: string;

  @Type(() => Date)
  @IsDate()
  publishedAt: Date;  // Automatically converted from string to Date

  @Type(() => Number)
  @IsNumber()
  views: number;  // Automatically converted from string to number
}

// Request: { "title": "Post", "publishedAt": "2024-01-01", "views": "123" }
// After transform: { title: 'Post', publishedAt: Date object, views: 123 }
```

**Nested Transformation:**
```typescript
export class AddressDto {
  @IsString()
  street: string;

  @IsString()
  city: string;
}

export class CreateUserDto {
  @IsString()
  name: string;

  @ValidateNested()
  @Type(() => AddressDto)  // Transform to AddressDto instance
  address: AddressDto;
}
```

**Array Transformation:**
```typescript
export class TagDto {
  @IsString()
  name: string;
}

export class CreatePostDto {
  @IsString()
  title: string;

  @ValidateNested({ each: true })
  @Type(() => TagDto)
  tags: TagDto[];  // Each element transformed to TagDto instance
}
```

**Exclude Properties:**
```typescript
import { Exclude, Expose } from 'class-transformer';

export class UserResponseDto {
  @Expose()
  id: string;

  @Expose()
  name: string;

  @Expose()
  email: string;

  @Exclude()  // Don't include in response
  password: string;
}

// When returning user, password is automatically excluded
@Get(':id')
findOne(@Param('id') id: string): Promise<UserResponseDto> {
  return this.userService.findOne(id);
}
```

**Transform Decorators:**
```typescript
import { Transform } from 'class-transformer';

export class CreateUserDto {
  @Transform(({ value }) => value.toLowerCase())
  @IsEmail()
  email: string;  // Automatically lowercased

  @Transform(({ value }) => value.trim())
  @IsString()
  name: string;  // Automatically trimmed
}
```

**With ValidationPipe:**
```typescript
// Enable transformation in ValidationPipe
app.useGlobalPipes(
  new ValidationPipe({
    transform: true,  // Enable class-transformer
    transformOptions: {
      enableImplicitConversion: true,  // Auto-convert primitives
    },
  }),
);

// Now DTOs are transformed to class instances
@Post()
create(@Body() dto: CreateUserDto) {
  console.log(dto instanceof CreateUserDto); // true
  return dto;
}
```

**Interview Tip**: `class-transformer` converts plain objects to class instances and vice versa. Use `@Type()` for type conversions (string to Date, string to number). Use `@Exclude()` to hide sensitive fields. Enable with `transform: true` in ValidationPipe.

</details>

### 21. What important options can you pass to ValidationPipe (`whitelist`, `forbidNonWhitelisted`, `transform`)?

<details>
<summary>Answer</summary>

`ValidationPipe` accepts several important options to control validation behavior.

**Key Options:**
```typescript
new ValidationPipe({
  whitelist: boolean,
  forbidNonWhitelisted: boolean,
  transform: boolean,
  transformOptions: object,
  disableErrorMessages: boolean,
  skipMissingProperties: boolean,
})
```

**1. whitelist - Strip Extra Properties:**
```typescript
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,  // Remove properties not in DTO
  }),
);

export class CreateUserDto {
  @IsString()
  name: string;

  @IsEmail()
  email: string;
}

// Request: { "name": "John", "email": "john@example.com", "hacker": "malicious" }
// After whitelist: { "name": "John", "email": "john@example.com" }
// 'hacker' property is stripped
```

**2. forbidNonWhitelisted - Throw Error for Extra Properties:**
```typescript
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,
    forbidNonWhitelisted: true,  // Throw error instead of stripping
  }),
);

// Request: { "name": "John", "hacker": "bad" }
// Response: 400 Bad Request
// Error: "property hacker should not exist"
```

**3. transform - Auto Transform to Class Instance:**
```typescript
app.useGlobalPipes(
  new ValidationPipe({
    transform: true,  // Transform plain objects to DTO instances
  }),
);

@Post()
create(@Body() dto: CreateUserDto) {
  console.log(dto instanceof CreateUserDto); // true
}
```

**4. transformOptions - Implicit Type Conversion:**
```typescript
app.useGlobalPipes(
  new ValidationPipe({
    transform: true,
    transformOptions: {
      enableImplicitConversion: true,  // Auto-convert types
    },
  }),
);

export class QueryDto {
  @IsNumber()
  page: number;  // '123' → 123 (string to number)

  @IsBoolean()
  active: boolean;  // 'true' → true (string to boolean)
}
```

**5. disableErrorMessages - Hide Error Details:**
```typescript
app.useGlobalPipes(
  new ValidationPipe({
    disableErrorMessages: true,  // Don't show validation details (production)
  }),
);

// Response: 400 Bad Request (no details)
```

**All Options Together:**
```typescript
app.useGlobalPipes(
  new ValidationPipe({
    // Strip properties not in DTO
    whitelist: true,

    // Throw error if non-whitelisted property exists
    forbidNonWhitelisted: true,

    // Transform to class instance
    transform: true,

    // Transform options
    transformOptions: {
      enableImplicitConversion: true,  // Auto-convert primitives
      excludeExtraneousValues: true,   // Only include @Expose() properties
    },

    // Don't show error messages in production
    disableErrorMessages: process.env.NODE_ENV === 'production',

    // Skip validation for undefined properties
    skipMissingProperties: false,

    // Skip validation for null properties
    skipNullProperties: false,

    // Skip validation for undefined properties
    skipUndefinedProperties: false,

    // Validation error options
    validationError: {
      target: false,  // Don't include target object in error
      value: false,   // Don't include value in error
    },
  }),
);
```

**Interview Tip**: Key options: `whitelist` (strip extra properties), `forbidNonWhitelisted` (error on extra properties), `transform` (convert to class instances), `transformOptions.enableImplicitConversion` (auto-convert types). Use `whitelist: true` and `forbidNonWhitelisted: true` for security.

</details>

### 22. What does the `whitelist` option do?

<details>
<summary>Answer</summary>

`whitelist: true` **strips properties** that don't have validation decorators in the DTO.

**Without whitelist:**
```typescript
export class CreateUserDto {
  @IsString()
  name: string;

  @IsEmail()
  email: string;
}

// ValidationPipe without whitelist
new ValidationPipe({ whitelist: false })

// Request: { "name": "John", "email": "john@example.com", "isAdmin": true }
// Received: { "name": "John", "email": "john@example.com", "isAdmin": true }
// 'isAdmin' is NOT stripped (security risk!)
```

**With whitelist:**
```typescript
// ValidationPipe with whitelist
new ValidationPipe({ whitelist: true })

// Request: { "name": "John", "email": "john@example.com", "isAdmin": true }
// Received: { "name": "John", "email": "john@example.com" }
// 'isAdmin' is stripped (safe)
```

**Why It's Important (Security):**
```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  email: string;

  @Column({ default: false })
  isAdmin: boolean;  // Sensitive property
}

export class CreateUserDto {
  @IsString()
  name: string;

  @IsEmail()
  email: string;
  // No isAdmin property
}

// Without whitelist: User could send { "name": "Hacker", "email": "hack@er.com", "isAdmin": true }
// Hacker becomes admin!

// With whitelist: 'isAdmin' is stripped, user cannot become admin
```

**Example:**
```typescript
// main.ts
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,
  }),
);

// DTO
export class CreatePostDto {
  @IsString()
  title: string;

  @IsString()
  content: string;
}

// Request
POST /posts
{
  "title": "My Post",
  "content": "Content here",
  "maliciousCode": "<script>alert('XSS')</script>",
  "extraField": "ignored"
}

// Received in handler
{
  "title": "My Post",
  "content": "Content here"
}
// maliciousCode and extraField are stripped
```

**Interview Tip**: `whitelist: true` removes properties without validation decorators. Prevents users from sending extra fields like `isAdmin`, `role`, etc. **Always use in production** for security. Combine with `forbidNonWhitelisted: true` to throw errors instead of silently stripping.

</details>

### 23. What does the `transform` option enable?

<details>
<summary>Answer</summary>

`transform: true` converts plain objects to **DTO class instances** and enables **type transformation**.

**Without transform:**
```typescript
new ValidationPipe({ transform: false })

@Post()
create(@Body() dto: CreateUserDto) {
  console.log(dto instanceof CreateUserDto); // false (plain object)
  console.log(typeof dto); // 'object'
}
```

**With transform:**
```typescript
new ValidationPipe({ transform: true })

@Post()
create(@Body() dto: CreateUserDto) {
  console.log(dto instanceof CreateUserDto); // true (class instance)
}
```

**Type Transformation:**
```typescript
app.useGlobalPipes(
  new ValidationPipe({
    transform: true,
    transformOptions: {
      enableImplicitConversion: true,  // Auto-convert primitives
    },
  }),
);

export class QueryDto {
  @IsNumber()
  page: number;

  @IsBoolean()
  active: boolean;
}

@Get()
findAll(@Query() query: QueryDto) {
  console.log(typeof query.page);    // 'number' (was string)
  console.log(typeof query.active);  // 'boolean' (was string)
}

// GET /posts?page=10&active=true
// query.page = 10 (number, not '10')
// query.active = true (boolean, not 'true')
```

**Date Transformation:**
```typescript
export class CreateEventDto {
  @IsString()
  title: string;

  @Type(() => Date)
  @IsDate()
  startDate: Date;
}

// Request: { "title": "Event", "startDate": "2024-01-01T00:00:00Z" }
// With transform: true
// dto.startDate is Date object, not string
```

**Route Parameters:**
```typescript
app.useGlobalPipes(
  new ValidationPipe({
    transform: true,
    transformOptions: {
      enableImplicitConversion: true,
    },
  }),
);

@Get(':id')
findOne(@Param('id') id: number) {
  console.log(typeof id); // 'number' (was string)
  return { id };
}

// GET /users/123
// id = 123 (number), not '123' (string)
```

**Benefits:**
```typescript
// 1. Type Safety
@Post()
create(@Body() dto: CreateUserDto) {
  // dto is CreateUserDto instance, can use methods
  dto.someMethod();  // Works if CreateUserDto has this method
}

// 2. Auto Type Conversion
@Get()
findAll(@Query('limit') limit: number) {
  // limit is number, not string
  const skip = (page - 1) * limit;  // Math works directly
}

// 3. Nested Objects
export class CreateUserDto {
  @ValidateNested()
  @Type(() => AddressDto)
  address: AddressDto;  // Transformed to AddressDto instance
}
```

**With Custom Transformations:**
```typescript
import { Transform } from 'class-transformer';

export class CreateUserDto {
  @Transform(({ value }) => value.toLowerCase())
  @IsEmail()
  email: string;  // Automatically lowercased

  @Transform(({ value }) => value.trim())
  @IsString()
  name: string;  // Automatically trimmed
}
```

**Interview Tip**: `transform: true` converts plain objects to DTO class instances and enables type transformations (string to number, string to Date). Use `transformOptions.enableImplicitConversion: true` for automatic primitive conversions. Essential for type safety and avoiding manual conversions.

</details>

## Validation Decorators

### 24. What are the most common validation decorators from class-validator?

<details>
<summary>Answer</summary>

`class-validator` provides many decorators for different validation scenarios.

**String Validators:**
```typescript
import {
  IsString,
  MinLength,
  MaxLength,
  Length,
  IsEmail,
  IsUrl,
  IsUUID,
  Matches,
  Contains,
  IsAlpha,
  IsAlphanumeric,
} from 'class-validator';

export class StringValidationDto {
  @IsString()
  name: string;

  @IsString()
  @MinLength(3)
  @MaxLength(50)
  username: string;

  @IsString()
  @Length(5, 10)  // Between 5-10 characters
  code: string;

  @IsEmail()
  email: string;

  @IsUrl()
  website: string;

  @IsUUID()
  id: string;

  @Matches(/^[A-Z]{3}-\d{4}$/)  // Custom regex
  productCode: string;

  @Contains('nestjs')  // Must contain substring
  description: string;

  @IsAlpha()  // Only letters
  firstName: string;

  @IsAlphanumeric()  // Letters and numbers
  zipCode: string;
}
```

**Number Validators:**
```typescript
import {
  IsNumber,
  IsInt,
  Min,
  Max,
  IsPositive,
  IsNegative,
  IsDivisibleBy,
} from 'class-validator';

export class NumberValidationDto {
  @IsNumber()
  price: number;

  @IsInt()
  quantity: number;

  @IsNumber()
  @Min(0)
  @Max(100)
  percentage: number;

  @IsPositive()
  revenue: number;

  @IsNegative()
  debt: number;

  @IsDivisibleBy(5)
  rating: number;
}
```

**Boolean Validators:**
```typescript
import { IsBoolean } from 'class-validator';

export class BooleanValidationDto {
  @IsBoolean()
  isActive: boolean;

  @IsBoolean()
  published: boolean;
}
```

**Date Validators:**
```typescript
import { IsDate, IsDateString, MinDate, MaxDate } from 'class-validator';
import { Type } from 'class-transformer';

export class DateValidationDto {
  @IsDate()
  @Type(() => Date)
  birthDate: Date;

  @IsDateString()  // ISO 8601 date string
  createdAt: string;

  @MinDate(new Date('2020-01-01'))
  startDate: Date;

  @MaxDate(new Date('2030-12-31'))
  endDate: Date;
}
```

**Array Validators:**
```typescript
import {
  IsArray,
  ArrayMinSize,
  ArrayMaxSize,
  ArrayNotEmpty,
  ArrayUnique,
} from 'class-validator';

export class ArrayValidationDto {
  @IsArray()
  @IsString({ each: true })  // Each element must be string
  tags: string[];

  @IsArray()
  @ArrayMinSize(1)
  @ArrayMaxSize(10)
  items: string[];

  @IsArray()
  @ArrayNotEmpty()
  categories: string[];

  @IsArray()
  @ArrayUnique()
  uniqueIds: number[];
}
```

**Enum Validators:**
```typescript
import { IsEnum, IsIn } from 'class-validator';

enum UserRole {
  ADMIN = 'admin',
  USER = 'user',
  GUEST = 'guest',
}

export class EnumValidationDto {
  @IsEnum(UserRole)
  role: UserRole;

  @IsIn(['active', 'inactive', 'pending'])
  status: string;
}
```

**Object Validators:**
```typescript
import { IsObject, ValidateNested } from 'class-validator';
import { Type } from 'class-transformer';

export class AddressDto {
  @IsString()
  street: string;

  @IsString()
  city: string;
}

export class ObjectValidationDto {
  @IsObject()
  metadata: object;

  @ValidateNested()
  @Type(() => AddressDto)
  address: AddressDto;

  @ValidateNested({ each: true })
  @Type(() => AddressDto)
  addresses: AddressDto[];
}
```

**Optional Validators:**
```typescript
import { IsOptional, IsNotEmpty, IsDefined } from 'class-validator';

export class OptionalValidationDto {
  @IsOptional()  // Can be undefined
  @IsString()
  middleName?: string;

  @IsNotEmpty()  // Cannot be empty string
  name: string;

  @IsDefined()  // Cannot be undefined
  email: string;
}
```

**Most Common Summary:**
```typescript
export class CommonValidatorsDto {
  // Strings
  @IsString()
  @MinLength(3)
  @MaxLength(50)
  name: string;

  @IsEmail()
  email: string;

  // Numbers
  @IsNumber()
  @Min(0)
  @Max(100)
  age: number;

  // Booleans
  @IsBoolean()
  active: boolean;

  // Arrays
  @IsArray()
  @IsString({ each: true })
  tags: string[];

  // Enums
  @IsEnum(['ACTIVE', 'INACTIVE'])
  status: string;

  // Optional
  @IsOptional()
  @IsString()
  optional?: string;

  // Nested
  @ValidateNested()
  @Type(() => NestedDto)
  nested: NestedDto;
}
```

**Interview Tip**: Most common validators: `@IsString()`, `@IsNumber()`, `@IsEmail()`, `@MinLength()`, `@MaxLength()`, `@Min()`, `@Max()`, `@IsArray()`, `@IsEnum()`, `@IsOptional()`, `@ValidateNested()`. Import from `class-validator`.

</details>

### 25. How do you use `@IsString()`, `@IsNumber()`, `@IsEmail()`?

<details>
<summary>Answer</summary>

These decorators validate that properties are of the correct type.

**@IsString() - Validate String:**
```typescript
import { IsString } from 'class-validator';

export class CreateUserDto {
  @IsString()
  name: string;

  @IsString()
  @MinLength(3)
  username: string;
}

// Valid: { "name": "John", "username": "john123" }
// Invalid: { "name": 123, "username": true }
// Error: "name must be a string"
```

**@IsNumber() - Validate Number:**
```typescript
import { IsNumber } from 'class-validator';

export class CreateProductDto {
  @IsNumber()
  price: number;

  @IsNumber()
  @Min(0)
  quantity: number;
}

// Valid: { "price": 19.99, "quantity": 10 }
// Invalid: { "price": "19.99", "quantity": "10" }
// Error: "price must be a number"

// With transform: true, "19.99" → 19.99 (auto-converted)
```

**@IsEmail() - Validate Email:**
```typescript
import { IsEmail } from 'class-validator';

export class CreateUserDto {
  @IsEmail()
  email: string;

  @IsEmail()
  @IsOptional()
  secondaryEmail?: string;
}

// Valid: { "email": "user@example.com" }
// Invalid: { "email": "not-an-email" }
// Error: "email must be an email"
```

**Combined Usage:**
```typescript
export class CreateUserDto {
  @IsString()
  @MinLength(3)
  @MaxLength(50)
  name: string;

  @IsEmail()
  email: string;

  @IsNumber()
  @Min(18)
  @Max(100)
  age: number;

  @IsString()
  @IsOptional()
  bio?: string;
}

// Valid request:
{
  "name": "John Doe",
  "email": "john@example.com",
  "age": 25,
  "bio": "Software developer"
}

// Invalid request:
{
  "name": "Jo",  // Too short
  "email": "invalid",  // Not an email
  "age": 15  // Too young
}

// Response: 400 Bad Request
{
  "message": [
    "name must be longer than or equal to 3 characters",
    "email must be an email",
    "age must not be less than 18"
  ]
}
```

**Real-World Example:**
```typescript
export class RegisterDto {
  @IsString()
  @MinLength(2)
  @MaxLength(100)
  firstName: string;

  @IsString()
  @MinLength(2)
  @MaxLength(100)
  lastName: string;

  @IsEmail()
  @Transform(({ value }) => value.toLowerCase())
  email: string;

  @IsString()
  @MinLength(8)
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
  password: string;

  @IsNumber()
  @Min(18)
  age: number;
}

@Post('register')
async register(@Body(ValidationPipe) dto: RegisterDto) {
  return this.authService.register(dto);
}
```

**Interview Tip**: Use `@IsString()` for strings, `@IsNumber()` for numbers, `@IsEmail()` for emails. Combine with length/range validators (`@MinLength()`, `@Min()`, etc.). ValidationPipe automatically validates and returns errors for invalid types.

</details>

### 26. How do you use `@MinLength()` and `@MaxLength()`?

<details>
<summary>Answer</summary>

`@MinLength()` and `@MaxLength()` validate string length constraints.

**@MinLength() - Minimum Length:**
```typescript
import { IsString, MinLength } from 'class-validator';

export class CreateUserDto {
  @IsString()
  @MinLength(3)
  username: string;  // At least 3 characters

  @IsString()
  @MinLength(8)
  password: string;  // At least 8 characters
}

// Valid: { "username": "john", "password": "12345678" }
// Invalid: { "username": "jo", "password": "123" }
// Error: "username must be longer than or equal to 3 characters"
```

**@MaxLength() - Maximum Length:**
```typescript
import { IsString, MaxLength } from 'class-validator';

export class CreatePostDto {
  @IsString()
  @MaxLength(100)
  title: string;  // Maximum 100 characters

  @IsString()
  @MaxLength(5000)
  content: string;  // Maximum 5000 characters
}

// Valid: { "title": "Short title", "content": "..." }
// Invalid: { "title": "x".repeat(101) }
// Error: "title must be shorter than or equal to 100 characters"
```

**Combined MinLength and MaxLength:**
```typescript
export class CreateUserDto {
  @IsString()
  @MinLength(3)
  @MaxLength(20)
  username: string;  // Between 3-20 characters

  @IsString()
  @MinLength(8)
  @MaxLength(100)
  password: string;  // Between 8-100 characters

  @IsString()
  @MinLength(2)
  @MaxLength(50)
  firstName: string;
}

// Valid: { "username": "john123", "password": "securePass123", "firstName": "John" }
// Invalid: { "username": "jo", "password": "123", "firstName": "J" }
```

**Using @Length() (Alternative):**
```typescript
import { Length } from 'class-validator';

export class CreateDto {
  @IsString()
  @Length(5, 10)  // Exactly between 5-10 characters
  code: string;

  @IsString()
  @Length(5)  // Exactly 5 characters
  zipCode: string;
}
```

**With Custom Error Messages:**
```typescript
export class CreateUserDto {
  @IsString()
  @MinLength(3, {
    message: 'Username is too short (minimum 3 characters)',
  })
  @MaxLength(20, {
    message: 'Username is too long (maximum 20 characters)',
  })
  username: string;

  @IsString()
  @MinLength(8, {
    message: 'Password must be at least 8 characters',
  })
  password: string;
}
```

**Real-World Example:**
```typescript
export class CreatePostDto {
  @IsString()
  @MinLength(10, { message: 'Title must be at least 10 characters' })
  @MaxLength(100, { message: 'Title cannot exceed 100 characters' })
  title: string;

  @IsString()
  @MinLength(100, { message: 'Content must be at least 100 characters' })
  @MaxLength(10000, { message: 'Content cannot exceed 10000 characters' })
  content: string;

  @IsString()
  @MaxLength(500, { message: 'Summary cannot exceed 500 characters' })
  @IsOptional()
  summary?: string;
}
```

**Interview Tip**: Use `@MinLength(n)` for minimum string length, `@MaxLength(n)` for maximum. Combine both for range validation. Alternative: `@Length(min, max)`. Add custom messages with `{ message: '...' }` option.

</details>

### 27. How do you use `@IsOptional()` for optional fields?

<details>
<summary>Answer</summary>

`@IsOptional()` allows a property to be `undefined` or `null`, skipping validation if not provided.

**Basic Usage:**
```typescript
import { IsOptional, IsString, IsEmail } from 'class-validator';

export class CreateUserDto {
  @IsString()
  name: string;  // Required

  @IsEmail()
  email: string;  // Required

  @IsOptional()
  @IsString()
  middleName?: string;  // Optional

  @IsOptional()
  @IsString()
  bio?: string;  // Optional
}

// Valid: { "name": "John", "email": "john@example.com" }
// Valid: { "name": "John", "email": "john@example.com", "middleName": "M" }
// Invalid: { "name": "John", "email": "john@example.com", "middleName": 123 }
// Error: "middleName must be a string"
```

**With Default Values:**
```typescript
export class QueryDto {
  @IsOptional()
  @IsNumber()
  @Type(() => Number)
  page?: number = 1;  // Defaults to 1

  @IsOptional()
  @IsNumber()
  @Type(() => Number)
  limit?: number = 10;  // Defaults to 10
}

@Get()
findAll(@Query() query: QueryDto) {
  const page = query.page ?? 1;
  const limit = query.limit ?? 10;
  return { page, limit };
}
```

**Optional with Validation:**
```typescript
export class UpdateUserDto {
  @IsOptional()
  @IsString()
  @MinLength(3)
  @MaxLength(50)
  name?: string;  // If provided, must be 3-50 chars

  @IsOptional()
  @IsEmail()
  email?: string;  // If provided, must be valid email

  @IsOptional()
  @IsNumber()
  @Min(18)
  age?: number;  // If provided, must be >= 18
}

// Valid: {}  // All optional
// Valid: { "name": "John" }
// Valid: { "name": "John", "email": "john@example.com" }
// Invalid: { "name": "Jo" }  // Too short
// Error: "name must be longer than or equal to 3 characters"
```

**Optional Arrays:**
```typescript
export class CreatePostDto {
  @IsString()
  title: string;

  @IsString()
  content: string;

  @IsOptional()
  @IsArray()
  @IsString({ each: true })
  tags?: string[];  // Optional array
}

// Valid: { "title": "Post", "content": "Content" }
// Valid: { "title": "Post", "content": "Content", "tags": ["tech"] }
```

**Optional Nested Objects:**
```typescript
export class AddressDto {
  @IsString()
  street: string;

  @IsString()
  city: string;
}

export class CreateUserDto {
  @IsString()
  name: string;

  @IsOptional()
  @ValidateNested()
  @Type(() => AddressDto)
  address?: AddressDto;  // Optional nested object
}

// Valid: { "name": "John" }
// Valid: { "name": "John", "address": { "street": "Main St", "city": "NYC" } }
```

**@IsOptional() vs @IsDefined():**
```typescript
export class ExampleDto {
  @IsOptional()  // Can be undefined
  optional?: string;

  @IsDefined()  // Cannot be undefined
  required: string;

  @IsNotEmpty()  // Cannot be undefined or empty string
  notEmpty: string;
}
```

**Real-World Example:**
```typescript
export class UpdateProfileDto {
  @IsOptional()
  @IsString()
  @MinLength(2)
  @MaxLength(100)
  firstName?: string;

  @IsOptional()
  @IsString()
  @MinLength(2)
  @MaxLength(100)
  lastName?: string;

  @IsOptional()
  @IsString()
  @MaxLength(500)
  bio?: string;

  @IsOptional()
  @IsUrl()
  website?: string;

  @IsOptional()
  @IsString()
  avatarUrl?: string;
}

@Put('profile')
updateProfile(@Body() dto: UpdateProfileDto) {
  // Update only provided fields
  return this.userService.updateProfile(dto);
}
```

**Interview Tip**: Use `@IsOptional()` for optional properties that can be `undefined` or `null`. Validation only runs if value is provided. Common in Update DTOs where only changed fields are sent. Place `@IsOptional()` before other validators.

</details>

### 28. What is `@ValidateNested()` used for?

<details>
<summary>Answer</summary>

`@ValidateNested()` validates nested objects and arrays of objects recursively.

**Basic Nested Validation:**
```typescript
import { ValidateNested } from 'class-validator';
import { Type } from 'class-transformer';

export class AddressDto {
  @IsString()
  street: string;

  @IsString()
  city: string;

  @IsString()
  @Length(5, 5)
  zipCode: string;
}

export class CreateUserDto {
  @IsString()
  name: string;

  @IsEmail()
  email: string;

  @ValidateNested()  // Validate nested AddressDto
  @Type(() => AddressDto)  // Transform to AddressDto instance
  address: AddressDto;
}

// Valid:
{
  "name": "John",
  "email": "john@example.com",
  "address": {
    "street": "123 Main St",
    "city": "New York",
    "zipCode": "10001"
  }
}

// Invalid:
{
  "name": "John",
  "email": "john@example.com",
  "address": {
    "street": "123 Main St",
    "city": "New York",
    "zipCode": "123"  // Too short
  }
}
// Error: "address.zipCode must be longer than or equal to 5 characters"
```

**Array of Nested Objects:**
```typescript
export class PhoneDto {
  @IsString()
  type: string;  // 'mobile', 'home', 'work'

  @IsString()
  @Matches(/^\d{10}$/)
  number: string;
}

export class CreateUserDto {
  @IsString()
  name: string;

  @ValidateNested({ each: true })  // Validate each array element
  @Type(() => PhoneDto)
  @IsArray()
  phones: PhoneDto[];
}

// Valid:
{
  "name": "John",
  "phones": [
    { "type": "mobile", "number": "1234567890" },
    { "type": "work", "number": "0987654321" }
  ]
}
```

**Deep Nesting:**
```typescript
export class CountryDto {
  @IsString()
  name: string;

  @IsString()
  code: string;
}

export class AddressDto {
  @IsString()
  street: string;

  @IsString()
  city: string;

  @ValidateNested()
  @Type(() => CountryDto)
  country: CountryDto;
}

export class CreateUserDto {
  @IsString()
  name: string;

  @ValidateNested()
  @Type(() => AddressDto)
  address: AddressDto;
}

// Valid:
{
  "name": "John",
  "address": {
    "street": "Main St",
    "city": "NYC",
    "country": {
      "name": "United States",
      "code": "US"
    }
  }
}
```

**Optional Nested:**
```typescript
export class CreateUserDto {
  @IsString()
  name: string;

  @IsOptional()
  @ValidateNested()
  @Type(() => AddressDto)
  address?: AddressDto;  // Optional nested object
}

// Valid: { "name": "John" }
// Valid: { "name": "John", "address": { "street": "...", "city": "..." } }
```

**Real-World Example:**
```typescript
export class TagDto {
  @IsString()
  name: string;
}

export class AuthorDto {
  @IsString()
  name: string;

  @IsEmail()
  email: string;
}

export class CreatePostDto {
  @IsString()
  @MinLength(10)
  title: string;

  @IsString()
  @MinLength(100)
  content: string;

  @ValidateNested()
  @Type(() => AuthorDto)
  author: AuthorDto;

  @ValidateNested({ each: true })
  @Type(() => TagDto)
  @IsArray()
  tags: TagDto[];
}

@Post()
create(@Body(ValidationPipe) dto: CreatePostDto) {
  return this.postService.create(dto);
}

// Request:
{
  "title": "My Blog Post",
  "content": "Lorem ipsum...",
  "author": {
    "name": "John Doe",
    "email": "john@example.com"
  },
  "tags": [
    { "name": "tech" },
    { "name": "typescript" }
  ]
}
```

**Interview Tip**: Use `@ValidateNested()` to validate nested objects and `@ValidateNested({ each: true })` for arrays of objects. Must use with `@Type(() => DtoClass)` from `class-transformer`. Enables deep validation of complex nested structures.

</details>

## Transformation

### 29. How do you use Pipes for data transformation?

<details>
<summary>Answer</summary>

Pipes transform data from one type to another (string to number, trim strings, etc.).

**Built-in Transformation Pipes:**
```typescript
import { ParseIntPipe, ParseBoolPipe, ParseFloatPipe } from '@nestjs/common';

@Controller('posts')
export class PostsController {
  @Get(':id')
  findOne(
    @Param('id', ParseIntPipe) id: number,  // string → number
    @Query('published', ParseBoolPipe) published: boolean,  // string → boolean
    @Query('rating', ParseFloatPipe) rating: number,  // string → float
  ) {
    console.log(typeof id);        // 'number'
    console.log(typeof published); // 'boolean'
    console.log(typeof rating);    // 'number'
    return { id, published, rating };
  }
}

// GET /posts/123?published=true&rating=4.5
// id = 123 (number)
// published = true (boolean)
// rating = 4.5 (number)
```

**Custom Transformation Pipe:**
```typescript
@Injectable()
export class TrimPipe implements PipeTransform<string, string> {
  transform(value: string): string {
    return value.trim();
  }
}

@Post()
create(@Body('name', TrimPipe) name: string) {
  // '  John  ' → 'John'
  return { name };
}
```

**Transform to Uppercase:**
```typescript
@Injectable()
export class UpperCasePipe implements PipeTransform<string, string> {
  transform(value: string): string {
    return value.toUpperCase();
  }
}

@Get()
search(@Query('q', UpperCasePipe) query: string) {
  // 'hello' → 'HELLO'
  return { query };
}
```

**Transform with class-transformer:**
```typescript
import { Transform } from 'class-transformer';

export class CreateUserDto {
  @Transform(({ value }) => value.trim())
  @IsString()
  name: string;

  @Transform(({ value }) => value.toLowerCase())
  @IsEmail()
  email: string;

  @Transform(({ value }) => parseInt(value, 10))
  @IsNumber()
  age: number;
}

@Post()
create(@Body(ValidationPipe) dto: CreateUserDto) {
  // name is trimmed, email is lowercased, age is converted to number
  return dto;
}
```

**Auto-transformation with ValidationPipe:**
```typescript
// Enable in main.ts
app.useGlobalPipes(
  new ValidationPipe({
    transform: true,  // Enable transformation
    transformOptions: {
      enableImplicitConversion: true,  // Auto-convert primitives
    },
  }),
);

// Now types are auto-converted
export class QueryDto {
  @IsNumber()
  page: number;  // '123' → 123

  @IsBoolean()
  active: boolean;  // 'true' → true

  @IsDate()
  @Type(() => Date)
  date: Date;  // '2024-01-01' → Date object
}

@Get()
findAll(@Query() query: QueryDto) {
  console.log(typeof query.page);    // 'number'
  console.log(typeof query.active);  // 'boolean'
  console.log(query.date instanceof Date); // true
  return query;
}
```

**Chain Multiple Transformations:**
```typescript
@Get()
search(
  @Query('q', TrimPipe, UpperCasePipe) query: string
) {
  // '  hello  ' → 'hello' → 'HELLO'
  return { query };
}
```

**Interview Tip**: Use built-in pipes (`ParseIntPipe`, `ParseBoolPipe`) for common transformations. Create custom pipes implementing `PipeTransform` for custom logic. Enable `transform: true` in ValidationPipe for automatic type conversions. Use `@Transform()` decorator for field-level transformations.

</details>

### 30. How do you convert string to number using `ParseIntPipe`?

<details>
<summary>Answer</summary>

`ParseIntPipe` automatically converts string parameters to integers.

**Route Parameters:**
```typescript
import { ParseIntPipe } from '@nestjs/common';

@Controller('users')
export class UsersController {
  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number) {
    console.log(typeof id); // 'number'
    console.log(id);        // 123
    return { id };
  }
}

// GET /users/123 → id = 123 (number, not '123')
// GET /users/abc → 400 Bad Request: "Validation failed (numeric string is expected)"
```

**Query Parameters:**
```typescript
@Get()
findAll(
  @Query('page', ParseIntPipe) page: number,
  @Query('limit', ParseIntPipe) limit: number,
) {
  console.log(typeof page);  // 'number'
  console.log(typeof limit); // 'number'
  
  const skip = (page - 1) * limit;  // Math works directly
  return { page, limit, skip };
}

// GET /users?page=2&limit=10
// page = 2, limit = 10 (both numbers)
```

**With Default Value:**
```typescript
import { DefaultValuePipe, ParseIntPipe } from '@nestjs/common';

@Get()
findAll(
  @Query('page', new DefaultValuePipe(1), ParseIntPipe) page: number,
  @Query('limit', new DefaultValuePipe(10), ParseIntPipe) limit: number,
) {
  return { page, limit };
}

// GET /users → page = 1, limit = 10 (defaults)
// GET /users?page=5 → page = 5, limit = 10
```

**With Options:**
```typescript
@Get(':id')
findOne(
  @Param(
    'id',
    new ParseIntPipe({
      errorHttpStatusCode: HttpStatus.NOT_ACCEPTABLE,  // Custom status
      exceptionFactory: (error) => new BadRequestException('Invalid ID'),
    }),
  )
  id: number,
) {
  return { id };
}
```

**Optional Integer:**
```typescript
@Get()
findAll(
  @Query('userId', new ParseIntPipe({ optional: true })) userId?: number,
) {
  // userId can be undefined
  return { userId };
}

// GET /posts → userId = undefined
// GET /posts?userId=123 → userId = 123
```

**Alternative: Auto-transform with ValidationPipe:**
```typescript
// Enable globally
app.useGlobalPipes(
  new ValidationPipe({
    transform: true,
    transformOptions: {
      enableImplicitConversion: true,
    },
  }),
);

// DTO
export class QueryDto {
  @IsNumber()
  page: number;  // Automatically converted from string

  @IsNumber()
  limit: number;
}

@Get()
findAll(@Query() query: QueryDto) {
  console.log(typeof query.page);  // 'number'
  return query;
}

// GET /posts?page=2&limit=10
// query.page = 2 (number, not '2')
```

**Interview Tip**: Use `ParseIntPipe` to convert string params to integers. Throws `BadRequestException` if not a valid number. Use with `@Param()`, `@Query()`, or `@Body()`. Alternative: enable `transform: true` in ValidationPipe for automatic conversion.

</details>

### 31. What is auto-transformation in ValidationPipe?

<details>
<summary>Answer</summary>

Auto-transformation automatically converts plain objects to DTO class instances and converts primitive types.

**Enable Auto-transformation:**
```typescript
// main.ts
app.useGlobalPipes(
  new ValidationPipe({
    transform: true,  // Enable auto-transformation
    transformOptions: {
      enableImplicitConversion: true,  // Auto-convert primitives
    },
  }),
);
```

**Type Conversion:**
```typescript
export class QueryDto {
  @IsNumber()
  page: number;  // '10' → 10

  @IsBoolean()
  published: boolean;  // 'true' → true

  @IsNumber()
  limit: number;  // '20' → 20
}

@Get()
findAll(@Query() query: QueryDto) {
  console.log(typeof query.page);      // 'number' (not 'string')
  console.log(typeof query.published); // 'boolean' (not 'string')
  console.log(typeof query.limit);     // 'number' (not 'string')
  return query;
}

// GET /posts?page=2&published=true&limit=10
// query = { page: 2, published: true, limit: 10 }
// All types auto-converted!
```

**Date Transformation:**
```typescript
import { Type } from 'class-transformer';

export class CreateEventDto {
  @IsString()
  title: string;

  @Type(() => Date)
  @IsDate()
  startDate: Date;  // '2024-01-01' → Date object

  @Type(() => Date)
  @IsDate()
  endDate: Date;
}

// Request: { "title": "Event", "startDate": "2024-01-01", "endDate": "2024-12-31" }
// After transform: startDate and endDate are Date objects
```

**Class Instance Transformation:**
```typescript
export class CreateUserDto {
  @IsString()
  name: string;

  @IsEmail()
  email: string;
}

@Post()
create(@Body() dto: CreateUserDto) {
  console.log(dto instanceof CreateUserDto); // true (with transform)
  return dto;
}

// Without transform: dto is plain object
// With transform: dto is CreateUserDto instance
```

**Route Parameters:**
```typescript
export class ParamDto {
  @IsNumber()
  id: number;  // Auto-converted from string
}

@Get(':id')
findOne(@Param() params: ParamDto) {
  console.log(typeof params.id); // 'number'
  return params;
}

// GET /users/123 → params.id = 123 (number)
```

**Benefits:**
```typescript
// Without transform (default)
@Get()
findAll(@Query('page') page: string) {
  const pageNum = parseInt(page, 10);  // Manual conversion
  return { page: pageNum };
}

// With transform
@Get()
findAll(@Query('page') page: number) {
  // page is already a number!
  return { page };
}
```

**Nested Objects:**
```typescript
export class AddressDto {
  @IsString()
  street: string;

  @IsString()
  city: string;
}

export class CreateUserDto {
  @IsString()
  name: string;

  @ValidateNested()
  @Type(() => AddressDto)
  address: AddressDto;  // Transformed to AddressDto instance
}

@Post()
create(@Body() dto: CreateUserDto) {
  console.log(dto.address instanceof AddressDto); // true
}
```

**Interview Tip**: Enable auto-transformation with `transform: true` and `transformOptions.enableImplicitConversion: true` in ValidationPipe. Automatically converts strings to numbers/booleans, plain objects to class instances, date strings to Date objects. Eliminates manual type conversions.

</details>

## Error Handling

### 32. What exception do Pipes throw on validation failure?

<details>
<summary>Answer</summary>

Pipes throw `BadRequestException` (400) on validation failure by default.

**Default Behavior:**
```typescript
import { ParseIntPipe } from '@nestjs/common';

@Get(':id')
findOne(@Param('id', ParseIntPipe) id: number) {
  return { id };
}

// GET /users/abc
// Response: 400 Bad Request
{
  "statusCode": 400,
  "message": "Validation failed (numeric string is expected)",
  "error": "Bad Request"
}
```

**ValidationPipe Errors:**
```typescript
export class CreateUserDto {
  @IsString()
  @MinLength(3)
  name: string;

  @IsEmail()
  email: string;

  @IsNumber()
  @Min(18)
  age: number;
}

@Post()
create(@Body(ValidationPipe) dto: CreateUserDto) {
  return dto;
}

// Invalid request:
// POST /users { "name": "Jo", "email": "invalid", "age": 15 }

// Response: 400 Bad Request
{
  "statusCode": 400,
  "message": [
    "name must be longer than or equal to 3 characters",
    "email must be an email",
    "age must not be less than 18"
  ],
  "error": "Bad Request"
}
```

**Custom Exception in Pipe:**
```typescript
@Injectable()
export class CustomPipe implements PipeTransform {
  transform(value: any) {
    if (!value) {
      throw new BadRequestException('Value is required');
    }
    
    if (typeof value !== 'string') {
      throw new BadRequestException('Value must be a string');
    }
    
    return value;
  }
}
```

**Different Exception Types:**
```typescript
@Injectable()
export class StrictValidationPipe implements PipeTransform {
  transform(value: any) {
    if (!value) {
      // 400 Bad Request
      throw new BadRequestException('Missing value');
    }
    
    if (value === 'forbidden') {
      // 403 Forbidden
      throw new ForbiddenException('This value is forbidden');
    }
    
    if (value === 'notfound') {
      // 404 Not Found
      throw new NotFoundException('Value not found');
    }
    
    return value;
  }
}
```

**Custom Error with ParseIntPipe:**
```typescript
@Get(':id')
findOne(
  @Param(
    'id',
    new ParseIntPipe({
      exceptionFactory: (error) => {
        return new BadRequestException(`Invalid ID: ${error}`);
      },
    }),
  )
  id: number,
) {
  return { id };
}

// GET /users/abc
// Response: 400 Bad Request "Invalid ID: Validation failed"
```

**Interview Tip**: Pipes throw `BadRequestException` (400) by default. ValidationPipe returns array of validation error messages. Custom pipes can throw any HttpException. Use `exceptionFactory` option to customize error messages.

</details>

### 33. How do you customize validation error messages?

<details>
<summary>Answer</summary>

Customize error messages using decorator options or custom exception factories.

**Decorator-Level Messages:**
```typescript
export class CreateUserDto {
  @IsString({ message: 'Name must be a string' })
  @MinLength(3, { message: 'Name must be at least 3 characters long' })
  @MaxLength(50, { message: 'Name cannot exceed 50 characters' })
  name: string;

  @IsEmail({}, { message: 'Please provide a valid email address' })
  email: string;

  @IsNumber({}, { message: 'Age must be a number' })
  @Min(18, { message: 'You must be at least 18 years old' })
  @Max(100, { message: 'Age cannot exceed 100' })
  age: number;
}

// Invalid request: { "name": "Jo", "email": "invalid", "age": 15 }
// Response:
{
  "statusCode": 400,
  "message": [
    "Name must be at least 3 characters long",
    "Please provide a valid email address",
    "You must be at least 18 years old"
  ]
}
```

**Dynamic Messages with Context:**
```typescript
export class CreatePostDto {
  @IsString()
  @MinLength(10, {
    message: (args) => {
      return `Title is too short. Minimum length is ${args.constraints[0]} characters`;
    },
  })
  title: string;

  @IsArray()
  @ArrayMinSize(1, {
    message: (args) => {
      return `You must provide at least ${args.constraints[0]} tag`;
    },
  })
  tags: string[];
}
```

**Custom Exception Factory:**
```typescript
// main.ts
app.useGlobalPipes(
  new ValidationPipe({
    exceptionFactory: (errors) => {
      const messages = errors.map((error) => ({
        field: error.property,
        errors: Object.values(error.constraints || {}),
      }));
      
      return new BadRequestException({
        statusCode: 400,
        message: 'Validation failed',
        errors: messages,
      });
    },
  }),
);

// Response:
{
  "statusCode": 400,
  "message": "Validation failed",
  "errors": [
    {
      "field": "name",
      "errors": ["name must be longer than or equal to 3 characters"]
    },
    {
      "field": "email",
      "errors": ["email must be an email"]
    }
  ]
}
```

**Custom Validation Messages:**
```typescript
export class RegisterDto {
  @IsString({ message: 'Username is required and must be a string' })
  @MinLength(3, { message: 'Username must be at least 3 characters' })
  @MaxLength(20, { message: 'Username cannot be longer than 20 characters' })
  @Matches(/^[a-zA-Z0-9_]+$/, {
    message: 'Username can only contain letters, numbers, and underscores',
  })
  username: string;

  @IsEmail({}, { message: 'Invalid email format' })
  email: string;

  @IsString({ message: 'Password is required' })
  @MinLength(8, { message: 'Password must be at least 8 characters' })
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/, {
    message: 'Password must contain uppercase, lowercase, and number',
  })
  password: string;
}
```

**Internationalization (i18n):**
```typescript
// Create message constants
const ValidationMessages = {
  NAME_REQUIRED: 'Name is required',
  NAME_MIN_LENGTH: 'Name must be at least 3 characters',
  EMAIL_INVALID: 'Please provide a valid email',
  AGE_MIN: 'You must be at least 18 years old',
};

export class CreateUserDto {
  @IsString({ message: ValidationMessages.NAME_REQUIRED })
  @MinLength(3, { message: ValidationMessages.NAME_MIN_LENGTH })
  name: string;

  @IsEmail({}, { message: ValidationMessages.EMAIL_INVALID })
  email: string;

  @Min(18, { message: ValidationMessages.AGE_MIN })
  age: number;
}
```

**Interview Tip**: Customize messages with `{ message: 'text' }` option in decorators. Use `exceptionFactory` in ValidationPipe for custom error format. Dynamic messages with function: `message: (args) => {...}`. Store messages in constants for reusability and i18n.

</details>

### 34. What is the structure of ValidationPipe error responses?

<details>
<summary>Answer</summary>

ValidationPipe returns a structured error response with status 400 and validation messages.

**Default Error Structure:**
```typescript
{
  "statusCode": 400,
  "message": [
    "name must be longer than or equal to 3 characters",
    "email must be an email",
    "age must not be less than 18"
  ],
  "error": "Bad Request"
}
```

**Example Request/Response:**
```typescript
export class CreateUserDto {
  @IsString()
  @MinLength(3)
  name: string;

  @IsEmail()
  email: string;

  @IsNumber()
  @Min(18)
  age: number;
}

// Invalid Request:
POST /users
{
  "name": "Jo",
  "email": "invalid",
  "age": 15
}

// Response: 400 Bad Request
{
  "statusCode": 400,
  "message": [
    "name must be longer than or equal to 3 characters",
    "email must be an email",
    "age must not be less than 18"
  ],
  "error": "Bad Request"
}
```

**Single Error:**
```typescript
// Request:
POST /users { "name": "Jo", "email": "john@example.com", "age": 25 }

// Response:
{
  "statusCode": 400,
  "message": [
    "name must be longer than or equal to 3 characters"
  ],
  "error": "Bad Request"
}
```

**Nested Object Errors:**
```typescript
export class AddressDto {
  @IsString()
  street: string;

  @IsString()
  @Length(5, 5)
  zipCode: string;
}

export class CreateUserDto {
  @IsString()
  name: string;

  @ValidateNested()
  @Type(() => AddressDto)
  address: AddressDto;
}

// Invalid Request:
{
  "name": "John",
  "address": {
    "street": "",
    "zipCode": "123"
  }
}

// Response:
{
  "statusCode": 400,
  "message": [
    "address.street should not be empty",
    "address.zipCode must be longer than or equal to 5 characters"
  ],
  "error": "Bad Request"
}
```

**Custom Error Structure:**
```typescript
app.useGlobalPipes(
  new ValidationPipe({
    exceptionFactory: (errors: ValidationError[]) => {
      const formattedErrors = errors.map((error) => ({
        property: error.property,
        value: error.value,
        constraints: error.constraints,
        children: error.children,
      }));
      
      return new BadRequestException({
        statusCode: 400,
        message: 'Validation failed',
        errors: formattedErrors,
      });
    },
  }),
);

// Response:
{
  "statusCode": 400,
  "message": "Validation failed",
  "errors": [
    {
      "property": "name",
      "value": "Jo",
      "constraints": {
        "minLength": "name must be longer than or equal to 3 characters"
      }
    },
    {
      "property": "email",
      "value": "invalid",
      "constraints": {
        "isEmail": "email must be an email"
      }
    }
  ]
}
```

**Simplified Error Structure:**
```typescript
app.useGlobalPipes(
  new ValidationPipe({
    exceptionFactory: (errors: ValidationError[]) => {
      const messages = {};
      
      errors.forEach((error) => {
        messages[error.property] = Object.values(error.constraints || {})[0];
      });
      
      return new BadRequestException({
        statusCode: 400,
        errors: messages,
      });
    },
  }),
);

// Response:
{
  "statusCode": 400,
  "errors": {
    "name": "name must be longer than or equal to 3 characters",
    "email": "email must be an email",
    "age": "age must not be less than 18"
  }
}
```

**Interview Tip**: Default structure has `statusCode: 400`, `message` (array of error strings), `error: "Bad Request"`. Nested errors use dot notation (`address.zipCode`). Customize with `exceptionFactory` to format errors differently. ValidationError contains `property`, `value`, `constraints`, `children`.

</details>

## File Upload Pipes

### 35. How do you validate file uploads using `ParseFilePipe`?

<details>
<summary>Answer</summary>

`ParseFilePipe` validates uploaded files (size, type, etc.) before handler execution.

**Basic File Validation:**
```typescript
import { 
  ParseFilePipe, 
  MaxFileSizeValidator, 
  FileTypeValidator 
} from '@nestjs/common';

@Post('upload')
@UseInterceptors(FileInterceptor('file'))
uploadFile(
  @UploadedFile(
    new ParseFilePipe({
      validators: [
        new MaxFileSizeValidator({ maxSize: 1024 * 1024 * 5 }), // 5MB
        new FileTypeValidator({ fileType: 'image/jpeg' }),
      ],
    }),
  )
  file: Express.Multer.File,
) {
  return {
    filename: file.originalname,
    size: file.size,
    mimetype: file.mimetype,
  };
}
```

**Multiple File Types:**
```typescript
@Post('upload')
@UseInterceptors(FileInterceptor('file'))
uploadFile(
  @UploadedFile(
    new ParseFilePipe({
      validators: [
        new MaxFileSizeValidator({ maxSize: 10 * 1024 * 1024 }), // 10MB
        new FileTypeValidator({ fileType: '(jpeg|jpg|png|gif)' }), // Regex
      ],
    }),
  )
  file: Express.Multer.File,
) {
  return { message: 'File uploaded successfully', file };
}
```

**Optional File Upload:**
```typescript
@Post('upload')
@UseInterceptors(FileInterceptor('file'))
uploadFile(
  @UploadedFile(
    new ParseFilePipe({
      validators: [
        new MaxFileSizeValidator({ maxSize: 5 * 1024 * 1024 }),
      ],
      fileIsRequired: false,  // File is optional
    }),
  )
  file?: Express.Multer.File,
) {
  if (!file) {
    return { message: 'No file uploaded' };
  }
  return { file };
}
```

**Custom File Validator:**
```typescript
import { FileValidator } from '@nestjs/common';

export class CustomImageValidator extends FileValidator {
  constructor(private readonly allowedExtensions: string[]) {
    super({});
  }

  isValid(file: Express.Multer.File): boolean {
    const extension = file.originalname.split('.').pop();
    return this.allowedExtensions.includes(extension.toLowerCase());
  }

  buildErrorMessage(): string {
    return `File extension must be one of: ${this.allowedExtensions.join(', ')}`;
  }
}

@Post('upload')
@UseInterceptors(FileInterceptor('image'))
uploadImage(
  @UploadedFile(
    new ParseFilePipe({
      validators: [
        new MaxFileSizeValidator({ maxSize: 5 * 1024 * 1024 }),
        new CustomImageValidator(['jpg', 'jpeg', 'png', 'webp']),
      ],
    }),
  )
  image: Express.Multer.File,
) {
  return { image };
}
```

**Multiple Files:**
```typescript
@Post('upload-multiple')
@UseInterceptors(FilesInterceptor('files', 10))  // Max 10 files
uploadFiles(
  @UploadedFiles(
    new ParseFilePipe({
      validators: [
        new MaxFileSizeValidator({ maxSize: 2 * 1024 * 1024 }), // 2MB per file
        new FileTypeValidator({ fileType: 'image/*' }),
      ],
    }),
  )
  files: Express.Multer.File[],
) {
  return {
    count: files.length,
    files: files.map((f) => ({ name: f.originalname, size: f.size })),
  };
}
```

**Interview Tip**: Use `ParseFilePipe` with validators: `MaxFileSizeValidator` (size limit), `FileTypeValidator` (MIME type). Apply to `@UploadedFile()` decorator. Set `fileIsRequired: false` for optional uploads. Create custom validators extending `FileValidator`.

</details>

### 36. How do you restrict file size and types?

<details>
<summary>Answer</summary>

Restrict file size and types using `MaxFileSizeValidator` and `FileTypeValidator`.

**Restrict File Size:**
```typescript
import { MaxFileSizeValidator } from '@nestjs/common';

@Post('upload')
@UseInterceptors(FileInterceptor('file'))
uploadFile(
  @UploadedFile(
    new ParseFilePipe({
      validators: [
        new MaxFileSizeValidator({ maxSize: 1024 * 1024 * 5 }), // 5MB in bytes
      ],
    }),
  )
  file: Express.Multer.File,
) {
  return { file };
}

// File > 5MB → 400 Bad Request: "File size exceeds maximum allowed size"
```

**Restrict File Type (MIME Type):**
```typescript
import { FileTypeValidator } from '@nestjs/common';

@Post('upload-image')
@UseInterceptors(FileInterceptor('image'))
uploadImage(
  @UploadedFile(
    new ParseFilePipe({
      validators: [
        new FileTypeValidator({ fileType: 'image/jpeg' }),
      ],
    }),
  )
  image: Express.Multer.File,
) {
  return { image };
}

// Non-JPEG file → 400 Bad Request: "File type does not match"
```

**Multiple File Types (Regex):**
```typescript
@Post('upload')
@UseInterceptors(FileInterceptor('file'))
uploadFile(
  @UploadedFile(
    new ParseFilePipe({
      validators: [
        new FileTypeValidator({ fileType: 'image/(jpeg|jpg|png|gif)' }),
      ],
    }),
  )
  file: Express.Multer.File,
) {
  return { file };
}

// Accepts: JPEG, JPG, PNG, GIF
```

**Combined Size and Type:**
```typescript
@Post('upload-avatar')
@UseInterceptors(FileInterceptor('avatar'))
uploadAvatar(
  @UploadedFile(
    new ParseFilePipe({
      validators: [
        new MaxFileSizeValidator({ maxSize: 2 * 1024 * 1024 }), // 2MB
        new FileTypeValidator({ fileType: 'image/(jpeg|png)' }), // JPEG or PNG
      ],
    }),
  )
  avatar: Express.Multer.File,
) {
  return {
    message: 'Avatar uploaded',
    size: avatar.size,
    type: avatar.mimetype,
  };
}
```

**PDF Files:**
```typescript
@Post('upload-document')
@UseInterceptors(FileInterceptor('document'))
uploadDocument(
  @UploadedFile(
    new ParseFilePipe({
      validators: [
        new MaxFileSizeValidator({ maxSize: 10 * 1024 * 1024 }), // 10MB
        new FileTypeValidator({ fileType: 'application/pdf' }),
      ],
    }),
  )
  document: Express.Multer.File,
) {
  return { document };
}
```

**Video Files:**
```typescript
@Post('upload-video')
@UseInterceptors(FileInterceptor('video'))
uploadVideo(
  @UploadedFile(
    new ParseFilePipe({
      validators: [
        new MaxFileSizeValidator({ maxSize: 50 * 1024 * 1024 }), // 50MB
        new FileTypeValidator({ fileType: 'video/(mp4|avi|mov)' }),
      ],
    }),
  )
  video: Express.Multer.File,
) {
  return { video };
}
```

**Custom Error Messages:**
```typescript
@Post('upload')
@UseInterceptors(FileInterceptor('file'))
uploadFile(
  @UploadedFile(
    new ParseFilePipe({
      validators: [
        new MaxFileSizeValidator({
          maxSize: 5 * 1024 * 1024,
          message: 'File size must not exceed 5MB',
        }),
        new FileTypeValidator({
          fileType: 'image/*',
          message: 'Only image files are allowed',
        }),
      ],
    }),
  )
  file: Express.Multer.File,
) {
  return { file };
}
```

**Interview Tip**: Use `MaxFileSizeValidator` for file size (in bytes), `FileTypeValidator` for MIME type validation. Combine multiple validators in `ParseFilePipe`. Use regex in `fileType` for multiple types: `'image/(jpeg|png|gif)'`. Size in bytes: 1MB = 1024 * 1024.

</details>

## Pipes vs Other Features

### 37. What is the difference between Pipes and Guards?

<details>
<summary>Answer</summary>

**Pipes** transform/validate data; **Guards** authorize access.

**Comparison:**
| Feature | Pipes | Guards |
|---------|-------|--------|
| Purpose | Transform & validate data | Authorize access |
| Return | Transformed value | boolean |
| When | After Guards | Before Pipes |
| Use Case | Data validation, type conversion | Authentication, authorization |
| Throw | BadRequestException (400) | UnauthorizedException (401), ForbiddenException (403) |

**Pipes:**
```typescript
@Injectable()
export class ValidationPipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    // Validate and transform data
    if (!this.isValid(value)) {
      throw new BadRequestException('Invalid data');
    }
    return this.transformData(value);
  }
}

@Post()
create(@Body(ValidationPipe) dto: CreateUserDto) {
  // dto is validated and transformed
  return dto;
}
```

**Guards:**
```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    // Check if user is authenticated
    const request = context.switchToHttp().getRequest();
    return !!request.user;  // true = allow, false = deny
  }
}

@Post()
@UseGuards(AuthGuard)
create(@Body() dto: CreateUserDto) {
  // Only executes if AuthGuard returns true
  return dto;
}
```

**Execution Order:**
```
Request
  ↓
Middleware
  ↓
Guards ← Check authorization first
  ↓
Interceptors (before)
  ↓
Pipes ← Validate/transform data
  ↓
Handler
```

**When to Use:**
```typescript
// ✅ Use Guards for:
// - Authentication (is user logged in?)
// - Authorization (does user have permission?)
// - Role checking (is user admin?)

@Get()
@UseGuards(AuthGuard, RolesGuard)
@Roles('admin')
getData() {}

// ✅ Use Pipes for:
// - Data validation (is email valid?)
// - Type conversion (string to number)
// - Data transformation (trim, lowercase)

@Post()
create(@Body(ValidationPipe) dto: CreateUserDto) {}
```

**Real Example:**
```typescript
@Controller('posts')
export class PostsController {
  @Post()
  @UseGuards(JwtAuthGuard)  // 1. Check if authenticated
  create(
    @Body(ValidationPipe) dto: CreatePostDto,  // 2. Validate data
  ) {
    return this.postService.create(dto);
  }

  @Delete(':id')
  @UseGuards(JwtAuthGuard, RolesGuard)  // 1. Auth, then check role
  @Roles('admin')
  remove(
    @Param('id', ParseIntPipe) id: number,  // 2. Convert to number
  ) {
    return this.postService.remove(id);
  }
}
```

**Interview Tip**: Guards authorize (who can access?), Pipes validate data (is data correct?). Guards execute before Pipes. Guards return boolean, Pipes return transformed value. Use Guards for authentication/authorization, Pipes for validation/transformation.

</details>

### 38. When should you use Pipes vs Guards?

<details>
<summary>Answer</summary>

Use **Guards** for access control; use **Pipes** for data validation.

**Use Guards When:**
1. ✅ Check if user is authenticated
2. ✅ Check if user has required role/permission
3. ✅ Conditional route access based on business logic
4. ✅ Rate limiting
5. ✅ Feature flags

```typescript
@Get('admin')
@UseGuards(AuthGuard, RolesGuard)
@Roles('admin')
getAdminData() {
  // Guards ensure user is authenticated and has admin role
}
```

**Use Pipes When:**
1. ✅ Validate request data (DTOs)
2. ✅ Transform data types (string to number)
3. ✅ Sanitize input (trim, lowercase)
4. ✅ Parse and validate files
5. ✅ Load entities from database

```typescript
@Post()
create(
  @Body(ValidationPipe) dto: CreateUserDto,
  @Query('notify', ParseBoolPipe) notify: boolean,
) {
  // Pipes ensure dto is valid and notify is boolean
}
```

**Decision Tree:**
```
Question: What do I need to do?

├─ Check WHO can access this route?
│  └─ Use Guards (AuthGuard, RolesGuard)
│
└─ Validate/transform WHAT data is sent?
   └─ Use Pipes (ValidationPipe, ParseIntPipe)
```

**Examples:**

**Authentication/Authorization → Guards:**
```typescript
// Check if user is logged in
@Get('profile')
@UseGuards(JwtAuthGuard)
getProfile() {}

// Check if user has admin role
@Delete(':id')
@UseGuards(JwtAuthGuard, RolesGuard)
@Roles('admin')
delete() {}
```

**Data Validation → Pipes:**
```typescript
// Validate DTO
@Post()
create(@Body(ValidationPipe) dto: CreateUserDto) {}

// Convert types
@Get(':id')
findOne(@Param('id', ParseIntPipe) id: number) {}

// Validate file
@Post('upload')
upload(@UploadedFile(ParseFilePipe) file: File) {}
```

**Together:**
```typescript
@Controller('posts')
export class PostsController {
  @Post()
  @UseGuards(JwtAuthGuard)  // Guard: Check authentication
  create(
    @Body(ValidationPipe) dto: CreatePostDto,  // Pipe: Validate data
  ) {
    return this.postService.create(dto);
  }

  @Put(':id')
  @UseGuards(JwtAuthGuard, RolesGuard)  // Guards: Auth + roles
  @Roles('admin', 'editor')
  update(
    @Param('id', ParseIntPipe) id: number,  // Pipe: Convert to number
    @Body(ValidationPipe) dto: UpdatePostDto,  // Pipe: Validate DTO
  ) {
    return this.postService.update(id, dto);
  }
}
```

**Interview Tip**: Use Guards for **access control** (who can access?), Pipes for **data validation** (is data valid?). Guards check authentication/authorization, Pipes validate/transform request data. Both can be used together - Guards first, then Pipes.

</details>

### 39. What is the order of execution (Pipes come after Guards)?

<details>
<summary>Answer</summary>

Pipes execute **after Guards** and **before route handlers**.

**Complete Request Lifecycle:**
```
1. Middleware
2. Guards
3. Interceptors (before)
4. PIPES ← Execute here
5. Route Handler
6. Interceptors (after)
7. Exception Filters
```

**Visual Flow:**
```typescript
Client Request
    ↓
[Middleware] - Logging, CORS, authentication setup
    ↓
[Guards] - Authorization, role checking
    ↓
[Interceptors (before)] - Transform request
    ↓
[PIPES] - Validate and transform parameters ← HERE
    ↓
[Route Handler] - Business logic
    ↓
[Interceptors (after)] - Transform response
    ↓
[Exception Filters] - Handle errors
    ↓
Response to Client
```

**Example:**
```typescript
@Controller('posts')
@UseInterceptors(LoggingInterceptor)
export class PostsController {
  @Post()
  @UseGuards(AuthGuard, RolesGuard)
  create(
    @Body(ValidationPipe) dto: CreatePostDto,
    @Query('notify', ParseBoolPipe) notify: boolean,
  ) {
    console.log('Handler executed');
    return dto;
  }
}

// Execution order:
// 1. LoggingInterceptor (before)
// 2. AuthGuard checks authentication
// 3. RolesGuard checks roles
// 4. ValidationPipe validates dto ← PIPES
// 5. ParseBoolPipe converts notify
// 6. Handler executes
// 7. LoggingInterceptor (after)
```

**Why This Order?**
```typescript
// Makes sense because:

// 1. No point validating data if user isn't authorized
@Post()
@UseGuards(AuthGuard)  // Check auth first
create(@Body(ValidationPipe) dto: CreateUserDto) {
  // Don't waste resources validating if user can't access
}

// 2. Guards are cheaper than validation
@Delete(':id')
@UseGuards(AdminGuard)  // Fast authorization check
remove(@Param('id', ParseIntPipe) id: number) {
  // Expensive database validation only if authorized
}
```

**Multiple Guards and Pipes:**
```typescript
@Get(':userId/posts/:postId')
@UseGuards(AuthGuard, OwnershipGuard)  // Execute in order
findPost(
  @Param('userId', ParseIntPipe) userId: number,  // Then pipes
  @Param('postId', ParseIntPipe) postId: number,
) {
  return { userId, postId };
}

// Order:
// 1. AuthGuard
// 2. OwnershipGuard
// 3. ParseIntPipe (userId)
// 4. ParseIntPipe (postId)
// 5. Handler
```

**Fail Fast:**
```typescript
@Post()
@UseGuards(PremiumUserGuard)  // Reject non-premium users early
create(
  @Body(ValidationPipe) dto: ExpensiveValidationDto,  // Skip if not premium
) {
  // Don't waste resources on expensive validation
  // if user doesn't have premium access
}
```

**Interview Tip**: Execution order: Middleware → **Guards** → Interceptors → **Pipes** → Handler. Pipes come after Guards because authorization should happen before expensive validation. Guards return true/false for access, Pipes validate/transform data. Fail fast on authorization before validating data.

</details>
