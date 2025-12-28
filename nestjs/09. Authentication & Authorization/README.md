# NestJS Authentication & Authorization - Top Interview Questions

## Fundamentals

### 1. What is the difference between Authentication and Authorization?

<details>
<summary>Answer</summary>

**Authentication** verifies *who you are* (identity). **Authorization** verifies *what you can do* (permissions).

| Aspect | Authentication | Authorization |
|--------|---------------|---------------|
| **Purpose** | Verify identity | Verify permissions |
| **Question** | Who are you? | What can you do? |
| **Process** | Login with credentials | Check user roles/permissions |
| **Example** | Username + Password | Admin can delete, User can only read |
| **Happens** | First | After authentication |
| **Failure** | 401 Unauthorized | 403 Forbidden |

**Examples:**

```typescript
// Authentication: Verify user identity
@Post('login')
async login(@Body() loginDto: LoginDto) {
  // Verify username and password
  const user = await this.authService.validateUser(
    loginDto.username,
    loginDto.password
  );
  
  if (!user) {
    throw new UnauthorizedException('Invalid credentials'); // 401
  }
  
  // Generate JWT token
  return this.authService.login(user);
}

// Authorization: Check if user has permission
@Get('admin/users')
@UseGuards(JwtAuthGuard, RolesGuard)
@Roles('admin') // Only admins can access
async getAllUsers() {
  // If user is not admin, returns 403 Forbidden
  return this.userService.findAll();
}
```

**Interview Tip**: Authentication = Identity verification (who you are). Authorization = Permission check (what you can do). Authentication happens first, then authorization. Use 401 for failed authentication, 403 for failed authorization.

</details>

### 2. What is Passport and how does it integrate with NestJS?

<details>
<summary>Answer</summary>

**Passport** is a popular Node.js authentication middleware supporting 500+ strategies. NestJS provides `@nestjs/passport` wrapper for easy integration.

**Installation:**

```bash
npm install @nestjs/passport passport
npm install @nestjs/jwt passport-jwt
npm install bcrypt
npm install @types/passport-jwt @types/bcrypt --save-dev
```

**How Passport Works in NestJS:**

1. **Strategy**: Defines authentication logic (JWT, Local, OAuth, etc.)
2. **Guard**: Applies strategy to routes
3. **Validate**: Verifies credentials and returns user

**Basic Integration:**

```typescript
// auth.module.ts
import { Module } from '@nestjs/common';
import { PassportModule } from '@nestjs/passport';
import { JwtModule } from '@nestjs/jwt';
import { AuthService } from './auth.service';
import { LocalStrategy } from './strategies/local.strategy';
import { JwtStrategy } from './strategies/jwt.strategy';

@Module({
  imports: [
    PassportModule,
    JwtModule.register({
      secret: 'your-secret-key',
      signOptions: { expiresIn: '1h' },
    }),
  ],
  providers: [AuthService, LocalStrategy, JwtStrategy],
  exports: [AuthService],
})
export class AuthModule {}
```

**Strategy Example:**

```typescript
// strategies/jwt.strategy.ts
import { Injectable } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      secretOrKey: 'your-secret-key',
    });
  }

  // Passport automatically calls this method after verifying token
  async validate(payload: any) {
    return { userId: payload.sub, username: payload.username };
  }
}
```

**Using Guard:**

```typescript
// Protected route
@Get('profile')
@UseGuards(AuthGuard('jwt')) // Uses JwtStrategy
getProfile(@Request() req) {
  return req.user; // User from validate() method
}
```

**Interview Tip**: Passport provides authentication strategies. NestJS wraps it with `@nestjs/passport`. Create strategies by extending `PassportStrategy`. Use `AuthGuard()` to protect routes. The `validate()` method in strategy returns user object attached to request.

</details>

### 3. What are the common authentication strategies (JWT, Local, OAuth)?

<details>
<summary>Answer</summary>

**Common Strategies:**

| Strategy | Use Case | Pros | Cons |
|----------|----------|------|------|
| **Local** | Username/Password | Simple, full control | Need to manage sessions |
| **JWT** | Stateless API auth | Scalable, no server storage | Cannot revoke easily |
| **OAuth 2.0** | Third-party login (Google, Facebook) | User convenience, no password management | Dependency on provider |
| **Session** | Traditional web apps | Can revoke easily | Requires server storage |

**1. Local Strategy (Username/Password):**

```typescript
// strategies/local.strategy.ts
import { Strategy } from 'passport-local';
import { PassportStrategy } from '@nestjs/passport';
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { AuthService } from '../auth.service';

@Injectable()
export class LocalStrategy extends PassportStrategy(Strategy) {
  constructor(private authService: AuthService) {
    super({
      usernameField: 'email', // Default is 'username'
      passwordField: 'password',
    });
  }

  async validate(username: string, password: string): Promise<any> {
    const user = await this.authService.validateUser(username, password);
    if (!user) {
      throw new UnauthorizedException('Invalid credentials');
    }
    return user;
  }
}

// Usage in controller
@Post('login')
@UseGuards(AuthGuard('local'))
async login(@Request() req) {
  return this.authService.login(req.user);
}
```

**2. JWT Strategy (Stateless Token):**

```typescript
// strategies/jwt.strategy.ts
import { ExtractJwt, Strategy } from 'passport-jwt';
import { PassportStrategy } from '@nestjs/passport';
import { Injectable } from '@nestjs/common';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: process.env.JWT_SECRET,
    });
  }

  async validate(payload: any) {
    return { 
      userId: payload.sub, 
      username: payload.username,
      roles: payload.roles 
    };
  }
}

// Usage
@Get('profile')
@UseGuards(AuthGuard('jwt'))
getProfile(@Request() req) {
  return req.user;
}
```

**3. OAuth Strategy (Google):**

```typescript
// strategies/google.strategy.ts
import { PassportStrategy } from '@nestjs/passport';
import { Strategy, VerifyCallback } from 'passport-google-oauth20';
import { Injectable } from '@nestjs/common';

@Injectable()
export class GoogleStrategy extends PassportStrategy(Strategy, 'google') {
  constructor() {
    super({
      clientID: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
      callbackURL: 'http://localhost:3000/auth/google/callback',
      scope: ['email', 'profile'],
    });
  }

  async validate(
    accessToken: string,
    refreshToken: string,
    profile: any,
    done: VerifyCallback,
  ): Promise<any> {
    const { name, emails, photos } = profile;
    const user = {
      email: emails[0].value,
      firstName: name.givenName,
      lastName: name.familyName,
      picture: photos[0].value,
    };
    done(null, user);
  }
}

// Usage
@Get('google')
@UseGuards(AuthGuard('google'))
async googleAuth() {
  // Redirects to Google
}

@Get('google/callback')
@UseGuards(AuthGuard('google'))
googleAuthRedirect(@Request() req) {
  return this.authService.googleLogin(req.user);
}
```

**Interview Tip**: Local = username/password. JWT = stateless token-based. OAuth = third-party providers (Google, Facebook). Use Local for simple apps, JWT for APIs, OAuth for social login. Register all strategies in AuthModule providers.

</details>

## JWT Authentication

### 4. What is JWT (JSON Web Token)?

<details>
<summary>Answer</summary>

**JWT** is an open standard (RFC 7519) for securely transmitting information as a JSON object. Used for stateless authentication.

**Key Characteristics:**

- **Stateless**: Server doesn't store tokens
- **Self-contained**: Contains user info in payload
- **Portable**: Works across domains
- **Secure**: Digitally signed

**JWT Structure: `header.payload.signature`**

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

**How JWT Works:**

```typescript
// 1. User logs in with credentials
POST /auth/login
{
  "username": "john",
  "password": "secret123"
}

// 2. Server validates and returns JWT
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}

// 3. Client includes JWT in subsequent requests
GET /api/profile
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

// 4. Server verifies JWT and returns data
```

**Advantages:**

- No server-side session storage
- Scalable (works with load balancers)
- Cross-domain authentication
- Mobile-friendly

**Disadvantages:**

- Cannot revoke before expiration
- Token size larger than session ID
- If secret compromised, all tokens invalid

**Interview Tip**: JWT is stateless, self-contained token. Format: `header.payload.signature`. Server doesn't store tokens. Client sends token in Authorization header. Use for APIs and microservices.

</details>

### 5. What are the three parts of a JWT (header, payload, signature)?

<details>
<summary>Answer</summary>

JWT has three parts separated by dots: `header.payload.signature`

**1. Header** - Metadata about token

```json
{
  "alg": "HS256",  // Signing algorithm
  "typ": "JWT"      // Token type
}
```

**2. Payload** - Claims (user data)

```json
{
  "sub": "1234567890",        // Subject (user ID)
  "username": "john",
  "email": "john@example.com",
  "roles": ["user", "admin"],
  "iat": 1516239022,          // Issued at
  "exp": 1516242622           // Expiration
}
```

**3. Signature** - Verification

```javascript
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret
)
```

**Complete Example:**

```typescript
// Generating JWT in NestJS
import { JwtService } from '@nestjs/jwt';

@Injectable()
export class AuthService {
  constructor(private jwtService: JwtService) {}

  async login(user: User) {
    // Payload (claims)
    const payload = { 
      sub: user.id,           // Standard claim: subject
      username: user.username,
      email: user.email,
      roles: user.roles,
      iat: Math.floor(Date.now() / 1000),  // Issued at
    };

    // JwtService automatically adds header and creates signature
    return {
      access_token: this.jwtService.sign(payload),
    };
  }

  async verifyToken(token: string) {
    try {
      // Verifies signature and returns payload
      const payload = this.jwtService.verify(token);
      return payload;
    } catch (error) {
      throw new UnauthorizedException('Invalid token');
    }
  }
}
```

**Standard Claims (Payload):**

| Claim | Name | Description |
|-------|------|-------------|
| `sub` | Subject | User ID |
| `iat` | Issued At | Token creation time |
| `exp` | Expiration | Token expiry time |
| `iss` | Issuer | Token issuer |
| `aud` | Audience | Token recipient |
| `nbf` | Not Before | Token valid from |

**Interview Tip**: JWT = `header.payload.signature`. Header has algorithm info. Payload has user data (claims). Signature verifies integrity using secret. All three parts are Base64URL encoded. Never put sensitive data in payload (it's not encrypted, just encoded).

</details>

### 6. How do you install and configure `JwtModule`?

<details>
<summary>Answer</summary>

**Installation:**

```bash
npm install @nestjs/jwt @nestjs/passport passport passport-jwt
npm install @types/passport-jwt --save-dev
```

**Basic Configuration:**

```typescript
// auth.module.ts
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { PassportModule } from '@nestjs/passport';
import { AuthService } from './auth.service';
import { AuthController } from './auth.controller';
import { JwtStrategy } from './strategies/jwt.strategy';
import { UsersModule } from '../users/users.module';

@Module({
  imports: [
    UsersModule,
    PassportModule,
    JwtModule.register({
      secret: 'your-secret-key', // Should be in env variable
      signOptions: { 
        expiresIn: '1h',  // Token expires in 1 hour
      },
    }),
  ],
  providers: [AuthService, JwtStrategy],
  controllers: [AuthController],
  exports: [AuthService],
})
export class AuthModule {}
```

**Advanced Configuration with ConfigService:**

```typescript
// auth.module.ts
import { ConfigModule, ConfigService } from '@nestjs/config';

@Module({
  imports: [
    PassportModule.register({ defaultStrategy: 'jwt' }),
    JwtModule.registerAsync({
      imports: [ConfigModule],
      useFactory: async (configService: ConfigService) => ({
        secret: configService.get<string>('JWT_SECRET'),
        signOptions: {
          expiresIn: configService.get<string>('JWT_EXPIRES_IN', '1h'),
          issuer: 'my-app',
          audience: 'my-app-users',
        },
      }),
      inject: [ConfigService],
    }),
  ],
  // ... rest of module
})
export class AuthModule {}
```

**Environment Variables (.env):**

```env
JWT_SECRET=your-very-secure-secret-key-min-32-characters
JWT_EXPIRES_IN=1h
JWT_REFRESH_SECRET=another-secure-secret-for-refresh-tokens
JWT_REFRESH_EXPIRES_IN=7d
```

**Global JwtModule (App-wide):**

```typescript
// app.module.ts
@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),
    JwtModule.registerAsync({
      global: true, // Makes JwtModule available everywhere
      imports: [ConfigModule],
      useFactory: (configService: ConfigService) => ({
        secret: configService.get('JWT_SECRET'),
        signOptions: { expiresIn: '15m' },
      }),
      inject: [ConfigService],
    }),
    // Other modules
  ],
})
export class AppModule {}
```

**Multiple JWT Configurations:**

```typescript
// Different tokens for access and refresh
JwtModule.register({
  secret: process.env.JWT_ACCESS_SECRET,
  signOptions: { expiresIn: '15m' },
})

// In service, use custom options
this.jwtService.sign(payload, {
  secret: process.env.JWT_REFRESH_SECRET,
  expiresIn: '7d',
});
```

**Interview Tip**: Install `@nestjs/jwt`, `@nestjs/passport`, `passport-jwt`. Use `JwtModule.register()` for sync config or `registerAsync()` for async (with ConfigService). Set `secret` and `expiresIn`. Always use environment variables for secrets. Use `global: true` to make JwtService available everywhere.

</details>
### 7. How do you generate and sign JWT tokens using `JwtService`?

<details>
<summary>Answer</summary>

**JwtService** provides methods to sign (create) and verify JWT tokens.

**Basic Token Generation:**

```typescript
// auth.service.ts
import { Injectable } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';

@Injectable()
export class AuthService {
  constructor(private jwtService: JwtService) {}

  async login(user: any) {
    // Define payload (claims)
    const payload = { 
      sub: user.id,        // Subject: user ID
      username: user.username,
      email: user.email,
    };

    // Sign and return token
    return {
      access_token: this.jwtService.sign(payload),
      user: {
        id: user.id,
        username: user.username,
        email: user.email,
      },
    };
  }
}
```

**Custom Options (Override Module Config):**

```typescript
async generateAccessToken(user: User) {
  const payload = { sub: user.id, username: user.username };
  
  return this.jwtService.sign(payload, {
    secret: 'custom-secret',  // Override module secret
    expiresIn: '15m',         // Override expiration
    issuer: 'my-app',
    audience: 'api-users',
  });
}
```

**Generate Multiple Token Types:**

```typescript
async generateTokens(user: User) {
  const payload = { sub: user.id, username: user.username };

  // Access token - short lived
  const accessToken = this.jwtService.sign(payload, {
    secret: process.env.JWT_ACCESS_SECRET,
    expiresIn: '15m',
  });

  // Refresh token - long lived
  const refreshToken = this.jwtService.sign(payload, {
    secret: process.env.JWT_REFRESH_SECRET,
    expiresIn: '7d',
  });

  return {
    access_token: accessToken,
    refresh_token: refreshToken,
  };
}
```

**Async Token Generation:**

```typescript
async signAsync(payload: any, options?: JwtSignOptions): Promise<string> {
  return this.jwtService.signAsync(payload, options);
}
```

**Complete Login Example:**

```typescript
// auth.service.ts
@Injectable()
export class AuthService {
  constructor(
    private jwtService: JwtService,
    private usersService: UsersService,
  ) {}

  async validateUser(username: string, password: string): Promise<any> {
    const user = await this.usersService.findByUsername(username);
    
    if (user && await bcrypt.compare(password, user.password)) {
      const { password, ...result } = user;
      return result;
    }
    return null;
  }

  async login(user: any) {
    const payload = { 
      sub: user.id,
      username: user.username,
      email: user.email,
      roles: user.roles,
    };

    return {
      access_token: this.jwtService.sign(payload),
      expires_in: 3600, // 1 hour in seconds
      token_type: 'Bearer',
    };
  }
}

// auth.controller.ts
@Controller('auth')
export class AuthController {
  constructor(private authService: AuthService) {}

  @Post('login')
  @UseGuards(AuthGuard('local'))
  async login(@Request() req) {
    return this.authService.login(req.user);
  }
}
```

**Verify/Decode Token:**

```typescript
// Verify and decode
async verifyToken(token: string) {
  try {
    const payload = this.jwtService.verify(token);
    return payload;
  } catch (error) {
    throw new UnauthorizedException('Invalid or expired token');
  }
}

// Decode without verification (not recommended for auth)
decodeToken(token: string) {
  return this.jwtService.decode(token);
}
```

**Interview Tip**: Use `jwtService.sign(payload)` to generate token. Payload contains user data (claims). Use `sign()` for sync, `signAsync()` for async. Can override options per token. Use `verify()` to validate token. Never include sensitive data (passwords) in payload.

</details>

### 8. What is a JWT secret and expiration time?

<details>
<summary>Answer</summary>

**JWT Secret**: Private key used to sign and verify tokens. **Expiration**: Time when token becomes invalid.

**JWT Secret:**

```typescript
// ❌ BAD: Hardcoded secret
JwtModule.register({
  secret: 'my-secret-123',
  signOptions: { expiresIn: '1h' },
})

// ✅ GOOD: Environment variable
JwtModule.registerAsync({
  imports: [ConfigModule],
  useFactory: (configService: ConfigService) => ({
    secret: configService.get<string>('JWT_SECRET'),
    signOptions: { expiresIn: '1h' },
  }),
  inject: [ConfigService],
})
```

**Environment Variables (.env):**

```env
# Minimum 32 characters, random string
JWT_SECRET=8f3c9d4e6b7a2f1e9c8d5b6a7e3f2c1d9e8f7a6b5c4d3e2f1a0b9c8d7e6f5a4b3
JWT_EXPIRES_IN=15m
JWT_REFRESH_SECRET=2c1d9e8f7a6b5c4d3e2f1a0b9c8d7e6f5a4b38f3c9d4e6b7a2f1e9c8d5b6a7e3f
JWT_REFRESH_EXPIRES_IN=7d
```

**Generating Secure Secret:**

```bash
# Node.js
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"

# OpenSSL
openssl rand -base64 32
```

**Expiration Time:**

```typescript
// Different expiration formats
JwtModule.register({
  secret: process.env.JWT_SECRET,
  signOptions: {
    expiresIn: '1h',      // 1 hour
    // expiresIn: '15m',  // 15 minutes
    // expiresIn: '7d',   // 7 days
    // expiresIn: 3600,   // 3600 seconds (1 hour)
  },
})
```

**Token Expiration Best Practices:**

| Token Type | Recommended Expiration | Use Case |
|------------|------------------------|----------|
| Access Token | 15 minutes - 1 hour | API requests |
| Refresh Token | 7 days - 30 days | Token renewal |
| Email Verification | 24 hours | Account verification |
| Password Reset | 1 hour | Password reset |

**Checking Token Expiration:**

```typescript
@Injectable()
export class AuthService {
  constructor(private jwtService: JwtService) {}

  async validateToken(token: string) {
    try {
      const payload = this.jwtService.verify(token);
      
      // Check if token is expired
      const now = Math.floor(Date.now() / 1000);
      if (payload.exp && payload.exp < now) {
        throw new UnauthorizedException('Token expired');
      }
      
      return payload;
    } catch (error) {
      if (error.name === 'TokenExpiredError') {
        throw new UnauthorizedException('Token has expired');
      }
      throw new UnauthorizedException('Invalid token');
    }
  }
}
```

**Custom Expiration Per Token:**

```typescript
// Different expiration for different tokens
async generateTokens(user: User) {
  const payload = { sub: user.id, username: user.username };

  // Short-lived access token
  const accessToken = this.jwtService.sign(payload, {
    expiresIn: '15m',
  });

  // Long-lived refresh token
  const refreshToken = this.jwtService.sign(payload, {
    secret: process.env.JWT_REFRESH_SECRET,
    expiresIn: '7d',
  });

  // Email verification token
  const verificationToken = this.jwtService.sign(
    { sub: user.id, purpose: 'email-verification' },
    { expiresIn: '24h' }
  );

  return { accessToken, refreshToken, verificationToken };
}
```

**Interview Tip**: Secret is used to sign/verify tokens. Use strong, random secret (min 32 chars) from environment variables. Expiration prevents indefinite token validity. Access tokens: 15min-1h. Refresh tokens: 7-30 days. Token expires when `exp` claim < current time.

</details>

### 9. What is the difference between access tokens and refresh tokens?

<details>
<summary>Answer</summary>

**Access Token**: Short-lived token for API requests. **Refresh Token**: Long-lived token to get new access tokens.

**Comparison:**

| Feature | Access Token | Refresh Token |
|---------|-------------|---------------|
| **Lifetime** | Short (15min-1h) | Long (7-30 days) |
| **Purpose** | API authentication | Renew access token |
| **Storage** | Memory/localStorage | httpOnly cookie |
| **Included in** | Every API request | Only token refresh |
| **Risk if stolen** | Low (expires fast) | High (long validity) |
| **Can revoke** | No (stateless) | Yes (stored in DB) |

**Implementation:**

```typescript
// auth.service.ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { RefreshToken } from './entities/refresh-token.entity';

@Injectable()
export class AuthService {
  constructor(
    private jwtService: JwtService,
    @InjectRepository(RefreshToken)
    private refreshTokenRepository: Repository<RefreshToken>,
  ) {}

  async login(user: any) {
    const tokens = await this.generateTokens(user);
    
    // Store refresh token in database
    await this.saveRefreshToken(user.id, tokens.refreshToken);
    
    return tokens;
  }

  async generateTokens(user: any) {
    const payload = { 
      sub: user.id, 
      username: user.username,
      roles: user.roles,
    };

    // Access token - short lived
    const accessToken = this.jwtService.sign(payload, {
      secret: process.env.JWT_ACCESS_SECRET,
      expiresIn: '15m',
    });

    // Refresh token - long lived
    const refreshToken = this.jwtService.sign(
      { sub: user.id },  // Less info in refresh token
      {
        secret: process.env.JWT_REFRESH_SECRET,
        expiresIn: '7d',
      }
    );

    return {
      access_token: accessToken,
      refresh_token: refreshToken,
      expires_in: 900, // 15 minutes in seconds
    };
  }

  async saveRefreshToken(userId: number, token: string) {
    // Hash token before storing
    const hashedToken = await bcrypt.hash(token, 10);
    
    await this.refreshTokenRepository.save({
      userId,
      token: hashedToken,
      expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000), // 7 days
    });
  }

  async refreshAccessToken(refreshToken: string) {
    try {
      // Verify refresh token
      const payload = this.jwtService.verify(refreshToken, {
        secret: process.env.JWT_REFRESH_SECRET,
      });

      // Check if refresh token exists in database
      const storedToken = await this.refreshTokenRepository.findOne({
        where: { userId: payload.sub },
      });

      if (!storedToken) {
        throw new UnauthorizedException('Invalid refresh token');
      }

      // Generate new access token
      const newAccessToken = this.jwtService.sign({
        sub: payload.sub,
        username: payload.username,
        roles: payload.roles,
      }, {
        secret: process.env.JWT_ACCESS_SECRET,
        expiresIn: '15m',
      });

      return {
        access_token: newAccessToken,
        expires_in: 900,
      };
    } catch (error) {
      throw new UnauthorizedException('Invalid or expired refresh token');
    }
  }

  async revokeRefreshToken(userId: number) {
    // Delete refresh token from database (logout)
    await this.refreshTokenRepository.delete({ userId });
  }
}
```

**Refresh Token Entity:**

```typescript
// entities/refresh-token.entity.ts
import { Entity, Column, PrimaryGeneratedColumn, CreateDateColumn } from 'typeorm';

@Entity()
export class RefreshToken {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  userId: number;

  @Column()
  token: string;  // Hashed

  @Column()
  expiresAt: Date;

  @CreateDateColumn()
  createdAt: Date;
}
```

**Controller:**

```typescript
// auth.controller.ts
@Controller('auth')
export class AuthController {
  constructor(private authService: AuthService) {}

  @Post('login')
  @UseGuards(AuthGuard('local'))
  async login(@Request() req) {
    return this.authService.login(req.user);
  }

  @Post('refresh')
  async refresh(@Body('refresh_token') refreshToken: string) {
    return this.authService.refreshAccessToken(refreshToken);
  }

  @Post('logout')
  @UseGuards(AuthGuard('jwt'))
  async logout(@Request() req) {
    await this.authService.revokeRefreshToken(req.user.userId);
    return { message: 'Logged out successfully' };
  }
}
```

**Client Usage:**

```javascript
// Login
const response = await fetch('/auth/login', {
  method: 'POST',
  body: JSON.stringify({ username, password }),
});
const { access_token, refresh_token } = await response.json();

// Store tokens
localStorage.setItem('access_token', access_token);
localStorage.setItem('refresh_token', refresh_token);

// API request with access token
const data = await fetch('/api/profile', {
  headers: { Authorization: `Bearer ${access_token}` },
});

// When access token expires, refresh it
const newTokens = await fetch('/auth/refresh', {
  method: 'POST',
  body: JSON.stringify({ refresh_token }),
});
```

**Interview Tip**: Access token: short-lived (15min-1h), used for API requests. Refresh token: long-lived (7-30d), used to get new access tokens. Store refresh tokens in database for revocation. Refresh token only contains user ID. Access token contains full user info and roles.

</details>

## Local Strategy (Username/Password)

### 10. How do you implement Local Strategy for login?

<details>
<summary>Answer</summary>

**Local Strategy** validates username/password credentials using Passport.

**Installation:**

```bash
npm install passport-local
npm install @types/passport-local --save-dev
```

**1. Create Local Strategy:**

```typescript
// strategies/local.strategy.ts
import { Strategy } from 'passport-local';
import { PassportStrategy } from '@nestjs/passport';
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { AuthService } from '../auth.service';

@Injectable()
export class LocalStrategy extends PassportStrategy(Strategy) {
  constructor(private authService: AuthService) {
    super({
      usernameField: 'email',    // Default is 'username'
      passwordField: 'password', // Default is 'password'
    });
  }

  // Passport calls this method automatically
  async validate(email: string, password: string): Promise<any> {
    const user = await this.authService.validateUser(email, password);
    
    if (!user) {
      throw new UnauthorizedException('Invalid email or password');
    }
    
    // This user object will be attached to request: req.user
    return user;
  }
}
```

**2. Implement Auth Service:**

```typescript
// auth.service.ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { UsersService } from '../users/users.service';
import * as bcrypt from 'bcrypt';

@Injectable()
export class AuthService {
  constructor(private usersService: UsersService) {}

  async validateUser(email: string, password: string): Promise<any> {
    // Find user by email
    const user = await this.usersService.findByEmail(email);
    
    if (!user) {
      return null;
    }

    // Compare passwords
    const isPasswordValid = await bcrypt.compare(password, user.password);
    
    if (!isPasswordValid) {
      return null;
    }

    // Remove password from result
    const { password: _, ...result } = user;
    return result;
  }
}
```

**3. Register Strategy in Module:**

```typescript
// auth.module.ts
import { Module } from '@nestjs/common';
import { PassportModule } from '@nestjs/passport';
import { LocalStrategy } from './strategies/local.strategy';
import { AuthService } from './auth.service';
import { AuthController } from './auth.controller';
import { UsersModule } from '../users/users.module';

@Module({
  imports: [
    UsersModule,
    PassportModule,
  ],
  providers: [AuthService, LocalStrategy],
  controllers: [AuthController],
})
export class AuthModule {}
```

**4. Use in Controller:**

```typescript
// auth.controller.ts
import { Controller, Post, UseGuards, Request, Body } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';
import { AuthService } from './auth.service';

@Controller('auth')
export class AuthController {
  constructor(private authService: AuthService) {}

  @Post('login')
  @UseGuards(AuthGuard('local'))  // Uses LocalStrategy
  async login(@Request() req) {
    // req.user comes from LocalStrategy.validate()
    return {
      message: 'Login successful',
      user: req.user,
    };
  }
}
```

**Complete Flow:**

1. Client sends POST to `/auth/login` with email and password
2. `AuthGuard('local')` activates LocalStrategy
3. LocalStrategy extracts email/password from request body
4. Calls `validate(email, password)` method
5. `validate()` calls `authService.validateUser()`
6. If valid, user attached to `req.user`
7. Controller receives request with `req.user`

**Interview Tip**: LocalStrategy extends `PassportStrategy(Strategy)` from 'passport-local'. Constructor configures field names. `validate()` method checks credentials. Return user object (becomes `req.user`). Throw `UnauthorizedException` if invalid. Use `@UseGuards(AuthGuard('local'))` on login route.

</details>

### 11. What is the `validate()` method in Passport strategies?

<details>
<summary>Answer</summary>

**`validate()` method** is called by Passport after initial verification to validate user and return user object.

**How validate() Works:**

1. Passport extracts credentials (username/password, JWT, OAuth token)
2. Passport does initial verification (JWT signature, token format)
3. Passport calls your `validate()` method
4. You perform additional validation (check database, permissions)
5. Return user object → attached to `req.user`
6. Throw exception → authentication fails

**Local Strategy validate():**

```typescript
@Injectable()
export class LocalStrategy extends PassportStrategy(Strategy) {
  constructor(private authService: AuthService) {
    super();
  }

  // Called after Passport extracts username/password from body
  async validate(username: string, password: string): Promise<any> {
    // Check if user exists and password matches
    const user = await this.authService.validateUser(username, password);
    
    if (!user) {
      throw new UnauthorizedException('Invalid credentials');
    }
    
    // Returned user is attached to request as req.user
    return user;
  }
}
```

**JWT Strategy validate():**

```typescript
@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      secretOrKey: process.env.JWT_SECRET,
    });
  }

  // Called after Passport verifies JWT signature
  async validate(payload: any) {
    // Payload is the decoded JWT
    // You can add additional checks here (check if user still exists, is active, etc.)
    
    return { 
      userId: payload.sub, 
      username: payload.username,
      roles: payload.roles,
    };
  }
}
```

**OAuth Strategy validate():**

```typescript
@Injectable()
export class GoogleStrategy extends PassportStrategy(Strategy, 'google') {
  constructor(private usersService: UsersService) {
    super({
      clientID: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
      callbackURL: 'http://localhost:3000/auth/google/callback',
      scope: ['email', 'profile'],
    });
  }

  // Called after Google OAuth verification
  async validate(
    accessToken: string,
    refreshToken: string,
    profile: any,
  ): Promise<any> {
    const { emails, name, photos } = profile;
    
    // Check if user exists, create if not
    let user = await this.usersService.findByEmail(emails[0].value);
    
    if (!user) {
      user = await this.usersService.create({
        email: emails[0].value,
        firstName: name.givenName,
        lastName: name.familyName,
        picture: photos[0].value,
      });
    }
    
    return user;
  }
}
```

**Advanced validate() with Database Check:**

```typescript
@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(
    private usersService: UsersService,
  ) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      secretOrKey: process.env.JWT_SECRET,
    });
  }

  async validate(payload: any) {
    // Check if user still exists in database
    const user = await this.usersService.findById(payload.sub);
    
    if (!user) {
      throw new UnauthorizedException('User not found');
    }

    // Check if user is active
    if (!user.isActive) {
      throw new UnauthorizedException('User is inactive');
    }

    // Check if user's password was changed after token issued
    if (user.passwordChangedAt && payload.iat < user.passwordChangedAt) {
      throw new UnauthorizedException('Password changed, please login again');
    }

    return {
      userId: user.id,
      username: user.username,
      email: user.email,
      roles: user.roles,
    };
  }
}
```

**Accessing req.user in Controller:**

```typescript
@Controller('profile')
export class ProfileController {
  @Get()
  @UseGuards(AuthGuard('jwt'))
  getProfile(@Request() req) {
    // req.user is what validate() returned
    return req.user;
  }

  @Get('custom')
  @UseGuards(AuthGuard('jwt'))
  getCustom(@User() user: any) {  // Custom decorator
    return user;
  }
}
```

**Interview Tip**: `validate()` is called after Passport's initial verification. For LocalStrategy: validates username/password. For JwtStrategy: validates decoded JWT payload. For OAuth: processes profile data. Return user object → becomes `req.user`. Throw exception → authentication fails. Can add additional checks (DB lookup, account status).

</details>

### 12. How do you validate username and password?

<details>
<summary>Answer</summary>

Use **bcrypt** to hash passwords during registration and compare during login.

**Installation:**

```bash
npm install bcrypt
npm install @types/bcrypt --save-dev
```

**1. Password Hashing During Registration:**

```typescript
// users.service.ts
import { Injectable, ConflictException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import * as bcrypt from 'bcrypt';
import { User } from './entities/user.entity';
import { CreateUserDto } from './dto/create-user.dto';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>,
  ) {}

  async create(createUserDto: CreateUserDto): Promise<User> {
    // Check if user already exists
    const existingUser = await this.usersRepository.findOne({
      where: { email: createUserDto.email },
    });

    if (existingUser) {
      throw new ConflictException('Email already exists');
    }

    // Hash password with salt rounds (10-12 recommended)
    const saltRounds = 10;
    const hashedPassword = await bcrypt.hash(createUserDto.password, saltRounds);

    // Create user with hashed password
    const user = this.usersRepository.create({
      ...createUserDto,
      password: hashedPassword,
    });

    const savedUser = await this.usersRepository.save(user);

    // Remove password from response
    const { password, ...result } = savedUser;
    return result as User;
  }

  async findByEmail(email: string): Promise<User | undefined> {
    return this.usersRepository.findOne({ where: { email } });
  }
}
```

**2. Password Validation During Login:**

```typescript
// auth.service.ts
import { Injectable } from '@nestjs/common';
import { UsersService } from '../users/users.service';
import * as bcrypt from 'bcrypt';

@Injectable()
export class AuthService {
  constructor(private usersService: UsersService) {}

  async validateUser(email: string, password: string): Promise<any> {
    // Find user by email
    const user = await this.usersService.findByEmail(email);

    if (!user) {
      return null; // User not found
    }

    // Compare plain text password with hashed password
    const isPasswordValid = await bcrypt.compare(password, user.password);

    if (!isPasswordValid) {
      return null; // Invalid password
    }

    // Return user without password
    const { password: _, ...result } = user;
    return result;
  }
}
```

**3. Complete Registration Flow:**

```typescript
// auth.controller.ts
@Controller('auth')
export class AuthController {
  constructor(
    private authService: AuthService,
    private usersService: UsersService,
  ) {}

  @Post('register')
  async register(@Body() createUserDto: CreateUserDto) {
    // Validate input
    if (!createUserDto.email || !createUserDto.password) {
      throw new BadRequestException('Email and password are required');
    }

    if (createUserDto.password.length < 8) {
      throw new BadRequestException('Password must be at least 8 characters');
    }

    // Create user (password hashed in service)
    const user = await this.usersService.create(createUserDto);

    return {
      message: 'User registered successfully',
      user,
    };
  }

  @Post('login')
  @UseGuards(AuthGuard('local'))
  async login(@Request() req) {
    return this.authService.login(req.user);
  }
}
```

**4. Password Validation DTO:**

```typescript
// dto/create-user.dto.ts
import { IsEmail, IsString, MinLength, MaxLength } from 'class-validator';

export class CreateUserDto {
  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8, { message: 'Password must be at least 8 characters' })
  @MaxLength(50, { message: 'Password must not exceed 50 characters' })
  password: string;

  @IsString()
  firstName: string;

  @IsString()
  lastName: string;
}
```

**5. User Entity:**

```typescript
// entities/user.entity.ts
import { Entity, Column, PrimaryGeneratedColumn, CreateDateColumn } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ unique: true })
  email: string;

  @Column()
  password: string;  // Stored as hashed

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column({ default: true })
  isActive: boolean;

  @CreateDateColumn()
  createdAt: Date;
}
```

**Security Best Practices:**

```typescript
// Advanced password validation
async validatePassword(password: string, user: User): Promise<boolean> {
  // 1. Compare password
  const isValid = await bcrypt.compare(password, user.password);
  
  if (!isValid) {
    return false;
  }

  // 2. Check if password needs rehashing (security improvement)
  const needsRehash = await this.needsRehash(user.password);
  if (needsRehash) {
    await this.rehashPassword(user, password);
  }

  return true;
}

private async needsRehash(hashedPassword: string): Promise<boolean> {
  // Check if hash uses old/weak salt rounds
  const currentRounds = 12;
  // Implementation depends on bcrypt version
  return false; // Simplified
}

private async rehashPassword(user: User, password: string): Promise<void> {
  const newHash = await bcrypt.hash(password, 12);
  user.password = newHash;
  await this.usersRepository.save(user);
}
```

**Interview Tip**: Use bcrypt to hash passwords. `bcrypt.hash(password, saltRounds)` during registration (10-12 rounds). `bcrypt.compare(plainText, hashed)` during login. NEVER store plain text passwords. Return user without password field. Validate password length/complexity with DTOs.

</details>

## JWT Strategy

### 13. How do you create a `JwtStrategy` class?

<details>
<summary>Answer</summary>

**JwtStrategy** extends `PassportStrategy` to validate JWT tokens from Authorization header.

**Installation:**

```bash
npm install passport-jwt
npm install @types/passport-jwt --save-dev
```

**Complete JwtStrategy Implementation:**

```typescript
// strategies/jwt.strategy.ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(private configService: ConfigService) {
    super({
      // Extract JWT from Authorization header: "Bearer <token>"
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      
      // Reject expired tokens
      ignoreExpiration: false,
      
      // Secret key to verify signature
      secretOrKey: configService.get<string>('JWT_SECRET'),
    });
  }

  // Called after Passport verifies JWT signature
  async validate(payload: any) {
    // Payload contains decoded JWT claims
    // Return user object that will be attached to req.user
    return {
      userId: payload.sub,
      username: payload.username,
      email: payload.email,
      roles: payload.roles,
    };
  }
}
```

**Register in Module:**

```typescript
// auth.module.ts
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { PassportModule } from '@nestjs/passport';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { JwtStrategy } from './strategies/jwt.strategy';
import { AuthService } from './auth.service';
import { AuthController } from './auth.controller';

@Module({
  imports: [
    ConfigModule,
    PassportModule,
    JwtModule.registerAsync({
      imports: [ConfigModule],
      useFactory: (configService: ConfigService) => ({
        secret: configService.get<string>('JWT_SECRET'),
        signOptions: {
          expiresIn: configService.get<string>('JWT_EXPIRES_IN', '1h'),
        },
      }),
      inject: [ConfigService],
    }),
  ],
  providers: [AuthService, JwtStrategy],
  controllers: [AuthController],
  exports: [AuthService],
})
export class AuthModule {}
```

**Using JwtStrategy in Routes:**

```typescript
// profile.controller.ts
import { Controller, Get, UseGuards, Request } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Controller('profile')
export class ProfileController {
  @Get()
  @UseGuards(AuthGuard('jwt'))  // Uses JwtStrategy
  getProfile(@Request() req) {
    // req.user comes from JwtStrategy.validate()
    return req.user;
  }
}
```

**Advanced JwtStrategy with Database Validation:**

```typescript
// strategies/jwt.strategy.ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';
import { UsersService } from '../users/users.service';

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

  async validate(payload: any) {
    // Additional validation: Check if user exists in database
    const user = await this.usersService.findById(payload.sub);

    if (!user) {
      throw new UnauthorizedException('User not found');
    }

    // Check if user is active
    if (!user.isActive) {
      throw new UnauthorizedException('Account is deactivated');
    }

    // Check if token was issued before password change
    if (user.passwordChangedAt) {
      const passwordChangedTimestamp = Math.floor(
        user.passwordChangedAt.getTime() / 1000,
      );
      
      if (payload.iat < passwordChangedTimestamp) {
        throw new UnauthorizedException(
          'Password changed recently, please login again',
        );
      }
    }

    return {
      userId: user.id,
      username: user.username,
      email: user.email,
      roles: user.roles,
    };
  }
}
```

**Interview Tip**: JwtStrategy extends `PassportStrategy(Strategy)` from 'passport-jwt'. Configure in constructor: `jwtFromRequest`, `secretOrKey`, `ignoreExpiration`. Passport verifies JWT signature automatically. `validate()` receives decoded payload. Return user object (becomes `req.user`). Can add DB validation in `validate()`. Use `@UseGuards(AuthGuard('jwt'))` to protect routes.

</details>

### 14. How do you extract JWT from Authorization header using `ExtractJwt.fromAuthHeaderAsBearerToken()`?

<details>
<summary>Answer</summary>

**`ExtractJwt.fromAuthHeaderAsBearerToken()`** extracts JWT from `Authorization: Bearer <token>` header.

**Basic Usage:**

```typescript
// strategies/jwt.strategy.ts
import { ExtractJwt, Strategy } from 'passport-jwt';
import { PassportStrategy } from '@nestjs/passport';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      // Extracts JWT from: Authorization: Bearer <token>
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      secretOrKey: 'your-secret-key',
    });
  }

  async validate(payload: any) {
    return { userId: payload.sub, username: payload.username };
  }
}
```

**Client Request Format:**

```javascript
// Client side: Add JWT to Authorization header
fetch('/api/profile', {
  method: 'GET',
  headers: {
    'Authorization': `Bearer ${accessToken}`,
    'Content-Type': 'application/json',
  },
});
```

**Other Extraction Methods:**

```typescript
// 1. From Bearer token in Authorization header (most common)
jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken()
// Header: Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

// 2. From custom header
jwtFromRequest: ExtractJwt.fromHeader('x-auth-token')
// Header: x-auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

// 3. From query parameter
jwtFromRequest: ExtractJwt.fromUrlQueryParameter('token')
// URL: /api/profile?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

// 4. From body field
jwtFromRequest: ExtractJwt.fromBodyField('token')
// Body: { token: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." }

// 5. From cookie
jwtFromRequest: (req) => {
  if (req && req.cookies) {
    return req.cookies['jwt'];
  }
  return null;
}
// Cookie: jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

// 6. Multiple extractors (fallback)
jwtFromRequest: ExtractJwt.fromExtractors([
  ExtractJwt.fromAuthHeaderAsBearerToken(),
  ExtractJwt.fromUrlQueryParameter('token'),
  (req) => req.cookies?.jwt,
])
// Tries Bearer, then query, then cookie
```

**Custom Extractor:**

```typescript
// Custom extractor function
const cookieExtractor = (req: Request) => {
  let token = null;
  
  if (req && req.cookies) {
    token = req.cookies['access_token'];
  }
  
  return token;
};

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: cookieExtractor,
      secretOrKey: 'your-secret-key',
    });
  }
}
```

**Combined Extractor (Bearer or Cookie):**

```typescript
@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromExtractors([
        ExtractJwt.fromAuthHeaderAsBearerToken(),
        (req: Request) => {
          // Fallback to cookie if no Bearer token
          return req?.cookies?.access_token || null;
        },
      ]),
      secretOrKey: process.env.JWT_SECRET,
    });
  }

  async validate(payload: any) {
    return {
      userId: payload.sub,
      username: payload.username,
    };
  }
}
```

**Testing JWT Extraction:**

```bash
# cURL with Bearer token
curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  http://localhost:3000/api/profile

# Postman
# Tab: Authorization
# Type: Bearer Token
# Token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Interview Tip**: `ExtractJwt.fromAuthHeaderAsBearerToken()` extracts JWT from `Authorization: Bearer <token>` header. Most common method for APIs. Can use other extractors: `fromHeader()`, `fromUrlQueryParameter()`, `fromBodyField()`, or custom function. Use `fromExtractors([])` for multiple fallback options. Cookie extraction requires custom function.

</details>

### 15. How do you verify JWT tokens in the strategy?

<details>
<summary>Answer</summary>

Passport automatically verifies JWT signature using `secretOrKey`. You can add additional validation in `validate()`.

**Automatic Verification by Passport:**

```typescript
// strategies/jwt.strategy.ts
@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,  // Rejects expired tokens
      secretOrKey: process.env.JWT_SECRET,  // Used to verify signature
    });
  }

  // Called ONLY if signature is valid and token not expired
  async validate(payload: any) {
    return { userId: payload.sub, username: payload.username };
  }
}
```

**What Passport Verifies Automatically:**

1. **Signature**: Token signed with correct secret
2. **Expiration**: Token not expired (if `ignoreExpiration: false`)
3. **Format**: Valid JWT structure (header.payload.signature)

**Manual Token Verification:**

```typescript
// auth.service.ts
import { JwtService } from '@nestjs/jwt';

@Injectable()
export class AuthService {
  constructor(private jwtService: JwtService) {}

  async verifyToken(token: string) {
    try {
      // Verifies signature and expiration
      const payload = this.jwtService.verify(token, {
        secret: process.env.JWT_SECRET,
      });
      
      return payload;
    } catch (error) {
      if (error.name === 'TokenExpiredError') {
        throw new UnauthorizedException('Token has expired');
      }
      if (error.name === 'JsonWebTokenError') {
        throw new UnauthorizedException('Invalid token');
      }
      throw new UnauthorizedException('Token verification failed');
    }
  }

  async decodeToken(token: string) {
    // Decode without verification (not recommended for auth)
    return this.jwtService.decode(token);
  }
}
```

**Additional Validation in validate():**

```typescript
// strategies/jwt.strategy.ts
@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(
    private usersService: UsersService,
    private configService: ConfigService,
  ) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: configService.get('JWT_SECRET'),
    });
  }

  async validate(payload: any) {
    // Passport already verified signature and expiration
    
    // 1. Check if user still exists
    const user = await this.usersService.findById(payload.sub);
    if (!user) {
      throw new UnauthorizedException('User not found');
    }

    // 2. Check if user is active
    if (!user.isActive) {
      throw new UnauthorizedException('User account is deactivated');
    }

    // 3. Check if token was issued before password change
    if (user.passwordChangedAt) {
      const passwordChangedTimestamp = Math.floor(
        user.passwordChangedAt.getTime() / 1000,
      );
      
      if (payload.iat < passwordChangedTimestamp) {
        throw new UnauthorizedException(
          'Token invalid due to password change',
        );
      }
    }

    // 4. Check if token is blacklisted (logout/revoked)
    const isBlacklisted = await this.checkTokenBlacklist(payload.jti);
    if (isBlacklisted) {
      throw new UnauthorizedException('Token has been revoked');
    }

    // 5. Verify token audience/issuer
    if (payload.iss !== 'my-app' || payload.aud !== 'api-users') {
      throw new UnauthorizedException('Invalid token claims');
    }

    return {
      userId: user.id,
      username: user.username,
      email: user.email,
      roles: user.roles,
    };
  }

  private async checkTokenBlacklist(jti: string): Promise<boolean> {
    // Check Redis or database for blacklisted tokens
    return false; // Simplified
  }
}
```

**Token Verification Errors:**

```typescript
try {
  const payload = this.jwtService.verify(token);
} catch (error) {
  switch (error.name) {
    case 'TokenExpiredError':
      // Token expired: payload.exp < current time
      throw new UnauthorizedException('Token expired');
      
    case 'JsonWebTokenError':
      // Invalid token: bad signature, malformed, etc.
      throw new UnauthorizedException('Invalid token');
      
    case 'NotBeforeError':
      // Token used before nbf (not before) claim
      throw new UnauthorizedException('Token not yet valid');
      
    default:
      throw new UnauthorizedException('Token verification failed');
  }
}
```

**Asymmetric Keys (RS256):**

```typescript
// Using public/private key pair
@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      secretOrPublicKey: fs.readFileSync('path/to/public.key'),  // Public key
      algorithms: ['RS256'],
    });
  }

  async validate(payload: any) {
    return { userId: payload.sub };
  }
}
```

**Interview Tip**: Passport auto-verifies JWT signature and expiration using `secretOrKey`. Set `ignoreExpiration: false` to reject expired tokens. `validate()` is only called if verification succeeds. Add custom validation in `validate()`: check user exists, is active, password not changed. Use try-catch with `jwtService.verify()` for manual verification. Check error types: TokenExpiredError, JsonWebTokenError.

</details>

### 16. What should the `validate()` method return?

<details>
<summary>Answer</summary>

**`validate()` should return a user object** that Passport attaches to `req.user`.

**Basic Return:**

```typescript
@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      secretOrKey: process.env.JWT_SECRET,
    });
  }

  async validate(payload: any) {
    // Return minimal user info
    return {
      userId: payload.sub,
      username: payload.username,
    };
  }
}

// In controller
@Get('profile')
@UseGuards(AuthGuard('jwt'))
getProfile(@Request() req) {
  console.log(req.user); // { userId: 1, username: 'john' }
  return req.user;
}
```

**Complete User Object:**

```typescript
async validate(payload: any) {
  return {
    userId: payload.sub,
    username: payload.username,
    email: payload.email,
    roles: payload.roles,
    permissions: payload.permissions,
  };
}
```

**With Database Lookup:**

```typescript
@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(
    private usersService: UsersService,
  ) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      secretOrKey: process.env.JWT_SECRET,
    });
  }

  async validate(payload: any) {
    // Fetch fresh user data from database
    const user = await this.usersService.findById(payload.sub);

    if (!user) {
      throw new UnauthorizedException('User not found');
    }

    // Return complete user object
    return {
      userId: user.id,
      username: user.username,
      email: user.email,
      roles: user.roles,
      firstName: user.firstName,
      lastName: user.lastName,
      isActive: user.isActive,
    };
  }
}
```

**Return Types:**

```typescript
// 1. Simple object
return { userId: 1, username: 'john' };

// 2. User entity
return user;  // TypeORM entity

// 3. Custom user type
interface JwtPayload {
  userId: number;
  username: string;
  email: string;
  roles: string[];
}

async validate(payload: any): Promise<JwtPayload> {
  return {
    userId: payload.sub,
    username: payload.username,
    email: payload.email,
    roles: payload.roles || [],
  };
}

// 4. With additional properties
return {
  ...user,
  permissions: await this.getPermissions(user.roles),
  lastLogin: new Date(),
};
```

**Returning null or throwing exception:**

```typescript
async validate(payload: any) {
  const user = await this.usersService.findById(payload.sub);

  // ❌ Returning null or undefined causes 401 Unauthorized
  if (!user) {
    return null;  // Passport treats as authentication failure
  }

  // ✅ Better: Throw explicit exception
  if (!user) {
    throw new UnauthorizedException('User not found');
  }

  // ✅ Best: Check account status
  if (!user.isActive) {
    throw new UnauthorizedException('Account is deactivated');
  }

  return user;
}
```

**Accessing in Controllers:**

```typescript
// Method 1: @Request() decorator
@Get('profile')
@UseGuards(AuthGuard('jwt'))
getProfile(@Request() req) {
  return req.user;  // Object from validate()
}

// Method 2: Custom @User() decorator
@Get('profile')
@UseGuards(AuthGuard('jwt'))
getProfile(@User() user: any) {
  return user;  // Same object
}

// Method 3: Destructuring
@Get('profile')
@UseGuards(AuthGuard('jwt'))
getProfile(@User('userId') userId: number) {
  return { userId };  // Extract specific property
}
```

**Custom User Decorator:**

```typescript
// decorators/user.decorator.ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const User = createParamDecorator(
  (data: string, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const user = request.user;

    // Return specific property if data provided
    return data ? user?.[data] : user;
  },
);

// Usage
@Get('posts')
@UseGuards(AuthGuard('jwt'))
getUserPosts(@User('userId') userId: number) {
  return this.postsService.findByUserId(userId);
}
```

**Interview Tip**: `validate()` returns user object attached to `req.user`. Include essential info: userId, username, email, roles. Can return JWT payload data or fetch from database. Return `null` or throw exception for failed validation. Use custom decorator to access user in controllers. Don't include sensitive data (password).

</details>

## Guards

### 17. How do you create a JWT Auth Guard using `AuthGuard('jwt')`?

<details>
<summary>Answer</summary>

**Auth Guard** applies authentication strategy to routes. `AuthGuard('jwt')` uses JwtStrategy.

**Basic Usage:**

```typescript
// Simply use AuthGuard with strategy name
import { Controller, Get, UseGuards } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Controller('profile')
export class ProfileController {
  @Get()
  @UseGuards(AuthGuard('jwt'))  // Uses JwtStrategy
  getProfile(@Request() req) {
    return req.user;
  }
}
```

**Custom JWT Guard (Recommended):**

```typescript
// guards/jwt-auth.guard.ts
import { Injectable } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  // Can add custom logic here
}
```

**Usage with Custom Guard:**

```typescript
import { JwtAuthGuard } from './guards/jwt-auth.guard';

@Controller('posts')
export class PostsController {
  @Get()
  @UseGuards(JwtAuthGuard)  // Cleaner than AuthGuard('jwt')
  findAll() {
    return this.postsService.findAll();
  }
}
```

**Advanced Custom Guard with Error Handling:**

```typescript
// guards/jwt-auth.guard.ts
import {
  Injectable,
  ExecutionContext,
  UnauthorizedException,
} from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  // Override canActivate for custom logic before strategy
  canActivate(context: ExecutionContext) {
    // Add custom authentication logic
    return super.canActivate(context);
  }

  // Override handleRequest for custom error handling
  handleRequest(err: any, user: any, info: any) {
    // Handle different error types
    if (err || !user) {
      if (info?.name === 'TokenExpiredError') {
        throw new UnauthorizedException('Token has expired');
      }
      if (info?.name === 'JsonWebTokenError') {
        throw new UnauthorizedException('Invalid token');
      }
      throw err || new UnauthorizedException('Authentication failed');
    }
    return user;
  }
}
```

**Global Guard (Apply to All Routes):**

```typescript
// main.ts
import { NestFactory, Reflector } from '@nestjs/core';
import { AppModule } from './app.module';
import { JwtAuthGuard } from './auth/guards/jwt-auth.guard';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Apply JWT guard globally
  const reflector = app.get(Reflector);
  app.useGlobalGuards(new JwtAuthGuard(reflector));
  
  await app.listen(3000);
}
bootstrap();
```

**Or in app.module.ts:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { APP_GUARD } from '@nestjs/core';
import { JwtAuthGuard } from './auth/guards/jwt-auth.guard';

@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: JwtAuthGuard,
    },
  ],
})
export class AppModule {}
```

**Multiple Guards:**

```typescript
// Apply multiple guards (all must pass)
@Get('admin')
@UseGuards(JwtAuthGuard, RolesGuard)
@Roles('admin')
getAdminData() {
  return { message: 'Admin only data' };
}
```

**Guard with Dependency Injection:**

```typescript
// guards/jwt-auth.guard.ts
@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  constructor(
    private reflector: Reflector,
    private configService: ConfigService,
  ) {
    super();
  }

  canActivate(context: ExecutionContext) {
    // Use injected services
    const isEnabled = this.configService.get('AUTH_ENABLED');
    if (!isEnabled) {
      return true; // Skip authentication if disabled
    }
    
    return super.canActivate(context);
  }
}
```

**Interview Tip**: Use `@UseGuards(AuthGuard('jwt'))` to protect routes. Create custom guard extending `AuthGuard('jwt')` for cleaner code. Override `handleRequest()` for custom error handling. Override `canActivate()` for pre-validation logic. Use `APP_GUARD` for global application. Guards run before route handlers.

</details>

### 18. How do you apply Auth Guard to routes and controllers?

<details>
<summary>Answer</summary>

Apply guards at **method level**, **controller level**, or **globally**.

**1. Method Level (Single Route):**

```typescript
import { Controller, Get, Post, UseGuards } from '@nestjs/common';
import { JwtAuthGuard } from './guards/jwt-auth.guard';

@Controller('posts')
export class PostsController {
  // Public route - no guard
  @Get('public')
  getPublicPosts() {
    return this.postsService.findPublic();
  }

  // Protected route - requires auth
  @Get('my-posts')
  @UseGuards(JwtAuthGuard)
  getMyPosts(@User() user) {
    return this.postsService.findByUserId(user.userId);
  }

  // Protected route
  @Post()
  @UseGuards(JwtAuthGuard)
  create(@Body() createPostDto: CreatePostDto, @User() user) {
    return this.postsService.create(createPostDto, user.userId);
  }
}
```

**2. Controller Level (All Routes):**

```typescript
// All routes in this controller require authentication
@Controller('profile')
@UseGuards(JwtAuthGuard)
export class ProfileController {
  @Get()
  getProfile(@User() user) {
    return user;
  }

  @Put()
  updateProfile(@User() user, @Body() updateDto: UpdateProfileDto) {
    return this.profileService.update(user.userId, updateDto);
  }

  @Get('settings')
  getSettings(@User() user) {
    return this.profileService.getSettings(user.userId);
  }
}
```

**3. Global Application Level:**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { JwtAuthGuard } from './auth/guards/jwt-auth.guard';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Apply to all routes globally
  app.useGlobalGuards(new JwtAuthGuard());
  
  await app.listen(3000);
}
bootstrap();
```

**Or with dependency injection:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { APP_GUARD } from '@nestjs/core';
import { JwtAuthGuard } from './auth/guards/jwt-auth.guard';

@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: JwtAuthGuard,  // Now can use DI
    },
  ],
})
export class AppModule {}
```

**4. Multiple Guards (Both Must Pass):**

```typescript
@Controller('admin')
export class AdminController {
  // User must be authenticated AND have admin role
  @Get('users')
  @UseGuards(JwtAuthGuard, RolesGuard)
  @Roles('admin')
  getAllUsers() {
    return this.usersService.findAll();
  }

  // Multiple guards in array
  @Delete(':id')
  @UseGuards(JwtAuthGuard, RolesGuard, ThrottlerGuard)
  @Roles('admin')
  deleteUser(@Param('id') id: number) {
    return this.usersService.delete(id);
  }
}
```

**5. Mix Public and Protected Routes:**

```typescript
@Controller('posts')
export class PostsController {
  // Public - no auth required
  @Get()
  findAll() {
    return this.postsService.findAll();
  }

  // Public - no auth required
  @Get(':id')
  findOne(@Param('id') id: number) {
    return this.postsService.findOne(id);
  }

  // Protected - auth required
  @Post()
  @UseGuards(JwtAuthGuard)
  create(@Body() createPostDto, @User() user) {
    return this.postsService.create(createPostDto, user.userId);
  }

  // Protected - auth required
  @Put(':id')
  @UseGuards(JwtAuthGuard)
  update(@Param('id') id: number, @Body() updateDto, @User() user) {
    return this.postsService.update(id, updateDto, user.userId);
  }
}
```

**6. Guard Execution Order:**

```typescript
// Guards execute in order: Global → Controller → Method
@Controller('test')
@UseGuards(RolesGuard)  // 2. Runs second
export class TestController {
  @Get()
  @UseGuards(ThrottlerGuard)  // 3. Runs third
  test() {
    return 'OK';
  }
}

// In main.ts or app.module.ts
app.useGlobalGuards(new JwtAuthGuard());  // 1. Runs first
```

**7. Conditional Guard Application:**

```typescript
// Apply guard based on environment
@Controller('data')
export class DataController {
  constructor(private configService: ConfigService) {}

  @Get()
  @UseGuards(
    ...(process.env.NODE_ENV === 'production' ? [JwtAuthGuard] : [])
  )
  getData() {
    return this.dataService.findAll();
  }
}
```

**Interview Tip**: Apply guards with `@UseGuards()` decorator. Method level: single route. Controller level: all routes in controller. Global level: all routes in application (use `APP_GUARD`). Multiple guards: all must pass. Execution order: Global → Controller → Method. Use global + @Public() decorator for mixed auth.

</details>

### 19. How do you make certain routes public using custom decorator?

<details>
<summary>Answer</summary>

Use **metadata** with custom `@Public()` decorator to skip authentication on specific routes.

**1. Create Public Decorator:**

```typescript
// decorators/public.decorator.ts
import { SetMetadata } from '@nestjs/common';

export const IS_PUBLIC_KEY = 'isPublic';
export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);
```

**2. Update JWT Guard to Check Metadata:**

```typescript
// guards/jwt-auth.guard.ts
import { Injectable, ExecutionContext } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';
import { Reflector } from '@nestjs/core';
import { IS_PUBLIC_KEY } from '../decorators/public.decorator';

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  constructor(private reflector: Reflector) {
    super();
  }

  canActivate(context: ExecutionContext) {
    // Check if route is marked as public
    const isPublic = this.reflector.getAllAndOverride<boolean>(
      IS_PUBLIC_KEY,
      [
        context.getHandler(),  // Method level
        context.getClass(),    // Controller level
      ],
    );

    // Skip authentication for public routes
    if (isPublic) {
      return true;
    }

    // Run normal JWT authentication
    return super.canActivate(context);
  }
}
```

**3. Apply Global Guard + Public Decorator:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { APP_GUARD } from '@nestjs/core';
import { JwtAuthGuard } from './auth/guards/jwt-auth.guard';

@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: JwtAuthGuard,  // All routes protected by default
    },
  ],
})
export class AppModule {}
```

**4. Use @Public() Decorator:**

```typescript
// auth.controller.ts
import { Controller, Post, Body } from '@nestjs/common';
import { Public } from './decorators/public.decorator';

@Controller('auth')
export class AuthController {
  constructor(private authService: AuthService) {}

  // Public route - no authentication
  @Public()
  @Post('register')
  register(@Body() createUserDto: CreateUserDto) {
    return this.authService.register(createUserDto);
  }

  // Public route - no authentication
  @Public()
  @Post('login')
  login(@Body() loginDto: LoginDto) {
    return this.authService.login(loginDto);
  }

  // Protected route - requires authentication
  @Post('logout')
  logout(@User() user) {
    return this.authService.logout(user.userId);
  }
}
```

**5. Mixed Public/Protected Routes:**

```typescript
// posts.controller.ts
import { Controller, Get, Post, UseGuards } from '@nestjs/common';
import { Public } from '../decorators/public.decorator';

@Controller('posts')
export class PostsController {
  // Public - anyone can view
  @Public()
  @Get()
  findAll() {
    return this.postsService.findAll();
  }

  // Public - anyone can view single post
  @Public()
  @Get(':id')
  findOne(@Param('id') id: number) {
    return this.postsService.findOne(id);
  }

  // Protected - must be authenticated
  @Post()
  create(@Body() createPostDto, @User() user) {
    return this.postsService.create(createPostDto, user.userId);
  }

  // Protected - must be authenticated
  @Put(':id')
  update(@Param('id') id: number, @Body() updateDto, @User() user) {
    return this.postsService.update(id, updateDto, user.userId);
  }
}
```

**6. Public at Controller Level:**

```typescript
// Mark entire controller as public
@Controller('public')
@Public()
export class PublicController {
  @Get('health')
  healthCheck() {
    return { status: 'OK' };
  }

  @Get('info')
  getInfo() {
    return { version: '1.0.0' };
  }
}
```

**7. Alternative: @SkipAuth() Decorator:**

```typescript
// decorators/skip-auth.decorator.ts
import { SetMetadata } from '@nestjs/common';

export const SKIP_AUTH_KEY = 'skipAuth';
export const SkipAuth = () => SetMetadata(SKIP_AUTH_KEY, true);

// Usage
@Controller('api')
export class ApiController {
  @SkipAuth()
  @Get('public-data')
  getPublicData() {
    return { data: 'public' };
  }
}
```

**8. Complete Example:**

```typescript
// app structure
// 1. Global guard
@Module({
  providers: [
    { provide: APP_GUARD, useClass: JwtAuthGuard },
  ],
})
export class AppModule {}

// 2. Public decorator
export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);

// 3. Guard checks metadata
@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  constructor(private reflector: Reflector) {
    super();
  }

  canActivate(context: ExecutionContext) {
    const isPublic = this.reflector.getAllAndOverride(IS_PUBLIC_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);
    
    if (isPublic) return true;
    return super.canActivate(context);
  }
}

// 4. Controllers
@Controller('posts')
export class PostsController {
  @Public()  // Anyone can access
  @Get()
  findAll() {}

  @Post()  // Requires auth (global guard)
  create() {}
}
```

**Interview Tip**: Create `@Public()` decorator with `SetMetadata()`. Update guard to use `Reflector` and check metadata. Use `getAllAndOverride()` to check method and controller levels. Apply global guard + `@Public()` for mixed routes. This pattern is cleaner than applying guards individually. Metadata key should be a constant.

</details>

## Login/Register Flow

### 20. How do you implement user registration?

<details>
<summary>Answer</summary>

**User registration** involves validation, password hashing, and storing user in database.

**Complete Registration Implementation:**

```typescript
// dto/register.dto.ts
import { IsEmail, IsString, MinLength, MaxLength, Matches } from 'class-validator';

export class RegisterDto {
  @IsEmail({}, { message: 'Invalid email address' })
  email: string;

  @IsString()
  @MinLength(8, { message: 'Password must be at least 8 characters' })
  @MaxLength(50, { message: 'Password must not exceed 50 characters' })
  @Matches(
    /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/,
    { message: 'Password must contain uppercase, lowercase, number and special character' }
  )
  password: string;

  @IsString()
  @MinLength(2)
  @MaxLength(50)
  firstName: string;

  @IsString()
  @MinLength(2)
  @MaxLength(50)
  lastName: string;
}
```

```typescript
// entities/user.entity.ts
import { Entity, Column, PrimaryGeneratedColumn, CreateDateColumn, UpdateDateColumn } from 'typeorm';

@Entity('users')
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ unique: true })
  email: string;

  @Column()
  password: string;  // Will be hashed

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column({ default: true })
  isActive: boolean;

  @Column({ default: false })
  isEmailVerified: boolean;

  @Column({ type: 'simple-array', default: 'user' })
  roles: string[];

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}
```

```typescript
// users/users.service.ts
import { Injectable, ConflictException, BadRequestException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import * as bcrypt from 'bcrypt';
import { User } from './entities/user.entity';
import { RegisterDto } from '../auth/dto/register.dto';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>,
  ) {}

  async create(registerDto: RegisterDto): Promise<User> {
    // 1. Check if user already exists
    const existingUser = await this.usersRepository.findOne({
      where: { email: registerDto.email },
    });

    if (existingUser) {
      throw new ConflictException('Email already registered');
    }

    // 2. Hash password
    const saltRounds = 10;
    const hashedPassword = await bcrypt.hash(registerDto.password, saltRounds);

    // 3. Create user entity
    const user = this.usersRepository.create({
      ...registerDto,
      password: hashedPassword,
      roles: ['user'],  // Default role
    });

    // 4. Save to database
    const savedUser = await this.usersRepository.save(user);

    // 5. Remove password from response
    delete savedUser.password;
    return savedUser;
  }

  async findByEmail(email: string): Promise<User | undefined> {
    return this.usersRepository.findOne({ where: { email } });
  }

  async findById(id: number): Promise<User | undefined> {
    return this.usersRepository.findOne({ where: { id } });
  }
}
```

```typescript
// auth/auth.service.ts
import { Injectable } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { UsersService } from '../users/users.service';
import { RegisterDto } from './dto/register.dto';

@Injectable()
export class AuthService {
  constructor(
    private usersService: UsersService,
    private jwtService: JwtService,
  ) {}

  async register(registerDto: RegisterDto) {
    // Create user
    const user = await this.usersService.create(registerDto);

    // Generate JWT token
    const payload = {
      sub: user.id,
      email: user.email,
      roles: user.roles,
    };

    const accessToken = this.jwtService.sign(payload);

    return {
      message: 'Registration successful',
      user: {
        id: user.id,
        email: user.email,
        firstName: user.firstName,
        lastName: user.lastName,
      },
      access_token: accessToken,
    };
  }
}
```

```typescript
// auth/auth.controller.ts
import { Controller, Post, Body, HttpCode, HttpStatus } from '@nestjs/common';
import { Public } from './decorators/public.decorator';
import { AuthService } from './auth.service';
import { RegisterDto } from './dto/register.dto';

@Controller('auth')
export class AuthController {
  constructor(private authService: AuthService) {}

  @Public()
  @Post('register')
  @HttpCode(HttpStatus.CREATED)
  async register(@Body() registerDto: RegisterDto) {
    return this.authService.register(registerDto);
  }
}
```

**With Email Verification:**

```typescript
// auth.service.ts
@Injectable()
export class AuthService {
  constructor(
    private usersService: UsersService,
    private jwtService: JwtService,
    private mailService: MailService,
  ) {}

  async register(registerDto: RegisterDto) {
    // Create user
    const user = await this.usersService.create(registerDto);

    // Generate email verification token
    const verificationToken = this.jwtService.sign(
      { sub: user.id, purpose: 'email-verification' },
      { expiresIn: '24h' }
    );

    // Send verification email
    await this.mailService.sendVerificationEmail(
      user.email,
      verificationToken
    );

    return {
      message: 'Registration successful. Please check your email to verify your account.',
      user: {
        id: user.id,
        email: user.email,
        firstName: user.firstName,
        lastName: user.lastName,
      },
    };
  }

  async verifyEmail(token: string) {
    try {
      const payload = this.jwtService.verify(token);
      
      if (payload.purpose !== 'email-verification') {
        throw new BadRequestException('Invalid verification token');
      }

      await this.usersService.markEmailAsVerified(payload.sub);

      return { message: 'Email verified successfully' };
    } catch (error) {
      throw new BadRequestException('Invalid or expired verification token');
    }
  }
}
```

**Client-Side Request:**

```javascript
// Registration request
const response = await fetch('/auth/register', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    email: 'john@example.com',
    password: 'SecurePass123!',
    firstName: 'John',
    lastName: 'Doe',
  }),
});

const data = await response.json();
// { message, user, access_token }

// Store token
localStorage.setItem('access_token', data.access_token);
```

**Interview Tip**: Registration flow: 1) Validate input with DTO, 2) Check if user exists, 3) Hash password with bcrypt, 4) Save to database, 5) Return JWT or send verification email. Use ValidationPipe for automatic DTO validation. Hash password with 10-12 salt rounds. Never return password in response. Check for duplicate email before saving.

</details>

### 21. How do you hash passwords using bcrypt?

<details>
<summary>Answer</summary>

**bcrypt** is a password hashing function designed to be slow and resistant to brute force attacks.

**Installation:**

```bash
npm install bcrypt
npm install @types/bcrypt --save-dev
```

**Basic Password Hashing:**

```typescript
import * as bcrypt from 'bcrypt';

// Hash password
const password = 'mySecurePassword123';
const saltRounds = 10;  // Higher = more secure but slower (10-12 recommended)
const hashedPassword = await bcrypt.hash(password, saltRounds);

// Compare passwords
const isMatch = await bcrypt.compare(password, hashedPassword);
console.log(isMatch);  // true
```

**Registration with Hashing:**

```typescript
// users.service.ts
import { Injectable, ConflictException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import * as bcrypt from 'bcrypt';
import { User } from './entities/user.entity';

@Injectable()
export class UsersService {
  private readonly SALT_ROUNDS = 10;

  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>,
  ) {}

  async create(email: string, password: string, firstName: string, lastName: string): Promise<User> {
    // Check if user exists
    const existingUser = await this.usersRepository.findOne({
      where: { email },
    });

    if (existingUser) {
      throw new ConflictException('Email already exists');
    }

    // Hash password
    const hashedPassword = await bcrypt.hash(password, this.SALT_ROUNDS);

    // Create and save user
    const user = this.usersRepository.create({
      email,
      password: hashedPassword,
      firstName,
      lastName,
    });

    const savedUser = await this.usersRepository.save(user);

    // Remove password from returned object
    delete savedUser.password;
    return savedUser;
  }
}
```

**Login with Password Verification:**

```typescript
// auth.service.ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { UsersService } from '../users/users.service';
import * as bcrypt from 'bcrypt';

@Injectable()
export class AuthService {
  constructor(private usersService: UsersService) {}

  async validateUser(email: string, password: string): Promise<any> {
    // Find user
    const user = await this.usersService.findByEmail(email);

    if (!user) {
      return null;  // User not found
    }

    // Compare password with hashed password
    const isPasswordValid = await bcrypt.compare(password, user.password);

    if (!isPasswordValid) {
      return null;  // Invalid password
    }

    // Return user without password
    const { password: _, ...result } = user;
    return result;
  }
}
```

**Salt Rounds Comparison:**

```typescript
// Different salt rounds affect security and performance
const password = 'myPassword123';

// Salt rounds = 10 (Recommended for most applications)
const hash10 = await bcrypt.hash(password, 10);  // ~65ms

// Salt rounds = 12 (More secure, slower)
const hash12 = await bcrypt.hash(password, 12);  // ~260ms

// Salt rounds = 14 (Very secure, much slower)
const hash14 = await bcrypt.hash(password, 14);  // ~1000ms
```

| Salt Rounds | Time | Security | Use Case |
|-------------|------|----------|----------|
| 10 | ~65ms | Good | Most applications |
| 12 | ~260ms | Better | High security apps |
| 14 | ~1000ms | Best | Critical systems |

**Using Pre-Save Hook (Alternative):**

```typescript
// entities/user.entity.ts
import { Entity, Column, PrimaryGeneratedColumn, BeforeInsert, BeforeUpdate } from 'typeorm';
import * as bcrypt from 'bcrypt';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ unique: true })
  email: string;

  @Column()
  password: string;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  // Automatically hash password before saving
  @BeforeInsert()
  @BeforeUpdate()
  async hashPassword() {
    if (this.password && !this.password.startsWith('$2b$')) {
      // Only hash if password is not already hashed
      this.password = await bcrypt.hash(this.password, 10);
    }
  }

  // Method to validate password
  async validatePassword(password: string): Promise<boolean> {
    return bcrypt.compare(password, this.password);
  }
}
```

**Change Password:**

```typescript
// users.service.ts
async changePassword(userId: number, oldPassword: string, newPassword: string) {
  const user = await this.usersRepository.findOne({ where: { id: userId } });

  if (!user) {
    throw new NotFoundException('User not found');
  }

  // Verify old password
  const isOldPasswordValid = await bcrypt.compare(oldPassword, user.password);

  if (!isOldPasswordValid) {
    throw new UnauthorizedException('Current password is incorrect');
  }

  // Hash new password
  const hashedNewPassword = await bcrypt.hash(newPassword, 10);

  // Update password
  user.password = hashedNewPassword;
  user.passwordChangedAt = new Date();  // Track when password changed
  await this.usersRepository.save(user);

  return { message: 'Password changed successfully' };
}
```

**Security Best Practices:**

```typescript
// 1. Use environment variable for salt rounds
const saltRounds = parseInt(process.env.BCRYPT_SALT_ROUNDS || '10', 10);

// 2. Never log passwords
console.log('User:', user.email);  // ✅ Good
console.log('Password:', password);  // ❌ Never do this

// 3. Always remove password from responses
const { password, ...userWithoutPassword } = user;
return userWithoutPassword;

// 4. Use timing-safe comparison for additional security
const isValid = await bcrypt.compare(password, user.password);
// bcrypt.compare is timing-safe by default

// 5. Rate limit login attempts
@UseGuards(ThrottlerGuard)
@Post('login')
async login() { }
```

**Interview Tip**: Use bcrypt for password hashing. `bcrypt.hash(password, saltRounds)` to hash. `bcrypt.compare(plain, hashed)` to verify. Use 10-12 salt rounds. Higher rounds = more secure but slower. bcrypt automatically generates salt. Never store plain text passwords. Always remove password from API responses. bcrypt is intentionally slow to prevent brute force attacks.

</details>

### 22. How do you implement login endpoint and return JWT?

<details>
<summary>Answer</summary>

**Login flow**: Validate credentials → Generate JWT → Return token.

**Complete Login Implementation:**

```typescript
// dto/login.dto.ts
import { IsEmail, IsString, MinLength } from 'class-validator';

export class LoginDto {
  @IsEmail()
  email: string;

  @IsString()
  @MinLength(1)
  password: string;
}
```

```typescript
// auth.service.ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { UsersService } from '../users/users.service';
import * as bcrypt from 'bcrypt';
import { LoginDto } from './dto/login.dto';

@Injectable()
export class AuthService {
  constructor(
    private usersService: UsersService,
    private jwtService: JwtService,
  ) {}

  // Validate user credentials
  async validateUser(email: string, password: string): Promise<any> {
    const user = await this.usersService.findByEmail(email);

    if (!user) {
      return null;
    }

    // Compare passwords
    const isPasswordValid = await bcrypt.compare(password, user.password);

    if (!isPasswordValid) {
      return null;
    }

    // Return user without password
    const { password: _, ...result } = user;
    return result;
  }

  // Generate JWT token
  async login(user: any) {
    // Create JWT payload
    const payload = {
      sub: user.id,
      email: user.email,
      username: user.username,
      roles: user.roles,
    };

    // Sign and return token
    return {
      access_token: this.jwtService.sign(payload),
      token_type: 'Bearer',
      expires_in: 3600,  // 1 hour in seconds
      user: {
        id: user.id,
        email: user.email,
        firstName: user.firstName,
        lastName: user.lastName,
        roles: user.roles,
      },
    };
  }
}
```

```typescript
// strategies/local.strategy.ts
import { Strategy } from 'passport-local';
import { PassportStrategy } from '@nestjs/passport';
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { AuthService } from '../auth.service';

@Injectable()
export class LocalStrategy extends PassportStrategy(Strategy) {
  constructor(private authService: AuthService) {
    super({
      usernameField: 'email',
      passwordField: 'password',
    });
  }

  async validate(email: string, password: string): Promise<any> {
    const user = await this.authService.validateUser(email, password);

    if (!user) {
      throw new UnauthorizedException('Invalid email or password');
    }

    return user;
  }
}
```

```typescript
// guards/local-auth.guard.ts
import { Injectable } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class LocalAuthGuard extends AuthGuard('local') {}
```

```typescript
// auth.controller.ts
import { Controller, Post, Body, UseGuards, Request, HttpCode, HttpStatus } from '@nestjs/common';
import { Public } from './decorators/public.decorator';
import { AuthService } from './auth.service';
import { LocalAuthGuard } from './guards/local-auth.guard';
import { LoginDto } from './dto/login.dto';

@Controller('auth')
export class AuthController {
  constructor(private authService: AuthService) {}

  // Method 1: Using LocalAuthGuard (Recommended)
  @Public()
  @Post('login')
  @UseGuards(LocalAuthGuard)
  @HttpCode(HttpStatus.OK)
  async login(@Request() req) {
    // req.user comes from LocalStrategy.validate()
    return this.authService.login(req.user);
  }

  // Method 2: Manual validation (Alternative)
  @Public()
  @Post('login-manual')
  @HttpCode(HttpStatus.OK)
  async loginManual(@Body() loginDto: LoginDto) {
    const user = await this.authService.validateUser(
      loginDto.email,
      loginDto.password
    );

    if (!user) {
      throw new UnauthorizedException('Invalid credentials');
    }

    return this.authService.login(user);
  }
}
```

```typescript
// auth.module.ts
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { PassportModule } from '@nestjs/passport';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { AuthService } from './auth.service';
import { AuthController } from './auth.controller';
import { LocalStrategy } from './strategies/local.strategy';
import { JwtStrategy } from './strategies/jwt.strategy';
import { UsersModule } from '../users/users.module';

@Module({
  imports: [
    UsersModule,
    PassportModule,
    JwtModule.registerAsync({
      imports: [ConfigModule],
      useFactory: (configService: ConfigService) => ({
        secret: configService.get<string>('JWT_SECRET'),
        signOptions: {
          expiresIn: configService.get<string>('JWT_EXPIRES_IN', '1h'),
        },
      }),
      inject: [ConfigService],
    }),
  ],
  providers: [AuthService, LocalStrategy, JwtStrategy],
  controllers: [AuthController],
  exports: [AuthService],
})
export class AuthModule {}
```

**Client-Side Usage:**

```javascript
// Login request
const response = await fetch('/auth/login', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    email: 'john@example.com',
    password: 'SecurePass123!',
  }),
});

const data = await response.json();
/*
{
  access_token: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  token_type: "Bearer",
  expires_in: 3600,
  user: {
    id: 1,
    email: "john@example.com",
    firstName: "John",
    lastName: "Doe",
    roles: ["user"]
  }
}
*/

// Store token
localStorage.setItem('access_token', data.access_token);

// Use token in subsequent requests
const profileResponse = await fetch('/api/profile', {
  headers: {
    'Authorization': `Bearer ${data.access_token}`,
  },
});
```

**With Refresh Token:**

```typescript
async login(user: any) {
  const payload = { sub: user.id, email: user.email, roles: user.roles };

  // Generate access token
  const accessToken = this.jwtService.sign(payload, {
    secret: process.env.JWT_ACCESS_SECRET,
    expiresIn: '15m',
  });

  // Generate refresh token
  const refreshToken = this.jwtService.sign(
    { sub: user.id },
    {
      secret: process.env.JWT_REFRESH_SECRET,
      expiresIn: '7d',
    }
  );

  // Store refresh token in database
  await this.saveRefreshToken(user.id, refreshToken);

  return {
    access_token: accessToken,
    refresh_token: refreshToken,
    token_type: 'Bearer',
    expires_in: 900,  // 15 minutes
    user: {
      id: user.id,
      email: user.email,
      roles: user.roles,
    },
  };
}
```

**Interview Tip**: Login flow: 1) Use LocalStrategy to validate credentials, 2) Generate JWT with `jwtService.sign()`, 3) Return access_token and user info. Use `@UseGuards(LocalAuthGuard)` on login route. Payload should include: sub (user ID), email, roles. Return token_type "Bearer" and expires_in. Store token in localStorage or httpOnly cookie. Use token in Authorization header for protected requests.

</details>

## Role-Based Access Control (RBAC)

### 23. What is RBAC?

<details>
<summary>Answer</summary>

**RBAC (Role-Based Access Control)** restricts system access based on user roles.

**Key Concepts:**

- **Role**: Collection of permissions (admin, user, moderator)
- **Permission**: Specific action (read, write, delete)
- **User**: Assigned one or more roles

**Example Roles:**

| Role | Permissions | Access |
|------|-------------|--------|
| **admin** | All permissions | Full access |
| **moderator** | read, write, delete posts | Content management |
| **user** | read, write own posts | Basic access |
| **guest** | read only | View only |

**Simple Role Structure:**

```typescript
// entities/user.entity.ts
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  email: string;

  @Column()
  password: string;

  // Simple role array
  @Column('simple-array', { default: 'user' })
  roles: string[];  // ['user'], ['admin'], ['user', 'moderator']
}
```

**RBAC Flow:**

```typescript
// 1. User logs in
POST /auth/login
{
  "email": "admin@example.com",
  "password": "password123"
}

// 2. JWT contains roles
{
  "sub": 1,
  "email": "admin@example.com",
  "roles": ["admin"],
  "iat": 1516239022,
  "exp": 1516242622
}

// 3. Request to protected endpoint
GET /admin/users
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

// 4. Guards check roles
@UseGuards(JwtAuthGuard, RolesGuard)
@Roles('admin')
getUsers() { }

// 5. Access granted if user has 'admin' role
```

**Complex RBAC with Permissions:**

```typescript
// entities/role.entity.ts
@Entity()
export class Role {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ unique: true })
  name: string;  // 'admin', 'moderator', 'user'

  @Column('simple-array')
  permissions: string[];  // ['users:read', 'users:write', 'posts:delete']

  @ManyToMany(() => User, user => user.roles)
  users: User[];
}

// entities/user.entity.ts
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @ManyToMany(() => Role)
  @JoinTable()
  roles: Role[];
}
```

**RBAC vs ABAC:**

| Feature | RBAC | ABAC |
|---------|------|------|
| **Based on** | User roles | Attributes (user, resource, environment) |
| **Complexity** | Simple | Complex |
| **Use Case** | Standard apps | Enterprise systems |
| **Example** | User has 'admin' role | User is owner AND resource is draft |

**Interview Tip**: RBAC = Role-Based Access Control. Restricts access based on user roles. User has roles → Roles have permissions → Access granted/denied. Store roles in JWT payload. Use @Roles() decorator + RolesGuard. Common roles: admin, user, moderator. Simpler than ABAC for most applications.

</details>

### 24. How do you implement role-based authorization?

<details>
<summary>Answer</summary>

Implement RBAC with **@Roles() decorator** and **RolesGuard**.

**1. Store Roles in User Entity:**

```typescript
// entities/user.entity.ts
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  email: string;

  @Column()
  password: string;

  @Column('simple-array', { default: 'user' })
  roles: string[];  // ['user'], ['admin'], ['moderator', 'user']
}
```

**2. Include Roles in JWT:**

```typescript
// auth.service.ts
async login(user: any) {
  const payload = {
    sub: user.id,
    email: user.email,
    roles: user.roles,  // Include roles in JWT
  };

  return {
    access_token: this.jwtService.sign(payload),
  };
}
```

**3. Return Roles from JwtStrategy:**

```typescript
// strategies/jwt.strategy.ts
@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      secretOrKey: process.env.JWT_SECRET,
    });
  }

  async validate(payload: any) {
    return {
      userId: payload.sub,
      email: payload.email,
      roles: payload.roles,  // Attach roles to req.user
    };
  }
}
```

**4. Create @Roles() Decorator:**

```typescript
// decorators/roles.decorator.ts
import { SetMetadata } from '@nestjs/common';

export const ROLES_KEY = 'roles';
export const Roles = (...roles: string[]) => SetMetadata(ROLES_KEY, roles);
```

**5. Create RolesGuard:**

```typescript
// guards/roles.guard.ts
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { ROLES_KEY } from '../decorators/roles.decorator';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // Get required roles from @Roles() decorator
    const requiredRoles = this.reflector.getAllAndOverride<string[]>(
      ROLES_KEY,
      [
        context.getHandler(),  // Method level
        context.getClass(),    // Controller level
      ],
    );

    // If no roles required, allow access
    if (!requiredRoles) {
      return true;
    }

    // Get user from request (set by JwtStrategy)
    const { user } = context.switchToHttp().getRequest();

    // Check if user has any of the required roles
    return requiredRoles.some((role) => user.roles?.includes(role));
  }
}
```

**6. Apply Guards in Controller:**

```typescript
// users.controller.ts
import { Controller, Get, Post, Delete, UseGuards, Param } from '@nestjs/common';
import { JwtAuthGuard } from './guards/jwt-auth.guard';
import { RolesGuard } from './guards/roles.guard';
import { Roles } from './decorators/roles.decorator';

@Controller('users')
@UseGuards(JwtAuthGuard, RolesGuard)  // Apply both guards
export class UsersController {
  constructor(private usersService: UsersService) {}

  // Only admin can access
  @Get()
  @Roles('admin')
  findAll() {
    return this.usersService.findAll();
  }

  // Admin or moderator can access
  @Get('active')
  @Roles('admin', 'moderator')
  findActive() {
    return this.usersService.findActive();
  }

  // Only admin can delete
  @Delete(':id')
  @Roles('admin')
  remove(@Param('id') id: number) {
    return this.usersService.remove(id);
  }

  // Any authenticated user (no @Roles decorator)
  @Get('me')
  getProfile(@User() user) {
    return this.usersService.findById(user.userId);
  }
}
```

**7. Global Application:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { APP_GUARD } from '@nestjs/core';
import { JwtAuthGuard } from './auth/guards/jwt-auth.guard';
import { RolesGuard } from './auth/guards/roles.guard';

@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: JwtAuthGuard,  // Global JWT auth
    },
    {
      provide: APP_GUARD,
      useClass: RolesGuard,  // Global roles check
    },
  ],
})
export class AppModule {}
```

**Advanced RolesGuard with ForbiddenException:**

```typescript
// guards/roles.guard.ts
import { Injectable, CanActivate, ExecutionContext, ForbiddenException } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { ROLES_KEY } from '../decorators/roles.decorator';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<string[]>(
      ROLES_KEY,
      [context.getHandler(), context.getClass()],
    );

    if (!requiredRoles) {
      return true;
    }

    const { user } = context.switchToHttp().getRequest();

    const hasRole = requiredRoles.some((role) => user.roles?.includes(role));

    if (!hasRole) {
      throw new ForbiddenException(
        `User requires one of these roles: ${requiredRoles.join(', ')}`
      );
    }

    return true;
  }
}
```

**Multiple Roles Logic:**

```typescript
// Require ALL roles (AND logic)
@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<string[]>(
      ROLES_KEY,
      [context.getHandler(), context.getClass()],
    );

    if (!requiredRoles) {
      return true;
    }

    const { user } = context.switchToHttp().getRequest();

    // User must have ALL required roles
    return requiredRoles.every((role) => user.roles?.includes(role));
  }
}
```

**Interview Tip**: RBAC implementation: 1) Store roles in User entity, 2) Include roles in JWT, 3) Create @Roles() decorator with SetMetadata, 4) Create RolesGuard using Reflector, 5) Apply both JwtAuthGuard and RolesGuard. RolesGuard checks if user has required roles. Use `some()` for OR logic, `every()` for AND logic. Return 403 Forbidden if roles don't match.

</details>

### 25. How do you create a `@Roles()` decorator?

<details>
<summary>Answer</summary>

**@Roles() decorator** uses `SetMetadata()` to attach role requirements to routes.

**Basic @Roles() Decorator:**

```typescript
// decorators/roles.decorator.ts
import { SetMetadata } from '@nestjs/common';

// Metadata key constant
export const ROLES_KEY = 'roles';

// Decorator that accepts role names
export const Roles = (...roles: string[]) => SetMetadata(ROLES_KEY, roles);
```

**Usage:**

```typescript
import { Roles } from './decorators/roles.decorator';

@Controller('admin')
export class AdminController {
  // Single role
  @Get('users')
  @Roles('admin')
  getAllUsers() {
    return this.usersService.findAll();
  }

  // Multiple roles (user needs ONE of these)
  @Get('posts')
  @Roles('admin', 'moderator')
  getAllPosts() {
    return this.postsService.findAll();
  }

  // No @Roles = any authenticated user
  @Get('profile')
  getProfile() {
    return this.userService.getProfile();
  }
}
```

**How It Works:**

```typescript
// 1. @Roles('admin') is called
Roles('admin')

// 2. Returns SetMetadata
SetMetadata('roles', ['admin'])

// 3. Metadata attached to route handler
{
  'roles': ['admin']
}

// 4. RolesGuard reads metadata using Reflector
const requiredRoles = this.reflector.get('roles', context.getHandler());
// Returns: ['admin']
```

**Advanced @Roles() with TypeScript:**

```typescript
// Define role enum for type safety
export enum Role {
  Admin = 'admin',
  Moderator = 'moderator',
  User = 'user',
  Guest = 'guest',
}

// Type-safe Roles decorator
export const Roles = (...roles: Role[]) => SetMetadata(ROLES_KEY, roles);

// Usage with enum
import { Role } from './enums/role.enum';

@Get('users')
@Roles(Role.Admin, Role.Moderator)  // Type-safe
getAllUsers() {}
```

**Custom Decorator with Validation:**

```typescript
// decorators/roles.decorator.ts
import { SetMetadata } from '@nestjs/common';

export const ROLES_KEY = 'roles';

const VALID_ROLES = ['admin', 'moderator', 'user', 'guest'];

export const Roles = (...roles: string[]) => {
  // Validate roles at compile time
  const invalidRoles = roles.filter(role => !VALID_ROLES.includes(role));
  
  if (invalidRoles.length > 0) {
    throw new Error(`Invalid roles: ${invalidRoles.join(', ')}`);
  }

  return SetMetadata(ROLES_KEY, roles);
};
```

**Combining Multiple Decorators:**

```typescript
// decorators/auth.decorator.ts
import { applyDecorators, UseGuards } from '@nestjs/common';
import { JwtAuthGuard } from '../guards/jwt-auth.guard';
import { RolesGuard } from '../guards/roles.guard';
import { Roles } from './roles.decorator';

// Composite decorator
export function Auth(...roles: string[]) {
  return applyDecorators(
    UseGuards(JwtAuthGuard, RolesGuard),
    Roles(...roles),
  );
}

// Usage - Single decorator instead of multiple
@Get('users')
@Auth('admin')  // Combines guards + roles
getAllUsers() {}
```

**Reading Metadata in Guard:**

```typescript
// guards/roles.guard.ts
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { ROLES_KEY } from '../decorators/roles.decorator';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // Method 1: Get from handler only
    const rolesFromMethod = this.reflector.get<string[]>(
      ROLES_KEY,
      context.getHandler(),
    );

    // Method 2: Get from class only
    const rolesFromClass = this.reflector.get<string[]>(
      ROLES_KEY,
      context.getClass(),
    );

    // Method 3: Get from both (method overrides class)
    const roles = this.reflector.getAllAndOverride<string[]>(
      ROLES_KEY,
      [context.getHandler(), context.getClass()],
    );

    // Method 4: Merge both (combines method + class)
    const mergedRoles = this.reflector.getAllAndMerge<string[]>(
      ROLES_KEY,
      [context.getHandler(), context.getClass()],
    );

    if (!roles) {
      return true;
    }

    const { user } = context.switchToHttp().getRequest();
    return roles.some((role) => user.roles?.includes(role));
  }
}
```

**Controller and Method Level:**

```typescript
@Controller('posts')
@Roles('user')  // All routes require 'user' role
export class PostsController {
  @Get()
  findAll() {
    // Requires: 'user' (from controller)
  }

  @Post()
  @Roles('admin')  // Method overrides controller
  create() {
    // Requires: 'admin' (method overrides)
  }

  @Delete(':id')
  @Roles('admin', 'moderator')
  remove() {
    // Requires: 'admin' OR 'moderator'
  }
}
```

**Interview Tip**: Create @Roles() decorator using `SetMetadata()`. Takes role names as arguments. Attaches metadata to route handler. RolesGuard reads metadata with Reflector. Use constant for metadata key. Use enum for type-safe roles. Can combine with other decorators using `applyDecorators()`. Use `getAllAndOverride()` to check method then class level.

</details>

### 26. How do you create a Roles Guard using Reflector?

<details>
<summary>Answer</summary>

**RolesGuard** uses **Reflector** to read @Roles() metadata and check user permissions.

**Complete RolesGuard Implementation:**

```typescript
// guards/roles.guard.ts
import { Injectable, CanActivate, ExecutionContext, ForbiddenException } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { ROLES_KEY } from '../decorators/roles.decorator';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // 1. Get required roles from @Roles() decorator
    const requiredRoles = this.reflector.getAllAndOverride<string[]>(
      ROLES_KEY,
      [
        context.getHandler(),  // Check method level first
        context.getClass(),    // Then controller level
      ],
    );

    // 2. If no @Roles() decorator, allow access
    if (!requiredRoles || requiredRoles.length === 0) {
      return true;
    }

    // 3. Get user from request (attached by JwtStrategy)
    const { user } = context.switchToHttp().getRequest();

    // 4. Check if user exists
    if (!user) {
      return false;
    }

    // 5. Check if user has at least one required role
    const hasRole = requiredRoles.some((role) => user.roles?.includes(role));

    // 6. Throw exception if user doesn't have required role
    if (!hasRole) {
      throw new ForbiddenException(
        `Access denied. Required roles: ${requiredRoles.join(', ')}`
      );
    }

    return true;
  }
}
```

**Register in Module:**

```typescript
// auth.module.ts
import { Module } from '@nestjs/common';
import { RolesGuard } from './guards/roles.guard';

@Module({
  providers: [RolesGuard],
  exports: [RolesGuard],
})
export class AuthModule {}
```

**Usage in Controllers:**

```typescript
// users.controller.ts
import { Controller, Get, UseGuards } from '@nestjs/common';
import { JwtAuthGuard } from './guards/jwt-auth.guard';
import { RolesGuard } from './guards/roles.guard';
import { Roles } from './decorators/roles.decorator';

@Controller('users')
@UseGuards(JwtAuthGuard, RolesGuard)  // Both guards required
export class UsersController {
  @Get()
  @Roles('admin')  // RolesGuard checks this
  findAll() {
    return this.usersService.findAll();
  }

  @Get('active')
  @Roles('admin', 'moderator')  // User needs admin OR moderator
  findActive() {
    return this.usersService.findActive();
  }
}
```

**Advanced RolesGuard - Require ALL Roles:**

```typescript
// guards/roles-all.guard.ts
@Injectable()
export class RolesAllGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<string[]>(
      ROLES_KEY,
      [context.getHandler(), context.getClass()],
    );

    if (!requiredRoles) {
      return true;
    }

    const { user } = context.switchToHttp().getRequest();

    // User must have ALL required roles (AND logic)
    const hasAllRoles = requiredRoles.every((role) => 
      user.roles?.includes(role)
    );

    if (!hasAllRoles) {
      throw new ForbiddenException(
        `Access denied. User must have ALL roles: ${requiredRoles.join(', ')}`
      );
    }

    return true;
  }
}
```

**RolesGuard with Logging:**

```typescript
@Injectable()
export class RolesGuard implements CanActivate {
  private readonly logger = new Logger(RolesGuard.name);

  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<string[]>(
      ROLES_KEY,
      [context.getHandler(), context.getClass()],
    );

    if (!requiredRoles) {
      return true;
    }

    const { user } = context.switchToHttp().getRequest();

    this.logger.log(`Checking roles for user: ${user.email}`);
    this.logger.log(`Required roles: ${requiredRoles.join(', ')}`);
    this.logger.log(`User roles: ${user.roles?.join(', ')}`);

    const hasRole = requiredRoles.some((role) => user.roles?.includes(role));

    if (!hasRole) {
      this.logger.warn(`Access denied for user: ${user.email}`);
      throw new ForbiddenException('Insufficient permissions');
    }

    this.logger.log(`Access granted for user: ${user.email}`);
    return true;
  }
}
```

**Reflector Methods:**

```typescript
// 1. get() - Get from single target
const roles = this.reflector.get<string[]>(ROLES_KEY, context.getHandler());

// 2. getAll() - Get from multiple targets (returns array of arrays)
const rolesArray = this.reflector.getAll<string[]>(ROLES_KEY, [
  context.getHandler(),
  context.getClass(),
]);
// Returns: [['admin'], ['user']]

// 3. getAllAndOverride() - Method overrides class
const roles = this.reflector.getAllAndOverride<string[]>(ROLES_KEY, [
  context.getHandler(),  // If exists, use this
  context.getClass(),    // Otherwise, use this
]);

// 4. getAllAndMerge() - Combines method and class
const roles = this.reflector.getAllAndMerge<string[]>(ROLES_KEY, [
  context.getHandler(),  // ['admin']
  context.getClass(),    // ['user']
]);
// Returns: ['admin', 'user']
```

**Global RolesGuard:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { APP_GUARD } from '@nestjs/core';
import { RolesGuard } from './auth/guards/roles.guard';

@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: RolesGuard,  // Applied globally
    },
  ],
})
export class AppModule {}
```

**Testing RolesGuard:**

```typescript
// roles.guard.spec.ts
describe('RolesGuard', () => {
  let guard: RolesGuard;
  let reflector: Reflector;

  beforeEach(() => {
    reflector = new Reflector();
    guard = new RolesGuard(reflector);
  });

  it('should allow access when no roles required', () => {
    const context = createMockExecutionContext();
    jest.spyOn(reflector, 'getAllAndOverride').mockReturnValue(undefined);

    expect(guard.canActivate(context)).toBe(true);
  });

  it('should allow access when user has required role', () => {
    const context = createMockExecutionContext({
      user: { roles: ['admin'] },
    });
    jest.spyOn(reflector, 'getAllAndOverride').mockReturnValue(['admin']);

    expect(guard.canActivate(context)).toBe(true);
  });

  it('should deny access when user lacks required role', () => {
    const context = createMockExecutionContext({
      user: { roles: ['user'] },
    });
    jest.spyOn(reflector, 'getAllAndOverride').mockReturnValue(['admin']);

    expect(() => guard.canActivate(context)).toThrow(ForbiddenException);
  });
});
```

**Interview Tip**: RolesGuard implements `CanActivate`. Inject Reflector in constructor. Use `getAllAndOverride()` to read @Roles() metadata. Check method level first, then controller level. Compare required roles with user.roles. Return true to allow, false or throw exception to deny. Use `some()` for OR logic (any role), `every()` for AND logic (all roles). Guard runs after authentication guard.

</details>

## Refresh Tokens

### 27. What are refresh tokens and why are they needed?

<details>
<summary>Answer</summary>

**Refresh tokens** allow obtaining new access tokens without re-authentication. Improves security and user experience.

**Why Needed:**

| Problem | Solution |
|---------|----------|
| Access tokens expire quickly (15min-1h) | Refresh token gets new access token |
| Re-login every hour is bad UX | Seamless token refresh |
| Long-lived access tokens are risky | Short access + long refresh tokens |
| Can't revoke stateless JWT | Store refresh tokens in DB for revocation |

**Access vs Refresh Tokens:**

| Feature | Access Token | Refresh Token |
|---------|-------------|---------------|
| **Lifetime** | Short (15min-1h) | Long (7-30 days) |
| **Purpose** | API authentication | Get new access token |
| **Storage** | Memory | httpOnly cookie / DB |
| **Sent with** | Every request | Only refresh endpoint |
| **Revocable** | No (stateless) | Yes (stored in DB) |

**Flow:**

```typescript
// 1. User logs in
POST /auth/login
{
  "email": "user@example.com",
  "password": "password123"
}

// 2. Server returns both tokens
{
  "access_token": "eyJhbGc...",  // Short-lived (15 min)
  "refresh_token": "eyJzdWI...",  // Long-lived (7 days)
  "expires_in": 900
}

// 3. Client uses access token for API requests
GET /api/profile
Authorization: Bearer eyJhbGc...

// 4. Access token expires after 15 minutes
// Response: 401 Unauthorized

// 5. Client uses refresh token to get new access token
POST /auth/refresh
{
  "refresh_token": "eyJzdWI..."
}

// 6. Server returns new access token
{
  "access_token": "eyJnbGc...",  // New token
  "expires_in": 900
}

// 7. Client continues with new access token
```

**Interview Tip**: Refresh tokens solve access token expiration. Access tokens expire fast (15min-1h) for security. Refresh tokens last longer (7-30d) and get new access tokens. Prevents re-login. Refresh tokens stored in DB for revocation. Access tokens are stateless, refresh tokens are stateful.

</details>

### 28. How do you implement refresh token flow?

<details>
<summary>Answer</summary>

**Complete Refresh Token Implementation:**

```typescript
// entities/refresh-token.entity.ts
import { Entity, Column, PrimaryGeneratedColumn, CreateDateColumn } from 'typeorm';

@Entity('refresh_tokens')
export class RefreshToken {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  userId: number;

  @Column()
  token: string;  // Hashed refresh token

  @Column()
  expiresAt: Date;

  @Column({ default: false })
  isRevoked: boolean;

  @CreateDateColumn()
  createdAt: Date;
}
```

```typescript
// auth.service.ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import * as bcrypt from 'bcrypt';
import { RefreshToken } from './entities/refresh-token.entity';
import { UsersService } from '../users/users.service';

@Injectable()
export class AuthService {
  constructor(
    private jwtService: JwtService,
    private usersService: UsersService,
    @InjectRepository(RefreshToken)
    private refreshTokenRepository: Repository<RefreshToken>,
  ) {}

  async login(user: any) {
    const tokens = await this.generateTokens(user);
    
    // Store refresh token in database
    await this.saveRefreshToken(user.id, tokens.refresh_token);
    
    return tokens;
  }

  async generateTokens(user: any) {
    const payload = {
      sub: user.id,
      email: user.email,
      roles: user.roles,
    };

    // Access token - short lived
    const accessToken = this.jwtService.sign(payload, {
      secret: process.env.JWT_ACCESS_SECRET,
      expiresIn: '15m',
    });

    // Refresh token - long lived, minimal payload
    const refreshToken = this.jwtService.sign(
      { sub: user.id },
      {
        secret: process.env.JWT_REFRESH_SECRET,
        expiresIn: '7d',
      }
    );

    return {
      access_token: accessToken,
      refresh_token: refreshToken,
      token_type: 'Bearer',
      expires_in: 900,  // 15 minutes
    };
  }

  async saveRefreshToken(userId: number, token: string) {
    // Hash refresh token before storing
    const hashedToken = await bcrypt.hash(token, 10);
    
    // Delete old refresh tokens for this user
    await this.refreshTokenRepository.delete({ userId });
    
    // Save new refresh token
    const refreshToken = this.refreshTokenRepository.create({
      userId,
      token: hashedToken,
      expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000), // 7 days
    });

    await this.refreshTokenRepository.save(refreshToken);
  }

  async refreshAccessToken(refreshToken: string) {
    try {
      // 1. Verify refresh token
      const payload = this.jwtService.verify(refreshToken, {
        secret: process.env.JWT_REFRESH_SECRET,
      });

      // 2. Check if user exists
      const user = await this.usersService.findById(payload.sub);
      if (!user) {
        throw new UnauthorizedException('User not found');
      }

      // 3. Verify refresh token exists in database
      const storedTokens = await this.refreshTokenRepository.find({
        where: { userId: user.id, isRevoked: false },
      });

      const isValid = await Promise.any(
        storedTokens.map(async (stored) => {
          return await bcrypt.compare(refreshToken, stored.token);
        })
      );

      if (!isValid) {
        throw new UnauthorizedException('Invalid refresh token');
      }

      // 4. Generate new access token
      const newAccessToken = this.jwtService.sign({
        sub: user.id,
        email: user.email,
        roles: user.roles,
      }, {
        secret: process.env.JWT_ACCESS_SECRET,
        expiresIn: '15m',
      });

      return {
        access_token: newAccessToken,
        token_type: 'Bearer',
        expires_in: 900,
      };
    } catch (error) {
      throw new UnauthorizedException('Invalid or expired refresh token');
    }
  }

  async logout(userId: number) {
    // Revoke all refresh tokens for user
    await this.refreshTokenRepository.update(
      { userId },
      { isRevoked: true }
    );
  }
}
```

```typescript
// auth.controller.ts
import { Controller, Post, Body, UseGuards, Request } from '@nestjs/common';
import { Public } from './decorators/public.decorator';
import { JwtAuthGuard } from './guards/jwt-auth.guard';
import { AuthService } from './auth.service';

@Controller('auth')
export class AuthController {
  constructor(private authService: AuthService) {}

  @Public()
  @Post('login')
  async login(@Body() loginDto: LoginDto) {
    const user = await this.authService.validateUser(
      loginDto.email,
      loginDto.password
    );

    if (!user) {
      throw new UnauthorizedException('Invalid credentials');
    }

    return this.authService.login(user);
  }

  @Public()
  @Post('refresh')
  async refresh(@Body('refresh_token') refreshToken: string) {
    if (!refreshToken) {
      throw new UnauthorizedException('Refresh token required');
    }

    return this.authService.refreshAccessToken(refreshToken);
  }

  @Post('logout')
  @UseGuards(JwtAuthGuard)
  async logout(@Request() req) {
    await this.authService.logout(req.user.userId);
    return { message: 'Logged out successfully' };
  }
}
```

**Client-Side Implementation:**

```javascript
// Login and store tokens
const login = async (email, password) => {
  const response = await fetch('/auth/login', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, password }),
  });

  const data = await response.json();
  localStorage.setItem('access_token', data.access_token);
  localStorage.setItem('refresh_token', data.refresh_token);
};

// API request with auto-refresh
const apiRequest = async (url) => {
  let token = localStorage.getItem('access_token');
  
  let response = await fetch(url, {
    headers: { 'Authorization': `Bearer ${token}` },
  });

  // If 401, try refreshing token
  if (response.status === 401) {
    const refreshToken = localStorage.getItem('refresh_token');
    
    const refreshResponse = await fetch('/auth/refresh', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ refresh_token: refreshToken }),
    });

    if (refreshResponse.ok) {
      const data = await refreshResponse.json();
      localStorage.setItem('access_token', data.access_token);
      
      // Retry original request with new token
      response = await fetch(url, {
        headers: { 'Authorization': `Bearer ${data.access_token}` },
      });
    } else {
      // Refresh failed, redirect to login
      window.location.href = '/login';
    }
  }

  return response.json();
};
```

**Interview Tip**: Refresh token flow: 1) Login returns both tokens, 2) Store refresh token in DB (hashed), 3) Use access token for requests, 4) When access expires, use refresh token to get new access token, 5) Verify refresh token in DB. Access token: short (15min), Refresh token: long (7d). Logout revokes refresh tokens. Hash refresh tokens in DB.

</details>

### 29. Where should you store refresh tokens?

<details>
<summary>Answer</summary>

**Best practice**: Store refresh tokens in **httpOnly cookies** (client) and **database** (server).

**Storage Options Comparison:**

| Storage | Security | XSS Risk | CSRF Risk | Access | Use Case |
|---------|----------|----------|-----------|--------|----------|
| **httpOnly Cookie** | ✅ High | ✅ Protected | ⚠️ Vulnerable | Auto-sent | **Recommended for web apps** |
| **localStorage** | ❌ Low | ❌ Vulnerable | ✅ Protected | Manual | ❌ Not recommended |
| **sessionStorage** | ⚠️ Medium | ❌ Vulnerable | ✅ Protected | Manual | Tab-specific only |
| **Memory** | ✅ High | ✅ Protected | ✅ Protected | Lost on refresh | SPAs with refresh |
| **Database** | ✅ High | N/A | N/A | Server-side | **Required for revocation** |

**Recommended Approach: httpOnly Cookie + Database:**

```typescript
// auth.controller.ts
import { Controller, Post, Body, Res, Req, UseGuards } from '@nestjs/common';
import { Response, Request } from 'express';
import { Public } from './decorators/public.decorator';

@Controller('auth')
export class AuthController {
  constructor(private authService: AuthService) {}

  @Public()
  @Post('login')
  async login(
    @Body() loginDto: LoginDto,
    @Res({ passthrough: true }) response: Response,
  ) {
    const user = await this.authService.validateUser(
      loginDto.email,
      loginDto.password
    );

    if (!user) {
      throw new UnauthorizedException('Invalid credentials');
    }

    const tokens = await this.authService.login(user);

    // Store refresh token in httpOnly cookie
    response.cookie('refresh_token', tokens.refresh_token, {
      httpOnly: true,  // Not accessible via JavaScript
      secure: process.env.NODE_ENV === 'production',  // HTTPS only in production
      sameSite: 'strict',  // CSRF protection
      maxAge: 7 * 24 * 60 * 60 * 1000,  // 7 days
    });

    // Return only access token
    return {
      access_token: tokens.access_token,
      token_type: 'Bearer',
      expires_in: 900,
    };
  }

  @Public()
  @Post('refresh')
  async refresh(
    @Req() request: Request,
    @Res({ passthrough: true }) response: Response,
  ) {
    // Get refresh token from cookie
    const refreshToken = request.cookies['refresh_token'];

    if (!refreshToken) {
      throw new UnauthorizedException('Refresh token not found');
    }

    const tokens = await this.authService.refreshAccessToken(refreshToken);

    // Update refresh token cookie
    response.cookie('refresh_token', tokens.refresh_token, {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'strict',
      maxAge: 7 * 24 * 60 * 60 * 1000,
    });

    return {
      access_token: tokens.access_token,
      token_type: 'Bearer',
      expires_in: 900,
    };
  }

  @Post('logout')
  @UseGuards(JwtAuthGuard)
  async logout(
    @Req() request: Request,
    @Res({ passthrough: true }) response: Response,
  ) {
    await this.authService.logout(request.user.userId);
    
    // Clear cookie
    response.clearCookie('refresh_token');
    
    return { message: 'Logged out successfully' };
  }
}
```

**Enable Cookie Parser:**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import * as cookieParser from 'cookie-parser';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Enable cookie parser
  app.use(cookieParser());
  
  await app.listen(3000);
}
bootstrap();
```

**Install Dependencies:**

```bash
npm install cookie-parser
npm install @types/cookie-parser --save-dev
```

**Database Storage (Required for Revocation):**

```typescript
// Always store refresh tokens in database
interface RefreshTokenEntity {
  id: number;
  userId: number;
  token: string;  // Hashed
  expiresAt: Date;
  isRevoked: boolean;
  createdAt: Date;
}

// Why database storage:
// 1. Can revoke tokens on logout
// 2. Can revoke all sessions
// 3. Can track active sessions
// 4. Can implement "logout from all devices"
```

**Interview Tip**: Best practice: httpOnly cookies (client) + database (server). httpOnly protects from XSS. SameSite protects from CSRF. Database enables revocation. Never use localStorage for refresh tokens (XSS vulnerable). Use secure:true in production (HTTPS only). Hash tokens in database. Clear cookie on logout.

</details>

## OAuth 2.0

### 30. What is OAuth 2.0?

<details>
<summary>Answer</summary>

**OAuth 2.0** is an authorization framework that enables third-party applications to access user data without sharing passwords.

**Key Concepts:**

- **Resource Owner**: User
- **Client**: Your application
- **Authorization Server**: Google, Facebook, GitHub
- **Resource Server**: API with user data

**OAuth Flow:**

```
1. User clicks "Login with Google"
2. Redirect to Google login page
3. User grants permissions
4. Google redirects back with authorization code
5. Exchange code for access token
6. Use access token to get user profile
7. Create/login user in your system
```

**OAuth 2.0 vs Traditional Login:**

| Feature | Traditional | OAuth 2.0 |
|---------|-------------|----------|
| **Password** | User creates password | No password needed |
| **Security** | You manage passwords | Provider handles security |
| **User Experience** | Create account | One-click login |
| **Trust** | User trusts you | User trusts Google/Facebook |
| **Maintenance** | Password reset, etc. | Provider handles it |

**Common OAuth Providers:**

- Google
- Facebook  
- GitHub
- Twitter/X
- Microsoft
- Apple

**Interview Tip**: OAuth 2.0 = Authorization framework for third-party login. User grants permission → Provider returns access token → Get user profile → Create/login user. Benefits: No password management, better UX, leverages trusted providers. Use passport-google-oauth20 for Google OAuth in NestJS.

</details>

### 31. How do you implement Google OAuth Strategy?

<details>
<summary>Answer</summary>

**Complete Google OAuth Implementation:**

**1. Install Dependencies:**

```bash
npm install passport-google-oauth20
npm install @types/passport-google-oauth20 --save-dev
```

**2. Get Google OAuth Credentials:**

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create project
3. Enable Google+ API
4. Create OAuth 2.0 credentials
5. Add authorized redirect URI: `http://localhost:3000/auth/google/callback`

**3. Environment Variables:**

```env
GOOGLE_CLIENT_ID=your-client-id.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=your-client-secret
GOOGLE_CALLBACK_URL=http://localhost:3000/auth/google/callback
```

**4. Create Google Strategy:**

```typescript
// strategies/google.strategy.ts
import { PassportStrategy } from '@nestjs/passport';
import { Strategy, VerifyCallback } from 'passport-google-oauth20';
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { AuthService } from '../auth.service';

@Injectable()
export class GoogleStrategy extends PassportStrategy(Strategy, 'google') {
  constructor(
    private configService: ConfigService,
    private authService: AuthService,
  ) {
    super({
      clientID: configService.get<string>('GOOGLE_CLIENT_ID'),
      clientSecret: configService.get<string>('GOOGLE_CLIENT_SECRET'),
      callbackURL: configService.get<string>('GOOGLE_CALLBACK_URL'),
      scope: ['email', 'profile'],
    });
  }

  async validate(
    accessToken: string,
    refreshToken: string,
    profile: any,
    done: VerifyCallback,
  ): Promise<any> {
    const { name, emails, photos } = profile;

    const user = {
      email: emails[0].value,
      firstName: name.givenName,
      lastName: name.familyName,
      picture: photos[0].value,
      accessToken,
    };

    done(null, user);
  }
}
```

**5. Update Auth Service:**

```typescript
// auth.service.ts
@Injectable()
export class AuthService {
  constructor(
    private usersService: UsersService,
    private jwtService: JwtService,
  ) {}

  async googleLogin(googleUser: any) {
    // Check if user exists
    let user = await this.usersService.findByEmail(googleUser.email);

    // Create user if doesn't exist
    if (!user) {
      user = await this.usersService.create({
        email: googleUser.email,
        firstName: googleUser.firstName,
        lastName: googleUser.lastName,
        picture: googleUser.picture,
        provider: 'google',
        isEmailVerified: true,  // Email verified by Google
      });
    }

    // Generate JWT
    const payload = {
      sub: user.id,
      email: user.email,
      roles: user.roles,
    };

    return {
      access_token: this.jwtService.sign(payload),
      user: {
        id: user.id,
        email: user.email,
        firstName: user.firstName,
        lastName: user.lastName,
      },
    };
  }
}
```

**6. Create Auth Guard:**

```typescript
// guards/google-auth.guard.ts
import { Injectable } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class GoogleAuthGuard extends AuthGuard('google') {}
```

**7. Controller:**

```typescript
// auth.controller.ts
import { Controller, Get, UseGuards, Req, Res } from '@nestjs/common';
import { Public } from './decorators/public.decorator';
import { GoogleAuthGuard } from './guards/google-auth.guard';
import { AuthService } from './auth.service';
import { Response } from 'express';

@Controller('auth')
export class AuthController {
  constructor(private authService: AuthService) {}

  @Public()
  @Get('google')
  @UseGuards(GoogleAuthGuard)
  async googleAuth() {
    // Redirects to Google
  }

  @Public()
  @Get('google/callback')
  @UseGuards(GoogleAuthGuard)
  async googleAuthRedirect(
    @Req() req,
    @Res() res: Response,
  ) {
    // Handle Google callback
    const result = await this.authService.googleLogin(req.user);

    // Redirect to frontend with token
    res.redirect(`http://localhost:3001/auth/callback?token=${result.access_token}`);
  }
}
```

**8. Register Strategy:**

```typescript
// auth.module.ts
import { Module } from '@nestjs/common';
import { GoogleStrategy } from './strategies/google.strategy';
import { AuthService } from './auth.service';
import { AuthController } from './auth.controller';

@Module({
  imports: [UsersModule, JwtModule, ConfigModule],
  providers: [AuthService, GoogleStrategy],
  controllers: [AuthController],
})
export class AuthModule {}
```

**9. Frontend Integration:**

```html
<!-- Login button -->
<a href="http://localhost:3000/auth/google">
  <button>Login with Google</button>
</a>
```

```javascript
// Handle callback
const params = new URLSearchParams(window.location.search);
const token = params.get('token');

if (token) {
  localStorage.setItem('access_token', token);
  // Redirect to dashboard
  window.location.href = '/dashboard';
}
```

**Interview Tip**: Google OAuth: Install passport-google-oauth20. Get credentials from Google Cloud Console. Create GoogleStrategy with clientID, clientSecret, callbackURL. Set scope: ['email', 'profile']. Create/login user in validate(). Generate JWT. Redirect to frontend with token. No password needed.

</details>

## Password Management

### 32. How do you implement password reset functionality?

<details>
<summary>Answer</summary>

**Password reset flow**: Request reset → Send email → Verify token → Reset password.

**Complete Implementation:**

```typescript
// dto/forgot-password.dto.ts
import { IsEmail } from 'class-validator';

export class ForgotPasswordDto {
  @IsEmail()
  email: string;
}

export class ResetPasswordDto {
  @IsString()
  token: string;

  @IsString()
  @MinLength(8)
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
  newPassword: string;
}
```

```typescript
// auth.service.ts
import { Injectable, BadRequestException, NotFoundException } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { UsersService } from '../users/users.service';
import { MailService } from '../mail/mail.service';
import * as bcrypt from 'bcrypt';

@Injectable()
export class AuthService {
  constructor(
    private usersService: UsersService,
    private jwtService: JwtService,
    private mailService: MailService,
  ) {}

  async forgotPassword(email: string) {
    // Find user
    const user = await this.usersService.findByEmail(email);

    if (!user) {
      // Don't reveal if user exists (security)
      return {
        message: 'If email exists, password reset link has been sent',
      };
    }

    // Generate reset token (short-lived)
    const resetToken = this.jwtService.sign(
      {
        sub: user.id,
        purpose: 'password-reset',
      },
      {
        secret: process.env.JWT_RESET_SECRET,
        expiresIn: '1h',  // Token valid for 1 hour
      }
    );

    // Send email
    const resetUrl = `${process.env.FRONTEND_URL}/reset-password?token=${resetToken}`;
    await this.mailService.sendPasswordResetEmail(user.email, resetUrl);

    return {
      message: 'If email exists, password reset link has been sent',
    };
  }

  async resetPassword(token: string, newPassword: string) {
    try {
      // Verify token
      const payload = this.jwtService.verify(token, {
        secret: process.env.JWT_RESET_SECRET,
      });

      // Check purpose
      if (payload.purpose !== 'password-reset') {
        throw new BadRequestException('Invalid reset token');
      }

      // Get user
      const user = await this.usersService.findById(payload.sub);

      if (!user) {
        throw new NotFoundException('User not found');
      }

      // Hash new password
      const hashedPassword = await bcrypt.hash(newPassword, 10);

      // Update password
      user.password = hashedPassword;
      user.passwordChangedAt = new Date();
      await this.usersService.save(user);

      // Revoke all refresh tokens
      await this.revokeAllRefreshTokens(user.id);

      return {
        message: 'Password reset successfully',
      };
    } catch (error) {
      if (error.name === 'TokenExpiredError') {
        throw new BadRequestException('Reset token has expired');
      }
      throw new BadRequestException('Invalid or expired reset token');
    }
  }

  private async revokeAllRefreshTokens(userId: number) {
    // Implementation depends on your refresh token storage
  }
}
```

```typescript
// auth.controller.ts
import { Controller, Post, Body } from '@nestjs/common';
import { Public } from './decorators/public.decorator';
import { AuthService } from './auth.service';
import { ForgotPasswordDto, ResetPasswordDto } from './dto/password.dto';

@Controller('auth')
export class AuthController {
  constructor(private authService: AuthService) {}

  @Public()
  @Post('forgot-password')
  async forgotPassword(@Body() forgotPasswordDto: ForgotPasswordDto) {
    return this.authService.forgotPassword(forgotPasswordDto.email);
  }

  @Public()
  @Post('reset-password')
  async resetPassword(@Body() resetPasswordDto: ResetPasswordDto) {
    return this.authService.resetPassword(
      resetPasswordDto.token,
      resetPasswordDto.newPassword,
    );
  }
}
```

```typescript
// mail.service.ts
import { Injectable } from '@nestjs/common';
import * as nodemailer from 'nodemailer';

@Injectable()
export class MailService {
  private transporter;

  constructor() {
    this.transporter = nodemailer.createTransporter({
      host: process.env.MAIL_HOST,
      port: process.env.MAIL_PORT,
      auth: {
        user: process.env.MAIL_USER,
        pass: process.env.MAIL_PASSWORD,
      },
    });
  }

  async sendPasswordResetEmail(email: string, resetUrl: string) {
    await this.transporter.sendMail({
      from: process.env.MAIL_FROM,
      to: email,
      subject: 'Password Reset Request',
      html: `
        <p>You requested a password reset</p>
        <p>Click this link to reset your password:</p>
        <a href="${resetUrl}">${resetUrl}</a>
        <p>Link expires in 1 hour</p>
        <p>If you didn't request this, please ignore this email</p>
      `,
    });
  }
}
```

**Frontend Example:**

```javascript
// Request password reset
const forgotPassword = async (email) => {
  await fetch('/auth/forgot-password', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email }),
  });
  // Show message to check email
};

// Reset password with token from email
const resetPassword = async (token, newPassword) => {
  const response = await fetch('/auth/reset-password', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ token, newPassword }),
  });

  if (response.ok) {
    // Redirect to login
    window.location.href = '/login';
  }
};
```

**Interview Tip**: Password reset: 1) User requests reset with email, 2) Generate short-lived JWT (1h), 3) Send email with reset link, 4) Verify token, 5) Hash and update password, 6) Revoke refresh tokens. Don't reveal if email exists. Use separate secret for reset tokens. Track passwordChangedAt to invalidate old tokens.

</details>

### 33. How do you implement change password?

<details>
<summary>Answer</summary>

**Change password**: Verify current password → Update to new password.

```typescript
// dto/change-password.dto.ts
import { IsString, MinLength, Matches } from 'class-validator';

export class ChangePasswordDto {
  @IsString()
  currentPassword: string;

  @IsString()
  @MinLength(8)
  @Matches(
    /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])/,
    { message: 'Password too weak' }
  )
  newPassword: string;
}
```

```typescript
// users.service.ts
import { Injectable, UnauthorizedException, BadRequestException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import * as bcrypt from 'bcrypt';
import { User } from './entities/user.entity';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>,
  ) {}

  async changePassword(
    userId: number,
    currentPassword: string,
    newPassword: string,
  ) {
    // Get user with password
    const user = await this.usersRepository.findOne({
      where: { id: userId },
      select: ['id', 'email', 'password'],
    });

    if (!user) {
      throw new NotFoundException('User not found');
    }

    // Verify current password
    const isCurrentPasswordValid = await bcrypt.compare(
      currentPassword,
      user.password,
    );

    if (!isCurrentPasswordValid) {
      throw new UnauthorizedException('Current password is incorrect');
    }

    // Check if new password is different
    const isSamePassword = await bcrypt.compare(newPassword, user.password);
    if (isSamePassword) {
      throw new BadRequestException(
        'New password must be different from current password',
      );
    }

    // Hash new password
    const hashedPassword = await bcrypt.hash(newPassword, 10);

    // Update password and timestamp
    user.password = hashedPassword;
    user.passwordChangedAt = new Date();
    await this.usersRepository.save(user);

    return {
      message: 'Password changed successfully',
    };
  }
}
```

```typescript
// users.controller.ts
import { Controller, Put, Body, UseGuards, Request } from '@nestjs/common';
import { JwtAuthGuard } from '../auth/guards/jwt-auth.guard';
import { UsersService } from './users.service';
import { ChangePasswordDto } from './dto/change-password.dto';

@Controller('users')
@UseGuards(JwtAuthGuard)
export class UsersController {
  constructor(private usersService: UsersService) {}

  @Put('change-password')
  async changePassword(
    @Request() req,
    @Body() changePasswordDto: ChangePasswordDto,
  ) {
    return this.usersService.changePassword(
      req.user.userId,
      changePasswordDto.currentPassword,
      changePasswordDto.newPassword,
    );
  }
}
```

**With Refresh Token Revocation:**

```typescript
async changePassword(
  userId: number,
  currentPassword: string,
  newPassword: string,
) {
  // ... validation and password update ...

  // Revoke all refresh tokens (logout from all devices)
  await this.refreshTokenRepository.update(
    { userId },
    { isRevoked: true }
  );

  return {
    message: 'Password changed successfully. Please login again on all devices.',
  };
}
```

**Frontend Example:**

```javascript
const changePassword = async (currentPassword, newPassword) => {
  const token = localStorage.getItem('access_token');

  const response = await fetch('/users/change-password', {
    method: 'PUT',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${token}`,
    },
    body: JSON.stringify({
      currentPassword,
      newPassword,
    }),
  });

  if (response.ok) {
    alert('Password changed successfully');
  } else {
    const error = await response.json();
    alert(error.message);
  }
};
```

**Interview Tip**: Change password: 1) Verify current password with bcrypt.compare(), 2) Check new password is different, 3) Hash new password, 4) Update password and passwordChangedAt, 5) Optionally revoke refresh tokens. Requires authentication. Return 401 if current password wrong. Use strong password validation.

</details>

## Security Best Practices

### 34. Should you store JWT in cookies or headers?

<details>
<summary>Answer</summary>

**Depends on use case**. For SPAs: **headers** (localStorage). For traditional web: **httpOnly cookies**.

**Comparison:**

| Storage | XSS Protection | CSRF Protection | Auto-sent | Best For |
|---------|----------------|-----------------|-----------|----------|
| **httpOnly Cookie** | ✅ Yes | ❌ No (need CSRF token) | Yes | Server-rendered apps |
| **localStorage** | ❌ No | ✅ Yes | No | SPAs, Mobile apps |
| **Memory** | ✅ Yes | ✅ Yes | No | SPAs with auto-refresh |

**httpOnly Cookie Approach:**

```typescript
// Login - set httpOnly cookie
@Post('login')
async login(
  @Body() loginDto: LoginDto,
  @Res({ passthrough: true }) response: Response,
) {
  const tokens = await this.authService.login(loginDto);
  response.cookie('access_token', tokens.access_token, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict',
    maxAge: 15 * 60 * 1000,
  });
  return { message: 'Login successful' };
}
```

**localStorage Approach:**

```typescript
// Client stores in localStorage
localStorage.setItem('access_token', data.access_token);

// Client sends in Authorization header
fetch('/api/profile', {
  headers: { 'Authorization': `Bearer ${localStorage.getItem('access_token')}` },
});
```

**Interview Tip**: httpOnly cookies protect from XSS but need CSRF protection. localStorage vulnerable to XSS but immune to CSRF. Best practice: Access token in header, refresh token in httpOnly cookie.

</details>

### 35. What is httpOnly and secure cookie?

<details>
<summary>Answer</summary>

**httpOnly**: Cookie cannot be accessed by JavaScript. **secure**: Cookie only sent over HTTPS.

```typescript
response.cookie('token', value, {
  httpOnly: true,   // Cannot access via document.cookie
  secure: true,     // HTTPS only
  sameSite: 'strict',  // CSRF protection
  maxAge: 3600000,  // Expires in 1 hour
});
```

**Interview Tip**: httpOnly prevents JavaScript access (XSS protection). secure requires HTTPS. sameSite prevents CSRF. Use all three for production.

</details>

### 36. Should you include sensitive data in JWT payload?

<details>
<summary>Answer</summary>

**No**. JWT payload is encoded (Base64), not encrypted. Anyone can decode it.

```typescript
// ✅ Good - Non-sensitive data
const payload = { sub: user.id, email: user.email, roles: user.roles };

// ❌ Bad - Sensitive data
const payload = { password: user.password, ssn: user.ssn };
```

**Interview Tip**: JWT payload is encoded, NOT encrypted. Only include: user ID, email, roles. Never include: passwords, SSNs, credit cards, API keys.

</details>

### 37. How do you implement logout?

<details>
<summary>Answer</summary>

```typescript
// auth.service.ts
async logout(userId: number) {
  await this.refreshTokenRepository.update({ userId }, { isRevoked: true });
  return { message: 'Logged out successfully' };
}

// auth.controller.ts
@Post('logout')
@UseGuards(JwtAuthGuard)
async logout(@Request() req, @Res({ passthrough: true }) response: Response) {
  await this.authService.logout(req.user.userId);
  response.clearCookie('refresh_token');
  return { message: 'Logged out successfully' };
}
```

**Interview Tip**: Logout: 1) Revoke refresh tokens in database, 2) Clear cookies, 3) Client clears localStorage. Access tokens remain valid until expiration (JWT limitation).

</details>

### 38. What is token blacklisting?

<details>
<summary>Answer</summary>

**Token blacklisting** stores revoked tokens to prevent use before expiration.

```typescript
async validate(req: Request, payload: any) {
  const token = req.headers.authorization?.replace('Bearer ', '');
  if (await this.authService.isTokenBlacklisted(token)) {
    throw new UnauthorizedException('Token revoked');
  }
  return { userId: payload.sub };
}
```

**Interview Tip**: Blacklisting stores revoked tokens. Requires database lookup. Alternative: Short-lived tokens (15min) + refresh token revocation.

</details>

### 39. How do you prevent brute force attacks?

<details>
<summary>Answer</summary>

**Prevent brute force** with rate limiting and account lockout.

```typescript
// Rate limiting
@Module({
  imports: [ThrottlerModule.forRoot([{ ttl: 60000, limit: 10 }])],
})

// Account lockout
@Column({ default: 0 })
failedLoginAttempts: number;

@Column({ nullable: true })
lockedUntil: Date;

// Lock after 5 failed attempts
if (user.failedLoginAttempts >= 5) {
  user.lockedUntil = new Date(Date.now() + 15 * 60 * 1000);
}
```

**Interview Tip**: Prevent brute force: 1) Rate limiting (5 attempts/min), 2) Account lockout (lock after 5 failures), 3) Progressive delays, 4) CAPTCHA.

</details>

## User Context

### 40. How do you access current user in controllers using custom decorator?

<details>
<summary>Answer</summary>

Create **@User() decorator** to extract user from request.

```typescript
// decorators/user.decorator.ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const User = createParamDecorator(
  (data: string, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const user = request.user;
    return data ? user?.[data] : user;
  },
);

// Usage
@Get('my-posts')
getMyPosts(@User() user: any) {
  return this.postsService.findByUserId(user.userId);
}

@Post()
create(@User('userId') userId: number, @Body() createPostDto) {
  return this.postsService.create(createPostDto, userId);
}
```

**Interview Tip**: Create @User() decorator with `createParamDecorator()`. Extracts user from request.user. Can access entire user or specific properties: `@User()` or `@User('userId')`.

</details>

### 41. How do you attach user to request in strategy?

<details>
<summary>Answer</summary>

**JwtStrategy's validate() method** attaches user to request.

```typescript
// strategies/jwt.strategy.ts
@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      secretOrKey: process.env.JWT_SECRET,
    });
  }
  
  // Return value is attached to req.user
  async validate(payload: any) {
    return {
      userId: payload.sub,
      email: payload.email,
      roles: payload.roles,
    };
  }
}
```

**Interview Tip**: JwtStrategy's `validate()` attaches user to request. Return value becomes `req.user`. Passport calls validate() after verifying JWT signature.

</details>

## Common Issues

### 42. How do you handle token expiration?

<details>
<summary>Answer</summary>

**Handle expiration** with refresh tokens.

```javascript
// Client-side auto-refresh
const apiRequest = async (url) => {
  let response = await fetch(url, {
    headers: { 'Authorization': `Bearer ${token}` },
  });
  
  // If 401, try refreshing
  if (response.status === 401) {
    const refreshResponse = await fetch('/auth/refresh', {
      method: 'POST',
      body: JSON.stringify({ refresh_token: refreshToken }),
    });
    
    if (refreshResponse.ok) {
      const data = await refreshResponse.json();
      localStorage.setItem('access_token', data.access_token);
      // Retry request
      response = await fetch(url, {
        headers: { 'Authorization': `Bearer ${data.access_token}` },
      });
    } else {
      window.location.href = '/login';
    }
  }
  return response.json();
};
```

**Interview Tip**: Handle expiration with refresh tokens. Client detects 401 → Calls refresh endpoint → Gets new token → Retries request. Use axios interceptor for automatic retry.

</details>

### 43. How do you handle CORS with authentication?

<details>
<summary>Answer</summary>

**Enable CORS** with proper configuration.

```typescript
// main.ts
app.enableCors({
  origin: ['http://localhost:3001', 'https://myapp.com'],
  credentials: true,  // Allow cookies
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization'],
});

// Client-side
fetch('http://localhost:3000/api/profile', {
  credentials: 'include',  // Send cookies
  headers: { 'Authorization': `Bearer ${token}` },
});
```

**Interview Tip**: Enable CORS with `enableCors()`. Set `credentials: true` for cookies. Client must use `credentials: 'include'`. Allow Authorization header in `allowedHeaders`.

</details>

### 44. What causes "Unauthorized" errors and how to debug?

<details>
<summary>Answer</summary>

**Common causes:**

1. **Missing Token**: No Authorization header
2. **Malformed Token**: Missing "Bearer " prefix
3. **Expired Token**: Token past expiration time
4. **Wrong Secret**: JWT_SECRET mismatch
5. **Strategy Not Registered**: JwtStrategy not in providers

**Debug Steps:**

```typescript
// Log in JwtStrategy
async validate(payload: any) {
  console.log('JWT Payload:', payload);
  console.log('Token exp:', payload.exp);
  return { userId: payload.sub };
}

// Custom guard logging
@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  handleRequest(err, user, info) {
    console.log('Guard error:', err);
    console.log('User:', user);
    console.log('Info:', info);
    
    if (err || !user) {
      throw err || new UnauthorizedException();
    }
    return user;
  }
}
```

**Interview Tip**: 401 causes: Missing token, expired token, wrong secret. Debug: Log in validate(), check token extraction, verify secret matches. Use handleRequest() to log guard errors.

</details>

---

## Summary

This guide covered 44 comprehensive questions on Authentication & Authorization in NestJS. Key topics include:

- **Fundamentals**: Authentication vs Authorization, Passport integration, common strategies
- **JWT**: Structure, configuration, token generation, access vs refresh tokens
- **Strategies**: Local (username/password), JWT (token-based), OAuth (Google)
- **Guards**: JWT Auth Guard, applying to routes, public decorator
- **Login/Register**: User registration, password hashing with bcrypt, login flow
- **RBAC**: Role-based access control, @Roles() decorator, RolesGuard
- **Refresh Tokens**: Purpose, implementation, storage (httpOnly cookies + DB)
- **OAuth 2.0**: Google OAuth integration, no password management
- **Password Management**: Reset functionality, change password
- **Security**: JWT storage, httpOnly cookies, logout, blacklisting, brute force prevention
- **User Context**: @User() decorator, attaching user to request
- **Common Issues**: Token expiration, CORS, debugging Unauthorized errors

Each answer includes production-ready code examples and interview tips for effective preparation.
