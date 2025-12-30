# Security Best Practices

## Table of Contents
1. [Security Principles](#security-principles)
2. [Authentication](#authentication)
3. [Authorization](#authorization)
4. [Data Protection](#data-protection)
5. [API Security](#api-security)
6. [Frontend Security](#frontend-security)
7. [Database Security](#database-security)
8. [Infrastructure Security](#infrastructure-security)
9. [Common Vulnerabilities](#common-vulnerabilities)
10. [Security Checklist](#security-checklist)

---

## Security Principles

### Defense in Depth
```
Multiple layers of security

┌────────────────────────────────────┐
│  Perimeter: Firewall, WAF          │
├────────────────────────────────────┤
│  Network: VPC, Security Groups     │
├────────────────────────────────────┤
│  Application: Authentication       │
├────────────────────────────────────┤
│  Data: Encryption, Validation      │
├────────────────────────────────────┤
│  Infrastructure: Updates, Patches  │
└────────────────────────────────────┘

If one layer fails, others still protect
```

### Principle of Least Privilege
```
Give minimum necessary permissions

Bad:
User: Admin access to everything
API: Full database access
Container: Root user

Good:
User: Read-only access to own data
API: Specific table permissions only
Container: Non-root user
```

### Security Mindset
```
Think like an attacker:
- What could go wrong?
- How could this be exploited?
- What's the worst case scenario?
- How do we detect attacks?
```

---

## Authentication

### Password Security

#### Hashing (Never Store Plain Text!)
```typescript
// NEVER do this!
❌ user.password = plainPassword;

// Always hash
import bcrypt from 'bcrypt';

export class AuthService {
  // Hash password before storing
  async hashPassword(password: string): Promise<string> {
    const saltRounds = 10;
    return bcrypt.hash(password, saltRounds);
  }

  // Verify password
  async verifyPassword(
    plainPassword: string,
    hashedPassword: string
  ): Promise<boolean> {
    return bcrypt.compare(plainPassword, hashedPassword);
  }
}

// Usage
const hashedPassword = await authService.hashPassword('MyPassword123!');
// Store: $2b$10$xyz... (never store plain password)

// Login
const isValid = await authService.verifyPassword(
  inputPassword,
  user.hashedPassword
);
```

#### Password Requirements
```typescript
// Validate password strength
export class PasswordValidator {
  validate(password: string): { valid: boolean; errors: string[] } {
    const errors: string[] = [];

    if (password.length < 8) {
      errors.push('Password must be at least 8 characters');
    }

    if (!/[A-Z]/.test(password)) {
      errors.push('Password must contain uppercase letter');
    }

    if (!/[a-z]/.test(password)) {
      errors.push('Password must contain lowercase letter');
    }

    if (!/[0-9]/.test(password)) {
      errors.push('Password must contain number');
    }

    if (!/[!@#$%^&*]/.test(password)) {
      errors.push('Password must contain special character');
    }

    // Check against common passwords
    const commonPasswords = ['password', '12345678', 'qwerty'];
    if (commonPasswords.includes(password.toLowerCase())) {
      errors.push('Password is too common');
    }

    return {
      valid: errors.length === 0,
      errors
    };
  }
}
```

### JWT (JSON Web Tokens)
```typescript
// auth.service.ts
import jwt from 'jsonwebtoken';

export class AuthService {
  private jwtSecret = process.env.JWT_SECRET!;
  private jwtRefreshSecret = process.env.JWT_REFRESH_SECRET!;

  // Generate access token (short-lived)
  generateAccessToken(userId: string): string {
    return jwt.sign(
      { userId, type: 'access' },
      this.jwtSecret,
      { expiresIn: '15m' } // 15 minutes
    );
  }

  // Generate refresh token (long-lived)
  generateRefreshToken(userId: string): string {
    return jwt.sign(
      { userId, type: 'refresh' },
      this.jwtRefreshSecret,
      { expiresIn: '7d' } // 7 days
    );
  }

  // Verify access token
  verifyAccessToken(token: string): { userId: string } {
    try {
      const payload = jwt.verify(token, this.jwtSecret);
      return payload as { userId: string };
    } catch (error) {
      throw new UnauthorizedException('Invalid token');
    }
  }

  // Refresh tokens
  async refreshTokens(refreshToken: string) {
    try {
      const payload = jwt.verify(refreshToken, this.jwtRefreshSecret);
      const userId = (payload as any).userId;

      // Check if refresh token is blacklisted
      const isBlacklisted = await this.isTokenBlacklisted(refreshToken);
      if (isBlacklisted) {
        throw new UnauthorizedException('Token revoked');
      }

      // Generate new tokens
      return {
        accessToken: this.generateAccessToken(userId),
        refreshToken: this.generateRefreshToken(userId)
      };
    } catch (error) {
      throw new UnauthorizedException('Invalid refresh token');
    }
  }
}
```

### Authentication Guard
```typescript
// auth.guard.ts
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Request } from 'express';

@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private authService: AuthService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest<Request>();
    
    // Extract token from header
    const token = this.extractToken(request);
    if (!token) {
      throw new UnauthorizedException('No token provided');
    }

    try {
      // Verify token
      const payload = this.authService.verifyAccessToken(token);
      
      // Attach user to request
      request['user'] = await this.getUserById(payload.userId);
      
      return true;
    } catch (error) {
      throw new UnauthorizedException('Invalid token');
    }
  }

  private extractToken(request: Request): string | null {
    const authHeader = request.headers.authorization;
    if (!authHeader || !authHeader.startsWith('Bearer ')) {
      return null;
    }
    return authHeader.substring(7);
  }
}

// Usage in controller
@Controller('users')
@UseGuards(AuthGuard)
export class UserController {
  @Get('profile')
  getProfile(@Request() req) {
    return req.user; // Authenticated user
  }
}
```

### OAuth 2.0 / Social Login
```typescript
// Google OAuth example
import { OAuth2Client } from 'google-auth-library';

export class GoogleAuthService {
  private client = new OAuth2Client(process.env.GOOGLE_CLIENT_ID);

  async verifyGoogleToken(token: string) {
    try {
      const ticket = await this.client.verifyIdToken({
        idToken: token,
        audience: process.env.GOOGLE_CLIENT_ID
      });

      const payload = ticket.getPayload();
      
      return {
        email: payload.email,
        name: payload.name,
        picture: payload.picture,
        googleId: payload.sub
      };
    } catch (error) {
      throw new UnauthorizedException('Invalid Google token');
    }
  }

  async loginWithGoogle(token: string) {
    const googleUser = await this.verifyGoogleToken(token);
    
    // Find or create user
    let user = await this.userRepository.findOne({
      where: { email: googleUser.email }
    });

    if (!user) {
      user = await this.userRepository.create({
        email: googleUser.email,
        name: googleUser.name,
        profilePicture: googleUser.picture,
        googleId: googleUser.googleId
      });
    }

    // Generate JWT
    return {
      accessToken: this.generateAccessToken(user.id),
      refreshToken: this.generateRefreshToken(user.id),
      user
    };
  }
}
```

### Multi-Factor Authentication (MFA)
```typescript
// TOTP (Time-based One-Time Password)
import speakeasy from 'speakeasy';
import QRCode from 'qrcode';

export class MFAService {
  // Generate secret for user
  async generateSecret(userId: string) {
    const secret = speakeasy.generateSecret({
      name: `MyApp (${userId})`,
      issuer: 'MyApp'
    });

    // Generate QR code
    const qrCodeUrl = await QRCode.toDataURL(secret.otpauth_url);

    // Store secret in database (encrypted!)
    await this.userRepository.update(userId, {
      mfaSecret: this.encrypt(secret.base32)
    });

    return {
      secret: secret.base32,
      qrCode: qrCodeUrl
    };
  }

  // Verify TOTP token
  async verifyToken(userId: string, token: string): Promise<boolean> {
    const user = await this.userRepository.findOne(userId);
    const secret = this.decrypt(user.mfaSecret);

    return speakeasy.totp.verify({
      secret,
      encoding: 'base32',
      token,
      window: 2 // Allow 2 steps before/after (60s)
    });
  }

  // Login with MFA
  async loginWithMFA(email: string, password: string, mfaToken: string) {
    // Verify password
    const user = await this.authService.validateUser(email, password);
    
    if (!user) {
      throw new UnauthorizedException('Invalid credentials');
    }

    // Verify MFA token
    if (user.mfaEnabled) {
      const isValidMFA = await this.verifyToken(user.id, mfaToken);
      
      if (!isValidMFA) {
        throw new UnauthorizedException('Invalid MFA token');
      }
    }

    return this.authService.generateTokens(user.id);
  }
}
```

---

## Authorization

### Role-Based Access Control (RBAC)
```typescript
// Define roles and permissions
export enum Role {
  ADMIN = 'admin',
  USER = 'user',
  MODERATOR = 'moderator'
}

export enum Permission {
  CREATE_USER = 'create:user',
  READ_USER = 'read:user',
  UPDATE_USER = 'update:user',
  DELETE_USER = 'delete:user',
  MANAGE_ROLES = 'manage:roles'
}

// Role to permissions mapping
const rolePermissions: Record<Role, Permission[]> = {
  [Role.ADMIN]: [
    Permission.CREATE_USER,
    Permission.READ_USER,
    Permission.UPDATE_USER,
    Permission.DELETE_USER,
    Permission.MANAGE_ROLES
  ],
  [Role.MODERATOR]: [
    Permission.READ_USER,
    Permission.UPDATE_USER
  ],
  [Role.USER]: [
    Permission.READ_USER
  ]
};

// Roles guard
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.get<Role[]>(
      'roles',
      context.getHandler()
    );

    if (!requiredRoles) {
      return true; // No roles required
    }

    const request = context.switchToHttp().getRequest();
    const user = request.user;

    return requiredRoles.some(role => user.roles?.includes(role));
  }
}

// Usage
@Controller('admin')
@UseGuards(AuthGuard, RolesGuard)
export class AdminController {
  @Post('users')
  @Roles(Role.ADMIN)
  createUser(@Body() data: CreateUserDto) {
    // Only admins can access
  }

  @Get('users')
  @Roles(Role.ADMIN, Role.MODERATOR)
  getUsers() {
    // Admins and moderators can access
  }
}
```

### Permission-Based Access Control
```typescript
// Check specific permissions
@Injectable()
export class PermissionsGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredPermissions = this.reflector.get<Permission[]>(
      'permissions',
      context.getHandler()
    );

    if (!requiredPermissions) {
      return true;
    }

    const request = context.switchToHttp().getRequest();
    const user = request.user;
    
    const userPermissions = this.getUserPermissions(user.roles);

    return requiredPermissions.every(permission =>
      userPermissions.includes(permission)
    );
  }

  private getUserPermissions(roles: Role[]): Permission[] {
    return roles.flatMap(role => rolePermissions[role] || []);
  }
}
```

### Resource-Based Authorization
```typescript
// User can only access their own resources
@Injectable()
export class ResourceOwnerGuard implements CanActivate {
  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    const resourceId = request.params.id;

    // Admin can access all resources
    if (user.roles.includes(Role.ADMIN)) {
      return true;
    }

    // Check if user owns the resource
    const resource = await this.getResource(resourceId);
    return resource.userId === user.id;
  }
}

// Usage
@Controller('posts')
@UseGuards(AuthGuard)
export class PostController {
  @Put(':id')
  @UseGuards(ResourceOwnerGuard)
  updatePost(@Param('id') id: string, @Body() data: UpdatePostDto) {
    // Only owner or admin can update
  }
}
```

---

## Data Protection

### Encryption at Rest
```typescript
// Encrypt sensitive data in database
import crypto from 'crypto';

export class EncryptionService {
  private algorithm = 'aes-256-gcm';
  private key = Buffer.from(process.env.ENCRYPTION_KEY!, 'hex'); // 32 bytes

  encrypt(text: string): string {
    const iv = crypto.randomBytes(16);
    const cipher = crypto.createCipheriv(this.algorithm, this.key, iv);
    
    let encrypted = cipher.update(text, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    const authTag = cipher.getAuthTag();
    
    // Return: iv:authTag:encrypted
    return `${iv.toString('hex')}:${authTag.toString('hex')}:${encrypted}`;
  }

  decrypt(encryptedText: string): string {
    const [ivHex, authTagHex, encrypted] = encryptedText.split(':');
    
    const iv = Buffer.from(ivHex, 'hex');
    const authTag = Buffer.from(authTagHex, 'hex');
    
    const decipher = crypto.createDecipheriv(this.algorithm, this.key, iv);
    decipher.setAuthTag(authTag);
    
    let decrypted = decipher.update(encrypted, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    
    return decrypted;
  }
}

// Usage in entity
@Entity()
export class User {
  @Column()
  email: string;

  @Column()
  private _ssn: string; // Social Security Number

  @AfterLoad()
  decryptSensitiveData() {
    if (this._ssn) {
      this._ssn = encryptionService.decrypt(this._ssn);
    }
  }

  @BeforeInsert()
  @BeforeUpdate()
  encryptSensitiveData() {
    if (this._ssn) {
      this._ssn = encryptionService.encrypt(this._ssn);
    }
  }

  get ssn(): string {
    return this._ssn;
  }

  set ssn(value: string) {
    this._ssn = value;
  }
}
```

### Encryption in Transit (HTTPS/TLS)
```typescript
// Force HTTPS
import helmet from 'helmet';
import { NestFactory } from '@nestjs/core';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Security middleware
  app.use(helmet());

  // Force HTTPS in production
  if (process.env.NODE_ENV === 'production') {
    app.use((req, res, next) => {
      if (!req.secure) {
        return res.redirect(301, `https://${req.headers.host}${req.url}`);
      }
      next();
    });
  }

  await app.listen(3000);
}
```

### Secrets Management
```typescript
// Use environment variables
// NEVER hardcode secrets!

// Bad
❌ const apiKey = 'sk_live_abc123xyz';
❌ const dbPassword = 'MyPassword123';

// Good
✅ const apiKey = process.env.API_KEY;
✅ const dbPassword = process.env.DB_PASSWORD;

// .env (NOT committed to git)
API_KEY=sk_live_abc123xyz
DB_PASSWORD=MyPassword123
JWT_SECRET=super_secret_key_change_this

// .env.example (committed to git)
API_KEY=your_api_key_here
DB_PASSWORD=your_db_password_here
JWT_SECRET=your_jwt_secret_here
```

### AWS Secrets Manager
```typescript
import {
  SecretsManagerClient,
  GetSecretValueCommand
} from '@aws-sdk/client-secrets-manager';

export class SecretsService {
  private client = new SecretsManagerClient({ region: 'us-east-1' });

  async getSecret(secretName: string): Promise<any> {
    const command = new GetSecretValueCommand({
      SecretId: secretName
    });

    const response = await this.client.send(command);
    return JSON.parse(response.SecretString);
  }
}

// Usage
const dbConfig = await secretsService.getSecret('prod/database');
// Returns: { host, port, username, password }
```

---

## API Security

### Rate Limiting
```typescript
// Prevent brute force attacks
import rateLimit from 'express-rate-limit';

// Global rate limit
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requests per window
  message: 'Too many requests, please try again later',
  standardHeaders: true,
  legacyHeaders: false
});

app.use(limiter);

// Strict rate limit for login
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5, // 5 login attempts per 15 minutes
  skipSuccessfulRequests: true
});

app.post('/auth/login', loginLimiter, authController.login);

// IP-based rate limiting
import { ThrottlerModule } from '@nestjs/throttler';

@Module({
  imports: [
    ThrottlerModule.forRoot({
      ttl: 60, // Time to live (seconds)
      limit: 10 // Max requests per ttl
    })
  ]
})
export class AppModule {}
```

### Input Validation
```typescript
// Validate all inputs!
import { IsEmail, IsString, MinLength, MaxLength } from 'class-validator';

export class CreateUserDto {
  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  @MaxLength(100)
  password: string;

  @IsString()
  @MinLength(2)
  @MaxLength(50)
  name: string;
}

// Sanitize inputs
import { Transform } from 'class-transformer';
import DOMPurify from 'isomorphic-dompurify';

export class CreatePostDto {
  @IsString()
  @Transform(({ value }) => DOMPurify.sanitize(value))
  content: string;
}
```

### CORS Configuration
```typescript
// Configure CORS properly
app.enableCors({
  origin: process.env.NODE_ENV === 'production'
    ? ['https://myapp.com', 'https://www.myapp.com']
    : ['http://localhost:3000', 'http://localhost:3001'],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization']
});
```

### API Keys
```typescript
// Protect API with keys
@Injectable()
export class ApiKeyGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const apiKey = request.headers['x-api-key'];

    if (!apiKey) {
      throw new UnauthorizedException('API key required');
    }

    // Verify API key (check database or cache)
    const isValid = this.validateApiKey(apiKey);
    
    if (!isValid) {
      throw new UnauthorizedException('Invalid API key');
    }

    return true;
  }

  private async validateApiKey(apiKey: string): Promise<boolean> {
    // Check in database or Redis
    const key = await this.apiKeyRepository.findOne({
      where: { key: apiKey, active: true }
    });

    if (!key) {
      return false;
    }

    // Update last used
    await this.apiKeyRepository.update(key.id, {
      lastUsed: new Date()
    });

    return true;
  }
}
```

---

## Frontend Security

### XSS (Cross-Site Scripting) Prevention
```typescript
// React automatically escapes
// But be careful with dangerouslySetInnerHTML!

// Bad
❌ <div dangerouslySetInnerHTML={{ __html: userInput }} />

// Good - Sanitize first
import DOMPurify from 'dompurify';

function SafeContent({ html }: { html: string }) {
  const sanitized = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a'],
    ALLOWED_ATTR: ['href']
  });

  return <div dangerouslySetInnerHTML={{ __html: sanitized }} />;
}
```

### CSRF (Cross-Site Request Forgery) Prevention
```typescript
// Use CSRF tokens
import csurf from 'csurf';

const csrfProtection = csurf({
  cookie: {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict'
  }
});

app.use(csrfProtection);

// Send token to frontend
app.get('/api/csrf-token', (req, res) => {
  res.json({ csrfToken: req.csrfToken() });
});

// Frontend - Include token in requests
const csrfToken = await fetch('/api/csrf-token').then(r => r.json());

fetch('/api/data', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'CSRF-Token': csrfToken.csrfToken
  },
  body: JSON.stringify(data)
});
```

### Content Security Policy (CSP)
```typescript
// Prevent XSS attacks
import helmet from 'helmet';

app.use(
  helmet.contentSecurityPolicy({
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'", "https://cdn.example.com"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", "data:", "https:"],
      connectSrc: ["'self'", "https://api.example.com"],
      fontSrc: ["'self'", "https://fonts.googleapis.com"],
      objectSrc: ["'none'"],
      upgradeInsecureRequests: []
    }
  })
);
```

### Secure Cookie Configuration
```typescript
// Set cookies securely
res.cookie('token', jwt, {
  httpOnly: true,      // Not accessible via JavaScript
  secure: true,        // HTTPS only
  sameSite: 'strict',  // CSRF protection
  maxAge: 3600000,     // 1 hour
  domain: '.myapp.com' // Subdomain access
});
```

---

## Database Security

### SQL Injection Prevention
```typescript
// ALWAYS use parameterized queries!

// Bad - SQL Injection vulnerable
❌ const query = `SELECT * FROM users WHERE email = '${email}'`;
// Attacker input: ' OR '1'='1
// Query becomes: SELECT * FROM users WHERE email = '' OR '1'='1'
// Returns all users!

// Good - Parameterized query
✅ const query = 'SELECT * FROM users WHERE email = $1';
const result = await db.query(query, [email]);

// TypeORM (automatically safe)
await userRepository.findOne({
  where: { email }  // Safe from SQL injection
});

// Raw query with TypeORM
await connection.query(
  'SELECT * FROM users WHERE email = $1 AND active = $2',
  [email, true]
);
```

### Database Access Control
```typescript
// Use separate database users with limited permissions

// Application user (limited)
CREATE USER app_user WITH PASSWORD 'secure_password';
GRANT SELECT, INSERT, UPDATE ON users TO app_user;
GRANT SELECT, INSERT, UPDATE, DELETE ON posts TO app_user;
-- No DROP, TRUNCATE, or admin privileges

// Read-only user (for analytics)
CREATE USER readonly_user WITH PASSWORD 'secure_password';
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly_user;

// Connection
const config = {
  type: 'postgres',
  host: process.env.DB_HOST,
  username: 'app_user',  // Not root or admin!
  password: process.env.DB_PASSWORD,
  database: 'myapp'
};
```

### Sensitive Data Storage
```typescript
// Hash passwords, encrypt sensitive data

@Entity()
export class User {
  @Column()
  email: string;

  @Column()
  password: string; // Hashed with bcrypt

  @Column({ nullable: true })
  creditCard: string; // Encrypted

  @Column({ select: false }) // Don't return in queries by default
  ssn: string; // Encrypted

  @Column({ nullable: true })
  apiKey: string; // Hashed
}

// Don't return sensitive fields
const user = await userRepository.findOne(
  { where: { id } },
  { select: ['id', 'email', 'name'] } // Exclude password, ssn, etc.
);
```

---

## Infrastructure Security

### Network Security
```typescript
// AWS Security Groups
{
  "SecurityGroups": [{
    "GroupName": "web-server",
    "IpPermissions": [
      {
        "IpProtocol": "tcp",
        "FromPort": 443,
        "ToPort": 443,
        "IpRanges": [{ "CidrIp": "0.0.0.0/0" }] // HTTPS from anywhere
      },
      {
        "IpProtocol": "tcp",
        "FromPort": 22,
        "ToPort": 22,
        "IpRanges": [{ "CidrIp": "10.0.0.0/8" }] // SSH from VPN only
      }
    ]
  }]
}

// Database security group
{
  "GroupName": "database",
  "IpPermissions": [{
    "IpProtocol": "tcp",
    "FromPort": 5432,
    "ToPort": 5432,
    "UserIdGroupPairs": [{
      "GroupId": "sg-web-server" // Only from web servers
    }]
  }]
}
```

### Container Security
```dockerfile
# Use minimal base images
FROM node:18-alpine  # Not node:18 (smaller attack surface)

# Run as non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# Copy only necessary files
COPY --chown=appuser:appgroup package*.json ./
RUN npm ci --only=production
COPY --chown=appuser:appgroup . .

# No secrets in image
# Use environment variables or secret management
```

### Regular Updates
```bash
# Keep dependencies updated
npm audit          # Check vulnerabilities
npm audit fix      # Auto-fix
npm outdated       # Check outdated packages

# Automated updates
npm install -g npm-check-updates
ncu -u             # Update package.json
npm install        # Install updates

# Dependabot (GitHub)
# Automatically creates PRs for updates
```

---

## Common Vulnerabilities

### 1. SQL Injection
```typescript
// Vulnerable
const email = req.query.email;
const query = `SELECT * FROM users WHERE email = '${email}'`;

// Attack: ?email=' OR '1'='1
// Query: SELECT * FROM users WHERE email = '' OR '1'='1'
// Returns all users!

// Fix: Use parameterized queries
const query = 'SELECT * FROM users WHERE email = $1';
await db.query(query, [email]);
```

### 2. XSS (Cross-Site Scripting)
```html
<!-- Vulnerable -->
<div>{userInput}</div>

<!-- Attack: userInput = "<script>alert('XSS')</script>" -->
<!-- Browser executes malicious script! -->

<!-- Fix: Escape HTML (React does this automatically) -->
<!-- Or sanitize with DOMPurify -->
```

### 3. CSRF (Cross-Site Request Forgery)
```html
<!-- Attack: Evil website -->
<form action="https://bank.com/transfer" method="POST">
  <input name="to" value="attacker" />
  <input name="amount" value="10000" />
</form>
<script>document.forms[0].submit();</script>

<!-- Victim is logged in to bank.com -->
<!-- Form auto-submits, transferring money! -->

<!-- Fix: Use CSRF tokens -->
<!-- Or SameSite cookies -->
```

### 4. Authentication Bypass
```typescript
// Vulnerable - Missing authentication
@Get('admin/users')
getAllUsers() {
  return this.users; // No auth check!
}

// Fix: Require authentication
@Get('admin/users')
@UseGuards(AuthGuard, AdminGuard)
getAllUsers() {
  return this.users;
}
```

### 5. Broken Access Control
```typescript
// Vulnerable - No ownership check
@Delete('posts/:id')
async deletePost(@Param('id') id: string) {
  await this.postRepository.delete(id);
  // Any user can delete any post!
}

// Fix: Check ownership
@Delete('posts/:id')
async deletePost(@Param('id') id: string, @User() user) {
  const post = await this.postRepository.findOne(id);
  
  if (post.userId !== user.id && !user.isAdmin) {
    throw new ForbiddenException();
  }
  
  await this.postRepository.delete(id);
}
```

### 6. Sensitive Data Exposure
```typescript
// Vulnerable - Returning password
@Get('users/:id')
async getUser(@Param('id') id: string) {
  return this.userRepository.findOne(id);
  // Returns: { id, email, password: 'hashed', ssn, ... }
}

// Fix: Exclude sensitive fields
@Get('users/:id')
async getUser(@Param('id') id: string) {
  const user = await this.userRepository.findOne(id, {
    select: ['id', 'email', 'name', 'createdAt']
  });
  return user;
}
```

### 7. Missing Rate Limiting
```typescript
// Vulnerable - No rate limit
@Post('login')
async login(@Body() credentials: LoginDto) {
  return this.authService.login(credentials);
  // Attacker can try millions of passwords!
}

// Fix: Add rate limiting
@Post('login')
@Throttle(5, 900) // 5 attempts per 15 minutes
async login(@Body() credentials: LoginDto) {
  return this.authService.login(credentials);
}
```

---

## Security Checklist

### Development
```
✅ All inputs validated
✅ SQL queries parameterized
✅ Passwords hashed (bcrypt)
✅ Secrets in environment variables
✅ HTTPS enforced
✅ CORS configured
✅ Helmet middleware enabled
✅ Rate limiting implemented
✅ Authentication on all protected routes
✅ Authorization checks (ownership)
✅ Error messages don't leak info
✅ Logging (but not sensitive data)
```

### Deployment
```
✅ Use HTTPS/TLS
✅ Secure cookies (httpOnly, secure, sameSite)
✅ CSP headers configured
✅ Database in private network
✅ Firewall rules configured
✅ SSH keys (not passwords)
✅ Regular backups
✅ Monitoring and alerting
✅ Dependency scanning
✅ Security headers (Helmet)
```

### Production
```
✅ NODE_ENV=production
✅ Debug mode disabled
✅ Error stack traces hidden
✅ Secrets in secure storage (Vault, Secrets Manager)
✅ Database credentials rotated
✅ API keys rotated
✅ SSL certificate valid
✅ Security audit completed
✅ Penetration testing done
✅ Incident response plan ready
```

---

## Interview Questions

### Basic Questions

**Q1: What is the difference between authentication and authorization?**
```
Authentication (AuthN):
- Who are you?
- Verifying identity
- Login with username/password
- Example: User logs in with email and password

Authorization (AuthZ):
- What can you do?
- Verifying permissions
- Check access rights
- Example: Admin can delete users, regular users cannot

Both are needed:
1. Authenticate: Verify user is who they claim
2. Authorize: Check if user can perform action
```

**Q2: How do you securely store passwords?**
```
NEVER store plain text passwords!

Use hashing:
1. Hash with bcrypt (or argon2, scrypt)
   - One-way function (can't reverse)
   - Includes salt (random data)
   - Slow by design (prevents brute force)

2. Store hash in database
   - "password" → "$2b$10$xyz..."

3. Verify by comparing hashes
   - bcrypt.compare(input, stored_hash)

Don't:
❌ Store plain text
❌ Use MD5 or SHA1 (too fast)
❌ Use same salt for all users
❌ Encrypt passwords (encryption is reversible)
```

**Q3: What is SQL injection and how to prevent it?**
```
SQL Injection:
Attacker inserts malicious SQL code

Example:
// Vulnerable code
query = "SELECT * FROM users WHERE email = '" + email + "'";

// Attack
email = "' OR '1'='1";
// Query becomes:
SELECT * FROM users WHERE email = '' OR '1'='1'
// Returns ALL users!

Prevention:
1. Use parameterized queries
   query = "SELECT * FROM users WHERE email = $1";
   execute(query, [email]);

2. Use ORM (TypeORM, Prisma)
   await userRepository.findOne({ where: { email } });

3. Validate inputs
   - Check format
   - Whitelist allowed characters
```

### Advanced Questions

**Q4: Explain JWT and its security considerations**
```
JWT (JSON Web Token):
- Stateless authentication
- Encoded, NOT encrypted
- Can be decoded by anyone

Structure:
header.payload.signature

Security Considerations:

1. Short Expiration
   - Access token: 15 minutes
   - Refresh token: 7 days

2. Secure Secret
   - Strong, random secret key
   - Never expose in code
   - Rotate periodically

3. HTTPS Only
   - JWT stolen = account compromised
   - Must use HTTPS

4. Store Securely
   - Not in localStorage (XSS risk)
   - httpOnly cookie (better)
   - Or in memory

5. Validate Everything
   - Signature
   - Expiration
   - Issuer
   - Audience

6. Don't Store Sensitive Data
   - JWT can be decoded
   - Only store non-sensitive data (user ID)
```

**Q5: How do you prevent XSS attacks?**
```
XSS (Cross-Site Scripting):
Attacker injects malicious JavaScript

Types:
1. Stored XSS: Saved in database
2. Reflected XSS: In URL/input
3. DOM-based XSS: In client-side code

Prevention:

1. Escape Output
   React: Automatically escapes
   Vue: v-text (not v-html)
   Angular: Sanitizes by default

2. Sanitize HTML
   import DOMPurify from 'dompurify';
   const clean = DOMPurify.sanitize(dirty);

3. Content Security Policy
   helmet.contentSecurityPolicy({
     directives: {
       scriptSrc: ["'self'", "trusted-cdn.com"]
     }
   });

4. Validate Inputs
   - Whitelist allowed characters
   - Reject suspicious patterns

5. HttpOnly Cookies
   - JavaScript can't access
   - Prevents cookie theft

6. Use Frameworks
   - React, Vue, Angular auto-escape
   - Don't use dangerouslySetInnerHTML
```

**Q6: What are security headers and why use them?**
```
Security Headers: HTTP response headers that improve security

Using Helmet middleware:

1. X-Frame-Options
   - Prevents clickjacking
   - Value: DENY or SAMEORIGIN

2. X-Content-Type-Options
   - Prevents MIME sniffing
   - Value: nosniff

3. X-XSS-Protection
   - Browser XSS filter
   - Value: 1; mode=block

4. Strict-Transport-Security (HSTS)
   - Force HTTPS
   - Value: max-age=31536000

5. Content-Security-Policy (CSP)
   - Control resource loading
   - Prevent XSS

6. Referrer-Policy
   - Control referrer info
   - Value: no-referrer

Implementation:
import helmet from 'helmet';
app.use(helmet());
```

**Q7: How do you implement rate limiting?**
```
Rate Limiting: Restrict number of requests

Why:
- Prevent brute force attacks
- Prevent DoS attacks
- Protect API resources

Implementation:

1. Simple Rate Limit
   - 100 requests per 15 minutes
   - Per IP address

2. Endpoint-Specific
   - Login: 5 attempts per 15 min
   - API: 1000 requests per hour
   - File upload: 10 per hour

3. User-Based
   - Free tier: 100 req/day
   - Paid tier: Unlimited

Example:
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  message: 'Too many requests'
});

app.use('/api/', limiter);

// Stricter for login
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5
});

app.post('/login', loginLimiter, loginHandler);

Storage:
- Memory (simple, resets on restart)
- Redis (distributed, persistent)
```

**Q8: What is CSRF and how to prevent it?**
```
CSRF (Cross-Site Request Forgery):
Attacker tricks user into making unwanted request

Example Attack:
1. User logged in to bank.com
2. User visits evil.com
3. evil.com contains:
   <form action="https://bank.com/transfer">
     <input name="to" value="attacker" />
     <input name="amount" value="10000" />
   </form>
   <script>form.submit();</script>
4. Browser sends cookies automatically
5. Money transferred!

Prevention:

1. CSRF Tokens
   - Server generates unique token
   - Include in forms
   - Verify on submission
   
   <input type="hidden" name="_csrf" value="abc123" />

2. SameSite Cookies
   Set-Cookie: token=xxx; SameSite=Strict
   - Browser won't send from other sites

3. Custom Headers
   - AJAX requests with custom header
   - Attackers can't set custom headers
   
   fetch('/api', {
     headers: { 'X-CSRF-Token': token }
   });

4. Verify Origin/Referer
   - Check request came from your site
   
5. Re-authenticate Sensitive Actions
   - Require password for sensitive operations
```

---

## Summary

### Security Layers
```
1. Perimeter
   - Firewall, WAF, DDoS protection

2. Network
   - VPC, security groups, private subnets

3. Application
   - Authentication, authorization
   - Input validation, rate limiting

4. Data
   - Encryption (rest and transit)
   - Hashing, sanitization

5. Monitoring
   - Logging, alerting
   - Intrusion detection
```

### Key Takeaways
1. **Defense in Depth**: Multiple security layers
2. **Least Privilege**: Minimum necessary permissions
3. **Validate Everything**: Never trust user input
4. **Encrypt Sensitive Data**: Both at rest and in transit
5. **Use Standard Libraries**: Don't roll your own crypto
6. **Keep Updated**: Regular security patches
7. **Monitor Actively**: Detect attacks early
8. **Plan for Incidents**: Have response plan ready

### Top Priorities
1. ✅ HTTPS everywhere
2. ✅ Hash passwords (bcrypt)
3. ✅ Validate all inputs
4. ✅ Use parameterized queries
5. ✅ Implement authentication/authorization
6. ✅ Rate limiting
7. ✅ Security headers (Helmet)
8. ✅ Regular updates

Security is not optional - it's essential! 🔒
