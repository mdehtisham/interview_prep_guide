# React 18+ Features

> Advanced / Senior Level (3-5 years)

---

## Questions

81. What are React Server Components?
82. What is the difference between Server and Client Components?
83. What is Concurrent Rendering in React 18?
84. What is useTransition hook and when do you use it?
85. What is useDeferredValue hook?
86. What is Automatic Batching in React 18?
87. What is the new Suspense SSR architecture?
88. What is startTransition API?
89. How do Server Components improve performance?
90. What is Streaming SSR?

---

## Detailed Answers

### 81. What are React Server Components?

<details>
<summary>View Answer</summary>

**React Server Components (RSC)**

A new paradigm introduced in React 18 that allows components to render on the server and send minimal data to the client, reducing bundle size and improving performance.

---

## What Are Server Components?

**Server Components** are React components that run **only on the server** and never ship JavaScript to the client.

### Key Characteristics

1. **Run on server only** - Never execute in browser
2. **Zero bundle size** - Don't add to client JavaScript
3. **Direct backend access** - Can query databases, read files
4. **Async by default** - Can be async functions
5. **No client interactivity** - No useState, useEffect, event handlers
6. **Send only data** - Client receives rendered output

---

## Basic Example

```jsx
// app/page.js (Server Component by default in Next.js 13+)
import db from './database';

// Server Component - runs on server
async function ProductList() {
  // Direct database access (no API needed)
  const products = await db.products.findMany();
  
  return (
    <div>
      <h1>Products</h1>
      {products.map(product => (
        <div key={product.id}>
          <h2>{product.name}</h2>
          <p>${product.price}</p>
        </div>
      ))}
    </div>
  );
}

export default ProductList;
```

**Benefits:**
- No database library shipped to client
- Direct database access (no API endpoint)
- Faster initial load
- Better SEO

---

## Server vs Client Components

### Server Component (Default)

```jsx
// app/Header.jsx
// Server Component (default in App Router)

async function Header() {
  // Can access backend directly
  const user = await getUser();
  
  return (
    <header>
      <h1>Welcome {user.name}</h1>
    </header>
  );
}
```

### Client Component (Opt-in)

```jsx
'use client';  // Directive to mark as Client Component

import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

---

## When to Use Server Components

### 1. Fetching Data

```jsx
// app/posts/page.jsx
import { db } from '@/lib/database';

async function PostsPage() {
  const posts = await db.post.findMany({
    orderBy: { createdAt: 'desc' }
  });
  
  return (
    <div>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.content}</p>
        </article>
      ))}
    </div>
  );
}

export default PostsPage;
```

---

### 2. Accessing Backend Resources

```jsx
// app/config/page.jsx
import fs from 'fs/promises';
import path from 'path';

async function ConfigPage() {
  // Read file from filesystem
  const configPath = path.join(process.cwd(), 'config.json');
  const config = JSON.parse(await fs.readFile(configPath, 'utf-8'));
  
  return (
    <div>
      <h1>Configuration</h1>
      <pre>{JSON.stringify(config, null, 2)}</pre>
    </div>
  );
}

export default ConfigPage;
```

---

### 3. Keeping Sensitive Data on Server

```jsx
// app/dashboard/page.jsx

async function Dashboard() {
  // API keys stay on server
  const response = await fetch('https://api.example.com/data', {
    headers: {
      'Authorization': `Bearer ${process.env.API_KEY}`  // Never exposed to client
    }
  });
  
  const data = await response.json();
  
  return <div>{/* Render data */}</div>;
}
```

---

### 4. Large Dependencies

```jsx
// app/markdown/page.jsx
import { marked } from 'marked';  // Heavy library stays on server

async function MarkdownPage({ content }) {
  const html = marked(content);
  
  return <div dangerouslySetInnerHTML={{ __html: html }} />;
}
```

**Benefit:** `marked` library (50KB) doesn't go to client

---

## Composition Pattern

**Mix Server and Client Components:**

```jsx
// app/page.jsx (Server Component)
import ClientButton from './ClientButton';

async function Page() {
  const data = await fetchData();  // Server-side data fetching
  
  return (
    <div>
      <h1>Server Component</h1>
      <p>Data: {data}</p>
      
      {/* Client Component for interactivity */}
      <ClientButton />
    </div>
  );
}
```

```jsx
// app/ClientButton.jsx (Client Component)
'use client';

import { useState } from 'react';

export default function ClientButton() {
  const [count, setCount] = useState(0);
  
  return (
    <button onClick={() => setCount(count + 1)}>
      Clicked {count} times
    </button>
  );
}
```

---

## Data Passing

### Server Component → Client Component (Props)

```jsx
// app/page.jsx (Server)
import ClientList from './ClientList';

async function Page() {
  const products = await db.products.findMany();
  
  // Pass data as props
  return <ClientList products={products} />;
}
```

```jsx
// app/ClientList.jsx (Client)
'use client';

import { useState } from 'react';

export default function ClientList({ products }) {
  const [filter, setFilter] = useState('');
  
  const filtered = products.filter(p => 
    p.name.includes(filter)
  );
  
  return (
    <div>
      <input 
        value={filter}
        onChange={e => setFilter(e.target.value)}
        placeholder="Filter products..."
      />
      {filtered.map(p => <div key={p.id}>{p.name}</div>)}
    </div>
  );
}
```

---

## Limitations of Server Components

**Cannot use:**

1. **State:** `useState`, `useReducer`
2. **Effects:** `useEffect`, `useLayoutEffect`
3. **Event handlers:** `onClick`, `onChange`, etc.
4. **Browser APIs:** `window`, `localStorage`, etc.
5. **Context:** `useContext` (for client state)
6. **Lifecycle hooks**

```jsx
// ❌ WRONG - Server Component
async function ServerComponent() {
  const [count, setCount] = useState(0);  // ❌ Error
  
  useEffect(() => {  // ❌ Error
    console.log('mounted');
  }, []);
  
  return (
    <button onClick={() => {}}>  {/* ❌ Error */}
      Click
    </button>
  );
}
```

```jsx
// ✅ CORRECT - Client Component
'use client';

import { useState, useEffect } from 'react';

function ClientComponent() {
  const [count, setCount] = useState(0);  // ✅ OK
  
  useEffect(() => {  // ✅ OK
    console.log('mounted');
  }, []);
  
  return (
    <button onClick={() => setCount(count + 1)}>  {/* ✅ OK */}
      Count: {count}
    </button>
  );
}
```

---

## Complete Example: Blog Post

```jsx
// app/posts/[id]/page.jsx (Server Component)
import { db } from '@/lib/database';
import CommentForm from './CommentForm';
import LikeButton from './LikeButton';

async function PostPage({ params }) {
  // Server-side data fetching
  const post = await db.post.findUnique({
    where: { id: params.id },
    include: {
      author: true,
      comments: {
        include: { author: true },
        orderBy: { createdAt: 'desc' }
      }
    }
  });
  
  if (!post) {
    return <div>Post not found</div>;
  }
  
  return (
    <article>
      <h1>{post.title}</h1>
      <p>By {post.author.name}</p>
      
      <div>{post.content}</div>
      
      {/* Client Component for interactivity */}
      <LikeButton postId={post.id} initialLikes={post.likes} />
      
      <h2>Comments ({post.comments.length})</h2>
      {post.comments.map(comment => (
        <div key={comment.id}>
          <strong>{comment.author.name}:</strong> {comment.text}
        </div>
      ))}
      
      {/* Client Component for form */}
      <CommentForm postId={post.id} />
    </article>
  );
}

export default PostPage;
```

```jsx
// app/posts/[id]/LikeButton.jsx (Client Component)
'use client';

import { useState } from 'react';

export default function LikeButton({ postId, initialLikes }) {
  const [likes, setLikes] = useState(initialLikes);
  const [loading, setLoading] = useState(false);
  
  const handleLike = async () => {
    setLoading(true);
    
    const response = await fetch(`/api/posts/${postId}/like`, {
      method: 'POST'
    });
    
    if (response.ok) {
      setLikes(likes + 1);
    }
    
    setLoading(false);
  };
  
  return (
    <button onClick={handleLike} disabled={loading}>
      ❤️ {likes} Likes
    </button>
  );
}
```

```jsx
// app/posts/[id]/CommentForm.jsx (Client Component)
'use client';

import { useState } from 'react';

export default function CommentForm({ postId }) {
  const [comment, setComment] = useState('');
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    await fetch(`/api/posts/${postId}/comments`, {
      method: 'POST',
      body: JSON.stringify({ text: comment })
    });
    
    setComment('');
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <textarea
        value={comment}
        onChange={e => setComment(e.target.value)}
        placeholder="Add a comment..."
      />
      <button type="submit">Post Comment</button>
    </form>
  );
}
```

---

## Benefits of Server Components

### 1. Smaller Bundle Size

**Before (Client Component):**
```jsx
'use client';

import { marked } from 'marked';  // 50KB added to bundle

function Post({ content }) {
  const html = marked(content);
  return <div dangerouslySetInnerHTML={{ __html: html }} />;
}

// Bundle: +50KB
```

**After (Server Component):**
```jsx
// Server Component (no 'use client')
import { marked } from 'marked';  // Runs on server, 0KB to client

function Post({ content }) {
  const html = marked(content);
  return <div dangerouslySetInnerHTML={{ __html: html }} />;
}

// Bundle: +0KB
```

---

### 2. Direct Backend Access

**Before:**
```jsx
// Client Component
'use client';

function Products() {
  const [products, setProducts] = useState([]);
  
  useEffect(() => {
    fetch('/api/products')  // Extra API route needed
      .then(r => r.json())
      .then(setProducts);
  }, []);
  
  return <div>{/* render products */}</div>;
}

// Need separate API route:
// app/api/products/route.js
export async function GET() {
  const products = await db.products.findMany();
  return Response.json(products);
}
```

**After:**
```jsx
// Server Component
async function Products() {
  const products = await db.products.findMany();  // Direct access
  
  return <div>{/* render products */}</div>;
}

// No API route needed!
```

---

### 3. Better Performance

- **Faster initial load** - Less JavaScript to download/parse
- **Better caching** - Server responses cached on CDN
- **Parallel data fetching** - Multiple Server Components fetch in parallel
- **No waterfall** - No client-side fetch chains

---

## How Server Components Work

### 1. Render on Server

```jsx
// Server renders this component
async function UserProfile() {
  const user = await db.user.findUnique({ where: { id: 1 } });
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

### 2. Send to Client

**Not HTML, but a special format:**

```json
[
  "div",
  null,
  [
    ["h1", null, "John Doe"],
    ["p", null, "john@example.com"]
  ]
]
```

### 3. Client Reconstructs

Client receives this data and reconstructs the component tree.

---

## Comparison Table

| Feature | Server Component | Client Component |
|---------|-----------------|------------------|
| **Runs on** | Server only | Client (browser) |
| **Bundle size** | Zero | Added to bundle |
| **Data fetching** | Direct backend access | Fetch from API |
| **State** | ❌ No | ✅ Yes |
| **Effects** | ❌ No | ✅ Yes |
| **Event handlers** | ❌ No | ✅ Yes |
| **Browser APIs** | ❌ No | ✅ Yes |
| **Can be async** | ✅ Yes | ❌ No |
| **Access secrets** | ✅ Yes (safe) | ❌ No (exposed) |
| **Import server libs** | ✅ Yes | ❌ No |
| **SEO** | ✅ Better | ⚠️ Depends |
| **Interactivity** | ❌ No | ✅ Yes |

---

## Best Practices

**1. Default to Server Components**
```jsx
// Server Component by default
async function Page() {
  // Server-side logic
}
```

**2. Use Client Components only when needed**
```jsx
// Only add 'use client' when you need:
// - State
// - Effects  
// - Event handlers
// - Browser APIs
'use client';

function Interactive() {
  const [state, setState] = useState();
  // ...
}
```

**3. Move 'use client' boundary down**
```jsx
// ❌ Bad - entire component tree is client
'use client';

function Page() {
  return (
    <div>
      <Header />  {/* Client */}
      <Content />  {/* Client */}
      <Button />  {/* Only this needs client! */}
    </div>
  );
}
```

```jsx
// ✅ Good - only Button is client
function Page() {  // Server
  return (
    <div>
      <Header />  {/* Server */}
      <Content />  {/* Server */}
      <ClientButton />  {/* Client */}
    </div>
  );
}
```

**4. Pass data down**
```jsx
// ✅ Server fetches, Client renders
async function Page() {
  const data = await fetchData();
  return <ClientComponent data={data} />;
}
```

---

**Interview Tips:**
- **Server Components** = run on server only
- **Zero bundle size** = don't ship JavaScript to client
- **Direct backend access** = no API routes needed
- **Can be async** = await in component body
- **No interactivity** = no state, effects, or event handlers
- **'use client'** directive = opt into Client Component
- **Default to Server** = use Client only when needed
- **Composition** = mix Server and Client components
- **Pass data via props** = Server → Client
- **Better performance** = less JavaScript, faster loads
- **Keep secrets safe** = API keys stay on server
- **Large dependencies** = stay on server (0KB to client)
- **Direct database access** = no API endpoint needed
- **Better SEO** = server-rendered content
- **Parallel fetching** = multiple components fetch independently
- **No waterfalls** = avoid client-side fetch chains
- **Streaming** = send components as they're ready
- **Not just SSR** = different paradigm, more granular
- **Next.js 13+ App Router** = enables Server Components
- **React 18+** = required for Server Components
- **Move 'use client' down** = minimize client bundle
- **Server = fetch**, **Client = interact**
- **Benefits**: smaller bundles, direct backend access, better performance, improved SEO
- **Limitations**: no state, effects, event handlers, or browser APIs
- **New mental model** = server-first by default

</details>

---

### 82. What is the difference between Server and Client Components?

<details>
<summary>View Answer</summary>

**Server vs Client Components**

A fundamental distinction in modern React architecture that determines where components execute and what capabilities they have.

---

## Quick Comparison

| Aspect | Server Component | Client Component |
|--------|-----------------|------------------|
| **Execution** | Server only | Browser (client) |
| **Marker** | Default (no marker) | `'use client'` directive |
| **Bundle size** | 0KB (not shipped) | Added to bundle |
| **Async** | ✅ Can be async | ❌ Cannot be async |
| **State** | ❌ No useState | ✅ Yes |
| **Effects** | ❌ No useEffect | ✅ Yes |
| **Events** | ❌ No onClick, etc. | ✅ Yes |
| **Browser APIs** | ❌ No window, localStorage | ✅ Yes |
| **Backend access** | ✅ Direct DB, filesystem | ❌ Must use API |
| **Secrets** | ✅ Can use safely | ❌ Exposed to client |
| **SEO** | ✅ Excellent | ⚠️ Depends |
| **Interactivity** | ❌ No | ✅ Yes |
| **Dependencies** | Stay on server | Shipped to client |

---

## Server Components

### Definition

**Server Components** render exclusively on the server and send only their output to the client.

```jsx
// app/products/page.jsx
// Server Component (default, no marker needed)

import { db } from '@/lib/database';

async function ProductsPage() {
  // ✅ Direct database access
  const products = await db.product.findMany();
  
  return (
    <div>
      <h1>Products</h1>
      {products.map(p => (
        <div key={p.id}>
          <h2>{p.name}</h2>
          <p>${p.price}</p>
        </div>
      ))}
    </div>
  );
}

export default ProductsPage;
```

### Characteristics

**✅ Can do:**
- Fetch data directly from database
- Read files from filesystem
- Access environment variables safely
- Use Node.js APIs
- Be async functions
- Import server-only libraries
- Call other Server Components
- Render Client Components as children

**❌ Cannot do:**
- Use `useState`, `useReducer`
- Use `useEffect`, `useLayoutEffect`
- Add event handlers (`onClick`, `onChange`, etc.)
- Use browser APIs (`window`, `localStorage`, etc.)
- Use Context for client state
- Import client-only libraries

---

## Client Components

### Definition

**Client Components** run in the browser and provide interactivity.

```jsx
// app/components/Counter.jsx
'use client';  // ⚠️ Required directive

import { useState } from 'react';

function Counter() {
  // ✅ Can use state
  const [count, setCount] = useState(0);
  
  // ✅ Can handle events
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}

export default Counter;
```

### Characteristics

**✅ Can do:**
- Use `useState`, `useReducer`
- Use `useEffect`, `useLayoutEffect`
- Add event handlers
- Use browser APIs
- Use Context for client state
- Import client libraries
- Provide interactivity
- Access browser features

**❌ Cannot do:**
- Be async functions
- Access backend directly (must use API)
- Use Node.js APIs
- Read filesystem
- Access server-only secrets safely

---

## Detailed Examples

### Server Component Features

**1. Direct Database Access**

```jsx
// Server Component
import { db } from '@/lib/prisma';

async function UserProfile({ userId }) {
  // Direct database query
  const user = await db.user.findUnique({
    where: { id: userId },
    include: { posts: true }
  });
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>Posts: {user.posts.length}</p>
    </div>
  );
}
```

---

**2. File System Access**

```jsx
// Server Component
import fs from 'fs/promises';
import path from 'path';

async function ChangelogPage() {
  // Read file from disk
  const filePath = path.join(process.cwd(), 'CHANGELOG.md');
  const content = await fs.readFile(filePath, 'utf-8');
  
  return (
    <div>
      <h1>Changelog</h1>
      <pre>{content}</pre>
    </div>
  );
}
```

---

**3. Environment Variables (Safe)**

```jsx
// Server Component
async function Dashboard() {
  // API key stays on server, never exposed
  const response = await fetch('https://api.example.com/data', {
    headers: {
      'Authorization': `Bearer ${process.env.SECRET_API_KEY}`
    }
  });
  
  const data = await response.json();
  
  return <div>{/* Render data */}</div>;
}
```

---

**4. Server-only Libraries (Zero Bundle Cost)**

```jsx
// Server Component
import { marked } from 'marked';  // 50KB stays on server
import highlightjs from 'highlight.js';  // 100KB stays on server

function BlogPost({ markdown }) {
  const html = marked(markdown);
  const highlighted = highlightjs.highlightAuto(html);
  
  return <div dangerouslySetInnerHTML={{ __html: highlighted.value }} />;
}

// Client bundle: +0KB from these libraries
```

---

### Client Component Features

**1. State Management**

```jsx
'use client';

import { useState } from 'react';

function ToggleButton() {
  const [isOn, setIsOn] = useState(false);
  
  return (
    <button onClick={() => setIsOn(!isOn)}>
      {isOn ? 'ON' : 'OFF'}
    </button>
  );
}
```

---

**2. Effects & Lifecycle**

```jsx
'use client';

import { useState, useEffect } from 'react';

function WindowSize() {
  const [width, setWidth] = useState(0);
  
  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth);
    
    handleResize();
    window.addEventListener('resize', handleResize);
    
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  return <div>Window width: {width}px</div>;
}
```

---

**3. Event Handlers**

```jsx
'use client';

import { useState } from 'react';

function SearchBox() {
  const [query, setQuery] = useState('');
  
  return (
    <input
      value={query}
      onChange={e => setQuery(e.target.value)}
      onFocus={() => console.log('focused')}
      onBlur={() => console.log('blurred')}
      placeholder="Search..."
    />
  );
}
```

---

**4. Browser APIs**

```jsx
'use client';

import { useState, useEffect } from 'react';

function LocalStorage() {
  const [value, setValue] = useState('');
  
  useEffect(() => {
    // Access localStorage
    const stored = localStorage.getItem('myKey');
    if (stored) setValue(stored);
  }, []);
  
  const handleSave = () => {
    localStorage.setItem('myKey', value);
  };
  
  return (
    <div>
      <input value={value} onChange={e => setValue(e.target.value)} />
      <button onClick={handleSave}>Save</button>
    </div>
  );
}
```

---

## Composition Patterns

### Pattern 1: Server Component with Client Children

```jsx
// app/page.jsx (Server Component)
import ClientSidebar from './ClientSidebar';
import ClientButton from './ClientButton';

async function Page() {
  // Server-side data fetching
  const data = await fetchData();
  
  return (
    <div>
      <h1>Server Component</h1>
      <p>Data from server: {data}</p>
      
      {/* Client Components for interactivity */}
      <ClientSidebar />
      <ClientButton />
    </div>
  );
}
```

---

### Pattern 2: Pass Data from Server to Client

```jsx
// app/page.jsx (Server)
import ProductList from './ProductList';

async function Page() {
  const products = await db.product.findMany();
  
  // Pass server data to client component
  return <ProductList initialProducts={products} />;
}
```

```jsx
// app/ProductList.jsx (Client)
'use client';

import { useState } from 'react';

function ProductList({ initialProducts }) {
  const [products, setProducts] = useState(initialProducts);
  const [filter, setFilter] = useState('');
  
  const filtered = products.filter(p => 
    p.name.toLowerCase().includes(filter.toLowerCase())
  );
  
  return (
    <div>
      <input 
        value={filter}
        onChange={e => setFilter(e.target.value)}
        placeholder="Filter..."
      />
      {filtered.map(p => <div key={p.id}>{p.name}</div>)}
    </div>
  );
}
```

---

### Pattern 3: Server Component as Child of Client

**❌ Cannot import Server Component in Client Component:**

```jsx
'use client';

import ServerComponent from './ServerComponent';  // ❌ Error

function ClientComponent() {
  return <ServerComponent />;  // ❌ Won't work
}
```

**✅ Pass Server Component as children prop:**

```jsx
// app/page.jsx (Server)
import ClientWrapper from './ClientWrapper';
import ServerContent from './ServerContent';

function Page() {
  return (
    <ClientWrapper>
      <ServerContent />  {/* Server Component */}
    </ClientWrapper>
  );
}
```

```jsx
// ClientWrapper.jsx (Client)
'use client';

import { useState } from 'react';

function ClientWrapper({ children }) {
  const [isOpen, setIsOpen] = useState(true);
  
  return (
    <div>
      <button onClick={() => setIsOpen(!isOpen)}>Toggle</button>
      {isOpen && children}  {/* Server Component rendered here */}
    </div>
  );
}
```

---

## When to Use Each

### Use Server Components When:

- ✅ Fetching data from database
- ✅ Accessing backend resources (files, APIs with secrets)
- ✅ Keeping sensitive information on server
- ✅ Using large dependencies (reduce bundle size)
- ✅ Rendering static content
- ✅ No interactivity needed

**Example:**
```jsx
// Server Component - perfect for this
async function BlogPost({ id }) {
  const post = await db.post.findUnique({ where: { id } });
  return <article>{post.content}</article>;
}
```

---

### Use Client Components When:

- ✅ Need interactivity (clicks, typing, etc.)
- ✅ Using state or effects
- ✅ Accessing browser APIs
- ✅ Using event listeners
- ✅ Using client-only libraries
- ✅ Using Context for client state

**Example:**
```jsx
// Client Component - needs interactivity
'use client';

function LikeButton() {
  const [likes, setLikes] = useState(0);
  return (
    <button onClick={() => setLikes(likes + 1)}>
      ❤️ {likes}
    </button>
  );
}
```

---

## Common Pitfalls

### Pitfall 1: Forgetting 'use client'

```jsx
// ❌ Error - needs 'use client'
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);  // Error in Server Component
  return <div>{count}</div>;
}
```

```jsx
// ✅ Correct
'use client';

import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  return <div>{count}</div>;
}
```

---

### Pitfall 2: Making Everything Client

```jsx
// ❌ Bad - entire app is client
'use client';

function App() {
  return (
    <div>
      <Header />  {/* Unnecessarily client */}
      <Content />  {/* Unnecessarily client */}
      <Footer />  {/* Unnecessarily client */}
    </div>
  );
}
```

```jsx
// ✅ Good - only interactive parts are client
function App() {  // Server by default
  return (
    <div>
      <Header />  {/* Server */}
      <Content />  {/* Server */}
      <InteractiveFooter />  {/* Client - only this needs it */}
    </div>
  );
}
```

---

### Pitfall 3: Importing Server Code in Client

```jsx
// ❌ Error
'use client';

import { db } from './database';  // ❌ Server-only module

function Component() {
  // Can't use db here
}
```

```jsx
// ✅ Correct - fetch from API
'use client';

function Component() {
  const [data, setData] = useState([]);
  
  useEffect(() => {
    fetch('/api/data')
      .then(r => r.json())
      .then(setData);
  }, []);
}
```

---

### Pitfall 4: Async Client Components

```jsx
// ❌ Error - Client Components cannot be async
'use client';

async function ClientComponent() {  // ❌ Error
  const data = await fetchData();
  return <div>{data}</div>;
}
```

```jsx
// ✅ Correct - use useEffect for async in Client
'use client';

import { useState, useEffect } from 'react';

function ClientComponent() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetchData().then(setData);
  }, []);
  
  return <div>{data}</div>;
}
```

---

## Performance Comparison

### Bundle Size Impact

**Scenario: Markdown rendering**

```jsx
// Client Component approach
'use client';
import { marked } from 'marked';  // +50KB

function Post({ content }) {
  return <div dangerouslySetInnerHTML={{ __html: marked(content) }} />;
}

// Total bundle: Base + 50KB
```

```jsx
// Server Component approach
import { marked } from 'marked';  // 0KB to client

function Post({ content }) {
  return <div dangerouslySetInnerHTML={{ __html: marked(content) }} />;
}

// Total bundle: Base + 0KB
```

**Savings: 50KB per component like this**

---

### Data Fetching Comparison

**Client Component:**
```jsx
'use client';

function Products() {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetch('/api/products')
      .then(r => r.json())
      .then(data => {
        setProducts(data);
        setLoading(false);
      });
  }, []);
  
  if (loading) return <div>Loading...</div>;
  return <div>{/* render */}</div>;
}

// 1. Load JavaScript
// 2. Execute React
// 3. Render component
// 4. Fetch data (waterfall)
// 5. Re-render with data
```

**Server Component:**
```jsx
async function Products() {
  const products = await db.product.findMany();
  return <div>{/* render */}</div>;
}

// 1. Fetch data on server
// 2. Render with data
// 3. Send to client
// 4. Display immediately
```

**Faster:** Server Component avoids client-side waterfall

---

## Migration Guide

### Converting Client to Server

**Before (Client):**
```jsx
'use client';

import { useState, useEffect } from 'react';

function UserProfile() {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetch('/api/user')
      .then(r => r.json())
      .then(setUser);
  }, []);
  
  if (!user) return <div>Loading...</div>;
  
  return <div>{user.name}</div>;
}
```

**After (Server):**
```jsx
import { db } from './database';

async function UserProfile() {
  const user = await db.user.findFirst();
  
  return <div>{user.name}</div>;
}
```

**Benefits:**
- No loading state needed
- No useEffect
- Direct database access
- Faster rendering

---

## Complete Example: E-commerce Product Page

```jsx
// app/products/[id]/page.jsx (Server Component)
import { db } from '@/lib/database';
import AddToCartButton from './AddToCartButton';
import ReviewForm from './ReviewForm';

async function ProductPage({ params }) {
  // Server-side: Direct database access
  const product = await db.product.findUnique({
    where: { id: params.id },
    include: {
      reviews: {
        include: { user: true },
        orderBy: { createdAt: 'desc' }
      }
    }
  });
  
  // Server-side: Calculate average rating
  const avgRating = product.reviews.reduce((sum, r) => sum + r.rating, 0) / product.reviews.length;
  
  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <p className="price">${product.price}</p>
      <p>Rating: {avgRating.toFixed(1)} ⭐</p>
      
      {/* Client Component for interactivity */}
      <AddToCartButton productId={product.id} />
      
      <h2>Reviews ({product.reviews.length})</h2>
      {product.reviews.map(review => (
        <div key={review.id}>
          <strong>{review.user.name}</strong>
          <span>{' ⭐'.repeat(review.rating)}</span>
          <p>{review.text}</p>
        </div>
      ))}
      
      {/* Client Component for form */}
      <ReviewForm productId={product.id} />
    </div>
  );
}

export default ProductPage;
```

```jsx
// app/products/[id]/AddToCartButton.jsx (Client Component)
'use client';

import { useState } from 'react';
import { useCart } from '@/hooks/useCart';

function AddToCartButton({ productId }) {
  const [quantity, setQuantity] = useState(1);
  const { addItem } = useCart();
  
  const handleAdd = () => {
    addItem(productId, quantity);
  };
  
  return (
    <div>
      <input
        type="number"
        value={quantity}
        onChange={e => setQuantity(Number(e.target.value))}
        min="1"
      />
      <button onClick={handleAdd}>Add to Cart</button>
    </div>
  );
}

export default AddToCartButton;
```

```jsx
// app/products/[id]/ReviewForm.jsx (Client Component)
'use client';

import { useState } from 'react';

function ReviewForm({ productId }) {
  const [rating, setRating] = useState(5);
  const [text, setText] = useState('');
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    await fetch('/api/reviews', {
      method: 'POST',
      body: JSON.stringify({ productId, rating, text })
    });
    
    setText('');
    setRating(5);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <select value={rating} onChange={e => setRating(Number(e.target.value))}>
        {[1, 2, 3, 4, 5].map(n => <option key={n} value={n}>{n} ⭐</option>)}
      </select>
      
      <textarea
        value={text}
        onChange={e => setText(e.target.value)}
        placeholder="Write your review..."
      />
      
      <button type="submit">Submit Review</button>
    </form>
  );
}

export default ReviewForm;
```

---

**Interview Tips:**
- **Server Components** = default, no marker needed
- **Client Components** = need `'use client'` directive
- Server = **data fetching**, Client = **interactivity**
- Server can render Client, but not vice versa (use children prop)
- **Pass data down** from Server to Client via props
- **Zero bundle size** for Server Components
- Server can be **async**, Client cannot
- Server = no state/effects, Client = full React features
- **Default to Server**, opt into Client when needed
- **Move 'use client' down** the tree (minimize client code)
- Server accesses **backend directly** (DB, files, secrets)
- Client must use **API routes** for backend access
- **Large dependencies** stay on server = smaller bundles
- Better **SEO** with Server Components
- **Faster loads** with less JavaScript
- Client Components still **server-rendered** initially (SSR)
- **Composition** = mix both types strategically
- **Cannot import** Server Component in Client Component
- Use **children prop** to pass Server to Client
- **'use client'** applies to module and its imports
- Next.js 13+ **App Router** enables this architecture
- **Mental model**: Server = static/data, Client = dynamic/interactive
- **Bundle impact**: Each Client Component adds JavaScript
- **Performance**: Server Components reduce client-side work
- **Security**: Secrets safe in Server Components

</details>

---

### 83. What is Concurrent Rendering in React 18?

<details>
<summary>View Answer</summary>

**Concurrent Rendering**

A groundbreaking feature in React 18 that allows React to work on multiple renders simultaneously and interrupt less important work to prioritize urgent updates.

---

## What is Concurrent Rendering?

**Concurrent Rendering** is a new rendering mechanism where React can:

1. **Start rendering an update**
2. **Pause in the middle**
3. **Work on something more urgent**
4. **Resume or abandon the paused work**

### Before React 18 (Synchronous Rendering)

```
User types → React starts rendering → Blocks UI → Render completes → UI updates
                     ↓
              UI FROZEN (not responsive)
```

**Problem:** Once rendering starts, it must finish. UI becomes unresponsive.

---

### With React 18 (Concurrent Rendering)

```
User types → React starts rendering (low priority)
                     ↓
User clicks → React pauses first render
                     ↓
             Handles urgent click (high priority)
                     ↓
             Resumes or abandons first render
```

**Benefit:** React can interrupt long renders to keep UI responsive.

---

## Key Concepts

### 1. Interruptible Rendering

React can pause and resume rendering work.

```jsx
import { useState, useTransition } from 'react';

function App() {
  const [isPending, startTransition] = useTransition();
  const [input, setInput] = useState('');
  const [list, setList] = useState([]);
  
  const handleChange = (e) => {
    const value = e.target.value;
    
    // Urgent: Update input immediately
    setInput(value);
    
    // Non-urgent: Update list (can be interrupted)
    startTransition(() => {
      const newList = Array(20000)
        .fill(null)
        .map((_, i) => `${value} - ${i}`);
      setList(newList);
    });
  };
  
  return (
    <div>
      <input value={input} onChange={handleChange} />
      {isPending && <div>Loading...</div>}
      <ul>
        {list.map((item, i) => <li key={i}>{item}</li>)}
      </ul>
    </div>
  );
}
```

**How it works:**
- Input updates immediately (high priority)
- List rendering can be paused (low priority)
- If user keeps typing, React pauses/abandons list renders
- UI stays responsive

---

### 2. Multiple Priority Levels

React categorizes updates by priority:

| Priority | Examples | Behavior |
|----------|----------|----------|
| **Urgent** | User input, clicks | Must happen immediately |
| **Transition** | Navigation, filtering | Can be interrupted |
| **Deferred** | Analytics, logging | Lowest priority |

```jsx
import { useState, useTransition } from 'react';

function SearchPage() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();
  
  const handleSearch = (e) => {
    const value = e.target.value;
    
    // URGENT: Update input (high priority)
    setQuery(value);
    
    // TRANSITION: Update results (low priority, can be interrupted)
    startTransition(() => {
      const filtered = expensiveSearch(value);
      setResults(filtered);
    });
  };
  
  return (
    <div>
      <input value={query} onChange={handleSearch} placeholder="Search..." />
      {isPending ? <div>Searching...</div> : (
        <ul>
          {results.map(item => <li key={item.id}>{item.name}</li>)}
        </ul>
      )}
    </div>
  );
}
```

---

### 3. Time Slicing

React breaks work into small chunks and yields control back to browser.

```
Synchronous (Old):
[========== RENDER 100ms ==========] ← UI frozen

Concurrent (New):
[== 5ms ==] yield [== 5ms ==] yield [== 5ms ==] yield ...
     ↑              ↑              ↑
   Browser can   Handle user    Stay responsive
   paint         events
```

---

## Enabling Concurrent Features

### 1. Using createRoot (Required)

**Old (React 17):**
```jsx
import ReactDOM from 'react-dom';

// Legacy rendering (synchronous)
ReactDOM.render(<App />, document.getElementById('root'));
```

**New (React 18):**
```jsx
import { createRoot } from 'react-dom/client';

// Concurrent rendering (enables new features)
const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

⚠️ **Must use `createRoot`** to enable concurrent features.

---

### 2. Opt-in with Concurrent Features

Concurrent rendering is **opt-in** through specific APIs:

- **useTransition** - Mark updates as non-urgent
- **useDeferredValue** - Defer updating a value
- **Suspense** - Declarative loading states
- **startTransition** - Standalone transition API

```jsx
import { useTransition, useDeferredValue, Suspense } from 'react';

// Using concurrent features
function App() {
  // Opt-in to concurrent rendering
  const [isPending, startTransition] = useTransition();
  const deferredValue = useDeferredValue(value);
  
  return (
    <Suspense fallback={<Loading />}>
      {/* Component */}
    </Suspense>
  );
}
```

---

## Real-World Examples

### Example 1: Responsive Search

**Problem:** Search with large dataset freezes UI while filtering.

**Without Concurrent Rendering:**
```jsx
function Search() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  
  const handleChange = (e) => {
    const value = e.target.value;
    setQuery(value);
    
    // Blocks UI for 500ms while filtering
    const filtered = largeDataset.filter(item => 
      item.name.includes(value)
    );
    setResults(filtered);
  };
  
  return (
    <div>
      <input value={query} onChange={handleChange} />  {/* Laggy! */}
      <ul>{results.map(item => <li key={item.id}>{item.name}</li>)}</ul>
    </div>
  );
}
```

**With Concurrent Rendering:**
```jsx
import { useState, useTransition } from 'react';

function Search() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();
  
  const handleChange = (e) => {
    const value = e.target.value;
    
    // Urgent: Input stays responsive
    setQuery(value);
    
    // Non-urgent: Filtering can be interrupted
    startTransition(() => {
      const filtered = largeDataset.filter(item => 
        item.name.includes(value)
      );
      setResults(filtered);
    });
  };
  
  return (
    <div>
      <input value={query} onChange={handleChange} />  {/* Smooth! */}
      {isPending && <div className="spinner">Filtering...</div>}
      <ul>{results.map(item => <li key={item.id}>{item.name}</li>)}</ul>
    </div>
  );
}
```

**Result:** Input stays responsive even while filtering 10,000+ items.

---

### Example 2: Tab Switching

**Without Concurrent Rendering:**
```jsx
function Tabs() {
  const [tab, setTab] = useState('home');
  
  return (
    <div>
      <button onClick={() => setTab('home')}>Home</button>
      <button onClick={() => setTab('posts')}>Posts (slow)</button>
      <button onClick={() => setTab('profile')}>Profile</button>
      
      {/* Switching to posts freezes for 1 second */}
      {tab === 'home' && <Home />}
      {tab === 'posts' && <SlowPostsList />}  {/* 1 second to render */}
      {tab === 'profile' && <Profile />}
    </div>
  );
}
```

**Problem:** Clicking "Posts" tab freezes UI for 1 second.

**With Concurrent Rendering:**
```jsx
import { useState, useTransition } from 'react';

function Tabs() {
  const [tab, setTab] = useState('home');
  const [isPending, startTransition] = useTransition();
  
  const switchTab = (newTab) => {
    startTransition(() => {
      setTab(newTab);
    });
  };
  
  return (
    <div>
      <button onClick={() => switchTab('home')}>Home</button>
      <button onClick={() => switchTab('posts')} disabled={isPending}>
        Posts {isPending && '(loading...)'}
      </button>
      <button onClick={() => switchTab('profile')}>Profile</button>
      
      {/* Old tab stays visible while new tab loads */}
      {tab === 'home' && <Home />}
      {tab === 'posts' && <SlowPostsList />}
      {tab === 'profile' && <Profile />}
    </div>
  );
}
```

**Result:** UI stays responsive. Old tab visible while new tab renders.

---

### Example 3: Live Filtering

```jsx
import { useState, useDeferredValue } from 'react';

function ProductList({ products }) {
  const [filter, setFilter] = useState('');
  
  // Defer updating the filter for rendering
  const deferredFilter = useDeferredValue(filter);
  
  // Use deferred value for expensive computation
  const filteredProducts = products.filter(p => 
    p.name.toLowerCase().includes(deferredFilter.toLowerCase())
  );
  
  return (
    <div>
      {/* Input uses current value (responsive) */}
      <input 
        value={filter}
        onChange={e => setFilter(e.target.value)}
        placeholder="Filter products..."
      />
      
      {/* List uses deferred value (can lag behind) */}
      <ul>
        {filteredProducts.map(p => <li key={p.id}>{p.name}</li>)}
      </ul>
    </div>
  );
}
```

**Benefit:** Input updates immediately. List updates when React has time.

---

## How It Works Internally

### Fiber Architecture

React uses **Fiber** architecture (since React 16) which enables concurrent rendering:

1. **Work is divided into units** (fibers)
2. **Each unit can be paused/resumed**
3. **Priority assigned to each update**
4. **React schedules work accordingly**

```
Old (Stack Reconciler):
├── Render entire tree
└── Cannot be interrupted

New (Fiber Reconciler):
├── Component A (5ms) ← Can pause here
├── Component B (5ms) ← Can pause here
├── Component C (5ms) ← Can pause here
└── Continue or restart based on priority
```

---

## Comparison: Before vs After

### Before React 18

```jsx
function App() {
  const [text, setText] = useState('');
  const [items, setItems] = useState([]);
  
  const handleChange = (e) => {
    const value = e.target.value;
    setText(value);
    
    // Both updates have same priority
    // UI freezes until both complete
    const newItems = Array(20000)
      .fill(null)
      .map((_, i) => `${value} ${i}`);
    setItems(newItems);
  };
  
  return (
    <div>
      <input value={text} onChange={handleChange} />  {/* Laggy */}
      <List items={items} />
    </div>
  );
}
```

**Problem:**
- Both updates have same priority
- Rendering 20,000 items blocks input
- UI feels frozen

---

### After React 18

```jsx
import { useState, useTransition } from 'react';

function App() {
  const [text, setText] = useState('');
  const [items, setItems] = useState([]);
  const [isPending, startTransition] = useTransition();
  
  const handleChange = (e) => {
    const value = e.target.value;
    
    // High priority: immediate
    setText(value);
    
    // Low priority: can be interrupted
    startTransition(() => {
      const newItems = Array(20000)
        .fill(null)
        .map((_, i) => `${value} ${i}`);
      setItems(newItems);
    });
  };
  
  return (
    <div>
      <input value={text} onChange={handleChange} />  {/* Smooth! */}
      {isPending && <Spinner />}
      <List items={items} />
    </div>
  );
}
```

**Benefits:**
- Input updates have high priority (immediate)
- List updates have low priority (interruptible)
- UI stays responsive
- Better user experience

---

## Concurrent Features APIs

### 1. useTransition

```jsx
const [isPending, startTransition] = useTransition();

startTransition(() => {
  // Mark this state update as non-urgent
  setState(newValue);
});
```

---

### 2. useDeferredValue

```jsx
const deferredValue = useDeferredValue(value);
// Use deferredValue for expensive renders
```

---

### 3. startTransition (standalone)

```jsx
import { startTransition } from 'react';

startTransition(() => {
  setState(newValue);
});
```

---

### 4. Suspense (enhanced)

```jsx
<Suspense fallback={<Loading />}>
  <AsyncComponent />
</Suspense>
```

---

## Benefits

### 1. Responsive UI

✅ Input fields never freeze  
✅ Animations stay smooth  
✅ UI always responds to user  

---

### 2. Better User Experience

✅ Keep showing old content while loading new  
✅ Show loading indicators for slow updates  
✅ Prioritize what users see first  

---

### 3. Automatic Optimization

✅ React handles prioritization  
✅ No manual debouncing needed (for most cases)  
✅ Works automatically with concurrent APIs  

---

## Common Misconceptions

### ❌ Misconception 1: "Concurrent rendering makes everything faster"

**Reality:** It makes UI more **responsive**, not necessarily faster.
- Total work might be same or slightly more
- But UI doesn't freeze during work
- Better perceived performance

---

### ❌ Misconception 2: "All components automatically use concurrent rendering"

**Reality:** It's **opt-in** through specific APIs.
- Must use `createRoot` (not `ReactDOM.render`)
- Must use concurrent features (useTransition, useDeferredValue, etc.)
- Regular state updates still synchronous

---

### ❌ Misconception 3: "It uses multiple threads"

**Reality:** Still single-threaded, but **interruptible**.
- JavaScript is single-threaded
- React can pause/resume work
- Yields control to browser between chunks

---

## Migration to React 18

### Step 1: Update React

```bash
npm install react@18 react-dom@18
```

---

### Step 2: Use createRoot

**Before:**
```jsx
import ReactDOM from 'react-dom';

ReactDOM.render(<App />, document.getElementById('root'));
```

**After:**
```jsx
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

---

### Step 3: Adopt Concurrent Features Gradually

```jsx
// Gradually add concurrent features where beneficial
import { useTransition } from 'react';

function Component() {
  const [isPending, startTransition] = useTransition();
  
  // Wrap non-urgent updates
  startTransition(() => {
    setLargeList(newData);
  });
}
```

---

## When to Use Concurrent Features

### ✅ Good Use Cases:

1. **Search/filtering with large datasets**
2. **Complex UI transitions**
3. **Tab switching**
4. **Route navigation**
5. **Lazy loading content**
6. **Real-time updates that don't need immediate display**

### ❌ Not Needed For:

1. **Simple state updates**
2. **Already fast renders**
3. **Critical user input** (keep high priority)
4. **Small datasets**

---

## Performance Comparison

```jsx
// Benchmark: Filtering 10,000 items while typing

// Without concurrent rendering:
// Input delay: 200-500ms per keystroke ❌
// UI freezes during render ❌

// With concurrent rendering:
// Input delay: <16ms per keystroke ✅
// UI stays responsive ✅
// List updates slightly delayed but smooth ✅
```

---

**Interview Tips:**
- **Concurrent rendering** = React 18's ability to interrupt rendering
- **Interruptible** = can pause/resume work
- **Priority-based** = urgent updates processed first
- **Time slicing** = work divided into small chunks
- **Opt-in** = use createRoot + concurrent features
- **useTransition** = mark updates as non-urgent
- **useDeferredValue** = defer value updates
- **Fiber architecture** = enables concurrent rendering
- **Not multi-threaded** = still single-threaded JavaScript
- **Responsive, not faster** = UI doesn't freeze
- **Keep old UI** = while rendering new UI
- **Automatic batching** = multiple setState batched together
- **Suspense** = works better with concurrent rendering
- **createRoot required** = must migrate from ReactDOM.render
- **Backward compatible** = existing code still works
- **Gradual adoption** = add concurrent features incrementally
- **Better UX** = smoother, more responsive interface
- **High vs low priority** = input urgent, lists non-urgent
- **No debouncing needed** = for many cases (React handles it)
- **Paused renders** = can be abandoned if stale
- **Multiple renders in progress** = at different priorities
- **Browser idle time** = React uses efficiently
- **New mental model** = think in terms of priorities
- **React 18+ only** = not available in React 17 or earlier
- **Game changer** = fundamentally different rendering approach

</details>

---

### 84. What is useTransition hook and when do you use it?

<details>
<summary>View Answer</summary>

**useTransition Hook**

A React 18 hook that lets you mark state updates as non-urgent (transitions), keeping your UI responsive during expensive renders.

---

## What is useTransition?

**useTransition** allows you to update state without blocking the UI.

### Syntax

```jsx
import { useTransition } from 'react';

function Component() {
  const [isPending, startTransition] = useTransition();
  
  // isPending: boolean - true while transition is in progress
  // startTransition: function - wraps non-urgent state updates
}
```

---

## Basic Example

### Without useTransition (Blocking)

```jsx
import { useState } from 'react';

function SlowList() {
  const [input, setInput] = useState('');
  const [list, setList] = useState([]);
  
  const handleChange = (e) => {
    const value = e.target.value;
    setInput(value);
    
    // Generates 20,000 items - BLOCKS UI for ~200ms
    const newList = Array(20000)
      .fill(null)
      .map((_, i) => `${value} - Item ${i}`);
    setList(newList);
  };
  
  return (
    <div>
      <input 
        value={input} 
        onChange={handleChange} 
        placeholder="Type here..."  {/* LAGGY! */}
      />
      <ul>
        {list.map((item, i) => <li key={i}>{item}</li>)}
      </ul>
    </div>
  );
}
```

**Problem:** Input is laggy. Every keystroke freezes UI for 200ms.

---

### With useTransition (Non-blocking)

```jsx
import { useState, useTransition } from 'react';

function SlowList() {
  const [input, setInput] = useState('');
  const [list, setList] = useState([]);
  const [isPending, startTransition] = useTransition();
  
  const handleChange = (e) => {
    const value = e.target.value;
    
    // URGENT: Update input immediately (high priority)
    setInput(value);
    
    // NON-URGENT: Update list (low priority, can be interrupted)
    startTransition(() => {
      const newList = Array(20000)
        .fill(null)
        .map((_, i) => `${value} - Item ${i}`);
      setList(newList);
    });
  };
  
  return (
    <div>
      <input 
        value={input} 
        onChange={handleChange} 
        placeholder="Type here..."  {/* SMOOTH! */}
      />
      
      {/* Show loading indicator while list is updating */}
      {isPending && <div className="loading">Updating list...</div>}
      
      <ul>
        {list.map((item, i) => <li key={i}>{item}</li>)}
      </ul>
    </div>
  );
}
```

**Result:**
- ✅ Input is smooth and responsive
- ✅ List updates without blocking UI
- ✅ Loading indicator shows while updating
- ✅ If user keeps typing, React abandons old renders

---

## When to Use useTransition

### 1. Search/Filtering

```jsx
import { useState, useTransition } from 'react';

function SearchProducts({ products }) {
  const [query, setQuery] = useState('');
  const [filteredProducts, setFilteredProducts] = useState(products);
  const [isPending, startTransition] = useTransition();
  
  const handleSearch = (e) => {
    const value = e.target.value;
    
    // Update input immediately
    setQuery(value);
    
    // Filter products (non-urgent)
    startTransition(() => {
      const filtered = products.filter(p => 
        p.name.toLowerCase().includes(value.toLowerCase())
      );
      setFilteredProducts(filtered);
    });
  };
  
  return (
    <div>
      <input 
        value={query}
        onChange={handleSearch}
        placeholder="Search products..."
      />
      
      {isPending ? (
        <div className="spinner">Searching...</div>
      ) : (
        <div className="results">{filteredProducts.length} results</div>
      )}
      
      <ul>
        {filteredProducts.map(p => (
          <li key={p.id}>{p.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

### 2. Tab Switching

```jsx
import { useState, useTransition } from 'react';

function Tabs() {
  const [activeTab, setActiveTab] = useState('home');
  const [isPending, startTransition] = useTransition();
  
  const switchTab = (tab) => {
    startTransition(() => {
      setActiveTab(tab);
    });
  };
  
  return (
    <div>
      <div className="tabs">
        <button 
          onClick={() => switchTab('home')}
          className={activeTab === 'home' ? 'active' : ''}
        >
          Home
        </button>
        
        <button 
          onClick={() => switchTab('posts')}
          className={activeTab === 'posts' ? 'active' : ''}
        >
          Posts {isPending && '...'}
        </button>
        
        <button 
          onClick={() => switchTab('profile')}
          className={activeTab === 'profile' ? 'active' : ''}
        >
          Profile
        </button>
      </div>
      
      {/* Keep showing old tab while new tab loads */}
      <div className="content">
        {activeTab === 'home' && <HomeTab />}
        {activeTab === 'posts' && <PostsTab />}  {/* Slow to render */}
        {activeTab === 'profile' && <ProfileTab />}
      </div>
    </div>
  );
}
```

**Benefit:** Old tab stays visible while new tab is rendering. No blank screen.

---

### 3. Navigation

```jsx
import { useTransition } from 'react';
import { useNavigate } from 'react-router-dom';

function ProductCard({ product }) {
  const navigate = useNavigate();
  const [isPending, startTransition] = useTransition();
  
  const handleClick = () => {
    startTransition(() => {
      navigate(`/products/${product.id}`);
    });
  };
  
  return (
    <div onClick={handleClick} className="card">
      <img src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      {isPending && <div className="spinner">Loading...</div>}
    </div>
  );
}
```

---

### 4. Sorting/Filtering Large Lists

```jsx
import { useState, useTransition } from 'react';

function DataTable({ data }) {
  const [sortedData, setSortedData] = useState(data);
  const [isPending, startTransition] = useTransition();
  
  const handleSort = (field) => {
    startTransition(() => {
      const sorted = [...data].sort((a, b) => {
        return a[field] > b[field] ? 1 : -1;
      });
      setSortedData(sorted);
    });
  };
  
  return (
    <div>
      <button onClick={() => handleSort('name')} disabled={isPending}>
        Sort by Name {isPending && '...'}
      </button>
      <button onClick={() => handleSort('price')} disabled={isPending}>
        Sort by Price {isPending && '...'}
      </button>
      
      <table>
        <tbody>
          {sortedData.map(item => (
            <tr key={item.id}>
              <td>{item.name}</td>
              <td>${item.price}</td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

---

## isPending Flag

The `isPending` boolean indicates when transition is in progress.

### Show Loading Indicators

```jsx
function Component() {
  const [isPending, startTransition] = useTransition();
  
  return (
    <div>
      {isPending && (
        <div className="loading-overlay">
          <Spinner />
          <p>Loading...</p>
        </div>
      )}
      
      {/* Content */}
    </div>
  );
}
```

---

### Disable Buttons

```jsx
function Component() {
  const [isPending, startTransition] = useTransition();
  
  return (
    <button 
      onClick={() => startTransition(() => doSomething())}
      disabled={isPending}
    >
      {isPending ? 'Loading...' : 'Submit'}
    </button>
  );
}
```

---

### Show Progress

```jsx
function Component() {
  const [isPending, startTransition] = useTransition();
  
  return (
    <div>
      {isPending && <ProgressBar />}
      <button onClick={() => startTransition(() => doSomething())}>
        Click me
      </button>
    </div>
  );
}
```

---

## Multiple Transitions

```jsx
import { useState, useTransition } from 'react';

function MultipleTransitions() {
  const [tab, setTab] = useState('home');
  const [filter, setFilter] = useState('');
  const [isPending, startTransition] = useTransition();
  
  return (
    <div>
      {/* Both transitions use same isPending */}
      <button onClick={() => startTransition(() => setTab('posts'))}>
        Posts {isPending && '...'}
      </button>
      
      <input 
        value={filter}
        onChange={e => {
          setFilter(e.target.value);
          startTransition(() => applyFilter(e.target.value));
        }}
      />
      
      {isPending && <Spinner />}
    </div>
  );
}
```

---

## useTransition vs startTransition

### useTransition (Hook)

```jsx
import { useTransition } from 'react';

function Component() {
  const [isPending, startTransition] = useTransition();
  
  return (
    <div>
      {isPending && <Spinner />}  {/* Can track pending state */}
      <button onClick={() => startTransition(() => doSomething())}>
        Click
      </button>
    </div>
  );
}
```

**Use when:** You need to show loading indicator.

---

### startTransition (Standalone)

```jsx
import { startTransition } from 'react';

function Component() {
  const handleClick = () => {
    startTransition(() => {
      doSomething();
    });
  };
  
  return <button onClick={handleClick}>Click</button>;
}
```

**Use when:** You don't need loading indicator.

---

## Comparison Table

| Feature | useTransition | startTransition |
|---------|---------------|------------------|
| **Type** | Hook | Function |
| **Returns** | [isPending, startTransition] | Nothing |
| **isPending** | ✅ Yes | ❌ No |
| **Use case** | Need loading indicator | Don't need indicator |
| **Where** | Inside component | Anywhere |
| **Example** | Search with spinner | Simple navigation |

---

## Real-World Example: Dashboard Filters

```jsx
import { useState, useTransition } from 'react';

function Dashboard({ data }) {
  const [dateRange, setDateRange] = useState('week');
  const [category, setCategory] = useState('all');
  const [filteredData, setFilteredData] = useState(data);
  const [isPending, startTransition] = useTransition();
  
  const applyFilters = (newDateRange, newCategory) => {
    startTransition(() => {
      // Expensive filtering operation
      const filtered = data.filter(item => {
        const dateMatch = item.date.match(newDateRange);
        const categoryMatch = newCategory === 'all' || item.category === newCategory;
        return dateMatch && categoryMatch;
      });
      
      setFilteredData(filtered);
    });
  };
  
  const handleDateChange = (range) => {
    setDateRange(range);  // Update immediately
    applyFilters(range, category);  // Non-urgent
  };
  
  const handleCategoryChange = (cat) => {
    setCategory(cat);  // Update immediately
    applyFilters(dateRange, cat);  // Non-urgent
  };
  
  return (
    <div className="dashboard">
      <div className="filters">
        <select value={dateRange} onChange={e => handleDateChange(e.target.value)}>
          <option value="day">Today</option>
          <option value="week">This Week</option>
          <option value="month">This Month</option>
        </select>
        
        <select value={category} onChange={e => handleCategoryChange(e.target.value)}>
          <option value="all">All Categories</option>
          <option value="sales">Sales</option>
          <option value="marketing">Marketing</option>
          <option value="support">Support</option>
        </select>
        
        {isPending && <Spinner />}
      </div>
      
      <div className="data-grid">
        {filteredData.map(item => (
          <DataCard key={item.id} data={item} />
        ))}
      </div>
    </div>
  );
}
```

---

## Advanced: Custom Hook

```jsx
import { useState, useTransition } from 'react';

function useTransitionState(initialValue) {
  const [state, setState] = useState(initialValue);
  const [isPending, startTransition] = useTransition();
  
  const setStateTransition = (value) => {
    startTransition(() => {
      setState(value);
    });
  };
  
  return [state, setStateTransition, isPending];
}

// Usage
function Component() {
  const [items, setItems, isPending] = useTransitionState([]);
  
  return (
    <div>
      <button onClick={() => setItems(generateLargeList())}>
        Load Items
      </button>
      {isPending && <Spinner />}
      <ul>{items.map(item => <li key={item}>{item}</li>)}</ul>
    </div>
  );
}
```

---

## What NOT to Use It For

### ❌ Don't wrap user input updates

```jsx
// ❌ WRONG - Input will be laggy
const handleChange = (e) => {
  startTransition(() => {
    setValue(e.target.value);  // Don't defer input updates!
  });
};
```

```jsx
// ✅ CORRECT - Input is immediate
const handleChange = (e) => {
  const value = e.target.value;
  setValue(value);  // Immediate
  
  startTransition(() => {
    setDerivedValue(expensiveComputation(value));  // Deferred
  });
};
```

---

### ❌ Don't use for critical updates

```jsx
// ❌ WRONG - Payment confirmation should be immediate
const handlePay = () => {
  startTransition(() => {
    processPayment();  // Don't defer critical actions!
  });
};
```

```jsx
// ✅ CORRECT - Critical actions are immediate
const handlePay = async () => {
  await processPayment();  // Immediate
  
  startTransition(() => {
    updateAnalytics();  // Non-critical, can defer
  });
};
```

---

## Best Practices

**1. Use for expensive non-critical updates**
```jsx
startTransition(() => {
  setLargeList(expensiveComputation());
});
```

**2. Keep user input responsive**
```jsx
setInput(value);  // Immediate
startTransition(() => setResults(filter(value)));  // Deferred
```

**3. Show loading indicators**
```jsx
{isPending && <Spinner />}
```

**4. Disable actions during transitions**
```jsx
<button disabled={isPending}>Submit</button>
```

**5. Use for navigation**
```jsx
startTransition(() => {
  navigate('/new-page');
});
```

---

## Performance Impact

### Benchmark: Filtering 10,000 Items

**Without useTransition:**
```
Keystroke 1: 200ms (frozen)
Keystroke 2: 200ms (frozen)
Keystroke 3: 200ms (frozen)
Total: 600ms of frozen UI
```

**With useTransition:**
```
Keystroke 1: <16ms (smooth)
Keystroke 2: <16ms (smooth)
Keystroke 3: <16ms (smooth)
Total: <50ms perceived delay
```

**Result:** 12x better perceived performance!

---

## Browser Support

✅ Works in all modern browsers that support React 18  
✅ No additional polyfills needed  
✅ Gracefully degrades in older React versions (just works synchronously)  

---

**Interview Tips:**
- **useTransition** = mark updates as non-urgent
- Returns **[isPending, startTransition]**
- **isPending** = true while transition in progress
- **startTransition** = function to wrap non-urgent updates
- **Use for**: search, filtering, navigation, tab switching
- **Don't use for**: user input, critical updates
- **Keeps UI responsive** during expensive renders
- **Can be interrupted** if higher priority update comes
- **Old UI stays visible** while new UI renders
- **Show loading indicators** with isPending
- **Disable buttons** during transition
- **Multiple transitions** share same isPending
- **vs startTransition**: useTransition provides isPending
- **React 18+ only** feature
- **Requires createRoot** to work
- **Concurrent rendering** must be enabled
- **No performance cost** if not used
- **Gradual adoption** - add where beneficial
- **Better than debouncing** for many cases
- **Automatic optimization** by React
- **Part of Concurrent React** features
- **Works with Suspense** for data fetching
- **Abandoned renders** if user keeps interacting
- **Priority-based** scheduling
- **Time slicing** enables interruptibility

</details>

---

### 85. What is useDeferredValue hook?

<details>
<summary>View Answer</summary>

**useDeferredValue Hook**

A React 18 hook that lets you defer updating a value to keep the UI responsive during expensive renders.

---

## What is useDeferredValue?

**useDeferredValue** accepts a value and returns a deferred version that may "lag behind" the original during expensive updates.

### Syntax

```jsx
import { useDeferredValue } from 'react';

function Component() {
  const deferredValue = useDeferredValue(value);
  
  // value: current value (always up-to-date)
  // deferredValue: may lag behind during expensive renders
}
```

---

## Basic Example

### Without useDeferredValue (Blocking)

```jsx
import { useState } from 'react';

function SearchPage() {
  const [query, setQuery] = useState('');
  
  // Expensive filtering blocks input
  const results = products.filter(p => 
    p.name.toLowerCase().includes(query.toLowerCase())
  );
  
  return (
    <div>
      <input 
        value={query}
        onChange={e => setQuery(e.target.value)}
        placeholder="Search..."  {/* LAGGY! */}
      />
      
      <ul>
        {results.map(p => <li key={p.id}>{p.name}</li>)}
      </ul>
    </div>
  );
}
```

**Problem:** Input freezes while filtering large list.

---

### With useDeferredValue (Non-blocking)

```jsx
import { useState, useDeferredValue } from 'react';

function SearchPage() {
  const [query, setQuery] = useState('');
  
  // Defer the query value for expensive operations
  const deferredQuery = useDeferredValue(query);
  
  // Use deferred value for filtering
  const results = products.filter(p => 
    p.name.toLowerCase().includes(deferredQuery.toLowerCase())
  );
  
  return (
    <div>
      <input 
        value={query}  {/* Always current */}
        onChange={e => setQuery(e.target.value)}
        placeholder="Search..."  {/* SMOOTH! */}
      />
      
      {/* Show indicator when values differ */}
      {query !== deferredQuery && <div className="loading">Updating...</div>}
      
      <ul>
        {results.map(p => <li key={p.id}>{p.name}</li>)}
      </ul>
    </div>
  );
}
```

**Result:**
- ✅ Input stays responsive
- ✅ Results update without blocking
- ✅ Can detect when deferred value is "stale"

---

## How It Works

### Timeline

```
User types "a":
  query = "a" (immediate)
  deferredQuery = "" (still old)
  → Input shows "a"
  → List still shows old results
  → React renders new list in background

After render completes:
  deferredQuery = "a" (caught up)
  → List shows filtered results

User types "ab" (before previous render finishes):
  query = "ab" (immediate)
  deferredQuery = "" (still old)
  → React abandons previous render
  → Starts new render with "ab"
```

---

## When to Use useDeferredValue

### 1. Search/Filtering

```jsx
import { useState, useDeferredValue } from 'react';

function ProductSearch({ products }) {
  const [search, setSearch] = useState('');
  const deferredSearch = useDeferredValue(search);
  
  const filteredProducts = products.filter(product => 
    product.name.toLowerCase().includes(deferredSearch.toLowerCase())
  );
  
  return (
    <div>
      <input
        value={search}
        onChange={e => setSearch(e.target.value)}
        placeholder="Search products..."
      />
      
      {search !== deferredSearch && <Spinner />}
      
      <div className="results">
        {filteredProducts.map(p => (
          <ProductCard key={p.id} product={p} />
        ))}
      </div>
    </div>
  );
}
```

---

### 2. Slow Components

```jsx
import { useState, useDeferredValue, memo } from 'react';

// Expensive component
const SlowList = memo(({ items }) => {
  // Simulate expensive rendering
  const startTime = performance.now();
  while (performance.now() - startTime < 100) {
    // Block for 100ms
  }
  
  return (
    <ul>
      {items.map(item => <li key={item}>{item}</li>)}
    </ul>
  );
});

function App() {
  const [text, setText] = useState('');
  const deferredText = useDeferredValue(text);
  
  return (
    <div>
      <input 
        value={text}
        onChange={e => setText(e.target.value)}
      />
      
      {/* Pass deferred value to slow component */}
      <SlowList items={[deferredText]} />
    </div>
  );
}
```

---

### 3. Live Preview

```jsx
import { useState, useDeferredValue } from 'react';
import { marked } from 'marked';

function MarkdownEditor() {
  const [markdown, setMarkdown] = useState('');
  const deferredMarkdown = useDeferredValue(markdown);
  
  // Expensive: Convert markdown to HTML
  const html = marked(deferredMarkdown);
  
  return (
    <div className="editor">
      <div className="input-pane">
        <textarea
          value={markdown}  {/* Current value */}
          onChange={e => setMarkdown(e.target.value)}
          placeholder="Write markdown..."
        />
      </div>
      
      <div className="preview-pane">
        {markdown !== deferredMarkdown && <div>Updating preview...</div>}
        <div dangerouslySetInnerHTML={{ __html: html }} />
      </div>
    </div>
  );
}
```

---

### 4. Chart Updates

```jsx
import { useState, useDeferredValue } from 'react';
import { LineChart } from 'recharts';

function ChartDashboard({ data }) {
  const [dateRange, setDateRange] = useState('week');
  const deferredDateRange = useDeferredValue(dateRange);
  
  // Expensive: Filter and process data
  const chartData = processChartData(data, deferredDateRange);
  
  return (
    <div>
      <select 
        value={dateRange}
        onChange={e => setDateRange(e.target.value)}
      >
        <option value="day">Today</option>
        <option value="week">This Week</option>
        <option value="month">This Month</option>
      </select>
      
      {dateRange !== deferredDateRange && <div>Loading chart...</div>}
      
      <LineChart data={chartData} />
    </div>
  );
}
```

---

## Detecting Stale Values

Check if deferred value has caught up:

```jsx
import { useState, useDeferredValue } from 'react';

function Component() {
  const [value, setValue] = useState('');
  const deferredValue = useDeferredValue(value);
  
  // Check if deferred value is stale
  const isStale = value !== deferredValue;
  
  return (
    <div>
      <input value={value} onChange={e => setValue(e.target.value)} />
      
      {isStale && (
        <div className="stale-indicator">
          <Spinner />
          <span>Updating...</span>
        </div>
      )}
      
      <Content value={deferredValue} />
    </div>
  );
}
```

---

## useDeferredValue vs useTransition

### useDeferredValue

```jsx
import { useState, useDeferredValue } from 'react';

function Component() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  
  return (
    <div>
      <input value={query} onChange={e => setQuery(e.target.value)} />
      <Results query={deferredQuery} />
    </div>
  );
}
```

**Use when:** You have a value that you want to defer for expensive renders.

---

### useTransition

```jsx
import { useState, useTransition } from 'react';

function Component() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();
  
  const handleChange = (e) => {
    const value = e.target.value;
    setQuery(value);
    
    startTransition(() => {
      const filtered = expensiveFilter(value);
      setResults(filtered);
    });
  };
  
  return (
    <div>
      <input value={query} onChange={handleChange} />
      {isPending && <Spinner />}
      <Results data={results} />
    </div>
  );
}
```

**Use when:** You control the state update and want to mark it as non-urgent.

---

## Comparison Table

| Feature | useDeferredValue | useTransition |
|---------|------------------|---------------|
| **What it defers** | A value | State updates |
| **Returns** | Deferred value | [isPending, startTransition] |
| **Control** | Automatic | Manual |
| **Use case** | Defer a prop/value | Defer state update |
| **isPending** | Compare values | Built-in flag |
| **When to use** | Can't control update | Control the update |
| **Example** | Filtering with prop | Filtering with state |

---

## When to Use Which?

### Use useDeferredValue when:

✅ Value comes from props  
✅ Can't wrap setState in transition  
✅ Want simpler API  
✅ Don't need explicit isPending  

```jsx
function ChildComponent({ searchQuery }) {  // From props
  const deferredQuery = useDeferredValue(searchQuery);
  // Use deferredQuery for expensive operations
}
```

---

### Use useTransition when:

✅ Control the state update  
✅ Need isPending flag  
✅ Multiple state updates to defer  
✅ Want explicit control  

```jsx
function ParentComponent() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();
  
  const handleChange = (e) => {
    setQuery(e.target.value);
    startTransition(() => {
      setResults(expensiveFilter(e.target.value));
    });
  };
}
```

---

## Real-World Example: Data Table

```jsx
import { useState, useDeferredValue, memo } from 'react';

const DataTable = memo(({ data, searchTerm }) => {
  const filtered = data.filter(row => {
    return Object.values(row).some(value => 
      String(value).toLowerCase().includes(searchTerm.toLowerCase())
    );
  });
  
  return (
    <table>
      <thead>
        <tr>
          <th>ID</th>
          <th>Name</th>
          <th>Email</th>
          <th>Status</th>
        </tr>
      </thead>
      <tbody>
        {filtered.map(row => (
          <tr key={row.id}>
            <td>{row.id}</td>
            <td>{row.name}</td>
            <td>{row.email}</td>
            <td>{row.status}</td>
          </tr>
        ))}
      </tbody>
    </table>
  );
});

function Dashboard({ data }) {
  const [searchTerm, setSearchTerm] = useState('');
  const deferredSearchTerm = useDeferredValue(searchTerm);
  
  const isSearching = searchTerm !== deferredSearchTerm;
  
  return (
    <div className="dashboard">
      <div className="search-bar">
        <input
          type="search"
          value={searchTerm}
          onChange={e => setSearchTerm(e.target.value)}
          placeholder="Search all columns..."
        />
        {isSearching && (
          <div className="searching-indicator">
            <Spinner size="small" />
            <span>Searching...</span>
          </div>
        )}
      </div>
      
      <div className="results-count">
        {!isSearching && (
          <span>{data.length} records</span>
        )}
      </div>
      
      <DataTable data={data} searchTerm={deferredSearchTerm} />
    </div>
  );
}
```

---

## With memo for Optimization

Combine with `memo` to prevent unnecessary re-renders:

```jsx
import { useState, useDeferredValue, memo } from 'react';

// Memoized component only re-renders when items change
const List = memo(({ items }) => {
  console.log('List rendered with', items.length, 'items');
  
  return (
    <ul>
      {items.map(item => <li key={item.id}>{item.name}</li>)}
    </ul>
  );
});

function App() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  
  // Expensive filtering
  const filteredItems = items.filter(item => 
    item.name.includes(deferredQuery)
  );
  
  return (
    <div>
      <input value={query} onChange={e => setQuery(e.target.value)} />
      
      {/* List only re-renders when filteredItems changes */}
      {/* Not when query changes */}
      <List items={filteredItems} />
    </div>
  );
}
```

---

## Multiple Deferred Values

```jsx
import { useState, useDeferredValue } from 'react';

function ComplexSearch({ products }) {
  const [name, setName] = useState('');
  const [category, setCategory] = useState('');
  const [price, setPrice] = useState('');
  
  // Defer all filter values
  const deferredName = useDeferredValue(name);
  const deferredCategory = useDeferredValue(category);
  const deferredPrice = useDeferredValue(price);
  
  // Use deferred values for filtering
  const filtered = products.filter(p => {
    const nameMatch = p.name.includes(deferredName);
    const categoryMatch = !deferredCategory || p.category === deferredCategory;
    const priceMatch = !deferredPrice || p.price <= deferredPrice;
    return nameMatch && categoryMatch && priceMatch;
  });
  
  const isFiltering = 
    name !== deferredName ||
    category !== deferredCategory ||
    price !== deferredPrice;
  
  return (
    <div>
      <input value={name} onChange={e => setName(e.target.value)} placeholder="Name" />
      <select value={category} onChange={e => setCategory(e.target.value)}>
        <option value="">All Categories</option>
        <option value="electronics">Electronics</option>
        <option value="clothing">Clothing</option>
      </select>
      <input value={price} onChange={e => setPrice(e.target.value)} placeholder="Max Price" />
      
      {isFiltering && <div>Filtering...</div>}
      
      <ul>
        {filtered.map(p => <li key={p.id}>{p.name}</li>)}
      </ul>
    </div>
  );
}
```

---

## Advanced: Custom Hook

```jsx
import { useDeferredValue } from 'react';

function useDeferredFilter(items, query) {
  const deferredQuery = useDeferredValue(query);
  
  const filteredItems = items.filter(item => 
    item.name.toLowerCase().includes(deferredQuery.toLowerCase())
  );
  
  const isFiltering = query !== deferredQuery;
  
  return { filteredItems, isFiltering };
}

// Usage
function SearchComponent({ items }) {
  const [query, setQuery] = useState('');
  const { filteredItems, isFiltering } = useDeferredFilter(items, query);
  
  return (
    <div>
      <input value={query} onChange={e => setQuery(e.target.value)} />
      {isFiltering && <Spinner />}
      <List items={filteredItems} />
    </div>
  );
}
```

---

## Performance Considerations

### Before (No Deferral)

```
User types "a":
  → Input value = "a"
  → Filter 10,000 items (200ms)
  → UI frozen for 200ms ❌
  → Results displayed

User types "b":
  → Wait for previous render to finish
  → Input value = "ab"
  → Filter 10,000 items (200ms)
  → UI frozen for 200ms ❌
```

---

### After (With useDeferredValue)

```
User types "a":
  → Input value = "a" (immediate) ✅
  → Start filtering in background
  → UI stays responsive ✅

User types "b" (before filtering completes):
  → Input value = "ab" (immediate) ✅
  → Abandon previous filter
  → Start new filter with "ab"
  → UI stays responsive ✅
```

---

## Common Pitfalls

### Pitfall 1: Not memoizing expensive component

```jsx
// ❌ Component re-renders on every query change
function List({ items }) {
  return <ul>{items.map(i => <li key={i}>{i}</li>)}</ul>;
}

function App() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  
  return (
    <div>
      <input value={query} onChange={e => setQuery(e.target.value)} />
      <List items={[deferredQuery]} />  {/* Re-renders on query change */}
    </div>
  );
}
```

```jsx
// ✅ Memoized component only re-renders when items change
const List = memo(({ items }) => {
  return <ul>{items.map(i => <li key={i}>{i}</li>)}</ul>;
});

function App() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  
  return (
    <div>
      <input value={query} onChange={e => setQuery(e.target.value)} />
      <List items={[deferredQuery]} />  {/* Only re-renders when deferredQuery changes */}
    </div>
  );
}
```

---

### Pitfall 2: Using for critical updates

```jsx
// ❌ Don't defer critical updates
function Payment() {
  const [amount, setAmount] = useState(0);
  const deferredAmount = useDeferredValue(amount);  // ❌ Bad!
  
  return <button onClick={() => processPayment(deferredAmount)}>Pay</button>;
}
```

```jsx
// ✅ Critical updates should be immediate
function Payment() {
  const [amount, setAmount] = useState(0);
  
  return <button onClick={() => processPayment(amount)}>Pay</button>;
}
```

---

## Best Practices

**1. Use with memo**
```jsx
const ExpensiveComponent = memo(({ value }) => {
  // Expensive rendering
});

function Parent() {
  const [value, setValue] = useState('');
  const deferredValue = useDeferredValue(value);
  
  return <ExpensiveComponent value={deferredValue} />;
}
```

**2. Show loading indicators**
```jsx
const isStale = value !== deferredValue;
{isStale && <Spinner />}
```

**3. Don't defer user input**
```jsx
// ❌ Don't defer the input value itself
<input value={deferredValue} onChange={...} />

// ✅ Defer derived/computed values
<input value={value} onChange={...} />
<Results query={deferredValue} />
```

**4. Use for expensive operations**
```jsx
// Good candidates:
// - Large list filtering
// - Complex calculations
// - Heavy rendering
// - Chart updates
```

---

## Browser Support

✅ React 18+  
✅ All modern browsers  
✅ No polyfills needed  

---

**Interview Tips:**
- **useDeferredValue** = defer updating a value
- Returns **deferred version** of the value
- **May lag behind** during expensive renders
- **Use for**: search, filtering, slow components
- **Input stays responsive** while deferred value updates
- **Detect staleness**: `value !== deferredValue`
- **Combine with memo** for best performance
- **vs useTransition**: useDeferredValue for values, useTransition for state updates
- **Automatic**: no manual wrapping needed
- **Simpler API** than useTransition for many cases
- **Use when can't control** the state update
- **Good for props** passed from parent
- **React abandons old renders** if new value comes
- **No isPending flag** (check values instead)
- **Part of Concurrent React** features
- **React 18+ only**
- **Requires createRoot** to work
- **Non-blocking** updates
- **Priority-based** scheduling
- **Time slicing** enables smooth updates
- **Keep old UI** while rendering new
- **Better perceived performance**
- **No manual debouncing** needed
- **Gradual adoption** possible
- **Works with Suspense** and other concurrent features

</details>

---

### 86. What is Automatic Batching in React 18?

<details>
<summary>View Answer</summary>

**Automatic Batching**

A React 18 feature that automatically groups multiple state updates into a single re-render for better performance, even outside of event handlers.

---

## What is Batching?

**Batching** = React combines multiple state updates into a single re-render to improve performance.

### Example

```jsx
function Component() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);
  
  const handleClick = () => {
    setCount(c => c + 1);  // Update 1
    setFlag(f => !f);      // Update 2
    // React batches both: only 1 re-render ✅
  };
  
  return <button onClick={handleClick}>Click</button>;
}
```

**Without batching:** 2 state updates = 2 re-renders  
**With batching:** 2 state updates = 1 re-render ✅

---

## React 17 vs React 18

### React 17 (Limited Batching)

**Batched:** Inside event handlers only

```jsx
// React 17
function Component() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);
  
  // ✅ Batched (inside event handler)
  const handleClick = () => {
    setCount(c => c + 1);
    setFlag(f => !f);
    // Only 1 re-render ✅
  };
  
  // ❌ NOT batched (inside Promise)
  const handleClickAsync = () => {
    fetch('/api/data').then(() => {
      setCount(c => c + 1);
      setFlag(f => !f);
      // 2 re-renders ❌
    });
  };
  
  // ❌ NOT batched (inside setTimeout)
  const handleClickTimeout = () => {
    setTimeout(() => {
      setCount(c => c + 1);
      setFlag(f => !f);
      // 2 re-renders ❌
    }, 1000);
  };
}
```

---

### React 18 (Automatic Batching)

**Batched:** Everywhere, automatically

```jsx
// React 18
function Component() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);
  
  // ✅ Batched (event handler)
  const handleClick = () => {
    setCount(c => c + 1);
    setFlag(f => !f);
    // Only 1 re-render ✅
  };
  
  // ✅ Batched (Promise)
  const handleClickAsync = () => {
    fetch('/api/data').then(() => {
      setCount(c => c + 1);
      setFlag(f => !f);
      // Only 1 re-render ✅ (NEW!)
    });
  };
  
  // ✅ Batched (setTimeout)
  const handleClickTimeout = () => {
    setTimeout(() => {
      setCount(c => c + 1);
      setFlag(f => !f);
      // Only 1 re-render ✅ (NEW!)
    }, 1000);
  };
  
  // ✅ Batched (native event listener)
  useEffect(() => {
    const handler = () => {
      setCount(c => c + 1);
      setFlag(f => !f);
      // Only 1 re-render ✅ (NEW!)
    };
    window.addEventListener('scroll', handler);
    return () => window.removeEventListener('scroll', handler);
  }, []);
}
```

---

## Comparison Table

| Context | React 17 | React 18 |
|---------|----------|----------|
| **Event handlers** | ✅ Batched | ✅ Batched |
| **Promises** | ❌ Not batched | ✅ Batched |
| **setTimeout** | ❌ Not batched | ✅ Batched |
| **setInterval** | ❌ Not batched | ✅ Batched |
| **Native events** | ❌ Not batched | ✅ Batched |
| **async/await** | ❌ Not batched | ✅ Batched |

---

## Examples

### Example 1: Fetch Request

**React 17:**
```jsx
// React 17
function UserProfile() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  const loadUser = async () => {
    setLoading(true);  // Re-render 1
    
    try {
      const data = await fetch('/api/user').then(r => r.json());
      setUser(data);     // Re-render 2 ❌
      setLoading(false); // Re-render 3 ❌
      // Total: 3 re-renders
    } catch (err) {
      setError(err);     // Re-render 2 ❌
      setLoading(false); // Re-render 3 ❌
    }
  };
  
  return <button onClick={loadUser}>Load User</button>;
}
```

**React 18:**
```jsx
// React 18
function UserProfile() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  const loadUser = async () => {
    setLoading(true);  // Re-render 1
    
    try {
      const data = await fetch('/api/user').then(r => r.json());
      setUser(data);     // \
      setLoading(false); //  > Batched: Only 1 re-render ✅
      // Total: 2 re-renders (50% fewer!)
    } catch (err) {
      setError(err);     // \
      setLoading(false); //  > Batched: Only 1 re-render ✅
    }
  };
  
  return <button onClick={loadUser}>Load User</button>;
}
```

---

### Example 2: setTimeout

**React 17:**
```jsx
// React 17
function Counter() {
  const [count, setCount] = useState(0);
  const [doubled, setDoubled] = useState(0);
  
  const increment = () => {
    setTimeout(() => {
      setCount(c => c + 1);     // Re-render 1 ❌
      setDoubled(d => d + 2);   // Re-render 2 ❌
      // Total: 2 re-renders
    }, 100);
  };
  
  return <button onClick={increment}>Increment</button>;
}
```

**React 18:**
```jsx
// React 18
function Counter() {
  const [count, setCount] = useState(0);
  const [doubled, setDoubled] = useState(0);
  
  const increment = () => {
    setTimeout(() => {
      setCount(c => c + 1);   // \
      setDoubled(d => d + 2); //  > Batched: Only 1 re-render ✅
      // Total: 1 re-render
    }, 100);
  };
  
  return <button onClick={increment}>Increment</button>;
}
```

---

### Example 3: Native Event Listeners

**React 17:**
```jsx
// React 17
function ScrollTracker() {
  const [scrollY, setScrollY] = useState(0);
  const [isScrolling, setIsScrolling] = useState(false);
  
  useEffect(() => {
    const handleScroll = () => {
      setScrollY(window.scrollY);     // Re-render 1 ❌
      setIsScrolling(true);           // Re-render 2 ❌
      // Total: 2 re-renders per scroll event
    };
    
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, []);
  
  return <div>Scroll Y: {scrollY}</div>;
}
```

**React 18:**
```jsx
// React 18
function ScrollTracker() {
  const [scrollY, setScrollY] = useState(0);
  const [isScrolling, setIsScrolling] = useState(false);
  
  useEffect(() => {
    const handleScroll = () => {
      setScrollY(window.scrollY);   // \
      setIsScrolling(true);         //  > Batched: Only 1 re-render ✅
      // Total: 1 re-render per scroll event
    };
    
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, []);
  
  return <div>Scroll Y: {scrollY}</div>;
}
```

---

## How to Enable

### Use createRoot (Required)

**React 17 (No automatic batching):**
```jsx
import ReactDOM from 'react-dom';

ReactDOM.render(<App />, document.getElementById('root'));
```

**React 18 (Automatic batching enabled):**
```jsx
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

⚠️ **Must use `createRoot`** to get automatic batching.

---

## Opting Out with flushSync

If you need immediate re-renders (rare), use `flushSync`:

```jsx
import { flushSync } from 'react-dom';

function Component() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);
  
  const handleClick = () => {
    flushSync(() => {
      setCount(c => c + 1);  // Forces immediate re-render
    });
    
    // Now count is updated in DOM
    console.log(document.getElementById('count').textContent);  // Shows new count
    
    flushSync(() => {
      setFlag(f => !f);  // Forces another immediate re-render
    });
  };
  
  return (
    <div>
      <div id="count">{count}</div>
      <button onClick={handleClick}>Click</button>
    </div>
  );
}
```

**⚠️ Warning:** `flushSync` hurts performance. Use sparingly.

---

## When You Might Need flushSync

### 1. Measuring DOM after update

```jsx
import { flushSync } from 'react-dom';

function Component() {
  const [items, setItems] = useState([]);
  const listRef = useRef();
  
  const addItem = () => {
    flushSync(() => {
      setItems([...items, newItem]);
    });
    
    // Now DOM is updated, can measure
    const height = listRef.current.scrollHeight;
    listRef.current.scrollTop = height;
  };
  
  return <ul ref={listRef}>{items.map(i => <li key={i}>{i}</li>)}</ul>;
}
```

---

### 2. Third-party library integration

```jsx
import { flushSync } from 'react-dom';
import $ from 'jquery';

function Component() {
  const [value, setValue] = useState('');
  
  const updateValue = (newValue) => {
    flushSync(() => {
      setValue(newValue);
    });
    
    // Now DOM is updated, jQuery sees new value
    $('#element').somePlugin();
  };
}
```

---

## Performance Benefits

### Fewer Re-renders

**React 17:**
```
setState() → Re-render 1
setState() → Re-render 2
setState() → Re-render 3
Total: 3 re-renders
```

**React 18:**
```
setState()
setState()
setState()
   ↓
Batch all updates
   ↓
Re-render once
Total: 1 re-render (3x faster!)
```

---

### Real Numbers

```jsx
// Benchmark: 1000 updates

// React 17
for (let i = 0; i < 1000; i++) {
  setState1(...)  // 1000 re-renders
  setState2(...)  // 1000 re-renders
}
// Total: 2000 re-renders

// React 18
for (let i = 0; i < 1000; i++) {
  setState1(...)  // \
  setState2(...)  //  > Batched
}
// Total: 1 re-render (2000x faster!)
```

---

## Common Scenarios

### 1. API Calls

```jsx
const loadData = async () => {
  setLoading(true);
  
  try {
    const data = await api.fetchData();
    setData(data);      // Batched with next line ✅
    setLoading(false);
  } catch (error) {
    setError(error);    // Batched with next line ✅
    setLoading(false);
  }
};
```

---

### 2. Form Submission

```jsx
const handleSubmit = async (e) => {
  e.preventDefault();
  
  setSubmitting(true);
  setError(null);
  
  try {
    await submitForm(formData);
    setSuccess(true);     // Batched ✅
    setSubmitting(false);
    resetForm();
  } catch (err) {
    setError(err);        // Batched ✅
    setSubmitting(false);
  }
};
```

---

### 3. WebSocket Messages

```jsx
useEffect(() => {
  const ws = new WebSocket('ws://localhost:8080');
  
  ws.onmessage = (event) => {
    const message = JSON.parse(event.data);
    
    setMessages(prev => [...prev, message]);  // Batched ✅
    setUnreadCount(count => count + 1);
    setLastUpdate(Date.now());
  };
  
  return () => ws.close();
}, []);
```

---

## Edge Cases

### Updates in Different Functions

```jsx
// React 18: Still batched! ✅
function Component() {
  const [a, setA] = useState(0);
  const [b, setB] = useState(0);
  
  const updateA = () => setA(1);
  const updateB = () => setB(2);
  
  const handleClick = () => {
    updateA();  // \
    updateB();  //  > Batched together ✅
  };
}
```

---

### Nested State Updates

```jsx
// React 18: Batched! ✅
function Parent() {
  const [parentState, setParentState] = useState(0);
  
  return (
    <Child onUpdate={() => {
      setParentState(1);           // \
      // Child also updates state   //  > All batched ✅
    }} />
  );
}
```

---

## Migration Guide

### Step 1: Update to React 18

```bash
npm install react@18 react-dom@18
```

---

### Step 2: Use createRoot

```jsx
// Old
import ReactDOM from 'react-dom';
ReactDOM.render(<App />, root);

// New
import { createRoot } from 'react-dom/client';
const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

---

### Step 3: Test Your App

Most apps work without changes, but test:

- State updates in promises
- State updates in timeouts
- Native event listeners
- Third-party integrations

---

### Step 4: Remove Manual Batching

If you used `unstable_batchedUpdates` in React 17, you can remove it:

```jsx
// React 17
import { unstable_batchedUpdates } from 'react-dom';

setTimeout(() => {
  unstable_batchedUpdates(() => {
    setState1(...);
    setState2(...);
  });
}, 1000);

// React 18 - no longer needed!
setTimeout(() => {
  setState1(...);  // Automatically batched ✅
  setState2(...);
}, 1000);
```

---

## Debugging Batching

Check if batching is working:

```jsx
function Component() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);
  
  console.log('Render:', count, flag);
  
  const handleClick = () => {
    setCount(c => c + 1);
    setFlag(f => !f);
  };
  
  // React 17: Logs twice
  // React 18: Logs once ✅
}
```

---

## Best Practices

**1. Don't worry about batching**
```jsx
// Just write natural code
setState1(...);
setState2(...);
setState3(...);
// React handles batching automatically ✅
```

**2. Avoid flushSync unless necessary**
```jsx
// ❌ Usually not needed
flushSync(() => setState(...));

// ✅ Let React batch automatically
setState(...);
```

**3. Remove unstable_batchedUpdates**
```jsx
// React 18 doesn't need it
```

**4. Test async code**
```jsx
// Make sure async updates work correctly
// React 18 batches them now
```

---

## Summary

### React 17:
- ✅ Batching in event handlers
- ❌ No batching in promises, timeouts, native events

### React 18:
- ✅ Batching everywhere automatically
- ✅ Better performance out of the box
- ✅ No code changes needed (usually)
- ✅ Opt-out with flushSync if needed

---

**Interview Tips:**
- **Automatic batching** = React 18 feature
- **Batching** = combine multiple setState into one re-render
- **Everywhere** = promises, timeouts, native events (not just event handlers)
- **React 17** = only batched in event handlers
- **React 18** = batched everywhere automatically
- **Performance** = fewer re-renders = faster
- **createRoot required** = must use new API
- **Backward compatible** = most apps work without changes
- **flushSync** = opt-out for immediate re-render (rare)
- **No manual batching** = remove unstable_batchedUpdates
- **Async/await** = now batched in React 18
- **setTimeout** = now batched in React 18
- **Promises** = now batched in React 18
- **fetch calls** = now batched in React 18
- **Native events** = now batched in React 18
- **50-70% fewer re-renders** in many apps
- **Better perceived performance**
- **No code changes** needed for most apps
- **Gradual migration** possible
- **Part of concurrent features**
- **Transparent optimization**
- **Just works** - no configuration needed
- **Testing important** for async code
- **Third-party libraries** may need updates
- **Breaking change** only if relied on multiple re-renders
- **Major performance win** for free

</details>

---

### 87. What is the new Suspense SSR architecture?

<details>
<summary>View Answer</summary>

**New Suspense SSR Architecture**

React 18 introduces a completely reimagined Server-Side Rendering (SSR) architecture that enables streaming HTML and selective hydration, making SSR apps faster and more responsive.

---

## Problems with Old SSR (React 17)

### The Old Approach (All-or-Nothing)

In React 17, SSR had a waterfall problem:

```
1. Server: Fetch ALL data for entire page
   ↓
2. Server: Render ALL components to HTML
   ↓
3. Client: Load ALL JavaScript
   ↓
4. Client: Hydrate ENTIRE page
   ↓
5. Page interactive
```

**Problems:**
- ❌ Must wait for ALL data before sending ANY HTML
- ❌ Must load ALL JavaScript before hydrating
- ❌ Must hydrate ENTIRE page before ANY interaction
- ❌ Slow components block entire page
- ❌ Poor Time to Interactive (TTI)

---

### Example of Old Problem

```jsx
// React 17 SSR
function App() {
  return (
    <Layout>
      <NavBar />               {/* Fast */}
      <Sidebar />              {/* Fast */}
      <Comments />             {/* SLOW - blocks everything! */}
      <Footer />               {/* Fast */}
    </Layout>
  );
}

// Server must:
// 1. Wait for Comments data (5 seconds)
// 2. Render entire page
// 3. Send HTML
// Result: User waits 5 seconds to see ANYTHING
```

---

## New Suspense SSR (React 18)

### Key Improvements

1. **Streaming HTML** - Send HTML as it's ready
2. **Selective Hydration** - Hydrate parts independently
3. **Progressive Enhancement** - Page usable before full hydration
4. **Concurrent Features** - Prioritize user interactions

---

## 1. Streaming HTML

**Send HTML in chunks as components become ready**

### Before (React 17)

```jsx
// Must wait for ALL data
const html = renderToString(<App />);
res.send(html);  // Sends after everything is ready
```

**Timeline:**
```
0s:    Start fetching data
5s:    All data ready
5.1s:  Render complete
5.1s:  Send HTML
       ↓
       User sees page
```

---

### After (React 18)

```jsx
import { renderToPipeableStream } from 'react-dom/server';

function App() {
  return (
    <Layout>
      <NavBar />
      <Sidebar />
      
      {/* Wrap slow component in Suspense */}
      <Suspense fallback={<CommentsPlaceholder />}>
        <Comments />
      </Suspense>
      
      <Footer />
    </Layout>
  );
}

// Server
const { pipe } = renderToPipeableStream(<App />);
pipe(res);  // Streams HTML as it's ready!
```

**Timeline:**
```
0s:    Send initial HTML (NavBar, Sidebar, Placeholder, Footer)
       ↓
       User sees page immediately!
       ↓
5s:    Comments data ready
5s:    Stream Comments HTML
       ↓
       Comments replace placeholder
```

**Benefit:** User sees page in 0.1s instead of 5s!

---

## 2. Selective Hydration

**Hydrate components independently as their JavaScript loads**

### Before (React 17)

```jsx
// Must hydrate entire page at once
ReactDOM.hydrate(<App />, root);

// Timeline:
// 1. Wait for ALL JavaScript (2 MB)
// 2. Hydrate ENTIRE page
// 3. Page becomes interactive
// Time: 10 seconds on slow connection
```

**Problem:** Can't interact with ANY part until ALL parts are hydrated.

---

### After (React 18)

```jsx
import { hydrateRoot } from 'react-dom/client';

function App() {
  return (
    <Layout>
      <NavBar />  {/* Hydrates first */}
      
      <Suspense fallback={<Spinner />}>
        <Comments />  {/* Hydrates when ready */}
      </Suspense>
      
      <Footer />  {/* Hydrates independently */}
    </Layout>
  );
}

hydrateRoot(root, <App />);
```

**Timeline:**
```
0s:    HTML visible
1s:    NavBar JavaScript loads → NavBar hydrates → NavBar interactive!
2s:    Footer JavaScript loads → Footer hydrates → Footer interactive!
5s:    Comments JavaScript loads → Comments hydrate → Comments interactive!
```

**Benefit:** Parts of page interactive immediately!

---

## 3. Progressive Enhancement

**Page is usable before full hydration**

### Example

```jsx
function ProductPage() {
  return (
    <div>
      {/* Already interactive */}
      <Header>
        <SearchBox />  {/* Can use immediately */}
        <Cart />       {/* Can use immediately */}
      </Header>
      
      {/* Main content visible but not yet interactive */}
      <Suspense fallback={<ProductSkeleton />}>
        <ProductDetails />  {/* Hydrates when JavaScript loads */}
      </Suspense>
      
      {/* Reviews load later */}
      <Suspense fallback={<ReviewsSkeleton />}>
        <Reviews />  {/* Hydrates when JavaScript loads */}
      </Suspense>
    </div>
  );
}
```

**User Experience:**
1. Sees entire page immediately (HTML)
2. Can use search box right away (hydrated first)
3. Product details become interactive (hydrated second)
4. Reviews become interactive (hydrated last)

**No more blank screen or frozen page!**

---

## 4. Priority-Based Hydration

**React prioritizes hydrating what user is trying to interact with**

### Example

```jsx
function App() {
  return (
    <div>
      <Suspense fallback={<Spinner />}>
        <Sidebar />  {/* Hydrating... */}
      </Suspense>
      
      <Suspense fallback={<Spinner />}>
        <MainContent />  {/* Not yet hydrating */}
      </Suspense>
      
      <Suspense fallback={<Spinner />}>
        <Comments />  {/* Not yet hydrating */}
      </Suspense>
    </div>
  );
}
```

**Scenario:**
1. Sidebar is hydrating
2. User clicks on MainContent
3. React **pauses** Sidebar hydration
4. React **prioritizes** MainContent hydration
5. MainContent becomes interactive immediately
6. React resumes Sidebar hydration

**Benefit:** Most responsive UX possible!

---

## Complete Example

### Server Code

```jsx
// server.js
import express from 'express';
import { renderToPipeableStream } from 'react-dom/server';
import App from './App';

const app = express();

app.get('/', (req, res) => {
  const { pipe, abort } = renderToPipeableStream(
    <App />,
    {
      bootstrapScripts: ['/client.js'],
      onShellReady() {
        // Send initial HTML immediately
        res.setHeader('Content-Type', 'text/html');
        pipe(res);
      },
      onShellError(error) {
        res.status(500).send('Server error');
      },
      onAllReady() {
        // All content ready (for crawlers)
      },
      onError(error) {
        console.error(error);
      }
    }
  );
  
  setTimeout(abort, 10000);  // Abort after 10s
});

app.listen(3000);
```

---

### App Component

```jsx
// App.jsx
import { Suspense } from 'react';

function App() {
  return (
    <html>
      <head>
        <title>My App</title>
      </head>
      <body>
        <Header />
        
        {/* Sidebar loads separately */}
        <Suspense fallback={<SidebarSkeleton />}>
          <Sidebar />
        </Suspense>
        
        <main>
          {/* Main content loads separately */}
          <Suspense fallback={<ContentSkeleton />}>
            <MainContent />
          </Suspense>
          
          {/* Comments load last */}
          <Suspense fallback={<CommentsSkeleton />}>
            <Comments />
          </Suspense>
        </main>
        
        <Footer />
      </body>
    </html>
  );
}

export default App;
```

---

### Client Code

```jsx
// client.js
import { hydrateRoot } from 'react-dom/client';
import App from './App';

hydrateRoot(document, <App />);
```

---

### Async Components (Server)

```jsx
// Comments.jsx (Server Component)
async function Comments({ postId }) {
  // Fetch data on server
  const comments = await db.comments.findMany({
    where: { postId }
  });
  
  return (
    <div className="comments">
      {comments.map(comment => (
        <Comment key={comment.id} data={comment} />
      ))}
    </div>
  );
}

export default Comments;
```

---

## Comparison: Old vs New

### Old SSR (React 17)

```
         SERVER                     CLIENT
┌─────────────────────────────────────────────┐
│ 1. Fetch ALL data (5s)           │     [Waiting...]
│ 2. Render ALL HTML (1s)          │     [Waiting...]
│ 3. Send ALL HTML                 │ →  HTML received
└─────────────────────────────────────────────┘
                                      4. Load ALL JS (3s)
                                      5. Hydrate ALL (1s)
                                      6. Interactive!

Total: 10 seconds until interactive
```

---

### New SSR (React 18)

```
         SERVER                     CLIENT
┌─────────────────────────────────────────────┐
│ 1. Render fast parts (0.1s)      │ →  HTML received
│ 2. Stream HTML                   │     Page visible!
└─────────────────────────────────────────────┘
                                      3. Load Header JS (0.5s)
                                      4. Hydrate Header
                                      5. Header interactive! ✅
                                      
┌─────────────────────────────────────────────┐
│ 6. Slow data ready (5s)          │
│ 7. Stream slow parts             │ →  Content updated
└─────────────────────────────────────────────┘
                                      8. Load content JS (1s)
                                      9. Hydrate content
                                      10. Content interactive! ✅

Total: 0.6 seconds until first interaction!
      6 seconds until full interaction!
```

---

## Benefits Summary

### 1. Faster First Paint

**Before:** 5+ seconds to see anything  
**After:** <1 second to see page

---

### 2. Progressive Interactivity

**Before:** Must wait for entire page  
**After:** Parts interactive immediately

---

### 3. Better Perceived Performance

**Before:** Long blank screen  
**After:** Something visible immediately

---

### 4. Resilient to Slow Components

**Before:** One slow component blocks everything  
**After:** Slow components don't block fast ones

---

### 5. Better User Experience

**Before:** Frustrating wait  
**After:** Can start using app immediately

---

## Real-World Example: Blog Post

```jsx
// BlogPost.jsx
import { Suspense } from 'react';

function BlogPost({ postId }) {
  return (
    <article>
      {/* Fast: Immediate */}
      <Header />
      
      {/* Fast: Loads quickly */}
      <Suspense fallback={<PostSkeleton />}>
        <PostContent postId={postId} />
      </Suspense>
      
      {/* Slow: Can take 2-3 seconds */}
      <Suspense fallback={<CommentsSkeleton />}>
        <Comments postId={postId} />
      </Suspense>
      
      {/* Slow: External API call */}
      <Suspense fallback={<RecommendationsSkeleton />}>
        <Recommendations postId={postId} />
      </Suspense>
      
      {/* Fast: Static */}
      <Footer />
    </article>
  );
}

// User experience:
// 0.1s: Sees header, post skeleton, comments skeleton, footer
// 0.5s: Post content appears, header becomes interactive
// 2.0s: Comments appear and become interactive
// 3.0s: Recommendations appear
// Total: Page usable at 0.5s instead of 3s!
```

---

## Migration Guide

### 1. Update React

```bash
npm install react@18 react-dom@18
```

---

### 2. Switch to renderToPipeableStream

**Before:**
```jsx
import { renderToString } from 'react-dom/server';

const html = renderToString(<App />);
res.send(html);
```

**After:**
```jsx
import { renderToPipeableStream } from 'react-dom/server';

const { pipe } = renderToPipeableStream(<App />, {
  bootstrapScripts: ['/client.js'],
  onShellReady() {
    res.setHeader('Content-Type', 'text/html');
    pipe(res);
  }
});
```

---

### 3. Use hydrateRoot

**Before:**
```jsx
import { hydrate } from 'react-dom';

hydrate(<App />, document.getElementById('root'));
```

**After:**
```jsx
import { hydrateRoot } from 'react-dom/client';

hydrateRoot(document, <App />);
```

---

### 4. Add Suspense Boundaries

```jsx
import { Suspense } from 'react';

function App() {
  return (
    <div>
      {/* Wrap slow components */}
      <Suspense fallback={<Spinner />}>
        <SlowComponent />
      </Suspense>
    </div>
  );
}
```

---

## Best Practices

**1. Use Suspense for slow components**
```jsx
<Suspense fallback={<Skeleton />}>
  <SlowComponent />
</Suspense>
```

**2. Provide good fallbacks**
```jsx
// ✅ Good: Skeleton matches content layout
<Suspense fallback={<CommentsSkeleton />}>
  <Comments />
</Suspense>

// ❌ Bad: Generic spinner
<Suspense fallback={<div>Loading...</div>}>
  <Comments />
</Suspense>
```

**3. Multiple Suspense boundaries**
```jsx
// Different boundaries for different sections
<Suspense fallback={<SidebarSkeleton />}>
  <Sidebar />
</Suspense>
<Suspense fallback={<ContentSkeleton />}>
  <Content />
</Suspense>
```

**4. Use onShellReady for streaming**
```jsx
renderToPipeableStream(<App />, {
  onShellReady() {
    pipe(res);  // Start streaming immediately
  }
});
```

---

**Interview Tips:**
- **New Suspense SSR** = React 18 streaming + selective hydration
- **Old SSR problem** = all-or-nothing waterfall
- **Streaming HTML** = send chunks as ready, not all at once
- **Selective hydration** = hydrate parts independently
- **Progressive enhancement** = page usable before full hydration
- **Priority-based** = hydrate what user interacts with first
- **renderToPipeableStream** = new server rendering API
- **hydrateRoot** = new client hydration API
- **Suspense boundaries** = wrap slow components
- **Benefits**: faster first paint, progressive interactivity, better UX
- **No blank screen** = something visible immediately
- **Resilient** = slow components don't block fast ones
- **onShellReady** = start streaming initial HTML
- **onAllReady** = when everything is ready (for crawlers)
- **Fallback UI** = shown while component loads
- **React 18+ only** = not available in React 17
- **Backward compatible** = existing SSR apps still work
- **Gradual adoption** = add Suspense boundaries incrementally
- **Works with Server Components** = powerful combination
- **Time to Interactive** = dramatically reduced
- **Perceived performance** = much better
- **Multiple Suspense** = for different page sections
- **Code splitting** = works great with streaming
- **Skeleton screens** = better than spinners
- **Game changer** = for SSR performance

</details>

---

### 88. What is startTransition API?

<details>
<summary>View Answer</summary>

**startTransition API**

A React 18 function that lets you mark state updates as non-urgent transitions, keeping your UI responsive without needing the useTransition hook.

---

## What is startTransition?

**startTransition** is a standalone function (not a hook) that wraps state updates to mark them as low-priority transitions.

### Syntax

```jsx
import { startTransition } from 'react';

startTransition(() => {
  // State updates here are non-urgent
  setState(newValue);
});
```

**Key difference from useTransition:**
- `startTransition` = function (no isPending flag)
- `useTransition` = hook (with isPending flag)

---

## Basic Example

### Without startTransition (Blocking)

```jsx
import { useState } from 'react';

function SearchBox({ data }) {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  
  const handleChange = (e) => {
    const value = e.target.value;
    setQuery(value);  // Urgent
    
    // Expensive filtering blocks UI
    const filtered = data.filter(item => 
      item.name.includes(value)
    );
    setResults(filtered);  // Blocks input!
  };
  
  return (
    <div>
      <input value={query} onChange={handleChange} />  {/* Laggy! */}
      <List items={results} />
    </div>
  );
}
```

**Problem:** Input freezes while filtering.

---

### With startTransition (Non-blocking)

```jsx
import { useState, startTransition } from 'react';

function SearchBox({ data }) {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  
  const handleChange = (e) => {
    const value = e.target.value;
    
    // Urgent: Update input immediately
    setQuery(value);
    
    // Non-urgent: Wrap in startTransition
    startTransition(() => {
      const filtered = data.filter(item => 
        item.name.includes(value)
      );
      setResults(filtered);
    });
  };
  
  return (
    <div>
      <input value={query} onChange={handleChange} />  {/* Smooth! */}
      <List items={results} />
    </div>
  );
}
```

**Result:** Input stays responsive!

---

## startTransition vs useTransition

### startTransition (Function)

```jsx
import { startTransition } from 'react';

function Component() {
  const handleClick = () => {
    startTransition(() => {
      setState(newValue);
    });
  };
  
  // No isPending flag available
  return <button onClick={handleClick}>Click</button>;
}
```

**Use when:** You don't need loading indicator.

---

### useTransition (Hook)

```jsx
import { useTransition } from 'react';

function Component() {
  const [isPending, startTransition] = useTransition();
  
  const handleClick = () => {
    startTransition(() => {
      setState(newValue);
    });
  };
  
  // isPending flag available
  return (
    <div>
      {isPending && <Spinner />}
      <button onClick={handleClick}>Click</button>
    </div>
  );
}
```

**Use when:** You need loading indicator.

---

## Comparison Table

| Feature | startTransition | useTransition |
|---------|-----------------|---------------|
| **Type** | Function | Hook |
| **Import** | `import { startTransition }` | `import { useTransition }` |
| **Returns** | Nothing | `[isPending, startTransition]` |
| **isPending** | ❌ No | ✅ Yes |
| **Where** | Anywhere | Inside component |
| **Loading indicator** | ❌ No | ✅ Yes |
| **Simpler** | ✅ Yes | More features |

---

## When to Use startTransition

### 1. Outside Components

```jsx
import { startTransition } from 'react';

// In a utility function (outside component)
export function updateGlobalState(value) {
  startTransition(() => {
    globalStore.setState(value);
  });
}
```

**Can't use useTransition here** (hooks only in components).

---

### 2. No Loading Indicator Needed

```jsx
import { useState, startTransition } from 'react';

function TabsSimple() {
  const [tab, setTab] = useState('home');
  
  return (
    <div>
      <button onClick={() => startTransition(() => setTab('home'))}>
        Home
      </button>
      <button onClick={() => startTransition(() => setTab('posts'))}>
        Posts
      </button>
      
      {tab === 'home' && <Home />}
      {tab === 'posts' && <Posts />}
    </div>
  );
}
```

**Simple:** No need for isPending.

---

### 3. Event Handlers

```jsx
import { useState, startTransition } from 'react';

function Component() {
  const [filter, setFilter] = useState('');
  const [items, setItems] = useState([]);
  
  const applyFilter = (value) => {
    setFilter(value);  // Urgent
    
    startTransition(() => {
      // Non-urgent filtering
      const filtered = largeDataset.filter(item => 
        item.name.includes(value)
      );
      setItems(filtered);
    });
  };
  
  return (
    <input onChange={e => applyFilter(e.target.value)} />
  );
}
```

---

### 4. Navigation

```jsx
import { startTransition } from 'react';
import { useNavigate } from 'react-router-dom';

function ProductCard({ product }) {
  const navigate = useNavigate();
  
  const goToProduct = () => {
    startTransition(() => {
      navigate(`/products/${product.id}`);
    });
  };
  
  return (
    <div onClick={goToProduct}>
      <h3>{product.name}</h3>
    </div>
  );
}
```

---

## Real-World Examples

### Example 1: Search

```jsx
import { useState, startTransition } from 'react';

function GlobalSearch({ products }) {
  const [searchTerm, setSearchTerm] = useState('');
  const [results, setResults] = useState([]);
  
  const handleSearch = (e) => {
    const value = e.target.value;
    
    // Update input immediately (high priority)
    setSearchTerm(value);
    
    // Search is non-urgent (low priority)
    startTransition(() => {
      const filtered = products.filter(p => {
        return p.name.toLowerCase().includes(value.toLowerCase()) ||
               p.description.toLowerCase().includes(value.toLowerCase()) ||
               p.category.toLowerCase().includes(value.toLowerCase());
      });
      setResults(filtered);
    });
  };
  
  return (
    <div className="search">
      <input
        type="search"
        value={searchTerm}
        onChange={handleSearch}
        placeholder="Search products..."
      />
      
      <div className="results">
        {results.map(product => (
          <ProductCard key={product.id} product={product} />
        ))}
      </div>
    </div>
  );
}
```

---

### Example 2: Sorting

```jsx
import { useState, startTransition } from 'react';

function DataTable({ data }) {
  const [sortBy, setSortBy] = useState('name');
  const [sortedData, setSortedData] = useState(data);
  
  const handleSort = (field) => {
    // Update sort indicator immediately
    setSortBy(field);
    
    // Sort in background
    startTransition(() => {
      const sorted = [...data].sort((a, b) => {
        if (a[field] < b[field]) return -1;
        if (a[field] > b[field]) return 1;
        return 0;
      });
      setSortedData(sorted);
    });
  };
  
  return (
    <table>
      <thead>
        <tr>
          <th onClick={() => handleSort('name')}>
            Name {sortBy === 'name' && '↑'}
          </th>
          <th onClick={() => handleSort('price')}>
            Price {sortBy === 'price' && '↑'}
          </th>
          <th onClick={() => handleSort('date')}>
            Date {sortBy === 'date' && '↑'}
          </th>
        </tr>
      </thead>
      <tbody>
        {sortedData.map(row => (
          <tr key={row.id}>
            <td>{row.name}</td>
            <td>{row.price}</td>
            <td>{row.date}</td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

---

### Example 3: Tab Switching

```jsx
import { useState, startTransition } from 'react';

function Dashboard() {
  const [activeTab, setActiveTab] = useState('overview');
  
  const switchTab = (tab) => {
    startTransition(() => {
      setActiveTab(tab);
    });
  };
  
  return (
    <div className="dashboard">
      <nav>
        <button 
          onClick={() => switchTab('overview')}
          className={activeTab === 'overview' ? 'active' : ''}
        >
          Overview
        </button>
        <button 
          onClick={() => switchTab('analytics')}
          className={activeTab === 'analytics' ? 'active' : ''}
        >
          Analytics
        </button>
        <button 
          onClick={() => switchTab('reports')}
          className={activeTab === 'reports' ? 'active' : ''}
        >
          Reports
        </button>
      </nav>
      
      <div className="content">
        {activeTab === 'overview' && <OverviewTab />}
        {activeTab === 'analytics' && <AnalyticsTab />}  {/* Slow */}
        {activeTab === 'reports' && <ReportsTab />}      {/* Slow */}
      </div>
    </div>
  );
}
```

---

### Example 4: Filters

```jsx
import { useState, startTransition } from 'react';

function ProductFilters({ products }) {
  const [category, setCategory] = useState('all');
  const [priceRange, setPriceRange] = useState([0, 1000]);
  const [filtered, setFiltered] = useState(products);
  
  const applyFilters = (newCategory, newPriceRange) => {
    startTransition(() => {
      const result = products.filter(p => {
        const categoryMatch = newCategory === 'all' || p.category === newCategory;
        const priceMatch = p.price >= newPriceRange[0] && p.price <= newPriceRange[1];
        return categoryMatch && priceMatch;
      });
      setFiltered(result);
    });
  };
  
  const handleCategoryChange = (e) => {
    const newCategory = e.target.value;
    setCategory(newCategory);
    applyFilters(newCategory, priceRange);
  };
  
  const handlePriceChange = (newRange) => {
    setPriceRange(newRange);
    applyFilters(category, newRange);
  };
  
  return (
    <div>
      <select value={category} onChange={handleCategoryChange}>
        <option value="all">All Categories</option>
        <option value="electronics">Electronics</option>
        <option value="clothing">Clothing</option>
        <option value="books">Books</option>
      </select>
      
      <PriceSlider value={priceRange} onChange={handlePriceChange} />
      
      <ProductGrid products={filtered} />
    </div>
  );
}
```

---

## How It Works

### Priority System

```jsx
startTransition(() => {
  setState(value);  // Marked as LOW PRIORITY
});

setState(value);  // Normal: HIGH PRIORITY
```

**React behavior:**
1. High priority updates execute first
2. Low priority updates can be interrupted
3. Low priority updates resume when React has time

---

### Example Timeline

```jsx
function Component() {
  const [input, setInput] = useState('');
  const [list, setList] = useState([]);
  
  const handleChange = (e) => {
    const value = e.target.value;
    
    setInput(value);  // High priority
    
    startTransition(() => {
      setList(generateList(value));  // Low priority
    });
  };
}
```

**Timeline:**
```
User types "a":
  1. setInput("a") executes immediately (high priority)
  2. Input shows "a"
  3. startTransition starts rendering list
  
User types "b" (before list finishes):
  1. setInput("ab") interrupts transition
  2. Input shows "ab" immediately
  3. Previous list render abandoned
  4. New transition starts with "ab"
  
User stops typing:
  1. List render completes
  2. List updates with final results
```

---

## Best Practices

### 1. Use for non-urgent updates

```jsx
// ✅ Good: Non-urgent updates
startTransition(() => {
  setSearchResults(filteredData);
  setAnalytics(processedData);
  setChart(chartData);
});

// ❌ Bad: Urgent user input
startTransition(() => {
  setInputValue(e.target.value);  // Don't defer input!
});
```

---

### 2. Keep user input responsive

```jsx
const handleChange = (e) => {
  const value = e.target.value;
  
  setInput(value);  // ✅ Urgent: immediate
  
  startTransition(() => {
    setResults(expensiveFilter(value));  // ✅ Non-urgent: deferred
  });
};
```

---

### 3. Use for expensive computations

```jsx
startTransition(() => {
  // Good candidates:
  // - Large list filtering
  // - Complex calculations
  // - Heavy rendering
  // - Chart updates
  setSortedData(expensiveSort(data));
});
```

---

### 4. Don't wrap critical actions

```jsx
// ❌ Bad: Payment confirmation is critical
startTransition(() => {
  processPayment();
});

// ✅ Good: Critical actions are immediate
processPayment();
startTransition(() => {
  updateAnalytics();  // Non-critical
});
```

---

## Common Patterns

### Pattern 1: Search with History

```jsx
import { useState, startTransition } from 'react';

function SearchWithHistory() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [history, setHistory] = useState([]);
  
  const handleSearch = (value) => {
    // Update query immediately
    setQuery(value);
    
    // Update results and history in transition
    startTransition(() => {
      const filtered = performSearch(value);
      setResults(filtered);
      
      if (value) {
        setHistory(prev => [value, ...prev.slice(0, 4)]);
      }
    });
  };
  
  return (
    <div>
      <input value={query} onChange={e => handleSearch(e.target.value)} />
      <Results items={results} />
      <History items={history} />
    </div>
  );
}
```

---

### Pattern 2: Multi-Step Form

```jsx
import { useState, startTransition } from 'react';

function MultiStepForm() {
  const [step, setStep] = useState(1);
  const [formData, setFormData] = useState({});
  
  const nextStep = () => {
    startTransition(() => {
      setStep(step + 1);
    });
  };
  
  return (
    <div>
      {step === 1 && <Step1 onNext={nextStep} />}
      {step === 2 && <Step2 onNext={nextStep} />}
      {step === 3 && <Step3 onSubmit={() => {}} />}
    </div>
  );
}
```

---

## Debugging

### Check if transition is working

```jsx
import { useState, startTransition } from 'react';

function Component() {
  const [value, setValue] = useState('');
  
  console.log('Render:', value);
  
  const handleChange = (e) => {
    setValue(e.target.value);
    
    startTransition(() => {
      console.log('Transition update');  // May not execute if interrupted
      performExpensiveUpdate();
    });
  };
}
```

---

## Performance Impact

### Without startTransition

```
User types "a":
  Input update + List update = 200ms (frozen)
  
User types "b":
  Input update + List update = 200ms (frozen)
  
Total: 400ms of frozen UI
```

---

### With startTransition

```
User types "a":
  Input update = <16ms (smooth)
  List update = deferred
  
User types "b":
  Input update = <16ms (smooth)
  Previous list update abandoned
  New list update = deferred
  
Total: <32ms perceived delay
```

**Result:** 12x better!

---

**Interview Tips:**
- **startTransition** = mark updates as non-urgent
- **Standalone function** (not a hook)
- **No isPending** (use useTransition for that)
- **Use anywhere** (not limited to components)
- **vs useTransition**: startTransition has no isPending flag
- **Keeps UI responsive** during expensive updates
- **Can be interrupted** if higher priority update comes
- **React 18+ only** feature
- **Requires createRoot** to work
- **Concurrent rendering** must be enabled
- **Use for**: search, filtering, sorting, navigation, non-critical updates
- **Don't use for**: user input, critical actions, urgent updates
- **Simpler than useTransition** when you don't need loading indicator
- **Good for**: event handlers, utility functions, outside components
- **Priority-based** scheduling
- **Automatic optimization** by React
- **Part of Concurrent React** features
- **No performance cost** if not used
- **Better than debouncing** for many cases
- **Works with** other concurrent features
- **Import from 'react'** not 'react-dom'
- **Wraps multiple** state updates if needed
- **Old updates abandoned** if new ones come
- **Time slicing** enables interruptibility
- **Gradual adoption** - add where beneficial

</details>

---

### 89. How do Server Components improve performance?

<details>
<summary>View Answer</summary>

**Server Components Performance Benefits**

React Server Components dramatically improve performance through zero bundle size, direct backend access, automatic code splitting, and reduced client-side work.

---

## Key Performance Improvements

### 1. Zero Bundle Size

**The biggest win: Server Components don't ship JavaScript to the client**

#### Before (Client Component)

```jsx
'use client';

import { marked } from 'marked';           // 50 KB
import { format } from 'date-fns';         // 70 KB
import hljs from 'highlight.js';           // 100 KB
import { Chart } from 'chart.js';          // 200 KB

function BlogPost({ markdown }) {
  const html = marked(markdown);
  const highlighted = hljs.highlightAuto(html);
  
  return <div dangerouslySetInnerHTML={{ __html: highlighted.value }} />;
}

// Client bundle: +420 KB! 📦
```

---

#### After (Server Component)

```jsx
// Server Component (no 'use client')
import { marked } from 'marked';           // 0 KB to client
import { format } from 'date-fns';         // 0 KB to client
import hljs from 'highlight.js';           // 0 KB to client
import { Chart } from 'chart.js';          // 0 KB to client

function BlogPost({ markdown }) {
  const html = marked(markdown);
  const highlighted = hljs.highlightAuto(html);
  
  return <div dangerouslySetInnerHTML={{ __html: highlighted.value }} />;
}

// Client bundle: +0 KB! ✅
// 420 KB saved!
```

**Impact:** Faster downloads, faster parse time, less memory usage.

---

### 2. Direct Backend Access (No Extra API Calls)

#### Before (Client Component)

```jsx
// Client Component
'use client';

import { useState, useEffect } from 'react';

function ProductList() {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    // Extra network request
    fetch('/api/products')
      .then(r => r.json())
      .then(data => {
        setProducts(data);
        setLoading(false);
      });
  }, []);
  
  if (loading) return <Spinner />;
  
  return (
    <ul>
      {products.map(p => <li key={p.id}>{p.name}</li>)}
    </ul>
  );
}

// Also need API route:
// app/api/products/route.js
export async function GET() {
  const products = await db.products.findMany();
  return Response.json(products);
}

// Timeline:
// 1. Load JavaScript (500ms)
// 2. Execute React (100ms)
// 3. Mount component (50ms)
// 4. Fetch from API (200ms) ← Extra request!
// 5. Re-render with data (50ms)
// Total: 900ms
```

---

#### After (Server Component)

```jsx
// Server Component
import { db } from '@/lib/database';

async function ProductList() {
  // Direct database access - no API needed!
  const products = await db.products.findMany();
  
  return (
    <ul>
      {products.map(p => <li key={p.id}>{p.name}</li>)}
    </ul>
  );
}

// No API route needed!

// Timeline:
// 1. Fetch from DB on server (50ms)
// 2. Render HTML (20ms)
// 3. Send to client (30ms)
// Total: 100ms

// 9x faster! ⚡
```

**Savings:**
- No extra API endpoint
- No extra network request
- No client-side loading state
- No waterfall (parallel requests on server)

---

### 3. Reduced JavaScript Parse/Execution Time

**Timeline Comparison:**

#### Client Component Timeline

```
1. Download JS bundle (1000ms on 3G)
2. Parse JavaScript (300ms)
3. Execute React (150ms)
4. Mount component (50ms)
5. Fetch data (200ms)
6. Re-render (50ms)

Total: 1750ms until interactive
```

---

#### Server Component Timeline

```
1. Receive HTML (100ms)
2. Display content immediately
3. (Optional) Hydrate interactive parts (200ms)

Total: 300ms until interactive

5.8x faster! ⚡
```

---

### 4. Automatic Code Splitting

**Every Server Component is automatically code-split**

#### Before (Manual Code Splitting)

```jsx
import { lazy, Suspense } from 'react';

// Must manually lazy-load
const HeavyComponent = lazy(() => import('./HeavyComponent'));
const Chart = lazy(() => import('./Chart'));
const Editor = lazy(() => import('./Editor'));

function Dashboard() {
  return (
    <div>
      <Suspense fallback={<Spinner />}>
        <HeavyComponent />
      </Suspense>
      <Suspense fallback={<Spinner />}>
        <Chart />
      </Suspense>
      <Suspense fallback={<Spinner />}>
        <Editor />
      </Suspense>
    </div>
  );
}
```

---

#### After (Automatic Code Splitting)

```jsx
// Server Components - automatically code-split!
function Dashboard() {
  return (
    <div>
      <HeavyComponent />  {/* Auto-split */}
      <Chart />           {/* Auto-split */}
      <Editor />          {/* Auto-split */}
    </div>
  );
}

// Each Server Component is automatically:
// 1. Code-split
// 2. Loaded on demand
// 3. Never sent to client if not needed
```

**Benefit:** Zero configuration, optimal splitting.

---

### 5. Parallel Data Fetching

#### Before (Waterfall)

```jsx
// Client Component - Waterfall problem
'use client';

function Page() {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);
  
  useEffect(() => {
    // 1. Fetch user (200ms)
    fetch('/api/user')
      .then(r => r.json())
      .then(userData => {
        setUser(userData);
        
        // 2. Then fetch posts (200ms) ← Waterfall!
        fetch(`/api/posts?userId=${userData.id}`)
          .then(r => r.json())
          .then(setPosts);
      });
  }, []);
  
  // Total: 400ms (sequential)
}
```

---

#### After (Parallel)

```jsx
// Server Components - Parallel by default!
async function Page() {
  // Both fetch in parallel on server
  const [user, posts] = await Promise.all([
    db.user.findFirst(),        // 200ms \
    db.posts.findMany()         // 200ms  } Parallel!
  ]);
  
  // Total: 200ms (parallel)
  
  return (
    <div>
      <User data={user} />
      <Posts data={posts} />
    </div>
  );
}

// 2x faster! ⚡
```

---

### 6. No Prop Drilling / Context Overhead

#### Before (Client Components with Context)

```jsx
'use client';

// Context provider adds overhead
const UserContext = createContext();

function App() {
  const [user, setUser] = useState(null);
  
  return (
    <UserContext.Provider value={user}>  {/* Re-renders all consumers */}
      <Header />
      <Sidebar />
      <Content />
    </UserContext.Provider>
  );
}

// Every consumer re-renders when user changes
function Header() {
  const user = useContext(UserContext);  // Re-renders
  return <div>{user.name}</div>;
}
```

---

#### After (Server Components)

```jsx
// Server Components - just pass props
async function App() {
  const user = await db.user.findFirst();
  
  return (
    <>
      <Header user={user} />    {/* No re-renders */}
      <Sidebar user={user} />   {/* No re-renders */}
      <Content user={user} />   {/* No re-renders */}
    </>
  );
}

function Header({ user }) {
  return <div>{user.name}</div>;
}

// No context, no re-renders, no overhead!
```

---

## Real-World Performance Metrics

### Example: E-commerce Product Page

#### Before (All Client Components)

```
Metrics:
- Bundle size: 800 KB
- Time to Interactive: 4.2s
- First Contentful Paint: 2.1s
- Total Blocking Time: 890ms
- Page Load: 5.8s
```

---

#### After (Server Components)

```
Metrics:
- Bundle size: 180 KB (77% reduction)
- Time to Interactive: 0.9s (4.6x faster)
- First Contentful Paint: 0.3s (7x faster)
- Total Blocking Time: 120ms (7.4x faster)
- Page Load: 1.4s (4.1x faster)
```

**Real improvement: 4-7x faster across all metrics!**

---

## Complete Example: Blog Platform

### Before (Client Components)

```jsx
// app/posts/[id]/page.jsx
'use client';

import { useState, useEffect } from 'react';
import { marked } from 'marked';              // 50 KB
import hljs from 'highlight.js';              // 100 KB
import { formatDistance } from 'date-fns';    // 70 KB

function BlogPost({ params }) {
  const [post, setPost] = useState(null);
  const [comments, setComments] = useState([]);
  const [related, setRelated] = useState([]);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    // Three sequential requests (waterfall)
    fetch(`/api/posts/${params.id}`)
      .then(r => r.json())
      .then(postData => {
        setPost(postData);
        return fetch(`/api/posts/${params.id}/comments`);
      })
      .then(r => r.json())
      .then(commentsData => {
        setComments(commentsData);
        return fetch(`/api/posts/${params.id}/related`);
      })
      .then(r => r.json())
      .then(relatedData => {
        setRelated(relatedData);
        setLoading(false);
      });
  }, [params.id]);
  
  if (loading) return <Spinner />;
  
  const html = marked(post.content);
  const highlighted = hljs.highlightAuto(html);
  
  return (
    <article>
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: highlighted.value }} />
      <Comments data={comments} />
      <Related posts={related} />
    </article>
  );
}

// Performance:
// - Bundle: +220 KB (marked + hljs + date-fns)
// - Requests: 3 sequential (600ms waterfall)
// - Time to Interactive: 3.5s
```

---

### After (Server Components)

```jsx
// app/posts/[id]/page.jsx
import { db } from '@/lib/database';
import { marked } from 'marked';              // 0 KB to client!
import hljs from 'highlight.js';              // 0 KB to client!
import { formatDistance } from 'date-fns';    // 0 KB to client!
import CommentForm from './CommentForm';      // Only this is client

async function BlogPost({ params }) {
  // Parallel data fetching on server
  const [post, comments, related] = await Promise.all([
    db.post.findUnique({ where: { id: params.id } }),
    db.comment.findMany({ where: { postId: params.id } }),
    db.post.findMany({ where: { categoryId: post.categoryId }, take: 5 })
  ]);
  
  // Process on server
  const html = marked(post.content);
  const highlighted = hljs.highlightAuto(html);
  
  return (
    <article>
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: highlighted.value }} />
      
      <div>
        {comments.map(comment => (
          <div key={comment.id}>
            <strong>{comment.author}</strong>
            <p>{comment.text}</p>
          </div>
        ))}
      </div>
      
      {/* Only form is client component */}
      <CommentForm postId={post.id} />
      
      <aside>
        <h3>Related Posts</h3>
        {related.map(p => <PostCard key={p.id} post={p} />)}
      </aside>
    </article>
  );
}

// Performance:
// - Bundle: +5 KB (only CommentForm client code)
// - Requests: 0 from client (all on server)
// - Time to Interactive: 0.4s

// 8.75x faster! ⚡
// 98% smaller bundle! 📦
```

---

## Performance Wins Summary

### 1. Bundle Size

```
Typical SPA: 500-1500 KB
With Server Components: 50-200 KB

Savings: 75-90% smaller bundles
```

### 2. Network Requests

```
Client Components: N API requests from browser
Server Components: 0 API requests from browser

Benefit: Fewer round trips, faster loads
```

### 3. Data Fetching

```
Client: Sequential (waterfall)
Server: Parallel by default

Speed: 2-5x faster
```

### 4. Parse Time

```
Client: Parse all JavaScript
Server: Parse only interactive parts

Reduction: 60-80% less parsing
```

### 5. Execution Time

```
Client: Execute all components
Server: Execute only interactive parts

Reduction: 70-90% less execution
```

### 6. Memory Usage

```
Client: Load everything in memory
Server: Only interactive parts

Reduction: 60-80% less memory
```

---

## Comparison Chart

### Time to Interactive

```
┌─────────────────────────────────────────┐
│ Client Components        ████████████████│ 4.2s
│ Server Components        ███             │ 0.8s
└─────────────────────────────────────────┘
                          5.25x faster!
```

### Bundle Size

```
┌─────────────────────────────────────────┐
│ Client Components        ████████████████│ 800 KB
│ Server Components        ██              │ 120 KB
└─────────────────────────────────────────┘
                          85% reduction!
```

### Network Requests

```
┌─────────────────────────────────────────┐
│ Client Components        ████████        │ 8 requests
│ Server Components        █               │ 1 request
└─────────────────────────────────────────┘
                          87% fewer!
```

---

## Why It's So Fast

### 1. Less JavaScript = Faster Everything

```
Less to download   → Faster load
Less to parse      → Faster startup
Less to execute    → Faster interactive
Less in memory     → Better performance
```

### 2. Server is Faster than Browser

```
Server CPU:  Fast (datacenter)
Browser CPU: Slow (user device)

Server Network:  Fast (datacenter)
Browser Network: Slow (3G/4G)

Server to DB:    <1ms latency
Browser to API:  50-200ms latency
```

### 3. No Client-Side Waterfalls

```
Client: Request 1 → Request 2 → Request 3 (sequential)
Server: All requests in parallel
```

---

## Best Practices for Maximum Performance

**1. Default to Server Components**
```jsx
// Server Component (default)
function Component() {
  // Fast by default
}
```

**2. Use Client only for interactivity**
```jsx
'use client';
// Only for: state, effects, event handlers
```

**3. Move heavy libraries to server**
```jsx
// Server Component
import { heavy } from 'heavy-library';  // 0 KB to client
```

**4. Fetch data in parallel**
```jsx
const [a, b, c] = await Promise.all([...]);  // Parallel!
```

**5. Keep client components small**
```jsx
// Small client component, rest is server
'use client';
function Button() {  // Minimal JS
  return <button onClick={...}>Click</button>;
}
```

---

**Interview Tips:**
- **Zero bundle size** = biggest win (server code never sent to client)
- **Direct backend access** = no extra API routes or requests
- **Automatic code splitting** = every server component split automatically
- **Parallel data fetching** = multiple queries run simultaneously
- **Reduced parsing** = less JavaScript to parse
- **Reduced execution** = less JavaScript to execute
- **Less memory** = only interactive parts in browser
- **No waterfalls** = avoid client-side fetch chains
- **Faster Time to Interactive** = 3-8x improvement
- **Smaller bundles** = 75-90% reduction
- **Better Core Web Vitals** = improved FCP, LCP, TTI
- **Server faster than browser** = better CPU, network, database access
- **No prop drilling overhead** = just pass props directly
- **No context re-renders** = components don't re-render unnecessarily
- **Heavy libraries on server** = markdown, charts, etc. stay server-side
- **SEO benefits** = fully rendered HTML
- **Works on slow devices** = less client-side work
- **Works on slow networks** = less to download
- **Real metrics**: 4-7x faster in production
- **Composition** = mix server and client strategically
- **Default to server** = opt into client only when needed
- **Progressive enhancement** = works even if JS fails
- **Streaming** = can send parts as they're ready
- **Suspense** = can defer expensive parts
- **Next.js 13+ App Router** = enables these benefits
- **React 18+** = required for server components

</details>

---

### 90. What is Streaming SSR?

<details>
<summary>View Answer</summary>

**Streaming SSR (Server-Side Rendering)**

A React 18 feature that allows the server to send HTML to the browser in chunks (streams) as components become ready, rather than waiting for the entire page to render.

---

## What is Streaming?

**Streaming = Send HTML progressively, not all at once**

### Old SSR (React 17)

```
Server:
1. Fetch ALL data (5 seconds)
2. Render ALL components
3. Send complete HTML
   ↓
Client receives everything at once

Total wait: 5 seconds
```

---

### New Streaming SSR (React 18)

```
Server:
1. Render fast components
2. STREAM HTML immediately (0.1s)
   ↓
Client starts displaying page!

3. Slow component finishes
4. STREAM that HTML (2s later)
   ↓
Client updates page with new content

Time to first content: 0.1s (50x faster!)
```

---

## How It Works

### Traditional SSR

```jsx
// React 17 - renderToString
import { renderToString } from 'react-dom/server';

const html = renderToString(<App />);  // Wait for everything
res.send(html);                         // Send all at once
```

**Problem:** Must wait for slowest component.

---

### Streaming SSR

```jsx
// React 18 - renderToPipeableStream
import { renderToPipeableStream } from 'react-dom/server';

const { pipe } = renderToPipeableStream(<App />, {
  onShellReady() {
    res.setHeader('Content-Type', 'text/html');
    pipe(res);  // Start streaming!
  }
});
```

**Benefit:** Send HTML as it's ready.

---

## Example

### App Structure

```jsx
import { Suspense } from 'react';

function App() {
  return (
    <html>
      <body>
        {/* Fast components - stream immediately */}
        <Header />
        <Nav />
        
        {/* Slow component - wrap in Suspense */}
        <Suspense fallback={<CommentsSkeleton />}>
          <Comments />  {/* Takes 3 seconds to load */}
        </Suspense>
        
        <Footer />
      </body>
    </html>
  );
}
```

---

### Timeline

**0.1 seconds:**
```html
<!-- Server sends this immediately -->
<html>
  <body>
    <header>My Site</header>
    <nav>Home | About | Contact</nav>
    
    <!-- Placeholder while Comments loads -->
    <div class="skeleton">Loading comments...</div>
    
    <footer>© 2024</footer>
  </body>
</html>
```

**User sees page immediately! ✅**

---

**3 seconds later:**

```html
<!-- Server streams this additional HTML -->
<script>
  // React replaces skeleton with actual comments
  $RC("comments-boundary", "<div>...actual comments...</div>");
</script>
```

**Comments appear seamlessly! ✅**

---

## Server Code

### Complete Server Setup

```jsx
// server.js
import express from 'express';
import { renderToPipeableStream } from 'react-dom/server';
import App from './App';

const app = express();

app.get('/', (req, res) => {
  const { pipe, abort } = renderToPipeableStream(
    <App />,
    {
      bootstrapScripts: ['/client.js'],
      
      onShellReady() {
        // Called when initial HTML is ready
        // Send HTTP headers and start streaming
        res.statusCode = 200;
        res.setHeader('Content-Type', 'text/html');
        pipe(res);
      },
      
      onShellError(error) {
        // Called if initial render fails
        res.statusCode = 500;
        res.send('<!doctype html><p>Error</p>');
      },
      
      onAllReady() {
        // Called when everything is rendered
        // Good for crawlers that don't support streaming
      },
      
      onError(error) {
        console.error(error);
      }
    }
  );
  
  // Timeout after 10 seconds
  setTimeout(() => abort(), 10000);
});

app.listen(3000);
```

---

## Client Code

```jsx
// client.js
import { hydrateRoot } from 'react-dom/client';
import App from './App';

// Hydrate the streamed HTML
hydrateRoot(document, <App />);
```

---

## Real-World Example: Blog Post

```jsx
// app/posts/[id]/page.jsx
import { Suspense } from 'react';
import { db } from '@/lib/database';

async function PostPage({ params }) {
  // This loads quickly
  const post = await db.post.findUnique({
    where: { id: params.id }
  });
  
  return (
    <article>
      {/* Stream immediately: */}
      <h1>{post.title}</h1>
      <p className="author">By {post.author}</p>
      <div className="content">{post.content}</div>
      
      {/* Stream later when ready: */}
      <Suspense fallback={<CommentsSkeleton />}>
        <Comments postId={params.id} />
      </Suspense>
      
      {/* Stream even later: */}
      <Suspense fallback={<RecommendationsSkeleton />}>
        <Recommendations postId={params.id} />
      </Suspense>
    </article>
  );
}

// Slow components
async function Comments({ postId }) {
  // Takes 2 seconds
  const comments = await db.comment.findMany({
    where: { postId }
  });
  
  return (
    <div className="comments">
      {comments.map(c => <Comment key={c.id} data={c} />)}
    </div>
  );
}

async function Recommendations({ postId }) {
  // Takes 3 seconds (external API)
  const recs = await fetch(`https://api.example.com/recommend?id=${postId}`)
    .then(r => r.json());
  
  return (
    <div className="recommendations">
      {recs.map(r => <PostCard key={r.id} post={r} />)}
    </div>
  );
}
```

---

### User Experience Timeline

```
0.0s: User requests page
0.1s: ✅ Post title, author, content visible
      Loading placeholders for comments and recommendations
      Page is usable!
      
2.0s: ✅ Comments stream in and replace placeholder
      User can read comments
      
3.0s: ✅ Recommendations stream in and replace placeholder
      Page is complete

Total: 0.1s to useful content (vs 3s with old SSR)
```

---

## Benefits

### 1. Faster First Paint

**Old SSR:**
```
Wait: 5 seconds (blank screen)
Paint: All content at once
```

**Streaming SSR:**
```
Paint: 0.1 seconds (initial content)
Stream: Additional content arrives progressively
```

**50x faster first paint!**

---

### 2. Progressive Enhancement

**Content becomes available as it's ready:**

```
0.1s: Header, navigation ✅
0.5s: Main content ✅
2.0s: Comments ✅
3.0s: Recommendations ✅
```

User can start reading at 0.5s, doesn't wait for everything.

---

### 3. No Blank Screen

**Old SSR:** Blank screen until everything ready  
**Streaming SSR:** Something visible immediately

---

### 4. Resilient to Slow Components

**Old SSR:** One slow component blocks entire page  
**Streaming SSR:** Slow components don't block fast ones

---

### 5. Better Perceived Performance

**Users perceive page as 5-10x faster** because content appears immediately.

---

## Streaming with Suspense

### Multiple Suspense Boundaries

```jsx
function Page() {
  return (
    <div>
      {/* Streams immediately */}
      <Header />
      
      {/* Independent streaming boundaries */}
      <Suspense fallback={<SidebarSkeleton />}>
        <Sidebar />  {/* Streams when ready */}
      </Suspense>
      
      <main>
        <Suspense fallback={<ContentSkeleton />}>
          <MainContent />  {/* Streams when ready */}
        </Suspense>
        
        <Suspense fallback={<CommentsSkeleton />}>
          <Comments />  {/* Streams when ready */}
        </Suspense>
      </main>
      
      {/* Streams immediately */}
      <Footer />
    </div>
  );
}
```

**Each Suspense boundary streams independently!**

---

## Comparison: Old vs New

### Old SSR (renderToString)

```jsx
import { renderToString } from 'react-dom/server';

app.get('/', async (req, res) => {
  // Must wait for ALL async operations
  const html = renderToString(<App />);
  
  // Send complete HTML
  res.send(`
    <!DOCTYPE html>
    <html>
      <body>
        <div id="root">${html}</div>
        <script src="/client.js"></script>
      </body>
    </html>
  `);
});

// Timeline:
// 1. Wait for all data (5s)
// 2. Render all components
// 3. Send HTML
// User waits 5 seconds to see ANYTHING
```

---

### New Streaming SSR (renderToPipeableStream)

```jsx
import { renderToPipeableStream } from 'react-dom/server';

app.get('/', (req, res) => {
  const { pipe } = renderToPipeableStream(
    <App />,
    {
      bootstrapScripts: ['/client.js'],
      onShellReady() {
        res.setHeader('Content-Type', 'text/html');
        pipe(res);  // Start streaming!
      }
    }
  );
});

// Timeline:
// 1. Render fast components (0.1s)
// 2. Stream initial HTML
// User sees page in 0.1 seconds!
// 3. Stream slow components as they finish
```

---

## Advanced: Nested Suspense

```jsx
function BlogPost({ id }) {
  return (
    <article>
      <h1>{post.title}</h1>
      
      {/* Outer Suspense */}
      <Suspense fallback={<SidebarSkeleton />}>
        <Sidebar>
          {/* Nested Suspense */}
          <Suspense fallback={<RecentPostsSkeleton />}>
            <RecentPosts />
          </Suspense>
          
          <Suspense fallback={<TagsSkeleton />}>
            <Tags />
          </Suspense>
        </Sidebar>
      </Suspense>
      
      <Suspense fallback={<CommentsSkeleton />}>
        <Comments postId={id} />
      </Suspense>
    </article>
  );
}
```

**Streaming order:**
1. Title (immediate)
2. Sidebar shell when ready
3. RecentPosts when ready
4. Tags when ready
5. Comments when ready

---

## Streaming with Server Components

**Powerful combination:**

```jsx
// Server Component with streaming
async function Page() {
  return (
    <div>
      {/* Fast Server Component - streams immediately */}
      <Header />
      
      {/* Slow Server Component - streams later */}
      <Suspense fallback={<ProductsSkeleton />}>
        <Products />  {/* Fetches from DB, streams when ready */}
      </Suspense>
      
      {/* Client Component for interactivity */}
      <Cart />  {/* Streams immediately, hydrates when JS loads */}
    </div>
  );
}

async function Products() {
  // This runs on server
  const products = await db.products.findMany();
  
  return (
    <div>
      {products.map(p => <ProductCard key={p.id} product={p} />)}
    </div>
  );
}
```

---

## Error Handling

### Error Boundaries with Streaming

```jsx
import { Suspense } from 'react';
import { ErrorBoundary } from 'react-error-boundary';

function Page() {
  return (
    <div>
      <Header />
      
      <ErrorBoundary fallback={<ErrorMessage />}>
        <Suspense fallback={<Loading />}>
          <SlowComponent />  {/* Might throw error */}
        </Suspense>
      </ErrorBoundary>
      
      <Footer />
    </div>
  );
}

// If SlowComponent errors:
// 1. Header and Footer still render
// 2. Error boundary catches error
// 3. Shows error message in that section only
// 4. Rest of page works fine
```

---

## Best Practices

**1. Use Suspense for slow components**
```jsx
<Suspense fallback={<Skeleton />}>
  <SlowComponent />
</Suspense>
```

**2. Provide good fallback UI**
```jsx
// ✅ Good: Matches layout
<Suspense fallback={<CommentsSkeleton />}>
  <Comments />
</Suspense>

// ❌ Bad: Generic spinner
<Suspense fallback={<div>Loading...</div>}>
  <Comments />
</Suspense>
```

**3. Don't over-suspend**
```jsx
// ❌ Bad: Too granular
<Suspense fallback={<Spinner />}>
  <span>{user.name}</span>
</Suspense>

// ✅ Good: Sensible boundaries
<Suspense fallback={<UserCardSkeleton />}>
  <UserCard user={user} />
</Suspense>
```

**4. Use onShellReady**
```jsx
renderToPipeableStream(<App />, {
  onShellReady() {
    pipe(res);  // Start streaming immediately
  }
});
```

**5. Set timeout**
```jsx
const { pipe, abort } = renderToPipeableStream(...);
setTimeout(abort, 10000);  // Abort after 10s
```

---

## Performance Metrics

### Time to First Byte (TTFB)

```
Old SSR: 2-5 seconds
Streaming SSR: 0.1-0.3 seconds

10-50x improvement!
```

### First Contentful Paint (FCP)

```
Old SSR: 2-5 seconds
Streaming SSR: 0.2-0.5 seconds

10x improvement!
```

### Time to Interactive (TTI)

```
Old SSR: 3-6 seconds
Streaming SSR: 0.5-1.5 seconds

5x improvement!
```

---

## Browser Support

✅ Chrome, Edge, Firefox, Safari  
✅ All modern browsers  
✅ Graceful degradation for old browsers  

---

**Interview Tips:**
- **Streaming SSR** = send HTML in chunks, not all at once
- **renderToPipeableStream** = React 18 streaming API
- **Progressive** = content appears as it's ready
- **Suspense boundaries** = mark what can stream separately
- **No blank screen** = something visible immediately
- **Faster perceived performance** = 5-10x improvement
- **vs old SSR**: don't wait for slowest component
- **onShellReady** = start streaming initial HTML
- **Fallback UI** = shown while component loads
- **Works with Server Components** = powerful combination
- **Multiple boundaries** = different sections stream independently
- **Error resilient** = errors don't break entire page
- **SEO friendly** = search engines see content immediately
- **Graceful degradation** = works even without JavaScript
- **React 18+ only** = not available in React 17
- **Requires Suspense** = wrap slow components
- **Time to First Byte** = 10-50x faster
- **First Contentful Paint** = 10x faster
- **Better Core Web Vitals** = improved all metrics
- **No waterfall** = multiple parts can stream in parallel
- **Nested Suspense** = fine-grained streaming control
- **Client-side** = continues with selective hydration
- **Production ready** = used by major sites
- **Game changer** = revolutionizes SSR performance

</details>
