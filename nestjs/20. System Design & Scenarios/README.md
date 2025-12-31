# NestJS System Design & Real-World Scenarios - Top Interview Questions

## API Design

<details>
<summary><strong>1. How do you design a RESTful API in NestJS?</strong></summary>

**Answer:**

Design a RESTful API in NestJS following these principles:

**1. Resource-Based Structure:**
```typescript
// users/users.controller.ts
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  // GET /users - Get all users
  @Get()
  async findAll(@Query() query: FindUsersDto): Promise<User[]> {
    return this.usersService.findAll(query);
  }

  // GET /users/:id - Get single user
  @Get(':id')
  async findOne(@Param('id', ParseIntPipe) id: number): Promise<User> {
    return this.usersService.findOne(id);
  }

  // POST /users - Create user
  @Post()
  @HttpCode(HttpStatus.CREATED)
  async create(@Body() createUserDto: CreateUserDto): Promise<User> {
    return this.usersService.create(createUserDto);
  }

  // PUT /users/:id - Full update
  @Put(':id')
  async update(
    @Param('id', ParseIntPipe) id: number,
    @Body() updateUserDto: UpdateUserDto,
  ): Promise<User> {
    return this.usersService.update(id, updateUserDto);
  }

  // PATCH /users/:id - Partial update
  @Patch(':id')
  async partialUpdate(
    @Param('id', ParseIntPipe) id: number,
    @Body() partialUpdateDto: Partial<UpdateUserDto>,
  ): Promise<User> {
    return this.usersService.partialUpdate(id, partialUpdateDto);
  }

  // DELETE /users/:id - Delete user
  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  async remove(@Param('id', ParseIntPipe) id: number): Promise<void> {
    return this.usersService.remove(id);
  }
}
```

**2. Nested Resources:**
```typescript
// users/users.controller.ts
@Controller('users')
export class UsersController {
  // GET /users/:userId/posts - Get user's posts
  @Get(':userId/posts')
  async getUserPosts(
    @Param('userId', ParseIntPipe) userId: number,
    @Query() query: FindPostsDto,
  ): Promise<Post[]> {
    return this.postsService.findByUser(userId, query);
  }

  // POST /users/:userId/posts - Create post for user
  @Post(':userId/posts')
  async createUserPost(
    @Param('userId', ParseIntPipe) userId: number,
    @Body() createPostDto: CreatePostDto,
  ): Promise<Post> {
    return this.postsService.create(userId, createPostDto);
  }
}
```

**3. DTOs with Validation:**
```typescript
// users/dto/create-user.dto.ts
import { IsEmail, IsString, MinLength, MaxLength, IsOptional } from 'class-validator';
import { ApiProperty } from '@nestjs/swagger';

export class CreateUserDto {
  @ApiProperty({ example: 'john.doe@example.com' })
  @IsEmail()
  email: string;

  @ApiProperty({ example: 'SecurePass123!' })
  @IsString()
  @MinLength(8)
  @MaxLength(50)
  password: string;

  @ApiProperty({ example: 'John Doe' })
  @IsString()
  @MinLength(2)
  @MaxLength(100)
  name: string;

  @ApiProperty({ example: 'Developer', required: false })
  @IsOptional()
  @IsString()
  role?: string;
}

// users/dto/update-user.dto.ts
import { PartialType } from '@nestjs/mapped-types';
export class UpdateUserDto extends PartialType(CreateUserDto) {}
```

**4. Response Standardization:**
```typescript
// common/interceptors/transform.interceptor.ts
@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, Response<T>> {
  intercept(context: ExecutionContext, next: CallHandler): Observable<Response<T>> {
    return next.handle().pipe(
      map(data => ({
        success: true,
        data,
        timestamp: new Date().toISOString(),
        path: context.switchToHttp().getRequest().url,
      })),
    );
  }
}

// Apply globally in main.ts
app.useGlobalInterceptors(new TransformInterceptor());
```

**5. Error Handling:**
```typescript
// common/filters/http-exception.filter.ts
@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const status = exception.getStatus();
    const exceptionResponse = exception.getResponse();

    const errorResponse = {
      success: false,
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      method: request.method,
      message: typeof exceptionResponse === 'string' 
        ? exceptionResponse 
        : (exceptionResponse as any).message,
      errors: typeof exceptionResponse === 'object' 
        ? (exceptionResponse as any).errors 
        : undefined,
    };

    response.status(status).json(errorResponse);
  }
}
```

**6. Swagger Documentation:**
```typescript
// main.ts
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';

const config = new DocumentBuilder()
  .setTitle('Users API')
  .setDescription('RESTful API for user management')
  .setVersion('1.0')
  .addBearerAuth()
  .addTag('users')
  .build();

const document = SwaggerModule.createDocument(app, config);
SwaggerModule.setup('api/docs', app, document);

// In controller, add decorators
@ApiTags('users')
@Controller('users')
export class UsersController {
  @ApiOperation({ summary: 'Get all users' })
  @ApiResponse({ status: 200, description: 'Returns all users', type: [User] })
  @Get()
  async findAll() { ... }

  @ApiOperation({ summary: 'Create a new user' })
  @ApiResponse({ status: 201, description: 'User created successfully', type: User })
  @ApiResponse({ status: 400, description: 'Invalid input' })
  @Post()
  async create(@Body() createUserDto: CreateUserDto) { ... }
}
```

**7. Pagination & Filtering:**
```typescript
// common/dto/pagination.dto.ts
export class PaginationDto {
  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  page?: number = 1;

  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  @Max(100)
  limit?: number = 10;
}

// users/dto/find-users.dto.ts
export class FindUsersDto extends PaginationDto {
  @IsOptional()
  @IsString()
  search?: string;

  @IsOptional()
  @IsString()
  sortBy?: string = 'createdAt';

  @IsOptional()
  @IsEnum(['ASC', 'DESC'])
  order?: 'ASC' | 'DESC' = 'DESC';
}

// users/users.service.ts
async findAll(query: FindUsersDto): Promise<{ data: User[]; meta: any }> {
  const { page, limit, search, sortBy, order } = query;
  const skip = (page - 1) * limit;

  const [data, total] = await this.userRepository.findAndCount({
    where: search ? { name: Like(`%${search}%`) } : {},
    order: { [sortBy]: order },
    skip,
    take: limit,
  });

  return {
    data,
    meta: {
      page,
      limit,
      total,
      totalPages: Math.ceil(total / limit),
    },
  };
}
```

**8. Versioning Strategy:**
```typescript
// main.ts
app.enableVersioning({
  type: VersioningType.URI,
  defaultVersion: '1',
});

// users/users.controller.ts
@Controller({ path: 'users', version: '1' })
export class UsersV1Controller { ... }

@Controller({ path: 'users', version: '2' })
export class UsersV2Controller { ... }
```

**Key Takeaway:** Design RESTful APIs with resource-based URLs, proper HTTP methods (GET/POST/PUT/PATCH/DELETE), DTOs with validation, standardized responses, comprehensive error handling, Swagger documentation, pagination support, and versioning strategy. Use decorators (@Controller, @Get, @Post, @Body, @Param, @Query) for clean, declarative code.

</details>

<details>
<summary><strong>2. What are REST best practices (resource naming, HTTP methods, status codes)?</strong></summary>

**Answer:**

**REST Best Practices:**

**1. Resource Naming Conventions:**

```typescript
// ✅ GOOD - Use plural nouns for collections
@Controller('users')          // /users
@Controller('products')       // /products
@Controller('orders')         // /orders

// ❌ BAD - Avoid verbs in URLs
@Controller('getUsers')       // /getUsers - WRONG
@Controller('createProduct')  // /createProduct - WRONG

// ✅ GOOD - Use hierarchical structure for relationships
GET    /users/:userId/orders           // Get user's orders
GET    /users/:userId/orders/:orderId  // Get specific order
POST   /users/:userId/orders           // Create order for user

// ✅ GOOD - Use kebab-case for multi-word resources
@Controller('user-profiles')    // /user-profiles
@Controller('order-items')      // /order-items

// ✅ GOOD - Keep URLs short and intuitive
GET /products/123/reviews       // Good
GET /api/v1/products/123/customer-reviews/all  // Too verbose

// ❌ BAD - Don't nest too deeply (max 2-3 levels)
GET /users/123/orders/456/items/789/reviews  // Too deep
// Better: GET /order-items/789/reviews
```

**2. HTTP Methods Usage:**

```typescript
@Controller('products')
export class ProductsController {
  // GET - Retrieve resources (Safe & Idempotent)
  @Get()  // GET /products - List all products
  @HttpCode(HttpStatus.OK) // 200
  async findAll(): Promise<Product[]> { ... }

  @Get(':id')  // GET /products/123 - Get single product
  @HttpCode(HttpStatus.OK) // 200
  async findOne(@Param('id') id: string): Promise<Product> { ... }

  // POST - Create new resource (Not Idempotent)
  @Post()  // POST /products - Create product
  @HttpCode(HttpStatus.CREATED) // 201
  async create(@Body() dto: CreateProductDto): Promise<Product> { ... }

  // PUT - Full replacement (Idempotent)
  @Put(':id')  // PUT /products/123 - Replace entire product
  @HttpCode(HttpStatus.OK) // 200
  async update(
    @Param('id') id: string,
    @Body() dto: UpdateProductDto, // All fields required
  ): Promise<Product> { ... }

  // PATCH - Partial update (Idempotent)
  @Patch(':id')  // PATCH /products/123 - Update specific fields
  @HttpCode(HttpStatus.OK) // 200
  async partialUpdate(
    @Param('id') id: string,
    @Body() dto: Partial<UpdateProductDto>, // Only changed fields
  ): Promise<Product> { ... }

  // DELETE - Remove resource (Idempotent)
  @Delete(':id')  // DELETE /products/123
  @HttpCode(HttpStatus.NO_CONTENT) // 204
  async remove(@Param('id') id: string): Promise<void> { ... }
}

// Bulk operations
@Post('bulk')  // POST /products/bulk - Create multiple
@HttpCode(HttpStatus.CREATED) // 201
async createBulk(@Body() dto: CreateProductDto[]): Promise<Product[]> { ... }

@Delete('bulk')  // DELETE /products/bulk?ids=1,2,3
@HttpCode(HttpStatus.NO_CONTENT) // 204
async removeBulk(@Query('ids') ids: string): Promise<void> { ... }
```

**3. HTTP Status Codes:**

```typescript
// Success Codes (2xx)
@Post()
@HttpCode(HttpStatus.CREATED) // 201 - Resource created
async create(@Body() dto: CreateDto) { ... }

@Get()
@HttpCode(HttpStatus.OK) // 200 - Success with response body
async findAll() { ... }

@Delete(':id')
@HttpCode(HttpStatus.NO_CONTENT) // 204 - Success without response body
async remove(@Param('id') id: string) { ... }

@Post('process')
@HttpCode(HttpStatus.ACCEPTED) // 202 - Request accepted for async processing
async process(@Body() dto: ProcessDto) { ... }

// Client Error Codes (4xx)
@Get(':id')
async findOne(@Param('id') id: string) {
  const product = await this.service.findOne(id);
  if (!product) {
    throw new NotFoundException('Product not found'); // 404
  }
  return product;
}

@Post()
async create(@Body() dto: CreateDto) {
  // Validation fails automatically with 400 - Bad Request
  return this.service.create(dto);
}

@Get('protected')
@UseGuards(AuthGuard)
async getProtected() {
  // If not authenticated: 401 - Unauthorized
  // If authenticated but no permission: 403 - Forbidden
  return this.service.getProtectedData();
}

@Post()
async create(@Body() dto: CreateDto) {
  const exists = await this.service.exists(dto.email);
  if (exists) {
    throw new ConflictException('Email already exists'); // 409
  }
  return this.service.create(dto);
}

@Post('upload')
async upload(@Body() dto: UploadDto) {
  if (dto.fileSize > MAX_SIZE) {
    throw new PayloadTooLargeException(); // 413
  }
  return this.service.upload(dto);
}

@Get()
async findAll(@Query() query: QueryDto) {
  if (query.invalidParam) {
    throw new UnprocessableEntityException('Invalid parameter'); // 422
  }
  return this.service.findAll(query);
}

@Post()
async create(@Body() dto: CreateDto) {
  // Rate limit exceeded
  throw new HttpException('Too Many Requests', HttpStatus.TOO_MANY_REQUESTS); // 429
}

// Server Error Codes (5xx)
@Get()
async findAll() {
  try {
    return await this.service.findAll();
  } catch (error) {
    throw new InternalServerErrorException('Server error occurred'); // 500
  }
}

@Post()
async create(@Body() dto: CreateDto) {
  throw new ServiceUnavailableException('Service temporarily unavailable'); // 503
}

@Get()
async findAll() {
  throw new GatewayTimeoutException('Upstream service timeout'); // 504
}
```

**4. Response Format Standardization:**

```typescript
// Success Response
{
  "success": true,
  "data": {
    "id": 123,
    "name": "Product Name",
    "price": 99.99
  },
  "meta": {
    "timestamp": "2025-12-30T10:00:00Z",
    "version": "1.0"
  }
}

// List Response with Pagination
{
  "success": true,
  "data": [ /* array of items */ ],
  "meta": {
    "page": 1,
    "limit": 10,
    "total": 100,
    "totalPages": 10
  }
}

// Error Response
{
  "success": false,
  "statusCode": 400,
  "message": "Validation failed",
  "errors": [
    {
      "field": "email",
      "message": "Invalid email format"
    }
  ],
  "timestamp": "2025-12-30T10:00:00Z",
  "path": "/api/users"
}
```

**5. Query Parameters Best Practices:**

```typescript
// Filtering
GET /products?category=electronics&minPrice=100&maxPrice=500

// Sorting
GET /products?sortBy=price&order=asc

// Pagination
GET /products?page=2&limit=20

// Searching
GET /products?search=laptop

// Field selection (sparse fieldsets)
GET /products?fields=id,name,price

// Multiple filters with operators
GET /products?price[gte]=100&price[lte]=500&stock[gt]=0

@Controller('products')
export class ProductsController {
  @Get()
  async findAll(
    @Query('category') category?: string,
    @Query('minPrice', new DefaultValuePipe(0), ParseIntPipe) minPrice?: number,
    @Query('maxPrice', new DefaultValuePipe(10000), ParseIntPipe) maxPrice?: number,
    @Query('sortBy', new DefaultValuePipe('createdAt')) sortBy?: string,
    @Query('order', new DefaultValuePipe('DESC')) order?: 'ASC' | 'DESC',
    @Query('page', new DefaultValuePipe(1), ParseIntPipe) page?: number,
    @Query('limit', new DefaultValuePipe(10), ParseIntPipe) limit?: number,
    @Query('search') search?: string,
  ) {
    return this.service.findAll({
      category, minPrice, maxPrice, sortBy, order, page, limit, search
    });
  }
}
```

**6. Header Usage:**

```typescript
// Request Headers
@Post()
async create(
  @Headers('authorization') auth: string,
  @Headers('content-type') contentType: string,
  @Headers('x-request-id') requestId: string,
  @Body() dto: CreateDto,
) { ... }

// Response Headers
@Post()
@Header('X-RateLimit-Limit', '100')
@Header('X-RateLimit-Remaining', '99')
async create(@Body() dto: CreateDto) { ... }

// Or set headers dynamically
@Post()
async create(@Res() response: Response, @Body() dto: CreateDto) {
  const result = await this.service.create(dto);
  response
    .header('Location', `/products/${result.id}`)
    .status(201)
    .json(result);
}
```

**7. HATEOAS (Hypermedia) - Advanced:**

```typescript
// Include links in responses for discoverability
{
  "id": 123,
  "name": "Product",
  "price": 99.99,
  "_links": {
    "self": { "href": "/products/123" },
    "update": { "href": "/products/123", "method": "PUT" },
    "delete": { "href": "/products/123", "method": "DELETE" },
    "reviews": { "href": "/products/123/reviews" },
    "category": { "href": "/categories/5" }
  }
}
```

**Key Takeaway:** Follow REST conventions: use plural nouns for resources (/users, /products), apply correct HTTP methods (GET-retrieve, POST-create, PUT-replace, PATCH-update, DELETE-remove), return appropriate status codes (200-OK, 201-Created, 204-No Content, 400-Bad Request, 404-Not Found, 500-Server Error), standardize response format with success/error structure, implement pagination/filtering/sorting with query parameters, use proper headers, and keep URLs clean and hierarchical (max 2-3 levels deep).

</details>

<details>
<summary><strong>3. How do you version your APIs?</strong></summary>

**Answer:**

Implement API versioning in NestJS using multiple strategies:

**1. URI Versioning (Most Common):**

```typescript
// main.ts - Enable URI versioning
import { VersioningType } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Enable versioning
  app.enableVersioning({
    type: VersioningType.URI,
    defaultVersion: '1', // Default version if not specified
  });
  
  await app.listen(3000);
}
bootstrap();

// users.v1.controller.ts - Version 1
@Controller({
  path: 'users',
  version: '1',
})
export class UsersV1Controller {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  async findAll(): Promise<UserV1Dto[]> {
    // Version 1 logic with old schema
    return this.usersService.findAllV1();
  }

  @Get(':id')
  async findOne(@Param('id') id: string): Promise<UserV1Dto> {
    return this.usersService.findOneV1(id);
  }
}

// users.v2.controller.ts - Version 2
@Controller({
  path: 'users',
  version: '2',
})
export class UsersV2Controller {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  async findAll(@Query() query: PaginationDto): Promise<PaginatedResponse<UserV2Dto>> {
    // Version 2 with enhanced features (pagination, new fields)
    return this.usersService.findAllV2(query);
  }

  @Get(':id')
  async findOne(@Param('id') id: string): Promise<UserV2Dto> {
    // Version 2 returns additional fields
    return this.usersService.findOneV2(id);
  }

  // New endpoint only in v2
  @Get(':id/profile')
  async getProfile(@Param('id') id: string): Promise<UserProfileDto> {
    return this.usersService.getUserProfile(id);
  }
}

// Access: GET /v1/users and GET /v2/users
```

**2. Header Versioning:**

```typescript
// main.ts
app.enableVersioning({
  type: VersioningType.HEADER,
  header: 'X-API-Version', // Custom header name
});

// Controller remains the same
@Controller({
  path: 'users',
  version: '1',
})
export class UsersV1Controller { ... }

// Client request with header:
// GET /users
// Headers: X-API-Version: 1
```

**3. Media Type Versioning (Accept Header):**

```typescript
// main.ts
app.enableVersioning({
  type: VersioningType.MEDIA_TYPE,
  key: 'v=', // Version key in Accept header
});

// Controller
@Controller('users')
export class UsersController {
  @Version('1')
  @Get()
  async findAllV1(): Promise<UserV1Dto[]> { ... }

  @Version('2')
  @Get()
  async findAllV2(): Promise<UserV2Dto[]> { ... }
}

// Client request:
// GET /users
// Headers: Accept: application/json;v=1
```

**4. Custom Versioning:**

```typescript
// main.ts
app.enableVersioning({
  type: VersioningType.CUSTOM,
  extractor: (request: Request) => {
    // Extract version from custom logic
    const version = request.headers['api-version'];
    if (version) {
      return Array.isArray(version) ? version[0] : version;
    }
    // Can extract from subdomain, query param, etc.
    return '1'; // Default version
  },
});
```

**5. Multiple Versions Support:**

```typescript
// Support multiple versions on single endpoint
@Controller('users')
export class UsersController {
  @Version(['1', '2']) // Works for both v1 and v2
  @Get(':id')
  async findOne(@Param('id') id: string): Promise<UserDto> {
    return this.usersService.findOne(id);
  }

  @Version('2') // Only available in v2
  @Get('search')
  async search(@Query() query: SearchDto): Promise<UserDto[]> {
    return this.usersService.search(query);
  }
}
```

**6. Version-Specific DTOs:**

```typescript
// dtos/v1/user-v1.dto.ts
export class UserV1Dto {
  @ApiProperty()
  id: string;

  @ApiProperty()
  email: string;

  @ApiProperty()
  name: string;

  @ApiProperty()
  createdAt: Date;
}

// dtos/v2/user-v2.dto.ts
export class UserV2Dto {
  @ApiProperty()
  id: string;

  @ApiProperty()
  email: string;

  @ApiProperty()
  firstName: string; // Split from 'name'

  @ApiProperty()
  lastName: string;

  @ApiProperty()
  phoneNumber: string; // New field in v2

  @ApiProperty()
  avatar: string; // New field in v2

  @ApiProperty()
  createdAt: Date;

  @ApiProperty()
  updatedAt: Date; // New field in v2
}

// Transformation service
@Injectable()
export class UserTransformService {
  toV1(user: User): UserV1Dto {
    return {
      id: user.id,
      email: user.email,
      name: `${user.firstName} ${user.lastName}`,
      createdAt: user.createdAt,
    };
  }

  toV2(user: User): UserV2Dto {
    return {
      id: user.id,
      email: user.email,
      firstName: user.firstName,
      lastName: user.lastName,
      phoneNumber: user.phoneNumber,
      avatar: user.avatar,
      createdAt: user.createdAt,
      updatedAt: user.updatedAt,
    };
  }
}
```

**7. Service Layer Handling:**

```typescript
// users.service.ts
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>,
    private transformService: UserTransformService,
  ) {}

  // V1 - Returns basic fields
  async findAllV1(): Promise<UserV1Dto[]> {
    const users = await this.userRepository.find({
      select: ['id', 'email', 'firstName', 'lastName', 'createdAt'],
    });
    return users.map(user => this.transformService.toV1(user));
  }

  // V2 - Returns all fields with pagination
  async findAllV2(query: PaginationDto): Promise<PaginatedResponse<UserV2Dto>> {
    const { page = 1, limit = 10 } = query;
    const [users, total] = await this.userRepository.findAndCount({
      skip: (page - 1) * limit,
      take: limit,
    });

    return {
      data: users.map(user => this.transformService.toV2(user)),
      meta: {
        page,
        limit,
        total,
        totalPages: Math.ceil(total / limit),
      },
    };
  }
}
```

**8. Swagger Documentation for Versions:**

```typescript
// main.ts
const config = new DocumentBuilder()
  .setTitle('API Documentation')
  .setVersion('2.0')
  .addServer('/v1', 'API Version 1')
  .addServer('/v2', 'API Version 2')
  .build();

const document = SwaggerModule.createDocument(app, config);
SwaggerModule.setup('api/docs', app, document);

// Or separate Swagger docs per version
const v1Config = new DocumentBuilder()
  .setTitle('API v1')
  .setVersion('1.0')
  .build();

const v1Document = SwaggerModule.createDocument(app, v1Config, {
  include: [UsersV1Module], // Only include v1 modules
});
SwaggerModule.setup('api/v1/docs', app, v1Document);

const v2Config = new DocumentBuilder()
  .setTitle('API v2')
  .setVersion('2.0')
  .build();

const v2Document = SwaggerModule.createDocument(app, v2Config, {
  include: [UsersV2Module], // Only include v2 modules
});
SwaggerModule.setup('api/v2/docs', app, v2Document);
```

**9. Deprecation Strategy:**

```typescript
// users.v1.controller.ts
@Controller({
  path: 'users',
  version: '1',
})
@ApiDeprecated() // Mark as deprecated in Swagger
export class UsersV1Controller {
  @Get()
  @Header('X-API-Deprecated', 'true')
  @Header('X-API-Sunset', '2026-12-31') // Sunset date
  @Header('Link', '</v2/users>; rel="successor-version"')
  async findAll(): Promise<UserV1Dto[]> {
    // Log deprecation warning
    this.logger.warn('API v1 /users called - This version is deprecated');
    return this.usersService.findAllV1();
  }
}

// Deprecation interceptor
@Injectable()
export class DeprecationInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const response = context.switchToHttp().getResponse();
    
    // Check if endpoint is deprecated (based on version or metadata)
    const version = request.params?.version || request.headers['x-api-version'];
    
    if (version === '1') {
      response.setHeader('Deprecation', 'true');
      response.setHeader('Sunset', 'Sat, 31 Dec 2026 23:59:59 GMT');
      response.setHeader('Link', '<https://api.example.com/v2/docs>; rel="deprecation"');
    }
    
    return next.handle();
  }
}
```

**10. Migration Strategy:**

```typescript
// Feature flag for gradual migration
@Injectable()
export class UsersService {
  constructor(
    private configService: ConfigService,
    private featureFlagService: FeatureFlagService,
  ) {}

  async findAll(version: string): Promise<any> {
    // Gradual rollout: some v1 users get v2 features
    const useV2Features = this.featureFlagService.isEnabled('v2-features-for-v1');
    
    if (version === '2' || useV2Features) {
      return this.findAllV2();
    }
    
    return this.findAllV1();
  }
}

// Module organization
@Module({
  imports: [
    UsersV1Module, // Contains v1 controllers and services
    UsersV2Module, // Contains v2 controllers and services
  ],
})
export class UsersModule {}
```

**Best Practices:**
- Use URI versioning (/v1/users) for simplicity and visibility
- Keep old versions running while clients migrate (6-12 months)
- Document breaking changes clearly in release notes
- Use semantic versioning (major.minor.patch)
- Deprecate old versions gracefully with sunset dates
- Share common logic between versions (service layer)
- Version only controllers and DTOs, not services
- Test all versions in CI/CD pipeline
- Monitor usage of old versions to plan deprecation
- Consider backward compatibility for minor changes

**Key Takeaway:** Use URI versioning (`app.enableVersioning({ type: VersioningType.URI })`) with `@Controller({ path: 'users', version: '1' })` for clear, visible API versions accessible at /v1/users and /v2/users. Support multiple versions simultaneously, use version-specific DTOs for schema changes, implement graceful deprecation with sunset dates, and maintain separate Swagger docs per version. Keep service layer version-agnostic where possible.

</details>

<details>
<summary><strong>4. What is the difference between PUT and PATCH?</strong></summary>

**Answer:**

**PUT vs PATCH Differences:**

**1. Conceptual Difference:**

```typescript
// PUT - Complete replacement (all fields required)
// PATCH - Partial update (only changed fields)

// Example Entity
class User {
  id: string;
  email: string;
  firstName: string;
  lastName: string;
  phoneNumber: string;
  role: string;
  isActive: boolean;
}

// Current state in database:
{
  id: '123',
  email: 'john@example.com',
  firstName: 'John',
  lastName: 'Doe',
  phoneNumber: '+1234567890',
  role: 'user',
  isActive: true
}
```

**2. PUT Implementation (Full Replacement):**

```typescript
// users/dto/update-user.dto.ts
export class UpdateUserDto {
  @IsEmail()
  @IsNotEmpty()
  email: string;

  @IsString()
  @IsNotEmpty()
  firstName: string;

  @IsString()
  @IsNotEmpty()
  lastName: string;

  @IsString()
  @IsNotEmpty()
  phoneNumber: string;

  @IsString()
  @IsNotEmpty()
  role: string;

  @IsBoolean()
  @IsNotEmpty()
  isActive: boolean;
}

// users.controller.ts
@Controller('users')
export class UsersController {
  @Put(':id')
  @HttpCode(HttpStatus.OK)
  async update(
    @Param('id') id: string,
    @Body() updateUserDto: UpdateUserDto, // ALL fields required
  ): Promise<User> {
    return this.usersService.update(id, updateUserDto);
  }
}

// users.service.ts
async update(id: string, updateUserDto: UpdateUserDto): Promise<User> {
  const user = await this.userRepository.findOne({ where: { id } });
  if (!user) {
    throw new NotFoundException(`User ${id} not found`);
  }

  // Replace ALL fields
  user.email = updateUserDto.email;
  user.firstName = updateUserDto.firstName;
  user.lastName = updateUserDto.lastName;
  user.phoneNumber = updateUserDto.phoneNumber;
  user.role = updateUserDto.role;
  user.isActive = updateUserDto.isActive;

  return this.userRepository.save(user);
}

// PUT Request - Must provide ALL fields
// PUT /users/123
{
  "email": "john@example.com",
  "firstName": "John",
  "lastName": "Smith",         // Only changing lastName
  "phoneNumber": "+1234567890",
  "role": "user",
  "isActive": true
}

// Result: Full replacement - lastName updated, all other fields reaffirmed
```

**3. PATCH Implementation (Partial Update):**

```typescript
// users/dto/patch-user.dto.ts
export class PatchUserDto {
  @IsEmail()
  @IsOptional()
  email?: string;

  @IsString()
  @IsOptional()
  firstName?: string;

  @IsString()
  @IsOptional()
  lastName?: string;

  @IsString()
  @IsOptional()
  phoneNumber?: string;

  @IsString()
  @IsOptional()
  role?: string;

  @IsBoolean()
  @IsOptional()
  isActive?: boolean;
}

// users.controller.ts
@Controller('users')
export class UsersController {
  @Patch(':id')
  @HttpCode(HttpStatus.OK)
  async partialUpdate(
    @Param('id') id: string,
    @Body() patchUserDto: PatchUserDto, // Only changed fields
  ): Promise<User> {
    return this.usersService.partialUpdate(id, patchUserDto);
  }
}

// users.service.ts
async partialUpdate(id: string, patchUserDto: PatchUserDto): Promise<User> {
  const user = await this.userRepository.findOne({ where: { id } });
  if (!user) {
    throw new NotFoundException(`User ${id} not found`);
  }

  // Update only provided fields
  Object.assign(user, patchUserDto);
  
  // Or more explicit:
  if (patchUserDto.email !== undefined) user.email = patchUserDto.email;
  if (patchUserDto.firstName !== undefined) user.firstName = patchUserDto.firstName;
  if (patchUserDto.lastName !== undefined) user.lastName = patchUserDto.lastName;
  if (patchUserDto.phoneNumber !== undefined) user.phoneNumber = patchUserDto.phoneNumber;
  if (patchUserDto.role !== undefined) user.role = patchUserDto.role;
  if (patchUserDto.isActive !== undefined) user.isActive = patchUserDto.isActive;

  return this.userRepository.save(user);
}

// PATCH Request - Only send changed fields
// PATCH /users/123
{
  "lastName": "Smith"  // Only updating lastName
}

// Result: Only lastName updated, all other fields unchanged
```

**4. Idempotency:**

```typescript
// PUT is idempotent - same request multiple times = same result
// PATCH may or may not be idempotent depending on implementation

// Idempotent PATCH (field replacement)
PATCH /users/123
{ "lastName": "Smith" }
// Call 1: lastName = "Smith"
// Call 2: lastName = "Smith" (idempotent)

// Non-idempotent PATCH (operations)
PATCH /users/123
{ "operation": "increment", "field": "loginCount", "value": 1 }
// Call 1: loginCount = 11 (was 10)
// Call 2: loginCount = 12 (NOT idempotent)

// Recommendation: Make PATCH idempotent by using field replacement, not operations
```

**5. Comparison Table:**

| Aspect | PUT | PATCH |
|--------|-----|-------|
| **Purpose** | Complete replacement | Partial update |
| **Fields** | All required | Only changed fields |
| **Idempotent** | Always | Should be (depends on implementation) |
| **Missing fields** | Set to null/default | Left unchanged |
| **Payload size** | Larger (all fields) | Smaller (only changes) |
| **Use case** | Replace entire resource | Update specific fields |
| **Safety** | May accidentally null fields | Safer for partial updates |

**6. Real-World Example - User Profile:**

```typescript
@Controller('users')
export class UsersController {
  // PUT - Update complete profile (all fields required)
  @Put(':id/profile')
  async updateProfile(
    @Param('id') id: string,
    @Body() dto: UpdateProfileDto, // All profile fields required
  ): Promise<UserProfile> {
    // Client must send: firstName, lastName, bio, avatar, phone, address, etc.
    return this.usersService.updateProfile(id, dto);
  }

  // PATCH - Update specific profile fields
  @Patch(':id/profile')
  async patchProfile(
    @Param('id') id: string,
    @Body() dto: PatchProfileDto, // Only changed fields
  ): Promise<UserProfile> {
    // Client can send: { "bio": "New bio" } - only update bio
    return this.usersService.patchProfile(id, dto);
  }

  // PATCH - Specific action endpoints (common pattern)
  @Patch(':id/activate')
  async activate(@Param('id') id: string): Promise<User> {
    return this.usersService.updateStatus(id, { isActive: true });
  }

  @Patch(':id/deactivate')
  async deactivate(@Param('id') id: string): Promise<User> {
    return this.usersService.updateStatus(id, { isActive: false });
  }

  @Patch(':id/change-role')
  async changeRole(
    @Param('id') id: string,
    @Body() dto: ChangeRoleDto, // { role: 'admin' }
  ): Promise<User> {
    return this.usersService.updateRole(id, dto.role);
  }
}
```

**7. Using PartialType Helper:**

```typescript
import { PartialType } from '@nestjs/mapped-types';
// or for Swagger: import { PartialType } from '@nestjs/swagger';

// Base DTO with all fields required
export class CreateUserDto {
  @IsEmail()
  email: string;

  @IsString()
  firstName: string;

  @IsString()
  lastName: string;

  @IsString()
  phoneNumber: string;
}

// PUT DTO - all fields required (same as CreateUserDto)
export class UpdateUserDto extends CreateUserDto {}

// PATCH DTO - all fields optional (automatically)
export class PatchUserDto extends PartialType(CreateUserDto) {}
// Equivalent to:
// export class PatchUserDto {
//   email?: string;
//   firstName?: string;
//   lastName?: string;
//   phoneNumber?: string;
// }
```

**8. Error Handling:**

```typescript
@Put(':id')
async update(@Param('id') id: string, @Body() dto: UpdateUserDto) {
  // PUT with missing required fields -> 400 Bad Request (validation fails)
  // Example: { "firstName": "John" } without other required fields
  return this.usersService.update(id, dto);
}

@Patch(':id')
async patch(@Param('id') id: string, @Body() dto: PatchUserDto) {
  // PATCH with empty body -> Could be 400 or just return unchanged resource
  if (Object.keys(dto).length === 0) {
    throw new BadRequestException('No fields provided for update');
  }
  
  // PATCH with invalid field values -> 400 Bad Request
  return this.usersService.patch(id, dto);
}
```

**9. Advanced PATCH with JSON Patch (RFC 6902):**

```typescript
// JSON Patch format for complex operations
PATCH /users/123
Content-Type: application/json-patch+json
[
  { "op": "replace", "path": "/lastName", "value": "Smith" },
  { "op": "add", "path": "/phoneNumbers/-", "value": "+9876543210" },
  { "op": "remove", "path": "/middleName" }
]

// Implementation with json-patch library
import * as jsonpatch from 'fast-json-patch';

@Patch(':id')
async patchWithJsonPatch(
  @Param('id') id: string,
  @Body() patches: jsonpatch.Operation[],
) {
  const user = await this.usersService.findOne(id);
  const updatedUser = jsonpatch.applyPatch(user, patches).newDocument;
  return this.usersService.save(updatedUser);
}
```

**When to Use Each:**

- **Use PUT when:**
  - Replacing entire resource
  - Client has full resource representation
  - Want to ensure all fields are explicitly set
  - Implementing "save" or "overwrite" functionality

- **Use PATCH when:**
  - Updating specific fields only
  - Client doesn't have full resource representation
  - Minimizing bandwidth (mobile apps)
  - Implementing "edit" functionality (forms with partial fields)
  - Performing specific actions (activate/deactivate/change-role)

**Key Takeaway:** PUT replaces entire resource requiring ALL fields (`UpdateUserDto` with all required fields), PATCH updates only specified fields (`PatchUserDto` with all optional fields using `PartialType`). PUT is always idempotent, PATCH should be. Use PUT for full replacement, PATCH for partial updates. In NestJS, implement as `@Put(':id')` with complete DTO and `@Patch(':id')` with partial DTO (fields marked `@IsOptional()`).

</details>

<details>
<summary><strong>5. How do you handle API pagination?</strong></summary>

**Answer:**

Implement pagination in NestJS using multiple strategies:

**1. Offset-Based Pagination (Page & Limit):**

```typescript
// common/dto/pagination.dto.ts
import { Type } from 'class-transformer';
import { IsInt, IsOptional, Min, Max } from 'class-validator';
import { ApiPropertyOptional } from '@nestjs/swagger';

export class PaginationDto {
  @ApiPropertyOptional({ default: 1, minimum: 1 })
  @Type(() => Number)
  @IsInt()
  @Min(1)
  @IsOptional()
  page?: number = 1;

  @ApiPropertyOptional({ default: 10, minimum: 1, maximum: 100 })
  @Type(() => Number)
  @IsInt()
  @Min(1)
  @Max(100) // Prevent requesting too many items
  @IsOptional()
  limit?: number = 10;
}

// common/interfaces/paginated-response.interface.ts
export interface PaginatedResponse<T> {
  data: T[];
  meta: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
    hasNextPage: boolean;
    hasPrevPage: boolean;
  };
  links?: {
    first: string;
    prev: string | null;
    next: string | null;
    last: string;
  };
}

// products/products.controller.ts
@Controller('products')
export class ProductsController {
  @Get()
  @ApiQuery({ name: 'page', required: false, type: Number })
  @ApiQuery({ name: 'limit', required: false, type: Number })
  async findAll(
    @Query() paginationDto: PaginationDto,
  ): Promise<PaginatedResponse<Product>> {
    return this.productsService.findAll(paginationDto);
  }
}

// products/products.service.ts
@Injectable()
export class ProductsService {
  constructor(
    @InjectRepository(Product)
    private productRepository: Repository<Product>,
  ) {}

  async findAll(paginationDto: PaginationDto): Promise<PaginatedResponse<Product>> {
    const { page = 1, limit = 10 } = paginationDto;
    const skip = (page - 1) * limit;

    // Get data and total count
    const [data, total] = await this.productRepository.findAndCount({
      skip,
      take: limit,
      order: { createdAt: 'DESC' },
    });

    const totalPages = Math.ceil(total / limit);
    const hasNextPage = page < totalPages;
    const hasPrevPage = page > 1;

    return {
      data,
      meta: {
        page,
        limit,
        total,
        totalPages,
        hasNextPage,
        hasPrevPage,
      },
      links: {
        first: `/products?page=1&limit=${limit}`,
        prev: hasPrevPage ? `/products?page=${page - 1}&limit=${limit}` : null,
        next: hasNextPage ? `/products?page=${page + 1}&limit=${limit}` : null,
        last: `/products?page=${totalPages}&limit=${limit}`,
      },
    };
  }
}

// Request: GET /products?page=2&limit=20
// Response:
{
  "data": [ /* 20 products */ ],
  "meta": {
    "page": 2,
    "limit": 20,
    "total": 150,
    "totalPages": 8,
    "hasNextPage": true,
    "hasPrevPage": true
  },
  "links": {
    "first": "/products?page=1&limit=20",
    "prev": "/products?page=1&limit=20",
    "next": "/products?page=3&limit=20",
    "last": "/products?page=8&limit=20"
  }
}
```

**2. Cursor-Based Pagination (For Infinite Scroll):**

```typescript
// common/dto/cursor-pagination.dto.ts
export class CursorPaginationDto {
  @ApiPropertyOptional()
  @IsOptional()
  @IsString()
  cursor?: string; // Base64 encoded last item ID

  @ApiPropertyOptional({ default: 20 })
  @Type(() => Number)
  @IsInt()
  @Min(1)
  @Max(100)
  @IsOptional()
  limit?: number = 20;
}

// common/interfaces/cursor-paginated-response.interface.ts
export interface CursorPaginatedResponse<T> {
  data: T[];
  meta: {
    hasMore: boolean;
    nextCursor: string | null;
    limit: number;
  };
}

// products/products.service.ts
async findAllCursor(dto: CursorPaginationDto): Promise<CursorPaginatedResponse<Product>> {
  const { cursor, limit = 20 } = dto;
  
  let query = this.productRepository
    .createQueryBuilder('product')
    .orderBy('product.id', 'DESC')
    .take(limit + 1); // Fetch one extra to check if more exist

  // If cursor exists, fetch items after cursor
  if (cursor) {
    const decodedCursor = Buffer.from(cursor, 'base64').toString('utf-8');
    query = query.where('product.id < :cursor', { cursor: decodedCursor });
  }

  const data = await query.getMany();
  const hasMore = data.length > limit;

  // Remove extra item used for hasMore check
  if (hasMore) {
    data.pop();
  }

  // Generate next cursor from last item
  const nextCursor = hasMore && data.length > 0
    ? Buffer.from(data[data.length - 1].id.toString()).toString('base64')
    : null;

  return {
    data,
    meta: {
      hasMore,
      nextCursor,
      limit,
    },
  };
}

// Request 1: GET /products?limit=20
// Response:
{
  "data": [ /* 20 products */ ],
  "meta": {
    "hasMore": true,
    "nextCursor": "MTIz", // Base64 encoded ID
    "limit": 20
  }
}

// Request 2: GET /products?cursor=MTIz&limit=20 (next page)
// Response: Next 20 items after cursor
```

**3. Keyset Pagination (Most Efficient for Large Datasets):**

```typescript
// For pagination based on timestamp or sequential ID
export class KeysetPaginationDto {
  @IsOptional()
  @Type(() => Date)
  @IsDate()
  after?: Date; // createdAt of last item from previous page

  @IsOptional()
  @IsInt()
  @Min(1)
  @Max(100)
  limit?: number = 20;
}

async findAllKeyset(dto: KeysetPaginationDto): Promise<KeysetPaginatedResponse<Product>> {
  const { after, limit = 20 } = dto;

  const query = this.productRepository
    .createQueryBuilder('product')
    .orderBy('product.createdAt', 'DESC')
    .addOrderBy('product.id', 'DESC') // Tie-breaker for same timestamp
    .take(limit + 1);

  if (after) {
    query.where('product.createdAt < :after', { after });
  }

  const data = await query.getMany();
  const hasMore = data.length > limit;

  if (hasMore) {
    data.pop();
  }

  const nextAfter = hasMore && data.length > 0
    ? data[data.length - 1].createdAt
    : null;

  return {
    data,
    meta: {
      hasMore,
      nextAfter,
      limit,
    },
  };
}

// More efficient than offset for large datasets
// No skip/offset calculation needed
```

**4. Paginated Response Interceptor:**

```typescript
// common/interceptors/pagination.interceptor.ts
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

@Injectable()
export class PaginationInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const baseUrl = `${request.protocol}://${request.get('host')}${request.path}`;

    return next.handle().pipe(
      map(response => {
        if (response && response.meta && response.meta.page) {
          // Add full URLs to links
          const { page, limit, totalPages } = response.meta;
          
          response.links = {
            self: `${baseUrl}?page=${page}&limit=${limit}`,
            first: `${baseUrl}?page=1&limit=${limit}`,
            prev: page > 1 ? `${baseUrl}?page=${page - 1}&limit=${limit}` : null,
            next: page < totalPages ? `${baseUrl}?page=${page + 1}&limit=${limit}` : null,
            last: `${baseUrl}?page=${totalPages}&limit=${limit}`,
          };
        }
        return response;
      }),
    );
  }
}

// Apply to controller
@UseInterceptors(PaginationInterceptor)
@Controller('products')
export class ProductsController { ... }
```

**5. Pagination with TypeORM Paginate Package:**

```typescript
// Install: npm install nestjs-typeorm-paginate

import { paginate, Pagination, IPaginationOptions } from 'nestjs-typeorm-paginate';

async findAllWithPaginate(options: IPaginationOptions): Promise<Pagination<Product>> {
  const queryBuilder = this.productRepository.createQueryBuilder('product');
  
  return paginate<Product>(queryBuilder, options);
}

// Controller
@Get()
async findAll(
  @Query('page', new DefaultValuePipe(1), ParseIntPipe) page: number,
  @Query('limit', new DefaultValuePipe(10), ParseIntPipe) limit: number,
): Promise<Pagination<Product>> {
  return this.productsService.findAllWithPaginate({ page, limit });
}

// Automatic response format:
{
  "items": [ /* products */ ],
  "meta": {
    "totalItems": 150,
    "itemCount": 10,
    "itemsPerPage": 10,
    "totalPages": 15,
    "currentPage": 1
  },
  "links": {
    "first": "...",
    "previous": "...",
    "next": "...",
    "last": "..."
  }
}
```

**6. Pagination with Filters:**

```typescript
export class ProductQueryDto extends PaginationDto {
  @IsOptional()
  @IsString()
  search?: string;

  @IsOptional()
  @IsString()
  category?: string;

  @IsOptional()
  @Type(() => Number)
  @IsNumber()
  minPrice?: number;

  @IsOptional()
  @Type(() => Number)
  @IsNumber()
  maxPrice?: number;
}

async findAll(queryDto: ProductQueryDto): Promise<PaginatedResponse<Product>> {
  const { page = 1, limit = 10, search, category, minPrice, maxPrice } = queryDto;
  const skip = (page - 1) * limit;

  const queryBuilder = this.productRepository
    .createQueryBuilder('product')
    .skip(skip)
    .take(limit);

  // Apply filters
  if (search) {
    queryBuilder.andWhere(
      'product.name ILIKE :search OR product.description ILIKE :search',
      { search: `%${search}%` },
    );
  }

  if (category) {
    queryBuilder.andWhere('product.category = :category', { category });
  }

  if (minPrice !== undefined) {
    queryBuilder.andWhere('product.price >= :minPrice', { minPrice });
  }

  if (maxPrice !== undefined) {
    queryBuilder.andWhere('product.price <= :maxPrice', { maxPrice });
  }

  const [data, total] = await queryBuilder.getManyAndCount();

  return {
    data,
    meta: {
      page,
      limit,
      total,
      totalPages: Math.ceil(total / limit),
      hasNextPage: page < Math.ceil(total / limit),
      hasPrevPage: page > 1,
    },
  };
}

// Request: GET /products?page=1&limit=20&search=laptop&category=electronics&minPrice=500
```

**7. HTTP Headers for Pagination (Alternative):**

```typescript
@Get()
async findAll(
  @Query() paginationDto: PaginationDto,
  @Res() response: Response,
) {
  const result = await this.productsService.findAll(paginationDto);
  
  // Set pagination info in headers
  response.setHeader('X-Total-Count', result.meta.total);
  response.setHeader('X-Total-Pages', result.meta.totalPages);
  response.setHeader('X-Current-Page', result.meta.page);
  response.setHeader('X-Per-Page', result.meta.limit);
  
  // Set Link header (GitHub style)
  const links = [];
  if (result.meta.hasNextPage) {
    links.push(`</products?page=${result.meta.page + 1}&limit=${result.meta.limit}>; rel="next"`);
  }
  if (result.meta.hasPrevPage) {
    links.push(`</products?page=${result.meta.page - 1}&limit=${result.meta.limit}>; rel="prev"`);
  }
  if (links.length > 0) {
    response.setHeader('Link', links.join(', '));
  }
  
  return response.json(result.data); // Return only data array
}
```

**8. Performance Optimization:**

```typescript
// Add index on commonly paginated columns
@Entity()
@Index(['createdAt', 'id']) // Composite index for keyset pagination
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  @Index() // Index for filtering
  category: string;

  @Column()
  @Index() // Index for sorting/pagination
  createdAt: Date;
}

// Use select to limit fields
async findAll(dto: PaginationDto) {
  const { page = 1, limit = 10 } = dto;
  
  return this.productRepository.findAndCount({
    select: ['id', 'name', 'price', 'createdAt'], // Only needed fields
    skip: (page - 1) * limit,
    take: limit,
  });
}

// Cache total count for large datasets
async findAll(dto: PaginationDto) {
  const cacheKey = `products:total`;
  let total = await this.cacheManager.get<number>(cacheKey);
  
  if (!total) {
    total = await this.productRepository.count();
    await this.cacheManager.set(cacheKey, total, 300); // Cache for 5 minutes
  }
  
  // ... rest of pagination logic
}
```

**Comparison Table:**

| Type | Use Case | Pros | Cons |
|------|----------|------|------|
| **Offset** | Admin panels, small datasets | Simple, jump to any page | Slow for large offsets, inconsistent with updates |
| **Cursor** | Infinite scroll, feeds | Consistent results, efficient | Can't jump to specific page |
| **Keyset** | Large datasets, high performance | Very fast, consistent | Requires indexed column, can't jump to page |

**Key Takeaway:** Implement offset-based pagination with `PaginationDto` containing page/limit, use TypeORM's `skip` and `take` with `findAndCount()`, return `PaginatedResponse` with data array and meta (page, limit, total, totalPages, hasNextPage, hasPrevPage). For infinite scroll use cursor-based pagination, for large datasets use keyset pagination with indexed columns. Add validation (`@Max(100)` on limit), include links for navigation, and optimize with database indexes.

</details>

<details>
<summary><strong>6. How do you implement filtering, sorting, and searching?</strong></summary>

**Answer:**

Implement filtering, sorting, and searching in NestJS APIs:

**1. Complete Query DTO:**

```typescript
// common/dto/query.dto.ts
import { IsOptional, IsString, IsEnum, IsInt, Min, Max } from 'class-validator';
import { Type } from 'class-transformer';
import { ApiPropertyOptional } from '@nestjs/swagger';

export enum SortOrder {
  ASC = 'ASC',
  DESC = 'DESC',
}

export class QueryDto {
  // Pagination
  @ApiPropertyOptional({ default: 1 })
  @Type(() => Number)
  @IsInt()
  @Min(1)
  @IsOptional()
  page?: number = 1;

  @ApiPropertyOptional({ default: 10 })
  @Type(() => Number)
  @IsInt()
  @Min(1)
  @Max(100)
  @IsOptional()
  limit?: number = 10;

  // Sorting
  @ApiPropertyOptional({ default: 'createdAt' })
  @IsString()
  @IsOptional()
  sortBy?: string = 'createdAt';

  @ApiPropertyOptional({ enum: SortOrder, default: SortOrder.DESC })
  @IsEnum(SortOrder)
  @IsOptional()
  order?: SortOrder = SortOrder.DESC;

  // Searching
  @ApiPropertyOptional()
  @IsString()
  @IsOptional()
  search?: string;
}

// products/dto/find-products.dto.ts
export class FindProductsDto extends QueryDto {
  // Filtering
  @ApiPropertyOptional()
  @IsString()
  @IsOptional()
  category?: string;

  @ApiPropertyOptional()
  @Type(() => Number)
  @IsInt()
  @Min(0)
  @IsOptional()
  minPrice?: number;

  @ApiPropertyOptional()
  @Type(() => Number)
  @IsInt()
  @Min(0)
  @IsOptional()
  maxPrice?: number;

  @ApiPropertyOptional()
  @IsBoolean()
  @IsOptional()
  @Transform(({ value }) => value === 'true' || value === true)
  inStock?: boolean;

  @ApiPropertyOptional()
  @IsString()
  @IsOptional()
  brand?: string;

  @ApiPropertyOptional({ type: [String] })
  @IsOptional()
  @Transform(({ value }) => (typeof value === 'string' ? value.split(',') : value))
  @IsArray()
  @IsString({ each: true })
  tags?: string[];
}
```

**2. Controller Implementation:**

```typescript
// products/products.controller.ts
@Controller('products')
@ApiTags('products')
export class ProductsController {
  constructor(private readonly productsService: ProductsService) {}

  @Get()
  @ApiOperation({ summary: 'Get products with filtering, sorting, and searching' })
  @ApiQuery({ name: 'page', required: false, type: Number })
  @ApiQuery({ name: 'limit', required: false, type: Number })
  @ApiQuery({ name: 'sortBy', required: false, type: String })
  @ApiQuery({ name: 'order', required: false, enum: SortOrder })
  @ApiQuery({ name: 'search', required: false, type: String })
  @ApiQuery({ name: 'category', required: false, type: String })
  @ApiQuery({ name: 'minPrice', required: false, type: Number })
  @ApiQuery({ name: 'maxPrice', required: false, type: Number })
  @ApiQuery({ name: 'inStock', required: false, type: Boolean })
  @ApiQuery({ name: 'brand', required: false, type: String })
  @ApiQuery({ name: 'tags', required: false, type: String, description: 'Comma-separated tags' })
  async findAll(
    @Query() queryDto: FindProductsDto,
  ): Promise<PaginatedResponse<Product>> {
    return this.productsService.findAll(queryDto);
  }
}

// Request examples:
// GET /products?page=1&limit=20
// GET /products?search=laptop&category=electronics
// GET /products?minPrice=500&maxPrice=1500&inStock=true
// GET /products?sortBy=price&order=ASC
// GET /products?tags=gaming,rgb&brand=ASUS
// GET /products?search=laptop&category=electronics&minPrice=800&sortBy=price&order=DESC&page=2
```

**3. Service Implementation with TypeORM:**

```typescript
// products/products.service.ts
@Injectable()
export class ProductsService {
  constructor(
    @InjectRepository(Product)
    private productRepository: Repository<Product>,
  ) {}

  async findAll(queryDto: FindProductsDto): Promise<PaginatedResponse<Product>> {
    const {
      page = 1,
      limit = 10,
      sortBy = 'createdAt',
      order = SortOrder.DESC,
      search,
      category,
      minPrice,
      maxPrice,
      inStock,
      brand,
      tags,
    } = queryDto;

    const queryBuilder = this.productRepository.createQueryBuilder('product');

    // === SEARCHING ===
    if (search) {
      queryBuilder.andWhere(
        '(product.name ILIKE :search OR product.description ILIKE :search OR product.sku ILIKE :search)',
        { search: `%${search}%` },
      );
    }

    // === FILTERING ===
    if (category) {
      queryBuilder.andWhere('product.category = :category', { category });
    }

    if (brand) {
      queryBuilder.andWhere('product.brand = :brand', { brand });
    }

    if (minPrice !== undefined) {
      queryBuilder.andWhere('product.price >= :minPrice', { minPrice });
    }

    if (maxPrice !== undefined) {
      queryBuilder.andWhere('product.price <= :maxPrice', { maxPrice });
    }

    if (inStock !== undefined) {
      if (inStock) {
        queryBuilder.andWhere('product.stock > 0');
      } else {
        queryBuilder.andWhere('product.stock = 0');
      }
    }

    if (tags && tags.length > 0) {
      // For JSONB or array column
      queryBuilder.andWhere('product.tags && :tags', { tags });
      // OR for many-to-many relationship:
      // queryBuilder.innerJoin('product.tags', 'tag')
      //   .andWhere('tag.name IN (:...tags)', { tags });
    }

    // === SORTING ===
    // Validate sortBy to prevent SQL injection
    const allowedSortFields = ['name', 'price', 'createdAt', 'stock', 'rating'];
    const validSortBy = allowedSortFields.includes(sortBy) ? sortBy : 'createdAt';
    
    queryBuilder.orderBy(`product.${validSortBy}`, order);
    
    // Add secondary sort for consistent ordering
    if (validSortBy !== 'id') {
      queryBuilder.addOrderBy('product.id', 'ASC');
    }

    // === PAGINATION ===
    const skip = (page - 1) * limit;
    queryBuilder.skip(skip).take(limit);

    // Execute query
    const [data, total] = await queryBuilder.getManyAndCount();

    const totalPages = Math.ceil(total / limit);

    return {
      data,
      meta: {
        page,
        limit,
        total,
        totalPages,
        hasNextPage: page < totalPages,
        hasPrevPage: page > 1,
      },
    };
  }
}
```

**4. Advanced Filtering with Operators:**

```typescript
// Support advanced query operators
// GET /products?price[gte]=100&price[lte]=500&stock[gt]=0

export class AdvancedFilterDto {
  @IsOptional()
  price?: {
    gte?: number;  // greater than or equal
    lte?: number;  // less than or equal
    gt?: number;   // greater than
    lt?: number;   // less than
    eq?: number;   // equal
    ne?: number;   // not equal
  };

  @IsOptional()
  stock?: {
    gt?: number;
    gte?: number;
  };
}

// Parser for advanced filters
function applyAdvancedFilters(
  queryBuilder: SelectQueryBuilder<any>,
  filters: any,
  entityName: string,
) {
  Object.entries(filters).forEach(([field, operators]) => {
    if (operators && typeof operators === 'object') {
      Object.entries(operators).forEach(([operator, value]) => {
        switch (operator) {
          case 'gte':
            queryBuilder.andWhere(`${entityName}.${field} >= :${field}_gte`, {
              [`${field}_gte`]: value,
            });
            break;
          case 'lte':
            queryBuilder.andWhere(`${entityName}.${field} <= :${field}_lte`, {
              [`${field}_lte`]: value,
            });
            break;
          case 'gt':
            queryBuilder.andWhere(`${entityName}.${field} > :${field}_gt`, {
              [`${field}_gt`]: value,
            });
            break;
          case 'lt':
            queryBuilder.andWhere(`${entityName}.${field} < :${field}_lt`, {
              [`${field}_lt`]: value,
            });
            break;
          case 'eq':
            queryBuilder.andWhere(`${entityName}.${field} = :${field}_eq`, {
              [`${field}_eq`]: value,
            });
            break;
          case 'ne':
            queryBuilder.andWhere(`${entityName}.${field} != :${field}_ne`, {
              [`${field}_ne`]: value,
            });
            break;
        }
      });
    }
  });
}
```

**5. Full-Text Search:**

```typescript
// For PostgreSQL full-text search
async searchProducts(query: string): Promise<Product[]> {
  return this.productRepository
    .createQueryBuilder('product')
    .where(
      `to_tsvector('english', product.name || ' ' || product.description) @@ plainto_tsquery('english', :query)`,
      { query },
    )
    .orderBy(
      `ts_rank(to_tsvector('english', product.name || ' ' || product.description), plainto_tsquery('english', :query))`,
      'DESC',
    )
    .getMany();
}

// Add GIN index for better performance:
// CREATE INDEX idx_product_search ON products USING GIN (to_tsvector('english', name || ' ' || description));

// Or use TypeORM-specific search
@Entity()
export class Product {
  @Column()
  @Index({ fulltext: true })
  name: string;

  @Column('text')
  @Index({ fulltext: true })
  description: string;
}

// Search with relevance scoring
async searchWithRelevance(searchTerm: string) {
  return this.productRepository
    .createQueryBuilder('product')
    .select('product.*')
    .addSelect(
      `GREATEST(
        similarity(product.name, :searchTerm),
        similarity(product.description, :searchTerm)
      )`,
      'relevance',
    )
    .where(
      `product.name ILIKE :like OR product.description ILIKE :like`,
      { like: `%${searchTerm}%`, searchTerm },
    )
    .orderBy('relevance', 'DESC')
    .getRawMany();
}
```

**6. Date Range Filtering:**

```typescript
export class DateRangeDto {
  @IsOptional()
  @Type(() => Date)
  @IsDate()
  startDate?: Date;

  @IsOptional()
  @Type(() => Date)
  @IsDate()
  endDate?: Date;
}

// In service
if (startDate) {
  queryBuilder.andWhere('product.createdAt >= :startDate', { startDate });
}

if (endDate) {
  queryBuilder.andWhere('product.createdAt <= :endDate', { endDate });
}

// Request: GET /products?startDate=2025-01-01&endDate=2025-12-31
```

**7. Multiple Sort Fields:**

```typescript
// Support multiple sort fields
// GET /products?sort=-price,+name (- for DESC, + for ASC)

export class MultiSortDto {
  @IsOptional()
  @IsString()
  sort?: string; // Format: "-price,+name,createdAt"
}

// Parser
function applySorting(queryBuilder: SelectQueryBuilder<any>, sortString: string) {
  if (!sortString) return;

  const sortFields = sortString.split(',');
  sortFields.forEach((field) => {
    const trimmed = field.trim();
    if (trimmed.startsWith('-')) {
      queryBuilder.addOrderBy(`product.${trimmed.slice(1)}`, 'DESC');
    } else if (trimmed.startsWith('+')) {
      queryBuilder.addOrderBy(`product.${trimmed.slice(1)}`, 'ASC');
    } else {
      queryBuilder.addOrderBy(`product.${trimmed}`, 'ASC');
    }
  });
}
```

**8. Field Selection (Sparse Fieldsets):**

```typescript
// Allow clients to select specific fields
// GET /products?fields=id,name,price

export class FieldSelectionDto {
  @IsOptional()
  @IsString()
  fields?: string; // Comma-separated field names
}

// In service
if (fields) {
  const fieldArray = fields.split(',').map(f => f.trim());
  const allowedFields = ['id', 'name', 'price', 'description', 'category'];
  const validFields = fieldArray.filter(f => allowedFields.includes(f));
  
  if (validFields.length > 0) {
    queryBuilder.select(validFields.map(f => `product.${f}`));
  }
}
```

**9. Reusable Query Builder:**

```typescript
// common/builders/query-builder.service.ts
@Injectable()
export class QueryBuilderService {
  applyFilters<T>(
    qb: SelectQueryBuilder<T>,
    filters: Record<string, any>,
    entityName: string,
  ): SelectQueryBuilder<T> {
    Object.entries(filters).forEach(([key, value]) => {
      if (value !== undefined && value !== null && value !== '') {
        if (typeof value === 'string') {
          qb.andWhere(`${entityName}.${key} ILIKE :${key}`, { [key]: `%${value}%` });
        } else if (Array.isArray(value)) {
          qb.andWhere(`${entityName}.${key} IN (:...${key})`, { [key]: value });
        } else {
          qb.andWhere(`${entityName}.${key} = :${key}`, { [key]: value });
        }
      }
    });
    return qb;
  }

  applySorting<T>(
    qb: SelectQueryBuilder<T>,
    sortBy: string,
    order: 'ASC' | 'DESC',
    entityName: string,
    allowedFields: string[],
  ): SelectQueryBuilder<T> {
    if (allowedFields.includes(sortBy)) {
      qb.orderBy(`${entityName}.${sortBy}`, order);
    }
    return qb;
  }

  applyPagination<T>(
    qb: SelectQueryBuilder<T>,
    page: number,
    limit: number,
  ): SelectQueryBuilder<T> {
    qb.skip((page - 1) * limit).take(limit);
    return qb;
  }
}
```

**10. Performance Optimization:**

```typescript
// Add indexes for frequently filtered/sorted columns
@Entity()
@Index(['category', 'price']) // Composite index for common filter combination
@Index(['createdAt', 'id'])   // For pagination
export class Product {
  @Column()
  @Index() // Single column index
  category: string;

  @Column('decimal')
  @Index()
  price: number;

  @Column()
  @Index()
  brand: string;

  @Column()
  @Index({ fulltext: true }) // Full-text search index
  name: string;
}

// Use query result caching for expensive queries
async findAll(queryDto: FindProductsDto) {
  const cacheKey = JSON.stringify(queryDto);
  
  return this.productRepository
    .createQueryBuilder('product')
    // ... apply filters ...
    .cache(cacheKey, 60000) // Cache for 60 seconds
    .getManyAndCount();
}
```

**Key Takeaway:** Implement comprehensive querying with `QueryDto` extending pagination, add filtering with `andWhere()` conditions, sorting with validated `sortBy` and `order` parameters, searching with ILIKE for PostgreSQL or full-text search, support advanced operators (gte/lte/gt/lt), date ranges, multiple sort fields, and field selection. Use TypeORM QueryBuilder for dynamic queries, validate/whitelist fields to prevent SQL injection, add database indexes on filtered/sorted columns, and optionally cache expensive queries.

</details>

## Authentication & Authorization Systems

<details>
<summary><strong>7. How would you design a complete authentication system?</strong></summary>

**Answer:**

Design a complete authentication system with JWT tokens, refresh tokens, and proper security:

**1. User Entity & Database Schema:**

```typescript
// users/entities/user.entity.ts
import { Entity, Column, PrimaryGeneratedColumn, CreateDateColumn, UpdateDateColumn, Index } from 'typeorm';
import * as bcrypt from 'bcrypt';

@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  @Index()
  email: string;

  @Column()
  password: string; // Hashed password

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column({ default: false })
  isEmailVerified: boolean;

  @Column({ nullable: true })
  emailVerificationToken: string;

  @Column({ nullable: true })
  passwordResetToken: string;

  @Column({ nullable: true })
  passwordResetExpires: Date;

  @Column({ default: 'user' })
  role: string; // 'user', 'admin', 'moderator'

  @Column({ default: true })
  isActive: boolean;

  @Column({ nullable: true })
  lastLoginAt: Date;

  @Column({ nullable: true })
  refreshToken: string; // Hashed refresh token

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  // Method to hash password before saving
  async hashPassword(): Promise<void> {
    this.password = await bcrypt.hash(this.password, 10);
  }

  // Method to compare passwords
  async comparePassword(plainPassword: string): Promise<boolean> {
    return bcrypt.compare(plainPassword, this.password);
  }
}

// Database migration
// CREATE INDEX idx_users_email ON users(email);
// CREATE INDEX idx_users_email_verification_token ON users(email_verification_token);
// CREATE INDEX idx_users_password_reset_token ON users(password_reset_token);
```

**2. DTOs for Authentication:**

```typescript
// auth/dto/register.dto.ts
import { IsEmail, IsString, MinLength, MaxLength, Matches } from 'class-validator';
import { ApiProperty } from '@nestjs/swagger';

export class RegisterDto {
  @ApiProperty({ example: 'john.doe@example.com' })
  @IsEmail()
  email: string;

  @ApiProperty({ example: 'SecurePass123!' })
  @IsString()
  @MinLength(8)
  @MaxLength(50)
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/, {
    message: 'Password must contain uppercase, lowercase, number and special character',
  })
  password: string;

  @ApiProperty({ example: 'John' })
  @IsString()
  @MinLength(2)
  firstName: string;

  @ApiProperty({ example: 'Doe' })
  @IsString()
  @MinLength(2)
  lastName: string;
}

// auth/dto/login.dto.ts
export class LoginDto {
  @ApiProperty({ example: 'john.doe@example.com' })
  @IsEmail()
  email: string;

  @ApiProperty({ example: 'SecurePass123!' })
  @IsString()
  password: string;
}

// auth/dto/refresh-token.dto.ts
export class RefreshTokenDto {
  @ApiProperty()
  @IsString()
  refreshToken: string;
}
```

**3. JWT Strategy & Configuration:**

```typescript
// auth/strategies/jwt.strategy.ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';
import { ConfigService } from '@nestjs/config';
import { UsersService } from '../../users/users.service';

export interface JwtPayload {
  sub: string; // user ID
  email: string;
  role: string;
  type: 'access' | 'refresh';
}

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(
    private configService: ConfigService,
    private usersService: UsersService,
  ) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: configService.get<string>('JWT_SECRET'),
    });
  }

  async validate(payload: JwtPayload) {
    if (payload.type !== 'access') {
      throw new UnauthorizedException('Invalid token type');
    }

    const user = await this.usersService.findById(payload.sub);
    if (!user || !user.isActive) {
      throw new UnauthorizedException('User not found or inactive');
    }

    // Return user object (available as req.user in controllers)
    return {
      id: user.id,
      email: user.email,
      role: user.role,
      firstName: user.firstName,
      lastName: user.lastName,
    };
  }
}

// auth/strategies/jwt-refresh.strategy.ts
@Injectable()
export class JwtRefreshStrategy extends PassportStrategy(Strategy, 'jwt-refresh') {
  constructor(
    private configService: ConfigService,
    private usersService: UsersService,
  ) {
    super({
      jwtFromRequest: ExtractJwt.fromBodyField('refreshToken'),
      ignoreExpiration: false,
      secretOrKey: configService.get<string>('JWT_REFRESH_SECRET'),
      passReqToCallback: true, // Pass request to validate method
    });
  }

  async validate(req: Request, payload: JwtPayload) {
    if (payload.type !== 'refresh') {
      throw new UnauthorizedException('Invalid token type');
    }

    const refreshToken = req.body?.refreshToken;
    const user = await this.usersService.findById(payload.sub);
    
    if (!user || !user.refreshToken) {
      throw new UnauthorizedException('User not found');
    }

    // Verify refresh token matches stored hash
    const isValid = await bcrypt.compare(refreshToken, user.refreshToken);
    if (!isValid) {
      throw new UnauthorizedException('Invalid refresh token');
    }

    return { id: user.id, email: user.email, role: user.role };
  }
}
```

**4. Authentication Service:**

```typescript
// auth/auth.service.ts
import { Injectable, UnauthorizedException, ConflictException } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { ConfigService } from '@nestjs/config';
import { UsersService } from '../users/users.service';
import { RegisterDto } from './dto/register.dto';
import { LoginDto } from './dto/login.dto';
import * as bcrypt from 'bcrypt';

@Injectable()
export class AuthService {
  constructor(
    private usersService: UsersService,
    private jwtService: JwtService,
    private configService: ConfigService,
  ) {}

  // Register new user
  async register(registerDto: RegisterDto) {
    // Check if user already exists
    const existingUser = await this.usersService.findByEmail(registerDto.email);
    if (existingUser) {
      throw new ConflictException('Email already registered');
    }

    // Create user with hashed password
    const user = await this.usersService.create({
      ...registerDto,
      password: await bcrypt.hash(registerDto.password, 10),
    });

    // Generate email verification token
    const verificationToken = this.generateVerificationToken();
    await this.usersService.update(user.id, {
      emailVerificationToken: verificationToken,
    });

    // Send verification email (async, don't wait)
    this.sendVerificationEmail(user.email, verificationToken).catch(err =>
      console.error('Failed to send verification email:', err),
    );

    // Generate tokens
    const tokens = await this.generateTokens(user);

    return {
      user: {
        id: user.id,
        email: user.email,
        firstName: user.firstName,
        lastName: user.lastName,
        role: user.role,
      },
      ...tokens,
    };
  }

  // Login user
  async login(loginDto: LoginDto) {
    // Find user by email
    const user = await this.usersService.findByEmail(loginDto.email);
    if (!user) {
      throw new UnauthorizedException('Invalid credentials');
    }

    // Check if user is active
    if (!user.isActive) {
      throw new UnauthorizedException('Account is deactivated');
    }

    // Verify password
    const isPasswordValid = await bcrypt.compare(loginDto.password, user.password);
    if (!isPasswordValid) {
      throw new UnauthorizedException('Invalid credentials');
    }

    // Update last login
    await this.usersService.update(user.id, { lastLoginAt: new Date() });

    // Generate tokens
    const tokens = await this.generateTokens(user);

    return {
      user: {
        id: user.id,
        email: user.email,
        firstName: user.firstName,
        lastName: user.lastName,
        role: user.role,
        isEmailVerified: user.isEmailVerified,
      },
      ...tokens,
    };
  }

  // Refresh access token
  async refreshTokens(userId: string) {
    const user = await this.usersService.findById(userId);
    if (!user) {
      throw new UnauthorizedException('User not found');
    }

    return this.generateTokens(user);
  }

  // Logout user (invalidate refresh token)
  async logout(userId: string) {
    await this.usersService.update(userId, { refreshToken: null });
    return { message: 'Logged out successfully' };
  }

  // Generate access and refresh tokens
  private async generateTokens(user: User) {
    const payload: JwtPayload = {
      sub: user.id,
      email: user.email,
      role: user.role,
      type: 'access',
    };

    const refreshPayload: JwtPayload = {
      ...payload,
      type: 'refresh',
    };

    const [accessToken, refreshToken] = await Promise.all([
      this.jwtService.signAsync(payload, {
        secret: this.configService.get<string>('JWT_SECRET'),
        expiresIn: '15m', // Short-lived access token
      }),
      this.jwtService.signAsync(refreshPayload, {
        secret: this.configService.get<string>('JWT_REFRESH_SECRET'),
        expiresIn: '7d', // Long-lived refresh token
      }),
    ]);

    // Store hashed refresh token in database
    const hashedRefreshToken = await bcrypt.hash(refreshToken, 10);
    await this.usersService.update(user.id, { refreshToken: hashedRefreshToken });

    return {
      accessToken,
      refreshToken,
      expiresIn: 900, // 15 minutes in seconds
      tokenType: 'Bearer',
    };
  }

  // Generate email verification token
  private generateVerificationToken(): string {
    return require('crypto').randomBytes(32).toString('hex');
  }

  // Send verification email (integrate with email service)
  private async sendVerificationEmail(email: string, token: string) {
    const verificationUrl = `${this.configService.get('APP_URL')}/auth/verify-email?token=${token}`;
    // Send email with verificationUrl
    // await this.emailService.send({ to: email, template: 'verify-email', data: { verificationUrl } });
  }
}
```

**5. Auth Controller:**

```typescript
// auth/auth.controller.ts
import { Controller, Post, Body, UseGuards, Request, HttpCode, HttpStatus } from '@nestjs/common';
import { ApiTags, ApiOperation, ApiBearerAuth } from '@nestjs/swagger';
import { AuthService } from './auth.service';
import { RegisterDto } from './dto/register.dto';
import { LoginDto } from './dto/login.dto';
import { RefreshTokenDto } from './dto/refresh-token.dto';
import { JwtAuthGuard } from './guards/jwt-auth.guard';
import { JwtRefreshGuard } from './guards/jwt-refresh.guard';

@ApiTags('auth')
@Controller('auth')
export class AuthController {
  constructor(private readonly authService: AuthService) {}

  @Post('register')
  @HttpCode(HttpStatus.CREATED)
  @ApiOperation({ summary: 'Register a new user' })
  async register(@Body() registerDto: RegisterDto) {
    return this.authService.register(registerDto);
  }

  @Post('login')
  @HttpCode(HttpStatus.OK)
  @ApiOperation({ summary: 'Login user' })
  async login(@Body() loginDto: LoginDto) {
    return this.authService.login(loginDto);
  }

  @Post('refresh')
  @UseGuards(JwtRefreshGuard)
  @HttpCode(HttpStatus.OK)
  @ApiOperation({ summary: 'Refresh access token' })
  async refresh(@Request() req, @Body() refreshTokenDto: RefreshTokenDto) {
    return this.authService.refreshTokens(req.user.id);
  }

  @Post('logout')
  @UseGuards(JwtAuthGuard)
  @ApiBearerAuth()
  @HttpCode(HttpStatus.OK)
  @ApiOperation({ summary: 'Logout user' })
  async logout(@Request() req) {
    return this.authService.logout(req.user.id);
  }

  @Post('me')
  @UseGuards(JwtAuthGuard)
  @ApiBearerAuth()
  @ApiOperation({ summary: 'Get current user' })
  async getProfile(@Request() req) {
    return req.user;
  }
}
```

**6. Auth Guards:**

```typescript
// auth/guards/jwt-auth.guard.ts
import { Injectable, ExecutionContext } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';
import { Reflector } from '@nestjs/core';

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  constructor(private reflector: Reflector) {
    super();
  }

  canActivate(context: ExecutionContext) {
    // Check if route is marked as public
    const isPublic = this.reflector.get<boolean>('isPublic', context.getHandler());
    if (isPublic) {
      return true;
    }

    return super.canActivate(context);
  }
}

// auth/guards/jwt-refresh.guard.ts
@Injectable()
export class JwtRefreshGuard extends AuthGuard('jwt-refresh') {}

// auth/decorators/public.decorator.ts
import { SetMetadata } from '@nestjs/common';

export const Public = () => SetMetadata('isPublic', true);

// Usage:
// @Public()
// @Get('public-route')
// async publicRoute() { ... }
```

**7. Auth Module Configuration:**

```typescript
// auth/auth.module.ts
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { PassportModule } from '@nestjs/passport';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { AuthService } from './auth.service';
import { AuthController } from './auth.controller';
import { UsersModule } from '../users/users.module';
import { JwtStrategy } from './strategies/jwt.strategy';
import { JwtRefreshStrategy } from './strategies/jwt-refresh.strategy';

@Module({
  imports: [
    UsersModule,
    PassportModule,
    JwtModule.registerAsync({
      imports: [ConfigModule],
      useFactory: async (configService: ConfigService) => ({
        secret: configService.get<string>('JWT_SECRET'),
        signOptions: {
          expiresIn: '15m',
        },
      }),
      inject: [ConfigService],
    }),
  ],
  controllers: [AuthController],
  providers: [AuthService, JwtStrategy, JwtRefreshStrategy],
  exports: [AuthService],
})
export class AuthModule {}
```

**8. Environment Configuration:**

```bash
# .env
JWT_SECRET=your-super-secret-jwt-key-change-in-production
JWT_REFRESH_SECRET=your-super-secret-refresh-key-change-in-production
APP_URL=http://localhost:3000
```

**9. Global Auth Guard (Optional):**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { APP_GUARD } from '@nestjs/core';
import { JwtAuthGuard } from './auth/guards/jwt-auth.guard';

@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: JwtAuthGuard, // All routes protected by default
    },
  ],
})
export class AppModule {}

// Now use @Public() decorator to make routes public
```

**10. Usage in Protected Routes:**

```typescript
// products/products.controller.ts
@Controller('products')
export class ProductsController {
  @Get()
  @Public() // Public route - no auth required
  async findAll() {
    return this.productsService.findAll();
  }

  @Post()
  @UseGuards(JwtAuthGuard) // Protected route - requires JWT
  async create(@Request() req, @Body() createDto: CreateProductDto) {
    // req.user contains authenticated user data
    return this.productsService.create(createDto, req.user.id);
  }

  @Delete(':id')
  @UseGuards(JwtAuthGuard)
  @Roles('admin') // Only admin can delete
  async remove(@Param('id') id: string) {
    return this.productsService.remove(id);
  }
}
```

**Key Takeaway:** Complete auth system uses JWT for access tokens (15min expiry), refresh tokens (7 days) stored as hashed values, bcrypt for password hashing (10 rounds), Passport JWT strategy for authentication, guards (@UseGuards(JwtAuthGuard)) for route protection, email verification flow, and proper token refresh mechanism. Store refresh tokens hashed in database, validate token type in strategies, implement logout by clearing refresh tokens, and use @Public() decorator for non-protected routes.

</details>

<details>
<summary><strong>8. How do you implement multi-factor authentication (MFA)?</strong></summary>

**Answer:**

Implement MFA using TOTP (Time-based One-Time Password) with authenticator apps:

**1. Install Dependencies:**

```bash
npm install otplib qrcode
npm install -D @types/qrcode
```

**2. Extend User Entity:**

```typescript
// users/entities/user.entity.ts
@Entity('users')
export class User {
  // ... existing fields ...

  @Column({ default: false })
  isMfaEnabled: boolean;

  @Column({ nullable: true })
  mfaSecret: string; // Encrypted TOTP secret

  @Column({ type: 'simple-array', nullable: true })
  mfaBackupCodes: string[]; // Hashed backup codes

  @Column({ nullable: true })
  mfaEnabledAt: Date;
}
```

**3. MFA DTOs:**

```typescript
// auth/dto/enable-mfa.dto.ts
export class EnableMfaDto {
  @ApiProperty()
  @IsString()
  @Length(6, 6)
  code: string; // 6-digit TOTP code to verify setup
}

// auth/dto/verify-mfa.dto.ts
export class VerifyMfaDto {
  @ApiProperty()
  @IsEmail()
  email: string;

  @ApiProperty()
  @IsString()
  password: string;

  @ApiProperty()
  @IsString()
  @Length(6, 6)
  mfaCode: string; // 6-digit TOTP code
}

// auth/dto/disable-mfa.dto.ts
export class DisableMfaDto {
  @ApiProperty()
  @IsString()
  password: string; // Require password to disable MFA

  @ApiProperty()
  @IsString()
  @Length(6, 6)
  mfaCode: string; // Current valid TOTP code
}

// auth/dto/use-backup-code.dto.ts
export class UseBackupCodeDto {
  @ApiProperty()
  @IsEmail()
  email: string;

  @ApiProperty()
  @IsString()
  password: string;

  @ApiProperty()
  @IsString()
  backupCode: string; // One of the backup codes
}
```

**4. MFA Service:**

```typescript
// auth/mfa.service.ts
import { Injectable, UnauthorizedException, BadRequestException } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { authenticator } from 'otplib';
import * as qrcode from 'qrcode';
import * as crypto from 'crypto';
import * as bcrypt from 'bcrypt';
import { UsersService } from '../users/users.service';

@Injectable()
export class MfaService {
  constructor(
    private usersService: UsersService,
    private configService: ConfigService,
  ) {
    // Configure TOTP settings
    authenticator.options = {
      window: 1, // Allow 1 step before/after for clock skew (30 seconds window)
    };
  }

  // Generate MFA secret and QR code
  async generateMfaSecret(userId: string) {
    const user = await this.usersService.findById(userId);
    if (!user) {
      throw new UnauthorizedException('User not found');
    }

    if (user.isMfaEnabled) {
      throw new BadRequestException('MFA is already enabled');
    }

    // Generate secret
    const secret = authenticator.generateSecret();

    // Create TOTP URL for QR code
    const appName = this.configService.get<string>('APP_NAME', 'MyApp');
    const otpauthUrl = authenticator.keyuri(user.email, appName, secret);

    // Generate QR code as data URL
    const qrCodeDataUrl = await qrcode.toDataURL(otpauthUrl);

    // Store secret temporarily (encrypted)
    const encryptedSecret = this.encryptSecret(secret);
    await this.usersService.update(userId, { mfaSecret: encryptedSecret });

    return {
      secret, // Show to user once (for manual entry)
      qrCode: qrCodeDataUrl, // QR code image
      backupCodes: null, // Will be generated after verification
    };
  }

  // Enable MFA after verifying setup code
  async enableMfa(userId: string, code: string) {
    const user = await this.usersService.findById(userId);
    if (!user || !user.mfaSecret) {
      throw new BadRequestException('MFA setup not initiated');
    }

    // Decrypt secret
    const secret = this.decryptSecret(user.mfaSecret);

    // Verify the code
    const isValid = authenticator.verify({ token: code, secret });
    if (!isValid) {
      throw new UnauthorizedException('Invalid MFA code');
    }

    // Generate backup codes
    const backupCodes = this.generateBackupCodes();
    const hashedBackupCodes = await Promise.all(
      backupCodes.map(code => bcrypt.hash(code, 10)),
    );

    // Enable MFA
    await this.usersService.update(userId, {
      isMfaEnabled: true,
      mfaBackupCodes: hashedBackupCodes,
      mfaEnabledAt: new Date(),
    });

    return {
      message: 'MFA enabled successfully',
      backupCodes, // Show to user once - they should save these
    };
  }

  // Verify MFA code during login
  async verifyMfaCode(userId: string, code: string): Promise<boolean> {
    const user = await this.usersService.findById(userId);
    if (!user || !user.isMfaEnabled || !user.mfaSecret) {
      throw new UnauthorizedException('MFA not enabled');
    }

    // Decrypt secret
    const secret = this.decryptSecret(user.mfaSecret);

    // Verify code
    return authenticator.verify({ token: code, secret });
  }

  // Verify backup code
  async verifyBackupCode(userId: string, backupCode: string): Promise<boolean> {
    const user = await this.usersService.findById(userId);
    if (!user || !user.isMfaEnabled || !user.mfaBackupCodes) {
      throw new UnauthorizedException('MFA not enabled');
    }

    // Check each hashed backup code
    for (let i = 0; i < user.mfaBackupCodes.length; i++) {
      const isMatch = await bcrypt.compare(backupCode, user.mfaBackupCodes[i]);
      if (isMatch) {
        // Remove used backup code
        const updatedCodes = [...user.mfaBackupCodes];
        updatedCodes.splice(i, 1);
        await this.usersService.update(userId, { mfaBackupCodes: updatedCodes });
        return true;
      }
    }

    return false;
  }

  // Disable MFA
  async disableMfa(userId: string, password: string, code: string) {
    const user = await this.usersService.findById(userId);
    if (!user || !user.isMfaEnabled) {
      throw new BadRequestException('MFA is not enabled');
    }

    // Verify password
    const isPasswordValid = await bcrypt.compare(password, user.password);
    if (!isPasswordValid) {
      throw new UnauthorizedException('Invalid password');
    }

    // Verify MFA code
    const isCodeValid = await this.verifyMfaCode(userId, code);
    if (!isCodeValid) {
      throw new UnauthorizedException('Invalid MFA code');
    }

    // Disable MFA
    await this.usersService.update(userId, {
      isMfaEnabled: false,
      mfaSecret: null,
      mfaBackupCodes: null,
      mfaEnabledAt: null,
    });

    return { message: 'MFA disabled successfully' };
  }

  // Generate 10 backup codes
  private generateBackupCodes(): string[] {
    const codes: string[] = [];
    for (let i = 0; i < 10; i++) {
      // Generate 8-character alphanumeric codes
      const code = crypto.randomBytes(4).toString('hex').toUpperCase();
      codes.push(code);
    }
    return codes;
  }

  // Encrypt MFA secret for storage
  private encryptSecret(secret: string): string {
    const algorithm = 'aes-256-cbc';
    const key = Buffer.from(this.configService.get<string>('ENCRYPTION_KEY'), 'hex');
    const iv = crypto.randomBytes(16);
    
    const cipher = crypto.createCipheriv(algorithm, key, iv);
    let encrypted = cipher.update(secret, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    return iv.toString('hex') + ':' + encrypted;
  }

  // Decrypt MFA secret
  private decryptSecret(encryptedSecret: string): string {
    const algorithm = 'aes-256-cbc';
    const key = Buffer.from(this.configService.get<string>('ENCRYPTION_KEY'), 'hex');
    
    const parts = encryptedSecret.split(':');
    const iv = Buffer.from(parts[0], 'hex');
    const encrypted = parts[1];
    
    const decipher = crypto.createDecipheriv(algorithm, key, iv);
    let decrypted = decipher.update(encrypted, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    
    return decrypted;
  }
}
```

**5. Modified Auth Service for MFA Login:**

```typescript
// auth/auth.service.ts
@Injectable()
export class AuthService {
  constructor(
    private usersService: UsersService,
    private jwtService: JwtService,
    private mfaService: MfaService,
    private configService: ConfigService,
  ) {}

  // Modified login to check for MFA
  async login(loginDto: LoginDto) {
    const user = await this.usersService.findByEmail(loginDto.email);
    if (!user || !user.isActive) {
      throw new UnauthorizedException('Invalid credentials');
    }

    const isPasswordValid = await bcrypt.compare(loginDto.password, user.password);
    if (!isPasswordValid) {
      throw new UnauthorizedException('Invalid credentials');
    }

    // If MFA is enabled, require MFA code
    if (user.isMfaEnabled) {
      return {
        requiresMfa: true,
        tempToken: await this.generateTempToken(user.id), // Temporary token for MFA step
        message: 'Please provide MFA code',
      };
    }

    // No MFA - proceed with normal login
    await this.usersService.update(user.id, { lastLoginAt: new Date() });
    const tokens = await this.generateTokens(user);

    return {
      user: {
        id: user.id,
        email: user.email,
        firstName: user.firstName,
        lastName: user.lastName,
        role: user.role,
      },
      ...tokens,
    };
  }

  // Complete login with MFA code
  async loginWithMfa(email: string, password: string, mfaCode: string) {
    const user = await this.usersService.findByEmail(email);
    if (!user || !user.isActive) {
      throw new UnauthorizedException('Invalid credentials');
    }

    const isPasswordValid = await bcrypt.compare(password, user.password);
    if (!isPasswordValid) {
      throw new UnauthorizedException('Invalid credentials');
    }

    if (!user.isMfaEnabled) {
      throw new BadRequestException('MFA is not enabled for this account');
    }

    // Verify MFA code
    const isMfaValid = await this.mfaService.verifyMfaCode(user.id, mfaCode);
    if (!isMfaValid) {
      throw new UnauthorizedException('Invalid MFA code');
    }

    // MFA successful - generate tokens
    await this.usersService.update(user.id, { lastLoginAt: new Date() });
    const tokens = await this.generateTokens(user);

    return {
      user: {
        id: user.id,
        email: user.email,
        firstName: user.firstName,
        lastName: user.lastName,
        role: user.role,
      },
      ...tokens,
    };
  }

  // Login with backup code
  async loginWithBackupCode(email: string, password: string, backupCode: string) {
    const user = await this.usersService.findByEmail(email);
    if (!user || !user.isActive) {
      throw new UnauthorizedException('Invalid credentials');
    }

    const isPasswordValid = await bcrypt.compare(password, user.password);
    if (!isPasswordValid) {
      throw new UnauthorizedException('Invalid credentials');
    }

    // Verify backup code
    const isBackupCodeValid = await this.mfaService.verifyBackupCode(user.id, backupCode);
    if (!isBackupCodeValid) {
      throw new UnauthorizedException('Invalid backup code');
    }

    // Backup code successful - generate tokens
    await this.usersService.update(user.id, { lastLoginAt: new Date() });
    const tokens = await this.generateTokens(user);

    return {
      user: {
        id: user.id,
        email: user.email,
        firstName: user.firstName,
        lastName: user.lastName,
        role: user.role,
      },
      ...tokens,
      warning: 'Backup code used. Please regenerate backup codes.',
    };
  }

  // Generate temporary token for MFA step
  private async generateTempToken(userId: string): Promise<string> {
    return this.jwtService.signAsync(
      { sub: userId, type: 'mfa-temp' },
      { secret: this.configService.get('JWT_SECRET'), expiresIn: '5m' },
    );
  }
}
```

**6. MFA Controller:**

```typescript
// auth/mfa.controller.ts
@Controller('auth/mfa')
@ApiTags('auth-mfa')
export class MfaController {
  constructor(
    private readonly authService: AuthService,
    private readonly mfaService: MfaService,
  ) {}

  @Post('setup')
  @UseGuards(JwtAuthGuard)
  @ApiBearerAuth()
  @ApiOperation({ summary: 'Initiate MFA setup - get QR code' })
  async setupMfa(@Request() req) {
    return this.mfaService.generateMfaSecret(req.user.id);
  }

  @Post('enable')
  @UseGuards(JwtAuthGuard)
  @ApiBearerAuth()
  @ApiOperation({ summary: 'Enable MFA after verifying setup code' })
  async enableMfa(@Request() req, @Body() enableMfaDto: EnableMfaDto) {
    return this.mfaService.enableMfa(req.user.id, enableMfaDto.code);
  }

  @Post('disable')
  @UseGuards(JwtAuthGuard)
  @ApiBearerAuth()
  @ApiOperation({ summary: 'Disable MFA' })
  async disableMfa(@Request() req, @Body() disableMfaDto: DisableMfaDto) {
    return this.mfaService.disableMfa(
      req.user.id,
      disableMfaDto.password,
      disableMfaDto.code,
    );
  }

  @Post('verify')
  @HttpCode(HttpStatus.OK)
  @ApiOperation({ summary: 'Complete login with MFA code' })
  async verifyMfa(@Body() verifyMfaDto: VerifyMfaDto) {
    return this.authService.loginWithMfa(
      verifyMfaDto.email,
      verifyMfaDto.password,
      verifyMfaDto.mfaCode,
    );
  }

  @Post('backup-code')
  @HttpCode(HttpStatus.OK)
  @ApiOperation({ summary: 'Login with backup code' })
  async useBackupCode(@Body() useBackupCodeDto: UseBackupCodeDto) {
    return this.authService.loginWithBackupCode(
      useBackupCodeDto.email,
      useBackupCodeDto.password,
      useBackupCodeDto.backupCode,
    );
  }
}
```

**7. Environment Variables:**

```bash
# .env
ENCRYPTION_KEY=your-32-byte-hex-key-for-mfa-secret-encryption
APP_NAME=MyApp
```

**Generate encryption key:**
```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

**8. MFA Flow:**

```
1. User enables MFA:
   POST /auth/mfa/setup → Returns QR code and secret
   User scans QR code with authenticator app (Google Authenticator, Authy, etc.)
   POST /auth/mfa/enable { code: "123456" } → Verifies setup, returns backup codes
   User saves backup codes securely

2. User logs in with MFA:
   POST /auth/login { email, password } → Returns { requiresMfa: true }
   POST /auth/mfa/verify { email, password, mfaCode: "123456" } → Returns access/refresh tokens

3. User lost device (use backup code):
   POST /auth/mfa/backup-code { email, password, backupCode: "ABC123" } → Returns tokens
   
4. User disables MFA:
   POST /auth/mfa/disable { password, mfaCode: "123456" } → Disables MFA
```

**Key Takeaway:** Implement MFA using otplib for TOTP generation, qrcode for QR code generation, store encrypted MFA secrets in database, generate and hash backup codes (10 codes), modify login flow to check isMfaEnabled flag, require 6-digit code verification after password validation, support backup codes for account recovery, and allow MFA disable with password + valid code. Use authenticator.verify() with 30-second window for clock skew tolerance.

</details>

<details>
<summary><strong>9. How do you handle password reset flow?</strong></summary>

**Answer:**

Implement secure password reset with token-based verification:

**1. Password Reset DTOs:**

```typescript
// auth/dto/forgot-password.dto.ts
export class ForgotPasswordDto {
  @ApiProperty({ example: 'user@example.com' })
  @IsEmail()
  email: string;
}

// auth/dto/reset-password.dto.ts
export class ResetPasswordDto {
  @ApiProperty()
  @IsString()
  token: string;

  @ApiProperty({ example: 'NewSecurePass123!' })
  @IsString()
  @MinLength(8)
  @MaxLength(50)
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/, {
    message: 'Password must contain uppercase, lowercase, number and special character',
  })
  newPassword: string;

  @ApiProperty()
  @IsString()
  confirmPassword: string;

  @Validate(MatchesProperty, ['newPassword'])
  get passwordsMatch() {
    return this.newPassword === this.confirmPassword;
  }
}
```

**2. Password Reset Service:**

```typescript
// auth/password-reset.service.ts
import { Injectable, BadRequestException, NotFoundException } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import * as crypto from 'crypto';
import * as bcrypt from 'bcrypt';
import { UsersService } from '../users/users.service';
import { EmailService } from '../email/email.service';

@Injectable()
export class PasswordResetService {
  constructor(
    private usersService: UsersService,
    private emailService: EmailService,
    private configService: ConfigService,
  ) {}

  // Request password reset
  async requestPasswordReset(email: string): Promise<{ message: string }> {
    const user = await this.usersService.findByEmail(email);
    
    // Don't reveal if email exists (security best practice)
    if (!user) {
      return {
        message: 'If an account with that email exists, a password reset link has been sent.',
      };
    }

    // Check if user is active
    if (!user.isActive) {
      return {
        message: 'If an account with that email exists, a password reset link has been sent.',
      };
    }

    // Generate secure token
    const resetToken = this.generateResetToken();
    const resetTokenHash = await bcrypt.hash(resetToken, 10);
    
    // Set token expiration (1 hour from now)
    const expiresAt = new Date();
    expiresAt.setHours(expiresAt.getHours() + 1);

    // Save token and expiration to database
    await this.usersService.update(user.id, {
      passwordResetToken: resetTokenHash,
      passwordResetExpires: expiresAt,
    });

    // Send reset email (async, don't wait)
    const resetUrl = `${this.configService.get('FRONTEND_URL')}/reset-password?token=${resetToken}&email=${email}`;
    
    this.emailService
      .sendPasswordResetEmail(user.email, resetUrl, user.firstName)
      .catch(err => console.error('Failed to send password reset email:', err));

    return {
      message: 'If an account with that email exists, a password reset link has been sent.',
    };
  }

  // Reset password with token
  async resetPassword(
    email: string,
    token: string,
    newPassword: string,
  ): Promise<{ message: string }> {
    const user = await this.usersService.findByEmail(email);
    
    if (!user || !user.passwordResetToken || !user.passwordResetExpires) {
      throw new BadRequestException('Invalid or expired reset token');
    }

    // Check if token has expired
    if (new Date() > user.passwordResetExpires) {
      // Clean up expired token
      await this.usersService.update(user.id, {
        passwordResetToken: null,
        passwordResetExpires: null,
      });
      throw new BadRequestException('Reset token has expired');
    }

    // Verify token
    const isTokenValid = await bcrypt.compare(token, user.passwordResetToken);
    if (!isTokenValid) {
      throw new BadRequestException('Invalid reset token');
    }

    // Check if new password is different from old password
    const isSamePassword = await bcrypt.compare(newPassword, user.password);
    if (isSamePassword) {
      throw new BadRequestException('New password must be different from old password');
    }

    // Hash new password
    const hashedPassword = await bcrypt.hash(newPassword, 10);

    // Update password and clear reset token
    await this.usersService.update(user.id, {
      password: hashedPassword,
      passwordResetToken: null,
      passwordResetExpires: null,
      refreshToken: null, // Invalidate all sessions
    });

    // Send confirmation email
    this.emailService
      .sendPasswordChangedEmail(user.email, user.firstName)
      .catch(err => console.error('Failed to send password changed email:', err));

    return {
      message: 'Password has been reset successfully. Please login with your new password.',
    };
  }

  // Validate reset token (for frontend to check before showing form)
  async validateResetToken(email: string, token: string): Promise<{ valid: boolean }> {
    const user = await this.usersService.findByEmail(email);
    
    if (!user || !user.passwordResetToken || !user.passwordResetExpires) {
      return { valid: false };
    }

    // Check expiration
    if (new Date() > user.passwordResetExpires) {
      return { valid: false };
    }

    // Verify token
    const isValid = await bcrypt.compare(token, user.passwordResetToken);
    return { valid: isValid };
  }

  // Generate secure random token
  private generateResetToken(): string {
    return crypto.randomBytes(32).toString('hex');
  }
}
```

**3. Password Reset Controller:**

```typescript
// auth/auth.controller.ts (add these endpoints)
@Controller('auth')
export class AuthController {
  constructor(
    private readonly authService: AuthService,
    private readonly passwordResetService: PasswordResetService,
  ) {}

  @Post('forgot-password')
  @HttpCode(HttpStatus.OK)
  @ApiOperation({ summary: 'Request password reset' })
  @ApiResponse({ status: 200, description: 'Password reset email sent if account exists' })
  async forgotPassword(@Body() forgotPasswordDto: ForgotPasswordDto) {
    return this.passwordResetService.requestPasswordReset(forgotPasswordDto.email);
  }

  @Post('reset-password')
  @HttpCode(HttpStatus.OK)
  @ApiOperation({ summary: 'Reset password with token' })
  @ApiResponse({ status: 200, description: 'Password reset successful' })
  @ApiResponse({ status: 400, description: 'Invalid or expired token' })
  async resetPassword(@Body() resetPasswordDto: ResetPasswordDto) {
    return this.passwordResetService.resetPassword(
      resetPasswordDto.email,
      resetPasswordDto.token,
      resetPasswordDto.newPassword,
    );
  }

  @Get('reset-password/validate')
  @ApiOperation({ summary: 'Validate reset token' })
  @ApiQuery({ name: 'email', type: String })
  @ApiQuery({ name: 'token', type: String })
  async validateResetToken(
    @Query('email') email: string,
    @Query('token') token: string,
  ) {
    return this.passwordResetService.validateResetToken(email, token);
  }
}
```

**4. Email Templates:**

```typescript
// email/email.service.ts
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import * as nodemailer from 'nodemailer';

@Injectable()
export class EmailService {
  private transporter: nodemailer.Transporter;

  constructor(private configService: ConfigService) {
    this.transporter = nodemailer.createTransport({
      host: configService.get('SMTP_HOST'),
      port: configService.get('SMTP_PORT'),
      secure: false,
      auth: {
        user: configService.get('SMTP_USER'),
        pass: configService.get('SMTP_PASS'),
      },
    });
  }

  async sendPasswordResetEmail(to: string, resetUrl: string, userName: string) {
    const html = `
      <!DOCTYPE html>
      <html>
        <head>
          <style>
            body { font-family: Arial, sans-serif; line-height: 1.6; }
            .container { max-width: 600px; margin: 0 auto; padding: 20px; }
            .button {
              display: inline-block;
              padding: 12px 24px;
              background-color: #007bff;
              color: #ffffff;
              text-decoration: none;
              border-radius: 4px;
              margin: 20px 0;
            }
            .footer { margin-top: 30px; font-size: 12px; color: #666; }
          </style>
        </head>
        <body>
          <div class="container">
            <h2>Password Reset Request</h2>
            <p>Hi ${userName},</p>
            <p>We received a request to reset your password. Click the button below to reset it:</p>
            <a href="${resetUrl}" class="button">Reset Password</a>
            <p>Or copy and paste this link into your browser:</p>
            <p><a href="${resetUrl}">${resetUrl}</a></p>
            <p><strong>This link will expire in 1 hour.</strong></p>
            <p>If you didn't request a password reset, please ignore this email or contact support if you have concerns.</p>
            <div class="footer">
              <p>This is an automated email, please do not reply.</p>
            </div>
          </div>
        </body>
      </html>
    `;

    await this.transporter.sendMail({
      from: this.configService.get('SMTP_FROM'),
      to,
      subject: 'Password Reset Request',
      html,
    });
  }

  async sendPasswordChangedEmail(to: string, userName: string) {
    const html = `
      <!DOCTYPE html>
      <html>
        <body>
          <div style="font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto; padding: 20px;">
            <h2>Password Changed Successfully</h2>
            <p>Hi ${userName},</p>
            <p>Your password has been changed successfully.</p>
            <p>If you didn't make this change, please contact our support team immediately.</p>
            <p style="margin-top: 30px; font-size: 12px; color: #666;">
              This is an automated email, please do not reply.
            </p>
          </div>
        </body>
      </html>
    `;

    await this.transporter.sendMail({
      from: this.configService.get('SMTP_FROM'),
      to,
      subject: 'Password Changed Successfully',
      html,
    });
  }
}
```

**5. Rate Limiting (Prevent Abuse):**

```typescript
// Install: npm install @nestjs/throttler

// app.module.ts
import { ThrottlerModule } from '@nestjs/throttler';

@Module({
  imports: [
    ThrottlerModule.forRoot({
      ttl: 60, // Time to live in seconds
      limit: 3, // Maximum number of requests within TTL
    }),
  ],
})
export class AppModule {}

// auth.controller.ts
import { Throttle } from '@nestjs/throttler';

@Post('forgot-password')
@Throttle(3, 60) // 3 requests per 60 seconds
async forgotPassword(@Body() forgotPasswordDto: ForgotPasswordDto) {
  return this.passwordResetService.requestPasswordReset(forgotPasswordDto.email);
}
```

**6. Complete Flow:**

```typescript
/*
PASSWORD RESET FLOW:

1. User requests password reset:
   POST /auth/forgot-password { email: "user@example.com" }
   → Generates token, saves hash + expiration to DB
   → Sends email with reset link
   → Returns generic message (don't reveal if email exists)

2. User clicks link in email:
   GET /auth/reset-password/validate?email=user@example.com&token=abc123
   → Frontend validates token before showing form
   → Returns { valid: true/false }

3. User submits new password:
   POST /auth/reset-password {
     email: "user@example.com",
     token: "abc123",
     newPassword: "NewPass123!",
     confirmPassword: "NewPass123!"
   }
   → Validates token and expiration
   → Checks new password != old password
   → Updates password, clears token
   → Invalidates all existing sessions (refresh tokens)
   → Sends confirmation email

4. User logs in with new password:
   POST /auth/login { email, password: "NewPass123!" }
   → Normal login flow
*/
```

**7. Security Best Practices:**

```typescript
// Additional security measures

// 1. Log password reset attempts
@Injectable()
export class AuditLogService {
  async logPasswordResetRequest(email: string, ip: string, userAgent: string) {
    // Log to database or monitoring service
    await this.auditLogRepository.save({
      action: 'PASSWORD_RESET_REQUEST',
      email,
      ip,
      userAgent,
      timestamp: new Date(),
    });
  }

  async logPasswordResetSuccess(userId: string, ip: string) {
    await this.auditLogRepository.save({
      action: 'PASSWORD_RESET_SUCCESS',
      userId,
      ip,
      timestamp: new Date(),
    });
  }
}

// 2. Implement cooldown period
async requestPasswordReset(email: string, ip: string) {
  // Check if reset was requested recently
  const recentReset = await this.checkRecentResetRequest(email);
  if (recentReset && new Date() < recentReset.cooldownUntil) {
    return {
      message: 'Please wait before requesting another reset.',
    };
  }

  // ... rest of reset logic
}

// 3. Notify user of all password reset attempts
async sendSecurityAlert(email: string) {
  await this.emailService.sendSecurityAlert(
    email,
    'Someone requested a password reset for your account.',
  );
}
```

**8. Environment Variables:**

```bash
# .env
FRONTEND_URL=http://localhost:3000
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password
SMTP_FROM="MyApp <noreply@myapp.com>"
```

**Key Takeaway:** Password reset flow generates secure random token (32 bytes hex), stores hashed token + expiration (1 hour) in database, sends email with reset link, validates token on submission, requires new password different from old, clears all refresh tokens on reset, sends confirmation email. Use rate limiting (3 requests/60s) to prevent abuse, don't reveal if email exists, implement audit logging for security monitoring.

</details>

<details>
<summary><strong>10. How do you implement email verification?</strong></summary>

**Answer:**

Implement email verification for new user registrations:

**1. Email Verification DTOs:**

```typescript
// auth/dto/verify-email.dto.ts
export class VerifyEmailDto {
  @ApiProperty()
  @IsString()
  token: string;
}

// auth/dto/resend-verification.dto.ts
export class ResendVerificationDto {
  @ApiProperty({ example: 'user@example.com' })
  @IsEmail()
  email: string;
}
```

**2. Email Verification Service:**

```typescript
// auth/email-verification.service.ts
import { Injectable, BadRequestException, NotFoundException } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import * as crypto from 'crypto';
import { UsersService } from '../users/users.service';
import { EmailService } from '../email/email.service';

@Injectable()
export class EmailVerificationService {
  constructor(
    private usersService: UsersService,
    private emailService: EmailService,
    private configService: ConfigService,
  ) {}

  // Send verification email to new user
  async sendVerificationEmail(userId: string): Promise<void> {
    const user = await this.usersService.findById(userId);
    if (!user) {
      throw new NotFoundException('User not found');
    }

    if (user.isEmailVerified) {
      throw new BadRequestException('Email is already verified');
    }

    // Generate verification token
    const verificationToken = this.generateVerificationToken();
    
    // Save token to database
    await this.usersService.update(userId, {
      emailVerificationToken: verificationToken,
    });

    // Send verification email
    const verificationUrl = `${this.configService.get('FRONTEND_URL')}/verify-email?token=${verificationToken}`;
    
    await this.emailService.sendVerificationEmail(
      user.email,
      verificationUrl,
      user.firstName,
    );
  }

  // Verify email with token
  async verifyEmail(token: string): Promise<{ message: string }> {
    const user = await this.usersService.findByVerificationToken(token);
    
    if (!user) {
      throw new BadRequestException('Invalid verification token');
    }

    if (user.isEmailVerified) {
      return {
        message: 'Email is already verified',
      };
    }

    // Mark email as verified
    await this.usersService.update(user.id, {
      isEmailVerified: true,
      emailVerificationToken: null, // Clear token after use
    });

    // Send welcome email
    this.emailService
      .sendWelcomeEmail(user.email, user.firstName)
      .catch(err => console.error('Failed to send welcome email:', err));

    return {
      message: 'Email verified successfully',
    };
  }

  // Resend verification email
  async resendVerificationEmail(email: string): Promise<{ message: string }> {
    const user = await this.usersService.findByEmail(email);
    
    // Don't reveal if email exists
    if (!user) {
      return {
        message: 'If an account exists, a verification email has been sent.',
      };
    }

    if (user.isEmailVerified) {
      throw new BadRequestException('Email is already verified');
    }

    // Check rate limiting (don't send too frequently)
    const lastSentAt = await this.getLastVerificationEmailSent(user.id);
    if (lastSentAt) {
      const timeSinceLastSent = Date.now() - lastSentAt.getTime();
      const cooldownPeriod = 60 * 1000; // 1 minute
      
      if (timeSinceLastSent < cooldownPeriod) {
        throw new BadRequestException('Please wait before requesting another verification email');
      }
    }

    // Generate new token
    const verificationToken = this.generateVerificationToken();
    
    await this.usersService.update(user.id, {
      emailVerificationToken: verificationToken,
    });

    // Send email
    const verificationUrl = `${this.configService.get('FRONTEND_URL')}/verify-email?token=${verificationToken}`;
    
    await this.emailService.sendVerificationEmail(
      user.email,
      verificationUrl,
      user.firstName,
    );

    return {
      message: 'If an account exists, a verification email has been sent.',
    };
  }

  // Check if email is verified (used in guards)
  async checkEmailVerified(userId: string): Promise<boolean> {
    const user = await this.usersService.findById(userId);
    return user?.isEmailVerified || false;
  }

  // Generate verification token
  private generateVerificationToken(): string {
    return crypto.randomBytes(32).toString('hex');
  }

  // Get last verification email sent time
  private async getLastVerificationEmailSent(userId: string): Promise<Date | null> {
    // Implement caching or database tracking
    // For now, simplified version
    return null;
  }
}
```

**3. Modified Registration Flow:**

```typescript
// auth/auth.service.ts
@Injectable()
export class AuthService {
  constructor(
    private usersService: UsersService,
    private jwtService: JwtService,
    private emailVerificationService: EmailVerificationService,
    private configService: ConfigService,
  ) {}

  async register(registerDto: RegisterDto) {
    // Check if user exists
    const existingUser = await this.usersService.findByEmail(registerDto.email);
    if (existingUser) {
      throw new ConflictException('Email already registered');
    }

    // Create user with isEmailVerified = false
    const user = await this.usersService.create({
      ...registerDto,
      password: await bcrypt.hash(registerDto.password, 10),
      isEmailVerified: false,
    });

    // Send verification email (async)
    this.emailVerificationService
      .sendVerificationEmail(user.id)
      .catch(err => console.error('Failed to send verification email:', err));

    // Generate tokens
    const tokens = await this.generateTokens(user);

    return {
      user: {
        id: user.id,
        email: user.email,
        firstName: user.firstName,
        lastName: user.lastName,
        role: user.role,
        isEmailVerified: user.isEmailVerified,
      },
      ...tokens,
      message: 'Registration successful. Please check your email to verify your account.',
    };
  }
}
```

**4. Email Verification Controller:**

```typescript
// auth/auth.controller.ts (add these endpoints)
@Controller('auth')
export class AuthController {
  constructor(
    private readonly authService: AuthService,
    private readonly emailVerificationService: EmailVerificationService,
  ) {}

  @Post('verify-email')
  @HttpCode(HttpStatus.OK)
  @ApiOperation({ summary: 'Verify email with token' })
  @ApiResponse({ status: 200, description: 'Email verified successfully' })
  @ApiResponse({ status: 400, description: 'Invalid token' })
  async verifyEmail(@Body() verifyEmailDto: VerifyEmailDto) {
    return this.emailVerificationService.verifyEmail(verifyEmailDto.token);
  }

  @Post('resend-verification')
  @HttpCode(HttpStatus.OK)
  @Throttle(3, 300) // 3 requests per 5 minutes
  @ApiOperation({ summary: 'Resend verification email' })
  async resendVerification(@Body() resendDto: ResendVerificationDto) {
    return this.emailVerificationService.resendVerificationEmail(resendDto.email);
  }

  @Get('check-verification')
  @UseGuards(JwtAuthGuard)
  @ApiBearerAuth()
  @ApiOperation({ summary: 'Check if current user email is verified' })
  async checkVerification(@Request() req) {
    const isVerified = await this.emailVerificationService.checkEmailVerified(req.user.id);
    return { isEmailVerified: isVerified };
  }
}
```

**5. Email Verification Guard (Optional - Enforce Verification):**

```typescript
// auth/guards/email-verified.guard.ts
import { Injectable, CanActivate, ExecutionContext, ForbiddenException } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { EmailVerificationService } from '../email-verification.service';

@Injectable()
export class EmailVerifiedGuard implements CanActivate {
  constructor(
    private reflector: Reflector,
    private emailVerificationService: EmailVerificationService,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    // Check if route allows unverified emails
    const allowUnverified = this.reflector.get<boolean>(
      'allowUnverified',
      context.getHandler(),
    );
    
    if (allowUnverified) {
      return true;
    }

    const request = context.switchToHttp().getRequest();
    const user = request.user;

    if (!user) {
      return false;
    }

    const isVerified = await this.emailVerificationService.checkEmailVerified(user.id);
    
    if (!isVerified) {
      throw new ForbiddenException('Please verify your email address to access this resource');
    }

    return true;
  }
}

// Decorator for routes that allow unverified users
export const AllowUnverified = () => SetMetadata('allowUnverified', true);

// Usage:
@Post('create-product')
@UseGuards(JwtAuthGuard, EmailVerifiedGuard) // Requires verified email
async createProduct(@Body() dto: CreateProductDto) { ... }

@Get('profile')
@UseGuards(JwtAuthGuard)
@AllowUnverified() // Allows unverified users
async getProfile(@Request() req) { ... }
```

**6. Email Templates:**

```typescript
// email/email.service.ts
async sendVerificationEmail(to: string, verificationUrl: string, userName: string) {
  const html = `
    <!DOCTYPE html>
    <html>
      <head>
        <style>
          body { font-family: Arial, sans-serif; line-height: 1.6; }
          .container { max-width: 600px; margin: 0 auto; padding: 20px; }
          .button {
            display: inline-block;
            padding: 12px 24px;
            background-color: #28a745;
            color: #ffffff;
            text-decoration: none;
            border-radius: 4px;
            margin: 20px 0;
          }
          .footer { margin-top: 30px; font-size: 12px; color: #666; }
        </style>
      </head>
      <body>
        <div class="container">
          <h2>Welcome to MyApp!</h2>
          <p>Hi ${userName},</p>
          <p>Thank you for registering. Please verify your email address by clicking the button below:</p>
          <a href="${verificationUrl}" class="button">Verify Email</a>
          <p>Or copy and paste this link into your browser:</p>
          <p><a href="${verificationUrl}">${verificationUrl}</a></p>
          <p><strong>This link will not expire.</strong></p>
          <p>If you didn't create an account, please ignore this email.</p>
          <div class="footer">
            <p>This is an automated email, please do not reply.</p>
          </div>
        </div>
      </body>
    </html>
  `;

  await this.transporter.sendMail({
    from: this.configService.get('SMTP_FROM'),
    to,
    subject: 'Verify Your Email Address',
    html,
  });
}

async sendWelcomeEmail(to: string, userName: string) {
  const html = `
    <!DOCTYPE html>
    <html>
      <body>
        <div style="font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto; padding: 20px;">
          <h2>Email Verified Successfully!</h2>
          <p>Hi ${userName},</p>
          <p>Your email has been verified successfully. You now have full access to all features.</p>
          <p>Thank you for joining us!</p>
          <p style="margin-top: 30px; font-size: 12px; color: #666;">
            This is an automated email, please do not reply.
          </p>
        </div>
      </body>
    </html>
  `;

  await this.transporter.sendMail({
    from: this.configService.get('SMTP_FROM'),
    to,
    subject: 'Welcome to MyApp!',
    html,
  });
}
```

**7. Users Service Addition:**

```typescript
// users/users.service.ts
@Injectable()
export class UsersService {
  // ... existing methods ...

  // Find user by verification token
  async findByVerificationToken(token: string): Promise<User | null> {
    return this.userRepository.findOne({
      where: { emailVerificationToken: token },
    });
  }

  // Update method should support emailVerificationToken
  async update(id: string, updateData: Partial<User>): Promise<User> {
    await this.userRepository.update(id, updateData);
    return this.findById(id);
  }
}
```

**8. Complete Flow:**

```typescript
/*
EMAIL VERIFICATION FLOW:

1. User registers:
   POST /auth/register { email, password, firstName, lastName }
   → Creates user with isEmailVerified = false
   → Generates verification token
   → Sends verification email
   → Returns tokens (user can use app but with limited access)

2. User checks email and clicks verification link:
   Frontend redirects to: /verify-email?token=abc123
   POST /auth/verify-email { token: "abc123" }
   → Validates token
   → Sets isEmailVerified = true
   → Clears token
   → Sends welcome email

3. User tries to access protected resources:
   - Without EmailVerifiedGuard: Full access
   - With EmailVerifiedGuard: Blocked with 403 if not verified

4. User didn't receive email:
   POST /auth/resend-verification { email: "user@example.com" }
   → Generates new token
   → Sends new verification email
   → Rate limited (3 per 5 minutes)

5. Check verification status:
   GET /auth/check-verification (with JWT)
   → Returns { isEmailVerified: true/false }
*/
```

**9. Interceptor for Verification Reminder:**

```typescript
// common/interceptors/verification-reminder.interceptor.ts
@Injectable()
export class VerificationReminderInterceptor implements NestInterceptor {
  constructor(private emailVerificationService: EmailVerificationService) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(async data => {
        const request = context.switchToHttp().getRequest();
        const user = request.user;

        if (user && !user.isEmailVerified) {
          // Add reminder in response
          return {
            ...data,
            _reminder: 'Please verify your email address for full account access',
          };
        }

        return data;
      }),
    );
  }
}
```

**Key Takeaway:** Email verification generates unique token on registration, stores in database (emailVerificationToken field), sends email with verification link, validates token on submission setting isEmailVerified=true, supports resend with rate limiting (3/5min), optionally use EmailVerifiedGuard to enforce verification for sensitive endpoints, send welcome email post-verification, allow @AllowUnverified() decorator for routes accessible without verification.

</details>

<details>
<summary><strong>11. How do you design role-based access control (RBAC)?</strong></summary>

**Answer:**

Implement flexible RBAC with roles, guards, and decorators:

**1. Role Enum & User Entity:**

```typescript
// users/entities/role.enum.ts
export enum Role {
  ADMIN = 'admin',
  MODERATOR = 'moderator',
  USER = 'user',
  GUEST = 'guest',
}

// users/entities/user.entity.ts
@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  email: string;

  @Column()
  password: string;

  @Column({
    type: 'enum',
    enum: Role,
    default: Role.USER,
  })
  role: Role;

  // Or for multiple roles
  @Column('simple-array', { default: '' })
  roles: Role[];

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}
```

**2. Roles Decorator:**

```typescript
// auth/decorators/roles.decorator.ts
import { SetMetadata } from '@nestjs/common';
import { Role } from '../../users/entities/role.enum';

export const ROLES_KEY = 'roles';
export const Roles = (...roles: Role[]) => SetMetadata(ROLES_KEY, roles);

// Usage:
// @Roles(Role.ADMIN)
// @Roles(Role.ADMIN, Role.MODERATOR)
```

**3. Roles Guard:**

```typescript
// auth/guards/roles.guard.ts
import { Injectable, CanActivate, ExecutionContext, ForbiddenException } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { Role } from '../../users/entities/role.enum';
import { ROLES_KEY } from '../decorators/roles.decorator';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // Get required roles from decorator
    const requiredRoles = this.reflector.getAllAndOverride<Role[]>(ROLES_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);

    // If no roles required, allow access
    if (!requiredRoles) {
      return true;
    }

    const { user } = context.switchToHttp().getRequest();
    
    if (!user) {
      throw new ForbiddenException('User not authenticated');
    }

    // Check if user has any of the required roles
    const hasRole = requiredRoles.some((role) => user.role === role);
    
    // For multiple roles support:
    // const hasRole = requiredRoles.some((role) => user.roles?.includes(role));

    if (!hasRole) {
      throw new ForbiddenException(
        `Access denied. Required roles: ${requiredRoles.join(', ')}`,
      );
    }

    return true;
  }
}
```

**4. Apply Guards in Controllers:**

```typescript
// products/products.controller.ts
@Controller('products')
@UseGuards(JwtAuthGuard, RolesGuard) // Apply guards at controller level
export class ProductsController {
  // Public endpoint - no role required
  @Get()
  @Public() // From auth module
  findAll() {
    return this.productsService.findAll();
  }

  // Any authenticated user
  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.productsService.findOne(id);
  }

  // Only users with ADMIN or MODERATOR role
  @Post()
  @Roles(Role.ADMIN, Role.MODERATOR)
  create(@Body() createProductDto: CreateProductDto) {
    return this.productsService.create(createProductDto);
  }

  // Only admins
  @Delete(':id')
  @Roles(Role.ADMIN)
  remove(@Param('id') id: string) {
    return this.productsService.remove(id);
  }

  // Only admins
  @Patch(':id')
  @Roles(Role.ADMIN)
  update(@Param('id') id: string, @Body() updateDto: UpdateProductDto) {
    return this.productsService.update(id, updateDto);
  }
}
```

**5. Global Roles Guard Setup:**

```typescript
// app.module.ts
import { APP_GUARD } from '@nestjs/core';
import { RolesGuard } from './auth/guards/roles.guard';

@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: JwtAuthGuard, // First check authentication
    },
    {
      provide: APP_GUARD,
      useClass: RolesGuard, // Then check authorization
    },
  ],
})
export class AppModule {}
```

**6. Role Hierarchy (Advanced):**

```typescript
// auth/role-hierarchy.ts
export const ROLE_HIERARCHY: Record<Role, number> = {
  [Role.GUEST]: 0,
  [Role.USER]: 1,
  [Role.MODERATOR]: 2,
  [Role.ADMIN]: 3,
};

// auth/guards/roles.guard.ts (enhanced)
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<Role[]>(ROLES_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);

    if (!requiredRoles) {
      return true;
    }

    const { user } = context.switchToHttp().getRequest();
    
    if (!user) {
      throw new ForbiddenException('User not authenticated');
    }

    // Check if user's role level is high enough
    const userRoleLevel = ROLE_HIERARCHY[user.role];
    const requiredLevel = Math.min(...requiredRoles.map(role => ROLE_HIERARCHY[role]));

    if (userRoleLevel < requiredLevel) {
      throw new ForbiddenException(
        `Access denied. Required role level: ${requiredLevel}`,
      );
    }

    return true;
  }
}
```

**7. Role Management Endpoints:**

```typescript
// users/users.controller.ts
@Controller('users')
@UseGuards(JwtAuthGuard, RolesGuard)
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  // Only admins can change user roles
  @Patch(':id/role')
  @Roles(Role.ADMIN)
  @ApiOperation({ summary: 'Change user role' })
  async updateRole(
    @Param('id') id: string,
    @Body() updateRoleDto: UpdateRoleDto,
    @Request() req,
  ) {
    // Prevent demoting yourself
    if (id === req.user.id) {
      throw new BadRequestException('Cannot change your own role');
    }

    // Prevent creating super admins unless you are one
    if (updateRoleDto.role === Role.ADMIN && req.user.role !== Role.ADMIN) {
      throw new ForbiddenException('Only admins can create other admins');
    }

    return this.usersService.updateRole(id, updateRoleDto.role);
  }

  // List all users with their roles (admin/moderator only)
  @Get()
  @Roles(Role.ADMIN, Role.MODERATOR)
  async findAll(@Query() query: PaginationDto) {
    return this.usersService.findAll(query);
  }
}

// users/dto/update-role.dto.ts
export class UpdateRoleDto {
  @ApiProperty({ enum: Role })
  @IsEnum(Role)
  role: Role;

  @ApiProperty({ required: false })
  @IsString()
  @IsOptional()
  reason?: string; // Audit trail
}
```

**8. User Service Role Methods:**

```typescript
// users/users.service.ts
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>,
    private auditService: AuditService,
  ) {}

  async updateRole(userId: string, newRole: Role): Promise<User> {
    const user = await this.findById(userId);
    
    if (!user) {
      throw new NotFoundException('User not found');
    }

    const oldRole = user.role;
    user.role = newRole;
    
    await this.userRepository.save(user);

    // Log role change for audit
    await this.auditService.logRoleChange(userId, oldRole, newRole);

    return user;
  }

  async hasRole(userId: string, role: Role): Promise<boolean> {
    const user = await this.findById(userId);
    return user?.role === role;
  }

  async getUsersByRole(role: Role): Promise<User[]> {
    return this.userRepository.find({ where: { role } });
  }
}
```

**9. Combine with Resource Ownership:**

```typescript
// auth/guards/resource-ownership.guard.ts
@Injectable()
export class ResourceOwnershipGuard implements CanActivate {
  constructor(
    private reflector: Reflector,
    private postsService: PostsService,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    const resourceId = request.params.id;

    // Admins can access everything
    if (user.role === Role.ADMIN) {
      return true;
    }

    // Check if user owns the resource
    const resource = await this.postsService.findOne(resourceId);
    
    if (!resource) {
      throw new NotFoundException('Resource not found');
    }

    if (resource.authorId !== user.id) {
      throw new ForbiddenException('You can only modify your own resources');
    }

    return true;
  }
}

// Usage:
@Patch(':id')
@UseGuards(JwtAuthGuard, ResourceOwnershipGuard)
async update(@Param('id') id: string, @Body() dto: UpdatePostDto) {
  return this.postsService.update(id, dto);
}
```

**10. Multiple Roles Per User:**

```typescript
// For systems where users can have multiple roles

// users/entities/user.entity.ts
@Entity('users')
export class User {
  @Column('simple-array')
  roles: Role[];
}

// auth/guards/roles.guard.ts (modified)
canActivate(context: ExecutionContext): boolean {
  const requiredRoles = this.reflector.getAllAndOverride<Role[]>(ROLES_KEY, [
    context.getHandler(),
    context.getClass(),
  ]);

  if (!requiredRoles) {
    return true;
  }

  const { user } = context.switchToHttp().getRequest();
  
  // Check if user has ANY of the required roles
  const hasRole = requiredRoles.some((role) => user.roles.includes(role));

  if (!hasRole) {
    throw new ForbiddenException(
      `Access denied. Required one of: ${requiredRoles.join(', ')}`,
    );
  }

  return true;
}
```

**11. Current User Decorator:**

```typescript
// auth/decorators/current-user.decorator.ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const CurrentUser = createParamDecorator(
  (data: string | undefined, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const user = request.user;

    return data ? user?.[data] : user;
  },
);

// Usage:
@Get('profile')
async getProfile(@CurrentUser() user: User) {
  return user;
}

@Get('role')
async getRole(@CurrentUser('role') role: Role) {
  return { role };
}
```

**12. Complete RBAC Flow:**

```typescript
/*
RBAC FLOW:

1. User logs in:
   POST /auth/login → JWT with user.role embedded

2. User makes request to protected endpoint:
   GET /products/123
   Headers: Authorization: Bearer <JWT>
   
3. JwtAuthGuard validates JWT:
   → Extracts user from token
   → Attaches to request.user

4. RolesGuard checks authorization:
   → Gets required roles from @Roles() decorator
   → Compares with request.user.role
   → Allows or denies access

5. Admin changes user role:
   PATCH /users/abc/role { role: "moderator" }
   → Validates requester is admin
   → Updates user.role in database
   → Logs to audit trail
   → User's next login will have new role

GUARD ORDER MATTERS:
1. JwtAuthGuard (authentication)
2. RolesGuard (authorization)
3. ResourceOwnershipGuard (resource-specific)
*/
```

**13. Audit Logging:**

```typescript
// audit/audit.service.ts
@Injectable()
export class AuditService {
  constructor(
    @InjectRepository(AuditLog)
    private auditLogRepository: Repository<AuditLog>,
  ) {}

  async logRoleChange(userId: string, oldRole: Role, newRole: Role) {
    await this.auditLogRepository.save({
      action: 'ROLE_CHANGE',
      userId,
      details: {
        oldRole,
        newRole,
      },
      timestamp: new Date(),
    });
  }

  async logAccessDenied(userId: string, resource: string, requiredRole: Role) {
    await this.auditLogRepository.save({
      action: 'ACCESS_DENIED',
      userId,
      details: {
        resource,
        requiredRole,
        userRole: userId, // Fetch from user
      },
      timestamp: new Date(),
    });
  }
}
```

**Key Takeaway:** RBAC uses Role enum (ADMIN, MODERATOR, USER, GUEST), @Roles() decorator to mark required roles, RolesGuard to check user.role against required roles using Reflector, guard order matters (JwtAuthGuard first for authentication, then RolesGuard for authorization), can implement role hierarchy for level-based access, combine with resource ownership checks, support multiple roles per user with array, use CurrentUser decorator for convenience, log role changes for audit trail.

</details>

<details>
<summary><strong>12. How do you implement permission-based authorization?</strong></summary>

**Answer:**

Implement granular permission-based authorization for fine-grained access control:

**1. Permission Enum & Entity:**

```typescript
// auth/entities/permission.enum.ts
export enum Permission {
  // User permissions
  CREATE_USER = 'create:user',
  READ_USER = 'read:user',
  UPDATE_USER = 'update:user',
  DELETE_USER = 'delete:user',
  
  // Product permissions
  CREATE_PRODUCT = 'create:product',
  READ_PRODUCT = 'read:product',
  UPDATE_PRODUCT = 'update:product',
  DELETE_PRODUCT = 'delete:product',
  PUBLISH_PRODUCT = 'publish:product',
  
  // Order permissions
  CREATE_ORDER = 'create:order',
  READ_ORDER = 'read:order',
  UPDATE_ORDER = 'update:order',
  CANCEL_ORDER = 'cancel:order',
  REFUND_ORDER = 'refund:order',
  
  // Content permissions
  MODERATE_CONTENT = 'moderate:content',
  DELETE_CONTENT = 'delete:content',
  
  // System permissions
  VIEW_ANALYTICS = 'view:analytics',
  MANAGE_SETTINGS = 'manage:settings',
  VIEW_LOGS = 'view:logs',
}

// auth/entities/role-permissions.ts
export const ROLE_PERMISSIONS: Record<Role, Permission[]> = {
  [Role.ADMIN]: [
    // Admins have all permissions
    ...Object.values(Permission),
  ],
  
  [Role.MODERATOR]: [
    Permission.READ_USER,
    Permission.READ_PRODUCT,
    Permission.UPDATE_PRODUCT,
    Permission.MODERATE_CONTENT,
    Permission.DELETE_CONTENT,
    Permission.READ_ORDER,
    Permission.UPDATE_ORDER,
    Permission.VIEW_ANALYTICS,
  ],
  
  [Role.USER]: [
    Permission.READ_USER,
    Permission.UPDATE_USER, // Only their own
    Permission.READ_PRODUCT,
    Permission.CREATE_ORDER,
    Permission.READ_ORDER, // Only their own
  ],
  
  [Role.GUEST]: [
    Permission.READ_PRODUCT,
  ],
};
```

**2. Permission Decorator:**

```typescript
// auth/decorators/permissions.decorator.ts
import { SetMetadata } from '@nestjs/common';
import { Permission } from '../entities/permission.enum';

export const PERMISSIONS_KEY = 'permissions';
export const RequirePermissions = (...permissions: Permission[]) => 
  SetMetadata(PERMISSIONS_KEY, permissions);

// Require ALL permissions
export const RequireAllPermissions = (...permissions: Permission[]) =>
  SetMetadata(PERMISSIONS_KEY, { permissions, requireAll: true });

// Require ANY permission
export const RequireAnyPermission = (...permissions: Permission[]) =>
  SetMetadata(PERMISSIONS_KEY, { permissions, requireAll: false });
```

**3. Permissions Guard:**

```typescript
// auth/guards/permissions.guard.ts
import { Injectable, CanActivate, ExecutionContext, ForbiddenException } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { Permission } from '../entities/permission.enum';
import { PERMISSIONS_KEY } from '../decorators/permissions.decorator';
import { ROLE_PERMISSIONS } from '../entities/role-permissions';

@Injectable()
export class PermissionsGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // Get required permissions from decorator
    const permissionsMeta = this.reflector.getAllAndOverride<
      Permission[] | { permissions: Permission[]; requireAll: boolean }
    >(PERMISSIONS_KEY, [context.getHandler(), context.getClass()]);

    if (!permissionsMeta) {
      return true; // No permissions required
    }

    const { user } = context.switchToHttp().getRequest();
    
    if (!user) {
      throw new ForbiddenException('User not authenticated');
    }

    // Get user's permissions based on role
    const userPermissions = ROLE_PERMISSIONS[user.role] || [];

    // Handle different decorator types
    let requiredPermissions: Permission[];
    let requireAll = false;

    if (Array.isArray(permissionsMeta)) {
      requiredPermissions = permissionsMeta;
      requireAll = false; // Default: require ANY
    } else {
      requiredPermissions = permissionsMeta.permissions;
      requireAll = permissionsMeta.requireAll;
    }

    // Check permissions
    let hasPermission: boolean;

    if (requireAll) {
      // User must have ALL required permissions
      hasPermission = requiredPermissions.every((permission) =>
        userPermissions.includes(permission),
      );
    } else {
      // User must have at least ONE required permission
      hasPermission = requiredPermissions.some((permission) =>
        userPermissions.includes(permission),
      );
    }

    if (!hasPermission) {
      throw new ForbiddenException(
        `Missing required permissions: ${requiredPermissions.join(', ')}`,
      );
    }

    return true;
  }
}
```

**4. Apply Permissions in Controllers:**

```typescript
// products/products.controller.ts
@Controller('products')
@UseGuards(JwtAuthGuard, PermissionsGuard)
export class ProductsController {
  @Get()
  @Public() // Public endpoint
  findAll() {
    return this.productsService.findAll();
  }

  @Get(':id')
  @RequirePermissions(Permission.READ_PRODUCT)
  findOne(@Param('id') id: string) {
    return this.productsService.findOne(id);
  }

  @Post()
  @RequirePermissions(Permission.CREATE_PRODUCT)
  create(@Body() createProductDto: CreateProductDto) {
    return this.productsService.create(createProductDto);
  }

  @Patch(':id')
  @RequirePermissions(Permission.UPDATE_PRODUCT)
  update(@Param('id') id: string, @Body() updateDto: UpdateProductDto) {
    return this.productsService.update(id, updateDto);
  }

  @Post(':id/publish')
  @RequireAllPermissions(Permission.UPDATE_PRODUCT, Permission.PUBLISH_PRODUCT)
  publish(@Param('id') id: string) {
    return this.productsService.publish(id);
  }

  @Delete(':id')
  @RequirePermissions(Permission.DELETE_PRODUCT)
  remove(@Param('id') id: string) {
    return this.productsService.remove(id);
  }
}

// orders/orders.controller.ts
@Controller('orders')
@UseGuards(JwtAuthGuard, PermissionsGuard)
export class OrdersController {
  @Post(':id/refund')
  @RequirePermissions(Permission.REFUND_ORDER)
  refundOrder(@Param('id') id: string) {
    return this.ordersService.refund(id);
  }

  @Patch(':id/cancel')
  @RequireAnyPermission(Permission.CANCEL_ORDER, Permission.UPDATE_ORDER)
  cancelOrder(@Param('id') id: string, @CurrentUser() user: User) {
    // Additional check: users can cancel their own orders
    return this.ordersService.cancel(id, user.id);
  }
}
```

**5. Database-Driven Permissions (Advanced):**

```typescript
// auth/entities/permission.entity.ts
@Entity('permissions')
export class PermissionEntity {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  name: string;

  @Column()
  description: string;

  @Column()
  resource: string; // e.g., 'product', 'order', 'user'

  @Column()
  action: string; // e.g., 'create', 'read', 'update', 'delete'

  @ManyToMany(() => RoleEntity, role => role.permissions)
  roles: RoleEntity[];
}

// auth/entities/role.entity.ts
@Entity('roles')
export class RoleEntity {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  name: string;

  @Column()
  description: string;

  @ManyToMany(() => PermissionEntity, permission => permission.roles, {
    eager: true, // Load permissions with role
  })
  @JoinTable({
    name: 'role_permissions',
    joinColumn: { name: 'role_id' },
    inverseJoinColumn: { name: 'permission_id' },
  })
  permissions: PermissionEntity[];

  @OneToMany(() => User, user => user.role)
  users: User[];
}

// users/entities/user.entity.ts
@Entity('users')
export class User {
  @ManyToOne(() => RoleEntity, role => role.users, { eager: true })
  @JoinColumn({ name: 'role_id' })
  role: RoleEntity;

  // Direct permissions (overrides)
  @ManyToMany(() => PermissionEntity, { eager: true })
  @JoinTable({
    name: 'user_permissions',
    joinColumn: { name: 'user_id' },
    inverseJoinColumn: { name: 'permission_id' },
  })
  permissions: PermissionEntity[];
}
```

**6. Enhanced Permissions Guard with Database:**

```typescript
// auth/guards/permissions.guard.ts (database version)
@Injectable()
export class PermissionsGuard implements CanActivate {
  constructor(
    private reflector: Reflector,
    private authService: AuthService,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const requiredPermissions = this.reflector.get<string[]>(
      PERMISSIONS_KEY,
      context.getHandler(),
    );

    if (!requiredPermissions) {
      return true;
    }

    const { user } = context.switchToHttp().getRequest();
    
    if (!user) {
      throw new ForbiddenException('User not authenticated');
    }

    // Get user permissions from database (combines role + user-specific)
    const userPermissions = await this.authService.getUserPermissions(user.id);

    const hasPermission = requiredPermissions.every((permission) =>
      userPermissions.some(p => p.name === permission),
    );

    if (!hasPermission) {
      throw new ForbiddenException(
        `Missing required permissions: ${requiredPermissions.join(', ')}`,
      );
    }

    return true;
  }
}

// auth/auth.service.ts
@Injectable()
export class AuthService {
  async getUserPermissions(userId: string): Promise<PermissionEntity[]> {
    const user = await this.userRepository.findOne({
      where: { id: userId },
      relations: ['role', 'role.permissions', 'permissions'],
    });

    if (!user) {
      return [];
    }

    // Combine role permissions + user-specific permissions
    const rolePermissions = user.role?.permissions || [];
    const userPermissions = user.permissions || [];

    // Remove duplicates
    const allPermissions = [...rolePermissions, ...userPermissions];
    const uniquePermissions = Array.from(
      new Map(allPermissions.map(p => [p.id, p])).values(),
    );

    return uniquePermissions;
  }
}
```

**7. Permission Management Endpoints:**

```typescript
// admin/permissions.controller.ts
@Controller('admin/permissions')
@UseGuards(JwtAuthGuard, PermissionsGuard)
export class PermissionsController {
  constructor(private readonly permissionsService: PermissionsService) {}

  @Get()
  @RequirePermissions(Permission.MANAGE_SETTINGS)
  findAll() {
    return this.permissionsService.findAll();
  }

  @Post('grant')
  @RequirePermissions(Permission.MANAGE_SETTINGS)
  grantPermissionToUser(@Body() dto: GrantPermissionDto) {
    return this.permissionsService.grantToUser(dto.userId, dto.permissionId);
  }

  @Post('revoke')
  @RequirePermissions(Permission.MANAGE_SETTINGS)
  revokePermissionFromUser(@Body() dto: RevokePermissionDto) {
    return this.permissionsService.revokeFromUser(dto.userId, dto.permissionId);
  }

  @Get('user/:userId')
  @RequirePermissions(Permission.READ_USER)
  getUserPermissions(@Param('userId') userId: string) {
    return this.permissionsService.getUserPermissions(userId);
  }
}

// admin/dto/grant-permission.dto.ts
export class GrantPermissionDto {
  @IsUUID()
  userId: string;

  @IsUUID()
  permissionId: string;

  @IsString()
  @IsOptional()
  reason?: string;
}
```

**8. Check Permissions Programmatically:**

```typescript
// auth/auth.service.ts
@Injectable()
export class AuthService {
  async hasPermission(userId: string, permission: string): Promise<boolean> {
    const permissions = await this.getUserPermissions(userId);
    return permissions.some(p => p.name === permission);
  }

  async hasAllPermissions(userId: string, permissions: string[]): Promise<boolean> {
    const userPermissions = await this.getUserPermissions(userId);
    return permissions.every(p => 
      userPermissions.some(up => up.name === p),
    );
  }

  async hasAnyPermission(userId: string, permissions: string[]): Promise<boolean> {
    const userPermissions = await this.getUserPermissions(userId);
    return permissions.some(p =>
      userPermissions.some(up => up.name === p),
    );
  }
}

// Usage in service:
async deleteProduct(id: string, userId: string) {
  const hasPermission = await this.authService.hasPermission(
    userId,
    Permission.DELETE_PRODUCT,
  );

  if (!hasPermission) {
    throw new ForbiddenException('Missing delete permission');
  }

  return this.productRepository.delete(id);
}
```

**9. Permission Caching:**

```typescript
// auth/auth.service.ts (with caching)
import { CACHE_MANAGER, Inject } from '@nestjs/common';
import { Cache } from 'cache-manager';

@Injectable()
export class AuthService {
  constructor(
    @Inject(CACHE_MANAGER) private cacheManager: Cache,
  ) {}

  async getUserPermissions(userId: string): Promise<PermissionEntity[]> {
    // Try cache first
    const cacheKey = `user:${userId}:permissions`;
    const cached = await this.cacheManager.get<PermissionEntity[]>(cacheKey);
    
    if (cached) {
      return cached;
    }

    // Fetch from database
    const permissions = await this.fetchPermissionsFromDB(userId);

    // Cache for 5 minutes
    await this.cacheManager.set(cacheKey, permissions, { ttl: 300 });

    return permissions;
  }

  async invalidateUserPermissions(userId: string) {
    const cacheKey = `user:${userId}:permissions`;
    await this.cacheManager.del(cacheKey);
  }

  // Call when user role changes or permissions are granted/revoked
  async onPermissionsChanged(userId: string) {
    await this.invalidateUserPermissions(userId);
  }
}
```

**10. Resource-Specific Permissions:**

```typescript
// For permissions tied to specific resources (e.g., "can edit Post #123")

// auth/decorators/resource-permission.decorator.ts
export const RequireResourcePermission = (
  resource: string,
  action: string,
) => SetMetadata('resource-permission', { resource, action });

// auth/guards/resource-permission.guard.ts
@Injectable()
export class ResourcePermissionGuard implements CanActivate {
  async canActivate(context: ExecutionContext): Promise<boolean> {
    const { user } = context.switchToHttp().getRequest();
    const resourceId = context.switchToHttp().getRequest().params.id;
    
    const permissionMeta = this.reflector.get<{ resource: string; action: string }>(
      'resource-permission',
      context.getHandler(),
    );

    if (!permissionMeta) {
      return true;
    }

    // Check if user owns resource OR has admin permission
    const hasPermission = await this.checkResourcePermission(
      user.id,
      resourceId,
      permissionMeta.resource,
      permissionMeta.action,
    );

    if (!hasPermission) {
      throw new ForbiddenException('You do not have permission for this resource');
    }

    return true;
  }

  private async checkResourcePermission(
    userId: string,
    resourceId: string,
    resource: string,
    action: string,
  ): Promise<boolean> {
    // Implementation depends on your business logic
    // Example: check ownership or admin status
    return true;
  }
}

// Usage:
@Patch(':id')
@RequireResourcePermission('post', 'update')
async updatePost(@Param('id') id: string, @Body() dto: UpdatePostDto) {
  return this.postsService.update(id, dto);
}
```

**11. Permission Seeding:**

```typescript
// database/seeds/permissions.seed.ts
export class PermissionsSeed {
  async run(dataSource: DataSource) {
    const permissionRepo = dataSource.getRepository(PermissionEntity);
    const roleRepo = dataSource.getRepository(RoleEntity);

    // Create permissions
    const permissions = await permissionRepo.save([
      { name: 'create:product', resource: 'product', action: 'create' },
      { name: 'read:product', resource: 'product', action: 'read' },
      { name: 'update:product', resource: 'product', action: 'update' },
      { name: 'delete:product', resource: 'product', action: 'delete' },
      // ... more permissions
    ]);

    // Create roles with permissions
    const adminRole = await roleRepo.save({
      name: 'admin',
      permissions: permissions, // All permissions
    });

    const userRole = await roleRepo.save({
      name: 'user',
      permissions: permissions.filter(p => 
        p.action === 'read' || (p.action === 'create' && p.resource !== 'user'),
      ),
    });
  }
}
```

**Key Takeaway:** Permission-based authorization provides fine-grained access control using Permission enum (resource:action format like 'create:product'), @RequirePermissions() decorator marking required permissions, PermissionsGuard checking user permissions from role-based mapping or database, supports RequireAllPermissions (AND logic) and RequireAnyPermission (OR logic), can combine role permissions + user-specific overrides, cache permissions for performance (5min TTL), invalidate cache on permission changes, seed permissions in database, implement resource-specific permissions for ownership checks, more flexible than pure RBAC.

</details>

## Real-World Application Scenarios

<details>
<summary><strong>13. How would you build a chat application using NestJS?</strong></summary>

**Answer:**

Build a real-time chat application with WebSockets, rooms, and message persistence:

**1. Install Dependencies:**

```bash
npm install @nestjs/websockets @nestjs/platform-socket.io socket.io
npm install @nestjs/typeorm typeorm pg
```

**2. Message Entity:**

```typescript
// chat/entities/message.entity.ts
@Entity('messages')
export class Message {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column('text')
  content: string;

  @ManyToOne(() => User, user => user.sentMessages)
  @JoinColumn({ name: 'sender_id' })
  sender: User;

  @Column({ name: 'sender_id' })
  senderId: string;

  @ManyToOne(() => Room, room => room.messages, { onDelete: 'CASCADE' })
  @JoinColumn({ name: 'room_id' })
  room: Room;

  @Column({ name: 'room_id' })
  roomId: string;

  @Column({ type: 'enum', enum: ['text', 'image', 'file'], default: 'text' })
  type: string;

  @Column({ nullable: true })
  fileUrl: string;

  @Column({ default: false })
  isEdited: boolean;

  @Column({ nullable: true, type: 'timestamp' })
  editedAt: Date;

  @CreateDateColumn()
  createdAt: Date;

  @Column({ default: false })
  isDeleted: boolean;
}

// chat/entities/room.entity.ts
@Entity('rooms')
export class Room {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  name: string;

  @Column({ type: 'enum', enum: ['private', 'group'], default: 'private' })
  type: string;

  @ManyToMany(() => User, user => user.rooms)
  @JoinTable({
    name: 'room_members',
    joinColumn: { name: 'room_id' },
    inverseJoinColumn: { name: 'user_id' },
  })
  members: User[];

  @OneToMany(() => Message, message => message.room)
  messages: Message[];

  @ManyToOne(() => User)
  @JoinColumn({ name: 'created_by' })
  createdBy: User;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}
```

**3. WebSocket Gateway:**

```typescript
// chat/chat.gateway.ts
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  MessageBody,
  ConnectedSocket,
  OnGatewayConnection,
  OnGatewayDisconnect,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
import { UseGuards } from '@nestjs/common';
import { WsJwtGuard } from '../auth/guards/ws-jwt.guard';
import { CurrentUser } from '../auth/decorators/current-user.decorator';

@WebSocketGateway({
  cors: {
    origin: '*', // Configure properly in production
  },
  namespace: '/chat',
})
@UseGuards(WsJwtGuard)
export class ChatGateway implements OnGatewayConnection, OnGatewayDisconnect {
  @WebSocketServer()
  server: Server;

  // Track online users
  private onlineUsers = new Map<string, string>(); // userId -> socketId

  constructor(
    private chatService: ChatService,
    private roomService: RoomService,
  ) {}

  // User connects
  async handleConnection(client: Socket) {
    try {
      const user = client.data.user; // Set by WsJwtGuard
      
      if (!user) {
        client.disconnect();
        return;
      }

      // Track user
      this.onlineUsers.set(user.id, client.id);
      
      // Join user's rooms
      const userRooms = await this.roomService.getUserRooms(user.id);
      userRooms.forEach(room => {
        client.join(`room:${room.id}`);
      });

      // Notify others that user is online
      this.server.emit('user:online', {
        userId: user.id,
        username: user.username,
      });

      console.log(`User ${user.username} connected`);
    } catch (error) {
      console.error('Connection error:', error);
      client.disconnect();
    }
  }

  // User disconnects
  handleDisconnect(client: Socket) {
    const user = client.data.user;
    
    if (user) {
      this.onlineUsers.delete(user.id);
      
      // Notify others that user is offline
      this.server.emit('user:offline', {
        userId: user.id,
        username: user.username,
      });

      console.log(`User ${user.username} disconnected`);
    }
  }

  // Send message
  @SubscribeMessage('message:send')
  async handleMessage(
    @MessageBody() data: { roomId: string; content: string; type?: string },
    @ConnectedSocket() client: Socket,
    @CurrentUser() user: User,
  ) {
    try {
      // Verify user is member of room
      const isMember = await this.roomService.isMember(data.roomId, user.id);
      if (!isMember) {
        client.emit('error', { message: 'Not a member of this room' });
        return;
      }

      // Save message to database
      const message = await this.chatService.createMessage({
        content: data.content,
        senderId: user.id,
        roomId: data.roomId,
        type: data.type || 'text',
      });

      // Populate sender info
      const messageWithSender = await this.chatService.getMessageWithSender(message.id);

      // Emit to all room members
      this.server.to(`room:${data.roomId}`).emit('message:new', messageWithSender);

      return { success: true, message: messageWithSender };
    } catch (error) {
      client.emit('error', { message: error.message });
      return { success: false, error: error.message };
    }
  }

  // Edit message
  @SubscribeMessage('message:edit')
  async handleEditMessage(
    @MessageBody() data: { messageId: string; content: string },
    @ConnectedSocket() client: Socket,
    @CurrentUser() user: User,
  ) {
    try {
      const message = await this.chatService.getMessageById(data.messageId);
      
      // Verify ownership
      if (message.senderId !== user.id) {
        throw new Error('Cannot edit message');
      }

      // Update message
      const updated = await this.chatService.editMessage(data.messageId, data.content);

      // Notify room
      this.server.to(`room:${message.roomId}`).emit('message:edited', updated);

      return { success: true };
    } catch (error) {
      client.emit('error', { message: error.message });
      return { success: false, error: error.message };
    }
  }

  // Delete message
  @SubscribeMessage('message:delete')
  async handleDeleteMessage(
    @MessageBody() data: { messageId: string },
    @ConnectedSocket() client: Socket,
    @CurrentUser() user: User,
  ) {
    try {
      const message = await this.chatService.getMessageById(data.messageId);
      
      // Verify ownership or admin
      if (message.senderId !== user.id && user.role !== Role.ADMIN) {
        throw new Error('Cannot delete message');
      }

      await this.chatService.deleteMessage(data.messageId);

      // Notify room
      this.server.to(`room:${message.roomId}`).emit('message:deleted', {
        messageId: data.messageId,
      });

      return { success: true };
    } catch (error) {
      client.emit('error', { message: error.message });
      return { success: false, error: error.message };
    }
  }

  // User is typing
  @SubscribeMessage('typing:start')
  handleTypingStart(
    @MessageBody() data: { roomId: string },
    @ConnectedSocket() client: Socket,
    @CurrentUser() user: User,
  ) {
    client.to(`room:${data.roomId}`).emit('typing:start', {
      userId: user.id,
      username: user.username,
      roomId: data.roomId,
    });
  }

  @SubscribeMessage('typing:stop')
  handleTypingStop(
    @MessageBody() data: { roomId: string },
    @ConnectedSocket() client: Socket,
    @CurrentUser() user: User,
  ) {
    client.to(`room:${data.roomId}`).emit('typing:stop', {
      userId: user.id,
      roomId: data.roomId,
    });
  }

  // Join room
  @SubscribeMessage('room:join')
  async handleJoinRoom(
    @MessageBody() data: { roomId: string },
    @ConnectedSocket() client: Socket,
    @CurrentUser() user: User,
  ) {
    try {
      // Add user to room members
      await this.roomService.addMember(data.roomId, user.id);

      // Join socket room
      client.join(`room:${data.roomId}`);

      // Notify others
      this.server.to(`room:${data.roomId}`).emit('room:user-joined', {
        userId: user.id,
        username: user.username,
        roomId: data.roomId,
      });

      return { success: true };
    } catch (error) {
      return { success: false, error: error.message };
    }
  }

  // Leave room
  @SubscribeMessage('room:leave')
  async handleLeaveRoom(
    @MessageBody() data: { roomId: string },
    @ConnectedSocket() client: Socket,
    @CurrentUser() user: User,
  ) {
    try {
      await this.roomService.removeMember(data.roomId, user.id);
      client.leave(`room:${data.roomId}`);

      this.server.to(`room:${data.roomId}`).emit('room:user-left', {
        userId: user.id,
        username: user.username,
        roomId: data.roomId,
      });

      return { success: true };
    } catch (error) {
      return { success: false, error: error.message };
    }
  }

  // Send typing notification to specific room
  sendTypingNotification(roomId: string, userId: string, isTyping: boolean) {
    const event = isTyping ? 'typing:start' : 'typing:stop';
    this.server.to(`room:${roomId}`).emit(event, { userId, roomId });
  }
}
```

**4. WebSocket JWT Guard:**

```typescript
// auth/guards/ws-jwt.guard.ts
import { CanActivate, ExecutionContext, Injectable } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { WsException } from '@nestjs/websockets';
import { Socket } from 'socket.io';
import { UsersService } from '../../users/users.service';

@Injectable()
export class WsJwtGuard implements CanActivate {
  constructor(
    private jwtService: JwtService,
    private usersService: UsersService,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    try {
      const client: Socket = context.switchToWs().getClient();
      
      // Extract token from handshake auth or query
      const token =
        client.handshake.auth?.token ||
        client.handshake.headers?.authorization?.replace('Bearer ', '');

      if (!token) {
        throw new WsException('Unauthorized');
      }

      // Verify token
      const payload = this.jwtService.verify(token);
      
      // Get user
      const user = await this.usersService.findById(payload.sub);
      
      if (!user) {
        throw new WsException('User not found');
      }

      // Attach user to socket
      client.data.user = user;

      return true;
    } catch (error) {
      throw new WsException('Unauthorized');
    }
  }
}
```

**5. Chat Service:**

```typescript
// chat/chat.service.ts
@Injectable()
export class ChatService {
  constructor(
    @InjectRepository(Message)
    private messageRepository: Repository<Message>,
    @InjectRepository(Room)
    private roomRepository: Repository<Room>,
  ) {}

  async createMessage(data: CreateMessageDto): Promise<Message> {
    const message = this.messageRepository.create(data);
    return this.messageRepository.save(message);
  }

  async getMessageById(id: string): Promise<Message> {
    return this.messageRepository.findOne({
      where: { id },
      relations: ['sender', 'room'],
    });
  }

  async getMessageWithSender(id: string): Promise<Message> {
    return this.messageRepository.findOne({
      where: { id },
      relations: ['sender'],
      select: {
        sender: {
          id: true,
          username: true,
          email: true,
          avatar: true,
        },
      },
    });
  }

  async getRoomMessages(
    roomId: string,
    page: number = 1,
    limit: number = 50,
  ): Promise<PaginatedResponse<Message>> {
    const [messages, total] = await this.messageRepository.findAndCount({
      where: { roomId, isDeleted: false },
      relations: ['sender'],
      order: { createdAt: 'DESC' },
      skip: (page - 1) * limit,
      take: limit,
    });

    return {
      data: messages.reverse(), // Oldest first
      meta: {
        page,
        limit,
        total,
        totalPages: Math.ceil(total / limit),
      },
    };
  }

  async editMessage(id: string, content: string): Promise<Message> {
    await this.messageRepository.update(id, {
      content,
      isEdited: true,
      editedAt: new Date(),
    });
    
    return this.getMessageWithSender(id);
  }

  async deleteMessage(id: string): Promise<void> {
    await this.messageRepository.update(id, { isDeleted: true });
  }

  async searchMessages(roomId: string, query: string): Promise<Message[]> {
    return this.messageRepository
      .createQueryBuilder('message')
      .leftJoinAndSelect('message.sender', 'sender')
      .where('message.roomId = :roomId', { roomId })
      .andWhere('message.isDeleted = false')
      .andWhere('message.content ILIKE :query', { query: `%${query}%` })
      .orderBy('message.createdAt', 'DESC')
      .take(50)
      .getMany();
  }
}
```

**6. Room Service:**

```typescript
// chat/room.service.ts
@Injectable()
export class RoomService {
  constructor(
    @InjectRepository(Room)
    private roomRepository: Repository<Room>,
  ) {}

  async createRoom(data: CreateRoomDto, creatorId: string): Promise<Room> {
    const room = this.roomRepository.create({
      ...data,
      createdBy: { id: creatorId },
      members: [{ id: creatorId }, ...data.memberIds.map(id => ({ id }))],
    });

    return this.roomRepository.save(room);
  }

  async getUserRooms(userId: string): Promise<Room[]> {
    return this.roomRepository
      .createQueryBuilder('room')
      .leftJoin('room.members', 'member')
      .where('member.id = :userId', { userId })
      .getMany();
  }

  async getRoom(id: string): Promise<Room> {
    return this.roomRepository.findOne({
      where: { id },
      relations: ['members', 'createdBy'],
    });
  }

  async isMember(roomId: string, userId: string): Promise<boolean> {
    const room = await this.roomRepository
      .createQueryBuilder('room')
      .leftJoin('room.members', 'member')
      .where('room.id = :roomId', { roomId })
      .andWhere('member.id = :userId', { userId })
      .getOne();

    return !!room;
  }

  async addMember(roomId: string, userId: string): Promise<void> {
    const room = await this.getRoom(roomId);
    
    if (!room.members.find(m => m.id === userId)) {
      room.members.push({ id: userId } as User);
      await this.roomRepository.save(room);
    }
  }

  async removeMember(roomId: string, userId: string): Promise<void> {
    const room = await this.getRoom(roomId);
    room.members = room.members.filter(m => m.id !== userId);
    await this.roomRepository.save(room);
  }

  // Find or create private room between two users
  async getOrCreatePrivateRoom(user1Id: string, user2Id: string): Promise<Room> {
    // Check if private room exists
    const existingRoom = await this.roomRepository
      .createQueryBuilder('room')
      .leftJoin('room.members', 'member')
      .where('room.type = :type', { type: 'private' })
      .andWhere('member.id IN (:...userIds)', { userIds: [user1Id, user2Id] })
      .groupBy('room.id')
      .having('COUNT(DISTINCT member.id) = 2')
      .getOne();

    if (existingRoom) {
      return existingRoom;
    }

    // Create new private room
    return this.createRoom(
      {
        name: `Private Chat`,
        type: 'private',
        memberIds: [user2Id],
      },
      user1Id,
    );
  }
}
```

**7. REST Controller for Room Management:**

```typescript
// chat/chat.controller.ts
@Controller('chat')
@UseGuards(JwtAuthGuard)
export class ChatController {
  constructor(
    private chatService: ChatService,
    private roomService: RoomService,
  ) {}

  @Post('rooms')
  async createRoom(@Body() dto: CreateRoomDto, @CurrentUser() user: User) {
    return this.roomService.createRoom(dto, user.id);
  }

  @Get('rooms')
  async getUserRooms(@CurrentUser() user: User) {
    return this.roomService.getUserRooms(user.id);
  }

  @Get('rooms/:id')
  async getRoom(@Param('id') id: string, @CurrentUser() user: User) {
    const isMember = await this.roomService.isMember(id, user.id);
    
    if (!isMember) {
      throw new ForbiddenException('Not a member of this room');
    }

    return this.roomService.getRoom(id);
  }

  @Get('rooms/:id/messages')
  async getRoomMessages(
    @Param('id') id: string,
    @Query() query: PaginationDto,
    @CurrentUser() user: User,
  ) {
    const isMember = await this.roomService.isMember(id, user.id);
    
    if (!isMember) {
      throw new ForbiddenException('Not a member of this room');
    }

    return this.chatService.getRoomMessages(id, query.page, query.limit);
  }

  @Post('rooms/private')
  async getOrCreatePrivateRoom(
    @Body() dto: { userId: string },
    @CurrentUser() user: User,
  ) {
    return this.roomService.getOrCreatePrivateRoom(user.id, dto.userId);
  }

  @Get('rooms/:id/search')
  async searchMessages(
    @Param('id') id: string,
    @Query('q') query: string,
    @CurrentUser() user: User,
  ) {
    const isMember = await this.roomService.isMember(id, user.id);
    
    if (!isMember) {
      throw new ForbiddenException('Not a member of this room');
    }

    return this.chatService.searchMessages(id, query);
  }
}
```

**8. Client-Side Connection (React Example):**

```typescript
// Frontend example using socket.io-client
import { io } from 'socket.io-client';

const socket = io('http://localhost:3000/chat', {
  auth: {
    token: localStorage.getItem('access_token'),
  },
});

// Connect
socket.on('connect', () => {
  console.log('Connected to chat server');
});

// Listen for new messages
socket.on('message:new', (message) => {
  console.log('New message:', message);
  // Update UI
});

// Listen for user status
socket.on('user:online', ({ userId, username }) => {
  console.log(`${username} is online`);
});

// Send message
function sendMessage(roomId: string, content: string) {
  socket.emit('message:send', { roomId, content }, (response) => {
    if (response.success) {
      console.log('Message sent');
    }
  });
}

// Typing indicator
let typingTimeout;
function handleTyping(roomId: string) {
  socket.emit('typing:start', { roomId });
  
  clearTimeout(typingTimeout);
  typingTimeout = setTimeout(() => {
    socket.emit('typing:stop', { roomId });
  }, 1000);
}
```

**Key Takeaway:** Chat application uses @nestjs/websockets with Socket.IO, WebSocket gateway handles connections and message events, WsJwtGuard authenticates WebSocket connections via handshake token, rooms implemented with Socket.IO room feature (client.join/leave), message persistence with TypeORM entities (Message, Room), real-time events (message:new, typing:start, user:online), track online users with Map, REST API for room management and message history with pagination, support private (1-on-1) and group rooms, message edit/delete with ownership checks, search messages with ILIKE query.

</details>

<details>
<summary><strong>14. How would you build a notification system?</strong></summary>

**Answer:**

Build a scalable notification system with multiple channels (in-app, email, push):

**1. Install Dependencies:**

```bash
npm install @nestjs/bull bull
npm install @nestjs/cache-manager cache-manager
npm install firebase-admin # For push notifications
npm install nodemailer # For emails
```

**2. Notification Entity:**

```typescript
// notifications/entities/notification.entity.ts
@Entity('notifications')
export class Notification {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @ManyToOne(() => User, user => user.notifications)
  @JoinColumn({ name: 'user_id' })
  user: User;

  @Column({ name: 'user_id' })
  userId: string;

  @Column()
  title: string;

  @Column('text')
  message: string;

  @Column({
    type: 'enum',
    enum: ['info', 'success', 'warning', 'error'],
    default: 'info',
  })
  type: string;

  @Column({
    type: 'enum',
    enum: ['new_message', 'new_follower', 'post_liked', 'comment', 'system'],
  })
  category: string;

  @Column('json', { nullable: true })
  data: Record<string, any>; // Additional data (e.g., postId, userId)

  @Column({ default: false })
  read: boolean;

  @Column({ nullable: true, type: 'timestamp' })
  readAt: Date;

  @Column({ default: false })
  sent: boolean; // For external notifications (email, push)

  @Column('simple-array', { default: '' })
  channels: string[]; // ['in-app', 'email', 'push']

  @Column({ nullable: true })
  actionUrl: string; // Deep link or URL

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}

// notifications/entities/notification-preference.entity.ts
@Entity('notification_preferences')
export class NotificationPreference {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @OneToOne(() => User)
  @JoinColumn({ name: 'user_id' })
  user: User;

  @Column({ name: 'user_id', unique: true })
  userId: string;

  @Column({ default: true })
  inAppEnabled: boolean;

  @Column({ default: true })
  emailEnabled: boolean;

  @Column({ default: true })
  pushEnabled: boolean;

  @Column('json', { default: {} })
  categoryPreferences: Record<string, { email: boolean; push: boolean }>; // Category-specific

  @Column({ default: false })
  emailDigest: boolean; // Send email digest instead of individual emails

  @Column({ default: 'daily' })
  digestFrequency: string; // 'daily', 'weekly'

  @UpdateDateColumn()
  updatedAt: Date;
}
```

**3. Notification Service:**

```typescript
// notifications/notification.service.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { InjectQueue } from '@nestjs/bull';
import { Queue } from 'bull';

@Injectable()
export class NotificationService {
  constructor(
    @InjectRepository(Notification)
    private notificationRepository: Repository<Notification>,
    @InjectRepository(NotificationPreference)
    private preferenceRepository: Repository<NotificationPreference>,
    @InjectQueue('notifications')
    private notificationQueue: Queue,
  ) {}

  async create(data: CreateNotificationDto): Promise<Notification> {
    // Get user preferences
    const preferences = await this.getUserPreferences(data.userId);

    // Determine channels based on preferences
    const channels = this.determineChannels(data.category, preferences);

    // Create notification
    const notification = this.notificationRepository.create({
      ...data,
      channels,
    });

    await this.notificationRepository.save(notification);

    // Queue for sending
    await this.queueNotification(notification);

    return notification;
  }

  async createMany(notifications: CreateNotificationDto[]): Promise<void> {
    // Bulk create notifications
    const entities = await Promise.all(
      notifications.map(async (data) => {
        const preferences = await this.getUserPreferences(data.userId);
        const channels = this.determineChannels(data.category, preferences);
        return this.notificationRepository.create({ ...data, channels });
      }),
    );

    await this.notificationRepository.save(entities);

    // Queue all notifications
    await Promise.all(entities.map(n => this.queueNotification(n)));
  }

  private async queueNotification(notification: Notification) {
    // Add to queue for async processing
    if (notification.channels.includes('email')) {
      await this.notificationQueue.add('send-email', { notificationId: notification.id });
    }
    
    if (notification.channels.includes('push')) {
      await this.notificationQueue.add('send-push', { notificationId: notification.id });
    }

    // In-app notifications don't need queuing (already in DB)
  }

  async getUserNotifications(
    userId: string,
    options: { page: number; limit: number; unreadOnly?: boolean },
  ): Promise<PaginatedResponse<Notification>> {
    const query = this.notificationRepository
      .createQueryBuilder('notification')
      .where('notification.userId = :userId', { userId })
      .orderBy('notification.createdAt', 'DESC');

    if (options.unreadOnly) {
      query.andWhere('notification.read = false');
    }

    const [notifications, total] = await query
      .skip((options.page - 1) * options.limit)
      .take(options.limit)
      .getManyAndCount();

    return {
      data: notifications,
      meta: {
        page: options.page,
        limit: options.limit,
        total,
        totalPages: Math.ceil(total / options.limit),
      },
    };
  }

  async markAsRead(id: string, userId: string): Promise<void> {
    await this.notificationRepository.update(
      { id, userId },
      { read: true, readAt: new Date() },
    );
  }

  async markAllAsRead(userId: string): Promise<void> {
    await this.notificationRepository.update(
      { userId, read: false },
      { read: true, readAt: new Date() },
    );
  }

  async getUnreadCount(userId: string): Promise<number> {
    return this.notificationRepository.count({
      where: { userId, read: false },
    });
  }

  async deleteNotification(id: string, userId: string): Promise<void> {
    await this.notificationRepository.delete({ id, userId });
  }

  async getUserPreferences(userId: string): Promise<NotificationPreference> {
    let preferences = await this.preferenceRepository.findOne({
      where: { userId },
    });

    if (!preferences) {
      // Create default preferences
      preferences = this.preferenceRepository.create({
        userId,
        inAppEnabled: true,
        emailEnabled: true,
        pushEnabled: true,
        categoryPreferences: {},
      });
      await this.preferenceRepository.save(preferences);
    }

    return preferences;
  }

  async updatePreferences(
    userId: string,
    data: UpdatePreferencesDto,
  ): Promise<NotificationPreference> {
    const preferences = await this.getUserPreferences(userId);
    Object.assign(preferences, data);
    return this.preferenceRepository.save(preferences);
  }

  private determineChannels(
    category: string,
    preferences: NotificationPreference,
  ): string[] {
    const channels: string[] = [];

    if (preferences.inAppEnabled) {
      channels.push('in-app');
    }

    // Check category-specific preferences
    const categoryPref = preferences.categoryPreferences[category];

    if (preferences.emailEnabled && categoryPref?.email !== false) {
      if (!preferences.emailDigest) {
        channels.push('email');
      }
    }

    if (preferences.pushEnabled && categoryPref?.push !== false) {
      channels.push('push');
    }

    return channels;
  }

  // Helper method to notify multiple users
  async notifyUsers(
    userIds: string[],
    notification: Omit<CreateNotificationDto, 'userId'>,
  ): Promise<void> {
    const notifications = userIds.map(userId => ({
      ...notification,
      userId,
    }));

    await this.createMany(notifications);
  }
}
```

**4. Notification Queue Processor:**

```typescript
// notifications/processors/notification.processor.ts
import { Process, Processor } from '@nestjs/bull';
import { Job } from 'bull';

@Processor('notifications')
export class NotificationProcessor {
  constructor(
    private emailService: EmailService,
    private pushService: PushNotificationService,
    private notificationRepository: Repository<Notification>,
  ) {}

  @Process('send-email')
  async handleEmailNotification(job: Job<{ notificationId: string }>) {
    const notification = await this.notificationRepository.findOne({
      where: { id: job.data.notificationId },
      relations: ['user'],
    });

    if (!notification || notification.sent) {
      return;
    }

    try {
      await this.emailService.sendNotificationEmail(notification);
      
      // Mark as sent
      await this.notificationRepository.update(notification.id, { sent: true });
    } catch (error) {
      console.error('Failed to send email notification:', error);
      throw error; // Bull will retry
    }
  }

  @Process('send-push')
  async handlePushNotification(job: Job<{ notificationId: string }>) {
    const notification = await this.notificationRepository.findOne({
      where: { id: job.data.notificationId },
      relations: ['user'],
    });

    if (!notification) {
      return;
    }

    try {
      await this.pushService.sendPushNotification(notification);
    } catch (error) {
      console.error('Failed to send push notification:', error);
      throw error;
    }
  }
}
```

**5. Push Notification Service (Firebase):**

```typescript
// notifications/services/push-notification.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import * as admin from 'firebase-admin';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class PushNotificationService implements OnModuleInit {
  constructor(private configService: ConfigService) {}

  onModuleInit() {
    // Initialize Firebase Admin
    admin.initializeApp({
      credential: admin.credential.cert({
        projectId: this.configService.get('FIREBASE_PROJECT_ID'),
        privateKey: this.configService.get('FIREBASE_PRIVATE_KEY')?.replace(/\\n/g, '\n'),
        clientEmail: this.configService.get('FIREBASE_CLIENT_EMAIL'),
      }),
    });
  }

  async sendPushNotification(notification: Notification): Promise<void> {
    const user = notification.user;
    
    // Get user's FCM tokens (stored in user profile or separate table)
    const tokens = await this.getUserTokens(user.id);

    if (tokens.length === 0) {
      return;
    }

    const message = {
      notification: {
        title: notification.title,
        body: notification.message,
      },
      data: {
        notificationId: notification.id,
        category: notification.category,
        actionUrl: notification.actionUrl || '',
        ...notification.data,
      },
      tokens,
    };

    try {
      const response = await admin.messaging().sendMulticast(message);
      
      console.log(`Push notifications sent: ${response.successCount} successful, ${response.failureCount} failed`);

      // Remove invalid tokens
      if (response.failureCount > 0) {
        const tokensToRemove: string[] = [];
        response.responses.forEach((resp, idx) => {
          if (!resp.success && resp.error?.code === 'messaging/invalid-registration-token') {
            tokensToRemove.push(tokens[idx]);
          }
        });

        if (tokensToRemove.length > 0) {
          await this.removeUserTokens(user.id, tokensToRemove);
        }
      }
    } catch (error) {
      console.error('Failed to send push notification:', error);
      throw error;
    }
  }

  private async getUserTokens(userId: string): Promise<string[]> {
    // Fetch FCM tokens for user from database
    // Implementation depends on your user devices tracking
    return [];
  }

  private async removeUserTokens(userId: string, tokens: string[]): Promise<void> {
    // Remove invalid tokens from database
  }
}
```

**6. Notification Gateway (Real-time):**

```typescript
// notifications/notification.gateway.ts
@WebSocketGateway({ namespace: '/notifications' })
@UseGuards(WsJwtGuard)
export class NotificationGateway implements OnGatewayConnection {
  @WebSocketServer()
  server: Server;

  private userSockets = new Map<string, string>(); // userId -> socketId

  handleConnection(client: Socket) {
    const user = client.data.user;
    if (user) {
      this.userSockets.set(user.id, client.id);
      client.join(`user:${user.id}`);
    }
  }

  handleDisconnect(client: Socket) {
    const user = client.data.user;
    if (user) {
      this.userSockets.delete(user.id);
    }
  }

  // Send notification to specific user in real-time
  sendToUser(userId: string, notification: Notification) {
    this.server.to(`user:${userId}`).emit('notification:new', notification);
  }

  // Broadcast to multiple users
  sendToUsers(userIds: string[], notification: any) {
    userIds.forEach(userId => {
      this.server.to(`user:${userId}`).emit('notification:new', notification);
    });
  }
}
```

**7. Notification Controller:**

```typescript
// notifications/notification.controller.ts
@Controller('notifications')
@UseGuards(JwtAuthGuard)
export class NotificationController {
  constructor(
    private notificationService: NotificationService,
    private notificationGateway: NotificationGateway,
  ) {}

  @Get()
  async getNotifications(
    @CurrentUser() user: User,
    @Query() query: PaginationDto,
    @Query('unreadOnly') unreadOnly?: boolean,
  ) {
    return this.notificationService.getUserNotifications(user.id, {
      page: query.page || 1,
      limit: query.limit || 20,
      unreadOnly: unreadOnly === 'true',
    });
  }

  @Get('unread-count')
  async getUnreadCount(@CurrentUser() user: User) {
    const count = await this.notificationService.getUnreadCount(user.id);
    return { count };
  }

  @Patch(':id/read')
  async markAsRead(@Param('id') id: string, @CurrentUser() user: User) {
    await this.notificationService.markAsRead(id, user.id);
    return { success: true };
  }

  @Post('mark-all-read')
  async markAllAsRead(@CurrentUser() user: User) {
    await this.notificationService.markAllAsRead(user.id);
    return { success: true };
  }

  @Delete(':id')
  async deleteNotification(@Param('id') id: string, @CurrentUser() user: User) {
    await this.notificationService.deleteNotification(id, user.id);
    return { success: true };
  }

  @Get('preferences')
  async getPreferences(@CurrentUser() user: User) {
    return this.notificationService.getUserPreferences(user.id);
  }

  @Patch('preferences')
  async updatePreferences(
    @CurrentUser() user: User,
    @Body() dto: UpdatePreferencesDto,
  ) {
    return this.notificationService.updatePreferences(user.id, dto);
  }
}
```

**8. Usage Example - Triggering Notifications:**

```typescript
// posts/posts.service.ts
@Injectable()
export class PostsService {
  constructor(
    private notificationService: NotificationService,
    private notificationGateway: NotificationGateway,
  ) {}

  async likePost(postId: string, userId: string) {
    const post = await this.findOne(postId);
    
    // Create notification for post author
    if (post.authorId !== userId) {
      const notification = await this.notificationService.create({
        userId: post.authorId,
        title: 'New Like',
        message: `Someone liked your post`,
        type: 'info',
        category: 'post_liked',
        data: { postId, likerId: userId },
        actionUrl: `/posts/${postId}`,
      });

      // Send real-time notification
      this.notificationGateway.sendToUser(post.authorId, notification);
    }

    // ... rest of like logic
  }

  async createComment(postId: string, userId: string, content: string) {
    const post = await this.findOne(postId);
    
    // Notify post author
    if (post.authorId !== userId) {
      const notification = await this.notificationService.create({
        userId: post.authorId,
        title: 'New Comment',
        message: `Someone commented on your post`,
        type: 'info',
        category: 'comment',
        data: { postId, commenterId: userId },
        actionUrl: `/posts/${postId}`,
      });

      this.notificationGateway.sendToUser(post.authorId, notification);
    }

    // ... rest of comment logic
  }
}
```

**9. Notification Module:**

```typescript
// notifications/notification.module.ts
import { BullModule } from '@nestjs/bull';

@Module({
  imports: [
    TypeOrmModule.forFeature([Notification, NotificationPreference]),
    BullModule.registerQueue({
      name: 'notifications',
    }),
  ],
  providers: [
    NotificationService,
    NotificationProcessor,
    PushNotificationService,
    EmailService,
    NotificationGateway,
  ],
  controllers: [NotificationController],
  exports: [NotificationService, NotificationGateway],
})
export class NotificationModule {}
```

**Key Takeaway:** Notification system supports multiple channels (in-app, email, push via Firebase), uses Bull queue for async processing, stores notifications in database with Notification entity, user preferences control channels per category, WebSocket gateway for real-time delivery, tracks read/unread status, supports bulk notifications with notifyUsers(), email digest option, push notifications remove invalid FCM tokens, Bull processor handles retries on failure, expose REST API for fetching/marking notifications.

</details>

<details>
<parameter name="<strong">15. How would you build a file upload/download service?</strong></summary>

**Answer:**

Build a secure file upload/download service with storage options (local, S3), virus scanning, and streaming:

**1. Install Dependencies:**

```bash
npm install @nestjs/platform-express multer
npm install aws-sdk @aws-sdk/client-s3 @aws-sdk/s3-request-presigner
npm install file-type mime-types
npm install clamscan # For virus scanning
```

**2. File Entity:**

```typescript
// files/entities/file.entity.ts
@Entity('files')
export class File {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  filename: string;

  @Column()
  originalName: string;

  @Column()
  mimeType: string;

  @Column({ type: 'bigint' })
  size: number; // in bytes

  @Column()
  path: string; // Local path or S3 key

  @Column({ type: 'enum', enum: ['local', 's3', 'gcs'], default: 'local' })
  storage: string;

  @Column({ nullable: true })
  url: string; // Public URL if applicable

  @ManyToOne(() => User, user => user.uploadedFiles)
  @JoinColumn({ name: 'uploaded_by' })
  uploadedBy: User;

  @Column({ name: 'uploaded_by' })
  uploadedById: string;

  @Column({ default: false })
  isPublic: boolean;

  @Column({ nullable: true })
  folder: string; // Logical folder/category

  @Column({ nullable: true })
  description: string;

  @Column({ type: 'enum', enum: ['pending', 'scanning', 'safe', 'infected'], default: 'pending' })
  scanStatus: string;

  @Column({ nullable: true, type: 'timestamp' })
  scannedAt: Date;

  @Column({ default: 0 })
  downloadCount: number;

  @Column({ nullable: true, type: 'timestamp' })
  expiresAt: Date; // For temporary files

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}
```

**3. File Upload Service:**

```typescript
// files/file-upload.service.ts
import { Injectable, BadRequestException, NotFoundException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { S3Client, PutObjectCommand, GetObjectCommand, DeleteObjectCommand } from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';
import { ConfigService } from '@nestjs/config';
import * as fs from 'fs';
import * as path from 'path';
import * as crypto from 'crypto';
import { fileTypeFromBuffer } from 'file-type';

@Injectable()
export class FileUploadService {
  private s3Client: S3Client;
  private readonly uploadDir = './uploads';

  constructor(
    @InjectRepository(File)
    private fileRepository: Repository<File>,
    private configService: ConfigService,
  ) {
    // Initialize S3 client
    this.s3Client = new S3Client({
      region: configService.get('AWS_REGION'),
      credentials: {
        accessKeyId: configService.get('AWS_ACCESS_KEY_ID'),
        secretAccessKey: configService.get('AWS_SECRET_ACCESS_KEY'),
      },
    });

    // Ensure upload directory exists
    if (!fs.existsSync(this.uploadDir)) {
      fs.mkdirSync(this.uploadDir, { recursive: true });
    }
  }

  async uploadFile(
    file: Express.Multer.File,
    userId: string,
    options: {
      storage?: 'local' | 's3';
      isPublic?: boolean;
      folder?: string;
      expiresIn?: number; // Days
    } = {},
  ): Promise<File> {
    const storage = options.storage || this.configService.get('DEFAULT_STORAGE', 'local');

    // Validate file
    await this.validateFile(file);

    // Generate unique filename
    const filename = this.generateFilename(file.originalname);
    
    let filePath: string;
    let url: string | null = null;

    // Upload based on storage type
    if (storage === 's3') {
      filePath = await this.uploadToS3(file, filename, options.folder);
      
      if (options.isPublic) {
        url = this.getS3PublicUrl(filePath);
      }
    } else {
      filePath = await this.uploadToLocal(file, filename, options.folder);
      
      if (options.isPublic) {
        url = this.getLocalPublicUrl(filePath);
      }
    }

    // Calculate expiration
    const expiresAt = options.expiresIn 
      ? new Date(Date.now() + options.expiresIn * 24 * 60 * 60 * 1000)
      : null;

    // Save to database
    const fileEntity = this.fileRepository.create({
      filename,
      originalName: file.originalname,
      mimeType: file.mimetype,
      size: file.size,
      path: filePath,
      storage,
      url,
      uploadedById: userId,
      isPublic: options.isPublic || false,
      folder: options.folder,
      expiresAt,
      scanStatus: 'pending',
    });

    await this.fileRepository.save(fileEntity);

    // Queue for virus scanning
    // await this.virusScanQueue.add('scan', { fileId: fileEntity.id });

    return fileEntity;
  }

  async uploadMultiple(
    files: Express.Multer.File[],
    userId: string,
    options: any = {},
  ): Promise<File[]> {
    const uploadPromises = files.map(file => this.uploadFile(file, userId, options));
    return Promise.all(uploadPromises);
  }

  private async uploadToS3(
    file: Express.Multer.File,
    filename: string,
    folder?: string,
  ): Promise<string> {
    const key = folder ? `${folder}/${filename}` : filename;

    const command = new PutObjectCommand({
      Bucket: this.configService.get('AWS_S3_BUCKET'),
      Key: key,
      Body: file.buffer,
      ContentType: file.mimetype,
      // ACL: 'public-read', // Only if public
    });

    await this.s3Client.send(command);

    return key;
  }

  private async uploadToLocal(
    file: Express.Multer.File,
    filename: string,
    folder?: string,
  ): Promise<string> {
    const dir = folder ? path.join(this.uploadDir, folder) : this.uploadDir;

    // Create directory if doesn't exist
    if (!fs.existsSync(dir)) {
      fs.mkdirSync(dir, { recursive: true });
    }

    const filePath = path.join(dir, filename);
    
    // Write file
    fs.writeFileSync(filePath, file.buffer);

    return path.relative(this.uploadDir, filePath);
  }

  async downloadFile(fileId: string, userId: string): Promise<{ stream: any; file: File }> {
    const file = await this.fileRepository.findOne({
      where: { id: fileId },
    });

    if (!file) {
      throw new NotFoundException('File not found');
    }

    // Check permissions
    if (!file.isPublic && file.uploadedById !== userId) {
      throw new BadRequestException('Access denied');
    }

    // Increment download count
    await this.fileRepository.increment({ id: fileId }, 'downloadCount', 1);

    // Get file stream
    let stream;
    
    if (file.storage === 's3') {
      stream = await this.downloadFromS3(file.path);
    } else {
      stream = await this.downloadFromLocal(file.path);
    }

    return { stream, file };
  }

  private async downloadFromS3(key: string): Promise<any> {
    const command = new GetObjectCommand({
      Bucket: this.configService.get('AWS_S3_BUCKET'),
      Key: key,
    });

    const response = await this.s3Client.send(command);
    return response.Body;
  }

  private async downloadFromLocal(filePath: string): Promise<fs.ReadStream> {
    const fullPath = path.join(this.uploadDir, filePath);
    
    if (!fs.existsSync(fullPath)) {
      throw new NotFoundException('File not found on disk');
    }

    return fs.createReadStream(fullPath);
  }

  async getSignedDownloadUrl(fileId: string, userId: string, expiresIn = 3600): Promise<string> {
    const file = await this.fileRepository.findOne({
      where: { id: fileId },
    });

    if (!file) {
      throw new NotFoundException('File not found');
    }

    // Check permissions
    if (!file.isPublic && file.uploadedById !== userId) {
      throw new BadRequestException('Access denied');
    }

    if (file.storage === 's3') {
      const command = new GetObjectCommand({
        Bucket: this.configService.get('AWS_S3_BUCKET'),
        Key: file.path,
      });

      return getSignedUrl(this.s3Client, command, { expiresIn });
    } else {
      // For local files, generate a temporary token
      return this.generateTemporaryDownloadToken(fileId, expiresIn);
    }
  }

  async deleteFile(fileId: string, userId: string): Promise<void> {
    const file = await this.fileRepository.findOne({
      where: { id: fileId },
    });

    if (!file) {
      throw new NotFoundException('File not found');
    }

    // Check ownership
    if (file.uploadedById !== userId) {
      throw new BadRequestException('Access denied');
    }

    // Delete from storage
    if (file.storage === 's3') {
      await this.deleteFromS3(file.path);
    } else {
      await this.deleteFromLocal(file.path);
    }

    // Delete from database
    await this.fileRepository.delete(fileId);
  }

  private async deleteFromS3(key: string): Promise<void> {
    const command = new DeleteObjectCommand({
      Bucket: this.configService.get('AWS_S3_BUCKET'),
      Key: key,
    });

    await this.s3Client.send(command);
  }

  private async deleteFromLocal(filePath: string): Promise<void> {
    const fullPath = path.join(this.uploadDir, filePath);
    
    if (fs.existsSync(fullPath)) {
      fs.unlinkSync(fullPath);
    }
  }

  async getUserFiles(
    userId: string,
    options: { page: number; limit: number; folder?: string },
  ): Promise<PaginatedResponse<File>> {
    const query = this.fileRepository.createQueryBuilder('file')
      .where('file.uploadedById = :userId', { userId })
      .orderBy('file.createdAt', 'DESC');

    if (options.folder) {
      query.andWhere('file.folder = :folder', { folder: options.folder });
    }

    const [files, total] = await query
      .skip((options.page - 1) * options.limit)
      .take(options.limit)
      .getManyAndCount();

    return {
      data: files,
      meta: {
        page: options.page,
        limit: options.limit,
        total,
        totalPages: Math.ceil(total / options.limit),
      },
    };
  }

  private async validateFile(file: Express.Multer.File): Promise<void> {
    const maxSize = this.configService.get('MAX_FILE_SIZE', 10 * 1024 * 1024); // 10MB default

    if (file.size > maxSize) {
      throw new BadRequestException(`File size exceeds ${maxSize / 1024 / 1024}MB limit`);
    }

    // Validate file type from buffer (more secure than mime type)
    const fileType = await fileTypeFromBuffer(file.buffer);
    
    const allowedTypes = this.configService.get('ALLOWED_FILE_TYPES', [
      'image/jpeg',
      'image/png',
      'image/gif',
      'application/pdf',
      'text/plain',
    ]);

    if (fileType && !allowedTypes.includes(fileType.mime)) {
      throw new BadRequestException(`File type ${fileType.mime} not allowed`);
    }
  }

  private generateFilename(originalName: string): string {
    const ext = path.extname(originalName);
    const randomName = crypto.randomBytes(16).toString('hex');
    return `${Date.now()}-${randomName}${ext}`;
  }

  private getS3PublicUrl(key: string): string {
    const bucket = this.configService.get('AWS_S3_BUCKET');
    const region = this.configService.get('AWS_REGION');
    return `https://${bucket}.s3.${region}.amazonaws.com/${key}`;
  }

  private getLocalPublicUrl(filePath: string): string {
    const baseUrl = this.configService.get('APP_URL');
    return `${baseUrl}/files/public/${filePath}`;
  }

  private generateTemporaryDownloadToken(fileId: string, expiresIn: number): string {
    // Generate temporary token for local file downloads
    const token = crypto.randomBytes(32).toString('hex');
    // Store token with expiration in cache/database
    return `${this.configService.get('APP_URL')}/files/download/${fileId}?token=${token}`;
  }

  async cleanupExpiredFiles(): Promise<void> {
    const expiredFiles = await this.fileRepository.find({
      where: {
        expiresAt: LessThan(new Date()),
      },
    });

    for (const file of expiredFiles) {
      try {
        if (file.storage === 's3') {
          await this.deleteFromS3(file.path);
        } else {
          await this.deleteFromLocal(file.path);
        }
        
        await this.fileRepository.delete(file.id);
      } catch (error) {
        console.error(`Failed to delete expired file ${file.id}:`, error);
      }
    }
  }
}
```

**4. File Upload Controller:**

```typescript
// files/files.controller.ts
import {
  Controller,
  Post,
  Get,
  Delete,
  Param,
  Query,
  UseGuards,
  UseInterceptors,
  UploadedFile,
  UploadedFiles,
  Res,
  StreamableFile,
  Body,
} from '@nestjs/common';
import { FileInterceptor, FilesInterceptor } from '@nestjs/platform-express';
import { Response } from 'express';

@Controller('files')
@UseGuards(JwtAuthGuard)
export class FilesController {
  constructor(private fileUploadService: FileUploadService) {}

  @Post('upload')
  @UseInterceptors(
    FileInterceptor('file', {
      limits: {
        fileSize: 10 * 1024 * 1024, // 10MB
      },
    }),
  )
  async uploadFile(
    @UploadedFile() file: Express.Multer.File,
    @CurrentUser() user: User,
    @Body() options: { folder?: string; isPublic?: boolean; expiresIn?: number },
  ) {
    return this.fileUploadService.uploadFile(file, user.id, options);
  }

  @Post('upload-multiple')
  @UseInterceptors(
    FilesInterceptor('files', 10, { // Max 10 files
      limits: {
        fileSize: 10 * 1024 * 1024,
      },
    }),
  )
  async uploadMultiple(
    @UploadedFiles() files: Express.Multer.File[],
    @CurrentUser() user: User,
    @Body() options: any,
  ) {
    return this.fileUploadService.uploadMultiple(files, user.id, options);
  }

  @Get()
  async getUserFiles(
    @CurrentUser() user: User,
    @Query() query: PaginationDto & { folder?: string },
  ) {
    return this.fileUploadService.getUserFiles(user.id, {
      page: query.page || 1,
      limit: query.limit || 20,
      folder: query.folder,
    });
  }

  @Get(':id')
  async getFile(@Param('id') id: string, @CurrentUser() user: User) {
    return this.fileUploadService.getFileById(id, user.id);
  }

  @Get(':id/download')
  async downloadFile(
    @Param('id') id: string,
    @CurrentUser() user: User,
    @Res({ passthrough: true }) res: Response,
  ) {
    const { stream, file } = await this.fileUploadService.downloadFile(id, user.id);

    res.set({
      'Content-Type': file.mimeType,
      'Content-Disposition': `attachment; filename="${file.originalName}"`,
      'Content-Length': file.size,
    });

    return new StreamableFile(stream);
  }

  @Get(':id/signed-url')
  async getSignedUrl(
    @Param('id') id: string,
    @CurrentUser() user: User,
    @Query('expiresIn') expiresIn?: number,
  ) {
    const url = await this.fileUploadService.getSignedDownloadUrl(
      id,
      user.id,
      expiresIn ? parseInt(expiresIn) : 3600,
    );
    
    return { url };
  }

  @Delete(':id')
  async deleteFile(@Param('id') id: string, @CurrentUser() user: User) {
    await this.fileUploadService.deleteFile(id, user.id);
    return { success: true };
  }

  @Get('public/:path(*)')
  @Public()
  async servePublicFile(@Param('path') filePath: string, @Res() res: Response) {
    // Serve public local files
    const fullPath = path.join('./uploads', filePath);
    
    if (!fs.existsSync(fullPath)) {
      throw new NotFoundException('File not found');
    }

    return res.sendFile(fullPath, { root: '.' });
  }
}
```

**5. Virus Scanning Service:**

```typescript
// files/services/virus-scan.service.ts
import { Injectable } from '@nestjs/common';
import NodeClam from 'clamscan';

@Injectable()
export class VirusScanService {
  private clamScan: NodeClam;

  async onModuleInit() {
    this.clamScan = await new NodeClam().init({
      clamdscan: {
        host: 'localhost',
        port: 3310,
      },
    });
  }

  async scanFile(filePath: string): Promise<{ isInfected: boolean; viruses?: string[] }> {
    try {
      const { isInfected, viruses } = await this.clamScan.scanFile(filePath);
      return { isInfected, viruses };
    } catch (error) {
      console.error('Virus scan failed:', error);
      return { isInfected: false }; // Fail open or closed based on policy
    }
  }

  async scanBuffer(buffer: Buffer): Promise<{ isInfected: boolean }> {
    try {
      const { isInfected } = await this.clamScan.scanStream(buffer);
      return { isInfected };
    } catch (error) {
      console.error('Virus scan failed:', error);
      return { isInfected: false };
    }
  }
}
```

**6. Image Processing Service:**

```typescript
// files/services/image-processing.service.ts
import { Injectable } from '@nestjs/common';
import * as sharp from 'sharp';

@Injectable()
export class ImageProcessingService {
  async createThumbnail(
    buffer: Buffer,
    width = 200,
    height = 200,
  ): Promise<Buffer> {
    return sharp(buffer)
      .resize(width, height, {
        fit: 'cover',
        position: 'center',
      })
      .jpeg({ quality: 80 })
      .toBuffer();
  }

  async optimizeImage(buffer: Buffer, quality = 85): Promise<Buffer> {
    return sharp(buffer)
      .jpeg({ quality, progressive: true })
      .toBuffer();
  }

  async convertToWebP(buffer: Buffer): Promise<Buffer> {
    return sharp(buffer).webp({ quality: 90 }).toBuffer();
  }

  async getImageMetadata(buffer: Buffer) {
    return sharp(buffer).metadata();
  }

  async resize(
    buffer: Buffer,
    width: number,
    height?: number,
  ): Promise<Buffer> {
    return sharp(buffer)
      .resize(width, height, {
        fit: 'inside',
        withoutEnlargement: true,
      })
      .toBuffer();
  }
}
```

**7. Chunked Upload for Large Files:**

```typescript
// files/chunked-upload.service.ts
@Injectable()
export class ChunkedUploadService {
  private uploads = new Map<string, {
    chunks: Buffer[];
    metadata: any;
    expiresAt: Date;
  }>();

  async initializeUpload(userId: string, metadata: any): Promise<string> {
    const uploadId = crypto.randomBytes(16).toString('hex');
    
    this.uploads.set(uploadId, {
      chunks: [],
      metadata: { ...metadata, userId },
      expiresAt: new Date(Date.now() + 24 * 60 * 60 * 1000), // 24 hours
    });

    return uploadId;
  }

  async uploadChunk(
    uploadId: string,
    chunkIndex: number,
    chunk: Buffer,
  ): Promise<void> {
    const upload = this.uploads.get(uploadId);
    
    if (!upload) {
      throw new NotFoundException('Upload session not found');
    }

    upload.chunks[chunkIndex] = chunk;
  }

  async completeUpload(
    uploadId: string,
    fileUploadService: FileUploadService,
  ): Promise<File> {
    const upload = this.uploads.get(uploadId);
    
    if (!upload) {
      throw new NotFoundException('Upload session not found');
    }

    // Combine chunks
    const completeBuffer = Buffer.concat(upload.chunks);

    // Create file object
    const file: Express.Multer.File = {
      buffer: completeBuffer,
      originalname: upload.metadata.filename,
      mimetype: upload.metadata.mimeType,
      size: completeBuffer.length,
      fieldname: 'file',
      encoding: '7bit',
      stream: null,
      destination: '',
      filename: '',
      path: '',
    };

    // Upload file
    const result = await fileUploadService.uploadFile(
      file,
      upload.metadata.userId,
      upload.metadata.options || {},
    );

    // Cleanup
    this.uploads.delete(uploadId);

    return result;
  }

  async abortUpload(uploadId: string): Promise<void> {
    this.uploads.delete(uploadId);
  }
}
```

**8. Scheduled Cleanup Task:**

```typescript
// files/tasks/file-cleanup.task.ts
import { Injectable } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';

@Injectable()
export class FileCleanupTask {
  constructor(private fileUploadService: FileUploadService) {}

  @Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT)
  async handleExpiredFiles() {
    console.log('Running file cleanup task...');
    await this.fileUploadService.cleanupExpiredFiles();
    console.log('File cleanup completed');
  }
}
```

**Key Takeaway:** File upload service supports multiple storage backends (local filesystem, AWS S3) with storage enum, uses Multer for multipart/form-data handling, validates files by buffer content with file-type package, generates unique filenames with crypto.randomBytes(), implements streaming downloads with StreamableFile, creates signed URLs for temporary access (S3 presigned URLs), tracks file metadata in database (size, mimeType, uploadedBy), supports virus scanning with ClamAV, image processing with sharp (thumbnails, optimization), chunked uploads for large files, scheduled cleanup of expired files with @Cron(), enforces file size limits and type restrictions, returns public URLs for isPublic files.

</details>

<details>
<summary><strong>16. How would you build a payment processing system?</strong></summary>

**Answer:**

Build a secure payment processing system with Stripe integration, webhook handling, and idempotency:

**1. Install Dependencies:**

```bash
npm install stripe
npm install @nestjs/schedule # For retry jobs
```

**2. Payment Entities:**

```typescript
// payments/entities/payment.entity.ts
@Entity('payments')
export class Payment {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  idempotencyKey: string; // Prevent duplicate charges

  @ManyToOne(() => User, user => user.payments)
  @JoinColumn({ name: 'user_id' })
  user: User;

  @Column({ name: 'user_id' })
  userId: string;

  @Column({ type: 'decimal', precision: 10, scale: 2 })
  amount: number;

  @Column({ length: 3 })
  currency: string; // USD, EUR, GBP

  @Column({
    type: 'enum',
    enum: ['pending', 'processing', 'succeeded', 'failed', 'canceled', 'refunded'],
    default: 'pending',
  })
  status: string;

  @Column({ nullable: true })
  stripePaymentIntentId: string;

  @Column({ nullable: true })
  stripeChargeId: string;

  @Column({ nullable: true })
  stripeCustomerId: string;

  @Column({ type: 'enum', enum: ['card', 'bank_transfer', 'wallet'], default: 'card' })
  paymentMethod: string;

  @Column({ nullable: true })
  last4: string; // Last 4 digits of card

  @Column({ nullable: true })
  cardBrand: string; // visa, mastercard, etc.

  @Column('text', { nullable: true })
  description: string;

  @Column('json', { nullable: true })
  metadata: Record<string, any>; // Order ID, product IDs, etc.

  @Column({ nullable: true })
  failureReason: string;

  @Column({ nullable: true })
  receiptUrl: string;

  @Column({ nullable: true, type: 'timestamp' })
  paidAt: Date;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}

// payments/entities/refund.entity.ts
@Entity('refunds')
export class Refund {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @ManyToOne(() => Payment, payment => payment.refunds)
  @JoinColumn({ name: 'payment_id' })
  payment: Payment;

  @Column({ name: 'payment_id' })
  paymentId: string;

  @Column({ type: 'decimal', precision: 10, scale: 2 })
  amount: number;

  @Column({ nullable: true })
  stripeRefundId: string;

  @Column({ type: 'enum', enum: ['pending', 'succeeded', 'failed'], default: 'pending' })
  status: string;

  @Column('text', { nullable: true })
  reason: string;

  @ManyToOne(() => User)
  @JoinColumn({ name: 'requested_by' })
  requestedBy: User;

  @Column({ name: 'requested_by' })
  requestedById: string;

  @CreateDateColumn()
  createdAt: Date;
}
```

**3. Payment Service:**

```typescript
// payments/payment.service.ts
import { Injectable, BadRequestException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import Stripe from 'stripe';
import { ConfigService } from '@nestjs/config';
import * as crypto from 'crypto';

@Injectable()
export class PaymentService {
  private stripe: Stripe;

  constructor(
    @InjectRepository(Payment)
    private paymentRepository: Repository<Payment>,
    @InjectRepository(Refund)
    private refundRepository: Repository<Refund>,
    private configService: ConfigService,
  ) {
    this.stripe = new Stripe(configService.get('STRIPE_SECRET_KEY'), {
      apiVersion: '2023-10-16',
    });
  }

  async createPaymentIntent(data: CreatePaymentDto, userId: string): Promise<Payment> {
    // Generate idempotency key
    const idempotencyKey = data.idempotencyKey || this.generateIdempotencyKey();

    // Check if payment already exists with this key
    const existing = await this.paymentRepository.findOne({
      where: { idempotencyKey },
    });

    if (existing) {
      return existing; // Return existing payment (idempotency)
    }

    // Get or create Stripe customer
    const customer = await this.getOrCreateCustomer(userId);

    try {
      // Create Stripe Payment Intent
      const paymentIntent = await this.stripe.paymentIntents.create(
        {
          amount: Math.round(data.amount * 100), // Convert to cents
          currency: data.currency.toLowerCase(),
          customer: customer.id,
          description: data.description,
          metadata: data.metadata || {},
          payment_method_types: ['card'],
        },
        {
          idempotencyKey, // Stripe idempotency
        },
      );

      // Save to database
      const payment = this.paymentRepository.create({
        idempotencyKey,
        userId,
        amount: data.amount,
        currency: data.currency,
        status: 'pending',
        stripePaymentIntentId: paymentIntent.id,
        stripeCustomerId: customer.id,
        description: data.description,
        metadata: data.metadata,
      });

      await this.paymentRepository.save(payment);

      return payment;
    } catch (error) {
      console.error('Failed to create payment intent:', error);
      throw new BadRequestException('Failed to create payment');
    }
  }

  async confirmPayment(paymentId: string, paymentMethodId: string): Promise<Payment> {
    const payment = await this.paymentRepository.findOne({
      where: { id: paymentId },
    });

    if (!payment) {
      throw new BadRequestException('Payment not found');
    }

    if (payment.status !== 'pending') {
      throw new BadRequestException('Payment already processed');
    }

    try {
      // Confirm payment intent
      const paymentIntent = await this.stripe.paymentIntents.confirm(
        payment.stripePaymentIntentId,
        {
          payment_method: paymentMethodId,
        },
      );

      // Update payment status
      payment.status = 'processing';
      payment.stripeChargeId = paymentIntent.latest_charge as string;

      if (paymentIntent.status === 'succeeded') {
        payment.status = 'succeeded';
        payment.paidAt = new Date();
        
        // Get payment method details
        const paymentMethod = await this.stripe.paymentMethods.retrieve(paymentMethodId);
        payment.last4 = paymentMethod.card?.last4;
        payment.cardBrand = paymentMethod.card?.brand;
      }

      await this.paymentRepository.save(payment);

      return payment;
    } catch (error) {
      payment.status = 'failed';
      payment.failureReason = error.message;
      await this.paymentRepository.save(payment);

      throw new BadRequestException('Payment failed: ' + error.message);
    }
  }

  async handleWebhook(signature: string, payload: Buffer): Promise<void> {
    const webhookSecret = this.configService.get('STRIPE_WEBHOOK_SECRET');

    let event: Stripe.Event;

    try {
      // Verify webhook signature
      event = this.stripe.webhooks.constructEvent(payload, signature, webhookSecret);
    } catch (error) {
      console.error('Webhook signature verification failed:', error);
      throw new BadRequestException('Invalid signature');
    }

    // Handle different event types
    switch (event.type) {
      case 'payment_intent.succeeded':
        await this.handlePaymentSucceeded(event.data.object as Stripe.PaymentIntent);
        break;

      case 'payment_intent.payment_failed':
        await this.handlePaymentFailed(event.data.object as Stripe.PaymentIntent);
        break;

      case 'charge.refunded':
        await this.handleChargeRefunded(event.data.object as Stripe.Charge);
        break;

      case 'customer.subscription.created':
      case 'customer.subscription.updated':
      case 'customer.subscription.deleted':
        // Handle subscription events
        break;

      default:
        console.log(`Unhandled event type: ${event.type}`);
    }
  }

  private async handlePaymentSucceeded(paymentIntent: Stripe.PaymentIntent): Promise<void> {
    const payment = await this.paymentRepository.findOne({
      where: { stripePaymentIntentId: paymentIntent.id },
    });

    if (!payment) {
      console.error('Payment not found for intent:', paymentIntent.id);
      return;
    }

    payment.status = 'succeeded';
    payment.paidAt = new Date();
    payment.receiptUrl = paymentIntent.charges.data[0]?.receipt_url;
    
    await this.paymentRepository.save(payment);

    // Trigger post-payment actions (e.g., send email, fulfill order)
    // await this.orderService.fulfillOrder(payment.metadata.orderId);
  }

  private async handlePaymentFailed(paymentIntent: Stripe.PaymentIntent): Promise<void> {
    const payment = await this.paymentRepository.findOne({
      where: { stripePaymentIntentId: paymentIntent.id },
    });

    if (!payment) {
      return;
    }

    payment.status = 'failed';
    payment.failureReason = paymentIntent.last_payment_error?.message || 'Unknown error';
    
    await this.paymentRepository.save(payment);
  }

  private async handleChargeRefunded(charge: Stripe.Charge): Promise<void> {
    const payment = await this.paymentRepository.findOne({
      where: { stripeChargeId: charge.id },
    });

    if (!payment) {
      return;
    }

    payment.status = 'refunded';
    await this.paymentRepository.save(payment);
  }

  async refundPayment(
    paymentId: string,
    amount: number,
    reason: string,
    userId: string,
  ): Promise<Refund> {
    const payment = await this.paymentRepository.findOne({
      where: { id: paymentId },
    });

    if (!payment) {
      throw new BadRequestException('Payment not found');
    }

    if (payment.status !== 'succeeded') {
      throw new BadRequestException('Can only refund succeeded payments');
    }

    if (amount > payment.amount) {
      throw new BadRequestException('Refund amount exceeds payment amount');
    }

    try {
      // Create Stripe refund
      const stripeRefund = await this.stripe.refunds.create({
        payment_intent: payment.stripePaymentIntentId,
        amount: Math.round(amount * 100), // Convert to cents
        reason: 'requested_by_customer',
      });

      // Save refund to database
      const refund = this.refundRepository.create({
        paymentId,
        amount,
        reason,
        requestedById: userId,
        stripeRefundId: stripeRefund.id,
        status: stripeRefund.status === 'succeeded' ? 'succeeded' : 'pending',
      });

      await this.refundRepository.save(refund);

      // Update payment status if fully refunded
      if (amount === payment.amount) {
        payment.status = 'refunded';
        await this.paymentRepository.save(payment);
      }

      return refund;
    } catch (error) {
      console.error('Refund failed:', error);
      throw new BadRequestException('Refund failed: ' + error.message);
    }
  }

  async getUserPayments(
    userId: string,
    options: { page: number; limit: number },
  ): Promise<PaginatedResponse<Payment>> {
    const [payments, total] = await this.paymentRepository.findAndCount({
      where: { userId },
      order: { createdAt: 'DESC' },
      skip: (options.page - 1) * options.limit,
      take: options.limit,
    });

    return {
      data: payments,
      meta: {
        page: options.page,
        limit: options.limit,
        total,
        totalPages: Math.ceil(total / options.limit),
      },
    };
  }

  async getPaymentById(id: string, userId: string): Promise<Payment> {
    const payment = await this.paymentRepository.findOne({
      where: { id, userId },
    });

    if (!payment) {
      throw new BadRequestException('Payment not found');
    }

    return payment;
  }

  private async getOrCreateCustomer(userId: string): Promise<Stripe.Customer> {
    // Check if customer already exists in database
    const existingPayment = await this.paymentRepository.findOne({
      where: { userId },
      order: { createdAt: 'DESC' },
    });

    if (existingPayment?.stripeCustomerId) {
      return this.stripe.customers.retrieve(existingPayment.stripeCustomerId) as Promise<Stripe.Customer>;
    }

    // Create new customer
    const user = await this.userRepository.findOne({ where: { id: userId } });
    
    return this.stripe.customers.create({
      email: user.email,
      metadata: { userId },
    });
  }

  private generateIdempotencyKey(): string {
    return crypto.randomBytes(16).toString('hex');
  }

  async cancelPayment(paymentId: string, userId: string): Promise<Payment> {
    const payment = await this.paymentRepository.findOne({
      where: { id: paymentId, userId },
    });

    if (!payment) {
      throw new BadRequestException('Payment not found');
    }

    if (payment.status !== 'pending') {
      throw new BadRequestException('Can only cancel pending payments');
    }

    try {
      await this.stripe.paymentIntents.cancel(payment.stripePaymentIntentId);
      
      payment.status = 'canceled';
      await this.paymentRepository.save(payment);

      return payment;
    } catch (error) {
      throw new BadRequestException('Cancel failed: ' + error.message);
    }
  }
}
```

**4. Payment Controller:**

```typescript
// payments/payment.controller.ts
@Controller('payments')
@UseGuards(JwtAuthGuard)
export class PaymentController {
  constructor(private paymentService: PaymentService) {}

  @Post()
  async createPayment(
    @Body() dto: CreatePaymentDto,
    @CurrentUser() user: User,
  ) {
    return this.paymentService.createPaymentIntent(dto, user.id);
  }

  @Post(':id/confirm')
  async confirmPayment(
    @Param('id') id: string,
    @Body() dto: { paymentMethodId: string },
  ) {
    return this.paymentService.confirmPayment(id, dto.paymentMethodId);
  }

  @Get()
  async getUserPayments(
    @CurrentUser() user: User,
    @Query() query: PaginationDto,
  ) {
    return this.paymentService.getUserPayments(user.id, {
      page: query.page || 1,
      limit: query.limit || 20,
    });
  }

  @Get(':id')
  async getPayment(@Param('id') id: string, @CurrentUser() user: User) {
    return this.paymentService.getPaymentById(id, user.id);
  }

  @Post(':id/refund')
  @Roles(Role.ADMIN)
  async refundPayment(
    @Param('id') id: string,
    @Body() dto: { amount: number; reason: string },
    @CurrentUser() user: User,
  ) {
    return this.paymentService.refundPayment(
      id,
      dto.amount,
      dto.reason,
      user.id,
    );
  }

  @Post(':id/cancel')
  async cancelPayment(@Param('id') id: string, @CurrentUser() user: User) {
    return this.paymentService.cancelPayment(id, user.id);
  }

  @Post('webhook')
  @Public()
  async handleWebhook(
    @Headers('stripe-signature') signature: string,
    @Req() req: Request,
  ) {
    // Raw body needed for signature verification
    await this.paymentService.handleWebhook(signature, req.body);
    return { received: true };
  }
}
```

**5. DTOs:**

```typescript
// payments/dto/create-payment.dto.ts
export class CreatePaymentDto {
  @IsNumber()
  @Min(0.5)
  amount: number;

  @IsString()
  @IsIn(['USD', 'EUR', 'GBP'])
  currency: string;

  @IsString()
  @IsOptional()
  description?: string;

  @IsObject()
  @IsOptional()
  metadata?: Record<string, any>;

  @IsString()
  @IsOptional()
  idempotencyKey?: string;
}
```

**6. Webhook Configuration:**

```typescript
// main.ts - Configure raw body for webhooks
import { NestFactory } from '@nestjs/core';
import { json } from 'express';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Use raw body for webhook endpoint
  app.use('/payments/webhook', json({ verify: (req, res, buf) => {
    req['rawBody'] = buf;
  }}));

  await app.listen(3000);
}
```

**Key Takeaway:** Payment system integrates Stripe with stripe npm package, creates PaymentIntent for secure card payments, implements idempotency with unique keys preventing duplicate charges, verifies webhook signatures with stripe.webhooks.constructEvent(), handles payment lifecycle events (payment_intent.succeeded, payment_intent.payment_failed, charge.refunded), stores payment metadata in database (status, amount, stripePaymentIntentId, last4, cardBrand), supports refunds with stripe.refunds.create(), uses Stripe Customer for recurring payments, requires raw body for webhook signature verification, implements cancel for pending payments, tracks payment status enum (pending, processing, succeeded, failed, canceled, refunded).

</details>

<details>
<summary><strong>17. How would you build an e-commerce backend?</strong></summary>

**Answer:**

Build a complete e-commerce system with products, cart, orders, and inventory management:

**1. Core Entities:**

```typescript
// products/entities/product.entity.ts
@Entity('products')
export class Product {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  name: string;

  @Column()
  slug: string;

  @Column('text')
  description: string;

  @Column({ type: 'decimal', precision: 10, scale: 2 })
  price: number;

  @Column({ type: 'decimal', precision: 10, scale: 2, nullable: true })
  compareAtPrice: number; // Original price for discounts

  @Column()
  sku: string; // Stock Keeping Unit

  @Column({ default: 0 })
  stock: number;

  @Column({ default: true })
  inStock: boolean;

  @Column({ default: true })
  isActive: boolean;

  @Column('simple-array', { nullable: true })
  images: string[];

  @ManyToOne(() => Category, category => category.products)
  @JoinColumn({ name: 'category_id' })
  category: Category;

  @Column({ name: 'category_id' })
  categoryId: string;

  @Column({ type: 'json', nullable: true })
  attributes: Record<string, any>; // Size, color, weight, etc.

  @Column({ type: 'decimal', precision: 5, scale: 2, nullable: true })
  weight: number; // For shipping

  @Column('simple-array', { nullable: true })
  tags: string[];

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}

// orders/entities/order.entity.ts
@Entity('orders')
export class Order {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  orderNumber: string; // ORD-20231225-001

  @ManyToOne(() => User, user => user.orders)
  @JoinColumn({ name: 'user_id' })
  user: User;

  @Column({ name: 'user_id' })
  userId: string;

  @OneToMany(() => OrderItem, item => item.order, { cascade: true })
  items: OrderItem[];

  @Column({ type: 'decimal', precision: 10, scale: 2 })
  subtotal: number;

  @Column({ type: 'decimal', precision: 10, scale: 2, default: 0 })
  tax: number;

  @Column({ type: 'decimal', precision: 10, scale: 2, default: 0 })
  shipping: number;

  @Column({ type: 'decimal', precision: 10, scale: 2, default: 0 })
  discount: number;

  @Column({ type: 'decimal', precision: 10, scale: 2 })
  total: number;

  @Column({ type: 'enum', enum: ['pending', 'paid', 'processing', 'shipped', 'delivered', 'canceled'], default: 'pending' })
  status: string;

  @Column({ type: 'json' })
  shippingAddress: {
    name: string;
    street: string;
    city: string;
    state: string;
    zipCode: string;
    country: string;
    phone: string;
  };

  @Column({ type: 'json', nullable: true })
  billingAddress: any;

  @Column({ nullable: true })
  paymentId: string;

  @Column({ nullable: true })
  trackingNumber: string;

  @Column({ nullable: true, type: 'timestamp' })
  paidAt: Date;

  @Column({ nullable: true, type: 'timestamp' })
  shippedAt: Date;

  @Column({ nullable: true, type: 'timestamp' })
  deliveredAt: Date;

  @CreateDateColumn()
  createdAt: Date;
}

@Entity('order_items')
export class OrderItem {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @ManyToOne(() => Order, order => order.items)
  @JoinColumn({ name: 'order_id' })
  order: Order;

  @Column({ name: 'order_id' })
  orderId: string;

  @ManyToOne(() => Product)
  @JoinColumn({ name: 'product_id' })
  product: Product;

  @Column({ name: 'product_id' })
  productId: string;

  @Column()
  productName: string; // Snapshot at order time

  @Column({ type: 'decimal', precision: 10, scale: 2 })
  price: number;

  @Column()
  quantity: number;

  @Column({ type: 'decimal', precision: 10, scale: 2 })
  total: number;

  @Column({ type: 'json', nullable: true })
  attributes: any; // Size, color selected
}

// cart/entities/cart.entity.ts
@Entity('carts')
export class Cart {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @OneToOne(() => User)
  @JoinColumn({ name: 'user_id' })
  user: User;

  @Column({ name: 'user_id', unique: true })
  userId: string;

  @OneToMany(() => CartItem, item => item.cart, { cascade: true, eager: true })
  items: CartItem[];

  @UpdateDateColumn()
  updatedAt: Date;
}

@Entity('cart_items')
export class CartItem {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @ManyToOne(() => Cart, cart => cart.items)
  @JoinColumn({ name: 'cart_id' })
  cart: Cart;

  @Column({ name: 'cart_id' })
  cartId: string;

  @ManyToOne(() => Product, { eager: true })
  @JoinColumn({ name: 'product_id' })
  product: Product;

  @Column({ name: 'product_id' })
  productId: string;

  @Column()
  quantity: number;

  @Column({ type: 'json', nullable: true })
  selectedAttributes: any; // Size: 'M', Color: 'Red'

  @CreateDateColumn()
  addedAt: Date;
}
```

**2. Cart Service:**

```typescript
// cart/cart.service.ts
@Injectable()
export class CartService {
  constructor(
    @InjectRepository(Cart)
    private cartRepository: Repository<Cart>,
    @InjectRepository(CartItem)
    private cartItemRepository: Repository<CartItem>,
    @InjectRepository(Product)
    private productRepository: Repository<Product>,
  ) {}

  async getOrCreateCart(userId: string): Promise<Cart> {
    let cart = await this.cartRepository.findOne({
      where: { userId },
      relations: ['items', 'items.product'],
    });

    if (!cart) {
      cart = this.cartRepository.create({ userId, items: [] });
      await this.cartRepository.save(cart);
    }

    return cart;
  }

  async addItem(
    userId: string,
    productId: string,
    quantity: number,
    attributes?: any,
  ): Promise<Cart> {
    const cart = await this.getOrCreateCart(userId);
    const product = await this.productRepository.findOne({ where: { id: productId } });

    if (!product) {
      throw new NotFoundException('Product not found');
    }

    if (!product.inStock || product.stock < quantity) {
      throw new BadRequestException('Product out of stock');
    }

    // Check if item already in cart
    const existingItem = cart.items.find(
      item => item.productId === productId &&
      JSON.stringify(item.selectedAttributes) === JSON.stringify(attributes)
    );

    if (existingItem) {
      existingItem.quantity += quantity;
      await this.cartItemRepository.save(existingItem);
    } else {
      const newItem = this.cartItemRepository.create({
        cartId: cart.id,
        productId,
        quantity,
        selectedAttributes: attributes,
      });
      await this.cartItemRepository.save(newItem);
    }

    return this.getOrCreateCart(userId);
  }

  async updateItemQuantity(
    userId: string,
    itemId: string,
    quantity: number,
  ): Promise<Cart> {
    const cart = await this.getOrCreateCart(userId);
    const item = cart.items.find(i => i.id === itemId);

    if (!item) {
      throw new NotFoundException('Item not found in cart');
    }

    if (quantity === 0) {
      await this.cartItemRepository.delete(itemId);
    } else {
      item.quantity = quantity;
      await this.cartItemRepository.save(item);
    }

    return this.getOrCreateCart(userId);
  }

  async removeItem(userId: string, itemId: string): Promise<Cart> {
    await this.cartItemRepository.delete(itemId);
    return this.getOrCreateCart(userId);
  }

  async clearCart(userId: string): Promise<void> {
    const cart = await this.getOrCreateCart(userId);
    await this.cartItemRepository.delete({ cartId: cart.id });
  }

  async getCartTotal(userId: string): Promise<number> {
    const cart = await this.getOrCreateCart(userId);
    return cart.items.reduce((total, item) => {
      return total + (item.product.price * item.quantity);
    }, 0);
  }
}
```

**3. Order Service:**

```typescript
// orders/order.service.ts
@Injectable()
export class OrderService {
  constructor(
    @InjectRepository(Order)
    private orderRepository: Repository<Order>,
    @InjectRepository(OrderItem)
    private orderItemRepository: Repository<OrderItem>,
    @InjectRepository(Product)
    private productRepository: Repository<Product>,
    private cartService: CartService,
    private paymentService: PaymentService,
    private inventoryService: InventoryService,
  ) {}

  async createOrder(userId: string, dto: CreateOrderDto): Promise<Order> {
    // Get user's cart
    const cart = await this.cartService.getOrCreateCart(userId);

    if (cart.items.length === 0) {
      throw new BadRequestException('Cart is empty');
    }

    // Validate stock availability
    for (const item of cart.items) {
      const product = await this.productRepository.findOne({
        where: { id: item.productId },
      });

      if (!product.inStock || product.stock < item.quantity) {
        throw new BadRequestException(`${product.name} is out of stock`);
      }
    }

    // Calculate totals
    const subtotal = cart.items.reduce(
      (sum, item) => sum + item.product.price * item.quantity,
      0,
    );
    const tax = subtotal * 0.08; // 8% tax
    const shipping = this.calculateShipping(cart.items);
    const discount = await this.calculateDiscount(userId, subtotal);
    const total = subtotal + tax + shipping - discount;

    // Create order
    const order = this.orderRepository.create({
      userId,
      orderNumber: await this.generateOrderNumber(),
      subtotal,
      tax,
      shipping,
      discount,
      total,
      shippingAddress: dto.shippingAddress,
      billingAddress: dto.billingAddress,
      status: 'pending',
    });

    await this.orderRepository.save(order);

    // Create order items
    const orderItems = cart.items.map(item =>
      this.orderItemRepository.create({
        orderId: order.id,
        productId: item.productId,
        productName: item.product.name,
        price: item.product.price,
        quantity: item.quantity,
        total: item.product.price * item.quantity,
        attributes: item.selectedAttributes,
      }),
    );

    await this.orderItemRepository.save(orderItems);

    // Reserve inventory
    for (const item of cart.items) {
      await this.inventoryService.reserveStock(item.productId, item.quantity);
    }

    // Clear cart
    await this.cartService.clearCart(userId);

    // Process payment if payment method provided
    if (dto.paymentMethodId) {
      await this.processPayment(order.id, dto.paymentMethodId);
    }

    return this.getOrderById(order.id, userId);
  }

  async processPayment(orderId: string, paymentMethodId: string): Promise<void> {
    const order = await this.orderRepository.findOne({
      where: { id: orderId },
      relations: ['items'],
    });

    if (!order) {
      throw new NotFoundException('Order not found');
    }

    try {
      const payment = await this.paymentService.createPaymentIntent(
        {
          amount: order.total,
          currency: 'USD',
          description: `Order ${order.orderNumber}`,
          metadata: { orderId: order.id },
        },
        order.userId,
      );

      await this.paymentService.confirmPayment(payment.id, paymentMethodId);

      order.status = 'paid';
      order.paymentId = payment.id;
      order.paidAt = new Date();
      
      await this.orderRepository.save(order);

      // Deduct inventory after successful payment
      for (const item of order.items) {
        await this.inventoryService.deductStock(item.productId, item.quantity);
      }
    } catch (error) {
      // Release reserved inventory on payment failure
      for (const item of order.items) {
        await this.inventoryService.releaseReservedStock(item.productId, item.quantity);
      }
      
      order.status = 'canceled';
      await this.orderRepository.save(order);
      
      throw error;
    }
  }

  async updateOrderStatus(
    orderId: string,
    status: string,
    trackingNumber?: string,
  ): Promise<Order> {
    const order = await this.orderRepository.findOne({ where: { id: orderId } });

    if (!order) {
      throw new NotFoundException('Order not found');
    }

    order.status = status;

    if (status === 'shipped' && trackingNumber) {
      order.trackingNumber = trackingNumber;
      order.shippedAt = new Date();
    }

    if (status === 'delivered') {
      order.deliveredAt = new Date();
    }

    await this.orderRepository.save(order);

    return order;
  }

  async cancelOrder(orderId: string, userId: string): Promise<Order> {
    const order = await this.orderRepository.findOne({
      where: { id: orderId, userId },
      relations: ['items'],
    });

    if (!order) {
      throw new NotFoundException('Order not found');
    }

    if (order.status === 'shipped' || order.status === 'delivered') {
      throw new BadRequestException('Cannot cancel shipped/delivered orders');
    }

    // Refund if paid
    if (order.status === 'paid' && order.paymentId) {
      await this.paymentService.refundPayment(
        order.paymentId,
        order.total,
        'Order canceled',
        userId,
      );
    }

    // Restore inventory
    for (const item of order.items) {
      await this.inventoryService.restoreStock(item.productId, item.quantity);
    }

    order.status = 'canceled';
    await this.orderRepository.save(order);

    return order;
  }

  async getUserOrders(
    userId: string,
    options: { page: number; limit: number },
  ): Promise<PaginatedResponse<Order>> {
    const [orders, total] = await this.orderRepository.findAndCount({
      where: { userId },
      relations: ['items', 'items.product'],
      order: { createdAt: 'DESC' },
      skip: (options.page - 1) * options.limit,
      take: options.limit,
    });

    return {
      data: orders,
      meta: {
        page: options.page,
        limit: options.limit,
        total,
        totalPages: Math.ceil(total / options.limit),
      },
    };
  }

  async getOrderById(id: string, userId: string): Promise<Order> {
    const order = await this.orderRepository.findOne({
      where: { id, userId },
      relations: ['items', 'items.product'],
    });

    if (!order) {
      throw new NotFoundException('Order not found');
    }

    return order;
  }

  private async generateOrderNumber(): Promise<string> {
    const date = new Date().toISOString().slice(0, 10).replace(/-/g, '');
    const count = await this.orderRepository.count();
    return `ORD-${date}-${String(count + 1).padStart(4, '0')}`;
  }

  private calculateShipping(items: CartItem[]): number {
    const totalWeight = items.reduce(
      (sum, item) => sum + (item.product.weight || 0) * item.quantity,
      0,
    );

    if (totalWeight === 0) return 0;
    if (totalWeight < 5) return 5;
    if (totalWeight < 10) return 10;
    return 15;
  }

  private async calculateDiscount(userId: string, subtotal: number): Promise<number> {
    // Apply coupon codes, loyalty points, etc.
    return 0;
  }
}
```

**4. Inventory Service:**

```typescript
// inventory/inventory.service.ts
@Injectable()
export class InventoryService {
  constructor(
    @InjectRepository(Product)
    private productRepository: Repository<Product>,
  ) {}

  async reserveStock(productId: string, quantity: number): Promise<void> {
    const product = await this.productRepository.findOne({
      where: { id: productId },
    });

    if (product.stock < quantity) {
      throw new BadRequestException('Insufficient stock');
    }

    // Use optimistic locking with version column or database transactions
    await this.productRepository.decrement({ id: productId }, 'stock', quantity);

    // Check if out of stock
    const updated = await this.productRepository.findOne({ where: { id: productId } });
    if (updated.stock === 0) {
      updated.inStock = false;
      await this.productRepository.save(updated);
    }
  }

  async deductStock(productId: string, quantity: number): Promise<void> {
    // Already deducted in reserveStock, this is for confirmed orders
    // Can implement separate reserved_stock column
  }

  async releaseReservedStock(productId: string, quantity: number): Promise<void> {
    await this.productRepository.increment({ id: productId }, 'stock', quantity);

    const product = await this.productRepository.findOne({ where: { id: productId } });
    if (product.stock > 0) {
      product.inStock = true;
      await this.productRepository.save(product);
    }
  }

  async restoreStock(productId: string, quantity: number): Promise<void> {
    await this.releaseReservedStock(productId, quantity);
  }

  async getLowStockProducts(threshold = 10): Promise<Product[]> {
    return this.productRepository.find({
      where: {
        stock: LessThan(threshold),
        isActive: true,
      },
    });
  }
}
```

**5. Product Search Service:**

```typescript
// products/product-search.service.ts
@Injectable()
export class ProductSearchService {
  constructor(
    @InjectRepository(Product)
    private productRepository: Repository<Product>,
  ) {}

  async search(query: SearchProductsDto): Promise<PaginatedResponse<Product>> {
    const qb = this.productRepository
      .createQueryBuilder('product')
      .leftJoinAndSelect('product.category', 'category')
      .where('product.isActive = :isActive', { isActive: true });

    // Text search
    if (query.q) {
      qb.andWhere(
        '(product.name ILIKE :search OR product.description ILIKE :search OR product.tags @> ARRAY[:tag])',
        { search: `%${query.q}%`, tag: query.q },
      );
    }

    // Category filter
    if (query.categoryId) {
      qb.andWhere('product.categoryId = :categoryId', {
        categoryId: query.categoryId,
      });
    }

    // Price range
    if (query.minPrice) {
      qb.andWhere('product.price >= :minPrice', { minPrice: query.minPrice });
    }
    if (query.maxPrice) {
      qb.andWhere('product.price <= :maxPrice', { maxPrice: query.maxPrice });
    }

    // In stock filter
    if (query.inStockOnly) {
      qb.andWhere('product.inStock = true');
    }

    // Sorting
    const sortField = query.sortBy || 'createdAt';
    const sortOrder = query.order || 'DESC';
    qb.orderBy(`product.${sortField}`, sortOrder as 'ASC' | 'DESC');

    // Pagination
    const [products, total] = await qb
      .skip((query.page - 1) * query.limit)
      .take(query.limit)
      .getManyAndCount();

    return {
      data: products,
      meta: {
        page: query.page,
        limit: query.limit,
        total,
        totalPages: Math.ceil(total / query.limit),
      },
    };
  }
}
```

**Key Takeaway:** E-commerce backend requires Product entity with stock/pricing/images, Cart with CartItems for shopping session, Order with OrderItems for purchase records, inventory management with reserveStock()/deductStock()/restoreStock(), order lifecycle (pending → paid → processing → shipped → delivered), payment integration with createPaymentIntent() and refunds, shipping calculation based on weight, order number generation (ORD-20231225-0001), stock validation before order creation, optimistic locking for concurrent stock updates, search with filters (category, price range, text search), cancel orders with inventory restoration and payment refunds.

</details>

<details>
<summary><strong>18. How would you build a social media feed?</strong></summary>

**Answer:**

Build a scalable social media feed with follow/unfollow, posts, likes, comments, and timeline generation:

**1. Core Entities:**

```typescript
// posts/entities/post.entity.ts
@Entity('posts')
export class Post {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column('text')
  content: string;

  @ManyToOne(() => User, user => user.posts)
  @JoinColumn({ name: 'author_id' })
  author: User;

  @Column({ name: 'author_id' })
  authorId: string;

  @Column('simple-array', { nullable: true })
  images: string[];

  @Column({ nullable: true })
  videoUrl: string;

  @Column({ default: 0 })
  likesCount: number;

  @Column({ default: 0 })
  commentsCount: number;

  @Column({ default: 0 })
  sharesCount: number;

  @Column({ default: true })
  isPublic: boolean;

  @Column('simple-array', { nullable: true })
  hashtags: string[];

  @Column({ nullable: true })
  location: string;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  @Index()
  @Column({ type: 'timestamp', default: () => 'CURRENT_TIMESTAMP' })
  publishedAt: Date;
}

// users/entities/follow.entity.ts
@Entity('follows')
@Unique(['followerId', 'followingId'])
export class Follow {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @ManyToOne(() => User, user => user.following)
  @JoinColumn({ name: 'follower_id' })
  follower: User;

  @Column({ name: 'follower_id' })
  @Index()
  followerId: string;

  @ManyToOne(() => User, user => user.followers)
  @JoinColumn({ name: 'following_id' })
  following: User;

  @Column({ name: 'following_id' })
  @Index()
  followingId: string;

  @CreateDateColumn()
  createdAt: Date;
}

// posts/entities/like.entity.ts
@Entity('likes')
@Unique(['userId', 'postId'])
export class Like {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @ManyToOne(() => User)
  @JoinColumn({ name: 'user_id' })
  user: User;

  @Column({ name: 'user_id' })
  @Index()
  userId: string;

  @ManyToOne(() => Post)
  @JoinColumn({ name: 'post_id' })
  post: Post;

  @Column({ name: 'post_id' })
  @Index()
  postId: string;

  @CreateDateColumn()
  createdAt: Date;
}

// posts/entities/comment.entity.ts
@Entity('comments')
export class Comment {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column('text')
  content: string;

  @ManyToOne(() => User)
  @JoinColumn({ name: 'author_id' })
  author: User;

  @Column({ name: 'author_id' })
  authorId: string;

  @ManyToOne(() => Post)
  @JoinColumn({ name: 'post_id' })
  post: Post;

  @Column({ name: 'post_id' })
  @Index()
  postId: string;

  @ManyToOne(() => Comment, { nullable: true })
  @JoinColumn({ name: 'parent_id' })
  parent: Comment;

  @Column({ name: 'parent_id', nullable: true })
  parentId: string;

  @Column({ default: 0 })
  likesCount: number;

  @CreateDateColumn()
  createdAt: Date;
}
```

**2. Feed Service (Timeline Generation):**

```typescript
// feed/feed.service.ts
import { CACHE_MANAGER, Inject } from '@nestjs/common';
import { Cache } from 'cache-manager';

@Injectable()
export class FeedService {
  constructor(
    @InjectRepository(Post)
    private postRepository: Repository<Post>,
    @InjectRepository(Follow)
    private followRepository: Repository<Follow>,
    @InjectRepository(Like)
    private likeRepository: Repository<Like>,
    @Inject(CACHE_MANAGER)
    private cacheManager: Cache,
  ) {}

  // Pull-based feed (query on demand)
  async getHomeFeed(
    userId: string,
    options: { page: number; limit: number },
  ): Promise<PaginatedResponse<Post>> {
    // Get users that current user follows
    const following = await this.followRepository.find({
      where: { followerId: userId },
      select: ['followingId'],
    });

    const followingIds = following.map(f => f.followingId);
    followingIds.push(userId); // Include own posts

    // Get posts from followed users
    const [posts, total] = await this.postRepository.findAndCount({
      where: {
        authorId: In(followingIds),
        isPublic: true,
      },
      relations: ['author'],
      order: { publishedAt: 'DESC' },
      skip: (options.page - 1) * options.limit,
      take: options.limit,
    });

    // Enrich posts with like status for current user
    const enrichedPosts = await Promise.all(
      posts.map(post => this.enrichPost(post, userId)),
    );

    return {
      data: enrichedPosts,
      meta: {
        page: options.page,
        limit: options.limit,
        total,
        totalPages: Math.ceil(total / options.limit),
      },
    };
  }

  // Push-based feed (pre-computed, fanout on write)
  async getFanoutFeed(
    userId: string,
    options: { page: number; limit: number },
  ): Promise<any[]> {
    const cacheKey = `feed:${userId}`;
    
    // Try to get from cache (Redis)
    const cachedFeed = await this.cacheManager.get<string[]>(cacheKey);

    if (cachedFeed) {
      // Get posts from cached IDs
      const start = (options.page - 1) * options.limit;
      const end = start + options.limit;
      const postIds = cachedFeed.slice(start, end);

      const posts = await this.postRepository.findByIds(postIds, {
        relations: ['author'],
      });

      // Sort by cached order
      const postsMap = new Map(posts.map(p => [p.id, p]));
      return postIds.map(id => postsMap.get(id)).filter(Boolean);
    }

    // Fallback to pull-based if cache miss
    const result = await this.getHomeFeed(userId, options);
    return result.data;
  }

  // Fanout post to followers' feeds (on post creation)
  async fanoutPost(postId: string, authorId: string): Promise<void> {
    // Get author's followers
    const followers = await this.followRepository.find({
      where: { followingId: authorId },
      select: ['followerId'],
    });

    // Add post to each follower's feed cache
    for (const follower of followers) {
      const cacheKey = `feed:${follower.followerId}`;
      const feed = await this.cacheManager.get<string[]>(cacheKey) || [];
      
      // Add to beginning of feed
      feed.unshift(postId);
      
      // Keep only recent 1000 posts
      if (feed.length > 1000) {
        feed.length = 1000;
      }

      await this.cacheManager.set(cacheKey, feed, { ttl: 86400 }); // 24 hours
    }
  }

  // Discover feed (trending, recommended)
  async getDiscoverFeed(
    userId: string,
    options: { page: number; limit: number },
  ): Promise<PaginatedResponse<Post>> {
    // Get trending posts (high engagement in last 24 hours)
    const oneDayAgo = new Date(Date.now() - 24 * 60 * 60 * 1000);

    const [posts, total] = await this.postRepository.findAndCount({
      where: {
        isPublic: true,
        createdAt: MoreThan(oneDayAgo),
      },
      order: {
        likesCount: 'DESC',
        commentsCount: 'DESC',
      },
      skip: (options.page - 1) * options.limit,
      take: options.limit,
      relations: ['author'],
    });

    const enrichedPosts = await Promise.all(
      posts.map(post => this.enrichPost(post, userId)),
    );

    return {
      data: enrichedPosts,
      meta: {
        page: options.page,
        limit: options.limit,
        total,
        totalPages: Math.ceil(total / options.limit),
      },
    };
  }

  // Enrich post with user-specific data
  private async enrichPost(post: any, userId: string): Promise<any> {
    // Check if user liked the post
    const liked = await this.likeRepository.findOne({
      where: { postId: post.id, userId },
    });

    return {
      ...post,
      isLiked: !!liked,
    };
  }

  // Get user profile posts
  async getUserPosts(
    targetUserId: string,
    currentUserId: string,
    options: { page: number; limit: number },
  ): Promise<PaginatedResponse<Post>> {
    const [posts, total] = await this.postRepository.findAndCount({
      where: {
        authorId: targetUserId,
        isPublic: true, // Add privacy check based on relationship
      },
      order: { publishedAt: 'DESC' },
      skip: (options.page - 1) * options.limit,
      take: options.limit,
      relations: ['author'],
    });

    const enrichedPosts = await Promise.all(
      posts.map(post => this.enrichPost(post, currentUserId)),
    );

    return {
      data: enrichedPosts,
      meta: {
        page: options.page,
        limit: options.limit,
        total,
        totalPages: Math.ceil(total / options.limit),
      },
    };
  }
}
```

**3. Post Service:**

```typescript
// posts/post.service.ts
@Injectable()
export class PostService {
  constructor(
    @InjectRepository(Post)
    private postRepository: Repository<Post>,
    @InjectRepository(Like)
    private likeRepository: Repository<Like>,
    @InjectRepository(Comment)
    private commentRepository: Repository<Comment>,
    private feedService: FeedService,
  ) {}

  async createPost(userId: string, dto: CreatePostDto): Promise<Post> {
    // Extract hashtags
    const hashtags = this.extractHashtags(dto.content);

    const post = this.postRepository.create({
      ...dto,
      authorId: userId,
      hashtags,
    });

    await this.postRepository.save(post);

    // Fanout to followers' feeds
    await this.feedService.fanoutPost(post.id, userId);

    return post;
  }

  async likePost(postId: string, userId: string): Promise<void> {
    // Check if already liked
    const existing = await this.likeRepository.findOne({
      where: { postId, userId },
    });

    if (existing) {
      throw new BadRequestException('Already liked');
    }

    // Create like
    const like = this.likeRepository.create({ postId, userId });
    await this.likeRepository.save(like);

    // Increment count
    await this.postRepository.increment({ id: postId }, 'likesCount', 1);
  }

  async unlikePost(postId: string, userId: string): Promise<void> {
    const result = await this.likeRepository.delete({ postId, userId });

    if (result.affected === 0) {
      throw new BadRequestException('Not liked');
    }

    // Decrement count
    await this.postRepository.decrement({ id: postId }, 'likesCount', 1);
  }

  async addComment(postId: string, userId: string, content: string, parentId?: string): Promise<Comment> {
    const comment = this.commentRepository.create({
      postId,
      authorId: userId,
      content,
      parentId,
    });

    await this.commentRepository.save(comment);

    // Increment count
    await this.postRepository.increment({ id: postId }, 'commentsCount', 1);

    return comment;
  }

  async getPostComments(
    postId: string,
    options: { page: number; limit: number },
  ): Promise<PaginatedResponse<Comment>> {
    const [comments, total] = await this.commentRepository.findAndCount({
      where: { postId, parentId: IsNull() }, // Top-level comments only
      relations: ['author'],
      order: { createdAt: 'DESC' },
      skip: (options.page - 1) * options.limit,
      take: options.limit,
    });

    return {
      data: comments,
      meta: {
        page: options.page,
        limit: options.limit,
        total,
        totalPages: Math.ceil(total / options.limit),
      },
    };
  }

  async deletePost(postId: string, userId: string): Promise<void> {
    const post = await this.postRepository.findOne({
      where: { id: postId, authorId: userId },
    });

    if (!post) {
      throw new NotFoundException('Post not found or access denied');
    }

    await this.postRepository.delete(postId);
  }

  private extractHashtags(content: string): string[] {
    const regex = /#(\w+)/g;
    const matches = content.match(regex);
    return matches ? matches.map(tag => tag.slice(1).toLowerCase()) : [];
  }
}
```

**4. Follow Service:**

```typescript
// users/follow.service.ts
@Injectable()
export class FollowService {
  constructor(
    @InjectRepository(Follow)
    private followRepository: Repository<Follow>,
    @InjectRepository(User)
    private userRepository: Repository<User>,
  ) {}

  async followUser(followerId: string, followingId: string): Promise<void> {
    if (followerId === followingId) {
      throw new BadRequestException('Cannot follow yourself');
    }

    // Check if already following
    const existing = await this.followRepository.findOne({
      where: { followerId, followingId },
    });

    if (existing) {
      throw new BadRequestException('Already following');
    }

    const follow = this.followRepository.create({ followerId, followingId });
    await this.followRepository.save(follow);

    // Update counts
    await this.userRepository.increment({ id: followerId }, 'followingCount', 1);
    await this.userRepository.increment({ id: followingId }, 'followersCount', 1);
  }

  async unfollowUser(followerId: string, followingId: string): Promise<void> {
    const result = await this.followRepository.delete({ followerId, followingId });

    if (result.affected === 0) {
      throw new BadRequestException('Not following');
    }

    // Update counts
    await this.userRepository.decrement({ id: followerId }, 'followingCount', 1);
    await this.userRepository.decrement({ id: followingId }, 'followersCount', 1);
  }

  async getFollowers(
    userId: string,
    options: { page: number; limit: number },
  ): Promise<PaginatedResponse<User>> {
    const [follows, total] = await this.followRepository.findAndCount({
      where: { followingId: userId },
      relations: ['follower'],
      skip: (options.page - 1) * options.limit,
      take: options.limit,
    });

    return {
      data: follows.map(f => f.follower),
      meta: {
        page: options.page,
        limit: options.limit,
        total,
        totalPages: Math.ceil(total / options.limit),
      },
    };
  }

  async getFollowing(
    userId: string,
    options: { page: number; limit: number },
  ): Promise<PaginatedResponse<User>> {
    const [follows, total] = await this.followRepository.findAndCount({
      where: { followerId: userId },
      relations: ['following'],
      skip: (options.page - 1) * options.limit,
      take: options.limit,
    });

    return {
      data: follows.map(f => f.following),
      meta: {
        page: options.page,
        limit: options.limit,
        total,
        totalPages: Math.ceil(total / options.limit),
      },
    };
  }

  async isFollowing(followerId: string, followingId: string): Promise<boolean> {
    const follow = await this.followRepository.findOne({
      where: { followerId, followingId },
    });
    return !!follow;
  }
}
```

**Key Takeaway:** Social feed requires Post entity with engagement counts (likesCount, commentsCount), Follow relationship with unique constraint on (followerId, followingId), Like with unique (userId, postId), Comment with optional parentId for threading. Feed generation uses pull-based (query on demand with IN clause) or push-based (fanout on write to Redis cache). Timeline queries follow users with `followingIds = following.map(f => f.followingId)` then `WHERE authorId IN (followingIds)`, sorted by publishedAt DESC. Like/unlike increments/decrements counts, hashtag extraction with regex `/#(\w+)/g`, discover feed uses trending algorithm (high engagement in 24h), enrich posts with isLiked status, cache feeds in Redis with TTL.

</details>

## Scalability & Performance

<details>
<summary><strong>19. How do you design for high availability?</strong></summary>

**Answer:**

Design for high availability with redundancy, health checks, and failover mechanisms:

**1. Application Redundancy (Multiple Instances):**

```typescript
// health/health.controller.ts
import { Controller, Get } from '@nestjs/common';
import { HealthCheck, HealthCheckService, TypeOrmHealthIndicator, DiskHealthIndicator, MemoryHealthIndicator } from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private db: TypeOrmHealthIndicator,
    private disk: DiskHealthIndicator,
    private memory: MemoryHealthIndicator,
  ) {}

  @Get()
  @HealthCheck()
  check() {
    return this.health.check([
      () => this.db.pingCheck('database'),
      () => this.disk.checkStorage('storage', { path: '/', thresholdPercent: 0.9 }),
      () => this.memory.checkHeap('memory_heap', 150 * 1024 * 1024),
    ]);
  }

  @Get('readiness')
  @HealthCheck()
  readiness() {
    // Check if app is ready to serve traffic
    return this.health.check([
      () => this.db.pingCheck('database'),
      // Check Redis, external APIs, etc.
    ]);
  }

  @Get('liveness')
  @HealthCheck()
  liveness() {
    // Check if app is alive (basic)
    return { status: 'ok' };
  }
}
```

**2. Load Balancer Configuration (nginx):**

```nginx
# nginx.conf
upstream nestjs_backend {
    least_conn; # or ip_hash for sticky sessions
    server app1:3000 max_fails=3 fail_timeout=30s;
    server app2:3000 max_fails=3 fail_timeout=30s;
    server app3:3000 max_fails=3 fail_timeout=30s;
}

server {
    listen 80;
    
    location / {
        proxy_pass http://nestjs_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        
        # Health check
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503;
        proxy_connect_timeout 5s;
        proxy_read_timeout 60s;
    }
    
    location /health {
        proxy_pass http://nestjs_backend/health;
        access_log off;
    }
}
```

**3. Database High Availability:**

```typescript
// database/database.module.ts (Master-Replica setup)
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      replication: {
        master: {
          host: process.env.DB_MASTER_HOST,
          port: 5432,
          username: process.env.DB_USER,
          password: process.env.DB_PASSWORD,
          database: process.env.DB_NAME,
        },
        slaves: [
          {
            host: process.env.DB_REPLICA1_HOST,
            port: 5432,
            username: process.env.DB_USER,
            password: process.env.DB_PASSWORD,
            database: process.env.DB_NAME,
          },
          {
            host: process.env.DB_REPLICA2_HOST,
            port: 5432,
            username: process.env.DB_USER,
            password: process.env.DB_PASSWORD,
            database: process.env.DB_NAME,
          },
        ],
      },
      // Writes go to master, reads to replicas
    }),
  ],
})
export class DatabaseModule {}
```

**4. Circuit Breaker Pattern:**

```typescript
// common/decorators/circuit-breaker.decorator.ts
import { CircuitBreaker } from 'opossum';

export function UseCircuitBreaker(options: any = {}) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    
    const breaker = new CircuitBreaker(originalMethod, {
      timeout: options.timeout || 3000,
      errorThresholdPercentage: options.errorThresholdPercentage || 50,
      resetTimeout: options.resetTimeout || 30000,
    });

    breaker.on('open', () => console.log(`Circuit breaker opened for ${propertyKey}`));
    breaker.on('halfOpen', () => console.log(`Circuit breaker half-open for ${propertyKey}`));
    breaker.on('close', () => console.log(`Circuit breaker closed for ${propertyKey}`));

    descriptor.value = async function (...args: any[]) {
      return breaker.fire(...args);
    };

    return descriptor;
  };
}

// Usage:
@Injectable()
export class ExternalApiService {
  @UseCircuitBreaker({ timeout: 5000 })
  async callExternalAPI() {
    // API call that might fail
  }
}
```

**5. Graceful Shutdown:**

```typescript
// main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Enable shutdown hooks
  app.enableShutdownHooks();

  await app.listen(3000);

  // Handle graceful shutdown
  process.on('SIGTERM', async () => {
    console.log('SIGTERM received, closing server gracefully');
    await app.close();
    process.exit(0);
  });
}
```

**6. Docker Compose HA Setup:**

```yaml
# docker-compose.yml
version: '3.8'

services:
  app1:
    image: nestjs-app:latest
    environment:
      - NODE_ENV=production
      - DB_HOST=postgres-master
    depends_on:
      - postgres-master
      - redis
    restart: always
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  app2:
    image: nestjs-app:latest
    environment:
      - NODE_ENV=production
      - DB_HOST=postgres-master
    depends_on:
      - postgres-master
      - redis
    restart: always

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - app1
      - app2

  postgres-master:
    image: postgres:15
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - pg-master-data:/var/lib/postgresql/data
    command: postgres -c wal_level=replica -c max_wal_senders=3

  redis:
    image: redis:7-alpine
    restart: always
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data

volumes:
  pg-master-data:
  redis-data:
```

**Key Takeaway:** High availability requires multiple app instances behind load balancer (nginx with least_conn), health checks (@nestjs/terminus with readiness/liveness endpoints), database replication (master-replicas), circuit breaker for external services, graceful shutdown with enableShutdownHooks(), auto-restart with Docker restart: always, load balancer failover with max_fails=3, monitoring with healthchecks, redundancy eliminates single points of failure.

</details>

<details>
<summary><strong>20. How do you handle millions of concurrent users?</strong></summary>

**Answer:**

Handle high concurrency with horizontal scaling, caching, queue processing, and connection pooling:

**1. Connection Pooling:**

```typescript
// database.config.ts
export default {
  type: 'postgres',
  host: process.env.DB_HOST,
  port: 5432,
  username: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  // Connection pool configuration
  extra: {
    max: 100, // Maximum pool size
    min: 10,  // Minimum pool size
    idleTimeoutMillis: 30000,
    connectionTimeoutMillis: 2000,
  },
};
```

**2. Redis Caching Layer:**

```typescript
// cache/cache.module.ts
import { CacheModule } from '@nestjs/cache-manager';
import * as redisStore from 'cache-manager-redis-store';

@Module({
  imports: [
    CacheModule.register({
      store: redisStore,
      host: process.env.REDIS_HOST,
      port: 6379,
      ttl: 600, // 10 minutes default
      max: 1000, // Maximum number of items
    }),
  ],
})
export class CacheConfigModule {}

// Service with caching
@Injectable()
export class ProductService {
  constructor(
    @InjectRepository(Product)
    private productRepo: Repository<Product>,
    @Inject(CACHE_MANAGER)
    private cacheManager: Cache,
  ) {}

  async getProduct(id: string): Promise<Product> {
    const cacheKey = `product:${id}`;
    
    // Try cache first
    const cached = await this.cacheManager.get<Product>(cacheKey);
    if (cached) return cached;

    // Fetch from DB
    const product = await this.productRepo.findOne({ where: { id } });
    
    // Cache for 1 hour
    await this.cacheManager.set(cacheKey, product, { ttl: 3600 });
    
    return product;
  }
}
```

**3. Rate Limiting:**

```typescript
// rate-limit.config.ts
import { ThrottlerModule } from '@nestjs/throttler';

@Module({
  imports: [
    ThrottlerModule.forRoot({
      ttl: 60, // 60 seconds
      limit: 100, // 100 requests per TTL per user
      storage: new ThrottlerStorageRedisService(redis),
    }),
  ],
})
export class AppModule {}

// Per-route rate limits
@Controller('api')
export class ApiController {
  @Throttle(10, 60) // 10 requests per minute
  @Get('expensive-operation')
  async expensiveOperation() {
    // ...
  }
}
```

**4. Background Job Processing:**

```typescript
// queues/queue.module.ts (Bull for async processing)
import { BullModule } from '@nestjs/bull';

@Module({
  imports: [
    BullModule.forRoot({
      redis: {
        host: process.env.REDIS_HOST,
        port: 6379,
      },
    }),
    BullModule.registerQueue(
      { name: 'email' },
      { name: 'image-processing' },
      { name: 'analytics' },
    ),
  ],
})
export class QueueModule {}

// Offload heavy tasks to queue
@Injectable()
export class UserService {
  constructor(
    @InjectQueue('email') private emailQueue: Queue,
  ) {}

  async createUser(dto: CreateUserDto) {
    const user = await this.userRepo.save(dto);
    
    // Async email (don't block response)
    await this.emailQueue.add('welcome', { userId: user.id });
    
    return user;
  }
}
```

**5. Read/Write Separation:**

```typescript
// Use replicas for read queries
@Injectable()
export class ProductService {
  async getProducts(query: any) {
    // Force read from replica
    return this.productRepo
      .createQueryBuilder('product', 'slave') // Use replica
      .where('product.isActive = true')
      .getMany();
  }

  async updateProduct(id: string, dto: UpdateProductDto) {
    // Writes go to master automatically
    return this.productRepo.update(id, dto);
  }
}
```

**6. API Response Compression:**

```typescript
// main.ts
import compression from 'compression';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Enable gzip compression
  app.use(compression());
  
  await app.listen(3000);
}
```

**7. Kubernetes HPA (Horizontal Pod Autoscaler):**

```yaml
# k8s/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nestjs-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nestjs-app
  minReplicas: 3
  maxReplicas: 50
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300 # Wait 5 min before scaling down
    scaleUp:
      stabilizationWindowSeconds: 0 # Scale up immediately
```

**8. Database Query Optimization:**

```typescript
// Optimize queries with indexes and selective fields
@Injectable()
export class PostService {
  async getFeed(userId: string, page: number) {
    return this.postRepo
      .createQueryBuilder('post')
      .select(['post.id', 'post.title', 'post.createdAt']) // Only needed fields
      .leftJoin('post.author', 'author')
      .addSelect(['author.id', 'author.username']) // Selective join
      .where('post.authorId IN (:...ids)', { ids: followingIds })
      .orderBy('post.createdAt', 'DESC')
      .take(20)
      .skip((page - 1) * 20)
      .cache(60000) // Query result cache (1 min)
      .getMany();
  }
}

// Add database indexes
@Entity('posts')
@Index(['authorId', 'createdAt']) // Composite index for common query
export class Post {
  @Column()
  @Index() // Single column index
  authorId: string;
}
```

**9. WebSocket Connection Management:**

```typescript
// Limit WebSocket connections per server
@WebSocketGateway({ maxHttpBufferSize: 1e6 })
export class ChatGateway {
  private readonly maxConnectionsPerServer = 10000;
  private connectionCount = 0;

  handleConnection(client: Socket) {
    if (this.connectionCount >= this.maxConnectionsPerServer) {
      client.disconnect();
      return;
    }
    
    this.connectionCount++;
  }

  handleDisconnect() {
    this.connectionCount--;
  }
}
```

**Key Takeaway:** Handle millions of users with horizontal scaling (K8s HPA scaling 3-50 pods based on CPU 70%), connection pooling (max: 100), Redis caching (TTL 600s), rate limiting (100 req/60s per user), background jobs with Bull queues, read replicas for SELECT queries, gzip compression, database indexes on frequently queried columns, query result caching, selective field loading, offload WebSocket connections across multiple servers, use CDN for static assets.

</details>

<details>
<summary><strong>21. How do you implement horizontal scaling?</strong></summary>

**Answer:**

Implement horizontal scaling with stateless architecture, session storage, and load balancing:

**1. Stateless Application Design:**

```typescript
// DON'T: Store state in memory
class AuthService {
  private sessions = new Map(); // ❌ Won't work across instances
}

// DO: Store state in external storage
@Injectable()
export class AuthService {
  constructor(
    @Inject(CACHE_MANAGER) private cache: Cache,
  ) {}

  async createSession(userId: string) {
    const sessionId = uuid();
    await this.cache.set(`session:${sessionId}`, userId, { ttl: 3600 });
    return sessionId;
  }
}
```

**2. Session Storage in Redis:**

```typescript
// session.module.ts
import session from 'express-session';
import RedisStore from 'connect-redis';
import { createClient } from 'redis';

@Module({})
export class SessionModule {
  static forRoot() {
    return {
      module: SessionModule,
      imports: [],
      providers: [
        {
          provide: 'SESSION_MIDDLEWARE',
          useFactory: async () => {
            const redisClient = createClient({
              url: process.env.REDIS_URL,
            });
            await redisClient.connect();

            return session({
              store: new RedisStore({ client: redisClient }),
              secret: process.env.SESSION_SECRET,
              resave: false,
              saveUninitialized: false,
              cookie: { maxAge: 3600000 }, // 1 hour
            });
          },
        },
      ],
      exports: ['SESSION_MIDDLEWARE'],
    };
  }
}
```

**3. Docker Swarm Deploy:**

```yaml
# docker-stack.yml
version: '3.8'

services:
  app:
    image: nestjs-app:latest
    deploy:
      replicas: 5 # Scale to 5 instances
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
    environment:
      - REDIS_HOST=redis
      - DB_HOST=postgres
    networks:
      - app-network

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    deploy:
      replicas: 2
    configs:
      - source: nginx_config
        target: /etc/nginx/nginx.conf
    networks:
      - app-network

configs:
  nginx_config:
    file: ./nginx.conf

networks:
  app-network:
    driver: overlay
```

**4. Kubernetes Deployment:**

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nestjs-app
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nestjs
  template:
    metadata:
      labels:
        app: nestjs
    spec:
      containers:
      - name: app
        image: nestjs-app:latest
        ports:
        - containerPort: 3000
        env:
        - name: REDIS_HOST
          value: redis-service
        - name: DB_HOST
          value: postgres-service
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health/liveness
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health/readiness
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: nestjs-service
spec:
  selector:
    app: nestjs
  ports:
  - port: 80
    targetPort: 3000
  type: LoadBalancer
```

**5. File Upload Considerations:**

```typescript
// Use external storage for uploads (S3, not local disk)
@Injectable()
export class FileService {
  async uploadFile(file: Express.Multer.File) {
    // ❌ DON'T: Save to local disk (not shared across instances)
    // fs.writeFileSync('/uploads/file.jpg', file.buffer);

    // ✅ DO: Use S3 or shared storage
    await this.s3.upload({
      Bucket: process.env.S3_BUCKET,
      Key: file.originalname,
      Body: file.buffer,
    });
  }
}
```

**6. Distributed Locking:**

```typescript
// Use Redis for distributed locks
import Redlock from 'redlock';

@Injectable()
export class InventoryService {
  private redlock: Redlock;

  constructor(@Inject('REDIS_CLIENT') redis: Redis) {
    this.redlock = new Redlock([redis]);
  }

  async updateStock(productId: string, quantity: number) {
    const lock = await this.redlock.acquire([`lock:product:${productId}`], 5000);

    try {
      // Critical section - only one instance can execute
      const product = await this.productRepo.findOne({ where: { id: productId } });
      product.stock -= quantity;
      await this.productRepo.save(product);
    } finally {
      await lock.release();
    }
  }
}
```

**7. PM2 Cluster Mode:**

```javascript
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'nestjs-app',
    script: './dist/main.js',
    instances: 'max', // Use all CPU cores
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
    },
  }],
};

// Start with: pm2 start ecosystem.config.js
```

**Key Takeaway:** Horizontal scaling requires stateless app design (no in-memory state), external session storage in Redis with connect-redis, shared file storage (S3 not local disk), distributed locking with Redlock for critical sections, load balancer distributing traffic, container orchestration (Docker Swarm replicas: 5 or K8s Deployment replicas: 5), health checks for pod readiness, resource limits (CPU/memory), PM2 cluster mode for CPU utilization, sticky sessions if needed with ip_hash.

</details>

<details>
<summary><strong>22. What is database sharding and when to use it?</strong></summary>

**Answer:**

Database sharding partitions data horizontally across multiple database instances:

**1. Sharding Strategies:**

```typescript
// Range-based sharding (by user ID ranges)
export class UserShardingService {
  getShardForUser(userId: number): string {
    if (userId < 1000000) return 'shard1';
    if (userId < 2000000) return 'shard2';
    return 'shard3';
  }
}

// Hash-based sharding (consistent hashing)
export class OrderShardingService {
  private readonly SHARD_COUNT = 4;

  getShardForOrder(orderId: string): string {
    const hash = this.hashString(orderId);
    const shardIndex = hash % this.SHARD_COUNT;
    return `shard${shardIndex + 1}`;
  }

  private hashString(str: string): number {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      hash = (hash << 5) - hash + str.charCodeAt(i);
      hash |= 0; // Convert to 32-bit integer
    }
    return Math.abs(hash);
  }
}

// Geographic sharding
export class GeoShardingService {
  getShardForRegion(country: string): string {
    if (['US', 'CA', 'MX'].includes(country)) return 'shard-americas';
    if (['GB', 'FR', 'DE'].includes(country)) return 'shard-europe';
    if (['JP', 'CN', 'IN'].includes(country)) return 'shard-asia';
    return 'shard-default';
  }
}
```

**2. Shard Configuration:**

```typescript
// database/shard.config.ts
export const SHARD_CONNECTIONS = {
  shard1: {
    type: 'postgres',
    host: 'db-shard1.example.com',
    port: 5432,
    username: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    database: 'app_shard1',
    entities: [User, Order],
  },
  shard2: {
    type: 'postgres',
    host: 'db-shard2.example.com',
    port: 5432,
    username: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    database: 'app_shard2',
    entities: [User, Order],
  },
  shard3: {
    type: 'postgres',
    host: 'db-shard3.example.com',
    port: 5432,
    username: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    database: 'app_shard3',
    entities: [User, Order],
  },
};

// database/shard.module.ts
@Module({
  imports: [
    TypeOrmModule.forRoot({
      ...SHARD_CONNECTIONS.shard1,
      name: 'shard1',
    }),
    TypeOrmModule.forRoot({
      ...SHARD_CONNECTIONS.shard2,
      name: 'shard2',
    }),
    TypeOrmModule.forRoot({
      ...SHARD_CONNECTIONS.shard3,
      name: 'shard3',
    }),
  ],
})
export class ShardModule {}
```

**3. Shard-Aware Repository:**

```typescript
// users/shard-aware-user.repository.ts
@Injectable()
export class ShardAwareUserRepository {
  private shards: Map<string, DataSource>;

  constructor(
    private shardingService: UserShardingService,
    @InjectDataSource('shard1') shard1: DataSource,
    @InjectDataSource('shard2') shard2: DataSource,
    @InjectDataSource('shard3') shard3: DataSource,
  ) {
    this.shards = new Map([
      ['shard1', shard1],
      ['shard2', shard2],
      ['shard3', shard3],
    ]);
  }

  async findById(userId: number): Promise<User> {
    const shardName = this.shardingService.getShardForUser(userId);
    const shard = this.shards.get(shardName);
    
    return shard.getRepository(User).findOne({ where: { id: userId } });
  }

  async create(userData: CreateUserDto): Promise<User> {
    // Assign user ID first (maybe from distributed ID generator)
    const userId = await this.generateUserId();
    
    const shardName = this.shardingService.getShardForUser(userId);
    const shard = this.shards.get(shardName);
    
    const user = shard.getRepository(User).create({ ...userData, id: userId });
    return shard.getRepository(User).save(user);
  }

  async findByEmail(email: string): Promise<User> {
    // Query all shards (scatter-gather)
    const promises = Array.from(this.shards.values()).map(shard =>
      shard.getRepository(User).findOne({ where: { email } }),
    );

    const results = await Promise.all(promises);
    return results.find(user => user !== null);
  }

  private async generateUserId(): Promise<number> {
    // Use distributed ID generator (Snowflake, Twitter's Snowflake, etc.)
    return Date.now(); // Simplified
  }
}
```

**4. Cross-Shard Queries:**

```typescript
// When to shard and avoid cross-shard joins
@Injectable()
export class OrderService {
  // ✅ GOOD: Query within single shard
  async getUserOrders(userId: number) {
    const shardName = this.shardingService.getShardForUser(userId);
    return this.getShardRepository(shardName)
      .find({ where: { userId } });
  }

  // ❌ BAD: Cross-shard join (avoid!)
  async getOrdersWithUserDetails() {
    // This requires joining users from different shards
    // Solution: Denormalize data or use application-level joins
  }

  // ✅ GOOD: Denormalization
  @Entity()
  class Order {
    @Column()
    userId: number;

    // Denormalized user data to avoid cross-shard join
    @Column()
    userName: string;

    @Column()
    userEmail: string;
  }
}
```

**5. Shard Mapping Table:**

```typescript
// Global routing database (non-sharded)
@Entity('shard_mappings')
export class ShardMapping {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  entityId: string; // User ID, Order ID, etc.

  @Column()
  entityType: string; // 'user', 'order'

  @Column()
  shardName: string; // 'shard1', 'shard2', 'shard3'

  @CreateDateColumn()
  createdAt: Date;
}

// Lookup service
@Injectable()
export class ShardMappingService {
  constructor(
    @InjectRepository(ShardMapping)
    private mappingRepo: Repository<ShardMapping>,
  ) {}

  async getShardForEntity(entityId: string, entityType: string): Promise<string> {
    const mapping = await this.mappingRepo.findOne({
      where: { entityId, entityType },
    });

    if (!mapping) {
      throw new NotFoundException('Entity shard mapping not found');
    }

    return mapping.shardName;
  }

  async recordShardMapping(entityId: string, entityType: string, shardName: string) {
    await this.mappingRepo.save({ entityId, entityType, shardName });
  }
}
```

**6. When to Use Sharding:**

```typescript
/*
USE SHARDING WHEN:
1. Single database cannot handle read/write load
   - > 10,000 writes/second
   - > 100GB data size growing rapidly
   
2. Vertical scaling maxed out
   - Already using largest DB instance
   - Cost of scaling up prohibitive

3. Geographic distribution needed
   - Comply with data residency laws
   - Reduce latency for global users

4. Clear sharding key exists
   - User ID, tenant ID, region
   - Queries mostly within shard

AVOID SHARDING IF:
1. Can use read replicas instead
2. Require complex cross-shard joins
3. Dataset < 100GB
4. No clear sharding key
5. Need ACID transactions across shards

SHARDING CHALLENGES:
- Rebalancing data when adding shards
- Cross-shard queries expensive
- Complex application logic
- Distributed transactions difficult
- Schema changes across all shards
*/
```

**7. Rebalancing Strategy:**

```typescript
// Adding new shard (reshard)
@Injectable()
export class ReshardService {
  async addNewShard(newShardName: string) {
    // 1. Add new shard connection
    // 2. Update sharding logic to include new shard
    // 3. Migrate subset of data to new shard
    // 4. Update shard mappings
    // 5. Gradually route traffic to new shard
  }

  async migrateShard(fromShard: string, toShard: string, userIds: number[]) {
    for (const userId of userIds) {
      const user = await this.getUserFromShard(userId, fromShard);
      await this.saveUserToShard(user, toShard);
      await this.updateShardMapping(userId, toShard);
      await this.deleteUserFromShard(userId, fromShard);
    }
  }
}
```

**Key Takeaway:** Sharding horizontally partitions data across databases when single DB can't handle load (>100GB data, >10k writes/sec). Strategies: range-based (userId < 1000000 → shard1), hash-based (hash(orderId) % shardCount), geographic (country → regional shard). Configure multiple TypeORM DataSources with @InjectDataSource('shard1'). Shard-aware repositories route queries to correct shard. Avoid cross-shard JOINs (use denormalization). Maintain shard mapping table for lookups. Use when vertical scaling maxed and clear sharding key exists. Challenges: rebalancing, distributed transactions, complex queries.

</details>

<details>
<summary><strong>23. How do you implement read replicas for databases?</strong></summary>

**Answer:**

Implement read replicas to distribute read load and improve query performance:

**1. TypeORM Replication Configuration:**

```typescript
// database/database.config.ts
export default {
  type: 'postgres',
  replication: {
    master: {
      host: process.env.DB_MASTER_HOST || 'postgres-master',
      port: 5432,
      username: process.env.DB_USER,
      password: process.env.DB_PASSWORD,
      database: process.env.DB_NAME,
    },
    slaves: [
      {
        host: process.env.DB_REPLICA1_HOST || 'postgres-replica1',
        port: 5432,
        username: process.env.DB_USER,
        password: process.env.DB_PASSWORD,
        database: process.env.DB_NAME,
      },
      {
        host: process.env.DB_REPLICA2_HOST || 'postgres-replica2',
        port: 5432,
        username: process.env.DB_USER,
        password: process.env.DB_PASSWORD,
        database: process.env.DB_NAME,
      },
    ],
  },
  entities: [__dirname + '/../**/*.entity{.ts,.js}'],
  synchronize: false,
  logging: true,
};

// app.module.ts
@Module({
  imports: [
    TypeOrmModule.forRoot(databaseConfig),
  ],
})
export class AppModule {}
```

**2. Explicit Replica Selection:**

```typescript
// Reads automatically go to replicas, writes to master
@Injectable()
export class ProductService {
  constructor(
    @InjectRepository(Product)
    private productRepo: Repository<Product>,
  ) {}

  // Read from replica (automatic)
  async getProducts() {
    return this.productRepo.find(); // Uses replica
  }

  // Write to master (automatic)
  async createProduct(dto: CreateProductDto) {
    return this.productRepo.save(dto); // Uses master
  }

  // Force master for consistency
  async getProductConsistent(id: string) {
    return this.productRepo
      .createQueryBuilder('product', 'master') // Explicitly use master
      .where('product.id = :id', { id })
      .getOne();
  }

  // Use specific replica
  async getProductsFromReplica() {
    return this.productRepo
      .createQueryBuilder('product', 'slave') // Use replica
      .getMany();
  }
}
```

**3. PostgreSQL Master-Replica Setup:**

```sql
-- Master configuration (postgresql.conf)
wal_level = replica
max_wal_senders = 3
wal_keep_size = 64MB

-- Create replication user
CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD 'password';

-- Replica configuration (recovery.conf or postgresql.conf in PG 12+)
primary_conninfo = 'host=postgres-master port=5432 user=replicator password=password'
```

**4. Docker Compose with Replicas:**

```yaml
# docker-compose.yml
version: '3.8'

services:
  postgres-master:
    image: postgres:15
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_REPLICATION_USER: replicator
      POSTGRES_REPLICATION_PASSWORD: replpass
    volumes:
      - master-data:/var/lib/postgresql/data
    command: |
      postgres
      -c wal_level=replica
      -c max_wal_senders=3
      -c max_replication_slots=3
    ports:
      - "5432:5432"

  postgres-replica1:
    image: postgres:15
    environment:
      PGUSER: replicator
      PGPASSWORD: replpass
    command: |
      bash -c "
      until pg_basebackup -h postgres-master -D /var/lib/postgresql/data -U replicator -vP -W
      do
        echo 'Waiting for master to be ready'
        sleep 1s
      done
      echo 'primary_conninfo = ''host=postgres-master port=5432 user=replicator password=replpass''' > /var/lib/postgresql/data/postgresql.auto.conf
      postgres
      "
    volumes:
      - replica1-data:/var/lib/postgresql/data
    depends_on:
      - postgres-master

volumes:
  master-data:
  replica1-data:
```

**5. Handle Replication Lag:**

```typescript
// Check replication lag before critical reads
@Injectable()
export class ReplicationService {
  constructor(
    @InjectDataSource() private dataSource: DataSource,
  ) {}

  async getReplicationLag(): Promise<number> {
    // Query replica for lag in seconds
    const result = await this.dataSource.query(`
      SELECT EXTRACT(EPOCH FROM (now() - pg_last_xact_replay_timestamp()))::int AS lag_seconds
    `);
    
    return result[0]?.lag_seconds || 0;
  }

  async waitForReplication(maxWaitMs: number = 5000) {
    const startTime = Date.now();
    
    while (Date.now() - startTime < maxWaitMs) {
      const lag = await this.getReplicationLag();
      
      if (lag < 1) { // Less than 1 second lag
        return;
      }
      
      await new Promise(resolve => setTimeout(resolve, 100));
    }
    
    throw new Error('Replication lag timeout');
  }
}

// Usage for read-after-write consistency
@Injectable()
export class OrderService {
  constructor(
    private replicationService: ReplicationService,
  ) {}

  async createAndGet(dto: CreateOrderDto) {
    const order = await this.orderRepo.save(dto); // Write to master
    
    // Wait for replication before reading
    await this.replicationService.waitForReplication();
    
    return this.orderRepo.findOne({ where: { id: order.id } }); // Read from replica
  }
}
```

**6. Health Check for Replicas:**

```typescript
// health/replica-health.indicator.ts
import { Injectable } from '@nestjs/common';
import { HealthIndicator, HealthIndicatorResult, HealthCheckError } from '@nestjs/terminus';

@Injectable()
export class ReplicaHealthIndicator extends HealthIndicator {
  constructor(@InjectDataSource() private dataSource: DataSource) {
    super();
  }

  async isHealthy(key: string): Promise<HealthIndicatorResult> {
    try {
      // Check if replica is replicating
      const result = await this.dataSource.query(`
        SELECT pg_is_in_recovery() AS is_replica,
               pg_last_wal_receive_lsn() AS receive_lsn,
               pg_last_wal_replay_lsn() AS replay_lsn
      `);

      const isHealthy = result[0].is_replica === true;

      if (!isHealthy) {
        throw new Error('Replica not in recovery mode');
      }

      return this.getStatus(key, true, { replica: 'active' });
    } catch (error) {
      throw new HealthCheckError('Replica check failed', this.getStatus(key, false));
    }
  }
}
```

**7. Read Preference Strategy:**

```typescript
// Advanced: Route based on query type
export enum ReadPreference {
  PRIMARY = 'primary', // Always master
  PRIMARY_PREFERRED = 'primaryPreferred', // Master unless down
  SECONDARY = 'secondary', // Always replica
  SECONDARY_PREFERRED = 'secondaryPreferred', // Replica unless down
  NEAREST = 'nearest', // Lowest latency
}

@Injectable()
export class SmartQueryRouter {
  constructor(
    @InjectDataSource('master') private master: DataSource,
    @InjectDataSource('replica1') private replica1: DataSource,
    @InjectDataSource('replica2') private replica2: DataSource,
  ) {}

  getDataSource(preference: ReadPreference): DataSource {
    switch (preference) {
      case ReadPreference.PRIMARY:
        return this.master;
      
      case ReadPreference.SECONDARY:
        // Round-robin between replicas
        return Math.random() > 0.5 ? this.replica1 : this.replica2;
      
      case ReadPreference.SECONDARY_PREFERRED:
        // Try replica, fallback to master if unavailable
        return this.replica1 || this.master;
      
      default:
        return this.master;
    }
  }
}
```

**Key Takeaway:** Read replicas configured in TypeORM with replication: { master, slaves } separate read/write traffic. Reads automatically use replicas (slaves), writes use master. Force master with .createQueryBuilder('entity', 'master') for read-after-write consistency. PostgreSQL requires wal_level=replica and replication user. Handle replication lag by checking pg_last_xact_replay_timestamp() and waiting if needed. Use multiple replicas for load distribution. Benefits: horizontal read scaling, geographic distribution, backup/failover. Limitation: eventual consistency (lag typically <1s).

</details>

<details>
<summary><strong>24. How do you use CDN for static assets?</strong></summary>

**Answer:**

Use CDN (CloudFront, Cloudflare) to cache and serve static assets globally:

**1. S3 + CloudFront Setup:**

```typescript
// config/cdn.config.ts
export const CDN_CONFIG = {
  enabled: process.env.CDN_ENABLED === 'true',
  domain: process.env.CDN_DOMAIN, // d123456.cloudfront.net
  s3Bucket: process.env.S3_BUCKET,
  s3Region: process.env.AWS_REGION,
};

// assets/asset-url.service.ts
@Injectable()
export class AssetUrlService {
  getAssetUrl(path: string): string {
    if (CDN_CONFIG.enabled) {
      return `https://${CDN_CONFIG.domain}/${path}`;
    }
    
    // Fallback to S3 direct
    return `https://${CDN_CONFIG.s3Bucket}.s3.${CDN_CONFIG.s3Region}.amazonaws.com/${path}`;
  }

  getImageUrl(key: string, transforms?: ImageTransforms): string {
    let url = this.getAssetUrl(`images/${key}`);
    
    // CloudFront with Lambda@Edge for image transformations
    if (transforms) {
      const params = new URLSearchParams();
      if (transforms.width) params.set('w', transforms.width.toString());
      if (transforms.height) params.set('h', transforms.height.toString());
      if (transforms.quality) params.set('q', transforms.quality.toString());
      
      url += `?${params.toString()}`;
    }
    
    return url;
  }
}
```

**2. Upload with Cache Headers:**

```typescript
// files/cdn-upload.service.ts
@Injectable()
export class CdnUploadService {
  constructor(private s3: S3) {}

  async uploadAsset(file: Express.Multer.File, folder: string) {
    const key = `${folder}/${Date.now()}-${file.originalname}`;
    
    await this.s3.putObject({
      Bucket: CDN_CONFIG.s3Bucket,
      Key: key,
      Body: file.buffer,
      ContentType: file.mimetype,
      
      // CDN caching headers
      CacheControl: 'public, max-age=31536000, immutable', // 1 year
      
      // Optional: Set ACL for public read
      ACL: 'public-read',
      
      // Metadata
      Metadata: {
        originalName: file.originalname,
        uploadedAt: new Date().toISOString(),
      },
    }).promise();

    return this.assetUrlService.getAssetUrl(key);
  }

  async uploadImage(file: Express.Multer.File) {
    // Optimize before upload
    const optimized = await sharp(file.buffer)
      .resize(2000, 2000, { fit: 'inside', withoutEnlargement: true })
      .jpeg({ quality: 85, progressive: true })
      .toBuffer();

    const key = `images/${Date.now()}-${file.originalname}`;
    
    await this.s3.putObject({
      Bucket: CDN_CONFIG.s3Bucket,
      Key: key,
      Body: optimized,
      ContentType: 'image/jpeg',
      CacheControl: 'public, max-age=31536000, immutable',
    }).promise();

    return this.assetUrlService.getImageUrl(key);
  }
}
```

**3. API Response with CDN URLs:**

```typescript
// products/products.service.ts
@Injectable()
export class ProductsService {
  constructor(private assetUrlService: AssetUrlService) {}

  async getProduct(id: string) {
    const product = await this.productRepo.findOne({ where: { id } });
    
    // Transform S3 keys to CDN URLs
    return {
      ...product,
      images: product.images.map(key => 
        this.assetUrlService.getImageUrl(key, { width: 800, quality: 85 })
      ),
      thumbnail: this.assetUrlService.getImageUrl(product.thumbnail, {
        width: 200,
        height: 200,
        quality: 75,
      }),
    };
  }
}
```

**4. CloudFront Configuration (Terraform):**

```hcl
# infrastructure/cloudfront.tf
resource "aws_cloudfront_distribution" "cdn" {
  enabled             = true
  is_ipv6_enabled     = true
  price_class         = "PriceClass_All"
  
  origin {
    domain_name = aws_s3_bucket.assets.bucket_regional_domain_name
    origin_id   = "S3-assets"
    
    s3_origin_config {
      origin_access_identity = aws_cloudfront_origin_access_identity.oai.cloudfront_access_identity_path
    }
  }

  default_cache_behavior {
    allowed_methods  = ["GET", "HEAD", "OPTIONS"]
    cached_methods   = ["GET", "HEAD"]
    target_origin_id = "S3-assets"

    forwarded_values {
      query_string = true  # For image transformations
      headers      = ["Origin"]

      cookies {
        forward = "none"
      }
    }

    viewer_protocol_policy = "redirect-to-https"
    min_ttl                = 0
    default_ttl            = 86400    # 1 day
    max_ttl                = 31536000 # 1 year
    compress               = true
  }

  restrictions {
    geo_restriction {
      restriction_type = "none"
    }
  }

  viewer_certificate {
    cloudfront_default_certificate = true
  }
}
```

**5. Cache Invalidation:**

```typescript
// cdn/cdn-invalidation.service.ts
import { CloudFront } from 'aws-sdk';

@Injectable()
export class CdnInvalidationService {
  private cloudfront: CloudFront;

  constructor() {
    this.cloudfront = new CloudFront({
      region: process.env.AWS_REGION,
    });
  }

  async invalidatePaths(paths: string[]) {
    const params = {
      DistributionId: process.env.CLOUDFRONT_DISTRIBUTION_ID,
      InvalidationBatch: {
        CallerReference: Date.now().toString(),
        Paths: {
          Quantity: paths.length,
          Items: paths.map(path => `/${path}`),
        },
      },
    };

    const result = await this.cloudfront.createInvalidation(params).promise();
    return result.Invalidation.Id;
  }

  async invalidateAll() {
    return this.invalidatePaths(['/*']); // Invalidate everything
  }
}

// Usage when updating product images
@Injectable()
export class ProductsService {
  async updateProductImage(productId: string, newImage: string) {
    const product = await this.productRepo.findOne({ where: { id: productId } });
    
    // Update database
    product.image = newImage;
    await this.productRepo.save(product);

    // Invalidate old CDN cache
    await this.cdnInvalidationService.invalidatePaths([
      `images/${product.image}`,
    ]);
  }
}
```

**6. Serve Static Files through CDN:**

```typescript
// Serve NestJS static assets through CDN in production
// main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  if (process.env.NODE_ENV === 'development') {
    // Serve locally in dev
    app.useStaticAssets(join(__dirname, '..', 'public'));
  }
  // In production, reference CDN URLs in templates

  await app.listen(3000);
}

// HTML template with CDN
const html = `
  <!DOCTYPE html>
  <html>
    <head>
      <link rel="stylesheet" href="${process.env.CDN_URL}/css/styles.css">
    </head>
    <body>
      <img src="${process.env.CDN_URL}/images/logo.png" alt="Logo">
      <script src="${process.env.CDN_URL}/js/app.js"></script>
    </body>
  </html>
`;
```

**7. Cloudflare CDN Configuration:**

```typescript
// Alternative: Cloudflare CDN with Cloudflare Workers
// cloudflare-worker.js (edge computing)
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  const url = new URL(request.url)
  
  // Image transformation at edge
  if (url.pathname.startsWith('/images/')) {
    const width = url.searchParams.get('w') || '800'
    const quality = url.searchParams.get('q') || '85'
    
    // Cloudflare Image Resizing
    return fetch(request, {
      cf: {
        image: {
          width: parseInt(width),
          quality: parseInt(quality),
          format: 'auto', // WebP for supported browsers
        }
      }
    })
  }
  
  return fetch(request)
}
```

**Key Takeaway:** CDN caches static assets (images, CSS, JS) at edge locations globally reducing latency. Upload to S3 with CacheControl: 'public, max-age=31536000' for long TTL. Configure CloudFront distribution with S3 origin. Use assetUrlService to generate CDN URLs (https://d123456.cloudfront.net/path). Support image transformations via query params (?w=800&q=85) with Lambda@Edge or Cloudflare Workers. Invalidate cache after updates with cloudfront.createInvalidation(). Benefits: faster load times, reduced origin load, geographic distribution, DDoS protection. Optimize images before upload (sharp resize/compress).

</details>

<details>
<summary><strong>25. How do you implement distributed caching (Redis cluster)?</strong></summary>

**Answer:**

Implement Redis cluster for distributed caching with high availability and partitioning:

**1. Redis Cluster Configuration:**

```typescript
// cache/redis-cluster.config.ts
import { CacheModule } from '@nestjs/cache-manager';
import { redisStore } from 'cache-manager-redis-yet';

@Module({
  imports: [
    CacheModule.registerAsync({
      isGlobal: true,
      useFactory: async () => {
        return {
          store: redisStore,
          // Redis Cluster configuration
          clusterConfig: {
            nodes: [
              { host: 'redis-node-1', port: 6379 },
              { host: 'redis-node-2', port: 6379 },
              { host: 'redis-node-3', port: 6379 },
              { host: 'redis-node-4', port: 6379 },
              { host: 'redis-node-5', port: 6379 },
              { host: 'redis-node-6', port: 6379 },
            ],
            options: {
              redisOptions: {
                password: process.env.REDIS_PASSWORD,
              },
              // Retry strategy
              retryDelayOnFailover: 100,
              retryDelayOnClusterDown: 300,
              maxRedirections: 16,
              scaleReads: 'slave', // Read from replicas
            },
          },
          ttl: 300, // Default 5 minutes
        };
      },
    }),
  ],
})
export class CacheConfigModule {}
```

**2. IORedis Cluster Client:**

```typescript
// cache/redis.provider.ts
import { Cluster } from 'ioredis';

export const REDIS_CLUSTER = 'REDIS_CLUSTER';

export const redisClusterProvider = {
  provide: REDIS_CLUSTER,
  useFactory: () => {
    const cluster = new Cluster([
      { host: 'redis-node-1', port: 6379 },
      { host: 'redis-node-2', port: 6379 },
      { host: 'redis-node-3', port: 6379 },
    ], {
      redisOptions: {
        password: process.env.REDIS_PASSWORD,
      },
      clusterRetryStrategy: (times) => {
        return Math.min(100 * times, 2000);
      },
      enableReadyCheck: true,
      maxRedirections: 16,
      scaleReads: 'slave',
    });

    cluster.on('error', (err) => console.error('Redis Cluster Error:', err));
    cluster.on('connect', () => console.log('Redis Cluster Connected'));
    cluster.on('ready', () => console.log('Redis Cluster Ready'));

    return cluster;
  },
};

@Module({
  providers: [redisClusterProvider],
  exports: [REDIS_CLUSTER],
})
export class RedisModule {}
```

**3. Distributed Cache Service:**

```typescript
// cache/distributed-cache.service.ts
import { Injectable, Inject } from '@nestjs/common';
import { CACHE_MANAGER } from '@nestjs/cache-manager';
import { Cache } from 'cache-manager';
import { Cluster } from 'ioredis';
import { REDIS_CLUSTER } from './redis.provider';

@Injectable()
export class DistributedCacheService {
  constructor(
    @Inject(CACHE_MANAGER) private cacheManager: Cache,
    @Inject(REDIS_CLUSTER) private redisCluster: Cluster,
  ) {}

  // Simple get/set with cache-manager
  async get<T>(key: string): Promise<T | null> {
    return this.cacheManager.get<T>(key);
  }

  async set<T>(key: string, value: T, ttl?: number): Promise<void> {
    await this.cacheManager.set(key, value, ttl);
  }

  async del(key: string): Promise<void> {
    await this.cacheManager.del(key);
  }

  // Advanced operations with ioredis cluster
  async mget(keys: string[]): Promise<(string | null)[]> {
    // Multi-get (may hit multiple nodes)
    return this.redisCluster.mget(...keys);
  }

  async mset(keyValues: Record<string, any>): Promise<void> {
    const pairs = Object.entries(keyValues).flat();
    await this.redisCluster.mset(...pairs);
  }

  // Hash operations (stays on same node due to hash tag)
  async hset(key: string, field: string, value: any): Promise<void> {
    await this.redisCluster.hset(key, field, JSON.stringify(value));
  }

  async hget(key: string, field: string): Promise<any> {
    const value = await this.redisCluster.hget(key, field);
    return value ? JSON.parse(value) : null;
  }

  async hgetall(key: string): Promise<Record<string, any>> {
    const data = await this.redisCluster.hgetall(key);
    const result: Record<string, any> = {};
    
    for (const [field, value] of Object.entries(data)) {
      result[field] = JSON.parse(value);
    }
    
    return result;
  }

  // Sorted sets for leaderboards
  async zadd(key: string, score: number, member: string): Promise<void> {
    await this.redisCluster.zadd(key, score, member);
  }

  async zrange(key: string, start: number, stop: number): Promise<string[]> {
    return this.redisCluster.zrange(key, start, stop);
  }

  async zrevrange(key: string, start: number, stop: number): Promise<string[]> {
    return this.redisCluster.zrevrange(key, start, stop);
  }

  // Pub/Sub (works across cluster)
  async publish(channel: string, message: any): Promise<void> {
    await this.redisCluster.publish(channel, JSON.stringify(message));
  }

  subscribe(channel: string, callback: (message: any) => void): void {
    this.redisCluster.subscribe(channel);
    this.redisCluster.on('message', (ch, msg) => {
      if (ch === channel) {
        callback(JSON.parse(msg));
      }
    });
  }

  // Cache invalidation pattern
  async invalidatePattern(pattern: string): Promise<void> {
    // Get all keys matching pattern across all nodes
    const nodes = this.redisCluster.nodes('master');
    
    for (const node of nodes) {
      let cursor = '0';
      do {
        const [newCursor, keys] = await node.scan(
          cursor,
          'MATCH',
          pattern,
          'COUNT',
          100,
        );
        cursor = newCursor;
        
        if (keys.length > 0) {
          await this.redisCluster.del(...keys);
        }
      } while (cursor !== '0');
    }
  }
}
```

**4. Cache-Aside Pattern:**

```typescript
// products/products.service.ts
@Injectable()
export class ProductsService {
  constructor(
    @InjectRepository(Product) private productRepo: Repository<Product>,
    private cache: DistributedCacheService,
  ) {}

  async getProduct(id: string): Promise<Product> {
    const cacheKey = `product:${id}`;
    
    // Try cache first
    const cached = await this.cache.get<Product>(cacheKey);
    if (cached) {
      return cached;
    }

    // Cache miss - fetch from database
    const product = await this.productRepo.findOne({ where: { id } });
    
    if (!product) {
      throw new NotFoundException('Product not found');
    }

    // Store in cache (TTL 1 hour)
    await this.cache.set(cacheKey, product, 3600);
    
    return product;
  }

  async updateProduct(id: string, dto: UpdateProductDto): Promise<Product> {
    const product = await this.productRepo.findOne({ where: { id } });
    Object.assign(product, dto);
    
    const updated = await this.productRepo.save(product);
    
    // Invalidate cache
    await this.cache.del(`product:${id}`);
    
    return updated;
  }

  async searchProducts(query: string): Promise<Product[]> {
    const cacheKey = `search:${query}`;
    
    const cached = await this.cache.get<Product[]>(cacheKey);
    if (cached) return cached;

    const products = await this.productRepo
      .createQueryBuilder('product')
      .where('product.name ILIKE :query', { query: `%${query}%` })
      .getMany();

    // Cache search results for 5 minutes
    await this.cache.set(cacheKey, products, 300);
    
    return products;
  }
}
```

**5. Session Storage in Cluster:**

```typescript
// auth/session.service.ts
@Injectable()
export class SessionService {
  constructor(private cache: DistributedCacheService) {}

  async createSession(userId: string, data: any): Promise<string> {
    const sessionId = randomBytes(32).toString('hex');
    const sessionKey = `session:${sessionId}`;
    
    await this.cache.set(sessionKey, {
      userId,
      ...data,
      createdAt: new Date(),
    }, 86400); // 24 hours

    return sessionId;
  }

  async getSession(sessionId: string): Promise<any> {
    return this.cache.get(`session:${sessionId}`);
  }

  async extendSession(sessionId: string): Promise<void> {
    const session = await this.getSession(sessionId);
    if (session) {
      await this.cache.set(`session:${sessionId}`, session, 86400);
    }
  }

  async destroySession(sessionId: string): Promise<void> {
    await this.cache.del(`session:${sessionId}`);
  }
}
```

**6. Rate Limiting with Cluster:**

```typescript
// rate-limit/cluster-rate-limit.service.ts
@Injectable()
export class ClusterRateLimitService {
  constructor(private cache: DistributedCacheService) {}

  async checkRateLimit(
    key: string,
    limit: number,
    windowSeconds: number,
  ): Promise<{ allowed: boolean; remaining: number }> {
    const now = Date.now();
    const windowKey = `ratelimit:${key}:${Math.floor(now / (windowSeconds * 1000))}`;

    // Increment counter
    const count = await this.cache.redisCluster.incr(windowKey);
    
    // Set expiry on first request
    if (count === 1) {
      await this.cache.redisCluster.expire(windowKey, windowSeconds);
    }

    const allowed = count <= limit;
    const remaining = Math.max(0, limit - count);

    return { allowed, remaining };
  }

  // Sliding window rate limit (more accurate)
  async slidingWindowRateLimit(
    key: string,
    limit: number,
    windowSeconds: number,
  ): Promise<boolean> {
    const now = Date.now();
    const windowStart = now - windowSeconds * 1000;
    const sortedSetKey = `ratelimit:sliding:${key}`;

    // Remove old entries
    await this.cache.redisCluster.zremrangebyscore(sortedSetKey, 0, windowStart);

    // Count current requests
    const count = await this.cache.redisCluster.zcard(sortedSetKey);

    if (count >= limit) {
      return false; // Rate limit exceeded
    }

    // Add current request
    await this.cache.redisCluster.zadd(sortedSetKey, now, `${now}-${Math.random()}`);
    
    // Set expiry
    await this.cache.redisCluster.expire(sortedSetKey, windowSeconds);

    return true;
  }
}
```

**7. Docker Compose Redis Cluster:**

```yaml
# docker-compose.redis-cluster.yml
version: '3.8'

services:
  redis-node-1:
    image: redis:7-alpine
    command: redis-server --cluster-enabled yes --cluster-config-file nodes.conf --cluster-node-timeout 5000 --appendonly yes --port 6379
    ports:
      - "6379:6379"
    volumes:
      - redis-1-data:/data

  redis-node-2:
    image: redis:7-alpine
    command: redis-server --cluster-enabled yes --cluster-config-file nodes.conf --cluster-node-timeout 5000 --appendonly yes --port 6379
    ports:
      - "6380:6379"
    volumes:
      - redis-2-data:/data

  redis-node-3:
    image: redis:7-alpine
    command: redis-server --cluster-enabled yes --cluster-config-file nodes.conf --cluster-node-timeout 5000 --appendonly yes --port 6379
    ports:
      - "6381:6379"
    volumes:
      - redis-3-data:/data

  redis-node-4:
    image: redis:7-alpine
    command: redis-server --cluster-enabled yes --cluster-config-file nodes.conf --cluster-node-timeout 5000 --appendonly yes --port 6379
    ports:
      - "6382:6379"
    volumes:
      - redis-4-data:/data

  redis-node-5:
    image: redis:7-alpine
    command: redis-server --cluster-enabled yes --cluster-config-file nodes.conf --cluster-node-timeout 5000 --appendonly yes --port 6379
    ports:
      - "6383:6379"
    volumes:
      - redis-5-data:/data

  redis-node-6:
    image: redis:7-alpine
    command: redis-server --cluster-enabled yes --cluster-config-file nodes.conf --cluster-node-timeout 5000 --appendonly yes --port 6379
    ports:
      - "6384:6379"
    volumes:
      - redis-6-data:/data

  # Cluster creation (run once)
  redis-cluster-init:
    image: redis:7-alpine
    depends_on:
      - redis-node-1
      - redis-node-2
      - redis-node-3
      - redis-node-4
      - redis-node-5
      - redis-node-6
    command: >
      sh -c "sleep 5 && redis-cli --cluster create
      redis-node-1:6379
      redis-node-2:6379
      redis-node-3:6379
      redis-node-4:6379
      redis-node-5:6379
      redis-node-6:6379
      --cluster-replicas 1 --cluster-yes"

volumes:
  redis-1-data:
  redis-2-data:
  redis-3-data:
  redis-4-data:
  redis-5-data:
  redis-6-data:
```

**8. Cache Warming Strategy:**

```typescript
// cache/cache-warming.service.ts
@Injectable()
export class CacheWarmingService {
  constructor(
    private cache: DistributedCacheService,
    private productsService: ProductsService,
  ) {}

  @Cron(CronExpression.EVERY_HOUR)
  async warmPopularProducts() {
    // Fetch most popular products from analytics
    const popularProductIds = await this.getPopularProductIds();

    for (const productId of popularProductIds) {
      const product = await this.productsService.getProductFromDb(productId);
      await this.cache.set(`product:${productId}`, product, 3600);
    }
  }

  async warmUserSpecificCache(userId: string) {
    // Pre-cache user's frequent queries
    const userPreferences = await this.getUserPreferences(userId);
    
    for (const preference of userPreferences) {
      const data = await this.fetchData(preference);
      await this.cache.set(`user:${userId}:${preference}`, data, 1800);
    }
  }

  private async getPopularProductIds(): Promise<string[]> {
    // Implementation to get top 100 products
    return [];
  }
}
```

**Key Takeaway:** Redis Cluster provides distributed caching with automatic sharding across nodes (minimum 6 nodes: 3 masters + 3 replicas). Configure with ioredis Cluster client specifying all nodes. Data partitioned by hash slots (16384 slots). Use hash tags `{user:123}` to keep related keys on same node. Implement cache-aside pattern: check cache → DB on miss → populate cache. Session storage distributed across cluster. Rate limiting with INCR for fixed window or ZSET for sliding window. Benefits: horizontal scalability, high availability with automatic failover, no single point of failure. Use scaleReads: 'slave' to offload reads to replicas. Invalidate with DEL or pattern matching with SCAN across all master nodes.

</details>

## Microservices Architecture

<details>
<summary><strong>26. When should you split a monolith into microservices?</strong></summary>

**Answer:**

Split a monolith into microservices when specific conditions are met:

**1. Signs You Need Microservices:**

```typescript
/*
SPLIT MONOLITH WHEN:

1. Team Scaling Issues
   - Multiple teams working on same codebase causing conflicts
   - Deployment bottlenecks (one team blocks others)
   - Different teams need different release cycles

2. Technical Scalability
   - Different components need independent scaling
   - One module consumes disproportionate resources
   - Cannot scale monolith cost-effectively

3. Technology Diversity
   - Different parts benefit from different tech stacks
   - Need specialized databases per domain
   - Want to experiment with new technologies safely

4. Business Domain Separation
   - Clear bounded contexts exist
   - Different business units own different parts
   - Parts have independent business logic

5. Deployment & Reliability
   - Changes in one area shouldn't risk entire system
   - Need independent deployment cycles
   - Want isolated failure domains

DON'T SPLIT IF:
- Small team (<10 developers)
- Unclear domain boundaries
- Low traffic (<1000 req/sec)
- Shared database is acceptable
- No independent scaling needs
- Team lacks distributed systems experience
*/
```

**2. Domain-Driven Design Approach:**

```typescript
// Identify Bounded Contexts (example: e-commerce)

// ❌ MONOLITH - Everything coupled
@Module({
  imports: [
    // All in one
    UsersModule,
    ProductsModule,
    OrdersModule,
    PaymentsModule,
    InventoryModule,
    ShippingModule,
    NotificationsModule,
  ],
})
export class MonolithAppModule {}

// ✅ MICROSERVICES - Separated by domain

// 1. User Service (Identity & Access Management)
@Module({
  imports: [
    TypeOrmModule.forFeature([User, Role, Permission]),
  ],
  controllers: [UsersController, AuthController],
  providers: [UsersService, AuthService, JwtStrategy],
})
export class UserServiceModule {}

// 2. Product Catalog Service
@Module({
  imports: [
    TypeOrmModule.forFeature([Product, Category, ProductVariant]),
  ],
  controllers: [ProductsController, CategoriesController],
  providers: [ProductsService, SearchService],
})
export class ProductServiceModule {}

// 3. Order Management Service
@Module({
  imports: [
    TypeOrmModule.forFeature([Order, OrderItem, Cart]),
    ClientsModule.register([
      { name: 'PAYMENT_SERVICE', transport: Transport.TCP, options: { host: 'payment-service', port: 3001 } },
      { name: 'INVENTORY_SERVICE', transport: Transport.TCP, options: { host: 'inventory-service', port: 3002 } },
    ]),
  ],
  controllers: [OrdersController],
  providers: [OrdersService],
})
export class OrderServiceModule {}

// 4. Payment Service
@Module({
  imports: [
    TypeOrmModule.forFeature([Payment, Refund]),
  ],
  controllers: [PaymentsController],
  providers: [PaymentsService, StripeService],
})
export class PaymentServiceModule {}

// 5. Inventory Service
@Module({
  imports: [
    TypeOrmModule.forFeature([Inventory, StockMovement]),
  ],
  controllers: [InventoryController],
  providers: [InventoryService],
})
export class InventoryServiceModule {}
```

**3. Strangler Fig Pattern (Gradual Migration):**

```typescript
// Step 1: Extract one service at a time
// Start with least coupled, high-value service

// Original Monolith
@Controller('api/payments')
export class MonolithPaymentsController {
  // All payment logic here
}

// Step 2: Create new microservice
// new payment-service/src/main.ts
async function bootstrap() {
  const app = await NestFactory.create(PaymentServiceModule);
  await app.listen(3001);
}

// Step 3: Proxy from monolith to new service
@Controller('api/payments')
export class ProxyPaymentsController {
  constructor(
    @Inject('PAYMENT_SERVICE') private paymentClient: ClientProxy,
  ) {}

  @Post()
  createPayment(@Body() dto: CreatePaymentDto) {
    // Forward to microservice
    return this.paymentClient.send('create_payment', dto);
  }

  @Get(':id')
  getPayment(@Param('id') id: string) {
    return this.paymentClient.send('get_payment', { id });
  }
}

// Step 4: Gradually move more endpoints
// Step 5: Eventually retire monolith code for this domain
```

**4. Service Boundaries Checklist:**

```typescript
// Evaluate each proposed service boundary

interface ServiceBoundaryEvaluation {
  serviceName: string;
  
  // Should be YES for good boundary
  hasIndependentBusinessCapability: boolean;  // Own business logic
  hasClearDataOwnership: boolean;            // Own database/tables
  hasMinimalDependencies: boolean;           // Few calls to other services
  canBeDeployedIndependently: boolean;       // No tight coupling
  hasDistinctScalingNeeds: boolean;          // Different resource needs
  ownedByOneTeam: boolean;                   // Single team ownership
  
  // Should be LOW
  crossServiceTransactions: number;          // Avoid distributed transactions
  sharedDatabaseTables: number;              // Each service own data
  synchronousCallsRequired: number;          // Prefer async
}

// Example evaluation
const paymentServiceEvaluation: ServiceBoundaryEvaluation = {
  serviceName: 'PaymentService',
  hasIndependentBusinessCapability: true,    // ✅ Payment processing logic
  hasClearDataOwnership: true,               // ✅ Owns payments, refunds tables
  hasMinimalDependencies: true,              // ✅ Only calls Stripe, publishes events
  canBeDeployedIndependently: true,          // ✅ No deployment dependencies
  hasDistinctScalingNeeds: true,             // ✅ Needs high availability
  ownedByOneTeam: true,                      // ✅ Payments team
  crossServiceTransactions: 0,               // ✅ No distributed transactions
  sharedDatabaseTables: 0,                   // ✅ Own database
  synchronousCallsRequired: 1,               // ✅ Only order service calls it
};
```

**5. Communication After Split:**

```typescript
// Use async messaging for loose coupling

// Order Service (publisher)
@Injectable()
export class OrdersService {
  constructor(
    @Inject('RABBITMQ_CLIENT') private rabbitClient: ClientProxy,
  ) {}

  async createOrder(dto: CreateOrderDto) {
    const order = await this.orderRepo.save(dto);

    // Publish event (don't wait for response)
    this.rabbitClient.emit('order.created', {
      orderId: order.id,
      userId: order.userId,
      items: order.items,
      total: order.total,
    });

    return order;
  }
}

// Inventory Service (subscriber)
@Controller()
export class InventoryEventsController {
  constructor(private inventoryService: InventoryService) {}

  @EventPattern('order.created')
  async handleOrderCreated(data: any) {
    // Reserve inventory asynchronously
    await this.inventoryService.reserveStock(data.items);
  }
}

// Payment Service (subscriber)
@Controller()
export class PaymentEventsController {
  constructor(private paymentService: PaymentService) {}

  @EventPattern('order.created')
  async handleOrderCreated(data: any) {
    // Process payment asynchronously
    await this.paymentService.processPayment(data);
  }
}
```

**6. Database Per Service Pattern:**

```typescript
// Each service has its own database

// User Service - PostgreSQL
const userServiceDb = {
  type: 'postgres',
  host: 'user-service-db',
  database: 'users',
  entities: [User, Role],
};

// Product Service - PostgreSQL with full-text search
const productServiceDb = {
  type: 'postgres',
  host: 'product-service-db',
  database: 'products',
  entities: [Product, Category],
};

// Order Service - PostgreSQL
const orderServiceDb = {
  type: 'postgres',
  host: 'order-service-db',
  database: 'orders',
  entities: [Order, OrderItem],
};

// Analytics Service - MongoDB (different database type)
const analyticsServiceDb = {
  type: 'mongodb',
  url: 'mongodb://analytics-db:27017/analytics',
};

// If need data from another service, use API call or event
@Injectable()
export class OrdersService {
  constructor(
    @Inject('USER_SERVICE') private userClient: ClientProxy,
  ) {}

  async getOrderWithUser(orderId: string) {
    const order = await this.orderRepo.findOne({ where: { id: orderId } });
    
    // Fetch user data from User Service API
    const user = await firstValueFrom(
      this.userClient.send('get_user', { userId: order.userId }),
    );

    return { ...order, user };
  }
}
```

**7. Migration Strategy:**

```typescript
// Phase 1: Prepare (Monolith)
// - Add feature flags
// - Identify service boundaries
// - Set up monitoring
// - Create message bus infrastructure

// Phase 2: Extract First Service (Low Risk)
// - Start with least coupled service (e.g., Notifications)
// - Use strangler fig pattern
// - Run both monolith and service in parallel
// - Compare responses for validation

// Phase 3: Extract Core Services
// - Move high-value, well-bounded services
// - Implement event-driven patterns
// - Set up API gateway
// - Implement distributed tracing

// Phase 4: Decompose Remaining
// - Extract remaining domains
// - Retire monolith code progressively
// - Consolidate infrastructure

// Feature flag for gradual rollout
@Injectable()
export class OrdersService {
  async createOrder(dto: CreateOrderDto) {
    const useNewPaymentService = await this.featureFlags.isEnabled(
      'new_payment_service',
      dto.userId,
    );

    if (useNewPaymentService) {
      // Call new Payment microservice
      return this.paymentClient.send('create_payment', dto);
    } else {
      // Use old monolith code
      return this.legacyPaymentService.createPayment(dto);
    }
  }
}
```

**Key Takeaway:** Split monolith when: team >10 developers, clear bounded contexts exist, independent scaling needed, different release cycles required. Use Domain-Driven Design to identify service boundaries. Each service: independent deployment, own database, single responsibility, owned by one team. Migrate gradually with strangler fig pattern. Use async messaging (events) for loose coupling. Start with least coupled, high-value services. Don't split if: small team, unclear boundaries, low traffic, lack of distributed systems expertise. Evaluate boundaries: independent business capability, clear data ownership, minimal dependencies.

</details>

<details>
<summary><strong>27. How do you handle inter-service communication?</strong></summary>

**Answer:**

Handle inter-service communication with sync HTTP, async messaging, or gRPC:

**1. NestJS Microservices with TCP:**

```typescript
// Service 1: Order Service (client)
import { ClientsModule, Transport } from '@nestjs/microservices';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'PAYMENT_SERVICE',
        transport: Transport.TCP,
        options: {
          host: 'payment-service',
          port: 3001,
        },
      },
    ]),
  ],
})
export class OrderServiceModule {}

@Injectable()
export class OrdersService {
  constructor(
    @Inject('PAYMENT_SERVICE') private paymentClient: ClientProxy,
  ) {}

  async createOrder(dto: CreateOrderDto) {
    const order = await this.orderRepo.save(dto);

    // Synchronous request-response
    const payment = await firstValueFrom(
      this.paymentClient.send('process_payment', {
        orderId: order.id,
        amount: order.total,
      }),
    );

    order.paymentId = payment.id;
    await this.orderRepo.save(order);

    return order;
  }
}

// Service 2: Payment Service (server)
import { Controller } from '@nestjs/common';
import { MessagePattern } from '@nestjs/microservices';

@Controller()
export class PaymentController {
  constructor(private paymentService: PaymentService) {}

  @MessagePattern('process_payment')
  async processPayment(data: { orderId: string; amount: number }) {
    return this.paymentService.processPayment(data);
  }

  @MessagePattern('get_payment')
  async getPayment(data: { id: string }) {
    return this.paymentService.getPayment(data.id);
  }
}

// main.ts - Payment Service
async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    PaymentServiceModule,
    {
      transport: Transport.TCP,
      options: { host: '0.0.0.0', port: 3001 },
    },
  );
  await app.listen();
}
```

**2. RabbitMQ (Async Event-Driven):**

```typescript
// Shared events (common package)
export const EVENTS = {
  ORDER_CREATED: 'order.created',
  ORDER_CANCELLED: 'order.cancelled',
  PAYMENT_COMPLETED: 'payment.completed',
  PAYMENT_FAILED: 'payment.failed',
  INVENTORY_RESERVED: 'inventory.reserved',
  INVENTORY_RELEASED: 'inventory.released',
};

// Order Service (Publisher)
@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'RABBITMQ_CLIENT',
        transport: Transport.RMQ,
        options: {
          urls: [process.env.RABBITMQ_URL],
          queue: 'orders_queue',
          queueOptions: { durable: true },
        },
      },
    ]),
  ],
})
export class OrderServiceModule {}

@Injectable()
export class OrdersService {
  constructor(
    @Inject('RABBITMQ_CLIENT') private rabbitClient: ClientProxy,
  ) {}

  async createOrder(dto: CreateOrderDto) {
    const order = await this.orderRepo.save({ ...dto, status: 'pending' });

    // Emit event (fire-and-forget, non-blocking)
    this.rabbitClient.emit(EVENTS.ORDER_CREATED, {
      orderId: order.id,
      userId: order.userId,
      items: order.items,
      total: order.total,
      timestamp: new Date(),
    });

    return order;
  }
}

// Inventory Service (Subscriber)
async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    InventoryServiceModule,
    {
      transport: Transport.RMQ,
      options: {
        urls: [process.env.RABBITMQ_URL],
        queue: 'inventory_queue',
        queueOptions: { durable: true },
        prefetchCount: 10, // Process 10 messages at a time
      },
    },
  );
  await app.listen();
}

@Controller()
export class InventoryEventsController {
  constructor(private inventoryService: InventoryService) {}

  @EventPattern(EVENTS.ORDER_CREATED)
  async handleOrderCreated(data: any) {
    try {
      await this.inventoryService.reserveStock(data.items);
      
      // Publish success event
      this.rabbitClient.emit(EVENTS.INVENTORY_RESERVED, {
        orderId: data.orderId,
        timestamp: new Date(),
      });
    } catch (error) {
      // Publish failure event
      this.rabbitClient.emit('inventory.reservation.failed', {
        orderId: data.orderId,
        error: error.message,
      });
    }
  }

  @EventPattern(EVENTS.ORDER_CANCELLED)
  async handleOrderCancelled(data: any) {
    await this.inventoryService.releaseStock(data.orderId);
  }
}

// Payment Service (Subscriber)
@Controller()
export class PaymentEventsController {
  @EventPattern(EVENTS.ORDER_CREATED)
  async handleOrderCreated(data: any) {
    const payment = await this.paymentService.processPayment(data);

    if (payment.status === 'succeeded') {
      this.rabbitClient.emit(EVENTS.PAYMENT_COMPLETED, {
        orderId: data.orderId,
        paymentId: payment.id,
      });
    } else {
      this.rabbitClient.emit(EVENTS.PAYMENT_FAILED, {
        orderId: data.orderId,
        reason: payment.failureReason,
      });
    }
  }
}
```

**3. gRPC (High Performance):**

```protobuf
// protos/payment.proto
syntax = "proto3";

package payment;

service PaymentService {
  rpc ProcessPayment (PaymentRequest) returns (PaymentResponse);
  rpc GetPayment (GetPaymentRequest) returns (Payment);
  rpc RefundPayment (RefundRequest) returns (RefundResponse);
}

message PaymentRequest {
  string order_id = 1;
  double amount = 2;
  string currency = 3;
  string payment_method = 4;
}

message PaymentResponse {
  string payment_id = 1;
  string status = 2;
  string message = 3;
}

message Payment {
  string id = 1;
  string order_id = 2;
  double amount = 3;
  string status = 4;
}

message GetPaymentRequest {
  string id = 1;
}

message RefundRequest {
  string payment_id = 1;
  double amount = 2;
}

message RefundResponse {
  string refund_id = 1;
  string status = 2;
}
```

```typescript
// Payment Service (gRPC Server)
import { Controller } from '@nestjs/common';
import { GrpcMethod } from '@nestjs/microservices';

@Controller()
export class PaymentGrpcController {
  constructor(private paymentService: PaymentService) {}

  @GrpcMethod('PaymentService', 'ProcessPayment')
  async processPayment(data: any) {
    const payment = await this.paymentService.processPayment(data);
    return {
      paymentId: payment.id,
      status: payment.status,
      message: 'Payment processed successfully',
    };
  }

  @GrpcMethod('PaymentService', 'GetPayment')
  async getPayment(data: { id: string }) {
    return this.paymentService.getPayment(data.id);
  }
}

// main.ts
async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    PaymentServiceModule,
    {
      transport: Transport.GRPC,
      options: {
        package: 'payment',
        protoPath: join(__dirname, './protos/payment.proto'),
        url: '0.0.0.0:5001',
      },
    },
  );
  await app.listen();
}

// Order Service (gRPC Client)
@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'PAYMENT_PACKAGE',
        transport: Transport.GRPC,
        options: {
          package: 'payment',
          protoPath: join(__dirname, './protos/payment.proto'),
          url: 'payment-service:5001',
        },
      },
    ]),
  ],
})
export class OrderServiceModule {}

@Injectable()
export class OrdersService {
  constructor(
    @Inject('PAYMENT_PACKAGE') private paymentClient: ClientGrpc,
  ) {}

  private paymentService: any;

  onModuleInit() {
    this.paymentService = this.paymentClient.getService('PaymentService');
  }

  async createOrder(dto: CreateOrderDto) {
    const order = await this.orderRepo.save(dto);

    // Call gRPC service
    const payment = await firstValueFrom(
      this.paymentService.ProcessPayment({
        orderId: order.id,
        amount: order.total,
        currency: 'USD',
        paymentMethod: dto.paymentMethod,
      }),
    );

    order.paymentId = payment.paymentId;
    await this.orderRepo.save(order);

    return order;
  }
}
```

**4. REST API Communication (HTTP):**

```typescript
// Using HttpService for REST calls between services
import { HttpService } from '@nestjs/axios';
import { firstValueFrom } from 'rxjs';

@Injectable()
export class OrdersService {
  constructor(private httpService: HttpService) {}

  async createOrder(dto: CreateOrderDto) {
    const order = await this.orderRepo.save(dto);

    // Call Payment Service REST API
    try {
      const response = await firstValueFrom(
        this.httpService.post('http://payment-service:3001/api/payments', {
          orderId: order.id,
          amount: order.total,
        }),
      );

      order.paymentId = response.data.id;
      await this.orderRepo.save(order);
    } catch (error) {
      // Handle payment service failure
      order.status = 'payment_failed';
      await this.orderRepo.save(order);
      throw new BadRequestException('Payment failed');
    }

    return order;
  }

  async getOrderWithUserDetails(orderId: string) {
    const order = await this.orderRepo.findOne({ where: { id: orderId } });

    // Parallel API calls to multiple services
    const [user, inventory] = await Promise.all([
      firstValueFrom(
        this.httpService.get(`http://user-service:3000/api/users/${order.userId}`),
      ),
      firstValueFrom(
        this.httpService.get(`http://inventory-service:3002/api/inventory/check`, {
          params: { orderIds: orderId },
        }),
      ),
    ]);

    return {
      ...order,
      user: user.data,
      inventory: inventory.data,
    };
  }
}
```

**5. Service Mesh (Resilience Patterns):**

```typescript
// Circuit breaker with axios-retry
import axiosRetry from 'axios-retry';

@Injectable()
export class ResilientHttpService {
  private readonly httpService: HttpService;

  constructor() {
    this.httpService = new HttpService();
    
    // Configure retries
    axiosRetry(this.httpService.axiosRef, {
      retries: 3,
      retryDelay: axiosRetry.exponentialDelay,
      retryCondition: (error) => {
        return axiosRetry.isNetworkOrIdempotentRequestError(error) ||
               error.response?.status === 503;
      },
    });
  }

  async callWithCircuitBreaker(url: string, data: any) {
    const timeout = 5000; // 5 seconds
    
    try {
      const response = await firstValueFrom(
        this.httpService.post(url, data, { timeout }),
      );
      return response.data;
    } catch (error) {
      // Fallback logic
      console.error(`Service call failed: ${url}`, error.message);
      throw new ServiceUnavailableException('External service unavailable');
    }
  }
}
```

**6. Kafka (Event Streaming):**

```typescript
// High-throughput event streaming
@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'KAFKA_CLIENT',
        transport: Transport.KAFKA,
        options: {
          client: {
            clientId: 'order-service',
            brokers: ['kafka:9092'],
          },
          consumer: {
            groupId: 'order-consumer-group',
          },
        },
      },
    ]),
  ],
})
export class OrderServiceModule {}

@Injectable()
export class OrdersService {
  constructor(@Inject('KAFKA_CLIENT') private kafkaClient: ClientKafka) {}

  async onModuleInit() {
    await this.kafkaClient.connect();
  }

  async createOrder(dto: CreateOrderDto) {
    const order = await this.orderRepo.save(dto);

    // Publish to Kafka topic
    this.kafkaClient.emit('order.events', {
      key: order.id,
      value: JSON.stringify({
        type: 'ORDER_CREATED',
        orderId: order.id,
        userId: order.userId,
        total: order.total,
        timestamp: new Date().toISOString(),
      }),
    });

    return order;
  }
}

// Consumer
@Controller()
export class InventoryKafkaController {
  @EventPattern('order.events')
  async handleOrderEvent(@Payload() message: any) {
    const event = JSON.parse(message.value);
    
    if (event.type === 'ORDER_CREATED') {
      await this.inventoryService.reserveStock(event);
    }
  }
}
```

**Key Takeaway:** Inter-service communication patterns: **Sync (TCP/gRPC/HTTP)** for immediate response needed, **Async (RabbitMQ/Kafka)** for loose coupling and resilience. NestJS supports Transport.TCP (request-response), Transport.RMQ (events), Transport.GRPC (high performance), HTTP (REST). Use async messaging for workflows (order → inventory → payment). Implement retries, circuit breakers, timeouts. gRPC faster than REST (protobuf binary). RabbitMQ for task queues with durability. Kafka for event streaming with replay. Choose based on: latency requirements, coupling tolerance, data volume, reliability needs.

</details>

<details>
<summary><strong>28. How do you manage distributed transactions?</strong></summary>

**Answer:**

Manage distributed transactions with saga pattern, eventual consistency, and compensating actions:

**1. Two-Phase Commit (Avoid in Microservices):**

```typescript
/*
2PC (Two-Phase Commit) - NOT RECOMMENDED for microservices

Phase 1: Prepare/Vote
- Coordinator asks all services: "Can you commit?"
- Each service locks resources and votes YES/NO

Phase 2: Commit/Abort
- If all YES: Coordinator tells all to COMMIT
- If any NO: Coordinator tells all to ROLLBACK

PROBLEMS:
- Blocking protocol (resources locked during voting)
- Single point of failure (coordinator)
- Not suitable for microservices (high latency, low availability)
- Services must support XA transactions
*/
```

**2. Saga Pattern - Choreography:**

```typescript
// Each service listens to events and publishes new events
// No central coordinator

// Order Service
@Injectable()
export class OrdersService {
  constructor(
    @Inject('RABBITMQ_CLIENT') private eventBus: ClientProxy,
  ) {}

  async createOrder(dto: CreateOrderDto) {
    // Step 1: Create order in pending state
    const order = await this.orderRepo.save({
      ...dto,
      status: 'PENDING',
    });

    // Publish event to start saga
    this.eventBus.emit('order.created', {
      orderId: order.id,
      userId: order.userId,
      items: order.items,
      total: order.total,
    });

    return order;
  }

  // Listen for success/failure from other services
  @EventPattern('payment.completed')
  async handlePaymentCompleted(data: { orderId: string; paymentId: string }) {
    await this.orderRepo.update(data.orderId, {
      status: 'PAID',
      paymentId: data.paymentId,
    });
  }

  @EventPattern('inventory.reserved')
  async handleInventoryReserved(data: { orderId: string }) {
    await this.orderRepo.update(data.orderId, {
      status: 'CONFIRMED',
    });
  }

  // Compensating action for failure
  @EventPattern('payment.failed')
  async handlePaymentFailed(data: { orderId: string; reason: string }) {
    await this.orderRepo.update(data.orderId, {
      status: 'FAILED',
      failureReason: data.reason,
    });

    // Trigger compensations
    this.eventBus.emit('order.cancelled', { orderId: data.orderId });
  }
}

// Inventory Service
@Controller()
export class InventoryEventsController {
  constructor(
    private inventoryService: InventoryService,
    @Inject('RABBITMQ_CLIENT') private eventBus: ClientProxy,
  ) {}

  @EventPattern('order.created')
  async handleOrderCreated(data: any) {
    try {
      // Step 2: Reserve inventory
      await this.inventoryService.reserveStock(data.items);

      this.eventBus.emit('inventory.reserved', {
        orderId: data.orderId,
      });
    } catch (error) {
      // Insufficient stock - trigger rollback
      this.eventBus.emit('inventory.reservation.failed', {
        orderId: data.orderId,
        reason: error.message,
      });
    }
  }

  // Compensating transaction
  @EventPattern('order.cancelled')
  async handleOrderCancelled(data: { orderId: string }) {
    await this.inventoryService.releaseStock(data.orderId);
  }
}

// Payment Service
@Controller()
export class PaymentEventsController {
  constructor(
    private paymentService: PaymentService,
    @Inject('RABBITMQ_CLIENT') private eventBus: ClientProxy,
  ) {}

  @EventPattern('inventory.reserved')
  async handleInventoryReserved(data: any) {
    try {
      // Step 3: Process payment
      const payment = await this.paymentService.processPayment(data);

      this.eventBus.emit('payment.completed', {
        orderId: data.orderId,
        paymentId: payment.id,
      });
    } catch (error) {
      this.eventBus.emit('payment.failed', {
        orderId: data.orderId,
        reason: error.message,
      });
    }
  }

  // Compensating transaction
  @EventPattern('order.cancelled')
  async handleOrderCancelled(data: { orderId: string }) {
    await this.paymentService.refundPayment(data.orderId);
  }
}
```

**3. Saga Pattern - Orchestration:**

```typescript
// Central orchestrator manages the saga

@Entity('saga_instances')
export class SagaInstance {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  sagaType: string; // 'order_fulfillment'

  @Column()
  status: string; // 'STARTED', 'COMPLETED', 'FAILED', 'COMPENSATING'

  @Column('jsonb')
  data: any; // Order data

  @Column('simple-array')
  completedSteps: string[]; // ['reserve_inventory', 'process_payment']

  @Column({ nullable: true })
  currentStep: string;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}

@Injectable()
export class OrderFulfillmentSaga {
  private steps = [
    { name: 'reserve_inventory', service: 'INVENTORY_SERVICE', compensate: 'release_inventory' },
    { name: 'process_payment', service: 'PAYMENT_SERVICE', compensate: 'refund_payment' },
    { name: 'arrange_shipping', service: 'SHIPPING_SERVICE', compensate: 'cancel_shipping' },
  ];

  constructor(
    @InjectRepository(SagaInstance)
    private sagaRepo: Repository<SagaInstance>,
    @Inject('INVENTORY_SERVICE') private inventoryClient: ClientProxy,
    @Inject('PAYMENT_SERVICE') private paymentClient: ClientProxy,
    @Inject('SHIPPING_SERVICE') private shippingClient: ClientProxy,
  ) {}

  async startSaga(orderData: any): Promise<SagaInstance> {
    // Create saga instance
    const saga = await this.sagaRepo.save({
      sagaType: 'order_fulfillment',
      status: 'STARTED',
      data: orderData,
      completedSteps: [],
      currentStep: this.steps[0].name,
    });

    // Execute steps sequentially
    await this.executeSaga(saga.id);

    return saga;
  }

  private async executeSaga(sagaId: string) {
    const saga = await this.sagaRepo.findOne({ where: { id: sagaId } });

    for (const step of this.steps) {
      // Skip completed steps
      if (saga.completedSteps.includes(step.name)) {
        continue;
      }

      saga.currentStep = step.name;
      await this.sagaRepo.save(saga);

      try {
        // Execute step
        await this.executeStep(step, saga.data);

        // Mark step as completed
        saga.completedSteps.push(step.name);
        await this.sagaRepo.save(saga);
      } catch (error) {
        // Step failed - start compensation
        saga.status = 'COMPENSATING';
        await this.sagaRepo.save(saga);

        await this.compensate(saga);
        
        saga.status = 'FAILED';
        await this.sagaRepo.save(saga);
        
        throw error;
      }
    }

    // All steps completed
    saga.status = 'COMPLETED';
    saga.currentStep = null;
    await this.sagaRepo.save(saga);
  }

  private async executeStep(step: any, data: any): Promise<any> {
    const client = this.getClient(step.service);
    
    return firstValueFrom(
      client.send(step.name, data).pipe(
        timeout(10000), // 10 second timeout
      ),
    );
  }

  private async compensate(saga: SagaInstance) {
    // Execute compensating transactions in reverse order
    const completedSteps = [...saga.completedSteps].reverse();

    for (const stepName of completedSteps) {
      const step = this.steps.find(s => s.name === stepName);
      
      if (step && step.compensate) {
        try {
          const client = this.getClient(step.service);
          await firstValueFrom(
            client.send(step.compensate, {
              orderId: saga.data.orderId,
              reason: 'saga_rollback',
            }),
          );
        } catch (error) {
          // Log compensation failure (may need manual intervention)
          console.error(`Compensation failed for ${step.name}:`, error);
        }
      }
    }
  }

  private getClient(serviceName: string): ClientProxy {
    switch (serviceName) {
      case 'INVENTORY_SERVICE': return this.inventoryClient;
      case 'PAYMENT_SERVICE': return this.paymentClient;
      case 'SHIPPING_SERVICE': return this.shippingClient;
      default: throw new Error(`Unknown service: ${serviceName}`);
    }
  }
}

// Usage
@Injectable()
export class OrdersService {
  constructor(private saga: OrderFulfillmentSaga) {}

  async createOrder(dto: CreateOrderDto) {
    const order = await this.orderRepo.save({ ...dto, status: 'PENDING' });

    try {
      await this.saga.startSaga({
        orderId: order.id,
        userId: order.userId,
        items: order.items,
        total: order.total,
      });

      order.status = 'CONFIRMED';
    } catch (error) {
      order.status = 'FAILED';
    }

    return this.orderRepo.save(order);
  }
}
```

**4. Outbox Pattern (Reliable Event Publishing):**

```typescript
// Ensure events are published even if message broker fails

@Entity('outbox_events')
export class OutboxEvent {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  aggregateType: string; // 'Order'

  @Column()
  aggregateId: string; // Order ID

  @Column()
  eventType: string; // 'order.created'

  @Column('jsonb')
  payload: any;

  @Column({ default: false })
  published: boolean;

  @CreateDateColumn()
  createdAt: Date;
}

@Injectable()
export class OrdersService {
  constructor(
    @InjectRepository(Order) private orderRepo: Repository<Order>,
    @InjectRepository(OutboxEvent) private outboxRepo: Repository<OutboxEvent>,
    private dataSource: DataSource,
  ) {}

  async createOrder(dto: CreateOrderDto) {
    // Use transaction to ensure atomicity
    return this.dataSource.transaction(async (manager) => {
      // 1. Save order
      const order = await manager.save(Order, {
        ...dto,
        status: 'PENDING',
      });

      // 2. Save event to outbox table (same transaction)
      await manager.save(OutboxEvent, {
        aggregateType: 'Order',
        aggregateId: order.id,
        eventType: 'order.created',
        payload: {
          orderId: order.id,
          userId: order.userId,
          items: order.items,
          total: order.total,
        },
        published: false,
      });

      return order;
    });
  }
}

// Background job publishes events
@Injectable()
export class OutboxPublisher {
  constructor(
    @InjectRepository(OutboxEvent) private outboxRepo: Repository<OutboxEvent>,
    @Inject('RABBITMQ_CLIENT') private eventBus: ClientProxy,
  ) {}

  @Cron(CronExpression.EVERY_5_SECONDS)
  async publishPendingEvents() {
    const events = await this.outboxRepo.find({
      where: { published: false },
      order: { createdAt: 'ASC' },
      take: 100,
    });

    for (const event of events) {
      try {
        // Publish to message broker
        await firstValueFrom(
          this.eventBus.emit(event.eventType, event.payload),
        );

        // Mark as published
        event.published = true;
        await this.outboxRepo.save(event);
      } catch (error) {
        // Retry on next run
        console.error('Failed to publish event:', error);
      }
    }
  }
}
```

**5. Idempotency for Retries:**

```typescript
// Handle duplicate messages

@Entity('processed_events')
export class ProcessedEvent {
  @PrimaryColumn()
  eventId: string; // Message ID

  @Column()
  eventType: string;

  @CreateDateColumn()
  processedAt: Date;
}

@Injectable()
export class IdempotentEventHandler {
  constructor(
    @InjectRepository(ProcessedEvent)
    private processedRepo: Repository<ProcessedEvent>,
  ) {}

  async handleEvent(eventId: string, eventType: string, handler: () => Promise<void>) {
    // Check if already processed
    const existing = await this.processedRepo.findOne({
      where: { eventId },
    });

    if (existing) {
      console.log(`Event ${eventId} already processed, skipping`);
      return;
    }

    // Process event
    await handler();

    // Record as processed
    await this.processedRepo.save({
      eventId,
      eventType,
    });
  }
}

// Usage
@Controller()
export class InventoryEventsController {
  constructor(private idempotency: IdempotentEventHandler) {}

  @EventPattern('order.created')
  async handleOrderCreated(@Payload() message: any, @Ctx() context: any) {
    const eventId = context.getMessage().properties.messageId;

    await this.idempotency.handleEvent(
      eventId,
      'order.created',
      async () => {
        await this.inventoryService.reserveStock(message.items);
      },
    );
  }
}
```

**Key Takeaway:** Distributed transactions use saga pattern (not 2PC). **Choreography**: services react to events, no coordinator, loose coupling, harder to debug. **Orchestration**: central saga manager coordinates steps, easier to track, single point of failure. Implement compensating transactions for rollback (release_inventory, refund_payment). Use outbox pattern for reliable event publishing (save event in same DB transaction). Ensure idempotency (deduplicate messages with processed_events table). Accept eventual consistency. Track saga state (STARTED/COMPENSATING/COMPLETED/FAILED). Saga better than 2PC for microservices: non-blocking, higher availability, long-running workflows.

</details>

<details>
<summary><strong>29. What is saga pattern for distributed transactions?</strong></summary>

**Answer:**

Saga pattern breaks distributed transaction into local transactions with compensating actions:

**1. Saga Fundamentals:**

```typescript
/*
SAGA PATTERN CONCEPTS:

1. Sequence of Local Transactions
   - Each service performs its own local transaction
   - Publishes event on success
   - Next service reacts to event

2. Compensating Transactions
   - Reverse actions if saga fails
   - Execute in reverse order
   - Example: refund_payment, release_inventory

3. Eventual Consistency
   - System eventually becomes consistent
   - Intermediate states may be inconsistent
   - Need to handle partial states

4. Two Approaches:
   - Choreography: Decentralized, event-driven
   - Orchestration: Centralized coordinator

WHEN TO USE SAGA:
- Long-running business processes
- Multiple services involved
- No need for immediate consistency
- Can define compensating actions
*/
```

**2. E-Commerce Order Saga Example:**

```typescript
// Saga Definition
interface SagaStep {
  name: string;
  action: string; // Message pattern to send
  compensation: string; // Rollback action
  service: string;
}

const ORDER_FULFILLMENT_SAGA: SagaStep[] = [
  {
    name: 'Validate Order',
    action: 'validate_order',
    compensation: null, // No rollback needed
    service: 'ORDER_SERVICE',
  },
  {
    name: 'Reserve Inventory',
    action: 'reserve_inventory',
    compensation: 'release_inventory',
    service: 'INVENTORY_SERVICE',
  },
  {
    name: 'Authorize Payment',
    action: 'authorize_payment',
    compensation: 'cancel_payment_authorization',
    service: 'PAYMENT_SERVICE',
  },
  {
    name: 'Capture Payment',
    action: 'capture_payment',
    compensation: 'refund_payment',
    service: 'PAYMENT_SERVICE',
  },
  {
    name: 'Create Shipment',
    action: 'create_shipment',
    compensation: 'cancel_shipment',
    service: 'SHIPPING_SERVICE',
  },
  {
    name: 'Send Confirmation',
    action: 'send_order_confirmation',
    compensation: null,
    service: 'NOTIFICATION_SERVICE',
  },
];
```

**3. Saga Orchestrator Implementation:**

```typescript
// saga/saga-orchestrator.service.ts
@Injectable()
export class SagaOrchestrator {
  constructor(
    @InjectRepository(SagaExecution)
    private sagaExecutionRepo: Repository<SagaExecution>,
    private readonly moduleRef: ModuleRef,
  ) {}

  async execute(
    sagaDefinition: SagaStep[],
    initialData: any,
  ): Promise<SagaExecutionResult> {
    // Create saga execution record
    const execution = await this.sagaExecutionRepo.save({
      sagaType: sagaDefinition[0].name,
      status: SagaStatus.STARTED,
      data: initialData,
      completedSteps: [],
      compensatedSteps: [],
    });

    try {
      // Execute forward steps
      for (const step of sagaDefinition) {
        await this.executeStep(execution, step);
      }

      // Mark as completed
      execution.status = SagaStatus.COMPLETED;
      await this.sagaExecutionRepo.save(execution);

      return { success: true, executionId: execution.id };
    } catch (error) {
      // Compensate in reverse order
      execution.status = SagaStatus.COMPENSATING;
      await this.sagaExecutionRepo.save(execution);

      await this.compensate(execution, sagaDefinition);

      execution.status = SagaStatus.FAILED;
      execution.failureReason = error.message;
      await this.sagaExecutionRepo.save(execution);

      return { success: false, executionId: execution.id, error: error.message };
    }
  }

  private async executeStep(execution: SagaExecution, step: SagaStep) {
    try {
      const client = this.getServiceClient(step.service);
      
      // Execute step with timeout
      const result = await firstValueFrom(
        client.send(step.action, execution.data).pipe(
          timeout(15000), // 15 second timeout per step
        ),
      );

      // Update execution data with step result
      execution.data = { ...execution.data, [step.name]: result };
      execution.completedSteps.push(step.name);
      execution.currentStep = null;
      await this.sagaExecutionRepo.save(execution);

      return result;
    } catch (error) {
      // Step failed
      throw new Error(`Step ${step.name} failed: ${error.message}`);
    }
  }

  private async compensate(execution: SagaExecution, sagaDefinition: SagaStep[]) {
    // Reverse completed steps
    const stepsToCompensate = [...execution.completedSteps].reverse();

    for (const stepName of stepsToCompensate) {
      const step = sagaDefinition.find(s => s.name === stepName);
      
      if (!step || !step.compensation) {
        continue; // Skip if no compensation defined
      }

      try {
        const client = this.getServiceClient(step.service);
        
        await firstValueFrom(
          client.send(step.compensation, {
            executionId: execution.id,
            ...execution.data,
          }).pipe(
            timeout(15000),
          ),
        );

        execution.compensatedSteps.push(stepName);
        await this.sagaExecutionRepo.save(execution);
      } catch (error) {
        // Log compensation failure
        console.error(`Compensation failed for ${stepName}:`, error);
        
        // Could implement manual intervention queue here
        await this.notifyCompensationFailure(execution, stepName, error);
      }
    }
  }

  private getServiceClient(serviceName: string): ClientProxy {
    return this.moduleRef.get(`${serviceName}_CLIENT`, { strict: false });
  }

  private async notifyCompensationFailure(
    execution: SagaExecution,
    stepName: string,
    error: any,
  ) {
    // Send alert to ops team for manual intervention
    console.error('CRITICAL: Manual compensation required', {
      executionId: execution.id,
      step: stepName,
      error: error.message,
    });
  }
}

// Entity to track saga executions
@Entity('saga_executions')
export class SagaExecution {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  sagaType: string;

  @Column({
    type: 'enum',
    enum: SagaStatus,
  })
  status: SagaStatus;

  @Column('jsonb')
  data: any;

  @Column('simple-array')
  completedSteps: string[];

  @Column('simple-array', { default: '' })
  compensatedSteps: string[];

  @Column({ nullable: true })
  currentStep: string;

  @Column({ nullable: true })
  failureReason: string;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}

enum SagaStatus {
  STARTED = 'STARTED',
  COMPLETED = 'COMPLETED',
  COMPENSATING = 'COMPENSATING',
  FAILED = 'FAILED',
}
```

**4. Service Implementations:**

```typescript
// Inventory Service - Implements forward and compensating actions
@Controller()
export class InventoryController {
  constructor(private inventoryService: InventoryService) {}

  @MessagePattern('reserve_inventory')
  async reserveInventory(data: any) {
    const { items } = data;
    
    // Check stock availability
    for (const item of items) {
      const available = await this.inventoryService.checkStock(item.productId, item.quantity);
      if (!available) {
        throw new Error(`Insufficient stock for product ${item.productId}`);
      }
    }

    // Reserve stock (decrement available, increment reserved)
    const reservationId = await this.inventoryService.reserveStock(items);

    return {
      reservationId,
      message: 'Inventory reserved successfully',
    };
  }

  @MessagePattern('release_inventory')
  async releaseInventory(data: any) {
    const { 'Reserve Inventory': reservationData } = data;
    
    if (reservationData?.reservationId) {
      await this.inventoryService.releaseReservation(reservationData.reservationId);
    }

    return { message: 'Inventory released' };
  }
}

// Payment Service - Authorize then Capture pattern
@Controller()
export class PaymentController {
  constructor(private paymentService: PaymentService) {}

  @MessagePattern('authorize_payment')
  async authorizePayment(data: any) {
    const { total, userId, paymentMethod } = data;
    
    // Create payment authorization (hold funds)
    const authorization = await this.paymentService.authorize({
      amount: total,
      userId,
      paymentMethod,
    });

    return {
      authorizationId: authorization.id,
      status: authorization.status,
    };
  }

  @MessagePattern('capture_payment')
  async capturePayment(data: any) {
    const { 'Authorize Payment': authData } = data;
    
    // Capture the authorized payment
    const payment = await this.paymentService.capture(authData.authorizationId);

    return {
      paymentId: payment.id,
      status: 'captured',
    };
  }

  @MessagePattern('cancel_payment_authorization')
  async cancelAuthorization(data: any) {
    const { 'Authorize Payment': authData } = data;
    
    if (authData?.authorizationId) {
      await this.paymentService.cancelAuthorization(authData.authorizationId);
    }

    return { message: 'Authorization cancelled' };
  }

  @MessagePattern('refund_payment')
  async refundPayment(data: any) {
    const { 'Capture Payment': paymentData } = data;
    
    if (paymentData?.paymentId) {
      await this.paymentService.refund(paymentData.paymentId);
    }

    return { message: 'Payment refunded' };
  }
}
```

**5. Usage in Order Service:**

```typescript
@Injectable()
export class OrdersService {
  constructor(
    private sagaOrchestrator: SagaOrchestrator,
    @InjectRepository(Order) private orderRepo: Repository<Order>,
  ) {}

  async createOrder(dto: CreateOrderDto): Promise<Order> {
    // Create order in pending state
    const order = await this.orderRepo.save({
      ...dto,
      status: 'PENDING',
    });

    // Execute saga
    const result = await this.sagaOrchestrator.execute(
      ORDER_FULFILLMENT_SAGA,
      {
        orderId: order.id,
        userId: dto.userId,
        items: dto.items,
        total: dto.total,
        paymentMethod: dto.paymentMethod,
      },
    );

    if (result.success) {
      order.status = 'CONFIRMED';
      order.sagaExecutionId = result.executionId;
    } else {
      order.status = 'FAILED';
      order.failureReason = result.error;
    }

    return this.orderRepo.save(order);
  }

  // Query saga status
  async getOrderSagaStatus(orderId: string) {
    const order = await this.orderRepo.findOne({ where: { id: orderId } });
    
    if (!order.sagaExecutionId) {
      return null;
    }

    return this.sagaOrchestrator.getSagaExecution(order.sagaExecutionId);
  }
}
```

**6. Saga Monitoring Dashboard:**

```typescript
// Admin endpoint to view saga executions
@Controller('admin/sagas')
export class SagaMonitoringController {
  constructor(
    @InjectRepository(SagaExecution)
    private sagaRepo: Repository<SagaExecution>,
  ) {}

  @Get('failed')
  async getFailedSagas() {
    return this.sagaRepo.find({
      where: { status: SagaStatus.FAILED },
      order: { createdAt: 'DESC' },
      take: 100,
    });
  }

  @Get('in-progress')
  async getInProgressSagas() {
    return this.sagaRepo.find({
      where: [
        { status: SagaStatus.STARTED },
        { status: SagaStatus.COMPENSATING },
      ],
      order: { createdAt: 'DESC' },
    });
  }

  @Get(':id')
  async getSagaDetails(@Param('id') id: string) {
    return this.sagaRepo.findOne({ where: { id } });
  }

  @Post(':id/retry')
  async retrySaga(@Param('id') id: string) {
    // Retry failed saga from last successful step
    const execution = await this.sagaRepo.findOne({ where: { id } });
    
    if (execution.status !== SagaStatus.FAILED) {
      throw new BadRequestException('Can only retry failed sagas');
    }

    // Implementation to retry from checkpoint
  }
}
```

**Key Takeaway:** Saga pattern manages distributed transactions through sequence of local transactions with compensating actions. **Orchestration** (centralized coordinator) easier to monitor and debug, tracks state in saga_executions table with completedSteps/compensatedSteps. Steps execute sequentially, on failure compensate in reverse order (release_inventory → cancel_authorization → refund_payment). Each service implements both forward action (reserve_inventory) and compensation (release_inventory). Use authorize-then-capture for payments. Accept eventual consistency. Saga better than 2PC: non-blocking, no resource locking, works with long-running processes. Monitor saga status, implement retry logic, alert on compensation failures requiring manual intervention.

</details>

<details>
<summary><strong>30. How do you implement service discovery?</strong></summary>

**Answer:**

Implement service discovery for dynamic service registration and lookup:

**1. Consul Service Discovery:**

```typescript
// Install: npm install nest-consul consul

// consul.module.ts
import { Module } from '@nestjs/common';
import { ConsulModule } from 'nest-consul';

@Module({
  imports: [
    ConsulModule.forRoot({
      host: process.env.CONSUL_HOST || 'localhost',
      port: process.env.CONSUL_PORT || '8500',
    }),
  ],
})
export class AppConsulModule {}

// Service registration on startup
// main.ts
import { Consul } from 'consul';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  const serviceName = 'order-service';
  const servicePort = 3000;
  const serviceHost = process.env.SERVICE_HOST || 'localhost';
  
  // Register with Consul
  const consul = new Consul({
    host: process.env.CONSUL_HOST,
    port: process.env.CONSUL_PORT,
  });

  await consul.agent.service.register({
    id: `${serviceName}-${process.env.INSTANCE_ID || '1'}`,
    name: serviceName,
    address: serviceHost,
    port: servicePort,
    check: {
      http: `http://${serviceHost}:${servicePort}/health`,
      interval: '10s',
      timeout: '5s',
      deregistercriticalserviceafter: '1m',
    },
    tags: ['api', 'v1', 'orders'],
  });

  await app.listen(servicePort);

  // Deregister on shutdown
  process.on('SIGTERM', async () => {
    await consul.agent.service.deregister(`${serviceName}-${process.env.INSTANCE_ID}`);
    await app.close();
  });
}
bootstrap();
```

**2. Service Discovery Client:**

```typescript
// service-discovery/service-discovery.service.ts
import { Injectable } from '@nestjs/common';
import { Consul } from 'consul';

@Injectable()
export class ServiceDiscoveryService {
  private consul: Consul;
  private serviceCache: Map<string, ServiceInstance[]> = new Map();

  constructor() {
    this.consul = new Consul({
      host: process.env.CONSUL_HOST,
      port: process.env.CONSUL_PORT,
    });

    // Refresh cache periodically
    setInterval(() => this.refreshCache(), 30000);
  }

  async getServiceInstances(serviceName: string): Promise<ServiceInstance[]> {
    // Check cache first
    if (this.serviceCache.has(serviceName)) {
      return this.serviceCache.get(serviceName);
    }

    // Query Consul
    const result = await this.consul.health.service({
      service: serviceName,
      passing: true, // Only healthy instances
    });

    const instances = result.map(entry => ({
      id: entry.Service.ID,
      address: entry.Service.Address,
      port: entry.Service.Port,
      tags: entry.Service.Tags,
    }));

    this.serviceCache.set(serviceName, instances);
    return instances;
  }

  async getServiceUrl(serviceName: string, path: string = ''): Promise<string> {
    const instances = await this.getServiceInstances(serviceName);
    
    if (instances.length === 0) {
      throw new Error(`No instances available for service: ${serviceName}`);
    }

    // Load balancing: Round robin
    const instance = this.selectInstance(instances);
    
    return `http://${instance.address}:${instance.port}${path}`;
  }

  private selectInstance(instances: ServiceInstance[]): ServiceInstance {
    // Simple round-robin (can implement weighted, least connections, etc.)
    const index = Math.floor(Math.random() * instances.length);
    return instances[index];
  }

  private async refreshCache() {
    for (const serviceName of this.serviceCache.keys()) {
      try {
        await this.getServiceInstances(serviceName);
      } catch (error) {
        console.error(`Failed to refresh cache for ${serviceName}:`, error);
      }
    }
  }
}

interface ServiceInstance {
  id: string;
  address: string;
  port: number;
  tags: string[];
}
```

**3. HTTP Client with Service Discovery:**

```typescript
// http-client/discovered-http.service.ts
@Injectable()
export class DiscoveredHttpService {
  constructor(
    private httpService: HttpService,
    private serviceDiscovery: ServiceDiscoveryService,
  ) {}

  async get<T>(serviceName: string, path: string): Promise<T> {
    const url = await this.serviceDiscovery.getServiceUrl(serviceName, path);
    
    const response = await firstValueFrom(
      this.httpService.get<T>(url).pipe(
        timeout(5000),
        retry(2),
      ),
    );

    return response.data;
  }

  async post<T>(serviceName: string, path: string, data: any): Promise<T> {
    const url = await this.serviceDiscovery.getServiceUrl(serviceName, path);
    
    const response = await firstValueFrom(
      this.httpService.post<T>(url, data).pipe(
        timeout(5000),
        retry(2),
      ),
    );

    return response.data;
  }
}

// Usage
@Injectable()
export class OrdersService {
  constructor(private discoveredHttp: DiscoveredHttpService) {}

  async createOrder(dto: CreateOrderDto) {
    const order = await this.orderRepo.save(dto);

    // Call payment service using service discovery
    const payment = await this.discoveredHttp.post(
      'payment-service',
      '/api/payments',
      {
        orderId: order.id,
        amount: order.total,
      },
    );

    order.paymentId = payment.id;
    return this.orderRepo.save(order);
  }
}
```

**4. Kubernetes Service Discovery (DNS-based):**

```yaml
# k8s/payment-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: payment-service
spec:
  selector:
    app: payment
  ports:
  - port: 80
    targetPort: 3000
  type: ClusterIP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: payment
  template:
    metadata:
      labels:
        app: payment
    spec:
      containers:
      - name: payment
        image: payment-service:latest
        ports:
        - containerPort: 3000
        env:
        - name: SERVICE_NAME
          value: "payment-service"
```

```typescript
// In Kubernetes, use service DNS name directly
// No need for Consul when using K8s

@Injectable()
export class OrdersService {
  constructor(private httpService: HttpService) {}

  async createOrder(dto: CreateOrderDto) {
    // K8s DNS: <service-name>.<namespace>.svc.cluster.local
    // Within same namespace: just <service-name>
    const response = await firstValueFrom(
      this.httpService.post(
        'http://payment-service/api/payments', // K8s DNS
        {
          orderId: order.id,
          amount: order.total,
        },
      ),
    );

    return response.data;
  }
}
```

**5. Client-Side Load Balancing:**

```typescript
// load-balancer/client-load-balancer.service.ts
@Injectable()
export class ClientLoadBalancer {
  private instanceIndex: Map<string, number> = new Map();

  constructor(private serviceDiscovery: ServiceDiscoveryService) {}

  async callService<T>(
    serviceName: string,
    method: 'GET' | 'POST',
    path: string,
    data?: any,
  ): Promise<T> {
    const instances = await this.serviceDiscovery.getServiceInstances(serviceName);
    
    if (instances.length === 0) {
      throw new ServiceUnavailableException(`No instances for ${serviceName}`);
    }

    // Try instances until success or all fail
    let lastError: any;
    const maxAttempts = Math.min(instances.length, 3);

    for (let attempt = 0; attempt < maxAttempts; attempt++) {
      const instance = this.selectNextInstance(serviceName, instances);
      const url = `http://${instance.address}:${instance.port}${path}`;

      try {
        const response = await this.makeRequest<T>(method, url, data);
        return response;
      } catch (error) {
        lastError = error;
        console.warn(`Request to ${url} failed, trying next instance`);
      }
    }

    throw new ServiceUnavailableException(
      `All ${serviceName} instances failed: ${lastError.message}`,
    );
  }

  private selectNextInstance(
    serviceName: string,
    instances: ServiceInstance[],
  ): ServiceInstance {
    // Round-robin selection
    const currentIndex = this.instanceIndex.get(serviceName) || 0;
    const nextIndex = (currentIndex + 1) % instances.length;
    
    this.instanceIndex.set(serviceName, nextIndex);
    
    return instances[nextIndex];
  }

  private async makeRequest<T>(
    method: string,
    url: string,
    data?: any,
  ): Promise<T> {
    const axios = require('axios');
    const response = await axios({
      method,
      url,
      data,
      timeout: 5000,
    });
    return response.data;
  }
}
```

**6. Docker Compose with Service Discovery:**

```yaml
# docker-compose.yml
version: '3.8'

services:
  consul:
    image: consul:latest
    ports:
      - "8500:8500"
      - "8600:8600/udp"
    command: agent -server -ui -bootstrap-expect=1 -client=0.0.0.0

  order-service:
    image: order-service:latest
    environment:
      - CONSUL_HOST=consul
      - CONSUL_PORT=8500
      - SERVICE_HOST=order-service
    depends_on:
      - consul
    deploy:
      replicas: 2

  payment-service:
    image: payment-service:latest
    environment:
      - CONSUL_HOST=consul
      - CONSUL_PORT=8500
      - SERVICE_HOST=payment-service
    depends_on:
      - consul
    deploy:
      replicas: 3

  inventory-service:
    image: inventory-service:latest
    environment:
      - CONSUL_HOST=consul
      - CONSUL_PORT=8500
      - SERVICE_HOST=inventory-service
    depends_on:
      - consul
    deploy:
      replicas: 2
```

**Key Takeaway:** Service discovery enables dynamic service lookup. **Consul**: services register on startup with health check URL, clients query Consul for healthy instances, implement client-side load balancing (round-robin). **Kubernetes**: uses DNS-based discovery, services accessible via `<service-name>` DNS, automatic load balancing with ClusterIP. Register service with health check endpoint, deregister on shutdown. Cache service instances with periodic refresh. Implement fallback (try multiple instances). Benefits: dynamic scaling, no hardcoded IPs, automatic failover, health-based routing.

</details>

<details>
<summary><strong>31. How do you handle API gateway in microservices?</strong></summary>

**Answer:**

Implement API Gateway as single entry point for all client requests:

**1. NestJS API Gateway Setup:**

```typescript
// API Gateway Service
// gateway/gateway.module.ts
@Module({
  imports: [
    HttpModule,
    ClientsModule.register([
      {
        name: 'ORDER_SERVICE',
        transport: Transport.TCP,
        options: { host: 'order-service', port: 3001 },
      },
      {
        name: 'PAYMENT_SERVICE',
        transport: Transport.TCP,
        options: { host: 'payment-service', port: 3002 },
      },
      {
        name: 'USER_SERVICE',
        transport: Transport.TCP,
        options: { host: 'user-service', port: 3003 },
      },
    ]),
  ],
  controllers: [GatewayController],
  providers: [AuthGuard, RateLimitGuard],
})
export class GatewayModule {}

// gateway/gateway.controller.ts
@Controller('api')
@UseGuards(AuthGuard, RateLimitGuard)
export class GatewayController {
  constructor(
    @Inject('ORDER_SERVICE') private orderClient: ClientProxy,
    @Inject('PAYMENT_SERVICE') private paymentClient: ClientProxy,
    @Inject('USER_SERVICE') private userClient: ClientProxy,
  ) {}

  // Route to Order Service
  @Post('orders')
  async createOrder(@Body() dto: CreateOrderDto, @Request() req) {
    return firstValueFrom(
      this.orderClient.send('create_order', {
        ...dto,
        userId: req.user.id,
      }),
    );
  }

  @Get('orders/:id')
  async getOrder(@Param('id') id: string) {
    return firstValueFrom(
      this.orderClient.send('get_order', { id }),
    );
  }

  // Route to Payment Service
  @Post('payments')
  async createPayment(@Body() dto: CreatePaymentDto) {
    return firstValueFrom(
      this.paymentClient.send('create_payment', dto),
    );
  }

  // Aggregation: Fetch from multiple services
  @Get('orders/:id/details')
  async getOrderDetails(@Param('id') orderId: string) {
    // Parallel requests to multiple services
    const [order, payment, user] = await Promise.all([
      firstValueFrom(this.orderClient.send('get_order', { id: orderId })),
      firstValueFrom(this.paymentClient.send('get_payment_by_order', { orderId })),
      firstValueFrom(this.userClient.send('get_user', { id: 'userId' })),
    ]);

    return {
      order,
      payment,
      user,
    };
  }
}
```

**2. Authentication & Authorization at Gateway:**

```typescript
// gateway/auth.guard.ts
@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private jwtService: JwtService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = this.extractToken(request);

    if (!token) {
      throw new UnauthorizedException('No token provided');
    }

    try {
      const payload = await this.jwtService.verifyAsync(token);
      request.user = payload;
      return true;
    } catch {
      throw new UnauthorizedException('Invalid token');
    }
  }

  private extractToken(request: any): string | undefined {
    const [type, token] = request.headers.authorization?.split(' ') ?? [];
    return type === 'Bearer' ? token : undefined;
  }
}

// Apply to all routes
// main.ts
async function bootstrap() {
  const app = await NestFactory.create(GatewayModule);
  
  // Global guards
  app.useGlobalGuards(new AuthGuard(app.get(JwtService)));
  
  await app.listen(3000);
}
```

**3. Rate Limiting at Gateway:**

```typescript
// gateway/rate-limit.guard.ts
import { ThrottlerGuard } from '@nestjs/throttler';

@Injectable()
export class RateLimitGuard extends ThrottlerGuard {
  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    
    // Different limits per user tier
    const userTier = request.user?.tier || 'free';
    const limits = {
      free: { ttl: 60, limit: 10 },
      premium: { ttl: 60, limit: 100 },
      enterprise: { ttl: 60, limit: 1000 },
    };

    const { ttl, limit } = limits[userTier];
    
    // Override throttler options
    this.options.ttl = ttl;
    this.options.limit = limit;

    return super.canActivate(context);
  }
}
```

**4. Request/Response Transformation:**

```typescript
// gateway/transform.interceptor.ts
@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => ({
        statusCode: context.switchToHttp().getResponse().statusCode,
        timestamp: new Date().toISOString(),
        data,
      })),
      catchError(err => {
        // Standardize error responses
        return throwError(() => ({
          statusCode: err.status || 500,
          timestamp: new Date().toISOString(),
          message: err.message,
          error: err.name,
        }));
      }),
    );
  }
}

// Apply globally
app.useGlobalInterceptors(new TransformInterceptor());
```

**5. Circuit Breaker at Gateway:**

```typescript
// gateway/circuit-breaker.interceptor.ts
import CircuitBreaker from 'opossum';

@Injectable()
export class CircuitBreakerInterceptor implements NestInterceptor {
  private breakers: Map<string, CircuitBreaker> = new Map();

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const serviceName = this.getServiceName(context);
    let breaker = this.breakers.get(serviceName);

    if (!breaker) {
      breaker = new CircuitBreaker(async () => firstValueFrom(next.handle()), {
        timeout: 5000,
        errorThresholdPercentage: 50,
        resetTimeout: 30000,
      });

      breaker.on('open', () => console.log(`Circuit opened for ${serviceName}`));
      breaker.on('halfOpen', () => console.log(`Circuit half-open for ${serviceName}`));
      breaker.on('close', () => console.log(`Circuit closed for ${serviceName}`));

      this.breakers.set(serviceName, breaker);
    }

    return from(breaker.fire()).pipe(
      catchError(err => {
        if (err.message.includes('CircuitBreaker')) {
          throw new ServiceUnavailableException('Service temporarily unavailable');
        }
        throw err;
      }),
    );
  }

  private getServiceName(context: ExecutionContext): string {
    const request = context.switchToHttp().getRequest();
    return request.url.split('/')[2]; // Extract service from URL
  }
}
```

**6. Nginx as API Gateway:**

```nginx
# nginx.conf
upstream order_service {
    least_conn;
    server order-service-1:3001;
    server order-service-2:3001;
}

upstream payment_service {
    least_conn;
    server payment-service-1:3002;
    server payment-service-2:3002;
}

server {
    listen 80;

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;

    # CORS
    add_header Access-Control-Allow-Origin * always;
    add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS" always;

    # Route to Order Service
    location /api/orders {
        limit_req zone=api_limit burst=20 nodelay;
        proxy_pass http://order_service;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        
        # Timeout
        proxy_connect_timeout 5s;
        proxy_send_timeout 10s;
        proxy_read_timeout 10s;
    }

    # Route to Payment Service
    location /api/payments {
        limit_req zone=api_limit burst=20;
        proxy_pass http://payment_service;
        proxy_set_header Host $host;
    }

    # Health check
    location /health {
        access_log off;
        return 200 "OK";
    }
}
```

**7. Kong API Gateway (Docker Compose):**

```yaml
# docker-compose.yml
version: '3.8'

services:
  kong-database:
    image: postgres:13
    environment:
      POSTGRES_USER: kong
      POSTGRES_DB: kong
      POSTGRES_PASSWORD: kong

  kong:
    image: kong:latest
    environment:
      KONG_DATABASE: postgres
      KONG_PG_HOST: kong-database
      KONG_PG_PASSWORD: kong
      KONG_PROXY_ACCESS_LOG: /dev/stdout
      KONG_ADMIN_ACCESS_LOG: /dev/stdout
      KONG_PROXY_ERROR_LOG: /dev/stderr
      KONG_ADMIN_ERROR_LOG: /dev/stderr
      KONG_ADMIN_LISTEN: 0.0.0.0:8001
    ports:
      - "8000:8000"  # Proxy
      - "8001:8001"  # Admin API
    depends_on:
      - kong-database

  # Configure Kong routes
  kong-config:
    image: curlimages/curl:latest
    depends_on:
      - kong
    command: >
      sh -c "
      sleep 10 &&
      curl -i -X POST http://kong:8001/services/ \
        --data 'name=order-service' \
        --data 'url=http://order-service:3001' &&
      curl -i -X POST http://kong:8001/services/order-service/routes \
        --data 'paths[]=/api/orders' &&
      curl -i -X POST http://kong:8001/services/order-service/plugins \
        --data 'name=rate-limiting' \
        --data 'config.minute=100'
      "
```

**Key Takeaway:** API Gateway serves as single entry point, handles: authentication/authorization (verify JWT before routing), rate limiting (per user tier), request routing (forward to correct microservice), response aggregation (combine data from multiple services), protocol translation, circuit breaker (prevent cascading failures), CORS, logging. Implement with NestJS (custom gateway) or use Kong/nginx. Benefits: centralized security, reduced client complexity, protocol flexibility, monitoring. Guards apply before routing, interceptors transform requests/responses. Use service discovery for dynamic routing.

</details>

## Data Consistency

<details>
<summary><strong>32. What is eventual consistency vs strong consistency?</strong></summary>

**Answer:**

Understand consistency models in distributed systems:

**1. Strong Consistency:**

```typescript
/*
STRONG CONSISTENCY (Immediate Consistency)

Guarantees:
- After write completes, all reads see the new value immediately
- All replicas synchronized before write returns
- Linear history of operations

Example: Bank Transfer
- $100 transferred from Account A to Account B
- Both accounts immediately reflect new balance
- No intermediate state visible

Trade-offs:
- Lower availability (CAP theorem)
- Higher latency (wait for replication)
- Synchronous writes to all replicas
- Blocking operations

Use Cases:
- Financial transactions
- Inventory management (prevent overselling)
- Booking systems
- Any operation requiring ACID guarantees
*/

// Strong consistency with database transactions
@Injectable()
export class BankingService {
  constructor(private dataSource: DataSource) {}

  async transfer(fromAccountId: string, toAccountId: string, amount: number) {
    return this.dataSource.transaction(async (manager) => {
      // Lock rows for update (pessimistic locking)
      const fromAccount = await manager.findOne(Account, {
        where: { id: fromAccountId },
        lock: { mode: 'pessimistic_write' },
      });

      const toAccount = await manager.findOne(Account, {
        where: { id: toAccountId },
        lock: { mode: 'pessimistic_write' },
      });

      if (fromAccount.balance < amount) {
        throw new BadRequestException('Insufficient funds');
      }

      // Both updates in same transaction
      fromAccount.balance -= amount;
      toAccount.balance += amount;

      await manager.save([fromAccount, toAccount]);

      // Transaction commits - both accounts updated atomically
      return { success: true };
    });
  }

  async getBalance(accountId: string): Promise<number> {
    // Always reads latest committed value
    const account = await this.accountRepo.findOne({
      where: { id: accountId },
    });
    return account.balance;
  }
}
```

**2. Eventual Consistency:**

```typescript
/*
EVENTUAL CONSISTENCY

Guarantees:
- System will become consistent "eventually"
- Intermediate states may be inconsistent
- Different replicas may show different values temporarily
- All replicas converge to same value if no new updates

Example: Social Media Like Count
- User likes a post
- Like recorded immediately in one replica
- Like count propagates to other regions asynchronously
- Eventually all regions show correct count
- Brief inconsistency acceptable

Trade-offs:
- Higher availability (CAP theorem)
- Lower latency (no waiting for replication)
- Asynchronous replication
- Non-blocking operations

Use Cases:
- Social media feeds
- View counts, like counts
- Analytics dashboards
- Cached data
- DNS updates
*/

// Eventual consistency with event-driven architecture
@Injectable()
export class PostService {
  constructor(
    @InjectRepository(Post) private postRepo: Repository<Post>,
    @Inject(CACHE_MANAGER) private cache: Cache,
    @Inject('RABBITMQ_CLIENT') private eventBus: ClientProxy,
  ) {}

  async likePost(postId: string, userId: string) {
    // 1. Record like in primary database (immediate)
    await this.likeRepo.save({ postId, userId });

    // 2. Publish event (non-blocking)
    this.eventBus.emit('post.liked', { postId, userId });

    // No wait for cache update or other services
    return { success: true };
  }

  // Event handler updates denormalized count (eventual)
  @EventPattern('post.liked')
  async handlePostLiked(data: { postId: string }) {
    // Update like count asynchronously
    await this.postRepo.increment({ id: data.postId }, 'likesCount', 1);

    // Invalidate cache (eventual)
    await this.cache.del(`post:${data.postId}`);
  }

  async getPost(postId: string) {
    // May return slightly stale like count
    return this.postRepo.findOne({ where: { id: postId } });
  }
}
```

**3. Read-Your-Writes Consistency:**

```typescript
// Guarantee: User sees their own writes immediately
// Others may see stale data

@Injectable()
export class ProfileService {
  constructor(
    @InjectRepository(User) private userRepo: Repository<User>,
    @Inject(CACHE_MANAGER) private cache: Cache,
  ) {}

  async updateProfile(userId: string, dto: UpdateProfileDto) {
    // Update database
    await this.userRepo.update(userId, dto);

    // Invalidate cache for this specific user
    await this.cache.del(`user:${userId}`);

    // Store in user-specific cache immediately
    const updatedUser = await this.userRepo.findOne({ where: { id: userId } });
    await this.cache.set(`user:${userId}`, updatedUser, 300);

    return updatedUser;
  }

  async getProfile(userId: string, requestingUserId: string) {
    // If reading own profile, always fresh data
    if (userId === requestingUserId) {
      return this.userRepo.findOne({ where: { id: userId } });
    }

    // For others, can use cached (potentially stale) data
    const cached = await this.cache.get<User>(`user:${userId}`);
    if (cached) return cached;

    return this.userRepo.findOne({ where: { id: userId } });
  }
}
```

**4. Causal Consistency:**

```typescript
// Guarantee: Operations that are causally related are seen in order
// Example: Comment appears after the post it replies to

@Injectable()
export class CommentService {
  async addComment(dto: CreateCommentDto) {
    // Include parent comment timestamp in new comment
    const parentComment = dto.parentId
      ? await this.commentRepo.findOne({ where: { id: dto.parentId } })
      : null;

    const comment = await this.commentRepo.save({
      ...dto,
      vectorClock: this.incrementVectorClock(parentComment?.vectorClock),
    });

    this.eventBus.emit('comment.created', {
      commentId: comment.id,
      vectorClock: comment.vectorClock,
    });

    return comment;
  }

  async getComments(postId: string) {
    // Sort by vector clock to maintain causal order
    return this.commentRepo.find({
      where: { postId },
      order: { vectorClock: 'ASC' },
    });
  }

  private incrementVectorClock(parentClock?: string): string {
    // Simplified vector clock implementation
    const clock = parentClock ? JSON.parse(parentClock) : {};
    const nodeId = process.env.NODE_ID || 'node1';
    clock[nodeId] = (clock[nodeId] || 0) + 1;
    return JSON.stringify(clock);
  }
}
```

**5. Consistency Level Configuration:**

```typescript
// Allow choosing consistency level per operation

enum ConsistencyLevel {
  STRONG = 'strong',         // Wait for all replicas
  EVENTUAL = 'eventual',     // Return immediately
  QUORUM = 'quorum',         // Wait for majority
}

@Injectable()
export class ConfigurableReadService {
  async readData(
    key: string,
    consistency: ConsistencyLevel = ConsistencyLevel.EVENTUAL,
  ) {
    switch (consistency) {
      case ConsistencyLevel.STRONG:
        // Read from master only
        return this.readFromMaster(key);

      case ConsistencyLevel.QUORUM:
        // Read from multiple replicas, return when majority agree
        return this.readQuorum(key);

      case ConsistencyLevel.EVENTUAL:
        // Read from nearest replica (fastest, may be stale)
        return this.readFromReplica(key);
    }
  }

  private async readFromMaster(key: string) {
    // Always fresh data, higher latency
    return this.masterDb.findOne({ where: { key } });
  }

  private async readFromReplica(key: string) {
    // Potentially stale, lower latency
    return this.replicaDb.findOne({ where: { key } });
  }

  private async readQuorum(key: string) {
    // Read from multiple replicas
    const replicas = [this.masterDb, this.replica1Db, this.replica2Db];
    const results = await Promise.all(
      replicas.map(db => db.findOne({ where: { key } })),
    );

    // Return most recent version (by timestamp/version)
    return results.reduce((latest, current) =>
      current.updatedAt > latest.updatedAt ? current : latest,
    );
  }
}
```

**6. Handling Eventual Consistency in UI:**

```typescript
// Show optimistic updates with loading states

@Injectable()
export class PostService {
  async likePost(postId: string, userId: string) {
    // Return optimistic response immediately
    const optimisticResult = { 
      liked: true,
      syncing: true, // UI shows loading indicator
    };

    // Background sync
    this.syncLike(postId, userId);

    return optimisticResult;
  }

  private async syncLike(postId: string, userId: string) {
    try {
      await this.likeRepo.save({ postId, userId });
      this.eventBus.emit('like.synced', { postId, userId, status: 'success' });
    } catch (error) {
      this.eventBus.emit('like.synced', { postId, userId, status: 'failed' });
    }
  }
}

// WebSocket notification when sync completes
@WebSocketGateway()
export class SyncGateway {
  @SubscribeMessage('like.synced')
  handleLikeSynced(client: Socket, data: any) {
    client.emit('syncComplete', data);
  }
}
```

**Key Takeaway:** **Strong Consistency**: all replicas synchronized immediately, reads always return latest write, use for financial transactions/inventory (ACID guarantees), higher latency, lower availability. **Eventual Consistency**: replicas synchronized asynchronously, reads may return stale data temporarily, use for social features/analytics/caching, higher availability, lower latency. **Read-Your-Writes**: user sees own updates immediately. **Causal Consistency**: causally related operations maintain order. Choose based on business requirements: strong for money, eventual for engagement metrics. Implement with transactions (strong), events (eventual), replication lag monitoring.

</details>

<details>
<summary><strong>33. How do you handle data consistency across services?</strong></summary>

**Answer:**

Handle cross-service data consistency with patterns and techniques:

**1. Saga Pattern (Already covered in Q28-29):**

```typescript
// Use saga for multi-service transactions
// See Q28 for detailed implementation
```

**2. Event Sourcing:**

```typescript
// Store all changes as sequence of events

@Entity('events')
export class DomainEvent {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  aggregateId: string; // Order ID

  @Column()
  aggregateType: string; // 'Order'

  @Column()
  eventType: string; // 'OrderCreated', 'OrderPaid', 'OrderShipped'

  @Column('jsonb')
  data: any;

  @Column()
  version: number; // Event sequence number

  @CreateDateColumn()
  occurredAt: Date;
}

@Injectable()
export class EventStore {
  constructor(
    @InjectRepository(DomainEvent)
    private eventRepo: Repository<DomainEvent>,
  ) {}

  async appendEvent(
    aggregateId: string,
    aggregateType: string,
    eventType: string,
    data: any,
  ) {
    const lastEvent = await this.eventRepo.findOne({
      where: { aggregateId, aggregateType },
      order: { version: 'DESC' },
    });

    const version = (lastEvent?.version || 0) + 1;

    return this.eventRepo.save({
      aggregateId,
      aggregateType,
      eventType,
      data,
      version,
    });
  }

  async getEvents(aggregateId: string): Promise<DomainEvent[]> {
    return this.eventRepo.find({
      where: { aggregateId },
      order: { version: 'ASC' },
    });
  }

  // Rebuild state from events
  async rebuildAggregate(aggregateId: string): Promise<any> {
    const events = await this.getEvents(aggregateId);
    
    let state = {};
    for (const event of events) {
      state = this.applyEvent(state, event);
    }

    return state;
  }

  private applyEvent(state: any, event: DomainEvent): any {
    switch (event.eventType) {
      case 'OrderCreated':
        return { ...event.data, status: 'pending' };
      
      case 'OrderPaid':
        return { ...state, status: 'paid', paymentId: event.data.paymentId };
      
      case 'OrderShipped':
        return { ...state, status: 'shipped', trackingNumber: event.data.trackingNumber };
      
      default:
        return state;
    }
  }
}

// Usage
@Injectable()
export class OrdersService {
  constructor(private eventStore: EventStore) {}

  async createOrder(dto: CreateOrderDto) {
    const orderId = uuid();

    await this.eventStore.appendEvent(
      orderId,
      'Order',
      'OrderCreated',
      dto,
    );

    // Publish event to other services
    this.eventBus.emit('order.created', { orderId, ...dto });

    return { orderId };
  }

  async getOrder(orderId: string) {
    // Rebuild from events
    return this.eventStore.rebuildAggregate(orderId);
  }
}
```

**3. CQRS (Command Query Responsibility Segregation):**

```typescript
// Separate write and read models

// Write Model (Command Side)
@Injectable()
export class OrderCommandService {
  constructor(
    private eventStore: EventStore,
    @Inject('RABBITMQ_CLIENT') private eventBus: ClientProxy,
  ) {}

  async createOrder(command: CreateOrderCommand) {
    // Validate command
    if (command.items.length === 0) {
      throw new BadRequestException('Order must have items');
    }

    // Store event
    await this.eventStore.appendEvent(
      command.orderId,
      'Order',
      'OrderCreated',
      command,
    );

    // Publish to read side
    this.eventBus.emit('order.created', command);

    return { orderId: command.orderId };
  }
}

// Read Model (Query Side) - Denormalized for fast queries
@Entity('order_read_model')
export class OrderReadModel {
  @PrimaryColumn()
  id: string;

  @Column()
  userId: string;

  @Column('jsonb')
  items: any[];

  @Column('decimal')
  total: number;

  @Column()
  status: string;

  // Denormalized data from other services
  @Column()
  userName: string;

  @Column()
  userEmail: string;

  @Column({ nullable: true })
  paymentStatus: string;

  @UpdateDateColumn()
  updatedAt: Date;
}

@Injectable()
export class OrderQueryService {
  constructor(
    @InjectRepository(OrderReadModel)
    private orderReadRepo: Repository<OrderReadModel>,
  ) {}

  // Fast reads from denormalized table
  async getOrder(orderId: string) {
    return this.orderReadRepo.findOne({ where: { id: orderId } });
  }

  async getUserOrders(userId: string, page: number) {
    return this.orderReadRepo.find({
      where: { userId },
      order: { updatedAt: 'DESC' },
      take: 20,
      skip: (page - 1) * 20,
    });
  }
}

// Event Handler updates read model
@Controller()
export class OrderReadModelHandler {
  constructor(
    @InjectRepository(OrderReadModel)
    private orderReadRepo: Repository<OrderReadModel>,
  ) {}

  @EventPattern('order.created')
  async handleOrderCreated(data: any) {
    await this.orderReadRepo.save({
      id: data.orderId,
      userId: data.userId,
      items: data.items,
      total: data.total,
      status: 'pending',
      userName: data.userName, // From user service
      userEmail: data.userEmail,
    });
  }

  @EventPattern('payment.completed')
  async handlePaymentCompleted(data: any) {
    await this.orderReadRepo.update(data.orderId, {
      paymentStatus: 'paid',
    });
  }
}
```

**4. Distributed Lock for Consistency:**

```typescript
// Prevent concurrent modifications

import Redlock from 'redlock';

@Injectable()
export class InventoryService {
  private redlock: Redlock;

  constructor(
    @Inject('REDIS_CLIENT') redis: Redis,
    @InjectRepository(Inventory) private inventoryRepo: Repository<Inventory>,
  ) {
    this.redlock = new Redlock([redis], {
      retryCount: 10,
      retryDelay: 200,
    });
  }

  async reserveStock(productId: string, quantity: number) {
    const lockKey = `lock:inventory:${productId}`;
    
    // Acquire lock (30 second TTL)
    const lock = await this.redlock.acquire([lockKey], 30000);

    try {
      // Critical section - only one instance can execute
      const inventory = await this.inventoryRepo.findOne({
        where: { productId },
      });

      if (inventory.available < quantity) {
        throw new BadRequestException('Insufficient stock');
      }

      inventory.available -= quantity;
      inventory.reserved += quantity;
      
      await this.inventoryRepo.save(inventory);

      return { success: true, remaining: inventory.available };
    } finally {
      // Always release lock
      await lock.release();
    }
  }
}
```

**5. Versioning for Optimistic Concurrency:**

```typescript
// Detect conflicting updates

@Entity('products')
export class Product {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  name: string;

  @Column('int')
  stock: number;

  @VersionColumn()
  version: number; // Automatically incremented

  @UpdateDateColumn()
  updatedAt: Date;
}

@Injectable()
export class ProductService {
  async updateStock(productId: string, quantity: number, expectedVersion: number) {
    try {
      const result = await this.productRepo.update(
        {
          id: productId,
          version: expectedVersion, // Check version matches
        },
        {
          stock: quantity,
        },
      );

      if (result.affected === 0) {
        throw new ConflictException(
          'Product was modified by another process. Please retry.',
        );
      }

      return { success: true };
    } catch (error) {
      if (error instanceof ConflictException) {
        throw error;
      }
      throw new InternalServerErrorException('Update failed');
    }
  }
}
```

**6. Dual Writes Prevention:**

```typescript
// DON'T: Write to database and message broker separately (can fail partially)
async createOrder(dto: CreateOrderDto) {
  const order = await this.orderRepo.save(dto); // ✅ Succeeds
  await this.eventBus.emit('order.created', order); // ❌ Fails - inconsistent!
}

// DO: Use transactional outbox pattern (see Q28)
async createOrder(dto: CreateOrderDto) {
  return this.dataSource.transaction(async (manager) => {
    // 1. Save order
    const order = await manager.save(Order, dto);

    // 2. Save outbox event (same transaction)
    await manager.save(OutboxEvent, {
      aggregateId: order.id,
      eventType: 'order.created',
      payload: order,
    });

    // Both committed atomically
    return order;
  });
}
```

**Key Takeaway:** Cross-service consistency patterns: **Saga** for distributed transactions with compensation, **Event Sourcing** storing all changes as events (append-only log), **CQRS** separating write model (events) from read model (denormalized views), **Outbox Pattern** preventing dual writes (save event in same DB transaction), **Distributed Locks** with Redlock for critical sections, **Optimistic Locking** with version columns detecting conflicts. Use events for eventual consistency, transactions for strong consistency within service. Avoid direct database sharing between services. Monitor event processing lag.

</details>

<details>
<summary><strong>34. What are distributed transactions and their challenges?</strong></summary>

**Answer:**

```typescript
// Already covered extensively in Q28 (Distributed Transactions Management)
// and Q29 (Saga Pattern)
// This question is a duplicate - refer to those answers for:
// - Two-Phase Commit (2PC) limitations
// - Saga Pattern (Choreography vs Orchestration)
// - Compensating transactions
// - Outbox pattern
// - Challenges: network failures, partial failures, eventual consistency
```

**Key Takeaway:** See Q28 for complete answer on managing distributed transactions and Q29 for saga pattern implementation.

</details>

<details>
<summary><strong>35. How do you implement idempotency in APIs?</strong></summary>

**Answer:**

Implement idempotency to safely retry operations without side effects:

**1. Idempotency Key Header:**

```typescript
// Client sends unique key for each logical request

@Post('payments')
async createPayment(
  @Body() dto: CreatePaymentDto,
  @Headers('idempotency-key') idempotencyKey: string,
) {
  if (!idempotencyKey) {
    throw new BadRequestException('Idempotency-Key header required');
  }

  // Check if request already processed
  const existing = await this.idempotencyRepo.findOne({
    where: { key: idempotencyKey },
  });

  if (existing) {
    // Return cached response
    return JSON.parse(existing.response);
  }

  // Process request
  const payment = await this.paymentService.createPayment(dto);

  // Store result
  await this.idempotencyRepo.save({
    key: idempotencyKey,
    response: JSON.stringify(payment),
    expiresAt: new Date(Date.now() + 24 * 60 * 60 * 1000), // 24 hours
  });

  return payment;
}

@Entity('idempotency_keys')
export class IdempotencyKey {
  @PrimaryColumn()
  key: string;

  @Column('text')
  response: string;

  @Column()
  status: string; // 'processing', 'completed', 'failed'

  @Column({ nullable: true, type: 'timestamp' })
  expiresAt: Date;

  @CreateDateColumn()
  createdAt: Date;
}
```

**2. Idempotency Middleware:**

```typescript
// idempotency/idempotency.middleware.ts
@Injectable()
export class IdempotencyMiddleware implements NestMiddleware {
  constructor(
    @InjectRepository(IdempotencyKey)
    private idempotencyRepo: Repository<IdempotencyKey>,
  ) {}

  async use(req: Request, res: Response, next: NextFunction) {
    const idempotencyKey = req.headers['idempotency-key'] as string;

    // Skip GET requests (already idempotent)
    if (req.method === 'GET' || !idempotencyKey) {
      return next();
    }

    // Check existing
    const existing = await this.idempotencyRepo.findOne({
      where: { key: idempotencyKey },
    });

    if (existing) {
      if (existing.status === 'processing') {
        // Request still processing - return 409 Conflict
        return res.status(409).json({
          message: 'Request is being processed',
          retryAfter: 5,
        });
      }

      if (existing.status === 'completed') {
        // Return cached response
        return res.status(200).json(JSON.parse(existing.response));
      }

      if (existing.status === 'failed') {
        // Return cached error
        return res.status(500).json(JSON.parse(existing.response));
      }
    }

    // Mark as processing
    await this.idempotencyRepo.save({
      key: idempotencyKey,
      status: 'processing',
      response: null,
    });

    // Intercept response
    const originalJson = res.json.bind(res);
    res.json = (body: any) => {
      // Store response
      this.idempotencyRepo.update(idempotencyKey, {
        status: res.statusCode < 400 ? 'completed' : 'failed',
        response: JSON.stringify(body),
        expiresAt: new Date(Date.now() + 24 * 60 * 60 * 1000),
      });

      return originalJson(body);
    };

    next();
  }
}

// Apply to routes
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(IdempotencyMiddleware)
      .forRoutes({ path: '*', method: RequestMethod.POST });
  }
}
```

**3. Database-Level Idempotency:**

```typescript
// Use unique constraints

@Entity('payments')
export class Payment {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  idempotencyKey: string; // Unique constraint

  @Column()
  orderId: string;

  @Column('decimal')
  amount: number;

  @Column()
  status: string;
}

@Injectable()
export class PaymentService {
  async createPayment(dto: CreatePaymentDto, idempotencyKey: string) {
    try {
      // Attempt insert
      const payment = await this.paymentRepo.save({
        ...dto,
        idempotencyKey,
        status: 'pending',
      });

      // Process payment with Stripe
      const stripePayment = await this.stripe.paymentIntents.create({
        amount: dto.amount * 100,
        currency: 'usd',
      });

      payment.status = 'completed';
      await this.paymentRepo.save(payment);

      return payment;
    } catch (error) {
      if (error.code === '23505') {
        // Unique constraint violation - already exists
        return this.paymentRepo.findOne({
          where: { idempotencyKey },
        });
      }
      throw error;
    }
  }
}
```

**4. Message Deduplication:**

```typescript
// Prevent duplicate event processing

@Entity('processed_messages')
export class ProcessedMessage {
  @PrimaryColumn()
  messageId: string;

  @Column()
  eventType: string;

  @CreateDateColumn()
  processedAt: Date;
}

@Injectable()
export class IdempotentEventHandler {
  constructor(
    @InjectRepository(ProcessedMessage)
    private processedRepo: Repository<ProcessedMessage>,
  ) {}

  async handleEvent(
    messageId: string,
    eventType: string,
    handler: () => Promise<void>,
  ) {
    // Check if already processed
    const existing = await this.processedRepo.findOne({
      where: { messageId },
    });

    if (existing) {
      console.log(`Message ${messageId} already processed, skipping`);
      return;
    }

    // Process event
    await handler();

    // Record as processed
    await this.processedRepo.save({
      messageId,
      eventType,
    });
  }
}

// Usage
@Controller()
export class OrderEventsController {
  constructor(private idempotentHandler: IdempotentEventHandler) {}

  @EventPattern('order.created')
  async handleOrderCreated(@Payload() message: any, @Ctx() context: any) {
    const messageId = context.getMessage().properties.messageId;

    await this.idempotentHandler.handleEvent(
      messageId,
      'order.created',
      async () => {
        await this.inventoryService.reserveStock(message.items);
      },
    );
  }
}
```

**5. HTTP Idempotent Methods:**

```typescript
/*
NATURALLY IDEMPOTENT HTTP METHODS:

GET     - Safe and idempotent
PUT     - Idempotent (full replacement)
DELETE  - Idempotent (deleting twice has same effect)
PATCH   - Can be idempotent (depends on implementation)

NOT IDEMPOTENT:
POST    - Creates new resource each time (needs idempotency key)
*/

// Idempotent PUT (full replacement)
@Put(':id')
async updateUser(@Param('id') id: string, @Body() dto: UpdateUserDto) {
  // Always results in same state
  return this.userRepo.save({ id, ...dto });
}

// Non-idempotent PATCH (increment)
@Patch(':id/increment-views')
async incrementViews(@Param('id') id: string) {
  // Multiple calls have different effects
  await this.postRepo.increment({ id }, 'views', 1);
}

// Make PATCH idempotent with version
@Patch(':id')
async updatePost(
  @Param('id') id: string,
  @Body() dto: PatchPostDto,
  @Headers('if-match') version: string,
) {
  const result = await this.postRepo.update(
    { id, version: parseInt(version) },
    dto,
  );

  if (result.affected === 0) {
    throw new ConflictException('Resource was modified');
  }
}
```

**6. Cleanup Expired Keys:**

```typescript
// Remove old idempotency keys

@Injectable()
export class IdempotencyCleanupService {
  constructor(
    @InjectRepository(IdempotencyKey)
    private idempotencyRepo: Repository<IdempotencyKey>,
  ) {}

  @Cron(CronExpression.EVERY_HOUR)
  async cleanupExpiredKeys() {
    const deleted = await this.idempotencyRepo.delete({
      expiresAt: LessThan(new Date()),
    });

    console.log(`Cleaned up ${deleted.affected} expired idempotency keys`);
  }
}
```

**Key Takeaway:** Idempotency ensures repeated requests produce same result. Implement with: **Idempotency-Key header** (client-generated UUID), check processed_requests table before execution, return cached response if exists, unique database constraints (prevent duplicate inserts), message deduplication (processed_messages table with messageId), HTTP method semantics (GET/PUT/DELETE naturally idempotent, POST needs key). Store responses for 24 hours. Handle concurrent requests (status: processing returns 409). Cleanup expired keys. Critical for payment processing, webhook retries, network retry logic. Stripe uses idempotency keys for all POST requests.

</details>

## Error Handling & Resilience

<details>
<summary><strong>36. How do you implement circuit breaker pattern?</strong></summary>

**Answer:**

Circuit breaker prevents cascading failures by stopping requests to failing services:

**1. Os opossum Circuit Breaker:**

```bash
npm install opossum
```

```typescript
// circuit-breaker/circuit-breaker.service.ts
import CircuitBreaker from 'opossum';

@Injectable()
export class CircuitBreakerService {
  private breakers: Map<string, CircuitBreaker> = new Map();

  createBreaker(name: string, action: Function, options?: any) {
    const breaker = new CircuitBreaker(action, {
      timeout: options?.timeout || 3000, // Request timeout
      errorThresholdPercentage: options?.errorThresholdPercentage || 50, // Open when 50% fail
      resetTimeout: options?.resetTimeout || 30000, // Try again after 30s
      rollingCountTimeout: options?.rollingCountTimeout || 10000, // 10s window
      rollingCountBuckets: options?.rollingCountBuckets || 10,
      volumeThreshold: options?.volumeThreshold || 10, // Min requests before opening
    });

    // Event listeners
    breaker.on('open', () => {
      console.error(`Circuit breaker OPEN for ${name}`);
      // Send alert
    });

    breaker.on('halfOpen', () => {
      console.log(`Circuit breaker HALF-OPEN for ${name}, testing...`);
    });

    breaker.on('close', () => {
      console.log(`Circuit breaker CLOSED for ${name}, recovered`);
    });

    breaker.on('fallback', (result) => {
      console.log(`Circuit breaker FALLBACK executed for ${name}`);
    });

    this.breakers.set(name, breaker);
    return breaker;
  }

  getBreaker(name: string): CircuitBreaker | undefined {
    return this.breakers.get(name);
  }
}
```

**2. HTTP Client with Circuit Breaker:**

```typescript
// http/resilient-http.service.ts
@Injectable()
export class ResilientHttpService {
  private paymentBreaker: CircuitBreaker;

  constructor(
    private httpService: HttpService,
    private circuitBreakerService: CircuitBreakerService,
  ) {
    // Create circuit breaker for payment service
    this.paymentBreaker = this.circuitBreakerService.createBreaker(
      'payment-service',
      this.callPaymentService.bind(this),
      {
        timeout: 5000,
        errorThresholdPercentage: 50,
        resetTimeout: 30000,
      },
    );

    // Fallback when circuit is open
    this.paymentBreaker.fallback(() => ({
      status: 'unavailable',
      message: 'Payment service temporarily unavailable, please try again later',
    }));
  }

  async processPayment(data: any) {
    try {
      // Call through circuit breaker
      const result = await this.paymentBreaker.fire(data);
      return result;
    } catch (error) {
      if (error.message.includes('CircuitBreaker')) {
        throw new ServiceUnavailableException(
          'Payment service unavailable',
          error.message,
        );
      }
      throw error;
    }
  }

  private async callPaymentService(data: any) {
    const response = await firstValueFrom(
      this.httpService.post('http://payment-service/api/payments', data).pipe(
        timeout(5000),
      ),
    );
    return response.data;
  }
}

// Usage
@Injectable()
export class OrdersService {
  constructor(private resilientHttp: ResilientHttpService) {}

  async createOrder(dto: CreateOrderDto) {
    const order = await this.orderRepo.save(dto);

    try {
      const payment = await this.resilientHttp.processPayment({
        orderId: order.id,
        amount: order.total,
      });

      order.paymentId = payment.id;
      order.status = 'paid';
    } catch (error) {
      if (error instanceof ServiceUnavailableException) {
        // Handle gracefully - queue for later
        order.status = 'pending_payment';
        await this.queuePaymentRetry(order.id);
      }
    }

    return this.orderRepo.save(order);
  }
}
```

**3. Decorator for Circuit Breaker:**

```typescript
// decorators/circuit-breaker.decorator.ts
export function UseCircuitBreaker(options?: CircuitBreakerOptions) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor,
  ) {
    const originalMethod = descriptor.value;

    const breaker = new CircuitBreaker(originalMethod, {
      timeout: options?.timeout || 3000,
      errorThresholdPercentage: options?.errorThresholdPercentage || 50,
      resetTimeout: options?.resetTimeout || 30000,
    });

    breaker.fallback(() => options?.fallback || null);

    descriptor.value = async function (...args: any[]) {
      return breaker.fire.apply(this, args);
    };

    return descriptor;
  };
}

// Usage
@Injectable()
export class ExternalApiService {
  @UseCircuitBreaker({
    timeout: 5000,
    errorThresholdPercentage: 60,
    fallback: { data: [], cached: true },
  })
  async fetchData() {
    const response = await fetch('https://external-api.com/data');
    return response.json();
  }
}
```

**4. Circuit Breaker States:**

```typescript
/*
CIRCUIT BREAKER STATES:

1. CLOSED (Normal Operation)
   - All requests pass through
   - Track success/failure rate
   - If failures exceed threshold → OPEN

2. OPEN (Service Down)
   - All requests fail fast (no attempt)
   - Return fallback immediately
   - After resetTimeout → HALF_OPEN

3. HALF_OPEN (Testing Recovery)
   - Allow limited requests through
   - If succeed → CLOSE
   - If fail → OPEN

Metrics Tracked:
- Total requests
- Successful requests
- Failed requests
- Error percentage
- Average latency
*/

@Injectable()
export class CircuitBreakerMonitor {
  getStatistics(breakerName: string) {
    const breaker = this.circuitBreakerService.getBreaker(breakerName);
    
    if (!breaker) {
      return null;
    }

    const stats = breaker.stats;

    return {
      state: breaker.opened ? 'OPEN' : breaker.halfOpen ? 'HALF_OPEN' : 'CLOSED',
      totalRequests: stats.fires,
      successfulRequests: stats.successes,
      failedRequests: stats.failures,
      errorRate: (stats.failures / stats.fires) * 100,
      averageResponseTime: stats.latencyMean,
      timeouts: stats.timeouts,
      fallbacks: stats.fallbacks,
    };
  }
}
```

**5. Health Check Integration:**

```typescript
// health/circuit-breaker-health.indicator.ts
@Injectable()
export class CircuitBreakerHealthIndicator extends HealthIndicator {
  constructor(private circuitBreakerService: CircuitBreakerService) {
    super();
  }

  async isHealthy(key: string): Promise<HealthIndicatorResult> {
    const criticalBreakers = ['payment-service', 'inventory-service'];
    const openBreakers = [];

    for (const name of criticalBreakers) {
      const breaker = this.circuitBreakerService.getBreaker(name);
      if (breaker && breaker.opened) {
        openBreakers.push(name);
      }
    }

    const isHealthy = openBreakers.length === 0;

    const result = this.getStatus(key, isHealthy, {
      openCircuits: openBreakers,
    });

    if (!isHealthy) {
      throw new HealthCheckError('Circuit breakers are open', result);
    }

    return result;
  }
}

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private circuitBreakerHealth: CircuitBreakerHealthIndicator,
  ) {}

  @Get()
  @HealthCheck()
  check() {
    return this.health.check([
      () => this.circuitBreakerHealth.isHealthy('circuit-breakers'),
    ]);
  }
}
```

**Key Takeaway:** Circuit breaker prevents cascading failures in microservices. **States**: CLOSED (normal, track failures) → OPEN (fail fast, return fallback) → HALF_OPEN (test recovery) → CLOSED/OPEN. Use opossum library with configurable thresholds (errorThresholdPercentage: 50%, timeout: 3000ms, resetTimeout: 30s). Implement fallback responses. Monitor state changes, fire alerts on OPEN. Prevents resource exhaustion from calling failing services. Critical for external API calls, inter-service communication. volumeThreshold ensures minimum requests before opening.

</details>

<details>
<summary><strong>37. How do you handle service failures gracefully?</strong></summary>

**Answer:**

Handle service failures with fallbacks, degraded mode, and graceful degradation:

**1. Fallback Responses:**

```typescript
// Provide alternative when service unavailable

@Injectable()
export class ProductService {
  constructor(
    private httpService: HttpService,
    @Inject(CACHE_MANAGER) private cache: Cache,
    private productRepo: Repository<Product>,
  ) {}

  async getProductRecommendations(userId: string): Promise<Product[]> {
    try {
      // Try ML recommendation service
      const response = await firstValueFrom(
        this.httpService.get(`http://recommendation-service/users/${userId}/recommendations`).pipe(
          timeout(2000),
        ),
      );
      return response.data;
    } catch (error) {
      console.warn('Recommendation service unavailable, using fallback');

      // Fallback 1: Try cache
      const cached = await this.cache.get<Product[]>(`recommendations:${userId}`);
      if (cached) {
        return cached;
      }

      // Fallback 2: Return popular products
      return this.productRepo.find({
        where: { isActive: true },
        order: { salesCount: 'DESC' },
        take: 10,
      });
    }
  }
}
```

**2. Degraded Mode:**

```typescript
// Reduce functionality instead of complete failure

@Injectable()
export class CheckoutService {
  constructor(
    private inventoryClient: ClientProxy,
    private paymentClient: ClientProxy,
    private shippingClient: ClientProxy,
  ) {}

  async processCheckout(dto: CheckoutDto): Promise<CheckoutResult> {
    const result: CheckoutResult = {
      orderId: uuid(),
      status: 'pending',
      features: {
        inventory: 'available',
        payment: 'available',
        shipping: 'available',
      },
    };

    // Try inventory check (critical)
    try {
      await firstValueFrom(
        this.inventoryClient.send('check_stock', dto.items).pipe(
          timeout(3000),
        ),
      );
      result.features.inventory = 'checked';
    } catch (error) {
      // Inventory service down - allow order but flag for manual review
      result.features.inventory = 'degraded';
      result.warnings = ['Inventory check unavailable, order needs manual review'];
    }

    // Try payment processing (critical)
    try {
      const payment = await firstValueFrom(
        this.paymentClient.send('process_payment', {
          amount: dto.total,
        }).pipe(
          timeout(5000),
        ),
      );
      result.paymentId = payment.id;
      result.features.payment = 'processed';
    } catch (error) {
      // Payment failed - cannot proceed
      result.status = 'failed';
      result.error = 'Payment processing unavailable';
      return result;
    }

    // Try shipping calculation (non-critical)
    try {
      const shipping = await firstValueFrom(
        this.shippingClient.send('calculate_shipping', dto.address).pipe(
          timeout(2000),
        ),
      );
      result.shippingCost = shipping.cost;
      result.features.shipping = 'calculated';
    } catch (error) {
      // Shipping service down - use default
      result.shippingCost = 9.99;
      result.features.shipping = 'degraded';
      result.warnings.push('Using default shipping cost');
    }

    result.status = 'completed';
    return result;
  }
}
```

**3. Bulkhead Pattern:**

```typescript
// Isolate resources to prevent total failure

@Injectable()
export class TaskQueueService {
  // Separate queues for different task types
  private criticalQueue: Queue;
  private normalQueue: Queue;
  private lowPriorityQueue: Queue;

  constructor(@InjectQueue('tasks') private taskQueue: Queue) {
    this.initializeQueues();
  }

  private initializeQueues() {
    // Critical tasks: dedicated resources
    this.criticalQueue = new Queue('critical-tasks', {
      redis: { host: 'redis', port: 6379 },
      limiter: {
        max: 50, // Max 50 concurrent critical tasks
        duration: 1000,
      },
    });

    // Normal tasks: shared resources
    this.normalQueue = new Queue('normal-tasks', {
      redis: { host: 'redis', port: 6379 },
      limiter: {
        max: 100,
        duration: 1000,
      },
    });

    // Low priority: limited resources
    this.lowPriorityQueue = new Queue('low-priority-tasks', {
      redis: { host: 'redis', port: 6379 },
      limiter: {
        max: 20,
        duration: 1000,
      },
    });
  }

  async addTask(task: Task) {
    switch (task.priority) {
      case 'critical':
        return this.criticalQueue.add(task.name, task.data);
      case 'normal':
        return this.normalQueue.add(task.name, task.data);
      case 'low':
        return this.lowPriorityQueue.add(task.name, task.data);
    }
  }
}
```

**4. Timeout Strategy:**

```typescript
// Set aggressive timeouts to fail fast

@Injectable()
export class UserService {
  constructor(
    private httpService: HttpService,
    private userRepo: Repository<User>,
  ) {}

  async enrichUserData(userId: string): Promise<EnrichedUser> {
    const user = await this.userRepo.findOne({ where: { id: userId } });

    // Parallel requests with timeouts
    const [analytics, preferences, socialProfile] = await Promise.allSettled([
      this.getAnalytics(userId, 1000), // 1s timeout
      this.getPreferences(userId, 500), // 500ms timeout
      this.getSocialProfile(userId, 2000), // 2s timeout
    ]);

    return {
      ...user,
      analytics: analytics.status === 'fulfilled' ? analytics.value : null,
      preferences: preferences.status === 'fulfilled' ? preferences.value : {},
      socialProfile: socialProfile.status === 'fulfilled' ? socialProfile.value : null,
    };
  }

  private async getAnalytics(userId: string, timeoutMs: number) {
    return firstValueFrom(
      this.httpService.get(`http://analytics-service/users/${userId}`).pipe(
        timeout(timeoutMs),
        catchError(() => of(null)), // Return null on failure
      ),
    );
  }
}
```

**5. Health-Based Routing:**

```typescript
// Route traffic only to healthy instances

@Injectable()
export class HealthAwareLoadBalancer {
  private healthStatus: Map<string, boolean> = new Map();

  constructor(private serviceDiscovery: ServiceDiscoveryService) {
    // Check health periodically
    setInterval(() => this.checkHealth(), 10000);
  }

  async getHealthyInstance(serviceName: string): Promise<ServiceInstance> {
    const instances = await this.serviceDiscovery.getServiceInstances(serviceName);
    
    // Filter to healthy instances only
    const healthy = instances.filter(instance => 
      this.healthStatus.get(instance.id) !== false,
    );

    if (healthy.length === 0) {
      // No healthy instances - try any instance (degraded mode)
      console.warn(`No healthy instances for ${serviceName}, trying anyway`);
      return instances[0];
    }

    // Return random healthy instance
    return healthy[Math.floor(Math.random() * healthy.length)];
  }

  private async checkHealth() {
    const instances = await this.serviceDiscovery.getAllInstances();

    for (const instance of instances) {
      try {
        const response = await fetch(`http://${instance.address}:${instance.port}/health`, {
          timeout: 2000,
        });
        
        this.healthStatus.set(instance.id, response.ok);
      } catch (error) {
        this.healthStatus.set(instance.id, false);
      }
    }
  }
}
```

**6. Feature Flags for Gradual Degradation:**

```typescript
// Disable features progressively under load

@Injectable()
export class FeatureFlagService {
  private flags: Map<string, boolean> = new Map();

  constructor(@Inject(CACHE_MANAGER) private cache: Cache) {
    this.loadFlags();
  }

  async isEnabled(feature: string): Promise<boolean> {
    // Check system health
    const systemLoad = await this.getSystemLoad();

    // Automatically disable non-critical features under high load
    if (systemLoad > 80) {
      const nonCritical = ['recommendations', 'social_features', 'analytics'];
      if (nonCritical.includes(feature)) {
        return false;
      }
    }

    return this.flags.get(feature) ?? false;
  }

  async setFlag(feature: string, enabled: boolean) {
    this.flags.set(feature, enabled);
    await this.cache.set(`feature:${feature}`, enabled, 3600);
  }

  private async getSystemLoad(): Promise<number> {
    // Calculate based on CPU, memory, response times
    return 0; // Simplified
  }
}

// Usage
@Injectable()
export class ProductService {
  async getProduct(id: string) {
    const product = await this.productRepo.findOne({ where: { id } });

    // Optional enrichment
    if (await this.featureFlags.isEnabled('recommendations')) {
      product.recommendations = await this.getRecommendations(id);
    }

    if (await this.featureFlags.isEnabled('reviews')) {
      product.reviews = await this.getReviews(id);
    }

    return product;
  }
}
```

**Key Takeaway:** Graceful degradation strategies: **Fallback responses** (cache → default data → empty result), **Degraded mode** (reduce functionality, continue with warnings), **Bulkhead pattern** (isolate resources per priority), **Aggressive timeouts** (fail fast, don't wait), **Health-based routing** (send traffic only to healthy instances), **Feature flags** (disable non-critical features under load), **Promise.allSettled** (partial success acceptable). Prioritize critical flows (payment) over nice-to-have (recommendations). Log degraded states for monitoring. Better to serve partially than fail completely.

</details>

<details>
<summary><strong>38. What is retry strategy with exponential backoff?</strong></summary>

**Answer:**

Implement intelligent retry logic to handle transient failures:

**1. Exponential Backoff Implementation:**

```typescript
// retry/exponential-backoff.service.ts
@Injectable()
export class ExponentialBackoffService {
  async retry<T>(
    fn: () => Promise<T>,
    options: RetryOptions = {},
  ): Promise<T> {
    const {
      maxAttempts = 3,
      initialDelay = 1000, // 1 second
      maxDelay = 60000, // 1 minute
      backoffMultiplier = 2,
      shouldRetry = () => true,
    } = options;

    let attempt = 0;
    let delay = initialDelay;

    while (attempt < maxAttempts) {
      try {
        return await fn();
      } catch (error) {
        attempt++;

        if (attempt >= maxAttempts || !shouldRetry(error)) {
          throw error;
        }

        // Calculate next delay with jitter
        const jitter = Math.random() * 0.3 * delay; // ±30% randomness
        const nextDelay = Math.min(delay + jitter, maxDelay);

        console.log(
          `Attempt ${attempt} failed, retrying in ${Math.round(nextDelay)}ms...`,
        );

        await this.sleep(nextDelay);
        delay *= backoffMultiplier;
      }
    }
  }

  private sleep(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

interface RetryOptions {
  maxAttempts?: number;
  initialDelay?: number;
  maxDelay?: number;
  backoffMultiplier?: number;
  shouldRetry?: (error: any) => boolean;
}
```

**2. HTTP Client with Retry:**

```typescript
// http/retry-http.service.ts
import { retry, timer } from 'rxjs';

@Injectable()
export class RetryHttpService {
  constructor(private httpService: HttpService) {}

  async get<T>(url: string, options: RetryOptions = {}): Promise<T> {
    const {
      maxAttempts = 3,
      backoffDelay = 1000,
    } = options;

    const response = await firstValueFrom(
      this.httpService.get<T>(url).pipe(
        retry({
          count: maxAttempts,
          delay: (error, retryCount) => {
            // Exponential backoff: 1s, 2s, 4s, 8s...
            const delay = backoffDelay * Math.pow(2, retryCount - 1);
            
            console.log(`Retry ${retryCount} after ${delay}ms`);
            
            // Only retry on specific errors
            if (this.shouldRetry(error)) {
              return timer(delay);
            }
            
            throw error;
          },
        }),
        timeout(10000),
      ),
    );

    return response.data;
  }

  private shouldRetry(error: any): boolean {
    // Retry on network errors and 5xx status codes
    if (!error.response) {
      return true; // Network error
    }

    const status = error.response.status;
    
    // Retry on server errors (500-599)
    if (status >= 500 && status < 600) {
      return true;
    }

    // Retry on rate limit (429)
    if (status === 429) {
      return true;
    }

    // Don't retry on client errors (400-499 except 429)
    return false;
  }
}
```

**3. Decorator for Retry Logic:**

```typescript
// decorators/retry.decorator.ts
export function Retry(options: RetryDecoratorOptions = {}) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor,
  ) {
    const originalMethod = descriptor.value;

    descriptor.value = async function (...args: any[]) {
      const {
        maxAttempts = 3,
        backoffMs = 1000,
        shouldRetry = () => true,
      } = options;

      for (let attempt = 1; attempt <= maxAttempts; attempt++) {
        try {
          return await originalMethod.apply(this, args);
        } catch (error) {
          if (attempt === maxAttempts || !shouldRetry(error)) {
            throw error;
          }

          const delay = backoffMs * Math.pow(2, attempt - 1);
          console.log(`Attempt ${attempt} failed, retrying in ${delay}ms`);
          
          await new Promise(resolve => setTimeout(resolve, delay));
        }
      }
    };

    return descriptor;
  };
}

// Usage
@Injectable()
export class PaymentService {
  @Retry({
    maxAttempts: 5,
    backoffMs: 2000,
    shouldRetry: (error) => error.code !== 'CARD_DECLINED',
  })
  async processPayment(paymentData: any) {
    return this.stripe.paymentIntents.create(paymentData);
  }
}
```

**4. Queue-Based Retry:**

```typescript
// Bull queue with retry strategy
@Processor('email')
export class EmailProcessor {
  @Process('send')
  async sendEmail(job: Job) {
    try {
      await this.emailService.send(job.data);
    } catch (error) {
      // Job will automatically retry with exponential backoff
      throw error;
    }
  }
}

// Queue configuration
BullModule.registerQueue({
  name: 'email',
  defaultJobOptions: {
    attempts: 5, // Try 5 times
    backoff: {
      type: 'exponential',
      delay: 2000, // Start with 2s, then 4s, 8s, 16s, 32s
    },
    removeOnComplete: true,
    removeOnFail: false, // Keep failed jobs for investigation
  },
});
```

**5. Idempotent Retry:**

```typescript
// Ensure retries are safe

@Injectable()
export class IdempotentPaymentService {
  async processPayment(orderId: string, amount: number) {
    // Generate idempotency key
    const idempotencyKey = `payment_${orderId}`;

    return this.retryService.retry(
      async () => {
        return this.stripe.paymentIntents.create(
          {
            amount: amount * 100,
            currency: 'usd',
          },
          {
            idempotencyKey, // Stripe deduplicates by key
          },
        );
      },
      {
        maxAttempts: 3,
        initialDelay: 1000,
      },
    );
  }
}
```

**6. Circuit Breaker + Retry:**

```typescript
// Combine patterns for resilience

@Injectable()
export class ResilientService {
  constructor(
    private retryService: ExponentialBackoffService,
    private circuitBreaker: CircuitBreakerService,
  ) {}

  async callExternalService(data: any) {
    // First, check circuit breaker
    const breaker = this.circuitBreaker.getBreaker('external-service');
    
    if (breaker.opened) {
      throw new ServiceUnavailableException('Circuit breaker is open');
    }

    // Then, retry with exponential backoff
    return this.retryService.retry(
      async () => {
        const response = await fetch('https://api.example.com/data', {
          method: 'POST',
          body: JSON.stringify(data),
        });

        if (!response.ok) {
          throw new Error(`HTTP ${response.status}`);
        }

        return response.json();
      },
      {
        maxAttempts: 3,
        initialDelay: 1000,
        shouldRetry: (error) => {
          // Don't retry on 4xx client errors
          return !error.message.includes('HTTP 4');
        },
      },
    );
  }
}
```

**7. Retry with Dead Letter Queue:**

```typescript
// Move to DLQ after max retries

@Processor('orders')
export class OrderProcessor {
  constructor(
    @InjectQueue('orders') private orderQueue: Queue,
    @InjectQueue('dead-letter') private deadLetterQueue: Queue,
  ) {}

  @Process('process-order')
  async processOrder(job: Job) {
    try {
      await this.orderService.processOrder(job.data);
    } catch (error) {
      if (job.attemptsMade >= 5) {
        // Max retries reached - move to DLQ
        await this.deadLetterQueue.add('failed-order', {
          originalJob: job.data,
          error: error.message,
          attempts: job.attemptsMade,
        });

        // Don't throw - mark as complete to prevent further retries
        return;
      }

      // Let Bull retry
      throw error;
    }
  }
}
```

**Key Takeaway:** Exponential backoff increases delay between retries: 1s → 2s → 4s → 8s (backoffMultiplier: 2). Add jitter (random ±30%) to prevent thundering herd. **Retry only transient failures**: network errors, 5xx server errors, 429 rate limit. **Don't retry**: 4xx client errors (except 429), authentication failures. Use with idempotency keys (prevent duplicate operations). Combine with circuit breaker (stop retrying if service consistently failing). Configure max attempts (3-5), max delay (60s). Bull queues support automatic exponential backoff. Move to dead letter queue after max retries for manual investigation.

</details>

<details>
<summary><strong>39. How do you implement graceful shutdown?</strong></summary>

**Answer:**

Implement graceful shutdown to complete in-flight requests before terminating:

**1. Basic Graceful Shutdown:**

```typescript
// main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Enable graceful shutdown hooks
  app.enableShutdownHooks();

  await app.listen(3000);

  // Handle termination signals
  process.on('SIGTERM', async () => {
    console.log('SIGTERM signal received: closing HTTP server');
    await app.close();
    console.log('HTTP server closed');
    process.exit(0);
  });

  process.on('SIGINT', async () => {
    console.log('SIGINT signal received: closing HTTP server');
    await app.close();
    process.exit(0);
  });
}
bootstrap();
```

**2. Complete Shutdown with Cleanup:**

```typescript
// shutdown/graceful-shutdown.service.ts
@Injectable()
export class GracefulShutdownService implements OnModuleDestroy {
  private isShuttingDown = false;
  private activeConnections = 0;

  constructor(
    @InjectDataSource() private dataSource: DataSource,
    @Inject('REDIS_CLIENT') private redis: Redis,
    @InjectQueue('tasks') private taskQueue: Queue,
  ) {}

  async onModuleDestroy() {
    console.log('Starting graceful shutdown...');
    this.isShuttingDown = true;

    // 1. Stop accepting new requests
    console.log('Stopping new requests...');

    // 2. Wait for active connections to complete
    await this.waitForActiveConnections();

    // 3. Close database connections
    console.log('Closing database connections...');
    await this.dataSource.destroy();

    // 4. Close Redis connections
    console.log('Closing Redis connections...');
    await this.redis.quit();

    // 5. Close queue connections
    console.log('Closing queue connections...');
    await this.taskQueue.close();

    // 6. Finish any background tasks
    await this.finishBackgroundTasks();

    console.log('Graceful shutdown complete');
  }

  trackConnection() {
    this.activeConnections++;
  }

  releaseConnection() {
    this.activeConnections--;
  }

  isShutdown(): boolean {
    return this.isShuttingDown;
  }

  private async waitForActiveConnections(timeout = 30000) {
    const startTime = Date.now();

    while (this.activeConnections > 0) {
      if (Date.now() - startTime > timeout) {
        console.warn(
          `Shutdown timeout reached with ${this.activeConnections} active connections`,
        );
        break;
      }

      console.log(`Waiting for ${this.activeConnections} active connections...`);
      await new Promise(resolve => setTimeout(resolve, 1000));
    }
  }

  private async finishBackgroundTasks() {
    // Wait for scheduled tasks, cron jobs, etc.
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
}
```

**3. Middleware to Track Connections:**

```typescript
// middleware/connection-tracker.middleware.ts
@Injectable()
export class ConnectionTrackerMiddleware implements NestMiddleware {
  constructor(private shutdownService: GracefulShutdownService) {}

  use(req: Request, res: Response, next: NextFunction) {
    // Reject new requests during shutdown
    if (this.shutdownService.isShutdown()) {
      return res.status(503).json({
        error: 'Service Unavailable',
        message: 'Server is shutting down',
      });
    }

    // Track active connection
    this.shutdownService.trackConnection();

    // Release on response finish
    res.on('finish', () => {
      this.shutdownService.releaseConnection();
    });

    next();
  }
}

// Apply globally
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(ConnectionTrackerMiddleware).forRoutes('*');
  }
}
```

**4. Health Check for Shutdown:**

```typescript
// health/shutdown-health.indicator.ts
@Injectable()
export class ShutdownHealthIndicator extends HealthIndicator {
  constructor(private shutdownService: GracefulShutdownService) {
    super();
  }

  async isHealthy(key: string): Promise<HealthIndicatorResult> {
    const isShutdown = this.shutdownService.isShutdown();

    if (isShutdown) {
      throw new HealthCheckError(
        'Application is shutting down',
        this.getStatus(key, false, { shutdown: true }),
      );
    }

    return this.getStatus(key, true, { shutdown: false });
  }
}

// Load balancer checks this endpoint
@Controller('health')
export class HealthController {
  @Get('ready')
  @HealthCheck()
  readiness() {
    return this.health.check([
      () => this.shutdownHealth.isHealthy('shutdown'),
      () => this.dbHealth.isHealthy('database'),
    ]);
  }
}
```

**5. Kubernetes Graceful Shutdown:**

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nestjs-app
spec:
  template:
    spec:
      containers:
      - name: app
        image: nestjs-app:latest
        ports:
        - containerPort: 3000
        
        # Readiness probe - stop sending traffic when shutting down
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
        
        # Liveness probe - restart if unhealthy
        livenessProbe:
          httpGet:
            path: /health/live
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        
        # Lifecycle hooks
        lifecycle:
          preStop:
            exec:
              # Wait before sending SIGTERM
              command: ["/bin/sh", "-c", "sleep 10"]
        
      # Grace period for shutdown
      terminationGracePeriodSeconds: 30
```

**6. Worker Process Shutdown:**

```typescript
// workers/worker.service.ts
@Processor('tasks')
export class TaskProcessor implements OnModuleDestroy {
  private isShuttingDown = false;
  private activeJobs = 0;

  @Process('process-task')
  async handleTask(job: Job) {
    if (this.isShuttingDown) {
      // Reject new jobs during shutdown
      throw new Error('Worker is shutting down');
    }

    this.activeJobs++;

    try {
      await this.taskService.processTask(job.data);
    } finally {
      this.activeJobs--;
    }
  }

  async onModuleDestroy() {
    console.log('Worker shutting down...');
    this.isShuttingDown = true;

    // Wait for active jobs to complete
    while (this.activeJobs > 0) {
      console.log(`Waiting for ${this.activeJobs} active jobs...`);
      await new Promise(resolve => setTimeout(resolve, 1000));
    }

    // Pause queue (stop accepting new jobs)
    await this.taskQueue.pause();

    console.log('Worker shutdown complete');
  }
}
```

**7. PM2 Graceful Reload:**

```javascript
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'nestjs-app',
    script: './dist/main.js',
    instances: 4,
    exec_mode: 'cluster',
    
    // Graceful shutdown
    kill_timeout: 30000, // 30 seconds to finish requests
    wait_ready: true, // Wait for app.listen() before considering ready
    listen_timeout: 10000,
    
    // Graceful reload
    max_memory_restart: '1G',
  }],
};

// In main.ts, notify PM2 when ready
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);

  // Notify PM2 app is ready
  if (process.send) {
    process.send('ready');
  }
}
```

**Key Takeaway:** Graceful shutdown prevents dropped requests. **Steps**: enable shutdown hooks with app.enableShutdownHooks(), handle SIGTERM/SIGINT signals, reject new requests (503 Service Unavailable), wait for active connections to complete (track with middleware), close database/Redis/queue connections, finish background tasks, exit after cleanup (30s timeout). **Kubernetes**: readiness probe returns unhealthy during shutdown (stops new traffic), preStop hook delays SIGTERM (10s), terminationGracePeriodSeconds: 30 allows cleanup. Track activeConnections, wait before forcing shutdown. PM2: kill_timeout, wait_ready. Critical for zero-downtime deployments.

</details>

<details>
<summary><strong>40. How do you handle database connection failures?</strong></summary>

**Answer:**

Handle database connection failures with retry logic, connection pooling, and fallbacks:

**1. TypeORM Connection Retry:**

```typescript
// database.config.ts
export default {
  type: 'postgres',
  host: process.env.DB_HOST,
  port: 5432,
  username: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  
  // Connection pool configuration
  extra: {
    max: 20, // Maximum connections
    min: 5, // Minimum connections
    idleTimeoutMillis: 30000,
    connectionTimeoutMillis: 5000,
  },

  // Automatic reconnection
  connectTimeout: 10000,
  acquireTimeout: 10000,
  
  // Retry connection on startup
  retryAttempts: 10,
  retryDelay: 3000, // 3 seconds between retries
  
  // Auto-load entities
  entities: [__dirname + '/../**/*.entity{.ts,.js}'],
  synchronize: false,
  logging: ['error', 'warn'],
};
```

**2. Manual Connection Retry:**

```typescript
// database/database-connection.service.ts
@Injectable()
export class DatabaseConnectionService {
  private reconnectAttempts = 0;
  private maxReconnectAttempts = 10;

  constructor(@InjectDataSource() private dataSource: DataSource) {
    this.setupConnectionHandlers();
  }

  private setupConnectionHandlers() {
    // Handle connection errors
    this.dataSource.driver.connection?.on('error', (error) => {
      console.error('Database connection error:', error);
      this.handleConnectionError(error);
    });

    // Handle disconnection
    this.dataSource.driver.connection?.on('end', () => {
      console.warn('Database connection ended, attempting reconnect...');
      this.reconnect();
    });
  }

  private async handleConnectionError(error: any) {
    console.error('Database error:', error);

    // Check if error is connection-related
    if (this.isConnectionError(error)) {
      await this.reconnect();
    }
  }

  private async reconnect() {
    if (this.reconnectAttempts >= this.maxReconnectAttempts) {
      console.error('Max reconnection attempts reached, giving up');
      // Send critical alert
      this.sendAlert('Database connection failed after max retries');
      return;
    }

    this.reconnectAttempts++;
    const delay = Math.min(1000 * Math.pow(2, this.reconnectAttempts), 30000);

    console.log(`Reconnection attempt ${this.reconnectAttempts} in ${delay}ms`);

    await this.sleep(delay);

    try {
      if (!this.dataSource.isInitialized) {
        await this.dataSource.initialize();
      }

      console.log('Database reconnected successfully');
      this.reconnectAttempts = 0;
    } catch (error) {
      console.error('Reconnection failed:', error);
      await this.reconnect();
    }
  }

  private isConnectionError(error: any): boolean {
    const connectionErrors = [
      'ECONNREFUSED',
      'ENOTFOUND',
      'ETIMEDOUT',
      'ECONNRESET',
      'Connection terminated',
    ];

    return connectionErrors.some(err => 
      error.message?.includes(err) || error.code === err,
    );
  }

  private sleep(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }

  private sendAlert(message: string) {
    // Send to monitoring system
    console.error('ALERT:', message);
  }
}
```

**3. Query Retry with Fallback:**

```typescript
// database/resilient-repository.service.ts
@Injectable()
export class ResilientRepositoryService {
  constructor(
    @InjectDataSource() private dataSource: DataSource,
    @Inject(CACHE_MANAGER) private cache: Cache,
  ) {}

  async query<T>(
    queryFn: () => Promise<T>,
    cacheKey?: string,
    cacheTTL = 300,
  ): Promise<T> {
    try {
      const result = await this.retryQuery(queryFn);

      // Cache successful result
      if (cacheKey) {
        await this.cache.set(cacheKey, result, cacheTTL);
      }

      return result;
    } catch (error) {
      console.error('Query failed after retries:', error);

      // Try cache fallback
      if (cacheKey) {
        const cached = await this.cache.get<T>(cacheKey);
        if (cached) {
          console.warn('Using cached data due to database failure');
          return cached;
        }
      }

      throw error;
    }
  }

  private async retryQuery<T>(queryFn: () => Promise<T>): Promise<T> {
    const maxAttempts = 3;
    let attempt = 0;

    while (attempt < maxAttempts) {
      try {
        return await queryFn();
      } catch (error) {
        attempt++;

        if (attempt >= maxAttempts) {
          throw error;
        }

        if (this.isTransientError(error)) {
          const delay = 1000 * attempt;
          console.log(`Query failed, retrying in ${delay}ms...`);
          await this.sleep(delay);
        } else {
          // Not transient, don't retry
          throw error;
        }
      }
    }
  }

  private isTransientError(error: any): boolean {
    // Errors that might resolve on retry
    const transientErrors = [
      'ECONNRESET',
      'ETIMEDOUT',
      'Connection terminated',
      'deadlock detected',
    ];

    return transientErrors.some(err => error.message?.includes(err));
  }

  private sleep(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

// Usage
@Injectable()
export class ProductService {
  constructor(
    @InjectRepository(Product) private productRepo: Repository<Product>,
    private resilientRepo: ResilientRepositoryService,
  ) {}

  async getProduct(id: string) {
    return this.resilientRepo.query(
      () => this.productRepo.findOne({ where: { id } }),
      `product:${id}`,
      3600,
    );
  }
}
```

**4. Health Check for Database:**

```typescript
// health/database-health.indicator.ts
@Injectable()
export class DatabaseHealthIndicator extends HealthIndicator {
  constructor(@InjectDataSource() private dataSource: DataSource) {
    super();
  }

  async isHealthy(key: string): Promise<HealthIndicatorResult> {
    try {
      // Simple query to test connection
      await this.dataSource.query('SELECT 1');

      return this.getStatus(key, true, {
        isConnected: this.dataSource.isInitialized,
      });
    } catch (error) {
      throw new HealthCheckError(
        'Database connection failed',
        this.getStatus(key, false, {
          message: error.message,
        }),
      );
    }
  }
}

@Controller('health')
export class HealthController {
  @Get()
  @HealthCheck()
  check() {
    return this.health.check([
      () => this.dbHealth.isHealthy('database'),
    ]);
  }
}
```

**5. Read-Only Mode on Connection Failure:**

```typescript
// Service degradation when database unavailable

@Injectable()
export class OrderService {
  private readOnlyMode = false;

  constructor(
    @InjectDataSource() private dataSource: DataSource,
    @Inject(CACHE_MANAGER) private cache: Cache,
  ) {
    this.monitorConnection();
  }

  async getOrder(id: string) {
    if (this.readOnlyMode) {
      // Serve from cache only
      const cached = await this.cache.get(`order:${id}`);
      if (!cached) {
        throw new ServiceUnavailableException(
          'Service in read-only mode, data not cached',
        );
      }
      return cached;
    }

    // Normal database query
    return this.orderRepo.findOne({ where: { id } });
  }

  async createOrder(dto: CreateOrderDto) {
    if (this.readOnlyMode) {
      throw new ServiceUnavailableException(
        'Service in read-only mode, writes disabled',
      );
    }

    return this.orderRepo.save(dto);
  }

  private monitorConnection() {
    setInterval(async () => {
      try {
        await this.dataSource.query('SELECT 1');
        if (this.readOnlyMode) {
          console.log('Database recovered, exiting read-only mode');
          this.readOnlyMode = false;
        }
      } catch (error) {
        if (!this.readOnlyMode) {
          console.error('Database unavailable, entering read-only mode');
          this.readOnlyMode = true;
        }
      }
    }, 10000); // Check every 10 seconds
  }
}
```

**6. Database Circuit Breaker:**

```typescript
// Prevent overwhelming failed database

@Injectable()
export class DatabaseCircuitBreaker {
  private breaker: CircuitBreaker;

  constructor(@InjectDataSource() private dataSource: DataSource) {
    this.breaker = new CircuitBreaker(
      async (query: string, params?: any[]) => {
        return this.dataSource.query(query, params);
      },
      {
        timeout: 5000,
        errorThresholdPercentage: 50,
        resetTimeout: 30000,
      },
    );

    this.breaker.on('open', () => {
      console.error('Database circuit breaker OPEN');
    });

    this.breaker.fallback(() => {
      throw new ServiceUnavailableException('Database temporarily unavailable');
    });
  }

  async query<T>(query: string, params?: any[]): Promise<T> {
    return this.breaker.fire(query, params);
  }
}
```

**Key Takeaway:** Handle DB connection failures with: **TypeORM config** (retryAttempts: 10, retryDelay: 3000ms), **Connection pooling** (max: 20, min: 5, idleTimeoutMillis: 30s), **Automatic reconnection** with exponential backoff, **Query retry** for transient errors (ECONNRESET, ETIMEDOUT, deadlock), **Cache fallback** when DB unavailable, **Read-only mode** (serve cached data, block writes), **Health checks** (SELECT 1 test query), **Circuit breaker** prevents overwhelming failed DB. Monitor connection events (error, end). Differentiate transient vs permanent errors. Alert on max retry attempts. Critical for maintaining availability during DB maintenance/failures.

</details>

## Monitoring & Observability

<details>
<summary><strong>41. How do you implement comprehensive logging?</strong></summary>

**Answer:**

Implement structured logging with correlation IDs, log aggregation, and proper log levels:

**1. Winston Logger Setup:**

```bash
npm install winston nest-winston
```

```typescript
// logger/winston.config.ts
import { WinstonModule } from 'nest-winston';
import * as winston from 'winston';

export const winstonConfig = {
  transports: [
    // Console transport for development
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.colorize(),
        winston.format.printf(({ timestamp, level, message, context, trace, ...meta }) => {
          return `${timestamp} [${context}] ${level}: ${message} ${
            Object.keys(meta).length ? JSON.stringify(meta) : ''
          }`;
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
};

// main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule, {
    logger: WinstonModule.createLogger(winstonConfig),
  });
  
  await app.listen(3000);
}
```

**2. Structured Logging Service:**

```typescript
// logger/logger.service.ts
import { Injectable, LoggerService as NestLoggerService } from '@nestjs/common';
import { Logger } from 'winston';
import { Inject } from '@nestjs/common';
import { WINSTON_MODULE_PROVIDER } from 'nest-winston';

@Injectable()
export class LoggerService implements NestLoggerService {
  constructor(
    @Inject(WINSTON_MODULE_PROVIDER) private readonly logger: Logger,
  ) {}

  log(message: string, context?: string, metadata?: any) {
    this.logger.info(message, { context, ...metadata });
  }

  error(message: string, trace?: string, context?: string, metadata?: any) {
    this.logger.error(message, { context, trace, ...metadata });
  }

  warn(message: string, context?: string, metadata?: any) {
    this.logger.warn(message, { context, ...metadata });
  }

  debug(message: string, context?: string, metadata?: any) {
    this.logger.debug(message, { context, ...metadata });
  }

  verbose(message: string, context?: string, metadata?: any) {
    this.logger.verbose(message, { context, ...metadata });
  }

  // Structured logging methods
  logRequest(req: any, res: any, responseTime: number) {
    this.logger.info('HTTP Request', {
      method: req.method,
      url: req.url,
      statusCode: res.statusCode,
      responseTime: `${responseTime}ms`,
      userId: req.user?.id,
      correlationId: req.headers['x-correlation-id'],
      userAgent: req.headers['user-agent'],
      ip: req.ip,
    });
  }

  logError(error: Error, context?: string, metadata?: any) {
    this.logger.error(error.message, {
      context,
      stack: error.stack,
      name: error.name,
      ...metadata,
    });
  }

  logBusinessEvent(event: string, data: any) {
    this.logger.info('Business Event', {
      event,
      ...data,
    });
  }
}
```

**3. Request Correlation ID Middleware:**

```typescript
// middleware/correlation-id.middleware.ts
import { v4 as uuid } from 'uuid';

@Injectable()
export class CorrelationIdMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    // Get or generate correlation ID
    const correlationId = req.headers['x-correlation-id'] || uuid();
    req['correlationId'] = correlationId;

    // Add to response headers
    res.setHeader('x-correlation-id', correlationId);

    // Store in async context (for use in services)
    AsyncLocalStorage.getStore()?.set('correlationId', correlationId);

    next();
  }
}

// Apply globally
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(CorrelationIdMiddleware).forRoutes('*');
  }
}
```

**4. Logging Interceptor:**

```typescript
// interceptors/logging.interceptor.ts
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  constructor(private logger: LoggerService) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const now = Date.now();
    const request = context.switchToHttp().getRequest();
    const { method, url, headers, body, user } = request;

    // Log incoming request
    this.logger.log('Incoming Request', 'HTTP', {
      method,
      url,
      correlationId: headers['x-correlation-id'],
      userId: user?.id,
      body: this.sanitizeBody(body),
    });

    return next.handle().pipe(
      tap({
        next: (data) => {
          const response = context.switchToHttp().getResponse();
          const responseTime = Date.now() - now;

          this.logger.logRequest(request, response, responseTime);
        },
        error: (error) => {
          const responseTime = Date.now() - now;

          this.logger.error('Request Failed', error.stack, 'HTTP', {
            method,
            url,
            statusCode: error.status || 500,
            responseTime: `${responseTime}ms`,
            correlationId: headers['x-correlation-id'],
            userId: user?.id,
          });
        },
      }),
    );
  }

  private sanitizeBody(body: any): any {
    if (!body) return body;

    // Remove sensitive fields
    const sanitized = { ...body };
    const sensitiveFields = ['password', 'token', 'creditCard', 'ssn'];

    for (const field of sensitiveFields) {
      if (sanitized[field]) {
        sanitized[field] = '***REDACTED***';
      }
    }

    return sanitized;
  }
}

// Apply globally
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

**5. Exception Logging Filter:**

```typescript
// filters/exception-logging.filter.ts
@Catch()
export class ExceptionLoggingFilter implements ExceptionFilter {
  constructor(private logger: LoggerService) {}

  catch(exception: any, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const request = ctx.getRequest();
    const response = ctx.getResponse();

    const status = exception instanceof HttpException
      ? exception.getStatus()
      : 500;

    const errorResponse = {
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      method: request.method,
      message: exception.message || 'Internal server error',
      correlationId: request.headers['x-correlation-id'],
    };

    // Log error with full context
    this.logger.logError(exception, 'ExceptionFilter', {
      statusCode: status,
      path: request.url,
      method: request.method,
      userId: request.user?.id,
      correlationId: request.headers['x-correlation-id'],
      requestBody: request.body,
      query: request.query,
    });

    response.status(status).json(errorResponse);
  }
}
```

**6. Database Query Logging:**

```typescript
// Configure TypeORM to log slow queries
export default {
  type: 'postgres',
  // ... other config
  logging: ['error', 'warn', 'migration'],
  logger: 'advanced-console',
  maxQueryExecutionTime: 1000, // Log queries slower than 1s
};

// Custom query logger
import { Logger as TypeORMLogger } from 'typeorm';

export class DatabaseLogger implements TypeORMLogger {
  constructor(private logger: LoggerService) {}

  logQuery(query: string, parameters?: any[]) {
    this.logger.debug('SQL Query', 'Database', {
      query,
      parameters,
    });
  }

  logQueryError(error: string, query: string, parameters?: any[]) {
    this.logger.error('SQL Query Error', error, 'Database', {
      query,
      parameters,
    });
  }

  logQuerySlow(time: number, query: string, parameters?: any[]) {
    this.logger.warn('Slow SQL Query', 'Database', {
      executionTime: `${time}ms`,
      query,
      parameters,
    });
  }

  logSchemaBuild(message: string) {
    this.logger.log(message, 'DatabaseSchema');
  }

  logMigration(message: string) {
    this.logger.log(message, 'DatabaseMigration');
  }

  log(level: 'log' | 'info' | 'warn', message: any) {
    switch (level) {
      case 'log':
      case 'info':
        this.logger.log(message, 'Database');
        break;
      case 'warn':
        this.logger.warn(message, 'Database');
        break;
    }
  }
}
```

**7. Centralized Log Aggregation (ELK Stack):**

```typescript
// logger/elasticsearch-transport.ts
import { ElasticsearchTransport } from 'winston-elasticsearch';

const esTransport = new ElasticsearchTransport({
  level: 'info',
  clientOpts: {
    node: process.env.ELASTICSEARCH_URL || 'http://localhost:9200',
    auth: {
      username: process.env.ES_USERNAME,
      password: process.env.ES_PASSWORD,
    },
  },
  index: 'nestjs-logs',
  indexPrefix: 'logs',
  indexSuffixPattern: 'YYYY.MM.DD',
  transformer: (logData) => {
    return {
      '@timestamp': logData.timestamp,
      message: logData.message,
      level: logData.level,
      context: logData.meta.context,
      correlationId: logData.meta.correlationId,
      userId: logData.meta.userId,
      ...logData.meta,
    };
  },
});

// Add to Winston config
export const winstonConfig = {
  transports: [
    new winston.transports.Console(),
    esTransport,
  ],
};
```

**8. Docker Compose for ELK Stack:**

```yaml
# docker-compose.elk.yml
version: '3.8'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.5.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data

  logstash:
    image: docker.elastic.co/logstash/logstash:8.5.0
    ports:
      - "5044:5044"
    volumes:
      - ./logstash/pipeline:/usr/share/logstash/pipeline
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:8.5.0
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch

  filebeat:
    image: docker.elastic.co/beats/filebeat:8.5.0
    user: root
    volumes:
      - ./filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
    depends_on:
      - elasticsearch
      - logstash

volumes:
  elasticsearch-data:
```

**Key Takeaway:** Comprehensive logging needs: **Structured logs** (JSON format with timestamp/level/context/metadata), **Correlation IDs** (trace requests across services with x-correlation-id header), **Log levels** (ERROR for failures, WARN for degraded, INFO for business events, DEBUG for debugging), **Sanitize sensitive data** (passwords, tokens, PII), **Log aggregation** (ELK stack: Elasticsearch + Logstash + Kibana), **Query logging** (slow queries >1s), **Request/Response logging** (method, URL, status, response time, user ID), **Error context** (stack trace, correlation ID, user ID, request body). Use Winston with transports (Console, File, Elasticsearch). Critical for debugging production issues.

</details>

<details>
<summary><strong>42. How do you implement distributed tracing?</strong></summary>

**Answer:**

Implement distributed tracing with OpenTelemetry or Jaeger to track requests across microservices:

**1. OpenTelemetry Setup:**

```bash
npm install @opentelemetry/api @opentelemetry/sdk-node @opentelemetry/auto-instrumentations-node @opentelemetry/exporter-jaeger
```

```typescript
// tracing/tracing.ts
import { NodeSDK } from '@opentelemetry/sdk-node';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';
import { JaegerExporter } from '@opentelemetry/exporter-jaeger';
import { Resource } from '@opentelemetry/resources';
import { SemanticResourceAttributes } from '@opentelemetry/semantic-conventions';

const jaegerExporter = new JaegerExporter({
  endpoint: process.env.JAEGER_ENDPOINT || 'http://localhost:14268/api/traces',
});

const sdk = new NodeSDK({
  resource: new Resource({
    [SemanticResourceAttributes.SERVICE_NAME]: 'nestjs-api',
    [SemanticResourceAttributes.SERVICE_VERSION]: '1.0.0',
  }),
  traceExporter: jaegerExporter,
  instrumentations: [
    getNodeAutoInstrumentations({
      '@opentelemetry/instrumentation-http': {
        enabled: true,
      },
      '@opentelemetry/instrumentation-pg': {
        enabled: true,
      },
      '@opentelemetry/instrumentation-redis': {
        enabled: true,
      },
    }),
  ],
});

sdk.start();

process.on('SIGTERM', () => {
  sdk
    .shutdown()
    .then(() => console.log('Tracing terminated'))
    .catch((error) => console.error('Error terminating tracing', error))
    .finally(() => process.exit(0));
});

// Import at the top of main.ts
import './tracing/tracing';
```

**2. Custom Span Creation:**

```typescript
// tracing/tracing.service.ts
import { Injectable } from '@nestjs/common';
import { trace, context, SpanStatusCode } from '@opentelemetry/api';

@Injectable()
export class TracingService {
  private tracer = trace.getTracer('nestjs-api');

  async traceMethod<T>(
    name: string,
    fn: () => Promise<T>,
    attributes?: Record<string, any>,
  ): Promise<T> {
    const span = this.tracer.startSpan(name);

    if (attributes) {
      span.setAttributes(attributes);
    }

    try {
      const result = await context.with(
        trace.setSpan(context.active(), span),
        fn,
      );

      span.setStatus({ code: SpanStatusCode.OK });
      return result;
    } catch (error) {
      span.setStatus({
        code: SpanStatusCode.ERROR,
        message: error.message,
      });
      span.recordException(error);
      throw error;
    } finally {
      span.end();
    }
  }

  addEvent(name: string, attributes?: Record<string, any>) {
    const span = trace.getActiveSpan();
    span?.addEvent(name, attributes);
  }

  setAttribute(key: string, value: any) {
    const span = trace.getActiveSpan();
    span?.setAttribute(key, value);
  }
}

// Usage
@Injectable()
export class OrderService {
  constructor(private tracingService: TracingService) {}

  async createOrder(dto: CreateOrderDto) {
    return this.tracingService.traceMethod(
      'OrderService.createOrder',
      async () => {
        // Set custom attributes
        this.tracingService.setAttribute('user.id', dto.userId);
        this.tracingService.setAttribute('order.total', dto.total);

        const order = await this.orderRepo.save(dto);

        // Add event
        this.tracingService.addEvent('order.created', {
          orderId: order.id,
        });

        return order;
      },
      {
        'service.name': 'order-service',
        'operation': 'create',
      },
    );
  }
}
```

**3. Tracing Decorator:**

```typescript
// decorators/trace.decorator.ts
export function Trace(spanName?: string) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor,
  ) {
    const originalMethod = descriptor.value;
    const tracer = trace.getTracer('nestjs-api');

    descriptor.value = async function (...args: any[]) {
      const name = spanName || `${target.constructor.name}.${propertyKey}`;
      const span = tracer.startSpan(name);

      try {
        const result = await context.with(
          trace.setSpan(context.active(), span),
          () => originalMethod.apply(this, args),
        );

        span.setStatus({ code: SpanStatusCode.OK });
        return result;
      } catch (error) {
        span.setStatus({
          code: SpanStatusCode.ERROR,
          message: error.message,
        });
        span.recordException(error);
        throw error;
      } finally {
        span.end();
      }
    };

    return descriptor;
  };
}

// Usage
@Injectable()
export class ProductService {
  @Trace('ProductService.getProduct')
  async getProduct(id: string) {
    return this.productRepo.findOne({ where: { id } });
  }

  @Trace()
  async updateProduct(id: string, dto: UpdateProductDto) {
    return this.productRepo.update(id, dto);
  }
}
```

**4. HTTP Propagation (Context Passing):**

```typescript
// interceptors/trace-propagation.interceptor.ts
@Injectable()
export class TracePropagationInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const span = trace.getActiveSpan();

    if (span) {
      // Add trace context to all outgoing HTTP calls
      const traceId = span.spanContext().traceId;
      const spanId = span.spanContext().spanId;

      request['traceContext'] = {
        traceId,
        spanId,
      };
    }

    return next.handle();
  }
}

// HTTP client with trace propagation
@Injectable()
export class TracedHttpService {
  constructor(private httpService: HttpService) {}

  async get(url: string, traceContext?: any) {
    const headers = {
      'x-trace-id': traceContext?.traceId,
      'x-span-id': traceContext?.spanId,
    };

    return firstValueFrom(
      this.httpService.get(url, { headers }),
    );
  }

  async post(url: string, data: any, traceContext?: any) {
    const headers = {
      'x-trace-id': traceContext?.traceId,
      'x-span-id': traceContext?.spanId,
    };

    return firstValueFrom(
      this.httpService.post(url, data, { headers }),
    );
  }
}
```

**5. Database Query Tracing:**

```typescript
// subscribers/query-tracing.subscriber.ts
@EventSubscriber()
export class QueryTracingSubscriber implements EntitySubscriberInterface {
  beforeQuery(event: any) {
    const span = trace.getActiveSpan();
    
    if (span) {
      span.addEvent('database.query.start', {
        'db.statement': event.query,
        'db.system': 'postgresql',
      });
    }

    event['startTime'] = Date.now();
  }

  afterQuery(event: any) {
    const span = trace.getActiveSpan();
    const duration = Date.now() - event['startTime'];

    if (span) {
      span.addEvent('database.query.end', {
        'db.statement': event.query,
        'db.duration': duration,
      });

      // Warn on slow queries
      if (duration > 1000) {
        span.setAttribute('db.slow_query', true);
      }
    }
  }
}
```

**6. Microservice Trace Propagation:**

```typescript
// For microservices communication via RabbitMQ/Kafka

@Injectable()
export class OrderService {
  constructor(
    @Inject('PAYMENT_SERVICE') private paymentClient: ClientProxy,
    private tracingService: TracingService,
  ) {}

  async processOrder(order: Order) {
    return this.tracingService.traceMethod(
      'processOrder',
      async () => {
        // Get active trace context
        const span = trace.getActiveSpan();
        const traceContext = {
          traceId: span?.spanContext().traceId,
          spanId: span?.spanContext().spanId,
        };

        // Send trace context with message
        await firstValueFrom(
          this.paymentClient.send('process_payment', {
            orderId: order.id,
            amount: order.total,
            _traceContext: traceContext,
          }),
        );
      },
    );
  }
}

// Receiving service
@Controller()
export class PaymentController {
  @MessagePattern('process_payment')
  async processPayment(@Payload() data: any) {
    // Extract trace context
    const { _traceContext, ...paymentData } = data;

    // Continue trace
    if (_traceContext) {
      const span = trace.getTracer('payment-service').startSpan(
        'PaymentController.processPayment',
        {
          links: [
            {
              context: {
                traceId: _traceContext.traceId,
                spanId: _traceContext.spanId,
                traceFlags: 1,
              },
            },
          ],
        },
      );

      try {
        return await context.with(
          trace.setSpan(context.active(), span),
          () => this.paymentService.process(paymentData),
        );
      } finally {
        span.end();
      }
    }

    return this.paymentService.process(paymentData);
  }
}
```

**7. Jaeger Docker Compose:**

```yaml
# docker-compose.jaeger.yml
version: '3.8'

services:
  jaeger:
    image: jaegertracing/all-in-one:latest
    ports:
      - "5775:5775/udp"   # Zipkin compact
      - "6831:6831/udp"   # Jaeger agent
      - "6832:6832/udp"   # Jaeger agent
      - "5778:5778"       # HTTP config
      - "16686:16686"     # Jaeger UI
      - "14268:14268"     # Jaeger collector HTTP
      - "14250:14250"     # Jaeger GRPC
      - "9411:9411"       # Zipkin compatible
    environment:
      - COLLECTOR_ZIPKIN_HOST_PORT=:9411

  nestjs-api:
    build: .
    environment:
      - JAEGER_ENDPOINT=http://jaeger:14268/api/traces
    depends_on:
      - jaeger
```

**Key Takeaway:** Distributed tracing tracks requests across microservices with **OpenTelemetry** (vendor-neutral) or **Jaeger**. **Spans** represent operations (HTTP request, DB query, service call), **Trace ID** links all spans for a request. Automatically instrument HTTP, PostgreSQL, Redis with getNodeAutoInstrumentations(). Create custom spans with tracer.startSpan(), add attributes (user.id, order.total), record events (order.created). **Propagate context** via HTTP headers (x-trace-id, x-span-id) and message metadata (_traceContext). View traces in Jaeger UI (http://localhost:16686). Track latency bottlenecks, error paths, service dependencies. Critical for debugging microservice issues.

</details>

<details>
<summary><strong>43. How do you set up alerting for critical errors?</strong></summary>

**Answer:**

Set up alerting using monitoring platforms to notify on critical errors:

**1. Sentry Integration:**

```bash
npm install @sentry/node @sentry/tracing
```

```typescript
// sentry/sentry.config.ts
import * as Sentry from '@sentry/node';
import * as Tracing from '@sentry/tracing';

export function initSentry(app: INestApplication) {
  Sentry.init({
    dsn: process.env.SENTRY_DSN,
    environment: process.env.NODE_ENV || 'development',
    release: process.env.APP_VERSION || '1.0.0',
    
    // Performance monitoring
    tracesSampleRate: 0.1, // 10% of transactions
    
    // Error filtering
    beforeSend(event, hint) {
      // Don't send 404s
      if (event.exception?.values?.[0]?.value?.includes('NotFoundException')) {
        return null;
      }
      
      return event;
    },
    
    integrations: [
      new Tracing.Integrations.Postgres(),
      new Tracing.Integrations.Http({ tracing: true }),
    ],
  });
}

// main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  initSentry(app);
  
  await app.listen(3000);
}
```

**2. Sentry Exception Filter:**

```typescript
// filters/sentry.filter.ts
@Catch()
export class SentryExceptionFilter implements ExceptionFilter {
  catch(exception: any, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const request = ctx.getRequest();
    const response = ctx.getResponse();

    // Capture exception in Sentry
    Sentry.withScope((scope) => {
      // Add request context
      scope.setContext('http', {
        method: request.method,
        url: request.url,
        headers: request.headers,
        query: request.query,
        body: request.body,
      });

      // Add user context
      if (request.user) {
        scope.setUser({
          id: request.user.id,
          email: request.user.email,
          username: request.user.username,
        });
      }

      // Set severity
      if (exception.status >= 500) {
        scope.setLevel('error');
      } else if (exception.status >= 400) {
        scope.setLevel('warning');
      }

      // Set tags
      scope.setTags({
        endpoint: `${request.method} ${request.url}`,
        status: exception.status || 500,
      });

      Sentry.captureException(exception);
    });

    const status = exception instanceof HttpException
      ? exception.getStatus()
      : 500;

    response.status(status).json({
      statusCode: status,
      message: exception.message,
      timestamp: new Date().toISOString(),
    });
  }
}

// Apply globally
@Module({
  providers: [
    {
      provide: APP_FILTER,
      useClass: SentryExceptionFilter,
    },
  ],
})
export class AppModule {}
```

**3. Custom Alerts Service:**

```typescript
// alerts/alerts.service.ts
@Injectable()
export class AlertsService {
  constructor(
    private httpService: HttpService,
    private logger: LoggerService,
  ) {}

  async sendCriticalAlert(error: Error, context: any) {
    const alert = {
      severity: 'critical',
      title: error.name,
      message: error.message,
      timestamp: new Date().toISOString(),
      context: {
        service: 'nestjs-api',
        environment: process.env.NODE_ENV,
        ...context,
      },
      stack: error.stack,
    };

    // Send to multiple channels
    await Promise.allSettled([
      this.sendToSlack(alert),
      this.sendToPagerDuty(alert),
      this.sendToEmail(alert),
    ]);
  }

  private async sendToSlack(alert: any) {
    try {
      await firstValueFrom(
        this.httpService.post(process.env.SLACK_WEBHOOK_URL, {
          text: `🚨 *${alert.severity.toUpperCase()}*: ${alert.title}`,
          blocks: [
            {
              type: 'section',
              text: {
                type: 'mrkdwn',
                text: `*Message:* ${alert.message}\n*Service:* ${alert.context.service}\n*Environment:* ${alert.context.environment}`,
              },
            },
            {
              type: 'context',
              elements: [
                {
                  type: 'mrkdwn',
                  text: `Timestamp: ${alert.timestamp}`,
                },
              ],
            },
          ],
        }),
      );
    } catch (error) {
      this.logger.error('Failed to send Slack alert', error.stack);
    }
  }

  private async sendToPagerDuty(alert: any) {
    try {
      await firstValueFrom(
        this.httpService.post('https://events.pagerduty.com/v2/enqueue', {
          routing_key: process.env.PAGERDUTY_ROUTING_KEY,
          event_action: 'trigger',
          payload: {
            summary: `${alert.severity}: ${alert.title}`,
            severity: alert.severity,
            source: alert.context.service,
            custom_details: {
              message: alert.message,
              context: alert.context,
              stack: alert.stack,
            },
          },
        }),
      );
    } catch (error) {
      this.logger.error('Failed to send PagerDuty alert', error.stack);
    }
  }

  private async sendToEmail(alert: any) {
    // Email implementation
  }
}

// Usage
@Injectable()
export class PaymentService {
  constructor(private alertsService: AlertsService) {}

  async processPayment(data: any) {
    try {
      return await this.stripe.paymentIntents.create(data);
    } catch (error) {
      if (error.code === 'payment_provider_down') {
        // Critical alert
        await this.alertsService.sendCriticalAlert(error, {
          payment: data,
          userId: data.userId,
        });
      }
      throw error;
    }
  }
}
```

**4. Prometheus + Alertmanager:**

```typescript
// Install
// npm install @willsoto/nestjs-prometheus prom-client

// metrics/metrics.module.ts
import { PrometheusModule } from '@willsoto/nestjs-prometheus';
import { makeCounterProvider, makeHistogramProvider } from '@willsoto/nestjs-prometheus';

@Module({
  imports: [
    PrometheusModule.register({
      defaultMetrics: {
        enabled: true,
      },
      path: '/metrics',
    }),
  ],
  providers: [
    makeCounterProvider({
      name: 'http_requests_total',
      help: 'Total HTTP requests',
      labelNames: ['method', 'path', 'status'],
    }),
    makeHistogramProvider({
      name: 'http_request_duration_seconds',
      help: 'HTTP request duration',
      labelNames: ['method', 'path', 'status'],
      buckets: [0.1, 0.5, 1, 2, 5],
    }),
    makeCounterProvider({
      name: 'errors_total',
      help: 'Total errors',
      labelNames: ['type', 'severity'],
    }),
  ],
})
export class MetricsModule {}

// Use in service
@Injectable()
export class OrderService {
  constructor(
    @InjectMetric('errors_total') private errorsCounter: Counter,
  ) {}

  async createOrder(dto: CreateOrderDto) {
    try {
      return await this.orderRepo.save(dto);
    } catch (error) {
      this.errorsCounter.inc({
        type: error.name,
        severity: 'high',
      });
      throw error;
    }
  }
}
```

**5. Prometheus Alert Rules:**

```yaml
# prometheus/alert-rules.yml
groups:
  - name: application_alerts
    interval: 30s
    rules:
      # High error rate
      - alert: HighErrorRate
        expr: rate(errors_total[5m]) > 10
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value }} errors/sec"

      # Response time degradation
      - alert: HighLatency
        expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 2
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High latency detected"
          description: "95th percentile is {{ $value }}s"

      # Service down
      - alert: ServiceDown
        expr: up{job="nestjs-api"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Service is down"
          description: "Service {{ $labels.instance }} is down"

      # Database connection issues
      - alert: DatabaseConnectionFailure
        expr: database_connection_errors_total > 5
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Database connection failures"
          description: "{{ $value }} connection failures"

      # Memory usage
      - alert: HighMemoryUsage
        expr: process_resident_memory_bytes / 1024 / 1024 > 500
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage"
          description: "Memory usage is {{ $value }}MB"
```

**6. Alertmanager Configuration:**

```yaml
# alertmanager/config.yml
global:
  resolve_timeout: 5m
  slack_api_url: 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL'

route:
  group_by: ['alertname', 'severity']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
  receiver: 'default'
  
  routes:
    # Critical alerts to PagerDuty + Slack
    - match:
        severity: critical
      receiver: 'pagerduty-critical'
      continue: true
    
    # Warning alerts to Slack only
    - match:
        severity: warning
      receiver: 'slack-warnings'

receivers:
  - name: 'default'
    slack_configs:
      - channel: '#alerts'
        title: '{{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'

  - name: 'pagerduty-critical'
    pagerduty_configs:
      - service_key: 'YOUR_PAGERDUTY_KEY'
        description: '{{ .GroupLabels.alertname }}: {{ .CommonAnnotations.summary }}'

  - name: 'slack-warnings'
    slack_configs:
      - channel: '#warnings'
        title: '⚠️ Warning: {{ .GroupLabels.alertname }}'
```

**7. Health-Based Alerting:**

```typescript
// Schedule health checks and alert on failures

@Injectable()
export class HealthMonitorService {
  constructor(
    private health: HealthCheckService,
    private alertsService: AlertsService,
  ) {}

  @Cron('*/5 * * * *') // Every 5 minutes
  async checkHealth() {
    try {
      const result = await this.health.check([
        () => this.dbHealth.isHealthy('database'),
        () => this.redisHealth.isHealthy('redis'),
        () => this.circuitBreakerHealth.isHealthy('circuit-breakers'),
      ]);

      if (result.status === 'error') {
        await this.alertsService.sendCriticalAlert(
          new Error('Health check failed'),
          {
            checks: result.info,
            errors: result.error,
          },
        );
      }
    } catch (error) {
      await this.alertsService.sendCriticalAlert(error, {
        type: 'health-check-failure',
      });
    }
  }
}
```

**Key Takeaway:** Critical error alerting via multiple channels: **Sentry** (automatic error capturing, stack traces, user context, release tracking), **Slack** (webhooks for instant notifications), **PagerDuty** (on-call rotations, escalation policies), **Email** (detailed error reports). **Prometheus + Alertmanager** for metric-based alerts (high error rate, latency spikes, service down). Define alert rules (HighErrorRate, ServiceDown, DatabaseConnectionFailure). Route by severity: critical → PagerDuty + Slack, warning → Slack only. Include context (user ID, request data, environment). Filter noise (ignore 404s). Schedule health checks with @Cron, alert on failures. Critical for rapid incident response.

</details>

<details>
<summary><strong>44. What metrics should you track in production?</strong></summary>

**Answer:**

Track comprehensive metrics for performance, errors, business, and infrastructure:

**1. Prometheus Metrics Setup:**

```typescript
// metrics/metrics.service.ts
import { Injectable } from '@nestjs/common';
import { InjectMetric } from '@willsoto/nestjs-prometheus';
import { Counter, Histogram, Gauge } from 'prom-client';

@Injectable()
export class MetricsService {
  constructor(
    @InjectMetric('http_requests_total') private httpRequestsCounter: Counter,
    @InjectMetric('http_request_duration_seconds') private httpDurationHistogram: Histogram,
    @InjectMetric('database_queries_total') private dbQueriesCounter: Counter,
    @InjectMetric('database_query_duration_seconds') private dbDurationHistogram: Histogram,
    @InjectMetric('active_connections') private activeConnectionsGauge: Gauge,
    @InjectMetric('cache_hits_total') private cacheHitsCounter: Counter,
    @InjectMetric('cache_misses_total') private cacheMissesCounter: Counter,
    @InjectMetric('business_events_total') private businessEventsCounter: Counter,
  ) {}

  // HTTP metrics
  recordHttpRequest(method: string, path: string, status: number, duration: number) {
    this.httpRequestsCounter.inc({ method, path, status });
    this.httpDurationHistogram.observe({ method, path, status }, duration / 1000);
  }

  // Database metrics
  recordDbQuery(operation: string, table: string, duration: number) {
    this.dbQueriesCounter.inc({ operation, table });
    this.dbDurationHistogram.observe({ operation, table }, duration / 1000);
  }

  // Connection metrics
  incrementActiveConnections() {
    this.activeConnectionsGauge.inc();
  }

  decrementActiveConnections() {
    this.activeConnectionsGauge.dec();
  }

  // Cache metrics
  recordCacheHit(key: string) {
    this.cacheHitsCounter.inc({ key });
  }

  recordCacheMiss(key: string) {
    this.cacheMissesCounter.inc({ key });
  }

  // Business metrics
  recordBusinessEvent(event: string, metadata?: Record<string, string>) {
    this.businessEventsCounter.inc({ event, ...metadata });
  }
}
```

**2. Metrics Interceptor:**

```typescript
// interceptors/metrics.interceptor.ts
@Injectable()
export class MetricsInterceptor implements NestInterceptor {
  constructor(private metricsService: MetricsService) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const startTime = Date.now();

    this.metricsService.incrementActiveConnections();

    return next.handle().pipe(
      tap({
        next: () => {
          const response = context.switchToHttp().getResponse();
          const duration = Date.now() - startTime;

          this.metricsService.recordHttpRequest(
            request.method,
            this.normalizePath(request.route?.path || request.url),
            response.statusCode,
            duration,
          );

          this.metricsService.decrementActiveConnections();
        },
        error: (error) => {
          const duration = Date.now() - startTime;

          this.metricsService.recordHttpRequest(
            request.method,
            this.normalizePath(request.route?.path || request.url),
            error.status || 500,
            duration,
          );

          this.metricsService.decrementActiveConnections();
        },
      }),
    );
  }

  private normalizePath(path: string): string {
    // Replace IDs with placeholders
    return path.replace(/\/[0-9a-f-]{36}/g, '/:id')
               .replace(/\/\d+/g, '/:id');
  }
}
```

**3. Database Query Metrics:**

```typescript
// subscribers/metrics.subscriber.ts
@EventSubscriber()
export class MetricsSubscriber implements EntitySubscriberInterface {
  constructor(private metricsService: MetricsService) {}

  beforeQuery(event: any) {
    event['startTime'] = Date.now();
  }

  afterQuery(event: any) {
    const duration = Date.now() - event['startTime'];
    const operation = this.extractOperation(event.query);
    const table = this.extractTable(event.query);

    this.metricsService.recordDbQuery(operation, table, duration);
  }

  private extractOperation(query: string): string {
    const match = query.match(/^(SELECT|INSERT|UPDATE|DELETE)/i);
    return match ? match[1].toUpperCase() : 'UNKNOWN';
  }

  private extractTable(query: string): string {
    const match = query.match(/FROM\s+(\w+)|INTO\s+(\w+)|UPDATE\s+(\w+)/i);
    return match ? (match[1] || match[2] || match[3]) : 'unknown';
  }
}
```

**4. Cache Metrics:**

```typescript
// cache/cache.service.ts
@Injectable()
export class CacheService {
  constructor(
    @Inject(CACHE_MANAGER) private cache: Cache,
    private metricsService: MetricsService,
  ) {}

  async get<T>(key: string): Promise<T | undefined> {
    const value = await this.cache.get<T>(key);

    if (value !== undefined) {
      this.metricsService.recordCacheHit(this.normalizeKey(key));
    } else {
      this.metricsService.recordCacheMiss(this.normalizeKey(key));
    }

    return value;
  }

  async set(key: string, value: any, ttl?: number): Promise<void> {
    await this.cache.set(key, value, ttl);
  }

  private normalizeKey(key: string): string {
    // Group similar keys
    return key.replace(/:[0-9a-f-]+/, ':*').replace(/:\d+/, ':*');
  }
}
```

**5. Business Metrics:**

```typescript
// Track business-specific metrics

@Injectable()
export class OrderService {
  constructor(private metricsService: MetricsService) {}

  async createOrder(dto: CreateOrderDto) {
    const order = await this.orderRepo.save(dto);

    // Record business event
    this.metricsService.recordBusinessEvent('order.created', {
      userId: order.userId,
      total: order.total.toString(),
    });

    return order;
  }

  async completeOrder(orderId: string) {
    const order = await this.orderRepo.findOne({ where: { id: orderId } });
    order.status = 'completed';
    await this.orderRepo.save(order);

    this.metricsService.recordBusinessEvent('order.completed', {
      orderId: order.id,
    });
  }

  async cancelOrder(orderId: string, reason: string) {
    const order = await this.orderRepo.findOne({ where: { id: orderId } });
    order.status = 'cancelled';
    await this.orderRepo.save(order);

    this.metricsService.recordBusinessEvent('order.cancelled', {
      orderId: order.id,
      reason,
    });
  }
}
```

**6. Custom Gauges for System Metrics:**

```typescript
// metrics/system-metrics.service.ts
@Injectable()
export class SystemMetricsService implements OnModuleInit {
  constructor(
    @InjectMetric('nodejs_heap_size_used_bytes') private heapUsedGauge: Gauge,
    @InjectMetric('nodejs_external_memory_bytes') private externalMemGauge: Gauge,
    @InjectMetric('nodejs_event_loop_lag_seconds') private eventLoopLagGauge: Gauge,
    @InjectDataSource() private dataSource: DataSource,
    @Inject('REDIS_CLIENT') private redis: Redis,
  ) {}

  onModuleInit() {
    // Update system metrics every 10 seconds
    setInterval(() => this.collectSystemMetrics(), 10000);
  }

  private async collectSystemMetrics() {
    // Memory metrics
    const memUsage = process.memoryUsage();
    this.heapUsedGauge.set(memUsage.heapUsed);
    this.externalMemGauge.set(memUsage.external);

    // Event loop lag
    const start = Date.now();
    setImmediate(() => {
      const lag = Date.now() - start;
      this.eventLoopLagGauge.set(lag / 1000);
    });

    // Database pool metrics
    const pool = (this.dataSource.driver as any).master;
    if (pool) {
      this.dbPoolTotalGauge.set(pool.totalCount || 0);
      this.dbPoolIdleGauge.set(pool.idleCount || 0);
      this.dbPoolWaitingGauge.set(pool.waitingCount || 0);
    }

    // Redis metrics
    const redisInfo = await this.redis.info('stats');
    // Parse and set redis metrics
  }
}
```

**7. Grafana Dashboard Configuration:**

```json
{
  "dashboard": {
    "title": "NestJS Application Metrics",
    "panels": [
      {
        "title": "Request Rate",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])"
          }
        ]
      },
      {
        "title": "Request Duration (p95)",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))"
          }
        ]
      },
      {
        "title": "Error Rate",
        "targets": [
          {
            "expr": "rate(http_requests_total{status=~\"5..\"}[5m])"
          }
        ]
      },
      {
        "title": "Database Query Duration",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(database_query_duration_seconds_bucket[5m]))"
          }
        ]
      },
      {
        "title": "Cache Hit Rate",
        "targets": [
          {
            "expr": "rate(cache_hits_total[5m]) / (rate(cache_hits_total[5m]) + rate(cache_misses_total[5m])) * 100"
          }
        ]
      },
      {
        "title": "Active Connections",
        "targets": [
          {
            "expr": "active_connections"
          }
        ]
      },
      {
        "title": "Memory Usage",
        "targets": [
          {
            "expr": "nodejs_heap_size_used_bytes / 1024 / 1024"
          }
        ]
      },
      {
        "title": "Business Events",
        "targets": [
          {
            "expr": "rate(business_events_total[5m])"
          }
        ]
      }
    ]
  }
}
```

**Key Metrics Categories:**

```typescript
/*
1. HTTP METRICS:
   - http_requests_total (counter): Total requests by method/path/status
   - http_request_duration_seconds (histogram): Request latency
   - active_connections (gauge): Current active connections

2. DATABASE METRICS:
   - database_queries_total (counter): Queries by operation/table
   - database_query_duration_seconds (histogram): Query latency
   - database_connections_active (gauge): Active DB connections
   - database_connections_idle (gauge): Idle DB connections

3. CACHE METRICS:
   - cache_hits_total (counter): Cache hits by key pattern
   - cache_misses_total (counter): Cache misses
   - cache_hit_rate: hits / (hits + misses) * 100

4. ERROR METRICS:
   - errors_total (counter): Errors by type/severity
   - error_rate: errors per second

5. BUSINESS METRICS:
   - business_events_total: order.created, order.completed, user.signup
   - revenue_total: Total revenue
   - active_users_total: Active users

6. SYSTEM METRICS:
   - nodejs_heap_size_used_bytes: Memory usage
   - nodejs_event_loop_lag_seconds: Event loop lag
   - process_cpu_usage_percent: CPU usage

7. EXTERNAL SERVICES:
   - external_api_calls_total: Calls to external APIs
   - external_api_duration_seconds: External API latency
   - circuit_breaker_state: Open/Closed/HalfOpen
*/
```

**Key Takeaway:** Track metrics across **Performance** (request/query latency p95, throughput), **Errors** (error rate, 5xx responses), **Business** (orders created, revenue, signups), **Infrastructure** (memory, CPU, event loop lag, DB pool), **Cache** (hit rate, miss rate), **External** (API latency, circuit breaker state). Use **Prometheus** for scraping /metrics endpoint, **Grafana** for visualization. Expose as histograms (latency buckets), counters (totals), gauges (current values). Normalize paths (/users/:id), keys (user:*). Alert on high error rate, latency spikes, low cache hit rate. Critical for identifying bottlenecks and capacity planning.

</details>

## Security Scenarios

<details>
<summary><strong>45. How do you prevent DDoS attacks?</strong></summary>

**Answer:**

Implement multiple layers of DDoS protection at network, application, and infrastructure levels:

**1. Rate Limiting (Application Layer):**

```bash
npm install @nestjs/throttler
```

```typescript
// app.module.ts
@Module({
  imports: [
    ThrottlerModule.forRoot({
      ttl: 60, // Time window in seconds
      limit: 10, // Max requests per window
      storage: new ThrottlerStorageRedisService(new Redis({
        host: 'localhost',
        port: 6379,
      })),
    }),
  ],
  providers: [
    {
      provide: APP_GUARD,
      useClass: ThrottlerGuard,
    },
  ],
})
export class AppModule {}

// Custom rate limits per endpoint
@Controller('api')
export class ApiController {
  @Post('expensive-operation')
  @Throttle(5, 60) // 5 requests per minute
  async expensiveOperation() {
    // ...
  }

  @Get('data')
  @SkipThrottle() // Skip rate limiting
  async getData() {
    // ...
  }
}
```

**2. IP-Based Rate Limiting:**

```typescript
// guards/ip-rate-limit.guard.ts
@Injectable()
export class IpRateLimitGuard implements CanActivate {
  constructor(
    @Inject('REDIS_CLIENT') private redis: Redis,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const ip = this.getClientIp(request);

    // Use sliding window algorithm
    const key = `rate_limit:ip:${ip}`;
    const now = Date.now();
    const windowSize = 60000; // 1 minute
    const maxRequests = 100;

    // Add current request
    await this.redis.zadd(key, now, `${now}-${Math.random()}`);

    // Remove old entries
    await this.redis.zremrangebyscore(key, 0, now - windowSize);

    // Count requests in window
    const count = await this.redis.zcard(key);

    // Set expiry
    await this.redis.expire(key, 60);

    if (count > maxRequests) {
      throw new HttpException(
        'Too many requests from this IP',
        HttpStatus.TOO_MANY_REQUESTS,
      );
    }

    return true;
  }

  private getClientIp(request: any): string {
    return (
      request.headers['x-forwarded-for']?.split(',')[0] ||
      request.headers['x-real-ip'] ||
      request.connection.remoteAddress
    );
  }
}
```

**3. Request Size Limits:**

```typescript
// main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Limit request body size
  app.use(express.json({ limit: '10mb' }));
  app.use(express.urlencoded({ extended: true, limit: '10mb' }));

  // Limit file upload size
  app.use(express.raw({ limit: '50mb', type: 'application/octet-stream' }));

  await app.listen(3000);
}
```

**4. Connection Limits:**

```typescript
// middleware/connection-limit.middleware.ts
@Injectable()
export class ConnectionLimitMiddleware implements NestMiddleware {
  private activeConnections = 0;
  private maxConnections = 10000;

  use(req: Request, res: Response, next: NextFunction) {
    if (this.activeConnections >= this.maxConnections) {
      return res.status(503).json({
        error: 'Service temporarily unavailable',
        message: 'Too many connections',
      });
    }

    this.activeConnections++;

    res.on('finish', () => {
      this.activeConnections--;
    });

    next();
  }
}
```

**5. Slowloris Protection (Timeout):**

```typescript
// main.ts
import helmet from 'helmet';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Security headers
  app.use(helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        scriptSrc: ["'self'", "'unsafe-inline'"],
      },
    },
    hsts: {
      maxAge: 31536000,
      includeSubDomains: true,
    },
  }));

  // Request timeout
  app.use((req, res, next) => {
    req.setTimeout(30000, () => {
      res.status(408).json({ error: 'Request timeout' });
    });
    res.setTimeout(30000, () => {
      res.status(408).json({ error: 'Response timeout' });
    });
    next();
  });

  await app.listen(3000);
}
```

**6. CAPTCHA for Suspicious Activity:**

```typescript
// guards/captcha.guard.ts
import axios from 'axios';

@Injectable()
export class CaptchaGuard implements CanActivate {
  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const captchaToken = request.body.captchaToken;

    if (!captchaToken) {
      throw new BadRequestException('CAPTCHA token required');
    }

    // Verify with Google reCAPTCHA
    const response = await axios.post(
      'https://www.google.com/recaptcha/api/siteverify',
      null,
      {
        params: {
          secret: process.env.RECAPTCHA_SECRET,
          response: captchaToken,
        },
      },
    );

    if (!response.data.success || response.data.score < 0.5) {
      throw new ForbiddenException('CAPTCHA verification failed');
    }

    return true;
  }
}

// Apply to sensitive endpoints
@Controller('auth')
export class AuthController {
  @Post('login')
  @UseGuards(CaptchaGuard)
  async login(@Body() dto: LoginDto) {
    // ...
  }
}
```

**7. nginx Configuration for DDoS Protection:**

```nginx
# nginx.conf
http {
    # Limit connections per IP
    limit_conn_zone $binary_remote_addr zone=addr:10m;
    limit_conn addr 10;

    # Limit requests per IP
    limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;
    limit_req zone=one burst=20 nodelay;

    # Request body size limit
    client_max_body_size 10m;

    # Timeout settings
    client_body_timeout 10s;
    client_header_timeout 10s;
    keepalive_timeout 65s;
    send_timeout 10s;

    # Buffer size limits
    client_body_buffer_size 128k;
    client_header_buffer_size 1k;
    large_client_header_buffers 4 4k;

    upstream nestjs_backend {
        least_conn;
        server nestjs-1:3000;
        server nestjs-2:3000;
        server nestjs-3:3000;
    }

    server {
        listen 80;

        # Block common DDoS patterns
        location ~ /\. {
            deny all;
        }

        location ~* \.(sql|bak|old|conf|log)$ {
            deny all;
        }

        # Geographic filtering (optional)
        # geo $block_country {
        #     default 0;
        #     CN 1;  # Block China
        #     RU 1;  # Block Russia
        # }
        # if ($block_country) {
        #     return 403;
        # }

        location / {
            limit_req zone=one burst=20 nodelay;
            proxy_pass http://nestjs_backend;
            
            # Headers
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            
            # Timeout
            proxy_read_timeout 60s;
            proxy_connect_timeout 60s;
        }
    }
}
```

**8. CloudFlare DDoS Protection:**

```typescript
// Enable CloudFlare as reverse proxy
// CloudFlare provides:
// - Automatic DDoS mitigation
// - WAF (Web Application Firewall)
// - Rate limiting at edge
// - Bot detection
// - Geographic restrictions

// Verify CloudFlare headers
@Injectable()
export class CloudFlareMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    const cfRay = req.headers['cf-ray'];
    const cfConnectingIp = req.headers['cf-connecting-ip'];

    // Ensure request came through CloudFlare
    if (!cfRay && process.env.NODE_ENV === 'production') {
      return res.status(403).json({
        error: 'Direct access not allowed',
      });
    }

    // Use CloudFlare's real IP
    if (cfConnectingIp) {
      req['ip'] = cfConnectingIp;
    }

    next();
  }
}
```

**9. Anomaly Detection:**

```typescript
// Detect and block suspicious patterns

@Injectable()
export class AnomalyDetectionService {
  constructor(@Inject('REDIS_CLIENT') private redis: Redis) {}

  async checkAnomalous(ip: string, userId?: string): Promise<boolean> {
    const key = `anomaly:${ip}`;
    const score = await this.calculateAnomalyScore(ip, userId);

    if (score > 80) {
      // High anomaly score - likely attack
      await this.redis.setex(`blocked:${ip}`, 3600, '1'); // Block for 1 hour
      return true;
    }

    return false;
  }

  private async calculateAnomalyScore(ip: string, userId?: string): Promise<number> {
    let score = 0;

    // Check request rate
    const requestRate = await this.getRequestRate(ip);
    if (requestRate > 100) score += 30;

    // Check failed auth attempts
    const failedAuths = await this.redis.get(`failed_auth:${ip}`);
    if (parseInt(failedAuths || '0') > 5) score += 40;

    // Check user agent
    // Missing or suspicious user agent
    score += 10;

    // Check request patterns
    // Sequential resource enumeration
    const isEnumerating = await this.checkEnumeration(ip);
    if (isEnumerating) score += 20;

    return score;
  }

  private async getRequestRate(ip: string): Promise<number> {
    const key = `requests:${ip}`;
    return parseInt(await this.redis.get(key) || '0');
  }

  private async checkEnumeration(ip: string): Promise<boolean> {
    // Check if IP is sequentially trying different resource IDs
    return false; // Simplified
  }
}

@Injectable()
export class AnomalyGuard implements CanActivate {
  constructor(private anomalyService: AnomalyDetectionService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const ip = request.ip;

    const isAnomalous = await this.anomalyService.checkAnomalous(
      ip,
      request.user?.id,
    );

    if (isAnomalous) {
      throw new ForbiddenException('Suspicious activity detected');
    }

    return true;
  }
}
```

**10. Kubernetes Network Policies:**

```yaml
# k8s/network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-network-policy
spec:
  podSelector:
    matchLabels:
      app: nestjs-api
  policyTypes:
    - Ingress
    - Egress
  ingress:
    # Allow from load balancer only
    - from:
      - namespaceSelector:
          matchLabels:
            name: ingress-nginx
      ports:
        - protocol: TCP
          port: 3000
  egress:
    # Allow to database
    - to:
      - podSelector:
          matchLabels:
            app: postgres
      ports:
        - protocol: TCP
          port: 5432
    # Allow to Redis
    - to:
      - podSelector:
          matchLabels:
            app: redis
      ports:
        - protocol: TCP
          port: 6379
    # Allow DNS
    - to:
      - namespaceSelector: {}
      ports:
        - protocol: UDP
          port: 53
```

**Key Takeaway:** DDoS protection requires **multiple layers**: **Application** (@nestjs/throttler rate limiting, IP-based limits with Redis sliding window, request size limits, connection limits), **Nginx** (limit_req_zone, limit_conn_zone, timeout settings, buffer limits), **CloudFlare** (automatic DDoS mitigation, WAF, bot detection at edge), **CAPTCHA** (for suspicious activity, reCAPTCHA score >0.5), **Anomaly detection** (track request patterns, failed auth, user agents, block high anomaly score), **Kubernetes** (NetworkPolicy to restrict traffic sources). Implement rate limiting: 100 req/min per IP, 10 concurrent connections per IP. Timeout slow requests (30s). Monitor for patterns (enumeration, brute force). Block temporarily on detection (1 hour). Critical for availability under attack.

</details>

<details>
<summary><strong>46. How do you implement API rate limiting per user?</strong></summary>

**Answer:**

Already covered in Q35 (Idempotency) and Q31 (API Gateway), but here's user-specific implementation:

**1. User-Based Rate Limiting:**

```typescript
// guards/user-rate-limit.guard.ts
@Injectable()
export class UserRateLimitGuard implements CanActivate {
  constructor(@Inject('REDIS_CLIENT') private redis: Redis) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const userId = request.user?.id;

    if (!userId) {
      // Anonymous users - use IP-based limiting
      return this.checkIpRateLimit(request.ip);
    }

    // Get user's rate limit tier
    const tier = request.user.tier || 'free';
    const limits = this.getLimitsForTier(tier);

    const key = `rate_limit:user:${userId}`;
    const now = Date.now();
    const windowSize = limits.windowMs;

    // Sliding window with Redis sorted set
    await this.redis.zadd(key, now, `${now}-${Math.random()}`);
    await this.redis.zremrangebyscore(key, 0, now - windowSize);
    const count = await this.redis.zcard(key);
    await this.redis.expire(key, Math.ceil(windowSize / 1000));

    if (count > limits.maxRequests) {
      const resetTime = now + windowSize;
      throw new HttpException(
        {
          statusCode: 429,
          message: 'Rate limit exceeded',
          retryAfter: resetTime,
        },
        HttpStatus.TOO_MANY_REQUESTS,
      );
    }

    // Add rate limit headers
    const response = context.switchToHttp().getResponse();
    response.setHeader('X-RateLimit-Limit', limits.maxRequests);
    response.setHeader('X-RateLimit-Remaining', limits.maxRequests - count);
    response.setHeader('X-RateLimit-Reset', new Date(now + windowSize).toISOString());

    return true;
  }

  private getLimitsForTier(tier: string) {
    const limits = {
      free: { maxRequests: 100, windowMs: 60000 }, // 100/min
      basic: { maxRequests: 1000, windowMs: 60000 }, // 1000/min
      premium: { maxRequests: 10000, windowMs: 60000 }, // 10000/min
      enterprise: { maxRequests: 100000, windowMs: 60000 }, // 100000/min
    };

    return limits[tier] || limits.free;
  }

  private async checkIpRateLimit(ip: string): Promise<boolean> {
    const key = `rate_limit:ip:${ip}`;
    const count = await this.redis.incr(key);
    
    if (count === 1) {
      await this.redis.expire(key, 60);
    }

    if (count > 60) {
      throw new HttpException(
        'Rate limit exceeded for anonymous users',
        HttpStatus.TOO_MANY_REQUESTS,
      );
    }

    return true;
  }
}

// Apply globally or per controller
@Controller('api')
@UseGuards(JwtAuthGuard, UserRateLimitGuard)
export class ApiController {
  // All endpoints protected by user-based rate limiting
}
```

**2. Endpoint-Specific Rate Limits:**

```typescript
// decorators/rate-limit.decorator.ts
export const RATE_LIMIT_KEY = 'rate_limit';

export function RateLimit(maxRequests: number, windowMs: number) {
  return SetMetadata(RATE_LIMIT_KEY, { maxRequests, windowMs });
}

// guards/custom-rate-limit.guard.ts
@Injectable()
export class CustomRateLimitGuard implements CanActivate {
  constructor(
    private reflector: Reflector,
    @Inject('REDIS_CLIENT') private redis: Redis,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const limits = this.reflector.get<any>(
      RATE_LIMIT_KEY,
      context.getHandler(),
    );

    if (!limits) {
      return true; // No custom limit set
    }

    const request = context.switchToHttp().getRequest();
    const userId = request.user?.id || request.ip;
    const endpoint = `${request.method}:${request.route.path}`;

    const key = `rate_limit:${userId}:${endpoint}`;
    const count = await this.redis.incr(key);

    if (count === 1) {
      await this.redis.pexpire(key, limits.windowMs);
    }

    if (count > limits.maxRequests) {
      throw new HttpException(
        `Rate limit exceeded: ${limits.maxRequests} requests per ${limits.windowMs}ms`,
        HttpStatus.TOO_MANY_REQUESTS,
      );
    }

    return true;
  }
}

// Usage
@Controller('api')
export class ApiController {
  @Post('expensive-operation')
  @UseGuards(JwtAuthGuard, CustomRateLimitGuard)
  @RateLimit(5, 60000) // 5 requests per minute
  async expensiveOperation() {
    // ...
  }

  @Get('data')
  @UseGuards(JwtAuthGuard, CustomRateLimitGuard)
  @RateLimit(100, 60000) // 100 requests per minute
  async getData() {
    // ...
  }
}
```

**3. Token Bucket Algorithm:**

```typescript
// rate-limit/token-bucket.service.ts
@Injectable()
export class TokenBucketService {
  constructor(@Inject('REDIS_CLIENT') private redis: Redis) {}

  async consumeToken(
    userId: string,
    capacity: number = 10,
    refillRate: number = 1, // tokens per second
  ): Promise<boolean> {
    const key = `token_bucket:${userId}`;
    const now = Date.now() / 1000; // seconds

    // Get current bucket state
    const result = await this.redis.hgetall(key);
    let tokens = parseFloat(result.tokens || capacity.toString());
    let lastRefill = parseFloat(result.lastRefill || now.toString());

    // Refill tokens based on time elapsed
    const elapsed = now - lastRefill;
    const newTokens = Math.min(capacity, tokens + elapsed * refillRate);

    // Try to consume a token
    if (newTokens >= 1) {
      await this.redis.hset(key, {
        tokens: (newTokens - 1).toString(),
        lastRefill: now.toString(),
      });
      await this.redis.expire(key, 3600);
      return true;
    }

    // No tokens available
    await this.redis.hset(key, {
      tokens: newTokens.toString(),
      lastRefill: now.toString(),
    });
    await this.redis.expire(key, 3600);
    return false;
  }
}

@Injectable()
export class TokenBucketGuard implements CanActivate {
  constructor(private tokenBucket: TokenBucketService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const userId = request.user?.id;

    if (!userId) {
      return true; // Skip for unauthenticated
    }

    const allowed = await this.tokenBucket.consumeToken(userId);

    if (!allowed) {
      throw new HttpException(
        'Rate limit exceeded - no tokens available',
        HttpStatus.TOO_MANY_REQUESTS,
      );
    }

    return true;
  }
}
```

**Key Takeaway:** User-based rate limiting ties limits to authenticated users, not just IPs. Implement **tier-based limits** (free: 100/min, premium: 10000/min) with Redis sorted sets for sliding window. Add **rate limit headers** (X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset). Support **endpoint-specific limits** with @RateLimit decorator. Use **token bucket** for smooth rate limiting (refill at steady rate). Anonymous users fall back to IP-based limiting. Already covered comprehensively in previous questions - see Q31 (API Gateway) for multi-tier implementation.

</details>

<details>
<summary><strong>47. How do you handle security in microservices?</strong></summary>

**Answer:**

Implement comprehensive security across microservices architecture:

**1. Service-to-Service Authentication (JWT):**

```typescript
// auth/service-auth.service.ts
@Injectable()
export class ServiceAuthService {
  constructor(private jwtService: JwtService) {}

  generateServiceToken(serviceName: string): string {
    return this.jwtService.sign(
      {
        sub: serviceName,
        type: 'service',
        permissions: this.getServicePermissions(serviceName),
      },
      {
        secret: process.env.SERVICE_JWT_SECRET,
        expiresIn: '1h',
      },
    );
  }

  verifyServiceToken(token: string): any {
    try {
      return this.jwtService.verify(token, {
        secret: process.env.SERVICE_JWT_SECRET,
      });
    } catch (error) {
      throw new UnauthorizedException('Invalid service token');
    }
  }

  private getServicePermissions(serviceName: string): string[] {
    const permissions = {
      'order-service': ['read:orders', 'write:orders', 'read:products'],
      'payment-service': ['read:payments', 'write:payments', 'read:orders'],
      'inventory-service': ['read:inventory', 'write:inventory'],
    };

    return permissions[serviceName] || [];
  }
}

// guards/service-auth.guard.ts
@Injectable()
export class ServiceAuthGuard implements CanActivate {
  constructor(private serviceAuth: ServiceAuthService) {}

  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const token = this.extractToken(request);

    if (!token) {
      throw new UnauthorizedException('Service token required');
    }

    const payload = this.serviceAuth.verifyServiceToken(token);
    
    if (payload.type !== 'service') {
      throw new UnauthorizedException('Invalid token type');
    }

    request.service = payload;
    return true;
  }

  private extractToken(request: any): string | null {
    const authHeader = request.headers['x-service-token'];
    return authHeader || null;
  }
}
```

**2. mTLS (Mutual TLS) for Service Communication:**

```typescript
// For HTTPS with client certificates

// Generate certificates
// openssl req -x509 -newkey rsa:4096 -keyout service-key.pem -out service-cert.pem -days 365 -nodes

// main.ts
import * as https from 'https';
import * as fs from 'fs';

async function bootstrap() {
  const httpsOptions = {
    key: fs.readFileSync('./certs/service-key.pem'),
    cert: fs.readFileSync('./certs/service-cert.pem'),
    ca: fs.readFileSync('./certs/ca-cert.pem'),
    requestCert: true, // Require client certificate
    rejectUnauthorized: true, // Reject invalid certificates
  };

  const app = await NestFactory.create(AppModule, {
    httpsOptions,
  });

  await app.listen(3000);
}

// HTTP client with mTLS
@Injectable()
export class SecureHttpService {
  async callService(url: string, data: any) {
    const agent = new https.Agent({
      cert: fs.readFileSync('./certs/client-cert.pem'),
      key: fs.readFileSync('./certs/client-key.pem'),
      ca: fs.readFileSync('./certs/ca-cert.pem'),
    });

    const response = await axios.post(url, data, {
      httpsAgent: agent,
    });

    return response.data;
  }
}
```

**3. API Gateway as Security Perimeter:**

```typescript
// API Gateway handles authentication/authorization for all services

@Injectable()
export class GatewayAuthGuard implements CanActivate {
  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();

    // 1. Authenticate user
    const user = await this.authenticateUser(request);
    request.user = user;

    // 2. Check authorization for requested service
    const targetService = this.getTargetService(request);
    if (!this.hasPermission(user, targetService)) {
      throw new ForbiddenException('Access denied to service');
    }

    // 3. Generate service token for internal communication
    const serviceToken = this.generateServiceToken(user, targetService);
    request.headers['x-service-token'] = serviceToken;

    return true;
  }

  private async authenticateUser(request: any) {
    const token = request.headers['authorization']?.replace('Bearer ', '');
    if (!token) {
      throw new UnauthorizedException('Missing token');
    }
    
    return this.jwtService.verify(token);
  }

  private getTargetService(request: any): string {
    // Parse from URL: /api/orders -> order-service
    const path = request.url.split('/')[2];
    return `${path}-service`;
  }

  private hasPermission(user: any, service: string): boolean {
    const requiredRoles = {
      'order-service': ['user', 'admin'],
      'admin-service': ['admin'],
    };

    const required = requiredRoles[service] || [];
    return required.includes(user.role);
  }

  private generateServiceToken(user: any, service: string): string {
    return this.serviceAuth.generateServiceToken(service);
  }
}
```

**4. Service Mesh (Istio):**

```yaml
# istio/peer-authentication.yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: default
spec:
  mtls:
    mode: STRICT  # Enforce mTLS between all services

---
# Authorization policy
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: order-service-policy
  namespace: default
spec:
  selector:
    matchLabels:
      app: order-service
  action: ALLOW
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/default/sa/api-gateway"]
    to:
    - operation:
        methods: ["GET", "POST"]
        paths: ["/api/orders/*"]
```

**5. Secrets Management:**

```typescript
// Use external secrets manager (Vault, AWS Secrets Manager)

@Injectable()
export class SecretsService {
  private vault: any;

  constructor() {
    this.vault = require('node-vault')({
      apiVersion: 'v1',
      endpoint: process.env.VAULT_ADDR,
      token: process.env.VAULT_TOKEN,
    });
  }

  async getSecret(path: string): Promise<any> {
    try {
      const result = await this.vault.read(path);
      return result.data.data;
    } catch (error) {
      throw new Error(`Failed to fetch secret: ${error.message}`);
    }
  }

  async getDatabaseCredentials(): Promise<any> {
    return this.getSecret('secret/data/database');
  }

  async getApiKey(service: string): Promise<string> {
    const secrets = await this.getSecret(`secret/data/api-keys/${service}`);
    return secrets.key;
  }
}

// Usage
@Injectable()
export class DatabaseService {
  constructor(private secrets: SecretsService) {}

  async connect() {
    const credentials = await this.secrets.getDatabaseCredentials();
    
    return createConnection({
      type: 'postgres',
      host: credentials.host,
      username: credentials.username,
      password: credentials.password,
      database: credentials.database,
    });
  }
}
```

**6. Input Validation Across Services:**

```typescript
// Validate all inputs at service boundaries

@Injectable()
export class ValidationPipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    // Sanitize inputs
    if (typeof value === 'string') {
      value = this.sanitize(value);
    }

    // Validate with class-validator
    return value;
  }

  private sanitize(input: string): string {
    // Remove SQL injection attempts
    input = input.replace(/['";\\]/g, '');
    
    // Remove XSS attempts
    input = input.replace(/<script[^>]*>.*?<\/script>/gi, '');
    
    // Limit length
    if (input.length > 10000) {
      throw new BadRequestException('Input too long');
    }

    return input;
  }
}

// DTOs with validation
export class CreateOrderDto {
  @IsNotEmpty()
  @IsUUID()
  userId: string;

  @IsNotEmpty()
  @IsArray()
  @ValidateNested({ each: true })
  @Type(() => OrderItemDto)
  items: OrderItemDto[];

  @IsNumber()
  @Min(0)
  @Max(1000000)
  total: number;
}
```

**7. Audit Logging:**

```typescript
// Log all security-relevant events

@Injectable()
export class SecurityAuditService {
  constructor(private logger: LoggerService) {}

  logAuthAttempt(userId: string, success: boolean, ip: string) {
    this.logger.log('Authentication Attempt', 'SecurityAudit', {
      event: 'auth.attempt',
      userId,
      success,
      ip,
      timestamp: new Date().toISOString(),
    });
  }

  logServiceAccess(sourceService: string, targetService: string, operation: string) {
    this.logger.log('Service Access', 'SecurityAudit', {
      event: 'service.access',
      source: sourceService,
      target: targetService,
      operation,
      timestamp: new Date().toISOString(),
    });
  }

  logPermissionDenied(userId: string, resource: string, action: string) {
    this.logger.warn('Permission Denied', 'SecurityAudit', {
      event: 'permission.denied',
      userId,
      resource,
      action,
      timestamp: new Date().toISOString(),
    });
  }
}
```

**Key Takeaway:** Microservices security requires **service-to-service authentication** (JWT with x-service-token header, mTLS with client certificates), **API Gateway as security perimeter** (authenticate users, authorize service access, generate service tokens), **Service Mesh** (Istio with STRICT mTLS mode, AuthorizationPolicy for fine-grained access control), **Secrets management** (Vault/AWS Secrets Manager, never hardcode credentials), **Input validation** at all service boundaries (class-validator, sanitize inputs), **Audit logging** (auth attempts, service access, permission denied). Each service validates incoming tokens. Use least privilege principle (service only accesses what it needs). Critical for zero-trust architecture.

</details>

<details>
<summary><strong>48. How do you secure inter-service communication?</strong></summary>

**Answer:**

Secure communication between microservices using multiple strategies:

**1. Service Tokens (JWT-Based):**

```typescript
// http-client/secure-http.service.ts
@Injectable()
export class SecureHttpService {
  constructor(
    private httpService: HttpService,
    private serviceAuth: ServiceAuthService,
  ) {}

  async post(url: string, data: any): Promise<any> {
    // Generate service token
    const serviceName = process.env.SERVICE_NAME || 'api-gateway';
    const token = this.serviceAuth.generateServiceToken(serviceName);

    const response = await firstValueFrom(
      this.httpService.post(url, data, {
        headers: {
          'x-service-token': token,
          'x-request-id': uuid(),
        },
      }),
    );

    return response.data;
  }

  async get(url: string): Promise<any> {
    const serviceName = process.env.SERVICE_NAME || 'api-gateway';
    const token = this.serviceAuth.generateServiceToken(serviceName);

    const response = await firstValueFrom(
      this.httpService.get(url, {
        headers: {
          'x-service-token': token,
        },
      }),
    );

    return response.data;
  }
}

// Verify service token in receiving service
@Injectable()
export class ServiceTokenInterceptor implements NestInterceptor {
  constructor(private serviceAuth: ServiceAuthService) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const token = request.headers['x-service-token'];

    if (!token) {
      throw new UnauthorizedException('Service token required');
    }

    try {
      const payload = this.serviceAuth.verifyServiceToken(token);
      request.service = payload;
    } catch (error) {
      throw new UnauthorizedException('Invalid service token');
    }

    return next.handle();
  }
}
```

**2. Message Queue Security (RabbitMQ):**

```typescript
// Secure message queue communication

// Producer
@Injectable()
export class SecureMessageService {
  constructor(
    @Inject('RABBITMQ_CLIENT') private client: ClientProxy,
    private serviceAuth: ServiceAuthService,
  ) {}

  async sendSecureMessage(pattern: string, data: any) {
    const serviceName = process.env.SERVICE_NAME;
    const token = this.serviceAuth.generateServiceToken(serviceName);

    // Include token in message
    const secureMessage = {
      _serviceToken: token,
      _timestamp: Date.now(),
      data,
    };

    return firstValueFrom(
      this.client.send(pattern, secureMessage),
    );
  }
}

// Consumer
@Controller()
export class SecureMessageController {
  constructor(private serviceAuth: ServiceAuthService) {}

  @MessagePattern('process_payment')
  async processPayment(@Payload() message: any) {
    // Verify service token
    const token = message._serviceToken;
    if (!token) {
      throw new UnauthorizedException('Service token required');
    }

    const payload = this.serviceAuth.verifyServiceToken(token);

    // Check message freshness (prevent replay attacks)
    const messageAge = Date.now() - message._timestamp;
    if (messageAge > 60000) {
      throw new UnauthorizedException('Message too old');
    }

    // Process data
    return this.paymentService.process(message.data);
  }
}
```

**3. Network Segmentation (Kubernetes):**

```yaml
# Restrict network access between services

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: order-service-network-policy
spec:
  podSelector:
    matchLabels:
      app: order-service
  policyTypes:
    - Ingress
    - Egress
  ingress:
    # Only allow from API Gateway
    - from:
      - podSelector:
          matchLabels:
            app: api-gateway
      ports:
        - protocol: TCP
          port: 3000
  egress:
    # Allow to Payment Service
    - to:
      - podSelector:
          matchLabels:
            app: payment-service
      ports:
        - protocol: TCP
          port: 3001
    # Allow to Database
    - to:
      - podSelector:
          matchLabels:
            app: postgres
      ports:
        - protocol: TCP
          port: 5432
    # Allow DNS
    - to:
      - namespaceSelector: {}
      ports:
        - protocol: UDP
          port: 53
```

**4. Encrypted Messages:**

```typescript
// Encrypt sensitive data in messages

@Injectable()
export class MessageEncryptionService {
  private algorithm = 'aes-256-gcm';
  private key = Buffer.from(process.env.ENCRYPTION_KEY, 'hex'); // 32 bytes

  encrypt(data: any): { encrypted: string; iv: string; tag: string } {
    const iv = crypto.randomBytes(16);
    const cipher = crypto.createCipheriv(this.algorithm, this.key, iv);

    const text = JSON.stringify(data);
    let encrypted = cipher.update(text, 'utf8', 'hex');
    encrypted += cipher.final('hex');

    const tag = cipher.getAuthTag();

    return {
      encrypted,
      iv: iv.toString('hex'),
      tag: tag.toString('hex'),
    };
  }

  decrypt(encrypted: string, iv: string, tag: string): any {
    const decipher = crypto.createDecipheriv(
      this.algorithm,
      this.key,
      Buffer.from(iv, 'hex'),
    );

    decipher.setAuthTag(Buffer.from(tag, 'hex'));

    let decrypted = decipher.update(encrypted, 'hex', 'utf8');
    decrypted += decipher.final('utf8');

    return JSON.parse(decrypted);
  }
}

// Usage
@Injectable()
export class PaymentService {
  constructor(
    @Inject('PAYMENT_CLIENT') private paymentClient: ClientProxy,
    private encryption: MessageEncryptionService,
  ) {}

  async processPayment(paymentData: any) {
    // Encrypt sensitive payment data
    const encrypted = this.encryption.encrypt(paymentData);

    return firstValueFrom(
      this.paymentClient.send('process_payment', encrypted),
    );
  }
}

@Controller()
export class PaymentController {
  constructor(private encryption: MessageEncryptionService) {}

  @MessagePattern('process_payment')
  async handlePayment(@Payload() encryptedData: any) {
    // Decrypt message
    const paymentData = this.encryption.decrypt(
      encryptedData.encrypted,
      encryptedData.iv,
      encryptedData.tag,
    );

    return this.paymentService.process(paymentData);
  }
}
```

**5. IP Whitelisting:**

```typescript
// Only allow connections from known service IPs

@Injectable()
export class IpWhitelistGuard implements CanActivate {
  private allowedIps = [
    '10.0.1.10', // order-service
    '10.0.1.11', // payment-service
    '10.0.1.12', // inventory-service
  ];

  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const clientIp = request.ip || request.connection.remoteAddress;

    if (!this.allowedIps.includes(clientIp)) {
      throw new ForbiddenException('IP not whitelisted');
    }

    return true;
  }
}

// Apply to internal endpoints
@Controller('internal')
@UseGuards(IpWhitelistGuard)
export class InternalController {
  @Post('sync-inventory')
  async syncInventory() {
    // Only accessible from whitelisted IPs
  }
}
```

**6. Request Signing:**

```typescript
// Sign requests to verify integrity

@Injectable()
export class RequestSigningService {
  private secret = process.env.SIGNING_SECRET;

  signRequest(data: any): string {
    const payload = JSON.stringify(data);
    return crypto
      .createHmac('sha256', this.secret)
      .update(payload)
      .digest('hex');
  }

  verifySignature(data: any, signature: string): boolean {
    const expectedSignature = this.signRequest(data);
    return crypto.timingSafeEqual(
      Buffer.from(signature),
      Buffer.from(expectedSignature),
    );
  }
}

// HTTP client
@Injectable()
export class SignedHttpService {
  constructor(
    private httpService: HttpService,
    private signing: RequestSigningService,
  ) {}

  async post(url: string, data: any) {
    const signature = this.signing.signRequest(data);

    return firstValueFrom(
      this.httpService.post(url, data, {
        headers: {
          'x-signature': signature,
        },
      }),
    );
  }
}

// Verify signature
@Injectable()
export class SignatureVerificationInterceptor implements NestInterceptor {
  constructor(private signing: RequestSigningService) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const signature = request.headers['x-signature'];

    if (!signature) {
      throw new UnauthorizedException('Signature required');
    }

    if (!this.signing.verifySignature(request.body, signature)) {
      throw new UnauthorizedException('Invalid signature');
    }

    return next.handle();
  }
}
```

**Key Takeaway:** Secure inter-service communication with: **Service tokens** (JWT in x-service-token header, verify on each request), **Message encryption** (AES-256-GCM for sensitive data, iv + tag for authentication), **Network policies** (Kubernetes NetworkPolicy restricts pod-to-pod traffic), **Request signing** (HMAC-SHA256 signature in x-signature header, verify integrity), **IP whitelisting** (only allow known service IPs), **Message freshness** (reject messages older than 60s, prevent replay attacks), **mTLS** (mutual certificate authentication). For RabbitMQ: include service token in message payload. Service Mesh (Istio) handles mTLS automatically. Never trust internal network - assume breach. Critical for zero-trust microservices.

</details>

## Deployment & CI/CD

<details>
<summary><strong>49. How do you containerize a NestJS application using Docker?</strong></summary>

**Answer:**

Create optimized Docker images for NestJS applications with multi-stage builds:

**1. Multi-Stage Dockerfile:**

```dockerfile
# Dockerfile

# Stage 1: Build
FROM node:18-alpine AS builder

# Install build dependencies
RUN apk add --no-cache python3 make g++

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production && \
    npm cache clean --force

# Copy source code
COPY . .

# Build application
RUN npm run build

# Stage 2: Production
FROM node:18-alpine

# Install dumb-init for proper signal handling
RUN apk add --no-cache dumb-init

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nestjs -u 1001

WORKDIR /app

# Copy built app from builder
COPY --from=builder --chown=nestjs:nodejs /app/dist ./dist
COPY --from=builder --chown=nestjs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nestjs:nodejs /app/package*.json ./

# Switch to non-root user
USER nestjs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
    CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

# Use dumb-init to handle signals properly
ENTRYPOINT ["dumb-init", "--"]

# Start application
CMD ["node", "dist/main.js"]
```

**2. .dockerignore File:**

```
# .dockerignore
node_modules
dist
npm-debug.log
.env
.env.*
.git
.gitignore
README.md
.vscode
coverage
.nyc_output
*.log
```

**3. Docker Compose for Development:**

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - DB_HOST=postgres
      - DB_PORT=5432
      - DB_USERNAME=nestjs
      - DB_PASSWORD=password
      - DB_DATABASE=nestjs
      - REDIS_HOST=redis
      - REDIS_PORT=6379
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    volumes:
      - ./src:/app/src  # Hot reload in development
    command: npm run start:dev

  postgres:
    image: postgres:15-alpine
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_USER=nestjs
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=nestjs
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U nestjs"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    command: redis-server --appendonly yes

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - app

volumes:
  postgres-data:
  redis-data:
```

**4. Production Docker Compose:**

```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  app:
    image: nestjs-app:${VERSION:-latest}
    deploy:
      replicas: 3
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
      resources:
        limits:
          cpus: '1'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M
    environment:
      - NODE_ENV=production
      - DB_HOST=${DB_HOST}
      - DB_PASSWORD=${DB_PASSWORD}
    secrets:
      - db_password
      - jwt_secret
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

secrets:
  db_password:
    external: true
  jwt_secret:
    external: true
```

**5. Kubernetes Deployment:**

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nestjs-app
  labels:
    app: nestjs-app
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
        image: your-registry/nestjs-app:latest
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: db-host
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: db-password
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health/live
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 2
        lifecycle:
          preStop:
            exec:
              command: ["/bin/sh", "-c", "sleep 15"]

---
apiVersion: v1
kind: Service
metadata:
  name: nestjs-app
spec:
  selector:
    app: nestjs-app
  ports:
  - port: 80
    targetPort: 3000
  type: LoadBalancer

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  db-host: "postgres.default.svc.cluster.local"
  redis-host: "redis.default.svc.cluster.local"

---
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  db-password: <base64-encoded-password>
  jwt-secret: <base64-encoded-secret>
```

**6. Horizontal Pod Autoscaler:**

```yaml
# k8s/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nestjs-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nestjs-app
  minReplicas: 3
  maxReplicas: 50
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
      - type: Pods
        value: 2
        periodSeconds: 60
      selectPolicy: Max
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
```

**7. Build Script:**

```bash
#!/bin/bash
# build.sh

VERSION=${1:-latest}
REGISTRY=your-registry

# Build image
docker build -t $REGISTRY/nestjs-app:$VERSION .

# Tag as latest
docker tag $REGISTRY/nestjs-app:$VERSION $REGISTRY/nestjs-app:latest

# Push to registry
docker push $REGISTRY/nestjs-app:$VERSION
docker push $REGISTRY/nestjs-app:latest

echo "Built and pushed $REGISTRY/nestjs-app:$VERSION"
```

**8. Docker Security Best Practices:**

```dockerfile
# Security-hardened Dockerfile

FROM node:18-alpine AS builder

# Scan for vulnerabilities during build
RUN apk add --no-cache dumb-init && \
    npm install -g npm@latest

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production && \
    npm audit --production

COPY . .
RUN npm run build

FROM node:18-alpine

# Install security updates
RUN apk upgrade --no-cache && \
    apk add --no-cache dumb-init

# Create non-root user with specific UID/GID
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nestjs -u 1001 -G nodejs

WORKDIR /app

# Set proper permissions
COPY --from=builder --chown=nestjs:nodejs /app/dist ./dist
COPY --from=builder --chown=nestjs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nestjs:nodejs /app/package*.json ./

# Drop capabilities
RUN chmod -R 550 /app

USER nestjs

# Don't run as PID 1
ENTRYPOINT ["dumb-init", "--"]

CMD ["node", "dist/main.js"]

# Security labels
LABEL security.scan="true"
LABEL org.opencontainers.image.source="https://github.com/your-org/your-repo"
```

**Key Takeaway:** Containerize NestJS with **multi-stage Docker build** (builder stage installs deps + builds, production stage copies only dist/node_modules). Use **node:18-alpine** (smaller image, 30-50MB vs 1GB). Run as **non-root user** (nestjs:nodejs with UID 1001). Use **dumb-init** for proper signal handling (SIGTERM). Add **health checks** (GET /health every 30s). **Docker Compose** for local dev (postgres, redis, nginx). **Kubernetes** for production (Deployment with 3 replicas, HPA scales 3-50 based on CPU/memory, liveness/readiness probes, ConfigMap/Secrets for config). Implement **.dockerignore** (exclude node_modules, .git). Security: npm audit, apk upgrade, drop capabilities, proper permissions. Push to container registry with version tags.

</details>

<details>
<summary><strong>50. How do you implement CI/CD pipeline for NestJS?</strong></summary>

**Answer:**

Implement comprehensive CI/CD pipeline with automated testing, building, and deployment:

**1. GitHub Actions CI/CD:**

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  NODE_VERSION: '18'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  lint-and-test:
    name: Lint and Test
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

      redis:
        image: redis:7-alpine
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run unit tests
        run: npm run test:cov
        env:
          DB_HOST: localhost
          DB_PORT: 5432
          DB_USERNAME: test
          DB_PASSWORD: test
          DB_DATABASE: test
          REDIS_HOST: localhost
          REDIS_PORT: 6379

      - name: Run e2e tests
        run: npm run test:e2e
        env:
          DB_HOST: localhost
          DB_PORT: 5432
          DB_USERNAME: test
          DB_PASSWORD: test
          DB_DATABASE: test

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
          flags: unittests
          name: codecov-umbrella

      - name: SonarCloud Scan
        uses: SonarSource/sonarcloud-github-action@master
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

  security-scan:
    name: Security Scan
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Run npm audit
        run: npm audit --production

      - name: Run Snyk security scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high

  build-and-push:
    name: Build and Push Docker Image
    runs-on: ubuntu-latest
    needs: [lint-and-test, security-scan]
    if: github.ref == 'refs/heads/main'
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Log in to Container Registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=sha,prefix={{branch}}-
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}

      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: Scan image for vulnerabilities
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
          format: 'sarif'
          output: 'trivy-results.sarif'

      - name: Upload Trivy results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'

  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: build-and-push
    if: github.ref == 'refs/heads/main'
    environment:
      name: staging
      url: https://staging.example.com
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup kubectl
        uses: azure/setup-kubectl@v3

      - name: Configure kubectl
        run: |
          echo "${{ secrets.KUBECONFIG_STAGING }}" | base64 -d > kubeconfig
          export KUBECONFIG=kubeconfig

      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/nestjs-app \
            app=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }} \
            -n staging
          
          kubectl rollout status deployment/nestjs-app -n staging --timeout=5m

      - name: Run smoke tests
        run: |
          npm ci
          npm run test:smoke
        env:
          API_URL: https://staging.example.com

  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: deploy-staging
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://api.example.com
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup kubectl
        uses: azure/setup-kubectl@v3

      - name: Configure kubectl
        run: |
          echo "${{ secrets.KUBECONFIG_PRODUCTION }}" | base64 -d > kubeconfig
          export KUBECONFIG=kubeconfig

      - name: Blue-Green Deployment
        run: |
          # Deploy to green environment
          kubectl set image deployment/nestjs-app-green \
            app=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }} \
            -n production
          
          kubectl rollout status deployment/nestjs-app-green -n production --timeout=5m

          # Run health checks
          sleep 30
          curl -f https://green.api.example.com/health || exit 1

          # Switch traffic to green
          kubectl patch service nestjs-app -n production \
            -p '{"spec":{"selector":{"version":"green"}}}'

          # Wait and verify
          sleep 60

          # Scale down blue (old version)
          kubectl scale deployment/nestjs-app-blue --replicas=0 -n production

      - name: Notify Slack
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: 'Production deployment completed!'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
        if: always()
```

**2. GitLab CI/CD:**

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: "/certs"

test:
  stage: test
  image: node:18-alpine
  services:
    - postgres:15
    - redis:7-alpine
  variables:
    POSTGRES_DB: test
    POSTGRES_USER: test
    POSTGRES_PASSWORD: test
    DB_HOST: postgres
    REDIS_HOST: redis
  script:
    - npm ci
    - npm run lint
    - npm run test:cov
    - npm run test:e2e
  coverage: '/Statements\s*:\s*(\d+\.\d+)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

security:
  stage: test
  image: node:18-alpine
  script:
    - npm ci
    - npm audit --production
  allow_failure: true

build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE:latest
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker push $CI_REGISTRY_IMAGE:latest
  only:
    - main

deploy_staging:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl config use-context staging
    - kubectl set image deployment/nestjs-app app=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA -n staging
    - kubectl rollout status deployment/nestjs-app -n staging
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - main

deploy_production:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl config use-context production
    - kubectl set image deployment/nestjs-app app=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA -n production
    - kubectl rollout status deployment/nestjs-app -n production
  environment:
    name: production
    url: https://api.example.com
  when: manual
  only:
    - main
```

**3. Jenkins Pipeline:**

```groovy
// Jenkinsfile
pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'your-registry'
        IMAGE_NAME = 'nestjs-app'
        KUBECONFIG = credentials('kubeconfig')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }
        
        stage('Lint') {
            steps {
                sh 'npm run lint'
            }
        }
        
        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'npm run test:cov'
                    }
                }
                stage('E2E Tests') {
                    steps {
                        sh 'npm run test:e2e'
                    }
                }
            }
            post {
                always {
                    publishHTML([
                        reportDir: 'coverage',
                        reportFiles: 'index.html',
                        reportName: 'Coverage Report'
                    ])
                }
            }
        }
        
        stage('Security Scan') {
            steps {
                sh 'npm audit --production'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}")
                }
            }
        }
        
        stage('Push to Registry') {
            steps {
                script {
                    docker.withRegistry('https://your-registry', 'docker-credentials') {
                        docker.image("${DOCKER_REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}").push()
                        docker.image("${DOCKER_REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}").push('latest')
                    }
                }
            }
        }
        
        stage('Deploy to Staging') {
            steps {
                sh """
                    kubectl set image deployment/nestjs-app \
                        app=${DOCKER_REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER} \
                        -n staging
                    kubectl rollout status deployment/nestjs-app -n staging
                """
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to production?', ok: 'Deploy'
                sh """
                    kubectl set image deployment/nestjs-app \
                        app=${DOCKER_REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER} \
                        -n production
                    kubectl rollout status deployment/nestjs-app -n production
                """
            }
        }
    }
    
    post {
        success {
            slackSend color: 'good', message: "Build ${BUILD_NUMBER} succeeded!"
        }
        failure {
            slackSend color: 'danger', message: "Build ${BUILD_NUMBER} failed!"
        }
    }
}
```

**4. Database Migrations in CI/CD:**

```yaml
# migration-job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: run-migrations
spec:
  template:
    spec:
      containers:
      - name: migrations
        image: your-registry/nestjs-app:latest
        command: ["npm", "run", "migration:run"]
        env:
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: db-host
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: db-password
      restartPolicy: OnFailure
```

**5. Automated Rollback:**

```bash
#!/bin/bash
# rollback.sh

NAMESPACE=${1:-production}
DEPLOYMENT=nestjs-app

echo "Rolling back deployment in $NAMESPACE..."

# Undo last deployment
kubectl rollout undo deployment/$DEPLOYMENT -n $NAMESPACE

# Wait for rollback to complete
kubectl rollout status deployment/$DEPLOYMENT -n $NAMESPACE

# Verify health
HEALTH_URL="https://api.example.com/health"
RESPONSE=$(curl -s -o /dev/null -w "%{http_code}" $HEALTH_URL)

if [ $RESPONSE -eq 200 ]; then
    echo "Rollback successful! Health check passed."
else
    echo "Rollback completed but health check failed!"
    exit 1
fi
```

**Key Takeaway:** CI/CD pipeline stages: **Test** (lint, unit tests, e2e tests with postgres/redis services, coverage >80%), **Security** (npm audit, Snyk scan, Trivy image scan), **Build** (Docker multi-stage build, push to registry with SHA + latest tags), **Deploy Staging** (kubectl set image, rollout status, smoke tests), **Deploy Production** (manual approval, blue-green deployment, health checks, rollback on failure). Use **GitHub Actions** (matrix testing, environment protection rules), **GitLab CI** (auto DevOps), **Jenkins** (pipeline as code). Run **migrations** as Kubernetes Job before deployment. Implement **automated rollback** (kubectl rollout undo). Notify Slack/email on success/failure. Store secrets in CI/CD variables, not code. Critical for rapid, reliable releases.

</details>

---

## Congratulations! 🎉

You've completed all **50 System Design & Scenarios questions** for NestJS!

This comprehensive guide covers:
- ✅ **API Design & Real-World Applications** (Q1-Q18)
- ✅ **Scalability & Performance** (Q19-Q25)
- ✅ **Microservices Architecture** (Q26-Q31)
- ✅ **Data Consistency** (Q32-Q35)
- ✅ **Error Handling & Resilience** (Q36-Q40)
- ✅ **Monitoring & Observability** (Q41-Q44)
- ✅ **Security Scenarios** (Q45-Q48)
- ✅ **Deployment & CI/CD** (Q49-Q50)

Each question includes:
- Production-grade NestJS code examples
- Best practices and architectural patterns
- Configuration files (Docker, Kubernetes, nginx, etc.)
- Real-world considerations and trade-offs

**Use this guide to:**
- Prepare for senior/lead NestJS interviews
- Design scalable microservices architectures
- Implement enterprise-grade features
- Learn industry best practices

**Good luck with your interviews! 🚀**
