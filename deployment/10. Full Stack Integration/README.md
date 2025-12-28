# Full Stack Deployment Integration

## Table of Contents
1. [Complete Architecture](#complete-architecture)
2. [End-to-End Deployment Flow](#end-to-end-deployment-flow)
3. [React Frontend Deployment](#react-frontend-deployment)
4. [NestJS Backend Deployment](#nestjs-backend-deployment)
5. [PostgreSQL Database Deployment](#postgresql-database-deployment)
6. [Connecting All Components](#connecting-all-components)
7. [Production Deployment Scenarios](#production-deployment-scenarios)
8. [Deployment Strategies](#deployment-strategies)
9. [Troubleshooting](#troubleshooting)
10. [Complete Example Project](#complete-example-project)

---

## Complete Architecture

### Full Stack Application Components
```
┌─────────────────────────────────────────────────────────┐
│                      Internet                           │
└──────────────────────┬──────────────────────────────────┘
                       │
        ┌──────────────▼──────────────┐
        │   DNS (Route 53, Cloudflare) │
        │   myapp.com                  │
        └──────────┬──────────────────┘
                   │
        ┌──────────▼──────────────┐
        │   CDN (CloudFront)      │
        │   Cache static assets   │
        └──────────┬──────────────┘
                   │
         ┌─────────┴─────────┐
         │                   │
    ┌────▼─────┐      ┌─────▼──────┐
    │   S3 /   │      │  ALB / API  │
    │  Vercel  │      │   Gateway   │
    │ Frontend │      └─────┬───────┘
    │ (React)  │            │
    └──────────┘      ┌─────▼────────┐
                      │ EC2 / ECS /  │
                      │  Lambda      │
                      │   Backend    │
                      │  (NestJS)    │
                      └─────┬────────┘
                            │
                   ┌────────┴────────┐
                   │                 │
            ┌──────▼──────┐   ┌─────▼──────┐
            │     RDS     │   │   Redis    │
            │ PostgreSQL  │   │   Cache    │
            │  Database   │   └────────────┘
            └─────────────┘
```

### Component Interaction
```
1. User visits myapp.com
   ↓
2. DNS resolves to CloudFront
   ↓
3. CloudFront serves cached assets (or fetches from S3)
   ↓
4. React app loads in browser
   ↓
5. React makes API calls to api.myapp.com
   ↓
6. ALB routes to backend instances
   ↓
7. NestJS backend processes request
   ↓
8. Backend queries PostgreSQL database
   ↓
9. Backend checks Redis cache
   ↓
10. Response sent back through chain
    ↓
11. React updates UI
```

---

## End-to-End Deployment Flow

### Development to Production Pipeline
```
Developer Workflow:
1. Write code locally
2. Commit to Git
3. Push to GitHub
   ↓
CI/CD Pipeline Triggers:
4. GitHub Actions/GitLab CI runs
5. Install dependencies
6. Run tests (unit, integration)
7. Build application
8. Run security scans
   ↓
Build Artifacts:
9. Create Docker images
10. Tag with version
11. Push to registry (ECR, Docker Hub)
    ↓
Deployment:
12. Deploy to staging
13. Run smoke tests
14. Manual approval (optional)
15. Deploy to production
    ↓
Post-Deployment:
16. Health checks
17. Monitor metrics
18. Alert on errors
```

### Complete CI/CD Configuration
```yaml
# .github/workflows/deploy.yml
name: Deploy Full Stack Application

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '18.x'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # Job 1: Build and Test Frontend
  frontend:
    name: Frontend - Build & Test
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./frontend

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          cache-dependency-path: frontend/package-lock.json

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run tests
        run: npm test -- --coverage

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./frontend/coverage/lcov.info

      - name: Build
        run: npm run build
        env:
          REACT_APP_API_URL: ${{ secrets.API_URL }}
          REACT_APP_ENV: production

      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: frontend-build
          path: frontend/build
          retention-days: 7

  # Job 2: Build and Test Backend
  backend:
    name: Backend - Build & Test
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: testdb
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      
      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    defaults:
      run:
        working-directory: ./backend

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          cache-dependency-path: backend/package-lock.json

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run migrations
        run: npm run migration:run
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/testdb

      - name: Run tests
        run: npm test -- --coverage
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/testdb
          REDIS_URL: redis://localhost:6379

      - name: Build
        run: npm run build

      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: backend-build
          path: backend/dist
          retention-days: 7

  # Job 3: Build Docker Images
  docker-build:
    name: Build Docker Images
    needs: [frontend, backend]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Log in to registry
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

      - name: Build and push frontend image
        uses: docker/build-push-action@v4
        with:
          context: ./frontend
          push: true
          tags: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}-frontend:latest
          cache-from: type=registry,ref=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}-frontend:buildcache
          cache-to: type=registry,ref=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}-frontend:buildcache,mode=max

      - name: Build and push backend image
        uses: docker/build-push-action@v4
        with:
          context: ./backend
          push: true
          tags: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}-backend:latest
          cache-from: type=registry,ref=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}-backend:buildcache
          cache-to: type=registry,ref=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}-backend:buildcache,mode=max

  # Job 4: Deploy to Staging
  deploy-staging:
    name: Deploy to Staging
    needs: docker-build
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.myapp.com
    if: github.ref == 'refs/heads/develop'

    steps:
      - name: Deploy to staging
        run: |
          # Deploy frontend to S3
          aws s3 sync ./frontend/build s3://myapp-staging-frontend
          
          # Deploy backend to ECS
          aws ecs update-service \
            --cluster myapp-staging \
            --service backend \
            --force-new-deployment

      - name: Run smoke tests
        run: |
          curl -f https://staging-api.myapp.com/health || exit 1

  # Job 5: Deploy to Production
  deploy-production:
    name: Deploy to Production
    needs: docker-build
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://myapp.com
    if: github.ref == 'refs/heads/main'

    steps:
      - name: Deploy frontend to production
        run: |
          # Deploy to S3
          aws s3 sync ./frontend/build s3://myapp-production-frontend
          
          # Invalidate CloudFront cache
          aws cloudfront create-invalidation \
            --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} \
            --paths "/*"

      - name: Deploy backend to production
        run: |
          # Blue-green deployment with ECS
          aws ecs update-service \
            --cluster myapp-production \
            --service backend \
            --force-new-deployment

      - name: Run health checks
        run: |
          curl -f https://api.myapp.com/health || exit 1

      - name: Notify team
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: 'Production deployment completed! 🚀'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

---

## React Frontend Deployment

### Build Configuration
```javascript
// package.json
{
  "name": "myapp-frontend",
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "build:staging": "REACT_APP_ENV=staging npm run build",
    "build:production": "REACT_APP_ENV=production npm run build",
    "test": "react-scripts test --coverage",
    "lint": "eslint src/**/*.{ts,tsx}"
  }
}

// .env.production
REACT_APP_API_URL=https://api.myapp.com
REACT_APP_ENV=production
REACT_APP_SENTRY_DSN=https://...

// .env.staging
REACT_APP_API_URL=https://staging-api.myapp.com
REACT_APP_ENV=staging
```

### Environment Configuration
```typescript
// src/config/environment.ts
interface EnvironmentConfig {
  apiUrl: string;
  environment: 'development' | 'staging' | 'production';
  sentryDsn?: string;
  enableAnalytics: boolean;
}

const getConfig = (): EnvironmentConfig => {
  const env = process.env.REACT_APP_ENV || 'development';

  const configs: Record<string, EnvironmentConfig> = {
    development: {
      apiUrl: 'http://localhost:4000',
      environment: 'development',
      enableAnalytics: false
    },
    staging: {
      apiUrl: process.env.REACT_APP_API_URL!,
      environment: 'staging',
      sentryDsn: process.env.REACT_APP_SENTRY_DSN,
      enableAnalytics: false
    },
    production: {
      apiUrl: process.env.REACT_APP_API_URL!,
      environment: 'production',
      sentryDsn: process.env.REACT_APP_SENTRY_DSN,
      enableAnalytics: true
    }
  };

  return configs[env];
};

export const config = getConfig();
```

### API Service
```typescript
// src/services/api.ts
import axios from 'axios';
import { config } from '../config/environment';

const api = axios.create({
  baseURL: config.apiUrl,
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json'
  }
});

// Request interceptor (add auth token)
api.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('access_token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// Response interceptor (handle errors)
api.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;

    // Refresh token if 401
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;
      
      try {
        const refreshToken = localStorage.getItem('refresh_token');
        const response = await axios.post(
          `${config.apiUrl}/auth/refresh`,
          { refreshToken }
        );
        
        const { accessToken } = response.data;
        localStorage.setItem('access_token', accessToken);
        
        originalRequest.headers.Authorization = `Bearer ${accessToken}`;
        return api(originalRequest);
      } catch (refreshError) {
        // Redirect to login
        localStorage.clear();
        window.location.href = '/login';
        return Promise.reject(refreshError);
      }
    }

    return Promise.reject(error);
  }
);

export default api;
```

### Deployment Options

#### Option 1: Vercel
```bash
# Install Vercel CLI
npm install -g vercel

# Deploy
cd frontend
vercel --prod

# Environment variables
vercel env add REACT_APP_API_URL production
vercel env add REACT_APP_SENTRY_DSN production
```

#### Option 2: Netlify
```bash
# Install Netlify CLI
npm install -g netlify-cli

# Deploy
cd frontend
netlify deploy --prod

# netlify.toml
[build]
  command = "npm run build"
  publish = "build"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

#### Option 3: AWS S3 + CloudFront
```bash
# Build
npm run build:production

# Upload to S3
aws s3 sync build/ s3://myapp-frontend --delete

# Invalidate CloudFront cache
aws cloudfront create-invalidation \
  --distribution-id E123456789 \
  --paths "/*"
```

---

## NestJS Backend Deployment

### Production Build
```typescript
// package.json
{
  "name": "myapp-backend",
  "scripts": {
    "build": "nest build",
    "start": "node dist/main",
    "start:dev": "nest start --watch",
    "start:prod": "node dist/main",
    "migration:generate": "typeorm migration:generate",
    "migration:run": "typeorm migration:run",
    "migration:revert": "typeorm migration:revert"
  }
}

// tsconfig.json
{
  "compilerOptions": {
    "module": "commonjs",
    "declaration": true,
    "removeComments": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "allowSyntheticDefaultImports": true,
    "target": "ES2021",
    "sourceMap": false,
    "outDir": "./dist",
    "baseUrl": "./",
    "incremental": true,
    "skipLibCheck": true,
    "strictNullChecks": false,
    "noImplicitAny": false,
    "strictBindCallApply": false,
    "forceConsistentCasingInFileNames": false,
    "noFallthroughCasesInSwitch": false
  }
}
```

### Environment Configuration
```typescript
// src/config/configuration.ts
export default () => ({
  port: parseInt(process.env.PORT, 10) || 4000,
  nodeEnv: process.env.NODE_ENV || 'development',
  
  database: {
    type: 'postgres' as const,
    host: process.env.DATABASE_HOST,
    port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
    username: process.env.DATABASE_USERNAME,
    password: process.env.DATABASE_PASSWORD,
    database: process.env.DATABASE_NAME,
    ssl: process.env.DATABASE_SSL === 'true' ? {
      rejectUnauthorized: false
    } : false,
    synchronize: false, // Never true in production!
    logging: process.env.NODE_ENV === 'development',
    entities: ['dist/**/*.entity.js'],
    migrations: ['dist/migrations/*.js']
  },
  
  redis: {
    host: process.env.REDIS_HOST,
    port: parseInt(process.env.REDIS_PORT, 10) || 6379,
    password: process.env.REDIS_PASSWORD
  },
  
  jwt: {
    secret: process.env.JWT_SECRET,
    expiresIn: '15m'
  },
  
  cors: {
    origin: process.env.CORS_ORIGIN?.split(',') || ['http://localhost:3000'],
    credentials: true
  }
});
```

### Dockerfile
```dockerfile
# Multi-stage build
FROM node:18-alpine AS build

WORKDIR /app

# Copy package files
COPY package*.json ./
RUN npm ci

# Copy source
COPY . .

# Build
RUN npm run build

# Production stage
FROM node:18-alpine AS production

WORKDIR /app

# Copy package files
COPY package*.json ./
RUN npm ci --only=production

# Copy built app
COPY --from=build /app/dist ./dist

# Create non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
RUN chown -R appuser:appgroup /app
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
  CMD node -e "require('http').get('http://localhost:4000/health', (r) => { process.exit(r.statusCode === 200 ? 0 : 1) })"

EXPOSE 4000

CMD ["node", "dist/main.js"]
```

### Deployment Options

#### Option 1: Heroku
```bash
# Create Heroku app
heroku create myapp-backend

# Add PostgreSQL
heroku addons:create heroku-postgresql:mini

# Add Redis
heroku addons:create heroku-redis:mini

# Set environment variables
heroku config:set NODE_ENV=production
heroku config:set JWT_SECRET=your_secret_here

# Deploy
git push heroku main

# Run migrations
heroku run npm run migration:run
```

#### Option 2: AWS ECS (Elastic Container Service)
```bash
# Build and push Docker image
docker build -t myapp-backend .
docker tag myapp-backend:latest 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp-backend:latest
docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp-backend:latest

# Create ECS task definition (task-definition.json)
{
  "family": "myapp-backend",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "containerDefinitions": [{
    "name": "backend",
    "image": "123456789.dkr.ecr.us-east-1.amazonaws.com/myapp-backend:latest",
    "portMappings": [{
      "containerPort": 4000,
      "protocol": "tcp"
    }],
    "environment": [
      { "name": "NODE_ENV", "value": "production" }
    ],
    "secrets": [
      {
        "name": "DATABASE_URL",
        "valueFrom": "arn:aws:secretsmanager:us-east-1:123456789:secret:prod/db-url"
      }
    ],
    "logConfiguration": {
      "logDriver": "awslogs",
      "options": {
        "awslogs-group": "/ecs/myapp-backend",
        "awslogs-region": "us-east-1",
        "awslogs-stream-prefix": "ecs"
      }
    }
  }]
}

# Register task definition
aws ecs register-task-definition --cli-input-json file://task-definition.json

# Create service
aws ecs create-service \
  --cluster myapp-cluster \
  --service-name backend \
  --task-definition myapp-backend \
  --desired-count 2 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-xxx],securityGroups=[sg-xxx],assignPublicIp=ENABLED}"
```

#### Option 3: DigitalOcean App Platform
```yaml
# .do/app.yaml
name: myapp-backend
services:
  - name: api
    github:
      repo: username/myapp
      branch: main
    source_dir: /backend
    build_command: npm run build
    run_command: npm start
    environment_slug: node-js
    instance_count: 2
    instance_size_slug: basic-xs
    routes:
      - path: /
    envs:
      - key: NODE_ENV
        value: production
      - key: DATABASE_URL
        value: ${db.DATABASE_URL}
      - key: REDIS_URL
        value: ${redis.DATABASE_URL}
      - key: JWT_SECRET
        value: ${JWT_SECRET}
        type: SECRET

databases:
  - name: db
    engine: PG
    version: "14"

  - name: redis
    engine: REDIS
    version: "7"
```

---

## PostgreSQL Database Deployment

### Database Migration
```typescript
// Create migration
// npm run migration:generate -- -n CreateUsersTable

// migrations/1234567890123-CreateUsersTable.ts
import { MigrationInterface, QueryRunner, Table } from 'typeorm';

export class CreateUsersTable1234567890123 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.createTable(
      new Table({
        name: 'users',
        columns: [
          {
            name: 'id',
            type: 'uuid',
            isPrimary: true,
            generationStrategy: 'uuid',
            default: 'uuid_generate_v4()'
          },
          {
            name: 'email',
            type: 'varchar',
            isUnique: true
          },
          {
            name: 'password',
            type: 'varchar'
          },
          {
            name: 'name',
            type: 'varchar'
          },
          {
            name: 'created_at',
            type: 'timestamp',
            default: 'now()'
          },
          {
            name: 'updated_at',
            type: 'timestamp',
            default: 'now()'
          }
        ]
      }),
      true
    );

    // Create index
    await queryRunner.createIndex(
      'users',
      {
        name: 'IDX_USERS_EMAIL',
        columnNames: ['email']
      }
    );
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropTable('users');
  }
}
```

### Database Seeds
```typescript
// seeds/initial-data.seed.ts
import { DataSource } from 'typeorm';
import { User } from '../entities/user.entity';
import * as bcrypt from 'bcrypt';

export class InitialDataSeed {
  public async run(dataSource: DataSource): Promise<void> {
    const userRepository = dataSource.getRepository(User);

    // Create admin user
    const adminPassword = await bcrypt.hash('Admin123!', 10);
    await userRepository.save({
      email: 'admin@example.com',
      password: adminPassword,
      name: 'Admin User',
      role: 'admin'
    });

    console.log('Seed data created successfully');
  }
}
```

### Deployment Options

#### Option 1: AWS RDS
```bash
# Create RDS instance
aws rds create-db-instance \
  --db-instance-identifier myapp-db \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --engine-version 14.7 \
  --master-username admin \
  --master-user-password YourPasswordHere \
  --allocated-storage 20 \
  --vpc-security-group-ids sg-xxx \
  --db-subnet-group-name myapp-db-subnet \
  --backup-retention-period 7 \
  --publicly-accessible false

# Get endpoint
aws rds describe-db-instances \
  --db-instance-identifier myapp-db \
  --query 'DBInstances[0].Endpoint.Address'

# Connection string
postgresql://admin:password@myapp-db.xxx.rds.amazonaws.com:5432/myapp
```

#### Option 2: Heroku Postgres
```bash
# Add PostgreSQL addon
heroku addons:create heroku-postgresql:mini

# Get connection string
heroku config:get DATABASE_URL

# Connect
heroku pg:psql

# Backups (automatic with paid plans)
heroku pg:backups:schedule --at '02:00 America/New_York'
```

#### Option 3: DigitalOcean Managed Database
```bash
# Create via UI or API
doctl databases create myapp-db \
  --engine pg \
  --version 14 \
  --region nyc1 \
  --size db-s-1vcpu-1gb \
  --num-nodes 1

# Get connection details
doctl databases connection myapp-db
```

### Database Backup Strategy
```bash
# Automated backup script
#!/bin/bash

# backup-db.sh
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups"
DB_NAME="myapp"
BACKUP_FILE="$BACKUP_DIR/${DB_NAME}_${DATE}.sql.gz"

# Create backup
pg_dump $DATABASE_URL | gzip > $BACKUP_FILE

# Upload to S3
aws s3 cp $BACKUP_FILE s3://myapp-backups/

# Keep only last 30 days
find $BACKUP_DIR -name "*.sql.gz" -mtime +30 -delete

# Restore from backup
gunzip < backup.sql.gz | psql $DATABASE_URL
```

---

## Connecting All Components

### Full Stack Connection Flow

#### 1. Frontend to Backend Connection
```typescript
// Frontend: src/services/api.ts
import axios from 'axios';

const api = axios.create({
  baseURL: process.env.REACT_APP_API_URL, // https://api.myapp.com
  withCredentials: true, // Send cookies
  headers: {
    'Content-Type': 'application/json'
  }
});

// Make request
const response = await api.get('/users');
```

#### 2. Backend to Database Connection
```typescript
// Backend: src/app.module.ts
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      url: process.env.DATABASE_URL,
      // Or separate params
      host: process.env.DATABASE_HOST,
      port: parseInt(process.env.DATABASE_PORT),
      username: process.env.DATABASE_USERNAME,
      password: process.env.DATABASE_PASSWORD,
      database: process.env.DATABASE_NAME,
      entities: [__dirname + '/**/*.entity{.ts,.js}'],
      synchronize: false, // Never true in production!
      ssl: process.env.NODE_ENV === 'production' ? {
        rejectUnauthorized: false
      } : false
    })
  ]
})
export class AppModule {}
```

#### 3. Backend to Redis Connection
```typescript
// Backend: src/cache/cache.module.ts
import { CacheModule } from '@nestjs/cache-manager';
import * as redisStore from 'cache-manager-redis-store';

@Module({
  imports: [
    CacheModule.register({
      store: redisStore,
      url: process.env.REDIS_URL,
      ttl: 300 // 5 minutes default
    })
  ]
})
export class CacheConfigModule {}
```

### Environment Variables Setup

#### Frontend (.env.production)
```bash
REACT_APP_API_URL=https://api.myapp.com
REACT_APP_ENV=production
REACT_APP_SENTRY_DSN=https://xxx@sentry.io/123
```

#### Backend (.env.production)
```bash
NODE_ENV=production
PORT=4000

# Database
DATABASE_URL=postgresql://user:password@host:5432/dbname
# Or separate
DATABASE_HOST=myapp-db.xxx.rds.amazonaws.com
DATABASE_PORT=5432
DATABASE_USERNAME=admin
DATABASE_PASSWORD=secure_password
DATABASE_NAME=myapp
DATABASE_SSL=true

# Redis
REDIS_URL=redis://:password@host:6379
# Or separate
REDIS_HOST=myapp-redis.xxx.cache.amazonaws.com
REDIS_PORT=6379
REDIS_PASSWORD=redis_password

# Auth
JWT_SECRET=your_super_secret_jwt_key_here
JWT_REFRESH_SECRET=your_refresh_secret_here

# CORS
CORS_ORIGIN=https://myapp.com,https://www.myapp.com

# External Services
AWS_ACCESS_KEY_ID=xxx
AWS_SECRET_ACCESS_KEY=xxx
AWS_REGION=us-east-1
S3_BUCKET=myapp-uploads

SENDGRID_API_KEY=xxx
STRIPE_SECRET_KEY=xxx

# Monitoring
SENTRY_DSN=https://xxx@sentry.io/123
```

### Docker Compose (Development)
```yaml
# docker-compose.yml
version: '3.8'

services:
  frontend:
    build:
      context: ./frontend
      target: development
    ports:
      - "3000:3000"
    volumes:
      - ./frontend:/app
      - /app/node_modules
    environment:
      - REACT_APP_API_URL=http://localhost:4000
    depends_on:
      - backend

  backend:
    build:
      context: ./backend
      target: development
    ports:
      - "4000:4000"
    volumes:
      - ./backend:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://postgres:password@postgres:5432/myapp
      - REDIS_URL=redis://redis:6379
      - JWT_SECRET=dev_secret
    depends_on:
      - postgres
      - redis

  postgres:
    image: postgres:14-alpine
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=myapp
    volumes:
      - postgres-data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data

volumes:
  postgres-data:
  redis-data:
```

---

## How Components Connect After Deployment

### Detailed Connection Architecture

#### Complete Request Flow (Production)
```
User Browser
    │
    │ (1) User visits https://myapp.com
    │
    ▼
┌─────────────────────┐
│   DNS Provider      │  Resolves myapp.com → CloudFront IP
│  (Route 53/GoDaddy) │
└──────────┬──────────┘
           │
           │ (2) HTTPS request to CDN
           │
           ▼
┌─────────────────────┐
│   CDN/CloudFront    │  • Caches static files (JS, CSS, images)
│  (Edge Locations)   │  • Serves from nearest location
└──────────┬──────────┘  • Returns cached or fetches from origin
           │
           ├──────────────────────────┐
           │                          │
           │ (3a) Static files        │ (3b) API calls
           │     (Cached)             │      (Proxied)
           │                          │
           ▼                          ▼
┌─────────────────────┐    ┌─────────────────────┐
│   S3 / Netlify     │    │  Load Balancer      │
│   (React Build)     │    │  (ALB/ELB/Nginx)    │
│                     │    └──────────┬──────────┘
│  • index.html       │               │
│  • main.js          │               │ (4) Routes to healthy backend
│  • styles.css       │               │     instance (Round-robin/
│  • images/          │               │     Least connections)
└─────────────────────┘               │
                                      ▼
                           ┌─────────────────────┐
                           │  Backend Instance 1 │ ◄─┐
                           │    (NestJS)         │   │
                           └──────────┬──────────┘   │
                                      │               │ Auto-scaling
                           ┌──────────▼──────────┐   │ (based on CPU,
                           │  Backend Instance 2 │   │  memory, etc.)
                           │    (NestJS)         │   │
                           └──────────┬──────────┘   │
                                      │               │
                           ┌──────────▼──────────┐   │
                           │  Backend Instance N │ ◄─┘
                           │    (NestJS)         │
                           └──────────┬──────────┘
                                      │
                         ┌────────────┴────────────┐
                         │                         │
                         │ (5) Check cache first   │ (6) Query if cache miss
                         │                         │
                         ▼                         ▼
              ┌──────────────────┐    ┌──────────────────────┐
              │   Redis Cache    │    │   PostgreSQL RDS     │
              │                  │    │                      │
              │ • Session data   │    │ • Users table        │
              │ • API cache      │    │ • Posts table        │
              │ • Rate limiting  │    │ • Relationships      │
              └──────────────────┘    │ • Transactions       │
                                      └──────────────────────┘
```

### Step-by-Step Connection Details

#### Step 1: DNS Resolution
```
User enters: https://myapp.com
                │
                ▼
Browser queries DNS:
  What is the IP address of myapp.com?
                │
                ▼
DNS responds:
  myapp.com → 52.84.123.456 (CloudFront IP)
  
Time: ~10-50ms
Cached: Yes (browser caches DNS for TTL period)
```

#### Step 2: SSL/TLS Handshake
```
Browser → CDN: Hello, I want HTTPS
          CDN → Browser: Here's my SSL certificate
                         (issued by Let's Encrypt/ACM)
Browser: Certificate valid? ✓
         Establishes encrypted connection
         
Time: ~50-100ms (first time)
      ~0ms (if session resumed)
```

#### Step 3: Frontend Loading
```
Browser requests: https://myapp.com/
                         │
                         ▼
            CloudFront checks cache:
                    │
        ┌───────────┴───────────┐
        │                       │
    Cache Hit              Cache Miss
        │                       │
    Return cached          Fetch from S3
    (0-5ms)                (20-50ms)
        │                       │
        └───────────┬───────────┘
                    │
            Returns index.html
                    │
                    ▼
Browser parses HTML, requests assets:
  • /static/js/main.abc123.js
  • /static/css/main.xyz789.css
  • /static/media/logo.png
  
(All served from CloudFront cache)

Total load time: 200-500ms (first visit)
                 50-100ms (return visit, cached)
```

#### Step 4: Frontend to Backend API Connection

**4a. Initial API Request**
```javascript
// Frontend makes API call
// src/services/api.ts

const response = await axios.get('https://api.myapp.com/users');

// Behind the scenes:
```

```
Browser (React App)
    │
    │ HTTP GET https://api.myapp.com/users
    │ Headers:
    │   Authorization: Bearer eyJhbGc...
    │   Content-Type: application/json
    │   Origin: https://myapp.com
    │
    ▼
DNS Resolution (api.myapp.com)
    │
    │ Resolves to Load Balancer IP
    │
    ▼
┌─────────────────────────────┐
│  Application Load Balancer  │
│  (ALB on AWS)                │
│                              │
│  1. Checks health of backends│
│  2. Routes based on:         │
│     - Path rules             │
│     - Headers                │
│     - Round-robin            │
└──────────────┬───────────────┘
               │
               │ Forwards to healthy instance
               │
               ▼
┌─────────────────────────────┐
│  Backend Instance (NestJS)  │
│  Private IP: 10.0.1.50:4000 │
│                              │
│  1. Receives request         │
│  2. Validates JWT token      │
│  3. Checks authorization     │
│  4. Processes request        │
└──────────────┬───────────────┘
               │
               │ (See Step 5 & 6)
```

**4b. CORS Handling**
```typescript
// Backend: main.ts
app.enableCors({
  origin: [
    'https://myapp.com',
    'https://www.myapp.com'
  ],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
  allowedHeaders: ['Content-Type', 'Authorization']
});

// How it works:
// 1. Browser sends OPTIONS preflight request
// 2. Backend responds with allowed origins/methods
// 3. If allowed, browser sends actual request
// 4. Backend includes CORS headers in response
```

**4c. Authentication Flow**
```
Login Request:
POST https://api.myapp.com/auth/login
Body: { email, password }
    │
    ▼
Backend validates credentials
    │
    ▼
Generates JWT tokens:
  - Access Token (15 min)
  - Refresh Token (7 days)
    │
    ▼
Response:
{
  "accessToken": "eyJhbGc...",
  "refreshToken": "eyJhbGc...",
  "user": { id, email, name }
}
    │
    ▼
Frontend stores tokens:
  - Access token: Memory/Redux
  - Refresh token: httpOnly cookie
    │
    ▼
Subsequent requests include:
  Authorization: Bearer eyJhbGc...
```

#### Step 5: Backend to Cache (Redis) Connection

**5a. Connection Setup**
```typescript
// Backend connects to Redis on startup
// src/main.ts

const redisClient = new Redis({
  host: process.env.REDIS_HOST,     // myapp-redis.xxx.cache.amazonaws.com
  port: parseInt(process.env.REDIS_PORT), // 6379
  password: process.env.REDIS_PASSWORD,
  retryStrategy: (times) => {
    return Math.min(times * 50, 2000);
  },
  maxRetriesPerRequest: 3,
  enableReadyCheck: true
});

// Connection pool maintained
// Multiple backend instances share same Redis
```

**5b. Cache Check Flow**
```typescript
// Example: Get user profile
async getUserProfile(userId: string) {
  // 1. Check Redis cache first
  const cacheKey = `user:${userId}`;
  const cached = await this.redis.get(cacheKey);
  
  if (cached) {
    console.log('Cache hit!');
    return JSON.parse(cached);
  }
  
  // 2. Cache miss - query database
  console.log('Cache miss - querying database');
  const user = await this.userRepository.findOne(userId);
  
  // 3. Store in cache for future requests
  await this.redis.set(
    cacheKey,
    JSON.stringify(user),
    'EX',
    3600 // Expire after 1 hour
  );
  
  return user;
}

// Performance:
// Cache hit:  ~1-5ms
// Cache miss: ~50-200ms (includes DB query)
```

**5c. Session Management**
```typescript
// Store session in Redis
await redis.set(
  `session:${sessionId}`,
  JSON.stringify({
    userId: user.id,
    loginTime: Date.now(),
    ipAddress: req.ip
  }),
  'EX',
  86400 // 24 hours
);

// Multiple backend instances can access same session
// User can be authenticated on any instance
```

**5d. Rate Limiting with Redis**
```typescript
// Track API requests per user
const key = `rate-limit:${userId}:${endpoint}`;
const requests = await redis.incr(key);

if (requests === 1) {
  // First request - set expiry
  await redis.expire(key, 60); // 60 seconds window
}

if (requests > 100) {
  throw new TooManyRequestsException();
}
```

#### Step 6: Backend to Database Connection

**6a. Connection Pool**
```typescript
// Backend establishes connection pool on startup
// TypeORM configuration

{
  type: 'postgres',
  host: 'myapp-db.xxx.rds.amazonaws.com',
  port: 5432,
  username: 'app_user',
  password: process.env.DB_PASSWORD,
  database: 'myapp_production',
  
  // Connection pool settings
  extra: {
    max: 20,              // Maximum connections
    min: 5,               // Minimum connections
    idleTimeoutMillis: 30000,
    connectionTimeoutMillis: 2000,
  },
  
  // SSL (required for RDS)
  ssl: {
    rejectUnauthorized: false
  }
}

// How connection pool works:
// 1. Backend starts → creates 5 connections
// 2. Request comes → uses available connection
// 3. Query executes → connection returned to pool
// 4. High load → creates more connections (up to 20)
// 5. Low load → closes idle connections (down to 5)
```

**6b. Query Execution Flow**
```typescript
// Example: Create a post
async createPost(userId: string, data: CreatePostDto) {
  // 1. Get connection from pool (~0-1ms)
  
  // 2. Begin transaction
  return await this.dataSource.transaction(async (manager) => {
    
    // 3. Insert post
    const post = await manager.save(Post, {
      title: data.title,
      content: data.content,
      userId: userId,
      createdAt: new Date()
    });
    // Query: INSERT INTO posts (title, content, user_id, created_at) 
    //        VALUES ($1, $2, $3, $4) RETURNING *
    // Time: ~5-20ms
    
    // 4. Update user's post count
    await manager.increment(User, { id: userId }, 'postCount', 1);
    // Query: UPDATE users SET post_count = post_count + 1 
    //        WHERE id = $1
    // Time: ~5-15ms
    
    // 5. Commit transaction
    // Both queries succeed or both rollback
    
    return post;
  });
  // 6. Connection returned to pool
}

// Total time: ~15-50ms
```

**6c. Database Read Flow**
```typescript
// Example: Get posts with pagination
async getPosts(page: number, limit: number) {
  const [posts, total] = await this.postRepository.findAndCount({
    where: { published: true },
    relations: ['author', 'comments'],
    order: { createdAt: 'DESC' },
    skip: (page - 1) * limit,
    take: limit,
  });
  
  // Generated SQL:
  // SELECT posts.*, users.*, comments.*
  // FROM posts
  // LEFT JOIN users ON posts.user_id = users.id
  // LEFT JOIN comments ON comments.post_id = posts.id
  // WHERE posts.published = true
  // ORDER BY posts.created_at DESC
  // LIMIT 10 OFFSET 0
  
  // Execution:
  // 1. Database receives query
  // 2. Query planner optimizes
  // 3. Uses indexes for WHERE and ORDER BY
  // 4. Performs JOIN operations
  // 5. Returns result set
  
  // Time: 10-100ms (depends on data size and indexes)
  
  return {
    posts,
    total,
    page,
    totalPages: Math.ceil(total / limit)
  };
}
```

**6d. Connection Resilience**
```typescript
// Handle connection failures
@Module({
  imports: [
    TypeOrmModule.forRootAsync({
      useFactory: () => ({
        // ... config
        
        // Retry logic
        retryAttempts: 3,
        retryDelay: 3000,
        
        // Connection events
        logging: ['error', 'warn'],
        
        // Pool error handling
        poolErrorHandler: (err) => {
          logger.error('Database pool error', err);
          // Alert monitoring system
          alerting.send('Database pool error', err);
        }
      })
    })
  ]
})

// Automatic reconnection:
// 1. Connection lost
// 2. TypeORM detects failure
// 3. Waits 3 seconds
// 4. Attempts reconnection
// 5. Retries up to 3 times
// 6. If all fail, application alerts and may restart
```

### Network Security in Production

#### VPC Configuration (AWS Example)
```
┌─────────────────────────────────────────────────────┐
│                    VPC (10.0.0.0/16)                │
│                                                     │
│  ┌────────────────────────────────────────────┐   │
│  │         Public Subnet (10.0.1.0/24)        │   │
│  │                                             │   │
│  │  ┌──────────────┐      ┌──────────────┐   │   │
│  │  │     NAT      │      │     ALB      │   │   │
│  │  │   Gateway    │      │ (Port 443)   │   │   │
│  │  └──────────────┘      └──────┬───────┘   │   │
│  │         │                      │           │   │
│  └─────────┼──────────────────────┼───────────┘   │
│            │                      │               │
│  ┌─────────▼──────────────────────▼───────────┐   │
│  │      Private Subnet (10.0.2.0/24)          │   │
│  │                                             │   │
│  │  ┌──────────────┐      ┌──────────────┐   │   │
│  │  │   Backend    │      │   Backend    │   │   │
│  │  │ Instance 1   │      │ Instance 2   │   │   │
│  │  │  10.0.2.10   │      │  10.0.2.11   │   │   │
│  │  └──────┬───────┘      └──────┬───────┘   │   │
│  │         │                     │            │   │
│  └─────────┼─────────────────────┼────────────┘   │
│            │                     │                │
│  ┌─────────▼─────────────────────▼────────────┐   │
│  │      Private Subnet (10.0.3.0/24)          │   │
│  │         (Database Subnet)                   │   │
│  │                                             │   │
│  │  ┌──────────────┐      ┌──────────────┐   │   │
│  │  │ PostgreSQL   │      │    Redis     │   │   │
│  │  │    RDS       │      │  ElastiCache │   │   │
│  │  │  10.0.3.10   │      │  10.0.3.20   │   │   │
│  │  └──────────────┘      └──────────────┘   │   │
│  │                                             │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
└─────────────────────────────────────────────────────┘

Security Rules:
1. Public Subnet:
   - Inbound: 443 from 0.0.0.0/0 (internet)
   - Outbound: All

2. Backend Instances (Private):
   - Inbound: 4000 from ALB only
   - Outbound: All (through NAT for updates)

3. Database (Private):
   - Inbound: 5432 from Backend instances only
   - No outbound internet access

4. Redis (Private):
   - Inbound: 6379 from Backend instances only
   - No outbound internet access
```

### Load Balancing and Health Checks

#### How Load Balancer Routes Traffic
```
Request comes in:
    │
    ▼
┌─────────────────────────────┐
│  Application Load Balancer  │
│                              │
│  Active Backends:            │
│  ✓ Instance 1 (Healthy)     │ ◄─┐
│  ✓ Instance 2 (Healthy)     │   │ Health checks
│  ✗ Instance 3 (Unhealthy)   │   │ every 30s
│                              │   │
│  Routing Algorithm:          │   │
│  • Round-robin (default)     │   │
│  • Least connections         │ ──┘
│  • Sticky sessions (option)  │
└──────────────┬───────────────┘
               │
      ┌────────┼────────┐
      │        │        │
   Request  Request  Request
      1        2        3
      │        │        │
      ▼        ▼        ▼
   Instance Instance Skip
      1        2     Instance 3
                     (unhealthy)
```

#### Health Check Implementation
```typescript
// Backend: health.controller.ts
@Controller('health')
export class HealthController {
  constructor(
    @InjectConnection() private connection: Connection,
    @Inject('REDIS_CLIENT') private redis: Redis
  ) {}

  @Get()
  async check() {
    const health = {
      status: 'healthy',
      timestamp: new Date().toISOString(),
      uptime: process.uptime(),
      checks: {}
    };

    // Check database
    try {
      await this.connection.query('SELECT 1');
      health.checks['database'] = 'healthy';
    } catch (error) {
      health.checks['database'] = 'unhealthy';
      health.status = 'unhealthy';
    }

    // Check Redis
    try {
      await this.redis.ping();
      health.checks['redis'] = 'healthy';
    } catch (error) {
      health.checks['redis'] = 'unhealthy';
      health.status = 'unhealthy';
    }

    // Check memory
    const memUsage = process.memoryUsage();
    health.checks['memory'] = {
      rss: `${Math.round(memUsage.rss / 1024 / 1024)}MB`,
      heapUsed: `${Math.round(memUsage.heapUsed / 1024 / 1024)}MB`
    };

    const statusCode = health.status === 'healthy' ? 200 : 503;
    return { statusCode, ...health };
  }
}

// Load Balancer configuration:
// Health Check Path: /health
// Interval: 30 seconds
// Timeout: 5 seconds
// Healthy threshold: 2 consecutive successes
// Unhealthy threshold: 2 consecutive failures
```

### Complete Request-Response Cycle

#### Example: User Creates a Post
```
1. Frontend (React)
   ─────────────────────────────────────────────────
   const createPost = async (title, content) => {
     const response = await api.post('/posts', {
       title,
       content
     });
     return response.data;
   };
   
   Time: 0ms - Request initiated

2. DNS & SSL
   ─────────────────────────────────────────────────
   • DNS lookup (cached): 0ms
   • SSL handshake (reused): 0ms
   • Connection established: 0ms
   
   Time: 0ms - Connection ready

3. Load Balancer
   ─────────────────────────────────────────────────
   • Receives request at ALB
   • Checks backend health
   • Routes to Instance 2 (least connections)
   
   Time: 1-2ms - Routing decision

4. Backend - Authentication
   ─────────────────────────────────────────────────
   // auth.guard.ts
   • Extracts JWT from header
   • Verifies token signature
   • Checks expiration
   • Loads user from token
   
   Time: 5-10ms - Auth validation

5. Backend - Cache Check
   ─────────────────────────────────────────────────
   • Check Redis for user permissions
   • Cache hit: User has write permission
   
   Time: 1-3ms - Redis query

6. Backend - Validation
   ─────────────────────────────────────────────────
   // ValidationPipe
   • Validates DTO
   • Checks title length
   • Sanitizes content
   
   Time: 1-2ms - Input validation

7. Backend - Database Query
   ─────────────────────────────────────────────────
   // posts.service.ts
   • Get connection from pool: 0-1ms
   • Begin transaction
   • INSERT post: 10-20ms
   • UPDATE user stats: 5-10ms
   • Commit transaction
   • Return connection to pool
   
   Time: 20-40ms - Database operations

8. Backend - Cache Update
   ─────────────────────────────────────────────────
   // Invalidate related caches
   await redis.del(`user:${userId}:posts`);
   await redis.del(`posts:recent`);
   
   Time: 2-4ms - Cache invalidation

9. Backend - Response
   ─────────────────────────────────────────────────
   return {
     statusCode: 201,
     message: 'Post created',
     data: post
   };
   
   Time: 1ms - JSON serialization

10. Load Balancer → Frontend
    ─────────────────────────────────────────────────
    • Response through ALB
    • Compressed (gzip)
    • SSL encrypted
    
    Time: 1-2ms - Network transfer

11. Frontend Receives Response
    ─────────────────────────────────────────────────
    • Parse JSON
    • Update Redux store
    • Re-render component
    • Show success message
    
    Time: 5-10ms - UI update

═════════════════════════════════════════════════════
Total Time: 35-75ms (end-to-end)

Breakdown:
• Network: ~5ms
• Authentication: ~10ms
• Database: ~30ms
• Cache: ~5ms
• Processing: ~5ms
```

### Monitoring the Connections

#### Application Performance Monitoring
```typescript
// Track all connection metrics
import * as newrelic from 'newrelic';

// Database query monitoring
const startTime = Date.now();
const result = await this.postRepository.find();
const duration = Date.now() - startTime;

newrelic.recordMetric('Custom/Database/QueryTime', duration);

if (duration > 100) {
  logger.warn('Slow database query', {
    query: 'posts.find',
    duration,
    threshold: 100
  });
}

// Redis monitoring
const cacheStart = Date.now();
const cached = await redis.get(key);
const cacheDuration = Date.now() - cacheStart;

newrelic.recordMetric('Custom/Redis/ResponseTime', cacheDuration);

// API endpoint monitoring
@Controller('posts')
export class PostsController {
  @Get()
  @UseInterceptors(PerformanceInterceptor)
  async getPosts() {
    // Interceptor tracks timing automatically
  }
}
```

#### Connection Pool Monitoring
```typescript
// Monitor database connection pool
setInterval(() => {
  const pool = dataSource.driver.master;
  
  const metrics = {
    totalConnections: pool.totalCount,
    activeConnections: pool.activeCount,
    idleConnections: pool.idleCount,
    waitingClients: pool.waitingCount
  };
  
  logger.info('Connection pool stats', metrics);
  
  // Alert if pool exhausted
  if (metrics.waitingClients > 5) {
    alerting.send('Database connection pool exhausted', metrics);
  }
}, 60000); // Every minute
```

### Scaling Considerations

#### Horizontal Scaling
```
Single Instance:
┌──────────────┐
│   Backend    │  ← All traffic
│  (1 instance)│     Max: 1000 req/s
└──────────────┘

Auto-Scaling:
                    ┌──────────────┐
                    │   Backend 1  │  ← 25% traffic
                    └──────────────┘
                    ┌──────────────┐
┌─────────┐         │   Backend 2  │  ← 25% traffic
│   ALB   │ ───────►└──────────────┘
└─────────┘         ┌──────────────┐
                    │   Backend 3  │  ← 25% traffic
                    └──────────────┘
                    ┌──────────────┐
                    │   Backend 4  │  ← 25% traffic
                    └──────────────┘
                    
Total capacity: 4000 req/s
Redundancy: Yes (if one fails, others handle load)
```

#### Database Connection Scaling
```typescript
// Each backend instance has its own connection pool

Instance 1:  [DB Pool: 20 connections] ─┐
Instance 2:  [DB Pool: 20 connections] ─┤
Instance 3:  [DB Pool: 20 connections] ─┼─► PostgreSQL
Instance 4:  [DB Pool: 20 connections] ─┘   (Max: 100 connections)

// Configuration:
{
  extra: {
    max: 20,  // Per instance
  }
}

// Database can handle:
// - 4 instances × 20 connections = 80 connections
// - Leaves 20 for admin/maintenance
// - If scaling to 5 instances, reduce to max: 15
```

---

## Production Deployment Scenarios

### Scenario 1: Budget-Friendly (< $50/month)
```
Frontend: Vercel (Free)
Backend: Heroku Dyno ($7/month)
Database: Heroku Postgres Mini ($5/month)
Redis: Heroku Redis Mini ($3/month)
Monitoring: Sentry Free Tier

Total: ~$15/month

Steps:
1. Deploy frontend to Vercel
2. Deploy backend to Heroku
3. Add PostgreSQL addon
4. Add Redis addon
5. Configure environment variables
```

### Scenario 2: Production-Ready (< $200/month)
```
Frontend: Vercel Pro ($20/month)
Backend: DigitalOcean App Platform ($12/month)
Database: DigitalOcean Managed DB ($15/month)
Redis: DigitalOcean Managed Redis ($15/month)
CDN: CloudFlare Free
Monitoring: Datadog ($15/month)

Total: ~$77/month

Benefits:
- Better performance
- Managed services
- Auto-scaling
- Professional monitoring
```

### Scenario 3: Enterprise Scale (Flexible)
```
Frontend: AWS S3 + CloudFront
Backend: AWS ECS Fargate (Auto-scaling)
Database: AWS RDS (Multi-AZ)
Cache: AWS ElastiCache Redis
CDN: CloudFront
Monitoring: AWS CloudWatch + Datadog
Secrets: AWS Secrets Manager

Estimated: $200-2000/month (based on traffic)

Benefits:
- High availability
- Global distribution
- Auto-scaling
- Enterprise security
- Professional support
```

---

## Deployment Strategies

### Strategy 1: All-in-One Platform
```
Use single platform for everything

Example: Vercel
- Frontend: Vercel hosting
- Backend: Vercel serverless functions
- Database: Vercel Postgres

Pros:
- Simple setup
- Single platform
- Easy management

Cons:
- Vendor lock-in
- Limited customization
```

### Strategy 2: Best-of-Breed
```
Use best service for each component

Frontend: Vercel (best for Next.js/React)
Backend: Railway/Render (easy deployment)
Database: Supabase (managed PostgreSQL)

Pros:
- Use best tools
- Flexibility

Cons:
- Multiple platforms
- More complex
```

### Strategy 3: Cloud Provider
```
All services from AWS/GCP/Azure

Frontend: S3 + CloudFront
Backend: ECS/Lambda
Database: RDS
Cache: ElastiCache

Pros:
- Everything integrated
- Enterprise features
- Professional support

Cons:
- Complex setup
- Learning curve
- Higher cost
```

---

## Troubleshooting

### Common Issues

#### Issue 1: Frontend Can't Connect to Backend
```
Symptoms:
- CORS errors in browser console
- Network errors
- 404 on API calls

Solutions:
1. Check API URL
   console.log(process.env.REACT_APP_API_URL);

2. Verify CORS settings (backend)
   app.enableCors({
     origin: 'https://myapp.com',
     credentials: true
   });

3. Check network tab
   - Request URL correct?
   - Preflight (OPTIONS) request succeeding?

4. Test API directly
   curl https://api.myapp.com/health
```

#### Issue 2: Database Connection Failed
```
Symptoms:
- "Connection refused"
- "Authentication failed"
- Timeout errors

Solutions:
1. Verify connection string
   console.log(process.env.DATABASE_URL);

2. Check security groups (AWS)
   - Allow inbound on port 5432
   - From backend security group

3. Test connection
   psql $DATABASE_URL

4. Check SSL settings
   ssl: process.env.DATABASE_SSL === 'true'
```

#### Issue 3: Environment Variables Not Working
```
Symptoms:
- undefined values
- Using wrong environment

Solutions:
1. Check if variables are set
   console.log(process.env);

2. Restart application
   - Changes to .env require restart

3. Check build time vs runtime
   React: process.env.REACT_APP_* (build time)
   Node: process.env.* (runtime)

4. Verify on platform
   heroku config
   vercel env ls
```

#### Issue 4: Deployment Succeeds but App Doesn't Work
```
Symptoms:
- 502 Bad Gateway
- Application crashes
- Health check fails

Solutions:
1. Check logs
   heroku logs --tail
   aws logs tail /ecs/myapp

2. Verify environment
   - All required variables set?
   - Correct values?

3. Check health endpoint
   curl https://api.myapp.com/health

4. Verify database migrations
   npm run migration:run
```

#### Issue 5: Slow Application Performance
```
Symptoms:
- High response times
- Timeouts
- Poor user experience

Solutions:
1. Add caching
   - Redis for session/data
   - CDN for static assets

2. Optimize database queries
   - Add indexes
   - Use query builder efficiently
   - Enable connection pooling

3. Enable compression
   app.use(compression());

4. Optimize images
   - Use WebP format
   - Lazy loading
   - CDN

5. Monitor and profile
   - New Relic APM
   - Identify bottlenecks
```

---

## Complete Example Project

### Project Structure
```
myapp/
├── frontend/
│   ├── public/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── services/
│   │   │   └── api.ts
│   │   ├── config/
│   │   │   └── environment.ts
│   │   └── App.tsx
│   ├── package.json
│   ├── Dockerfile
│   └── .env.production
├── backend/
│   ├── src/
│   │   ├── modules/
│   │   │   ├── auth/
│   │   │   ├── users/
│   │   │   └── posts/
│   │   ├── config/
│   │   │   └── configuration.ts
│   │   ├── database/
│   │   │   ├── migrations/
│   │   │   └── seeds/
│   │   └── main.ts
│   ├── package.json
│   ├── Dockerfile
│   └── .env.production
├── .github/
│   └── workflows/
│       └── deploy.yml
├── docker-compose.yml
└── README.md
```

### Deployment Checklist
```
Pre-Deployment:
✅ All tests passing
✅ Code reviewed
✅ Environment variables configured
✅ Database migrations ready
✅ SSL certificates valid
✅ DNS configured
✅ Monitoring set up

Deployment:
✅ Build passes
✅ Database migrated
✅ Frontend deployed
✅ Backend deployed
✅ Health checks passing
✅ HTTPS working
✅ CORS configured

Post-Deployment:
✅ Smoke tests passed
✅ Monitoring active
✅ Logs checked
✅ Performance verified
✅ Team notified
✅ Documentation updated
```

---

## Summary

### Complete Deployment Flow
```
1. Development
   - Write code locally
   - Test with Docker Compose
   - Commit to Git

2. CI/CD
   - Automated testing
   - Build Docker images
   - Security scanning

3. Staging
   - Deploy to staging
   - Run integration tests
   - QA approval

4. Production
   - Blue-green deployment
   - Health checks
   - Monitor metrics

5. Maintenance
   - Monitor logs
   - Performance tuning
   - Regular updates
```

### Key Takeaways
1. **Automate Everything**: CI/CD for reliable deployments
2. **Use Environment Variables**: Never hardcode configuration
3. **Monitor Proactively**: Catch issues before users do
4. **Deploy Small Changes**: Easier to debug and rollback
5. **Test Thoroughly**: Staging should mirror production
6. **Have Rollback Plan**: Always be able to go back
7. **Document Process**: Team should know how to deploy
8. **Keep It Simple**: Start simple, add complexity as needed

### Recommended Stack for Beginners
```
Frontend: Vercel (Next.js/React)
Backend: Railway or Render (Node.js)
Database: Supabase (PostgreSQL)
Monitoring: Sentry (Errors)

Total Cost: $0-20/month
Setup Time: 1-2 hours
Maintenance: Low
```

### Recommended Stack for Production
```
Frontend: Vercel Pro or AWS S3 + CloudFront
Backend: AWS ECS or DigitalOcean App Platform
Database: AWS RDS or DigitalOcean Managed DB
Cache: Redis (managed)
CDN: CloudFront or CloudFlare
Monitoring: Datadog or New Relic

Total Cost: $100-500/month
Setup Time: 1-2 days
Maintenance: Medium
```

You now have everything you need to deploy a full-stack application! 🚀
