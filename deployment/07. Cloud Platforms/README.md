# Cloud Platforms for Deployment

## Table of Contents
1. [Cloud Computing Basics](#cloud-computing-basics)
2. [AWS (Amazon Web Services)](#aws-amazon-web-services)
3. [Google Cloud Platform (GCP)](#google-cloud-platform-gcp)
4. [Microsoft Azure](#microsoft-azure)
5. [Vercel](#vercel)
6. [Netlify](#netlify)
7. [Heroku](#heroku)
8. [DigitalOcean](#digitalocean)
9. [Cloud Comparison](#cloud-comparison)
10. [Full Stack Deployment on Cloud](#full-stack-deployment-on-cloud)

---

## Cloud Computing Basics

### What is Cloud Computing?
On-demand delivery of IT resources over the internet with pay-as-you-go pricing.

### Service Models
```
┌─────────────────────────────────────────┐
│  IaaS (Infrastructure as a Service)     │
│  - Virtual machines, storage, networks  │
│  - You manage: OS, runtime, application │
│  - Examples: AWS EC2, Azure VMs         │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│  PaaS (Platform as a Service)           │
│  - Managed runtime environment          │
│  - You manage: Application only         │
│  - Examples: Heroku, Google App Engine  │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│  SaaS (Software as a Service)           │
│  - Fully managed application            │
│  - You manage: Data only                │
│  - Examples: Gmail, Office 365          │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│  FaaS (Function as a Service)           │
│  - Serverless computing                 │
│  - Pay per execution                    │
│  - Examples: AWS Lambda, Azure Functions│
└─────────────────────────────────────────┘
```

### Cloud Deployment Models
```
Public Cloud:
- AWS, Azure, GCP
- Shared infrastructure
- Pay as you go
- High scalability

Private Cloud:
- Your own data center
- Dedicated infrastructure
- More control
- Higher cost

Hybrid Cloud:
- Mix of public and private
- Data on-premises, apps in cloud
- Flexibility
- Complex management
```

### Benefits of Cloud
```
1. Scalability
   - Scale up/down automatically
   - Handle traffic spikes

2. Cost-Effective
   - Pay for what you use
   - No upfront hardware cost

3. Global Reach
   - Deploy to multiple regions
   - Low latency worldwide

4. Reliability
   - High availability (99.9%+)
   - Automatic backups
   - Disaster recovery

5. Security
   - Professional security teams
   - Compliance certifications
   - Regular updates
```

---

## AWS (Amazon Web Services)

### Overview
Largest cloud provider with 200+ services covering compute, storage, databases, ML, and more.

### Core Services for Full Stack Applications

#### 1. EC2 (Elastic Compute Cloud)
```
Virtual servers in the cloud

Use Cases:
- Backend servers
- Custom applications
- Full control over OS

Example:
┌────────────────────────────┐
│  EC2 Instance              │
│  - t3.micro (1GB RAM)      │
│  - Ubuntu 22.04            │
│  - Node.js application     │
│  - Public IP: x.x.x.x      │
└────────────────────────────┘
```

**Deploying Node.js to EC2:**
```bash
# 1. Launch EC2 instance
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t3.micro \
  --key-name my-key \
  --security-group-ids sg-xxx

# 2. SSH into instance
ssh -i my-key.pem ubuntu@<instance-ip>

# 3. Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# 4. Clone and run application
git clone https://github.com/user/myapp.git
cd myapp
npm install
npm start

# 5. Use PM2 for process management
sudo npm install -g pm2
pm2 start npm --name "myapp" -- start
pm2 startup
pm2 save
```

#### 2. S3 (Simple Storage Service)
```
Object storage service

Use Cases:
- Frontend hosting (static files)
- User uploads (images, videos)
- Backups
- Data lakes

Features:
- 99.999999999% durability
- Unlimited storage
- $0.023 per GB/month
```

**Hosting React App on S3:**
```bash
# 1. Build React app
npm run build

# 2. Create S3 bucket
aws s3 mb s3://my-react-app

# 3. Enable static website hosting
aws s3 website s3://my-react-app \
  --index-document index.html \
  --error-document index.html

# 4. Upload build files
aws s3 sync build/ s3://my-react-app --acl public-read

# 5. Access at:
# http://my-react-app.s3-website-us-east-1.amazonaws.com
```

#### 3. RDS (Relational Database Service)
```
Managed database service

Supported:
- PostgreSQL
- MySQL
- MariaDB
- Oracle
- SQL Server

Benefits:
- Automated backups
- Automated patching
- Multi-AZ for high availability
- Read replicas for scaling
```

**Creating PostgreSQL Database:**
```bash
# Create database instance
aws rds create-db-instance \
  --db-instance-identifier myapp-db \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --master-username admin \
  --master-user-password MyPassword123 \
  --allocated-storage 20

# Get connection endpoint
aws rds describe-db-instances \
  --db-instance-identifier myapp-db \
  --query 'DBInstances[0].Endpoint.Address'

# Connect from application
DATABASE_URL=postgresql://admin:MyPassword123@myapp-db.xxx.rds.amazonaws.com:5432/mydb
```

#### 4. Lambda (Serverless Functions)
```
Run code without managing servers

Use Cases:
- API endpoints
- Background jobs
- Event processing
- Scheduled tasks

Pricing:
- Free: 1M requests/month
- $0.20 per 1M requests after
- Pay for execution time
```

**Lambda Function Example:**
```javascript
// handler.js
exports.handler = async (event) => {
  const body = JSON.parse(event.body);
  
  // Process request
  const result = await processData(body);
  
  return {
    statusCode: 200,
    headers: {
      'Access-Control-Allow-Origin': '*',
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(result)
  };
};

async function processData(data) {
  // Business logic
  return { success: true, data };
}
```

**Deploy with Serverless Framework:**
```yaml
# serverless.yml
service: myapp-api

provider:
  name: aws
  runtime: nodejs18.x
  region: us-east-1
  environment:
    DATABASE_URL: ${env:DATABASE_URL}

functions:
  getUsers:
    handler: handler.getUsers
    events:
      - http:
          path: users
          method: get
          cors: true
  
  createUser:
    handler: handler.createUser
    events:
      - http:
          path: users
          method: post
          cors: true

resources:
  Resources:
    UsersTable:
      Type: AWS::DynamoDB::Table
      Properties:
        TableName: users
        AttributeDefinitions:
          - AttributeName: id
            AttributeType: S
        KeySchema:
          - AttributeName: id
            KeyType: HASH
        BillingMode: PAY_PER_REQUEST
```

```bash
# Deploy
serverless deploy

# Outputs:
# endpoints:
#   GET - https://xxx.execute-api.us-east-1.amazonaws.com/dev/users
#   POST - https://xxx.execute-api.us-east-1.amazonaws.com/dev/users
```

#### 5. CloudFront (CDN)
```
Content Delivery Network

Use Cases:
- Speed up website
- Global distribution
- HTTPS/SSL
- DDoS protection

How it works:
User (India) → Edge Location (Mumbai) → Origin (S3 in US)
First request: Slow (fetch from origin)
Subsequent: Fast (cached at edge)
```

**Setup CloudFront:**
```bash
# Create distribution
aws cloudfront create-distribution \
  --origin-domain-name my-react-app.s3.amazonaws.com \
  --default-root-object index.html

# Get distribution URL
https://d111111abcdef8.cloudfront.net

# Add custom domain (optional)
my-app.com → CloudFront → S3
```

#### 6. Elastic Beanstalk
```
PaaS for deploying applications

Supports:
- Node.js
- Python
- Java
- .NET
- PHP
- Ruby
- Go
- Docker

Benefits:
- Auto-scaling
- Load balancing
- Monitoring
- Easy deployment
```

**Deploy Node.js App:**
```bash
# Install EB CLI
pip install awsebcli

# Initialize
eb init -p node.js-18 my-app

# Create environment and deploy
eb create production

# Deploy updates
eb deploy

# Open in browser
eb open
```

#### 7. Route 53 (DNS)
```
Domain name system service

Features:
- Domain registration
- DNS routing
- Health checks
- Traffic management
```

**Setup Domain:**
```bash
# 1. Register domain (or use existing)
aws route53domains register-domain --domain-name myapp.com

# 2. Create hosted zone
aws route53 create-hosted-zone --name myapp.com

# 3. Create A record
{
  "Changes": [{
    "Action": "CREATE",
    "ResourceRecordSet": {
      "Name": "myapp.com",
      "Type": "A",
      "AliasTarget": {
        "HostedZoneId": "Z2FDTNDATAQYW2",
        "DNSName": "d111111abcdef8.cloudfront.net",
        "EvaluateTargetHealth": false
      }
    }
  }]
}
```

### AWS Full Stack Architecture
```
┌──────────────────────────────────────────────────────┐
│                    Users                             │
└────────────────┬─────────────────────────────────────┘
                 │
         ┌───────▼────────┐
         │  Route 53      │  DNS
         │  myapp.com     │
         └───────┬────────┘
                 │
         ┌───────▼────────┐
         │  CloudFront    │  CDN
         │  (Edge Cache)  │
         └───────┬────────┘
                 │
        ┌────────┴─────────┐
        │                  │
   ┌────▼─────┐    ┌──────▼─────┐
   │    S3    │    │  API GW /  │
   │ (React)  │    │   Lambda   │
   └──────────┘    └──────┬─────┘
                          │
                   ┌──────▼─────┐
                   │    RDS     │
                   │ (Postgres) │
                   └────────────┘
```

---

## Google Cloud Platform (GCP)

### Overview
Google's cloud platform with strong data analytics and ML capabilities.

### Core Services

#### 1. Compute Engine
```
Virtual machines (similar to AWS EC2)

Machine Types:
- e2-micro: 0.25-2 vCPU, 1GB RAM (Free tier)
- e2-standard-2: 2 vCPU, 8GB RAM
- Custom: Configure your own
```

#### 2. Cloud Storage
```
Object storage (similar to S3)

Storage Classes:
- Standard: Frequent access
- Nearline: Once per month
- Coldline: Once per quarter
- Archive: Once per year

Pricing: $0.020 per GB/month (standard)
```

#### 3. Cloud SQL
```
Managed databases
- PostgreSQL
- MySQL
- SQL Server

Features:
- Automated backups
- High availability
- Read replicas
```

#### 4. Cloud Run
```
Serverless containers

Benefits:
- Deploy containers without managing infrastructure
- Auto-scaling (0 to 1000+)
- Pay per request
- Any language/framework
```

**Deploy to Cloud Run:**
```dockerfile
# Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 8080
CMD ["node", "server.js"]
```

```bash
# Build and push
gcloud builds submit --tag gcr.io/PROJECT_ID/myapp

# Deploy
gcloud run deploy myapp \
  --image gcr.io/PROJECT_ID/myapp \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated

# Get URL
https://myapp-xxx-uc.a.run.app
```

#### 5. Firebase (Frontend Focus)
```
Backend-as-a-Service (BaaS)

Services:
- Hosting: Static sites
- Firestore: NoSQL database
- Authentication: User management
- Cloud Functions: Serverless
- Storage: File uploads

Perfect for: React/Angular/Vue apps
```

**Deploy React to Firebase:**
```bash
# Install CLI
npm install -g firebase-tools

# Login
firebase login

# Initialize
firebase init

# Select:
# - Hosting
# - Firestore (optional)
# - Functions (optional)

# Configure
# Public directory: build
# Single-page app: Yes

# Build
npm run build

# Deploy
firebase deploy

# URL: https://myapp.web.app
```

**Firebase Configuration:**
```javascript
// src/firebase.ts
import { initializeApp } from 'firebase/app';
import { getFirestore } from 'firebase/firestore';
import { getAuth } from 'firebase/auth';

const firebaseConfig = {
  apiKey: process.env.REACT_APP_FIREBASE_API_KEY,
  authDomain: "myapp.firebaseapp.com",
  projectId: "myapp",
  storageBucket: "myapp.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abcdef"
};

const app = initializeApp(firebaseConfig);
export const db = getFirestore(app);
export const auth = getAuth(app);
```

### GCP Full Stack Architecture
```
┌──────────────────────────────────────────────────────┐
│                    Users                             │
└────────────────┬─────────────────────────────────────┘
                 │
         ┌───────▼────────┐
         │  Cloud CDN     │
         └───────┬────────┘
                 │
        ┌────────┴─────────┐
        │                  │
   ┌────▼─────┐    ┌──────▼─────┐
   │ Firebase │    │ Cloud Run  │
   │ Hosting  │    │  (Backend) │
   │ (React)  │    └──────┬─────┘
   └──────────┘           │
                   ┌──────▼─────┐
                   │ Cloud SQL  │
                   │ (Postgres) │
                   └────────────┘
```

---

## Microsoft Azure

### Overview
Microsoft's cloud platform with strong enterprise integration.

### Core Services

#### 1. Virtual Machines
```
Similar to AWS EC2, GCP Compute Engine

VM Sizes:
- B1s: 1 vCPU, 1GB RAM (Basic)
- D2s v3: 2 vCPU, 8GB RAM (General)
- F2s v2: 2 vCPU, 4GB RAM (Compute)
```

#### 2. Blob Storage
```
Object storage (like S3)

Tiers:
- Hot: Frequent access
- Cool: Infrequent access (30+ days)
- Archive: Rare access (180+ days)
```

#### 3. Azure SQL Database
```
Managed SQL Server

Features:
- Auto-tuning
- Threat detection
- Automatic backups
```

#### 4. App Service
```
PaaS for web apps

Supports:
- Node.js
- .NET
- Python
- Java
- PHP

Features:
- Auto-scaling
- Deployment slots
- Custom domains
- SSL certificates
```

**Deploy to App Service:**
```bash
# Install Azure CLI
# https://docs.microsoft.com/cli/azure/install-azure-cli

# Login
az login

# Create resource group
az group create --name myapp-rg --location eastus

# Create App Service plan
az appservice plan create \
  --name myapp-plan \
  --resource-group myapp-rg \
  --sku B1 \
  --is-linux

# Create web app
az webapp create \
  --resource-group myapp-rg \
  --plan myapp-plan \
  --name myapp-backend \
  --runtime "NODE:18-lts"

# Deploy from GitHub
az webapp deployment source config \
  --name myapp-backend \
  --resource-group myapp-rg \
  --repo-url https://github.com/user/myapp \
  --branch main \
  --manual-integration

# URL: https://myapp-backend.azurewebsites.net
```

#### 5. Azure Functions
```
Serverless compute (like AWS Lambda)

Triggers:
- HTTP
- Timer
- Queue
- Blob storage
- Event Hub
```

#### 6. Azure CDN
```
Content delivery network

Integration:
- Storage
- Web Apps
- Media Services
```

---

## Vercel

### Overview
Platform optimized for frontend frameworks (Next.js creators).

### Features
```
✅ Zero-config deployment
✅ Automatic HTTPS
✅ Global CDN
✅ Serverless functions
✅ Preview deployments
✅ Built-in CI/CD
```

### Supported Frameworks
```
- Next.js (best support)
- React
- Vue
- Angular
- Svelte
- Gatsby
- Nuxt
```

### Deployment
```bash
# Install Vercel CLI
npm install -g vercel

# Deploy
vercel

# First deployment:
# - Link to existing project or create new
# - Configure settings
# - Deploy

# Production deployment
vercel --prod

# Environment variables
vercel env add DATABASE_URL
vercel env add production
```

**vercel.json Configuration:**
```json
{
  "version": 2,
  "builds": [
    {
      "src": "package.json",
      "use": "@vercel/node"
    }
  ],
  "routes": [
    {
      "src": "/api/(.*)",
      "dest": "/api/$1"
    },
    {
      "src": "/(.*)",
      "dest": "/$1"
    }
  ],
  "env": {
    "DATABASE_URL": "@database-url"
  }
}
```

### Vercel Serverless Functions
```
pages/api/users.ts
```

```typescript
// pages/api/users.ts
import type { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method === 'GET') {
    const users = await fetchUsers();
    res.status(200).json(users);
  } else if (req.method === 'POST') {
    const user = await createUser(req.body);
    res.status(201).json(user);
  } else {
    res.status(405).json({ error: 'Method not allowed' });
  }
}
```

### Features

#### 1. Preview Deployments
```
Every git push creates a unique preview URL
- test-branch-abc123.vercel.app
- Share with team
- Test before production
```

#### 2. Automatic HTTPS
```
Every deployment gets SSL certificate
- Production: myapp.com
- Previews: preview-xxx.vercel.app
```

#### 3. Edge Functions
```typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  // Runs at the edge (close to user)
  const country = request.geo?.country || 'US';
  
  // Redirect based on location
  if (country === 'IN') {
    return NextResponse.redirect('https://myapp.in');
  }
  
  return NextResponse.next();
}
```

---

## Netlify

### Overview
Platform for modern web projects with focus on JAMstack.

### Features
```
✅ Git-based deployments
✅ Automatic builds
✅ Instant rollbacks
✅ Split testing (A/B)
✅ Forms handling
✅ Identity (authentication)
✅ Functions (serverless)
```

### Deployment
```bash
# Install Netlify CLI
npm install -g netlify-cli

# Login
netlify login

# Initialize
netlify init

# Deploy
netlify deploy

# Production deploy
netlify deploy --prod
```

**netlify.toml Configuration:**
```toml
[build]
  command = "npm run build"
  publish = "build"
  functions = "netlify/functions"

[build.environment]
  NODE_VERSION = "18"

[[redirects]]
  from = "/api/*"
  to = "/.netlify/functions/:splat"
  status = 200

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options = "DENY"
    X-XSS-Protection = "1; mode=block"
```

### Netlify Functions
```javascript
// netlify/functions/hello.js
exports.handler = async (event, context) => {
  return {
    statusCode: 200,
    body: JSON.stringify({
      message: 'Hello from Netlify Function'
    })
  };
};

// Access at: https://myapp.netlify.app/.netlify/functions/hello
```

### Features

#### 1. Forms
```html
<!-- Simple form handling -->
<form name="contact" method="POST" data-netlify="true">
  <input type="text" name="name" />
  <input type="email" name="email" />
  <textarea name="message"></textarea>
  <button type="submit">Send</button>
</form>

<!-- Netlify automatically:
  - Creates form endpoint
  - Stores submissions
  - Sends notifications
-->
```

#### 2. Split Testing
```toml
# Deploy two versions, split traffic
[context.branch-a]
  command = "npm run build:variant-a"

[context.branch-b]
  command = "npm run build:variant-b"

# Configure in UI:
# branch-a: 50%
# branch-b: 50%
```

---

## Heroku

### Overview
Cloud platform (PaaS) with simple deployment experience.

### Features
```
✅ Git-based deployment
✅ Automatic scaling
✅ Add-ons (database, cache, etc.)
✅ Free tier (with limitations)
✅ Support multiple languages
```

### Deployment
```bash
# Install Heroku CLI
# https://devcenter.heroku.com/articles/heroku-cli

# Login
heroku login

# Create app
heroku create myapp-backend

# Add database
heroku addons:create heroku-postgresql:mini

# Deploy
git push heroku main

# View logs
heroku logs --tail

# Open app
heroku open
```

**Procfile (Required):**
```
web: node server.js
worker: node worker.js
release: npm run migrate
```

**app.json (Optional):**
```json
{
  "name": "My App",
  "description": "Full stack application",
  "image": "heroku/nodejs",
  "repository": "https://github.com/user/myapp",
  "keywords": ["node", "express", "postgresql"],
  "addons": [
    "heroku-postgresql:mini",
    "heroku-redis:mini"
  ],
  "env": {
    "NODE_ENV": {
      "description": "Node environment",
      "value": "production"
    },
    "JWT_SECRET": {
      "description": "Secret for JWT tokens",
      "generator": "secret"
    }
  },
  "scripts": {
    "postdeploy": "npm run migrate"
  }
}
```

### Add-ons
```bash
# PostgreSQL
heroku addons:create heroku-postgresql:mini

# Redis
heroku addons:create heroku-redis:mini

# Papertrail (logging)
heroku addons:create papertrail:choklad

# SendGrid (email)
heroku addons:create sendgrid:starter

# New Relic (monitoring)
heroku addons:create newrelic:wayne
```

---

## DigitalOcean

### Overview
Developer-friendly cloud with simple pricing and good documentation.

### Services

#### 1. Droplets
```
Virtual machines

Pricing:
- $6/month: 1 vCPU, 1GB RAM, 25GB SSD
- $12/month: 1 vCPU, 2GB RAM, 50GB SSD
- $18/month: 2 vCPU, 2GB RAM, 60GB SSD
```

#### 2. App Platform
```
PaaS (like Heroku)

Features:
- Auto-deploy from Git
- Auto-scaling
- Managed databases
- CDN included

Pricing: $5/month (basic)
```

**Deploy to App Platform:**
```yaml
# .do/app.yaml
name: myapp
services:
- name: web
  github:
    repo: user/myapp
    branch: main
    deploy_on_push: true
  build_command: npm run build
  run_command: npm start
  environment_slug: node-js
  instance_count: 1
  instance_size_slug: basic-xxs
  routes:
  - path: /
  envs:
  - key: NODE_ENV
    value: production
  - key: DATABASE_URL
    value: ${db.DATABASE_URL}

databases:
- name: db
  engine: PG
  version: "14"
```

#### 3. Managed Databases
```
Supported:
- PostgreSQL
- MySQL
- Redis
- MongoDB

Pricing: $15/month (1GB RAM, 10GB storage)
```

#### 4. Spaces (Object Storage)
```
S3-compatible object storage

Pricing: $5/month (250GB, 1TB transfer)
```

---

## Cloud Comparison

### Pricing Comparison (Approximate)
```
┌────────────────┬─────────────┬──────────────┬──────────────┐
│ Service        │ AWS         │ GCP          │ Azure        │
├────────────────┼─────────────┼──────────────┼──────────────┤
│ VM (2 vCPU,    │ $17/month   │ $14/month    │ $20/month    │
│  8GB RAM)      │ (t3.large)  │ (e2-standard)│ (B2s)        │
├────────────────┼─────────────┼──────────────┼──────────────┤
│ Object Storage │ $23/TB/mo   │ $20/TB/mo    │ $21/TB/mo    │
├────────────────┼─────────────┼──────────────┼──────────────┤
│ Database       │ $15/month   │ $14/month    │ $20/month    │
│ (PostgreSQL)   │ (db.t3.     │ (db-f1-micro)│ (Basic)      │
│                │ micro)      │              │              │
├────────────────┼─────────────┼──────────────┼──────────────┤
│ Serverless     │ Lambda      │ Cloud Run    │ Functions    │
│                │ $0.20/1M    │ $0.40/1M     │ $0.20/1M     │
└────────────────┴─────────────┴──────────────┴──────────────┘
```

### Platform Comparison
```
┌──────────────┬──────────────┬──────────────┬──────────────┐
│ Platform     │ Best For     │ Pricing      │ Difficulty   │
├──────────────┼──────────────┼──────────────┼──────────────┤
│ Vercel       │ Next.js,     │ Free tier    │ Easy         │
│              │ Frontend     │ generous     │              │
├──────────────┼──────────────┼──────────────┼──────────────┤
│ Netlify      │ JAMstack,    │ Free tier    │ Easy         │
│              │ Static sites │ good         │              │
├──────────────┼──────────────┼──────────────┼──────────────┤
│ Heroku       │ Prototypes,  │ $7-$25/mo    │ Easy         │
│              │ MVPs         │ per dyno     │              │
├──────────────┼──────────────┼──────────────┼──────────────┤
│ DigitalOcean │ Full control │ $6-18/mo     │ Medium       │
│              │ Simple       │ per droplet  │              │
├──────────────┼──────────────┼──────────────┼──────────────┤
│ AWS          │ Enterprise,  │ Pay as go    │ Hard         │
│              │ All services │ Complex      │              │
├──────────────┼──────────────┼──────────────┼──────────────┤
│ GCP          │ Data/ML,     │ Pay as go    │ Hard         │
│              │ Kubernetes   │              │              │
├──────────────┼──────────────┼──────────────┼──────────────┤
│ Azure        │ Enterprise,  │ Pay as go    │ Hard         │
│              │ Microsoft    │              │              │
└──────────────┴──────────────┴──────────────┴──────────────┘
```

### Decision Guide
```
Choose Vercel/Netlify if:
✅ Frontend-focused (React, Next.js)
✅ Need fast deployment
✅ Want serverless functions
✅ Small to medium projects

Choose Heroku if:
✅ Need quick MVP/prototype
✅ Simple full-stack app
✅ Don't want DevOps complexity
✅ Okay with higher cost

Choose DigitalOcean if:
✅ Need full control
✅ Want simple pricing
✅ Medium-sized projects
✅ Comfortable with some DevOps

Choose AWS/GCP/Azure if:
✅ Enterprise-grade requirements
✅ Need specific services
✅ High scalability needed
✅ Have DevOps team/expertise
```

---

## Full Stack Deployment on Cloud

### Example: React + NestJS + PostgreSQL

#### Architecture
```
Internet
   │
   ▼
┌──────────────┐
│  Cloudflare  │  (CDN + SSL)
└──────┬───────┘
       │
   ┌───┴────┐
   │        │
   ▼        ▼
┌─────┐  ┌─────────┐
│ S3/ │  │  EC2/   │
│Vercel│  │ Heroku  │
│React│  │ NestJS  │
└─────┘  └────┬────┘
              │
          ┌───▼────┐
          │   RDS/ │
          │Postgres│
          └────────┘
```

### Option 1: All AWS
```bash
# Frontend: S3 + CloudFront
aws s3 sync build/ s3://myapp-frontend
aws cloudfront create-invalidation --distribution-id XXX --paths "/*"

# Backend: Elastic Beanstalk
eb init && eb create production

# Database: RDS
aws rds create-db-instance --db-instance-identifier myapp-db

# Connect them:
# Frontend: https://d111111abcdef8.cloudfront.net
# Backend: http://myapp-api.elasticbeanstalk.com
# Database: myapp-db.xxx.rds.amazonaws.com:5432
```

### Option 2: Hybrid (Vercel + Heroku)
```bash
# Frontend: Vercel
cd frontend
vercel --prod

# Backend: Heroku
cd backend
git push heroku main

# Database: Heroku PostgreSQL
heroku addons:create heroku-postgresql:mini

# Environment variables:
# Frontend (.env):
REACT_APP_API_URL=https://myapp-api.herokuapp.com

# Backend (Heroku config):
heroku config:set JWT_SECRET=xxx
heroku config:set DATABASE_URL=postgres://...
```

### Option 3: All Vercel (Next.js)
```typescript
// Next.js with API routes + Vercel Postgres

// pages/api/users.ts
import { sql } from '@vercel/postgres';

export default async function handler(req, res) {
  if (req.method === 'GET') {
    const { rows } = await sql`SELECT * FROM users`;
    res.json(rows);
  }
}

// Deploy
vercel --prod

// Add database
vercel postgres create

// Everything on Vercel:
// - Frontend: React
// - Backend: API routes
// - Database: Vercel Postgres
```

### Option 4: DigitalOcean App Platform
```yaml
# .do/app.yaml
name: myapp
services:
  # Frontend
  - name: frontend
    github:
      repo: user/myapp
      branch: main
    source_dir: /frontend
    build_command: npm run build
    run_command: npx serve build
  
  # Backend
  - name: backend
    github:
      repo: user/myapp
      branch: main
    source_dir: /backend
    build_command: npm run build
    run_command: npm start
    envs:
      - key: DATABASE_URL
        value: ${db.DATABASE_URL}
  
databases:
  - name: db
    engine: PG

# Deploy
doctl apps create --spec .do/app.yaml
```

---

## Interview Questions

### Basic Questions

**Q1: What is cloud computing?**
```
Delivery of computing services over the internet.

Services include:
- Compute (virtual machines)
- Storage (object storage)
- Databases (managed databases)
- Networking (load balancers, CDN)

Benefits:
- Pay as you go
- Scale on demand
- Global reach
- No hardware management
```

**Q2: IaaS vs PaaS vs SaaS?**
```
IaaS (Infrastructure as a Service):
- Virtual machines, storage, networks
- You manage: OS, runtime, app
- Example: AWS EC2, Azure VMs
- Control: High | Management: High

PaaS (Platform as a Service):
- Managed runtime environment
- You manage: Application code only
- Example: Heroku, Vercel
- Control: Medium | Management: Low

SaaS (Software as a Service):
- Fully managed application
- You manage: Data only
- Example: Gmail, Salesforce
- Control: Low | Management: None
```

**Q3: What is a CDN?**
```
Content Delivery Network

Purpose:
- Speed up content delivery
- Serve files from nearest location
- Reduce server load

How it works:
User (Tokyo) → Edge (Tokyo) → Origin (US)
First request: Fetch from origin
Cached at edge for 1 hour
Next requests: Serve from edge (fast!)

Benefits:
- Lower latency
- Better performance
- DDoS protection
- Reduced bandwidth cost
```

### Advanced Questions

**Q4: How do you deploy a full-stack app to AWS?**
```
Architecture:
Route 53 → CloudFront → S3 (Frontend)
                      ↓
                 API Gateway → Lambda/EC2 (Backend)
                      ↓
                    RDS (Database)

Steps:
1. Frontend (S3 + CloudFront):
   - Build React: npm run build
   - Upload to S3: aws s3 sync build/ s3://bucket
   - Create CloudFront distribution
   - Point Route 53 to CloudFront

2. Backend (Elastic Beanstalk):
   - Package: zip -r app.zip .
   - Deploy: eb create production
   - Or use Lambda + API Gateway for serverless

3. Database (RDS):
   - Create PostgreSQL instance
   - Run migrations
   - Update backend connection string

4. Connect:
   - Frontend env: REACT_APP_API_URL
   - Backend env: DATABASE_URL
   - Security groups for network access
```

**Q5: Blue-Green deployment in cloud?**
```
Setup:
- Blue environment: Current (v1.0)
- Green environment: New (v2.0)

AWS Example:
1. Create two Elastic Beanstalk environments
   - myapp-blue (live)
   - myapp-green (new version)

2. Deploy to green:
   eb deploy myapp-green

3. Test green:
   https://myapp-green.elasticbeanstalk.com

4. Swap URLs (switch traffic):
   eb swap myapp-blue --destination_name myapp-green

5. Rollback if needed:
   eb swap again

Benefits:
- Zero downtime
- Instant rollback
- Safe testing

Disadvantage:
- Double infrastructure cost
```

**Q6: How do you scale applications in cloud?**
```
Vertical Scaling (Scale Up):
- Increase instance size
- t3.micro → t3.large
- Limited by hardware
- Requires downtime

Horizontal Scaling (Scale Out):
- Add more instances
- 1 server → 10 servers
- Load balancer distributes traffic
- No downtime

Auto Scaling:
- Automatic based on metrics
- CPU > 70%: Add instance
- CPU < 30%: Remove instance

Example (AWS):
┌──────────────────┐
│  Load Balancer   │
└────────┬─────────┘
         │
    ┌────┴────┐
    │         │
┌───▼──┐  ┌───▼──┐
│ EC2  │  │ EC2  │
│  #1  │  │  #2  │
└──────┘  └──────┘

Auto Scaling Group:
- Min: 2 instances
- Max: 10 instances
- Desired: 2 instances
- Scale up on: CPU > 70%
- Scale down on: CPU < 30%
```

**Q7: What are serverless functions?**
```
Code that runs without managing servers.

Characteristics:
- Event-triggered
- Auto-scaling (0 to ∞)
- Pay per execution
- Stateless

Providers:
- AWS Lambda
- Google Cloud Functions
- Azure Functions
- Vercel Functions
- Netlify Functions

Use Cases:
✅ API endpoints
✅ Image processing
✅ Scheduled jobs
✅ Webhooks
✅ Real-time processing

Limitations:
❌ Cold starts (100-1000ms)
❌ Execution time limits (15 min)
❌ Not for long-running tasks
❌ Harder to debug

Example:
const users = [];

export default async (req, res) => {
  if (req.method === 'GET') {
    res.json(users);
  } else if (req.method === 'POST') {
    users.push(req.body);
    res.json({ success: true });
  }
};
```

**Q8: How do you secure cloud applications?**
```
1. Network Security:
   - Use VPC (Virtual Private Cloud)
   - Security groups (firewall rules)
   - Private subnets for databases

2. Authentication & Authorization:
   - IAM roles and policies
   - Principle of least privilege
   - Multi-factor authentication

3. Data Encryption:
   - In transit: HTTPS/TLS
   - At rest: Encrypt databases
   - Use key management services

4. Secrets Management:
   - Never hardcode secrets
   - Use AWS Secrets Manager
   - Or HashiCorp Vault

5. Monitoring:
   - Enable logging (CloudWatch)
   - Set up alerts
   - Monitor anomalies

6. Updates:
   - Keep software updated
   - Automated patching
   - Regular security scans

Example (AWS):
┌─────────────────── VPC ────────────────────┐
│                                             │
│  ┌── Public Subnet ───┐                    │
│  │                    │                    │
│  │  Load Balancer     │                    │
│  │  (HTTPS only)      │                    │
│  └──────────┬─────────┘                    │
│             │                              │
│  ┌──────────▼────── Private Subnet ──────┐│
│  │                                        ││
│  │  ┌─────────┐    ┌──────────┐          ││
│  │  │   EC2   │────│    RDS   │          ││
│  │  │ Backend │    │ Postgres │          ││
│  │  └─────────┘    └──────────┘          ││
│  │                                        ││
│  │  No public access to database          ││
│  └────────────────────────────────────────┘│
└─────────────────────────────────────────────┘
```

---

## Summary

### Cloud Platform Selection Guide
```
For Prototypes/MVPs:
→ Heroku or Vercel/Netlify
  Quick setup, minimal config

For Frontend-Heavy Apps:
→ Vercel or Netlify
  Optimized for React/Next.js

For Full Control:
→ DigitalOcean or AWS EC2
  VMs, custom configuration

For Enterprise:
→ AWS, GCP, or Azure
  All services, high scalability

For Serverless:
→ AWS Lambda or Vercel Functions
  No server management
```

### Deployment Checklist
```
✅ Environment variables configured
✅ Database migrations run
✅ SSL/HTTPS enabled
✅ Domain configured
✅ Monitoring set up
✅ Backups enabled
✅ CI/CD pipeline working
✅ Security groups configured
✅ Auto-scaling configured
✅ Error tracking (Sentry)
```

### Key Takeaways
1. **Start Simple**: Use PaaS (Heroku/Vercel) for MVPs
2. **Scale Gradually**: Move to IaaS (AWS/GCP) when needed
3. **Use Managed Services**: Databases, caching, CDN
4. **Automate Everything**: CI/CD pipelines, scaling
5. **Monitor Proactively**: Logs, metrics, alerts
6. **Secure by Default**: HTTPS, secrets, least privilege
7. **Global Distribution**: CDN for static assets
8. **Plan for Scale**: Auto-scaling, load balancing

Cloud platforms make deployment and scaling easier than ever! ☁️
