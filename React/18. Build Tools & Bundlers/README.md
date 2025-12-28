# Build Tools & Bundlers

> Expert / Architect Level (5+ years)

---

## Questions

171. What is Vite and how does it improve DX?
172. What is the difference between Webpack and Vite?
173. What is Turbopack and its advantages?
174. How do you configure code splitting strategies?
175. What is ESBuild and why is it fast?
176. What are the benefits of SWC compiler?
177. How do you optimize build times for large projects?
178. What is Rspack and how does it compare to Webpack?
179. What is the role of Babel in modern React projects?
180. How do you implement custom Webpack plugins?

---

## Detailed Answers

### 171. What is Vite and how does it improve DX?

<details>
<summary>View Answer</summary>

**Vite** (French for "fast") is a modern build tool created by Evan You (creator of Vue.js) that significantly improves developer experience through instant server start, lightning-fast Hot Module Replacement (HMR), and optimized production builds using esbuild and Rollup.

#### Core Concepts

**How Vite Works:**

1. **Development:**
   - Uses native ES modules (ESM)
   - No bundling in development
   - Serves source files on-demand
   - Instant server start
   - Fast HMR

2. **Production:**
   - Pre-bundles with esbuild
   - Bundles with Rollup
   - Optimized output
   - Code splitting
   - Tree shaking

---

#### Why Vite is Fast

**1. No Bundling in Development:**
```
Traditional bundler (Webpack):
Entry → Bundle all modules → Dev server → Browser
(Takes 30s-5min for large apps)

Vite:
Entry → Dev server (instant) → Serve on-demand → Browser
(Ready in ~200ms)
```

**2. Native ES Modules:**
```javascript
// Your code
import React from 'react';
import { Button } from './components/Button';

// Browser receives (no bundling)
import React from '/node_modules/.vite/deps/react.js'; // Pre-bundled
import { Button } from '/src/components/Button.jsx'; // Source file
```

**3. Dependency Pre-Bundling:**
- Dependencies (node_modules) pre-bundled with esbuild
- Cached aggressively
- Only re-bundle when dependencies change

---

#### Setting Up Vite with React

**1. Create New Project:**
```bash
# Using npm
npm create vite@latest my-react-app -- --template react
cd my-react-app
npm install
npm run dev

# Using yarn
yarn create vite my-react-app --template react

# Using pnpm
pnpm create vite my-react-app --template react

# TypeScript template
npm create vite@latest my-react-app -- --template react-ts
```

**2. Project Structure:**
```
my-react-app/
├── public/
│   └── vite.svg
├── src/
│   ├── assets/
│   ├── App.jsx
│   ├── App.css
│   ├── main.jsx       // Entry point
│   └── index.css
├── index.html         // HTML entry
├── package.json
└── vite.config.js
```

**3. Entry Point (index.html):**
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Vite + React</title>
  </head>
  <body>
    <div id="root"></div>
    <!-- Direct ESM import -->
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
```

---

#### Vite Configuration

**Basic Configuration:**
```javascript
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
    open: true, // Auto-open browser
  },
  build: {
    outDir: 'dist',
    sourcemap: true,
  },
});
```

**Advanced Configuration:**
```javascript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [
    react({
      // Enable Fast Refresh
      fastRefresh: true,
      // Babel plugins
      babel: {
        plugins: ['babel-plugin-styled-components'],
      },
    }),
  ],
  
  // Path aliases
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@components': path.resolve(__dirname, './src/components'),
      '@utils': path.resolve(__dirname, './src/utils'),
    },
  },
  
  // Server configuration
  server: {
    port: 3000,
    host: true, // Listen on all addresses
    open: true,
    cors: true,
    // Proxy API requests
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ''),
      },
    },
  },
  
  // Build configuration
  build: {
    outDir: 'dist',
    assetsDir: 'assets',
    sourcemap: true,
    // Rollup options
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          router: ['react-router-dom'],
        },
      },
    },
    // Target modern browsers
    target: 'esnext',
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true,
        drop_debugger: true,
      },
    },
    // Chunk size warning
    chunkSizeWarningLimit: 1000,
  },
  
  // Environment variables
  envPrefix: 'VITE_',
  
  // CSS configuration
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: `@import "@/styles/variables.scss";`,
      },
    },
    modules: {
      localsConvention: 'camelCase',
    },
  },
  
  // Optimization
  optimizeDeps: {
    include: ['react', 'react-dom'],
    exclude: ['some-large-dep'],
  },
});
```

---

#### DX Improvements

**1. Instant Server Start:**
```bash
# Webpack (large app)
$ npm start
Compiling...
[====================] 100% (takes 2-5 minutes)
Compiled successfully!

# Vite
$ npm run dev
vite v5.0.0 dev server running at:
> Local: http://localhost:3000/
ready in 347ms. ✨
```

**2. Lightning-Fast HMR:**
```javascript
// When you edit a file
// Webpack: Re-bundles affected modules (~1-3s)
// Vite: Updates instantly (~50-100ms)

// Example: Edit Button.jsx
import { useState } from 'react';

export function Button({ children }) {
  const [count, setCount] = useState(0);
  
  return (
    <button onClick={() => setCount(count + 1)}>
      {children} - Clicked {count} times
      {/* Change text here - updates instantly */}
    </button>
  );
}

// Vite HMR API (automatic with React plugin)
if (import.meta.hot) {
  import.meta.hot.accept();
}
```

**3. Environment Variables:**
```javascript
// .env
VITE_API_URL=http://localhost:8080
VITE_APP_TITLE=My App

// Usage in code
const apiUrl = import.meta.env.VITE_API_URL;
const title = import.meta.env.VITE_APP_TITLE;

// TypeScript support
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_API_URL: string;
  readonly VITE_APP_TITLE: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}
```

**4. Static Asset Handling:**
```javascript
// Import assets
import logo from './assets/logo.png';
import styles from './App.module.css';

function App() {
  return (
    <div className={styles.app}>
      <img src={logo} alt="Logo" />
    </div>
  );
}

// Public directory (served as-is)
// public/favicon.ico → http://localhost:3000/favicon.ico
```

**5. CSS Modules:**
```css
/* Button.module.css */
.button {
  padding: 10px 20px;
  background: blue;
  color: white;
}

.button:hover {
  background: darkblue;
}
```

```javascript
// Button.jsx
import styles from './Button.module.css';

export function Button({ children }) {
  return (
    <button className={styles.button}>
      {children}
    </button>
  );
}
```

**6. TypeScript Support:**
```typescript
// tsconfig.json (automatic)
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true
  },
  "include": ["src"]
}
```

---

#### Popular Vite Plugins

**1. React Plugin:**
```javascript
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [
    react({
      // Fast Refresh
      fastRefresh: true,
      // Babel options
      babel: {
        plugins: ['babel-plugin-styled-components'],
      },
    }),
  ],
});
```

**2. PWA Plugin:**
```javascript
import { VitePWA } from 'vite-plugin-pwa';

export default defineConfig({
  plugins: [
    react(),
    VitePWA({
      registerType: 'autoUpdate',
      manifest: {
        name: 'My App',
        short_name: 'App',
        theme_color: '#ffffff',
        icons: [
          {
            src: 'icon-192x192.png',
            sizes: '192x192',
            type: 'image/png',
          },
        ],
      },
    }),
  ],
});
```

**3. SVG Plugin:**
```javascript
import svgr from 'vite-plugin-svgr';

export default defineConfig({
  plugins: [react(), svgr()],
});

// Usage
import { ReactComponent as Logo } from './logo.svg';

function App() {
  return <Logo />;
}
```

**4. Compression Plugin:**
```javascript
import viteCompression from 'vite-plugin-compression';

export default defineConfig({
  plugins: [
    react(),
    viteCompression({
      algorithm: 'gzip',
      ext: '.gz',
    }),
  ],
});
```

---

#### Migration from Webpack/CRA

**Steps:**

1. **Install Vite:**
```bash
npm install --save-dev vite @vitejs/plugin-react
```

2. **Create vite.config.js:**
```javascript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
});
```

3. **Update index.html:**
```html
<!-- Move index.html to project root -->
<!-- Remove %PUBLIC_URL% references -->
<!-- Add module script -->
<script type="module" src="/src/index.jsx"></script>
```

4. **Update package.json:**
```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  }
}
```

5. **Update Environment Variables:**
```javascript
// Before (CRA)
process.env.REACT_APP_API_URL

// After (Vite)
import.meta.env.VITE_API_URL
```

6. **Update Imports:**
```javascript
// Before (require)
const logo = require('./logo.png');

// After (ESM)
import logo from './logo.png';
```

---

#### Performance Comparison

**Development:**
```
Metric              | Webpack (CRA) | Vite
--------------------|---------------|-------
Cold Start          | 30-120s       | 0.3-1s
HMR                 | 1-3s          | 50-200ms
File Size           | 2-5 MB bundle | No bundle
Memory Usage        | 500MB-2GB     | 100-300MB
```

**Production Build:**
```
Metric              | Webpack       | Vite (Rollup)
--------------------|---------------|---------------
Build Time          | 60-180s       | 30-90s
Output Size         | Similar       | Similar
Tree Shaking        | Yes           | Yes
Code Splitting      | Yes           | Yes
```

---

#### Common Patterns

**1. Lazy Loading:**
```javascript
import { lazy, Suspense } from 'react';

const Dashboard = lazy(() => import('./pages/Dashboard'));
const Profile = lazy(() => import('./pages/Profile'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/profile" element={<Profile />} />
      </Routes>
    </Suspense>
  );
}
```

**2. Dynamic Imports:**
```javascript
const loadComponent = async (name) => {
  const module = await import(`./components/${name}.jsx`);
  return module.default;
};

// Vite handles this efficiently
```

**3. JSON Imports:**
```javascript
import data from './data.json';

// With named imports
import { users } from './data.json';
```

**4. WebAssembly:**
```javascript
import init, { add } from './math.wasm';

await init();
const result = add(1, 2);
```

---

#### Limitations

**❌ Limited Browser Support:**
- Development requires modern browsers (ES modules support)
- Use `@vitejs/plugin-legacy` for older browsers in production

**❌ Plugin Ecosystem:**
- Smaller than Webpack (growing rapidly)
- Some Webpack plugins don't have Vite equivalents

**❌ CommonJS Dependencies:**
- Some old npm packages may have issues
- Vite tries to convert them automatically

---

#### Best Practices

**✅ Use Environment Variables Properly:**
```javascript
// .env
VITE_API_URL=http://localhost:8080

// Usage
const apiUrl = import.meta.env.VITE_API_URL;
```

**✅ Optimize Dependencies:**
```javascript
export default defineConfig({
  optimizeDeps: {
    include: ['react', 'react-dom', 'react-router-dom'],
  },
});
```

**✅ Use Path Aliases:**
```javascript
resolve: {
  alias: {
    '@': path.resolve(__dirname, './src'),
  },
}

// Usage
import Button from '@/components/Button';
```

**✅ Configure Code Splitting:**
```javascript
build: {
  rollupOptions: {
    output: {
      manualChunks: {
        vendor: ['react', 'react-dom'],
        utils: ['lodash', 'date-fns'],
      },
    },
  },
}
```

---

#### Interview Tips

✅ **Key Points:**
- Native ES modules in development (no bundling)
- Instant server start and fast HMR
- esbuild for dependency pre-bundling
- Rollup for production builds
- Better DX than traditional bundlers
- Growing ecosystem and adoption

✅ **When to Mention:**
- Build tools comparison
- Developer experience improvements
- Modern frontend tooling
- Performance optimization
- Migration from CRA/Webpack

✅ **Common Follow-ups:**
- "How does Vite compare to Webpack?"
- "What are the limitations of Vite?"
- "How does HMR work in Vite?"
- "Can Vite be used in production?"
- "How to migrate from CRA to Vite?"

✅ **Perfect Answer Structure:**
1. Define: Modern build tool using native ES modules
2. Speed: No bundling in dev, instant start, fast HMR
3. How: esbuild for deps, native ESM for source, Rollup for prod
4. DX: Fast feedback, better ergonomics
5. Config: Simple, plugin-based
6. Migration: Easy from CRA/Webpack
7. Trade-offs: Modern browsers in dev, growing ecosystem

</details>

---

### 172. What is the difference between Webpack and Vite?

<details>
<summary>View Answer</summary>

**Webpack** is a powerful, mature bundler that bundles all modules before serving, while **Vite** leverages native ES modules to serve source files on-demand without bundling in development. This fundamental architectural difference makes Vite significantly faster in development while Webpack offers more configuration flexibility.

#### Core Architecture Differences

**Webpack:**
```
Development Flow:
Entry Point → Resolve Dependencies → Bundle All Modules → Dev Server → Browser
              [Can take 30s-5min]              [Hot]     [Fast]

Production Flow:
Entry Point → Bundle & Optimize → Output Files
```

**Vite:**
```
Development Flow:
Entry Point → Dev Server (instant) → Serve Files On-Demand → Browser
              [~200-500ms]          [No bundling]       [Native ESM]

Production Flow:
Entry Point → Rollup Bundling → Output Files
              [Similar to Webpack]
```

---

#### Side-by-Side Comparison

| Feature | Webpack | Vite |
|---------|---------|------|
| **Development Bundling** | Bundles everything | No bundling (native ESM) |
| **Cold Start** | 30s - 5min (large apps) | 200ms - 1s |
| **HMR Speed** | 1-3s | 50-200ms |
| **Dependencies** | Re-bundles often | Pre-bundled with esbuild |
| **Production** | Webpack bundling | Rollup bundling |
| **Config Complexity** | Complex | Simple |
| **Plugin Ecosystem** | Massive (1000+) | Growing (300+) |
| **Browser Support** | All browsers | Modern (dev), All (prod) |
| **Maturity** | 10+ years | 3+ years |
| **Learning Curve** | Steep | Gentle |
| **Use Cases** | Universal | Modern projects |

---

#### Development Experience

**Webpack Configuration:**
```javascript
// webpack.config.js (complex)
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
  mode: 'development',
  entry: './src/index.jsx',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash].js',
  },
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-react', '@babel/preset-env'],
          },
        },
      },
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader'],
      },
      {
        test: /\.(png|svg|jpg|jpeg|gif)$/i,
        type: 'asset/resource',
      },
    ],
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html',
    }),
    new MiniCssExtractPlugin(),
  ],
  resolve: {
    extensions: ['.js', '.jsx'],
    alias: {
      '@': path.resolve(__dirname, 'src'),
    },
  },
  devServer: {
    port: 3000,
    hot: true,
    historyApiFallback: true,
  },
  optimization: {
    splitChunks: {
      chunks: 'all',
    },
  },
};
```

**Vite Configuration:**
```javascript
// vite.config.js (simple)
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src'),
    },
  },
  server: {
    port: 3000,
  },
});

// That's it! CSS, images, etc. work out of the box
```

---

#### How They Handle Dependencies

**Webpack:**
```javascript
// Every import triggers bundling
import React from 'react'; // Included in bundle
import { Button } from './components/Button'; // Included in bundle

// Result: One large bundle containing everything
// main.bundle.js (2-5 MB in development)
```

**Vite:**
```javascript
// Dependencies pre-bundled with esbuild
import React from 'react'; // Served from .vite/deps/react.js

// Source files served as-is
import { Button } from './components/Button'; // Served from /src/components/Button.jsx

// Result: No bundling, native ES module imports
// Browser requests files on-demand
```

---

#### Hot Module Replacement (HMR)

**Webpack HMR:**
```javascript
// Edit Button.jsx
export function Button() {
  return <button>Click me</button>; // Change text
}

// Webpack process:
// 1. Detect change
// 2. Re-bundle affected modules
// 3. Send update to browser (1-3 seconds)
// 4. Browser applies patch

// webpack.config.js
module.exports = {
  devServer: {
    hot: true, // Enable HMR
  },
};
```

**Vite HMR:**
```javascript
// Edit Button.jsx
export function Button() {
  return <button>Click me</button>; // Change text
}

// Vite process:
// 1. Detect change
// 2. Send update directly (50-200ms)
// 3. Browser re-imports only changed module

// HMR API (automatic with React plugin)
if (import.meta.hot) {
  import.meta.hot.accept();
}
```

---

#### Bundle Size & Performance

**Development Bundle Size:**

**Webpack:**
```
Small App:  main.bundle.js (500KB - 2MB)
Medium App: main.bundle.js (2-5MB)
Large App:  main.bundle.js (5-20MB)

+ Slower as app grows
+ Memory intensive
+ Longer initial load
```

**Vite:**
```
Any Size App: No bundle in development!

+ Files served individually
+ Browser caches effectively
+ Only loads what's needed
```

**Production Bundle (Similar):**
```
Both produce similar production bundles:
- Code splitting
- Tree shaking
- Minification
- Optimized output
```

---

#### Real-World Example

**Large Dashboard Application:**

**With Webpack:**
```bash
$ npm start
ℹ ️ Compiling...
[                    ] 0%  Starting compilation
[====                ] 20% Building modules
[========            ] 40% Sealing
[============        ] 60% Optimizing
[================    ] 80% Emitting
[====================] 100% Done

✓ Compiled successfully in 182.43s

webpack compiled with 1 warning
```

**With Vite:**
```bash
$ npm run dev

  VITE v5.0.8  ready in 347 ms

  ➜  Local:   http://localhost:3000/
  ➜  Network: use --host to expose
  ➜  press h to show help
```

---

#### Code Splitting

**Webpack:**
```javascript
// webpack.config.js
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          priority: 10,
        },
        common: {
          minChunks: 2,
          priority: 5,
          reuseExistingChunk: true,
        },
      },
    },
  },
};

// Dynamic imports
const Dashboard = lazy(() => import('./Dashboard'));
```

**Vite:**
```javascript
// vite.config.js
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          router: ['react-router-dom'],
        },
      },
    },
  },
});

// Dynamic imports work the same
const Dashboard = lazy(() => import('./Dashboard'));
```

---

#### CSS Handling

**Webpack:**
```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
      {
        test: /\.scss$/,
        use: ['style-loader', 'css-loader', 'sass-loader'],
      },
      {
        test: /\.module\.css$/,
        use: [
          'style-loader',
          {
            loader: 'css-loader',
            options: { modules: true },
          },
        ],
      },
    ],
  },
};
```

**Vite:**
```javascript
// vite.config.js - CSS works out of the box!
export default defineConfig({
  plugins: [react()],
  // Optional CSS configuration
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: `@import "@/styles/variables.scss";`,
      },
    },
    modules: {
      localsConvention: 'camelCase',
    },
  },
});

// Usage - same in both
import './App.css'; // Global CSS
import styles from './App.module.css'; // CSS Modules
import './App.scss'; // SCSS
```

---

#### Asset Handling

**Webpack:**
```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|svg|jpg|jpeg|gif)$/i,
        type: 'asset/resource',
      },
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/i,
        type: 'asset/resource',
      },
    ],
  },
};

// Usage
import logo from './logo.png';
<img src={logo} alt="Logo" />;
```

**Vite:**
```javascript
// No configuration needed!

// Usage - same as Webpack
import logo from './logo.png';
<img src={logo} alt="Logo" />;

// Additional Vite features
import logoUrl from './logo.png?url'; // Get URL
import logoRaw from './logo.svg?raw'; // Get raw content
```

---

#### Environment Variables

**Webpack (Create React App):**
```javascript
// .env
REACT_APP_API_URL=http://localhost:8080
REACT_APP_TITLE=My App

// Usage
const apiUrl = process.env.REACT_APP_API_URL;
const title = process.env.REACT_APP_TITLE;
```

**Vite:**
```javascript
// .env
VITE_API_URL=http://localhost:8080
VITE_TITLE=My App

// Usage
const apiUrl = import.meta.env.VITE_API_URL;
const title = import.meta.env.VITE_TITLE;

// Mode-specific files
// .env.development
// .env.production
```

---

#### Plugin Ecosystem

**Webpack Plugins:**
```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const CopyWebpackPlugin = require('copy-webpack-plugin');
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');

module.exports = {
  plugins: [
    new HtmlWebpackPlugin({ template: './public/index.html' }),
    new MiniCssExtractPlugin({ filename: '[name].[contenthash].css' }),
    new CopyWebpackPlugin({ patterns: [{ from: 'public', to: 'dist' }] }),
    new BundleAnalyzerPlugin(),
  ],
};
```

**Vite Plugins:**
```javascript
import react from '@vitejs/plugin-react';
import { VitePWA } from 'vite-plugin-pwa';
import viteCompression from 'vite-plugin-compression';
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    react(),
    VitePWA({ registerType: 'autoUpdate' }),
    viteCompression(),
    visualizer(),
  ],
});
```

---

#### When to Use Each

**Use Webpack When:**

✅ **Complex Build Requirements:**
- Need precise control over bundling
- Custom loaders and transformations
- Legacy browser support is critical

✅ **Existing Projects:**
- Large existing codebase
- Migration cost too high
- Team expertise in Webpack

✅ **Specific Features:**
- Module Federation
- Advanced code splitting strategies
- Specific Webpack-only plugins needed

**Use Vite When:**

✅ **New Projects:**
- Starting fresh
- Want fast development experience
- Modern tooling

✅ **Developer Experience Priority:**
- Fast feedback loop crucial
- Large development team
- Frequent code changes

✅ **Modern Stack:**
- Target modern browsers
- Use ES modules
- TypeScript/JSX

---

#### Migration Path

**From Webpack to Vite:**

```javascript
// 1. Install Vite
npm install --save-dev vite @vitejs/plugin-react

// 2. Create vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
});

// 3. Move index.html to root
// 4. Update scripts in package.json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  }
}

// 5. Update environment variables
// REACT_APP_* → VITE_*
// process.env.* → import.meta.env.*

// 6. Update imports
// require() → import
```

---

#### Performance Metrics (Real Project)

**Medium-Sized React App (50 components, 100 files):**

```
Metric                  | Webpack    | Vite       | Improvement
------------------------|------------|------------|-------------
Cold Start              | 45s        | 0.5s       | 90x faster
HMR (single file)       | 2.1s       | 0.08s      | 26x faster
Full Reload             | 43s        | 2.1s       | 20x faster
Memory Usage (dev)      | 1.2 GB     | 280 MB     | 4x less
Build Time (prod)       | 68s        | 52s        | 23% faster
Bundle Size (prod)      | 342 KB     | 336 KB     | Similar
```

---

#### Common Gotchas

**Webpack → Vite Migration:**

```javascript
// 1. CommonJS imports
// ❌ Webpack (works)
const React = require('react');

// ✅ Vite (ESM only)
import React from 'react';

// 2. Dynamic require
// ❌ Webpack
const Component = require(`./components/${name}`);

// ✅ Vite
const Component = await import(`./components/${name}.jsx`);

// 3. Environment variables
// ❌ Webpack
process.env.REACT_APP_API_URL

// ✅ Vite
import.meta.env.VITE_API_URL

// 4. Public assets
// ❌ Webpack
<img src="%PUBLIC_URL%/logo.png" />

// ✅ Vite
<img src="/logo.png" /> // Files in public/
```

---

#### Interview Tips

✅ **Key Points:**
- Webpack bundles everything; Vite uses native ESM in dev
- Vite is 10-100x faster in development
- Webpack has more mature ecosystem
- Both produce similar production bundles
- Vite has simpler configuration
- Choose based on project needs

✅ **When to Mention:**
- Build tool comparisons
- Performance optimization
- Developer experience
- Modern tooling choices
- Project setup decisions

✅ **Common Follow-ups:**
- "Why is Vite faster than Webpack?"
- "Can Vite replace Webpack entirely?"
- "What are the trade-offs?"
- "How do you migrate from Webpack to Vite?"
- "Which should I use for a new project?"

✅ **Perfect Answer Structure:**
1. Core difference: Bundling vs native ESM
2. Speed: Vite 10-100x faster in dev
3. Config: Vite simpler, Webpack more flexible
4. Ecosystem: Webpack larger, Vite growing
5. Use cases: New projects → Vite, Complex needs → Webpack
6. Production: Both produce similar outputs
7. Migration: Straightforward for most projects

</details>

---

### 173. What is Turbopack and its advantages?

<details>
<summary>View Answer</summary>

**Turbopack** is a Rust-based incremental bundler created by Vercel (the team behind Next.js and Webpack) as the successor to Webpack. It's designed to be extremely fast through incremental computation, native code execution, and optimized caching.

#### What is Turbopack?

**Key Characteristics:**
- Written in Rust (not JavaScript)
- Incremental bundler (only rebuilds what changed)
- Built-in HMR (Hot Module Replacement)
- Optimized for Next.js (but can be used standalone)
- Successor to Webpack
- Created by Tobias Koppers (Webpack creator)

**Architecture:**
```
Turbopack = Rust + Incremental Computation + Function-level Caching

Traditional Bundler:
File Change → Rebuild Everything → Update

Turbopack:
File Change → Rebuild Only Affected Functions → Update
```

---

#### Why Turbopack is Fast

**1. Written in Rust:**
```
JavaScript (Webpack/Vite):
- Interpreted language
- Garbage collection overhead
- Single-threaded (mostly)

Rust (Turbopack):
- Compiled to native code
- No GC overhead
- Efficient memory management
- Multi-threaded by default
```

**2. Incremental Computation:**
```
Traditional:
- Edit Button.jsx
- Re-process entire dependency tree
- Re-bundle affected modules

Turbopack:
- Edit Button.jsx
- Re-process only changed functions
- Cache everything else
- Update only affected chunks
```

**3. Function-Level Caching:**
```javascript
// Button.jsx
export function Button() {  // Function 1
  return <button>Click</button>;
}

export function IconButton() {  // Function 2
  return <button>🔘</button>;
}

// Edit Button() only
// Turbopack: Re-process Function 1, cache Function 2
// Webpack: Re-process entire file
```

**4. Lazy Bundling:**
```
Webpack:
- Bundles entire app on start
- Even code you don't visit

Turbopack:
- Bundles only requested routes/components
- On-demand compilation
- Faster initial load
```

---

#### Performance Benchmarks

**Claimed Performance (Vercel):**
```
Metric              | Webpack | Vite  | Turbopack
--------------------|---------|-------|----------
Cold Start (large)  | 30s     | 1.2s  | 0.7s
HMR Update          | 2s      | 0.08s | 0.01s
Code Changes        | Slow    | Fast  | Instant
```

**Real-World Example (1000 module app):**
```
Task                | Webpack 5 | Turbopack | Speedup
--------------------|-----------|-----------|--------
Initial Compile     | 16.5s     | 1.2s      | 13.7x
File Update (root)  | 2.3s      | 0.006s    | 383x
File Update (leaf)  | 1.1s      | 0.004s    | 275x
```

---

#### Using Turbopack with Next.js

**Next.js 13+ Integration:**

**1. Enable Turbopack:**
```json
// package.json
{
  "scripts": {
    "dev": "next dev --turbo",
    "build": "next build",
    "start": "next start"
  }
}
```

```bash
# Or run directly
npx next dev --turbo
```

**2. Project Structure:**
```
next-app/
├── app/
│   ├── page.tsx
│   ├── layout.tsx
│   └── api/
├── public/
├── next.config.js
└── package.json
```

**3. Configuration:**
```javascript
// next.config.js
module.exports = {
  // Turbopack is automatically used with --turbo flag
  // No special configuration needed
  
  // Standard Next.js config works
  reactStrictMode: true,
  images: {
    domains: ['example.com'],
  },
};
```

---

#### Features

**1. Hot Module Replacement (HMR):**
```javascript
// Automatic HMR - no configuration needed
// app/components/Button.tsx
export function Button({ children }: { children: React.ReactNode }) {
  return (
    <button className="btn">
      {children}
    </button>
  );
}

// Edit this file → Updates in ~10ms
// State is preserved
// No full reload
```

**2. CSS Support:**
```typescript
// app/page.tsx
import './page.css'; // Global CSS
import styles from './page.module.css'; // CSS Modules

export default function Page() {
  return (
    <div className={styles.container}>
      <h1>Hello Turbopack</h1>
    </div>
  );
}
```

**3. TypeScript Support:**
```typescript
// Native TypeScript support
// No babel or ts-loader needed
import type { User } from './types';

interface Props {
  user: User;
}

export default function Profile({ user }: Props) {
  return <div>{user.name}</div>;
}
```

**4. Image Optimization:**
```typescript
import Image from 'next/image';
import logo from './logo.png';

export default function Page() {
  return (
    <Image
      src={logo}
      alt="Logo"
      width={200}
      height={100}
      priority
    />
  );
}
```

**5. API Routes:**
```typescript
// app/api/users/route.ts
import { NextResponse } from 'next/server';

export async function GET() {
  const users = await fetchUsers();
  return NextResponse.json(users);
}

export async function POST(request: Request) {
  const body = await request.json();
  const user = await createUser(body);
  return NextResponse.json(user);
}
```

---

#### Advantages Over Webpack

**1. Speed:**
```
Scenario                    | Webpack | Turbopack
----------------------------|---------|----------
Large app cold start        | 30s     | <1s
Small change HMR            | 1-3s    | ~10ms
Full page refresh           | 20s     | <1s
Adding new route            | 15s     | <500ms
```

**2. Simplicity:**
```javascript
// Webpack - Complex configuration
module.exports = {
  module: {
    rules: [
      { test: /\.tsx?$/, use: 'ts-loader' },
      { test: /\.css$/, use: ['style-loader', 'css-loader'] },
      { test: /\.(png|jpg)$/, type: 'asset/resource' },
    ],
  },
  plugins: [/* many plugins */],
};

// Turbopack - Zero configuration
// Just works! No config needed for:
// - TypeScript
// - CSS/CSS Modules
// - Images
// - Fonts
```

**3. Memory Efficiency:**
```
App Size     | Webpack Memory | Turbopack Memory
-------------|----------------|------------------
Small        | 200-500 MB     | 50-100 MB
Medium       | 500-1500 MB    | 100-300 MB
Large        | 1.5-4 GB       | 300-800 MB
```

**4. Incremental Updates:**
```javascript
// Edit a component
// Webpack: Re-bundles entire dependency chain
// Turbopack: Re-compiles only the changed function

// Example:
function Button() {
  return <button>Click</button>; // Edit this line
}

// Webpack update: ~2000ms (large app)
// Turbopack update: ~10ms (any size app)
```

---

#### Advantages Over Vite

**1. Production Consistency:**
```
Vite:
- Dev: Native ESM (fast)
- Prod: Rollup bundling (different)
- Potential dev/prod inconsistencies

Turbopack:
- Dev: Turbopack bundling
- Prod: Turbopack bundling (same engine)
- Consistent behavior
```

**2. Large Apps:**
```
Vite:
- Fast for small/medium apps
- Slower as modules grow (1000+)
- Browser HTTP requests become bottleneck

Turbopack:
- Fast regardless of size
- Scales linearly
- Incremental computation
```

**3. HMR Reliability:**
```
Vite:
- Occasional HMR failures
- Full reload sometimes needed
- State loss on some updates

Turbopack:
- More reliable HMR
- Better state preservation
- Faster updates
```

---

#### Current Limitations

**❌ Still in Beta:**
```
- Not production-ready (as of 2024)
- Only works with Next.js dev mode
- Production builds use Webpack
- Some features missing
```

**❌ Limited Ecosystem:**
```
- Fewer plugins than Webpack
- Some Webpack plugins incompatible
- Growing but not mature
```

**❌ Next.js Only (Currently):**
```
- Tightly integrated with Next.js
- Not yet available as standalone bundler
- Can't use with Vite, CRA, etc.
```

**❌ Breaking Changes:**
```
- Beta software, API may change
- Upgrade path uncertain
- Documentation evolving
```

---

#### Architecture Deep Dive

**Incremental Computation Engine:**
```rust
// Pseudocode representation
struct TurboEngine {
    // Function-level cache
    cache: HashMap<FunctionId, CompiledOutput>,
    
    // Dependency graph
    graph: DependencyGraph,
}

impl TurboEngine {
    fn process_change(&mut self, file: File) {
        // Find affected functions
        let affected = self.graph.find_affected(file);
        
        // Recompile only affected functions
        for func in affected {
            let output = self.compile(func);
            self.cache.insert(func.id, output);
        }
        
        // Everything else uses cache
    }
}
```

**Comparison with Other Tools:**
```
Webpack:
- Module-level caching
- Rebuilds entire modules
- JavaScript-based

Vite:
- File-level serving
- No bundling in dev
- JavaScript-based

Turbopack:
- Function-level caching
- Rebuilds only changed functions
- Rust-based
```

---

#### Real-World Usage

**1. Development Workflow:**
```bash
# Start dev server with Turbopack
npm run dev -- --turbo

# Or with specific port
npm run dev -- --turbo -p 3001

# Console output
○ Turbopack (beta) started in 234ms
▲ Next.js 14.0.0
- Local:        http://localhost:3000
- Experiments:  turbo

✓ Ready in 234ms
```

**2. Making Changes:**
```typescript
// Edit any file
// app/components/Header.tsx
export function Header() {
  return (
    <header>
      <h1>My App</h1> {/* Change this */}
    </header>
  );
}

// Console:
✓ Compiled in 8ms (updating)
```

**3. Adding New Routes:**
```typescript
// Create new file: app/dashboard/page.tsx
export default function Dashboard() {
  return <div>Dashboard</div>;
}

// Turbopack automatically:
// - Detects new route
// - Compiles on-demand
// - Updates routing
// Time: ~100-200ms
```

---

#### Migration from Webpack

**For Next.js Apps:**
```javascript
// 1. Update Next.js to 13+
npm install next@latest

// 2. Update package.json
{
  "scripts": {
    "dev": "next dev --turbo",  // Add --turbo flag
    "build": "next build",
    "start": "next start"
  }
}

// 3. Remove Webpack config (if any)
// Delete next.config.js webpack customizations

// 4. Run dev server
npm run dev

// That's it! Most apps work without changes
```

**Compatibility:**
```javascript
// ✅ Works out of the box:
- TypeScript
- CSS Modules
- SASS/SCSS
- Images (jpg, png, svg)
- Fonts
- JSON imports
- API routes

// ⚠️ May need adjustments:
- Custom Webpack configs
- Webpack-specific plugins
- Module aliases (usually fine)

// ❌ Not supported (yet):
- Some advanced Webpack features
- Custom loaders
```

---

#### Future of Turbopack

**Roadmap:**
```
Phase 1 (Current): ✅
- Next.js dev mode
- Basic features
- Beta testing

Phase 2 (2024): 🔄
- Production builds
- More features
- Stability improvements

Phase 3 (Future): 📋
- Standalone bundler
- Framework agnostic
- Full Webpack replacement
```

**Goals:**
- Replace Webpack in Next.js completely
- Become general-purpose bundler
- Set new standard for build tools
- 10x faster than current tools

---

#### Best Practices

**✅ Use for Development:**
```bash
# Perfect for dev mode
npm run dev -- --turbo
```

**✅ Monitor Performance:**
```javascript
// Check compile times in console
// Should see:
✓ Compiled in 8ms
✓ Compiled in 12ms
```

**✅ Report Issues:**
```
- Still beta, bugs expected
- Report on Next.js GitHub
- Help improve the tool
```

**❌ Don't Use for Production (Yet):**
```
- Not stable enough
- Use standard Next.js build
- Wait for official release
```

---

#### Comparison Summary

| Feature | Webpack | Vite | Turbopack |
|---------|---------|------|----------|
| Language | JavaScript | JavaScript | Rust |
| Dev Speed | Slow | Fast | Fastest |
| Prod Speed | Standard | Standard | TBD |
| Maturity | Mature | Mature | Beta |
| Ecosystem | Huge | Growing | Small |
| Config | Complex | Simple | Zero |
| HMR | Good | Great | Excellent |
| Scale | All sizes | Best for small/medium | All sizes |
| Learning Curve | Steep | Easy | Easy |

---

#### Interview Tips

✅ **Key Points:**
- Rust-based bundler by Vercel (Webpack team)
- 10x+ faster than Webpack through incremental compilation
- Function-level caching, not module-level
- Currently Next.js only, beta stage
- Will replace Webpack in Next.js
- Not yet production-ready

✅ **When to Mention:**
- Build tool performance discussions
- Next.js ecosystem
- Modern tooling trends
- Rust in frontend tooling
- Future of bundlers

✅ **Common Follow-ups:**
- "How is Turbopack different from Webpack?"
- "Why is it faster?"
- "Can I use it outside Next.js?"
- "Is it production-ready?"
- "Will it replace Webpack?"

✅ **Perfect Answer Structure:**
1. Define: Rust-based incremental bundler
2. Speed: 10x+ faster through function-level caching
3. How: Incremental computation + Rust + lazy bundling
4. Status: Beta, Next.js dev mode only
5. Advantages: Speed, simplicity, consistency
6. Limitations: Beta, Next.js only, small ecosystem
7. Future: Will replace Webpack in Next.js, then standalone

</details>

---

### 174. How do you configure code splitting strategies?

<details>
<summary>View Answer</summary>

**Code splitting** is the practice of splitting your application bundle into smaller chunks that can be loaded on-demand or in parallel, improving initial load time and performance. Different bundlers (Webpack, Vite, Rollup) offer various configuration options for implementing code splitting strategies.

#### Why Code Splitting?

**Benefits:**
- ✅ Faster initial page load
- ✅ Better caching (unchanged chunks stay cached)
- ✅ Load code only when needed
- ✅ Parallel downloads
- ✅ Smaller bundle sizes

**Without Code Splitting:**
```
Bundle: main.js (2.5 MB)
- React + React DOM (150 KB)
- All routes (1.5 MB)
- All libraries (800 KB)
- All components (50 KB)

Result: User downloads 2.5 MB before seeing anything
```

**With Code Splitting:**
```
Chunks:
- vendor.js (150 KB) - React, React DOM
- main.js (50 KB) - App shell
- home.js (100 KB) - Home page (loaded immediately)
- dashboard.js (500 KB) - Dashboard (loaded when visited)
- admin.js (200 KB) - Admin panel (loaded when visited)

Result: User downloads 300 KB initially, rest on-demand
```

---

#### React.lazy() and Suspense (Built-in)

**Basic Route-Based Splitting:**
```typescript
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

// Lazy load route components
const Home = lazy(() => import('./pages/Home'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Profile = lazy(() => import('./pages/Profile'));
const Admin = lazy(() => import('./pages/Admin'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<div>Loading...</div>}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/profile" element={<Profile />} />
          <Route path="/admin" element={<Admin />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

**Component-Level Splitting:**
```typescript
import { lazy, Suspense } from 'react';

// Heavy components lazy loaded
const HeavyChart = lazy(() => import('./components/HeavyChart'));
const VideoPlayer = lazy(() => import('./components/VideoPlayer'));
const ImageEditor = lazy(() => import('./components/ImageEditor'));

function Dashboard() {
  const [showChart, setShowChart] = useState(false);

  return (
    <div>
      <h1>Dashboard</h1>
      
      <button onClick={() => setShowChart(true)}>
        Show Chart
      </button>

      {showChart && (
        <Suspense fallback={<div>Loading chart...</div>}>
          <HeavyChart />
        </Suspense>
      )}
    </div>
  );
}
```

**Multiple Suspense Boundaries:**
```typescript
function App() {
  return (
    <div>
      {/* Critical content - show skeleton */}
      <Suspense fallback={<HeaderSkeleton />}>
        <Header />
      </Suspense>

      {/* Main content */}
      <Suspense fallback={<MainSkeleton />}>
        <MainContent />
      </Suspense>

      {/* Sidebar - less critical */}
      <Suspense fallback={<SidebarSkeleton />}>
        <Sidebar />
      </Suspense>

      {/* Footer - lowest priority */}
      <Suspense fallback={null}>
        <Footer />
      </Suspense>
    </div>
  );
}
```

---

#### Webpack Configuration

**1. Basic Splitting:**
```javascript
// webpack.config.js
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all', // Split all chunks (async + initial)
    },
  },
};
```

**2. Vendor Chunk Strategy:**
```javascript
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        // Vendor libraries
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          priority: 10,
        },
        // React ecosystem
        react: {
          test: /[\\/]node_modules[\\/](react|react-dom|react-router-dom)[\\/]/,
          name: 'react-vendor',
          priority: 20,
        },
        // UI libraries
        ui: {
          test: /[\\/]node_modules[\\/](@mui|@emotion)[\\/]/,
          name: 'ui-vendor',
          priority: 15,
        },
        // Common code
        common: {
          minChunks: 2,
          name: 'common',
          priority: 5,
          reuseExistingChunk: true,
        },
      },
    },
  },
};
```

**3. Advanced Splitting:**
```javascript
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      minSize: 20000, // Minimum chunk size (bytes)
      minRemainingSize: 0,
      minChunks: 1, // Minimum times shared before splitting
      maxAsyncRequests: 30, // Max parallel requests
      maxInitialRequests: 30,
      enforceSizeThreshold: 50000,
      cacheGroups: {
        defaultVendors: {
          test: /[\\/]node_modules[\\/]/,
          priority: -10,
          reuseExistingChunk: true,
          name(module, chunks, cacheGroupKey) {
            // Generate name based on module
            const moduleFileName = module
              .identifier()
              .split('/')
              .reduceRight((item) => item);
            return `${cacheGroupKey}-${moduleFileName}`;
          },
        },
        default: {
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true,
        },
      },
    },
    // Runtime chunk for better caching
    runtimeChunk: 'single',
  },
};
```

**4. Magic Comments (Dynamic Imports):**
```javascript
// Webpack magic comments
const Dashboard = lazy(() =>
  import(
    /* webpackChunkName: "dashboard" */
    /* webpackPrefetch: true */
    './pages/Dashboard'
  )
);

const AdminPanel = lazy(() =>
  import(
    /* webpackChunkName: "admin" */
    /* webpackPreload: true */
    './pages/AdminPanel'
  )
);

// Load multiple modules together
const loadModules = () =>
  import(
    /* webpackChunkName: "analytics" */
    /* webpackMode: "lazy" */
    './analytics'
  );
```

---

#### Vite Configuration

**1. Manual Chunks:**
```javascript
// vite.config.js
import { defineConfig } from 'vite';

export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          // Vendor chunk
          vendor: ['react', 'react-dom'],
          // Router chunk
          router: ['react-router-dom'],
          // UI library chunk
          ui: ['@mui/material', '@emotion/react'],
          // Utils chunk
          utils: ['lodash', 'date-fns', 'axios'],
        },
      },
    },
  },
});
```

**2. Dynamic Manual Chunks:**
```javascript
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks(id) {
          // node_modules into vendor
          if (id.includes('node_modules')) {
            // Split large vendors
            if (id.includes('@mui')) {
              return 'vendor-mui';
            }
            if (id.includes('lodash')) {
              return 'vendor-lodash';
            }
            // Everything else in vendor
            return 'vendor';
          }
          
          // Feature-based splitting
          if (id.includes('/src/features/dashboard')) {
            return 'feature-dashboard';
          }
          if (id.includes('/src/features/admin')) {
            return 'feature-admin';
          }
        },
      },
    },
  },
});
```

**3. Chunk Size Warnings:**
```javascript
export default defineConfig({
  build: {
    chunkSizeWarningLimit: 500, // KB
    rollupOptions: {
      output: {
        manualChunks(id) {
          // Keep chunks under 500KB
          if (id.includes('node_modules')) {
            const packages = id.split('node_modules/')[1].split('/');
            const packageName = packages[0];
            
            // Large packages get their own chunks
            if (['@mui', 'lodash', 'moment'].includes(packageName)) {
              return `vendor-${packageName}`;
            }
            
            return 'vendor';
          }
        },
      },
    },
  },
});
```

---

#### Route-Based Code Splitting

**React Router v6:**
```typescript
import { lazy, Suspense } from 'react';
import { createBrowserRouter, RouterProvider } from 'react-router-dom';

// Lazy load routes
const Home = lazy(() => import('./pages/Home'));
const Products = lazy(() => import('./pages/Products'));
const ProductDetail = lazy(() => import('./pages/ProductDetail'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));

const router = createBrowserRouter([
  {
    path: '/',
    element: (
      <Suspense fallback={<PageLoader />}>
        <Home />
      </Suspense>
    ),
  },
  {
    path: '/products',
    element: (
      <Suspense fallback={<PageLoader />}>
        <Products />
      </Suspense>
    ),
  },
  {
    path: '/products/:id',
    element: (
      <Suspense fallback={<PageLoader />}>
        <ProductDetail />
      </Suspense>
    ),
  },
  {
    path: '/dashboard',
    element: (
      <Suspense fallback={<PageLoader />}>
        <Dashboard />
      </Suspense>
    ),
  },
]);

function App() {
  return <RouterProvider router={router} />;
}
```

**Nested Routes:**
```typescript
const DashboardLayout = lazy(() => import('./layouts/DashboardLayout'));
const DashboardHome = lazy(() => import('./pages/dashboard/Home'));
const DashboardAnalytics = lazy(() => import('./pages/dashboard/Analytics'));
const DashboardSettings = lazy(() => import('./pages/dashboard/Settings'));

const router = createBrowserRouter([
  {
    path: '/dashboard',
    element: (
      <Suspense fallback={<LayoutLoader />}>
        <DashboardLayout />
      </Suspense>
    ),
    children: [
      {
        index: true,
        element: (
          <Suspense fallback={<ContentLoader />}>
            <DashboardHome />
          </Suspense>
        ),
      },
      {
        path: 'analytics',
        element: (
          <Suspense fallback={<ContentLoader />}>
            <DashboardAnalytics />
          </Suspense>
        ),
      },
      {
        path: 'settings',
        element: (
          <Suspense fallback={<ContentLoader />}>
            <DashboardSettings />
          </Suspense>
        ),
      },
    ],
  },
]);
```

---

#### Library/Vendor Splitting

**Separate Large Libraries:**
```javascript
// webpack.config.js
module.exports = {
  optimization: {
    splitChunks: {
      cacheGroups: {
        // React (changes rarely)
        react: {
          test: /[\\/]node_modules[\\/](react|react-dom)[\\/]/,
          name: 'react',
          chunks: 'all',
          priority: 20,
        },
        // Large libraries (separate chunks for better caching)
        lodash: {
          test: /[\\/]node_modules[\\/]lodash[\\/]/,
          name: 'lodash',
          chunks: 'all',
          priority: 15,
        },
        moment: {
          test: /[\\/]node_modules[\\/]moment[\\/]/,
          name: 'moment',
          chunks: 'all',
          priority: 15,
        },
        // UI framework
        mui: {
          test: /[\\/]node_modules[\\/]@mui[\\/]/,
          name: 'mui',
          chunks: 'all',
          priority: 15,
        },
        // All other vendors
        vendors: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
          priority: 10,
        },
      },
    },
  },
};
```

---

#### Feature-Based Splitting

**Directory Structure:**
```
src/
├── features/
│   ├── auth/
│   │   ├── Login.tsx
│   │   ├── Register.tsx
│   │   └── ForgotPassword.tsx
│   ├── dashboard/
│   │   ├── Dashboard.tsx
│   │   ├── Analytics.tsx
│   │   └── Reports.tsx
│   └── admin/
│       ├── Users.tsx
│       └── Settings.tsx
```

**Configuration:**
```javascript
// webpack.config.js
module.exports = {
  optimization: {
    splitChunks: {
      cacheGroups: {
        // Split by feature
        auth: {
          test: /[\\/]src[\\/]features[\\/]auth[\\/]/,
          name: 'feature-auth',
          chunks: 'all',
          priority: 10,
        },
        dashboard: {
          test: /[\\/]src[\\/]features[\\/]dashboard[\\/]/,
          name: 'feature-dashboard',
          chunks: 'all',
          priority: 10,
        },
        admin: {
          test: /[\\/]src[\\/]features[\\/]admin[\\/]/,
          name: 'feature-admin',
          chunks: 'all',
          priority: 10,
        },
      },
    },
  },
};
```

**Lazy Load Features:**
```typescript
const AuthFeature = lazy(() => import('./features/auth'));
const DashboardFeature = lazy(() => import('./features/dashboard'));
const AdminFeature = lazy(() => import('./features/admin'));

function App() {
  return (
    <Routes>
      <Route
        path="/auth/*"
        element={
          <Suspense fallback={<Loader />}>
            <AuthFeature />
          </Suspense>
        }
      />
      <Route
        path="/dashboard/*"
        element={
          <Suspense fallback={<Loader />}>
            <DashboardFeature />
          </Suspense>
        }
      />
      <Route
        path="/admin/*"
        element={
          <Suspense fallback={<Loader />}>
            <AdminFeature />
          </Suspense>
        }
      />
    </Routes>
  );
}
```

---

#### Preload and Prefetch

**Prefetch (Low Priority):**
```typescript
// Load in idle time for future navigation
const Settings = lazy(() =>
  import(
    /* webpackPrefetch: true */
    './pages/Settings'
  )
);

// Generates: <link rel="prefetch" href="settings.chunk.js">
// Loaded when browser is idle
```

**Preload (High Priority):**
```typescript
// Load in parallel with parent
const CriticalComponent = lazy(() =>
  import(
    /* webpackPreload: true */
    './CriticalComponent'
  )
);

// Generates: <link rel="preload" href="critical.chunk.js">
// Loaded immediately
```

**Manual Prefetching:**
```typescript
import { useEffect } from 'react';

function HomePage() {
  useEffect(() => {
    // Prefetch dashboard when user lands on home
    const prefetchDashboard = () => import('./pages/Dashboard');
    
    // Start prefetch after 2 seconds
    const timer = setTimeout(prefetchDashboard, 2000);
    
    return () => clearTimeout(timer);
  }, []);

  return <div>Home Page</div>;
}
```

**On Hover Prefetch:**
```typescript
function Navigation() {
  const [prefetched, setPrefetched] = useState(new Set());

  const handleHover = (route: string) => {
    if (!prefetched.has(route)) {
      import(`./pages/${route}`);
      setPrefetched(prev => new Set(prev).add(route));
    }
  };

  return (
    <nav>
      <Link
        to="/dashboard"
        onMouseEnter={() => handleHover('Dashboard')}
      >
        Dashboard
      </Link>
      <Link
        to="/settings"
        onMouseEnter={() => handleHover('Settings')}
      >
        Settings
      </Link>
    </nav>
  );
}
```

---

#### Analyzing Bundle Size

**Webpack Bundle Analyzer:**
```javascript
// Install
npm install --save-dev webpack-bundle-analyzer

// webpack.config.js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: 'static',
      openAnalyzer: false,
      reportFilename: 'bundle-report.html',
    }),
  ],
};

// Run build and open report
npm run build
// Open bundle-report.html
```

**Rollup Visualizer (Vite):**
```javascript
// Install
npm install --save-dev rollup-plugin-visualizer

// vite.config.js
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    react(),
    visualizer({
      open: true,
      gzipSize: true,
      brotliSize: true,
    }),
  ],
});
```

---

#### Best Practices

**✅ Split by Routes:**
```typescript
// Each route = separate chunk
const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
```

**✅ Separate Vendors:**
```javascript
// React, lodash, etc. in separate chunks
manualChunks: {
  vendor: ['react', 'react-dom'],
  utils: ['lodash', 'date-fns'],
}
```

**✅ Lazy Load Heavy Components:**
```typescript
// Charts, editors, video players
const Chart = lazy(() => import('./HeavyChart'));
```

**✅ Use Prefetching:**
```typescript
// Prefetch likely next pages
import(/* webpackPrefetch: true */ './Dashboard');
```

**❌ Don't Over-Split:**
```javascript
// Too many small chunks = more HTTP requests
// Find balance
```

**❌ Don't Split Critical Path:**
```typescript
// Don't lazy load above-the-fold content
// Load immediately
```

---

#### Interview Tips

✅ **Key Points:**
- Split code into smaller chunks for better performance
- Route-based, component-based, vendor-based strategies
- React.lazy() + Suspense for components
- Webpack splitChunks, Vite manualChunks
- Prefetch/preload for optimization
- Balance: not too many, not too few chunks

✅ **When to Mention:**
- Performance optimization
- Bundle size reduction
- Initial load time improvement
- Lazy loading strategies
- Build configuration

✅ **Common Follow-ups:**
- "When should you use code splitting?"
- "What's the difference between prefetch and preload?"
- "How do you analyze bundle size?"
- "What are the trade-offs of code splitting?"
- "How does React.lazy() work?"

✅ **Perfect Answer Structure:**
1. Define: Splitting bundle into smaller chunks
2. Why: Faster initial load, better caching
3. Methods: Route, component, vendor splitting
4. Tools: React.lazy(), Webpack splitChunks, Vite manualChunks
5. Strategies: Routes first, then heavy components, separate vendors
6. Optimization: Prefetch, preload, analyze
7. Balance: Find sweet spot between too many/few chunks

</details>

---

### 175. What is ESBuild and why is it fast?

<details>
<summary>View Answer</summary>

**ESBuild** is an extremely fast JavaScript bundler and minifier written in Go. It's 10-100x faster than traditional JavaScript-based bundlers like Webpack, Rollup, and Parcel due to its compiled nature, parallelization, and optimized algorithms.

#### What is ESBuild?

**Key Characteristics:**
- Written in Go (compiled language)
- Extremely fast bundling and minification
- Built-in TypeScript and JSX support
- Tree shaking and code splitting
- Source maps generation
- Minimal configuration
- Used by Vite for dependency pre-bundling

**Speed Comparison:**
```
Bundling 10MB of JavaScript:

Tool        | Time      | Language
------------|-----------|----------
ESBuild     | 0.5s      | Go
Rollup      | 35s       | JavaScript
Webpack     | 41s       | JavaScript
Parcel      | 52s       | JavaScript

ESBuild is 70-100x faster!
```

---

#### Why ESBuild is Fast

**1. Written in Go (Not JavaScript):**
```
JavaScript Bundlers (Webpack, Rollup):
- Interpreted by V8/Node.js
- Single-threaded (mostly)
- Garbage collection overhead
- Dynamic typing overhead

ESBuild (Go):
- Compiled to native machine code
- Runs directly on CPU
- Efficient memory management
- Multi-threaded by default
- Static typing
```

**2. Parallelism:**
```go
// ESBuild uses all CPU cores
// Pseudocode representation
func Bundle(files []File) {
    // Parse all files in parallel
    parsed := ParallelMap(files, Parse)
    
    // Transform in parallel
    transformed := ParallelMap(parsed, Transform)
    
    // Link (this part is sequential but fast)
    output := Link(transformed)
}

// JavaScript bundlers:
// Mostly single-threaded
// Sequential processing
```

**3. Efficient Memory Usage:**
```
JavaScript Bundlers:
- Create many intermediate objects
- Frequent garbage collection
- AST transformations allocate memory

ESBuild:
- Minimal allocations
- Reuses memory when possible
- Efficient data structures
- Direct memory management
```

**4. No Plugin Overhead:**
```
Webpack:
- Extensive plugin system
- Each plugin adds overhead
- JavaScript function calls

ESBuild:
- Built-in features
- Native code execution
- Minimal overhead
```

**5. Optimized Algorithms:**
```
ESBuild:
- Everything done in 3 passes:
  1. Parse all files
  2. Link modules together
  3. Generate output
- No multiple traversals
- Efficient dependency resolution
```

---

#### Installation and Basic Usage

**Installation:**
```bash
# npm
npm install --save-dev esbuild

# yarn
yarn add --dev esbuild

# pnpm
pnpm add -D esbuild

# Or use npx (no installation)
npx esbuild app.js --bundle --outfile=out.js
```

**Command Line:**
```bash
# Bundle a single file
esbuild app.js --bundle --outfile=out.js

# With minification
esbuild app.js --bundle --minify --outfile=out.js

# With source maps
esbuild app.js --bundle --sourcemap --outfile=out.js

# Development mode with watch
esbuild app.js --bundle --outfile=out.js --watch

# Serve with hot reload
esbuild app.js --bundle --servedir=public --serve=8000
```

**Build Script:**
```javascript
// build.js
const esbuild = require('esbuild');

esbuild.build({
  entryPoints: ['src/app.js'],
  bundle: true,
  minify: true,
  sourcemap: true,
  target: 'es2020',
  outfile: 'dist/app.js',
}).catch(() => process.exit(1));
```

```json
// package.json
{
  "scripts": {
    "build": "node build.js"
  }
}
```

---

#### React with ESBuild

**Basic React Setup:**
```javascript
// build.js
const esbuild = require('esbuild');

esbuild.build({
  entryPoints: ['src/index.jsx'],
  bundle: true,
  minify: true,
  sourcemap: true,
  target: ['chrome90', 'firefox88', 'safari14'],
  outfile: 'dist/bundle.js',
  loader: {
    '.js': 'jsx',
    '.jsx': 'jsx',
  },
  define: {
    'process.env.NODE_ENV': '"production"',
  },
}).catch(() => process.exit(1));
```

**TypeScript + React:**
```javascript
esbuild.build({
  entryPoints: ['src/index.tsx'],
  bundle: true,
  minify: true,
  sourcemap: true,
  target: 'es2020',
  outfile: 'dist/bundle.js',
  loader: {
    '.ts': 'ts',
    '.tsx': 'tsx',
  },
  define: {
    'process.env.NODE_ENV': '"production"',
  },
});
```

**Development Server:**
```javascript
// serve.js
const esbuild = require('esbuild');

esbuild.serve(
  {
    servedir: 'public',
    port: 3000,
  },
  {
    entryPoints: ['src/index.jsx'],
    bundle: true,
    outfile: 'public/bundle.js',
    loader: {
      '.jsx': 'jsx',
    },
  }
).then(server => {
  console.log(`Server running at http://localhost:3000`);
});
```

---

#### Advanced Configuration

**1. Multiple Entry Points:**
```javascript
esbuild.build({
  entryPoints: [
    'src/app.js',
    'src/admin.js',
    'src/worker.js',
  ],
  bundle: true,
  outdir: 'dist',
  splitting: true, // Code splitting
  format: 'esm',
});
```

**2. CSS Handling:**
```javascript
esbuild.build({
  entryPoints: ['src/app.jsx'],
  bundle: true,
  outfile: 'dist/app.js',
  loader: {
    '.css': 'css',
    '.module.css': 'local-css', // CSS modules
    '.png': 'file',
    '.jpg': 'file',
    '.svg': 'dataurl', // Inline as base64
  },
});
```

**3. External Dependencies:**
```javascript
esbuild.build({
  entryPoints: ['src/app.js'],
  bundle: true,
  outfile: 'dist/app.js',
  external: ['react', 'react-dom'], // Don't bundle these
});
```

**4. Environment Variables:**
```javascript
esbuild.build({
  entryPoints: ['src/app.js'],
  bundle: true,
  outfile: 'dist/app.js',
  define: {
    'process.env.NODE_ENV': '"production"',
    'process.env.API_URL': '"https://api.example.com"',
    '__VERSION__': '"1.0.0"',
  },
});
```

**5. Plugins:**
```javascript
// Custom plugin example
const envPlugin = {
  name: 'env',
  setup(build) {
    build.onLoad({ filter: /\.js$/ }, async (args) => {
      const contents = await fs.promises.readFile(args.path, 'utf8');
      const replaced = contents.replace(
        /process\.env\.([A-Z_]+)/g,
        (match, key) => JSON.stringify(process.env[key])
      );
      return { contents: replaced, loader: 'js' };
    });
  },
};

esbuild.build({
  entryPoints: ['src/app.js'],
  bundle: true,
  outfile: 'dist/app.js',
  plugins: [envPlugin],
});
```

---

#### Watch Mode

**File Watching:**
```javascript
const esbuild = require('esbuild');

const ctx = await esbuild.context({
  entryPoints: ['src/app.jsx'],
  bundle: true,
  outfile: 'dist/bundle.js',
  loader: { '.jsx': 'jsx' },
});

// Enable watch mode
await ctx.watch();

console.log('Watching for changes...');
```

**With Dev Server:**
```javascript
const ctx = await esbuild.context({
  entryPoints: ['src/app.jsx'],
  bundle: true,
  outdir: 'dist',
  loader: { '.jsx': 'jsx' },
});

// Start server with live reload
await ctx.serve({
  servedir: 'public',
  port: 3000,
});

await ctx.watch();

console.log('Server running at http://localhost:3000');
```

---

#### ESBuild in Vite

**How Vite Uses ESBuild:**
```
Vite Architecture:

1. Dependency Pre-Bundling:
   - node_modules bundled with ESBuild
   - Fast pre-bundling (seconds, not minutes)
   - Cached aggressively

2. Development:
   - Source files: Native ESM (not ESBuild)
   - Dependencies: Pre-bundled with ESBuild

3. Production:
   - Uses Rollup (not ESBuild)
   - Better tree-shaking
   - More mature plugin ecosystem
```

**Vite Configuration:**
```javascript
// vite.config.js
import { defineConfig } from 'vite';

export default defineConfig({
  // ESBuild options
  esbuild: {
    jsxFactory: 'h',
    jsxFragment: 'Fragment',
    jsxInject: `import React from 'react'`,
  },
  
  optimizeDeps: {
    // Control ESBuild pre-bundling
    include: ['react', 'react-dom'],
    esbuildOptions: {
      target: 'esnext',
    },
  },
});
```

---

#### Performance Benchmarks

**Real-World Project (500 modules):**
```
Task                | Webpack | Rollup | Parcel | ESBuild
--------------------|---------|--------|--------|--------
Cold Build          | 25.3s   | 18.7s  | 31.2s  | 0.8s
Rebuild (1 file)    | 3.2s    | 2.1s   | 4.5s   | 0.05s
Production Build    | 42.6s   | 28.4s  | 45.1s  | 1.2s
Minification Only   | 8.1s    | 6.3s   | 9.2s   | 0.3s
```

**Large Project (3000+ modules):**
```
Task                | Webpack | ESBuild | Speedup
--------------------|---------|---------|--------
Initial Build       | 182s    | 3.4s    | 53x
Incremental Build   | 12s     | 0.1s    | 120x
Minification        | 31s     | 0.9s    | 34x
```

---

#### Limitations

**❌ No Hot Module Replacement (HMR):**
```
- Can watch files and rebuild
- But no built-in HMR like Webpack/Vite
- Need custom implementation
- Or use Vite (which uses ESBuild)
```

**❌ Limited Plugin Ecosystem:**
```
- Fewer plugins than Webpack
- API is different from Webpack
- Some transformations harder to implement
```

**❌ Less Mature Tree Shaking:**
```
- Tree shaking is good but not as advanced as Rollup
- Some edge cases not handled
- Improving but not production-ready for all cases
```

**❌ No Transpilation for Old Browsers:**
```javascript
// ESBuild supports:
target: ['es2020', 'chrome90', 'firefox88']

// But won't transform everything for IE11
// For old browsers, use Babel
```

---

#### ESBuild vs Other Tools

**ESBuild vs Webpack:**
```
Feature             | Webpack        | ESBuild
--------------------|----------------|------------------
Speed               | Slow           | Extremely Fast
Plugin Ecosystem    | Huge           | Growing
Configuration       | Complex        | Simple
HMR                 | Yes            | No (need wrapper)
Tree Shaking        | Excellent      | Good
Code Splitting      | Advanced       | Basic
Production Ready    | Yes            | Mostly
Learning Curve      | Steep          | Easy
```

**ESBuild vs Rollup:**
```
Feature             | Rollup         | ESBuild
--------------------|----------------|------------------
Speed               | Medium         | Extremely Fast
Tree Shaking        | Best-in-class  | Good
Output Size         | Smallest       | Small
Library Bundling    | Excellent      | Good
Plugin Ecosystem    | Mature         | Growing
```

**ESBuild vs SWC:**
```
Feature             | SWC            | ESBuild
--------------------|----------------|------------------
Language            | Rust           | Go
Speed               | Very Fast      | Very Fast
Focus               | Transpilation  | Bundling
Babel Compatible    | Yes            | No
Bundling            | Via plugins    | Built-in
```

---

#### Use Cases

**✅ Perfect For:**

1. **Development Builds:**
```javascript
// Super fast rebuilds during development
esbuild.build({
  entryPoints: ['src/app.js'],
  bundle: true,
  outfile: 'dist/app.js',
  sourcemap: true,
  watch: true,
});
```

2. **Dependency Pre-Bundling:**
```javascript
// Bundle node_modules for faster serving
esbuild.build({
  entryPoints: ['react', 'react-dom', 'lodash'],
  bundle: true,
  outdir: 'vendor',
  format: 'esm',
});
```

3. **TypeScript/JSX Transpilation:**
```javascript
// Fast TS/JSX compilation
esbuild.transform(code, {
  loader: 'tsx',
  target: 'es2020',
});
```

4. **Minification:**
```javascript
// Extremely fast minification
esbuild.build({
  entryPoints: ['dist/bundle.js'],
  minify: true,
  outfile: 'dist/bundle.min.js',
  allowOverwrite: true,
});
```

**❌ May Not Be Ideal For:**

1. **Complex Production Builds:**
   - Need advanced code splitting
   - Require extensive plugin customization
   - Legacy browser support

2. **When You Need HMR:**
   - Use Vite or Webpack instead
   - Or implement custom HMR

3. **Library Development:**
   - Rollup better for libraries
   - More control over output

---

#### Complete React Example

**Project Structure:**
```
my-app/
├── src/
│   ├── index.jsx
│   ├── App.jsx
│   └── components/
├── public/
│   ├── index.html
│   └── styles.css
├── build.js
└── package.json
```

**build.js:**
```javascript
const esbuild = require('esbuild');
const { copy } = require('esbuild-plugin-copy');

const isProduction = process.env.NODE_ENV === 'production';

esbuild.build({
  entryPoints: ['src/index.jsx'],
  bundle: true,
  minify: isProduction,
  sourcemap: !isProduction,
  target: ['chrome90', 'firefox88', 'safari14'],
  outfile: 'dist/bundle.js',
  
  loader: {
    '.jsx': 'jsx',
    '.js': 'jsx',
    '.png': 'file',
    '.jpg': 'file',
    '.svg': 'dataurl',
  },
  
  define: {
    'process.env.NODE_ENV': JSON.stringify(
      isProduction ? 'production' : 'development'
    ),
  },
  
  plugins: [
    copy({
      resolveFrom: 'cwd',
      assets: {
        from: ['./public/**/*'],
        to: ['./dist'],
      },
    }),
  ],
}).then(() => {
  console.log('✓ Build complete');
}).catch(() => process.exit(1));
```

**package.json:**
```json
{
  "scripts": {
    "dev": "node build.js",
    "build": "NODE_ENV=production node build.js"
  },
  "devDependencies": {
    "esbuild": "^0.19.0",
    "esbuild-plugin-copy": "^2.1.0"
  }
}
```

---

#### Interview Tips

✅ **Key Points:**
- Written in Go, compiled to native code
- 10-100x faster than JS-based bundlers
- Parallelizes everything across CPU cores
- Efficient memory usage and algorithms
- Built-in TypeScript, JSX support
- Used by Vite for dependency pre-bundling
- Trade-off: speed vs features/ecosystem

✅ **When to Mention:**
- Performance discussions
- Build tool comparisons
- Why Vite is fast
- Modern tooling trends
- Development experience optimization

✅ **Common Follow-ups:**
- "Why is ESBuild faster than Webpack?"
- "What are the limitations of ESBuild?"
- "When should you use ESBuild?"
- "How does Vite use ESBuild?"
- "Can ESBuild replace Webpack?"

✅ **Perfect Answer Structure:**
1. Define: Go-based bundler, extremely fast
2. Speed: 10-100x faster due to compiled nature
3. Why fast: Go, parallelism, efficient algorithms
4. Features: TypeScript, JSX, minification, tree shaking
5. Limitations: No HMR, smaller ecosystem
6. Use cases: Dev builds, dependency bundling, Vite
7. Trade-off: Speed vs advanced features

</details>

---

### 176. What are the benefits of SWC compiler?

<details>
<summary>View Answer</summary>

**SWC (Speedy Web Compiler)** is an extremely fast TypeScript/JavaScript compiler and bundler written in Rust. It's a drop-in replacement for Babel that's 20x faster in single-threaded and 70x faster in multi-core scenarios, while maintaining near-perfect compatibility with Babel's transform ecosystem.

#### What is SWC?

**Key Characteristics:**
- Written in Rust (compiled, not JavaScript)
- 20-70x faster than Babel
- Drop-in Babel replacement
- TypeScript, JSX, and modern JS support
- Plugin ecosystem (Babel-compatible)
- Used by Next.js, Parcel, Deno
- Bundling and minification

**Speed Comparison:**
```
Transpiling 100,000 lines of TypeScript:

Tool        | Time      | Language
------------|-----------|----------
SWC         | 0.8s      | Rust
ESBuild     | 1.2s      | Go
Babel       | 18s       | JavaScript
tsc         | 45s       | TypeScript

SWC is 20-55x faster!
```

---

#### Benefits of SWC

**1. Blazing Fast Performance:**
```
Real-world Next.js project (3000 modules):

Metric              | Babel     | SWC       | Speedup
--------------------|-----------|-----------|--------
Dev Server Start    | 42s       | 3.2s      | 13x
Fast Refresh        | 2.1s      | 0.15s     | 14x
Production Build    | 180s      | 12s       | 15x
Transpilation Only  | 24s       | 1.1s      | 22x
```

**2. Babel Compatibility:**
```javascript
// Most Babel presets work with SWC
// .babelrc
{
  "presets": [
    "@babel/preset-react",
    "@babel/preset-typescript"
  ],
  "plugins": [
    "@babel/plugin-proposal-class-properties"
  ]
}

// .swcrc (equivalent)
{
  "jsc": {
    "parser": {
      "syntax": "typescript",
      "tsx": true
    },
    "transform": {
      "react": {
        "runtime": "automatic"
      }
    }
  }
}
```

**3. Built-in Features:**
```
✅ TypeScript compilation
✅ JSX/TSX transformation
✅ Modern JS to ES5
✅ Minification
✅ Source maps
✅ Tree shaking
✅ Code splitting
✅ CSS modules
✅ Decorators
✅ Class properties
```

**4. Lower Memory Usage:**
```
Memory consumption for large project:

Babel: 1.2 GB
SWC:   180 MB

6.7x less memory usage
```

---

#### Installation and Setup

**Installation:**
```bash
# Core package
npm install --save-dev @swc/core

# CLI tool
npm install --save-dev @swc/cli

# For webpack
npm install --save-dev swc-loader

# For Jest
npm install --save-dev @swc/jest
```

**Basic Configuration:**
```json
// .swcrc
{
  "jsc": {
    "parser": {
      "syntax": "typescript",
      "tsx": true,
      "decorators": true,
      "dynamicImport": true
    },
    "transform": {
      "react": {
        "runtime": "automatic",
        "pragma": "React.createElement",
        "pragmaFrag": "React.Fragment",
        "throwIfNamespace": true,
        "development": false,
        "useBuiltins": false
      }
    },
    "target": "es2020",
    "loose": false,
    "externalHelpers": false,
    "keepClassNames": false
  },
  "module": {
    "type": "es6",
    "strict": false,
    "strictMode": true,
    "lazy": false,
    "noInterop": false
  },
  "minify": false,
  "sourceMaps": true
}
```

---

#### Using SWC with Different Tools

**1. Next.js (Built-in):**
```javascript
// next.config.js
module.exports = {
  // SWC is default in Next.js 12+
  // No configuration needed!
  
  // To customize SWC:
  swcMinify: true, // Use SWC for minification
  
  // Or configure compiler options:
  compiler: {
    // Remove console in production
    removeConsole: true,
    
    // React optimization
    reactRemoveProperties: true,
    
    // Styled components
    styledComponents: true,
    
    // Emotion
    emotion: true,
  },
};
```

**2. Webpack:**
```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.(js|jsx|ts|tsx)$/,
        exclude: /node_modules/,
        use: {
          loader: 'swc-loader',
          options: {
            jsc: {
              parser: {
                syntax: 'typescript',
                tsx: true,
              },
              transform: {
                react: {
                  runtime: 'automatic',
                },
              },
            },
          },
        },
      },
    ],
  },
};
```

**3. Jest:**
```javascript
// jest.config.js
module.exports = {
  transform: {
    '^.+\\.(t|j)sx?$': ['@swc/jest', {
      jsc: {
        parser: {
          syntax: 'typescript',
          tsx: true,
        },
        transform: {
          react: {
            runtime: 'automatic',
          },
        },
      },
    }],
  },
};
```

**4. Standalone Compilation:**
```javascript
// compile.js
const swc = require('@swc/core');
const fs = require('fs');

const source = fs.readFileSync('src/App.tsx', 'utf8');

swc.transform(source, {
  filename: 'App.tsx',
  jsc: {
    parser: {
      syntax: 'typescript',
      tsx: true,
    },
    transform: {
      react: {
        runtime: 'automatic',
      },
    },
    target: 'es2020',
  },
  module: {
    type: 'es6',
  },
}).then(output => {
  fs.writeFileSync('dist/App.js', output.code);
  if (output.map) {
    fs.writeFileSync('dist/App.js.map', output.map);
  }
});
```

**5. CLI Usage:**
```bash
# Compile single file
swc ./src/app.ts -o ./dist/app.js

# Compile directory
swc ./src -d ./dist

# Watch mode
swc ./src -d ./dist --watch

# With source maps
swc ./src -d ./dist --source-maps

# Minify
swc ./src -d ./dist --minify
```

---

#### React Configuration

**Modern React (Automatic JSX):**
```json
{
  "jsc": {
    "parser": {
      "syntax": "typescript",
      "tsx": true
    },
    "transform": {
      "react": {
        "runtime": "automatic",  // React 17+ automatic import
        "development": false,
        "refresh": false  // Set true for Fast Refresh in dev
      }
    }
  }
}
```

**Classic React:**
```json
{
  "jsc": {
    "parser": {
      "syntax": "typescript",
      "tsx": true
    },
    "transform": {
      "react": {
        "runtime": "classic",
        "pragma": "React.createElement",
        "pragmaFrag": "React.Fragment",
        "importSource": "react"
      }
    }
  }
}
```

**Fast Refresh (Development):**
```json
{
  "jsc": {
    "parser": {
      "syntax": "typescript",
      "tsx": true
    },
    "transform": {
      "react": {
        "runtime": "automatic",
        "development": true,
        "refresh": true  // Enable Fast Refresh
      }
    }
  }
}
```

---

#### Advanced Features

**1. Custom Plugins:**
```rust
// SWC plugins are written in Rust
// Example: Custom transform plugin
use swc_core::ecma::{
    ast::*,
    visit::{Fold, FoldWith},
};

pub struct CustomTransform;

impl Fold for CustomTransform {
    fn fold_expr(&mut self, expr: Expr) -> Expr {
        // Transform expressions
        expr.fold_children_with(self)
    }
}
```

**2. Minification:**
```json
{
  "minify": true,
  "jsc": {
    "minify": {
      "compress": {
        "unused": true
      },
      "mangle": true
    }
  }
}
```

**3. Module Resolution:**
```json
{
  "jsc": {
    "baseUrl": "./src",
    "paths": {
      "@components/*": ["components/*"],
      "@utils/*": ["utils/*"]
    }
  }
}
```

**4. Decorators:**
```json
{
  "jsc": {
    "parser": {
      "syntax": "typescript",
      "decorators": true
    },
    "transform": {
      "decoratorMetadata": true,
      "legacyDecorator": true
    }
  }
}
```

---

#### Migration from Babel

**Step-by-Step:**

**1. Install SWC:**
```bash
npm install --save-dev @swc/core swc-loader
npm uninstall babel-loader @babel/core @babel/preset-env @babel/preset-react
```

**2. Convert Babel Config:**
```javascript
// Before: .babelrc
{
  "presets": [
    "@babel/preset-env",
    "@babel/preset-react",
    "@babel/preset-typescript"
  ],
  "plugins": [
    "@babel/plugin-proposal-class-properties"
  ]
}

// After: .swcrc
{
  "jsc": {
    "parser": {
      "syntax": "typescript",
      "tsx": true,
      "decorators": true
    },
    "transform": {
      "react": {
        "runtime": "automatic"
      }
    },
    "target": "es2015"
  }
}
```

**3. Update Webpack:**
```javascript
// Before
module.exports = {
  module: {
    rules: [
      {
        test: /\.(js|jsx|ts|tsx)$/,
        use: 'babel-loader',
      },
    ],
  },
};

// After
module.exports = {
  module: {
    rules: [
      {
        test: /\.(js|jsx|ts|tsx)$/,
        use: 'swc-loader',
      },
    ],
  },
};
```

**4. Update Jest:**
```javascript
// Before
module.exports = {
  transform: {
    '^.+\\.(js|jsx|ts|tsx)$': 'babel-jest',
  },
};

// After
module.exports = {
  transform: {
    '^.+\\.(t|j)sx?$': '@swc/jest',
  },
};
```

---

#### Performance Optimization

**1. Parallel Compilation:**
```javascript
// SWC automatically uses all CPU cores
// No configuration needed

// For webpack with multiple entries:
module.exports = {
  module: {
    rules: [
      {
        test: /\.(js|jsx|ts|tsx)$/,
        use: {
          loader: 'swc-loader',
          options: {
            // SWC will parallelize automatically
          },
        },
      },
    ],
  },
  // Enable parallel builds
  parallelism: 100,
};
```

**2. Caching:**
```javascript
// webpack.config.js
module.exports = {
  cache: {
    type: 'filesystem',
    buildDependencies: {
      config: [__filename],
    },
  },
};
```

**3. Target Optimization:**
```json
// .swcrc
{
  "jsc": {
    "target": "es2020",  // Target modern browsers
    "loose": true,       // Faster, less spec-compliant
    "externalHelpers": true  // Reduce duplication
  }
}
```

---

#### SWC vs Other Tools

**SWC vs Babel:**
```
Feature              | Babel         | SWC
---------------------|---------------|------------------
Speed                | Baseline      | 20-70x faster
Language             | JavaScript    | Rust
Plugin Ecosystem     | Huge          | Growing
Babel Compatibility  | 100%          | ~95%
Memory Usage         | High          | Low
Production Ready     | Yes           | Yes
Configuration        | Complex       | Simpler
```

**SWC vs ESBuild:**
```
Feature              | ESBuild       | SWC
---------------------|---------------|------------------
Speed                | Very Fast     | Very Fast
Language             | Go            | Rust
Babel Compatible     | No            | Yes
Bundling             | Yes           | Yes (via spack)
Transpilation        | Basic         | Advanced
Plugin System        | Limited       | Babel-like
```

**SWC vs TypeScript Compiler:**
```
Feature              | tsc           | SWC
---------------------|---------------|------------------
Speed                | Slow          | 20x faster
Type Checking        | Yes           | No (use tsc separately)
Transpilation        | Yes           | Yes
Watch Mode           | Slow          | Fast
Memory Usage         | High          | Low
```

---

#### Real-World Adoption

**Next.js:**
```javascript
// Next.js 12+ uses SWC by default
// Replaced Babel completely
// Result: 5x faster Fast Refresh, 3x faster builds

next dev   // Uses SWC
next build // Uses SWC
```

**Deno:**
```typescript
// Deno uses SWC for TypeScript compilation
// Fast startup times
// Efficient bundling
```

**Parcel:**
```javascript
// Parcel 2 uses SWC
// Faster builds
// Better performance
```

---

#### Common Use Cases

**1. Large Projects:**
```
Project with 3000+ TypeScript files:
- Babel: 3-5 minute builds
- SWC: 15-30 second builds

10-20x improvement
```

**2. Fast Refresh:**
```
React Fast Refresh with large project:
- Babel: 2-3 seconds per change
- SWC: 100-200ms per change

10-30x faster feedback loop
```

**3. CI/CD:**
```
CI build time reduction:
- Before (Babel): 10 minutes
- After (SWC): 2 minutes

5x faster deployments
```

---

#### Limitations

**❌ No Type Checking:**
```typescript
// SWC only transpiles, doesn't check types
// Run tsc separately:

// package.json
{
  "scripts": {
    "typecheck": "tsc --noEmit",
    "build": "npm run typecheck && swc src -d dist"
  }
}
```

**❌ Some Babel Plugins Incompatible:**
```
// Most plugins work, but some don't:
// - Very custom transformations
// - Plugins using Babel internals
// - Check SWC compatibility before migrating
```

**❌ Smaller Ecosystem:**
```
// Fewer resources compared to Babel:
// - Less documentation
// - Fewer examples
// - Smaller community
// But growing rapidly!
```

---

#### Best Practices

**✅ Use for Transpilation:**
```json
// Fast TypeScript/JSX compilation
{
  "jsc": {
    "parser": { "syntax": "typescript", "tsx": true },
    "target": "es2020"
  }
}
```

**✅ Separate Type Checking:**
```bash
# Use tsc for type checking
npm run typecheck  # tsc --noEmit

# Use SWC for compilation
swc src -d dist
```

**✅ Enable in Next.js:**
```javascript
// next.config.js
module.exports = {
  swcMinify: true,  // Use SWC minifier
};
```

**✅ Cache Builds:**
```javascript
// webpack.config.js
module.exports = {
  cache: {
    type: 'filesystem',
  },
};
```

---

#### Interview Tips

✅ **Key Points:**
- Rust-based compiler, 20-70x faster than Babel
- Drop-in Babel replacement with high compatibility
- Used by Next.js, Deno, Parcel
- Transpilation only (no type checking)
- Much lower memory usage
- Growing plugin ecosystem

✅ **When to Mention:**
- Build performance optimization
- Modern tooling discussions
- Next.js features
- Babel alternatives
- Rust in frontend tooling

✅ **Common Follow-ups:**
- "How is SWC different from Babel?"
- "Why is SWC faster?"
- "Does SWC check TypeScript types?"
- "How compatible is SWC with Babel?"
- "Should I migrate from Babel to SWC?"

✅ **Perfect Answer Structure:**
1. Define: Rust-based compiler, Babel replacement
2. Speed: 20-70x faster due to compiled Rust
3. Features: TypeScript, JSX, minification, Babel compatibility
4. Adoption: Next.js, Deno, Parcel
5. Benefits: Speed, low memory, production-ready
6. Limitations: No type checking, smaller ecosystem
7. Migration: Easy from Babel, significant speedup

</details>

---

### 177. How do you optimize build times for large projects?

<details>
<summary>View Answer</summary>

**Optimizing build times** for large projects involves multiple strategies across bundler configuration, code organization, caching, parallelization, and tooling choices. A comprehensive approach can reduce build times from minutes to seconds.

#### Understanding the Problem

**Typical Large Project Issues:**
```
Before Optimization:
- Cold build: 5-10 minutes
- Incremental build: 30-60 seconds
- Hot reload: 2-5 seconds
- Production build: 15-30 minutes
- CI/CD pipeline: 20-40 minutes

Developer Impact:
- Slow feedback loops
- Reduced productivity
- Longer deployment times
- Higher infrastructure costs
```

---

#### 1. Use Faster Build Tools

**Modern Alternatives:**

**Option A: Vite (Development):**
```javascript
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  
  // Optimize dependency pre-bundling
  optimizeDeps: {
    include: ['react', 'react-dom', 'react-router-dom'],
    exclude: ['some-large-package'],
  },
  
  server: {
    // Faster HMR
    hmr: {
      overlay: false, // Disable error overlay for speed
    },
  },
});

// Result: Cold start <1s, HMR <100ms
```

**Option B: ESBuild (Bundling):**
```javascript
// build.js
const esbuild = require('esbuild');

esbuild.build({
  entryPoints: ['src/index.tsx'],
  bundle: true,
  minify: true,
  sourcemap: true,
  target: ['es2020'],
  outfile: 'dist/bundle.js',
});

// Result: 10-100x faster than Webpack
```

**Option C: SWC (Transpilation):**
```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.(js|jsx|ts|tsx)$/,
        use: {
          loader: 'swc-loader', // Instead of babel-loader
        },
      },
    ],
  },
};

// Result: 20x faster transpilation
```

---

#### 2. Webpack Optimization

**A. Enable Persistent Caching:**
```javascript
// webpack.config.js (Webpack 5)
module.exports = {
  cache: {
    type: 'filesystem',
    buildDependencies: {
      config: [__filename],
    },
    // Cache invalidation
    version: '1.0.0',
    // Cache location
    cacheDirectory: path.resolve(__dirname, '.webpack-cache'),
    // Cache compression
    compression: 'gzip',
  },
};

// First build: 120s
// Cached build: 8s (15x faster)
```

**B. Optimize Loaders:**
```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(js|jsx|ts|tsx)$/,
        // Exclude node_modules
        exclude: /node_modules/,
        use: {
          loader: 'swc-loader',
          options: {
            // Cache loader results
            cacheDirectory: true,
          },
        },
      },
    ],
  },
};
```

**C. Reduce Resolution Cost:**
```javascript
module.exports = {
  resolve: {
    // Only resolve these extensions
    extensions: ['.js', '.jsx', '.ts', '.tsx'],
    
    // Faster module resolution
    modules: [
      path.resolve(__dirname, 'src'),
      'node_modules',
    ],
    
    // Symlinks handling
    symlinks: false,
    
    // Aliases for faster resolution
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@components': path.resolve(__dirname, 'src/components'),
    },
  },
};
```

**D. Optimize Source Maps:**
```javascript
module.exports = {
  mode: 'development',
  
  // Development: faster but less detailed
  devtool: 'eval-cheap-module-source-map',
  
  // Production: skip or use simpler maps
  // devtool: false, // No source maps
  // devtool: 'source-map', // Full source maps (slower)
};
```

**E. Parallel Processing:**
```javascript
const TerserPlugin = require('terser-webpack-plugin');

module.exports = {
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin({
        parallel: true, // Use all CPU cores
        terserOptions: {
          compress: {
            drop_console: true,
          },
        },
      }),
    ],
  },
};
```

**F. Reduce Bundle Size:**
```javascript
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          priority: 10,
        },
      },
    },
    // Runtime chunk for better caching
    runtimeChunk: 'single',
  },
};
```

---

#### 3. Code-Level Optimizations

**A. Lazy Loading:**
```typescript
import { lazy, Suspense } from 'react';

// Don't load until needed
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Admin = lazy(() => import('./pages/Admin'));
const Settings = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/admin" element={<Admin />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}

// Reduces initial bundle size
// Faster initial build
```

**B. Dynamic Imports:**
```typescript
// Only load when needed
const loadHeavyLibrary = async () => {
  const module = await import('heavy-library');
  return module.default;
};

// Use with user interaction
const handleClick = async () => {
  const lib = await loadHeavyLibrary();
  lib.doSomething();
};
```

**C. Remove Unused Dependencies:**
```bash
# Analyze bundle
npm install --save-dev webpack-bundle-analyzer

# Find unused dependencies
npx depcheck

# Remove unused packages
npm uninstall unused-package
```

**D. Tree Shaking:**
```javascript
// Use named imports for better tree shaking
import { Button, Input } from '@mui/material';

// Instead of
import * as MUI from '@mui/material';
```

**E. Optimize Large Libraries:**
```javascript
// Before: Import entire lodash (70KB)
import _ from 'lodash';

// After: Import specific functions (5KB)
import debounce from 'lodash/debounce';
import throttle from 'lodash/throttle';

// Or use lodash-es for better tree shaking
import { debounce, throttle } from 'lodash-es';
```

---

#### 4. TypeScript Optimization

**A. Incremental Compilation:**
```json
// tsconfig.json
{
  "compilerOptions": {
    "incremental": true,
    "tsBuildInfoFile": ".tsbuildinfo",
    
    // Skip type checking for dependencies
    "skipLibCheck": true,
    
    // Faster module resolution
    "moduleResolution": "node",
    
    // Don't check all files
    "noEmitOnError": false
  },
  
  // Only include source files
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "build"]
}
```

**B. Separate Type Checking:**
```javascript
// webpack.config.js
const ForkTsCheckerWebpackPlugin = require('fork-ts-checker-webpack-plugin');

module.exports = {
  plugins: [
    // Run type checking in separate process
    new ForkTsCheckerWebpackPlugin({
      async: true, // Don't block compilation
      typescript: {
        configFile: 'tsconfig.json',
      },
    }),
  ],
};
```

**C. Use SWC Instead of tsc:**
```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'swc-loader', // 20x faster than ts-loader
        exclude: /node_modules/,
      },
    ],
  },
};

// Run type checking separately
// package.json
{
  "scripts": {
    "typecheck": "tsc --noEmit",
    "build": "webpack"
  }
}
```

---

#### 5. Development Server Optimization

**A. Webpack Dev Server:**
```javascript
module.exports = {
  devServer: {
    // Hot reload
    hot: true,
    
    // Lazy compilation (Webpack 5.17+)
    lazy: false,
    
    // Faster rebuilds
    watchOptions: {
      ignored: /node_modules/,
      aggregateTimeout: 300,
      poll: false, // Don't use polling
    },
    
    // Faster serving
    compress: true,
    
    // Don't open browser
    open: false,
  },
};
```

**B. Vite Dev Server:**
```javascript
export default defineConfig({
  server: {
    // Pre-bundle dependencies
    warmup: {
      clientFiles: ['./src/App.tsx', './src/main.tsx'],
    },
    
    // Faster HMR
    hmr: {
      overlay: false,
    },
  },
});
```

---

#### 6. Production Build Optimization

**A. Multi-Core Compilation:**
```javascript
const TerserPlugin = require('terser-webpack-plugin');
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');

module.exports = {
  optimization: {
    minimizer: [
      new TerserPlugin({
        parallel: true, // Use all cores
        extractComments: false,
      }),
      new CssMinimizerPlugin({
        parallel: true,
      }),
    ],
  },
};
```

**B. Disable Unnecessary Features:**
```javascript
module.exports = {
  performance: {
    hints: false, // Disable performance hints in prod
  },
  
  stats: {
    // Minimal stats output
    preset: 'minimal',
    moduleTrace: false,
    errorDetails: false,
  },
};
```

**C. Analyze and Optimize:**
```javascript
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: 'static',
      openAnalyzer: false,
      reportFilename: 'bundle-report.html',
    }),
  ],
};
```

---

#### 7. CI/CD Optimization

**A. GitHub Actions:**
```yaml
name: Build

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      # Cache dependencies
      - uses: actions/setup-node@v3
        with:
          node-version: 18
          cache: 'npm'
      
      # Cache build artifacts
      - uses: actions/cache@v3
        with:
          path: |
            .webpack-cache
            node_modules/.cache
          key: ${{ runner.os }}-build-${{ hashFiles('**/package-lock.json') }}
      
      - run: npm ci
      - run: npm run build
```

**B. Parallel Jobs:**
```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: npm test
  
  typecheck:
    runs-on: ubuntu-latest
    steps:
      - run: npm run typecheck
  
  build:
    needs: [test, typecheck]
    runs-on: ubuntu-latest
    steps:
      - run: npm run build
```

**C. Docker Layer Caching:**
```dockerfile
# Dockerfile
FROM node:18-alpine

WORKDIR /app

# Copy only package files first (layer caching)
COPY package*.json ./
RUN npm ci

# Then copy source
COPY . .

RUN npm run build
```

---

#### 8. Module Federation (Webpack 5)

**Split App into Micro-frontends:**
```javascript
// host/webpack.config.js
const ModuleFederationPlugin = require('webpack/lib/container/ModuleFederationPlugin');

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'host',
      remotes: {
        dashboard: 'dashboard@http://localhost:3001/remoteEntry.js',
        admin: 'admin@http://localhost:3002/remoteEntry.js',
      },
      shared: {
        react: { singleton: true },
        'react-dom': { singleton: true },
      },
    }),
  ],
};

// Each micro-frontend builds independently
// Faster parallel builds
// Smaller bundles
```

---

#### 9. Monitoring and Profiling

**A. Speed Measure Plugin:**
```javascript
const SpeedMeasurePlugin = require('speed-measure-webpack-plugin');
const smp = new SpeedMeasurePlugin();

module.exports = smp.wrap({
  // Your webpack config
});

// Output shows time for each loader/plugin
```

**B. Webpack Bundle Analyzer:**
```bash
npm install --save-dev webpack-bundle-analyzer
npm run build
# Open report to see what's taking space
```

**C. Build Time Tracking:**
```javascript
const webpack = require('webpack');
const config = require('./webpack.config');

const startTime = Date.now();

webpack(config, (err, stats) => {
  const buildTime = Date.now() - startTime;
  console.log(`Build completed in ${buildTime}ms`);
});
```

---

#### 10. Complete Optimization Example

**Before Optimization:**
```javascript
// webpack.config.js (slow)
module.exports = {
  mode: 'development',
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
  },
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        use: 'babel-loader', // Slow
      },
      {
        test: /\.tsx?$/,
        use: 'ts-loader', // Slow
      },
    ],
  },
  devtool: 'source-map', // Slow
};

// Build time: 120 seconds
// Incremental: 15 seconds
```

**After Optimization:**
```javascript
// webpack.config.js (fast)
const path = require('path');
const ForkTsCheckerWebpackPlugin = require('fork-ts-checker-webpack-plugin');

module.exports = {
  mode: 'development',
  entry: './src/index.tsx',
  
  output: {
    filename: '[name].bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
  
  // Persistent caching
  cache: {
    type: 'filesystem',
    buildDependencies: {
      config: [__filename],
    },
  },
  
  module: {
    rules: [
      {
        test: /\.(js|jsx|ts|tsx)$/,
        exclude: /node_modules/,
        use: {
          loader: 'swc-loader', // 20x faster
          options: {
            jsc: {
              parser: {
                syntax: 'typescript',
                tsx: true,
              },
            },
          },
        },
      },
    ],
  },
  
  resolve: {
    extensions: ['.tsx', '.ts', '.jsx', '.js'],
    alias: {
      '@': path.resolve(__dirname, 'src'),
    },
  },
  
  // Faster source maps
  devtool: 'eval-cheap-module-source-map',
  
  optimization: {
    splitChunks: {
      chunks: 'all',
    },
    runtimeChunk: 'single',
  },
  
  plugins: [
    // Async type checking
    new ForkTsCheckerWebpackPlugin({
      async: true,
    }),
  ],
  
  devServer: {
    hot: true,
    watchOptions: {
      ignored: /node_modules/,
    },
  },
};

// Build time: 8 seconds (15x faster)
// Incremental: 0.5 seconds (30x faster)
```

---

#### Results Summary

**Typical Improvements:**
```
Optimization              | Before  | After   | Improvement
--------------------------|---------|---------|------------
Cold build                | 120s    | 8s      | 15x
Incremental build         | 15s     | 0.5s    | 30x
Hot reload                | 3s      | 0.1s    | 30x
Production build          | 480s    | 45s     | 10x
CI/CD pipeline            | 900s    | 120s    | 7.5x
```

---

#### Best Practices Checklist

**✅ Tooling:**
- Use modern tools (Vite, ESBuild, SWC)
- Enable persistent caching
- Parallelize where possible

**✅ Configuration:**
- Exclude node_modules from loaders
- Use faster source maps in dev
- Optimize resolve configuration

**✅ Code:**
- Lazy load routes and heavy components
- Remove unused dependencies
- Use tree-shakeable imports

**✅ TypeScript:**
- Enable incremental compilation
- Separate type checking from transpilation
- Use skipLibCheck

**✅ CI/CD:**
- Cache dependencies and build artifacts
- Parallelize jobs
- Use Docker layer caching

**✅ Monitoring:**
- Measure build times
- Analyze bundle sizes
- Profile slow parts

---

#### Interview Tips

✅ **Key Points:**
- Use modern tools (Vite/ESBuild/SWC) for massive speedups
- Enable caching (persistent cache in Webpack 5)
- Parallelize compilation and minification
- Optimize TypeScript with incremental builds
- Lazy load routes and components
- Remove unused dependencies

✅ **When to Mention:**
- Performance optimization discussions
- Large-scale application development
- Developer experience improvements
- CI/CD optimization
- Build tool selection

✅ **Common Follow-ups:**
- "What's the biggest win for build time optimization?"
- "How do you profile slow builds?"
- "Should you use Webpack or Vite?"
- "How do you optimize CI/CD builds?"
- "What about TypeScript compilation?"

✅ **Perfect Answer Structure:**
1. Problem: Slow builds hurt productivity
2. Modern Tools: Vite/ESBuild/SWC (10-100x faster)
3. Caching: Persistent filesystem cache (Webpack 5)
4. Code: Lazy loading, tree shaking, remove unused deps
5. TypeScript: Incremental, separate type checking
6. CI/CD: Cache dependencies, parallelize
7. Results: From minutes to seconds

</details>

---

### 178. What is Rspack and how does it compare to Webpack?

<details>
<summary>View Answer</summary>

**Rspack** is a high-performance JavaScript bundler written in Rust, designed as a drop-in replacement for Webpack with 10x faster build speeds while maintaining compatibility with the Webpack ecosystem (loaders, plugins, configuration).

#### What is Rspack?

**Key Characteristics:**
- Written in Rust (not JavaScript)
- Webpack-compatible API
- 10x faster than Webpack 5
- Supports most Webpack loaders and plugins
- Created by ByteDance (TikTok's parent company)
- Production-ready
- Active development

**Why Rspack Exists:**
```
Problem:
- Webpack is slow for large projects
- ESBuild lacks Webpack compatibility
- Vite changes dev/prod architecture
- Turbopack is Next.js specific

Solution:
- Rspack = Webpack API + Rust performance
- Drop-in replacement
- Minimal migration effort
```

---

#### Performance Comparison

**Build Speed (Large Project - 3000 modules):**
```
Tool          | Cold Build | Hot Reload | Production
--------------|------------|------------|------------
Webpack 5     | 32s        | 2.1s       | 180s
Rspack        | 3.2s       | 0.2s       | 18s
Vite          | 1.8s       | 0.08s      | 22s
Turbopack     | 2.1s       | 0.1s       | N/A (beta)

Rspack Speedup vs Webpack:
- Cold build: 10x faster
- Hot reload: 10x faster  
- Production: 10x faster
```

**Real-World Project (ByteDance Internal):**
```
Metric              | Webpack | Rspack | Improvement
--------------------|---------|--------|------------
Dev Server Start    | 120s    | 8s     | 15x faster
HMR Update          | 3.2s    | 0.3s   | 10x faster
Production Build    | 420s    | 35s    | 12x faster
Memory Usage        | 2.5GB   | 800MB  | 3x less
```

---

#### Installation and Setup

**Install Rspack:**
```bash
# Create new project
npm create rspack@latest my-app

# Or add to existing project
npm install -D @rspack/core @rspack/cli

# For webpack compatibility layer
npm install -D @rspack/plugin-react-refresh
```

**Basic Configuration:**
```javascript
// rspack.config.js
const rspack = require('@rspack/core');

module.exports = {
  entry: './src/index.jsx',
  
  output: {
    filename: '[name].[contenthash].js',
    path: __dirname + '/dist',
  },
  
  module: {
    rules: [
      {
        test: /\.jsx?$/,
        use: {
          loader: 'builtin:swc-loader',
          options: {
            jsc: {
              parser: {
                syntax: 'ecmascript',
                jsx: true,
              },
            },
          },
        },
        type: 'javascript/auto',
      },
      {
        test: /\.css$/,
        use: [
          { loader: 'builtin:style-loader' },
          { loader: 'builtin:css-loader' },
        ],
        type: 'javascript/auto',
      },
    ],
  },
  
  plugins: [
    new rspack.HtmlRspackPlugin({
      template: './public/index.html',
    }),
  ],
  
  devServer: {
    port: 3000,
    hot: true,
  },
};
```

---

#### React Setup

**Complete React Configuration:**
```javascript
// rspack.config.js
const rspack = require('@rspack/core');
const RefreshPlugin = require('@rspack/plugin-react-refresh');

const isDev = process.env.NODE_ENV === 'development';

module.exports = {
  entry: './src/index.tsx',
  
  output: {
    filename: '[name].[contenthash].js',
    path: __dirname + '/dist',
    clean: true,
  },
  
  resolve: {
    extensions: ['.tsx', '.ts', '.jsx', '.js'],
    alias: {
      '@': __dirname + '/src',
    },
  },
  
  module: {
    rules: [
      {
        test: /\.(jsx?|tsx?)$/,
        use: {
          loader: 'builtin:swc-loader',
          options: {
            jsc: {
              parser: {
                syntax: 'typescript',
                tsx: true,
              },
              transform: {
                react: {
                  runtime: 'automatic',
                  development: isDev,
                  refresh: isDev,
                },
              },
            },
          },
        },
      },
      {
        test: /\.css$/,
        use: [
          { loader: 'builtin:style-loader' },
          { loader: 'builtin:css-loader' },
        ],
        type: 'javascript/auto',
      },
      {
        test: /\.module\.css$/,
        use: [
          { loader: 'builtin:style-loader' },
          {
            loader: 'builtin:css-loader',
            options: {
              modules: true,
            },
          },
        ],
        type: 'javascript/auto',
      },
    ],
  },
  
  plugins: [
    new rspack.HtmlRspackPlugin({
      template: './public/index.html',
    }),
    isDev && new RefreshPlugin(),
    new rspack.DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV),
    }),
  ].filter(Boolean),
  
  optimization: {
    minimize: !isDev,
    splitChunks: {
      chunks: 'all',
    },
  },
  
  devServer: {
    port: 3000,
    hot: true,
    historyApiFallback: true,
  },
};
```

**package.json:**
```json
{
  "scripts": {
    "dev": "rspack serve",
    "build": "rspack build",
    "preview": "rspack serve --mode production"
  },
  "devDependencies": {
    "@rspack/cli": "latest",
    "@rspack/core": "latest",
    "@rspack/plugin-react-refresh": "latest"
  },
  "dependencies": {
    "react": "^18.0.0",
    "react-dom": "^18.0.0"
  }
}
```

---

#### Migration from Webpack

**Step 1: Install Rspack:**
```bash
npm install -D @rspack/core @rspack/cli
```

**Step 2: Rename Config:**
```bash
# Rename webpack.config.js to rspack.config.js
mv webpack.config.js rspack.config.js
```

**Step 3: Update Imports:**
```javascript
// Before (Webpack)
const webpack = require('webpack');
const HtmlWebpackPlugin = require('html-webpack-plugin');

// After (Rspack)
const rspack = require('@rspack/core');
const HtmlWebpackPlugin = rspack.HtmlRspackPlugin;
```

**Step 4: Update Loaders:**
```javascript
// Webpack config
module.exports = {
  module: {
    rules: [
      {
        test: /\.jsx?$/,
        use: 'babel-loader',
      },
    ],
  },
};

// Rspack config (use built-in loaders)
module.exports = {
  module: {
    rules: [
      {
        test: /\.jsx?$/,
        use: {
          loader: 'builtin:swc-loader',
          options: {
            jsc: {
              parser: {
                syntax: 'ecmascript',
                jsx: true,
              },
            },
          },
        },
      },
    ],
  },
};
```

**Step 5: Update Scripts:**
```json
// package.json
{
  "scripts": {
    "dev": "rspack serve",
    "build": "rspack build"
  }
}
```

---

#### Webpack Compatibility

**What Works:**

**✅ Supported Features:**
- Entry/Output configuration
- Module rules (loaders)
- Resolve (aliases, extensions)
- Plugins (most)
- Optimization (splitChunks, minimize)
- DevServer
- Source maps
- Code splitting
- Tree shaking
- Hot Module Replacement

**✅ Supported Loaders:**
```javascript
// Most webpack loaders work
module.exports = {
  module: {
    rules: [
      { test: /\.css$/, use: 'css-loader' },
      { test: /\.scss$/, use: ['css-loader', 'sass-loader'] },
      { test: /\.less$/, use: ['css-loader', 'less-loader'] },
      { test: /\.(png|jpg|gif)$/, type: 'asset/resource' },
    ],
  },
};
```

**✅ Supported Plugins:**
- HtmlWebpackPlugin → HtmlRspackPlugin
- DefinePlugin
- ProvidePlugin
- MiniCssExtractPlugin (built-in)
- Copy Plugin
- And many more

**What Doesn't Work (Yet):**

**❌ Limited Support:**
- Some advanced Webpack plugins
- Custom webpack loaders (need conversion)
- Module Federation (in development)
- Some edge cases

---

#### Built-in Features

**1. Built-in SWC Loader:**
```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(js|jsx|ts|tsx)$/,
        use: {
          loader: 'builtin:swc-loader',
          options: {
            jsc: {
              parser: {
                syntax: 'typescript',
                tsx: true,
              },
            },
          },
        },
      },
    ],
  },
};
```

**2. Built-in CSS Loaders:**
```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          { loader: 'builtin:style-loader' },
          { loader: 'builtin:css-loader' },
        ],
        type: 'javascript/auto',
      },
    ],
  },
};
```

**3. Built-in Minification:**
```javascript
module.exports = {
  optimization: {
    minimize: true,
    // Uses SWC minifier (faster than Terser)
  },
};
```

---

#### Rspack vs Webpack Detailed

| Feature | Webpack 5 | Rspack |
|---------|-----------|--------|
| **Language** | JavaScript | Rust |
| **Speed** | Baseline | 10x faster |
| **Memory** | High | 3x lower |
| **Configuration** | Same | Same (compatible) |
| **Loaders** | All | Most (95%+) |
| **Plugins** | All | Most (90%+) |
| **Module Federation** | Yes | In development |
| **Tree Shaking** | Excellent | Excellent |
| **Code Splitting** | Advanced | Advanced |
| **HMR** | Good | Excellent |
| **Source Maps** | Yes | Yes |
| **Maturity** | Very mature | Production-ready |
| **Ecosystem** | Huge | Growing |
| **Learning Curve** | Steep | Same as Webpack |
| **Migration** | N/A | Easy from Webpack |

---

#### Rspack vs Other Tools

**Rspack vs Vite:**
```
Feature              | Vite          | Rspack
---------------------|---------------|------------------
Dev Architecture     | Native ESM    | Bundling
Prod Architecture    | Rollup        | Rspack
Consistency          | Dev ≠ Prod    | Dev = Prod
Webpack Compatible   | No            | Yes
Speed (Dev)          | Faster        | Fast
Speed (Prod)         | Fast          | Faster
Large Projects       | Slower        | Fast
Migration Effort     | Medium        | Easy (from Webpack)
```

**Rspack vs Turbopack:**
```
Feature              | Turbopack     | Rspack
---------------------|---------------|------------------
Language             | Rust          | Rust
Speed                | Very Fast     | Very Fast
Webpack Compatible   | Partial       | High
Framework            | Next.js only  | Framework agnostic
Production Ready     | No (beta)     | Yes
Standalone           | No            | Yes
```

**Rspack vs ESBuild:**
```
Feature              | ESBuild       | Rspack
---------------------|---------------|------------------
Language             | Go            | Rust
Speed                | Very Fast     | Fast
Webpack Compatible   | No            | Yes
Bundling             | Simple        | Advanced
Code Splitting       | Basic         | Advanced
Loaders              | Limited       | Webpack loaders
Plugins              | Limited       | Webpack plugins
```

---

#### Use Cases

**✅ Perfect For:**

1. **Large Webpack Projects:**
```
- Slow Webpack builds
- Want speed without rewrite
- Minimal migration effort
- Keep existing config
```

2. **Enterprise Applications:**
```
- Complex build requirements
- Need Webpack plugin ecosystem
- Production stability required
- Large teams
```

3. **Migration Path:**
```
- Moving from Webpack
- Don't want to change architecture
- Need compatibility
```

**⚠️ Consider Alternatives:**

1. **New Projects:**
   - Vite might be simpler
   - Less configuration

2. **Small Projects:**
   - Overhead not worth it
   - Vite/ESBuild sufficient

3. **Next.js Projects:**
   - Use Turbopack instead
   - Better integration

---

#### Limitations

**❌ Not Yet Supported:**
```
- Module Federation (coming soon)
- Some advanced webpack plugins
- 100% plugin compatibility
- Some edge cases
```

**❌ Beta Status for Some Features:**
```
- Still evolving
- Breaking changes possible
- Documentation incomplete
```

---

#### Best Practices

**✅ Use Built-in Loaders:**
```javascript
// Prefer built-in loaders (faster)
use: 'builtin:swc-loader',

// Instead of
use: 'babel-loader',
```

**✅ Enable Caching:**
```javascript
module.exports = {
  cache: true, // Or configure filesystem cache
};
```

**✅ Optimize for Production:**
```javascript
module.exports = {
  mode: 'production',
  optimization: {
    minimize: true,
    splitChunks: {
      chunks: 'all',
    },
  },
};
```

**✅ Monitor Performance:**
```javascript
// Track build times
const start = Date.now();
// ... build
const duration = Date.now() - start;
console.log(`Build took ${duration}ms`);
```

---

#### Interview Tips

✅ **Key Points:**
- Rust-based bundler, Webpack-compatible
- 10x faster than Webpack
- Drop-in replacement with minimal migration
- Production-ready, used by ByteDance
- Maintains Webpack API and ecosystem compatibility
- Built-in SWC for transpilation

✅ **When to Mention:**
- Webpack performance issues
- Build tool modernization
- Migration strategies
- Large project optimization
- Rust in frontend tooling

✅ **Common Follow-ups:**
- "How is Rspack different from Webpack?"
- "Why not use Vite instead?"
- "Is it production-ready?"
- "How easy is migration from Webpack?"
- "What about Turbopack vs Rspack?"

✅ **Perfect Answer Structure:**
1. Define: Rust-based Webpack replacement
2. Speed: 10x faster while keeping compatibility
3. Migration: Easy from Webpack, similar API
4. Compatibility: 90%+ of loaders/plugins work
5. Production: Ready, used by ByteDance
6. vs Others: Webpack compat vs Vite's speed, Framework-agnostic vs Turbopack
7. Use case: Large Webpack projects wanting speed

</details>

---

### 179. What is the role of Babel in modern React projects?

<details>
<summary>View Answer</summary>

**Babel** is a JavaScript compiler that transforms modern JavaScript (ES6+) and JSX into backwards-compatible JavaScript that can run in older browsers. In modern React projects, Babel's role has evolved with the rise of faster alternatives like SWC and ESBuild, but it remains important for advanced transformations and plugin ecosystem.

#### Core Functions of Babel

**1. JSX Transformation:**
```jsx
// Input (JSX)
const element = (
  <div className="container">
    <h1>Hello, {name}!</h1>
  </div>
);

// Output (JavaScript)
import { jsx as _jsx } from 'react/jsx-runtime';
const element = _jsx('div', {
  className: 'container',
  children: _jsx('h1', {
    children: ['Hello, ', name, '!']
  })
});
```

**2. ES6+ to ES5:**
```javascript
// Input (ES6+)
const greeting = (name) => `Hello, ${name}!`;
const { user, ...rest } = data;
const promise = async () => await fetch('/api');

// Output (ES5)
var greeting = function(name) {
  return 'Hello, ' + name + '!';
};
var user = data.user;
var rest = _objectWithoutProperties(data, ['user']);
var promise = function() {
  return regeneratorRuntime.async(function(_context) {
    return _context.abrupt('return', fetch('/api'));
  });
};
```

---

#### Babel Configuration for React

**Basic Setup:**
```bash
npm install --save-dev @babel/core @babel/preset-react @babel/preset-env
```

**.babelrc (Classic):**
```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "targets": {
          "browsers": [">0.25%", "not dead"]
        },
        "useBuiltIns": "usage",
        "corejs": 3
      }
    ],
    [
      "@babel/preset-react",
      {
        "runtime": "automatic"
      }
    ]
  ]
}
```

**babel.config.js (Modern):**
```javascript
module.exports = {
  presets: [
    [
      '@babel/preset-env',
      {
        // Only polyfill what's needed
        useBuiltIns: 'usage',
        corejs: 3,
        
        // Target environments
        targets: {
          browsers: [
            'last 2 Chrome versions',
            'last 2 Firefox versions',
            'last 2 Safari versions',
            'last 2 Edge versions',
          ],
        },
        
        // ES modules (for tree shaking)
        modules: false,
      },
    ],
    [
      '@babel/preset-react',
      {
        // New JSX transform (React 17+)
        runtime: 'automatic',
        
        // Development mode features
        development: process.env.NODE_ENV === 'development',
        
        // Import source for debugging
        importSource: '@emotion/react', // If using Emotion
      },
    ],
    '@babel/preset-typescript', // If using TypeScript
  ],
  
  plugins: [
    // Additional transformations
  ],
};
```

---

#### Key Presets for React

**1. @babel/preset-react:**
```json
{
  "presets": [
    [
      "@babel/preset-react",
      {
        "runtime": "automatic",
        "development": true,
        "importSource": "react"
      }
    ]
  ]
}
```

**Options:**
- `runtime: "automatic"` - New JSX transform (no need to import React)
- `runtime: "classic"` - Old transform (requires `import React`)
- `development: true` - Adds `__source` and `__self` for debugging
- `importSource` - Custom JSX pragma source

**2. @babel/preset-env:**
```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "targets": "defaults",
        "useBuiltIns": "usage",
        "corejs": 3
      }
    ]
  ]
}
```

**Options:**
- `targets` - Browser targets (determines what to transpile)
- `useBuiltIns` - How to handle polyfills (`usage`, `entry`, `false`)
- `corejs` - Version of core-js for polyfills

**3. @babel/preset-typescript:**
```json
{
  "presets": [
    [
      "@babel/preset-typescript",
      {
        "isTSX": true,
        "allExtensions": true
      }
    ]
  ]
}
```

---

#### Common Babel Plugins for React

**1. Transform Runtime (Avoid Code Duplication):**
```bash
npm install --save @babel/runtime
npm install --save-dev @babel/plugin-transform-runtime
```

```json
{
  "plugins": [
    [
      "@babel/plugin-transform-runtime",
      {
        "regenerator": true,
        "corejs": 3
      }
    ]
  ]
}
```

**2. Class Properties:**
```javascript
// Input
class MyComponent extends React.Component {
  state = { count: 0 };
  
  handleClick = () => {
    this.setState({ count: this.state.count + 1 });
  };
}

// Babel transforms class fields
```

```json
{
  "plugins": ["@babel/plugin-proposal-class-properties"]
}
```

**3. Optional Chaining & Nullish Coalescing:**
```javascript
// Input
const value = user?.profile?.name ?? 'Anonymous';

// Babel transforms to safe checks
```

```json
{
  "plugins": [
    "@babel/plugin-proposal-optional-chaining",
    "@babel/plugin-proposal-nullish-coalescing-operator"
  ]
}
```

**4. Styled Components:**
```bash
npm install --save-dev babel-plugin-styled-components
```

```json
{
  "plugins": [
    [
      "babel-plugin-styled-components",
      {
        "displayName": true,
        "fileName": true,
        "ssr": true
      }
    ]
  ]
}
```

**5. React Refresh (Fast Refresh):**
```bash
npm install --save-dev react-refresh @pmmmwh/react-refresh-webpack-plugin
```

```json
{
  "plugins": [
    "react-refresh/babel"
  ]
}
```

---

#### Integration with Build Tools

**Webpack Integration:**
```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.(js|jsx|ts|tsx)$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            // Cache transpilation results
            cacheDirectory: true,
            cacheCompression: false,
            
            // Babel config
            presets: [
              '@babel/preset-env',
              '@babel/preset-react',
              '@babel/preset-typescript',
            ],
          },
        },
      },
    ],
  },
};
```

**Vite (No Babel by Default):**
```javascript
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [
    react({
      // Use Babel for specific transformations
      babel: {
        plugins: [
          'babel-plugin-styled-components',
        ],
      },
    }),
  ],
});
```

**Next.js (Built-in Babel):**
```json
// .babelrc (customize Next.js Babel)
{
  "presets": ["next/babel"],
  "plugins": [
    "babel-plugin-styled-components"
  ]
}
```

---

#### Babel vs Modern Alternatives

**Babel vs SWC:**
```
Feature           | Babel        | SWC
------------------|--------------|------------------
Language          | JavaScript   | Rust
Speed             | Baseline     | 20x faster
Plugin Ecosystem  | Huge         | Growing
Custom Transforms | Extensive    | Limited
Production Ready  | Yes          | Yes
Type Checking     | No           | No
Use Case          | Complex      | Speed priority
```

**Example Migration:**
```javascript
// webpack.config.js

// Before (Babel)
module.exports = {
  module: {
    rules: [
      {
        test: /\.jsx?$/,
        use: 'babel-loader',
      },
    ],
  },
};

// After (SWC)
module.exports = {
  module: {
    rules: [
      {
        test: /\.jsx?$/,
        use: 'swc-loader',
      },
    ],
  },
};

// Build time: 45s → 2s (22x faster)
```

**Babel vs ESBuild:**
```
Feature           | Babel        | ESBuild
------------------|--------------|------------------
Language          | JavaScript   | Go
Speed             | Baseline     | 100x faster
Plugins           | Many         | Few
Minification      | Via plugin   | Built-in
Target Support    | Extensive    | Modern browsers
Use Case          | Full control | Speed + simplicity
```

---

#### When Babel is Still Necessary

**✅ Use Babel When:**

1. **Need Specific Plugins:**
```json
{
  "plugins": [
    "babel-plugin-styled-components",
    "babel-plugin-macros",
    "babel-plugin-transform-react-remove-prop-types",
    "@emotion/babel-plugin"
  ]
}

// SWC doesn't support all these yet
```

2. **Legacy Browser Support:**
```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "targets": {
          "ie": "11" // Need Babel for IE11
        }
      }
    ]
  ]
}
```

3. **Custom Transformations:**
```javascript
// Custom Babel plugin
module.exports = function() {
  return {
    visitor: {
      Identifier(path) {
        if (path.node.name === 'OLD_NAME') {
          path.node.name = 'NEW_NAME';
        }
      },
    },
  };
};
```

4. **Framework Requirements:**
```
- Emotion CSS-in-JS
- Styled Components with SSR
- Twin.macro
- React Native
```

---

#### When to Use Alternatives

**⚡ Use SWC When:**
- Speed is critical
- Standard React transformations
- Don't need complex plugins
- Modern browsers only

**⚡ Use ESBuild When:**
- Development builds
- Maximum speed needed
- Simple transpilation
- No legacy support required

**⚡ Use TypeScript Compiler When:**
- Pure TypeScript projects
- Need type checking
- No JSX transformations needed

---

#### Performance Comparison

**Large React Project (3000 files):**
```
Compiler    | Build Time | Incremental | Memory
------------|------------|-------------|--------
Babel       | 45s        | 3s          | 1.2GB
SWC         | 2s         | 0.2s        | 400MB
ESBuild     | 1s         | 0.1s        | 200MB
TypeScript  | 15s        | 2s          | 800MB

Babel Speed:
- SWC: 22x faster
- ESBuild: 45x faster
```

---

#### Modern React Setup Without Babel

**Vite (Default, No Babel):**
```javascript
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()], // Uses ESBuild by default
});

// Fast dev server + HMR
// No Babel unless explicitly added
```

**Next.js with SWC:**
```javascript
// next.config.js
module.exports = {
  // Use SWC instead of Babel (default in Next.js 12+)
  swcMinify: true,
};

// Remove .babelrc to use SWC
// 5x faster builds
```

**Create React App with CRACO:**
```bash
npm install @craco/craco @swc/core swc-loader
```

```javascript
// craco.config.js
module.exports = {
  webpack: {
    configure: (webpackConfig) => {
      // Replace babel-loader with swc-loader
      const babelLoader = webpackConfig.module.rules
        .find(rule => rule.oneOf)
        .oneOf.find(rule => rule.loader && rule.loader.includes('babel-loader'));
      
      babelLoader.loader = require.resolve('swc-loader');
      delete babelLoader.options;
      
      return webpackConfig;
    },
  },
};
```

---

#### Hybrid Approach (Best of Both Worlds)

**Use SWC for Transpilation, Babel for Plugins:**
```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.(js|jsx|ts|tsx)$/,
        exclude: /node_modules/,
        use: [
          // 1. Fast transpilation with SWC
          {
            loader: 'swc-loader',
            options: {
              jsc: {
                parser: {
                  syntax: 'typescript',
                  tsx: true,
                },
              },
            },
          },
          // 2. Specific transformations with Babel
          {
            loader: 'babel-loader',
            options: {
              plugins: [
                'babel-plugin-styled-components',
              ],
            },
          },
        ],
      },
    ],
  },
};
```

---

#### Complete Babel Setup Example

**package.json:**
```json
{
  "devDependencies": {
    "@babel/core": "^7.23.0",
    "@babel/preset-env": "^7.23.0",
    "@babel/preset-react": "^7.23.0",
    "@babel/preset-typescript": "^7.23.0",
    "@babel/plugin-transform-runtime": "^7.23.0",
    "babel-loader": "^9.1.0",
    "babel-plugin-styled-components": "^2.1.0"
  },
  "dependencies": {
    "@babel/runtime": "^7.23.0",
    "core-js": "^3.33.0"
  }
}
```

**babel.config.js:**
```javascript
module.exports = (api) => {
  // Cache based on NODE_ENV
  api.cache.using(() => process.env.NODE_ENV);
  
  const isDev = api.env('development');
  
  return {
    presets: [
      [
        '@babel/preset-env',
        {
          targets: {
            browsers: ['last 2 versions', '>1%', 'not dead'],
          },
          useBuiltIns: 'usage',
          corejs: 3,
          modules: false, // Keep ES modules for tree shaking
        },
      ],
      [
        '@babel/preset-react',
        {
          runtime: 'automatic',
          development: isDev,
        },
      ],
      [
        '@babel/preset-typescript',
        {
          isTSX: true,
          allExtensions: true,
        },
      ],
    ],
    
    plugins: [
      '@babel/plugin-transform-runtime',
      '@babel/plugin-proposal-class-properties',
      '@babel/plugin-proposal-optional-chaining',
      '@babel/plugin-proposal-nullish-coalescing-operator',
      [
        'babel-plugin-styled-components',
        {
          displayName: isDev,
          ssr: true,
        },
      ],
      isDev && 'react-refresh/babel',
    ].filter(Boolean),
  };
};
```

---

#### Debugging Babel

**See Transpilation Output:**
```bash
# Install Babel CLI
npm install --save-dev @babel/cli

# Transpile single file
npx babel src/App.jsx --out-file output.js

# See what Babel does
cat output.js
```

**Check Configuration:**
```javascript
// Add to webpack config
module.exports = {
  module: {
    rules: [
      {
        test: /\.jsx?$/,
        use: {
          loader: 'babel-loader',
          options: {
            // Debug Babel
            presets: [['@babel/preset-react', { development: true }]],
          },
        },
      },
    ],
  },
  
  // Log module processing
  stats: 'verbose',
};
```

---

#### Current State of Babel in React Ecosystem

**Declining Usage:**
```
Framework      | Default Compiler | Babel Option
---------------|------------------|-------------
Vite           | ESBuild          | Optional
Next.js 12+    | SWC              | Legacy
Create React App| Babel           | Default
Remix          | ESBuild          | Optional
Parcel         | SWC              | Optional
Rspack         | SWC              | Optional

Trend: Moving away from Babel for speed
```

**Still Common For:**
- Legacy CRA projects
- Custom transformations
- Specific plugins (Emotion, Styled Components)
- Library development
- IE11 support

---

#### Interview Tips

✅ **Key Points:**
- Babel compiles JSX and modern JS to browser-compatible code
- Critical presets: @babel/preset-env, @babel/preset-react
- Modern tools (SWC, ESBuild) are 20-100x faster
- Still needed for complex plugins and legacy support
- Most new projects use faster alternatives

✅ **When to Mention:**
- Build process discussions
- JSX transformation
- Browser compatibility
- Performance optimization
- Tool migration strategies

✅ **Common Follow-ups:**
- "Why is Babel slow?"
- "Should new projects use Babel?"
- "How does SWC compare to Babel?"
- "What are essential Babel plugins?"
- "Can you use Babel with Vite?"

✅ **Perfect Answer Structure:**
1. Role: Transpile JSX + modern JS to compatible code
2. Key Presets: preset-env (JS), preset-react (JSX)
3. Plugins: styled-components, transform-runtime, class properties
4. Modern Context: SWC/ESBuild are faster (20-100x)
5. Still Needed: Complex plugins, legacy browsers, custom transforms
6. Trend: Moving to faster alternatives, but Babel still common in legacy projects

</details>

---

### 180. How do you implement custom Webpack plugins?

<details>
<summary>View Answer</summary>

**Custom Webpack plugins** allow you to hook into the Webpack compilation process and perform custom operations. A plugin is a JavaScript object with an `apply` method that receives the compiler instance and can tap into various lifecycle hooks.

#### Understanding Webpack Plugin Architecture

**Plugin Lifecycle:**
```
1. Webpack starts
2. Plugins are instantiated
3. apply() method is called
4. Plugin taps into compiler/compilation hooks
5. Webpack runs (compilation happens)
6. Hooks are triggered at various stages
7. Plugin logic executes
8. Build completes
```

**Key Concepts:**
- **Compiler**: Represents the Webpack environment and configuration
- **Compilation**: Represents a single build of assets
- **Tapable**: Event system that powers Webpack hooks
- **Hooks**: Points in the build process where plugins can intervene

---

#### Basic Plugin Structure

**Simplest Plugin:**
```javascript
// HelloWorldPlugin.js
class HelloWorldPlugin {
  apply(compiler) {
    compiler.hooks.done.tap('HelloWorldPlugin', (stats) => {
      console.log('Hello World from Webpack!');
    });
  }
}

module.exports = HelloWorldPlugin;
```

**Usage:**
```javascript
// webpack.config.js
const HelloWorldPlugin = require('./HelloWorldPlugin');

module.exports = {
  plugins: [
    new HelloWorldPlugin(),
  ],
};
```

---

#### Plugin with Options

**Plugin with Configuration:**
```javascript
class FileListPlugin {
  constructor(options = {}) {
    this.options = {
      outputFile: 'filelist.md',
      ...options,
    };
  }
  
  apply(compiler) {
    // Hook into emit phase (before assets are written)
    compiler.hooks.emit.tapAsync(
      'FileListPlugin',
      (compilation, callback) => {
        // Get list of all files
        const fileList = Object.keys(compilation.assets)
          .map(filename => `- ${filename}`)
          .join('\n');
        
        const content = `# Build Files\n\n${fileList}`;
        
        // Add new asset to compilation
        compilation.assets[this.options.outputFile] = {
          source: () => content,
          size: () => content.length,
        };
        
        callback();
      }
    );
  }
}

module.exports = FileListPlugin;
```

**Usage:**
```javascript
module.exports = {
  plugins: [
    new FileListPlugin({
      outputFile: 'assets.md',
    }),
  ],
};
```

---

#### Common Webpack Hooks

**Compiler Hooks (Main Build Process):**
```javascript
class MyPlugin {
  apply(compiler) {
    // Before compilation starts
    compiler.hooks.beforeRun.tap('MyPlugin', (compiler) => {
      console.log('Build starting...');
    });
    
    // After compilation starts
    compiler.hooks.run.tap('MyPlugin', (compiler) => {
      console.log('Compilation started');
    });
    
    // Before emitting assets
    compiler.hooks.emit.tapAsync('MyPlugin', (compilation, callback) => {
      console.log('About to emit files...');
      callback();
    });
    
    // After assets are emitted
    compiler.hooks.afterEmit.tap('MyPlugin', (compilation) => {
      console.log('Files emitted');
    });
    
    // Build complete
    compiler.hooks.done.tap('MyPlugin', (stats) => {
      console.log('Build finished!');
      console.log('Time:', stats.endTime - stats.startTime, 'ms');
    });
  }
}
```

**Compilation Hooks (Individual Compilation):**
```javascript
class MyPlugin {
  apply(compiler) {
    compiler.hooks.compilation.tap('MyPlugin', (compilation) => {
      // Module processing
      compilation.hooks.buildModule.tap('MyPlugin', (module) => {
        console.log('Building module:', module.resource);
      });
      
      // After modules are built
      compilation.hooks.finishModules.tap('MyPlugin', (modules) => {
        console.log('Total modules:', modules.size);
      });
      
      // Optimizing assets
      compilation.hooks.optimizeAssets.tapAsync(
        'MyPlugin',
        (assets, callback) => {
          console.log('Optimizing', Object.keys(assets).length, 'assets');
          callback();
        }
      );
    });
  }
}
```

---

#### Tap Types

**Synchronous (tap):**
```javascript
compiler.hooks.done.tap('MyPlugin', (stats) => {
  // Synchronous callback
  console.log('Build done!');
  // No callback needed
});
```

**Asynchronous (tapAsync):**
```javascript
compiler.hooks.emit.tapAsync('MyPlugin', (compilation, callback) => {
  // Do async work
  setTimeout(() => {
    console.log('Async work done');
    callback(); // Must call callback when done
  }, 1000);
});
```

**Promise-based (tapPromise):**
```javascript
compiler.hooks.emit.tapPromise('MyPlugin', async (compilation) => {
  // Return a promise
  await someAsyncOperation();
  console.log('Async operation complete');
});
```

---

#### Real-World Example 1: Bundle Size Reporter

**Report Bundle Sizes:**
```javascript
class BundleSizeReporterPlugin {
  constructor(options = {}) {
    this.options = {
      minSize: 100 * 1024, // 100KB
      ...options,
    };
  }
  
  apply(compiler) {
    compiler.hooks.done.tap('BundleSizeReporterPlugin', (stats) => {
      const assets = stats.toJson().assets;
      
      console.log('\n📦 Bundle Size Report:');
      console.log('─'.repeat(50));
      
      assets
        .filter(asset => asset.size > this.options.minSize)
        .sort((a, b) => b.size - a.size)
        .forEach(asset => {
          const sizeKB = (asset.size / 1024).toFixed(2);
          const emoji = asset.size > 500 * 1024 ? '🔴' : '🟡';
          console.log(`${emoji} ${asset.name}: ${sizeKB} KB`);
        });
      
      console.log('─'.repeat(50) + '\n');
    });
  }
}

module.exports = BundleSizeReporterPlugin;
```

**Output:**
```
📦 Bundle Size Report:
──────────────────────────────────────────────────
🔴 main.bundle.js: 842.35 KB
🟡 vendor.bundle.js: 324.18 KB
🟡 styles.css: 156.42 KB
──────────────────────────────────────────────────
```

---

#### Real-World Example 2: Environment Variables Injector

**Inject Build Info:**
```javascript
class BuildInfoPlugin {
  apply(compiler) {
    compiler.hooks.compilation.tap('BuildInfoPlugin', (compilation) => {
      // Hook into processing assets
      compilation.hooks.processAssets.tap(
        {
          name: 'BuildInfoPlugin',
          stage: compilation.PROCESS_ASSETS_STAGE_ADDITIONS,
        },
        () => {
          const buildInfo = {
            buildTime: new Date().toISOString(),
            version: require('./package.json').version,
            commit: process.env.GIT_COMMIT || 'unknown',
            environment: process.env.NODE_ENV,
          };
          
          const content = `window.__BUILD_INFO__ = ${JSON.stringify(buildInfo, null, 2)};`;
          
          // Add build-info.js
          compilation.emitAsset(
            'build-info.js',
            new compiler.webpack.sources.RawSource(content)
          );
        }
      );
    });
  }
}

module.exports = BuildInfoPlugin;
```

**Generated build-info.js:**
```javascript
window.__BUILD_INFO__ = {
  "buildTime": "2025-12-26T10:30:45.123Z",
  "version": "1.2.3",
  "commit": "a1b2c3d",
  "environment": "production"
};
```

---

#### Real-World Example 3: Development Notifier

**Desktop Notifications:**
```javascript
const notifier = require('node-notifier');

class BuildNotifierPlugin {
  constructor(options = {}) {
    this.options = {
      title: 'Webpack Build',
      successSound: true,
      ...options,
    };
  }
  
  apply(compiler) {
    // Build started
    compiler.hooks.compile.tap('BuildNotifierPlugin', () => {
      this.startTime = Date.now();
    });
    
    // Build finished
    compiler.hooks.done.tap('BuildNotifierPlugin', (stats) => {
      const time = Date.now() - this.startTime;
      const hasErrors = stats.hasErrors();
      const hasWarnings = stats.hasWarnings();
      
      let message, icon;
      
      if (hasErrors) {
        message = '❌ Build failed!';
        icon = 'error';
      } else if (hasWarnings) {
        message = `⚠️ Build completed with warnings in ${time}ms`;
        icon = 'warning';
      } else {
        message = `✅ Build successful in ${time}ms`;
        icon = 'success';
      }
      
      notifier.notify({
        title: this.options.title,
        message: message,
        sound: this.options.successSound && !hasErrors,
        icon: icon,
      });
    });
  }
}

module.exports = BuildNotifierPlugin;
```

---

#### Real-World Example 4: Source Map Validator

**Validate Source Maps:**
```javascript
const { SourceMapConsumer } = require('source-map');

class SourceMapValidatorPlugin {
  apply(compiler) {
    compiler.hooks.emit.tapAsync(
      'SourceMapValidatorPlugin',
      async (compilation, callback) => {
        const mapAssets = Object.keys(compilation.assets)
          .filter(name => name.endsWith('.map'));
        
        console.log(`\n🗺️  Validating ${mapAssets.length} source maps...`);
        
        for (const mapFile of mapAssets) {
          try {
            const mapContent = compilation.assets[mapFile].source();
            const map = JSON.parse(mapContent);
            
            // Validate basic structure
            if (!map.version || !map.sources || !map.mappings) {
              console.error(`❌ Invalid source map: ${mapFile}`);
              continue;
            }
            
            // Test source map consumer
            await new SourceMapConsumer(map);
            console.log(`✅ Valid: ${mapFile}`);
            
          } catch (error) {
            console.error(`❌ Error in ${mapFile}:`, error.message);
          }
        }
        
        callback();
      }
    );
  }
}

module.exports = SourceMapValidatorPlugin;
```

---

#### Real-World Example 5: Performance Budget Plugin

**Enforce Size Limits:**
```javascript
class PerformanceBudgetPlugin {
  constructor(options = {}) {
    this.options = {
      maxAssetSize: 250 * 1024, // 250KB
      maxBundleSize: 500 * 1024, // 500KB
      ...options,
    };
  }
  
  apply(compiler) {
    compiler.hooks.afterEmit.tap('PerformanceBudgetPlugin', (compilation) => {
      const assets = Object.keys(compilation.assets).map(name => ({
        name,
        size: compilation.assets[name].size(),
      }));
      
      const violations = [];
      
      // Check individual assets
      assets.forEach(asset => {
        if (asset.size > this.options.maxAssetSize) {
          violations.push({
            file: asset.name,
            size: asset.size,
            limit: this.options.maxAssetSize,
            type: 'asset',
          });
        }
      });
      
      // Check total bundle size
      const totalSize = assets.reduce((sum, asset) => sum + asset.size, 0);
      if (totalSize > this.options.maxBundleSize) {
        violations.push({
          file: 'Total Bundle',
          size: totalSize,
          limit: this.options.maxBundleSize,
          type: 'bundle',
        });
      }
      
      // Report violations
      if (violations.length > 0) {
        console.error('\n🚨 Performance Budget Exceeded:');
        violations.forEach(v => {
          const sizeMB = (v.size / 1024 / 1024).toFixed(2);
          const limitMB = (v.limit / 1024 / 1024).toFixed(2);
          console.error(
            `   ${v.file}: ${sizeMB}MB (limit: ${limitMB}MB)`
          );
        });
        
        // Optionally fail the build
        if (this.options.failOnBudget) {
          throw new Error('Performance budget exceeded!');
        }
      }
    });
  }
}

module.exports = PerformanceBudgetPlugin;
```

---

#### Real-World Example 6: React Component Analyzer

**Analyze React Components:**
```javascript
const parser = require('@babel/parser');
const traverse = require('@babel/traverse').default;

class ReactComponentAnalyzerPlugin {
  apply(compiler) {
    compiler.hooks.compilation.tap(
      'ReactComponentAnalyzerPlugin',
      (compilation) => {
        const components = new Map();
        
        compilation.hooks.succeedModule.tap(
          'ReactComponentAnalyzerPlugin',
          (module) => {
            // Only analyze JS/JSX files
            if (!module.resource || !/\.(jsx?|tsx?)$/.test(module.resource)) {
              return;
            }
            
            try {
              const source = module.originalSource()?.source();
              if (!source) return;
              
              // Parse with Babel
              const ast = parser.parse(source, {
                sourceType: 'module',
                plugins: ['jsx', 'typescript'],
              });
              
              // Find components
              traverse(ast, {
                // Function components
                FunctionDeclaration(path) {
                  if (this.isReactComponent(path)) {
                    components.set(path.node.id.name, {
                      type: 'function',
                      file: module.resource,
                    });
                  }
                },
                
                // Class components
                ClassDeclaration(path) {
                  if (this.isReactComponent(path)) {
                    components.set(path.node.id.name, {
                      type: 'class',
                      file: module.resource,
                    });
                  }
                },
              });
              
            } catch (error) {
              // Ignore parse errors
            }
          }
        );
        
        compilation.hooks.afterProcessAssets.tap(
          'ReactComponentAnalyzerPlugin',
          () => {
            console.log(`\n⚛️  Found ${components.size} React components:`);
            console.log(`   Function components: ${[...components.values()].filter(c => c.type === 'function').length}`);
            console.log(`   Class components: ${[...components.values()].filter(c => c.type === 'class').length}`);
          }
        );
      }
    );
  }
  
  isReactComponent(path) {
    // Check if returns JSX or extends React.Component
    // Simplified check
    return true;
  }
}

module.exports = ReactComponentAnalyzerPlugin;
```

---

#### Accessing Compilation Assets

**Read Asset Content:**
```javascript
class AssetProcessorPlugin {
  apply(compiler) {
    compiler.hooks.emit.tap('AssetProcessorPlugin', (compilation) => {
      // Iterate through all assets
      Object.keys(compilation.assets).forEach(filename => {
        // Get asset
        const asset = compilation.assets[filename];
        
        // Read content
        const content = asset.source();
        const size = asset.size();
        
        console.log(`${filename}: ${size} bytes`);
      });
    });
  }
}
```

**Modify Asset Content:**
```javascript
class AssetModifierPlugin {
  apply(compiler) {
    compiler.hooks.emit.tap('AssetModifierPlugin', (compilation) => {
      Object.keys(compilation.assets).forEach(filename => {
        if (filename.endsWith('.js')) {
          const asset = compilation.assets[filename];
          let content = asset.source();
          
          // Add banner
          content = `/* Built at ${new Date().toISOString()} */\n${content}`;
          
          // Update asset
          compilation.assets[filename] = {
            source: () => content,
            size: () => content.length,
          };
        }
      });
    });
  }
}
```

**Add New Asset:**
```javascript
class AssetCreatorPlugin {
  apply(compiler) {
    compiler.hooks.emit.tap('AssetCreatorPlugin', (compilation) => {
      const content = JSON.stringify({
        version: '1.0.0',
        timestamp: Date.now(),
      });
      
      // Add new file to output
      compilation.assets['meta.json'] = {
        source: () => content,
        size: () => content.length,
      };
    });
  }
}
```

---

#### Testing Plugins

**Unit Test:**
```javascript
const webpack = require('webpack');
const MemoryFileSystem = require('memory-fs');
const MyPlugin = require('./MyPlugin');

describe('MyPlugin', () => {
  it('should create expected output', (done) => {
    const compiler = webpack({
      entry: './src/index.js',
      output: {
        path: '/dist',
        filename: 'bundle.js',
      },
      plugins: [new MyPlugin()],
    });
    
    // Use in-memory filesystem
    compiler.outputFileSystem = new MemoryFileSystem();
    
    compiler.run((err, stats) => {
      expect(err).toBeNull();
      expect(stats.hasErrors()).toBe(false);
      
      const output = compiler.outputFileSystem.readFileSync('/dist/bundle.js', 'utf8');
      expect(output).toContain('expected content');
      
      done();
    });
  });
});
```

---

#### Best Practices

**✅ Error Handling:**
```javascript
class SafePlugin {
  apply(compiler) {
    compiler.hooks.emit.tapAsync('SafePlugin', (compilation, callback) => {
      try {
        // Your logic
        callback();
      } catch (error) {
        // Report error
        compilation.errors.push(
          new Error(`SafePlugin: ${error.message}`)
        );
        callback();
      }
    });
  }
}
```

**✅ Performance:**
```javascript
class PerformantPlugin {
  apply(compiler) {
    // Cache expensive operations
    this.cache = new Map();
    
    compiler.hooks.emit.tap('PerformantPlugin', (compilation) => {
      // Use cache
      if (this.cache.has(compilation.hash)) {
        return this.cache.get(compilation.hash);
      }
      
      // Do work
      const result = expensiveOperation();
      this.cache.set(compilation.hash, result);
    });
  }
}
```

**✅ Logging:**
```javascript
class LoggingPlugin {
  apply(compiler) {
    const logger = compiler.getInfrastructureLogger('LoggingPlugin');
    
    compiler.hooks.compile.tap('LoggingPlugin', () => {
      logger.info('Compilation started');
    });
    
    compiler.hooks.done.tap('LoggingPlugin', (stats) => {
      logger.info(`Compilation finished in ${stats.endTime - stats.startTime}ms`);
    });
  }
}
```

---

#### Interview Tips

✅ **Key Points:**
- Plugin = class with `apply(compiler)` method
- Hook into compiler/compilation lifecycle events
- Use tap/tapAsync/tapPromise based on needs
- Common hooks: emit, done, compilation
- Can read, modify, or create assets

✅ **When to Mention:**
- Build process customization
- Webpack extensibility
- Custom build requirements
- Asset manipulation
- Development tooling

✅ **Common Follow-ups:**
- "What's the difference between compiler and compilation?"
- "When would you use tapAsync vs tapPromise?"
- "How do you debug webpack plugins?"
- "What are common use cases for custom plugins?"
- "How do plugins differ from loaders?"

✅ **Perfect Answer Structure:**
1. Definition: Class with apply() method
2. Hooks: Tap into lifecycle (emit, done, compilation)
3. Tap Types: tap (sync), tapAsync (callback), tapPromise (async/await)
4. Assets: Read/modify/create via compilation.assets
5. Use Cases: Bundle analysis, notifications, custom transformations
6. Example: Show simple plugin with practical use case

</details>
