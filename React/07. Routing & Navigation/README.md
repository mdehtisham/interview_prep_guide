# Routing & Navigation

> Intermediate / Mid-Level (1-3 years)

---

## Questions

61. What is React Router and how does it work?
62. What is the difference between BrowserRouter and HashRouter?
63. How do you implement nested routes?
64. What are route parameters and how do you access them?
65. How do you programmatically navigate in React Router v6?
66. What is the difference between Link and NavLink?
67. How do you protect routes (authentication)?
68. What are loader and action functions in React Router v6?
69. How do you handle 404 pages?
70. What is the useNavigate hook?

---

## Detailed Answers

### 61. What is React Router and how does it work?

<details>
<summary>View Answer</summary>

**React Router**

React Router is a **standard routing library** for React that enables **navigation** between different views/pages in a single-page application (SPA) without full page reloads.

**Core Concept:** Change what the user sees based on the **URL**, without requesting new HTML from the server.

---

## Why React Router?

**Without routing:**
```jsx
function App() {
  const [page, setPage] = useState('home');
  
  return (
    <div>
      <button onClick={() => setPage('home')}>Home</button>
      <button onClick={() => setPage('about')}>About</button>
      
      {page === 'home' && <Home />}
      {page === 'about' && <About />}
    </div>
  );
}

// Problems:
// - No browser back/forward support
// - Can't share URLs
// - No bookmarking
// - Manual state management
```

**With React Router:**
```jsx
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
      </nav>
      
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </BrowserRouter>
  );
}

// ✅ Browser back/forward works
// ✅ URLs are shareable
// ✅ Bookmarking works
// ✅ Automatic state sync with URL
```

---

## Installation

```bash
npm install react-router-dom
```

**Current version:** React Router v6 (major rewrite from v5)

---

## Basic Setup

```jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';

// Page components
function Home() {
  return <h1>Home Page</h1>;
}

function About() {
  return <h1>About Page</h1>;
}

function Contact() {
  return <h1>Contact Page</h1>;
}

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/contact" element={<Contact />} />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

---

## Core Components

### 1. BrowserRouter

**Wraps entire app, provides routing context**

```jsx
import { BrowserRouter } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      {/* Your app */}
    </BrowserRouter>
  );
}
```

**What it does:**
- Listens to URL changes
- Manages browser history
- Provides routing context to children

---

### 2. Routes

**Container for all Route components**

```jsx
import { Routes, Route } from 'react-router-dom';

<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/about" element={<About />} />
</Routes>
```

**What it does:**
- Matches current URL against all routes
- Renders first matching route
- Only one route rendered at a time

---

### 3. Route

**Defines a URL pattern and component to render**

```jsx
<Route path="/about" element={<About />} />
```

**Props:**
- `path`: URL pattern to match
- `element`: Component to render when matched
- `index`: Makes route the default child route

---

### 4. Link

**Navigation component (like `<a>` but doesn't reload page)**

```jsx
import { Link } from 'react-router-dom';

<Link to="/about">About</Link>
<Link to="/contact">Contact</Link>
```

**What it does:**
- Changes URL without page reload
- Updates browser history
- Renders as `<a>` tag in DOM

---

## How It Works

**Flow:**

```
1. User clicks <Link to="/about">
   ↓
2. Link updates browser URL to /about
   ↓
3. BrowserRouter detects URL change
   ↓
4. Routes component checks all routes
   ↓
5. Finds matching route: <Route path="/about" element={<About />} />
   ↓
6. Renders <About /> component
   ↓
7. User sees About page (no reload!)
```

**Behind the scenes:**
- Uses HTML5 History API (`pushState`, `replaceState`)
- Intercepts link clicks
- Prevents default browser navigation
- Updates URL and renders new component

---

## Complete Example

```jsx
import React from 'react';
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';
import './App.css';

// Page Components
function Home() {
  return (
    <div>
      <h1>Home Page</h1>
      <p>Welcome to our website!</p>
    </div>
  );
}

function About() {
  return (
    <div>
      <h1>About Page</h1>
      <p>Learn more about us.</p>
    </div>
  );
}

function Products() {
  return (
    <div>
      <h1>Products Page</h1>
      <p>Browse our products.</p>
    </div>
  );
}

function Contact() {
  return (
    <div>
      <h1>Contact Page</h1>
      <p>Get in touch with us.</p>
    </div>
  );
}

function NotFound() {
  return (
    <div>
      <h1>404 - Page Not Found</h1>
      <p>The page you're looking for doesn't exist.</p>
      <Link to="/">Go Home</Link>
    </div>
  );
}

// Layout with Navigation
function Layout({ children }) {
  return (
    <div>
      <nav className="navbar">
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
        <Link to="/products">Products</Link>
        <Link to="/contact">Contact</Link>
      </nav>
      
      <main className="content">
        {children}
      </main>
      
      <footer>
        <p>&copy; 2024 My Website</p>
      </footer>
    </div>
  );
}

function App() {
  return (
    <BrowserRouter>
      <Layout>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/products" element={<Products />} />
          <Route path="/contact" element={<Contact />} />
          <Route path="*" element={<NotFound />} />  {/* Catch-all route */}
        </Routes>
      </Layout>
    </BrowserRouter>
  );
}

export default App;
```

---

## Key Concepts

### 1. Path Matching

**Exact matching (default in v6):**
```jsx
<Route path="/about" element={<About />} />  // Matches /about only
<Route path="/about/team" element={<Team />} />  // Matches /about/team only
```

**Wildcard matching:**
```jsx
<Route path="*" element={<NotFound />} />  // Matches any unmatched route
```

---

### 2. Dynamic Routes

**URL parameters:**
```jsx
<Route path="/users/:id" element={<UserProfile />} />

// Matches:
// /users/1
// /users/123
// /users/abc
```

**Access parameters:**
```jsx
import { useParams } from 'react-router-dom';

function UserProfile() {
  const { id } = useParams();
  
  return <h1>User Profile: {id}</h1>;
}
```

---

### 3. Nested Routes

**Routes inside routes:**
```jsx
function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/dashboard" element={<Dashboard />}>
          <Route path="profile" element={<Profile />} />  {/* /dashboard/profile */}
          <Route path="settings" element={<Settings />} />  {/* /dashboard/settings */}
        </Route>
      </Routes>
    </BrowserRouter>
  );
}

function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      <nav>
        <Link to="/dashboard/profile">Profile</Link>
        <Link to="/dashboard/settings">Settings</Link>
      </nav>
      
      <Outlet />  {/* Nested routes render here */}
    </div>
  );
}
```

---

### 4. Programmatic Navigation

**Navigate without Link component:**
```jsx
import { useNavigate } from 'react-router-dom';

function LoginForm() {
  const navigate = useNavigate();
  
  const handleSubmit = (e) => {
    e.preventDefault();
    
    // Login logic...
    
    // Navigate to dashboard after login
    navigate('/dashboard');
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input type="text" placeholder="Username" />
      <input type="password" placeholder="Password" />
      <button type="submit">Login</button>
    </form>
  );
}
```

---

## React Router v6 vs v5

**Major changes in v6:**

| Feature | v5 | v6 |
|---------|----|----||
| Route matching | `<Switch>` | `<Routes>` |
| Route element | `component={Home}` | `element={<Home />}` |
| Path matching | Partial by default | Exact by default |
| Nested routes | Manual with `match.url` | Built-in with `<Outlet />` |
| Navigation | `<Redirect>` | `<Navigate>` |
| useHistory | `useHistory()` | `useNavigate()` |
| Route props | Automatic | Use hooks |
| Size | ~20KB | ~10KB |

**v5 syntax:**
```jsx
import { Switch, Route } from 'react-router-dom';

<Switch>
  <Route exact path="/" component={Home} />
  <Route path="/about" component={About} />
</Switch>
```

**v6 syntax:**
```jsx
import { Routes, Route } from 'react-router-dom';

<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/about" element={<About />} />
</Routes>
```

---

## Common Patterns

### Pattern 1: Layout Route

```jsx
function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Layout />}>
          <Route index element={<Home />} />
          <Route path="about" element={<About />} />
          <Route path="contact" element={<Contact />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
}

function Layout() {
  return (
    <div>
      <Header />
      <Outlet />  {/* Child routes render here */}
      <Footer />
    </div>
  );
}
```

---

### Pattern 2: Protected Routes

```jsx
function ProtectedRoute({ children }) {
  const isAuthenticated = useAuth();
  
  if (!isAuthenticated) {
    return <Navigate to="/login" />;
  }
  
  return children;
}

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/login" element={<Login />} />
        <Route
          path="/dashboard"
          element={
            <ProtectedRoute>
              <Dashboard />
            </ProtectedRoute>
          }
        />
      </Routes>
    </BrowserRouter>
  );
}
```

---

### Pattern 3: Loading States

```jsx
import { lazy, Suspense } from 'react';

const Dashboard = lazy(() => import('./Dashboard'));
const Profile = lazy(() => import('./Profile'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<div>Loading...</div>}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/profile" element={<Profile />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

---

## Benefits of React Router

**1. Single Page Application**
- No full page reloads
- Faster navigation
- Better user experience

**2. Browser History**
- Back/forward buttons work
- Bookmark pages
- Share URLs

**3. Declarative Routing**
- Define routes in JSX
- Easy to understand
- Component-based

**4. Code Splitting**
- Load routes on demand
- Smaller initial bundle
- Faster load time

**5. Nested Routes**
- Complex layouts
- Shared components
- Better organization

---

## Under the Hood

**React Router uses:**

1. **HTML5 History API**
   - `pushState()` - Add history entry
   - `replaceState()` - Replace current entry
   - `popstate` event - Detect back/forward

2. **Context API**
   - Provides routing info to components
   - No prop drilling needed

3. **React Hooks**
   - `useNavigate()` - Navigation
   - `useParams()` - URL parameters
   - `useLocation()` - Current location

**Example flow:**
```jsx
// User clicks: <Link to="/about">

1. Link component intercepts click
2. event.preventDefault() - Stop default navigation
3. history.pushState(null, '', '/about') - Update URL
4. React Router context updates
5. Routes component re-renders
6. Matches /about route
7. Renders About component
8. User sees new page (no reload!)
```

---

## Alternatives to React Router

**1. TanStack Router**
- Type-safe routing
- Built for TypeScript
- Advanced features

**2. Next.js (File-based)**
- File system routing
- Server-side rendering
- Opinionated

**3. Remix**
- Full-stack framework
- React Router v6 based
- Server rendering

**4. Reach Router**
- Simpler API
- Accessibility focused
- Merged into React Router v6

**React Router is still the most popular** (~80% market share)

---

**Interview Tips:**
- React Router = **routing library** for React
- Enables **navigation** in SPAs without page reloads
- Uses **HTML5 History API** under the hood
- **BrowserRouter** wraps entire app
- **Routes** container for all routes
- **Route** defines path and component
- **Link** for navigation (like `<a>` but no reload)
- Current version: **React Router v6**
- v6 uses **`<Routes>`** instead of `<Switch>`
- v6 uses **`element={<Component />}`** instead of `component={Component}`
- **Exact matching** by default in v6
- **Nested routes** with `<Outlet />`
- **useNavigate()** for programmatic navigation
- **useParams()** to access URL parameters
- **useLocation()** for current location
- **Path matching** uses patterns like `/users/:id`
- **`*` path** catches all unmatched routes (404)
- Supports **lazy loading** with React.lazy()
- Works with **Suspense** for loading states
- **Context API** provides routing info
- No prop drilling needed
- **Declarative** routing (define routes in JSX)
- Better than manual state management
- Browser **back/forward** works automatically

</details>

---

### 62. What is the difference between BrowserRouter and HashRouter?

<details>
<summary>View Answer</summary>

**BrowserRouter vs HashRouter**

Both are **router implementations** in React Router, but they use different strategies for keeping the UI in sync with the URL.

**Key Difference:**
- **BrowserRouter:** Uses regular URLs (`example.com/about`)
- **HashRouter:** Uses URL hash (`example.com/#/about`)

---

## BrowserRouter

**Uses HTML5 History API**

```jsx
import { BrowserRouter } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </BrowserRouter>
  );
}
```

**URLs look like:**
```
https://example.com/
https://example.com/about
https://example.com/products/123
https://example.com/dashboard/settings
```

**Clean, standard URLs**

---

## HashRouter

**Uses URL hash (#)**

```jsx
import { HashRouter } from 'react-router-dom';

function App() {
  return (
    <HashRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </HashRouter>
  );
}
```

**URLs look like:**
```
https://example.com/#/
https://example.com/#/about
https://example.com/#/products/123
https://example.com/#/dashboard/settings
```

**Hash in every URL**

---

## Key Differences

| Feature | BrowserRouter | HashRouter |
|---------|---------------|------------|
| **URL Format** | `/about` | `/#/about` |
| **Looks Clean** | ✅ Yes | ❌ No (has `#`) |
| **Server Config** | ✅ Required | ❌ Not needed |
| **SEO** | ✅ Good | ⚠️ Limited |
| **Legacy Browsers** | ⚠️ IE9- not supported | ✅ All browsers |
| **Recommended** | ✅ Yes | ❌ Only if needed |
| **API Used** | HTML5 History API | `window.location.hash` |

---

## How They Work

### BrowserRouter

**Uses HTML5 History API:**
```javascript
// When user clicks Link to="/about"

1. history.pushState(null, '', '/about')  // Change URL
2. React Router detects URL change
3. Renders matching component
4. Browser URL: example.com/about
```

**APIs used:**
- `window.history.pushState()`
- `window.history.replaceState()`
- `popstate` event

---

### HashRouter

**Uses URL hash:**
```javascript
// When user clicks Link to="/about"

1. window.location.hash = '#/about'  // Change hash
2. React Router detects hash change
3. Renders matching component
4. Browser URL: example.com/#/about
```

**APIs used:**
- `window.location.hash`
- `hashchange` event

---

## Server Configuration

### BrowserRouter Requires Server Config

**Problem:**
```
User visits: example.com/about
          ↓
Browser requests: GET /about from server
          ↓
Server: 404 Not Found (only index.html exists)
```

**Solution: Configure server to serve index.html for all routes**

**Apache (.htaccess):**
```apache
<IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteBase /
  RewriteRule ^index\.html$ - [L]
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteRule . /index.html [L]
</IfModule>
```

**Nginx:**
```nginx
location / {
  try_files $uri $uri/ /index.html;
}
```

**Express.js:**
```javascript
const express = require('express');
const path = require('path');
const app = express();

app.use(express.static(path.join(__dirname, 'build')));

// All routes serve index.html
app.get('/*', (req, res) => {
  res.sendFile(path.join(__dirname, 'build', 'index.html'));
});

app.listen(3000);
```

**Netlify (_redirects):**
```
/*    /index.html   200
```

**Vercel (vercel.json):**
```json
{
  "rewrites": [
    { "source": "/(.*)", "destination": "/" }
  ]
}
```

---

### HashRouter Doesn't Need Server Config

**Why:**
```
User visits: example.com/#/about
          ↓
Browser requests: GET / from server
          ↓
Server: Returns index.html ✅
          ↓
Browser: Reads hash (#/about)
          ↓
React Router: Renders About component
```

**The hash part never goes to the server!**
- `example.com/#/about` → Server sees `example.com/`
- Hash is client-side only
- No server configuration needed

---

## When to Use Each

### Use BrowserRouter (Recommended)

**✅ When:**
- Building modern web apps
- Have control over server configuration
- Want clean URLs
- SEO matters
- Using hosting with routing support (Netlify, Vercel)

**Example use cases:**
- Production web applications
- E-commerce sites
- SaaS applications
- Public websites
- Apps that need SEO

```jsx
import { BrowserRouter } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      {/* Your routes */}
    </BrowserRouter>
  );
}
```

---

### Use HashRouter

**✅ When:**
- Cannot configure server
- Deploying to static file server
- Building electron apps
- Testing locally without server
- Supporting very old browsers

**Example use cases:**
- Electron desktop apps
- Cordova/PhoneGap mobile apps
- GitHub Pages (without custom domain)
- Local HTML file (file:// protocol)
- Legacy browser support needed

```jsx
import { HashRouter } from 'react-router-dom';

function App() {
  return (
    <HashRouter>
      {/* Your routes */}
    </HashRouter>
  );
}
```

---

## SEO Implications

### BrowserRouter

**✅ Good for SEO:**
```html
<!-- Search engine sees: -->
<url>https://example.com/products/laptop</url>

<!-- Clean URL, properly indexed -->
```

**Benefits:**
- Clean URLs in search results
- Proper crawling
- Better ranking
- Social media sharing (nice previews)

---

### HashRouter

**⚠️ Limited SEO:**
```html
<!-- Search engine sees: -->
<url>https://example.com/#/products/laptop</url>

<!-- Hash ignored by some crawlers -->
```

**Issues:**
- Some crawlers ignore hash
- Duplicate content issues
- Poor social media previews
- Analytics complications

**Modern crawlers (Google) can handle hashes, but BrowserRouter is still better**

---

## Complete Comparison Example

```jsx
import { BrowserRouter, HashRouter, Routes, Route, Link } from 'react-router-dom';

// Same components work with both routers
function Home() {
  return <h1>Home</h1>;
}

function About() {
  return <h1>About</h1>;
}

function Products() {
  return <h1>Products</h1>;
}

// Using BrowserRouter
function AppWithBrowserRouter() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
        <Link to="/products">Products</Link>
      </nav>
      
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/products" element={<Products />} />
      </Routes>
    </BrowserRouter>
  );
}
// URLs: /  /about  /products

// Using HashRouter
function AppWithHashRouter() {
  return (
    <HashRouter>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
        <Link to="/products">Products</Link>
      </nav>
      
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/products" element={<Products />} />
      </Routes>
    </HashRouter>
  );
}
// URLs: /#/  /#/about  /#/products
```

**Everything else is identical!**
- Same components
- Same routes
- Same Links
- Same hooks
- Just swap the router

---

## Browser Support

### BrowserRouter

**Requires HTML5 History API:**
- ✅ Chrome 5+
- ✅ Firefox 4+
- ✅ Safari 5+
- ✅ Edge (all versions)
- ✅ IE 10+
- ❌ IE 9 and below

---

### HashRouter

**Works everywhere:**
- ✅ All modern browsers
- ✅ IE 9 and below
- ✅ Very old browsers

**Hash has been around forever**

---

## GitHub Pages Example

**Problem:** GitHub Pages doesn't support server-side routing

**Solution 1: Use HashRouter**
```jsx
import { HashRouter } from 'react-router-dom';

function App() {
  return (
    <HashRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </HashRouter>
  );
}

// URLs will be:
// https://username.github.io/repo/#/
// https://username.github.io/repo/#/about
```

**Solution 2: Use BrowserRouter with workaround**

Create `404.html` that redirects to `index.html`:
```html
<!DOCTYPE html>
<html>
  <head>
    <script>
      // Redirect to index.html with path as query parameter
      const path = window.location.pathname.slice(1);
      window.location.replace('/' + '?path=' + path);
    </script>
  </head>
  <body></body>
</html>
```

---

## Migration Between Routers

**Switching from HashRouter to BrowserRouter:**

```jsx
// Before
import { HashRouter } from 'react-router-dom';

function App() {
  return (
    <HashRouter>
      {/* routes */}
    </HashRouter>
  );
}

// After (only change one line!)
import { BrowserRouter } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      {/* routes - unchanged */}
    </BrowserRouter>
  );
}

// Don't forget to configure server!
```

**All other code stays the same!**

---

## Common Pitfalls

### Pitfall 1: Using BrowserRouter without server config

```
Deploy app with BrowserRouter
  ↓
Visit example.com/about directly
  ↓
404 Error!
```

**Fix:** Configure server or use HashRouter

---

### Pitfall 2: Mixing routers

```jsx
// ❌ Don't do this
<BrowserRouter>
  <HashRouter>
    {/* routes */}
  </HashRouter>
</BrowserRouter>
```

**Use only one router per app**

---

### Pitfall 3: Not using basename

**If app is in subdirectory:**

```jsx
// Wrong - routes won't work
<BrowserRouter>
  <Routes>
    <Route path="/" element={<Home />} />
  </Routes>
</BrowserRouter>
// Deployed at: example.com/my-app/
// Won't find routes!

// Correct - use basename
<BrowserRouter basename="/my-app">
  <Routes>
    <Route path="/" element={<Home />} />  {/* Matches /my-app/ */}
  </Routes>
</BrowserRouter>
```

**Same for HashRouter:**
```jsx
<HashRouter basename="/my-app">
  {/* routes */}
</HashRouter>
```

---

## Best Practices

**1. Default to BrowserRouter**
```jsx
// ✅ Good - modern, clean URLs
<BrowserRouter>
  {/* routes */}
</BrowserRouter>
```

**2. Use HashRouter only when needed**
```jsx
// ✅ Good - when you can't configure server
<HashRouter>
  {/* routes */}
</HashRouter>
```

**3. Configure server properly**
- Always set up fallback to index.html
- Test all routes after deployment

**4. Use basename for subdirectories**
```jsx
<BrowserRouter basename="/my-app">
  {/* routes */}
</BrowserRouter>
```

---

## Quick Decision Guide

**Choose BrowserRouter if:**
- ✅ Modern app
- ✅ Can configure server
- ✅ Want clean URLs
- ✅ SEO matters
- ✅ Using Netlify/Vercel/AWS Amplify

**Choose HashRouter if:**
- ✅ Cannot configure server
- ✅ Static file hosting
- ✅ GitHub Pages
- ✅ Electron app
- ✅ Local testing
- ✅ Very old browser support

**99% of the time: Use BrowserRouter**

---

**Interview Tips:**
- **BrowserRouter** uses HTML5 History API
- **HashRouter** uses URL hash
- BrowserRouter URLs: `/about` (clean)
- HashRouter URLs: `/#/about` (has hash)
- BrowserRouter **requires server config**
- HashRouter **works without server config**
- Hash never sent to server
- BrowserRouter better for **SEO**
- HashRouter works in **all browsers** (even IE9-)
- BrowserRouter needs **IE10+**
- **Recommended: BrowserRouter** for modern apps
- Use HashRouter for **Electron**, **GitHub Pages**, **static hosting**
- Can **switch between them** by changing one line
- All other code stays the same
- **basename** prop for subdirectories
- Server must return `index.html` for all routes
- Configure with `.htaccess`, `nginx.conf`, etc.
- **Netlify** and **Vercel** support BrowserRouter out of the box
- HashRouter good for **local testing**
- BrowserRouter uses `pushState()` and `popstate`
- HashRouter uses `window.location.hash`
- Clean URLs **better for sharing**
- Hash URLs **ugly but functional**
- Modern standard: **BrowserRouter**

</details>

---

### 63. How do you implement nested routes?

<details>
<summary>View Answer</summary>

**Nested Routes**

Nested routes allow you to **render routes inside other routes**, creating a **parent-child relationship** where the parent component stays visible while child routes change.

**Common use case:** Layouts with shared UI (header, sidebar) where only content area changes.

---

## Basic Concept

**Without nested routes:**
```jsx
function App() {
  return (
    <Routes>
      <Route path="/dashboard" element={<Dashboard />} />
      <Route path="/dashboard/profile" element={<DashboardProfile />} />
      <Route path="/dashboard/settings" element={<DashboardSettings />} />
    </Routes>
  );
}

// Problem: Need to duplicate Dashboard layout in every component
```

**With nested routes:**
```jsx
function App() {
  return (
    <Routes>
      <Route path="/dashboard" element={<Dashboard />}>
        <Route path="profile" element={<Profile />} />
        <Route path="settings" element={<Settings />} />
      </Route>
    </Routes>
  );
}

// Dashboard stays visible, only Profile/Settings swap out
```

---

## Implementation in React Router v6

### Step 1: Define Nested Routes

```jsx
import { Routes, Route } from 'react-router-dom';

function App() {
  return (
    <Routes>
      {/* Parent route */}
      <Route path="/dashboard" element={<DashboardLayout />}>
        {/* Child routes */}
        <Route path="profile" element={<Profile />} />
        <Route path="settings" element={<Settings />} />
        <Route path="analytics" element={<Analytics />} />
      </Route>
    </Routes>
  );
}
```

**URLs:**
- `/dashboard/profile` → Shows DashboardLayout + Profile
- `/dashboard/settings` → Shows DashboardLayout + Settings
- `/dashboard/analytics` → Shows DashboardLayout + Analytics

---

### Step 2: Use `<Outlet />` in Parent

**The `<Outlet />` component renders the matched child route**

```jsx
import { Outlet, Link } from 'react-router-dom';

function DashboardLayout() {
  return (
    <div className="dashboard">
      <header>
        <h1>Dashboard</h1>
      </header>
      
      <aside>
        <nav>
          <Link to="/dashboard/profile">Profile</Link>
          <Link to="/dashboard/settings">Settings</Link>
          <Link to="/dashboard/analytics">Analytics</Link>
        </nav>
      </aside>
      
      <main>
        {/* Child routes render here */}
        <Outlet />
      </main>
      
      <footer>
        <p>&copy; 2024 Dashboard</p>
      </footer>
    </div>
  );
}
```

**Flow:**
```
User visits: /dashboard/profile
     ↓
Renders: <DashboardLayout />
     ↓
<Outlet /> replaced with: <Profile />
     ↓
Result: Dashboard layout with Profile content
```

---

### Step 3: Create Child Components

```jsx
function Profile() {
  return (
    <div>
      <h2>Profile</h2>
      <p>User profile information...</p>
    </div>
  );
}

function Settings() {
  return (
    <div>
      <h2>Settings</h2>
      <p>User settings...</p>
    </div>
  );
}

function Analytics() {
  return (
    <div>
      <h2>Analytics</h2>
      <p>Analytics data...</p>
    </div>
  );
}
```

---

## Complete Example

```jsx
import React from 'react';
import { BrowserRouter, Routes, Route, Link, Outlet } from 'react-router-dom';
import './App.css';

// Main App
function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        
        {/* Nested routes */}
        <Route path="/dashboard" element={<DashboardLayout />}>
          <Route index element={<DashboardHome />} />  {/* Default child */}
          <Route path="profile" element={<Profile />} />
          <Route path="settings" element={<Settings />} />
          <Route path="analytics" element={<Analytics />} />
        </Route>
        
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}

// Home Page
function Home() {
  return (
    <div>
      <h1>Home Page</h1>
      <Link to="/dashboard">Go to Dashboard</Link>
    </div>
  );
}

// Dashboard Layout (Parent)
function DashboardLayout() {
  return (
    <div className="dashboard-layout">
      <header className="dashboard-header">
        <h1>Dashboard</h1>
        <Link to="/">Home</Link>
      </header>
      
      <div className="dashboard-container">
        <aside className="dashboard-sidebar">
          <nav>
            <Link to="/dashboard">Dashboard Home</Link>
            <Link to="/dashboard/profile">Profile</Link>
            <Link to="/dashboard/settings">Settings</Link>
            <Link to="/dashboard/analytics">Analytics</Link>
          </nav>
        </aside>
        
        <main className="dashboard-content">
          {/* Child routes render here */}
          <Outlet />
        </main>
      </div>
      
      <footer className="dashboard-footer">
        <p>&copy; 2024 My App</p>
      </footer>
    </div>
  );
}

// Dashboard Home (Default child)
function DashboardHome() {
  return (
    <div>
      <h2>Dashboard Home</h2>
      <p>Welcome to your dashboard!</p>
    </div>
  );
}

// Child Components
function Profile() {
  return (
    <div>
      <h2>Profile</h2>
      <form>
        <label>
          Name: <input type="text" />
        </label>
        <label>
          Email: <input type="email" />
        </label>
        <button type="submit">Save</button>
      </form>
    </div>
  );
}

function Settings() {
  return (
    <div>
      <h2>Settings</h2>
      <ul>
        <li><input type="checkbox" /> Enable notifications</li>
        <li><input type="checkbox" /> Dark mode</li>
        <li><input type="checkbox" /> Auto-save</li>
      </ul>
    </div>
  );
}

function Analytics() {
  return (
    <div>
      <h2>Analytics</h2>
      <div className="stats">
        <div>Total Users: 1,234</div>
        <div>Active Users: 567</div>
        <div>Revenue: $12,345</div>
      </div>
    </div>
  );
}

function NotFound() {
  return <h1>404 - Not Found</h1>;
}

export default App;
```

---

## Index Routes

**Default child route when parent path matches exactly**

```jsx
<Route path="/dashboard" element={<DashboardLayout />}>
  <Route index element={<DashboardHome />} />  {/* /dashboard */}
  <Route path="profile" element={<Profile />} />  {/* /dashboard/profile */}
</Route>
```

**Without `index`:**
- Visit `/dashboard` → Shows DashboardLayout but `<Outlet />` is empty

**With `index`:**
- Visit `/dashboard` → Shows DashboardLayout + DashboardHome

---

## Relative Paths

**Child routes can use relative paths:**

```jsx
<Route path="/dashboard" element={<DashboardLayout />}>
  {/* Relative paths (no leading /) */}
  <Route path="profile" element={<Profile />} />  {/* /dashboard/profile */}
  <Route path="settings" element={<Settings />} />  {/* /dashboard/settings */}
</Route>
```

**Also works in Links:**
```jsx
function DashboardLayout() {
  return (
    <div>
      <nav>
        {/* Relative to current route */}
        <Link to="profile">Profile</Link>  {/* Goes to /dashboard/profile */}
        <Link to="settings">Settings</Link>  {/* Goes to /dashboard/settings */}
        
        {/* Absolute paths also work */}
        <Link to="/dashboard/profile">Profile</Link>
      </nav>
      <Outlet />
    </div>
  );
}
```

---

## Multiple Levels of Nesting

**Routes can nest infinitely:**

```jsx
function App() {
  return (
    <Routes>
      {/* Level 1 */}
      <Route path="/admin" element={<AdminLayout />}>
        
        {/* Level 2 */}
        <Route path="users" element={<UsersLayout />}>
          
          {/* Level 3 */}
          <Route index element={<UsersList />} />  {/* /admin/users */}
          <Route path=":id" element={<UserDetails />} />  {/* /admin/users/123 */}
          <Route path=":id/edit" element={<UserEdit />} />  {/* /admin/users/123/edit */}
        </Route>
        
        <Route path="products" element={<ProductsLayout />}>
          <Route index element={<ProductsList />} />
          <Route path=":id" element={<ProductDetails />} />
        </Route>
      </Route>
    </Routes>
  );
}

// AdminLayout renders <Outlet />
// UsersLayout also renders <Outlet />
// Results in nested layouts
```

**URL structure:**
```
/admin/users          → AdminLayout > UsersLayout > UsersList
/admin/users/123      → AdminLayout > UsersLayout > UserDetails
/admin/users/123/edit → AdminLayout > UsersLayout > UserEdit
/admin/products       → AdminLayout > ProductsLayout > ProductsList
```

---

## Shared Layouts Example

**Common pattern: Different layouts for different sections**

```jsx
function App() {
  return (
    <BrowserRouter>
      <Routes>
        {/* Public routes with PublicLayout */}
        <Route path="/" element={<PublicLayout />}>
          <Route index element={<Home />} />
          <Route path="about" element={<About />} />
          <Route path="contact" element={<Contact />} />
        </Route>
        
        {/* Auth routes with AuthLayout */}
        <Route path="/auth" element={<AuthLayout />}>
          <Route path="login" element={<Login />} />
          <Route path="register" element={<Register />} />
          <Route path="forgot-password" element={<ForgotPassword />} />
        </Route>
        
        {/* Dashboard routes with DashboardLayout */}
        <Route path="/dashboard" element={<DashboardLayout />}>
          <Route index element={<DashboardHome />} />
          <Route path="profile" element={<Profile />} />
          <Route path="settings" element={<Settings />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
}

// Each layout has its own style/structure
function PublicLayout() {
  return (
    <div className="public-layout">
      <header>Public Header</header>
      <Outlet />
      <footer>Public Footer</footer>
    </div>
  );
}

function AuthLayout() {
  return (
    <div className="auth-layout">
      <div className="auth-box">
        <Outlet />
      </div>
    </div>
  );
}

function DashboardLayout() {
  return (
    <div className="dashboard-layout">
      <Sidebar />
      <main>
        <Outlet />
      </main>
    </div>
  );
}
```

---

## Passing Data to Nested Routes

**Option 1: Context API**

```jsx
import { createContext, useContext } from 'react';

const DashboardContext = createContext();

function DashboardLayout() {
  const [user, setUser] = useState({ name: 'John Doe' });
  
  return (
    <DashboardContext.Provider value={{ user, setUser }}>
      <div>
        <h1>Dashboard</h1>
        <Outlet />  {/* Child routes can access context */}
      </div>
    </DashboardContext.Provider>
  );
}

function Profile() {
  const { user } = useContext(DashboardContext);
  
  return <h2>Profile: {user.name}</h2>;
}
```

---

**Option 2: Outlet Context**

```jsx
function DashboardLayout() {
  const [user, setUser] = useState({ name: 'John Doe' });
  
  return (
    <div>
      <h1>Dashboard</h1>
      <Outlet context={{ user, setUser }} />  {/* Pass data to Outlet */}
    </div>
  );
}

function Profile() {
  const { user } = useOutletContext();  // Access passed data
  
  return <h2>Profile: {user.name}</h2>;
}
```

---

## React Router v5 vs v6

**v5 nested routes (complex):**
```jsx
function App() {
  return (
    <Switch>
      <Route path="/dashboard">
        <Dashboard />
      </Route>
    </Switch>
  );
}

function Dashboard() {
  const { path } = useRouteMatch();
  
  return (
    <div>
      <nav>
        <Link to={`${path}/profile`}>Profile</Link>
      </nav>
      
      <Switch>
        <Route path={`${path}/profile`}>
          <Profile />
        </Route>
      </Switch>
    </div>
  );
}
```

**v6 nested routes (simpler):**
```jsx
function App() {
  return (
    <Routes>
      <Route path="/dashboard" element={<Dashboard />}>
        <Route path="profile" element={<Profile />} />
      </Route>
    </Routes>
  );
}

function Dashboard() {
  return (
    <div>
      <nav>
        <Link to="/dashboard/profile">Profile</Link>
      </nav>
      <Outlet />  {/* Much simpler! */}
    </div>
  );
}
```

---

## Common Patterns

### Pattern 1: Admin Panel

```jsx
<Route path="/admin" element={<AdminLayout />}>
  <Route index element={<AdminDashboard />} />
  <Route path="users" element={<UsersList />} />
  <Route path="users/:id" element={<UserEdit />} />
  <Route path="products" element={<ProductsList />} />
  <Route path="products/:id" element={<ProductEdit />} />
  <Route path="settings" element={<Settings />} />
</Route>
```

---

### Pattern 2: E-commerce Product Categories

```jsx
<Route path="/shop" element={<ShopLayout />}>
  <Route index element={<AllProducts />} />
  <Route path="electronics" element={<Electronics />} />
  <Route path="clothing" element={<Clothing />} />
  <Route path="books" element={<Books />} />
</Route>
```

---

### Pattern 3: Settings with Tabs

```jsx
<Route path="/settings" element={<SettingsLayout />}>
  <Route index element={<Navigate to="profile" replace />} />
  <Route path="profile" element={<ProfileSettings />} />
  <Route path="account" element={<AccountSettings />} />
  <Route path="security" element={<SecuritySettings />} />
  <Route path="notifications" element={<NotificationSettings />} />
</Route>

function SettingsLayout() {
  return (
    <div>
      <h1>Settings</h1>
      <nav className="tabs">
        <NavLink to="/settings/profile">Profile</NavLink>
        <NavLink to="/settings/account">Account</NavLink>
        <NavLink to="/settings/security">Security</NavLink>
        <NavLink to="/settings/notifications">Notifications</NavLink>
      </nav>
      <Outlet />
    </div>
  );
}
```

---

### Pattern 4: Protected Dashboard

```jsx
function App() {
  return (
    <Routes>
      <Route path="/login" element={<Login />} />
      
      <Route
        path="/dashboard"
        element={
          <ProtectedRoute>
            <DashboardLayout />
          </ProtectedRoute>
        }
      >
        <Route index element={<DashboardHome />} />
        <Route path="profile" element={<Profile />} />
        <Route path="settings" element={<Settings />} />
      </Route>
    </Routes>
  );
}

function ProtectedRoute({ children }) {
  const isAuthenticated = useAuth();
  return isAuthenticated ? children : <Navigate to="/login" />;
}
```

---

## Styling Active Links

**Use NavLink in nested navigation:**

```jsx
import { NavLink } from 'react-router-dom';

function DashboardLayout() {
  return (
    <div>
      <nav>
        <NavLink
          to="/dashboard/profile"
          className={({ isActive }) => isActive ? 'active' : ''}
        >
          Profile
        </NavLink>
        <NavLink
          to="/dashboard/settings"
          className={({ isActive }) => isActive ? 'active' : ''}
        >
          Settings
        </NavLink>
      </nav>
      <Outlet />
    </div>
  );
}
```

**CSS:**
```css
nav a {
  padding: 10px;
  text-decoration: none;
  color: #333;
}

nav a.active {
  color: #007bff;
  border-bottom: 2px solid #007bff;
  font-weight: bold;
}
```

---

## Error Boundaries with Nested Routes

```jsx
import { ErrorBoundary } from 'react-error-boundary';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route
          path="/dashboard"
          element={
            <ErrorBoundary fallback={<ErrorPage />}>
              <DashboardLayout />
            </ErrorBoundary>
          }
        >
          <Route path="profile" element={<Profile />} />
          <Route path="settings" element={<Settings />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
}
```

---

## Best Practices

**1. Use `<Outlet />` in parent components**
```jsx
function ParentLayout() {
  return (
    <div>
      <Header />
      <Outlet />  {/* Required for nested routes */}
      <Footer />
    </div>
  );
}
```

**2. Use `index` route for default child**
```jsx
<Route path="/dashboard" element={<DashboardLayout />}>
  <Route index element={<DashboardHome />} />  {/* Default */}
  <Route path="profile" element={<Profile />} />
</Route>
```

**3. Keep nested routes shallow**
```jsx
// ✅ Good - 2 levels
/dashboard/profile

// ⚠️ Avoid - too deep
/dashboard/settings/account/security/two-factor/setup
```

**4. Use relative paths in nested routes**
```jsx
<Route path="/dashboard" element={<Dashboard />}>
  <Route path="profile" element={<Profile />} />  {/* Not /profile */}
</Route>
```

**5. Share common layouts**
```jsx
// ✅ Good - DRY
<Route path="/admin" element={<AdminLayout />}>
  <Route path="users" element={<Users />} />
  <Route path="products" element={<Products />} />
</Route>

// ❌ Bad - Repeated layout
<Route path="/admin/users" element={<AdminLayoutWithUsers />} />
<Route path="/admin/products" element={<AdminLayoutWithProducts />} />
```

---

**Interview Tips:**
- Nested routes = **routes inside routes**
- Create **parent-child relationship**
- Parent stays visible, child swaps out
- Use **`<Outlet />`** in parent to render child
- Define with **nested `<Route>` elements**
- Child paths are **relative** to parent
- **`index` route** = default child when parent matches exactly
- Multiple levels of nesting supported
- Common for **layouts** (header, sidebar, footer)
- React Router v6 **simpler** than v5
- v6 uses `<Outlet />`, v5 used manual matching
- Pass data with **Context** or **`useOutletContext()`**
- Use **NavLink** for active styling
- **Relative Links** work: `<Link to="profile">`
- Also supports **absolute paths**: `<Link to="/dashboard/profile">`
- **404 handling** at any level
- Works with **protected routes**
- Supports **lazy loading**
- **Error boundaries** can wrap layouts
- Path matching is **exact** by default in v6
- **Wildcards** work in nested routes too
- Keep nesting **shallow** (2-3 levels max)
- Share **common layouts** (DRY principle)
- Great for **admin panels**, **dashboards**, **settings pages**

</details>

---

### 64. What are route parameters and how do you access them?

<details>
<summary>View Answer</summary>

**Route Parameters**

Route parameters (or URL parameters) are **dynamic segments** in the URL that capture values and make them available to your components.

**Example:**
```
/users/123       → id = 123
/posts/hello     → slug = "hello"
/blog/2024/12    → year = 2024, month = 12
```

**Why use them:**
- Pass data through URL
- Shareable links
- Bookmarkable pages
- SEO-friendly
- Deep linking

---

## Defining Route Parameters

**Use `:paramName` syntax in path:**

```jsx
import { Routes, Route } from 'react-router-dom';

<Routes>
  {/* Single parameter */}
  <Route path="/users/:id" element={<UserProfile />} />
  
  {/* Multiple parameters */}
  <Route path="/blog/:year/:month/:day" element={<BlogPost />} />
  
  {/* Parameter with additional segments */}
  <Route path="/products/:id/reviews" element={<ProductReviews />} />
</Routes>
```

**Matches:**
```
/users/:id          → /users/123, /users/abc, /users/john
/blog/:year/:month  → /blog/2024/12, /blog/2023/01
/products/:id/reviews → /products/42/reviews
```

---

## Accessing Parameters with `useParams()`

**Hook that returns an object with all parameters:**

```jsx
import { useParams } from 'react-router-dom';

function UserProfile() {
  const { id } = useParams();
  
  return <h1>User ID: {id}</h1>;
}

// Visit /users/123 → Shows: User ID: 123
// Visit /users/456 → Shows: User ID: 456
```

**All parameters are strings!**
```jsx
function UserProfile() {
  const { id } = useParams();
  
  console.log(id);          // "123" (string)
  console.log(typeof id);   // "string"
  
  // Convert to number if needed
  const userId = Number(id);
  console.log(userId);      // 123 (number)
  
  return <h1>User {userId}</h1>;
}
```

---

## Complete Example

```jsx
import React from 'react';
import { BrowserRouter, Routes, Route, Link, useParams } from 'react-router-dom';

// Main App
function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/users/1">User 1</Link>
        <Link to="/users/2">User 2</Link>
        <Link to="/posts/hello-world">Post</Link>
      </nav>
      
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/users/:id" element={<UserProfile />} />
        <Route path="/posts/:slug" element={<Post />} />
        <Route path="/blog/:year/:month/:day" element={<BlogPost />} />
      </Routes>
    </BrowserRouter>
  );
}

// Home Page
function Home() {
  return <h1>Home Page</h1>;
}

// User Profile (single parameter)
function UserProfile() {
  const { id } = useParams();
  
  return (
    <div>
      <h1>User Profile</h1>
      <p>User ID: {id}</p>
    </div>
  );
}

// Post (slug parameter)
function Post() {
  const { slug } = useParams();
  
  return (
    <div>
      <h1>Post</h1>
      <p>Slug: {slug}</p>
    </div>
  );
}

// Blog Post (multiple parameters)
function BlogPost() {
  const { year, month, day } = useParams();
  
  return (
    <div>
      <h1>Blog Post</h1>
      <p>Date: {year}-{month}-{day}</p>
    </div>
  );
}

export default App;
```

---

## Multiple Parameters

```jsx
import { useParams } from 'react-router-dom';

// Route definition
<Route path="/shop/:category/:productId" element={<Product />} />

// Component
function Product() {
  const { category, productId } = useParams();
  
  return (
    <div>
      <h1>Product Details</h1>
      <p>Category: {category}</p>
      <p>Product ID: {productId}</p>
    </div>
  );
}

// Visit /shop/electronics/123
// Shows: Category: electronics, Product ID: 123
```

---

## Optional Parameters

**React Router v6 doesn't have built-in optional params, use multiple routes:**

```jsx
// Want: /users or /users/123

<Routes>
  <Route path="/users" element={<UsersList />} />
  <Route path="/users/:id" element={<UserProfile />} />
</Routes>
```

**Or handle in component:**
```jsx
<Route path="/users/:id?" element={<Users />} />  {/* v6 doesn't support */}

// Instead:
<Route path="/users/*" element={<Users />} />

function Users() {
  const params = useParams();
  const id = params['*'];  // Get wildcard value
  
  if (id) {
    return <UserProfile id={id} />;
  }
  return <UsersList />;
}
```

---

## Fetch Data with Parameters

**Common pattern: Fetch data based on route parameter**

```jsx
import { useParams } from 'react-router-dom';
import { useState, useEffect } from 'react';

function UserProfile() {
  const { id } = useParams();
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    // Fetch user data when id changes
    async function fetchUser() {
      try {
        setLoading(true);
        const response = await fetch(`https://api.example.com/users/${id}`);
        if (!response.ok) throw new Error('User not found');
        const data = await response.json();
        setUser(data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    }
    
    fetchUser();
  }, [id]);  // Re-fetch when id changes
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return <div>User not found</div>;
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>Email: {user.email}</p>
      <p>ID: {id}</p>
    </div>
  );
}
```

---

## Type Conversion

**Parameters are always strings, convert as needed:**

```jsx
import { useParams } from 'react-router-dom';

function Product() {
  const { id } = useParams();
  
  // Convert to number
  const productId = Number(id);
  
  // Or use parseInt
  const productId2 = parseInt(id, 10);
  
  // Validate
  if (isNaN(productId)) {
    return <div>Invalid product ID</div>;
  }
  
  return <div>Product ID: {productId}</div>;
}
```

---

## Validation

**Validate parameters before using:**

```jsx
import { useParams, Navigate } from 'react-router-dom';

function UserProfile() {
  const { id } = useParams();
  
  // Validate ID is a number
  if (!/^\d+$/.test(id)) {
    return <Navigate to="/404" replace />;
  }
  
  // Validate ID is within range
  const userId = Number(id);
  if (userId < 1 || userId > 1000) {
    return <div>User ID must be between 1 and 1000</div>;
  }
  
  return <div>Valid User ID: {userId}</div>;
}
```

---

## Wildcard Parameters

**Catch all remaining path segments:**

```jsx
<Route path="/docs/*" element={<Documentation />} />

function Documentation() {
  const params = useParams();
  const path = params['*'];  // Everything after /docs/
  
  return <div>Documentation path: {path}</div>;
}

// Visit /docs/api/authentication
// Shows: Documentation path: api/authentication
```

---

## Nested Route Parameters

**Parent and child routes can both have parameters:**

```jsx
function App() {
  return (
    <Routes>
      <Route path="/users/:userId" element={<UserLayout />}>
        <Route path="posts/:postId" element={<UserPost />} />
        <Route path="comments/:commentId" element={<UserComment />} />
      </Route>
    </Routes>
  );
}

function UserLayout() {
  const { userId } = useParams();
  
  return (
    <div>
      <h1>User {userId}</h1>
      <Outlet />  {/* Child routes render here */}
    </div>
  );
}

function UserPost() {
  const { userId, postId } = useParams();  // Access both!
  
  return (
    <div>
      <h2>Post {postId} by User {userId}</h2>
    </div>
  );
}

// Visit /users/123/posts/456
// Shows: User 123, Post 456 by User 123
```

---

## Real-World Example: E-commerce Product Page

```jsx
import React, { useState, useEffect } from 'react';
import { useParams, Link, Navigate } from 'react-router-dom';

// Route definition
<Route path="/products/:productId" element={<ProductPage />} />

// Product Page Component
function ProductPage() {
  const { productId } = useParams();
  const [product, setProduct] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    async function loadProduct() {
      try {
        setLoading(true);
        setError(null);
        
        const response = await fetch(`/api/products/${productId}`);
        
        if (response.status === 404) {
          setError('Product not found');
          return;
        }
        
        if (!response.ok) {
          throw new Error('Failed to load product');
        }
        
        const data = await response.json();
        setProduct(data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    }
    
    loadProduct();
  }, [productId]);  // Reload when productId changes
  
  // Loading state
  if (loading) {
    return (
      <div className="loading">
        <p>Loading product...</p>
      </div>
    );
  }
  
  // Error state
  if (error) {
    return (
      <div className="error">
        <h1>Error</h1>
        <p>{error}</p>
        <Link to="/products">Back to Products</Link>
      </div>
    );
  }
  
  // Product not found
  if (!product) {
    return <Navigate to="/404" replace />;
  }
  
  // Success - show product
  return (
    <div className="product-page">
      <Link to="/products">&larr; Back to Products</Link>
      
      <div className="product-details">
        <img src={product.image} alt={product.name} />
        
        <div className="product-info">
          <h1>{product.name}</h1>
          <p className="price">${product.price}</p>
          <p className="description">{product.description}</p>
          
          <button>Add to Cart</button>
          
          <div className="meta">
            <p>Product ID: {productId}</p>
            <p>Category: {product.category}</p>
            <p>In Stock: {product.stock}</p>
          </div>
        </div>
      </div>
      
      <div className="reviews">
        <h2>Customer Reviews</h2>
        {/* Reviews component */}
      </div>
      
      <div className="related-products">
        <h2>Related Products</h2>
        {/* Related products */}
      </div>
    </div>
  );
}
```

---

## Best Practices

**1. Validate parameters**
```jsx
const { id } = useParams();

if (!/^\d+$/.test(id)) {
  return <Navigate to="/404" />;
}
```

**2. Convert types appropriately**
```jsx
const { id } = useParams();
const userId = Number(id);  // Convert to number
```

**3. Handle missing/invalid data**
```jsx
if (!user) {
  return <div>User not found</div>;
}
```

**4. Use useEffect with parameter as dependency**
```jsx
useEffect(() => {
  fetchData(id);
}, [id]);  // Refetch when id changes
```

**5. Use descriptive parameter names**
```jsx
// ✅ Good
<Route path="/users/:userId" />
<Route path="/posts/:postId" />

// ❌ Avoid
<Route path="/users/:id" />
<Route path="/posts/:id" />  // Confusing when nested
```

**6. Keep URLs readable**
```jsx
// ✅ Good
/products/laptop-dell-xps-13

// ⚠️ Less readable
/products/SKU12345
```

---

## Common Patterns

### Pattern 1: Edit Form

```jsx
<Route path="/users/:id/edit" element={<EditUser />} />

function EditUser() {
  const { id } = useParams();
  const navigate = useNavigate();
  
  const handleSubmit = async (data) => {
    await updateUser(id, data);
    navigate(`/users/${id}`);
  };
  
  return <UserForm userId={id} onSubmit={handleSubmit} />;
}
```

---

### Pattern 2: Tabs with Parameter

```jsx
<Route path="/users/:id" element={<UserProfile />}>
  <Route path="posts" element={<UserPosts />} />
  <Route path="photos" element={<UserPhotos />} />
  <Route path="friends" element={<UserFriends />} />
</Route>

function UserProfile() {
  const { id } = useParams();
  
  return (
    <div>
      <h1>User {id}</h1>
      <nav>
        <Link to={`/users/${id}/posts`}>Posts</Link>
        <Link to={`/users/${id}/photos`}>Photos</Link>
        <Link to={`/users/${id}/friends`}>Friends</Link>
      </nav>
      <Outlet />
    </div>
  );
}
```

---

### Pattern 3: Master-Detail

```jsx
<Route path="/emails" element={<EmailList />} />
<Route path="/emails/:id" element={<EmailDetail />} />

function EmailList() {
  const emails = useEmails();
  
  return (
    <ul>
      {emails.map(email => (
        <li key={email.id}>
          <Link to={`/emails/${email.id}`}>{email.subject}</Link>
        </li>
      ))}
    </ul>
  );
}

function EmailDetail() {
  const { id } = useParams();
  const email = useEmail(id);
  
  return (
    <div>
      <h1>{email.subject}</h1>
      <p>{email.body}</p>
    </div>
  );
}
```

---

## Comparison: Parameters vs Query Strings

**Route Parameters:**
```
/users/123          ← Part of route
/products/laptop    ← Required for routing
```

**Query Strings:**
```
/products?category=electronics&sort=price  ← Optional filters
/search?q=laptop&page=2                     ← Search/pagination
```

**When to use route parameters:**
- Identifying a specific resource
- Required for page to work
- Part of the URL structure
- SEO-friendly URLs

**When to use query strings:**
- Optional filters
- Search terms
- Pagination
- Sorting options

---

**Interview Tips:**
- Route parameters = **dynamic segments** in URL
- Define with **`:paramName`** syntax
- Access with **`useParams()`** hook
- Returns **object** with all parameters
- All parameters are **strings** (convert if needed)
- Multiple parameters: `/blog/:year/:month/:day`
- **Re-fetch data** when parameter changes
- Use **useEffect** with parameter as dependency
- **Validate** parameters before using
- **Convert types** (Number, parseInt)
- Handle **missing/invalid** data
- Nested routes can access **parent parameters**
- Wildcard parameter: `/*` catches remaining path
- Access wildcard with `params['*']`
- Parameters vs query strings: **required vs optional**
- Parameters for **resource ID**, queries for **filters**
- Use **descriptive names** (userId, postId, not just id)
- Common pattern: **fetch data based on parameter**
- Common pattern: **edit forms** with ID parameter
- Common pattern: **master-detail** views
- Always handle **loading/error states**
- **Navigate** to 404 if invalid parameter
- Keep URLs **readable and SEO-friendly**
- Use slugs for better SEO: `/posts/hello-world` not `/posts/123`
- Parameters are **shareable** and **bookmarkable**

</details>

---

### 65. How do you programmatically navigate in React Router v6?

<details>
<summary>View Answer</summary>

**Programmatic Navigation**

Navigating to a different route **from code** (not from a link click), typically after an action like form submission, successful login, or data deletion.

**React Router v6 uses the `useNavigate()` hook**

---

## useNavigate() Hook

**Returns a function to navigate programmatically:**

```jsx
import { useNavigate } from 'react-router-dom';

function MyComponent() {
  const navigate = useNavigate();
  
  const handleClick = () => {
    navigate('/about');  // Navigate to /about
  };
  
  return <button onClick={handleClick}>Go to About</button>;
}
```

---

## Basic Usage

### Navigate to a path

```jsx
import { useNavigate } from 'react-router-dom';

function LoginForm() {
  const navigate = useNavigate();
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    // Login logic...
    const success = await login(username, password);
    
    if (success) {
      navigate('/dashboard');  // Redirect to dashboard
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input type="text" placeholder="Username" />
      <input type="password" placeholder="Password" />
      <button type="submit">Login</button>
    </form>
  );
}
```

---

### Navigate with replace

**Replace current entry in history (can't go back):**

```jsx
navigate('/dashboard', { replace: true });

// User can't click back to login page
```

**Use cases:**
- After login (don't let user go back to login page)
- After logout (don't let user go back to protected page)
- After form submission (prevent resubmission)

---

### Navigate with state

**Pass data to the next page:**

```jsx
function ProductList() {
  const navigate = useNavigate();
  
  const handleViewProduct = (product) => {
    navigate('/product-details', {
      state: { product }  // Pass product data
    });
  };
  
  return (
    <div>
      {products.map(product => (
        <button key={product.id} onClick={() => handleViewProduct(product)}>
          View {product.name}
        </button>
      ))}
    </div>
  );
}

function ProductDetails() {
  const location = useLocation();
  const product = location.state?.product;  // Access passed data
  
  if (!product) {
    return <div>Product not found</div>;
  }
  
  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.price}</p>
    </div>
  );
}
```

---

### Navigate back/forward

**Navigate through history:**

```jsx
const navigate = useNavigate();

// Go back one page
navigate(-1);

// Go forward one page
navigate(1);

// Go back two pages
navigate(-2);
```

**Common pattern:**
```jsx
function ProductDetails() {
  const navigate = useNavigate();
  
  return (
    <div>
      <button onClick={() => navigate(-1)}>
        &larr; Back
      </button>
      <h1>Product Details</h1>
    </div>
  );
}
```

---

## Complete Examples

### Example 1: Login Form with Redirect

```jsx
import React, { useState } from 'react';
import { useNavigate, useLocation } from 'react-router-dom';

function LoginPage() {
  const navigate = useNavigate();
  const location = useLocation();
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);
  
  // Get the page user was trying to access
  const from = location.state?.from?.pathname || '/dashboard';
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    setError('');
    setLoading(true);
    
    const formData = new FormData(e.target);
    const username = formData.get('username');
    const password = formData.get('password');
    
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ username, password })
      });
      
      if (!response.ok) {
        throw new Error('Invalid credentials');
      }
      
      const data = await response.json();
      localStorage.setItem('token', data.token);
      
      // Navigate to original destination or dashboard
      navigate(from, { replace: true });
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div className="login-page">
      <h1>Login</h1>
      
      {error && <div className="error">{error}</div>}
      
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          name="username"
          placeholder="Username"
          required
        />
        <input
          type="password"
          name="password"
          placeholder="Password"
          required
        />
        <button type="submit" disabled={loading}>
          {loading ? 'Logging in...' : 'Login'}
        </button>
      </form>
    </div>
  );
}
```

---

### Example 2: Create Post Form

```jsx
import React, { useState } from 'react';
import { useNavigate } from 'react-router-dom';

function CreatePost() {
  const navigate = useNavigate();
  const [title, setTitle] = useState('');
  const [content, setContent] = useState('');
  const [saving, setSaving] = useState(false);
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    setSaving(true);
    
    try {
      const response = await fetch('/api/posts', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ title, content })
      });
      
      const post = await response.json();
      
      // Navigate to new post with success message
      navigate(`/posts/${post.id}`, {
        state: { message: 'Post created successfully!' }
      });
    } catch (error) {
      alert('Failed to create post');
      setSaving(false);
    }
  };
  
  const handleCancel = () => {
    if (window.confirm('Discard changes?')) {
      navigate('/posts');
    }
  };
  
  return (
    <div className="create-post">
      <h1>Create New Post</h1>
      
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          placeholder="Title"
          value={title}
          onChange={(e) => setTitle(e.target.value)}
          required
        />
        <textarea
          placeholder="Content"
          value={content}
          onChange={(e) => setContent(e.target.value)}
          required
        />
        <div className="actions">
          <button type="submit" disabled={saving}>
            {saving ? 'Saving...' : 'Publish'}
          </button>
          <button type="button" onClick={handleCancel}>
            Cancel
          </button>
        </div>
      </form>
    </div>
  );
}
```

---

### Example 3: Delete with Confirmation

```jsx
import { useNavigate } from 'react-router-dom';

function ProductDetails({ product }) {
  const navigate = useNavigate();
  
  const handleDelete = async () => {
    if (!window.confirm('Are you sure you want to delete this product?')) {
      return;
    }
    
    try {
      await fetch(`/api/products/${product.id}`, {
        method: 'DELETE'
      });
      
      // Navigate to list with success message
      navigate('/products', {
        state: { message: 'Product deleted successfully' },
        replace: true  // Can't go back to deleted product
      });
    } catch (error) {
      alert('Failed to delete product');
    }
  };
  
  return (
    <div>
      <h1>{product.name}</h1>
      <button onClick={handleDelete}>Delete Product</button>
    </div>
  );
}
```

---

### Example 4: Multi-step Form

```jsx
import React, { useState } from 'react';
import { useNavigate } from 'react-router-dom';

function MultiStepForm() {
  const navigate = useNavigate();
  const [step, setStep] = useState(1);
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    address: ''
  });
  
  const handleNext = () => {
    if (step < 3) {
      setStep(step + 1);
    } else {
      // Submit form
      submitForm();
    }
  };
  
  const handleBack = () => {
    if (step > 1) {
      setStep(step - 1);
    } else {
      navigate(-1);  // Go back to previous page
    }
  };
  
  const submitForm = async () => {
    try {
      await fetch('/api/signup', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData)
      });
      
      // Navigate to success page
      navigate('/signup-complete', {
        state: { name: formData.name },
        replace: true
      });
    } catch (error) {
      alert('Signup failed');
    }
  };
  
  return (
    <div className="multi-step-form">
      <h1>Sign Up - Step {step} of 3</h1>
      
      {step === 1 && (
        <input
          type="text"
          placeholder="Name"
          value={formData.name}
          onChange={(e) => setFormData({ ...formData, name: e.target.value })}
        />
      )}
      
      {step === 2 && (
        <input
          type="email"
          placeholder="Email"
          value={formData.email}
          onChange={(e) => setFormData({ ...formData, email: e.target.value })}
        />
      )}
      
      {step === 3 && (
        <input
          type="text"
          placeholder="Address"
          value={formData.address}
          onChange={(e) => setFormData({ ...formData, address: e.target.value })}
        />
      )}
      
      <div className="actions">
        <button onClick={handleBack}>
          {step === 1 ? 'Cancel' : 'Back'}
        </button>
        <button onClick={handleNext}>
          {step === 3 ? 'Submit' : 'Next'}
        </button>
      </div>
    </div>
  );
}
```

---

## Navigation Options

**All options for `navigate()`:**

```jsx
navigate(to, options);
```

**Options:**
```jsx
{
  replace: false,        // Replace current entry in history
  state: undefined,      // Pass state to next location
  preventScrollReset: false,  // Don't scroll to top
  relative: 'route'      // Relative path resolution
}
```

**Examples:**
```jsx
// Basic navigation
navigate('/about');

// Replace current entry
navigate('/dashboard', { replace: true });

// Pass state
navigate('/details', { state: { id: 123 } });

// Multiple options
navigate('/success', {
  replace: true,
  state: { message: 'Form submitted!' }
});

// Relative navigation
navigate('../', { relative: 'path' });
```

---

## React Router v5 vs v6

**v5 used `useHistory()`:**
```jsx
import { useHistory } from 'react-router-dom';

function MyComponent() {
  const history = useHistory();
  
  history.push('/about');
  history.replace('/dashboard');
  history.goBack();
  history.goForward();
}
```

**v6 uses `useNavigate()`:**
```jsx
import { useNavigate } from 'react-router-dom';

function MyComponent() {
  const navigate = useNavigate();
  
  navigate('/about');
  navigate('/dashboard', { replace: true });
  navigate(-1);  // goBack
  navigate(1);   // goForward
}
```

**Migration:**
```jsx
// v5
history.push('/about');
history.replace('/dashboard');
history.goBack();

// v6 equivalent
navigate('/about');
navigate('/dashboard', { replace: true });
navigate(-1);
```

---

## Common Patterns

### Pattern 1: Redirect After Timeout

```jsx
import { useEffect } from 'react';
import { useNavigate } from 'react-router-dom';

function SuccessPage() {
  const navigate = useNavigate();
  
  useEffect(() => {
    // Redirect after 3 seconds
    const timer = setTimeout(() => {
      navigate('/dashboard');
    }, 3000);
    
    return () => clearTimeout(timer);
  }, [navigate]);
  
  return (
    <div>
      <h1>Success!</h1>
      <p>Redirecting to dashboard in 3 seconds...</p>
    </div>
  );
}
```

---

### Pattern 2: Navigate on External Event

```jsx
import { useEffect } from 'react';
import { useNavigate } from 'react-router-dom';

function ProtectedPage() {
  const navigate = useNavigate();
  
  useEffect(() => {
    // Listen for auth token expiry
    const handleTokenExpired = () => {
      navigate('/login', {
        state: { message: 'Session expired. Please login again.' },
        replace: true
      });
    };
    
    authService.on('tokenExpired', handleTokenExpired);
    
    return () => {
      authService.off('tokenExpired', handleTokenExpired);
    };
  }, [navigate]);
  
  return <div>Protected Content</div>;
}
```

---

### Pattern 3: Conditional Navigation

```jsx
import { useNavigate } from 'react-router-dom';

function Checkout() {
  const navigate = useNavigate();
  const cart = useCart();
  
  const handleCheckout = async () => {
    if (cart.items.length === 0) {
      alert('Your cart is empty');
      navigate('/products');
      return;
    }
    
    if (!isLoggedIn()) {
      // Save intended destination
      navigate('/login', {
        state: { from: '/checkout' }
      });
      return;
    }
    
    try {
      await processPayment();
      navigate('/order-success', { replace: true });
    } catch (error) {
      navigate('/order-failed', {
        state: { error: error.message }
      });
    }
  };
  
  return <button onClick={handleCheckout}>Checkout</button>;
}
```

---

### Pattern 4: Navigate from Outside Component

**Problem:** Can't use hooks outside components

**Solution 1: Navigate component**
```jsx
import { Navigate } from 'react-router-dom';

function ProtectedRoute({ isAuthenticated, children }) {
  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }
  
  return children;
}
```

**Solution 2: Create navigation service**
```jsx
// navigationService.js
let navigate;

export const setNavigate = (nav) => {
  navigate = nav;
};

export const navigateTo = (path, options) => {
  if (navigate) {
    navigate(path, options);
  }
};

// App.js
import { useNavigate } from 'react-router-dom';
import { setNavigate } from './navigationService';

function App() {
  const navigate = useNavigate();
  
  useEffect(() => {
    setNavigate(navigate);
  }, [navigate]);
  
  return <Routes>{/* routes */}</Routes>;
}

// Now use anywhere
import { navigateTo } from './navigationService';

function apiCall() {
  if (response.status === 401) {
    navigateTo('/login');
  }
}
```

---

## Navigate Component (Declarative)

**For declarative redirects:**

```jsx
import { Navigate } from 'react-router-dom';

function OldPage() {
  return <Navigate to="/new-page" replace />;
}

// With state
function UnauthorizedPage() {
  return (
    <Navigate
      to="/login"
      state={{ from: location.pathname }}
      replace
    />
  );
}

// Conditional redirect
function Dashboard({ isAuthenticated }) {
  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }
  
  return <div>Dashboard content</div>;
}
```

**`<Navigate>` vs `navigate()`:**
- `<Navigate>` = **Declarative** (component-based)
- `navigate()` = **Imperative** (function-based)

**Use `<Navigate>` when:**
- Rendering redirect based on condition
- Permanent redirects
- Inside render logic

**Use `navigate()` when:**
- After user action (click, submit)
- After async operation
- In event handlers
- In useEffect

---

## Best Practices

**1. Use `replace: true` for auth redirects**
```jsx
// After login
navigate('/dashboard', { replace: true });

// After logout
navigate('/login', { replace: true });
```

**2. Pass state for context**
```jsx
navigate('/error', {
  state: { message: 'Something went wrong' }
});
```

**3. Handle navigation in try-catch**
```jsx
try {
  await submitForm();
  navigate('/success');
} catch (error) {
  navigate('/error', { state: { error } });
}
```

**4. Add navigate to useEffect dependencies**
```jsx
useEffect(() => {
  if (condition) {
    navigate('/somewhere');
  }
}, [condition, navigate]);  // Include navigate
```

**5. Confirm before navigation (if needed)**
```jsx
const handleCancel = () => {
  if (hasUnsavedChanges && !window.confirm('Discard changes?')) {
    return;
  }
  navigate('/dashboard');
};
```

---

## Common Mistakes

**❌ Mistake 1: Navigating during render**
```jsx
// Wrong - causes infinite loop
function MyComponent() {
  const navigate = useNavigate();
  navigate('/somewhere');  // Don't do this!
  return <div>Content</div>;
}

// Correct - use useEffect or event handler
function MyComponent() {
  const navigate = useNavigate();
  
  useEffect(() => {
    navigate('/somewhere');
  }, []);
  
  return <div>Content</div>;
}
```

**❌ Mistake 2: Not handling navigation errors**
```jsx
// Wrong
await deleteItem();
navigate('/items');

// Correct
try {
  await deleteItem();
  navigate('/items');
} catch (error) {
  console.error('Delete failed:', error);
}
```

**❌ Mistake 3: Using navigate in component that might unmount**
```jsx
// Wrong - might cause "Can't perform a React state update on unmounted component"
const handleSubmit = async () => {
  await saveData();
  navigate('/success');
};

// Correct - check if mounted
const handleSubmit = async () => {
  const success = await saveData();
  if (success) {
    navigate('/success');
  }
};
```

---

**Interview Tips:**
- Programmatic navigation = **navigate from code**, not link clicks
- React Router v6 uses **`useNavigate()`** hook
- v5 used **`useHistory()`** (deprecated)
- **`navigate(path)`** - navigate to path
- **`navigate(-1)`** - go back
- **`navigate(1)`** - go forward
- **`replace: true`** - replace history entry (can't go back)
- **`state`** option - pass data to next page
- Access state with **`useLocation().state`**
- Use after **form submission**, **login**, **data operations**
- **`<Navigate>`** component for declarative redirects
- Navigate is **imperative**, `<Navigate>` is **declarative**
- Use `navigate()` in **event handlers** and **useEffect**
- Use `<Navigate>` in **render logic** for conditional redirects
- Always handle **errors** when navigating after async operations
- Use **`replace: true`** for login/logout to prevent back navigation
- Don't navigate **during render** (causes infinite loop)
- Add **navigate to useEffect dependencies**
- Can pass **state** for success messages, error info, etc.
- **Confirm** before navigation if unsaved changes
- Common pattern: redirect after **timeout**
- Common pattern: navigate on **auth token expiry**
- Common pattern: **multi-step forms** with back/next
- v6 simpler than v5 (fewer methods, cleaner API)

</details>

---

### 66. What is the difference between Link and NavLink?

<details>
<summary>View Answer</summary>

**Link vs NavLink**

Both are React Router components for **client-side navigation** without page reloads, but **NavLink** adds styling capabilities for active links.

**Quick comparison:**
- **Link:** Basic navigation, no active state
- **NavLink:** Navigation + **active state styling**

---

## Link Component

**Basic navigation component:**

```jsx
import { Link } from 'react-router-dom';

function Navigation() {
  return (
    <nav>
      <Link to="/">Home</Link>
      <Link to="/about">About</Link>
      <Link to="/contact">Contact</Link>
    </nav>
  );
}
```

**What it does:**
- Renders as `<a>` tag
- Prevents default browser navigation
- Uses React Router's navigation
- No page reload
- Updates URL and renders new component

**Rendered HTML:**
```html
<a href="/about">About</a>
```

---

## NavLink Component

**Navigation with active state:**

```jsx
import { NavLink } from 'react-router-dom';

function Navigation() {
  return (
    <nav>
      <NavLink to="/">Home</NavLink>
      <NavLink to="/about">About</NavLink>
      <NavLink to="/contact">Contact</NavLink>
    </nav>
  );
}
```

**What it does:**
- Everything Link does
- **Plus:** Adds `active` class to current route
- **Plus:** Provides `isActive` for custom styling
- **Plus:** Allows custom active styling logic

**Rendered HTML (when active):**
```html
<a href="/about" class="active">About</a>
```

---

## Key Differences

| Feature | Link | NavLink |
|---------|------|----------|
| **Navigation** | ✅ Yes | ✅ Yes |
| **Active class** | ❌ No | ✅ Yes (automatic) |
| **isActive prop** | ❌ No | ✅ Yes |
| **Custom styling** | Manual only | Built-in support |
| **Use case** | General links | Navigation menus |
| **Performance** | Slightly faster | Minimal overhead |
| **When to use** | Buttons, cards, text links | Nav bars, sidebars, menus |

---

## NavLink Active Styling

### Method 1: CSS class (default)

**NavLink automatically adds `active` class:**

```jsx
import { NavLink } from 'react-router-dom';

function Navigation() {
  return (
    <nav>
      <NavLink to="/">Home</NavLink>
      <NavLink to="/about">About</NavLink>
      <NavLink to="/contact">Contact</NavLink>
    </nav>
  );
}
```

**CSS:**
```css
nav a {
  color: #333;
  text-decoration: none;
  padding: 10px 20px;
}

nav a.active {
  color: #007bff;
  border-bottom: 2px solid #007bff;
  font-weight: bold;
}
```

---

### Method 2: Custom class name

**Function that receives `{ isActive }`:**

```jsx
import { NavLink } from 'react-router-dom';

function Navigation() {
  return (
    <nav>
      <NavLink
        to="/"
        className={({ isActive }) => isActive ? 'active' : ''}
      >
        Home
      </NavLink>
      <NavLink
        to="/about"
        className={({ isActive }) => isActive ? 'active' : ''}
      >
        About
      </NavLink>
    </nav>
  );
}
```

**With multiple classes:**
```jsx
<NavLink
  to="/dashboard"
  className={({ isActive }) => 
    isActive ? 'nav-link active' : 'nav-link'
  }
>
  Dashboard
</NavLink>
```

---

### Method 3: Inline styles

```jsx
import { NavLink } from 'react-router-dom';

function Navigation() {
  return (
    <nav>
      <NavLink
        to="/"
        style={({ isActive }) => ({
          color: isActive ? '#007bff' : '#333',
          fontWeight: isActive ? 'bold' : 'normal',
          borderBottom: isActive ? '2px solid #007bff' : 'none'
        })}
      >
        Home
      </NavLink>
    </nav>
  );
}
```

---

### Method 4: Custom rendering

```jsx
import { NavLink } from 'react-router-dom';

function Navigation() {
  return (
    <nav>
      <NavLink to="/">
        {({ isActive }) => (
          <span className={isActive ? 'active' : ''}>
            {isActive && '▶ '}
            Home
          </span>
        )}
      </NavLink>
    </nav>
  );
}
```

---

## Complete Example

```jsx
import React from 'react';
import { BrowserRouter, Routes, Route, Link, NavLink } from 'react-router-dom';
import './App.css';

function App() {
  return (
    <BrowserRouter>
      <div className="app">
        <header>
          <h1>My Website</h1>
          <Navigation />
        </header>
        
        <main>
          <Routes>
            <Route path="/" element={<Home />} />
            <Route path="/about" element={<About />} />
            <Route path="/services" element={<Services />} />
            <Route path="/contact" element={<Contact />} />
          </Routes>
        </main>
      </div>
    </BrowserRouter>
  );
}

// Navigation with NavLinks
function Navigation() {
  return (
    <nav className="navbar">
      <NavLink
        to="/"
        className={({ isActive }) => isActive ? 'nav-link active' : 'nav-link'}
      >
        Home
      </NavLink>
      <NavLink
        to="/about"
        className={({ isActive }) => isActive ? 'nav-link active' : 'nav-link'}
      >
        About
      </NavLink>
      <NavLink
        to="/services"
        className={({ isActive }) => isActive ? 'nav-link active' : 'nav-link'}
      >
        Services
      </NavLink>
      <NavLink
        to="/contact"
        className={({ isActive }) => isActive ? 'nav-link active' : 'nav-link'}
      >
        Contact
      </NavLink>
    </nav>
  );
}

// Pages
function Home() {
  return (
    <div>
      <h2>Home Page</h2>
      <p>Welcome to our website!</p>
      {/* Regular Link for non-nav links */}
      <Link to="/about" className="button">Learn More</Link>
    </div>
  );
}

function About() {
  return <h2>About Page</h2>;
}

function Services() {
  return <h2>Services Page</h2>;
}

function Contact() {
  return <h2>Contact Page</h2>;
}

export default App;
```

**CSS (App.css):**
```css
.navbar {
  display: flex;
  gap: 10px;
  padding: 20px;
  background: #f5f5f5;
}

.nav-link {
  padding: 10px 20px;
  text-decoration: none;
  color: #333;
  border-radius: 4px;
  transition: all 0.3s;
}

.nav-link:hover {
  background: #e0e0e0;
}

.nav-link.active {
  background: #007bff;
  color: white;
  font-weight: bold;
}

/* Regular links (not navigation) */
.button {
  display: inline-block;
  padding: 10px 20px;
  background: #007bff;
  color: white;
  text-decoration: none;
  border-radius: 4px;
}
```

---

## When to Use Which

### Use Link

**✅ For general navigation:**
```jsx
<Link to="/about">Learn more</Link>
<Link to="/products/123">View Product</Link>
```

**Use cases:**
- Buttons
- Cards
- Text links in content
- Breadcrumbs
- Footer links
- Links in lists
- "Read more" links
- Anywhere you don't need active state

---

### Use NavLink

**✅ For navigation menus:**
```jsx
<nav>
  <NavLink to="/">Home</NavLink>
  <NavLink to="/about">About</NavLink>
</nav>
```

**Use cases:**
- Main navigation bar
- Sidebar navigation
- Tab navigation
- Breadcrumbs (if highlighting current)
- Settings pages with tabs
- Dashboard navigation
- Anywhere you need to show current page

---

## Advanced NavLink Patterns

### Pattern 1: Active with Icon

```jsx
import { NavLink } from 'react-router-dom';
import { HomeIcon, UserIcon, SettingsIcon } from './icons';

function Sidebar() {
  return (
    <nav className="sidebar">
      <NavLink to="/dashboard">
        {({ isActive }) => (
          <>
            <HomeIcon className={isActive ? 'icon-active' : 'icon'} />
            <span className={isActive ? 'text-active' : 'text'}>
              Dashboard
            </span>
          </>
        )}
      </NavLink>
      
      <NavLink to="/profile">
        {({ isActive }) => (
          <>
            <UserIcon className={isActive ? 'icon-active' : 'icon'} />
            <span className={isActive ? 'text-active' : 'text'}>
              Profile
            </span>
          </>
        )}
      </NavLink>
    </nav>
  );
}
```

---

### Pattern 2: Partial Matching

**Match parent route even when on child route:**

```jsx
import { NavLink, useLocation } from 'react-router-dom';

function Navigation() {
  const location = useLocation();
  
  return (
    <nav>
      <NavLink
        to="/products"
        className={({ isActive }) => {
          // Also active for /products/123, /products/new, etc.
          const isProductsPage = location.pathname.startsWith('/products');
          return isProductsPage ? 'nav-link active' : 'nav-link';
        }}
      >
        Products
      </NavLink>
    </nav>
  );
}
```

---

### Pattern 3: Active with Badge

```jsx
import { NavLink } from 'react-router-dom';

function Navigation({ unreadCount }) {
  return (
    <nav>
      <NavLink to="/messages">
        {({ isActive }) => (
          <span className={isActive ? 'nav-item active' : 'nav-item'}>
            Messages
            {unreadCount > 0 && (
              <span className="badge">{unreadCount}</span>
            )}
          </span>
        )}
      </NavLink>
    </nav>
  );
}
```

---

### Pattern 4: Dropdown with Active Parent

```jsx
import { NavLink, useLocation } from 'react-router-dom';
import { useState } from 'react';

function Navigation() {
  const [isOpen, setIsOpen] = useState(false);
  const location = useLocation();
  
  const isProductsActive = location.pathname.startsWith('/products');
  
  return (
    <nav>
      <div className="dropdown">
        <button
          className={isProductsActive ? 'dropdown-toggle active' : 'dropdown-toggle'}
          onClick={() => setIsOpen(!isOpen)}
        >
          Products
        </button>
        
        {isOpen && (
          <div className="dropdown-menu">
            <NavLink to="/products/all">All Products</NavLink>
            <NavLink to="/products/featured">Featured</NavLink>
            <NavLink to="/products/sale">On Sale</NavLink>
          </div>
        )}
      </div>
    </nav>
  );
}
```

---

## NavLink Props

**All props available:**

```jsx
<NavLink
  to="/about"                    // Required: destination
  className={({ isActive }) => ...}  // Optional: classes
  style={({ isActive }) => ...}      // Optional: inline styles
  end={false}                        // Optional: exact match
  caseSensitive={false}              // Optional: case-sensitive
>
  About
</NavLink>
```

**`end` prop:**
```jsx
// Without end prop
<NavLink to="/products">Products</NavLink>
// Active for: /products, /products/123, /products/new

// With end prop
<NavLink to="/products" end>Products</NavLink>
// Active for: /products only (not /products/123)
```

---

## Accessibility

**NavLink enhances accessibility:**

```jsx
<NavLink
  to="/about"
  className={({ isActive }) => isActive ? 'active' : ''}
  aria-current={({ isActive }) => isActive ? 'page' : undefined}
>
  About
</NavLink>
```

**Rendered when active:**
```html
<a href="/about" class="active" aria-current="page">About</a>
```

**Screen readers announce current page**

---

## Performance

**Both are very fast, minimal difference:**

```jsx
// Link - Slightly faster (no active state checking)
<Link to="/about">About</Link>

// NavLink - Minimal overhead (checks if active)
<NavLink to="/about">About</NavLink>
```

**Difference is negligible in practice**

**Rule of thumb:**
- Use Link for 100+ links on page (rare)
- Use NavLink for navigation (10-20 links)

---

## Migration from v5

**React Router v5:**
```jsx
import { NavLink } from 'react-router-dom';

<NavLink
  to="/about"
  activeClassName="active"      // v5
  activeStyle={{ color: 'red' }} // v5
>
  About
</NavLink>
```

**React Router v6:**
```jsx
import { NavLink } from 'react-router-dom';

<NavLink
  to="/about"
  className={({ isActive }) => isActive ? 'active' : ''}  // v6
  style={({ isActive }) => ({ color: isActive ? 'red' : 'black' })}  // v6
>
  About
</NavLink>
```

**Changes:**
- v5: `activeClassName` and `activeStyle` props
- v6: Functions that receive `{ isActive }`

---

## Common Mistakes

**❌ Mistake 1: Using NavLink for all links**
```jsx
// Wrong - unnecessary for non-nav links
<div className="card">
  <NavLink to="/product/123">View Product</NavLink>
</div>

// Correct - use Link
<div className="card">
  <Link to="/product/123">View Product</Link>
</div>
```

**❌ Mistake 2: Not using className function**
```jsx
// Wrong - className is always applied
<NavLink to="/about" className="active">
  About
</NavLink>

// Correct - function receives isActive
<NavLink
  to="/about"
  className={({ isActive }) => isActive ? 'active' : ''}
>
  About
</NavLink>
```

**❌ Mistake 3: Forgetting CSS for active class**
```jsx
<NavLink to="/about">About</NavLink>

// Missing CSS:
// .active { /* your styles */ }
```

---

## Best Practices

**1. Use NavLink for navigation, Link for content**
```jsx
// Navigation - use NavLink
<nav>
  <NavLink to="/">Home</NavLink>
</nav>

// Content - use Link
<p>Check out our <Link to="/products">products</Link>!</p>
```

**2. Extract common className logic**
```jsx
const navLinkClass = ({ isActive }) => 
  isActive ? 'nav-link active' : 'nav-link';

<NavLink to="/" className={navLinkClass}>Home</NavLink>
<NavLink to="/about" className={navLinkClass}>About</NavLink>
```

**3. Use semantic HTML**
```jsx
// ✅ Good
<nav>
  <NavLink to="/">Home</NavLink>
</nav>

// ❌ Avoid
<div>
  <NavLink to="/">Home</NavLink>
</div>
```

**4. Add aria-current for accessibility**
```jsx
<NavLink
  to="/about"
  aria-current={({ isActive }) => isActive ? 'page' : undefined}
>
  About
</NavLink>
```

---

**Interview Tips:**
- **Link** = basic navigation
- **NavLink** = navigation with **active state**
- NavLink adds **`active` class** automatically
- Both prevent **page reload**
- Both render as **`<a>` tag**
- NavLink **slightly heavier** (negligible)
- Use **NavLink** for nav bars, sidebars, menus
- Use **Link** for content links, buttons, cards
- NavLink className/style props receive **`{ isActive }`**
- **`isActive`** = true when route matches
- Can customize with **className function**
- Can use **inline styles** with style prop
- Can render **custom content** based on isActive
- **`end` prop** for exact matching
- Active class improves **accessibility** (aria-current)
- React Router v6 uses **functions**, v5 used props
- v5: `activeClassName`, v6: `className={({ isActive }) => ...}`
- Common mistake: using NavLink everywhere (overkill)
- NavLink enhances **user experience** (shows current page)
- Both work with **nested routes**
- Both support **relative paths**
- NavLink perfect for **tabs**, **sidebars**, **menus**
- Link perfect for **buttons**, **cards**, **text links**

</details>

---

### 67. How do you protect routes (authentication)?

<details>
<summary>View Answer</summary>

**Protected Routes**

Protected routes (also called **private routes** or **auth routes**) are routes that require **authentication** before allowing access. If the user is not authenticated, they're redirected to a login page.

**Common use cases:**
- User dashboard
- Admin panel
- Profile pages
- Settings
- Checkout pages

---

## Basic Pattern

**Wrap protected component with auth check:**

```jsx
import { Navigate } from 'react-router-dom';

function ProtectedRoute({ children }) {
  const isAuthenticated = checkAuth(); // Your auth logic
  
  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }
  
  return children;
}

// Usage
<Route
  path="/dashboard"
  element={
    <ProtectedRoute>
      <Dashboard />
    </ProtectedRoute>
  }
/>
```

---

## Method 1: Protected Route Component

**Create reusable ProtectedRoute component:**

```jsx
import { Navigate, useLocation } from 'react-router-dom';

function ProtectedRoute({ children }) {
  const isAuthenticated = localStorage.getItem('token') !== null;
  const location = useLocation();
  
  if (!isAuthenticated) {
    // Redirect to login, save intended destination
    return <Navigate to="/login" state={{ from: location }} replace />;
  }
  
  return children;
}

export default ProtectedRoute;
```

**Use in routes:**
```jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import ProtectedRoute from './ProtectedRoute';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        {/* Public routes */}
        <Route path="/" element={<Home />} />
        <Route path="/login" element={<Login />} />
        <Route path="/register" element={<Register />} />
        
        {/* Protected routes */}
        <Route
          path="/dashboard"
          element={
            <ProtectedRoute>
              <Dashboard />
            </ProtectedRoute>
          }
        />
        <Route
          path="/profile"
          element={
            <ProtectedRoute>
              <Profile />
            </ProtectedRoute>
          }
        />
        <Route
          path="/settings"
          element={
            <ProtectedRoute>
              <Settings />
            </ProtectedRoute>
          }
        />
      </Routes>
    </BrowserRouter>
  );
}
```

---

## Method 2: Auth Context

**Better approach: Use Context for auth state**

**1. Create Auth Context:**
```jsx
import { createContext, useContext, useState, useEffect } from 'react';

const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    // Check if user is logged in on mount
    const token = localStorage.getItem('token');
    if (token) {
      // Verify token and load user
      fetchUser(token)
        .then(setUser)
        .catch(() => localStorage.removeItem('token'))
        .finally(() => setLoading(false));
    } else {
      setLoading(false);
    }
  }, []);
  
  const login = async (username, password) => {
    const response = await fetch('/api/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ username, password })
    });
    
    const data = await response.json();
    localStorage.setItem('token', data.token);
    setUser(data.user);
  };
  
  const logout = () => {
    localStorage.removeItem('token');
    setUser(null);
  };
  
  return (
    <AuthContext.Provider value={{ user, login, logout, loading }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  return useContext(AuthContext);
}
```

**2. Create Protected Route:**
```jsx
import { Navigate, useLocation } from 'react-router-dom';
import { useAuth } from './AuthContext';

function ProtectedRoute({ children }) {
  const { user, loading } = useAuth();
  const location = useLocation();
  
  if (loading) {
    return <div>Loading...</div>;
  }
  
  if (!user) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }
  
  return children;
}

export default ProtectedRoute;
```

**3. Wrap app with AuthProvider:**
```jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import { AuthProvider } from './AuthContext';
import ProtectedRoute from './ProtectedRoute';

function App() {
  return (
    <BrowserRouter>
      <AuthProvider>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/login" element={<Login />} />
          
          <Route
            path="/dashboard"
            element={
              <ProtectedRoute>
                <Dashboard />
              </ProtectedRoute>
            }
          />
        </Routes>
      </AuthProvider>
    </BrowserRouter>
  );
}
```

**4. Use auth in components:**
```jsx
import { useAuth } from './AuthContext';
import { useNavigate } from 'react-router-dom';

function Login() {
  const { login } = useAuth();
  const navigate = useNavigate();
  const location = useLocation();
  
  const from = location.state?.from?.pathname || '/dashboard';
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    await login(username, password);
    navigate(from, { replace: true });
  };
  
  return <form onSubmit={handleSubmit}>{/* form fields */}</form>;
}

function Dashboard() {
  const { user, logout } = useAuth();
  
  return (
    <div>
      <h1>Welcome, {user.name}!</h1>
      <button onClick={logout}>Logout</button>
    </div>
  );
}
```

---

## Method 3: Role-Based Protection

**Protect routes based on user roles:**

```jsx
import { Navigate, useLocation } from 'react-router-dom';
import { useAuth } from './AuthContext';

function ProtectedRoute({ children, requiredRole }) {
  const { user, loading } = useAuth();
  const location = useLocation();
  
  if (loading) {
    return <div>Loading...</div>;
  }
  
  if (!user) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }
  
  if (requiredRole && user.role !== requiredRole) {
    return <Navigate to="/unauthorized" replace />;
  }
  
  return children;
}

// Usage
<Route
  path="/admin"
  element={
    <ProtectedRoute requiredRole="admin">
      <AdminPanel />
    </ProtectedRoute>
  }
/>
```

**Multiple roles:**
```jsx
function ProtectedRoute({ children, allowedRoles = [] }) {
  const { user, loading } = useAuth();
  const location = useLocation();
  
  if (loading) {
    return <div>Loading...</div>;
  }
  
  if (!user) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }
  
  if (allowedRoles.length > 0 && !allowedRoles.includes(user.role)) {
    return <Navigate to="/unauthorized" replace />;
  }
  
  return children;
}

// Usage
<Route
  path="/admin"
  element={
    <ProtectedRoute allowedRoles={['admin', 'moderator']}>
      <AdminPanel />
    </ProtectedRoute>
  }
/>
```

---

## Method 4: Layout-Based Protection

**Protect entire layout with nested routes:**

```jsx
import { Outlet, Navigate } from 'react-router-dom';
import { useAuth } from './AuthContext';

function ProtectedLayout() {
  const { user, loading } = useAuth();
  
  if (loading) {
    return <div>Loading...</div>;
  }
  
  if (!user) {
    return <Navigate to="/login" replace />;
  }
  
  return (
    <div className="protected-layout">
      <header>
        <h1>Dashboard</h1>
        <button onClick={logout}>Logout</button>
      </header>
      <aside>
        <nav>
          <Link to="/dashboard">Home</Link>
          <Link to="/dashboard/profile">Profile</Link>
          <Link to="/dashboard/settings">Settings</Link>
        </nav>
      </aside>
      <main>
        <Outlet />  {/* Nested routes render here */}
      </main>
    </div>
  );
}

// Routes
function App() {
  return (
    <BrowserRouter>
      <AuthProvider>
        <Routes>
          <Route path="/login" element={<Login />} />
          
          {/* All nested routes are protected */}
          <Route path="/dashboard" element={<ProtectedLayout />}>
            <Route index element={<DashboardHome />} />
            <Route path="profile" element={<Profile />} />
            <Route path="settings" element={<Settings />} />
          </Route>
        </Routes>
      </AuthProvider>
    </BrowserRouter>
  );
}
```

---

## Complete Example: E-commerce App

```jsx
import React, { createContext, useContext, useState, useEffect } from 'react';
import {
  BrowserRouter,
  Routes,
  Route,
  Link,
  Navigate,
  useLocation,
  useNavigate,
  Outlet
} from 'react-router-dom';

// ==================== Auth Context ====================

const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    // Check if user is logged in
    const token = localStorage.getItem('token');
    if (token) {
      // In real app, verify token with backend
      setUser({ id: 1, name: 'John Doe', email: 'john@example.com' });
    }
    setLoading(false);
  }, []);
  
  const login = async (email, password) => {
    // In real app, call API
    const token = 'fake-jwt-token';
    localStorage.setItem('token', token);
    setUser({ id: 1, name: 'John Doe', email });
  };
  
  const logout = () => {
    localStorage.removeItem('token');
    setUser(null);
  };
  
  return (
    <AuthContext.Provider value={{ user, login, logout, loading }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}

// ==================== Protected Route ====================

function ProtectedRoute({ children }) {
  const { user, loading } = useAuth();
  const location = useLocation();
  
  if (loading) {
    return (
      <div className="loading">
        <p>Loading...</p>
      </div>
    );
  }
  
  if (!user) {
    // Save the location user was trying to access
    return <Navigate to="/login" state={{ from: location }} replace />;
  }
  
  return children;
}

// ==================== Pages ====================

function Home() {
  const { user } = useAuth();
  
  return (
    <div>
      <h1>E-commerce Store</h1>
      {user ? (
        <p>Welcome back, {user.name}!</p>
      ) : (
        <div>
          <Link to="/login">Login</Link> | <Link to="/register">Register</Link>
        </div>
      )}
      <Link to="/products">Browse Products</Link>
    </div>
  );
}

function Login() {
  const { login } = useAuth();
  const navigate = useNavigate();
  const location = useLocation();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  
  const from = location.state?.from?.pathname || '/';
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      await login(email, password);
      navigate(from, { replace: true });
    } catch (error) {
      alert('Login failed');
    }
  };
  
  return (
    <div className="login-page">
      <h1>Login</h1>
      <form onSubmit={handleSubmit}>
        <input
          type="email"
          placeholder="Email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          required
        />
        <input
          type="password"
          placeholder="Password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          required
        />
        <button type="submit">Login</button>
      </form>
      <p>
        Don't have an account? <Link to="/register">Register</Link>
      </p>
    </div>
  );
}

function Products() {
  return (
    <div>
      <h1>Products</h1>
      <Link to="/cart">View Cart (Protected)</Link>
    </div>
  );
}

function Cart() {
  const { user } = useAuth();
  
  return (
    <div>
      <h1>Shopping Cart</h1>
      <p>User: {user.email}</p>
      <Link to="/checkout">Proceed to Checkout</Link>
    </div>
  );
}

function Checkout() {
  return (
    <div>
      <h1>Checkout</h1>
      <p>Complete your purchase</p>
    </div>
  );
}

function Account() {
  const { user, logout } = useAuth();
  const navigate = useNavigate();
  
  const handleLogout = () => {
    logout();
    navigate('/');
  };
  
  return (
    <div>
      <h1>My Account</h1>
      <p>Name: {user.name}</p>
      <p>Email: {user.email}</p>
      <nav>
        <Link to="/account/orders">Orders</Link>
        <Link to="/account/profile">Profile</Link>
        <Link to="/account/settings">Settings</Link>
      </nav>
      <Outlet />
      <button onClick={handleLogout}>Logout</button>
    </div>
  );
}

function Orders() {
  return <h2>Your Orders</h2>;
}

function Profile() {
  return <h2>Edit Profile</h2>;
}

function AccountSettings() {
  return <h2>Account Settings</h2>;
}

// ==================== App ====================

function App() {
  return (
    <BrowserRouter>
      <AuthProvider>
        <Routes>
          {/* Public routes */}
          <Route path="/" element={<Home />} />
          <Route path="/login" element={<Login />} />
          <Route path="/register" element={<div>Register Page</div>} />
          <Route path="/products" element={<Products />} />
          
          {/* Protected routes */}
          <Route
            path="/cart"
            element={
              <ProtectedRoute>
                <Cart />
              </ProtectedRoute>
            }
          />
          <Route
            path="/checkout"
            element={
              <ProtectedRoute>
                <Checkout />
              </ProtectedRoute>
            }
          />
          
          {/* Protected nested routes */}
          <Route
            path="/account"
            element={
              <ProtectedRoute>
                <Account />
              </ProtectedRoute>
            }
          >
            <Route path="orders" element={<Orders />} />
            <Route path="profile" element={<Profile />} />
            <Route path="settings" element={<AccountSettings />} />
          </Route>
        </Routes>
      </AuthProvider>
    </BrowserRouter>
  );
}

export default App;
```

---

## Handling Redirects After Login

**Save and restore intended destination:**

```jsx
// ProtectedRoute saves location
function ProtectedRoute({ children }) {
  const { user } = useAuth();
  const location = useLocation();
  
  if (!user) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }
  
  return children;
}

// Login redirects to saved location
function Login() {
  const { login } = useAuth();
  const navigate = useNavigate();
  const location = useLocation();
  
  const from = location.state?.from?.pathname || '/dashboard';
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    await login(username, password);
    navigate(from, { replace: true });  // Go to original destination
  };
  
  return <form onSubmit={handleSubmit}>{/* form */}</form>;
}
```

**Example flow:**
```
1. User visits /dashboard/settings (protected)
2. Not authenticated, redirected to /login with state={{ from: '/dashboard/settings' }}
3. User logs in
4. Redirected to /dashboard/settings (original destination)
```

---

## Token Refresh Pattern

**Auto-refresh expired tokens:**

```jsx
import { useEffect } from 'react';
import { useNavigate } from 'react-router-dom';

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const navigate = useNavigate();
  
  useEffect(() => {
    // Check token expiry every minute
    const interval = setInterval(async () => {
      const token = localStorage.getItem('token');
      if (token) {
        try {
          // Try to refresh token
          const response = await fetch('/api/refresh', {
            headers: { 'Authorization': `Bearer ${token}` }
          });
          
          if (response.ok) {
            const data = await response.json();
            localStorage.setItem('token', data.token);
          } else {
            // Token expired, logout
            localStorage.removeItem('token');
            setUser(null);
            navigate('/login', {
              state: { message: 'Session expired. Please login again.' }
            });
          }
        } catch (error) {
          console.error('Token refresh failed:', error);
        }
      }
    }, 60000);  // Check every minute
    
    return () => clearInterval(interval);
  }, [navigate]);
  
  return (
    <AuthContext.Provider value={{ user, setUser }}>
      {children}
    </AuthContext.Provider>
  );
}
```

---

## Best Practices

**1. Use Context for auth state**
```jsx
// ✅ Good - centralized auth state
<AuthProvider>
  <App />
</AuthProvider>

// ❌ Avoid - checking localStorage everywhere
if (localStorage.getItem('token')) { /* ... */ }
```

**2. Show loading state**
```jsx
if (loading) {
  return <div>Loading...</div>;
}
```

**3. Save intended destination**
```jsx
return <Navigate to="/login" state={{ from: location }} replace />;
```

**4. Use replace for auth redirects**
```jsx
navigate('/dashboard', { replace: true });  // Can't go back to login
```

**5. Clear sensitive data on logout**
```jsx
const logout = () => {
  localStorage.removeItem('token');
  setUser(null);
  // Clear other data...
};
```

**6. Validate token on mount**
```jsx
useEffect(() => {
  const token = localStorage.getItem('token');
  if (token) {
    verifyToken(token).then(setUser);
  }
}, []);
```

---

## Common Patterns

### Pattern 1: Guest-Only Routes

**Redirect authenticated users away from login:**

```jsx
function GuestRoute({ children }) {
  const { user } = useAuth();
  
  if (user) {
    return <Navigate to="/dashboard" replace />;
  }
  
  return children;
}

// Usage
<Route
  path="/login"
  element={
    <GuestRoute>
      <Login />
    </GuestRoute>
  }
/>
```

---

### Pattern 2: Permission-Based Protection

```jsx
function ProtectedRoute({ children, requiredPermission }) {
  const { user, hasPermission } = useAuth();
  
  if (!user) {
    return <Navigate to="/login" replace />;
  }
  
  if (requiredPermission && !hasPermission(requiredPermission)) {
    return <Navigate to="/forbidden" replace />;
  }
  
  return children;
}

// Usage
<Route
  path="/admin/users"
  element={
    <ProtectedRoute requiredPermission="users.manage">
      <ManageUsers />
    </ProtectedRoute>
  }
/>
```

---

### Pattern 3: Conditional Rendering

```jsx
function Dashboard() {
  const { user } = useAuth();
  
  return (
    <div>
      <h1>Dashboard</h1>
      
      {user.role === 'admin' && (
        <Link to="/admin">Admin Panel</Link>
      )}
      
      {user.isPremium && (
        <PremiumFeatures />
      )}
    </div>
  );
}
```

---

**Interview Tips:**
- Protected routes = routes requiring **authentication**
- Redirect to **login** if not authenticated
- Use **`<Navigate>`** component for redirects
- Wrap protected content with **ProtectedRoute** component
- Check auth in ProtectedRoute, return children or redirect
- Use **Context API** for auth state management
- Save **intended destination** in location state
- Redirect to **original destination** after login
- Use **`replace: true`** to prevent going back
- Show **loading state** while checking auth
- **Role-based** protection: check user.role
- **Permission-based**: check specific permissions
- **Layout-based**: protect entire layout with nested routes
- **Guest routes**: redirect authenticated users away
- Token stored in **localStorage** or **cookies**
- Validate token on **app mount**
- **Auto-refresh** expired tokens
- Clear data on **logout**
- Common pattern: `<AuthProvider>` wraps entire app
- Common pattern: `useAuth()` hook for accessing auth state
- **JWT tokens** most common approach
- Include token in **API requests** (Authorization header)
- Handle **401 Unauthorized** responses globally
- Redirect to login on **token expiry**
- React Router v6 uses **`<Navigate>`**, v5 used **`<Redirect>`**

</details>

---

### 68. What are loader and action functions in React Router v6?

<details>
<summary>View Answer</summary>

**Loaders and Actions**

**Loaders** and **Actions** are React Router v6.4+ features that enable **data fetching** and **mutations** at the route level, before components render.

**Key concepts:**
- **Loader:** Fetch data **before** rendering route
- **Action:** Handle form submissions and mutations
- Move data logic **out of components**
- Better **loading states** and **error handling**
- Works with **`<Form>`** component

---

## Loaders

**Load data before component renders:**

```jsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom';

// Loader function
async function userLoader({ params }) {
  const response = await fetch(`/api/users/${params.id}`);
  if (!response.ok) {
    throw new Response('Not Found', { status: 404 });
  }
  return response.json();
}

// Route with loader
const router = createBrowserRouter([
  {
    path: '/users/:id',
    element: <UserProfile />,
    loader: userLoader,  // Called before rendering
  }
]);

function App() {
  return <RouterProvider router={router} />;
}
```

**Access loaded data:**
```jsx
import { useLoaderData } from 'react-router-dom';

function UserProfile() {
  const user = useLoaderData();  // Get data from loader
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

---

## Actions

**Handle form submissions:**

```jsx
import { createBrowserRouter, Form } from 'react-router-dom';

// Action function
async function createUserAction({ request }) {
  const formData = await request.formData();
  const user = {
    name: formData.get('name'),
    email: formData.get('email'),
  };
  
  const response = await fetch('/api/users', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(user),
  });
  
  if (!response.ok) {
    return { error: 'Failed to create user' };
  }
  
  return redirect('/users');
}

// Route with action
const router = createBrowserRouter([
  {
    path: '/users/new',
    element: <CreateUser />,
    action: createUserAction,  // Called on form submit
  }
]);

function CreateUser() {
  return (
    <Form method="post">
      <input name="name" placeholder="Name" required />
      <input name="email" type="email" placeholder="Email" required />
      <button type="submit">Create User</button>
    </Form>
  );
}
```

---

## Complete Example: Blog App

```jsx
import React from 'react';
import {
  createBrowserRouter,
  RouterProvider,
  Form,
  Link,
  useLoaderData,
  useActionData,
  redirect,
  useNavigation,
} from 'react-router-dom';

// ==================== Loaders ====================

// Load all posts
async function postsLoader() {
  const response = await fetch('/api/posts');
  if (!response.ok) {
    throw new Error('Failed to load posts');
  }
  return response.json();
}

// Load single post
async function postLoader({ params }) {
  const response = await fetch(`/api/posts/${params.id}`);
  if (!response.ok) {
    throw new Response('Post not found', { status: 404 });
  }
  return response.json();
}

// ==================== Actions ====================

// Create post
async function createPostAction({ request }) {
  const formData = await request.formData();
  
  const post = {
    title: formData.get('title'),
    content: formData.get('content'),
  };
  
  // Validation
  if (!post.title || post.title.length < 5) {
    return { error: 'Title must be at least 5 characters' };
  }
  
  try {
    const response = await fetch('/api/posts', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(post),
    });
    
    if (!response.ok) {
      throw new Error('Failed to create post');
    }
    
    const newPost = await response.json();
    return redirect(`/posts/${newPost.id}`);
  } catch (error) {
    return { error: error.message };
  }
}

// Update post
async function updatePostAction({ request, params }) {
  const formData = await request.formData();
  
  const post = {
    title: formData.get('title'),
    content: formData.get('content'),
  };
  
  const response = await fetch(`/api/posts/${params.id}`, {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(post),
  });
  
  if (!response.ok) {
    return { error: 'Failed to update post' };
  }
  
  return redirect(`/posts/${params.id}`);
}

// Delete post
async function deletePostAction({ params }) {
  const response = await fetch(`/api/posts/${params.id}`, {
    method: 'DELETE',
  });
  
  if (!response.ok) {
    return { error: 'Failed to delete post' };
  }
  
  return redirect('/posts');
}

// ==================== Components ====================

function PostsList() {
  const posts = useLoaderData();
  
  return (
    <div>
      <h1>Blog Posts</h1>
      <Link to="/posts/new">Create New Post</Link>
      
      <ul>
        {posts.map(post => (
          <li key={post.id}>
            <Link to={`/posts/${post.id}`}>{post.title}</Link>
          </li>
        ))}
      </ul>
    </div>
  );
}

function PostDetail() {
  const post = useLoaderData();
  
  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
      
      <Link to={`/posts/${post.id}/edit`}>Edit</Link>
      
      <Form method="delete">
        <button type="submit">Delete</button>
      </Form>
    </div>
  );
}

function CreatePost() {
  const actionData = useActionData();  // Get validation errors
  const navigation = useNavigation();  // Get loading state
  const isSubmitting = navigation.state === 'submitting';
  
  return (
    <div>
      <h1>Create New Post</h1>
      
      {actionData?.error && (
        <div className="error">{actionData.error}</div>
      )}
      
      <Form method="post">
        <input
          name="title"
          placeholder="Title"
          required
        />
        <textarea
          name="content"
          placeholder="Content"
          required
        />
        <button type="submit" disabled={isSubmitting}>
          {isSubmitting ? 'Creating...' : 'Create Post'}
        </button>
      </Form>
    </div>
  );
}

function EditPost() {
  const post = useLoaderData();
  const actionData = useActionData();
  const navigation = useNavigation();
  const isSubmitting = navigation.state === 'submitting';
  
  return (
    <div>
      <h1>Edit Post</h1>
      
      {actionData?.error && (
        <div className="error">{actionData.error}</div>
      )}
      
      <Form method="put">
        <input
          name="title"
          defaultValue={post.title}
          required
        />
        <textarea
          name="content"
          defaultValue={post.content}
          required
        />
        <button type="submit" disabled={isSubmitting}>
          {isSubmitting ? 'Saving...' : 'Save'}
        </button>
      </Form>
    </div>
  );
}

// ==================== Router ====================

const router = createBrowserRouter([
  {
    path: '/',
    element: <div><Link to="/posts">Go to Blog</Link></div>,
  },
  {
    path: '/posts',
    element: <PostsList />,
    loader: postsLoader,
  },
  {
    path: '/posts/new',
    element: <CreatePost />,
    action: createPostAction,
  },
  {
    path: '/posts/:id',
    element: <PostDetail />,
    loader: postLoader,
    action: deletePostAction,  // For delete form
  },
  {
    path: '/posts/:id/edit',
    element: <EditPost />,
    loader: postLoader,
    action: updatePostAction,
  },
]);

function App() {
  return <RouterProvider router={router} />;
}

export default App;
```

---

## Loader Features

### 1. Access route params

```jsx
async function userLoader({ params }) {
  const userId = params.id;
  const response = await fetch(`/api/users/${userId}`);
  return response.json();
}

const router = createBrowserRouter([
  {
    path: '/users/:id',
    loader: userLoader,
    element: <UserProfile />,
  }
]);
```

---

### 2. Access request object

```jsx
async function searchLoader({ request }) {
  const url = new URL(request.url);
  const query = url.searchParams.get('q');
  
  const response = await fetch(`/api/search?q=${query}`);
  return response.json();
}

const router = createBrowserRouter([
  {
    path: '/search',
    loader: searchLoader,
    element: <SearchResults />,
  }
]);

// Visit: /search?q=react
// loader receives request with full URL
```

---

### 3. Return multiple values

```jsx
async function dashboardLoader() {
  const [user, stats, notifications] = await Promise.all([
    fetch('/api/user').then(r => r.json()),
    fetch('/api/stats').then(r => r.json()),
    fetch('/api/notifications').then(r => r.json()),
  ]);
  
  return { user, stats, notifications };
}

function Dashboard() {
  const { user, stats, notifications } = useLoaderData();
  
  return (
    <div>
      <h1>Welcome, {user.name}</h1>
      <Stats data={stats} />
      <Notifications items={notifications} />
    </div>
  );
}
```

---

### 4. Throw errors

```jsx
async function userLoader({ params }) {
  const response = await fetch(`/api/users/${params.id}`);
  
  if (response.status === 404) {
    throw new Response('User not found', { status: 404 });
  }
  
  if (!response.ok) {
    throw new Error('Failed to load user');
  }
  
  return response.json();
}

// Handle errors with errorElement
const router = createBrowserRouter([
  {
    path: '/users/:id',
    loader: userLoader,
    element: <UserProfile />,
    errorElement: <ErrorPage />,
  }
]);
```

---

## Action Features

### 1. Access form data

```jsx
async function loginAction({ request }) {
  const formData = await request.formData();
  
  const username = formData.get('username');
  const password = formData.get('password');
  const remember = formData.get('remember') === 'on';
  
  // Login logic...
  return redirect('/dashboard');
}

<Form method="post">
  <input name="username" />
  <input name="password" type="password" />
  <input name="remember" type="checkbox" />
  <button type="submit">Login</button>
</Form>
```

---

### 2. Return validation errors

```jsx
async function createUserAction({ request }) {
  const formData = await request.formData();
  const email = formData.get('email');
  
  // Validation
  if (!email.includes('@')) {
    return { error: 'Invalid email address' };
  }
  
  // Create user...
  return redirect('/users');
}

function CreateUser() {
  const actionData = useActionData();
  
  return (
    <Form method="post">
      {actionData?.error && <p>{actionData.error}</p>}
      <input name="email" />
      <button type="submit">Create</button>
    </Form>
  );
}
```

---

### 3. Different actions based on intent

```jsx
async function postAction({ request }) {
  const formData = await request.formData();
  const intent = formData.get('intent');
  
  if (intent === 'delete') {
    // Delete post
    await fetch(`/api/posts/${formData.get('id')}`, { method: 'DELETE' });
    return redirect('/posts');
  }
  
  if (intent === 'like') {
    // Like post
    await fetch(`/api/posts/${formData.get('id')}/like`, { method: 'POST' });
    return null;  // Stay on page
  }
}

function Post() {
  return (
    <div>
      <Form method="post">
        <input type="hidden" name="id" value={post.id} />
        <button name="intent" value="like">Like</button>
        <button name="intent" value="delete">Delete</button>
      </Form>
    </div>
  );
}
```

---

## Loading States

**Track loading with `useNavigation()`:**

```jsx
import { useNavigation } from 'react-router-dom';

function App() {
  const navigation = useNavigation();
  
  return (
    <div>
      {navigation.state === 'loading' && <div>Loading...</div>}
      {navigation.state === 'submitting' && <div>Submitting...</div>}
      
      <Outlet />
    </div>
  );
}
```

**States:**
- `idle` - Nothing happening
- `loading` - Loader running
- `submitting` - Action running

---

## Revalidation

**Loaders automatically re-run after actions:**

```jsx
const router = createBrowserRouter([
  {
    path: '/posts',
    loader: postsLoader,  // Runs initially
    element: <PostsList />,
    children: [
      {
        path: 'new',
        action: createPostAction,  // After this runs...
        element: <CreatePost />,
      }
    ]
  }
]);

// Flow:
// 1. User submits form at /posts/new
// 2. createPostAction runs
// 3. Redirects to /posts
// 4. postsLoader re-runs (revalidation)
// 5. Fresh data displayed
```

---

## Error Handling

**Handle loader/action errors:**

```jsx
import { useRouteError } from 'react-router-dom';

function ErrorPage() {
  const error = useRouteError();
  
  return (
    <div>
      <h1>Oops!</h1>
      <p>{error.statusText || error.message}</p>
    </div>
  );
}

const router = createBrowserRouter([
  {
    path: '/users/:id',
    loader: userLoader,
    element: <UserProfile />,
    errorElement: <ErrorPage />,  // Shows on error
  }
]);
```

---

## Benefits

**1. Better loading states**
- No useState(true) in components
- Automatic loading indicators
- useNavigation() tracks loading

**2. Simpler components**
- No useEffect for data fetching
- No useState for data
- Components receive data via useLoaderData()

**3. Better error handling**
- Automatic error boundaries
- Centralized error handling
- useRouteError() for error info

**4. Automatic revalidation**
- Data refreshes after mutations
- No manual refetch needed
- Always fresh data

**5. Type-safe with TypeScript**
- Loaders typed properly
- Actions typed properly
- useLoaderData() infers types

---

## Best Practices

**1. Keep loaders focused**
```jsx
// ✅ Good - single responsibility
async function userLoader({ params }) {
  return fetch(`/api/users/${params.id}`).then(r => r.json());
}

// ❌ Avoid - too much logic
async function userLoader({ params }) {
  const user = await fetch(`/api/users/${params.id}`).then(r => r.json());
  const posts = await fetch(`/api/posts?user=${params.id}`).then(r => r.json());
  const friends = await fetch(`/api/friends?user=${params.id}`).then(r => r.json());
  return { user, posts, friends };
}
```

**2. Handle errors gracefully**
```jsx
async function loader({ params }) {
  const response = await fetch(`/api/data/${params.id}`);
  
  if (!response.ok) {
    throw new Response('Not found', { status: 404 });
  }
  
  return response.json();
}
```

**3. Return validation errors from actions**
```jsx
async function action({ request }) {
  const formData = await request.formData();
  
  // Validate
  const errors = validate(formData);
  if (errors) {
    return { errors };  // Return errors, don't redirect
  }
  
  // Process...
  return redirect('/success');
}
```

**4. Use FormData for forms**
```jsx
// ✅ Good - use FormData
const formData = await request.formData();
const name = formData.get('name');

// ❌ Avoid - manual JSON parsing
const body = await request.json();
```

---

**Interview Tips:**
- **Loaders** = fetch data **before** rendering
- **Actions** = handle **form submissions**
- Introduced in **React Router v6.4**
- Loaders run **before component renders**
- Actions run on **form submit**
- Access loader data with **`useLoaderData()`**
- Access action data with **`useActionData()`**
- Use **`<Form>`** component (not regular `<form>`)
- **`useNavigation()`** tracks loading state
- States: **idle**, **loading**, **submitting**
- Loaders receive **`{ params, request }`**
- Actions receive **`{ params, request }`**
- Get form data: **`await request.formData()`**
- Return **validation errors** from actions
- Throw **errors** from loaders for error handling
- Use **`errorElement`** to handle errors
- **`useRouteError()`** accesses error info
- **Automatic revalidation** after actions
- Loaders re-run after actions complete
- Use **`redirect()`** to navigate after action
- **Parallel data fetching** with Promise.all
- Better than **useEffect** for data fetching
- Simpler **loading states** (no useState)
- Works with **nested routes**
- Must use **`createBrowserRouter()`** (not `<BrowserRouter>`)
- More **declarative** than manual fetching

</details>

---

### 69. How do you handle 404 pages?

<details>
<summary>View Answer</summary>

**404 Pages**

A 404 page (Not Found page) is displayed when a user navigates to a route that **doesn't exist**. Proper 404 handling improves user experience and helps with navigation.

---

## Basic 404 Route

**Use wildcard `*` path to catch unmatched routes:**

```jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/contact" element={<Contact />} />
        
        {/* 404 - Must be last */}
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}

function NotFound() {
  return (
    <div>
      <h1>404 - Page Not Found</h1>
      <p>The page you're looking for doesn't exist.</p>
      <Link to="/">Go Home</Link>
    </div>
  );
}
```

**Important:** The `*` route must be **last** in your route list.

---

## Complete 404 Component

```jsx
import { Link, useLocation } from 'react-router-dom';

function NotFound() {
  const location = useLocation();
  
  return (
    <div className="not-found">
      <h1>404</h1>
      <h2>Page Not Found</h2>
      <p>
        The page <code>{location.pathname}</code> doesn't exist.
      </p>
      
      <div className="actions">
        <Link to="/" className="button">Go Home</Link>
        <Link to="/about" className="button">About Us</Link>
        <Link to="/contact" className="button">Contact</Link>
      </div>
    </div>
  );
}
```

**CSS:**
```css
.not-found {
  text-align: center;
  padding: 50px;
}

.not-found h1 {
  font-size: 120px;
  margin: 0;
  color: #e74c3c;
}

.not-found h2 {
  font-size: 32px;
  margin: 20px 0;
}

.not-found code {
  background: #f5f5f5;
  padding: 5px 10px;
  border-radius: 4px;
}

.not-found .actions {
  margin-top: 30px;
  display: flex;
  gap: 10px;
  justify-content: center;
}
```

---

## 404 with Nested Routes

**Place 404 at different nesting levels:**

```jsx
function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Layout />}>
          <Route index element={<Home />} />
          <Route path="about" element={<About />} />
          
          {/* Nested routes */}
          <Route path="blog" element={<BlogLayout />}>
            <Route index element={<BlogHome />} />
            <Route path=":slug" element={<BlogPost />} />
            {/* 404 for /blog/* */}
            <Route path="*" element={<BlogNotFound />} />
          </Route>
          
          {/* Global 404 */}
          <Route path="*" element={<NotFound />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
}

function BlogNotFound() {
  return (
    <div>
      <h1>Blog Post Not Found</h1>
      <Link to="/blog">Back to Blog</Link>
    </div>
  );
}
```

---

## 404 with Search Suggestions

**Suggest similar pages:**

```jsx
import { Link, useLocation } from 'react-router-dom';
import { useState, useEffect } from 'react';

function NotFound() {
  const location = useLocation();
  const [suggestions, setSuggestions] = useState([]);
  
  useEffect(() => {
    // Find similar routes
    const path = location.pathname;
    const allRoutes = [
      { path: '/products', label: 'Products' },
      { path: '/about', label: 'About' },
      { path: '/contact', label: 'Contact' },
      { path: '/blog', label: 'Blog' },
    ];
    
    // Simple similarity check
    const similar = allRoutes.filter(route => 
      path.toLowerCase().includes(route.path.slice(1).toLowerCase()) ||
      route.path.slice(1).toLowerCase().includes(path.slice(1).toLowerCase())
    );
    
    setSuggestions(similar);
  }, [location.pathname]);
  
  return (
    <div className="not-found">
      <h1>404 - Page Not Found</h1>
      <p>The page <code>{location.pathname}</code> doesn't exist.</p>
      
      {suggestions.length > 0 && (
        <div className="suggestions">
          <h3>Did you mean:</h3>
          <ul>
            {suggestions.map(route => (
              <li key={route.path}>
                <Link to={route.path}>{route.label}</Link>
              </li>
            ))}
          </ul>
        </div>
      )}
      
      <Link to="/">Go Home</Link>
    </div>
  );
}
```

---

## 404 with Redirect Timer

**Auto-redirect after countdown:**

```jsx
import { useEffect, useState } from 'react';
import { useNavigate, Link } from 'react-router-dom';

function NotFound() {
  const navigate = useNavigate();
  const [countdown, setCountdown] = useState(5);
  
  useEffect(() => {
    const timer = setInterval(() => {
      setCountdown(prev => prev - 1);
    }, 1000);
    
    const redirect = setTimeout(() => {
      navigate('/');
    }, 5000);
    
    return () => {
      clearInterval(timer);
      clearTimeout(redirect);
    };
  }, [navigate]);
  
  return (
    <div className="not-found">
      <h1>404 - Page Not Found</h1>
      <p>Redirecting to home page in {countdown} seconds...</p>
      <Link to="/">Go Home Now</Link>
    </div>
  );
}
```

---

## 404 with Navigation Back

**Let user go back to previous page:**

```jsx
import { useNavigate, Link, useLocation } from 'react-router-dom';

function NotFound() {
  const navigate = useNavigate();
  const location = useLocation();
  
  const handleGoBack = () => {
    // Check if there's history to go back to
    if (window.history.length > 2) {
      navigate(-1);
    } else {
      navigate('/');
    }
  };
  
  return (
    <div className="not-found">
      <h1>404 - Page Not Found</h1>
      <p>The page <code>{location.pathname}</code> doesn't exist.</p>
      
      <div className="actions">
        <button onClick={handleGoBack}>Go Back</button>
        <Link to="/">Go Home</Link>
      </div>
    </div>
  );
}
```

---

## Error Boundary for 404

**Handle route errors with error boundary:**

```jsx
import { useRouteError, Link } from 'react-router-dom';

function ErrorPage() {
  const error = useRouteError();
  
  // Check if it's a 404
  if (error.status === 404) {
    return (
      <div className="not-found">
        <h1>404 - Page Not Found</h1>
        <p>{error.statusText}</p>
        <Link to="/">Go Home</Link>
      </div>
    );
  }
  
  // Other errors
  return (
    <div className="error">
      <h1>Oops! Something went wrong</h1>
      <p>{error.message}</p>
      <Link to="/">Go Home</Link>
    </div>
  );
}

// Router configuration
const router = createBrowserRouter([
  {
    path: '/',
    element: <Layout />,
    errorElement: <ErrorPage />,  // Catches all errors
    children: [
      { index: true, element: <Home /> },
      { path: 'about', element: <About /> },
    ]
  }
]);
```

---

## 404 with Search

**Let users search for content:**

```jsx
import { useState } from 'react';
import { Link, useNavigate, useLocation } from 'react-router-dom';

function NotFound() {
  const location = useLocation();
  const navigate = useNavigate();
  const [searchQuery, setSearchQuery] = useState('');
  
  const handleSearch = (e) => {
    e.preventDefault();
    if (searchQuery.trim()) {
      navigate(`/search?q=${encodeURIComponent(searchQuery)}`);
    }
  };
  
  return (
    <div className="not-found">
      <h1>404 - Page Not Found</h1>
      <p>The page <code>{location.pathname}</code> doesn't exist.</p>
      
      <div className="search-section">
        <h3>Search for what you're looking for:</h3>
        <form onSubmit={handleSearch}>
          <input
            type="text"
            placeholder="Search..."
            value={searchQuery}
            onChange={(e) => setSearchQuery(e.target.value)}
          />
          <button type="submit">Search</button>
        </form>
      </div>
      
      <div className="quick-links">
        <h3>Quick Links:</h3>
        <ul>
          <li><Link to="/">Home</Link></li>
          <li><Link to="/products">Products</Link></li>
          <li><Link to="/about">About</Link></li>
          <li><Link to="/contact">Contact</Link></li>
        </ul>
      </div>
    </div>
  );
}
```

---

## Complete Example: Professional 404 Page

```jsx
import React from 'react';
import { Link, useLocation } from 'react-router-dom';
import './NotFound.css';

function NotFound() {
  const location = useLocation();
  
  return (
    <div className="not-found-page">
      <div className="not-found-content">
        {/* Large 404 Text */}
        <div className="error-code">
          <span className="four">4</span>
          <span className="zero">0</span>
          <span className="four">4</span>
        </div>
        
        {/* Error Message */}
        <h1>Page Not Found</h1>
        <p className="error-message">
          Sorry, the page <code>{location.pathname}</code> doesn't exist.
        </p>
        
        {/* Helpful Links */}
        <div className="helpful-links">
          <h2>Here are some helpful links:</h2>
          <div className="link-grid">
            <Link to="/" className="link-card">
              <span className="icon">🏠</span>
              <span className="label">Home</span>
            </Link>
            <Link to="/products" className="link-card">
              <span className="icon">🛍️</span>
              <span className="label">Products</span>
            </Link>
            <Link to="/about" className="link-card">
              <span className="icon">ℹ️</span>
              <span className="label">About Us</span>
            </Link>
            <Link to="/contact" className="link-card">
              <span className="icon">📧</span>
              <span className="label">Contact</span>
            </Link>
          </div>
        </div>
        
        {/* Primary Action */}
        <div className="primary-action">
          <Link to="/" className="btn-home">
            Take Me Home
          </Link>
        </div>
      </div>
    </div>
  );
}

export default NotFound;
```

**CSS (NotFound.css):**
```css
.not-found-page {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 20px;
}

.not-found-content {
  text-align: center;
  color: white;
  max-width: 800px;
}

.error-code {
  font-size: 150px;
  font-weight: bold;
  line-height: 1;
  margin-bottom: 20px;
}

.error-code .zero {
  display: inline-block;
  animation: spin 2s linear infinite;
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.not-found-content h1 {
  font-size: 48px;
  margin-bottom: 10px;
}

.error-message {
  font-size: 18px;
  margin-bottom: 40px;
  opacity: 0.9;
}

.error-message code {
  background: rgba(255, 255, 255, 0.2);
  padding: 5px 10px;
  border-radius: 4px;
  font-family: monospace;
}

.helpful-links h2 {
  font-size: 24px;
  margin-bottom: 20px;
}

.link-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 20px;
  margin-bottom: 40px;
}

.link-card {
  background: rgba(255, 255, 255, 0.1);
  padding: 20px;
  border-radius: 10px;
  text-decoration: none;
  color: white;
  transition: all 0.3s;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}

.link-card:hover {
  background: rgba(255, 255, 255, 0.2);
  transform: translateY(-5px);
}

.link-card .icon {
  font-size: 40px;
}

.link-card .label {
  font-size: 16px;
  font-weight: 500;
}

.btn-home {
  display: inline-block;
  padding: 15px 40px;
  background: white;
  color: #667eea;
  text-decoration: none;
  border-radius: 50px;
  font-weight: bold;
  font-size: 18px;
  transition: all 0.3s;
}

.btn-home:hover {
  transform: scale(1.05);
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
}
```

---

## Common Patterns

### Pattern 1: Different 404 for Different Sections

```jsx
function App() {
  return (
    <Routes>
      {/* Public 404 */}
      <Route path="/" element={<PublicLayout />}>
        <Route index element={<Home />} />
        <Route path="about" element={<About />} />
        <Route path="*" element={<PublicNotFound />} />
      </Route>
      
      {/* Admin 404 */}
      <Route path="/admin" element={<AdminLayout />}>
        <Route index element={<AdminDashboard />} />
        <Route path="users" element={<Users />} />
        <Route path="*" element={<AdminNotFound />} />
      </Route>
    </Routes>
  );
}
```

---

### Pattern 2: 404 with Analytics

```jsx
import { useEffect } from 'react';
import { useLocation } from 'react-router-dom';

function NotFound() {
  const location = useLocation();
  
  useEffect(() => {
    // Track 404 in analytics
    if (window.gtag) {
      window.gtag('event', 'page_not_found', {
        page_path: location.pathname,
      });
    }
    
    // Or send to your backend
    fetch('/api/log-404', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        path: location.pathname,
        referrer: document.referrer,
        timestamp: new Date().toISOString(),
      })
    });
  }, [location.pathname]);
  
  return (
    <div className="not-found">
      <h1>404 - Page Not Found</h1>
      <Link to="/">Go Home</Link>
    </div>
  );
}
```

---

### Pattern 3: Dynamic 404 Based on Route

```jsx
import { useLocation, Link } from 'react-router-dom';

function NotFound() {
  const location = useLocation();
  const path = location.pathname;
  
  // Check what section user was trying to access
  let message = 'Page not found.';
  let suggestion = '/';
  let suggestionLabel = 'Go Home';
  
  if (path.startsWith('/products')) {
    message = 'Product not found.';
    suggestion = '/products';
    suggestionLabel = 'View All Products';
  } else if (path.startsWith('/blog')) {
    message = 'Blog post not found.';
    suggestion = '/blog';
    suggestionLabel = 'View All Posts';
  } else if (path.startsWith('/users')) {
    message = 'User not found.';
    suggestion = '/users';
    suggestionLabel = 'View All Users';
  }
  
  return (
    <div className="not-found">
      <h1>404</h1>
      <p>{message}</p>
      <p>The page <code>{path}</code> doesn't exist.</p>
      <Link to={suggestion}>{suggestionLabel}</Link>
    </div>
  );
}
```

---

## Best Practices

**1. Always have a catch-all route**
```jsx
<Route path="*" element={<NotFound />} />
```

**2. Place 404 route last**
```jsx
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/about" element={<About />} />
  <Route path="*" element={<NotFound />} />  {/* Last! */}
</Routes>
```

**3. Show current path**
```jsx
<p>The page <code>{location.pathname}</code> doesn't exist.</p>
```

**4. Provide navigation options**
```jsx
<Link to="/">Home</Link>
<Link to="/about">About</Link>
<Link to="/contact">Contact</Link>
```

**5. Keep design consistent**
- Use same header/footer as rest of site
- Match brand colors
- Use familiar navigation

**6. Be helpful**
- Suggest similar pages
- Provide search
- Show popular pages
- Let user go back

**7. Log 404s**
- Track in analytics
- Find broken links
- Fix common issues

---

## Server Configuration

**Important:** For BrowserRouter, configure server to serve index.html for all routes:

**Netlify (_redirects):**
```
/*    /index.html   200
```

**Vercel (vercel.json):**
```json
{
  "rewrites": [
    { "source": "/(.*)", "destination": "/" }
  ]
}
```

**Apache (.htaccess):**
```apache
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.html [L]
```

**Nginx:**
```nginx
location / {
  try_files $uri $uri/ /index.html;
}
```

---

**Interview Tips:**
- 404 page = **page not found** error
- Use **`path="*"`** to catch unmatched routes
- Wildcard route must be **last** in route list
- `*` matches **any unmatched path**
- Access current path with **`useLocation()`**
- Show **helpful navigation** links
- Provide **search** functionality
- Suggest **similar pages**
- Let user **go back** or go home
- Keep **design consistent** with site
- **Log 404s** for analytics
- Track broken links
- Can have **different 404s** for different sections
- Use **nested routes** for section-specific 404s
- Auto-redirect after **countdown** (optional)
- Use **error boundaries** for error handling
- **`errorElement`** in createBrowserRouter
- **`useRouteError()`** to access error info
- Server must return **index.html** for all routes
- Configure **.htaccess**, **nginx**, etc.
- BrowserRouter needs **server configuration**
- HashRouter **doesn't need** server config
- Show **current pathname** to help user
- Professional 404 = helpful + branded
- Common to show **popular pages**
- Common to provide **search box**
- Don't just say "404" - be helpful!

</details>

---

### 70. What is the useNavigate hook?

<details>
<summary>View Answer</summary>

**useNavigate Hook**

The `useNavigate()` hook returns a **function** that lets you navigate programmatically (navigate from code, not from link clicks).

**Introduced in React Router v6** (replaces `useHistory` from v5)

---

## Basic Usage

```jsx
import { useNavigate } from 'react-router-dom';

function MyComponent() {
  const navigate = useNavigate();
  
  const handleClick = () => {
    navigate('/about');
  };
  
  return <button onClick={handleClick}>Go to About</button>;
}
```

---

## Syntax

```jsx
const navigate = useNavigate();

// Navigate to a path
navigate('/path');

// Navigate with options
navigate('/path', { replace: true, state: { ... } });

// Go back
navigate(-1);

// Go forward
navigate(1);

// Go back 2 pages
navigate(-2);
```

---

## Navigation Methods

### 1. Navigate to Path

```jsx
import { useNavigate } from 'react-router-dom';

function HomePage() {
  const navigate = useNavigate();
  
  return (
    <div>
      <button onClick={() => navigate('/about')}>About</button>
      <button onClick={() => navigate('/contact')}>Contact</button>
      <button onClick={() => navigate('/products/123')}>Product 123</button>
    </div>
  );
}
```

---

### 2. Navigate with Replace

**Replace current history entry (can't go back):**

```jsx
const navigate = useNavigate();

navigate('/dashboard', { replace: true });

// User can't press back button to return
```

**Use cases:**
- After login (don't let user go back to login page)
- After logout
- After form submission
- Permanent redirects

---

### 3. Navigate with State

**Pass data to next page:**

```jsx
import { useNavigate, useLocation } from 'react-router-dom';

// Sender
function ProductList() {
  const navigate = useNavigate();
  
  const handleSelectProduct = (product) => {
    navigate('/product-details', {
      state: { product }  // Pass data
    });
  };
  
  return (
    <div>
      {products.map(product => (
        <button key={product.id} onClick={() => handleSelectProduct(product)}>
          {product.name}
        </button>
      ))}
    </div>
  );
}

// Receiver
function ProductDetails() {
  const location = useLocation();
  const product = location.state?.product;  // Access data
  
  if (!product) {
    return <div>Product not found</div>;
  }
  
  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.price}</p>
    </div>
  );
}
```

---

### 4. Go Back/Forward

**Navigate through history:**

```jsx
const navigate = useNavigate();

// Go back one page
navigate(-1);

// Go forward one page
navigate(1);

// Go back two pages
navigate(-2);
```

**Example:**
```jsx
function ProductDetails() {
  const navigate = useNavigate();
  
  return (
    <div>
      <button onClick={() => navigate(-1)}>
        ← Back
      </button>
      <h1>Product Details</h1>
    </div>
  );
}
```

---

### 5. Relative Navigation

**Navigate relative to current route:**

```jsx
const navigate = useNavigate();

// From /products/123
navigate('..');        // → /products
navigate('../456');    // → /products/456
navigate('edit');      // → /products/123/edit
```

---

## Common Use Cases

### Use Case 1: Form Submission

```jsx
import { useState } from 'react';
import { useNavigate } from 'react-router-dom';

function CreatePost() {
  const navigate = useNavigate();
  const [title, setTitle] = useState('');
  const [content, setContent] = useState('');
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    try {
      const response = await fetch('/api/posts', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ title, content })
      });
      
      const post = await response.json();
      
      // Navigate to new post
      navigate(`/posts/${post.id}`);
    } catch (error) {
      console.error('Failed to create post:', error);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        value={title}
        onChange={(e) => setTitle(e.target.value)}
        placeholder="Title"
      />
      <textarea
        value={content}
        onChange={(e) => setContent(e.target.value)}
        placeholder="Content"
      />
      <button type="submit">Create</button>
    </form>
  );
}
```

---

### Use Case 2: Authentication

```jsx
import { useNavigate, useLocation } from 'react-router-dom';

function Login() {
  const navigate = useNavigate();
  const location = useLocation();
  
  // Get intended destination
  const from = location.state?.from?.pathname || '/dashboard';
  
  const handleLogin = async (e) => {
    e.preventDefault();
    
    try {
      await loginUser(username, password);
      
      // Navigate to intended destination
      navigate(from, { replace: true });
    } catch (error) {
      alert('Login failed');
    }
  };
  
  return <form onSubmit={handleLogin}>{/* form fields */}</form>;
}
```

---

### Use Case 3: Conditional Navigation

```jsx
import { useNavigate } from 'react-router-dom';

function Checkout() {
  const navigate = useNavigate();
  const cart = useCart();
  const user = useAuth();
  
  const handleCheckout = () => {
    // Check if cart is empty
    if (cart.items.length === 0) {
      alert('Cart is empty');
      navigate('/products');
      return;
    }
    
    // Check if user is logged in
    if (!user) {
      navigate('/login', {
        state: { from: '/checkout' }
      });
      return;
    }
    
    // Proceed to payment
    navigate('/payment');
  };
  
  return <button onClick={handleCheckout}>Checkout</button>;
}
```

---

### Use Case 4: Delete with Redirect

```jsx
import { useNavigate, useParams } from 'react-router-dom';

function PostDetail() {
  const navigate = useNavigate();
  const { id } = useParams();
  
  const handleDelete = async () => {
    if (!window.confirm('Delete this post?')) {
      return;
    }
    
    try {
      await fetch(`/api/posts/${id}`, { method: 'DELETE' });
      
      // Navigate to list, can't go back
      navigate('/posts', {
        replace: true,
        state: { message: 'Post deleted successfully' }
      });
    } catch (error) {
      alert('Failed to delete post');
    }
  };
  
  return (
    <div>
      <h1>Post Details</h1>
      <button onClick={handleDelete}>Delete</button>
    </div>
  );
}
```

---

### Use Case 5: Timed Redirect

```jsx
import { useEffect } from 'react';
import { useNavigate } from 'react-router-dom';

function SuccessPage() {
  const navigate = useNavigate();
  
  useEffect(() => {
    // Redirect after 3 seconds
    const timer = setTimeout(() => {
      navigate('/dashboard');
    }, 3000);
    
    return () => clearTimeout(timer);
  }, [navigate]);
  
  return (
    <div>
      <h1>Success!</h1>
      <p>Redirecting in 3 seconds...</p>
    </div>
  );
}
```

---

## Options Object

**All available options:**

```jsx
navigate(to, options);
```

**Options:**
```jsx
{
  replace: boolean,              // Replace history entry
  state: any,                    // Pass state to next location
  preventScrollReset: boolean,   // Don't scroll to top
  relative: 'route' | 'path'     // Relative resolution type
}
```

**Examples:**
```jsx
// Basic navigation
navigate('/about');

// With replace
navigate('/dashboard', { replace: true });

// With state
navigate('/details', {
  state: { id: 123, name: 'Product' }
});

// Multiple options
navigate('/success', {
  replace: true,
  state: { message: 'Saved!' }
});

// Prevent scroll reset
navigate('/page', {
  preventScrollReset: true
});
```

---

## React Router v5 vs v6

**v5 used `useHistory()`:**
```jsx
import { useHistory } from 'react-router-dom';

function MyComponent() {
  const history = useHistory();
  
  history.push('/about');
  history.replace('/dashboard');
  history.goBack();
  history.goForward();
  history.go(-2);
}
```

**v6 uses `useNavigate()`:**
```jsx
import { useNavigate } from 'react-router-dom';

function MyComponent() {
  const navigate = useNavigate();
  
  navigate('/about');
  navigate('/dashboard', { replace: true });
  navigate(-1);  // goBack
  navigate(1);   // goForward
  navigate(-2);  // go(-2)
}
```

**Migration:**
| v5 | v6 |
|----|----|
| `history.push('/path')` | `navigate('/path')` |
| `history.replace('/path')` | `navigate('/path', { replace: true })` |
| `history.goBack()` | `navigate(-1)` |
| `history.goForward()` | `navigate(1)` |
| `history.go(n)` | `navigate(n)` |

---

## Best Practices

**1. Add navigate to useEffect dependencies**
```jsx
useEffect(() => {
  if (condition) {
    navigate('/somewhere');
  }
}, [condition, navigate]);  // Include navigate
```

**2. Don't navigate during render**
```jsx
// ❌ Wrong - infinite loop
function MyComponent() {
  const navigate = useNavigate();
  navigate('/somewhere');  // Don't do this!
  return <div>Content</div>;
}

// ✅ Correct - use useEffect
function MyComponent() {
  const navigate = useNavigate();
  
  useEffect(() => {
    navigate('/somewhere');
  }, [navigate]);
  
  return <div>Content</div>;
}
```

**3. Use replace for auth redirects**
```jsx
// After login
navigate('/dashboard', { replace: true });

// After logout
navigate('/login', { replace: true });
```

**4. Pass state for context**
```jsx
navigate('/error', {
  state: { message: 'Something went wrong' }
});
```

**5. Handle navigation errors**
```jsx
try {
  await submitData();
  navigate('/success');
} catch (error) {
  navigate('/error', {
    state: { error: error.message }
  });
}
```

---

## Imperative vs Declarative

**Imperative (useNavigate):**
```jsx
function MyComponent() {
  const navigate = useNavigate();
  
  const handleClick = () => {
    navigate('/about');
  };
  
  return <button onClick={handleClick}>Go</button>;
}
```

**Declarative (<Navigate>):**
```jsx
import { Navigate } from 'react-router-dom';

function MyComponent({ isAuthenticated }) {
  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }
  
  return <div>Protected Content</div>;
}
```

**When to use which:**
- **useNavigate:** Event handlers, after async operations, in useEffect
- **<Navigate>:** Render logic, conditional redirects, declarative redirects

---

## Common Mistakes

**❌ Mistake 1: Calling navigate during render**
```jsx
function MyComponent() {
  const navigate = useNavigate();
  navigate('/somewhere');  // Causes infinite loop!
  return <div>Content</div>;
}
```

**❌ Mistake 2: Not handling async errors**
```jsx
await saveData();
navigate('/success');  // What if saveData fails?
```

**❌ Mistake 3: Forgetting replace for redirects**
```jsx
// After login
navigate('/dashboard');  // User can go back to login!

// Better:
navigate('/dashboard', { replace: true });
```

---

## TypeScript

**Type-safe navigation:**

```typescript
import { useNavigate } from 'react-router-dom';

function MyComponent() {
  const navigate = useNavigate();
  
  // TypeScript infers types
  navigate('/about');
  navigate(-1);
  navigate('/path', {
    replace: true,
    state: { id: 123 }
  });
  
  return <div>Content</div>;
}
```

---

**Interview Tips:**
- **useNavigate()** hook for programmatic navigation
- Returns **function** to navigate
- Introduced in **React Router v6**
- Replaces **useHistory()** from v5
- **`navigate(path)`** - navigate to path
- **`navigate(-1)`** - go back
- **`navigate(1)`** - go forward
- **`navigate(n)`** - go n entries in history
- **Second argument** = options object
- **`replace: true`** - replace history entry
- **`state`** - pass data to next page
- Access state with **`useLocation().state`**
- Use after **form submission**, **login**, **data operations**
- Use in **event handlers** and **useEffect**
- **Don't call during render** (causes infinite loop)
- Add to **useEffect dependencies**
- Use **replace** for auth redirects
- Can navigate **relatively** (../, edit, etc.)
- **Imperative** (useNavigate) vs **Declarative** (`<Navigate>`)
- Use useNavigate for **actions** and **async**
- Use `<Navigate>` for **conditional rendering**
- **v5:** history.push() → **v6:** navigate()
- **v5:** history.goBack() → **v6:** navigate(-1)
- Simpler API than v5
- Fewer methods to remember
- More intuitive than useHistory

</details>
