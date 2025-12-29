# NestJS Performance Optimization - Top Interview Questions

## Performance Fundamentals

1. What are the key performance metrics (response time, throughput, latency)?

<details>
<summary><strong>Answer</strong></summary>

**Key performance metrics** measure application speed and efficiency: **Response Time** (total time from request to response), **Throughput** (requests handled per second), **Latency** (delay before processing starts), **Error Rate** (% of failed requests), **Resource Utilization** (CPU, memory, disk). These metrics help identify bottlenecks, ensure SLA compliance, and optimize user experience.

### **Performance Metrics Explained**

```typescript
// 1. RESPONSE TIME (Total time for request-response cycle)
// Formula: Response Time = Queue Time + Processing Time + Network Time
// Target: < 200ms (excellent), < 1s (acceptable), > 3s (poor)

// 2. THROUGHPUT (Requests per second)
// Formula: Throughput = Total Requests / Time Period
// Example: 1000 requests in 10 seconds = 100 req/s

// 3. LATENCY (Time to first byte - TTFB)
// Formula: Latency = Time when first byte received - Request sent time
// Target: < 100ms (fast), 100-300ms (average), > 500ms (slow)

// 4. ERROR RATE (Percentage of failed requests)
// Formula: Error Rate = (Failed Requests / Total Requests) × 100
// Target: < 0.1% (excellent), < 1% (acceptable), > 5% (critical)

// 5. RESOURCE UTILIZATION
// CPU: < 70% (healthy), 70-85% (busy), > 85% (critical)
// Memory: < 80% (healthy), > 90% (warning)
// Disk I/O: Monitor read/write operations per second
```

### **Measuring Response Time**

```typescript
// Using custom middleware to measure response time
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class ResponseTimeMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    const start = Date.now();

    // Capture response finish event
    res.on('finish', () => {
      const duration = Date.now() - start;
      
      // Log response time
      console.log(`${req.method} ${req.originalUrl} - ${duration}ms`);
      
      // Set response time header
      res.setHeader('X-Response-Time', `${duration}ms`);
      
      // Alert if response time is too high
      if (duration > 1000) {
        console.warn(`⚠️  Slow response: ${req.originalUrl} took ${duration}ms`);
      }
    });

    next();
  }
}

// Register in module
export class AppModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(ResponseTimeMiddleware)
      .forRoutes('*');  // Apply to all routes
  }
}
```

### **Measuring Throughput**

```typescript
// Track requests per second
import { Injectable } from '@nestjs/common';

@Injectable()
export class MetricsService {
  private requestCount = 0;
  private startTime = Date.now();
  private requestsPerSecond: number[] = [];

  trackRequest() {
    this.requestCount++;

    // Calculate throughput every second
    const elapsed = (Date.now() - this.startTime) / 1000;
    if (elapsed >= 1) {
      const rps = this.requestCount / elapsed;
      this.requestsPerSecond.push(rps);
      
      console.log(`📊 Throughput: ${rps.toFixed(2)} req/s`);
      
      // Reset counters
      this.requestCount = 0;
      this.startTime = Date.now();
    }
  }

  getAverageThroughput(): number {
    if (this.requestsPerSecond.length === 0) return 0;
    
    const sum = this.requestsPerSecond.reduce((a, b) => a + b, 0);
    return sum / this.requestsPerSecond.length;
  }

  getMetrics() {
    return {
      currentRPS: this.requestCount,
      averageRPS: this.getAverageThroughput(),
      peakRPS: Math.max(...this.requestsPerSecond, 0),
    };
  }
}

// Use in interceptor
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class MetricsInterceptor implements NestInterceptor {
  constructor(private metricsService: MetricsService) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    this.metricsService.trackRequest();
    return next.handle();
  }
}
```

### **Complete Metrics Dashboard**

```typescript
// metrics.controller.ts
import { Controller, Get } from '@nestjs/common';
import { ApiTags, ApiOperation } from '@nestjs/swagger';

interface PerformanceMetrics {
  responseTime: {
    average: number;
    min: number;
    max: number;
    p95: number;  // 95th percentile
    p99: number;  // 99th percentile
  };
  throughput: {
    current: number;  // Current req/s
    average: number;  // Average req/s
    peak: number;     // Peak req/s
  };
  errors: {
    rate: number;     // Error percentage
    total: number;    // Total errors
    last: Error[];    // Recent errors
  };
  resources: {
    cpu: number;      // CPU usage %
    memory: number;   // Memory usage MB
    heap: {
      used: number;
      total: number;
    };
  };
}

@Controller('metrics')
@ApiTags('Monitoring')
export class MetricsController {
  private responseTimes: number[] = [];
  private errorCount = 0;
  private requestCount = 0;

  @Get()
  @ApiOperation({ summary: 'Get application performance metrics' })
  getMetrics(): PerformanceMetrics {
    const memUsage = process.memoryUsage();
    
    return {
      responseTime: {
        average: this.calculateAverage(this.responseTimes),
        min: Math.min(...this.responseTimes, 0),
        max: Math.max(...this.responseTimes, 0),
        p95: this.calculatePercentile(this.responseTimes, 95),
        p99: this.calculatePercentile(this.responseTimes, 99),
      },
      throughput: {
        current: this.requestCount,
        average: this.calculateAverage([this.requestCount]),
        peak: Math.max(this.requestCount, 0),
      },
      errors: {
        rate: (this.errorCount / this.requestCount) * 100 || 0,
        total: this.errorCount,
        last: [],  // Store recent errors
      },
      resources: {
        cpu: process.cpuUsage().user / 1000000,  // Convert to seconds
        memory: memUsage.heapUsed / 1024 / 1024,  // Convert to MB
        heap: {
          used: memUsage.heapUsed / 1024 / 1024,
          total: memUsage.heapTotal / 1024 / 1024,
        },
      },
    };
  }

  private calculateAverage(values: number[]): number {
    if (values.length === 0) return 0;
    return values.reduce((a, b) => a + b, 0) / values.length;
  }

  private calculatePercentile(values: number[], percentile: number): number {
    if (values.length === 0) return 0;
    
    const sorted = [...values].sort((a, b) => a - b);
    const index = Math.ceil((percentile / 100) * sorted.length) - 1;
    return sorted[index] || 0;
  }
}
```

### **Metric Targets**

```typescript
// Production performance targets
const PERFORMANCE_TARGETS = {
  responseTime: {
    target: 200,      // Target: 200ms
    warning: 500,     // Warning: 500ms
    critical: 1000,   // Critical: 1000ms
  },
  throughput: {
    minimum: 100,     // Minimum: 100 req/s
    target: 500,      // Target: 500 req/s
    maximum: 2000,    // Maximum capacity: 2000 req/s
  },
  errorRate: {
    target: 0.1,      // Target: 0.1%
    warning: 1,       // Warning: 1%
    critical: 5,      // Critical: 5%
  },
  cpu: {
    normal: 70,       // Normal: < 70%
    warning: 85,      // Warning: 85%
    critical: 95,     // Critical: 95%
  },
  memory: {
    normal: 80,       // Normal: < 80%
    warning: 90,      // Warning: 90%
    critical: 95,     // Critical: 95%
  },
};

// Health check based on metrics
@Injectable()
export class HealthService {
  checkHealth(metrics: PerformanceMetrics): 'healthy' | 'degraded' | 'critical' {
    const { responseTime, errors, resources } = metrics;

    // Critical conditions
    if (
      responseTime.average > PERFORMANCE_TARGETS.responseTime.critical ||
      errors.rate > PERFORMANCE_TARGETS.errorRate.critical ||
      resources.cpu > PERFORMANCE_TARGETS.cpu.critical
    ) {
      return 'critical';
    }

    // Degraded conditions
    if (
      responseTime.average > PERFORMANCE_TARGETS.responseTime.warning ||
      errors.rate > PERFORMANCE_TARGETS.errorRate.warning ||
      resources.cpu > PERFORMANCE_TARGETS.cpu.warning
    ) {
      return 'degraded';
    }

    return 'healthy';
  }
}
```

**Interview Tip**: **Key metrics** are **Response Time** (request to response), **Throughput** (requests/second), **Latency** (time to first byte), **Error Rate** (% failures), **Resource Utilization** (CPU/memory). **Targets**: response time < 200ms, throughput based on capacity, error rate < 1%, CPU < 70%. **Measure** using middleware (track timing), interceptors (count requests), `/metrics` endpoint (expose stats). **Monitor**: p95/p99 percentiles (not just average - catches outliers), trends over time, resource usage. **Production**: use APM tools (New Relic, Datadog), export metrics to Prometheus, set alerts on thresholds.

</details>
2. What tools can you use to measure NestJS performance?

<details>
<summary><strong>Answer</strong></summary>

**Performance measurement tools**: **APM** (New Relic, Datadog, Dynatrace), **Node.js built-in** (process.hrtime, perf_hooks), **Monitoring** (Prometheus + Grafana), **Profiling** (Chrome DevTools, clinic.js), **Load testing** (Artillery, k6, Apache JMeter), **Logging** (Winston, Pino with metrics). Each tool serves different purposes: real-time monitoring, profiling bottlenecks, load testing capacity, or production observability.

### **1. APM (Application Performance Monitoring)**

```typescript
// New Relic integration
// npm install newrelic

// newrelic.js (in project root)
exports.config = {
  app_name: ['NestJS API'],
  license_key: process.env.NEW_RELIC_LICENSE_KEY,
  logging: {
    level: 'info',
  },
  distributed_tracing: {
    enabled: true,
  },
  transaction_tracer: {
    enabled: true,
    transaction_threshold: 'apdex_f',
  },
};

// main.ts - Load New Relic first
require('newrelic');  // Must be first line
import { NestFactory } from '@nestjs/core';

// Datadog APM
// npm install dd-trace

// main.ts
import tracer from 'dd-trace';

tracer.init({
  service: 'nestjs-api',
  env: process.env.NODE_ENV,
  version: process.env.APP_VERSION,
  logInjection: true,
});

// Auto-instruments: HTTP, database, cache, external calls
```

### **2. Prometheus + Grafana**

```typescript
// npm install @willsoto/nestjs-prometheus prom-client

import { PrometheusModule } from '@willsoto/nestjs-prometheus';
import { Module } from '@nestjs/common';

@Module({
  imports: [
    PrometheusModule.register({
      path: '/metrics',  // Expose metrics endpoint
      defaultMetrics: {
        enabled: true,  // CPU, memory, event loop
      },
    }),
  ],
})
export class AppModule {}

// Custom metrics
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';
import { Counter, Histogram, Registry } from 'prom-client';
import { InjectMetric } from '@willsoto/nestjs-prometheus';

@Injectable()
export class MetricsInterceptor implements NestInterceptor {
  constructor(
    @InjectMetric('http_requests_total') private counter: Counter,
    @InjectMetric('http_request_duration_seconds') private histogram: Histogram,
  ) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const start = Date.now();
    const req = context.switchToHttp().getRequest();

    return next.handle().pipe(
      tap({
        next: () => {
          const duration = (Date.now() - start) / 1000;
          
          // Increment request counter
          this.counter.inc({
            method: req.method,
            route: req.route?.path || req.url,
            status: '2xx',
          });

          // Record duration
          this.histogram.observe(
            { method: req.method, route: req.route?.path },
            duration,
          );
        },
        error: (error) => {
          this.counter.inc({
            method: req.method,
            route: req.route?.path,
            status: error.status || '5xx',
          });
        },
      }),
    );
  }
}

// prometheus.module.ts
import { Module } from '@nestjs/common';
import { PrometheusModule, makeCounterProvider, makeHistogramProvider } from '@willsoto/nestjs-prometheus';

@Module({
  imports: [PrometheusModule.register()],
  providers: [
    makeCounterProvider({
      name: 'http_requests_total',
      help: 'Total HTTP requests',
      labelNames: ['method', 'route', 'status'],
    }),
    makeHistogramProvider({
      name: 'http_request_duration_seconds',
      help: 'HTTP request duration',
      labelNames: ['method', 'route'],
      buckets: [0.1, 0.5, 1, 2, 5],  // Response time buckets
    }),
  ],
  exports: [PrometheusModule],
})
export class MetricsModule {}
```

### **3. Node.js Built-in Tools**

```typescript
// Using performance hooks (built-in)
import { performance, PerformanceObserver } from 'perf_hooks';

// Measure function execution time
function measurePerformance() {
  performance.mark('start');
  
  // Your code here
  someExpensiveOperation();
  
  performance.mark('end');
  performance.measure('operation', 'start', 'end');
  
  const measure = performance.getEntriesByName('operation')[0];
  console.log(`Operation took ${measure.duration}ms`);
}

// Observe all measurements
const obs = new PerformanceObserver((items) => {
  items.getEntries().forEach((entry) => {
    console.log(`${entry.name}: ${entry.duration}ms`);
  });
});
obs.observe({ entryTypes: ['measure'] });

// High-resolution time (nanosecond precision)
const start = process.hrtime.bigint();
someOperation();
const end = process.hrtime.bigint();
const duration = Number(end - start) / 1000000;  // Convert to ms
console.log(`Duration: ${duration}ms`);
```

### **4. Chrome DevTools Profiling**

```typescript
// Enable inspector in production (use with caution)
// Start app with: node --inspect=0.0.0.0:9229 dist/main.js

// Or programmatically
import * as inspector from 'inspector';

// Enable inspector on specific route (dev only)
@Controller('debug')
export class DebugController {
  @Get('profile')
  async startProfiling() {
    if (process.env.NODE_ENV === 'production') {
      throw new ForbiddenException('Profiling disabled in production');
    }

    const session = new inspector.Session();
    session.connect();

    return new Promise((resolve) => {
      session.post('Profiler.enable', () => {
        session.post('Profiler.start', () => {
          setTimeout(() => {
            session.post('Profiler.stop', (err, { profile }) => {
              resolve(profile);
              session.disconnect();
            });
          }, 10000);  // Profile for 10 seconds
        });
      });
    });
  }
}

// Connect to running app:
// 1. Open chrome://inspect
// 2. Click "Open dedicated DevTools for Node"
// 3. Go to Profiler/Memory tabs
```

### **5. Load Testing Tools**

```yaml
# Artillery load test
# npm install -g artillery
# artillery run loadtest.yml

config:
  target: 'http://localhost:3000'
  phases:
    - duration: 60  # 1 minute
      arrivalRate: 10  # 10 requests/sec
    - duration: 120  # 2 minutes
      arrivalRate: 50  # Ramp up to 50 req/sec
  processor: "./helpers.js"

scenarios:
  - name: "User flow"
    flow:
      - get:
          url: "/api/users"
          capture:
            - json: "$.token"
              as: "authToken"
      - post:
          url: "/api/posts"
          headers:
            Authorization: "Bearer {{ authToken }}"
          json:
            title: "Test Post"
      - think: 2  # Wait 2 seconds
      - get:
          url: "/api/posts/{{ postId }}"
```

```javascript
// k6 load test
// npm install -g k6
// k6 run loadtest.js

import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '30s', target: 20 },   // Ramp up to 20 users
    { duration: '1m', target: 50 },    // Stay at 50 users
    { duration: '30s', target: 0 },    // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],  // 95% of requests < 500ms
    http_req_failed: ['rate<0.01'],    // Error rate < 1%
  },
};

export default function () {
  const res = http.get('http://localhost:3000/api/users');
  
  check(res, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
  });
  
  sleep(1);
}
```

### **6. Clinic.js for Node.js Profiling**

```bash
# Install clinic.js
npm install -g clinic

# Doctor - Detect performance issues
clinic doctor -- node dist/main.js
# Visit app, generate load
# Ctrl+C to stop and generate report

# Flame - CPU profiling (flamegraphs)
clinic flame -- node dist/main.js

# Bubbleprof - Async operations analysis
clinic bubbleprof -- node dist/main.js

# Heap Profiler - Memory analysis
clinic heapprofiler -- node dist/main.js
```

### **Complete Monitoring Setup**

```typescript
// monitoring.module.ts
import { Module } from '@nestjs/common';
import { PrometheusModule } from '@willsoto/nestjs-prometheus';
import { TerminusModule } from '@nestjs/terminus';

@Module({
  imports: [
    // Prometheus metrics
    PrometheusModule.register({
      path: '/metrics',
      defaultMetrics: { enabled: true },
    }),
    
    // Health checks
    TerminusModule,
  ],
  controllers: [MetricsController, HealthController],
})
export class MonitoringModule {}

// health.controller.ts
import { Controller, Get } from '@nestjs/common';
import {
  HealthCheckService,
  HttpHealthIndicator,
  TypeOrmHealthIndicator,
  MemoryHealthIndicator,
  DiskHealthIndicator,
} from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private http: HttpHealthIndicator,
    private db: TypeOrmHealthIndicator,
    private memory: MemoryHealthIndicator,
    private disk: DiskHealthIndicator,
  ) {}

  @Get()
  check() {
    return this.health.check([
      () => this.db.pingCheck('database'),
      () => this.memory.checkHeap('memory_heap', 150 * 1024 * 1024),  // 150MB
      () => this.memory.checkRSS('memory_rss', 300 * 1024 * 1024),   // 300MB
      () => this.disk.checkStorage('disk', { path: '/', thresholdPercent: 0.9 }),
    ]);
  }
}
```

**Interview Tip**: **Tools** for performance: **APM** (New Relic, Datadog - production monitoring), **Prometheus+Grafana** (metrics collection/visualization), **Node.js built-in** (process.hrtime, perf_hooks - micro-benchmarks), **Chrome DevTools** (CPU/memory profiling), **Load testing** (Artillery, k6 - capacity testing), **Clinic.js** (Node.js profiling). **Choose based on need**: APM for production observability, load testing for capacity planning, profiling tools for bottleneck identification. **Production setup**: APM + Prometheus + health checks. **Development**: Chrome DevTools + clinic.js for deep analysis.

</details>

3. What are common performance bottlenecks?

<details>
<summary><strong>Answer</strong></summary>

**Common bottlenecks**: **Database queries** (N+1, missing indexes, slow queries), **Synchronous operations** (blocking CPU-intensive tasks, file I/O), **Memory leaks** (event listeners, closures, cache unbounded growth), **External API calls** (no timeout, no retry, sequential), **Large payloads** (no compression, no pagination), **Lack of caching** (repeated DB calls, computed data), **Unoptimized code** (inefficient algorithms, unnecessary loops). **Identify with**: profiling tools, APM, slow query logs, memory snapshots.

### **1. Database Query Bottlenecks**

```typescript
// ❌ BAD: N+1 query problem
@Injectable()
export class PostsService {
  async getAllPostsWithAuthors() {
    const posts = await this.postsRepository.find();  // 1 query
    
    // N queries (one per post) - SLOW!
    for (const post of posts) {
      post.author = await this.usersRepository.findOne({
        where: { id: post.authorId },
      });
    }
    
    return posts;
  }
}

// ✅ GOOD: Use eager loading or joins
@Injectable()
export class PostsService {
  async getAllPostsWithAuthors() {
    // Single query with JOIN
    return this.postsRepository.find({
      relations: ['author'],  // TypeORM eager loads
    });
    
    // Or with QueryBuilder for complex queries
    return this.postsRepository
      .createQueryBuilder('post')
      .leftJoinAndSelect('post.author', 'author')
      .where('post.published = :published', { published: true })
      .getMany();
  }
}

// ❌ BAD: Missing indexes on frequently queried columns
@Entity()
export class User {
  @Column()
  email: string;  // No index - slow lookups!
  
  @Column()
  username: string;  // No index
}

// ✅ GOOD: Add indexes
@Entity()
@Index(['email'])  // Composite index
export class User {
  @Column({ unique: true })
  @Index()  // Single column index
  email: string;
  
  @Column()
  @Index()
  username: string;
  
  @Column()
  @Index()
  createdAt: Date;
}

// ❌ BAD: Loading unnecessary data
async getUsers() {
  return this.usersRepository.find();  // Loads ALL columns including BLOBs
}

// ✅ GOOD: Select only needed columns
async getUsers() {
  return this.usersRepository.find({
    select: ['id', 'username', 'email'],  // Exclude large columns
  });
}
```

### **2. Synchronous/Blocking Operations**

```typescript
// ❌ BAD: Synchronous file operations block event loop
import { readFileSync } from 'fs';

@Controller('files')
export class FilesController {
  @Get(':id')
  getFile(@Param('id') id: string) {
    // BLOCKS the entire server during read!
    const data = readFileSync(`./uploads/${id}`);
    return data;
  }
}

// ✅ GOOD: Use async operations
import { readFile } from 'fs/promises';

@Controller('files')
export class FilesController {
  @Get(':id')
  async getFile(@Param('id') id: string) {
    // Non-blocking async I/O
    const data = await readFile(`./uploads/${id}`);
    return data;
  }
}

// ❌ BAD: CPU-intensive operations in request handler
@Get('compute')
async heavyComputation() {
  // Blocks event loop for 5+ seconds!
  let result = 0;
  for (let i = 0; i < 10000000000; i++) {
    result += Math.sqrt(i);
  }
  return result;
}

// ✅ GOOD: Offload to worker threads
import { Worker } from 'worker_threads';

@Get('compute')
async heavyComputation() {
  return new Promise((resolve, reject) => {
    const worker = new Worker('./computation.worker.js');
    
    worker.on('message', resolve);
    worker.on('error', reject);
    worker.on('exit', (code) => {
      if (code !== 0) reject(new Error(`Worker stopped with exit code ${code}`));
    });
  });
}

// computation.worker.js
const { parentPort } = require('worker_threads');

let result = 0;
for (let i = 0; i < 10000000000; i++) {
  result += Math.sqrt(i);
}

parentPort.postMessage(result);
```

### **3. Memory Leaks**

```typescript
// ❌ BAD: Event listeners not cleaned up
@Injectable()
export class EventsService implements OnModuleInit {
  private eventEmitter = new EventEmitter();
  
  onModuleInit() {
    // Memory leak: listeners accumulate on each request
    this.eventEmitter.on('data', (data) => {
      this.processData(data);
    });
  }
}

// ✅ GOOD: Clean up listeners
@Injectable()
export class EventsService implements OnModuleInit, OnModuleDestroy {
  private eventEmitter = new EventEmitter();
  private dataListener: (data: any) => void;
  
  onModuleInit() {
    this.dataListener = (data) => this.processData(data);
    this.eventEmitter.on('data', this.dataListener);
  }
  
  onModuleDestroy() {
    // Clean up to prevent memory leaks
    this.eventEmitter.removeListener('data', this.dataListener);
  }
}

// ❌ BAD: Unbounded cache growth
@Injectable()
export class CacheService {
  private cache = new Map<string, any>();
  
  set(key: string, value: any) {
    this.cache.set(key, value);  // Never expires - memory leak!
  }
}

// ✅ GOOD: Use LRU cache with size limit
import LRU from 'lru-cache';

@Injectable()
export class CacheService {
  private cache = new LRU({
    max: 500,  // Maximum 500 items
    maxAge: 1000 * 60 * 60,  // 1 hour TTL
  });
  
  set(key: string, value: any) {
    this.cache.set(key, value);  // Auto-evicts old entries
  }
}
```

### **4. External API Call Bottlenecks**

```typescript
// ❌ BAD: Sequential external calls
async getUserData(userId: string) {
  const profile = await this.httpService.get(`/api/profile/${userId}`).toPromise();
  const orders = await this.httpService.get(`/api/orders/${userId}`).toPromise();
  const preferences = await this.httpService.get(`/api/preferences/${userId}`).toPromise();
  
  // Total time: 300ms + 200ms + 150ms = 650ms
  return { profile, orders, preferences };
}

// ✅ GOOD: Parallel execution
import { firstValueFrom } from 'rxjs';

async getUserData(userId: string) {
  // All requests execute in parallel
  const [profile, orders, preferences] = await Promise.all([
    firstValueFrom(this.httpService.get(`/api/profile/${userId}`)),
    firstValueFrom(this.httpService.get(`/api/orders/${userId}`)),
    firstValueFrom(this.httpService.get(`/api/preferences/${userId}`)),
  ]);
  
  // Total time: max(300ms, 200ms, 150ms) = 300ms
  return { profile, orders, preferences };
}

// ❌ BAD: No timeout/retry logic
@Injectable()
export class ExternalApiService {
  async fetchData() {
    // Hangs forever if external API is slow
    return this.httpService.get('https://slow-api.com/data').toPromise();
  }
}

// ✅ GOOD: Add timeout and retry
import { HttpService } from '@nestjs/axios';
import { timeout, retry, catchError } from 'rxjs/operators';
import { firstValueFrom } from 'rxjs';

@Injectable()
export class ExternalApiService {
  constructor(private httpService: HttpService) {}
  
  async fetchData() {
    return firstValueFrom(
      this.httpService.get('https://slow-api.com/data').pipe(
        timeout(5000),  // 5 second timeout
        retry(3),       // Retry up to 3 times
        catchError((error) => {
          this.logger.error('External API failed', error);
          throw new ServiceUnavailableException('External service unavailable');
        }),
      ),
    );
  }
}
```

### **5. Large Payloads**

```typescript
// ❌ BAD: Returning huge uncompressed payloads
@Get('users')
async getAllUsers() {
  // Returns 10,000 users with all fields (100MB response)
  return this.usersRepository.find();
}

// ✅ GOOD: Implement pagination + compression
import { Query } from '@nestjs/common';

@Get('users')
async getAllUsers(
  @Query('page') page = 1,
  @Query('limit') limit = 50,
) {
  const [users, total] = await this.usersRepository.findAndCount({
    take: limit,
    skip: (page - 1) * limit,
    select: ['id', 'username', 'email'],  // Only needed fields
  });
  
  return {
    data: users,
    meta: {
      total,
      page,
      limit,
      totalPages: Math.ceil(total / limit),
    },
  };
}

// Enable compression in main.ts
import * as compression from 'compression';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.use(compression());  // Gzip compression
  await app.listen(3000);
}
```

### **6. Lack of Caching**

```typescript
// ❌ BAD: Repeatedly computing expensive data
@Get('stats')
async getStatistics() {
  // Expensive aggregation query runs on every request
  const stats = await this.ordersRepository
    .createQueryBuilder('order')
    .select('SUM(order.total)', 'totalRevenue')
    .addSelect('COUNT(order.id)', 'totalOrders')
    .addSelect('AVG(order.total)', 'averageOrder')
    .getRawOne();
  
  return stats;
}

// ✅ GOOD: Cache computed results
import { CACHE_MANAGER, Inject } from '@nestjs/common';
import { Cache } from 'cache-manager';

@Get('stats')
async getStatistics() {
  const cacheKey = 'statistics:daily';
  
  // Check cache first
  let stats = await this.cacheManager.get(cacheKey);
  
  if (!stats) {
    // Compute if not cached
    stats = await this.ordersRepository
      .createQueryBuilder('order')
      .select('SUM(order.total)', 'totalRevenue')
      .addSelect('COUNT(order.id)', 'totalOrders')
      .addSelect('AVG(order.total)', 'averageOrder')
      .getRawOne();
    
    // Cache for 1 hour
    await this.cacheManager.set(cacheKey, stats, 3600);
  }
  
  return stats;
}
```

### **Bottleneck Detection Tools**

```typescript
// Custom profiling decorator
export function Profile(threshold = 100) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = async function (...args: any[]) {
      const start = Date.now();
      
      try {
        return await originalMethod.apply(this, args);
      } finally {
        const duration = Date.now() - start;
        
        if (duration > threshold) {
          console.warn(
            `⚠️ SLOW: ${target.constructor.name}.${propertyKey} took ${duration}ms`,
          );
        }
      }
    };
    
    return descriptor;
  };
}

// Usage
@Injectable()
export class UsersService {
  @Profile(200)  // Warn if takes > 200ms
  async findUserWithOrders(userId: string) {
    return this.usersRepository.findOne({
      where: { id: userId },
      relations: ['orders', 'orders.items'],
    });
  }
}
```

**Interview Tip**: **Common bottlenecks**: **Database** (N+1 queries, missing indexes, full table scans), **Blocking I/O** (synchronous file operations, CPU-intensive tasks in event loop), **Memory** (leaks from event listeners, unbounded caches), **External APIs** (sequential calls, no timeouts), **Large data** (no pagination, no compression), **Missing cache** (repeated expensive computations). **Detection**: Use profiling (clinic.js), APM tools (New Relic), slow query logs, memory profilers. **Fix approach**: Profile → identify bottleneck → optimize (add caching/indexing/async operations). **Production**: Monitor with APM to detect bottlenecks before they impact users.

</details>

## Caching

4. Why is caching important for performance?

<details>
<summary><strong>Answer</strong></summary>

**Caching importance**: **Reduces database load** (fewer queries), **Improves response time** (in-memory access ~1ms vs DB ~50-100ms), **Handles traffic spikes** (serves cached data during high load), **Reduces external API calls** (saves cost and latency), **Lowers infrastructure costs** (fewer DB connections, smaller instances). **Trade-offs**: Stale data risk, memory usage, cache invalidation complexity. **Use for**: computed results, frequently accessed data, expensive queries, external API responses.

### **1. Performance Impact**

Without cache: 1000 req/s → 1000 DB queries (~100ms each) → High DB load

With cache (95% hit rate): 1000 req/s → 50 DB queries + 950 cache hits (~1ms) → 95% DB load reduction, avg response ~6ms

### **2. Response Time Improvement**

```typescript
// WITHOUT cache: ~100ms per request
@Injectable()
export class UserService {
  async getProfile(userId: string) {
    return this.usersRepository.findOne({
      where: { id: userId },
      relations: ['profile', 'settings'],  // DB query: ~100ms
    });
  }
}

// WITH cache: ~1ms (cached), ~100ms (miss)
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User) private usersRepository: Repository<User>,
    @Inject(CACHE_MANAGER) private cacheManager: Cache,
  ) {}

  async getProfile(userId: string) {
    const cacheKey = `user:profile:${userId}`;
    
    let profile = await this.cacheManager.get(cacheKey);  // ~1ms
    
    if (!profile) {
      profile = await this.usersRepository.findOne({
        where: { id: userId },
        relations: ['profile', 'settings'],
      });
      await this.cacheManager.set(cacheKey, profile, 600);  // 10 min TTL
    }
    
    return profile;
  }
}
// Result: 100× faster for cached data
```

### **3. Cost Reduction**

```typescript
// External API scenario: Currency exchange rates
// API cost: $0.01 per request after 1000 free

// WITHOUT cache:
// 10,000 req/day × 30 days = 300,000 requests
// Cost: (300,000 - 1,000) × $0.01 = $2,990/month

// WITH cache (1 hour TTL):
// Rates update hourly → 24 API calls/day × 30 = 720 requests
// Cost: $0 (under free tier!)
// Savings: $2,990/month

@Injectable()
export class CurrencyService {
  async getExchangeRate(from: string, to: string) {
    const cacheKey = `exchange:${from}:${to}`;
    
    let rate = await this.cacheManager.get(cacheKey);
    
    if (!rate) {
      const response = await firstValueFrom(
        this.httpService.get(`https://api.rates.com/${from}/${to}`),
      );
      rate = response.data;
      
      // Cache for 1 hour (rates don't change frequently)
      await this.cacheManager.set(cacheKey, rate, 3600);
    }
    
    return rate;
  }
}
```

### **4. Handling Traffic Spikes**

```typescript
// Black Friday sale scenario
@Injectable()
export class ProductService {
  async getFeaturedProduct(productId: string) {
    const cacheKey = `product:featured:${productId}`;
    
    let product = await this.cacheManager.get(cacheKey);
    
    if (!product) {
      product = await this.productsRepository.findOne({
        where: { id: productId },
        relations: ['images', 'reviews'],
      });
      
      await this.cacheManager.set(cacheKey, product, 900);  // 15 min
    }
    
    return product;
  }
}

// Traffic: Normal 100 req/s → Sale 10,000 req/s
// WITHOUT cache: 10,000 DB queries/s → Database overload!
// WITH cache (99% hit): 100 DB queries/s → Database survives
```

**Interview Tip**: **Why cache**: **Performance** (100× faster: memory ~1ms vs DB ~100ms), **Scalability** (handle traffic spikes, 90-99% DB load reduction), **Cost** (smaller DB instances, fewer external API calls), **Availability** (serve cached data during DB issues). **Use cases**: Frequently accessed data (user profiles, product catalogs), expensive queries (aggregations), external APIs (weather, currency), computed results (statistics). **Trade-offs**: Stale data (use appropriate TTL), memory usage (cache eviction policies), invalidation complexity. **Production**: Multi-layer caching (L1 in-memory for hot data, L2 Redis for warm data). **Metrics**: Track cache hit rate (target >90%), response time improvement.

</details>

5. How do you implement caching in NestJS using `CacheModule`?

<details>
<summary><strong>Answer</strong></summary>

**CacheModule implementation**: **Install** `@nestjs/cache-manager` + `cache-manager`, **Import** CacheModule.register() in AppModule, **Inject** CACHE_MANAGER in services, **Use** get/set/del methods. **Default**: in-memory store. **Production**: Redis store. **Global**: CacheModule.register({ isGlobal: true }). **TTL**: Set default or per-key expiration.

```typescript
// 1. Install dependencies
// npm install @nestjs/cache-manager cache-manager

// 2. Import in AppModule
import { Module } from '@nestjs/common';
import { CacheModule } from '@nestjs/cache-manager';

@Module({
  imports: [
    CacheModule.register({
      isGlobal: true,  // Available in all modules
      ttl: 300,        // Default TTL: 5 minutes (in seconds)
      max: 100,        // Maximum number of items in cache
    }),
  ],
})
export class AppModule {}

// 3. Use in Service
import { Injectable, Inject } from '@nestjs/common';
import { CACHE_MANAGER } from '@nestjs/cache-manager';
import { Cache } from 'cache-manager';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>,
    @Inject(CACHE_MANAGER)
    private cacheManager: Cache,
  ) {}

  async getUser(id: string): Promise<User> {
    const cacheKey = `user:${id}`;
    
    // Try to get from cache
    let user = await this.cacheManager.get<User>(cacheKey);
    
    if (!user) {
      // Cache miss - fetch from database
      user = await this.usersRepository.findOne({ where: { id } });
      
      // Store in cache
      await this.cacheManager.set(cacheKey, user);
    }
    
    return user;
  }

  async updateUser(id: string, updateDto: UpdateUserDto): Promise<User> {
    const user = await this.usersRepository.save({ id, ...updateDto });
    
    // Invalidate cache after update
    await this.cacheManager.del(`user:${id}`);
    
    return user;
  }
}
```

### **Advanced Configuration**

```typescript
// Custom TTL per module
@Module({
  imports: [
    CacheModule.register({
      ttl: 600,  // 10 minutes default
      max: 1000,
    }),
  ],
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}

// Async configuration (dynamic TTL from config)
import { ConfigModule, ConfigService } from '@nestjs/config';

@Module({
  imports: [
    CacheModule.registerAsync({
      imports: [ConfigModule],
      useFactory: async (configService: ConfigService) => ({
        ttl: configService.get('CACHE_TTL'),
        max: configService.get('CACHE_MAX_ITEMS'),
      }),
      inject: [ConfigService],
    }),
  ],
})
export class AppModule {}
```

### **Complete Example with Cache Patterns**

```typescript
@Injectable()
export class ProductsService {
  constructor(
    @InjectRepository(Product)
    private productsRepository: Repository<Product>,
    @Inject(CACHE_MANAGER)
    private cacheManager: Cache,
  ) {}

  // Pattern 1: Cache-aside (lazy loading)
  async getProduct(id: string): Promise<Product> {
    const cacheKey = `product:${id}`;
    
    let product = await this.cacheManager.get<Product>(cacheKey);
    
    if (!product) {
      product = await this.productsRepository.findOne({
        where: { id },
        relations: ['category', 'images'],
      });
      
      if (product) {
        await this.cacheManager.set(cacheKey, product, 600);  // 10 min TTL
      }
    }
    
    return product;
  }

  // Pattern 2: Write-through cache
  async createProduct(createDto: CreateProductDto): Promise<Product> {
    const product = await this.productsRepository.save(createDto);
    
    // Immediately cache new product
    await this.cacheManager.set(`product:${product.id}`, product, 600);
    
    return product;
  }

  // Pattern 3: Cache invalidation
  async updateProduct(id: string, updateDto: UpdateProductDto): Promise<Product> {
    const product = await this.productsRepository.save({ id, ...updateDto });
    
    // Invalidate related caches
    await this.cacheManager.del(`product:${id}`);
    await this.cacheManager.del(`products:list`);
    await this.cacheManager.del(`category:${product.categoryId}:products`);
    
    return product;
  }

  // Pattern 4: Bulk cache operations
  async getProducts(categoryId: string): Promise<Product[]> {
    const cacheKey = `category:${categoryId}:products`;
    
    let products = await this.cacheManager.get<Product[]>(cacheKey);
    
    if (!products) {
      products = await this.productsRepository.find({
        where: { categoryId },
        order: { name: 'ASC' },
      });
      
      await this.cacheManager.set(cacheKey, products, 300);  // 5 min
    }
    
    return products;
  }

  // Pattern 5: Cache with custom TTL
  async getFeaturedProducts(): Promise<Product[]> {
    const cacheKey = 'products:featured';
    
    let products = await this.cacheManager.get<Product[]>(cacheKey);
    
    if (!products) {
      products = await this.productsRepository.find({
        where: { featured: true },
        take: 10,
      });
      
      // Short TTL for frequently changing data
      await this.cacheManager.set(cacheKey, products, 60);  // 1 min
    }
    
    return products;
  }
}
```

**Interview Tip**: **CacheModule setup**: Install `@nestjs/cache-manager`, import in AppModule with `.register({ isGlobal: true, ttl: 300 })`, inject `CACHE_MANAGER` token in services. **Methods**: `get(key)` retrieves, `set(key, value, ttl?)` stores, `del(key)` invalidates, `reset()` clears all. **Patterns**: Cache-aside (check cache → miss → fetch → store), write-through (update DB → update cache), cache invalidation (update/delete → invalidate). **Production**: Use Redis store for distributed caching, set appropriate TTLs, implement cache key strategy (prefixes like `user:`, `product:`), monitor cache hit rate.

</details>

6. What is `CacheInterceptor` and how do you use it?

<details>
<parameter name="newString">
**CacheInterceptor**: **Built-in interceptor** that automatically caches HTTP responses. **Apply**: globally (app.useGlobalInterceptors), controller-level (@UseInterceptors), or route-level. **Caches**: GET requests by default (uses URL as cache key). **Configurable**: Custom cache keys, TTL, track by (headers, query params). **Automatic**: No manual cache.get/set logic needed.

```typescript
// 1. Global application (caches ALL GET routes)
import { CacheModule, CacheInterceptor } from '@nestjs/cache-manager';
import { APP_INTERCEPTOR } from '@nestjs/core';

@Module({
  imports: [
    CacheModule.register({
      ttl: 300,
      max: 100,
    }),
  ],
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: CacheInterceptor,
    },
  ],
})
export class AppModule {}

// Now all GET requests are automatically cached!

// 2. Controller-level (cache specific controller)
import { Controller, UseInterceptors } from '@nestjs/common';
import { CacheInterceptor } from '@nestjs/cache-manager';

@Controller('products')
@UseInterceptors(CacheInterceptor)  // Cache all routes in this controller
export class ProductsController {
  @Get()
  findAll() {
    // Response automatically cached
    return this.productsService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    // Each URL cached separately: /products/1, /products/2, etc.
    return this.productsService.findOne(id);
  }
}

// 3. Route-level (cache specific endpoints)
@Controller('users')
export class UsersController {
  @Get()
  @UseInterceptors(CacheInterceptor)  // Only this route cached
  findAll() {
    return this.usersService.findAll();
  }

  @Get(':id')
  @UseInterceptors(CacheInterceptor)
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(id);
  }

  @Post()
  // POST not cached (CacheInterceptor only caches GET by default)
  create(@Body() createDto: CreateUserDto) {
    return this.usersService.create(createDto);
  }
}
```

### **Custom Cache Keys**

```typescript
// Custom cache key using @CacheKey decorator
import { CacheKey, CacheTTL } from '@nestjs/cache-manager';

@Controller('products')
export class ProductsController {
  @Get()
  @UseInterceptors(CacheInterceptor)
  @CacheKey('all_products')  // Custom key instead of URL
  @CacheTTL(60)  // Override TTL: 60 seconds
  findAll() {
    return this.productsService.findAll();
  }

  @Get('featured')
  @UseInterceptors(CacheInterceptor)
  @CacheKey('featured_products')
  @CacheTTL(300)  // 5 minutes
  getFeatured() {
    return this.productsService.getFeatured();
  }
}
```

### **Custom Cache Interceptor (Advanced)**

```typescript
// Track cache by user ID or other headers
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { CacheInterceptor } from '@nestjs/cache-manager';

@Injectable()
export class HttpCacheInterceptor extends CacheInterceptor {
  // Override to create custom cache keys
  trackBy(context: ExecutionContext): string | undefined {
    const request = context.switchToHttp().getRequest();
    const { httpAdapter } = this.httpAdapterHost;

    // Only cache GET requests
    if (httpAdapter.getRequestMethod(request) !== 'GET') {
      return undefined;
    }

    // Include user ID in cache key for user-specific caching
    const userId = request.user?.id;
    const url = httpAdapter.getRequestUrl(request);
    
    // Cache key: user:123:/api/products?category=electronics
    return userId ? `user:${userId}:${url}` : url;
  }
}

// Use custom interceptor
@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: HttpCacheInterceptor,
    },
  ],
})
export class AppModule {}
```

### **Exclude Specific Routes from Caching**

```typescript
import { SetMetadata } from '@nestjs/common';

// Custom decorator to skip cache
export const SkipCache = () => SetMetadata('skipCache', true);

// Custom interceptor that respects SkipCache
@Injectable()
export class CustomCacheInterceptor extends CacheInterceptor {
  async intercept(context: ExecutionContext, next: CallHandler) {
    const skipCache = this.reflector.get<boolean>(
      'skipCache',
      context.getHandler(),
    );

    if (skipCache) {
      return next.handle();  // Skip caching
    }

    return super.intercept(context, next);
  }
}

// Usage
@Controller('products')
@UseInterceptors(CustomCacheInterceptor)
export class ProductsController {
  @Get()
  findAll() {
    // Cached
    return this.productsService.findAll();
  }

  @Get('realtime')
  @SkipCache()  // This route NOT cached
  getRealtime() {
    return this.productsService.getRealtimeData();
  }
}
```

### **Cache by Query Parameters**

```typescript
@Injectable()
export class QueryCacheInterceptor extends CacheInterceptor {
  trackBy(context: ExecutionContext): string | undefined {
    const request = context.switchToHttp().getRequest();
    const { httpAdapter } = this.httpAdapterHost;

    if (httpAdapter.getRequestMethod(request) !== 'GET') {
      return undefined;
    }

    // Include query params in cache key
    const url = httpAdapter.getRequestUrl(request);
    const queryString = new URLSearchParams(request.query).toString();
    
    // Cache key: /api/products?category=electronics&page=1
    return queryString ? `${url}?${queryString}` : url;
  }
}

// Now different query params get separate cache entries
// GET /products?category=electronics → cached separately
// GET /products?category=books → cached separately
// GET /products?page=1 → cached separately from page=2
```

### **Conditional Caching**

```typescript
@Injectable()
export class ConditionalCacheInterceptor extends CacheInterceptor {
  trackBy(context: ExecutionContext): string | undefined {
    const request = context.switchToHttp().getRequest();
    const { httpAdapter } = this.httpAdapterHost;

    if (httpAdapter.getRequestMethod(request) !== 'GET') {
      return undefined;
    }

    // Don't cache if admin user (always fresh data)
    if (request.user?.role === 'admin') {
      return undefined;
    }

    // Don't cache if 'no-cache' header present
    if (request.headers['cache-control'] === 'no-cache') {
      return undefined;
    }

    return httpAdapter.getRequestUrl(request);
  }
}
```

**Interview Tip**: **CacheInterceptor**: Built-in HTTP response caching for GET requests. **Apply**: Globally (`APP_INTERCEPTOR`), controller-level (`@UseInterceptors`), or route-level. **Automatic**: Uses URL as cache key, no manual get/set needed. **Customize**: `@CacheKey` decorator for custom keys, `@CacheTTL` for per-route TTL, extend `CacheInterceptor` to override `trackBy()` for custom logic (user-specific, query params). **Best practices**: Cache public GET endpoints, exclude admin/realtime data, include relevant params in cache key (user ID, query string), set appropriate TTLs per endpoint. **Production**: Use with Redis for distributed caching across instances.

</details>

7. What cache stores does NestJS support (in-memory, Redis)?

<details>
<summary><strong>Answer</strong></summary>

**Supported cache stores**: **In-memory** (default, cache-manager), **Redis** (cache-manager-redis-store), **Memcached** (cache-manager-memcached-store), **MongoDB** (cache-manager-mongodb), **Filesystem** (cache-manager-fs), **Custom stores**. **Production**: Use Redis for distributed caching across multiple instances. **Development**: In-memory is sufficient. **Choose based on**: scalability needs, persistence requirements, infrastructure.

### **1. In-Memory Store (Default)**

```typescript
// npm install @nestjs/cache-manager cache-manager

import { CacheModule } from '@nestjs/cache-manager';

@Module({
  imports: [
    CacheModule.register({
      isGlobal: true,
      ttl: 300,  // 5 minutes
      max: 100,  // Max 100 items
    }),
  ],
})
export class AppModule {}

// ✅ Pros: Fast, no external dependencies, simple setup
// ❌ Cons: Not shared across instances, lost on restart, limited by RAM
// Use case: Single-instance apps, development, session storage
```

### **2. Redis Store (Production)**

```typescript
// npm install @nestjs/cache-manager cache-manager cache-manager-redis-store redis

import { CacheModule } from '@nestjs/cache-manager';
import * as redisStore from 'cache-manager-redis-store';
import type { RedisClientOptions } from 'redis';

@Module({
  imports: [
    CacheModule.register<RedisClientOptions>({
      isGlobal: true,
      store: redisStore,
      host: process.env.REDIS_HOST || 'localhost',
      port: parseInt(process.env.REDIS_PORT) || 6379,
      ttl: 300,
      max: 1000,
    }),
  ],
})
export class AppModule {}

// ✅ Pros: Distributed (shared across instances), persistent, scalable, supports clustering
// ❌ Cons: Requires Redis server, slightly slower than in-memory (~1-2ms)
// Use case: Production, microservices, load-balanced applications
```

### **3. Redis with Authentication & Advanced Config**

```typescript
import { CacheModule } from '@nestjs/cache-manager';
import { redisStore } from 'cache-manager-redis-store';

@Module({
  imports: [
    CacheModule.registerAsync({
      isGlobal: true,
      useFactory: async () => ({
        store: redisStore,
        host: process.env.REDIS_HOST,
        port: process.env.REDIS_PORT,
        password: process.env.REDIS_PASSWORD,
        db: 0,  // Redis database number
        ttl: 600,
        
        // Connection options
        socket: {
          reconnectStrategy: (retries) => {
            if (retries > 10) {
              return new Error('Redis max retries exceeded');
            }
            return retries * 100;  // Exponential backoff
          },
        },
        
        // Redis-specific options
        enableReadyCheck: true,
        maxRetriesPerRequest: 3,
      }),
    }),
  ],
})
export class AppModule {}
```

### **4. Multiple Cache Stores**

```typescript
// Different stores for different purposes
import { CacheModule } from '@nestjs/cache-manager';
import * as redisStore from 'cache-manager-redis-store';

@Module({
  imports: [
    // Default in-memory cache
    CacheModule.register({
      isGlobal: true,
      ttl: 60,
    }),
    
    // Named Redis cache for sessions
    CacheModule.registerAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (config: ConfigService) => ({
        store: redisStore,
        host: config.get('REDIS_HOST'),
        port: config.get('REDIS_PORT'),
      }),
    }),
  ],
})
export class AppModule {}

// Use specific cache store
@Injectable()
export class SessionService {
  constructor(
    @Inject(CACHE_MANAGER) private defaultCache: Cache,
    @Inject('REDIS_CACHE') private redisCache: Cache,
  ) {}
  
  async setSession(key: string, value: any) {
    // Use Redis for sessions (persistent)
    await this.redisCache.set(key, value);
  }
  
  async setTempData(key: string, value: any) {
    // Use in-memory for temporary data (fast)
    await this.defaultCache.set(key, value);
  }
}
```

### **5. Memcached Store**

```typescript
// npm install cache-manager-memcached-store

import { CacheModule } from '@nestjs/cache-manager';
import * as memcachedStore from 'cache-manager-memcached-store';

@Module({
  imports: [
    CacheModule.register({
      store: memcachedStore,
      hosts: ['localhost:11211'],
      ttl: 600,
    }),
  ],
})
export class AppModule {}

// ✅ Pros: Very fast, good for large-scale caching
// ❌ Cons: No persistence, data lost on restart
// Use case: High-traffic apps needing fast cache without persistence
```

### **6. Multi-Tier Caching (L1 + L2)**

```typescript
// Combine in-memory (L1) + Redis (L2) for optimal performance
import { Injectable, Inject } from '@nestjs/common';
import { CACHE_MANAGER } from '@nestjs/cache-manager';
import { Cache } from 'cache-manager';

@Injectable()
export class MultiTierCacheService {
  // L1: In-memory cache (fastest, ~0.1ms)
  private l1Cache = new Map<string, { value: any; expires: number }>();
  
  constructor(
    // L2: Redis cache (fast, ~1-2ms, shared across instances)
    @Inject(CACHE_MANAGER) private l2Cache: Cache,
  ) {}

  async get(key: string): Promise<any> {
    // Try L1 first
    const l1Entry = this.l1Cache.get(key);
    if (l1Entry && l1Entry.expires > Date.now()) {
      return l1Entry.value;
    }
    
    // Try L2 (Redis)
    const l2Value = await this.l2Cache.get(key);
    if (l2Value) {
      // Promote to L1
      this.l1Cache.set(key, {
        value: l2Value,
        expires: Date.now() + 60000,  // 1 min in L1
      });
      return l2Value;
    }
    
    return null;
  }

  async set(key: string, value: any, ttl: number): Promise<void> {
    // Set in both layers
    this.l1Cache.set(key, {
      value,
      expires: Date.now() + Math.min(ttl * 1000, 60000),
    });
    
    await this.l2Cache.set(key, value, ttl);
  }

  async del(key: string): Promise<void> {
    // Invalidate both layers
    this.l1Cache.delete(key);
    await this.l2Cache.del(key);
  }
}

// Performance:
// L1 hit (hot data): ~0.1ms
// L2 hit (warm data): ~1-2ms
// Miss (cold data): ~100ms (DB query)
```

### **7. Custom Cache Store**

```typescript
// Implement custom store for specific needs
import { Cache, Store } from 'cache-manager';

class CustomStore implements Store {
  private data = new Map<string, any>();

  async get<T>(key: string): Promise<T | undefined> {
    return this.data.get(key);
  }

  async set(key: string, value: any, ttl?: number): Promise<void> {
    this.data.set(key, value);
    
    if (ttl) {
      setTimeout(() => this.data.delete(key), ttl * 1000);
    }
  }

  async del(key: string): Promise<void> {
    this.data.delete(key);
  }

  async reset(): Promise<void> {
    this.data.clear();
  }
}

// Use custom store
@Module({
  imports: [
    CacheModule.register({
      store: new CustomStore(),
    }),
  ],
})
export class AppModule {}
```

**Interview Tip**: **Cache stores**: **In-memory** (default, fast but not shared across instances), **Redis** (production standard, distributed, persistent), **Memcached** (fast, no persistence), **Multi-tier** (L1 in-memory + L2 Redis for optimal performance). **Production choice**: Use **Redis** for scalability (shared across instances), persistence (survives restarts), and advanced features (pub/sub, clustering). **Setup**: Install `cache-manager-redis-store`, configure in CacheModule with host/port/password. **Performance**: In-memory ~0.1ms, Redis ~1-2ms, DB ~50-100ms. **Architecture**: Single instance → in-memory, Multiple instances → Redis, High performance → multi-tier (L1+L2).

</details>

8. How do you configure Redis as a cache store?

<details>
<summary><strong>Answer</strong></summary>

**Redis configuration**: **Install** `cache-manager-redis-store` + `redis`, **Configure** in CacheModule with host/port/password, **Use** registerAsync for environment variables, **Connection options**: reconnection strategy, database selection, **Clustering**: support for Redis Cluster, **Security**: TLS/SSL, authentication. **Production**: Use connection pooling, enable ready check, set max retries.

### **1. Basic Redis Configuration**

```typescript
// npm install @nestjs/cache-manager cache-manager-redis-store redis

import { CacheModule } from '@nestjs/cache-manager';
import * as redisStore from 'cache-manager-redis-store';
import type { RedisClientOptions } from 'redis';

@Module({
  imports: [
    CacheModule.register<RedisClientOptions>({
      isGlobal: true,
      store: redisStore,
      host: 'localhost',
      port: 6379,
      ttl: 600,  // 10 minutes default
    }),
  ],
})
export class AppModule {}

// Now all cache operations use Redis
@Injectable()
export class UsersService {
  constructor(@Inject(CACHE_MANAGER) private cache: Cache) {}
  
  async getUser(id: string) {
    const cached = await this.cache.get(`user:${id}`);
    if (cached) return cached;
    
    const user = await this.usersRepo.findOne({ where: { id } });
    await this.cache.set(`user:${id}`, user);  // Stored in Redis
    return user;
  }
}
```

### **2. Environment-Based Configuration**

```typescript
// .env file
REDIS_HOST=redis.example.com
REDIS_PORT=6379
REDIS_PASSWORD=secretpassword
REDIS_DB=0
CACHE_TTL=600

// app.module.ts
import { ConfigModule, ConfigService } from '@nestjs/config';
import { CacheModule } from '@nestjs/cache-manager';
import * as redisStore from 'cache-manager-redis-store';

@Module({
  imports: [
    ConfigModule.forRoot(),
    
    CacheModule.registerAsync({
      isGlobal: true,
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: async (configService: ConfigService) => ({
        store: redisStore,
        host: configService.get('REDIS_HOST'),
        port: configService.get('REDIS_PORT'),
        password: configService.get('REDIS_PASSWORD'),
        db: configService.get('REDIS_DB') || 0,
        ttl: configService.get('CACHE_TTL'),
      }),
    }),
  ],
})
export class AppModule {}
```

### **3. Advanced Redis Configuration**

```typescript
import { CacheModule } from '@nestjs/cache-manager';
import { redisStore } from 'cache-manager-redis-store';
import { RedisClientOptions } from 'redis';

@Module({
  imports: [
    CacheModule.registerAsync<RedisClientOptions>({
      isGlobal: true,
      useFactory: async () => ({
        store: redisStore,
        
        // Connection
        host: process.env.REDIS_HOST || 'localhost',
        port: parseInt(process.env.REDIS_PORT) || 6379,
        password: process.env.REDIS_PASSWORD,
        db: 0,  // Database number (0-15)
        
        // Cache settings
        ttl: 600,  // Default TTL in seconds
        max: 10000,  // Max items in cache
        
        // Connection options
        socket: {
          // Reconnection strategy
          reconnectStrategy: (retries: number) => {
            if (retries > 20) {
              console.error('Redis: Max retries exceeded');
              return new Error('Max retries exceeded');
            }
            // Exponential backoff: 100ms, 200ms, 400ms, ...
            return Math.min(retries * 100, 3000);
          },
          
          // Connection timeout
          connectTimeout: 10000,  // 10 seconds
          
          // Keep-alive
          keepAlive: 30000,  // 30 seconds
        },
        
        // Command options
        commandsQueueMaxLength: 1000,
        enableReadyCheck: true,
        maxRetriesPerRequest: 3,
        
        // Error handling
        enableOfflineQueue: true,  // Queue commands when disconnected
      }),
    }),
  ],
})
export class AppModule {}
```

### **4. Redis with TLS/SSL (Production)**

```typescript
import * as fs from 'fs';
import { CacheModule } from '@nestjs/cache-manager';

@Module({
  imports: [
    CacheModule.registerAsync({
      useFactory: () => ({
        store: redisStore,
        host: process.env.REDIS_HOST,
        port: parseInt(process.env.REDIS_PORT),
        password: process.env.REDIS_PASSWORD,
        
        // TLS/SSL configuration
        socket: {
          tls: true,
          rejectUnauthorized: true,
          
          // Custom certificates
          ca: fs.readFileSync('/path/to/ca-cert.pem'),
          cert: fs.readFileSync('/path/to/client-cert.pem'),
          key: fs.readFileSync('/path/to/client-key.pem'),
        },
      }),
    }),
  ],
})
export class AppModule {}
```

### **5. Redis Sentinel (High Availability)**

```typescript
// Redis Sentinel for automatic failover
import { CacheModule } from '@nestjs/cache-manager';

@Module({
  imports: [
    CacheModule.registerAsync({
      useFactory: () => ({
        store: redisStore,
        
        // Sentinel configuration
        sentinels: [
          { host: 'sentinel1.example.com', port: 26379 },
          { host: 'sentinel2.example.com', port: 26379 },
          { host: 'sentinel3.example.com', port: 26379 },
        ],
        name: 'mymaster',  // Master name in Sentinel
        password: process.env.REDIS_PASSWORD,
        db: 0,
        
        // Sentinel options
        sentinelRetryStrategy: (times: number) => {
          return Math.min(times * 100, 3000);
        },
      }),
    }),
  ],
})
export class AppModule {}
```

### **6. Redis Cluster**

```typescript
// Redis Cluster for horizontal scaling
import { CacheModule } from '@nestjs/cache-manager';
import { Cluster } from 'ioredis';

@Module({
  imports: [
    CacheModule.registerAsync({
      useFactory: () => {
        const cluster = new Cluster(
          [
            { host: 'redis-node1.example.com', port: 6379 },
            { host: 'redis-node2.example.com', port: 6379 },
            { host: 'redis-node3.example.com', port: 6379 },
          ],
          {
            redisOptions: {
              password: process.env.REDIS_PASSWORD,
            },
            clusterRetryStrategy: (times) => {
              return Math.min(100 * times, 3000);
            },
            enableReadyCheck: true,
            maxRedirections: 16,
          },
        );
        
        return {
          store: redisStore,
          client: cluster,
          ttl: 600,
        };
      },
    }),
  ],
})
export class AppModule {}
```

### **7. Custom Redis Module with Health Check**

```typescript
// redis.module.ts
import { Module, OnModuleInit } from '@nestjs/common';
import { CacheModule } from '@nestjs/cache-manager';
import * as redisStore from 'cache-manager-redis-store';

@Module({
  imports: [
    CacheModule.registerAsync({
      isGlobal: true,
      useFactory: () => ({
        store: redisStore,
        host: process.env.REDIS_HOST,
        port: process.env.REDIS_PORT,
        password: process.env.REDIS_PASSWORD,
        ttl: 600,
        
        socket: {
          reconnectStrategy: (retries) => {
            console.log(`Redis reconnection attempt ${retries}`);
            return retries * 100;
          },
        },
      }),
    }),
  ],
})
export class RedisModule implements OnModuleInit {
  constructor(@Inject(CACHE_MANAGER) private cache: Cache) {}
  
  async onModuleInit() {
    try {
      // Test Redis connection
      await this.cache.set('health:check', 'ok', 10);
      const value = await this.cache.get('health:check');
      
      if (value === 'ok') {
        console.log('✅ Redis connection successful');
      }
    } catch (error) {
      console.error('❌ Redis connection failed:', error);
      // Optionally: throw error to prevent app startup
    }
  }
}

// Health check endpoint
import { Controller, Get } from '@nestjs/common';
import { HealthCheck, HealthCheckService } from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    @Inject(CACHE_MANAGER) private cache: Cache,
  ) {}

  @Get('redis')
  @HealthCheck()
  async checkRedis() {
    try {
      await this.cache.set('health:redis', Date.now(), 10);
      const value = await this.cache.get('health:redis');
      
      return {
        status: 'ok',
        redis: {
          status: value ? 'up' : 'down',
          latency: Date.now() - value,
        },
      };
    } catch (error) {
      return {
        status: 'error',
        redis: { status: 'down', error: error.message },
      };
    }
  }
}
```

### **8. Multiple Redis Instances**

```typescript
// Different Redis instances for different purposes
import { CacheModule } from '@nestjs/cache-manager';

@Module({
  imports: [
    // Cache instance (db 0)
    CacheModule.registerAsync({
      useFactory: () => ({
        store: redisStore,
        host: 'localhost',
        port: 6379,
        db: 0,  // Database 0 for cache
        ttl: 600,
      }),
    }),
    
    // Session instance (db 1)
    CacheModule.registerAsync({
      useFactory: () => ({
        store: redisStore,
        host: 'localhost',
        port: 6379,
        db: 1,  // Database 1 for sessions
        ttl: 86400,  // 24 hours
      }),
    }),
  ],
})
export class AppModule {}
```

**Interview Tip**: **Redis setup**: Install `cache-manager-redis-store`, configure in `CacheModule.registerAsync()` with host/port/password from env vars. **Production config**: Enable reconnection strategy (exponential backoff), set connection timeout, use TLS for security, configure sentinel for HA or cluster for scaling. **Connection**: Use `socket.reconnectStrategy` for auto-reconnect, `enableReadyCheck` to verify connection, `maxRetriesPerRequest` for resilience. **Health check**: Test connection on startup, expose health endpoint. **Multiple instances**: Use different `db` numbers (0-15) or separate Redis servers. **Performance**: Connection pooling automatic, monitor latency with health checks.

</details>

9. How do you set cache TTL (Time To Live)?

<details>
<summary><strong>Answer</strong></summary>

**TTL configuration**: **Global default** in CacheModule.register({ ttl: 300 }), **Per-operation** cache.set(key, value, ttl), **Per-route** @CacheTTL(seconds) decorator, **Infinite** ttl: 0 (no expiration), **Strategy**: Short TTL for frequently changing data, long TTL for static data. **Units**: Seconds. **Production**: Use appropriate TTLs based on data volatility.

### **1. Global Default TTL**

```typescript
// Set default TTL for all cache operations
import { CacheModule } from '@nestjs/cache-manager';

@Module({
  imports: [
    CacheModule.register({
      isGlobal: true,
      ttl: 300,  // 5 minutes (in seconds) for all operations
      max: 1000,
    }),
  ],
})
export class AppModule {}

// All cache.set() operations use 300 seconds by default
@Injectable()
export class UsersService {
  async getUser(id: string) {
    await this.cache.set(`user:${id}`, user);  // Uses 300s TTL
  }
}
```

### **2. Per-Operation TTL**

```typescript
// Override TTL for specific cache operations
@Injectable()
export class ProductsService {
  constructor(@Inject(CACHE_MANAGER) private cache: Cache) {}

  async getFeaturedProducts() {
    const cached = await this.cache.get('products:featured');
    if (cached) return cached;
    
    const products = await this.productsRepo.find({ featured: true });
    
    // Short TTL: 1 minute (frequently updated)
    await this.cache.set('products:featured', products, 60);
    return products;
  }

  async getCategories() {
    const cached = await this.cache.get('categories');
    if (cached) return cached;
    
    const categories = await this.categoriesRepo.find();
    
    // Long TTL: 1 hour (rarely changes)
    await this.cache.set('categories', categories, 3600);
    return categories;
  }

  async getStaticContent() {
    const cached = await this.cache.get('static:footer');
    if (cached) return cached;
    
    const content = await this.contentRepo.findOne({ key: 'footer' });
    
    // Very long TTL: 24 hours (static content)
    await this.cache.set('static:footer', content, 86400);
    return content;
  }

  async getSessionData(sessionId: string) {
    const session = await this.sessionsRepo.findOne({ id: sessionId });
    
    // No expiration (ttl: 0 or undefined)
    await this.cache.set(`session:${sessionId}`, session, 0);
    return session;
  }
}
```

### **3. Route-Level TTL with @CacheTTL Decorator**

```typescript
import { CacheTTL, CacheKey, UseInterceptors } from '@nestjs/common';
import { CacheInterceptor } from '@nestjs/cache-manager';

@Controller('products')
@UseInterceptors(CacheInterceptor)
export class ProductsController {
  // Default TTL (from global config: 300s)
  @Get()
  findAll() {
    return this.productsService.findAll();
  }

  // Custom TTL: 1 minute
  @Get('trending')
  @CacheTTL(60)
  getTrending() {
    return this.productsService.getTrending();
  }

  // Custom TTL: 10 minutes
  @Get('categories')
  @CacheTTL(600)
  getCategories() {
    return this.productsService.getCategories();
  }

  // Custom TTL: 1 hour
  @Get('static')
  @CacheTTL(3600)
  getStaticData() {
    return this.productsService.getStaticData();
  }

  // Infinite cache (no expiration)
  @Get('permanent')
  @CacheTTL(0)
  getPermanentData() {
    return this.productsService.getPermanentData();
  }
}
```

### **4. Dynamic TTL Based on Data**

```typescript
// Calculate TTL based on data characteristics
@Injectable()
export class SmartCacheService {
  constructor(@Inject(CACHE_MANAGER) private cache: Cache) {}

  async cacheProduct(product: Product) {
    let ttl: number;
    
    // Dynamic TTL based on product stock
    if (product.stock < 10) {
      ttl = 60;  // 1 minute (low stock, check frequently)
    } else if (product.stock < 100) {
      ttl = 300;  // 5 minutes (medium stock)
    } else {
      ttl = 3600;  // 1 hour (high stock, stable)
    }
    
    await this.cache.set(`product:${product.id}`, product, ttl);
  }

  async cacheWithPopularity(key: string, data: any, views: number) {
    // Popular content = longer cache
    const ttl = views > 1000 ? 3600 : 300;
    await this.cache.set(key, data, ttl);
  }

  async cacheWithFreshness(key: string, data: any, updatedAt: Date) {
    // Recently updated = shorter cache
    const ageInHours = (Date.now() - updatedAt.getTime()) / (1000 * 60 * 60);
    
    if (ageInHours < 1) {
      await this.cache.set(key, data, 60);  // 1 min (fresh data)
    } else if (ageInHours < 24) {
      await this.cache.set(key, data, 600);  // 10 min (recent data)
    } else {
      await this.cache.set(key, data, 3600);  // 1 hour (old data)
    }
  }
}
```

### **5. TTL Strategy by Use Case**

```typescript
@Injectable()
export class CacheStrategyService {
  constructor(@Inject(CACHE_MANAGER) private cache: Cache) {}

  // Real-time data: Very short TTL
  async cacheStockPrice(symbol: string, price: number) {
    await this.cache.set(`stock:${symbol}`, price, 5);  // 5 seconds
  }

  // Frequently updated: Short TTL
  async cacheUserProfile(userId: string, profile: any) {
    await this.cache.set(`user:${userId}`, profile, 60);  // 1 minute
  }

  // Moderately volatile: Medium TTL
  async cacheProductList(categoryId: string, products: any[]) {
    await this.cache.set(`products:${categoryId}`, products, 300);  // 5 minutes
  }

  // Rarely changes: Long TTL
  async cacheConfiguration(config: any) {
    await this.cache.set('app:config', config, 3600);  // 1 hour
  }

  // Static content: Very long TTL
  async cacheStaticPage(slug: string, content: any) {
    await this.cache.set(`page:${slug}`, content, 86400);  // 24 hours
  }

  // Reference data: Infinite (manual invalidation)
  async cacheCountries(countries: any[]) {
    await this.cache.set('ref:countries', countries, 0);  // No expiration
  }

  // External API: Match API rate limits
  async cacheExternalApi(endpoint: string, data: any) {
    // If API updates every 15 minutes, cache for 15 minutes
    await this.cache.set(`api:${endpoint}`, data, 900);  // 15 minutes
  }
}
```

### **6. Environment-Based TTL**

```typescript
// Different TTL for dev/staging/production
import { ConfigService } from '@nestjs/config';

@Module({
  imports: [
    CacheModule.registerAsync({
      inject: [ConfigService],
      useFactory: (config: ConfigService) => ({
        ttl: config.get('NODE_ENV') === 'production' ? 600 : 60,
        // Production: 10 minutes, Dev: 1 minute
      }),
    }),
  ],
})
export class AppModule {}

// Or per-service
@Injectable()
export class CacheService {
  private readonly ttl: number;
  
  constructor(
    @Inject(CACHE_MANAGER) private cache: Cache,
    private config: ConfigService,
  ) {
    // Shorter TTL in development for faster testing
    this.ttl = this.config.get('NODE_ENV') === 'production' ? 600 : 30;
  }

  async set(key: string, value: any) {
    await this.cache.set(key, value, this.ttl);
  }
}
```

### **7. TTL with Sliding Expiration**

```typescript
// Extend TTL on each access (sliding window)
@Injectable()
export class SlidingCacheService {
  constructor(@Inject(CACHE_MANAGER) private cache: Cache) {}

  async getWithSlidingTTL(key: string, ttl: number = 300) {
    const value = await this.cache.get(key);
    
    if (value) {
      // Reset TTL on access (sliding expiration)
      await this.cache.set(key, value, ttl);
    }
    
    return value;
  }

  // Use case: User sessions (extend on activity)
  async getSession(sessionId: string) {
    const session = await this.getWithSlidingTTL(
      `session:${sessionId}`,
      1800,  // 30 minutes, refreshed on each request
    );
    return session;
  }
}
```

### **8. TTL Constants (Best Practice)**

```typescript
// Define TTL constants for consistency
export enum CacheTTL {
  REALTIME = 5,        // 5 seconds
  SHORT = 60,          // 1 minute
  MEDIUM = 300,        // 5 minutes
  LONG = 3600,         // 1 hour
  VERY_LONG = 86400,   // 24 hours
  INFINITE = 0,        // No expiration
}

@Injectable()
export class ProductsService {
  async getTrending() {
    const cached = await this.cache.get('trending');
    if (cached) return cached;
    
    const products = await this.productsRepo.getTrending();
    await this.cache.set('trending', products, CacheTTL.SHORT);  // Readable
    return products;
  }

  async getCategories() {
    const cached = await this.cache.get('categories');
    if (cached) return cached;
    
    const categories = await this.categoriesRepo.find();
    await this.cache.set('categories', categories, CacheTTL.LONG);
    return categories;
  }
}
```

**Interview Tip**: **TTL levels**: **Global** (CacheModule.register({ ttl: 300 })), **Per-operation** (cache.set(key, value, 60)), **Per-route** (@CacheTTL(300) decorator). **Strategy**: Realtime data = 5-30s, Frequently updated = 1-5 min, Moderate = 5-30 min, Rarely changes = 1-24 hours, Static = days or infinite (ttl: 0). **Dynamic TTL**: Calculate based on data characteristics (stock levels, popularity, freshness). **Best practices**: Use TTL constants for consistency, shorter TTL in dev for testing, sliding expiration for sessions (reset on access), match external API update frequency. **Production**: Monitor cache hit rates, adjust TTLs based on metrics.

</details>

10. How do you invalidate cache?

<details>
<summary><strong>Answer</strong></summary>

**Cache invalidation methods**: **Single key** (cache.del(key)), **Multiple keys** (Promise.all with del), **Pattern-based** (Redis SCAN + DEL), **Reset all** (cache.reset()), **Time-based** (TTL expiration), **Event-driven** (invalidate on create/update/delete). **Strategies**: Invalidate on write, tag-based invalidation, version prefixes. **Production**: Use cache tags, implement invalidation service, log invalidations for debugging.

### **1. Single Key Invalidation**

```typescript
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User) private usersRepo: Repository<User>,
    @Inject(CACHE_MANAGER) private cache: Cache,
  ) {}

  async updateUser(id: string, updateDto: UpdateUserDto) {
    const user = await this.usersRepo.save({ id, ...updateDto });
    
    // Invalidate specific user cache
    await this.cache.del(`user:${id}`);
    
    return user;
  }

  async deleteUser(id: string) {
    await this.usersRepo.delete(id);
    
    // Invalidate cache after deletion
    await this.cache.del(`user:${id}`);
  }
}
```

### **2. Multiple Related Keys Invalidation**

```typescript
@Injectable()
export class ProductsService {
  async updateProduct(id: string, updateDto: UpdateProductDto) {
    const product = await this.productsRepo.save({ id, ...updateDto });
    
    // Invalidate all related caches
    await Promise.all([
      this.cache.del(`product:${id}`),
      this.cache.del(`products:list`),
      this.cache.del(`products:category:${product.categoryId}`),
      this.cache.del(`products:featured`),
      this.cache.del(`search:results:${product.name}`),
    ]);
    
    return product;
  }

  async createProduct(createDto: CreateProductDto) {
    const product = await this.productsRepo.save(createDto);
    
    // Invalidate list caches when new product added
    await Promise.all([
      this.cache.del('products:list'),
      this.cache.del(`products:category:${product.categoryId}`),
      this.cache.del('products:count'),
    ]);
    
    return product;
  }
}
```

### **3. Pattern-Based Invalidation (Redis)**

```typescript
// Invalidate all keys matching a pattern
import { Injectable, Inject } from '@nestjs/common';
import { CACHE_MANAGER } from '@nestjs/cache-manager';
import { Cache } from 'cache-manager';
import { Redis } from 'ioredis';

@Injectable()
export class CacheInvalidationService {
  private redisClient: Redis;
  
  constructor(@Inject(CACHE_MANAGER) private cache: Cache) {
    // Access underlying Redis client
    this.redisClient = (this.cache.store as any).getClient();
  }

  // Delete all keys matching pattern
  async invalidatePattern(pattern: string): Promise<number> {
    return new Promise((resolve, reject) => {
      const stream = this.redisClient.scanStream({
        match: pattern,
        count: 100,
      });

      let deletedCount = 0;
      const pipeline = this.redisClient.pipeline();

      stream.on('data', (keys: string[]) => {
        if (keys.length) {
          keys.forEach((key) => pipeline.del(key));
          deletedCount += keys.length;
        }
      });

      stream.on('end', async () => {
        await pipeline.exec();
        resolve(deletedCount);
      });

      stream.on('error', reject);
    });
  }

  // Invalidate all user-related caches
  async invalidateUserCaches(userId: string) {
    await this.invalidatePattern(`user:${userId}:*`);
    // Matches: user:123:profile, user:123:orders, user:123:settings, etc.
  }

  // Invalidate all product caches for a category
  async invalidateCategoryCaches(categoryId: string) {
    await this.invalidatePattern(`products:category:${categoryId}:*`);
  }

  // Invalidate all search result caches
  async invalidateSearchCaches() {
    await this.invalidatePattern('search:*');
  }
}

// Usage
@Injectable()
export class UsersService {
  constructor(
    private cacheInvalidation: CacheInvalidationService,
  ) {}

  async updateUser(userId: string, updateDto: UpdateUserDto) {
    const user = await this.usersRepo.save({ id: userId, ...updateDto });
    
    // Invalidate all user-related caches at once
    await this.cacheInvalidation.invalidateUserCaches(userId);
    
    return user;
  }
}
```

### **4. Tag-Based Invalidation**

```typescript
// Implement cache tags for grouping related caches
@Injectable()
export class TaggedCacheService {
  private tagIndex = new Map<string, Set<string>>();
  
  constructor(@Inject(CACHE_MANAGER) private cache: Cache) {}

  async set(key: string, value: any, ttl: number, tags: string[] = []) {
    // Store data in cache
    await this.cache.set(key, value, ttl);
    
    // Index tags
    tags.forEach((tag) => {
      if (!this.tagIndex.has(tag)) {
        this.tagIndex.set(tag, new Set());
      }
      this.tagIndex.get(tag).add(key);
    });
  }

  async invalidateTag(tag: string) {
    const keys = this.tagIndex.get(tag);
    
    if (keys) {
      // Delete all keys with this tag
      await Promise.all(
        Array.from(keys).map((key) => this.cache.del(key)),
      );
      
      // Clean up tag index
      this.tagIndex.delete(tag);
    }
  }

  async invalidateTags(tags: string[]) {
    await Promise.all(tags.map((tag) => this.invalidateTag(tag)));
  }
}

// Usage with tags
@Injectable()
export class ProductsService {
  constructor(private taggedCache: TaggedCacheService) {}

  async getProduct(id: string) {
    const product = await this.productsRepo.findOne({ where: { id } });
    
    // Cache with tags
    await this.taggedCache.set(
      `product:${id}`,
      product,
      600,
      ['products', `category:${product.categoryId}`, 'inventory'],
    );
    
    return product;
  }

  async updateCategory(categoryId: string) {
    // Invalidate all products in this category
    await this.taggedCache.invalidateTag(`category:${categoryId}`);
  }

  async updateInventory() {
    // Invalidate all inventory-related caches
    await this.taggedCache.invalidateTag('inventory');
  }
}
```

### **5. Event-Driven Invalidation**

```typescript
// Automatic cache invalidation using events
import { Injectable, OnModuleInit } from '@nestjs/common';
import { EventEmitter2, OnEvent } from '@nestjs/event-emitter';

@Injectable()
export class CacheEventsService implements OnModuleInit {
  constructor(
    @Inject(CACHE_MANAGER) private cache: Cache,
    private eventEmitter: EventEmitter2,
  ) {}

  onModuleInit() {
    // Listen for cache invalidation events
    this.eventEmitter.on('cache.invalidate', this.handleInvalidation.bind(this));
  }

  private async handleInvalidation(payload: { keys: string[] }) {
    await Promise.all(
      payload.keys.map((key) => this.cache.del(key)),
    );
  }

  @OnEvent('user.updated')
  async handleUserUpdated(payload: { userId: string }) {
    await this.cache.del(`user:${payload.userId}`);
    await this.cache.del(`user:${payload.userId}:profile`);
  }

  @OnEvent('product.created')
  async handleProductCreated(payload: { categoryId: string }) {
    await this.cache.del('products:list');
    await this.cache.del(`products:category:${payload.categoryId}`);
  }

  @OnEvent('product.deleted')
  async handleProductDeleted(payload: { productId: string; categoryId: string }) {
    await Promise.all([
      this.cache.del(`product:${payload.productId}`),
      this.cache.del('products:list'),
      this.cache.del(`products:category:${payload.categoryId}`),
    ]);
  }
}

// Emit events in services
@Injectable()
export class ProductsService {
  constructor(private eventEmitter: EventEmitter2) {}

  async updateProduct(id: string, updateDto: UpdateProductDto) {
    const product = await this.productsRepo.save({ id, ...updateDto });
    
    // Emit event (cache automatically invalidated)
    this.eventEmitter.emit('product.updated', { productId: id });
    
    return product;
  }
}
```

### **6. Version-Based Cache Keys**

```typescript
// Use version in cache keys for easy invalidation
@Injectable()
export class VersionedCacheService {
  private version = 1;
  
  constructor(@Inject(CACHE_MANAGER) private cache: Cache) {}

  private getVersionedKey(key: string): string {
    return `v${this.version}:${key}`;
  }

  async get(key: string) {
    return this.cache.get(this.getVersionedKey(key));
  }

  async set(key: string, value: any, ttl?: number) {
    return this.cache.set(this.getVersionedKey(key), value, ttl);
  }

  // Invalidate ALL caches by incrementing version
  async invalidateAll() {
    this.version++;
    // All old versioned keys become inaccessible
    // Will be cleaned up by TTL expiration
  }
}

// Usage
@Injectable()
export class ProductsService {
  constructor(private versionedCache: VersionedCacheService) {}

  async getProducts() {
    // Cached with current version: v1:products:list
    let products = await this.versionedCache.get('products:list');
    
    if (!products) {
      products = await this.productsRepo.find();
      await this.versionedCache.set('products:list', products, 600);
    }
    
    return products;
  }

  async performMajorUpdate() {
    // Invalidate all caches by bumping version
    await this.versionedCache.invalidateAll();
    // Now all caches use v2:* keys, old v1:* keys ignored
  }
}
```

### **7. Scheduled Cache Cleanup**

```typescript
// Periodic cache cleanup using cron
import { Injectable } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';

@Injectable()
export class CacheCleanupService {
  constructor(
    @Inject(CACHE_MANAGER) private cache: Cache,
    private cacheInvalidation: CacheInvalidationService,
  ) {}

  // Clear all caches at midnight
  @Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT)
  async dailyCleanup() {
    await this.cache.reset();
    console.log('Daily cache cleanup completed');
  }

  // Clear specific caches every hour
  @Cron(CronExpression.EVERY_HOUR)
  async hourlyCleanup() {
    await Promise.all([
      this.cacheInvalidation.invalidatePattern('temp:*'),
      this.cacheInvalidation.invalidatePattern('session:expired:*'),
    ]);
  }

  // Clear old search results every 15 minutes
  @Cron(CronExpression.EVERY_15_MINUTES)
  async cleanupSearchCaches() {
    await this.cacheInvalidation.invalidatePattern('search:*');
  }
}
```

### **8. Cache Invalidation with Database Triggers**

```typescript
// Listen to database changes and invalidate cache
import { Injectable, OnModuleInit } from '@nestjs/common';
import { Connection } from 'typeorm';

@Injectable()
export class DatabaseCacheSync implements OnModuleInit {
  constructor(
    private connection: Connection,
    @Inject(CACHE_MANAGER) private cache: Cache,
  ) {}

  async onModuleInit() {
    // Listen to TypeORM events
    this.connection.subscribers.push({
      afterUpdate: async (event) => {
        const entity = event.entity;
        const entityName = event.metadata.name.toLowerCase();
        
        // Invalidate cache for updated entity
        await this.cache.del(`${entityName}:${entity.id}`);
      },
      
      afterRemove: async (event) => {
        const entity = event.entity;
        const entityName = event.metadata.name.toLowerCase();
        
        // Invalidate cache for deleted entity
        await this.cache.del(`${entityName}:${entity.id}`);
      },
    });
  }
}
```

**Interview Tip**: **Invalidation methods**: **Single** (cache.del(key)), **Multiple** (Promise.all with del), **Pattern** (Redis SCAN + DEL for wildcards like `user:*`), **All** (cache.reset()). **Strategies**: **Write-through** (invalidate on update/delete), **Event-driven** (emit events, listeners invalidate), **Tag-based** (group related caches, invalidate by tag), **Version-based** (increment version, old keys ignored). **Production**: Implement CacheInvalidationService with pattern matching, use tags for related data, log invalidations, schedule cleanup jobs. **Common patterns**: Invalidate parent+child (product → category list), invalidate on CUD operations, use Redis pub/sub for distributed invalidation across instances. **Performance**: Batch invalidations with Promise.all, use SCAN not KEYS in production.

</details>

11. What should you cache and what should you not cache?

<details>
<summary><strong>Answer</strong></summary>

**CACHE**: **Read-heavy data** (product catalogs, user profiles), **Expensive queries** (aggregations, joins), **External API responses** (weather, currency rates), **Computed results** (statistics, reports), **Static content** (pages, configurations). **DON'T CACHE**: **Sensitive data** (passwords, credit cards), **Real-time data** (stock prices, live scores), **User-specific private data** (unless secure), **Large objects** (>1MB), **Frequently changing data** (no benefit). **Rule**: Cache if read >> writes and staleness acceptable.

### **1. What TO Cache**

```typescript
// ✅ GOOD: Frequently accessed, rarely changed
@Injectable()
export class GoodCachingExamples {
  constructor(@Inject(CACHE_MANAGER) private cache: Cache) {}

  // 1. Product catalog (read-heavy, rarely changes)
  async getProducts() {
    const cached = await this.cache.get('products:list');
    if (cached) return cached;
    
    const products = await this.productsRepo.find();
    await this.cache.set('products:list', products, 600);  // 10 min
    return products;
  }

  // 2. User profiles (read >> writes)
  async getUserProfile(userId: string) {
    const cached = await this.cache.get(`user:${userId}:profile`);
    if (cached) return cached;
    
    const profile = await this.usersRepo.findOne({
      where: { id: userId },
      relations: ['settings', 'preferences'],
    });
    
    await this.cache.set(`user:${userId}:profile`, profile, 300);
    return profile;
  }

  // 3. Expensive aggregations/statistics
  async getDashboardStats() {
    const cached = await this.cache.get('dashboard:stats');
    if (cached) return cached;
    
    // Complex query: 3-5 seconds
    const stats = await this.ordersRepo
      .createQueryBuilder('order')
      .select('COUNT(*)', 'totalOrders')
      .addSelect('SUM(total)', 'revenue')
      .addSelect('AVG(total)', 'avgOrder')
      .where('createdAt > :date', { date: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000) })
      .getRawOne();
    
    await this.cache.set('dashboard:stats', stats, 600);  // 10 min
    return stats;
  }

  // 4. External API responses (costly, rate-limited)
  async getWeather(city: string) {
    const cached = await this.cache.get(`weather:${city}`);
    if (cached) return cached;
    
    const weather = await this.weatherApi.fetch(city);
    await this.cache.set(`weather:${city}`, weather, 1800);  // 30 min
    return weather;
  }

  // 5. Reference/lookup data (rarely changes)
  async getCountries() {
    const cached = await this.cache.get('ref:countries');
    if (cached) return cached;
    
    const countries = await this.countriesRepo.find();
    await this.cache.set('ref:countries', countries, 86400);  // 24 hours
    return countries;
  }

  // 6. Configuration/settings (static)
  async getAppConfig() {
    const cached = await this.cache.get('config:app');
    if (cached) return cached;
    
    const config = await this.configRepo.findOne({ where: { key: 'app' } });
    await this.cache.set('config:app', config, 3600);  // 1 hour
    return config;
  }

  // 7. Search results (repeated queries)
  async searchProducts(query: string) {
    const cacheKey = `search:${query.toLowerCase()}`;
    const cached = await this.cache.get(cacheKey);
    if (cached) return cached;
    
    const results = await this.productsRepo
      .createQueryBuilder('product')
      .where('product.name ILIKE :query', { query: `%${query}%` })
      .getMany();
    
    await this.cache.set(cacheKey, results, 300);  // 5 min
    return results;
  }

  // 8. Computed/derived data (expensive to calculate)
  async getProductRecommendations(userId: string) {
    const cached = await this.cache.get(`recommendations:${userId}`);
    if (cached) return cached;
    
    // Complex ML algorithm
    const recommendations = await this.mlService.computeRecommendations(userId);
    await this.cache.set(`recommendations:${userId}`, recommendations, 1800);
    return recommendations;
  }
}
```

### **2. What NOT to Cache**

```typescript
// ❌ BAD: Do NOT cache these
@Injectable()
export class BadCachingExamples {
  // 1. ❌ Sensitive data (security risk)
  async getPassword(userId: string) {
    // NEVER cache passwords, tokens, credit cards
    return this.usersRepo.findOne({ where: { id: userId }, select: ['password'] });
  }

  async getCreditCard(userId: string) {
    // NEVER cache payment information
    return this.paymentRepo.findOne({ where: { userId } });
  }

  // 2. ❌ Real-time data (stale cache defeats purpose)
  async getStockPrice(symbol: string) {
    // Don't cache - needs real-time updates
    return this.stockApi.getCurrentPrice(symbol);
  }

  async getLiveScore(gameId: string) {
    // Don't cache - changes every second
    return this.sportsApi.getLiveScore(gameId);
  }

  // 3. ❌ Frequently changing data (cache miss rate high)
  async getAvailableSeats(flightId: string) {
    // Changes with every booking - cache ineffective
    return this.flightsRepo.findOne({ where: { id: flightId }, select: ['availableSeats'] });
  }

  // 4. ❌ Large objects (memory waste)
  async getVideoFile(id: string) {
    // Don't cache large files (>1MB) in memory
    // Use CDN or streaming instead
    return this.videosRepo.findOne({ where: { id } });
  }

  // 5. ❌ User-specific private data (unless properly isolated)
  async getUserOrders(userId: string) {
    // Be careful - ensure cache keys include userId
    // Risk of leaking data between users
    return this.ordersRepo.find({ where: { userId } });
  }

  // 6. ❌ Data with strict consistency requirements
  async getAccountBalance(accountId: string) {
    // Financial data - must be accurate, don't cache
    return this.accountsRepo.findOne({ where: { id: accountId } });
  }

  // 7. ❌ One-time data (no reuse)
  async generateOTP(userId: string) {
    // Generated once, used once - no benefit from caching
    return this.otpService.generate(userId);
  }
}
```

### **3. Conditional Caching Strategy**

```typescript
// Smart caching decisions based on data characteristics
@Injectable()
export class SmartCachingService {
  constructor(@Inject(CACHE_MANAGER) private cache: Cache) {}

  async getProduct(id: string, userRole: string) {
    // Don't cache for admins (need fresh data)
    if (userRole === 'admin') {
      return this.productsRepo.findOne({ where: { id } });
    }
    
    // Cache for regular users
    const cached = await this.cache.get(`product:${id}`);
    if (cached) return cached;
    
    const product = await this.productsRepo.findOne({ where: { id } });
    await this.cache.set(`product:${id}`, product, 600);
    return product;
  }

  async getInventory(productId: string) {
    const product = await this.productsRepo.findOne({ where: { id: productId } });
    
    // Only cache if stock is high (stable)
    if (product.stock > 100) {
      const cached = await this.cache.get(`inventory:${productId}`);
      if (cached) return cached;
      
      await this.cache.set(`inventory:${productId}`, product.stock, 3600);
      return product.stock;
    }
    
    // Don't cache low stock (changes frequently)
    return product.stock;
  }

  async getData(key: string, size: number) {
    // Only cache small objects
    if (size > 1024 * 1024) {  // > 1MB
      return this.fetchLargeData(key);
    }
    
    const cached = await this.cache.get(key);
    if (cached) return cached;
    
    const data = await this.fetchData(key);
    await this.cache.set(key, data, 600);
    return data;
  }
}
```

### **4. Cache Decision Matrix**

```typescript
// Decision framework for caching
export class CacheDecisionHelper {
  shouldCache(data: any, metadata: CacheMetadata): boolean {
    // Check multiple criteria
    return (
      this.isReadHeavy(metadata) &&
      !this.isSensitive(data) &&
      !this.isRealTime(metadata) &&
      this.isAppropriateSize(data) &&
      this.hasSufficientTTL(metadata)
    );
  }

  private isReadHeavy(metadata: CacheMetadata): boolean {
    // Cache if read:write ratio > 10:1
    return metadata.readCount / metadata.writeCount > 10;
  }

  private isSensitive(data: any): boolean {
    // Don't cache sensitive fields
    const sensitiveFields = ['password', 'ssn', 'creditCard', 'token'];
    return Object.keys(data).some((key) => 
      sensitiveFields.includes(key.toLowerCase())
    );
  }

  private isRealTime(metadata: CacheMetadata): boolean {
    // Don't cache if requires real-time updates
    return metadata.updateFrequency < 60;  // Updates < 1 min
  }

  private isAppropriateSize(data: any): boolean {
    const size = JSON.stringify(data).length;
    return size < 1024 * 1024;  // < 1MB
  }

  private hasSufficientTTL(metadata: CacheMetadata): boolean {
    // Only cache if staleness acceptable
    return metadata.maxStalenessTolerance > 60;  // > 1 min stale OK
  }
}
```

### **5. Best Practices Summary**

```typescript
@Injectable()
export class CachingBestPractices {
  // ✅ DO Cache:
  // 1. Read-heavy data (products, categories, users)
  // 2. Expensive queries (aggregations, complex joins)
  // 3. External API responses (weather, exchange rates)
  // 4. Static/reference data (countries, settings)
  // 5. Computed results (recommendations, statistics)
  // 6. Search results (repeated queries)
  
  // ❌ DON'T Cache:
  // 1. Sensitive data (passwords, tokens, payment info)
  // 2. Real-time data (stock prices, live scores)
  // 3. Frequently changing data (inventory with low stock)
  // 4. Large objects (videos, large files > 1MB)
  // 5. User-specific private data (unless properly secured)
  // 6. Critical financial data (account balances)
  // 7. One-time use data (OTPs, nonces)
  
  // ⚠️ Cache with Caution:
  // 1. User-specific data (ensure proper key isolation)
  // 2. Paginated results (cache per page)
  // 3. Filtered/sorted data (include filters in cache key)
  // 4. Data with complex invalidation logic
}
```

**Interview Tip**: **CACHE**: **Read-heavy** data (products, users - read:write >10:1), **expensive operations** (complex queries, aggregations, external APIs), **static content** (configurations, reference data), **computed results** (ML recommendations, statistics). **DON'T CACHE**: **Sensitive** data (passwords, tokens, payment info), **real-time** data (stock prices, live scores), **frequently changing** data (low inventory), **large objects** (>1MB - use CDN instead), **strict consistency** requirements (financial balances). **Decision factors**: Read frequency, data size, staleness tolerance, security sensitivity, update frequency. **Production**: Monitor cache hit rate (target >90%), measure performance improvement, log cache misses to identify bad caching decisions. **Rule of thumb**: Cache if read:write >10:1 AND staleness acceptable AND not sensitive.

</details>

## Database Optimization

12. What are N+1 query problems and how do you prevent them?

<details>
<summary><strong>Answer</strong></summary>

**N+1 problem**: **One query** to get main entities + **N additional queries** to get related data for each entity. **Example**: Fetch 100 posts (1 query) + fetch author for each post (100 queries) = 101 total queries. **Prevent with**: **Eager loading** (relations), **JOIN queries** (QueryBuilder), **DataLoader** (batching). **Performance**: 101 queries → 1 query (100× faster). **Production**: Always use eager loading or joins for related data.

### **1. The N+1 Problem Explained**

```typescript
// ❌ BAD: N+1 Query Problem
@Injectable()
export class PostsService {
  async getAllPostsWithAuthors() {
    // Query 1: Fetch all posts
    const posts = await this.postsRepository.find();  // 100 posts
    
    // Queries 2-101: Fetch author for EACH post (N queries)
    for (const post of posts) {
      post.author = await this.usersRepository.findOne({
        where: { id: post.authorId },
      });
    }
    
    return posts;
    // Result: 101 queries total (1 + 100)
    // Performance: ~10 seconds (100ms per query × 101)
  }
}

// Real query log:
// SELECT * FROM posts;  (1 query - returns 100 posts)
// SELECT * FROM users WHERE id = 1;  (Query 2)
// SELECT * FROM users WHERE id = 2;  (Query 3)
// SELECT * FROM users WHERE id = 3;  (Query 4)
// ... 97 more queries
// SELECT * FROM users WHERE id = 100;  (Query 101)
```

### **2. Solution 1: Eager Loading with Relations**

```typescript
// ✅ GOOD: Use eager loading (single query with JOIN)
@Injectable()
export class PostsService {
  async getAllPostsWithAuthors() {
    // Single query with JOIN
    const posts = await this.postsRepository.find({
      relations: ['author'],  // TypeORM eager loads
    });
    
    return posts;
    // Result: 1 query total
    // Performance: ~100ms
    // Improvement: 100× faster!
  }
}

// SQL generated:
// SELECT posts.*, users.* 
// FROM posts 
// LEFT JOIN users ON posts.author_id = users.id;

// Multiple relations
async getPostsWithAuthorAndComments() {
  return this.postsRepository.find({
    relations: ['author', 'comments', 'comments.user'],
  });
  // Joins: posts ← author, posts ← comments ← user
}
```

### **3. Solution 2: QueryBuilder with Joins**

```typescript
// ✅ GOOD: Use QueryBuilder for complex queries
@Injectable()
export class PostsService {
  async getPostsWithDetails() {
    return this.postsRepository
      .createQueryBuilder('post')
      .leftJoinAndSelect('post.author', 'author')  // Load author
      .leftJoinAndSelect('post.comments', 'comments')  // Load comments
      .leftJoinAndSelect('comments.user', 'commentUser')  // Load comment authors
      .where('post.published = :published', { published: true })
      .orderBy('post.createdAt', 'DESC')
      .getMany();
    
    // Result: 1 query with multiple JOINs
    // Loads posts + authors + comments + comment users in single query
  }

  // Selective loading (only needed fields)
  async getPostsSummary() {
    return this.postsRepository
      .createQueryBuilder('post')
      .leftJoinAndSelect('post.author', 'author')
      .select([
        'post.id',
        'post.title',
        'post.createdAt',
        'author.id',
        'author.name',  // Only load needed author fields
      ])
      .getMany();
  }
}
```

### **4. Solution 3: DataLoader (Facebook Pattern)**

```typescript
// npm install dataloader

import DataLoader from 'dataloader';

@Injectable()
export class UsersService {
  private userLoader: DataLoader<string, User>;

  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>,
  ) {
    // Create DataLoader that batches user fetches
    this.userLoader = new DataLoader(async (userIds: string[]) => {
      // Single query for all user IDs
      const users = await this.usersRepository.findByIds(userIds);
      
      // Return users in same order as requested IDs
      return userIds.map((id) => users.find((user) => user.id === id));
    });
  }

  async loadUser(userId: string): Promise<User> {
    return this.userLoader.load(userId);
  }
}

// Usage in posts service
@Injectable()
export class PostsService {
  constructor(private usersService: UsersService) {}

  async getAllPostsWithAuthors() {
    const posts = await this.postsRepository.find();
    
    // DataLoader batches all these requests into 1 query
    await Promise.all(
      posts.map(async (post) => {
        post.author = await this.usersService.loadUser(post.authorId);
      }),
    );
    
    return posts;
    // Result: 2 queries (1 for posts, 1 batched for all users)
  }
}
```

### **5. Real-World Example: E-commerce Orders**

```typescript
// ❌ BAD: N+1 in orders with products
@Injectable()
export class OrdersService {
  async getOrders() {
    const orders = await this.ordersRepository.find();  // Query 1
    
    for (const order of orders) {
      // N queries for order items
      order.items = await this.orderItemsRepository.find({
        where: { orderId: order.id },
      });
      
      // N×M queries for products (nested N+1!)
      for (const item of order.items) {
        item.product = await this.productsRepository.findOne({
          where: { id: item.productId },
        });
      }
    }
    
    return orders;
    // 100 orders × 5 items avg = 600+ queries!
  }
}

// ✅ GOOD: Single query with nested joins
@Injectable()
export class OrdersService {
  async getOrders() {
    return this.ordersRepository
      .createQueryBuilder('order')
      .leftJoinAndSelect('order.items', 'items')
      .leftJoinAndSelect('items.product', 'product')
      .leftJoinAndSelect('order.customer', 'customer')
      .getMany();
    
    // Result: 1 query
    // SQL: SELECT orders.*, items.*, products.*, customers.*
    //      FROM orders
    //      LEFT JOIN order_items AS items ON ...
    //      LEFT JOIN products ON ...
    //      LEFT JOIN customers ON ...
  }
}
```

### **6. Detecting N+1 Problems**

```typescript
// Query logging interceptor to detect N+1
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class QueryCountInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const startQueries = this.getQueryCount();
    
    return next.handle().pipe(
      tap(() => {
        const endQueries = this.getQueryCount();
        const queryCount = endQueries - startQueries;
        
        // Warn if too many queries
        if (queryCount > 10) {
          console.warn(
            `⚠️ Potential N+1: ${queryCount} queries in single request`,
            { route: context.getHandler().name },
          );
        }
      }),
    );
  }

  private getQueryCount(): number {
    // Hook into TypeORM query counter
    // Implementation depends on ORM
    return 0;
  }
}

// Enable query logging in development
// ormconfig.ts
export default {
  type: 'postgres',
  logging: ['query', 'error'],  // Log all queries
  logger: 'advanced-console',
  // ...
};
```

### **7. Pagination with N+1 Prevention**

```typescript
// Paginated results without N+1
@Injectable()
export class PostsService {
  async getPaginatedPosts(page: number, limit: number) {
    return this.postsRepository
      .createQueryBuilder('post')
      .leftJoinAndSelect('post.author', 'author')
      .leftJoinAndSelect('post.category', 'category')
      .skip((page - 1) * limit)
      .take(limit)
      .getManyAndCount();
    
    // Returns: [posts[], totalCount]
    // Single query with JOIN + COUNT
  }
}
```

### **8. Performance Comparison**

```typescript
// Performance metrics

// ❌ N+1 Problem:
// - 101 queries (1 + 100)
// - Time: 10,100ms (100ms per query × 101)
// - DB connections: High usage
// - Network roundtrips: 101

// ✅ With Eager Loading:
// - 1 query (with JOIN)
// - Time: 150ms (single complex query)
// - DB connections: Minimal
// - Network roundtrips: 1
// - Improvement: 67× faster!

// ✅ With DataLoader:
// - 2 queries (posts + batched users)
// - Time: 200ms
// - DB connections: Low
// - Network roundtrips: 2
// - Improvement: 50× faster!
```

**Interview Tip**: **N+1 problem**: Fetching parent entities (1 query) then fetching related data for each in a loop (N queries) = 1+N queries. **Example**: 100 posts + 100 author queries = 101 queries taking ~10s instead of 1 query in ~150ms. **Solutions**: **Eager loading** (relations: ['author']), **QueryBuilder** with leftJoinAndSelect, **DataLoader** for batching. **Detection**: Enable query logging, count queries per request, use APM tools. **Best practice**: Always load related data with JOINs, never fetch relations in loops. **Performance**: Can improve 50-100× by eliminating N+1. **Production**: Monitor query counts, set alerts for >10 queries per request.

</details>

13. What is eager loading vs lazy loading in TypeORM?

<details>
<summary><strong>Answer</strong></summary>

**Eager loading**: Automatically loads relations with main entity in **single query** using JOINs. **Set with**: `@OneToMany({ eager: true })` or `relations: ['author']` option. **Lazy loading**: Loads relations **on-demand** when accessed (separate query). **Set with**: `Promise<User>` type on relation property. **Performance**: Eager = 1 query (fast, prevents N+1), Lazy = N+1 queries (slow). **Production**: Prefer eager loading or explicit joins. TypeORM v0.3+ disabled lazy loading by default.

### **1. Eager Loading (Automatic JOINs)**

```typescript
// Define eager loading in entity
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne, OneToMany } from 'typeorm';

@Entity()
export class Post {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  title: string;

  // Eager loading: Always loads author with post
  @ManyToOne(() => User, { eager: true })
  author: User;

  // Eager loading: Always loads comments
  @OneToMany(() => Comment, (comment) => comment.post, { eager: true })
  comments: Comment[];
}

// Usage - author and comments loaded automatically
@Injectable()
export class PostsService {
  async getPost(id: string) {
    // Single query with JOINs (no N+1!)
    const post = await this.postsRepository.findOne({ where: { id } });
    
    // author and comments already loaded
    console.log(post.author.name);  // No additional query
    console.log(post.comments.length);  // No additional query
    
    return post;
  }
}

// SQL generated:
// SELECT posts.*, users.*, comments.* 
// FROM posts
// LEFT JOIN users ON posts.author_id = users.id
// LEFT JOIN comments ON comments.post_id = posts.id
// WHERE posts.id = $1;
```

### **2. Lazy Loading (On-Demand)**

```typescript
// Define lazy loading in entity (TypeORM v0.2)
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne } from 'typeorm';

@Entity()
export class Post {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  title: string;

  // Lazy loading: Use Promise<User> type
  @ManyToOne(() => User)
  author: Promise<User>;

  @OneToMany(() => Comment, (comment) => comment.post)
  comments: Promise<Comment[]>;
}

// Usage - relations loaded when accessed
@Injectable()
export class PostsService {
  async getPost(id: string) {
    // Query 1: Fetch post only
    const post = await this.postsRepository.findOne({ where: { id } });
    
    // Query 2: Fetch author (when accessed)
    const author = await post.author;
    console.log(author.name);
    
    // Query 3: Fetch comments (when accessed)
    const comments = await post.comments;
    console.log(comments.length);
    
    return post;
    // Result: 3 queries total (N+1 problem if used in loop!)
  }
}

// ❌ Problem: N+1 with lazy loading in loops
async getAllPosts() {
  const posts = await this.postsRepository.find();  // Query 1
  
  for (const post of posts) {
    const author = await post.author;  // N queries!
    console.log(author.name);
  }
}
```

### **3. Explicit Loading (Best Practice)**

```typescript
// Prefer explicit loading over eager/lazy
@Entity()
export class Post {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  title: string;

  // No eager: true - load explicitly when needed
  @ManyToOne(() => User)
  author: User;

  @OneToMany(() => Comment, (comment) => comment.post)
  comments: Comment[];
}

@Injectable()
export class PostsService {
  // Load with relations option
  async getPostWithAuthor(id: string) {
    return this.postsRepository.findOne({
      where: { id },
      relations: ['author'],  // Explicit eager loading
    });
    // Result: 1 query with JOIN
  }

  // Load only post (no relations)
  async getPostOnly(id: string) {
    return this.postsRepository.findOne({ where: { id } });
    // Result: 1 query, no JOINs
  }

  // Load different relations based on need
  async getPostWithComments(id: string) {
    return this.postsRepository.findOne({
      where: { id },
      relations: ['comments', 'comments.user'],
    });
  }

  async getPostWithEverything(id: string) {
    return this.postsRepository.findOne({
      where: { id },
      relations: ['author', 'comments', 'category', 'tags'],
    });
  }
}
```

### **4. QueryBuilder for Complex Loading**

```typescript
// Use QueryBuilder for fine control
@Injectable()
export class PostsService {
  // Selective field loading with relations
  async getPostSummary(id: string) {
    return this.postsRepository
      .createQueryBuilder('post')
      .leftJoinAndSelect('post.author', 'author')
      .select([
        'post.id',
        'post.title',
        'post.createdAt',
        'author.id',
        'author.name',  // Only load author name, not entire user object
      ])
      .where('post.id = :id', { id })
      .getOne();
  }

  // Conditional loading
  async getPost(id: string, includeComments: boolean = false) {
    const query = this.postsRepository
      .createQueryBuilder('post')
      .leftJoinAndSelect('post.author', 'author')
      .where('post.id = :id', { id });

    if (includeComments) {
      query.leftJoinAndSelect('post.comments', 'comments');
    }

    return query.getOne();
  }

  // Nested relations
  async getPostWithNestedRelations(id: string) {
    return this.postsRepository
      .createQueryBuilder('post')
      .leftJoinAndSelect('post.author', 'author')
      .leftJoinAndSelect('author.profile', 'profile')  // Nested: author's profile
      .leftJoinAndSelect('post.comments', 'comments')
      .leftJoinAndSelect('comments.user', 'commentUser')  // Nested: comment authors
      .where('post.id = :id', { id })
      .getOne();
  }
}
```

### **5. Performance Comparison**

```typescript
// Scenario: Fetch 100 posts with authors

// ❌ Lazy Loading (N+1 problem)
async getPostsLazy() {
  const posts = await this.postsRepository.find();  // Query 1
  
  for (const post of posts) {
    const author = await post.author;  // Queries 2-101
    console.log(author.name);
  }
  // Result: 101 queries, ~10 seconds
}

// ✅ Eager Loading (entity-level)
// Entity: @ManyToOne(() => User, { eager: true })
async getPostsEager() {
  const posts = await this.postsRepository.find();
  // Result: 1 query with JOIN, ~150ms
  // But: ALWAYS loads author, even if not needed
}

// ✅ Explicit Loading (best)
async getPostsExplicit() {
  const posts = await this.postsRepository.find({
    relations: ['author'],
  });
  // Result: 1 query with JOIN, ~150ms
  // Benefit: Only loads when specified
}

// ✅ QueryBuilder (most flexible)
async getPostsQueryBuilder() {
  return this.postsRepository
    .createQueryBuilder('post')
    .leftJoinAndSelect('post.author', 'author')
    .getMany();
  // Result: 1 query with JOIN, ~150ms
  // Benefit: Full control over query
}
```

### **6. Eager Loading Gotchas**

```typescript
// ⚠️ Problem: Eager loading applies to ALL queries
@Entity()
export class Post {
  @ManyToOne(() => User, { eager: true })
  author: User;

  @OneToMany(() => Comment, (comment) => comment.post, { eager: true })
  comments: Comment[];  // Always loaded, even if not needed!
}

// This loads author + comments even if you only need the post
async getPostTitle(id: string) {
  const post = await this.postsRepository.findOne({ where: { id } });
  return post.title;
  // Still executes: SELECT posts.*, users.*, comments.* ... (unnecessary!)
}

// ✅ Solution: Don't use eager: true, use explicit loading
@Entity()
export class Post {
  @ManyToOne(() => User)
  author: User;  // No eager: true

  @OneToMany(() => Comment, (comment) => comment.post)
  comments: Comment[];
}

// Now you control what's loaded
async getPostTitle(id: string) {
  const post = await this.postsRepository.findOne({ where: { id } });
  return post.title;
  // Only: SELECT posts.* FROM posts WHERE id = $1
}

async getPostWithAuthor(id: string) {
  return this.postsRepository.findOne({
    where: { id },
    relations: ['author'],  // Load author only when needed
  });
}
```

### **7. Circular Eager Relations**

```typescript
// ⚠️ Problem: Circular eager loading causes infinite loops
@Entity()
export class User {
  @OneToMany(() => Post, (post) => post.author, { eager: true })
  posts: Post[];  // Eager load posts
}

@Entity()
export class Post {
  @ManyToOne(() => User, (user) => user.posts, { eager: true })
  author: User;  // Eager load author
}

// Fetching user loads posts, which load authors, which load posts, ...
// Result: Stack overflow!

// ✅ Solution: Only one side should be eager
@Entity()
export class User {
  @OneToMany(() => Post, (post) => post.author)  // Not eager
  posts: Post[];
}

@Entity()
export class Post {
  @ManyToOne(() => User, { eager: true })  // Eager OK here
  author: User;
}
```

### **8. Best Practices**

```typescript
// ✅ Best Practices Summary

// 1. Avoid entity-level eager: true (inflexible)
@Entity()
export class Post {
  @ManyToOne(() => User)  // No eager: true
  author: User;
}

// 2. Use explicit relations option
async getPost(id: string) {
  return this.postsRepository.findOne({
    where: { id },
    relations: ['author', 'comments'],  // Explicit
  });
}

// 3. Use QueryBuilder for complex queries
async getFilteredPosts() {
  return this.postsRepository
    .createQueryBuilder('post')
    .leftJoinAndSelect('post.author', 'author')
    .where('post.published = :published', { published: true })
    .getMany();
}

// 4. Never use lazy loading in production
// TypeORM v0.3+ disabled it by default

// 5. Load only needed fields
async getPostSummary(id: string) {
  return this.postsRepository
    .createQueryBuilder('post')
    .leftJoinAndSelect('post.author', 'author')
    .select(['post.id', 'post.title', 'author.name'])
    .where('post.id = :id', { id })
    .getOne();
}
```

**Interview Tip**: **Eager loading**: Automatically loads relations with main entity in single query using JOINs. Set with `{ eager: true }` on entity or `relations: []` in find options. **Lazy loading**: Loads relations on-demand (separate queries when accessed), causes N+1 problems. **Best practice**: Avoid both - use **explicit loading** with `relations` option or QueryBuilder for full control. **Performance**: Eager = 1 query (fast), Lazy = N queries (slow). **TypeORM v0.3+**: Lazy loading disabled by default. **Production**: Use QueryBuilder with leftJoinAndSelect for complex queries, load only needed fields with select(), monitor query counts.

</details>

14. How do you use database indexing to improve performance?

<details>
<summary><strong>Answer</strong></summary>

**Database indexes**: **Speed up queries** by creating search structures for frequently queried columns. **Types**: **Single-column** (@Index()), **Composite** (@Index(['col1', 'col2'])), **Unique** (@Column({ unique: true })), **Full-text** (for text search). **Performance**: Without index = full table scan (O(n)), With index = binary search (O(log n)). **Trade-offs**: Faster reads, slower writes, storage overhead. **Use on**: WHERE clauses, JOINs, ORDER BY, foreign keys.

### **1. Basic Index Creation**

```typescript
// Single column indexes
import { Entity, PrimaryGeneratedColumn, Column, Index } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  // Index on frequently queried column
  @Column()
  @Index()  // Creates: CREATE INDEX IDX_user_email ON users(email)
  email: string;

  @Column()
  @Index()  // Index for searching by username
  username: string;

  // Unique index (enforces uniqueness + speeds up lookups)
  @Column({ unique: true })  // Auto-creates unique index
  ssn: string;

  @Column()
  @Index()  // Index for filtering by status
  status: string;  // 'active', 'inactive', 'banned'

  @Column()
  @Index()  // Index for sorting/filtering by date
  createdAt: Date;
}

// Query performance:
// Without index: SELECT * FROM users WHERE email = 'user@example.com'
//                Full table scan: O(n) - checks every row
//                1M rows = ~1000ms

// With index:    Uses index lookup: O(log n)
//                1M rows = ~5ms (200x faster!)
```

### **2. Composite Indexes**

```typescript
// Multiple columns together (order matters!)
@Entity()
@Index(['status', 'createdAt'])  // Composite index for status + date queries
@Index(['userId', 'createdAt'])  // For user's posts sorted by date
export class Post {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  title: string;

  @Column()
  userId: string;

  @Column()
  status: string;  // 'draft', 'published', 'archived'

  @Column()
  createdAt: Date;
}

// ✅ Optimized queries using composite indexes:

// Uses index: ['status', 'createdAt']
SELECT * FROM posts 
WHERE status = 'published' 
ORDER BY createdAt DESC;
// Result: Fast (uses index)

// Uses index: ['userId', 'createdAt']
SELECT * FROM posts 
WHERE userId = '123' 
ORDER BY createdAt DESC;
// Result: Fast (uses index)

// ⚠️ Index order matters!
// Index: ['status', 'createdAt']
// - Works for: WHERE status = 'X'
// - Works for: WHERE status = 'X' AND createdAt > '...'
// - DOESN'T work for: WHERE createdAt > '...' (status not in WHERE)
//   (Needs to match left-most columns first)
```

### **3. Named Indexes**

```typescript
// Custom index names for better organization
@Entity()
@Index('IDX_user_email_status', ['email', 'status'])
@Index('IDX_user_created', ['createdAt'])
export class User {
  @Column()
  email: string;

  @Column()
  status: string;

  @Column()
  createdAt: Date;
}

// Benefits of named indexes:
// - Easier to identify in database
// - Easier to drop/recreate specific indexes
// - Better for migrations
```

### **4. Partial/Conditional Indexes (PostgreSQL)**

```typescript
// Index only subset of rows (saves space, faster)
import { Entity, Column, Index } from 'typeorm';

@Entity()
export class Order {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  status: string;

  @Column()
  createdAt: Date;
}

// Create partial index via migration
export class CreatePartialIndexes1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    // Index only active orders (not archived)
    await queryRunner.query(`
      CREATE INDEX IDX_orders_active 
      ON orders (createdAt) 
      WHERE status IN ('pending', 'processing', 'shipped')
    `);
    
    // Index only recent orders (last 30 days)
    await queryRunner.query(`
      CREATE INDEX IDX_orders_recent 
      ON orders (status, createdAt) 
      WHERE createdAt > NOW() - INTERVAL '30 days'
    `);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP INDEX IDX_orders_active`);
    await queryRunner.query(`DROP INDEX IDX_orders_recent`);
  }
}

// Benefits:
// - Smaller index size (only indexes subset of rows)
// - Faster index lookups
// - Useful for status-based queries (e.g., only index active records)
```

### **5. Full-Text Search Indexes**

```typescript
// For text search (PostgreSQL)
@Entity()
export class Article {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  title: string;

  @Column('text')
  content: string;
}

// Create full-text index via migration
export class CreateFullTextIndex1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    // Add tsvector column
    await queryRunner.query(`
      ALTER TABLE article 
      ADD COLUMN search_vector tsvector
    `);

    // Create GIN index for full-text search
    await queryRunner.query(`
      CREATE INDEX IDX_article_search 
      ON article USING GIN(search_vector)
    `);

    // Auto-update search_vector on changes
    await queryRunner.query(`
      CREATE TRIGGER article_search_update 
      BEFORE INSERT OR UPDATE ON article
      FOR EACH ROW EXECUTE FUNCTION
      tsvector_update_trigger(search_vector, 'pg_catalog.english', title, content)
    `);
  }
}

// Usage in service
@Injectable()
export class ArticlesService {
  async searchArticles(query: string) {
    return this.articlesRepository
      .createQueryBuilder('article')
      .where(`article.search_vector @@ plainto_tsquery('english', :query)`, { query })
      .orderBy(`ts_rank(article.search_vector, plainto_tsquery('english', :query))`, 'DESC')
      .getMany();
    // Fast full-text search with relevance ranking
  }
}
```

### **6. Index Performance Analysis**

```typescript
// Check if indexes are being used
@Injectable()
export class PerformanceService {
  constructor(private dataSource: DataSource) {}

  async analyzeQuery(query: string) {
    // EXPLAIN shows query execution plan
    const result = await this.dataSource.query(`EXPLAIN ANALYZE ${query}`);
    
    console.log(result);
    // Look for:
    // - "Index Scan" = Good (using index)
    // - "Seq Scan" = Bad (full table scan, needs index)
    // - Execution time
  }

  // Example: Check if email queries use index
  async checkEmailIndexUsage() {
    const result = await this.dataSource.query(`
      EXPLAIN ANALYZE 
      SELECT * FROM users WHERE email = 'test@example.com'
    `);
    
    // Expected: Index Scan using IDX_user_email
    // If shows "Seq Scan" → index not used or missing
  }
}
```

### **7. Index Best Practices**

```typescript
// ✅ DO Index:
@Entity()
export class Product {
  // 1. Foreign keys (for JOINs)
  @Column()
  @Index()
  categoryId: string;  // Used in JOINs

  // 2. WHERE clause columns
  @Column()
  @Index()
  status: string;  // WHERE status = 'active'

  // 3. ORDER BY columns
  @Column()
  @Index()
  price: number;  // ORDER BY price

  // 4. Composite for common query patterns
  @Column()
  createdAt: Date;
}

// Add composite index
@Index(['categoryId', 'status', 'price'])
export class Product {
  // Optimizes: WHERE categoryId = X AND status = 'active' ORDER BY price
}

// ❌ DON'T Index:
// 1. Low cardinality columns (few distinct values)
@Column()
isActive: boolean;  // Only 2 values (true/false) - index not helpful

// 2. Frequently updated columns (slows down writes)
@Column()
viewCount: number;  // Updated on every view - don't index

// 3. Large text columns
@Column('text')
description: string;  // Use full-text index instead

// 4. Columns never used in queries
@Column()
metadata: string;  // Never in WHERE/JOIN - no index needed
```

### **8. Monitoring Index Usage**

```typescript
// Check unused indexes (PostgreSQL)
export class IndexMonitoring {
  async findUnusedIndexes(dataSource: DataSource) {
    const result = await dataSource.query(`
      SELECT 
        schemaname,
        tablename,
        indexname,
        idx_scan as index_scans,
        pg_size_pretty(pg_relation_size(indexrelid)) as index_size
      FROM pg_stat_user_indexes
      WHERE idx_scan = 0
        AND indexrelname NOT LIKE '%_pkey'
      ORDER BY pg_relation_size(indexrelid) DESC
    `);
    
    // Shows indexes that are never used (waste of space)
    return result;
  }

  async findMissingIndexes(dataSource: DataSource) {
    // Analyze slow query log for sequential scans
    const result = await dataSource.query(`
      SELECT 
        schemaname,
        tablename,
        seq_scan,
        seq_tup_read,
        idx_scan,
        idx_tup_fetch
      FROM pg_stat_user_tables
      WHERE seq_scan > 1000  -- Many sequential scans
        AND idx_scan < seq_scan  -- More seq scans than index scans
      ORDER BY seq_scan DESC
    `);
    
    // Tables with many sequential scans likely need indexes
    return result;
  }
}
```

**Interview Tip**: **Indexes**: Speed up queries by creating search structures (B-tree by default). **Types**: Single-column (@Index()), composite (@Index(['col1', 'col2']) - order matters!), unique (enforces + speeds up), full-text (GIN for text search). **When to index**: WHERE clauses, JOIN columns, ORDER BY, foreign keys. **Performance**: 100-1000× faster for large tables (O(log n) vs O(n)). **Trade-offs**: Faster reads, slower writes (index must update), storage overhead. **Best practices**: Index high-cardinality columns, composite indexes for common query patterns, avoid indexing low-cardinality (boolean), use EXPLAIN to verify index usage. **Production**: Monitor unused indexes, analyze slow queries, use partial indexes for subsets.

</details>

15. How do you use query optimization and EXPLAIN?

<details>
<summary><strong>Answer</strong></summary>

**EXPLAIN**: Database command showing **query execution plan** (how DB will execute query). **Shows**: Index usage, join methods, estimated rows, execution time. **Usage**: `EXPLAIN SELECT ...` (plan) or `EXPLAIN ANALYZE` (plan + actual execution). **Optimization**: Look for **Seq Scan** (bad - needs index), prefer **Index Scan** (good), check **execution time**, optimize **high-cost** operations. **Production**: Enable slow query logging, use EXPLAIN on slow queries, add indexes based on analysis.

### **1. Basic EXPLAIN Usage**

```typescript
// Enable query logging in TypeORM
// ormconfig.ts
export default {
  type: 'postgres',
  logging: ['query', 'error'],
  logger: 'advanced-console',
  // ...
};

@Injectable()
export class QueryAnalysisService {
  constructor(private dataSource: DataSource) {}

  async explainQuery(query: string) {
    // EXPLAIN shows execution plan (doesn't run query)
    const plan = await this.dataSource.query(`EXPLAIN ${query}`);
    console.log('Query Plan:', plan);

    // EXPLAIN ANALYZE runs query and shows actual performance
    const analysis = await this.dataSource.query(`EXPLAIN ANALYZE ${query}`);
    console.log('Query Analysis:', analysis);
    
    return analysis;
  }
}

// Example output:
// EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';

// Without index:
// Seq Scan on users  (cost=0.00..180.00 rows=1 width=100)
//   Filter: (email = 'test@example.com'::text)
// ❌ "Seq Scan" = Full table scan (SLOW!)

// With index:
// Index Scan using IDX_user_email on users  (cost=0.29..8.30 rows=1 width=100)
//   Index Cond: (email = 'test@example.com'::text)
// ✅ "Index Scan" = Using index (FAST!)
```

### **2. Reading EXPLAIN Output**

```typescript
// Understanding EXPLAIN ANALYZE output
@Injectable()
export class ExplainService {
  async analyzeUserQuery() {
    const result = await this.dataSource.query(`
      EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON)
      SELECT u.*, p.*
      FROM users u
      LEFT JOIN posts p ON p.user_id = u.id
      WHERE u.status = 'active'
      ORDER BY u.created_at DESC
      LIMIT 10
    `);

    /*
    Key metrics to look for:
    
    1. Execution Time: Total query time
       - Target: <100ms for user-facing queries
       - >1000ms = needs optimization
    
    2. Node Types:
       - "Index Scan" = GOOD (using index)
       - "Seq Scan" = BAD (full table scan, add index!)
       - "Bitmap Heap Scan" = OK (multiple index conditions)
       - "Nested Loop" = OK for small datasets
       - "Hash Join" = Good for large joins
    
    3. Cost: Estimated resource usage
       - First number = startup cost
       - Second number = total cost
       - Lower = better
    
    4. Rows: Estimated vs Actual
       - Large difference = outdated statistics (run ANALYZE)
    
    5. Buffers: Memory usage
       - "shared hit" = From cache (fast)
       - "shared read" = From disk (slow)
    */

    return result;
  }
}
```

### **3. Common Query Problems & Fixes**

```typescript
// Problem 1: Sequential Scan (missing index)

// ❌ BAD Query:
const slowQuery = `
  SELECT * FROM users 
  WHERE status = 'active'  -- No index on status
`;

// EXPLAIN shows:
// Seq Scan on users  (cost=0.00..2500.00 rows=50000 width=100)
//   Filter: (status = 'active')
// Planning Time: 0.1ms
// Execution Time: 450ms  ❌ SLOW!

// ✅ FIX: Add index
@Entity()
export class User {
  @Column()
  @Index()  // Add index!
  status: string;
}

// After index:
// Index Scan using IDX_user_status on users  (cost=0.42..1850.50 rows=50000 width=100)
//   Index Cond: (status = 'active')
// Execution Time: 12ms  ✅ 37x faster!

// Problem 2: N+1 Queries

// ❌ BAD: Separate queries
async getPostsWithAuthors() {
  const posts = await this.postsRepo.find();  // Query 1
  
  for (const post of posts) {
    post.author = await this.usersRepo.findOne({ where: { id: post.authorId } });  // N queries
  }
}

// ✅ FIX: Use JOIN
async getPostsWithAuthors() {
  return this.postsRepo
    .createQueryBuilder('post')
    .leftJoinAndSelect('post.author', 'author')
    .getMany();
  
  // EXPLAIN shows:
  // Hash Join  (cost=50.00..1200.00 rows=1000 width=200)
  //   Hash Cond: (post.author_id = author.id)
  //   ->  Seq Scan on posts  (cost=0.00..200.00 rows=1000)
  //   ->  Hash  (cost=50.00..50.00 rows=1000 width=100)
  //         ->  Seq Scan on users author  (cost=0.00..50.00)
  // Execution Time: 25ms  ✅ Fast single query
}

// Problem 3: Inefficient ORDER BY

// ❌ BAD: ORDER BY non-indexed column
const slowSort = `
  SELECT * FROM products 
  ORDER BY price DESC  -- No index on price
  LIMIT 10
`;

// EXPLAIN shows:
// Limit  (cost=3500.00..3502.50 rows=10)
//   ->  Sort  (cost=3500.00..3750.00 rows=100000)  ❌ Expensive sort!
//         Sort Key: price DESC
//         ->  Seq Scan on products  (cost=0.00..1500.00 rows=100000)
// Execution Time: 680ms

// ✅ FIX: Add index on price
@Entity()
export class Product {
  @Column()
  @Index()  // Index for sorting
  price: number;
}

// After index:
// Limit  (cost=0.42..1.77 rows=10)
//   ->  Index Scan Backward using IDX_product_price on products
// Execution Time: 0.5ms  ✅ 1360x faster!
```

### **4. Query Optimization Techniques**

```typescript
@Injectable()
export class OptimizedQueriesService {
  // 1. Select only needed columns
  async getUserSummary(id: string) {
    // ❌ BAD: SELECT *
    const user = await this.usersRepo.findOne({ where: { id } });
    // Loads all columns including large BLOBs

    // ✅ GOOD: Select specific columns
    return this.usersRepo
      .createQueryBuilder('user')
      .select(['user.id', 'user.name', 'user.email'])  // Only needed fields
      .where('user.id = :id', { id })
      .getOne();
  }

  // 2. Use appropriate JOIN types
  async getActivePostsWithAuthors() {
    // ✅ GOOD: INNER JOIN (only posts with authors)
    return this.postsRepo
      .createQueryBuilder('post')
      .innerJoinAndSelect('post.author', 'author')  // INNER JOIN
      .where('post.status = :status', { status: 'active' })
      .getMany();
    
    // Uses INNER JOIN (faster than LEFT JOIN when relation is required)
  }

  // 3. Add WHERE clause before JOIN (filter early)
  async getRecentPostsByCategory(categoryId: string) {
    return this.postsRepo
      .createQueryBuilder('post')
      .where('post.categoryId = :categoryId', { categoryId })  // Filter first
      .andWhere('post.createdAt > :date', { date: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000) })
      .leftJoinAndSelect('post.author', 'author')  // Then join
      .orderBy('post.createdAt', 'DESC')
      .getMany();
    
    // Filters before JOIN = fewer rows to join = faster
  }

  // 4. Use LIMIT to reduce rows processed
  async getTopProducts() {
    return this.productsRepo
      .createQueryBuilder('product')
      .orderBy('product.sales', 'DESC')
      .limit(10)  // Only get top 10
      .getMany();
    
    // Database can stop processing after finding 10 rows
  }

  // 5. Avoid LIKE '%value%' (can't use index)
  async searchProducts(term: string) {
    // ❌ BAD: Leading wildcard
    return this.productsRepo
      .createQueryBuilder('product')
      .where('product.name LIKE :term', { term: `%${term}%` })
      .getMany();
    // Can't use index - full table scan

    // ✅ BETTER: Use full-text search or prefix match
    return this.productsRepo
      .createQueryBuilder('product')
      .where('product.name LIKE :term', { term: `${term}%` })  // Prefix match
      .getMany();
    // Can use index for prefix search

    // ✅ BEST: Full-text search index (from previous Q14)
  }

  // 6. Use covering indexes (all columns in SELECT are in index)
  async getProductPrices() {
    // If index includes [id, name, price]:
    return this.productsRepo
      .createQueryBuilder('product')
      .select(['product.id', 'product.name', 'product.price'])
      .getMany();
    // "Index Only Scan" - doesn't need to read table at all!
  }
}
```

### **5. Monitoring Slow Queries**

```typescript
// Slow query logging interceptor
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class SlowQueryInterceptor implements NestInterceptor {
  private queryStartTime: number;

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    this.queryStartTime = Date.now();

    return next.handle().pipe(
      tap(() => {
        const duration = Date.now() - this.queryStartTime;

        if (duration > 1000) {  // > 1 second
          console.warn(`⚠️ Slow query detected: ${duration}ms`, {
            route: context.getHandler().name,
            controller: context.getClass().name,
          });
        }
      }),
    );
  }
}

// PostgreSQL slow query log configuration
// postgresql.conf
/*
log_min_duration_statement = 1000  # Log queries > 1 second
log_line_prefix = '%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h '
log_statement = 'all'
log_duration = on
*/
```

### **6. Query Performance Testing**

```typescript
@Injectable()
export class QueryPerformanceService {
  constructor(private dataSource: DataSource) {}

  async benchmarkQuery(query: string, iterations: number = 10) {
    const times: number[] = [];

    for (let i = 0; i < iterations; i++) {
      const start = Date.now();
      await this.dataSource.query(query);
      times.push(Date.now() - start);
    }

    const avg = times.reduce((a, b) => a + b) / times.length;
    const min = Math.min(...times);
    const max = Math.max(...times);

    return {
      average: avg,
      min,
      max,
      times,
    };
  }

  async compareQueries(query1: string, query2: string) {
    console.log('Benchmarking Query 1...');
    const results1 = await this.benchmarkQuery(query1);

    console.log('Benchmarking Query 2...');
    const results2 = await this.benchmarkQuery(query2);

    const improvement = ((results1.average - results2.average) / results1.average) * 100;

    return {
      query1: results1,
      query2: results2,
      improvement: `${improvement.toFixed(2)}% ${improvement > 0 ? 'faster' : 'slower'}`,
    };
  }
}
```

### **7. Database Statistics**

```typescript
// Update database statistics for better query plans
export class DatabaseMaintenanceService {
  constructor(private dataSource: DataSource) {}

  async updateStatistics() {
    // PostgreSQL: Update statistics for query planner
    await this.dataSource.query('ANALYZE');
    // Run after bulk inserts/updates
  }

  async vacuumDatabase() {
    // PostgreSQL: Reclaim storage and update statistics
    await this.dataSource.query('VACUUM ANALYZE');
    // Run periodically (e.g., nightly)
  }

  async checkTableStats(tableName: string) {
    const stats = await this.dataSource.query(`
      SELECT 
        schemaname,
        tablename,
        n_live_tup as live_rows,
        n_dead_tup as dead_rows,
        last_vacuum,
        last_analyze
      FROM pg_stat_user_tables
      WHERE tablename = $1
    `, [tableName]);

    return stats;
  }
}
```

**Interview Tip**: **EXPLAIN**: Shows query execution plan. Use `EXPLAIN ANALYZE` to run query and see actual performance. **Read output**: Look for "Seq Scan" (bad - add index), prefer "Index Scan" (good), check execution time (<100ms target). **Optimization**: Select only needed columns, add indexes on WHERE/JOIN/ORDER BY columns, filter before JOIN, use LIMIT, avoid leading wildcards in LIKE. **Common issues**: Missing indexes (Seq Scan), N+1 queries (use JOIN), inefficient ORDER BY (index sort column), SELECT * (fetch specific columns). **Production**: Enable slow query logging (>1s), use EXPLAIN on slow queries, run ANALYZE after bulk changes, monitor with APM tools. **Performance gains**: Proper indexes can improve 100-1000×.

</details>

16. Should you use `find()` or QueryBuilder for complex queries?

<details>
<summary><strong>Answer</strong></summary>

**Use find()**: Simple queries (single table, basic WHERE), relations loading, quick prototyping. **Use QueryBuilder**: Complex conditions (OR, subqueries), custom joins, aggregations, raw SQL snippets, performance-critical queries. **Performance**: QueryBuilder generates optimized SQL, find() may create multiple queries. **Best practice**: Start with find(), switch to QueryBuilder when needed. **Production**: QueryBuilder for complex/performance-critical, find() for simple CRUD.

```typescript
// ✅ Use find() for simple queries
@Injectable()
export class SimpleQueriesService {
  // Simple lookup
  async getUser(id: string) {
    return this.usersRepo.findOne({ where: { id } });
  }

  // Basic filtering
  async getActiveUsers() {
    return this.usersRepo.find({
      where: { status: 'active' },
      order: { createdAt: 'DESC' },
      take: 10,
    });
  }

  // With relations
  async getUserWithProfile(id: string) {
    return this.usersRepo.findOne({
      where: { id },
      relations: ['profile', 'settings'],
    });
  }
}

// ✅ Use QueryBuilder for complex queries
@Injectable()
export class ComplexQueriesService {
  // Complex conditions (OR, nested)
  async searchUsers(filters: SearchFilters) {
    const query = this.usersRepo.createQueryBuilder('user');

    // Complex OR conditions
    query.where(
      '(user.email LIKE :term OR user.username LIKE :term OR user.name LIKE :term)',
      { term: `%${filters.searchTerm}%` },
    );

    // Additional filters
    if (filters.status) {
      query.andWhere('user.status = :status', { status: filters.status });
    }

    if (filters.minAge) {
      query.andWhere('user.age >= :minAge', { minAge: filters.minAge });
    }

    return query.getMany();
  }

  // Aggregations
  async getUserStatistics() {
    return this.usersRepo
      .createQueryBuilder('user')
      .select('user.status', 'status')
      .addSelect('COUNT(user.id)', 'count')
      .addSelect('AVG(user.age)', 'avgAge')
      .groupBy('user.status')
      .getRawMany();
  }

  // Subqueries
  async getActiveUsersWithRecentPosts() {
    return this.usersRepo
      .createQueryBuilder('user')
      .where((qb) => {
        const subQuery = qb
          .subQuery()
          .select('post.userId')
          .from('Post', 'post')
          .where('post.createdAt > :date', {
            date: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000),
          })
          .getQuery();
        return `user.id IN ${subQuery}`;
      })
      .getMany();
  }

  // Complex joins with conditions
  async getPostsWithCommentCount() {
    return this.postsRepo
      .createQueryBuilder('post')
      .leftJoinAndSelect('post.author', 'author')
      .leftJoin('post.comments', 'comments')
      .addSelect('COUNT(comments.id)', 'commentCount')
      .groupBy('post.id')
      .addGroupBy('author.id')
      .having('COUNT(comments.id) > :minComments', { minComments: 5 })
      .getRawAndEntities();
  }

  // Raw SQL snippets for performance
  async getTopProducts() {
    return this.productsRepo
      .createQueryBuilder('product')
      .addSelect(
        '(SELECT COUNT(*) FROM orders WHERE product_id = product.id)',
        'orderCount',
      )
      .orderBy('orderCount', 'DESC')
      .limit(10)
      .getRawMany();
  }
}

// Performance comparison
// Simple query: find() = 1 query, QueryBuilder = 1 query (similar)
// Complex query: find() = multiple queries, QueryBuilder = 1 optimized query (faster)
```

**Interview Tip**: **find()**: Simple, readable, good for basic CRUD (single table, simple WHERE, relations loading). **QueryBuilder**: Powerful for complex queries (OR conditions, subqueries, aggregations, custom joins, raw SQL). **When to use**: find() for simple lookups, QueryBuilder for multi-condition filters, aggregations, performance-critical queries. **Performance**: QueryBuilder generates single optimized SQL, find() may create multiple queries for complex scenarios. **Production**: Use QueryBuilder for complex business logic, find() for standard CRUD operations.

</details>

17. How do you implement database connection pooling?

<details>
<summary><strong>Answer</strong></summary>

**Connection pooling**: Reuse database connections instead of creating new ones per request. **Benefits**: Faster (no connection overhead), handles high concurrency, limits DB load. **Configuration**: Set min/max pool size, idle timeout, connection timeout. **TypeORM**: Configured in datasource options (poolSize/extra). **Production**: Pool size = 2-10 per CPU core, monitor active connections, set timeouts.

```typescript
// Basic connection pooling in TypeORM
// app.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: process.env.DB_HOST,
      port: parseInt(process.env.DB_PORT),
      username: process.env.DB_USER,
      password: process.env.DB_PASSWORD,
      database: process.env.DB_NAME,
      
      // Connection pooling settings
      poolSize: 10,  // Max connections in pool (default: 10)
      
      // PostgreSQL-specific pool options
      extra: {
        max: 20,  // Maximum pool size
        min: 5,   // Minimum pool size (keep idle connections)
        idleTimeoutMillis: 30000,  // Close idle connections after 30s
        connectionTimeoutMillis: 2000,  // Wait 2s for available connection
        
        // Connection validation
        keepAlive: true,
        keepAliveInitialDelayMillis: 10000,
      },
      
      entities: [User, Post, Comment],
      synchronize: false,  // Never true in production
    }),
  ],
})
export class AppModule {}

// Without pooling:
// Request 1: Create connection (100ms) + Query (10ms) + Close (50ms) = 160ms
// Request 2: Create connection (100ms) + Query (10ms) + Close (50ms) = 160ms
// Total: 320ms for 2 requests

// With pooling:
// Request 1: Get from pool (1ms) + Query (10ms) + Return to pool (1ms) = 12ms
// Request 2: Get from pool (1ms) + Query (10ms) + Return to pool (1ms) = 12ms
// Total: 24ms for 2 requests (13x faster!)
```

### **Advanced Configuration**

```typescript
// Environment-based pool configuration
import { ConfigModule, ConfigService } from '@nestjs/config';

@Module({
  imports: [
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (configService: ConfigService) => {
        const isProduction = configService.get('NODE_ENV') === 'production';
        
        return {
          type: 'postgres',
          host: configService.get('DB_HOST'),
          port: configService.get('DB_PORT'),
          username: configService.get('DB_USER'),
          password: configService.get('DB_PASSWORD'),
          database: configService.get('DB_NAME'),
          
          // Different pool sizes per environment
          poolSize: isProduction ? 20 : 5,
          
          extra: {
            // Production: Larger pool
            max: isProduction ? 50 : 10,
            min: isProduction ? 10 : 2,
            
            // Aggressive idle timeout in dev (save resources)
            idleTimeoutMillis: isProduction ? 30000 : 10000,
            
            // Connection timeout
            connectionTimeoutMillis: 5000,
            
            // Query timeout (prevent hanging queries)
            statement_timeout: 30000,  // 30 seconds max per query
            
            // Connection validation
            keepAlive: true,
            keepAliveInitialDelayMillis: isProduction ? 10000 : 30000,
            
            // SSL in production
            ssl: isProduction ? { rejectUnauthorized: false } : false,
          },
        };
      },
    }),
  ],
})
export class AppModule {}
```

### **Connection Pool Monitoring**

```typescript
// Monitor pool health
import { Injectable, OnModuleInit } from '@nestjs/common';
import { DataSource } from 'typeorm';

@Injectable()
export class DatabaseHealthService implements OnModuleInit {
  constructor(private dataSource: DataSource) {}

  async onModuleInit() {
    // Log pool status on startup
    setInterval(() => this.logPoolStatus(), 60000);  // Every minute
  }

  async logPoolStatus() {
    const pool = (this.dataSource.driver as any).master;
    
    console.log('Database Pool Status:', {
      totalCount: pool.totalCount,     // Total connections
      idleCount: pool.idleCount,       // Idle connections
      waitingCount: pool.waitingCount, // Requests waiting for connection
    });
    
    // Alert if pool is exhausted
    if (pool.waitingCount > 0) {
      console.warn('⚠️ Database pool exhausted! Requests waiting:', pool.waitingCount);
    }
  }

  async getPoolMetrics() {
    const pool = (this.dataSource.driver as any).master;
    
    return {
      total: pool.totalCount,
      idle: pool.idleCount,
      active: pool.totalCount - pool.idleCount,
      waiting: pool.waitingCount,
      maxConnections: pool.options.max,
      utilizationPercent: ((pool.totalCount - pool.idleCount) / pool.options.max) * 100,
    };
  }
}

// Expose metrics endpoint
@Controller('health')
export class HealthController {
  constructor(private dbHealth: DatabaseHealthService) {}

  @Get('database')
  async checkDatabase() {
    const metrics = await this.dbHealth.getPoolMetrics();
    
    return {
      status: metrics.utilizationPercent < 80 ? 'healthy' : 'warning',
      ...metrics,
    };
  }
}
```

### **Handling Pool Exhaustion**

```typescript
// Graceful handling when pool is full
import { Injectable } from '@nestjs/common';
import { DataSource } from 'typeorm';

@Injectable()
export class ResilientDatabaseService {
  constructor(private dataSource: DataSource) {}

  async executeWithRetry<T>(
    operation: () => Promise<T>,
    maxRetries: number = 3,
  ): Promise<T> {
    let lastError: Error;

    for (let attempt = 1; attempt <= maxRetries; attempt++) {
      try {
        return await operation();
      } catch (error) {
        lastError = error;

        // Check if error is due to pool exhaustion
        if (error.message?.includes('timeout') || 
            error.message?.includes('pool')) {
          
          console.warn(`Database pool busy, retry ${attempt}/${maxRetries}`);
          
          // Exponential backoff
          await this.sleep(attempt * 100);
          continue;
        }

        // Other errors, throw immediately
        throw error;
      }
    }

    throw new Error(`Operation failed after ${maxRetries} retries: ${lastError.message}`);
  }

  private sleep(ms: number): Promise<void> {
    return new Promise((resolve) => setTimeout(resolve, ms));
  }
}

// Usage
@Injectable()
export class UsersService {
  constructor(private resilientDb: ResilientDatabaseService) {}

  async getUser(id: string) {
    return this.resilientDb.executeWithRetry(() =>
      this.usersRepo.findOne({ where: { id } }),
    );
  }
}
```

### **Connection Pool Best Practices**

```typescript
// Calculate optimal pool size
// Formula: connections_per_instance = ((core_count * 2) + effective_spindle_count)
// For cloud: 10-20 connections per instance
// For 4-core server: (4 * 2) + 1 = 9 connections

// Example configurations:

// Small app (1-2 instances, low traffic)
const smallAppConfig = {
  poolSize: 10,
  extra: {
    max: 10,
    min: 2,
    idleTimeoutMillis: 30000,
  },
};

// Medium app (5-10 instances, moderate traffic)
const mediumAppConfig = {
  poolSize: 20,
  extra: {
    max: 20,
    min: 5,
    idleTimeoutMillis: 30000,
  },
};

// Large app (20+ instances, high traffic)
const largeAppConfig = {
  poolSize: 15,  // Per instance
  extra: {
    max: 15,
    min: 10,
    idleTimeoutMillis: 60000,  // Keep connections longer
  },
};
// Total DB connections = instances × max_per_instance
// 20 instances × 15 = 300 total connections
// Ensure DB max_connections > 300
```

**Interview Tip**: **Connection pooling**: Reuses DB connections (faster than creating new ones). **Benefits**: 10-100× faster requests (no connection overhead), handles concurrency, limits DB load. **Configuration**: poolSize (max connections), extra.min (idle connections kept), idleTimeoutMillis (close idle after timeout), connectionTimeoutMillis (wait for available connection). **Production**: Pool size = 10-20 per instance, monitor utilization (<80%), set query timeout (prevent hanging), use health checks. **Formula**: Pool size ≈ (cores × 2) + disk spindles. **Common issues**: Pool exhaustion (increase size or optimize queries), idle connections (tune timeout), connection leaks (always use try/finally).

</details>

18. How do you use pagination to reduce query load?

<details>
<summary><strong>Answer</strong></summary>

**Pagination**: Load data in **chunks/pages** instead of all at once. **Methods**: **Offset-based** (skip/take), **Cursor-based** (keyset), **Load more** (infinite scroll). **Benefits**: Faster queries, less memory, better UX. **TypeORM**: skip/take or take/skip. **Best practice**: Cursor-based for large datasets, offset for small. **Production**: Default page size 20-50, max 100, cache counts.

```typescript
// 1. Offset-based Pagination (skip/take)
@Injectable()
export class PostsService {
  async getPaginatedPosts(page: number = 1, limit: number = 20) {
    const [posts, total] = await this.postsRepo.findAndCount({
      skip: (page - 1) * limit,
      take: limit,
      order: { createdAt: 'DESC' },
    });

    return {
      data: posts,
      meta: {
        total,
        page,
        limit,
        totalPages: Math.ceil(total / limit),
        hasNextPage: page < Math.ceil(total / limit),
        hasPrevPage: page > 1,
      },
    };
  }
}

// Controller
@Controller('posts')
export class PostsController {
  @Get()
  async getPosts(
    @Query('page', new DefaultValuePipe(1), ParseIntPipe) page: number,
    @Query('limit', new DefaultValuePipe(20), ParseIntPipe) limit: number,
  ) {
    // Enforce max limit
    const safeLimit = Math.min(limit, 100);
    
    return this.postsService.getPaginatedPosts(page, safeLimit);
  }
}

// ❌ Without pagination: Load ALL posts
// SELECT * FROM posts ORDER BY created_at DESC
// Result: 100,000 rows × 1KB = 100MB (SLOW!)
// Query time: 5 seconds

// ✅ With pagination: Load page 1 (20 posts)
// SELECT * FROM posts ORDER BY created_at DESC LIMIT 20 OFFSET 0
// Result: 20 rows × 1KB = 20KB (FAST!)
// Query time: 50ms (100× faster!)
```

### **2. Cursor-based Pagination (Better for Large Datasets)**

```typescript
// Uses last record ID instead of offset
@Injectable()
export class PostsService {
  async getCursorPaginatedPosts(
    limit: number = 20,
    cursor?: string,  // Last post ID from previous page
  ) {
    const query = this.postsRepo
      .createQueryBuilder('post')
      .orderBy('post.createdAt', 'DESC')
      .addOrderBy('post.id', 'DESC')  // Tie-breaker for same timestamp
      .take(limit + 1);  // Fetch one extra to check if more pages

    if (cursor) {
      // Get posts after cursor
      query.where('post.id < :cursor', { cursor });
    }

    const posts = await query.getMany();
    const hasNextPage = posts.length > limit;

    // Remove extra item
    if (hasNextPage) {
      posts.pop();
    }

    return {
      data: posts,
      meta: {
        nextCursor: hasNextPage ? posts[posts.length - 1].id : null,
        hasNextPage,
      },
    };
  }
}

// Controller
@Get('feed')
async getFeed(
  @Query('cursor') cursor?: string,
  @Query('limit', new DefaultValuePipe(20), ParseIntPipe) limit: number = 20,
) {
  return this.postsService.getCursorPaginatedPosts(limit, cursor);
}

// Benefits of cursor-based:
// - Consistent performance (no offset, always uses index)
// - No skipped/duplicate items when data changes
// - Better for real-time feeds

// Offset-based problem:
// User on page 5, someone adds 20 posts → user sees duplicates on page 6
// Cursor-based: Always continues from exact position
```

### **3. QueryBuilder Pagination**

```typescript
@Injectable()
export class UsersService {
  async searchUsers(filters: SearchFilters) {
    const query = this.usersRepo
      .createQueryBuilder('user')
      .leftJoinAndSelect('user.profile', 'profile');

    // Apply filters
    if (filters.status) {
      query.where('user.status = :status', { status: filters.status });
    }

    if (filters.search) {
      query.andWhere('user.name ILIKE :search', { search: `%${filters.search}%` });
    }

    // Pagination
    query
      .skip((filters.page - 1) * filters.limit)
      .take(filters.limit);

    // Get results + total count
    const [users, total] = await query.getManyAndCount();

    return {
      data: users,
      meta: {
        total,
        page: filters.page,
        limit: filters.limit,
        totalPages: Math.ceil(total / filters.limit),
      },
    };
  }
}
```

### **4. Efficient Count Queries**

```typescript
// ❌ BAD: Count all rows on every page (slow for large tables)
async getPosts(page: number) {
  const [posts, total] = await this.postsRepo.findAndCount({
    skip: (page - 1) * 20,
    take: 20,
  });
  // Executes: SELECT COUNT(*) FROM posts  (slow!)
}

// ✅ GOOD: Cache total count
@Injectable()
export class PostsService {
  constructor(@Inject(CACHE_MANAGER) private cache: Cache) {}

  async getPosts(page: number, limit: number) {
    // Get total from cache
    let total = await this.cache.get<number>('posts:total');
    
    if (!total) {
      total = await this.postsRepo.count();
      await this.cache.set('posts:total', total, 300);  // Cache 5 min
    }

    const posts = await this.postsRepo.find({
      skip: (page - 1) * limit,
      take: limit,
      order: { createdAt: 'DESC' },
    });

    return { data: posts, meta: { total, page, limit } };
  }

  // Invalidate count cache on create/delete
  async createPost(createDto: CreatePostDto) {
    const post = await this.postsRepo.save(createDto);
    await this.cache.del('posts:total');  // Invalidate count
    return post;
  }
}
```

### **5. Infinite Scroll Pagination**

```typescript
// For mobile apps / infinite scroll UIs
@Controller('posts')
export class PostsController {
  @Get('infinite')
  async getInfiniteScroll(
    @Query('lastId') lastId?: string,
    @Query('limit', new DefaultValuePipe(20), ParseIntPipe) limit: number = 20,
  ) {
    const query = this.postsRepo
      .createQueryBuilder('post')
      .orderBy('post.createdAt', 'DESC')
      .take(limit + 1);

    if (lastId) {
      query.andWhere('post.id < :lastId', { lastId });
    }

    const posts = await query.getMany();
    const hasMore = posts.length > limit;

    if (hasMore) {
      posts.pop();
    }

    return {
      data: posts,
      hasMore,
      lastId: posts.length > 0 ? posts[posts.length - 1].id : null,
    };
  }
}

// Frontend usage:
// Initial load: GET /posts/infinite
// Load more: GET /posts/infinite?lastId=<last_post_id>
```

### **6. Performance Comparison**

```typescript
// Scenario: 1 million posts

// ❌ NO PAGINATION:
SELECT * FROM posts ORDER BY created_at DESC
// Returns: 1,000,000 rows
// Memory: 1GB
// Query time: 30 seconds
// Transfer time: 60 seconds
// Total: 90 seconds ❌

// ✅ OFFSET PAGINATION (Page 1):
SELECT * FROM posts ORDER BY created_at DESC LIMIT 20 OFFSET 0
// Returns: 20 rows
// Memory: 20KB
// Query time: 50ms
// Transfer time: 10ms
// Total: 60ms ✅ (1500× faster!)

// ⚠️ OFFSET PAGINATION (Page 50,000):
SELECT * FROM posts ORDER BY created_at DESC LIMIT 20 OFFSET 999980
// Database must scan 999,980 rows to skip them
// Query time: 5 seconds ⚠️ (slow for deep pages)

// ✅ CURSOR PAGINATION (Any page):
SELECT * FROM posts WHERE id < '<cursor>' ORDER BY created_at DESC LIMIT 20
// Uses index, always fast
// Query time: 50ms ✅ (consistent!)
```

### **7. Pagination DTO**

```typescript
// Reusable pagination classes
export class PaginationDto {
  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  page?: number = 1;

  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  @Max(100)
  limit?: number = 20;
}

export class PaginatedResult<T> {
  data: T[];
  meta: {
    total: number;
    page: number;
    limit: number;
    totalPages: number;
    hasNextPage: boolean;
    hasPrevPage: boolean;
  };
}

// Usage
@Controller('users')
export class UsersController {
  @Get()
  async getUsers(@Query() pagination: PaginationDto): Promise<PaginatedResult<User>> {
    return this.usersService.findAll(pagination);
  }
}
```

**Interview Tip**: **Pagination**: Load data in chunks (pages) instead of all at once. **Methods**: Offset-based (skip/take - simple, good for small datasets), cursor-based (use last ID - better for large datasets, consistent performance), infinite scroll (mobile-friendly). **Performance**: Loading 1M rows = 90s, loading 20 rows = 60ms (1500× faster!). **TypeORM**: findAndCount({ skip, take }) or QueryBuilder. **Best practices**: Default limit 20-50, max 100, cache total counts, use cursor for deep pagination. **Production**: Offset for pages 1-100, cursor for infinite scroll or large datasets, add indexes on sort columns.

</details>

## Compression

19. What is compression and why use it?

<details>
<summary><strong>Answer</strong></summary>

**Compression**: Reduces response size by encoding data (gzip/brotli). **Benefits**: **60-90% smaller** responses, faster transfer, lower bandwidth costs, better UX. **Algorithms**: gzip (widely supported), brotli (better compression, slower). **Use for**: Text (JSON, HTML, CSS, JS), not for images/video (already compressed). **Production**: Enable compression middleware, use brotli for static assets, gzip for dynamic.

```typescript
// Without compression:
GET /api/products → Response: 500 KB JSON
Transfer time: 500KB / 1Mbps = 4 seconds ❌

// With gzip compression:
GET /api/products → Response: 50 KB (90% smaller!)
Transfer time: 50KB / 1Mbps = 0.4 seconds ✅ (10× faster!)

// Compression savings:
Original JSON: 500 KB
Gzip: 50 KB (90% reduction)
Brotli: 40 KB (92% reduction) - better but slower
```

### **Why Use Compression?**

```typescript
// 1. Bandwidth Savings
// Without compression:
// 1 million requests × 500 KB = 500 GB bandwidth
// Cost: 500 GB × $0.10/GB = $50/month

// With compression (90% reduction):
// 1 million requests × 50 KB = 50 GB bandwidth
// Cost: 50 GB × $0.10/GB = $5/month
// Savings: $45/month (90% cost reduction!)

// 2. Faster Response Times
// Mobile 3G (1 Mbps):
// Without compression: 500 KB = 4 seconds
// With compression: 50 KB = 0.4 seconds (10× faster)

// 3. Better User Experience
// Page load time: 2s → 0.5s
// Bounce rate: 30% → 10% (users stay!)
// SEO ranking: Improves (Google prefers fast sites)

// 4. Reduced Server Load
// Less bandwidth = more concurrent requests
// 1 Gbps can handle:
// - Without compression: 250 requests/second (500 KB each)
// - With compression: 2,500 requests/second (50 KB each)
// 10× more capacity!
```

### **Compression Algorithms**

```typescript
// 1. Gzip (most common)
// - Good compression ratio (60-70%)
// - Fast compression/decompression
// - Widely supported (all browsers)
// - Best for: Dynamic responses

// 2. Brotli (better compression)
// - Better compression ratio (70-80%)
// - Slower compression (but faster decompression)
// - Modern browser support (95%+)
// - Best for: Static assets (pre-compressed)

// 3. Deflate (older)
// - Similar to gzip but less efficient
// - Legacy support
// - Rarely used today

// Compression comparison:
const exampleJSON = {
  // Large product catalog
  products: Array(1000).fill({
    id: '123e4567-e89b-12d3-a456-426614174000',
    name: 'Example Product Name',
    description: 'Long product description with many details...',
    price: 99.99,
    category: 'Electronics',
    tags: ['popular', 'new', 'featured'],
  }),
};

// Original size: 500 KB
// Gzip: 50 KB (90% reduction) - fast
// Brotli: 40 KB (92% reduction) - slower but better
```

### **What to Compress vs Not Compress**

```typescript
// ✅ COMPRESS (Text-based):
// - JSON API responses (70-90% reduction)
// - HTML pages (60-80% reduction)
// - CSS files (70-85% reduction)
// - JavaScript files (60-80% reduction)
// - XML/SVG (70-90% reduction)
// - Plain text (70-85% reduction)

// ❌ DON'T COMPRESS (Already compressed):
// - Images: JPEG, PNG, WebP, GIF (0-5% reduction, waste CPU)
// - Videos: MP4, WebM (0-2% reduction)
// - Audio: MP3, AAC (0-3% reduction)
// - Archives: ZIP, RAR, 7z (already compressed)
// - Fonts: WOFF, WOFF2 (already compressed)

// Example: API response
const apiResponse = {
  users: Array(100).fill({
    id: 'uuid',
    name: 'John Doe',
    email: 'john@example.com',
    profile: { bio: 'Long bio text...', avatar: 'url' },
  }),
};

// Without compression:
JSON.stringify(apiResponse).length; // 50,000 bytes

// With gzip:
// Compressed: 5,000 bytes (90% reduction!)
// Transfer time: 10× faster
```

### **Compression Trade-offs**

```typescript
// CPU vs Bandwidth trade-off

// Without compression:
// - CPU: Low (no compression overhead)
// - Bandwidth: High (large responses)
// - Response time: Slow (large transfer)
// - Cost: High bandwidth costs

// With compression:
// - CPU: Medium (compression overhead ~5ms)
// - Bandwidth: Low (90% reduction)
// - Response time: Fast (small transfer)
// - Cost: Low bandwidth costs

// Break-even analysis:
// Compression time: 5ms
// Transfer time saved: 400ms (500KB → 50KB)
// Net benefit: 395ms faster (worth it!)

// When to skip compression:
// - Responses < 1 KB (overhead > benefit)
// - Already compressed content (images/video)
// - Real-time data (<10ms latency requirement)
// - High CPU usage scenarios
```

### **Browser Support**

```typescript
// Compression negotiation (automatic)
// Client sends:
GET /api/products
Accept-Encoding: gzip, deflate, br  // br = Brotli

// Server responds:
HTTP/1.1 200 OK
Content-Encoding: gzip  // or br
Content-Length: 5000  // Compressed size

// Browser automatically decompresses response
// Developer sees original data (transparent!)

// Browser support:
// - Gzip: 100% (all browsers)
// - Brotli: 95%+ (IE 11 doesn't support)
// - Deflate: 100% (legacy)
```

**Interview Tip**: **Compression**: Reduces response size by 60-90% using gzip/brotli encoding. **Benefits**: Faster transfer (10×), lower bandwidth costs (90% savings), better UX (faster page loads), more capacity. **Algorithms**: gzip (fast, widely supported), brotli (better compression, modern browsers). **Use for**: JSON, HTML, CSS, JS, SVG (text-based). **Don't use for**: Images, videos, fonts (already compressed, waste CPU). **Production**: Enable compression middleware, use brotli for static assets (pre-compress), gzip for dynamic responses. **Trade-off**: Small CPU overhead (~5ms) for huge transfer time savings (~400ms).

</details>

20. How do you enable response compression using `compression` middleware?

<details>
<summary><strong>Answer</strong></summary>

**Install**: `npm install compression @types/compression`. **Enable**: Import in main.ts, `app.use(compression())`. **Configuration**: Set threshold (min size), filter (content types), compression level. **NestJS**: Use as global middleware. **Production**: Level 6 (balanced), threshold 1024 bytes, filter text types only.

```typescript
// 1. Install compression middleware
// npm install compression
// npm install --save-dev @types/compression

// main.ts - Basic setup
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import * as compression from 'compression';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Enable compression (default settings)
  app.use(compression());
  
  await app.listen(3000);
}
bootstrap();

// Default behavior:
// - Compresses all text-based responses
// - Uses gzip algorithm
// - Threshold: 1024 bytes (doesn't compress < 1KB)
// - Compression level: 6 (balanced)
```

### **Advanced Configuration**

```typescript
// main.ts - Production-grade configuration
import { NestFactory } from '@nestjs/core';
import * as compression from 'compression';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  app.use(
    compression({
      // Compression level: 0-9
      // 0 = no compression, 9 = max compression (slow)
      // 6 = default (good balance)
      level: 6,
      
      // Minimum size to compress (bytes)
      // Don't compress responses < 1KB (overhead > benefit)
      threshold: 1024,
      
      // Filter: Only compress specific content types
      filter: (req, res) => {
        // Don't compress if client doesn't support
        if (req.headers['x-no-compression']) {
          return false;
        }
        
        // Use default filter (compresses text/* types)
        return compression.filter(req, res);
      },
      
      // Memory level: 1-9 (trade-off: memory vs speed)
      // 8 = default (128KB memory)
      memLevel: 8,
    }),
  );
  
  await app.listen(3000);
}
bootstrap();
```

### **Custom Compression Filter**

```typescript
// Only compress specific routes or content types
import { Request, Response } from 'express';

app.use(
  compression({
    filter: (req: Request, res: Response) => {
      // Don't compress already compressed content
      const contentType = res.getHeader('Content-Type') as string;
      
      if (!contentType) return false;
      
      // Compress only text-based content
      const compressibleTypes = [
        'text/',           // text/html, text/plain, text/css
        'application/json',
        'application/javascript',
        'application/xml',
        'image/svg+xml',   // SVG is text-based
      ];
      
      const shouldCompress = compressibleTypes.some((type) =>
        contentType.includes(type),
      );
      
      // Don't compress images/videos/fonts
      const nonCompressible = [
        'image/jpeg',
        'image/png',
        'image/webp',
        'video/',
        'font/',
      ];
      
      const shouldNotCompress = nonCompressible.some((type) =>
        contentType.includes(type),
      );
      
      return shouldCompress && !shouldNotCompress;
    },
  }),
);
```

### **Environment-based Configuration**

```typescript
// Different settings per environment
import { ConfigService } from '@nestjs/config';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  const config = app.get(ConfigService);
  
  const isProduction = config.get('NODE_ENV') === 'production';
  
  app.use(
    compression({
      // Production: Higher compression (CPU available)
      // Development: Lower compression (faster debugging)
      level: isProduction ? 6 : 1,
      
      // Production: Compress > 1KB
      // Development: Compress > 10KB (less overhead)
      threshold: isProduction ? 1024 : 10240,
      
      // Production: Enable for all text types
      // Development: Only JSON (easier debugging)
      filter: (req, res) => {
        if (!isProduction) {
          const contentType = res.getHeader('Content-Type') as string;
          return contentType?.includes('application/json') ?? false;
        }
        return compression.filter(req, res);
      },
    }),
  );
  
  await app.listen(3000);
}
```

### **Compression with Caching**

```typescript
// Combine compression with cache headers for best performance
import { Controller, Get, Header, Res } from '@nestjs/common';
import { Response } from 'express';

@Controller('api')
export class ApiController {
  @Get('data')
  @Header('Cache-Control', 'public, max-age=3600')  // Cache 1 hour
  async getData(@Res() res: Response) {
    const data = { /* large dataset */ };
    
    // Compression middleware handles this automatically
    // Response is:
    // 1. Compressed (90% smaller)
    // 2. Cached (subsequent requests from cache)
    // Result: First request = 50ms, subsequent = 0ms (cached locally)
    
    return res.json(data);
  }
}
```

### **Monitoring Compression**

```typescript
// Log compression ratio for monitoring
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class CompressionMonitorMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    const originalWrite = res.write.bind(res);
    const originalEnd = res.end.bind(res);
    
    let originalSize = 0;
    
    // Intercept response
    res.write = function (chunk: any, ...args: any[]) {
      if (chunk) {
        originalSize += chunk.length || 0;
      }
      return originalWrite(chunk, ...args);
    };
    
    res.end = function (chunk: any, ...args: any[]) {
      if (chunk) {
        originalSize += chunk.length || 0;
      }
      
      // Log compression stats
      const compressedSize = res.getHeader('Content-Length');
      if (compressedSize && originalSize > 0) {
        const ratio = ((1 - Number(compressedSize) / originalSize) * 100).toFixed(1);
        console.log(`Compression: ${originalSize}B → ${compressedSize}B (${ratio}% reduction)`);
      }
      
      return originalEnd(chunk, ...args);
    };
    
    next();
  }
}

// Apply globally
@Module({
  providers: [CompressionMonitorMiddleware],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(CompressionMonitorMiddleware).forRoutes('*');
  }
}
```

### **Testing Compression**

```bash
# Test if compression is working
curl -H "Accept-Encoding: gzip" -I http://localhost:3000/api/data

# Response headers:
HTTP/1.1 200 OK
Content-Type: application/json
Content-Encoding: gzip  # <-- Compression enabled!
Content-Length: 5000    # Compressed size
Vary: Accept-Encoding

# Compare sizes:
curl -H "Accept-Encoding: gzip" http://localhost:3000/api/data --output compressed.json
# Size: 5KB (compressed)

curl http://localhost:3000/api/data --output uncompressed.json
# Size: 50KB (uncompressed)

# Compression ratio: 90%!
```

### **Performance Impact**

```typescript
// Benchmark: 50KB JSON response

// Without compression:
// - CPU time: 0ms
// - Response size: 50KB
// - Transfer time (1Mbps): 400ms
// - Total: 400ms

// With gzip (level 6):
// - CPU time: 5ms (compression)
// - Response size: 5KB (90% reduction)
// - Transfer time (1Mbps): 40ms
// - Total: 45ms (9× faster!)

// With gzip (level 9 - max):
// - CPU time: 15ms (compression)
// - Response size: 4.5KB (91% reduction)
// - Transfer time (1Mbps): 36ms
// - Total: 51ms (only 10% better, not worth it!)

// Optimal: Level 6 (default)
// Best balance of compression ratio vs CPU time
```

**Interview Tip**: **Setup**: Install `compression` package, enable in main.ts with `app.use(compression())`. **Configuration**: Level 6 (balanced compression), threshold 1024 bytes (don't compress small responses), filter text types only (JSON, HTML, CSS, JS). **Production**: Compression reduces responses by 60-90%, saves bandwidth, improves speed. **Monitoring**: Check Content-Encoding header, log compression ratios. **Best practices**: Enable globally, exclude images/videos, combine with caching, use level 6 (not 9 - diminishing returns). **Performance**: 5ms CPU overhead for 400ms transfer time savings (80× improvement!).

</details>

21. What content should be compressed?

<details>
<summary><strong>Answer</strong></summary>

**Compress**: **Text-based** content (JSON, HTML, CSS, JS, XML, SVG) - 60-90% reduction. **Don't compress**: Already compressed (images, videos, fonts, ZIP) - 0-5% reduction, wastes CPU. **Rule**: Compress if Content-Type starts with text/ or is application/json|xml|javascript. **Production**: Filter by content type, threshold > 1KB, monitor compression ratio.

```typescript
// ✅ SHOULD COMPRESS (High compression ratio)
const compressibleContent = {
  // 1. JSON API responses (70-90% reduction)
  'application/json': {
    example: { users: Array(100).fill({ id: 1, name: 'User', email: 'test@example.com' }) },
    originalSize: '50 KB',
    compressedSize: '5 KB',
    reduction: '90%',
    worthIt: '✅ YES',
  },
  
  // 2. HTML pages (60-80% reduction)
  'text/html': {
    example: '<html><body>...</body></html>',
    originalSize: '100 KB',
    compressedSize: '20 KB',
    reduction: '80%',
    worthIt: '✅ YES',
  },
  
  // 3. CSS stylesheets (70-85% reduction)
  'text/css': {
    example: '.class { property: value; }',
    originalSize: '50 KB',
    compressedSize: '10 KB',
    reduction: '80%',
    worthIt: '✅ YES',
  },
  
  // 4. JavaScript files (60-80% reduction)
  'application/javascript': {
    example: 'function() { ... }',
    originalSize: '200 KB',
    compressedSize: '40 KB',
    reduction: '80%',
    worthIt: '✅ YES',
  },
  
  // 5. XML/SVG (70-90% reduction)
  'application/xml': {
    example: '<root><item>...</item></root>',
    originalSize: '30 KB',
    compressedSize: '5 KB',
    reduction: '85%',
    worthIt: '✅ YES',
  },
  
  'image/svg+xml': {
    example: '<svg><path d="..."/></svg>',
    originalSize: '20 KB',
    compressedSize: '3 KB',
    reduction: '85%',
    worthIt: '✅ YES',
  },
  
  // 6. Plain text (70-85% reduction)
  'text/plain': {
    example: 'Large log file or text document',
    originalSize: '100 KB',
    compressedSize: '15 KB',
    reduction: '85%',
    worthIt: '✅ YES',
  },
};

// ❌ SHOULD NOT COMPRESS (Already compressed, waste CPU)
const nonCompressibleContent = {
  // 1. Images (0-5% reduction)
  'image/jpeg': { originalSize: '100 KB', compressedSize: '98 KB', reduction: '2%', worthIt: '❌ NO' },
  'image/png': { originalSize: '50 KB', compressedSize: '49 KB', reduction: '2%', worthIt: '❌ NO' },
  'image/webp': { originalSize: '30 KB', compressedSize: '30 KB', reduction: '0%', worthIt: '❌ NO' },
  'image/gif': { originalSize: '20 KB', compressedSize: '20 KB', reduction: '0%', worthIt: '❌ NO' },
  
  // 2. Videos (0-2% reduction)
  'video/mp4': { originalSize: '5 MB', compressedSize: '5 MB', reduction: '0%', worthIt: '❌ NO' },
  'video/webm': { originalSize: '3 MB', compressedSize: '3 MB', reduction: '0%', worthIt: '❌ NO' },
  
  // 3. Audio (0-3% reduction)
  'audio/mpeg': { originalSize: '3 MB', compressedSize: '3 MB', reduction: '0%', worthIt: '❌ NO' },
  'audio/aac': { originalSize: '2 MB', compressedSize: '2 MB', reduction: '0%', worthIt: '❌ NO' },
  
  // 4. Fonts (0-5% reduction)
  'font/woff': { originalSize: '50 KB', compressedSize: '48 KB', reduction: '4%', worthIt: '❌ NO' },
  'font/woff2': { originalSize: '30 KB', compressedSize: '30 KB', reduction: '0%', worthIt: '❌ NO' },
  
  // 5. Archives (0% reduction, may grow!)
  'application/zip': { originalSize: '1 MB', compressedSize: '1.1 MB', reduction: '-10%', worthIt: '❌ NO' },
  'application/gzip': { originalSize: '500 KB', compressedSize: '520 KB', reduction: '-4%', worthIt: '❌ NO' },
};
```

### **Smart Compression Filter**

```typescript
// main.ts - Intelligent content type filtering
import * as compression from 'compression';

app.use(
  compression({
    filter: (req, res) => {
      const contentType = res.getHeader('Content-Type') as string;
      
      if (!contentType) {
        return false;
      }
      
      // ✅ Compress text-based content
      const shouldCompress = [
        'text/',                    // text/html, text/css, text/plain
        'application/json',         // JSON APIs
        'application/javascript',   // JS files
        'application/xml',          // XML
        'application/x-javascript', // Legacy JS
        'image/svg+xml',            // SVG (text-based)
      ].some((type) => contentType.toLowerCase().includes(type));
      
      // ❌ Don't compress binary/already compressed
      const shouldNotCompress = [
        'image/',       // JPEG, PNG, WebP, GIF (except SVG)
        'video/',       // MP4, WebM
        'audio/',       // MP3, AAC
        'font/',        // WOFF, WOFF2
        'application/zip',
        'application/gzip',
        'application/pdf',  // PDFs are already optimized
      ].some((type) => contentType.toLowerCase().includes(type));
      
      // Special case: Don't compress SVG if it's in image/ path
      if (contentType.includes('image/svg+xml') && shouldNotCompress) {
        return true;  // SVG is text-based, compress it!
      }
      
      return shouldCompress && !shouldNotCompress;
    },
    
    // Only compress responses > 1KB
    threshold: 1024,
  }),
);
```

### **Size-based Compression**

```typescript
// Don't compress small responses (overhead > benefit)
app.use(
  compression({
    filter: (req, res) => {
      const contentLength = res.getHeader('Content-Length');
      
      // Skip if < 1KB (compression overhead > benefit)
      if (contentLength && Number(contentLength) < 1024) {
        return false;
      }
      
      // Also check content type
      return compression.filter(req, res);
    },
    
    threshold: 1024,  // Minimum 1KB
  }),
);

// Why not compress small responses?
// Response size: 500 bytes
// Compression time: 5ms
// Transfer time saved: 0.5ms (500B → 200B)
// Net result: -4.5ms slower! (overhead > benefit)

// Response size: 50 KB
// Compression time: 5ms
// Transfer time saved: 400ms (50KB → 5KB)
// Net result: 395ms faster! (worth it!)
```

### **Route-specific Compression**

```typescript
// Compress only specific routes
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';
import * as compression from 'compression';

@Injectable()
export class SelectiveCompressionMiddleware implements NestMiddleware {
  private compressor = compression({
    filter: (req, res) => {
      // ✅ Compress API responses (JSON)
      if (req.path.startsWith('/api/')) {
        return true;
      }
      
      // ❌ Don't compress file downloads
      if (req.path.startsWith('/download/')) {
        return false;
      }
      
      // ❌ Don't compress streaming endpoints
      if (req.path.includes('/stream/')) {
        return false;
      }
      
      // Default filter
      return compression.filter(req, res);
    },
  });

  use(req: Request, res: Response, next: NextFunction) {
    this.compressor(req, res, next);
  }
}

// Apply to specific routes
@Module({})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(SelectiveCompressionMiddleware)
      .forRoutes('api/*');  // Only compress /api/* routes
  }
}
```

### **Dynamic Compression Decision**

```typescript
// Decide compression based on response characteristics
@Controller('api')
export class DataController {
  @Get('small')
  @Header('X-No-Compression', '1')  // Hint: Don't compress
  getSmallData() {
    return { message: 'Small response' };  // < 1KB, skip compression
  }
  
  @Get('large')
  async getLargeData() {
    // Large response, compression will help
    return { data: Array(1000).fill({ /* large object */ }) };
  }
  
  @Get('binary')
  @Header('Content-Type', 'application/octet-stream')
  async getBinary(@Res() res: Response) {
    // Binary data, don't compress
    const buffer = await this.readBinaryFile();
    res.send(buffer);
  }
}
```

### **Compression Decision Matrix**

```typescript
// Decision logic
function shouldCompress(contentType: string, size: number): boolean {
  // 1. Too small? Don't compress
  if (size < 1024) {
    return false;  // < 1KB: overhead > benefit
  }
  
  // 2. Text-based? Compress
  const isText = (
    contentType.startsWith('text/') ||
    contentType.includes('json') ||
    contentType.includes('javascript') ||
    contentType.includes('xml')
  );
  
  if (isText) {
    return true;  // Text compresses well (60-90%)
  }
  
  // 3. Already compressed? Don't compress
  const isCompressed = (
    contentType.includes('image/') ||
    contentType.includes('video/') ||
    contentType.includes('audio/') ||
    contentType.includes('zip') ||
    contentType.includes('gzip')
  );
  
  if (isCompressed) {
    return false;  // Already compressed (0-5%)
  }
  
  // 4. Unknown type, default: Don't compress
  return false;
}

// Examples:
shouldCompress('application/json', 50000);    // true (text, large)
shouldCompress('text/html', 500);             // false (text, too small)
shouldCompress('image/jpeg', 100000);         // false (already compressed)
shouldCompress('application/octet-stream', 50000);  // false (unknown binary)
shouldCompress('image/svg+xml', 10000);       // true (SVG is text)
```

### **Performance Comparison**

```typescript
// Test: 50KB JSON response

// Content type: application/json
// Original size: 50,000 bytes
// Compressed size: 5,000 bytes (90% reduction)
// Compression time: 5ms
// Transfer time saved: 360ms (50KB → 5KB at 1Mbps)
// Net benefit: 355ms faster ✅ WORTH IT!

// Content type: image/jpeg
// Original size: 50,000 bytes
// Compressed size: 49,500 bytes (1% reduction)
// Compression time: 5ms
// Transfer time saved: 4ms (50KB → 49.5KB at 1Mbps)
// Net benefit: -1ms slower ❌ NOT WORTH IT!

// Rule of thumb:
// - Compress if reduction > 50% (text-based)
// - Don't compress if reduction < 10% (binary/compressed)
// - Threshold: 1KB minimum (small responses = overhead > benefit)
```

**Interview Tip**: **Compress**: Text-based content (JSON, HTML, CSS, JS, XML, SVG) - **60-90% reduction**, huge benefit. **Don't compress**: Already compressed (images/videos/fonts/ZIP) - **0-5% reduction**, wastes CPU. **Rule**: Compress if Content-Type is text/* or application/json|xml|javascript, AND size > 1KB. **Implementation**: Filter by content type, set threshold 1024 bytes, monitor compression ratio. **Production**: Text = compress, binary = skip, threshold > 1KB, level 6. **Performance**: JSON (50KB → 5KB, 90% smaller) vs JPEG (50KB → 49.5KB, 1% smaller - not worth it).

</details>

## Rate Limiting & Throttling

22. What is rate limiting?

<details>
<summary><strong>Answer</strong></summary>

**Rate limiting**: Restricts **number of requests** a client can make in a **time window** (e.g., 100 requests per minute). **Purpose**: Prevent **abuse** (DDoS, spam, brute-force), ensure **fair usage**, protect **resources**. **Implementation**: Track requests per IP/user, block when limit exceeded, return 429 Too Many Requests. **Production**: 100 req/min for general APIs, 10 req/min for auth endpoints, use Redis for distributed systems.

```typescript
// Without rate limiting:
// Attacker sends 1000 requests/second → Server crashes ❌
// Scrapers overload API → Legitimate users blocked ❌
// Brute-force login attempts → Account compromised ❌

// With rate limiting:
// Client: 100 requests/minute allowed
// Request 1-100: ✅ 200 OK
// Request 101: ❌ 429 Too Many Requests
// After 1 minute: Reset to 0, requests allowed again

// Example scenario:
const scenario = {
  endpoint: '/api/users',
  limit: 100,  // requests
  window: '1 minute',
  
  requests: [
    { time: '00:00', count: 50, status: '✅ OK' },
    { time: '00:30', count: 50, status: '✅ OK (total: 100)' },
    { time: '00:45', count: 1, status: '❌ 429 Too Many Requests' },
    { time: '01:00', count: 1, status: '✅ OK (window reset)' },
  ],
};
```

### **Why Rate Limiting is Important**

```typescript
// 1. Prevent DDoS Attacks
// Attack scenario:
// Botnet sends 100,000 requests/second
// Without rate limiting:
// - Server CPU: 100% (all cores maxed)
// - Memory: Out of memory (OOM)
// - Database: Overloaded, crashes
// - Result: Service down for all users ❌

// With rate limiting (100 req/min per IP):
// - Legitimate users: 100 req/min (normal usage)
// - Attackers: Blocked after 100 requests
// - Server: CPU 20%, stable ✅
// - Result: Service remains available ✅

// 2. Prevent Brute-Force Attacks
// Login endpoint without rate limiting:
// Attacker tries 1000 passwords/second
// Time to crack 8-char password: 2 hours ❌

// Login endpoint with rate limiting (10 req/min):
// Attacker tries 10 passwords/minute
// Time to crack same password: 83 days ✅
// Plus: Account lockout after 5 failed attempts

// 3. Ensure Fair Usage
// Scenario: API with 1000 requests/second capacity
// User A: 900 req/sec (greedy client)
// User B-Z: 100 req/sec total (starved) ❌

// With rate limiting (50 req/sec per user):
// User A: 50 req/sec (limited)
// User B-Z: 950 req/sec total (fair share) ✅

// 4. Protect Resources
// Database can handle 1000 queries/second
// Without rate limiting:
// Spike: 10,000 requests → Database crashes ❌

// With rate limiting:
// Spike: Requests queued/rejected → Database stable ✅

// 5. Cost Control
// External API costs $0.01 per request
// Without rate limiting:
// Bug causes 1M requests → $10,000 bill ❌

// With rate limiting (10k req/hour):
// Bug limited to 10k requests → $100 bill ✅
```

### **Rate Limiting Strategies**

```typescript
// 1. Fixed Window
// Simple: Count requests in fixed time windows
const fixedWindow = {
  window: '1 minute',
  limit: 100,
  
  '00:00-01:00': 100,  // 100 requests allowed
  '01:00-02:00': 100,  // Reset to 100
  
  // Problem: Burst at window boundaries
  // 00:59: 100 requests
  // 01:00: 100 requests
  // Total: 200 requests in 1 second! ⚠️
};

// 2. Sliding Window
// Better: Rolling time window
const slidingWindow = {
  window: '1 minute',
  limit: 100,
  
  // At 00:30:
  // Count requests from 23:30-00:30 (last 60 seconds)
  // Smooth, no burst problem ✅
};

// 3. Token Bucket
// Advanced: Refill tokens at fixed rate
const tokenBucket = {
  capacity: 100,       // Max tokens
  refillRate: 10,      // Tokens per second
  
  // Request costs 1 token
  // Allows bursts up to capacity
  // Then limited to refill rate
  
  burst: 100,          // Initial burst
  sustained: 10,       // req/sec sustained
};

// 4. Leaky Bucket
// Smooth: Process requests at constant rate
const leakyBucket = {
  capacity: 100,       // Queue size
  rate: 10,            // req/sec processed
  
  // Excess requests queued
  // Queue full → reject
  // Ensures smooth, constant rate
};
```

### **Rate Limit Scopes**

```typescript
// 1. Per IP Address (Anonymous users)
const perIP = {
  scope: 'IP',
  limit: 100,
  window: '1 minute',
  use: 'Public APIs, authentication endpoints',
};

// 2. Per User (Authenticated users)
const perUser = {
  scope: 'User ID',
  limit: 1000,         // Higher limit for auth users
  window: '1 minute',
  use: 'Authenticated APIs',
};

// 3. Per API Key (Third-party integrations)
const perAPIKey = {
  scope: 'API Key',
  limit: 10000,        // Much higher for paid tiers
  window: '1 hour',
  use: 'Partner integrations, paid APIs',
};

// 4. Per Endpoint (Different limits per endpoint)
const perEndpoint = {
  '/api/auth/login': { limit: 10, window: '1 minute' },   // Strict (brute-force)
  '/api/users': { limit: 100, window: '1 minute' },       // Normal
  '/api/search': { limit: 1000, window: '1 minute' },     // High (read-only)
};

// 5. Global (Server-wide)
const global = {
  scope: 'Server',
  limit: 10000,        // Total requests server can handle
  window: '1 second',
  use: 'Overall capacity protection',
};
```

### **Rate Limit Response**

```typescript
// Standard 429 response
// Request:
GET /api/users

// Response (rate limit exceeded):
HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 100          // Max requests per window
X-RateLimit-Remaining: 0        // Requests left
X-RateLimit-Reset: 1620000000   // Unix timestamp when resets
Retry-After: 60                 // Seconds until retry

{
  "statusCode": 429,
  "message": "Too Many Requests",
  "error": "ThrottlerException"
}

// Client handling:
if (response.status === 429) {
  const retryAfter = response.headers.get('Retry-After');
  // Wait before retrying
  await sleep(retryAfter * 1000);
  // Retry request
}
```

### **Rate Limiting Best Practices**

```typescript
// 1. Different limits for different endpoints
const limits = {
  // Strict: Auth endpoints (prevent brute-force)
  '/auth/login': { limit: 5, window: '1 minute' },
  '/auth/register': { limit: 3, window: '1 hour' },
  '/auth/forgot-password': { limit: 3, window: '1 hour' },
  
  // Moderate: Write operations
  '/api/posts': { limit: 100, window: '1 hour' },
  '/api/comments': { limit: 50, window: '1 hour' },
  
  // Generous: Read operations
  '/api/users': { limit: 1000, window: '1 hour' },
  '/api/search': { limit: 5000, window: '1 hour' },
};

// 2. Include rate limit headers
// Always return current limit status
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 42
X-RateLimit-Reset: 1620000060

// 3. Whitelist trusted IPs
// Skip rate limiting for internal services
const trustedIPs = ['10.0.0.0/8', '192.168.0.0/16'];

// 4. Use distributed rate limiting (Redis)
// For multi-instance deployments
// Ensure consistent limits across all servers

// 5. Exponential backoff for retries
// Client should back off after rate limit
const backoff = [1, 2, 4, 8, 16]; // seconds
```

**Interview Tip**: **Rate limiting**: Restricts requests per time window (e.g., 100 req/min) to prevent abuse. **Why important**: Prevent DDoS (1000 req/sec → blocked), brute-force (10 req/min on login = 83 days to crack), ensure fair usage (no user hogs resources), protect infrastructure. **Strategies**: Fixed window (simple), sliding window (smooth), token bucket (allows bursts), leaky bucket (constant rate). **Scopes**: Per IP (anonymous), per user (authenticated), per API key (partners), per endpoint (different limits). **Response**: 429 Too Many Requests with Retry-After header. **Production**: 100 req/min for APIs, 10 req/min for auth, use Redis for distributed systems, whitelist trusted IPs.

</details>

23. How do you implement rate limiting using `@nestjs/throttler`?

<details>
<summary><strong>Answer</strong></summary>

**Install**: `npm install @nestjs/throttler`. **Setup**: Import ThrottlerModule.forRoot({ ttl, limit }), add ThrottlerGuard globally or per controller. **Configuration**: ttl (time window in seconds), limit (max requests). **Usage**: Apply globally with APP_GUARD, or per route with @UseGuards(ThrottlerGuard). **Production**: Use Redis storage for distributed systems, different limits per endpoint.

```typescript
// 1. Install package
// npm install @nestjs/throttler

// 2. Basic setup (app.module.ts)
import { Module } from '@nestjs/common';
import { ThrottlerModule, ThrottlerGuard } from '@nestjs/throttler';
import { APP_GUARD } from '@nestjs/core';

@Module({
  imports: [
    // Configure rate limiting
    ThrottlerModule.forRoot([
      {
        ttl: 60000,   // Time window: 60 seconds (60,000 ms)
        limit: 100,   // Max 100 requests per 60 seconds
      },
    ]),
  ],
  providers: [
    // Apply globally to all routes
    {
      provide: APP_GUARD,
      useClass: ThrottlerGuard,
    },
  ],
})
export class AppModule {}

// Now all routes are rate-limited:
// GET /api/users - Max 100 requests per 60 seconds
// POST /api/posts - Max 100 requests per 60 seconds
// etc.
```

### **Advanced Configuration**

```typescript
// Multiple rate limit tiers
import { ThrottlerModule } from '@nestjs/throttler';

@Module({
  imports: [
    ThrottlerModule.forRoot([
      {
        name: 'short',
        ttl: 1000,    // 1 second
        limit: 10,    // Max 10 requests per second
      },
      {
        name: 'medium',
        ttl: 60000,   // 1 minute
        limit: 100,   // Max 100 requests per minute
      },
      {
        name: 'long',
        ttl: 3600000, // 1 hour
        limit: 1000,  // Max 1000 requests per hour
      },
    ]),
  ],
})
export class AppModule {}

// Checks all three limits:
// Request allowed only if:
// - < 10 req/sec AND
// - < 100 req/min AND
// - < 1000 req/hour
```

### **Environment-based Configuration**

```typescript
// Use ConfigService for different environments
import { ConfigModule, ConfigService } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot(),
    ThrottlerModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (config: ConfigService) => {
        const isProduction = config.get('NODE_ENV') === 'production';
        
        return [{
          // Production: Stricter limits
          // Development: Relaxed limits
          ttl: isProduction ? 60000 : 300000,    // 1 min vs 5 min
          limit: isProduction ? 100 : 1000,       // 100 vs 1000 requests
        }];
      },
    }),
  ],
})
export class AppModule {}
```

### **Redis Storage (Distributed Systems)**

```typescript
// For multiple server instances, use Redis
// npm install @nestjs/throttler-storage-redis ioredis

import { ThrottlerModule } from '@nestjs/throttler';
import { ThrottlerStorageRedisService } from '@nestjs/throttler-storage-redis';
import Redis from 'ioredis';

@Module({
  imports: [
    ThrottlerModule.forRootAsync({
      useFactory: () => ({
        throttlers: [{
          ttl: 60000,
          limit: 100,
        }],
        // Use Redis for distributed rate limiting
        storage: new ThrottlerStorageRedisService(
          new Redis({
            host: process.env.REDIS_HOST,
            port: parseInt(process.env.REDIS_PORT),
            password: process.env.REDIS_PASSWORD,
          }),
        ),
      }),
    }),
  ],
})
export class AppModule {}

// Benefits:
// - Consistent limits across all server instances
// - Prevents users from bypassing limits by hitting different servers
// - Centralized tracking
```

### **Per-Controller Rate Limiting**

```typescript
// Different limits for different controllers
import { Controller, Get, UseGuards } from '@nestjs/common';
import { ThrottlerGuard, Throttle } from '@nestjs/throttler';

// Apply guard to entire controller
@Controller('users')
@UseGuards(ThrottlerGuard)
export class UsersController {
  // Uses default limit (100 req/min)
  @Get()
  findAll() {
    return 'Users list';
  }
  
  // Override default limit for this route
  @Get(':id')
  @Throttle({ default: { limit: 200, ttl: 60000 } })  // 200 req/min
  findOne() {
    return 'User details';
  }
}

// No rate limiting (skip guard)
@Controller('public')
export class PublicController {
  @Get('status')
  getStatus() {
    return 'Server OK';  // No rate limit
  }
}
```

### **Skip Rate Limiting for Specific Routes**

```typescript
import { SkipThrottle } from '@nestjs/throttler';

@Controller('api')
@UseGuards(ThrottlerGuard)  // Apply to controller
export class ApiController {
  @Get('data')
  getData() {
    return 'Rate limited';  // 100 req/min
  }
  
  @Get('health')
  @SkipThrottle()  // Skip rate limiting for health check
  getHealth() {
    return 'OK';  // No rate limit
  }
  
  @Get('public')
  @SkipThrottle({ default: true })  // Skip specific throttler
  getPublic() {
    return 'Public data';  // No rate limit
  }
}
```

### **Custom Rate Limit by User/IP**

```typescript
// Track by user ID instead of IP
import { Injectable, ExecutionContext } from '@nestjs/common';
import { ThrottlerGuard } from '@nestjs/throttler';

@Injectable()
export class UserThrottlerGuard extends ThrottlerGuard {
  // Override: Use user ID instead of IP
  protected async getTracker(req: Record<string, any>): Promise<string> {
    // For authenticated requests, track by user ID
    if (req.user?.id) {
      return `user:${req.user.id}`;
    }
    
    // For anonymous requests, track by IP
    return req.ip;
  }
}

// Apply custom guard
@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: UserThrottlerGuard,  // Use custom guard
    },
  ],
})
export class AppModule {}

// Benefits:
// - Authenticated users: Tracked by ID (can't bypass with VPN)
// - Anonymous users: Tracked by IP
// - More accurate rate limiting
```

### **Custom Error Response**

```typescript
// Customize 429 error response
import { ThrottlerException } from '@nestjs/throttler';
import { ExceptionFilter, Catch, ArgumentsHost } from '@nestjs/common';

@Catch(ThrottlerException)
export class ThrottlerExceptionFilter implements ExceptionFilter {
  catch(exception: ThrottlerException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    
    // Custom response
    response.status(429).json({
      statusCode: 429,
      message: 'Too many requests. Please slow down.',
      error: 'Rate Limit Exceeded',
      retryAfter: 60,  // seconds
      limit: 100,
      window: '1 minute',
      path: request.url,
      timestamp: new Date().toISOString(),
    });
  }
}

// Apply filter globally
app.useGlobalFilters(new ThrottlerExceptionFilter());
```

### **Monitoring Rate Limits**

```typescript
// Track rate limit hits
import { Injectable, ExecutionContext } from '@nestjs/common';
import { ThrottlerGuard, ThrottlerException } from '@nestjs/throttler';

@Injectable()
export class MonitoredThrottlerGuard extends ThrottlerGuard {
  async handleRequest(
    context: ExecutionContext,
    limit: number,
    ttl: number,
    throttler: any,
  ): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    
    try {
      const allowed = await super.handleRequest(context, limit, ttl, throttler);
      
      // Log successful requests
      console.log(`Rate limit check passed for ${request.ip}`);
      
      return allowed;
    } catch (error) {
      // Log rate limit violations
      console.warn(`Rate limit exceeded for ${request.ip} on ${request.url}`);
      
      // Send to monitoring service
      // await this.monitoring.trackRateLimitHit(request.ip, request.url);
      
      throw error;
    }
  }
}
```

### **Endpoint-specific Limits**

```typescript
// Different limits for different endpoint types
@Controller('auth')
@UseGuards(ThrottlerGuard)
export class AuthController {
  // Very strict: Prevent brute-force
  @Post('login')
  @Throttle({ default: { limit: 5, ttl: 60000 } })  // 5 req/min
  async login() {
    return 'Login';
  }
  
  // Strict: Prevent spam
  @Post('register')
  @Throttle({ default: { limit: 3, ttl: 3600000 } })  // 3 req/hour
  async register() {
    return 'Register';
  }
  
  // Moderate: Normal usage
  @Post('forgot-password')
  @Throttle({ default: { limit: 3, ttl: 3600000 } })  // 3 req/hour
  async forgotPassword() {
    return 'Reset email sent';
  }
}

@Controller('api')
@UseGuards(ThrottlerGuard)
export class ApiController {
  // Generous: Read operations
  @Get('posts')
  @Throttle({ default: { limit: 1000, ttl: 60000 } })  // 1000 req/min
  async getPosts() {
    return 'Posts';
  }
  
  // Moderate: Write operations
  @Post('posts')
  @Throttle({ default: { limit: 100, ttl: 3600000 } })  // 100 req/hour
  async createPost() {
    return 'Post created';
  }
}
```

### **Testing Rate Limits**

```bash
# Test rate limiting with curl
for i in {1..110}; do
  curl http://localhost:3000/api/users
  echo "Request $i"
done

# Requests 1-100: 200 OK
# Request 101: 429 Too Many Requests

# Check rate limit headers:
curl -I http://localhost:3000/api/users

HTTP/1.1 200 OK
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 42
X-RateLimit-Reset: 1620000060
```

**Interview Tip**: **Setup**: Install `@nestjs/throttler`, configure ThrottlerModule.forRoot({ ttl, limit }), apply ThrottlerGuard globally (APP_GUARD) or per controller (@UseGuards). **Configuration**: ttl (time window in ms), limit (max requests). **Distributed**: Use Redis storage for multiple servers (consistent limits). **Customization**: Override getTracker() to track by user ID, @Throttle() decorator for per-route limits, @SkipThrottle() to exclude routes. **Production**: Default 100 req/min, strict for auth (5 req/min), generous for reads (1000 req/min), use Redis for scaling. **Response**: 429 with X-RateLimit-* headers.

</details>

24. What is the `@Throttle()` decorator?

<details>
<summary><strong>Answer</strong></summary>

**@Throttle()**: Decorator to **override** default rate limits for specific routes or controllers. **Syntax**: `@Throttle({ name: { limit, ttl } })`. **Use cases**: Stricter limits for auth endpoints (5 req/min), relaxed for read-only (1000 req/min), different tiers for paid users. **Placement**: Can be on method (route) or class (controller). **Production**: Use for sensitive endpoints, public APIs, heavy operations.

```typescript
import { Controller, Get, Post, UseGuards } from '@nestjs/common';
import { Throttle, ThrottlerGuard } from '@nestjs/throttler';

// Global default: 100 requests per 60 seconds
// Set in ThrottlerModule.forRoot([{ ttl: 60000, limit: 100 }])

@Controller('users')
@UseGuards(ThrottlerGuard)
export class UsersController {
  // ✅ Uses default limit (100 req/min)
  @Get()
  findAll() {
    return 'All users';
  }
  
  // ✅ Override: More requests allowed (read-only, cheap operation)
  @Get(':id')
  @Throttle({ default: { limit: 200, ttl: 60000 } })  // 200 req/min
  findOne() {
    return 'User details';
  }
  
  // ✅ Override: Fewer requests (write operation, expensive)
  @Post()
  @Throttle({ default: { limit: 10, ttl: 60000 } })  // 10 req/min
  create() {
    return 'User created';
  }
}
```

### **Multiple Named Throttlers**

```typescript
// Configure multiple throttlers in app.module.ts
import { ThrottlerModule } from '@nestjs/throttler';

@Module({
  imports: [
    ThrottlerModule.forRoot([
      {
        name: 'short',    // Per-second limit
        ttl: 1000,
        limit: 10,
      },
      {
        name: 'medium',   // Per-minute limit
        ttl: 60000,
        limit: 100,
      },
      {
        name: 'long',     // Per-hour limit
        ttl: 3600000,
        limit: 1000,
      },
    ]),
  ],
})
export class AppModule {}

// Use named throttlers in controller
@Controller('api')
@UseGuards(ThrottlerGuard)
export class ApiController {
  // Override multiple throttlers
  @Get('heavy')
  @Throttle({
    short: { limit: 5, ttl: 1000 },      // 5 req/sec
    medium: { limit: 50, ttl: 60000 },   // 50 req/min
    long: { limit: 500, ttl: 3600000 },  // 500 req/hour
  })
  async heavyOperation() {
    return 'Heavy result';
  }
  
  // Override only specific throttler
  @Get('light')
  @Throttle({ short: { limit: 20, ttl: 1000 } })  // Only override short
  async lightOperation() {
    return 'Light result';
  }
}
```

### **Controller-level vs Route-level**

```typescript
// Controller-level: Applies to all routes in controller
@Controller('posts')
@UseGuards(ThrottlerGuard)
@Throttle({ default: { limit: 50, ttl: 60000 } })  // All routes: 50 req/min
export class PostsController {
  @Get()  // 50 req/min
  findAll() {}
  
  @Get(':id')  // 50 req/min
  findOne() {}
  
  // Route-level override: More specific
  @Post()
  @Throttle({ default: { limit: 10, ttl: 60000 } })  // Override: 10 req/min
  create() {}
  
  // Route-level override: Less strict
  @Get('public')
  @Throttle({ default: { limit: 200, ttl: 60000 } })  // Override: 200 req/min
  getPublic() {}
}

// Precedence: Route > Controller > Global
```

### **Strict Limits for Sensitive Endpoints**

```typescript
@Controller('auth')
@UseGuards(ThrottlerGuard)
export class AuthController {
  // ❌ VERY STRICT: Prevent brute-force attacks
  @Post('login')
  @Throttle({ default: { limit: 5, ttl: 60000 } })  // 5 attempts per minute
  async login(@Body() credentials: LoginDto) {
    return this.authService.login(credentials);
  }
  
  // ❌ STRICT: Prevent spam registrations
  @Post('register')
  @Throttle({ default: { limit: 3, ttl: 3600000 } })  // 3 per hour
  async register(@Body() userData: RegisterDto) {
    return this.authService.register(userData);
  }
  
  // ❌ STRICT: Prevent password reset spam
  @Post('forgot-password')
  @Throttle({ default: { limit: 3, ttl: 3600000 } })  // 3 per hour
  async forgotPassword(@Body() email: string) {
    return this.authService.sendResetEmail(email);
  }
  
  // ✅ MODERATE: Token refresh
  @Post('refresh')
  @Throttle({ default: { limit: 20, ttl: 60000 } })  // 20 per minute
  async refreshToken(@Body() refreshToken: string) {
    return this.authService.refreshToken(refreshToken);
  }
}
```

### **Relaxed Limits for Read Operations**

```typescript
@Controller('public')
@UseGuards(ThrottlerGuard)
export class PublicController {
  // ✅ GENEROUS: Read-only, cheap operations
  @Get('posts')
  @Throttle({ default: { limit: 1000, ttl: 60000 } })  // 1000 req/min
  async getPosts() {
    return this.postsService.findAll();
  }
  
  // ✅ GENEROUS: Search (cached)
  @Get('search')
  @Throttle({ default: { limit: 500, ttl: 60000 } })  // 500 req/min
  async search(@Query('q') query: string) {
    return this.searchService.search(query);
  }
  
  // ❌ MODERATE: Expensive operation
  @Get('export')
  @Throttle({ default: { limit: 10, ttl: 3600000 } })  // 10 per hour
  async exportData() {
    return this.exportService.generateExport();
  }
}
```

### **Tiered Rate Limits by User Role**

```typescript
// Custom guard with role-based limits
import { Injectable, ExecutionContext } from '@nestjs/common';
import { ThrottlerGuard, ThrottlerException } from '@nestjs/throttler';
import { Reflector } from '@nestjs/core';

@Injectable()
export class RoleBasedThrottlerGuard extends ThrottlerGuard {
  constructor(private reflector: Reflector) {
    super();
  }

  protected async getThrottlerOptions(
    context: ExecutionContext,
  ): Promise<{ limit: number; ttl: number }> {
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    
    // Different limits based on user tier
    if (user?.role === 'premium') {
      return { limit: 1000, ttl: 60000 };  // 1000 req/min
    } else if (user?.role === 'standard') {
      return { limit: 500, ttl: 60000 };   // 500 req/min
    } else {
      return { limit: 100, ttl: 60000 };   // 100 req/min (free)
    }
  }
}

@Controller('api')
@UseGuards(RoleBasedThrottlerGuard)
export class ApiController {
  @Get('data')
  getData() {
    // Free: 100 req/min
    // Standard: 500 req/min
    // Premium: 1000 req/min
  }
}
```

### **Zero Limit (Disable Rate Limiting)**

```typescript
@Controller('api')
@UseGuards(ThrottlerGuard)
export class ApiController {
  // Normal rate limiting
  @Get('users')
  getUsers() {
    return 'Users';
  }
  
  // Disable rate limiting for this route
  @Get('health')
  @Throttle({ default: { limit: 0, ttl: 0 } })  // No rate limit
  getHealth() {
    return 'OK';
  }
  
  // Alternative: Use @SkipThrottle() (cleaner)
  @Get('status')
  @SkipThrottle()
  getStatus() {
    return 'Status';
  }
}
```

### **Response Headers**

```typescript
// @Throttle() automatically adds headers
@Get('data')
@Throttle({ default: { limit: 100, ttl: 60000 } })
getData() {
  return 'Data';
}

// Response headers:
HTTP/1.1 200 OK
X-RateLimit-Limit: 100          // Max requests
X-RateLimit-Remaining: 42       // Requests left
X-RateLimit-Reset: 1620000060   // Unix timestamp when resets

// When limit exceeded:
HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1620000060
Retry-After: 60                  // Seconds to wait
```

### **Best Practices**

```typescript
// 1. Strict for auth/write
const authLimits = {
  login: { limit: 5, ttl: 60000 },       // 5 req/min (brute-force protection)
  register: { limit: 3, ttl: 3600000 },  // 3 req/hour (spam prevention)
  reset: { limit: 3, ttl: 3600000 },     // 3 req/hour
};

// 2. Moderate for standard APIs
const apiLimits = {
  default: { limit: 100, ttl: 60000 },   // 100 req/min
  write: { limit: 50, ttl: 60000 },      // 50 req/min
};

// 3. Generous for read-only
const readLimits = {
  list: { limit: 1000, ttl: 60000 },     // 1000 req/min
  search: { limit: 500, ttl: 60000 },    // 500 req/min
};

// 4. Very strict for expensive operations
const expensiveLimits = {
  export: { limit: 10, ttl: 3600000 },   // 10 req/hour
  report: { limit: 5, ttl: 3600000 },    // 5 req/hour
  bulk: { limit: 3, ttl: 86400000 },     // 3 req/day
};
```

**Interview Tip**: **@Throttle()**: Decorator to **override** default rate limits per route/controller. **Syntax**: `@Throttle({ default: { limit: N, ttl: MS } })`. **Use cases**: Stricter for auth (5 req/min), relaxed for reads (1000 req/min), different tiers (free/premium). **Placement**: Route (most specific) > Controller > Global default. **Named throttlers**: Multiple limits (@Throttle({ short: {...}, long: {...} })). **Production**: Strict auth (5 req/min), moderate writes (50 req/min), generous reads (1000 req/min), disable with limit: 0 or @SkipThrottle(). **Response**: Adds X-RateLimit-* headers automatically.

</details>

25. Why is rate limiting important for API security and performance?

<details>
<summary><strong>Answer</strong></summary>

**Security**: Prevents **DDoS** attacks (blocks excessive traffic), **brute-force** (limits login attempts = 83 days to crack vs 2 hours), **API abuse** (scraping, spam). **Performance**: Protects **resources** (CPU, DB, memory), ensures **fair usage** (no user hogs capacity), maintains **SLA** (service stays available). **Cost**: Limits **external API** calls (prevents $10k bills), controls **bandwidth**. **Production**: Essential for public APIs, auth endpoints, production systems.

```typescript\n// Security Benefits:\n\n// 1. \u274c Prevent DDoS Attacks\n// Without rate limiting:\nconst ddosAttack = {\n  requests: 100_000,        // requests/second\n  serverCapacity: 1_000,    // requests/second\n  result: 'Server crashes', // 100\u00d7 over capacity\n};\n\n// With rate limiting (100 req/min per IP):\nconst protected = {\n  attackerBlocked: 'After 100 requests',\n  legitimateUsers: 'Unaffected',\n  serverStatus: 'Stable',\n};\n\n// Real-world impact:\n// - Without: Service down for everyone\n// - With: Attackers blocked, service remains available\n\n// 2. \u274c Prevent Brute-Force Attacks\n// Login endpoint without rate limiting:\nconst bruteForce = {\n  attemptsPerSecond: 1000,\n  passwordLength: 8,\n  possiblePasswords: 62 ** 8,  // 218 trillion\n  timeToChance: '7 hours',     // 218T / (1000 * 3600) \u274c\n};\n\n// With rate limiting (5 req/min):\nconst protected = {\n  attemptsPerMinute: 5,\n  timeToChance: '83 years',    // 218T / (5 * 60 * 24 * 365) \u2705\n  // Plus account lockout after 5 failed attempts\n};\n\n// 3. \u274c Prevent API Abuse\n// Scraper without rate limiting:\nconst scraper = {\n  requests: 1_000_000,         // per hour\n  serverCost: '$10,000',       // database overload\n  legitimateUsers: 'Blocked',  // no capacity left\n};\n\n// With rate limiting:\nconst protected = {\n  scrapersBlocked: 'After 100 requests',\n  serverCost: '$100',          // normal load\n  legitimateUsers: 'Normal service',\n};\n```\n\n### **Performance Benefits**\n\n```typescript\n// 1. Protect Resources\n// Database capacity: 1000 queries/second\n// Without rate limiting:\nconst unprotected = {\n  peakTraffic: 10_000,         // requests/second\n  databaseLoad: 10_000,        // queries/second\n  result: 'Database crashes',  // 10\u00d7 over capacity\n  recovery: '30 minutes downtime',\n};\n\n// With rate limiting (100 req/min per user):\nconst protected = {\n  peakTraffic: 10_000,         // requests/second\n  databaseLoad: 800,           // queries/second (within capacity)\n  result: 'Database stable',\n  recovery: 'No downtime',\n};\n\n// 2. Ensure Fair Usage\n// API capacity: 1000 req/sec\n// Without rate limiting:\nconst unfair = {\n  greedyUser: 900,             // req/sec (90% of capacity)\n  normalUsers: 100,            // req/sec total (starved)\n  result: 'Most users blocked',\n};\n\n// With rate limiting (50 req/sec per user):\nconst fair = {\n  maxPerUser: 50,              // req/sec\n  capacity: 1000,              // req/sec total\n  simultaneousUsers: 20,       // 1000 / 50\n  result: 'Fair share for all',\n};\n\n// 3. Maintain SLA (Service Level Agreement)\n// SLA: 99.9% uptime = 8.76 hours downtime/year\n// Without rate limiting:\nconst breachedSLA = {\n  ddosAttacks: 5,              // per year\n  downtimePerAttack: 4,        // hours\n  totalDowntime: 20,           // hours/year\n  slaUptime: '99.77%',        // BREACH! (> 8.76 hours)\n};\n\n// With rate limiting:\nconst metSLA = {\n  ddosAttacks: 5,              // per year\n  attacksMitigated: 5,         // all blocked\n  totalDowntime: 2,            // hours/year (maintenance only)\n  slaUptime: '99.98%',        // \u2705 SLA met\n};\n```\n\n### **Cost Benefits**\n\n```typescript\n// 1. Limit External API Costs\n// Third-party API: $0.01 per request\n// Without rate limiting:\nconst noCostControl = {\n  buggyCode: 'Infinite loop calling API',\n  requests: 1_000_000,         // in 1 hour\n  cost: '$10,000',            // \u274c OUCH!\n};\n\n// With rate limiting (1000 req/hour):\nconst costControl = {\n  buggyCode: 'Same infinite loop',\n  requestsAllowed: 1000,       // limited\n  cost: '$10',                // \u2705 Controlled\n  alert: 'Rate limit hit, investigate',\n};\n\n// 2. Bandwidth Costs\n// Response size: 500 KB\n// Bandwidth cost: $0.10/GB\n// Without rate limiting:\nconst highBandwidth = {\n  requests: 1_000_000,         // per hour\n  bandwidth: '500 GB',         // 1M \u00d7 500KB\n  cost: '$50',                // per hour = $1200/day\n};\n\n// With rate limiting (100 req/min = 6000 req/hour):\nconst controlledBandwidth = {\n  requests: 6000,              // per hour\n  bandwidth: '3 GB',           // 6000 \u00d7 500KB\n  cost: '$0.30',              // per hour = $7/day\n  savings: '$1193/day',       // 99.4% cost reduction!\n};\n```\n\n### **Real-world Attack Scenarios**\n\n```typescript\n// Scenario 1: Credential Stuffing\nconst credentialStuffing = {\n  attack: 'Automated login attempts with stolen credentials',\n  withoutRateLimit: {\n    attempts: 1000,            // per second\n    accounts: 100_000,         // to test\n    duration: '100 seconds',   // 100k / 1000\n    successfulBreaches: '5000', // 5% of accounts compromised\n  },\n  withRateLimit: {\n    limit: '5 attempts per minute',\n    blockedAfter: '5 attempts',\n    duration: '200,000 minutes', // 100k / 5\n    result: 'Attack becomes impractical (138 days)',\n  },\n};\n\n// Scenario 2: Web Scraping\nconst webScraping = {\n  attack: 'Competitor scraping product catalog',\n  withoutRateLimit: {\n    requests: 10_000,          // per minute\n    products: 100_000,         // scraped\n    duration: '10 minutes',    // complete scrape\n    databaseLoad: 'Overloaded',\n  },\n  withRateLimit: {\n    limit: '100 requests per minute',\n    products: 100_000,\n    duration: '1000 minutes',  // 16.7 hours\n    result: 'Scraping detected and blocked',\n  },\n};\n\n// Scenario 3: Resource Exhaustion\nconst resourceExhaustion = {\n  attack: 'Expensive query endpoint abuse',\n  endpoint: '/api/reports/generate',\n  queryCost: '5 seconds CPU time',\n  withoutRateLimit: {\n    requests: 100,             // simultaneous\n    cpuTime: '500 seconds',    // 100 \u00d7 5s\n    cores: 8,                  // CPU cores\n    result: 'Server frozen for 62 seconds', // 500s / 8 cores\n  },\n  withRateLimit: {\n    limit: '10 requests per hour',\n    requestsAllowed: 10,\n    cpuTime: '50 seconds',     // 10 \u00d7 5s\n    result: 'Server responsive (6s load)', // 50s / 8 cores\n  },\n};\n```\n\n### **Rate Limiting Strategies by Endpoint Type**\n\n```typescript\n// Different limits for different security/performance needs\nconst endpointLimits = {\n  // \u274c CRITICAL SECURITY: Auth endpoints\n  authentication: {\n    '/auth/login': { limit: 5, window: '1 minute', reason: 'Brute-force prevention' },\n    '/auth/register': { limit: 3, window: '1 hour', reason: 'Spam prevention' },\n    '/auth/reset': { limit: 3, window: '1 hour', reason: 'Abuse prevention' },\n  },\n  \n  // \u26a0\ufe0f HIGH SECURITY: Write operations\n  write: {\n    '/api/posts': { limit: 50, window: '1 hour', reason: 'Spam prevention' },\n    '/api/upload': { limit: 10, window: '1 hour', reason: 'Resource protection' },\n    '/api/comments': { limit: 30, window: '1 hour', reason: 'Spam prevention' },\n  },\n  \n  // \u2705 MODERATE: Read operations\n  read: {\n    '/api/users': { limit: 1000, window: '1 hour', reason: 'Fair usage' },\n    '/api/posts': { limit: 5000, window: '1 hour', reason: 'Cache-friendly' },\n  },\n  \n  // \u274c EXPENSIVE: Heavy operations\n  expensive: {\n    '/api/export': { limit: 10, window: '1 day', reason: 'Resource intensive' },\n    '/api/report': { limit: 5, window: '1 hour', reason: 'CPU intensive' },\n    '/api/search': { limit: 100, window: '1 minute', reason: 'Database load' },\n  },\n};\n```\n\n### **Monitoring and Alerting**\n\n```typescript\n// Track rate limit violations for security monitoring\nimport { Injectable } from '@nestjs/common';\n\n@Injectable()\nexport class RateLimitMonitoringService {\n  async logViolation(ip: string, endpoint: string, userAgent: string) {\n    console.warn(`Rate limit exceeded:`, {\n      ip,\n      endpoint,\n      userAgent,\n      timestamp: new Date().toISOString(),\n    });\n    \n    // Check for attack patterns\n    const violations = await this.getRecentViolations(ip);\n    \n    if (violations > 10) {\n      // Potential attack - alert security team\n      await this.sendSecurityAlert({\n        ip,\n        violations,\n        message: 'Possible DDoS or brute-force attack',\n      });\n      \n      // Consider IP ban\n      await this.banIP(ip, 3600); // Ban for 1 hour\n    }\n  }\n  \n  async getMetrics() {\n    return {\n      totalRequests: 1_000_000,\n      rateLimitHits: 5000,           // 0.5% hit rate\n      uniqueIPsBlocked: 250,\n      averageRequestsPerUser: 1000,\n      top10Users: [/* users using most requests */],\n    };\n  }\n}\n```\n\n**Interview Tip**: **Security**: Prevents DDoS (blocks excessive traffic), brute-force (5 req/min = 83 years to crack vs 2 hours), API abuse (scrapers, spam). **Performance**: Protects resources (DB, CPU), ensures fair usage (no single user hogs capacity), maintains SLA (99.9% uptime). **Cost**: Controls external API costs ($10 vs $10k with bug), saves bandwidth (99% reduction). **Real-world**: Critical for auth endpoints (5 req/min), moderate for writes (50 req/hour), generous for reads (1000 req/hour), expensive operations (10 req/day). **Production**: Essential for public APIs, prevents resource exhaustion, enables graceful degradation under load.\n\n</details>\n\n## Lazy Loading Modules

26. What is lazy loading of modules?

<details>
<summary><strong>Answer</strong></summary>

**Lazy loading**: Load modules **on-demand** (when needed) instead of **at startup**. **Benefits**: **Faster startup** (don't load unused modules), **lower memory** (only load what's used), **better scalability**. **Use case**: Admin panels, reports, background jobs, feature modules. **Implementation**: Use `LazyModuleLoader` service. **Production**: Lazy-load large/infrequent modules, eager-load core modules.

```typescript
// ❌ Without lazy loading (Eager loading)
// app.module.ts
@Module({
  imports: [
    CoreModule,
    UsersModule,        // Always loaded
    PostsModule,        // Always loaded
    AdminModule,        // Always loaded (rarely used!)
    ReportsModule,      // Always loaded (rarely used!)
    AnalyticsModule,    // Always loaded (rarely used!)
  ],
})
export class AppModule {}

// Startup:
// Loading all modules... (5 seconds)
// Memory: 500 MB
// Problem: Admin/Reports/Analytics loaded but rarely used (waste!)

// ✅ With lazy loading
@Module({
  imports: [
    CoreModule,
    UsersModule,        // Eager (frequently used)
    PostsModule,        // Eager (frequently used)
    // AdminModule,     // Lazy (load on-demand)
    // ReportsModule,   // Lazy (load on-demand)
    // AnalyticsModule, // Lazy (load on-demand)
  ],
})
export class AppModule {}

// Startup:
// Loading core modules... (1 second) ✅ 80% faster!
// Memory: 200 MB ✅ 60% less!
// Admin/Reports/Analytics loaded only when requested
```

### **Benefits of Lazy Loading**

```typescript
// 1. Faster Startup Time
const startupComparison = {
  eagerLoading: {
    modulesLoaded: 20,
    startupTime: '10 seconds',
    firstRequest: '10s + 50ms = 10.05s',
  },
  lazyLoading: {
    modulesLoaded: 5,          // Only core modules
    startupTime: '2 seconds',  // 80% faster!
    firstRequest: '2s + 50ms = 2.05s',
  },
  improvement: '8 seconds faster startup (80%)',
};

// 2. Lower Memory Usage
const memoryComparison = {
  eagerLoading: {
    modulesInMemory: 20,
    memory: '500 MB',
  },
  lazyLoading: {
    modulesInMemory: 5,        // Only loaded modules
    memory: '200 MB',          // 60% less!
    lazyModules: 'Loaded on demand, then cached',
  },
  improvement: '300 MB saved (60%)',
};

// 3. Better Scalability
const scalability = {
  scenario: '10 microservices, each with 20 feature modules',
  eagerLoading: {
    memoryPerInstance: '500 MB',
    instances: 10,
    totalMemory: '5 GB',
    cost: '$500/month',
  },
  lazyLoading: {
    memoryPerInstance: '200 MB',
    instances: 10,
    totalMemory: '2 GB',        // 60% less!
    cost: '$200/month',         // $300 saved/month
  },
};

// 4. Faster Development Iterations
const devExperience = {
  eagerLoading: {
    change: 'Edit one file',
    restart: 'Reload all 20 modules (10s)',
    iterationTime: '10s per change',
  },
  lazyLoading: {
    change: 'Edit one file',
    restart: 'Reload 5 core modules (2s)',
    iterationTime: '2s per change',
    productivity: '5× faster iterations!',
  },
};
```

### **When to Use Lazy Loading**

```typescript
// ✅ GOOD candidates for lazy loading:
const lazyLoadCandidates = [
  {
    module: 'AdminModule',
    reason: 'Rarely accessed (only admins, 1% of requests)',
    impact: 'Large module with many dependencies',
    savings: '100 MB memory, 2s startup',
  },
  {
    module: 'ReportsModule',
    reason: 'Infrequent usage (daily/weekly reports)',
    impact: 'Heavy dependencies (PDF generation, charts)',
    savings: '80 MB memory, 1.5s startup',
  },
  {
    module: 'AnalyticsModule',
    reason: 'Background jobs only (not per-request)',
    impact: 'Complex calculations, large libraries',
    savings: '120 MB memory, 3s startup',
  },
  {
    module: 'ExportModule',
    reason: 'Occasional use (data exports)',
    impact: 'File generation, compression',
    savings: '50 MB memory, 1s startup',
  },
];

// ❌ BAD candidates for lazy loading:
const eagerLoadCandidates = [
  {
    module: 'AuthModule',
    reason: 'Every request needs authentication',
    impact: 'Lazy loading adds latency to every request',
  },
  {
    module: 'DatabaseModule',
    reason: 'Core functionality used everywhere',
    impact: 'Must be available at startup',
  },
  {
    module: 'LoggingModule',
    reason: 'Used by all modules',
    impact: 'Needed immediately',
  },
  {
    module: 'ConfigModule',
    reason: 'Configuration needed at startup',
    impact: 'Can\'t start without it',
  },
];
```

### **Lazy Loading Flow**

```typescript
// Request flow comparison

// Eager loading:
// 1. App starts → Load ALL modules (10s)
// 2. First request to /admin → Instant (0ms) - already loaded
// 3. Total: 10s startup + 0ms = 10s

// Lazy loading:
// 1. App starts → Load CORE modules only (2s)
// 2. First request to /admin → Load AdminModule (500ms)
// 3. Subsequent /admin requests → Instant (0ms) - cached
// 4. Total: 2s startup + 500ms = 2.5s (75% faster!)

// Trade-off:
// - First request to lazy module: Slightly slower (+500ms)
// - Startup time: Much faster (8s saved)
// - Overall: Worth it for infrequent modules
```

### **Memory Impact**

```typescript
// Example app with 10 feature modules
const appModules = {
  core: {
    modules: ['Auth', 'Database', 'Config', 'Logging'],
    memoryPerModule: '20 MB',
    totalMemory: '80 MB',
    loadTime: '1s',
  },
  features: {
    modules: ['Users', 'Posts', 'Comments', 'Notifications'],
    memoryPerModule: '30 MB',
    totalMemory: '120 MB',
    loadTime: '2s',
  },
  infrequent: {
    modules: ['Admin', 'Reports', 'Analytics', 'Export'],
    memoryPerModule: '50 MB',
    totalMemory: '200 MB',
    loadTime: '3s',
  },
};

// Eager loading:
// Total memory: 80 + 120 + 200 = 400 MB
// Startup time: 1 + 2 + 3 = 6s

// Lazy loading (infrequent modules):
// Initial memory: 80 + 120 = 200 MB (50% less!)
// Startup time: 1 + 2 = 3s (50% faster!)
// After first admin request: 200 + 50 = 250 MB
// After first report request: 250 + 50 = 300 MB
// etc.
```

**Interview Tip**: **Lazy loading**: Load modules **on-demand** instead of at startup. **Benefits**: Faster startup (80% faster - 2s vs 10s), lower memory (60% less - 200MB vs 500MB), better scalability, faster dev iterations. **Use for**: Infrequent modules (admin panels, reports, analytics, exports), large modules with heavy dependencies, background jobs. **Don't use for**: Core modules (auth, database, config, logging), frequently used features. **Trade-off**: First request to lazy module +500ms, but overall app starts 8s faster. **Production**: Lazy-load large/infrequent modules, eager-load critical paths.

</details>

27. How do you implement lazy-loaded modules?

<details>
<summary><strong>Answer</strong></summary>

**Implementation**: Use `LazyModuleLoader` service. **Steps**: 1) Remove module from imports, 2) Inject LazyModuleLoader, 3) Call `load()` when needed, 4) Get service from loaded module. **Caching**: Modules loaded once, cached for subsequent calls. **Use case**: Load on-demand (admin routes, background jobs, webhooks). **Production**: Combine with caching, load in background for future requests.

```typescript
// 1. Create a feature module (admin.module.ts)
import { Module } from '@nestjs/common';
import { AdminService } from './admin.service';
import { AdminController } from './admin.controller';

@Module({
  providers: [AdminService],
  controllers: [AdminController],
})
export class AdminModule {}

// 2. DON'T import in app.module.ts (that's eager loading)
@Module({
  imports: [
    // AdminModule,  // ❌ DON'T do this (eager loading)
  ],
})
export class AppModule {}

// 3. Lazy load when needed
import { Injectable } from '@nestjs/common';
import { LazyModuleLoader } from '@nestjs/core';
import { AdminModule } from './admin/admin.module';
import { AdminService } from './admin/admin.service';

@Injectable()
export class AppService {
  constructor(private lazyModuleLoader: LazyModuleLoader) {}

  async loadAdminFeature() {
    // Load module on-demand
    const moduleRef = await this.lazyModuleLoader.load(() => AdminModule);
    
    // Get service from loaded module
    const adminService = moduleRef.get(AdminService);
    
    // Use the service
    return adminService.getAdminData();
  }
}

// First call: Loads module (~500ms)
// Subsequent calls: Returns cached module (~1ms) ✅
```

### **Lazy Loading in Controller**

```typescript
// Controller that lazy-loads admin features
import { Controller, Get } from '@nestjs/common';
import { LazyModuleLoader } from '@nestjs/core';
import { AdminModule } from './admin/admin.module';
import { AdminService } from './admin/admin.service';

@Controller('admin')
export class AppController {
  private adminModuleRef: any;

  constructor(private lazyModuleLoader: LazyModuleLoader) {}

  @Get('dashboard')
  async getAdminDashboard() {
    // Load module if not already loaded
    if (!this.adminModuleRef) {
      this.adminModuleRef = await this.lazyModuleLoader.load(() => AdminModule);
    }
    
    // Get service
    const adminService = this.adminModuleRef.get(AdminService);
    
    return adminService.getDashboard();
  }
  
  @Get('users')
  async getAdminUsers() {
    // Module already loaded (cached)
    if (!this.adminModuleRef) {
      this.adminModuleRef = await this.lazyModuleLoader.load(() => AdminModule);
    }
    
    const adminService = this.adminModuleRef.get(AdminService);
    return adminService.getUsers();
  }
}

// First request to /admin/dashboard: 500ms (loads module)
// Second request to /admin/users: 1ms (module cached) ✅
```

### **Background Job with Lazy Loading**

```typescript
// Load heavy modules only when background job runs
import { Injectable } from '@nestjs/common';
import { Cron } from '@nestjs/schedule';
import { LazyModuleLoader } from '@nestjs/core';
import { ReportsModule } from './reports/reports.module';
import { ReportsService } from './reports/reports.service';

@Injectable()
export class SchedulerService {
  constructor(private lazyModuleLoader: LazyModuleLoader) {}

  // Daily report at 2 AM
  @Cron('0 2 * * *')
  async generateDailyReport() {
    console.log('Loading ReportsModule...');
    
    // Load module only when job runs
    const moduleRef = await this.lazyModuleLoader.load(() => ReportsModule);
    const reportsService = moduleRef.get(ReportsService);
    
    // Generate report
    await reportsService.generateDailyReport();
    
    console.log('Report generated');
  }
}

// Benefits:
// - ReportsModule not loaded at startup (saves 2s, 100MB)
// - Loaded once per day when job runs
// - Cached for future jobs
```

### **Webhook Handler with Lazy Loading**

```typescript
// Load payment processing module only when webhook received
import { Controller, Post, Body } from '@nestjs/common';
import { LazyModuleLoader } from '@nestjs/core';
import { PaymentModule } from './payment/payment.module';
import { PaymentService } from './payment/payment.service';

@Controller('webhooks')
export class WebhooksController {
  private paymentModuleRef: any;

  constructor(private lazyModuleLoader: LazyModuleLoader) {}

  @Post('stripe')
  async handleStripeWebhook(@Body() payload: any) {
    // Load payment module on first webhook
    if (!this.paymentModuleRef) {
      console.log('Loading PaymentModule...');
      this.paymentModuleRef = await this.lazyModuleLoader.load(() => PaymentModule);
    }
    
    const paymentService = this.paymentModuleRef.get(PaymentService);
    return paymentService.processStripeWebhook(payload);
  }
}

// Benefits:
// - PaymentModule loaded only when webhooks arrive
// - Not all apps receive webhooks frequently
// - Saves startup time and memory
```

### **Dynamic Module Loading (Advanced)**

```typescript
// Load different modules based on conditions
import { Injectable } from '@nestjs/common';
import { LazyModuleLoader } from '@nestjs/core';

@Injectable()
export class DynamicFeatureService {
  constructor(private lazyModuleLoader: LazyModuleLoader) {}

  async processRequest(feature: string) {
    let moduleRef: any;

    // Load different module based on feature
    switch (feature) {
      case 'admin':
        const { AdminModule } = await import('./admin/admin.module');
        moduleRef = await this.lazyModuleLoader.load(() => AdminModule);
        break;
      
      case 'reports':
        const { ReportsModule } = await import('./reports/reports.module');
        moduleRef = await this.lazyModuleLoader.load(() => ReportsModule);
        break;
      
      case 'analytics':
        const { AnalyticsModule } = await import('./analytics/analytics.module');
        moduleRef = await this.lazyModuleLoader.load(() => AnalyticsModule);
        break;
      
      default:
        throw new Error(`Unknown feature: ${feature}`);
    }

    // Use loaded module
    return moduleRef;
  }
}

// Benefits:
// - Load only the module you need
// - Dynamic import for code splitting
// - Each module loaded once, then cached
```

### **Caching Loaded Modules**

```typescript
// Reusable lazy loading service with caching
import { Injectable } from '@nestjs/common';
import { LazyModuleLoader, ModuleRef } from '@nestjs/core';

@Injectable()
export class LazyLoadService {
  private loadedModules = new Map<any, ModuleRef>();

  constructor(private lazyModuleLoader: LazyModuleLoader) {}

  async loadModule<T>(moduleClass: any): Promise<ModuleRef> {
    // Check cache first
    if (this.loadedModules.has(moduleClass)) {
      console.log(`Module ${moduleClass.name} already loaded (cached)`);
      return this.loadedModules.get(moduleClass);
    }

    // Load module
    console.log(`Loading module ${moduleClass.name}...`);
    const moduleRef = await this.lazyModuleLoader.load(() => moduleClass);
    
    // Cache for future use
    this.loadedModules.set(moduleClass, moduleRef);
    
    return moduleRef;
  }
  
  getLoadedModules() {
    return Array.from(this.loadedModules.keys()).map(m => m.name);
  }
}

// Usage
@Injectable()
export class AppService {
  constructor(private lazyLoad: LazyLoadService) {}

  async useAdminFeature() {
    const moduleRef = await this.lazyLoad.loadModule(AdminModule);
    const adminService = moduleRef.get(AdminService);
    return adminService.getData();
  }
}
```

### **Pre-warming Lazy Modules**

```typescript
// Load lazy modules in background after startup
import { Injectable, OnApplicationBootstrap } from '@nestjs/common';
import { LazyModuleLoader } from '@nestjs/core';

@Injectable()
export class PrewarmService implements OnApplicationBootstrap {
  constructor(private lazyModuleLoader: LazyModuleLoader) {}

  async onApplicationBootstrap() {
    // Wait for app to start
    setTimeout(() => {
      this.prewarmModules();
    }, 5000);  // 5 seconds after startup
  }

  private async prewarmModules() {
    console.log('Pre-warming lazy modules...');
    
    try {
      // Load modules in background
      const { AdminModule } = await import('./admin/admin.module');
      await this.lazyModuleLoader.load(() => AdminModule);
      console.log('AdminModule pre-warmed');
      
      const { ReportsModule } = await import('./reports/reports.module');
      await this.lazyModuleLoader.load(() => ReportsModule);
      console.log('ReportsModule pre-warmed');
      
      // Now first requests to these modules will be instant
    } catch (error) {
      console.error('Error pre-warming modules:', error);
    }
  }
}

// Benefits:
// - Fast startup (doesn't block app start)
// - Modules ready when first request arrives
// - Best of both worlds
```

### **Error Handling**

```typescript
import { Injectable, Logger } from '@nestjs/common';
import { LazyModuleLoader } from '@nestjs/core';

@Injectable()
export class SafeLazyLoadService {
  private readonly logger = new Logger(SafeLazyLoadService.name);

  constructor(private lazyModuleLoader: LazyModuleLoader) {}

  async loadModuleSafely<T>(moduleClass: any, serviceName: string): Promise<T | null> {
    try {
      this.logger.log(`Loading module ${moduleClass.name}...`);
      
      const moduleRef = await this.lazyModuleLoader.load(() => moduleClass);
      const service = moduleRef.get<T>(serviceName);
      
      this.logger.log(`Module ${moduleClass.name} loaded successfully`);
      return service;
    } catch (error) {
      this.logger.error(`Failed to load module ${moduleClass.name}:`, error.stack);
      
      // Graceful degradation
      return null;
    }
  }
}

// Usage with fallback
@Injectable()
export class AppService {
  constructor(private safeLazyLoad: SafeLazyLoadService) {}

  async getAdminData() {
    const adminService = await this.safeLazyLoad.loadModuleSafely(
      AdminModule,
      'AdminService',
    );
    
    if (!adminService) {
      return { error: 'Admin feature temporarily unavailable' };
    }
    
    return adminService.getData();
  }
}
```

### **Performance Comparison**

```typescript
// Scenario: App with Admin, Reports, Analytics modules

// ❌ Eager loading:
const eagerLoading = {
  startup: {
    time: '10 seconds',
    modulesLoaded: ['Core', 'Admin', 'Reports', 'Analytics'],
  },
  firstAdminRequest: {
    time: '50ms',  // Instant (already loaded)
  },
  totalToFirstRequest: '10,050ms',
};

// ✅ Lazy loading:
const lazyLoading = {
  startup: {
    time: '3 seconds',   // 70% faster!
    modulesLoaded: ['Core'],
  },
  firstAdminRequest: {
    time: '550ms',       // Load AdminModule (500ms) + request (50ms)
  },
  secondAdminRequest: {
    time: '50ms',        // Cached!
  },
  totalToFirstRequest: '3,550ms',  // 65% faster overall!
};
```

**Interview Tip**: **Implementation**: Inject `LazyModuleLoader`, call `load(() => ModuleClass)` on-demand, get services from returned ModuleRef. **Caching**: Modules loaded once, cached automatically (first call 500ms, subsequent 1ms). **Use cases**: Admin panels (load when admin logs in), background jobs (load when job runs), webhooks (load when received), feature flags (load specific features dynamically). **Best practices**: Cache loaded modules, handle errors gracefully, pre-warm in background after startup for zero latency. **Production**: Combine with dynamic imports for code splitting, monitor loading times, provide fallbacks.

</details>

28. When should you use lazy loading?

<details>
<summary><strong>Answer</strong></summary>

**Use lazy loading for**: **Infrequent** modules (admin, reports, analytics - <10% requests), **large** modules (heavy dependencies, >50MB), **background** jobs (scheduled tasks, webhooks), **optional** features (premium, beta). **Don't use for**: **Core** modules (auth, database, logging - every request), **frequent** features (>50% requests), **critical** paths (must be instant). **Decision**: If module used <10% requests AND >20MB size → lazy load. **Production**: Lazy-load admin/reports, eager-load API endpoints.

```typescript
// Decision matrix for lazy loading

// ✅ EXCELLENT candidates:
const excellentCandidates = [
  {
    module: 'AdminModule',
    usage: '1% of requests',        // Rarely used
    size: '100 MB',                 // Large
    startupTime: '3s',
    decision: '✅ LAZY LOAD',
    reason: 'Saves 3s startup, 100MB memory, used rarely',
  },
  {
    module: 'ReportsModule',
    usage: '0.1% of requests',      // Very rare
    size: '80 MB',
    dependencies: ['PDFKit', 'ChartJS', 'ExcelJS'],
    decision: '✅ LAZY LOAD',
    reason: 'Heavy dependencies, infrequent use',
  },
  {
    module: 'AnalyticsModule',
    usage: 'Background jobs only',  // Not per-request
    size: '120 MB',
    runFrequency: 'Once per day',
    decision: '✅ LAZY LOAD',
    reason: 'Not needed at startup, huge memory savings',
  },
  {
    module: 'ExportModule',
    usage: '5% of requests',
    size: '50 MB',
    operation: 'CSV/Excel export',
    decision: '✅ LAZY LOAD',
    reason: 'Occasional use, reasonable size',
  },
];

// ⚠️ MAYBE candidates:
const maybeCandidates = [
  {
    module: 'NotificationsModule',
    usage: '30% of requests',       // Moderate usage
    size: '20 MB',                  // Small-medium
    decision: '⚠️ DEPENDS',
    reason: 'Used often, but could lazy-load for faster startup',
    recommendation: 'Eager load (too frequent)',
  },
  {
    module: 'SearchModule',
    usage: '15% of requests',
    size: '40 MB',
    decision: '⚠️ DEPENDS',
    reason: 'Moderate usage and size',
    recommendation: 'Pre-warm after startup',
  },
];

// ❌ BAD candidates:
const badCandidates = [
  {
    module: 'AuthModule',
    usage: '100% of requests',      // Every request!
    size: '10 MB',
    decision: '❌ EAGER LOAD',
    reason: 'Critical path, used by every request',
  },
  {
    module: 'DatabaseModule',
    usage: '90% of requests',
    size: '5 MB',
    decision: '❌ EAGER LOAD',
    reason: 'Core functionality, must be ready at startup',
  },
  {
    module: 'LoggingModule',
    usage: '100% of requests',
    size: '2 MB',
    decision: '❌ EAGER LOAD',
    reason: 'Needed immediately, tiny size',
  },
  {
    module: 'UsersModule',
    usage: '80% of requests',
    size: '30 MB',
    decision: '❌ EAGER LOAD',
    reason: 'Frequently used, adds latency if lazy',
  },
];
```

### **Cost-Benefit Analysis**

```typescript
// Calculate if lazy loading is worth it
function shouldLazyLoad(module: {
  name: string;
  usagePercent: number;  // % of requests
  sizeM B: number;
  loadTimeMs: number;
}) {
  const criteria = {
    // 1. Infrequent use (<10% of requests)
    infrequent: module.usagePercent < 10,
    
    // 2. Large size (>20 MB)
    large: module.sizeMB > 20,
    
    // 3. Slow to load (>500ms)
    slow: module.loadTimeMs > 500,
  };
  
  // Decision logic
  if (criteria.infrequent && (criteria.large || criteria.slow)) {
    return {
      decision: 'LAZY LOAD ✅',
      reason: 'Infrequent use + (large OR slow)',
    };
  }
  
  if (module.usagePercent > 50) {
    return {
      decision: 'EAGER LOAD ❌',
      reason: 'Used frequently (>50% requests)',
    };
  }
  
  return {
    decision: 'MAYBE ⚠️',
    reason: 'Consider pre-warming or feature flags',
  };
}

// Examples:
shouldLazyLoad({ name: 'AdminModule', usagePercent: 1, sizeMB: 100, loadTimeMs: 3000 });
// → LAZY LOAD ✅ (1% usage + 100MB size)

shouldLazyLoad({ name: 'UsersModule', usagePercent: 80, sizeMB: 30, loadTimeMs: 500 });
// → EAGER LOAD ❌ (80% usage)

shouldLazyLoad({ name: 'SearchModule', usagePercent: 15, sizeMB: 40, loadTimeMs: 800 });
// → MAYBE ⚠️ (15% usage, moderate size)
```

### **Real-world Scenarios**

```typescript
// Scenario 1: E-commerce Platform
const ecommerceApp = {
  coreModules: [
    { name: 'ProductsModule', eager: true, reason: 'Main feature' },
    { name: 'CartModule', eager: true, reason: 'Used by 90% of users' },
    { name: 'OrdersModule', eager: true, reason: 'Critical path' },
    { name: 'AuthModule', eager: true, reason: 'Every request' },
  ],
  lazyModules: [
    { name: 'AdminModule', lazy: true, reason: 'Staff only (1% requests)' },
    { name: 'ReportsModule', lazy: true, reason: 'Daily reports only' },
    { name: 'AnalyticsModule', lazy: true, reason: 'Background processing' },
    { name: 'ExportModule', lazy: true, reason: 'Occasional exports' },
  ],
  benefits: {
    startupTime: '12s → 4s (67% faster)',
    memory: '800MB → 300MB (62% less)',
    cost: '$1200/mo → $500/mo',
  },
};

// Scenario 2: SaaS Dashboard
const saasApp = {
  eagerLoad: [
    'AuthModule',      // Authentication (100% requests)
    'DashboardModule', // Main UI (90% requests)
    'APIModule',       // External API (80% requests)
    'NotificationsModule', // Real-time updates (70% requests)
  ],
  lazyLoad: [
    'BillingModule',   // Billing page (5% requests)
    'SettingsModule',  // Settings page (10% requests)
    'IntegrationsModule', // Third-party integrations (15% requests)
    'DocsModule',      // Documentation viewer (8% requests)
  ],
  strategy: 'Pre-warm frequently accessed lazy modules after 30s',
};

// Scenario 3: Microservice
const microservice = {
  eagerLoad: [
    'CoreModule',      // Essential functionality
    'HealthModule',    // Health checks
    'MetricsModule',   // Monitoring
  ],
  lazyLoad: [
    'MigrationModule', // Database migrations (startup only)
    'SeedModule',      // Data seeding (dev/test only)
    'CleanupModule',   // Cleanup jobs (scheduled)
  ],
  benefits: 'Fast startup in production, load utilities only when needed',
};
```

### **Feature Flags with Lazy Loading**

```typescript
// Load features based on flags
import { Injectable } from '@nestjs/common';
import { LazyModuleLoader } from '@nestjs/core';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class FeatureFlagService {
  constructor(
    private lazyModuleLoader: LazyModuleLoader,
    private config: ConfigService,
  ) {}

  async loadFeature(featureName: string) {
    // Check if feature is enabled
    const enabled = this.config.get(`FEATURES.${featureName}`);
    
    if (!enabled) {
      throw new Error(`Feature ${featureName} is disabled`);
    }
    
    // Load feature module
    switch (featureName) {
      case 'beta-analytics':
        const { BetaAnalyticsModule } = await import('./beta-analytics/beta-analytics.module');
        return this.lazyModuleLoader.load(() => BetaAnalyticsModule);
      
      case 'premium-export':
        const { PremiumExportModule } = await import('./premium-export/premium-export.module');
        return this.lazyModuleLoader.load(() => PremiumExportModule);
      
      default:
        throw new Error(`Unknown feature: ${featureName}`);
    }
  }
}

// Benefits:
// - Beta features only loaded for beta users
// - Premium features only for premium users
// - A/B testing without loading unused code
```

### **Performance Impact**

```typescript
// Calculate performance impact
const performanceImpact = {
  eagerLoading: {
    startup: '15 seconds',
    memory: '1 GB',
    firstRequest: '50ms',
    totalToFirstResponse: '15,050ms',
  },
  lazyLoading: {
    startup: '5 seconds',            // 67% faster
    memory: '300 MB',                // 70% less
    firstAdminRequest: '550ms',      // Load module (500ms)
    secondAdminRequest: '50ms',      // Cached
    totalToFirstResponse: '5,550ms', // 63% faster
  },
  trade-off: {
    downside: '+500ms first request to lazy module',
    upside: '10s faster startup, 700MB memory saved',
    verdict: 'Worth it for infrequent modules',
  },
};
```

### **Decision Flowchart**

```typescript
// Is the module used in <10% of requests?
if (usagePercent < 10) {
  // YES → Is it >20MB or >500ms to load?
  if (sizeMB > 20 || loadTimeMs > 500) {
    return 'LAZY LOAD ✅';  // Infrequent + large/slow
  }
  return 'MAYBE (consider pre-warming)';
}

// NO → Is it used in >50% of requests?
if (usagePercent > 50) {
  return 'EAGER LOAD ❌';  // Too frequent
}

// Between 10-50%
if (sizeMB > 50 && usagePercent < 30) {
  return 'LAZY LOAD with pre-warming';  // Large but moderate use
}

return 'EAGER LOAD ❌';  // Default: eager load
```

**Interview Tip**: **Use for**: Infrequent modules (<10% requests - admin/reports/analytics), large modules (>50MB, heavy dependencies), background jobs (scheduled tasks, webhooks), optional features (premium/beta). **Don't use for**: Core modules (auth/database/logging - 100% requests), frequent features (>50% requests), critical paths (adds latency). **Decision**: Lazy load if <10% usage AND (>20MB OR >500ms load time). **Best practice**: Lazy-load admin panels/reports, eager-load API endpoints, pre-warm frequently accessed lazy modules after startup. **Production**: 67% faster startup, 70% less memory, +500ms first request (acceptable for infrequent features).

</details>

## Streaming

29. What is streaming and when should you use it?

<details>
<summary><strong>Answer</strong></summary>

**Streaming**: Send data in **chunks** (incrementally) instead of loading entire response in memory. **Benefits**: **Low memory** (constant usage), handles **large files** (GB-sized), **faster TTFB** (Time To First Byte), **better UX** (progressive loading). **Use for**: Large files (videos, exports), real-time data (logs, events), database cursors. **Production**: Stream files >10MB, use Node.js Streams API, set proper headers.

```typescript
// ❌ Without streaming (BAD for large files)
@Get('export')
async exportData() {
  // Load ENTIRE file into memory
  const data = await this.generateLargeCSV();  // 500 MB!
  return data;  // ❌ 500 MB in memory, slow response
}

// Memory usage: 500 MB (one file)
// 10 concurrent requests: 5 GB! (OOM crash)
// TTFB: 30 seconds (must generate entire file first)

// ✅ With streaming (GOOD)
@Get('export')
export Data(@Res() res: Response) {
  // Stream data in chunks
  const stream = this.generateLargeCSVStream();
  stream.pipe(res);  // ✅ Constant 10 MB memory
}

// Memory usage: 10 MB (buffer)
// 10 concurrent requests: 100 MB (stable!)
// TTFB: 50ms (start sending immediately)
```

### **When to Use Streaming**

```typescript
// ✅ USE STREAMING:
const streamingUseCases = [
  {
    scenario: 'Large file downloads',
    example: 'Export 1 GB CSV, 500 MB PDF report',
    withoutStreaming: '1 GB memory per request = OOM',
    withStreaming: '10 MB memory per request',
    benefit: '100× less memory',
  },
  {
    scenario: 'Video/audio streaming',
    example: 'Stream video file to client',
    withoutStreaming: 'Must load entire 2 GB video',
    withStreaming: 'Send 1 MB chunks progressively',
    benefit: 'Instant playback, seek support',
  },
  {
    scenario: 'Real-time logs',
    example: 'Stream server logs to dashboard',
    withoutStreaming: 'Buffer all logs, send at end',
    withStreaming: 'Send logs as they appear',
    benefit: 'Real-time updates',
  },
  {
    scenario: 'Database cursors',
    example: 'Export 10M database records',
    withoutStreaming: 'Load all 10M records (20 GB RAM)',
    withStreaming: 'Process 1000 records at a time (20 MB)',
    benefit: '1000× less memory',
  },
  {
    scenario: 'Image processing',
    example: 'Transform and send image',
    withoutStreaming: 'Load → Process → Send (3× memory)',
    withStreaming: 'Stream → Transform → Send (1× memory)',
    benefit: '67% less memory',
  },
];

// ❌ DON'T USE STREAMING:
const nonStreamingUseCases = [
  {
    scenario: 'Small JSON responses',
    size: '< 1 MB',
    reason: 'Overhead > benefit',
  },
  {
    scenario: 'Data that needs processing',
    example: 'Aggregate/transform entire dataset',
    reason: 'Must have complete data',
  },
  {
    scenario: 'Cached responses',
    example: 'Response already in memory/cache',
    reason: 'Already optimized',
  },
];
```

### **Memory Comparison**

```typescript
// Scenario: Export 1 GB CSV file

// Without streaming:
const withoutStreaming = {
  fileSize: '1 GB',
  memoryPerRequest: '1 GB',  // Entire file in memory
  concurrent Requests: 5,
  totalMemory: '5 GB',       // 5 × 1 GB
  serverRAM: '4 GB',
  result: '❌ OUT OF MEMORY CRASH',
};

// With streaming:
const withStreaming = {
  fileSize: '1 GB',
  memoryPerRequest: '10 MB', // Buffer size only
  concurrentRequests: 5,
  totalMemory: '50 MB',      // 5 × 10 MB
  serverRAM: '4 GB',
  result: '✅ STABLE',
  capacityIncrease: '80×',   // 4GB / 50MB = 80 requests
};
```

### **Response Time Comparison**

```typescript
// Large file (500 MB)

// Without streaming:
const bufferResponse = {
  step1: 'Generate entire 500 MB file (30s)',
  step2: 'Load into memory (2s)',
  step3: 'Send to client (20s on slow connection)',
  TTFB: '32 seconds',  // Time to first byte
  totalTime: '52 seconds',
  userExperience: '❌ Blank screen for 32s, then downloads',
};

// With streaming:
const streamedResponse = {
  step1: 'Generate first chunk (50ms)',
  step2: 'Send chunk to client (100ms)',
  step3: 'Continue generating + sending in parallel',
  TTFB: '150ms',       // ✅ 213× faster!
  totalTime: '25 seconds',  // ✅ 2× faster
  userExperience: '✅ Progress bar appears immediately',
};
```

### **Streaming Benefits**

```typescript
// 1. Constant Memory Usage
const memoryBenefit = {
  scenario: 'Download 10 GB file',
  nonStreaming: '10 GB memory',
  streaming: '10 MB memory',
  saving: '99.9%',
};

// 2. Faster Time To First Byte
const ttfbBenefit = {
  scenario: 'Large report generation',
  nonStreaming: 'Wait 30s, then download',
  streaming: 'Start download in 100ms',
  improvement: '300× faster TTFB',
};

// 3. Scalability
const scalabilityBenefit = {
  serverRAM: '8 GB',
  requestSize: '1 GB',
  nonStreaming: '8 concurrent requests max',
  streaming: '800 concurrent requests',  // 10 MB each
  improvement: '100× more capacity',
};

// 4. Better User Experience
const uxBenefit = {
  nonStreaming: 'Blank screen, then instant download',
  streaming: 'Progressive loading, progress bar, cancellable',
  result: 'Users don\'t think app is frozen',
};
```

### **Common Streaming Scenarios**

```typescript
// 1. CSV Export (1M records)
const csvExport = {
  recordCount: 1_000_000,
  recordSize: '1 KB',
  totalSize: '1 GB',
  nonStreaming: {
    memory: '1 GB',
    time: '60s',
    concurrent: '4 requests (4 GB RAM)',
  },
  streaming: {
    memory: '10 MB',
    time: '30s',
    concurrent: '400 requests (4 GB RAM)',
  },
};

// 2. Video Streaming (2 GB movie)
const videoStreaming = {
  fileSize: '2 GB',
  nonStreaming: {
    mustDownload: 'Entire 2 GB before playing',
    time: '5 minutes',
    userExperience: '❌ Wait 5 min',
  },
  streaming: {
    buffering: '1 MB chunks',
    time: '2 seconds to start',
    userExperience: '✅ Instant playback',
    features: 'Seek, pause, resume',
  },
};

// 3. Log Streaming
const logStreaming = {
  logSize: 'Unbounded (continuous)',
  nonStreaming: {
    problem: 'Can\'t buffer infinite logs',
    solution: 'Not possible',
  },
  streaming: {
    approach: 'Send logs as they\'re generated',
    memory: 'Constant 1 MB',
    userExperience: 'Real-time updates',
  },
};
```

**Interview Tip**: **Streaming**: Send data in chunks (incrementally) instead of buffering entire response. **Benefits**: Constant memory (10MB vs 1GB), faster TTFB (100ms vs 30s), handles large files (GB-sized), better UX (progressive loading). **Use for**: Files >10MB (exports, videos, PDFs), real-time data (logs, events, WebSockets), database cursors (millions of records), image/video processing. **Don't use for**: Small responses (<1MB - overhead), data needing aggregation (must see all data), cached responses (already optimized). **Production**: 100× more concurrent requests, 213× faster TTFB, 99.9% less memory.

</details>

30. How do you stream large responses?

<details>
<summary><strong>Answer</strong></summary>

**Implementation**: Use `@Res() res: Response`, create readable stream, pipe to response. **Methods**: StreamableFile (NestJS), Node.js streams, RxJS observables. **Headers**: Set Content-Type, Content-Disposition, Transfer-Encoding: chunked. **Production**: Stream database queries, large JSON, use compression, handle backpressure.

```typescript
import { Controller, Get, Res, StreamableFile } from '@nestjs/common';
import { Response } from 'express';
import { createReadStream } from 'fs';
import { Readable } from 'stream';

// Method 1: StreamableFile (NestJS way)
@Controller('files')
export class FilesController {
  @Get('download')
  getFile(): StreamableFile {
    const file = createReadStream('./large-file.pdf');
    return new StreamableFile(file, {
      type: 'application/pdf',
      disposition: 'attachment; filename="report.pdf"',
    });
  }
}

// Method 2: Manual streaming with pipe
@Controller('data')
export class DataController {
  @Get('export')
  async exportData(@Res() res: Response) {
    res.setHeader('Content-Type', 'text/csv');
    res.setHeader('Content-Disposition', 'attachment; filename="export.csv"');
    res.setHeader('Transfer-Encoding', 'chunked');
    
    // Create stream
    const stream = await this.createDataStream();
    
    // Pipe to response
    stream.pipe(res);
  }
  
  private async createDataStream(): Promise<Readable> {
    const stream = new Readable({
      read() {},
    });
    
    // Generate data in chunks
    const records = await this.getRecordsCount();
    const batchSize = 1000;
    
    for (let i = 0; i < records; i += batchSize) {
      const batch = await this.getRecordsBatch(i, batchSize);
      const csv = this.convertToCSV(batch);
      stream.push(csv);
    }
    
    stream.push(null); // End stream
    return stream;
  }
}
```

**Interview Tip**: Use StreamableFile for files, manual streams for dynamic data. Set proper headers (Content-Type, Transfer-Encoding: chunked). Stream database queries in batches (1000 records/chunk), pipe to response, handle errors with stream.on('error'). Benefits: 100× less memory, instant TTFB.

</details>

31. How do you stream file downloads?

<details>
<summary><strong>Answer</strong></summary>

**Implementation**: Use `createReadStream()`, wrap in StreamableFile or pipe to @Res(). **Headers**: Content-Type (file type), Content-Disposition (attachment; filename), Content-Length (file size). **Features**: Resume support (Range headers), progress tracking, error handling. **Production**: Stream from S3/disk, use compression, handle large files (GB-sized).

```typescript
import { Controller, Get, Param, Res, StreamableFile, Header } from '@nestjs/common';
import { createReadStream, statSync } from 'fs';
import { join } from 'path';

@Controller('downloads')
export class DownloadsController {
  // Simple file download
  @Get('file/:id')
  async downloadFile(@Param('id') id: string): Promise<StreamableFile> {
    const filePath = join(process.cwd(), 'uploads', `${id}.pdf`);
    const file = createReadStream(filePath);
    
    return new StreamableFile(file, {
      type: 'application/pdf',
      disposition: `attachment; filename="document-${id}.pdf"`,
    });
  }
  
  // With file size for progress
  @Get('large/:id')
  async downloadLargeFile(
    @Param('id') id: string,
    @Res({ passthrough: true }) res: Response,
  ): Promise<StreamableFile> {
    const filePath = join(process.cwd(), 'uploads', `${id}.zip`);
    const stats = statSync(filePath);
    
    res.set({
      'Content-Type': 'application/zip',
      'Content-Disposition': `attachment; filename="archive-${id}.zip"`,
      'Content-Length': stats.size,
    });
    
    const file = createReadStream(filePath);
    return new StreamableFile(file);
  }
}
```

**Interview Tip**: Use StreamableFile + createReadStream() for files. Set Content-Length for progress bars, Content-Disposition for download name. Support Range headers for resume. Stream from S3 using AWS SDK streams. Memory: 10MB constant vs file size.

</details>

## Async Operations

32. How do you optimize async operations?

<details>
<summary><strong>Answer</strong></summary>

**Optimization**: Use **Promise.all()** for parallel execution (5× faster), avoid **await in loops** (sequential = slow), use **Promise.allSettled()** for independent operations, **batch** API calls, **cache** results. **Performance**: Sequential = 5s, parallel = 1s. **Production**: Parallelize independent operations, use batching, implement timeouts, handle errors gracefully.

```typescript
// ❌ BAD: Sequential (slow)
async getBadUserData(userId: string) {
  const user = await this.users.findOne(userId);      // 200ms
  const posts = await this.posts.findByUser(userId);  // 300ms
  const comments = await this.comments.findByUser(userId); // 400ms
  
  return { user, posts, comments };
  // Total: 200 + 300 + 400 = 900ms ❌
}

// ✅ GOOD: Parallel (5× faster)
async getGoodUserData(userId: string) {
  const [user, posts, comments] = await Promise.all([
    this.users.findOne(userId),      // 200ms
    this.posts.findByUser(userId),   // 300ms
    this.comments.findByUser(userId), // 400ms
  ]);
  
  return { user, posts, comments };
  // Total: max(200, 300, 400) = 400ms ✅ (2.25× faster!)
}

// ❌ BAD: await in loop (sequential)
async processUsersBad(userIds: string[]) {
  const results = [];
  for (const id of userIds) {
    const user = await this.users.findOne(id); // 100ms each
    results.push(user);
  }
  return results;
  // 10 users: 10 × 100ms = 1000ms ❌
}

// ✅ GOOD: Parallel with Promise.all
async processUsersGood(userIds: string[]) {
  return Promise.all(
    userIds.map(id => this.users.findOne(id)),
  );
  // 10 users: 100ms (all parallel) ✅ (10× faster!)
}
```

**Interview Tip**: Use Promise.all() for independent parallel operations (2-10× faster). Avoid await in loops (sequential). Use Promise.allSettled() when errors shouldn't stop other operations. Batch API calls (100 at a time). Set timeouts to prevent hanging.

</details>

33. What is `Promise.all()` and when to use it for parallel execution?

<details>
<summary><strong>Answer</strong></summary>

**Promise.all()**: Executes promises **in parallel**, waits for **all** to complete. **Returns**: Array of results in same order. **Fails fast**: If one fails, entire Promise.all rejects. **Use for**: Independent operations (fetch user + posts), bulk operations, when all results needed. **Alternative**: Promise.allSettled() for error tolerance, Promise.race() for first result.

```typescript
// Example: Fetch multiple resources
async getDashboard(userId: string) {
  // ✅ All requests run in parallel
  const [user, stats, notifications, settings] = await Promise.all([
    this.users.findOne(userId),           // 200ms
    this.analytics.getUserStats(userId),  // 500ms
    this.notifications.getRecent(userId), // 300ms
    this.settings.getUserSettings(userId), // 100ms
  ]);
  
  return { user, stats, notifications, settings };
  // Total: max(200, 500, 300, 100) = 500ms
  // Sequential: 200 + 500 + 300 + 100 = 1100ms
  // Improvement: 2.2× faster
}

// Use Promise.allSettled() for error tolerance
async getDashboardSafe(userId: string) {
  const results = await Promise.allSettled([
    this.users.findOne(userId),
    this.analytics.getUserStats(userId),
    this.notifications.getRecent(userId),
    this.settings.getUserSettings(userId),
  ]);
  
  // Handle each result
  return {
    user: results[0].status === 'fulfilled' ? results[0].value : null,
    stats: results[1].status === 'fulfilled' ? results[1].value : {},
    notifications: results[2].status === 'fulfilled' ? results[2].value : [],
    settings: results[3].status === 'fulfilled' ? results[3].value : {},
  };
  // If one fails, others still work ✅
}
```

**Interview Tip**: Promise.all() runs promises in parallel, 2-10× faster than sequential. Fails fast if any rejects. Use for independent operations (fetch multiple resources). Promise.allSettled() for error tolerance (partial results). Promise.race() for timeout/fastest response.

</details>

34. Should you use callbacks, promises, or async/await?

<details>
<summary><strong>Answer</strong></summary>

**Use async/await** (modern, readable, error handling with try/catch). **Avoid callbacks** (callback hell, error-first pattern). **Promises** are good but async/await is cleaner. **Performance**: All equivalent (syntax sugar). **Production**: async/await everywhere, promisify legacy callbacks, use Promise.all() for parallel.

```typescript
// ❌ WORST: Callbacks (callback hell)
getUserData(userId, (err, user) => {
  if (err) return handleError(err);
  
  getPosts(user.id, (err, posts) => {
    if (err) return handleError(err);
    
    getComments(posts[0].id, (err, comments) => {
      if (err) return handleError(err);
      
      // Nested 3 levels deep ❌
      return { user, posts, comments };
    });
  });
});

// ⚠️ OK: Promises (better but verbose)
getUserData(userId)
  .then(user => getPosts(user.id))
  .then(posts => getComments(posts[0].id))
  .then(comments => ({ user, posts, comments }))
  .catch(err => handleError(err));

// ✅ BEST: async/await (clean, readable)
async getUserData(userId: string) {
  try {
    const user = await this.users.findOne(userId);
    const posts = await this.posts.findByUser(user.id);
    const comments = await this.comments.findByPost(posts[0].id);
    
    return { user, posts, comments };
  } catch (error) {
    this.handleError(error);
  }
}
```

**Interview Tip**: Use async/await (clean, readable, try/catch). Avoid callbacks (callback hell). Promises OK but async/await cleaner. Performance identical (syntax sugar). Promisify legacy callbacks with util.promisify().

</details>

## Load Balancing & Clustering

35. What is clustering in Node.js?

<details>
<summary><strong>Answer</strong></summary>

**Clustering**: Run multiple Node.js processes (workers) on **same machine** to utilize **all CPU cores**. **Master process** forks workers, distributes requests. **Benefits**: **4-8× throughput** (multi-core), **high availability** (worker crashes, others continue), **zero-downtime** restarts. **Implementation**: cluster module or PM2. **Production**: 1 worker per CPU core, use PM2 for auto-restart.

```typescript
// Without clustering (single process)
// 4-core CPU: Only 1 core used (75% CPU idle) ❌
// Max throughput: 1000 req/sec

// With clustering (4 workers)
// 4-core CPU: All 4 cores used (100% CPU utilized) ✅
// Max throughput: 4000 req/sec (4× improvement!)

// cluster.ts
import * as cluster from 'cluster';
import * as os from 'os';

if (cluster.isMaster) {
  const cpus = os.cpus().length;
  console.log(`Master process starting ${cpus} workers...`);
  
  // Fork workers (one per CPU core)
  for (let i = 0; i < cpus; i++) {
    cluster.fork();
  }
  
  // Worker crashed? Start new one
  cluster.on('exit', (worker) => {
    console.log(`Worker ${worker.process.pid} died, starting new`);
    cluster.fork();
  });
} else {
  // Worker process: Start NestJS app
  async function bootstrap() {
    const app = await NestFactory.create(AppModule);
    await app.listen(3000);
    console.log(`Worker ${process.pid} started`);
  }
  bootstrap();
}
```

**Interview Tip**: Clustering runs multiple Node.js processes (workers) to use all CPU cores. Master forks workers, load balances requests. Benefits: 4-8× throughput (4-core = 4 workers), high availability (worker crash = others continue). Use PM2 for production (auto-restart, monitoring).

</details>
36. How do you use PM2 for clustering NestJS applications?

<details>
<summary><strong>Answer</strong></summary>

**PM2**: Production process manager for Node.js. **Clustering**: `pm2 start app.js -i max` (auto-detects CPU cores). **Features**: Auto-restart on crash, zero-downtime reload (`pm2 reload`), monitoring (`pm2 monit`), logs. **Configuration**: ecosystem.config.js for settings. **Production**: Use cluster mode, enable watch in dev, log rotation, monitoring.

```typescript
// Install PM2
// npm install -g pm2

// Start with clustering (automatic)
pm2 start dist/main.js -i max --name "nestjs-app"
// -i max: Creates 1 worker per CPU core
// --name: App name for pm2 list

// Common PM2 commands
pm2 list              // List all apps
pm2 logs              // View logs
pm2 monit             // Monitor CPU/memory
pm2 reload nestjs-app // Zero-downtime reload
pm2 stop nestjs-app   // Stop app
pm2 restart nestjs-app // Restart app
pm2 delete nestjs-app  // Remove from PM2

// ecosystem.config.js (recommended)
module.exports = {
  apps: [{
    name: 'nestjs-app',
    script: './dist/main.js',
    instances: 'max',  // or 4, or -1 for max
    exec_mode: 'cluster',
    watch: false,  // true in dev
    max_memory_restart: '1G',
    env: {
      NODE_ENV: 'production',
      PORT: 3000,
    },
  }],
};

// Start with config
pm2 start ecosystem.config.js
```

**Interview Tip**: PM2 handles clustering automatically with `-i max` (1 worker/core). Features: auto-restart, zero-downtime reload, monitoring, logs. Use ecosystem.config.js for config. Production: cluster mode, max_memory_restart, log rotation, pm2 startup for auto-start.

</details>

37. What is horizontal scaling vs vertical scaling?

<details>
<summary><strong>Answer</strong></summary>

**Horizontal (scale out)**: Add more **servers/instances** (2 servers → 4 servers). **Vertical (scale up)**: Increase **resources** on same server (2 CPU → 4 CPU, 4GB RAM → 8GB RAM). **Horizontal**: Better (unlimited scaling, fault tolerance, load balancing). **Vertical**: Limited by hardware, single point of failure. **Production**: Horizontal with load balancer (multiple EC2 instances), vertical for databases.

```typescript
// Vertical Scaling (Scale Up)
const verticalScaling = {
  before: {
    servers: 1,
    cpu: '2 cores',
    ram: '4 GB',
    capacity: '1000 req/sec',
  },
  after: {
    servers: 1,  // Same server
    cpu: '8 cores',  // Upgraded
    ram: '16 GB',    // Upgraded
    capacity: '4000 req/sec',
  },
  pros: ['Simple', 'No code changes', 'No load balancer'],
  cons: ['Limited by hardware', 'Expensive', 'Single point of failure', 'Downtime to upgrade'],
  limit: 'Max ~96 cores, 384 GB RAM',
};

// Horizontal Scaling (Scale Out)
const horizontalScaling = {
  before: {
    servers: 1,
    cpuPerServer: '2 cores',
    ramPerServer: '4 GB',
    capacity: '1000 req/sec',
  },
  after: {
    servers: 4,  // Added 3 more servers
    cpuPerServer: '2 cores',  // Same
    ramPerServer: '4 GB',     // Same
    capacity: '4000 req/sec', // 4× capacity
  },
  pros: ['Unlimited scaling', 'Fault tolerant', 'Cheaper', 'No downtime'],
  cons: ['Need load balancer', 'Stateless required', 'More complex'],
  limit: 'No limit (add more servers)',
};
```

**Interview Tip**: Horizontal = add servers (2→4), Vertical = upgrade server (2 CPU→8 CPU). Horizontal better: unlimited scaling, fault tolerance, cheaper. Vertical: limited, single point of failure, downtime. Production: horizontal for apps (load balancer + multiple instances), vertical for databases.

</details>
38. How do you implement load balancing?

<details>
<summary><strong>Answer</strong></summary>

**Load balancer**: Distributes requests across multiple servers. **Methods**: **Nginx** (reverse proxy), **AWS ALB** (Application Load Balancer), **Kubernetes** (Ingress), PM2 cluster mode (single machine). **Algorithms**: Round-robin (equal distribution), least connections, IP hash. **Production**: Use cloud load balancer (ALB/ELB), health checks, sticky sessions for stateful apps.

```typescript
// Nginx load balancer config
upstream nestjs_backend {
  server 10.0.1.10:3000;  // Server 1
  server 10.0.1.11:3000;  // Server 2
  server 10.0.1.12:3000;  // Server 3
  server 10.0.1.13:3000;  // Server 4
}

server {
  listen 80;
  
  location / {
    proxy_pass http://nestjs_backend;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
  }
}

// AWS ALB (Application Load Balancer)
// - Auto-distributes traffic to EC2 instances
// - Health checks (remove unhealthy instances)
// - Auto-scaling (add/remove instances based on load)
// - SSL termination
```

**Interview Tip**: Load balancer distributes traffic across servers. Use Nginx (self-hosted), AWS ALB (cloud), or Kubernetes. Round-robin distributes equally. Health checks remove failed instances. Sticky sessions for stateful apps (session storage).

</details>

## Memory Management

39. What causes memory leaks in Node.js?

<details>
<summary><strong>Answer</strong></summary>

**Memory leaks**: Objects not garbage collected, memory grows until **OOM crash**. **Causes**: **Global variables** (never released), **event listeners** not removed, **closures** holding references, **timers** not cleared, **cache** without limits. **Detection**: Monitor heap size, use `--inspect`, Chrome DevTools. **Production**: Remove listeners (off/removeListener), clear timers, limit cache size, use WeakMap.

```typescript
// ❌ Cause 1: Event listeners not removed
class BadService {
  constructor(private events: EventEmitter) {
    this.events.on('data', this.handleData); // ❌ Never removed
  }
  
  handleData(data: any) {
    // Process data
  }
  // When BadService destroyed, listener still exists = memory leak
}

// ✅ Fix: Remove listeners
class GoodService implements OnDestroy {
  constructor(private events: EventEmitter) {
    this.events.on('data', this.handleData);
  }
  
  onDestroy() {
    this.events.removeListener('data', this.handleData); // ✅ Clean up
  }
}

// ❌ Cause 2: Timers not cleared
class BadTimer {
  constructor() {
    setInterval(() => {
      console.log('Running...'); // ❌ Never cleared
    }, 1000);
  }
}

// ✅ Fix: Clear timers
class GoodTimer implements OnDestroy {
  private interval: NodeJS.Timeout;
  
  constructor() {
    this.interval = setInterval(() => {
      console.log('Running...');
    }, 1000);
  }
  
  onDestroy() {
    clearInterval(this.interval); // ✅ Clean up
  }
}

// ❌ Cause 3: Unbounded cache
const badCache = new Map(); // ❌ Grows forever

function cache(key: string, value: any) {
  badCache.set(key, value); // Never expires
}

// ✅ Fix: Limited cache with expiration
class GoodCache {
  private cache = new Map<string, { value: any; expires: number }>();
  private maxSize = 1000;
  
  set(key: string, value: any, ttl: number = 3600) {
    // Remove oldest if full
    if (this.cache.size >= this.maxSize) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }
    
    this.cache.set(key, {
      value,
      expires: Date.now() + ttl * 1000,
    });
  }
  
  get(key: string): any {
    const item = this.cache.get(key);
    if (!item) return null;
    
    // Check expiration
    if (Date.now() > item.expires) {
      this.cache.delete(key);
      return null;
    }
    
    return item.value;
  }
}
```

**Interview Tip**: Memory leaks = objects not garbage collected. Causes: event listeners not removed, timers not cleared, unbounded cache, global variables, closures. Fix: removeListener() on destroy, clearInterval/clearTimeout, limit cache size (LRU), use WeakMap for references. Detect with heap snapshots, --inspect flag.

</details>
40. How do you detect memory leaks?

<details>
<summary><strong>Answer</strong></summary>

**Detection**: Monitor **heap size** over time (grows = leak), use `process.memoryUsage()`, **heap snapshots** (compare before/after), **Chrome DevTools** (--inspect), **clinic.js**. **Signs**: Memory grows continuously, doesn't return to baseline, OOM crashes. **Production**: Monitor with Datadog/New Relic, alerts on memory threshold, periodic restarts as workaround.

```typescript
// Monitor memory usage
import { Injectable, Logger } from '@nestjs/common';

@Injectable()
export class MemoryMonitorService {
  private readonly logger = new Logger(MemoryMonitorService.name);
  private baseline: number;
  
  startMonitoring() {
    // Record baseline
    this.baseline = process.memoryUsage().heapUsed;
    
    // Check every minute
    setInterval(() => {
      const usage = process.memoryUsage();
      const heapUsed = usage.heapUsed / 1024 / 1024; // MB
      const growth = ((usage.heapUsed - this.baseline) / this.baseline) * 100;
      
      this.logger.log(`Heap: ${heapUsed.toFixed(2)} MB, Growth: ${growth.toFixed(1)}%`);
      
      // Alert if memory doubled
      if (growth > 100) {
        this.logger.error('⚠️ Potential memory leak detected!');
      }
    }, 60000);
  }
}

// Take heap snapshots
// node --inspect dist/main.js
// Chrome: chrome://inspect → Open DevTools → Memory tab
// 1. Take snapshot
// 2. Perform action
// 3. Take another snapshot
// 4. Compare (objects that increased = leak)
```

**Interview Tip**: Monitor process.memoryUsage() over time (grows = leak). Use heap snapshots (before/after comparison). Chrome DevTools with --inspect flag. Signs: memory grows, doesn't GC, OOM crash. Production: Datadog/New Relic monitoring, alerts, pm2 max_memory_restart.

</details>

41. How do you profile memory usage?

<details>
<summary><strong>Answer</strong></summary>

**Profiling**: **Chrome DevTools** (--inspect, heap snapshots, allocation timeline), **clinic.js** (clinic doctor, clinic bubbleprof), `process.memoryUsage()` logging, **heapdump** package. **Analysis**: Find objects consuming most memory, identify retained objects, check garbage collection frequency. **Production**: Profile in staging, use sampling (low overhead), enable on-demand.

```typescript
// Enable Chrome DevTools profiling
// node --inspect dist/main.js
// Chrome: chrome://inspect

// Memory profiling service
import { Injectable } from '@nestjs/common';
import * as v8 from 'v8';
import * as fs from 'fs';

@Injectable()
export class MemoryProfiler {
  takeHeapSnapshot(filename: string) {
    const snapshot = v8.writeHeapSnapshot(filename);
    console.log(`Heap snapshot saved: ${snapshot}`);
  }
  
  getMemoryStats() {
    const usage = process.memoryUsage();
    return {
      rss: `${(usage.rss / 1024 / 1024).toFixed(2)} MB`, // Total
      heapTotal: `${(usage.heapTotal / 1024 / 1024).toFixed(2)} MB`,
      heapUsed: `${(usage.heapUsed / 1024 / 1024).toFixed(2)} MB`, // Used
      external: `${(usage.external / 1024 / 1024).toFixed(2)} MB`,
    };
  }
}

// Endpoint to trigger profiling
@Controller('debug')
export class DebugController {
  constructor(private profiler: MemoryProfiler) {}
  
  @Get('memory')
  getMemory() {
    return this.profiler.getMemoryStats();
  }
  
  @Post('heap-snapshot')
  takeSnapshot() {
    const filename = `heap-${Date.now()}.heapsnapshot`;
    this.profiler.takeHeapSnapshot(filename);
    return { message: 'Snapshot taken', filename };
  }
}
```

**Interview Tip**: Use Chrome DevTools (--inspect) for heap snapshots and allocation timeline. clinic.js for automated analysis. process.memoryUsage() for runtime monitoring. Compare snapshots to find leaks (retained objects). Profile in staging, not production (overhead).

</details>
42. What is garbage collection in Node.js?

<details>
<summary><strong>Answer</strong></summary>

**GC**: Automatic memory cleanup (V8 engine). **Phases**: **Young generation** (new objects, fast GC), **Old generation** (long-lived objects, slow GC). **Algorithm**: Mark-and-sweep (mark reachable, sweep unreachable). **Triggers**: Heap full, allocation threshold. **Tuning**: `--max-old-space-size` (increase heap limit). **Production**: Monitor GC pauses, avoid GC thrashing (too frequent).

```typescript
// GC generations
const gcGenerations = {
  young: {
    name: 'Young generation (Scavenge)',
    objects: 'New objects',
    algorithm: 'Fast copying GC',
    frequency: 'Very frequent',
    pauseTime: '1-10ms',
  },
  old: {
    name: 'Old generation (Mark-Sweep)',
    objects: 'Long-lived objects',
    algorithm: 'Mark-and-sweep',
    frequency: 'Less frequent',
    pauseTime: '100-500ms', // Can cause latency spikes
  },
};

// Increase heap size for large apps
// node --max-old-space-size=4096 dist/main.js  // 4 GB heap

// Monitor GC
// node --trace-gc dist/main.js
// Output: [12345:0x123456] Scavenge 50.2 (55.3) -> 45.1 (56.3) MB, 2.5ms

// GC stats
import { performance, PerformanceObserver } from 'perf_hooks';

const obs = new PerformanceObserver((list) => {
  const entry = list.getEntries()[0];
  console.log(`GC: ${entry.kind}, Duration: ${entry.duration}ms`);
});
obs.observe({ entryTypes: ['gc'], buffered: true });
```

**Interview Tip**: GC automatically frees unused memory. Young generation (new objects, fast 1-10ms), old generation (long-lived, slow 100-500ms). Mark-and-sweep algorithm. Increase heap with --max-old-space-size=4096. Monitor with --trace-gc. Avoid GC thrashing (too frequent = performance issue).

</details>

## Monitoring & Profiling

43. How do you monitor application performance?

<details>
<summary><strong>Answer</strong></summary>

**Monitoring**: **APM tools** (New Relic, Datadog, Dynatrace), **Metrics** (Prometheus + Grafana), **Logging** (Winston, Pino), **Health checks** (/health endpoint), **Custom metrics** (response time, error rate). **Track**: Latency, throughput, error rate, memory, CPU, database queries. **Production**: APM for deep insights, Prometheus for metrics, alerts on thresholds.

```typescript
// Health check endpoint
import { Controller, Get } from '@nestjs/common';
import { HealthCheck, HealthCheckService, TypeOrmHealthIndicator } from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private db: TypeOrmHealthIndicator,
  ) {}
  
  @Get()
  @HealthCheck()
  check() {
    return this.health.check([
      () => this.db.pingCheck('database'),
    ]);
  }
}

// Custom metrics middleware
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class MetricsMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    const start = Date.now();
    
    res.on('finish', () => {
      const duration = Date.now() - start;
      console.log(`${req.method} ${req.path} - ${res.statusCode} - ${duration}ms`);
      
      // Send to monitoring service
      // this.metrics.recordLatency(req.path, duration);
    });
    
    next();
  }
}
```

**Interview Tip**: Use APM (New Relic/Datadog) for traces, errors, performance. Prometheus for metrics (latency, throughput, errors). @nestjs/terminus for health checks. Track: p95 latency, error rate, memory/CPU, slow queries. Set alerts on thresholds (p95 > 1s, errors > 1%).

</details>

44. What tools can you use (New Relic, Datadog, Prometheus)?

<details>
<summary><strong>Answer</strong></summary>

**APM Tools**: **New Relic** (traces, errors, dashboards), **Datadog** (infrastructure + APM), **Dynatrace** (AI-powered), **AppDynamics**. **Metrics**: **Prometheus** (time-series DB) + **Grafana** (dashboards). **Logging**: **ELK Stack** (Elasticsearch, Logstash, Kibana), **Splunk**. **Tracing**: **Jaeger**, **Zipkin**. **Production**: APM for deep insights, Prometheus for metrics, ELK for logs.

```typescript
// New Relic setup
// npm install newrelic
// newrelic.js
exports.config = {
  app_name: ['NestJS App'],
  license_key: 'your-license-key',
  logging: { level: 'info' },
};

// main.ts
require('newrelic'); // Must be first
import { NestFactory } from '@nestjs/core';

// Datadog setup
// npm install dd-trace
import tracer from 'dd-trace';
tracer.init({ service: 'nestjs-app' });

// Prometheus metrics
// npm install @willsoto/nestjs-prometheus prom-client
import { PrometheusModule } from '@willsoto/nestjs-prometheus';

@Module({
  imports: [
    PrometheusModule.register({
      path: '/metrics',
      defaultMetrics: { enabled: true },
    }),
  ],
})
export class AppModule {}

// Custom Prometheus metric
import { Injectable } from '@nestjs/common';
import { InjectMetric } from '@willsoto/nestjs-prometheus';
import { Counter, Histogram } from 'prom-client';

@Injectable()
export class MetricsService {
  constructor(
    @InjectMetric('http_requests_total') private requests: Counter,
    @InjectMetric('http_request_duration') private latency: Histogram,
  ) {}
  
  recordRequest(method: string, path: string, statusCode: number, duration: number) {
    this.requests.inc({ method, path, statusCode });
    this.latency.observe({ method, path }, duration / 1000);
  }
}
```

**Interview Tip**: APM (New Relic/Datadog) for traces, errors, performance dashboards. Prometheus for metrics collection, Grafana for visualization. ELK Stack for centralized logging. Jaeger/Zipkin for distributed tracing. Production: APM + Prometheus + logs = complete observability.

</details>

45. How do you use Chrome DevTools for profiling?

<details>
<summary><strong>Answer</strong></summary>

**Setup**: Run with `node --inspect dist/main.js`, open `chrome://inspect`. **Tools**: **Performance** (CPU profiling, flame graph), **Memory** (heap snapshots, allocation timeline), **Network** (request timing). **Analysis**: Find hot functions (CPU), memory leaks (retained objects), slow queries. **Production**: Profile in staging (overhead in prod), use sampling, on-demand profiling.

```typescript
// Start with --inspect
node --inspect dist/main.js
// Or for remote debugging:
node --inspect=0.0.0.0:9229 dist/main.js

// Chrome DevTools:
// 1. Open chrome://inspect
// 2. Click "inspect" on your app
// 3. Go to Performance or Memory tab

// CPU Profiling:
// Performance tab → Record → Perform action → Stop
// Flame graph shows function call tree
// Find functions consuming most CPU time

// Memory Profiling:
// Memory tab → Take snapshot → Perform action → Take snapshot
// Compare snapshots to find leaks
// Filter by "Detached DOM tree" or "Array" to find leaks

// Programmatic profiling
import { Session } from 'inspector';
import { promisify } from 'util';

const session = new Session();
session.connect();

const post = promisify(session.post).bind(session);

// Start CPU profiling
await post('Profiler.enable');
await post('Profiler.start');

// ... perform operations ...

// Stop and get profile
const { profile } = await post('Profiler.stop');
console.log(JSON.stringify(profile));
```

**Interview Tip**: Use `node --inspect` + chrome://inspect. Performance tab for CPU profiling (flame graph, hot functions). Memory tab for heap snapshots (compare before/after for leaks). Network tab for request timing. Find bottlenecks: CPU (hot functions), memory (retained objects), I/O (slow queries).

</details>
