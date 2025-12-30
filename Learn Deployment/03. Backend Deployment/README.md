# Backend Deployment

## Table of Contents
- [What is Backend Deployment?](#what-is-backend-deployment)
- [Backend vs Frontend Deployment](#backend-vs-frontend-deployment)
- [Deployment Platforms](#deployment-platforms)
- [Server Setup and Configuration](#server-setup-and-configuration)
- [Process Management](#process-management)
- [Environment Variables and Secrets](#environment-variables-and-secrets)
- [Deployment Strategies](#deployment-strategies)
- [Load Balancing and Scaling](#load-balancing-and-scaling)
- [Common Issues and Solutions](#common-issues-and-solutions)
- [Interview Questions](#interview-questions)

---

## What is Backend Deployment?

Backend deployment involves deploying server-side code that handles business logic, database operations, authentication, and API endpoints. Unlike frontend static files, backend requires a running server process.

### Backend Architecture After Deployment:
```
Client Request → Load Balancer → Backend Server(s) → Database
                                        ↓
                                   External APIs
                                   File Storage
                                   Cache (Redis)
```

### Key Characteristics:
- **Server Process**: Continuously running application
- **Business Logic**: Data processing, validation, authentication
- **Database Access**: CRUD operations, queries
- **API Endpoints**: RESTful, GraphQL, WebSocket
- **Background Jobs**: Scheduled tasks, queue processing

---

## Backend vs Frontend Deployment

| Aspect | Frontend | Backend |
|--------|----------|---------|
| **Files** | Static (HTML, CSS, JS) | Dynamic (Server code) |
| **Execution** | Browser | Server |
| **Process** | No process needed | Requires running process |
| **Updates** | Replace files | Restart process |
| **Scaling** | CDN, caching | Multiple servers, load balancer |
| **Environment** | Build-time variables | Runtime variables |
| **Downtime** | Usually none | Possible during restart |
| **State** | Stateless | Can be stateful |

---

## Deployment Platforms

### 1. **Heroku** (PaaS)

**Best for**: Quick prototypes, simple apps, startups

#### **Deployment Process**:

```bash
# Install Heroku CLI
npm install -g heroku

# Login
heroku login

# Create app
heroku create myapp

# Add PostgreSQL database
heroku addons:create heroku-postgresql:hobby-dev

# Deploy via Git
git push heroku main

# View logs
heroku logs --tail

# Run migrations
heroku run npm run migration:run

# Scale dynos
heroku ps:scale web=2
```

#### **Configuration** (`Procfile`):
```procfile
web: node dist/main.js
worker: node dist/worker.js
release: npm run migration:run
```

#### **Environment Variables**:
```bash
# Set variables
heroku config:set NODE_ENV=production
heroku config:set DATABASE_URL=postgres://...
heroku config:set JWT_SECRET=mysecret

# View variables
heroku config

# Remove variable
heroku config:unset JWT_SECRET
```

#### **Pros**:
- Easy deployment (git push)
- Managed database, Redis
- Auto-scaling
- Built-in monitoring

#### **Cons**:
- Expensive for production
- Free tier sleeps after 30min inactivity
- Limited customization

---

### 2. **Railway** (Modern PaaS)

**Best for**: Modern apps, Docker support, affordable

#### **Deployment Process**:

```bash
# Install Railway CLI
npm install -g @railway/cli

# Login
railway login

# Initialize project
railway init

# Link to existing project
railway link

# Deploy
railway up

# Add database
railway add postgresql

# View logs
railway logs
```

#### **Configuration** (`railway.json` or `railway.toml`):
```json
{
  "build": {
    "builder": "NIXPACKS"
  },
  "deploy": {
    "startCommand": "npm start",
    "healthcheckPath": "/health",
    "restartPolicyType": "ON_FAILURE"
  }
}
```

#### **Environment Variables**:
```bash
# Set via CLI
railway variables set NODE_ENV=production

# Or via dashboard
# Project Settings → Variables
```

#### **Features**:
- Automatic deployments from Git
- Built-in databases (PostgreSQL, MongoDB, Redis)
- Private networking
- Volume storage
- Affordable pricing

---

### 3. **Render** (PaaS)

**Best for**: Full-stack apps, Docker, background workers

#### **Deployment Process**:

**Method 1: Connect Git Repository**
1. Go to Render dashboard
2. New → Web Service
3. Connect GitHub repository
4. Configure settings

**Method 2: Manual Deploy**
```bash
# Create render.yaml
# Deploy via dashboard
```

#### **Configuration** (`render.yaml`):
```yaml
services:
  - type: web
    name: myapp-api
    env: node
    buildCommand: npm install && npm run build
    startCommand: npm start
    envVars:
      - key: NODE_ENV
        value: production
      - key: DATABASE_URL
        fromDatabase:
          name: myapp-db
          property: connectionString
    healthCheckPath: /health
    autoDeploy: true

databases:
  - name: myapp-db
    databaseName: myapp
    user: myapp
    plan: starter
```

#### **Features**:
- Free SSL certificates
- Auto-deploy from Git
- Background workers
- Cron jobs
- PostgreSQL, Redis included
- Private services

---

### 4. **AWS Elastic Beanstalk** (PaaS on AWS)

**Best for**: AWS ecosystem, scalable apps

#### **Deployment Process**:

```bash
# Install EB CLI
pip install awsebcli

# Initialize
eb init

# Create environment
eb create production

# Deploy
eb deploy

# View logs
eb logs

# SSH into instance
eb ssh

# Set environment variables
eb setenv NODE_ENV=production DATABASE_URL=postgres://...

# Scale
eb scale 3
```

#### **Configuration** (`.ebextensions/nodecommand.config`):
```yaml
option_settings:
  aws:elasticbeanstalk:container:nodejs:
    NodeCommand: "npm start"
  aws:elasticbeanstalk:application:environment:
    NODE_ENV: production
```

#### **Application Configuration** (`.elasticbeanstalk/config.yml`):
```yaml
branch-defaults:
  main:
    environment: production
global:
  application_name: myapp
  default_platform: Node.js 18
  default_region: us-east-1
```

---

### 5. **DigitalOcean App Platform** (PaaS)

**Best for**: Simple apps, predictable pricing

#### **Deployment**:

1. Connect GitHub repository
2. Configure via dashboard or `.do/app.yaml`

#### **Configuration** (`.do/app.yaml`):
```yaml
name: myapp
services:
  - name: api
    github:
      repo: username/repo
      branch: main
      deploy_on_push: true
    build_command: npm run build
    run_command: npm start
    envs:
      - key: NODE_ENV
        value: "production"
      - key: DATABASE_URL
        type: SECRET
    health_check:
      http_path: /health
    instance_count: 2
    instance_size_slug: basic-xs

databases:
  - name: db
    engine: PG
    version: "14"
```

---

### 6. **VPS (Virtual Private Server)** - AWS EC2, DigitalOcean Droplets

**Best for**: Full control, custom requirements

#### **Setup Process**:

**1. Create Server**:
```bash
# SSH into server
ssh root@your-server-ip

# Update system
sudo apt update && sudo apt upgrade -y

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install PM2
sudo npm install -g pm2

# Install Nginx
sudo apt install nginx -y
```

**2. Deploy Application**:
```bash
# Clone repository
cd /var/www
git clone https://github.com/username/repo.git myapp
cd myapp

# Install dependencies
npm install

# Build TypeScript
npm run build

# Start with PM2
pm2 start dist/main.js --name myapp
pm2 save
pm2 startup
```

**3. Configure Nginx**:
```nginx
# /etc/nginx/sites-available/myapp
server {
    listen 80;
    server_name api.myapp.com;

    location / {
        proxy_pass http://localhost:4000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

```bash
# Enable site
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx

# Install SSL
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d api.myapp.com
```

**4. Environment Variables**:
```bash
# Create .env file
nano /var/www/myapp/.env

# Or use PM2 ecosystem
# ecosystem.config.js
module.exports = {
  apps: [{
    name: 'myapp',
    script: './dist/main.js',
    env: {
      NODE_ENV: 'production',
      PORT: 4000,
      DATABASE_URL: 'postgres://...'
    }
  }]
};

# Start with ecosystem
pm2 start ecosystem.config.js
```

---

### 7. **Docker + Container Platforms**

#### **Dockerfile** for Node.js:
```dockerfile
# Use official Node.js image
FROM node:18-alpine

# Create app directory
WORKDIR /usr/src/app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application files
COPY . .

# Build TypeScript
RUN npm run build

# Expose port
EXPOSE 4000

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD node healthcheck.js || exit 1

# Start application
CMD ["node", "dist/main.js"]
```

#### **docker-compose.yml**:
```yaml
version: '3.8'

services:
  api:
    build: .
    ports:
      - "4000:4000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://db:5432/myapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
    restart: unless-stopped

  db:
    image: postgres:14
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - /etc/letsencrypt:/etc/letsencrypt
    depends_on:
      - api
    restart: unless-stopped

volumes:
  postgres_data:
```

#### **Deploy to Cloud Run (Google Cloud)**:
```bash
# Build and push image
gcloud builds submit --tag gcr.io/PROJECT_ID/myapp

# Deploy
gcloud run deploy myapp \
  --image gcr.io/PROJECT_ID/myapp \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --set-env-vars NODE_ENV=production,DATABASE_URL=...
```

#### **Deploy to AWS ECS (Elastic Container Service)**:
```bash
# Build image
docker build -t myapp .

# Tag for ECR
docker tag myapp:latest AWS_ACCOUNT.dkr.ecr.us-east-1.amazonaws.com/myapp:latest

# Push to ECR
docker push AWS_ACCOUNT.dkr.ecr.us-east-1.amazonaws.com/myapp:latest

# Create/Update service
aws ecs update-service --cluster mycluster --service myapp --force-new-deployment
```

---

## Server Setup and Configuration

### 1. **Node.js Backend Setup**

#### **Express.js Server**:
```javascript
// src/server.js
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const morgan = require('morgan');

const app = express();
const PORT = process.env.PORT || 4000;

// Middleware
app.use(helmet()); // Security headers
app.use(cors({
  origin: process.env.FRONTEND_URL || 'https://myapp.com',
  credentials: true
}));
app.use(express.json());
app.use(morgan('combined')); // Logging

// Health check endpoint
app.get('/health', (req, res) => {
  res.status(200).json({ 
    status: 'healthy',
    timestamp: new Date().toISOString(),
    uptime: process.uptime()
  });
});

// API routes
app.use('/api/users', require('./routes/users'));
app.use('/api/posts', require('./routes/posts'));

// Error handling
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ 
    error: 'Internal Server Error',
    message: process.env.NODE_ENV === 'development' ? err.message : undefined
  });
});

// Graceful shutdown
process.on('SIGTERM', () => {
  console.log('SIGTERM received, closing server gracefully');
  server.close(() => {
    console.log('Server closed');
    process.exit(0);
  });
});

const server = app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

#### **NestJS Application**:
```typescript
// src/main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { ValidationPipe } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // CORS
  app.enableCors({
    origin: process.env.FRONTEND_URL,
    credentials: true,
  });
  
  // Global validation
  app.useGlobalPipes(new ValidationPipe({
    whitelist: true,
    transform: true,
  }));
  
  // Graceful shutdown
  app.enableShutdownHooks();
  
  const port = process.env.PORT || 3000;
  await app.listen(port);
  console.log(`Application running on: ${await app.getUrl()}`);
}
bootstrap();
```

### 2. **Database Connection**

#### **PostgreSQL with TypeORM**:
```typescript
// src/database/data-source.ts
import { DataSource } from 'typeorm';

export const AppDataSource = new DataSource({
  type: 'postgres',
  url: process.env.DATABASE_URL,
  ssl: process.env.NODE_ENV === 'production' ? { rejectUnauthorized: false } : false,
  entities: ['dist/**/*.entity.js'],
  migrations: ['dist/migrations/*.js'],
  synchronize: false, // NEVER true in production
  logging: process.env.NODE_ENV === 'development',
});
```

#### **MongoDB with Mongoose**:
```javascript
// src/database/connection.js
const mongoose = require('mongoose');

const connectDB = async () => {
  try {
    await mongoose.connect(process.env.MONGODB_URI, {
      useNewUrlParser: true,
      useUnifiedTopology: true,
    });
    console.log('MongoDB connected successfully');
  } catch (error) {
    console.error('MongoDB connection error:', error);
    process.exit(1);
  }
};

module.exports = connectDB;
```

### 3. **Redis Connection**:
```javascript
// src/cache/redis.js
const redis = require('redis');

const client = redis.createClient({
  url: process.env.REDIS_URL || 'redis://localhost:6379',
  socket: {
    reconnectStrategy: (retries) => {
      if (retries > 10) {
        return new Error('Too many retries');
      }
      return retries * 500;
    }
  }
});

client.on('error', (err) => console.error('Redis error:', err));
client.on('connect', () => console.log('Redis connected'));

module.exports = client;
```

---

## Process Management

### 1. **PM2 (Production Process Manager)**

#### **Basic Commands**:
```bash
# Start application
pm2 start app.js --name myapp

# Start with ecosystem file
pm2 start ecosystem.config.js

# List processes
pm2 list

# Monitor
pm2 monit

# Logs
pm2 logs myapp
pm2 logs --lines 100

# Restart
pm2 restart myapp

# Stop
pm2 stop myapp

# Delete
pm2 delete myapp

# Save process list
pm2 save

# Auto-start on system boot
pm2 startup
```

#### **Ecosystem Configuration** (`ecosystem.config.js`):
```javascript
module.exports = {
  apps: [{
    name: 'myapp',
    script: './dist/main.js',
    instances: 'max', // or number
    exec_mode: 'cluster',
    watch: false,
    max_memory_restart: '500M',
    env: {
      NODE_ENV: 'production',
      PORT: 4000
    },
    error_file: './logs/err.log',
    out_file: './logs/out.log',
    log_date_format: 'YYYY-MM-DD HH:mm:ss Z',
    merge_logs: true,
    autorestart: true,
    max_restarts: 10,
    min_uptime: '10s'
  }]
};
```

#### **Cluster Mode** (Multi-core):
```bash
# Start in cluster mode with 4 instances
pm2 start app.js -i 4

# Scale to 8 instances
pm2 scale myapp 8

# Use all CPU cores
pm2 start app.js -i max
```

### 2. **Systemd (Linux Service)**

#### **Create Service File** (`/etc/systemd/system/myapp.service`):
```ini
[Unit]
Description=My Node.js App
After=network.target

[Service]
Type=simple
User=nodejs
WorkingDirectory=/var/www/myapp
ExecStart=/usr/bin/node dist/main.js
Restart=on-failure
RestartSec=10
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=myapp
Environment=NODE_ENV=production
Environment=PORT=4000
EnvironmentFile=/var/www/myapp/.env

[Install]
WantedBy=multi-user.target
```

#### **Commands**:
```bash
# Reload systemd
sudo systemctl daemon-reload

# Start service
sudo systemctl start myapp

# Enable auto-start
sudo systemctl enable myapp

# Check status
sudo systemctl status myapp

# View logs
sudo journalctl -u myapp -f
```

### 3. **Docker Container Management**

```bash
# Run container
docker run -d \
  --name myapp \
  --restart unless-stopped \
  -p 4000:4000 \
  -e NODE_ENV=production \
  -e DATABASE_URL=postgres://... \
  myapp:latest

# View logs
docker logs -f myapp

# Restart container
docker restart myapp

# Execute command in container
docker exec -it myapp sh

# Remove and recreate
docker stop myapp
docker rm myapp
docker run ...
```

---

## Environment Variables and Secrets

### 1. **Environment Files**

```bash
# .env.production
NODE_ENV=production
PORT=4000
DATABASE_URL=postgresql://user:password@host:5432/dbname
REDIS_URL=redis://host:6379
JWT_SECRET=your-secret-key-here
JWT_EXPIRATION=7d
API_KEY=your-api-key
FRONTEND_URL=https://myapp.com
SENTRY_DSN=https://...
```

### 2. **Loading Environment Variables**

```javascript
// Load at application start
require('dotenv').config({ 
  path: `.env.${process.env.NODE_ENV}` 
});

// Validate required variables
const requiredEnvVars = [
  'DATABASE_URL',
  'JWT_SECRET',
  'REDIS_URL'
];

for (const envVar of requiredEnvVars) {
  if (!process.env[envVar]) {
    console.error(`Missing required environment variable: ${envVar}`);
    process.exit(1);
  }
}
```

### 3. **Secrets Management**

#### **AWS Secrets Manager**:
```javascript
const AWS = require('aws-sdk');
const secretsManager = new AWS.SecretsManager({ region: 'us-east-1' });

async function getSecret(secretName) {
  const data = await secretsManager.getSecretValue({ SecretId: secretName }).promise();
  return JSON.parse(data.SecretString);
}

// Usage
const dbCredentials = await getSecret('production/database');
const DATABASE_URL = `postgresql://${dbCredentials.username}:${dbCredentials.password}@...`;
```

#### **HashiCorp Vault**:
```javascript
const vault = require('node-vault')({
  endpoint: 'http://vault:8200',
  token: process.env.VAULT_TOKEN
});

async function getSecrets() {
  const result = await vault.read('secret/data/myapp');
  return result.data.data;
}
```

#### **Environment Variables from Platform**:
```bash
# Heroku
heroku config:set JWT_SECRET=mysecret

# Railway
railway variables set JWT_SECRET=mysecret

# Docker
docker run -e JWT_SECRET=mysecret myapp

# Kubernetes
kubectl create secret generic myapp-secrets \
  --from-literal=jwt-secret=mysecret
```

---

## Deployment Strategies

### 1. **Git Push Deployment**

```bash
# Setup Git remote
git remote add production user@server:/var/www/myapp.git

# Deploy
git push production main

# Post-receive hook (/var/www/myapp.git/hooks/post-receive)
#!/bin/bash
GIT_WORK_TREE=/var/www/myapp git checkout -f
cd /var/www/myapp
npm install
npm run build
pm2 restart myapp
```

### 2. **CI/CD Pipeline with GitHub Actions**

```yaml
# .github/workflows/deploy.yml
name: Deploy Backend

on:
  push:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linter
        run: npm run lint
      
      - name: Run tests
        run: npm test
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/test
  
  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to Heroku
        uses: akhileshns/heroku-deploy@v3.12.14
        with:
          heroku_api_key: ${{ secrets.HEROKU_API_KEY }}
          heroku_app_name: ${{ secrets.HEROKU_APP_NAME }}
          heroku_email: ${{ secrets.HEROKU_EMAIL }}
      
      - name: Run migrations
        run: |
          heroku run npm run migration:run --app=${{ secrets.HEROKU_APP_NAME }}
      
      - name: Health check
        run: |
          sleep 30
          curl --fail https://myapp.herokuapp.com/health || exit 1
```

### 3. **Docker Deployment with CI/CD**

```yaml
# .github/workflows/docker-deploy.yml
name: Build and Deploy Docker

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: username/myapp:latest
      
      - name: Deploy to server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            docker pull username/myapp:latest
            docker stop myapp || true
            docker rm myapp || true
            docker run -d \
              --name myapp \
              --restart unless-stopped \
              -p 4000:4000 \
              --env-file /home/user/.env \
              username/myapp:latest
```

### 4. **Blue-Green Deployment**

```bash
# Deploy to Green environment
pm2 start ecosystem.config.js --name myapp-green --env production

# Test Green
curl http://localhost:4001/health

# Switch Nginx to Green
sudo sed -i 's/localhost:4000/localhost:4001/g' /etc/nginx/sites-available/myapp
sudo nginx -t && sudo systemctl reload nginx

# Stop Blue
pm2 stop myapp-blue
```

### 5. **Rolling Deployment with Multiple Servers**

```bash
# Deploy script for multiple servers
#!/bin/bash
SERVERS=("server1" "server2" "server3")

for server in "${SERVERS[@]}"; do
  echo "Deploying to $server..."
  
  # Deploy to one server
  ssh user@$server "cd /var/www/myapp && git pull && npm install && npm run build && pm2 restart myapp"
  
  # Wait for health check
  sleep 10
  curl --fail http://$server/health || exit 1
  
  # Wait before next server
  sleep 30
done

echo "Rolling deployment complete!"
```

---

## Load Balancing and Scaling

### 1. **Nginx Load Balancer**

```nginx
# /etc/nginx/nginx.conf
upstream backend {
    least_conn; # or ip_hash, or default round-robin
    
    server 10.0.1.10:4000 max_fails=3 fail_timeout=30s;
    server 10.0.1.11:4000 max_fails=3 fail_timeout=30s;
    server 10.0.1.12:4000 max_fails=3 fail_timeout=30s;
    
    keepalive 32;
}

server {
    listen 80;
    server_name api.myapp.com;

    location / {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        
        # Health check
        proxy_next_upstream error timeout http_500 http_502 http_503;
    }
    
    location /health {
        access_log off;
        proxy_pass http://backend;
    }
}
```

### 2. **Horizontal Scaling**

```bash
# PM2 Cluster Mode
pm2 start app.js -i max

# Docker Compose Scale
docker-compose up -d --scale api=4

# Kubernetes Scale
kubectl scale deployment myapp --replicas=5
```

### 3. **Auto-Scaling (AWS)**

```yaml
# AWS Elastic Beanstalk auto-scaling
option_settings:
  aws:autoscaling:asg:
    MinSize: 2
    MaxSize: 10
  aws:autoscaling:trigger:
    MeasureName: CPUUtilization
    Unit: Percent
    UpperThreshold: 75
    LowerThreshold: 25
```

---

## Common Issues and Solutions

### Issue 1: Port Already in Use

**Problem**: `Error: listen EADDRINUSE: address already in use :::4000`

**Solutions**:
```bash
# Find process using port
lsof -i :4000
# Or
netstat -ano | findstr :4000

# Kill process
kill -9 PID

# Use different port
PORT=4001 npm start
```

### Issue 2: Database Connection Fails

**Problem**: Cannot connect to database after deployment

**Solutions**:
- Check DATABASE_URL environment variable
- Verify database is accessible from server
- Check firewall rules
- Enable SSL for production databases
- Verify database credentials

```javascript
// Add connection retry logic
const connectWithRetry = async () => {
  const maxRetries = 5;
  for (let i = 0; i < maxRetries; i++) {
    try {
      await AppDataSource.initialize();
      console.log('Database connected');
      return;
    } catch (error) {
      console.log(`Database connection attempt ${i + 1} failed`);
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 5000));
    }
  }
};
```

### Issue 3: Environment Variables Not Working

**Problem**: `process.env.VARIABLE_NAME` returns `undefined`

**Solutions**:
- Ensure .env file exists in correct location
- Load dotenv before accessing variables: `require('dotenv').config()`
- Check platform environment variable settings
- Restart application after setting variables
- For Docker: pass with `-e` flag or `--env-file`

### Issue 4: Application Crashes After Deployment

**Problem**: App runs locally but crashes in production

**Solutions**:
- Check logs: `pm2 logs` or `heroku logs --tail`
- Verify all dependencies installed
- Check Node version compatibility
- Ensure build step completed successfully
- Look for missing environment variables
- Check for memory limits

```javascript
// Add error handlers
process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Rejection:', reason);
  // Don't exit in production, log to error tracker
});

process.on('uncaughtException', (error) => {
  console.error('Uncaught Exception:', error);
  process.exit(1);
});
```

### Issue 5: CORS Errors After Deployment

**Problem**: Frontend cannot access backend API

**Solutions**:
```javascript
// Configure CORS properly
const cors = require('cors');

app.use(cors({
  origin: [
    'https://myapp.com',
    'https://www.myapp.com',
    process.env.FRONTEND_URL
  ],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));
```

### Issue 6: SSL/HTTPS Issues

**Problem**: Mixed content warnings, SSL errors

**Solutions**:
- Ensure all API calls use HTTPS in production
- Set `trust proxy` in Express for Heroku/AWS
- Use environment variable for API URL

```javascript
// Express behind proxy
app.set('trust proxy', 1);

// Frontend API URL
const API_URL = process.env.NODE_ENV === 'production' 
  ? 'https://api.myapp.com'
  : 'http://localhost:4000';
```

---

## Interview Questions

### Basic Level:

**Q1: What is the difference between deploying frontend and backend?**
**A:** Frontend involves serving static files (HTML, CSS, JS) with no running process required. Backend requires a continuously running server process to handle requests, execute business logic, and interact with databases.

**Q2: What is a PaaS and give examples?**
**A:** PaaS (Platform as a Service) is a cloud platform that manages infrastructure so you can focus on code. Examples: Heroku, Railway, Render, Google App Engine, AWS Elastic Beanstalk. Benefits: Easy deployment, managed databases, auto-scaling.

**Q3: What is PM2 and why use it?**
**A:** PM2 is a production process manager for Node.js applications. Benefits:
- Keeps app running (auto-restart on crash)
- Cluster mode for multi-core utilization
- Log management
- Zero-downtime reloads
- Monitoring

**Q4: How do you set environment variables in production?**
**A:** Methods:
- Platform dashboard (Heroku, Railway)
- CLI: `heroku config:set VAR=value`
- .env file with proper permissions
- Docker: `-e` flag or `--env-file`
- Secrets manager (AWS Secrets Manager, Vault)

**Q5: What is a health check endpoint?**
**A:** An endpoint that returns app status, used by load balancers to determine if instance is healthy:
```javascript
app.get('/health', async (req, res) => {
  try {
    await db.query('SELECT 1');
    res.status(200).json({ status: 'healthy' });
  } catch (error) {
    res.status(503).json({ status: 'unhealthy' });
  }
});
```

### Intermediate Level:

**Q6: Explain the deployment process for a Node.js app on a VPS.**
**A:**
1. Set up server (install Node.js, PM2, Nginx)
2. Clone repository
3. Install dependencies: `npm install`
4. Build if needed: `npm run build`
5. Set environment variables
6. Start with PM2: `pm2 start app.js`
7. Configure Nginx as reverse proxy
8. Set up SSL with Certbot
9. Configure PM2 to start on boot: `pm2 startup`

**Q7: What is a reverse proxy and why use Nginx?**
**A:** Reverse proxy sits between clients and backend servers. Nginx benefits:
- SSL termination (handle HTTPS)
- Load balancing (distribute traffic)
- Caching (improve performance)
- Serve static files efficiently
- Rate limiting
- Security (hide backend structure)

**Q8: How do you handle database migrations in production?**
**A:**
1. Test migrations in staging first
2. Backup database before migration
3. Use migration tools (TypeORM, Sequelize, Flyway)
4. Run migrations before deploying new code
5. Make migrations backward compatible when possible
6. Heroku: `heroku run npm run migration:run`
7. VPS: `npm run migration:run` via SSH or CI/CD

**Q9: What is the difference between scale up and scale out?**
**A:**
- **Scale Up (Vertical)**: Add more resources (CPU, RAM) to existing server. Limited by hardware.
- **Scale Out (Horizontal)**: Add more servers. Requires load balancer. More flexible and fault-tolerant.

**Q10: How do you achieve zero-downtime deployment?**
**A:** Strategies:
- Use load balancer with health checks
- Rolling deployment (update servers one by one)
- Blue-green deployment (switch traffic to new version)
- PM2 reload: `pm2 reload app`
- Database migrations must be backward compatible
- Use feature flags for risky changes

### Advanced Level:

**Q11: Explain a complete CI/CD pipeline for backend deployment.**
**A:**
```yaml
1. Developer pushes code → GitHub
2. CI triggers → GitHub Actions
3. Install dependencies → npm ci
4. Run linter → npm run lint
5. Run unit tests → npm test
6. Run integration tests → npm run test:e2e
7. Build Docker image → docker build
8. Push to registry → Docker Hub/ECR
9. Deploy to staging → Auto-deploy
10. Run smoke tests → Automated
11. Manual approval → Human verification
12. Deploy to production → Blue-green swap
13. Run database migrations → Automated
14. Health checks → Monitor endpoints
15. Rollback if needed → Automated
```

**Q12: How would you design a scalable backend architecture?**
**A:**
```
                    ┌─────────────┐
                    │   Route53   │ (DNS)
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │     ALB     │ (Load Balancer)
                    └──────┬──────┘
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
    ┌────▼────┐       ┌────▼────┐      ┌────▼────┐
    │  EC2/   │       │  EC2/   │      │  EC2/   │
    │ Server1 │       │ Server2 │      │ Server3 │
    └────┬────┘       └────┬────┘      └────┬────┘
         │                 │                 │
         └─────────────────┼─────────────────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
         ┌────▼────┐  ┌────▼────┐  ┌───▼────┐
         │   RDS   │  │  Redis  │  │   S3   │
         │(Primary)│  │ (Cache) │  │ (Files)│
         └────┬────┘  └─────────┘  └────────┘
              │
         ┌────▼────┐
         │   RDS   │
         │ (Read   │
         │ Replica)│
         └─────────┘
```

Key components:
- Load balancer (distribute traffic)
- Multiple app servers (horizontal scaling)
- Database read replicas (read scaling)
- Redis cache (reduce DB load)
- S3 for file storage (offload from app servers)
- Auto-scaling based on metrics

**Q13: How do you handle secrets in production?**
**A:** Best practices:
1. Never commit secrets to Git
2. Use environment variables at minimum
3. Better: Use secrets management service
   - AWS Secrets Manager
   - HashiCorp Vault
   - Azure Key Vault
4. Rotate secrets regularly
5. Use IAM roles instead of access keys (AWS)
6. Encrypt secrets at rest
7. Limit access (principle of least privilege)
8. Audit secret access

**Q14: Explain graceful shutdown and why it's important.**
**A:** Graceful shutdown allows app to finish processing requests before shutting down:
```javascript
let server;

process.on('SIGTERM', async () => {
  console.log('SIGTERM received, shutting down gracefully');
  
  // Stop accepting new connections
  server.close(() => {
    console.log('HTTP server closed');
  });
  
  // Close database connections
  await AppDataSource.destroy();
  
  // Give ongoing requests time to complete
  setTimeout(() => {
    console.log('Forcing shutdown');
    process.exit(0);
  }, 10000);
});

server = app.listen(PORT);
```

Important because:
- Prevents data loss
- Completes in-flight requests
- Closes DB connections properly
- Required for zero-downtime deployments

**Q15: How do you monitor backend application in production?**
**A:** Multi-layer monitoring:
1. **Application Monitoring**:
   - Error tracking (Sentry, Rollbar)
   - APM (New Relic, DataDog)
   - Custom metrics (response times, request counts)

2. **Infrastructure Monitoring**:
   - CPU, memory, disk usage
   - Network I/O
   - CloudWatch (AWS), Stackdriver (GCP)

3. **Logging**:
   - Centralized logging (ELK, Splunk, CloudWatch Logs)
   - Structured logging (JSON format)
   - Log levels (error, warn, info, debug)

4. **Uptime Monitoring**:
   - External monitors (Pingdom, UptimeRobot)
   - Alert on downtime

5. **Database Monitoring**:
   - Query performance
   - Connection pool usage
   - Slow query logs

6. **Alerting**:
   - PagerDuty, Opsgenie
   - Alert on thresholds
   - On-call rotations

**Q16: What is connection pooling and why is it important?**
**A:** Connection pooling reuses database connections instead of creating new ones for each request:
```javascript
// Without pooling (BAD)
// Creates new connection for every request
app.get('/users', async (req, res) => {
  const connection = await createConnection();
  const users = await connection.query('SELECT * FROM users');
  await connection.close();
  res.json(users);
});

// With pooling (GOOD)
const pool = new Pool({ max: 20, min: 5 });
app.get('/users', async (req, res) => {
  const users = await pool.query('SELECT * FROM users');
  res.json(users);
});
```

Benefits:
- Reduces connection overhead
- Improves performance
- Limits max connections to database
- Reuses existing connections

**Q17: How do you handle file uploads in production?**
**A:** Best practices:
1. Don't store files on application server (stateless)
2. Use cloud storage (S3, Google Cloud Storage)
3. Generate signed URLs for direct upload
4. Validate file types and sizes
5. Scan for malware
6. Use CDN for serving files
7. Implement rate limiting

```javascript
// Direct upload to S3
const s3 = new AWS.S3();

app.post('/upload-url', async (req, res) => {
  const { filename, filetype } = req.body;
  
  const s3Params = {
    Bucket: process.env.S3_BUCKET,
    Key: `uploads/${Date.now()}-${filename}`,
    Expires: 300,
    ContentType: filetype,
  };
  
  const uploadURL = await s3.getSignedUrlPromise('putObject', s3Params);
  res.json({ uploadURL });
});
```

**Q18: Explain rate limiting and how to implement it.**
**A:** Rate limiting restricts number of requests from a client:
```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP',
  standardHeaders: true,
  legacyHeaders: false,
});

app.use('/api/', limiter);

// Different limits for different endpoints
const strictLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 5 // 5 requests per hour
});

app.use('/api/auth/login', strictLimiter);
```

Benefits:
- Prevent abuse
- Protect against DDoS
- Ensure fair usage
- Reduce costs

**Q19: How do you handle background jobs in production?**
**A:** Use job queue systems:
```javascript
// Bull (Redis-based queue)
const Queue = require('bull');
const emailQueue = new Queue('email', process.env.REDIS_URL);

// Producer (add jobs)
app.post('/send-email', async (req, res) => {
  await emailQueue.add({
    to: req.body.email,
    subject: 'Welcome',
    body: 'Thanks for signing up'
  }, {
    attempts: 3,
    backoff: { type: 'exponential', delay: 5000 }
  });
  res.json({ message: 'Email queued' });
});

// Consumer (process jobs)
emailQueue.process(async (job) => {
  await sendEmail(job.data);
});
```

Use cases:
- Email sending
- Image processing
- Report generation
- Data imports
- Scheduled tasks

**Q20: What is the difference between stateless and stateful backends?**
**A:**
**Stateless**:
- No session data stored on server
- Each request is independent
- Easy to scale horizontally
- Load balancer can send requests to any server
- Session data in external store (Redis, JWT)
- Example: RESTful API

**Stateful**:
- Session data stored on server
- Requires sticky sessions (load balancer sends user to same server)
- Harder to scale
- More complex deployment
- Example: WebSocket server with in-memory session

Best practice: Design stateless backends, use Redis for session storage.

---

## Summary

Backend deployment is complex and requires careful planning. Unlike frontend static files, backend deployment involves running server processes, managing databases, handling secrets, and ensuring high availability. Understanding different deployment platforms, process management, monitoring, and scaling strategies is crucial for building reliable production systems.

**Key Takeaways**:
- Choose platform based on needs (PaaS for simplicity, VPS for control)
- Use process managers (PM2, systemd) for reliability
- Implement health checks for load balancers
- Use secrets management, never commit secrets
- Configure proper logging and monitoring
- Plan for horizontal scaling
- Implement CI/CD for reliable deployments
- Always have rollback strategy
- Test in staging before production
