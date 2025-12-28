# SSR & SSG

> Advanced / Senior Level (3-5 years)

---

## Questions

141. What is Server-Side Rendering (SSR)?
142. What is Static Site Generation (SSG)?
143. What is the difference between SSR and SSG?
144. What is Next.js and why is it popular?
145. What is the App Router in Next.js 13+?
146. What are Server Actions in Next.js?
147. What is Incremental Static Regeneration (ISR)?
148. How does Next.js handle data fetching?
149. What is Remix framework and how does it differ from Next.js?
150. What is hydration in SSR?

---

## Detailed Answers

### 141. What is Server-Side Rendering (SSR)?

<details>
<summary>View Answer</summary>

**Server-Side Rendering (SSR)** is a technique where React components are rendered to HTML on the server for each request, then sent to the browser. This provides faster initial page loads and better SEO compared to traditional client-side rendering.

#### 1. How SSR Works

**Traditional Client-Side Rendering (CSR):**
```html
<!-- 1. Server sends minimal HTML -->
<!DOCTYPE html>
<html>
  <head><title>My App</title></head>
  <body>
    <div id="root"></div>
    <script src="bundle.js"></script>
  </body>
</html>

<!-- 2. Browser downloads and executes JS -->
<!-- 3. React renders content client-side -->
<!-- 4. User sees content (slow initial load) -->
```

**Server-Side Rendering:**
```html
<!-- 1. Server renders React to HTML -->
<!DOCTYPE html>
<html>
  <head><title>My App</title></head>
  <body>
    <div id="root">
      <!-- Full HTML content already rendered! -->
      <div class="app">
        <h1>Welcome</h1>
        <p>This content is already rendered</p>
      </div>
    </div>
    <script src="bundle.js"></script>
  </body>
</html>

<!-- 2. User sees content immediately (fast!) -->
<!-- 3. React "hydrates" the existing HTML -->
<!-- 4. App becomes interactive -->
```

**SSR Flow Diagram:**
```typescript
/*
1. User Request
   |
   v
2. Server receives request
   |
   v
3. Server fetches data (API, database)
   |
   v
4. Server renders React components to HTML
   |
   v
5. Server sends fully rendered HTML + JS bundles
   |
   v
6. Browser displays HTML (content visible)
   |
   v
7. Browser downloads and executes JS
   |
   v
8. React "hydrates" HTML (attaches event listeners)
   |
   v
9. App is fully interactive
*/
```

#### 2. Benefits of SSR

**1. Faster First Contentful Paint (FCP):**
```typescript
// CSR: User sees blank page until JS loads
// Timeline: 0ms -> 3000ms (blank) -> Content

// SSR: User sees content immediately
// Timeline: 0ms -> Content -> 500ms (interactive)

// Result: Better perceived performance
```

**2. Better SEO:**
```typescript
// CSR: Search engine crawlers may not execute JS
<div id="root"></div> // Empty for crawlers!

// SSR: Crawlers see full HTML content
<div id="root">
  <h1>Product Name</h1>
  <p>Product description...</p>
  <meta name="description" content="..." />
</div>

// Result: Better search engine rankings
```

**3. Social Media Previews:**
```html
<!-- SSR: Open Graph tags rendered server-side -->
<head>
  <meta property="og:title" content="My Product" />
  <meta property="og:description" content="Check out this product" />
  <meta property="og:image" content="https://example.com/image.jpg" />
</head>

<!-- When shared on Twitter/Facebook, shows rich preview -->
```

**4. Performance on Slow Devices:**
```typescript
// CSR: Slow devices struggle with JS parsing/execution
// Low-end phone: 5-10 seconds to interactive

// SSR: Content visible before JS parsing
// Low-end phone: Content visible in 1-2 seconds
//                Interactive in 5-10 seconds

// Users can start reading content while JS loads
```

#### 3. Basic SSR Implementation

**Simple Node.js SSR:**
```typescript
// server.js
import express from 'express';
import React from 'react';
import { renderToString } from 'react-dom/server';
import App from './App';

const app = express();

app.get('*', (req, res) => {
  // 1. Render React component to HTML string
  const html = renderToString(<App />);
  
  // 2. Send HTML response
  res.send(`
    <!DOCTYPE html>
    <html>
      <head>
        <title>SSR App</title>
      </head>
      <body>
        <div id="root">${html}</div>
        <script src="/bundle.js"></script>
      </body>
    </html>
  `);
});

app.listen(3000);
```

**Client-Side Hydration:**
```typescript
// client.js
import React from 'react';
import { hydrateRoot } from 'react-dom/client';
import App from './App';

// Hydrate instead of render
const root = document.getElementById('root');
hydrateRoot(root, <App />);

// React attaches event listeners to existing HTML
// Doesn't re-render, just makes interactive
```

#### 4. SSR with Data Fetching

**Server-Side Data Fetching:**
```typescript
// UserProfile.tsx
interface UserProfileProps {
  user: {
    id: string;
    name: string;
    email: string;
  };
}

function UserProfile({ user }: UserProfileProps) {
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}

export default UserProfile;

// server.js
import express from 'express';
import { renderToString } from 'react-dom/server';
import UserProfile from './UserProfile';

const app = express();

app.get('/user/:id', async (req, res) => {
  // 1. Fetch data on server
  const response = await fetch(`https://api.example.com/users/${req.params.id}`);
  const user = await response.json();
  
  // 2. Render component with data
  const html = renderToString(<UserProfile user={user} />);
  
  // 3. Send HTML with embedded data
  res.send(`
    <!DOCTYPE html>
    <html>
      <head>
        <title>${user.name}'s Profile</title>
        <meta name="description" content="Profile of ${user.name}" />
      </head>
      <body>
        <div id="root">${html}</div>
        <script>
          window.__INITIAL_DATA__ = ${JSON.stringify({ user })};
        </script>
        <script src="/bundle.js"></script>
      </body>
    </html>
  `);
});
```

**Client Hydration with Initial Data:**
```typescript
// client.js
import { hydrateRoot } from 'react-dom/client';
import UserProfile from './UserProfile';

// Get initial data from server
const initialData = window.__INITIAL_DATA__;

// Hydrate with same data
hydrateRoot(
  document.getElementById('root'),
  <UserProfile user={initialData.user} />
);
```

#### 5. Next.js SSR Example

**Using getServerSideProps (Pages Router):**
```typescript
// pages/user/[id].tsx
import { GetServerSideProps } from 'next';

interface User {
  id: string;
  name: string;
  email: string;
}

interface UserPageProps {
  user: User;
}

// This page uses SSR
export default function UserPage({ user }: UserPageProps) {
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
      <p>ID: {user.id}</p>
    </div>
  );
}

// Runs on every request (server-side only)
export const getServerSideProps: GetServerSideProps<UserPageProps> = async (context) => {
  const { id } = context.params!;
  
  // Fetch data on server
  const res = await fetch(`https://api.example.com/users/${id}`);
  const user = await res.json();
  
  // Pass data to page component
  return {
    props: {
      user,
    },
  };
};
```

**Using App Router (Next.js 13+):**
```typescript
// app/user/[id]/page.tsx

// Server Component by default (SSR)
interface PageProps {
  params: { id: string };
}

async function getUser(id: string) {
  const res = await fetch(`https://api.example.com/users/${id}`, {
    cache: 'no-store', // Force SSR (no caching)
  });
  return res.json();
}

export default async function UserPage({ params }: PageProps) {
  // Data fetching directly in component
  const user = await getUser(params.id);
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
      <p>ID: {user.id}</p>
    </div>
  );
}

// Generate metadata (SSR)
export async function generateMetadata({ params }: PageProps) {
  const user = await getUser(params.id);
  
  return {
    title: `${user.name}'s Profile`,
    description: `Profile page for ${user.name}`,
    openGraph: {
      title: user.name,
      description: user.email,
    },
  };
}
```

#### 6. Performance Considerations

**Streaming SSR (React 18+):**
```typescript
// server.js with streaming
import { renderToPipeableStream } from 'react-dom/server';
import { Suspense } from 'react';

app.get('*', (req, res) => {
  const { pipe } = renderToPipeableStream(
    <html>
      <head>
        <title>Streaming SSR</title>
      </head>
      <body>
        <div id="root">
          <Suspense fallback={<div>Loading...</div>}>
            <SlowComponent />
          </Suspense>
        </div>
      </body>
    </html>,
    {
      onShellReady() {
        // Send initial HTML immediately
        res.setHeader('Content-Type', 'text/html');
        pipe(res);
      },
    }
  );
});

// Benefits:
// - Send HTML progressively
// - Show fallbacks while components load
// - Faster Time to First Byte (TTFB)
```

**Caching Strategies:**
```typescript
// Cache SSR responses
import express from 'express';
import NodeCache from 'node-cache';

const cache = new NodeCache({ stdTTL: 600 }); // 10 minutes

app.get('/user/:id', async (req, res) => {
  const cacheKey = `user-${req.params.id}`;
  
  // Check cache first
  const cached = cache.get(cacheKey);
  if (cached) {
    return res.send(cached);
  }
  
  // Fetch and render
  const user = await fetchUser(req.params.id);
  const html = renderPage(user);
  
  // Cache response
  cache.set(cacheKey, html);
  
  res.send(html);
});

// Reduces server load for popular pages
```

**Code Splitting for SSR:**
```typescript
// Use React.lazy for code splitting
import { lazy, Suspense } from 'react';

const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <div>
      <h1>My App</h1>
      <Suspense fallback={<div>Loading...</div>}>
        <HeavyComponent />
      </Suspense>
    </div>
  );
}

// Server sends minimal HTML initially
// Heavy component loads separately
```

#### 7. Common Challenges

**Challenge 1: Browser APIs Not Available**
```typescript
// ❌ Bad: window/document not available on server
function MyComponent() {
  const width = window.innerWidth; // Error on server!
  return <div>Width: {width}</div>;
}

// ✅ Good: Check if in browser
function MyComponent() {
  const [width, setWidth] = useState(0);
  
  useEffect(() => {
    // Only runs in browser
    setWidth(window.innerWidth);
  }, []);
  
  return <div>Width: {width || 'Unknown'}</div>;
}

// ✅ Alternative: Use client-side only rendering
'use client'; // Next.js 13+ directive

function MyComponent() {
  const width = window.innerWidth; // OK now
  return <div>Width: {width}</div>;
}
```

**Challenge 2: Different Server/Client State**
```typescript
// ❌ Bad: Hydration mismatch
function CurrentTime() {
  // Server renders one time, client renders another
  return <div>{new Date().toISOString()}</div>;
}

// ✅ Good: Consistent initial state
function CurrentTime({ initialTime }: { initialTime: string }) {
  const [time, setTime] = useState(initialTime);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setTime(new Date().toISOString());
    }, 1000);
    return () => clearInterval(interval);
  }, []);
  
  return <div>{time}</div>;
}

// Server passes initial time
export const getServerSideProps = async () => {
  return {
    props: {
      initialTime: new Date().toISOString(),
    },
  };
};
```

**Challenge 3: Increased Server Load**
```typescript
// SSR requires server resources for every request

// Mitigation strategies:
// 1. Caching (Redis, in-memory cache)
// 2. CDN caching for semi-static content
// 3. Selective SSR (only for important pages)
// 4. Consider SSG for static content
// 5. Load balancing and horizontal scaling
```

#### 8. When to Use SSR

**Use SSR When:**
```typescript
// ✅ Content changes frequently
// - News sites
// - Social media feeds
// - Real-time dashboards
// - Personalized content

// ✅ SEO is critical
// - E-commerce product pages
// - Blog posts
// - Marketing pages

// ✅ Need latest data on every request
// - Stock prices
// - Sports scores
// - User profiles

// Example: E-commerce product page
export const getServerSideProps = async ({ params }) => {
  // Always fetch latest product data
  const product = await fetchProduct(params.id);
  const inventory = await checkInventory(params.id);
  
  return {
    props: { product, inventory },
  };
};
```

**Avoid SSR When:**
```typescript
// ❌ Content is static
// - Documentation sites
// - Landing pages
// - Blog posts (use SSG instead)

// ❌ Highly interactive applications
// - Drawing apps
// - Games
// - Real-time collaborative tools
// (Use CSR instead)

// ❌ Private/authenticated content only
// - Admin dashboards
// - User-specific views
// (CSR with authentication is simpler)
```

#### 9. SSR Frameworks

**Popular SSR Frameworks:**
```typescript
// Next.js (React)
// - Most popular
// - Great DX
// - Built-in routing
// - Hybrid SSR/SSG

// Remix (React)
// - Modern approach
// - Nested routing
// - Progressive enhancement

// Gatsby (React)
// - SSG focused
// - Can do SSR with gatsby-plugin-ssr

// Nuxt.js (Vue)
// - Vue equivalent of Next.js

// SvelteKit (Svelte)
// - Svelte framework
```

#### 10. Best Practices

**1. Minimize Server Rendering Time:**
```typescript
// ✅ Good: Fetch data in parallel
export const getServerSideProps = async () => {
  const [user, posts, comments] = await Promise.all([
    fetchUser(),
    fetchPosts(),
    fetchComments(),
  ]);
  
  return { props: { user, posts, comments } };
};

// ❌ Bad: Sequential fetching
export const getServerSideProps = async () => {
  const user = await fetchUser();
  const posts = await fetchPosts();
  const comments = await fetchComments();
  // Slow!
};
```

**2. Handle Errors Gracefully:**
```typescript
export const getServerSideProps = async ({ params }) => {
  try {
    const data = await fetchData(params.id);
    return { props: { data } };
  } catch (error) {
    // Return notFound or error page
    return {
      notFound: true,
      // Or: redirect: { destination: '/error', permanent: false }
    };
  }
};
```

**3. Use Appropriate Caching:**
```typescript
// Set cache headers
export const getServerSideProps = async ({ res }) => {
  // Cache for 1 minute, revalidate in background
  res.setHeader(
    'Cache-Control',
    'public, s-maxage=60, stale-while-revalidate=120'
  );
  
  const data = await fetchData();
  return { props: { data } };
};
```

**4. Monitor Performance:**
```typescript
// Track SSR metrics
export const getServerSideProps = async () => {
  const startTime = Date.now();
  
  const data = await fetchData();
  
  const duration = Date.now() - startTime;
  console.log(`SSR took ${duration}ms`);
  
  // Send to monitoring service
  analytics.track('ssr_duration', duration);
  
  return { props: { data } };
};
```

#### Summary

**Server-Side Rendering (SSR):**
- Renders React components to HTML on the server for each request
- Provides faster initial page loads and better SEO
- HTML is sent to browser, then React "hydrates" it
- Content is visible before JavaScript loads

**Key Benefits:**
1. ⚡ **Fast First Paint**: Content visible immediately
2. 🔍 **Better SEO**: Search engines see full HTML
3. 📱 **Social Previews**: Rich cards on social media
4. 📊 **Analytics**: Better for performance metrics

**Key Challenges:**
1. 💻 **Server Load**: Requires server resources
2. ⏱️ **TTFB**: Can be slower than static content
3. 🧩 **Complexity**: More complex than CSR
4. 🚫 **Browser APIs**: Not available on server

**When to Use:**
- Dynamic content that changes frequently
- SEO-critical pages
- Pages needing social media previews
- Content requiring personalization

**Modern Approach:**
Use frameworks like Next.js or Remix that handle SSR complexity, provide hybrid rendering options, and optimize performance automatically.

</details>

---

### 142. What is Static Site Generation (SSG)?

<details>
<summary>View Answer</summary>

**Static Site Generation (SSG)** is a technique where React pages are pre-rendered to HTML at **build time**, creating static files that can be served instantly without server-side processing. This provides the best performance and scalability.

#### 1. How SSG Works

**Build-Time Rendering:**
```typescript
/*
SSG Build Process:

1. Developer runs build command
   npm run build
   |
   v
2. Framework fetches all data
   - API calls
   - Database queries
   - CMS content
   |
   v
3. Renders all pages to static HTML
   - about.html
   - blog/post-1.html
   - blog/post-2.html
   - products/1.html
   |
   v
4. Generates static assets
   - HTML files
   - CSS files
   - JavaScript bundles
   |
   v
5. Deploy to CDN/hosting
   - Vercel, Netlify, AWS S3
   |
   v
6. User requests page
   - Instant HTML delivery
   - No server processing needed
*/
```

**Request Flow:**
```typescript
// User requests: example.com/blog/hello-world

// SSG: Static file already exists
1. CDN delivers pre-built HTML instantly
2. Browser displays content (blazing fast!)
3. React hydrates page
4. Page becomes interactive

// No server computation needed!
// No database queries!
// Just static file delivery!
```

#### 2. Benefits of SSG

**1. Exceptional Performance:**
```typescript
// SSG delivers pre-built HTML from CDN
// - Time to First Byte: ~10-50ms
// - First Contentful Paint: ~100-300ms
// - Largest Contentful Paint: ~500-1000ms

// Compare with:
// SSR: 500-2000ms (server processing)
// CSR: 1000-3000ms (JS download + execution)

// Example Lighthouse scores:
// SSG: 100/100 Performance
// SSR: 80-95/100
// CSR: 50-70/100
```

**2. Lower Costs:**
```typescript
// SSG: Static files on CDN
// - No server needed (or minimal)
// - No compute costs per request
// - Dirt cheap hosting ($0-20/month)
// - Handles massive traffic easily

// SSR: Server for every request
// - Server costs scale with traffic
// - Database costs
// - $50-500+/month depending on traffic
```

**3. Better Scalability:**
```typescript
// SSG: Files served from CDN
// - Can handle millions of requests
// - No server bottleneck
// - Automatically scales globally
// - No "hug of death"

// SSR: Server capacity limited
// - Need load balancing
// - Horizontal scaling required
// - Database can become bottleneck
```

**4. Enhanced Security:**
```typescript
// SSG: Static files only
// - No server to hack
// - No database to exploit
// - No runtime vulnerabilities
// - Just HTML, CSS, JS

// SSR: Server attack surface
// - Server vulnerabilities
// - Database injection risks
// - Runtime exploits possible
```

**5. Excellent SEO:**
```typescript
// SSG: Perfect for SEO
// - Pre-rendered HTML
// - Fast page loads (ranking factor)
// - All content visible to crawlers
// - Optimal meta tags
```

#### 3. Basic SSG Implementation

**Next.js Static Generation:**
```typescript
// pages/blog/[slug].tsx
import { GetStaticProps, GetStaticPaths } from 'next';

interface Post {
  slug: string;
  title: string;
  content: string;
  date: string;
}

interface BlogPostProps {
  post: Post;
}

// Page component
export default function BlogPost({ post }: BlogPostProps) {
  return (
    <article>
      <h1>{post.title}</h1>
      <time>{post.date}</time>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  );
}

// Tell Next.js which paths to pre-render
export const getStaticPaths: GetStaticPaths = async () => {
  // Fetch all post slugs at build time
  const posts = await fetchAllPosts();
  
  const paths = posts.map(post => ({
    params: { slug: post.slug },
  }));
  
  return {
    paths,
    fallback: false, // 404 for other routes
  };
};

// Fetch data for each page at build time
export const getStaticProps: GetStaticProps<BlogPostProps> = async ({ params }) => {
  const post = await fetchPost(params!.slug as string);
  
  return {
    props: {
      post,
    },
  };
};
```

**Build Output:**
```bash
# When you run: npm run build

# Next.js generates:
.next/
  static/
    blog/
      hello-world.html       # Pre-rendered
      react-tutorial.html    # Pre-rendered
      nextjs-guide.html      # Pre-rendered

# Each HTML file is complete and ready to serve
# No server processing needed!
```

#### 4. SSG with Dynamic Routes

**Product Pages Example:**
```typescript
// pages/products/[id].tsx
import { GetStaticProps, GetStaticPaths } from 'next';

interface Product {
  id: string;
  name: string;
  description: string;
  price: number;
  image: string;
}

interface ProductPageProps {
  product: Product;
}

export default function ProductPage({ product }: ProductPageProps) {
  return (
    <div>
      <img src={product.image} alt={product.name} />
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <p>${product.price}</p>
      <button>Add to Cart</button>
    </div>
  );
}

// Generate paths for all products at build time
export const getStaticPaths: GetStaticPaths = async () => {
  // Fetch from database, API, or CMS
  const products = await fetch('https://api.example.com/products')
    .then(res => res.json());
  
  const paths = products.map((product: Product) => ({
    params: { id: product.id },
  }));
  
  return {
    paths,
    fallback: 'blocking', // Generate new pages on demand
  };
};

// Fetch data for each product
export const getStaticProps: GetStaticProps<ProductPageProps> = async ({ params }) => {
  const product = await fetch(`https://api.example.com/products/${params!.id}`)
    .then(res => res.json());
  
  if (!product) {
    return { notFound: true };
  }
  
  return {
    props: { product },
    revalidate: 3600, // Revalidate every hour (ISR)
  };
};
```

#### 5. SSG with Next.js App Router

**Server Components (Default SSG):**
```typescript
// app/blog/[slug]/page.tsx

interface Post {
  slug: string;
  title: string;
  content: string;
}

interface PageProps {
  params: { slug: string };
}

// This is a Server Component (runs at build time for SSG)
export default async function BlogPost({ params }: PageProps) {
  // Fetch data at build time
  const post = await fetch(`https://api.example.com/posts/${params.slug}`)
    .then(res => res.json());
  
  return (
    <article>
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  );
}

// Generate static params at build time
export async function generateStaticParams() {
  const posts = await fetch('https://api.example.com/posts')
    .then(res => res.json());
  
  return posts.map((post: Post) => ({
    slug: post.slug,
  }));
}

// Generate metadata
export async function generateMetadata({ params }: PageProps) {
  const post = await fetch(`https://api.example.com/posts/${params.slug}`)
    .then(res => res.json());
  
  return {
    title: post.title,
    description: post.excerpt,
  };
}
```

#### 6. Fallback Strategies

**fallback: false**
```typescript
export const getStaticPaths = async () => {
  const paths = [{ params: { id: '1' } }, { params: { id: '2' } }];
  
  return {
    paths,
    fallback: false, // Any other route returns 404
  };
};

// Use when:
// - Small number of pages
// - All pages known at build time
// - No new pages will be added
```

**fallback: true**
```typescript
export default function ProductPage({ product }: ProductPageProps) {
  const router = useRouter();
  
  // Show loading state while generating
  if (router.isFallback) {
    return <div>Loading...</div>;
  }
  
  return <div>{product.name}</div>;
}

export const getStaticPaths = async () => {
  // Only pre-render popular products
  const popularProducts = await fetchPopularProducts();
  
  const paths = popularProducts.map(p => ({ params: { id: p.id } }));
  
  return {
    paths,
    fallback: true, // Generate other pages on-demand
  };
};

// Use when:
// - Large number of pages
// - Can't pre-render all at build time
// - OK with showing loading state
```

**fallback: 'blocking'**
```typescript
export const getStaticPaths = async () => {
  const paths = await getPopularPaths();
  
  return {
    paths,
    fallback: 'blocking', // Wait for page generation
  };
};

// Use when:
// - Large number of pages
// - Don't want loading state
// - First request waits, subsequent cached
// - Better UX than fallback: true
```

#### 7. Data Sources for SSG

**From API:**
```typescript
export const getStaticProps = async () => {
  const res = await fetch('https://api.example.com/posts');
  const posts = await res.json();
  
  return {
    props: { posts },
  };
};
```

**From CMS (Contentful, Strapi):**
```typescript
import { createClient } from 'contentful';

const client = createClient({
  space: process.env.CONTENTFUL_SPACE_ID!,
  accessToken: process.env.CONTENTFUL_ACCESS_TOKEN!,
});

export const getStaticProps = async () => {
  const entries = await client.getEntries({
    content_type: 'blogPost',
  });
  
  return {
    props: {
      posts: entries.items,
    },
  };
};
```

**From Markdown Files:**
```typescript
import fs from 'fs';
import path from 'path';
import matter from 'gray-matter';

export const getStaticProps = async () => {
  const postsDirectory = path.join(process.cwd(), 'posts');
  const filenames = fs.readdirSync(postsDirectory);
  
  const posts = filenames.map(filename => {
    const filePath = path.join(postsDirectory, filename);
    const fileContents = fs.readFileSync(filePath, 'utf8');
    const { data, content } = matter(fileContents);
    
    return {
      slug: filename.replace('.md', ''),
      ...data,
      content,
    };
  });
  
  return {
    props: { posts },
  };
};
```

**From Database:**
```typescript
import { prisma } from '@/lib/prisma';

export const getStaticProps = async () => {
  const posts = await prisma.post.findMany({
    include: {
      author: true,
    },
    orderBy: {
      createdAt: 'desc',
    },
  });
  
  return {
    props: {
      posts: JSON.parse(JSON.stringify(posts)),
    },
  };
};
```

#### 8. When to Use SSG

**Perfect for SSG:**
```typescript
// ✅ Marketing/Landing pages
// - Content rarely changes
// - SEO critical
// - Performance critical

// ✅ Blog/Documentation
// - Content-focused
// - Updated infrequently
// - High read, low write

// ✅ E-commerce product catalogs
// - Product pages
// - Category pages
// - With ISR for updates

// ✅ Portfolio websites
// - Showcase work
// - Fast loading essential

// ✅ News/Magazine sites
// - Article pages
// - With ISR for new content
```

**Not Ideal for SSG:**
```typescript
// ❌ Real-time data
// - Stock prices
// - Sports scores
// - Live dashboards
// (Use SSR or CSR)

// ❌ Personalized content
// - User-specific dashboards
// - Private user data
// - Per-user recommendations
// (Use SSR or CSR with auth)

// ❌ Frequently changing content
// - Social media feeds
// - Chat applications
// - Live auctions
// (Use SSR or CSR with real-time updates)

// ❌ Millions of pages
// - User-generated content at scale
// - Build times become impractical
// (Use SSR or ISR)
```

#### 9. Optimizing SSG Builds

**Parallel Data Fetching:**
```typescript
export const getStaticProps = async () => {
  // ✅ Good: Fetch in parallel
  const [posts, categories, tags] = await Promise.all([
    fetchPosts(),
    fetchCategories(),
    fetchTags(),
  ]);
  
  return {
    props: { posts, categories, tags },
  };
  
  // ❌ Bad: Sequential fetching
  // const posts = await fetchPosts();
  // const categories = await fetchCategories();
  // const tags = await fetchTags();
};
```

**Incremental Builds:**
```typescript
// Only rebuild changed pages
// Supported by Vercel, Netlify

// next.config.js
module.exports = {
  experimental: {
    incrementalCacheHandlerPath: './cache-handler.js',
  },
};

// Speeds up deployment significantly
```

**Build-Time Optimization:**
```typescript
// Limit pre-rendered pages in development
export const getStaticPaths = async () => {
  const isDev = process.env.NODE_ENV === 'development';
  
  if (isDev) {
    // Only build a few pages in dev
    return {
      paths: [{ params: { id: '1' } }],
      fallback: 'blocking',
    };
  }
  
  // Build all pages in production
  const products = await fetchAllProducts();
  const paths = products.map(p => ({ params: { id: p.id } }));
  
  return {
    paths,
    fallback: false,
  };
};
```

#### 10. Best Practices

**1. Use ISR for Semi-Dynamic Content:**
```typescript
export const getStaticProps = async () => {
  const data = await fetchData();
  
  return {
    props: { data },
    revalidate: 60, // Regenerate page every 60 seconds
  };
};

// Best of both worlds:
// - Static speed
// - Content stays fresh
```

**2. Handle Build Failures:**
```typescript
export const getStaticProps = async ({ params }) => {
  try {
    const data = await fetchData(params.id);
    return { props: { data } };
  } catch (error) {
    console.error('Build error:', error);
    return {
      notFound: true, // Generate 404 page
    };
  }
};
```

**3. Optimize Images:**
```typescript
import Image from 'next/image';

function ProductImage({ product }) {
  return (
    <Image
      src={product.image}
      alt={product.name}
      width={800}
      height={600}
      placeholder="blur"
      blurDataURL={product.blurDataURL}
    />
  );
}

// Next.js optimizes images at build time for SSG
```

**4. Preload Critical Data:**
```typescript
export const getStaticProps = async () => {
  const criticalData = await fetchCriticalData();
  const optionalData = await fetchOptionalData().catch(() => null);
  
  return {
    props: {
      criticalData,
      optionalData,
    },
  };
};
```

**5. Monitor Build Times:**
```bash
# Check build duration
npm run build

# Output shows:
# Page                              Size     First Load JS
# ├ λ /api/hello                    0 B             0 B
# ├ ○ /                             2.5 kB         80 kB
# ├ ○ /blog/[slug]                  1.2 kB         78 kB
# └ ● /products/[id] (ISR: 60s)     3.1 kB         82 kB

# ○ Static (SSG)
# ● ISR (SSG with revalidation)
# λ Server (SSR)
```

#### Summary

**Static Site Generation (SSG):**
- Pre-renders pages to HTML at build time
- Serves static files from CDN
- Fastest possible page loads
- Lowest hosting costs
- Best for content that doesn't change frequently

**Key Benefits:**
1. ⚡ **Blazing Fast**: Pre-built HTML from CDN
2. 💰 **Cost-Effective**: Cheap static hosting
3. 🚀 **Scalable**: Handles massive traffic
4. 🔒 **Secure**: No server attack surface
5. 🔍 **SEO Perfect**: Optimal for search engines

**When to Use:**
- Blogs and documentation
- Marketing pages
- Product catalogs (with ISR)
- Portfolios
- Content-focused sites

**Best Practices:**
- Use ISR for semi-dynamic content
- Implement fallback strategies
- Optimize build performance
- Handle errors gracefully
- Monitor build times

**Modern Approach:**
Combine SSG with ISR (Incremental Static Regeneration) for the best of both worlds: static performance with content freshness.

</details>

---

### 143. What is the difference between SSR and SSG?

<details>
<summary>View Answer</summary>

**SSR (Server-Side Rendering)** generates HTML on the server for **each request**, while **SSG (Static Site Generation)** generates HTML at **build time** once. Understanding when to use each is crucial for optimal performance and user experience.

#### 1. Core Differences

**Timing:**
```typescript
// SSG: Build Time (Once)
build command → generate all HTML → deploy static files

// User Request:
User → CDN → Pre-built HTML (instant!)

// SSR: Request Time (Every request)
User Request → Server → Generate HTML → Send HTML

// SSG is faster but less dynamic
// SSR is slower but always fresh
```

**Visual Comparison:**
```typescript
/*
SSG (Static Site Generation):
┌─────────┐
│  Build  │ ← Happens once during deployment
└────┬────┘
     │ Generates HTML for all pages
     ↓
┌─────────┐
│  CDN    │ ← Serves pre-built HTML
└────┬────┘
     │ Lightning fast delivery
     ↓
  User sees page instantly

SSR (Server-Side Rendering):
  User Request
     ↓
┌─────────┐
│ Server  │ ← Generates HTML for this request
└────┬────┘
     │ Fetches data, renders React
     ↓
┌─────────┐
│  HTML   │ ← Fresh HTML sent to user
└─────────┘
  User sees page (slight delay)
*/
```

#### 2. Detailed Comparison Table

| Aspect | SSG | SSR |
|--------|-----|-----|
| **When HTML is generated** | Build time (once) | Request time (every request) |
| **Performance** | ⚡ Fastest (CDN) | 🐢 Slower (server processing) |
| **Data freshness** | ❌ Stale until rebuild | ✅ Always fresh |
| **Server load** | ✅ None (static files) | ❌ High (compute per request) |
| **Scalability** | ✅ Excellent (CDN) | ⚠️ Limited (server capacity) |
| **Cost** | ✅ Very low ($0-20/mo) | ❌ Higher ($50-500+/mo) |
| **Build time** | ⚠️ Can be slow (many pages) | ✅ N/A |
| **Content updates** | ❌ Requires rebuild | ✅ Immediate |
| **Personalization** | ❌ Same for all users | ✅ Per-user content |
| **SEO** | ✅ Perfect | ✅ Good |
| **Use case** | Blogs, marketing, docs | Dashboards, feeds, dynamic |

#### 3. Code Examples

**SSG Example (Next.js):**
```typescript
// pages/blog/[slug].tsx
import { GetStaticProps, GetStaticPaths } from 'next';

interface Post {
  slug: string;
  title: string;
  content: string;
}

interface BlogPostProps {
  post: Post;
}

// 🏗️ Generated at BUILD TIME
export const getStaticProps: GetStaticProps<BlogPostProps> = async ({ params }) => {
  // This runs ONCE during build
  console.log('Building page for:', params?.slug);
  
  const post = await fetch(`https://api.example.com/posts/${params?.slug}`)
    .then(res => res.json());
  
  return {
    props: { post },
    // Optional: Revalidate every hour (ISR)
    // revalidate: 3600,
  };
};

// Tell Next.js which pages to generate
export const getStaticPaths: GetStaticPaths = async () => {
  const posts = await fetch('https://api.example.com/posts')
    .then(res => res.json());
  
  return {
    paths: posts.map((post: Post) => ({
      params: { slug: post.slug },
    })),
    fallback: false,
  };
};

export default function BlogPost({ post }: BlogPostProps) {
  return (
    <article>
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  );
}

// Result: Static HTML files generated at build time
// /blog/hello-world.html
// /blog/react-guide.html
// /blog/nextjs-tips.html
```

**SSR Example (Next.js):**
```typescript
// pages/dashboard/[userId].tsx
import { GetServerSideProps } from 'next';

interface User {
  id: string;
  name: string;
  lastActive: string;
}

interface DashboardProps {
  user: User;
  stats: any;
}

// 🔄 Generated on EVERY REQUEST
export const getServerSideProps: GetServerSideProps<DashboardProps> = async (context) => {
  // This runs on EVERY page request
  console.log('Rendering dashboard for user:', context.params?.userId);
  
  const { userId } = context.params!;
  
  // Fetch fresh data
  const [user, stats] = await Promise.all([
    fetch(`https://api.example.com/users/${userId}`).then(r => r.json()),
    fetch(`https://api.example.com/users/${userId}/stats`).then(r => r.json()),
  ]);
  
  return {
    props: {
      user,
      stats,
    },
  };
};

export default function Dashboard({ user, stats }: DashboardProps) {
  return (
    <div>
      <h1>Welcome, {user.name}</h1>
      <p>Last active: {user.lastActive}</p>
      <div>{/* Stats display */}</div>
    </div>
  );
}

// Result: HTML generated fresh on each request
// Always shows latest data
```

#### 4. Performance Comparison

**Load Time Comparison:**
```typescript
// SSG Performance
Time to First Byte (TTFB):        10-50ms    ⚡⚡⚡
First Contentful Paint (FCP):     100-300ms  ⚡⚡⚡
Largest Contentful Paint (LCP):   500-1000ms ⚡⚡⚡
Time to Interactive (TTI):        1-2s       ⚡⚡

// SSR Performance
Time to First Byte (TTFB):        200-800ms  ⚡⚡
First Contentful Paint (FCP):     500-1500ms ⚡⚡
Largest Contentful Paint (LCP):   1-3s       ⚡
Time to Interactive (TTI):        2-4s       ⚡

// CSR Performance (for reference)
Time to First Byte (TTFB):        50-200ms   ⚡⚡⚡
First Contentful Paint (FCP):     2-4s       ❌
Largest Contentful Paint (LCP):   3-5s       ❌
Time to Interactive (TTI):        3-5s       ❌
```

**Real-World Example:**
```typescript
// E-commerce Product Page

// SSG approach:
// Build time: 10 minutes (10,000 products)
// Request time: 50ms
// Data: May be slightly stale
// Cost: $10/month (CDN)
// Good for: Stable products

// SSR approach:
// Build time: 0
// Request time: 300ms
// Data: Always fresh
// Cost: $100/month (server)
// Good for: Real-time inventory
```

#### 5. When to Use Each

**Use SSG When:**
```typescript
// ✅ Content doesn't change often
// Examples:
// - Blog posts
// - Documentation
// - Marketing pages
// - Landing pages

export const getStaticProps = async () => {
  const posts = await fetchBlogPosts();
  return {
    props: { posts },
    revalidate: 3600, // Update hourly with ISR
  };
};

// ✅ Performance is critical
// - E-commerce product pages
// - News articles
// - Public profiles

// ✅ Content is same for all users
// - Public pages
// - Shared content
// - No personalization needed

// ✅ Large traffic expected
// - Viral content
// - High-traffic sites
// - Need to scale easily
```

**Use SSR When:**
```typescript
// ✅ Content changes frequently
// Examples:
// - User dashboards
// - Social media feeds
// - Real-time data
// - Stock prices

export const getServerSideProps = async () => {
  const liveData = await fetchRealTimeData();
  return {
    props: { liveData },
  };
};

// ✅ Personalized content
// - User-specific views
// - Authenticated pages
// - Recommendations

export const getServerSideProps = async ({ req }) => {
  const session = await getSession(req);
  const userData = await fetchUserData(session.userId);
  return {
    props: { userData },
  };
};

// ✅ Request-dependent rendering
// - A/B testing
// - Geolocation-based content
// - Device-specific rendering

export const getServerSideProps = async ({ req }) => {
  const country = getCountryFromIP(req.ip);
  const content = await fetchContentForCountry(country);
  return {
    props: { content },
  };
};

// ✅ Can't pre-render all pages
// - User-generated content
// - Infinite pages
// - Dynamic routes with no fixed paths
```

#### 6. Hybrid Approach

**Combining SSG and SSR:**
```typescript
// pages/products/[id].tsx

// Use SSG for common products, SSR for others
export const getStaticPaths = async () => {
  // Pre-render only top 100 products
  const popularProducts = await fetchPopularProducts(100);
  
  return {
    paths: popularProducts.map(p => ({ params: { id: p.id } })),
    fallback: 'blocking', // SSR for other products
  };
};

export const getStaticProps = async ({ params }) => {
  const product = await fetchProduct(params.id);
  
  return {
    props: { product },
    revalidate: 3600, // ISR: Rebuild hourly
  };
};

// Result:
// - Top 100 products: SSG (ultra-fast)
// - Other products: SSR first visit, then cached
// - All products: Stay fresh with ISR
```

**ISR (Incremental Static Regeneration):**
```typescript
// Best of both worlds!
export const getStaticProps = async () => {
  const data = await fetchData();
  
  return {
    props: { data },
    revalidate: 60, // Regenerate every 60 seconds
  };
};

// How it works:
// 1. User A visits: Gets cached static page (fast!)
// 2. After 60s, User B visits: Still gets cached page
// 3. Next.js regenerates in background
// 4. User C visits: Gets new static page

// Benefits:
// ✅ Static performance (SSG)
// ✅ Fresh content (like SSR)
// ✅ Low server load
```

#### 7. Decision Matrix

**Decision Tree:**
```typescript
/*
Start: Do I need this page?
  |
  ├─→ Does content change frequently?
  │     ├─→ YES → Is it user-specific?
  │     │           ├─→ YES → Use SSR (or CSR)
  │     │           └─→ NO → Use ISR
  │     └─→ NO → Use SSG
  │
  ├─→ Do I have millions of pages?
  │     ├─→ YES → Use SSR with caching
  │     └─→ NO → Use SSG
  │
  ├─→ Is performance critical?
  │     ├─→ YES → Use SSG
  │     └─→ NO → Use SSR or CSR
  │
  └─→ Is it public content?
        ├─→ YES → Use SSG
        └─→ NO → Use SSR
*/
```

**Quick Reference:**
```typescript
// Blog Post → SSG
// - Rarely changes
// - Same for everyone
// - SEO critical

// User Dashboard → SSR
// - User-specific
// - Real-time data
// - Private content

// Product Page → SSG + ISR
// - Static performance
// - Inventory updates
// - High traffic

// Search Results → SSR
// - Query-dependent
// - Can't pre-render
// - Dynamic content

// Landing Page → SSG
// - Rarely changes
// - Marketing content
// - Performance critical
```

#### 8. Real-World Examples

**Blog Platform:**
```typescript
// Homepage: SSG (list of posts)
export const getStaticProps = async () => {
  const posts = await fetchRecentPosts();
  return {
    props: { posts },
    revalidate: 300, // Update every 5 minutes
  };
};

// Post Page: SSG with ISR
export const getStaticProps = async ({ params }) => {
  const post = await fetchPost(params.slug);
  return {
    props: { post },
    revalidate: 3600, // Update hourly
  };
};

// Admin Dashboard: SSR
export const getServerSideProps = async ({ req }) => {
  const session = await getSession(req);
  const analytics = await fetchAnalytics(session.userId);
  return {
    props: { analytics },
  };
};
```

**E-commerce Site:**
```typescript
// Category Pages: SSG
export const getStaticProps = async ({ params }) => {
  const products = await fetchCategoryProducts(params.category);
  return {
    props: { products },
    revalidate: 3600,
  };
};

// Product Pages: SSG + ISR
export const getStaticProps = async ({ params }) => {
  const product = await fetchProduct(params.id);
  return {
    props: { product },
    revalidate: 300, // Update every 5 minutes
  };
};

// User Cart: CSR (client-side only)
// - Highly dynamic
// - User-specific
// - No SEO needed

// Checkout: SSR
export const getServerSideProps = async ({ req }) => {
  const cart = await getCart(req);
  const shipping = await calculateShipping(cart);
  return {
    props: { cart, shipping },
  };
};
```

#### 9. Migration Strategies

**From SSR to SSG:**
```typescript
// Before: SSR (slow but fresh)
export const getServerSideProps = async () => {
  const data = await fetchData();
  return { props: { data } };
};

// After: SSG with ISR (fast and fresh)
export const getStaticProps = async () => {
  const data = await fetchData();
  return {
    props: { data },
    revalidate: 60, // Revalidate every minute
  };
};

// Benefits:
// - 10x faster page loads
// - Lower server costs
// - Better scalability
// - Still fresh content
```

**From SSG to SSR:**
```typescript
// Before: SSG (fast but stale)
export const getStaticProps = async () => {
  const data = await fetchData();
  return { props: { data } };
};

// After: SSR (slower but always fresh)
export const getServerSideProps = async () => {
  const data = await fetchData();
  return { props: { data } };
};

// When needed:
// - Data changes too frequently
// - ISR isn't fast enough
// - Need guaranteed freshness
```

#### 10. Best Practices

**1. Start with SSG:**
```typescript
// Default to SSG, upgrade to SSR only when needed
// SSG is faster, cheaper, more scalable

// Most pages can use SSG + ISR:
export const getStaticProps = async () => {
  const data = await fetchData();
  return {
    props: { data },
    revalidate: 60, // Adjust based on needs
  };
};
```

**2. Use ISR for Semi-Dynamic Content:**
```typescript
// Instead of SSR for everything
// Use ISR for content that updates periodically

// Good ISR use cases:
// - Product inventory (every 5 minutes)
// - Blog comments (every 1 minute)
// - News articles (every 30 seconds)
// - User counts (every 5 minutes)
```

**3. Combine Approaches:**
```typescript
// Use SSG for public content
// Use CSR for user-specific widgets

function ProductPage({ product }) {
  // SSG: Product details (same for all users)
  const [reviews, setReviews] = useState([]);
  const [userWishlist, setUserWishlist] = useState(false);
  
  useEffect(() => {
    // CSR: User-specific data
    fetchUserWishlist().then(setUserWishlist);
    fetchReviews().then(setReviews);
  }, []);
  
  return (
    <div>
      {/* SSG content */}
      <h1>{product.name}</h1>
      <p>{product.price}</p>
      
      {/* CSR content */}
      <button>{userWishlist ? 'Remove' : 'Add to Wishlist'}</button>
      <Reviews reviews={reviews} />
    </div>
  );
}
```

**4. Monitor Performance:**
```typescript
// Track metrics for each approach
// - TTFB (Time to First Byte)
// - FCP (First Contentful Paint)
// - LCP (Largest Contentful Paint)
// - Server costs
// - Build times

// Adjust strategy based on data
```

#### Summary

**SSR vs SSG:**

| Aspect | SSG | SSR |
|--------|-----|-----|
| **Speed** | ⚡⚡⚡ Fastest | ⚡⚡ Fast |
| **Freshness** | ❌ Build-time | ✅ Real-time |
| **Cost** | ✅ Lowest | ❌ Higher |
| **Scale** | ✅ Best | ⚠️ Limited |
| **Use Case** | Static content | Dynamic content |

**Decision Guide:**
1. **Default to SSG** for best performance
2. **Add ISR** for content that updates periodically
3. **Use SSR** only when you need guaranteed freshness
4. **Combine approaches** for optimal results

**Key Principle:**
> "Use the most static approach possible for your content update frequency."
> 
> Static content → SSG  
> Hourly updates → SSG + ISR  
> Frequent updates → SSR  
> Real-time → CSR

</details>

---

### 144. What is Next.js and why is it popular?

<details>
<summary>View Answer</summary>

**Next.js** is a React framework that provides a production-ready environment with features like server-side rendering, static site generation, API routes, and optimized performance out of the box. It's the most popular React framework for building modern web applications.

#### 1. What is Next.js?

**Overview:**
```typescript
// Next.js is a React framework that provides:
// 1. Hybrid rendering (SSR, SSG, ISR, CSR)
// 2. File-based routing
// 3. API routes
// 4. Built-in optimizations
// 5. TypeScript support
// 6. Fast Refresh
// 7. Image optimization
// 8. Code splitting
// 9. Edge runtime
// 10. Production-ready features

// Created by Vercel (formerly Zeit)
// First release: October 2016
// Current version: 14+ (as of 2024)
```

**Architecture:**
```typescript
/*
Next.js Architecture:

┌─────────────────────────────────────┐
│         React Application           │
├─────────────────────────────────────┤
│  Next.js Framework Layer            │
│  • Routing                          │
│  • Data Fetching                    │
│  • Rendering (SSR/SSG/ISR)          │
│  • API Routes                       │
│  • Optimizations                    │
├─────────────────────────────────────┤
│  Node.js / Edge Runtime             │
└─────────────────────────────────────┘
*/
```

#### 2. Why Next.js is Popular

**1. Zero Configuration:**
```bash
# Create Next.js app
npx create-next-app@latest my-app
cd my-app
npm run dev

# Already configured:
# ✅ TypeScript
# ✅ ESLint
# ✅ CSS/Sass
# ✅ Fast Refresh
# ✅ Code splitting
# ✅ Optimized builds
# ✅ Production ready

# No webpack config needed!
# No babel config needed!
# Just start building!
```

**2. Flexible Rendering:**
```typescript
// Next.js supports all rendering methods:

// SSG (Static Site Generation)
export const getStaticProps = async () => {
  const data = await fetchData();
  return { props: { data } };
};

// SSR (Server-Side Rendering)
export const getServerSideProps = async () => {
  const data = await fetchData();
  return { props: { data } };
};

// ISR (Incremental Static Regeneration)
export const getStaticProps = async () => {
  const data = await fetchData();
  return {
    props: { data },
    revalidate: 60, // Regenerate every 60s
  };
};

// CSR (Client-Side Rendering)
function MyComponent() {
  const [data, setData] = useState(null);
  useEffect(() => {
    fetchData().then(setData);
  }, []);
  return <div>{data}</div>;
}

// Mix and match per page!
```

**3. File-Based Routing:**
```typescript
// No router configuration needed!
// File structure = Routes

pages/
  index.tsx              → /
  about.tsx              → /about
  blog/
    index.tsx            → /blog
    [slug].tsx           → /blog/:slug
  products/
    [id].tsx             → /products/:id
    [category]/
      [id].tsx           → /products/:category/:id
  api/
    users.ts             → /api/users
    posts/
      [id].ts            → /api/posts/:id

// Dynamic routes automatically created!
```

**4. Built-in Optimizations:**
```typescript
// Automatic Code Splitting
// Each page only loads what it needs
import dynamic from 'next/dynamic';

const HeavyComponent = dynamic(() => import('./HeavyComponent'), {
  loading: () => <p>Loading...</p>,
});

// Image Optimization
import Image from 'next/image';

function Avatar() {
  return (
    <Image
      src="/avatar.jpg"
      alt="Avatar"
      width={200}
      height={200}
      priority // Load immediately
    />
  );
}
// Automatically:
// ✅ Resizes images
// ✅ Optimizes format (WebP)
// ✅ Lazy loads
// ✅ Responsive images

// Font Optimization
import { Inter } from 'next/font/google';

const inter = Inter({ subsets: ['latin'] });
// Automatically:
// ✅ Self-hosts fonts
// ✅ Zero layout shift
// ✅ Optimal loading

// Script Optimization
import Script from 'next/script';

function MyApp() {
  return (
    <Script
      src="https://analytics.example.com/script.js"
      strategy="lazyOnload" // Load when idle
    />
  );
}
```

**5. Great Developer Experience:**
```typescript
// Fast Refresh (Hot Module Replacement)
// Edit React components without losing state

// TypeScript Support
// Works out of the box
// No configuration needed

// ESLint Integration
// Built-in rules for Next.js

// Error Overlay
// Beautiful error messages in development

// Performance Metrics
// Built-in Web Vitals tracking
```

#### 3. Core Features

**1. File-Based Routing:**
```typescript
// pages/index.tsx
export default function Home() {
  return <h1>Home Page</h1>;
}

// pages/about.tsx
export default function About() {
  return <h1>About Page</h1>;
}

// pages/blog/[slug].tsx
import { useRouter } from 'next/router';

export default function BlogPost() {
  const router = useRouter();
  const { slug } = router.query;
  
  return <h1>Post: {slug}</h1>;
}

// pages/[...slug].tsx (Catch-all route)
export default function CatchAll() {
  const router = useRouter();
  const { slug } = router.query; // slug is an array
  
  return <h1>Path: {slug?.join('/')}</h1>;
}
```

**2. API Routes:**
```typescript
// pages/api/hello.ts
import { NextApiRequest, NextApiResponse } from 'next';

export default function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  res.status(200).json({ message: 'Hello World' });
}

// pages/api/users/[id].ts
export default async function handler(req, res) {
  const { id } = req.query;
  
  if (req.method === 'GET') {
    const user = await fetchUser(id);
    res.status(200).json(user);
  } else if (req.method === 'PUT') {
    const user = await updateUser(id, req.body);
    res.status(200).json(user);
  } else {
    res.status(405).json({ error: 'Method not allowed' });
  }
}

// Access at: /api/users/123
// Full backend capabilities in your React app!
```

**3. Data Fetching:**
```typescript
// getStaticProps (SSG)
export async function getStaticProps() {
  const posts = await fetchPosts();
  
  return {
    props: { posts },
    revalidate: 3600, // ISR: Rebuild every hour
  };
}

// getServerSideProps (SSR)
export async function getServerSideProps(context) {
  const { req, res, params, query } = context;
  
  // Set cache headers
  res.setHeader(
    'Cache-Control',
    'public, s-maxage=10, stale-while-revalidate=59'
  );
  
  const data = await fetchData(params.id);
  
  return {
    props: { data },
  };
}

// getStaticPaths (for dynamic SSG routes)
export async function getStaticPaths() {
  const posts = await fetchAllPosts();
  
  const paths = posts.map(post => ({
    params: { slug: post.slug },
  }));
  
  return {
    paths,
    fallback: 'blocking', // or true, or false
  };
}
```

**4. Middleware:**
```typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  // Authentication check
  const token = request.cookies.get('token');
  
  if (!token) {
    return NextResponse.redirect(new URL('/login', request.url));
  }
  
  // Add custom header
  const response = NextResponse.next();
  response.headers.set('x-custom-header', 'value');
  
  return response;
}

// Apply to specific routes
export const config = {
  matcher: ['/dashboard/:path*', '/admin/:path*'],
};
```

**5. App Router (Next.js 13+):**
```typescript
// app/page.tsx (Server Component by default)
export default async function HomePage() {
  const posts = await fetchPosts();
  
  return (
    <div>
      <h1>Blog Posts</h1>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
        </article>
      ))}
    </div>
  );
}

// app/blog/[slug]/page.tsx
interface PageProps {
  params: { slug: string };
}

export default async function BlogPost({ params }: PageProps) {
  const post = await fetchPost(params.slug);
  
  return (
    <article>
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  );
}

// Generate static params
export async function generateStaticParams() {
  const posts = await fetchPosts();
  
  return posts.map(post => ({
    slug: post.slug,
  }));
}
```

#### 4. Performance Features

**Automatic Static Optimization:**
```typescript
// Next.js automatically determines if a page can be static

// This page is automatically static (SSG):
export default function AboutPage() {
  return <h1>About Us</h1>;
}

// This page requires SSR:
export default function DashboardPage({ user }) {
  return <h1>Welcome, {user.name}</h1>;
}

export async function getServerSideProps() {
  const user = await fetchUser();
  return { props: { user } };
}

// Next.js chooses the best approach automatically!
```

**Incremental Static Regeneration (ISR):**
```typescript
// Update static pages without rebuilding entire site
export async function getStaticProps() {
  const products = await fetchProducts();
  
  return {
    props: { products },
    revalidate: 60, // Regenerate every 60 seconds
  };
}

// How it works:
// 1. User A requests page → Serves cached version (fast!)
// 2. After 60s, User B requests → Still serves cached version
// 3. Next.js regenerates page in background
// 4. User C requests → Gets updated version

// Benefits:
// ✅ Static performance
// ✅ Fresh content
// ✅ No full rebuild needed
```

**Edge Runtime:**
```typescript
// Run code at the edge (close to users)
export const config = {
  runtime: 'edge', // or 'nodejs'
};

export default function handler(req) {
  return new Response('Hello from the edge!', {
    status: 200,
    headers: { 'content-type': 'text/plain' },
  });
}

// Benefits:
// ✅ Lower latency
// ✅ Faster response times
// ✅ Global distribution
```

#### 5. Ecosystem & Integrations

**Popular Integrations:**
```typescript
// Tailwind CSS
// Built-in support, no config needed
import 'tailwindcss/tailwind.css';

// Redux / Zustand
// Works seamlessly
import { Provider } from 'react-redux';
import { store } from './store';

function MyApp({ Component, pageProps }) {
  return (
    <Provider store={store}>
      <Component {...pageProps} />
    </Provider>
  );
}

// React Query
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient();

function MyApp({ Component, pageProps }) {
  return (
    <QueryClientProvider client={queryClient}>
      <Component {...pageProps} />
    </QueryClientProvider>;
  );
}

// NextAuth.js (Authentication)
import NextAuth from 'next-auth';
import GoogleProvider from 'next-auth/providers/google';

export default NextAuth({
  providers: [
    GoogleProvider({
      clientId: process.env.GOOGLE_ID,
      clientSecret: process.env.GOOGLE_SECRET,
    }),
  ],
});

// Prisma (Database ORM)
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

export async function getServerSideProps() {
  const users = await prisma.user.findMany();
  return { props: { users } };
}
```

#### 6. Deployment

**Vercel (Native Platform):**
```bash
# Install Vercel CLI
npm i -g vercel

# Deploy
vercel

# Features:
# ✅ Automatic deployments from Git
# ✅ Preview deployments for PRs
# ✅ Global CDN
# ✅ Serverless functions
# ✅ Edge network
# ✅ Analytics
# ✅ Zero configuration
```

**Other Platforms:**
```bash
# Netlify
npm run build
# Deploy .next folder

# AWS Amplify
# Configure build settings
# Deploy from Git

# Docker
# Dockerfile included
docker build -t my-next-app .
docker run -p 3000:3000 my-next-app

# Self-hosted
npm run build
npm run start
# Runs on Node.js server
```

#### 7. Real-World Use Cases

**E-commerce:**
```typescript
// Product pages: SSG with ISR
// Cart: Client-side state
// Checkout: SSR
// Admin: SSR with authentication

// Example:
export async function getStaticProps({ params }) {
  const product = await fetchProduct(params.id);
  const reviews = await fetchReviews(params.id);
  
  return {
    props: { product, reviews },
    revalidate: 300, // Update every 5 minutes
  };
}
```

**SaaS Application:**
```typescript
// Landing page: SSG
// Documentation: SSG
// Dashboard: SSR
// API: API routes

// Example middleware for auth:
export function middleware(request) {
  const token = request.cookies.get('token');
  
  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }
}
```

**Content Platform:**
```typescript
// Articles: SSG with ISR
// Author pages: SSG
// Comments: Client-side
// Search: API routes

// Example:
export async function getStaticProps() {
  const articles = await cms.getArticles();
  
  return {
    props: { articles },
    revalidate: 60, // Update every minute
  };
}
```

#### 8. Comparison with Alternatives

**Next.js vs Create React App:**
```typescript
// Create React App:
// ❌ Client-side only
// ❌ No SSR/SSG
// ❌ Manual routing setup
// ❌ No API routes
// ✅ Simple
// ✅ Good for SPAs

// Next.js:
// ✅ SSR/SSG/ISR
// ✅ File-based routing
// ✅ API routes
// ✅ Built-in optimizations
// ✅ Production-ready
// ⚠️ More complex
```

**Next.js vs Remix:**
```typescript
// Remix:
// ✅ Nested routing
// ✅ Progressive enhancement
// ✅ Great DX
// ⚠️ Smaller ecosystem

// Next.js:
// ✅ Larger ecosystem
// ✅ More mature
// ✅ Better documentation
// ✅ Vercel integration
// ⚠️ More opinionated
```

**Next.js vs Gatsby:**
```typescript
// Gatsby:
// ✅ SSG focused
// ✅ GraphQL layer
// ✅ Plugin ecosystem
// ❌ Slower builds
// ❌ No SSR

// Next.js:
// ✅ Hybrid (SSR + SSG)
// ✅ Faster builds
// ✅ More flexible
// ✅ Better for dynamic content
// ❌ No built-in GraphQL
```

#### 9. Best Practices

**1. Choose Right Rendering Method:**
```typescript
// Use SSG by default
export const getStaticProps = async () => {
  const data = await fetchData();
  return { props: { data } };
};

// Add ISR for semi-dynamic content
export const getStaticProps = async () => {
  const data = await fetchData();
  return {
    props: { data },
    revalidate: 60,
  };
};

// Use SSR only when necessary
export const getServerSideProps = async () => {
  const data = await fetchData();
  return { props: { data } };
};
```

**2. Optimize Images:**
```typescript
// Always use next/image
import Image from 'next/image';

function ProductImage({ product }) {
  return (
    <Image
      src={product.image}
      alt={product.name}
      width={800}
      height={600}
      placeholder="blur"
      priority={product.featured}
    />
  );
}
```

**3. Use Dynamic Imports:**
```typescript
// Code split heavy components
import dynamic from 'next/dynamic';

const Chart = dynamic(() => import('./Chart'), {
  ssr: false, // Don't render on server
  loading: () => <Skeleton />,
});
```

**4. Implement Error Handling:**
```typescript
// pages/404.tsx
export default function Custom404() {
  return <h1>404 - Page Not Found</h1>;
}

// pages/500.tsx
export default function Custom500() {
  return <h1>500 - Server Error</h1>;
}

// pages/_error.tsx
function Error({ statusCode }) {
  return (
    <p>
      {statusCode
        ? `An error ${statusCode} occurred on server`
        : 'An error occurred on client'}
    </p>
  );
}
```

**5. Monitor Performance:**
```typescript
// pages/_app.tsx
import { useReportWebVitals } from 'next/web-vitals';

function MyApp({ Component, pageProps }) {
  useReportWebVitals((metric) => {
    console.log(metric);
    // Send to analytics
  });
  
  return <Component {...pageProps} />;
}
```

#### Summary

**Next.js is Popular Because:**

1. **🚀 Powerful**: Hybrid rendering, API routes, middleware
2. **⚡ Fast**: Built-in optimizations, automatic code splitting
3. **🛠️ Great DX**: File-based routing, Fast Refresh, TypeScript
4. **📦 Production-Ready**: SEO, performance, security built-in
5. **🌍 Scalable**: Deploy globally with ease
6. **📚 Mature**: Large ecosystem, excellent documentation
7. **🔧 Flexible**: Use any rendering method per page
8. **💰 Cost-Effective**: Efficient hosting, serverless options

**Core Features:**
- File-based routing
- Hybrid rendering (SSR/SSG/ISR)
- API routes
- Image optimization
- Font optimization
- Built-in TypeScript support
- Fast Refresh
- Middleware
- Edge Runtime

**Best For:**
- E-commerce sites
- Content platforms
- SaaS applications
- Marketing websites
- Blogs
- Any production React application

**Why Choose Next.js:**
It's the most complete, production-ready React framework with the best developer experience and performance out of the box.

</details>

---

### 145. What is the App Router in Next.js 13+?

<details>
<summary>View Answer</summary>

**The App Router** is Next.js 13+'s new routing system built on React Server Components, providing improved performance, better layouts, streaming, and a more intuitive developer experience. It represents a fundamental shift in how Next.js applications are built.

#### 1. App Router vs Pages Router

**Directory Structure Comparison:**
```typescript
// Pages Router (Old - Still supported)
pages/
  index.tsx              → /
  about.tsx              → /about
  blog/
    [slug].tsx           → /blog/:slug
  _app.tsx               // Global layout
  _document.tsx          // HTML document

// App Router (New - Recommended)
app/
  page.tsx               → /
  layout.tsx             // Root layout
  about/
    page.tsx             → /about
  blog/
    [slug]/
      page.tsx           → /blog/:slug
    layout.tsx           // Blog layout
  loading.tsx            // Loading UI
  error.tsx              // Error UI
```

**Key Differences:**
```typescript
/*
Pages Router:
- Client Components by default
- getServerSideProps/getStaticProps for data
- _app.tsx for global layout
- Page-level loading states

App Router:
- Server Components by default
- async/await directly in components
- Nested layouts
- Streaming and Suspense
- Built-in loading/error states
- Improved performance
*/
```

#### 2. Server Components (Default)

**Server Component (Default):**
```typescript
// app/blog/page.tsx
// This is a Server Component by default!

interface Post {
  id: string;
  title: string;
  content: string;
}

// Async component - runs on server
export default async function BlogPage() {
  // Fetch data directly in component
  const res = await fetch('https://api.example.com/posts', {
    cache: 'no-store', // Dynamic (like SSR)
    // or cache: 'force-cache' // Static (like SSG)
    // or next: { revalidate: 60 } // ISR
  });
  const posts: Post[] = await res.json();
  
  return (
    <div>
      <h1>Blog Posts</h1>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.content}</p>
        </article>
      ))}
    </div>
  );
}

// Benefits:
// ✅ No need for getServerSideProps
// ✅ Direct database access
// ✅ Zero client JavaScript for this component
// ✅ Better security (API keys safe)
```

**Client Component (Opt-in):**
```typescript
// app/components/Counter.tsx
'use client'; // Opt-in to client rendering

import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>
        Increment
      </button>
    </div>
  );
}

// Use 'use client' when you need:
// - useState, useEffect, useContext
// - Event listeners (onClick, onChange)
// - Browser APIs (localStorage, window)
// - Custom hooks
// - Class components
```

**Composing Server and Client Components:**
```typescript
// app/dashboard/page.tsx (Server Component)
import Counter from './Counter'; // Client Component
import UserList from './UserList'; // Server Component

export default async function Dashboard() {
  // Fetch on server
  const users = await fetchUsers();
  
  return (
    <div>
      {/* Server Component - no JS sent to client */}
      <UserList users={users} />
      
      {/* Client Component - interactive */}
      <Counter />
    </div>
  );
}
```

#### 3. Layouts

**Root Layout (Required):**
```typescript
// app/layout.tsx
import './globals.css';
import { Inter } from 'next/font/google';

const inter = Inter({ subsets: ['latin'] });

export const metadata = {
  title: 'My App',
  description: 'Created with Next.js',
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className={inter.className}>
        <nav>
          <a href="/">Home</a>
          <a href="/about">About</a>
        </nav>
        {children}
        <footer>© 2024 My App</footer>
      </body>
    </html>
  );
}
```

**Nested Layouts:**
```typescript
// app/blog/layout.tsx
export default function BlogLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div>
      <aside>
        <h3>Blog Sidebar</h3>
        <ul>
          <li>Recent Posts</li>
          <li>Categories</li>
        </ul>
      </aside>
      <main>{children}</main>
    </div>
  );
}

// app/blog/page.tsx
export default function BlogHome() {
  return <h1>Blog Home</h1>;
  // Wrapped by both RootLayout and BlogLayout
}

// app/blog/[slug]/page.tsx
export default function BlogPost() {
  return <article>Post content</article>;
  // Also wrapped by both layouts
}
```

**Layout Composition:**
```typescript
/*
URL: /blog/hello-world

Rendered structure:
<RootLayout>
  <nav>...</nav>
  <BlogLayout>
    <aside>Blog Sidebar</aside>
    <main>
      <BlogPost /> <!-- Current page -->
    </main>
  </BlogLayout>
  <footer>...</footer>
</RootLayout>

Layouts:
- Don't re-render when navigating between pages
- Can fetch data
- Can be nested
- Shared across multiple pages
*/
```

#### 4. File Conventions

**Special Files:**
```typescript
// app/page.tsx - Page component
export default function Page() {
  return <h1>Page</h1>;
}

// app/layout.tsx - Layout wrapper
export default function Layout({ children }) {
  return <div>{children}</div>;
}

// app/loading.tsx - Loading UI (Suspense boundary)
export default function Loading() {
  return <div>Loading...</div>;
}

// app/error.tsx - Error UI
'use client';

export default function Error({
  error,
  reset,
}: {
  error: Error;
  reset: () => void;
}) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <p>{error.message}</p>
      <button onClick={reset}>Try again</button>
    </div>
  );
}

// app/not-found.tsx - 404 UI
export default function NotFound() {
  return <h2>404 - Page Not Found</h2>;
}

// app/template.tsx - Similar to layout but re-renders
export default function Template({ children }) {
  return <div>{children}</div>;
}
```

**Route Organization:**
```typescript
app/
  (marketing)/         // Route group (doesn't affect URL)
    about/
      page.tsx         → /about
    contact/
      page.tsx         → /contact
    layout.tsx         // Shared layout for marketing pages
  
  (shop)/              // Another route group
    products/
      page.tsx         → /products
    cart/
      page.tsx         → /cart
    layout.tsx         // Shared layout for shop pages
  
  dashboard/
    @sidebar/          // Parallel route
      page.tsx
    @content/          // Parallel route
      page.tsx
    layout.tsx
```

#### 5. Loading States and Streaming

**Loading UI:**
```typescript
// app/dashboard/loading.tsx
export default function Loading() {
  return (
    <div className="animate-pulse">
      <div className="h-8 bg-gray-200 rounded w-1/4 mb-4"></div>
      <div className="h-64 bg-gray-200 rounded"></div>
    </div>
  );
}

// Automatically wraps page in <Suspense>
// Shows while page.tsx is loading
```

**Streaming with Suspense:**
```typescript
// app/dashboard/page.tsx
import { Suspense } from 'react';
import Posts from './Posts';
import Analytics from './Analytics';

export default function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      
      {/* Stream independently */}
      <Suspense fallback={<div>Loading posts...</div>}>
        <Posts />
      </Suspense>
      
      <Suspense fallback={<div>Loading analytics...</div>}>
        <Analytics />
      </Suspense>
    </div>
  );
}

// Posts.tsx (Server Component)
async function Posts() {
  const posts = await fetchPosts(); // Can be slow
  return <div>{/* Render posts */}</div>;
}

// Analytics.tsx (Server Component)
async function Analytics() {
  const data = await fetchAnalytics(); // Can be slow
  return <div>{/* Render analytics */}</div>;
}

// Benefits:
// ✅ Page shell loads immediately
// ✅ Components stream in as ready
// ✅ Better perceived performance
```

#### 6. Data Fetching

**Fetching in Server Components:**
```typescript
// app/posts/page.tsx
async function getPosts() {
  const res = await fetch('https://api.example.com/posts', {
    // Static (SSG)
    cache: 'force-cache',
    
    // Or Dynamic (SSR)
    // cache: 'no-store',
    
    // Or ISR
    // next: { revalidate: 3600 },
  });
  
  if (!res.ok) throw new Error('Failed to fetch');
  return res.json();
}

export default async function PostsPage() {
  const posts = await getPosts();
  
  return (
    <div>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
        </article>
      ))}
    </div>
  );
}
```

**Parallel Data Fetching:**
```typescript
// app/user/[id]/page.tsx
async function getUser(id: string) {
  const res = await fetch(`https://api.example.com/users/${id}`);
  return res.json();
}

async function getUserPosts(id: string) {
  const res = await fetch(`https://api.example.com/users/${id}/posts`);
  return res.json();
}

export default async function UserPage({ params }: { params: { id: string } }) {
  // Fetch in parallel
  const [user, posts] = await Promise.all([
    getUser(params.id),
    getUserPosts(params.id),
  ]);
  
  return (
    <div>
      <h1>{user.name}</h1>
      <h2>Posts</h2>
      {posts.map(post => <div key={post.id}>{post.title}</div>)}
    </div>
  );
}
```

**Sequential Data Fetching:**
```typescript
export default async function Page() {
  // Fetch user first
  const user = await getUser();
  
  // Then fetch posts based on user
  const posts = await getUserPosts(user.id);
  
  return <div>{/* Render */}</div>;
}
```

#### 7. Dynamic Routes

**Dynamic Segments:**
```typescript
// app/blog/[slug]/page.tsx
interface PageProps {
  params: { slug: string };
  searchParams: { [key: string]: string | string[] | undefined };
}

export default async function BlogPost({ params, searchParams }: PageProps) {
  const post = await getPost(params.slug);
  
  return (
    <article>
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  );
}

// Generate static params (like getStaticPaths)
export async function generateStaticParams() {
  const posts = await getPosts();
  
  return posts.map(post => ({
    slug: post.slug,
  }));
}

// Generate metadata
export async function generateMetadata({ params }: PageProps) {
  const post = await getPost(params.slug);
  
  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      description: post.excerpt,
      images: [post.coverImage],
    },
  };
}
```

**Catch-all Routes:**
```typescript
// app/docs/[...slug]/page.tsx
interface PageProps {
  params: { slug: string[] };
}

export default function DocsPage({ params }: PageProps) {
  // /docs/a/b/c → params.slug = ['a', 'b', 'c']
  const path = params.slug.join('/');
  
  return <div>Path: {path}</div>;
}
```

#### 8. Error Handling

**Error Boundaries:**
```typescript
// app/error.tsx
'use client'; // Must be client component

import { useEffect } from 'react';

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    // Log error to error reporting service
    console.error(error);
  }, [error]);
  
  return (
    <div>
      <h2>Something went wrong!</h2>
      <p>{error.message}</p>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

**Not Found:**
```typescript
// app/blog/[slug]/page.tsx
import { notFound } from 'next/navigation';

export default async function BlogPost({ params }) {
  const post = await getPost(params.slug);
  
  if (!post) {
    notFound(); // Triggers not-found.tsx
  }
  
  return <article>{post.title}</article>;
}

// app/blog/[slug]/not-found.tsx
export default function NotFound() {
  return <h2>Blog post not found</h2>;
}
```

#### 9. Metadata

**Static Metadata:**
```typescript
// app/about/page.tsx
import { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'About Us',
  description: 'Learn more about our company',
  openGraph: {
    title: 'About Us',
    description: 'Learn more about our company',
    images: ['/og-image.jpg'],
  },
  twitter: {
    card: 'summary_large_image',
    title: 'About Us',
    description: 'Learn more about our company',
  },
};

export default function AboutPage() {
  return <h1>About Us</h1>;
}
```

**Dynamic Metadata:**
```typescript
// app/blog/[slug]/page.tsx
export async function generateMetadata({ params }): Promise<Metadata> {
  const post = await getPost(params.slug);
  
  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      description: post.excerpt,
      images: [post.coverImage],
      type: 'article',
      publishedTime: post.publishedAt,
      authors: [post.author.name],
    },
  };
}
```

#### 10. Migration from Pages Router

**Step-by-Step Migration:**
```typescript
// 1. Create app directory alongside pages
project/
  pages/          // Keep existing
  app/            // Add new
    layout.tsx

// 2. Move routes incrementally
// pages/about.tsx → app/about/page.tsx

// Before (Pages Router)
export default function About() {
  return <h1>About</h1>;
}

// After (App Router)
export default function AboutPage() {
  return <h1>About</h1>;
}

// 3. Convert data fetching
// Before
export async function getServerSideProps() {
  const data = await fetchData();
  return { props: { data } };
}

export default function Page({ data }) {
  return <div>{data}</div>;
}

// After
export default async function Page() {
  const data = await fetchData();
  return <div>{data}</div>;
}

// 4. Move _app.tsx logic to layout.tsx
// Before: pages/_app.tsx
export default function App({ Component, pageProps }) {
  return (
    <div>
      <Header />
      <Component {...pageProps} />
      <Footer />
    </div>
  );
}

// After: app/layout.tsx
export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <Header />
        {children}
        <Footer />
      </body>
    </html>
  );
}
```

#### Summary

**App Router Key Features:**

1. **Server Components by Default**
   - Zero client JavaScript for non-interactive components
   - Direct database/API access
   - Better performance

2. **Improved Layouts**
   - Nested layouts
   - Shared UI that doesn't re-render
   - Better code organization

3. **Streaming & Suspense**
   - Progressive rendering
   - Better loading states
   - Improved UX

4. **Simplified Data Fetching**
   - async/await in components
   - No need for getServerSideProps
   - More intuitive

5. **Better File Conventions**
   - loading.tsx for loading states
   - error.tsx for error boundaries
   - not-found.tsx for 404s

**When to Use:**
- ✅ New projects (recommended)
- ✅ Apps needing better performance
- ✅ Complex layouts
- ✅ Gradually migrate existing apps

**Migration:**
- Can coexist with Pages Router
- Migrate incrementally
- App Router takes precedence

</details>

---

### 146. What are Server Actions in Next.js?

<details>
<summary>View Answer</summary>

**Server Actions** are asynchronous functions that run on the server, allowing you to perform server-side data mutations directly from React components without creating API routes. They enable progressive enhancement and provide a seamless way to handle form submissions and data mutations.

#### 1. What are Server Actions?

**Concept:**
```typescript
// Server Actions allow you to:
// 1. Mutate data on the server
// 2. Call directly from components
// 3. No API routes needed
// 4. Type-safe
// 5. Progressive enhancement
// 6. Automatic revalidation

// Traditional approach (API Route):
// Component → fetch('/api/create') → API Route → Database

// Server Actions:
// Component → Server Action → Database
// Simpler, type-safe, less boilerplate!
```

**Basic Example:**
```typescript
// app/actions.ts
'use server'; // Mark as Server Action

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string;
  const content = formData.get('content') as string;
  
  // Direct database access
  await db.post.create({
    data: { title, content },
  });
  
  // Revalidate cache
  revalidatePath('/posts');
}

// app/new-post/page.tsx
import { createPost } from '../actions';

export default function NewPostPage() {
  return (
    <form action={createPost}>
      <input name="title" placeholder="Title" />
      <textarea name="content" placeholder="Content" />
      <button type="submit">Create Post</button>
    </form>
  );
}

// No API route needed!
// No useState for loading!
// Works with JavaScript disabled!
```

#### 2. Creating Server Actions

**File-Level Server Actions:**
```typescript
// app/actions.ts
'use server'; // All exports are Server Actions

import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';

export async function createUser(formData: FormData) {
  const name = formData.get('name') as string;
  const email = formData.get('email') as string;
  
  // Validate
  if (!email.includes('@')) {
    return { error: 'Invalid email' };
  }
  
  // Create user
  const user = await db.user.create({
    data: { name, email },
  });
  
  // Revalidate and redirect
  revalidatePath('/users');
  redirect(`/users/${user.id}`);
}

export async function updateUser(id: string, formData: FormData) {
  const name = formData.get('name') as string;
  
  await db.user.update({
    where: { id },
    data: { name },
  });
  
  revalidatePath(`/users/${id}`);
}

export async function deleteUser(id: string) {
  await db.user.delete({
    where: { id },
  });
  
  revalidatePath('/users');
  redirect('/users');
}
```

**Inline Server Actions:**
```typescript
// app/posts/page.tsx
export default function PostsPage() {
  async function createPost(formData: FormData) {
    'use server'; // Inline Server Action
    
    const title = formData.get('title') as string;
    await db.post.create({ data: { title } });
    revalidatePath('/posts');
  }
  
  return (
    <form action={createPost}>
      <input name="title" />
      <button>Create</button>
    </form>
  );
}
```

#### 3. Form Handling

**Basic Form:**
```typescript
// app/contact/page.tsx
import { submitContactForm } from './actions';

export default function ContactPage() {
  return (
    <form action={submitContactForm}>
      <input name="name" placeholder="Name" required />
      <input name="email" type="email" placeholder="Email" required />
      <textarea name="message" placeholder="Message" required />
      <button type="submit">Send Message</button>
    </form>
  );
}

// app/contact/actions.ts
'use server';

export async function submitContactForm(formData: FormData) {
  const data = {
    name: formData.get('name') as string,
    email: formData.get('email') as string,
    message: formData.get('message') as string,
  };
  
  // Send email
  await sendEmail(data);
  
  // Save to database
  await db.contactSubmission.create({ data });
  
  return { success: true };
}
```

**With Client Component and useFormState:**
```typescript
// app/components/ContactForm.tsx
'use client';

import { useFormState, useFormStatus } from 'react-dom';
import { submitContactForm } from '../actions';

function SubmitButton() {
  const { pending } = useFormStatus();
  
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Submitting...' : 'Submit'}
    </button>
  );
}

export default function ContactForm() {
  const [state, formAction] = useFormState(submitContactForm, null);
  
  return (
    <form action={formAction}>
      <input name="name" placeholder="Name" />
      <input name="email" type="email" placeholder="Email" />
      <textarea name="message" placeholder="Message" />
      
      {state?.error && (
        <p className="text-red-500">{state.error}</p>
      )}
      
      {state?.success && (
        <p className="text-green-500">Message sent!</p>
      )}
      
      <SubmitButton />
    </form>
  );
}
```

**With useTransition:**
```typescript
'use client';

import { useTransition } from 'react';
import { createPost } from './actions';

export default function CreatePostForm() {
  const [isPending, startTransition] = useTransition();
  
  async function handleSubmit(formData: FormData) {
    startTransition(async () => {
      await createPost(formData);
    });
  }
  
  return (
    <form action={handleSubmit}>
      <input name="title" />
      <textarea name="content" />
      <button disabled={isPending}>
        {isPending ? 'Creating...' : 'Create Post'}
      </button>
    </form>
  );
}
```

#### 4. Data Mutations

**CRUD Operations:**
```typescript
// app/actions/posts.ts
'use server';

import { revalidatePath, revalidateTag } from 'next/cache';
import { redirect } from 'next/navigation';

// CREATE
export async function createPost(formData: FormData) {
  const post = await db.post.create({
    data: {
      title: formData.get('title') as string,
      content: formData.get('content') as string,
      authorId: formData.get('authorId') as string,
    },
  });
  
  revalidateTag('posts');
  redirect(`/posts/${post.id}`);
}

// READ (usually not a Server Action, just fetch)
// But can be used for dynamic data
export async function getPostById(id: string) {
  'use server';
  return await db.post.findUnique({ where: { id } });
}

// UPDATE
export async function updatePost(id: string, formData: FormData) {
  const post = await db.post.update({
    where: { id },
    data: {
      title: formData.get('title') as string,
      content: formData.get('content') as string,
    },
  });
  
  revalidatePath(`/posts/${id}`);
  return post;
}

// DELETE
export async function deletePost(id: string) {
  await db.post.delete({ where: { id } });
  
  revalidateTag('posts');
  redirect('/posts');
}
```

**Optimistic Updates:**
```typescript
'use client';

import { useOptimistic } from 'react';
import { likePost } from './actions';

interface Post {
  id: string;
  likes: number;
}

export default function PostLikes({ post }: { post: Post }) {
  const [optimisticLikes, addOptimisticLike] = useOptimistic(
    post.likes,
    (state, amount: number) => state + amount
  );
  
  async function handleLike() {
    addOptimisticLike(1); // Update UI immediately
    await likePost(post.id); // Update server
  }
  
  return (
    <button onClick={handleLike}>
      ♥ {optimisticLikes} likes
    </button>
  );
}

// app/actions.ts
'use server';

export async function likePost(postId: string) {
  await db.post.update({
    where: { id: postId },
    data: { likes: { increment: 1 } },
  });
  
  revalidatePath('/posts');
}
```

#### 5. Validation and Error Handling

**With Zod Validation:**
```typescript
// app/actions/auth.ts
'use server';

import { z } from 'zod';

const signupSchema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
  name: z.string().min(2, 'Name must be at least 2 characters'),
});

export async function signup(formData: FormData) {
  // Validate
  const validatedFields = signupSchema.safeParse({
    email: formData.get('email'),
    password: formData.get('password'),
    name: formData.get('name'),
  });
  
  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors,
    };
  }
  
  // Create user
  try {
    const user = await db.user.create({
      data: validatedFields.data,
    });
    
    redirect('/dashboard');
  } catch (error) {
    return {
      error: 'Failed to create user',
    };
  }
}
```

**Error Handling:**
```typescript
'use server';

export async function updateProfile(userId: string, formData: FormData) {
  try {
    // Check permissions
    const session = await getSession();
    if (session.userId !== userId) {
      throw new Error('Unauthorized');
    }
    
    // Update
    const user = await db.user.update({
      where: { id: userId },
      data: {
        name: formData.get('name') as string,
        bio: formData.get('bio') as string,
      },
    });
    
    revalidatePath(`/profile/${userId}`);
    
    return { success: true, user };
  } catch (error) {
    console.error('Update profile error:', error);
    
    return {
      success: false,
      error: error instanceof Error ? error.message : 'Unknown error',
    };
  }
}
```

#### 6. Authentication and Authorization

**Protected Server Actions:**
```typescript
// app/actions/admin.ts
'use server';

import { auth } from '@/lib/auth';
import { redirect } from 'next/navigation';

export async function deleteUser(userId: string) {
  const session = await auth();
  
  // Check authentication
  if (!session?.user) {
    redirect('/login');
  }
  
  // Check authorization
  if (session.user.role !== 'admin') {
    throw new Error('Unauthorized');
  }
  
  // Perform action
  await db.user.delete({ where: { id: userId } });
  
  revalidatePath('/admin/users');
}
```

**With Cookies:**
```typescript
'use server';

import { cookies } from 'next/headers';

export async function login(formData: FormData) {
  const email = formData.get('email') as string;
  const password = formData.get('password') as string;
  
  // Verify credentials
  const user = await verifyCredentials(email, password);
  
  if (!user) {
    return { error: 'Invalid credentials' };
  }
  
  // Create session
  const session = await createSession(user.id);
  
  // Set cookie
  cookies().set('session', session.token, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'lax',
    maxAge: 60 * 60 * 24 * 7, // 1 week
  });
  
  redirect('/dashboard');
}

export async function logout() {
  'use server';
  
  cookies().delete('session');
  redirect('/');
}
```

#### 7. Revalidation

**Revalidate Path:**
```typescript
'use server';

import { revalidatePath } from 'next/cache';

export async function updatePost(id: string, formData: FormData) {
  await db.post.update({
    where: { id },
    data: { title: formData.get('title') as string },
  });
  
  // Revalidate specific path
  revalidatePath(`/posts/${id}`);
  
  // Revalidate multiple paths
  revalidatePath('/posts');
  revalidatePath(`/posts/${id}`);
}
```

**Revalidate Tag:**
```typescript
// app/posts/page.tsx
export default async function PostsPage() {
  const posts = await fetch('https://api.example.com/posts', {
    next: { tags: ['posts'] }, // Tag this data
  }).then(r => r.json());
  
  return <div>{/* Render posts */}</div>;
}

// app/actions.ts
'use server';

import { revalidateTag } from 'next/cache';

export async function createPost(formData: FormData) {
  await db.post.create({ data: { /* ... */ } });
  
  // Revalidate all data tagged with 'posts'
  revalidateTag('posts');
}
```

#### 8. Progressive Enhancement

**Works Without JavaScript:**
```typescript
// This form works even if JavaScript is disabled!
export default function ContactPage() {
  async function handleSubmit(formData: FormData) {
    'use server';
    
    await submitContact({
      name: formData.get('name') as string,
      email: formData.get('email') as string,
    });
    
    redirect('/thank-you');
  }
  
  return (
    <form action={handleSubmit}>
      <input name="name" required />
      <input name="email" type="email" required />
      <button type="submit">Submit</button>
    </form>
  );
}

// Benefits:
// ✅ Accessible
// ✅ Works on slow connections
// ✅ Works with JavaScript disabled
// ✅ Progressive enhancement
```

#### 9. Best Practices

**1. Use Server Actions for Mutations:**
```typescript
// ✅ Good: Use Server Actions for mutations
export async function createPost(formData: FormData) {
  'use server';
  await db.post.create({ data: { /* ... */ } });
}

// ❌ Avoid: Using Server Actions for reads
// (use direct fetch or Server Components instead)
```

**2. Validate Input:**
```typescript
'use server';

import { z } from 'zod';

const schema = z.object({
  title: z.string().min(1).max(100),
  content: z.string().min(1),
});

export async function createPost(formData: FormData) {
  const validated = schema.parse({
    title: formData.get('title'),
    content: formData.get('content'),
  });
  
  await db.post.create({ data: validated });
}
```

**3. Handle Errors Gracefully:**
```typescript
export async function updateUser(id: string, formData: FormData) {
  try {
    await db.user.update({
      where: { id },
      data: { /* ... */ },
    });
    
    return { success: true };
  } catch (error) {
    return {
      success: false,
      error: 'Failed to update user',
    };
  }
}
```

**4. Revalidate Appropriately:**
```typescript
'use server';

export async function updatePost(id: string, data: any) {
  await db.post.update({ where: { id }, data });
  
  // Revalidate specific pages affected
  revalidatePath(`/posts/${id}`);
  revalidatePath('/posts');
  revalidateTag('posts');
}
```

**5. Keep Actions Small and Focused:**
```typescript
// ✅ Good: Single responsibility
export async function createPost(data: PostData) {
  'use server';
  return await db.post.create({ data });
}

export async function updatePost(id: string, data: PostData) {
  'use server';
  return await db.post.update({ where: { id }, data });
}

// ❌ Avoid: One action doing too much
export async function managePost(action: string, data: any) {
  'use server';
  if (action === 'create') { /* ... */ }
  if (action === 'update') { /* ... */ }
  if (action === 'delete') { /* ... */ }
}
```

#### Summary

**Server Actions provide:**

1. **Simplified Data Mutations**
   - No API routes needed
   - Direct database access
   - Type-safe by default

2. **Progressive Enhancement**
   - Works without JavaScript
   - Better accessibility
   - Improved user experience

3. **Automatic Revalidation**
   - revalidatePath()
   - revalidateTag()
   - Keep data fresh

4. **Built-in Optimizations**
   - Automatic code splitting
   - Streaming responses
   - Optimistic updates

**Key Features:**
- `'use server'` directive
- Form actions
- useFormState hook
- useFormStatus hook
- useOptimistic hook
- revalidatePath/revalidateTag

**When to Use:**
- ✅ Form submissions
- ✅ Data mutations (Create, Update, Delete)
- ✅ Authentication flows
- ✅ Any server-side operation

**Benefits:**
- Simpler code
- Better DX
- Type safety
- Progressive enhancement
- No API boilerplate

</details>

---

### 147. What is Incremental Static Regeneration (ISR)?

<details>
<summary>View Answer</summary>

**Incremental Static Regeneration (ISR)** allows you to update static pages after build time without rebuilding the entire site. It combines the benefits of static generation (speed) with the freshness of server-side rendering.

#### 1. What is ISR?

**Concept:**
```typescript
/*
ISR Flow:

1. Build Time:
   - Generate static HTML for pages
   - Deploy to CDN

2. User Request (within revalidate period):
   User → CDN → Cached static HTML (fast!)

3. User Request (after revalidate period):
   User A → CDN → Stale HTML (still fast!)
   ↓
   Next.js regenerates page in background
   ↓
   User B → CDN → Fresh HTML

Benefits:
- ✅ Static performance
- ✅ Fresh content
- ✅ No full rebuild needed
- ✅ Scales to millions of pages
*/
```

**Stale-While-Revalidate Strategy:**
```typescript
/*
Timeline:

0s:    Page generated at build
       |
60s:   User A visits → Gets cached page (fast!)
       |
120s:  User B visits → Gets cached page (fast!)
       |
180s:  User C visits → Gets cached page
       → Regeneration triggered in background
       |
185s:  New version ready
       |
200s:  User D visits → Gets NEW cached page

Key: Users always get fast response!
*/
```

#### 2. Basic ISR Implementation

**Pages Router:**
```typescript
// pages/posts/[id].tsx
import { GetStaticProps, GetStaticPaths } from 'next';

interface Post {
  id: string;
  title: string;
  content: string;
  updatedAt: string;
}

interface PostPageProps {
  post: Post;
}

export default function PostPage({ post }: PostPageProps) {
  return (
    <article>
      <h1>{post.title}</h1>
      <time>Updated: {post.updatedAt}</time>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  );
}

// Generate static pages at build time
export const getStaticPaths: GetStaticPaths = async () => {
  // Pre-generate popular posts
  const popularPosts = await fetchPopularPosts();
  
  return {
    paths: popularPosts.map(post => ({
      params: { id: post.id },
    })),
    fallback: 'blocking', // Generate other pages on-demand
  };
};

// Fetch data with revalidation
export const getStaticProps: GetStaticProps<PostPageProps> = async ({ params }) => {
  const post = await fetchPost(params!.id as string);
  
  if (!post) {
    return { notFound: true };
  }
  
  return {
    props: { post },
    revalidate: 60, // Regenerate every 60 seconds
  };
};
```

**App Router:**
```typescript
// app/posts/[id]/page.tsx

interface Post {
  id: string;
  title: string;
  content: string;
}

interface PageProps {
  params: { id: string };
}

async function getPost(id: string) {
  const res = await fetch(`https://api.example.com/posts/${id}`, {
    next: { revalidate: 60 }, // ISR: Revalidate every 60 seconds
  });
  
  if (!res.ok) throw new Error('Failed to fetch');
  return res.json();
}

export default async function PostPage({ params }: PageProps) {
  const post = await getPost(params.id);
  
  return (
    <article>
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  );
}

// Generate static params
export async function generateStaticParams() {
  const posts = await fetchPosts();
  
  return posts.map(post => ({
    id: post.id,
  }));
}
```

#### 3. Revalidation Strategies

**Time-Based Revalidation:**
```typescript
// Revalidate every N seconds
export const getStaticProps = async () => {
  const data = await fetchData();
  
  return {
    props: { data },
    revalidate: 60, // 60 seconds
  };
};

// Different revalidation periods:
// revalidate: 1        // 1 second (near real-time)
// revalidate: 60       // 1 minute (frequent updates)
// revalidate: 3600     // 1 hour (moderate updates)
// revalidate: 86400    // 24 hours (daily updates)
// revalidate: false    // Never revalidate (pure static)
```

**On-Demand Revalidation:**
```typescript
// pages/api/revalidate.ts
import { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  // Check secret to prevent unauthorized revalidation
  if (req.query.secret !== process.env.REVALIDATE_SECRET) {
    return res.status(401).json({ message: 'Invalid token' });
  }
  
  try {
    // Revalidate specific path
    await res.revalidate('/posts/123');
    
    return res.json({ revalidated: true });
  } catch (err) {
    return res.status(500).send('Error revalidating');
  }
}

// Trigger from CMS webhook:
// POST /api/revalidate?secret=TOKEN&path=/posts/123
```

**App Router On-Demand Revalidation:**
```typescript
// app/actions.ts
'use server';

import { revalidatePath, revalidateTag } from 'next/cache';

export async function updatePost(id: string, data: any) {
  // Update post in database
  await db.post.update({ where: { id }, data });
  
  // Revalidate specific path
  revalidatePath(`/posts/${id}`);
  
  // Or revalidate by tag
  revalidateTag('posts');
}

// Usage in component:
import { updatePost } from './actions';

export default function EditForm({ post }) {
  async function handleSubmit(formData: FormData) {
    'use server';
    await updatePost(post.id, Object.fromEntries(formData));
  }
  
  return <form action={handleSubmit}>{/* Form fields */}</form>;
}
```

**Tag-Based Revalidation:**
```typescript
// app/posts/[id]/page.tsx
async function getPost(id: string) {
  const res = await fetch(`https://api.example.com/posts/${id}`, {
    next: {
      revalidate: 3600,
      tags: ['posts', `post-${id}`], // Add cache tags
    },
  });
  return res.json();
}

// Revalidate all posts
revalidateTag('posts');

// Revalidate specific post
revalidateTag('post-123');
```

#### 4. Fallback Options

**fallback: false**
```typescript
export const getStaticPaths = async () => {
  const paths = [{ params: { id: '1' } }, { params: { id: '2' } }];
  
  return {
    paths,
    fallback: false, // Return 404 for other routes
  };
};

// Use when:
// - Small, fixed number of pages
// - All pages known at build time
// - No new pages will be added
```

**fallback: true**
```typescript
export default function Page({ post }) {
  const router = useRouter();
  
  // Show loading state while generating
  if (router.isFallback) {
    return <div>Loading...</div>;
  }
  
  return <div>{post.title}</div>;
}

export const getStaticPaths = async () => {
  // Pre-generate only popular pages
  const popularPosts = await fetchPopularPosts(10);
  
  return {
    paths: popularPosts.map(p => ({ params: { id: p.id } })),
    fallback: true, // Generate others on first request
  };
};

// Use when:
// - Many pages (thousands+)
// - Want to show loading state
// - OK with brief loading indicator
```

**fallback: 'blocking'**
```typescript
export const getStaticPaths = async () => {
  const paths = await getPopularPaths();
  
  return {
    paths,
    fallback: 'blocking', // SSR on first request, then cache
  };
};

// Use when:
// - Many pages
// - Don't want loading state
// - Better SEO (no loading spinner)
// - First visitor waits, others get cached version
```

#### 5. Real-World Examples

**E-commerce Product Pages:**
```typescript
// pages/products/[id].tsx
export const getStaticProps = async ({ params }) => {
  const [product, reviews, inventory] = await Promise.all([
    fetchProduct(params.id),
    fetchReviews(params.id),
    fetchInventory(params.id),
  ]);
  
  return {
    props: { product, reviews, inventory },
    revalidate: 300, // Revalidate every 5 minutes
  };
};

export const getStaticPaths = async () => {
  // Pre-generate top 1000 products
  const popularProducts = await fetchPopularProducts(1000);
  
  return {
    paths: popularProducts.map(p => ({ params: { id: p.id } })),
    fallback: 'blocking', // Generate others on-demand
  };
};

// Result:
// - Top products: Pre-built, ultra-fast
// - Other products: Generated on first visit
// - All products: Stay fresh with 5-minute revalidation
```

**Blog with CMS:**
```typescript
// pages/blog/[slug].tsx
export const getStaticProps = async ({ params }) => {
  const post = await cms.getPost(params.slug);
  
  if (!post) {
    return { notFound: true };
  }
  
  return {
    props: { post },
    revalidate: 60, // Revalidate every minute
  };
};

// pages/api/revalidate-post.ts
export default async function handler(req, res) {
  // Called by CMS webhook when post is updated
  if (req.query.secret !== process.env.CMS_WEBHOOK_SECRET) {
    return res.status(401).json({ message: 'Invalid secret' });
  }
  
  try {
    await res.revalidate(`/blog/${req.body.slug}`);
    return res.json({ revalidated: true });
  } catch (err) {
    return res.status(500).send('Error revalidating');
  }
}

// CMS Webhook:
// POST /api/revalidate-post?secret=TOKEN
// Body: { slug: 'my-post' }
```

**News Site:**
```typescript
// pages/news/[id].tsx
export const getStaticProps = async ({ params }) => {
  const article = await fetchArticle(params.id);
  
  return {
    props: { article },
    revalidate: 30, // Revalidate every 30 seconds
  };
};

export const getStaticPaths = async () => {
  // Pre-generate latest 100 articles
  const latestArticles = await fetchLatestArticles(100);
  
  return {
    paths: latestArticles.map(a => ({ params: { id: a.id } })),
    fallback: 'blocking',
  };
};

// Fresh content every 30 seconds
// Static performance for all users
```

#### 6. Monitoring and Debugging

**Check Revalidation Status:**
```typescript
// Add header to check cache status
export const getStaticProps = async ({ res }) => {
  const data = await fetchData();
  
  // Set cache headers
  res.setHeader(
    'Cache-Control',
    'public, s-maxage=60, stale-while-revalidate=120'
  );
  
  return {
    props: {
      data,
      generatedAt: new Date().toISOString(),
    },
    revalidate: 60,
  };
};

// Display generation time
export default function Page({ data, generatedAt }) {
  return (
    <div>
      <p>Generated at: {generatedAt}</p>
      {/* Content */}
    </div>
  );
}
```

**Log Regenerations:**
```typescript
export const getStaticProps = async ({ params }) => {
  console.log(`Regenerating page: ${params.id} at ${new Date().toISOString()}`);
  
  const data = await fetchData(params.id);
  
  // Send to analytics
  analytics.track('page_regenerated', {
    page: params.id,
    timestamp: Date.now(),
  });
  
  return {
    props: { data },
    revalidate: 60,
  };
};
```

#### 7. Best Practices

**1. Choose Appropriate Revalidation Periods:**
```typescript
// Content Type → Revalidation Period

// Real-time data (stock prices)
// Use SSR instead of ISR

// Frequently updated (news)
revalidate: 30 // 30 seconds

// Moderate updates (blog posts)
revalidate: 300 // 5 minutes

// Occasional updates (product pages)
revalidate: 3600 // 1 hour

// Rare updates (documentation)
revalidate: 86400 // 24 hours

// Static content (about page)
revalidate: false // Never revalidate
```

**2. Combine with On-Demand Revalidation:**
```typescript
// Use time-based as fallback
export const getStaticProps = async () => {
  const data = await fetchData();
  
  return {
    props: { data },
    revalidate: 3600, // Fallback: revalidate every hour
  };
};

// But also revalidate immediately when content changes
// via webhook or admin action
export async function revalidateOnUpdate() {
  await res.revalidate('/path');
}
```

**3. Handle Errors Gracefully:**
```typescript
export const getStaticProps = async ({ params }) => {
  try {
    const data = await fetchData(params.id);
    
    return {
      props: { data },
      revalidate: 60,
    };
  } catch (error) {
    console.error('Error fetching data:', error);
    
    // Return existing cached page if regeneration fails
    // Or return notFound
    return { notFound: true };
  }
};
```

**4. Use Fallback Wisely:**
```typescript
// For millions of pages
export const getStaticPaths = async () => {
  // Pre-generate top 100
  const topPages = await fetchTop(100);
  
  return {
    paths: topPages.map(p => ({ params: { id: p.id } })),
    fallback: 'blocking', // Generate others on-demand
  };
};

// Avoids long build times
// Still provides static performance for most visitors
```

**5. Monitor Build and Regeneration:**
```typescript
// Track metrics:
// - Build time
// - Number of pages generated
// - Regeneration frequency
// - Cache hit rate
// - User-perceived performance

// Set up alerts for:
// - Failed regenerations
// - Slow data fetching
// - High regeneration rate (might need longer revalidate)
```

#### 8. Limitations and Considerations

**Build Time:**
```typescript
// ISR doesn't eliminate build time for initial pages

// 10,000 pages × 100ms = 16 minutes build time
// Solution: Use fallback to pre-generate fewer pages

export const getStaticPaths = async () => {
  // Only pre-generate 1000 popular pages
  const popular = await fetchPopular(1000);
  
  return {
    paths: popular.map(p => ({ params: { id: p.id } })),
    fallback: 'blocking', // Others generated on-demand
  };
};

// Result: 1000 pages × 100ms = 100 seconds build time
```

**Stale Content Window:**
```typescript
// With revalidate: 60
// Content can be up to 60 seconds stale

// If unacceptable, consider:
// 1. Shorter revalidate period
// 2. On-demand revalidation
// 3. SSR for critical pages
```

**Cache Warming:**
```typescript
// After deployment, first visitors might see slower load
// while pages regenerate

// Solution: Warm cache after deployment
// Script to visit all popular pages
const popularPages = await fetchPopular();
for (const page of popularPages) {
  await fetch(`https://yoursite.com/posts/${page.id}`);
}
```

#### Summary

**Incremental Static Regeneration (ISR):**
- Updates static pages without full rebuild
- Combines static speed with content freshness
- Stale-while-revalidate strategy
- Scales to millions of pages

**Key Features:**
1. **Time-Based Revalidation**: `revalidate: 60`
2. **On-Demand Revalidation**: `res.revalidate('/path')`
3. **Tag-Based Revalidation**: `revalidateTag('posts')`
4. **Fallback Strategies**: `fallback: true | false | 'blocking'`

**When to Use:**
- ✅ Content updates periodically
- ✅ Need static performance
- ✅ Can tolerate brief staleness
- ✅ Millions of pages
- ✅ E-commerce, blogs, news sites

**Best Practices:**
- Choose appropriate revalidation periods
- Combine time-based with on-demand
- Use fallback for large sites
- Handle errors gracefully
- Monitor regeneration metrics

**Key Benefit:**
> "Static performance with dynamic content freshness - best of both worlds!"

</details>

---

### 148. How does Next.js handle data fetching?

<details>
<summary>View Answer</summary>

**Next.js provides multiple data fetching methods** optimized for different use cases - from static generation to server-side rendering to client-side fetching. The approach varies between the Pages Router and the new App Router.

#### 1. Pages Router Data Fetching

**getStaticProps (SSG):**
```typescript
// pages/posts/index.tsx
import { GetStaticProps } from 'next';

interface Post {
  id: string;
  title: string;
  excerpt: string;
}

interface PostsPageProps {
  posts: Post[];
}

// Runs at BUILD TIME
export const getStaticProps: GetStaticProps<PostsPageProps> = async () => {
  // Fetch data from API
  const res = await fetch('https://api.example.com/posts');
  const posts = await res.json();
  
  // Or fetch from database
  // const posts = await db.post.findMany();
  
  // Or read from filesystem
  // const posts = await readPostsFromDisk();
  
  return {
    props: {
      posts,
    },
    revalidate: 3600, // ISR: Revalidate every hour
  };
};

export default function PostsPage({ posts }: PostsPageProps) {
  return (
    <div>
      <h1>Blog Posts</h1>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.excerpt}</p>
        </article>
      ))}
    </div>
  );
}

// Use getStaticProps when:
// ✅ Data is same for all users
// ✅ Data is available at build time
// ✅ Page can be pre-rendered
// ✅ Performance is critical
```

**getServerSideProps (SSR):**
```typescript
// pages/dashboard/[userId].tsx
import { GetServerSideProps } from 'next';

interface User {
  id: string;
  name: string;
  email: string;
}

interface Stats {
  views: number;
  likes: number;
}

interface DashboardProps {
  user: User;
  stats: Stats;
}

// Runs on EVERY REQUEST
export const getServerSideProps: GetServerSideProps<DashboardProps> = async (context) => {
  const { params, req, res, query } = context;
  
  // Access cookies
  const token = req.cookies.token;
  
  // Set cache headers
  res.setHeader(
    'Cache-Control',
    'public, s-maxage=10, stale-while-revalidate=59'
  );
  
  // Fetch data
  const [user, stats] = await Promise.all([
    fetchUser(params!.userId as string, token),
    fetchUserStats(params!.userId as string),
  ]);
  
  // Handle not found
  if (!user) {
    return {
      notFound: true,
    };
  }
  
  // Handle redirect
  if (user.blocked) {
    return {
      redirect: {
        destination: '/blocked',
        permanent: false,
      },
    };
  }
  
  return {
    props: {
      user,
      stats,
    },
  };
};

export default function Dashboard({ user, stats }: DashboardProps) {
  return (
    <div>
      <h1>Welcome, {user.name}</h1>
      <p>Email: {user.email}</p>
      <div>
        <p>Views: {stats.views}</p>
        <p>Likes: {stats.likes}</p>
      </div>
    </div>
  );
}

// Use getServerSideProps when:
// ✅ Data changes frequently
// ✅ Data is user-specific
// ✅ Need request data (cookies, headers)
// ✅ Can't pre-render
```

**getStaticPaths (Dynamic SSG):**
```typescript
// pages/posts/[slug].tsx
import { GetStaticProps, GetStaticPaths } from 'next';

interface Post {
  slug: string;
  title: string;
  content: string;
}

interface PostPageProps {
  post: Post;
}

// Define which paths to pre-render
export const getStaticPaths: GetStaticPaths = async () => {
  // Fetch all posts
  const posts = await fetchAllPosts();
  
  // Generate paths
  const paths = posts.map(post => ({
    params: { slug: post.slug },
  }));
  
  return {
    paths,
    fallback: 'blocking', // or true, or false
  };
};

// Fetch data for each post
export const getStaticProps: GetStaticProps<PostPageProps> = async ({ params }) => {
  const post = await fetchPost(params!.slug as string);
  
  if (!post) {
    return { notFound: true };
  }
  
  return {
    props: { post },
    revalidate: 60,
  };
};

export default function PostPage({ post }: PostPageProps) {
  return (
    <article>
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  );
}
```

#### 2. App Router Data Fetching

**Server Components (Default):**
```typescript
// app/posts/page.tsx
// This is a Server Component by default

interface Post {
  id: string;
  title: string;
  content: string;
}

// Async component - fetch directly
export default async function PostsPage() {
  // Static (SSG)
  const posts = await fetch('https://api.example.com/posts', {
    cache: 'force-cache', // Default
  }).then(res => res.json());
  
  return (
    <div>
      <h1>Posts</h1>
      {posts.map((post: Post) => (
        <article key={post.id}>
          <h2>{post.title}</h2>
        </article>
      ))}
    </div>
  );
}

// Benefits:
// ✅ Direct async/await in component
// ✅ No getServerSideProps needed
// ✅ Cleaner code
// ✅ Better TypeScript support
```

**Caching Options:**
```typescript
// 1. Static Data (SSG)
const data = await fetch('https://api.example.com/data', {
  cache: 'force-cache', // Default - static generation
});

// 2. Dynamic Data (SSR)
const data = await fetch('https://api.example.com/data', {
  cache: 'no-store', // Always fetch fresh
});

// 3. Revalidated Data (ISR)
const data = await fetch('https://api.example.com/data', {
  next: { revalidate: 3600 }, // Revalidate every hour
});

// 4. Tagged Cache
const data = await fetch('https://api.example.com/data', {
  next: {
    revalidate: 3600,
    tags: ['posts'], // Cache tag for invalidation
  },
});
```

**Parallel Data Fetching:**
```typescript
// app/user/[id]/page.tsx

async function getUser(id: string) {
  const res = await fetch(`https://api.example.com/users/${id}`);
  return res.json();
}

async function getUserPosts(id: string) {
  const res = await fetch(`https://api.example.com/users/${id}/posts`);
  return res.json();
}

async function getUserStats(id: string) {
  const res = await fetch(`https://api.example.com/users/${id}/stats`);
  return res.json();
}

export default async function UserPage({ params }: { params: { id: string } }) {
  // Fetch in parallel
  const [user, posts, stats] = await Promise.all([
    getUser(params.id),
    getUserPosts(params.id),
    getUserStats(params.id),
  ]);
  
  return (
    <div>
      <h1>{user.name}</h1>
      <div>
        <h2>Posts ({posts.length})</h2>
        {posts.map(post => <div key={post.id}>{post.title}</div>)}
      </div>
      <div>
        <h2>Stats</h2>
        <p>Views: {stats.views}</p>
      </div>
    </div>
  );
}
```

**Sequential Data Fetching:**
```typescript
export default async function Page() {
  // Fetch user first
  const user = await getUser();
  
  // Then fetch their specific data
  const preferences = await getUserPreferences(user.id);
  const recommendations = await getRecommendations(preferences);
  
  return (
    <div>
      <h1>{user.name}</h1>
      <RecommendationList items={recommendations} />
    </div>
  );
}
```

**Streaming with Suspense:**
```typescript
// app/dashboard/page.tsx
import { Suspense } from 'react';

// Slow components
async function Posts() {
  const posts = await fetchPosts(); // Slow
  return <div>{/* Render posts */}</div>;
}

async function Analytics() {
  const data = await fetchAnalytics(); // Slow
  return <div>{/* Render analytics */}</div>;
}

export default function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      
      {/* Stream components independently */}
      <Suspense fallback={<div>Loading posts...</div>}>
        <Posts />
      </Suspense>
      
      <Suspense fallback={<div>Loading analytics...</div>}>
        <Analytics />
      </Suspense>
    </div>
  );
}

// Benefits:
// ✅ Page shell renders immediately
// ✅ Components stream as ready
// ✅ Better perceived performance
```

#### 3. Client-Side Data Fetching

**useEffect:**
```typescript
'use client';

import { useState, useEffect } from 'react';

interface Post {
  id: string;
  title: string;
}

export default function Posts() {
  const [posts, setPosts] = useState<Post[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  
  useEffect(() => {
    fetch('/api/posts')
      .then(res => res.json())
      .then(data => {
        setPosts(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, []);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  
  return (
    <div>
      {posts.map(post => (
        <div key={post.id}>{post.title}</div>
      ))}
    </div>
  );
}
```

**SWR (Recommended for Client-Side):**
```typescript
'use client';

import useSWR from 'swr';

const fetcher = (url: string) => fetch(url).then(r => r.json());

export default function Posts() {
  const { data, error, isLoading } = useSWR('/api/posts', fetcher, {
    refreshInterval: 3000, // Revalidate every 3 seconds
    revalidateOnFocus: true,
    revalidateOnReconnect: true,
  });
  
  if (error) return <div>Failed to load</div>;
  if (isLoading) return <div>Loading...</div>;
  
  return (
    <div>
      {data.map(post => (
        <div key={post.id}>{post.title}</div>
      ))}
    </div>
  );
}

// Benefits:
// ✅ Automatic revalidation
// ✅ Cache management
// ✅ Dedupe requests
// ✅ Focus revalidation
// ✅ Optimistic UI
```

**React Query:**
```typescript
'use client';

import { useQuery } from '@tanstack/react-query';

const fetchPosts = async () => {
  const res = await fetch('/api/posts');
  if (!res.ok) throw new Error('Failed to fetch');
  return res.json();
};

export default function Posts() {
  const { data, error, isLoading } = useQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts,
    staleTime: 5 * 60 * 1000, // 5 minutes
    cacheTime: 10 * 60 * 1000, // 10 minutes
  });
  
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return (
    <div>
      {data.map(post => (
        <div key={post.id}>{post.title}</div>
      ))}
    </div>
  );
}
```

#### 4. Route Handlers (API Routes)

**Pages Router API Routes:**
```typescript
// pages/api/posts.ts
import { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method === 'GET') {
    const posts = await db.post.findMany();
    res.status(200).json(posts);
  } else if (req.method === 'POST') {
    const post = await db.post.create({ data: req.body });
    res.status(201).json(post);
  } else {
    res.status(405).json({ error: 'Method not allowed' });
  }
}
```

**App Router Route Handlers:**
```typescript
// app/api/posts/route.ts
import { NextRequest, NextResponse } from 'next/server';

// GET /api/posts
export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const category = searchParams.get('category');
  
  const posts = await db.post.findMany({
    where: category ? { category } : undefined,
  });
  
  return NextResponse.json(posts);
}

// POST /api/posts
export async function POST(request: NextRequest) {
  const body = await request.json();
  
  const post = await db.post.create({ data: body });
  
  return NextResponse.json(post, { status: 201 });
}

// Dynamic route: app/api/posts/[id]/route.ts
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const post = await db.post.findUnique({
    where: { id: params.id },
  });
  
  if (!post) {
    return NextResponse.json(
      { error: 'Post not found' },
      { status: 404 }
    );
  }
  
  return NextResponse.json(post);
}
```

#### 5. Data Fetching Patterns

**Hybrid Approach:**
```typescript
// Server Component fetches initial data
export default async function ProductPage({ params }) {
  // SSG: Fetch product details
  const product = await fetch(`/api/products/${params.id}`, {
    cache: 'force-cache',
  }).then(r => r.json());
  
  return (
    <div>
      {/* Static content */}
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      
      {/* Client component for dynamic data */}
      <ReviewsSection productId={params.id} />
      <RecommendationsSection productId={params.id} />
    </div>
  );
}

// Client Component for user-specific data
'use client';

function ReviewsSection({ productId }) {
  const { data: reviews } = useSWR(`/api/reviews/${productId}`, fetcher);
  return <div>{/* Render reviews */}</div>;
}
```

**Optimistic Updates:**
```typescript
'use client';

import { useSWRConfig } from 'swr';

export default function AddPost() {
  const { mutate } = useSWRConfig();
  
  async function handleSubmit(data: FormData) {
    const newPost = { title: data.get('title'), content: data.get('content') };
    
    // Optimistic update
    mutate(
      '/api/posts',
      async (posts: any[]) => [...posts, { ...newPost, id: 'temp' }],
      false // Don't revalidate yet
    );
    
    // Send to server
    try {
      const response = await fetch('/api/posts', {
        method: 'POST',
        body: JSON.stringify(newPost),
      });
      
      // Revalidate with server data
      mutate('/api/posts');
    } catch (error) {
      // Revert on error
      mutate('/api/posts');
    }
  }
  
  return <form action={handleSubmit}>{/* Form fields */}</form>;
}
```

#### 6. Caching Strategies

**Request Memoization:**
```typescript
// Same request in multiple components = single fetch

// Component A
const user = await fetch('/api/user');

// Component B (same request)
const user = await fetch('/api/user'); // Deduplicated!

// Only 1 request sent during render
```

**Data Cache:**
```typescript
// Persistent cache across requests
const data = await fetch('/api/data', {
  next: { revalidate: 3600 }, // Cache for 1 hour
});

// Subsequent requests use cache until revalidation
```

**Full Route Cache:**
```typescript
// Entire route HTML cached at build time
// Automatically happens with static rendering
```

**Router Cache:**
```typescript
// Client-side cache of visited routes
// Automatic in App Router
```

#### 7. Best Practices

**1. Choose Right Method:**
```typescript
// Static data → Server Component with cache: 'force-cache'
// Frequent updates → cache: 'no-store'
// Periodic updates → next: { revalidate: 60 }
// User-specific → Client-side with SWR/React Query
```

**2. Fetch Data Where Needed:**
```typescript
// ✅ Good: Fetch in component that needs it
export default async function Page() {
  const posts = await getPosts();
  return <PostList posts={posts} />;
}

// ❌ Bad: Prop drilling
export default async function Page() {
  const posts = await getPosts();
  return <Layout posts={posts}>
    <Sidebar posts={posts}>
      <PostList posts={posts} />
    </Sidebar>
  </Layout>;
}
```

**3. Handle Errors:**
```typescript
export default async function Page() {
  try {
    const data = await fetchData();
    return <div>{data}</div>;
  } catch (error) {
    return <ErrorComponent error={error} />;
  }
}

// Or use error.tsx for automatic error handling
```

**4. Use TypeScript:**
```typescript
interface Post {
  id: string;
  title: string;
  content: string;
}

async function getPosts(): Promise<Post[]> {
  const res = await fetch('/api/posts');
  return res.json();
}

export default async function Page() {
  const posts = await getPosts();
  // TypeScript knows posts is Post[]
  return <div>{posts[0].title}</div>;
}
```

**5. Optimize Performance:**
```typescript
// Fetch in parallel
const [user, posts, comments] = await Promise.all([
  getUser(),
  getPosts(),
  getComments(),
]);

// Use Suspense for streaming
<Suspense fallback={<Loading />}>
  <SlowComponent />
</Suspense>

// Cache appropriately
const data = await fetch('/api/data', {
  next: { revalidate: 3600 },
});
```

#### Summary

**Data Fetching Methods:**

**Pages Router:**
- `getStaticProps` - SSG (build time)
- `getServerSideProps` - SSR (request time)
- `getStaticPaths` - Dynamic SSG routes

**App Router:**
- Server Components - async/await directly
- `cache: 'force-cache'` - SSG
- `cache: 'no-store'` - SSR
- `next: { revalidate: N }` - ISR

**Client-Side:**
- `useEffect` + `fetch`
- SWR (recommended)
- React Query

**Best Practices:**
1. Use Server Components by default
2. Client Components for interactivity
3. Fetch data where needed
4. Cache appropriately
5. Handle errors gracefully
6. Use TypeScript for type safety

**Key Principle:**
> "Fetch on the server when possible, on the client when necessary."

</details>

---

### 149. What is Remix framework and how does it differ from Next.js?

<details>
<summary>View Answer</summary>

**Remix** is a full-stack web framework built on Web Standards and React Router, focusing on server-side rendering, progressive enhancement, and leveraging the web platform. While both Remix and Next.js are React frameworks for building web applications, they have different philosophies and approaches.

#### 1. What is Remix?

**Core Philosophy:**
```typescript
/*
Remix Principles:

1. Embrace Web Standards:
   - Use native Request/Response
   - FormData for mutations
   - Standard HTTP caching
   - No custom abstractions

2. Progressive Enhancement:
   - Works without JavaScript
   - JavaScript enhances experience
   - Forms work server-side first

3. Server-First:
   - Data loading on server
   - Mutations on server
   - Error handling on server

4. Nested Routes:
   - Parallel data loading
   - Layout composition
   - Code splitting by route
*/
```

**Basic Remix App:**
```typescript
// app/routes/_index.tsx
import { json, LoaderFunctionArgs } from '@remix-run/node';
import { useLoaderData, Form } from '@remix-run/react';

interface Post {
  id: string;
  title: string;
  content: string;
}

// Load data on server
export async function loader({ request }: LoaderFunctionArgs) {
  const posts = await db.post.findMany();
  return json({ posts });
}

// Handle mutations on server
export async function action({ request }: ActionFunctionArgs) {
  const formData = await request.formData();
  const title = formData.get('title');
  const content = formData.get('content');
  
  const post = await db.post.create({
    data: { title, content },
  });
  
  return json({ post });
}

export default function Index() {
  const { posts } = useLoaderData<typeof loader>();
  
  return (
    <div>
      <h1>Posts</h1>
      
      {/* Form works without JavaScript */}
      <Form method="post">
        <input name="title" required />
        <textarea name="content" required />
        <button type="submit">Create Post</button>
      </Form>
      
      <ul>
        {posts.map(post => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </div>
  );
}
```

#### 2. Key Differences from Next.js

**Rendering Strategy:**
```typescript
// NEXT.JS: Multiple rendering modes

// 1. Static (SSG)
export async function generateStaticParams() {
  return [{ id: '1' }, { id: '2' }];
}

// 2. Dynamic (SSR)
const data = await fetch('/api/data', { cache: 'no-store' });

// 3. ISR
const data = await fetch('/api/data', { next: { revalidate: 60 } });

// REMIX: Always server-side
// Uses HTTP caching for performance

export async function loader({ request }) {
  const data = await fetchData();
  
  // Standard HTTP caching
  return json(data, {
    headers: {
      'Cache-Control': 'public, max-age=60, s-maxage=3600',
    },
  });
}
```

**Routing:**
```typescript
// NEXT.JS: File-based routing
/*
pages/
  index.tsx          → /
  about.tsx          → /about
  posts/
    index.tsx        → /posts
    [id].tsx         → /posts/:id

OR App Router:
app/
  page.tsx           → /
  about/
    page.tsx         → /about
  posts/
    page.tsx         → /posts
    [id]/
      page.tsx       → /posts/:id
*/

// REMIX: Flat file routing with nested layouts
/*
app/routes/
  _index.tsx         → /
  about.tsx          → /about
  posts._index.tsx   → /posts
  posts.$id.tsx      → /posts/:id
  posts.new.tsx      → /posts/new
  
  // Nested layouts
  dashboard.tsx      → Layout for /dashboard/*
  dashboard._index.tsx   → /dashboard
  dashboard.settings.tsx → /dashboard/settings
*/
```

**Data Loading:**
```typescript
// NEXT.JS (Pages Router)
export async function getServerSideProps() {
  const data = await fetchData();
  return { props: { data } };
}

// NEXT.JS (App Router)
export default async function Page() {
  const data = await fetch('/api/data');
  return <div>{data}</div>;
}

// REMIX: loader function
export async function loader({ params, request }) {
  // Direct database access
  const data = await db.getData(params.id);
  
  // Or fetch from API
  const response = await fetch('/api/data');
  
  // Return Response or use json helper
  return json({ data });
}

// Component
export default function Page() {
  const { data } = useLoaderData<typeof loader>();
  return <div>{data}</div>;
}
```

**Mutations:**
```typescript
// NEXT.JS: API Routes or Server Actions

// API Route
// pages/api/posts.ts
export default async function handler(req, res) {
  if (req.method === 'POST') {
    const post = await db.post.create({ data: req.body });
    res.json(post);
  }
}

// Client Component
const handleSubmit = async (data) => {
  await fetch('/api/posts', {
    method: 'POST',
    body: JSON.stringify(data),
  });
};

// Or Server Action (App Router)
'use server';
export async function createPost(formData: FormData) {
  const post = await db.post.create({
    data: { title: formData.get('title') },
  });
}

// REMIX: action function
export async function action({ request }) {
  const formData = await request.formData();
  const title = formData.get('title');
  
  const post = await db.post.create({
    data: { title },
  });
  
  return redirect(`/posts/${post.id}`);
}

// Component (works without JS!)
export default function NewPost() {
  return (
    <Form method="post">
      <input name="title" />
      <button type="submit">Create</button>
    </Form>
  );
}
```

**Nested Routes and Layouts:**
```typescript
// NEXT.JS App Router
// app/dashboard/layout.tsx
export default function DashboardLayout({ children }) {
  return (
    <div>
      <nav>Dashboard Nav</nav>
      {children}
    </div>
  );
}

// REMIX: Nested routes with parallel data loading
// app/routes/dashboard.tsx
export async function loader() {
  const user = await getUser(); // Loads in parallel with child routes
  return json({ user });
}

export default function Dashboard() {
  const { user } = useLoaderData<typeof loader>();
  
  return (
    <div>
      <nav>Dashboard Nav - {user.name}</nav>
      {/* Child route renders here */}
      <Outlet />
    </div>
  );
}

// app/routes/dashboard.settings.tsx
export async function loader() {
  const settings = await getSettings(); // Loads in parallel with parent
  return json({ settings });
}

export default function Settings() {
  const { settings } = useLoaderData<typeof loader>();
  return <div>{/* Settings UI */}</div>;
}

// Remix loads both loaders in parallel!
```

#### 3. Feature Comparison

**API Routes:**
```typescript
// NEXT.JS: Built-in API routes
// pages/api/users/[id].ts
export default function handler(req, res) {
  if (req.method === 'GET') {
    res.json({ user });
  }
}

// REMIX: Use resource routes
// app/routes/api.users.$id.ts
export async function loader({ params }) {
  const user = await db.user.findUnique({
    where: { id: params.id },
  });
  return json({ user });
}

// Or use action for mutations
export async function action({ request, params }) {
  const formData = await request.formData();
  // Handle mutation
  return json({ success: true });
}
```

**Error Handling:**
```typescript
// NEXT.JS: error.tsx boundary
// app/error.tsx
'use client';
export default function Error({ error, reset }) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={reset}>Try again</button>
    </div>
  );
}

// REMIX: ErrorBoundary export
// app/routes/posts.$id.tsx
export function ErrorBoundary() {
  const error = useRouteError();
  
  if (isRouteErrorResponse(error)) {
    return (
      <div>
        <h1>{error.status} {error.statusText}</h1>
        <p>{error.data}</p>
      </div>
    );
  }
  
  return <div>Unexpected Error</div>;
}
```

**Image Optimization:**
```typescript
// NEXT.JS: Built-in Image component
import Image from 'next/image';

<Image
  src="/photo.jpg"
  width={500}
  height={300}
  alt="Photo"
  priority
/>

// REMIX: No built-in optimization
// Use external service or custom solution
<img
  src={cloudinaryUrl}
  alt="Photo"
  loading="lazy"
/>

// Or use a library like remix-image
```

**Middleware:**
```typescript
// NEXT.JS: middleware.ts
import { NextResponse } from 'next/server';

export function middleware(request) {
  if (!request.cookies.has('token')) {
    return NextResponse.redirect('/login');
  }
}

export const config = {
  matcher: '/dashboard/:path*',
};

// REMIX: Handle in loader
export async function loader({ request }) {
  const session = await getSession(request);
  
  if (!session) {
    throw redirect('/login');
  }
  
  return json({ data });
}
```

#### 4. Performance and Optimization

**Caching:**
```typescript
// NEXT.JS: Multiple caching layers
// Automatic in App Router
const data = await fetch('/api/data', {
  next: { revalidate: 3600 },
});

// REMIX: Standard HTTP caching
export async function loader() {
  const data = await fetchData();
  
  return json(data, {
    headers: {
      // CDN caches for 1 hour
      'Cache-Control': 'public, max-age=3600, s-maxage=86400',
    },
  });
}
```

**Code Splitting:**
```typescript
// NEXT.JS: Automatic by page/component
import dynamic from 'next/dynamic';

const DynamicComponent = dynamic(() => import('./Component'));

// REMIX: Automatic by route
// Each route is a separate bundle
// Nested routes = nested bundles
```

**Prefetching:**
```typescript
// NEXT.JS: Automatic on Link hover
import Link from 'next/link';

<Link href="/posts" prefetch={true}>
  Posts
</Link>

// REMIX: Prefetch on Link hover/focus
import { Link } from '@remix-run/react';

<Link to="/posts" prefetch="intent">
  Posts
</Link>

// prefetch="intent" = prefetch on hover/focus
// prefetch="render" = prefetch immediately
// prefetch="none" = no prefetch
```

#### 5. When to Choose Each

**Choose Next.js When:**
```typescript
/*
✅ Static site generation is primary need
✅ Want ISR for content sites
✅ Need built-in image optimization
✅ Want zero-config deployment (Vercel)
✅ Large ecosystem and community
✅ Marketing sites, blogs, e-commerce
✅ Prefer multiple rendering options
✅ Need automatic code splitting
*/

// Example: E-commerce product pages
export async function generateStaticParams() {
  const products = await fetchProducts();
  return products.map(p => ({ id: p.id }));
}

export default async function ProductPage({ params }) {
  const product = await fetch(`/api/products/${params.id}`, {
    next: { revalidate: 3600 },
  });
  
  return <ProductDetails product={product} />;
}
```

**Choose Remix When:**
```typescript
/*
✅ Building dynamic, data-driven apps
✅ Progressive enhancement is important
✅ Want simpler mental model (always SSR)
✅ Need nested routes with parallel loading
✅ Form-heavy applications
✅ Want to use Web Standards
✅ Prefer traditional server-side patterns
✅ SaaS applications, dashboards, admin panels
*/

// Example: Form-heavy application
export async function action({ request }) {
  const formData = await request.formData();
  const intent = formData.get('intent');
  
  switch (intent) {
    case 'create':
      return await createItem(formData);
    case 'update':
      return await updateItem(formData);
    case 'delete':
      return await deleteItem(formData);
  }
}

export default function ItemManager() {
  const actionData = useActionData<typeof action>();
  
  return (
    <Form method="post">
      {/* Works without JS */}
      <input name="name" required />
      <button name="intent" value="create">Create</button>
    </Form>
  );
}
```

#### 6. Real-World Example Comparison

**Blog Application:**
```typescript
// NEXT.JS: Great fit with SSG + ISR
// app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  const posts = await fetchPosts();
  return posts.map(p => ({ slug: p.slug }));
}

export default async function Post({ params }) {
  const post = await fetch(`/api/posts/${params.slug}`, {
    next: { revalidate: 3600 },
  });
  
  return <Article post={post} />;
}

// Result:
// - All posts pre-rendered at build
// - New posts available within 1 hour
// - Blazing fast performance

// REMIX: Also works, uses HTTP caching
// app/routes/blog.$slug.tsx
export async function loader({ params }) {
  const post = await db.post.findUnique({
    where: { slug: params.slug },
  });
  
  return json(post, {
    headers: {
      'Cache-Control': 'public, max-age=3600',
    },
  });
}

// Result:
// - SSR on first request
// - CDN caches for 1 hour
// - New posts available immediately
```

**SaaS Dashboard:**
```typescript
// REMIX: Better fit with nested routes
// app/routes/dashboard.tsx
export async function loader({ request }) {
  const user = await requireUser(request);
  const stats = await getStats(user.id);
  return json({ user, stats });
}

// app/routes/dashboard.projects.tsx
export async function loader({ request }) {
  const user = await requireUser(request);
  const projects = await getProjects(user.id);
  return json({ projects });
}

// app/routes/dashboard.projects.$id.tsx
export async function loader({ params, request }) {
  const project = await getProject(params.id);
  return json({ project });
}

// All loaders run in parallel!
// Each route only loads its data

// NEXT.JS: Also works but less elegant
// Need to handle auth in each page
// Or use middleware + parallel fetching
```

#### 7. Migration Considerations

**Next.js to Remix:**
```typescript
// 1. getServerSideProps → loader
// Next.js
export async function getServerSideProps() {
  const data = await fetchData();
  return { props: { data } };
}

// Remix
export async function loader() {
  const data = await fetchData();
  return json({ data });
}

// 2. API routes → resource routes
// 3. Image optimization → external service
// 4. Middleware → loader checks
```

**Remix to Next.js:**
```typescript
// 1. loader → Server Component or getServerSideProps
// 2. action → Server Actions or API routes
// 3. Form → Client Component with fetch or Server Action
// 4. Nested routes → App Router layouts
```

#### Summary

**Remix:**
- Full-stack framework with server-first approach
- Embraces Web Standards (Request, Response, FormData)
- Progressive enhancement (works without JS)
- Nested routes with parallel data loading
- Always server-renders, uses HTTP caching
- Great for: SaaS apps, dashboards, form-heavy apps

**Next.js:**
- Flexible rendering (SSG, SSR, ISR, CSR)
- Built-in optimizations (images, fonts, scripts)
- Large ecosystem and community
- Multiple deployment options
- Great for: Content sites, e-commerce, marketing

**Key Philosophical Difference:**
- **Next.js**: "Render how you want" - multiple options
- **Remix**: "Always server-render" - simpler model

**Both are excellent choices** - pick based on:
- Your application type
- Team preferences
- Performance requirements
- Deployment constraints

</details>

---

### 150. What is hydration in SSR?

<details>
<summary>View Answer</summary>

**Hydration** is the process where React "attaches" event listeners and state to server-rendered HTML, making it interactive. The server sends static HTML, then the client-side JavaScript "hydrates" it by adding interactivity without re-rendering the DOM.

#### 1. What is Hydration?

**The Process:**
```typescript
/*
SSR + Hydration Flow:

1. Server:
   React Component → renderToString() → HTML string
   
2. Browser receives:
   <div id="root">
     <button class="btn">Click me</button>  <!-- Static, no JS yet -->
   </div>
   
3. JavaScript loads:
   React downloads and parses
   
4. Hydration:
   React "adopts" existing DOM
   Attaches event listeners
   Initializes state
   
5. Result:
   <button class="btn" onClick={handler}>Click me</button>  <!-- Now interactive! -->
*/
```

**Visual Timeline:**
```typescript
/*
0ms:   Server sends HTML
       |
       v
       ┌───────────────────┐
       │ User sees content  │  ✅ Fast!
       │ (Static HTML)      │  ❌ Not interactive
       └───────────────────┘
       |
1000ms: JS downloads
       |
1500ms: React hydrates
       |
       v
       ┌───────────────────┐
       │ User can interact  │  ✅ Interactive!
       │ (Hydrated app)     │  ✅ Event handlers work
       └───────────────────┘

Key Metric: Time to Interactive (TTI)
*/
```

#### 2. Hydration vs Client-Side Rendering

**Client-Side Rendering (SPA):**
```typescript
// 1. Server sends minimal HTML
/*
<html>
  <body>
    <div id="root"></div>  <!-- Empty! -->
    <script src="bundle.js"></script>
  </body>
</html>
*/

// 2. JavaScript runs
ReactDOM.render(<App />, document.getElementById('root'));

// 3. React renders entire tree
// User sees: blank page → loading → content

// Timeline:
// 0ms:    Blank page
// 1000ms: JS downloads
// 1500ms: React renders
// 1500ms: Content visible + interactive

// Problem: Slow First Contentful Paint (FCP)
```

**Server-Side Rendering + Hydration:**
```typescript
// 1. Server sends full HTML
/*
<html>
  <body>
    <div id="root">
      <header><h1>My App</h1></header>
      <main><p>Welcome!</p></main>
      <footer>Copyright 2024</footer>
    </div>
    <script src="bundle.js"></script>
  </body>
</html>
*/

// 2. JavaScript runs
ReactDOM.hydrateRoot(document.getElementById('root'), <App />);

// 3. React hydrates existing DOM
// User sees: content (instant!) → interactive

// Timeline:
// 0ms:    Content visible (from HTML)
// 1000ms: JS downloads
// 1500ms: React hydrates
// 1500ms: Interactive

// Benefit: Fast First Contentful Paint (FCP)
```

#### 3. How Hydration Works

**Server-Side (Generate HTML):**
```typescript
// server.tsx
import { renderToString } from 'react-dom/server';
import App from './App';

function handleRequest(req, res) {
  // Render React to HTML string
  const html = renderToString(<App />);
  
  // Send complete HTML
  res.send(`
    <!DOCTYPE html>
    <html>
      <head>
        <title>My App</title>
      </head>
      <body>
        <div id="root">${html}</div>
        <script src="/bundle.js"></script>
      </body>
    </html>
  `);
}
```

**Client-Side (Hydrate):**
```typescript
// client.tsx
import { hydrateRoot } from 'react-dom/client';
import App from './App';

// Hydrate existing HTML
const root = document.getElementById('root');
hydrateRoot(root, <App />);

// React process:
// 1. Build virtual DOM tree
// 2. Compare with existing DOM
// 3. Attach event listeners
// 4. Initialize state
// 5. Set up effects
```

**Component Example:**
```typescript
// App.tsx - Same code runs on server and client
function Counter() {
  const [count, setCount] = useState(0);
  
  // On server: runs once, returns HTML
  // On client: hydrates, attaches onClick
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}

// Server renders:
// <div>
//   <p>Count: 0</p>
//   <button>Increment</button>
// </div>

// Client hydrates:
// Attaches onClick handler to button
// Now clicking works!
```

#### 4. Hydration Challenges

**Hydration Mismatch:**
```typescript
// ❌ BAD: Different content on server vs client
function BadComponent() {
  // Server: uses server date
  // Client: uses client date
  // Result: Mismatch!
  return <div>Current time: {new Date().toISOString()}</div>;
}

// Error in console:
// "Hydration failed because the initial UI does not match 
//  what was rendered on the server."

// ✅ GOOD: Consistent rendering
function GoodComponent({ serverTime }: { serverTime: string }) {
  // Both use same value from props
  return <div>Server time: {serverTime}</div>;
}

// Or use useEffect for client-only code
function GoodComponent2() {
  const [time, setTime] = useState<string | null>(null);
  
  useEffect(() => {
    // Only runs on client after hydration
    setTime(new Date().toISOString());
  }, []);
  
  if (!time) {
    // Server and initial client render
    return <div>Loading time...</div>;
  }
  
  // After hydration
  return <div>Client time: {time}</div>;
}
```

**Common Mismatch Causes:**
```typescript
// 1. Date/Time
function TimeDisplay() {
  const [mounted, setMounted] = useState(false);
  
  useEffect(() => {
    setMounted(true);
  }, []);
  
  if (!mounted) return <div>--:--</div>;
  return <div>{new Date().toLocaleTimeString()}</div>;
}

// 2. Random values
function RandomColor() {
  const [color, setColor] = useState<string | null>(null);
  
  useEffect(() => {
    setColor(`#${Math.random().toString(16).slice(2, 8)}`);
  }, []);
  
  return <div style={{ backgroundColor: color || 'gray' }}>Hello</div>;
}

// 3. Browser-only APIs
function WindowSize() {
  const [size, setSize] = useState({ width: 0, height: 0 });
  
  useEffect(() => {
    // window not available on server
    setSize({
      width: window.innerWidth,
      height: window.innerHeight,
    });
  }, []);
  
  return <div>{size.width} x {size.height}</div>;
}

// 4. localStorage/sessionStorage
function StoredValue() {
  const [value, setValue] = useState<string | null>(null);
  
  useEffect(() => {
    setValue(localStorage.getItem('key'));
  }, []);
  
  return <div>{value}</div>;
}
```

**Suppressing Hydration Warnings:**
```typescript
// Use suppressHydrationWarning for intentional mismatches
function TimeStamp() {
  return (
    <time suppressHydrationWarning>
      {new Date().toISOString()}
    </time>
  );
}

// Use sparingly - only when you know what you're doing!
```

#### 5. Optimizing Hydration

**Selective Hydration (React 18+):**
```typescript
// Prioritize important components
import { Suspense } from 'react';

function App() {
  return (
    <div>
      {/* Critical content - hydrates first */}
      <Header />
      <MainContent />
      
      {/* Non-critical - hydrates later */}
      <Suspense fallback={<CommentsSkeleton />}>
        <Comments />
      </Suspense>
      
      <Suspense fallback={<SidebarSkeleton />}>
        <Sidebar />
      </Suspense>
    </div>
  );
}

// React hydrates in priority order:
// 1. Header, MainContent
// 2. User interacts with Comments? → Hydrate Comments first!
// 3. Otherwise hydrate in order
```

**Progressive Hydration:**
```typescript
// Hydrate components as they become visible
import { lazy, Suspense } from 'react';

const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <div>
      <Header />
      
      {/* Only loads/hydrates when scrolled into view */}
      <Suspense fallback={<div>Loading...</div>}>
        <HeavyComponent />
      </Suspense>
    </div>
  );
}
```

**Partial Hydration:**
```typescript
// Don't hydrate static content
function Article({ content }: { content: string }) {
  return (
    <article>
      {/* Static content - no hydration needed */}
      <div dangerouslySetInnerHTML={{ __html: content }} />
      
      {/* Interactive - needs hydration */}
      <CommentForm />
    </article>
  );
}
```

**Islands Architecture (Astro, Fresh):**
```typescript
/*
Concept: Only hydrate interactive "islands"

Page Layout:
┌───────────────────────────┐
│ Header (static)               │
├───────────────────────────┤
│ ┌──────────────────────┐  │
│ │ Search (interactive) │  │ ← Hydrate
│ └──────────────────────┘  │
│ Article content (static)       │
│ ┌──────────────────────┐  │
│ │ Comments (interactive)│  │ ← Hydrate
│ └──────────────────────┘  │
│ Footer (static)                │
└───────────────────────────┘

Result: Minimal JavaScript, fast hydration
*/
```

#### 6. Next.js Hydration

**Automatic Hydration:**
```typescript
// Next.js handles hydration automatically

// pages/index.tsx
export default function Home({ data }: { data: any }) {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <h1>{data.title}</h1>
      <button onClick={() => setCount(count + 1)}>
        Count: {count}
      </button>
    </div>
  );
}

export async function getServerSideProps() {
  const data = await fetchData();
  return { props: { data } };
}

// Next.js:
// 1. Renders page on server
// 2. Sends HTML to browser
// 3. Automatically hydrates on client
```

**Client-Only Components:**
```typescript
import dynamic from 'next/dynamic';

// Don't SSR, only render on client
const DynamicComponent = dynamic(
  () => import('./HeavyComponent'),
  { ssr: false }
);

export default function Page() {
  return (
    <div>
      <h1>My Page</h1>
      {/* Only renders on client */}
      <DynamicComponent />
    </div>
  );
}
```

#### 7. Best Practices

**1. Ensure Server/Client Consistency:**
```typescript
// ✅ Pass server data to client
export async function getServerSideProps() {
  return {
    props: {
      serverTime: new Date().toISOString(),
    },
  };
}

export default function Page({ serverTime }: { serverTime: string }) {
  return <div>Page rendered at: {serverTime}</div>;
}
```

**2. Use useEffect for Client-Only Code:**
```typescript
function Component() {
  const [mounted, setMounted] = useState(false);
  
  useEffect(() => {
    setMounted(true);
  }, []);
  
  // Server and initial client render
  if (!mounted) {
    return <div>Loading...</div>;
  }
  
  // After hydration - can use browser APIs
  return <div>Width: {window.innerWidth}</div>;
}
```

**3. Minimize JavaScript:**
```typescript
// Less JS = faster hydration

// Use static HTML where possible
// Lazy load heavy components
// Code split aggressively
```

**4. Optimize Bundle Size:**
```typescript
// Smaller bundles = faster download = faster hydration

// Use dynamic imports
const Chart = dynamic(() => import('./Chart'));

// Tree-shake unused code
// Minimize dependencies
```

**5. Monitor Hydration Performance:**
```typescript
// Measure Time to Interactive (TTI)
if (typeof window !== 'undefined') {
  window.addEventListener('load', () => {
    const [navigation] = performance.getEntriesByType('navigation');
    console.log('TTI:', navigation.domInteractive);
  });
}
```

#### 8. Debugging Hydration Issues

**React DevTools:**
```typescript
// Enable highlighting hydration errors
// In console:
// React DevTools > Settings > Highlight hydration errors
```

**Console Warnings:**
```typescript
// React provides detailed error messages:
/*
Warning: Text content did not match. 
Server: "Server time" 
Client: "Client time"
*/

// Find the component causing the mismatch
// Fix by ensuring consistent rendering
```

**Debug Component:**
```typescript
function DebugHydration({ children }: { children: React.ReactNode }) {
  const [hydrated, setHydrated] = useState(false);
  
  useEffect(() => {
    console.log('Component hydrated');
    setHydrated(true);
  }, []);
  
  return (
    <div data-hydrated={hydrated}>
      {children}
    </div>
  );
}
```

#### Summary

**Hydration** is the process of making server-rendered HTML interactive:

**Benefits:**
- ✅ Fast First Contentful Paint (FCP)
- ✅ Better SEO (content immediately visible)
- ✅ Improved perceived performance
- ✅ Works without JavaScript (initially)

**Challenges:**
- ❌ Hydration mismatches
- ❌ Increased Time to Interactive (TTI)
- ❌ Larger JavaScript bundles
- ❌ Complexity of SSR setup

**Best Practices:**
1. Ensure server/client consistency
2. Use `useEffect` for client-only code
3. Minimize JavaScript bundles
4. Use selective/progressive hydration
5. Monitor hydration performance

**Key Metrics:**
- **FCP** (First Contentful Paint): When content appears
- **TTI** (Time to Interactive): When page is interactive
- Goal: Minimize gap between FCP and TTI

**Modern Approaches:**
- Selective hydration (React 18+)
- Progressive hydration
- Islands architecture
- Streaming SSR

> "Hydration bridges the gap between fast initial load (SSR) and rich interactivity (client-side React)."

</details>

---

## Completion Status

Topic 15: SSR & SSG - **COMPLETE** ✅

- Q141: Server-Side Rendering (SSR) ✅
- Q142: Static Site Generation (SSG) ✅
- Q143: SSR vs SSG differences ✅
- Q144: Next.js framework ✅
- Q145: App Router in Next.js 13+ ✅
- Q146: Server Actions in Next.js ✅
- Q147: Incremental Static Regeneration (ISR) ✅
- Q148: Next.js data fetching ✅
- Q149: Remix framework comparison ✅
- Q150: Hydration in SSR ✅

**All 10 questions completed!** 🎉
