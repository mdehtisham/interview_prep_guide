# Behavioral Questions

> Bonus: Behavioral & System Design

---

## Questions

261. Describe a challenging React project you worked on
262. How do you stay updated with React ecosystem changes?
263. How do you handle technical debt in React projects?
264. Describe a time you optimized a slow React application
265. How do you review code from other React developers?
266. How do you mentor junior React developers?
267. How do you make technical decisions in React projects?
268. Describe a time you had to refactor legacy React code
269. How do you balance speed vs quality in development?
270. How do you handle disagreements about architecture decisions?

---

## Detailed Answers

### 261. Describe a challenging React project you worked on

<details>
<summary>View Answer</summary>

**Behavioral questions** assess your **problem-solving skills**, **technical experience**, and **communication abilities**. Use the **STAR method** (Situation, Task, Action, Result) to structure your response.

---

#### **STAR Method Framework**

**S**ituation: Context and background
**T**ask: Your specific responsibility
**A**ction: Steps you took
**R**esult: Measurable outcomes

---

#### **Example 1: E-commerce Platform Migration**

**Situation:**
- Legacy e-commerce platform built with jQuery and Backbone.js
- 200K+ lines of code, 5 years old
- Performance issues: 8-10s initial load time
- No TypeScript, no testing, poor mobile experience
- Team of 6 developers, tight 6-month deadline

**Task:**
- Lead the migration to React + TypeScript
- Improve performance by 50%+
- Maintain feature parity during migration
- Zero downtime deployment
- Train team on React best practices

**Action:**

1. **Assessment & Planning:**
```typescript
// Created migration strategy document
const migrationPhases = [
  {
    phase: 1,
    duration: '2 months',
    scope: 'Setup infrastructure',
    tasks: [
      'React + TypeScript setup',
      'CI/CD pipeline',
      'Design system foundation',
      'Testing framework',
    ],
  },
  {
    phase: 2,
    duration: '3 months',
    scope: 'Gradual migration',
    tasks: [
      'Migrate product pages',
      'Shopping cart refactor',
      'Checkout flow',
      'User dashboard',
    ],
  },
  {
    phase: 3,
    duration: '1 month',
    scope: 'Optimization & rollout',
    tasks: [
      'Performance optimization',
      'A/B testing',
      'Gradual rollout',
      'Monitoring & fixes',
    ],
  },
];
```

2. **Strangler Fig Pattern:**
```typescript
// Gradual migration - old and new code coexist
// router.tsx
const Router = () => (
  <BrowserRouter>
    <Routes>
      {/* New React pages */}
      <Route path="/products" element={<ProductsPage />} />
      <Route path="/cart" element={<CartPage />} />
      
      {/* Legacy pages via iframe wrapper */}
      <Route path="/account" element={<LegacyWrapper src="/old/account" />} />
      <Route path="/orders" element={<LegacyWrapper src="/old/orders" />} />
    </Routes>
  </BrowserRouter>
);

// LegacyWrapper component for gradual transition
const LegacyWrapper = ({ src }: { src: string }) => {
  return (
    <div className="legacy-container">
      <iframe src={src} style={{ width: '100%', border: 'none' }} />
    </div>
  );
};
```

3. **Performance Optimization:**
```typescript
// Code splitting by route
const ProductsPage = lazy(() => import('./pages/Products'));
const CartPage = lazy(() => import('./pages/Cart'));

// Image optimization
<picture>
  <source srcSet="/product-320w.webp 320w" type="image/webp" />
  <source srcSet="/product-640w.webp 640w" type="image/webp" />
  <img src="/product.jpg" loading="lazy" alt="Product" />
</picture>

// API response caching with React Query
const { data } = useQuery(['products'], fetchProducts, {
  staleTime: 5 * 60 * 1000, // 5 minutes
  cacheTime: 10 * 60 * 1000, // 10 minutes
});
```

4. **State Management:**
```typescript
// Zustand for global state (replacing Redux)
import create from 'zustand';
import { devtools, persist } from 'zustand/middleware';

interface CartStore {
  items: CartItem[];
  addItem: (item: CartItem) => void;
  removeItem: (id: string) => void;
  total: number;
}

const useCartStore = create<CartStore>()()
  devtools(
    persist(
      (set, get) => ({
        items: [],
        addItem: (item) => set((state) => ({ 
          items: [...state.items, item] 
        })),
        removeItem: (id) => set((state) => ({
          items: state.items.filter(i => i.id !== id)
        })),
        total: () => {
          return get().items.reduce((sum, item) => sum + item.price, 0);
        },
      }),
      { name: 'cart-storage' }
    )
  )
);
```

5. **Testing Strategy:**
```typescript
// Unit tests with Vitest
import { describe, it, expect } from 'vitest';
import { renderHook, act } from '@testing-library/react';
import { useCartStore } from './cartStore';

describe('CartStore', () => {
  it('adds items to cart', () => {
    const { result } = renderHook(() => useCartStore());
    
    act(() => {
      result.current.addItem({ id: '1', name: 'Product', price: 99 });
    });
    
    expect(result.current.items).toHaveLength(1);
    expect(result.current.total).toBe(99);
  });
});

// E2E tests with Playwright
import { test, expect } from '@playwright/test';

test('complete purchase flow', async ({ page }) => {
  await page.goto('/products');
  await page.click('[data-testid="product-1"]');
  await page.click('[data-testid="add-to-cart"]');
  await page.goto('/cart');
  await page.click('[data-testid="checkout"]');
  
  await expect(page.locator('[data-testid="success-message"]'))
    .toBeVisible();
});
```

**Result:**
- **70% faster load time**: 8s → 2.4s (First Contentful Paint)
- **Lighthouse score**: 45 → 92 (Performance)
- **50% fewer bugs**: Better TypeScript + testing
- **Mobile traffic increased 40%**: Better mobile UX
- **Developer velocity**: 30% faster feature development
- **Zero downtime**: Gradual rollout with feature flags
- **Team upskilled**: All 6 developers proficient in React

**Key Learnings:**
- Strangler Fig pattern works well for large migrations
- Gradual rollout reduces risk
- Performance monitoring is critical
- TypeScript catches bugs early
- E2E tests provide confidence

---

#### **Example 2: Real-Time Collaboration Tool**

**Situation:**
- Building a Figma-like design tool
- Real-time collaboration with 100+ concurrent users
- Complex canvas rendering and state synchronization
- WebSocket connection management

**Task:**
- Implement real-time collaborative editing
- Handle network failures gracefully
- Ensure smooth 60fps canvas performance
- Conflict resolution for concurrent edits

**Action:**

```typescript
// WebSocket connection with reconnection logic
import { useEffect, useRef, useState } from 'react';

interface UseWebSocketOptions {
  url: string;
  onMessage: (data: any) => void;
  reconnectAttempts?: number;
  reconnectDelay?: number;
}

function useWebSocket({ url, onMessage, reconnectAttempts = 5 }: UseWebSocketOptions) {
  const ws = useRef<WebSocket | null>(null);
  const [isConnected, setIsConnected] = useState(false);
  const reconnectCount = useRef(0);

  useEffect(() => {
    const connect = () => {
      ws.current = new WebSocket(url);

      ws.current.onopen = () => {
        setIsConnected(true);
        reconnectCount.current = 0;
        console.log('WebSocket connected');
      };

      ws.current.onmessage = (event) => {
        const data = JSON.parse(event.data);
        onMessage(data);
      };

      ws.current.onclose = () => {
        setIsConnected(false);
        
        // Exponential backoff reconnection
        if (reconnectCount.current < reconnectAttempts) {
          const delay = Math.min(1000 * Math.pow(2, reconnectCount.current), 30000);
          setTimeout(() => {
            reconnectCount.current++;
            connect();
          }, delay);
        }
      };

      ws.current.onerror = (error) => {
        console.error('WebSocket error:', error);
      };
    };

    connect();

    return () => {
      ws.current?.close();
    };
  }, [url]);

  const sendMessage = (data: any) => {
    if (ws.current?.readyState === WebSocket.OPEN) {
      ws.current.send(JSON.stringify(data));
    }
  };

  return { sendMessage, isConnected };
}

// CRDT-based conflict resolution
import * as Y from 'yjs';
import { WebsocketProvider } from 'y-websocket';

function CollaborativeCanvas() {
  const ydoc = useRef(new Y.Doc());
  const provider = useRef<WebsocketProvider>();

  useEffect(() => {
    provider.current = new WebsocketProvider(
      'wss://collaboration-server.com',
      'room-123',
      ydoc.current
    );

    const yarray = ydoc.current.getArray('shapes');
    
    // Listen to changes
    yarray.observe((event) => {
      // Update canvas
      renderCanvas(yarray.toArray());
    });

    return () => {
      provider.current?.destroy();
    };
  }, []);

  const addShape = (shape: Shape) => {
    const yarray = ydoc.current.getArray('shapes');
    yarray.push([shape]);
  };

  return <Canvas onShapeAdd={addShape} />;
}

// Optimized canvas rendering with RAF
function useCanvasRenderer(canvasRef: RefObject<HTMLCanvasElement>) {
  const frameId = useRef<number>();
  const [shapes, setShapes] = useState<Shape[]>([]);

  const render = useCallback(() => {
    const canvas = canvasRef.current;
    if (!canvas) return;

    const ctx = canvas.getContext('2d')!;
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // Render all shapes
    shapes.forEach(shape => {
      renderShape(ctx, shape);
    });

    frameId.current = requestAnimationFrame(render);
  }, [shapes]);

  useEffect(() => {
    render();
    return () => {
      if (frameId.current) {
        cancelAnimationFrame(frameId.current);
      }
    };
  }, [render]);

  return { setShapes };
}
```

**Result:**
- **Supported 500+ concurrent users** (5x initial goal)
- **99.9% uptime** with automatic reconnection
- **<50ms latency** for collaborative edits
- **60fps canvas performance** maintained
- **Zero data loss** with CRDT conflict resolution
- **$2M Series A funding** based on product demo

---

#### **Example 3: Performance Crisis**

**Situation:**
- Dashboard app with severe performance issues
- 15-20 second load time for data-heavy pages
- Frequent browser crashes with large datasets
- Customer complaints increasing

**Task:**
- Identify and fix performance bottlenecks
- Reduce load time to under 3 seconds
- Handle 10,000+ rows without crashes

**Action:**

```typescript
// Problem: Rendering 10,000 rows directly
// ❌ Before
function DataTable({ data }: { data: Row[] }) {
  return (
    <table>
      {data.map(row => <TableRow key={row.id} row={row} />)}
    </table>
  );
}

// ✅ Solution: Virtual scrolling with react-window
import { FixedSizeList } from 'react-window';

function VirtualizedTable({ data }: { data: Row[] }) {
  const Row = ({ index, style }: { index: number; style: React.CSSProperties }) => (
    <div style={style}>
      <TableRow row={data[index]} />
    </div>
  );

  return (
    <FixedSizeList
      height={600}
      itemCount={data.length}
      itemSize={50}
      width="100%"
    >
      {Row}
    </FixedSizeList>
  );
}

// Problem: Heavy computation on every render
// ❌ Before
function Dashboard({ transactions }: { transactions: Transaction[] }) {
  const total = transactions.reduce((sum, t) => sum + t.amount, 0);
  const average = total / transactions.length;
  const categories = groupByCategory(transactions); // Expensive!
  
  return <div>{/* ... */}</div>;
}

// ✅ Solution: useMemo for expensive calculations
function Dashboard({ transactions }: { transactions: Transaction[] }) {
  const stats = useMemo(() => {
    const total = transactions.reduce((sum, t) => sum + t.amount, 0);
    const average = total / transactions.length;
    const categories = groupByCategory(transactions);
    return { total, average, categories };
  }, [transactions]);

  return <div>{/* Use stats */}</div>;
}

// Problem: Fetching all data upfront
// ✅ Solution: Pagination + infinite scroll
function useInfiniteData() {
  const { data, fetchNextPage, hasNextPage } = useInfiniteQuery(
    ['transactions'],
    ({ pageParam = 0 }) => fetchTransactions(pageParam, 50),
    {
      getNextPageParam: (lastPage, pages) => lastPage.nextCursor,
    }
  );

  return { data: data?.pages.flatMap(p => p.items) || [], fetchNextPage, hasNextPage };
}
```

**Result:**
- **92% faster**: 18s → 1.4s initial load
- **No crashes**: Virtual scrolling handles 100K+ rows
- **Memory usage**: 800MB → 120MB
- **Customer satisfaction**: Complaints dropped 95%
- **Churn reduced**: 3 major clients stayed

---

#### **Interview Tips**

1. **Be Specific**: Use real numbers and metrics
2. **Show Impact**: Explain business outcomes
3. **Acknowledge Trade-offs**: Discuss alternatives considered
4. **Be Honest**: Admit mistakes and learnings
5. **Stay Technical**: Include code examples if asked
6. **Highlight Collaboration**: Mention team work
7. **Focus on Your Role**: Use "I" not just "we"
8. **Prepare Multiple Stories**: Have 3-4 examples ready

**Topics to Cover:**
- Technical challenges
- Performance optimization
- Team collaboration
- Architecture decisions
- Time pressure/deadlines
- Conflict resolution
- Learning new technologies
- Code quality improvements

</details>

---

### 262. How do you stay updated with React ecosystem changes?

<details>
<summary>View Answer</summary>

**Staying current** with React's rapid evolution requires a **systematic approach** to learning, combining **official resources**, **community engagement**, and **hands-on practice**.

---

#### **1. Official Sources**

**React Documentation:**
- **react.dev**: Official docs (re-written in 2023)
- **React Blog**: Major releases and announcements
- **React RFC**: Proposals for new features
- **React GitHub**: Issues, PRs, discussions

```typescript
// Example: Following React 19 changes
// Subscribe to React GitHub releases
// https://github.com/facebook/react/releases

// New features to watch:
// - React Server Components
// - Server Actions
// - Asset Loading
// - Document Metadata
// - use() hook
```

**Key Resources:**
```typescript
const officialResources = [
  'https://react.dev',
  'https://react.dev/blog',
  'https://github.com/reactjs/rfcs',
  'https://github.com/facebook/react',
];
```

---

#### **2. Newsletters & Email Digests**

**Weekly Newsletters:**

1. **React Status** (weekly)
   - Curated React news and articles
   - https://react.statuscode.com

2. **This Week in React** (weekly)
   - React + React Native news
   - by Sebastien Lorber

3. **Bytes.dev** (weekly)
   - JavaScript/React news with humor
   - High engagement content

4. **Frontend Focus** (weekly)
   - Broader frontend perspective
   - Includes React content

```typescript
// Personal learning schedule
const learningRoutine = {
  monday: 'Read React Status newsletter',
  tuesday: 'Check React RFC updates',
  wednesday: 'Watch conference talks',
  thursday: 'Read technical blog posts',
  friday: 'Experiment with new features',
  weekend: 'Build side projects',
};
```

---

#### **3. Social Media & Community**

**Twitter/X:**

```typescript
const reactExperts = [
  '@dan_abramov',      // React core team
  '@acdlite',          // React core team
  '@sebmarkbage',      // React core team
  '@sophiebits',       // React core team
  '@kentcdodds',       // Educator
  '@ryanflorence',     // Remix creator
  '@thebuilder_io',    // Builder.io
  '@wesbos',           // Educator
  '@addyosmani',       // Google Chrome
  '@housecor',         // Educator
];
```

**Reddit:**
- r/reactjs - 600K+ members
- r/javascript - General JS discussions
- r/Frontend - Broader frontend topics

**Discord/Slack Communities:**
- Reactiflux (150K+ members)
- React Native Community
- Next.js Discord
- Remix Discord

---

#### **4. YouTube Channels & Video Content**

```typescript
const youtubeChannels = [
  // In-depth tutorials
  'Jack Herrington',        // No BS TS/React
  'Web Dev Simplified',     // Beginner-friendly
  'Fireship',               // Quick overviews
  'Theo - t3.gg',          // Full-stack React
  'Josh tried coding',      // Modern React
  
  // Conference talks
  'React Conf',             // Official conferences
  'Vercel',                 // Next.js content
  'Frontend Masters',       // Professional courses
];
```

**Weekly Routine:**
```typescript
const videoLearning = {
  duration: '30 minutes/day',
  playbackSpeed: '1.5x',
  focus: 'New features and patterns',
  notesTaking: true,
};
```

---

#### **5. Blogs & Technical Articles**

**Individual Blogs:**

```typescript
const topBlogs = [
  'overreacted.io',           // Dan Abramov
  'kentcdodds.com',           // Kent C. Dodds
  'joshwcomeau.com',          // Josh Comeau
  'robinwieruch.de',          // Robin Wieruch
  'tkdodo.eu/blog',           // TkDodo (React Query)
  'leerob.io',                // Lee Robinson (Vercel)
];
```

**Company Blogs:**
- Vercel Blog (Next.js updates)
- Meta Engineering (React updates)
- Netflix Tech Blog
- Airbnb Engineering
- Shopify Engineering

**Reading Strategy:**
```typescript
class ArticleReader {
  private readingList: Article[] = [];
  
  addToReadingList(article: Article) {
    this.readingList.push({
      ...article,
      priority: this.calculatePriority(article),
      estimatedTime: this.estimateReadTime(article),
    });
  }
  
  private calculatePriority(article: Article): number {
    let priority = 0;
    if (article.topic.includes('React 19')) priority += 10;
    if (article.author === 'Dan Abramov') priority += 5;
    if (article.hasCodeExamples) priority += 3;
    return priority;
  }
}
```

---

#### **6. Conferences & Events**

**Major Conferences:**

```typescript
const conferences = [
  {
    name: 'React Conf',
    frequency: 'Annual',
    format: 'In-person + Virtual',
    focus: 'Official React announcements',
  },
  {
    name: 'React Summit',
    frequency: '2x/year',
    location: 'Amsterdam / Remote',
    focus: 'Community talks',
  },
  {
    name: 'Next.js Conf',
    frequency: 'Annual',
    format: 'Virtual',
    focus: 'Next.js ecosystem',
  },
  {
    name: 'React Advanced',
    location: 'London',
    focus: 'Advanced patterns',
  },
];
```

**Local Meetups:**
- React meetup groups (meetup.com)
- JavaScript user groups
- Tech company events
- Hackathons

---

#### **7. Hands-On Learning**

**Side Projects:**

```typescript
// Example: Testing React 19 features
const sideProject = {
  name: 'React 19 Playground',
  goal: 'Experiment with new features',
  features: [
    'use() hook for promises',
    'Server Components',
    'Server Actions',
    'useOptimistic hook',
  ],
  
  learningOutcomes: [
    'Understand new patterns',
    'Identify migration challenges',
    'Write about learnings',
  ],
};

// Upgrade existing projects
const upgradeSchedule = {
  'Q1 2024': 'Upgrade to React 18',
  'Q2 2024': 'Migrate to Server Components',
  'Q3 2024': 'Implement Server Actions',
  'Q4 2024': 'Full React 19 adoption',
};
```

**CodeSandbox/StackBlitz:**
```typescript
// Quick prototyping
const prototype = {
  tool: 'CodeSandbox',
  purpose: 'Test new patterns quickly',
  frequency: 'Weekly',
  shareWith: 'Team for discussion',
};
```

---

#### **8. Online Courses & Workshops**

**Platforms:**

```typescript
const learningPlatforms = [
  {
    name: 'Frontend Masters',
    pros: ['Expert instructors', 'In-depth courses'],
    courses: ['React Performance', 'Advanced Patterns'],
  },
  {
    name: 'Epic React by Kent C. Dodds',
    format: 'Self-paced workshops',
    focus: 'Practical, hands-on',
  },
  {
    name: 'egghead.io',
    format: 'Short video lessons',
    updateFrequency: 'Regular',
  },
  {
    name: 'Udemy',
    pros: ['Affordable', 'Wide selection'],
    updateFrequency: 'Varies',
  },
];
```

---

#### **9. Documentation & Release Notes**

**Systematic Approach:**

```typescript
class UpdateTracker {
  private libraries = [
    'react',
    'react-dom',
    'next',
    'react-query',
    'zustand',
    'react-router',
    'vite',
  ];
  
  checkUpdates() {
    this.libraries.forEach(async (lib) => {
      const latest = await this.getLatestVersion(lib);
      const current = this.getCurrentVersion(lib);
      
      if (this.isMajorUpdate(current, latest)) {
        this.readChangelog(lib, latest);
        this.testInSandbox(lib, latest);
        this.planMigration(lib, latest);
      }
    });
  }
  
  private readChangelog(lib: string, version: string) {
    // Read CHANGELOG.md thoroughly
    // Note breaking changes
    // Understand new features
  }
  
  private testInSandbox(lib: string, version: string) {
    // Create test project
    // Try new features
    // Identify issues
  }
}
```

---

#### **10. Knowledge Sharing**

**Internal Team Sharing:**

```typescript
const knowledgeSharing = {
  // Weekly tech talks
  techTalks: {
    frequency: 'Weekly',
    duration: '30 minutes',
    format: 'Demo + discussion',
    topics: ['New React features', 'Performance tips', 'Best practices'],
  },
  
  // Internal documentation
  documentation: {
    platform: 'Notion/Confluence',
    content: [
      'React upgrade guides',
      'Pattern library',
      'Performance tips',
      'Common pitfalls',
    ],
  },
  
  // Code reviews
  codeReviews: {
    focus: 'Share modern patterns',
    educate: 'Explain better approaches',
    discuss: 'New features adoption',
  },
};
```

**External Sharing:**

```typescript
const publicContributions = {
  blog: {
    platform: 'dev.to / Medium',
    frequency: 'Monthly',
    topics: ['Learnings', 'Tutorials', 'Case studies'],
  },
  
  openSource: {
    contribute: 'React libraries',
    create: 'Useful packages',
    document: 'Usage examples',
  },
  
  speaking: {
    localMeetups: 'Present learnings',
    conferences: 'Submit talk proposals',
    podcasts: 'Share experiences',
  },
};
```

---

#### **11. Structured Learning Plan**

**Weekly Schedule:**

```typescript
const weeklyPlan = {
  monday: {
    morning: 'Read newsletters (30 min)',
    lunch: 'Watch YouTube video (20 min)',
    evening: 'Check Twitter/Reddit (15 min)',
  },
  
  tuesday: {
    morning: 'Read technical article (45 min)',
    evening: 'Experiment with new feature (1 hour)',
  },
  
  wednesday: {
    lunch: 'Conference talk replay (30 min)',
    evening: 'Side project work (1-2 hours)',
  },
  
  thursday: {
    morning: 'Check GitHub releases (15 min)',
    evening: 'Read documentation (30 min)',
  },
  
  friday: {
    morning: 'Team knowledge sharing (30 min)',
    afternoon: 'Write blog post / notes (1 hour)',
  },
  
  weekend: {
    saturday: 'Side project / experimentation (2-3 hours)',
    sunday: 'Review week learnings, plan next week',
  },
};
```

---

#### **12. Quality Filters**

**Evaluating Information:**

```typescript
class InformationFilter {
  evaluateSource(source: Source): number {
    let score = 0;
    
    // Authority
    if (source.author.isReactCoreMember) score += 10;
    if (source.author.isRecognizedExpert) score += 7;
    
    // Recency
    const monthsOld = this.getMonthsSincePublished(source.date);
    if (monthsOld < 3) score += 8;
    else if (monthsOld < 6) score += 5;
    else if (monthsOld < 12) score += 2;
    
    // Technical depth
    if (source.hasCodeExamples) score += 5;
    if (source.hasPerformanceData) score += 4;
    if (source.discussesTradeoffs) score += 3;
    
    // Peer validation
    if (source.upvotes > 100) score += 3;
    if (source.comments > 20) score += 2;
    
    return score;
  }
  
  shouldRead(source: Source): boolean {
    return this.evaluateSource(source) > 15;
  }
}
```

---

#### **Summary**

**Daily Habits (30-60 minutes):**
- Check Twitter/Reddit for news
- Read one technical article
- Watch one YouTube video
- Experiment with code

**Weekly Activities (3-5 hours):**
- Read newsletters thoroughly
- Attend meetup/watch conference talk
- Work on side project
- Share knowledge with team

**Monthly Goals:**
- Complete one course/workshop
- Write one blog post
- Contribute to open source
- Try one new library/pattern

**Key Principles:**
1. **Consistent**: Daily learning habit
2. **Focused**: Quality over quantity
3. **Hands-on**: Learn by building
4. **Share**: Teach to reinforce learning
5. **Critical**: Evaluate sources carefully
6. **Balanced**: Don't chase every trend
7. **Practical**: Apply learnings at work
8. **Community**: Engage with others

**Tools to Track Learning:**
- Notion/Obsidian for notes
- Pocket/Instapaper for reading list
- GitHub stars for resources
- Anki for spaced repetition
- Daily journaling

Staying updated is a **continuous journey** requiring **discipline**, **curiosity**, and **active participation** in the React community.

</details>

---

### 263. How do you handle technical debt in React projects?

<details>
<summary>View Answer</summary>

**Technical debt** is inevitable in software development. The key is to **manage it systematically**, **prioritize effectively**, and **prevent accumulation** through good practices.

---

#### **1. Identifying Technical Debt**

**Types of Technical Debt:**

```typescript
enum TechnicalDebtType {
  CODE_QUALITY = 'Code Quality',           // Poor patterns, duplications
  ARCHITECTURE = 'Architecture',           // Scalability issues
  PERFORMANCE = 'Performance',             // Slow rendering, memory leaks
  TESTING = 'Testing',                     // Missing or poor test coverage
  DEPENDENCIES = 'Dependencies',           // Outdated libraries
  DOCUMENTATION = 'Documentation',         // Missing or outdated docs
  SECURITY = 'Security',                   // Vulnerabilities
  ACCESSIBILITY = 'Accessibility',         // A11y issues
}

interface DebtItem {
  id: string;
  type: TechnicalDebtType;
  description: string;
  impact: 'Low' | 'Medium' | 'High' | 'Critical';
  effort: 'Small' | 'Medium' | 'Large';
  affectedAreas: string[];
  createdAt: Date;
  estimatedCost: number; // hours
}
```

**Detection Methods:**

```typescript
// 1. Static analysis with ESLint
// .eslintrc.js
module.exports = {
  extends: [
    'eslint:recommended',
    'plugin:react/recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:react-hooks/recommended',
  ],
  rules: {
    'complexity': ['warn', 10],              // Cyclomatic complexity
    'max-lines': ['warn', 300],              // File length
    'max-depth': ['warn', 3],                // Nesting depth
    'no-console': 'warn',
    'react/prop-types': 'off',               // Using TypeScript
    '@typescript-eslint/no-explicit-any': 'error',
  },
};

// 2. Code complexity analysis with SonarQube
const sonarConfig = {
  'sonar.projectKey': 'my-react-app',
  'sonar.sources': 'src',
  'sonar.tests': 'src',
  'sonar.test.inclusions': '**/*.test.tsx,**/*.spec.tsx',
  'sonar.javascript.lcov.reportPaths': 'coverage/lcov.info',
  'sonar.coverage.exclusions': '**/*.test.tsx,**/*.stories.tsx',
};

// 3. Bundle size monitoring
// webpack-bundle-analyzer
import { BundleAnalyzerPlugin } from 'webpack-bundle-analyzer';

export default {
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: 'static',
      openAnalyzer: false,
      reportFilename: 'bundle-report.html',
    }),
  ],
};

// 4. Performance monitoring
const performanceObserver = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (entry.duration > 50) {
      console.warn(`Slow component: ${entry.name} (${entry.duration}ms)`);
      // Log to debt tracker
      trackDebt({
        type: TechnicalDebtType.PERFORMANCE,
        description: `Slow render: ${entry.name}`,
        impact: 'Medium',
      });
    }
  }
});
```

---

#### **2. Prioritization Framework**

**Impact vs. Effort Matrix:**

```typescript
interface DebtPriority {
  score: number;
  priority: 'P0' | 'P1' | 'P2' | 'P3';
  recommendation: string;
}

class TechnicalDebtPrioritizer {
  calculatePriority(debt: DebtItem): DebtPriority {
    const impactScore = this.getImpactScore(debt.impact);
    const effortScore = this.getEffortScore(debt.effort);
    const ageScore = this.getAgeScore(debt.createdAt);
    
    // Priority score: Higher impact, lower effort = higher priority
    const score = (impactScore * 10) + ageScore - effortScore;
    
    if (debt.type === TechnicalDebtType.SECURITY) {
      return { score: 100, priority: 'P0', recommendation: 'Fix immediately' };
    }
    
    if (score > 80) {
      return { score, priority: 'P0', recommendation: 'Fix this sprint' };
    } else if (score > 60) {
      return { score, priority: 'P1', recommendation: 'Fix within 2 sprints' };
    } else if (score > 40) {
      return { score, priority: 'P2', recommendation: 'Plan for next quarter' };
    } else {
      return { score, priority: 'P3', recommendation: 'Backlog' };
    }
  }
  
  private getImpactScore(impact: string): number {
    const scores = { Low: 1, Medium: 3, High: 7, Critical: 10 };
    return scores[impact] || 1;
  }
  
  private getEffortScore(effort: string): number {
    const scores = { Small: 1, Medium: 3, Large: 7 };
    return scores[effort] || 1;
  }
  
  private getAgeScore(createdAt: Date): number {
    const months = (Date.now() - createdAt.getTime()) / (1000 * 60 * 60 * 24 * 30);
    return Math.min(months * 2, 10); // Max 10 points for age
  }
}

// Example usage
const prioritizer = new TechnicalDebtPrioritizer();

const debts: DebtItem[] = [
  {
    id: '1',
    type: TechnicalDebtType.CODE_QUALITY,
    description: 'UserProfile component is 800 lines, needs refactoring',
    impact: 'Medium',
    effort: 'Medium',
    affectedAreas: ['user-profile'],
    createdAt: new Date('2024-06-01'),
    estimatedCost: 16,
  },
  {
    id: '2',
    type: TechnicalDebtType.PERFORMANCE,
    description: 'Dashboard re-renders entire tree on every update',
    impact: 'High',
    effort: 'Small',
    affectedAreas: ['dashboard'],
    createdAt: new Date('2024-01-01'),
    estimatedCost: 4,
  },
  {
    id: '3',
    type: TechnicalDebtType.DEPENDENCIES,
    description: 'React Router v5 needs upgrade to v6',
    impact: 'Medium',
    effort: 'Large',
    affectedAreas: ['routing', 'navigation'],
    createdAt: new Date('2023-10-01'),
    estimatedCost: 40,
  },
];

const prioritized = debts
  .map(debt => ({
    debt,
    priority: prioritizer.calculatePriority(debt),
  }))
  .sort((a, b) => b.priority.score - a.priority.score);

console.log(prioritized);
// Output:
// 1. Dashboard performance (P0) - High impact, small effort
// 2. UserProfile refactor (P1) - Medium impact, medium effort  
// 3. React Router upgrade (P2) - Medium impact, large effort
```

---

#### **3. Systematic Refactoring Approach**

**Example: Breaking Down Large Component**

```typescript
// ❌ Before: 800-line UserProfile component
function UserProfile({ userId }: { userId: string }) {
  // 50+ state variables
  const [profile, setProfile] = useState(null);
  const [orders, setOrders] = useState([]);
  const [reviews, setReviews] = useState([]);
  const [favorites, setFavorites] = useState([]);
  // ... 40+ more states
  
  // 20+ useEffect hooks
  useEffect(() => { /* fetch profile */ }, [userId]);
  useEffect(() => { /* fetch orders */ }, [userId]);
  // ... 18+ more effects
  
  // 30+ event handlers
  const handleUpdateProfile = () => { /* ... */ };
  const handleDeleteAccount = () => { /* ... */ };
  // ... 28+ more handlers
  
  // 500+ lines of JSX
  return (
    <div>
      {/* Massive JSX tree */}
    </div>
  );
}

// ✅ After: Refactored into smaller components
// 1. Main component
function UserProfile({ userId }: { userId: string }) {
  return (
    <div className="user-profile">
      <UserProfileHeader userId={userId} />
      <UserProfileTabs userId={userId} />
    </div>
  );
}

// 2. Extract header
function UserProfileHeader({ userId }: { userId: string }) {
  const { data: profile } = useUserProfile(userId);
  
  if (!profile) return <ProfileSkeleton />;
  
  return (
    <header>
      <UserAvatar src={profile.avatar} />
      <UserInfo profile={profile} />
      <UserActions userId={userId} />
    </header>
  );
}

// 3. Extract tabs
function UserProfileTabs({ userId }: { userId: string }) {
  const [activeTab, setActiveTab] = useState('orders');
  
  return (
    <Tabs value={activeTab} onChange={setActiveTab}>
      <TabList>
        <Tab value="orders">Orders</Tab>
        <Tab value="reviews">Reviews</Tab>
        <Tab value="favorites">Favorites</Tab>
      </TabList>
      
      <TabPanel value="orders">
        <UserOrders userId={userId} />
      </TabPanel>
      <TabPanel value="reviews">
        <UserReviews userId={userId} />
      </TabPanel>
      <TabPanel value="favorites">
        <UserFavorites userId={userId} />
      </TabPanel>
    </Tabs>
  );
}

// 4. Custom hook for data fetching
function useUserProfile(userId: string) {
  return useQuery(['user', userId], () => fetchUserProfile(userId), {
    staleTime: 5 * 60 * 1000,
  });
}
```

**Strangler Fig Pattern for Large Refactors:**

```typescript
// Gradual migration strategy
const RefactorStrategy = {
  week1: 'Extract UserProfileHeader',
  week2: 'Extract UserOrders tab',
  week3: 'Extract UserReviews tab',
  week4: 'Extract UserFavorites tab',
  week5: 'Migrate to custom hooks',
  week6: 'Add tests and cleanup',
};

// Feature flag for gradual rollout
function UserProfileContainer({ userId }: { userId: string }) {
  const useNewProfile = useFeatureFlag('new-user-profile');
  
  if (useNewProfile) {
    return <UserProfileRefactored userId={userId} />;
  }
  
  return <UserProfileLegacy userId={userId} />;
}
```

---

#### **4. Prevention Strategies**

**Code Review Checklist:**

```typescript
const codeReviewChecklist = [
  // Architecture
  '☐ Component is single-responsibility',
  '☐ No prop drilling > 2 levels',
  '☐ Appropriate state management',
  
  // Performance
  '☐ No unnecessary re-renders',
  '☐ Heavy computations are memoized',
  '☐ Large lists are virtualized',
  
  // Code Quality
  '☐ No code duplication',
  '☐ Complex logic is extracted',
  '☐ Meaningful variable names',
  '☐ No magic numbers',
  
  // Testing
  '☐ Unit tests for logic',
  '☐ Integration tests for flows',
  '☐ Edge cases covered',
  
  // Accessibility
  '☐ Semantic HTML',
  '☐ Keyboard navigation',
  '☐ ARIA labels where needed',
  
  // Documentation
  '☐ Complex logic explained',
  '☐ Public APIs documented',
  '☐ README updated if needed',
];
```

**Pre-commit Hooks:**

```json
// package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged",
      "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
    }
  },
  "lint-staged": {
    "*.{ts,tsx}": [
      "eslint --fix",
      "prettier --write",
      "vitest related --run"
    ]
  }
}
```

**Definition of Done:**

```typescript
const definitionOfDone = {
  code: [
    'Code reviewed and approved',
    'No ESLint warnings',
    'TypeScript strict mode passes',
    'No console.log statements',
  ],
  
  testing: [
    'Unit tests written (>80% coverage)',
    'Integration tests for critical paths',
    'Manual testing completed',
    'Accessibility tested',
  ],
  
  documentation: [
    'Code comments for complex logic',
    'Component props documented',
    'README updated if needed',
    'CHANGELOG.md updated',
  ],
  
  performance: [
    'Bundle size impact checked',
    'No performance regressions',
    'Lighthouse score > 90',
  ],
};
```

---

#### **5. Tracking System**

**Technical Debt Register:**

```typescript
// GitHub Issues with labels
const debtIssueTemplate = {
  title: '[Tech Debt] Component refactoring: UserProfile',
  labels: ['tech-debt', 'code-quality', 'P1'],
  body: `
## Description
UserProfile component is 800 lines and handles too many responsibilities.

## Impact
- Difficult to maintain
- Slower development velocity
- High risk of bugs

## Proposed Solution
1. Extract header into UserProfileHeader
2. Extract tabs into separate components
3. Move data fetching to custom hooks

## Effort Estimate
16 hours (2 days)

## Success Criteria
- [ ] Component < 200 lines
- [ ] Test coverage > 80%
- [ ] No regressions in functionality
  `,
};

// Automated debt tracking
class DebtTracker {
  private debts: Map<string, DebtItem> = new Map();
  
  async trackNewDebt(debt: DebtItem) {
    this.debts.set(debt.id, debt);
    
    // Create GitHub issue
    await this.createGitHubIssue(debt);
    
    // Log to analytics
    this.logToAnalytics(debt);
    
    // Notify team
    if (debt.impact === 'Critical') {
      await this.notifyTeam(debt);
    }
  }
  
  generateReport(): DebtReport {
    const debts = Array.from(this.debts.values());
    
    return {
      total: debts.length,
      byType: this.groupBy(debts, 'type'),
      byImpact: this.groupBy(debts, 'impact'),
      totalCost: debts.reduce((sum, d) => sum + d.estimatedCost, 0),
      oldestDebt: this.getOldest(debts),
      recommendations: this.generateRecommendations(debts),
    };
  }
}
```

**Sprint Planning:**

```typescript
const sprintPlanning = {
  capacity: 80, // hours
  allocation: {
    newFeatures: 0.70,  // 70% - 56 hours
    techDebt: 0.20,     // 20% - 16 hours
    bugFixes: 0.10,     // 10% - 8 hours
  },
  
  techDebtBudget: {
    hours: 16,
    stories: [
      { id: 'DEBT-1', title: 'Refactor UserProfile', estimate: 8 },
      { id: 'DEBT-2', title: 'Add tests for Dashboard', estimate: 4 },
      { id: 'DEBT-3', title: 'Update React Router to v6', estimate: 4 },
    ],
  },
};
```

---

#### **6. Communication with Stakeholders**

**Explaining Technical Debt:**

```typescript
const debtMetaphor = {
  explanation: `
    Technical debt is like a credit card:
    - Taking shortcuts = borrowing money
    - Interest = slower development over time
    - Minimum payment = small refactors
    - Paying off principal = major refactors
    - Bankruptcy = complete rewrite
  `,
  
  businessImpact: [
    'Slower feature development (30-50% overhead)',
    'More bugs and customer complaints',
    'Difficulty hiring (poor codebase)',
    'Higher maintenance costs',
    'Risk of system failure',
  ],
};

// Quantify the cost
class DebtCostCalculator {
  calculateSlowdown(debtLevel: 'low' | 'medium' | 'high'): number {
    const slowdownFactors = {
      low: 1.1,      // 10% slower
      medium: 1.3,   // 30% slower
      high: 1.5,     // 50% slower
    };
    return slowdownFactors[debtLevel];
  }
  
  estimateRefactorROI(debt: DebtItem): ROI {
    const currentSlowdown = 1.3; // 30% slower
    const afterRefactor = 1.0;   // Normal speed
    
    const weeklyDevHours = 160;  // 4 devs * 40 hours
    const hourlyRate = 100;      // $100/hour
    
    const weeklyCost = weeklyDevHours * hourlyRate * (currentSlowdown - 1);
    const refactorCost = debt.estimatedCost * hourlyRate;
    const breakEvenWeeks = refactorCost / weeklyCost;
    
    return {
      refactorCost,
      weeklySavings: weeklyCost,
      breakEvenWeeks,
      yearOneSavings: (52 * weeklyCost) - refactorCost,
    };
  }
}

const roi = new DebtCostCalculator().estimateRefactorROI({
  estimatedCost: 40, // hours
  // ... other fields
});

console.log(roi);
// {
//   refactorCost: $4,000,
//   weeklySavings: $4,800,
//   breakEvenWeeks: 0.83 weeks,
//   yearOneSavings: $245,600
// }
```

---

#### **7. Real-World Example**

**Scenario: State Management Refactor**

```typescript
// Problem: Prop drilling hell
function App() {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  const [cart, setCart] = useState([]);
  
  return (
    <Layout user={user} theme={theme}>
      <Header user={user} cart={cart} setCart={setCart} />
      <Main user={user} theme={theme}>
        <ProductList cart={cart} setCart={setCart} user={user} />
      </Main>
    </Layout>
  );
}

// Solution: Context + Custom Hooks
const UserContext = createContext(null);
const ThemeContext = createContext('light');
const CartContext = createContext(null);

function App() {
  return (
    <UserProvider>
      <ThemeProvider>
        <CartProvider>
          <Layout>
            <Header />
            <Main>
              <ProductList />
            </Main>
          </Layout>
        </CartProvider>
      </ThemeProvider>
    </UserProvider>
  );
}

// Usage in components
function ProductList() {
  const { user } = useUser();
  const { cart, addToCart } = useCart();
  const { theme } = useTheme();
  
  // No prop drilling!
}
```

---

#### **Summary**

**Key Principles:**
1. **Acknowledge**: Technical debt is normal
2. **Measure**: Track and quantify debt
3. **Prioritize**: Impact vs. effort
4. **Budget**: Allocate 15-20% of sprint capacity
5. **Prevent**: Code reviews, standards, automation
6. **Communicate**: Explain business impact
7. **Iterate**: Small, continuous improvements

**Red Flags to Watch:**
- Components > 300 lines
- Cyclomatic complexity > 10
- Test coverage < 70%
- Bundle size growing significantly
- Velocity decreasing sprint over sprint
- Increasing bug count
- Developers avoiding certain areas

**Balance:**
- Ship features vs. perfect code
- Technical excellence vs. business needs
- Short-term speed vs. long-term velocity
- Pragmatic debt vs. reckless debt

Technical debt is a **strategic tool** when managed properly, not just a problem to avoid.

</details>

---

### 264. Describe a time you optimized a slow React application

<details>
<summary>View Answer</summary>

**Performance optimization** requires a **systematic approach**: measure first, identify bottlenecks, implement targeted fixes, and validate improvements.

---

#### **Case Study: E-commerce Dashboard Optimization**

**Situation:**
- Admin dashboard for e-commerce platform
- 50+ concurrent users
- Performance degraded over 6 months of feature additions
- **Key issues:**
  - Initial load: 12 seconds
  - Time to Interactive (TTI): 18 seconds
  - Dashboard page: 8-10 second render
  - Frequent browser freezes
  - Customer complaints increasing

**Task:**
- Reduce initial load time to < 3 seconds
- Achieve 60fps scrolling performance
- Support 10,000+ rows in data tables
- Fix memory leaks causing crashes
- Timeline: 3 weeks

---

#### **Action: Step-by-Step Optimization**

### **Phase 1: Measurement & Profiling**

```typescript
// 1. React DevTools Profiler
import { Profiler } from 'react';

function Dashboard() {
  const onRenderCallback = (
    id: string,
    phase: 'mount' | 'update',
    actualDuration: number,
  ) => {
    if (actualDuration > 16) { // More than one frame (16ms at 60fps)
      console.warn(`Slow ${phase}: ${id} took ${actualDuration}ms`);
      
      // Send to analytics
      analytics.track('slow_render', {
        component: id,
        duration: actualDuration,
        phase,
      });
    }
  };
  
  return (
    <Profiler id="Dashboard" onRender={onRenderCallback}>
      <DashboardContent />
    </Profiler>
  );
}

// 2. Chrome DevTools Performance tab
// Identified issues:
// - Long tasks (>50ms)
// - Layout thrashing
// - Memory leaks

// 3. Lighthouse audit
const initialScores = {
  performance: 32,
  firstContentfulPaint: 4200, // ms
  timeToInteractive: 18000,   // ms
  totalBlockingTime: 3400,    // ms
  largestContentfulPaint: 8900, // ms
};

// 4. Bundle analysis
import { BundleAnalyzerPlugin } from 'webpack-bundle-analyzer';

// Findings:
// - moment.js: 288KB (only using 5% of features)
// - lodash: 72KB (importing entire library)
// - chart.js: 180KB (not code-split)
// - Total bundle: 2.4MB
```

---

### **Phase 2: Identified Bottlenecks**

```typescript
const bottlenecks = [
  {
    issue: 'Massive re-renders',
    cause: 'Dashboard re-renders on every Redux action',
    component: 'Dashboard.tsx',
    impact: 'High',
  },
  {
    issue: 'Rendering 5000 rows directly',
    cause: 'No virtualization in DataTable',
    component: 'DataTable.tsx',
    impact: 'Critical',
  },
  {
    issue: 'Large bundle size',
    cause: 'Heavy dependencies, no code splitting',
    impact: 'Critical',
  },
  {
    issue: 'Memory leak',
    cause: 'WebSocket connection not cleaned up',
    component: 'RealtimeUpdates.tsx',
    impact: 'High',
  },
  {
    issue: 'Expensive computations',
    cause: 'Filtering/sorting on every render',
    component: 'ProductList.tsx',
    impact: 'Medium',
  },
];
```

---

### **Phase 3: Optimizations Implemented**

#### **1. Fixed Unnecessary Re-renders**

```typescript
// ❌ Before: Dashboard re-renders on ANY Redux state change
function Dashboard() {
  const state = useSelector(state => state); // Subscribes to entire state!
  
  return (
    <div>
      <Sidebar />
      <MainContent data={state.products} />
      <RightPanel stats={state.stats} />
    </div>
  );
}

// ✅ After: Select only needed data
function Dashboard() {
  const products = useSelector(
    (state) => state.products,
    (a, b) => a.length === b.length && a[0]?.id === b[0]?.id // Custom equality
  );
  const stats = useSelector((state) => state.stats);
  
  return (
    <div>
      <Sidebar />
      <MainContent data={products} />
      <RightPanel stats={stats} />
    </div>
  );
}

// ✅ Better: React.memo for expensive components
const Sidebar = React.memo(function Sidebar() {
  // Only re-renders when props change
  return <aside>{/* ... */}</aside>;
});

const MainContent = React.memo(
  function MainContent({ data }: { data: Product[] }) {
    return <main>{/* ... */}</main>;
  },
  (prevProps, nextProps) => {
    // Custom comparison
    return prevProps.data.length === nextProps.data.length;
  }
);
```

#### **2. Virtualized Large Lists**

```typescript
// ❌ Before: Rendering 5000 rows directly
function DataTable({ data }: { data: Product[] }) {
  return (
    <table>
      <tbody>
        {data.map(product => (
          <TableRow key={product.id} product={product} />
        ))}
      </tbody>
    </table>
  );
}
// Result: 8-10 second render, browser freeze

// ✅ After: Virtual scrolling with react-window
import { FixedSizeList as List } from 'react-window';
import AutoSizer from 'react-virtualized-auto-sizer';

function DataTable({ data }: { data: Product[] }) {
  const Row = ({ index, style }: { index: number; style: React.CSSProperties }) => (
    <div style={style}>
      <TableRow product={data[index]} />
    </div>
  );
  
  return (
    <AutoSizer>
      {({ height, width }) => (
        <List
          height={height}
          itemCount={data.length}
          itemSize={50}
          width={width}
          overscanCount={5} // Render 5 extra rows for smooth scrolling
        >
          {Row}
        </List>
      )}
    </AutoSizer>
  );
}
// Result: <100ms render, smooth 60fps scrolling
```

#### **3. Code Splitting**

```typescript
// ❌ Before: Everything in main bundle
import Dashboard from './pages/Dashboard';
import Reports from './pages/Reports';
import Settings from './pages/Settings';
import ChartComponent from './components/Chart';

// ✅ After: Route-based code splitting
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Reports = lazy(() => import('./pages/Reports'));
const Settings = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <Suspense fallback={<PageLoader />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/reports" element={<Reports />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}

// ✅ Component-level code splitting for heavy components
function Dashboard() {
  const [showChart, setShowChart] = useState(false);
  
  const ChartComponent = useMemo(
    () => lazy(() => import('./components/Chart')),
    []
  );
  
  return (
    <div>
      <button onClick={() => setShowChart(true)}>Show Chart</button>
      {showChart && (
        <Suspense fallback={<div>Loading chart...</div>}>
          <ChartComponent data={chartData} />
        </Suspense>
      )}
    </div>
  );
}
```

#### **4. Bundle Optimization**

```typescript
// ❌ Before: Heavy dependencies
import moment from 'moment';      // 288KB
import _ from 'lodash';           // 72KB
import Chart from 'chart.js';     // 180KB

const formatted = moment(date).format('YYYY-MM-DD');
const sorted = _.sortBy(array, 'name');

// ✅ After: Lightweight alternatives
import { format } from 'date-fns';  // 13KB (tree-shakeable)
import sortBy from 'lodash-es/sortBy'; // 5KB (individual import)
import { Chart } from 'react-chartjs-2'; // Lazy loaded

const formatted = format(date, 'yyyy-MM-dd');
const sorted = sortBy(array, 'name');

// Webpack config for tree shaking
export default {
  optimization: {
    usedExports: true,
    sideEffects: false,
  },
};

// Result:
// Bundle size: 2.4MB → 680KB (72% reduction)
// Initial load: 12s → 2.8s
```

#### **5. Memoization**

```typescript
// ❌ Before: Expensive computation on every render
function ProductList({ products, searchTerm, filters }) {
  // Runs on EVERY render, even when products haven't changed!
  const filtered = products
    .filter(p => p.name.includes(searchTerm))
    .filter(p => filters.category ? p.category === filters.category : true)
    .sort((a, b) => b.sales - a.sales);
  
  const stats = {
    total: filtered.length,
    totalRevenue: filtered.reduce((sum, p) => sum + p.revenue, 0),
    avgPrice: filtered.reduce((sum, p) => sum + p.price, 0) / filtered.length,
  };
  
  return (
    <div>
      <Stats data={stats} />
      {filtered.map(p => <ProductCard key={p.id} product={p} />)}
    </div>
  );
}

// ✅ After: useMemo for expensive computations
function ProductList({ products, searchTerm, filters }) {
  const filtered = useMemo(() => {
    return products
      .filter(p => p.name.toLowerCase().includes(searchTerm.toLowerCase()))
      .filter(p => filters.category ? p.category === filters.category : true)
      .sort((a, b) => b.sales - a.sales);
  }, [products, searchTerm, filters]);
  
  const stats = useMemo(() => ({
    total: filtered.length,
    totalRevenue: filtered.reduce((sum, p) => sum + p.revenue, 0),
    avgPrice: filtered.reduce((sum, p) => sum + p.price, 0) / filtered.length,
  }), [filtered]);
  
  return (
    <div>
      <Stats data={stats} />
      {filtered.map(p => <ProductCard key={p.id} product={p} />)}
    </div>
  );
}

// Result: 450ms → 12ms for filtering/sorting
```

#### **6. Fixed Memory Leak**

```typescript
// ❌ Before: WebSocket not cleaned up
function RealtimeUpdates() {
  const [updates, setUpdates] = useState([]);
  
  useEffect(() => {
    const ws = new WebSocket('wss://api.example.com/updates');
    
    ws.onmessage = (event) => {
      setUpdates(prev => [...prev, JSON.parse(event.data)]);
    };
    
    // ❌ No cleanup! Memory leak on unmount
  }, []);
  
  return <UpdatesList updates={updates} />;
}
// Result: Memory grows from 50MB to 800MB+ over 2 hours

// ✅ After: Proper cleanup
function RealtimeUpdates() {
  const [updates, setUpdates] = useState([]);
  
  useEffect(() => {
    const ws = new WebSocket('wss://api.example.com/updates');
    
    ws.onmessage = (event) => {
      setUpdates(prev => {
        // Also limit array size to prevent unbounded growth
        const newUpdates = [...prev, JSON.parse(event.data)];
        return newUpdates.slice(-100); // Keep only last 100
      });
    };
    
    // ✅ Cleanup function
    return () => {
      ws.close();
    };
  }, []);
  
  return <UpdatesList updates={updates} />;
}
// Result: Stable memory usage ~80MB
```

#### **7. Debouncing & Throttling**

```typescript
// ❌ Before: API call on every keystroke
function SearchBar() {
  const [query, setQuery] = useState('');
  
  const handleChange = (e: ChangeEvent<HTMLInputElement>) => {
    setQuery(e.target.value);
    fetchResults(e.target.value); // API call!
  };
  
  return <input value={query} onChange={handleChange} />;
}
// Result: 50+ API calls when typing "react performance"

// ✅ After: Debounced search
import { useDebouncedCallback } from 'use-debounce';

function SearchBar() {
  const [query, setQuery] = useState('');
  
  const debouncedSearch = useDebouncedCallback(
    (value: string) => {
      fetchResults(value);
    },
    500 // Wait 500ms after user stops typing
  );
  
  const handleChange = (e: ChangeEvent<HTMLInputElement>) => {
    const value = e.target.value;
    setQuery(value);
    debouncedSearch(value);
  };
  
  return <input value={query} onChange={handleChange} />;
}
// Result: 1 API call for "react performance"
```

---

#### **Result: Dramatic Improvements**

**Performance Metrics:**

```typescript
const results = {
  before: {
    initialLoad: 12000,        // ms
    timeToInteractive: 18000,  // ms
    dashboardRender: 8500,     // ms
    bundleSize: 2400,          // KB
    lighthouseScore: 32,
    memoryUsage: 800,          // MB after 2 hours
  },
  
  after: {
    initialLoad: 2800,         // ms - 77% faster ✅
    timeToInteractive: 4200,   // ms - 77% faster ✅
    dashboardRender: 120,      // ms - 99% faster ✅
    bundleSize: 680,           // KB - 72% smaller ✅
    lighthouseScore: 94,       // +62 points ✅
    memoryUsage: 95,           // MB - 88% less ✅
  },
  
  improvements: {
    initialLoad: '-77%',
    timeToInteractive: '-77%',
    dashboardRender: '-99%',
    bundleSize: '-72%',
    lighthouseScore: '+194%',
    memoryUsage: '-88%',
  },
};
```

**Business Impact:**
- **User satisfaction**: Complaints dropped 95%
- **Bounce rate**: 45% → 12%
- **Task completion**: +38%
- **Support tickets**: -60%
- **Mobile users**: +120% (now usable on mobile)
- **Retention**: 3 major clients considering cancellation stayed

---

#### **Key Learnings**

**1. Always Measure First:**
```typescript
const optimizationProcess = [
  '1. Profile and measure',
  '2. Identify bottlenecks',
  '3. Prioritize by impact',
  '4. Implement targeted fixes',
  '5. Measure again',
  '6. Iterate',
];
// Don't optimize blindly!
```

**2. Low-Hanging Fruit:**
- React.memo for expensive components
- useMemo/useCallback for expensive computations
- Virtual scrolling for large lists
- Code splitting by route
- Replace heavy dependencies

**3. Tools Used:**
- React DevTools Profiler
- Chrome DevTools Performance tab
- Lighthouse
- webpack-bundle-analyzer
- React Query DevTools
- why-did-you-render

**4. Common Pitfalls:**
- Premature optimization
- Optimizing without measuring
- Forgetting cleanup in useEffect
- Over-using useMemo (has its own cost)
- Not considering bundle size

**5. Systematic Approach:**
```typescript
const performanceChecklist = [
  '☐ Profile application',
  '☐ Check bundle size',
  '☐ Audit dependencies',
  '☐ Review component structure',
  '☐ Check for memory leaks',
  '☐ Optimize critical path',
  '☐ Add performance monitoring',
  '☐ Set performance budgets',
];
```

---

#### **Interview Tips**

1. **Show Methodology**: Explain your systematic approach
2. **Use Metrics**: Provide before/after numbers
3. **Explain Trade-offs**: Discuss what you didn't do and why
4. **Show Business Impact**: Connect to user experience/revenue
5. **Mention Tools**: Show familiarity with profiling tools
6. **Be Specific**: Use concrete examples, not generalities
7. **Show Learning**: What would you do differently?

**Story Structure:**
- **Context**: What was slow and why it mattered
- **Analysis**: How you identified the problems
- **Solution**: Specific optimizations implemented
- **Results**: Measurable improvements
- **Learning**: Key takeaways

</details>

---

### 265. How do you review code from other React developers?

<details>
<summary>View Answer</summary>

**Code reviews** are crucial for **maintaining quality**, **knowledge sharing**, and **team growth**. A good review balances **thoroughness** with **empathy** and **constructive feedback**.

---

#### **1. Review Framework**

**My Code Review Process:**

```typescript
const reviewProcess = [
  {
    step: 1,
    name: 'Initial Assessment',
    duration: '2-3 minutes',
    actions: [
      'Read PR description',
      'Check ticket/issue link',
      'Understand the context',
      'Review test plan',
      'Check file changes count',
    ],
  },
  {
    step: 2,
    name: 'Automated Checks',
    duration: '1 minute',
    actions: [
      'Verify CI/CD passes',
      'Check test coverage',
      'Review lint/type errors',
      'Check bundle size impact',
    ],
  },
  {
    step: 3,
    name: 'Code Review',
    duration: '15-30 minutes',
    actions: [
      'Review architecture',
      'Check code quality',
      'Verify best practices',
      'Look for edge cases',
      'Review tests',
    ],
  },
  {
    step: 4,
    name: 'Testing',
    duration: '5-10 minutes',
    actions: [
      'Pull branch locally',
      'Test functionality',
      'Check edge cases',
      'Test on different browsers',
    ],
  },
  {
    step: 5,
    name: 'Feedback',
    duration: '5-10 minutes',
    actions: [
      'Write constructive comments',
      'Approve or request changes',
      'Offer to pair if needed',
    ],
  },
];
```

---

#### **2. Review Checklist**

**Architecture & Design:**

```typescript
const architectureChecks = [
  // Component Design
  '☐ Single Responsibility Principle followed',
  '☐ Component is appropriately sized (<300 lines)',
  '☐ Logic extracted to custom hooks where appropriate',
  '☐ No prop drilling >2 levels',
  '☐ Composition over inheritance',
  
  // State Management
  '☐ State is co-located appropriately',
  '☐ No unnecessary global state',
  '☐ Derived state is computed, not stored',
  '☐ State updates are immutable',
  
  // Data Flow
  '☐ One-way data flow maintained',
  '☐ Side effects properly handled',
  '☐ No direct DOM manipulation',
];
```

**Code Quality:**

```typescript
const codeQualityChecks = [
  // Readability
  '☐ Variable names are descriptive',
  '☐ No magic numbers/strings',
  '☐ Complex logic has comments',
  '☐ Functions are pure when possible',
  
  // React Best Practices
  '☐ Keys are stable and unique',
  '☐ useEffect dependencies are correct',
  '☐ No eslint-disable without explanation',
  '☐ Hooks follow rules of hooks',
  '☐ Event handlers properly named (handleX)',
  
  // Performance
  '☐ No unnecessary re-renders',
  '☐ Heavy computations are memoized',
  '☐ Large lists are virtualized',
  '☐ Images are optimized',
  '☐ Code splitting where appropriate',
  
  // Error Handling
  '☐ Error boundaries implemented',
  '☐ Loading states handled',
  '☐ Error states handled',
  '☐ Edge cases considered',
];
```

**Testing:**

```typescript
const testingChecks = [
  '☐ Unit tests for logic',
  '☐ Integration tests for flows',
  '☐ Tests are readable',
  '☐ Tests cover edge cases',
  '☐ No flaky tests',
  '☐ Test coverage >80%',
  '☐ Tests follow AAA pattern (Arrange, Act, Assert)',
];
```

**Accessibility:**

```typescript
const a11yChecks = [
  '☐ Semantic HTML used',
  '☐ ARIA labels where needed',
  '☐ Keyboard navigation works',
  '☐ Focus management proper',
  '☐ Color contrast sufficient',
  '☐ Screen reader tested',
];
```

**Security:**

```typescript
const securityChecks = [
  '☐ No hardcoded secrets',
  '☐ User input sanitized',
  '☐ XSS prevention (dangerouslySetInnerHTML avoided)',
  '☐ CSRF protection in place',
  '☐ Dependencies have no vulnerabilities',
];
```

---

#### **3. Feedback Strategy**

**Comment Types:**

```typescript
type CommentType = 'nitpick' | 'suggestion' | 'question' | 'issue' | 'blocker';

interface ReviewComment {
  type: CommentType;
  message: string;
  severity: 'low' | 'medium' | 'high';
  canMerge: boolean;
}

const commentExamples = [
  {
    type: 'nitpick',
    message: '**Nitpick:** Consider using a more descriptive variable name here. `data` is too generic.',
    severity: 'low',
    canMerge: true,
  },
  {
    type: 'suggestion',
    message: '**Suggestion:** You could simplify this with optional chaining: `user?.profile?.name`',
    severity: 'low',
    canMerge: true,
  },
  {
    type: 'question',
    message: '**Question:** Why did we choose to store this in state instead of deriving it? Just want to understand the reasoning.',
    severity: 'medium',
    canMerge: true,
  },
  {
    type: 'issue',
    message: '**Issue:** This will cause a memory leak. The WebSocket connection needs to be cleaned up in useEffect return.',
    severity: 'high',
    canMerge: false,
  },
  {
    type: 'blocker',
    message: '**Blocker:** This exposes user passwords in the API response. This must be fixed before merging.',
    severity: 'high',
    canMerge: false,
  },
];
```

**Constructive Feedback Framework:**

```typescript
const feedbackPrinciples = {
  // 1. Be specific
  bad: '❌ This code is bad',
  good: '✅ This function does too much. Consider extracting the validation logic into a separate function.',
  
  // 2. Explain why
  bad: '❌ Don\'t use any here',
  good: '✅ Using `any` bypasses TypeScript\'s type checking. Let\'s define a proper interface for this data.',
  
  // 3. Offer solutions
  bad: '❌ This is inefficient',
  good: '✅ This filters the array on every render. Consider using useMemo here:\n```ts\nconst filtered = useMemo(() => items.filter(...), [items]);\n```',
  
  // 4. Ask questions
  bad: '❌ This is wrong',
  good: '✅ I\'m curious about this approach. Have you considered using React Query for this data fetching? It handles caching automatically.',
  
  // 5. Praise good work
  good: '✅ Nice! I like how you extracted this into a custom hook. Makes it very reusable.',
};
```

**Example Review Comments:**

```typescript
// ✅ Good review comments
const goodComments = [
  // Specific and actionable
  `This effect will run on every render because it's missing dependencies.
  
  Add [userId, fetchUser] to the dependency array:
  
  \`\`\`typescript
  useEffect(() => {
    fetchUser(userId);
  }, [userId, fetchUser]);
  \`\`\`
  
  Or wrap fetchUser in useCallback if it's defined in the component.`,
  
  // Educational
  `Good use of React.memo! 👍
  
  One optimization: you can provide a custom comparison function as the second argument if you only want to re-render when specific props change:
  
  \`\`\`typescript
  export default React.memo(UserCard, (prev, next) => {
    return prev.user.id === next.user.id;
  });
  \`\`\``,
  
  // Collaborative
  `I see you're managing this complex state with multiple useStates. Have you considered using useReducer? It might make the state transitions clearer.
  
  Happy to pair on this if you'd like!`,
];

// ❌ Bad review comments
const badComments = [
  'This is wrong',
  'You should know better',
  'We never do it this way',
  'Just use X library',
  'This is too complicated',
];
```

---

#### **4. Common Issues & How I Address Them**

**Issue 1: Unnecessary Re-renders**

```typescript
// Code being reviewed:
function ParentComponent() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <ExpensiveChild data={someData} />
    </div>
  );
}

// My review comment:
/*
**Issue:** ExpensiveChild will re-render every time count changes,
even though its props haven't changed.

**Fix:** Wrap it in React.memo:

```typescript
const ExpensiveChild = React.memo(({ data }) => {
  // ... component code
});
```

This will prevent re-renders when parent updates but props stay the same.

**Why this matters:** For expensive components, unnecessary re-renders
can cause performance issues.
*/
```

**Issue 2: Incorrect Dependencies**

```typescript
// Code being reviewed:
function SearchComponent({ userId }) {
  const [results, setResults] = useState([]);
  
  useEffect(() => {
    fetchResults(userId).then(setResults);
  }, []); // ⚠️ Missing userId dependency
}

// My review comment:
/*
**Blocker:** Missing dependency in useEffect.

**Problem:** If userId changes, the effect won't re-run, and you'll
show stale results.

**Fix:**
```typescript
useEffect(() => {
  fetchResults(userId).then(setResults);
}, [userId]); // ✅ Add userId
```

**Also consider:** Adding cleanup for race conditions:
```typescript
useEffect(() => {
  let cancelled = false;
  
  fetchResults(userId).then(data => {
    if (!cancelled) setResults(data);
  });
  
  return () => { cancelled = true; };
}, [userId]);
```
*/
```

**Issue 3: Prop Drilling**

```typescript
// Code being reviewed:
function App() {
  const [user, setUser] = useState(null);
  return <Layout user={user}>
    <Header user={user}>
      <UserMenu user={user}>
        <UserAvatar user={user} />
      </UserMenu>
    </Header>
  </Layout>;
}

// My review comment:
/*
**Suggestion:** This is a good example of prop drilling.

Consider using Context for widely-used data like user:

```typescript
// UserContext.tsx
const UserContext = createContext(null);

export const UserProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  return (
    <UserContext.Provider value={{ user, setUser }}>
      {children}
    </UserContext.Provider>
  );
};

export const useUser = () => useContext(UserContext);

// Usage in any component:
function UserAvatar() {
  const { user } = useUser();
  // No prop drilling!
}
```

This makes the code cleaner and components more independent.
*/
```

**Issue 4: Not Handling Loading/Error States**

```typescript
// Code being reviewed:
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);
  
  return <div>{user.name}</div>; // ⚠️ Will crash if user is null
}

// My review comment:
/*
**Issue:** Missing loading and error states.

**Problems:**
1. Will crash on first render (user is null)
2. No feedback while loading
3. No error handling if fetch fails

**Fix:**
```typescript
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    setLoading(true);
    fetchUser(userId)
      .then(setUser)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [userId]);
  
  if (loading) return <Spinner />;
  if (error) return <Error message={error.message} />;
  if (!user) return <NotFound />;
  
  return <div>{user.name}</div>;
}
```

**Even better:** Use React Query which handles this automatically:
```typescript
function UserProfile({ userId }) {
  const { data: user, isLoading, error } = useQuery(
    ['user', userId],
    () => fetchUser(userId)
  );
  
  if (isLoading) return <Spinner />;
  if (error) return <Error message={error.message} />;
  
  return <div>{user.name}</div>;
}
```
*/
```

---

#### **5. Automated Review Tools**

**Setup:**

```typescript
// .github/workflows/pr-review.yml
name: PR Review Automation

on: [pull_request]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      # 1. Lint
      - name: Run ESLint
        run: npm run lint
      
      # 2. Type check
      - name: TypeScript Check
        run: npm run type-check
      
      # 3. Tests
      - name: Run Tests
        run: npm run test -- --coverage
      
      # 4. Bundle size check
      - name: Bundle Size
        uses: andresz1/size-limit-action@v1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
      
      # 5. Lighthouse CI
      - name: Lighthouse
        uses: treosh/lighthouse-ci-action@v9
        with:
          urls: |
            http://localhost:3000
          uploadArtifacts: true
      
      # 6. Code quality
      - name: SonarCloud Scan
        uses: SonarSource/sonarcloud-github-action@master
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

**Danger JS for automated comments:**

```typescript
// dangerfile.ts
import { danger, warn, fail, message } from 'danger';

// Check PR size
const bigPRThreshold = 500;
if (danger.git.lines_of_code > bigPRThreshold) {
  warn(`This PR has ${danger.git.lines_of_code} lines. Consider breaking it into smaller PRs.`);
}

// Require PR description
if (danger.github.pr.body.length < 10) {
  fail('Please add a description to your PR.');
}

// Check for tests
const hasAppChanges = danger.git.modified_files.some(f => f.includes('src/'));
const hasTestChanges = danger.git.modified_files.some(f => f.includes('.test.'));

if (hasAppChanges && !hasTestChanges) {
  warn('This PR modifies app code but doesn\'t include tests.');
}

// Check for console.log
const jsFiles = danger.git.created_files.filter(f => f.endsWith('.ts') || f.endsWith('.tsx'));
for (const file of jsFiles) {
  const content = await danger.github.utils.fileContents(file);
  if (content.includes('console.log')) {
    warn(`${file} contains console.log statements.`);
  }
}

// Celebrate good practices
if (hasTestChanges) {
  message('Thanks for adding tests! 🎉');
}
```

---

#### **6. Review Etiquette**

```typescript
const reviewEtiquette = {
  timing: {
    respondWithin: '24 hours',
    blockingIssues: 'Comment within 4 hours',
    finalApproval: 'Within 1 business day',
  },
  
  tone: {
    principles: [
      'Assume good intent',
      'Be humble ("I think" vs "You should")',
      'Ask questions, don\'t demand',
      'Praise good work',
      'Criticize code, not people',
    ],
    
    examples: {
      bad: '❌ This is terrible. Why would you do this?',
      good: '✅ I\'m concerned about the performance here. What do you think about trying approach X?',
    },
  },
  
  scope: {
    inScope: [
      'Code quality',
      'Architecture',
      'Performance',
      'Security',
      'Tests',
      'Accessibility',
    ],
    outOfScope: [
      'Personal coding style (if follows team standards)',
      'Bike-shedding (arguing over trivial details)',
      'Scope creep ("while you\'re here, also...")',
    ],
  },
  
  approval: {
    approveIf: [
      'No blocking issues',
      'Tests pass',
      'Code quality acceptable',
      'Minor issues can be addressed later',
    ],
    requestChangesIf: [
      'Security vulnerabilities',
      'Major bugs',
      'Breaks existing functionality',
      'Missing critical tests',
    ],
  },
};
```

---

#### **7. Handling Disagreements**

```typescript
const disagreementResolution = [
  {
    scenario: 'Subjective preference',
    resolution: 'Defer to team standards. If no standard, author\'s choice wins.',
    example: 'Named exports vs default exports',
  },
  {
    scenario: 'Performance concern',
    resolution: 'Measure, don\'t guess. Profile before optimizing.',
    example: 'Debating if component needs React.memo',
  },
  {
    scenario: 'Architecture debate',
    resolution: 'Discuss synchronously (call or meeting). Document decision.',
    example: 'Context vs prop drilling vs state management library',
  },
  {
    scenario: 'Time pressure',
    resolution: 'Create tech debt ticket for future improvement.',
    example: 'Quick fix needed, refactor can wait',
  },
];
```

---

#### **Summary**

**My Code Review Philosophy:**

1. **Thorough but timely**: Complete reviews within 24 hours
2. **Educational**: Explain the "why", not just the "what"
3. **Collaborative**: Offer to pair on complex issues
4. **Balanced**: Praise good work, not just point out issues
5. **Objective**: Focus on code quality, not personal preferences
6. **Empathetic**: Remember we're all learning

**Key Questions I Ask:**
- Is it correct?
- Is it performant?
- Is it maintainable?
- Is it testable?
- Is it accessible?
- Is it secure?

**Goals of Code Review:**
- Catch bugs and issues
- Maintain code quality
- Share knowledge
- Enforce standards
- Learn from each other
- Build team culture

Good code reviews make the **codebase better** and the **team stronger**.

</details>

---

### 266. How do you mentor junior React developers?

<details>
<summary>View Answer</summary>

**Mentoring junior developers** is about **empowering growth**, **building confidence**, and **fostering independent problem-solving** while maintaining **code quality** and **team velocity**.

---

#### **1. Mentoring Philosophy**

```typescript
const mentoringPrinciples = {
  // Core beliefs
  beliefs: [
    'Everyone was a beginner once',
    'Mistakes are learning opportunities',
    'Questions are signs of growth, not weakness',
    'Independent problem-solving > quick answers',
    'Confidence comes from small wins',
  ],
  
  // Goals
  goals: [
    'Build technical competence',
    'Develop problem-solving skills',
    'Foster independence',
    'Grow confidence',
    'Integrate into team',
  ],
  
  // Approach
  approach: {
    teaching: 'Socratic method (guide, don\'t tell)',
    feedback: 'Frequent, specific, and constructive',
    difficulty: 'Progressive (start easy, increase complexity)',
    support: 'Available but not hovering',
  },
};
```

---

#### **2. Structured Learning Path**

**Week 1-2: Foundations**

```typescript
const week1_2 = {
  focus: 'Environment setup and React basics',
  
  tasks: [
    {
      task: 'Setup development environment',
      deliverable: 'Running React app locally',
      resources: ['Official React docs', 'Team setup guide'],
      support: 'Pair on setup, troubleshoot together',
    },
    {
      task: 'Build a simple component',
      deliverable: 'UserCard component with props',
      example: `
function UserCard({ name, email, avatar }) {
  return (
    <div className="user-card">
      <img src={avatar} alt={name} />
      <h3>{name}</h3>
      <p>{email}</p>
    </div>
  );
}`,
      learningGoals: [
        'JSX syntax',
        'Props',
        'Component structure',
      ],
    },
    {
      task: 'Add interactivity with state',
      deliverable: 'Counter component',
      example: `
function Counter() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}`,
      learningGoals: ['useState', 'Event handlers', 'Re-rendering'],
    },
  ],
  
  checkIn: {
    frequency: 'Daily',
    format: '15-minute standup',
    questions: [
      'What did you learn yesterday?',
      'What are you working on today?',
      'Any blockers?',
    ],
  },
};
```

**Week 3-4: Forms & Side Effects**

```typescript
const week3_4 = {
  focus: 'Forms, API calls, and useEffect',
  
  tasks: [
    {
      task: 'Build a form component',
      deliverable: 'Controlled form with validation',
      example: `
function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [errors, setErrors] = useState({});
  
  const handleSubmit = (e) => {
    e.preventDefault();
    
    const newErrors = {};
    if (!email) newErrors.email = 'Email required';
    if (password.length < 8) newErrors.password = 'Min 8 characters';
    
    if (Object.keys(newErrors).length > 0) {
      setErrors(newErrors);
      return;
    }
    
    // Submit...
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input 
        value={email} 
        onChange={(e) => setEmail(e.target.value)}
      />
      {errors.email && <span>{errors.email}</span>}
      {/* ... */}
    </form>
  );
}`,
      learningGoals: [
        'Controlled components',
        'Form validation',
        'Error handling',
      ],
    },
    {
      task: 'Fetch data from API',
      deliverable: 'UserList component',
      example: `
function UserList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetch('/api/users')
      .then(res => res.json())
      .then(setUsers)
      .catch(setError)
      .finally(() => setLoading(false));
  }, []);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return (
    <ul>
      {users.map(user => <li key={user.id}>{user.name}</li>)}
    </ul>
  );
}`,
      learningGoals: [
        'useEffect',
        'Data fetching',
        'Loading/error states',
        'Async operations',
      ],
    },
  ],
};
```

**Week 5-8: Advanced Patterns**

```typescript
const week5_8 = {
  focus: 'Custom hooks, context, performance',
  
  tasks: [
    {
      task: 'Create custom hook',
      deliverable: 'useApi hook',
      example: `
function useApi(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    setLoading(true);
    fetch(url)
      .then(res => res.json())
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [url]);
  
  return { data, loading, error };
}

// Usage
function Users() {
  const { data: users, loading, error } = useApi('/api/users');
  // ...
}`,
      learningGoals: [
        'Custom hooks',
        'Code reuse',
        'Abstraction',
      ],
    },
    {
      task: 'Implement Context API',
      deliverable: 'Theme context',
      learningGoals: [
        'Context API',
        'Global state',
        'Provider pattern',
      ],
    },
    {
      task: 'Optimize performance',
      deliverable: 'Use React.memo and useMemo',
      learningGoals: [
        'Performance optimization',
        'Memoization',
        'Profiling',
      ],
    },
  ],
  
  milestone: 'Ship first feature independently',
};
```

---

#### **3. Teaching Through Code Review**

**Educational Code Review:**

```typescript
// Junior's code:
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => setUser(data));
  }, []);
  
  return <div>{user?.name}</div>;
}

// My review comment:
/*
Great start! The component works, and I like that you're using 
optional chaining (?.) to prevent errors. 👍

I have a few suggestions to make this more robust:

1. **Add userId to dependencies**:
   Right now, if userId changes, the effect won't re-run.
   ```typescript
   useEffect(() => {
     // ...
   }, [userId]); // ✅ Add this
   ```

2. **Handle loading and error states**:
   Users should see feedback while data loads.
   ```typescript
   const [loading, setLoading] = useState(true);
   const [error, setError] = useState(null);
   
   useEffect(() => {
     setLoading(true);
     fetch(`/api/users/${userId}`)
       .then(res => res.json())
       .then(data => {
         setUser(data);
         setLoading(false);
       })
       .catch(err => {
         setError(err);
         setLoading(false);
       });
   }, [userId]);
   
   if (loading) return <Spinner />;
   if (error) return <Error message={error.message} />;
   ```

3. **Clean up for race conditions** (advanced):
   If userId changes quickly, you might get responses out of order.
   ```typescript
   useEffect(() => {
     let cancelled = false;
     
     fetch(`/api/users/${userId}`)
       .then(res => res.json())
       .then(data => {
         if (!cancelled) setUser(data);
       });
     
     return () => { cancelled = true; };
   }, [userId]);
   ```

Don't worry if #3 seems complex - it's an edge case. Focus on #1 and #2 first.

Want to pair on this? Happy to walk through it together!
*/
```

**Progressive Complexity:**

```typescript
const reviewFeedbackLevels = {
  week1_2: {
    focus: ['Syntax', 'Basic patterns'],
    example: 'Use const instead of let here since the value never changes',
  },
  
  week3_4: {
    focus: ['Component structure', 'Props vs state'],
    example: 'Consider deriving this value instead of storing it in state',
  },
  
  week5_8: {
    focus: ['Performance', 'Patterns', 'Architecture'],
    example: 'This component re-renders often. Consider using useMemo here',
  },
  
  month3_plus: {
    focus: ['System design', 'Trade-offs', 'Best practices'],
    example: 'What are the trade-offs of using Context vs prop drilling here?',
  },
};
```

---

#### **4. Pair Programming Sessions**

**Format:**

```typescript
const pairProgrammingSession = {
  frequency: '2-3 times per week',
  duration: '1-2 hours',
  
  structure: {
    first15min: 'Review task and approach',
    next60min: 'Code together (driver/navigator)',
    last15min: 'Recap learnings',
  },
  
  roles: {
    driver: 'Types the code',
    navigator: 'Guides and reviews',
    switch: 'Every 20-30 minutes',
  },
  
  guidelines: {
    mentor: [
      'Let junior drive 60% of the time',
      'Ask questions instead of giving answers',
      'Think out loud to model problem-solving',
      'Celebrate small wins',
    ],
    junior: [
      'Ask questions freely',
      'Propose solutions before asking',
      'Take notes during session',
      'Try independently after',
    ],
  },
};

// Example session dialogue:
const sessionExample = `
Mentor: "So we need to fetch user data when the component mounts. 
         What hook should we use?"

Junior: "useEffect?"

Mentor: "Exactly! And what would the dependency array be?"

Junior: "An empty array, since we only want it once?"

Mentor: "Perfect! Go ahead and write that out."

// Junior types...

Mentor: "Great. Now, what happens if the fetch fails?"

Junior: "Oh... we should handle errors."

Mentor: "Right. How would you do that with promises?"

Junior: "Using .catch()?"

Mentor: "Yep! Try adding that."
`;
```

---

#### **5. Common Mistakes & How I Address Them**

```typescript
const commonMistakes = [
  {
    mistake: 'Modifying state directly',
    example: `
// ❌ Wrong
const [items, setItems] = useState([]);
items.push(newItem); // Mutating!

// ✅ Correct
setItems([...items, newItem]);`,
    teaching: `
"I see you're modifying the state directly. In React, we need to treat 
state as immutable. Can you think of a way to add an item without modifying 
the original array?"

// Let them think...

"Right! We can use the spread operator. This creates a new array, which 
tells React to re-render."
    `,
  },
  
  {
    mistake: 'Forgetting keys in lists',
    example: `
// ❌ Wrong
{users.map(user => <UserCard user={user} />)}

// ✅ Correct  
{users.map(user => <UserCard key={user.id} user={user} />)}`,
    teaching: `
"Check the console - see that warning about keys? Keys help React identify 
which items changed. What do you think we should use as a key here?"

// They might say index

"Index works, but there's a better option. What if the list gets reordered? 
We want something unique and stable, like an id."
    `,
  },
  
  {
    mistake: 'Unnecessary state',
    example: `
// ❌ Storing derived data
const [users, setUsers] = useState([]);
const [userCount, setUserCount] = useState(0);

useEffect(() => {
  setUserCount(users.length);
}, [users]);

// ✅ Derive instead
const [users, setUsers] = useState([]);
const userCount = users.length;`,
    teaching: `
"I notice you're storing userCount in state. But it's always equal to 
users.length, right? What if we just calculated it when we need it?

This is called 'derived state' - we derive it from existing state instead 
of storing it separately. Makes the code simpler and prevents bugs!"
    `,
  },
  
  {
    mistake: 'Not cleaning up side effects',
    example: `
// ❌ No cleanup
useEffect(() => {
  const timer = setInterval(() => {
    console.log('tick');
  }, 1000);
}, []);

// ✅ With cleanup
useEffect(() => {
  const timer = setInterval(() => {
    console.log('tick');
  }, 1000);
  
  return () => clearInterval(timer); // Cleanup!
}, []);`,
    teaching: `
"Nice use of setInterval! Quick question: what happens when the component 
unmounts? The interval keeps running, right?

That's a memory leak. We need to clean it up. useEffect can return a cleanup 
function that runs when the component unmounts. Try adding that!"
    `,
  },
];
```

---

#### **6. Resources & Learning Materials**

```typescript
const learningResources = {
  documentation: [
    {
      name: 'Official React Docs',
      url: 'https://react.dev',
      when: 'Daily reference',
      focus: 'Learn section (interactive)',
    },
    {
      name: 'TypeScript Handbook',
      url: 'https://www.typescriptlang.org/docs/',
      when: 'Type-related questions',
    },
  ],
  
  courses: [
    {
      name: 'Epic React by Kent C. Dodds',
      type: 'Interactive workshop',
      level: 'Beginner to Advanced',
      investment: 'Company pays',
    },
  ],
  
  practice: [
    {
      name: 'Frontend Mentor',
      url: 'https://www.frontendmentor.io',
      why: 'Real-world designs to implement',
    },
    {
      name: 'React Challenges',
      url: 'https://github.com/alexgurr/react-coding-challenges',
      why: 'Hands-on coding practice',
    },
  ],
  
  internal: [
    'Team wiki',
    'Component library documentation',
    'Architecture decision records (ADRs)',
    'Code examples repository',
  ],
};
```

---

#### **7. Building Confidence**

```typescript
const confidenceBuilding = {
  strategies: [
    {
      strategy: 'Start with wins',
      implementation: 'Assign achievable tasks first',
      example: 'Build a simple component before tackling forms',
    },
    {
      strategy: 'Celebrate progress',
      implementation: 'Acknowledge every milestone',
      example: '"Great job shipping your first feature! 🎉"',
    },
    {
      strategy: 'Normalize mistakes',
      implementation: 'Share your own mistakes',
      example: '"I made this same mistake last week. Here\'s how I fixed it."',
    },
    {
      strategy: 'Gradual complexity',
      implementation: 'Increase difficulty progressively',
      example: 'Simple component → Form → API integration → Custom hook',
    },
    {
      strategy: 'Encourage questions',
      implementation: 'Create safe space for asking',
      example: '"No question is too basic. I ask questions daily!"',
    },
  ],
  
  progressMarkers: [
    { week: 1, milestone: 'Built first component' },
    { week: 2, milestone: 'Submitted first PR' },
    { week: 4, milestone: 'Completed feature independently' },
    { week: 8, milestone: 'Mentoring another junior' },
    { week: 12, milestone: 'Leading small project' },
  ],
};
```

---

#### **8. Tracking Progress**

```typescript
interface SkillAssessment {
  skill: string;
  level: 'Learning' | 'Practicing' | 'Proficient' | 'Expert';
  notes: string;
}

const trackProgress = {
  frequency: 'Monthly 1-on-1',
  
  assessment: [
    { skill: 'JSX & Components', level: 'Proficient', notes: 'Solid foundation' },
    { skill: 'State Management', level: 'Practicing', notes: 'Good progress, review useReducer' },
    { skill: 'Side Effects', level: 'Learning', notes: 'Understanding growing, needs more practice' },
    { skill: 'Performance', level: 'Learning', notes: 'Introduced concepts, will practice next sprint' },
  ],
  
  discussion: [
    'What went well this month?',
    'What was challenging?',
    'What do you want to learn next?',
    'How can I better support you?',
  ],
  
  goals: {
    nextMonth: 'Master useEffect and custom hooks',
    nextQuarter: 'Lead a small feature end-to-end',
    nextYear: 'Become mid-level developer',
  },
};
```

---

#### **9. Real Example: Mentoring Success Story**

**Background:**
- Junior developer, fresh bootcamp graduate
- Strong JavaScript, new to React
- Overwhelmed by codebase complexity

**Approach:**

```typescript
const mentoringPlan = {
  month1: {
    focus: 'Foundations and confidence',
    tasks: ['Small bug fixes', 'Simple components', 'Read existing code'],
    support: 'Daily check-ins, pair programming 3x/week',
    result: 'Shipped 5 small features, confidence growing',
  },
  
  month2: {
    focus: 'Forms and API integration',
    tasks: ['Build user profile form', 'Implement search feature'],
    support: 'Weekly pairing, thorough code reviews',
    result: 'Independently built and shipped form feature',
  },
  
  month3: {
    focus: 'Advanced patterns',
    tasks: ['Create custom hooks', 'Optimize performance'],
    support: 'Bi-weekly pairing, increasing independence',
    result: 'Created reusable useApi hook used by team',
  },
  
  month6: {
    status: 'Thriving',
    achievements: [
      'Leading small features independently',
      'Helping newer juniors',
      'Contributing to code review',
      'Proposed architecture improvements',
    ],
    feedback: '"I feel confident now. Thank you for your patience!"',
  },
};
```

---

#### **Summary**

**Key Principles:**
1. **Patience**: Everyone learns at their own pace
2. **Guidance**: Ask questions, don't just give answers
3. **Practice**: Learning by doing is most effective
4. **Safety**: Create space for mistakes and questions
5. **Progression**: Gradually increase complexity
6. **Celebration**: Acknowledge every win
7. **Investment**: Mentoring is investing in team's future

**Success Metrics:**
- Junior completes tasks independently
- Asks thoughtful questions
- Contributes to code reviews
- Helps other juniors
- Shows growing confidence
- Delivers quality work

**Personal Rewards:**
- Deepen your own understanding (teaching reveals gaps)
- Build strong team relationships
- Develop leadership skills
- Create lasting impact
- See others grow and succeed

**Remember:** Every senior developer was once a junior. **Mentoring is how we pay it forward.**

</details>

---

### 267. How do you make technical decisions in React projects?

<details>
<summary>View Answer</summary>

**Technical decisions** require **systematic evaluation** of options, considering **trade-offs**, **team context**, and **business goals**. Good decisions balance **pragmatism** with **technical excellence**.

---

#### **1. Decision-Making Framework**

```typescript
interface TechnicalDecision {
  context: string;
  problem: string;
  options: Option[];
  criteria: Criteria[];
  decision: string;
  rationale: string;
  tradeoffs: Tradeoff[];
  reversibility: 'Easy' | 'Medium' | 'Hard' | 'Irreversible';
  stakeholders: string[];
}

const decisionProcess = [
  {
    step: 1,
    name: 'Understand the problem',
    questions: [
      'What problem are we solving?',
      'Why is this a problem now?',
      'What are the constraints?',
      'Who is affected?',
      'What is the urgency?',
    ],
    output: 'Clear problem statement',
  },
  {
    step: 2,
    name: 'Gather context',
    activities: [
      'Research existing solutions',
      'Check team expertise',
      'Review current architecture',
      'Understand business goals',
      'Consider timeline/budget',
    ],
    output: 'Contextual constraints',
  },
  {
    step: 3,
    name: 'Generate options',
    approach: [
      'Brainstorm 3-5 options',
      'Include "do nothing" if valid',
      'Consider both new and proven solutions',
      'Think short-term and long-term',
    ],
    output: 'List of viable options',
  },
  {
    step: 4,
    name: 'Evaluate options',
    criteria: [
      'Technical fit',
      'Team expertise',
      'Performance',
      'Maintainability',
      'Cost (time & money)',
      'Risk',
      'Scalability',
    ],
    output: 'Scored comparison',
  },
  {
    step: 5,
    name: 'Make decision',
    considerations: [
      'Consult team/stakeholders',
      'Consider reversibility',
      'Document reasoning',
      'Define success metrics',
    ],
    output: 'Decision with rationale',
  },
  {
    step: 6,
    name: 'Implement & validate',
    activities: [
      'Prototype if needed',
      'Implement decision',
      'Measure results',
      'Adjust if needed',
    ],
    output: 'Validated decision',
  },
];
```

---

#### **2. Evaluation Criteria**

```typescript
interface EvaluationCriteria {
  category: string;
  weight: number; // 1-10
  questions: string[];
}

const evaluationCriteria: EvaluationCriteria[] = [
  {
    category: 'Technical Fit',
    weight: 9,
    questions: [
      'Does it solve the problem well?',
      'Does it integrate with existing stack?',
      'Is it production-ready?',
      'What are the limitations?',
    ],
  },
  {
    category: 'Team Capability',
    weight: 8,
    questions: [
      'Do we have expertise?',
      'How steep is the learning curve?',
      'Can we hire for this?',
      'Is there good documentation?',
    ],
  },
  {
    category: 'Performance',
    weight: 8,
    questions: [
      'What is the performance impact?',
      'How does it scale?',
      'What is the bundle size impact?',
      'Are there benchmarks?',
    ],
  },
  {
    category: 'Developer Experience',
    weight: 7,
    questions: [
      'Is it easy to use?',
      'Good TypeScript support?',
      'Quality of tooling?',
      'Active community?',
    ],
  },
  {
    category: 'Maintainability',
    weight: 8,
    questions: [
      'Is it actively maintained?',
      'How stable is the API?',
      'Quality of codebase?',
      'Release frequency?',
    ],
  },
  {
    category: 'Cost',
    weight: 7,
    questions: [
      'Implementation time?',
      'Training required?',
      'Licensing costs?',
      'Infrastructure costs?',
    ],
  },
  {
    category: 'Risk',
    weight: 9,
    questions: [
      'How risky is adoption?',
      'What if it fails?',
      'Can we migrate away?',
      'Bus factor (key person dependency)?',
    ],
  },
  {
    category: 'Ecosystem',
    weight: 6,
    questions: [
      'Size of community?',
      'Available plugins/integrations?',
      'Quality of documentation?',
      'Job market demand?',
    ],
  },
];
```

---

#### **3. Real Decision Examples**

**Example 1: Choosing State Management**

```typescript
const stateManagementDecision = {
  context: {
    projectSize: 'Medium (50K LOC)',
    teamSize: 5,
    expertise: 'Strong React, some Redux',
    timeline: '3 months',
  },
  
  problem: `
    - Current prop drilling is unmaintainable
    - State scattered across many components
    - No single source of truth
    - Difficult to debug state changes
  `,
  
  options: [
    {
      name: 'Context API',
      pros: [
        'Built into React',
        'No dependencies',
        'Simple for basic cases',
        'Team already knows it',
      ],
      cons: [
        'Performance issues with frequent updates',
        'No built-in dev tools',
        'Can lead to context hell',
        'No middleware support',
      ],
      score: 6,
    },
    {
      name: 'Redux Toolkit',
      pros: [
        'Industry standard',
        'Excellent dev tools',
        'Predictable state updates',
        'Great for complex state',
        'Strong typing with TypeScript',
      ],
      cons: [
        'More boilerplate',
        'Learning curve for team',
        'Overkill for simple state',
        'Bundle size increase',
      ],
      score: 8,
    },
    {
      name: 'Zustand',
      pros: [
        'Minimal boilerplate',
        'Small bundle size (1KB)',
        'Simple API',
        'Good performance',
        'Easy to learn',
      ],
      cons: [
        'Smaller ecosystem',
        'Less established',
        'Fewer dev tools',
        'Team unfamiliar',
      ],
      score: 7,
    },
    {
      name: 'Jotai',
      pros: [
        'Atomic state management',
        'Very flexible',
        'Great for derived state',
        'Modern approach',
      ],
      cons: [
        'Different mental model',
        'Steeper learning curve',
        'Less mature',
        'Team unfamiliar',
      ],
      score: 6,
    },
  ],
  
  decision: 'Redux Toolkit',
  
  rationale: `
    While Zustand is appealing for its simplicity, Redux Toolkit wins because:
    
    1. **Team familiarity**: Half the team has Redux experience
    2. **Dev tools**: Redux DevTools are invaluable for debugging
    3. **Scalability**: As the app grows, Redux scales well
    4. **Hiring**: Easier to find developers with Redux experience
    5. **Ecosystem**: Rich ecosystem of middleware and tools
    
    The boilerplate concern is addressed by RTK's modern APIs which are
    much simpler than classic Redux.
  `,
  
  tradeoffs: [
    'More setup time initially vs. long-term maintainability',
    'Larger bundle vs. better dev experience',
    'Learning curve vs. industry-standard knowledge',
  ],
  
  successMetrics: [
    'Team trained within 2 weeks',
    'Reduced prop drilling by 80%',
    'State bugs reduced by 50%',
    'Faster feature development',
  ],
  
  implementation: `
// store.ts
import { configureStore } from '@reduxjs/toolkit';
import userReducer from './features/userSlice';
import productsReducer from './features/productsSlice';

export const store = configureStore({
  reducer: {
    user: userReducer,
    products: productsReducer,
  },
});

// features/userSlice.ts
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

export const fetchUser = createAsyncThunk(
  'user/fetchUser',
  async (userId: string) => {
    const response = await fetch(\`/api/users/\${userId}\`);
    return response.json();
  }
);

const userSlice = createSlice({
  name: 'user',
  initialState: {
    data: null,
    loading: false,
    error: null,
  },
  reducers: {
    updateUser: (state, action) => {
      state.data = { ...state.data, ...action.payload };
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUser.pending, (state) => {
        state.loading = true;
      })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.loading = false;
        state.data = action.payload;
      })
      .addCase(fetchUser.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      });
  },
});

export const { updateUser } = userSlice.actions;
export default userSlice.reducer;
  `,
};
```

**Example 2: Build Tool Selection**

```typescript
const buildToolDecision = {
  context: {
    project: 'New React application',
    team: 'Modern stack preference',
    requirements: ['Fast dev experience', 'TypeScript', 'Quick builds'],
  },
  
  problem: 'Need to choose build tool for new project',
  
  options: [
    {
      name: 'Create React App (CRA)',
      pros: ['Official', 'Zero config', 'Battle-tested'],
      cons: ['Slow', 'Difficult to customize', 'Outdated tooling'],
      score: 5,
    },
    {
      name: 'Vite',
      pros: ['Extremely fast', 'Modern', 'Great DX', 'Easy config'],
      cons: ['Less mature', 'Smaller ecosystem'],
      score: 9,
    },
    {
      name: 'Next.js',
      pros: ['Full framework', 'SSR/SSG', 'Production-ready'],
      cons: ['Opinionated', 'Overkill if no SSR needed'],
      score: 7,
    },
  ],
  
  decision: 'Vite',
  
  rationale: `
    Vite is the clear winner for this SPA:
    - Dev server starts in <2 seconds vs CRA's 30+ seconds
    - HMR is nearly instant
    - Build times are 3-5x faster
    - Modern tooling (esbuild, rollup)
    - Great TypeScript support
    - Next.js is overkill without SSR needs
  `,
  
  reversibility: 'Medium',
  note: 'Can migrate to Next.js later if SSR becomes needed',
};
```

**Example 3: Component Library**

```typescript
const componentLibraryDecision = {
  context: {
    need: 'UI component library',
    timeline: 'Tight (2 months to MVP)',
    design: 'Custom design system',
  },
  
  options: [
    {
      name: 'Build from scratch',
      evaluation: {
        timeline: '6+ months',
        quality: 'Perfect fit',
        maintenance: 'High burden',
        score: 4,
      },
    },
    {
      name: 'Material UI',
      evaluation: {
        timeline: 'Immediate',
        quality: 'Good, needs customization',
        maintenance: 'Community maintained',
        score: 7,
      },
    },
    {
      name: 'Headless UI (Radix)',
      evaluation: {
        timeline: '1 month',
        quality: 'Excellent, need to style',
        maintenance: 'Low burden',
        score: 9,
      },
    },
  ],
  
  decision: 'Radix UI (headless)',
  
  rationale: `
    Radix provides:
    - Unstyled components (perfect for custom design)
    - Accessibility built-in
    - No design opinions to override
    - Small bundle size
    - We style with Tailwind CSS
    
    Best of both worlds: speed + customization
  `,
  
  implementation: `
import * as Dialog from '@radix-ui/react-dialog';

export function Modal({ children, open, onClose }) {
  return (
    <Dialog.Root open={open} onOpenChange={onClose}>
      <Dialog.Portal>
        <Dialog.Overlay className="fixed inset-0 bg-black/50" />
        <Dialog.Content className="fixed top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 bg-white rounded-lg p-6">
          {children}
        </Dialog.Content>
      </Dialog.Portal>
    </Dialog.Root>
  );
}
// Accessible, customizable, perfect!
  `,
};
```

---

#### **4. Trade-off Analysis**

```typescript
const tradeoffFramework = {
  categories: [
    {
      name: 'Speed vs Quality',
      questions: [
        'How critical is time-to-market?',
        'What is the cost of lower quality?',
        'Can we refactor later?',
      ],
      example: {
        scenario: 'MVP for investor demo',
        decision: 'Favor speed, document tech debt',
      },
    },
    {
      name: 'Flexibility vs Simplicity',
      questions: [
        'Do we know future requirements?',
        'What is the cost of over-engineering?',
        'Can we add flexibility later?',
      ],
      example: {
        scenario: 'Internal tool with clear scope',
        decision: 'Favor simplicity, YAGNI principle',
      },
    },
    {
      name: 'Performance vs Maintainability',
      questions: [
        'What are performance requirements?',
        'How often will this be modified?',
        'Can we optimize later?',
      ],
      example: {
        scenario: 'Admin dashboard (low traffic)',
        decision: 'Favor maintainability, premature optimization is evil',
      },
    },
    {
      name: 'Build vs Buy',
      questions: [
        'Is this core to our business?',
        'What is the total cost of ownership?',
        'What is our competitive advantage?',
      ],
      example: {
        scenario: 'Authentication system',
        decision: 'Buy (Auth0) - not our core competency',
      },
    },
  ],
};
```

---

#### **5. Decision Documentation (ADR)**

**Architecture Decision Record:**

```markdown
# ADR-001: State Management with Redux Toolkit

## Status
Accepted

## Context
Our React application has grown to 50K LOC with complex state management needs.
Current approach (prop drilling + scattered useState) is unmaintainable.

Constraints:
- Team of 5 developers (2 with Redux experience)
- 3-month timeline for refactor
- Need debugging tools
- App will continue growing

## Decision
We will use Redux Toolkit for global state management.

## Consequences

### Positive
- Single source of truth for application state
- Time-travel debugging with Redux DevTools
- Predictable state updates
- Strong TypeScript integration
- Easier to onboard new developers (industry standard)
- Scales well as application grows

### Negative
- Initial setup time (estimated 1-2 weeks)
- Learning curve for 3 team members
- Slightly larger bundle size (+15KB gzipped)
- More boilerplate than Context API

### Neutral
- Need to define clear guidelines for what goes in Redux vs local state
- Will create custom hooks to simplify usage

## Alternatives Considered

### Context API
- Rejected: Performance issues with frequent updates
- Rejected: No dev tools
- Rejected: Doesn't scale well

### Zustand
- Considered: Simpler API, smaller bundle
- Rejected: Less established, team unfamiliar
- May revisit for future projects

### Jotai
- Rejected: Different mental model requires more learning
- Rejected: Less mature ecosystem

## Implementation Notes
- Use Redux Toolkit's modern APIs (createSlice, createAsyncThunk)
- Co-locate slices with features
- Use selectors for derived state
- Document state structure in wiki

## Success Metrics
- Team trained by Week 2
- First feature migrated by Week 3
- Full migration by Month 3
- 50% reduction in state-related bugs
- Improved developer velocity

## Review Date
6 months after implementation (June 2024)

## References
- [Redux Toolkit Documentation](https://redux-toolkit.js.org/)
- [Team training materials](link)
- [Prototype branch](link)
```

---

#### **6. Involving Stakeholders**

```typescript
const stakeholderInvolvement = {
  types: [
    {
      stakeholder: 'Engineering Team',
      when: 'All technical decisions',
      how: 'RFCs, team meetings, async feedback',
      why: 'They implement and maintain',
    },
    {
      stakeholder: 'Product Manager',
      when: 'Decisions affecting timeline/features',
      how: 'Impact analysis, trade-off discussions',
      why: 'Owns product roadmap',
    },
    {
      stakeholder: 'Engineering Manager',
      when: 'Resource-intensive decisions',
      how: 'Budget/timeline planning',
      why: 'Owns team capacity',
    },
    {
      stakeholder: 'Designers',
      when: 'UI library, styling approach',
      how: 'Design system discussions',
      why: 'Owns user experience',
    },
    {
      stakeholder: 'Security Team',
      when: 'Auth, data handling, dependencies',
      how: 'Security review',
      why: 'Owns security posture',
    },
  ],
  
  process: [
    '1. Draft proposal (RFC)',
    '2. Share with relevant stakeholders',
    '3. Collect feedback (async + sync)',
    '4. Address concerns',
    '5. Make decision',
    '6. Document and communicate',
  ],
};
```

---

#### **7. When to Re-evaluate**

```typescript
const reevaluationTriggers = [
  {
    trigger: 'Major version release',
    action: 'Review if upgrade makes sense',
    example: 'React 19 released',
  },
  {
    trigger: 'Performance issues',
    action: 'Profile and consider alternatives',
    example: 'Bundle size exceeds budget',
  },
  {
    trigger: 'Maintenance burden',
    action: 'Evaluate if worth continuing',
    example: 'Spending 50% time on old library',
  },
  {
    trigger: 'Security vulnerabilities',
    action: 'Assess risk and migration path',
    example: 'Critical CVE with no fix',
  },
  {
    trigger: 'Team feedback',
    action: 'Listen to developer pain points',
    example: 'Consistent complaints about DX',
  },
  {
    trigger: 'Technology shift',
    action: 'Evaluate if still best choice',
    example: 'Industry moving to new standard',
  },
];
```

---

#### **8. Common Decision Pitfalls**

```typescript
const commonPitfalls = [
  {
    pitfall: 'Resume-Driven Development',
    description: 'Choosing tech because you want to learn it',
    avoid: 'Prioritize project needs over personal interests',
  },
  {
    pitfall: 'Hype-Driven Development',
    description: 'Choosing latest trend without evaluation',
    avoid: 'Let tech mature, wait for proven track record',
  },
  {
    pitfall: 'Analysis Paralysis',
    description: 'Over-analyzing, never deciding',
    avoid: 'Set decision deadline, "good enough" is fine',
  },
  {
    pitfall: 'Not Invented Here',
    description: 'Rebuilding instead of using existing solutions',
    avoid: 'Default to "buy/use", only build if necessary',
  },
  {
    pitfall: 'Ignoring Team Context',
    description: 'Choosing tech team can\'t support',
    avoid: 'Consider team skills and capacity',
  },
  {
    pitfall: 'Premature Optimization',
    description: 'Optimizing before knowing if needed',
    avoid: 'Measure first, optimize later',
  },
  {
    pitfall: 'Sunk Cost Fallacy',
    description: 'Continuing with wrong tech because invested',
    avoid: 'Be willing to pivot if decision proves wrong',
  },
];
```

---

#### **Summary**

**Decision-Making Principles:**

1. **Understand the problem deeply** before evaluating solutions
2. **Consider context**: team, timeline, business goals
3. **Generate multiple options** (3-5 typically)
4. **Evaluate systematically** using consistent criteria
5. **Document your reasoning** (ADRs)
6. **Involve stakeholders** appropriately
7. **Accept trade-offs** - no perfect solution
8. **Be pragmatic** - good enough often beats perfect
9. **Stay reversible** when possible
10. **Measure outcomes** and learn

**Key Questions:**
- Does it solve the problem?
- Can the team support it?
- What are the trade-offs?
- What is the risk?
- Is it reversible?
- What is the total cost?

**Remember:**
- **Context matters**: Same problem, different context = different decision
- **Trade-offs exist**: Every decision has pros and cons
- **Reversibility**: Prefer decisions that can be changed
- **Document**: Future you will thank you
- **Learn**: Reflect on decisions, improve process

Good technical decisions balance **technical merit**, **team capability**, **business needs**, and **pragmatism**.

</details>

---

### 268. Describe a time you had to refactor legacy React code

<details>
<summary>View Answer</summary>

**Legacy code refactoring** requires **careful planning**, **incremental changes**, and **balancing improvement with business needs**. Success means delivering value while minimizing risk.

---

#### **Case Study: E-commerce Legacy Refactor**

**Situation:**
- E-commerce platform built in 2018 with React 16.3
- 150K+ lines of code, 200+ components
- Mixture of class and functional components
- Redux with legacy patterns (connect HOCs)
- No TypeScript, minimal testing (<20% coverage)
- jQuery still in codebase for some features
- Performance issues (8s initial load)
- Team complaints about slow development
- High bug rate in production

**Key Problems:**
```typescript
const problemAreas = [
  {
    area: 'Architecture',
    issues: [
      'Massive god components (2000+ lines)',
      'Tight coupling between components',
      'Business logic mixed with UI',
      'No clear separation of concerns',
    ],
    impact: 'Hard to maintain, test, and extend',
  },
  {
    area: 'Code Quality',
    issues: [
      'Inconsistent patterns (class + functional)',
      'No type safety',
      'Code duplication everywhere',
      'Dead code accumulation',
    ],
    impact: 'High bug rate, slow development',
  },
  {
    area: 'Performance',
    issues: [
      'No code splitting',
      'Unnecessary re-renders',
      'Large bundle (3MB)',
      'No lazy loading',
    ],
    impact: 'Poor user experience, high bounce rate',
  },
  {
    area: 'Testing',
    issues: [
      '<20% test coverage',
      'Brittle tests',
      'Hard to test components',
    ],
    impact: 'Fear of making changes, frequent regressions',
  },
];
```

**Task:**
- Refactor codebase while maintaining feature parity
- Improve performance by 50%+
- Increase test coverage to 80%+
- Migrate to TypeScript
- Zero downtime, no disruption to users
- Timeline: 6 months (with ongoing feature work)

---

#### **Action: Refactoring Strategy**

### **Phase 1: Assessment (Week 1-2)**

```typescript
// 1. Code analysis
const analysis = {
  tools: ['SonarQube', 'webpack-bundle-analyzer', 'ESLint', 'Chrome DevTools'],
  
  findings: {
    largestComponents: [
      { name: 'ProductPage', lines: 2500, issues: 'Does everything' },
      { name: 'CheckoutFlow', lines: 1800, issues: 'Complex state management' },
      { name: 'Dashboard', lines: 1500, issues: 'Performance issues' },
    ],
    
    codeSmells: [
      'Prop drilling 5+ levels deep',
      'Duplicate logic across components',
      '50+ connect() HOCs',
      'Mixed class and functional components',
      'Inline styles and CSS modules mixed',
    ],
    
    dependencies: {
      outdated: ['react@16.3', 'redux@3.7', 'react-router@4'],
      unnecessary: ['moment.js', 'lodash (full)', 'jQuery'],
      vulnerable: ['3 high-severity CVEs'],
    },
    
    bundleAnalysis: {
      mainBundle: '3.2MB',
      largestDependencies: [
        'moment.js: 288KB',
        'lodash: 72KB',
        'chart.js: 180KB',
      ],
    },
  },
};

// 2. Impact mapping
const impactMap = [
  {
    component: 'ProductPage',
    usageFrequency: 'Very High',
    businessImpact: 'Critical',
    refactorComplexity: 'High',
    priority: 'P0',
  },
  {
    component: 'CheckoutFlow',
    usageFrequency: 'High',
    businessImpact: 'Critical',
    refactorComplexity: 'Very High',
    priority: 'P0',
  },
  {
    component: 'Dashboard',
    usageFrequency: 'Medium',
    businessImpact: 'Medium',
    refactorComplexity: 'Medium',
    priority: 'P1',
  },
];
```

### **Phase 2: Strategy & Planning (Week 3-4)**

```typescript
const refactoringStrategy = {
  approach: 'Strangler Fig Pattern',
  description: 'Gradually replace old code with new, old and new coexist',
  
  principles: [
    'Incremental changes, not big rewrites',
    'Always shippable (no long-lived branches)',
    'Feature flags for gradual rollout',
    'Automated testing before refactoring',
    'One area at a time',
  ],
  
  phases: [
    {
      phase: 1,
      name: 'Foundation',
      duration: '1 month',
      goals: [
        'Set up TypeScript',
        'Update dependencies',
        'Add testing infrastructure',
        'Set up code quality tools',
      ],
    },
    {
      phase: 2,
      name: 'Critical Path',
      duration: '2 months',
      goals: [
        'Refactor ProductPage',
        'Refactor CheckoutFlow',
        'Improve performance',
      ],
    },
    {
      phase: 3,
      name: 'Systematic Refactor',
      duration: '2 months',
      goals: [
        'Refactor remaining god components',
        'Migrate all Redux to Redux Toolkit',
        'Increase test coverage',
      ],
    },
    {
      phase: 4,
      name: 'Polish',
      duration: '1 month',
      goals: [
        'Remove dead code',
        'Full TypeScript migration',
        'Documentation',
      ],
    },
  ],
};
```

### **Phase 3: Implementation**

**Example 1: Refactoring God Component**

```typescript
// ❌ Before: 2500-line ProductPage.jsx
class ProductPage extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      product: null,
      selectedVariant: null,
      quantity: 1,
      reviews: [],
      recommendations: [],
      isInCart: false,
      // ... 50+ more state variables
    };
  }
  
  componentDidMount() {
    // 200+ lines of initialization logic
    this.fetchProduct();
    this.fetchReviews();
    this.fetchRecommendations();
    this.trackPageView();
    this.initializeAnalytics();
    // ... 15+ more method calls
  }
  
  fetchProduct() {
    // 100+ lines
  }
  
  handleAddToCart() {
    // 80+ lines
  }
  
  handleVariantChange() {
    // 60+ lines
  }
  
  // ... 40+ more methods
  
  render() {
    // 800+ lines of JSX
    return (
      <div>
        {/* Massive JSX tree */}
      </div>
    );
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(ProductPage);

// ✅ After: Decomposed into focused components

// 1. Main component (orchestration only)
function ProductPage() {
  const { id } = useParams();
  const { data: product, isLoading, error } = useProduct(id);
  
  if (isLoading) return <ProductSkeleton />;
  if (error) return <ErrorBoundary error={error} />;
  if (!product) return <NotFound />;
  
  return (
    <div className="product-page">
      <ProductImageGallery images={product.images} />
      <ProductInfo product={product} />
      <ProductVariants product={product} />
      <ProductActions product={product} />
      <ProductReviews productId={product.id} />
      <ProductRecommendations productId={product.id} />
    </div>
  );
}

// 2. Custom hook for data fetching
function useProduct(id: string) {
  return useQuery(['product', id], () => fetchProduct(id), {
    staleTime: 5 * 60 * 1000,
    onSuccess: (product) => {
      // Track page view
      analytics.track('product_viewed', {
        productId: product.id,
        productName: product.name,
      });
    },
  });
}

// 3. ProductInfo component (100 lines)
function ProductInfo({ product }: { product: Product }) {
  return (
    <div className="product-info">
      <h1>{product.name}</h1>
      <ProductPrice price={product.price} />
      <ProductRating rating={product.rating} />
      <ProductDescription description={product.description} />
    </div>
  );
}

// 4. ProductVariants with local state (150 lines)
function ProductVariants({ product }: { product: Product }) {
  const [selectedVariant, setSelectedVariant] = useState(product.variants[0]);
  
  return (
    <div className="variants">
      {product.variants.map(variant => (
        <VariantOption
          key={variant.id}
          variant={variant}
          selected={selectedVariant.id === variant.id}
          onSelect={setSelectedVariant}
        />
      ))}
    </div>
  );
}

// 5. ProductActions with cart logic (120 lines)
function ProductActions({ product }: { product: Product }) {
  const { addToCart } = useCart();
  const [quantity, setQuantity] = useState(1);
  const [isAdding, setIsAdding] = useState(false);
  
  const handleAddToCart = async () => {
    setIsAdding(true);
    try {
      await addToCart(product, quantity);
      toast.success('Added to cart!');
    } catch (error) {
      toast.error('Failed to add to cart');
    } finally {
      setIsAdding(false);
    }
  };
  
  return (
    <div className="product-actions">
      <QuantitySelector value={quantity} onChange={setQuantity} />
      <Button onClick={handleAddToCart} loading={isAdding}>
        Add to Cart
      </Button>
    </div>
  );
}

// Result:
// - 2500 lines → ~800 lines total (split across files)
// - Each component has single responsibility
// - Easy to test each piece
// - Reusable components
// - Better performance (smaller components)
```

**Example 2: Migrating Redux**

```typescript
// ❌ Before: Classic Redux with connect HOC

// actions.js
export const FETCH_PRODUCTS_REQUEST = 'FETCH_PRODUCTS_REQUEST';
export const FETCH_PRODUCTS_SUCCESS = 'FETCH_PRODUCTS_SUCCESS';
export const FETCH_PRODUCTS_FAILURE = 'FETCH_PRODUCTS_FAILURE';

export const fetchProducts = () => {
  return async (dispatch) => {
    dispatch({ type: FETCH_PRODUCTS_REQUEST });
    try {
      const response = await fetch('/api/products');
      const products = await response.json();
      dispatch({ type: FETCH_PRODUCTS_SUCCESS, payload: products });
    } catch (error) {
      dispatch({ type: FETCH_PRODUCTS_FAILURE, payload: error });
    }
  };
};

// reducer.js
const initialState = {
  products: [],
  loading: false,
  error: null,
};

export default function productsReducer(state = initialState, action) {
  switch (action.type) {
    case FETCH_PRODUCTS_REQUEST:
      return { ...state, loading: true };
    case FETCH_PRODUCTS_SUCCESS:
      return { ...state, loading: false, products: action.payload };
    case FETCH_PRODUCTS_FAILURE:
      return { ...state, loading: false, error: action.payload };
    default:
      return state;
  }
}

// Component
class ProductList extends React.Component {
  componentDidMount() {
    this.props.fetchProducts();
  }
  
  render() {
    const { products, loading, error } = this.props;
    // ...
  }
}

const mapStateToProps = (state) => ({
  products: state.products.products,
  loading: state.products.loading,
  error: state.products.error,
});

const mapDispatchToProps = {
  fetchProducts,
};

export default connect(mapStateToProps, mapDispatchToProps)(ProductList);

// ✅ After: Redux Toolkit + React Query

// productsSlice.ts (optional, if needed)
import { createSlice } from '@reduxjs/toolkit';

const productsSlice = createSlice({
  name: 'products',
  initialState: {
    filters: {},
    sort: 'popular',
  },
  reducers: {
    setFilters: (state, action) => {
      state.filters = action.payload;
    },
    setSort: (state, action) => {
      state.sort = action.payload;
    },
  },
});

export const { setFilters, setSort } = productsSlice.actions;
export default productsSlice.reducer;

// hooks/useProducts.ts (React Query for server state)
import { useQuery } from '@tanstack/react-query';

export function useProducts() {
  return useQuery(['products'], fetchProducts, {
    staleTime: 5 * 60 * 1000,
  });
}

// Component (much simpler!)
function ProductList() {
  const { data: products, isLoading, error } = useProducts();
  const filters = useSelector((state) => state.products.filters);
  
  if (isLoading) return <Spinner />;
  if (error) return <Error message={error.message} />;
  
  const filtered = filterProducts(products, filters);
  
  return (
    <div>
      {filtered.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}

// Benefits:
// - Less boilerplate (no action types, mapStateToProps)
// - Better separation (server state vs UI state)
// - Automatic caching, refetching with React Query
// - Type-safe with TypeScript
```

**Example 3: Adding TypeScript Incrementally**

```typescript
// Strategy: Rename .js to .tsx incrementally, fix errors

// tsconfig.json (permissive at first)
{
  "compilerOptions": {
    "allowJs": true,           // Allow .js files
    "checkJs": false,          // Don't check .js files
    "noImplicitAny": false,    // Allow implicit any
    "strict": false,           // Not strict yet
    // Gradually tighten over time
  }
}

// Week 1-2: Convert types and utilities
// types/product.ts
export interface Product {
  id: string;
  name: string;
  price: number;
  images: string[];
  variants: ProductVariant[];
}

export interface ProductVariant {
  id: string;
  sku: string;
  price: number;
  attributes: Record<string, string>;
}

// Week 3-4: Convert leaf components (no dependencies)
// components/Button.tsx
interface ButtonProps {
  children: React.ReactNode;
  onClick?: () => void;
  variant?: 'primary' | 'secondary';
  loading?: boolean;
}

export function Button({ 
  children, 
  onClick, 
  variant = 'primary',
  loading = false,
}: ButtonProps) {
  return (
    <button 
      className={`btn btn-${variant}`}
      onClick={onClick}
      disabled={loading}
    >
      {loading ? <Spinner /> : children}
    </button>
  );
}

// Week 5-8: Convert pages and complex components
// Gradually enable stricter checks
```

### **Phase 4: Testing**

```typescript
// Added comprehensive tests during refactor

// ProductPage.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { ProductPage } from './ProductPage';

describe('ProductPage', () => {
  it('renders product information', async () => {
    render(<ProductPage />);
    
    await waitFor(() => {
      expect(screen.getByText('Product Name')).toBeInTheDocument();
    });
  });
  
  it('adds product to cart', async () => {
    const { addToCart } = useCart();
    render(<ProductPage />);
    
    await userEvent.click(screen.getByText('Add to Cart'));
    
    await waitFor(() => {
      expect(addToCart).toHaveBeenCalledWith(expect.objectContaining({
        id: 'product-1',
      }));
    });
  });
});

// Result: Coverage increased from 20% to 85%
```

---

#### **Result: Transformative Improvement**

```typescript
const results = {
  metrics: {
    before: {
      initialLoad: 8000,        // ms
      bundleSize: 3200,         // KB
      testCoverage: 20,         // %
      bugRate: 15,              // bugs/sprint
      deploymentTime: 45,       // minutes
      developerVelocity: 20,    // story points/sprint
      codeQuality: 'C',         // SonarQube grade
    },
    after: {
      initialLoad: 2100,        // ms - 74% faster! ✅
      bundleSize: 850,          // KB - 73% smaller! ✅
      testCoverage: 85,         // % - 4x improvement! ✅
      bugRate: 3,               // bugs/sprint - 80% fewer! ✅
      deploymentTime: 12,       // minutes - 73% faster! ✅
      developerVelocity: 35,    // story points/sprint - 75% faster! ✅
      codeQuality: 'A',         // SonarQube grade ✅
    },
  },
  
  businessImpact: {
    userExperience: [
      'Bounce rate: 45% → 18%',
      'Conversion rate: +22%',
      'User satisfaction: 3.2 → 4.6/5',
    ],
    development: [
      'New feature delivery: 2x faster',
      'Onboarding time: 3 weeks → 1 week',
      'Developer satisfaction: Much higher',
    ],
    business: [
      'Support tickets: -60%',
      'Revenue: +15% (better UX)',
      'Engineering costs: -30% (efficiency)',
    ],
  },
};
```

---

#### **Key Learnings**

```typescript
const lessons = [
  {
    lesson: 'Incremental is key',
    detail: 'Small, continuous improvements beat big rewrites',
    example: 'Refactored one component per week, always shippable',
  },
  {
    lesson: 'Test before refactoring',
    detail: 'Tests give confidence that refactor didn\'t break things',
    example: 'Added tests first, then refactored with confidence',
  },
  {
    lesson: 'Strangler Fig pattern works',
    detail: 'New code coexists with old, gradually replace',
    example: 'Feature flags allowed gradual rollout',
  },
  {
    lesson: 'Focus on value',
    detail: 'Prioritize high-impact, high-traffic areas first',
    example: 'ProductPage and Checkout first (80% of traffic)',
  },
  {
    lesson: 'Team involvement crucial',
    detail: 'Everyone needs to buy in and participate',
    example: 'Weekly refactoring guild, shared ownership',
  },
  {
    lesson: 'Measure everything',
    detail: 'Track metrics to prove value and guide decisions',
    example: 'Performance budgets, coverage requirements',
  },
];
```

---

#### **Interview Tips**

**Structure Your Answer:**
1. **Context**: What was the legacy codebase like?
2. **Challenge**: What specific problems existed?
3. **Approach**: How did you tackle it systematically?
4. **Implementation**: Give concrete examples
5. **Results**: Show measurable improvements
6. **Learnings**: What would you do differently?

**Key Points to Cover:**
- Assessment and planning
- Incremental approach (not big rewrite)
- Testing strategy
- Team involvement
- Risk mitigation
- Measurable outcomes
- Business impact

**Show You Understand:**
- Trade-offs of rewrite vs refactor
- Importance of backward compatibility
- Value of automated testing
- Need for stakeholder communication
- Balance between tech debt and features

**Red Flags to Avoid:**
- "I rewrote everything from scratch"
- "I did it all myself"
- No mention of testing
- No measurable outcomes
- Ignoring business impact

Legacy code refactoring is about **delivering value incrementally** while **managing risk** and **improving both user and developer experience**.

</details>

---

### 269. How do you balance speed vs quality in development?

<details>
<summary>View Answer</summary>

**Balancing speed and quality** is one of the most critical skills in software development. It requires **understanding context**, **assessing trade-offs**, and **making pragmatic decisions** aligned with business goals.

---

#### **1. The Fundamental Framework**

```typescript
const balanceFramework = {
  principle: 'Quality is not binary - it exists on a spectrum',
  
  factors: [
    {
      factor: 'Context',
      questions: [
        'What stage is the product? (MVP vs mature)',
        'What is the business impact?',
        'What is the cost of bugs?',
        'What is the cost of delay?',
      ],
    },
    {
      factor: 'Reversibility',
      questions: [
        'How easy is it to change later?',
        'Is this a core architectural decision?',
        'Can we refactor incrementally?',
      ],
    },
    {
      factor: 'Risk',
      questions: [
        'What is the blast radius of failure?',
        'How many users are affected?',
        'Are there financial/legal implications?',
      ],
    },
  ],
  
  guidingPrinciples: [
    'Always maintain minimum viable quality',
    'Invest more quality in hard-to-change decisions',
    'Speed on non-critical paths, quality on critical paths',
    'Document technical debt deliberately taken',
    'Plan to pay back tech debt incrementally',
  ],
};
```

---

#### **2. Quality Spectrum**

```typescript
interface QualityLevel {
  level: string;
  characteristics: string[];
  whenToUse: string;
  example: string;
}

const qualitySpectrum: QualityLevel[] = [
  {
    level: 'Minimum Viable Quality',
    characteristics: [
      'Works for happy path',
      'Basic error handling',
      'Minimal tests (smoke tests)',
      'Quick and dirty OK',
    ],
    whenToUse: 'Prototypes, throwaway code, proof of concepts',
    example: 'Weekend hackathon project',
  },
  {
    level: 'MVP Quality',
    characteristics: [
      'Core functionality works',
      'Major edge cases covered',
      'Some automated tests',
      'Basic monitoring',
      'Known tech debt documented',
    ],
    whenToUse: 'Initial product launch, validating market fit',
    example: 'First version to early customers',
  },
  {
    level: 'Production Quality',
    characteristics: [
      'Comprehensive error handling',
      'Good test coverage (70-80%)',
      'Performance optimized',
      'Security reviewed',
      'Monitoring and logging',
      'Documentation complete',
    ],
    whenToUse: 'General product features, stable systems',
    example: 'Standard feature release',
  },
  {
    level: 'Critical Quality',
    characteristics: [
      'Exhaustive testing (>90% coverage)',
      'Multiple code reviews',
      'Security audit',
      'Performance testing',
      'Chaos engineering',
      'Extensive documentation',
      'Gradual rollout plan',
    ],
    whenToUse: 'Payment systems, core infrastructure, high-risk features',
    example: 'Checkout flow, authentication system',
  },
];
```

---

#### **3. Decision Framework**

```typescript
class QualityDecisionMaker {
  decideQualityLevel(feature: Feature): QualityLevel {
    let score = 0;
    
    // Factor 1: User impact
    if (feature.userImpact === 'critical') score += 10;
    else if (feature.userImpact === 'high') score += 7;
    else if (feature.userImpact === 'medium') score += 4;
    else score += 1;
    
    // Factor 2: Revenue impact
    if (feature.revenueImpact === 'direct') score += 10;
    else if (feature.revenueImpact === 'indirect') score += 5;
    
    // Factor 3: Reversibility
    if (feature.reversibility === 'hard') score += 8;
    else if (feature.reversibility === 'medium') score += 4;
    
    // Factor 4: Blast radius
    if (feature.blastRadius === 'entire-system') score += 10;
    else if (feature.blastRadius === 'multiple-areas') score += 6;
    else if (feature.blastRadius === 'single-area') score += 2;
    
    // Factor 5: Regulatory/compliance
    if (feature.hasComplianceRequirements) score += 10;
    
    // Decide based on score
    if (score >= 30) return 'Critical Quality';
    if (score >= 20) return 'Production Quality';
    if (score >= 10) return 'MVP Quality';
    return 'Minimum Viable Quality';
  }
}

// Example usage
const checkoutFeature = {
  userImpact: 'critical',           // +10
  revenueImpact: 'direct',          // +10
  reversibility: 'hard',            // +8
  blastRadius: 'entire-system',     // +10
  hasComplianceRequirements: true,  // +10
  // Total: 48 -> Critical Quality
};

const dashboardWidget = {
  userImpact: 'medium',             // +4
  revenueImpact: 'indirect',        // +5
  reversibility: 'easy',            // +0
  blastRadius: 'single-area',       // +2
  hasComplianceRequirements: false, // +0
  // Total: 11 -> MVP Quality
};
```

---

#### **4. Practical Quality Gates**

```typescript
const qualityGates = {
  nonNegotiable: [
    {
      gate: 'Security',
      rule: 'Never ship known security vulnerabilities',
      check: 'npm audit, Snyk, manual security review',
      reason: 'Legal and reputation risk too high',
    },
    {
      gate: 'Data integrity',
      rule: 'Never risk user data corruption',
      check: 'Database migrations tested, backups verified',
      reason: 'Irreversible damage',
    },
    {
      gate: 'Core functionality',
      rule: 'Happy path must work',
      check: 'Smoke tests pass',
      reason: 'Product unusable otherwise',
    },
  ],
  
  negotiable: [
    {
      gate: 'Test coverage',
      ideal: '80%',
      mvp: '60%',
      rationale: 'Can add tests incrementally',
    },
    {
      gate: 'Performance',
      ideal: '<2s load time',
      mvp: '<5s load time',
      rationale: 'Can optimize after launch if needed',
    },
    {
      gate: 'Edge cases',
      ideal: 'All covered',
      mvp: 'Major ones covered',
      rationale: 'Can handle as bugs reported',
    },
    {
      gate: 'Documentation',
      ideal: 'Complete',
      mvp: 'Basic',
      rationale: 'Can improve over time',
    },
  ],
};
```

---

#### **5. Real-World Scenarios**

**Scenario 1: MVP Launch**

```typescript
const mvpScenario = {
  context: {
    situation: 'Need to launch MVP in 6 weeks',
    goal: 'Validate product-market fit',
    users: '100 beta customers',
    funding: 'Runway depends on showing traction',
  },
  
  decision: {
    approach: 'Focus on core user journey, defer polish',
    
    whatWeDid: [
      '✅ Core features work well',
      '✅ Payment integration secure',
      '✅ Basic error handling',
      '✅ Smoke tests',
      '✅ Basic monitoring',
    ],
    
    whatWeDeferred: [
      '⌛ Edge cases (documented as known issues)',
      '⌛ Comprehensive test suite',
      '⌛ Performance optimization',
      '⌛ Admin dashboard polish',
      '⌛ Internationalization',
    ],
    
    safeguards: [
      'Manual QA for each release',
      'Limited beta rollout',
      'Daily monitoring',
      'Hot-fix process ready',
      'Tech debt backlog tracked',
    ],
  },
  
  outcome: {
    success: 'Launched on time, validated PMF',
    techDebt: 'Paid back over next 3 months',
    lesson: 'Speed was right call - learned what to build next',
  },
};
```

**Scenario 2: Payment System**

```typescript
const paymentScenario = {
  context: {
    situation: 'Building payment processing',
    risk: 'Financial and legal implications',
    users: '10,000+ customers',
  },
  
  decision: {
    approach: 'No shortcuts - full quality',
    
    qualityInvestments: [
      '✅ Comprehensive test suite (95% coverage)',
      '✅ Security audit by external firm',
      '✅ Code review by 3 senior engineers',
      '✅ Integration tests with payment provider',
      '✅ Extensive error handling',
      '✅ Transaction logging and monitoring',
      '✅ Gradual rollout (1% → 10% → 50% → 100%)',
      '✅ Rollback plan tested',
    ],
    
    timeline: '6 weeks (vs 2 weeks for quick version)',
    
    rationale: [
      'Payment bugs affect revenue directly',
      'Financial regulations must be met',
      'User trust critical',
      'Very hard to fix bugs in production',
      'High reputational risk',
    ],
  },
  
  outcome: {
    success: 'Zero payment issues in first 6 months',
    cost: 'Extra 4 weeks upfront',
    value: 'Saved months of support and fixes',
    lesson: 'Quality investment paid off 10x',
  },
};
```

**Scenario 3: Refactoring vs New Feature**

```typescript
const refactorScenario = {
  context: {
    situation: 'Product Manager wants new feature ASAP',
    problem: 'Current codebase is brittle, needs refactoring',
    pressure: 'Competitor just launched similar feature',
  },
  
  approach: {
    strategy: 'Hybrid - minimal refactor + feature',
    
    weekByWeek: [
      {
        week: 1,
        focus: 'Refactor only the area needed for new feature',
        deliverable: 'Clean foundation',
      },
      {
        week: 2,
        focus: 'Build feature on clean foundation',
        deliverable: 'Feature works well',
      },
      {
        week: 3,
        focus: 'Testing and polish',
        deliverable: 'Ship to production',
      },
    ],
    
    compromise: {
      gain: 'Feature ships in 3 weeks with good quality',
      cost: 'Not full refactor (tech debt remains elsewhere)',
      plan: 'Continue refactoring incrementally',
    },
  },
  
  conversation: `
Me: "I understand the urgency. Here's what I propose:

Option A: Ship feature in 1 week
- Risk: Built on shaky foundation
- Result: Will create more tech debt
- Future cost: 3x longer to add next feature

Option B: Refactor everything first, then feature (6 weeks)
- Risk: Too slow, competitor advantage
- Result: Clean codebase
- Future cost: None, but missed market window

Option C: Targeted refactor + feature (3 weeks) ← My recommendation
- Risk: Moderate
- Result: Feature on solid foundation
- Future cost: Some tech debt remains, but manageable

I recommend Option C. We get speed AND quality where it matters most."

PM: "Option C works. Let's do it."
  `,
  
  outcome: {
    result: 'Shipped quality feature in 3 weeks',
    lesson: 'Communication and compromise key',
  },
};
```

---

#### **6. Managing Technical Debt**

```typescript
const techDebtStrategy = {
  principle: 'Technical debt is a tool, not a failure',
  
  deliberateDebt: {
    definition: 'Consciously choosing speed over perfection',
    requirements: [
      'Document what was cut',
      'Explain why',
      'Estimate payback cost',
      'Create ticket to address',
      'Set timeline for payback',
    ],
    
    example: `
// TODO [TECH-DEBT-123]: Quick implementation for MVP
// Current: Linear search O(n)
// Future: Should use indexed lookup O(1) when dataset grows >1000 items
// Estimated refactor: 2 hours
// Priority: P2 (address before v2.0)
function findUser(users: User[], id: string) {
  return users.find(u => u.id === id);
}
    `,
  },
  
  debtPayback: {
    allocation: 'Reserve 15-20% of sprint capacity',
    prioritization: [
      'Security vulnerabilities',
      'Performance bottlenecks',
      'Blocking future features',
      'Team velocity impact',
    ],
    
    tracking: {
      tool: 'Jira/Linear with "Tech Debt" label',
      review: 'Monthly debt review meeting',
      metrics: [
        'Total debt count',
        'Debt by age',
        'Debt by priority',
        'Debt payback rate',
      ],
    },
  },
  
  communicating: {
    toStakeholders: `
"Think of technical debt like a credit card:
- Sometimes it makes sense to use it (speed to market)
- But we need to pay it back (or interest compounds)
- Too much debt slows everything down
- We're allocating 15% of time to pay it back
- This keeps our velocity sustainable"
    `,
  },
};
```

---

#### **7. Speed Techniques That Don't Sacrifice Quality**

```typescript
const smartSpeed = [
  {
    technique: 'Use proven libraries',
    example: 'React Query instead of custom data fetching',
    benefit: 'Battle-tested, saves time, better quality',
  },
  {
    technique: 'Component libraries',
    example: 'Radix UI + Tailwind instead of custom components',
    benefit: 'Accessible out of the box, faster',
  },
  {
    technique: 'Code generation',
    example: 'Generate TypeScript types from OpenAPI specs',
    benefit: 'Automated, always in sync, no manual errors',
  },
  {
    technique: 'Good tooling',
    example: 'Vite for fast dev, Prettier for formatting',
    benefit: 'Less time on setup/formatting, more on features',
  },
  {
    technique: 'Narrow scope',
    example: 'Ship 80% solution that solves 100% of core use case',
    benefit: 'Faster to market, can iterate based on feedback',
  },
  {
    technique: 'Timeboxing',
    example: 'Spend max 2 days on problem before asking for help',
    benefit: 'Prevents rabbit holes',
  },
];
```

---

#### **8. Red Flags (When NOT to Cut Quality)**

```typescript
const redFlags = [
  {
    area: 'Security',
    example: 'Skipping input validation',
    consequence: 'Data breach, legal liability',
    verdict: '❌ Never skip',
  },
  {
    area: 'Data integrity',
    example: 'No database migration testing',
    consequence: 'Data corruption, irreversible',
    verdict: '❌ Never skip',
  },
  {
    area: 'Core architecture',
    example: 'Hardcoding instead of proper abstraction',
    consequence: 'Impossible to change later',
    verdict: '❌ Never skip',
  },
  {
    area: 'Error handling',
    example: 'No try-catch in async operations',
    consequence: 'App crashes, poor UX',
    verdict: '❌ Never skip',
  },
  {
    area: 'Critical user flows',
    example: 'No testing on signup/checkout',
    consequence: 'Revenue loss, user frustration',
    verdict: '❌ Never skip',
  },
];
```

---

#### **9. Communication is Key**

```typescript
const communicationExamples = {
  withPM: {
    scenario: 'PM wants feature faster',
    
    bad: '❌ "It\'ll take as long as it takes. You don\'t understand technical complexity."',
    
    good: `✅ "I hear the urgency. Let me break down the options:

Full implementation: 3 weeks
- All edge cases covered
- Comprehensive tests
- Scalable architecture

MVP version: 1.5 weeks
- Core functionality works
- Major edge cases covered
- Basic tests
- Can refine later

Quick & dirty: 3 days
- Works for happy path only
- High risk of bugs
- Will slow us down later

I recommend MVP version. We get speed without sacrificing core quality. Thoughts?"`,
  },
  
  withTeam: {
    scenario: 'Team wants to refactor everything',
    
    bad: '❌ "PM says we have to ship now. Just make it work."',
    
    good: `✅ "I agree the code needs improvement. Here\'s my thinking:

We have business pressure to ship this month. Rather than perfect refactor,
let\'s refactor just the area we need to touch for this feature. That way:
- Feature ships on time
- We improve code quality where it matters
- We don\'t accumulate more debt
- We can continue refactoring incrementally

Let\'s allocate 15% of our time to ongoing refactoring. Deal?"`,
  },
};
```

---

#### **10. Personal Guidelines**

```typescript
const myApproach = {
  default: 'Aim for Production Quality',
  
  adjustments: [
    {
      situation: 'MVP or prototype',
      adjustment: 'MVP Quality OK if:
        - Tech debt documented
        - Plan to pay back exists
        - Risk is acceptable
        - Stakeholders aware',
    },
    {
      situation: 'Critical path feature',
      adjustment: 'Increase to Critical Quality:
        - More reviews
        - More testing
        - Gradual rollout
        - Monitoring intensive',
    },
  ],
  
  neverCompromise: [
    'Security',
    'Data integrity',
    'User privacy',
    'Core architecture',
    'Error handling',
  ],
  
  alwaysDo: [
    'Code review',
    'Basic testing',
    'Error handling',
    'Logging',
    'Documentation for tricky parts',
  ],
  
  canDefer: [
    'Perfect test coverage',
    'Complete edge cases',
    'Performance optimization (if acceptable)',
    'Polish and animations',
    'Comprehensive documentation',
  ],
};
```

---

#### **Summary**

**Key Principles:**

1. **Context determines quality level** - MVP needs different quality than payment system
2. **Quality is a spectrum, not binary** - Choose appropriate level
3. **Invest quality where it's hard to change** - Core architecture, security, data integrity
4. **Technical debt is a tool** - Use deliberately, pay back systematically
5. **Communication is critical** - Align with stakeholders on trade-offs
6. **Never compromise on security/data** - Some things are non-negotiable
7. **Speed techniques exist** - Use good libraries, narrow scope, timeboxing
8. **Measure and iterate** - Track velocity and quality metrics

**The Balance:**
```
Speed ←→ Quality

Not: "Either fast OR good"
But: "Fast AND good enough for context"

The art is knowing what "good enough" means for each situation.
```

**Decision Framework:**
1. Assess context (MVP? Production? Critical?)
2. Evaluate risk (reversible? blast radius?)
3. Choose appropriate quality level
4. Document deliberate trade-offs
5. Communicate with stakeholders
6. Ship and iterate
7. Pay back tech debt incrementally

**Remember:** Perfect is the enemy of good. Ship good enough, iterate to great.

</details>

---

### 270. How do you handle disagreements about architecture decisions?

<details>
<summary>View Answer</summary>

**Architecture disagreements** are natural and healthy. The key is to **facilitate constructive debate**, **evaluate objectively**, and **build consensus** while making decisions efficiently.

---

#### **1. Fundamental Approach**

```typescript
const approachToDisagreements = {
  mindset: [
    'Disagreements are opportunities, not problems',
    'Everyone wants what\'s best for the project',
    'Different perspectives lead to better decisions',
    'Focus on ideas, not people',
    'Be willing to be wrong',
  ],
  
  goals: [
    'Find the best solution, not "win" the argument',
    'Ensure all perspectives heard',
    'Make decision based on data and reasoning',
    'Build team alignment',
    'Move forward decisively',
  ],
  
  antipatterns: [
    '❌ "I have more experience, so I\'m right"',
    '❌ "We\'ve always done it this way"',
    '❌ "This is my favorite technology"',
    '❌ "Let\'s just do both" (when mutually exclusive)',
    '❌ Letting debate drag on indefinitely',
  ],
};
```

---

#### **2. Structured Debate Framework**

```typescript
const debateProcess = [
  {
    step: 1,
    name: 'Clarify the disagreement',
    duration: '15-30 minutes',
    activities: [
      'State the decision to be made clearly',
      'Identify specific points of disagreement',
      'Understand each position fully',
      'Separate facts from opinions',
    ],
    output: 'Clear understanding of different viewpoints',
  },
  {
    step: 2,
    name: 'Establish evaluation criteria',
    duration: '15 minutes',
    activities: [
      'Agree on what matters (performance? maintainability? time?)',
      'Prioritize criteria',
      'Get stakeholder input if needed',
    ],
    output: 'Shared criteria for evaluating options',
  },
  {
    step: 3,
    name: 'Present arguments',
    duration: '30-60 minutes',
    activities: [
      'Each side presents their case',
      'Use data and examples',
      'Address trade-offs honestly',
      'Listen actively',
    ],
    output: 'Full understanding of each option',
  },
  {
    step: 4,
    name: 'Explore alternatives',
    duration: '20-30 minutes',
    activities: [
      'Brainstorm hybrid solutions',
      'Challenge assumptions',
      'Look for win-win',
    ],
    output: 'Additional options to consider',
  },
  {
    step: 5,
    name: 'Make decision',
    duration: '15 minutes',
    activities: [
      'Evaluate against criteria',
      'Seek consensus if possible',
      'Escalate to decision-maker if needed',
      'Document decision and rationale',
    ],
    output: 'Clear decision with reasoning',
  },
  {
    step: 6,
    name: 'Commit to decision',
    duration: '5 minutes',
    activities: [
      'Everyone commits to supporting decision',
      'Identify success metrics',
      'Set review date',
    ],
    output: 'Team alignment',
  },
];
```

---

#### **3. Real-World Example 1: State Management Choice**

```typescript
const stateManagementDisagreement = {
  context: {
    team: 5,
    project: 'New React application',
    timeline: '3 months to MVP',
  },
  
  disagreement: {
    developer1: {
      position: 'Redux Toolkit',
      reasoning: [
        'Industry standard',
        'Excellent dev tools',
        'Team has experience',
        'Scales well',
      ],
      concerns: ['More boilerplate', 'Slower initial setup'],
    },
    
    developer2: {
      position: 'Zustand',
      reasoning: [
        'Much simpler API',
        'Less boilerplate',
        'Faster to implement',
        'Smaller bundle size',
      ],
      concerns: ['Team unfamiliar', 'Smaller ecosystem'],
    },
    
    developer3: {
      position: 'Just use Context API',
      reasoning: [
        'Built into React',
        'No dependencies',
        'Team already knows it',
      ],
      concerns: ['Performance issues', 'No dev tools'],
    },
  },
  
  resolution: {
    facilitator: 'Tech lead (me)',
    
    process: `
Me: "Great, we have three strong options. Let's work through this systematically.

First, let's agree on what matters most:
1. Team velocity - how fast can we ship features?
2. Maintainability - how easy to maintain long-term?
3. Learning curve - how much time to get productive?
4. Performance - does it meet our needs?

Everyone agree these are the key criteria?"

Team: "Yes"

Me: "Okay, let's score each option:"
    `,
    
    evaluation: [
      {
        option: 'Redux Toolkit',
        teamVelocity: 7,      // Good after initial setup
        maintainability: 9,    // Excellent
        learningCurve: 6,      // Moderate
        performance: 9,        // Excellent
        total: 31,
      },
      {
        option: 'Zustand',
        teamVelocity: 8,       // Fast
        maintainability: 7,    // Good
        learningCurve: 8,      // Easy
        performance: 8,        // Very good
        total: 31,
      },
      {
        option: 'Context API',
        teamVelocity: 7,       // Quick start
        maintainability: 5,    // Gets messy at scale
        learningCurve: 9,      // Already know it
        performance: 5,        // Performance issues
        total: 26,
      },
    ],
    
    discussion: `
Me: "Interesting - Redux and Zustand tied. Context API is out due to performance concerns.

For the tie-breaker, let's consider:
- We're building an MVP, speed matters
- But we expect this to grow significantly
- Team has 2 people with Redux experience

Dev2, you make great points about Zustand's simplicity. For a smaller app, it would be my choice too.

But given we expect significant growth and have Redux experience on the team, I lean toward Redux Toolkit.

However, here's a compromise: Let's build a small feature with Zustand this sprint as a prototype. If the team loves it and it works well, we commit to Zustand. If we hit issues or the learning curve is steeper than expected, we go with Redux.

This gives us real experience to make an informed decision. Sound fair?"

Team: "That works."

[After prototype]

Me: "Okay, we tried Zustand. Feedback?"

Dev2: "It works great, super simple."

Dev1: "I see the appeal, but I'm worried about debugging without Redux DevTools."

Dev4: "Actually, Zustand has DevTools integration too. I set it up."

Dev1: "Oh! That changes things. Okay, I'm on board."

Me: "Sounds like we have consensus. Zustand it is. Everyone commits to this?"

Team: "Yes."

Me: "Great. I'll document the decision. We'll revisit in 3 months to see if it's working well."
    `,
    
    outcome: {
      decision: 'Zustand',
      rationale: 'Prototype showed it meets our needs with better DX',
      commitment: 'Full team alignment',
      review: 'Scheduled for 3 months',
    },
  },
};
```

---

#### **4. Real-World Example 2: Component Architecture**

```typescript
const componentArchitectureDebate = {
  context: {
    situation: 'Building design system',
    disagreement: 'Compound components vs prop-heavy components',
  },
  
  positions: {
    senior1: {
      approach: 'Compound components',
      example: `
// Compound component pattern
<Select>
  <Select.Trigger>Choose option</Select.Trigger>
  <Select.Content>
    <Select.Item value="1">Option 1</Select.Item>
    <Select.Item value="2">Option 2</Select.Item>
  </Select.Content>
</Select>
      `,
      pros: ['Flexible', 'Composable', 'Clear structure'],
      cons: ['More verbose', 'Harder to enforce conventions'],
    },
    
    senior2: {
      approach: 'Props-based',
      example: `
// Props-based pattern
<Select
  placeholder="Choose option"
  options={[
    { value: '1', label: 'Option 1' },
    { value: '2', label: 'Option 2' },
  ]}
/>
      `,
      pros: ['Concise', 'Type-safe', 'Easy to use'],
      cons: ['Less flexible', 'Props explosion for complex cases'],
    },
  },
  
  resolution: {
    approach: 'Data-driven decision',
    
    experiment: `
Me: "Both have merit. Let's do this:

1. Implement both patterns for Select component
2. Have 3 team members use each version for a week
3. Gather feedback on:
   - Ease of use
   - Flexibility for different use cases
   - Code readability
   - Type safety

Then we decide based on real experience."
    `,
    
    results: {
      feedback: [
        'Compound components more flexible for complex cases',
        'Props-based faster for simple cases',
        'Compound components harder to misuse',
        'Props-based had TypeScript issues with many props',
      ],
      
      decision: 'Hybrid approach',
      
      solution: `
// Simple cases: props-based shorthand
<Select options={simpleOptions} />

// Complex cases: compound components
<Select>
  <Select.Trigger>
    <CustomIcon /> Choose option
  </Select.Trigger>
  <Select.Content>
    <Select.Group label="Group 1">
      <Select.Item value="1">Option 1</Select.Item>
    </Select.Group>
  </Select.Content>
</Select>

// Best of both worlds!
      `,
    },
    
    outcome: 'Both seniors happy with compromise, team benefits from flexibility',
  },
};
```

---

#### **5. Conflict Resolution Techniques**

```typescript
const resolutionTechniques = [
  {
    technique: 'Prototype',
    when: 'Both options seem viable',
    how: 'Build small version of each, compare',
    benefit: 'Real data beats theory',
    example: 'Try both state management libraries for a week',
  },
  {
    technique: 'Time-box decision',
    when: 'Discussion becoming circular',
    how: 'Set deadline, make decision by then',
    benefit: 'Prevents analysis paralysis',
    example: '"Let\'s decide by end of day Friday"',
  },
  {
    technique: 'Escalate',
    when: 'Can\'t reach consensus',
    how: 'Senior engineer/architect makes call',
    benefit: 'Unblocks team',
    example: 'CTO makes final decision on architecture',
  },
  {
    technique: 'Defer',
    when: 'Decision not time-sensitive',
    how: 'Gather more data, revisit later',
    benefit: 'Better informed decision',
    example: 'Wait for library to mature before adopting',
  },
  {
    technique: 'Compromise',
    when: 'Both sides have valid points',
    how: 'Find middle ground or hybrid',
    benefit: 'Buy-in from both sides',
    example: 'Use Redux for complex state, Context for simple',
  },
  {
    technique: 'Vote',
    when: 'Team is split, no clear winner',
    how: 'Democratic decision',
    benefit: 'Team ownership of decision',
    example: 'Team votes on testing library to use',
  },
];
```

---

#### **6. Communication Strategies**

```typescript
const communicationStrategies = {
  activeListening: {
    techniques: [
      'Paraphrase to confirm understanding',
      'Ask clarifying questions',
      'Don\'t interrupt',
      'Acknowledge valid points',
    ],
    
    example: `
"Let me make sure I understand your concern. You're worried that Redux will slow down our initial development because of the setup time and boilerplate. Is that right?"

"That's a fair point. The initial setup does take longer."
    `,
  },
  
  steelmanning: {
    definition: 'Present the strongest version of the opposing argument',
    benefit: 'Shows respect, finds common ground',
    
    example: `
"The argument for Zustand is actually quite strong:
- It genuinely is simpler to set up
- The API is more intuitive
- For our MVP timeline, speed matters
- The smaller bundle size is a real benefit

These are all valid points. My concern is mainly about..."
    `,
  },
  
  dataOverOpinions: {
    approach: 'Ground discussion in facts',
    
    examples: [
      'Show bundle size comparisons',
      'Benchmark performance',
      'Compare API surfaces',
      'Reference case studies',
      'Look at GitHub issues/activity',
    ],
    
    example: `
"Let's look at the data:
- Redux: 2.6KB gzipped
- Zustand: 1.2KB gzipped
- Context: 0KB (built-in)

For our 200KB bundle, this 1.4KB difference is <1%. So bundle size shouldn't be the deciding factor."
    `,
  },
  
  focusOnGoals: {
    approach: 'Align on shared objectives',
    
    example: `
"We all want the same thing:
- Ship MVP in 3 months
- Maintainable codebase
- Good developer experience
- App that performs well

The question is which option best achieves these goals. Let's evaluate each option against these criteria."
    `,
  },
};
```

---

#### **7. When I'm Wrong**

```typescript
const whenImWrong = {
  signs: [
    'Team consistently pushing back',
    'Data doesn\'t support my position',
    'Other engineers raise valid concerns I can\'t address',
    'My reasoning is based on outdated knowledge',
  ],
  
  response: {
    acknowledge: `
"You know what, you're right. I was thinking about this wrong.

I was anchored on Redux because that's what I've used successfully before. But looking at our specific needs and the data we've gathered, Zustand is clearly the better choice here.

Thanks for pushing back and helping me see this more clearly. Let's go with Zustand."
    `,
    
    principles: [
      'Admit when wrong quickly',
      'Thank others for their input',
      'Model intellectual humility',
      'Focus on best outcome, not ego',
    ],
  },
  
  impact: {
    teamCulture: 'Builds trust and psychological safety',
    futureDebates: 'Team more willing to challenge ideas',
    personalGrowth: 'Learn and improve',
  },
};
```

---

#### **8. Decision Documentation**

```typescript
// Architecture Decision Record (ADR)
const adrTemplate = `
# ADR-005: State Management with Zustand

## Status
Accepted (Supersedes ADR-003: Context API)

## Context
We need a state management solution for our React application.
Initially considered Redux Toolkit, Zustand, and Context API.

Key requirements:
- Fast to implement (3-month MVP timeline)
- Good developer experience
- Scalable as app grows
- Good TypeScript support

## Decision
We will use Zustand for global state management.

## Participants
- Senior Engineer 1 (advocated Redux)
- Senior Engineer 2 (advocated Zustand)
- Junior Engineers 1-3
- Tech Lead (facilitator)

## Evaluation Process
1. Initial discussion identified three options
2. Agreed on evaluation criteria
3. Built prototype with Zustand
4. Team used prototype for 1 week
5. Gathered feedback
6. Made decision based on experience

## Rationale
After prototyping, Zustand proved to be:
- Much faster to set up (2 hours vs 1 day for Redux)
- Simpler API led to faster feature development
- DevTools integration addressed debugging concern
- Sufficient for our current and near-future needs
- Team picked it up quickly (<1 day learning curve)

While Redux Toolkit would scale to larger apps, Zustand meets our MVP needs with better DX. If we outgrow it, we can migrate (estimated 2 weeks effort).

## Consequences

### Positive
- Faster feature development
- Better developer experience
- Smaller bundle size
- Quick team onboarding

### Negative
- Smaller ecosystem than Redux
- Less familiar to new hires
- May need to migrate if app becomes very complex

### Mitigations
- Document our usage patterns
- Keep state logic in custom hooks (easier to migrate)
- Re-evaluate at 10K LOC (estimated 6 months)

## Alternatives Considered

### Redux Toolkit
- More established, larger ecosystem
- Rejected: Slower initial development for MVP needs
- May revisit if app complexity grows significantly

### Context API
- Built-in, zero dependencies
- Rejected: Performance issues, no dev tools

## Implementation Notes
- Co-locate stores with features
- Use TypeScript for all stores
- Document store structure in README

## Success Metrics
- Feature development velocity
- Developer satisfaction survey
- App performance metrics

## Review Date
January 2026 (3 months after MVP launch)

## Discussion Thread
https://github.com/company/project/discussions/123
`;
```

---

#### **9. Building Consensus**

```typescript
const consensusBuilding = {
  beforeDebate: [
    'Create safe environment for disagreement',
    'Set expectation that debate is healthy',
    'Establish decision-making process upfront',
    'Agree on evaluation criteria',
  ],
  
  duringDebate: [
    'Ensure everyone\'s voice is heard',
    'Focus on ideas, not people',
    'Use data to inform discussion',
    'Look for win-win solutions',
    'Timebox discussions',
  ],
  
  afterDecision: [
    'Document decision and rationale',
    'Get explicit commitment from all',
    'Thank everyone for their input',
    'Set review date',
    'Move forward united',
  ],
  
  disagreeAndCommit: {
    principle: 'OK to disagree, but commit to decision once made',
    
    example: `
"I still think Redux would have been better for the long term, but I respect the decision we made as a team. I'm fully committed to making Zustand work well for us."
    `,
  },
};
```

---

#### **10. Red Flags to Avoid**

```typescript
const redFlags = [
  {
    behavior: 'Appeal to authority',
    example: '❌ "I\'m the senior engineer, we\'re doing it my way"',
    why: 'Shuts down discussion, builds resentment',
    instead: '✅ "Here\'s my reasoning... what do you think?"',
  },
  {
    behavior: 'Personal attacks',
    example: '❌ "You only want Zustand because it\'s trendy"',
    why: 'Makes it personal, damages relationships',
    instead: '✅ "What are the technical merits of Zustand?"',
  },
  {
    behavior: 'Indefinite debate',
    example: '❌ Debating same points for weeks',
    why: 'Blocks progress, team frustration',
    instead: '✅ Timebox, prototype, or escalate',
  },
  {
    behavior: 'Hidden agendas',
    example: '❌ Pushing technology for resume building',
    why: 'Not aligned with project needs',
    instead: '✅ Honest about motivations',
  },
  {
    behavior: 'Not listening',
    example: '❌ Just waiting to talk, not hearing others',
    why: 'Misses good ideas, shows disrespect',
    instead: '✅ Active listening, paraphrasing',
  },
];
```

---

#### **Summary**

**Key Principles:**

1. **Embrace disagreement** - It leads to better decisions
2. **Listen actively** - Understand before responding
3. **Use data** - Facts over opinions
4. **Focus on goals** - What's best for the project?
5. **Be humble** - Be willing to be wrong
6. **Facilitate, don't dictate** - Guide discussion
7. **Document decisions** - Clear reasoning for future
8. **Commit to outcome** - United front after decision

**Process:**
1. Clarify the disagreement
2. Establish evaluation criteria
3. Present all arguments
4. Prototype if helpful
5. Make decision based on data
6. Document thoroughly
7. Get team commitment
8. Review and iterate

**Communication:**
- "Help me understand your thinking..."
- "That's a valid point. My concern is..."
- "Let's look at the data..."
- "What if we tried..."
- "I hear you. Here's what I'm thinking..."

**Remember:**
- Best idea should win, regardless of who suggested it
- Disagreement is about ideas, not people
- Today's disagreement is tomorrow's collaboration
- Good process leads to good decisions AND team cohesion

Healthy debate makes **better products** and **stronger teams**.

</details>
