# Real-World Scenarios

> Expert / Architect Level (5+ years)

---

## Questions

231. How do you handle memory leaks in React?
232. How do you debug performance issues in production?
233. How do you implement undo/redo functionality?
234. How do you handle offline functionality in React apps?
235. How do you implement real-time collaboration features?
236. How do you handle large file uploads with progress?
237. How do you implement infinite scrolling efficiently?
238. How do you handle internationalization (i18n)?
239. How do you implement dark mode in React?
240. How do you handle browser compatibility issues?

---

## Detailed Answers

### 231. How do you handle memory leaks in React?

<details>
<summary>View Answer</summary>

**Memory leaks** in React occur when components hold references to resources (timers, subscriptions, event listeners) after unmounting, preventing garbage collection. This leads to increased memory usage, degraded performance, and potential crashes.

---

#### **Common Causes of Memory Leaks**

**1. Uncleared Timers:**

```typescript
// ❌ Memory leak - timer continues after unmount
function BadComponent() {
  useEffect(() => {
    setInterval(() => {
      console.log('Running...');
    }, 1000);
    // No cleanup!
  }, []);
}

// ✅ Fixed - cleanup clears timer
function GoodComponent() {
  useEffect(() => {
    const intervalId = setInterval(() => {
      console.log('Running...');
    }, 1000);
    
    return () => clearInterval(intervalId);  // Cleanup
  }, []);
}
```

**2. Event Listeners:**

```typescript
// ❌ Memory leak - event listener not removed
function BadComponent() {
  useEffect(() => {
    const handleScroll = () => {
      console.log('Scrolling...');
    };
    
    window.addEventListener('scroll', handleScroll);
    // No cleanup!
  }, []);
}

// ✅ Fixed - cleanup removes listener
function GoodComponent() {
  useEffect(() => {
    const handleScroll = () => {
      console.log('Scrolling...');
    };
    
    window.addEventListener('scroll', handleScroll);
    
    return () => window.removeEventListener('scroll', handleScroll);
  }, []);
}
```

**3. Subscriptions (WebSocket, EventEmitter):**

```typescript
// ❌ Memory leak - subscription not closed
function BadComponent() {
  useEffect(() => {
    const socket = new WebSocket('ws://localhost:3000');
    
    socket.onmessage = (event) => {
      console.log(event.data);
    };
    // No cleanup!
  }, []);
}

// ✅ Fixed - cleanup closes connection
function GoodComponent() {
  useEffect(() => {
    const socket = new WebSocket('ws://localhost:3000');
    
    socket.onmessage = (event) => {
      console.log(event.data);
    };
    
    return () => socket.close();
  }, []);
}
```

**4. Async Operations with State Updates:**

```typescript
// ❌ Memory leak - setState after unmount
function BadComponent() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetch('/api/data')
      .then(res => res.json())
      .then(setData);  // Component may be unmounted!
  }, []);
}

// ✅ Fixed - check if mounted
function GoodComponent() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    let isMounted = true;
    
    fetch('/api/data')
      .then(res => res.json())
      .then(result => {
        if (isMounted) {
          setData(result);
        }
      });
    
    return () => {
      isMounted = false;
    };
  }, []);
}

// ✅ Better - use AbortController
function BestComponent() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    const controller = new AbortController();
    
    fetch('/api/data', { signal: controller.signal })
      .then(res => res.json())
      .then(setData)
      .catch(err => {
        if (err.name !== 'AbortError') {
          console.error(err);
        }
      });
    
    return () => controller.abort();
  }, []);
}
```

**5. References in Closures:**

```typescript
// ❌ Memory leak - closure holds old reference
function BadComponent() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      console.log(count);  // Always logs 0 (stale closure)
      setCount(count + 1);  // Creates memory leak
    }, 1000);
    
    return () => clearInterval(interval);
  }, []);  // Empty deps - closure captures initial count
}

// ✅ Fixed - use functional update
function GoodComponent() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setCount(c => c + 1);  // Functional update, no closure issue
    }, 1000);
    
    return () => clearInterval(interval);
  }, []);
}
```

---

#### **Detection Tools**

**1. React DevTools Profiler:**

```typescript
// Identify components that don't unmount properly
// 1. Open React DevTools
// 2. Go to Profiler tab
// 3. Record interactions
// 4. Look for components that remain mounted when they shouldn't
```

**2. Chrome DevTools Memory Profiler:**

```typescript
// Detect memory leaks
// 1. Open Chrome DevTools → Memory tab
// 2. Take heap snapshot
// 3. Perform actions (mount/unmount components)
// 4. Take another snapshot
// 5. Compare snapshots
// 6. Look for detached DOM nodes and increasing object counts
```

**3. Memory Leak Detector Hook:**

```typescript
// src/hooks/useMemoryLeakDetector.ts
import { useEffect, useRef } from 'react';

export function useMemoryLeakDetector(componentName: string) {
  const mountTimeRef = useRef<number>();
  
  useEffect(() => {
    mountTimeRef.current = Date.now();
    console.log(`[${componentName}] Mounted`);
    
    return () => {
      const lifetime = Date.now() - (mountTimeRef.current || 0);
      console.log(`[${componentName}] Unmounted after ${lifetime}ms`);
      
      // Check for active timers/listeners
      if (process.env.NODE_ENV === 'development') {
        setTimeout(() => {
          console.warn(
            `[${componentName}] Checking for leaks 1 second after unmount`
          );
        }, 1000);
      }
    };
  }, [componentName]);
}

// Usage
function MyComponent() {
  useMemoryLeakDetector('MyComponent');
  // ... rest of component
}
```

---

#### **Prevention Patterns**

**1. Custom Hook for Cleanup:**

```typescript
// src/hooks/useCleanup.ts
import { useEffect, useRef } from 'react';

export function useCleanup() {
  const cleanupFns = useRef<Array<() => void>>([]);
  
  const addCleanup = (fn: () => void) => {
    cleanupFns.current.push(fn);
  };
  
  useEffect(() => {
    return () => {
      cleanupFns.current.forEach(fn => fn());
      cleanupFns.current = [];
    };
  }, []);
  
  return addCleanup;
}

// Usage
function Component() {
  const addCleanup = useCleanup();
  
  useEffect(() => {
    const interval = setInterval(() => console.log('tick'), 1000);
    addCleanup(() => clearInterval(interval));
    
    const socket = new WebSocket('ws://localhost:3000');
    addCleanup(() => socket.close());
    
    const listener = () => console.log('resize');
    window.addEventListener('resize', listener);
    addCleanup(() => window.removeEventListener('resize', listener));
  }, [addCleanup]);
}
```

**2. Safe Async Hook:**

```typescript
// src/hooks/useSafeAsync.ts
import { useEffect, useCallback, useRef } from 'react';

export function useSafeAsync() {
  const isMountedRef = useRef(true);
  
  useEffect(() => {
    isMountedRef.current = true;
    return () => {
      isMountedRef.current = false;
    };
  }, []);
  
  const safeAsync = useCallback(
    <T>(promise: Promise<T>): Promise<T> => {
      return new Promise((resolve, reject) => {
        promise
          .then(result => {
            if (isMountedRef.current) {
              resolve(result);
            }
          })
          .catch(error => {
            if (isMountedRef.current) {
              reject(error);
            }
          });
      });
    },
    []
  );
  
  return safeAsync;
}

// Usage
function Component() {
  const [data, setData] = useState(null);
  const safeAsync = useSafeAsync();
  
  useEffect(() => {
    safeAsync(fetch('/api/data').then(r => r.json()))
      .then(setData)
      .catch(console.error);
  }, [safeAsync]);
}
```

**3. Timer Management Hook:**

```typescript
// src/hooks/useTimeout.ts
import { useEffect, useRef } from 'react';

export function useTimeout(callback: () => void, delay: number | null) {
  const savedCallback = useRef(callback);
  
  useEffect(() => {
    savedCallback.current = callback;
  }, [callback]);
  
  useEffect(() => {
    if (delay === null) return;
    
    const id = setTimeout(() => savedCallback.current(), delay);
    return () => clearTimeout(id);
  }, [delay]);
}

export function useInterval(callback: () => void, delay: number | null) {
  const savedCallback = useRef(callback);
  
  useEffect(() => {
    savedCallback.current = callback;
  }, [callback]);
  
  useEffect(() => {
    if (delay === null) return;
    
    const id = setInterval(() => savedCallback.current(), delay);
    return () => clearInterval(id);
  }, [delay]);
}

// Usage
function CountdownTimer() {
  const [count, setCount] = useState(10);
  
  useInterval(() => {
    setCount(c => c - 1);
  }, count > 0 ? 1000 : null);  // Auto-stops at 0
  
  return <div>{count}</div>;
}
```

**4. Event Listener Hook:**

```typescript
// src/hooks/useEventListener.ts
import { useEffect, useRef } from 'react';

export function useEventListener<K extends keyof WindowEventMap>(
  eventName: K,
  handler: (event: WindowEventMap[K]) => void,
  element: Window | HTMLElement = window
) {
  const savedHandler = useRef(handler);
  
  useEffect(() => {
    savedHandler.current = handler;
  }, [handler]);
  
  useEffect(() => {
    const isSupported = element && element.addEventListener;
    if (!isSupported) return;
    
    const eventListener = (event: Event) => {
      savedHandler.current(event as WindowEventMap[K]);
    };
    
    element.addEventListener(eventName, eventListener);
    
    return () => {
      element.removeEventListener(eventName, eventListener);
    };
  }, [eventName, element]);
}

// Usage
function Component() {
  const [scrollY, setScrollY] = useState(0);
  
  useEventListener('scroll', () => {
    setScrollY(window.scrollY);
  });
  
  return <div>Scroll position: {scrollY}</div>;
}
```

---

#### **Real-World Example: Dashboard with Multiple Subscriptions**

```typescript
// src/components/Dashboard.tsx
import { useState, useEffect, useRef } from 'react';

interface DashboardData {
  metrics: Record<string, number>;
  alerts: string[];
  users: number;
}

function Dashboard() {
  const [data, setData] = useState<DashboardData>({
    metrics: {},
    alerts: [],
    users: 0,
  });
  
  const wsRef = useRef<WebSocket | null>(null);
  const intervalRef = useRef<number | null>(null);
  const isMountedRef = useRef(true);
  
  useEffect(() => {
    isMountedRef.current = true;
    
    // 1. WebSocket connection
    const ws = new WebSocket('ws://localhost:3000/dashboard');
    wsRef.current = ws;
    
    ws.onmessage = (event) => {
      if (!isMountedRef.current) return;
      
      const update = JSON.parse(event.data);
      setData(prev => ({
        ...prev,
        ...update,
      }));
    };
    
    ws.onerror = (error) => {
      console.error('WebSocket error:', error);
    };
    
    // 2. Polling for metrics
    intervalRef.current = window.setInterval(async () => {
      if (!isMountedRef.current) return;
      
      try {
        const response = await fetch('/api/metrics');
        const metrics = await response.json();
        
        if (isMountedRef.current) {
          setData(prev => ({ ...prev, metrics }));
        }
      } catch (error) {
        console.error('Failed to fetch metrics:', error);
      }
    }, 5000);
    
    // 3. Event listeners
    const handleVisibilityChange = () => {
      if (!isMountedRef.current) return;
      
      if (document.hidden) {
        // Pause updates when tab is hidden
        ws.close();
        if (intervalRef.current) {
          clearInterval(intervalRef.current);
        }
      } else {
        // Resume updates when tab is visible
        // Reconnect WebSocket and restart polling
      }
    };
    
    document.addEventListener('visibilitychange', handleVisibilityChange);
    
    // 4. Cleanup all subscriptions
    return () => {
      isMountedRef.current = false;
      
      // Close WebSocket
      if (wsRef.current) {
        wsRef.current.close();
        wsRef.current = null;
      }
      
      // Clear interval
      if (intervalRef.current) {
        clearInterval(intervalRef.current);
        intervalRef.current = null;
      }
      
      // Remove event listener
      document.removeEventListener('visibilitychange', handleVisibilityChange);
    };
  }, []);
  
  return (
    <div className="dashboard">
      <h1>Dashboard</h1>
      
      <div className="metrics">
        {Object.entries(data.metrics).map(([key, value]) => (
          <div key={key} className="metric">
            <span>{key}</span>
            <strong>{value}</strong>
          </div>
        ))}
      </div>
      
      <div className="alerts">
        {data.alerts.map((alert, i) => (
          <div key={i} className="alert">{alert}</div>
        ))}
      </div>
      
      <div className="users">
        Active Users: {data.users}
      </div>
    </div>
  );
}

export default Dashboard;
```

---

#### **Testing for Memory Leaks**

**1. Automated Testing:**

```typescript
// src/components/__tests__/Component.test.tsx
import { render, unmountComponentAtNode } from 'react-dom';
import { act } from 'react-dom/test-utils';
import Component from '../Component';

describe('Memory Leak Tests', () => {
  let container: HTMLDivElement;
  
  beforeEach(() => {
    container = document.createElement('div');
    document.body.appendChild(container);
  });
  
  afterEach(() => {
    unmountComponentAtNode(container);
    container.remove();
  });
  
  test('cleans up timers on unmount', () => {
    jest.useFakeTimers();
    
    act(() => {
      render(<Component />, container);
    });
    
    // Unmount component
    act(() => {
      unmountComponentAtNode(container);
    });
    
    // Verify no timers are running
    expect(jest.getTimerCount()).toBe(0);
    
    jest.useRealTimers();
  });
  
  test('removes event listeners on unmount', () => {
    const addEventListenerSpy = jest.spyOn(window, 'addEventListener');
    const removeEventListenerSpy = jest.spyOn(window, 'removeEventListener');
    
    act(() => {
      render(<Component />, container);
    });
    
    const addCallCount = addEventListenerSpy.mock.calls.length;
    
    act(() => {
      unmountComponentAtNode(container);
    });
    
    const removeCallCount = removeEventListenerSpy.mock.calls.length;
    
    // All added listeners should be removed
    expect(removeCallCount).toBe(addCallCount);
    
    addEventListenerSpy.mockRestore();
    removeEventListenerSpy.mockRestore();
  });
});
```

**2. Memory Profiling Script:**

```typescript
// scripts/memory-test.ts
import puppeteer from 'puppeteer';

async function testMemoryLeaks() {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  
  await page.goto('http://localhost:3000');
  
  // Take baseline measurement
  const baseline = await page.metrics();
  console.log('Baseline memory:', baseline.JSHeapUsedSize);
  
  // Mount/unmount component 100 times
  for (let i = 0; i < 100; i++) {
    await page.click('#mount-component');
    await page.waitForTimeout(100);
    await page.click('#unmount-component');
    await page.waitForTimeout(100);
  }
  
  // Force garbage collection
  await page.evaluate(() => {
    if (window.gc) {
      window.gc();
    }
  });
  
  // Measure final memory
  const final = await page.metrics();
  console.log('Final memory:', final.JSHeapUsedSize);
  
  const increase = final.JSHeapUsedSize - baseline.JSHeapUsedSize;
  const increasePercent = (increase / baseline.JSHeapUsedSize) * 100;
  
  console.log(`Memory increase: ${increase} bytes (${increasePercent.toFixed(2)}%)`);
  
  // Fail if memory increased by more than 10%
  if (increasePercent > 10) {
    throw new Error('Possible memory leak detected!');
  }
  
  await browser.close();
}

testMemoryLeaks().catch(console.error);
```

---

#### **ESLint Rules for Prevention**

```json
// .eslintrc.json
{
  "plugins": ["react-hooks"],
  "rules": {
    "react-hooks/rules-of-hooks": "error",
    "react-hooks/exhaustive-deps": "warn",
    "no-console": ["warn", { "allow": ["warn", "error"] }]
  }
}
```

**Custom ESLint Rule:**

```typescript
// eslint-plugin-custom/rules/cleanup-effect.js
module.exports = {
  meta: {
    type: 'problem',
    docs: {
      description: 'Enforce cleanup in useEffect with timers/listeners',
    },
  },
  create(context) {
    return {
      CallExpression(node) {
        // Check for useEffect
        if (node.callee.name !== 'useEffect') return;
        
        const effectCallback = node.arguments[0];
        if (!effectCallback) return;
        
        const body = effectCallback.body;
        const hasCleanup = body.body.some(
          stmt => stmt.type === 'ReturnStatement'
        );
        
        // Check for timers
        const hasTimer = /setInterval|setTimeout/.test(context.getSourceCode().getText(body));
        
        // Check for event listeners
        const hasListener = /addEventListener/.test(context.getSourceCode().getText(body));
        
        if ((hasTimer || hasListener) && !hasCleanup) {
          context.report({
            node,
            message: 'useEffect with timers/listeners should return cleanup function',
          });
        }
      },
    };
  },
};
```

---

#### **Best Practices Checklist**

✅ **Always return cleanup functions from useEffect**

```typescript
useEffect(() => {
  const id = setInterval(() => {}, 1000);
  return () => clearInterval(id);  // ✅ Always cleanup
}, []);
```

✅ **Use AbortController for fetch requests**

```typescript
useEffect(() => {
  const controller = new AbortController();
  fetch('/api', { signal: controller.signal });
  return () => controller.abort();  // ✅ Cancel pending requests
}, []);
```

✅ **Check mounted state before setState**

```typescript
useEffect(() => {
  let isMounted = true;
  asyncOperation().then(data => {
    if (isMounted) setData(data);  // ✅ Check before updating
  });
  return () => { isMounted = false; };
}, []);
```

✅ **Remove all event listeners**

```typescript
useEffect(() => {
  const handler = () => {};
  window.addEventListener('scroll', handler);
  return () => window.removeEventListener('scroll', handler);  // ✅ Remove
}, []);
```

✅ **Close WebSocket connections**

```typescript
useEffect(() => {
  const ws = new WebSocket('ws://...');
  return () => ws.close();  // ✅ Close connection
}, []);
```

✅ **Clear all timers**

```typescript
useEffect(() => {
  const timeout = setTimeout(() => {}, 1000);
  const interval = setInterval(() => {}, 1000);
  return () => {
    clearTimeout(timeout);    // ✅ Clear all timers
    clearInterval(interval);
  };
}, []);
```

---

#### **Summary**

**Memory Leak Prevention in React:**

1. **Always cleanup in useEffect**: Return cleanup function for timers, listeners, subscriptions
2. **Use AbortController**: Cancel pending fetch requests on unmount
3. **Check mounted state**: Prevent setState on unmounted components
4. **Remove event listeners**: Use cleanup to remove all listeners
5. **Close connections**: Clean up WebSockets, SSE, and other connections
6. **Use custom hooks**: Create reusable hooks for common cleanup patterns
7. **Test regularly**: Use Chrome DevTools and automated tests
8. **Enable strict mode**: Catches some leak patterns in development

**Detection Tools:**
- React DevTools Profiler
- Chrome Memory Profiler
- Custom memory leak detectors
- Automated testing

**Common Patterns:**
- `useCleanup` hook for multiple cleanups
- `useSafeAsync` for safe async operations
- `useEventListener` for automatic cleanup
- `useInterval`/`useTimeout` for safe timers

Proper memory management ensures React applications remain **fast, responsive, and stable** even after prolonged use.

</details>

---

### 232. How do you debug performance issues in production?

<details>
<summary>View Answer</summary>

**Debugging performance issues in production** requires a different approach than development, as you can't use React DevTools profiler or enable debug mode. You need production-safe monitoring, real user metrics (RUM), and lightweight profiling techniques.

---

#### **Performance Monitoring Tools**

**1. Web Vitals (Core Metrics):**

```typescript
// src/utils/webVitals.ts
import { getCLS, getFID, getFCP, getLCP, getTTFB } from 'web-vitals';

interface AnalyticsPayload {
  name: string;
  value: number;
  id: string;
  delta: number;
}

function sendToAnalytics(metric: AnalyticsPayload) {
  // Send to your analytics service
  fetch('/api/analytics', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      metric: metric.name,
      value: metric.value,
      id: metric.id,
      timestamp: Date.now(),
      url: window.location.href,
      userAgent: navigator.userAgent,
    }),
  });
}

export function initWebVitals() {
  getCLS(sendToAnalytics);  // Cumulative Layout Shift
  getFID(sendToAnalytics);  // First Input Delay
  getFCP(sendToAnalytics);  // First Contentful Paint
  getLCP(sendToAnalytics);  // Largest Contentful Paint
  getTTFB(sendToAnalytics); // Time to First Byte
}

// Initialize in your app
// src/index.tsx
import { initWebVitals } from './utils/webVitals';

initWebVitals();
```

**2. Performance Observer API:**

```typescript
// src/utils/performanceMonitor.ts
export class PerformanceMonitor {
  private static instance: PerformanceMonitor;
  private observers: PerformanceObserver[] = [];

  static getInstance() {
    if (!this.instance) {
      this.instance = new PerformanceMonitor();
    }
    return this.instance;
  }

  init() {
    // Monitor long tasks (tasks taking > 50ms)
    this.observeLongTasks();
    
    // Monitor resource loading
    this.observeResources();
    
    // Monitor navigation timing
    this.observeNavigation();
  }

  private observeLongTasks() {
    if (!('PerformanceObserver' in window)) return;

    try {
      const observer = new PerformanceObserver((list) => {
        for (const entry of list.getEntries()) {
          if (entry.duration > 50) {
            this.reportIssue({
              type: 'long-task',
              duration: entry.duration,
              startTime: entry.startTime,
              name: entry.name,
            });
          }
        }
      });

      observer.observe({ entryTypes: ['longtask'] });
      this.observers.push(observer);
    } catch (e) {
      console.error('Long task observer failed:', e);
    }
  }

  private observeResources() {
    if (!('PerformanceObserver' in window)) return;

    const observer = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        const resource = entry as PerformanceResourceTiming;
        
        // Report slow resources (> 1s)
        if (resource.duration > 1000) {
          this.reportIssue({
            type: 'slow-resource',
            url: resource.name,
            duration: resource.duration,
            size: resource.transferSize,
          });
        }
      }
    });

    observer.observe({ entryTypes: ['resource'] });
    this.observers.push(observer);
  }

  private observeNavigation() {
    if (!('PerformanceObserver' in window)) return;

    const observer = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        const nav = entry as PerformanceNavigationTiming;
        
        this.reportIssue({
          type: 'navigation',
          domContentLoaded: nav.domContentLoadedEventEnd - nav.domContentLoadedEventStart,
          loadComplete: nav.loadEventEnd - nav.loadEventStart,
          domInteractive: nav.domInteractive - nav.fetchStart,
        });
      }
    });

    observer.observe({ entryTypes: ['navigation'] });
    this.observers.push(observer);
  }

  private reportIssue(data: any) {
    // Send to monitoring service
    fetch('/api/performance-issue', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        ...data,
        timestamp: Date.now(),
        url: window.location.href,
        userAgent: navigator.userAgent,
      }),
    }).catch(console.error);
  }

  disconnect() {
    this.observers.forEach(observer => observer.disconnect());
    this.observers = [];
  }
}

// Initialize
PerformanceMonitor.getInstance().init();
```

---

#### **React-Specific Performance Tracking**

**1. Component Render Tracking:**

```typescript
// src/utils/renderTracker.ts
import { useEffect, useRef } from 'react';

interface RenderMetric {
  component: string;
  renderCount: number;
  lastRenderTime: number;
  totalRenderTime: number;
}

const renderMetrics = new Map<string, RenderMetric>();

export function useRenderTracking(componentName: string) {
  const renderCount = useRef(0);
  const startTime = useRef(performance.now());

  useEffect(() => {
    renderCount.current++;
    const renderTime = performance.now() - startTime.current;

    const existing = renderMetrics.get(componentName) || {
      component: componentName,
      renderCount: 0,
      lastRenderTime: 0,
      totalRenderTime: 0,
    };

    renderMetrics.set(componentName, {
      ...existing,
      renderCount: renderCount.current,
      lastRenderTime: renderTime,
      totalRenderTime: existing.totalRenderTime + renderTime,
    });

    // Report slow renders (> 16ms for 60fps)
    if (renderTime > 16 && process.env.NODE_ENV === 'production') {
      fetch('/api/slow-render', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          component: componentName,
          renderTime,
          renderCount: renderCount.current,
          url: window.location.href,
        }),
      }).catch(() => {});
    }
  });

  // Reset start time for next render
  startTime.current = performance.now();
}

// Export metrics for debugging
export function getRenderMetrics() {
  return Array.from(renderMetrics.values())
    .sort((a, b) => b.totalRenderTime - a.totalRenderTime);
}

// Usage in components
function ExpensiveComponent() {
  useRenderTracking('ExpensiveComponent');
  // ... component logic
}
```

**2. User Interaction Tracking:**

```typescript
// src/utils/interactionTracker.ts
export function trackInteraction(action: string, metadata?: any) {
  const startTime = performance.now();

  return () => {
    const duration = performance.now() - startTime;

    // Report slow interactions (> 100ms)
    if (duration > 100) {
      fetch('/api/slow-interaction', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          action,
          duration,
          metadata,
          timestamp: Date.now(),
          url: window.location.href,
        }),
      }).catch(() => {});
    }
  };
}

// Usage
function Button() {
  const handleClick = () => {
    const endTracking = trackInteraction('button-click', { buttonId: 'submit' });
    
    // Perform action
    performExpensiveOperation();
    
    endTracking();
  };

  return <button onClick={handleClick}>Submit</button>;
}
```

---

#### **Production Debugging Techniques**

**1. Feature Flags for Debugging:**

```typescript
// src/utils/featureFlags.ts
interface FeatureFlags {
  enableDebugMode: boolean;
  enablePerformanceLogging: boolean;
  logSlowRenders: boolean;
}

export const featureFlags: FeatureFlags = {
  enableDebugMode: localStorage.getItem('debug') === 'true',
  enablePerformanceLogging: localStorage.getItem('perf-log') === 'true',
  logSlowRenders: localStorage.getItem('log-slow-renders') === 'true',
};

// Enable debugging: localStorage.setItem('debug', 'true')
// Then reload page

// src/components/Component.tsx
import { featureFlags } from '../utils/featureFlags';

function Component() {
  useEffect(() => {
    if (featureFlags.enableDebugMode) {
      console.log('[Component] Mounted', { props, state });
    }
  }, []);

  if (featureFlags.logSlowRenders) {
    useRenderTracking('Component');
  }

  // ... component logic
}
```

**2. Conditional Error Boundaries:**

```typescript
// src/components/ProductionErrorBoundary.tsx
import React, { Component, ErrorInfo } from 'react';

interface Props {
  children: React.ReactNode;
  fallback?: React.ReactNode;
}

interface State {
  hasError: boolean;
  error: Error | null;
  errorInfo: ErrorInfo | null;
}

export class ProductionErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = {
      hasError: false,
      error: null,
      errorInfo: null,
    };
  }

  static getDerivedStateFromError(error: Error): Partial<State> {
    return { hasError: true };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    // Log to monitoring service
    this.logErrorToService(error, errorInfo);

    this.setState({
      error,
      errorInfo,
    });
  }

  private logErrorToService(error: Error, errorInfo: ErrorInfo) {
    fetch('/api/error', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        message: error.message,
        stack: error.stack,
        componentStack: errorInfo.componentStack,
        timestamp: Date.now(),
        url: window.location.href,
        userAgent: navigator.userAgent,
      }),
    }).catch(console.error);
  }

  render() {
    if (this.state.hasError) {
      // Show debug info if flag enabled
      if (featureFlags.enableDebugMode) {
        return (
          <div style={{ padding: 20, border: '2px solid red' }}>
            <h2>Error Details (Debug Mode)</h2>
            <pre style={{ color: 'red' }}>
              {this.state.error?.message}
            </pre>
            <details>
              <summary>Stack Trace</summary>
              <pre>{this.state.error?.stack}</pre>
            </details>
            <details>
              <summary>Component Stack</summary>
              <pre>{this.state.errorInfo?.componentStack}</pre>
            </details>
          </div>
        );
      }

      return this.props.fallback || <div>Something went wrong.</div>;
    }

    return this.props.children;
  }
}
```

---

#### **Real-World Example: E-Commerce Performance Monitoring**

```typescript
// src/monitoring/ecommerceMonitoring.ts
import { getCLS, getFID, getLCP } from 'web-vitals';

interface PageMetrics {
  page: string;
  metrics: {
    cls: number;
    fid: number;
    lcp: number;
  };
  interactions: Array<{
    action: string;
    duration: number;
  }>;
  slowComponents: Array<{
    name: string;
    renderTime: number;
  }>;
}

class EcommerceMonitoring {
  private metrics: Partial<PageMetrics> = {
    page: window.location.pathname,
    interactions: [],
    slowComponents: [],
  };

  init() {
    // Track Core Web Vitals
    getCLS((metric) => {
      this.metrics.metrics = { ...this.metrics.metrics, cls: metric.value } as any;
    });

    getFID((metric) => {
      this.metrics.metrics = { ...this.metrics.metrics, fid: metric.value } as any;
    });

    getLCP((metric) => {
      this.metrics.metrics = { ...this.metrics.metrics, lcp: metric.value } as any;
      
      // Send metrics when LCP is captured
      this.sendMetrics();
    });

    // Track critical user flows
    this.trackAddToCart();
    this.trackCheckout();
    this.trackProductView();
  }

  private trackAddToCart() {
    // Listen for custom events
    window.addEventListener('addToCart', ((event: CustomEvent) => {
      const startTime = event.detail.startTime;
      const endTime = performance.now();
      const duration = endTime - startTime;

      this.metrics.interactions?.push({
        action: 'add-to-cart',
        duration,
      });

      // Alert if slow (> 500ms)
      if (duration > 500) {
        this.sendAlert({
          type: 'slow-add-to-cart',
          duration,
          productId: event.detail.productId,
        });
      }
    }) as EventListener);
  }

  private trackCheckout() {
    window.addEventListener('checkoutComplete', ((event: CustomEvent) => {
      const duration = event.detail.duration;

      this.metrics.interactions?.push({
        action: 'checkout',
        duration,
      });

      // Send immediately for critical path
      this.sendMetrics();
    }) as EventListener);
  }

  private trackProductView() {
    // Track time to interactive for product pages
    if (window.location.pathname.includes('/product/')) {
      const observer = new PerformanceObserver((list) => {
        for (const entry of list.getEntries()) {
          if (entry.name === 'product-interactive') {
            this.metrics.interactions?.push({
              action: 'product-view',
              duration: entry.duration,
            });
          }
        }
      });

      observer.observe({ entryTypes: ['measure'] });
    }
  }

  trackSlowComponent(name: string, renderTime: number) {
    this.metrics.slowComponents?.push({ name, renderTime });
  }

  private sendMetrics() {
    fetch('/api/metrics', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        ...this.metrics,
        timestamp: Date.now(),
        sessionId: sessionStorage.getItem('sessionId'),
      }),
    }).catch(() => {});
  }

  private sendAlert(alert: any) {
    fetch('/api/alert', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        ...alert,
        timestamp: Date.now(),
        url: window.location.href,
      }),
    }).catch(() => {});
  }
}

export const monitoring = new EcommerceMonitoring();

// Initialize in app
monitoring.init();
```

**Usage in Components:**

```typescript
// src/components/ProductPage.tsx
import { monitoring } from '../monitoring/ecommerceMonitoring';

function ProductPage({ productId }: Props) {
  const [product, setProduct] = useState(null);
  const startTime = useRef(performance.now());

  useEffect(() => {
    fetchProduct(productId).then((data) => {
      setProduct(data);
      
      // Mark as interactive
      performance.mark('product-interactive');
      performance.measure(
        'product-interactive',
        'navigationStart',
        'product-interactive'
      );
    });
  }, [productId]);

  const handleAddToCart = () => {
    const eventStartTime = performance.now();
    
    addToCart(product).then(() => {
      // Dispatch custom event for monitoring
      window.dispatchEvent(
        new CustomEvent('addToCart', {
          detail: {
            startTime: eventStartTime,
            productId: product.id,
          },
        })
      );
    });
  };

  // Track slow renders
  useRenderTracking('ProductPage');

  if (!product) return <Skeleton />;

  return (
    <div>
      <h1>{product.name}</h1>
      <p>${product.price}</p>
      <button onClick={handleAddToCart}>Add to Cart</button>
    </div>
  );
}
```

---

#### **Integration with Monitoring Services**

**1. Sentry Integration:**

```typescript
// src/monitoring/sentry.ts
import * as Sentry from '@sentry/react';
import { BrowserTracing } from '@sentry/tracing';

Sentry.init({
  dsn: process.env.REACT_APP_SENTRY_DSN,
  integrations: [
    new BrowserTracing(),
    new Sentry.Replay({
      maskAllText: false,
      blockAllMedia: false,
    }),
  ],
  
  // Performance monitoring
  tracesSampleRate: 0.1, // 10% of transactions
  
  // Session replay
  replaysSessionSampleRate: 0.01, // 1% of sessions
  replaysOnErrorSampleRate: 1.0, // 100% on errors
  
  // Custom performance tracking
  beforeSend(event, hint) {
    // Add custom context
    if (event.contexts) {
      event.contexts.render = {
        metrics: getRenderMetrics(),
      };
    }
    return event;
  },
});

// Track custom performance metrics
export function trackTransaction(name: string, operation: string) {
  const transaction = Sentry.startTransaction({ name, op: operation });
  
  return {
    finish: () => transaction.finish(),
    setTag: (key: string, value: string) => transaction.setTag(key, value),
    setData: (key: string, value: any) => transaction.setData(key, value),
  };
}
```

**2. DataDog RUM:**

```typescript
// src/monitoring/datadog.ts
import { datadogRum } from '@datadog/browser-rum';

datadogRum.init({
  applicationId: process.env.REACT_APP_DD_APP_ID!,
  clientToken: process.env.REACT_APP_DD_CLIENT_TOKEN!,
  site: 'datadoghq.com',
  service: 'ecommerce-frontend',
  env: process.env.NODE_ENV,
  version: process.env.REACT_APP_VERSION,
  
  // Session replay
  sessionSampleRate: 100,
  sessionReplaySampleRate: 20,
  
  // Track user actions
  trackUserInteractions: true,
  trackResources: true,
  trackLongTasks: true,
  
  // Default privacy level
  defaultPrivacyLevel: 'mask-user-input',
});

// Start tracking
datadogRum.startSessionReplayRecording();

// Custom actions
export function trackAction(name: string, context?: any) {
  datadogRum.addAction(name, context);
}

// Custom timing
export function trackTiming(name: string, duration: number) {
  datadogRum.addTiming(name, duration);
}
```

---

#### **Debug Console for Production**

```typescript
// src/utils/debugConsole.ts
class DebugConsole {
  private logs: Array<{ type: string; message: string; timestamp: number }> = [];
  private isEnabled = false;

  enable() {
    this.isEnabled = true;
    localStorage.setItem('debug-console', 'enabled');
  }

  disable() {
    this.isEnabled = false;
    localStorage.removeItem('debug-console');
  }

  log(message: string, data?: any) {
    if (!this.isEnabled) return;

    const entry = {
      type: 'log',
      message,
      data,
      timestamp: Date.now(),
    };

    this.logs.push(entry);
    console.log(`[DEBUG] ${message}`, data);
  }

  error(message: string, error?: any) {
    if (!this.isEnabled) return;

    const entry = {
      type: 'error',
      message,
      error,
      timestamp: Date.now(),
    };

    this.logs.push(entry);
    console.error(`[DEBUG] ${message}`, error);
  }

  getLogs() {
    return this.logs;
  }

  downloadLogs() {
    const blob = new Blob([JSON.stringify(this.logs, null, 2)], {
      type: 'application/json',
    });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `debug-logs-${Date.now()}.json`;
    a.click();
  }
}

export const debugConsole = new DebugConsole();

// Enable via console: debugConsole.enable()
// Download logs: debugConsole.downloadLogs()
```

---

#### **Best Practices**

**1. Sampling for High-Traffic Apps:**

```typescript
// Only track 10% of sessions to reduce overhead
const shouldTrack = Math.random() < 0.1;

if (shouldTrack) {
  initPerformanceTracking();
}
```

**2. Lazy Load Monitoring Code:**

```typescript
// Only load monitoring in production
if (process.env.NODE_ENV === 'production') {
  import('./monitoring/sentry').then(({ init }) => init());
  import('./monitoring/datadog').then(({ init }) => init());
}
```

**3. Aggregate Before Sending:**

```typescript
// Batch metrics to reduce network requests
const metricsBatch: any[] = [];

function addMetric(metric: any) {
  metricsBatch.push(metric);
  
  // Send every 10 metrics or after 30 seconds
  if (metricsBatch.length >= 10) {
    sendMetricsBatch();
  }
}

setInterval(sendMetricsBatch, 30000);
```

---

#### **Summary**

**Production Performance Debugging:**

1. **Web Vitals**: Track CLS, FID, FCP, LCP, TTFB
2. **Performance Observer**: Monitor long tasks, slow resources, navigation
3. **Custom Tracking**: Component renders, user interactions, critical flows
4. **Feature Flags**: Enable debugging for specific users
5. **Error Boundaries**: Catch and report errors with context
6. **Monitoring Services**: Sentry, DataDog, New Relic for comprehensive tracking
7. **Debug Console**: Production-safe logging with download capability
8. **Sampling**: Reduce overhead by tracking subset of users

**Key Metrics to Track:**
- Core Web Vitals (LCP, FID, CLS)
- Time to Interactive (TTI)
- Component render times
- User interaction latency
- API response times
- Resource loading times
- Error rates and types

Production debugging requires a **proactive monitoring approach** with the right balance of detailed tracking and performance overhead.

</details>

---

### 233. How do you implement undo/redo functionality?

<details>
<summary>View Answer</summary>

**Undo/Redo functionality** allows users to reverse and reapply actions, improving user experience by making mistakes recoverable. Implementation typically involves maintaining a **history stack** of state snapshots or actions.

---

#### **Basic Approaches**

**1. State Snapshot Approach:**
Store complete copies of state at each step.

```typescript
interface HistoryState<T> {
  past: T[];
  present: T;
  future: T[];
}
```

**2. Command Pattern Approach:**
Store actions with undo/redo logic (more efficient for complex state).

```typescript
interface Command {
  execute: () => void;
  undo: () => void;
}
```

---

#### **Implementation: useUndo Hook**

**Basic Version (State Snapshots):**

```typescript
// src/hooks/useUndo.ts
import { useState, useCallback } from 'react';

interface UseUndoReturn<T> {
  state: T;
  setState: (newState: T | ((prev: T) => T)) => void;
  undo: () => void;
  redo: () => void;
  canUndo: boolean;
  canRedo: boolean;
  reset: (newState: T) => void;
  history: {
    past: T[];
    present: T;
    future: T[];
  };
}

export function useUndo<T>(initialState: T): UseUndoReturn<T> {
  const [history, setHistory] = useState({
    past: [] as T[],
    present: initialState,
    future: [] as T[],
  });

  const setState = useCallback((newState: T | ((prev: T) => T)) => {
    setHistory((currentHistory) => {
      const newPresent =
        typeof newState === 'function'
          ? (newState as (prev: T) => T)(currentHistory.present)
          : newState;

      // Don't add if state hasn't changed
      if (newPresent === currentHistory.present) {
        return currentHistory;
      }

      return {
        past: [...currentHistory.past, currentHistory.present],
        present: newPresent,
        future: [], // Clear future when new action is performed
      };
    });
  }, []);

  const undo = useCallback(() => {
    setHistory((currentHistory) => {
      if (currentHistory.past.length === 0) return currentHistory;

      const previous = currentHistory.past[currentHistory.past.length - 1];
      const newPast = currentHistory.past.slice(0, -1);

      return {
        past: newPast,
        present: previous,
        future: [currentHistory.present, ...currentHistory.future],
      };
    });
  }, []);

  const redo = useCallback(() => {
    setHistory((currentHistory) => {
      if (currentHistory.future.length === 0) return currentHistory;

      const next = currentHistory.future[0];
      const newFuture = currentHistory.future.slice(1);

      return {
        past: [...currentHistory.past, currentHistory.present],
        present: next,
        future: newFuture,
      };
    });
  }, []);

  const reset = useCallback((newState: T) => {
    setHistory({
      past: [],
      present: newState,
      future: [],
    });
  }, []);

  return {
    state: history.present,
    setState,
    undo,
    redo,
    canUndo: history.past.length > 0,
    canRedo: history.future.length > 0,
    reset,
    history,
  };
}
```

**Usage:**

```typescript
// src/components/TextEditor.tsx
import { useUndo } from '../hooks/useUndo';

function TextEditor() {
  const {
    state: text,
    setState: setText,
    undo,
    redo,
    canUndo,
    canRedo,
  } = useUndo('');

  return (
    <div>
      <div className="toolbar">
        <button onClick={undo} disabled={!canUndo}>
          ↶ Undo
        </button>
        <button onClick={redo} disabled={!canRedo}>
          ↷ Redo
        </button>
      </div>

      <textarea
        value={text}
        onChange={(e) => setText(e.target.value)}
        placeholder="Start typing..."
      />
    </div>
  );
}
```

---

#### **Advanced: Debounced History**

**Problem**: Every keystroke creates a history entry.

**Solution**: Debounce state updates to create logical history points.

```typescript
// src/hooks/useUndoWithDebounce.ts
import { useState, useCallback, useRef, useEffect } from 'react';
import { useUndo } from './useUndo';

export function useUndoWithDebounce<T>(
  initialState: T,
  delay: number = 500
) {
  const [tempState, setTempState] = useState(initialState);
  const {
    state: historicalState,
    setState: setHistoricalState,
    undo,
    redo,
    canUndo,
    canRedo,
    reset,
  } = useUndo(initialState);

  const timeoutRef = useRef<number | null>(null);

  // Sync temp state with historical state when undo/redo
  useEffect(() => {
    setTempState(historicalState);
  }, [historicalState]);

  const setState = useCallback(
    (newState: T | ((prev: T) => T)) => {
      const nextState =
        typeof newState === 'function'
          ? (newState as (prev: T) => T)(tempState)
          : newState;

      setTempState(nextState);

      // Clear existing timeout
      if (timeoutRef.current) {
        clearTimeout(timeoutRef.current);
      }

      // Set new timeout to add to history
      timeoutRef.current = window.setTimeout(() => {
        setHistoricalState(nextState);
      }, delay);
    },
    [tempState, delay, setHistoricalState]
  );

  return {
    state: tempState,
    setState,
    undo,
    redo,
    canUndo,
    canRedo,
    reset,
  };
}

// Usage
function TextEditor() {
  const {
    state: text,
    setState: setText,
    undo,
    redo,
    canUndo,
    canRedo,
  } = useUndoWithDebounce('', 500); // Group changes within 500ms

  return (
    <div>
      <button onClick={undo} disabled={!canUndo}>Undo</button>
      <button onClick={redo} disabled={!canRedo}>Redo</button>
      <textarea value={text} onChange={(e) => setText(e.target.value)} />
    </div>
  );
}
```

---

#### **Command Pattern Implementation**

**For Complex State Changes:**

```typescript
// src/hooks/useCommandHistory.ts
import { useState, useCallback } from 'react';

export interface Command<T> {
  execute: (state: T) => T;
  undo: (state: T) => T;
  description?: string;
}

export function useCommandHistory<T>(initialState: T) {
  const [state, setState] = useState(initialState);
  const [history, setHistory] = useState<Command<T>[]>([]);
  const [currentIndex, setCurrentIndex] = useState(-1);

  const executeCommand = useCallback((command: Command<T>) => {
    setState((currentState) => {
      const newState = command.execute(currentState);

      // Remove future commands
      setHistory((hist) => [
        ...hist.slice(0, currentIndex + 1),
        command,
      ]);
      setCurrentIndex((idx) => idx + 1);

      return newState;
    });
  }, [currentIndex]);

  const undo = useCallback(() => {
    if (currentIndex < 0) return;

    const command = history[currentIndex];
    setState((currentState) => command.undo(currentState));
    setCurrentIndex((idx) => idx - 1);
  }, [history, currentIndex]);

  const redo = useCallback(() => {
    if (currentIndex >= history.length - 1) return;

    const command = history[currentIndex + 1];
    setState((currentState) => command.execute(currentState));
    setCurrentIndex((idx) => idx + 1);
  }, [history, currentIndex]);

  return {
    state,
    executeCommand,
    undo,
    redo,
    canUndo: currentIndex >= 0,
    canRedo: currentIndex < history.length - 1,
    history: history.map((cmd) => cmd.description || 'Unknown'),
  };
}
```

**Usage with Command Pattern:**

```typescript
// src/components/DrawingApp.tsx
import { useCommandHistory, Command } from '../hooks/useCommandHistory';

interface DrawingState {
  shapes: Array<{ id: string; type: string; x: number; y: number }>;
}

function DrawingApp() {
  const { state, executeCommand, undo, redo, canUndo, canRedo } =
    useCommandHistory<DrawingState>({
      shapes: [],
    });

  const addShape = (shape: DrawingState['shapes'][0]) => {
    const command: Command<DrawingState> = {
      execute: (state) => ({
        shapes: [...state.shapes, shape],
      }),
      undo: (state) => ({
        shapes: state.shapes.filter((s) => s.id !== shape.id),
      }),
      description: `Add ${shape.type}`,
    };

    executeCommand(command);
  };

  const deleteShape = (shapeId: string) => {
    const shapeToDelete = state.shapes.find((s) => s.id === shapeId);
    if (!shapeToDelete) return;

    const command: Command<DrawingState> = {
      execute: (state) => ({
        shapes: state.shapes.filter((s) => s.id !== shapeId),
      }),
      undo: (state) => ({
        shapes: [...state.shapes, shapeToDelete],
      }),
      description: `Delete ${shapeToDelete.type}`,
    };

    executeCommand(command);
  };

  const moveShape = (shapeId: string, newX: number, newY: number) => {
    const shape = state.shapes.find((s) => s.id === shapeId);
    if (!shape) return;

    const oldX = shape.x;
    const oldY = shape.y;

    const command: Command<DrawingState> = {
      execute: (state) => ({
        shapes: state.shapes.map((s) =>
          s.id === shapeId ? { ...s, x: newX, y: newY } : s
        ),
      }),
      undo: (state) => ({
        shapes: state.shapes.map((s) =>
          s.id === shapeId ? { ...s, x: oldX, y: oldY } : s
        ),
      }),
      description: `Move ${shape.type}`,
    };

    executeCommand(command);
  };

  return (
    <div>
      <div className="toolbar">
        <button onClick={undo} disabled={!canUndo}>
          Undo
        </button>
        <button onClick={redo} disabled={!canRedo}>
          Redo
        </button>
        <button onClick={() => addShape({ id: Date.now().toString(), type: 'circle', x: 100, y: 100 })}>
          Add Circle
        </button>
      </div>

      <svg width="800" height="600">
        {state.shapes.map((shape) => (
          <circle
            key={shape.id}
            cx={shape.x}
            cy={shape.y}
            r="30"
            fill="blue"
            onClick={() => deleteShape(shape.id)}
            onMouseDown={(e) => {
              // Implement drag logic that calls moveShape
            }}
          />
        ))}
      </svg>
    </div>
  );
}
```

---

#### **Real-World Example: Rich Text Editor**

```typescript
// src/components/RichTextEditor.tsx
import { useUndoWithDebounce } from '../hooks/useUndoWithDebounce';
import { useState } from 'react';

interface EditorState {
  content: string;
  selection: { start: number; end: number };
  formatting: {
    bold: boolean;
    italic: boolean;
    underline: boolean;
  };
}

function RichTextEditor() {
  const {
    state,
    setState,
    undo,
    redo,
    canUndo,
    canRedo,
  } = useUndoWithDebounce<EditorState>(
    {
      content: '',
      selection: { start: 0, end: 0 },
      formatting: { bold: false, italic: false, underline: false },
    },
    500
  );

  const handleTextChange = (newContent: string) => {
    setState((prev) => ({
      ...prev,
      content: newContent,
    }));
  };

  const toggleBold = () => {
    setState((prev) => ({
      ...prev,
      formatting: { ...prev.formatting, bold: !prev.formatting.bold },
    }));
  };

  const toggleItalic = () => {
    setState((prev) => ({
      ...prev,
      formatting: { ...prev.formatting, italic: !prev.formatting.italic },
    }));
  };

  // Keyboard shortcuts
  const handleKeyDown = (e: React.KeyboardEvent) => {
    if (e.ctrlKey || e.metaKey) {
      if (e.key === 'z') {
        e.preventDefault();
        if (e.shiftKey) {
          redo();
        } else {
          undo();
        }
      }
      if (e.key === 'y') {
        e.preventDefault();
        redo();
      }
    }
  };

  return (
    <div className="editor" onKeyDown={handleKeyDown}>
      <div className="toolbar">
        <button onClick={undo} disabled={!canUndo} title="Undo (Ctrl+Z)">
          ↶ Undo
        </button>
        <button onClick={redo} disabled={!canRedo} title="Redo (Ctrl+Shift+Z)">
          ↷ Redo
        </button>
        <div className="separator" />
        <button
          onClick={toggleBold}
          className={state.formatting.bold ? 'active' : ''}
        >
          <strong>B</strong>
        </button>
        <button
          onClick={toggleItalic}
          className={state.formatting.italic ? 'active' : ''}
        >
          <em>I</em>
        </button>
      </div>

      <textarea
        value={state.content}
        onChange={(e) => handleTextChange(e.target.value)}
        className="content"
        style={{
          fontWeight: state.formatting.bold ? 'bold' : 'normal',
          fontStyle: state.formatting.italic ? 'italic' : 'normal',
          textDecoration: state.formatting.underline ? 'underline' : 'none',
        }}
      />
    </div>
  );
}

export default RichTextEditor;
```

---

#### **With Redux/Zustand**

**Redux Implementation:**

```typescript
// src/store/undoableReducer.ts
import { Reducer } from 'redux';

interface UndoableState<T> {
  past: T[];
  present: T;
  future: T[];
}

export function undoable<T>(
  reducer: Reducer<T>,
  config: { limit?: number } = {}
): Reducer<UndoableState<T>> {
  const { limit = 50 } = config;

  const initialState: UndoableState<T> = {
    past: [],
    present: reducer(undefined, { type: '@@INIT' }) as T,
    future: [],
  };

  return (state = initialState, action): UndoableState<T> => {
    switch (action.type) {
      case 'UNDO': {
        if (state.past.length === 0) return state;

        const previous = state.past[state.past.length - 1];
        const newPast = state.past.slice(0, -1);

        return {
          past: newPast,
          present: previous,
          future: [state.present, ...state.future],
        };
      }

      case 'REDO': {
        if (state.future.length === 0) return state;

        const next = state.future[0];
        const newFuture = state.future.slice(1);

        return {
          past: [...state.past, state.present],
          present: next,
          future: newFuture,
        };
      }

      case 'CLEAR_HISTORY': {
        return {
          past: [],
          present: state.present,
          future: [],
        };
      }

      default: {
        const newPresent = reducer(state.present, action);

        if (newPresent === state.present) {
          return state;
        }

        return {
          past: [...state.past.slice(-limit + 1), state.present],
          present: newPresent,
          future: [],
        };
      }
    }
  };
}

// Usage
import { createStore } from 'redux';
import { undoable } from './undoableReducer';

const todoReducer = (state = [], action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, action.payload];
    case 'REMOVE_TODO':
      return state.filter(todo => todo.id !== action.payload);
    default:
      return state;
  }
};

const store = createStore(undoable(todoReducer, { limit: 50 }));

// Actions
store.dispatch({ type: 'UNDO' });
store.dispatch({ type: 'REDO' });
```

**Zustand Implementation:**

```typescript
// src/store/useStore.ts
import create from 'zustand';
import { temporal } from 'zundo';

interface State {
  todos: Array<{ id: string; text: string }>;
  addTodo: (text: string) => void;
  removeTodo: (id: string) => void;
}

export const useStore = create<State>()((
  temporal(
    (set) => ({
      todos: [],
      addTodo: (text) =>
        set((state) => ({
          todos: [...state.todos, { id: Date.now().toString(), text }],
        })),
      removeTodo: (id) =>
        set((state) => ({
          todos: state.todos.filter((todo) => todo.id !== id),
        })),
    }),
    { limit: 50 }
  )
));

// Usage
function TodoList() {
  const { todos, addTodo, removeTodo } = useStore();
  const { undo, redo, canUndo, canRedo } = useStore.temporal.getState();

  return (
    <div>
      <button onClick={undo} disabled={!canUndo}>Undo</button>
      <button onClick={redo} disabled={!canRedo}>Redo</button>
      {/* ... */}
    </div>
  );
}
```

---

#### **Optimization: History Limit**

```typescript
// Limit history to prevent memory issues
export function useUndoWithLimit<T>(
  initialState: T,
  limit: number = 50
) {
  const [history, setHistory] = useState({
    past: [] as T[],
    present: initialState,
    future: [] as T[],
  });

  const setState = useCallback((newState: T | ((prev: T) => T)) => {
    setHistory((currentHistory) => {
      const newPresent =
        typeof newState === 'function'
          ? (newState as (prev: T) => T)(currentHistory.present)
          : newState;

      if (newPresent === currentHistory.present) return currentHistory;

      // Keep only last 'limit' items
      const newPast =
        currentHistory.past.length >= limit
          ? [...currentHistory.past.slice(1), currentHistory.present]
          : [...currentHistory.past, currentHistory.present];

      return {
        past: newPast,
        present: newPresent,
        future: [],
      };
    });
  }, [limit]);

  // ... rest of implementation
}
```

---

#### **Keyboard Shortcuts**

```typescript
// src/hooks/useUndoShortcuts.ts
import { useEffect } from 'react';

export function useUndoShortcuts(
  undo: () => void,
  redo: () => void,
  canUndo: boolean,
  canRedo: boolean
) {
  useEffect(() => {
    const handleKeyDown = (e: KeyboardEvent) => {
      // Ctrl+Z or Cmd+Z for undo
      if ((e.ctrlKey || e.metaKey) && e.key === 'z' && !e.shiftKey) {
        e.preventDefault();
        if (canUndo) undo();
      }

      // Ctrl+Shift+Z or Cmd+Shift+Z or Ctrl+Y for redo
      if (
        ((e.ctrlKey || e.metaKey) && e.key === 'z' && e.shiftKey) ||
        (e.ctrlKey && e.key === 'y')
      ) {
        e.preventDefault();
        if (canRedo) redo();
      }
    };

    window.addEventListener('keydown', handleKeyDown);
    return () => window.removeEventListener('keydown', handleKeyDown);
  }, [undo, redo, canUndo, canRedo]);
}

// Usage
function Editor() {
  const { state, setState, undo, redo, canUndo, canRedo } = useUndo('');
  
  useUndoShortcuts(undo, redo, canUndo, canRedo);

  return <textarea value={state} onChange={(e) => setState(e.target.value)} />;
}
```

---

#### **Best Practices**

**1. Debounce rapid changes** (typing, dragging)
**2. Limit history size** to prevent memory issues
**3. Use command pattern** for complex operations
**4. Add keyboard shortcuts** (Ctrl+Z, Ctrl+Shift+Z)
**5. Show history list** for power users
**6. Clear future** when new action performed
**7. Deep clone state** for objects/arrays
**8. Optimize with immer** for immutable updates

---

#### **Summary**

**Undo/Redo Implementation:**

1. **State Snapshots**: Simple approach, stores complete state copies
2. **Command Pattern**: More efficient, stores actions with execute/undo logic
3. **History Structure**: past[], present, future[]
4. **Debouncing**: Group rapid changes into logical history points
5. **History Limit**: Prevent memory issues by limiting stored states
6. **Keyboard Shortcuts**: Ctrl+Z (undo), Ctrl+Shift+Z (redo)
7. **Integration**: Works with useState, Redux, Zustand
8. **Optimization**: Use immer for efficient immutable updates

**Key Patterns:**
- `useUndo` hook for basic undo/redo
- `useUndoWithDebounce` for text editing
- `useCommandHistory` for complex operations
- `undoable` HOC for Redux
- `temporal` middleware for Zustand

Undo/redo functionality significantly improves **user experience and confidence**, allowing users to experiment without fear of losing work.

</details>

---

### 234. How do you handle offline functionality in React apps?

<details>
<summary>View Answer</summary>

**Offline functionality** allows React apps to work without internet connection by caching assets, storing data locally, and syncing when connection is restored. This is essential for **Progressive Web Apps (PWA)** and improving user experience in unreliable network conditions.

---

#### **Core Technologies**

**1. Service Workers** - Cache assets and intercept network requests
**2. IndexedDB** - Store large amounts of structured data
**3. Cache API** - Store and retrieve HTTP responses
**4. Background Sync** - Sync data when connection is restored
**5. LocalStorage/SessionStorage** - Store small amounts of key-value data

---

#### **Service Worker Setup**

**1. Register Service Worker:**

```typescript
// src/index.tsx
import { registerServiceWorker } from './serviceWorkerRegistration';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);

// Register service worker
registerServiceWorker();
```

**2. Service Worker Registration:**

```typescript
// src/serviceWorkerRegistration.ts
export function registerServiceWorker() {
  if ('serviceWorker' in navigator) {
    window.addEventListener('load', () => {
      navigator.serviceWorker
        .register('/service-worker.js')
        .then((registration) => {
          console.log('SW registered:', registration);

          // Check for updates
          registration.addEventListener('updatefound', () => {
            const newWorker = registration.installing;
            if (newWorker) {
              newWorker.addEventListener('statechange', () => {
                if (newWorker.state === 'installed' && navigator.serviceWorker.controller) {
                  // New content available, prompt user to refresh
                  if (window.confirm('New version available! Refresh to update?')) {
                    window.location.reload();
                  }
                }
              });
            }
          });
        })
        .catch((error) => {
          console.error('SW registration failed:', error);
        });
    });
  }
}

export function unregisterServiceWorker() {
  if ('serviceWorker' in navigator) {
    navigator.serviceWorker.ready
      .then((registration) => {
        registration.unregister();
      })
      .catch((error) => {
        console.error('SW unregistration failed:', error);
      });
  }
}
```

**3. Service Worker Implementation:**

```javascript
// public/service-worker.js
const CACHE_NAME = 'my-app-v1';
const STATIC_CACHE = 'static-v1';
const DYNAMIC_CACHE = 'dynamic-v1';

const STATIC_ASSETS = [
  '/',
  '/index.html',
  '/static/js/main.js',
  '/static/css/main.css',
  '/manifest.json',
  '/logo192.png',
];

// Install event - cache static assets
self.addEventListener('install', (event) => {
  console.log('[SW] Installing...');
  
  event.waitUntil(
    caches.open(STATIC_CACHE).then((cache) => {
      console.log('[SW] Caching static assets');
      return cache.addAll(STATIC_ASSETS);
    })
  );
  
  self.skipWaiting(); // Activate immediately
});

// Activate event - clean up old caches
self.addEventListener('activate', (event) => {
  console.log('[SW] Activating...');
  
  event.waitUntil(
    caches.keys().then((cacheNames) => {
      return Promise.all(
        cacheNames
          .filter((name) => name !== STATIC_CACHE && name !== DYNAMIC_CACHE)
          .map((name) => caches.delete(name))
      );
    })
  );
  
  return self.clients.claim(); // Take control of all pages
});

// Fetch event - serve from cache, fallback to network
self.addEventListener('fetch', (event) => {
  const { request } = event;
  
  // Cache-first strategy for static assets
  if (request.url.includes('/static/')) {
    event.respondWith(
      caches.match(request).then((response) => {
        return response || fetch(request).then((fetchResponse) => {
          return caches.open(STATIC_CACHE).then((cache) => {
            cache.put(request, fetchResponse.clone());
            return fetchResponse;
          });
        });
      })
    );
    return;
  }
  
  // Network-first strategy for API calls
  if (request.url.includes('/api/')) {
    event.respondWith(
      fetch(request)
        .then((response) => {
          // Cache successful responses
          if (response.ok) {
            return caches.open(DYNAMIC_CACHE).then((cache) => {
              cache.put(request, response.clone());
              return response;
            });
          }
          return response;
        })
        .catch(() => {
          // Fallback to cache if network fails
          return caches.match(request).then((response) => {
            return response || new Response(
              JSON.stringify({ error: 'Offline', cached: true }),
              { headers: { 'Content-Type': 'application/json' } }
            );
          });
        })
    );
    return;
  }
  
  // Default: cache-first
  event.respondWith(
    caches.match(request).then((response) => {
      return response || fetch(request);
    })
  );
});
```

---

#### **Cache Strategies**

**1. Cache-First (Cache Falling Back to Network):**

```javascript
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then((response) => {
      return response || fetch(event.request);
    })
  );
});
```

**2. Network-First (Network Falling Back to Cache):**

```javascript
self.addEventListener('fetch', (event) => {
  event.respondWith(
    fetch(event.request)
      .then((response) => {
        // Cache the response
        return caches.open(CACHE_NAME).then((cache) => {
          cache.put(event.request, response.clone());
          return response;
        });
      })
      .catch(() => {
        // Fallback to cache
        return caches.match(event.request);
      })
  );
});
```

**3. Stale-While-Revalidate:**

```javascript
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then((cachedResponse) => {
      const fetchPromise = fetch(event.request).then((networkResponse) => {
        // Update cache in background
        caches.open(CACHE_NAME).then((cache) => {
          cache.put(event.request, networkResponse.clone());
        });
        return networkResponse;
      });
      
      // Return cached response immediately, update in background
      return cachedResponse || fetchPromise;
    })
  );
});
```

---

#### **IndexedDB for Data Storage**

**1. Setup IndexedDB:**

```typescript
// src/utils/db.ts
import { openDB, DBSchema, IDBPDatabase } from 'idb';

interface MyDB extends DBSchema {
  todos: {
    key: string;
    value: {
      id: string;
      text: string;
      completed: boolean;
      createdAt: number;
      syncStatus: 'synced' | 'pending' | 'error';
    };
    indexes: { 'by-sync-status': 'syncStatus' };
  };
  pendingActions: {
    key: number;
    value: {
      id: number;
      type: 'create' | 'update' | 'delete';
      data: any;
      timestamp: number;
    };
  };
}

let dbInstance: IDBPDatabase<MyDB> | null = null;

export async function getDB(): Promise<IDBPDatabase<MyDB>> {
  if (dbInstance) return dbInstance;

  dbInstance = await openDB<MyDB>('my-app-db', 1, {
    upgrade(db) {
      // Create todos store
      if (!db.objectStoreNames.contains('todos')) {
        const todoStore = db.createObjectStore('todos', { keyPath: 'id' });
        todoStore.createIndex('by-sync-status', 'syncStatus');
      }

      // Create pending actions store
      if (!db.objectStoreNames.contains('pendingActions')) {
        db.createObjectStore('pendingActions', {
          keyPath: 'id',
          autoIncrement: true,
        });
      }
    },
  });

  return dbInstance;
}

// CRUD operations
export async function addTodo(todo: MyDB['todos']['value']) {
  const db = await getDB();
  await db.add('todos', todo);
}

export async function getAllTodos() {
  const db = await getDB();
  return db.getAll('todos');
}

export async function updateTodo(id: string, updates: Partial<MyDB['todos']['value']>) {
  const db = await getDB();
  const todo = await db.get('todos', id);
  if (todo) {
    await db.put('todos', { ...todo, ...updates });
  }
}

export async function deleteTodo(id: string) {
  const db = await getDB();
  await db.delete('todos', id);
}

// Pending actions for sync
export async function addPendingAction(
  type: 'create' | 'update' | 'delete',
  data: any
) {
  const db = await getDB();
  await db.add('pendingActions', {
    type,
    data,
    timestamp: Date.now(),
  } as any);
}

export async function getPendingActions() {
  const db = await getDB();
  return db.getAll('pendingActions');
}

export async function clearPendingAction(id: number) {
  const db = await getDB();
  await db.delete('pendingActions', id);
}
```

---

#### **Offline Detection Hook**

```typescript
// src/hooks/useOnlineStatus.ts
import { useState, useEffect } from 'react';

export function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {
    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);

    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);

    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  return isOnline;
}
```

---

#### **Background Sync**

**1. Register Sync:**

```typescript
// src/utils/sync.ts
export async function registerBackgroundSync(tag: string = 'sync-todos') {
  if ('serviceWorker' in navigator && 'sync' in self.registration) {
    try {
      const registration = await navigator.serviceWorker.ready;
      await registration.sync.register(tag);
      console.log('Background sync registered');
    } catch (error) {
      console.error('Background sync registration failed:', error);
    }
  }
}
```

**2. Handle Sync in Service Worker:**

```javascript
// public/service-worker.js
self.addEventListener('sync', (event) => {
  console.log('[SW] Sync event:', event.tag);
  
  if (event.tag === 'sync-todos') {
    event.waitUntil(syncTodos());
  }
});

async function syncTodos() {
  try {
    // Open IndexedDB
    const db = await openIndexedDB();
    const pendingActions = await db.getAll('pendingActions');

    // Sync each pending action
    for (const action of pendingActions) {
      try {
        const response = await fetch('/api/sync', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(action),
        });

        if (response.ok) {
          // Remove from pending actions
          await db.delete('pendingActions', action.id);
        }
      } catch (error) {
        console.error('Failed to sync action:', error);
      }
    }

    // Notify clients that sync is complete
    const clients = await self.clients.matchAll();
    clients.forEach((client) => {
      client.postMessage({ type: 'SYNC_COMPLETE' });
    });
  } catch (error) {
    console.error('Sync failed:', error);
  }
}

function openIndexedDB() {
  return new Promise((resolve, reject) => {
    const request = indexedDB.open('my-app-db', 1);
    request.onsuccess = () => resolve(request.result);
    request.onerror = () => reject(request.error);
  });
}
```

---

#### **Real-World Example: Offline-First Todo App**

```typescript
// src/components/TodoApp.tsx
import { useState, useEffect } from 'react';
import { useOnlineStatus } from '../hooks/useOnlineStatus';
import {
  getAllTodos,
  addTodo,
  updateTodo,
  deleteTodo,
  addPendingAction,
} from '../utils/db';
import { registerBackgroundSync } from '../utils/sync';

interface Todo {
  id: string;
  text: string;
  completed: boolean;
  createdAt: number;
  syncStatus: 'synced' | 'pending' | 'error';
}

function TodoApp() {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [newTodoText, setNewTodoText] = useState('');
  const isOnline = useOnlineStatus();
  const [isSyncing, setIsSyncing] = useState(false);

  // Load todos from IndexedDB on mount
  useEffect(() => {
    loadTodos();
  }, []);

  // Listen for sync completion messages
  useEffect(() => {
    if ('serviceWorker' in navigator) {
      navigator.serviceWorker.addEventListener('message', (event) => {
        if (event.data.type === 'SYNC_COMPLETE') {
          loadTodos(); // Reload todos after sync
        }
      });
    }
  }, []);

  // Sync when coming online
  useEffect(() => {
    if (isOnline) {
      syncPendingChanges();
    }
  }, [isOnline]);

  async function loadTodos() {
    const loadedTodos = await getAllTodos();
    setTodos(loadedTodos);
  }

  async function handleAddTodo() {
    if (!newTodoText.trim()) return;

    const newTodo: Todo = {
      id: `todo-${Date.now()}`,
      text: newTodoText,
      completed: false,
      createdAt: Date.now(),
      syncStatus: isOnline ? 'pending' : 'pending',
    };

    // Save to IndexedDB immediately
    await addTodo(newTodo);
    setTodos((prev) => [...prev, newTodo]);
    setNewTodoText('');

    // Queue for sync
    await addPendingAction('create', newTodo);

    // Try to sync immediately if online
    if (isOnline) {
      syncPendingChanges();
    } else {
      // Register background sync for when connection is restored
      registerBackgroundSync();
    }
  }

  async function handleToggleTodo(id: string) {
    const todo = todos.find((t) => t.id === id);
    if (!todo) return;

    const updated = { ...todo, completed: !todo.completed, syncStatus: 'pending' as const };

    // Update IndexedDB
    await updateTodo(id, { completed: updated.completed, syncStatus: 'pending' });
    setTodos((prev) => prev.map((t) => (t.id === id ? updated : t)));

    // Queue for sync
    await addPendingAction('update', updated);

    if (isOnline) {
      syncPendingChanges();
    } else {
      registerBackgroundSync();
    }
  }

  async function handleDeleteTodo(id: string) {
    // Delete from IndexedDB
    await deleteTodo(id);
    setTodos((prev) => prev.filter((t) => t.id !== id));

    // Queue for sync
    await addPendingAction('delete', { id });

    if (isOnline) {
      syncPendingChanges();
    } else {
      registerBackgroundSync();
    }
  }

  async function syncPendingChanges() {
    setIsSyncing(true);

    try {
      const { getPendingActions, clearPendingAction } = await import('../utils/db');
      const pendingActions = await getPendingActions();

      for (const action of pendingActions) {
        try {
          const response = await fetch('/api/todos/sync', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(action),
          });

          if (response.ok) {
            // Clear from pending
            await clearPendingAction(action.id);

            // Update sync status
            if (action.type === 'create' || action.type === 'update') {
              await updateTodo(action.data.id, { syncStatus: 'synced' });
            }
          }
        } catch (error) {
          console.error('Failed to sync action:', error);
          // Mark as error
          if (action.data.id) {
            await updateTodo(action.data.id, { syncStatus: 'error' });
          }
        }
      }

      // Reload todos to reflect sync status
      await loadTodos();
    } catch (error) {
      console.error('Sync failed:', error);
    } finally {
      setIsSyncing(false);
    }
  }

  return (
    <div className="todo-app">
      {/* Online/Offline Indicator */}
      <div className={`status-bar ${isOnline ? 'online' : 'offline'}`}>
        {isOnline ? '🟢 Online' : '🔴 Offline'}
        {isSyncing && ' - Syncing...'}
      </div>

      {/* Add Todo Form */}
      <div className="add-todo">
        <input
          type="text"
          value={newTodoText}
          onChange={(e) => setNewTodoText(e.target.value)}
          onKeyDown={(e) => e.key === 'Enter' && handleAddTodo()}
          placeholder="Add a todo..."
        />
        <button onClick={handleAddTodo}>Add</button>
      </div>

      {/* Todo List */}
      <ul className="todo-list">
        {todos.map((todo) => (
          <li key={todo.id} className={`todo-item ${todo.syncStatus}`}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => handleToggleTodo(todo.id)}
            />
            <span className={todo.completed ? 'completed' : ''}>
              {todo.text}
            </span>
            {todo.syncStatus === 'pending' && <span className="badge">⏳ Pending</span>}
            {todo.syncStatus === 'error' && <span className="badge error">❌ Error</span>}
            <button onClick={() => handleDeleteTodo(todo.id)}>Delete</button>
          </li>
        ))}
      </ul>

      {/* Manual Sync Button */}
      {!isOnline && (
        <div className="offline-message">
          <p>You're offline. Changes will sync when connection is restored.</p>
        </div>
      )}

      {isOnline && todos.some((t) => t.syncStatus === 'pending') && (
        <button onClick={syncPendingChanges} disabled={isSyncing}>
          {isSyncing ? 'Syncing...' : 'Sync Now'}
        </button>
      )}
    </div>
  );
}

export default TodoApp;
```

---

#### **PWA Manifest**

```json
// public/manifest.json
{
  "name": "My Offline App",
  "short_name": "OfflineApp",
  "description": "An offline-capable Progressive Web App",
  "start_url": "/",
  "display": "standalone",
  "theme_color": "#000000",
  "background_color": "#ffffff",
  "icons": [
    {
      "src": "logo192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "logo512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}
```

---

#### **Best Practices**

**1. Cache Versioning:**

```javascript
const CACHE_VERSION = 'v2';
const CACHE_NAME = `my-app-${CACHE_VERSION}`;

// Update version when assets change
```

**2. Selective Caching:**

```javascript
// Don't cache everything
const shouldCache = (request) => {
  return (
    request.method === 'GET' &&
    !request.url.includes('/api/auth') && // Don't cache auth
    !request.url.includes('analytics') // Don't cache analytics
  );
};
```

**3. Conflict Resolution:**

```typescript
// Use timestamps or version numbers
interface Todo {
  id: string;
  text: string;
  version: number; // Increment on each update
  lastModified: number;
}

// Server resolves conflicts based on version/timestamp
```

**4. Optimize Cache Size:**

```javascript
const MAX_CACHE_SIZE = 50;

async function trimCache(cacheName, maxItems) {
  const cache = await caches.open(cacheName);
  const keys = await cache.keys();
  
  if (keys.length > maxItems) {
    await cache.delete(keys[0]);
    trimCache(cacheName, maxItems); // Recursive
  }
}
```

---

#### **Testing Offline Functionality**

**1. Chrome DevTools:**
- Open DevTools → Network tab
- Select "Offline" from throttling dropdown
- Test app behavior

**2. Automated Testing:**

```typescript
// src/__tests__/offline.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import TodoApp from '../TodoApp';

// Mock navigator.onLine
Object.defineProperty(navigator, 'onLine', {
  writable: true,
  value: false,
});

test('works offline', async () => {
  render(<TodoApp />);
  
  // Should show offline indicator
  expect(screen.getByText(/offline/i)).toBeInTheDocument();
  
  // Can still add todos
  const input = screen.getByPlaceholderText(/add a todo/i);
  await userEvent.type(input, 'Test todo{enter}');
  
  // Todo should appear
  expect(screen.getByText('Test todo')).toBeInTheDocument();
  
  // Should show pending status
  expect(screen.getByText(/pending/i)).toBeInTheDocument();
});
```

---

#### **Summary**

**Offline Functionality in React:**

1. **Service Workers**: Cache assets and intercept network requests
2. **Cache Strategies**: Cache-first, network-first, stale-while-revalidate
3. **IndexedDB**: Store structured data locally
4. **Background Sync**: Sync data when connection restored
5. **Online Detection**: Monitor network status with `navigator.onLine`
6. **Pending Queue**: Queue actions when offline, sync when online
7. **Conflict Resolution**: Handle conflicts with timestamps/versions
8. **PWA**: Make app installable with manifest.json

**Key Patterns:**
- Optimistic UI updates
- Local-first architecture
- Background sync for reliability
- Cache versioning for updates
- Selective caching for efficiency

Offline functionality transforms web apps into **resilient, always-available experiences** that work regardless of network conditions.

</details>

---

### 235. How do you implement real-time collaboration features?

<details>
<summary>View Answer</summary>

**Real-time collaboration** allows multiple users to work on the same document simultaneously, seeing each other's changes instantly. This requires **conflict resolution algorithms** (Operational Transformation or CRDT), **WebSocket connections**, and **presence awareness**.

---

#### **Core Technologies**

**1. WebSocket/WebRTC** - Real-time bidirectional communication
**2. Operational Transformation (OT)** - Conflict resolution for sequential edits
**3. CRDT (Conflict-free Replicated Data Types)** - Eventually consistent data structures
**4. ShareDB/Yjs** - Libraries for collaborative editing
**5. Presence Awareness** - Show who's online and where they're editing

---

#### **WebSocket Setup**

**1. Client-Side WebSocket Hook:**

```typescript
// src/hooks/useWebSocket.ts
import { useEffect, useRef, useState } from 'react';

interface UseWebSocketOptions {
  url: string;
  onMessage?: (data: any) => void;
  onOpen?: () => void;
  onClose?: () => void;
  onError?: (error: Event) => void;
  reconnect?: boolean;
  reconnectInterval?: number;
}

export function useWebSocket(options: UseWebSocketOptions) {
  const {
    url,
    onMessage,
    onOpen,
    onClose,
    onError,
    reconnect = true,
    reconnectInterval = 3000,
  } = options;

  const [isConnected, setIsConnected] = useState(false);
  const wsRef = useRef<WebSocket | null>(null);
  const reconnectTimeoutRef = useRef<number>();

  useEffect(() => {
    function connect() {
      const ws = new WebSocket(url);
      wsRef.current = ws;

      ws.onopen = () => {
        console.log('[WS] Connected');
        setIsConnected(true);
        onOpen?.();
      };

      ws.onmessage = (event) => {
        try {
          const data = JSON.parse(event.data);
          onMessage?.(data);
        } catch (error) {
          console.error('[WS] Failed to parse message:', error);
        }
      };

      ws.onclose = () => {
        console.log('[WS] Disconnected');
        setIsConnected(false);
        onClose?.();

        // Reconnect
        if (reconnect) {
          reconnectTimeoutRef.current = window.setTimeout(() => {
            console.log('[WS] Reconnecting...');
            connect();
          }, reconnectInterval);
        }
      };

      ws.onerror = (error) => {
        console.error('[WS] Error:', error);
        onError?.(error);
      };
    }

    connect();

    return () => {
      if (reconnectTimeoutRef.current) {
        clearTimeout(reconnectTimeoutRef.current);
      }
      if (wsRef.current) {
        wsRef.current.close();
      }
    };
  }, [url, onMessage, onOpen, onClose, onError, reconnect, reconnectInterval]);

  const send = (data: any) => {
    if (wsRef.current?.readyState === WebSocket.OPEN) {
      wsRef.current.send(JSON.stringify(data));
    } else {
      console.warn('[WS] Not connected, cannot send message');
    }
  };

  return { isConnected, send };
}
```

---

#### **Operational Transformation (OT)**

**Basic OT for Text Editing:**

```typescript
// src/utils/ot.ts

// Operation types
type Operation =
  | { type: 'insert'; position: number; text: string }
  | { type: 'delete'; position: number; length: number }
  | { type: 'retain'; length: number };

export class OTEngine {
  // Transform operation A against operation B
  // Returns transformed A that can be applied after B
  static transform(opA: Operation, opB: Operation): Operation {
    if (opA.type === 'insert' && opB.type === 'insert') {
      if (opA.position <= opB.position) {
        return opA;
      } else {
        return {
          ...opA,
          position: opA.position + opB.text.length,
        };
      }
    }

    if (opA.type === 'insert' && opB.type === 'delete') {
      if (opA.position <= opB.position) {
        return opA;
      } else if (opA.position > opB.position + opB.length) {
        return {
          ...opA,
          position: opA.position - opB.length,
        };
      } else {
        return {
          ...opA,
          position: opB.position,
        };
      }
    }

    if (opA.type === 'delete' && opB.type === 'insert') {
      if (opA.position < opB.position) {
        return opA;
      } else {
        return {
          ...opA,
          position: opA.position + opB.text.length,
        };
      }
    }

    if (opA.type === 'delete' && opB.type === 'delete') {
      if (opA.position + opA.length <= opB.position) {
        return opA;
      } else if (opA.position >= opB.position + opB.length) {
        return {
          ...opA,
          position: opA.position - opB.length,
        };
      } else {
        // Overlapping deletes - complex case
        const newLength = Math.max(0, opA.length - opB.length);
        return {
          type: 'delete',
          position: Math.min(opA.position, opB.position),
          length: newLength,
        };
      }
    }

    return opA;
  }

  // Apply operation to text
  static apply(text: string, operation: Operation): string {
    switch (operation.type) {
      case 'insert':
        return (
          text.slice(0, operation.position) +
          operation.text +
          text.slice(operation.position)
        );
      case 'delete':
        return (
          text.slice(0, operation.position) +
          text.slice(operation.position + operation.length)
        );
      case 'retain':
        return text;
      default:
        return text;
    }
  }
}
```

---

#### **Real-World Example: Collaborative Text Editor**

```typescript
// src/components/CollaborativeEditor.tsx
import { useState, useEffect, useRef, useCallback } from 'react';
import { useWebSocket } from '../hooks/useWebSocket';
import { OTEngine } from '../utils/ot';

interface User {
  id: string;
  name: string;
  color: string;
  cursor?: number;
}

interface Message {
  type: 'operation' | 'cursor' | 'presence';
  userId: string;
  data: any;
}

function CollaborativeEditor({ documentId, userId }: Props) {
  const [text, setText] = useState('');
  const [users, setUsers] = useState<User[]>([]);
  const [pendingOps, setPendingOps] = useState<any[]>([]);
  const textareaRef = useRef<HTMLTextAreaElement>(null);
  const versionRef = useRef(0);

  const { isConnected, send } = useWebSocket({
    url: `ws://localhost:3000/collab/${documentId}`,
    onMessage: handleMessage,
    onOpen: () => {
      // Send join message
      send({
        type: 'join',
        userId,
        documentId,
      });
    },
  });

  function handleMessage(message: Message) {
    switch (message.type) {
      case 'operation':
        handleRemoteOperation(message.data);
        break;
      case 'cursor':
        updateUserCursor(message.userId, message.data.cursor);
        break;
      case 'presence':
        setUsers(message.data.users);
        break;
      case 'init':
        setText(message.data.content);
        versionRef.current = message.data.version;
        break;
    }
  }

  function handleRemoteOperation(operation: any) {
    // Transform against pending operations
    let transformedOp = operation;
    for (const pendingOp of pendingOps) {
      transformedOp = OTEngine.transform(transformedOp, pendingOp);
    }

    // Apply to text
    setText((currentText) => OTEngine.apply(currentText, transformedOp));
    versionRef.current++;
  }

  function handleLocalChange(e: React.ChangeEvent<HTMLTextAreaElement>) {
    const newText = e.target.value;
    const selectionStart = e.target.selectionStart;

    // Calculate operation
    const operation = calculateOperation(text, newText, selectionStart);

    if (operation) {
      // Apply locally
      setText(newText);

      // Add to pending operations
      setPendingOps((ops) => [...ops, operation]);

      // Send to server
      send({
        type: 'operation',
        userId,
        documentId,
        operation,
        version: versionRef.current,
      });

      versionRef.current++;
    }
  }

  function calculateOperation(
    oldText: string,
    newText: string,
    cursorPosition: number
  ) {
    if (newText.length > oldText.length) {
      // Insert
      const insertedText = newText.slice(
        cursorPosition - (newText.length - oldText.length),
        cursorPosition
      );
      return {
        type: 'insert' as const,
        position: cursorPosition - insertedText.length,
        text: insertedText,
      };
    } else if (newText.length < oldText.length) {
      // Delete
      return {
        type: 'delete' as const,
        position: cursorPosition,
        length: oldText.length - newText.length,
      };
    }
    return null;
  }

  function handleCursorChange() {
    if (!textareaRef.current) return;

    const cursor = textareaRef.current.selectionStart;

    // Send cursor position
    send({
      type: 'cursor',
      userId,
      documentId,
      cursor,
    });
  }

  function updateUserCursor(userId: string, cursor: number) {
    setUsers((prevUsers) =>
      prevUsers.map((user) =>
        user.id === userId ? { ...user, cursor } : user
      )
    );
  }

  // Clear pending operation when acknowledged
  useEffect(() => {
    const listener = (message: any) => {
      if (message.type === 'ack' && message.userId === userId) {
        setPendingOps((ops) => ops.slice(1));
      }
    };

    // Add listener logic here
  }, [userId]);

  return (
    <div className="collaborative-editor">
      {/* Connection Status */}
      <div className="status-bar">
        <span className={isConnected ? 'online' : 'offline'}>
          {isConnected ? '🟢 Connected' : '🔴 Disconnected'}
        </span>
        {pendingOps.length > 0 && (
          <span className="syncing">⏳ Syncing {pendingOps.length} changes...</span>
        )}
      </div>

      {/* Active Users */}
      <div className="users">
        {users.map((user) => (
          <div key={user.id} className="user">
            <div
              className="user-avatar"
              style={{ backgroundColor: user.color }}
            >
              {user.name[0]}
            </div>
            <span>{user.name}</span>
          </div>
        ))}
      </div>

      {/* Editor */}
      <div className="editor-container">
        <textarea
          ref={textareaRef}
          value={text}
          onChange={handleLocalChange}
          onSelect={handleCursorChange}
          className="editor"
          placeholder="Start typing..."
        />

        {/* Cursor overlays */}
        {users
          .filter((user) => user.id !== userId && user.cursor !== undefined)
          .map((user) => (
            <div
              key={user.id}
              className="remote-cursor"
              style={{
                left: calculateCursorPosition(user.cursor),
                borderColor: user.color,
              }}
            >
              <div
                className="cursor-label"
                style={{ backgroundColor: user.color }}
              >
                {user.name}
              </div>
            </div>
          ))}
      </div>
    </div>
  );
}

function calculateCursorPosition(cursor: number): number {
  // Calculate pixel position based on cursor index
  // This is simplified - real implementation would be more complex
  return cursor * 8; // Approximate character width
}

export default CollaborativeEditor;
```

---

#### **CRDT-Based Collaboration (Yjs)**

**More Modern Approach:**

```typescript
// src/components/YjsEditor.tsx
import { useEffect, useState } from 'react';
import * as Y from 'yjs';
import { WebsocketProvider } from 'y-websocket';
import { Awareness } from 'y-protocols/awareness';

function YjsEditor({ documentId, userId, userName }: Props) {
  const [doc] = useState(() => new Y.Doc());
  const [provider, setProvider] = useState<WebsocketProvider | null>(null);
  const [text, setText] = useState('');
  const [users, setUsers] = useState<Map<number, any>>(new Map());

  useEffect(() => {
    // Create WebSocket provider
    const wsProvider = new WebsocketProvider(
      'ws://localhost:3000',
      documentId,
      doc
    );

    setProvider(wsProvider);

    // Get shared text type
    const yText = doc.getText('content');

    // Set user info in awareness
    wsProvider.awareness.setLocalStateField('user', {
      id: userId,
      name: userName,
      color: generateColor(userId),
    });

    // Listen to text changes
    const updateHandler = () => {
      setText(yText.toString());
    };

    yText.observe(updateHandler);

    // Listen to awareness changes (presence)
    const awarenessHandler = () => {
      setUsers(new Map(wsProvider.awareness.getStates()));
    };

    wsProvider.awareness.on('change', awarenessHandler);

    return () => {
      yText.unobserve(updateHandler);
      wsProvider.awareness.off('change', awarenessHandler);
      wsProvider.destroy();
    };
  }, [doc, documentId, userId, userName]);

  const handleChange = (e: React.ChangeEvent<HTMLTextAreaElement>) => {
    const newValue = e.target.value;
    const yText = doc.getText('content');

    // Calculate diff and apply to Yjs
    doc.transact(() => {
      const currentText = yText.toString();
      
      // Simple approach: delete all and insert new
      // Real implementation would calculate minimal diff
      yText.delete(0, currentText.length);
      yText.insert(0, newValue);
    });
  };

  const updateCursor = (position: number) => {
    provider?.awareness.setLocalStateField('cursor', position);
  };

  return (
    <div className="yjs-editor">
      <div className="users">
        {Array.from(users.entries())
          .filter(([id]) => id !== provider?.awareness.clientID)
          .map(([id, state]) => (
            <div key={id} className="user">
              <div
                className="avatar"
                style={{ backgroundColor: state.user?.color }}
              >
                {state.user?.name?.[0]}
              </div>
              <span>{state.user?.name}</span>
            </div>
          ))}
      </div>

      <textarea
        value={text}
        onChange={handleChange}
        onSelect={(e) => updateCursor(e.currentTarget.selectionStart)}
        className="editor"
      />
    </div>
  );
}

function generateColor(userId: string): string {
  const colors = [
    '#FF6B6B',
    '#4ECDC4',
    '#45B7D1',
    '#FFA07A',
    '#98D8C8',
    '#6C5CE7',
  ];
  const index = userId.split('').reduce((acc, char) => acc + char.charCodeAt(0), 0);
  return colors[index % colors.length];
}

export default YjsEditor;
```

---

#### **Presence Awareness**

```typescript
// src/components/PresenceIndicator.tsx
import { useEffect, useState } from 'react';

interface Presence {
  userId: string;
  name: string;
  avatar: string;
  color: string;
  lastSeen: number;
  isTyping: boolean;
  cursor?: { line: number; column: number };
}

function PresenceIndicator({ documentId }: { documentId: string }) {
  const [presences, setPresences] = useState<Presence[]>([]);

  useEffect(() => {
    const ws = new WebSocket(`ws://localhost:3000/presence/${documentId}`);

    ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      
      if (data.type === 'presence-update') {
        setPresences(data.presences);
      }
    };

    // Send heartbeat every 5 seconds
    const heartbeat = setInterval(() => {
      ws.send(JSON.stringify({ type: 'heartbeat' }));
    }, 5000);

    return () => {
      clearInterval(heartbeat);
      ws.close();
    };
  }, [documentId]);

  return (
    <div className="presence-indicator">
      <div className="active-users">
        {presences.map((presence) => (
          <div
            key={presence.userId}
            className="user-badge"
            style={{ borderColor: presence.color }}
          >
            <img src={presence.avatar} alt={presence.name} />
            {presence.isTyping && <span className="typing-indicator">✍️</span>}
          </div>
        ))}
      </div>
      <span className="count">
        {presences.length} {presences.length === 1 ? 'user' : 'users'} online
      </span>
    </div>
  );
}

export default PresenceIndicator;
```

---

#### **Conflict Resolution Strategies**

**1. Last Write Wins:**

```typescript
interface Document {
  content: string;
  version: number;
  lastModified: number;
}

function resolveConflict(local: Document, remote: Document): Document {
  // Use timestamp to determine winner
  return local.lastModified > remote.lastModified ? local : remote;
}
```

**2. Version Vectors:**

```typescript
interface VersionVector {
  [userId: string]: number;
}

function compareVersionVectors(
  v1: VersionVector,
  v2: VersionVector
): 'before' | 'after' | 'concurrent' {
  let v1Greater = false;
  let v2Greater = false;

  const allKeys = new Set([...Object.keys(v1), ...Object.keys(v2)]);

  for (const key of allKeys) {
    const v1Val = v1[key] || 0;
    const v2Val = v2[key] || 0;

    if (v1Val > v2Val) v1Greater = true;
    if (v2Val > v1Val) v2Greater = true;
  }

  if (v1Greater && !v2Greater) return 'after';
  if (v2Greater && !v1Greater) return 'before';
  return 'concurrent';
}
```

---

#### **Optimistic Updates**

```typescript
function useOptimisticCollaboration() {
  const [localState, setLocalState] = useState<any>(null);
  const [serverState, setServerState] = useState<any>(null);
  const [pendingChanges, setPendingChanges] = useState<any[]>([]);

  const applyChange = (change: any) => {
    // Apply optimistically
    setLocalState((current) => applyOperation(current, change));

    // Queue for server
    setPendingChanges((pending) => [...pending, change]);

    // Send to server
    sendToServer(change)
      .then((response) => {
        // Remove from pending
        setPendingChanges((pending) => pending.filter((c) => c.id !== change.id));
        
        // Update server state
        setServerState(response.state);
      })
      .catch((error) => {
        // Revert optimistic change
        setLocalState(serverState);
        setPendingChanges((pending) => pending.filter((c) => c.id !== change.id));
      });
  };

  return { localState, applyChange, hasPendingChanges: pendingChanges.length > 0 };
}
```

---

#### **Best Practices**

**1. Use Established Libraries:**
- **Yjs**: CRDT-based, great for rich text
- **Automerge**: Pure CRDT implementation
- **ShareDB**: OT-based, proven at scale

**2. Implement Presence:**
- Show active users
- Display cursors/selections
- Show typing indicators

**3. Handle Network Issues:**
- Reconnection logic
- Offline support
- Conflict resolution

**4. Optimize Performance:**
- Debounce rapid changes
- Send minimal diffs
- Use binary protocols for large documents

**5. Security:**
- Authenticate WebSocket connections
- Validate all operations
- Implement access control

---

#### **Summary**

**Real-Time Collaboration:**

1. **WebSocket**: Bidirectional real-time communication
2. **OT (Operational Transformation)**: Transform conflicting operations
3. **CRDT**: Conflict-free data structures (Yjs, Automerge)
4. **Presence**: Show active users and their cursors
5. **Conflict Resolution**: Last-write-wins, version vectors, 3-way merge
6. **Optimistic Updates**: Apply changes immediately, rollback on failure
7. **Libraries**: Use Yjs for CRDT or ShareDB for OT

**Key Components:**
- WebSocket connection management
- Operation transformation engine
- Presence/awareness system
- Cursor tracking and display
- Conflict resolution strategy
- Offline support with sync

**Approaches:**
- **OT**: Google Docs-style, sequential operations
- **CRDT**: Figma-style, commutative operations

Real-time collaboration enables **Google Docs-like experiences** where multiple users can edit simultaneously without conflicts.

</details>

---

### 236. How do you handle large file uploads with progress?

<details>
<summary>View Answer</summary>

**Large file uploads** require special handling to show progress, support cancellation, enable resumption on failure, and avoid timeouts. Key techniques include **chunked uploads**, **progress tracking**, and **resumable uploads**.

---

#### **Basic File Upload with Progress**

**Using XMLHttpRequest (Native Progress Support):**

```typescript
// src/hooks/useFileUpload.ts
import { useState, useCallback } from 'react';

interface UploadState {
  progress: number;
  isUploading: boolean;
  error: string | null;
  uploadedUrl: string | null;
}

export function useFileUpload() {
  const [state, setState] = useState<UploadState>({
    progress: 0,
    isUploading: false,
    error: null,
    uploadedUrl: null,
  });

  const [abortController, setAbortController] = useState<AbortController | null>(null);

  const upload = useCallback(async (file: File, url: string) => {
    setState({
      progress: 0,
      isUploading: true,
      error: null,
      uploadedUrl: null,
    });

    return new Promise<string>((resolve, reject) => {
      const xhr = new XMLHttpRequest();
      const controller = new AbortController();
      setAbortController(controller);

      // Track upload progress
      xhr.upload.addEventListener('progress', (e) => {
        if (e.lengthComputable) {
          const percentComplete = (e.loaded / e.total) * 100;
          setState((prev) => ({ ...prev, progress: percentComplete }));
        }
      });

      // Handle completion
      xhr.addEventListener('load', () => {
        if (xhr.status >= 200 && xhr.status < 300) {
          const response = JSON.parse(xhr.responseText);
          setState({
            progress: 100,
            isUploading: false,
            error: null,
            uploadedUrl: response.url,
          });
          resolve(response.url);
        } else {
          const error = `Upload failed: ${xhr.statusText}`;
          setState({
            progress: 0,
            isUploading: false,
            error,
            uploadedUrl: null,
          });
          reject(new Error(error));
        }
      });

      // Handle errors
      xhr.addEventListener('error', () => {
        const error = 'Upload failed: Network error';
        setState({
          progress: 0,
          isUploading: false,
          error,
          uploadedUrl: null,
        });
        reject(new Error(error));
      });

      // Handle abort
      xhr.addEventListener('abort', () => {
        setState({
          progress: 0,
          isUploading: false,
          error: 'Upload cancelled',
          uploadedUrl: null,
        });
        reject(new Error('Upload cancelled'));
      });

      // Listen to abort signal
      controller.signal.addEventListener('abort', () => {
        xhr.abort();
      });

      // Send request
      xhr.open('POST', url);
      
      const formData = new FormData();
      formData.append('file', file);
      
      xhr.send(formData);
    });
  }, []);

  const cancel = useCallback(() => {
    if (abortController) {
      abortController.abort();
      setAbortController(null);
    }
  }, [abortController]);

  return { ...state, upload, cancel };
}
```

**Usage:**

```typescript
// src/components/FileUploader.tsx
import { useFileUpload } from '../hooks/useFileUpload';

function FileUploader() {
  const { progress, isUploading, error, uploadedUrl, upload, cancel } = useFileUpload();

  const handleFileChange = async (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (!file) return;

    try {
      const url = await upload(file, '/api/upload');
      console.log('Upload complete:', url);
    } catch (error) {
      console.error('Upload failed:', error);
    }
  };

  return (
    <div className="file-uploader">
      <input
        type="file"
        onChange={handleFileChange}
        disabled={isUploading}
      />

      {isUploading && (
        <div className="progress-container">
          <div className="progress-bar">
            <div
              className="progress-fill"
              style={{ width: `${progress}%` }}
            />
          </div>
          <span>{progress.toFixed(1)}%</span>
          <button onClick={cancel}>Cancel</button>
        </div>
      )}

      {error && <div className="error">{error}</div>}
      {uploadedUrl && <div className="success">Upload complete: {uploadedUrl}</div>}
    </div>
  );
}

export default FileUploader;
```

---

#### **Chunked Upload (For Very Large Files)**

**Split file into chunks and upload sequentially:**

```typescript
// src/utils/chunkedUpload.ts

interface ChunkUploadOptions {
  file: File;
  uploadUrl: string;
  chunkSize?: number; // Default 5MB
  onProgress?: (progress: number) => void;
  onChunkComplete?: (chunkIndex: number, totalChunks: number) => void;
}

export async function uploadFileInChunks(options: ChunkUploadOptions): Promise<string> {
  const {
    file,
    uploadUrl,
    chunkSize = 5 * 1024 * 1024, // 5MB chunks
    onProgress,
    onChunkComplete,
  } = options;

  const totalChunks = Math.ceil(file.size / chunkSize);
  const uploadId = `upload-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;

  for (let chunkIndex = 0; chunkIndex < totalChunks; chunkIndex++) {
    const start = chunkIndex * chunkSize;
    const end = Math.min(start + chunkSize, file.size);
    const chunk = file.slice(start, end);

    const formData = new FormData();
    formData.append('chunk', chunk);
    formData.append('chunkIndex', chunkIndex.toString());
    formData.append('totalChunks', totalChunks.toString());
    formData.append('uploadId', uploadId);
    formData.append('filename', file.name);

    const response = await fetch(uploadUrl, {
      method: 'POST',
      body: formData,
    });

    if (!response.ok) {
      throw new Error(`Chunk ${chunkIndex + 1} upload failed`);
    }

    // Update progress
    const progress = ((chunkIndex + 1) / totalChunks) * 100;
    onProgress?.(progress);
    onChunkComplete?.(chunkIndex, totalChunks);
  }

  // Finalize upload
  const finalizeResponse = await fetch(`${uploadUrl}/finalize`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ uploadId, filename: file.name }),
  });

  const result = await finalizeResponse.json();
  return result.url;
}
```

**Hook for Chunked Upload:**

```typescript
// src/hooks/useChunkedUpload.ts
import { useState, useCallback } from 'react';
import { uploadFileInChunks } from '../utils/chunkedUpload';

export function useChunkedUpload() {
  const [progress, setProgress] = useState(0);
  const [isUploading, setIsUploading] = useState(false);
  const [currentChunk, setCurrentChunk] = useState(0);
  const [totalChunks, setTotalChunks] = useState(0);
  const [error, setError] = useState<string | null>(null);

  const upload = useCallback(async (file: File, url: string) => {
    setIsUploading(true);
    setProgress(0);
    setError(null);

    try {
      const uploadedUrl = await uploadFileInChunks({
        file,
        uploadUrl: url,
        chunkSize: 5 * 1024 * 1024, // 5MB chunks
        onProgress: setProgress,
        onChunkComplete: (chunkIndex, total) => {
          setCurrentChunk(chunkIndex + 1);
          setTotalChunks(total);
        },
      });

      setIsUploading(false);
      return uploadedUrl;
    } catch (err) {
      const errorMessage = err instanceof Error ? err.message : 'Upload failed';
      setError(errorMessage);
      setIsUploading(false);
      throw err;
    }
  }, []);

  return {
    progress,
    isUploading,
    currentChunk,
    totalChunks,
    error,
    upload,
  };
}
```

---

#### **Resumable Upload**

**Upload with ability to resume after network failure:**

```typescript
// src/utils/resumableUpload.ts

interface ResumableUploadState {
  uploadId: string;
  uploadedChunks: number[];
  totalChunks: number;
}

export class ResumableUpload {
  private state: ResumableUploadState | null = null;
  private storageKey: string;

  constructor(private file: File) {
    this.storageKey = `upload-state-${file.name}-${file.size}`;
    this.loadState();
  }

  private loadState() {
    const saved = localStorage.getItem(this.storageKey);
    if (saved) {
      this.state = JSON.parse(saved);
    }
  }

  private saveState() {
    if (this.state) {
      localStorage.setItem(this.storageKey, JSON.stringify(this.state));
    }
  }

  async upload(
    uploadUrl: string,
    chunkSize: number = 5 * 1024 * 1024,
    onProgress?: (progress: number) => void
  ): Promise<string> {
    const totalChunks = Math.ceil(this.file.size / chunkSize);

    // Initialize state if new upload
    if (!this.state) {
      this.state = {
        uploadId: `upload-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
        uploadedChunks: [],
        totalChunks,
      };
      this.saveState();
    }

    // Upload remaining chunks
    for (let chunkIndex = 0; chunkIndex < totalChunks; chunkIndex++) {
      // Skip already uploaded chunks
      if (this.state.uploadedChunks.includes(chunkIndex)) {
        continue;
      }

      const start = chunkIndex * chunkSize;
      const end = Math.min(start + chunkSize, this.file.size);
      const chunk = this.file.slice(start, end);

      const formData = new FormData();
      formData.append('chunk', chunk);
      formData.append('chunkIndex', chunkIndex.toString());
      formData.append('totalChunks', totalChunks.toString());
      formData.append('uploadId', this.state.uploadId);
      formData.append('filename', this.file.name);

      try {
        const response = await fetch(uploadUrl, {
          method: 'POST',
          body: formData,
        });

        if (!response.ok) {
          throw new Error(`Chunk ${chunkIndex + 1} upload failed`);
        }

        // Mark chunk as uploaded
        this.state.uploadedChunks.push(chunkIndex);
        this.saveState();

        // Update progress
        const progress = (this.state.uploadedChunks.length / totalChunks) * 100;
        onProgress?.(progress);
      } catch (error) {
        console.error(`Failed to upload chunk ${chunkIndex}:`, error);
        throw error; // Can be retried later
      }
    }

    // Finalize upload
    const finalizeResponse = await fetch(`${uploadUrl}/finalize`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        uploadId: this.state.uploadId,
        filename: this.file.name,
      }),
    });

    const result = await finalizeResponse.json();

    // Clear state after successful upload
    localStorage.removeItem(this.storageKey);
    this.state = null;

    return result.url;
  }

  canResume(): boolean {
    return this.state !== null && this.state.uploadedChunks.length > 0;
  }

  getProgress(): number {
    if (!this.state) return 0;
    return (this.state.uploadedChunks.length / this.state.totalChunks) * 100;
  }
}
```

---

#### **Drag and Drop Upload**

```typescript
// src/components/DragDropUploader.tsx
import { useState, useRef } from 'react';
import { useFileUpload } from '../hooks/useFileUpload';

function DragDropUploader() {
  const [isDragging, setIsDragging] = useState(false);
  const { progress, isUploading, error, upload } = useFileUpload();
  const dragCounter = useRef(0);

  const handleDragEnter = (e: React.DragEvent) => {
    e.preventDefault();
    e.stopPropagation();
    dragCounter.current++;
    if (e.dataTransfer.items && e.dataTransfer.items.length > 0) {
      setIsDragging(true);
    }
  };

  const handleDragLeave = (e: React.DragEvent) => {
    e.preventDefault();
    e.stopPropagation();
    dragCounter.current--;
    if (dragCounter.current === 0) {
      setIsDragging(false);
    }
  };

  const handleDragOver = (e: React.DragEvent) => {
    e.preventDefault();
    e.stopPropagation();
  };

  const handleDrop = async (e: React.DragEvent) => {
    e.preventDefault();
    e.stopPropagation();
    setIsDragging(false);
    dragCounter.current = 0;

    const files = Array.from(e.dataTransfer.files);
    if (files.length > 0) {
      const file = files[0];
      try {
        await upload(file, '/api/upload');
      } catch (error) {
        console.error('Upload failed:', error);
      }
    }
  };

  return (
    <div
      className={`drag-drop-zone ${isDragging ? 'dragging' : ''}`}
      onDragEnter={handleDragEnter}
      onDragLeave={handleDragLeave}
      onDragOver={handleDragOver}
      onDrop={handleDrop}
    >
      {isUploading ? (
        <div className="uploading">
          <div className="progress-circle">
            <svg viewBox="0 0 100 100">
              <circle
                cx="50"
                cy="50"
                r="45"
                fill="none"
                stroke="#e0e0e0"
                strokeWidth="8"
              />
              <circle
                cx="50"
                cy="50"
                r="45"
                fill="none"
                stroke="#4CAF50"
                strokeWidth="8"
                strokeDasharray={`${progress * 2.827} 283`}
                strokeDashoffset="0"
                transform="rotate(-90 50 50)"
              />
            </svg>
            <span className="progress-text">{Math.round(progress)}%</span>
          </div>
        </div>
      ) : (
        <div className="drop-message">
          <svg className="upload-icon" viewBox="0 0 24 24">
            <path d="M9 16h6v-6h4l-7-7-7 7h4zm-4 2h14v2H5z" />
          </svg>
          <p>Drag and drop files here</p>
          <p className="or">or</p>
          <label className="browse-button">
            Browse Files
            <input type="file" style={{ display: 'none' }} />
          </label>
        </div>
      )}

      {error && <div className="error">{error}</div>}
    </div>
  );
}

export default DragDropUploader;
```

---

#### **Multiple File Upload**

```typescript
// src/components/MultiFileUploader.tsx
import { useState } from 'react';

interface FileUploadState {
  file: File;
  progress: number;
  status: 'pending' | 'uploading' | 'complete' | 'error';
  error?: string;
  url?: string;
}

function MultiFileUploader() {
  const [files, setFiles] = useState<FileUploadState[]>([]);

  const handleFileSelect = (e: React.ChangeEvent<HTMLInputElement>) => {
    const selectedFiles = Array.from(e.target.files || []);
    
    const newFiles: FileUploadState[] = selectedFiles.map((file) => ({
      file,
      progress: 0,
      status: 'pending',
    }));

    setFiles((prev) => [...prev, ...newFiles]);
  };

  const uploadFile = async (index: number) => {
    const fileState = files[index];
    
    // Update status to uploading
    setFiles((prev) =>
      prev.map((f, i) => (i === index ? { ...f, status: 'uploading' as const } : f))
    );

    const xhr = new XMLHttpRequest();

    xhr.upload.addEventListener('progress', (e) => {
      if (e.lengthComputable) {
        const progress = (e.loaded / e.total) * 100;
        setFiles((prev) =>
          prev.map((f, i) => (i === index ? { ...f, progress } : f))
        );
      }
    });

    xhr.addEventListener('load', () => {
      if (xhr.status >= 200 && xhr.status < 300) {
        const response = JSON.parse(xhr.responseText);
        setFiles((prev) =>
          prev.map((f, i) =>
            i === index
              ? { ...f, status: 'complete' as const, url: response.url }
              : f
          )
        );
      } else {
        setFiles((prev) =>
          prev.map((f, i) =>
            i === index
              ? { ...f, status: 'error' as const, error: 'Upload failed' }
              : f
          )
        );
      }
    });

    xhr.open('POST', '/api/upload');
    
    const formData = new FormData();
    formData.append('file', fileState.file);
    
    xhr.send(formData);
  };

  const uploadAll = () => {
    files.forEach((file, index) => {
      if (file.status === 'pending' || file.status === 'error') {
        uploadFile(index);
      }
    });
  };

  const removeFile = (index: number) => {
    setFiles((prev) => prev.filter((_, i) => i !== index));
  };

  return (
    <div className="multi-file-uploader">
      <div className="upload-controls">
        <label className="file-input">
          Select Files
          <input type="file" multiple onChange={handleFileSelect} />
        </label>
        <button onClick={uploadAll} disabled={files.length === 0}>
          Upload All
        </button>
      </div>

      <div className="file-list">
        {files.map((fileState, index) => (
          <div key={index} className={`file-item ${fileState.status}`}>
            <div className="file-info">
              <span className="file-name">{fileState.file.name}</span>
              <span className="file-size">
                {(fileState.file.size / 1024 / 1024).toFixed(2)} MB
              </span>
            </div>

            {fileState.status === 'uploading' && (
              <div className="progress-bar">
                <div
                  className="progress-fill"
                  style={{ width: `${fileState.progress}%` }}
                />
              </div>
            )}

            {fileState.status === 'pending' && (
              <button onClick={() => uploadFile(index)}>Upload</button>
            )}

            {fileState.status === 'complete' && (
              <span className="status-icon">✓</span>
            )}

            {fileState.status === 'error' && (
              <div className="error">
                <span className="status-icon">✗</span>
                <button onClick={() => uploadFile(index)}>Retry</button>
              </div>
            )}

            <button onClick={() => removeFile(index)} className="remove">
              ×
            </button>
          </div>
        ))}
      </div>
    </div>
  );
}

export default MultiFileUploader;
```

---

#### **Real-World Example: Cloud Storage Uploader**

```typescript
// src/components/CloudStorageUploader.tsx
import { useState } from 'react';
import { ResumableUpload } from '../utils/resumableUpload';

interface UploadJob {
  id: string;
  file: File;
  progress: number;
  status: 'pending' | 'uploading' | 'paused' | 'complete' | 'error';
  resumable: ResumableUpload;
  error?: string;
}

function CloudStorageUploader() {
  const [uploads, setUploads] = useState<UploadJob[]>([]);

  const addFiles = (files: File[]) => {
    const newUploads: UploadJob[] = files.map((file) => ({
      id: `upload-${Date.now()}-${Math.random()}`,
      file,
      progress: 0,
      status: 'pending',
      resumable: new ResumableUpload(file),
    }));

    setUploads((prev) => [...prev, ...newUploads]);

    // Auto-start uploads
    newUploads.forEach((upload) => startUpload(upload.id));
  };

  const startUpload = async (uploadId: string) => {
    const upload = uploads.find((u) => u.id === uploadId);
    if (!upload) return;

    setUploads((prev) =>
      prev.map((u) => (u.id === uploadId ? { ...u, status: 'uploading' as const } : u))
    );

    try {
      await upload.resumable.upload(
        '/api/upload/chunks',
        5 * 1024 * 1024, // 5MB chunks
        (progress) => {
          setUploads((prev) =>
            prev.map((u) => (u.id === uploadId ? { ...u, progress } : u))
          );
        }
      );

      setUploads((prev) =>
        prev.map((u) =>
          u.id === uploadId ? { ...u, status: 'complete' as const, progress: 100 } : u
        )
      );
    } catch (error) {
      setUploads((prev) =>
        prev.map((u) =>
          u.id === uploadId
            ? {
                ...u,
                status: 'error' as const,
                error: error instanceof Error ? error.message : 'Upload failed',
              }
            : u
        )
      );
    }
  };

  const pauseUpload = (uploadId: string) => {
    // In real implementation, would abort the current request
    setUploads((prev) =>
      prev.map((u) => (u.id === uploadId ? { ...u, status: 'paused' as const } : u))
    );
  };

  const resumeUpload = (uploadId: string) => {
    startUpload(uploadId);
  };

  const cancelUpload = (uploadId: string) => {
    setUploads((prev) => prev.filter((u) => u.id !== uploadId));
  };

  return (
    <div className="cloud-storage-uploader">
      <div className="upload-zone"
        onDrop={(e) => {
          e.preventDefault();
          const files = Array.from(e.dataTransfer.files);
          addFiles(files);
        }}
        onDragOver={(e) => e.preventDefault()}
      >
        <p>Drop files here or click to browse</p>
        <input
          type="file"
          multiple
          onChange={(e) => {
            const files = Array.from(e.target.files || []);
            addFiles(files);
          }}
        />
      </div>

      <div className="uploads-list">
        {uploads.map((upload) => (
          <div key={upload.id} className={`upload-item ${upload.status}`}>
            <div className="file-icon">📄</div>
            
            <div className="file-details">
              <div className="file-name">{upload.file.name}</div>
              <div className="file-size">
                {(upload.file.size / 1024 / 1024).toFixed(2)} MB
              </div>
              
              {upload.status === 'uploading' && (
                <div className="progress">
                  <div className="progress-bar">
                    <div
                      className="progress-fill"
                      style={{ width: `${upload.progress}%` }}
                    />
                  </div>
                  <span>{upload.progress.toFixed(0)}%</span>
                </div>
              )}

              {upload.status === 'complete' && (
                <div className="success">✓ Upload complete</div>
              )}

              {upload.status === 'error' && (
                <div className="error">{upload.error}</div>
              )}
            </div>

            <div className="upload-actions">
              {upload.status === 'uploading' && (
                <button onClick={() => pauseUpload(upload.id)}>Pause</button>
              )}
              
              {upload.status === 'paused' && (
                <button onClick={() => resumeUpload(upload.id)}>Resume</button>
              )}
              
              {(upload.status === 'error' || upload.status === 'paused') && (
                <button onClick={() => startUpload(upload.id)}>Retry</button>
              )}
              
              <button onClick={() => cancelUpload(upload.id)}>×</button>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}

export default CloudStorageUploader;
```

---

#### **Best Practices**

**1. File Validation:**

```typescript
function validateFile(file: File): string | null {
  const maxSize = 100 * 1024 * 1024; // 100MB
  const allowedTypes = ['image/jpeg', 'image/png', 'application/pdf'];

  if (file.size > maxSize) {
    return 'File too large (max 100MB)';
  }

  if (!allowedTypes.includes(file.type)) {
    return 'Invalid file type';
  }

  return null;
}
```

**2. Chunk Size Selection:**

```typescript
function getOptimalChunkSize(fileSize: number): number {
  if (fileSize < 10 * 1024 * 1024) return 1 * 1024 * 1024; // 1MB
  if (fileSize < 100 * 1024 * 1024) return 5 * 1024 * 1024; // 5MB
  return 10 * 1024 * 1024; // 10MB
}
```

**3. Retry Logic:**

```typescript
async function uploadWithRetry(
  uploadFn: () => Promise<void>,
  maxRetries: number = 3
) {
  let lastError: Error | null = null;

  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      await uploadFn();
      return;
    } catch (error) {
      lastError = error as Error;
      console.log(`Attempt ${attempt + 1} failed, retrying...`);
      await new Promise((resolve) => setTimeout(resolve, 1000 * Math.pow(2, attempt)));
    }
  }

  throw lastError;
}
```

---

#### **Summary**

**Large File Upload Handling:**

1. **Progress Tracking**: Use XMLHttpRequest.upload.progress event
2. **Chunked Upload**: Split large files into manageable chunks
3. **Resumable Upload**: Save progress, resume after failures
4. **Cancellation**: Support aborting uploads with AbortController
5. **Multiple Files**: Handle concurrent uploads with queue
6. **Drag and Drop**: Improve UX with drag-and-drop interface
7. **Validation**: Check file size and type before upload
8. **Retry Logic**: Automatically retry failed chunks

**Key Techniques:**
- XMLHttpRequest for native progress support
- FormData for file transmission
- Chunking for large files (5-10MB chunks)
- LocalStorage for resumable upload state
- Optimistic UI with progress indicators

Proper file upload handling ensures **reliable, user-friendly** upload experiences even for very large files.

</details>

---

### 237. How do you implement infinite scrolling efficiently?

<details>
<summary>View Answer</summary>

**Infinite scrolling** loads content progressively as the user scrolls, creating a seamless browsing experience. Key challenges include **performance optimization**, **preventing memory leaks**, and **handling edge cases** like slow networks.

---

#### **Basic Infinite Scroll with Intersection Observer**

**Using Intersection Observer API:**

```typescript
// src/hooks/useInfiniteScroll.ts
import { useEffect, useRef, useCallback } from 'react';

interface UseInfiniteScrollOptions {
  loading: boolean;
  hasMore: boolean;
  onLoadMore: () => void;
  threshold?: number;
  rootMargin?: string;
}

export function useInfiniteScroll(options: UseInfiniteScrollOptions) {
  const { loading, hasMore, onLoadMore, threshold = 1.0, rootMargin = '100px' } = options;
  
  const observerRef = useRef<IntersectionObserver | null>(null);
  const loadMoreRef = useRef<HTMLDivElement | null>(null);

  const handleObserver = useCallback(
    (entries: IntersectionObserverEntry[]) => {
      const [entry] = entries;
      
      if (entry.isIntersecting && !loading && hasMore) {
        onLoadMore();
      }
    },
    [loading, hasMore, onLoadMore]
  );

  useEffect(() => {
    const element = loadMoreRef.current;
    if (!element) return;

    // Create observer
    observerRef.current = new IntersectionObserver(handleObserver, {
      threshold,
      rootMargin,
    });

    // Observe element
    observerRef.current.observe(element);

    // Cleanup
    return () => {
      if (observerRef.current) {
        observerRef.current.disconnect();
      }
    };
  }, [handleObserver, threshold, rootMargin]);

  return loadMoreRef;
}
```

**Usage:**

```typescript
// src/components/InfiniteScrollList.tsx
import { useState, useEffect } from 'react';
import { useInfiniteScroll } from '../hooks/useInfiniteScroll';

interface Post {
  id: number;
  title: string;
  content: string;
}

function InfiniteScrollList() {
  const [posts, setPosts] = useState<Post[]>([]);
  const [page, setPage] = useState(1);
  const [loading, setLoading] = useState(false);
  const [hasMore, setHasMore] = useState(true);

  const loadMoreRef = useInfiniteScroll({
    loading,
    hasMore,
    onLoadMore: () => setPage((p) => p + 1),
    rootMargin: '200px', // Load before reaching bottom
  });

  useEffect(() => {
    loadPosts();
  }, [page]);

  async function loadPosts() {
    if (loading) return;

    setLoading(true);

    try {
      const response = await fetch(`/api/posts?page=${page}&limit=20`);
      const newPosts = await response.json();

      if (newPosts.length === 0) {
        setHasMore(false);
      } else {
        setPosts((prev) => [...prev, ...newPosts]);
      }
    } catch (error) {
      console.error('Failed to load posts:', error);
    } finally {
      setLoading(false);
    }
  }

  return (
    <div className="infinite-scroll-list">
      <div className="posts">
        {posts.map((post) => (
          <article key={post.id} className="post">
            <h2>{post.title}</h2>
            <p>{post.content}</p>
          </article>
        ))}
      </div>

      {/* Loading indicator */}
      {loading && (
        <div className="loading">
          <div className="spinner" />
          <p>Loading more posts...</p>
        </div>
      )}

      {/* Load more trigger */}
      {!loading && hasMore && (
        <div ref={loadMoreRef} className="load-more-trigger" />
      )}

      {/* End message */}
      {!hasMore && (
        <div className="end-message">
          <p>You've reached the end!</p>
        </div>
      )}
    </div>
  );
}

export default InfiniteScrollList;
```

---

#### **Virtual Scrolling (For Large Lists)**

**Using react-window for efficient rendering:**

```typescript
// src/components/VirtualizedList.tsx
import { FixedSizeList as List } from 'react-window';
import InfiniteLoader from 'react-window-infinite-loader';
import { useState, useCallback } from 'react';

interface Item {
  id: number;
  title: string;
  description: string;
}

function VirtualizedList() {
  const [items, setItems] = useState<Item[]>([]);
  const [loading, setLoading] = useState(false);
  const [hasNextPage, setHasNextPage] = useState(true);

  const loadMoreItems = useCallback(
    async (startIndex: number, stopIndex: number) => {
      if (loading) return;

      setLoading(true);

      try {
        const response = await fetch(
          `/api/items?start=${startIndex}&end=${stopIndex}`
        );
        const newItems = await response.json();

        setItems((prev) => {
          const updated = [...prev];
          newItems.forEach((item: Item, index: number) => {
            updated[startIndex + index] = item;
          });
          return updated;
        });

        if (newItems.length === 0) {
          setHasNextPage(false);
        }
      } catch (error) {
        console.error('Failed to load items:', error);
      } finally {
        setLoading(false);
      }
    },
    [loading]
  );

  const isItemLoaded = (index: number) => !!items[index];

  const itemCount = hasNextPage ? items.length + 10 : items.length;

  const Row = ({ index, style }: { index: number; style: React.CSSProperties }) => {
    const item = items[index];

    if (!item) {
      return (
        <div style={style} className="list-item loading">
          Loading...
        </div>
      );
    }

    return (
      <div style={style} className="list-item">
        <h3>{item.title}</h3>
        <p>{item.description}</p>
      </div>
    );
  };

  return (
    <InfiniteLoader
      isItemLoaded={isItemLoaded}
      itemCount={itemCount}
      loadMoreItems={loadMoreItems}
      threshold={5} // Start loading 5 items before reaching end
    >
      {({ onItemsRendered, ref }) => (
        <List
          height={600}
          itemCount={itemCount}
          itemSize={100}
          width="100%"
          onItemsRendered={onItemsRendered}
          ref={ref}
        >
          {Row}
        </List>
      )}
    </InfiniteLoader>
  );
}

export default VirtualizedList;
```

---

#### **Advanced: With React Query**

**Infinite queries with caching:**

```typescript
// src/hooks/useInfinitePosts.ts
import { useInfiniteQuery } from '@tanstack/react-query';

interface Post {
  id: number;
  title: string;
  content: string;
}

interface PostsResponse {
  posts: Post[];
  nextCursor: number | null;
}

async function fetchPosts({ pageParam = 0 }): Promise<PostsResponse> {
  const response = await fetch(`/api/posts?cursor=${pageParam}&limit=20`);
  return response.json();
}

export function useInfinitePosts() {
  return useInfiniteQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts,
    getNextPageParam: (lastPage) => lastPage.nextCursor,
    initialPageParam: 0,
  });
}
```

**Component:**

```typescript
// src/components/InfinitePostsWithQuery.tsx
import { useInfinitePosts } from '../hooks/useInfinitePosts';
import { useInView } from 'react-intersection-observer';
import { useEffect } from 'react';

function InfinitePostsWithQuery() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
    isLoading,
    isError,
  } = useInfinitePosts();

  const { ref, inView } = useInView({
    threshold: 0,
    rootMargin: '200px',
  });

  // Load more when in view
  useEffect(() => {
    if (inView && hasNextPage && !isFetchingNextPage) {
      fetchNextPage();
    }
  }, [inView, hasNextPage, isFetchingNextPage, fetchNextPage]);

  if (isLoading) {
    return <div>Loading...</div>;
  }

  if (isError) {
    return <div>Error loading posts</div>;
  }

  const posts = data?.pages.flatMap((page) => page.posts) ?? [];

  return (
    <div className="infinite-posts">
      {posts.map((post) => (
        <article key={post.id} className="post">
          <h2>{post.title}</h2>
          <p>{post.content}</p>
        </article>
      ))}

      {/* Load more trigger */}
      {hasNextPage && (
        <div ref={ref} className="load-more">
          {isFetchingNextPage ? 'Loading more...' : 'Load more'}
        </div>
      )}

      {!hasNextPage && <div className="end">No more posts</div>}
    </div>
  );
}

export default InfinitePostsWithQuery;
```

---

#### **Real-World Example: Product List with Filters**

```typescript
// src/components/ProductList.tsx
import { useState, useEffect, useCallback } from 'react';
import { useInfiniteScroll } from '../hooks/useInfiniteScroll';

interface Product {
  id: number;
  name: string;
  price: number;
  image: string;
  category: string;
}

interface Filters {
  category: string;
  minPrice: number;
  maxPrice: number;
  search: string;
}

function ProductList() {
  const [products, setProducts] = useState<Product[]>([]);
  const [page, setPage] = useState(1);
  const [loading, setLoading] = useState(false);
  const [hasMore, setHasMore] = useState(true);
  const [filters, setFilters] = useState<Filters>({
    category: '',
    minPrice: 0,
    maxPrice: 10000,
    search: '',
  });

  const loadMoreRef = useInfiniteScroll({
    loading,
    hasMore,
    onLoadMore: () => setPage((p) => p + 1),
  });

  // Reset when filters change
  useEffect(() => {
    setProducts([]);
    setPage(1);
    setHasMore(true);
  }, [filters]);

  useEffect(() => {
    loadProducts();
  }, [page, filters]);

  async function loadProducts() {
    if (loading) return;

    setLoading(true);

    try {
      const params = new URLSearchParams({
        page: page.toString(),
        limit: '20',
        category: filters.category,
        minPrice: filters.minPrice.toString(),
        maxPrice: filters.maxPrice.toString(),
        search: filters.search,
      });

      const response = await fetch(`/api/products?${params}`);
      const data = await response.json();

      if (data.products.length === 0) {
        setHasMore(false);
      } else {
        setProducts((prev) => (page === 1 ? data.products : [...prev, ...data.products]));
      }
    } catch (error) {
      console.error('Failed to load products:', error);
    } finally {
      setLoading(false);
    }
  }

  const handleFilterChange = useCallback(
    (key: keyof Filters, value: string | number) => {
      setFilters((prev) => ({ ...prev, [key]: value }));
    },
    []
  );

  return (
    <div className="product-list-container">
      {/* Filters */}
      <aside className="filters">
        <h3>Filters</h3>
        
        <div className="filter-group">
          <label>Search</label>
          <input
            type="text"
            value={filters.search}
            onChange={(e) => handleFilterChange('search', e.target.value)}
            placeholder="Search products..."
          />
        </div>

        <div className="filter-group">
          <label>Category</label>
          <select
            value={filters.category}
            onChange={(e) => handleFilterChange('category', e.target.value)}
          >
            <option value="">All Categories</option>
            <option value="electronics">Electronics</option>
            <option value="clothing">Clothing</option>
            <option value="books">Books</option>
          </select>
        </div>

        <div className="filter-group">
          <label>Price Range</label>
          <input
            type="range"
            min="0"
            max="10000"
            value={filters.maxPrice}
            onChange={(e) => handleFilterChange('maxPrice', Number(e.target.value))}
          />
          <span>${filters.minPrice} - ${filters.maxPrice}</span>
        </div>
      </aside>

      {/* Product Grid */}
      <main className="products">
        <div className="product-grid">
          {products.map((product) => (
            <div key={product.id} className="product-card">
              <img src={product.image} alt={product.name} />
              <h3>{product.name}</h3>
              <p className="price">${product.price}</p>
              <button>Add to Cart</button>
            </div>
          ))}
        </div>

        {/* Loading State */}
        {loading && (
          <div className="loading-grid">
            {[...Array(8)].map((_, i) => (
              <div key={i} className="product-card skeleton">
                <div className="skeleton-image" />
                <div className="skeleton-title" />
                <div className="skeleton-price" />
              </div>
            ))}
          </div>
        )}

        {/* Load More Trigger */}
        {!loading && hasMore && <div ref={loadMoreRef} style={{ height: '20px' }} />}

        {/* End Message */}
        {!hasMore && products.length > 0 && (
          <div className="end-message">
            <p>You've viewed all {products.length} products</p>
          </div>
        )}

        {/* Empty State */}
        {!loading && products.length === 0 && (
          <div className="empty-state">
            <p>No products found</p>
            <button onClick={() => setFilters({
              category: '',
              minPrice: 0,
              maxPrice: 10000,
              search: '',
            })}>
              Clear Filters
            </button>
          </div>
        )}
      </main>
    </div>
  );
}

export default ProductList;
```

---

#### **Error Handling & Retry**

```typescript
// src/components/InfiniteScrollWithRetry.tsx
import { useState } from 'react';

function InfiniteScrollWithRetry() {
  const [items, setItems] = useState<any[]>([]);
  const [error, setError] = useState<string | null>(null);
  const [loading, setLoading] = useState(false);
  const [page, setPage] = useState(1);

  async function loadMore() {
    setLoading(true);
    setError(null);

    try {
      const response = await fetch(`/api/items?page=${page}`);
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }

      const newItems = await response.json();
      setItems((prev) => [...prev, ...newItems]);
      setPage((p) => p + 1);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Failed to load');
    } finally {
      setLoading(false);
    }
  }

  return (
    <div>
      {items.map((item) => (
        <div key={item.id}>{item.name}</div>
      ))}

      {loading && <div>Loading...</div>}

      {error && (
        <div className="error">
          <p>Error: {error}</p>
          <button onClick={loadMore}>Retry</button>
        </div>
      )}

      {!loading && !error && (
        <button onClick={loadMore}>Load More</button>
      )}
    </div>
  );
}
```

---

#### **Performance Optimizations**

**1. Debounce Scroll Events:**

```typescript
import { useMemo } from 'react';
import debounce from 'lodash/debounce';

function useInfiniteScrollWithDebounce() {
  const debouncedLoadMore = useMemo(
    () => debounce(() => loadMore(), 300),
    []
  );

  // Use debouncedLoadMore in scroll handler
}
```

**2. Memoize Items:**

```typescript
import { memo } from 'react';

const ProductCard = memo(({ product }: { product: Product }) => (
  <div className="product-card">
    <img src={product.image} alt={product.name} />
    <h3>{product.name}</h3>
    <p>${product.price}</p>
  </div>
));
```

**3. Image Lazy Loading:**

```typescript
<img
  src={product.image}
  alt={product.name}
  loading="lazy" // Native lazy loading
/>
```

**4. Prefetch Next Page:**

```typescript
useEffect(() => {
  if (page < totalPages) {
    // Prefetch next page
    fetch(`/api/items?page=${page + 1}`);
  }
}, [page]);
```

---

#### **Best Practices**

**1. Use Intersection Observer** (better than scroll events)
**2. Set appropriate rootMargin** (load before reaching bottom)
**3. Virtual scrolling** for very large lists (10k+ items)
**4. Debounce/throttle** rapid scroll events
**5. Show loading states** (skeleton screens)
**6. Handle errors gracefully** with retry button
**7. Prevent duplicate requests** with loading flag
**8. Reset on filter changes**
**9. Cache results** with React Query or SWR
**10. Optimize images** with lazy loading

---

#### **Summary**

**Infinite Scrolling Implementation:**

1. **Intersection Observer**: Modern, efficient way to detect scrolling
2. **Virtual Scrolling**: For lists with 1000+ items (react-window)
3. **Pagination**: Load data in pages/batches
4. **Loading States**: Show skeletons/spinners while loading
5. **Error Handling**: Retry button for failed loads
6. **Performance**: Memoization, debouncing, lazy loading
7. **React Query**: Built-in infinite query support with caching

**Key Components:**
- `useInfiniteScroll` hook with Intersection Observer
- Load more trigger element (sentinel)
- Loading indicators
- End-of-list message
- Error handling with retry

**Optimization Techniques:**
- Virtual scrolling for large lists
- Image lazy loading
- Debounced scroll handlers
- Memoized components
- Prefetching next page

Infinite scrolling creates **seamless, engaging browsing experiences** similar to social media feeds and e-commerce sites.

</details>

---

### 238. How do you handle internationalization (i18n)?

<details>
<summary>View Answer</summary>

**Internationalization (i18n)** enables your React app to support multiple languages and locales. The most popular solution is **react-i18next**, which provides translation management, language switching, and formatting utilities.

---

#### **Basic Setup with react-i18next**

**1. Installation:**

```bash
npm install react-i18next i18next
```

**2. Configuration:**

```typescript
// src/i18n/config.ts
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import LanguageDetector from 'i18next-browser-languagedetector';

// Translation resources
const resources = {
  en: {
    translation: {
      welcome: 'Welcome to React',
      greeting: 'Hello, {{name}}!',
      itemCount: 'You have {{count}} item',
      itemCount_plural: 'You have {{count}} items',
      nav: {
        home: 'Home',
        about: 'About',
        contact: 'Contact',
      },
      form: {
        submit: 'Submit',
        cancel: 'Cancel',
        email: 'Email Address',
        password: 'Password',
      },
    },
  },
  es: {
    translation: {
      welcome: 'Bienvenido a React',
      greeting: '¡Hola, {{name}}!',
      itemCount: 'Tienes {{count}} artículo',
      itemCount_plural: 'Tienes {{count}} artículos',
      nav: {
        home: 'Inicio',
        about: 'Acerca de',
        contact: 'Contacto',
      },
      form: {
        submit: 'Enviar',
        cancel: 'Cancelar',
        email: 'Correo Electrónico',
        password: 'Contraseña',
      },
    },
  },
  fr: {
    translation: {
      welcome: 'Bienvenue à React',
      greeting: 'Bonjour, {{name}}!',
      itemCount: 'Vous avez {{count}} article',
      itemCount_plural: 'Vous avez {{count}} articles',
      nav: {
        home: 'Accueil',
        about: 'À propos',
        contact: 'Contact',
      },
      form: {
        submit: 'Soumettre',
        cancel: 'Annuler',
        email: 'Adresse Email',
        password: 'Mot de passe',
      },
    },
  },
};

i18n
  .use(LanguageDetector) // Detect user language
  .use(initReactI18next) // Pass i18n to react-i18next
  .init({
    resources,
    fallbackLng: 'en',
    debug: process.env.NODE_ENV === 'development',
    
    interpolation: {
      escapeValue: false, // React already escapes
    },

    detection: {
      // Order of language detection
      order: ['localStorage', 'navigator', 'htmlTag'],
      caches: ['localStorage'],
    },
  });

export default i18n;
```

**3. Initialize in App:**

```typescript
// src/main.tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import './i18n/config'; // Initialize i18n
import App from './App';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

---

#### **Using Translations**

**1. Basic Translation Hook:**

```typescript
// src/components/Welcome.tsx
import { useTranslation } from 'react-i18next';

function Welcome() {
  const { t } = useTranslation();

  return (
    <div>
      <h1>{t('welcome')}</h1>
      <p>{t('greeting', { name: 'John' })}</p>
    </div>
  );
}

export default Welcome;
```

**2. Pluralization:**

```typescript
function ItemCounter({ count }: { count: number }) {
  const { t } = useTranslation();

  return <p>{t('itemCount', { count })}</p>;
}
// count=1: "You have 1 item"
// count=5: "You have 5 items"
```

**3. Nested Translations:**

```typescript
function Navigation() {
  const { t } = useTranslation();

  return (
    <nav>
      <a href="/">{t('nav.home')}</a>
      <a href="/about">{t('nav.about')}</a>
      <a href="/contact">{t('nav.contact')}</a>
    </nav>
  );
}
```

---

#### **Language Switcher**

```typescript
// src/components/LanguageSwitcher.tsx
import { useTranslation } from 'react-i18next';

const languages = [
  { code: 'en', name: 'English', flag: '🇺🇸' },
  { code: 'es', name: 'Español', flag: '🇪🇸' },
  { code: 'fr', name: 'Français', flag: '🇫🇷' },
  { code: 'de', name: 'Deutsch', flag: '🇩🇪' },
  { code: 'ja', name: '日本語', flag: '🇯🇵' },
];

function LanguageSwitcher() {
  const { i18n } = useTranslation();

  const changeLanguage = (languageCode: string) => {
    i18n.changeLanguage(languageCode);
  };

  return (
    <div className="language-switcher">
      <select
        value={i18n.language}
        onChange={(e) => changeLanguage(e.target.value)}
      >
        {languages.map((lang) => (
          <option key={lang.code} value={lang.code}>
            {lang.flag} {lang.name}
          </option>
        ))}
      </select>
    </div>
  );
}

export default LanguageSwitcher;
```

**Dropdown Menu Style:**

```typescript
// src/components/LanguageMenu.tsx
import { useState } from 'react';
import { useTranslation } from 'react-i18next';

function LanguageMenu() {
  const { i18n } = useTranslation();
  const [isOpen, setIsOpen] = useState(false);

  const currentLanguage = languages.find((lang) => lang.code === i18n.language);

  return (
    <div className="language-menu">
      <button
        onClick={() => setIsOpen(!isOpen)}
        className="language-button"
      >
        {currentLanguage?.flag} {currentLanguage?.name}
        <span className="arrow">{isOpen ? '▲' : '▼'}</span>
      </button>

      {isOpen && (
        <ul className="language-dropdown">
          {languages.map((lang) => (
            <li
              key={lang.code}
              onClick={() => {
                i18n.changeLanguage(lang.code);
                setIsOpen(false);
              }}
              className={i18n.language === lang.code ? 'active' : ''}
            >
              {lang.flag} {lang.name}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

---

#### **Advanced Features**

**1. Date and Number Formatting:**

```typescript
// src/utils/formatters.ts
import { useTranslation } from 'react-i18next';

export function useFormatting() {
  const { i18n } = useTranslation();

  const formatDate = (date: Date, options?: Intl.DateTimeFormatOptions) => {
    return new Intl.DateTimeFormat(i18n.language, options).format(date);
  };

  const formatNumber = (num: number, options?: Intl.NumberFormatOptions) => {
    return new Intl.NumberFormat(i18n.language, options).format(num);
  };

  const formatCurrency = (amount: number, currency: string = 'USD') => {
    return new Intl.NumberFormat(i18n.language, {
      style: 'currency',
      currency,
    }).format(amount);
  };

  const formatRelativeTime = (date: Date) => {
    const rtf = new Intl.RelativeTimeFormat(i18n.language, { numeric: 'auto' });
    const diffInSeconds = (date.getTime() - Date.now()) / 1000;

    if (Math.abs(diffInSeconds) < 60) {
      return rtf.format(Math.round(diffInSeconds), 'second');
    } else if (Math.abs(diffInSeconds) < 3600) {
      return rtf.format(Math.round(diffInSeconds / 60), 'minute');
    } else if (Math.abs(diffInSeconds) < 86400) {
      return rtf.format(Math.round(diffInSeconds / 3600), 'hour');
    } else {
      return rtf.format(Math.round(diffInSeconds / 86400), 'day');
    }
  };

  return { formatDate, formatNumber, formatCurrency, formatRelativeTime };
}
```

**Usage:**

```typescript
function ProductCard({ product }: { product: Product }) {
  const { formatCurrency, formatDate } = useFormatting();

  return (
    <div className="product-card">
      <h3>{product.name}</h3>
      <p className="price">{formatCurrency(product.price)}</p>
      <p className="date">
        Available: {formatDate(product.availableDate, { dateStyle: 'medium' })}
      </p>
    </div>
  );
}
```

**2. Loading Translations from Backend:**

```typescript
// src/i18n/backend.ts
import i18n from 'i18next';
import Backend from 'i18next-http-backend';
import { initReactI18next } from 'react-i18next';
import LanguageDetector from 'i18next-browser-languagedetector';

i18n
  .use(Backend) // Load translations from backend
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    fallbackLng: 'en',
    backend: {
      // Path to translations
      loadPath: '/locales/{{lng}}/{{ns}}.json',
      // Optional: custom request options
      requestOptions: {
        cache: 'no-cache',
      },
    },
    ns: ['common', 'auth', 'dashboard'],
    defaultNS: 'common',
  });

export default i18n;
```

**3. Namespaces for Organization:**

```typescript
// Split translations into namespaces
const resources = {
  en: {
    common: {
      submit: 'Submit',
      cancel: 'Cancel',
    },
    auth: {
      login: 'Log In',
      logout: 'Log Out',
      register: 'Sign Up',
    },
    dashboard: {
      welcome: 'Welcome to Dashboard',
      stats: 'Statistics',
    },
  },
};

// Usage
function LoginForm() {
  const { t } = useTranslation('auth');
  return <button>{t('login')}</button>;
}
```

**4. RTL (Right-to-Left) Support:**

```typescript
// src/components/RTLProvider.tsx
import { useEffect } from 'react';
import { useTranslation } from 'react-i18next';

const RTL_LANGUAGES = ['ar', 'he', 'fa'];

function RTLProvider({ children }: { children: React.ReactNode }) {
  const { i18n } = useTranslation();

  useEffect(() => {
    const isRTL = RTL_LANGUAGES.includes(i18n.language);
    document.documentElement.dir = isRTL ? 'rtl' : 'ltr';
    document.documentElement.lang = i18n.language;
  }, [i18n.language]);

  return <>{children}</>;
}

export default RTLProvider;
```

**CSS for RTL:**

```css
/* src/styles/rtl.css */
[dir="rtl"] .margin-left {
  margin-left: 0;
  margin-right: 1rem;
}

[dir="rtl"] .text-left {
  text-align: right;
}

[dir="rtl"] .icon-before::before {
  margin-left: 0.5rem;
  margin-right: 0;
}

/* Use logical properties (modern approach) */
.container {
  margin-inline-start: 1rem; /* Automatically adjusts for RTL */
  padding-inline-end: 2rem;
}
```

---

#### **Real-World Example: E-commerce Site**

```typescript
// src/pages/ProductPage.tsx
import { useTranslation } from 'react-i18next';
import { useFormatting } from '../utils/formatters';
import LanguageSwitcher from '../components/LanguageSwitcher';

interface Product {
  id: number;
  name: string;
  description: string;
  price: number;
  stock: number;
  releaseDate: Date;
}

function ProductPage({ product }: { product: Product }) {
  const { t } = useTranslation(['common', 'product']);
  const { formatCurrency, formatDate } = useFormatting();

  return (
    <div className="product-page">
      <header>
        <LanguageSwitcher />
      </header>

      <main>
        <div className="product-details">
          <h1>{product.name}</h1>
          <p className="description">{product.description}</p>

          <div className="pricing">
            <span className="price">{formatCurrency(product.price)}</span>
            {product.stock > 0 ? (
              <span className="in-stock">{t('product:inStock')}</span>
            ) : (
              <span className="out-of-stock">{t('product:outOfStock')}</span>
            )}
          </div>

          <div className="release-info">
            <span>{t('product:releaseDate')}:</span>
            <span>{formatDate(product.releaseDate, { dateStyle: 'long' })}</span>
          </div>

          <div className="actions">
            <button className="add-to-cart">
              {t('common:addToCart')}
            </button>
            <button className="buy-now">
              {t('common:buyNow')}
            </button>
          </div>

          <div className="stock-info">
            {t('product:itemsLeft', { count: product.stock })}
          </div>
        </div>
      </main>
    </div>
  );
}

export default ProductPage;
```

**Translation Files:**

```json
// public/locales/en/product.json
{
  "inStock": "In Stock",
  "outOfStock": "Out of Stock",
  "releaseDate": "Release Date",
  "itemsLeft": "Only {{count}} item left",
  "itemsLeft_plural": "{{count}} items available",
  "addedToCart": "Added to cart successfully!",
  "specifications": "Specifications",
  "reviews": "Customer Reviews"
}
```

```json
// public/locales/es/product.json
{
  "inStock": "En Stock",
  "outOfStock": "Agotado",
  "releaseDate": "Fecha de Lanzamiento",
  "itemsLeft": "Solo {{count}} artículo disponible",
  "itemsLeft_plural": "{{count}} artículos disponibles",
  "addedToCart": "¡Agregado al carrito exitosamente!",
  "specifications": "Especificaciones",
  "reviews": "Opiniones de Clientes"
}
```

---

#### **Dynamic Content Translation**

**Translating API responses:**

```typescript
// src/hooks/useTranslatedContent.ts
import { useTranslation } from 'react-i18next';
import { useEffect, useState } from 'react';

interface Content {
  id: number;
  title_en: string;
  title_es: string;
  title_fr: string;
  description_en: string;
  description_es: string;
  description_fr: string;
}

export function useTranslatedContent(content: Content) {
  const { i18n } = useTranslation();
  const [translated, setTranslated] = useState({
    title: '',
    description: '',
  });

  useEffect(() => {
    const lang = i18n.language.split('-')[0]; // Get base language
    
    setTranslated({
      title: content[`title_${lang}`] || content.title_en,
      description: content[`description_${lang}`] || content.description_en,
    });
  }, [content, i18n.language]);

  return translated;
}
```

---

#### **Lazy Loading Translations**

```typescript
// src/i18n/lazyConfig.ts
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import Backend from 'i18next-http-backend';

i18n
  .use(Backend)
  .use(initReactI18next)
  .init({
    fallbackLng: 'en',
    backend: {
      loadPath: '/locales/{{lng}}/{{ns}}.json',
    },
    ns: ['common'], // Load common namespace initially
    defaultNS: 'common',
    react: {
      useSuspense: true, // Use Suspense for loading
    },
  });

// Load namespace on demand
export async function loadNamespace(ns: string) {
  if (!i18n.hasResourceBundle(i18n.language, ns)) {
    await i18n.loadNamespaces(ns);
  }
}
```

**Usage with Suspense:**

```typescript
import { Suspense } from 'react';
import { useTranslation } from 'react-i18next';

function DashboardContent() {
  const { t } = useTranslation('dashboard');
  return <div>{t('welcome')}</div>;
}

function Dashboard() {
  return (
    <Suspense fallback={<div>Loading translations...</div>}>
      <DashboardContent />
    </Suspense>
  );
}
```

---

#### **Best Practices**

**1. Organize by Feature:**

```
public/locales/
  en/
    common.json
    auth.json
    dashboard.json
    products.json
  es/
    common.json
    auth.json
    ...
```

**2. Use Keys Descriptively:**

```typescript
// Good
t('auth.login.emailPlaceholder')
t('product.addToCart.success')

// Avoid
t('text1')
t('btn')
```

**3. Handle Missing Translations:**

```typescript
i18n.init({
  saveMissing: true, // Log missing keys
  missingKeyHandler: (lngs, ns, key) => {
    console.warn(`Missing translation: ${key}`);
  },
});
```

**4. Type Safety with TypeScript:**

```typescript
// src/types/i18next.d.ts
import 'react-i18next';
import common from '../locales/en/common.json';
import auth from '../locales/en/auth.json';

declare module 'react-i18next' {
  interface CustomTypeOptions {
    resources: {
      common: typeof common;
      auth: typeof auth;
    };
  }
}

// Now get autocomplete:
const { t } = useTranslation('auth');
t('login.email'); // Autocompleted!
```

---

#### **Testing i18n**

```typescript
// src/tests/i18n.test.tsx
import { render, screen } from '@testing-library/react';
import { I18nextProvider } from 'react-i18next';
import i18n from '../i18n/config';
import Welcome from '../components/Welcome';

test('renders translated text', async () => {
  await i18n.changeLanguage('en');
  
  render(
    <I18nextProvider i18n={i18n}>
      <Welcome />
    </I18nextProvider>
  );

  expect(screen.getByText('Welcome to React')).toBeInTheDocument();
});

test('switches language', async () => {
  await i18n.changeLanguage('es');
  
  render(
    <I18nextProvider i18n={i18n}>
      <Welcome />
    </I18nextProvider>
  );

  expect(screen.getByText('Bienvenido a React')).toBeInTheDocument();
});
```

---

#### **Summary**

**Internationalization Implementation:**

1. **react-i18next**: Most popular i18n library for React
2. **Translation Files**: JSON files organized by language and namespace
3. **Language Switching**: useTranslation hook with changeLanguage
4. **Formatting**: Intl API for dates, numbers, currencies
5. **Pluralization**: Automatic plural forms
6. **RTL Support**: Dynamic direction attribute
7. **Lazy Loading**: Load translations on demand
8. **Type Safety**: TypeScript definitions for autocomplete

**Key Features:**
- Translation interpolation with variables
- Nested translation keys
- Multiple namespaces for organization
- Language detection from browser/localStorage
- Backend integration for dynamic translations
- Suspense support for async loading

**Tools & Libraries:**
- **react-i18next**: React bindings
- **i18next-http-backend**: Load from server
- **i18next-browser-languagedetector**: Auto-detect language
- **Intl API**: Native formatting (dates, numbers, currencies)

Proper i18n makes your app **accessible to global audiences** and improves user experience across different regions and cultures.

</details>

---

### 239. How do you implement dark mode in React?

<details>
<summary>View Answer</summary>

**Dark mode** provides a darker color scheme that's easier on the eyes in low-light environments. Modern implementations use **CSS variables**, **context**, and **localStorage** for persistence.

---

#### **Basic Implementation with Context**

**1. Theme Context:**

```typescript
// src/context/ThemeContext.tsx
import { createContext, useContext, useState, useEffect, ReactNode } from 'react';

type Theme = 'light' | 'dark';

interface ThemeContextType {
  theme: Theme;
  toggleTheme: () => void;
  setTheme: (theme: Theme) => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export function ThemeProvider({ children }: { children: ReactNode }) {
  const [theme, setThemeState] = useState<Theme>(() => {
    // Check localStorage first
    const saved = localStorage.getItem('theme');
    if (saved === 'light' || saved === 'dark') {
      return saved;
    }

    // Check system preference
    if (window.matchMedia('(prefers-color-scheme: dark)').matches) {
      return 'dark';
    }

    return 'light';
  });

  // Apply theme to document
  useEffect(() => {
    document.documentElement.setAttribute('data-theme', theme);
    localStorage.setItem('theme', theme);
  }, [theme]);

  // Listen to system preference changes
  useEffect(() => {
    const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
    
    const handleChange = (e: MediaQueryListEvent) => {
      // Only auto-switch if user hasn't manually set preference
      const hasManualPreference = localStorage.getItem('theme');
      if (!hasManualPreference) {
        setThemeState(e.matches ? 'dark' : 'light');
      }
    };

    mediaQuery.addEventListener('change', handleChange);
    return () => mediaQuery.removeEventListener('change', handleChange);
  }, []);

  const toggleTheme = () => {
    setThemeState((prev) => (prev === 'light' ? 'dark' : 'light'));
  };

  const setTheme = (newTheme: Theme) => {
    setThemeState(newTheme);
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}
```

**2. CSS Variables:**

```css
/* src/styles/theme.css */

:root[data-theme="light"] {
  --bg-primary: #ffffff;
  --bg-secondary: #f5f5f5;
  --bg-tertiary: #e0e0e0;
  
  --text-primary: #000000;
  --text-secondary: #666666;
  --text-tertiary: #999999;
  
  --border-color: #dddddd;
  --shadow: rgba(0, 0, 0, 0.1);
  
  --accent-primary: #007bff;
  --accent-secondary: #6c757d;
  
  --success: #28a745;
  --warning: #ffc107;
  --error: #dc3545;
}

:root[data-theme="dark"] {
  --bg-primary: #1a1a1a;
  --bg-secondary: #2d2d2d;
  --bg-tertiary: #3d3d3d;
  
  --text-primary: #ffffff;
  --text-secondary: #b0b0b0;
  --text-tertiary: #808080;
  
  --border-color: #404040;
  --shadow: rgba(0, 0, 0, 0.5);
  
  --accent-primary: #0d6efd;
  --accent-secondary: #6c757d;
  
  --success: #198754;
  --warning: #ffc107;
  --error: #dc3545;
}

/* Apply variables */
body {
  background-color: var(--bg-primary);
  color: var(--text-primary);
  transition: background-color 0.3s ease, color 0.3s ease;
}

.card {
  background-color: var(--bg-secondary);
  border: 1px solid var(--border-color);
  box-shadow: 0 2px 4px var(--shadow);
}

.button {
  background-color: var(--accent-primary);
  color: var(--text-primary);
}

.text-secondary {
  color: var(--text-secondary);
}
```

**3. Theme Toggle Component:**

```typescript
// src/components/ThemeToggle.tsx
import { useTheme } from '../context/ThemeContext';
import { Sun, Moon } from 'lucide-react'; // Icon library

function ThemeToggle() {
  const { theme, toggleTheme } = useTheme();

  return (
    <button
      onClick={toggleTheme}
      className="theme-toggle"
      aria-label="Toggle theme"
    >
      {theme === 'light' ? (
        <Moon size={20} />
      ) : (
        <Sun size={20} />
      )}
    </button>
  );
}

export default ThemeToggle;
```

**CSS for Toggle:**

```css
/* src/components/ThemeToggle.css */
.theme-toggle {
  padding: 0.5rem;
  background-color: var(--bg-secondary);
  border: 1px solid var(--border-color);
  border-radius: 50%;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s ease;
}

.theme-toggle:hover {
  background-color: var(--bg-tertiary);
  transform: scale(1.1);
}

.theme-toggle svg {
  color: var(--text-primary);
}
```

---

#### **Advanced Toggle with Animation**

```typescript
// src/components/AnimatedThemeToggle.tsx
import { useTheme } from '../context/ThemeContext';
import './AnimatedThemeToggle.css';

function AnimatedThemeToggle() {
  const { theme, toggleTheme } = useTheme();

  return (
    <button
      className={`theme-toggle-switch ${theme}`}
      onClick={toggleTheme}
      aria-label="Toggle theme"
    >
      <div className="toggle-track">
        <div className="toggle-thumb">
          {theme === 'light' ? '☀️' : '🌙'}
        </div>
      </div>
    </button>
  );
}

export default AnimatedThemeToggle;
```

```css
/* src/components/AnimatedThemeToggle.css */
.theme-toggle-switch {
  background: none;
  border: none;
  cursor: pointer;
  padding: 0;
}

.toggle-track {
  width: 60px;
  height: 30px;
  background-color: var(--bg-tertiary);
  border-radius: 15px;
  position: relative;
  transition: background-color 0.3s ease;
}

.theme-toggle-switch:hover .toggle-track {
  background-color: var(--accent-primary);
}

.toggle-thumb {
  width: 26px;
  height: 26px;
  background-color: var(--bg-primary);
  border-radius: 50%;
  position: absolute;
  top: 2px;
  left: 2px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 14px;
  transition: transform 0.3s ease;
  box-shadow: 0 2px 4px var(--shadow);
}

.theme-toggle-switch.dark .toggle-thumb {
  transform: translateX(30px);
}
```

---

#### **Three-Option Theme Selector**

```typescript
// src/components/ThemeSelector.tsx
import { useTheme } from '../context/ThemeContext';

type ThemeOption = 'light' | 'dark' | 'system';

function ThemeSelector() {
  const { theme, setTheme } = useTheme();
  const [selected, setSelected] = useState<ThemeOption>('system');

  const handleChange = (option: ThemeOption) => {
    setSelected(option);
    
    if (option === 'system') {
      const systemPreference = window.matchMedia('(prefers-color-scheme: dark)').matches
        ? 'dark'
        : 'light';
      setTheme(systemPreference);
      localStorage.removeItem('theme'); // Remove manual preference
    } else {
      setTheme(option);
    }
  };

  return (
    <div className="theme-selector">
      <label>Theme</label>
      <div className="theme-options">
        {(['light', 'dark', 'system'] as ThemeOption[]).map((option) => (
          <button
            key={option}
            className={`theme-option ${selected === option ? 'active' : ''}`}
            onClick={() => handleChange(option)}
          >
            {option === 'light' && '☀️ Light'}
            {option === 'dark' && '🌙 Dark'}
            {option === 'system' && '💻 System'}
          </button>
        ))}
      </div>
    </div>
  );
}

export default ThemeSelector;
```

---

#### **Dark Mode with Tailwind CSS**

**1. Configure Tailwind:**

```javascript
// tailwind.config.js
module.exports = {
  darkMode: 'class', // or 'media' for system preference
  content: ['./src/**/*.{js,jsx,ts,tsx}'],
  theme: {
    extend: {
      colors: {
        primary: {
          light: '#3b82f6',
          dark: '#1e40af',
        },
      },
    },
  },
};
```

**2. Theme Provider:**

```typescript
// src/context/TailwindThemeContext.tsx
import { createContext, useContext, useEffect, useState } from 'react';

type Theme = 'light' | 'dark';

const TailwindThemeContext = createContext<{
  theme: Theme;
  toggleTheme: () => void;
} | undefined>(undefined);

export function TailwindThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<Theme>(() => {
    const saved = localStorage.getItem('theme');
    return (saved as Theme) || 'light';
  });

  useEffect(() => {
    const root = document.documentElement;
    
    if (theme === 'dark') {
      root.classList.add('dark');
    } else {
      root.classList.remove('dark');
    }
    
    localStorage.setItem('theme', theme);
  }, [theme]);

  const toggleTheme = () => {
    setTheme((prev) => (prev === 'light' ? 'dark' : 'light'));
  };

  return (
    <TailwindThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </TailwindThemeContext.Provider>
  );
}

export const useTailwindTheme = () => {
  const context = useContext(TailwindThemeContext);
  if (!context) throw new Error('useTailwindTheme must be used within provider');
  return context;
};
```

**3. Usage:**

```typescript
function Card() {
  return (
    <div className="bg-white dark:bg-gray-800 text-gray-900 dark:text-white p-4 rounded-lg shadow-lg">
      <h2 className="text-xl font-bold mb-2">Card Title</h2>
      <p className="text-gray-600 dark:text-gray-300">Card content goes here</p>
    </div>
  );
}
```

---

#### **Dark Mode with Styled Components**

```typescript
// src/theme/theme.ts
export const lightTheme = {
  colors: {
    background: '#ffffff',
    surface: '#f5f5f5',
    text: '#000000',
    textSecondary: '#666666',
    border: '#dddddd',
    primary: '#007bff',
  },
};

export const darkTheme = {
  colors: {
    background: '#1a1a1a',
    surface: '#2d2d2d',
    text: '#ffffff',
    textSecondary: '#b0b0b0',
    border: '#404040',
    primary: '#0d6efd',
  },
};

export type Theme = typeof lightTheme;
```

```typescript
// src/App.tsx
import { ThemeProvider } from 'styled-components';
import { useState } from 'react';
import { lightTheme, darkTheme } from './theme/theme';

function App() {
  const [isDark, setIsDark] = useState(false);

  return (
    <ThemeProvider theme={isDark ? darkTheme : lightTheme}>
      <GlobalStyles />
      <button onClick={() => setIsDark(!isDark)}>Toggle Theme</button>
      <AppContent />
    </ThemeProvider>
  );
}
```

```typescript
// src/components/Card.tsx
import styled from 'styled-components';

const CardContainer = styled.div`
  background-color: ${(props) => props.theme.colors.surface};
  color: ${(props) => props.theme.colors.text};
  border: 1px solid ${(props) => props.theme.colors.border};
  padding: 1rem;
  border-radius: 8px;
  transition: all 0.3s ease;
`;

const CardTitle = styled.h2`
  color: ${(props) => props.theme.colors.text};
  margin-bottom: 0.5rem;
`;

function Card({ title, children }: { title: string; children: React.ReactNode }) {
  return (
    <CardContainer>
      <CardTitle>{title}</CardTitle>
      {children}
    </CardContainer>
  );
}
```

---

#### **Real-World Example: Dashboard**

```typescript
// src/pages/Dashboard.tsx
import { useTheme } from '../context/ThemeContext';
import ThemeToggle from '../components/ThemeToggle';
import './Dashboard.css';

function Dashboard() {
  const { theme } = useTheme();

  return (
    <div className="dashboard">
      <header className="dashboard-header">
        <h1>Dashboard</h1>
        <div className="header-actions">
          <ThemeToggle />
          <button className="profile-button">Profile</button>
        </div>
      </header>

      <main className="dashboard-content">
        <aside className="sidebar">
          <nav>
            <a href="#home" className="nav-item active">
              🏠 Home
            </a>
            <a href="#analytics" className="nav-item">
              📊 Analytics
            </a>
            <a href="#settings" className="nav-item">
              ⚙️ Settings
            </a>
          </nav>
        </aside>

        <section className="main-content">
          <div className="stats-grid">
            <div className="stat-card">
              <h3>Total Users</h3>
              <p className="stat-value">1,234</p>
              <span className="stat-change positive">+12%</span>
            </div>
            
            <div className="stat-card">
              <h3>Revenue</h3>
              <p className="stat-value">$45,678</p>
              <span className="stat-change positive">+8%</span>
            </div>
            
            <div className="stat-card">
              <h3>Active Sessions</h3>
              <p className="stat-value">567</p>
              <span className="stat-change negative">-3%</span>
            </div>
          </div>

          <div className="chart-container">
            <h2>Activity Over Time</h2>
            <div className="chart-placeholder">
              {/* Chart component here */}
              <p>Chart visualization (theme: {theme})</p>
            </div>
          </div>
        </section>
      </main>
    </div>
  );
}

export default Dashboard;
```

```css
/* src/pages/Dashboard.css */
.dashboard {
  min-height: 100vh;
  background-color: var(--bg-primary);
}

.dashboard-header {
  background-color: var(--bg-secondary);
  border-bottom: 1px solid var(--border-color);
  padding: 1rem 2rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.dashboard-header h1 {
  color: var(--text-primary);
  margin: 0;
}

.header-actions {
  display: flex;
  gap: 1rem;
  align-items: center;
}

.dashboard-content {
  display: flex;
  height: calc(100vh - 70px);
}

.sidebar {
  width: 250px;
  background-color: var(--bg-secondary);
  border-right: 1px solid var(--border-color);
  padding: 1rem;
}

.nav-item {
  display: block;
  padding: 0.75rem 1rem;
  color: var(--text-secondary);
  text-decoration: none;
  border-radius: 8px;
  margin-bottom: 0.5rem;
  transition: all 0.2s ease;
}

.nav-item:hover {
  background-color: var(--bg-tertiary);
  color: var(--text-primary);
}

.nav-item.active {
  background-color: var(--accent-primary);
  color: white;
}

.main-content {
  flex: 1;
  padding: 2rem;
  overflow-y: auto;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1.5rem;
  margin-bottom: 2rem;
}

.stat-card {
  background-color: var(--bg-secondary);
  border: 1px solid var(--border-color);
  border-radius: 12px;
  padding: 1.5rem;
  box-shadow: 0 2px 8px var(--shadow);
  transition: transform 0.2s ease;
}

.stat-card:hover {
  transform: translateY(-4px);
}

.stat-card h3 {
  color: var(--text-secondary);
  font-size: 0.875rem;
  margin: 0 0 0.5rem 0;
  text-transform: uppercase;
}

.stat-value {
  color: var(--text-primary);
  font-size: 2rem;
  font-weight: bold;
  margin: 0.5rem 0;
}

.stat-change {
  font-size: 0.875rem;
  font-weight: 600;
}

.stat-change.positive {
  color: var(--success);
}

.stat-change.negative {
  color: var(--error);
}

.chart-container {
  background-color: var(--bg-secondary);
  border: 1px solid var(--border-color);
  border-radius: 12px;
  padding: 1.5rem;
  box-shadow: 0 2px 8px var(--shadow);
}

.chart-container h2 {
  color: var(--text-primary);
  margin: 0 0 1rem 0;
}

.chart-placeholder {
  height: 300px;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: var(--bg-tertiary);
  border-radius: 8px;
  color: var(--text-secondary);
}
```

---

#### **Preventing Flash of Incorrect Theme**

**Inline script in HTML:**

```html
<!-- public/index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>My App</title>
  
  <!-- Prevent flash of wrong theme -->
  <script>
    (function() {
      const theme = localStorage.getItem('theme') ||
        (window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light');
      document.documentElement.setAttribute('data-theme', theme);
    })();
  </script>
</head>
<body>
  <div id="root"></div>
</body>
</html>
```

---

#### **Best Practices**

**1. Use CSS Variables** for easy theme switching
**2. Persist Theme** in localStorage
**3. Respect System Preference** as default
**4. Smooth Transitions** between themes
**5. Test Contrast** for accessibility (WCAG AA/AAA)
**6. Theme All Components** consistently
**7. Prevent Flash** with inline script
**8. Provide Manual Toggle** for user control
**9. Use Semantic Color Names** (primary, secondary, not blue, red)
**10. Test in Both Modes** during development

---

#### **Summary**

**Dark Mode Implementation:**

1. **Context API**: Manage theme state globally
2. **CSS Variables**: Define colors for both themes
3. **localStorage**: Persist user preference
4. **System Preference**: Detect with `prefers-color-scheme`
5. **Toggle Component**: UI for switching themes
6. **Smooth Transitions**: Animate color changes
7. **Prevent Flash**: Set theme before render

**Key Techniques:**
- `data-theme` attribute on root element
- CSS custom properties (variables)
- `matchMedia` for system preference
- `localStorage` for persistence
- Context for global state

**Popular Approaches:**
- **CSS Variables**: Most flexible
- **Tailwind Dark Mode**: Built-in dark: prefix
- **Styled Components**: ThemeProvider with theme objects
- **CSS Modules**: Separate stylesheets

Proper dark mode implementation provides **better UX** for users in different lighting conditions and can **reduce eye strain** during extended use.

</details>

---

### 240. How do you handle browser compatibility issues?

<details>
<summary>View Answer</summary>

**Browser compatibility** ensures your React app works across different browsers and versions. Key strategies include **transpilation**, **polyfills**, **feature detection**, and **progressive enhancement**.

---

#### **1. Babel for JavaScript Transpilation**

**Configure Babel to transpile modern JavaScript:**

```json
// .babelrc or babel.config.json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "targets": {
          "browsers": ["> 0.5%", "last 2 versions", "not dead", "not IE 11"]
        },
        "useBuiltIns": "usage",
        "corejs": 3
      }
    ],
    "@babel/preset-react",
    "@babel/preset-typescript"
  ]
}
```

**Browserslist Configuration:**

```json
// package.json
{
  "browserslist": [
    "> 0.5%",
    "last 2 versions",
    "Firefox ESR",
    "not dead",
    "not IE 11"
  ]
}
```

---

#### **2. Polyfills**

**Automatic Polyfills with core-js:**

```bash
npm install core-js regenerator-runtime
```

```typescript
// src/main.tsx
import 'core-js/stable';
import 'regenerator-runtime/runtime';

import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

**Selective Polyfills:**

```typescript
// src/polyfills.ts
import 'core-js/features/promise';
import 'core-js/features/array/from';
import 'core-js/features/array/includes';
import 'core-js/features/object/assign';
import 'whatwg-fetch';

if (!('IntersectionObserver' in window)) {
  import('intersection-observer');
}
```

---

#### **3. CSS Compatibility**

**PostCSS with Autoprefixer:**

```javascript
// postcss.config.js
module.exports = {
  plugins: [
    require('autoprefixer')({
      overrideBrowserslist: ['> 0.5%', 'last 2 versions'],
    }),
  ],
};
```

---

#### **4. Feature Detection**

```typescript
// src/utils/browserSupport.ts
export const browserSupport = {
  cssGrid: () => CSS.supports('display', 'grid'),
  intersectionObserver: () => 'IntersectionObserver' in window,
  webWorkers: () => typeof Worker !== 'undefined',
  localStorage: () => {
    try {
      const test = '__test__';
      localStorage.setItem(test, test);
      localStorage.removeItem(test);
      return true;
    } catch (e) {
      return false;
    }
  },
  touch: () => 'ontouchstart' in window || navigator.maxTouchPoints > 0,
};
```

**Hook for Feature Detection:**

```typescript
// src/hooks/useBrowserSupport.ts
import { useState, useEffect } from 'react';
import { browserSupport } from '../utils/browserSupport';

export function useBrowserSupport() {
  const [features, setFeatures] = useState({
    cssGrid: false,
    intersectionObserver: false,
    localStorage: false,
  });

  useEffect(() => {
    setFeatures({
      cssGrid: browserSupport.cssGrid(),
      intersectionObserver: browserSupport.intersectionObserver(),
      localStorage: browserSupport.localStorage(),
    });
  }, []);

  return features;
}
```

---

#### **5. Progressive Enhancement**

**Fallback Components:**

```typescript
function ImageWithFallback({ src, fallbackSrc, alt }: ImageProps) {
  const [error, setError] = useState(false);

  return (
    <img
      src={error ? fallbackSrc : src}
      alt={alt}
      onError={() => setError(true)}
    />
  );
}
```

**Picture Element:**

```typescript
function ResponsiveImage({ src, alt }: { src: string; alt: string }) {
  const basePath = src.replace(/\.[^.]+$/, '');

  return (
    <picture>
      <source srcSet={`${basePath}.avif`} type="image/avif" />
      <source srcSet={`${basePath}.webp`} type="image/webp" />
      <img src={`${basePath}.jpg`} alt={alt} />
    </picture>
  );
}
```

---

#### **6. Browser Detection**

```typescript
// src/utils/browserDetection.ts
export function detectBrowser() {
  const ua = navigator.userAgent;

  return {
    isChrome: /Chrome/.test(ua) && /Google Inc/.test(navigator.vendor),
    isFirefox: /Firefox/.test(ua),
    isSafari: /Safari/.test(ua) && /Apple Computer/.test(navigator.vendor),
    isEdge: /Edg/.test(ua),
  };
}
```

---

#### **7. Conditional Loading**

```typescript
// src/utils/loadPolyfills.ts
export async function loadPolyfills() {
  const promises: Promise<any>[] = [];

  if (!('IntersectionObserver' in window)) {
    promises.push(import('intersection-observer'));
  }

  if (!('ResizeObserver' in window)) {
    promises.push(import('@juggle/resize-observer'));
  }

  await Promise.all(promises);
}
```

---

#### **8. Testing**

**Playwright for Multi-Browser Testing:**

```typescript
// tests/compatibility.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Browser Compatibility', () => {
  test('works in Chromium', async ({ page }) => {
    await page.goto('http://localhost:3000');
    await expect(page.locator('h1')).toBeVisible();
  });

  test('works in Firefox', async ({ page, browserName }) => {
    test.skip(browserName !== 'firefox');
    await page.goto('http://localhost:3000');
    await expect(page.locator('h1')).toBeVisible();
  });
});
```

---

#### **Best Practices**

1. **Define Browser Support**: Use browserslist
2. **Transpilation**: Babel + @babel/preset-env
3. **Polyfills**: Load conditionally
4. **Feature Detection**: Test features, not browsers
5. **Progressive Enhancement**: Build baseline, enhance for modern
6. **CSS Prefixes**: Autoprefixer
7. **Testing**: Multi-browser testing with Playwright

---

#### **Summary**

**Browser Compatibility Strategies:**

1. **Transpilation**: Babel + @babel/preset-env
2. **Polyfills**: core-js for missing features
3. **CSS Prefixes**: Autoprefixer
4. **Feature Detection**: Test capabilities
5. **Progressive Enhancement**: Baseline + enhancements
6. **Testing**: Cross-browser validation

**Key Tools:**
- Babel: JavaScript transpilation
- core-js: JavaScript polyfills  
- Autoprefixer: CSS vendor prefixes
- browserslist: Define target browsers
- Playwright: Cross-browser testing

Proper browser compatibility ensures your app **works for all users** regardless of their browser.

</details>
