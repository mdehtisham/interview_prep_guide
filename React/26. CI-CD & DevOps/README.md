# CI/CD & DevOps

> Expert / Architect Level (5+ years)

---

## Questions

251. How do you set up CI/CD for React applications?
252. What is the deployment strategy for React apps?
253. How do you implement preview deployments?
254. What is the difference between CSR, SSR, and SSG deployments?
255. How do you optimize React apps for CDN delivery?
256. What is edge rendering and when to use it?
257. How do you implement blue-green deployments?
258. What is canary deployment for React apps?
259. How do you handle environment variables securely?
260. What is Docker and containerization for React apps?

---

## Detailed Answers

### 251. How do you set up CI/CD for React applications?

<details>
<summary>View Answer</summary>

**CI/CD (Continuous Integration/Continuous Deployment)** automates the process of testing, building, and deploying React applications. This ensures **code quality**, **faster deployments**, and **reduced manual errors**.

---

#### **1. GitHub Actions CI/CD Pipeline**

**Basic Workflow:**

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [18.x, 20.x]
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linter
        run: npm run lint
      
      - name: Run type check
        run: npm run type-check
      
      - name: Run tests
        run: npm run test:ci
        env:
          CI: true
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/coverage-final.json
```

**Deployment Workflow:**

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20.x'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build application
        run: npm run build
        env:
          VITE_API_URL: ${{ secrets.VITE_API_URL }}
          VITE_APP_VERSION: ${{ github.sha }}
      
      - name: Run tests
        run: npm run test:ci
      
      - name: Deploy to production
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          command: pages deploy dist --project-name=my-app
```

---

#### **2. Advanced GitHub Actions with Caching**

```yaml
# .github/workflows/advanced-ci.yml
name: Advanced CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '20.x'
  CACHE_KEY_PREFIX: npm-deps

jobs:
  setup:
    runs-on: ubuntu-latest
    outputs:
      cache-hit: ${{ steps.cache.outputs.cache-hit }}
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
      
      - name: Cache dependencies
        id: cache
        uses: actions/cache@v3
        with:
          path: |
            node_modules
            ~/.npm
          key: ${{ runner.os }}-${{ env.CACHE_KEY_PREFIX }}-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-${{ env.CACHE_KEY_PREFIX }}-
      
      - name: Install dependencies
        if: steps.cache.outputs.cache-hit != 'true'
        run: npm ci

  lint:
    needs: setup
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
      
      - name: Restore cache
        uses: actions/cache@v3
        with:
          path: node_modules
          key: ${{ runner.os }}-${{ env.CACHE_KEY_PREFIX }}-${{ hashFiles('**/package-lock.json') }}
      
      - name: Run ESLint
        run: npm run lint
      
      - name: Run Prettier
        run: npm run format:check

  test:
    needs: setup
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
      
      - name: Restore cache
        uses: actions/cache@v3
        with:
          path: node_modules
          key: ${{ runner.os }}-${{ env.CACHE_KEY_PREFIX }}-${{ hashFiles('**/package-lock.json') }}
      
      - name: Run unit tests
        run: npm run test:ci
      
      - name: Run E2E tests
        run: npm run test:e2e
      
      - name: Generate coverage report
        run: npm run coverage
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3

  build:
    needs: [lint, test]
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
      
      - name: Restore cache
        uses: actions/cache@v3
        with:
          path: node_modules
          key: ${{ runner.os }}-${{ env.CACHE_KEY_PREFIX }}-${{ hashFiles('**/package-lock.json') }}
      
      - name: Build application
        run: npm run build
        env:
          VITE_API_URL: ${{ secrets.VITE_API_URL }}
      
      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: dist
          path: dist/
          retention-days: 7

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Download build artifacts
        uses: actions/download-artifact@v3
        with:
          name: dist
          path: dist/
      
      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'
```

---

#### **3. GitLab CI/CD Pipeline**

```yaml
# .gitlab-ci.yml
image: node:20

stages:
  - install
  - test
  - build
  - deploy

variables:
  NPM_CONFIG_CACHE: $CI_PROJECT_DIR/.npm
  CYPRESS_CACHE_FOLDER: $CI_PROJECT_DIR/.cypress

cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/
    - .npm/
    - .cypress/

install:
  stage: install
  script:
    - npm ci
  artifacts:
    paths:
      - node_modules/
    expire_in: 1 hour

lint:
  stage: test
  dependencies:
    - install
  script:
    - npm run lint
    - npm run format:check

unit-test:
  stage: test
  dependencies:
    - install
  script:
    - npm run test:ci
  coverage: '/All files[^|]*\|[^|]*\s+([\d\.]+)/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

e2e-test:
  stage: test
  dependencies:
    - install
  script:
    - npm run test:e2e
  artifacts:
    when: on_failure
    paths:
      - cypress/screenshots/
      - cypress/videos/
    expire_in: 1 week

build:
  stage: build
  dependencies:
    - install
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week

deploy-staging:
  stage: deploy
  dependencies:
    - build
  script:
    - npm install -g netlify-cli
    - netlify deploy --site $NETLIFY_SITE_ID --auth $NETLIFY_AUTH_TOKEN --dir=dist
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - develop

deploy-production:
  stage: deploy
  dependencies:
    - build
  script:
    - npm install -g netlify-cli
    - netlify deploy --site $NETLIFY_SITE_ID --auth $NETLIFY_AUTH_TOKEN --dir=dist --prod
  environment:
    name: production
    url: https://example.com
  only:
    - main
  when: manual
```

---

#### **4. CircleCI Configuration**

```yaml
# .circleci/config.yml
version: 2.1

orbs:
  node: circleci/node@5.1

jobs:
  test:
    docker:
      - image: cimg/node:20.10
    
    steps:
      - checkout
      
      - restore_cache:
          keys:
            - v1-dependencies-{{ checksum "package-lock.json" }}
            - v1-dependencies-
      
      - run:
          name: Install dependencies
          command: npm ci
      
      - save_cache:
          paths:
            - node_modules
          key: v1-dependencies-{{ checksum "package-lock.json" }}
      
      - run:
          name: Run linter
          command: npm run lint
      
      - run:
          name: Run tests
          command: npm run test:ci
      
      - run:
          name: Generate coverage
          command: npm run coverage
      
      - store_test_results:
          path: test-results
      
      - store_artifacts:
          path: coverage

  build:
    docker:
      - image: cimg/node:20.10
    
    steps:
      - checkout
      
      - restore_cache:
          keys:
            - v1-dependencies-{{ checksum "package-lock.json" }}
      
      - run:
          name: Build application
          command: npm run build
      
      - persist_to_workspace:
          root: .
          paths:
            - dist

  deploy:
    docker:
      - image: cimg/node:20.10
    
    steps:
      - attach_workspace:
          at: .
      
      - run:
          name: Deploy to AWS S3
          command: |
            aws s3 sync dist/ s3://$S3_BUCKET --delete
            aws cloudfront create-invalidation --distribution-id $CLOUDFRONT_ID --paths "/*"

workflows:
  version: 2
  build-test-deploy:
    jobs:
      - test
      - build:
          requires:
            - test
      - deploy:
          requires:
            - build
          filters:
            branches:
              only: main
```

---

#### **5. Package.json Scripts for CI/CD**

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
    "lint:fix": "eslint . --ext ts,tsx --fix",
    "format": "prettier --write \"src/**/*.{ts,tsx,json,css,md}\"",
    "format:check": "prettier --check \"src/**/*.{ts,tsx,json,css,md}\"",
    "type-check": "tsc --noEmit",
    "test": "vitest",
    "test:ci": "vitest run --coverage",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "coverage": "vitest run --coverage",
    "prepare": "husky install"
  }
}
```

---

#### **6. Environment Variables Management**

**GitHub Actions Secrets:**

```yaml
# .github/workflows/deploy.yml
env:
  VITE_API_URL: ${{ secrets.VITE_API_URL }}
  VITE_SENTRY_DSN: ${{ secrets.VITE_SENTRY_DSN }}
  VITE_GA_ID: ${{ secrets.VITE_GA_ID }}
```

**Environment-specific .env files:**

```bash
# .env.production
VITE_API_URL=https://api.production.com
VITE_ENV=production

# .env.staging
VITE_API_URL=https://api.staging.com
VITE_ENV=staging

# .env.development
VITE_API_URL=http://localhost:3000
VITE_ENV=development
```

**Load environment in workflow:**

```yaml
- name: Load environment variables
  run: |
    echo "VITE_API_URL=${{ secrets.VITE_API_URL }}" >> $GITHUB_ENV
    echo "VITE_APP_VERSION=${{ github.sha }}" >> $GITHUB_ENV
```

---

#### **7. Docker Integration**

**Multi-stage Dockerfile:**

```dockerfile
# Dockerfile
FROM node:20-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine

COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

**GitHub Actions with Docker:**

```yaml
# .github/workflows/docker.yml
name: Docker Build and Push

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            myapp/react-app:latest
            myapp/react-app:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

---

#### **8. Pre-commit Hooks with Husky**

```bash
npm install -D husky lint-staged
npm run prepare
```

**Configure Husky:**

```bash
# .husky/pre-commit
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

npx lint-staged
```

**Configure lint-staged:**

```json
// package.json
{
  "lint-staged": {
    "*.{ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{json,css,md}": [
      "prettier --write"
    ]
  }
}
```

---

#### **Best Practices**

1. **Automate Everything**: Tests, linting, building, deployment
2. **Fast Feedback**: Run quick checks first (lint, type-check)
3. **Caching**: Cache dependencies for faster builds
4. **Parallel Jobs**: Run tests and lint in parallel
5. **Fail Fast**: Stop pipeline on first failure
6. **Security**: Use secrets for sensitive data
7. **Artifacts**: Store build outputs for debugging
8. **Notifications**: Alert team on failures

---

#### **Summary**

**CI/CD Pipeline Stages:**

1. **Install**: Cache and install dependencies
2. **Lint**: ESLint, Prettier, TypeScript checks
3. **Test**: Unit tests, integration tests, E2E tests
4. **Build**: Create production bundle
5. **Deploy**: Push to hosting platform

**Popular CI/CD Platforms:**
- **GitHub Actions**: Integrated with GitHub
- **GitLab CI**: Built into GitLab
- **CircleCI**: Fast with Docker support
- **Jenkins**: Self-hosted, highly configurable
- **Travis CI**: Simple YAML configuration
- **Azure Pipelines**: Microsoft ecosystem

**Key Features:**
- Automated testing on every commit
- Code quality checks (lint, format)
- Coverage reporting
- Environment-specific builds
- Automated deployments
- Rollback capabilities
- Build artifacts storage
- Notification systems

**Benefits:**
- Faster time to production
- Consistent build process
- Early bug detection
- Reduced manual errors
- Better code quality
- Improved collaboration

Proper CI/CD setup ensures **reliable**, **fast**, and **automated** delivery of React applications.

</details>

---

### 252. What is the deployment strategy for React apps?

<details>
<summary>View Answer</summary>

**React deployment strategies** vary based on rendering method (CSR/SSR/SSG), hosting requirements, and scalability needs. Choosing the right strategy impacts **performance**, **cost**, and **user experience**.

---

#### **1. Static Hosting (CSR - Client-Side Rendering)**

**Best for**: Traditional React SPAs with client-side routing

**Build Process:**

```bash
# Build for production
npm run build

# Output: dist/ folder with static files
# - index.html
# - assets/
#   - main-[hash].js
#   - main-[hash].css
```

**Deployment Platforms:**

1. **Vercel** (Recommended for simplicity)

```bash
# Install Vercel CLI
npm install -g vercel

# Deploy
vercel --prod
```

**vercel.json configuration:**

```json
{
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "rewrites": [
    { "source": "/(.*)", "destination": "/index.html" }
  ],
  "headers": [
    {
      "source": "/assets/(.*)",
      "headers": [
        {
          "key": "Cache-Control",
          "value": "public, max-age=31536000, immutable"
        }
      ]
    }
  ]
}
```

2. **Netlify**

```bash
# Install Netlify CLI
npm install -g netlify-cli

# Deploy
netlify deploy --prod --dir=dist
```

**netlify.toml configuration:**

```toml
[build]
  command = "npm run build"
  publish = "dist"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

[[headers]]
  for = "/assets/*"
  [headers.values]
    Cache-Control = "public, max-age=31536000, immutable"

[[headers]]
  for = "/*.js"
  [headers.values]
    Content-Type = "application/javascript; charset=utf-8"

[[headers]]
  for = "/*.css"
  [headers.values]
    Content-Type = "text/css; charset=utf-8"
```

3. **AWS S3 + CloudFront**

```bash
# Build application
npm run build

# Sync to S3
aws s3 sync dist/ s3://my-bucket-name --delete

# Invalidate CloudFront cache
aws cloudfront create-invalidation \
  --distribution-id E1234567890ABC \
  --paths "/*"
```

**S3 bucket policy:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket-name/*"
    }
  ]
}
```

---

#### **2. SSR Deployment (Next.js, Remix)**

**Node.js Server:**

```javascript
// server.js (Custom Next.js server)
const { createServer } = require('http');
const { parse } = require('url');
const next = require('next');

const dev = process.env.NODE_ENV !== 'production';
const app = next({ dev });
const handle = app.getRequestHandler();

app.prepare().then(() => {
  createServer((req, res) => {
    const parsedUrl = parse(req.url, true);
    handle(req, res, parsedUrl);
  }).listen(3000, (err) => {
    if (err) throw err;
    console.log('> Ready on http://localhost:3000');
  });
});
```

**Deployment Options:**

1. **Vercel (Native Next.js support)**

```bash
vercel --prod
```

2. **AWS Elastic Beanstalk**

```bash
# Initialize EB
eb init

# Create environment
eb create production-env

# Deploy
eb deploy
```

**Procfile:**

```
web: npm start
```

3. **Docker + Kubernetes**

```dockerfile
# Dockerfile
FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app

ENV NODE_ENV production

COPY --from=deps /app/node_modules ./node_modules
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public
COPY --from=builder /app/package.json ./package.json

EXPOSE 3000

CMD ["npm", "start"]
```

**Kubernetes Deployment:**

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: react-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: react-app
  template:
    metadata:
      labels:
        app: react-app
    spec:
      containers:
      - name: react-app
        image: myregistry/react-app:latest
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        resources:
          limits:
            memory: "512Mi"
            cpu: "500m"
          requests:
            memory: "256Mi"
            cpu: "250m"
---
apiVersion: v1
kind: Service
metadata:
  name: react-app-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 3000
  selector:
    app: react-app
```

---

#### **3. Edge Deployment (Cloudflare, Vercel Edge)**

**Cloudflare Pages:**

```bash
# Install Wrangler
npm install -g wrangler

# Deploy
wrangler pages deploy dist
```

**Cloudflare Workers (Edge Functions):**

```typescript
// functions/api/[[path]].ts
export async function onRequest(context) {
  const { request, env } = context;
  
  // Handle API requests at the edge
  if (request.url.includes('/api/')) {
    return new Response(JSON.stringify({ message: 'Hello from edge' }), {
      headers: { 'Content-Type': 'application/json' },
    });
  }
  
  // Serve static assets
  return context.next();
}
```

**Vercel Edge Functions:**

```typescript
// pages/api/edge.ts
import { NextRequest, NextResponse } from 'next/server';

export const config = {
  runtime: 'edge',
};

export default function handler(req: NextRequest) {
  return NextResponse.json({
    message: 'Hello from Vercel Edge',
    region: process.env.VERCEL_REGION,
  });
}
```

---

#### **4. Serverless Deployment**

**AWS Lambda + API Gateway:**

```javascript
// serverless.yml
service: react-app

provider:
  name: aws
  runtime: nodejs20.x
  region: us-east-1

functions:
  app:
    handler: lambda.handler
    events:
      - http:
          path: /
          method: ANY
      - http:
          path: /{proxy+}
          method: ANY

package:
  exclude:
    - node_modules/**
    - src/**
```

**Lambda Handler:**

```javascript
// lambda.js
const awsServerlessExpress = require('aws-serverless-express');
const app = require('./server');

const server = awsServerlessExpress.createServer(app);

exports.handler = (event, context) => {
  awsServerlessExpress.proxy(server, event, context);
};
```

---

#### **5. Multi-Environment Strategy**

**Environment Configuration:**

```typescript
// src/config/environment.ts
const environments = {
  development: {
    apiUrl: 'http://localhost:3000',
    sentryDsn: '',
    analyticsId: '',
  },
  staging: {
    apiUrl: 'https://api-staging.example.com',
    sentryDsn: 'https://sentry-staging',
    analyticsId: 'UA-STAGING',
  },
  production: {
    apiUrl: 'https://api.example.com',
    sentryDsn: 'https://sentry-prod',
    analyticsId: 'UA-PROD',
  },
};

export const config = environments[import.meta.env.MODE || 'development'];
```

**Branch-based Deployment:**

```yaml
# .github/workflows/deploy-env.yml
name: Deploy by Environment

on:
  push:
    branches:
      - main
      - staging
      - develop

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Determine environment
        id: env
        run: |
          if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
            echo "environment=production" >> $GITHUB_OUTPUT
            echo "url=https://app.example.com" >> $GITHUB_OUTPUT
          elif [[ "${{ github.ref }}" == "refs/heads/staging" ]]; then
            echo "environment=staging" >> $GITHUB_OUTPUT
            echo "url=https://staging.example.com" >> $GITHUB_OUTPUT
          else
            echo "environment=development" >> $GITHUB_OUTPUT
            echo "url=https://dev.example.com" >> $GITHUB_OUTPUT
          fi
      
      - name: Build with environment
        run: npm run build
        env:
          VITE_ENV: ${{ steps.env.outputs.environment }}
      
      - name: Deploy
        run: |
          echo "Deploying to ${{ steps.env.outputs.environment }}"
          # Deployment command
```

---

#### **6. Rollback Strategy**

**Version Tagging:**

```bash
# Tag release
git tag -a v1.2.3 -m "Release version 1.2.3"
git push origin v1.2.3

# Rollback to previous version
git checkout v1.2.2
git push origin HEAD:main --force
```

**Vercel Rollback:**

```bash
# List deployments
vercel list

# Rollback to specific deployment
vercel rollback [deployment-url]
```

**AWS S3 Versioning:**

```bash
# Enable versioning
aws s3api put-bucket-versioning \
  --bucket my-bucket \
  --versioning-configuration Status=Enabled

# List versions
aws s3api list-object-versions --bucket my-bucket

# Restore previous version
aws s3api copy-object \
  --copy-source my-bucket/index.html?versionId=VERSION_ID \
  --bucket my-bucket \
  --key index.html
```

---

#### **7. Deployment Checklist**

```typescript
// src/utils/deploymentChecks.ts
export function performDeploymentChecks() {
  const checks = {
    envVariables: checkEnvironmentVariables(),
    buildOptimization: checkBuildOptimization(),
    security: checkSecurityHeaders(),
    performance: checkPerformanceMetrics(),
    monitoring: checkMonitoringSetup(),
  };

  const failed = Object.entries(checks).filter(([, passed]) => !passed);
  
  if (failed.length > 0) {
    console.error('Deployment checks failed:', failed);
    process.exit(1);
  }
}

function checkEnvironmentVariables() {
  const required = ['VITE_API_URL', 'VITE_SENTRY_DSN'];
  return required.every(key => import.meta.env[key]);
}

function checkBuildOptimization() {
  // Check if source maps are disabled in production
  return import.meta.env.PROD ? !import.meta.env.VITE_SOURCEMAP : true;
}

function checkSecurityHeaders() {
  // Verify security headers are configured
  return true; // Implement actual check
}

function checkPerformanceMetrics() {
  // Verify performance budgets
  return true; // Implement actual check
}

function checkMonitoringSetup() {
  // Verify monitoring tools are initialized
  return !!window.Sentry && !!window.gtag;
}
```

---

#### **Platform Comparison**

| Platform | Type | SSR Support | Edge | Pricing | Best For |
|----------|------|-------------|------|---------|----------|
| **Vercel** | Serverless | Yes (Next.js) | Yes | Free tier + usage | Next.js apps |
| **Netlify** | Static/Serverless | Limited | Yes | Free tier + usage | Static sites |
| **Cloudflare Pages** | Edge | Yes | Yes | Free tier + usage | Global apps |
| **AWS S3+CloudFront** | Static CDN | No | Yes | Pay per use | Large scale |
| **Heroku** | Container | Yes | No | $7+/month | Node.js apps |
| **AWS Amplify** | Managed | Yes | Yes | Pay per use | AWS ecosystem |
| **DigitalOcean** | VPS | Yes | No | $5+/month | Full control |

---

#### **Best Practices**

1. **Automated Deployments**: Use CI/CD pipelines
2. **Environment Separation**: Separate dev, staging, production
3. **Version Control**: Tag releases for easy rollback
4. **Health Checks**: Monitor deployment success
5. **Gradual Rollouts**: Use canary/blue-green deployments
6. **Cache Strategy**: Optimize caching for static assets
7. **Security**: Use HTTPS, security headers, CSP
8. **Monitoring**: Set up error tracking and analytics
9. **Documentation**: Document deployment process
10. **Backup**: Keep deployment artifacts

---

#### **Summary**

**Deployment Strategies:**

1. **Static Hosting**: Vercel, Netlify, S3 (CSR apps)
2. **SSR Hosting**: Vercel, Node servers (Next.js, Remix)
3. **Edge Deployment**: Cloudflare, Vercel Edge (global)
4. **Serverless**: AWS Lambda, Vercel Functions
5. **Container**: Docker + Kubernetes (scalable)

**Key Considerations:**
- **Rendering method**: CSR vs SSR vs SSG
- **Scaling needs**: Static vs dynamic scaling
- **Geographic distribution**: CDN vs edge vs regions
- **Cost**: Free tiers vs usage-based vs fixed
- **Control**: Managed platforms vs self-hosted

**Recommended Stack:**
- **Simple SPA**: Vercel/Netlify + GitHub Actions
- **SSR App**: Vercel + Next.js
- **Enterprise**: AWS S3 + CloudFront + CI/CD
- **Global App**: Cloudflare Pages + Workers

Proper deployment strategy ensures **reliable**, **fast**, and **scalable** React applications.

</details>

---

### 253. How do you implement preview deployments?

<details>
<summary>View Answer</summary>

**Preview deployments** (also called **deploy previews**) create temporary environments for each pull request or branch, enabling **testing** and **review** before merging to production.

---

#### **1. Vercel Automatic Preview Deployments**

**Setup:**

Vercel automatically creates preview deployments for every push to a branch.

```bash
# Install Vercel
npm install -g vercel

# Link project
vercel link

# Every push creates a preview deployment
git push origin feature-branch
```

**GitHub Integration:**

Vercel automatically comments on PRs with preview URLs:

```
✅ Preview deployment ready!
🔍 Inspect: https://vercel.com/user/project/deployment-id
🌐 Preview: https://project-git-feature-user.vercel.app
```

**Configure Preview Settings:**

```json
// vercel.json
{
  "github": {
    "enabled": true,
    "autoAlias": true,
    "silent": false,
    "autoJobCancelation": true
  },
  "env": {
    "VITE_API_URL": "https://api-staging.example.com"
  },
  "build": {
    "env": {
      "VITE_PREVIEW": "true"
    }
  }
}
```

---

#### **2. Netlify Deploy Previews**

**Automatic Setup:**

Netlify creates deploy previews for all pull requests by default.

**netlify.toml configuration:**

```toml
[build]
  command = "npm run build"
  publish = "dist"

[context.deploy-preview]
  command = "npm run build:preview"
  
[context.deploy-preview.environment]
  VITE_API_URL = "https://api-preview.example.com"
  VITE_PREVIEW_MODE = "true"

[context.branch-deploy]
  command = "npm run build"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

[[headers]]
  for = "/assets/*"
  [headers.values]
    Cache-Control = "public, max-age=31536000, immutable"
```

**Netlify CLI:**

```bash
# Deploy preview manually
netlify deploy --alias=feature-xyz

# Deploy to production
netlify deploy --prod
```

---

#### **3. GitHub Actions Custom Preview Deployments**

**Workflow for PR Previews:**

```yaml
# .github/workflows/preview-deploy.yml
name: Preview Deployment

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  preview:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20.x'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build application
        run: npm run build
        env:
          VITE_API_URL: ${{ secrets.PREVIEW_API_URL }}
          VITE_PREVIEW: 'true'
          VITE_PR_NUMBER: ${{ github.event.pull_request.number }}
      
      - name: Deploy to preview environment
        id: deploy
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          command: pages deploy dist --project-name=my-app --branch=pr-${{ github.event.pull_request.number }}
      
      - name: Comment PR with preview URL
        uses: actions/github-script@v7
        with:
          script: |
            const prNumber = context.issue.number;
            const previewUrl = `https://pr-${prNumber}.my-app.pages.dev`;
            
            github.rest.issues.createComment({
              issue_number: prNumber,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `## 🚀 Preview Deployment\n\n✅ Your preview is ready!\n\n🔗 **Preview URL:** ${previewUrl}\n\n> This preview will be automatically deleted when the PR is closed.`
            });
```

**Cleanup Workflow:**

```yaml
# .github/workflows/cleanup-preview.yml
name: Cleanup Preview

on:
  pull_request:
    types: [closed]

jobs:
  cleanup:
    runs-on: ubuntu-latest
    
    steps:
      - name: Delete preview deployment
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          command: pages deployment delete --project-name=my-app --branch=pr-${{ github.event.pull_request.number }}
      
      - name: Comment PR
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '✨ Preview deployment has been cleaned up.'
            });
```

---

#### **4. AWS S3 Preview Deployments**

**Script for S3 Preview:**

```bash
#!/bin/bash
# scripts/deploy-preview.sh

PR_NUMBER=$1
BUCKET_NAME="my-app-previews"
PREVIEW_PREFIX="pr-${PR_NUMBER}"

# Build application
npm run build

# Deploy to S3 with PR prefix
aws s3 sync dist/ s3://${BUCKET_NAME}/${PREVIEW_PREFIX}/ --delete

# Output preview URL
echo "Preview URL: https://${BUCKET_NAME}.s3-website-us-east-1.amazonaws.com/${PREVIEW_PREFIX}/"
```

**GitHub Actions Integration:**

```yaml
# .github/workflows/s3-preview.yml
name: S3 Preview Deploy

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20.x'
      
      - run: npm ci
      - run: npm run build
      
      - name: Deploy to S3
        run: |
          aws s3 sync dist/ s3://my-app-previews/pr-${{ github.event.pull_request.number }}/ --delete
      
      - name: Comment preview URL
        uses: actions/github-script@v7
        with:
          script: |
            const previewUrl = `https://my-app-previews.s3-website-us-east-1.amazonaws.com/pr-${context.issue.number}/`;
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `🔍 Preview: ${previewUrl}`
            });
```

---

#### **5. Docker-based Preview Environments**

**Docker Compose for Previews:**

```yaml
# docker-compose.preview.yml
version: '3.8'

services:
  preview-${PR_NUMBER}:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        - VITE_API_URL=${PREVIEW_API_URL}
        - VITE_PR_NUMBER=${PR_NUMBER}
    ports:
      - "${PORT}:80"
    environment:
      - NODE_ENV=production
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.pr-${PR_NUMBER}.rule=Host(`pr-${PR_NUMBER}.preview.example.com`)"
      - "traefik.http.services.pr-${PR_NUMBER}.loadbalancer.server.port=80"
```

**Deployment Script:**

```bash
#!/bin/bash
# scripts/deploy-docker-preview.sh

PR_NUMBER=$1
PORT=$((8000 + PR_NUMBER))

export PR_NUMBER
export PORT
export PREVIEW_API_URL="https://api-preview.example.com"

# Build and start container
docker-compose -f docker-compose.preview.yml up -d

echo "Preview available at: http://pr-${PR_NUMBER}.preview.example.com"
```

---

#### **6. Preview Environment with Database**

**Create Isolated Preview Database:**

```yaml
# .github/workflows/preview-with-db.yml
name: Preview with Database

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Create preview database
        run: |
          # Create database for this PR
          curl -X POST https://api.planetscale.com/v1/organizations/$ORG/databases \
            -H "Authorization: Bearer ${{ secrets.PLANETSCALE_TOKEN }}" \
            -d '{"name": "myapp-pr-${{ github.event.pull_request.number }}"}'
      
      - name: Run migrations
        run: |
          DATABASE_URL=${{ secrets.PREVIEW_DB_URL }} npm run migrate
      
      - name: Seed preview data
        run: |
          DATABASE_URL=${{ secrets.PREVIEW_DB_URL }} npm run seed:preview
      
      - name: Deploy application
        run: |
          # Deploy with preview database URL
          vercel --env DATABASE_URL=${{ secrets.PREVIEW_DB_URL }}
```

---

#### **7. Preview Deployment Manager**

```typescript
// scripts/preview-manager.ts
import { Octokit } from '@octokit/rest';

interface PreviewDeployment {
  prNumber: number;
  url: string;
  status: 'building' | 'ready' | 'error';
  createdAt: Date;
  updatedAt: Date;
}

class PreviewDeploymentManager {
  private octokit: Octokit;
  private deployments: Map<number, PreviewDeployment> = new Map();

  constructor(token: string) {
    this.octokit = new Octokit({ auth: token });
  }

  async createPreview(prNumber: number) {
    const deployment: PreviewDeployment = {
      prNumber,
      url: `https://pr-${prNumber}.preview.example.com`,
      status: 'building',
      createdAt: new Date(),
      updatedAt: new Date(),
    };

    this.deployments.set(prNumber, deployment);

    // Comment on PR
    await this.commentOnPR(prNumber, `
## 🔨 Building Preview Deployment

Your preview is being built...

⏳ Status: Building
    `);

    return deployment;
  }

  async updatePreviewStatus(
    prNumber: number,
    status: PreviewDeployment['status']
  ) {
    const deployment = this.deployments.get(prNumber);
    if (!deployment) return;

    deployment.status = status;
    deployment.updatedAt = new Date();

    const message = status === 'ready'
      ? `
## ✅ Preview Deployment Ready

🔗 **Preview URL:** ${deployment.url}

> Preview updated at ${deployment.updatedAt.toLocaleString()}
      `
      : `
## ❌ Preview Deployment Failed

Please check the build logs for errors.
      `;

    await this.commentOnPR(prNumber, message);
  }

  async deletePreview(prNumber: number) {
    this.deployments.delete(prNumber);
    
    await this.commentOnPR(prNumber, `
## 🧹 Preview Cleaned Up

Preview deployment has been removed.
    `);
  }

  private async commentOnPR(prNumber: number, body: string) {
    const [owner, repo] = process.env.GITHUB_REPOSITORY!.split('/');
    
    await this.octokit.issues.createComment({
      owner,
      repo,
      issue_number: prNumber,
      body,
    });
  }

  async listActivePreviews() {
    return Array.from(this.deployments.values())
      .filter(d => d.status === 'ready')
      .sort((a, b) => b.updatedAt.getTime() - a.updatedAt.getTime());
  }
}

export const previewManager = new PreviewDeploymentManager(
  process.env.GITHUB_TOKEN!
);
```

---

#### **8. Testing Preview Deployments**

**Automated E2E Tests on Preview:**

```yaml
# .github/workflows/preview-e2e.yml
name: E2E Tests on Preview

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  e2e:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Wait for preview deployment
        uses: nev7n/wait_for_response@v1
        with:
          url: 'https://pr-${{ github.event.pull_request.number }}.preview.example.com'
          responseCode: 200
          timeout: 300000
          interval: 5000
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20.x'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run E2E tests
        run: npm run test:e2e
        env:
          PLAYWRIGHT_BASE_URL: https://pr-${{ github.event.pull_request.number }}.preview.example.com
      
      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: playwright-report
          path: playwright-report/
```

---

#### **9. Preview Deployment Dashboard**

```typescript
// src/components/PreviewDashboard.tsx
import { useState, useEffect } from 'react';

interface Preview {
  id: string;
  prNumber: number;
  branch: string;
  url: string;
  status: string;
  author: string;
  createdAt: string;
}

function PreviewDashboard() {
  const [previews, setPreviews] = useState<Preview[]>([]);

  useEffect(() => {
    fetchPreviews();
  }, []);

  const fetchPreviews = async () => {
    const response = await fetch('/api/previews');
    const data = await response.json();
    setPreviews(data);
  };

  const deletePreview = async (id: string) => {
    await fetch(`/api/previews/${id}`, { method: 'DELETE' });
    fetchPreviews();
  };

  return (
    <div className="preview-dashboard">
      <h1>Active Preview Deployments</h1>
      
      <div className="preview-list">
        {previews.map(preview => (
          <div key={preview.id} className="preview-card">
            <div className="preview-header">
              <h3>PR #{preview.prNumber} - {preview.branch}</h3>
              <span className={`status status-${preview.status}`}>
                {preview.status}
              </span>
            </div>
            
            <div className="preview-info">
              <p>Author: {preview.author}</p>
              <p>Created: {new Date(preview.createdAt).toLocaleString()}</p>
            </div>
            
            <div className="preview-actions">
              <a href={preview.url} target="_blank" rel="noopener">
                Open Preview
              </a>
              <button onClick={() => deletePreview(preview.id)}>
                Delete
              </button>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

#### **Best Practices**

1. **Automatic Cleanup**: Delete previews when PR is closed
2. **Environment Isolation**: Use separate databases/APIs for previews
3. **Security**: Require authentication for preview environments
4. **Cost Management**: Set TTL for preview deployments
5. **Clear URLs**: Use predictable preview URLs (pr-123.preview.com)
6. **Status Updates**: Comment on PR with preview status
7. **E2E Testing**: Run automated tests on previews
8. **Visual Regression**: Compare screenshots with production

---

#### **Summary**

**Preview Deployment Platforms:**

1. **Vercel**: Automatic previews for every push
2. **Netlify**: Deploy previews with custom configuration
3. **Cloudflare Pages**: Branch-based preview deployments
4. **Custom S3**: Manual preview deployments with S3
5. **Docker**: Container-based preview environments

**Key Features:**
- Automatic creation on PR open
- Unique URL per preview
- Automatic cleanup on PR close
- PR comments with preview URL
- Environment isolation
- E2E test integration
- Visual regression testing

**Benefits:**
- Review changes before merging
- Test features in isolation
- Share work with stakeholders
- Catch issues early
- Improve collaboration
- Reduce production bugs

**Use Cases:**
- Feature review
- Design approval
- Client demos
- E2E testing
- QA validation
- Stakeholder feedback

Preview deployments enable **faster feedback loops** and **better collaboration** in the development process.

</details>

---

### 254. What is the difference between CSR, SSR, and SSG deployments?

<details>
<summary>View Answer</summary>

**CSR (Client-Side Rendering)**, **SSR (Server-Side Rendering)**, and **SSG (Static Site Generation)** are different rendering strategies that impact **deployment**, **performance**, and **SEO**.

---

#### **1. CSR (Client-Side Rendering)**

**How it Works:**

1. Server sends minimal HTML with JavaScript bundles
2. Browser downloads and executes JavaScript
3. JavaScript fetches data and renders UI
4. Page becomes interactive

**Traditional React App:**

```html
<!-- dist/index.html -->
<!DOCTYPE html>
<html>
  <head>
    <title>My App</title>
    <link rel="stylesheet" href="/assets/main.abc123.css">
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/assets/main.def456.js"></script>
  </body>
</html>
```

**Deployment:**

```bash
# Build
npm run build

# Deploy to static hosting
# Output: Static files (HTML, JS, CSS)
# Hosting: Vercel, Netlify, S3, CDN
```

**Characteristics:**
- ✅ Simple deployment (static files)
- ✅ Cheap hosting (CDN, S3)
- ✅ Good for dynamic, interactive apps
- ❌ Slow initial load
- ❌ Poor SEO (empty HTML)
- ❌ No content for web crawlers

**Use Cases:**
- Internal dashboards
- Admin panels
- SaaS applications
- Apps behind authentication

---

#### **2. SSR (Server-Side Rendering)**

**How it Works:**

1. Server renders React components to HTML
2. Server sends fully-rendered HTML
3. Browser displays HTML immediately
4. Browser downloads JavaScript
5. React "hydrates" the page (makes it interactive)

**Next.js SSR Example:**

```typescript
// pages/posts/[id].tsx
import { GetServerSideProps } from 'next';

interface Post {
  id: string;
  title: string;
  content: string;
}

interface Props {
  post: Post;
}

export default function PostPage({ post }: Props) {
  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </div>
  );
}

// Runs on every request
export const getServerSideProps: GetServerSideProps<Props> = async (context) => {
  const { id } = context.params!;
  
  // Fetch data on server
  const response = await fetch(`https://api.example.com/posts/${id}`);
  const post = await response.json();

  return {
    props: { post },
  };
};
```

**Server Response:**

```html
<!DOCTYPE html>
<html>
  <head>
    <title>My Post</title>
  </head>
  <body>
    <div id="__next">
      <div>
        <h1>My Post Title</h1>
        <p>This content was rendered on the server</p>
      </div>
    </div>
    <script src="/_next/static/chunks/main.js"></script>
  </body>
</html>
```

**Deployment:**

```bash
# Build Next.js app
npm run build

# Start Node.js server
npm start

# Or deploy to serverless
vercel deploy --prod
```

**Deployment Requirements:**
- Node.js server (Vercel, AWS Lambda, DigitalOcean)
- Always-on server or serverless functions
- Database/API access from server

**Characteristics:**
- ✅ Fast initial load (HTML ready)
- ✅ Great SEO (full HTML content)
- ✅ Dynamic content on each request
- ❌ Requires Node.js server
- ❌ Higher hosting costs
- ❌ Server load increases with traffic

**Use Cases:**
- E-commerce product pages
- News websites
- Social media feeds
- Personalized content
- Real-time data

---

#### **3. SSG (Static Site Generation)**

**How it Works:**

1. Pages are pre-rendered at **build time**
2. Static HTML files are generated
3. Files are deployed to CDN
4. Browser receives pre-rendered HTML
5. React hydrates for interactivity

**Next.js SSG Example:**

```typescript
// pages/posts/[id].tsx
import { GetStaticProps, GetStaticPaths } from 'next';

interface Post {
  id: string;
  title: string;
  content: string;
}

interface Props {
  post: Post;
}

export default function PostPage({ post }: Props) {
  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </div>
  );
}

// Generate static paths at build time
export const getStaticPaths: GetStaticPaths = async () => {
  const response = await fetch('https://api.example.com/posts');
  const posts = await response.json();

  const paths = posts.map((post: Post) => ({
    params: { id: post.id },
  }));

  return {
    paths,
    fallback: 'blocking', // or false or true
  };
};

// Generate static props at build time
export const getStaticProps: GetStaticProps<Props> = async (context) => {
  const { id } = context.params!;
  
  const response = await fetch(`https://api.example.com/posts/${id}`);
  const post = await response.json();

  return {
    props: { post },
    revalidate: 3600, // ISR: Regenerate every hour
  };
};
```

**Build Output:**

```bash
# Build generates static HTML files
.next/
  static/
    pages/
      posts/
        1.html
        2.html
        3.html
```

**Deployment:**

```bash
# Build static pages
npm run build

# Export static files (if needed)
npm run export

# Deploy to CDN/static hosting
# Hosting: Vercel, Netlify, S3, CloudFront
```

**Characteristics:**
- ✅ Fastest load times (pre-rendered HTML)
- ✅ Best SEO (full HTML at build)
- ✅ Cheapest hosting (static files)
- ✅ Scalable (CDN distribution)
- ❌ Build time increases with pages
- ❌ Content not real-time
- ❌ Requires rebuild for updates

**Use Cases:**
- Blogs
- Documentation sites
- Marketing pages
- Portfolio sites
- Content-heavy sites with infrequent updates

---

#### **4. ISR (Incremental Static Regeneration)**

**Hybrid approach combining SSG + SSR:**

```typescript
export const getStaticProps: GetStaticProps = async () => {
  const data = await fetchData();

  return {
    props: { data },
    revalidate: 60, // Regenerate page every 60 seconds
  };
};
```

**How ISR Works:**

1. Page is statically generated at build time
2. Served from cache (fast)
3. After `revalidate` time, next request triggers regeneration
4. Old page served while new one generates
5. New page cached and served to subsequent requests

**Benefits:**
- Static performance with dynamic content
- No full rebuild required
- Automatic updates

---

#### **5. Comparison Table**

| Feature | CSR | SSR | SSG |
|---------|-----|-----|-----|
| **Initial Load** | Slow ❌ | Fast ✅ | Fastest ✅ |
| **SEO** | Poor ❌ | Excellent ✅ | Excellent ✅ |
| **Hosting** | Static (cheap) ✅ | Server (expensive) ❌ | Static (cheap) ✅ |
| **Deployment** | Simple ✅ | Complex ❌ | Simple ✅ |
| **Scalability** | High ✅ | Medium ⚠️ | Highest ✅ |
| **Real-time Data** | Yes ✅ | Yes ✅ | No ❌ |
| **Build Time** | Fast ✅ | N/A | Slow (many pages) ⚠️ |
| **Time to Interactive (TTI)** | Slow ❌ | Medium ⚠️ | Fast ✅ |
| **Server Required** | No ✅ | Yes ❌ | No ✅ |
| **Cost** | Low ✅ | High ❌ | Low ✅ |

---

#### **6. Deployment Architecture Examples**

**CSR Deployment:**

```
User → CDN (CloudFront) → S3 (static files)
     ↓
     API Server (data)
```

**SSR Deployment:**

```
User → Load Balancer → Node.js Servers (render on each request)
                     ↓
                     Database/API
```

**SSG Deployment:**

```
Build Time: CI/CD → Generate static pages → Deploy to CDN
Runtime: User → CDN (pre-rendered HTML)
```

---

#### **7. Hybrid Strategy**

**Next.js allows mixing strategies:**

```typescript
// pages/index.tsx - SSG for homepage
export const getStaticProps = async () => {
  return { props: { /*...*/ } };
};

// pages/dashboard.tsx - CSR for authenticated page
export default function Dashboard() {
  const { data } = useSWR('/api/user', fetcher);
  return <div>{/*...*/}</div>;
};

// pages/products/[id].tsx - SSR for dynamic product pages
export const getServerSideProps = async () => {
  return { props: { /*...*/ } };
};

// pages/blog/[slug].tsx - SSG with ISR for blog posts
export const getStaticProps = async () => {
  return { 
    props: { /*...*/ },
    revalidate: 3600 // ISR
  };
};
```

---

#### **8. Deployment Configurations**

**Vercel (All strategies):**

```json
// vercel.json
{
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
  "crons": [
    {
      "path": "/api/revalidate",
      "schedule": "0 */4 * * *"
    }
  ]
}
```

**Nginx for SSR:**

```nginx
server {
  listen 80;
  server_name example.com;

  location / {
    proxy_pass http://localhost:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }

  # Cache static assets
  location /_next/static {
    proxy_pass http://localhost:3000;
    add_header Cache-Control "public, max-age=31536000, immutable";
  }
}
```

---

#### **Decision Matrix**

**Choose CSR when:**
- Building internal tools/dashboards
- App is behind authentication
- SEO not important
- Frequent data updates
- Limited budget

**Choose SSR when:**
- SEO is critical
- Content changes frequently
- Personalized content per user
- Real-time data needed
- E-commerce sites

**Choose SSG when:**
- Content doesn't change often
- SEO is important
- Blog or documentation site
- Marketing pages
- Maximum performance needed

**Choose ISR when:**
- Need SSG benefits with semi-frequent updates
- Large site with many pages
- Content updates predictably
- Want automatic cache invalidation

---

#### **Best Practices**

1. **Hybrid Approach**: Mix strategies per page
2. **Performance**: Measure Core Web Vitals
3. **Caching**: Implement aggressive caching for static assets
4. **CDN**: Use CDN for all strategies
5. **Monitoring**: Track rendering performance
6. **Fallback**: Implement loading states
7. **Error Handling**: Handle server/build errors
8. **Testing**: Test each rendering strategy

---

#### **Summary**

**Rendering Strategies:**

1. **CSR**: Client renders everything (traditional React)
2. **SSR**: Server renders on each request (Next.js)
3. **SSG**: Pre-render at build time (Next.js, Gatsby)
4. **ISR**: SSG with automatic regeneration (Next.js)

**Key Differences:**

| Aspect | CSR | SSR | SSG |
|--------|-----|-----|-----|
| **When Rendered** | Runtime (browser) | Runtime (server) | Build time |
| **HTML Content** | Empty | Full | Full |
| **Data Freshness** | Real-time | Real-time | Build-time |
| **Server Need** | No | Yes | No |

**Modern Approach:**
- Use **SSG** for static content
- Use **SSR** for dynamic content
- Use **CSR** for authenticated sections
- Use **ISR** for semi-static content

Choosing the right strategy impacts **performance**, **SEO**, **cost**, and **user experience**.

</details>

---

### 255. How do you optimize React apps for CDN delivery?

<details>
<summary>View Answer</summary>

**CDN (Content Delivery Network)** optimization improves **loading speed**, **reduces latency**, and **scales** React applications globally by serving assets from edge locations close to users.

---

#### **1. Asset Fingerprinting / Content Hashing**

**Vite Configuration:**

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        // Generate hashed filenames
        entryFileNames: 'assets/[name].[hash].js',
        chunkFileNames: 'assets/[name].[hash].js',
        assetFileNames: 'assets/[name].[hash].[ext]',
      },
    },
    // Enable source maps for production debugging
    sourcemap: false,
    // Set chunk size warning limit
    chunkSizeWarningLimit: 1000,
  },
});
```

**Output:**

```
dist/
  index.html
  assets/
    index.a3f2c1b9.js
    vendor.8d4e2f19.js
    styles.f6b3a8c4.css
    logo.7c9d4b2e.svg
```

---

#### **2. Cache Headers Configuration**

**Nginx Configuration:**

```nginx
server {
  listen 80;
  server_name example.com;
  root /var/www/html;

  # HTML files - No cache (always check for updates)
  location ~* \.html$ {
    add_header Cache-Control "no-cache, no-store, must-revalidate";
    add_header Pragma "no-cache";
    add_header Expires "0";
  }

  # JavaScript and CSS with hash - Long-term cache
  location ~* ^/assets/.*\.(js|css)$ {
    add_header Cache-Control "public, max-age=31536000, immutable";
    access_log off;
  }

  # Images with hash - Long-term cache
  location ~* ^/assets/.*\.(jpg|jpeg|png|gif|ico|svg|webp|avif)$ {
    add_header Cache-Control "public, max-age=31536000, immutable";
    access_log off;
  }

  # Fonts with hash - Long-term cache
  location ~* ^/assets/.*\.(woff|woff2|ttf|otf|eot)$ {
    add_header Cache-Control "public, max-age=31536000, immutable";
    add_header Access-Control-Allow-Origin "*";
    access_log off;
  }

  # Service worker - No cache
  location = /service-worker.js {
    add_header Cache-Control "no-cache, no-store, must-revalidate";
  }

  # Enable gzip compression
  gzip on;
  gzip_vary on;
  gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript image/svg+xml;
  gzip_min_length 1000;
}
```

**Vercel Configuration:**

```json
// vercel.json
{
  "headers": [
    {
      "source": "/assets/(.*)",
      "headers": [
        {
          "key": "Cache-Control",
          "value": "public, max-age=31536000, immutable"
        }
      ]
    },
    {
      "source": "/(.*).html",
      "headers": [
        {
          "key": "Cache-Control",
          "value": "no-cache, no-store, must-revalidate"
        }
      ]
    },
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "X-Content-Type-Options",
          "value": "nosniff"
        },
        {
          "key": "X-Frame-Options",
          "value": "DENY"
        },
        {
          "key": "X-XSS-Protection",
          "value": "1; mode=block"
        }
      ]
    }
  ]
}
```

---

#### **3. Code Splitting for Optimal Caching**

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          // Vendor chunk for stable dependencies
          vendor: ['react', 'react-dom', 'react-router-dom'],
          // UI library chunk
          ui: ['@mui/material', '@emotion/react', '@emotion/styled'],
          // Charts chunk
          charts: ['recharts', 'd3'],
        },
      },
    },
  },
});
```

**Automatic Code Splitting:**

```typescript
// src/App.tsx
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';

// Lazy load routes
const Home = lazy(() => import('./pages/Home'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Profile = lazy(() => import('./pages/Profile'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/profile" element={<Profile />} />
      </Routes>
    </Suspense>
  );
}
```

---

#### **4. Asset Compression**

**Brotli and Gzip:**

```typescript
// vite.config.ts
import viteCompression from 'vite-plugin-compression';

export default defineConfig({
  plugins: [
    react(),
    // Gzip compression
    viteCompression({
      algorithm: 'gzip',
      ext: '.gz',
      threshold: 1024, // Only compress files > 1KB
    }),
    // Brotli compression (better compression)
    viteCompression({
      algorithm: 'brotliCompress',
      ext: '.br',
      threshold: 1024,
    }),
  ],
});
```

**Output:**

```
dist/assets/
  index.a3f2c1b9.js
  index.a3f2c1b9.js.gz   (gzip compressed)
  index.a3f2c1b9.js.br   (brotli compressed)
```

---

#### **5. AWS CloudFront Configuration**

**CloudFront Distribution:**

```typescript
// infrastructure/cloudfront.ts (CDK)
import * as cloudfront from 'aws-cdk-lib/aws-cloudfront';
import * as origins from 'aws-cdk-lib/aws-cloudfront-origins';
import * as s3 from 'aws-cdk-lib/aws-s3';

const bucket = new s3.Bucket(this, 'WebsiteBucket');

const distribution = new cloudfront.Distribution(this, 'Distribution', {
  defaultBehavior: {
    origin: new origins.S3Origin(bucket),
    viewerProtocolPolicy: cloudfront.ViewerProtocolPolicy.REDIRECT_TO_HTTPS,
    allowedMethods: cloudfront.AllowedMethods.ALLOW_GET_HEAD_OPTIONS,
    cachedMethods: cloudfront.CachedMethods.CACHE_GET_HEAD_OPTIONS,
    compress: true, // Enable automatic compression
    cachePolicy: new cloudfront.CachePolicy(this, 'CachePolicy', {
      cachePolicyName: 'ReactAppCache',
      minTtl: Duration.seconds(0),
      maxTtl: Duration.days(365),
      defaultTtl: Duration.days(1),
      enableAcceptEncodingGzip: true,
      enableAcceptEncodingBrotli: true,
      headerBehavior: cloudfront.CacheHeaderBehavior.none(),
      queryStringBehavior: cloudfront.CacheQueryStringBehavior.none(),
    }),
  },
  additionalBehaviors: {
    // Cache static assets aggressively
    '/assets/*': {
      origin: new origins.S3Origin(bucket),
      viewerProtocolPolicy: cloudfront.ViewerProtocolPolicy.REDIRECT_TO_HTTPS,
      compress: true,
      cachePolicy: cloudfront.CachePolicy.CACHING_OPTIMIZED,
    },
  },
  errorResponses: [
    {
      httpStatus: 404,
      responseHttpStatus: 200,
      responsePagePath: '/index.html',
      ttl: Duration.minutes(5),
    },
  ],
});
```

**Terraform Configuration:**

```hcl
# cloudfront.tf
resource "aws_cloudfront_distribution" "react_app" {
  enabled             = true
  default_root_object = "index.html"
  price_class         = "PriceClass_100"

  origin {
    domain_name = aws_s3_bucket.website.bucket_regional_domain_name
    origin_id   = "S3-${aws_s3_bucket.website.id}"

    s3_origin_config {
      origin_access_identity = aws_cloudfront_origin_access_identity.oai.cloudfront_access_identity_path
    }
  }

  default_cache_behavior {
    allowed_methods  = ["GET", "HEAD", "OPTIONS"]
    cached_methods   = ["GET", "HEAD"]
    target_origin_id = "S3-${aws_s3_bucket.website.id}"

    forwarded_values {
      query_string = false
      cookies {
        forward = "none"
      }
    }

    viewer_protocol_policy = "redirect-to-https"
    min_ttl                = 0
    default_ttl            = 86400
    max_ttl                = 31536000
    compress               = true
  }

  ordered_cache_behavior {
    path_pattern     = "/assets/*"
    allowed_methods  = ["GET", "HEAD"]
    cached_methods   = ["GET", "HEAD"]
    target_origin_id = "S3-${aws_s3_bucket.website.id}"

    forwarded_values {
      query_string = false
      cookies {
        forward = "none"
      }
    }

    min_ttl                = 31536000
    default_ttl            = 31536000
    max_ttl                = 31536000
    compress               = true
    viewer_protocol_policy = "redirect-to-https"
  }

  custom_error_response {
    error_code         = 404
    response_code      = 200
    response_page_path = "/index.html"
  }

  restrictions {
    geo_restriction {
      restriction_type = "none"
    }
  }

  viewer_certificate {
    acm_certificate_arn = aws_acm_certificate.cert.arn
    ssl_support_method  = "sni-only"
  }
}
```

---

#### **6. Resource Hints**

**Preload Critical Assets:**

```html
<!-- dist/index.html -->
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>My App</title>
    
    <!-- Preload critical CSS -->
    <link rel="preload" href="/assets/main.abc123.css" as="style">
    
    <!-- Preload critical JS -->
    <link rel="preload" href="/assets/main.def456.js" as="script">
    
    <!-- Preload critical font -->
    <link rel="preload" href="/assets/fonts/inter.woff2" as="font" type="font/woff2" crossorigin>
    
    <!-- DNS prefetch for API -->
    <link rel="dns-prefetch" href="https://api.example.com">
    
    <!-- Preconnect to API -->
    <link rel="preconnect" href="https://api.example.com">
    
    <!-- Preconnect to analytics -->
    <link rel="preconnect" href="https://www.google-analytics.com">
    
    <link rel="stylesheet" href="/assets/main.abc123.css">
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/assets/main.def456.js"></script>
  </body>
</html>
```

**Dynamic Preloading:**

```typescript
// src/utils/preload.ts
export function preloadRoute(route: string) {
  const link = document.createElement('link');
  link.rel = 'prefetch';
  link.href = route;
  document.head.appendChild(link);
}

// Usage in component
function Navigation() {
  return (
    <nav>
      <Link 
        to="/dashboard" 
        onMouseEnter={() => preloadRoute('/dashboard')}
      >
        Dashboard
      </Link>
    </nav>
  );
}
```

---

#### **7. Image Optimization**

**Modern Image Formats:**

```typescript
// vite.config.ts
import imagemin from 'vite-plugin-imagemin';

export default defineConfig({
  plugins: [
    react(),
    imagemin({
      gifsicle: { optimizationLevel: 7 },
      optipng: { optimizationLevel: 7 },
      mozjpeg: { quality: 80 },
      pngquant: { quality: [0.8, 0.9], speed: 4 },
      svgo: {
        plugins: [
          { name: 'removeViewBox', active: false },
          { name: 'removeEmptyAttrs', active: true },
        ],
      },
      webp: { quality: 80 },
      avif: { quality: 80 },
    }),
  ],
});
```

**Responsive Images Component:**

```typescript
interface OptimizedImageProps {
  src: string;
  alt: string;
  sizes?: string;
}

function OptimizedImage({ src, alt, sizes }: OptimizedImageProps) {
  const basePath = src.replace(/\.[^.]+$/, '');
  
  return (
    <picture>
      <source
        type="image/avif"
        srcSet={`
          ${basePath}-320w.avif 320w,
          ${basePath}-640w.avif 640w,
          ${basePath}-1024w.avif 1024w
        `}
        sizes={sizes || "100vw"}
      />
      <source
        type="image/webp"
        srcSet={`
          ${basePath}-320w.webp 320w,
          ${basePath}-640w.webp 640w,
          ${basePath}-1024w.webp 1024w
        `}
        sizes={sizes || "100vw"}
      />
      <img
        src={`${basePath}.jpg`}
        alt={alt}
        loading="lazy"
        decoding="async"
      />
    </picture>
  );
}
```

---

#### **8. CDN Cache Invalidation**

**CloudFront Invalidation Script:**

```bash
#!/bin/bash
# scripts/invalidate-cdn.sh

DISTRIBUTION_ID="E1234567890ABC"

# Invalidate all files
aws cloudfront create-invalidation \
  --distribution-id $DISTRIBUTION_ID \
  --paths "/*"

# Or invalidate specific paths
aws cloudfront create-invalidation \
  --distribution-id $DISTRIBUTION_ID \
  --paths "/index.html" "/service-worker.js"
```

**GitHub Actions Deployment with Invalidation:**

```yaml
# .github/workflows/deploy-cdn.yml
name: Deploy to CDN

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Build application
        run: |
          npm ci
          npm run build
      
      - name: Deploy to S3
        run: |
          aws s3 sync dist/ s3://my-bucket --delete
      
      - name: Invalidate CloudFront cache
        run: |
          aws cloudfront create-invalidation \
            --distribution-id ${{ secrets.CLOUDFRONT_ID }} \
            --paths "/*"
```

---

#### **9. Service Worker for Offline Support**

```typescript
// src/service-worker.ts
const CACHE_NAME = 'v1';
const ASSETS_TO_CACHE = [
  '/',
  '/index.html',
  '/assets/main.abc123.js',
  '/assets/main.def456.css',
];

self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => {
      return cache.addAll(ASSETS_TO_CACHE);
    })
  );
});

self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then((response) => {
      // Cache hit - return from cache
      if (response) {
        return response;
      }
      
      // Clone the request
      const fetchRequest = event.request.clone();
      
      return fetch(fetchRequest).then((response) => {
        // Check if valid response
        if (!response || response.status !== 200 || response.type !== 'basic') {
          return response;
        }
        
        // Clone response
        const responseToCache = response.clone();
        
        caches.open(CACHE_NAME).then((cache) => {
          cache.put(event.request, responseToCache);
        });
        
        return response;
      });
    })
  );
});
```

---

#### **Best Practices**

1. **Immutable Assets**: Use content hashing for cache busting
2. **Long TTL**: Set max-age=31536000 for hashed assets
3. **No Cache HTML**: Always fresh HTML to load latest assets
4. **Compression**: Enable Brotli and Gzip
5. **Code Splitting**: Separate vendor and app code
6. **Image Optimization**: Use modern formats (WebP, AVIF)
7. **Resource Hints**: Preload/prefetch critical resources
8. **HTTP/2**: Enable multiplexing and server push
9. **CDN Edge Locations**: Use CDN with global coverage
10. **Cache Invalidation**: Invalidate on deployment

---

#### **Performance Checklist**

```typescript
// Deployment checklist
const CDN_OPTIMIZATION_CHECKLIST = [
  '✅ Content hashing enabled',
  '✅ Cache headers configured',
  '✅ Gzip/Brotli compression enabled',
  '✅ Code splitting implemented',
  '✅ Images optimized (WebP/AVIF)',
  '✅ CDN configured with edge locations',
  '✅ HTTPS enabled',
  '✅ HTTP/2 enabled',
  '✅ Resource hints added',
  '✅ Service worker for offline support',
  '✅ Cache invalidation automated',
  '✅ Performance monitoring setup',
];
```

---

#### **Summary**

**CDN Optimization Strategies:**

1. **Asset Fingerprinting**: Content-based hashing for cache busting
2. **Cache Headers**: Long TTL for assets, no-cache for HTML
3. **Compression**: Brotli + Gzip for smaller transfers
4. **Code Splitting**: Separate vendor, UI, and app code
5. **Image Optimization**: Modern formats and lazy loading
6. **Resource Hints**: Preload, prefetch, preconnect
7. **CDN Configuration**: CloudFront, Cloudflare, etc.
8. **Cache Invalidation**: Automated on deployment
9. **Service Worker**: Offline support and caching

**Key Metrics to Monitor:**
- Time to First Byte (TTFB)
- First Contentful Paint (FCP)
- Largest Contentful Paint (LCP)
- Cache Hit Ratio
- CDN bandwidth usage

**Benefits:**
- Faster page loads globally
- Reduced server load
- Better SEO rankings
- Improved user experience
- Lower bandwidth costs

Proper CDN optimization ensures **fast**, **reliable**, and **scalable** React application delivery worldwide.

</details>

---

### 256. What is edge rendering and when to use it?

<details>
<summary>View Answer</summary>

**Edge rendering** executes code at **CDN edge locations** close to users, reducing latency by avoiding round trips to origin servers. It's ideal for **dynamic content**, **personalization**, and **A/B testing**.

---

#### **1. What is Edge Computing?**

**Traditional Architecture:**

```
User (Tokyo) → CDN (Tokyo) → Origin Server (US) → Database (US)
                                   ^
                                   |
                      High latency (~200ms)
```

**Edge Architecture:**

```
User (Tokyo) → Edge (Tokyo) → Data (nearby)
                   ^
                   |
         Low latency (~10ms)
```

**Benefits:**
- ✅ Reduced latency (10-50ms vs 100-300ms)
- ✅ Better performance globally
- ✅ Lower load on origin servers
- ✅ Scalability at the edge

---

#### **2. Cloudflare Workers**

**Basic Edge Function:**

```typescript
// worker.ts
export default {
  async fetch(request: Request): Promise<Response> {
    const url = new URL(request.url);
    
    // Personalize content based on location
    const country = request.cf?.country || 'US';
    
    // Modify HTML at the edge
    const response = await fetch(request);
    const html = await response.text();
    
    const modifiedHtml = html.replace(
      '<div id="root"></div>',
      `<div id="root" data-country="${country}"></div>`
    );
    
    return new Response(modifiedHtml, {
      headers: {
        'Content-Type': 'text/html',
        'Cache-Control': 'public, max-age=3600',
      },
    });
  },
};
```

**Edge A/B Testing:**

```typescript
// workers/ab-test.ts
export default {
  async fetch(request: Request): Promise<Response> {
    const url = new URL(request.url);
    
    // Get or create user variant
    const cookie = request.headers.get('Cookie') || '';
    let variant = cookie.match(/variant=([AB])/)?.[1];
    
    if (!variant) {
      // Assign random variant
      variant = Math.random() < 0.5 ? 'A' : 'B';
    }
    
    // Fetch appropriate version
    const response = await fetch(
      `https://origin.example.com/${variant}/index.html`
    );
    
    // Set variant cookie
    const modifiedResponse = new Response(response.body, response);
    modifiedResponse.headers.set(
      'Set-Cookie',
      `variant=${variant}; Path=/; Max-Age=86400`
    );
    
    return modifiedResponse;
  },
};
```

**Edge API Routes:**

```typescript
// workers/api.ts
import { Router } from 'itty-router';

const router = Router();

// Edge API endpoint
router.get('/api/user/:id', async (request) => {
  const { id } = request.params;
  
  // Access edge KV storage
  const user = await USER_KV.get(id, 'json');
  
  if (!user) {
    return new Response('Not found', { status: 404 });
  }
  
  return new Response(JSON.stringify(user), {
    headers: { 'Content-Type': 'application/json' },
  });
});

// Personalized content
router.get('/', async (request) => {
  const country = request.cf?.country;
  const city = request.cf?.city;
  
  // Fetch localized content
  const content = await fetch(
    `https://api.example.com/content?country=${country}`
  );
  
  return content;
});

export default {
  fetch: router.handle,
};
```

---

#### **3. Vercel Edge Functions**

**Edge Middleware:**

```typescript
// middleware.ts
import { NextRequest, NextResponse } from 'next/server';

export function middleware(request: NextRequest) {
  const { geo, ip } = request;
  
  // Add geolocation data to request
  const requestHeaders = new Headers(request.headers);
  requestHeaders.set('x-user-country', geo?.country || 'US');
  requestHeaders.set('x-user-city', geo?.city || 'Unknown');
  requestHeaders.set('x-user-ip', ip || 'Unknown');
  
  // Personalize based on location
  if (geo?.country === 'CN') {
    return NextResponse.rewrite(new URL('/cn', request.url));
  }
  
  return NextResponse.next({
    request: {
      headers: requestHeaders,
    },
  });
}

export const config = {
  matcher: '/:path*',
};
```

**Edge API Route:**

```typescript
// pages/api/edge.ts
import { NextRequest } from 'next/server';

export const config = {
  runtime: 'edge',
};

export default async function handler(req: NextRequest) {
  const { geo } = req;
  
  // Get user's location
  const location = {
    country: geo?.country,
    city: geo?.city,
    region: geo?.region,
    latitude: geo?.latitude,
    longitude: geo?.longitude,
  };
  
  return new Response(JSON.stringify(location), {
    headers: {
      'Content-Type': 'application/json',
      'Cache-Control': 'public, max-age=60',
    },
  });
}
```

**Edge-Rendered Page:**

```typescript
// pages/products/[id].tsx
import { NextRequest } from 'next/server';

export const config = {
  runtime: 'edge',
};

export default function Product({ product, userCountry }) {
  return (
    <div>
      <h1>{product.name}</h1>
      <p>
        Price: {product.prices[userCountry]} {getCurrency(userCountry)}
      </p>
    </div>
  );
}

export async function getServerSideProps(context: any) {
  const { id } = context.params;
  const userCountry = context.req.headers['x-user-country'] || 'US';
  
  // Fetch product data
  const response = await fetch(`https://api.example.com/products/${id}`);
  const product = await response.json();
  
  return {
    props: {
      product,
      userCountry,
    },
  };
}
```

---

#### **4. Edge-Side Includes (ESI)**

**HTML with ESI Tags:**

```html
<!DOCTYPE html>
<html>
  <head>
    <title>My App</title>
  </head>
  <body>
    <!-- Static content cached at edge -->
    <header>
      <h1>Welcome</h1>
    </header>
    
    <!-- Dynamic content rendered at edge -->
    <!--esi <esi:include src="/fragments/user-banner" /> -->
    
    <main>
      <!-- Static content -->
      <p>This is cached content</p>
      
      <!-- Personalized content -->
      <!--esi <esi:include src="/fragments/recommendations" /> -->
    </main>
  </body>
</html>
```

**Edge Worker with ESI:**

```typescript
export default {
  async fetch(request: Request): Promise<Response> {
    // Fetch base HTML
    const response = await fetch(request);
    const html = await response.text();
    
    // Process ESI tags
    const processedHtml = await processESI(html, request);
    
    return new Response(processedHtml, {
      headers: response.headers,
    });
  },
};

async function processESI(html: string, request: Request): Promise<string> {
  const esiRegex = /<!--esi <esi:include src="([^"]+)" \/> -->/g;
  
  let match;
  let result = html;
  
  while ((match = esiRegex.exec(html)) !== null) {
    const [fullMatch, src] = match;
    
    // Fetch fragment
    const fragmentResponse = await fetch(
      new URL(src, request.url).toString()
    );
    const fragment = await fragmentResponse.text();
    
    // Replace ESI tag with content
    result = result.replace(fullMatch, fragment);
  }
  
  return result;
}
```

---

#### **5. Edge Caching with Personalization**

```typescript
// workers/cache-personalized.ts
export default {
  async fetch(request: Request, env: any): Promise<Response> {
    const url = new URL(request.url);
    const country = request.cf?.country || 'US';
    
    // Create cache key with country
    const cacheKey = new Request(`${url.toString()}?country=${country}`, request);
    
    // Check cache
    const cache = caches.default;
    let response = await cache.match(cacheKey);
    
    if (!response) {
      // Fetch and personalize
      response = await fetch(request);
      const html = await response.text();
      
      // Add country-specific content
      const personalizedHtml = html.replace(
        '{{COUNTRY}}',
        country
      );
      
      response = new Response(personalizedHtml, response);
      
      // Cache personalized response
      response.headers.set('Cache-Control', 'public, max-age=3600');
      await cache.put(cacheKey, response.clone());
    }
    
    return response;
  },
};
```

---

#### **6. Streaming SSR at the Edge**

```typescript
// pages/stream.tsx
import { Suspense } from 'react';
import { renderToReadableStream } from 'react-dom/server';

export const config = {
  runtime: 'edge',
};

export default async function handler(req: NextRequest) {
  const stream = await renderToReadableStream(
    <html>
      <body>
        <h1>Streaming Content</h1>
        <Suspense fallback={<div>Loading...</div>}>
          <AsyncContent />
        </Suspense>
      </body>
    </html>,
    {
      onError(error) {
        console.error(error);
      },
    }
  );
  
  return new Response(stream, {
    headers: {
      'Content-Type': 'text/html',
      'Transfer-Encoding': 'chunked',
    },
  });
}

async function AsyncContent() {
  const data = await fetchData();
  return <div>{data}</div>;
}
```

---

#### **7. Use Cases for Edge Rendering**

**1. Geolocation-based Redirects:**

```typescript
export default {
  async fetch(request: Request): Promise<Response> {
    const country = request.cf?.country;
    
    // Redirect to country-specific site
    const countryDomains = {
      'US': 'example.com',
      'UK': 'example.co.uk',
      'DE': 'example.de',
    };
    
    const domain = countryDomains[country] || 'example.com';
    
    if (!request.url.includes(domain)) {
      return Response.redirect(`https://${domain}`, 302);
    }
    
    return fetch(request);
  },
};
```

**2. Bot Detection and Rendering:**

```typescript
export function middleware(request: NextRequest) {
  const userAgent = request.headers.get('user-agent') || '';
  
  // Detect bots
  const isBot = /googlebot|bingbot|slurp|duckduckbot/i.test(userAgent);
  
  if (isBot) {
    // Serve pre-rendered HTML for bots
    return NextResponse.rewrite(
      new URL('/prerendered' + request.nextUrl.pathname, request.url)
    );
  }
  
  return NextResponse.next();
}
```

**3. Feature Flags at Edge:**

```typescript
export default {
  async fetch(request: Request, env: any): Promise<Response> {
    const userId = request.headers.get('x-user-id');
    
    // Check feature flag from edge KV
    const featureEnabled = await env.FEATURES.get(`new_ui:${userId}`);
    
    if (featureEnabled === 'true') {
      return fetch(`https://origin.example.com/new-ui${new URL(request.url).pathname}`);
    }
    
    return fetch(request);
  },
};
```

**4. Image Optimization:**

```typescript
export default {
  async fetch(request: Request): Promise<Response> {
    const url = new URL(request.url);
    
    // Extract image parameters
    const width = url.searchParams.get('w') || '800';
    const format = url.searchParams.get('f') || 'webp';
    
    // Fetch and transform image at edge
    const imageRequest = new Request(url.pathname);
    const response = await fetch(imageRequest);
    
    // Transform with Cloudflare Image Resizing
    return fetch(imageRequest, {
      cf: {
        image: {
          width: parseInt(width),
          format,
          quality: 85,
        },
      },
    });
  },
};
```

---

#### **8. When to Use Edge Rendering**

**✅ Good Use Cases:**

1. **Personalization**: Show content based on location, device, etc.
2. **A/B Testing**: Route users to variants at the edge
3. **Authentication**: Handle auth checks before hitting origin
4. **Redirects**: Smart routing based on user attributes
5. **Bot Detection**: Serve different content to crawlers
6. **Rate Limiting**: Protect APIs at the edge
7. **Content Transformation**: Modify HTML/images on the fly
8. **Geofencing**: Restrict access by location

**❌ Not Ideal For:**

1. **Heavy Computation**: Limited CPU time at edge
2. **Large Dependencies**: Bundle size restrictions
3. **Database Access**: No direct DB connections
4. **Long-running Tasks**: Execution time limits
5. **File System**: No local file system access

---

#### **Performance Comparison**

```typescript
// Traditional SSR (Origin Server)
Time to First Byte: 200-500ms
Server Processing: 50-100ms
Network Latency: 150-400ms

// Edge SSR
Time to First Byte: 50-150ms
Edge Processing: 10-50ms
Network Latency: 10-50ms

// Improvement: 60-70% faster TTFB
```

---

#### **Best Practices**

1. **Keep Edge Functions Small**: Minimize dependencies
2. **Cache Aggressively**: Use edge caching when possible
3. **Fail Gracefully**: Handle errors, fallback to origin
4. **Monitor Performance**: Track edge function execution time
5. **Use KV Storage**: For user preferences, flags, etc.
6. **Avoid Heavy Computation**: Offload to origin if needed
7. **Test Edge Functions**: Test locally before deployment
8. **Version Control**: Use Git for edge function code

---

#### **Summary**

**Edge Rendering Platforms:**

1. **Cloudflare Workers**: Global edge network, KV storage
2. **Vercel Edge Functions**: Integrated with Next.js
3. **Deno Deploy**: Edge runtime for Deno
4. **AWS Lambda@Edge**: CloudFront edge functions
5. **Fastly Compute@Edge**: WebAssembly at edge

**Key Features:**
- Execute code at edge locations
- Reduce latency (10-50ms vs 100-300ms)
- Personalize content dynamically
- A/B testing without origin load
- Global scalability

**Architecture:**
```
User → Edge (render/process) → [Optional: Origin]
      10-50ms              0-100ms
```

**Benefits:**
- ✅ Lower latency globally
- ✅ Better user experience
- ✅ Reduced origin load
- ✅ Dynamic personalization
- ✅ Cost-effective scaling

**Trade-offs:**
- ❌ Limited execution time
- ❌ Smaller bundle sizes
- ❌ No direct database access
- ❌ Limited CPU/memory

Edge rendering is the **future of web performance**, bringing computation **closer to users** for **instant responses**.

</details>

---

### 257. How do you implement blue-green deployments?

<details>
<summary>View Answer</summary>

**Blue-green deployment** is a release strategy that maintains two identical production environments ("blue" and "green"). Traffic is switched from one to the other during deployment, enabling **zero-downtime releases** and **instant rollback**.

---

#### **1. Blue-Green Deployment Concept**

**Architecture:**

```
Load Balancer
     |
     |--- Blue Environment (v1.0 - currently serving traffic)
     |
     |--- Green Environment (v1.1 - new version, idle)

1. Deploy new version to Green
2. Test Green environment
3. Switch traffic from Blue to Green
4. Blue becomes idle (rollback target)
```

**Benefits:**
- ✅ Zero downtime deployments
- ✅ Instant rollback (switch back to Blue)
- ✅ Full environment testing before cutover
- ✅ Reduced deployment risk

---

#### **2. AWS Elastic Beanstalk Blue-Green**

**Environment Setup:**

```bash
# Create blue environment
eb create production-blue \
  --instance-type t3.medium \
  --envvars NODE_ENV=production,VERSION=1.0.0

# Create green environment
eb create production-green \
  --instance-type t3.medium \
  --envvars NODE_ENV=production,VERSION=1.1.0
```

**DNS Swap:**

```bash
# Deploy new version to green
eb deploy production-green

# Test green environment
curl https://production-green.example.com

# Swap URLs (switches traffic)
eb swap production-blue --destination production-green

# Now:
# - production-blue.example.com points to v1.1
# - production-green.example.com points to v1.0 (rollback ready)
```

**Automated Script:**

```bash
#!/bin/bash
# scripts/blue-green-deploy.sh

BLUE_ENV="production-blue"
GREEN_ENV="production-green"
NEW_VERSION=$1

echo "Deploying version ${NEW_VERSION} to green environment..."
eb deploy $GREEN_ENV --label $NEW_VERSION

echo "Running smoke tests..."
curl -f https://${GREEN_ENV}.example.com/health || exit 1

echo "Swapping environments..."
eb swap $BLUE_ENV --destination $GREEN_ENV

echo "Deployment complete! Blue is now idle (rollback ready)"
```

---

#### **3. AWS Application Load Balancer (ALB)**

**Target Group Setup:**

```typescript
// infrastructure/alb.ts (CDK)
import * as elbv2 from 'aws-cdk-lib/aws-elasticloadbalancingv2';
import * as ec2 from 'aws-cdk-lib/aws-ec2';

// Create ALB
const alb = new elbv2.ApplicationLoadBalancer(this, 'ALB', {
  vpc,
  internetFacing: true,
});

// Blue target group
const blueTargetGroup = new elbv2.ApplicationTargetGroup(this, 'BlueTargetGroup', {
  vpc,
  port: 80,
  protocol: elbv2.ApplicationProtocol.HTTP,
  targets: [blueAutoScalingGroup],
  healthCheck: {
    path: '/health',
    interval: Duration.seconds(30),
  },
});

// Green target group
const greenTargetGroup = new elbv2.ApplicationTargetGroup(this, 'GreenTargetGroup', {
  vpc,
  port: 80,
  protocol: elbv2.ApplicationProtocol.HTTP,
  targets: [greenAutoScalingGroup],
  healthCheck: {
    path: '/health',
    interval: Duration.seconds(30),
  },
});

// Listener with blue as default
const listener = alb.addListener('Listener', {
  port: 80,
  defaultTargetGroups: [blueTargetGroup],
});

// Traffic switching function
export function switchTraffic(targetEnv: 'blue' | 'green') {
  const targetGroup = targetEnv === 'blue' ? blueTargetGroup : greenTargetGroup;
  listener.addTargetGroups('NewDefault', {
    targetGroups: [targetGroup],
  });
}
```

**Deployment Script:**

```bash
#!/bin/bash
# scripts/alb-blue-green.sh

ALB_ARN="arn:aws:elasticloadbalancing:..."
BLUE_TG_ARN="arn:aws:elasticloadbalancing:..."
GREEN_TG_ARN="arn:aws:elasticloadbalancing:..."

# Get current target group
CURRENT_TG=$(aws elbv2 describe-listeners \
  --load-balancer-arn $ALB_ARN \
  --query 'Listeners[0].DefaultActions[0].TargetGroupArn' \
  --output text)

# Determine target environment
if [ "$CURRENT_TG" = "$BLUE_TG_ARN" ]; then
  TARGET_ENV="green"
  TARGET_TG=$GREEN_TG_ARN
else
  TARGET_ENV="blue"
  TARGET_TG=$BLUE_TG_ARN
fi

echo "Deploying to ${TARGET_ENV} environment..."

# Deploy new version
aws deploy create-deployment \
  --application-name my-app \
  --deployment-group-name ${TARGET_ENV}-group \
  --s3-location bucket=my-bucket,key=app.zip,bundleType=zip

# Wait for deployment
aws deploy wait deployment-successful --deployment-id $DEPLOYMENT_ID

# Run health checks
for i in {1..10}; do
  STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://${TARGET_ENV}.internal/health)
  if [ $STATUS -eq 200 ]; then
    echo "Health check passed"
    break
  fi
  sleep 5
done

# Switch traffic
aws elbv2 modify-listener \
  --listener-arn $LISTENER_ARN \
  --default-actions Type=forward,TargetGroupArn=$TARGET_TG

echo "Traffic switched to ${TARGET_ENV}"
```

---

#### **4. Kubernetes Blue-Green Deployment**

**Blue Deployment:**

```yaml
# k8s/blue-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: react-app-blue
  labels:
    app: react-app
    version: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: react-app
      version: blue
  template:
    metadata:
      labels:
        app: react-app
        version: blue
    spec:
      containers:
      - name: react-app
        image: myregistry/react-app:v1.0.0
        ports:
        - containerPort: 80
        env:
        - name: VERSION
          value: "1.0.0"
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 5
```

**Green Deployment:**

```yaml
# k8s/green-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: react-app-green
  labels:
    app: react-app
    version: green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: react-app
      version: green
  template:
    metadata:
      labels:
        app: react-app
        version: green
    spec:
      containers:
      - name: react-app
        image: myregistry/react-app:v1.1.0
        ports:
        - containerPort: 80
        env:
        - name: VERSION
          value: "1.1.0"
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 5
```

**Service (Traffic Router):**

```yaml
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: react-app-service
spec:
  selector:
    app: react-app
    version: blue  # Initially routes to blue
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
```

**Deployment Script:**

```bash
#!/bin/bash
# scripts/k8s-blue-green.sh

NEW_VERSION=$1
CURRENT_ENV=$(kubectl get service react-app-service -o jsonpath='{.spec.selector.version}')

if [ "$CURRENT_ENV" = "blue" ]; then
  TARGET_ENV="green"
  IDLE_ENV="blue"
else
  TARGET_ENV="blue"
  IDLE_ENV="green"
fi

echo "Current: $CURRENT_ENV, Deploying to: $TARGET_ENV"

# Update target deployment with new image
kubectl set image deployment/react-app-${TARGET_ENV} \
  react-app=myregistry/react-app:${NEW_VERSION}

# Wait for rollout
kubectl rollout status deployment/react-app-${TARGET_ENV}

# Run smoke tests
kubectl run test-pod --rm -i --restart=Never \
  --image=curlimages/curl -- \
  curl -f http://react-app-${TARGET_ENV}/health

if [ $? -eq 0 ]; then
  echo "Health check passed. Switching traffic..."
  
  # Update service selector
  kubectl patch service react-app-service -p \
    '{"spec":{"selector":{"version":"'${TARGET_ENV}'"}}}'
  
  echo "Traffic switched to ${TARGET_ENV}"
  echo "${IDLE_ENV} is now idle (rollback ready)"
else
  echo "Health check failed. Keeping traffic on ${CURRENT_ENV}"
  exit 1
fi
```

---

#### **5. Vercel/Netlify Blue-Green**

**Vercel Deployment:**

```bash
# Deploy to preview (green)
vercel deploy --prod=false
# Output: https://my-app-abc123.vercel.app

# Test preview environment
curl https://my-app-abc123.vercel.app

# Promote to production (switch from blue to green)
vercel promote https://my-app-abc123.vercel.app
```

**GitHub Actions Workflow:**

```yaml
# .github/workflows/blue-green-vercel.yml
name: Blue-Green Deployment

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to Vercel (Green)
        id: deploy
        run: |
          npm i -g vercel
          DEPLOYMENT_URL=$(vercel deploy --token=${{ secrets.VERCEL_TOKEN }} --prod=false)
          echo "url=$DEPLOYMENT_URL" >> $GITHUB_OUTPUT
      
      - name: Run E2E tests on Green
        run: |
          npm run test:e2e
        env:
          BASE_URL: ${{ steps.deploy.outputs.url }}
      
      - name: Promote to Production (Blue → Green)
        if: success()
        run: |
          vercel promote ${{ steps.deploy.outputs.url }} \
            --token=${{ secrets.VERCEL_TOKEN }} \
            --scope=${{ secrets.VERCEL_SCOPE }}
      
      - name: Rollback on failure
        if: failure()
        run: |
          echo "Deployment failed. Green environment not promoted."
          vercel remove ${{ steps.deploy.outputs.url }} --yes
```

---

#### **6. Database Migrations**

**Backward-Compatible Migrations:**

```typescript
// migrations/001_add_column.ts
export async function up(db: Database) {
  // Add new column (backward compatible)
  await db.schema.alterTable('users', (table) => {
    table.string('phone').nullable(); // nullable for compatibility
  });
}

export async function down(db: Database) {
  await db.schema.alterTable('users', (table) => {
    table.dropColumn('phone');
  });
}
```

**Multi-Phase Migration Strategy:**

```typescript
// Phase 1: Add nullable column (deploy green)
await db.schema.alterTable('users', (table) => {
  table.string('new_field').nullable();
});

// Phase 2: Backfill data (both blue and green running)
await db('users').update({
  new_field: db.raw('old_field'),
});

// Phase 3: Make not-null (after full cutover)
await db.schema.alterTable('users', (table) => {
  table.string('new_field').notNullable().alter();
});

// Phase 4: Drop old column (next deployment)
await db.schema.alterTable('users', (table) => {
  table.dropColumn('old_field');
});
```

---

#### **7. Automated Blue-Green Pipeline**

```yaml
# .github/workflows/blue-green-pipeline.yml
name: Blue-Green Deployment Pipeline

on:
  push:
    branches: [main]

env:
  AWS_REGION: us-east-1
  APP_NAME: react-app

jobs:
  determine-environment:
    runs-on: ubuntu-latest
    outputs:
      current: ${{ steps.detect.outputs.current }}
      target: ${{ steps.detect.outputs.target }}
    
    steps:
      - name: Detect current environment
        id: detect
        run: |
          CURRENT=$(aws elbv2 describe-target-groups \
            --query 'TargetGroups[?contains(TargetGroupName, `blue`)].TargetGroupName' \
            --output text)
          
          if [ -z "$CURRENT" ]; then
            echo "current=blue" >> $GITHUB_OUTPUT
            echo "target=green" >> $GITHUB_OUTPUT
          else
            echo "current=green" >> $GITHUB_OUTPUT
            echo "target=blue" >> $GITHUB_OUTPUT
          fi

  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v3
        with:
          name: build
          path: dist/

  deploy-target:
    needs: [determine-environment, build]
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/download-artifact@v3
        with:
          name: build
          path: dist/
      
      - name: Deploy to ${{ needs.determine-environment.outputs.target }}
        run: |
          aws deploy create-deployment \
            --application-name $APP_NAME \
            --deployment-group ${{ needs.determine-environment.outputs.target }}

  smoke-test:
    needs: [determine-environment, deploy-target]
    runs-on: ubuntu-latest
    
    steps:
      - name: Run smoke tests
        run: |
          for i in {1..10}; do
            STATUS=$(curl -s -o /dev/null -w "%{http_code}" \
              https://${{ needs.determine-environment.outputs.target }}.example.com/health)
            [ $STATUS -eq 200 ] && break
            sleep 10
          done
          [ $STATUS -eq 200 ] || exit 1

  switch-traffic:
    needs: [determine-environment, smoke-test]
    runs-on: ubuntu-latest
    
    steps:
      - name: Switch traffic to ${{ needs.determine-environment.outputs.target }}
        run: |
          aws elbv2 modify-listener \
            --listener-arn ${{ secrets.LISTENER_ARN }} \
            --default-actions Type=forward,TargetGroupArn=${{ secrets[format('{0}_TG_ARN', needs.determine-environment.outputs.target)] }}
      
      - name: Notify deployment
        run: |
          echo "Deployment complete!"
          echo "Active: ${{ needs.determine-environment.outputs.target }}"
          echo "Idle: ${{ needs.determine-environment.outputs.current }}"
```

---

#### **8. Rollback Strategy**

**Instant Rollback:**

```bash
#!/bin/bash
# scripts/rollback.sh

CURRENT_ENV=$(get_current_environment)

if [ "$CURRENT_ENV" = "blue" ]; then
  ROLLBACK_ENV="green"
else
  ROLLBACK_ENV="blue"
fi

echo "Rolling back to ${ROLLBACK_ENV}..."

# Switch traffic back
switch_traffic $ROLLBACK_ENV

echo "Rollback complete! Traffic reverted to ${ROLLBACK_ENV}"
```

---

#### **Best Practices**

1. **Health Checks**: Always verify target environment health
2. **Smoke Tests**: Run automated tests before switching
3. **Gradual Rollout**: Consider canary before full switch
4. **Database Compatibility**: Ensure backward-compatible migrations
5. **Monitoring**: Monitor both environments during switch
6. **Rollback Plan**: Always keep previous version ready
7. **Cost Management**: Terminate idle environment after validation
8. **Documentation**: Document rollback procedures

---

#### **Summary**

**Blue-Green Deployment Process:**

1. **Deploy**: New version to idle environment (green)
2. **Test**: Run smoke tests and health checks
3. **Switch**: Route traffic from blue to green
4. **Monitor**: Watch metrics and errors
5. **Rollback**: Switch back to blue if issues arise

**Key Benefits:**
- Zero downtime deployments
- Instant rollback capability
- Full testing before production
- Reduced deployment risk

**Platforms:**
- AWS (Elastic Beanstalk, ALB, ECS)
- Kubernetes (Service selector switching)
- Vercel/Netlify (Built-in preview + promote)
- Cloud Run, App Engine, Azure App Service

**Trade-offs:**
- ✅ Zero downtime
- ✅ Safe rollback
- ❌ Double infrastructure cost during deployment
- ❌ Database migration complexity

Blue-green deployment ensures **safe**, **reliable** releases with **minimal risk** and **instant rollback** capability.

</details>

---

### 258. What is canary deployment for React apps?

<details>
<summary>View Answer</summary>

**Canary deployment** gradually rolls out new versions to a small subset of users before full deployment, allowing you to **test in production** with minimal risk and **automatically rollback** if issues are detected.

---

#### **1. Canary Deployment Concept**

**Progressive Rollout:**

```
Phase 1: 5% of traffic → Canary (new version)
         95% of traffic → Stable (old version)
         ↓ Monitor metrics

Phase 2: 25% → Canary
         75% → Stable
         ↓ Monitor metrics

Phase 3: 50% → Canary
         50% → Stable
         ↓ Monitor metrics

Phase 4: 100% → Canary (full rollout)
         0% → Stable (terminated)
```

**Benefits:**
- ✅ Gradual exposure reduces risk
- ✅ Real user feedback before full rollout
- ✅ Automated rollback on errors
- ✅ Faster deployment cycles

---

#### **2. Kubernetes Canary Deployment**

**Stable Deployment:**

```yaml
# k8s/stable-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: react-app-stable
  labels:
    app: react-app
    version: stable
spec:
  replicas: 9  # 90% of traffic
  selector:
    matchLabels:
      app: react-app
      version: stable
  template:
    metadata:
      labels:
        app: react-app
        version: stable
    spec:
      containers:
      - name: react-app
        image: myregistry/react-app:v1.0.0
        ports:
        - containerPort: 80
```

**Canary Deployment:**

```yaml
# k8s/canary-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: react-app-canary
  labels:
    app: react-app
    version: canary
spec:
  replicas: 1  # 10% of traffic
  selector:
    matchLabels:
      app: react-app
      version: canary
  template:
    metadata:
      labels:
        app: react-app
        version: canary
    spec:
      containers:
      - name: react-app
        image: myregistry/react-app:v1.1.0
        ports:
        - containerPort: 80
```

**Service (Load Balances Both):**

```yaml
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: react-app-service
spec:
  selector:
    app: react-app  # Routes to both stable and canary
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
```

**Progressive Rollout Script:**

```bash
#!/bin/bash
# scripts/canary-rollout.sh

NEW_VERSION=$1
PHASES=(1 3 5 10)  # 10%, 30%, 50%, 100%

for phase in "${PHASES[@]}"; do
  CANARY_REPLICAS=$phase
  STABLE_REPLICAS=$((10 - phase))
  
  echo "Phase: ${phase}0% canary, $((10-phase))0% stable"
  
  # Scale deployments
  kubectl scale deployment react-app-canary --replicas=$CANARY_REPLICAS
  kubectl scale deployment react-app-stable --replicas=$STABLE_REPLICAS
  
  # Wait for rollout
  kubectl rollout status deployment/react-app-canary
  
  # Monitor metrics for 5 minutes
  echo "Monitoring metrics..."
  sleep 300
  
  # Check error rate
  ERROR_RATE=$(get_error_rate_from_prometheus)
  
  if (( $(echo "$ERROR_RATE > 0.05" | bc -l) )); then
    echo "Error rate too high! Rolling back..."
    kubectl scale deployment react-app-canary --replicas=0
    kubectl scale deployment react-app-stable --replicas=10
    exit 1
  fi
  
  echo "Phase successful. Continuing rollout..."
done

echo "Canary fully deployed. Updating stable..."
kubectl set image deployment/react-app-stable react-app=$NEW_VERSION
kubectl scale deployment react-app-canary --replicas=0
```

---

#### **3. Istio Service Mesh Canary**

**VirtualService for Traffic Splitting:**

```yaml
# k8s/virtual-service.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: react-app-vs
spec:
  hosts:
  - react-app.example.com
  http:
  - match:
    - headers:
        canary:
          exact: "true"
    route:
    - destination:
        host: react-app
        subset: canary
      weight: 100
  - route:
    - destination:
        host: react-app
        subset: stable
      weight: 90
    - destination:
        host: react-app
        subset: canary
      weight: 10
```

**DestinationRule:**

```yaml
# k8s/destination-rule.yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: react-app-dr
spec:
  host: react-app
  subsets:
  - name: stable
    labels:
      version: stable
  - name: canary
    labels:
      version: canary
```

**Progressive Traffic Shift:**

```bash
#!/bin/bash
# scripts/istio-canary.sh

for weight in 10 25 50 75 100; do
  echo "Routing ${weight}% to canary"
  
  kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: react-app-vs
spec:
  hosts:
  - react-app.example.com
  http:
  - route:
    - destination:
        host: react-app
        subset: stable
      weight: $((100 - weight))
    - destination:
        host: react-app
        subset: canary
      weight: ${weight}
EOF

  echo "Monitoring for 5 minutes..."
  sleep 300
  
  # Check metrics
  if ! check_canary_health; then
    echo "Canary failed. Rolling back..."
    rollback_canary
    exit 1
  fi
done

echo "Canary deployment successful!"
```

---

#### **4. Flagger Automated Canary**

**Flagger Canary Resource:**

```yaml
# k8s/flagger-canary.yaml
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: react-app
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: react-app
  
  service:
    port: 80
  
  # Progressive traffic shift
  analysis:
    interval: 1m
    threshold: 5
    maxWeight: 50
    stepWeight: 10
    
    # Metrics for automated promotion/rollback
    metrics:
    - name: request-success-rate
      thresholdRange:
        min: 99
      interval: 1m
    
    - name: request-duration
      thresholdRange:
        max: 500
      interval: 1m
    
    # Webhooks for custom checks
    webhooks:
    - name: load-test
      url: http://flagger-loadtester/
      timeout: 5s
      metadata:
        type: cmd
        cmd: "hey -z 1m -q 10 -c 2 http://react-app-canary/"
```

**Automatic Rollout:**

```
1. Flagger detects new deployment
2. Creates canary pods
3. Routes 10% traffic → canary
4. Checks metrics every 1 minute
5. If metrics OK → increase to 20%
6. Repeat until 50% or failure
7. Promote or rollback automatically
```

---

#### **5. AWS ALB Weighted Target Groups**

**ALB Configuration:**

```typescript
// infrastructure/alb-canary.ts
import * as elbv2 from 'aws-cdk-lib/aws-elasticloadbalancingv2';

const stableTargetGroup = new elbv2.ApplicationTargetGroup(this, 'Stable', {
  vpc,
  port: 80,
  targets: [stableAutoScalingGroup],
});

const canaryTargetGroup = new elbv2.ApplicationTargetGroup(this, 'Canary', {
  vpc,
  port: 80,
  targets: [canaryAutoScalingGroup],
});

// Listener with weighted routing
const listener = alb.addListener('Listener', {
  port: 80,
  defaultAction: elbv2.ListenerAction.weightedForward([
    {
      targetGroup: stableTargetGroup,
      weight: 90,
    },
    {
      targetGroup: canaryTargetGroup,
      weight: 10,
    },
  ]),
});
```

**Progressive Weight Update:**

```bash
#!/bin/bash
# scripts/alb-canary.sh

LISTENER_ARN="arn:aws:elasticloadbalancing:..."
STABLE_TG_ARN="arn:aws:elasticloadbalancing:.../stable"
CANARY_TG_ARN="arn:aws:elasticloadbalancing:.../canary"

for canary_weight in 10 25 50 75 100; do
  stable_weight=$((100 - canary_weight))
  
  echo "Routing ${canary_weight}% to canary"
  
  aws elbv2 modify-listener \
    --listener-arn $LISTENER_ARN \
    --default-actions \
      Type=forward,ForwardConfig="{
        TargetGroups=[
          {TargetGroupArn=${STABLE_TG_ARN},Weight=${stable_weight}},
          {TargetGroupArn=${CANARY_TG_ARN},Weight=${canary_weight}}
        ]
      }"
  
  echo "Monitoring for 5 minutes..."
  sleep 300
  
  # Check CloudWatch metrics
  ERROR_COUNT=$(aws cloudwatch get-metric-statistics \
    --namespace AWS/ApplicationELB \
    --metric-name HTTPCode_Target_5XX_Count \
    --dimensions Name=TargetGroup,Value=$CANARY_TG_ARN \
    --start-time $(date -u -d '5 minutes ago' +%Y-%m-%dT%H:%M:%S) \
    --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
    --period 300 \
    --statistics Sum \
    --query 'Datapoints[0].Sum' \
    --output text)
  
  if [ "$ERROR_COUNT" -gt 10 ]; then
    echo "Too many errors! Rolling back..."
    aws elbv2 modify-listener \
      --listener-arn $LISTENER_ARN \
      --default-actions Type=forward,TargetGroupArn=$STABLE_TG_ARN
    exit 1
  fi
done

echo "Canary successful! Promoting..."
```

---

#### **6. Feature Flag Based Canary**

**LaunchDarkly Integration:**

```typescript
// src/App.tsx
import { useLDClient, useFlags } from 'launchdarkly-react-client-sdk';

function App() {
  const { newFeature } = useFlags();
  const ldClient = useLDClient();
  
  // Track canary exposure
  useEffect(() => {
    if (newFeature) {
      ldClient?.track('canary-exposed', {
        version: 'v2.0',
      });
    }
  }, [newFeature, ldClient]);
  
  return (
    <div>
      {newFeature ? <NewComponent /> : <OldComponent />}
    </div>
  );
}
```

**Progressive Rollout with LaunchDarkly:**

```typescript
// LaunchDarkly dashboard or API
{
  "key": "new-feature",
  "variations": [
    { "value": false, "name": "off" },
    { "value": true, "name": "on" }
  ],
  "rules": [
    {
      "variation": 1,
      "rollout": {
        "variations": [
          { "variation": 0, "weight": 90000 },  // 90%
          { "variation": 1, "weight": 10000 }   // 10%
        ]
      }
    }
  ]
}
```

---

#### **7. Monitoring and Metrics**

**Prometheus Queries:**

```yaml
# Canary error rate
sum(rate(http_requests_total{version="canary",status=~"5.."}[5m])) 
/ 
sum(rate(http_requests_total{version="canary"}[5m]))

# Compare canary vs stable latency
histogram_quantile(0.95, 
  sum(rate(http_request_duration_seconds_bucket{version="canary"}[5m])) by (le)
)
-
histogram_quantile(0.95, 
  sum(rate(http_request_duration_seconds_bucket{version="stable"}[5m])) by (le)
)
```

**Automated Health Check:**

```typescript
// scripts/check-canary-health.ts
import axios from 'axios';

interface CanaryMetrics {
  errorRate: number;
  latencyP95: number;
  requestCount: number;
}

async function checkCanaryHealth(): Promise<boolean> {
  const prometheusUrl = 'http://prometheus:9090/api/v1/query';
  
  // Query error rate
  const errorRate = await axios.get(prometheusUrl, {
    params: {
      query: 'sum(rate(http_requests_total{version="canary",status=~"5.."}[5m])) / sum(rate(http_requests_total{version="canary"}[5m]))'
    }
  });
  
  const canaryErrorRate = parseFloat(errorRate.data.data.result[0]?.value[1] || '0');
  
  // Query stable error rate for comparison
  const stableErrorRate = await getStableErrorRate();
  
  // Canary should not be worse than stable
  if (canaryErrorRate > stableErrorRate * 1.5) {
    console.error('Canary error rate too high!');
    return false;
  }
  
  // Check latency
  const canaryLatency = await getCanaryLatency();
  const stableLatency = await getStableLatency();
  
  if (canaryLatency > stableLatency * 1.2) {
    console.error('Canary latency too high!');
    return false;
  }
  
  return true;
}
```

---

#### **8. Automated Rollback**

```yaml
# .github/workflows/canary-with-rollback.yml
name: Canary Deployment with Auto Rollback

on:
  push:
    branches: [main]

jobs:
  deploy-canary:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy canary
        run: |
          kubectl set image deployment/react-app-canary \
            react-app=myregistry/react-app:${{ github.sha }}
          kubectl scale deployment react-app-canary --replicas=1
      
      - name: Wait and monitor
        id: monitor
        run: |
          for i in {1..10}; do
            sleep 60
            
            ERROR_RATE=$(curl -s "http://prometheus:9090/api/v1/query?query=..." | jq -r '.data.result[0].value[1]')
            
            if (( $(echo "$ERROR_RATE > 0.05" | bc -l) )); then
              echo "rollback=true" >> $GITHUB_OUTPUT
              exit 1
            fi
          done
      
      - name: Rollback on failure
        if: failure()
        run: |
          echo "Rolling back canary..."
          kubectl scale deployment react-app-canary --replicas=0
          kubectl rollout undo deployment/react-app-canary
      
      - name: Promote canary
        if: success()
        run: |
          echo "Promoting canary to stable..."
          kubectl set image deployment/react-app-stable \
            react-app=myregistry/react-app:${{ github.sha }}
          kubectl scale deployment react-app-canary --replicas=0
```

---

#### **Best Practices**

1. **Small Increments**: Start with 5-10% traffic
2. **Metrics-Driven**: Monitor error rate, latency, success rate
3. **Automated Rollback**: Set thresholds for auto-rollback
4. **Gradual Rollout**: Don't rush to 100%
5. **User Stickiness**: Keep users on same version during session
6. **Health Checks**: Verify canary before increasing traffic
7. **Alerting**: Alert on canary failures
8. **Documentation**: Document rollback procedures

---

#### **Summary**

**Canary Deployment Process:**

1. **Deploy**: New version to canary with 5-10% traffic
2. **Monitor**: Track error rates, latency, metrics
3. **Increase**: Gradually increase traffic (25%, 50%, 75%)
4. **Validate**: Check metrics at each phase
5. **Promote or Rollback**: 100% rollout or revert

**Key Metrics:**
- Error rate (5xx responses)
- Latency (p50, p95, p99)
- Request count
- Success rate
- Resource usage

**Platforms:**
- Kubernetes (manual scaling)
- Istio (traffic splitting)
- Flagger (automated)
- AWS ALB (weighted target groups)
- Feature flags (LaunchDarkly, Split.io)

**Benefits:**
- ✅ Lower risk than blue-green
- ✅ Real user testing
- ✅ Automated rollback
- ✅ Gradual exposure

**Trade-offs:**
- ✅ Safer than all-at-once
- ❌ Slower than blue-green
- ❌ More complex monitoring
- ❌ Requires traffic splitting infrastructure

Canary deployment enables **safe**, **gradual** rollouts with **real user feedback** and **automated rollback** on issues.

</details>

---

### 259. How do you handle environment variables securely?

<details>
<summary>View Answer</summary>

**Secure environment variable management** is critical for protecting **API keys**, **secrets**, and **sensitive configuration**. Improper handling can lead to security breaches and data leaks.

---

#### **1. Environment Variables in React**

**Vite Environment Variables:**

```bash
# .env.local (NOT committed to git)
VITE_API_URL=https://api.example.com
VITE_API_KEY=sk_live_abc123xyz
VITE_SENTRY_DSN=https://xxx@sentry.io/123

# .env.production
VITE_API_URL=https://api.production.com

# .env.development
VITE_API_URL=http://localhost:3000
```

**Usage in Code:**

```typescript
// src/config/api.ts
const config = {
  apiUrl: import.meta.env.VITE_API_URL,
  apiKey: import.meta.env.VITE_API_KEY,
  sentryDsn: import.meta.env.VITE_SENTRY_DSN,
};

export default config;
```

**⚠️ Security Warning:**

```typescript
// ❌ BAD: Environment variables are bundled into client code
// Anyone can view them in browser DevTools
console.log(import.meta.env.VITE_API_KEY); // Visible in bundle!

// ✅ GOOD: Use server-side proxy for sensitive operations
// Client only has public identifiers
const publicConfig = {
  apiUrl: import.meta.env.VITE_API_URL, // OK - just a URL
  appId: import.meta.env.VITE_APP_ID,   // OK - public identifier
};
```

---

#### **2. Gitignore Configuration**

```bash
# .gitignore

# Environment files
.env
.env.local
.env.*.local
.env.production.local
.env.development.local

# Keep example file
!.env.example

# Secrets
secrets/
*.pem
*.key
*.crt

# IDE
.vscode/settings.json
.idea/
```

**Example Template:**

```bash
# .env.example (safe to commit)
VITE_API_URL=
VITE_APP_ID=
VITE_SENTRY_DSN=
# DO NOT commit actual values!
```

---

#### **3. CI/CD Secrets Management**

**GitHub Actions Secrets:**

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
      - uses: actions/checkout@v4
      
      - name: Build with secrets
        run: npm run build
        env:
          VITE_API_URL: ${{ secrets.VITE_API_URL }}
          VITE_API_KEY: ${{ secrets.VITE_API_KEY }}
          VITE_SENTRY_DSN: ${{ secrets.VITE_SENTRY_DSN }}
      
      - name: Deploy
        run: npm run deploy
        env:
          VERCEL_TOKEN: ${{ secrets.VERCEL_TOKEN }}
```

**GitLab CI Secrets:**

```yaml
# .gitlab-ci.yml
variables:
  VITE_API_URL: ${VITE_API_URL}
  
build:
  script:
    - npm run build
  only:
    - main
```

**CircleCI Secrets:**

```yaml
# .circleci/config.yml
version: 2.1

jobs:
  build:
    docker:
      - image: node:20
    steps:
      - checkout
      - run:
          name: Build
          command: npm run build
          environment:
            VITE_API_URL: ${VITE_API_URL}
```

---

#### **4. AWS Secrets Manager**

**Store Secrets:**

```bash
# Create secret
aws secretsmanager create-secret \
  --name react-app/production \
  --secret-string '{
    "VITE_API_KEY": "sk_live_abc123",
    "VITE_DATABASE_URL": "postgresql://..."
  }'
```

**Retrieve in CI/CD:**

```yaml
# .github/workflows/deploy.yml
steps:
  - name: Get secrets from AWS
    run: |
      SECRET=$(aws secretsmanager get-secret-value \
        --secret-id react-app/production \
        --query SecretString \
        --output text)
      
      echo "VITE_API_KEY=$(echo $SECRET | jq -r .VITE_API_KEY)" >> $GITHUB_ENV
      echo "VITE_DATABASE_URL=$(echo $SECRET | jq -r .VITE_DATABASE_URL)" >> $GITHUB_ENV
    env:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      AWS_REGION: us-east-1
  
  - name: Build
    run: npm run build
```

**Node.js Script:**

```typescript
// scripts/load-secrets.ts
import { 
  SecretsManagerClient, 
  GetSecretValueCommand 
} from '@aws-sdk/client-secrets-manager';
import fs from 'fs';

async function loadSecrets() {
  const client = new SecretsManagerClient({ region: 'us-east-1' });
  
  const command = new GetSecretValueCommand({
    SecretId: 'react-app/production',
  });
  
  const response = await client.send(command);
  const secrets = JSON.parse(response.SecretString!);
  
  // Write to .env file (not committed)
  const envContent = Object.entries(secrets)
    .map(([key, value]) => `${key}=${value}`)
    .join('\n');
  
  fs.writeFileSync('.env.local', envContent);
  console.log('Secrets loaded successfully');
}

loadSecrets();
```

---

#### **5. HashiCorp Vault Integration**

**Setup Vault Client:**

```typescript
// src/utils/vault.ts
import axios from 'axios';

class VaultClient {
  private baseUrl: string;
  private token: string;

  constructor(baseUrl: string, token: string) {
    this.baseUrl = baseUrl;
    this.token = token;
  }

  async getSecret(path: string): Promise<any> {
    const response = await axios.get(
      `${this.baseUrl}/v1/secret/data/${path}`,
      {
        headers: {
          'X-Vault-Token': this.token,
        },
      }
    );
    
    return response.data.data.data;
  }
}

export const vault = new VaultClient(
  process.env.VAULT_ADDR!,
  process.env.VAULT_TOKEN!
);
```

**Load Secrets at Build Time:**

```typescript
// scripts/build-with-vault.ts
import { vault } from './vault';
import { exec } from 'child_process';
import { promisify } from 'util';

const execAsync = promisify(exec);

async function buildWithSecrets() {
  // Fetch secrets from Vault
  const secrets = await vault.getSecret('react-app/production');
  
  // Set environment variables
  const env = {
    ...process.env,
    ...secrets,
  };
  
  // Build with secrets
  await execAsync('npm run build', { env });
  
  console.log('Build completed with secrets from Vault');
}

buildWithSecrets();
```

---

#### **6. Encrypted Environment Variables**

**Using SOPS (Secrets OPerationS):**

```bash
# Install SOPS
brew install sops

# Encrypt .env file
sops --encrypt .env.production > .env.production.enc

# Commit encrypted file
git add .env.production.enc

# Decrypt at build time
sops --decrypt .env.production.enc > .env.production
```

**GitHub Actions with SOPS:**

```yaml
# .github/workflows/deploy.yml
steps:
  - name: Install SOPS
    run: |
      wget https://github.com/mozilla/sops/releases/download/v3.7.3/sops-v3.7.3.linux
      chmod +x sops-v3.7.3.linux
      sudo mv sops-v3.7.3.linux /usr/local/bin/sops
  
  - name: Decrypt secrets
    run: sops --decrypt .env.production.enc > .env.production
    env:
      SOPS_AGE_KEY: ${{ secrets.SOPS_AGE_KEY }}
  
  - name: Build
    run: npm run build
```

---

#### **7. Runtime Configuration (Server-Side)**

**API Proxy Pattern:**

```typescript
// pages/api/config.ts (Next.js)
import { NextApiRequest, NextApiResponse } from 'next';

export default function handler(req: NextApiRequest, res: NextApiResponse) {
  // Server-side only - secrets never exposed to client
  const config = {
    apiUrl: process.env.API_URL,
    // Don't send sensitive keys to client!
    // apiKey: process.env.API_KEY, // ❌ DON'T DO THIS
  };
  
  res.status(200).json(config);
}
```

**Client Fetches Config:**

```typescript
// src/config/runtime.ts
let runtimeConfig: any = null;

export async function loadRuntimeConfig() {
  if (runtimeConfig) return runtimeConfig;
  
  const response = await fetch('/api/config');
  runtimeConfig = await response.json();
  
  return runtimeConfig;
}

// Usage
const App = () => {
  const [config, setConfig] = useState(null);
  
  useEffect(() => {
    loadRuntimeConfig().then(setConfig);
  }, []);
  
  if (!config) return <Loading />;
  
  return <Dashboard config={config} />;
};
```

---

#### **8. Environment-Specific Builds**

**Package.json Scripts:**

```json
{
  "scripts": {
    "build:dev": "vite build --mode development",
    "build:staging": "vite build --mode staging",
    "build:prod": "vite build --mode production"
  }
}
```

**Vite Mode Configuration:**

```typescript
// vite.config.ts
import { defineConfig, loadEnv } from 'vite';

export default defineConfig(({ mode }) => {
  const env = loadEnv(mode, process.cwd(), '');
  
  return {
    define: {
      __APP_ENV__: JSON.stringify(mode),
    },
    plugins: [react()],
  };
});
```

---

#### **9. Validation and Type Safety**

```typescript
// src/config/env.ts
import { z } from 'zod';

const envSchema = z.object({
  VITE_API_URL: z.string().url(),
  VITE_APP_ID: z.string().min(1),
  VITE_SENTRY_DSN: z.string().url().optional(),
  VITE_ENV: z.enum(['development', 'staging', 'production']),
});

function validateEnv() {
  try {
    const env = envSchema.parse(import.meta.env);
    return env;
  } catch (error) {
    console.error('❌ Invalid environment variables:', error);
    throw new Error('Invalid environment configuration');
  }
}

export const env = validateEnv();

// Usage with type safety
const apiUrl = env.VITE_API_URL; // TypeScript knows this is a string
```

---

#### **10. Security Best Practices**

**Pre-commit Hook:**

```bash
# .husky/pre-commit
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

# Check for secrets in staged files
if git diff --cached --name-only | xargs grep -E 'sk_live|api_key|secret_key|password' 2>/dev/null; then
  echo "❌ Potential secret detected in staged files!"
  echo "Please remove sensitive data before committing."
  exit 1
fi
```

**Git Secrets Scanner:**

```bash
# Install git-secrets
brew install git-secrets

# Initialize
git secrets --install

# Add patterns
git secrets --add 'sk_live_[0-9a-zA-Z]{32}'
git secrets --add 'api_key=[0-9a-zA-Z]+'
git secrets --add 'password=[0-9a-zA-Z]+'

# Scan repository
git secrets --scan
```

**Environment Variable Checklist:**

```typescript
// scripts/security-check.ts
const securityChecks = {
  envFilesNotCommitted: () => {
    // Check if .env files are in .gitignore
  },
  
  noSecretsInCode: () => {
    // Scan code for hardcoded secrets
  },
  
  ciSecretsConfigured: () => {
    // Verify CI/CD secrets are set
  },
  
  validationEnabled: () => {
    // Check if env validation is in place
  },
};
```

---

#### **11. Production Deployment Pattern**

```yaml
# .github/workflows/production-deploy.yml
name: Production Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    
    steps:
      - uses: actions/checkout@v4
      
      # Option 1: Use GitHub Secrets
      - name: Build with GitHub Secrets
        run: npm run build
        env:
          VITE_API_URL: ${{ secrets.PROD_API_URL }}
          VITE_SENTRY_DSN: ${{ secrets.PROD_SENTRY_DSN }}
      
      # Option 2: Load from AWS Secrets Manager
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Load secrets from AWS
        run: |
          aws secretsmanager get-secret-value \
            --secret-id react-app/prod \
            --query SecretString \
            --output text > secrets.json
          
          # Convert to env vars
          cat secrets.json | jq -r 'to_entries|map("\(.key)=\(.value|tostring)")|.[]' >> $GITHUB_ENV
      
      - name: Build
        run: npm run build
      
      - name: Deploy to production
        run: npm run deploy:prod
```

---

#### **Best Practices Summary**

1. **Never Commit Secrets**: Use .gitignore for .env files
2. **Use CI/CD Secrets**: Store in GitHub/GitLab/CircleCI secrets
3. **Rotate Regularly**: Change API keys and secrets periodically
4. **Least Privilege**: Only expose what's necessary to client
5. **Validate Environment**: Use schema validation (Zod)
6. **Encrypt at Rest**: Use SOPS or similar for encrypted secrets
7. **Audit Access**: Monitor who accesses secrets
8. **Separate by Environment**: Different secrets for dev/staging/prod
9. **Use Secret Managers**: AWS Secrets Manager, Vault, etc.
10. **Scan for Leaks**: Use git-secrets, pre-commit hooks

---

#### **Summary**

**Environment Variable Security:**

1. **Never commit** .env files with secrets
2. **Use CI/CD secrets** for deployment
3. **Validate** environment variables at build time
4. **Encrypt** secrets in repository if needed (SOPS)
5. **Use secret managers** (AWS Secrets Manager, Vault)
6. **Runtime config** for sensitive server-side values
7. **Scan for leaks** with git-secrets
8. **Rotate secrets** regularly

**Key Principles:**
- Client-side env vars are **always visible** in bundle
- Only expose **public identifiers** to client
- Keep **API keys and secrets** server-side
- Use **proxy APIs** for sensitive operations
- Validate **environment configuration** at startup

**Tools:**
- **GitHub/GitLab Secrets**: Built-in CI/CD secrets
- **AWS Secrets Manager**: Managed secret storage
- **HashiCorp Vault**: Enterprise secret management
- **SOPS**: Encrypted file management
- **git-secrets**: Prevent secret commits
- **Zod**: Runtime validation

Proper environment variable management prevents **security breaches** and ensures **safe deployment** of React applications.

</details>

---

### 260. What is Docker and containerization for React apps?

<details>
<summary>View Answer</summary>

**Docker** is a containerization platform that packages applications with their dependencies into **isolated**, **portable** containers. For React apps, Docker ensures **consistent environments** across development, testing, and production.

---

#### **1. Basic Dockerfile for React**

```dockerfile
# Dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

EXPOSE 3000

CMD ["npm", "run", "preview"]
```

**Build and Run:**

```bash
# Build image
docker build -t react-app .

# Run container
docker run -p 3000:3000 react-app

# Access at http://localhost:3000
```

---

#### **2. Multi-Stage Dockerfile (Production)**

```dockerfile
# Stage 1: Build
FROM node:20-alpine AS builder

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
FROM nginx:alpine

# Copy built assets from builder
COPY --from=builder /app/dist /usr/share/nginx/html

# Copy nginx configuration
COPY nginx.conf /etc/nginx/nginx.conf

# Expose port
EXPOSE 80

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost/ || exit 1

# Start nginx
CMD ["nginx", "-g", "daemon off;"]
```

**Nginx Configuration:**

```nginx
# nginx.conf
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
  worker_connections 1024;
}

http {
  include /etc/nginx/mime.types;
  default_type application/octet-stream;
  
  log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';
  
  access_log /var/log/nginx/access.log main;
  
  sendfile on;
  tcp_nopush on;
  keepalive_timeout 65;
  gzip on;
  gzip_types text/plain text/css application/json application/javascript text/xml application/xml;
  
  server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;
    
    # SPA routing
    location / {
      try_files $uri $uri/ /index.html;
    }
    
    # Cache static assets
    location ~* \.(?:css|js|jpg|jpeg|png|gif|ico|svg|woff|woff2|ttf|eot)$ {
      expires 1y;
      add_header Cache-Control "public, immutable";
    }
    
    # Security headers
    add_header X-Frame-Options "DENY" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
  }
}
```

---

#### **3. Optimized Production Dockerfile**

```dockerfile
# Multi-stage build with optimizations
FROM node:20-alpine AS deps

WORKDIR /app

# Copy only package files for better caching
COPY package.json package-lock.json ./

RUN npm ci --only=production

# Build stage
FROM node:20-alpine AS builder

WORKDIR /app

# Copy dependencies from deps stage
COPY --from=deps /app/node_modules ./node_modules

# Copy source
COPY . .

# Build with production environment
ENV NODE_ENV=production
RUN npm run build

# Remove development dependencies
RUN npm prune --production

# Production stage
FROM nginx:alpine AS production

# Install nodejs for runtime if needed
RUN apk add --no-cache nodejs npm

# Copy built files
COPY --from=builder /app/dist /usr/share/nginx/html

# Copy nginx config
COPY nginx.conf /etc/nginx/nginx.conf

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Set ownership
RUN chown -R nodejs:nodejs /usr/share/nginx/html

EXPOSE 80

HEALTHCHECK CMD wget --quiet --tries=1 --spider http://localhost/ || exit 1

CMD ["nginx", "-g", "daemon off;"]
```

---

#### **4. Docker Compose for Development**

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      # Hot reload
      - ./src:/app/src
      - ./public:/app/public
      # Don't override node_modules
      - /app/node_modules
    environment:
      - VITE_API_URL=http://api:4000
      - CHOKIDAR_USEPOLLING=true
    depends_on:
      - api
    networks:
      - app-network
  
  api:
    image: node:20-alpine
    working_dir: /app
    volumes:
      - ./api:/app
    command: npm start
    ports:
      - "4000:4000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/myapp
    depends_on:
      - db
    networks:
      - app-network
  
  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=myapp
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-network

volumes:
  postgres-data:

networks:
  app-network:
    driver: bridge
```

**Development Dockerfile:**

```dockerfile
# Dockerfile.dev
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0"]
```

**Run Development Environment:**

```bash
# Start all services
docker-compose up

# Run in background
docker-compose up -d

# View logs
docker-compose logs -f app

# Stop services
docker-compose down
```

---

#### **5. Docker with Environment Variables**

```dockerfile
# Dockerfile with build args
FROM node:20-alpine AS builder

WORKDIR /app

# Accept build arguments
ARG VITE_API_URL
ARG VITE_APP_VERSION

COPY package*.json ./
RUN npm ci

COPY . .

# Build with environment variables
RUN VITE_API_URL=${VITE_API_URL} \
    VITE_APP_VERSION=${VITE_APP_VERSION} \
    npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**Build with Arguments:**

```bash
# Build with environment variables
docker build \
  --build-arg VITE_API_URL=https://api.example.com \
  --build-arg VITE_APP_VERSION=1.2.3 \
  -t react-app:1.2.3 .

# Using .env file
docker build --build-arg $(cat .env | xargs) -t react-app .
```

---

#### **6. Kubernetes Deployment**

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: react-app
  labels:
    app: react-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: react-app
  template:
    metadata:
      labels:
        app: react-app
    spec:
      containers:
      - name: react-app
        image: myregistry/react-app:latest
        ports:
        - containerPort: 80
        env:
        - name: NODE_ENV
          value: "production"
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 30
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
---
apiVersion: v1
kind: Service
metadata:
  name: react-app-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
  selector:
    app: react-app
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: react-app-ingress
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - example.com
    secretName: react-app-tls
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: react-app-service
            port:
              number: 80
```

**Deploy to Kubernetes:**

```bash
# Apply configuration
kubectl apply -f k8s/

# Check status
kubectl get pods
kubectl get services

# View logs
kubectl logs -f deployment/react-app

# Scale
kubectl scale deployment react-app --replicas=5

# Update image
kubectl set image deployment/react-app react-app=myregistry/react-app:v2
```

---

#### **7. CI/CD with Docker**

```yaml
# .github/workflows/docker.yml
name: Docker Build and Push

on:
  push:
    branches: [main]
    tags:
      - 'v*'

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          build-args: |
            VITE_API_URL=${{ secrets.VITE_API_URL }}
            VITE_APP_VERSION=${{ github.sha }}
      
      - name: Deploy to Kubernetes
        if: github.ref == 'refs/heads/main'
        run: |
          kubectl set image deployment/react-app \
            react-app=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:sha-${{ github.sha }}
        env:
          KUBECONFIG: ${{ secrets.KUBECONFIG }}
```

---

#### **8. Docker Best Practices**

**1. Use .dockerignore:**

```bash
# .dockerignore
node_modules
npm-debug.log
.git
.gitignore
.env
.env.local
dist
build
coverage
.vscode
.idea
*.md
Dockerfile
docker-compose.yml
.dockerignore
```

**2. Layer Caching:**

```dockerfile
# ✅ GOOD: Copy package files first
COPY package*.json ./
RUN npm ci
COPY . .

# ❌ BAD: Copy everything first (cache invalidated on any file change)
COPY . .
RUN npm ci
```

**3. Minimize Image Size:**

```dockerfile
# Use alpine variants
FROM node:20-alpine  # ~40MB vs node:20 ~300MB

# Multi-stage build
FROM nginx:alpine    # ~24MB vs nginx ~133MB

# Remove unnecessary files
RUN npm prune --production && \
    npm cache clean --force
```

**4. Security:**

```dockerfile
# Run as non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
USER nodejs

# Scan for vulnerabilities
# docker scan react-app
```

---

#### **9. Container Orchestration Comparison**

| Feature | Docker Compose | Kubernetes | Docker Swarm |
|---------|---------------|------------|-------------|
| **Complexity** | Low | High | Medium |
| **Scalability** | Limited | Excellent | Good |
| **Production Ready** | Development | Yes | Yes |
| **Learning Curve** | Easy | Steep | Moderate |
| **Auto-scaling** | No | Yes | Limited |
| **Self-healing** | No | Yes | Yes |
| **Best For** | Local dev | Production | Small clusters |

---

#### **Summary**

**Docker Benefits for React Apps:**

1. **Consistency**: Same environment everywhere
2. **Isolation**: No conflicts with host system
3. **Portability**: Run anywhere Docker runs
4. **Scalability**: Easy horizontal scaling
5. **CI/CD**: Automated builds and deployments
6. **Reproducibility**: Exact same build every time

**Key Concepts:**

- **Image**: Template for containers (like a class)
- **Container**: Running instance (like an object)
- **Dockerfile**: Instructions to build image
- **Docker Compose**: Multi-container applications
- **Registry**: Store and distribute images (Docker Hub, GitHub Container Registry)

**Common Commands:**

```bash
# Build
docker build -t myapp .

# Run
docker run -p 3000:80 myapp

# List containers
docker ps

# Stop container
docker stop <container-id>

# Remove container
docker rm <container-id>

# View logs
docker logs <container-id>

# Execute command in container
docker exec -it <container-id> sh
```

**Production Deployment:**

1. Build optimized multi-stage image
2. Push to container registry
3. Deploy to orchestration platform (Kubernetes)
4. Configure load balancing and auto-scaling
5. Set up monitoring and logging
6. Implement health checks and rolling updates

Docker ensures **reliable**, **consistent**, and **scalable** deployment of React applications.

</details>
