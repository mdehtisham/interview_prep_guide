# Monitoring & Observability

> Expert / Architect Level (5+ years)

---

## Questions

241. How do you implement error tracking (Sentry, etc.)?
242. How do you monitor React app performance?
243. What is Real User Monitoring (RUM)?
244. How do you implement analytics in React apps?
245. What is Core Web Vitals and how to optimize them?
246. How do you track user behavior in React?
247. How do you implement A/B testing in React?
248. What is OpenTelemetry and its use in React?
249. How do you debug production issues?
250. How do you implement feature usage tracking?

---

## Detailed Answers

### 241. How do you implement error tracking (Sentry, etc.)?

<details>
<summary>View Answer</summary>

**Error tracking** captures and monitors runtime errors in production. **Sentry** is the most popular solution for React applications.

---

#### **1. Sentry Setup**

**Installation:**

```bash
npm install @sentry/react
```

**Basic Configuration:**

```typescript
// src/main.tsx
import * as Sentry from '@sentry/react';

Sentry.init({
  dsn: 'https://your-dsn@sentry.io/project-id',
  environment: process.env.NODE_ENV,
  release: process.env.VITE_APP_VERSION || '1.0.0',
  
  tracesSampleRate: 1.0,
  replaysSessionSampleRate: 0.1,
  replaysOnErrorSampleRate: 1.0,
  
  integrations: [
    new Sentry.BrowserTracing(),
    new Sentry.Replay(),
  ],
  
  beforeSend(event) {
    // Filter browser extension errors
    if (event.exception?.values?.[0]?.stacktrace?.frames?.some(
      frame => frame.filename?.includes('chrome-extension://')
    )) {
      return null;
    }
    return event;
  },
  
  ignoreErrors: [
    'ResizeObserver loop limit exceeded',
    'Non-Error promise rejection captured',
  ],
});
```

---

#### **2. Error Boundary Integration**

```typescript
// src/App.tsx
import * as Sentry from '@sentry/react';

function App() {
  return (
    <Sentry.ErrorBoundary
      fallback={({ error, resetError }) => (
        <ErrorFallback error={error} resetError={resetError} />
      )}
      showDialog
    >
      <AppContent />
    </Sentry.ErrorBoundary>
  );
}
```

**Custom Error Fallback:**

```typescript
function ErrorFallback({ error, resetError }: ErrorFallbackProps) {
  return (
    <div className="error-fallback">
      <h1>Oops! Something went wrong</h1>
      <p>We've been notified and are working on a fix.</p>
      
      <div className="error-actions">
        <button onClick={resetError}>Try Again</button>
        <button onClick={() => window.location.href = '/'}>Go Home</button>
      </div>
    </div>
  );
}
```

---

#### **3. Manual Error Capturing**

```typescript
import * as Sentry from '@sentry/react';

try {
  await fetchData();
} catch (error) {
  Sentry.captureException(error, {
    tags: { section: 'data-fetching' },
    extra: { userId: currentUser.id },
  });
}
```

**Capture Messages:**

```typescript
Sentry.captureMessage('User attempted unauthorized access', {
  level: 'warning',
  tags: { feature: 'authentication' },
});
```

**Custom Hook:**

```typescript
// src/hooks/useErrorTracking.ts
import { useCallback } from 'react';
import * as Sentry from '@sentry/react';

export function useErrorTracking() {
  const captureError = useCallback((error: Error, context?: any) => {
    Sentry.captureException(error, context);
  }, []);

  const captureMessage = useCallback((message: string, level = 'info') => {
    Sentry.captureMessage(message, level);
  }, []);

  return { captureError, captureMessage };
}
```

---

#### **4. Context and Breadcrumbs**

**Set User Context:**

```typescript
Sentry.setUser({
  id: user.id,
  email: user.email,
  username: user.username,
});

// Clear on logout
Sentry.setUser(null);
```

**Add Breadcrumbs:**

```typescript
Sentry.addBreadcrumb({
  category: 'navigation',
  message: 'User navigated to dashboard',
  level: 'info',
  data: { from: '/home', to: '/dashboard' },
});
```

**Custom Breadcrumb Hook:**

```typescript
import { useEffect } from 'react';
import * as Sentry from '@sentry/react';

export function useBreadcrumb(category: string, message: string) {
  useEffect(() => {
    Sentry.addBreadcrumb({
      category,
      message,
      level: 'info',
      timestamp: Date.now() / 1000,
    });
  }, [category, message]);
}
```

---

#### **5. Performance Monitoring**

**Custom Transactions:**

```typescript
const transaction = Sentry.startTransaction({
  name: 'Load Dashboard',
  op: 'data.load',
});

try {
  const userSpan = transaction.startChild({
    op: 'http.client',
    description: 'GET /api/user',
  });
  const user = await fetchUser();
  userSpan.finish();

  return user;
} finally {
  transaction.finish();
}
```

---

#### **6. Source Maps**

**Vite Plugin:**

```bash
npm install @sentry/vite-plugin --save-dev
```

```typescript
// vite.config.ts
import { sentryVitePlugin } from '@sentry/vite-plugin';

export default defineConfig({
  plugins: [
    react(),
    sentryVitePlugin({
      org: 'your-org',
      project: 'your-project',
      authToken: process.env.SENTRY_AUTH_TOKEN,
    }),
  ],
  build: {
    sourcemap: true,
  },
});
```

---

#### **7. Release Tracking**

```typescript
Sentry.init({
  release: `my-app@${process.env.VITE_APP_VERSION}`,
});
```

**Create Release:**

```bash
sentry-cli releases new "my-app@1.0.0"
sentry-cli releases files "my-app@1.0.0" upload-sourcemaps ./dist
sentry-cli releases finalize "my-app@1.0.0"
```

---

#### **8. Real-World Example**

```typescript
// src/services/errorTracking.ts
import * as Sentry from '@sentry/react';

export class ErrorTrackingService {
  private static instance: ErrorTrackingService;

  static getInstance() {
    if (!ErrorTrackingService.instance) {
      ErrorTrackingService.instance = new ErrorTrackingService();
    }
    return ErrorTrackingService.instance;
  }

  setUser(user: { id: string; email: string }) {
    Sentry.setUser({ id: user.id, email: user.email });
  }

  clearUser() {
    Sentry.setUser(null);
  }

  captureError(error: Error, context?: any) {
    Sentry.captureException(error, context);
  }

  captureMessage(message: string, level = 'info') {
    Sentry.captureMessage(message, level);
  }

  addBreadcrumb(breadcrumb: any) {
    Sentry.addBreadcrumb(breadcrumb);
  }
}

export const errorTracking = ErrorTrackingService.getInstance();
```

---

#### **Best Practices**

1. **Filter Sensitive Data**: Remove PII in beforeSend
2. **Sample Errors**: Use sampleRate in production
3. **Set Context**: User, tags, extra data
4. **Upload Source Maps**: For readable stack traces
5. **Track Releases**: Version your deployments
6. **Ignore Noise**: Filter known errors
7. **Add Breadcrumbs**: Track user actions

---

#### **Summary**

**Error Tracking with Sentry:**

1. **Installation**: @sentry/react package
2. **Configuration**: DSN, environment, release
3. **Error Boundaries**: Automatic React error catching
4. **Manual Capturing**: captureException, captureMessage
5. **Context**: User info, breadcrumbs, tags
6. **Performance**: Transaction tracking
7. **Source Maps**: Upload for debugging
8. **Releases**: Version tracking

**Key Features:**
- Automatic error capture
- User context tracking
- Breadcrumbs for debugging
- Performance monitoring
- Source maps
- Session replay

Proper error tracking enables **proactive bug fixing** and provides insights into **real user experiences**.

</details>

---

### 242. How do you monitor React app performance?

<details>
<summary>View Answer</summary>

**Performance monitoring** tracks key metrics to identify bottlenecks and ensure optimal user experience. Key approaches include **Web Vitals**, **React Profiler**, **Performance API**, and **custom metrics**.

---

#### **1. Web Vitals Monitoring**

**Using web-vitals Library:**

```bash
npm install web-vitals
```

```typescript
// src/utils/webVitals.ts
import { onCLS, onFID, onFCP, onLCP, onTTFB, onINP } from 'web-vitals';

function sendToAnalytics(metric: any) {
  // Send to your analytics service
  console.log(metric);
  
  // Example: Send to Google Analytics
  if (window.gtag) {
    window.gtag('event', metric.name, {
      value: Math.round(metric.value),
      metric_id: metric.id,
      metric_value: metric.value,
      metric_delta: metric.delta,
    });
  }
}

export function reportWebVitals() {
  onCLS(sendToAnalytics);  // Cumulative Layout Shift
  onFID(sendToAnalytics);  // First Input Delay
  onFCP(sendToAnalytics);  // First Contentful Paint
  onLCP(sendToAnalytics);  // Largest Contentful Paint
  onTTFB(sendToAnalytics); // Time to First Byte
  onINP(sendToAnalytics);  // Interaction to Next Paint
}
```

**Initialize in App:**

```typescript
// src/main.tsx
import { reportWebVitals } from './utils/webVitals';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);

// Start monitoring
reportWebVitals();
```

---

#### **2. React Profiler API**

**Component Profiling:**

```typescript
// src/components/ProfiledComponent.tsx
import { Profiler, ProfilerOnRenderCallback } from 'react';

const onRenderCallback: ProfilerOnRenderCallback = (
  id,
  phase,
  actualDuration,
  baseDuration,
  startTime,
  commitTime
) => {
  console.log({
    id,
    phase, // "mount" or "update"
    actualDuration, // Time spent rendering
    baseDuration, // Estimated time without memoization
    startTime,
    commitTime,
  });

  // Send to monitoring service
  if (actualDuration > 16) { // Slower than 60fps
    console.warn(`Slow render: ${id} took ${actualDuration}ms`);
  }
};

function App() {
  return (
    <Profiler id="App" onRender={onRenderCallback}>
      <Dashboard />
      <Profiler id="UserList" onRender={onRenderCallback}>
        <UserList />
      </Profiler>
    </Profiler>
  );
}
```

**Custom Profiling Hook:**

```typescript
// src/hooks/usePerformanceMonitor.ts
import { useEffect, useRef } from 'react';

export function usePerformanceMonitor(componentName: string) {
  const renderCount = useRef(0);
  const startTime = useRef(performance.now());

  useEffect(() => {
    renderCount.current++;
    const renderTime = performance.now() - startTime.current;

    console.log(`${componentName} render #${renderCount.current}: ${renderTime}ms`);

    // Track slow renders
    if (renderTime > 100) {
      console.warn(`Slow render detected in ${componentName}`);
    }

    startTime.current = performance.now();
  });

  return renderCount.current;
}

// Usage
function MyComponent() {
  usePerformanceMonitor('MyComponent');
  return <div>Content</div>;
}
```

---

#### **3. Performance Observer API**

**Monitor Long Tasks:**

```typescript
// src/utils/performanceObserver.ts

export function observeLongTasks() {
  if (!('PerformanceObserver' in window)) return;

  const observer = new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
      if (entry.duration > 50) {
        console.warn('Long task detected:', {
          duration: entry.duration,
          startTime: entry.startTime,
          name: entry.name,
        });

        // Send to analytics
        sendMetric({
          name: 'long_task',
          value: entry.duration,
          type: entry.name,
        });
      }
    }
  });

  observer.observe({ entryTypes: ['longtask'] });
}
```

**Monitor Resource Loading:**

```typescript
export function observeResourceTiming() {
  const observer = new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
      const resource = entry as PerformanceResourceTiming;
      
      console.log({
        name: resource.name,
        duration: resource.duration,
        transferSize: resource.transferSize,
        type: resource.initiatorType,
      });

      // Alert on slow resources
      if (resource.duration > 1000) {
        console.warn('Slow resource:', resource.name);
      }
    }
  });

  observer.observe({ entryTypes: ['resource'] });
}
```

**Monitor Navigation Timing:**

```typescript
export function observeNavigationTiming() {
  const observer = new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
      const nav = entry as PerformanceNavigationTiming;
      
      console.log({
        dns: nav.domainLookupEnd - nav.domainLookupStart,
        tcp: nav.connectEnd - nav.connectStart,
        ttfb: nav.responseStart - nav.requestStart,
        download: nav.responseEnd - nav.responseStart,
        domInteractive: nav.domInteractive - nav.fetchStart,
        domComplete: nav.domComplete - nav.fetchStart,
      });
    }
  });

  observer.observe({ entryTypes: ['navigation'] });
}
```

---

#### **4. Custom Performance Metrics**

**Track Custom Metrics:**

```typescript
// src/utils/performanceMetrics.ts

class PerformanceMetrics {
  private marks = new Map<string, number>();

  mark(name: string) {
    const time = performance.now();
    this.marks.set(name, time);
    performance.mark(name);
  }

  measure(name: string, startMark: string, endMark?: string) {
    const startTime = this.marks.get(startMark);
    if (!startTime) {
      console.warn(`Start mark "${startMark}" not found`);
      return;
    }

    const endTime = endMark ? this.marks.get(endMark) : performance.now();
    if (!endTime) {
      console.warn(`End mark "${endMark}" not found`);
      return;
    }

    const duration = endTime - startTime;

    performance.measure(name, startMark, endMark);

    console.log(`${name}: ${duration}ms`);

    return duration;
  }

  clear() {
    this.marks.clear();
    performance.clearMarks();
    performance.clearMeasures();
  }
}

export const metrics = new PerformanceMetrics();
```

**Usage:**

```typescript
import { metrics } from './utils/performanceMetrics';

function Dashboard() {
  useEffect(() => {
    metrics.mark('dashboard-start');

    loadDashboardData().then(() => {
      metrics.mark('dashboard-data-loaded');
      metrics.measure('dashboard-load', 'dashboard-start', 'dashboard-data-loaded');
    });
  }, []);

  return <div>Dashboard</div>;
}
```

---

#### **5. Performance Budget Monitoring**

```typescript
// src/utils/performanceBudget.ts

interface PerformanceBudget {
  LCP: number;  // ms
  FID: number;  // ms
  CLS: number;  // score
  TTI: number;  // ms
  bundleSize: number; // bytes
}

const budget: PerformanceBudget = {
  LCP: 2500,
  FID: 100,
  CLS: 0.1,
  TTI: 3800,
  bundleSize: 200 * 1024, // 200KB
};

export function checkPerformanceBudget(metric: string, value: number): boolean {
  const budgetValue = budget[metric as keyof PerformanceBudget];
  
  if (budgetValue === undefined) return true;

  const withinBudget = value <= budgetValue;

  if (!withinBudget) {
    console.warn(`Performance budget exceeded for ${metric}:`, {
      actual: value,
      budget: budgetValue,
      exceeded: value - budgetValue,
    });
  }

  return withinBudget;
}
```

---

#### **6. Real-World Performance Dashboard**

```typescript
// src/components/PerformanceDashboard.tsx
import { useState, useEffect } from 'react';
import { onCLS, onFID, onLCP, onTTFB } from 'web-vitals';

interface Metric {
  name: string;
  value: number;
  rating: 'good' | 'needs-improvement' | 'poor';
}

function PerformanceDashboard() {
  const [metrics, setMetrics] = useState<Metric[]>([]);

  useEffect(() => {
    const updateMetric = (metric: any) => {
      setMetrics((prev) => {
        const existing = prev.findIndex((m) => m.name === metric.name);
        const newMetric: Metric = {
          name: metric.name,
          value: metric.value,
          rating: metric.rating,
        };

        if (existing >= 0) {
          const updated = [...prev];
          updated[existing] = newMetric;
          return updated;
        }
        return [...prev, newMetric];
      });
    };

    onLCP(updateMetric);
    onFID(updateMetric);
    onCLS(updateMetric);
    onTTFB(updateMetric);
  }, []);

  const getRatingColor = (rating: string) => {
    switch (rating) {
      case 'good': return 'green';
      case 'needs-improvement': return 'orange';
      case 'poor': return 'red';
      default: return 'gray';
    }
  };

  if (process.env.NODE_ENV !== 'development') return null;

  return (
    <div className="performance-dashboard">
      <h3>Performance Metrics</h3>
      <div className="metrics-grid">
        {metrics.map((metric) => (
          <div key={metric.name} className="metric-card">
            <div className="metric-name">{metric.name}</div>
            <div
              className="metric-value"
              style={{ color: getRatingColor(metric.rating) }}
            >
              {Math.round(metric.value)}
            </div>
            <div className="metric-rating">{metric.rating}</div>
          </div>
        ))}
      </div>
    </div>
  );
}

export default PerformanceDashboard;
```

---

#### **7. Integration with Monitoring Tools**

**Send to Sentry:**

```typescript
import * as Sentry from '@sentry/react';
import { onCLS, onFID, onLCP } from 'web-vitals';

function sendToSentry(metric: any) {
  Sentry.addBreadcrumb({
    category: 'web-vitals',
    message: `${metric.name}: ${metric.value}`,
    level: metric.rating === 'poor' ? 'warning' : 'info',
    data: {
      value: metric.value,
      rating: metric.rating,
      delta: metric.delta,
    },
  });

  // Create transaction for performance tracking
  const transaction = Sentry.startTransaction({
    name: metric.name,
    op: 'web-vitals',
  });
  transaction.setMeasurement(metric.name, metric.value, 'millisecond');
  transaction.finish();
}

onLCP(sendToSentry);
onFID(sendToSentry);
onCLS(sendToSentry);
```

**Send to Custom Backend:**

```typescript
function sendToBackend(metric: any) {
  fetch('/api/metrics', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      name: metric.name,
      value: metric.value,
      rating: metric.rating,
      id: metric.id,
      navigationType: metric.navigationType,
      timestamp: Date.now(),
      url: window.location.href,
      userAgent: navigator.userAgent,
    }),
  });
}
```

---

#### **8. Component-Level Performance Tracking**

```typescript
// src/hoc/withPerformanceTracking.tsx
import { ComponentType, useEffect, useRef } from 'react';

export function withPerformanceTracking<P extends object>(
  Component: ComponentType<P>,
  componentName: string
) {
  return function PerformanceTrackedComponent(props: P) {
    const mountTime = useRef(performance.now());
    const renderCount = useRef(0);

    useEffect(() => {
      renderCount.current++;
      const renderTime = performance.now() - mountTime.current;

      // Log first render time
      if (renderCount.current === 1) {
        console.log(`${componentName} initial render: ${renderTime}ms`);
      }

      // Track slow updates
      if (renderCount.current > 1 && renderTime > 16) {
        console.warn(`${componentName} slow update: ${renderTime}ms`);
      }

      mountTime.current = performance.now();
    });

    return <Component {...props} />;
  };
}

// Usage
const TrackedDashboard = withPerformanceTracking(Dashboard, 'Dashboard');
```

---

#### **Best Practices**

1. **Track Core Web Vitals**: LCP, FID, CLS
2. **Use React Profiler**: Identify slow components
3. **Monitor Long Tasks**: Tasks > 50ms block main thread
4. **Set Performance Budgets**: Define acceptable thresholds
5. **Track Custom Metrics**: Business-critical operations
6. **Sample in Production**: Don't track 100% of users
7. **Correlate with Errors**: Link performance to error rates
8. **Monitor Real Users**: Synthetic tests don't show real experience

---

#### **Summary**

**React Performance Monitoring:**

1. **Web Vitals**: Track LCP, FID, CLS, TTFB
2. **React Profiler**: Component render performance
3. **Performance API**: Marks, measures, observers
4. **Custom Metrics**: Track specific operations
5. **Performance Budgets**: Define thresholds
6. **Integration**: Send to Sentry, DataDog, etc.
7. **Real-time Dashboard**: Development monitoring

**Key Metrics:**
- **LCP**: Largest Contentful Paint (< 2.5s)
- **FID**: First Input Delay (< 100ms)
- **CLS**: Cumulative Layout Shift (< 0.1)
- **TTFB**: Time to First Byte (< 800ms)
- **TTI**: Time to Interactive (< 3.8s)

**Tools:**
- web-vitals: Official Web Vitals library
- React Profiler: Built-in profiling
- Performance API: Browser native
- Sentry: Error + performance monitoring
- DataDog RUM: Full-stack monitoring

Proper performance monitoring ensures **optimal user experience** and helps identify **bottlenecks before they impact users**.

</details>

---

### 243. What is Real User Monitoring (RUM)?

<details>
<summary>View Answer</summary>

**Real User Monitoring (RUM)** captures actual user interactions and performance metrics from real browsers, providing insights into how users experience your application in production.

---

#### **1. RUM vs Synthetic Monitoring**

| Aspect | RUM | Synthetic Monitoring |
|--------|-----|---------------------|
| **Data Source** | Real users | Simulated scripts |
| **Coverage** | All users, devices, networks | Predefined scenarios |
| **Cost** | Higher (per session) | Lower (fixed scripts) |
| **Accuracy** | Real experience | Controlled environment |
| **Use Case** | Production monitoring | Pre-deployment testing |

---

#### **2. Core RUM Metrics**

**Performance Metrics:**
- Page load time
- Time to Interactive (TTI)
- First Contentful Paint (FCP)
- Largest Contentful Paint (LCP)
- First Input Delay (FID)
- Cumulative Layout Shift (CLS)

**User Experience Metrics:**
- Session duration
- Page views per session
- Bounce rate
- Error rate
- User flows
- Geographic distribution

---

#### **3. Basic RUM Implementation**

```typescript
// src/utils/rum.ts

interface RUMData {
  sessionId: string;
  userId?: string;
  pageUrl: string;
  timestamp: number;
  metrics: {
    loadTime: number;
    fcp: number;
    lcp: number;
    fid: number;
    cls: number;
  };
  userAgent: string;
  viewport: string;
  connection: string;
}

class RUM {
  private sessionId: string;
  private data: RUMData[] = [];

  constructor() {
    this.sessionId = this.generateSessionId();
    this.init();
  }

  private generateSessionId(): string {
    return `session-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }

  private init() {
    // Capture page load
    window.addEventListener('load', () => {
      this.capturePageLoad();
    });

    // Capture navigation
    this.observeNavigation();

    // Capture errors
    window.addEventListener('error', (e) => {
      this.captureError(e);
    });

    // Capture unhandled rejections
    window.addEventListener('unhandledrejection', (e) => {
      this.captureRejection(e);
    });

    // Send data periodically
    setInterval(() => this.flush(), 30000); // Every 30s

    // Send data before unload
    window.addEventListener('beforeunload', () => this.flush());
  }

  private capturePageLoad() {
    const navigation = performance.getEntriesByType('navigation')[0] as PerformanceNavigationTiming;
    
    if (!navigation) return;

    const data: RUMData = {
      sessionId: this.sessionId,
      pageUrl: window.location.href,
      timestamp: Date.now(),
      metrics: {
        loadTime: navigation.loadEventEnd - navigation.fetchStart,
        fcp: 0, // Will be updated by observer
        lcp: 0,
        fid: 0,
        cls: 0,
      },
      userAgent: navigator.userAgent,
      viewport: `${window.innerWidth}x${window.innerHeight}`,
      connection: (navigator as any).connection?.effectiveType || 'unknown',
    };

    this.data.push(data);
  }

  private observeNavigation() {
    const observer = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        console.log('Navigation:', entry);
      }
    });

    observer.observe({ entryTypes: ['navigation'] });
  }

  private captureError(error: ErrorEvent) {
    this.send({
      type: 'error',
      sessionId: this.sessionId,
      message: error.message,
      stack: error.error?.stack,
      url: window.location.href,
      timestamp: Date.now(),
    });
  }

  private captureRejection(event: PromiseRejectionEvent) {
    this.send({
      type: 'unhandledRejection',
      sessionId: this.sessionId,
      reason: event.reason,
      url: window.location.href,
      timestamp: Date.now(),
    });
  }

  private flush() {
    if (this.data.length === 0) return;

    this.send({
      type: 'rum',
      data: this.data,
    });

    this.data = [];
  }

  private send(data: any) {
    // Use sendBeacon for reliable delivery
    if (navigator.sendBeacon) {
      navigator.sendBeacon('/api/rum', JSON.stringify(data));
    } else {
      fetch('/api/rum', {
        method: 'POST',
        body: JSON.stringify(data),
        keepalive: true,
      });
    }
  }

  setUserId(userId: string) {
    this.data.forEach((d) => (d.userId = userId));
  }
}

export const rum = new RUM();
```

---

#### **4. RUM with DataDog**

**Installation:**

```bash
npm install @datadog/browser-rum
```

**Configuration:**

```typescript
// src/utils/datadog.ts
import { datadogRum } from '@datadog/browser-rum';

datadogRum.init({
  applicationId: 'your-app-id',
  clientToken: 'your-client-token',
  site: 'datadoghq.com',
  service: 'my-react-app',
  env: process.env.NODE_ENV,
  version: process.env.VITE_APP_VERSION,
  
  sessionSampleRate: 100,
  sessionReplaySampleRate: 20,
  
  trackUserInteractions: true,
  trackResources: true,
  trackLongTasks: true,
  
  defaultPrivacyLevel: 'mask-user-input',
});

datadogRum.startSessionReplayRecording();
```

**Custom Actions:**

```typescript
import { datadogRum } from '@datadog/browser-rum';

// Track custom user actions
datadogRum.addAction('checkout_completed', {
  orderId: '12345',
  amount: 99.99,
});

// Add user context
datadogRum.setUser({
  id: user.id,
  email: user.email,
  name: user.name,
  plan: user.subscriptionTier,
});

// Add custom context
datadogRum.addRumGlobalContext('env', 'production');
```

---

#### **5. RUM with Google Analytics 4**

```typescript
// src/utils/ga4.ts

export function initGA4() {
  // Load GA4
  const script = document.createElement('script');
  script.src = `https://www.googletagmanager.com/gtag/js?id=${GA_MEASUREMENT_ID}`;
  script.async = true;
  document.head.appendChild(script);

  window.dataLayer = window.dataLayer || [];
  function gtag(...args: any[]) {
    window.dataLayer.push(args);
  }

  gtag('js', new Date());
  gtag('config', GA_MEASUREMENT_ID, {
    send_page_view: true,
    custom_map: {
      dimension1: 'user_type',
      dimension2: 'subscription_tier',
    },
  });

  // Track Web Vitals
  onCLS((metric) => {
    gtag('event', 'web_vitals', {
      name: metric.name,
      value: Math.round(metric.value * 1000),
      event_category: 'Web Vitals',
      event_label: metric.id,
      non_interaction: true,
    });
  });

  onFID((metric) => {
    gtag('event', 'web_vitals', {
      name: metric.name,
      value: Math.round(metric.value),
      event_category: 'Web Vitals',
    });
  });

  onLCP((metric) => {
    gtag('event', 'web_vitals', {
      name: metric.name,
      value: Math.round(metric.value),
      event_category: 'Web Vitals',
    });
  });
}
```

---

#### **6. Session Recording**

**Using rrweb (session replay):**

```bash
npm install rrweb
```

```typescript
// src/utils/sessionRecording.ts
import * as rrweb from 'rrweb';

class SessionRecorder {
  private events: any[] = [];
  private stopFn: (() => void) | null = null;

  start() {
    this.stopFn = rrweb.record({
      emit: (event) => {
        this.events.push(event);

        // Send in batches
        if (this.events.length >= 100) {
          this.flush();
        }
      },
      sampling: {
        // Sample mouse movements
        mousemove: true,
        mouseInteraction: true,
        scroll: 150, // ms
        input: 'last', // Record last input
      },
      recordCanvas: true,
    });
  }

  stop() {
    this.stopFn?.();
    this.flush();
  }

  private flush() {
    if (this.events.length === 0) return;

    fetch('/api/session-recording', {
      method: 'POST',
      body: JSON.stringify({ events: this.events }),
    });

    this.events = [];
  }
}

export const sessionRecorder = new SessionRecorder();
```

---

#### **7. User Journey Tracking**

```typescript
// src/utils/userJourney.ts

interface JourneyStep {
  timestamp: number;
  path: string;
  action: string;
  duration: number;
  metadata?: Record<string, any>;
}

class UserJourneyTracker {
  private journey: JourneyStep[] = [];
  private currentStepStart: number = Date.now();

  trackStep(path: string, action: string, metadata?: any) {
    const now = Date.now();
    const duration = now - this.currentStepStart;

    this.journey.push({
      timestamp: now,
      path,
      action,
      duration,
      metadata,
    });

    this.currentStepStart = now;

    // Send completed journeys
    if (this.journey.length >= 10) {
      this.send();
    }
  }

  getJourney(): JourneyStep[] {
    return [...this.journey];
  }

  private send() {
    fetch('/api/user-journey', {
      method: 'POST',
      body: JSON.stringify({
        journey: this.journey,
        sessionId: sessionStorage.getItem('sessionId'),
      }),
    });

    this.journey = [];
  }
}

export const journeyTracker = new UserJourneyTracker();

// Usage with React Router
function App() {
  const location = useLocation();

  useEffect(() => {
    journeyTracker.trackStep(location.pathname, 'navigate');
  }, [location]);

  return <Routes>{/* ... */}</Routes>;
}
```

---

#### **8. Real-World RUM Integration**

```typescript
// src/services/monitoring.ts
import { datadogRum } from '@datadog/browser-rum';
import * as Sentry from '@sentry/react';
import { onCLS, onFID, onLCP, onTTFB } from 'web-vitals';

class MonitoringService {
  init() {
    this.initDatadog();
    this.initSentry();
    this.trackWebVitals();
    this.trackUserBehavior();
  }

  private initDatadog() {
    datadogRum.init({
      applicationId: import.meta.env.VITE_DD_APP_ID,
      clientToken: import.meta.env.VITE_DD_CLIENT_TOKEN,
      site: 'datadoghq.com',
      service: 'my-app',
      env: import.meta.env.VITE_ENV,
      sessionSampleRate: 100,
      sessionReplaySampleRate: 20,
      trackUserInteractions: true,
      trackResources: true,
      trackLongTasks: true,
    });
  }

  private initSentry() {
    Sentry.init({
      dsn: import.meta.env.VITE_SENTRY_DSN,
      environment: import.meta.env.VITE_ENV,
      tracesSampleRate: 0.1,
    });
  }

  private trackWebVitals() {
    const sendMetric = (metric: any) => {
      // Send to DataDog
      datadogRum.addAction(metric.name, {
        value: metric.value,
        rating: metric.rating,
      });

      // Send to Sentry
      Sentry.addBreadcrumb({
        category: 'web-vitals',
        message: `${metric.name}: ${metric.value}`,
        level: metric.rating === 'poor' ? 'warning' : 'info',
      });
    };

    onLCP(sendMetric);
    onFID(sendMetric);
    onCLS(sendMetric);
    onTTFB(sendMetric);
  }

  private trackUserBehavior() {
    // Track clicks
    document.addEventListener('click', (e) => {
      const target = e.target as HTMLElement;
      datadogRum.addAction('click', {
        element: target.tagName,
        text: target.textContent?.slice(0, 50),
      });
    });

    // Track visibility changes
    document.addEventListener('visibilitychange', () => {
      datadogRum.addAction('visibility_change', {
        hidden: document.hidden,
      });
    });
  }

  setUser(user: { id: string; email: string }) {
    datadogRum.setUser({
      id: user.id,
      email: user.email,
    });

    Sentry.setUser({
      id: user.id,
      email: user.email,
    });
  }

  trackEvent(name: string, properties?: Record<string, any>) {
    datadogRum.addAction(name, properties);
  }
}

export const monitoring = new MonitoringService();
```

---

#### **Best Practices**

1. **Sample Production Traffic**: Don't track 100% of users
2. **Protect Privacy**: Mask sensitive data
3. **Track Critical Paths**: Focus on key user journeys
4. **Set Alerts**: Notify on performance degradation
5. **Correlate Metrics**: Link performance to business outcomes
6. **Use Session Replay Wisely**: Only on errors or sampled
7. **Monitor Geographic Distribution**: Performance varies by region
8. **Track Device Types**: Mobile vs desktop differences

---

#### **Summary**

**Real User Monitoring (RUM):**

1. **Definition**: Monitoring actual user experiences in production
2. **Core Metrics**: Web Vitals, page load, user interactions
3. **Implementation**: Custom or third-party (DataDog, New Relic)
4. **Session Recording**: Replay user sessions for debugging
5. **User Journey**: Track complete user flows
6. **Privacy**: Mask sensitive information
7. **Integration**: Combine with error tracking

**Key Components:**
- Performance metrics collection
- User interaction tracking
- Session recording
- Error correlation
- Geographic and device analysis
- Custom event tracking

**Popular RUM Tools:**
- **DataDog RUM**: Full-stack monitoring
- **New Relic**: Application performance
- **Google Analytics 4**: User behavior + performance
- **Sentry**: Error + performance tracking
- **LogRocket**: Session replay + analytics

RUM provides **real-world performance data** and helps identify issues that only occur in production with actual users.

</details>

---

### 244. How do you implement analytics in React apps?

<details>
<summary>View Answer</summary>

**Analytics** track user behavior, measure engagement, and provide insights into how users interact with your app. Popular solutions include **Google Analytics**, **Mixpanel**, **Segment**, and **Amplitude**.

---

#### **1. Google Analytics 4 (GA4)**

**Installation:**

```bash
npm install react-ga4
```

**Setup:**

```typescript
// src/utils/analytics/ga4.ts
import ReactGA from 'react-ga4';

const GA_MEASUREMENT_ID = import.meta.env.VITE_GA_MEASUREMENT_ID;

export function initGA4() {
  ReactGA.initialize(GA_MEASUREMENT_ID, {
    gaOptions: {
      send_page_view: false, // Manually control page views
    },
  });
}

export function trackPageView(path: string, title?: string) {
  ReactGA.send({ hitType: 'pageview', page: path, title });
}

export function trackEvent(category: string, action: string, label?: string, value?: number) {
  ReactGA.event({
    category,
    action,
    label,
    value,
  });
}

export function setUserProperties(properties: Record<string, any>) {
  ReactGA.set(properties);
}

export function setUserId(userId: string) {
  ReactGA.set({ userId });
}
```

**React Router Integration:**

```typescript
// src/App.tsx
import { useEffect } from 'react';
import { useLocation } from 'react-router-dom';
import { initGA4, trackPageView } from './utils/analytics/ga4';

function App() {
  const location = useLocation();

  useEffect(() => {
    initGA4();
  }, []);

  useEffect(() => {
    trackPageView(location.pathname + location.search);
  }, [location]);

  return <Routes>{/* ... */}</Routes>;
}
```

**Track Custom Events:**

```typescript
import { trackEvent } from './utils/analytics/ga4';

function ProductCard({ product }: { product: Product }) {
  const handleAddToCart = () => {
    trackEvent('Ecommerce', 'Add to Cart', product.name, product.price);
    addToCart(product);
  };

  return (
    <div>
      <h3>{product.name}</h3>
      <button onClick={handleAddToCart}>Add to Cart</button>
    </div>
  );
}
```

---

#### **2. Mixpanel**

**Installation:**

```bash
npm install mixpanel-browser
```

**Setup:**

```typescript
// src/utils/analytics/mixpanel.ts
import mixpanel from 'mixpanel-browser';

const MIXPANEL_TOKEN = import.meta.env.VITE_MIXPANEL_TOKEN;

export function initMixpanel() {
  mixpanel.init(MIXPANEL_TOKEN, {
    debug: process.env.NODE_ENV === 'development',
    track_pageview: true,
    persistence: 'localStorage',
  });
}

export function identifyUser(userId: string, properties?: Record<string, any>) {
  mixpanel.identify(userId);
  if (properties) {
    mixpanel.people.set(properties);
  }
}

export function trackEvent(eventName: string, properties?: Record<string, any>) {
  mixpanel.track(eventName, properties);
}

export function setUserProperties(properties: Record<string, any>) {
  mixpanel.people.set(properties);
}

export function trackPageView(pageName: string) {
  mixpanel.track_pageview({ page: pageName });
}
```

**Usage:**

```typescript
import { trackEvent, identifyUser } from './utils/analytics/mixpanel';

function LoginForm() {
  const handleLogin = async (credentials: Credentials) => {
    const user = await login(credentials);
    
    // Identify user
    identifyUser(user.id, {
      email: user.email,
      name: user.name,
      signupDate: user.createdAt,
      plan: user.subscriptionTier,
    });

    // Track login event
    trackEvent('User Logged In', {
      method: 'email',
      timestamp: Date.now(),
    });
  };

  return <form>{/* ... */}</form>;
}
```

---

#### **3. Segment**

**Installation:**

```bash
npm install @segment/analytics-next
```

**Setup:**

```typescript
// src/utils/analytics/segment.ts
import { AnalyticsBrowser } from '@segment/analytics-next';

const SEGMENT_WRITE_KEY = import.meta.env.VITE_SEGMENT_WRITE_KEY;

export const analytics = AnalyticsBrowser.load({
  writeKey: SEGMENT_WRITE_KEY,
});

export function trackPage(name: string, properties?: Record<string, any>) {
  analytics.page(name, properties);
}

export function trackEvent(event: string, properties?: Record<string, any>) {
  analytics.track(event, properties);
}

export function identifyUser(userId: string, traits?: Record<string, any>) {
  analytics.identify(userId, traits);
}

export function trackGroup(groupId: string, traits?: Record<string, any>) {
  analytics.group(groupId, traits);
}
```

**Usage:**

```typescript
import { trackEvent, identifyUser } from './utils/analytics/segment';

function CheckoutPage() {
  const handlePurchase = async (order: Order) => {
    await processPayment(order);

    // Track purchase
    trackEvent('Order Completed', {
      orderId: order.id,
      revenue: order.total,
      products: order.items.map(item => ({
        productId: item.id,
        name: item.name,
        price: item.price,
        quantity: item.quantity,
      })),
    });
  };

  return <CheckoutForm onSubmit={handlePurchase} />;
}
```

---

#### **4. Custom Analytics Hook**

```typescript
// src/hooks/useAnalytics.ts
import { useCallback } from 'react';
import * as ga4 from '../utils/analytics/ga4';
import * as mixpanel from '../utils/analytics/mixpanel';
import * as segment from '../utils/analytics/segment';

export function useAnalytics() {
  const trackEvent = useCallback((event: string, properties?: Record<string, any>) => {
    // Send to all platforms
    ga4.trackEvent('Custom', event, JSON.stringify(properties));
    mixpanel.trackEvent(event, properties);
    segment.trackEvent(event, properties);
  }, []);

  const trackPageView = useCallback((path: string) => {
    ga4.trackPageView(path);
    mixpanel.trackPageView(path);
    segment.trackPage(path);
  }, []);

  const identifyUser = useCallback((userId: string, traits?: Record<string, any>) => {
    ga4.setUserId(userId);
    mixpanel.identifyUser(userId, traits);
    segment.identifyUser(userId, traits);
  }, []);

  return { trackEvent, trackPageView, identifyUser };
}
```

**Usage:**

```typescript
function FeatureButton() {
  const { trackEvent } = useAnalytics();

  const handleClick = () => {
    trackEvent('Feature Used', {
      feature: 'export',
      timestamp: Date.now(),
    });
    exportData();
  };

  return <button onClick={handleClick}>Export</button>;
}
```

---

#### **5. E-commerce Tracking**

**Enhanced E-commerce with GA4:**

```typescript
// src/utils/analytics/ecommerce.ts
import ReactGA from 'react-ga4';

export function trackProductView(product: Product) {
  ReactGA.event('view_item', {
    currency: 'USD',
    value: product.price,
    items: [{
      item_id: product.id,
      item_name: product.name,
      item_category: product.category,
      price: product.price,
    }],
  });
}

export function trackAddToCart(product: Product, quantity: number) {
  ReactGA.event('add_to_cart', {
    currency: 'USD',
    value: product.price * quantity,
    items: [{
      item_id: product.id,
      item_name: product.name,
      item_category: product.category,
      price: product.price,
      quantity,
    }],
  });
}

export function trackPurchase(order: Order) {
  ReactGA.event('purchase', {
    transaction_id: order.id,
    value: order.total,
    currency: 'USD',
    tax: order.tax,
    shipping: order.shipping,
    items: order.items.map(item => ({
      item_id: item.productId,
      item_name: item.name,
      price: item.price,
      quantity: item.quantity,
    })),
  });
}

export function trackBeginCheckout(cart: Cart) {
  ReactGA.event('begin_checkout', {
    currency: 'USD',
    value: cart.total,
    items: cart.items.map(item => ({
      item_id: item.productId,
      item_name: item.name,
      price: item.price,
      quantity: item.quantity,
    })),
  });
}
```

---

#### **6. Funnel Tracking**

```typescript
// src/utils/analytics/funnel.ts

type FunnelStep = 'view' | 'add_to_cart' | 'checkout' | 'payment' | 'complete';

class FunnelTracker {
  private currentStep: FunnelStep | null = null;
  private sessionData: Record<string, any> = {};

  startFunnel(data?: Record<string, any>) {
    this.currentStep = 'view';
    this.sessionData = { ...data, startTime: Date.now() };
    this.trackStep('view');
  }

  nextStep(step: FunnelStep, data?: Record<string, any>) {
    this.currentStep = step;
    this.sessionData = { ...this.sessionData, ...data };
    this.trackStep(step);
  }

  completeFunnel(data?: Record<string, any>) {
    this.nextStep('complete', data);
    
    const duration = Date.now() - this.sessionData.startTime;
    
    // Track completion
    trackEvent('Funnel Completed', {
      duration,
      steps: this.getStepHistory(),
      ...this.sessionData,
    });

    this.reset();
  }

  abandonFunnel() {
    if (!this.currentStep) return;

    trackEvent('Funnel Abandoned', {
      step: this.currentStep,
      ...this.sessionData,
    });

    this.reset();
  }

  private trackStep(step: FunnelStep) {
    trackEvent('Funnel Step', {
      step,
      ...this.sessionData,
    });
  }

  private getStepHistory(): FunnelStep[] {
    // Implementation to track step history
    return [];
  }

  private reset() {
    this.currentStep = null;
    this.sessionData = {};
  }
}

export const funnelTracker = new FunnelTracker();
```

---

#### **7. Real-World Analytics Service**

```typescript
// src/services/analytics.ts
import ReactGA from 'react-ga4';
import mixpanel from 'mixpanel-browser';
import { AnalyticsBrowser } from '@segment/analytics-next';

class AnalyticsService {
  private initialized = false;
  private queue: Array<() => void> = [];

  init() {
    if (this.initialized) return;

    // Initialize GA4
    ReactGA.initialize(import.meta.env.VITE_GA_MEASUREMENT_ID);

    // Initialize Mixpanel
    mixpanel.init(import.meta.env.VITE_MIXPANEL_TOKEN);

    // Initialize Segment
    AnalyticsBrowser.load({
      writeKey: import.meta.env.VITE_SEGMENT_WRITE_KEY,
    });

    this.initialized = true;

    // Process queued events
    this.queue.forEach(fn => fn());
    this.queue = [];
  }

  trackPageView(path: string, title?: string) {
    this.execute(() => {
      ReactGA.send({ hitType: 'pageview', page: path, title });
      mixpanel.track_pageview({ page: path });
    });
  }

  trackEvent(event: string, properties?: Record<string, any>) {
    this.execute(() => {
      ReactGA.event(event, properties);
      mixpanel.track(event, properties);
    });
  }

  identifyUser(userId: string, traits?: Record<string, any>) {
    this.execute(() => {
      ReactGA.set({ userId });
      mixpanel.identify(userId);
      if (traits) {
        mixpanel.people.set(traits);
      }
    });
  }

  trackTiming(category: string, variable: string, value: number) {
    this.execute(() => {
      ReactGA.send({
        hitType: 'timing',
        timingCategory: category,
        timingVar: variable,
        timingValue: value,
      });
    });
  }

  trackError(error: Error, fatal = false) {
    this.execute(() => {
      ReactGA.send({
        hitType: 'exception',
        exDescription: error.message,
        exFatal: fatal,
      });

      mixpanel.track('Error', {
        message: error.message,
        stack: error.stack,
        fatal,
      });
    });
  }

  private execute(fn: () => void) {
    if (this.initialized) {
      fn();
    } else {
      this.queue.push(fn);
    }
  }
}

export const analytics = new AnalyticsService();
```

**Usage:**

```typescript
// src/App.tsx
import { useEffect } from 'react';
import { useLocation } from 'react-router-dom';
import { analytics } from './services/analytics';

function App() {
  const location = useLocation();
  const { user } = useAuth();

  useEffect(() => {
    analytics.init();
  }, []);

  useEffect(() => {
    if (user) {
      analytics.identifyUser(user.id, {
        email: user.email,
        name: user.name,
        plan: user.subscriptionTier,
      });
    }
  }, [user]);

  useEffect(() => {
    analytics.trackPageView(location.pathname);
  }, [location]);

  return <Routes>{/* ... */}</Routes>;
}
```

---

#### **8. Privacy and Consent**

```typescript
// src/components/CookieConsent.tsx
import { useState } from 'react';
import { analytics } from '../services/analytics';

function CookieConsent() {
  const [accepted, setAccepted] = useState(
    localStorage.getItem('analytics-consent') === 'true'
  );

  const handleAccept = () => {
    localStorage.setItem('analytics-consent', 'true');
    setAccepted(true);
    analytics.init();
  };

  const handleReject = () => {
    localStorage.setItem('analytics-consent', 'false');
    setAccepted(false);
  };

  if (accepted || localStorage.getItem('analytics-consent')) {
    return null;
  }

  return (
    <div className="cookie-consent">
      <p>We use cookies to improve your experience and analytics.</p>
      <button onClick={handleAccept}>Accept</button>
      <button onClick={handleReject}>Reject</button>
    </div>
  );
}
```

---

#### **Best Practices**

1. **Track Key Events**: Focus on business-critical actions
2. **User Privacy**: Respect GDPR/CCPA, get consent
3. **Event Naming**: Use consistent naming conventions
4. **Data Quality**: Validate before sending
5. **Performance**: Batch events, use sendBeacon
6. **Testing**: Verify tracking in development
7. **Documentation**: Document all tracked events
8. **Segmentation**: Track user properties for analysis

---

#### **Summary**

**Analytics Implementation:**

1. **Google Analytics 4**: Page views, events, e-commerce
2. **Mixpanel**: User behavior, funnels, cohorts
3. **Segment**: Single API for multiple platforms
4. **Custom Hooks**: useAnalytics for easy integration
5. **E-commerce**: Track product views, cart, purchases
6. **Funnel Tracking**: Monitor user journeys
7. **Privacy**: Cookie consent, GDPR compliance

**Key Events to Track:**
- Page views
- User authentication (login/logout)
- Feature usage
- Form submissions
- Errors
- E-commerce actions
- User journeys/funnels

**Popular Tools:**
- **Google Analytics**: Free, comprehensive
- **Mixpanel**: User behavior, retention
- **Segment**: Central analytics hub
- **Amplitude**: Product analytics
- **Heap**: Automatic event capture

Proper analytics implementation provides **actionable insights** into user behavior and helps **optimize product decisions**.

</details>

---

### 245. What is Core Web Vitals and how to optimize them?

<details>
<summary>View Answer</summary>

**Core Web Vitals** are Google's metrics for measuring user experience on the web. They consist of three key metrics: **LCP** (loading), **FID/INP** (interactivity), and **CLS** (visual stability).

---

#### **1. Core Web Vitals Explained**

**Largest Contentful Paint (LCP):**
- **Definition**: Time until largest content element is visible
- **Good**: < 2.5 seconds
- **Needs Improvement**: 2.5 - 4.0 seconds
- **Poor**: > 4.0 seconds
- **Measures**: Loading performance

**First Input Delay (FID) / Interaction to Next Paint (INP):**
- **FID**: Time from first user interaction to browser response
- **INP**: Latency of all interactions (replacing FID)
- **Good**: FID < 100ms, INP < 200ms
- **Needs Improvement**: FID 100-300ms, INP 200-500ms
- **Poor**: FID > 300ms, INP > 500ms
- **Measures**: Interactivity/Responsiveness

**Cumulative Layout Shift (CLS):**
- **Definition**: Sum of all unexpected layout shifts
- **Good**: < 0.1
- **Needs Improvement**: 0.1 - 0.25
- **Poor**: > 0.25
- **Measures**: Visual stability

---

#### **2. Measuring Core Web Vitals**

**Using web-vitals Library:**

```typescript
// src/utils/coreWebVitals.ts
import { onLCP, onFID, onCLS, onINP } from 'web-vitals';

interface VitalMetric {
  name: string;
  value: number;
  rating: 'good' | 'needs-improvement' | 'poor';
  delta: number;
  id: string;
}

function sendToAnalytics(metric: VitalMetric) {
  console.log(metric);
  
  // Send to your analytics
  fetch('/api/vitals', {
    method: 'POST',
    body: JSON.stringify(metric),
    headers: { 'Content-Type': 'application/json' },
  });
}

export function measureCoreWebVitals() {
  onLCP(sendToAnalytics);
  onFID(sendToAnalytics);
  onCLS(sendToAnalytics);
  onINP(sendToAnalytics); // New metric replacing FID
}
```

**Initialize:**

```typescript
// src/main.tsx
import { measureCoreWebVitals } from './utils/coreWebVitals';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);

measureCoreWebVitals();
```

---

#### **3. Optimizing LCP (Largest Contentful Paint)**

**Common Issues:**
- Slow server response
- Render-blocking resources
- Slow resource load times
- Client-side rendering delays

**Optimization Strategies:**

**A. Optimize Images:**

```typescript
// Use next-gen formats and lazy loading
function OptimizedImage({ src, alt }: ImageProps) {
  return (
    <picture>
      <source srcSet={`${src}.avif`} type="image/avif" />
      <source srcSet={`${src}.webp`} type="image/webp" />
      <img
        src={`${src}.jpg`}
        alt={alt}
        loading="lazy"
        decoding="async"
      />
    </picture>
  );
}
```

**B. Preload Critical Resources:**

```html
<!-- In index.html -->
<head>
  <!-- Preload hero image -->
  <link rel="preload" as="image" href="/hero.webp" />
  
  <!-- Preload critical fonts -->
  <link rel="preload" as="font" href="/fonts/main.woff2" type="font/woff2" crossorigin />
  
  <!-- Preconnect to API -->
  <link rel="preconnect" href="https://api.example.com" />
</head>
```

**C. Code Splitting:**

```typescript
// Lazy load non-critical components
import { lazy, Suspense } from 'react';

const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}
```

**D. Server-Side Rendering (SSR):**

```typescript
// Next.js example
export async function getServerSideProps() {
  const data = await fetchCriticalData();
  
  return {
    props: { data },
  };
}

function Page({ data }: PageProps) {
  // Data is already available on first render
  return <div>{data.title}</div>;
}
```

**E. Optimize Fonts:**

```css
/* Use font-display: swap */
@font-face {
  font-family: 'MyFont';
  src: url('/fonts/myfont.woff2') format('woff2');
  font-display: swap; /* Show fallback font immediately */
}
```

---

#### **4. Optimizing FID/INP (Interactivity)**

**Common Issues:**
- Long-running JavaScript
- Large JavaScript bundles
- Heavy event handlers
- Main thread blocking

**Optimization Strategies:**

**A. Code Splitting:**

```typescript
// Split large bundles
import { lazy } from 'react';

const HeavyComponent = lazy(() => import('./HeavyComponent'));
```

**B. Optimize Event Handlers:**

```typescript
// Debounce expensive operations
import { debounce } from 'lodash';
import { useMemo } from 'react';

function SearchInput() {
  const handleSearch = useMemo(
    () => debounce((query: string) => {
      performSearch(query);
    }, 300),
    []
  );

  return <input onChange={(e) => handleSearch(e.target.value)} />;
}
```

**C. Use Web Workers:**

```typescript
// src/workers/heavyComputation.worker.ts
self.addEventListener('message', (e) => {
  const result = performHeavyComputation(e.data);
  self.postMessage(result);
});

// Usage
import Worker from './workers/heavyComputation.worker?worker';

function Component() {
  const processData = (data: any) => {
    const worker = new Worker();
    
    worker.postMessage(data);
    
    worker.onmessage = (e) => {
      console.log('Result:', e.data);
      worker.terminate();
    };
  };

  return <button onClick={() => processData(data)}>Process</button>;
}
```

**D. Break Up Long Tasks:**

```typescript
// Use scheduler API (experimental)
async function processLargeDataset(items: any[]) {
  for (const item of items) {
    await processItem(item);
    
    // Yield to browser
    if ('scheduler' in window) {
      await (window as any).scheduler.yield();
    } else {
      await new Promise(resolve => setTimeout(resolve, 0));
    }
  }
}
```

**E. Optimize React Renders:**

```typescript
import { memo, useMemo, useCallback } from 'react';

const ExpensiveList = memo(({ items }: { items: Item[] }) => {
  return (
    <ul>
      {items.map((item) => (
        <ExpensiveItem key={item.id} item={item} />
      ))}
    </ul>
  );
});

const ExpensiveItem = memo(({ item }: { item: Item }) => {
  const computed = useMemo(() => expensiveCalculation(item), [item]);
  return <li>{computed}</li>;
});
```

---

#### **5. Optimizing CLS (Cumulative Layout Shift)**

**Common Issues:**
- Images without dimensions
- Ads/embeds/iframes without dimensions
- Dynamically injected content
- Web fonts causing FOIT/FOUT

**Optimization Strategies:**

**A. Set Image Dimensions:**

```typescript
// Always specify width and height
function ImageWithDimensions({ src, alt }: ImageProps) {
  return (
    <img
      src={src}
      alt={alt}
      width={800}
      height={600}
      style={{ maxWidth: '100%', height: 'auto' }}
    />
  );
}

// Or use aspect ratio
const ImageContainer = styled.div`
  aspect-ratio: 16 / 9;
  width: 100%;
  
  img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
`;
```

**B. Reserve Space for Dynamic Content:**

```typescript
// Skeleton loader
function ArticleCard({ isLoading, article }: Props) {
  if (isLoading) {
    return (
      <div className="article-card" style={{ minHeight: '200px' }}>
        <div className="skeleton skeleton-image" style={{ height: '120px' }} />
        <div className="skeleton skeleton-title" style={{ height: '24px' }} />
        <div className="skeleton skeleton-text" style={{ height: '16px' }} />
      </div>
    );
  }

  return (
    <div className="article-card">
      <img src={article.image} alt={article.title} />
      <h3>{article.title}</h3>
      <p>{article.excerpt}</p>
    </div>
  );
}
```

**C. Avoid Inserting Content Above Existing Content:**

```typescript
// Load new content at the bottom
function InfiniteList() {
  const [items, setItems] = useState<Item[]>([]);

  const loadMore = async () => {
    const newItems = await fetchItems();
    // Append to end, not beginning
    setItems((prev) => [...prev, ...newItems]);
  };

  return (
    <div>
      {items.map((item) => <Item key={item.id} {...item} />)}
      <button onClick={loadMore}>Load More</button>
    </div>
  );
}
```

**D. Use CSS Transform for Animations:**

```css
/* Good: Use transform (doesn't trigger layout) */
.animate {
  transform: translateY(10px);
  transition: transform 0.3s;
}

/* Bad: Don't animate top/left/margin */
.animate-bad {
  top: 10px; /* Triggers layout */
  transition: top 0.3s;
}
```

**E. Optimize Font Loading:**

```css
/* Use font-display to prevent FOIT/FOUT */
@font-face {
  font-family: 'MyFont';
  src: url('/fonts/myfont.woff2') format('woff2');
  font-display: optional; /* Best for CLS */
}

/* Or preload fonts */
```

```html
<link rel="preload" as="font" href="/fonts/myfont.woff2" type="font/woff2" crossorigin />
```

---

#### **6. Real-World Optimization Example**

```typescript
// src/components/OptimizedHero.tsx
import { useState, useEffect } from 'react';

function OptimizedHero() {
  const [imageLoaded, setImageLoaded] = useState(false);

  return (
    <section className="hero" style={{ minHeight: '500px' }}>
      {/* Reserve space to prevent CLS */}
      <div className="hero-image-container" style={{ aspectRatio: '16/9' }}>
        <picture>
          <source srcSet="/hero.avif" type="image/avif" />
          <source srcSet="/hero.webp" type="image/webp" />
          <img
            src="/hero.jpg"
            alt="Hero"
            width={1920}
            height={1080}
            loading="eager" // LCP element should load eagerly
            decoding="async"
            onLoad={() => setImageLoaded(true)}
            style={{
              width: '100%',
              height: '100%',
              objectFit: 'cover',
            }}
          />
        </picture>
      </div>

      <div className="hero-content">
        <h1>Welcome</h1>
        <p>This loads immediately (no CLS)</p>
      </div>
    </section>
  );
}

export default OptimizedHero;
```

---

#### **7. Monitoring and Alerting**

```typescript
// src/utils/vitalsMonitoring.ts
import { onLCP, onFID, onCLS, onINP } from 'web-vitals';

const THRESHOLDS = {
  LCP: 2500,
  FID: 100,
  INP: 200,
  CLS: 0.1,
};

function monitorVitals() {
  const checkThreshold = (metric: any) => {
    const threshold = THRESHOLDS[metric.name as keyof typeof THRESHOLDS];
    
    if (metric.value > threshold) {
      // Alert!
      console.warn(`${metric.name} exceeded threshold:`, {
        value: metric.value,
        threshold,
        rating: metric.rating,
      });

      // Send alert to monitoring service
      fetch('/api/alerts', {
        method: 'POST',
        body: JSON.stringify({
          metric: metric.name,
          value: metric.value,
          threshold,
          url: window.location.href,
        }),
      });
    }
  };

  onLCP(checkThreshold);
  onFID(checkThreshold);
  onCLS(checkThreshold);
  onINP(checkThreshold);
}

export { monitorVitals };
```

---

#### **8. Performance Budget**

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    visualizer({
      filename: './dist/stats.html',
      gzipSize: true,
    }),
  ],
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          router: ['react-router-dom'],
        },
      },
    },
    // Set size warnings
    chunkSizeWarningLimit: 500, // KB
  },
});
```

---

#### **Best Practices**

**For LCP:**
1. Optimize images (WebP, AVIF)
2. Use CDN for static assets
3. Implement code splitting
4. Enable compression (Brotli/Gzip)
5. Use SSR for critical content
6. Preload critical resources
7. Minimize render-blocking resources

**For FID/INP:**
1. Reduce JavaScript execution time
2. Split code into smaller chunks
3. Defer non-critical JavaScript
4. Use Web Workers for heavy tasks
5. Optimize event handlers
6. Minimize main thread work

**For CLS:**
1. Set image/video dimensions
2. Reserve space for ads/embeds
3. Avoid inserting content above fold
4. Use CSS transform for animations
5. Preload fonts with font-display
6. Use skeleton loaders

---

#### **Summary**

**Core Web Vitals:**

1. **LCP**: Largest Contentful Paint (< 2.5s)
   - Optimize images
   - Code splitting
   - SSR
   - Preload critical resources

2. **FID/INP**: First Input Delay / Interaction to Next Paint (< 100ms/200ms)
   - Reduce JavaScript
   - Use Web Workers
   - Optimize event handlers
   - Break up long tasks

3. **CLS**: Cumulative Layout Shift (< 0.1)
   - Set dimensions
   - Reserve space
   - Avoid content shifts
   - Optimize fonts

**Measurement:**
- web-vitals library
- Chrome DevTools
- Lighthouse
- PageSpeed Insights
- Real User Monitoring

**Tools:**
- **Lighthouse**: Synthetic testing
- **PageSpeed Insights**: Real + lab data
- **Chrome UX Report**: Real user data
- **web-vitals**: Measurement library
- **WebPageTest**: Detailed analysis

Optimizing Core Web Vitals improves **user experience**, **SEO rankings**, and **conversion rates**.

</details>

---

### 246. How do you track user behavior in React?

<details>
<summary>View Answer</summary>

**User behavior tracking** captures how users interact with your application, providing insights into usage patterns, pain points, and opportunities for improvement.

---

#### **1. Basic Event Tracking**

**Click Tracking:**

```typescript
// src/utils/tracking.ts

interface TrackingEvent {
  event: string;
  properties?: Record<string, any>;
  timestamp: number;
}

class BehaviorTracker {
  private events: TrackingEvent[] = [];

  track(event: string, properties?: Record<string, any>) {
    const trackingEvent: TrackingEvent = {
      event,
      properties,
      timestamp: Date.now(),
    };

    this.events.push(trackingEvent);
    console.log('Tracked:', trackingEvent);

    // Send to analytics
    this.send(trackingEvent);
  }

  private send(event: TrackingEvent) {
    fetch('/api/track', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(event),
    });
  }
}

export const tracker = new BehaviorTracker();
```

**Hook for Tracking:**

```typescript
// src/hooks/useTracking.ts
import { useCallback } from 'react';
import { tracker } from '../utils/tracking';

export function useTracking() {
  const trackClick = useCallback((element: string, properties?: any) => {
    tracker.track('click', { element, ...properties });
  }, []);

  const trackPageView = useCallback((page: string) => {
    tracker.track('page_view', { page });
  }, []);

  const trackEvent = useCallback((event: string, properties?: any) => {
    tracker.track(event, properties);
  }, []);

  return { trackClick, trackPageView, trackEvent };
}
```

**Usage:**

```typescript
function Button() {
  const { trackClick } = useTracking();

  const handleClick = () => {
    trackClick('submit_button', {
      location: 'checkout',
      value: 99.99,
    });
    submitForm();
  };

  return <button onClick={handleClick}>Submit</button>;
}
```

---

#### **2. Automatic Click Tracking**

```typescript
// src/utils/autoTracking.ts

export function enableAutoTracking() {
  document.addEventListener('click', (e) => {
    const target = e.target as HTMLElement;
    
    // Get meaningful element info
    const elementInfo = {
      tag: target.tagName.toLowerCase(),
      id: target.id,
      class: target.className,
      text: target.textContent?.slice(0, 50),
      href: target instanceof HTMLAnchorElement ? target.href : undefined,
    };

    // Track click
    tracker.track('auto_click', {
      element: elementInfo,
      x: e.clientX,
      y: e.clientY,
      page: window.location.pathname,
    });
  });
}
```

---

#### **3. Form Interaction Tracking**

```typescript
// src/hooks/useFormTracking.ts
import { useEffect, useRef } from 'react';
import { tracker } from '../utils/tracking';

export function useFormTracking(formName: string) {
  const startTime = useRef<number>(Date.now());
  const fieldInteractions = useRef<Record<string, number>>({});

  const trackFieldFocus = (fieldName: string) => {
    tracker.track('form_field_focus', {
      form: formName,
      field: fieldName,
    });
  };

  const trackFieldBlur = (fieldName: string, value: any) => {
    fieldInteractions.current[fieldName] =
      (fieldInteractions.current[fieldName] || 0) + 1;

    tracker.track('form_field_blur', {
      form: formName,
      field: fieldName,
      hasValue: !!value,
      interactions: fieldInteractions.current[fieldName],
    });
  };

  const trackSubmit = (success: boolean) => {
    const duration = Date.now() - startTime.current;

    tracker.track('form_submit', {
      form: formName,
      success,
      duration,
      fieldInteractions: Object.keys(fieldInteractions.current).length,
    });
  };

  const trackAbandon = () => {
    const duration = Date.now() - startTime.current;

    tracker.track('form_abandon', {
      form: formName,
      duration,
      fieldsInteracted: Object.keys(fieldInteractions.current),
    });
  };

  useEffect(() => {
    tracker.track('form_view', { form: formName });

    return () => {
      // Track abandon if form wasn't submitted
      if (Object.keys(fieldInteractions.current).length > 0) {
        trackAbandon();
      }
    };
  }, [formName]);

  return { trackFieldFocus, trackFieldBlur, trackSubmit };
}
```

**Usage:**

```typescript
function ContactForm() {
  const { trackFieldFocus, trackFieldBlur, trackSubmit } = useFormTracking('contact');

  const handleSubmit = async (e: FormEvent) => {
    e.preventDefault();
    
    try {
      await submitForm();
      trackSubmit(true);
    } catch (error) {
      trackSubmit(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="email"
        onFocus={() => trackFieldFocus('email')}
        onBlur={(e) => trackFieldBlur('email', e.target.value)}
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

#### **4. Scroll Depth Tracking**

```typescript
// src/hooks/useScrollTracking.ts
import { useEffect, useRef } from 'react';
import { tracker } from '../utils/tracking';

export function useScrollTracking(pageName: string) {
  const maxScroll = useRef(0);
  const milestones = useRef(new Set<number>());

  useEffect(() => {
    const handleScroll = () => {
      const windowHeight = window.innerHeight;
      const documentHeight = document.documentElement.scrollHeight;
      const scrollTop = window.scrollY;

      const scrollPercent = Math.round(
        (scrollTop / (documentHeight - windowHeight)) * 100
      );

      // Update max scroll
      if (scrollPercent > maxScroll.current) {
        maxScroll.current = scrollPercent;
      }

      // Track milestones (25%, 50%, 75%, 100%)
      [25, 50, 75, 100].forEach((milestone) => {
        if (scrollPercent >= milestone && !milestones.current.has(milestone)) {
          milestones.current.add(milestone);
          tracker.track('scroll_depth', {
            page: pageName,
            depth: milestone,
          });
        }
      });
    };

    window.addEventListener('scroll', handleScroll, { passive: true });

    return () => {
      window.removeEventListener('scroll', handleScroll);
      
      // Track final scroll depth
      tracker.track('scroll_complete', {
        page: pageName,
        maxDepth: maxScroll.current,
      });
    };
  }, [pageName]);
}
```

---

#### **5. Time on Page Tracking**

```typescript
// src/hooks/useTimeTracking.ts
import { useEffect, useRef } from 'react';
import { tracker } from '../utils/tracking';

export function useTimeTracking(pageName: string) {
  const startTime = useRef(Date.now());
  const activeTime = useRef(0);
  const lastActiveTime = useRef(Date.now());
  const isActive = useRef(true);

  useEffect(() => {
    // Track visibility changes
    const handleVisibilityChange = () => {
      if (document.hidden) {
        isActive.current = false;
        const sessionTime = Date.now() - lastActiveTime.current;
        activeTime.current += sessionTime;
      } else {
        isActive.current = true;
        lastActiveTime.current = Date.now();
      }
    };

    // Track user activity
    const handleActivity = () => {
      if (!isActive.current) {
        isActive.current = true;
        lastActiveTime.current = Date.now();
      }
    };

    document.addEventListener('visibilitychange', handleVisibilityChange);
    document.addEventListener('mousemove', handleActivity);
    document.addEventListener('keydown', handleActivity);
    document.addEventListener('scroll', handleActivity);

    return () => {
      document.removeEventListener('visibilitychange', handleVisibilityChange);
      document.removeEventListener('mousemove', handleActivity);
      document.removeEventListener('keydown', handleActivity);
      document.removeEventListener('scroll', handleActivity);

      // Track total time
      const totalTime = Date.now() - startTime.current;
      const finalActiveTime = isActive.current
        ? activeTime.current + (Date.now() - lastActiveTime.current)
        : activeTime.current;

      tracker.track('time_on_page', {
        page: pageName,
        totalTime,
        activeTime: finalActiveTime,
        idleTime: totalTime - finalActiveTime,
      });
    };
  }, [pageName]);
}
```

---

#### **6. User Flow Tracking**

```typescript
// src/utils/userFlow.ts

interface FlowStep {
  page: string;
  timestamp: number;
  duration: number;
}

class UserFlowTracker {
  private flow: FlowStep[] = [];
  private currentStep: { page: string; startTime: number } | null = null;

  startStep(page: string) {
    // Complete previous step
    if (this.currentStep) {
      const duration = Date.now() - this.currentStep.startTime;
      this.flow.push({
        page: this.currentStep.page,
        timestamp: this.currentStep.startTime,
        duration,
      });
    }

    // Start new step
    this.currentStep = {
      page,
      startTime: Date.now(),
    };
  }

  getFlow(): FlowStep[] {
    return [...this.flow];
  }

  sendFlow() {
    if (this.flow.length === 0) return;

    tracker.track('user_flow', {
      flow: this.flow,
      steps: this.flow.length,
      totalDuration: this.flow.reduce((sum, step) => sum + step.duration, 0),
    });
  }
}

export const flowTracker = new UserFlowTracker();
```

**Usage with React Router:**

```typescript
import { useEffect } from 'react';
import { useLocation } from 'react-router-dom';
import { flowTracker } from '../utils/userFlow';

function App() {
  const location = useLocation();

  useEffect(() => {
    flowTracker.startStep(location.pathname);
  }, [location]);

  useEffect(() => {
    // Send flow on app unmount
    return () => flowTracker.sendFlow();
  }, []);

  return <Routes>{/* ... */}</Routes>;
}
```

---

#### **7. Session Recording Integration**

**Using LogRocket:**

```bash
npm install logrocket
```

```typescript
// src/utils/logrocket.ts
import LogRocket from 'logrocket';

export function initLogRocket() {
  LogRocket.init('your-app-id', {
    // Configuration
    console: {
      shouldAggregateConsoleErrors: true,
    },
    network: {
      requestSanitizer: (request) => {
        // Remove sensitive headers
        if (request.headers['Authorization']) {
          request.headers['Authorization'] = '[REDACTED]';
        }
        return request;
      },
    },
  });
}

export function identifyUser(user: { id: string; email: string; name: string }) {
  LogRocket.identify(user.id, {
    email: user.email,
    name: user.name,
  });
}

export function trackAction(name: string, properties?: Record<string, any>) {
  LogRocket.track(name, properties);
}
```

---

#### **8. Heatmap Tracking**

**Using Hotjar:**

```typescript
// src/utils/hotjar.ts

export function initHotjar() {
  const hjid = import.meta.env.VITE_HOTJAR_ID;
  const hjsv = import.meta.env.VITE_HOTJAR_VERSION;

  (function(h: any, o: any, t: any, j: any, a?: any, r?: any) {
    h.hj = h.hj || function() { (h.hj.q = h.hj.q || []).push(arguments) };
    h._hjSettings = { hjid, hjsv };
    a = o.getElementsByTagName('head')[0];
    r = o.createElement('script');
    r.async = 1;
    r.src = t + h._hjSettings.hjid + j + h._hjSettings.hjsv;
    a.appendChild(r);
  })(window, document, 'https://static.hotjar.com/c/hotjar-', '.js?sv=');
}

export function triggerHotjarEvent(eventName: string) {
  if (window.hj) {
    window.hj('event', eventName);
  }
}
```

---

#### **9. Real-World Behavior Analytics Service**

```typescript
// src/services/behaviorAnalytics.ts
import { tracker } from '../utils/tracking';
import { flowTracker } from '../utils/userFlow';

class BehaviorAnalyticsService {
  private sessionId: string;
  private userId?: string;

  constructor() {
    this.sessionId = this.generateSessionId();
    this.init();
  }

  private generateSessionId(): string {
    return `session-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }

  private init() {
    // Track page visibility
    this.trackVisibility();
    
    // Track rage clicks
    this.trackRageClicks();
    
    // Track dead clicks
    this.trackDeadClicks();
    
    // Track errors
    this.trackErrors();
  }

  setUser(userId: string, properties?: Record<string, any>) {
    this.userId = userId;
    tracker.track('user_identified', {
      userId,
      ...properties,
    });
  }

  trackPageView(page: string, properties?: Record<string, any>) {
    tracker.track('page_view', {
      page,
      sessionId: this.sessionId,
      userId: this.userId,
      referrer: document.referrer,
      ...properties,
    });
  }

  trackEvent(event: string, properties?: Record<string, any>) {
    tracker.track(event, {
      sessionId: this.sessionId,
      userId: this.userId,
      ...properties,
    });
  }

  private trackVisibility() {
    document.addEventListener('visibilitychange', () => {
      tracker.track(document.hidden ? 'page_hidden' : 'page_visible', {
        sessionId: this.sessionId,
      });
    });
  }

  private trackRageClicks() {
    let clickCount = 0;
    let lastClickTime = 0;
    let lastTarget: EventTarget | null = null;

    document.addEventListener('click', (e) => {
      const now = Date.now();
      
      if (e.target === lastTarget && now - lastClickTime < 1000) {
        clickCount++;
        
        if (clickCount >= 3) {
          tracker.track('rage_click', {
            element: (e.target as HTMLElement).tagName,
            clicks: clickCount,
          });
        }
      } else {
        clickCount = 1;
      }
      
      lastClickTime = now;
      lastTarget = e.target;
    });
  }

  private trackDeadClicks() {
    document.addEventListener('click', (e) => {
      const target = e.target as HTMLElement;
      
      // Check if element is interactive
      const isInteractive = 
        target.tagName === 'BUTTON' ||
        target.tagName === 'A' ||
        target.onclick !== null ||
        target.getAttribute('role') === 'button';
      
      if (!isInteractive) {
        tracker.track('dead_click', {
          element: target.tagName,
          text: target.textContent?.slice(0, 50),
        });
      }
    });
  }

  private trackErrors() {
    window.addEventListener('error', (e) => {
      tracker.track('javascript_error', {
        message: e.message,
        filename: e.filename,
        line: e.lineno,
        column: e.colno,
      });
    });

    window.addEventListener('unhandledrejection', (e) => {
      tracker.track('unhandled_rejection', {
        reason: e.reason,
      });
    });
  }
}

export const behaviorAnalytics = new BehaviorAnalyticsService();
```

---

#### **Best Practices**

1. **Privacy First**: Respect user privacy and GDPR/CCPA
2. **Meaningful Events**: Track actionable events
3. **Performance**: Don't impact app performance
4. **User Consent**: Get consent before tracking
5. **Data Quality**: Validate before sending
6. **Sampling**: Don't track 100% of users
7. **Security**: Don't track sensitive data
8. **Documentation**: Document tracked events

---

#### **Summary**

**User Behavior Tracking:**

1. **Event Tracking**: Clicks, form interactions, custom events
2. **Scroll Depth**: How far users scroll
3. **Time on Page**: Active vs idle time
4. **User Flows**: Navigation patterns
5. **Form Analytics**: Field interactions, abandonment
6. **Session Recording**: Video replays
7. **Heatmaps**: Click/scroll/movement maps
8. **Error Tracking**: JavaScript errors, rage clicks

**Key Metrics:**
- Click-through rates
- Form completion rates
- Time on page
- Scroll depth
- User flows
- Bounce rate
- Engagement score

**Popular Tools:**
- **Google Analytics**: General analytics
- **Mixpanel**: Event-based tracking
- **Hotjar**: Heatmaps + recordings
- **LogRocket**: Session replay + analytics
- **FullStory**: Session replay
- **Heap**: Auto-capture events

Proper behavior tracking provides **actionable insights** into how users interact with your app and where improvements are needed.

</details>

---

### 247. How do you implement A/B testing in React?

<details>
<summary>View Answer</summary>

**A/B testing** (split testing) allows you to compare different versions of your UI to determine which performs better. Implementation typically involves **feature flags**, **variant assignment**, and **analytics integration**.

---

#### **1. Basic A/B Test Hook**

```typescript
// src/hooks/useABTest.ts
import { useState, useEffect } from 'react';

type Variant = 'A' | 'B';

interface ABTestConfig {
  testName: string;
  variants: Variant[];
  weights?: number[]; // Optional weights, defaults to equal split
}

export function useABTest(config: ABTestConfig): Variant {
  const { testName, variants, weights = [50, 50] } = config;
  const [variant, setVariant] = useState<Variant>('A');

  useEffect(() => {
    // Check if user already has a variant assigned
    const storageKey = `ab_test_${testName}`;
    const storedVariant = localStorage.getItem(storageKey) as Variant;

    if (storedVariant && variants.includes(storedVariant)) {
      setVariant(storedVariant);
    } else {
      // Assign variant based on weights
      const assignedVariant = assignVariant(variants, weights);
      setVariant(assignedVariant);
      localStorage.setItem(storageKey, assignedVariant);

      // Track assignment
      trackEvent('ab_test_assigned', {
        test: testName,
        variant: assignedVariant,
      });
    }
  }, [testName, variants, weights]);

  return variant;
}

function assignVariant(variants: Variant[], weights: number[]): Variant {
  const random = Math.random() * 100;
  let cumulative = 0;

  for (let i = 0; i < variants.length; i++) {
    cumulative += weights[i];
    if (random < cumulative) {
      return variants[i];
    }
  }

  return variants[0];
}

function trackEvent(event: string, properties: any) {
  // Send to analytics
  console.log('Track:', event, properties);
}
```

**Usage:**

```typescript
function CheckoutButton() {
  const variant = useABTest({
    testName: 'checkout_button_color',
    variants: ['A', 'B'],
    weights: [50, 50],
  });

  const buttonColor = variant === 'A' ? 'blue' : 'green';
  const buttonText = variant === 'A' ? 'Checkout' : 'Buy Now';

  const handleClick = () => {
    trackEvent('checkout_clicked', {
      test: 'checkout_button_color',
      variant,
    });
    proceedToCheckout();
  };

  return (
    <button
      style={{ backgroundColor: buttonColor }}
      onClick={handleClick}
    >
      {buttonText}
    </button>
  );
}
```

---

#### **2. Advanced A/B Test Hook with Analytics**

```typescript
// src/hooks/useExperiment.ts
import { useState, useEffect, useCallback } from 'react';
import { analytics } from '../services/analytics';

interface Experiment<T extends string> {
  id: string;
  variants: T[];
  weights?: number[];
}

interface ExperimentResult<T> {
  variant: T;
  trackConversion: (value?: number) => void;
  trackEvent: (eventName: string, properties?: any) => void;
}

export function useExperiment<T extends string>(
  experiment: Experiment<T>
): ExperimentResult<T> {
  const { id, variants, weights } = experiment;
  const [variant, setVariant] = useState<T>(variants[0]);

  useEffect(() => {
    const storageKey = `experiment_${id}`;
    const stored = localStorage.getItem(storageKey) as T;

    if (stored && variants.includes(stored)) {
      setVariant(stored);
    } else {
      const assigned = assignVariant(variants, weights);
      setVariant(assigned);
      localStorage.setItem(storageKey, assigned);

      // Track exposure
      analytics.trackEvent('experiment_exposure', {
        experimentId: id,
        variant: assigned,
      });
    }
  }, [id, variants, weights]);

  const trackConversion = useCallback((value?: number) => {
    analytics.trackEvent('experiment_conversion', {
      experimentId: id,
      variant,
      value,
    });
  }, [id, variant]);

  const trackEvent = useCallback((eventName: string, properties?: any) => {
    analytics.trackEvent(eventName, {
      experimentId: id,
      variant,
      ...properties,
    });
  }, [id, variant]);

  return { variant, trackConversion, trackEvent };
}

function assignVariant<T>(variants: T[], weights?: number[]): T {
  if (!weights || weights.length !== variants.length) {
    // Equal distribution
    const index = Math.floor(Math.random() * variants.length);
    return variants[index];
  }

  const random = Math.random() * 100;
  let cumulative = 0;

  for (let i = 0; i < variants.length; i++) {
    cumulative += weights[i];
    if (random < cumulative) {
      return variants[i];
    }
  }

  return variants[0];
}
```

**Usage:**

```typescript
function PricingPage() {
  const { variant, trackConversion, trackEvent } = useExperiment({
    id: 'pricing_layout',
    variants: ['grid', 'list', 'cards'],
    weights: [33, 33, 34],
  });

  const handleSubscribe = (plan: string, price: number) => {
    trackConversion(price);
    subscribe(plan);
  };

  const handleViewDetails = (plan: string) => {
    trackEvent('view_plan_details', { plan });
  };

  return (
    <div className={`pricing-${variant}`}>
      {/* Render different layouts based on variant */}
    </div>
  );
}
```

---

#### **3. Feature Flags with LaunchDarkly**

**Installation:**

```bash
npm install launchdarkly-react-client-sdk
```

**Setup:**

```typescript
// src/App.tsx
import { withLDProvider } from 'launchdarkly-react-client-sdk';

function App() {
  return (
    <div>
      <Header />
      <Routes />
    </div>
  );
}

export default withLDProvider({
  clientSideID: import.meta.env.VITE_LAUNCHDARKLY_CLIENT_ID,
  context: {
    kind: 'user',
    key: 'user-123',
    name: 'John Doe',
    email: 'john@example.com',
  },
})(App);
```

**Usage:**

```typescript
import { useFlags } from 'launchdarkly-react-client-sdk';

function FeatureComponent() {
  const flags = useFlags();

  if (flags.newCheckoutFlow) {
    return <NewCheckout />;
  }

  return <OldCheckout />;
}
```

**With Variations:**

```typescript
import { useLDClient } from 'launchdarkly-react-client-sdk';

function PricingPage() {
  const ldClient = useLDClient();
  const layout = ldClient?.variation('pricing-layout', 'grid');

  return <div className={`pricing-${layout}`}>{/* ... */}</div>;
}
```

---

#### **4. A/B Testing with Optimizely**

**Installation:**

```bash
npm install @optimizely/react-sdk
```

**Setup:**

```typescript
// src/App.tsx
import {
  OptimizelyProvider,
  createInstance,
} from '@optimizely/react-sdk';

const optimizelyClient = createInstance({
  sdkKey: import.meta.env.VITE_OPTIMIZELY_SDK_KEY,
});

function App() {
  return (
    <OptimizelyProvider
      optimizely={optimizelyClient}
      user={{ id: 'user-123' }}
    >
      <AppContent />
    </OptimizelyProvider>
  );
}
```

**Usage:**

```typescript
import { useDecision } from '@optimizely/react-sdk';

function ProductPage() {
  const [decision] = useDecision('product_page_layout');

  const layout = decision.variationKey || 'control';

  useEffect(() => {
    if (decision.enabled) {
      console.log('Experiment enabled:', layout);
    }
  }, [decision, layout]);

  return (
    <div>
      {layout === 'variant_a' && <LayoutA />}
      {layout === 'variant_b' && <LayoutB />}
      {layout === 'control' && <LayoutControl />}
    </div>
  );
}
```

---

#### **5. Multivariate Testing**

```typescript
// src/hooks/useMultivariateTest.ts
import { useState, useEffect } from 'react';

interface Factor {
  name: string;
  variants: string[];
}

interface MVTConfig {
  testName: string;
  factors: Factor[];
}

type VariantCombination = Record<string, string>;

export function useMultivariateTest(config: MVTConfig): VariantCombination {
  const { testName, factors } = config;
  const [combination, setCombination] = useState<VariantCombination>({});

  useEffect(() => {
    const storageKey = `mvt_${testName}`;
    const stored = localStorage.getItem(storageKey);

    if (stored) {
      setCombination(JSON.parse(stored));
    } else {
      // Assign random variant for each factor
      const assigned: VariantCombination = {};
      
      factors.forEach((factor) => {
        const randomIndex = Math.floor(Math.random() * factor.variants.length);
        assigned[factor.name] = factor.variants[randomIndex];
      });

      setCombination(assigned);
      localStorage.setItem(storageKey, JSON.stringify(assigned));

      // Track assignment
      trackEvent('mvt_assigned', {
        test: testName,
        combination: assigned,
      });
    }
  }, [testName, factors]);

  return combination;
}
```

**Usage:**

```typescript
function ProductCard() {
  const combination = useMultivariateTest({
    testName: 'product_card_optimization',
    factors: [
      { name: 'buttonColor', variants: ['blue', 'green', 'red'] },
      { name: 'buttonText', variants: ['Add to Cart', 'Buy Now', 'Purchase'] },
      { name: 'imageSize', variants: ['small', 'medium', 'large'] },
    ],
  });

  return (
    <div className="product-card">
      <img
        src="product.jpg"
        className={`image-${combination.imageSize}`}
      />
      <button style={{ backgroundColor: combination.buttonColor }}>
        {combination.buttonText}
      </button>
    </div>
  );
}
```

---

#### **6. Statistical Significance Tracking**

```typescript
// src/utils/statistics.ts

interface ExperimentData {
  variantA: {
    visitors: number;
    conversions: number;
  };
  variantB: {
    visitors: number;
    conversions: number;
  };
}

export function calculateSignificance(data: ExperimentData): {
  significant: boolean;
  pValue: number;
  confidence: number;
} {
  const { variantA, variantB } = data;

  // Calculate conversion rates
  const rateA = variantA.conversions / variantA.visitors;
  const rateB = variantB.conversions / variantB.visitors;

  // Calculate pooled probability
  const totalConversions = variantA.conversions + variantB.conversions;
  const totalVisitors = variantA.visitors + variantB.visitors;
  const pooledProbability = totalConversions / totalVisitors;

  // Calculate standard error
  const se = Math.sqrt(
    pooledProbability * (1 - pooledProbability) *
    (1 / variantA.visitors + 1 / variantB.visitors)
  );

  // Calculate z-score
  const zScore = (rateB - rateA) / se;

  // Calculate p-value (two-tailed test)
  const pValue = 2 * (1 - normalCDF(Math.abs(zScore)));

  return {
    significant: pValue < 0.05,
    pValue,
    confidence: (1 - pValue) * 100,
  };
}

function normalCDF(x: number): number {
  // Approximation of cumulative distribution function
  const t = 1 / (1 + 0.2316419 * Math.abs(x));
  const d = 0.3989423 * Math.exp(-x * x / 2);
  const p =
    d *
    t *
    (0.3193815 +
      t * (-0.3565638 + t * (1.781478 + t * (-1.821256 + t * 1.330274))));
  return x > 0 ? 1 - p : p;
}
```

---

#### **7. Real-World Example: E-commerce A/B Test**

```typescript
// src/components/CheckoutFlow.tsx
import { useExperiment } from '../hooks/useExperiment';

function CheckoutFlow() {
  const { variant, trackConversion, trackEvent } = useExperiment({
    id: 'checkout_flow_redesign',
    variants: ['single_page', 'multi_step'],
  });

  const [currentStep, setCurrentStep] = useState(0);

  useEffect(() => {
    trackEvent('checkout_started');
  }, []);

  const handleStepComplete = (step: number) => {
    trackEvent('checkout_step_completed', { step });
    setCurrentStep(step + 1);
  };

  const handlePurchase = async (orderData: Order) => {
    try {
      const order = await processOrder(orderData);
      trackConversion(order.total);
      trackEvent('purchase_completed', {
        orderId: order.id,
        total: order.total,
      });
    } catch (error) {
      trackEvent('purchase_failed', {
        error: error.message,
      });
    }
  };

  if (variant === 'single_page') {
    return (
      <SinglePageCheckout
        onPurchase={handlePurchase}
        onFieldInteraction={(field) => trackEvent('field_interaction', { field })}
      />
    );
  }

  return (
    <MultiStepCheckout
      currentStep={currentStep}
      onStepComplete={handleStepComplete}
      onPurchase={handlePurchase}
    />
  );
}
```

---

#### **8. A/B Test Dashboard Component**

```typescript
// src/components/ABTestDashboard.tsx
import { useState, useEffect } from 'react';
import { calculateSignificance } from '../utils/statistics';

interface TestResult {
  testId: string;
  variantA: { visitors: number; conversions: number };
  variantB: { visitors: number; conversions: number };
}

function ABTestDashboard() {
  const [tests, setTests] = useState<TestResult[]>([]);

  useEffect(() => {
    fetchTestResults().then(setTests);
  }, []);

  return (
    <div className="ab-test-dashboard">
      <h2>Active A/B Tests</h2>
      {tests.map((test) => {
        const stats = calculateSignificance(test);
        const rateA = (test.variantA.conversions / test.variantA.visitors) * 100;
        const rateB = (test.variantB.conversions / test.variantB.visitors) * 100;
        const improvement = ((rateB - rateA) / rateA) * 100;

        return (
          <div key={test.testId} className="test-card">
            <h3>{test.testId}</h3>
            
            <div className="variants">
              <div className="variant">
                <h4>Variant A (Control)</h4>
                <p>Visitors: {test.variantA.visitors}</p>
                <p>Conversions: {test.variantA.conversions}</p>
                <p>Rate: {rateA.toFixed(2)}%</p>
              </div>
              
              <div className="variant">
                <h4>Variant B</h4>
                <p>Visitors: {test.variantB.visitors}</p>
                <p>Conversions: {test.variantB.conversions}</p>
                <p>Rate: {rateB.toFixed(2)}%</p>
              </div>
            </div>

            <div className="results">
              <p>Improvement: {improvement.toFixed(2)}%</p>
              <p>Confidence: {stats.confidence.toFixed(2)}%</p>
              <p className={stats.significant ? 'significant' : 'not-significant'}>
                {stats.significant ? '✓ Statistically Significant' : '✗ Not Significant'}
              </p>
            </div>
          </div>
        );
      })}
    </div>
  );
}
```

---

#### **Best Practices**

1. **Statistical Significance**: Run tests until you have enough data
2. **Sample Size**: Calculate required sample size before starting
3. **One Variable**: Test one thing at a time (unless MVT)
4. **Random Assignment**: Ensure unbiased variant assignment
5. **Consistent Experience**: Same user always sees same variant
6. **Track Everything**: Exposure, conversions, and interactions
7. **Avoid Bias**: Don't peek at results too early
8. **Document Tests**: Keep track of what you're testing and why

---

#### **Summary**

**A/B Testing Implementation:**

1. **Custom Hooks**: useABTest, useExperiment for simple tests
2. **Feature Flags**: LaunchDarkly, Optimizely for enterprise
3. **Variant Assignment**: Random with consistent user experience
4. **Analytics Integration**: Track exposures and conversions
5. **Multivariate Testing**: Test multiple factors simultaneously
6. **Statistical Analysis**: Calculate significance and confidence
7. **Real-time Monitoring**: Dashboard for active experiments

**Key Concepts:**
- **Control vs Variant**: Baseline vs new version
- **Conversion Rate**: % of users who complete goal action
- **Statistical Significance**: Confidence in results (p < 0.05)
- **Sample Size**: Enough users for reliable results
- **A/A Testing**: Sanity check with identical variants

**Popular Tools:**
- **LaunchDarkly**: Feature flags + experiments
- **Optimizely**: Full experimentation platform
- **Google Optimize**: Free, integrates with GA
- **Split.io**: Feature flags + A/B testing
- **VWO**: Visual editor + testing

Proper A/B testing enables **data-driven decisions** and **continuous optimization** of user experience.

</details>

---

### 248. What is OpenTelemetry and its use in React?

<details>
<summary>View Answer</summary>

**OpenTelemetry (OTel)** is an open-source observability framework for collecting **traces**, **metrics**, and **logs** from applications. It provides vendor-neutral instrumentation for monitoring distributed systems.

---

#### **1. OpenTelemetry Concepts**

**Core Components:**

- **Traces**: Track requests through distributed systems
- **Spans**: Individual units of work within a trace
- **Metrics**: Numerical measurements (counters, gauges, histograms)
- **Context Propagation**: Pass trace context between services
- **Exporters**: Send data to backends (Jaeger, Zipkin, Prometheus)

**Benefits:**
- Vendor-neutral (switch backends easily)
- Standardized instrumentation
- Distributed tracing across microservices
- Rich ecosystem and community support

---

#### **2. Basic Setup**

**Installation:**

```bash
npm install @opentelemetry/api \
  @opentelemetry/sdk-trace-web \
  @opentelemetry/sdk-trace-base \
  @opentelemetry/instrumentation-fetch \
  @opentelemetry/instrumentation-xml-http-request \
  @opentelemetry/exporter-trace-otlp-http
```

**Configuration:**

```typescript
// src/utils/telemetry.ts
import { WebTracerProvider } from '@opentelemetry/sdk-trace-web';
import { SimpleSpanProcessor } from '@opentelemetry/sdk-trace-base';
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http';
import { FetchInstrumentation } from '@opentelemetry/instrumentation-fetch';
import { XMLHttpRequestInstrumentation } from '@opentelemetry/instrumentation-xml-http-request';
import { registerInstrumentations } from '@opentelemetry/instrumentation';
import { Resource } from '@opentelemetry/resources';
import { SemanticResourceAttributes } from '@opentelemetry/semantic-conventions';

export function initTelemetry() {
  // Create resource
  const resource = new Resource({
    [SemanticResourceAttributes.SERVICE_NAME]: 'my-react-app',
    [SemanticResourceAttributes.SERVICE_VERSION]: '1.0.0',
  });

  // Create provider
  const provider = new WebTracerProvider({ resource });

  // Configure exporter
  const exporter = new OTLPTraceExporter({
    url: 'http://localhost:4318/v1/traces',
  });

  // Add span processor
  provider.addSpanProcessor(new SimpleSpanProcessor(exporter));

  // Register provider
  provider.register();

  // Auto-instrument fetch and XHR
  registerInstrumentations({
    instrumentations: [
      new FetchInstrumentation({
        propagateTraceHeaderCorsUrls: [/.*/],
        clearTimingResources: true,
      }),
      new XMLHttpRequestInstrumentation({
        propagateTraceHeaderCorsUrls: [/.*/],
      }),
    ],
  });
}
```

**Initialize in App:**

```typescript
// src/main.tsx
import { initTelemetry } from './utils/telemetry';

// Initialize before rendering
initTelemetry();

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

---

#### **3. Manual Instrumentation**

**Creating Spans:**

```typescript
// src/utils/tracer.ts
import { trace, Span, SpanStatusCode } from '@opentelemetry/api';

const tracer = trace.getTracer('my-react-app');

export function createSpan(name: string, fn: (span: Span) => Promise<any>) {
  return tracer.startActiveSpan(name, async (span) => {
    try {
      const result = await fn(span);
      span.setStatus({ code: SpanStatusCode.OK });
      return result;
    } catch (error) {
      span.setStatus({
        code: SpanStatusCode.ERROR,
        message: error instanceof Error ? error.message : 'Unknown error',
      });
      span.recordException(error as Error);
      throw error;
    } finally {
      span.end();
    }
  });
}
```

**Usage:**

```typescript
import { createSpan } from '../utils/tracer';

async function fetchUserData(userId: string) {
  return createSpan('fetchUserData', async (span) => {
    span.setAttribute('user.id', userId);
    
    const response = await fetch(`/api/users/${userId}`);
    const data = await response.json();
    
    span.setAttribute('user.name', data.name);
    span.addEvent('User data fetched');
    
    return data;
  });
}
```

---

#### **4. React Component Instrumentation**

**HOC for Tracing:**

```typescript
// src/hoc/withTracing.tsx
import { ComponentType, useEffect } from 'react';
import { trace, Span } from '@opentelemetry/api';

const tracer = trace.getTracer('react-components');

export function withTracing<P extends object>(
  Component: ComponentType<P>,
  componentName: string
) {
  return function TracedComponent(props: P) {
    useEffect(() => {
      const span = tracer.startSpan(`${componentName}.mount`);
      span.setAttribute('component.name', componentName);
      
      return () => {
        span.end();
      };
    }, []);

    return <Component {...props} />;
  };
}

// Usage
const TracedDashboard = withTracing(Dashboard, 'Dashboard');
```

**Hook for Component Tracing:**

```typescript
// src/hooks/useTracing.ts
import { useEffect, useRef } from 'react';
import { trace, Span } from '@opentelemetry/api';

const tracer = trace.getTracer('react-hooks');

export function useTracing(operationName: string) {
  const spanRef = useRef<Span | null>(null);

  useEffect(() => {
    spanRef.current = tracer.startSpan(operationName);
    
    return () => {
      spanRef.current?.end();
    };
  }, [operationName]);

  const addEvent = (name: string, attributes?: any) => {
    spanRef.current?.addEvent(name, attributes);
  };

  const setAttribute = (key: string, value: any) => {
    spanRef.current?.setAttribute(key, value);
  };

  return { addEvent, setAttribute };
}
```

**Usage:**

```typescript
function UserProfile({ userId }: { userId: string }) {
  const { addEvent, setAttribute } = useTracing('UserProfile.render');
  const [user, setUser] = useState(null);

  useEffect(() => {
    setAttribute('user.id', userId);
    addEvent('Fetching user data');
    
    fetchUser(userId).then((data) => {
      setUser(data);
      addEvent('User data loaded');
    });
  }, [userId]);

  return <div>{/* ... */}</div>;
}
```

---

#### **5. Metrics Collection**

**Setup Metrics:**

```bash
npm install @opentelemetry/sdk-metrics
```

```typescript
// src/utils/metrics.ts
import { MeterProvider, PeriodicExportingMetricReader } from '@opentelemetry/sdk-metrics';
import { OTLPMetricExporter } from '@opentelemetry/exporter-metrics-otlp-http';
import { Resource } from '@opentelemetry/resources';
import { metrics } from '@opentelemetry/api';

export function initMetrics() {
  const resource = new Resource({
    'service.name': 'my-react-app',
  });

  const exporter = new OTLPMetricExporter({
    url: 'http://localhost:4318/v1/metrics',
  });

  const meterProvider = new MeterProvider({
    resource,
    readers: [
      new PeriodicExportingMetricReader({
        exporter,
        exportIntervalMillis: 60000, // Export every minute
      }),
    ],
  });

  metrics.setGlobalMeterProvider(meterProvider);
}

// Create meters
const meter = metrics.getMeter('react-app-metrics');

// Counter for page views
export const pageViewCounter = meter.createCounter('page_views', {
  description: 'Count of page views',
});

// Histogram for API response times
export const apiResponseTime = meter.createHistogram('api_response_time', {
  description: 'API response time in milliseconds',
  unit: 'ms',
});

// Gauge for active users
export const activeUsersGauge = meter.createObservableGauge('active_users', {
  description: 'Number of active users',
});
```

**Usage:**

```typescript
import { pageViewCounter, apiResponseTime } from '../utils/metrics';
import { useLocation } from 'react-router-dom';

function App() {
  const location = useLocation();

  useEffect(() => {
    pageViewCounter.add(1, {
      page: location.pathname,
    });
  }, [location]);

  return <Routes>{/* ... */}</Routes>;
}

// Track API response time
async function fetchData() {
  const startTime = performance.now();
  
  try {
    const response = await fetch('/api/data');
    const data = await response.json();
    
    const duration = performance.now() - startTime;
    apiResponseTime.record(duration, {
      endpoint: '/api/data',
      status: response.status,
    });
    
    return data;
  } catch (error) {
    const duration = performance.now() - startTime;
    apiResponseTime.record(duration, {
      endpoint: '/api/data',
      status: 'error',
    });
    throw error;
  }
}
```

---

#### **6. Context Propagation**

**Propagate Trace Context to Backend:**

```typescript
import { context, propagation } from '@opentelemetry/api';

async function apiCall(url: string, options: RequestInit = {}) {
  // Get current context
  const ctx = context.active();
  
  // Inject trace context into headers
  const headers = new Headers(options.headers);
  propagation.inject(ctx, headers, {
    set: (carrier, key, value) => {
      carrier.set(key, value);
    },
  });

  return fetch(url, {
    ...options,
    headers,
  });
}
```

---

#### **7. Real-World Example: E-commerce Checkout**

```typescript
// src/services/checkoutService.ts
import { trace, SpanStatusCode } from '@opentelemetry/api';
import { pageViewCounter, apiResponseTime } from '../utils/metrics';

const tracer = trace.getTracer('checkout-service');

export async function processCheckout(orderData: OrderData) {
  return tracer.startActiveSpan('processCheckout', async (parentSpan) => {
    try {
      parentSpan.setAttribute('order.items', orderData.items.length);
      parentSpan.setAttribute('order.total', orderData.total);

      // Validate order
      const validationSpan = tracer.startSpan('validateOrder', {
        parent: parentSpan,
      });
      await validateOrder(orderData);
      validationSpan.end();

      // Process payment
      const paymentSpan = tracer.startSpan('processPayment', {
        parent: parentSpan,
      });
      const startTime = performance.now();
      
      try {
        const payment = await processPayment(orderData.payment);
        paymentSpan.setAttribute('payment.id', payment.id);
        paymentSpan.setStatus({ code: SpanStatusCode.OK });
        
        const duration = performance.now() - startTime;
        apiResponseTime.record(duration, {
          endpoint: 'payment',
          status: 'success',
        });
      } catch (error) {
        paymentSpan.setStatus({
          code: SpanStatusCode.ERROR,
          message: 'Payment failed',
        });
        paymentSpan.recordException(error as Error);
        throw error;
      } finally {
        paymentSpan.end();
      }

      // Create order
      const orderSpan = tracer.startSpan('createOrder', {
        parent: parentSpan,
      });
      const order = await createOrder(orderData);
      orderSpan.setAttribute('order.id', order.id);
      orderSpan.end();

      // Send confirmation
      const emailSpan = tracer.startSpan('sendConfirmation', {
        parent: parentSpan,
      });
      await sendConfirmationEmail(order);
      emailSpan.end();

      parentSpan.addEvent('Checkout completed');
      parentSpan.setStatus({ code: SpanStatusCode.OK });

      // Track conversion metric
      pageViewCounter.add(1, { event: 'checkout_completed' });

      return order;
    } catch (error) {
      parentSpan.setStatus({
        code: SpanStatusCode.ERROR,
        message: error instanceof Error ? error.message : 'Checkout failed',
      });
      parentSpan.recordException(error as Error);
      throw error;
    } finally {
      parentSpan.end();
    }
  });
}
```

---

#### **8. Integration with Backends**

**Jaeger (for tracing):**

```typescript
import { JaegerExporter } from '@opentelemetry/exporter-jaeger';

const exporter = new JaegerExporter({
  endpoint: 'http://localhost:14268/api/traces',
});
```

**Prometheus (for metrics):**

```typescript
import { PrometheusExporter } from '@opentelemetry/exporter-prometheus';

const exporter = new PrometheusExporter(
  {
    port: 9464,
    endpoint: '/metrics',
  },
  () => {
    console.log('Prometheus scrape endpoint: http://localhost:9464/metrics');
  }
);
```

**Zipkin:**

```typescript
import { ZipkinExporter } from '@opentelemetry/exporter-zipkin';

const exporter = new ZipkinExporter({
  url: 'http://localhost:9411/api/v2/spans',
  serviceName: 'my-react-app',
});
```

---

#### **Best Practices**

1. **Meaningful Span Names**: Use descriptive operation names
2. **Add Context**: Include relevant attributes and events
3. **Sampling**: Don't trace 100% in production
4. **Error Handling**: Record exceptions in spans
5. **Performance**: Keep span creation lightweight
6. **Security**: Don't include sensitive data in traces
7. **Propagation**: Pass context to backend services
8. **Metrics**: Combine traces with metrics for full picture

---

#### **Summary**

**OpenTelemetry in React:**

1. **Setup**: Initialize provider, exporters, and instrumentations
2. **Auto-instrumentation**: Fetch and XHR automatically traced
3. **Manual Spans**: Create spans for custom operations
4. **Component Tracing**: Track React component lifecycle
5. **Metrics**: Collect counters, histograms, gauges
6. **Context Propagation**: Pass trace context to backend
7. **Error Tracking**: Record exceptions in spans
8. **Backends**: Export to Jaeger, Zipkin, Prometheus

**Key Concepts:**
- **Trace**: End-to-end request flow
- **Span**: Unit of work within trace
- **Attributes**: Key-value metadata
- **Events**: Point-in-time occurrences
- **Context**: Propagation across boundaries

**Benefits:**
- Vendor-neutral observability
- Distributed tracing
- Performance monitoring
- Root cause analysis
- Service dependency mapping

**Use Cases:**
- Debug production issues
- Monitor API performance
- Track user journeys
- Identify bottlenecks
- Analyze microservice interactions

OpenTelemetry provides **comprehensive observability** for modern React applications, especially in **distributed architectures**.

</details>

---

### 249. How do you debug production issues?

<details>
<summary>View Answer</summary>

**Production debugging** requires different tools and techniques than local development. You need **error tracking**, **logging**, **remote debugging**, and **observability** to identify and fix issues.

---

#### **1. Error Tracking with Source Maps**

**Setup Sentry with Source Maps:**

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { sentryVitePlugin } from '@sentry/vite-plugin';

export default defineConfig({
  plugins: [
    react(),
    sentryVitePlugin({
      org: 'my-org',
      project: 'my-project',
      authToken: process.env.SENTRY_AUTH_TOKEN,
      sourcemaps: {
        assets: './dist/**',
        filesToDeleteAfterUpload: './dist/**/*.map',
      },
    }),
  ],
  build: {
    sourcemap: true, // Generate source maps
  },
});
```

**Sentry Configuration:**

```typescript
// src/utils/sentry.ts
import * as Sentry from '@sentry/react';

export function initSentry() {
  Sentry.init({
    dsn: import.meta.env.VITE_SENTRY_DSN,
    environment: import.meta.env.MODE,
    release: import.meta.env.VITE_APP_VERSION,
    
    // Enable performance monitoring
    integrations: [
      new Sentry.BrowserTracing({
        tracingOrigins: ['localhost', /^https:\/\/api\.example\.com/],
      }),
      new Sentry.Replay({
        maskAllText: false,
        blockAllMedia: false,
      }),
    ],

    // Performance monitoring
    tracesSampleRate: 0.1, // 10% of transactions
    
    // Session replay
    replaysSessionSampleRate: 0.1, // 10% of sessions
    replaysOnErrorSampleRate: 1.0, // 100% of sessions with errors

    // Filter errors
    beforeSend(event, hint) {
      // Don't send errors from browser extensions
      if (event.exception?.values?.[0]?.stacktrace?.frames?.some(
        frame => frame.filename?.includes('extension://') ||
                 frame.filename?.includes('chrome-extension://')
      )) {
        return null;
      }

      // Add custom context
      event.contexts = {
        ...event.contexts,
        device: {
          screen_resolution: `${window.screen.width}x${window.screen.height}`,
          viewport: `${window.innerWidth}x${window.innerHeight}`,
        },
      };

      return event;
    },
  });
}
```

---

#### **2. Enhanced Logging System**

```typescript
// src/utils/logger.ts
import * as Sentry from '@sentry/react';

enum LogLevel {
  DEBUG = 'debug',
  INFO = 'info',
  WARN = 'warn',
  ERROR = 'error',
}

interface LogContext {
  userId?: string;
  sessionId?: string;
  component?: string;
  action?: string;
  [key: string]: any;
}

class Logger {
  private context: LogContext = {};
  private enabled: boolean;

  constructor() {
    this.enabled = import.meta.env.MODE !== 'production' ||
                   localStorage.getItem('debug') === 'true';
  }

  setContext(context: LogContext) {
    this.context = { ...this.context, ...context };
    Sentry.setContext('logger', context);
  }

  private log(level: LogLevel, message: string, data?: any) {
    const logData = {
      level,
      message,
      timestamp: new Date().toISOString(),
      context: this.context,
      data,
    };

    // Console output
    if (this.enabled) {
      const consoleMethod = level === LogLevel.DEBUG ? 'debug' :
                           level === LogLevel.INFO ? 'info' :
                           level === LogLevel.WARN ? 'warn' : 'error';
      console[consoleMethod](`[${level.toUpperCase()}]`, message, data || '');
    }

    // Send to backend
    if (level === LogLevel.ERROR || level === LogLevel.WARN) {
      this.sendToBackend(logData);
    }

    // Add breadcrumb to Sentry
    Sentry.addBreadcrumb({
      category: 'log',
      message,
      level: level as any,
      data: { ...this.context, ...data },
    });
  }

  private sendToBackend(logData: any) {
    // Batch and send logs
    if ('sendBeacon' in navigator) {
      navigator.sendBeacon(
        '/api/logs',
        JSON.stringify(logData)
      );
    }
  }

  debug(message: string, data?: any) {
    this.log(LogLevel.DEBUG, message, data);
  }

  info(message: string, data?: any) {
    this.log(LogLevel.INFO, message, data);
  }

  warn(message: string, data?: any) {
    this.log(LogLevel.WARN, message, data);
  }

  error(message: string, error?: Error, data?: any) {
    this.log(LogLevel.ERROR, message, { ...data, error: error?.message });
    
    if (error) {
      Sentry.captureException(error, {
        contexts: {
          logger: this.context,
        },
        extra: data,
      });
    }
  }
}

export const logger = new Logger();
```

**Usage:**

```typescript
import { logger } from '../utils/logger';

function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    logger.setContext({ component: 'UserProfile', userId });
    logger.info('Fetching user profile');

    fetchUser(userId)
      .then((data) => {
        setUser(data);
        logger.debug('User profile loaded', { user: data });
      })
      .catch((error) => {
        logger.error('Failed to fetch user profile', error, { userId });
      });
  }, [userId]);

  return <div>{/* ... */}</div>;
}
```

---

#### **3. Remote Debugging with Console**

**Remote Console Implementation:**

```typescript
// src/utils/remoteConsole.ts
class RemoteConsole {
  private ws: WebSocket | null = null;
  private buffer: any[] = [];
  private enabled: boolean;

  constructor() {
    this.enabled = localStorage.getItem('remote_debug') === 'true';
    if (this.enabled) {
      this.connect();
    }
  }

  private connect() {
    this.ws = new WebSocket('ws://localhost:8080/debug');
    
    this.ws.onopen = () => {
      console.log('Remote console connected');
      this.flushBuffer();
    };

    this.ws.onclose = () => {
      console.log('Remote console disconnected');
      setTimeout(() => this.connect(), 5000);
    };
  }

  private send(data: any) {
    if (this.ws?.readyState === WebSocket.OPEN) {
      this.ws.send(JSON.stringify(data));
    } else {
      this.buffer.push(data);
    }
  }

  private flushBuffer() {
    while (this.buffer.length > 0) {
      const data = this.buffer.shift();
      this.send(data);
    }
  }

  log(level: string, args: any[]) {
    if (!this.enabled) return;

    this.send({
      type: 'console',
      level,
      args: args.map(arg => {
        try {
          return JSON.stringify(arg);
        } catch {
          return String(arg);
        }
      }),
      timestamp: Date.now(),
      url: window.location.href,
    });
  }
}

const remoteConsole = new RemoteConsole();

// Override console methods
if (import.meta.env.MODE === 'production') {
  const originalConsole = { ...console };
  
  ['log', 'debug', 'info', 'warn', 'error'].forEach((method) => {
    console[method] = (...args: any[]) => {
      originalConsole[method](...args);
      remoteConsole.log(method, args);
    };
  });
}
```

---

#### **4. Network Request Debugging**

```typescript
// src/utils/networkMonitor.ts
import { logger } from './logger';

interface RequestLog {
  id: string;
  url: string;
  method: string;
  status?: number;
  duration?: number;
  error?: string;
  requestHeaders?: any;
  responseHeaders?: any;
  requestBody?: any;
  responseBody?: any;
}

class NetworkMonitor {
  private requests: Map<string, RequestLog> = new Map();
  private enabled: boolean;

  constructor() {
    this.enabled = localStorage.getItem('debug_network') === 'true';
    if (this.enabled) {
      this.interceptFetch();
      this.interceptXHR();
    }
  }

  private interceptFetch() {
    const originalFetch = window.fetch;
    
    window.fetch = async (...args) => {
      const [url, options = {}] = args;
      const requestId = this.generateId();
      const startTime = performance.now();

      const requestLog: RequestLog = {
        id: requestId,
        url: String(url),
        method: options.method || 'GET',
        requestHeaders: options.headers,
        requestBody: options.body,
      };

      this.requests.set(requestId, requestLog);
      logger.debug('Network request started', requestLog);

      try {
        const response = await originalFetch(...args);
        const clonedResponse = response.clone();
        const duration = performance.now() - startTime;

        // Try to parse response body
        let responseBody;
        try {
          responseBody = await clonedResponse.text();
          if (responseBody) {
            responseBody = JSON.parse(responseBody);
          }
        } catch {}

        requestLog.status = response.status;
        requestLog.duration = duration;
        requestLog.responseHeaders = Object.fromEntries(response.headers.entries());
        requestLog.responseBody = responseBody;

        if (response.ok) {
          logger.debug('Network request completed', requestLog);
        } else {
          logger.warn('Network request failed', requestLog);
        }

        return response;
      } catch (error) {
        const duration = performance.now() - startTime;
        requestLog.duration = duration;
        requestLog.error = error instanceof Error ? error.message : 'Unknown error';
        
        logger.error('Network request error', error as Error, requestLog);
        throw error;
      }
    };
  }

  private interceptXHR() {
    const XHROpen = XMLHttpRequest.prototype.open;
    const XHRSend = XMLHttpRequest.prototype.send;

    XMLHttpRequest.prototype.open = function(...args) {
      (this as any)._requestId = Math.random().toString(36);
      (this as any)._method = args[0];
      (this as any)._url = args[1];
      return XHROpen.apply(this, args as any);
    };

    XMLHttpRequest.prototype.send = function(body) {
      const startTime = performance.now();
      const requestId = (this as any)._requestId;

      this.addEventListener('loadend', () => {
        const duration = performance.now() - startTime;
        logger.debug('XHR request completed', {
          id: requestId,
          method: (this as any)._method,
          url: (this as any)._url,
          status: this.status,
          duration,
        });
      });

      return XHRSend.call(this, body);
    };
  }

  private generateId(): string {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }

  getRequests(): RequestLog[] {
    return Array.from(this.requests.values());
  }
}

export const networkMonitor = new NetworkMonitor();
```

---

#### **5. State Debugging**

```typescript
// src/utils/stateDebugger.ts
import { useEffect, useRef } from 'react';
import { logger } from './logger';

export function useStateDebugger<T>(
  value: T,
  name: string
) {
  const prevValue = useRef<T>();
  const renderCount = useRef(0);

  useEffect(() => {
    renderCount.current++;

    if (prevValue.current !== undefined && prevValue.current !== value) {
      logger.debug(`State changed: ${name}`, {
        previous: prevValue.current,
        current: value,
        renderCount: renderCount.current,
      });
    }

    prevValue.current = value;
  });

  return value;
}

// Usage
function Counter() {
  const [count, setCount] = useState(0);
  useStateDebugger(count, 'count');

  return (
    <button onClick={() => setCount(c => c + 1)}>
      Count: {count}
    </button>
  );
}
```

---

#### **6. Performance Profiling**

```typescript
// src/utils/performanceProfiler.ts
import { logger } from './logger';

class PerformanceProfiler {
  private marks: Map<string, number> = new Map();

  mark(name: string) {
    const timestamp = performance.now();
    this.marks.set(name, timestamp);
    performance.mark(name);
  }

  measure(name: string, startMark: string, endMark?: string) {
    if (endMark) {
      performance.measure(name, startMark, endMark);
    } else {
      performance.measure(name, startMark);
    }

    const measure = performance.getEntriesByName(name)[0];
    logger.info(`Performance: ${name}`, {
      duration: measure.duration,
      startTime: measure.startTime,
    });

    return measure.duration;
  }

  getMetrics() {
    const navigation = performance.getEntriesByType('navigation')[0] as PerformanceNavigationTiming;
    
    return {
      dns: navigation.domainLookupEnd - navigation.domainLookupStart,
      tcp: navigation.connectEnd - navigation.connectStart,
      ttfb: navigation.responseStart - navigation.requestStart,
      download: navigation.responseEnd - navigation.responseStart,
      domInteractive: navigation.domInteractive,
      domComplete: navigation.domComplete,
      loadComplete: navigation.loadEventEnd,
    };
  }
}

export const profiler = new PerformanceProfiler();

// Usage
function DataFetchingComponent() {
  useEffect(() => {
    profiler.mark('fetch-start');
    
    fetchData().then(() => {
      profiler.mark('fetch-end');
      profiler.measure('data-fetch', 'fetch-start', 'fetch-end');
    });
  }, []);

  return <div>{/* ... */}</div>;
}
```

---

#### **7. User Session Replay**

**Using LogRocket:**

```typescript
import LogRocket from 'logrocket';
import setupLogRocketReact from 'logrocket-react';

export function initLogRocket() {
  if (import.meta.env.MODE === 'production') {
    LogRocket.init('my-app/my-project');
    setupLogRocketReact(LogRocket);

    // Identify user
    LogRocket.identify('user-123', {
      name: 'John Doe',
      email: 'john@example.com',
    });

    // Integrate with Sentry
    LogRocket.getSessionURL((sessionURL) => {
      Sentry.configureScope((scope) => {
        scope.setExtra('sessionURL', sessionURL);
      });
    });
  }
}
```

---

#### **8. Real-World Debugging Workflow**

```typescript
// src/services/debugService.ts
import * as Sentry from '@sentry/react';
import { logger } from '../utils/logger';
import { profiler } from '../utils/performanceProfiler';
import { networkMonitor } from '../utils/networkMonitor';

class DebugService {
  private sessionId: string;

  constructor() {
    this.sessionId = this.generateSessionId();
    this.setupGlobalErrorHandlers();
    this.collectEnvironmentInfo();
  }

  private generateSessionId(): string {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }

  private setupGlobalErrorHandlers() {
    // Unhandled promise rejections
    window.addEventListener('unhandledrejection', (event) => {
      logger.error('Unhandled promise rejection', event.reason, {
        sessionId: this.sessionId,
        promise: event.promise,
      });
      
      Sentry.captureException(event.reason, {
        tags: { type: 'unhandled-rejection' },
      });
    });

    // Global errors
    window.addEventListener('error', (event) => {
      logger.error('Global error', new Error(event.message), {
        sessionId: this.sessionId,
        filename: event.filename,
        lineno: event.lineno,
        colno: event.colno,
      });
    });
  }

  private collectEnvironmentInfo() {
    const info = {
      sessionId: this.sessionId,
      userAgent: navigator.userAgent,
      viewport: `${window.innerWidth}x${window.innerHeight}`,
      screen: `${window.screen.width}x${window.screen.height}`,
      language: navigator.language,
      timezone: Intl.DateTimeFormat().resolvedOptions().timeZone,
      online: navigator.onLine,
      cookiesEnabled: navigator.cookieEnabled,
      url: window.location.href,
      referrer: document.referrer,
    };

    logger.setContext(info);
    Sentry.setContext('environment', info);
  }

  captureError(error: Error, context?: any) {
    logger.error('Captured error', error, context);
    
    Sentry.captureException(error, {
      contexts: {
        debug: {
          sessionId: this.sessionId,
          ...context,
        },
      },
      extra: {
        networkRequests: networkMonitor.getRequests(),
        performanceMetrics: profiler.getMetrics(),
      },
    });
  }

  captureMessage(message: string, level: 'info' | 'warning' | 'error' = 'info') {
    logger[level === 'warning' ? 'warn' : level](message);
    Sentry.captureMessage(message, level as any);
  }

  // Enable debug mode
  enableDebugMode() {
    localStorage.setItem('debug', 'true');
    localStorage.setItem('debug_network', 'true');
    localStorage.setItem('remote_debug', 'true');
    console.log('Debug mode enabled. Refresh the page.');
  }

  // Collect debug report
  getDebugReport() {
    return {
      sessionId: this.sessionId,
      timestamp: new Date().toISOString(),
      url: window.location.href,
      performance: profiler.getMetrics(),
      network: networkMonitor.getRequests(),
      localStorage: { ...localStorage },
      sessionStorage: { ...sessionStorage },
    };
  }
}

export const debugService = new DebugService();

// Expose to window for manual debugging
if (typeof window !== 'undefined') {
  (window as any).debug = {
    enable: () => debugService.enableDebugMode(),
    report: () => console.log(debugService.getDebugReport()),
    clear: () => {
      localStorage.clear();
      sessionStorage.clear();
      console.log('Storage cleared');
    },
  };
}
```

---

#### **Best Practices**

1. **Source Maps**: Always upload source maps for production debugging
2. **Error Boundaries**: Catch React errors gracefully
3. **Logging**: Implement structured logging with context
4. **Session Replay**: Use tools like LogRocket or Sentry Replay
5. **Monitoring**: Set up alerts for critical errors
6. **Network Inspection**: Log API requests and responses
7. **Performance**: Track metrics and identify bottlenecks
8. **Context**: Include user, session, and environment info

---

#### **Summary**

**Production Debugging Tools:**

1. **Error Tracking**: Sentry with source maps for stack traces
2. **Logging**: Structured logger with context and levels
3. **Remote Console**: WebSocket-based remote debugging
4. **Network Monitor**: Intercept and log all HTTP requests
5. **State Debugger**: Track state changes and re-renders
6. **Performance Profiler**: Measure operation durations
7. **Session Replay**: Record user sessions with LogRocket
8. **Debug Service**: Centralized debugging utilities

**Key Techniques:**
- Source map upload for readable stack traces
- Breadcrumbs for debugging context
- User identification for tracking
- Environment info collection
- Network request logging
- Performance metrics tracking
- Global error handlers
- Remote debugging capabilities

**Debugging Workflow:**
1. Error occurs in production
2. Sentry captures with source maps
3. View readable stack trace
4. Check session replay for context
5. Review network requests
6. Analyze performance metrics
7. Reproduce locally if needed
8. Deploy fix and verify

Proper production debugging setup enables **rapid issue identification** and **efficient resolution**.

</details>

---

### 250. How do you implement feature usage tracking?

<details>
<summary>View Answer</summary>

**Feature usage tracking** helps understand how users interact with your application's features. This enables **data-driven decisions** about feature development, deprecation, and optimization.

---

#### **1. Basic Feature Tracking Hook**

```typescript
// src/hooks/useFeatureTracking.ts
import { useEffect, useRef } from 'react';
import { analytics } from '../services/analytics';

interface FeatureTrackingOptions {
  featureName: string;
  trackMount?: boolean;
  trackUnmount?: boolean;
  properties?: Record<string, any>;
}

export function useFeatureTracking(options: FeatureTrackingOptions) {
  const { featureName, trackMount = true, trackUnmount = false, properties = {} } = options;
  const mountTime = useRef<number>(0);

  useEffect(() => {
    mountTime.current = Date.now();

    if (trackMount) {
      analytics.trackEvent('feature_viewed', {
        feature: featureName,
        ...properties,
      });
    }

    return () => {
      if (trackUnmount) {
        const timeSpent = Date.now() - mountTime.current;
        analytics.trackEvent('feature_exited', {
          feature: featureName,
          timeSpent,
          ...properties,
        });
      }
    };
  }, [featureName, trackMount, trackUnmount]);

  const trackInteraction = (action: string, data?: any) => {
    analytics.trackEvent('feature_interaction', {
      feature: featureName,
      action,
      ...properties,
      ...data,
    });
  };

  const trackSuccess = (data?: any) => {
    analytics.trackEvent('feature_success', {
      feature: featureName,
      ...properties,
      ...data,
    });
  };

  const trackError = (error: Error, data?: any) => {
    analytics.trackEvent('feature_error', {
      feature: featureName,
      error: error.message,
      ...properties,
      ...data,
    });
  };

  return {
    trackInteraction,
    trackSuccess,
    trackError,
  };
}
```

**Usage:**

```typescript
function AdvancedSearch() {
  const { trackInteraction, trackSuccess } = useFeatureTracking({
    featureName: 'advanced_search',
    properties: { source: 'search_page' },
  });

  const handleFilterChange = (filter: string, value: any) => {
    trackInteraction('filter_changed', { filter, value });
  };

  const handleSearch = () => {
    trackInteraction('search_executed');
    // Perform search
    trackSuccess({ resultsCount: 42 });
  };

  return (
    <div>
      {/* Search UI */}
    </div>
  );
}
```

---

#### **2. Feature Flag Analytics**

```typescript
// src/services/featureFlags.ts
import { analytics } from './analytics';

interface FeatureFlag {
  name: string;
  enabled: boolean;
  variant?: string;
  metadata?: any;
}

class FeatureFlagsService {
  private flags: Map<string, FeatureFlag> = new Map();
  private exposures: Set<string> = new Set();

  async initialize() {
    // Fetch flags from backend or feature flag service
    const response = await fetch('/api/feature-flags');
    const flags = await response.json();

    flags.forEach((flag: FeatureFlag) => {
      this.flags.set(flag.name, flag);
    });
  }

  isEnabled(flagName: string): boolean {
    const flag = this.flags.get(flagName);
    const enabled = flag?.enabled ?? false;

    // Track exposure (first time user sees this flag)
    if (!this.exposures.has(flagName)) {
      this.exposures.add(flagName);
      analytics.trackEvent('feature_flag_exposed', {
        flag: flagName,
        enabled,
        variant: flag?.variant,
      });
    }

    return enabled;
  }

  getVariant(flagName: string): string | undefined {
    const flag = this.flags.get(flagName);
    return flag?.variant;
  }

  trackFeatureUsage(flagName: string, action: string, properties?: any) {
    const flag = this.flags.get(flagName);
    
    analytics.trackEvent('feature_used', {
      flag: flagName,
      enabled: flag?.enabled,
      variant: flag?.variant,
      action,
      ...properties,
    });
  }
}

export const featureFlags = new FeatureFlagsService();
```

**Usage with React:**

```typescript
// src/hooks/useFeatureFlag.ts
import { useState, useEffect } from 'react';
import { featureFlags } from '../services/featureFlags';

export function useFeatureFlag(flagName: string) {
  const [enabled, setEnabled] = useState(false);
  const [variant, setVariant] = useState<string>();

  useEffect(() => {
    setEnabled(featureFlags.isEnabled(flagName));
    setVariant(featureFlags.getVariant(flagName));
  }, [flagName]);

  const trackUsage = (action: string, properties?: any) => {
    featureFlags.trackFeatureUsage(flagName, action, properties);
  };

  return { enabled, variant, trackUsage };
}

// Component usage
function NewFeatureComponent() {
  const { enabled, trackUsage } = useFeatureFlag('new_checkout_flow');

  if (!enabled) {
    return <OldCheckout />;
  }

  const handleCheckout = () => {
    trackUsage('checkout_started');
    // Proceed with checkout
  };

  return <NewCheckout onCheckout={handleCheckout} />;
}
```

---

#### **3. Feature Adoption Metrics**

```typescript
// src/services/featureMetrics.ts
import { analytics } from './analytics';

interface FeatureMetric {
  featureName: string;
  firstUsed?: Date;
  lastUsed?: Date;
  useCount: number;
  totalTimeSpent: number;
  actions: Map<string, number>;
}

class FeatureMetricsService {
  private metrics: Map<string, FeatureMetric> = new Map();
  private storageKey = 'feature_metrics';

  constructor() {
    this.loadMetrics();
  }

  private loadMetrics() {
    try {
      const stored = localStorage.getItem(this.storageKey);
      if (stored) {
        const data = JSON.parse(stored);
        Object.entries(data).forEach(([key, value]: [string, any]) => {
          this.metrics.set(key, {
            ...value,
            firstUsed: value.firstUsed ? new Date(value.firstUsed) : undefined,
            lastUsed: value.lastUsed ? new Date(value.lastUsed) : undefined,
            actions: new Map(Object.entries(value.actions || {})),
          });
        });
      }
    } catch (error) {
      console.error('Failed to load feature metrics', error);
    }
  }

  private saveMetrics() {
    try {
      const data: any = {};
      this.metrics.forEach((metric, key) => {
        data[key] = {
          ...metric,
          actions: Object.fromEntries(metric.actions),
        };
      });
      localStorage.setItem(this.storageKey, JSON.stringify(data));
    } catch (error) {
      console.error('Failed to save feature metrics', error);
    }
  }

  trackFeatureUsage(featureName: string, action?: string) {
    let metric = this.metrics.get(featureName);

    if (!metric) {
      metric = {
        featureName,
        firstUsed: new Date(),
        lastUsed: new Date(),
        useCount: 0,
        totalTimeSpent: 0,
        actions: new Map(),
      };
      this.metrics.set(featureName, metric);
    }

    metric.useCount++;
    metric.lastUsed = new Date();

    if (action) {
      const actionCount = metric.actions.get(action) || 0;
      metric.actions.set(action, actionCount + 1);
    }

    this.saveMetrics();

    // Send to analytics
    analytics.trackEvent('feature_usage_metric', {
      feature: featureName,
      action,
      useCount: metric.useCount,
      daysSinceFirstUse: this.daysSince(metric.firstUsed!),
    });
  }

  trackTimeSpent(featureName: string, duration: number) {
    const metric = this.metrics.get(featureName);
    if (metric) {
      metric.totalTimeSpent += duration;
      this.saveMetrics();
    }
  }

  getFeatureAdoption(featureName: string) {
    const metric = this.metrics.get(featureName);
    if (!metric) return null;

    return {
      adopted: metric.useCount > 0,
      useCount: metric.useCount,
      firstUsed: metric.firstUsed,
      lastUsed: metric.lastUsed,
      daysSinceFirstUse: this.daysSince(metric.firstUsed!),
      daysSinceLastUse: this.daysSince(metric.lastUsed!),
      averageTimeSpent: metric.totalTimeSpent / metric.useCount,
      topActions: this.getTopActions(metric.actions),
    };
  }

  private daysSince(date: Date): number {
    return Math.floor((Date.now() - date.getTime()) / (1000 * 60 * 60 * 24));
  }

  private getTopActions(actions: Map<string, number>): Array<[string, number]> {
    return Array.from(actions.entries())
      .sort((a, b) => b[1] - a[1])
      .slice(0, 5);
  }

  // Send aggregated metrics to backend
  async syncMetrics() {
    const metricsData = Array.from(this.metrics.values());
    
    try {
      await fetch('/api/feature-metrics', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(metricsData),
      });
    } catch (error) {
      console.error('Failed to sync metrics', error);
    }
  }
}

export const featureMetrics = new FeatureMetricsService();

// Sync metrics periodically
setInterval(() => featureMetrics.syncMetrics(), 5 * 60 * 1000); // Every 5 minutes
```

---

#### **4. Feature Funnel Tracking**

```typescript
// src/hooks/useFeatureFunnel.ts
import { useState, useEffect } from 'react';
import { analytics } from '../services/analytics';

interface FunnelStep {
  name: string;
  completed: boolean;
  timestamp?: number;
}

export function useFeatureFunnel(funnelName: string, steps: string[]) {
  const [currentStep, setCurrentStep] = useState(0);
  const [funnelSteps, setFunnelSteps] = useState<FunnelStep[]>(
    steps.map(name => ({ name, completed: false }))
  );

  useEffect(() => {
    // Track funnel started
    analytics.trackEvent('funnel_started', {
      funnel: funnelName,
      steps,
    });
  }, [funnelName]);

  const completeStep = (stepIndex: number) => {
    setFunnelSteps(prev => {
      const updated = [...prev];
      updated[stepIndex] = {
        ...updated[stepIndex],
        completed: true,
        timestamp: Date.now(),
      };
      return updated;
    });

    setCurrentStep(stepIndex + 1);

    analytics.trackEvent('funnel_step_completed', {
      funnel: funnelName,
      step: steps[stepIndex],
      stepIndex,
      totalSteps: steps.length,
    });

    // Track funnel completion
    if (stepIndex === steps.length - 1) {
      const startTime = funnelSteps[0].timestamp || Date.now();
      const endTime = Date.now();
      const duration = endTime - startTime;

      analytics.trackEvent('funnel_completed', {
        funnel: funnelName,
        duration,
        steps: funnelSteps,
      });
    }
  };

  const abandonFunnel = () => {
    const completedSteps = funnelSteps.filter(s => s.completed).length;
    
    analytics.trackEvent('funnel_abandoned', {
      funnel: funnelName,
      abandonedAt: steps[currentStep],
      completedSteps,
      totalSteps: steps.length,
      progress: (completedSteps / steps.length) * 100,
    });
  };

  return {
    currentStep,
    funnelSteps,
    completeStep,
    abandonFunnel,
  };
}
```

**Usage:**

```typescript
function OnboardingFlow() {
  const { currentStep, completeStep, abandonFunnel } = useFeatureFunnel(
    'user_onboarding',
    ['account_created', 'profile_completed', 'first_action', 'tutorial_finished']
  );

  useEffect(() => {
    // Track abandonment when user leaves
    return () => abandonFunnel();
  }, []);

  const handleProfileSubmit = () => {
    completeStep(1); // Complete "profile_completed" step
  };

  return (
    <div>
      {currentStep === 0 && <CreateAccount onComplete={() => completeStep(0)} />}
      {currentStep === 1 && <CompleteProfile onComplete={handleProfileSubmit} />}
      {currentStep === 2 && <FirstAction onComplete={() => completeStep(2)} />}
      {currentStep === 3 && <Tutorial onComplete={() => completeStep(3)} />}
    </div>
  );
}
```

---

#### **5. Cohort Analysis**

```typescript
// src/services/cohortAnalytics.ts
import { analytics } from './analytics';

interface UserCohort {
  cohortId: string;
  cohortName: string;
  signupDate: Date;
  features: Set<string>;
}

class CohortAnalytics {
  private cohort: UserCohort | null = null;

  identifyUser(userId: string, signupDate: Date) {
    // Determine cohort based on signup date
    const cohortId = this.getCohortId(signupDate);
    
    this.cohort = {
      cohortId,
      cohortName: this.getCohortName(signupDate),
      signupDate,
      features: new Set(),
    };

    analytics.setUserProperties({
      cohort_id: cohortId,
      signup_date: signupDate.toISOString(),
    });
  }

  trackFeatureAdoption(featureName: string) {
    if (!this.cohort) return;

    if (!this.cohort.features.has(featureName)) {
      this.cohort.features.add(featureName);

      const daysSinceSignup = this.daysSince(this.cohort.signupDate);

      analytics.trackEvent('cohort_feature_adoption', {
        cohort_id: this.cohort.cohortId,
        cohort_name: this.cohort.cohortName,
        feature: featureName,
        days_since_signup: daysSinceSignup,
        total_features_adopted: this.cohort.features.size,
      });
    }
  }

  private getCohortId(signupDate: Date): string {
    const year = signupDate.getFullYear();
    const month = signupDate.getMonth() + 1;
    return `${year}-${month.toString().padStart(2, '0')}`;
  }

  private getCohortName(signupDate: Date): string {
    const monthNames = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun',
                        'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'];
    return `${monthNames[signupDate.getMonth()]} ${signupDate.getFullYear()}`;
  }

  private daysSince(date: Date): number {
    return Math.floor((Date.now() - date.getTime()) / (1000 * 60 * 60 * 24));
  }
}

export const cohortAnalytics = new CohortAnalytics();
```

---

#### **6. Real-World Example: Feature Dashboard**

```typescript
// src/components/FeatureUsageDashboard.tsx
import { useState, useEffect } from 'react';
import { featureMetrics } from '../services/featureMetrics';

interface FeatureStats {
  name: string;
  useCount: number;
  adopted: boolean;
  daysSinceFirstUse: number;
  daysSinceLastUse: number;
  averageTimeSpent: number;
  topActions: Array<[string, number]>;
}

function FeatureUsageDashboard() {
  const [features, setFeatures] = useState<FeatureStats[]>([]);

  useEffect(() => {
    const featureNames = [
      'advanced_search',
      'export_data',
      'bulk_edit',
      'custom_reports',
      'api_integration',
    ];

    const stats = featureNames
      .map(name => ({
        name,
        ...featureMetrics.getFeatureAdoption(name),
      }))
      .filter(f => f.adopted);

    setFeatures(stats as FeatureStats[]);
  }, []);

  return (
    <div className="feature-dashboard">
      <h2>Feature Usage Dashboard</h2>
      
      <div className="feature-grid">
        {features.map(feature => (
          <div key={feature.name} className="feature-card">
            <h3>{feature.name}</h3>
            
            <div className="stats">
              <div className="stat">
                <label>Total Uses</label>
                <value>{feature.useCount}</value>
              </div>
              
              <div className="stat">
                <label>Days Active</label>
                <value>{feature.daysSinceFirstUse}</value>
              </div>
              
              <div className="stat">
                <label>Last Used</label>
                <value>
                  {feature.daysSinceLastUse === 0
                    ? 'Today'
                    : `${feature.daysSinceLastUse} days ago`}
                </value>
              </div>
              
              <div className="stat">
                <label>Avg Time</label>
                <value>{Math.round(feature.averageTimeSpent / 1000)}s</value>
              </div>
            </div>

            <div className="top-actions">
              <h4>Top Actions</h4>
              <ul>
                {feature.topActions.map(([action, count]) => (
                  <li key={action}>
                    {action}: {count}
                  </li>
                ))}
              </ul>
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

1. **Privacy First**: Get user consent for tracking
2. **Anonymization**: Don't track PII unless necessary
3. **Sampling**: Don't track 100% of events in high-traffic apps
4. **Batching**: Batch analytics events to reduce network calls
5. **Offline Support**: Queue events when offline
6. **Clear Events**: Use consistent naming conventions
7. **Context**: Include relevant context with each event
8. **Testing**: Verify tracking in development

---

#### **Summary**

**Feature Usage Tracking Implementation:**

1. **Basic Tracking**: useFeatureTracking hook for views and interactions
2. **Feature Flags**: Track flag exposures and usage
3. **Adoption Metrics**: Monitor feature adoption over time
4. **Funnel Analysis**: Track multi-step feature workflows
5. **Cohort Analysis**: Analyze feature adoption by user cohorts
6. **Dashboard**: Visualize feature usage metrics

**Key Metrics:**
- **Adoption Rate**: % of users who used feature
- **Engagement**: Frequency and duration of use
- **Retention**: How often users return to feature
- **Funnel Completion**: % completing multi-step flows
- **Time to Adoption**: How long until first use
- **Feature Stickiness**: DAU/MAU ratio

**Use Cases:**
- Identify underutilized features
- Measure feature success
- Inform product roadmap
- A/B test feature variations
- Detect feature issues early
- Justify development resources
- Improve onboarding flows

**Integration Points:**
- Google Analytics 4
- Mixpanel
- Amplitude
- Segment
- LaunchDarkly
- Custom analytics backend

Proper feature usage tracking enables **data-driven product decisions** and **continuous improvement** of user experience.

</details>
