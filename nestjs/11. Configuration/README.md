# NestJS Configuration - Top Interview Questions

## Configuration Fundamentals

### 1. What is `@nestjs/config` package?

<details>
<summary>Answer</summary>

**`@nestjs/config`** is an official NestJS package that provides a **ConfigModule** and **ConfigService** for managing application configuration using environment variables and configuration files.

**Key Features:**

| Feature | Description |
|---------|-------------|
| **Environment Variables** | Load and parse `.env` files using dotenv |
| **ConfigService** | Injectable service to access configuration values |
| **Validation** | Validate configuration using Joi or class-validator |
| **Type Safety** | TypeScript generics for type-safe configuration |
| **Namespacing** | Organize configuration into logical namespaces |
| **Global Module** | Share configuration across all modules |
| **Custom Loaders** | Load configuration from multiple sources |

**Installation:**

```bash
npm install @nestjs/config
# or
yarn add @nestjs/config

# Optional: For validation
npm install joi
# or
npm install class-validator class-transformer
```

**Basic Usage:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true, // Makes ConfigModule available globally
      envFilePath: '.env', // Path to .env file
      ignoreEnvFile: false, // Set to true in production to use system env vars
    }),
  ],
})
export class AppModule {}
```

**Using ConfigService:**

```typescript
// app.service.ts
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class AppService {
  constructor(private configService: ConfigService) {}

  getDatabaseConfig() {
    // Get configuration values with type safety
    const host = this.configService.get<string>('DATABASE_HOST');
    const port = this.configService.get<number>('DATABASE_PORT');
    
    return { host, port };
  }

  // Get with default value
  getAppPort(): number {
    return this.configService.get<number>('PORT', 3000);
  }
}
```

**What it Does:**

```typescript
// Without @nestjs/config (manual approach)
// ❌ No type safety, no validation, scattered access
const dbHost = process.env.DATABASE_HOST;
const dbPort = parseInt(process.env.DATABASE_PORT || '5432');

// With @nestjs/config
// ✅ Centralized, type-safe, validated, testable
const dbHost = this.configService.get<string>('DATABASE_HOST');
const dbPort = this.configService.get<number>('DATABASE_PORT', 5432);
```

**Behind the Scenes:**

```typescript
// What ConfigModule does internally:
// 1. Loads .env file using dotenv
// 2. Parses environment variables
// 3. Validates against schema (if provided)
// 4. Makes ConfigService available for DI
// 5. Caches configuration for performance

// Simplified internal implementation
class ConfigService {
  private envConfig: Record<string, any>;

  constructor() {
    // Load environment variables
    this.envConfig = dotenv.parse(fs.readFileSync('.env'));
  }

  get<T>(key: string, defaultValue?: T): T {
    return (this.envConfig[key] as T) ?? defaultValue;
  }
}
```

**Real-World Example:**

```typescript
// config/configuration.ts
export default () => ({
  // Application settings
  port: parseInt(process.env.PORT, 10) || 3000,
  environment: process.env.NODE_ENV || 'development',
  
  // Database configuration
  database: {
    host: process.env.DATABASE_HOST || 'localhost',
    port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
    username: process.env.DATABASE_USERNAME,
    password: process.env.DATABASE_PASSWORD,
    database: process.env.DATABASE_NAME,
  },
  
  // JWT configuration
  jwt: {
    secret: process.env.JWT_SECRET,
    expiresIn: process.env.JWT_EXPIRES_IN || '1d',
  },
  
  // External APIs
  api: {
    stripeKey: process.env.STRIPE_API_KEY,
    sendgridKey: process.env.SENDGRID_API_KEY,
    awsRegion: process.env.AWS_REGION || 'us-east-1',
  },
});

// app.module.ts
import configuration from './config/configuration';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      load: [configuration], // Load custom configuration
    }),
  ],
})
export class AppModule {}
```

**Interview Tip**: `@nestjs/config` is a **wrapper around dotenv** that provides **dependency injection**, **type safety**, **validation**, and **testability** for configuration management. It uses **ConfigModule** to load configuration and **ConfigService** to access values. Key benefits: **centralized configuration**, **environment variable validation**, **type-safe access**, **easy testing**, and **production-ready** patterns.

</details>

### 2. Why do you need configuration management?

<details>
<summary>Answer</summary>

**Configuration management** is essential for separating code from environment-specific settings, enabling the same codebase to run in different environments without code changes.

**Why You Need Configuration Management:**

| Reason | Without Config Management | With Config Management |
|--------|---------------------------|------------------------|
| **Environment Flexibility** | Hardcoded values, must rebuild for each env | Change env vars without code changes |
| **Security** | Secrets in source code | Secrets in secure env files |
| **Testability** | Hard to test with different values | Easy to mock configuration |
| **Deployment** | Different builds per environment | Single build, multiple deployments |
| **Team Collaboration** | Merge conflicts on config values | Each dev has own `.env` file |
| **Scalability** | Difficult to manage many services | Centralized config per service |

**1. Environment Flexibility:**

```typescript
// ❌ BAD: Hardcoded configuration
@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost', // What if production uses different host?
      port: 5432,
      username: 'dev_user', // Different in production
      password: 'dev_password', // Security risk!
      database: 'dev_db',
    }),
  ],
})
export class AppModule {}

// ✅ GOOD: Configuration-driven
@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),
    TypeOrmModule.forRootAsync({
      inject: [ConfigService],
      useFactory: (configService: ConfigService) => ({
        type: 'postgres',
        host: configService.get('DATABASE_HOST'),
        port: configService.get('DATABASE_PORT'),
        username: configService.get('DATABASE_USERNAME'),
        password: configService.get('DATABASE_PASSWORD'),
        database: configService.get('DATABASE_NAME'),
      }),
    }),
  ],
})
export class AppModule {}
```

**2. Security and Secret Management:**

```typescript
// ❌ BAD: Secrets in source code
class PaymentService {
  private stripeKey = 'sk_live_51HxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxR'; // EXPOSED IN GIT!
  
  async processPayment(amount: number) {
    const stripe = new Stripe(this.stripeKey);
    // ...
  }
}

// ✅ GOOD: Secrets from environment
class PaymentService {
  constructor(private configService: ConfigService) {}
  
  async processPayment(amount: number) {
    // Secret loaded from secure environment
    const stripeKey = this.configService.get<string>('STRIPE_SECRET_KEY');
    const stripe = new Stripe(stripeKey);
    // ...
  }
}

// .env (not committed to git)
STRIPE_SECRET_KEY=sk_live_51Hxxxxxxxxx...

// .env.example (committed to git)
STRIPE_SECRET_KEY=your_stripe_secret_key_here
```

**3. Different Environments:**

```typescript
// Development (.env.development)
NODE_ENV=development
DATABASE_HOST=localhost
DATABASE_PORT=5432
LOG_LEVEL=debug
API_URL=http://localhost:3000
CACHE_TTL=60

// Production (.env.production)
NODE_ENV=production
DATABASE_HOST=prod-db.us-east-1.rds.amazonaws.com
DATABASE_PORT=5432
LOG_LEVEL=error
API_URL=https://api.example.com
CACHE_TTL=3600

// Testing (.env.test)
NODE_ENV=test
DATABASE_HOST=localhost
DATABASE_PORT=5433
LOG_LEVEL=silent
API_URL=http://localhost:3001
CACHE_TTL=0
```

**4. Testability:**

```typescript
// ❌ BAD: Hard to test
class EmailService {
  async sendEmail(to: string, subject: string) {
    const apiKey = 'SG.xxxxxxxxxxxxxxxxxxx'; // Hardcoded
    const sendgrid = new SendGrid(apiKey);
    await sendgrid.send({ to, subject });
  }
}

// Test must hit real SendGrid API

// ✅ GOOD: Easy to test with mock config
class EmailService {
  constructor(private configService: ConfigService) {}
  
  async sendEmail(to: string, subject: string) {
    const apiKey = this.configService.get('SENDGRID_API_KEY');
    const sendgrid = new SendGrid(apiKey);
    await sendgrid.send({ to, subject });
  }
}

// Test with mock configuration
const mockConfigService = {
  get: jest.fn().mockReturnValue('test-api-key'),
};

const service = new EmailService(mockConfigService);
// Test without hitting real API
```

**5. Twelve-Factor App Compliance:**

```typescript
// The Twelve-Factor App methodology says:
// "Store config in the environment"
// https://12factor.net/config

// ✅ GOOD: Follows 12-factor principles
// - Strict separation of config from code
// - No config files checked into source control
// - Environment variables for configuration
// - No grouping of environments (no "dev" vs "prod" code paths)

@Injectable()
export class AppService {
  constructor(private configService: ConfigService) {
    // Configuration comes from environment
    this.initializeApp();
  }
  
  private initializeApp() {
    const env = this.configService.get('NODE_ENV');
    const port = this.configService.get('PORT');
    
    console.log(`Starting app in ${env} mode on port ${port}`);
  }
}
```

**6. CI/CD Pipeline Integration:**

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Build Docker image
        run: docker build -t myapp .
      
      - name: Deploy to AWS
        env:
          # Configuration injected at deployment time
          DATABASE_HOST: ${{ secrets.PROD_DB_HOST }}
          DATABASE_PASSWORD: ${{ secrets.PROD_DB_PASSWORD }}
          JWT_SECRET: ${{ secrets.PROD_JWT_SECRET }}
          STRIPE_API_KEY: ${{ secrets.STRIPE_API_KEY }}
        run: |
          # Single build deployed with different configs
          kubectl set env deployment/myapp \
            DATABASE_HOST=$DATABASE_HOST \
            DATABASE_PASSWORD=$DATABASE_PASSWORD
```

**7. Team Development:**

```typescript
// Developer A's .env
DATABASE_HOST=localhost
DATABASE_PORT=5432
REDIS_HOST=localhost
AWS_PROFILE=dev-a

// Developer B's .env
DATABASE_HOST=192.168.1.100
DATABASE_PORT=5433
REDIS_HOST=192.168.1.101
AWS_PROFILE=dev-b

// .gitignore includes .env
// No merge conflicts on local preferences
// Each developer has their own configuration
```

**8. Feature Flags and Dynamic Behavior:**

```typescript
// Enable/disable features without code deployment
class FeatureService {
  constructor(private configService: ConfigService) {}
  
  isFeatureEnabled(feature: string): boolean {
    // Toggle features via environment variables
    return this.configService.get(`FEATURE_${feature.toUpperCase()}`) === 'true';
  }
}

// .env
FEATURE_NEW_CHECKOUT=true
FEATURE_BETA_DASHBOARD=false
FEATURE_ANALYTICS=true

// Usage
if (this.featureService.isFeatureEnabled('NEW_CHECKOUT')) {
  // Use new checkout flow
} else {
  // Use old checkout flow
}
```

**Interview Tip**: Configuration management is needed for **environment flexibility** (dev/staging/prod), **security** (secrets not in code), **testability** (mock configs), **scalability** (manage many services), **CI/CD integration** (single build, multiple deployments), **team collaboration** (no config conflicts), and **Twelve-Factor App compliance**. Key principle: **separate code from configuration** - code should be stateless, configuration should be stateful.

</details>

### 3. What is the purpose of `.env` files?

<details>
<summary>Answer</summary>

**`.env` files** store **environment variables** in a simple key-value format, allowing you to configure your application without hardcoding values or changing code.

**Purpose of `.env` Files:**

| Purpose | Description | Example |
|---------|-------------|---------|
| **Store Configuration** | Keep environment-specific settings | `PORT=3000` |
| **Manage Secrets** | Store sensitive data securely | `DATABASE_PASSWORD=secret123` |
| **Environment Separation** | Different configs per environment | `.env.dev`, `.env.prod` |
| **Developer Convenience** | Easy local development setup | Each dev has own `.env` |
| **Version Control** | Template files (`.env.example`) in git | Secrets excluded via `.gitignore` |
| **No Code Changes** | Change behavior without redeployment | Update `.env` and restart |

**Basic `.env` File Structure:**

```bash
# .env file format

# Application Settings
NODE_ENV=development
PORT=3000
APP_NAME=MyApp
APP_URL=http://localhost:3000

# Database Configuration
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_USERNAME=postgres
DATABASE_PASSWORD=secret123
DATABASE_NAME=myapp_db

# Redis Configuration
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=

# JWT Configuration
JWT_SECRET=super-secret-key-change-in-production
JWT_EXPIRES_IN=1d

# External APIs
STRIPE_API_KEY=sk_test_51xxxxxxxxxxxxx
SENDGRID_API_KEY=SG.xxxxxxxxxxxxxxxxx
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
AWS_REGION=us-east-1

# Feature Flags
FEATURE_NEW_UI=true
FEATURE_ANALYTICS=false

# Logging
LOG_LEVEL=debug
LOG_FILE=logs/app.log
```

**How NestJS Loads `.env` Files:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot({
      // Load .env file from project root
      envFilePath: '.env',
      
      // Or load multiple env files (first takes precedence)
      // envFilePath: ['.env.local', '.env'],
      
      // Ignore .env file in production (use system env vars)
      // ignoreEnvFile: process.env.NODE_ENV === 'production',
      
      isGlobal: true,
    }),
  ],
})
export class AppModule {}

// Internally uses dotenv package:
// require('dotenv').config({ path: '.env' });
```

**Accessing Variables from `.env`:**

```typescript
// Before ConfigModule loads (in main.ts)
console.log(process.env.PORT); // undefined (not loaded yet)

// After ConfigModule loads (in services/controllers)
@Injectable()
export class AppService {
  constructor(private configService: ConfigService) {
    // ✅ Using ConfigService (recommended)
    const port = this.configService.get<number>('PORT');
    
    // ✅ Using process.env (works but less recommended)
    const port2 = process.env.PORT;
  }
}
```

**Multiple Environment Files:**

```typescript
// Different .env files for each environment

// .env.development
NODE_ENV=development
DATABASE_HOST=localhost
LOG_LEVEL=debug
API_URL=http://localhost:3000

// .env.production
NODE_ENV=production
DATABASE_HOST=prod-db.example.com
LOG_LEVEL=error
API_URL=https://api.example.com

// .env.test
NODE_ENV=test
DATABASE_HOST=localhost
LOG_LEVEL=silent
API_URL=http://localhost:3001

// Load based on NODE_ENV
@Module({
  imports: [
    ConfigModule.forRoot({
      envFilePath: `.env.${process.env.NODE_ENV || 'development'}`,
      isGlobal: true,
    }),
  ],
})
export class AppModule {}
```

**`.env.example` Template File:**

```bash
# .env.example (committed to git)
# This file serves as a template for developers

# Application Settings
NODE_ENV=development
PORT=3000

# Database Configuration
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_USERNAME=your_db_username
DATABASE_PASSWORD=your_db_password
DATABASE_NAME=your_db_name

# JWT Configuration
JWT_SECRET=generate_a_secure_secret_key

# External APIs (Get your own keys)
STRIPE_API_KEY=your_stripe_key_here
SENDGRID_API_KEY=your_sendgrid_key_here
```

**`.gitignore` Configuration:**

```bash
# .gitignore

# Environment files (contains secrets)
.env
.env.local
.env.*.local

# Keep template files
!.env.example

# Keep production env file template (if needed)
!.env.production.example
```

**Security Best Practices:**

```typescript
// ❌ BAD: Secrets in code
const stripeKey = 'sk_live_51xxxxxxxxxxxxxxxxx';
const jwtSecret = 'my-secret-key';

// ✅ GOOD: Secrets in .env
// .env
STRIPE_API_KEY=sk_live_51xxxxxxxxxxxxxxxxx
JWT_SECRET=randomly-generated-secure-key

// Code
const stripeKey = this.configService.get('STRIPE_API_KEY');
const jwtSecret = this.configService.get('JWT_SECRET');
```

**Loading .env in Different Scenarios:**

```typescript
// 1. Default behavior (load .env from root)
ConfigModule.forRoot()

// 2. Custom path
ConfigModule.forRoot({
  envFilePath: './config/.env',
})

// 3. Multiple files (priority order)
ConfigModule.forRoot({
  envFilePath: [
    '.env.local',     // Highest priority
    `.env.${process.env.NODE_ENV}`,
    '.env',          // Lowest priority
  ],
})

// 4. Ignore .env file (production)
ConfigModule.forRoot({
  ignoreEnvFile: process.env.NODE_ENV === 'production',
  // Use system environment variables in production
})
```

**Variable Expansion (Referencing Other Variables):**

```bash
# .env with variable expansion
APP_NAME=MyApp
APP_VERSION=1.0.0

# Reference other variables
APP_TITLE=${APP_NAME} v${APP_VERSION}

DATABASE_URL=postgres://${DATABASE_USERNAME}:${DATABASE_PASSWORD}@${DATABASE_HOST}:${DATABASE_PORT}/${DATABASE_NAME}

# Using in code
const dbUrl = this.configService.get('DATABASE_URL');
// postgres://user:password@localhost:5432/mydb
```

**Real-World Example - Full Setup:**

```bash
# .env (local development)
# Application
NODE_ENV=development
PORT=3000
APP_NAME=E-Commerce API
APP_URL=http://localhost:3000

# Database
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_USERNAME=dev_user
DATABASE_PASSWORD=dev_pass
DATABASE_NAME=ecommerce_dev

# Redis Cache
REDIS_HOST=localhost
REDIS_PORT=6379

# JWT
JWT_SECRET=dev-secret-key-change-in-prod
JWT_EXPIRES_IN=7d

# Payment Gateway
STRIPE_PUBLIC_KEY=pk_test_xxxxx
STRIPE_SECRET_KEY=sk_test_xxxxx
STRIPE_WEBHOOK_SECRET=whsec_xxxxx

# Email Service
SENDGRID_API_KEY=SG.xxxxxxxxx
FROM_EMAIL=noreply@example.com

# AWS S3 (File uploads)
AWS_ACCESS_KEY_ID=AKIAxxxxxxxx
AWS_SECRET_ACCESS_KEY=xxxxxxxxxxxx
AWS_REGION=us-east-1
AWS_S3_BUCKET=my-app-uploads

# Monitoring
SENTRY_DSN=https://xxxxx@sentry.io/xxxxx

# Feature Flags
ENABLE_PROMOTIONS=true
ENABLE_NEW_CHECKOUT=false
```

**Interview Tip**: `.env` files store **environment variables** in key-value format for **configuration management**. Key purposes: **separate code from config**, **store secrets securely**, **support multiple environments**, **enable developer flexibility**. Best practices: **never commit `.env` to git** (use `.gitignore`), **commit `.env.example` as template**, **use different files per environment** (`.env.dev`, `.env.prod`), **load via `@nestjs/config`**, and **validate required variables**. In production, use **system environment variables** or **secret management services** (AWS Secrets Manager, HashiCorp Vault).

</details>
4. How do you install and set up `ConfigModule`?

## ConfigModule Setup

### 4. How do you install and set up `ConfigModule`?

<details>
<summary>Answer</summary>

**Installing and setting up ConfigModule** involves installing the package and configuring it in your root module.

**Step 1: Installation**

```bash
# Install @nestjs/config package
npm install @nestjs/config

# Or using yarn
yarn add @nestjs/config

# The package comes with dotenv as a dependency
# No need to install dotenv separately
```

**Step 2: Create `.env` File**

```bash
# .env (create in project root)
PORT=3000
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_USERNAME=postgres
DATABASE_PASSWORD=secret
DATABASE_NAME=myapp
JWT_SECRET=my-secret-key
```

**Step 3: Basic Setup in AppModule**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  imports: [
    // Import ConfigModule at the root
    ConfigModule.forRoot({
      isGlobal: true, // Makes ConfigModule available everywhere
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

**Step 4: Using ConfigService**

```typescript
// app.service.ts
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class AppService {
  constructor(private configService: ConfigService) {}

  getAppInfo() {
    // Access environment variables through ConfigService
    return {
      port: this.configService.get<number>('PORT'),
      database: {
        host: this.configService.get<string>('DATABASE_HOST'),
        port: this.configService.get<number>('DATABASE_PORT'),
      },
    };
  }
}
```

**Step 5: Use in main.ts (Optional)**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { ConfigService } from '@nestjs/config';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Get ConfigService from the app instance
  const configService = app.get(ConfigService);
  
  // Use config values
  const port = configService.get<number>('PORT', 3000);
  
  await app.listen(port);
  console.log(`Application is running on: http://localhost:${port}`);
}
bootstrap();
```

**Complete Project Structure:**

```
my-app/
├── src/
│   ├── app.module.ts        # ConfigModule imported here
│   ├── app.service.ts       # ConfigService used here
│   ├── main.ts             # Bootstrap with port from config
│   └── ...
├── .env                     # Environment variables (not committed)
├── .env.example            # Template (committed to git)
├── .gitignore              # Ignore .env files
├── package.json
└── tsconfig.json
```

**Advanced Setup with Options:**

```typescript
// app.module.ts - Production-ready setup
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot({
      // Make available globally (no need to import in feature modules)
      isGlobal: true,
      
      // Specify custom .env file path
      envFilePath: ['.env.local', '.env'],
      
      // Ignore .env file in production (use system env vars)
      ignoreEnvFile: process.env.NODE_ENV === 'production',
      
      // Expand variables in .env file
      expandVariables: true,
      
      // Cache environment variables for performance
      cache: true,
    }),
  ],
})
export class AppModule {}
```

**Setup with Validation:**

```bash
# Install validation library
npm install joi
```

```typescript
// app.module.ts - With Joi validation
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import * as Joi from 'joi';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      
      // Validate environment variables on startup
      validationSchema: Joi.object({
        NODE_ENV: Joi.string()
          .valid('development', 'production', 'test')
          .default('development'),
        PORT: Joi.number().default(3000),
        DATABASE_HOST: Joi.string().required(),
        DATABASE_PORT: Joi.number().default(5432),
        DATABASE_USERNAME: Joi.string().required(),
        DATABASE_PASSWORD: Joi.string().required(),
        JWT_SECRET: Joi.string().required(),
      }),
      
      // Stop app if validation fails
      validationOptions: {
        allowUnknown: true, // Allow other env vars
        abortEarly: false,  // Show all validation errors
      },
    }),
  ],
})
export class AppModule {}
```

**Setup with Custom Configuration Files:**

```typescript
// config/configuration.ts
export default () => ({
  port: parseInt(process.env.PORT, 10) || 3000,
  database: {
    host: process.env.DATABASE_HOST,
    port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
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
      load: [configuration], // Load custom config
    }),
  ],
})
export class AppModule {}
```

**Testing the Setup:**

```typescript
// app.controller.ts - Test endpoint
import { Controller, Get } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';

@Controller()
export class AppController {
  constructor(private configService: ConfigService) {}

  @Get('config')
  getConfig() {
    return {
      environment: this.configService.get('NODE_ENV'),
      port: this.configService.get('PORT'),
      databaseHost: this.configService.get('DATABASE_HOST'),
    };
  }
}

// Test: curl http://localhost:3000/config
// {
//   "environment": "development",
//   "port": "3000",
//   "databaseHost": "localhost"
// }
```

**Common Setup Patterns:**

```typescript
// Pattern 1: Minimal Setup (Quick Start)
ConfigModule.forRoot()

// Pattern 2: Global Setup (Most Common)
ConfigModule.forRoot({ isGlobal: true })

// Pattern 3: Environment-Specific
ConfigModule.forRoot({
  envFilePath: `.env.${process.env.NODE_ENV}`,
  isGlobal: true,
})

// Pattern 4: Production-Ready
ConfigModule.forRoot({
  isGlobal: true,
  envFilePath: ['.env.local', '.env'],
  ignoreEnvFile: process.env.NODE_ENV === 'production',
  validationSchema: Joi.object({ /* ... */ }),
  cache: true,
})
```

**Troubleshooting Setup Issues:**

```typescript
// Issue 1: ConfigService is undefined
// ❌ Forgot to import ConfigModule
@Module({
  imports: [], // ConfigModule missing!
  providers: [MyService],
})

// ✅ Fixed
@Module({
  imports: [ConfigModule.forRoot({ isGlobal: true })],
  providers: [MyService],
})

// Issue 2: Environment variables not loading
// ❌ .env file in wrong location
my-app/
└── config/
    └── .env  // Wrong location

// ✅ Fixed - .env should be in project root
my-app/
└── .env  // Correct location

// Or specify custom path
ConfigModule.forRoot({
  envFilePath: './config/.env',
})

// Issue 3: undefined values from ConfigService
// Check if .env file exists
// Check if environment variables are set
// Check if ConfigModule is imported before using ConfigService
```

**Interview Tip**: Setup involves **4 steps**: 1) Install `@nestjs/config`, 2) Create `.env` file, 3) Import `ConfigModule.forRoot()` in AppModule with `isGlobal: true`, 4) Inject `ConfigService` in services/controllers. For **production**, add **validation** (Joi), set `ignoreEnvFile: true` to use system env vars, enable **caching**, and use **multiple env files** for different environments. Always add `.env` to `.gitignore` and commit `.env.example` as a template.

</details>

### 5. How do you import `ConfigModule` globally using `ConfigModule.forRoot()`?

<details>
<summary>Answer</summary>

**Importing ConfigModule globally** makes ConfigService available in all modules without needing to import ConfigModule in each feature module.

**Global Import Syntax:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true, // 👈 This makes it global
    }),
  ],
})
export class AppModule {}
```

**How `isGlobal: true` Works:**

```typescript
// Without isGlobal: true (Local Registration)
// ❌ Must import ConfigModule in every feature module

// users/users.module.ts
@Module({
  imports: [ConfigModule], // Must import here
  providers: [UsersService],
})
export class UsersModule {}

// orders/orders.module.ts
@Module({
  imports: [ConfigModule], // Must import here too
  providers: [OrdersService],
})
export class OrdersModule {}

// auth/auth.module.ts
@Module({
  imports: [ConfigModule], // And here
  providers: [AuthService],
})
export class AuthModule {}
```

```typescript
// With isGlobal: true (Global Registration)
// ✅ Import once in AppModule, use everywhere

// app.module.ts
@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }), // Import once
    UsersModule,
    OrdersModule,
    AuthModule,
  ],
})
export class AppModule {}

// users/users.module.ts
@Module({
  // No need to import ConfigModule
  providers: [UsersService], // Can still inject ConfigService
})
export class UsersModule {}

// users/users.service.ts
@Injectable()
export class UsersService {
  // ✅ ConfigService available without importing ConfigModule
  constructor(private configService: ConfigService) {}
  
  getDatabaseConfig() {
    return this.configService.get('DATABASE_HOST');
  }
}
```

**Complete Example:**

```typescript
// app.module.ts - Root Module
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { UsersModule } from './users/users.module';
import { OrdersModule } from './orders/orders.module';
import { AuthModule } from './auth/auth.module';

@Module({
  imports: [
    // Global ConfigModule - import once
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: '.env',
      cache: true,
    }),
    
    // Feature modules - don't need to import ConfigModule
    UsersModule,
    OrdersModule,
    AuthModule,
  ],
})
export class AppModule {}
```

```typescript
// users/users.module.ts - Feature Module
import { Module } from '@nestjs/common';
import { UsersService } from './users.service';
import { UsersController } from './users.controller';

@Module({
  // ✅ No ConfigModule import needed
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}
```

```typescript
// users/users.service.ts - Service
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class UsersService {
  // ✅ ConfigService automatically available
  constructor(private configService: ConfigService) {}

  async createUser(userData: any) {
    // Access configuration
    const jwtSecret = this.configService.get('JWT_SECRET');
    const tokenExpiry = this.configService.get('JWT_EXPIRES_IN');
    
    // Use configuration...
  }
}
```

**Global vs Local Registration:**

| Aspect | `isGlobal: true` | `isGlobal: false` (default) |
|--------|------------------|---------------------------|
| **Import Location** | Once in AppModule | In every feature module |
| **Code Duplication** | None | Import in each module |
| **Best For** | Most applications | Library development |
| **Module Coupling** | Low | Higher |
| **Recommended** | ✅ Yes (for apps) | Only for libraries |

**When to Use Global Registration:**

```typescript
// ✅ USE GLOBAL (isGlobal: true) for:

// 1. Application Development (most common)
@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),
  ],
})
export class AppModule {}

// 2. When many modules need configuration
@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),
    UsersModule,      // Needs config
    OrdersModule,     // Needs config
    PaymentsModule,   // Needs config
    NotificationsModule, // Needs config
    // ... 20 more modules that need config
  ],
})
export class AppModule {}

// 3. Cross-cutting concerns (database, JWT, etc.)
@Injectable()
export class DatabaseService {
  constructor(private configService: ConfigService) {
    // Every module needs database config
  }
}
```

```typescript
// ❌ USE LOCAL (isGlobal: false) for:

// 1. Library/Package Development
// Allow consumers to decide where to import
@Module({
  imports: [ConfigModule.forFeature(/* ... */)],
})
export class MyLibraryModule {}

// 2. Testing specific modules in isolation
// Want explicit imports for test clarity
```

**Multiple ConfigModule Registrations:**

```typescript
// app.module.ts
@Module({
  imports: [
    // Global registration for common config
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: '.env',
    }),
    
    // Feature modules can still use forFeature()
    // for module-specific configuration
    UsersModule,
  ],
})
export class AppModule {}

// users/users.module.ts
@Module({
  imports: [
    // Additional module-specific config (merged with global)
    ConfigModule.forFeature(databaseConfig),
  ],
})
export class UsersModule {}
```

**Behind the Scenes:**

```typescript
// What isGlobal: true does internally:

// Without isGlobal
@Module({
  imports: [ConfigModule.forRoot()],
})
// ConfigModule is scoped to this module tree only

// With isGlobal: true
@Module({
  imports: [ConfigModule.forRoot({ isGlobal: true })],
})
// NestJS registers ConfigModule in global scope
// Makes ConfigModule's exports (ConfigService) available
// in all modules without explicit imports
```

**Real-World Production Setup:**

```typescript
// app.module.ts - Complete production setup
import { Module } from '@nestjs/common';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { TypeOrmModule } from '@nestjs/typeorm';
import * as Joi from 'joi';

@Module({
  imports: [
    // 1. Global ConfigModule (loaded first)
    ConfigModule.forRoot({
      isGlobal: true, // Available everywhere
      envFilePath: `.env.${process.env.NODE_ENV || 'development'}`,
      ignoreEnvFile: process.env.NODE_ENV === 'production',
      validationSchema: Joi.object({
        NODE_ENV: Joi.string().required(),
        PORT: Joi.number().required(),
        DATABASE_HOST: Joi.string().required(),
        DATABASE_PORT: Joi.number().required(),
      }),
      cache: true,
    }),
    
    // 2. Database module uses ConfigService (available globally)
    TypeOrmModule.forRootAsync({
      inject: [ConfigService], // Can inject without importing ConfigModule
      useFactory: (configService: ConfigService) => ({
        type: 'postgres',
        host: configService.get('DATABASE_HOST'),
        port: configService.get('DATABASE_PORT'),
        username: configService.get('DATABASE_USERNAME'),
        password: configService.get('DATABASE_PASSWORD'),
        database: configService.get('DATABASE_NAME'),
      }),
    }),
    
    // 3. All feature modules can use ConfigService
    UsersModule,
    AuthModule,
    OrdersModule,
  ],
})
export class AppModule {}
```

**Testing with Global ConfigModule:**

```typescript
// users.service.spec.ts
describe('UsersService', () => {
  let service: UsersService;
  
  beforeEach(async () => {
    const module = await Test.createTestingModule({
      imports: [
        // Global ConfigModule in tests
        ConfigModule.forRoot({
          isGlobal: true,
          ignoreEnvFile: true,
          load: [
            () => ({
              JWT_SECRET: 'test-secret',
              DATABASE_HOST: 'localhost',
            }),
          ],
        }),
      ],
      providers: [UsersService],
    }).compile();
    
    service = module.get<UsersService>(UsersService);
  });
  
  it('should use config', () => {
    // ConfigService available globally in tests
    expect(service).toBeDefined();
  });
});
```

**Interview Tip**: Use **`ConfigModule.forRoot({ isGlobal: true })`** in **AppModule** to make **ConfigService available everywhere** without importing ConfigModule in feature modules. This is the **recommended approach for applications** (not libraries). Benefits: **less boilerplate**, **no repeated imports**, **cleaner code**. Import **once at root**, use **everywhere**. For **production**, combine with **validation**, **caching**, and **environment-specific** env files.

</details>

### 6. What options can you pass to `ConfigModule.forRoot()`?

<details>
<summary>Answer</summary>

**`ConfigModule.forRoot()`** accepts various options to customize configuration loading, validation, and behavior.

**All Available Options:**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| **`isGlobal`** | `boolean` | `false` | Make ConfigModule available globally |
| **`cache`** | `boolean` | `false` | Cache environment variables |
| **`envFilePath`** | `string \| string[]` | `.env` | Path(s) to .env file(s) |
| **`ignoreEnvFile`** | `boolean` | `false` | Ignore .env file (use system env vars) |
| **`ignoreEnvVars`** | `boolean` | `false` | Ignore all environment variables |
| **`load`** | `Array<() => object>` | `[]` | Custom configuration factories |
| **`validationSchema`** | `Joi.Schema` | `undefined` | Joi validation schema |
| **`validationOptions`** | `object` | `undefined` | Joi validation options |
| **`validate`** | `function` | `undefined` | Custom validation function |
| **`expandVariables`** | `boolean` | `false` | Enable variable expansion in .env |

**1. Basic Options:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot({
      // Make available globally (most common)
      isGlobal: true,
      
      // Cache environment variables for performance
      cache: true,
      
      // Path to .env file
      envFilePath: '.env',
    }),
  ],
})
export class AppModule {}
```

**2. Multiple Environment Files:**

```typescript
ConfigModule.forRoot({
  // Load multiple .env files (first found takes precedence)
  envFilePath: [
    '.env.local',                           // Highest priority
    `.env.${process.env.NODE_ENV}`,        // Environment-specific
    '.env',                                 // Default
  ],
  
  isGlobal: true,
})

// Example file priority:
// 1. .env.local (personal overrides)
// 2. .env.development (if NODE_ENV=development)
// 3. .env (defaults)
```

**3. Production Configuration:**

```typescript
ConfigModule.forRoot({
  isGlobal: true,
  
  // Ignore .env file in production (use system env vars)
  ignoreEnvFile: process.env.NODE_ENV === 'production',
  
  // Enable caching
  cache: true,
  
  // Use environment-specific file in development
  envFilePath: process.env.NODE_ENV === 'production' 
    ? undefined 
    : `.env.${process.env.NODE_ENV || 'development'}`,
})
```

**4. Custom Configuration Loaders:**

```typescript
// config/database.config.ts
export default () => ({
  database: {
    host: process.env.DATABASE_HOST || 'localhost',
    port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
    username: process.env.DATABASE_USERNAME,
    password: process.env.DATABASE_PASSWORD,
  },
});

// config/app.config.ts
export default () => ({
  app: {
    port: parseInt(process.env.PORT, 10) || 3000,
    environment: process.env.NODE_ENV || 'development',
  },
});

// app.module.ts
import databaseConfig from './config/database.config';
import appConfig from './config/app.config';

ConfigModule.forRoot({
  isGlobal: true,
  
  // Load custom configuration files
  load: [databaseConfig, appConfig],
})

// Access: configService.get('database.host')
```

**5. Validation with Joi:**

```bash
npm install joi
```

```typescript
import * as Joi from 'joi';

ConfigModule.forRoot({
  isGlobal: true,
  
  // Validate environment variables
  validationSchema: Joi.object({
    // Node environment
    NODE_ENV: Joi.string()
      .valid('development', 'production', 'test', 'staging')
      .default('development'),
    
    // Application
    PORT: Joi.number().port().default(3000),
    
    // Database (required)
    DATABASE_HOST: Joi.string().required(),
    DATABASE_PORT: Joi.number().port().default(5432),
    DATABASE_USERNAME: Joi.string().required(),
    DATABASE_PASSWORD: Joi.string().required(),
    DATABASE_NAME: Joi.string().required(),
    
    // JWT (required in production)
    JWT_SECRET: Joi.string()
      .min(32)
      .required()
      .messages({
        'string.min': 'JWT_SECRET must be at least 32 characters',
        'any.required': 'JWT_SECRET is required',
      }),
    JWT_EXPIRES_IN: Joi.string().default('1d'),
    
    // Redis (optional)
    REDIS_HOST: Joi.string().optional(),
    REDIS_PORT: Joi.number().port().optional(),
  }),
  
  // Validation options
  validationOptions: {
    allowUnknown: true, // Allow env vars not in schema
    abortEarly: false,  // Show all validation errors
  },
})

// If validation fails, app throws error on startup:
// Error: Config validation error: "DATABASE_HOST" is required
```

**6. Custom Validation Function:**

```typescript
ConfigModule.forRoot({
  isGlobal: true,
  
  // Custom validation instead of Joi
  validate: (config: Record<string, any>) => {
    // Custom validation logic
    const requiredKeys = ['DATABASE_HOST', 'DATABASE_PASSWORD', 'JWT_SECRET'];
    
    for (const key of requiredKeys) {
      if (!config[key]) {
        throw new Error(`Missing required environment variable: ${key}`);
      }
    }
    
    // Validate JWT_SECRET length
    if (config.JWT_SECRET.length < 32) {
      throw new Error('JWT_SECRET must be at least 32 characters');
    }
    
    // Validate port number
    const port = parseInt(config.PORT, 10);
    if (isNaN(port) || port < 1 || port > 65535) {
      throw new Error('PORT must be a valid port number');
    }
    
    // Return validated config
    return config;
  },
})
```

**7. Variable Expansion:**

```bash
# .env with variable expansion
APP_NAME=MyApp
APP_VERSION=1.0.0

# Reference other variables using ${VAR} syntax
APP_TITLE=${APP_NAME} v${APP_VERSION}

DATABASE_USERNAME=admin
DATABASE_PASSWORD=secret
DATABASE_URL=postgres://${DATABASE_USERNAME}:${DATABASE_PASSWORD}@localhost:5432/mydb

API_BASE_URL=https://api.example.com
API_USERS_ENDPOINT=${API_BASE_URL}/users
API_ORDERS_ENDPOINT=${API_BASE_URL}/orders
```

```typescript
ConfigModule.forRoot({
  isGlobal: true,
  
  // Enable variable expansion
  expandVariables: true,
})

// Access expanded values
const appTitle = configService.get('APP_TITLE');
// Result: "MyApp v1.0.0"

const dbUrl = configService.get('DATABASE_URL');
// Result: "postgres://admin:secret@localhost:5432/mydb"
```

**8. Ignore Environment Variables:**

```typescript
ConfigModule.forRoot({
  // Ignore system environment variables entirely
  ignoreEnvVars: true,
  
  // Only use configuration from custom loaders
  load: [
    () => ({
      port: 3000,
      database: { host: 'localhost' },
    }),
  ],
})

// Useful for:
// - Testing with hardcoded config
// - Completely custom configuration source
// - Ignoring system environment pollution
```

**9. Complete Production Configuration:**

```typescript
// app.module.ts - Production-ready setup
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import * as Joi from 'joi';
import databaseConfig from './config/database.config';
import appConfig from './config/app.config';
import jwtConfig from './config/jwt.config';

@Module({
  imports: [
    ConfigModule.forRoot({
      // 1. Global availability
      isGlobal: true,
      
      // 2. Performance optimization
      cache: true,
      
      // 3. Environment-specific files
      envFilePath: [
        '.env.local',
        `.env.${process.env.NODE_ENV || 'development'}`,
        '.env',
      ],
      
      // 4. Ignore .env in production (use system env vars)
      ignoreEnvFile: process.env.NODE_ENV === 'production',
      
      // 5. Custom configuration loaders
      load: [databaseConfig, appConfig, jwtConfig],
      
      // 6. Variable expansion
      expandVariables: true,
      
      // 7. Validation
      validationSchema: Joi.object({
        NODE_ENV: Joi.string()
          .valid('development', 'production', 'test', 'staging')
          .required(),
        PORT: Joi.number().port().default(3000),
        DATABASE_HOST: Joi.string().required(),
        DATABASE_PORT: Joi.number().port().required(),
        DATABASE_USERNAME: Joi.string().required(),
        DATABASE_PASSWORD: Joi.string().required(),
        DATABASE_NAME: Joi.string().required(),
        JWT_SECRET: Joi.string().min(32).required(),
        REDIS_HOST: Joi.string().optional(),
        REDIS_PORT: Joi.number().port().optional(),
      }),
      
      // 8. Validation options
      validationOptions: {
        allowUnknown: true,  // Allow extra env vars
        abortEarly: false,   // Show all errors
      },
    }),
  ],
})
export class AppModule {}
```

**10. Testing Configuration:**

```typescript
// test/app.e2e-spec.ts
describe('AppController (e2e)', () => {
  let app: INestApplication;
  
  beforeAll(async () => {
    const moduleFixture = await Test.createTestingModule({
      imports: [
        ConfigModule.forRoot({
          isGlobal: true,
          
          // Ignore .env in tests
          ignoreEnvFile: true,
          ignoreEnvVars: true,
          
          // Use test configuration
          load: [
            () => ({
              port: 3001,
              database: {
                host: 'localhost',
                port: 5433,
                name: 'test_db',
              },
              jwt: {
                secret: 'test-secret',
              },
            }),
          ],
        }),
        AppModule,
      ],
    }).compile();
    
    app = moduleFixture.createNestApplication();
    await app.init();
  });
});
```

**Interview Tip**: **Key options** for `ConfigModule.forRoot()`: **`isGlobal: true`** (make available everywhere), **`cache: true`** (performance), **`envFilePath`** (specify .env location), **`ignoreEnvFile`** (use system env vars in production), **`load`** (custom config files), **`validationSchema`** (Joi validation), **`expandVariables`** (variable substitution). **Production setup**: enable caching, validate required vars, ignore .env file, use environment-specific files. **Best practice**: combine multiple options for robust configuration management.

</details>

### 7. What does `isGlobal: true` do?

<details>
<summary>Answer</summary>

**`isGlobal: true`** registers ConfigModule as a **global module**, making ConfigService available in all modules without needing to import ConfigModule in each feature module.

**Without `isGlobal` (Default Behavior):**

```typescript
// app.module.ts
@Module({
  imports: [
    ConfigModule.forRoot(), // isGlobal defaults to false
    UsersModule,
    OrdersModule,
  ],
})
export class AppModule {}

// users/users.module.ts - Must import ConfigModule
@Module({
  imports: [ConfigModule], // ❌ Required in every module
  providers: [UsersService],
})
export class UsersModule {}

// orders/orders.module.ts - Must import ConfigModule
@Module({
  imports: [ConfigModule], // ❌ Required here too
  providers: [OrdersService],
})
export class OrdersModule {}
```

**With `isGlobal: true` (Recommended):**

```typescript
// app.module.ts
@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }), // ✅ Import once
    UsersModule,
    OrdersModule,
  ],
})
export class AppModule {}

// users/users.module.ts - No ConfigModule import needed
@Module({
  providers: [UsersService], // ✅ Can inject ConfigService directly
})
export class UsersModule {}

// users/users.service.ts
@Injectable()
export class UsersService {
  constructor(private configService: ConfigService) {} // ✅ Available
}

// orders/orders.module.ts - No ConfigModule import needed
@Module({
  providers: [OrdersService], // ✅ ConfigService available
})
export class OrdersModule {}
```

**Benefits:**

| Benefit | Description |
|---------|-------------|
| **Less Boilerplate** | Import once instead of in every module |
| **Cleaner Code** | No repeated ConfigModule imports |
| **DRY Principle** | Don't Repeat Yourself |
| **Easier Maintenance** | Change config setup in one place |
| **Standard Practice** | Most NestJS apps use `isGlobal: true` |

**Real-World Example:**

```typescript
// Large application structure
@Module({
  imports: [
    // Import once with isGlobal
    ConfigModule.forRoot({ isGlobal: true }),
    
    // 20+ feature modules - all can use ConfigService
    UsersModule,
    AuthModule,
    OrdersModule,
    PaymentsModule,
    NotificationsModule,
    AnalyticsModule,
    ReportsModule,
    // ... 15 more modules
  ],
})
export class AppModule {}

// Each module can inject ConfigService without importing ConfigModule
@Injectable()
export class PaymentsService {
  constructor(private configService: ConfigService) {
    const stripeKey = this.configService.get('STRIPE_API_KEY');
  }
}
```

**When NOT to Use `isGlobal: true`:**

```typescript
// Library/Package development
// Let consumers decide where to import ConfigModule
@Module({
  imports: [ConfigModule.forRoot()], // isGlobal: false
  exports: [MyLibraryService],
})
export class MyLibraryModule {}
```

**Interview Tip**: `isGlobal: true` makes ConfigModule **globally available**, eliminating the need to import it in every feature module. **Use for applications** (not libraries). Benefits: **less boilerplate**, **cleaner code**, **single source of truth**. This is the **standard practice** in NestJS applications.

</details>

### 8. How do you specify custom `.env` file paths using `envFilePath`?

<details>
<summary>Answer</summary>

**`envFilePath`** option allows you to specify custom paths to `.env` files, supporting both single and multiple file paths with priority ordering.

**Single File Path:**

```typescript
// app.module.ts
ConfigModule.forRoot({
  // Default: loads .env from project root
  envFilePath: '.env',
})

// Custom path: load from config directory
ConfigModule.forRoot({
  envFilePath: './config/.env',
})

// Absolute path
ConfigModule.forRoot({
  envFilePath: '/usr/local/app/.env',
})
```

**Multiple File Paths (Priority Order):**

```typescript
// First file found takes precedence
ConfigModule.forRoot({
  envFilePath: [
    '.env.local',    // Highest priority (developer overrides)
    '.env',          // Default (fallback)
  ],
})

// Result: Variables in .env.local override those in .env
```

**Environment-Specific Files:**

```typescript
// Load different .env based on NODE_ENV
ConfigModule.forRoot({
  envFilePath: `.env.${process.env.NODE_ENV || 'development'}`,
})

// Loads:
// - .env.development (if NODE_ENV=development)
// - .env.production (if NODE_ENV=production)
// - .env.test (if NODE_ENV=test)
```

**Production-Ready Setup:**

```typescript
// app.module.ts
@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: [
        '.env.local',                              // Personal overrides
        `.env.${process.env.NODE_ENV}`,           // Environment-specific
        '.env',                                    // Default fallback
      ],
    }),
  ],
})
export class AppModule {}

// Project structure:
// my-app/
// ├── .env                  # Default values (committed)
// ├── .env.local           # Personal overrides (not committed)
// ├── .env.development     # Dev settings (committed)
// ├── .env.production      # Prod settings (committed)
// └── .env.test            # Test settings (committed)
```

**Example Configuration Files:**

```bash
# .env (default values)
PORT=3000
DATABASE_HOST=localhost
DATABASE_PORT=5432
LOG_LEVEL=info

# .env.development (overrides for development)
PORT=3001
LOG_LEVEL=debug
DATABASE_NAME=myapp_dev

# .env.production (overrides for production)
LOG_LEVEL=error
DATABASE_HOST=prod-db.example.com
DATABASE_NAME=myapp_prod

# .env.local (personal developer overrides, not committed)
DATABASE_PORT=5433
REDIS_HOST=192.168.1.100
```

**Priority Example:**

```typescript
ConfigModule.forRoot({
  envFilePath: ['.env.local', '.env.development', '.env'],
})

// If .env has: PORT=3000
// And .env.development has: PORT=3001
// And .env.local has: PORT=3002
// Result: PORT=3002 (from .env.local, highest priority)
```

**Conditional Loading:**

```typescript
// Load file based on condition
ConfigModule.forRoot({
  envFilePath: process.env.NODE_ENV === 'production'
    ? '.env.production'
    : '.env.development',
})

// Skip .env in production (use system env vars)
ConfigModule.forRoot({
  envFilePath: process.env.NODE_ENV === 'production'
    ? undefined
    : '.env',
})
```

**Complex Real-World Example:**

```typescript
// app.module.ts
const getEnvFilePaths = (): string[] => {
  const nodeEnv = process.env.NODE_ENV || 'development';
  
  const paths = [
    '.env.local',              // Personal overrides
    `.env.${nodeEnv}.local`,   // Personal env-specific
    `.env.${nodeEnv}`,         // Environment-specific
    '.env',                    // Default
  ];
  
  return paths;
};

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: getEnvFilePaths(),
      // In production, system env vars override file vars
      ignoreEnvFile: process.env.NODE_ENV === 'production',
    }),
  ],
})
export class AppModule {}
```

**.gitignore Setup:**

```bash
# .gitignore
# Ignore local environment files
.env.local
.env.*.local

# Keep template and environment-specific files
!.env.example
!.env.development
!.env.production
!.env.test
```

**Interview Tip**: Use `envFilePath` to specify **custom `.env` file locations**. Supports **single path** (string) or **multiple paths** (array) with **priority ordering** (first found wins). **Best practice**: use **multiple files** with priority order: `.env.local` (personal) → `.env.{NODE_ENV}` (environment-specific) → `.env` (defaults). In **production**, use system environment variables instead of files.

</details>

### 9. What is `ignoreEnvFile` option used for?

<details>
<summary>Answer</summary>

**`ignoreEnvFile`** option tells ConfigModule to **skip loading `.env` files** and only use environment variables from the system/container, typically used in production environments.

**Basic Usage:**

```typescript
// app.module.ts
ConfigModule.forRoot({
  // Ignore .env file (use system environment variables only)
  ignoreEnvFile: true,
})
```

**Production Pattern (Recommended):**

```typescript
// app.module.ts
ConfigModule.forRoot({
  isGlobal: true,
  
  // Use .env in development, system env vars in production
  ignoreEnvFile: process.env.NODE_ENV === 'production',
  
  // Only load .env files in non-production
  envFilePath: process.env.NODE_ENV === 'production'
    ? undefined
    : ['.env.local', '.env'],
})
```

**Why Ignore `.env` Files in Production:**

| Reason | Description |
|--------|-------------|
| **Security** | `.env` files shouldn't be in production containers |
| **Secret Management** | Use secret stores (AWS Secrets Manager, Vault) |
| **Container Standards** | Docker/K8s inject env vars at runtime |
| **12-Factor App** | Configuration via environment variables |
| **No File System** | Serverless/cloud platforms may not have file system |

**Development vs Production:**

```typescript
// Development (.env file)
// ✅ .env file loaded from project
ConfigModule.forRoot({
  ignoreEnvFile: false, // Default
  envFilePath: '.env',
})

// Developer's .env:
DATABASE_HOST=localhost
DATABASE_PORT=5432
JWT_SECRET=dev-secret-key

// Production (system environment variables)
// ✅ .env file ignored, system env vars used
ConfigModule.forRoot({
  ignoreEnvFile: true,
})

// Docker/K8s injects env vars:
// docker run -e DATABASE_HOST=prod-db.aws.com \
//            -e DATABASE_PORT=5432 \
//            -e JWT_SECRET=prod-secret-from-vault
```

**Docker Example:**

```dockerfile
# Dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

# Don't copy .env file to production image
# Environment variables injected at runtime

CMD ["node", "dist/main.js"]
```

```yaml
# docker-compose.yml (production)
version: '3.8'
services:
  app:
    image: myapp:latest
    environment:
      # Inject environment variables
      NODE_ENV: production
      DATABASE_HOST: postgres
      DATABASE_PORT: 5432
      DATABASE_USERNAME: ${DB_USER}
      DATABASE_PASSWORD: ${DB_PASSWORD}
      JWT_SECRET: ${JWT_SECRET_FROM_VAULT}
```

**Kubernetes ConfigMap/Secret:**

```yaml
# k8s-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      containers:
      - name: myapp
        image: myapp:latest
        env:
        # Environment variables from ConfigMap
        - name: DATABASE_HOST
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: database-host
        # Sensitive data from Secret
        - name: DATABASE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: database-password
```

**AWS ECS Task Definition:**

```json
{
  "family": "myapp",
  "containerDefinitions": [{
    "name": "myapp",
    "image": "myapp:latest",
    "environment": [
      {"name": "NODE_ENV", "value": "production"},
      {"name": "DATABASE_HOST", "value": "prod-db.region.rds.amazonaws.com"}
    ],
    "secrets": [
      {
        "name": "DATABASE_PASSWORD",
        "valueFrom": "arn:aws:secretsmanager:region:account:secret:db-password"
      },
      {
        "name": "JWT_SECRET",
        "valueFrom": "arn:aws:secretsmanager:region:account:secret:jwt-secret"
      }
    ]
  }]
}
```

**Complete Configuration Pattern:**

```typescript
// app.module.ts - Production-ready
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';

const isProduction = process.env.NODE_ENV === 'production';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      
      // Ignore .env files in production
      ignoreEnvFile: isProduction,
      
      // Only use .env files in non-production
      envFilePath: isProduction
        ? undefined
        : ['.env.local', `.env.${process.env.NODE_ENV}`, '.env'],
      
      // Cache for performance
      cache: true,
      
      // Validate environment variables
      validationSchema: Joi.object({
        NODE_ENV: Joi.string().required(),
        DATABASE_HOST: Joi.string().required(),
        DATABASE_PASSWORD: Joi.string().required(),
        JWT_SECRET: Joi.string().min(32).required(),
      }),
    }),
  ],
})
export class AppModule {}
```

**Testing with `ignoreEnvFile`:**

```typescript
// test setup
describe('AppController (e2e)', () => {
  beforeAll(async () => {
    const module = await Test.createTestingModule({
      imports: [
        ConfigModule.forRoot({
          ignoreEnvFile: true, // Don't load .env in tests
          ignoreEnvVars: true, // Also ignore system env vars
          
          // Use test-specific config
          load: [
            () => ({
              port: 3001,
              database: { host: 'localhost', port: 5433 },
              jwt: { secret: 'test-secret' },
            }),
          ],
        }),
      ],
    }).compile();
  });
});
```

**When to Use:**

```typescript
// ✅ USE ignoreEnvFile: true FOR:
// 1. Production deployments (Docker, K8s, ECS)
// 2. CI/CD pipelines
// 3. Serverless environments (Lambda, Cloud Functions)
// 4. When using secret management services
// 5. Container orchestration platforms

// ❌ DON'T USE ignoreEnvFile: true FOR:
// 1. Local development
// 2. Development/staging environments
// 3. When team uses .env files for consistency
```

**Security Benefits:**

```typescript
// ❌ BAD: .env file in production Docker image
# Dockerfile
COPY .env .  # DON'T DO THIS!

// ✅ GOOD: Environment variables at runtime
# docker run
docker run -e DATABASE_PASSWORD=$(vault read secret/db-password) myapp

// ConfigModule setup
ConfigModule.forRoot({
  ignoreEnvFile: process.env.NODE_ENV === 'production',
})
```

**Interview Tip**: Use `ignoreEnvFile: true` in **production** to skip `.env` files and use **system environment variables** injected by container orchestration (Docker, Kubernetes, ECS). Benefits: **better security** (no secrets in images), **follows 12-factor app**, **works with secret managers** (AWS Secrets Manager, Vault), **standard for cloud deployments**. Pattern: `ignoreEnvFile: process.env.NODE_ENV === 'production'` - use `.env` in development, system env vars in production.

</details>

## Accessing Configuration

### 10. What is `ConfigService` and how do you inject it?

<details>
<summary>Answer</summary>

**`ConfigService`** is an injectable service provided by `@nestjs/config` that allows you to access configuration values from environment variables and custom configuration files.

**Basic Injection:**

```typescript
// app.service.ts
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class AppService {
  // Inject ConfigService via constructor
  constructor(private configService: ConfigService) {}

  getAppInfo() {
    const port = this.configService.get('PORT');
    const dbHost = this.configService.get('DATABASE_HOST');
    
    return { port, dbHost };
  }
}
```

**Injection in Different Components:**

```typescript
// 1. In Services
@Injectable()
export class UsersService {
  constructor(private configService: ConfigService) {}
  
  async hashPassword(password: string) {
    const rounds = this.configService.get<number>('BCRYPT_ROUNDS', 10);
    return bcrypt.hash(password, rounds);
  }
}

// 2. In Controllers
@Controller('users')
export class UsersController {
  constructor(private configService: ConfigService) {}
  
  @Get('info')
  getInfo() {
    return {
      apiVersion: this.configService.get('API_VERSION'),
      environment: this.configService.get('NODE_ENV'),
    };
  }
}

// 3. In Guards
@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private configService: ConfigService) {}
  
  canActivate(context: ExecutionContext): boolean {
    const jwtSecret = this.configService.get('JWT_SECRET');
    // Use jwtSecret for verification
    return true;
  }
}

// 4. In Middleware
@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  constructor(private configService: ConfigService) {}
  
  use(req: Request, res: Response, next: NextFunction) {
    const logLevel = this.configService.get('LOG_LEVEL');
    console.log(`[${logLevel}] ${req.method} ${req.path}`);
    next();
  }
}

// 5. In Interceptors
@Injectable()
export class TimeoutInterceptor implements NestInterceptor {
  constructor(private configService: ConfigService) {}
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const timeout = this.configService.get<number>('REQUEST_TIMEOUT', 5000);
    return next.handle().pipe(timeout(timeout));
  }
}
```

**Injection in Async Providers:**

```typescript
// Database provider using ConfigService
@Module({
  imports: [ConfigModule],
  providers: [
    {
      provide: 'DATABASE_CONNECTION',
      inject: [ConfigService], // Inject ConfigService
      useFactory: async (configService: ConfigService) => {
        const host = configService.get('DATABASE_HOST');
        const port = configService.get('DATABASE_PORT');
        
        return createConnection({ host, port });
      },
    },
  ],
})
export class DatabaseModule {}
```

**Using in main.ts:**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { ConfigService } from '@nestjs/config';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Get ConfigService from app instance
  const configService = app.get(ConfigService);
  
  // Use configuration
  const port = configService.get<number>('PORT', 3000);
  const corsOrigin = configService.get('CORS_ORIGIN');
  
  // Configure CORS
  app.enableCors({
    origin: corsOrigin,
  });
  
  await app.listen(port);
  console.log(`Application running on port ${port}`);
}
bootstrap();
```

**Type-Safe Injection:**

```typescript
// config/configuration.interface.ts
export interface AppConfig {
  port: number;
  environment: string;
  database: {
    host: string;
    port: number;
    name: string;
  };
}

// service with type-safe config
@Injectable()
export class AppService {
  constructor(private configService: ConfigService<AppConfig>) {}
  
  getPort(): number {
    return this.configService.get<number>('port');
  }
  
  getDatabaseHost(): string {
    return this.configService.get('database.host');
  }
}
```

**Without Global Module:**

```typescript
// If ConfigModule is NOT global, must import in feature module
@Module({
  imports: [ConfigModule], // Import ConfigModule
  providers: [UsersService],
})
export class UsersModule {}

@Injectable()
export class UsersService {
  constructor(private configService: ConfigService) {} // Now available
}
```

**With Global Module (Recommended):**

```typescript
// app.module.ts
@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }), // Global
  ],
})
export class AppModule {}

// users.module.ts - No ConfigModule import needed
@Module({
  providers: [UsersService], // ConfigService automatically available
})
export class UsersModule {}

@Injectable()
export class UsersService {
  constructor(private configService: ConfigService) {} // Available globally
}
```

**Multiple ConfigService Instances:**

```typescript
// Each instance shares the same configuration
@Injectable()
export class Service1 {
  constructor(private configService: ConfigService) {}
}

@Injectable()
export class Service2 {
  constructor(private configService: ConfigService) {}
}

// Both access the same configuration values
```

**Interview Tip**: **ConfigService** is an injectable service from `@nestjs/config` for accessing configuration. **Inject via constructor** in services, controllers, guards, middleware, interceptors. Use **`private configService: ConfigService`** in constructor. If ConfigModule is **global**, ConfigService is available everywhere without importing. Use **generics** (`ConfigService<T>`) for type safety. Can inject in **async providers** with `inject: [ConfigService]`.

</details>

### 11. How do you get configuration values using `ConfigService.get()`?

<details>
<summary>Answer</summary>

**`ConfigService.get()`** retrieves configuration values by key, with support for type safety, default values, and nested properties.

**Basic Usage:**

```typescript
@Injectable()
export class AppService {
  constructor(private configService: ConfigService) {}

  getConfig() {
    // Get string value
    const host = this.configService.get('DATABASE_HOST');
    
    // Get number value
    const port = this.configService.get('DATABASE_PORT');
    
    // Get boolean value
    const debug = this.configService.get('DEBUG_MODE');
    
    return { host, port, debug };
  }
}
```

**Type-Safe get() with Generics:**

```typescript
@Injectable()
export class DatabaseService {
  constructor(private configService: ConfigService) {}

  getConnectionConfig() {
    // Specify return type with generics
    const host = this.configService.get<string>('DATABASE_HOST');
    const port = this.configService.get<number>('DATABASE_PORT');
    const ssl = this.configService.get<boolean>('DATABASE_SSL');
    const timeout = this.configService.get<number>('DATABASE_TIMEOUT');
    
    return { host, port, ssl, timeout };
  }
}
```

**With Default Values:**

```typescript
@Injectable()
export class AppService {
  constructor(private configService: ConfigService) {}

  getConfig() {
    // Provide default value as second parameter
    const port = this.configService.get<number>('PORT', 3000);
    const env = this.configService.get<string>('NODE_ENV', 'development');
    const maxRetries = this.configService.get<number>('MAX_RETRIES', 3);
    const enableCache = this.configService.get<boolean>('ENABLE_CACHE', true);
    
    return { port, env, maxRetries, enableCache };
  }
}
```

**Accessing Nested Configuration:**

```typescript
// config/database.config.ts
export default () => ({
  database: {
    host: process.env.DATABASE_HOST,
    port: parseInt(process.env.DATABASE_PORT, 10),
    credentials: {
      username: process.env.DATABASE_USERNAME,
      password: process.env.DATABASE_PASSWORD,
    },
  },
});

// Load in ConfigModule
ConfigModule.forRoot({
  load: [databaseConfig],
})

// Access nested values using dot notation
@Injectable()
export class DatabaseService {
  constructor(private configService: ConfigService) {}

  getConfig() {
    // Nested access with dot notation
    const host = this.configService.get<string>('database.host');
    const port = this.configService.get<number>('database.port');
    const username = this.configService.get<string>('database.credentials.username');
    const password = this.configService.get<string>('database.credentials.password');
    
    return { host, port, username, password };
  }
}
```

**Get Entire Object:**

```typescript
// Get entire nested object
const databaseConfig = this.configService.get('database');
// Returns: { host, port, credentials: { username, password } }

const credentials = this.configService.get('database.credentials');
// Returns: { username, password }
```

**Using getOrThrow():**

```typescript
@Injectable()
export class AppService {
  constructor(private configService: ConfigService) {}

  getRequiredConfig() {
    // Throws error if value is undefined
    const jwtSecret = this.configService.getOrThrow<string>('JWT_SECRET');
    const dbPassword = this.configService.getOrThrow<string>('DATABASE_PASSWORD');
    
    // With custom error message
    const apiKey = this.configService.getOrThrow<string>(
      'API_KEY',
      'API_KEY is required for production'
    );
    
    return { jwtSecret, dbPassword, apiKey };
  }
}

// If JWT_SECRET is not set:
// Error: Missing required configuration: JWT_SECRET
```

**Real-World Examples:**

```typescript
// 1. Database Configuration
@Injectable()
export class DatabaseService {
  private readonly config: any;
  
  constructor(private configService: ConfigService) {
    this.config = {
      type: 'postgres',
      host: this.configService.get('DATABASE_HOST', 'localhost'),
      port: this.configService.get<number>('DATABASE_PORT', 5432),
      username: this.configService.get('DATABASE_USERNAME'),
      password: this.configService.get('DATABASE_PASSWORD'),
      database: this.configService.get('DATABASE_NAME'),
      synchronize: this.configService.get<boolean>('DATABASE_SYNC', false),
      logging: this.configService.get<boolean>('DATABASE_LOGGING', false),
    };
  }
}

// 2. JWT Configuration
@Injectable()
export class AuthService {
  constructor(private configService: ConfigService) {}

  generateToken(payload: any) {
    const secret = this.configService.getOrThrow<string>('JWT_SECRET');
    const expiresIn = this.configService.get<string>('JWT_EXPIRES_IN', '1d');
    
    return jwt.sign(payload, secret, { expiresIn });
  }
}

// 3. External API Configuration
@Injectable()
export class PaymentService {
  constructor(private configService: ConfigService) {}

  async processPayment(amount: number) {
    const stripeKey = this.configService.getOrThrow<string>('STRIPE_SECRET_KEY');
    const webhookSecret = this.configService.get('STRIPE_WEBHOOK_SECRET');
    const currency = this.configService.get<string>('PAYMENT_CURRENCY', 'USD');
    
    const stripe = new Stripe(stripeKey);
    return stripe.paymentIntents.create({ amount, currency });
  }
}

// 4. Feature Flags
@Injectable()
export class FeatureService {
  constructor(private configService: ConfigService) {}

  isFeatureEnabled(feature: string): boolean {
    const key = `FEATURE_${feature.toUpperCase()}`;
    return this.configService.get<boolean>(key, false);
  }
}

// Usage
if (this.featureService.isFeatureEnabled('new_checkout')) {
  // Use new checkout flow
}
```

**Common Patterns:**

```typescript
@Injectable()
export class AppConfig {
  constructor(private configService: ConfigService) {}

  // Getter methods for clean access
  get port(): number {
    return this.configService.get<number>('PORT', 3000);
  }

  get isDevelopment(): boolean {
    return this.configService.get('NODE_ENV') === 'development';
  }

  get isProduction(): boolean {
    return this.configService.get('NODE_ENV') === 'production';
  }

  get databaseUrl(): string {
    const host = this.configService.get('DATABASE_HOST');
    const port = this.configService.get('DATABASE_PORT');
    const user = this.configService.get('DATABASE_USERNAME');
    const pass = this.configService.get('DATABASE_PASSWORD');
    const db = this.configService.get('DATABASE_NAME');
    
    return `postgres://${user}:${pass}@${host}:${port}/${db}`;
  }
}
```

**Type Conversions:**

```typescript
@Injectable()
export class ConfigHelper {
  constructor(private configService: ConfigService) {}

  // String to number
  getNumber(key: string, defaultValue: number = 0): number {
    const value = this.configService.get(key);
    return value ? parseInt(value, 10) : defaultValue;
  }

  // String to boolean
  getBoolean(key: string, defaultValue: boolean = false): boolean {
    const value = this.configService.get(key);
    return value === 'true' || value === '1' || defaultValue;
  }

  // String to array
  getArray(key: string, separator: string = ','): string[] {
    const value = this.configService.get(key);
    return value ? value.split(separator).map(v => v.trim()) : [];
  }
}

// Usage
const port = this.configHelper.getNumber('PORT', 3000);
const debug = this.configHelper.getBoolean('DEBUG', false);
const allowedOrigins = this.configHelper.getArray('CORS_ORIGINS'); // ['http://localhost:3000', 'https://app.com']
```

**Caching Configuration Values:**

```typescript
@Injectable()
export class CachedConfigService {
  private cache = new Map<string, any>();

  constructor(private configService: ConfigService) {}

  get<T>(key: string, defaultValue?: T): T {
    // Check cache first
    if (this.cache.has(key)) {
      return this.cache.get(key);
    }

    // Get from ConfigService
    const value = this.configService.get<T>(key, defaultValue);
    
    // Cache the value
    this.cache.set(key, value);
    
    return value;
  }
}
```

**Interview Tip**: Use **`configService.get<T>(key, defaultValue)`** to retrieve configuration values. **First parameter**: key name (supports dot notation for nested values). **Second parameter**: optional default value. Use **generics** (`<T>`) for type safety. Use **`getOrThrow()`** for required values (throws if missing). Access **nested config** with dot notation: `'database.host'`. Common pattern: **cache frequently accessed values** or create **getter methods** for clean access.

</details>

### 12. How do you provide default values in `get()` method?

<details>
<summary>Answer</summary>

**Default values** in `ConfigService.get()` are provided as the **second parameter** and are used when the configuration key is undefined or missing.

**Basic Syntax:**

```typescript
// configService.get<T>(key, defaultValue)
const port = this.configService.get<number>('PORT', 3000);
//                                          key ↑     ↑ default value
```

**Common Default Value Patterns:**

```typescript
@Injectable()
export class AppService {
  constructor(private configService: ConfigService) {}

  getAppConfig() {
    // String defaults
    const env = this.configService.get<string>('NODE_ENV', 'development');
    const logLevel = this.configService.get<string>('LOG_LEVEL', 'info');
    
    // Number defaults
    const port = this.configService.get<number>('PORT', 3000);
    const timeout = this.configService.get<number>('REQUEST_TIMEOUT', 5000);
    const maxRetries = this.configService.get<number>('MAX_RETRIES', 3);
    
    // Boolean defaults
    const debug = this.configService.get<boolean>('DEBUG_MODE', false);
    const enableCache = this.configService.get<boolean>('ENABLE_CACHE', true);
    const ssl = this.configService.get<boolean>('DATABASE_SSL', false);
    
    // Array defaults (from comma-separated string)
    const origins = this.configService.get<string>('CORS_ORIGINS', '').split(',');
    
    return {
      env, logLevel, port, timeout, maxRetries,
      debug, enableCache, ssl, origins,
    };
  }
}
```

**Database Configuration with Defaults:**

```typescript
@Injectable()
export class DatabaseConfig {
  constructor(private configService: ConfigService) {}

  get config() {
    return {
      type: 'postgres' as const,
      
      // Connection defaults
      host: this.configService.get<string>('DATABASE_HOST', 'localhost'),
      port: this.configService.get<number>('DATABASE_PORT', 5432),
      
      // Credentials (no defaults - should be required)
      username: this.configService.getOrThrow<string>('DATABASE_USERNAME'),
      password: this.configService.getOrThrow<string>('DATABASE_PASSWORD'),
      database: this.configService.getOrThrow<string>('DATABASE_NAME'),
      
      // Connection pool defaults
      maxConnections: this.configService.get<number>('DATABASE_MAX_CONNECTIONS', 10),
      minConnections: this.configService.get<number>('DATABASE_MIN_CONNECTIONS', 2),
      connectionTimeout: this.configService.get<number>('DATABASE_TIMEOUT', 30000),
      
      // Feature flags with defaults
      synchronize: this.configService.get<boolean>('DATABASE_SYNC', false),
      logging: this.configService.get<boolean>('DATABASE_LOGGING', false),
      ssl: this.configService.get<boolean>('DATABASE_SSL', false),
    };
  }
}
```

**When Value Exists vs When Missing:**

```bash
# .env file
PORT=4000
DEBUG_MODE=true
# MAX_RETRIES is not set
```

```typescript
@Injectable()
export class AppService {
  constructor(private configService: ConfigService) {}

  getConfig() {
    // PORT exists in .env → returns 4000
    const port = this.configService.get<number>('PORT', 3000);
    console.log(port); // 4000 (from .env)
    
    // DEBUG_MODE exists in .env → returns 'true' (string)
    const debug = this.configService.get('DEBUG_MODE', 'false');
    console.log(debug); // 'true' (from .env)
    
    // MAX_RETRIES missing from .env → returns default
    const retries = this.configService.get<number>('MAX_RETRIES', 3);
    console.log(retries); // 3 (default value)
    
    // UNKNOWN_KEY missing → returns default
    const unknown = this.configService.get<string>('UNKNOWN_KEY', 'fallback');
    console.log(unknown); // 'fallback' (default value)
  }
}
```

**Object/Complex Defaults:**

```typescript
@Injectable()
export class RedisConfig {
  constructor(private configService: ConfigService) {}

  get config() {
    // Simple defaults
    const host = this.configService.get<string>('REDIS_HOST', 'localhost');
    const port = this.configService.get<number>('REDIS_PORT', 6379);
    
    // Complex object defaults
    const retryStrategy = this.configService.get('REDIS_RETRY_STRATEGY', {
      retries: 3,
      factor: 2,
      minTimeout: 1000,
      maxTimeout: 60000,
    });
    
    return { host, port, retryStrategy };
  }
}
```

**Environment-Specific Defaults:**

```typescript
@Injectable()
export class AppConfig {
  constructor(private configService: ConfigService) {}

  private get isDevelopment(): boolean {
    return this.configService.get('NODE_ENV') === 'development';
  }

  get logLevel(): string {
    // Different defaults for different environments
    const defaultLevel = this.isDevelopment ? 'debug' : 'error';
    return this.configService.get<string>('LOG_LEVEL', defaultLevel);
  }

  get cacheTimeout(): number {
    // Longer cache in production
    const defaultTimeout = this.isDevelopment ? 60 : 3600;
    return this.configService.get<number>('CACHE_TTL', defaultTimeout);
  }

  get corsOrigins(): string[] {
    // Allow all in development, specific in production
    const defaultOrigins = this.isDevelopment 
      ? ['*'] 
      : ['https://app.example.com'];
    
    const origins = this.configService.get<string>('CORS_ORIGINS');
    return origins ? origins.split(',') : defaultOrigins;
  }
}
```

**Computed Defaults:**

```typescript
@Injectable()
export class JwtConfig {
  constructor(private configService: ConfigService) {}

  get secret(): string {
    // Generate random secret if not provided (development only)
    const isDev = this.configService.get('NODE_ENV') === 'development';
    const defaultSecret = isDev 
      ? crypto.randomBytes(32).toString('hex') 
      : undefined; // Force error in production
    
    return this.configService.get<string>('JWT_SECRET', defaultSecret);
  }

  get expiresIn(): string {
    return this.configService.get<string>('JWT_EXPIRES_IN', '1d');
  }
}
```

**Best Practices:**

```typescript
@Injectable()
export class ConfigService {
  constructor(private config: ConfigService) {}

  // ✅ GOOD: Provide sensible defaults for optional config
  getCacheTimeout(): number {
    return this.config.get<number>('CACHE_TIMEOUT', 300); // 5 minutes
  }

  getLogLevel(): string {
    return this.config.get<string>('LOG_LEVEL', 'info');
  }

  // ✅ GOOD: No default for required/sensitive config
  getDatabasePassword(): string {
    return this.config.getOrThrow<string>('DATABASE_PASSWORD');
    // Throws if not set
  }

  getJwtSecret(): string {
    return this.config.getOrThrow<string>('JWT_SECRET');
    // No default for security-critical values
  }

  // ❌ BAD: Default for security-critical config
  getBadJwtSecret(): string {
    return this.config.get<string>('JWT_SECRET', 'insecure-default');
    // Never provide defaults for secrets!
  }
}
```

**Default Value Priority:**

```typescript
// Priority order (highest to lowest):
// 1. Environment variable (from .env or system)
// 2. Default value in get()
// 3. undefined (if no default provided)

const value = this.configService.get<string>('MY_KEY', 'default-value');

// If MY_KEY=production in .env → returns 'production'
// If MY_KEY not set → returns 'default-value'
// If no default provided → returns undefined
```

**Interview Tip**: Provide **default values** as the **second parameter** in `configService.get()`: `get<T>(key, defaultValue)`. Use defaults for **optional configuration** (port, timeout, log level). **Never provide defaults** for **security-critical values** (passwords, secrets, API keys) - use `getOrThrow()` instead. Defaults can be **environment-specific** (different for dev vs prod). **Best practice**: sensible defaults for convenience, explicit requirements for security.

</details>

### 13. How do you access nested configuration?

<details>
<summary>Answer</summary>

**Nested configuration** is accessed using **dot notation** in the configuration key, allowing you to organize config into logical groups.

**Creating Nested Configuration:**

```typescript
// config/configuration.ts
export default () => ({
  // Top-level
  port: parseInt(process.env.PORT, 10) || 3000,
  
  // Nested database config
  database: {
    host: process.env.DATABASE_HOST || 'localhost',
    port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
    username: process.env.DATABASE_USERNAME,
    password: process.env.DATABASE_PASSWORD,
    options: {
      ssl: process.env.DATABASE_SSL === 'true',
      synchronize: process.env.DATABASE_SYNC === 'true',
      logging: process.env.DATABASE_LOGGING === 'true',
    },
  },
  
  // Nested JWT config
  jwt: {
    secret: process.env.JWT_SECRET,
    expiresIn: process.env.JWT_EXPIRES_IN || '1d',
    refreshExpiresIn: process.env.JWT_REFRESH_EXPIRES_IN || '7d',
  },
  
  // Nested API config
  api: {
    stripe: {
      publicKey: process.env.STRIPE_PUBLIC_KEY,
      secretKey: process.env.STRIPE_SECRET_KEY,
      webhookSecret: process.env.STRIPE_WEBHOOK_SECRET,
    },
    sendgrid: {
      apiKey: process.env.SENDGRID_API_KEY,
      fromEmail: process.env.SENDGRID_FROM_EMAIL,
    },
  },
});

// Load in AppModule
@Module({
  imports: [
    ConfigModule.forRoot({
      load: [configuration],
      isGlobal: true,
    }),
  ],
})
export class AppModule {}
```

**Accessing with Dot Notation:**

```typescript
@Injectable()
export class AppService {
  constructor(private configService: ConfigService) {}

  getConfig() {
    // Access nested properties with dot notation
    
    // Level 1 nesting
    const dbHost = this.configService.get<string>('database.host');
    const jwtSecret = this.configService.get<string>('jwt.secret');
    
    // Level 2 nesting
    const dbSsl = this.configService.get<boolean>('database.options.ssl');
    const stripeKey = this.configService.get<string>('api.stripe.secretKey');
    
    // Level 3+ nesting
    const deepValue = this.configService.get('some.very.deep.nested.value');
    
    return { dbHost, jwtSecret, dbSsl, stripeKey };
  }
}
```

**Get Entire Object:**

```typescript
@Injectable()
export class DatabaseService {
  constructor(private configService: ConfigService) {}

  getConfig() {
    // Get entire nested object
    const dbConfig = this.configService.get('database');
    // Returns: { host, port, username, password, options: { ssl, synchronize, logging } }
    
    // Get nested object
    const dbOptions = this.configService.get('database.options');
    // Returns: { ssl, synchronize, logging }
    
    // Get specific value
    const dbHost = this.configService.get('database.host');
    // Returns: 'localhost'
  }
}
```

**Type-Safe Nested Access:**

```typescript
// Define interface for type safety
interface DatabaseConfig {
  host: string;
  port: number;
  username: string;
  password: string;
  options: {
    ssl: boolean;
    synchronize: boolean;
    logging: boolean;
  };
}

interface AppConfig {
  port: number;
  database: DatabaseConfig;
  jwt: {
    secret: string;
    expiresIn: string;
  };
}

@Injectable()
export class TypeSafeService {
  constructor(private configService: ConfigService) {}

  getDatabaseConfig(): DatabaseConfig {
    return this.configService.get<DatabaseConfig>('database');
  }

  getJwtSecret(): string {
    return this.configService.get<string>('jwt.secret');
  }
}
```

**Real-World Example - Database:**

```typescript
// config/database.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('database', () => ({
  type: 'postgres',
  host: process.env.DATABASE_HOST || 'localhost',
  port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
  username: process.env.DATABASE_USERNAME,
  password: process.env.DATABASE_PASSWORD,
  database: process.env.DATABASE_NAME,
  pool: {
    min: parseInt(process.env.DATABASE_POOL_MIN, 10) || 2,
    max: parseInt(process.env.DATABASE_POOL_MAX, 10) || 10,
    idleTimeout: parseInt(process.env.DATABASE_IDLE_TIMEOUT, 10) || 30000,
  },
  options: {
    ssl: process.env.DATABASE_SSL === 'true',
    synchronize: process.env.DATABASE_SYNC === 'true',
  },
}));

// Access nested config
@Injectable()
export class DatabaseService {
  constructor(private configService: ConfigService) {}

  createConnection() {
    return {
      type: this.configService.get('database.type'),
      host: this.configService.get('database.host'),
      port: this.configService.get('database.port'),
      username: this.configService.get('database.username'),
      password: this.configService.get('database.password'),
      
      // Nested pool config
      poolMin: this.configService.get('database.pool.min'),
      poolMax: this.configService.get('database.pool.max'),
      
      // Nested options
      ssl: this.configService.get('database.options.ssl'),
    };
  }
}
```

**Multiple Nested Configurations:**

```typescript
// config/app.config.ts
export default () => ({
  server: {
    port: parseInt(process.env.PORT, 10) || 3000,
    host: process.env.HOST || '0.0.0.0',
    cors: {
      enabled: process.env.CORS_ENABLED === 'true',
      origins: process.env.CORS_ORIGINS?.split(',') || ['*'],
    },
  },
  security: {
    rateLimit: {
      windowMs: parseInt(process.env.RATE_LIMIT_WINDOW, 10) || 15 * 60 * 1000,
      max: parseInt(process.env.RATE_LIMIT_MAX, 10) || 100,
    },
    helmet: {
      enabled: process.env.HELMET_ENABLED !== 'false',
    },
  },
  cache: {
    redis: {
      host: process.env.REDIS_HOST || 'localhost',
      port: parseInt(process.env.REDIS_PORT, 10) || 6379,
      ttl: parseInt(process.env.REDIS_TTL, 10) || 300,
    },
  },
});

// Access various nested configs
@Injectable()
export class AppConfig {
  constructor(private configService: ConfigService) {}

  getServerPort(): number {
    return this.configService.get<number>('server.port');
  }

  getCorsOrigins(): string[] {
    return this.configService.get<string[]>('server.cors.origins');
  }

  getRateLimitConfig() {
    return {
      windowMs: this.configService.get<number>('security.rateLimit.windowMs'),
      max: this.configService.get<number>('security.rateLimit.max'),
    };
  }

  getRedisConfig() {
    return this.configService.get('cache.redis');
  }
}
```

**Default Values with Nested Config:**

```typescript
@Injectable()
export class ConfigService {
  constructor(private config: ConfigService) {}

  // Nested config with defaults
  getDatabaseHost(): string {
    return this.config.get<string>('database.host', 'localhost');
  }

  getDatabasePoolSize(): number {
    return this.config.get<number>('database.pool.max', 10);
  }

  getJwtExpiry(): string {
    return this.config.get<string>('jwt.expiresIn', '1d');
  }

  getStripeWebhookSecret(): string {
    return this.config.get<string>('api.stripe.webhookSecret', '');
  }
}
```

**Interview Tip**: Access **nested configuration** using **dot notation**: `'database.host'`, `'database.options.ssl'`. Create nested config by returning **objects** in custom configuration files. Use `registerAs()` for **namespaced config**. Can retrieve **entire objects** (`get('database')`) or **specific nested values** (`get('database.host')`). Supports **multiple levels** of nesting. Use **TypeScript interfaces** for type safety. **Best practice**: organize related config into logical groups (database, jwt, api, etc.).

</details>

## Environment Variables

### 14. How do you define environment variables in `.env` files?

<details>
<summary>Answer</summary>

**Environment variables** in `.env` files are defined using **`KEY=value`** syntax, with one variable per line.

**Basic `.env` File Syntax:**

```bash
# .env file

# Application Configuration
NODE_ENV=development
PORT=3000
HOST=localhost

# Database Configuration
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_USERNAME=admin
DATABASE_PASSWORD=secret123
DATABASE_NAME=myapp_db

# JWT Configuration
JWT_SECRET=your-secret-key-here
JWT_EXPIRES_IN=1d
JWT_REFRESH_EXPIRES_IN=7d

# External API Keys
STRIPE_SECRET_KEY=sk_test_xxxxxxxxxxxxx
STRIPE_PUBLIC_KEY=pk_test_xxxxxxxxxxxxx
SENDGRID_API_KEY=SG.xxxxxxxxxxxxx

# Feature Flags
ENABLE_CACHE=true
DEBUG_MODE=false
```

**Syntax Rules:**

```bash
# 1. No spaces around equals sign
PORT=3000                    # ✅ Correct
PORT = 3000                  # ❌ Wrong

# 2. No quotes needed for simple values
DATABASE_NAME=myapp          # ✅ Correct
DATABASE_NAME="myapp"        # ⚠️ Works, but quotes are included in value

# 3. Use quotes for values with spaces
APP_NAME="My Application"    # ✅ Correct
APP_NAME=My Application      # ❌ Wrong (only "My" will be captured)

# 4. Use quotes for special characters
PASSWORD="P@ssw0rd!#$"       # ✅ Correct
URL="https://example.com?key=value&other=test"  # ✅ Correct

# 5. Comments start with #
# This is a comment
PORT=3000  # This is also a comment

# 6. Empty values
OPTIONAL_VALUE=              # Empty string
# DISABLED_VALUE=            # Commented out (undefined)

# 7. Multi-line values (must use quotes)
PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA...
...more lines...
-----END RSA PRIVATE KEY-----"
```

**Data Types in `.env` (All Stored as Strings):**

```bash
# String values
APP_NAME=MyApp
LOG_LEVEL=info

# Number values (stored as strings)
PORT=3000
MAX_CONNECTIONS=100
TIMEOUT=5000

# Boolean values (stored as strings)
DEBUG_MODE=true
ENABLE_SSL=false
USE_CACHE=1               # Can use 1/0 for boolean

# Array values (comma-separated strings)
CORS_ORIGINS=http://localhost:3000,https://app.com
ALLOWED_IPS=192.168.1.1,192.168.1.2,192.168.1.3

# JSON values (as strings)
DATABASE_OPTIONS={"ssl":true,"logging":false}
```

**Converting Types in Configuration:**

```typescript
// config/configuration.ts
export default () => ({
  // Parse numbers
  port: parseInt(process.env.PORT, 10) || 3000,
  timeout: parseInt(process.env.TIMEOUT, 10) || 5000,
  
  // Parse booleans
  debugMode: process.env.DEBUG_MODE === 'true',
  enableSsl: process.env.ENABLE_SSL === 'true',
  useCache: process.env.USE_CACHE === '1',
  
  // Parse arrays
  corsOrigins: process.env.CORS_ORIGINS?.split(',') || [],
  allowedIps: process.env.ALLOWED_IPS?.split(',').map(ip => ip.trim()) || [],
  
  // Parse JSON
  databaseOptions: JSON.parse(process.env.DATABASE_OPTIONS || '{}'),
});
```

**Multiple Environment Files:**

```bash
# .env (base/default configuration)
NODE_ENV=development
PORT=3000
DATABASE_HOST=localhost

# .env.development (development overrides)
DEBUG_MODE=true
DATABASE_LOGGING=true
LOG_LEVEL=debug

# .env.production (production overrides)
DEBUG_MODE=false
DATABASE_LOGGING=false
LOG_LEVEL=error
DATABASE_HOST=prod-db.example.com

# .env.test (testing overrides)
NODE_ENV=test
PORT=3001
DATABASE_NAME=myapp_test
```

**Loading Multiple Files:**

```typescript
// app.module.ts
@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: [
        '.env',                           // Base config
        `.env.${process.env.NODE_ENV}`,  // Environment-specific
        '.env.local',                     // Local overrides (not committed)
      ],
    }),
  ],
})
export class AppModule {}
```

**Real-World Example - Complete `.env` File:**

```bash
# .env

# ===================================
# Application Settings
# ===================================
NODE_ENV=development
APP_NAME=My NestJS App
APP_URL=http://localhost:3000
PORT=3000
HOST=0.0.0.0

# ===================================
# Database Configuration
# ===================================
DATABASE_TYPE=postgres
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_USERNAME=admin
DATABASE_PASSWORD=secretPassword123!
DATABASE_NAME=myapp_development
DATABASE_SYNC=false
DATABASE_LOGGING=true
DATABASE_SSL=false
DATABASE_MAX_CONNECTIONS=10
DATABASE_MIN_CONNECTIONS=2

# ===================================
# Redis Configuration
# ===================================
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=
REDIS_TTL=300

# ===================================
# JWT Configuration
# ===================================
JWT_SECRET=your-super-secret-jwt-key-change-in-production
JWT_EXPIRES_IN=1d
JWT_REFRESH_SECRET=your-refresh-secret-key
JWT_REFRESH_EXPIRES_IN=7d

# ===================================
# Email Configuration (SendGrid)
# ===================================
SENDGRID_API_KEY=SG.xxxxxxxxxxxxxxxxxxxxxx
SENDGRID_FROM_EMAIL=noreply@example.com
SENDGRID_FROM_NAME=My App

# ===================================
# Payment Gateway (Stripe)
# ===================================
STRIPE_SECRET_KEY=sk_test_xxxxxxxxxxxxxxxx
STRIPE_PUBLIC_KEY=pk_test_xxxxxxxxxxxxxxxx
STRIPE_WEBHOOK_SECRET=whsec_xxxxxxxxxxxxxx
STRIPE_CURRENCY=USD

# ===================================
# AWS Configuration
# ===================================
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=AKIAxxxxxxxxxxxxxx
AWS_SECRET_ACCESS_KEY=xxxxxxxxxxxxxxxxxxxxxxxxx
AWS_S3_BUCKET=my-app-uploads

# ===================================
# CORS Configuration
# ===================================
CORS_ENABLED=true
CORS_ORIGINS=http://localhost:3000,http://localhost:4200

# ===================================
# Rate Limiting
# ===================================
RATE_LIMIT_TTL=60
RATE_LIMIT_MAX=100

# ===================================
# Logging
# ===================================
LOG_LEVEL=debug
LOG_FILE_PATH=./logs/app.log
LOG_MAX_SIZE=10m
LOG_MAX_FILES=7

# ===================================
# Feature Flags
# ===================================
ENABLE_CACHE=true
ENABLE_SWAGGER=true
ENABLE_COMPRESSION=true
DEBUG_MODE=true
```

**Variable Expansion (Requires `expandVariables: true`):**

```bash
# .env
BASE_URL=https://api.example.com
API_VERSION=v1

# Variable expansion
API_ENDPOINT=${BASE_URL}/${API_VERSION}
# Results in: https://api.example.com/v1

DATABASE_URL=postgres://${DB_USER}:${DB_PASS}@${DB_HOST}:${DB_PORT}/${DB_NAME}
```

```typescript
// Enable in ConfigModule
ConfigModule.forRoot({
  expandVariables: true,  // Enable variable expansion
});
```

**Best Practices:**

```bash
# ✅ GOOD PRACTICES

# 1. Use descriptive, SCREAMING_SNAKE_CASE names
DATABASE_HOST=localhost
JWT_SECRET=secret

# 2. Group related variables with comments
# Database Configuration
DATABASE_HOST=localhost
DATABASE_PORT=5432

# 3. Provide default-friendly values for development
PORT=3000
DEBUG_MODE=true

# 4. Use consistent naming conventions
DATABASE_HOST=localhost      # Prefix with feature
DATABASE_PORT=5432
DATABASE_USERNAME=admin

# 5. Document required vs optional
JWT_SECRET=secret           # Required in production
LOG_LEVEL=info             # Optional, defaults to 'info'

# ❌ BAD PRACTICES

# 1. Avoid inconsistent naming
dbHost=localhost           # ❌ camelCase
database_port=5432         # ❌ snake_case
DATABASEUSERNAME=admin     # ❌ No separators

# 2. Don't commit sensitive data
JWT_SECRET=production-secret-key  # ❌ Never commit real secrets

# 3. Avoid unclear names
KEY=abc123                 # ❌ What key?
VAL=true                   # ❌ What value?
```

**Interview Tip**: Define environment variables in `.env` files using **`KEY=value`** syntax (one per line). **No spaces** around equals sign. **All values stored as strings** - convert to numbers/booleans in config. Use **quotes** for values with spaces or special characters. Support **multiple files** (`.env`, `.env.development`, `.env.production`). Use **comments** with `#`. Enable **variable expansion** with `expandVariables: true`. **Best practice**: SCREAMING_SNAKE_CASE naming, group related variables, document required vs optional.

</details>

### 15. How do you access `process.env` in NestJS?

<details>
<summary>Answer</summary>

**`process.env`** is a Node.js global object containing environment variables. In NestJS, you can access it **directly** or through **ConfigService** (preferred).

**Direct Access (Not Recommended):**

```typescript
@Injectable()
export class AppService {
  getPort(): number {
    // Direct access to process.env
    return parseInt(process.env.PORT, 10) || 3000;
  }

  getDatabaseConfig() {
    return {
      host: process.env.DATABASE_HOST,
      port: parseInt(process.env.DATABASE_PORT, 10),
      username: process.env.DATABASE_USERNAME,
      password: process.env.DATABASE_PASSWORD,
    };
  }
}
```

**Why Direct Access Has Issues:**

```typescript
// ❌ Problems with direct process.env access

// 1. No type safety
const port = process.env.PORT;  // type: string | undefined
const parsed = parseInt(port, 10);  // NaN if PORT is undefined

// 2. No default values (must handle manually)
const timeout = parseInt(process.env.TIMEOUT, 10) || 5000;

// 3. No validation
const invalidPort = process.env.PORT;  // Could be "abc" - no validation

// 4. Hard to test
class MyService {
  getPort() {
    return process.env.PORT;  // How do you mock this?
  }
}

// 5. Scattered throughout codebase
// Different files access process.env directly - hard to track
```

**Recommended: Use ConfigService:**

```typescript
@Injectable()
export class AppService {
  constructor(private configService: ConfigService) {}

  getPort(): number {
    // ✅ With type safety and default value
    return this.configService.get<number>('PORT', 3000);
  }

  getDatabaseConfig() {
    return {
      host: this.configService.get<string>('DATABASE_HOST', 'localhost'),
      port: this.configService.get<number>('DATABASE_PORT', 5432),
      username: this.configService.getOrThrow<string>('DATABASE_USERNAME'),
      password: this.configService.getOrThrow<string>('DATABASE_PASSWORD'),
    };
  }
}
```

**ConfigService vs process.env Comparison:**

| Feature | `process.env` | `ConfigService` |
|---------|--------------|-----------------|
| **Type Safety** | ❌ No (string \| undefined) | ✅ Yes (with generics) |
| **Default Values** | ❌ Manual `\|\|` operator | ✅ Built-in `get(key, default)` |
| **Validation** | ❌ None | ✅ Joi/class-validator |
| **Required Values** | ❌ Manual checks | ✅ `getOrThrow()` |
| **Testability** | ❌ Hard to mock | ✅ Easy to mock |
| **Centralized** | ❌ Scattered access | ✅ Single source of truth |
| **Nested Config** | ❌ Flat keys only | ✅ Dot notation support |
| **Custom Loaders** | ❌ Not supported | ✅ `load` option |
| **Caching** | ❌ No built-in caching | ✅ Can enable caching |

**When Direct `process.env` Access is Acceptable:**

```typescript
// 1. In custom configuration files (before ConfigService)
// config/configuration.ts
export default () => ({
  port: parseInt(process.env.PORT, 10) || 3000,  // ✅ OK here
  database: {
    host: process.env.DATABASE_HOST || 'localhost',  // ✅ OK here
  },
});

// 2. In main.ts (before app is created)
// src/main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  const port = process.env.PORT || 3000;  // ✅ OK - ConfigService not available yet
  await app.listen(port);
}

// 3. Module configuration (forRootAsync with ConfigService is better)
@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: process.env.DATABASE_HOST || 'localhost',  // ⚠️ Works, but ConfigService is better
    }),
  ],
})
```

**Accessing `process.env` in main.ts:**

```typescript
// src/main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { ConfigService } from '@nestjs/config';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Option 1: Direct process.env (before ConfigService)
  const portFromEnv = parseInt(process.env.PORT, 10) || 3000;

  // Option 2: Get ConfigService from app (preferred)
  const configService = app.get(ConfigService);
  const port = configService.get<number>('PORT', 3000);

  await app.listen(port);
  console.log(`Application running on port ${port}`);
}
bootstrap();
```

**How ConfigService Loads from `process.env`:**

```typescript
// Behind the scenes, ConfigService:
// 1. Reads .env file using dotenv
// 2. Merges with existing process.env
// 3. Makes variables available via get()

// .env file
PORT=4000
DEBUG_MODE=true

// When ConfigModule loads:
require('dotenv').config();  // Loads .env into process.env

// Now both work:
console.log(process.env.PORT);  // '4000'
console.log(configService.get('PORT'));  // '4000'
```

**Priority Order (Highest to Lowest):**

```bash
# 1. System environment variables (highest priority)
export PORT=5000

# 2. .env file
PORT=4000

# 3. Default value in ConfigService
configService.get('PORT', 3000)
```

```typescript
// If system env PORT=5000 and .env has PORT=4000:
const port = configService.get('PORT');  // Returns '5000' (system env wins)

// If neither set:
const port = configService.get('PORT', 3000);  // Returns 3000 (default)
```

**Best Practices:**

```typescript
// ✅ GOOD: Use ConfigService in services/controllers
@Injectable()
export class UserService {
  constructor(private configService: ConfigService) {}

  getApiKey(): string {
    return this.configService.getOrThrow<string>('API_KEY');
  }
}

// ✅ GOOD: Use process.env in config files
// config/database.config.ts
export default () => ({
  host: process.env.DATABASE_HOST || 'localhost',
});

// ✅ GOOD: Use ConfigService in main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  const configService = app.get(ConfigService);
  const port = configService.get('PORT', 3000);
  await app.listen(port);
}

// ❌ BAD: Direct process.env in services
@Injectable()
export class BadService {
  getApiKey(): string {
    return process.env.API_KEY;  // ❌ No validation, hard to test
  }
}
```

**Interview Tip**: `process.env` is a **Node.js global** containing environment variables. **Prefer ConfigService** over direct `process.env` access for: type safety, default values, validation, testability, centralized config. Direct `process.env` acceptable in: **custom config files**, **main.ts before app creation**, **module imports** (though `forRootAsync` with ConfigService is better). **Priority**: system env vars → .env file → default values. **Best practice**: always use ConfigService in services/controllers.

</details>

### 16. Should you commit `.env` files to version control?

<details>
<summary>Answer</summary>

**No, you should NEVER commit `.env` files to version control.** They contain sensitive data like passwords, API keys, and secrets that should not be exposed.

**Why NOT to Commit `.env` Files:**

```bash
# ❌ NEVER COMMIT .env FILES

# .env contains sensitive data:
DATABASE_PASSWORD=mySecretPassword123!
JWT_SECRET=super-secret-key-for-production
STRIPE_SECRET_KEY=sk_live_xxxxxxxxxxxxxxxx
AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
SENDGRID_API_KEY=SG.xxxxxxxxxxxxxxxxxxxx

# If committed:
# ❌ Passwords exposed in Git history
# ❌ API keys accessible to anyone with repo access
# ❌ Security vulnerabilities
# ❌ Can't easily rotate secrets (history remains)
# ❌ Different environments need different values
```

**Proper `.gitignore` Configuration:**

```bash
# .gitignore

# Environment files
.env
.env.local
.env.*.local
.env.development.local
.env.test.local
.env.production.local

# But DO commit these:
# .env.example  ✅ (template without real values)
# .env.development ✅ (if no sensitive data)
# .env.test ✅ (for test environment with fake data)
```

**What to Commit Instead:**

**1. `.env.example` (Template File):**

```bash
# .env.example
# Copy this to .env and fill in your values

# Application
NODE_ENV=development
PORT=3000

# Database
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_USERNAME=your_username_here
DATABASE_PASSWORD=your_password_here
DATABASE_NAME=myapp_db

# JWT
JWT_SECRET=your_jwt_secret_here
JWT_EXPIRES_IN=1d

# External APIs
STRIPE_SECRET_KEY=your_stripe_key_here
SENDGRID_API_KEY=your_sendgrid_key_here
```

**2. Development Configuration (Optional):**

```bash
# .env.development
# Safe to commit if no real credentials

NODE_ENV=development
PORT=3000
DATABASE_HOST=localhost
DEBUG_MODE=true
LOG_LEVEL=debug

# Don't include real secrets even in dev config
# JWT_SECRET=  # Leave empty - developers set their own
```

**3. Test Configuration:**

```bash
# .env.test
# Safe to commit - uses fake test data

NODE_ENV=test
PORT=3001
DATABASE_HOST=localhost
DATABASE_NAME=myapp_test
DATABASE_USERNAME=test_user
DATABASE_PASSWORD=test_password  # Not a real password
JWT_SECRET=test-secret-key-not-for-production
```

**Project Setup for New Developers:**

```bash
# README.md

## Setup Instructions

1. Clone the repository
   git clone https://github.com/yourorg/yourproject.git

2. Install dependencies
   npm install

3. Copy environment template
   cp .env.example .env

4. Fill in your local values in .env
   # Edit .env with your database credentials, API keys, etc.

5. Run the application
   npm run start:dev
```

**How Production Secrets Are Managed:**

```bash
# Production environments should NOT use .env files

# Instead, use:

# 1. Environment Variables (Server/Container)
export DATABASE_PASSWORD=prod-secret-password
export JWT_SECRET=prod-jwt-secret

# 2. Cloud Provider Secrets (AWS, Azure, GCP)
# - AWS Secrets Manager
# - AWS Parameter Store
# - Azure Key Vault
# - Google Secret Manager

# 3. Container Orchestration (Docker/Kubernetes)
# - Kubernetes Secrets
# - Docker Swarm Secrets

# 4. CI/CD Platform Secrets
# - GitHub Secrets
# - GitLab CI/CD Variables
# - CircleCI Environment Variables
```

**Real-World Disaster Scenario:**

```bash
# What happens when you commit .env:

# 1. Developer accidentally commits .env
git add .
git commit -m "Add configuration"
git push origin main

# 2. Secrets now in Git history FOREVER
# Even if you delete the file later:
git rm .env
git commit -m "Remove .env"
# ❌ Still in history! Anyone can access with:
git log --all --full-history -- .env
git show <commit-hash>:.env

# 3. Must rotate ALL secrets
# - Change all passwords
# - Regenerate all API keys
# - Update production systems
# - Invalidate old credentials

# 4. Potential security breach
# - Attackers scan GitHub for exposed secrets
# - Automated bots search for API keys
# - Can lead to data breach, financial loss
```

**How to Remove Accidentally Committed Secrets:**

```bash
# ⚠️ If you accidentally committed .env:

# 1. IMMEDIATELY rotate all secrets
# - Change passwords
# - Regenerate API keys
# - Invalidate tokens

# 2. Remove from Git history (complex)
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch .env" \
  --prune-empty --tag-name-filter cat -- --all

# 3. Force push (dangerous)
git push origin --force --all

# 4. Notify team to re-clone
# All team members must re-clone the repository
```

**Environment-Specific Files:**

```bash
# What you CAN commit (with caution):

# ✅ .env.example (no real secrets)
NODE_ENV=development
DATABASE_HOST=localhost
API_KEY=your_key_here

# ✅ .env.test (fake test data only)
NODE_ENV=test
DATABASE_PASSWORD=test123  # Not a real password
JWT_SECRET=test-secret

# ❌ .env (real secrets)
DATABASE_PASSWORD=RealP@ssw0rd123!
JWT_SECRET=real-production-secret

# ❌ .env.production (real secrets)
DATABASE_PASSWORD=ProdP@ssw0rd!
STRIPE_SECRET_KEY=sk_live_xxxxx

# ❌ .env.local (personal secrets)
MY_AWS_KEY=AKIAIOSFODNN7EXAMPLE
```

**Security Best Practices:**

```typescript
// 1. Validate required secrets at startup
@Module({
  imports: [
    ConfigModule.forRoot({
      validationSchema: Joi.object({
        DATABASE_PASSWORD: Joi.string().required(),
        JWT_SECRET: Joi.string().min(32).required(),
        API_KEY: Joi.string().required(),
      }),
    }),
  ],
})
export class AppModule {}

// 2. Never log secrets
@Injectable()
export class ConfigService {
  logConfig() {
    console.log('Database Host:', this.config.get('DATABASE_HOST'));  // ✅ OK
    console.log('Database Password:', '***');  // ✅ Redacted
    console.log('Database Password:', this.config.get('DATABASE_PASSWORD'));  // ❌ NEVER!
  }
}

// 3. Use different secrets per environment
const secret = this.config.getOrThrow(
  process.env.NODE_ENV === 'production' 
    ? 'JWT_SECRET_PROD' 
    : 'JWT_SECRET_DEV'
);
```

**Interview Tip**: **NEVER commit `.env` files** to version control - they contain **sensitive secrets** (passwords, API keys, tokens). Add `.env` to `.gitignore`. Instead commit: **`.env.example`** (template with placeholder values), **`.env.test`** (fake test data), optionally **`.env.development`** (if no real secrets). In production: use **environment variables**, **cloud secrets management** (AWS Secrets Manager, Azure Key Vault), or **CI/CD secrets**. If accidentally committed: **immediately rotate all secrets** and remove from Git history. **Best practice**: template files for documentation, real secrets only in environment.

</details>

### 17. What is `.env.example` file for?

<details>
<summary>Answer</summary>

**`.env.example`** is a **template file** that shows what environment variables your application needs, **without containing real secrets**. It serves as documentation and a starting point for new developers.

**Purpose of `.env.example`:**

```bash
# .env.example

# This file is:
# ✅ Committed to version control (Git)
# ✅ Contains variable names and structure
# ✅ Shows required vs optional variables
# ✅ Includes comments explaining each variable
# ❌ Does NOT contain real secrets or passwords
# ❌ Does NOT contain actual API keys

# Developers copy this to .env and fill in real values
```

**Example `.env.example` File:**

```bash
# .env.example
# Copy this file to .env and fill in your values:
# cp .env.example .env

# ============================================
# Application Configuration
# ============================================
NODE_ENV=development
PORT=3000
APP_NAME=My NestJS Application
APP_URL=http://localhost:3000

# ============================================
# Database Configuration
# ============================================
# PostgreSQL connection settings
DATABASE_TYPE=postgres
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_USERNAME=your_database_username
DATABASE_PASSWORD=your_database_password
DATABASE_NAME=your_database_name
DATABASE_SYNC=false
DATABASE_LOGGING=true

# ============================================
# JWT Configuration
# ============================================
# Generate a secure random string for production
# Recommended: 32+ characters
JWT_SECRET=your_jwt_secret_here
JWT_EXPIRES_IN=1d
JWT_REFRESH_EXPIRES_IN=7d

# ============================================
# Redis Configuration (Optional)
# ============================================
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=
REDIS_TTL=300

# ============================================
# Email Service (SendGrid)
# ============================================
# Get your API key from: https://app.sendgrid.com/settings/api_keys
SENDGRID_API_KEY=your_sendgrid_api_key
SENDGRID_FROM_EMAIL=noreply@yourdomain.com
SENDGRID_FROM_NAME=Your App Name

# ============================================
# Payment Gateway (Stripe)
# ============================================
# Test keys from: https://dashboard.stripe.com/test/apikeys
STRIPE_SECRET_KEY=sk_test_your_stripe_secret_key
STRIPE_PUBLIC_KEY=pk_test_your_stripe_public_key
STRIPE_WEBHOOK_SECRET=whsec_your_webhook_secret

# ============================================
# AWS Configuration (Optional)
# ============================================
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=your_aws_access_key
AWS_SECRET_ACCESS_KEY=your_aws_secret_key
AWS_S3_BUCKET=your-s3-bucket-name

# ============================================
# Feature Flags
# ============================================
ENABLE_CACHE=true
ENABLE_SWAGGER=true
DEBUG_MODE=false
```

**vs Actual `.env` File:**

```bash
# .env (NOT committed - contains real secrets)

NODE_ENV=development
PORT=3000
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_USERNAME=admin
DATABASE_PASSWORD=RealSecretPassword123!  # ← Real password
DATABASE_NAME=myapp_development

JWT_SECRET=a8f5f167f44f4964e6c998dee827110c  # ← Real secret
STRIPE_SECRET_KEY=sk_test_51Hj...  # ← Real API key
SENDGRID_API_KEY=SG.abc123xyz...  # ← Real API key
```

**How Developers Use It:**

```bash
# Setup workflow for new developer:

# 1. Clone repository
git clone https://github.com/company/project.git
cd project

# 2. Install dependencies
npm install

# 3. Copy .env.example to .env
cp .env.example .env

# 4. Edit .env with real values
# Open .env in editor and fill in:
# - Database credentials
# - API keys
# - Secrets
nano .env  # or code .env

# 5. Run application
npm run start:dev
```

**Complete Example with Documentation:**

```bash
# .env.example

# ============================================
# REQUIRED VARIABLES
# These must be set for the application to run
# ============================================

# Database connection (REQUIRED)
# Create a PostgreSQL database and user, then fill these in
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_USERNAME=your_username
DATABASE_PASSWORD=your_password
DATABASE_NAME=your_database_name

# JWT Secret (REQUIRED)
# Generate with: node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
JWT_SECRET=your_jwt_secret_minimum_32_characters

# ============================================
# OPTIONAL VARIABLES
# These have defaults but can be customized
# ============================================

# Server configuration (Optional - defaults to 3000)
PORT=3000
HOST=localhost

# Environment (Optional - defaults to 'development')
# Options: development, production, test
NODE_ENV=development

# Logging (Optional - defaults to 'info')
# Options: error, warn, info, debug
LOG_LEVEL=debug

# ============================================
# THIRD-PARTY SERVICES
# Only required if using these features
# ============================================

# Stripe (Optional - only for payment features)
# Get test keys from: https://dashboard.stripe.com/test/apikeys
# STRIPE_SECRET_KEY=sk_test_your_key_here
# STRIPE_PUBLIC_KEY=pk_test_your_key_here

# SendGrid (Optional - only for email features)
# Sign up at: https://signup.sendgrid.com/
# SENDGRID_API_KEY=your_api_key_here
# SENDGRID_FROM_EMAIL=noreply@yourdomain.com

# AWS S3 (Optional - only for file uploads)
# AWS_REGION=us-east-1
# AWS_ACCESS_KEY_ID=your_access_key
# AWS_SECRET_ACCESS_KEY=your_secret_key
# AWS_S3_BUCKET=your-bucket-name

# ============================================
# DEVELOPMENT TOOLS
# These are for development convenience
# ============================================

# Enable Swagger API documentation
ENABLE_SWAGGER=true

# Enable debug mode (verbose logging)
DEBUG_MODE=true

# Enable hot reload
WATCH_MODE=true
```

**Different `.env.example` for Different Environments:**

```bash
# .env.example (base template)
PORT=3000
DATABASE_HOST=localhost

# .env.development.example
NODE_ENV=development
DEBUG_MODE=true
DATABASE_LOGGING=true
ENABLE_SWAGGER=true

# .env.production.example
NODE_ENV=production
DEBUG_MODE=false
DATABASE_LOGGING=false
ENABLE_SWAGGER=false
# Note: Use system environment variables for secrets in production
```

**README.md Integration:**

```markdown
# My NestJS Application

## Environment Setup

### 1. Copy Environment Template
```bash
cp .env.example .env
```

### 2. Configure Required Variables

Edit `.env` and set these **required** values:

- `DATABASE_PASSWORD`: Your PostgreSQL password
- `JWT_SECRET`: Generate with `openssl rand -hex 32`
- `API_KEY`: Your API key from the service provider

### 3. Optional Configuration

Customize these **optional** variables in `.env`:

- `PORT`: Server port (default: 3000)
- `LOG_LEVEL`: Logging level (default: info)
- `ENABLE_CACHE`: Enable Redis caching (default: false)

### 4. Environment-Specific Files

- **Development**: Uses `.env` file
- **Production**: Uses system environment variables (don't use `.env` in prod)
- **Testing**: Uses `.env.test` file

## Getting Started

```bash
npm install
cp .env.example .env
# Edit .env with your values
npm run start:dev
```
```

**Validation Against Template:**

```typescript
// Optional: Script to check if .env has all required variables from .env.example

// scripts/check-env.ts
import * as fs from 'fs';
import * as dotenv from 'dotenv';

const example = dotenv.parse(fs.readFileSync('.env.example'));
const actual = dotenv.parse(fs.readFileSync('.env'));

const missing = Object.keys(example).filter(key => !(key in actual));

if (missing.length > 0) {
  console.error('Missing environment variables:', missing);
  process.exit(1);
}

console.log('All required environment variables are set!');
```

**Best Practices:**

```bash
# ✅ GOOD: Descriptive comments
# Database password (minimum 8 characters)
DATABASE_PASSWORD=your_password_here

# JWT secret key (generate with: openssl rand -hex 32)
JWT_SECRET=your_jwt_secret_here

# ✅ GOOD: Show example format
CORS_ORIGINS=http://localhost:3000,https://app.example.com

# ✅ GOOD: Mark optional vs required
# (REQUIRED) Database host
DATABASE_HOST=localhost

# (Optional) Redis cache TTL in seconds (default: 300)
REDIS_TTL=300

# ✅ GOOD: Include helpful links
# Stripe API keys from: https://dashboard.stripe.com/test/apikeys
STRIPE_SECRET_KEY=sk_test_your_key

# ❌ BAD: Real values
DATABASE_PASSWORD=ActualPassword123!  # ❌ Never put real values

# ❌ BAD: No documentation
VAR1=value1  # ❌ What is this?
KEY=xyz  # ❌ What key?

# ❌ BAD: Inconsistent format
some_var=value  # snake_case
SomeOther=value  # PascalCase
```

**Interview Tip**: **`.env.example`** is a **template file** showing required environment variables **without real secrets**. It's **committed to Git** (unlike `.env`). Purpose: **documentation** for new developers, **structure reference**, shows **required vs optional** variables. Developers: **copy to `.env`** and **fill in real values**. Include: **variable names**, **comments**, **example formats**, **helpful links**, mark **required vs optional**. **Best practice**: comprehensive comments, clear structure, generation commands for secrets, integration with README.

</details>

## Validation

### 18. How do you validate environment variables using Joi?

<details>
<summary>Answer</summary>

**Joi** is a powerful schema validation library that NestJS ConfigModule uses to **validate environment variables at application startup**. If validation fails, the app won't start.

**Installation:**

```bash
npm install joi
# or
yarn add joi
```

**Basic Joi Validation:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import * as Joi from 'joi';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      validationSchema: Joi.object({
        // Define validation rules for each environment variable
        NODE_ENV: Joi.string()
          .valid('development', 'production', 'test')
          .default('development'),
        
        PORT: Joi.number()
          .default(3000),
        
        DATABASE_HOST: Joi.string()
          .required(),
        
        DATABASE_PORT: Joi.number()
          .default(5432),
        
        DATABASE_PASSWORD: Joi.string()
          .required(),
      }),
    }),
  ],
})
export class AppModule {}
```

**Joi Validation Methods:**

```typescript
import * as Joi from 'joi';

const validationSchema = Joi.object({
  // 1. String validation
  DATABASE_HOST: Joi.string()
    .required()                    // Must be present
    .hostname()                    // Must be valid hostname
    .default('localhost'),         // Default if not provided
  
  // 2. Number validation
  PORT: Joi.number()
    .required()
    .port()                        // Must be valid port (1-65535)
    .default(3000),
  
  DATABASE_PORT: Joi.number()
    .integer()                     // Must be integer
    .min(1)                        // Minimum value
    .max(65535)                    // Maximum value
    .default(5432),
  
  TIMEOUT: Joi.number()
    .positive()                    // Must be positive
    .default(5000),
  
  // 3. String with options
  NODE_ENV: Joi.string()
    .valid('development', 'production', 'test', 'staging')
    .default('development'),
  
  LOG_LEVEL: Joi.string()
    .valid('error', 'warn', 'info', 'debug')
    .default('info'),
  
  // 4. Boolean validation
  DEBUG_MODE: Joi.boolean()
    .default(false),
  
  ENABLE_CACHE: Joi.boolean()
    .truthy('1', 'true')          // '1' or 'true' = true
    .falsy('0', 'false')          // '0' or 'false' = false
    .default(true),
  
  // 5. String patterns (regex)
  DATABASE_PASSWORD: Joi.string()
    .required()
    .min(8)                        // Minimum length
    .pattern(/^[a-zA-Z0-9!@#$%^&*]+$/)  // Allowed characters
    .messages({
      'string.pattern.base': 'Password must contain only alphanumeric and special characters',
    }),
  
  // 6. Email validation
  ADMIN_EMAIL: Joi.string()
    .email()
    .required(),
  
  // 7. URL validation
  API_URL: Joi.string()
    .uri()
    .required(),
  
  CALLBACK_URL: Joi.string()
    .uri({ scheme: ['http', 'https'] })  // Only http/https
    .required(),
  
  // 8. JWT secret (hex string)
  JWT_SECRET: Joi.string()
    .required()
    .hex()                         // Must be hexadecimal
    .min(32)                       // At least 32 characters
    .messages({
      'string.hex': 'JWT_SECRET must be a hexadecimal string',
      'string.min': 'JWT_SECRET must be at least 32 characters',
    }),
  
  // 9. Conditional validation
  REDIS_PASSWORD: Joi.string()
    .when('REDIS_ENABLED', {
      is: Joi.boolean().truthy('1', 'true'),
      then: Joi.required(),        // Required if REDIS_ENABLED is true
      otherwise: Joi.optional(),
    }),
  
  // 10. Array validation (comma-separated)
  CORS_ORIGINS: Joi.string()
    .pattern(/^https?:\/\//)       // Must start with http:// or https://
    .default('http://localhost:3000'),
});
```

**Complete Real-World Example:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import * as Joi from 'joi';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: '.env',
      validationSchema: Joi.object({
        // Application
        NODE_ENV: Joi.string()
          .valid('development', 'production', 'test', 'staging')
          .default('development'),
        
        PORT: Joi.number()
          .port()
          .default(3000),
        
        // Database
        DATABASE_TYPE: Joi.string()
          .valid('postgres', 'mysql', 'mariadb', 'sqlite')
          .default('postgres'),
        
        DATABASE_HOST: Joi.string()
          .hostname()
          .required(),
        
        DATABASE_PORT: Joi.number()
          .port()
          .default(5432),
        
        DATABASE_USERNAME: Joi.string()
          .required(),
        
        DATABASE_PASSWORD: Joi.string()
          .required()
          .min(8),
        
        DATABASE_NAME: Joi.string()
          .required(),
        
        DATABASE_SYNC: Joi.boolean()
          .default(false),
        
        DATABASE_LOGGING: Joi.boolean()
          .default(false),
        
        // JWT
        JWT_SECRET: Joi.string()
          .required()
          .min(32),
        
        JWT_EXPIRES_IN: Joi.string()
          .pattern(/^\d+[smhd]$/)  // e.g., '1d', '24h', '60m'
          .default('1d'),
        
        // Redis (optional)
        REDIS_HOST: Joi.string()
          .hostname()
          .optional(),
        
        REDIS_PORT: Joi.number()
          .port()
          .optional(),
        
        // External APIs
        STRIPE_SECRET_KEY: Joi.string()
          .pattern(/^sk_(test|live)_/)
          .when('NODE_ENV', {
            is: 'production',
            then: Joi.required(),
            otherwise: Joi.optional(),
          }),
        
        SENDGRID_API_KEY: Joi.string()
          .pattern(/^SG\./)
          .optional(),
        
        // Security
        CORS_ORIGINS: Joi.string()
          .default('http://localhost:3000'),
        
        RATE_LIMIT_TTL: Joi.number()
          .positive()
          .default(60),
        
        RATE_LIMIT_MAX: Joi.number()
          .positive()
          .default(100),
      }),
      validationOptions: {
        allowUnknown: true,   // Allow env vars not in schema
        abortEarly: false,    // Show all validation errors
      },
    }),
  ],
})
export class AppModule {}
```

**Custom Error Messages:**

```typescript
const validationSchema = Joi.object({
  DATABASE_PASSWORD: Joi.string()
    .required()
    .min(8)
    .pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]+$/)
    .messages({
      'any.required': 'DATABASE_PASSWORD is required',
      'string.min': 'DATABASE_PASSWORD must be at least 8 characters',
      'string.pattern.base': 'DATABASE_PASSWORD must contain uppercase, lowercase, number, and special character',
    }),
  
  PORT: Joi.number()
    .port()
    .messages({
      'number.base': 'PORT must be a number',
      'number.port': 'PORT must be a valid port number (1-65535)',
    }),
});
```

**Validation Options:**

```typescript
ConfigModule.forRoot({
  validationSchema: Joi.object({...}),
  validationOptions: {
    // Allow environment variables not defined in schema
    allowUnknown: true,     // Default: true
    
    // Stop validation on first error
    abortEarly: false,      // Default: true (show all errors)
    
    // Cache validation results
    cache: true,
    
    // Context to pass to schema
    context: {
      env: process.env.NODE_ENV,
    },
    
    // Debug mode
    debug: false,
    
    // Convert types automatically
    convert: true,          // '3000' → 3000, 'true' → true
    
    // Presence mode
    presence: 'optional',   // 'optional' | 'required' | 'forbidden'
  },
});
```

**What Happens During Validation:**

```typescript
// Validation runs at application startup

// 1. ConfigModule loads .env file
// 2. Joi validates against schema
// 3. If valid → app starts
// 4. If invalid → app crashes with error message

// Example error:
/*
Error: Config validation error:
"DATABASE_PASSWORD" is required
"PORT" must be a number
"NODE_ENV" must be one of [development, production, test]
*/
```

**Extracting Schema to Separate File:**

```typescript
// config/validation.schema.ts
import * as Joi from 'joi';

export const validationSchema = Joi.object({
  NODE_ENV: Joi.string()
    .valid('development', 'production', 'test')
    .default('development'),
  
  PORT: Joi.number()
    .port()
    .default(3000),
  
  DATABASE_HOST: Joi.string().required(),
  DATABASE_PORT: Joi.number().default(5432),
  DATABASE_USERNAME: Joi.string().required(),
  DATABASE_PASSWORD: Joi.string().required().min(8),
  DATABASE_NAME: Joi.string().required(),
  
  JWT_SECRET: Joi.string().required().min(32),
  JWT_EXPIRES_IN: Joi.string().default('1d'),
});

// app.module.ts
import { validationSchema } from './config/validation.schema';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      validationSchema,
    }),
  ],
})
export class AppModule {}
```

**Interview Tip**: Use **Joi** to **validate environment variables** at application startup. Install with `npm install joi`. Define **validationSchema** in `ConfigModule.forRoot()` using `Joi.object()`. Common validations: **string** (`.string()`), **number** (`.number()`), **boolean** (`.boolean()`), **required** (`.required()`), **default** (`.default()`), **valid options** (`.valid()`), **patterns** (`.pattern()`). Set **validationOptions** for `allowUnknown`, `abortEarly`. App **won't start** if validation fails. **Best practice**: validate all required vars, use specific validators (`.port()`, `.email()`, `.uri()`), custom error messages.

</details>

### 19. What is `validationSchema` in ConfigModule?

<details>
<summary>Answer</summary>

**`validationSchema`** is a **ConfigModule option** that accepts a **Joi schema** to validate environment variables when the application starts. It ensures all required configuration is present and correctly formatted.

**Basic Usage:**

```typescript
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import * as Joi from 'joi';

@Module({
  imports: [
    ConfigModule.forRoot({
      validationSchema: Joi.object({
        // Joi validation schema here
        PORT: Joi.number().default(3000),
        DATABASE_HOST: Joi.string().required(),
      }),
    }),
  ],
})
export class AppModule {}
```

**Purpose of validationSchema:**

```typescript
// validationSchema ensures:

// 1. Required variables are present
validationSchema: Joi.object({
  DATABASE_PASSWORD: Joi.string().required(),  // Must exist
  JWT_SECRET: Joi.string().required(),
})

// 2. Variables have correct types
validationSchema: Joi.object({
  PORT: Joi.number(),              // Must be parseable as number
  DEBUG_MODE: Joi.boolean(),       // Must be 'true'/'false'
})

// 3. Variables have valid values
validationSchema: Joi.object({
  NODE_ENV: Joi.string().valid('development', 'production', 'test'),
  LOG_LEVEL: Joi.string().valid('error', 'warn', 'info', 'debug'),
})

// 4. Variables match patterns
validationSchema: Joi.object({
  EMAIL: Joi.string().email(),     // Must be valid email
  URL: Joi.string().uri(),         // Must be valid URI
})

// 5. Default values are applied
validationSchema: Joi.object({
  PORT: Joi.number().default(3000),  // Uses 3000 if not set
})
```

**Complete Example:**

```typescript
// app.module.ts
@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: '.env',
      
      // Validation schema
      validationSchema: Joi.object({
        // Application settings
        NODE_ENV: Joi.string()
          .valid('development', 'production', 'test')
          .default('development'),
        
        PORT: Joi.number()
          .port()
          .default(3000),
        
        // Database configuration
        DATABASE_HOST: Joi.string()
          .required(),
        
        DATABASE_PORT: Joi.number()
          .default(5432),
        
        DATABASE_USERNAME: Joi.string()
          .required(),
        
        DATABASE_PASSWORD: Joi.string()
          .required()
          .min(8),
        
        // Security
        JWT_SECRET: Joi.string()
          .required()
          .min(32),
      }),
      
      // Validation options
      validationOptions: {
        allowUnknown: true,   // Allow variables not in schema
        abortEarly: false,    // Show all errors, not just first
      },
    }),
  ],
})
export class AppModule {}
```

**How validationSchema Works:**

```typescript
// Execution flow:

// 1. Application starts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  // ↑ ConfigModule.forRoot() executes here
}

// 2. ConfigModule loads environment variables
// - Reads .env file
// - Merges with process.env

// 3. Joi validates against validationSchema
// - Checks required fields
// - Validates types
// - Applies defaults
// - Checks patterns

// 4. If validation fails:
// - Throws error with details
// - Application exits
// - Shows which variables failed

// 5. If validation passes:
// - Continues application startup
// - ConfigService becomes available
```

**Validation Error Examples:**

```bash
# Missing required variable
Error: Config validation error: "DATABASE_PASSWORD" is required

# Invalid type
Error: Config validation error: "PORT" must be a number

# Invalid enum value
Error: Config validation error: "NODE_ENV" must be one of [development, production, test]

# Pattern mismatch
Error: Config validation error: "EMAIL" must be a valid email

# Multiple errors (with abortEarly: false)
Error: Config validation error:
"DATABASE_PASSWORD" is required
"PORT" must be a number
"NODE_ENV" must be one of [development, production, test]
```

**validationSchema vs validationOptions:**

```typescript
ConfigModule.forRoot({
  // Joi schema object
  validationSchema: Joi.object({
    PORT: Joi.number().required(),
    DATABASE_HOST: Joi.string().required(),
  }),
  
  // How validation should behave
  validationOptions: {
    allowUnknown: true,   // Allow extra env vars not in schema
    abortEarly: false,    // Show all errors at once
    cache: true,          // Cache validation result
    convert: true,        // Auto-convert types ('3000' → 3000)
    presence: 'optional', // Default presence mode
  },
})
```

**Common validationOptions:**

```typescript
validationOptions: {
  // 1. allowUnknown
  allowUnknown: true,    // ✅ Allow RANDOM_VAR not in schema
  allowUnknown: false,   // ❌ Error if RANDOM_VAR not in schema
  
  // 2. abortEarly
  abortEarly: true,      // Stop at first error
  abortEarly: false,     // Show all errors
  
  // 3. convert (type conversion)
  convert: true,         // '3000' → 3000, 'true' → true
  convert: false,        // Keep as strings
  
  // 4. presence
  presence: 'optional',  // All fields optional by default
  presence: 'required',  // All fields required by default
  presence: 'forbidden', // No fields allowed (rare)
}
```

**Without validationSchema:**

```typescript
// ❌ No validation - problems at runtime
@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      // No validationSchema
    }),
  ],
})

// Problems:
// - Missing DATABASE_PASSWORD? Runtime error when connecting
// - PORT='abc'? Runtime error when starting server
// - No JWT_SECRET? Security vulnerability
// - Wrong NODE_ENV? Wrong config loaded
```

**With validationSchema:**

```typescript
// ✅ Validation - fails fast at startup
@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      validationSchema: Joi.object({
        DATABASE_PASSWORD: Joi.string().required(),
        PORT: Joi.number().port(),
        JWT_SECRET: Joi.string().required().min(32),
        NODE_ENV: Joi.string().valid('development', 'production'),
      }),
    }),
  ],
})

// Benefits:
// - Missing DATABASE_PASSWORD? App won't start
// - PORT='abc'? App won't start
// - No JWT_SECRET? App won't start
// - Wrong NODE_ENV? App won't start
// - Clear error messages for developers
```

**Organizing validationSchema:**

```typescript
// Option 1: Inline (simple apps)
ConfigModule.forRoot({
  validationSchema: Joi.object({
    PORT: Joi.number().default(3000),
  }),
})

// Option 2: Separate file (recommended for larger apps)
// config/validation.schema.ts
export const validationSchema = Joi.object({
  PORT: Joi.number().default(3000),
  DATABASE_HOST: Joi.string().required(),
  // ... more validations
});

// app.module.ts
import { validationSchema } from './config/validation.schema';

ConfigModule.forRoot({
  validationSchema,
})

// Option 3: Modular schemas
// config/schemas/database.schema.ts
export const databaseSchema = {
  DATABASE_HOST: Joi.string().required(),
  DATABASE_PORT: Joi.number().default(5432),
};

// config/schemas/jwt.schema.ts
export const jwtSchema = {
  JWT_SECRET: Joi.string().required().min(32),
  JWT_EXPIRES_IN: Joi.string().default('1d'),
};

// config/validation.schema.ts
import { databaseSchema } from './schemas/database.schema';
import { jwtSchema } from './schemas/jwt.schema';

export const validationSchema = Joi.object({
  ...databaseSchema,
  ...jwtSchema,
});
```

**Interview Tip**: **`validationSchema`** is a **ConfigModule option** that accepts a **Joi schema** to validate environment variables at startup. Purpose: ensure **required vars exist**, **correct types**, **valid values**, **apply defaults**. If validation fails: app **won't start**, shows **error details**. Works with **`validationOptions`** to control behavior (`allowUnknown`, `abortEarly`, `convert`). **Best practice**: always use validationSchema, validate all required/critical config, show all errors with `abortEarly: false`, extract to separate file for large schemas.

</details>

### 20. How do you make environment variables required?

<details>
<summary>Answer</summary>

**Environment variables** can be made **required** using multiple approaches: **Joi validation**, **`getOrThrow()`** method, or **custom validation**.

**Method 1: Joi Validation (Recommended - Fails at Startup):**

```typescript
// app.module.ts
import * as Joi from 'joi';

@Module({
  imports: [
    ConfigModule.forRoot({
      validationSchema: Joi.object({
        // Required with .required()
        DATABASE_PASSWORD: Joi.string().required(),
        JWT_SECRET: Joi.string().required(),
        API_KEY: Joi.string().required(),
        
        // Optional (no .required())
        PORT: Joi.number().default(3000),
        DEBUG_MODE: Joi.boolean().default(false),
      }),
    }),
  ],
})
export class AppModule {}

// If DATABASE_PASSWORD is missing from .env:
// Error: Config validation error: "DATABASE_PASSWORD" is required
// App won't start ✅
```

**Method 2: ConfigService.getOrThrow() (Fails When Accessed):**

```typescript
@Injectable()
export class DatabaseService {
  constructor(private configService: ConfigService) {}

  connect() {
    // getOrThrow() throws if variable is missing
    const host = this.configService.getOrThrow<string>('DATABASE_HOST');
    const password = this.configService.getOrThrow<string>('DATABASE_PASSWORD');
    
    // If DATABASE_PASSWORD is missing:
    // Error: An error occurred. Please check your environment configuration.
    // Throws when method is called ✅
    
    return { host, password };
  }
}
```

**Method 3: Custom Validation (Manual Checks):**

```typescript
// Custom validation function
function validateRequiredEnvVars() {
  const required = ['DATABASE_PASSWORD', 'JWT_SECRET', 'API_KEY'];
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
  }
}

// Call at application startup
async function bootstrap() {
  validateRequiredEnvVars();  // Check before creating app
  
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
```

**Comparison of Methods:**

| Method | When It Fails | Error Message | Use Case |
|--------|--------------|---------------|----------|
| **Joi `.required()`** | App startup | Clear validation error | ✅ Recommended - validates all config upfront |
| **`getOrThrow()`** | When accessed | Generic error message | ⚠️ Fails later, not at startup |
| **Custom validation** | App startup | Custom error message | ⚠️ More code, less maintainable |

**Real-World Example - Production Config:**

```typescript
// All critical production variables as required
@Module({
  imports: [
    ConfigModule.forRoot({
      validationSchema: Joi.object({
        // Application (required)
        NODE_ENV: Joi.string()
          .valid('development', 'production', 'test')
          .required(),
        
        // Database (all required for production)
        DATABASE_HOST: Joi.string()
          .required(),
        
        DATABASE_PORT: Joi.number()
          .required(),
        
        DATABASE_USERNAME: Joi.string()
          .required(),
        
        DATABASE_PASSWORD: Joi.string()
          .required()
          .min(12)  // Strong password requirement
          .messages({
            'string.min': 'DATABASE_PASSWORD must be at least 12 characters for security',
          }),
        
        DATABASE_NAME: Joi.string()
          .required(),
        
        // Security (required)
        JWT_SECRET: Joi.string()
          .required()
          .min(32)
          .messages({
            'string.min': 'JWT_SECRET must be at least 32 characters',
          }),
        
        JWT_REFRESH_SECRET: Joi.string()
          .required()
          .min(32),
        
        // External APIs (required for core functionality)
        STRIPE_SECRET_KEY: Joi.string()
          .required()
          .pattern(/^sk_(test|live)_/),
        
        SENDGRID_API_KEY: Joi.string()
          .required()
          .pattern(/^SG\./),
        
        // Optional with defaults
        PORT: Joi.number()
          .default(3000),
        
        LOG_LEVEL: Joi.string()
          .default('info'),
      }),
    }),
  ],
})
export class AppModule {}
```

**Conditional Required (Based on Environment):**

```typescript
const validationSchema = Joi.object({
  NODE_ENV: Joi.string()
    .valid('development', 'production', 'test')
    .default('development'),
  
  // Required in production, optional in development
  STRIPE_SECRET_KEY: Joi.string()
    .when('NODE_ENV', {
      is: 'production',
      then: Joi.required()
        .pattern(/^sk_live_/),  // Must be live key in prod
      otherwise: Joi.optional()
        .pattern(/^sk_test_/),  // Can be test key in dev
    }),
  
  // Required if feature is enabled
  REDIS_PASSWORD: Joi.string()
    .when('ENABLE_REDIS', {
      is: Joi.boolean().truthy('1', 'true'),
      then: Joi.required(),
      otherwise: Joi.optional(),
    }),
  
  // Required only in production
  SSL_CERT_PATH: Joi.string()
    .when('NODE_ENV', {
      is: 'production',
      then: Joi.required(),
      otherwise: Joi.optional(),
    }),
});
```

**Using getOrThrow() for Runtime Checks:**

```typescript
@Injectable()
export class PaymentService {
  private stripeKey: string;
  
  constructor(private configService: ConfigService) {
    // Option 1: Check in constructor (fails early)
    this.stripeKey = this.configService.getOrThrow<string>('STRIPE_SECRET_KEY');
  }
  
  processPayment(amount: number) {
    // Option 2: Check when used (fails late)
    const key = this.configService.getOrThrow<string>('STRIPE_SECRET_KEY');
    
    return this.stripe.charge({ amount, key });
  }
}

// Custom error message with getOrThrow()
const password = this.configService.getOrThrow<string>(
  'DATABASE_PASSWORD',
  new Error('DATABASE_PASSWORD is required but not set in environment')
);
```

**Mixing Required and Optional:**

```typescript
@Module({
  imports: [
    ConfigModule.forRoot({
      validationSchema: Joi.object({
        // REQUIRED: Core functionality
        DATABASE_HOST: Joi.string().required(),
        DATABASE_PASSWORD: Joi.string().required(),
        JWT_SECRET: Joi.string().required(),
        
        // OPTIONAL: Has defaults
        PORT: Joi.number().default(3000),
        LOG_LEVEL: Joi.string().default('info'),
        
        // OPTIONAL: Feature flags
        ENABLE_CACHE: Joi.boolean().default(false),
        ENABLE_SWAGGER: Joi.boolean().default(true),
        
        // OPTIONAL: Third-party integrations
        SENDGRID_API_KEY: Joi.string().optional(),
        STRIPE_SECRET_KEY: Joi.string().optional(),
        
        // REQUIRED IF SET: Must be valid format if provided
        ADMIN_EMAIL: Joi.string()
          .email()
          .optional(),  // Optional, but must be valid email if set
        
        API_URL: Joi.string()
          .uri()
          .optional(),  // Optional, but must be valid URL if set
      }),
    }),
  ],
})
```

**Environment-Specific Required Variables:**

```bash
# .env.example
# REQUIRED in all environments:
DATABASE_HOST=required
DATABASE_PASSWORD=required
JWT_SECRET=required

# REQUIRED in production only:
# STRIPE_SECRET_KEY=required_in_production
# SSL_CERT_PATH=required_in_production

# OPTIONAL (have defaults):
# PORT=3000
# LOG_LEVEL=info
```

**Best Practices:**

```typescript
// ✅ GOOD: Make security-critical vars required
JWT_SECRET: Joi.string().required().min(32)
DATABASE_PASSWORD: Joi.string().required()
API_KEY: Joi.string().required()

// ✅ GOOD: Use getOrThrow() for required runtime config
const apiKey = this.configService.getOrThrow('EXTERNAL_API_KEY');

// ✅ GOOD: Provide defaults for optional config
PORT: Joi.number().default(3000)
LOG_LEVEL: Joi.string().default('info')

// ❌ BAD: No validation, fails at runtime
const password = this.configService.get('DATABASE_PASSWORD');
// Could be undefined → crash when connecting to DB

// ❌ BAD: Everything required (inflexible)
REDIS_HOST: Joi.string().required()  // Should be optional if Redis is optional

// ❌ BAD: Default for security-critical config
JWT_SECRET: Joi.string().default('insecure-default')  // Never!
```

**Interview Tip**: Make environment variables **required** with **Joi `.required()`** in validationSchema (recommended - fails at startup) or **`getOrThrow()`** (fails when accessed). Joi approach: validates **all config upfront**, provides **clear error messages**. Use **`.when()`** for **conditional requirements** (production vs development). Required for: **secrets** (JWT_SECRET, passwords), **critical config** (database credentials), **security settings**. Optional for: **feature flags**, **defaults available**, **third-party integrations**. **Best practice**: Joi validation for fail-fast behavior, clear error messages.

</details>

### 21. What happens if required env variables are missing?

<details>
<summary>Answer</summary>

**If required environment variables are missing**, the behavior depends on how you've configured validation:

**With Joi Validation (Recommended):**

```typescript
// app.module.ts
@Module({
  imports: [
    ConfigModule.forRoot({
      validationSchema: Joi.object({
        DATABASE_PASSWORD: Joi.string().required(),
        JWT_SECRET: Joi.string().required(),
      }),
    }),
  ],
})
export class AppModule {}

// If DATABASE_PASSWORD is missing:
// Application WILL NOT START ✅
```

**Error Output:**

```bash
# Terminal output when required variables are missing:

Error: Config validation error: "DATABASE_PASSWORD" is required
    at Object.<anonymous> (/app/src/main.ts:10:5)
    at Module._compile (node:internal/modules/cjs/loader:1105:14)

# Application exits with code 1
# Process stops before app.listen()
```

**Multiple Missing Variables (abortEarly: false):**

```typescript
ConfigModule.forRoot({
  validationSchema: Joi.object({
    DATABASE_PASSWORD: Joi.string().required(),
    JWT_SECRET: Joi.string().required(),
    API_KEY: Joi.string().required(),
  }),
  validationOptions: {
    abortEarly: false,  // Show all errors, not just first
  },
})

// If all three are missing:
/*
Error: Config validation error:
"DATABASE_PASSWORD" is required
"JWT_SECRET" is required
"API_KEY" is required
*/
```

**Without Validation (Fails Later):**

```typescript
// ❌ No validation in ConfigModule
@Module({
  imports: [
    ConfigModule.forRoot({
      // No validationSchema
    }),
  ],
})

// Application STARTS ❌ (problem!)

@Injectable()
export class DatabaseService {
  constructor(private configService: ConfigService) {}

  async connect() {
    // Missing DATABASE_PASSWORD returns undefined
    const password = this.configService.get('DATABASE_PASSWORD');
    // password = undefined
    
    // Later fails when trying to connect:
    const connection = await createConnection({
      password,  // undefined!
    });
    // Error: Connection failed - authentication error
    // Runtime error ❌ (not at startup)
  }
}
```

**Using getOrThrow() Without Joi:**

```typescript
@Injectable()
export class AuthService {
  constructor(private configService: ConfigService) {}

  generateToken() {
    // getOrThrow() throws if missing
    const secret = this.configService.getOrThrow<string>('JWT_SECRET');
    // If JWT_SECRET missing:
    // Error: An error occurred. Please check your environment configuration.
    
    return jwt.sign({}, secret);
  }
}

// Error happens when generateToken() is called, not at startup ⚠️
```

**Timeline of When Errors Occur:**

```typescript
// Scenario 1: With Joi Validation
// 1. npm run start
// 2. ConfigModule.forRoot() executes
// 3. Joi validates environment variables
// 4. Missing variable detected → CRASH ✅ (fails fast)
// 5. Application never starts
// Time: ~50ms

// Scenario 2: Without Joi, using getOrThrow()
// 1. npm run start
// 2. ConfigModule.forRoot() executes
// 3. No validation
// 4. Application starts successfully ⚠️
// 5. User makes request
// 6. generateToken() called
// 7. getOrThrow() detects missing JWT_SECRET → CRASH ❌
// Time: Could be seconds, minutes, or hours later

// Scenario 3: Without Joi, using get()
// 1. npm run start
// 2. Application starts ⚠️
// 3. User makes request
// 4. get('JWT_SECRET') returns undefined
// 5. jwt.sign({}, undefined) → error
// Time: Runtime error, unclear cause
```

**Production Impact:**

```typescript
// Without validation:
// ❌ App starts successfully in production
// ❌ Deploys to servers
// ❌ Passes health checks
// ❌ First user request crashes the service
// ❌ Database connection fails
// ❌ Authentication broken
// ❌ Hard to debug (why did it work in dev?)

// With Joi validation:
// ✅ Deployment fails immediately
// ✅ CI/CD catches the issue
// ✅ Never reaches production
// ✅ Clear error message
// ✅ Easy to fix before going live
```

**Real-World Failure Scenario:**

```typescript
// Production deployment without JWT_SECRET

// 1. Developer forgets to set JWT_SECRET in production environment
// 2. Application starts successfully (no validation)
// 3. Users can access public endpoints
// 4. First user tries to login
// 5. AuthService.generateToken() called
// 6. jwt.sign({}, undefined) fails
// 7. All auth requests fail
// 8. Site appears "broken" to users
// 9. Takes time to debug (check logs, find missing config)
// 10. Emergency fix required

// VS with Joi validation:
// 1. Developer forgets to set JWT_SECRET
// 2. Deployment script runs
// 3. Application startup fails immediately
// 4. Joi shows: "JWT_SECRET" is required
// 5. Developer adds JWT_SECRET
// 6. Redeploys successfully
// 7. No user impact ✅
```

**Error Handling Patterns:**

```typescript
// Pattern 1: Fail fast at startup (Recommended)
@Module({
  imports: [
    ConfigModule.forRoot({
      validationSchema: Joi.object({
        CRITICAL_VAR: Joi.string().required(),
      }),
    }),
  ],
})
// Missing CRITICAL_VAR → App won't start ✅

// Pattern 2: Graceful degradation (Feature flags)
@Injectable()
export class FeatureService {
  constructor(private configService: ConfigService) {}

  async sendEmail() {
    const apiKey = this.configService.get('SENDGRID_API_KEY');
    
    if (!apiKey) {
      // Gracefully handle missing config
      console.warn('SENDGRID_API_KEY not set, skipping email');
      return;  // Don't crash, just skip feature
    }
    
    // Send email
  }
}

// Pattern 3: Default fallback
@Injectable()
export class CacheService {
  constructor(private configService: ConfigService) {}

  getTimeout(): number {
    // Provide default if missing
    return this.configService.get<number>('CACHE_TIMEOUT', 300);
  }
}
```

**Logging and Monitoring:**

```typescript
// Custom error handling for better debugging
@Module({
  imports: [
    ConfigModule.forRoot({
      validationSchema: Joi.object({
        DATABASE_PASSWORD: Joi.string().required(),
        JWT_SECRET: Joi.string().required(),
      }),
      validationOptions: {
        abortEarly: false,
      },
    }),
  ],
})

// In production, log validation errors before crash
async function bootstrap() {
  try {
    const app = await NestFactory.create(AppModule);
    await app.listen(3000);
  } catch (error) {
    // Log to monitoring service
    logger.error('Failed to start application', {
      error: error.message,
      stack: error.stack,
      environment: process.env.NODE_ENV,
    });
    
    // Alert team
    await notifyTeam('App startup failed', error.message);
    
    // Exit
    process.exit(1);
  }
}
```

**CI/CD Integration:**

```yaml
# .github/workflows/deploy.yml

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      
      - name: Set environment variables
        run: |
          echo "DATABASE_PASSWORD=${{ secrets.DATABASE_PASSWORD }}" >> .env
          echo "JWT_SECRET=${{ secrets.JWT_SECRET }}" >> .env
      
      - name: Validate configuration
        run: npm run validate-config  # Custom script to check config
      
      - name: Build application
        run: npm run build
      
      - name: Deploy
        run: npm run deploy
        # ✅ Validation fails → build stops → deployment prevented
```

**Interview Tip**: With **Joi validation**, missing required variables cause the app to **fail at startup** with **clear error messages** (recommended). Without validation, app **starts successfully** but **fails at runtime** when variable is accessed (dangerous in production). Use `validationOptions: { abortEarly: false }` to show **all missing variables at once**. **`getOrThrow()`** throws error when accessed (not at startup). **Best practice**: Joi validation for **fail-fast behavior**, prevents broken deployments, catches issues in CI/CD, clear debugging.

</details>

## Custom Configuration Files

### 22. How do you create custom configuration files (e.g., database.config.ts)?

<details>
<summary>Answer</summary>

**Custom configuration files** organize related settings into separate modules, making configuration more maintainable and type-safe. They transform environment variables into structured objects.

**Basic Custom Configuration File:**

```typescript
// config/database.config.ts
export default () => ({
  database: {
    type: 'postgres',
    host: process.env.DATABASE_HOST || 'localhost',
    port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
    username: process.env.DATABASE_USERNAME,
    password: process.env.DATABASE_PASSWORD,
    database: process.env.DATABASE_NAME,
    synchronize: process.env.DATABASE_SYNC === 'true',
    logging: process.env.DATABASE_LOGGING === 'true',
  },
});

// Load in AppModule
import databaseConfig from './config/database.config';

@Module({
  imports: [
    ConfigModule.forRoot({
      load: [databaseConfig],  // Load custom config
      isGlobal: true,
    }),
  ],
})
export class AppModule {}

// Access in service
@Injectable()
export class AppService {
  constructor(private configService: ConfigService) {}

  getDatabaseHost() {
    return this.configService.get('database.host');
  }
}
```

**Multiple Custom Configuration Files:**

```typescript
// config/database.config.ts
export default () => ({
  database: {
    host: process.env.DATABASE_HOST || 'localhost',
    port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
    username: process.env.DATABASE_USERNAME,
    password: process.env.DATABASE_PASSWORD,
  },
});

// config/jwt.config.ts
export default () => ({
  jwt: {
    secret: process.env.JWT_SECRET,
    expiresIn: process.env.JWT_EXPIRES_IN || '1d',
    refreshSecret: process.env.JWT_REFRESH_SECRET,
    refreshExpiresIn: process.env.JWT_REFRESH_EXPIRES_IN || '7d',
  },
});

// config/redis.config.ts
export default () => ({
  redis: {
    host: process.env.REDIS_HOST || 'localhost',
    port: parseInt(process.env.REDIS_PORT, 10) || 6379,
    password: process.env.REDIS_PASSWORD,
    ttl: parseInt(process.env.REDIS_TTL, 10) || 300,
  },
});

// config/app.config.ts
export default () => ({
  app: {
    port: parseInt(process.env.PORT, 10) || 3000,
    environment: process.env.NODE_ENV || 'development',
    apiPrefix: process.env.API_PREFIX || 'api',
    corsOrigins: process.env.CORS_ORIGINS?.split(',') || ['*'],
  },
});

// Load all configs in AppModule
import databaseConfig from './config/database.config';
import jwtConfig from './config/jwt.config';
import redisConfig from './config/redis.config';
import appConfig from './config/app.config';

@Module({
  imports: [
    ConfigModule.forRoot({
      load: [databaseConfig, jwtConfig, redisConfig, appConfig],
      isGlobal: true,
    }),
  ],
})
export class AppModule {}
```

**Complex Configuration with Computation:**

```typescript
// config/aws.config.ts
export default () => {
  const region = process.env.AWS_REGION || 'us-east-1';
  const bucket = process.env.AWS_S3_BUCKET;
  
  return {
    aws: {
      region,
      credentials: {
        accessKeyId: process.env.AWS_ACCESS_KEY_ID,
        secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
      },
      s3: {
        bucket,
        // Computed URL
        url: bucket ? `https://${bucket}.s3.${region}.amazonaws.com` : null,
        // Configuration based on environment
        signedUrlExpiration: process.env.NODE_ENV === 'production' ? 3600 : 7200,
      },
    },
  };
};
```

**Configuration with TypeScript Types:**

```typescript
// config/types/database.types.ts
export interface DatabaseConfig {
  type: 'postgres' | 'mysql' | 'mariadb';
  host: string;
  port: number;
  username: string;
  password: string;
  database: string;
  pool: {
    min: number;
    max: number;
  };
  options: {
    ssl: boolean;
    synchronize: boolean;
    logging: boolean;
  };
}

// config/database.config.ts
import { DatabaseConfig } from './types/database.types';

export default (): { database: DatabaseConfig } => ({
  database: {
    type: (process.env.DATABASE_TYPE as any) || 'postgres',
    host: process.env.DATABASE_HOST || 'localhost',
    port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
    username: process.env.DATABASE_USERNAME!,
    password: process.env.DATABASE_PASSWORD!,
    database: process.env.DATABASE_NAME!,
    pool: {
      min: parseInt(process.env.DATABASE_POOL_MIN, 10) || 2,
      max: parseInt(process.env.DATABASE_POOL_MAX, 10) || 10,
    },
    options: {
      ssl: process.env.DATABASE_SSL === 'true',
      synchronize: process.env.DATABASE_SYNC === 'true',
      logging: process.env.DATABASE_LOGGING === 'true',
    },
  },
});
```

**Folder Structure for Config Files:**

```
src/
├── config/
│   ├── types/
│   │   ├── database.types.ts
│   │   ├── jwt.types.ts
│   │   └── app.types.ts
│   ├── database.config.ts
│   ├── jwt.config.ts
│   ├── redis.config.ts
│   ├── app.config.ts
│   ├── aws.config.ts
│   └── index.ts              # Exports all configs
├── app.module.ts
└── main.ts
```

**Index File to Export All Configs:**

```typescript
// config/index.ts
export { default as databaseConfig } from './database.config';
export { default as jwtConfig } from './jwt.config';
export { default as redisConfig } from './redis.config';
export { default as appConfig } from './app.config';

// app.module.ts
import * as configs from './config';

@Module({
  imports: [
    ConfigModule.forRoot({
      load: Object.values(configs),  // Load all configs
      isGlobal: true,
    }),
  ],
})
export class AppModule {}
```

**Environment-Specific Configuration:**

```typescript
// config/app.config.ts
export default () => {
  const isDevelopment = process.env.NODE_ENV === 'development';
  const isProduction = process.env.NODE_ENV === 'production';
  
  return {
    app: {
      port: parseInt(process.env.PORT, 10) || (isDevelopment ? 3000 : 8080),
      
      // Development vs Production settings
      logging: {
        level: isDevelopment ? 'debug' : 'error',
        pretty: isDevelopment,
      },
      
      // CORS settings
      cors: {
        enabled: process.env.CORS_ENABLED === 'true',
        origins: isDevelopment 
          ? ['http://localhost:3000', 'http://localhost:4200']
          : process.env.CORS_ORIGINS?.split(',') || [],
      },
      
      // Cache settings
      cache: {
        ttl: isDevelopment ? 60 : 3600,  // 1 minute dev, 1 hour prod
        enabled: isProduction,
      },
      
      // Swagger (only in development)
      swagger: {
        enabled: isDevelopment,
        path: 'api-docs',
      },
    },
  };
};
```

**Configuration with Validation:**

```typescript
// config/database.config.ts
export default () => {
  // Validate required variables
  const requiredVars = ['DATABASE_HOST', 'DATABASE_USERNAME', 'DATABASE_PASSWORD'];
  const missing = requiredVars.filter(v => !process.env[v]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required database config: ${missing.join(', ')}`);
  }
  
  const port = parseInt(process.env.DATABASE_PORT, 10);
  if (isNaN(port) || port < 1 || port > 65535) {
    throw new Error('DATABASE_PORT must be a valid port number');
  }
  
  return {
    database: {
      host: process.env.DATABASE_HOST,
      port,
      username: process.env.DATABASE_USERNAME,
      password: process.env.DATABASE_PASSWORD,
      database: process.env.DATABASE_NAME || 'myapp',
    },
  };
};
```

**Accessing Custom Config:**

```typescript
@Injectable()
export class DatabaseService {
  constructor(private configService: ConfigService) {}

  getConnection() {
    // Access nested properties
    const host = this.configService.get('database.host');
    const port = this.configService.get('database.port');
    
    // Or get entire object
    const dbConfig = this.configService.get('database');
    
    return createConnection(dbConfig);
  }
}
```

**Interview Tip**: **Custom configuration files** organize related environment variables into **separate modules** (database.config.ts, jwt.config.ts, etc.). Export **default function** returning configuration object. Load with **`load` option** in ConfigModule.forRoot(). Access via **ConfigService** using dot notation (`'database.host'`). **Benefits**: organized code, type safety, computed values, validation, environment-specific logic. **Best practice**: one file per feature/service, TypeScript interfaces for types, group in `config/` folder.

</details>

### 23. How do you use `registerAs()` to create namespaced configuration?

<details>
<summary>Answer</summary>

**`registerAs()`** is a NestJS utility that creates **namespaced configuration**, allowing you to inject specific config sections with type safety using `@Inject()`.

**Basic registerAs() Usage:**

```typescript
// config/database.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('database', () => ({
  // Namespace: 'database'
  host: process.env.DATABASE_HOST || 'localhost',
  port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
  username: process.env.DATABASE_USERNAME,
  password: process.env.DATABASE_PASSWORD,
  database: process.env.DATABASE_NAME,
}));

// Load in AppModule
import databaseConfig from './config/database.config';

@Module({
  imports: [
    ConfigModule.forRoot({
      load: [databaseConfig],
      isGlobal: true,
    }),
  ],
})
export class AppModule {}
```

**Accessing Namespaced Config:**

```typescript
// Method 1: Using ConfigService with dot notation
@Injectable()
export class DatabaseService {
  constructor(private configService: ConfigService) {}

  getHost() {
    return this.configService.get('database.host');
  }

  getConfig() {
    return this.configService.get('database');
  }
}

// Method 2: Using @Inject() with namespace (Type-Safe)
import { Inject, Injectable } from '@nestjs/common';
import databaseConfig from './config/database.config';
import { ConfigType } from '@nestjs/config';

@Injectable()
export class DatabaseService {
  constructor(
    @Inject(databaseConfig.KEY)
    private dbConfig: ConfigType<typeof databaseConfig>,
  ) {}

  getHost() {
    return this.dbConfig.host;  // ✅ Type-safe!
  }

  getConfig() {
    return this.dbConfig;  // ✅ Full type safety
  }
}
```

**Multiple Namespaced Configurations:**

```typescript
// config/database.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('database', () => ({
  host: process.env.DATABASE_HOST || 'localhost',
  port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
  username: process.env.DATABASE_USERNAME,
  password: process.env.DATABASE_PASSWORD,
}));

// config/jwt.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('jwt', () => ({
  secret: process.env.JWT_SECRET,
  expiresIn: process.env.JWT_EXPIRES_IN || '1d',
  refreshSecret: process.env.JWT_REFRESH_SECRET,
  refreshExpiresIn: process.env.JWT_REFRESH_EXPIRES_IN || '7d',
}));

// config/redis.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('redis', () => ({
  host: process.env.REDIS_HOST || 'localhost',
  port: parseInt(process.env.REDIS_PORT, 10) || 6379,
  password: process.env.REDIS_PASSWORD,
  ttl: parseInt(process.env.REDIS_TTL, 10) || 300,
}));

// app.module.ts
import databaseConfig from './config/database.config';
import jwtConfig from './config/jwt.config';
import redisConfig from './config/redis.config';

@Module({
  imports: [
    ConfigModule.forRoot({
      load: [databaseConfig, jwtConfig, redisConfig],
      isGlobal: true,
    }),
  ],
})
export class AppModule {}
```

**Type-Safe Injection with @Inject():**

```typescript
// config/database.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('database', () => ({
  host: process.env.DATABASE_HOST || 'localhost',
  port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
  username: process.env.DATABASE_USERNAME,
  password: process.env.DATABASE_PASSWORD,
  pool: {
    min: parseInt(process.env.DATABASE_POOL_MIN, 10) || 2,
    max: parseInt(process.env.DATABASE_POOL_MAX, 10) || 10,
  },
}));

// database.service.ts
import { Injectable, Inject } from '@nestjs/common';
import { ConfigType } from '@nestjs/config';
import databaseConfig from './config/database.config';

@Injectable()
export class DatabaseService {
  constructor(
    @Inject(databaseConfig.KEY)  // Inject namespaced config
    private dbConfig: ConfigType<typeof databaseConfig>,
  ) {}

  createConnection() {
    // ✅ Full TypeScript autocomplete and type checking
    console.log(this.dbConfig.host);       // Type: string
    console.log(this.dbConfig.port);       // Type: number
    console.log(this.dbConfig.pool.min);   // Type: number
    
    return {
      host: this.dbConfig.host,
      port: this.dbConfig.port,
      username: this.dbConfig.username,
      password: this.dbConfig.password,
      pool: {
        min: this.dbConfig.pool.min,
        max: this.dbConfig.pool.max,
      },
    };
  }
}
```

**ConfigType Utility:**

```typescript
// ConfigType infers the return type of your config function

import { ConfigType } from '@nestjs/config';
import databaseConfig from './config/database.config';

// Type is inferred from registerAs() return value
type DatabaseConfigType = ConfigType<typeof databaseConfig>;
// Equivalent to:
// {
//   host: string;
//   port: number;
//   username: string | undefined;
//   password: string | undefined;
//   pool: { min: number; max: number };
// }
```

**Real-World Example - JWT Configuration:**

```typescript
// config/jwt.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('jwt', () => ({
  access: {
    secret: process.env.JWT_SECRET || 'default-secret',
    expiresIn: process.env.JWT_EXPIRES_IN || '15m',
  },
  refresh: {
    secret: process.env.JWT_REFRESH_SECRET || 'refresh-secret',
    expiresIn: process.env.JWT_REFRESH_EXPIRES_IN || '7d',
  },
  issuer: process.env.JWT_ISSUER || 'my-app',
  audience: process.env.JWT_AUDIENCE || 'my-app-users',
}));

// auth.service.ts
import { Injectable, Inject } from '@nestjs/common';
import { ConfigType } from '@nestjs/config';
import jwtConfig from './config/jwt.config';
import { JwtService } from '@nestjs/jwt';

@Injectable()
export class AuthService {
  constructor(
    @Inject(jwtConfig.KEY)
    private jwt: ConfigType<typeof jwtConfig>,
    private jwtService: JwtService,
  ) {}

  generateAccessToken(payload: any) {
    return this.jwtService.sign(payload, {
      secret: this.jwt.access.secret,
      expiresIn: this.jwt.access.expiresIn,
      issuer: this.jwt.issuer,
      audience: this.jwt.audience,
    });
  }

  generateRefreshToken(payload: any) {
    return this.jwtService.sign(payload, {
      secret: this.jwt.refresh.secret,
      expiresIn: this.jwt.refresh.expiresIn,
    });
  }
}
```

**Comparison: registerAs() vs Plain Config:**

```typescript
// ❌ Without registerAs() (No namespace, no type safety)
// config/database.config.ts
export default () => ({
  databaseHost: process.env.DATABASE_HOST,
  databasePort: parseInt(process.env.DATABASE_PORT, 10),
});

// Service
@Injectable()
export class Service {
  constructor(private config: ConfigService) {}

  getHost() {
    return this.config.get('databaseHost');  // ❌ String key, no autocomplete
  }
}

// ✅ With registerAs() (Namespaced, type-safe)
// config/database.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('database', () => ({
  host: process.env.DATABASE_HOST,
  port: parseInt(process.env.DATABASE_PORT, 10),
}));

// Service
import { ConfigType } from '@nestjs/config';
import databaseConfig from './config/database.config';

@Injectable()
export class Service {
  constructor(
    @Inject(databaseConfig.KEY)
    private dbConfig: ConfigType<typeof databaseConfig>,
  ) {}

  getHost() {
    return this.dbConfig.host;  // ✅ Type-safe, autocomplete
  }
}
```

**registerAs() with Complex Types:**

```typescript
// config/app.config.ts
import { registerAs } from '@nestjs/config';

interface CorsConfig {
  enabled: boolean;
  origins: string[];
  credentials: boolean;
}

interface RateLimitConfig {
  ttl: number;
  limit: number;
}

export default registerAs('app', () => ({
  port: parseInt(process.env.PORT, 10) || 3000,
  environment: process.env.NODE_ENV || 'development',
  
  cors: {
    enabled: process.env.CORS_ENABLED === 'true',
    origins: process.env.CORS_ORIGINS?.split(',') || ['*'],
    credentials: true,
  } as CorsConfig,
  
  rateLimit: {
    ttl: parseInt(process.env.RATE_LIMIT_TTL, 10) || 60,
    limit: parseInt(process.env.RATE_LIMIT_MAX, 10) || 100,
  } as RateLimitConfig,
}));

// app.service.ts
import { Injectable, Inject } from '@nestjs/common';
import { ConfigType } from '@nestjs/config';
import appConfig from './config/app.config';

@Injectable()
export class AppService {
  constructor(
    @Inject(appConfig.KEY)
    private app: ConfigType<typeof appConfig>,
  ) {}

  getCorsConfig() {
    // ✅ Type-safe access
    return {
      enabled: this.app.cors.enabled,
      origins: this.app.cors.origins,
    };
  }
}
```

**Interview Tip**: **`registerAs()`** creates **namespaced configuration** with first parameter as **namespace name** ('database', 'jwt', etc.). Enables **type-safe injection** using **`@Inject(config.KEY)`** and **`ConfigType<typeof config>`**. Access via: **ConfigService** with dot notation (`'database.host'`) or **direct injection** (recommended for type safety). **Benefits**: TypeScript autocomplete, compile-time type checking, organized configuration, easier testing. **Best practice**: always use registerAs() for custom configs, inject with ConfigType for type safety.

</details>

### 24. How do you load custom config files using `load` option?

<details>
<summary>Answer</summary>

The **`load` option** in `ConfigModule.forRoot()` accepts an **array of configuration factories** (functions that return configuration objects), allowing you to load multiple custom configuration files.

**Basic Usage:**

```typescript
// config/database.config.ts
export default () => ({
  database: {
    host: process.env.DATABASE_HOST || 'localhost',
    port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
  },
});

// app.module.ts
import databaseConfig from './config/database.config';

@Module({
  imports: [
    ConfigModule.forRoot({
      load: [databaseConfig],  // Load custom config file
      isGlobal: true,
    }),
  ],
})
export class AppModule {}
```

**Loading Multiple Configuration Files:**

```typescript
// config/database.config.ts
export default () => ({
  database: {
    host: process.env.DATABASE_HOST || 'localhost',
    port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
    username: process.env.DATABASE_USERNAME,
    password: process.env.DATABASE_PASSWORD,
  },
});

// config/jwt.config.ts
export default () => ({
  jwt: {
    secret: process.env.JWT_SECRET,
    expiresIn: process.env.JWT_EXPIRES_IN || '1d',
  },
});

// config/redis.config.ts
export default () => ({
  redis: {
    host: process.env.REDIS_HOST || 'localhost',
    port: parseInt(process.env.REDIS_PORT, 10) || 6379,
  },
});

// app.module.ts
import databaseConfig from './config/database.config';
import jwtConfig from './config/jwt.config';
import redisConfig from './config/redis.config';

@Module({
  imports: [
    ConfigModule.forRoot({
      load: [databaseConfig, jwtConfig, redisConfig],  // Load all configs
      isGlobal: true,
    }),
  ],
})
export class AppModule {}

// Access in services
@Injectable()
export class AppService {
  constructor(private configService: ConfigService) {}

  getConfig() {
    return {
      dbHost: this.configService.get('database.host'),
      jwtSecret: this.configService.get('jwt.secret'),
      redisPort: this.configService.get('redis.port'),
    };
  }
}
```

**Using registerAs() with load:**

```typescript
// config/database.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('database', () => ({
  host: process.env.DATABASE_HOST || 'localhost',
  port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
  username: process.env.DATABASE_USERNAME,
  password: process.env.DATABASE_PASSWORD,
}));

// config/jwt.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('jwt', () => ({
  secret: process.env.JWT_SECRET,
  expiresIn: process.env.JWT_EXPIRES_IN || '1d',
  refreshSecret: process.env.JWT_REFRESH_SECRET,
  refreshExpiresIn: process.env.JWT_REFRESH_EXPIRES_IN || '7d',
}));

// app.module.ts
import databaseConfig from './config/database.config';
import jwtConfig from './config/jwt.config';

@Module({
  imports: [
    ConfigModule.forRoot({
      load: [databaseConfig, jwtConfig],  // Load namespaced configs
      isGlobal: true,
    }),
  ],
})
export class AppModule {}
```

**Organizing Config Files with Index:**

```typescript
// config/index.ts
export { default as appConfig } from './app.config';
export { default as databaseConfig } from './database.config';
export { default as jwtConfig } from './jwt.config';
export { default as redisConfig } from './redis.config';
export { default as awsConfig } from './aws.config';
export { default as emailConfig } from './email.config';

// app.module.ts
import * as configs from './config';

@Module({
  imports: [
    ConfigModule.forRoot({
      load: Object.values(configs),  // Load all configs from index
      isGlobal: true,
    }),
  ],
})
export class AppModule {}
```

**Complete Real-World Example:**

```typescript
// config/app.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('app', () => ({
  port: parseInt(process.env.PORT, 10) || 3000,
  environment: process.env.NODE_ENV || 'development',
  apiPrefix: process.env.API_PREFIX || 'api',
  corsOrigins: process.env.CORS_ORIGINS?.split(',') || ['*'],
}));

// config/database.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('database', () => ({
  type: 'postgres',
  host: process.env.DATABASE_HOST || 'localhost',
  port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
  username: process.env.DATABASE_USERNAME,
  password: process.env.DATABASE_PASSWORD,
  database: process.env.DATABASE_NAME,
  synchronize: process.env.DATABASE_SYNC === 'true',
  logging: process.env.DATABASE_LOGGING === 'true',
  pool: {
    min: parseInt(process.env.DATABASE_POOL_MIN, 10) || 2,
    max: parseInt(process.env.DATABASE_POOL_MAX, 10) || 10,
  },
}));

// config/jwt.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('jwt', () => ({
  access: {
    secret: process.env.JWT_SECRET,
    expiresIn: process.env.JWT_EXPIRES_IN || '15m',
  },
  refresh: {
    secret: process.env.JWT_REFRESH_SECRET,
    expiresIn: process.env.JWT_REFRESH_EXPIRES_IN || '7d',
  },
}));

// config/redis.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('redis', () => ({
  host: process.env.REDIS_HOST || 'localhost',
  port: parseInt(process.env.REDIS_PORT, 10) || 6379,
  password: process.env.REDIS_PASSWORD,
  ttl: parseInt(process.env.REDIS_TTL, 10) || 300,
}));

// config/aws.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('aws', () => ({
  region: process.env.AWS_REGION || 'us-east-1',
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
  },
  s3: {
    bucket: process.env.AWS_S3_BUCKET,
  },
}));

// config/index.ts
export { default as appConfig } from './app.config';
export { default as databaseConfig } from './database.config';
export { default as jwtConfig } from './jwt.config';
export { default as redisConfig } from './redis.config';
export { default as awsConfig } from './aws.config';

// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import * as configs from './config';
import * as Joi from 'joi';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: '.env',
      
      // Load all custom configuration files
      load: Object.values(configs),
      
      // Validation schema
      validationSchema: Joi.object({
        NODE_ENV: Joi.string().valid('development', 'production', 'test').default('development'),
        PORT: Joi.number().default(3000),
        DATABASE_HOST: Joi.string().required(),
        DATABASE_PASSWORD: Joi.string().required(),
        JWT_SECRET: Joi.string().required(),
      }),
    }),
  ],
})
export class AppModule {}
```

**Conditional Loading:**

```typescript
// Load different configs based on environment
const configs = [
  appConfig,
  databaseConfig,
  jwtConfig,
];

// Only load Redis config in production
if (process.env.NODE_ENV === 'production') {
  configs.push(redisConfig);
}

// Only load AWS config if credentials are set
if (process.env.AWS_ACCESS_KEY_ID) {
  configs.push(awsConfig);
}

@Module({
  imports: [
    ConfigModule.forRoot({
      load: configs,  // Conditionally loaded configs
      isGlobal: true,
    }),
  ],
})
export class AppModule {}
```

**Config Files with Validation:**

```typescript
// config/database.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('database', () => {
  // Validate required vars in config file
  const requiredVars = ['DATABASE_HOST', 'DATABASE_USERNAME', 'DATABASE_PASSWORD'];
  const missing = requiredVars.filter(v => !process.env[v]);
  
  if (missing.length > 0) {
    throw new Error(`Missing database config: ${missing.join(', ')}`);
  }
  
  return {
    host: process.env.DATABASE_HOST,
    port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
    username: process.env.DATABASE_USERNAME,
    password: process.env.DATABASE_PASSWORD,
    database: process.env.DATABASE_NAME || 'myapp',
  };
});

// Load with error handling
@Module({
  imports: [
    ConfigModule.forRoot({
      load: [databaseConfig],  // Will throw if validation fails
      isGlobal: true,
    }),
  ],
})
export class AppModule {}
```

**Dynamic Configuration:**

```typescript
// config/app.config.ts
export default () => {
  const isDevelopment = process.env.NODE_ENV === 'development';
  const isProduction = process.env.NODE_ENV === 'production';
  
  return {
    app: {
      port: parseInt(process.env.PORT, 10) || (isDevelopment ? 3000 : 8080),
      
      // Development-specific config
      ...(isDevelopment && {
        swagger: {
          enabled: true,
          path: 'api-docs',
        },
        hotReload: true,
      }),
      
      // Production-specific config
      ...(isProduction && {
        cache: {
          enabled: true,
          ttl: 3600,
        },
        compression: true,
      }),
    },
  };
};
```

**Loading Order and Priority:**

```typescript
// Load order matters - later configs can override earlier ones
@Module({
  imports: [
    ConfigModule.forRoot({
      envFilePath: ['.env', '.env.local'],  // .env.local overrides .env
      
      load: [
        baseConfig,      // 1. Base configuration
        databaseConfig,  // 2. Database config (can reference base)
        cacheConfig,     // 3. Cache config (can reference database)
      ],
      
      isGlobal: true,
    }),
  ],
})
export class AppModule {}
```

**Async Configuration Loading:**

```typescript
// config/database.config.ts
export default async () => {
  // Could fetch config from remote service
  const remoteConfig = await fetchRemoteConfig();
  
  return {
    database: {
      host: remoteConfig.dbHost || process.env.DATABASE_HOST,
      port: remoteConfig.dbPort || parseInt(process.env.DATABASE_PORT, 10),
    },
  };
};

// Note: Async configs should use ConfigModule.forRootAsync()
```

**Folder Structure:**

```
src/
├── config/
│   ├── types/
│   │   ├── app.types.ts
│   │   ├── database.types.ts
│   │   └── jwt.types.ts
│   ├── app.config.ts
│   ├── database.config.ts
│   ├── jwt.config.ts
│   ├── redis.config.ts
│   ├── aws.config.ts
│   └── index.ts          # Exports all configs
├── app.module.ts
└── main.ts
```

**Best Practices:**

```typescript
// ✅ GOOD: Organize related configs into separate files
load: [databaseConfig, jwtConfig, redisConfig]

// ✅ GOOD: Use index.ts to export all configs
import * as configs from './config';
load: Object.values(configs)

// ✅ GOOD: Use registerAs() for namespacing
registerAs('database', () => ({ ... }))

// ✅ GOOD: Validate in config files for early error detection
if (!process.env.JWT_SECRET) throw new Error('JWT_SECRET required');

// ❌ BAD: Loading all configs in one giant file
load: [allConfigsInOneFile]

// ❌ BAD: No namespacing (flat structure)
export default () => ({ dbHost, dbPort, jwtSecret, redisHost })

// ❌ BAD: No organization
load: [config1, config2, config3, config4, config5...]
```

**Interview Tip**: The **`load` option** accepts an **array of configuration factory functions** to load custom config files. Each factory returns a configuration object. Use with **`registerAs()`** for namespaced configs. Load order: `.env file → custom configs → validation`. Access via **`ConfigService.get()`** using namespace: `'database.host'`. Export all configs from **`config/index.ts`** and load with `Object.values(configs)`. **Best practice**: one file per feature/domain, use registerAs() for namespacing, validate in config files, organize in config folder.

</details>

### 25. How do you inject namespaced config using `@Inject()`?

<details>
<summary>Answer</summary>

**`@Inject()`** allows you to inject **namespaced configuration** created with `registerAs()` directly into services with **full type safety**, avoiding string-based keys and providing TypeScript autocomplete.

**Basic @Inject() Usage:**

```typescript
// config/database.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('database', () => ({
  host: process.env.DATABASE_HOST || 'localhost',
  port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
  username: process.env.DATABASE_USERNAME,
  password: process.env.DATABASE_PASSWORD,
}));

// database.service.ts
import { Injectable, Inject } from '@nestjs/common';
import { ConfigType } from '@nestjs/config';
import databaseConfig from './config/database.config';

@Injectable()
export class DatabaseService {
  constructor(
    @Inject(databaseConfig.KEY)  // Inject using config.KEY
    private dbConfig: ConfigType<typeof databaseConfig>,  // Type-safe
  ) {}

  getConnection() {
    // ✅ Full TypeScript autocomplete and type checking
    return {
      host: this.dbConfig.host,       // Type: string
      port: this.dbConfig.port,       // Type: number
      username: this.dbConfig.username,
      password: this.dbConfig.password,
    };
  }
}
```

**ConfigType Utility:**

```typescript
// ConfigType infers the return type from registerAs()
import { ConfigType } from '@nestjs/config';
import databaseConfig from './config/database.config';

// ConfigType<typeof databaseConfig> = return type of the factory function
type DatabaseConfigType = ConfigType<typeof databaseConfig>;
// Equivalent to:
// {
//   host: string;
//   port: number;
//   username: string | undefined;
//   password: string | undefined;
// }
```

**Multiple Config Injections:**

```typescript
// config/database.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('database', () => ({
  host: process.env.DATABASE_HOST || 'localhost',
  port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
}));

// config/jwt.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('jwt', () => ({
  secret: process.env.JWT_SECRET,
  expiresIn: process.env.JWT_EXPIRES_IN || '1d',
}));

// auth.service.ts
import { Injectable, Inject } from '@nestjs/common';
import { ConfigType } from '@nestjs/config';
import databaseConfig from './config/database.config';
import jwtConfig from './config/jwt.config';

@Injectable()
export class AuthService {
  constructor(
    @Inject(databaseConfig.KEY)
    private dbConfig: ConfigType<typeof databaseConfig>,
    
    @Inject(jwtConfig.KEY)
    private jwt: ConfigType<typeof jwtConfig>,
  ) {}

  async authenticate() {
    // Access both configs with type safety
    const dbHost = this.dbConfig.host;
    const secret = this.jwt.secret;
    
    return { dbHost, secret };
  }
}
```

**Real-World Example - JWT Configuration:**

```typescript
// config/jwt.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('jwt', () => ({
  access: {
    secret: process.env.JWT_SECRET || 'secret',
    expiresIn: process.env.JWT_EXPIRES_IN || '15m',
    algorithm: 'HS256' as const,
  },
  refresh: {
    secret: process.env.JWT_REFRESH_SECRET || 'refresh-secret',
    expiresIn: process.env.JWT_REFRESH_EXPIRES_IN || '7d',
    algorithm: 'HS256' as const,
  },
  issuer: process.env.JWT_ISSUER || 'my-app',
  audience: process.env.JWT_AUDIENCE || 'my-app-users',
}));

// auth.service.ts
import { Injectable, Inject } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { ConfigType } from '@nestjs/config';
import jwtConfig from './config/jwt.config';

@Injectable()
export class AuthService {
  constructor(
    @Inject(jwtConfig.KEY)
    private jwt: ConfigType<typeof jwtConfig>,
    private jwtService: JwtService,
  ) {}

  generateAccessToken(userId: string) {
    const payload = { sub: userId };
    
    // ✅ Type-safe access to nested config
    return this.jwtService.sign(payload, {
      secret: this.jwt.access.secret,
      expiresIn: this.jwt.access.expiresIn,
      algorithm: this.jwt.access.algorithm,
      issuer: this.jwt.issuer,
      audience: this.jwt.audience,
    });
  }

  generateRefreshToken(userId: string) {
    const payload = { sub: userId };
    
    return this.jwtService.sign(payload, {
      secret: this.jwt.refresh.secret,
      expiresIn: this.jwt.refresh.expiresIn,
      algorithm: this.jwt.refresh.algorithm,
    });
  }

  verifyAccessToken(token: string) {
    return this.jwtService.verify(token, {
      secret: this.jwt.access.secret,
      issuer: this.jwt.issuer,
      audience: this.jwt.audience,
    });
  }
}
```

**Complex Nested Configuration:**

```typescript
// config/app.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('app', () => ({
  port: parseInt(process.env.PORT, 10) || 3000,
  environment: process.env.NODE_ENV || 'development',
  
  cors: {
    enabled: process.env.CORS_ENABLED === 'true',
    origins: process.env.CORS_ORIGINS?.split(',') || ['*'],
    credentials: true,
    methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
  },
  
  rateLimit: {
    ttl: parseInt(process.env.RATE_LIMIT_TTL, 10) || 60,
    limit: parseInt(process.env.RATE_LIMIT_MAX, 10) || 100,
  },
  
  logging: {
    level: process.env.LOG_LEVEL || 'info',
    format: process.env.LOG_FORMAT || 'json',
  },
}));

// app.service.ts
import { Injectable, Inject } from '@nestjs/common';
import { ConfigType } from '@nestjs/config';
import appConfig from './config/app.config';

@Injectable()
export class AppService {
  constructor(
    @Inject(appConfig.KEY)
    private app: ConfigType<typeof appConfig>,
  ) {}

  getCorsConfig() {
    // ✅ Deep nested access with type safety
    return {
      enabled: this.app.cors.enabled,
      origins: this.app.cors.origins,
      credentials: this.app.cors.credentials,
      methods: this.app.cors.methods,
    };
  }

  getRateLimitConfig() {
    return {
      ttl: this.app.rateLimit.ttl,
      limit: this.app.rateLimit.limit,
    };
  }

  getPort(): number {
    return this.app.port;  // Type: number (not string!)
  }
}
```

**@Inject() vs ConfigService Comparison:**

```typescript
// Method 1: ConfigService (String-based, No autocomplete)
@Injectable()
export class ServiceA {
  constructor(private configService: ConfigService) {}

  getHost() {
    return this.configService.get('database.host');  // ❌ String key, no type safety
  }
}

// Method 2: @Inject() (Type-safe, Autocomplete)
import { ConfigType } from '@nestjs/config';
import databaseConfig from './config/database.config';

@Injectable()
export class ServiceB {
  constructor(
    @Inject(databaseConfig.KEY)
    private dbConfig: ConfigType<typeof databaseConfig>,
  ) {}

  getHost() {
    return this.dbConfig.host;  // ✅ Type-safe, autocomplete
  }
}
```

**Using Both @Inject() and ConfigService:**

```typescript
@Injectable()
export class MixedService {
  constructor(
    // Inject specific namespace for frequently used config
    @Inject(databaseConfig.KEY)
    private dbConfig: ConfigType<typeof databaseConfig>,
    
    // Keep ConfigService for dynamic/rare access
    private configService: ConfigService,
  ) {}

  getDbHost() {
    // Use injected config for type safety
    return this.dbConfig.host;
  }

  getDynamicValue(key: string) {
    // Use ConfigService for dynamic keys
    return this.configService.get(key);
  }
}
```

**Testing with Injected Config:**

```typescript
// database.service.spec.ts
import { Test } from '@nestjs/testing';
import { DatabaseService } from './database.service';
import databaseConfig from './config/database.config';

describe('DatabaseService', () => {
  let service: DatabaseService;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        DatabaseService,
        {
          provide: databaseConfig.KEY,  // Mock the injected config
          useValue: {
            host: 'test-host',
            port: 5432,
            username: 'test-user',
            password: 'test-password',
          },
        },
      ],
    }).compile();

    service = module.get<DatabaseService>(DatabaseService);
  });

  it('should use injected config', () => {
    const connection = service.getConnection();
    expect(connection.host).toBe('test-host');
  });
});
```

**When to Use @Inject() vs ConfigService:**

| Scenario | Use @Inject() | Use ConfigService |
|----------|--------------|-------------------|
| **Static config access** | ✅ Yes (type-safe) | ⚠️ Works (no types) |
| **Dynamic key access** | ❌ No | ✅ Yes |
| **Type safety needed** | ✅ Yes | ❌ No |
| **Autocomplete needed** | ✅ Yes | ❌ No |
| **Testing** | ✅ Easy to mock | ⚠️ Mock entire service |
| **Frequently accessed config** | ✅ Recommended | ⚠️ Works |
| **One-off config access** | ⚠️ Overkill | ✅ Better |

**Best Practices:**

```typescript
// ✅ GOOD: Use @Inject() for frequently accessed config
@Injectable()
export class DatabaseService {
  constructor(
    @Inject(databaseConfig.KEY)
    private dbConfig: ConfigType<typeof databaseConfig>,
  ) {}
}

// ✅ GOOD: Use ConfigService for dynamic access
@Injectable()
export class DynamicService {
  constructor(private configService: ConfigService) {}
  
  getValue(key: string) {
    return this.configService.get(key);
  }
}

// ✅ GOOD: Combine both when needed
@Injectable()
export class HybridService {
  constructor(
    @Inject(databaseConfig.KEY)
    private dbConfig: ConfigType<typeof databaseConfig>,
    private configService: ConfigService,
  ) {}
}

// ❌ BAD: String-based access when namespace available
@Injectable()
export class BadService {
  constructor(private configService: ConfigService) {}
  
  getHost() {
    return this.configService.get('database.host');  // No type safety
  }
}
```

**Interview Tip**: Use **`@Inject(config.KEY)`** to inject namespaced configuration with **type safety**. Requires **`registerAs()`** and **`ConfigType<typeof config>`** for types. Benefits: **TypeScript autocomplete**, **compile-time checking**, **easier testing**. Syntax: `@Inject(databaseConfig.KEY) private dbConfig: ConfigType<typeof databaseConfig>`. Access properties directly: `this.dbConfig.host`. **vs ConfigService**: @Inject() provides type safety, ConfigService is for dynamic keys. **Best practice**: use @Inject() for frequently accessed static config, ConfigService for dynamic access.

</details>

## Type Safety

### 26. How do you create TypeScript interfaces for configuration?

<details>
<summary>Answer</summary>

**TypeScript interfaces** for configuration provide **type safety**, **autocomplete**, and **documentation** for your configuration structure.

**Basic Interface Definition:**

```typescript
// config/types/database.interface.ts
export interface DatabaseConfig {
  type: 'postgres' | 'mysql' | 'mariadb';
  host: string;
  port: number;
  username: string;
  password: string;
  database: string;
  synchronize: boolean;
  logging: boolean;
}

// config/database.config.ts
import { registerAs } from '@nestjs/config';
import { DatabaseConfig } from './types/database.interface';

export default registerAs('database', (): DatabaseConfig => ({
  type: (process.env.DATABASE_TYPE as any) || 'postgres',
  host: process.env.DATABASE_HOST || 'localhost',
  port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
  username: process.env.DATABASE_USERNAME || '',
  password: process.env.DATABASE_PASSWORD || '',
  database: process.env.DATABASE_NAME || '',
  synchronize: process.env.DATABASE_SYNC === 'true',
  logging: process.env.DATABASE_LOGGING === 'true',
}));

// Usage with type safety
import { Inject, Injectable } from '@nestjs/common';
import { ConfigType } from '@nestjs/config';
import databaseConfig from './config/database.config';

@Injectable()
export class DatabaseService {
  constructor(
    @Inject(databaseConfig.KEY)
    private dbConfig: ConfigType<typeof databaseConfig>,
  ) {}

  getConnection() {
    // ✅ Full type safety and autocomplete
    const host: string = this.dbConfig.host;
    const port: number = this.dbConfig.port;
    
    return { host, port };
  }
}
```

**Complete Application Configuration Interface:**

```typescript
// config/types/app-config.interface.ts

// Database Configuration
export interface DatabaseConfig {
  type: 'postgres' | 'mysql' | 'mariadb' | 'sqlite';
  host: string;
  port: number;
  username: string;
  password: string;
  database: string;
  pool: {
    min: number;
    max: number;
    idleTimeout: number;
  };
  options: {
    ssl: boolean;
    synchronize: boolean;
    logging: boolean;
    migrations: string[];
    entities: string[];
  };
}

// JWT Configuration
export interface JwtConfig {
  access: {
    secret: string;
    expiresIn: string;
    algorithm: 'HS256' | 'HS384' | 'HS512';
  };
  refresh: {
    secret: string;
    expiresIn: string;
  };
  issuer: string;
  audience: string;
}

// Redis Configuration
export interface RedisConfig {
  host: string;
  port: number;
  password?: string;
  db: number;
  ttl: number;
  maxRetriesPerRequest: number;
}

// AWS Configuration
export interface AwsConfig {
  region: string;
  credentials: {
    accessKeyId: string;
    secretAccessKey: string;
  };
  s3: {
    bucket: string;
    endpoint?: string;
  };
}

// Application Configuration
export interface AppConfig {
  port: number;
  environment: 'development' | 'production' | 'test' | 'staging';
  apiPrefix: string;
  cors: {
    enabled: boolean;
    origins: string[];
    credentials: boolean;
    methods: string[];
  };
  rateLimit: {
    ttl: number;
    limit: number;
  };
  logging: {
    level: 'error' | 'warn' | 'info' | 'debug';
    format: 'json' | 'text';
    filePath?: string;
  };
  swagger: {
    enabled: boolean;
    path: string;
  };
}

// Complete Configuration Interface
export interface Configuration {
  app: AppConfig;
  database: DatabaseConfig;
  jwt: JwtConfig;
  redis: RedisConfig;
  aws: AwsConfig;
}
```

**Using Interfaces with Configuration Files:**

```typescript
// config/database.config.ts
import { registerAs } from '@nestjs/config';
import { DatabaseConfig } from './types/app-config.interface';

export default registerAs('database', (): DatabaseConfig => ({
  type: (process.env.DATABASE_TYPE as any) || 'postgres',
  host: process.env.DATABASE_HOST || 'localhost',
  port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
  username: process.env.DATABASE_USERNAME!,
  password: process.env.DATABASE_PASSWORD!,
  database: process.env.DATABASE_NAME!,
  pool: {
    min: parseInt(process.env.DATABASE_POOL_MIN, 10) || 2,
    max: parseInt(process.env.DATABASE_POOL_MAX, 10) || 10,
    idleTimeout: parseInt(process.env.DATABASE_IDLE_TIMEOUT, 10) || 30000,
  },
  options: {
    ssl: process.env.DATABASE_SSL === 'true',
    synchronize: process.env.DATABASE_SYNC === 'true',
    logging: process.env.DATABASE_LOGGING === 'true',
    migrations: ['dist/migrations/*.js'],
    entities: ['dist/**/*.entity.js'],
  },
}));

// config/jwt.config.ts
import { registerAs } from '@nestjs/config';
import { JwtConfig } from './types/app-config.interface';

export default registerAs('jwt', (): JwtConfig => ({
  access: {
    secret: process.env.JWT_SECRET || 'secret',
    expiresIn: process.env.JWT_EXPIRES_IN || '15m',
    algorithm: 'HS256',
  },
  refresh: {
    secret: process.env.JWT_REFRESH_SECRET || 'refresh-secret',
    expiresIn: process.env.JWT_REFRESH_EXPIRES_IN || '7d',
  },
  issuer: process.env.JWT_ISSUER || 'my-app',
  audience: process.env.JWT_AUDIENCE || 'my-app-users',
}));

// config/app.config.ts
import { registerAs } from '@nestjs/config';
import { AppConfig } from './types/app-config.interface';

export default registerAs('app', (): AppConfig => ({
  port: parseInt(process.env.PORT, 10) || 3000,
  environment: (process.env.NODE_ENV as any) || 'development',
  apiPrefix: process.env.API_PREFIX || 'api',
  cors: {
    enabled: process.env.CORS_ENABLED === 'true',
    origins: process.env.CORS_ORIGINS?.split(',') || ['*'],
    credentials: true,
    methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'OPTIONS'],
  },
  rateLimit: {
    ttl: parseInt(process.env.RATE_LIMIT_TTL, 10) || 60,
    limit: parseInt(process.env.RATE_LIMIT_MAX, 10) || 100,
  },
  logging: {
    level: (process.env.LOG_LEVEL as any) || 'info',
    format: (process.env.LOG_FORMAT as any) || 'json',
    filePath: process.env.LOG_FILE_PATH,
  },
  swagger: {
    enabled: process.env.SWAGGER_ENABLED !== 'false',
    path: process.env.SWAGGER_PATH || 'api-docs',
  },
}));
```

**Type-Safe Service Usage:**

```typescript
// database.service.ts
import { Injectable, Inject } from '@nestjs/common';
import { ConfigType } from '@nestjs/config';
import { createConnection, Connection } from 'typeorm';
import databaseConfig from './config/database.config';

@Injectable()
export class DatabaseService {
  constructor(
    @Inject(databaseConfig.KEY)
    private dbConfig: ConfigType<typeof databaseConfig>,
  ) {}

  async createConnection(): Promise<Connection> {
    // ✅ Full type safety - TypeScript knows all properties
    return createConnection({
      type: this.dbConfig.type,
      host: this.dbConfig.host,
      port: this.dbConfig.port,
      username: this.dbConfig.username,
      password: this.dbConfig.password,
      database: this.dbConfig.database,
      synchronize: this.dbConfig.options.synchronize,
      logging: this.dbConfig.options.logging,
      entities: this.dbConfig.options.entities,
      migrations: this.dbConfig.options.migrations,
      extra: {
        min: this.dbConfig.pool.min,
        max: this.dbConfig.pool.max,
        idleTimeoutMillis: this.dbConfig.pool.idleTimeout,
      },
    });
  }
}
```

**Generic Configuration Service:**

```typescript
// config/config.service.ts
import { Injectable } from '@nestjs/common';
import { ConfigService as NestConfigService } from '@nestjs/config';
import { Configuration } from './types/app-config.interface';

@Injectable()
export class TypedConfigService {
  constructor(private configService: NestConfigService) {}

  // Type-safe get for entire config sections
  get<K extends keyof Configuration>(key: K): Configuration[K] {
    return this.configService.get<Configuration[K]>(key)!;
  }

  // Type-safe get for nested properties
  getDatabase() {
    return this.configService.get<Configuration['database']>('database')!;
  }

  getJwt() {
    return this.configService.get<Configuration['jwt']>('jwt')!;
  }

  getApp() {
    return this.configService.get<Configuration['app']>('app')!;
  }
}

// Usage
@Injectable()
export class AppService {
  constructor(private config: TypedConfigService) {}

  getConfig() {
    const dbConfig = this.config.getDatabase();  // ✅ Type: DatabaseConfig
    const jwtConfig = this.config.getJwt();      // ✅ Type: JwtConfig
    const appConfig = this.config.getApp();      // ✅ Type: AppConfig
    
    return { dbConfig, jwtConfig, appConfig };
  }
}
```

**Readonly Configuration:**

```typescript
// Make configuration immutable
export interface DatabaseConfig {
  readonly type: 'postgres' | 'mysql' | 'mariadb';
  readonly host: string;
  readonly port: number;
  readonly username: string;
  readonly password: string;
  readonly pool: {
    readonly min: number;
    readonly max: number;
  };
}

// Or use Readonly utility type
export type DatabaseConfig = Readonly<{
  type: 'postgres' | 'mysql' | 'mariadb';
  host: string;
  port: number;
  pool: Readonly<{
    min: number;
    max: number;
  }>;
}>;
```

**Optional Properties:**

```typescript
// Use optional properties for non-required config
export interface RedisConfig {
  host: string;
  port: number;
  password?: string;        // Optional
  db?: number;              // Optional
  ttl: number;
  tls?: {                   // Optional nested object
    enabled: boolean;
    cert?: string;
    key?: string;
  };
}
```

**Union Types for Environment-Specific Config:**

```typescript
export type Environment = 'development' | 'production' | 'test' | 'staging';

export interface AppConfig {
  port: number;
  environment: Environment;
  debug: boolean;
  
  // Different types based on environment
  cache: Environment extends 'production'
    ? { enabled: true; ttl: number }
    : { enabled: false };
}
```

**Folder Structure:**

```
src/
├── config/
│   ├── types/
│   │   ├── app-config.interface.ts
│   │   ├── database.interface.ts
│   │   ├── jwt.interface.ts
│   │   └── index.ts
│   ├── app.config.ts
│   ├── database.config.ts
│   ├── jwt.config.ts
│   └── index.ts
├── app.module.ts
└── main.ts
```

**Exporting Types:**

```typescript
// config/types/index.ts
export * from './app-config.interface';
export * from './database.interface';
export * from './jwt.interface';

// Usage
import { DatabaseConfig, JwtConfig, AppConfig } from './config/types';
```

**Interview Tip**: Create **TypeScript interfaces** for configuration to provide **type safety** and **autocomplete**. Define interfaces in `config/types/` folder. Use with **`registerAs()`** return type: `registerAs('database', (): DatabaseConfig => ({...}))`. Access with **`ConfigType<typeof config>`** for full type safety. Interfaces should include: **required properties**, **optional with `?`**, **union types** for enums, **nested objects** for complex config. **Best practice**: one interface per config domain, use readonly for immutability, export from index file, document with JSDoc comments.

</details>

### 27. How do you use generics with `ConfigService.get<T>()`?

<details>
<summary>Answer</summary>

**Generics** in `ConfigService.get<T>()` provide **type safety** by specifying the expected return type, allowing TypeScript to enforce correct types and provide autocomplete.

**Basic Generic Usage:**

```typescript
@Injectable()
export class AppService {
  constructor(private configService: ConfigService) {}

  getConfig() {
    // ✅ With generic - type-safe
    const port = this.configService.get<number>('PORT');
    // port type: number | undefined
    
    const host = this.configService.get<string>('HOST');
    // host type: string | undefined
    
    const debug = this.configService.get<boolean>('DEBUG_MODE');
    // debug type: boolean | undefined
    
    // ❌ Without generic - any type
    const portAny = this.configService.get('PORT');
    // portAny type: any (no type safety!)
  }
}
```

**With Default Values:**

```typescript
@Injectable()
export class AppService {
  constructor(private configService: ConfigService) {}

  getConfig() {
    // Generic with default value
    const port = this.configService.get<number>('PORT', 3000);
    // port type: number (not undefined because of default)
    
    const host = this.configService.get<string>('HOST', 'localhost');
    // host type: string
    
    const timeout = this.configService.get<number>('TIMEOUT', 5000);
    // timeout type: number
  }
}
```

**Common Type Patterns:**

```typescript
@Injectable()
export class ConfigExamplesService {
  constructor(private configService: ConfigService) {}

  getTypes() {
    // 1. String
    const dbHost = this.configService.get<string>('DATABASE_HOST', 'localhost');
    
    // 2. Number
    const port = this.configService.get<number>('PORT', 3000);
    const timeout = this.configService.get<number>('TIMEOUT', 5000);
    
    // 3. Boolean
    const debug = this.configService.get<boolean>('DEBUG_MODE', false);
    const ssl = this.configService.get<boolean>('DATABASE_SSL', false);
    
    // 4. String array (from comma-separated)
    const origins = this.configService.get<string>('CORS_ORIGINS', '')
      .split(',')
      .filter(Boolean);
    
    // 5. Enum/Union type
    type Environment = 'development' | 'production' | 'test';
    const env = this.configService.get<Environment>('NODE_ENV', 'development');
    
    // 6. Object (nested config)
    const dbConfig = this.configService.get<{
      host: string;
      port: number;
      username: string;
    }>('database');
    
    return { dbHost, port, timeout, debug, ssl, origins, env, dbConfig };
  }
}
```

**Type-Safe Interface Usage:**

```typescript
// Define configuration interfaces
interface DatabaseConfig {
  host: string;
  port: number;
  username: string;
  password: string;
  pool: {
    min: number;
    max: number;
  };
}

interface JwtConfig {
  secret: string;
  expiresIn: string;
  algorithm: 'HS256' | 'HS384' | 'HS512';
}

@Injectable()
export class TypedConfigService {
  constructor(private configService: ConfigService) {}

  // Use interfaces with generics
  getDatabaseConfig(): DatabaseConfig {
    return this.configService.get<DatabaseConfig>('database')!;
  }

  getJwtConfig(): JwtConfig {
    return this.configService.get<JwtConfig>('jwt')!;
  }

  // Nested property access with type
  getDatabaseHost(): string {
    return this.configService.get<string>('database.host', 'localhost');
  }

  getJwtSecret(): string {
    return this.configService.get<string>('jwt.secret')!;
  }
}
```

**Generic with Complex Types:**

```typescript
interface CorsConfig {
  enabled: boolean;
  origins: string[];
  credentials: boolean;
  methods: string[];
}

interface RateLimitConfig {
  ttl: number;
  limit: number;
}

interface AppConfiguration {
  port: number;
  environment: string;
  cors: CorsConfig;
  rateLimit: RateLimitConfig;
}

@Injectable()
export class AppConfigService {
  constructor(private configService: ConfigService) {}

  // Get entire nested object with type
  getCorsConfig(): CorsConfig {
    return this.configService.get<CorsConfig>('app.cors')!;
  }

  getRateLimitConfig(): RateLimitConfig {
    return this.configService.get<RateLimitConfig>('app.rateLimit')!;
  }

  // Get complete app config
  getAppConfig(): AppConfiguration {
    return this.configService.get<AppConfiguration>('app')!;
  }

  // Individual properties with types
  getPort(): number {
    return this.configService.get<number>('app.port', 3000);
  }

  getCorsOrigins(): string[] {
    return this.configService.get<CorsConfig>('app.cors')!.origins;
  }
}
```

**Type Guards for Runtime Validation:**

```typescript
@Injectable()
export class SafeConfigService {
  constructor(private configService: ConfigService) {}

  // Helper to safely get number
  getNumber(key: string, defaultValue: number): number {
    const value = this.configService.get<number>(key, defaultValue);
    return typeof value === 'number' ? value : defaultValue;
  }

  // Helper to safely get string
  getString(key: string, defaultValue = ''): string {
    const value = this.configService.get<string>(key, defaultValue);
    return typeof value === 'string' ? value : defaultValue;
  }

  // Helper to safely get boolean
  getBoolean(key: string, defaultValue = false): boolean {
    const value = this.configService.get<string>(key);
    if (value === undefined) return defaultValue;
    return value === 'true' || value === '1';
  }

  // Helper to safely get array
  getArray(key: string, separator = ','): string[] {
    const value = this.configService.get<string>(key);
    return value ? value.split(separator).map(v => v.trim()) : [];
  }

  // Usage
  getConfig() {
    const port = this.getNumber('PORT', 3000);
    const host = this.getString('HOST', 'localhost');
    const debug = this.getBoolean('DEBUG_MODE', false);
    const origins = this.getArray('CORS_ORIGINS');
    
    return { port, host, debug, origins };
  }
}
```

**Generic with Literal Types:**

```typescript
@Injectable()
export class LiteralConfigService {
  constructor(private configService: ConfigService) {}

  getEnvironment(): 'development' | 'production' | 'test' {
    return this.configService.get<'development' | 'production' | 'test'>(
      'NODE_ENV',
      'development'
    );
  }

  getLogLevel(): 'error' | 'warn' | 'info' | 'debug' {
    return this.configService.get<'error' | 'warn' | 'info' | 'debug'>(
      'LOG_LEVEL',
      'info'
    );
  }

  getDatabaseType(): 'postgres' | 'mysql' | 'mariadb' {
    return this.configService.get<'postgres' | 'mysql' | 'mariadb'>(
      'DATABASE_TYPE',
      'postgres'
    );
  }
}
```

**Comparison: With vs Without Generics:**

```typescript
@Injectable()
export class ComparisonService {
  constructor(private configService: ConfigService) {}

  // ❌ WITHOUT GENERICS - No type safety
  withoutGenerics() {
    const port = this.configService.get('PORT');  // type: any
    const result = port + 1000;  // No error, but port might be string!
    const upper = port.toUpperCase();  // No error, but port might be number!
  }

  // ✅ WITH GENERICS - Type safe
  withGenerics() {
    const port = this.configService.get<number>('PORT', 3000);  // type: number
    const result = port + 1000;  // ✅ OK
    // const upper = port.toUpperCase();  // ❌ Compile error - number has no toUpperCase
  }
}
```

**Real-World Example:**

```typescript
interface EmailConfig {
  provider: 'sendgrid' | 'mailgun' | 'ses';
  apiKey: string;
  fromEmail: string;
  fromName: string;
  templates: {
    welcome: string;
    passwordReset: string;
    emailVerification: string;
  };
}

@Injectable()
export class EmailService {
  constructor(private configService: ConfigService) {}

  // Type-safe config access
  private getEmailConfig(): EmailConfig {
    return this.configService.get<EmailConfig>('email')!;
  }

  async sendWelcomeEmail(to: string) {
    const config = this.getEmailConfig();
    
    // ✅ TypeScript knows all properties
    return this.sendEmail({
      to,
      from: config.fromEmail,
      fromName: config.fromName,
      template: config.templates.welcome,
      apiKey: config.apiKey,
    });
  }

  private async sendEmail(options: any) {
    // Email sending logic
  }
}
```

**Generic Helper Methods:**

```typescript
@Injectable()
export class ConfigHelperService {
  constructor(private configService: ConfigService) {}

  // Generic helper with type parameter
  getRequired<T>(key: string): T {
    const value = this.configService.getOrThrow<T>(key);
    return value;
  }

  getOptional<T>(key: string, defaultValue: T): T {
    return this.configService.get<T>(key, defaultValue);
  }

  // Usage
  getConfig() {
    const port = this.getRequired<number>('PORT');
    const host = this.getOptional<string>('HOST', 'localhost');
    const debug = this.getOptional<boolean>('DEBUG_MODE', false);
    
    return { port, host, debug };
  }
}
```

**Interview Tip**: Use **generics** in `ConfigService.get<T>()` to specify **expected return type**: `get<number>('PORT')`, `get<string>('HOST')`, `get<boolean>('DEBUG')`. Provides **type safety** and **autocomplete**. Works with: **primitives** (string, number, boolean), **interfaces**, **union types**, **literal types**. Without generic: returns **any** (no type checking). With default value: `get<number>('PORT', 3000)` returns **non-undefined type**. **Best practice**: always use generics for type safety, define interfaces for complex config, use literal types for enums.

</details>

## Multiple Environments

### 28. How do you manage different environments (development, production, testing)?

<details>
<summary>Answer</summary>

**Different environments** (development, production, testing) are managed using **environment-specific configuration files**, **NODE_ENV variable**, and **conditional logic** in configuration.

**Approach 1: Multiple .env Files (Recommended):**

```bash
# Project structure
.
├── .env                    # Base config (committed as example)
├── .env.development        # Development overrides
├── .env.production         # Production overrides
├── .env.test               # Test overrides
├── .env.local              # Local overrides (not committed)
└── .gitignore

# .gitignore
.env.local
.env.*.local
```

**Base Configuration (.env):**

```bash
# .env - Base configuration with safe defaults
NODE_ENV=development
PORT=3000
API_PREFIX=api

# Database (defaults)
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_SYNC=false
DATABASE_LOGGING=false

# Logging
LOG_LEVEL=info
LOG_FORMAT=json
```

**Development Configuration (.env.development):**

```bash
# .env.development - Development-specific settings
NODE_ENV=development
PORT=3000

# Database (local development)
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_USERNAME=dev_user
DATABASE_PASSWORD=dev_password
DATABASE_NAME=myapp_dev
DATABASE_SYNC=true          # Auto-sync in development
DATABASE_LOGGING=true       # Enable logging in development

# Development tools
SWAGGER_ENABLED=true
DEBUG_MODE=true

# Logging (verbose in dev)
LOG_LEVEL=debug
LOG_FORMAT=text

# CORS (allow all in dev)
CORS_ORIGINS=http://localhost:3000,http://localhost:4200

# JWT (shorter expiry for testing)
JWT_SECRET=dev-secret-key-not-for-production
JWT_EXPIRES_IN=1h
```

**Production Configuration (.env.production):**

```bash
# .env.production - Production settings
# NOTE: In real production, use environment variables, not .env file!

NODE_ENV=production
PORT=8080

# Database (production - use secrets manager)
DATABASE_HOST=prod-db.example.com
DATABASE_PORT=5432
# DATABASE_USERNAME - Set via environment variable
# DATABASE_PASSWORD - Set via secrets manager
# DATABASE_NAME - Set via environment variable
DATABASE_SYNC=false         # NEVER sync in production
DATABASE_LOGGING=false      # Minimal logging

# Production features
SWAGGER_ENABLED=false       # Disable Swagger in prod
DEBUG_MODE=false

# Logging (minimal in prod)
LOG_LEVEL=error
LOG_FORMAT=json

# CORS (specific origins only)
CORS_ORIGINS=https://app.example.com,https://www.example.com

# JWT (longer expiry)
JWT_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d
```

**Test Configuration (.env.test):**

```bash
# .env.test - Testing settings
NODE_ENV=test
PORT=3001

# Database (test database)
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_USERNAME=test_user
DATABASE_PASSWORD=test_password
DATABASE_NAME=myapp_test
DATABASE_SYNC=true          # Sync for fresh test DB
DATABASE_LOGGING=false      # No logging in tests

# Test-specific
JWT_SECRET=test-secret-key
JWT_EXPIRES_IN=1h

# Disable external services in tests
SENDGRID_API_KEY=test-key
STRIPE_SECRET_KEY=test-key
```

**Loading Environment-Specific Files:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      
      // Load environment-specific file
      envFilePath: [
        `.env.${process.env.NODE_ENV}`,  // .env.development, .env.production, etc.
        '.env',                           // Fallback to base .env
      ],
      
      // Or more explicit:
      // envFilePath: process.env.NODE_ENV === 'production'
      //   ? '.env.production'
      //   : process.env.NODE_ENV === 'test'
      //   ? '.env.test'
      //   : '.env.development',
    }),
  ],
})
export class AppModule {}
```

**Approach 2: Conditional Configuration Logic:**

```typescript
// config/app.config.ts
import { registerAs } from '@nestjs/config';

const isDevelopment = process.env.NODE_ENV === 'development';
const isProduction = process.env.NODE_ENV === 'production';
const isTest = process.env.NODE_ENV === 'test';

export default registerAs('app', () => ({
  // Environment-specific port
  port: parseInt(process.env.PORT, 10) || (isDevelopment ? 3000 : 8080),
  
  environment: process.env.NODE_ENV || 'development',
  
  // Development-only features
  swagger: {
    enabled: isDevelopment || process.env.SWAGGER_ENABLED === 'true',
    path: 'api-docs',
  },
  
  // Logging configuration
  logging: {
    level: isDevelopment ? 'debug' : isTest ? 'warn' : 'error',
    format: isDevelopment ? 'text' : 'json',
    pretty: isDevelopment,
  },
  
  // Cache settings
  cache: {
    enabled: isProduction,
    ttl: isDevelopment ? 60 : 3600,  // 1 min dev, 1 hour prod
  },
  
  // Security settings
  cors: {
    enabled: true,
    origins: isDevelopment 
      ? ['http://localhost:3000', 'http://localhost:4200']
      : process.env.CORS_ORIGINS?.split(',') || [],
    credentials: true,
  },
  
  // Rate limiting
  rateLimit: {
    enabled: isProduction,
    ttl: isDevelopment ? 60 : 15,
    limit: isDevelopment ? 1000 : 100,
  },
}));
```

**Environment-Specific Validation:**

```typescript
// app.module.ts
import * as Joi from 'joi';

const isProduction = process.env.NODE_ENV === 'production';

@Module({
  imports: [
    ConfigModule.forRoot({
      validationSchema: Joi.object({
        NODE_ENV: Joi.string()
          .valid('development', 'production', 'test', 'staging')
          .default('development'),
        
        PORT: Joi.number().default(3000),
        
        // Required in production, optional in development
        DATABASE_PASSWORD: isProduction
          ? Joi.string().required()
          : Joi.string().optional(),
        
        JWT_SECRET: isProduction
          ? Joi.string().required().min(32)
          : Joi.string().default('dev-secret'),
        
        // Production-only requirements
        SSL_CERT: isProduction
          ? Joi.string().required()
          : Joi.string().optional(),
      }),
    }),
  ],
})
export class AppModule {}
```

**Running Different Environments:**

```json
// package.json
{
  "scripts": {
    "start": "NODE_ENV=development nest start",
    "start:dev": "NODE_ENV=development nest start --watch",
    "start:debug": "NODE_ENV=development nest start --debug --watch",
    "start:prod": "NODE_ENV=production node dist/main",
    "test": "NODE_ENV=test jest",
    "test:e2e": "NODE_ENV=test jest --config ./test/jest-e2e.json"
  }
}

// Windows (PowerShell)
{
  "scripts": {
    "start:dev": "set NODE_ENV=development && nest start --watch",
    "start:prod": "set NODE_ENV=production && node dist/main",
    "test": "set NODE_ENV=test && jest"
  }
}

// Cross-platform (using cross-env)
{
  "scripts": {
    "start:dev": "cross-env NODE_ENV=development nest start --watch",
    "start:prod": "cross-env NODE_ENV=production node dist/main",
    "test": "cross-env NODE_ENV=test jest"
  }
}
```

**Environment Detection Service:**

```typescript
// config/environment.service.ts
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class EnvironmentService {
  constructor(private configService: ConfigService) {}

  get isDevelopment(): boolean {
    return this.configService.get('NODE_ENV') === 'development';
  }

  get isProduction(): boolean {
    return this.configService.get('NODE_ENV') === 'production';
  }

  get isTest(): boolean {
    return this.configService.get('NODE_ENV') === 'test';
  }

  get isStaging(): boolean {
    return this.configService.get('NODE_ENV') === 'staging';
  }

  get environment(): string {
    return this.configService.get('NODE_ENV', 'development');
  }
}

// Usage
@Injectable()
export class AppService {
  constructor(private env: EnvironmentService) {}

  getFeatures() {
    return {
      swagger: this.env.isDevelopment,
      debugging: this.env.isDevelopment,
      caching: this.env.isProduction,
      rateLimit: this.env.isProduction,
    };
  }
}
```

**Docker Environment Variables:**

```yaml
# docker-compose.yml
version: '3.8'

services:
  app-dev:
    build: .
    environment:
      NODE_ENV: development
      DATABASE_HOST: postgres-dev
      DATABASE_PASSWORD: dev_password
    env_file:
      - .env.development
  
  app-prod:
    build: .
    environment:
      NODE_ENV: production
      DATABASE_HOST: postgres-prod
      # Use secrets for sensitive data
    secrets:
      - database_password
      - jwt_secret

secrets:
  database_password:
    external: true
  jwt_secret:
    external: true
```

**Best Practices:**

```typescript
// ✅ GOOD: Separate files per environment
.env.development
.env.production
.env.test

// ✅ GOOD: Environment detection
const isProd = process.env.NODE_ENV === 'production';

// ✅ GOOD: Different validation per environment
DATABASE_PASSWORD: isProd ? Joi.required() : Joi.optional()

// ✅ GOOD: Safe defaults for development
PORT: 3000
DEBUG_MODE: true

// ❌ BAD: Real secrets in development config
.env.development: JWT_SECRET=production-secret  // ❌

// ❌ BAD: Same config for all environments
DATABASE_SYNC=true  // ❌ Never in production!

// ❌ BAD: No environment detection
// Always behaves the same way regardless of environment
```

**Interview Tip**: Manage environments using **multiple .env files** (`.env.development`, `.env.production`, `.env.test`) loaded based on **NODE_ENV**. Use **`envFilePath` array** to specify environment-specific files. Add **conditional logic** in config files based on `process.env.NODE_ENV`. **Development**: verbose logging, Swagger enabled, auto-sync. **Production**: minimal logging, no Swagger, strict validation. **Test**: isolated database, fast tests. Set NODE_ENV in **package.json scripts**. **Best practice**: never commit real secrets, different validation per environment, use secrets manager in production.

</details>

### 29. How do you load different `.env` files per environment?

<details>
<summary>Answer</summary>

**Different `.env` files** are loaded per environment using the **`envFilePath` option** in `ConfigModule.forRoot()`, typically based on the **`NODE_ENV`** environment variable.

**Basic Setup:**

```bash
# Create environment-specific files
.
├── .env                  # Base/default config
├── .env.development      # Development overrides
├── .env.production       # Production overrides
├── .env.test             # Test overrides
├── .env.staging          # Staging overrides (optional)
└── .env.local            # Local overrides (not committed)
```

**Method 1: Array of envFilePaths (Recommended):**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      
      // Load files in order (later files override earlier ones)
      envFilePath: [
        `.env.${process.env.NODE_ENV}`,  // Environment-specific file
        '.env',                           // Base configuration
      ],
    }),
  ],
})
export class AppModule {}

// With NODE_ENV=development:
// 1. Loads .env.development
// 2. Loads .env (for missing variables)
// Result: .env.development overrides .env
```

**Method 2: Conditional envFilePath:**

```typescript
// app.module.ts
const environment = process.env.NODE_ENV || 'development';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: `.env.${environment}`,  // Load specific file
    }),
  ],
})
export class AppModule {}

// NODE_ENV=production → loads .env.production
// NODE_ENV=development → loads .env.development
// NODE_ENV=test → loads .env.test
```

**Method 3: Multiple Files with Fallbacks:**

```typescript
// app.module.ts
@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      
      // Load order (last has highest priority):
      envFilePath: [
        '.env',                              // 1. Base (lowest priority)
        `.env.${process.env.NODE_ENV}`,     // 2. Environment-specific
        '.env.local',                        // 3. Local overrides (highest priority)
      ],
    }),
  ],
})
export class AppModule {}

// Priority: .env.local > .env.{NODE_ENV} > .env
```

**Complete Example with All Environments:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';

function getEnvFilePath(): string[] {
  const env = process.env.NODE_ENV || 'development';
  
  return [
    '.env',                    // Base configuration
    `.env.${env}`,            // Environment-specific
    `.env.${env}.local`,      // Environment-specific local overrides
    '.env.local',             // General local overrides
  ].filter(file => {
    // Optional: Check if file exists
    try {
      require('fs').accessSync(file);
      return true;
    } catch {
      return false;
    }
  });
}

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: getEnvFilePath(),
    }),
  ],
})
export class AppModule {}
```

**Environment Files Content:**

```bash
# .env (Base - committed)
PORT=3000
API_PREFIX=api
DATABASE_HOST=localhost
DATABASE_PORT=5432
```

```bash
# .env.development (Development - committed)
NODE_ENV=development
PORT=3000
DATABASE_NAME=myapp_dev
DATABASE_SYNC=true
DATABASE_LOGGING=true
DEBUG_MODE=true
SWAGGER_ENABLED=true
LOG_LEVEL=debug
```

```bash
# .env.production (Production - committed as template)
NODE_ENV=production
PORT=8080
DATABASE_NAME=myapp_prod
DATABASE_SYNC=false
DATABASE_LOGGING=false
DEBUG_MODE=false
SWAGGER_ENABLED=false
LOG_LEVEL=error
# Note: Real secrets should be set via environment variables
```

```bash
# .env.test (Test - committed)
NODE_ENV=test
PORT=3001
DATABASE_NAME=myapp_test
DATABASE_SYNC=true
DATABASE_LOGGING=false
JWT_SECRET=test-secret-key
```

```bash
# .env.local (Local overrides - NOT committed)
DATABASE_USERNAME=my_local_user
DATABASE_PASSWORD=my_local_password
JWT_SECRET=my-local-jwt-secret
# Developer-specific overrides
```

**Package.json Scripts:**

```json
// package.json
{
  "scripts": {
    // Development
    "start:dev": "NODE_ENV=development nest start --watch",
    
    // Production
    "build": "nest build",
    "start:prod": "NODE_ENV=production node dist/main",
    
    // Testing
    "test": "NODE_ENV=test jest",
    "test:watch": "NODE_ENV=test jest --watch",
    "test:e2e": "NODE_ENV=test jest --config ./test/jest-e2e.json",
    
    // Staging (optional)
    "start:staging": "NODE_ENV=staging node dist/main"
  }
}
```

**Cross-Platform Support:**

```bash
# Install cross-env for Windows compatibility
npm install --save-dev cross-env
```

```json
// package.json (cross-platform)
{
  "scripts": {
    "start:dev": "cross-env NODE_ENV=development nest start --watch",
    "start:prod": "cross-env NODE_ENV=production node dist/main",
    "test": "cross-env NODE_ENV=test jest"
  }
}
```

**Verifying Loaded Configuration:**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { ConfigService } from '@nestjs/config';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  const configService = app.get(ConfigService);
  
  // Log loaded configuration (for debugging)
  console.log('Environment:', configService.get('NODE_ENV'));
  console.log('Port:', configService.get('PORT'));
  console.log('Database:', configService.get('DATABASE_NAME'));
  console.log('Debug Mode:', configService.get('DEBUG_MODE'));
  
  const port = configService.get('PORT', 3000);
  await app.listen(port);
  
  console.log(`Application running on port ${port}`);
}
bootstrap();
```

**Git Configuration:**

```bash
# .gitignore

# Don't commit files with real secrets
.env.local
.env.*.local
.env.production.local

# DO commit these (as templates)
# .env
# .env.development
# .env.production (without real secrets)
# .env.test
```

**Docker Integration:**

```dockerfile
# Dockerfile
FROM node:18-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./
RUN npm ci --only=production

# Copy application
COPY . .
RUN npm run build

# Set environment
ENV NODE_ENV=production

# Run application
CMD ["node", "dist/main"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  app-dev:
    build: .
    environment:
      NODE_ENV: development
    env_file:
      - .env.development
    ports:
      - "3000:3000"
  
  app-prod:
    build: .
    environment:
      NODE_ENV: production
    env_file:
      - .env.production
    ports:
      - "8080:8080"
```

**Priority Order:**

```typescript
// When loading multiple files:
envFilePath: ['.env', '.env.development', '.env.local']

// Priority (highest to lowest):
// 1. .env.local (highest - overrides everything)
// 2. .env.development (middle - overrides base)
// 3. .env (lowest - base defaults)

// If same variable in multiple files:
// PORT=3000 in .env
// PORT=4000 in .env.development
// PORT=5000 in .env.local
// Result: PORT=5000 (from .env.local)
```

**Best Practices:**

```typescript
// ✅ GOOD: Multiple files with fallbacks
envFilePath: ['.env', `.env.${process.env.NODE_ENV}`, '.env.local']

// ✅ GOOD: Commit template files
.env.development (committed)
.env.production (committed, no real secrets)
.env.test (committed)

// ✅ GOOD: Ignore local overrides
.env.local (in .gitignore)

// ✅ GOOD: Document in README
// "Copy .env.example to .env.local and fill in your values"

// ❌ BAD: Single file for all environments
envFilePath: '.env'  // No environment-specific config

// ❌ BAD: Commit real secrets
.env.production with real DATABASE_PASSWORD  // ❌

// ❌ BAD: No fallback
envFilePath: `.env.${process.env.NODE_ENV}`  // Fails if NODE_ENV not set
```

**Interview Tip**: Load different `.env` files using **`envFilePath` array** in ConfigModule: `['.env', '.env.${NODE_ENV}', '.env.local']`. Later files **override earlier** ones. Set **NODE_ENV** in package.json scripts: `"start:dev": "NODE_ENV=development nest start"`. Common files: **`.env`** (base), **`.env.development`** (dev), **`.env.production`** (prod template), **`.env.test`** (tests), **`.env.local`** (local overrides, not committed). Use **cross-env** for Windows. **Best practice**: commit templates without real secrets, use .env.local for local overrides, document in README.

</details>

### 30. What is `NODE_ENV` and how do you use it?

<details>
<summary>Answer</summary>

**`NODE_ENV`** is a **standard environment variable** used to specify the **runtime environment** of a Node.js application (development, production, test, etc.). It's the primary way to configure environment-specific behavior.

**Common NODE_ENV Values:**

```bash
# Standard values
NODE_ENV=development  # Local development
NODE_ENV=production   # Production deployment
NODE_ENV=test         # Running tests
NODE_ENV=staging      # Staging environment (optional)
```

**Setting NODE_ENV:**

```bash
# Method 1: Command line (Unix/Mac)
NODE_ENV=production node dist/main.js

# Method 2: Command line (Windows CMD)
set NODE_ENV=production && node dist/main.js

# Method 3: Command line (Windows PowerShell)
$env:NODE_ENV="production"; node dist/main.js

# Method 4: Cross-platform (using cross-env)
cross-env NODE_ENV=production node dist/main.js

# Method 5: In .env file
NODE_ENV=development

# Method 6: System environment variable
export NODE_ENV=production  # Unix/Mac
setx NODE_ENV production    # Windows (persistent)
```

**Package.json Scripts:**

```json
// package.json
{
  "scripts": {
    // Development
    "start": "nest start",
    "start:dev": "cross-env NODE_ENV=development nest start --watch",
    "start:debug": "cross-env NODE_ENV=development nest start --debug --watch",
    
    // Production
    "build": "nest build",
    "start:prod": "cross-env NODE_ENV=production node dist/main",
    
    // Testing
    "test": "cross-env NODE_ENV=test jest",
    "test:watch": "cross-env NODE_ENV=test jest --watch",
    "test:cov": "cross-env NODE_ENV=test jest --coverage",
    "test:e2e": "cross-env NODE_ENV=test jest --config ./test/jest-e2e.json",
    
    // Staging
    "start:staging": "cross-env NODE_ENV=staging node dist/main"
  }
}
```

**Using NODE_ENV in Configuration:**

```typescript
// config/app.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('app', () => {
  const nodeEnv = process.env.NODE_ENV || 'development';
  const isDevelopment = nodeEnv === 'development';
  const isProduction = nodeEnv === 'production';
  const isTest = nodeEnv === 'test';
  
  return {
    environment: nodeEnv,
    
    // Port based on environment
    port: parseInt(process.env.PORT, 10) || (isDevelopment ? 3000 : 8080),
    
    // Logging configuration
    logging: {
      level: isDevelopment ? 'debug' : isProduction ? 'error' : 'warn',
      format: isDevelopment ? 'text' : 'json',
      pretty: isDevelopment,
    },
    
    // Debug mode
    debug: isDevelopment,
    
    // Swagger documentation
    swagger: {
      enabled: isDevelopment || process.env.SWAGGER_ENABLED === 'true',
      path: 'api-docs',
    },
    
    // CORS settings
    cors: {
      enabled: true,
      origins: isDevelopment 
        ? ['http://localhost:3000', 'http://localhost:4200']
        : process.env.CORS_ORIGINS?.split(',') || [],
    },
    
    // Cache settings
    cache: {
      enabled: isProduction,
      ttl: isDevelopment ? 60 : 3600,
    },
    
    // Rate limiting
    rateLimit: {
      enabled: isProduction,
      ttl: 60,
      limit: isDevelopment ? 1000 : 100,
    },
  };
});
```

**Loading Environment-Specific .env Files:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      
      // Load .env file based on NODE_ENV
      envFilePath: [
        `.env.${process.env.NODE_ENV}`,  // e.g., .env.production
        '.env',                           // Fallback
      ],
    }),
  ],
})
export class AppModule {}
```

**Environment Detection Service:**

```typescript
// config/environment.service.ts
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class EnvironmentService {
  constructor(private configService: ConfigService) {}

  get nodeEnv(): string {
    return this.configService.get<string>('NODE_ENV', 'development');
  }

  get isDevelopment(): boolean {
    return this.nodeEnv === 'development';
  }

  get isProduction(): boolean {
    return this.nodeEnv === 'production';
  }

  get isTest(): boolean {
    return this.nodeEnv === 'test';
  }

  get isStaging(): boolean {
    return this.nodeEnv === 'staging';
  }

  // Helper method for environment-specific logic
  getValueByEnvironment<T>(values: {
    development?: T;
    production?: T;
    test?: T;
    staging?: T;
    default: T;
  }): T {
    return values[this.nodeEnv] ?? values.default;
  }
}

// Usage
@Injectable()
export class AppService {
  constructor(private env: EnvironmentService) {}

  getConfig() {
    // Simple checks
    if (this.env.isDevelopment) {
      console.log('Running in development mode');
    }
    
    // Environment-specific values
    const timeout = this.env.getValueByEnvironment({
      development: 30000,
      production: 5000,
      test: 1000,
      default: 10000,
    });
    
    return { timeout };
  }
}
```

**Conditional Middleware Based on NODE_ENV:**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  const nodeEnv = process.env.NODE_ENV || 'development';
  
  // Swagger only in development
  if (nodeEnv === 'development' || process.env.SWAGGER_ENABLED === 'true') {
    const config = new DocumentBuilder()
      .setTitle('API Documentation')
      .setVersion('1.0')
      .build();
    const document = SwaggerModule.createDocument(app, config);
    SwaggerModule.setup('api-docs', app, document);
    console.log('Swagger enabled at /api-docs');
  }
  
  // Development-specific middleware
  if (nodeEnv === 'development') {
    app.enableCors({ origin: '*' });
    console.log('CORS enabled for all origins (development)');
  } else {
    // Production CORS
    app.enableCors({
      origin: process.env.CORS_ORIGINS?.split(',') || [],
      credentials: true,
    });
  }
  
  // Production optimizations
  if (nodeEnv === 'production') {
    app.enableShutdownHooks();
    // Enable compression, helmet, etc.
  }
  
  const port = process.env.PORT || 3000;
  await app.listen(port);
  
  console.log(`Application running in ${nodeEnv} mode on port ${port}`);
}
bootstrap();
```

**Database Configuration Based on NODE_ENV:**

```typescript
// config/database.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('database', () => {
  const isProduction = process.env.NODE_ENV === 'production';
  const isDevelopment = process.env.NODE_ENV === 'development';
  const isTest = process.env.NODE_ENV === 'test';
  
  return {
    type: 'postgres',
    host: process.env.DATABASE_HOST || 'localhost',
    port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
    username: process.env.DATABASE_USERNAME,
    password: process.env.DATABASE_PASSWORD,
    database: process.env.DATABASE_NAME,
    
    // NEVER synchronize in production!
    synchronize: isDevelopment || isTest,
    
    // Verbose logging in development
    logging: isDevelopment ? 'all' : false,
    
    // SSL in production
    ssl: isProduction ? { rejectUnauthorized: false } : false,
    
    // Connection pool
    extra: {
      max: isProduction ? 20 : 5,
      min: isProduction ? 5 : 1,
    },
  };
});
```

**Joi Validation with NODE_ENV:**

```typescript
// app.module.ts
import * as Joi from 'joi';

@Module({
  imports: [
    ConfigModule.forRoot({
      validationSchema: Joi.object({
        // Validate NODE_ENV
        NODE_ENV: Joi.string()
          .valid('development', 'production', 'test', 'staging')
          .default('development'),
        
        // Different validation based on NODE_ENV
        DATABASE_PASSWORD: process.env.NODE_ENV === 'production'
          ? Joi.string().required().min(12)
          : Joi.string().default('dev-password'),
        
        JWT_SECRET: process.env.NODE_ENV === 'production'
          ? Joi.string().required().min(32)
          : Joi.string().default('dev-secret'),
      }),
    }),
  ],
})
export class AppModule {}
```

**TypeORM Integration:**

```typescript
// app.module.ts
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (configService: ConfigService) => {
        const nodeEnv = configService.get('NODE_ENV');
        
        return {
          type: 'postgres',
          host: configService.get('DATABASE_HOST'),
          port: configService.get('DATABASE_PORT'),
          username: configService.get('DATABASE_USERNAME'),
          password: configService.get('DATABASE_PASSWORD'),
          database: configService.get('DATABASE_NAME'),
          
          // Environment-specific settings
          synchronize: nodeEnv !== 'production',  // Only in dev/test
          logging: nodeEnv === 'development',
          ssl: nodeEnv === 'production',
          
          entities: ['dist/**/*.entity.js'],
          migrations: ['dist/migrations/*.js'],
        };
      },
    }),
  ],
})
export class AppModule {}
```

**Docker and NODE_ENV:**

```dockerfile
# Dockerfile
FROM node:18-alpine AS development
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
ENV NODE_ENV=development
CMD ["npm", "run", "start:dev"]

FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build
ENV NODE_ENV=production

FROM node:18-alpine AS production
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
COPY package*.json ./
ENV NODE_ENV=production
CMD ["node", "dist/main"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  app-dev:
    build:
      context: .
      target: development
    environment:
      NODE_ENV: development
    ports:
      - "3000:3000"
  
  app-prod:
    build:
      context: .
      target: production
    environment:
      NODE_ENV: production
    ports:
      - "8080:8080"
```

**Best Practices:**

```typescript
// ✅ GOOD: Check NODE_ENV for environment-specific logic
if (process.env.NODE_ENV === 'production') {
  // Production-only code
}

// ✅ GOOD: Default to development
const env = process.env.NODE_ENV || 'development';

// ✅ GOOD: Use strict equality
if (nodeEnv === 'production') { }

// ✅ GOOD: Validate NODE_ENV values
NODE_ENV: Joi.string().valid('development', 'production', 'test')

// ❌ BAD: Truthy check (NODE_ENV could be any string)
if (process.env.NODE_ENV) { }  // Always true if set

// ❌ BAD: Case-sensitive issues
if (process.env.NODE_ENV === 'Production') { }  // Won't match 'production'

// ❌ BAD: No default value
const env = process.env.NODE_ENV;  // Could be undefined

// ❌ BAD: Dangerous defaults in production
synchronize: process.env.DATABASE_SYNC || true  // Never!
```

**Common Patterns:**

```typescript
// Pattern 1: Environment-specific imports
const logger = process.env.NODE_ENV === 'production'
  ? require('./prod-logger')
  : require('./dev-logger');

// Pattern 2: Conditional feature flags
const features = {
  swagger: process.env.NODE_ENV !== 'production',
  debug: process.env.NODE_ENV === 'development',
  caching: process.env.NODE_ENV === 'production',
};

// Pattern 3: Environment-based timeouts
const timeout = {
  development: 30000,
  production: 5000,
  test: 1000,
}[process.env.NODE_ENV || 'development'];

// Pattern 4: Type-safe environment check
type Environment = 'development' | 'production' | 'test' | 'staging';
const env = (process.env.NODE_ENV || 'development') as Environment;
```

**Interview Tip**: **`NODE_ENV`** is a standard environment variable specifying the **runtime environment** (development, production, test, staging). Set in **package.json scripts**: `"start:prod": "NODE_ENV=production node dist/main"`. Use to: **load environment-specific .env files** (`.env.${NODE_ENV}`), **enable/disable features** (Swagger, debug mode), **configure logging levels**, **optimize for production**. Common values: **development** (local dev), **production** (live), **test** (testing). **Best practice**: always validate with Joi, provide default value, use cross-env for Windows compatibility, never synchronize database in production.

</details>

## Best Practices

### 31. Should you use ConfigService or process.env directly?

<details>
<summary>Answer</summary>

**Use ConfigService** over `process.env` directly in most cases. ConfigService provides **type safety**, **validation**, **testing**, and **centralization** that `process.env` lacks.

**ConfigService vs process.env Comparison:**

| Feature | ConfigService | process.env |
|---------|--------------|-------------|
| **Type Safety** | ✅ Yes (with generics) | ❌ No (always string \| undefined) |
| **Default Values** | ✅ Built-in `get(key, default)` | ❌ Manual `\|\|` operator |
| **Validation** | ✅ Joi/class-validator | ❌ None |
| **Required Values** | ✅ `getOrThrow()` | ❌ Manual checks |
| **Testability** | ✅ Easy to mock | ❌ Hard to mock |
| **Centralized** | ✅ Single source | ❌ Scattered access |
| **Nested Config** | ✅ Dot notation | ❌ Flat keys only |
| **Documentation** | ✅ Self-documenting | ❌ No structure |
| **Custom Loaders** | ✅ Supported | ❌ Not supported |
| **Caching** | ✅ Available | ❌ No caching |

**When to Use ConfigService:**

```typescript
// ✅ GOOD: Use ConfigService in services/controllers
@Injectable()
export class UserService {
  constructor(private configService: ConfigService) {}

  async sendEmail(email: string) {
    // Type-safe, validated, testable
    const apiKey = this.configService.getOrThrow<string>('SENDGRID_API_KEY');
    const fromEmail = this.configService.get<string>('FROM_EMAIL', 'noreply@app.com');
    
    return this.sendgridClient.send({
      to: email,
      from: fromEmail,
      apiKey,
    });
  }
}

// ✅ GOOD: Access nested configuration
const dbHost = this.configService.get<string>('database.host', 'localhost');
const jwtSecret = this.configService.get<string>('jwt.secret');

// ✅ GOOD: Type safety with generics
const port = this.configService.get<number>('PORT', 3000);
const debug = this.configService.get<boolean>('DEBUG_MODE', false);
```

**When process.env is Acceptable:**

```typescript
// ✅ ACCEPTABLE: In configuration files (before ConfigService is available)
// config/database.config.ts
export default () => ({
  host: process.env.DATABASE_HOST || 'localhost',
  port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
  username: process.env.DATABASE_USERNAME,
});

// ✅ ACCEPTABLE: In main.ts (before app is created)
// main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // ConfigService not available yet
  const port = parseInt(process.env.PORT, 10) || 3000;
  await app.listen(port);
}

// ✅ ACCEPTABLE: For NODE_ENV checks
if (process.env.NODE_ENV === 'production') {
  // Environment detection
}

// ✅ ACCEPTABLE: In module imports (before DI is ready)
@Module({
  imports: [
    TypeOrmModule.forRoot({
      host: process.env.DATABASE_HOST || 'localhost',
    }),
  ],
})
```

**Problems with process.env:**

```typescript
// ❌ BAD: No type safety
@Injectable()
export class BadService {
  getPort() {
    const port = process.env.PORT;  // Type: string | undefined
    return port + 1000;  // Runtime error if PORT is undefined!
  }
}

// ❌ BAD: No validation
const port = process.env.PORT;  // Could be "abc" - no validation

// ❌ BAD: Hard to test
class MyService {
  getApiKey() {
    return process.env.API_KEY;  // How do you mock this?
  }
}

// ❌ BAD: No default values (must handle manually)
const timeout = process.env.TIMEOUT || '5000';  // Manual default
const timeoutNum = parseInt(timeout, 10);       // Manual parsing

// ❌ BAD: Scattered throughout codebase
// file1.ts: process.env.DATABASE_HOST
// file2.ts: process.env.DATABASE_HOST
// file3.ts: process.env.DATABASE_HOST
// Hard to track where config is used
```

**Best Practice: Use ConfigService Everywhere:**

```typescript
// ✅ RECOMMENDED PATTERN

// 1. Define configuration in config files (process.env OK here)
// config/app.config.ts
export default registerAs('app', () => ({
  port: parseInt(process.env.PORT, 10) || 3000,
  apiKey: process.env.API_KEY,
}));

// 2. Use ConfigService in application code
@Injectable()
export class AppService {
  constructor(private configService: ConfigService) {}

  getPort(): number {
    return this.configService.get<number>('app.port', 3000);
  }

  getApiKey(): string {
    return this.configService.getOrThrow<string>('app.apiKey');
  }
}

// 3. Easy to test
describe('AppService', () => {
  it('should get port', () => {
    const mockConfig = {
      get: jest.fn().mockReturnValue(4000),
    };
    
    const service = new AppService(mockConfig as any);
    expect(service.getPort()).toBe(4000);
  });
});
```

**Migration from process.env to ConfigService:**

```typescript
// Before (using process.env)
@Injectable()
export class OldService {
  sendEmail() {
    const apiKey = process.env.SENDGRID_API_KEY;  // ❌
    const from = process.env.FROM_EMAIL || 'noreply@app.com';  // ❌
    
    if (!apiKey) {
      throw new Error('API key missing');
    }
    
    return this.client.send({ apiKey, from });
  }
}

// After (using ConfigService)
@Injectable()
export class NewService {
  constructor(private configService: ConfigService) {}

  sendEmail() {
    const apiKey = this.configService.getOrThrow<string>('SENDGRID_API_KEY');  // ✅
    const from = this.configService.get<string>('FROM_EMAIL', 'noreply@app.com');  // ✅
    
    return this.client.send({ apiKey, from });
  }
}
```

**Exception: Very Simple Scripts:**

```typescript
// For standalone scripts/utilities, process.env might be OK
// scripts/seed-database.ts
const dbHost = process.env.DATABASE_HOST || 'localhost';
const dbPort = parseInt(process.env.DATABASE_PORT, 10) || 5432;

// But even here, ConfigService is better if you're using NestJS
```

**Interview Tip**: **Use ConfigService** over `process.env` in application code for: **type safety** (generics), **validation** (Joi), **default values** (`get(key, default)`), **required checks** (`getOrThrow()`), **testability** (easy mocking), **nested config** (dot notation). **process.env acceptable** in: **config files** (before ConfigService), **main.ts** (before app creation), **NODE_ENV checks**. **Best practice**: ConfigService everywhere in services/controllers, process.env only in config files and main.ts.

</details>

### 32. Where should you validate configuration?

<details>
<summary>Answer</summary>

**Validate configuration at startup** (application bootstrap) to **fail fast** if configuration is invalid. This prevents runtime errors and ensures the application never starts with bad configuration.

**Validation Locations:**

```
1. Startup Validation (RECOMMENDED) ✅
   - ConfigModule.forRoot({ validationSchema })
   - Validates before app starts
   - Fails immediately if invalid
   
2. Runtime Validation (AVOID) ❌
   - Validating when accessing values
   - App starts with bad config
   - Errors occur during requests
```

**Recommended: Joi Validation at Startup:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import * as Joi from 'joi';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      
      // ✅ BEST: Validate at startup
      validationSchema: Joi.object({
        // Node environment
        NODE_ENV: Joi.string()
          .valid('development', 'production', 'test', 'staging')
          .default('development'),
        
        // Server
        PORT: Joi.number().port().default(3000),
        HOST: Joi.string().default('localhost'),
        
        // Database
        DATABASE_HOST: Joi.string().required(),
        DATABASE_PORT: Joi.number().port().default(5432),
        DATABASE_USERNAME: Joi.string().required(),
        DATABASE_PASSWORD: Joi.string().required().min(8),
        DATABASE_NAME: Joi.string().required(),
        
        // JWT
        JWT_SECRET: Joi.string().required().min(32),
        JWT_EXPIRATION: Joi.string().default('1h'),
        
        // Email
        SENDGRID_API_KEY: Joi.string().required(),
        FROM_EMAIL: Joi.string().email().required(),
        
        // Redis
        REDIS_HOST: Joi.string().default('localhost'),
        REDIS_PORT: Joi.number().port().default(6379),
        
        // AWS
        AWS_REGION: Joi.string().default('us-east-1'),
        AWS_ACCESS_KEY_ID: Joi.when('NODE_ENV', {
          is: 'production',
          then: Joi.string().required(),
          otherwise: Joi.string().optional(),
        }),
        AWS_SECRET_ACCESS_KEY: Joi.when('NODE_ENV', {
          is: 'production',
          then: Joi.string().required(),
          otherwise: Joi.string().optional(),
        }),
        
        // Features
        ENABLE_SWAGGER: Joi.boolean().default(false),
        ENABLE_CACHING: Joi.boolean().default(true),
      }),
      
      // Validation options
      validationOptions: {
        abortEarly: false,  // Show all errors
        allowUnknown: true,  // Allow extra env vars
      },
    }),
  ],
})
export class AppModule {}
```

**Startup vs Runtime Validation:**

```typescript
// ❌ BAD: Runtime validation (errors happen during requests)
@Injectable()
export class BadService {
  constructor(private configService: ConfigService) {}

  async sendEmail() {
    // Validation happens HERE (runtime)
    const apiKey = this.configService.get<string>('SENDGRID_API_KEY');
    
    if (!apiKey) {
      // Error occurs when user tries to send email!
      throw new Error('API key missing');
    }
    
    return this.emailClient.send({ apiKey });
  }
}

// ✅ GOOD: Startup validation (app won't start if invalid)
// app.module.ts - Validation configured at module level
ConfigModule.forRoot({
  validationSchema: Joi.object({
    SENDGRID_API_KEY: Joi.string().required(),  // ✅ Validated at startup
  }),
});

// Service can safely access the value
@Injectable()
export class GoodService {
  constructor(private configService: ConfigService) {}

  async sendEmail() {
    // No validation needed - guaranteed to exist
    const apiKey = this.configService.getOrThrow<string>('SENDGRID_API_KEY');
    return this.emailClient.send({ apiKey });
  }
}
```

**Advanced Joi Validation Patterns:**

```typescript
// config/validation.schema.ts
import * as Joi from 'joi';

export const validationSchema = Joi.object({
  // Conditional validation based on NODE_ENV
  NODE_ENV: Joi.string()
    .valid('development', 'production', 'test', 'staging')
    .default('development'),
  
  // Required in production only
  SSL_CERT: Joi.when('NODE_ENV', {
    is: 'production',
    then: Joi.string().required(),
    otherwise: Joi.string().optional(),
  }),
  
  // Number with range
  MAX_CONNECTIONS: Joi.number().min(1).max(100).default(10),
  
  // Port number
  PORT: Joi.number().port().default(3000),
  
  // Email format
  ADMIN_EMAIL: Joi.string().email().required(),
  
  // URL format
  API_URL: Joi.string().uri().required(),
  
  // Enum values
  LOG_LEVEL: Joi.string()
    .valid('error', 'warn', 'info', 'debug')
    .default('info'),
  
  // Boolean
  ENABLE_CORS: Joi.boolean().default(true),
  
  // Regex pattern (alphanumeric)
  API_KEY: Joi.string().pattern(/^[a-zA-Z0-9]+$/).min(20).required(),
  
  // Array (comma-separated string)
  CORS_ORIGINS: Joi.string().custom((value) => {
    return value.split(',').map(s => s.trim());
  }),
  
  // Multiple of
  BATCH_SIZE: Joi.number().multiple(10).default(100),
  
  // IP address
  ALLOWED_IP: Joi.string().ip().optional(),
  
  // Date
  TRIAL_END_DATE: Joi.date().iso().greater('now'),
  
  // Custom validation
  DATABASE_URL: Joi.string().custom((value, helpers) => {
    if (!value.startsWith('postgresql://') && !value.startsWith('postgres://')) {
      return helpers.error('Must be a PostgreSQL connection string');
    }
    return value;
  }),
});
```

**Class-Validator Alternative:**

```typescript
// config/env.validation.ts
import { plainToClass } from 'class-transformer';
import { IsEnum, IsNumber, IsString, Min, Max, validateSync } from 'class-validator';

enum Environment {
  Development = 'development',
  Production = 'production',
  Test = 'test',
}

class EnvironmentVariables {
  @IsEnum(Environment)
  NODE_ENV: Environment;

  @IsNumber()
  @Min(0)
  @Max(65535)
  PORT: number;

  @IsString()
  DATABASE_HOST: string;

  @IsNumber()
  @Min(1024)
  @Max(65535)
  DATABASE_PORT: number;

  @IsString()
  DATABASE_USERNAME: string;

  @IsString()
  DATABASE_PASSWORD: string;

  @IsString()
  JWT_SECRET: string;
}

export function validate(config: Record<string, unknown>) {
  const validatedConfig = plainToClass(EnvironmentVariables, config, {
    enableImplicitConversion: true,
  });

  const errors = validateSync(validatedConfig, {
    skipMissingProperties: false,
  });

  if (errors.length > 0) {
    throw new Error(errors.toString());
  }

  return validatedConfig;
}

// app.module.ts
@Module({
  imports: [
    ConfigModule.forRoot({
      validate,  // ✅ Use class-validator
    }),
  ],
})
export class AppModule {}
```

**Custom Validation Function:**

```typescript
// config/validation.ts
export function validateConfig(config: Record<string, unknown>) {
  const errors: string[] = [];
  
  // Required fields
  const required = ['DATABASE_HOST', 'DATABASE_PASSWORD', 'JWT_SECRET'];
  for (const field of required) {
    if (!config[field]) {
      errors.push(`${field} is required`);
    }
  }
  
  // Type checks
  if (config.PORT && isNaN(Number(config.PORT))) {
    errors.push('PORT must be a number');
  }
  
  // Length checks
  if (config.JWT_SECRET && (config.JWT_SECRET as string).length < 32) {
    errors.push('JWT_SECRET must be at least 32 characters');
  }
  
  // Enum checks
  const validEnvs = ['development', 'production', 'test', 'staging'];
  if (config.NODE_ENV && !validEnvs.includes(config.NODE_ENV as string)) {
    errors.push(`NODE_ENV must be one of: ${validEnvs.join(', ')}`);
  }
  
  // Conditional validation
  if (config.NODE_ENV === 'production') {
    if (!config.SSL_CERT) {
      errors.push('SSL_CERT is required in production');
    }
  }
  
  if (errors.length > 0) {
    throw new Error(`Configuration validation failed:\n${errors.join('\n')}`);
  }
  
  return config;
}

// app.module.ts
@Module({
  imports: [
    ConfigModule.forRoot({
      validate: validateConfig,  // ✅ Custom validation
    }),
  ],
})
export class AppModule {}
```

**Validation Error Handling:**

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  try {
    const app = await NestFactory.create(AppModule);
    await app.listen(3000);
    console.log('Application started successfully');
  } catch (error) {
    // Configuration validation errors will be caught here
    console.error('Failed to start application:');
    console.error(error.message);
    
    // Log specific validation errors
    if (error.details) {
      error.details.forEach((detail: any) => {
        console.error(`- ${detail.message}`);
      });
    }
    
    process.exit(1);  // Exit with error code
  }
}
bootstrap();
```

**Validation Best Practices:**

```typescript
// ✅ GOOD: Comprehensive validation at startup
validationSchema: Joi.object({
  // Validate everything upfront
  PORT: Joi.number().port().required(),
  DATABASE_URL: Joi.string().uri().required(),
  JWT_SECRET: Joi.string().min(32).required(),
})

// ✅ GOOD: Fail fast with abortEarly: false
validationOptions: {
  abortEarly: false,  // Show ALL errors at once
}

// ✅ GOOD: Environment-specific validation
AWS_KEY: Joi.when('NODE_ENV', {
  is: 'production',
  then: Joi.required(),
  otherwise: Joi.optional(),
})

// ✅ GOOD: Descriptive error messages
JWT_SECRET: Joi.string()
  .required()
  .min(32)
  .messages({
    'string.min': 'JWT_SECRET must be at least 32 characters for security',
    'any.required': 'JWT_SECRET is required for authentication',
  })

// ❌ BAD: No validation
ConfigModule.forRoot()  // Missing validationSchema

// ❌ BAD: Runtime validation
if (!this.configService.get('API_KEY')) {
  throw new Error('Missing');  // Should be caught at startup!
}

// ❌ BAD: abortEarly: true (default)
validationOptions: {
  abortEarly: true,  // Only shows FIRST error
}
```

**Where to Put Validation Schema:**

```
project/
├── src/
│   ├── config/
│   │   ├── validation.schema.ts     # ✅ Joi schema
│   │   ├── env.validation.ts        # ✅ class-validator
│   │   └── index.ts                 # Export all
│   ├── app.module.ts                # Import and use
│   └── main.ts
```

```typescript
// config/validation.schema.ts
import * as Joi from 'joi';

export const validationSchema = Joi.object({
  // All validation rules
});

// config/index.ts
export { validationSchema } from './validation.schema';

// app.module.ts
import { validationSchema } from './config';

@Module({
  imports: [
    ConfigModule.forRoot({
      validationSchema,  // ✅ Imported from dedicated file
    }),
  ],
})
export class AppModule {}
```

**Production Validation Checklist:**

```typescript
// ✅ Essential validations for production
validationSchema: Joi.object({
  // 1. Environment
  NODE_ENV: Joi.string().valid('production').required(),
  
  // 2. Security
  JWT_SECRET: Joi.string().min(32).required(),
  SESSION_SECRET: Joi.string().min(32).required(),
  
  // 3. Database
  DATABASE_URL: Joi.string().uri().required(),
  DATABASE_SSL: Joi.boolean().valid(true).required(),  // Force SSL
  
  // 4. APIs
  API_KEY: Joi.string().required(),
  
  // 5. Logging
  LOG_LEVEL: Joi.string().valid('error', 'warn').required(),  // Not 'debug'
  
  // 6. Features
  DEBUG_MODE: Joi.boolean().valid(false).required(),  // Never true in prod
})
```

**Interview Tip**: **Validate at startup** using **Joi** in ConfigModule.forRoot({ validationSchema }). This ensures **fail-fast behavior** - the app **won't start** with invalid configuration. Use **abortEarly: false** to show **all errors** at once. **Never validate at runtime** (when accessing values) - errors should be caught **before the app starts**, not during user requests. Common validations: **required()**, **min()/max()**, **port()**, **email()**, **uri()**, **valid()** for enums. **Production**: validate security requirements (SSL, strong secrets, no debug mode).

</details>

### 33. How do you handle sensitive data like API keys?

<details>
<summary>Answer</summary>

**Never commit sensitive data to version control.** Use **environment variables**, **.env files** (gitignored), **secret managers** (AWS Secrets Manager, Azure Key Vault), and **encryption** for sensitive configuration.

**Security Best Practices:**

```
1. Environment Variables (.env) ✅
   - Store in .env file
   - Add .env to .gitignore
   - Use .env.example as template
   
2. Secret Managers (Production) ✅
   - AWS Secrets Manager
   - Azure Key Vault
   - HashiCorp Vault
   - Google Secret Manager
   
3. Encryption at Rest ✅
   - Encrypt sensitive values
   - Use key management services
   
4. NEVER Commit Secrets ❌
   - NO hardcoded secrets
   - NO secrets in code
   - NO .env in git
```

**1. Using .env Files (Development):**

```bash
# .env (NEVER commit this!)
DATABASE_PASSWORD=SuperSecretPassword123!
JWT_SECRET=my-super-secret-jwt-key-with-32-chars-min
SENDGRID_API_KEY=SG.xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
STRIPE_SECRET_KEY=sk_test_xxxxxxxxxxxxxxxxxxxxx
GOOGLE_CLIENT_SECRET=xxxxxxxxxxxxxxxxxxxxx
```

```bash
# .gitignore (MUST include .env)
.env
.env.local
.env.*.local
.env.production
```

```bash
# .env.example (template for developers - SAFE to commit)
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_USERNAME=postgres
DATABASE_PASSWORD=   # Set this in your .env file
DATABASE_NAME=myapp

JWT_SECRET=          # Must be at least 32 characters
JWT_EXPIRATION=1h

SENDGRID_API_KEY=    # Get from SendGrid dashboard
FROM_EMAIL=

AWS_ACCESS_KEY_ID=   # AWS IAM credentials
AWS_SECRET_ACCESS_KEY=
AWS_REGION=us-east-1

STRIPE_SECRET_KEY=   # From Stripe dashboard
```

**2. ConfigService for Accessing Secrets:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import * as Joi from 'joi';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      
      // ✅ Validate sensitive data
      validationSchema: Joi.object({
        // Require secrets in production
        JWT_SECRET: Joi.string().required().min(32),
        DATABASE_PASSWORD: Joi.string().required().min(8),
        SENDGRID_API_KEY: Joi.string().required(),
        
        // Production-only secrets
        AWS_ACCESS_KEY_ID: Joi.when('NODE_ENV', {
          is: 'production',
          then: Joi.string().required(),
          otherwise: Joi.string().optional(),
        }),
      }),
    }),
  ],
})
export class AppModule {}
```

```typescript
// auth/auth.service.ts
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { JwtService } from '@nestjs/jwt';

@Injectable()
export class AuthService {
  constructor(
    private configService: ConfigService,
    private jwtService: JwtService,
  ) {}

  async generateToken(userId: string) {
    // ✅ Access secret from ConfigService
    const secret = this.configService.getOrThrow<string>('JWT_SECRET');
    
    return this.jwtService.sign({ sub: userId }, { secret });
  }
}
```

**3. AWS Secrets Manager Integration:**

```bash
# Install AWS SDK
npm install @aws-sdk/client-secrets-manager
```

```typescript
// config/secrets.config.ts
import { registerAs } from '@nestjs/config';
import {
  SecretsManagerClient,
  GetSecretValueCommand,
} from '@aws-sdk/client-secrets-manager';

// Fetch secrets from AWS Secrets Manager
async function getAwsSecret(secretName: string): Promise<any> {
  const client = new SecretsManagerClient({
    region: process.env.AWS_REGION || 'us-east-1',
  });

  try {
    const response = await client.send(
      new GetSecretValueCommand({
        SecretId: secretName,
      }),
    );

    return JSON.parse(response.SecretString || '{}');
  } catch (error) {
    console.error(`Failed to fetch secret ${secretName}:`, error);
    throw error;
  }
}

export default registerAs('secrets', async () => {
  const nodeEnv = process.env.NODE_ENV;

  // In production, fetch from AWS Secrets Manager
  if (nodeEnv === 'production') {
    const secrets = await getAwsSecret('prod/myapp/secrets');
    
    return {
      databasePassword: secrets.DATABASE_PASSWORD,
      jwtSecret: secrets.JWT_SECRET,
      sendgridApiKey: secrets.SENDGRID_API_KEY,
      stripeSecretKey: secrets.STRIPE_SECRET_KEY,
    };
  }

  // In development, use .env file
  return {
    databasePassword: process.env.DATABASE_PASSWORD,
    jwtSecret: process.env.JWT_SECRET,
    sendgridApiKey: process.env.SENDGRID_API_KEY,
    stripeSecretKey: process.env.STRIPE_SECRET_KEY,
  };
});
```

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import secretsConfig from './config/secrets.config';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      load: [secretsConfig],  // ✅ Load async secrets
    }),
  ],
})
export class AppModule {}
```

```typescript
// Usage
@Injectable()
export class PaymentService {
  constructor(private configService: ConfigService) {}

  async processPayment() {
    // Access secret from AWS Secrets Manager (production) or .env (dev)
    const stripeKey = this.configService.get<string>('secrets.stripeSecretKey');
    
    return stripe.charges.create({ apiKey: stripeKey });
  }
}
```

**4. Azure Key Vault Integration:**

```bash
npm install @azure/keyvault-secrets @azure/identity
```

```typescript
// config/azure-secrets.config.ts
import { registerAs } from '@nestjs/config';
import { SecretClient } from '@azure/keyvault-secrets';
import { DefaultAzureCredential } from '@azure/identity';

async function getAzureSecret(secretName: string): Promise<string> {
  const keyVaultName = process.env.KEY_VAULT_NAME;
  const url = `https://${keyVaultName}.vault.azure.net`;

  const credential = new DefaultAzureCredential();
  const client = new SecretClient(url, credential);

  try {
    const secret = await client.getSecret(secretName);
    return secret.value || '';
  } catch (error) {
    console.error(`Failed to fetch secret ${secretName}:`, error);
    throw error;
  }
}

export default registerAs('azureSecrets', async () => {
  if (process.env.NODE_ENV === 'production') {
    return {
      databasePassword: await getAzureSecret('database-password'),
      jwtSecret: await getAzureSecret('jwt-secret'),
      apiKey: await getAzureSecret('api-key'),
    };
  }

  // Development fallback
  return {
    databasePassword: process.env.DATABASE_PASSWORD,
    jwtSecret: process.env.JWT_SECRET,
    apiKey: process.env.API_KEY,
  };
});
```

**5. Encrypting Secrets in .env:**

```bash
npm install dotenv-vault
```

```typescript
// config/encrypted.config.ts
import { config } from 'dotenv-vault';

// Load encrypted .env file
config();

export default () => ({
  jwtSecret: process.env.JWT_SECRET,
  apiKey: process.env.API_KEY,
});
```

**6. Environment-Specific Secret Management:**

```typescript
// config/secrets.strategy.ts
interface SecretsProvider {
  getSecret(key: string): Promise<string>;
}

// Development: .env files
class EnvSecretsProvider implements SecretsProvider {
  async getSecret(key: string): Promise<string> {
    return process.env[key] || '';
  }
}

// Production: AWS Secrets Manager
class AwsSecretsProvider implements SecretsProvider {
  private client: SecretsManagerClient;

  constructor() {
    this.client = new SecretsManagerClient({
      region: process.env.AWS_REGION,
    });
  }

  async getSecret(key: string): Promise<string> {
    const response = await this.client.send(
      new GetSecretValueCommand({ SecretId: key }),
    );
    return response.SecretString || '';
  }
}

// Factory to get the right provider
export function getSecretsProvider(): SecretsProvider {
  const nodeEnv = process.env.NODE_ENV;

  switch (nodeEnv) {
    case 'production':
      return new AwsSecretsProvider();
    case 'development':
    case 'test':
    default:
      return new EnvSecretsProvider();
  }
}
```

**7. Secret Rotation:**

```typescript
// config/rotating-secrets.config.ts
import { Injectable } from '@nestjs/common';
import { Cron } from '@nestjs/schedule';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class SecretsRotationService {
  constructor(private configService: ConfigService) {}

  // Rotate secrets every 24 hours
  @Cron('0 0 * * *')
  async rotateSecrets() {
    console.log('Rotating secrets...');
    
    // Fetch new secrets from secret manager
    const newSecrets = await this.fetchLatestSecrets();
    
    // Update in-memory cache
    this.updateSecretsCache(newSecrets);
    
    console.log('Secrets rotated successfully');
  }

  private async fetchLatestSecrets() {
    // Implementation depends on your secret manager
  }

  private updateSecretsCache(secrets: any) {
    // Update cached secrets
  }
}
```

**8. CI/CD Secret Management:**

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      # ✅ Use GitHub Secrets (never hardcode)
      - name: Deploy to production
        env:
          DATABASE_PASSWORD: ${{ secrets.DATABASE_PASSWORD }}
          JWT_SECRET: ${{ secrets.JWT_SECRET }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        run: |
          npm run deploy
```

**Security Checklist:**

```typescript
// ✅ GOOD: Never hardcode secrets
const apiKey = this.configService.getOrThrow('API_KEY');

// ✅ GOOD: .env in .gitignore
// .gitignore: .env

// ✅ GOOD: Validate secret requirements
JWT_SECRET: Joi.string().required().min(32)

// ✅ GOOD: Use secret managers in production
if (NODE_ENV === 'production') {
  // AWS Secrets Manager, Azure Key Vault
}

// ✅ GOOD: Rotate secrets regularly
@Cron('0 0 * * *') async rotateSecrets() { }

// ✅ GOOD: Encrypt secrets at rest
// Use encryption for stored secrets

// ❌ BAD: Hardcoded secrets
const apiKey = 'sk_test_123456789';  // NEVER!

// ❌ BAD: Secrets in code
export const JWT_SECRET = 'my-secret';  // NEVER!

// ❌ BAD: .env committed to git
// Missing .gitignore for .env

// ❌ BAD: Logging secrets
console.log('API Key:', apiKey);  // NEVER log secrets!

// ❌ BAD: Exposing secrets in errors
throw new Error(`Failed with key: ${apiKey}`);  // NEVER!
```

**Docker Secrets:**

```dockerfile
# Dockerfile
FROM node:18-alpine

# ❌ BAD: Secrets in Dockerfile
# ENV JWT_SECRET=my-secret  # NEVER!

# ✅ GOOD: Pass secrets at runtime
# docker run -e JWT_SECRET=xxx myapp
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    image: myapp
    environment:
      - NODE_ENV=production
    secrets:
      - database_password
      - jwt_secret

secrets:
  database_password:
    external: true
  jwt_secret:
    external: true
```

**Interview Tip**: **Never commit secrets** to git. Use **.env files** (add to .gitignore) for development. Use **.env.example** as template without actual secrets. In **production**, use **secret managers**: **AWS Secrets Manager**, **Azure Key Vault**, **HashiCorp Vault**. Access via **ConfigService**, validate with **Joi** (required(), min()), **rotate regularly**. **Never** hardcode, log, or expose secrets in errors. Use **CI/CD secrets** (GitHub Secrets, GitLab Variables) for deployment.

</details>

### 34. Should configuration be synchronous or asynchronous?

<details>
<summary>Answer</summary>

**Use synchronous configuration by default** for simple env variables. **Use asynchronous configuration** when you need to fetch secrets from external sources (AWS Secrets Manager, databases, APIs) or perform I/O operations during config loading.

**Synchronous vs Asynchronous Configuration:**

| Feature | Synchronous (Default) | Asynchronous |
|---------|----------------------|--------------|
| **Loading** | Immediate | Fetches async data |
| **Use Case** | .env files, static values | Secret managers, databases, APIs |
| **Performance** | Fast startup | Slower startup (I/O) |
| **Complexity** | Simple | More complex |
| **When to Use** | Most scenarios | External data sources |
| **Module Method** | `ConfigModule.forRoot()` | `ConfigModule.forRootAsync()` |

**Synchronous Configuration (Default):**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import databaseConfig from './config/database.config';

@Module({
  imports: [
    // ✅ SYNCHRONOUS: Simple .env loading
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: '.env',
      load: [databaseConfig],  // Synchronous config factory
    }),
  ],
})
export class AppModule {}
```

```typescript
// config/database.config.ts - Synchronous
import { registerAs } from '@nestjs/config';

export default registerAs('database', () => ({
  // ✅ Simple, synchronous values
  host: process.env.DATABASE_HOST || 'localhost',
  port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
  username: process.env.DATABASE_USERNAME,
  password: process.env.DATABASE_PASSWORD,
  database: process.env.DATABASE_NAME,
}));
```

**Asynchronous Configuration (External Data Sources):**

```typescript
// config/secrets.config.ts - Asynchronous
import { registerAs } from '@nestjs/config';
import { SecretsManagerClient, GetSecretValueCommand } from '@aws-sdk/client-secrets-manager';

// ✅ ASYNC: Fetch from AWS Secrets Manager
export default registerAs('secrets', async () => {
  if (process.env.NODE_ENV !== 'production') {
    // Development: use .env (synchronous)
    return {
      databasePassword: process.env.DATABASE_PASSWORD,
      jwtSecret: process.env.JWT_SECRET,
    };
  }

  // Production: fetch from AWS (asynchronous)
  const client = new SecretsManagerClient({ region: 'us-east-1' });
  
  const command = new GetSecretValueCommand({
    SecretId: 'prod/myapp/secrets',
  });
  
  const response = await client.send(command);
  const secrets = JSON.parse(response.SecretString || '{}');

  return {
    databasePassword: secrets.DATABASE_PASSWORD,
    jwtSecret: secrets.JWT_SECRET,
  };
});
```

```typescript
// app.module.ts - Using async config
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import secretsConfig from './config/secrets.config';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      load: [secretsConfig],  // ✅ Async config automatically handled
    }),
  ],
})
export class AppModule {}
```

**Async Config Use Cases:**

```typescript
// 1. AWS Secrets Manager
export default registerAs('aws', async () => {
  const client = new SecretsManagerClient({ region: 'us-east-1' });
  const response = await client.send(
    new GetSecretValueCommand({ SecretId: 'myapp/secrets' }),
  );
  return JSON.parse(response.SecretString);
});

// 2. Azure Key Vault
export default registerAs('azure', async () => {
  const client = new SecretClient(vaultUrl, credential);
  const secret = await client.getSecret('database-password');
  return { databasePassword: secret.value };
});

// 3. Fetch from API
export default registerAs('api', async () => {
  const response = await fetch('https://config-server.com/api/config');
  const config = await response.json();
  return config;
});

// 4. Query Database
export default registerAs('dbConfig', async () => {
  const client = new Client({ connectionString: process.env.CONFIG_DB_URL });
  await client.connect();
  const result = await client.query('SELECT * FROM app_config');
  await client.end();
  return result.rows[0];
});

// 5. File System (async read)
export default registerAs('file', async () => {
  const fs = require('fs').promises;
  const content = await fs.readFile('./config.json', 'utf8');
  return JSON.parse(content);
});
```

**Module Configuration: forRoot() vs forRootAsync():**

```typescript
// Synchronous: forRoot()
@Module({
  imports: [
    TypeOrmModule.forRoot({
      // ✅ Simple, synchronous values
      type: 'postgres',
      host: process.env.DATABASE_HOST,
      port: parseInt(process.env.DATABASE_PORT, 10),
      username: process.env.DATABASE_USERNAME,
      password: process.env.DATABASE_PASSWORD,
    }),
  ],
})
export class AppModule {}

// Asynchronous: forRootAsync()
@Module({
  imports: [
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: async (configService: ConfigService) => {
        // ✅ Can perform async operations
        const secret = await fetchSecretFromVault();
        
        return {
          type: 'postgres',
          host: configService.get('DATABASE_HOST'),
          port: configService.get('DATABASE_PORT'),
          password: secret.databasePassword,  // Async secret
        };
      },
    }),
  ],
})
export class AppModule {}
```

**Performance Considerations:**

```typescript
// ❌ BAD: Unnecessary async (slower startup)
export default registerAs('config', async () => {
  // No actual async operation needed!
  return {
    port: process.env.PORT,
    host: process.env.HOST,
  };
});

// ✅ GOOD: Synchronous (faster)
export default registerAs('config', () => ({
  port: process.env.PORT,
  host: process.env.HOST,
}));

// ✅ GOOD: Async only when needed
export default registerAs('config', async () => {
  // Fetch from external source (actually async)
  const secrets = await secretsManager.getSecrets();
  return secrets;
});
```

**Mixed Sync/Async Configuration:**

```typescript
// app.module.ts
@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      load: [
        // Mix sync and async configs
        appConfig,        // ✅ Synchronous
        databaseConfig,   // ✅ Synchronous
        secretsConfig,    // ✅ Asynchronous (AWS Secrets Manager)
        cacheConfig,      // ✅ Synchronous
      ],
    }),
  ],
})
export class AppModule {}
```

**Error Handling in Async Config:**

```typescript
// config/secrets.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('secrets', async () => {
  try {
    // Attempt to fetch from secret manager
    const client = new SecretsManagerClient({ region: 'us-east-1' });
    const response = await client.send(
      new GetSecretValueCommand({ SecretId: 'myapp/secrets' }),
    );
    
    return JSON.parse(response.SecretString || '{}');
  } catch (error) {
    console.error('Failed to load secrets from AWS:', error);
    
    // ✅ FALLBACK: Use environment variables
    if (process.env.NODE_ENV === 'development') {
      console.warn('Falling back to environment variables');
      return {
        databasePassword: process.env.DATABASE_PASSWORD,
        jwtSecret: process.env.JWT_SECRET,
      };
    }
    
    // ❌ In production, fail fast
    throw new Error('Failed to load production secrets');
  }
});
```

**Caching Async Config:**

```typescript
// config/cached-secrets.config.ts
import { registerAs } from '@nestjs/config';

let cachedSecrets: any = null;

export default registerAs('secrets', async () => {
  // ✅ Cache to avoid repeated fetches
  if (cachedSecrets) {
    return cachedSecrets;
  }

  const client = new SecretsManagerClient({ region: 'us-east-1' });
  const response = await client.send(
    new GetSecretValueCommand({ SecretId: 'myapp/secrets' }),
  );
  
  cachedSecrets = JSON.parse(response.SecretString || '{}');
  
  return cachedSecrets;
});
```

**Testing Async Configuration:**

```typescript
// app.module.spec.ts
import { Test } from '@nestjs/testing';
import { ConfigModule } from '@nestjs/config';
import secretsConfig from './config/secrets.config';

describe('AppModule with async config', () => {
  it('should load async configuration', async () => {
    // ✅ Test async config loading
    const module = await Test.createTestingModule({
      imports: [
        ConfigModule.forRoot({
          load: [secretsConfig],  // Async config
        }),
      ],
    }).compile();

    const configService = module.get(ConfigService);
    const secret = configService.get('secrets.jwtSecret');
    
    expect(secret).toBeDefined();
  });
});
```

**Best Practices:**

```typescript
// ✅ GOOD: Use sync config for simple values
export default registerAs('app', () => ({
  port: parseInt(process.env.PORT, 10),
  host: process.env.HOST,
}));

// ✅ GOOD: Use async config for external data
export default registerAs('secrets', async () => {
  const secrets = await secretsManager.getSecrets();
  return secrets;
});

// ✅ GOOD: Fallback to sync in development
export default registerAs('config', async () => {
  if (process.env.NODE_ENV === 'development') {
    return { secret: process.env.SECRET };  // Sync
  }
  return await fetchFromVault();  // Async
});

// ✅ GOOD: Handle async errors
try {
  const secrets = await fetchSecrets();
  return secrets;
} catch (error) {
  console.error('Failed to fetch secrets:', error);
  throw error;
}

// ✅ GOOD: Cache async results
if (cache.has('secrets')) {
  return cache.get('secrets');
}
const secrets = await fetchSecrets();
cache.set('secrets', secrets);

// ❌ BAD: Unnecessary async
export default registerAs('config', async () => {
  return { port: process.env.PORT };  // No async needed!
});

// ❌ BAD: No error handling
export default registerAs('secrets', async () => {
  const secrets = await fetchSecrets();  // What if this fails?
  return secrets;
});

// ❌ BAD: Repeated async fetches
export default registerAs('secrets', async () => {
  // Fetches every time, should cache!
  return await fetchSecrets();
});
```

**When to Use Each:**

```typescript
// Use SYNCHRONOUS when:
// ✅ Loading .env files
// ✅ Reading environment variables
// ✅ Static configuration values
// ✅ Simple transformations (parseInt, etc.)
// ✅ Fast startup is critical

ConfigModule.forRoot({
  envFilePath: '.env',
  load: [appConfig, databaseConfig],  // Sync configs
});

// Use ASYNCHRONOUS when:
// ✅ Fetching from AWS Secrets Manager
// ✅ Fetching from Azure Key Vault
// ✅ Querying configuration database
// ✅ Calling configuration API
// ✅ Reading async file system operations
// ✅ Any I/O operation

ConfigModule.forRoot({
  load: [
    secretsConfig,      // Async: AWS Secrets Manager
    remoteConfig,       // Async: Config API
    dbConfig,           // Async: Database query
  ],
});
```

**Real-World Example:**

```typescript
// config/app.config.ts (Synchronous)
export default registerAs('app', () => ({
  port: parseInt(process.env.PORT, 10) || 3000,
  environment: process.env.NODE_ENV || 'development',
}));

// config/database.config.ts (Synchronous)
export default registerAs('database', () => ({
  host: process.env.DATABASE_HOST || 'localhost',
  port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
}));

// config/secrets.config.ts (Asynchronous - only in production)
export default registerAs('secrets', async () => {
  if (process.env.NODE_ENV !== 'production') {
    // Development: sync
    return {
      jwtSecret: process.env.JWT_SECRET,
      apiKey: process.env.API_KEY,
    };
  }

  // Production: async (AWS Secrets Manager)
  const client = new SecretsManagerClient({ region: 'us-east-1' });
  const response = await client.send(
    new GetSecretValueCommand({ SecretId: 'prod/secrets' }),
  );
  
  return JSON.parse(response.SecretString);
});

// app.module.ts
@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      load: [
        appConfig,       // Sync
        databaseConfig,  // Sync
        secretsConfig,   // Async (only in production)
      ],
    }),
  ],
})
export class AppModule {}
```

**Interview Tip**: Use **synchronous configuration** (default) for **.env files** and **simple values** - faster startup, simpler code. Use **asynchronous configuration** when fetching from **external sources**: **AWS Secrets Manager**, **Azure Key Vault**, **databases**, **APIs**. Async supported via **registerAs** with async function. **Best practice**: sync for development (.env), async for production (secret managers). Always **handle errors** and **cache async results** to avoid repeated fetches. **Performance**: sync is faster, async needed for I/O operations.

</details>

### 35. How do you use ConfigModule with database connections (`forRootAsync`)?

<details>
<summary>Answer</summary>

**Use `forRootAsync()`** to configure modules (like TypeORM, Mongoose) that depend on ConfigService. This ensures configuration is loaded before the module is initialized.

**Why forRootAsync()?**

```
1. Dependency Injection ✅
   - ConfigService not available in forRoot()
   - forRootAsync() allows injecting ConfigService
   
2. Dynamic Configuration ✅
   - Load config from external sources
   - Fetch secrets asynchronously
   
3. Type Safety ✅
   - Access typed configuration
   - Use ConfigService methods
```

**TypeORM with forRootAsync():**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { TypeOrmModule } from '@nestjs/typeorm';
import databaseConfig from './config/database.config';

@Module({
  imports: [
    // 1. Import ConfigModule first
    ConfigModule.forRoot({
      isGlobal: true,
      load: [databaseConfig],
    }),

    // 2. Use forRootAsync() to inject ConfigService
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],  // Import ConfigModule
      inject: [ConfigService],  // Inject ConfigService
      
      // ✅ useFactory gets ConfigService as parameter
      useFactory: (configService: ConfigService) => ({
        type: 'postgres',
        host: configService.get<string>('database.host'),
        port: configService.get<number>('database.port'),
        username: configService.get<string>('database.username'),
        password: configService.get<string>('database.password'),
        database: configService.get<string>('database.name'),
        entities: [__dirname + '/**/*.entity{.ts,.js}'],
        synchronize: configService.get<string>('NODE_ENV') !== 'production',
        logging: configService.get<string>('NODE_ENV') === 'development',
      }),
    }),
  ],
})
export class AppModule {}
```

**Database Configuration File:**

```typescript
// config/database.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('database', () => ({
  type: 'postgres',
  host: process.env.DATABASE_HOST || 'localhost',
  port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
  username: process.env.DATABASE_USERNAME || 'postgres',
  password: process.env.DATABASE_PASSWORD,
  name: process.env.DATABASE_NAME || 'myapp',
  
  // Connection pool
  pool: {
    max: parseInt(process.env.DATABASE_POOL_MAX, 10) || 10,
    min: parseInt(process.env.DATABASE_POOL_MIN, 10) || 2,
  },
  
  // SSL settings
  ssl: process.env.NODE_ENV === 'production' ? {
    rejectUnauthorized: false,
  } : false,
}));
```

```typescript
// app.module.ts - Using namespaced config
TypeOrmModule.forRootAsync({
  imports: [ConfigModule],
  inject: [ConfigService],
  useFactory: (configService: ConfigService) => {
    const dbConfig = configService.get('database');  // Get entire namespace
    
    return {
      type: dbConfig.type,
      host: dbConfig.host,
      port: dbConfig.port,
      username: dbConfig.username,
      password: dbConfig.password,
      database: dbConfig.name,
      ssl: dbConfig.ssl,
      extra: dbConfig.pool,
      entities: [__dirname + '/**/*.entity{.ts,.js}'],
      synchronize: process.env.NODE_ENV !== 'production',
    };
  },
}),
```

**Mongoose with forRootAsync():**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { MongooseModule } from '@nestjs/mongoose';
import { ConfigModule, ConfigService } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),

    // ✅ Mongoose with ConfigService
    MongooseModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (configService: ConfigService) => ({
        uri: configService.get<string>('MONGODB_URI'),
        useNewUrlParser: true,
        useUnifiedTopology: true,
        
        // Connection options
        maxPoolSize: configService.get<number>('MONGODB_POOL_SIZE', 10),
        serverSelectionTimeoutMS: 5000,
        
        // Authentication
        user: configService.get<string>('MONGODB_USERNAME'),
        pass: configService.get<string>('MONGODB_PASSWORD'),
        
        // SSL
        ssl: configService.get<string>('NODE_ENV') === 'production',
      }),
    }),
  ],
})
export class AppModule {}
```

**JWT Module with forRootAsync():**

```typescript
// auth/auth.module.ts
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { ConfigModule, ConfigService } from '@nestjs/config';

@Module({
  imports: [
    // ✅ JWT with ConfigService
    JwtModule.registerAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (configService: ConfigService) => ({
        secret: configService.getOrThrow<string>('JWT_SECRET'),
        signOptions: {
          expiresIn: configService.get<string>('JWT_EXPIRATION', '1h'),
          issuer: configService.get<string>('JWT_ISSUER', 'myapp'),
          audience: configService.get<string>('JWT_AUDIENCE', 'myapp-users'),
        },
      }),
    }),
  ],
})
export class AuthModule {}
```

**Throttler (Rate Limiting) with forRootAsync():**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ThrottlerModule } from '@nestjs/throttler';
import { ConfigModule, ConfigService } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),

    // ✅ Throttler with ConfigService
    ThrottlerModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (configService: ConfigService) => ({
        ttl: configService.get<number>('THROTTLE_TTL', 60),
        limit: configService.get<number>('THROTTLE_LIMIT', 10),
      }),
    }),
  ],
})
export class AppModule {}
```

**Async Configuration with Secret Fetching:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { SecretsManagerClient, GetSecretValueCommand } from '@aws-sdk/client-secrets-manager';

@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),

    // ✅ Async factory with external secret fetching
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: async (configService: ConfigService) => {
        let password: string;

        // Fetch password from AWS Secrets Manager in production
        if (configService.get('NODE_ENV') === 'production') {
          const client = new SecretsManagerClient({ region: 'us-east-1' });
          const response = await client.send(
            new GetSecretValueCommand({ SecretId: 'db-password' }),
          );
          password = JSON.parse(response.SecretString || '{}').password;
        } else {
          // Use .env in development
          password = configService.get<string>('DATABASE_PASSWORD');
        }

        return {
          type: 'postgres',
          host: configService.get<string>('DATABASE_HOST'),
          port: configService.get<number>('DATABASE_PORT'),
          username: configService.get<string>('DATABASE_USERNAME'),
          password,  // ✅ Fetched asynchronously
          database: configService.get<string>('DATABASE_NAME'),
          entities: [__dirname + '/**/*.entity{.ts,.js}'],
          synchronize: false,
        };
      },
    }),
  ],
})
export class AppModule {}
```

**Using Custom Configuration Token:**

```typescript
// config/database.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('database', () => ({
  host: process.env.DATABASE_HOST || 'localhost',
  port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
  username: process.env.DATABASE_USERNAME,
  password: process.env.DATABASE_PASSWORD,
  name: process.env.DATABASE_NAME,
}));

// app.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ConfigModule, ConfigService } from '@nestjs/config';
import databaseConfig from './config/database.config';
import { ConfigType } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      load: [databaseConfig],
    }),

    // ✅ Inject typed configuration
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      inject: [databaseConfig.KEY],  // Inject config token
      useFactory: (dbConfig: ConfigType<typeof databaseConfig>) => ({
        type: 'postgres',
        host: dbConfig.host,          // ✅ Type-safe access
        port: dbConfig.port,
        username: dbConfig.username,
        password: dbConfig.password,
        database: dbConfig.name,
        entities: [__dirname + '/**/*.entity{.ts,.js}'],
        synchronize: false,
      }),
    }),
  ],
})
export class AppModule {}
```

**Multiple Database Connections:**

```typescript
// app.module.ts
@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),

    // ✅ Primary database
    TypeOrmModule.forRootAsync({
      name: 'default',
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (configService: ConfigService) => ({
        type: 'postgres',
        host: configService.get('PRIMARY_DB_HOST'),
        port: configService.get('PRIMARY_DB_PORT'),
        username: configService.get('PRIMARY_DB_USERNAME'),
        password: configService.get('PRIMARY_DB_PASSWORD'),
        database: configService.get('PRIMARY_DB_NAME'),
        entities: [__dirname + '/**/*.entity{.ts,.js}'],
      }),
    }),

    // ✅ Secondary database (e.g., analytics)
    TypeOrmModule.forRootAsync({
      name: 'analytics',
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (configService: ConfigService) => ({
        type: 'postgres',
        host: configService.get('ANALYTICS_DB_HOST'),
        port: configService.get('ANALYTICS_DB_PORT'),
        username: configService.get('ANALYTICS_DB_USERNAME'),
        password: configService.get('ANALYTICS_DB_PASSWORD'),
        database: configService.get('ANALYTICS_DB_NAME'),
        entities: [__dirname + '/analytics/**/*.entity{.ts,.js}'],
      }),
    }),
  ],
})
export class AppModule {}
```

**Redis with forRootAsync():**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { CacheModule } from '@nestjs/cache-manager';
import { ConfigModule, ConfigService } from '@nestjs/config';
import * as redisStore from 'cache-manager-redis-store';

@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),

    // ✅ Redis cache with ConfigService
    CacheModule.registerAsync({
      isGlobal: true,
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (configService: ConfigService) => ({
        store: redisStore,
        host: configService.get<string>('REDIS_HOST', 'localhost'),
        port: configService.get<number>('REDIS_PORT', 6379),
        password: configService.get<string>('REDIS_PASSWORD'),
        ttl: configService.get<number>('CACHE_TTL', 3600),
        max: configService.get<number>('CACHE_MAX_ITEMS', 100),
      }),
    }),
  ],
})
export class AppModule {}
```

**Environment-Specific Database Configuration:**

```typescript
// app.module.ts
TypeOrmModule.forRootAsync({
  imports: [ConfigModule],
  inject: [ConfigService],
  useFactory: (configService: ConfigService) => {
    const nodeEnv = configService.get<string>('NODE_ENV');
    const isProduction = nodeEnv === 'production';
    const isDevelopment = nodeEnv === 'development';

    return {
      type: 'postgres',
      host: configService.get<string>('DATABASE_HOST'),
      port: configService.get<number>('DATABASE_PORT'),
      username: configService.get<string>('DATABASE_USERNAME'),
      password: configService.get<string>('DATABASE_PASSWORD'),
      database: configService.get<string>('DATABASE_NAME'),
      
      // Environment-specific settings
      synchronize: isDevelopment,  // ❌ NEVER in production!
      logging: isDevelopment ? 'all' : false,
      ssl: isProduction ? { rejectUnauthorized: false } : false,
      
      // Connection pool
      extra: {
        max: isProduction ? 20 : 5,
        min: isProduction ? 5 : 1,
        idleTimeoutMillis: isProduction ? 30000 : 10000,
      },
      
      entities: [__dirname + '/**/*.entity{.ts,.js}'],
      migrations: [__dirname + '/migrations/**/*{.ts,.js}'],
    };
  },
}),
```

**Best Practices:**

```typescript
// ✅ GOOD: Use forRootAsync() with ConfigService
TypeOrmModule.forRootAsync({
  imports: [ConfigModule],
  inject: [ConfigService],
  useFactory: (configService: ConfigService) => ({
    host: configService.get('DATABASE_HOST'),
  }),
})

// ✅ GOOD: Type-safe configuration access
useFactory: (configService: ConfigService) => ({
  host: configService.get<string>('DATABASE_HOST'),
  port: configService.get<number>('DATABASE_PORT', 5432),
})

// ✅ GOOD: Environment-specific configuration
synchronize: configService.get('NODE_ENV') !== 'production'

// ✅ GOOD: Async secret fetching
useFactory: async (configService: ConfigService) => {
  const password = await fetchSecret();
  return { password };
}

// ✅ GOOD: Inject typed configuration
inject: [databaseConfig.KEY]
useFactory: (dbConfig: ConfigType<typeof databaseConfig>) => ({ })

// ❌ BAD: Using forRoot() directly (no ConfigService)
TypeOrmModule.forRoot({
  host: process.env.DATABASE_HOST,  // No validation, no type safety
})

// ❌ BAD: Not importing ConfigModule
TypeOrmModule.forRootAsync({
  // imports: [ConfigModule],  // Missing!
  inject: [ConfigService],
  useFactory: (configService: ConfigService) => ({ }),
})

// ❌ BAD: synchronize: true in production
synchronize: true  // DANGEROUS! Can destroy data!

// ❌ BAD: No default values
port: configService.get<number>('DATABASE_PORT')  // Undefined if missing
```

**Interview Tip**: Use **forRootAsync()** to configure modules (TypeORM, Mongoose, JWT) that need **ConfigService**. Pattern: **imports: [ConfigModule]**, **inject: [ConfigService]**, **useFactory** receives ConfigService as parameter. Enables **type-safe** access, **validation**, **environment-specific** settings. For typed config, inject **config.KEY** token. **Never** use synchronize: true in production. **Best practice**: ConfigModule first, then forRootAsync() modules. Supports **async operations** (fetch secrets from AWS Secrets Manager).

</details>

## Common Patterns

### 36. How do you implement configuration for TypeORM using ConfigService?

<details>
<summary>Answer</summary>

**Create a dedicated database configuration file** using `registerAs()`, then inject it into TypeORM using `forRootAsync()` with ConfigService or typed configuration injection.

**Complete TypeORM Configuration Pattern:**

```typescript
// config/database.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('database', () => ({
  type: 'postgres' as const,
  host: process.env.DATABASE_HOST || 'localhost',
  port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
  username: process.env.DATABASE_USERNAME || 'postgres',
  password: process.env.DATABASE_PASSWORD,
  database: process.env.DATABASE_NAME || 'myapp',
  
  // Synchronization (NEVER in production!)
  synchronize: process.env.NODE_ENV !== 'production',
  
  // Logging
  logging: process.env.NODE_ENV === 'development',
  
  // SSL
  ssl: process.env.NODE_ENV === 'production' ? {
    rejectUnauthorized: false,
  } : false,
  
  // Connection pool
  extra: {
    max: parseInt(process.env.DATABASE_POOL_MAX, 10) || 10,
    min: parseInt(process.env.DATABASE_POOL_MIN, 10) || 2,
    idleTimeoutMillis: 30000,
    connectionTimeoutMillis: 2000,
  },
  
  // Retry
  retryAttempts: parseInt(process.env.DATABASE_RETRY_ATTEMPTS, 10) || 3,
  retryDelay: parseInt(process.env.DATABASE_RETRY_DELAY, 10) || 3000,
  
  // Auto load entities
  autoLoadEntities: true,
}));
```

```bash
# .env
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_USERNAME=postgres
DATABASE_PASSWORD=your_password
DATABASE_NAME=myapp_dev

# Connection pool
DATABASE_POOL_MAX=10
DATABASE_POOL_MIN=2

# Retry
DATABASE_RETRY_ATTEMPTS=3
DATABASE_RETRY_DELAY=3000

NODE_ENV=development
```

**Method 1: Using ConfigService (Standard):**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { TypeOrmModule } from '@nestjs/typeorm';
import databaseConfig from './config/database.config';

@Module({
  imports: [
    // 1. Load ConfigModule with database config
    ConfigModule.forRoot({
      isGlobal: true,
      load: [databaseConfig],
    }),

    // 2. Configure TypeORM using ConfigService
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (configService: ConfigService) => ({
        type: configService.get<'postgres'>('database.type'),
        host: configService.get<string>('database.host'),
        port: configService.get<number>('database.port'),
        username: configService.get<string>('database.username'),
        password: configService.get<string>('database.password'),
        database: configService.get<string>('database.database'),
        
        // Options
        synchronize: configService.get<boolean>('database.synchronize'),
        logging: configService.get<boolean>('database.logging'),
        ssl: configService.get('database.ssl'),
        extra: configService.get('database.extra'),
        
        // Retry
        retryAttempts: configService.get<number>('database.retryAttempts'),
        retryDelay: configService.get<number>('database.retryDelay'),
        
        // Auto-load entities from modules
        autoLoadEntities: configService.get<boolean>('database.autoLoadEntities'),
        
        // Or explicit entity paths
        entities: [__dirname + '/**/*.entity{.ts,.js}'],
        migrations: [__dirname + '/migrations/**/*{.ts,.js}'],
      }),
    }),
  ],
})
export class AppModule {}
```

**Method 2: Using Typed Configuration Injection (Recommended):**

```typescript
// config/database.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('database', () => ({
  type: 'postgres' as const,
  host: process.env.DATABASE_HOST || 'localhost',
  port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
  username: process.env.DATABASE_USERNAME || 'postgres',
  password: process.env.DATABASE_PASSWORD,
  database: process.env.DATABASE_NAME || 'myapp',
  synchronize: process.env.NODE_ENV !== 'production',
  logging: process.env.NODE_ENV === 'development',
  autoLoadEntities: true,
  ssl: process.env.NODE_ENV === 'production' ? {
    rejectUnauthorized: false,
  } : false,
}));

// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule, ConfigType } from '@nestjs/config';
import { TypeOrmModule } from '@nestjs/typeorm';
import databaseConfig from './config/database.config';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      load: [databaseConfig],
    }),

    // ✅ RECOMMENDED: Type-safe injection using config.KEY
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      inject: [databaseConfig.KEY],  // Inject typed config
      useFactory: (dbConfig: ConfigType<typeof databaseConfig>) => ({
        type: dbConfig.type,
        host: dbConfig.host,
        port: dbConfig.port,
        username: dbConfig.username,
        password: dbConfig.password,
        database: dbConfig.database,
        synchronize: dbConfig.synchronize,
        logging: dbConfig.logging,
        ssl: dbConfig.ssl,
        autoLoadEntities: dbConfig.autoLoadEntities,
        entities: [__dirname + '/**/*.entity{.ts,.js}'],
      }),
    }),
  ],
})
export class AppModule {}
```

**Advanced: Multiple Database Connections:**

```typescript
// config/databases.config.ts
import { registerAs } from '@nestjs/config';

// Primary database
export const primaryDbConfig = registerAs('primaryDb', () => ({
  type: 'postgres' as const,
  host: process.env.PRIMARY_DB_HOST || 'localhost',
  port: parseInt(process.env.PRIMARY_DB_PORT, 10) || 5432,
  username: process.env.PRIMARY_DB_USERNAME,
  password: process.env.PRIMARY_DB_PASSWORD,
  database: process.env.PRIMARY_DB_NAME,
  synchronize: false,
  autoLoadEntities: true,
}));

// Analytics database (read-only replica)
export const analyticsDbConfig = registerAs('analyticsDb', () => ({
  type: 'postgres' as const,
  host: process.env.ANALYTICS_DB_HOST || 'localhost',
  port: parseInt(process.env.ANALYTICS_DB_PORT, 10) || 5433,
  username: process.env.ANALYTICS_DB_USERNAME,
  password: process.env.ANALYTICS_DB_PASSWORD,
  database: process.env.ANALYTICS_DB_NAME,
  synchronize: false,
  logging: false,
}));
```

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule, ConfigType } from '@nestjs/config';
import { TypeOrmModule } from '@nestjs/typeorm';
import { primaryDbConfig, analyticsDbConfig } from './config/databases.config';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      load: [primaryDbConfig, analyticsDbConfig],
    }),

    // Primary database connection
    TypeOrmModule.forRootAsync({
      name: 'default',  // Default connection
      imports: [ConfigModule],
      inject: [primaryDbConfig.KEY],
      useFactory: (config: ConfigType<typeof primaryDbConfig>) => ({
        ...config,
        entities: [__dirname + '/**/*.entity{.ts,.js}'],
      }),
    }),

    // Analytics database connection
    TypeOrmModule.forRootAsync({
      name: 'analytics',  // Named connection
      imports: [ConfigModule],
      inject: [analyticsDbConfig.KEY],
      useFactory: (config: ConfigType<typeof analyticsDbConfig>) => ({
        ...config,
        entities: [__dirname + '/analytics/**/*.entity{.ts,.js}'],
      }),
    }),
  ],
})
export class AppModule {}
```

**Using in Services:**

```typescript
// users/users.service.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository, InjectConnection } from '@nestjs/typeorm';
import { Repository, Connection } from 'typeorm';
import { User } from './entities/user.entity';

@Injectable()
export class UsersService {
  constructor(
    // Primary database
    @InjectRepository(User)
    private usersRepository: Repository<User>,
    
    // Analytics database
    @InjectConnection('analytics')
    private analyticsConnection: Connection,
  ) {}

  async findAll() {
    // Query primary database
    return this.usersRepository.find();
  }

  async getAnalytics() {
    // Query analytics database
    return this.analyticsConnection
      .getRepository('Analytics')
      .find();
  }
}
```

**TypeORM Data Source (TypeORM 0.3+):**

```typescript
// config/typeorm.config.ts
import { DataSource, DataSourceOptions } from 'typeorm';
import { ConfigService } from '@nestjs/config';

export const getTypeOrmConfig = (configService: ConfigService): DataSourceOptions => ({
  type: 'postgres',
  host: configService.get<string>('DATABASE_HOST'),
  port: configService.get<number>('DATABASE_PORT'),
  username: configService.get<string>('DATABASE_USERNAME'),
  password: configService.get<string>('DATABASE_PASSWORD'),
  database: configService.get<string>('DATABASE_NAME'),
  
  synchronize: false,  // NEVER true in production
  logging: configService.get<string>('NODE_ENV') === 'development',
  
  entities: [__dirname + '/../**/*.entity{.ts,.js}'],
  migrations: [__dirname + '/../migrations/**/*{.ts,.js}'],
  subscribers: [__dirname + '/../**/*.subscriber{.ts,.js}'],
  
  ssl: configService.get<string>('NODE_ENV') === 'production' ? {
    rejectUnauthorized: false,
  } : false,
});

// For migrations CLI
export default new DataSource({
  type: 'postgres',
  host: process.env.DATABASE_HOST || 'localhost',
  port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
  username: process.env.DATABASE_USERNAME || 'postgres',
  password: process.env.DATABASE_PASSWORD,
  database: process.env.DATABASE_NAME || 'myapp',
  entities: [__dirname + '/../**/*.entity{.ts,.js}'],
  migrations: [__dirname + '/../migrations/**/*{.ts,.js}'],
});
```

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { getTypeOrmConfig } from './config/typeorm.config';

@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),

    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (configService: ConfigService) => 
        getTypeOrmConfig(configService),
    }),
  ],
})
export class AppModule {}
```

**Validation for Database Config:**

```typescript
// config/validation.schema.ts
import * as Joi from 'joi';

export const validationSchema = Joi.object({
  // Database
  DATABASE_HOST: Joi.string().required(),
  DATABASE_PORT: Joi.number().port().default(5432),
  DATABASE_USERNAME: Joi.string().required(),
  DATABASE_PASSWORD: Joi.string().required().min(8),
  DATABASE_NAME: Joi.string().required(),
  
  // Connection pool
  DATABASE_POOL_MAX: Joi.number().min(1).max(100).default(10),
  DATABASE_POOL_MIN: Joi.number().min(1).max(50).default(2),
  
  // Retry
  DATABASE_RETRY_ATTEMPTS: Joi.number().min(0).max(10).default(3),
  DATABASE_RETRY_DELAY: Joi.number().min(0).max(10000).default(3000),
  
  // Environment
  NODE_ENV: Joi.string()
    .valid('development', 'production', 'test')
    .default('development'),
});
```

```typescript
// app.module.ts
@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      load: [databaseConfig],
      validationSchema,  // ✅ Validate database config
    }),
  ],
})
export class AppModule {}
```

**Environment-Specific Configuration:**

```bash
# .env.development
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_USERNAME=dev_user
DATABASE_PASSWORD=dev_password
DATABASE_NAME=myapp_dev
DATABASE_POOL_MAX=5
NODE_ENV=development

# .env.production
DATABASE_HOST=prod-db.example.com
DATABASE_PORT=5432
DATABASE_USERNAME=prod_user
DATABASE_PASSWORD=strong_production_password_123
DATABASE_NAME=myapp_prod
DATABASE_POOL_MAX=20
DATABASE_POOL_MIN=5
NODE_ENV=production
```

**Docker Configuration:**

```yaml
# docker-compose.yml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_DB: myapp
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  app:
    build: .
    environment:
      DATABASE_HOST: postgres
      DATABASE_PORT: 5432
      DATABASE_USERNAME: postgres
      DATABASE_PASSWORD: password
      DATABASE_NAME: myapp
      NODE_ENV: development
    ports:
      - "3000:3000"
    depends_on:
      - postgres

volumes:
  postgres_data:
```

**Best Practices:**

```typescript
// ✅ GOOD: Dedicated config file
// config/database.config.ts
export default registerAs('database', () => ({ }))

// ✅ GOOD: Type-safe injection
inject: [databaseConfig.KEY]
useFactory: (dbConfig: ConfigType<typeof databaseConfig>) => ({ })

// ✅ GOOD: NEVER synchronize in production
synchronize: process.env.NODE_ENV !== 'production'

// ✅ GOOD: Environment-specific settings
logging: process.env.NODE_ENV === 'development'
ssl: process.env.NODE_ENV === 'production' ? { } : false

// ✅ GOOD: Connection pool sizing
extra: {
  max: process.env.NODE_ENV === 'production' ? 20 : 5,
  min: process.env.NODE_ENV === 'production' ? 5 : 1,
}

// ✅ GOOD: Auto-load entities
autoLoadEntities: true

// ✅ GOOD: Retry logic
retryAttempts: 3,
retryDelay: 3000,

// ❌ BAD: Hardcoded credentials
password: 'hardcoded_password'  // NEVER!

// ❌ BAD: synchronize: true in production
synchronize: true  // Dangerous! Can destroy data!

// ❌ BAD: No SSL in production
ssl: false  // Should be true in production

// ❌ BAD: Using forRoot() directly
TypeOrmModule.forRoot({
  host: process.env.DATABASE_HOST,  // No validation
})
```

**Interview Tip**: Create **dedicated config file** using **registerAs('database')** with database settings. Use **TypeOrmModule.forRootAsync()** to inject config. Two methods: **ConfigService** (inject ConfigService, access via get()) or **typed config** (inject databaseConfig.KEY, type-safe access). **Critical**: **NEVER** synchronize: true in production (can destroy data). Use **environment-specific** settings: logging/SSL based on NODE_ENV, connection pool sizing. **Validate** with Joi (required fields, port numbers). Support **multiple databases** with named connections.

</details>

### 37. How do you implement configuration for JWT using ConfigService?

<details>
<summary>Answer</summary>

**Create a JWT configuration file** using `registerAs()`, then inject it into JwtModule using `registerAsync()` with ConfigService for type-safe, validated JWT setup.

**Complete JWT Configuration Pattern:**

```typescript
// config/jwt.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('jwt', () => ({
  // Secret key (should be long and complex)
  secret: process.env.JWT_SECRET,
  
  // Access token settings
  accessToken: {
    secret: process.env.JWT_ACCESS_SECRET || process.env.JWT_SECRET,
    expiresIn: process.env.JWT_ACCESS_EXPIRATION || '15m',
  },
  
  // Refresh token settings
  refreshToken: {
    secret: process.env.JWT_REFRESH_SECRET || process.env.JWT_SECRET,
    expiresIn: process.env.JWT_REFRESH_EXPIRATION || '7d',
  },
  
  // JWT options
  issuer: process.env.JWT_ISSUER || 'myapp',
  audience: process.env.JWT_AUDIENCE || 'myapp-users',
  
  // Algorithm
  algorithm: process.env.JWT_ALGORITHM || 'HS256',
  
  // Public/private keys (for RS256)
  publicKey: process.env.JWT_PUBLIC_KEY,
  privateKey: process.env.JWT_PRIVATE_KEY,
}));
```

```bash
# .env
# JWT Secrets (REQUIRED - minimum 32 characters)
JWT_SECRET=your-super-secret-jwt-key-min-32-chars-long-for-security
JWT_ACCESS_SECRET=your-access-token-secret-key-here
JWT_REFRESH_SECRET=your-refresh-token-secret-key-here

# JWT Expiration
JWT_ACCESS_EXPIRATION=15m
JWT_REFRESH_EXPIRATION=7d

# JWT Metadata
JWT_ISSUER=myapp
JWT_AUDIENCE=myapp-users

# Algorithm (HS256, HS384, HS512, RS256, RS384, RS512)
JWT_ALGORITHM=HS256

# For RS256 (asymmetric)
# JWT_PUBLIC_KEY=-----BEGIN PUBLIC KEY-----...
# JWT_PRIVATE_KEY=-----BEGIN PRIVATE KEY-----...
```

**Method 1: Simple JWT Setup with ConfigService:**

```typescript
// auth/auth.module.ts
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { ConfigModule, ConfigService } from '@nestjs/config';
import jwtConfig from '../config/jwt.config';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      load: [jwtConfig],
    }),

    // ✅ Configure JwtModule with ConfigService
    JwtModule.registerAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (configService: ConfigService) => ({
        secret: configService.getOrThrow<string>('jwt.secret'),
        signOptions: {
          expiresIn: configService.get<string>('jwt.accessToken.expiresIn', '15m'),
          issuer: configService.get<string>('jwt.issuer', 'myapp'),
          audience: configService.get<string>('jwt.audience', 'myapp-users'),
        },
      }),
    }),
  ],
  providers: [AuthService],
  exports: [AuthService],
})
export class AuthModule {}
```

**Method 2: Typed Configuration Injection (Recommended):**

```typescript
// auth/auth.module.ts
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { ConfigModule, ConfigType } from '@nestjs/config';
import jwtConfig from '../config/jwt.config';

@Module({
  imports: [
    ConfigModule.forFeature(jwtConfig),

    // ✅ RECOMMENDED: Type-safe JWT configuration
    JwtModule.registerAsync({
      imports: [ConfigModule],
      inject: [jwtConfig.KEY],
      useFactory: (config: ConfigType<typeof jwtConfig>) => ({
        secret: config.secret,
        signOptions: {
          expiresIn: config.accessToken.expiresIn,
          issuer: config.issuer,
          audience: config.audience,
        },
      }),
    }),
  ],
})
export class AuthModule {}
```

**Using JWT in Auth Service:**

```typescript
// auth/auth.service.ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class AuthService {
  constructor(
    private jwtService: JwtService,
    private configService: ConfigService,
  ) {}

  // Generate access token
  async generateAccessToken(userId: string, email: string) {
    const payload = { sub: userId, email };
    
    return this.jwtService.sign(payload, {
      secret: this.configService.getOrThrow<string>('jwt.accessToken.secret'),
      expiresIn: this.configService.get<string>('jwt.accessToken.expiresIn', '15m'),
      issuer: this.configService.get<string>('jwt.issuer'),
      audience: this.configService.get<string>('jwt.audience'),
    });
  }

  // Generate refresh token
  async generateRefreshToken(userId: string) {
    const payload = { sub: userId, type: 'refresh' };
    
    return this.jwtService.sign(payload, {
      secret: this.configService.getOrThrow<string>('jwt.refreshToken.secret'),
      expiresIn: this.configService.get<string>('jwt.refreshToken.expiresIn', '7d'),
      issuer: this.configService.get<string>('jwt.issuer'),
    });
  }

  // Generate both tokens
  async generateTokens(userId: string, email: string) {
    const [accessToken, refreshToken] = await Promise.all([
      this.generateAccessToken(userId, email),
      this.generateRefreshToken(userId),
    ]);

    return {
      accessToken,
      refreshToken,
      expiresIn: this.configService.get<string>('jwt.accessToken.expiresIn'),
    };
  }

  // Verify access token
  async verifyAccessToken(token: string) {
    try {
      return this.jwtService.verify(token, {
        secret: this.configService.getOrThrow<string>('jwt.accessToken.secret'),
        issuer: this.configService.get<string>('jwt.issuer'),
        audience: this.configService.get<string>('jwt.audience'),
      });
    } catch (error) {
      throw new UnauthorizedException('Invalid token');
    }
  }

  // Verify refresh token
  async verifyRefreshToken(token: string) {
    try {
      return this.jwtService.verify(token, {
        secret: this.configService.getOrThrow<string>('jwt.refreshToken.secret'),
      });
    } catch (error) {
      throw new UnauthorizedException('Invalid refresh token');
    }
  }
}
```

**JWT Strategy with ConfigService:**

```typescript
// auth/strategies/jwt.strategy.ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(private configService: ConfigService) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: configService.getOrThrow<string>('jwt.accessToken.secret'),
      issuer: configService.get<string>('jwt.issuer'),
      audience: configService.get<string>('jwt.audience'),
    });
  }

  async validate(payload: any) {
    if (!payload.sub) {
      throw new UnauthorizedException();
    }

    return {
      userId: payload.sub,
      email: payload.email,
    };
  }
}
```

**Asymmetric Keys (RS256):**

```typescript
// config/jwt.config.ts
import { registerAs } from '@nestjs/config';
import { readFileSync } from 'fs';
import { join } from 'path';

export default registerAs('jwt', () => {
  const algorithm = process.env.JWT_ALGORITHM || 'HS256';

  // For asymmetric algorithms (RS256, RS384, RS512)
  if (algorithm.startsWith('RS')) {
    return {
      algorithm,
      publicKey: process.env.JWT_PUBLIC_KEY || 
                 readFileSync(join(__dirname, '../keys/public.pem'), 'utf8'),
      privateKey: process.env.JWT_PRIVATE_KEY || 
                  readFileSync(join(__dirname, '../keys/private.pem'), 'utf8'),
      expiresIn: process.env.JWT_EXPIRATION || '15m',
      issuer: process.env.JWT_ISSUER || 'myapp',
      audience: process.env.JWT_AUDIENCE || 'myapp-users',
    };
  }

  // For symmetric algorithms (HS256, HS384, HS512)
  return {
    algorithm,
    secret: process.env.JWT_SECRET,
    expiresIn: process.env.JWT_EXPIRATION || '15m',
    issuer: process.env.JWT_ISSUER || 'myapp',
    audience: process.env.JWT_AUDIENCE || 'myapp-users',
  };
});
```

```typescript
// auth/auth.module.ts - RS256 setup
JwtModule.registerAsync({
  imports: [ConfigModule],
  inject: [jwtConfig.KEY],
  useFactory: (config: ConfigType<typeof jwtConfig>) => {
    if (config.algorithm.startsWith('RS')) {
      return {
        privateKey: config.privateKey,
        publicKey: config.publicKey,
        signOptions: {
          expiresIn: config.expiresIn,
          algorithm: config.algorithm as Algorithm,
          issuer: config.issuer,
          audience: config.audience,
        },
      };
    }

    return {
      secret: config.secret,
      signOptions: {
        expiresIn: config.expiresIn,
        algorithm: config.algorithm as Algorithm,
        issuer: config.issuer,
        audience: config.audience,
      },
    };
  },
}),
```

**Validation Schema for JWT:**

```typescript
// config/validation.schema.ts
import * as Joi from 'joi';

export const validationSchema = Joi.object({
  // JWT Secret (required, minimum 32 characters)
  JWT_SECRET: Joi.string().required().min(32).messages({
    'string.min': 'JWT_SECRET must be at least 32 characters for security',
    'any.required': 'JWT_SECRET is required for authentication',
  }),
  
  // Access/Refresh secrets
  JWT_ACCESS_SECRET: Joi.string().min(32).optional(),
  JWT_REFRESH_SECRET: Joi.string().min(32).optional(),
  
  // Expiration (examples: 15m, 1h, 7d, 30d)
  JWT_ACCESS_EXPIRATION: Joi.string()
    .pattern(/^\d+[smhd]$/)
    .default('15m'),
  JWT_REFRESH_EXPIRATION: Joi.string()
    .pattern(/^\d+[smhd]$/)
    .default('7d'),
  
  // Metadata
  JWT_ISSUER: Joi.string().default('myapp'),
  JWT_AUDIENCE: Joi.string().default('myapp-users'),
  
  // Algorithm
  JWT_ALGORITHM: Joi.string()
    .valid('HS256', 'HS384', 'HS512', 'RS256', 'RS384', 'RS512')
    .default('HS256'),
  
  // Asymmetric keys (required if using RS algorithms)
  JWT_PUBLIC_KEY: Joi.when('JWT_ALGORITHM', {
    is: Joi.string().pattern(/^RS/),
    then: Joi.string().required(),
    otherwise: Joi.string().optional(),
  }),
  JWT_PRIVATE_KEY: Joi.when('JWT_ALGORITHM', {
    is: Joi.string().pattern(/^RS/),
    then: Joi.string().required(),
    otherwise: Joi.string().optional(),
  }),
});
```

**Multiple JWT Modules:**

```typescript
// auth/auth.module.ts
@Module({
  imports: [
    // Access tokens
    JwtModule.registerAsync({
      name: 'access',
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (configService: ConfigService) => ({
        secret: configService.get('jwt.accessToken.secret'),
        signOptions: { expiresIn: '15m' },
      }),
    }),

    // Refresh tokens
    JwtModule.registerAsync({
      name: 'refresh',
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (configService: ConfigService) => ({
        secret: configService.get('jwt.refreshToken.secret'),
        signOptions: { expiresIn: '7d' },
      }),
    }),
  ],
})
export class AuthModule {}
```

```typescript
// auth/auth.service.ts
import { Inject } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';

@Injectable()
export class AuthService {
  constructor(
    @Inject('access') private accessJwtService: JwtService,
    @Inject('refresh') private refreshJwtService: JwtService,
  ) {}

  generateTokens(userId: string) {
    return {
      accessToken: this.accessJwtService.sign({ sub: userId }),
      refreshToken: this.refreshJwtService.sign({ sub: userId }),
    };
  }
}
```

**Best Practices:**

```typescript
// ✅ GOOD: Strong secret (32+ characters)
JWT_SECRET=a-very-long-and-complex-secret-key-with-at-least-32-characters

// ✅ GOOD: Separate secrets for access/refresh
JWT_ACCESS_SECRET=access-token-secret-32-chars
JWT_REFRESH_SECRET=refresh-token-secret-32-chars

// ✅ GOOD: Short-lived access tokens
JWT_ACCESS_EXPIRATION=15m  // 15 minutes

// ✅ GOOD: Long-lived refresh tokens
JWT_REFRESH_EXPIRATION=7d  // 7 days

// ✅ GOOD: Type-safe config injection
inject: [jwtConfig.KEY]
useFactory: (config: ConfigType<typeof jwtConfig>) => ({ })

// ✅ GOOD: Validate JWT_SECRET length
JWT_SECRET: Joi.string().required().min(32)

// ✅ GOOD: Use getOrThrow for required values
const secret = this.configService.getOrThrow<string>('jwt.secret');

// ❌ BAD: Weak secret
JWT_SECRET=secret  // Too short!

// ❌ BAD: Same secret for access and refresh
// Use different secrets for better security

// ❌ BAD: Long-lived access tokens
JWT_ACCESS_EXPIRATION=30d  // Security risk!

// ❌ BAD: Hardcoded secret
secret: 'hardcoded-secret'  // NEVER!

// ❌ BAD: No validation
// Missing Joi validation for JWT_SECRET
```

**Interview Tip**: Create **JWT config file** using **registerAs('jwt')** with secrets, expiration, issuer, audience. Use **JwtModule.registerAsync()** to inject config. Two methods: **ConfigService** (inject ConfigService) or **typed config** (inject jwtConfig.KEY). **Best practices**: **strong secret** (32+ chars), **validate with Joi** (required, min length), **short-lived access tokens** (15m), **long-lived refresh tokens** (7d), **separate secrets** for access/refresh. Support **asymmetric keys** (RS256) for microservices. Use **getOrThrow()** for required secrets.

</details>

### 38. How do you cache configuration values?

<details>
<summary>Answer</summary>

**ConfigService caches values by default** when using the `cache` option. For expensive operations (fetching from external APIs, databases), manually cache results in config factories or create a caching wrapper service.

**ConfigService Built-in Caching:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      cache: true,  // ✅ Enable caching (default behavior)
    }),
  ],
})
export class AppModule {}
```

**Manual Caching in Config Factory:**

```typescript
// config/external-api.config.ts
import { registerAs } from '@nestjs/config';

// Cache variable outside the factory
let cachedApiConfig: any = null;
let lastFetchTime = 0;
const CACHE_TTL = 5 * 60 * 1000; // 5 minutes

export default registerAs('externalApi', async () => {
  const now = Date.now();

  // ✅ Return cached value if still valid
  if (cachedApiConfig && (now - lastFetchTime) < CACHE_TTL) {
    console.log('Returning cached API config');
    return cachedApiConfig;
  }

  // Fetch fresh data
  console.log('Fetching fresh API config');
  const response = await fetch('https://api.example.com/config');
  const config = await response.json();

  // Update cache
  cachedApiConfig = config;
  lastFetchTime = now;

  return config;
});
```

**Caching Wrapper Service:**

```typescript
// config/cached-config.service.ts
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';

interface CacheEntry<T> {
  value: T;
  timestamp: number;
}

@Injectable()
export class CachedConfigService {
  private cache = new Map<string, CacheEntry<any>>();
  private readonly DEFAULT_TTL = 5 * 60 * 1000; // 5 minutes

  constructor(private configService: ConfigService) {}

  /**
   * Get config value with caching
   */
  get<T>(key: string, ttl: number = this.DEFAULT_TTL): T {
    const cached = this.cache.get(key);
    const now = Date.now();

    // Return cached value if valid
    if (cached && (now - cached.timestamp) < ttl) {
      return cached.value;
    }

    // Fetch fresh value
    const value = this.configService.get<T>(key);

    // Update cache
    this.cache.set(key, {
      value,
      timestamp: now,
    });

    return value;
  }

  /**
   * Get config value with fallback
   */
  getOrDefault<T>(key: string, defaultValue: T, ttl?: number): T {
    const value = this.get<T>(key, ttl);
    return value !== undefined ? value : defaultValue;
  }

  /**
   * Invalidate specific cache entry
   */
  invalidate(key: string): void {
    this.cache.delete(key);
  }

  /**
   * Clear entire cache
   */
  clearCache(): void {
    this.cache.clear();
  }

  /**
   * Get cache statistics
   */
  getCacheStats() {
    return {
      size: this.cache.size,
      keys: Array.from(this.cache.keys()),
    };
  }
}
```

**Interview Tip**: **ConfigService caches by default** with `cache: true` option. For **expensive operations** (external APIs, databases, secret managers), implement **manual caching** in config factories with TTL. Use **caching wrapper service** with Map/LRU cache for in-memory caching or **Redis** for distributed caching. **Best practices**: set **appropriate TTL** (5-10 minutes), provide **invalidation mechanism**, **warm up critical config** on startup. Cache **computed values** (database URLs) and **async fetches** (AWS Secrets Manager). Don't cache simple env var reads (already fast).

</details>

## Testing

### 39. How do you mock ConfigService in tests?

<details>
<summary>Answer</summary>

**Mock ConfigService** using Jest mocks to return predefined values for tests. Create a mock factory or use `@nestjs/testing` utilities to provide fake configuration.

**Simple Jest Mock:**

```typescript
// users/users.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { ConfigService } from '@nestjs/config';
import { UsersService } from './users.service';

describe('UsersService', () => {
  let service: UsersService;
  let configService: ConfigService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: ConfigService,
          useValue: {
            // ✅ Mock get() method
            get: jest.fn((key: string) => {
              const config = {
                'DATABASE_HOST': 'localhost',
                'DATABASE_PORT': 5432,
                'API_KEY': 'test-api-key',
                'JWT_SECRET': 'test-jwt-secret',
              };
              return config[key];
            }),
            
            // ✅ Mock getOrThrow() method
            getOrThrow: jest.fn((key: string) => {
              const config = {
                'API_KEY': 'test-api-key',
              };
              if (!(key in config)) {
                throw new Error(`Config key ${key} not found`);
              }
              return config[key];
            }),
          },
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
    configService = module.get<ConfigService>(ConfigService);
  });

  it('should get API key from config', () => {
    const apiKey = configService.get('API_KEY');
    expect(apiKey).toBe('test-api-key');
  });
});
```

**Mock ConfigService Factory:**

```typescript
// test/mocks/config.service.mock.ts
export const createMockConfigService = (config: Record<string, any>) => ({
  get: jest.fn((key: string, defaultValue?: any) => {
    return config[key] ?? defaultValue;
  }),
  
  getOrThrow: jest.fn((key: string) => {
    if (!(key in config)) {
      throw new Error(`Configuration key "${key}" does not exist`);
    }
    return config[key];
  }),
});

// Usage in tests
import { createMockConfigService } from '../test/mocks/config.service.mock';

describe('UsersService', () => {
  let service: UsersService;

  beforeEach(async () => {
    const mockConfig = createMockConfigService({
      'DATABASE_HOST': 'localhost',
      'DATABASE_PORT': 5432,
      'API_KEY': 'test-key',
    });

    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        { provide: ConfigService, useValue: mockConfig },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
  });
});
```

**Using forRoot() in Tests:**

```typescript
// users/users.service.spec.ts
describe('UsersService', () => {
  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      imports: [
        // ✅ Load test configuration
        ConfigModule.forRoot({
          isGlobal: true,
          ignoreEnvFile: true,  // Ignore .env files in tests
          load: [
            () => ({
              DATABASE_HOST: 'localhost',
              API_KEY: 'test-api-key',
              JWT_SECRET: 'test-jwt-secret-32-chars-min',
            }),
          ],
        }),
      ],
      providers: [UsersService],
    }).compile();

    service = module.get<UsersService>(UsersService);
  });
});
```

**Mocking Specific Methods:**

```typescript
describe('UsersService', () => {
  let mockConfigService: jest.Mocked<ConfigService>;

  beforeEach(async () => {
    mockConfigService = {
      get: jest.fn(),
      getOrThrow: jest.fn(),
    } as any;

    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        { provide: ConfigService, useValue: mockConfigService },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
  });

  it('should use API key from config', () => {
    // ✅ Mock return value for this test
    mockConfigService.get.mockReturnValue('my-test-api-key');

    const result = service.getApiKey();

    expect(mockConfigService.get).toHaveBeenCalledWith('API_KEY');
    expect(result).toBe('my-test-api-key');
  });

  it('should throw when config is missing', () => {
    // ✅ Mock to throw error
    mockConfigService.getOrThrow.mockImplementation(() => {
      throw new Error('Config not found');
    });

    expect(() => service.getRequiredConfig()).toThrow('Config not found');
  });
});
```

**Interview Tip**: **Mock ConfigService** in tests using **Jest mocks** with `useValue`. Create **mock factory** returning predefined values: `{ get: jest.fn(), getOrThrow: jest.fn() }`. **Best practices**: use **ignoreEnvFile: true** to avoid .env dependencies, create **test config files** for reusability, **mock return values** per test with `mockReturnValue()`, **clean up** env vars in afterEach(). For integration tests, use **ConfigModule.forRoot()** with test config factory. Mock **both get() and getOrThrow()** methods. Test success and error scenarios.

</details>

### 40. How do you test with different configuration values?

<details>
<summary>Answer</summary>

**Create separate test modules** with different ConfigModule setups, use parameterized tests, or dynamically change mock return values to test various configuration scenarios.

**Separate Test Suites:**

```typescript
describe('UsersService - Development Config', () => {
  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      imports: [
        ConfigModule.forRoot({
          ignoreEnvFile: true,
          load: [
            () => ({
              NODE_ENV: 'development',
              DEBUG: true,
              DATABASE_HOST: 'localhost',
            }),
          ],
        }),
      ],
      providers: [UsersService],
    }).compile();

    service = module.get<UsersService>(UsersService);
  });

  it('should enable debug mode', () => {
    expect(service.isDebugEnabled()).toBe(true);
  });
});

describe('UsersService - Production Config', () => {
  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      imports: [
        ConfigModule.forRoot({
          ignoreEnvFile: true,
          load: [
            () => ({
              NODE_ENV: 'production',
              DEBUG: false,
              DATABASE_HOST: 'prod-db.example.com',
            }),
          ],
        }),
      ],
      providers: [UsersService],
    }).compile();

    service = module.get<UsersService>(UsersService);
  });

  it('should disable debug mode', () => {
    expect(service.isDebugEnabled()).toBe(false);
  });
});
```

**Parameterized Tests:**

```typescript
describe('UsersService with different environments', () => {
  // ✅ Test multiple configurations
  describe.each([
    {
      env: 'development',
      config: { NODE_ENV: 'development', DEBUG: true, DATABASE_HOST: 'localhost' },
      expected: { debug: true, host: 'localhost' },
    },
    {
      env: 'production',
      config: { NODE_ENV: 'production', DEBUG: false, DATABASE_HOST: 'prod-db' },
      expected: { debug: false, host: 'prod-db' },
    },
  ])('in $env environment', ({ config, expected }) => {
    let service: UsersService;

    beforeEach(async () => {
      const module = await Test.createTestingModule({
        imports: [
          ConfigModule.forRoot({
            ignoreEnvFile: true,
            load: [() => config],
          }),
        ],
        providers: [UsersService],
      }).compile();

      service = module.get<UsersService>(UsersService);
    });

    it(`should have debug=${expected.debug}`, () => {
      expect(service.isDebugEnabled()).toBe(expected.debug);
    });

    it(`should connect to ${expected.host}`, () => {
      expect(service.getDatabaseHost()).toBe(expected.host);
    });
  });
});
```

**Dynamic Mock Configuration:**

```typescript
describe('UsersService', () => {
  let mockConfigService: jest.Mocked<ConfigService>;

  beforeEach(async () => {
    mockConfigService = { get: jest.fn(), getOrThrow: jest.fn() } as any;

    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        { provide: ConfigService, useValue: mockConfigService },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
  });

  it('should work with small timeout', () => {
    mockConfigService.get.mockReturnValue(1000);
    expect(service.getTimeout()).toBe(1000);
  });

  it('should work with large timeout', () => {
    mockConfigService.get.mockReturnValue(30000);
    expect(service.getTimeout()).toBe(30000);
  });
});
```

**Configuration Factory:**

```typescript
// test/config/config-factory.ts
export class TestConfigFactory {
  static createDevelopmentConfig() {
    return () => ({
      NODE_ENV: 'development',
      DEBUG: true,
      DATABASE_HOST: 'localhost',
    });
  }

  static createProductionConfig() {
    return () => ({
      NODE_ENV: 'production',
      DEBUG: false,
      DATABASE_HOST: 'prod-db.example.com',
    });
  }

  static createCustomConfig(overrides: Record<string, any>) {
    return () => ({
      ...this.createTestConfig()(),
      ...overrides,
    });
  }
}

// Usage
import { TestConfigFactory } from '../test/config/config-factory';

describe('UsersService', () => {
  beforeEach(async () => {
    const module = await Test.createTestingModule({
      imports: [
        ConfigModule.forRoot({
          load: [TestConfigFactory.createDevelopmentConfig()],
        }),
      ],
      providers: [UsersService],
    }).compile();
  });
});
```

**Interview Tip**: Test different configs using **separate describe blocks** per environment or **parameterized tests** with `describe.each()`. Create **config factories** for reusability (dev, prod, test configs). Use **mockReturnValue()** to change mock values per test. **Best practices**: **ignoreEnvFile: true** to avoid .env dependencies, test **edge cases** (missing/invalid values), use **ConfigModule.forRoot()** with test config factories. Test **all environments** (development, production, test) and **feature flag combinations**. Clean up between tests with `afterEach()`.

</details>
