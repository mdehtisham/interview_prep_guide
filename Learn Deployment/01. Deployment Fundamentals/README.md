# Deployment Fundamentals

## Table of Contents
- [What is Deployment?](#what-is-deployment)
- [Types of Deployment](#types-of-deployment)
- [Deployment Environments](#deployment-environments)
- [Key Deployment Concepts](#key-deployment-concepts)
- [Common Deployment Platforms](#common-deployment-platforms)
- [Interview Questions](#interview-questions)

---

## What is Deployment?

**Deployment** is the process of making your application available to users by moving it from a development environment to a production environment where it can be accessed over the internet.

### Deployment Process Overview:
```
Local Development → Build/Compile → Test → Deploy → Production Server → Users Access
```

### Key Stages:
1. **Development**: Writing code locally
2. **Building**: Compiling and bundling code
3. **Testing**: Running tests to ensure quality
4. **Staging**: Deploying to a test environment
5. **Production**: Deploying to live servers
6. **Monitoring**: Tracking performance and errors

---

## Types of Deployment

### 1. **Manual Deployment**
- Developer manually uploads files to server
- Uses FTP, SSH, or cloud console
- **Pros**: Simple, direct control
- **Cons**: Error-prone, time-consuming, not scalable

### 2. **Automated Deployment**
- Uses CI/CD pipelines
- Automated testing and deployment
- **Pros**: Fast, reliable, consistent
- **Cons**: Initial setup complexity

### 3. **Continuous Deployment (CD)**
- Every code change automatically deployed to production
- Requires extensive automated testing
- **Use case**: SaaS applications with rapid iterations

### 4. **Continuous Delivery**
- Code is always ready to deploy, but requires manual approval
- Automated pipeline up to staging
- **Use case**: Enterprise applications with strict change control

### 5. **Blue-Green Deployment**
- Two identical environments (Blue = current, Green = new)
- Switch traffic from Blue to Green after testing
- **Advantage**: Zero downtime, easy rollback

### 6. **Canary Deployment**
- Deploy to small subset of users first
- Gradually increase if no issues
- **Advantage**: Risk mitigation

### 7. **Rolling Deployment**
- Update servers one by one
- Always some servers available
- **Advantage**: No downtime

---

## Deployment Environments

### 1. **Development (Dev)**
- Local machine or shared dev server
- Frequent changes
- Connected to test database
- **URL Example**: `localhost:3000` or `dev.myapp.com`

### 2. **Staging (QA/UAT)**
- Mirror of production environment
- Used for final testing
- Connected to staging database
- **URL Example**: `staging.myapp.com`

### 3. **Production (Prod)**
- Live environment accessible to end users
- Highest security and performance
- Connected to production database
- **URL Example**: `myapp.com` or `www.myapp.com`

### Environment Configuration Matrix:
```
┌─────────────┬──────────────┬──────────────┬──────────────┐
│ Aspect      │ Development  │ Staging      │ Production   │
├─────────────┼──────────────┼──────────────┼──────────────┤
│ Database    │ Test DB      │ Staging DB   │ Production DB│
│ API Keys    │ Test Keys    │ Test Keys    │ Live Keys    │
│ Debugging   │ Enabled      │ Enabled      │ Disabled     │
│ Logging     │ Verbose      │ Detailed     │ Error only   │
│ Caching     │ Minimal      │ Moderate     │ Aggressive   │
│ Monitoring  │ Optional     │ Recommended  │ Required     │
└─────────────┴──────────────┴──────────────┴──────────────┘
```

---

## Key Deployment Concepts

### 1. **Build Process**
Converting source code into executable/deployable format:
```bash
# Frontend (React/Vue/Angular)
npm run build  # Creates optimized production build

# Backend (Node.js/TypeScript)
tsc  # Compiles TypeScript to JavaScript
npm run build
```

**Build Output**:
- Minified code
- Bundled assets
- Optimized images
- Compiled stylesheets

### 2. **Environment Variables**
Configuration values that change per environment:
```bash
# .env.development
DATABASE_URL=localhost:5432/myapp_dev
API_URL=http://localhost:4000
DEBUG=true

# .env.production
DATABASE_URL=prod-db.amazonaws.com:5432/myapp_prod
API_URL=https://api.myapp.com
DEBUG=false
```

### 3. **DNS (Domain Name System)**
Maps domain names to IP addresses:
```
myapp.com → DNS Server → 52.23.45.67 (Your Server IP)
```

**Common DNS Records**:
- **A Record**: Maps domain to IPv4 address
- **CNAME**: Alias one domain to another
- **MX**: Mail exchange servers
- **TXT**: Text records (verification, SPF)

### 4. **SSL/TLS Certificates**
Encrypts data between client and server (HTTPS):
- **Let's Encrypt**: Free automated certificates
- **Paid Certificates**: Extended validation, wildcard
- **Purpose**: Security, SEO ranking, user trust

### 5. **Load Balancing**
Distributes traffic across multiple servers:
```
       Load Balancer
       /     |     \
   Server1 Server2 Server3
```

### 6. **Reverse Proxy**
Sits between client and server:
- **Nginx**: Popular choice
- **Apache**: Traditional option
- **Benefits**: Security, caching, SSL termination, load balancing

### 7. **CDN (Content Delivery Network)**
Distributes static assets globally:
```
User in Japan → Tokyo CDN Server (fast)
User in USA → New York CDN Server (fast)
```

**Popular CDNs**: Cloudflare, AWS CloudFront, Fastly

### 8. **Containers**
Packages application with all dependencies:
```dockerfile
# Dockerfile
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["node", "server.js"]
```

### 9. **Orchestration**
Manages multiple containers:
- **Docker Compose**: Multi-container apps
- **Kubernetes (K8s)**: Enterprise-scale orchestration
- **Docker Swarm**: Docker's native orchestration

### 10. **Server Types**

#### **Traditional Server (VPS)**
- Full control over OS
- Manual configuration
- Examples: DigitalOcean Droplets, AWS EC2

#### **Platform as a Service (PaaS)**
- Managed infrastructure
- Focus on code, not servers
- Examples: Heroku, Railway, Render

#### **Serverless**
- No server management
- Pay per execution
- Examples: AWS Lambda, Vercel, Netlify

#### **Static Hosting**
- Only for static files
- Very fast and cheap
- Examples: GitHub Pages, Netlify, Vercel

---

## Common Deployment Platforms

### Frontend Deployment:
| Platform | Best For | Free Tier |
|----------|----------|-----------|
| **Vercel** | Next.js, React, Vue | Yes |
| **Netlify** | Static sites, JAMstack | Yes |
| **AWS S3 + CloudFront** | Enterprise static sites | Limited |
| **GitHub Pages** | Documentation, portfolios | Yes |
| **Firebase Hosting** | Google ecosystem apps | Yes |
| **Surge** | Quick static deployments | Yes |

### Backend Deployment:
| Platform | Best For | Free Tier |
|----------|----------|-----------|
| **Heroku** | Quick prototypes | Limited |
| **Railway** | Modern apps | Limited |
| **Render** | Full-stack apps | Yes |
| **AWS Elastic Beanstalk** | Scalable apps | Limited |
| **DigitalOcean App Platform** | Dockerized apps | No |
| **Google Cloud Run** | Containerized apps | Limited |
| **Azure App Service** | Enterprise apps | Limited |
| **Fly.io** | Edge computing | Limited |

### Database Deployment:
| Platform | Database Types | Free Tier |
|----------|----------------|-----------|
| **AWS RDS** | PostgreSQL, MySQL, etc. | Limited |
| **MongoDB Atlas** | MongoDB | Yes (512MB) |
| **Supabase** | PostgreSQL | Yes |
| **PlanetScale** | MySQL | Yes |
| **Neon** | PostgreSQL | Yes |
| **ElephantSQL** | PostgreSQL | Yes (20MB) |
| **Firebase Firestore** | NoSQL | Yes |

---

## Interview Questions

### Basic Level:

**Q1: What is deployment?**
**A:** Deployment is the process of making an application available to users by moving it from a development environment to a production server where it can be accessed over the internet.

**Q2: What is the difference between staging and production environments?**
**A:** 
- **Staging**: A pre-production environment that mirrors production, used for final testing before going live
- **Production**: The live environment where real users access the application

**Q3: What are environment variables and why are they important?**
**A:** Environment variables are configuration values that can change based on the environment (dev, staging, prod). They're important because:
- Keep sensitive data (API keys, database passwords) out of code
- Allow same codebase to work in different environments
- Enable easy configuration changes without code modification

**Q4: What is a build process?**
**A:** A build process converts source code into an optimized, deployable format. For frontend, it includes minification, bundling, and asset optimization. For backend, it may include compilation (TypeScript to JavaScript) and dependency bundling.

**Q5: What is the difference between HTTP and HTTPS?**
**A:** 
- **HTTP**: Unencrypted communication, data sent in plain text
- **HTTPS**: Encrypted communication using SSL/TLS certificates, secure data transmission

### Intermediate Level:

**Q6: Explain the difference between horizontal and vertical scaling.**
**A:** 
- **Vertical Scaling (Scale Up)**: Adding more resources (CPU, RAM) to existing server. Limited by hardware constraints.
- **Horizontal Scaling (Scale Out)**: Adding more servers to handle load. More flexible and fault-tolerant but requires load balancing.

**Q7: What is a reverse proxy and why use it?**
**A:** A reverse proxy (like Nginx) sits between clients and servers:
- **Benefits**: SSL termination, caching, load balancing, security (hides backend structure)
- **Common use**: Serve static files, route requests to different backend services

**Q8: What is the difference between CI and CD?**
**A:** 
- **CI (Continuous Integration)**: Automatically building and testing code when changes are pushed
- **CD (Continuous Delivery)**: Code is always ready to deploy but requires manual approval
- **CD (Continuous Deployment)**: Automatically deploys every change to production after passing tests

**Q9: What is containerization and its benefits?**
**A:** Containerization (Docker) packages an application with all its dependencies:
- **Benefits**: Consistency across environments, isolation, easy scaling, version control
- **Use case**: Deploy same container from dev to prod without "works on my machine" issues

**Q10: How do you handle database migrations in production?**
**A:** 
- Use migration tools (TypeORM, Sequelize, Flyway)
- Test migrations in staging first
- Backup database before migration
- Use rolling migrations (backward compatible changes)
- Monitor for issues after deployment

### Advanced Level:

**Q11: Explain Blue-Green vs Canary deployment strategies.**
**A:** 
- **Blue-Green**: Two identical environments. Switch all traffic at once from old (Blue) to new (Green). Instant rollback possible.
- **Canary**: Deploy to small percentage of users first (5%), gradually increase if metrics are good. Less risk but slower rollout.

**Q12: How would you achieve zero-downtime deployment?**
**A:** 
1. Use load balancer with health checks
2. Deploy to one server at a time (rolling deployment)
3. Load balancer removes unhealthy servers from pool
4. Database migrations must be backward compatible
5. Use feature flags for new features

**Q13: What is a CDN and when should you use it?**
**A:** CDN caches static assets on servers worldwide close to users:
- **When to use**: Global user base, large media files, high traffic
- **Benefits**: Faster load times, reduced server load, DDoS protection
- **Examples**: Images, CSS, JS, videos

**Q14: How do you handle secrets and sensitive data in deployment?**
**A:** 
- Never commit secrets to version control
- Use environment variables in production
- Use secret management services (AWS Secrets Manager, HashiCorp Vault)
- Rotate secrets regularly
- Use IAM roles for cloud services instead of access keys

**Q15: What monitoring should be in place after deployment?**
**A:** 
- **Application Monitoring**: Error tracking (Sentry, Rollbar)
- **Performance Monitoring**: Response times, throughput (New Relic, DataDog)
- **Infrastructure Monitoring**: CPU, memory, disk usage (CloudWatch, Prometheus)
- **Logging**: Centralized logs (ELK Stack, Splunk)
- **Uptime Monitoring**: Alert when site is down (UptimeRobot, Pingdom)
- **User Analytics**: Real user monitoring (Google Analytics, Mixpanel)

**Q16: Explain the concept of Infrastructure as Code (IaC).**
**A:** IaC manages infrastructure through code rather than manual processes:
- **Tools**: Terraform, CloudFormation, Pulumi
- **Benefits**: Version control, reproducibility, documentation, automation
- **Example**: Define entire AWS infrastructure in Terraform files
```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}
```

**Q17: How do you handle rollback in case of failed deployment?**
**A:** 
- **Automated rollback**: CI/CD detects health check failures and reverts
- **Manual rollback**: Keep previous version available, switch back quickly
- **Database**: Harder to rollback, use forward fixes or restore from backup
- **Versioning**: Tag deployments, use container image tags
- **Feature flags**: Disable new features without redeploying

**Q18: What is the difference between stateless and stateful applications in deployment?**
**A:** 
- **Stateless**: No session data stored on server. Each request independent. Easy to scale horizontally. Example: RESTful API
- **Stateful**: Session data stored on server. Requires sticky sessions or external session store. Example: WebSocket server
- **Best practice**: Design stateless applications, use external stores (Redis) for session data

**Q19: How do you ensure database consistency during deployment with multiple backend instances?**
**A:** 
- Use database connection pooling
- Implement proper locking mechanisms
- Use transactions for critical operations
- Consider eventual consistency for distributed systems
- Use database migration tools with locking (TypeORM, Flyway)
- Coordinate deployments (deploy one instance at a time)

**Q20: What are health checks and how are they implemented?**
**A:** Health checks verify if application is running correctly:
```javascript
// Express.js health check endpoint
app.get('/health', async (req, res) => {
  try {
    // Check database connection
    await db.query('SELECT 1');
    // Check external services
    // await externalAPI.ping();
    
    res.status(200).json({ 
      status: 'healthy',
      timestamp: new Date().toISOString()
    });
  } catch (error) {
    res.status(503).json({ 
      status: 'unhealthy',
      error: error.message 
    });
  }
});
```
Load balancers use these to route traffic only to healthy instances.

---

## Summary

Deployment is a critical phase in the software development lifecycle. Understanding the fundamentals of deployment environments, build processes, infrastructure options, and deployment strategies is essential for any developer. This foundation will help you make informed decisions when deploying frontend, backend, and database components of your applications.

**Key Takeaways**:
- Deployment involves multiple environments (dev, staging, production)
- Automation through CI/CD is crucial for reliability
- Environment variables separate configuration from code
- Different applications require different deployment strategies
- Monitoring and rollback plans are essential for production stability
