# NestJS Decorators - Top Interview Questions

# NestJS Decorators - Top Interview Questions

## Decorator Fundamentals

### 1. What are Decorators in TypeScript and how does NestJS use them?

<details>
<summary>Answer</summary>

**Decorators** are special TypeScript features that add metadata and behavior to classes, methods, properties, and parameters. NestJS heavily uses decorators for dependency injection, routing, validation, and more.

**What are Decorators:**
- Functions that modify classes, methods, properties, or parameters
- Prefixed with `@` symbol
- Executed at runtime when the class is defined
- Can attach metadata or modify behavior

**NestJS Core Usage:**

```typescript
// 1. Class Decorators - Define components
@Controller('users')  // HTTP controller
@Injectable()         // Injectable provider
@Module({})          // Application module
export class UsersController {}

// 2. Method Decorators - Define routes/behavior
@Get()              // HTTP GET endpoint
@Post()             // HTTP POST endpoint
@UseGuards(AuthGuard)  // Apply guard
async findAll() {}

// 3. Parameter Decorators - Extract request data
@Get(':id')
findOne(
  @Param('id') id: string,      // URL parameter
  @Query('filter') filter: string,  // Query string
  @Body() dto: CreateUserDto,   // Request body
) {}

// 4. Property Decorators - Mark properties
export class CreateUserDto {
  @IsString()     // Validation
  @ApiProperty()  // Swagger docs
  name: string;
}
```

**How NestJS Uses Them:**

```typescript
// Complete example showing all decorator types
@Controller('users')  // Class decorator
export class UsersController {
  constructor(
    private readonly usersService: UsersService,  // DI via constructor
  ) {}

  @Get()  // Method decorator
  @UseGuards(AuthGuard)  // Method decorator
  @HttpCode(200)  // Method decorator
  findAll(
    @Query('page') page: number,  // Parameter decorator
    @Headers('authorization') token: string,  // Parameter decorator
  ) {
    return this.usersService.findAll(page);
  }

  @Post()  // Method decorator
  create(
    @Body() createUserDto: CreateUserDto,  // Parameter decorator
  ) {
    return this.usersService.create(createUserDto);
  }
}
```

**Behind the Scenes:**

```typescript
// TypeScript compiles decorators with metadata
// tsconfig.json must have:
{
  "emitDecoratorMetadata": true,
  "experimentalDecorators": true
}

// Decorators attach metadata that NestJS reads:
@Get('users')  // Attaches route path metadata
@HttpCode(201)  // Attaches status code metadata
create() {}

// NestJS reads this metadata to:
// 1. Register routes
// 2. Set up DI
// 3. Apply guards/interceptors
// 4. Validate data
```

**Real-World Example:**

```typescript
// DTOs with validation decorators
export class CreateUserDto {
  @IsString()
  @IsNotEmpty()
  @MinLength(3)
  @MaxLength(50)
  @ApiProperty({ description: 'User name' })
  name: string;

  @IsEmail()
  @ApiProperty({ description: 'User email' })
  email: string;

  @IsInt()
  @Min(18)
  @Max(120)
  @IsOptional()
  age?: number;
}

// Controller with multiple decorators
@Controller('users')
@UseInterceptors(LoggingInterceptor)
@ApiTags('users')
export class UsersController {
  @Post()
  @HttpCode(HttpStatus.CREATED)
  @UseGuards(AuthGuard, RolesGuard)
  @Roles('admin')
  @ApiOperation({ summary: 'Create user' })
  @ApiResponse({ status: 201, description: 'User created' })
  @ApiResponse({ status: 403, description: 'Forbidden' })
  create(@Body() dto: CreateUserDto) {
    return this.usersService.create(dto);
  }
}
```

**Benefits of Decorators:**
1. **Declarative** - Express intent clearly
2. **Reusable** - DRY principle
3. **Composable** - Stack multiple decorators
4. **Readable** - Clean, self-documenting code
5. **Metadata-driven** - NestJS configures automatically

**Interview Tip**: Decorators are TypeScript functions prefixed with `@` that add metadata and behavior. NestJS uses them extensively for routing (`@Controller`, `@Get`), DI (`@Injectable`), parameter extraction (`@Body`, `@Param`), and validation (`@IsString`). They make code declarative and enable NestJS's metadata-driven architecture.

</details>

### 2. What types of decorators exist (Class, Method, Parameter, Property)?

<details>
<summary>Answer</summary>

TypeScript supports **four types of decorators**, all used extensively in NestJS:

**1. Class Decorators** - Applied to classes

```typescript
// Define what the class represents
@Controller('users')    // HTTP controller
@Injectable()          // Injectable service
@Module({})           // Application module
@Global()             // Global module
@Catch(HttpException) // Exception filter
export class UsersController {}

// Real example
@Controller('auth')
@ApiTags('authentication')
@UseGuards(ThrottlerGuard)
export class AuthController {
  // ...
}
```

**2. Method Decorators** - Applied to class methods

```typescript
@Controller('users')
export class UsersController {
  // HTTP method decorators
  @Get()              // GET endpoint
  @Post()             // POST endpoint
  @Put(':id')         // PUT with param
  @Delete(':id')      // DELETE endpoint
  @Patch(':id')       // PATCH endpoint
  
  // Behavior decorators
  @UseGuards(AuthGuard)       // Apply guard
  @UseInterceptors(LoggingInterceptor)  // Apply interceptor
  @UsePipes(ValidationPipe)   // Apply pipe
  
  // Response decorators
  @HttpCode(201)      // Set status code
  @Header('X-Custom', 'value')  // Set header
  @Redirect('https://example.com', 301)  // Redirect
  
  // Metadata decorators
  @SetMetadata('roles', ['admin'])
  @Roles('admin', 'user')
  
  // Documentation decorators
  @ApiOperation({ summary: 'Get users' })
  @ApiResponse({ status: 200, type: [User] })
  
  findAll() {
    return [];
  }
}
```

**3. Parameter Decorators** - Applied to method parameters

```typescript
@Controller('users')
export class UsersController {
  @Get(':id')
  findOne(
    // Extract from URL path
    @Param('id') id: string,
    @Param() allParams: any,
    
    // Extract from query string
    @Query('filter') filter: string,
    @Query() allQuery: any,
    
    // Extract from request body
    @Body() dto: CreateUserDto,
    @Body('name') name: string,
    
    // Extract from headers
    @Headers('authorization') token: string,
    @Headers() allHeaders: any,
    
    // Request/Response objects
    @Req() request: Request,
    @Res() response: Response,
    
    // Session/IP
    @Session() session: any,
    @Ip() ip: string,
    
    // Custom decorators
    @User() currentUser: UserEntity,
  ) {
    return { id, filter };
  }
}
```

**4. Property Decorators** - Applied to class properties

```typescript
// DTOs with validation
export class CreateUserDto {
  @IsString()          // Validation
  @IsNotEmpty()        // Validation
  @MinLength(3)        // Validation
  @MaxLength(50)       // Validation
  @ApiProperty()       // Swagger docs
  @Transform(({ value }) => value.trim())  // Transformation
  name: string;

  @IsEmail()
  @ApiProperty({ example: 'user@example.com' })
  email: string;

  @IsOptional()
  @IsInt()
  @Min(18)
  age?: number;
}

// Entities with TypeORM
@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  @Index()
  name: string;

  @Column({ unique: true })
  email: string;

  @CreateDateColumn()
  createdAt: Date;
}

// Dependency injection
@Injectable()
export class UsersService {
  @InjectRepository(User)  // Property injection (not common)
  private repository: Repository<User>;
}
```

**Complete Example Using All Types:**

```typescript
// Class decorator
@Controller('products')
@ApiTags('products')
@UseInterceptors(CacheInterceptor)
export class ProductsController {
  constructor(
    private readonly productsService: ProductsService,
  ) {}

  // Method decorators
  @Get()
  @HttpCode(HttpStatus.OK)
  @UseGuards(AuthGuard)
  @ApiOperation({ summary: 'List all products' })
  @ApiResponse({ status: 200, type: [Product] })
  findAll(
    // Parameter decorators
    @Query('page') page: number,
    @Query('limit') limit: number,
    @Headers('authorization') token: string,
    @User() currentUser: UserEntity,
  ) {
    return this.productsService.findAll({ page, limit });
  }

  @Post()
  @HttpCode(HttpStatus.CREATED)
  @Roles('admin')
  create(
    @Body() createProductDto: CreateProductDto,
    @User() currentUser: UserEntity,
  ) {
    return this.productsService.create(createProductDto);
  }
}

// DTO with property decorators
export class CreateProductDto {
  @IsString()
  @IsNotEmpty()
  @ApiProperty()
  name: string;

  @IsNumber()
  @Min(0)
  @ApiProperty()
  price: number;

  @IsOptional()
  @IsString()
  description?: string;
}
```

**Comparison Table:**

| Type | Applied To | NestJS Examples | Purpose |
|------|-----------|-----------------|---------|
| Class | Classes | `@Controller()`, `@Injectable()`, `@Module()` | Define component type |
| Method | Methods | `@Get()`, `@Post()`, `@UseGuards()` | Define routes & behavior |
| Parameter | Parameters | `@Param()`, `@Body()`, `@Query()` | Extract request data |
| Property | Properties | `@IsString()`, `@Column()`, `@ApiProperty()` | Validate & document |

**Decorator Execution Order:**

```typescript
// Executed bottom-to-top, right-to-left
@First()
@Second()
class MyClass {
  @Third()
  @Fourth()
  method(@Fifth() @Sixth() param: string) {}
}

// Order: Sixth → Fifth → Fourth → Third → Second → First
```

**Interview Tip**: Four decorator types: **Class** (define components like `@Controller`), **Method** (define routes/behavior like `@Get`), **Parameter** (extract data like `@Body`), and **Property** (validate/document like `@IsString`). NestJS uses all four types to create a declarative, metadata-driven architecture.

</details>

### 3. Can you create custom decorators in NestJS?

<details>
<summary>Answer</summary>

Yes! NestJS provides utilities to create **custom decorators** for cleaner, reusable code.

**Custom Parameter Decorator:**

```typescript
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

// Extract current user from request
export const User = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return request.user;
  },
);

// Usage
@Controller('profile')
export class ProfileController {
  @Get()
  getProfile(@User() user: UserEntity) {
    return user;  // Directly get authenticated user
  }
}
```

**Custom Decorator with Data:**

```typescript
// Extract specific user property
export const User = createParamDecorator(
  (data: string, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const user = request.user;
    
    return data ? user?.[data] : user;
  },
);

// Usage
@Get()
getProfile(
  @User() user: UserEntity,        // Full user object
  @User('id') userId: string,      // Just user.id
  @User('email') email: string,    // Just user.email
) {
  return { user, userId, email };
}
```

**Custom Method Decorator (Roles):**

```typescript
import { SetMetadata } from '@nestjs/common';

// Define roles decorator
export const Roles = (...roles: string[]) => SetMetadata('roles', roles);

// Usage
@Controller('admin')
export class AdminController {
  @Get()
  @Roles('admin', 'superadmin')  // Set metadata
  findAll() {
    return 'Only admins can see this';
  }
}

// Guard reads metadata
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    const request = context.switchToHttp().getRequest();
    return roles.includes(request.user.role);
  }
}
```

**Composite Decorator (Multiple Combined):**

```typescript
import { applyDecorators, UseGuards, SetMetadata } from '@nestjs/common';

// Combine multiple decorators
export function Auth(...roles: string[]) {
  return applyDecorators(
    SetMetadata('roles', roles),
    UseGuards(AuthGuard, RolesGuard),
    ApiBearerAuth(),
    ApiUnauthorizedResponse({ description: 'Unauthorized' }),
  );
}

// Usage - One decorator instead of four!
@Controller('users')
export class UsersController {
  @Get()
  @Auth('admin', 'user')  // Applies all decorators
  findAll() {
    return 'Protected and documented!';
  }
}
```

**Custom Property Decorator:**

```typescript
import { Transform } from 'class-transformer';

// Trim string decorator
export function Trim() {
  return Transform(({ value }) => {
    return typeof value === 'string' ? value.trim() : value;
  });
}

// Usage in DTO
export class CreateUserDto {
  @IsString()
  @Trim()  // Automatically trim whitespace
  name: string;

  @IsEmail()
  @Trim()
  email: string;
}
```

**Real-World Examples:**

**1. Extract User ID:**
```typescript
export const UserId = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return request.user?.id;
  },
);

@Get('profile')
getProfile(@UserId() userId: string) {
  return this.usersService.findById(userId);
}
```

**2. Public Route Decorator:**
```typescript
export const IS_PUBLIC_KEY = 'isPublic';
export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);

// Usage
@Controller('auth')
export class AuthController {
  @Public()  // Skip auth guard
  @Post('login')
  login(@Body() credentials: LoginDto) {
    return this.authService.login(credentials);
  }
}

// Guard checks metadata
@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext) {
    const isPublic = this.reflector.get(IS_PUBLIC_KEY, context.getHandler());
    if (isPublic) return true;  // Skip auth
    
    // Check authentication
    const request = context.switchToHttp().getRequest();
    return !!request.user;
  }
}
```

**3. Cookie Decorator:**
```typescript
export const Cookies = createParamDecorator(
  (data: string, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return data ? request.cookies?.[data] : request.cookies;
  },
);

@Get()
findAll(
  @Cookies('sessionId') sessionId: string,
  @Cookies() allCookies: any,
) {
  return { sessionId, allCookies };
}
```

**4. IP Address Decorator:**
```typescript
export const RealIp = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return (
      request.headers['x-forwarded-for'] ||
      request.headers['x-real-ip'] ||
      request.connection.remoteAddress ||
      request.ip
    );
  },
);

@Post()
create(@RealIp() ip: string, @Body() dto: any) {
  console.log(`Request from IP: ${ip}`);
}
```

**5. Pagination Decorator:**
```typescript
export const Pagination = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const page = parseInt(request.query.page) || 1;
    const limit = parseInt(request.query.limit) || 10;
    
    return {
      page,
      limit,
      skip: (page - 1) * limit,
    };
  },
);

@Get()
findAll(@Pagination() pagination: { page: number; limit: number; skip: number }) {
  return this.usersService.findAll(pagination);
}
```

**6. Protected Admin Route:**
```typescript
export function AdminOnly() {
  return applyDecorators(
    UseGuards(AuthGuard, RolesGuard),
    Roles('admin'),
    ApiTags('admin'),
    ApiBearerAuth(),
    ApiForbiddenResponse({ description: 'Admin access required' }),
  );
}

@Controller('admin')
export class AdminController {
  @Get('users')
  @AdminOnly()  // All protections in one decorator
  getUsers() {
    return this.usersService.findAll();
  }
}
```

**Interview Tip**: Yes! Create custom decorators using `createParamDecorator()` for parameters, `SetMetadata()` for metadata, and `applyDecorators()` to combine multiple decorators. Common use cases: extracting user data, role-based access, combining auth/docs decorators, and custom parameter extraction.

</details>

## Route Decorators

### 4. What is the `@Controller()` decorator?

<details>
<summary>Answer</summary>

`@Controller()` is a **class decorator** that marks a class as a NestJS controller, which handles incoming HTTP requests and returns responses.

**Basic Usage:**

```typescript
import { Controller, Get } from '@nestjs/common';

@Controller()  // Basic controller
export class AppController {
  @Get()
  findAll() {
    return 'Hello World';
  }
}
// Routes: GET /
```

**With Route Prefix:**

```typescript
@Controller('users')  // Prefix all routes with /users
export class UsersController {
  @Get()          // GET /users
  findAll() {}

  @Get(':id')     // GET /users/:id
  findOne() {}

  @Post()         // POST /users
  create() {}
}
```

**With Version:**

```typescript
@Controller({ path: 'users', version: '1' })
export class UsersV1Controller {
  @Get()  // GET /v1/users
  findAll() {}
}

@Controller({ path: 'users', version: '2' })
export class UsersV2Controller {
  @Get()  // GET /v2/users
  findAll() {}
}
```

**With Host:**

```typescript
// Route only for specific subdomain
@Controller({ host: 'admin.example.com' })
export class AdminController {
  @Get()
  dashboard() {
    return 'Admin Dashboard';
  }
}

// Dynamic subdomain
@Controller({ host: ':tenant.example.com' })
export class TenantController {
  @Get()
  index(@HostParam('tenant') tenant: string) {
    return `Tenant: ${tenant}`;
  }
}
```

**Complete Controller Example:**

```typescript
@Controller('products')
export class ProductsController {
  constructor(
    private readonly productsService: ProductsService,
  ) {}

  // GET /products
  @Get()
  findAll(@Query('category') category?: string) {
    return this.productsService.findAll(category);
  }

  // GET /products/:id
  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.productsService.findOne(id);
  }

  // POST /products
  @Post()
  create(@Body() createProductDto: CreateProductDto) {
    return this.productsService.create(createProductDto);
  }

  // PUT /products/:id
  @Put(':id')
  update(
    @Param('id') id: string,
    @Body() updateProductDto: UpdateProductDto,
  ) {
    return this.productsService.update(id, updateProductDto);
  }

  // DELETE /products/:id
  @Delete(':id')
  remove(@Param('id') id: string) {
    return this.productsService.remove(id);
  }
}
```

**Register in Module:**

```typescript
@Module({
  controllers: [UsersController],  // Register controller
  providers: [UsersService],
})
export class UsersModule {}
```

**Interview Tip**: `@Controller(prefix)` marks a class as a controller that handles HTTP requests. The prefix becomes the base route for all endpoints in that controller. Register controllers in the module's `controllers` array.

</details>

### 5. What are the main HTTP Method decorators (`@Get()`, `@Post()`, `@Put()`, `@Delete()`, `@Patch()`)?

<details>
<summary>Answer</summary>

NestJS provides decorators for all standard HTTP methods to define route handlers.

**Main HTTP Method Decorators:**

```typescript
import { Controller, Get, Post, Put, Patch, Delete } from '@nestjs/common';

@Controller('users')
export class UsersController {
  // GET - Retrieve data
  @Get()              // GET /users
  findAll() {
    return this.usersService.findAll();
  }

  @Get(':id')         // GET /users/:id
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(id);
  }

  // POST - Create new resource
  @Post()             // POST /users
  create(@Body() dto: CreateUserDto) {
    return this.usersService.create(dto);
  }

  // PUT - Replace entire resource
  @Put(':id')         // PUT /users/:id
  replace(@Param('id') id: string, @Body() dto: UpdateUserDto) {
    return this.usersService.replace(id, dto);
  }

  // PATCH - Partial update
  @Patch(':id')       // PATCH /users/:id
  update(@Param('id') id: string, @Body() dto: Partial<UpdateUserDto>) {
    return this.usersService.update(id, dto);
  }

  // DELETE - Remove resource
  @Delete(':id')      // DELETE /users/:id
  remove(@Param('id') id: string) {
    return this.usersService.remove(id);
  }
}
```

**With Route Paths:**

```typescript
@Controller('products')
export class ProductsController {
  @Get('featured')          // GET /products/featured
  getFeatured() {}

  @Get('search')            // GET /products/search
  search(@Query() query) {}

  @Get(':id')               // GET /products/:id
  findOne(@Param('id') id: string) {}

  @Post(':id/reviews')      // POST /products/:id/reviews
  addReview(@Param('id') id: string, @Body() dto: ReviewDto) {}
}
```

**Other HTTP Methods:**

```typescript
import { Options, Head, All } from '@nestjs/common';

@Options('*')       // OPTIONS - CORS preflight
getOptions() {}

@Head(':id')        // HEAD - Headers only, no body
checkExists(@Param('id') id: string) {}

@All('*')          // ANY method - Catch-all
handleAll(@Req() req: Request) {
  return `Method: ${req.method}`;
}
```

**PUT vs PATCH:**

```typescript
// PUT - Replace entire resource (all fields required)
@Put(':id')
replace(@Param('id') id: string, @Body() dto: UserDto) {
  // Must provide ALL fields
  return this.usersService.replace(id, dto);
}
// PUT /users/1 { "name": "John", "email": "john@example.com", "age": 30 }

// PATCH - Update specific fields (partial)
@Patch(':id')
update(@Param('id') id: string, @Body() dto: Partial<UserDto>) {
  // Only update provided fields
  return this.usersService.update(id, dto);
}
// PATCH /users/1 { "name": "Jane" }  // Only updates name
```

**RESTful CRUD:**

```typescript
@Controller('articles')
export class ArticlesController {
  // CREATE
  @Post()
  @HttpCode(HttpStatus.CREATED)
  create(@Body() dto: CreateArticleDto) {
    return this.articlesService.create(dto);
  }

  // READ - List
  @Get()
  findAll(@Query() query: PaginationDto) {
    return this.articlesService.findAll(query);
  }

  // READ - Single
  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.articlesService.findOne(id);
  }

  // UPDATE - Full
  @Put(':id')
  replace(@Param('id') id: string, @Body() dto: ArticleDto) {
    return this.articlesService.replace(id, dto);
  }

  // UPDATE - Partial
  @Patch(':id')
  update(@Param('id') id: string, @Body() dto: Partial<ArticleDto>) {
    return this.articlesService.update(id, dto);
  }

  // DELETE
  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  remove(@Param('id') id: string) {
    return this.articlesService.remove(id);
  }
}
```

**Interview Tip**: Main HTTP decorators: `@Get()` (retrieve), `@Post()` (create), `@Put()` (replace), `@Patch()` (partial update), `@Delete()` (remove). Use PATCH for partial updates and PUT for full replacements. Each decorator maps to the corresponding HTTP verb.

</details>

### 6. How do you define route parameters in decorators?

<details>
<summary>Answer</summary>

Define route parameters using `:paramName` syntax in route decorators and extract them with `@Param()`.

**Basic Route Parameter:**

```typescript
@Controller('users')
export class UsersController {
  @Get(':id')  // :id is route parameter
  findOne(@Param('id') id: string) {
    return `User ${id}`;
  }
}
// GET /users/123 → id = '123'
```

**Multiple Parameters:**

```typescript
@Get(':userId/posts/:postId')
findPost(
  @Param('userId') userId: string,
  @Param('postId') postId: string,
) {
  return `User ${userId}, Post ${postId}`;
}
// GET /users/123/posts/456
// userId = '123', postId = '456'
```

**All Parameters:**

```typescript
@Get(':userId/posts/:postId')
findPost(@Param() params: any) {
  return params;  // { userId: '123', postId: '456' }
}
```

**Optional Parameters (Use Query Instead):**

```typescript
// ❌ Route params are NOT optional
@Get(':id/:subId')  // Both required

// ✅ Use query params for optional
@Get(':id')
findOne(
  @Param('id') id: string,
  @Query('includeDeleted') includeDeleted?: boolean,
) {}
// GET /users/123?includeDeleted=true
```

**Nested Resources:**

```typescript
@Controller('users')
export class UsersController {
  // GET /users/:userId/orders
  @Get(':userId/orders')
  getUserOrders(@Param('userId') userId: string) {
    return this.ordersService.findByUser(userId);
  }

  // GET /users/:userId/orders/:orderId
  @Get(':userId/orders/:orderId')
  getUserOrder(
    @Param('userId') userId: string,
    @Param('orderId') orderId: string,
  ) {
    return this.ordersService.findOne(userId, orderId);
  }

  // POST /users/:userId/orders
  @Post(':userId/orders')
  createOrder(
    @Param('userId') userId: string,
    @Body() dto: CreateOrderDto,
  ) {
    return this.ordersService.create(userId, dto);
  }
}
```

**With Validation Pipe:**

```typescript
import { ParseIntPipe, ParseUUIDPipe } from '@nestjs/common';

@Get(':id')
findOne(
  @Param('id', ParseIntPipe) id: number,  // Auto-convert to number
) {
  return this.usersService.findOne(id);
}

@Get(':uuid')
findByUuid(
  @Param('uuid', ParseUUIDPipe) uuid: string,  // Validate UUID format
) {
  return this.usersService.findByUuid(uuid);
}
```

**Custom Validation:**

```typescript
@Get(':id')
async findOne(
  @Param('id', new ParseIntPipe({ errorHttpStatusCode: HttpStatus.NOT_ACCEPTABLE }))
  id: number,
) {
  return this.usersService.findOne(id);
}
```

**Real-World Example:**

```typescript
@Controller('api/v1/organizations')
export class OrganizationsController {
  // GET /api/v1/organizations/:orgId
  @Get(':orgId')
  findOrg(@Param('orgId', ParseUUIDPipe) orgId: string) {
    return this.orgsService.findOne(orgId);
  }

  // GET /api/v1/organizations/:orgId/teams
  @Get(':orgId/teams')
  getTeams(@Param('orgId', ParseUUIDPipe) orgId: string) {
    return this.teamsService.findByOrg(orgId);
  }

  // GET /api/v1/organizations/:orgId/teams/:teamId
  @Get(':orgId/teams/:teamId')
  getTeam(
    @Param('orgId', ParseUUIDPipe) orgId: string,
    @Param('teamId', ParseUUIDPipe) teamId: string,
  ) {
    return this.teamsService.findOne(orgId, teamId);
  }

  // GET /api/v1/organizations/:orgId/teams/:teamId/members
  @Get(':orgId/teams/:teamId/members')
  getTeamMembers(
    @Param('orgId', ParseUUIDPipe) orgId: string,
    @Param('teamId', ParseUUIDPipe) teamId: string,
    @Query('role') role?: string,
  ) {
    return this.membersService.findByTeam(orgId, teamId, role);
  }
}
```

**Interview Tip**: Define route parameters with `:paramName` in decorators (`@Get(':id')`), extract with `@Param('id')`. Use validation pipes like `ParseIntPipe` or `ParseUUIDPipe` for automatic validation and transformation. Route parameters are required; use query parameters for optional values.

</details>

## Parameter Decorators

### 7. What is `@Param()` and when do you use it?

<details>
<summary>Answer</summary>

`@Param()` extracts **route parameters** from the URL path.

**Basic Usage:**
```typescript
@Get(':id')
findOne(@Param('id') id: string) {
  return `User ID: ${id}`;
}
// GET /users/123 → id = '123'
```

**Multiple Parameters:**
```typescript
@Get(':userId/posts/:postId')
findPost(
  @Param('userId') userId: string,
  @Param('postId') postId: string,
) {
  return { userId, postId };
}
// GET /users/123/posts/456
// userId = '123', postId = '456'
```

**All Parameters:**
```typescript
@Get(':userId/posts/:postId')
findPost(@Param() params: { userId: string; postId: string }) {
  return params;  // { userId: '123', postId: '456' }
}
```

**With Validation:**
```typescript
@Get(':id')
findOne(@Param('id', ParseIntPipe) id: number) {
  return this.usersService.findOne(id);  // id is number
}

@Get(':uuid')
findByUuid(@Param('uuid', ParseUUIDPipe) uuid: string) {
  return this.usersService.findByUuid(uuid);  // Validated UUID
}
```

**Interview Tip**: `@Param('paramName')` extracts route parameters from URL paths (`:id`). Use validation pipes to automatically convert and validate types. Route parameters are always part of the URL path.

</details>

### 8. What is `@Query()` and how is it different from `@Param()`?

<details>
<summary>Answer</summary>

`@Query()` extracts **query string parameters**, while `@Param()` extracts **route parameters**.

**@Query() - Query Strings:**
```typescript
@Get()
findAll(
  @Query('page') page: number,
  @Query('limit') limit: number,
) {
  return { page, limit };
}
// GET /users?page=1&limit=10
// page = 1, limit = 10
```

**@Param() - URL Path:**
```typescript
@Get(':id')
findOne(@Param('id') id: string) {
  return { id };
}
// GET /users/123
// id = '123'
```

**Comparison:**
| Feature | @Param() | @Query() |
|---------|----------|----------|
| Source | URL path | Query string |
| Required | Yes | No (optional) |
| Example URL | `/users/123` | `/users?page=1` |
| Syntax | `:paramName` | `?key=value` |

**Combined Usage:**
```typescript
@Get(':category')
findByCategory(
  @Param('category') category: string,  // From path
  @Query('page') page: number,          // From query
  @Query('limit') limit: number,        // From query
  @Query('sort') sort?: string,         // Optional query
) {
  return { category, page, limit, sort };
}
// GET /products/electronics?page=1&limit=20&sort=price
```

**All Query Params:**
```typescript
@Get()
findAll(@Query() query: any) {
  return query;  // { page: '1', limit: '10', search: 'test' }
}
```

**With Validation:**
```typescript
@Get()
findAll(
  @Query('page', new ParseIntPipe({ optional: true })) page: number = 1,
  @Query('limit', ParseIntPipe) limit: number,
) {}
```

**Interview Tip**: `@Param()` extracts from URL path (required), `@Query()` extracts from query string (optional). Use `@Param()` for resource identifiers, `@Query()` for filters, pagination, and sorting.

</details>

### 9. What is `@Body()` used for?

<details>
<summary>Answer</summary>

`@Body()` extracts data from the **HTTP request body** (typically JSON).

**Basic Usage:**
```typescript
@Post()
create(@Body() createUserDto: CreateUserDto) {
  return this.usersService.create(createUserDto);
}
// POST /users
// Body: { "name": "John", "email": "john@example.com" }
```

**Extract Specific Field:**
```typescript
@Post()
create(
  @Body('name') name: string,
  @Body('email') email: string,
) {
  return { name, email };
}
```

**With DTO and Validation:**
```typescript
export class CreateUserDto {
  @IsString()
  @IsNotEmpty()
  name: string;

  @IsEmail()
  email: string;

  @IsInt()
  @Min(18)
  age: number;
}

@Post()
@UsePipes(new ValidationPipe())
create(@Body() createUserDto: CreateUserDto) {
  // DTO automatically validated
  return this.usersService.create(createUserDto);
}
```

**Multiple Body Sources:**
```typescript
@Post(':id')
create(
  @Body() dto: CreateUserDto,        // Full body
  @Param('id') id: string,           // URL param
  @Query('notify') notify: boolean,  // Query param
) {
  return this.usersService.create(id, dto, notify);
}
// POST /users/123?notify=true
// Body: { "name": "John" }
```

**Nested Properties:**
```typescript
@Post()
create(
  @Body('user.name') userName: string,
  @Body('user.email') userEmail: string,
) {}
// Body: { "user": { "name": "John", "email": "john@example.com" } }
```

**Interview Tip**: `@Body()` extracts request body data (JSON). Use with DTOs and validation pipes for type-safe, validated data. Extract specific fields with `@Body('fieldName')` or entire body with `@Body()`.

</details>

### 10. What is `@Headers()` decorator?

<details>
<summary>Answer</summary>

`@Headers()` extracts HTTP headers from the request.

**Basic Usage:**
```typescript
@Get()
findAll(@Headers('authorization') auth: string) {
  return `Token: ${auth}`;
}
// Header: Authorization: Bearer abc123
```

**All Headers:**
```typescript
@Get()
findAll(@Headers() headers: any) {
  return headers;  // All request headers
}
```

**Multiple Headers:**
```typescript
@Get()
findAll(
  @Headers('authorization') auth: string,
  @Headers('user-agent') userAgent: string,
  @Headers('content-type') contentType: string,
) {
  return { auth, userAgent, contentType };
}
```

**Case Insensitive:**
```typescript
@Get()
findAll(
  @Headers('Authorization') auth1: string,  // Works
  @Headers('authorization') auth2: string,  // Also works
  @Headers('AUTHORIZATION') auth3: string,  // Also works
) {}
```

**Real-World Examples:**
```typescript
// API Key Authentication
@Get()
findAll(@Headers('x-api-key') apiKey: string) {
  if (!this.authService.validateApiKey(apiKey)) {
    throw new UnauthorizedException();
  }
  return this.usersService.findAll();
}

// User Agent Tracking
@Post()
create(
  @Body() dto: CreateUserDto,
  @Headers('user-agent') userAgent: string,
) {
  return this.usersService.create(dto, { userAgent });
}

// Custom Tenant Header
@Get()
findAll(@Headers('x-tenant-id') tenantId: string) {
  return this.usersService.findByTenant(tenantId);
}
```

**Interview Tip**: `@Headers('headerName')` extracts specific HTTP headers. Header names are case-insensitive. Use for authentication tokens, API keys, tenant IDs, and tracking information.

</details>

### 11. What are `@Req()` and `@Res()` decorators?

<details>
<summary>Answer</summary>

`@Req()` and `@Res()` provide access to the underlying **Request** and **Response** objects.

**@Req() - Request Object:**
```typescript
@Get()
findAll(@Req() request: Request) {
  console.log(request.method);   // GET
  console.log(request.url);      // /users
  console.log(request.headers);  // All headers
  console.log(request.ip);       // Client IP
  return request.user;           // User from auth middleware
}
```

**@Res() - Response Object (⚠️ Use Carefully):**
```typescript
@Get()
findAll(@Res() response: Response) {
  response.status(200).json({ message: 'Hello' });
  // Must manually send response!
}
```

**⚠️ Problem with @Res():**
```typescript
// ❌ Breaks NestJS features
@Get()
findAll(@Res() res: Response) {
  res.json({ data: [] });
  // Interceptors don't run!
  // Exception filters don't work!
  // Must handle everything manually!
}
```

**✅ Solution - Use passthrough:**
```typescript
@Get()
findAll(@Res({ passthrough: true }) res: Response) {
  res.set('X-Custom-Header', 'value');
  return { data: [] };  // NestJS handles response
  // Interceptors and filters still work!
}
```

**When to Use @Req():**
```typescript
// Access authenticated user
@Get('profile')
getProfile(@Req() req: Request) {
  return req.user;  // Set by AuthGuard
}

// Check IP address
@Post()
create(@Req() req: Request, @Body() dto: any) {
  console.log(`Request from: ${req.ip}`);
  return this.service.create(dto);
}

// Access custom properties
@Get()
findAll(@Req() req: Request & { userId: string }) {
  const userId = req.userId;  // Custom middleware added this
  return this.service.findByUser(userId);
}
```

**When to Use @Res() with passthrough:**
```typescript
// Set custom headers
@Get()
findAll(@Res({ passthrough: true }) res: Response) {
  res.set('X-Total-Count', '100');
  return this.service.findAll();
}

// Set cookies
@Post('login')
login(
  @Body() credentials: LoginDto,
  @Res({ passthrough: true }) res: Response,
) {
  const token = this.authService.login(credentials);
  res.cookie('auth-token', token, { httpOnly: true });
  return { success: true };
}
```

**Interview Tip**: Avoid `@Res()` unless necessary (breaks NestJS features). Use `@Req()` to access request data like authenticated user or IP. If you must use `@Res()`, add `{ passthrough: true }` to keep interceptors and filters working.

</details>

### 12. How do you extract specific parameters (e.g., `@Param('id')`)?

<details>
<summary>Answer</summary>

Pass the parameter name as a string to extract specific values from `@Param()`, `@Query()`, `@Body()`, or `@Headers()`.

**Extract Specific Values:**
```typescript
// Route parameters
@Get(':id')
findOne(@Param('id') id: string) {}

// Query parameters
@Get()
findAll(@Query('page') page: number) {}

// Body fields
@Post()
create(@Body('name') name: string) {}

// Headers
@Get()
findAll(@Headers('authorization') token: string) {}
```

**Multiple Specific Values:**
```typescript
@Get(':userId/posts/:postId')
findPost(
  @Param('userId') userId: string,
  @Param('postId') postId: string,
  @Query('include') include: string,
  @Headers('authorization') auth: string,
) {
  return { userId, postId, include, auth };
}
```

**With Type Conversion:**
```typescript
@Get(':id')
findOne(
  @Param('id', ParseIntPipe) id: number,  // Convert to number
  @Query('limit', ParseIntPipe) limit: number,
) {
  return { id, limit };  // Both are numbers
}
```

**Nested Properties:**
```typescript
@Post()
create(
  @Body('user.name') userName: string,
  @Body('user.email') userEmail: string,
  @Body('settings.theme') theme: string,
) {}
// Body: {
//   "user": { "name": "John", "email": "john@example.com" },
//   "settings": { "theme": "dark" }
// }
```

**All vs Specific:**
```typescript
// Get all parameters
@Get(':id')
findOne(@Param() params: any) {
  return params;  // { id: '123' }
}

// Get specific parameter
@Get(':id')
findOne(@Param('id') id: string) {
  return id;  // '123'
}

// Get all query params
@Get()
findAll(@Query() query: any) {
  return query;  // { page: '1', limit: '10' }
}

// Get specific query param
@Get()
findAll(@Query('page') page: string) {
  return page;  // '1'
}
```

**Interview Tip**: Pass parameter name as string to extract specific values: `@Param('id')`, `@Query('page')`, `@Body('name')`, `@Headers('auth')`. Without a name, the decorator returns all values as an object.

</details>

## Response Decorators

### 13. What is `@HttpCode()` decorator?

<details>
<summary>Answer</summary>

`@HttpCode()` sets a **custom HTTP status code** for the response.

**Basic Usage:**
```typescript
@Post()
@HttpCode(HttpStatus.CREATED)  // 201
create(@Body() dto: CreateUserDto) {
  return this.usersService.create(dto);
}
```

**Common Status Codes:**
```typescript
import { HttpCode, HttpStatus } from '@nestjs/common';

// 200 OK
@Post('login')
@HttpCode(HttpStatus.OK)
login() {}

// 201 Created (default for POST)
@Post()
@HttpCode(HttpStatus.CREATED)
create() {}

// 204 No Content
@Delete(':id')
@HttpCode(HttpStatus.NO_CONTENT)
remove() {}

// 202 Accepted
@Post('async-task')
@HttpCode(HttpStatus.ACCEPTED)
queueTask() {}
```

**Override Defaults:**
```typescript
// POST defaults to 201, override to 200
@Post('search')
@HttpCode(HttpStatus.OK)  // 200 instead of 201
search() {}

// DELETE defaults to 200, override to 204
@Delete(':id')
@HttpCode(HttpStatus.NO_CONTENT)  // 204 instead of 200
remove() {}
```

**Interview Tip**: `@HttpCode(status)` sets custom HTTP status codes. Use 201 for creation, 204 for no content, 200 for non-creating POST endpoints, and 202 for async operations.

</details>

### 14. What is `@Header()` decorator?

<details>
<summary>Answer</summary>

`@Header()` sets **custom HTTP response headers**.

**Basic Usage:**
```typescript
@Get()
@Header('X-Custom-Header', 'value')
findAll() {
  return [];
}
```

**Multiple Headers:**
```typescript
@Get()
@Header('Cache-Control', 'max-age=3600')
@Header('X-API-Version', 'v1')
findAll() {}
```

**Common Use Cases:**
```typescript
// Caching
@Get()
@Header('Cache-Control', 'public, max-age=3600')
findAll() {}

// CORS
@Get()
@Header('Access-Control-Allow-Origin', '*')
findAll() {}

// Content Type
@Get('download')
@Header('Content-Type', 'application/pdf')
@Header('Content-Disposition', 'attachment; filename=report.pdf')
download() {}
```

**Dynamic Headers (Use @Res()):**
```typescript
@Get()
findAll(@Res({ passthrough: true }) res: Response) {
  res.set('X-Total-Count', '100');
  res.set('X-Page', '1');
  return [];
}
```

**Interview Tip**: `@Header(key, value)` sets static response headers. For dynamic headers, use `@Res({ passthrough: true })` with `res.set()`.

</details>

### 15. What is `@Redirect()` decorator?

<details>
<summary>Answer</summary>

`@Redirect()` redirects requests to a different URL.

**Basic Usage:**
```typescript
@Get('old-route')
@Redirect('https://example.com/new-route', 301)
redirect() {}
```

**With Status Code:**
```typescript
@Get('docs')
@Redirect('https://docs.example.com', 301)  // Permanent
redirectToDocs() {}

@Get('temp')
@Redirect('/new-temp', 302)  // Temporary
redirectTemp() {}
```

**Dynamic Redirect:**
```typescript
@Get('redirect')
@Redirect()  // No URL specified
redirect(@Query('url') url: string) {
  return {
    url: url || 'https://default.com',
    statusCode: 302,
  };
}
```

**Conditional Redirect:**
```typescript
@Get('users/:id')
@Redirect()
findUser(@Param('id') id: string) {
  if (id === 'me') {
    return { url: '/profile', statusCode: 302 };
  }
  // No redirect object = no redirect
}
```

**Interview Tip**: `@Redirect(url, statusCode)` redirects to another URL. Return `{ url, statusCode }` from handler for dynamic redirects. Use 301 for permanent, 302 for temporary.

</details>

## Module & Provider Decorators

### 16. What is `@Module()` decorator and its properties?

<details>
<summary>Answer</summary>

`@Module()` defines a **module** - the basic building block of NestJS apps.

**Basic Module:**
```typescript
@Module({
  imports: [],      // Other modules
  controllers: [],  // HTTP controllers
  providers: [],    // Services, repositories
  exports: [],      // Make providers available to other modules
})
export class UsersModule {}
```

**Complete Example:**
```typescript
@Module({
  imports: [
    TypeOrmModule.forFeature([User]),  // Import other modules
    ConfigModule,
  ],
  controllers: [UsersController],      // Register controllers
  providers: [
    UsersService,                      // Register services
    UsersRepository,
    {
      provide: 'USER_REPOSITORY',      // Custom provider
      useFactory: () => new UsersRepository(),
    },
  ],
  exports: [UsersService],             // Export for other modules
})
export class UsersModule {}
```

**Properties:**
- **imports** - Import other modules
- **controllers** - Register HTTP controllers
- **providers** - Register services, guards, interceptors
- **exports** - Make providers available to importing modules

**Interview Tip**: `@Module()` defines modules with `imports` (other modules), `controllers` (HTTP handlers), `providers` (services), and `exports` (share providers). Modules organize code into cohesive units.

</details>

### 17. What is `@Global()` decorator?

<details>
<summary>Answer</summary>

`@Global()` makes a module **globally available** - no need to import it everywhere.

**Usage:**
```typescript
@Global()
@Module({
  providers: [LoggerService, ConfigService],
  exports: [LoggerService, ConfigService],
})
export class CoreModule {}
```

**Without @Global:**
```typescript
// Must import CoreModule everywhere
@Module({
  imports: [CoreModule],  // Required
  controllers: [UsersController],
})
export class UsersModule {}
```

**With @Global:**
```typescript
// No need to import CoreModule
@Module({
  // imports: [CoreModule],  // Not needed!
  controllers: [UsersController],
})
export class UsersModule {}

// Can still inject global services
@Controller()
export class UsersController {
  constructor(
    private logger: LoggerService,  // From global module
    private config: ConfigService,  // From global module
  ) {}
}
```

**Best Practice:**
```typescript
// Use for truly global services
@Global()
@Module({
  providers: [
    LoggerService,    // Used everywhere
    ConfigService,    // Used everywhere
    DatabaseService,  // Used everywhere
  ],
  exports: [LoggerService, ConfigService, DatabaseService],
})
export class CoreModule {}
```

**Interview Tip**: `@Global()` makes a module available everywhere without importing. Use sparingly for truly global services like logging, configuration, and database connections. Don't overuse - most modules should be explicitly imported.

</details>

### 18. What is `@Injectable()` decorator?

<details>
<summary>Answer</summary>

`@Injectable()` marks a class as a **provider** that can be injected via dependency injection.

**Basic Usage:**
```typescript
@Injectable()
export class UsersService {
  findAll() {
    return [];
  }
}
```

**With Dependencies:**
```typescript
@Injectable()
export class UsersService {
  constructor(
    private repository: UsersRepository,
    private emailService: EmailService,
  ) {}
}
```

**With Scope:**
```typescript
@Injectable({ scope: Scope.REQUEST })  // New instance per request
export class RequestService {}

@Injectable({ scope: Scope.TRANSIENT })  // New instance per injection
export class TransientService {}
```

**What Needs @Injectable:**
- ✅ Services
- ✅ Repositories
- ✅ Guards, Interceptors, Pipes, Filters
- ❌ Controllers (use `@Controller`)
- ❌ DTOs (plain classes)

**Interview Tip**: `@Injectable()` marks classes as providers for dependency injection. Use on services, repositories, guards, interceptors, pipes, and filters. Register in module's `providers` array.

</details>

### 19. What is `@Inject()` and when do you need it?

<details>
<summary>Answer</summary>

`@Inject()` explicitly specifies which **token** to inject when using custom providers.

**When You Don't Need It:**
```typescript
// Class token - no @Inject needed
@Injectable()
export class UsersService {
  constructor(
    private repository: UsersRepository,  // Class token
  ) {}
}
```

**When You Need It:**
```typescript
// String token - need @Inject
@Injectable()
export class UsersService {
  constructor(
    @Inject('DATABASE_CONNECTION') private connection: Connection,
    @Inject('CONFIG') private config: any,
  ) {}
}

// Register with string token
@Module({
  providers: [
    {
      provide: 'DATABASE_CONNECTION',
      useValue: createConnection(),
    },
  ],
})
```

**With useFactory:**
```typescript
@Module({
  providers: [
    {
      provide: 'CACHE_MANAGER',
      useFactory: () => createCacheManager(),
    },
  ],
})

@Injectable()
export class UsersService {
  constructor(
    @Inject('CACHE_MANAGER') private cache: CacheManager,
  ) {}
}
```

**Interview Tip**: `@Inject(token)` is needed for custom provider tokens (strings, symbols). Not needed for class tokens. Use with `useValue`, `useFactory`, or `useClass` providers.

</details>

### 20. What is `@Optional()` decorator?

<details>
<summary>Answer</summary>

`@Optional()` marks a dependency as **optional** - won't throw if not found.

**Usage:**
```typescript
@Injectable()
export class UsersService {
  constructor(
    private repository: UsersRepository,           // Required
    @Optional() private cache?: CacheService,      // Optional
    @Optional() private logger?: LoggerService,    // Optional
  ) {}

  findAll() {
    // Check if optional dependency exists
    if (this.cache) {
      return this.cache.get('users');
    }
    return this.repository.findAll();
  }
}
```

**With @Inject:**
```typescript
@Injectable()
export class UsersService {
  constructor(
    @Optional()
    @Inject('CACHE_MANAGER')
    private cache?: CacheManager,
  ) {}
}
```

**Without @Optional (Error):**
```typescript
@Injectable()
export class UsersService {
  constructor(
    private cache: CacheService,  // If not provided → Error!
  ) {}
}
```

**Interview Tip**: `@Optional()` prevents errors when a dependency isn't provided. Use for truly optional dependencies like caching or logging. Check if the dependency exists before using it.

</details>

## Custom Decorators

### 21. How do you create a custom parameter decorator?

<details>
<summary>Answer</summary>

Use `createParamDecorator()` to create custom parameter decorators.

**Basic Custom Decorator:**
```typescript
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const User = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return request.user;
  },
);

// Usage
@Get('profile')
getProfile(@User() user: UserEntity) {
  return user;
}
```

**With Data Parameter:**
```typescript
export const User = createParamDecorator(
  (data: string, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const user = request.user;
    
    return data ? user?.[data] : user;
  },
);

// Usage
@Get()
findAll(
  @User() user: UserEntity,        // Full user
  @User('id') userId: string,      // Just user.id
  @User('email') email: string,    // Just user.email
) {}
```

**Interview Tip**: Use `createParamDecorator((data, ctx) => {})` to create custom parameter decorators. Access request via `ctx.switchToHttp().getRequest()`. Return the value to inject.

</details>

### 22. What is `createParamDecorator()` function?

<details>
<summary>Answer</summary>

`createParamDecorator()` is the factory function for creating custom parameter decorators.

**Signature:**
```typescript
createParamDecorator(
  (data: any, ctx: ExecutionContext) => any
)
```

**Parameters:**
- **data** - Value passed to decorator (`@User('id')` → data = 'id')
- **ctx** - ExecutionContext for accessing request/response

**Examples:**

**Extract User:**
```typescript
export const User = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return request.user;
  },
);
```

**Extract IP:**
```typescript
export const RealIp = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return request.headers['x-forwarded-for'] || request.ip;
  },
);
```

**Extract Cookies:**
```typescript
export const Cookies = createParamDecorator(
  (data: string, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return data ? request.cookies?.[data] : request.cookies;
  },
);
```

**Interview Tip**: `createParamDecorator()` creates custom parameter decorators. First parameter is optional data, second is `ExecutionContext` for accessing request. Return the value to inject into the handler.

</details>

### 23. What is `applyDecorators()` used for?

<details>
<summary>Answer</summary>

`applyDecorators()` **combines multiple decorators** into one reusable decorator.

**Basic Usage:**
```typescript
import { applyDecorators, UseGuards, SetMetadata } from '@nestjs/common';

export function Auth(...roles: string[]) {
  return applyDecorators(
    SetMetadata('roles', roles),
    UseGuards(AuthGuard, RolesGuard),
    ApiBearerAuth(),
    ApiUnauthorizedResponse({ description: 'Unauthorized' }),
  );
}

// Usage - One decorator instead of four!
@Get()
@Auth('admin')
findAll() {}
```

**Real-World Example:**
```typescript
export function AdminOnly() {
  return applyDecorators(
    UseGuards(AuthGuard, RolesGuard, ThrottleGuard),
    Roles('admin'),
    ApiTags('admin'),
    ApiBearerAuth(),
    ApiUnauthorizedResponse({ description: 'Unauthorized' }),
    ApiForbiddenResponse({ description: 'Admin only' }),
    ApiTooManyRequestsResponse({ description: 'Rate limit exceeded' }),
  );
}

@Controller('admin')
export class AdminController {
  @Get('users')
  @AdminOnly()  // Applies all decorators above
  getUsers() {}
}
```

**Interview Tip**: `applyDecorators()` combines multiple decorators into one. Use for common decorator combinations like authentication + authorization + documentation.

</details>

### 24. How do you access `ExecutionContext` in custom decorators?

<details>
<summary>Answer</summary>

`ExecutionContext` is passed as the second parameter to `createParamDecorator()`.

**Access Request:**
```typescript
export const User = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return request.user;
  },
);
```

**ExecutionContext Methods:**
```typescript
export const CustomDecorator = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    // HTTP context
    const request = ctx.switchToHttp().getRequest();
    const response = ctx.switchToHttp().getResponse();
    
    // Request properties
    const user = request.user;
    const headers = request.headers;
    const body = request.body;
    const params = request.params;
    const query = request.query;
    
    // Metadata
    const handler = ctx.getHandler();
    const controller = ctx.getClass();
    
    return /* extracted value */;
  },
);
```

**Interview Tip**: `ExecutionContext` is the second parameter in `createParamDecorator()`. Use `ctx.switchToHttp().getRequest()` to access the request object and extract data.

</details>

### 25. How do you create a decorator to extract the current user?

<details>
<summary>Answer</summary>

Create a decorator using `createParamDecorator()` that extracts user from `request.user`.

**Complete Example:**
```typescript
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const CurrentUser = createParamDecorator(
  (data: string | undefined, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const user = request.user;
    
    // If data provided, return specific property
    return data ? user?.[data] : user;
  },
);

// Usage
@Controller('profile')
export class ProfileController {
  @Get()
  @UseGuards(AuthGuard)  // Sets request.user
  getProfile(@CurrentUser() user: User) {
    return user;
  }

  @Get('email')
  @UseGuards(AuthGuard)
  getEmail(@CurrentUser('email') email: string) {
    return { email };
  }

  @Patch()
  @UseGuards(AuthGuard)
  update(
    @CurrentUser('id') userId: string,
    @Body() updateDto: UpdateProfileDto,
  ) {
    return this.profileService.update(userId, updateDto);
  }
}
```

**Interview Tip**: Create user decorator with `createParamDecorator()` that accesses `request.user`. Support optional data parameter to extract specific user properties like `@User('id')` or `@User('email')`.

</details>

## Metadata Decorators

### 26. What is `@SetMetadata()` decorator?

<details>
<summary>Answer</summary>

`@SetMetadata()` attaches **custom metadata** to routes that can be read by guards, interceptors, or filters.

**Basic Usage:**
```typescript
import { SetMetadata } from '@nestjs/common';

@Get()
@SetMetadata('roles', ['admin', 'user'])
findAll() {}
```

**Custom Decorator:**
```typescript
export const Roles = (...roles: string[]) => SetMetadata('roles', roles);

@Get()
@Roles('admin', 'superadmin')
findAll() {}
```

**Read in Guard:**
```typescript
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    const request = context.switchToHttp().getRequest();
    return roles.includes(request.user.role);
  }
}
```

**Interview Tip**: `@SetMetadata(key, value)` attaches metadata to routes. Create custom decorators for better readability. Read metadata in guards/interceptors using `Reflector`.

</details>

### 27. What is the `Reflector` class and how do you use it?

<details>
<summary>Answer</summary>

`Reflector` reads **metadata** attached to routes by decorators like `@SetMetadata()`.

**Usage in Guard:**
```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}  // Inject Reflector

  canActivate(context: ExecutionContext): boolean {
    // Read metadata from handler
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    
    if (!roles) {
      return true;  // No roles required
    }
    
    const request = context.switchToHttp().getRequest();
    return roles.includes(request.user?.role);
  }
}
```

**Read from Class or Handler:**
```typescript
// Read from handler (method)
const handlerRoles = this.reflector.get('roles', context.getHandler());

// Read from class (controller)
const classRoles = this.reflector.get('roles', context.getClass());

// Read from both (handler takes precedence)
const roles = this.reflector.getAllAndOverride('roles', [
  context.getHandler(),
  context.getClass(),
]);
```

**Interview Tip**: `Reflector` reads metadata set by `@SetMetadata()`. Inject it in guards/interceptors and use `reflector.get(key, target)` to read metadata from handlers or classes.

</details>

### 28. How do you retrieve metadata in Guards or Interceptors?

<details>
<summary>Answer</summary>

Use `Reflector` to retrieve metadata in guards and interceptors.

**In Guard:**
```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // Read 'isPublic' metadata
    const isPublic = this.reflector.get<boolean>(
      'isPublic',
      context.getHandler(),
    );
    
    if (isPublic) {
      return true;  // Skip auth for public routes
    }
    
    // Check authentication
    const request = context.switchToHttp().getRequest();
    return !!request.user;
  }
}
```

**In Interceptor:**
```typescript
@Injectable()
export class CacheInterceptor implements NestInterceptor {
  constructor(private reflector: Reflector) {}

  intercept(context: ExecutionContext, next: CallHandler) {
    // Read cache TTL from metadata
    const ttl = this.reflector.get<number>('cacheTTL', context.getHandler());
    
    if (!ttl) {
      return next.handle();  // No caching
    }
    
    // Apply caching with TTL
    return next.handle().pipe(/* caching logic */);
  }
}
```

**Interview Tip**: Inject `Reflector` and use `reflector.get(key, target)` to read metadata. Common in guards for authorization and interceptors for caching/logging configuration.

</details>

## Validation Decorators

### 29. What validation decorators are commonly used from `class-validator`?

<details>
<summary>Answer</summary>

`class-validator` provides decorators for DTO validation.

**Common Validators:**
```typescript
import {
  IsString, IsNumber, IsEmail, IsBoolean, IsDate,
  IsNotEmpty, IsOptional, Min, Max, MinLength, MaxLength,
  IsInt, IsPositive, IsArray, ValidateNested,
} from 'class-validator';

export class CreateUserDto {
  @IsString()
  @IsNotEmpty()
  @MinLength(3)
  @MaxLength(50)
  name: string;

  @IsEmail()
  email: string;

  @IsInt()
  @Min(18)
  @Max(120)
  age: number;

  @IsBoolean()
  @IsOptional()
  isActive?: boolean;

  @IsArray()
  @IsString({ each: true })
  tags: string[];
}
```

**Interview Tip**: Common validators: `@IsString()`, `@IsNumber()`, `@IsEmail()`, `@IsNotEmpty()`, `@Min()`, `@Max()`, `@MinLength()`, `@IsOptional()`, `@IsArray()`. Use with `ValidationPipe` for automatic validation.

</details>

### 30. What is `@IsString()`, `@IsNumber()`, `@IsEmail()`?

<details>
<summary>Answer</summary>

Type validation decorators from `class-validator`.

**Usage:**
```typescript
export class CreateUserDto {
  @IsString()
  name: string;

  @IsNumber()
  age: number;

  @IsEmail()
  email: string;

  @IsInt()
  score: number;

  @IsBoolean()
  isActive: boolean;
}
```

**Interview Tip**: `@IsString()` validates string type, `@IsNumber()` validates number, `@IsEmail()` validates email format. Use for type checking in DTOs.

</details>

### 31. What is `@ValidateNested()` used for?

<details>
<summary>Answer</summary>

`@ValidateNested()` validates nested objects in DTOs.

**Usage:**
```typescript
import { ValidateNested } from 'class-validator';
import { Type } from 'class-transformer';

class AddressDto {
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

  @ValidateNested({ each: true })
  @Type(() => AddressDto)
  @IsArray()
  addresses: AddressDto[];
}
```

**Interview Tip**: `@ValidateNested()` validates nested objects. Use with `@Type()` from `class-transformer` to specify the nested class type.

</details>

## Swagger Decorators

### 32. What is `@ApiTags()` decorator?

<details>
<summary>Answer</summary>

`@ApiTags()` groups routes in Swagger documentation.

**Usage:**
```typescript
@Controller('users')
@ApiTags('users')
export class UsersController {}
```

**Interview Tip**: `@ApiTags(tag)` groups endpoints in Swagger UI. Apply to controllers for organized documentation.

</details>

### 33. What is `@ApiProperty()` used for?

<details>
<summary>Answer</summary>

`@ApiProperty()` documents DTO properties in Swagger.

**Usage:**
```typescript
export class CreateUserDto {
  @ApiProperty({ description: 'User name', example: 'John' })
  @IsString()
  name: string;

  @ApiProperty({ description: 'User email', example: 'john@example.com' })
  @IsEmail()
  email: string;

  @ApiProperty({ required: false, default: 18 })
  @IsOptional()
  age?: number;
}
```

**Interview Tip**: `@ApiProperty()` documents DTO fields in Swagger. Add descriptions, examples, and default values for better API docs.

</details>

### 34. What is `@ApiResponse()` decorator?

<details>
<summary>Answer</summary>

`@ApiResponse()` documents possible responses in Swagger.

**Usage:**
```typescript
@Post()
@ApiResponse({ status: 201, description: 'User created', type: User })
@ApiResponse({ status: 400, description: 'Bad request' })
@ApiResponse({ status: 409, description: 'User already exists' })
create(@Body() dto: CreateUserDto) {}
```

**Interview Tip**: `@ApiResponse({ status, description })` documents possible HTTP responses. Add for each status code your endpoint can return.

</details>

### 35. What is `@ApiBearerAuth()` for JWT authentication?

<details>
<summary>Answer</summary>

`@ApiBearerAuth()` adds JWT authentication to Swagger docs.

**Setup:**
```typescript
// main.ts
const config = new DocumentBuilder()
  .addBearerAuth()
  .build();
```

**Usage:**
```typescript
@Controller('users')
@ApiBearerAuth()
export class UsersController {
  @Get()
  @UseGuards(AuthGuard)
  findAll() {}
}
```

**Interview Tip**: `@ApiBearerAuth()` adds JWT auth to Swagger. Configure with `DocumentBuilder.addBearerAuth()` in main.ts. Apply to controllers or routes requiring authentication.

</details>
