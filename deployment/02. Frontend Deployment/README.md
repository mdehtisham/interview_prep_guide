# Frontend Deployment

## Table of Contents
- [What is Frontend Deployment?](#what-is-frontend-deployment)
- [Frontend Build Process](#frontend-build-process)
- [Static Site Hosting](#static-site-hosting)
- [Deployment Platforms](#deployment-platforms)
- [Deployment Strategies](#deployment-strategies)
- [Configuration and Environment Variables](#configuration-and-environment-variables)
- [Performance Optimization](#performance-optimization)
- [Common Issues and Solutions](#common-issues-and-solutions)
- [Interview Questions](#interview-questions)

---

## What is Frontend Deployment?

Frontend deployment is the process of making your client-side application (HTML, CSS, JavaScript) accessible to users via the internet. Unlike backend deployment, frontend typically involves serving static files that run in the user's browser.

### Frontend Architecture After Deployment:
```
User's Browser → DNS → CDN (Optional) → Web Server → Static Files (HTML/CSS/JS)
                                              ↓
                                         API Calls → Backend Server
```

### Key Characteristics:
- **Static Files**: Pre-built HTML, CSS, JS, images
- **Client-Side Execution**: Code runs in user's browser
- **API Communication**: Connects to backend via HTTP/HTTPS
- **No Server Logic**: Business logic handled by backend or client

---

## Frontend Build Process

### 1. **Build Command**

Different frameworks have different build commands:

```bash
# React (Create React App)
npm run build
# Output: build/ directory

# React (Vite)
npm run build
# Output: dist/ directory

# Vue.js
npm run build
# Output: dist/ directory

# Angular
ng build --prod
# Output: dist/ directory

# Next.js (SSR/SSG)
npm run build
npm run start  # or next start

# Svelte
npm run build
# Output: public/build/
```

### 2. **What Happens During Build?**

```javascript
// Before Build (Development)
import React from 'react';
import './styles.css';
import logo from './logo.png';

function App() {
  const apiUrl = process.env.REACT_APP_API_URL;
  return <div className="app">Hello World</div>;
}

// After Build (Production)
// - All imports bundled into single/few files
// - CSS extracted and minified
// - Images optimized and hashed
// - Environment variables injected
// - Code minified and compressed
```

**Build Output Example**:
```
build/
├── index.html
├── static/
│   ├── css/
│   │   └── main.abc123.css  (minified, hashed)
│   ├── js/
│   │   ├── main.def456.js   (minified, hashed)
│   │   └── vendors.ghi789.js
│   └── media/
│       └── logo.jkl012.png  (optimized, hashed)
└── asset-manifest.json
```

### 3. **Build Optimizations**

#### **Code Splitting**
```javascript
// Lazy loading components
const Dashboard = React.lazy(() => import('./Dashboard'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <Dashboard />
    </Suspense>
  );
}
```

#### **Tree Shaking**
Removes unused code:
```javascript
// You import only one function
import { formatDate } from 'date-library';

// Build process only includes formatDate,
// not the entire library
```

#### **Minification**
```javascript
// Before
function calculateTotal(items) {
  let total = 0;
  for (let item of items) {
    total += item.price;
  }
  return total;
}

// After
function t(i){let t=0;for(let n of i)t+=n.price;return t}
```

#### **Asset Optimization**
- Images compressed (WebP, AVIF formats)
- SVG optimization
- Font subsetting
- Compression (Gzip, Brotli)

---

## Static Site Hosting

### What is Static Hosting?

Static hosting serves pre-built files without server-side processing. Perfect for frontend applications built with React, Vue, Angular.

### Static vs Dynamic Hosting:

```
Static Hosting:
User → Server → Returns HTML/CSS/JS files → Browser executes

Dynamic Hosting (SSR):
User → Server → Server renders HTML → Returns complete page → Browser displays
```

### Static Hosting Requirements:

1. **Web Server** (Nginx, Apache, or CDN)
2. **File Upload** (FTP, Git, CLI)
3. **Domain Name** (Optional but recommended)
4. **SSL Certificate** (For HTTPS)

---

## Deployment Platforms

### 1. **Vercel**

**Best for**: Next.js, React, Vue, Svelte

**Deployment Steps**:
```bash
# Install Vercel CLI
npm install -g vercel

# Deploy
vercel

# Production deployment
vercel --prod
```

**Configuration** (`vercel.json`):
```json
{
  "version": 2,
  "builds": [
    {
      "src": "package.json",
      "use": "@vercel/static-build",
      "config": { "distDir": "build" }
    }
  ],
  "routes": [
    { "src": "/api/(.*)", "dest": "/api/$1" },
    { "src": "/(.*)", "dest": "/index.html" }
  ]
}
```

**Features**:
- Automatic HTTPS
- Global CDN
- Instant rollbacks
- Preview deployments for PRs
- Environment variables
- Analytics

**Connecting to Backend**:
```javascript
// Frontend code
const API_URL = process.env.REACT_APP_API_URL; // https://api.myapp.com
fetch(`${API_URL}/users`)
  .then(res => res.json())
  .then(data => console.log(data));
```

---

### 2. **Netlify**

**Best for**: JAMstack, Static sites, React, Vue

**Deployment Methods**:

#### **Method 1: Git Integration**
1. Connect GitHub/GitLab repository
2. Netlify auto-deploys on push
3. Configure build settings

#### **Method 2: CLI**
```bash
# Install Netlify CLI
npm install -g netlify-cli

# Login
netlify login

# Deploy
netlify deploy

# Production
netlify deploy --prod
```

#### **Method 3: Drag & Drop**
- Build locally: `npm run build`
- Drag `build/` folder to Netlify dashboard

**Configuration** (`netlify.toml`):
```toml
[build]
  command = "npm run build"
  publish = "build"

[build.environment]
  NODE_VERSION = "18"

[[redirects]]
  from = "/api/*"
  to = "https://api.myapp.com/:splat"
  status = 200

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

**Features**:
- Continuous deployment from Git
- Form handling
- Serverless functions
- Split testing
- Deploy previews
- Custom domains

**Proxy Backend Requests**:
```toml
# Avoid CORS by proxying backend requests
[[redirects]]
  from = "/api/*"
  to = "https://backend.myapp.com/api/:splat"
  status = 200
  force = true
```

---

### 3. **AWS S3 + CloudFront**

**Best for**: Enterprise applications, full AWS ecosystem

**Deployment Steps**:

#### **1. Create S3 Bucket**
```bash
aws s3 mb s3://myapp-frontend
```

#### **2. Configure Bucket for Static Hosting**
```bash
aws s3 website s3://myapp-frontend \
  --index-document index.html \
  --error-document index.html
```

#### **3. Upload Build Files**
```bash
npm run build
aws s3 sync build/ s3://myapp-frontend --delete
```

#### **4. Set Bucket Policy**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "PublicReadGetObject",
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::myapp-frontend/*"
  }]
}
```

#### **5. Create CloudFront Distribution**
```bash
aws cloudfront create-distribution \
  --origin-domain-name myapp-frontend.s3.amazonaws.com \
  --default-root-object index.html
```

**Benefits of CloudFront CDN**:
- Global edge locations
- HTTPS support
- Custom domain
- Caching
- DDoS protection

**Automated Deployment Script**:
```bash
#!/bin/bash
# deploy.sh

# Build
npm run build

# Upload to S3
aws s3 sync build/ s3://myapp-frontend --delete

# Invalidate CloudFront cache
aws cloudfront create-invalidation \
  --distribution-id E1234567890 \
  --paths "/*"

echo "Deployment complete!"
```

---

### 4. **Firebase Hosting**

**Best for**: Google ecosystem, serverless

**Setup**:
```bash
# Install Firebase CLI
npm install -g firebase-tools

# Login
firebase login

# Initialize
firebase init hosting

# Deploy
firebase deploy --only hosting
```

**Configuration** (`firebase.json`):
```json
{
  "hosting": {
    "public": "build",
    "ignore": ["firebase.json", "**/.*", "**/node_modules/**"],
    "rewrites": [
      {
        "source": "/api/**",
        "function": "api"
      },
      {
        "source": "**",
        "destination": "/index.html"
      }
    ],
    "headers": [
      {
        "source": "/static/**",
        "headers": [
          {
            "key": "Cache-Control",
            "value": "public, max-age=31536000, immutable"
          }
        ]
      }
    ]
  }
}
```

**Features**:
- Free SSL
- Global CDN
- Custom domains
- Rollback support
- Integration with Firebase services

---

### 5. **GitHub Pages**

**Best for**: Documentation, portfolios, simple sites

**Limitations**: 
- Static sites only
- No server-side code
- No environment variables at build time

**Deployment**:

#### **Method 1: gh-pages Package**
```bash
npm install --save-dev gh-pages

# Add to package.json
{
  "homepage": "https://yourusername.github.io/repo-name",
  "scripts": {
    "predeploy": "npm run build",
    "deploy": "gh-pages -d build"
  }
}

# Deploy
npm run deploy
```

#### **Method 2: GitHub Actions**
```yaml
# .github/workflows/deploy.yml
name: Deploy to GitHub Pages

on:
  push:
    branches: [ main ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Setup Node
        uses: actions/setup-node@v2
        with:
          node-version: '18'
      
      - name: Install Dependencies
        run: npm ci
      
      - name: Build
        run: npm run build
        env:
          REACT_APP_API_URL: ${{ secrets.API_URL }}
      
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./build
```

---

## Deployment Strategies

### 1. **Manual Deployment**

```bash
# Build locally
npm run build

# Upload via FTP/SCP
scp -r build/* user@server:/var/www/html/

# Or use rsync
rsync -avz build/ user@server:/var/www/html/
```

### 2. **CI/CD with GitHub Actions**

```yaml
# .github/workflows/deploy.yml
name: Deploy Frontend

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
      
      - name: Build
        run: npm run build
        env:
          REACT_APP_API_URL: ${{ secrets.API_URL }}
          REACT_APP_ANALYTICS_ID: ${{ secrets.ANALYTICS_ID }}
      
      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v20
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.ORG_ID }}
          vercel-project-id: ${{ secrets.PROJECT_ID }}
          vercel-args: '--prod'
```

### 3. **GitLab CI/CD**

```yaml
# .gitlab-ci.yml
stages:
  - build
  - deploy

variables:
  NODE_VERSION: "18"

build:
  stage: build
  image: node:18
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - build/
    expire_in: 1 hour

deploy_production:
  stage: deploy
  image: alpine:latest
  before_script:
    - apk add --no-cache curl
  script:
    - curl -X POST $WEBHOOK_URL
  only:
    - main
```

---

## Configuration and Environment Variables

### 1. **Create React App**

```bash
# .env.development
REACT_APP_API_URL=http://localhost:4000
REACT_APP_ANALYTICS_ID=dev-123

# .env.production
REACT_APP_API_URL=https://api.myapp.com
REACT_APP_ANALYTICS_ID=prod-456
```

**Usage**:
```javascript
const apiUrl = process.env.REACT_APP_API_URL;
console.log(apiUrl); // https://api.myapp.com in production
```

**Important Notes**:
- Must prefix with `REACT_APP_`
- Variables are injected at **build time**, not runtime
- Don't put secrets here (they're visible in bundled code)

### 2. **Vite**

```bash
# .env.production
VITE_API_URL=https://api.myapp.com
VITE_APP_TITLE=My App
```

**Usage**:
```javascript
const apiUrl = import.meta.env.VITE_API_URL;
```

### 3. **Next.js**

```bash
# .env.production
NEXT_PUBLIC_API_URL=https://api.myapp.com
DATABASE_URL=postgres://...  # Server-side only
```

**Usage**:
```javascript
// Client-side (must prefix NEXT_PUBLIC_)
const apiUrl = process.env.NEXT_PUBLIC_API_URL;

// Server-side (no prefix needed)
const dbUrl = process.env.DATABASE_URL;
```

### 4. **Platform-Specific Environment Variables**

#### **Vercel**:
```bash
vercel env add REACT_APP_API_URL production
# Enter value when prompted
```

Or via dashboard: Settings → Environment Variables

#### **Netlify**:
Site settings → Build & deploy → Environment → Environment variables

#### **AWS Amplify**:
```yaml
# amplify.yml
version: 1
env:
  variables:
    REACT_APP_API_URL: https://api.myapp.com
frontend:
  phases:
    build:
      commands:
        - npm run build
  artifacts:
    baseDirectory: build
    files:
      - '**/*'
```

---

## Performance Optimization

### 1. **Caching Strategy**

```nginx
# Nginx configuration
location /static/ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}

location / {
    expires -1;
    add_header Cache-Control "no-cache, no-store, must-revalidate";
}
```

### 2. **Compression**

```javascript
// Express.js serving static files
const compression = require('compression');
const express = require('express');
const path = require('path');

const app = express();

// Enable Gzip compression
app.use(compression());

// Serve static files
app.use(express.static(path.join(__dirname, 'build'), {
  maxAge: '1y',
  immutable: true
}));

app.get('*', (req, res) => {
  res.sendFile(path.join(__dirname, 'build', 'index.html'));
});
```

### 3. **Asset Optimization**

```javascript
// webpack.config.js (for custom setups)
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          priority: 10
        }
      }
    }
  }
};
```

### 4. **Lazy Loading**

```javascript
// React lazy loading
const Home = React.lazy(() => import('./pages/Home'));
const About = React.lazy(() => import('./pages/About'));
const Dashboard = React.lazy(() => import('./pages/Dashboard'));

function App() {
  return (
    <Router>
      <Suspense fallback={<LoadingSpinner />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/dashboard" element={<Dashboard />} />
        </Routes>
      </Suspense>
    </Router>
  );
}
```

### 5. **CDN for Assets**

```javascript
// Upload images/videos to CDN
const imageUrl = 'https://cdn.myapp.com/images/logo.png';

// Or use services like Cloudinary
<img 
  src="https://res.cloudinary.com/myapp/image/upload/w_400,h_300,c_fill/logo.png"
  alt="Logo"
/>
```

---

## Common Issues and Solutions

### Issue 1: Routes Not Working (404 on Refresh)

**Problem**: SPA routing fails on page refresh

**Solution**: Configure server to always serve `index.html`

```nginx
# Nginx
location / {
    try_files $uri $uri/ /index.html;
}
```

```json
// Vercel (vercel.json)
{
  "rewrites": [
    { "source": "/(.*)", "destination": "/index.html" }
  ]
}
```

```toml
# Netlify (netlify.toml)
[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

### Issue 2: Environment Variables Not Working

**Problem**: `undefined` when accessing env variables

**Solutions**:
- Verify correct prefix (`REACT_APP_`, `VITE_`, `NEXT_PUBLIC_`)
- Rebuild application (env vars injected at build time)
- Check deployment platform environment settings
- Restart dev server after adding new env vars

### Issue 3: CORS Errors

**Problem**: Cannot access backend API from frontend

**Solutions**:

#### **Option 1: Configure Backend CORS**
```javascript
// Express.js backend
const cors = require('cors');
app.use(cors({
  origin: 'https://myapp.com',
  credentials: true
}));
```

#### **Option 2: Use Proxy**
```json
// package.json (development only)
{
  "proxy": "http://localhost:4000"
}
```

#### **Option 3: Platform Proxy**
```toml
# Netlify
[[redirects]]
  from = "/api/*"
  to = "https://backend.com/api/:splat"
  status = 200
```

### Issue 4: Large Bundle Size

**Solutions**:
- Code splitting
- Tree shaking
- Remove unused dependencies
- Use production builds
- Analyze bundle: `npm run build -- --stats`

```bash
# Analyze bundle with webpack-bundle-analyzer
npm install --save-dev webpack-bundle-analyzer
```

### Issue 5: Cache Not Updating

**Problem**: Users see old version after deployment

**Solutions**:
- File hashing (automatic in most tools)
- Cache invalidation (CloudFront, CDN)
- Service worker update strategy
- Version headers

---

## Interview Questions

### Basic Level:

**Q1: What is the difference between deploying frontend vs backend?**
**A:** Frontend deployment involves serving static files (HTML, CSS, JS) that run in the user's browser, while backend deployment involves running server-side code that processes requests, handles business logic, and manages databases.

**Q2: What happens during `npm run build`?**
**A:** The build process:
- Transpiles modern JavaScript to browser-compatible code
- Minifies and compresses code
- Bundles multiple files into optimized chunks
- Optimizes assets (images, fonts)
- Injects environment variables
- Creates production-ready static files

**Q3: Can you put API keys in frontend environment variables?**
**A:** No, frontend environment variables are embedded in the bundled code and visible to anyone who inspects the browser. API keys should be kept on the backend. Only use public/non-sensitive values in frontend env vars.

**Q4: What is a CDN and why use it for frontend?**
**A:** A CDN (Content Delivery Network) distributes static files across global servers. Benefits:
- Faster load times (files served from nearest location)
- Reduced server load
- Better scalability
- DDoS protection

**Q5: How do you deploy a React app to Netlify?**
**A:**
1. Connect GitHub repository to Netlify
2. Configure build settings: Build command: `npm run build`, Publish directory: `build`
3. Add environment variables if needed
4. Netlify auto-deploys on git push

### Intermediate Level:

**Q6: How do frontend and backend connect after deployment?**
**A:** Frontend makes HTTP requests to backend API endpoints:
```javascript
// Frontend calls backend API
fetch('https://api.myapp.com/users', {
  headers: { 'Authorization': `Bearer ${token}` }
})
```
Backend must:
- Be accessible via public URL
- Configure CORS to allow frontend domain
- Handle authentication/authorization

**Q7: What is the purpose of hash in filenames (main.abc123.js)?**
**A:** Hashes enable cache busting. When code changes, hash changes, forcing browsers to download new version instead of using cached old version. Files with same content always have same hash.

**Q8: How do you handle different API URLs for dev and production?**
**A:** Use environment-specific .env files:
```bash
# .env.development
REACT_APP_API_URL=http://localhost:4000

# .env.production
REACT_APP_API_URL=https://api.myapp.com
```

**Q9: Explain SPA routing issue on page refresh and its solution.**
**A:** SPAs handle routing client-side. On page refresh, browser requests `/about` from server, which doesn't exist (404). Solution: Configure server to always return `index.html` for all routes, then React Router handles navigation.

**Q10: What is the difference between S3 and CloudFront?**
**A:**
- **S3**: Storage service, single region, slower for global users
- **CloudFront**: CDN that caches S3 content at edge locations worldwide, faster global access
Best practice: Store in S3, serve via CloudFront

### Advanced Level:

**Q11: How would you implement a deployment pipeline with automated testing?**
**A:**
```yaml
# CI/CD Pipeline
1. Code Push → GitHub
2. Trigger → GitHub Actions/GitLab CI
3. Install dependencies → npm ci
4. Run linting → npm run lint
5. Run unit tests → npm test
6. Run E2E tests → npm run test:e2e
7. Build → npm run build
8. Deploy to staging → Auto deploy
9. Run smoke tests → Automated
10. Manual approval → Human check
11. Deploy to production → Auto deploy
12. Monitor → Error tracking
```

**Q12: How do you handle secrets that are needed at build time?**
**A:** 
- Store secrets in CI/CD platform (GitHub Secrets, GitLab Variables)
- Inject during build in pipeline
- Never commit to Git
- For truly sensitive operations, move logic to backend
```yaml
# GitHub Actions
- name: Build
  run: npm run build
  env:
    REACT_APP_API_URL: ${{ secrets.API_URL }}
```

**Q13: What is the difference between SSG, SSR, and CSR deployment?**
**A:**
- **CSR (Client-Side Rendering)**: Serve empty HTML + JS bundle, render in browser. Deploy as static files. Example: CRA on Netlify
- **SSR (Server-Side Rendering)**: Server renders HTML per request. Needs Node server. Example: Next.js on Vercel
- **SSG (Static Site Generation)**: Pre-render pages at build time. Deploy as static files. Example: Next.js export on S3

**Q14: How would you implement zero-downtime frontend deployment?**
**A:**
1. Most platforms handle this automatically (Vercel, Netlify)
2. For custom setup:
   - Upload new build to different directory
   - Run smoke tests
   - Atomic symlink swap: `ln -sfn /var/www/new /var/www/current`
   - Clear CDN cache if needed
3. Old version remains accessible during deployment

**Q15: How do you debug production issues in frontend?**
**A:**
- **Error Tracking**: Sentry, Rollbar, LogRocket
- **Source Maps**: Upload to error tracking service (keep private)
- **Logging**: Console logs, custom tracking
- **User Session Replay**: LogRocket, FullStory
- **Performance Monitoring**: Lighthouse, Web Vitals
- **Network Tab**: Check API calls, response times

**Q16: Explain cache invalidation strategies for frontend.**
**A:**
- **File Hashing**: Automatic, new filename = new file
- **Service Worker**: Control cache programmatically
- **Cache-Control Headers**: Set expiry times
- **CDN Invalidation**: Manually clear CDN cache
- **Version Query String**: `app.js?v=1.2.3` (less recommended)
- **HTML No-Cache**: Always fetch fresh HTML, hash JS/CSS

**Q17: How would you deploy a frontend to multiple regions?**
**A:**
- Use global CDN (CloudFront, Cloudflare)
- Deploy to multiple edge locations automatically
- Set up Route53/DNS for geo-routing
- For SSR: Deploy backend servers in multiple regions with load balancer
- Database: Read replicas in each region or global database (DynamoDB Global Tables)

**Q18: What are preview/staging deployments and why are they important?**
**A:** Preview deployments create temporary environments for each PR/branch:
- **Benefits**: Test changes before merging, share with stakeholders, catch issues early
- **Implementation**: Platforms like Vercel/Netlify create unique URL per PR
- **Use case**: Review UI changes, test integrations, demo features

**Q19: How do you handle frontend migrations with backward compatibility?**
**A:**
- **Feature Flags**: Enable/disable features without redeploying
- **Gradual Rollout**: Deploy to percentage of users first
- **API Versioning**: Support old and new API simultaneously
- **Backward Compatible Changes**: New backend supports old frontend
- **Coordinated Deployment**: Deploy backend first (backward compatible), then frontend

**Q20: Explain the role of service workers in deployment.**
**A:** Service workers cache assets and enable offline functionality:
```javascript
// service-worker.js
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open('v1').then((cache) => {
      return cache.addAll([
        '/',
        '/static/css/main.css',
        '/static/js/main.js'
      ]);
    })
  );
});
```
**Deployment considerations**:
- Update service worker version on each deployment
- Implement update notification
- Clear old caches
- Be careful: aggressive caching can prevent users from seeing updates

---

## Summary

Frontend deployment transforms your development code into optimized, production-ready static assets served to users globally. Key considerations include choosing the right platform, properly configuring environment variables, optimizing performance, and implementing CI/CD for reliable, automated deployments.

**Key Takeaways**:
- Build process optimizes code for production
- Static hosting is simple but SPA routing requires configuration
- Environment variables are injected at build time
- CDN improves global performance
- CI/CD pipelines automate testing and deployment
- Different platforms offer different trade-offs (simplicity vs control)
