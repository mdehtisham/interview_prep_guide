# Advanced Patterns & Architecture

> Expert / Architect Level (5+ years)

---

## Questions

161. How do you implement microfrontends with React?
162. What is Module Federation in Webpack?
163. How do you design a scalable React architecture?
164. What is Clean Architecture in React context?
165. How do you implement error boundaries effectively?
166. What is the Error Boundary pattern and its limitations?
167. How do you handle asynchronous errors in React?
168. What is Atomic Design methodology?
169. How do you implement feature flags in React?
170. What is Progressive Enhancement in React apps?

---

## Detailed Answers

### 161. How do you implement microfrontends with React?

<details>
<summary>View Answer</summary>

**Microfrontends** is an architectural pattern that extends microservices principles to frontend development, where the UI is composed of multiple independent applications owned by different teams. Each microfrontend can be developed, tested, and deployed independently.

#### Core Concepts

**What are Microfrontends?**
- Independent frontend applications
- Each owned by separate teams
- Can use different frameworks (React, Vue, Angular)
- Integrated at runtime into a single application
- Share common infrastructure but remain loosely coupled

**Benefits:**
- Team autonomy and faster development
- Independent deployments
- Technology diversity (gradual migrations)
- Isolated failures
- Easier scaling of teams

**Challenges:**
- Increased complexity
- Performance overhead
- Shared dependencies
- Consistent UX across apps
- Testing integration

---

#### Implementation Approaches

**1. Build-Time Integration (NPM Packages)**

Publish microfrontends as npm packages and install them.

```json
// package.json of host app
{
  "dependencies": {
    "@company/header-mfe": "^1.0.0",
    "@company/products-mfe": "^2.1.0",
    "@company/checkout-mfe": "^1.5.0"
  }
}
```

```tsx
// Host application
import Header from '@company/header-mfe';
import Products from '@company/products-mfe';
import Checkout from '@company/checkout-mfe';

function App() {
  return (
    <div>
      <Header />
      <Routes>
        <Route path="/products" element={<Products />} />
        <Route path="/checkout" element={<Checkout />} />
      </Routes>
    </div>
  );
}
```

**Pros:**
- Simple to implement
- Type safety with TypeScript
- Easy to share code

**Cons:**
- Not true runtime composition
- Need to rebuild/redeploy host for updates
- All code bundled together

---

**2. Runtime Integration via Module Federation (Webpack 5)**

Load microfrontends dynamically at runtime.

**Container/Host App (webpack.config.js):**
```javascript
const ModuleFederationPlugin = require('webpack/lib/container/ModuleFederationPlugin');

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'container',
      remotes: {
        header: 'header@http://localhost:3001/remoteEntry.js',
        products: 'products@http://localhost:3002/remoteEntry.js',
        checkout: 'checkout@http://localhost:3003/remoteEntry.js',
      },
      shared: {
        react: { singleton: true, requiredVersion: '^18.0.0' },
        'react-dom': { singleton: true, requiredVersion: '^18.0.0' },
        'react-router-dom': { singleton: true },
      },
    }),
  ],
};
```

**Microfrontend App (webpack.config.js):**
```javascript
const ModuleFederationPlugin = require('webpack/lib/container/ModuleFederationPlugin');

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'products',
      filename: 'remoteEntry.js',
      exposes: {
        './ProductsApp': './src/ProductsApp',
        './ProductList': './src/components/ProductList',
      },
      shared: {
        react: { singleton: true, requiredVersion: '^18.0.0' },
        'react-dom': { singleton: true, requiredVersion: '^18.0.0' },
      },
    }),
  ],
};
```

**Usage in Host:**
```tsx
import React, { lazy, Suspense } from 'react';

// Dynamic import
const ProductsApp = lazy(() => import('products/ProductsApp'));
const Header = lazy(() => import('header/Header'));

function App() {
  return (
    <div>
      <Suspense fallback={<div>Loading Header...</div>}>
        <Header />
      </Suspense>
      
      <Suspense fallback={<div>Loading Products...</div>}>
        <ProductsApp />
      </Suspense>
    </div>
  );
}
```

---

**3. iframe Integration**

Load microfrontends in iframes.

```tsx
function App() {
  return (
    <div>
      <iframe
        src="http://localhost:3001/header"
        width="100%"
        height="80"
        frameBorder="0"
      />
      
      <iframe
        src="http://localhost:3002/products"
        width="100%"
        height="600"
        frameBorder="0"
      />
    </div>
  );
}
```

**Communication via postMessage:**
```tsx
// Parent (Host)
function App() {
  const iframeRef = useRef<HTMLIFrameElement>(null);

  const sendMessage = (data: any) => {
    iframeRef.current?.contentWindow?.postMessage(data, '*');
  };

  useEffect(() => {
    const handleMessage = (event: MessageEvent) => {
      if (event.origin !== 'http://localhost:3001') return;
      console.log('Received:', event.data);
    };

    window.addEventListener('message', handleMessage);
    return () => window.removeEventListener('message', handleMessage);
  }, []);

  return <iframe ref={iframeRef} src="http://localhost:3001" />;
}

// Child (Microfrontend)
function ChildApp() {
  useEffect(() => {
    // Send message to parent
    window.parent.postMessage({ type: 'READY' }, '*');

    // Listen to parent messages
    const handleMessage = (event: MessageEvent) => {
      console.log('Received from parent:', event.data);
    };

    window.addEventListener('message', handleMessage);
    return () => window.removeEventListener('message', handleMessage);
  }, []);

  return <div>Child App</div>;
}
```

**Pros:**
- Complete isolation
- Technology agnostic
- Easy security boundaries

**Cons:**
- Performance overhead
- Complex communication
- Routing challenges
- SEO issues

---

**4. Web Components**

Wrap React apps as Web Components.

**Create Web Component:**
```tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import ProductList from './ProductList';

class ProductsWebComponent extends HTMLElement {
  connectedCallback() {
    const mountPoint = document.createElement('div');
    this.attachShadow({ mode: 'open' }).appendChild(mountPoint);

    const root = ReactDOM.createRoot(mountPoint);
    root.render(<ProductList />);
  }

  disconnectedCallback() {
    // Cleanup
  }
}

customElements.define('products-app', ProductsWebComponent);
```

**Usage:**
```html
<!-- Can be used anywhere -->
<products-app></products-app>
```

```tsx
// In React
function App() {
  return (
    <div>
      <header-app />
      <products-app />
      <footer-app />
    </div>
  );
}
```

---

#### Complete Module Federation Example

**Shared Types Package:**
```typescript
// @company/shared-types/index.ts
export interface User {
  id: string;
  name: string;
  email: string;
}

export interface NavigationEvent {
  path: string;
  data?: any;
}

export interface MicroFrontendProps {
  user?: User;
  onNavigate?: (event: NavigationEvent) => void;
}
```

**Host Application:**
```tsx
// host/src/App.tsx
import React, { lazy, Suspense, useState } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import type { User, NavigationEvent } from '@company/shared-types';

const Header = lazy(() => import('header/Header'));
const Products = lazy(() => import('products/App'));
const Checkout = lazy(() => import('checkout/App'));

function App() {
  const [user, setUser] = useState<User | undefined>();

  const handleNavigate = (event: NavigationEvent) => {
    console.log('Navigate to:', event.path);
    // Handle navigation
  };

  return (
    <BrowserRouter>
      <div className="app">
        <Suspense fallback={<div>Loading...</div>}>
          <Header user={user} onNavigate={handleNavigate} />
        </Suspense>

        <main>
          <Suspense fallback={<div>Loading content...</div>}>
            <Routes>
              <Route
                path="/products/*"
                element={<Products user={user} onNavigate={handleNavigate} />}
              />
              <Route
                path="/checkout/*"
                element={<Checkout user={user} onNavigate={handleNavigate} />}
              />
            </Routes>
          </Suspense>
        </main>
      </div>
    </BrowserRouter>
  );
}

export default App;
```

**Products Microfrontend:**
```tsx
// products/src/App.tsx
import React from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import type { MicroFrontendProps } from '@company/shared-types';
import ProductList from './components/ProductList';
import ProductDetail from './components/ProductDetail';

function ProductsApp({ user, onNavigate }: MicroFrontendProps) {
  return (
    <Routes>
      <Route path="/" element={<ProductList user={user} />} />
      <Route path="/:id" element={<ProductDetail user={user} />} />
    </Routes>
  );
}

export default ProductsApp;
```

```javascript
// products/webpack.config.js
const ModuleFederationPlugin = require('webpack/lib/container/ModuleFederationPlugin');
const deps = require('./package.json').dependencies;

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'products',
      filename: 'remoteEntry.js',
      exposes: {
        './App': './src/App',
      },
      shared: {
        react: {
          singleton: true,
          requiredVersion: deps.react,
        },
        'react-dom': {
          singleton: true,
          requiredVersion: deps['react-dom'],
        },
        'react-router-dom': {
          singleton: true,
          requiredVersion: deps['react-router-dom'],
        },
      },
    }),
  ],
};
```

---

#### Communication Between Microfrontends

**1. Custom Events:**
```tsx
// Emit event from MFE
function ProductCard({ product }: { product: Product }) {
  const handleAddToCart = () => {
    const event = new CustomEvent('addToCart', {
      detail: { productId: product.id, quantity: 1 },
    });
    window.dispatchEvent(event);
  };

  return <button onClick={handleAddToCart}>Add to Cart</button>;
}

// Listen in another MFE or host
function CartBadge() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const handleAddToCart = (event: Event) => {
      const customEvent = event as CustomEvent;
      setCount(prev => prev + 1);
      console.log('Added:', customEvent.detail);
    };

    window.addEventListener('addToCart', handleAddToCart);
    return () => window.removeEventListener('addToCart', handleAddToCart);
  }, []);

  return <span className="cart-badge">{count}</span>;
}
```

**2. Shared Event Bus:**
```typescript
// shared/EventBus.ts
type EventCallback = (data: any) => void;

class EventBus {
  private events: Map<string, EventCallback[]> = new Map();

  subscribe(event: string, callback: EventCallback): () => void {
    if (!this.events.has(event)) {
      this.events.set(event, []);
    }
    this.events.get(event)!.push(callback);

    // Return unsubscribe function
    return () => {
      const callbacks = this.events.get(event)!;
      const index = callbacks.indexOf(callback);
      if (index > -1) {
        callbacks.splice(index, 1);
      }
    };
  }

  publish(event: string, data?: any): void {
    const callbacks = this.events.get(event) || [];
    callbacks.forEach(callback => callback(data));
  }
}

export const eventBus = new EventBus();
```

**Usage:**
```tsx
import { eventBus } from '@company/shared/EventBus';

// Publish from one MFE
function ProductList() {
  const handleProductClick = (product: Product) => {
    eventBus.publish('product:selected', product);
  };

  return <div>...</div>;
}

// Subscribe in another MFE
function ProductDetails() {
  useEffect(() => {
    const unsubscribe = eventBus.subscribe('product:selected', (product) => {
      console.log('Product selected:', product);
    });

    return unsubscribe;
  }, []);

  return <div>...</div>;
}
```

**3. Shared State (Context/Store):**
```tsx
// shared/GlobalStore.ts
import { createContext, useContext, useState, ReactNode } from 'react';

interface GlobalState {
  user?: User;
  cart: CartItem[];
}

interface GlobalContextType {
  state: GlobalState;
  updateUser: (user: User) => void;
  addToCart: (item: CartItem) => void;
}

const GlobalContext = createContext<GlobalContextType | undefined>(undefined);

export function GlobalProvider({ children }: { children: ReactNode }) {
  const [state, setState] = useState<GlobalState>({
    cart: [],
  });

  const updateUser = (user: User) => {
    setState(prev => ({ ...prev, user }));
  };

  const addToCart = (item: CartItem) => {
    setState(prev => ({ ...prev, cart: [...prev.cart, item] }));
  };

  return (
    <GlobalContext.Provider value={{ state, updateUser, addToCart }}>
      {children}
    </GlobalContext.Provider>
  );
}

export function useGlobalState() {
  const context = useContext(GlobalContext);
  if (!context) throw new Error('Must use within GlobalProvider');
  return context;
}
```

**Host wraps all MFEs:**
```tsx
import { GlobalProvider } from '@company/shared/GlobalStore';

function App() {
  return (
    <GlobalProvider>
      <Header />
      <Routes>
        <Route path="/products/*" element={<Products />} />
        <Route path="/cart/*" element={<Cart />} />
      </Routes>
    </GlobalProvider>
  );
}
```

---

#### Routing Strategies

**1. Host-Controlled Routing:**
```tsx
// Host controls all routes
function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/products/*" element={<ProductsMFE />} />
        <Route path="/checkout/*" element={<CheckoutMFE />} />
      </Routes>
    </BrowserRouter>
  );
}
```

**2. Delegated Routing:**
```tsx
// Each MFE handles its own sub-routes
function ProductsMFE() {
  return (
    <Routes>
      <Route path="/" element={<ProductList />} />
      <Route path="/:id" element={<ProductDetail />} />
      <Route path="/:id/reviews" element={<ProductReviews />} />
    </Routes>
  );
}
```

---

#### Styling Strategies

**1. CSS Modules (Scoped):**
```tsx
import styles from './ProductCard.module.css';

function ProductCard() {
  return <div className={styles.card}>...</div>;
}
```

**2. CSS-in-JS (Scoped):**
```tsx
import styled from 'styled-components';

const Card = styled.div`
  padding: 20px;
  border: 1px solid #ccc;
`;
```

**3. Design Tokens (Shared):**
```typescript
// @company/design-tokens
export const colors = {
  primary: '#007bff',
  secondary: '#6c757d',
};

export const spacing = {
  sm: '8px',
  md: '16px',
  lg: '24px',
};
```

---

#### Best Practices

**1. Define Clear Boundaries:**
```
✅ One MFE per business domain
✅ Independent teams and codebases
✅ Clear ownership

❌ Too many small MFEs
❌ Shared business logic across MFEs
```

**2. Shared Dependencies:**
```javascript
// Share common libraries as singletons
shared: {
  react: { singleton: true },
  'react-dom': { singleton: true },
  'react-router-dom': { singleton: true },
}
```

**3. Versioning Strategy:**
```
- Semantic versioning for shared libraries
- Backward compatibility for APIs
- Gradual rollouts
- Feature flags for new features
```

**4. Performance:**
```
- Code splitting per MFE
- Lazy load non-critical MFEs
- Shared chunk optimization
- Monitor bundle sizes
```

**5. Error Handling:**
```tsx
// Wrap each MFE in error boundary
function App() {
  return (
    <div>
      <ErrorBoundary fallback={<HeaderFallback />}>
        <Header />
      </ErrorBoundary>
      
      <ErrorBoundary fallback={<div>Products unavailable</div>}>
        <Products />
      </ErrorBoundary>
    </div>
  );
}
```

---

#### Testing

**Integration Testing:**
```typescript
import { render, screen } from '@testing-library/react';

describe('Host + Products MFE', () => {
  it('loads products microfrontend', async () => {
    render(<App />);
    
    // Wait for MFE to load
    expect(await screen.findByText('Product List')).toBeInTheDocument();
  });

  it('communicates between MFEs', async () => {
    render(<App />);
    
    // Click in products MFE
    fireEvent.click(screen.getByText('Add to Cart'));
    
    // Verify cart MFE updates
    expect(await screen.findByText('Cart (1)')).toBeInTheDocument();
  });
});
```

---

#### When to Use Microfrontends

**✅ Good Use Cases:**
- Large applications with multiple teams
- Different release cycles needed
- Gradual migration from legacy apps
- Teams with different tech stacks
- Long-term scalability requirements

**❌ Avoid When:**
- Small applications or teams
- Tight coupling between features
- Performance is critical
- Simple monoliths work fine
- Team lacks experience with distributed systems

---

#### Interview Tips

✅ **Key Points:**
- Architecture pattern for scaling teams
- Independent deployment and development
- Multiple integration strategies (Module Federation, iframes, build-time)
- Trade-off: complexity vs team autonomy
- Communication via events, shared state, or props

✅ **When to Mention:**
- Large-scale applications
- Enterprise architecture
- Team organization and scaling
- Technology migration strategies
- Monolith to distributed architecture

✅ **Common Follow-ups:**
- "What's the best way to share state between microfrontends?"
- "How do you handle routing?"
- "What about performance implications?"
- "How to test microfrontend integration?"
- "When would you NOT use microfrontends?"

</details>

---

### 162. What is Module Federation in Webpack?

<details>
<summary>View Answer</summary>

**Module Federation** is a Webpack 5 feature that enables JavaScript applications to dynamically load code from other independently built and deployed applications at runtime. It's the most powerful approach for implementing microfrontends with minimal configuration.

#### Core Concepts

**What is Module Federation?**
- Webpack 5 plugin for runtime code sharing
- Load remote modules dynamically
- Share dependencies between applications
- Each app can be a host (consumer) or remote (provider)
- Bidirectional relationships possible

**Key Terms:**
- **Host/Container**: Application that loads remote modules
- **Remote**: Application that exposes modules
- **Shared**: Dependencies shared between host and remotes
- **Exposed**: Modules made available to other apps
- **Remote Entry**: Entry point file for remote modules

---

#### Basic Configuration

**Remote Application (exposes modules):**
```javascript
// products-app/webpack.config.js
const ModuleFederationPlugin = require('webpack/lib/container/ModuleFederationPlugin');
const deps = require('./package.json').dependencies;

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      // Unique name for this remote
      name: 'products',
      
      // Output filename for remote entry
      filename: 'remoteEntry.js',
      
      // Modules to expose
      exposes: {
        './ProductList': './src/components/ProductList',
        './ProductDetail': './src/components/ProductDetail',
        './App': './src/App',
      },
      
      // Shared dependencies
      shared: {
        react: {
          singleton: true,
          requiredVersion: deps.react,
          eager: false,
        },
        'react-dom': {
          singleton: true,
          requiredVersion: deps['react-dom'],
        },
      },
    }),
  ],
};
```

**Host Application (consumes remotes):**
```javascript
// host-app/webpack.config.js
const ModuleFederationPlugin = require('webpack/lib/container/ModuleFederationPlugin');
const deps = require('./package.json').dependencies;

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'host',
      
      // Remote applications to consume
      remotes: {
        products: 'products@http://localhost:3001/remoteEntry.js',
        checkout: 'checkout@http://localhost:3002/remoteEntry.js',
        header: 'header@http://localhost:3003/remoteEntry.js',
      },
      
      // Shared dependencies
      shared: {
        react: {
          singleton: true,
          requiredVersion: deps.react,
        },
        'react-dom': {
          singleton: true,
          requiredVersion: deps['react-dom'],
        },
        'react-router-dom': {
          singleton: true,
        },
      },
    }),
  ],
};
```

**Using Remote Modules:**
```tsx
// host-app/src/App.tsx
import React, { lazy, Suspense } from 'react';

// Dynamic imports from remotes
const ProductList = lazy(() => import('products/ProductList'));
const ProductDetail = lazy(() => import('products/ProductDetail'));
const Header = lazy(() => import('header/App'));
const Checkout = lazy(() => import('checkout/App'));

function App() {
  return (
    <div>
      <Suspense fallback={<div>Loading Header...</div>}>
        <Header />
      </Suspense>

      <Suspense fallback={<div>Loading Products...</div>}>
        <ProductList />
      </Suspense>
    </div>
  );
}

export default App;
```

---

#### Shared Dependencies

**Singleton Sharing:**
```javascript
shared: {
  react: {
    // Only one version of React loads (prevents duplicate React instances)
    singleton: true,
    
    // Required version range
    requiredVersion: '^18.0.0',
    
    // Load immediately vs on-demand
    eager: false,
    
    // Show warning if version mismatch
    strictVersion: false,
  },
}
```

**Sharing Strategies:**

1. **Singleton (Most Common):**
```javascript
shared: {
  react: { singleton: true },
  'react-dom': { singleton: true },
}
// Only one instance across all apps
```

2. **Version-based:**
```javascript
shared: {
  lodash: {
    singleton: false,  // Allow multiple versions
    requiredVersion: '^4.17.0',
  },
}
// Each app can have its own version
```

3. **Eager Loading:**
```javascript
shared: {
  react: {
    singleton: true,
    eager: true,  // Load immediately, not lazy
  },
}
```

---

#### Complete Real-World Example

**Project Structure:**
```
microfrontend-app/
├── host/                    # Container app
│   ├── src/
│   │   ├── App.tsx
│   │   └── bootstrap.tsx
│   ├── webpack.config.js
│   └── package.json
├── products/                # Products MFE
│   ├── src/
│   │   ├── App.tsx
│   │   ├── ProductList.tsx
│   │   └── bootstrap.tsx
│   ├── webpack.config.js
│   └── package.json
├── cart/                    # Cart MFE
│   ├── src/
│   │   ├── App.tsx
│   │   ├── Cart.tsx
│   │   └── bootstrap.tsx
│   ├── webpack.config.js
│   └── package.json
└── shared/                  # Shared types/utils
    └── types.ts
```

**Products MFE:**
```javascript
// products/webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin');
const ModuleFederationPlugin = require('webpack/lib/container/ModuleFederationPlugin');
const path = require('path');
const deps = require('./package.json').dependencies;

module.exports = {
  entry: './src/index.ts',
  mode: 'development',
  devServer: {
    port: 3001,
    historyApiFallback: true,
  },
  output: {
    publicPath: 'http://localhost:3001/',
  },
  resolve: {
    extensions: ['.tsx', '.ts', '.js'],
  },
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/,
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
    ],
  },
  plugins: [
    new ModuleFederationPlugin({
      name: 'products',
      filename: 'remoteEntry.js',
      exposes: {
        './ProductList': './src/components/ProductList',
        './ProductDetail': './src/components/ProductDetail',
        './App': './src/App',
      },
      shared: {
        react: { singleton: true, requiredVersion: deps.react },
        'react-dom': { singleton: true, requiredVersion: deps['react-dom'] },
        'react-router-dom': { singleton: true },
      },
    }),
    new HtmlWebpackPlugin({
      template: './public/index.html',
    }),
  ],
};
```

**Products App Component:**
```tsx
// products/src/App.tsx
import React from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import ProductList from './components/ProductList';
import ProductDetail from './components/ProductDetail';

function App() {
  return (
    <div className="products-app">
      <h2>Products MFE</h2>
      <Routes>
        <Route path="/" element={<ProductList />} />
        <Route path="/:id" element={<ProductDetail />} />
      </Routes>
    </div>
  );
}

export default App;
```

**Bootstrap Pattern (Important!):**
```tsx
// products/src/index.ts
import('./bootstrap');

// products/src/bootstrap.tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import { BrowserRouter } from 'react-router-dom';
import App from './App';

const root = ReactDOM.createRoot(
  document.getElementById('root') as HTMLElement
);

root.render(
  <BrowserRouter>
    <App />
  </BrowserRouter>
);
```

**Host Application:**
```javascript
// host/webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin');
const ModuleFederationPlugin = require('webpack/lib/container/ModuleFederationPlugin');
const deps = require('./package.json').dependencies;

module.exports = {
  entry: './src/index.ts',
  mode: 'development',
  devServer: {
    port: 3000,
    historyApiFallback: true,
  },
  output: {
    publicPath: 'http://localhost:3000/',
  },
  resolve: {
    extensions: ['.tsx', '.ts', '.js'],
  },
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/,
      },
    ],
  },
  plugins: [
    new ModuleFederationPlugin({
      name: 'host',
      remotes: {
        products: 'products@http://localhost:3001/remoteEntry.js',
        cart: 'cart@http://localhost:3002/remoteEntry.js',
      },
      shared: {
        react: { singleton: true, requiredVersion: deps.react },
        'react-dom': { singleton: true, requiredVersion: deps['react-dom'] },
        'react-router-dom': { singleton: true },
      },
    }),
    new HtmlWebpackPlugin({
      template: './public/index.html',
    }),
  ],
};
```

**Host App:**
```tsx
// host/src/App.tsx
import React, { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

const ProductsApp = lazy(() => import('products/App'));
const CartApp = lazy(() => import('cart/App'));

function App() {
  return (
    <BrowserRouter>
      <div className="host-app">
        <nav>
          <Link to="/">Home</Link>
          <Link to="/products">Products</Link>
          <Link to="/cart">Cart</Link>
        </nav>

        <main>
          <Suspense fallback={<div>Loading...</div>}>
            <Routes>
              <Route path="/" element={<div>Home Page</div>} />
              <Route path="/products/*" element={<ProductsApp />} />
              <Route path="/cart/*" element={<CartApp />} />
            </Routes>
          </Suspense>
        </main>
      </div>
    </BrowserRouter>
  );
}

export default App;
```

---

#### TypeScript Support

**Type Declarations for Remotes:**
```typescript
// host/src/types/remotes.d.ts
declare module 'products/App' {
  const App: React.ComponentType;
  export default App;
}

declare module 'products/ProductList' {
  interface ProductListProps {
    onProductClick?: (id: string) => void;
  }
  const ProductList: React.ComponentType<ProductListProps>;
  export default ProductList;
}

declare module 'cart/App' {
  const App: React.ComponentType;
  export default App;
}
```

**Shared Types Package:**
```typescript
// shared/types.ts
export interface Product {
  id: string;
  name: string;
  price: number;
  image: string;
}

export interface CartItem extends Product {
  quantity: number;
}

export interface MicroFrontendProps {
  basename?: string;
  onNavigate?: (path: string) => void;
}
```

---

#### Dynamic Remote URLs

**Load Remotes from Config:**
```javascript
// webpack.config.js
const ModuleFederationPlugin = require('webpack/lib/container/ModuleFederationPlugin');

// Read from environment or config
const REMOTES = {
  products: process.env.PRODUCTS_URL || 'http://localhost:3001',
  cart: process.env.CART_URL || 'http://localhost:3002',
};

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'host',
      remotes: {
        products: `products@${REMOTES.products}/remoteEntry.js`,
        cart: `cart@${REMOTES.cart}/remoteEntry.js`,
      },
      shared: { /* ... */ },
    }),
  ],
};
```

**Runtime Remote Loading:**
```typescript
// Dynamically load remote at runtime
function loadRemote(url: string, scope: string, module: string) {
  return async () => {
    await __webpack_init_sharing__('default');
    const container = window[scope];
    await container.init(__webpack_share_scopes__.default);
    const factory = await container.get(module);
    return factory();
  };
}

// Usage
const ProductList = React.lazy(
  loadRemote(
    'http://localhost:3001/remoteEntry.js',
    'products',
    './ProductList'
  )
);
```

---

#### Error Handling

**Component Error Boundary:**
```tsx
import React, { Component, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
  error?: Error;
}

class RemoteErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false };

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('Remote module error:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        this.props.fallback || (
          <div>
            <h3>Failed to load remote module</h3>
            <p>{this.state.error?.message}</p>
            <button onClick={() => window.location.reload()}>
              Reload Page
            </button>
          </div>
        )
      );
    }

    return this.props.children;
  }
}

// Usage
function App() {
  return (
    <RemoteErrorBoundary fallback={<div>Products unavailable</div>}>
      <Suspense fallback={<div>Loading...</div>}>
        <ProductsApp />
      </Suspense>
    </RemoteErrorBoundary>
  );
}
```

---

#### Version Management

**Strict Version Checking:**
```javascript
shared: {
  react: {
    singleton: true,
    requiredVersion: deps.react,
    strictVersion: true,  // Throw error on mismatch
  },
}
```

**Fallback for Version Conflicts:**
```javascript
shared: {
  lodash: {
    singleton: false,  // Allow multiple versions
    requiredVersion: false,  // Any version works
  },
}
```

---

#### Performance Optimization

**Preload Remotes:**
```tsx
// Preload remote modules
function App() {
  useEffect(() => {
    // Preload products module
    import('products/App');
  }, []);

  return <div>...</div>;
}
```

**Chunking Strategy:**
```javascript
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'async',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          priority: 10,
        },
      },
    },
  },
};
```

---

#### Bidirectional Hosting

**Both Host and Remote:**
```javascript
// App can be both consumer and provider
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'app1',
      filename: 'remoteEntry.js',
      
      // This app exposes modules
      exposes: {
        './Button': './src/components/Button',
      },
      
      // This app consumes remote modules
      remotes: {
        app2: 'app2@http://localhost:3002/remoteEntry.js',
      },
      
      shared: ['react', 'react-dom'],
    }),
  ],
};
```

---

#### Best Practices

**1. Bootstrap Pattern:**
```typescript
// Always use bootstrap pattern for async loading
// index.ts
import('./bootstrap');

// bootstrap.tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
// ... render
```

**2. Singleton Dependencies:**
```javascript
// Always singleton for React, React-DOM, Router
shared: {
  react: { singleton: true },
  'react-dom': { singleton: true },
  'react-router-dom': { singleton: true },
}
```

**3. Versioning:**
```javascript
// Use semantic versioning
requiredVersion: deps.react,  // From package.json
strictVersion: false,         // Allow minor version differences
```

**4. Error Boundaries:**
```tsx
// Wrap each remote in error boundary
<ErrorBoundary>
  <Suspense fallback={<Loading />}>
    <RemoteModule />
  </Suspense>
</ErrorBoundary>
```

**5. Type Safety:**
```typescript
// Declare module types
declare module 'products/App' {
  const App: React.ComponentType;
  export default App;
}
```

---

#### Common Issues

**Issue 1: Shared Dependencies Not Working**
```javascript
// ❌ Wrong
shared: ['react', 'react-dom']

// ✅ Correct
shared: {
  react: { singleton: true },
  'react-dom': { singleton: true },
}
```

**Issue 2: Multiple React Instances**
```javascript
// Ensure singleton in all configs
shared: {
  react: { 
    singleton: true,
    eager: true,  // Load immediately
  },
}
```

**Issue 3: CORS Errors**
```javascript
// Dev server config
devServer: {
  headers: {
    'Access-Control-Allow-Origin': '*',
  },
}
```

---

#### Interview Tips

✅ **Key Points:**
- Webpack 5 plugin for runtime code sharing
- Enables microfrontend architecture
- Dynamic remote module loading
- Singleton dependency sharing
- Requires bootstrap pattern for async initialization

✅ **When to Mention:**
- Microfrontend implementations
- Code sharing between apps
- Independent deployments
- Team autonomy and scaling
- Modern build tool features

✅ **Common Follow-ups:**
- "How does shared dependency resolution work?"
- "What's the bootstrap pattern and why is it needed?"
- "How to handle version conflicts?"
- "What about TypeScript support?"
- "Performance implications?"

✅ **Perfect Answer Structure:**
1. Define: Webpack 5 runtime code sharing
2. Key config: exposes, remotes, shared
3. Bootstrap pattern: Async initialization required
4. Singleton sharing: Prevent duplicate dependencies
5. Use case: Microfrontends, independent deployments
6. Benefits: Team autonomy, independent releases

</details>

---

### 163. How do you design a scalable React architecture?

<details>
<summary>View Answer</summary>

**Scalable React architecture** is about structuring your application to handle growth in features, team size, and complexity while maintaining code quality, performance, and developer productivity. It involves proper folder structure, separation of concerns, and architectural patterns.

#### Core Principles

**1. Separation of Concerns**
- UI components separate from business logic
- Data fetching separate from presentation
- State management isolated from views
- Routing separate from components

**2. Single Responsibility**
- Each component/module does one thing
- Easy to understand and test
- Reusable across application

**3. Loose Coupling**
- Components don't depend on implementation details
- Use interfaces/contracts
- Dependency injection

**4. High Cohesion**
- Related code stays together
- Feature-based organization
- Co-location of related files

---

#### Folder Structure Strategies

**1. Feature-Based Structure (Recommended for Scale)**

```
src/
├── features/
│   ├── auth/
│   │   ├── components/
│   │   │   ├── LoginForm.tsx
│   │   │   ├── RegisterForm.tsx
│   │   │   └── AuthGuard.tsx
│   │   ├── hooks/
│   │   │   ├── useAuth.ts
│   │   │   └── useLogin.ts
│   │   ├── api/
│   │   │   └── authApi.ts
│   │   ├── store/
│   │   │   ├── authSlice.ts
│   │   │   └── authSelectors.ts
│   │   ├── types/
│   │   │   └── auth.types.ts
│   │   ├── utils/
│   │   │   └── tokenUtils.ts
│   │   └── index.ts
│   │
│   ├── products/
│   │   ├── components/
│   │   │   ├── ProductList.tsx
│   │   │   ├── ProductCard.tsx
│   │   │   └── ProductDetail.tsx
│   │   ├── hooks/
│   │   │   ├── useProducts.ts
│   │   │   └── useProductFilters.ts
│   │   ├── api/
│   │   │   └── productsApi.ts
│   │   ├── store/
│   │   │   └── productsSlice.ts
│   │   ├── types/
│   │   │   └── product.types.ts
│   │   └── index.ts
│   │
│   └── cart/
│       ├── components/
│       ├── hooks/
│       ├── api/
│       ├── store/
│       └── index.ts
│
├── shared/
│   ├── components/
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   ├── Button.test.tsx
│   │   │   ├── Button.stories.tsx
│   │   │   └── Button.module.css
│   │   ├── Input/
│   │   ├── Modal/
│   │   └── index.ts
│   │
│   ├── hooks/
│   │   ├── useDebounce.ts
│   │   ├── useLocalStorage.ts
│   │   └── useIntersectionObserver.ts
│   │
│   ├── utils/
│   │   ├── formatting.ts
│   │   ├── validation.ts
│   │   └── api.ts
│   │
│   ├── constants/
│   │   ├── routes.ts
│   │   └── config.ts
│   │
│   └── types/
│       └── common.types.ts
│
├── layouts/
│   ├── MainLayout.tsx
│   ├── AuthLayout.tsx
│   └── DashboardLayout.tsx
│
├── pages/
│   ├── HomePage.tsx
│   ├── ProductsPage.tsx
│   ├── CartPage.tsx
│   └── NotFoundPage.tsx
│
├── services/
│   ├── api/
│   │   ├── client.ts
│   │   ├── interceptors.ts
│   │   └── endpoints.ts
│   ├── storage/
│   │   └── localStorage.ts
│   └── analytics/
│       └── analytics.ts
│
├── store/
│   ├── index.ts
│   ├── rootReducer.ts
│   └── middleware.ts
│
├── routes/
│   ├── index.tsx
│   ├── PrivateRoute.tsx
│   └── routes.config.ts
│
├── App.tsx
├── index.tsx
└── setupTests.ts
```

**2. Layer-Based Structure (Traditional)**

```
src/
├── components/
│   ├── common/
│   ├── features/
│   └── layouts/
├── hooks/
├── services/
├── store/
│   ├── slices/
│   └── selectors/
├── utils/
├── types/
└── pages/
```

**3. Atomic Design Structure**

```
src/
├── components/
│   ├── atoms/       # Basic building blocks (Button, Input)
│   ├── molecules/   # Simple combinations (FormField, Card)
│   ├── organisms/   # Complex components (Header, ProductCard)
│   ├── templates/   # Page layouts
│   └── pages/       # Actual pages
├── hooks/
├── services/
└── store/
```

---

#### Layered Architecture

**Architecture Layers:**

```
┌─────────────────────────────────┐
│     Presentation Layer          │  Components, Pages
├─────────────────────────────────┤
│     Application Layer           │  Hooks, State Management
├─────────────────────────────────┤
│     Domain Layer                │  Business Logic, Models
├─────────────────────────────────┤
│     Infrastructure Layer        │  API, Storage, External Services
└─────────────────────────────────┘
```

**Implementation:**

**1. Presentation Layer (View):**
```tsx
// features/products/components/ProductList.tsx
import { useProducts } from '../hooks/useProducts';
import { ProductCard } from './ProductCard';

export function ProductList() {
  const { products, loading, error } = useProducts();

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div className="product-list">
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}
```

**2. Application Layer (Hooks/State):**
```typescript
// features/products/hooks/useProducts.ts
import { useQuery } from '@tanstack/react-query';
import { productService } from '../services/productService';
import type { Product } from '../types';

export function useProducts() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['products'],
    queryFn: () => productService.getAll(),
  });

  return {
    products: data || [],
    loading: isLoading,
    error,
  };
}
```

**3. Domain Layer (Business Logic):**
```typescript
// features/products/services/productService.ts
import { apiClient } from '@/services/api';
import type { Product } from '../types';

class ProductService {
  async getAll(): Promise<Product[]> {
    const response = await apiClient.get<Product[]>('/products');
    return response.data;
  }

  async getById(id: string): Promise<Product> {
    const response = await apiClient.get<Product>(`/products/${id}`);
    return response.data;
  }

  async create(product: Omit<Product, 'id'>): Promise<Product> {
    const response = await apiClient.post<Product>('/products', product);
    return response.data;
  }

  calculateDiscount(product: Product, discountPercent: number): number {
    return product.price * (1 - discountPercent / 100);
  }

  isAvailable(product: Product): boolean {
    return product.stock > 0 && product.status === 'active';
  }
}

export const productService = new ProductService();
```

**4. Infrastructure Layer (API/Storage):**
```typescript
// services/api/client.ts
import axios, { AxiosInstance } from 'axios';

class ApiClient {
  private client: AxiosInstance;

  constructor() {
    this.client = axios.create({
      baseURL: process.env.REACT_APP_API_URL,
      timeout: 10000,
    });

    this.setupInterceptors();
  }

  private setupInterceptors() {
    // Request interceptor
    this.client.interceptors.request.use(
      (config) => {
        const token = localStorage.getItem('token');
        if (token) {
          config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
      },
      (error) => Promise.reject(error)
    );

    // Response interceptor
    this.client.interceptors.response.use(
      (response) => response,
      (error) => {
        if (error.response?.status === 401) {
          // Handle unauthorized
          window.location.href = '/login';
        }
        return Promise.reject(error);
      }
    );
  }

  get<T>(url: string) {
    return this.client.get<T>(url);
  }

  post<T>(url: string, data: any) {
    return this.client.post<T>(url, data);
  }

  put<T>(url: string, data: any) {
    return this.client.put<T>(url, data);
  }

  delete<T>(url: string) {
    return this.client.delete<T>(url);
  }
}

export const apiClient = new ApiClient();
```

---

#### State Management Architecture

**Redux Toolkit Structure:**

```typescript
// store/index.ts
import { configureStore } from '@reduxjs/toolkit';
import { setupListeners } from '@reduxjs/toolkit/query';
import authReducer from '@/features/auth/store/authSlice';
import productsReducer from '@/features/products/store/productsSlice';
import cartReducer from '@/features/cart/store/cartSlice';
import { apiSlice } from './api/apiSlice';

export const store = configureStore({
  reducer: {
    auth: authReducer,
    products: productsReducer,
    cart: cartReducer,
    [apiSlice.reducerPath]: apiSlice.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(apiSlice.middleware),
});

setupListeners(store.dispatch);

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

**Feature Slice:**
```typescript
// features/products/store/productsSlice.ts
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';
import { productService } from '../services/productService';
import type { Product } from '../types';

interface ProductsState {
  items: Product[];
  selectedProduct: Product | null;
  loading: boolean;
  error: string | null;
  filters: {
    category?: string;
    priceRange?: [number, number];
    search?: string;
  };
}

const initialState: ProductsState = {
  items: [],
  selectedProduct: null,
  loading: false,
  error: null,
  filters: {},
};

export const fetchProducts = createAsyncThunk(
  'products/fetchAll',
  async () => {
    return await productService.getAll();
  }
);

const productsSlice = createSlice({
  name: 'products',
  initialState,
  reducers: {
    setFilters: (state, action: PayloadAction<ProductsState['filters']>) => {
      state.filters = action.payload;
    },
    clearFilters: (state) => {
      state.filters = {};
    },
    selectProduct: (state, action: PayloadAction<Product>) => {
      state.selectedProduct = action.payload;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchProducts.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchProducts.fulfilled, (state, action) => {
        state.loading = false;
        state.items = action.payload;
      })
      .addCase(fetchProducts.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message || 'Failed to fetch products';
      });
  },
});

export const { setFilters, clearFilters, selectProduct } = productsSlice.actions;
export default productsSlice.reducer;
```

**Selectors:**
```typescript
// features/products/store/selectors.ts
import { createSelector } from '@reduxjs/toolkit';
import type { RootState } from '@/store';

const selectProductsState = (state: RootState) => state.products;

export const selectAllProducts = createSelector(
  [selectProductsState],
  (state) => state.items
);

export const selectFilteredProducts = createSelector(
  [selectAllProducts, (state: RootState) => state.products.filters],
  (products, filters) => {
    let filtered = products;

    if (filters.category) {
      filtered = filtered.filter(p => p.category === filters.category);
    }

    if (filters.search) {
      const search = filters.search.toLowerCase();
      filtered = filtered.filter(p =>
        p.name.toLowerCase().includes(search)
      );
    }

    if (filters.priceRange) {
      const [min, max] = filters.priceRange;
      filtered = filtered.filter(p => p.price >= min && p.price <= max);
    }

    return filtered;
  }
);

export const selectProductById = createSelector(
  [selectAllProducts, (_: RootState, productId: string) => productId],
  (products, productId) => products.find(p => p.id === productId)
);
```

---

#### Dependency Injection Pattern

**Service Interface:**
```typescript
// services/storage/IStorageService.ts
export interface IStorageService {
  get<T>(key: string): T | null;
  set<T>(key: string, value: T): void;
  remove(key: string): void;
  clear(): void;
}
```

**Implementation:**
```typescript
// services/storage/LocalStorageService.ts
import { IStorageService } from './IStorageService';

export class LocalStorageService implements IStorageService {
  get<T>(key: string): T | null {
    const item = localStorage.getItem(key);
    return item ? JSON.parse(item) : null;
  }

  set<T>(key: string, value: T): void {
    localStorage.setItem(key, JSON.stringify(value));
  }

  remove(key: string): void {
    localStorage.removeItem(key);
  }

  clear(): void {
    localStorage.clear();
  }
}

// services/storage/SessionStorageService.ts
export class SessionStorageService implements IStorageService {
  get<T>(key: string): T | null {
    const item = sessionStorage.getItem(key);
    return item ? JSON.parse(item) : null;
  }

  set<T>(key: string, value: T): void {
    sessionStorage.setItem(key, JSON.stringify(value));
  }

  remove(key: string): void {
    sessionStorage.removeItem(key);
  }

  clear(): void {
    sessionStorage.clear();
  }
}
```

**Service Provider:**
```tsx
// contexts/ServiceContext.tsx
import React, { createContext, useContext, ReactNode } from 'react';
import { IStorageService } from '@/services/storage/IStorageService';
import { LocalStorageService } from '@/services/storage/LocalStorageService';

interface Services {
  storage: IStorageService;
}

const ServiceContext = createContext<Services | undefined>(undefined);

export function ServiceProvider({ children }: { children: ReactNode }) {
  const services: Services = {
    storage: new LocalStorageService(),
  };

  return (
    <ServiceContext.Provider value={services}>
      {children}
    </ServiceContext.Provider>
  );
}

export function useServices() {
  const context = useContext(ServiceContext);
  if (!context) throw new Error('useServices must be within ServiceProvider');
  return context;
}

// Usage in component
function MyComponent() {
  const { storage } = useServices();
  
  const saveData = () => {
    storage.set('key', { data: 'value' });
  };

  return <button onClick={saveData}>Save</button>;
}
```

---

#### Code Splitting Strategies

**Route-Based Splitting:**
```tsx
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

const HomePage = lazy(() => import('@/pages/HomePage'));
const ProductsPage = lazy(() => import('@/pages/ProductsPage'));
const CartPage = lazy(() => import('@/pages/CartPage'));
const CheckoutPage = lazy(() => import('@/pages/CheckoutPage'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<LoadingSpinner />}>
        <Routes>
          <Route path="/" element={<HomePage />} />
          <Route path="/products" element={<ProductsPage />} />
          <Route path="/cart" element={<CartPage />} />
          <Route path="/checkout" element={<CheckoutPage />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

**Component-Based Splitting:**
```tsx
import { lazy, Suspense } from 'react';

const HeavyChart = lazy(() => import('./HeavyChart'));
const ComplexModal = lazy(() => import('./ComplexModal'));

function Dashboard() {
  const [showModal, setShowModal] = useState(false);

  return (
    <div>
      <h1>Dashboard</h1>
      
      <Suspense fallback={<div>Loading chart...</div>}>
        <HeavyChart data={data} />
      </Suspense>

      {showModal && (
        <Suspense fallback={<div>Loading...</div>}>
          <ComplexModal onClose={() => setShowModal(false)} />
        </Suspense>
      )}
    </div>
  );
}
```

---

#### Performance Optimization

**Memoization Strategy:**
```tsx
import { memo, useMemo, useCallback } from 'react';

// Memoize expensive components
const ProductCard = memo(function ProductCard({ product }: Props) {
  return <div>{product.name}</div>;
});

function ProductList({ products, onProductClick }: Props) {
  // Memoize expensive calculations
  const sortedProducts = useMemo(() => {
    return [...products].sort((a, b) => a.price - b.price);
  }, [products]);

  // Memoize callbacks
  const handleClick = useCallback((id: string) => {
    onProductClick(id);
  }, [onProductClick]);

  return (
    <div>
      {sortedProducts.map(product => (
        <ProductCard
          key={product.id}
          product={product}
          onClick={handleClick}
        />
      ))}
    </div>
  );
}
```

**Virtual Scrolling:**
```tsx
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualList({ items }: { items: Product[] }) {
  const parentRef = useRef<HTMLDivElement>(null);

  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 100,
  });

  return (
    <div ref={parentRef} style={{ height: '600px', overflow: 'auto' }}>
      <div
        style={{
          height: `${virtualizer.getTotalSize()}px`,
          position: 'relative',
        }}
      >
        {virtualizer.getVirtualItems().map((virtualItem) => (
          <div
            key={virtualItem.key}
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              transform: `translateY(${virtualItem.start}px)`,
            }}
          >
            <ProductCard product={items[virtualItem.index]} />
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

#### Testing Architecture

**Test Organization:**
```
features/products/
├── components/
│   ├── ProductList.tsx
│   └── __tests__/
│       ├── ProductList.test.tsx
│       └── ProductList.integration.test.tsx
├── hooks/
│   ├── useProducts.ts
│   └── __tests__/
│       └── useProducts.test.ts
└── services/
    ├── productService.ts
    └── __tests__/
        └── productService.test.ts
```

**Unit Test Example:**
```typescript
// productService.test.ts
import { productService } from '../productService';
import type { Product } from '../../types';

describe('ProductService', () => {
  describe('calculateDiscount', () => {
    it('calculates discount correctly', () => {
      const product: Product = {
        id: '1',
        name: 'Test',
        price: 100,
        stock: 10,
        status: 'active',
      };

      const discounted = productService.calculateDiscount(product, 20);
      expect(discounted).toBe(80);
    });
  });

  describe('isAvailable', () => {
    it('returns true when in stock and active', () => {
      const product: Product = {
        id: '1',
        name: 'Test',
        price: 100,
        stock: 10,
        status: 'active',
      };

      expect(productService.isAvailable(product)).toBe(true);
    });

    it('returns false when out of stock', () => {
      const product: Product = {
        id: '1',
        name: 'Test',
        price: 100,
        stock: 0,
        status: 'active',
      };

      expect(productService.isAvailable(product)).toBe(false);
    });
  });
});
```

---

#### Best Practices Checklist

**✅ Structure:**
- Feature-based folder organization
- Co-locate related files
- Clear separation of concerns
- Consistent naming conventions

**✅ Components:**
- Small, focused components
- Composition over inheritance
- Props interface for every component
- Proper TypeScript typing

**✅ State Management:**
- Local state when possible
- Lift state only when needed
- Normalized state shape
- Memoized selectors

**✅ Performance:**
- Code splitting by route
- Lazy loading heavy components
- Memoization (memo, useMemo, useCallback)
- Virtual scrolling for large lists

**✅ Code Quality:**
- ESLint + Prettier
- TypeScript strict mode
- Unit tests for business logic
- Integration tests for flows

**✅ Documentation:**
- README for each feature
- JSDoc for complex functions
- Storybook for components
- Architecture decision records (ADRs)

---

#### Interview Tips

✅ **Key Points:**
- Feature-based structure for scalability
- Layered architecture (Presentation, Application, Domain, Infrastructure)
- Separation of concerns
- Dependency injection for testability
- Code splitting and lazy loading
- Proper state management strategy

✅ **When to Mention:**
- Large application design
- Team scaling questions
- Code organization
- Maintainability concerns
- Performance optimization

✅ **Common Follow-ups:**
- "How do you decide what goes in shared vs feature folders?"
- "When to use Context vs Redux?"
- "How to handle code splitting?"
- "What testing strategy for large apps?"
- "How to prevent circular dependencies?"

</details>

---

### 164. What is Clean Architecture in React context?

<details>
<summary>View Answer</summary>

**Clean Architecture** in React is an adaptation of Robert C. Martin's (Uncle Bob) Clean Architecture principles to frontend development. It emphasizes separation of concerns, dependency inversion, and independence from frameworks, UI, and external agencies.

#### Core Principles

**1. Dependency Rule**
- Dependencies point inward (toward business logic)
- Inner layers don't know about outer layers
- Business logic independent of UI/framework

**2. Independence**
- Independent of frameworks (React is just a detail)
- Independent of UI (swap React for Vue/Angular)
- Independent of database/API
- Testable without external dependencies

**3. Layers (Concentric Circles)**
```
┌────────────────────────────────────────┐
│  Frameworks & Drivers (React, API)     │  ← Outer
├────────────────────────────────────────┤
│  Interface Adapters (Presenters)       │
├────────────────────────────────────────┤
│  Use Cases (Application Business Rules)│
├────────────────────────────────────────┤
│  Entities (Enterprise Business Rules)  │  ← Inner
└────────────────────────────────────────┘
```

---

#### Layer Breakdown

**1. Entities (Domain Layer) - Core Business Logic**

```typescript
// domain/entities/Product.ts
export class Product {
  constructor(
    public readonly id: string,
    public readonly name: string,
    public readonly price: number,
    public readonly description: string,
    public readonly stock: number
  ) {}

  isAvailable(): boolean {
    return this.stock > 0;
  }

  calculateDiscountedPrice(discountPercent: number): number {
    return this.price * (1 - discountPercent / 100);
  }

  canPurchase(quantity: number): boolean {
    return this.stock >= quantity && quantity > 0;
  }
}

export class Order {
  constructor(
    public readonly id: string,
    public readonly userId: string,
    public readonly items: OrderItem[],
    public readonly status: OrderStatus,
    public readonly createdAt: Date
  ) {}

  calculateTotal(): number {
    return this.items.reduce((sum, item) => sum + item.subtotal(), 0);
  }

  canBeCancelled(): boolean {
    return this.status === 'pending' || this.status === 'processing';
  }
}

export class OrderItem {
  constructor(
    public readonly product: Product,
    public readonly quantity: number
  ) {}

  subtotal(): number {
    return this.product.price * this.quantity;
  }
}

type OrderStatus = 'pending' | 'processing' | 'shipped' | 'delivered' | 'cancelled';
```

**2. Use Cases (Application Layer) - Application-Specific Business Rules**

```typescript
// application/useCases/CreateOrderUseCase.ts
import { Order, Product, OrderItem } from '@/domain/entities';
import { IOrderRepository } from '@/domain/repositories/IOrderRepository';
import { IProductRepository } from '@/domain/repositories/IProductRepository';

export interface CreateOrderInput {
  userId: string;
  items: Array<{ productId: string; quantity: number }>;
}

export interface CreateOrderOutput {
  orderId: string;
  total: number;
  success: boolean;
  error?: string;
}

export class CreateOrderUseCase {
  constructor(
    private orderRepository: IOrderRepository,
    private productRepository: IProductRepository
  ) {}

  async execute(input: CreateOrderInput): Promise<CreateOrderOutput> {
    try {
      // Fetch products
      const products = await Promise.all(
        input.items.map(item => this.productRepository.findById(item.productId))
      );

      // Validate stock availability
      for (let i = 0; i < products.length; i++) {
        const product = products[i];
        const requestedQty = input.items[i].quantity;

        if (!product.canPurchase(requestedQty)) {
          return {
            orderId: '',
            total: 0,
            success: false,
            error: `Product ${product.name} is not available in requested quantity`,
          };
        }
      }

      // Create order items
      const orderItems = products.map(
        (product, i) => new OrderItem(product, input.items[i].quantity)
      );

      // Create order
      const order = new Order(
        this.generateId(),
        input.userId,
        orderItems,
        'pending',
        new Date()
      );

      // Save order
      await this.orderRepository.save(order);

      // Update product stock
      for (let i = 0; i < products.length; i++) {
        await this.productRepository.updateStock(
          products[i].id,
          products[i].stock - input.items[i].quantity
        );
      }

      return {
        orderId: order.id,
        total: order.calculateTotal(),
        success: true,
      };
    } catch (error) {
      return {
        orderId: '',
        total: 0,
        success: false,
        error: error instanceof Error ? error.message : 'Unknown error',
      };
    }
  }

  private generateId(): string {
    return `order-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }
}
```

```typescript
// application/useCases/GetProductsUseCase.ts
import { Product } from '@/domain/entities/Product';
import { IProductRepository } from '@/domain/repositories/IProductRepository';

export interface GetProductsFilters {
  category?: string;
  minPrice?: number;
  maxPrice?: number;
  inStock?: boolean;
}

export class GetProductsUseCase {
  constructor(private productRepository: IProductRepository) {}

  async execute(filters?: GetProductsFilters): Promise<Product[]> {
    let products = await this.productRepository.findAll();

    if (!filters) return products;

    // Apply filters
    if (filters.category) {
      products = products.filter(p => p.category === filters.category);
    }

    if (filters.minPrice !== undefined) {
      products = products.filter(p => p.price >= filters.minPrice!);
    }

    if (filters.maxPrice !== undefined) {
      products = products.filter(p => p.price <= filters.maxPrice!);
    }

    if (filters.inStock) {
      products = products.filter(p => p.isAvailable());
    }

    return products;
  }
}
```

**3. Interface Adapters (Repositories & Presenters)**

```typescript
// domain/repositories/IProductRepository.ts (Port)
export interface IProductRepository {
  findAll(): Promise<Product[]>;
  findById(id: string): Promise<Product>;
  save(product: Product): Promise<void>;
  updateStock(id: string, newStock: number): Promise<void>;
  delete(id: string): Promise<void>;
}

// domain/repositories/IOrderRepository.ts (Port)
export interface IOrderRepository {
  findAll(): Promise<Order[]>;
  findById(id: string): Promise<Order>;
  findByUserId(userId: string): Promise<Order[]>;
  save(order: Order): Promise<void>;
  updateStatus(id: string, status: OrderStatus): Promise<void>;
}
```

```typescript
// infrastructure/repositories/ApiProductRepository.ts (Adapter)
import { IProductRepository } from '@/domain/repositories/IProductRepository';
import { Product } from '@/domain/entities/Product';
import { apiClient } from '../api/apiClient';

interface ProductDTO {
  id: string;
  name: string;
  price: number;
  description: string;
  stock: number;
  category: string;
}

export class ApiProductRepository implements IProductRepository {
  async findAll(): Promise<Product[]> {
    const response = await apiClient.get<ProductDTO[]>('/products');
    return response.data.map(this.toDomain);
  }

  async findById(id: string): Promise<Product> {
    const response = await apiClient.get<ProductDTO>(`/products/${id}`);
    return this.toDomain(response.data);
  }

  async save(product: Product): Promise<void> {
    const dto = this.toDTO(product);
    await apiClient.post('/products', dto);
  }

  async updateStock(id: string, newStock: number): Promise<void> {
    await apiClient.patch(`/products/${id}/stock`, { stock: newStock });
  }

  async delete(id: string): Promise<void> {
    await apiClient.delete(`/products/${id}`);
  }

  private toDomain(dto: ProductDTO): Product {
    return new Product(
      dto.id,
      dto.name,
      dto.price,
      dto.description,
      dto.stock
    );
  }

  private toDTO(product: Product): ProductDTO {
    return {
      id: product.id,
      name: product.name,
      price: product.price,
      description: product.description,
      stock: product.stock,
      category: '', // Would be on Product entity
    };
  }
}
```

**4. Frameworks & Drivers (React Components)**

```tsx
// presentation/components/ProductList.tsx
import React, { useEffect, useState } from 'react';
import { Product } from '@/domain/entities/Product';
import { GetProductsUseCase } from '@/application/useCases/GetProductsUseCase';
import { useInjection } from '@/presentation/hooks/useInjection';

export function ProductList() {
  const [products, setProducts] = useState<Product[]>([]);
  const [loading, setLoading] = useState(true);
  const getProductsUseCase = useInjection<GetProductsUseCase>('GetProductsUseCase');

  useEffect(() => {
    loadProducts();
  }, []);

  const loadProducts = async () => {
    setLoading(true);
    try {
      const result = await getProductsUseCase.execute({ inStock: true });
      setProducts(result);
    } catch (error) {
      console.error('Failed to load products:', error);
    } finally {
      setLoading(false);
    }
  };

  if (loading) return <div>Loading...</div>;

  return (
    <div className="product-list">
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}
```

```tsx
// presentation/components/CreateOrder.tsx
import React, { useState } from 'react';
import { CreateOrderUseCase } from '@/application/useCases/CreateOrderUseCase';
import { useInjection } from '@/presentation/hooks/useInjection';

interface Props {
  userId: string;
  cartItems: Array<{ productId: string; quantity: number }>;
}

export function CreateOrder({ userId, cartItems }: Props) {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const createOrderUseCase = useInjection<CreateOrderUseCase>('CreateOrderUseCase');

  const handleCreateOrder = async () => {
    setLoading(true);
    setError(null);

    try {
      const result = await createOrderUseCase.execute({
        userId,
        items: cartItems,
      });

      if (result.success) {
        alert(`Order created! Total: $${result.total}`);
        // Navigate to order confirmation
      } else {
        setError(result.error || 'Failed to create order');
      }
    } catch (err) {
      setError('An unexpected error occurred');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div>
      {error && <div className="error">{error}</div>}
      <button onClick={handleCreateOrder} disabled={loading}>
        {loading ? 'Creating Order...' : 'Place Order'}
      </button>
    </div>
  );
}
```

---

#### Dependency Injection Container

```typescript
// infrastructure/di/container.ts
import { ApiProductRepository } from '../repositories/ApiProductRepository';
import { ApiOrderRepository } from '../repositories/ApiOrderRepository';
import { GetProductsUseCase } from '@/application/useCases/GetProductsUseCase';
import { CreateOrderUseCase } from '@/application/useCases/CreateOrderUseCase';
import { IProductRepository } from '@/domain/repositories/IProductRepository';
import { IOrderRepository } from '@/domain/repositories/IOrderRepository';

type Constructor<T = any> = new (...args: any[]) => T;

class Container {
  private services = new Map<string, any>();
  private factories = new Map<string, () => any>();

  register<T>(name: string, factory: () => T): void {
    this.factories.set(name, factory);
  }

  resolve<T>(name: string): T {
    // Check if already instantiated
    if (this.services.has(name)) {
      return this.services.get(name);
    }

    // Get factory and create instance
    const factory = this.factories.get(name);
    if (!factory) {
      throw new Error(`Service ${name} not registered`);
    }

    const instance = factory();
    this.services.set(name, instance);
    return instance;
  }
}

export const container = new Container();

// Register repositories
container.register<IProductRepository>('IProductRepository', () => {
  return new ApiProductRepository();
});

container.register<IOrderRepository>('IOrderRepository', () => {
  return new ApiOrderRepository();
});

// Register use cases
container.register('GetProductsUseCase', () => {
  const productRepo = container.resolve<IProductRepository>('IProductRepository');
  return new GetProductsUseCase(productRepo);
});

container.register('CreateOrderUseCase', () => {
  const orderRepo = container.resolve<IOrderRepository>('IOrderRepository');
  const productRepo = container.resolve<IProductRepository>('IProductRepository');
  return new CreateOrderUseCase(orderRepo, productRepo);
});
```

```tsx
// presentation/hooks/useInjection.ts
import { container } from '@/infrastructure/di/container';

export function useInjection<T>(name: string): T {
  return container.resolve<T>(name);
}
```

```tsx
// Alternative: React Context for DI
import React, { createContext, useContext, ReactNode } from 'react';
import { container } from '@/infrastructure/di/container';

const DIContext = createContext(container);

export function DIProvider({ children }: { children: ReactNode }) {
  return <DIContext.Provider value={container}>{children}</DIContext.Provider>;
}

export function useInjection<T>(name: string): T {
  const container = useContext(DIContext);
  return container.resolve<T>(name);
}
```

---

#### Folder Structure

```
src/
├── domain/                    # Entities + Repository Interfaces
│   ├── entities/
│   │   ├── Product.ts
│   │   ├── Order.ts
│   │   ├── OrderItem.ts
│   │   └── User.ts
│   ├── repositories/
│   │   ├── IProductRepository.ts
│   │   ├── IOrderRepository.ts
│   │   └── IUserRepository.ts
│   └── valueObjects/
│       ├── Email.ts
│       └── Money.ts
│
├── application/               # Use Cases
│   ├── useCases/
│   │   ├── GetProductsUseCase.ts
│   │   ├── GetProductByIdUseCase.ts
│   │   ├── CreateOrderUseCase.ts
│   │   ├── CancelOrderUseCase.ts
│   │   └── LoginUserUseCase.ts
│   ├── ports/
│   │   └── IEmailService.ts
│   └── dto/
│       ├── CreateOrderDTO.ts
│       └── LoginDTO.ts
│
├── infrastructure/            # External Implementations
│   ├── repositories/
│   │   ├── ApiProductRepository.ts
│   │   ├── ApiOrderRepository.ts
│   │   └── LocalStorageUserRepository.ts
│   ├── api/
│   │   ├── apiClient.ts
│   │   └── endpoints.ts
│   ├── services/
│   │   └── EmailService.ts
│   └── di/
│       └── container.ts
│
├── presentation/              # React Components
│   ├── components/
│   │   ├── products/
│   │   │   ├── ProductList.tsx
│   │   │   ├── ProductCard.tsx
│   │   │   └── ProductDetail.tsx
│   │   ├── orders/
│   │   │   ├── OrderList.tsx
│   │   │   └── CreateOrder.tsx
│   │   └── common/
│   │       ├── Button.tsx
│   │       └── Input.tsx
│   ├── hooks/
│   │   ├── useInjection.ts
│   │   └── useProducts.ts
│   ├── pages/
│   │   ├── HomePage.tsx
│   │   ├── ProductsPage.tsx
│   │   └── OrdersPage.tsx
│   └── providers/
│       └── DIProvider.tsx
│
├── App.tsx
└── index.tsx
```

---

#### Testing Benefits

**Unit Test (Use Case):**
```typescript
// application/useCases/__tests__/CreateOrderUseCase.test.ts
import { CreateOrderUseCase } from '../CreateOrderUseCase';
import { Product } from '@/domain/entities/Product';
import { IProductRepository } from '@/domain/repositories/IProductRepository';
import { IOrderRepository } from '@/domain/repositories/IOrderRepository';

// Mock repositories
class MockProductRepository implements IProductRepository {
  private products: Product[] = [
    new Product('1', 'Product 1', 100, 'Description', 10),
  ];

  async findAll() { return this.products; }
  async findById(id: string) { return this.products[0]; }
  async save(product: Product) {}
  async updateStock(id: string, newStock: number) {}
  async delete(id: string) {}
}

class MockOrderRepository implements IOrderRepository {
  async findAll() { return []; }
  async findById(id: string) { return null as any; }
  async findByUserId(userId: string) { return []; }
  async save(order: Order) {}
  async updateStatus(id: string, status: OrderStatus) {}
}

describe('CreateOrderUseCase', () => {
  it('creates order successfully', async () => {
    const productRepo = new MockProductRepository();
    const orderRepo = new MockOrderRepository();
    const useCase = new CreateOrderUseCase(orderRepo, productRepo);

    const result = await useCase.execute({
      userId: 'user1',
      items: [{ productId: '1', quantity: 2 }],
    });

    expect(result.success).toBe(true);
    expect(result.total).toBe(200);
    expect(result.orderId).toBeTruthy();
  });

  it('fails when product out of stock', async () => {
    const productRepo = new MockProductRepository();
    const orderRepo = new MockOrderRepository();
    const useCase = new CreateOrderUseCase(orderRepo, productRepo);

    const result = await useCase.execute({
      userId: 'user1',
      items: [{ productId: '1', quantity: 100 }], // More than stock
    });

    expect(result.success).toBe(false);
    expect(result.error).toContain('not available');
  });
});
```

**Component Test (with Mocked Use Case):**
```typescript
// presentation/components/__tests__/ProductList.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import { ProductList } from '../ProductList';
import { Product } from '@/domain/entities/Product';
import { GetProductsUseCase } from '@/application/useCases/GetProductsUseCase';

// Mock the use case
jest.mock('@/presentation/hooks/useInjection', () => ({
  useInjection: () => ({
    execute: jest.fn().mockResolvedValue([
      new Product('1', 'Test Product', 100, 'Description', 10),
    ]),
  }),
}));

describe('ProductList', () => {
  it('renders products', async () => {
    render(<ProductList />);

    await waitFor(() => {
      expect(screen.getByText('Test Product')).toBeInTheDocument();
    });
  });
});
```

---

#### Benefits of Clean Architecture

**✅ Testability:**
- Business logic independent of UI
- Mock dependencies easily
- Test without React/API

**✅ Flexibility:**
- Swap React for another framework
- Change API without touching business logic
- Replace state management easily

**✅ Maintainability:**
- Clear separation of concerns
- Easy to locate code
- Changes isolated to specific layers

**✅ Scalability:**
- Add features without affecting existing code
- Multiple teams can work independently
- Clear boundaries and contracts

---

#### When to Use Clean Architecture

**✅ Good For:**
- Large, complex applications
- Long-term projects (5+ years)
- Multiple teams
- Complex business logic
- Need for high testability
- Potential framework migrations

**❌ Overkill For:**
- Small applications
- Prototypes/MVPs
- Simple CRUD apps
- Short-lived projects
- Small teams (1-2 developers)

---

#### Interview Tips

✅ **Key Points:**
- Dependency Rule: dependencies point inward
- Business logic in domain/application layers
- React is a detail (outer layer)
- Repository pattern for data access
- Use cases for application logic
- Dependency injection for loose coupling

✅ **When to Mention:**
- Enterprise application architecture
- Long-term maintainability
- Testing strategies
- Framework independence
- Complex business logic

✅ **Common Follow-ups:**
- "Isn't this overkill for most React apps?"
- "How do you handle state management?"
- "What about performance?"
- "How to implement dependency injection in React?"
- "Difference between Clean Architecture and typical React app?"

✅ **Perfect Answer Structure:**
1. Define: Separation of concerns with dependency inversion
2. Layers: Entities, Use Cases, Interface Adapters, Frameworks
3. Dependency Rule: Inner layers don't know outer layers
4. Benefits: Testable, maintainable, framework-independent
5. Trade-offs: More complex, more boilerplate
6. When to use: Large apps with complex business logic

</details>

---

### 165. How do you implement error boundaries effectively?

<details>
<summary>View Answer</summary>

**Error Boundaries** are React components that catch JavaScript errors anywhere in their child component tree, log those errors, and display a fallback UI. Implementing them effectively involves strategic placement, proper error handling, logging, and user experience considerations.

#### Basic Error Boundary Implementation

**Simple Error Boundary:**
```tsx
import React, { Component, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
  error: Error | null;
}

class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error: Error): State {
    // Update state so next render shows fallback UI
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    // Log error to error reporting service
    console.error('Error caught by boundary:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        this.props.fallback || (
          <div>
            <h2>Something went wrong</h2>
            <details style={{ whiteSpace: 'pre-wrap' }}>
              {this.state.error?.toString()}
            </details>
          </div>
        )
      );
    }

    return this.props.children;
  }
}

export default ErrorBoundary;
```

**Usage:**
```tsx
function App() {
  return (
    <ErrorBoundary fallback={<div>Error occurred!</div>}>
      <MyComponent />
    </ErrorBoundary>
  );
}
```

---

#### Advanced Error Boundary with Features

**Feature-Rich Error Boundary:**
```tsx
import React, { Component, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback?: (error: Error, reset: () => void) => ReactNode;
  onError?: (error: Error, errorInfo: React.ErrorInfo) => void;
  resetKeys?: Array<string | number>;
  isolate?: boolean;
}

interface State {
  hasError: boolean;
  error: Error | null;
  errorInfo: React.ErrorInfo | null;
}

class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = {
      hasError: false,
      error: null,
      errorInfo: null,
    };
  }

  static getDerivedStateFromError(error: Error): Partial<State> {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    // Log to error reporting service
    this.setState({ errorInfo });
    this.props.onError?.(error, errorInfo);

    // Log to console in development
    if (process.env.NODE_ENV === 'development') {
      console.error('Error Boundary caught:', error);
      console.error('Component Stack:', errorInfo.componentStack);
    }
  }

  componentDidUpdate(prevProps: Props) {
    // Reset error when resetKeys change
    if (this.props.resetKeys) {
      const hasResetKeyChanged = this.props.resetKeys.some(
        (key, index) => key !== prevProps.resetKeys?.[index]
      );

      if (hasResetKeyChanged && this.state.hasError) {
        this.reset();
      }
    }
  }

  reset = () => {
    this.setState({
      hasError: false,
      error: null,
      errorInfo: null,
    });
  };

  render() {
    if (this.state.hasError && this.state.error) {
      // Custom fallback
      if (this.props.fallback) {
        return this.props.fallback(this.state.error, this.reset);
      }

      // Default fallback UI
      return (
        <div className="error-boundary-fallback">
          <h2>Something went wrong</h2>
          <details>
            <summary>Error details</summary>
            <pre>{this.state.error.toString()}</pre>
            {this.state.errorInfo && (
              <pre>{this.state.errorInfo.componentStack}</pre>
            )}
          </details>
          <button onClick={this.reset}>Try again</button>
        </div>
      );
    }

    return this.props.children;
  }
}

export default ErrorBoundary;
```

**Usage with Reset:**
```tsx
function App() {
  const [userId, setUserId] = useState('1');

  return (
    <ErrorBoundary
      resetKeys={[userId]} // Reset when userId changes
      fallback={(error, reset) => (
        <div>
          <h2>Error: {error.message}</h2>
          <button onClick={reset}>Try Again</button>
        </div>
      )}
      onError={(error, errorInfo) => {
        // Send to error tracking service
        logErrorToService(error, errorInfo);
      }}
    >
      <UserProfile userId={userId} />
    </ErrorBoundary>
  );
}
```

---

#### Strategic Placement

**1. App-Level Error Boundary:**
```tsx
function App() {
  return (
    <ErrorBoundary
      fallback={(error, reset) => (
        <div className="app-error">
          <h1>Application Error</h1>
          <p>We're sorry, something went wrong.</p>
          <button onClick={() => window.location.reload()}>
            Reload Application
          </button>
        </div>
      )}
    >
      <Router>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/dashboard" element={<Dashboard />} />
        </Routes>
      </Router>
    </ErrorBoundary>
  );
}
```

**2. Route-Level Error Boundaries:**
```tsx
function App() {
  return (
    <Router>
      <Routes>
        <Route
          path="/"
          element={
            <ErrorBoundary fallback={<HomePage fallback="Home page error" />}>
              <Home />
            </ErrorBoundary>
          }
        />
        <Route
          path="/dashboard"
          element={
            <ErrorBoundary fallback={<DashboardError />}>
              <Dashboard />
            </ErrorBoundary>
          }
        />
        <Route
          path="/profile"
          element={
            <ErrorBoundary>
              <Profile />
            </ErrorBoundary>
          }
        />
      </Routes>
    </Router>
  );
}
```

**3. Feature-Level Error Boundaries:**
```tsx
function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>

      <ErrorBoundary
        fallback={<div>Failed to load statistics</div>}
        isolate
      >
        <Statistics />
      </ErrorBoundary>

      <ErrorBoundary
        fallback={<div>Failed to load chart</div>}
        isolate
      >
        <Chart />
      </ErrorBoundary>

      <ErrorBoundary
        fallback={<div>Failed to load user list</div>}
        isolate
      >
        <UserList />
      </ErrorBoundary>
    </div>
  );
}
```

**4. Component-Level Error Boundaries:**
```tsx
function ProductList() {
  return (
    <div>
      {products.map(product => (
        <ErrorBoundary
          key={product.id}
          fallback={<ProductCardFallback />}
        >
          <ProductCard product={product} />
        </ErrorBoundary>
      ))}
    </div>
  );
}
```

---

#### Error Logging Integration

**Integration with Error Tracking Services:**
```typescript
// services/errorTracking.ts
import * as Sentry from '@sentry/react';

interface ErrorContext {
  userId?: string;
  componentStack?: string;
  extra?: Record<string, any>;
}

export function logError(error: Error, context?: ErrorContext) {
  // Development logging
  if (process.env.NODE_ENV === 'development') {
    console.error('Error:', error);
    console.error('Context:', context);
    return;
  }

  // Production error tracking
  Sentry.captureException(error, {
    contexts: {
      react: {
        componentStack: context?.componentStack,
      },
    },
    user: context?.userId ? { id: context.userId } : undefined,
    extra: context?.extra,
  });
}
```

**Error Boundary with Tracking:**
```tsx
import { logError } from '@/services/errorTracking';

class ErrorBoundary extends Component<Props, State> {
  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    logError(error, {
      componentStack: errorInfo.componentStack,
      userId: this.props.userId,
      extra: {
        route: window.location.pathname,
        timestamp: new Date().toISOString(),
      },
    });

    this.props.onError?.(error, errorInfo);
  }

  // ... rest of implementation
}
```

---

#### Custom Error Boundary Hook (React 18+)

**Using react-error-boundary Library:**
```tsx
import { ErrorBoundary, useErrorHandler } from 'react-error-boundary';

// Error fallback component
function ErrorFallback({ error, resetErrorBoundary }: any) {
  return (
    <div role="alert">
      <h2>Something went wrong:</h2>
      <pre>{error.message}</pre>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

// Usage
function App() {
  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onError={(error, errorInfo) => {
        console.error('Logged error:', error, errorInfo);
      }}
      onReset={() => {
        // Reset app state
      }}
    >
      <MyApp />
    </ErrorBoundary>
  );
}

// Programmatic error handling in components
function MyComponent() {
  const handleError = useErrorHandler();

  const handleClick = async () => {
    try {
      await fetchData();
    } catch (error) {
      // Throw to nearest error boundary
      handleError(error);
    }
  };

  return <button onClick={handleClick}>Fetch Data</button>;
}
```

---

#### Error Recovery Strategies

**1. Auto-Retry Mechanism:**
```tsx
interface Props {
  children: ReactNode;
  maxRetries?: number;
}

interface State {
  hasError: boolean;
  error: Error | null;
  retryCount: number;
}

class RetryErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false, error: null, retryCount: 0 };
  }

  static getDerivedStateFromError(error: Error): Partial<State> {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('Error:', error, errorInfo);

    // Auto-retry after delay
    const maxRetries = this.props.maxRetries ?? 3;
    if (this.state.retryCount < maxRetries) {
      setTimeout(() => {
        this.setState(prev => ({
          hasError: false,
          error: null,
          retryCount: prev.retryCount + 1,
        }));
      }, 2000);
    }
  }

  render() {
    if (this.state.hasError) {
      const maxRetries = this.props.maxRetries ?? 3;
      if (this.state.retryCount < maxRetries) {
        return (
          <div>
            <p>Error occurred. Retrying... ({this.state.retryCount + 1}/{maxRetries})</p>
          </div>
        );
      }
      return (
        <div>
          <h2>Failed after {maxRetries} attempts</h2>
          <p>{this.state.error?.message}</p>
        </div>
      );
    }

    return this.props.children;
  }
}
```

**2. Graceful Degradation:**
```tsx
function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>

      {/* Critical feature - show error */}
      <ErrorBoundary fallback={<div>Critical error</div>}>
        <UserProfile />
      </ErrorBoundary>

      {/* Non-critical - hide on error */}
      <ErrorBoundary fallback={null}>
        <OptionalWidget />
      </ErrorBoundary>

      {/* Non-critical - show simplified version */}
      <ErrorBoundary
        fallback={<SimpleChart />} // Fallback to simpler component
      >
        <ComplexChart />
      </ErrorBoundary>
    </div>
  );
}
```

**3. User Feedback:**
```tsx
function ErrorFallback({ error, reset }: { error: Error; reset: () => void }) {
  const [feedbackSent, setFeedbackSent] = useState(false);

  const handleSendFeedback = async () => {
    await fetch('/api/feedback', {
      method: 'POST',
      body: JSON.stringify({
        error: error.message,
        stack: error.stack,
        url: window.location.href,
      }),
    });
    setFeedbackSent(true);
  };

  return (
    <div className="error-fallback">
      <h2>We encountered an error</h2>
      <p>We're sorry for the inconvenience.</p>
      
      <div className="error-actions">
        <button onClick={reset}>Try Again</button>
        <button onClick={() => window.location.reload()}>
          Reload Page
        </button>
        {!feedbackSent ? (
          <button onClick={handleSendFeedback}>Send Error Report</button>
        ) : (
          <span>Thank you for reporting!</span>
        )}
      </div>

      <details>
        <summary>Technical Details</summary>
        <pre>{error.message}</pre>
      </details>
    </div>
  );
}
```

---

#### Nested Error Boundaries

**Hierarchical Error Handling:**
```tsx
function App() {
  return (
    // Top-level: Catch all errors
    <ErrorBoundary
      fallback={(error) => (
        <div>
          <h1>Application Error</h1>
          <p>Please reload the page</p>
        </div>
      )}
    >
      <Router>
        <Routes>
          <Route
            path="/dashboard"
            element={
              // Route-level: Catch route-specific errors
              <ErrorBoundary fallback={<DashboardError />}>
                <DashboardLayout>
                  <div>
                    {/* Feature-level: Isolate feature errors */}
                    <ErrorBoundary fallback={<WidgetError />}>
                      <Widget1 />
                    </ErrorBoundary>

                    <ErrorBoundary fallback={<WidgetError />}>
                      <Widget2 />
                    </ErrorBoundary>
                  </div>
                </DashboardLayout>
              </ErrorBoundary>
            }
          />
        </Routes>
      </Router>
    </ErrorBoundary>
  );
}
```

---

#### Error Boundary Context

**Shared Error State:**
```tsx
import React, { createContext, useContext, useState, ReactNode } from 'react';

interface ErrorContextType {
  errors: Map<string, Error>;
  addError: (key: string, error: Error) => void;
  clearError: (key: string) => void;
  clearAllErrors: () => void;
}

const ErrorContext = createContext<ErrorContextType | undefined>(undefined);

export function ErrorProvider({ children }: { children: ReactNode }) {
  const [errors, setErrors] = useState<Map<string, Error>>(new Map());

  const addError = (key: string, error: Error) => {
    setErrors(prev => new Map(prev).set(key, error));
  };

  const clearError = (key: string) => {
    setErrors(prev => {
      const newMap = new Map(prev);
      newMap.delete(key);
      return newMap;
    });
  };

  const clearAllErrors = () => {
    setErrors(new Map());
  };

  return (
    <ErrorContext.Provider value={{ errors, addError, clearError, clearAllErrors }}>
      {children}
    </ErrorContext.Provider>
  );
}

export function useErrorContext() {
  const context = useContext(ErrorContext);
  if (!context) throw new Error('useErrorContext must be within ErrorProvider');
  return context;
}

// Usage
function App() {
  return (
    <ErrorProvider>
      <ErrorBoundary
        onError={(error) => {
          const { addError } = useErrorContext();
          addError('app-error', error);
        }}
      >
        <MyApp />
      </ErrorBoundary>
    </ErrorProvider>
  );
}
```

---

#### Testing Error Boundaries

**Unit Tests:**
```typescript
import { render, screen } from '@testing-library/react';
import ErrorBoundary from '../ErrorBoundary';

const ThrowError = () => {
  throw new Error('Test error');
};

describe('ErrorBoundary', () => {
  it('renders children when no error', () => {
    render(
      <ErrorBoundary>
        <div>Content</div>
      </ErrorBoundary>
    );

    expect(screen.getByText('Content')).toBeInTheDocument();
  });

  it('renders fallback on error', () => {
    // Suppress console.error for this test
    const spy = jest.spyOn(console, 'error').mockImplementation();

    render(
      <ErrorBoundary fallback={<div>Error occurred</div>}>
        <ThrowError />
      </ErrorBoundary>
    );

    expect(screen.getByText('Error occurred')).toBeInTheDocument();
    spy.mockRestore();
  });

  it('calls onError callback', () => {
    const onError = jest.fn();
    const spy = jest.spyOn(console, 'error').mockImplementation();

    render(
      <ErrorBoundary onError={onError}>
        <ThrowError />
      </ErrorBoundary>
    );

    expect(onError).toHaveBeenCalled();
    spy.mockRestore();
  });
});
```

---

#### Best Practices

**✅ Strategic Placement:**
- App-level boundary for critical errors
- Route-level boundaries for page isolation
- Feature-level boundaries for independent features
- Component-level for high-risk components

**✅ User Experience:**
- Clear error messages for users
- Recovery options (retry, reload)
- Graceful degradation when possible
- Hide errors for non-critical features

**✅ Error Logging:**
- Log all errors to tracking service
- Include component stack trace
- Add user context and metadata
- Different handling for dev vs production

**✅ Error Recovery:**
- Provide reset functionality
- Auto-retry for transient errors
- Clear state on recovery
- Use resetKeys to auto-reset on prop changes

**✅ Testing:**
- Test error scenarios
- Test fallback UI
- Test error logging
- Test recovery mechanisms

---

#### Common Patterns Summary

```tsx
// 1. Basic error boundary
<ErrorBoundary fallback={<ErrorUI />}>
  <Component />
</ErrorBoundary>

// 2. With custom fallback function
<ErrorBoundary
  fallback={(error, reset) => (
    <div>
      <p>{error.message}</p>
      <button onClick={reset}>Retry</button>
    </div>
  )}
>
  <Component />
</ErrorBoundary>

// 3. With error logging
<ErrorBoundary
  onError={(error, errorInfo) => {
    logErrorToService(error, errorInfo);
  }}
>
  <Component />
</ErrorBoundary>

// 4. With auto-reset on prop change
<ErrorBoundary resetKeys={[userId]}>
  <UserProfile userId={userId} />
</ErrorBoundary>

// 5. Multiple boundaries for isolation
<div>
  <ErrorBoundary fallback={<Error1 />}>
    <Feature1 />
  </ErrorBoundary>
  
  <ErrorBoundary fallback={<Error2 />}>
    <Feature2 />
  </ErrorBoundary>
</div>
```

---

#### Interview Tips

✅ **Key Points:**
- Class components with getDerivedStateFromError and componentDidCatch
- Strategic placement at different levels (app, route, feature, component)
- Integration with error tracking services
- User experience: fallback UI, reset functionality
- Limitations: doesn't catch async errors, event handlers

✅ **When to Mention:**
- Error handling strategies
- Production error monitoring
- User experience optimization
- Component isolation
- Resilient application architecture

✅ **Common Follow-ups:**
- "What errors do error boundaries NOT catch?"
- "How to handle async errors?"
- "Where should you place error boundaries?"
- "How to integrate with error tracking services?"
- "What about error boundaries in functional components?"

✅ **Perfect Answer Structure:**
1. Define: Class components that catch errors in children
2. Implementation: getDerivedStateFromError + componentDidCatch
3. Placement: App, route, feature, component levels
4. Features: Fallback UI, reset, logging, auto-retry
5. UX: Clear messaging, recovery options
6. Limitations: Async errors, event handlers, SSR

</details>

---

### 166. What is the Error Boundary pattern and its limitations?

<details>
<summary>View Answer</summary>

**Error Boundary pattern** is a React design pattern using class components to catch JavaScript errors anywhere in the child component tree, log errors, and display fallback UI instead of crashing the entire application. It's based on two lifecycle methods: `getDerivedStateFromError` and `componentDidCatch`.

#### The Pattern

**Core Implementation:**
```tsx
import React, { Component, ReactNode, ErrorInfo } from 'react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
  error: Error | null;
}

class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  // Called during render phase - must be pure
  // Use to update state for fallback UI
  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  // Called during commit phase - can have side effects
  // Use for logging errors
  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Error caught:', error);
    console.error('Error info:', errorInfo.componentStack);
    // Log to error reporting service
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || <h1>Something went wrong.</h1>;
    }

    return this.props.children;
  }
}

export default ErrorBoundary;
```

**Usage:**
```tsx
function App() {
  return (
    <ErrorBoundary fallback={<ErrorPage />}>
      <MyApplication />
    </ErrorBoundary>
  );
}
```

---

#### What Error Boundaries Catch

**✅ Errors Caught:**

1. **Render Errors:**
```tsx
function Component() {
  // ✅ Caught by error boundary
  throw new Error('Render error');
  return <div>Content</div>;
}
```

2. **Lifecycle Method Errors:**
```tsx
class MyComponent extends Component {
  componentDidMount() {
    // ✅ Caught by error boundary
    throw new Error('Lifecycle error');
  }

  render() {
    return <div>Content</div>;
  }
}
```

3. **Constructor Errors:**
```tsx
class MyComponent extends Component {
  constructor(props) {
    super(props);
    // ✅ Caught by error boundary
    throw new Error('Constructor error');
  }
}
```

4. **Child Component Errors:**
```tsx
function Parent() {
  return (
    <ErrorBoundary>
      <Child /> {/* ✅ Errors in Child are caught */}
    </ErrorBoundary>
  );
}
```

5. **Hook Errors (during render):**
```tsx
function Component() {
  const data = useMemo(() => {
    // ✅ Caught by error boundary
    throw new Error('Memo error');
  }, []);

  return <div>{data}</div>;
}
```

---

#### Limitations (What They DON'T Catch)

**❌ 1. Event Handler Errors:**

```tsx
function Component() {
  const handleClick = () => {
    // ❌ NOT caught by error boundary
    throw new Error('Click error');
  };

  return <button onClick={handleClick}>Click</button>;
}
```

**Workaround:**
```tsx
import { useErrorHandler } from 'react-error-boundary';

function Component() {
  const handleError = useErrorHandler();

  const handleClick = () => {
    try {
      throw new Error('Click error');
    } catch (error) {
      handleError(error); // Manually throw to error boundary
    }
  };

  return <button onClick={handleClick}>Click</button>;
}

// Or with try-catch
function Component() {
  const [error, setError] = useState(null);

  const handleClick = () => {
    try {
      throw new Error('Click error');
    } catch (err) {
      setError(err);
    }
  };

  if (error) throw error; // Throw in render to be caught

  return <button onClick={handleClick}>Click</button>;
}
```

**❌ 2. Asynchronous Errors:**

```tsx
function Component() {
  useEffect(() => {
    // ❌ NOT caught by error boundary
    setTimeout(() => {
      throw new Error('Async error');
    }, 1000);
  }, []);

  return <div>Component</div>;
}
```

**Workaround:**
```tsx
function Component() {
  const [error, setError] = useState(null);

  useEffect(() => {
    setTimeout(() => {
      try {
        throw new Error('Async error');
      } catch (err) {
        setError(err); // Set state instead
      }
    }, 1000);
  }, []);

  if (error) throw error; // Re-throw in render

  return <div>Component</div>;
}
```

**❌ 3. Async/Await Errors:**

```tsx
function Component() {
  useEffect(() => {
    const fetchData = async () => {
      // ❌ NOT caught by error boundary
      throw new Error('Fetch error');
    };
    fetchData();
  }, []);

  return <div>Component</div>;
}
```

**Workaround:**
```tsx
function Component() {
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await fetch('/api/data');
        if (!response.ok) throw new Error('Fetch failed');
        const data = await response.json();
      } catch (err) {
        setError(err);
      }
    };
    fetchData();
  }, []);

  if (error) throw error;

  return <div>Component</div>;
}
```

**❌ 4. Server-Side Rendering (SSR) Errors:**

```tsx
// During SSR
function Component() {
  // ❌ Error boundaries don't work the same in SSR
  throw new Error('SSR error');
}
```

**Workaround:**
```tsx
// Use try-catch on server
try {
  const html = ReactDOMServer.renderToString(
    <ErrorBoundary>
      <App />
    </ErrorBoundary>
  );
} catch (error) {
  // Handle SSR errors separately
  console.error('SSR error:', error);
  const html = ReactDOMServer.renderToString(<ErrorPage />);
}
```

**❌ 5. Errors in Error Boundary Itself:**

```tsx
class ErrorBoundary extends Component {
  componentDidCatch(error, errorInfo) {
    // ❌ If THIS throws, it's not caught
    throw new Error('Error in error boundary');
  }

  render() {
    // ❌ Errors in error boundary render not caught
    if (this.state.hasError) {
      throw new Error('Fallback error');
    }
    return this.props.children;
  }
}
```

**Workaround:**
```tsx
// Nest error boundaries
<ErrorBoundary>
  <ErrorBoundary>
    <App />
  </ErrorBoundary>
</ErrorBoundary>
```

**❌ 6. Errors During Hydration (React 18):**

```tsx
// React 18 hydration errors may not be caught consistently
function Component() {
  // ❌ Hydration mismatch may not trigger error boundary
  const isClient = typeof window !== 'undefined';
  return <div>{isClient ? 'Client' : 'Server'}</div>;
}
```

---

#### Handling Async Errors Properly

**Custom Hook for Async Error Handling:**
```typescript
import { useState, useCallback } from 'react';

function useAsyncError() {
  const [, setError] = useState();

  return useCallback(
    (error: Error) => {
      setError(() => {
        throw error;
      });
    },
    []
  );
}

// Usage
function Component() {
  const throwError = useAsyncError();

  const handleClick = async () => {
    try {
      await fetchData();
    } catch (error) {
      throwError(error); // Will be caught by error boundary
    }
  };

  return <button onClick={handleClick}>Fetch</button>;
}
```

**Query Library Integration:**
```tsx
import { useQuery } from '@tanstack/react-query';

function Component() {
  const { data, error } = useQuery({
    queryKey: ['data'],
    queryFn: fetchData,
    useErrorBoundary: true, // Throw errors to error boundary
  });

  // Error is thrown to nearest error boundary
  return <div>{data}</div>;
}
```

---

#### Error Boundary Alternatives

**1. React-Error-Boundary Library:**
```tsx
import { ErrorBoundary, useErrorHandler } from 'react-error-boundary';

function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div>
      <h2>Error: {error.message}</h2>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

function App() {
  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onReset={() => {
        // Reset app state
      }}
    >
      <MyApp />
    </ErrorBoundary>
  );
}

// In child components - handle async errors
function ChildComponent() {
  const handleError = useErrorHandler();

  const handleClick = async () => {
    try {
      await riskyOperation();
    } catch (error) {
      handleError(error); // Throws to error boundary
    }
  };

  return <button onClick={handleClick}>Click</button>;
}
```

**2. Suspense for Data Fetching (React 18):**
```tsx
import { Suspense } from 'react';

// Errors in suspended components are caught
function App() {
  return (
    <ErrorBoundary fallback={<ErrorUI />}>
      <Suspense fallback={<Loading />}>
        <DataComponent /> {/* Errors caught */}
      </Suspense>
    </ErrorBoundary>
  );
}
```

---

#### Best Practices for Limitations

**1. Event Handler Pattern:**
```tsx
function Component() {
  const [error, setError] = useState<Error | null>(null);

  // Wrap event handlers with try-catch
  const handleClick = () => {
    try {
      riskyOperation();
    } catch (err) {
      setError(err as Error);
    }
  };

  // Re-throw in render for error boundary
  if (error) throw error;

  return <button onClick={handleClick}>Click</button>;
}
```

**2. Async Pattern:**
```tsx
function Component() {
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    const loadData = async () => {
      try {
        await fetchData();
      } catch (err) {
        setError(err as Error);
      }
    };
    loadData();
  }, []);

  if (error) throw error;

  return <div>Component</div>;
}
```

**3. Global Error Handler:**
```tsx
// Setup global error handler
window.addEventListener('error', (event) => {
  console.error('Uncaught error:', event.error);
  // Send to error tracking service
});

window.addEventListener('unhandledrejection', (event) => {
  console.error('Unhandled promise rejection:', event.reason);
  // Send to error tracking service
});
```

---

#### Comparison Table

| Error Type | Caught by Error Boundary | Solution |
|------------|-------------------------|----------|
| Render errors | ✅ Yes | Use error boundary |
| Lifecycle errors | ✅ Yes | Use error boundary |
| Constructor errors | ✅ Yes | Use error boundary |
| Event handlers | ❌ No | Try-catch + rethrow in render |
| Async/await | ❌ No | Try-catch + set state + rethrow |
| setTimeout/setInterval | ❌ No | Try-catch + set state + rethrow |
| Promise rejections | ❌ No | .catch() + set state + rethrow |
| SSR errors | ❌ Partial | Server-side try-catch |
| Error boundary itself | ❌ No | Nest error boundaries |
| Hydration errors | ❌ Inconsistent | Avoid hydration mismatches |

---

#### Why These Limitations Exist

**1. Event Handlers:**
- Executed outside React's render cycle
- React doesn't know when they're called
- Developer should handle them explicitly

**2. Async Code:**
- Executes after render completes
- Error boundary only active during render
- No way to catch errors after render

**3. Design Philosophy:**
- Error boundaries designed for render-time errors
- Async errors are application logic concerns
- Separation of concerns

---

#### Workaround Summary

```tsx
// Pattern 1: Rethrow in render
function Component() {
  const [error, setError] = useState(null);

  const handleAsync = async () => {
    try {
      await operation();
    } catch (err) {
      setError(err);
    }
  };

  if (error) throw error; // Caught by error boundary

  return <button onClick={handleAsync}>Click</button>;
}

// Pattern 2: useErrorHandler hook
import { useErrorHandler } from 'react-error-boundary';

function Component() {
  const handleError = useErrorHandler();

  const handleAsync = async () => {
    try {
      await operation();
    } catch (err) {
      handleError(err); // Throws to error boundary
    }
  };

  return <button onClick={handleAsync}>Click</button>;
}

// Pattern 3: Query library
import { useQuery } from '@tanstack/react-query';

function Component() {
  const { data } = useQuery({
    queryKey: ['data'],
    queryFn: fetchData,
    useErrorBoundary: true, // Auto-throw to boundary
  });

  return <div>{data}</div>;
}
```

---

#### Interview Tips

✅ **Key Points:**
- Class component pattern with two lifecycle methods
- getDerivedStateFromError: update state (render phase)
- componentDidCatch: side effects like logging (commit phase)
- Catches render-time errors only
- Doesn't catch: event handlers, async, promises, SSR, errors in itself

✅ **When to Mention:**
- Error handling strategies
- Limitations of React error handling
- Production error monitoring
- Async error handling
- Defensive programming

✅ **Common Follow-ups:**
- "Why don't error boundaries catch async errors?"
- "How to handle errors in event handlers?"
- "Can you use error boundaries with functional components?"
- "What's the difference between getDerivedStateFromError and componentDidCatch?"
- "How to handle promise rejections?"

✅ **Perfect Answer Structure:**
1. Define: Class component pattern for catching errors
2. What it catches: Render, lifecycle, constructor errors
3. Limitations: Event handlers, async, promises, SSR
4. Why limitations exist: Outside render cycle
5. Workarounds: Try-catch + rethrow, useErrorHandler, query libraries
6. Best practice: Use for render errors, handle async separately

✅ **Remember:**
- Error boundaries are class components (no functional alternative)
- getDerivedStateFromError is for state updates (pure)
- componentDidCatch is for side effects (logging)
- Most limitations have workarounds via rethrow pattern
- Production apps need both error boundaries AND async error handling

</details>

---

### 167. How do you handle asynchronous errors in React?

<details>
<summary>View Answer</summary>

**Asynchronous errors** in React (from promises, async/await, setTimeout, event handlers) are not caught by error boundaries. You need specific strategies to handle them properly including try-catch blocks, error states, custom hooks, and integration with libraries.

#### Why Error Boundaries Don't Catch Async Errors

**Error boundaries only catch errors during:**
- Rendering
- Lifecycle methods
- Constructors of child components

**They DON'T catch:**
- Event handler errors
- Asynchronous code (setTimeout, promises)
- Server-side rendering
- Errors in the error boundary itself

```tsx
// ❌ NOT caught by error boundary
function Component() {
  useEffect(() => {
    setTimeout(() => {
      throw new Error('Async error'); // Not caught!
    }, 1000);
  }, []);

  return <div>Component</div>;
}
```

---

#### Basic Patterns

**1. Try-Catch with State:**
```tsx
function Component() {
  const [data, setData] = useState(null);
  const [error, setError] = useState<Error | null>(null);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    const fetchData = async () => {
      setLoading(true);
      setError(null);

      try {
        const response = await fetch('/api/data');
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        const result = await response.json();
        setData(result);
      } catch (err) {
        setError(err as Error);
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, []);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  if (!data) return null;

  return <div>{JSON.stringify(data)}</div>;
}
```

**2. Rethrow to Error Boundary:**
```tsx
function Component() {
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await fetch('/api/data');
        const data = await response.json();
      } catch (err) {
        setError(err as Error);
      }
    };

    fetchData();
  }, []);

  // Throw in render to be caught by error boundary
  if (error) throw error;

  return <div>Component</div>;
}
```

**3. Event Handler Errors:**
```tsx
function Component() {
  const [error, setError] = useState<Error | null>(null);

  const handleClick = async () => {
    try {
      await riskyOperation();
    } catch (err) {
      setError(err as Error);
      // Optionally log to error service
      console.error('Error in event handler:', err);
    }
  };

  if (error) return <div>Error: {error.message}</div>;

  return <button onClick={handleClick}>Click Me</button>;
}
```

---

#### Custom Hooks for Error Handling

**1. useAsyncError Hook:**
```typescript
import { useState, useCallback } from 'react';

/**
 * Hook to throw async errors to error boundary
 */
function useAsyncError() {
  const [, setError] = useState();

  return useCallback(
    (error: Error) => {
      setError(() => {
        throw error;
      });
    },
    []
  );
}

// Usage
function Component() {
  const throwError = useAsyncError();

  useEffect(() => {
    const fetchData = async () => {
      try {
        await fetch('/api/data');
      } catch (error) {
        throwError(error as Error); // Throws to error boundary
      }
    };

    fetchData();
  }, [throwError]);

  return <div>Component</div>;
}
```

**2. useAsync Hook:**
```typescript
import { useState, useEffect, useCallback } from 'react';

interface AsyncState<T> {
  data: T | null;
  error: Error | null;
  loading: boolean;
}

function useAsync<T>(
  asyncFunction: () => Promise<T>,
  immediate = true
) {
  const [state, setState] = useState<AsyncState<T>>({
    data: null,
    error: null,
    loading: immediate,
  });

  const execute = useCallback(async () => {
    setState({ data: null, error: null, loading: true });

    try {
      const data = await asyncFunction();
      setState({ data, error: null, loading: false });
      return data;
    } catch (error) {
      setState({ data: null, error: error as Error, loading: false });
      throw error;
    }
  }, [asyncFunction]);

  useEffect(() => {
    if (immediate) {
      execute();
    }
  }, [execute, immediate]);

  return { ...state, execute };
}

// Usage
function Component() {
  const { data, error, loading } = useAsync(
    async () => {
      const response = await fetch('/api/data');
      return response.json();
    },
    true
  );

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  if (!data) return null;

  return <div>{JSON.stringify(data)}</div>;
}
```

**3. useSafeAsync Hook (with cleanup):**
```typescript
import { useState, useEffect, useCallback, useRef } from 'react';

function useSafeAsync<T>() {
  const [state, setState] = useState<{
    data: T | null;
    error: Error | null;
    loading: boolean;
  }>({ data: null, error: null, loading: false });

  const isMountedRef = useRef(true);

  useEffect(() => {
    return () => {
      isMountedRef.current = false;
    };
  }, []);

  const run = useCallback(async (promise: Promise<T>) => {
    setState({ data: null, error: null, loading: true });

    try {
      const data = await promise;
      if (isMountedRef.current) {
        setState({ data, error: null, loading: false });
      }
      return data;
    } catch (error) {
      if (isMountedRef.current) {
        setState({ data: null, error: error as Error, loading: false });
      }
      throw error;
    }
  }, []);

  return { ...state, run };
}

// Usage
function Component() {
  const { data, error, loading, run } = useSafeAsync<DataType>();

  useEffect(() => {
    run(fetch('/api/data').then(r => r.json()));
  }, [run]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return <div>{data && JSON.stringify(data)}</div>;
}
```

---

#### React Query / TanStack Query Integration

**1. Basic Usage with Error Handling:**
```tsx
import { useQuery } from '@tanstack/react-query';

function Component() {
  const { data, error, isLoading, isError } = useQuery({
    queryKey: ['userData'],
    queryFn: async () => {
      const response = await fetch('/api/user');
      if (!response.ok) {
        throw new Error('Failed to fetch user');
      }
      return response.json();
    },
    retry: 3, // Retry 3 times on failure
    retryDelay: attemptIndex => Math.min(1000 * 2 ** attemptIndex, 30000),
  });

  if (isLoading) return <div>Loading...</div>;
  if (isError) return <div>Error: {error.message}</div>;

  return <div>{data.name}</div>;
}
```

**2. Throw to Error Boundary:**
```tsx
import { useQuery } from '@tanstack/react-query';

function Component() {
  const { data } = useQuery({
    queryKey: ['userData'],
    queryFn: fetchUser,
    useErrorBoundary: true, // Throw errors to error boundary
  });

  return <div>{data.name}</div>;
}

// Wrapped with error boundary
function App() {
  return (
    <ErrorBoundary fallback={<ErrorFallback />}>
      <Component />
    </ErrorBoundary>
  );
}
```

**3. Global Error Handler:**
```tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      onError: (error) => {
        // Global error handler
        console.error('Query error:', error);
        // Log to error tracking service
        logErrorToService(error);
      },
      retry: (failureCount, error) => {
        // Custom retry logic
        if (error.status === 404) return false;
        return failureCount < 3;
      },
    },
    mutations: {
      onError: (error) => {
        console.error('Mutation error:', error);
        toast.error('Operation failed');
      },
    },
  },
});

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <MyApp />
    </QueryClientProvider>
  );
}
```

**4. Error Boundary Integration:**
```tsx
import { useQueryErrorResetBoundary } from '@tanstack/react-query';
import { ErrorBoundary } from 'react-error-boundary';

function App() {
  const { reset } = useQueryErrorResetBoundary();

  return (
    <ErrorBoundary
      onReset={reset}
      fallbackRender={({ error, resetErrorBoundary }) => (
        <div>
          <h2>Error: {error.message}</h2>
          <button onClick={resetErrorBoundary}>Try again</button>
        </div>
      )}
    >
      <Component />
    </ErrorBoundary>
  );
}
```

---

#### React Error Boundary Library

**Using react-error-boundary:**
```tsx
import { ErrorBoundary, useErrorHandler } from 'react-error-boundary';

function ErrorFallback({ error, resetErrorBoundary }: any) {
  return (
    <div role="alert">
      <p>Something went wrong:</p>
      <pre>{error.message}</pre>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

function App() {
  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onReset={() => {
        // Reset app state here
      }}
    >
      <MyComponent />
    </ErrorBoundary>
  );
}

// In child components
function MyComponent() {
  const handleError = useErrorHandler();

  const handleClick = async () => {
    try {
      await riskyOperation();
    } catch (error) {
      handleError(error); // Throws to nearest error boundary
    }
  };

  return <button onClick={handleClick}>Do Something</button>;
}
```

---

#### Global Error Handlers

**1. Window Error Events:**
```typescript
// In main App or index file
useEffect(() => {
  // Catch unhandled errors
  const handleError = (event: ErrorEvent) => {
    console.error('Uncaught error:', event.error);
    // Log to error tracking service
    logErrorToService(event.error);
  };

  // Catch unhandled promise rejections
  const handleUnhandledRejection = (event: PromiseRejectionEvent) => {
    console.error('Unhandled promise rejection:', event.reason);
    logErrorToService(event.reason);
  };

  window.addEventListener('error', handleError);
  window.addEventListener('unhandledrejection', handleUnhandledRejection);

  return () => {
    window.removeEventListener('error', handleError);
    window.removeEventListener('unhandledrejection', handleUnhandledRejection);
  };
}, []);
```

**2. Axios Interceptors:**
```typescript
import axios from 'axios';

const apiClient = axios.create({
  baseURL: '/api',
});

// Response interceptor for error handling
apiClient.interceptors.response.use(
  (response) => response,
  (error) => {
    // Handle different error types
    if (error.response) {
      // Server responded with error status
      const status = error.response.status;
      
      switch (status) {
        case 401:
          // Redirect to login
          window.location.href = '/login';
          break;
        case 403:
          console.error('Forbidden');
          break;
        case 404:
          console.error('Not found');
          break;
        case 500:
          console.error('Server error');
          break;
        default:
          console.error('API error:', error);
      }
    } else if (error.request) {
      // Request made but no response
      console.error('Network error:', error);
    } else {
      // Something else happened
      console.error('Error:', error.message);
    }

    // Log to error tracking service
    logErrorToService(error);

    return Promise.reject(error);
  }
);

export default apiClient;
```

---

#### Error Tracking Services

**1. Sentry Integration:**
```typescript
import * as Sentry from '@sentry/react';
import { BrowserTracing } from '@sentry/tracing';

Sentry.init({
  dsn: 'your-sentry-dsn',
  integrations: [
    new BrowserTracing(),
    new Sentry.Replay(),
  ],
  tracesSampleRate: 1.0,
  replaysSessionSampleRate: 0.1,
  replaysOnErrorSampleRate: 1.0,
});

// Usage in component
function Component() {
  const handleClick = async () => {
    try {
      await riskyOperation();
    } catch (error) {
      Sentry.captureException(error, {
        contexts: {
          component: {
            name: 'Component',
          },
        },
        tags: {
          section: 'user-action',
        },
      });
    }
  };

  return <button onClick={handleClick}>Click</button>;
}
```

**2. Custom Error Logger:**
```typescript
// services/errorLogger.ts
interface ErrorContext {
  component?: string;
  userId?: string;
  action?: string;
  extra?: Record<string, any>;
}

class ErrorLogger {
  private static instance: ErrorLogger;

  private constructor() {}

  static getInstance(): ErrorLogger {
    if (!ErrorLogger.instance) {
      ErrorLogger.instance = new ErrorLogger();
    }
    return ErrorLogger.instance;
  }

  logError(error: Error, context?: ErrorContext) {
    const errorData = {
      message: error.message,
      stack: error.stack,
      timestamp: new Date().toISOString(),
      url: window.location.href,
      userAgent: navigator.userAgent,
      ...context,
    };

    // Development
    if (process.env.NODE_ENV === 'development') {
      console.error('Error logged:', errorData);
      return;
    }

    // Production - send to backend
    fetch('/api/errors', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(errorData),
    }).catch(err => {
      console.error('Failed to log error:', err);
    });
  }
}

export default ErrorLogger.getInstance();

// Usage
import errorLogger from '@/services/errorLogger';

function Component() {
  const handleClick = async () => {
    try {
      await riskyOperation();
    } catch (error) {
      errorLogger.logError(error as Error, {
        component: 'Component',
        action: 'button-click',
        userId: currentUser.id,
      });
    }
  };

  return <button onClick={handleClick}>Click</button>;
}
```

---

#### Complete Error Handling Pattern

**Comprehensive Example:**
```tsx
import { useState, useEffect } from 'react';
import { ErrorBoundary } from 'react-error-boundary';
import { useQuery } from '@tanstack/react-query';
import errorLogger from '@/services/errorLogger';

// 1. Error Fallback Component
function ErrorFallback({ error, resetErrorBoundary }: any) {
  useEffect(() => {
    errorLogger.logError(error, {
      component: 'ErrorFallback',
    });
  }, [error]);

  return (
    <div className="error-container">
      <h2>Something went wrong</h2>
      <p>{error.message}</p>
      <button onClick={resetErrorBoundary}>Try Again</button>
    </div>
  );
}

// 2. Component with async operations
function DataComponent() {
  const [manualError, setManualError] = useState<Error | null>(null);

  // React Query with error handling
  const { data, error, isLoading } = useQuery({
    queryKey: ['data'],
    queryFn: fetchData,
    onError: (err) => {
      errorLogger.logError(err as Error, {
        component: 'DataComponent',
        action: 'data-fetch',
      });
    },
  });

  // Manual async operation
  const handleAction = async () => {
    try {
      await performAction();
    } catch (err) {
      const error = err as Error;
      errorLogger.logError(error, {
        component: 'DataComponent',
        action: 'manual-action',
      });
      setManualError(error);
    }
  };

  // Throw to error boundary
  if (manualError) throw manualError;

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      <div>{JSON.stringify(data)}</div>
      <button onClick={handleAction}>Perform Action</button>
    </div>
  );
}

// 3. App with error boundary
function App() {
  useEffect(() => {
    // Global error handlers
    const handleError = (event: ErrorEvent) => {
      errorLogger.logError(event.error, {
        type: 'uncaught-error',
      });
    };

    const handleUnhandledRejection = (event: PromiseRejectionEvent) => {
      errorLogger.logError(
        new Error(event.reason),
        { type: 'unhandled-rejection' }
      );
    };

    window.addEventListener('error', handleError);
    window.addEventListener('unhandledrejection', handleUnhandledRejection);

    return () => {
      window.removeEventListener('error', handleError);
      window.removeEventListener('unhandledrejection', handleUnhandledRejection);
    };
  }, []);

  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onReset={() => {
        // Reset app state
      }}
    >
      <DataComponent />
    </ErrorBoundary>
  );
}

export default App;
```

---

#### Best Practices

**✅ Always Handle Async Errors:**
```tsx
// ❌ Bad - unhandled rejection
const fetchData = async () => {
  const data = await fetch('/api/data');
  return data.json();
};

// ✅ Good - handled
const fetchData = async () => {
  try {
    const response = await fetch('/api/data');
    if (!response.ok) throw new Error('Fetch failed');
    return await response.json();
  } catch (error) {
    console.error('Error fetching data:', error);
    throw error; // Re-throw after logging
  }
};
```

**✅ Use Custom Hooks:**
```tsx
// Encapsulate error handling logic
function useDataFetch(url: string) {
  const [state, setState] = useState({
    data: null,
    error: null,
    loading: true,
  });

  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await fetch(url);
        const data = await response.json();
        setState({ data, error: null, loading: false });
      } catch (error) {
        setState({ data: null, error: error as Error, loading: false });
      }
    };

    fetchData();
  }, [url]);

  return state;
}
```

**✅ Log Errors:**
```tsx
// Always log for debugging
try {
  await operation();
} catch (error) {
  console.error('Operation failed:', error);
  errorLogger.logError(error);
  // Then handle the error
}
```

**✅ Provide User Feedback:**
```tsx
const handleSubmit = async () => {
  try {
    await submitForm();
    toast.success('Form submitted!');
  } catch (error) {
    toast.error('Failed to submit form');
    console.error(error);
  }
};
```

---

#### Strategy Summary

| Pattern | Use Case | Example |
|---------|----------|----------|
| Try-Catch + State | Simple local errors | `catch (err) { setError(err) }` |
| Rethrow to Boundary | Critical errors | `if (error) throw error` |
| useAsyncError Hook | Programmatic throw | `throwError(error)` |
| React Query | Data fetching | `useQuery({ useErrorBoundary: true })` |
| useErrorHandler | Library integration | `handleError(error)` |
| Global Handlers | Uncaught errors | `window.addEventListener('error')` |
| Error Logging | Production monitoring | `Sentry.captureException(error)` |

---

#### Interview Tips

✅ **Key Points:**
- Error boundaries don't catch async errors
- Use try-catch with state management
- Rethrow in render to use error boundaries
- Use custom hooks to encapsulate error handling
- Integrate with React Query or similar libraries
- Set up global error handlers for uncaught errors
- Log errors to tracking services in production

✅ **When to Mention:**
- Async operations and error handling
- Data fetching strategies
- Production error monitoring
- User experience during errors
- Debugging strategies

✅ **Common Follow-ups:**
- "Why don't error boundaries catch async errors?"
- "How do you handle API errors?"
- "What error tracking services have you used?"
- "How do you provide user feedback for errors?"
- "How do you handle network failures?"

✅ **Perfect Answer Structure:**
1. Problem: Error boundaries don't catch async errors
2. Basic Solution: Try-catch with state
3. Advanced: Custom hooks (useAsync, useAsyncError)
4. Libraries: React Query, react-error-boundary
5. Global: Window error events, axios interceptors
6. Production: Error logging services (Sentry)
7. Best Practice: Combine multiple strategies

</details>

---

### 168. What is Atomic Design methodology?

<details>
<summary>View Answer</summary>

**Atomic Design** is a methodology for creating design systems with five distinct levels of components, inspired by chemistry. Created by Brad Frost, it breaks down interfaces into fundamental building blocks (atoms) that combine to form increasingly complex structures (molecules, organisms, templates, pages).

#### The Five Levels

**1. Atoms** - Basic building blocks (buttons, inputs, labels)  
**2. Molecules** - Simple groups of atoms (search form, card header)  
**3. Organisms** - Complex UI components (header, form, product grid)  
**4. Templates** - Page-level layout structures  
**5. Pages** - Specific instances of templates with real content  

---

#### Level 1: Atoms

**Definition:** Basic HTML elements and fundamental components that can't be broken down further without losing their meaning.

**Examples:**
- Buttons
- Inputs
- Labels
- Icons
- Typography
- Colors

**Implementation:**
```tsx
// atoms/Button.tsx
import React from 'react';
import './Button.css';

interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'outline';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
  children: React.ReactNode;
  onClick?: () => void;
}

export function Button({
  variant = 'primary',
  size = 'medium',
  disabled = false,
  children,
  onClick,
}: ButtonProps) {
  return (
    <button
      className={`btn btn--${variant} btn--${size}`}
      disabled={disabled}
      onClick={onClick}
    >
      {children}
    </button>
  );
}

// atoms/Input.tsx
interface InputProps {
  type?: 'text' | 'email' | 'password' | 'number';
  placeholder?: string;
  value?: string;
  onChange?: (e: React.ChangeEvent<HTMLInputElement>) => void;
  error?: boolean;
}

export function Input({
  type = 'text',
  placeholder,
  value,
  onChange,
  error,
}: InputProps) {
  return (
    <input
      type={type}
      placeholder={placeholder}
      value={value}
      onChange={onChange}
      className={`input ${error ? 'input--error' : ''}`}
    />
  );
}

// atoms/Label.tsx
interface LabelProps {
  htmlFor?: string;
  required?: boolean;
  children: React.ReactNode;
}

export function Label({ htmlFor, required, children }: LabelProps) {
  return (
    <label htmlFor={htmlFor} className="label">
      {children}
      {required && <span className="label__required">*</span>}
    </label>
  );
}

// atoms/Icon.tsx
interface IconProps {
  name: string;
  size?: 'small' | 'medium' | 'large';
  color?: string;
}

export function Icon({ name, size = 'medium', color }: IconProps) {
  return (
    <i
      className={`icon icon--${name} icon--${size}`}
      style={{ color }}
      aria-hidden="true"
    />
  );
}
```

---

#### Level 2: Molecules

**Definition:** Simple groups of atoms functioning together as a unit. They have a single, clear purpose.

**Examples:**
- Search form (input + button)
- Form field (label + input + error message)
- Card header (title + icon)
- Navigation item (icon + label)

**Implementation:**
```tsx
// molecules/FormField.tsx
import { Label } from '../atoms/Label';
import { Input } from '../atoms/Input';
import './FormField.css';

interface FormFieldProps {
  id: string;
  label: string;
  type?: 'text' | 'email' | 'password';
  value: string;
  onChange: (e: React.ChangeEvent<HTMLInputElement>) => void;
  error?: string;
  required?: boolean;
  placeholder?: string;
}

export function FormField({
  id,
  label,
  type = 'text',
  value,
  onChange,
  error,
  required,
  placeholder,
}: FormFieldProps) {
  return (
    <div className="form-field">
      <Label htmlFor={id} required={required}>
        {label}
      </Label>
      <Input
        type={type}
        id={id}
        value={value}
        onChange={onChange}
        placeholder={placeholder}
        error={!!error}
      />
      {error && <span className="form-field__error">{error}</span>}
    </div>
  );
}

// molecules/SearchBar.tsx
import { Input } from '../atoms/Input';
import { Button } from '../atoms/Button';
import { Icon } from '../atoms/Icon';

interface SearchBarProps {
  value: string;
  onChange: (e: React.ChangeEvent<HTMLInputElement>) => void;
  onSubmit: () => void;
  placeholder?: string;
}

export function SearchBar({
  value,
  onChange,
  onSubmit,
  placeholder = 'Search...',
}: SearchBarProps) {
  return (
    <div className="search-bar">
      <Input
        type="text"
        value={value}
        onChange={onChange}
        placeholder={placeholder}
      />
      <Button variant="primary" onClick={onSubmit}>
        <Icon name="search" size="small" />
        Search
      </Button>
    </div>
  );
}

// molecules/CardHeader.tsx
import { Icon } from '../atoms/Icon';

interface CardHeaderProps {
  title: string;
  icon?: string;
  action?: React.ReactNode;
}

export function CardHeader({ title, icon, action }: CardHeaderProps) {
  return (
    <div className="card-header">
      <div className="card-header__title">
        {icon && <Icon name={icon} />}
        <h3>{title}</h3>
      </div>
      {action && <div className="card-header__action">{action}</div>}
    </div>
  );
}
```

---

#### Level 3: Organisms

**Definition:** Complex UI components composed of molecules and/or atoms. They form distinct sections of an interface.

**Examples:**
- Header (logo, navigation, search)
- Login form (multiple form fields + button)
- Product card (image, title, price, button)
- Sidebar navigation

**Implementation:**
```tsx
// organisms/Header.tsx
import { Button } from '../atoms/Button';
import { SearchBar } from '../molecules/SearchBar';
import { Icon } from '../atoms/Icon';
import './Header.css';

interface HeaderProps {
  logo: string;
  navigationItems: Array<{ label: string; href: string }>;
  onSearch: (query: string) => void;
  user?: { name: string; avatar: string };
  onLogin?: () => void;
  onLogout?: () => void;
}

export function Header({
  logo,
  navigationItems,
  onSearch,
  user,
  onLogin,
  onLogout,
}: HeaderProps) {
  const [searchQuery, setSearchQuery] = React.useState('');

  return (
    <header className="header">
      <div className="header__logo">
        <img src={logo} alt="Logo" />
      </div>

      <nav className="header__nav">
        {navigationItems.map(item => (
          <a key={item.href} href={item.href} className="header__nav-item">
            {item.label}
          </a>
        ))}
      </nav>

      <div className="header__search">
        <SearchBar
          value={searchQuery}
          onChange={(e) => setSearchQuery(e.target.value)}
          onSubmit={() => onSearch(searchQuery)}
        />
      </div>

      <div className="header__user">
        {user ? (
          <div className="header__user-menu">
            <img src={user.avatar} alt={user.name} />
            <span>{user.name}</span>
            <Button variant="outline" onClick={onLogout}>
              Logout
            </Button>
          </div>
        ) : (
          <Button variant="primary" onClick={onLogin}>
            Login
          </Button>
        )}
      </div>
    </header>
  );
}

// organisms/LoginForm.tsx
import { useState } from 'react';
import { FormField } from '../molecules/FormField';
import { Button } from '../atoms/Button';

interface LoginFormProps {
  onSubmit: (email: string, password: string) => Promise<void>;
  onForgotPassword: () => void;
}

export function LoginForm({ onSubmit, onForgotPassword }: LoginFormProps) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [errors, setErrors] = useState<Record<string, string>>({});
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setErrors({});
    setLoading(true);

    try {
      await onSubmit(email, password);
    } catch (error) {
      setErrors({ form: 'Invalid email or password' });
    } finally {
      setLoading(false);
    }
  };

  return (
    <form className="login-form" onSubmit={handleSubmit}>
      <h2>Login</h2>

      <FormField
        id="email"
        label="Email"
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        error={errors.email}
        required
      />

      <FormField
        id="password"
        label="Password"
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        error={errors.password}
        required
      />

      {errors.form && (
        <div className="login-form__error">{errors.form}</div>
      )}

      <Button
        variant="primary"
        size="large"
        disabled={loading}
        onClick={handleSubmit}
      >
        {loading ? 'Logging in...' : 'Login'}
      </Button>

      <button
        type="button"
        className="login-form__forgot-password"
        onClick={onForgotPassword}
      >
        Forgot password?
      </button>
    </form>
  );
}

// organisms/ProductCard.tsx
import { Button } from '../atoms/Button';
import { Icon } from '../atoms/Icon';

interface Product {
  id: string;
  name: string;
  price: number;
  image: string;
  rating: number;
  inStock: boolean;
}

interface ProductCardProps {
  product: Product;
  onAddToCart: (productId: string) => void;
  onViewDetails: (productId: string) => void;
}

export function ProductCard({
  product,
  onAddToCart,
  onViewDetails,
}: ProductCardProps) {
  return (
    <div className="product-card">
      <div className="product-card__image">
        <img src={product.image} alt={product.name} />
        {!product.inStock && (
          <div className="product-card__badge">Out of Stock</div>
        )}
      </div>

      <div className="product-card__content">
        <h3 className="product-card__title">{product.name}</h3>

        <div className="product-card__rating">
          {Array.from({ length: 5 }).map((_, i) => (
            <Icon
              key={i}
              name={i < product.rating ? 'star-filled' : 'star-outline'}
              size="small"
            />
          ))}
        </div>

        <div className="product-card__price">${product.price.toFixed(2)}</div>

        <div className="product-card__actions">
          <Button
            variant="primary"
            disabled={!product.inStock}
            onClick={() => onAddToCart(product.id)}
          >
            Add to Cart
          </Button>
          <Button
            variant="outline"
            onClick={() => onViewDetails(product.id)}
          >
            Details
          </Button>
        </div>
      </div>
    </div>
  );
}
```

---

#### Level 4: Templates

**Definition:** Page-level objects that place organisms into a layout. They focus on structure rather than content.

**Examples:**
- Main layout template
- Dashboard template
- Article template
- Checkout template

**Implementation:**
```tsx
// templates/MainLayout.tsx
import { Header } from '../organisms/Header';
import './MainLayout.css';

interface MainLayoutProps {
  header: React.ComponentProps<typeof Header>;
  sidebar?: React.ReactNode;
  children: React.ReactNode;
  footer?: React.ReactNode;
}

export function MainLayout({
  header,
  sidebar,
  children,
  footer,
}: MainLayoutProps) {
  return (
    <div className="main-layout">
      <Header {...header} />

      <div className="main-layout__body">
        {sidebar && (
          <aside className="main-layout__sidebar">{sidebar}</aside>
        )}

        <main className="main-layout__content">{children}</main>
      </div>

      {footer && <footer className="main-layout__footer">{footer}</footer>}
    </div>
  );
}

// templates/DashboardTemplate.tsx
import { Header } from '../organisms/Header';

interface DashboardTemplateProps {
  header: React.ReactNode;
  sidebar: React.ReactNode;
  stats: React.ReactNode;
  mainContent: React.ReactNode;
  recentActivity: React.ReactNode;
}

export function DashboardTemplate({
  header,
  sidebar,
  stats,
  mainContent,
  recentActivity,
}: DashboardTemplateProps) {
  return (
    <div className="dashboard-template">
      <div className="dashboard-template__header">{header}</div>

      <div className="dashboard-template__body">
        <aside className="dashboard-template__sidebar">{sidebar}</aside>

        <div className="dashboard-template__main">
          <section className="dashboard-template__stats">{stats}</section>
          <section className="dashboard-template__content">
            {mainContent}
          </section>
        </div>

        <aside className="dashboard-template__activity">
          {recentActivity}
        </aside>
      </div>
    </div>
  );
}

// templates/ArticleTemplate.tsx
interface ArticleTemplateProps {
  header: React.ReactNode;
  hero: React.ReactNode;
  content: React.ReactNode;
  sidebar: React.ReactNode;
  relatedArticles: React.ReactNode;
  comments: React.ReactNode;
  footer: React.ReactNode;
}

export function ArticleTemplate({
  header,
  hero,
  content,
  sidebar,
  relatedArticles,
  comments,
  footer,
}: ArticleTemplateProps) {
  return (
    <div className="article-template">
      {header}

      <div className="article-template__hero">{hero}</div>

      <div className="article-template__body">
        <article className="article-template__content">{content}</article>
        <aside className="article-template__sidebar">{sidebar}</aside>
      </div>

      <section className="article-template__related">
        {relatedArticles}
      </section>

      <section className="article-template__comments">{comments}</section>

      {footer}
    </div>
  );
}
```

---

#### Level 5: Pages

**Definition:** Specific instances of templates with real, representative content. Shows what the end user will see.

**Implementation:**
```tsx
// pages/HomePage.tsx
import { MainLayout } from '../templates/MainLayout';
import { Header } from '../organisms/Header';
import { ProductCard } from '../organisms/ProductCard';
import { useProducts } from '../hooks/useProducts';

export function HomePage() {
  const { products, loading } = useProducts();

  return (
    <MainLayout
      header={{
        logo: '/logo.png',
        navigationItems: [
          { label: 'Home', href: '/' },
          { label: 'Products', href: '/products' },
          { label: 'About', href: '/about' },
        ],
        onSearch: (query) => console.log('Search:', query),
      }}
    >
      <section className="home-page">
        <h1>Featured Products</h1>

        {loading ? (
          <div>Loading...</div>
        ) : (
          <div className="product-grid">
            {products.map(product => (
              <ProductCard
                key={product.id}
                product={product}
                onAddToCart={(id) => console.log('Add to cart:', id)}
                onViewDetails={(id) => console.log('View details:', id)}
              />
            ))}
          </div>
        )}
      </section>
    </MainLayout>
  );
}

// pages/DashboardPage.tsx
import { DashboardTemplate } from '../templates/DashboardTemplate';
import { Header } from '../organisms/Header';
import { useDashboardData } from '../hooks/useDashboardData';

export function DashboardPage() {
  const { stats, activities, charts } = useDashboardData();

  return (
    <DashboardTemplate
      header={
        <Header
          logo="/logo.png"
          navigationItems={[
            { label: 'Dashboard', href: '/dashboard' },
            { label: 'Analytics', href: '/analytics' },
          ]}
          onSearch={(q) => console.log(q)}
          user={{ name: 'John Doe', avatar: '/avatar.png' }}
        />
      }
      sidebar={<DashboardSidebar />}
      stats={<StatsOverview data={stats} />}
      mainContent={<ChartSection charts={charts} />}
      recentActivity={<ActivityFeed activities={activities} />}
    />
  );
}
```

---

#### Folder Structure

```
src/
├── components/
│   ├── atoms/
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   ├── Button.css
│   │   │   ├── Button.test.tsx
│   │   │   └── index.ts
│   │   ├── Input/
│   │   ├── Label/
│   │   ├── Icon/
│   │   └── index.ts
│   │
│   ├── molecules/
│   │   ├── FormField/
│   │   │   ├── FormField.tsx
│   │   │   ├── FormField.css
│   │   │   ├── FormField.test.tsx
│   │   │   └── index.ts
│   │   ├── SearchBar/
│   │   ├── CardHeader/
│   │   └── index.ts
│   │
│   ├── organisms/
│   │   ├── Header/
│   │   │   ├── Header.tsx
│   │   │   ├── Header.css
│   │   │   ├── Header.test.tsx
│   │   │   └── index.ts
│   │   ├── LoginForm/
│   │   ├── ProductCard/
│   │   └── index.ts
│   │
│   ├── templates/
│   │   ├── MainLayout/
│   │   │   ├── MainLayout.tsx
│   │   │   ├── MainLayout.css
│   │   │   └── index.ts
│   │   ├── DashboardTemplate/
│   │   └── index.ts
│   │
│   └── pages/
│       ├── HomePage/
│       │   ├── HomePage.tsx
│       │   ├── HomePage.css
│       │   └── index.ts
│       ├── DashboardPage/
│       └── index.ts
│
├── hooks/
├── utils/
└── styles/
    ├── tokens/
    │   ├── colors.css
    │   ├── typography.css
    │   └── spacing.css
    └── global.css
```

---

#### Benefits

**✅ Consistency:**
- Reusable components across the application
- Design system enforces consistent UI/UX
- Shared component library

**✅ Scalability:**
- Easy to add new pages using existing components
- Component composition over duplication
- Clear component hierarchy

**✅ Maintainability:**
- Changes to atoms propagate to all molecules/organisms
- Single source of truth for each component
- Easier to refactor and update

**✅ Collaboration:**
- Clear vocabulary for designers and developers
- Better communication between teams
- Easier onboarding for new team members

**✅ Testing:**
- Test components in isolation
- Build up from simple to complex
- Easier to identify bugs

**✅ Documentation:**
- Storybook integration for component showcase
- Visual regression testing
- Living style guide

---

#### Tools Integration

**1. Storybook:**
```tsx
// Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';

const meta: Meta<typeof Button> = {
  title: 'Atoms/Button',
  component: Button,
  tags: ['autodocs'],
};

export default meta;
type Story = StoryObj<typeof Button>;

export const Primary: Story = {
  args: {
    variant: 'primary',
    children: 'Primary Button',
  },
};

export const Secondary: Story = {
  args: {
    variant: 'secondary',
    children: 'Secondary Button',
  },
};

export const Small: Story = {
  args: {
    size: 'small',
    children: 'Small Button',
  },
};
```

**2. Design Tokens:**
```css
/* tokens/colors.css */
:root {
  /* Primary Colors */
  --color-primary-50: #e3f2fd;
  --color-primary-500: #2196f3;
  --color-primary-900: #0d47a1;

  /* Semantic Colors */
  --color-success: #4caf50;
  --color-error: #f44336;
  --color-warning: #ff9800;
}

/* tokens/typography.css */
:root {
  --font-family-base: 'Inter', sans-serif;
  --font-size-xs: 0.75rem;
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.125rem;
  --font-weight-normal: 400;
  --font-weight-bold: 700;
}

/* tokens/spacing.css */
:root {
  --spacing-xs: 0.25rem;
  --spacing-sm: 0.5rem;
  --spacing-md: 1rem;
  --spacing-lg: 1.5rem;
  --spacing-xl: 2rem;
}
```

---

#### Comparison with Other Patterns

| Aspect | Atomic Design | Feature-Based | Layer-Based |
|--------|--------------|---------------|-------------|
| Organization | Component complexity | Feature modules | Technical layers |
| Hierarchy | 5 levels (atoms to pages) | By feature | By layer (UI, logic, data) |
| Reusability | High (atoms/molecules) | Medium | Medium |
| Learning Curve | Medium (need to understand levels) | Low | Low |
| Best For | Design systems, component libraries | Large apps with distinct features | Separation of concerns |

---

#### When to Use Atomic Design

**✅ Good Fit:**
- Building a design system
- Need high component reusability
- Multiple teams working on UI
- Consistent design language required
- Component library development

**❌ May Not Fit:**
- Very small applications
- Rapid prototyping
- Tight deadlines
- Team unfamiliar with the methodology
- Highly custom, one-off interfaces

---

#### Common Pitfalls

**❌ Over-abstraction:**
```tsx
// Too granular
<Text size="large" weight="bold" color="primary">
  Title
</Text>

// Better - semantic component
<Title>Title</Title>
```

**❌ Unclear Boundaries:**
```tsx
// Is this a molecule or organism?
<UserCard /> // Could be either

// Be pragmatic, focus on reusability over strict classification
```

**❌ Rigid Adherence:**
```tsx
// Don't force everything into the methodology
// If it doesn't fit, that's okay
```

---

#### Best Practices

**✅ Start Small:**
- Begin with atoms and molecules
- Build up to organisms gradually
- Don't try to build everything at once

**✅ Document Everything:**
- Use Storybook or similar tools
- Write clear prop documentation
- Include usage examples

**✅ Design Tokens:**
- Define colors, typography, spacing
- Use CSS variables or design token system
- Maintain consistency

**✅ Flexible Interpretation:**
- Adapt the methodology to your needs
- Don't be dogmatic about levels
- Focus on component reusability

**✅ Component Testing:**
- Test atoms in isolation
- Integration tests for organisms
- Visual regression testing

---

#### Interview Tips

✅ **Key Points:**
- 5 levels: atoms, molecules, organisms, templates, pages
- Inspired by chemistry - building blocks to complex structures
- Promotes component reusability and consistency
- Good for design systems and component libraries
- Works well with Storybook and design tokens

✅ **When to Mention:**
- Component architecture discussions
- Design system development
- Code organization strategies
- Reusability and maintainability
- Collaboration between design and development

✅ **Common Follow-ups:**
- "What's the difference between molecules and organisms?"
- "How do you decide component granularity?"
- "When would you NOT use Atomic Design?"
- "How does it integrate with feature-based architecture?"
- "What tools complement Atomic Design?"

✅ **Perfect Answer Structure:**
1. Define: 5-level methodology from atoms to pages
2. Hierarchy: Atoms → Molecules → Organisms → Templates → Pages
3. Benefits: Reusability, consistency, scalability, collaboration
4. Tools: Storybook, design tokens, component libraries
5. When to use: Design systems, large teams, consistent UI
6. Flexibility: Adapt to your needs, don't be dogmatic

</details>

---

### 169. How do you implement feature flags in React?

<details>
<summary>View Answer</summary>

**Feature flags** (also called feature toggles) allow you to enable or disable features in your React application without deploying new code. They're essential for A/B testing, gradual rollouts, canary releases, and controlling feature access.

#### Why Use Feature Flags?

**Benefits:**
- ✅ Deploy code before features are ready
- ✅ A/B testing and experimentation
- ✅ Gradual rollout to users
- ✅ Quick feature kill switch
- ✅ User/role-based access
- ✅ Environment-specific features
- ✅ Reduce merge conflicts

---

#### Basic Implementation

**1. Simple Boolean Flags:**
```typescript
// config/featureFlags.ts
export const featureFlags = {
  newDashboard: true,
  darkMode: false,
  betaFeatures: false,
  advancedSearch: true,
} as const;

export type FeatureFlag = keyof typeof featureFlags;
```

**2. Feature Flag Hook:**
```typescript
// hooks/useFeatureFlag.ts
import { featureFlags, FeatureFlag } from '@/config/featureFlags';

export function useFeatureFlag(flag: FeatureFlag): boolean {
  return featureFlags[flag];
}

// Usage
function Component() {
  const hasNewDashboard = useFeatureFlag('newDashboard');

  return (
    <div>
      {hasNewDashboard ? <NewDashboard /> : <OldDashboard />}
    </div>
  );
}
```

**3. Feature Flag Component:**
```tsx
// components/FeatureFlag.tsx
import { featureFlags, FeatureFlag } from '@/config/featureFlags';

interface FeatureFlagProps {
  flag: FeatureFlag;
  children: React.ReactNode;
  fallback?: React.ReactNode;
}

export function FeatureFlag({ flag, children, fallback = null }: FeatureFlagProps) {
  const isEnabled = featureFlags[flag];
  return isEnabled ? <>{children}</> : <>{fallback}</>;
}

// Usage
function App() {
  return (
    <div>
      <FeatureFlag flag="newDashboard">
        <NewDashboard />
      </FeatureFlag>

      <FeatureFlag flag="darkMode" fallback={<LightModeButton />}>
        <DarkModeButton />
      </FeatureFlag>
    </div>
  );
}
```

---

#### Environment-Based Feature Flags

**Configuration per Environment:**
```typescript
// config/featureFlags.ts
const env = process.env.NODE_ENV;

interface FeatureFlags {
  newDashboard: boolean;
  darkMode: boolean;
  analytics: boolean;
  debugTools: boolean;
}

const developmentFlags: FeatureFlags = {
  newDashboard: true,
  darkMode: true,
  analytics: false,
  debugTools: true,
};

const stagingFlags: FeatureFlags = {
  newDashboard: true,
  darkMode: true,
  analytics: true,
  debugTools: true,
};

const productionFlags: FeatureFlags = {
  newDashboard: false,
  darkMode: true,
  analytics: true,
  debugTools: false,
};

export const featureFlags: FeatureFlags = {
  development: developmentFlags,
  staging: stagingFlags,
  production: productionFlags,
}[env] || productionFlags;
```

---

#### Context-Based Feature Flags

**Feature Flag Context:**
```tsx
// contexts/FeatureFlagContext.tsx
import React, { createContext, useContext, useState, useEffect } from 'react';

interface FeatureFlags {
  [key: string]: boolean;
}

interface FeatureFlagContextType {
  flags: FeatureFlags;
  isEnabled: (flag: string) => boolean;
  enable: (flag: string) => void;
  disable: (flag: string) => void;
  toggle: (flag: string) => void;
}

const FeatureFlagContext = createContext<FeatureFlagContextType | undefined>(
  undefined
);

export function FeatureFlagProvider({ children }: { children: React.ReactNode }) {
  const [flags, setFlags] = useState<FeatureFlags>(() => {
    // Load from localStorage
    const stored = localStorage.getItem('featureFlags');
    return stored ? JSON.parse(stored) : {};
  });

  useEffect(() => {
    // Persist to localStorage
    localStorage.setItem('featureFlags', JSON.stringify(flags));
  }, [flags]);

  const isEnabled = (flag: string): boolean => {
    return flags[flag] ?? false;
  };

  const enable = (flag: string) => {
    setFlags(prev => ({ ...prev, [flag]: true }));
  };

  const disable = (flag: string) => {
    setFlags(prev => ({ ...prev, [flag]: false }));
  };

  const toggle = (flag: string) => {
    setFlags(prev => ({ ...prev, [flag]: !prev[flag] }));
  };

  return (
    <FeatureFlagContext.Provider value={{ flags, isEnabled, enable, disable, toggle }}>
      {children}
    </FeatureFlagContext.Provider>
  );
}

export function useFeatureFlags() {
  const context = useContext(FeatureFlagContext);
  if (!context) {
    throw new Error('useFeatureFlags must be used within FeatureFlagProvider');
  }
  return context;
}

// Usage
function App() {
  return (
    <FeatureFlagProvider>
      <MyApp />
    </FeatureFlagProvider>
  );
}

function Component() {
  const { isEnabled, toggle } = useFeatureFlags();

  return (
    <div>
      {isEnabled('newDashboard') && <NewDashboard />}
      
      <button onClick={() => toggle('darkMode')}>
        Toggle Dark Mode
      </button>
    </div>
  );
}
```

---

#### Remote Feature Flags

**Fetch from Server:**
```typescript
// services/featureFlagService.ts
interface FeatureFlags {
  [key: string]: boolean;
}

class FeatureFlagService {
  private flags: FeatureFlags = {};
  private listeners: Set<() => void> = new Set();

  async initialize() {
    try {
      const response = await fetch('/api/feature-flags');
      this.flags = await response.json();
      this.notify();
    } catch (error) {
      console.error('Failed to load feature flags:', error);
      // Use default flags
      this.flags = this.getDefaultFlags();
    }
  }

  isEnabled(flag: string): boolean {
    return this.flags[flag] ?? false;
  }

  getFlags(): FeatureFlags {
    return { ...this.flags };
  }

  subscribe(listener: () => void) {
    this.listeners.add(listener);
    return () => this.listeners.delete(listener);
  }

  private notify() {
    this.listeners.forEach(listener => listener());
  }

  private getDefaultFlags(): FeatureFlags {
    return {
      newDashboard: false,
      darkMode: true,
      betaFeatures: false,
    };
  }
}

export const featureFlagService = new FeatureFlagService();
```

**React Integration:**
```tsx
// hooks/useFeatureFlags.ts
import { useState, useEffect } from 'react';
import { featureFlagService } from '@/services/featureFlagService';

export function useFeatureFlags() {
  const [flags, setFlags] = useState(featureFlagService.getFlags());

  useEffect(() => {
    const unsubscribe = featureFlagService.subscribe(() => {
      setFlags(featureFlagService.getFlags());
    });

    return unsubscribe;
  }, []);

  return {
    flags,
    isEnabled: (flag: string) => featureFlagService.isEnabled(flag),
  };
}

// Initialize on app load
// App.tsx
import { useEffect, useState } from 'react';
import { featureFlagService } from '@/services/featureFlagService';

function App() {
  const [flagsLoaded, setFlagsLoaded] = useState(false);

  useEffect(() => {
    featureFlagService.initialize().then(() => {
      setFlagsLoaded(true);
    });
  }, []);

  if (!flagsLoaded) {
    return <LoadingScreen />;
  }

  return <MyApp />;
}
```

---

#### User-Based Feature Flags

**Role-Based Access:**
```typescript
// services/featureFlagService.ts
interface User {
  id: string;
  email: string;
  role: 'admin' | 'user' | 'beta-tester';
}

interface FeatureFlagConfig {
  enabled: boolean;
  allowedRoles?: string[];
  allowedUsers?: string[];
  percentage?: number; // Rollout percentage
}

type FeatureFlags = {
  [key: string]: FeatureFlagConfig;
};

class FeatureFlagService {
  private flags: FeatureFlags = {};
  private user: User | null = null;

  setUser(user: User) {
    this.user = user;
  }

  async loadFlags() {
    const response = await fetch('/api/feature-flags');
    this.flags = await response.json();
  }

  isEnabled(flagName: string): boolean {
    const flag = this.flags[flagName];
    if (!flag || !flag.enabled) return false;

    // Check role-based access
    if (flag.allowedRoles && this.user) {
      if (!flag.allowedRoles.includes(this.user.role)) {
        return false;
      }
    }

    // Check user-based access
    if (flag.allowedUsers && this.user) {
      if (!flag.allowedUsers.includes(this.user.id)) {
        return false;
      }
    }

    // Check percentage rollout
    if (flag.percentage !== undefined && this.user) {
      const hash = this.hashUser(this.user.id, flagName);
      return hash < flag.percentage;
    }

    return true;
  }

  private hashUser(userId: string, flagName: string): number {
    // Simple hash for percentage rollout
    const str = `${userId}-${flagName}`;
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      hash = (hash << 5) - hash + str.charCodeAt(i);
      hash = hash & hash;
    }
    return Math.abs(hash % 100);
  }
}

export const featureFlagService = new FeatureFlagService();
```

**Usage:**
```tsx
function App() {
  const { user } = useAuth();
  const { isEnabled } = useFeatureFlags();

  useEffect(() => {
    if (user) {
      featureFlagService.setUser(user);
    }
  }, [user]);

  return (
    <div>
      {isEnabled('adminPanel') && <AdminPanel />}
      {isEnabled('betaFeatures') && <BetaFeatures />}
    </div>
  );
}
```

---

#### Third-Party Solutions

**1. LaunchDarkly Integration:**
```typescript
// LaunchDarkly setup
import * as LDClient from 'launchdarkly-js-client-sdk';

const client = LDClient.initialize(
  'your-client-side-id',
  {
    key: 'user-key',
    email: 'user@example.com',
  }
);

// React hook
import { useState, useEffect } from 'react';

export function useLDFlag(flagKey: string, defaultValue: boolean = false) {
  const [flagValue, setFlagValue] = useState(defaultValue);

  useEffect(() => {
    client.waitForInitialization().then(() => {
      setFlagValue(client.variation(flagKey, defaultValue));

      // Listen for changes
      client.on(`change:${flagKey}`, (newValue) => {
        setFlagValue(newValue);
      });
    });
  }, [flagKey, defaultValue]);

  return flagValue;
}

// Usage
function Component() {
  const showNewFeature = useLDFlag('new-feature', false);

  return (
    <div>
      {showNewFeature ? <NewFeature /> : <OldFeature />}
    </div>
  );
}
```

**2. Split.io Integration:**
```typescript
import { SplitFactory } from '@splitsoftware/splitio';

const factory = SplitFactory({
  core: {
    authorizationKey: 'your-key',
    key: 'user-id',
  },
});

const client = factory.client();

// React hook
export function useSplitTreatment(splitName: string) {
  const [treatment, setTreatment] = useState('control');

  useEffect(() => {
    client.on(client.Event.SDK_READY, () => {
      const result = client.getTreatment(splitName);
      setTreatment(result);
    });
  }, [splitName]);

  return treatment;
}

// Usage
function Component() {
  const treatment = useSplitTreatment('new-checkout-flow');

  if (treatment === 'on') {
    return <NewCheckout />;
  }

  return <OldCheckout />;
}
```

**3. Flagsmith Integration:**
```typescript
import flagsmith from 'flagsmith';

flagsmith.init({
  environmentID: 'your-environment-id',
  onChange: (oldFlags, params) => {
    // Flags changed
  },
});

// React hook
export function useFlagsmith(featureName: string) {
  const [enabled, setEnabled] = useState(false);
  const [value, setValue] = useState(null);

  useEffect(() => {
    flagsmith.getValue(featureName).then((val) => {
      setValue(val);
    });

    flagsmith.hasFeature(featureName).then((has) => {
      setEnabled(has);
    });
  }, [featureName]);

  return { enabled, value };
}
```

---

#### Advanced Patterns

**1. Gradual Rollout:**
```typescript
interface FeatureFlag {
  enabled: boolean;
  rolloutPercentage: number; // 0-100
  rolloutStartDate?: Date;
  rolloutEndDate?: Date;
}

function isFeatureEnabled(flag: FeatureFlag, userId: string): boolean {
  if (!flag.enabled) return false;

  // Check date range
  const now = new Date();
  if (flag.rolloutStartDate && now < flag.rolloutStartDate) {
    return false;
  }
  if (flag.rolloutEndDate && now > flag.rolloutEndDate) {
    return true; // Fully rolled out
  }

  // Check percentage
  const userHash = hashString(userId) % 100;
  return userHash < flag.rolloutPercentage;
}

function hashString(str: string): number {
  let hash = 0;
  for (let i = 0; i < str.length; i++) {
    const char = str.charCodeAt(i);
    hash = (hash << 5) - hash + char;
    hash = hash & hash;
  }
  return Math.abs(hash);
}
```

**2. A/B Testing:**
```typescript
type Variant = 'control' | 'variant-a' | 'variant-b';

interface ABTest {
  name: string;
  variants: {
    control: number; // percentage
    'variant-a': number;
    'variant-b': number;
  };
}

function getVariant(test: ABTest, userId: string): Variant {
  const hash = hashString(`${test.name}-${userId}`) % 100;

  if (hash < test.variants.control) {
    return 'control';
  } else if (hash < test.variants.control + test.variants['variant-a']) {
    return 'variant-a';
  } else {
    return 'variant-b';
  }
}

// Usage
function Component() {
  const { user } = useAuth();
  const variant = getVariant(
    {
      name: 'checkout-flow',
      variants: {
        control: 33,
        'variant-a': 33,
        'variant-b': 34,
      },
    },
    user.id
  );

  switch (variant) {
    case 'variant-a':
      return <CheckoutFlowA />;
    case 'variant-b':
      return <CheckoutFlowB />;
    default:
      return <CheckoutFlowControl />;
  }
}
```

**3. Feature Flag Analytics:**
```typescript
// Track feature flag usage
class FeatureFlagAnalytics {
  track(flagName: string, enabled: boolean, userId?: string) {
    // Send to analytics service
    analytics.track('Feature Flag Evaluated', {
      flag: flagName,
      enabled,
      userId,
      timestamp: new Date(),
    });
  }
}

const analytics = new FeatureFlagAnalytics();

// In your feature flag hook
export function useFeatureFlag(flagName: string) {
  const { user } = useAuth();
  const enabled = featureFlagService.isEnabled(flagName);

  useEffect(() => {
    analytics.track(flagName, enabled, user?.id);
  }, [flagName, enabled, user?.id]);

  return enabled;
}
```

---

#### Developer Tools

**Feature Flag Debug Panel:**
```tsx
// components/FeatureFlagDebugPanel.tsx
import { useState } from 'react';
import { useFeatureFlags } from '@/hooks/useFeatureFlags';

export function FeatureFlagDebugPanel() {
  const { flags, toggle } = useFeatureFlags();
  const [isOpen, setIsOpen] = useState(false);

  // Only show in development
  if (process.env.NODE_ENV !== 'development') {
    return null;
  }

  return (
    <div className="feature-flag-debug-panel">
      <button onClick={() => setIsOpen(!isOpen)}>
        🚩 Feature Flags
      </button>

      {isOpen && (
        <div className="panel">
          <h3>Feature Flags</h3>
          {Object.entries(flags).map(([key, enabled]) => (
            <div key={key} className="flag-item">
              <label>
                <input
                  type="checkbox"
                  checked={enabled}
                  onChange={() => toggle(key)}
                />
                {key}
              </label>
            </div>
          ))}
        </div>
      )}
    </div>
  );
}

// Add to App
function App() {
  return (
    <div>
      <MyApp />
      <FeatureFlagDebugPanel />
    </div>
  );
}
```

---

#### Testing with Feature Flags

**Mock Feature Flags in Tests:**
```typescript
// __tests__/Component.test.tsx
import { render, screen } from '@testing-library/react';
import { FeatureFlagProvider } from '@/contexts/FeatureFlagContext';
import Component from './Component';

function renderWithFlags(component: React.ReactNode, flags: Record<string, boolean>) {
  return render(
    <FeatureFlagProvider initialFlags={flags}>
      {component}
    </FeatureFlagProvider>
  );
}

describe('Component', () => {
  it('renders new feature when flag enabled', () => {
    renderWithFlags(<Component />, { newFeature: true });
    expect(screen.getByText('New Feature')).toBeInTheDocument();
  });

  it('renders old feature when flag disabled', () => {
    renderWithFlags(<Component />, { newFeature: false });
    expect(screen.getByText('Old Feature')).toBeInTheDocument();
  });
});
```

---

#### Best Practices

**✅ Clear Naming:**
```typescript
// ✅ Good - descriptive names
const flags = {
  enableNewCheckoutFlow: true,
  showBetaDashboard: false,
  allowBulkImport: true,
};

// ❌ Bad - vague names
const flags = {
  feature1: true,
  newStuff: false,
  test: true,
};
```

**✅ Default to Safe:**
```typescript
// Default to disabled for new features
const flags = {
  experimentalFeature: false, // Safe default
};
```

**✅ Clean Up Old Flags:**
```typescript
// Remove flags after full rollout
// Before
if (isEnabled('newDashboard')) {
  return <NewDashboard />;
}
return <OldDashboard />;

// After full rollout - remove flag
return <NewDashboard />;
```

**✅ Document Flags:**
```typescript
// Document purpose and owner
const flags = {
  /**
   * New dashboard redesign
   * Owner: UI Team
   * Target: Q1 2024 rollout
   * Jira: PROJ-123
   */
  newDashboard: false,
};
```

**✅ Monitor Usage:**
```typescript
// Track flag evaluations for analytics
useEffect(() => {
  if (isEnabled('newFeature')) {
    analytics.track('Feature Flag: newFeature', { enabled: true });
  }
}, []);
```

---

#### Interview Tips

✅ **Key Points:**
- Enable/disable features without deployment
- Multiple approaches: config files, context, remote services
- User-based, role-based, percentage rollouts
- A/B testing and experimentation
- Third-party solutions: LaunchDarkly, Split.io, Flagsmith
- Clean up after full rollout

✅ **When to Mention:**
- Deployment strategies
- A/B testing
- Gradual feature rollout
- User segmentation
- Risk mitigation
- Trunk-based development

✅ **Common Follow-ups:**
- "How do you test with feature flags?"
- "What's the difference between feature flags and environment variables?"
- "How do you handle feature flag cleanup?"
- "What are the downsides of feature flags?"
- "How do you implement percentage rollouts?"

✅ **Perfect Answer Structure:**
1. Define: Toggle features without deployment
2. Basic: Boolean flags in config
3. Advanced: Context-based, remote flags
4. User-based: Role, percentage, A/B testing
5. Tools: LaunchDarkly, Split.io, Flagsmith
6. Best practices: Clear naming, cleanup, monitoring
7. Use cases: Gradual rollout, A/B testing, kill switch

</details>

---

### 170. What is Progressive Enhancement in React apps?

<details>
<summary>View Answer</summary>

**Progressive Enhancement** is a web development strategy where you start with a basic, functional experience that works for everyone, then layer on enhanced features for browsers and devices that support them. In React, this means ensuring your app works without JavaScript, then adding interactive features progressively.

#### Core Principles

1. **Content First** - HTML provides the structure and content
2. **Semantic Markup** - Use proper HTML elements
3. **CSS Enhancement** - Style progressively
4. **JavaScript Enhancement** - Add interactivity as an enhancement
5. **Graceful Degradation** - Falls back to basic functionality

---

#### Progressive Enhancement vs Graceful Degradation

**Progressive Enhancement (Bottom-up):**
```
Basic HTML (works for everyone)
  ↓
+ CSS (enhanced visuals)
  ↓
+ JavaScript (interactivity)
  ↓
+ Advanced features (cutting-edge browsers)
```

**Graceful Degradation (Top-down):**
```
Full-featured app
  ↓
Remove features for older browsers
  ↓
Fallbacks for missing capabilities
```

---

#### Server-Side Rendering (SSR)

**Next.js Example:**
```tsx
// pages/products/[id].tsx
import { GetServerSideProps } from 'next';

interface Product {
  id: string;
  name: string;
  price: number;
  description: string;
}

interface Props {
  product: Product;
}

// Works without JavaScript - HTML is rendered on server
export default function ProductPage({ product }: Props) {
  return (
    <div>
      <h1>{product.name}</h1>
      <p className="price">${product.price}</p>
      <p>{product.description}</p>
      
      {/* Basic form works without JS */}
      <form action="/api/cart" method="POST">
        <input type="hidden" name="productId" value={product.id} />
        <button type="submit">Add to Cart</button>
      </form>
    </div>
  );
}

// Server-side data fetching
export const getServerSideProps: GetServerSideProps<Props> = async (context) => {
  const { id } = context.params!;
  const product = await fetchProduct(id as string);
  
  return {
    props: { product },
  };
};
```

**Enhanced with Client-Side JavaScript:**
```tsx
import { useState } from 'react';
import { useRouter } from 'next/router';

export default function ProductPage({ product }: Props) {
  const router = useRouter();
  const [adding, setAdding] = useState(false);
  const [added, setAdded] = useState(false);

  const handleAddToCart = async (e: React.FormEvent) => {
    e.preventDefault(); // Enhance with JS
    
    setAdding(true);
    try {
      await fetch('/api/cart', {
        method: 'POST',
        body: JSON.stringify({ productId: product.id }),
        headers: { 'Content-Type': 'application/json' },
      });
      
      setAdded(true);
      
      // Show toast notification (JS enhancement)
      toast.success('Added to cart!');
    } catch (error) {
      // If JS fails, form still submits normally
      console.error(error);
    } finally {
      setAdding(false);
    }
  };

  return (
    <div>
      <h1>{product.name}</h1>
      <p className="price">${product.price}</p>
      <p>{product.description}</p>
      
      {/* Form works with or without JS */}
      <form action="/api/cart" method="POST" onSubmit={handleAddToCart}>
        <input type="hidden" name="productId" value={product.id} />
        <button type="submit" disabled={adding}>
          {adding ? 'Adding...' : added ? 'Added!' : 'Add to Cart'}
        </button>
      </form>
    </div>
  );
}
```

---

#### Feature Detection

**Check Browser Capabilities:**
```typescript
// utils/featureDetection.ts
export const features = {
  // Check if JavaScript is enabled
  hasJS: typeof window !== 'undefined',
  
  // Check for modern features
  hasIntersectionObserver: 
    typeof window !== 'undefined' && 'IntersectionObserver' in window,
  
  hasServiceWorker:
    typeof window !== 'undefined' && 'serviceWorker' in navigator,
  
  hasLocalStorage: (() => {
    try {
      return typeof window !== 'undefined' && 'localStorage' in window;
    } catch {
      return false;
    }
  })(),
  
  hasWebGL: (() => {
    try {
      const canvas = document.createElement('canvas');
      return !!(
        canvas.getContext('webgl') || 
        canvas.getContext('experimental-webgl')
      );
    } catch {
      return false;
    }
  })(),
  
  hasTouchScreen:
    typeof window !== 'undefined' &&
    ('ontouchstart' in window || navigator.maxTouchPoints > 0),
};
```

**Conditional Features:**
```tsx
import { features } from '@/utils/featureDetection';

function ProductGallery({ images }: { images: string[] }) {
  // Basic: Static images (works without JS)
  if (!features.hasJS) {
    return (
      <div className="gallery">
        {images.map((img, i) => (
          <img key={i} src={img} alt={`Product ${i + 1}`} loading="lazy" />
        ))}
      </div>
    );
  }
  
  // Enhanced: Interactive gallery with JS
  return <InteractiveGallery images={images} />;
}

function InteractiveGallery({ images }: { images: string[] }) {
  const [current, setCurrent] = useState(0);
  
  // Further enhancement: lazy loading with IntersectionObserver
  if (features.hasIntersectionObserver) {
    return <LazyLoadGallery images={images} />;
  }
  
  // Fallback: Simple carousel
  return (
    <div className="carousel">
      <img src={images[current]} alt="Product" />
      <button onClick={() => setCurrent((c) => Math.max(0, c - 1))}>
        Previous
      </button>
      <button onClick={() => setCurrent((c) => Math.min(images.length - 1, c + 1))}>
        Next
      </button>
    </div>
  );
}
```

---

#### Semantic HTML

**Proper HTML Structure:**
```tsx
// ✅ Good - Semantic HTML works without JS
function Navigation() {
  return (
    <nav aria-label="Main navigation">
      <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/products">Products</a></li>
        <li><a href="/about">About</a></li>
      </ul>
    </nav>
  );
}

// ❌ Bad - Breaks without JS
function Navigation() {
  const navigate = useNavigate();
  
  return (
    <div>
      <span onClick={() => navigate('/')}>Home</span>
      <span onClick={() => navigate('/products')}>Products</span>
      <span onClick={() => navigate('/about')}>About</span>
    </div>
  );
}
```

**Enhanced Navigation:**
```tsx
import { Link } from 'react-router-dom';

function Navigation() {
  // Uses proper <a> tags, enhanced with React Router
  return (
    <nav aria-label="Main navigation">
      <ul>
        <li><Link to="/">Home</Link></li>
        <li><Link to="/products">Products</Link></li>
        <li><Link to="/about">About</Link></li>
      </ul>
    </nav>
  );
}
```

---

#### Forms with Progressive Enhancement

**Basic Form (Works Without JS):**
```tsx
function ContactForm() {
  return (
    <form action="/api/contact" method="POST">
      <div>
        <label htmlFor="name">Name:</label>
        <input 
          type="text" 
          id="name" 
          name="name" 
          required 
          aria-required="true"
        />
      </div>
      
      <div>
        <label htmlFor="email">Email:</label>
        <input 
          type="email" 
          id="email" 
          name="email" 
          required
          aria-required="true"
        />
      </div>
      
      <div>
        <label htmlFor="message">Message:</label>
        <textarea 
          id="message" 
          name="message" 
          required
          aria-required="true"
        />
      </div>
      
      <button type="submit">Send Message</button>
    </form>
  );
}
```

**Enhanced with JavaScript:**
```tsx
import { useState } from 'react';

function ContactForm() {
  const [formState, setFormState] = useState({
    name: '',
    email: '',
    message: '',
  });
  const [status, setStatus] = useState<'idle' | 'submitting' | 'success' | 'error'>('idle');
  const [errors, setErrors] = useState<Record<string, string>>({});

  // Client-side validation (enhancement)
  const validate = () => {
    const newErrors: Record<string, string> = {};
    
    if (!formState.name.trim()) {
      newErrors.name = 'Name is required';
    }
    
    if (!formState.email.includes('@')) {
      newErrors.email = 'Invalid email';
    }
    
    if (formState.message.length < 10) {
      newErrors.message = 'Message must be at least 10 characters';
    }
    
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleSubmit = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault(); // Enhance with AJAX
    
    if (!validate()) return;
    
    setStatus('submitting');
    
    try {
      const formData = new FormData(e.currentTarget);
      
      const response = await fetch('/api/contact', {
        method: 'POST',
        body: formData,
      });
      
      if (response.ok) {
        setStatus('success');
        setFormState({ name: '', email: '', message: '' });
      } else {
        setStatus('error');
      }
    } catch (error) {
      // If AJAX fails, form can still be submitted normally
      setStatus('error');
    }
  };

  return (
    <form action="/api/contact" method="POST" onSubmit={handleSubmit}>
      <div>
        <label htmlFor="name">Name:</label>
        <input
          type="text"
          id="name"
          name="name"
          value={formState.name}
          onChange={(e) => setFormState(s => ({ ...s, name: e.target.value }))}
          required
          aria-required="true"
          aria-invalid={!!errors.name}
          aria-describedby={errors.name ? 'name-error' : undefined}
        />
        {errors.name && (
          <span id="name-error" role="alert" className="error">
            {errors.name}
          </span>
        )}
      </div>
      
      <div>
        <label htmlFor="email">Email:</label>
        <input
          type="email"
          id="email"
          name="email"
          value={formState.email}
          onChange={(e) => setFormState(s => ({ ...s, email: e.target.value }))}
          required
          aria-required="true"
          aria-invalid={!!errors.email}
        />
        {errors.email && (
          <span role="alert" className="error">{errors.email}</span>
        )}
      </div>
      
      <div>
        <label htmlFor="message">Message:</label>
        <textarea
          id="message"
          name="message"
          value={formState.message}
          onChange={(e) => setFormState(s => ({ ...s, message: e.target.value }))}
          required
          aria-required="true"
        />
        {errors.message && (
          <span role="alert" className="error">{errors.message}</span>
        )}
      </div>
      
      <button type="submit" disabled={status === 'submitting'}>
        {status === 'submitting' ? 'Sending...' : 'Send Message'}
      </button>
      
      {status === 'success' && (
        <div role="status" className="success">
          Message sent successfully!
        </div>
      )}
      
      {status === 'error' && (
        <div role="alert" className="error">
          Failed to send message. Please try again.
        </div>
      )}
    </form>
  );
}
```

---

#### CSS Progressive Enhancement

**Feature Queries:**
```css
/* Base styles - work everywhere */
.grid {
  display: block;
}

.grid-item {
  margin-bottom: 1rem;
}

/* Enhanced with Flexbox */
@supports (display: flex) {
  .grid {
    display: flex;
    flex-wrap: wrap;
    gap: 1rem;
  }
  
  .grid-item {
    flex: 1 1 300px;
    margin-bottom: 0;
  }
}

/* Further enhanced with Grid */
@supports (display: grid) {
  .grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 1rem;
  }
  
  .grid-item {
    flex: initial;
  }
}
```

---

#### Images with Progressive Enhancement

**Responsive Images:**
```tsx
function ProductImage({ product }: { product: Product }) {
  return (
    <picture>
      {/* WebP for modern browsers */}
      <source
        srcSet={`
          ${product.image.webp.small} 400w,
          ${product.image.webp.medium} 800w,
          ${product.image.webp.large} 1200w
        `}
        sizes="(max-width: 640px) 400px, (max-width: 1024px) 800px, 1200px"
        type="image/webp"
      />
      
      {/* AVIF for cutting-edge browsers */}
      <source
        srcSet={`
          ${product.image.avif.small} 400w,
          ${product.image.avif.medium} 800w,
          ${product.image.avif.large} 1200w
        `}
        sizes="(max-width: 640px) 400px, (max-width: 1024px) 800px, 1200px"
        type="image/avif"
      />
      
      {/* Fallback JPEG for all browsers */}
      <img
        src={product.image.jpeg.medium}
        srcSet={`
          ${product.image.jpeg.small} 400w,
          ${product.image.jpeg.medium} 800w,
          ${product.image.jpeg.large} 1200w
        `}
        sizes="(max-width: 640px) 400px, (max-width: 1024px) 800px, 1200px"
        alt={product.name}
        loading="lazy"
        decoding="async"
      />
    </picture>
  );
}
```

**Lazy Loading:**
```tsx
import { useEffect, useRef, useState } from 'react';

function LazyImage({ src, alt }: { src: string; alt: string }) {
  const [isLoaded, setIsLoaded] = useState(false);
  const imgRef = useRef<HTMLImageElement>(null);

  useEffect(() => {
    // Enhancement: Use IntersectionObserver
    if (!('IntersectionObserver' in window)) {
      // Fallback: Load immediately
      setIsLoaded(true);
      return;
    }

    const observer = new IntersectionObserver(
      (entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            setIsLoaded(true);
            observer.disconnect();
          }
        });
      },
      { rootMargin: '100px' }
    );

    if (imgRef.current) {
      observer.observe(imgRef.current);
    }

    return () => observer.disconnect();
  }, []);

  return (
    <img
      ref={imgRef}
      src={isLoaded ? src : undefined}
      alt={alt}
      loading="lazy" // Native lazy loading as fallback
      style={{ minHeight: '200px', backgroundColor: '#f0f0f0' }}
    />
  );
}
```

---

#### No-JavaScript Fallback

**Detect JavaScript Disabled:**
```html
<!-- In HTML head -->
<noscript>
  <style>
    .requires-js { display: none !important; }
    .no-js-message { display: block !important; }
  </style>
</noscript>
```

**React Component:**
```tsx
function App() {
  return (
    <div>
      {/* Show if JS disabled */}
      <noscript>
        <div className="no-js-message" role="alert">
          <p>This site works best with JavaScript enabled.</p>
          <p>Core functionality is still available without JavaScript.</p>
        </div>
      </noscript>
      
      {/* Main app */}
      <MainContent />
    </div>
  );
}
```

---

#### Best Practices

**✅ Start with HTML:**
```tsx
// Always use semantic HTML first
<button type="button" onClick={handleClick}>
  Click me
</button>

// Not
<div onClick={handleClick}>Click me</div>
```

**✅ Use Progressive Form Actions:**
```tsx
// Form works with POST even if JS fails
<form action="/api/submit" method="POST" onSubmit={handleSubmit}>
  {/* fields */}
</form>
```

**✅ Enhance Navigation:**
```tsx
// Start with <a> tags, enhance with client-side routing
<Link to="/products">Products</Link> // renders <a href="/products">
```

**✅ Provide Fallbacks:**
```tsx
function VideoPlayer({ src }: { src: string }) {
  return (
    <video controls>
      <source src={src} type="video/mp4" />
      {/* Fallback */}
      <p>
        Your browser doesn't support video.
        <a href={src}>Download the video</a>
      </p>
    </video>
  );
}
```

**✅ Test Without JavaScript:**
```bash
# Chrome DevTools: Disable JavaScript
# Settings > Debugger > Disable JavaScript

# Or use curl to test server rendering
curl http://localhost:3000
```

---

#### Interview Tips

✅ **Key Points:**
- Start with functional HTML/CSS, add JS as enhancement
- Server-side rendering ensures content loads without JS
- Use semantic HTML (proper elements, not divs)
- Forms work with native HTML submission, enhanced with AJAX
- Feature detection over browser detection
- Graceful fallbacks for unsupported features

✅ **When to Mention:**
- Performance optimization
- Accessibility discussions
- SEO considerations
- Server-side rendering (Next.js, Remix)
- Browser compatibility
- Resilient applications

✅ **Common Follow-ups:**
- "How does SSR help with progressive enhancement?"
- "What's the difference between progressive enhancement and graceful degradation?"
- "How do you test without JavaScript?"
- "Does progressive enhancement matter for SPAs?"
- "How do you handle forms progressively?"

✅ **Perfect Answer Structure:**
1. Define: Bottom-up approach, starting with basic functionality
2. Core: HTML content, CSS styling, JS enhancement
3. SSR: Server renders HTML, works without JS
4. Forms: Native submission, enhanced with AJAX
5. Feature detection: Check capabilities, provide fallbacks
6. Benefits: Performance, accessibility, SEO, resilience
7. Tools: Next.js, semantic HTML, feature queries

</details>
