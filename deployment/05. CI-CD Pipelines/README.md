# CI/CD Pipelines

## Table of Contents
1. [What is CI/CD?](#what-is-cicd)
2. [Continuous Integration (CI)](#continuous-integration-ci)
3. [Continuous Deployment (CD)](#continuous-deployment-cd)
4. [Popular CI/CD Tools](#popular-cicd-tools)
5. [GitHub Actions](#github-actions)
6. [GitLab CI/CD](#gitlab-cicd)
7. [Jenkins](#jenkins)
8. [CircleCI](#circleci)
9. [Pipeline Best Practices](#pipeline-best-practices)
10. [Multi-Environment Deployments](#multi-environment-deployments)

---

## What is CI/CD?

### Definition
CI/CD (Continuous Integration/Continuous Deployment) is a method to frequently deliver apps to customers by introducing automation into the stages of app development.

### Key Components
```
Developer → Git Push → CI Pipeline → Automated Tests → CD Pipeline → Production
```

### Benefits
- **Faster Time to Market**: Automated deployments reduce manual work
- **Early Bug Detection**: Tests run on every commit
- **Consistent Deployments**: Same process every time
- **Rollback Capability**: Easy to revert to previous versions
- **Developer Productivity**: Less time on deployment, more on coding

### CI/CD vs Traditional Deployment
```
Traditional:
- Manual builds
- Manual testing
- Manual deployment
- High risk of human error

CI/CD:
- Automated builds
- Automated testing
- Automated deployment
- Consistent and reliable
```

---

## Continuous Integration (CI)

### What is CI?
Process of automatically building and testing code changes when developers push to a repository.

### CI Workflow
```yaml
1. Developer commits code
2. CI server detects change
3. Code is checked out
4. Dependencies installed
5. Code is built
6. Tests are run
7. Results reported back
```

### Key CI Practices
```javascript
// 1. Frequent Commits
// Commit code multiple times per day

// 2. Automated Build
{
  "scripts": {
    "build": "npm run build",
    "test": "npm run test",
    "lint": "eslint src/**/*.ts"
  }
}

// 3. Automated Testing
describe('User Service', () => {
  it('should create user', async () => {
    const user = await userService.create({
      email: 'test@example.com'
    });
    expect(user).toBeDefined();
  });
});

// 4. Fast Builds
// Keep builds under 10 minutes

// 5. Test in Production-like Environment
// Use Docker containers for consistency
```

### CI Pipeline Stages
```yaml
stages:
  - install:
      - Install dependencies
      - Cache node_modules
  
  - lint:
      - Check code style
      - Run ESLint
      - Run Prettier check
  
  - build:
      - Compile TypeScript
      - Build frontend assets
      - Optimize images
  
  - test:
      - Unit tests
      - Integration tests
      - Code coverage
  
  - security:
      - Dependency scanning
      - SAST (Static Analysis)
      - License checking
```

---

## Continuous Deployment (CD)

### What is CD?
Automatically deploying every change that passes all stages of the production pipeline to production.

### CD vs Continuous Delivery
```
Continuous Delivery:
- Automatically deploy to staging
- Manual approval for production
- Human gate-keeping

Continuous Deployment:
- Fully automated to production
- No manual intervention
- Higher confidence required
```

### CD Workflow
```yaml
1. Code passes CI pipeline
2. Build artifacts created
3. Deploy to staging environment
4. Run smoke tests
5. Deploy to production (canary/blue-green)
6. Monitor metrics
7. Automatic rollback if issues detected
```

### Deployment Strategies

#### 1. Rolling Deployment
```yaml
# Deploy to instances gradually
Instance 1: v1 → v2
Instance 2: v1 → v2
Instance 3: v1 → v2
Instance 4: v1 → v2

# Advantage: Zero downtime
# Disadvantage: Both versions running simultaneously
```

#### 2. Blue-Green Deployment
```yaml
# Two identical environments
Blue (Current):  v1 → Live Traffic
Green (New):     v2 → No Traffic

# After testing, switch traffic
Blue (Old):      v1 → No Traffic (Keep for rollback)
Green (Current): v2 → Live Traffic
```

```javascript
// Example: AWS Elastic Beanstalk Blue-Green
{
  "environments": {
    "blue": {
      "version": "1.0.0",
      "status": "Live",
      "traffic": "100%"
    },
    "green": {
      "version": "2.0.0",
      "status": "Staging",
      "traffic": "0%"
    }
  }
}

// Swap environments
await elasticBeanstalk.swapEnvironmentCNAMEs({
  sourceEnvironmentName: 'myapp-blue',
  destinationEnvironmentName: 'myapp-green'
});
```

#### 3. Canary Deployment
```yaml
# Gradually shift traffic to new version
v1: 100% traffic
v2: 0% traffic

# Increase gradually
v1: 90% traffic
v2: 10% traffic (monitor metrics)

v1: 70% traffic
v2: 30% traffic

v1: 0% traffic
v2: 100% traffic (if no issues)
```

```javascript
// Example: Nginx canary configuration
upstream backend_v1 {
  server app-v1:3000;
}

upstream backend_v2 {
  server app-v2:3000;
}

split_clients "${remote_addr}" $backend_version {
  90%     v1;  # 90% to old version
  10%     v2;  # 10% to new version (canary)
}

server {
  location / {
    if ($backend_version = "v1") {
      proxy_pass http://backend_v1;
    }
    if ($backend_version = "v2") {
      proxy_pass http://backend_v2;
    }
  }
}
```

#### 4. Feature Flags
```typescript
// Gradual feature rollout
interface FeatureFlag {
  name: string;
  enabled: boolean;
  rolloutPercentage: number;
  userSegments?: string[];
}

class FeatureFlagService {
  isEnabled(flagName: string, userId: string): boolean {
    const flag = this.getFlag(flagName);
    
    if (!flag.enabled) return false;
    
    // Percentage-based rollout
    const userHash = this.hashUserId(userId);
    if (userHash < flag.rolloutPercentage) {
      return true;
    }
    
    return false;
  }
}

// Usage in application
if (featureFlags.isEnabled('new-checkout', user.id)) {
  return <NewCheckoutComponent />;
} else {
  return <OldCheckoutComponent />;
}
```

---

## Popular CI/CD Tools

### Comparison Table
```
┌──────────────┬──────────────┬─────────────┬──────────────┐
│ Tool         │ Hosting      │ Best For    │ Pricing      │
├──────────────┼──────────────┼─────────────┼──────────────┤
│ GitHub       │ Cloud/       │ GitHub      │ Free for     │
│ Actions      │ Self-hosted  │ projects    │ public repos │
├──────────────┼──────────────┼─────────────┼──────────────┤
│ GitLab CI    │ Cloud/       │ GitLab      │ Free tier    │
│              │ Self-hosted  │ projects    │ available    │
├──────────────┼──────────────┼─────────────┼──────────────┤
│ Jenkins      │ Self-hosted  │ Enterprise, │ Free (open   │
│              │              │ Custom      │ source)      │
├──────────────┼──────────────┼─────────────┼──────────────┤
│ CircleCI     │ Cloud        │ Quick setup │ Free tier    │
├──────────────┼──────────────┼─────────────┼──────────────┤
│ Travis CI    │ Cloud        │ Open source │ Free for OSS │
├──────────────┼──────────────┼─────────────┼──────────────┤
│ Azure        │ Cloud        │ Microsoft   │ Free tier    │
│ Pipelines    │              │ stack       │              │
└──────────────┴──────────────┴─────────────┴──────────────┘
```

---

## GitHub Actions

### What is GitHub Actions?
CI/CD platform integrated directly into GitHub repositories.

### Key Concepts
```yaml
Workflow: Automated process defined in YAML
Job: Set of steps that execute on the same runner
Step: Individual task (run command or action)
Action: Reusable unit of code
Runner: Server that runs workflows
```

### Basic Workflow Structure
```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

# When to run
on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

# Environment variables
env:
  NODE_VERSION: '18.x'

# Jobs to run
jobs:
  build-and-test:
    runs-on: ubuntu-latest
    
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
    
    - name: Run tests
      run: npm test
    
    - name: Build application
      run: npm run build
    
    - name: Upload artifacts
      uses: actions/upload-artifact@v3
      with:
        name: build-files
        path: dist/
```

### Full Stack Deployment Example
```yaml
# .github/workflows/deploy.yml
name: Deploy Full Stack Application

on:
  push:
    branches: [ main ]

jobs:
  # Job 1: Build and Test Frontend
  frontend:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./frontend
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18.x'
        cache: 'npm'
        cache-dependency-path: frontend/package-lock.json
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run tests
      run: npm test
    
    - name: Build
      run: npm run build
      env:
        REACT_APP_API_URL: ${{ secrets.API_URL }}
    
    - name: Deploy to Vercel
      uses: amondnet/vercel-action@v25
      with:
        vercel-token: ${{ secrets.VERCEL_TOKEN }}
        vercel-org-id: ${{ secrets.ORG_ID }}
        vercel-project-id: ${{ secrets.PROJECT_ID }}
        working-directory: ./frontend
  
  # Job 2: Build and Test Backend
  backend:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
    
    defaults:
      run:
        working-directory: ./backend
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18.x'
        cache: 'npm'
        cache-dependency-path: backend/package-lock.json
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run database migrations
      run: npm run migration:run
      env:
        DATABASE_URL: postgresql://postgres:postgres@localhost:5432/testdb
    
    - name: Run tests
      run: npm test
      env:
        DATABASE_URL: postgresql://postgres:postgres@localhost:5432/testdb
    
    - name: Build
      run: npm run build
    
    - name: Deploy to Heroku
      uses: akhileshns/heroku-deploy@v3.12.14
      with:
        heroku_api_key: ${{ secrets.HEROKU_API_KEY }}
        heroku_app_name: "my-app-backend"
        heroku_email: ${{ secrets.HEROKU_EMAIL }}
        appdir: "backend"
  
  # Job 3: Run E2E Tests (depends on frontend and backend)
  e2e-tests:
    needs: [frontend, backend]
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18.x'
    
    - name: Install Playwright
      run: npx playwright install --with-deps
    
    - name: Run E2E tests
      run: npx playwright test
      env:
        BASE_URL: ${{ secrets.FRONTEND_URL }}
        API_URL: ${{ secrets.BACKEND_URL }}
```

### Advanced GitHub Actions Features

#### 1. Matrix Builds
```yaml
# Test on multiple Node versions and OS
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [16.x, 18.x, 20.x]
    
    steps:
    - uses: actions/checkout@v3
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
    - run: npm ci
    - run: npm test
```

#### 2. Conditional Steps
```yaml
steps:
  - name: Deploy to production
    if: github.ref == 'refs/heads/main'
    run: npm run deploy:prod
  
  - name: Deploy to staging
    if: github.ref == 'refs/heads/develop'
    run: npm run deploy:staging
```

#### 3. Caching Dependencies
```yaml
- name: Cache node modules
  uses: actions/cache@v3
  with:
    path: |
      ~/.npm
      node_modules
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
```

#### 4. Secrets Management
```yaml
# Use encrypted secrets
env:
  DATABASE_URL: ${{ secrets.DATABASE_URL }}
  API_KEY: ${{ secrets.API_KEY }}
  AWS_ACCESS_KEY: ${{ secrets.AWS_ACCESS_KEY }}
```

---

## GitLab CI/CD

### What is GitLab CI/CD?
Built-in CI/CD tool in GitLab with powerful features and easy configuration.

### Basic Pipeline Configuration
```yaml
# .gitlab-ci.yml
stages:
  - install
  - test
  - build
  - deploy

variables:
  NODE_VERSION: "18"

# Cache dependencies
cache:
  paths:
    - node_modules/
    - .npm/

# Install dependencies
install:
  stage: install
  image: node:${NODE_VERSION}
  script:
    - npm ci --cache .npm --prefer-offline
  artifacts:
    paths:
      - node_modules/
    expire_in: 1 day

# Run tests
test:unit:
  stage: test
  image: node:${NODE_VERSION}
  script:
    - npm run test:unit
  coverage: '/Statements\s+:\s+(\d+\.\d+)%/'

test:integration:
  stage: test
  image: node:${NODE_VERSION}
  services:
    - postgres:14
  variables:
    POSTGRES_DB: testdb
    POSTGRES_USER: postgres
    POSTGRES_PASSWORD: postgres
  script:
    - npm run test:integration

# Build application
build:
  stage: build
  image: node:${NODE_VERSION}
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week

# Deploy to staging
deploy:staging:
  stage: deploy
  image: alpine:latest
  script:
    - apk add --no-cache curl
    - curl -X POST $STAGING_WEBHOOK_URL
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - develop

# Deploy to production
deploy:production:
  stage: deploy
  image: alpine:latest
  script:
    - apk add --no-cache curl
    - curl -X POST $PRODUCTION_WEBHOOK_URL
  environment:
    name: production
    url: https://example.com
  only:
    - main
  when: manual  # Require manual approval
```

### Advanced GitLab CI Features

#### 1. Dynamic Child Pipelines
```yaml
# Parent pipeline
trigger-child:
  trigger:
    include:
      - local: .gitlab-ci-child.yml
    strategy: depend
```

#### 2. Multi-Project Pipelines
```yaml
# Trigger pipeline in another project
trigger-microservice:
  trigger:
    project: group/microservice-project
    branch: main
```

#### 3. GitLab Pages Deployment
```yaml
pages:
  stage: deploy
  script:
    - npm run build
    - mv dist public
  artifacts:
    paths:
      - public
  only:
    - main
```

---

## Jenkins

### What is Jenkins?
Open-source automation server for building, testing, and deploying software.

### Jenkinsfile (Declarative Pipeline)
```groovy
// Jenkinsfile
pipeline {
    agent any
    
    environment {
        NODE_VERSION = '18'
        DOCKER_IMAGE = 'myapp'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Install Dependencies') {
            steps {
                nodejs(nodeJSInstallationName: "Node ${NODE_VERSION}") {
                    sh 'npm ci'
                }
            }
        }
        
        stage('Lint') {
            steps {
                nodejs(nodeJSInstallationName: "Node ${NODE_VERSION}") {
                    sh 'npm run lint'
                }
            }
        }
        
        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        nodejs(nodeJSInstallationName: "Node ${NODE_VERSION}") {
                            sh 'npm run test:unit'
                        }
                    }
                }
                stage('Integration Tests') {
                    steps {
                        nodejs(nodeJSInstallationName: "Node ${NODE_VERSION}") {
                            sh 'npm run test:integration'
                        }
                    }
                }
            }
        }
        
        stage('Build') {
            steps {
                nodejs(nodeJSInstallationName: "Node ${NODE_VERSION}") {
                    sh 'npm run build'
                }
            }
        }
        
        stage('Docker Build') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                    docker.build("${DOCKER_IMAGE}:latest")
                }
            }
        }
        
        stage('Docker Push') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-credentials') {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                        docker.image("${DOCKER_IMAGE}:latest").push()
                    }
                }
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                sh '''
                    kubectl set image deployment/myapp \
                    myapp=${DOCKER_IMAGE}:${DOCKER_TAG} \
                    --namespace=staging
                '''
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to production?', ok: 'Deploy'
                sh '''
                    kubectl set image deployment/myapp \
                    myapp=${DOCKER_IMAGE}:${DOCKER_TAG} \
                    --namespace=production
                '''
            }
        }
    }
    
    post {
        always {
            junit 'test-results/**/*.xml'
            publishHTML([
                reportDir: 'coverage',
                reportFiles: 'index.html',
                reportName: 'Coverage Report'
            ])
        }
        success {
            slackSend(
                color: 'good',
                message: "Build ${env.BUILD_NUMBER} succeeded"
            )
        }
        failure {
            slackSend(
                color: 'danger',
                message: "Build ${env.BUILD_NUMBER} failed"
            )
        }
    }
}
```

### Scripted Pipeline (Advanced)
```groovy
node {
    def app
    
    stage('Clone repository') {
        checkout scm
    }
    
    stage('Build image') {
        app = docker.build("myapp:${env.BUILD_ID}")
    }
    
    stage('Test image') {
        app.inside {
            sh 'npm test'
        }
    }
    
    stage('Push image') {
        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
            app.push("${env.BUILD_NUMBER}")
            app.push("latest")
        }
    }
    
    stage('Deploy') {
        if (env.BRANCH_NAME == 'main') {
            sh 'kubectl apply -f k8s/'
            sh "kubectl set image deployment/myapp myapp=myapp:${env.BUILD_ID}"
        }
    }
}
```

---

## CircleCI

### Configuration
```yaml
# .circleci/config.yml
version: 2.1

# Define reusable commands
commands:
  install-deps:
    steps:
      - restore_cache:
          keys:
            - v1-deps-{{ checksum "package-lock.json" }}
            - v1-deps-
      - run:
          name: Install Dependencies
          command: npm ci
      - save_cache:
          key: v1-deps-{{ checksum "package-lock.json" }}
          paths:
            - node_modules

# Define jobs
jobs:
  test:
    docker:
      - image: cimg/node:18.0
      - image: cimg/postgres:14.0
        environment:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: testdb
    
    steps:
      - checkout
      - install-deps
      
      - run:
          name: Wait for Postgres
          command: |
            dockerize -wait tcp://localhost:5432 -timeout 1m
      
      - run:
          name: Run Tests
          command: npm test
      
      - store_test_results:
          path: test-results
      
      - store_artifacts:
          path: coverage
  
  build:
    docker:
      - image: cimg/node:18.0
    
    steps:
      - checkout
      - install-deps
      
      - run:
          name: Build Application
          command: npm run build
      
      - persist_to_workspace:
          root: .
          paths:
            - dist
  
  deploy:
    docker:
      - image: cimg/node:18.0
    
    steps:
      - checkout
      - attach_workspace:
          at: .
      
      - run:
          name: Deploy to Production
          command: |
            npm install -g vercel
            vercel --prod --token=$VERCEL_TOKEN

# Define workflows
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

## Pipeline Best Practices

### 1. Keep Pipelines Fast
```yaml
# Bad: Sequential jobs (slow)
stages:
  - lint (2 min)
  - test (5 min)
  - build (3 min)
Total: 10 minutes

# Good: Parallel jobs (fast)
stages:
  - lint (2 min) } 
  - test (5 min) } Run in parallel
  - build (3 min) }
Total: 5 minutes
```

### 2. Fail Fast
```yaml
# Run quick checks first
stages:
  - lint          # 30 seconds - catch syntax errors
  - unit-tests    # 2 minutes - catch logic errors
  - build         # 3 minutes - catch build errors
  - integration   # 10 minutes - catch integration errors
  - deploy        # 5 minutes - only if all passed
```

### 3. Use Caching
```yaml
# Cache dependencies
cache:
  paths:
    - node_modules/
    - .npm/
    - ~/.cache/Cypress

# Cache Docker layers
- name: Cache Docker layers
  uses: actions/cache@v3
  with:
    path: /tmp/.buildx-cache
    key: ${{ runner.os }}-buildx-${{ github.sha }}
```

### 4. Environment Parity
```dockerfile
# Use same Node version everywhere
FROM node:18.16.0-alpine

# Same in CI
image: node:18.16.0

# Same in development
{
  "engines": {
    "node": "18.16.0"
  }
}
```

### 5. Secrets Management
```yaml
# Never commit secrets
# Use environment variables
env:
  DATABASE_URL: ${{ secrets.DATABASE_URL }}
  API_KEY: ${{ secrets.API_KEY }}

# Use secret management tools
- AWS Secrets Manager
- HashiCorp Vault
- Azure Key Vault
```

### 6. Build Once, Deploy Many
```yaml
# Build artifact once
build:
  - npm run build
  - Create Docker image: myapp:v1.2.3

# Deploy same artifact everywhere
deploy-staging:
  - Deploy: myapp:v1.2.3

deploy-production:
  - Deploy: myapp:v1.2.3  # Same image!
```

### 7. Version Everything
```yaml
# Tag with version and commit
docker build -t myapp:1.2.3-${GIT_COMMIT_SHA} .
docker build -t myapp:1.2.3 .
docker build -t myapp:latest .
```

### 8. Automated Rollbacks
```typescript
// Deployment script with health checks
async function deploy() {
  const oldVersion = await getCurrentVersion();
  
  try {
    await deployNewVersion();
    await runHealthChecks();
    
    if (await isHealthy()) {
      console.log('Deployment successful');
    } else {
      throw new Error('Health checks failed');
    }
  } catch (error) {
    console.log('Rolling back to', oldVersion);
    await rollback(oldVersion);
    throw error;
  }
}
```

---

## Multi-Environment Deployments

### Environment Strategy
```
Development → Staging → Production

Development:
- Auto-deploy on every commit
- Latest features
- Can be unstable

Staging:
- Production-like environment
- QA testing
- Performance testing

Production:
- Manual approval
- Stable releases only
- Blue-green deployment
```

### Environment-Specific Configuration
```yaml
# .github/workflows/deploy.yml
jobs:
  deploy:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        environment: [development, staging, production]
    
    steps:
    - name: Deploy to ${{ matrix.environment }}
      run: |
        npm run build:${{ matrix.environment }}
        npm run deploy:${{ matrix.environment }}
      env:
        API_URL: ${{ secrets[format('{0}_API_URL', matrix.environment)] }}
        DATABASE_URL: ${{ secrets[format('{0}_DB_URL', matrix.environment)] }}
```

### Configuration Files
```typescript
// config/environments.ts
interface EnvironmentConfig {
  apiUrl: string;
  databaseUrl: string;
  logLevel: string;
  features: Record<string, boolean>;
}

const environments: Record<string, EnvironmentConfig> = {
  development: {
    apiUrl: 'http://localhost:3000',
    databaseUrl: 'postgresql://localhost:5432/dev',
    logLevel: 'debug',
    features: {
      newFeature: true
    }
  },
  staging: {
    apiUrl: 'https://api-staging.example.com',
    databaseUrl: process.env.STAGING_DATABASE_URL!,
    logLevel: 'info',
    features: {
      newFeature: true
    }
  },
  production: {
    apiUrl: 'https://api.example.com',
    databaseUrl: process.env.PRODUCTION_DATABASE_URL!,
    logLevel: 'error',
    features: {
      newFeature: false  // Feature flag
    }
  }
};

export const config = environments[process.env.NODE_ENV || 'development'];
```

---

## Interview Questions

### Basic Questions

**Q1: What is the difference between Continuous Integration and Continuous Deployment?**
```
Continuous Integration (CI):
- Automatically build and test code
- Runs on every commit
- Catches bugs early
- Example: Run tests on pull request

Continuous Deployment (CD):
- Automatically deploy to production
- Requires passing CI
- No manual intervention
- Example: Auto-deploy main branch to production

Key Difference:
CI = Test automatically
CD = Deploy automatically
```

**Q2: What are the benefits of CI/CD?**
```
1. Faster Development Cycles
   - Deploy multiple times per day
   - Quick feedback on changes

2. Reduced Risk
   - Small, incremental changes
   - Easy to identify issues
   - Quick rollbacks

3. Improved Quality
   - Automated testing
   - Consistent processes
   - Early bug detection

4. Better Collaboration
   - Clear deployment process
   - Automated documentation
   - Shared responsibility
```

**Q3: Explain Blue-Green Deployment**
```
Two identical environments:
- Blue: Current production (v1.0)
- Green: New version (v2.0)

Process:
1. Deploy v2.0 to Green
2. Test Green thoroughly
3. Switch traffic: Blue → Green
4. Keep Blue for rollback
5. If issues: Switch back to Blue

Benefits:
- Zero downtime
- Instant rollback
- Safe testing

Disadvantage:
- Double infrastructure cost
```

### Advanced Questions

**Q4: How do you handle database migrations in CI/CD?**
```typescript
// Strategy 1: Backward Compatible Migrations
// Step 1: Add new column (optional)
ALTER TABLE users ADD COLUMN email_verified BOOLEAN DEFAULT false;

// Deploy application that handles both old and new schema
if (user.email_verified !== undefined) {
  // Use new column
} else {
  // Fallback for old schema
}

// Step 2: Migrate data
UPDATE users SET email_verified = true WHERE confirmed = true;

// Step 3: Remove old column
ALTER TABLE users DROP COLUMN confirmed;

// Strategy 2: Blue-Green with Database
// 1. Create read replica
// 2. Apply migrations to replica
// 3. Switch application to replica
// 4. Promote replica to primary

// Strategy 3: Feature Flags
if (featureFlags.isEnabled('new-schema')) {
  // Use new schema
} else {
  // Use old schema
}
```

**Q5: How do you implement canary deployments?**
```yaml
# Kubernetes canary deployment
# Step 1: Deploy canary version (10% traffic)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-canary
spec:
  replicas: 1  # 10% of traffic
  template:
    metadata:
      labels:
        app: myapp
        version: v2.0
        track: canary
---
# Step 2: Existing stable version (90% traffic)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-stable
spec:
  replicas: 9  # 90% of traffic
  template:
    metadata:
      labels:
        app: myapp
        version: v1.0
        track: stable
---
# Service distributes traffic based on replica count
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp  # Matches both stable and canary
```

**Q6: How do you handle secrets in CI/CD pipelines?**
```yaml
# Bad Practices (Never do this):
DATABASE_URL=postgresql://user:password@host/db  # In code
AWS_SECRET_KEY=abc123  # In .env file committed to git

# Good Practices:

# 1. Use CI/CD platform secrets
# GitHub Actions
env:
  DATABASE_URL: ${{ secrets.DATABASE_URL }}

# 2. Use secret management services
steps:
  - name: Get secrets from AWS Secrets Manager
    run: |
      SECRETS=$(aws secretsmanager get-secret-value \
        --secret-id prod/myapp/db \
        --query SecretString \
        --output text)
      export DATABASE_URL=$(echo $SECRETS | jq -r '.DATABASE_URL')

# 3. Use environment-specific secrets
development:
  secrets:
    - dev-secrets
staging:
  secrets:
    - staging-secrets
production:
  secrets:
    - prod-secrets

# 4. Rotate secrets regularly
- Schedule: Every 90 days
- Automated rotation scripts
- Zero-downtime rotation
```

**Q7: How do you ensure zero-downtime deployments?**
```typescript
// 1. Health Checks
app.get('/health', (req, res) => {
  res.json({
    status: 'healthy',
    version: process.env.APP_VERSION,
    uptime: process.uptime()
  });
});

// 2. Graceful Shutdown
process.on('SIGTERM', async () => {
  console.log('SIGTERM received, shutting down gracefully');
  
  // Stop accepting new requests
  server.close(async () => {
    // Wait for existing requests to complete
    await Promise.all(activeRequests);
    
    // Close database connections
    await database.disconnect();
    
    process.exit(0);
  });
  
  // Force shutdown after 30 seconds
  setTimeout(() => {
    console.error('Forced shutdown');
    process.exit(1);
  }, 30000);
});

// 3. Rolling Deployment Configuration
// Kubernetes
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Create 1 new pod before killing old
      maxUnavailable: 0  # Never have less than desired replicas
  
  # Health checks
  readinessProbe:  # When pod is ready for traffic
    httpGet:
      path: /health
      port: 3000
    initialDelaySeconds: 5
    periodSeconds: 10
  
  livenessProbe:  # When pod should be restarted
    httpGet:
      path: /health
      port: 3000
    initialDelaySeconds: 15
    periodSeconds: 20

// 4. Database Migrations
// Always backward compatible
// - Add columns before using them
// - Remove columns after stopping usage
```

**Q8: What is your rollback strategy?**
```yaml
# Strategy 1: Redeploy Previous Version
deploy-production:
  script:
    - VERSION=${CI_COMMIT_TAG}
    - docker push myapp:${VERSION}
    - kubectl set image deployment/myapp myapp=myapp:${VERSION}
  
  # Rollback job
rollback-production:
  when: manual
  script:
    - PREVIOUS_VERSION=$(git describe --tags --abbrev=0 HEAD^)
    - kubectl set image deployment/myapp myapp=myapp:${PREVIOUS_VERSION}

# Strategy 2: Blue-Green Rollback
# Just switch traffic back to blue environment
rollback:
  - Switch load balancer: Green → Blue
  - Time: < 1 minute

# Strategy 3: Automatic Rollback on Failure
deploy:
  script:
    - deploy_new_version()
    - wait 5m  # Monitor metrics
    - if error_rate > 1% then rollback
    - if response_time > 1s then rollback

# Strategy 4: Feature Flag Rollback
// Instant rollback without redeployment
featureFlags.disable('new-feature');
```

---

## Summary

### CI/CD Pipeline Flow
```
1. Code Push
   ↓
2. CI Pipeline Triggers
   - Install dependencies
   - Run linter
   - Run tests
   - Build application
   ↓
3. Create Artifacts
   - Docker images
   - Build files
   - Tagged versions
   ↓
4. CD Pipeline Triggers
   - Deploy to staging
   - Run integration tests
   - Deploy to production
   ↓
5. Monitor
   - Check metrics
   - Alert on errors
   - Auto-rollback if needed
```

### Key Takeaways
1. **Automate Everything**: Build, test, and deploy automatically
2. **Fail Fast**: Run quick checks first
3. **Keep It Fast**: Use caching and parallelization
4. **Deploy Small Changes**: Easier to debug and rollback
5. **Monitor Actively**: Watch metrics after deployment
6. **Have Rollback Plan**: Always be able to go back
7. **Use Feature Flags**: Control features without deployment
8. **Test in Production-like Environment**: Staging = mini production

CI/CD enables rapid, reliable, and repeatable software delivery! 🚀
