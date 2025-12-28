# Monitoring and Logging

## Table of Contents
1. [Why Monitoring and Logging?](#why-monitoring-and-logging)
2. [Application Logging](#application-logging)
3. [Log Levels](#log-levels)
4. [Structured Logging](#structured-logging)
5. [Log Management Services](#log-management-services)
6. [Application Monitoring (APM)](#application-monitoring-apm)
7. [Error Tracking](#error-tracking)
8. [Performance Monitoring](#performance-monitoring)
9. [Infrastructure Monitoring](#infrastructure-monitoring)
10. [Alerting](#alerting)

---

## Why Monitoring and Logging?

### The Problem
```
Production Issue:
"The app is slow!"
"Users are getting errors!"
"Database is down!"

Without Monitoring:
😰 Don't know what happened
😰 Don't know when it started
😰 Don't know what caused it
😰 Can't reproduce
😰 Guessing in the dark

With Monitoring:
😊 See exact error
😊 Know when it occurred
😊 Understand root cause
😊 Track over time
😊 Fix proactively
```

### Key Benefits
```
1. Debugging
   - Trace errors in production
   - Understand user behavior
   - Reproduce issues

2. Performance
   - Identify bottlenecks
   - Optimize slow endpoints
   - Monitor resource usage

3. Reliability
   - Detect outages early
   - Alert before users notice
   - Track uptime

4. Business Insights
   - User analytics
   - Feature usage
   - Conversion funnels

5. Compliance
   - Audit trails
   - Security monitoring
   - Legal requirements
```

### Observability Pillars
```
┌────────────────────────────────────────┐
│  Metrics                               │
│  - CPU usage: 75%                      │
│  - Response time: 250ms                │
│  - Error rate: 0.5%                    │
└────────────────────────────────────────┘

┌────────────────────────────────────────┐
│  Logs                                  │
│  - Application logs                    │
│  - Access logs                         │
│  - Error logs                          │
└────────────────────────────────────────┘

┌────────────────────────────────────────┐
│  Traces                                │
│  - Request flow                        │
│  - Service dependencies                │
│  - Distributed tracing                 │
└────────────────────────────────────────┘
```

---

## Application Logging

### What to Log?
```
✅ DO Log:
- Errors and exceptions
- Authentication attempts
- Important business events
- Performance metrics
- External API calls
- Database queries (slow ones)

❌ DON'T Log:
- Passwords or secrets
- Credit card numbers
- Personal data (GDPR)
- Excessive debug info in production
```

### Basic Logging (Console)
```typescript
// Bad - No context
console.log('User created');

// Good - With context
console.log('User created', {
  userId: user.id,
  email: user.email,
  timestamp: new Date().toISOString()
});

// Better - Structured
logger.info('User created', {
  userId: user.id,
  email: user.email,
  ip: req.ip,
  userAgent: req.get('user-agent')
});
```

### Winston (Node.js Logger)
```typescript
// logger.ts
import winston from 'winston';

export const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: {
    service: 'my-app',
    environment: process.env.NODE_ENV
  },
  transports: [
    // Write to console
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.simple()
      )
    }),
    
    // Write to file
    new winston.transports.File({
      filename: 'logs/error.log',
      level: 'error'
    }),
    new winston.transports.File({
      filename: 'logs/combined.log'
    })
  ]
});

// In production, don't log to files
// Use external service instead
if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple()
  }));
}
```

### Usage in Application
```typescript
// user.service.ts
import { logger } from './logger';

export class UserService {
  async createUser(data: CreateUserDto) {
    try {
      logger.info('Creating user', { email: data.email });
      
      const user = await this.userRepository.create(data);
      
      logger.info('User created successfully', {
        userId: user.id,
        email: user.email
      });
      
      return user;
    } catch (error) {
      logger.error('Failed to create user', {
        email: data.email,
        error: error.message,
        stack: error.stack
      });
      throw error;
    }
  }

  async login(email: string, password: string) {
    logger.info('Login attempt', { email });
    
    const user = await this.findByEmail(email);
    
    if (!user) {
      logger.warn('Login failed - user not found', { email });
      throw new UnauthorizedException();
    }
    
    const isValid = await this.validatePassword(password, user.password);
    
    if (!isValid) {
      logger.warn('Login failed - invalid password', {
        email,
        userId: user.id
      });
      throw new UnauthorizedException();
    }
    
    logger.info('Login successful', {
      userId: user.id,
      email
    });
    
    return this.generateToken(user);
  }
}
```

### Request Logging Middleware
```typescript
// logging.middleware.ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';
import { logger } from './logger';

@Injectable()
export class LoggingMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    const start = Date.now();
    
    // Log request
    logger.info('Incoming request', {
      method: req.method,
      url: req.url,
      ip: req.ip,
      userAgent: req.get('user-agent')
    });
    
    // Log response
    res.on('finish', () => {
      const duration = Date.now() - start;
      
      logger.info('Request completed', {
        method: req.method,
        url: req.url,
        statusCode: res.statusCode,
        duration: `${duration}ms`
      });
    });
    
    next();
  }
}

// Apply globally
// app.module.ts
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggingMiddleware)
      .forRoutes('*');
  }
}
```

---

## Log Levels

### Standard Log Levels
```
ERROR   - Something failed
WARN    - Something unexpected but handled
INFO    - Important business events
DEBUG   - Detailed diagnostic info
TRACE   - Very detailed (function entry/exit)

Production: INFO and above (INFO, WARN, ERROR)
Development: DEBUG and above
Testing: ERROR only
```

### Usage Examples
```typescript
// ERROR - Application errors
logger.error('Database connection failed', {
  error: error.message,
  database: 'postgres',
  host: process.env.DB_HOST
});

// WARN - Unexpected but handled
logger.warn('Rate limit exceeded', {
  ip: req.ip,
  endpoint: req.url,
  attempts: rateLimitAttempts
});

// INFO - Business events
logger.info('Payment processed', {
  userId: user.id,
  amount: payment.amount,
  currency: payment.currency,
  transactionId: payment.id
});

// DEBUG - Diagnostic
logger.debug('Cache hit', {
  key: cacheKey,
  ttl: cacheTTL
});

// TRACE - Very detailed
logger.trace('Function called', {
  function: 'calculateTotal',
  args: { items, tax, discount }
});
```

---

## Structured Logging

### Why Structured Logging?
```
Unstructured (Hard to parse):
"User john@example.com created at 2025-12-28 with ID 123"

Structured (Easy to query):
{
  "message": "User created",
  "userId": 123,
  "email": "john@example.com",
  "timestamp": "2025-12-28T10:30:00Z",
  "level": "info"
}

Benefits:
- Easy to search: email="john@example.com"
- Easy to filter: level="error"
- Easy to analyze: GROUP BY userId
- Machine-readable
```

### JSON Logging
```typescript
// Configure Winston for JSON
const logger = winston.createLogger({
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.Console()
  ]
});

// Usage
logger.info('User created', {
  userId: 123,
  email: 'john@example.com',
  plan: 'premium',
  referralSource: 'google'
});

// Output:
{
  "level": "info",
  "message": "User created",
  "userId": 123,
  "email": "john@example.com",
  "plan": "premium",
  "referralSource": "google",
  "timestamp": "2025-12-28T10:30:00.000Z",
  "service": "my-app"
}
```

### Context Propagation
```typescript
// Add request ID to all logs in request lifecycle
import { v4 as uuidv4 } from 'uuid';
import { AsyncLocalStorage } from 'async_hooks';

const asyncLocalStorage = new AsyncLocalStorage();

// Middleware to set request context
app.use((req, res, next) => {
  const requestId = uuidv4();
  const context = {
    requestId,
    userId: req.user?.id,
    ip: req.ip
  };
  
  asyncLocalStorage.run(context, () => {
    next();
  });
});

// Logger wrapper
class ContextLogger {
  private getContext() {
    return asyncLocalStorage.getStore() || {};
  }
  
  info(message: string, meta: any = {}) {
    logger.info(message, {
      ...this.getContext(),
      ...meta
    });
  }
  
  error(message: string, meta: any = {}) {
    logger.error(message, {
      ...this.getContext(),
      ...meta
    });
  }
}

export const contextLogger = new ContextLogger();

// Usage - requestId automatically included
contextLogger.info('User created', { userId: 123 });
// Output includes: requestId, userId (from context), and userId (from meta)
```

---

## Log Management Services

### Popular Services
```
┌──────────────┬─────────────┬───────────────┬──────────────┐
│ Service      │ Best For    │ Pricing       │ Features     │
├──────────────┼─────────────┼───────────────┼──────────────┤
│ ELK Stack    │ Self-hosted │ Free (costs   │ Full control │
│ (Elastic)    │ Enterprise  │ infrastructure│ Powerful     │
├──────────────┼─────────────┼───────────────┼──────────────┤
│ Datadog      │ All-in-one  │ $15/host/mo   │ APM, metrics │
│              │ monitoring  │               │ logs, traces │
├──────────────┼─────────────┼───────────────┼──────────────┤
│ Loggly       │ Simple logs │ $79/mo        │ Easy setup   │
├──────────────┼─────────────┼───────────────┼──────────────┤
│ Papertrail   │ Simple logs │ Free-$175/mo  │ Simple UI    │
├──────────────┼─────────────┼───────────────┼──────────────┤
│ CloudWatch   │ AWS apps    │ $0.50/GB      │ AWS native   │
├──────────────┼─────────────┼───────────────┼──────────────┤
│ Google Cloud │ GCP apps    │ $0.50/GB      │ GCP native   │
│ Logging      │             │               │              │
└──────────────┴─────────────┴───────────────┴──────────────┘
```

### ELK Stack (Elasticsearch, Logstash, Kibana)
```yaml
# docker-compose.yml
version: '3.8'

services:
  elasticsearch:
    image: elasticsearch:8.11.0
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data

  logstash:
    image: logstash:8.11.0
    ports:
      - "5000:5000"
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    depends_on:
      - elasticsearch

  kibana:
    image: kibana:8.11.0
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch

volumes:
  elasticsearch-data:
```

```ruby
# logstash.conf
input {
  tcp {
    port => 5000
    codec => json
  }
}

filter {
  # Parse timestamp
  date {
    match => [ "timestamp", "ISO8601" ]
  }
  
  # Add geoip for IP addresses
  geoip {
    source => "ip"
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "app-logs-%{+YYYY.MM.dd}"
  }
}
```

```typescript
// Send logs to Logstash
import winston from 'winston';
import 'winston-logstash';

const logger = winston.createLogger({
  transports: [
    new winston.transports.Logstash({
      host: 'localhost',
      port: 5000
    })
  ]
});
```

### Datadog
```typescript
// Install
npm install dd-trace winston-transport-datadog

// datadog.ts
import tracer from 'dd-trace';

tracer.init({
  service: 'my-app',
  env: process.env.NODE_ENV,
  version: process.env.APP_VERSION,
  logInjection: true
});

export default tracer;

// logger.ts
import winston from 'winston';
import DatadogWinston from 'winston-transport-datadog';

const logger = winston.createLogger({
  transports: [
    new DatadogWinston({
      apiKey: process.env.DATADOG_API_KEY,
      service: 'my-app',
      ddsource: 'nodejs',
      ddtags: `env:${process.env.NODE_ENV}`
    })
  ]
});
```

### AWS CloudWatch
```typescript
// Install
npm install winston-cloudwatch

// logger.ts
import winston from 'winston';
import CloudWatchTransport from 'winston-cloudwatch';

const logger = winston.createLogger({
  transports: [
    new CloudWatchTransport({
      logGroupName: '/aws/app/my-app',
      logStreamName: `${process.env.NODE_ENV}-${Date.now()}`,
      awsRegion: 'us-east-1',
      awsAccessKeyId: process.env.AWS_ACCESS_KEY_ID,
      awsSecretKey: process.env.AWS_SECRET_ACCESS_KEY
    })
  ]
});

// Query logs with AWS CLI
aws logs filter-log-events \
  --log-group-name /aws/app/my-app \
  --filter-pattern "ERROR" \
  --start-time 1640000000000
```

---

## Application Monitoring (APM)

### What is APM?
Application Performance Monitoring - track app performance, errors, and user experience.

### Key Metrics
```
1. Response Time
   - Average: 250ms
   - P95: 500ms (95% under this)
   - P99: 1000ms

2. Throughput
   - Requests per second: 100 req/s
   - Requests per minute: 6000 req/m

3. Error Rate
   - Percentage: 0.5%
   - Count: 50 errors/hour

4. Apdex Score
   - User satisfaction metric
   - 0.0 - 1.0 (higher is better)
```

### New Relic
```bash
# Install
npm install newrelic

# Create newrelic.js
cat > newrelic.js << EOF
exports.config = {
  app_name: ['My App'],
  license_key: 'YOUR_LICENSE_KEY',
  logging: {
    level: 'info'
  }
}
EOF

# Load at app startup (must be first!)
// server.ts
require('newrelic');
import { NestFactory } from '@nestjs/core';
// ... rest of app
```

### Custom Metrics
```typescript
// Custom transaction
import newrelic from 'newrelic';

async function processPayment(payment: Payment) {
  return newrelic.startBackgroundTransaction('processPayment', async () => {
    try {
      const result = await stripeAPI.charge(payment);
      
      // Record custom metric
      newrelic.recordMetric('Custom/PaymentAmount', payment.amount);
      newrelic.recordMetric('Custom/PaymentSuccess', 1);
      
      return result;
    } catch (error) {
      newrelic.recordMetric('Custom/PaymentFailure', 1);
      newrelic.noticeError(error);
      throw error;
    }
  });
}

// Custom attributes
newrelic.addCustomAttributes({
  userId: user.id,
  plan: user.plan,
  region: user.region
});
```

---

## Error Tracking

### Sentry
```bash
# Install
npm install @sentry/node @sentry/tracing

# Initialize
// sentry.ts
import * as Sentry from '@sentry/node';
import * as Tracing from '@sentry/tracing';

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  release: process.env.APP_VERSION,
  
  // Performance monitoring
  tracesSampleRate: 1.0,
  
  // Integrations
  integrations: [
    new Sentry.Integrations.Http({ tracing: true }),
    new Tracing.Integrations.Postgres()
  ]
});

// Use in app
import * as Sentry from '@sentry/node';

try {
  await riskyOperation();
} catch (error) {
  // Capture error with context
  Sentry.captureException(error, {
    tags: {
      section: 'payment',
      method: 'processPayment'
    },
    user: {
      id: user.id,
      email: user.email
    },
    extra: {
      paymentId: payment.id,
      amount: payment.amount
    }
  });
  
  throw error;
}
```

### Error Boundaries (React)
```typescript
// ErrorBoundary.tsx
import React, { Component, ErrorInfo } from 'react';
import * as Sentry from '@sentry/react';

interface Props {
  children: React.ReactNode;
}

interface State {
  hasError: boolean;
}

class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(_: Error): State {
    return { hasError: true };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    // Log to Sentry
    Sentry.captureException(error, {
      contexts: {
        react: {
          componentStack: errorInfo.componentStack
        }
      }
    });
    
    console.error('React error:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h1>Something went wrong</h1>
          <button onClick={() => this.setState({ hasError: false })}>
            Try again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

// Usage
<ErrorBoundary>
  <App />
</ErrorBoundary>
```

### Breadcrumbs
```typescript
// Track user actions leading to error
Sentry.addBreadcrumb({
  category: 'navigation',
  message: 'User navigated to checkout',
  level: 'info'
});

Sentry.addBreadcrumb({
  category: 'user',
  message: 'User clicked payment button',
  level: 'info'
});

// When error occurs, Sentry shows all breadcrumbs
// Helps understand what user did before error
```

---

## Performance Monitoring

### Response Time Tracking
```typescript
// Middleware to track response times
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class PerformanceMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    const start = process.hrtime.bigint();
    
    res.on('finish', () => {
      const end = process.hrtime.bigint();
      const duration = Number(end - start) / 1000000; // Convert to ms
      
      // Log slow requests
      if (duration > 1000) {
        logger.warn('Slow request', {
          method: req.method,
          url: req.url,
          duration: `${duration}ms`,
          statusCode: res.statusCode
        });
      }
      
      // Send to metrics service
      metrics.timing('http.response_time', duration, {
        method: req.method,
        route: req.route?.path,
        status: res.statusCode
      });
    });
    
    next();
  }
}
```

### Database Query Monitoring
```typescript
// TypeORM logger
import { Logger } from 'typeorm';

export class DatabaseLogger implements Logger {
  logQuery(query: string, parameters?: any[]) {
    const start = Date.now();
    
    // Execute query tracking
    logger.debug('SQL Query', {
      query: query.substring(0, 100), // Truncate long queries
      parameters
    });
  }

  logQuerySlow(time: number, query: string) {
    logger.warn('Slow query detected', {
      duration: `${time}ms`,
      query: query.substring(0, 200)
    });
    
    // Alert if very slow
    if (time > 5000) {
      alerting.send('Critical: Very slow database query', {
        duration: time,
        query
      });
    }
  }
}

// Enable in TypeORM config
{
  type: 'postgres',
  logger: new DatabaseLogger(),
  maxQueryExecutionTime: 1000 // Log queries > 1s
}
```

### Frontend Performance (React)
```typescript
// Web Vitals
import { getCLS, getFID, getFCP, getLCP, getTTFB } from 'web-vitals';

function sendToAnalytics(metric: any) {
  // Send to backend
  fetch('/api/analytics', {
    method: 'POST',
    body: JSON.stringify({
      name: metric.name,
      value: metric.value,
      id: metric.id
    })
  });
}

getCLS(sendToAnalytics);  // Cumulative Layout Shift
getFID(sendToAnalytics);  // First Input Delay
getFCP(sendToAnalytics);  // First Contentful Paint
getLCP(sendToAnalytics);  // Largest Contentful Paint
getTTFB(sendToAnalytics); // Time to First Byte

// React Profiler
import { Profiler } from 'react';

function onRenderCallback(
  id: string,
  phase: 'mount' | 'update',
  actualDuration: number,
  baseDuration: number,
  startTime: number,
  commitTime: number
) {
  // Log slow renders
  if (actualDuration > 16) { // > 1 frame at 60fps
    console.warn('Slow render', {
      component: id,
      phase,
      duration: actualDuration
    });
  }
}

<Profiler id="App" onRender={onRenderCallback}>
  <App />
</Profiler>
```

---

## Infrastructure Monitoring

### Prometheus + Grafana
```yaml
# docker-compose.yml
services:
  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana-data:/var/lib/grafana
    depends_on:
      - prometheus

volumes:
  prometheus-data:
  grafana-data:
```

```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node-app'
    static_configs:
      - targets: ['app:3000']
```

```typescript
// Expose metrics endpoint
import client from 'prom-client';

// Default metrics (CPU, memory, etc.)
client.collectDefaultMetrics();

// Custom metrics
const httpRequestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code']
});

const activeUsers = new client.Gauge({
  name: 'active_users',
  help: 'Number of active users'
});

// Middleware
app.use((req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    httpRequestDuration
      .labels(req.method, req.route?.path, res.statusCode)
      .observe(duration);
  });
  
  next();
});

// Metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', client.register.contentType);
  res.end(await client.register.metrics());
});
```

### Health Checks
```typescript
// health.controller.ts
@Controller('health')
export class HealthController {
  constructor(
    private database: Connection,
    private redis: Redis
  ) {}

  @Get()
  async check() {
    const checks = {
      status: 'healthy',
      timestamp: new Date().toISOString(),
      uptime: process.uptime(),
      checks: {}
    };

    // Database check
    try {
      await this.database.query('SELECT 1');
      checks.checks.database = 'healthy';
    } catch (error) {
      checks.checks.database = 'unhealthy';
      checks.status = 'unhealthy';
    }

    // Redis check
    try {
      await this.redis.ping();
      checks.checks.redis = 'healthy';
    } catch (error) {
      checks.checks.redis = 'unhealthy';
      checks.status = 'unhealthy';
    }

    // Memory check
    const memory = process.memoryUsage();
    checks.checks.memory = {
      rss: `${Math.round(memory.rss / 1024 / 1024)}MB`,
      heapUsed: `${Math.round(memory.heapUsed / 1024 / 1024)}MB`
    };

    return checks;
  }
}
```

---

## Alerting

### Alert Rules
```
When to alert:
✅ Error rate > 1%
✅ Response time > 1s (P95)
✅ CPU > 80% for 5 min
✅ Memory > 90%
✅ Disk space < 10%
✅ Service down
✅ Database connection failed

When NOT to alert:
❌ Single error (noise)
❌ Brief spike (normal)
❌ Non-critical warning
```

### PagerDuty Integration
```typescript
// alerting.service.ts
import axios from 'axios';

export class AlertingService {
  private pagerDutyKey = process.env.PAGERDUTY_KEY;

  async sendAlert(severity: 'critical' | 'error' | 'warning', message: string, details: any) {
    // Only alert on critical/error
    if (severity === 'warning') {
      logger.warn(message, details);
      return;
    }

    try {
      await axios.post('https://events.pagerduty.com/v2/enqueue', {
        routing_key: this.pagerDutyKey,
        event_action: 'trigger',
        payload: {
          summary: message,
          severity,
          source: 'my-app',
          custom_details: details
        }
      });
    } catch (error) {
      logger.error('Failed to send alert', { error });
    }
  }
}

// Usage
if (errorRate > 0.01) {
  await alerting.sendAlert('critical', 'High error rate detected', {
    errorRate: `${errorRate * 100}%`,
    timeWindow: '5 minutes'
  });
}
```

### Slack Notifications
```typescript
// slack.service.ts
import { WebClient } from '@slack/web-api';

export class SlackService {
  private client = new WebClient(process.env.SLACK_TOKEN);
  private channel = '#alerts';

  async sendAlert(message: string, details: any) {
    await this.client.chat.postMessage({
      channel: this.channel,
      text: message,
      attachments: [{
        color: 'danger',
        fields: Object.entries(details).map(([key, value]) => ({
          title: key,
          value: String(value),
          short: true
        }))
      }]
    });
  }
}

// Usage
await slack.sendAlert('🚨 High error rate detected', {
  errorRate: '2.5%',
  affected: '150 users',
  timeWindow: '5 minutes'
});
```

---

## Interview Questions

### Basic Questions

**Q1: Why is logging important?**
```
1. Debugging
   - Understand what went wrong
   - Trace errors in production
   - Reproduce issues

2. Monitoring
   - Track application health
   - Detect issues early
   - Performance analysis

3. Security
   - Track authentication attempts
   - Audit trails
   - Detect attacks

4. Business Insights
   - User behavior
   - Feature usage
   - Conversion tracking
```

**Q2: What are log levels?**
```
ERROR:   Application failed
WARN:    Unexpected but handled
INFO:    Important events
DEBUG:   Diagnostic information
TRACE:   Very detailed info

Production: INFO, WARN, ERROR only
Development: All levels
```

**Q3: What should you NOT log?**
```
❌ Passwords
❌ API keys
❌ Credit card numbers
❌ Social security numbers
❌ Private tokens
❌ Personal data (GDPR)

Instead:
✅ Masked: "****1234"
✅ Hashed: "abc123..."
✅ Redacted: "[REDACTED]"
```

### Advanced Questions

**Q4: Explain structured logging**
```
Unstructured:
"User john@example.com logged in at 10:30"

Structured (JSON):
{
  "level": "info",
  "message": "User logged in",
  "email": "john@example.com",
  "timestamp": "2025-12-28T10:30:00Z",
  "ip": "192.168.1.1"
}

Benefits:
- Easy to query: email="john@example.com"
- Easy to filter: level="error"
- Machine-readable
- Better analytics

Tools:
- Winston (Node.js)
- Logrus (Go)
- Serilog (.NET)
```

**Q5: How do you monitor application performance?**
```
1. Response Time
   - Measure all HTTP requests
   - Track P50, P95, P99
   - Alert on slow endpoints

2. Error Rate
   - Count errors per time window
   - Alert if > threshold
   - Track by endpoint

3. Resource Usage
   - CPU percentage
   - Memory usage
   - Disk space

4. Database Performance
   - Query execution time
   - Connection pool usage
   - Slow query log

Tools:
- New Relic
- Datadog
- Prometheus + Grafana

Example:
const start = Date.now();
await handleRequest();
const duration = Date.now() - start;

metrics.timing('request.duration', duration);
if (duration > 1000) {
  logger.warn('Slow request', { duration });
}
```

**Q6: What is distributed tracing?**
```
Track request across multiple services

Example:
User Request
   │
   ▼
API Gateway (100ms)
   │
   ├─> Auth Service (50ms)
   │
   ├─> User Service (200ms)
   │   └─> Database (150ms)
   │
   └─> Payment Service (300ms)
       └─> External API (250ms)

Total: 600ms

Trace shows:
- Which service is slowest (Payment)
- Why it's slow (External API)
- Full request path
- Each service's contribution

Tools:
- Jaeger
- Zipkin
- Datadog APM
- New Relic

Implementation:
1. Generate trace ID
2. Pass in headers: X-Trace-Id
3. Each service logs with same ID
4. Visualize in UI
```

**Q7: How do you set up alerts?**
```
1. Define What to Alert On:
   - Error rate > 1%
   - Response time > 1s (P95)
   - Service down
   - Database connection failed

2. Set Thresholds:
   - Warning: 0.5% error rate
   - Critical: 1% error rate
   - Duration: 5 minutes (avoid noise)

3. Choose Channels:
   - Critical: PagerDuty (wake up engineer)
   - Error: Slack (team notification)
   - Warning: Email (check next day)

4. Avoid Alert Fatigue:
   - Don't alert on every error
   - Use time windows
   - Prioritize alerts
   - Auto-resolve when fixed

Example Alert Rule:
if (errorRate > 0.01 for 5 minutes) {
  sendToPagerDuty('High error rate');
  sendToSlack('🚨 Error rate: 2.5%');
}
```

**Q8: ELK Stack vs Datadog?**
```
ELK Stack:
Pros:
- Free and open source
- Full control
- Powerful query language
- Self-hosted

Cons:
- Complex setup
- Need to manage infrastructure
- Requires expertise
- Pay for hosting

Best for:
- Large enterprises
- On-premises requirements
- Full control needed

Datadog:
Pros:
- Easy setup (minutes)
- Fully managed
- All-in-one (logs, metrics, APM)
- Great UI

Cons:
- Expensive ($15-100/host/month)
- Vendor lock-in
- Less control

Best for:
- Startups
- Fast moving teams
- Cloud-native apps
- Want convenience
```

---

## Summary

### Monitoring Strategy
```
1. Application Logs
   - Winston/Bunyan for Node.js
   - Structured JSON format
   - Context propagation

2. Error Tracking
   - Sentry for exceptions
   - Error boundaries (React)
   - Source maps

3. Performance Monitoring
   - New Relic/Datadog APM
   - Response time tracking
   - Database query monitoring

4. Infrastructure Monitoring
   - Prometheus + Grafana
   - CPU, memory, disk
   - Container metrics

5. Alerting
   - PagerDuty for critical
   - Slack for team
   - Defined thresholds
```

### Best Practices
1. **Log Everything Important**: But not too much
2. **Structured Logging**: Use JSON format
3. **Context Matters**: Include request ID, user ID
4. **Set Up Alerts**: But avoid alert fatigue
5. **Monitor Proactively**: Don't wait for users to report
6. **Dashboard Visibility**: Make metrics accessible
7. **Regular Review**: Check logs and metrics regularly
8. **Test Monitoring**: Trigger alerts to verify they work

### Key Takeaways
- Monitoring is essential for production applications
- Use structured logging for better analysis
- Track errors, performance, and infrastructure
- Set up meaningful alerts
- Choose tools based on team size and budget
- Start simple, add complexity as needed

Good monitoring means fewer surprises in production! 📊
