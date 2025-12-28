# Performance & Optimization

> Expert / Architect Level (5+ years)

---

## Questions

151. How do you optimize bundle size in React applications?
152. What is tree shaking and how does it work?
153. How do you implement dynamic imports?
154. What are the performance implications of Context API?
155. How do you optimize rendering performance at scale?
156. What is the profiler API in React?
157. How do you implement server-side caching strategies?
158. What is edge computing and how does it relate to React?
159. How do you optimize images in React applications?
160. What is the impact of third-party libraries on performance?

---

## Detailed Answers

### 151. How do you optimize bundle size in React applications?

<details>
<summary>View Answer</summary>

**Optimizing bundle size** is crucial for improving load times, reducing bandwidth costs, and enhancing user experience. Smaller bundles load faster, especially on slow networks and mobile devices.

#### 1. Why Bundle Size Matters

**Performance Impact:**
```typescript
/*
Bundle Size Impact:

100 KB bundle:
- 3G (750 Kbps): ~1 second download
- 4G (10 Mbps): ~80ms download
- WiFi (50 Mbps): ~16ms download

500 KB bundle:
- 3G: ~5 seconds download
- 4G: ~400ms download
- WiFi: ~80ms download

1 MB bundle:
- 3G: ~10 seconds download
- 4G: ~800ms download
- WiFi: ~160ms download

Goal: Keep initial bundle < 200 KB (gzipped)
*/
```

#### 2. Analyze Your Bundle

**Webpack Bundle Analyzer:**
```bash
# Install
npm install --save-dev webpack-bundle-analyzer
```

```javascript
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
```

**Next.js Bundle Analysis:**
```bash
# Install
npm install @next/bundle-analyzer
```

```javascript
// next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

module.exports = withBundleAnalyzer({
  // Your Next.js config
});
```

```bash
# Run analysis
ANALYZE=true npm run build
```

**Source Map Explorer:**
```bash
# Install
npm install --save-dev source-map-explorer

# Add to package.json
"scripts": {
  "analyze": "source-map-explorer 'build/static/js/*.js'"
}

# Run
npm run build
npm run analyze
```

#### 3. Code Splitting

**Dynamic Imports:**
```typescript
// ❌ BAD: Import everything upfront
import HeavyComponent from './HeavyComponent';
import ChartLibrary from './ChartLibrary';
import VideoPlayer from './VideoPlayer';

function App() {
  return (
    <div>
      <HeavyComponent />
      <ChartLibrary />
      <VideoPlayer />
    </div>
  );
}

// Result: 500 KB initial bundle

// ✅ GOOD: Load on demand
import { lazy, Suspense } from 'react';

const HeavyComponent = lazy(() => import('./HeavyComponent'));
const ChartLibrary = lazy(() => import('./ChartLibrary'));
const VideoPlayer = lazy(() => import('./VideoPlayer'));

function App() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <HeavyComponent />
        <ChartLibrary />
        <VideoPlayer />
      </Suspense>
    </div>
  );
}

// Result: 100 KB initial, 400 KB loaded on demand
```

**Route-Based Splitting:**
```typescript
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

// Lazy load route components
const Home = lazy(() => import('./pages/Home'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Profile = lazy(() => import('./pages/Profile'));
const Settings = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<LoadingSpinner />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/profile" element={<Profile />} />
          <Route path="/settings" element={<Settings />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}

// Each route loads only when visited
```

**Next.js Dynamic Imports:**
```typescript
import dynamic from 'next/dynamic';

// Client-side only component
const DynamicChart = dynamic(() => import('./Chart'), {
  ssr: false,
  loading: () => <p>Loading chart...</p>,
});

// With named export
const DynamicModal = dynamic(
  () => import('./Modal').then(mod => mod.Modal)
);

export default function Page() {
  return (
    <div>
      <h1>Analytics</h1>
      <DynamicChart data={chartData} />
    </div>
  );
}
```

#### 4. Tree Shaking

**Use ES Modules:**
```typescript
// ✅ GOOD: ES modules (tree-shakeable)
import { Button, Input } from 'my-ui-library';

// ❌ BAD: CommonJS (not tree-shakeable)
const { Button, Input } = require('my-ui-library');
```

**Import Only What You Need:**
```typescript
// ❌ BAD: Import entire library
import _ from 'lodash';
const result = _.uniq([1, 2, 2, 3]);
// Bundle includes ALL of lodash (~70 KB)

// ✅ GOOD: Import specific function
import uniq from 'lodash/uniq';
const result = uniq([1, 2, 2, 3]);
// Bundle includes only uniq (~5 KB)

// ✅ EVEN BETTER: Use lodash-es
import { uniq } from 'lodash-es';
const result = uniq([1, 2, 2, 3]);
// Tree-shakeable, only uniq included
```

**Avoid Barrel Imports:**
```typescript
// ❌ BAD: Barrel export prevents tree-shaking
// components/index.ts
export { Button } from './Button';
export { Input } from './Input';
export { Modal } from './Modal';
export { Dropdown } from './Dropdown';
// ... 50 more components

// App.tsx
import { Button } from './components'; // Imports ALL components!

// ✅ GOOD: Direct imports
import { Button } from './components/Button';
import { Input } from './components/Input';
// Only imports what's needed
```

#### 5. Replace Heavy Libraries

**Lodash Alternatives:**
```typescript
// ❌ lodash: 70 KB
import _ from 'lodash';

// ✅ Native JavaScript: 0 KB
const unique = [...new Set(array)];
const sorted = [...array].sort();
const mapped = array.map(x => x * 2);
const filtered = array.filter(x => x > 10);
```

**Moment.js Alternatives:**
```typescript
// ❌ moment: 288 KB
import moment from 'moment';
const date = moment().format('YYYY-MM-DD');

// ✅ date-fns: 13 KB (tree-shakeable)
import { format } from 'date-fns';
const date = format(new Date(), 'yyyy-MM-dd');

// ✅ day.js: 7 KB
import dayjs from 'dayjs';
const date = dayjs().format('YYYY-MM-DD');

// ✅ Native Intl API: 0 KB
const date = new Intl.DateTimeFormat('en-US').format(new Date());
```

**Icon Libraries:**
```typescript
// ❌ Font Awesome (all icons): 900 KB
import { FontAwesomeIcon } from '@fortawesome/react-fontawesome';
import { fas } from '@fortawesome/free-solid-svg-icons';

// ✅ Individual icons: 5 KB each
import { FontAwesomeIcon } from '@fortawesome/react-fontawesome';
import { faHeart, faUser } from '@fortawesome/free-solid-svg-icons';

// ✅ React Icons (tree-shakeable): 2 KB per icon
import { FaHeart, FaUser } from 'react-icons/fa';
```

#### 6. Optimize Dependencies

**Check Package Size Before Installing:**
```bash
# Use bundlephobia.com or
npx bundle-phobia <package-name>

# Example
npx bundle-phobia moment
# Size: 288.45 KB (gzipped: 71.56 KB)

npx bundle-phobia date-fns
# Size: 78.14 KB (gzipped: 13.28 KB)
```

**Remove Unused Dependencies:**
```bash
# Find unused dependencies
npx depcheck

# Remove them
npm uninstall unused-package
```

**Use Lighter Alternatives:**
```typescript
/*
Heavy Library → Lighter Alternative:

moment (288 KB) → date-fns (13 KB) or dayjs (7 KB)
lodash (70 KB) → Native JS or lodash-es
axios (13 KB) → Native fetch (0 KB)
request (184 KB) → node-fetch or axios
jquery (90 KB) → Native DOM APIs
chartjs (240 KB) → recharts (93 KB) or lightweight SVG
*/
```

#### 7. Minification and Compression

**Production Build:**
```bash
# Create React App
npm run build

# Next.js
npm run build

# Vite
npm run build

# All automatically minify code
```

**Manual Terser Configuration:**
```javascript
// webpack.config.js
const TerserPlugin = require('terser-webpack-plugin');

module.exports = {
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin({
        terserOptions: {
          compress: {
            drop_console: true, // Remove console.log
            drop_debugger: true,
          },
        },
      }),
    ],
  },
};
```

**Enable Gzip/Brotli Compression:**
```javascript
// Express server
const compression = require('compression');
app.use(compression());

// Or configure at CDN/nginx level
// nginx.conf
gzip on;
gzip_types text/plain text/css application/json application/javascript;
gzip_min_length 1000;
```

#### 8. Optimize Images and Assets

**Image Optimization:**
```typescript
// ❌ BAD: Large images
<img src="photo.jpg" alt="Photo" /> // 2 MB

// ✅ GOOD: Next.js Image component
import Image from 'next/image';

<Image
  src="/photo.jpg"
  width={800}
  height={600}
  alt="Photo"
  loading="lazy"
  quality={75}
/>
// Automatically optimized, lazy loaded, WebP format

// ✅ Manual optimization
// Convert to WebP, compress, use responsive images
<picture>
  <source srcSet="photo.webp" type="image/webp" />
  <source srcSet="photo.jpg" type="image/jpeg" />
  <img src="photo.jpg" alt="Photo" loading="lazy" />
</picture>
```

**SVG Optimization:**
```typescript
// Import SVG as React component
import { ReactComponent as Logo } from './logo.svg';

<Logo width={50} height={50} />

// Or use SVGR for inline SVGs (reduces HTTP requests)
```

#### 9. Code Splitting Strategies

**Vendor Chunking:**
```javascript
// webpack.config.js
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        // Separate vendor code
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          priority: 10,
        },
        // Separate common code
        common: {
          minChunks: 2,
          priority: 5,
          reuseExistingChunk: true,
        },
      },
    },
  },
};
```

**Component-Level Splitting:**
```typescript
// Split heavy components
const Editor = lazy(() => import('./Editor'));
const VideoPlayer = lazy(() => import('./VideoPlayer'));
const PDFViewer = lazy(() => import('./PDFViewer'));

function DocumentViewer({ type, file }) {
  return (
    <Suspense fallback={<Loading />}>
      {type === 'text' && <Editor file={file} />}
      {type === 'video' && <VideoPlayer file={file} />}
      {type === 'pdf' && <PDFViewer file={file} />}
    </Suspense>
  );
}
```

#### 10. Remove Development Code

**Environment Variables:**
```typescript
// Development logging
if (process.env.NODE_ENV === 'development') {
  console.log('Debug info:', data);
}

// Production build removes this code automatically
```

**Conditional Imports:**
```typescript
// Only import dev tools in development
if (process.env.NODE_ENV === 'development') {
  import('./devTools').then(({ setupDevTools }) => {
    setupDevTools();
  });
}
```

#### 11. Use Production Builds

**React Production Mode:**
```typescript
// ❌ Development: 1.2 MB
// Includes warnings, debugging tools, dev-only code

// ✅ Production: 130 KB
// Minified, optimized, warnings removed
npm run build
```

**Check Build Size:**
```bash
# Create React App
npm run build
# Output shows gzipped sizes

# Next.js
npm run build
# Shows detailed bundle analysis
```

#### 12. Lazy Load Non-Critical Resources

**Intersection Observer:**
```typescript
import { useEffect, useRef, useState } from 'react';

function LazyImage({ src, alt }: { src: string; alt: string }) {
  const [isVisible, setIsVisible] = useState(false);
  const imgRef = useRef<HTMLImageElement>(null);
  
  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setIsVisible(true);
          observer.disconnect();
        }
      },
      { threshold: 0.1 }
    );
    
    if (imgRef.current) {
      observer.observe(imgRef.current);
    }
    
    return () => observer.disconnect();
  }, []);
  
  return (
    <img
      ref={imgRef}
      src={isVisible ? src : ''}
      alt={alt}
      loading="lazy"
    />
  );
}
```

#### 13. Optimize Third-Party Scripts

**Defer Non-Critical Scripts:**
```html
<!-- ❌ Blocks rendering -->
<script src="analytics.js"></script>

<!-- ✅ Loads after page renders -->
<script src="analytics.js" defer></script>

<!-- ✅ Loads asynchronously -->
<script src="analytics.js" async></script>
```

**Load Third-Party Code Conditionally:**
```typescript
// Only load analytics after user interaction
useEffect(() => {
  const handleFirstInteraction = () => {
    import('./analytics').then(({ initAnalytics }) => {
      initAnalytics();
    });
    
    window.removeEventListener('click', handleFirstInteraction);
    window.removeEventListener('scroll', handleFirstInteraction);
  };
  
  window.addEventListener('click', handleFirstInteraction);
  window.addEventListener('scroll', handleFirstInteraction);
}, []);
```

#### 14. Monitor Bundle Size

**Bundle Size Budgets:**
```javascript
// webpack.config.js
module.exports = {
  performance: {
    maxEntrypointSize: 250000, // 250 KB
    maxAssetSize: 250000,
    hints: 'error', // Fail build if exceeded
  },
};
```

**CI/CD Integration:**
```yaml
# .github/workflows/ci.yml
name: Check Bundle Size
on: [pull_request]

jobs:
  check-size:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: andresz1/size-limit-action@v1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

#### 15. Best Practices Summary

**Checklist:**
```typescript
/*
✅ Analyze bundle with webpack-bundle-analyzer
✅ Code split by route and component
✅ Use dynamic imports for heavy components
✅ Enable tree shaking (ES modules)
✅ Import only what you need from libraries
✅ Replace heavy libraries with lighter alternatives
✅ Remove unused dependencies
✅ Minify and compress code
✅ Optimize images (WebP, lazy loading)
✅ Defer non-critical scripts
✅ Use production builds
✅ Set bundle size budgets
✅ Monitor size in CI/CD
*/
```

**Target Bundle Sizes:**
```typescript
/*
Recommended Bundle Sizes (gzipped):

Initial Bundle:
- Excellent: < 100 KB
- Good: 100-200 KB
- Acceptable: 200-300 KB
- Poor: > 300 KB

Total JavaScript:
- Excellent: < 300 KB
- Good: 300-500 KB
- Acceptable: 500-700 KB
- Poor: > 700 KB

Goal: Load critical content in < 3 seconds on 3G
*/
```

#### Summary

**Key Optimization Techniques:**

1. **Code Splitting**
   - Route-based splitting
   - Component-based splitting
   - Vendor chunking

2. **Tree Shaking**
   - Use ES modules
   - Import specific functions
   - Avoid barrel exports

3. **Dependency Optimization**
   - Check package sizes
   - Use lighter alternatives
   - Remove unused packages

4. **Asset Optimization**
   - Compress images
   - Lazy load media
   - Use modern formats (WebP)

5. **Build Optimization**
   - Minify code
   - Enable compression
   - Use production builds

**Tools:**
- webpack-bundle-analyzer
- source-map-explorer
- bundlephobia.com
- lighthouse

**Impact:**
> "Reducing bundle size from 500 KB to 200 KB can decrease load time by 60% on 3G networks!"

</details>

---

### 152. What is tree shaking and how does it work?

<details>
<summary>View Answer</summary>

**Tree shaking** is a dead code elimination technique that removes unused code from your bundle. It's called "tree shaking" because it shakes off the "dead leaves" (unused exports) from the dependency tree.

#### 1. What is Tree Shaking?

**Concept:**
```typescript
/*
Without Tree Shaking:

// utils.js (100 KB)
export function usedFunction() { ... }      // Used in app
export function unusedFunction1() { ... }  // Not used
export function unusedFunction2() { ... }  // Not used
export function unusedFunction3() { ... }  // Not used

// app.js
import { usedFunction } from './utils';

// Bundle includes ALL functions (100 KB)

---

With Tree Shaking:

// Same utils.js

// app.js
import { usedFunction } from './utils';

// Bundle includes ONLY usedFunction (10 KB)
// 90% reduction!
*/
```

**How It Works:**
```typescript
/*
Tree Shaking Process:

1. Static Analysis:
   - Analyze import/export statements
   - Build dependency graph
   - Mark used exports

2. Dead Code Detection:
   - Find exports that are never imported
   - Mark as "dead code"

3. Elimination:
   - Remove dead code during bundling
   - Keep only used exports

4. Minification:
   - Further optimize remaining code
   - Remove whitespace, rename variables
*/
```

#### 2. Requirements for Tree Shaking

**ES Modules (ES6) Required:**
```typescript
// ✅ GOOD: ES modules (tree-shakeable)
export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}

// app.js
import { add } from './math'; // Only 'add' included in bundle

// ❌ BAD: CommonJS (NOT tree-shakeable)
module.exports = {
  add: function(a, b) {
    return a + b;
  },
  subtract: function(a, b) {
    return a - b;
  },
};

// app.js
const { add } = require('./math'); // ENTIRE module included!
```

**Why ES Modules Work:**
```typescript
/*
ES Modules:
- Static imports/exports
- Can analyze at build time
- Know exactly what's used

CommonJS:
- Dynamic requires
- Can't analyze until runtime
- Must include everything

Example:
*/

// ❌ Dynamic (can't tree shake)
const moduleName = condition ? 'moduleA' : 'moduleB';
const module = require(moduleName);

// ✅ Static (can tree shake)
import { funcA } from './moduleA';
import { funcB } from './moduleB';
```

#### 3. Tree Shaking in Action

**Example Library:**
```typescript
// utils.ts (before tree shaking)
export function formatDate(date: Date): string {
  return date.toISOString();
}

export function formatCurrency(amount: number): string {
  return `$${amount.toFixed(2)}`;
}

export function formatPercentage(value: number): string {
  return `${(value * 100).toFixed(1)}%`;
}

export function formatPhoneNumber(phone: string): string {
  return phone.replace(/(\d{3})(\d{3})(\d{4})/, '($1) $2-$3');
}

// Each function is ~1 KB
// Total: 4 KB
```

**Usage in App:**
```typescript
// app.ts
import { formatCurrency } from './utils';

const price = formatCurrency(29.99);
console.log(price); // $29.99

// Without tree shaking: 4 KB bundled
// With tree shaking: 1 KB bundled (only formatCurrency)
```

**Bundle Output:**
```javascript
// Without tree shaking (4 KB)
function formatDate(date) { return date.toISOString(); }
function formatCurrency(amount) { return '$' + amount.toFixed(2); }
function formatPercentage(value) { return (value * 100).toFixed(1) + '%'; }
function formatPhoneNumber(phone) { return phone.replace(/(\d{3})(\d{3})(\d{4})/, '($1) $2-$3'); }

const price = formatCurrency(29.99);

// With tree shaking (1 KB)
function formatCurrency(amount) { return '$' + amount.toFixed(2); }
const price = formatCurrency(29.99);
```

#### 4. Webpack Configuration

**Enable Tree Shaking:**
```javascript
// webpack.config.js
module.exports = {
  mode: 'production', // Enables tree shaking automatically
  
  optimization: {
    usedExports: true, // Mark unused exports
    minimize: true,     // Remove them
  },
};
```

**Manual Configuration:**
```javascript
// webpack.config.js
const TerserPlugin = require('terser-webpack-plugin');

module.exports = {
  mode: 'production',
  
  optimization: {
    usedExports: true,
    minimize: true,
    minimizer: [
      new TerserPlugin({
        terserOptions: {
          compress: {
            dead_code: true,
            unused: true,
          },
        },
      }),
    ],
    
    // Split vendor code for better caching
    splitChunks: {
      chunks: 'all',
    },
  },
};
```

#### 5. Package.json Configuration

**Side Effects:**
```json
// package.json
{
  "name": "my-library",
  "version": "1.0.0",
  "sideEffects": false
}

// Tells bundler:
// "This package has NO side effects"
// "You can safely remove any unused exports"
```

**With Side Effects:**
```json
{
  "name": "my-library",
  "sideEffects": [
    "*.css",
    "*.scss",
    "src/polyfills.ts"
  ]
}

// Tells bundler:
// "These files have side effects, don't remove them"
// "Everything else can be tree shaken"
```

**What are Side Effects?**
```typescript
// ❌ HAS side effects (modifies global state)
// polyfill.js
Array.prototype.includes = function() { ... };
window.myGlobal = 'value';

// Must mark as sideEffects: true
// Otherwise might be removed even if imported

// ✅ NO side effects (pure exports)
// utils.js
export function add(a, b) {
  return a + b;
}

// Can be tree shaken safely
```

#### 6. Common Pitfalls

**Barrel Exports (index.js):**
```typescript
// ❌ BAD: Prevents tree shaking
// components/index.ts
export { Button } from './Button';
export { Input } from './Input';
export { Modal } from './Modal';
export { Dropdown } from './Dropdown';
// ... 50 more components

// app.tsx
import { Button } from './components';
// Problem: webpack can't determine which components are used
// Might include ALL components!

// ✅ GOOD: Direct imports
// app.tsx
import { Button } from './components/Button';
import { Input } from './components/Input';
// Only imports needed components
```

**Default Exports:**
```typescript
// ⚠️ LESS OPTIMAL: Default export
// utils.ts
export default {
  add: (a, b) => a + b,
  subtract: (a, b) => a - b,
};

// app.ts
import utils from './utils';
const result = utils.add(1, 2);
// Entire object included (harder to tree shake)

// ✅ BETTER: Named exports
// utils.ts
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;

// app.ts
import { add } from './utils';
const result = add(1, 2);
// Only 'add' included
```

**Class with Methods:**
```typescript
// ⚠️ HARDER to tree shake
class MathUtils {
  static add(a, b) { return a + b; }
  static subtract(a, b) { return a - b; }
  static multiply(a, b) { return a * b; }
  static divide(a, b) { return a / b; }
}

export default MathUtils;

// Usage
import MathUtils from './MathUtils';
MathUtils.add(1, 2);
// Entire class included (all methods)

// ✅ EASIER to tree shake
export function add(a, b) { return a + b; }
export function subtract(a, b) { return a - b; }
export function multiply(a, b) { return a * b; }
export function divide(a, b) { return a / b; }

// Usage
import { add } from './math';
add(1, 2);
// Only 'add' included
```

#### 7. Real-World Examples

**Lodash:**
```typescript
// ❌ BAD: Imports entire library (70 KB)
import _ from 'lodash';
const result = _.uniq([1, 2, 2, 3]);

// ⚠️ BETTER: Import specific function (5 KB)
import uniq from 'lodash/uniq';
const result = uniq([1, 2, 2, 3]);

// ✅ BEST: Use lodash-es (tree-shakeable, 2 KB)
import { uniq } from 'lodash-es';
const result = uniq([1, 2, 2, 3]);
// Only 'uniq' function included
```

**React Icons:**
```typescript
// ✅ Tree-shakeable
import { FaHome, FaUser, FaCog } from 'react-icons/fa';

// Only includes 3 icons (~6 KB)
// Not the entire Font Awesome library (900 KB)
```

**Material-UI:**
```typescript
// ❌ BAD: Imports everything
import { Button } from '@mui/material';
// Entire library (~300 KB)

// ✅ GOOD: Direct import (with babel-plugin-import)
import Button from '@mui/material/Button';
// Only Button component (~30 KB)

// Or configure babel-plugin-import
// .babelrc
{
  "plugins": [
    [
      "babel-plugin-import",
      {
        "libraryName": "@mui/material",
        "libraryDirectory": "",
        "camel2DashComponentName": false
      }
    ]
  ]
}
```

#### 8. Verifying Tree Shaking

**Check Bundle:**
```bash
# Build and analyze
npm run build

# Look for specific function in bundle
grep "unusedFunction" build/static/js/*.js
# If found, tree shaking didn't work
# If not found, tree shaking worked!
```

**Webpack Stats:**
```javascript
// webpack.config.js
module.exports = {
  stats: {
    usedExports: true, // Show which exports are used
  },
};

// Output:
// [usedExports] ./utils.js
//   - formatCurrency (used)
//   - formatDate (unused)
//   - formatPercentage (unused)
```

**Bundle Analyzer:**
```bash
npm install --save-dev webpack-bundle-analyzer

# View what's included in bundle
npm run build -- --analyze
```

#### 9. Optimizing for Tree Shaking

**Write Tree-Shakeable Code:**
```typescript
// ✅ GOOD: Individual exports
export function add(a: number, b: number): number {
  return a + b;
}

export function subtract(a: number, b: number): number {
  return a - b;
}

// Each function is independently tree-shakeable

// ❌ BAD: Single object export
export default {
  add(a: number, b: number): number {
    return a + b;
  },
  subtract(a: number, b: number): number {
    return a - b;
  },
};

// Entire object must be included
```

**Avoid Side Effects:**
```typescript
// ❌ BAD: Side effect in module
// utils.ts
export function add(a, b) {
  return a + b;
}

// Side effect - runs when imported!
console.log('Utils loaded');

// ✅ GOOD: Pure exports only
// utils.ts
export function add(a, b) {
  return a + b;
}

// No side effects
```

**Mark Pure Functions:**
```javascript
// Use /*#__PURE__*/ annotation
export const complexCalculation = /*#__PURE__*/ (() => {
  const result = expensiveOperation();
  return () => result;
})();

// Tells bundler:
// "This function has no side effects"
// "Can remove if unused"
```

#### 10. Framework-Specific Tree Shaking

**Create React App:**
```bash
# Automatically enabled in production
npm run build

# Uses webpack with tree shaking enabled
```

**Next.js:**
```javascript
// next.config.js
module.exports = {
  // Tree shaking enabled by default
  
  // Additional optimization
  experimental: {
    modularizeImports: {
      '@mui/material': {
        transform: '@mui/material/{{member}}',
      },
    },
  },
};
```

**Vite:**
```javascript
// vite.config.js
export default {
  build: {
    minify: 'terser', // Enable tree shaking
    terserOptions: {
      compress: {
        dead_code: true,
        unused: true,
      },
    },
  },
};
```

#### 11. Best Practices

**Checklist:**
```typescript
/*
✅ Use ES modules (not CommonJS)
✅ Use named exports (not default)
✅ Avoid barrel exports (index.js)
✅ Set sideEffects: false in package.json
✅ Import specific functions, not entire libraries
✅ Use production builds
✅ Enable minification
✅ Analyze bundle to verify
✅ Use tree-shakeable libraries
✅ Avoid side effects in modules
*/
```

**Testing Tree Shaking:**
```typescript
// Create test file
// test-tree-shaking.js
export function used() {
  console.log('This is used');
}

export function unused() {
  console.log('This is NOT used');
}

// app.js
import { used } from './test-tree-shaking';
used();

// Build and check
npm run build
grep "unused" build/static/js/*.js
// If 'unused' not found, tree shaking works!
```

#### Summary

**Tree Shaking:**
- Removes unused code from bundles
- Reduces bundle size significantly
- Works with ES modules only
- Requires static imports/exports

**How It Works:**
1. Analyze import/export statements
2. Mark used exports
3. Remove unused code
4. Minify result

**Requirements:**
- Use ES6 modules (`import`/`export`)
- Avoid CommonJS (`require`/`module.exports`)
- Use production builds
- Set `sideEffects: false` when possible

**Best Practices:**
- Named exports > default exports
- Direct imports > barrel exports
- Individual functions > class methods
- Pure functions > side effects

**Impact:**
> "Tree shaking can reduce bundle size by 30-70% by eliminating unused code!"

**Key Takeaway:**
> "Use ES modules, named exports, and direct imports to maximize tree shaking effectiveness."

</details>

---

### 153. How do you implement dynamic imports?

<details>
<summary>View Answer</summary>

**Dynamic imports** allow you to load JavaScript modules on-demand rather than at the initial page load, reducing the initial bundle size and improving load times. React provides `React.lazy()` and native JavaScript `import()` for dynamic importing.

#### 1. Basic Dynamic Import with React.lazy

**Simple Component Loading:**
```typescript
import { lazy, Suspense } from 'react';

// Dynamic import - component loads only when needed
const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <div>
      <h1>My App</h1>
      
      {/* Show fallback while loading */}
      <Suspense fallback={<div>Loading...</div>}>
        <HeavyComponent />
      </Suspense>
    </div>
  );
}

// HeavyComponent.tsx
export default function HeavyComponent() {
  return <div>I'm a heavy component!</div>;
}

// Result:
// - Initial bundle: Small (no HeavyComponent)
// - When rendered: Loads HeavyComponent on demand
```

**With Loading Spinner:**
```typescript
import { lazy, Suspense } from 'react';

const Dashboard = lazy(() => import('./pages/Dashboard'));

function LoadingSpinner() {
  return (
    <div className="loading-spinner">
      <div className="spinner" />
      <p>Loading...</p>
    </div>
  );
}

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Dashboard />
    </Suspense>
  );
}
```

#### 2. Route-Based Code Splitting

**React Router with Lazy Loading:**
```typescript
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

// Lazy load route components
const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Profile = lazy(() => import('./pages/Profile'));
const Settings = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<PageLoader />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/profile" element={<Profile />} />
          <Route path="/settings" element={<Settings />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}

// Each route loads only when visited
// Home page bundle: ~50 KB
// Dashboard bundle: ~100 KB (loads when /dashboard visited)
```

**Nested Routes with Multiple Suspense Boundaries:**
```typescript
import { lazy, Suspense } from 'react';
import { Routes, Route, Outlet } from 'react-router-dom';

const Dashboard = lazy(() => import('./pages/Dashboard'));
const Analytics = lazy(() => import('./pages/Analytics'));
const Reports = lazy(() => import('./pages/Reports'));

function DashboardLayout() {
  return (
    <div>
      <nav>{/* Dashboard nav */}</nav>
      <Suspense fallback={<div>Loading section...</div>}>
        <Outlet />
      </Suspense>
    </div>
  );
}

function App() {
  return (
    <Routes>
      <Route path="/dashboard" element={<DashboardLayout />}>
        <Route index element={<Dashboard />} />
        <Route path="analytics" element={<Analytics />} />
        <Route path="reports" element={<Reports />} />
      </Route>
    </Routes>
  );
}
```

#### 3. Component-Level Code Splitting

**Modal Dialog:**
```typescript
import { lazy, Suspense, useState } from 'react';

// Only load modal when needed
const Modal = lazy(() => import('./Modal'));

function Page() {
  const [isModalOpen, setIsModalOpen] = useState(false);
  
  return (
    <div>
      <h1>My Page</h1>
      <button onClick={() => setIsModalOpen(true)}>
        Open Modal
      </button>
      
      {isModalOpen && (
        <Suspense fallback={<div>Loading modal...</div>}>
          <Modal onClose={() => setIsModalOpen(false)} />
        </Suspense>
      )}
    </div>
  );
}

// Modal only loads when button clicked
```

**Tab-Based Loading:**
```typescript
import { lazy, Suspense, useState } from 'react';

const ProfileTab = lazy(() => import('./tabs/ProfileTab'));
const SettingsTab = lazy(() => import('./tabs/SettingsTab'));
const NotificationsTab = lazy(() => import('./tabs/NotificationsTab'));

function UserDashboard() {
  const [activeTab, setActiveTab] = useState<'profile' | 'settings' | 'notifications'>('profile');
  
  return (
    <div>
      <div className="tabs">
        <button onClick={() => setActiveTab('profile')}>Profile</button>
        <button onClick={() => setActiveTab('settings')}>Settings</button>
        <button onClick={() => setActiveTab('notifications')}>Notifications</button>
      </div>
      
      <Suspense fallback={<div>Loading tab...</div>}>
        {activeTab === 'profile' && <ProfileTab />}
        {activeTab === 'settings' && <SettingsTab />}
        {activeTab === 'notifications' && <NotificationsTab />}
      </Suspense>
    </div>
  );
}

// Each tab loads only when clicked
```

#### 4. Named Exports

**Import Named Export:**
```typescript
import { lazy, Suspense } from 'react';

// Component with named export
const UserProfile = lazy(() => 
  import('./components/User').then(module => ({
    default: module.UserProfile,
  }))
);

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <UserProfile userId="123" />
    </Suspense>
  );
}

// components/User.tsx
export function UserProfile({ userId }: { userId: string }) {
  return <div>User: {userId}</div>;
}

export function UserSettings({ userId }: { userId: string }) {
  return <div>Settings for: {userId}</div>;
}
```

**Multiple Named Exports:**
```typescript
// Load multiple components from same file
const { UserProfile, UserSettings } = {
  UserProfile: lazy(() => 
    import('./components/User').then(m => ({ default: m.UserProfile }))
  ),
  UserSettings: lazy(() => 
    import('./components/User').then(m => ({ default: m.UserSettings }))
  ),
};
```

#### 5. Error Boundaries

**Handle Loading Errors:**
```typescript
import { Component, lazy, Suspense, ReactNode } from 'react';

interface ErrorBoundaryState {
  hasError: boolean;
  error: Error | null;
}

class ErrorBoundary extends Component<
  { children: ReactNode },
  ErrorBoundaryState
> {
  state = { hasError: false, error: null };
  
  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }
  
  componentDidCatch(error: Error, errorInfo: any) {
    console.error('Component loading error:', error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return (
        <div className="error">
          <h2>Failed to load component</h2>
          <p>{this.state.error?.message}</p>
          <button onClick={() => window.location.reload()}>
            Reload Page
          </button>
        </div>
      );
    }
    
    return this.props.children;
  }
}

const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <ErrorBoundary>
      <Suspense fallback={<div>Loading...</div>}>
        <HeavyComponent />
      </Suspense>
    </ErrorBoundary>
  );
}
```

**Retry Logic:**
```typescript
function lazyWithRetry<T extends React.ComponentType<any>>(
  componentImport: () => Promise<{ default: T }>
) {
  return lazy(() => {
    return new Promise<{ default: T }>((resolve, reject) => {
      const attemptImport = (retries: number = 3) => {
        componentImport()
          .then(resolve)
          .catch((error) => {
            if (retries > 0) {
              console.log(`Retrying import... (${retries} attempts left)`);
              setTimeout(() => attemptImport(retries - 1), 1000);
            } else {
              reject(error);
            }
          });
      };
      
      attemptImport();
    });
  });
}

// Usage
const Dashboard = lazyWithRetry(() => import('./pages/Dashboard'));
```

#### 6. Native Dynamic Import (Non-React)

**Basic JavaScript import():**
```typescript
// Load module dynamically
async function loadModule() {
  const module = await import('./utils');
  module.someFunction();
}

// Or with .then()
import('./utils').then(module => {
  module.someFunction();
});
```

**Load Library on Demand:**
```typescript
function ChartComponent({ data }: { data: number[] }) {
  const [Chart, setChart] = useState<any>(null);
  
  useEffect(() => {
    // Load Chart.js only when component mounts
    import('chart.js').then(({ Chart }) => {
      setChart(() => Chart);
    });
  }, []);
  
  if (!Chart) {
    return <div>Loading chart library...</div>;
  }
  
  return <canvas ref={canvasRef} />;
}
```

**Conditional Loading:**
```typescript
async function handleExport(format: 'pdf' | 'excel') {
  if (format === 'pdf') {
    const { exportToPDF } = await import('./exportPDF');
    exportToPDF(data);
  } else {
    const { exportToExcel } = await import('./exportExcel');
    exportToExcel(data);
  }
}

// Only loads the needed export module
```

#### 7. Preloading Components

**Eager Loading on Hover:**
```typescript
import { lazy, Suspense, useState } from 'react';

const Modal = lazy(() => import('./Modal'));

// Preload function
const preloadModal = () => import('./Modal');

function Page() {
  const [showModal, setShowModal] = useState(false);
  
  return (
    <div>
      <button
        onClick={() => setShowModal(true)}
        onMouseEnter={preloadModal} // Preload on hover
        onFocus={preloadModal}      // Preload on focus
      >
        Open Modal
      </button>
      
      {showModal && (
        <Suspense fallback={<div>Loading...</div>}>
          <Modal onClose={() => setShowModal(false)} />
        </Suspense>
      )}
    </div>
  );
}

// Modal loads instantly when clicked (already preloaded)
```

**Prefetch on Idle:**
```typescript
import { lazy, Suspense, useEffect } from 'react';

const HeavyComponent = lazy(() => import('./HeavyComponent'));

// Preload during browser idle time
const preloadHeavyComponent = () => import('./HeavyComponent');

function App() {
  useEffect(() => {
    // Prefetch when browser is idle
    if ('requestIdleCallback' in window) {
      requestIdleCallback(() => {
        preloadHeavyComponent();
      });
    } else {
      // Fallback: load after 2 seconds
      setTimeout(preloadHeavyComponent, 2000);
    }
  }, []);
  
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <HeavyComponent />
      </Suspense>
    </div>
  );
}
```

#### 8. Next.js Dynamic Imports

**Client-Side Only Component:**
```typescript
import dynamic from 'next/dynamic';

// Don't server-render (uses window, localStorage, etc.)
const ClientOnlyComponent = dynamic(
  () => import('./ClientOnlyComponent'),
  { ssr: false }
);

export default function Page() {
  return (
    <div>
      <h1>My Page</h1>
      <ClientOnlyComponent />
    </div>
  );
}
```

**With Custom Loading:**
```typescript
import dynamic from 'next/dynamic';

const DynamicChart = dynamic(
  () => import('./Chart'),
  {
    loading: () => <p>Loading chart...</p>,
    ssr: false,
  }
);

export default function Analytics() {
  return (
    <div>
      <h1>Analytics</h1>
      <DynamicChart data={chartData} />
    </div>
  );
}
```

**Named Export:**
```typescript
import dynamic from 'next/dynamic';

const DynamicComponent = dynamic(
  () => import('./components').then(mod => mod.SpecificComponent)
);
```

#### 9. Webpack Magic Comments

**Chunk Naming:**
```typescript
// Name the chunk for easier debugging
const Dashboard = lazy(() => 
  import(
    /* webpackChunkName: "dashboard" */
    './pages/Dashboard'
  )
);

// Output: dashboard.chunk.js
```

**Prefetch:**
```typescript
// Browser prefetches chunk when idle
const Settings = lazy(() => 
  import(
    /* webpackPrefetch: true */
    './pages/Settings'
  )
);

// Adds: <link rel="prefetch" href="settings.chunk.js">
```

**Preload:**
```typescript
// Browser preloads chunk immediately
const CriticalComponent = lazy(() => 
  import(
    /* webpackPreload: true */
    './CriticalComponent'
  )
);

// Adds: <link rel="preload" href="critical.chunk.js">
```

**Multiple Comments:**
```typescript
const Analytics = lazy(() => 
  import(
    /* webpackChunkName: "analytics" */
    /* webpackPrefetch: true */
    './pages/Analytics'
  )
);
```

#### 10. Advanced Patterns

**Lazy Loading with Props:**
```typescript
import { lazy, Suspense, ComponentType } from 'react';

type DynamicComponentProps = {
  type: 'video' | 'audio' | 'pdf';
  src: string;
};

function MediaViewer({ type, src }: DynamicComponentProps) {
  // Load different components based on type
  const Component = lazy(() => {
    switch (type) {
      case 'video':
        return import('./VideoPlayer');
      case 'audio':
        return import('./AudioPlayer');
      case 'pdf':
        return import('./PDFViewer');
      default:
        throw new Error('Unknown media type');
    }
  });
  
  return (
    <Suspense fallback={<div>Loading {type} player...</div>}>
      <Component src={src} />
    </Suspense>
  );
}
```

**Dynamic Import with Data Fetching:**
```typescript
import { lazy, Suspense } from 'react';

function UserProfile({ userId }: { userId: string }) {
  // Load component AND data together
  const ProfileComponent = lazy(async () => {
    const [component, userData] = await Promise.all([
      import('./ProfileComponent'),
      fetch(`/api/users/${userId}`).then(r => r.json()),
    ]);
    
    // Return component with data pre-loaded
    return {
      default: (props: any) => <component.default {...props} userData={userData} />,
    };
  });
  
  return (
    <Suspense fallback={<div>Loading profile...</div>}>
      <ProfileComponent />
    </Suspense>
  );
}
```

**Split by Feature:**
```typescript
// Admin features only loaded for admin users
import { lazy, Suspense } from 'react';

const AdminPanel = lazy(() => import('./features/admin/AdminPanel'));
const UserPanel = lazy(() => import('./features/user/UserPanel'));

function Dashboard({ isAdmin }: { isAdmin: boolean }) {
  return (
    <Suspense fallback={<div>Loading dashboard...</div>}>
      {isAdmin ? <AdminPanel /> : <UserPanel />}
    </Suspense>
  );
}

// Admin bundle only loaded for admins
```

#### 11. Performance Optimization

**Minimize Suspense Boundaries:**
```typescript
// ❌ BAD: Too many Suspense boundaries
function App() {
  return (
    <div>
      <Suspense fallback={<div>Loading 1...</div>}>
        <Component1 />
      </Suspense>
      <Suspense fallback={<div>Loading 2...</div>}>
        <Component2 />
      </Suspense>
      <Suspense fallback={<div>Loading 3...</div>}>
        <Component3 />
      </Suspense>
    </div>
  );
}
// Result: Multiple loading states, jarring UX

// ✅ GOOD: Single Suspense boundary
function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Component1 />
      <Component2 />
      <Component3 />
    </Suspense>
  );
}
// Result: Single loading state, smoother UX
```

**Strategic Code Splitting:**
```typescript
// Split at route level, not component level
// ✅ GOOD: Split by route
const HomePage = lazy(() => import('./pages/Home'));
const DashboardPage = lazy(() => import('./pages/Dashboard'));

// ❌ AVOID: Splitting too granularly
const Button = lazy(() => import('./Button')); // Too small!
const Header = lazy(() => import('./Header')); // Used everywhere!
```

#### 12. Best Practices

**Checklist:**
```typescript
/*
✅ Split at route level first
✅ Use Suspense with fallback UI
✅ Add error boundaries
✅ Preload on hover/focus for better UX
✅ Use webpack magic comments for naming
✅ Don't split too granularly
✅ Lazy load heavy libraries (charts, editors)
✅ Test with slow network (DevTools throttling)
✅ Monitor bundle sizes after changes
✅ Use production builds for testing
*/
```

**When to Use Dynamic Imports:**
```typescript
/*
✅ Route components
✅ Modal dialogs
✅ Tab content
✅ Heavy libraries (chart.js, video players)
✅ Admin/premium features
✅ Rarely used components

❌ Small, frequently used components
❌ Critical above-the-fold content
❌ Components < 10 KB
*/
```

#### Summary

**Dynamic Imports:**
- Load code on-demand, not upfront
- Reduce initial bundle size
- Improve load times
- Better user experience

**Implementation Methods:**
1. **React.lazy()** - For components
2. **import()** - For modules/libraries
3. **dynamic** (Next.js) - With SSR control

**Key Patterns:**
- Route-based splitting
- Component-level splitting
- Conditional loading
- Preloading/prefetching

**Best Practices:**
- Always use Suspense with lazy()
- Add error boundaries
- Preload for better UX
- Split strategically
- Test with slow networks

**Impact:**
> "Dynamic imports can reduce initial bundle size by 50-70%, drastically improving load times!"

**Golden Rule:**
> "Split by routes first, then split heavy features. Don't over-split!"

</details>

---

### 154. What are the performance implications of Context API?

<details>
<summary>View Answer</summary>

**The Context API** can cause performance issues if not used carefully. When a context value changes, **all components consuming that context re-render**, even if they only use a small part of the context value. This can lead to unnecessary re-renders.

#### 1. The Performance Problem

**How Context Works:**
```typescript
import { createContext, useContext, useState } from 'react';

// Create context
const UserContext = createContext<{
  user: { name: string; email: string };
  theme: string;
  setUser: (user: any) => void;
  setTheme: (theme: string) => void;
} | null>(null);

function UserProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState({ name: 'John', email: 'john@example.com' });
  const [theme, setTheme] = useState('light');
  
  return (
    <UserContext.Provider value={{ user, theme, setUser, setTheme }}>
      {children}
    </UserContext.Provider>
  );
}

// Component only needs theme
function ThemeToggle() {
  const context = useContext(UserContext);
  if (!context) throw new Error('Missing provider');
  
  console.log('ThemeToggle rendered'); // ⚠️ Logs on EVERY context change!
  
  return (
    <button onClick={() => context.setTheme(
      context.theme === 'light' ? 'dark' : 'light'
    )}>
      Toggle Theme
    </button>
  );
}

// Component only needs user
function UserGreeting() {
  const context = useContext(UserContext);
  if (!context) throw new Error('Missing provider');
  
  console.log('UserGreeting rendered'); // ⚠️ Logs on EVERY context change!
  
  return <div>Hello, {context.user.name}!</div>;
}

/*
Problem:
- Change theme → UserGreeting re-renders (unnecessary!)
- Change user → ThemeToggle re-renders (unnecessary!)
- Both re-render even though they use different data
*/
```

**Demonstration:**
```typescript
function App() {
  return (
    <UserProvider>
      <ThemeToggle />     {/* Re-renders on user OR theme change */}
      <UserGreeting />    {/* Re-renders on user OR theme change */}
    </UserProvider>
  );
}

// User changes: Both components re-render
// Theme changes: Both components re-render
// Result: Unnecessary re-renders
```

#### 2. Why This Happens

**Context Update Behavior:**
```typescript
/*
How React Context Works:

1. Provider value changes:
   value={{ user, theme, setUser, setTheme }}
   
2. React compares:
   Old value !== New value ?
   
3. If different:
   - Mark all consumers as needing update
   - Trigger re-render of ALL consuming components
   
4. No optimization:
   - Can't tell which part of context changed
   - Can't tell which part component uses
   - Re-renders everything

Key Issue: All-or-nothing updates
*/
```

**Reference Equality:**
```typescript
function BadProvider({ children }: { children: React.ReactNode }) {
  const [count, setCount] = useState(0);
  
  // ❌ BAD: New object every render!
  return (
    <CountContext.Provider value={{ count, setCount }}>
      {children}
    </CountContext.Provider>
  );
}

/*
Every render:
- Creates new { count, setCount } object
- Old value !== New value (different reference)
- ALL consumers re-render
- Even if count didn't change!
*/
```

#### 3. Optimization Techniques

**1. Split Contexts:**
```typescript
// ❌ BAD: Single context with multiple concerns
const AppContext = createContext({
  user: null,
  theme: 'light',
  settings: {},
  notifications: [],
});

// Change theme → Everything re-renders

// ✅ GOOD: Separate contexts
const UserContext = createContext(null);
const ThemeContext = createContext('light');
const SettingsContext = createContext({});
const NotificationsContext = createContext([]);

function App() {
  return (
    <UserProvider>
      <ThemeProvider>
        <SettingsProvider>
          <NotificationsProvider>
            {children}
          </NotificationsProvider>
        </SettingsProvider>
      </ThemeProvider>
    </UserProvider>
  );
}

// Now: Change theme → Only theme consumers re-render!
```

**2. Memoize Context Value:**
```typescript
import { createContext, useState, useMemo } from 'react';

function UserProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState({ name: 'John', email: 'john@example.com' });
  
  // ✅ Memoize value - only changes when user changes
  const value = useMemo(
    () => ({ user, setUser }),
    [user]
  );
  
  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
}

// Benefits:
// - Same object reference if user unchanged
// - Prevents unnecessary re-renders
// - Only updates when dependencies change
```

**3. Split State and Updaters:**
```typescript
// Separate state from update functions
const UserStateContext = createContext(null);
const UserDispatchContext = createContext(null);

function UserProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState({ name: 'John' });
  
  // Dispatch never changes (stable reference)
  const dispatch = useMemo(
    () => ({
      setUser,
      updateName: (name: string) => setUser(u => ({ ...u, name })),
    }),
    []
  );
  
  return (
    <UserStateContext.Provider value={user}>
      <UserDispatchContext.Provider value={dispatch}>
        {children}
      </UserDispatchContext.Provider>
    </UserStateContext.Provider>
  );
}

// Component that only updates
function UpdateUserButton() {
  const dispatch = useContext(UserDispatchContext);
  
  // ✅ Never re-renders when user changes!
  return (
    <button onClick={() => dispatch.updateName('Jane')}>
      Update Name
    </button>
  );
}

// Component that only reads
function UserDisplay() {
  const user = useContext(UserStateContext);
  
  // ✅ Re-renders only when user changes
  return <div>{user.name}</div>;
}
```

**4. Use Context Selectors (Workaround):**
```typescript
import { createContext, useContext, useRef, useSyncExternalStore } from 'react';

// Custom hook with selector
function useContextSelector<T, S>(
  context: React.Context<T>,
  selector: (value: T) => S
): S {
  const value = useContext(context);
  const selectedRef = useRef<S>();
  const selected = selector(value);
  
  // Only trigger re-render if selected value changed
  if (!Object.is(selectedRef.current, selected)) {
    selectedRef.current = selected;
  }
  
  return selectedRef.current as S;
}

// Usage
function ThemeToggle() {
  // Only re-renders when theme changes
  const theme = useContextSelector(
    AppContext,
    ctx => ctx.theme
  );
  
  return <button>Theme: {theme}</button>;
}

// Or use library: use-context-selector
import { createContext, useContextSelector } from 'use-context-selector';
```

**5. React.memo for Consumers:**
```typescript
import { memo } from 'react';

// Prevent re-render if props unchanged
const ExpensiveComponent = memo(function ExpensiveComponent({ 
  userId 
}: { 
  userId: string 
}) {
  const user = useContext(UserContext);
  
  // Only re-renders if:
  // - userId prop changes
  // - UserContext value changes
  
  return <div>{user.name}</div>;
});
```

#### 4. useReducer with Context

**Better State Management:**
```typescript
import { createContext, useReducer, useContext, useMemo } from 'react';

type State = {
  user: { name: string; email: string };
  theme: string;
};

type Action = 
  | { type: 'SET_USER'; payload: State['user'] }
  | { type: 'SET_THEME'; payload: string };

const StateContext = createContext<State | null>(null);
const DispatchContext = createContext<React.Dispatch<Action> | null>(null);

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'SET_USER':
      return { ...state, user: action.payload };
    case 'SET_THEME':
      return { ...state, theme: action.payload };
    default:
      return state;
  }
}

function AppProvider({ children }: { children: React.ReactNode }) {
  const [state, dispatch] = useReducer(reducer, {
    user: { name: 'John', email: 'john@example.com' },
    theme: 'light',
  });
  
  // Dispatch never changes
  return (
    <StateContext.Provider value={state}>
      <DispatchContext.Provider value={dispatch}>
        {children}
      </DispatchContext.Provider>
    </StateContext.Provider>
  );
}

// Custom hooks
function useAppState() {
  const context = useContext(StateContext);
  if (!context) throw new Error('Missing provider');
  return context;
}

function useAppDispatch() {
  const context = useContext(DispatchContext);
  if (!context) throw new Error('Missing provider');
  return context;
}

// Usage
function UserGreeting() {
  const { user } = useAppState();
  return <div>Hello, {user.name}!</div>;
}

function ThemeToggle() {
  const { theme } = useAppState();
  const dispatch = useAppDispatch();
  
  return (
    <button onClick={() => 
      dispatch({ type: 'SET_THEME', payload: theme === 'light' ? 'dark' : 'light' })
    }>
      Toggle Theme
    </button>
  );
}
```

#### 5. Alternative Solutions

**When Context is Too Heavy:**
```typescript
/*
Consider alternatives for:

1. Frequently changing values:
   → Use state management library (Zustand, Jotai)
   → Pass props directly
   → Use composition

2. Many components need same data:
   → React Query for server state
   → Zustand for client state
   → Recoil for atomic state

3. Complex state logic:
   → Redux Toolkit
   → MobX
   → XState
*/
```

**Zustand (Lightweight Alternative):**
```typescript
import create from 'zustand';

// Create store
const useStore = create<{
  user: { name: string };
  theme: string;
  setUser: (user: any) => void;
  setTheme: (theme: string) => void;
}>((set) => ({
  user: { name: 'John' },
  theme: 'light',
  setUser: (user) => set({ user }),
  setTheme: (theme) => set({ theme }),
}));

// Component only subscribes to theme
function ThemeToggle() {
  const theme = useStore(state => state.theme);
  const setTheme = useStore(state => state.setTheme);
  
  // ✅ Only re-renders when theme changes!
  return <button onClick={() => setTheme('dark')}>Theme: {theme}</button>;
}

// Component only subscribes to user
function UserGreeting() {
  const user = useStore(state => state.user);
  
  // ✅ Only re-renders when user changes!
  return <div>Hello, {user.name}!</div>;
}
```

#### 6. Real-World Example

**E-commerce App:**
```typescript
// Split contexts by concern

// 1. User context (changes rarely)
const UserContext = createContext(null);

function UserProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState(null);
  
  const value = useMemo(() => ({ user, setUser }), [user]);
  
  return <UserContext.Provider value={value}>{children}</UserContext.Provider>;
}

// 2. Cart context (changes frequently)
const CartContext = createContext(null);

function CartProvider({ children }: { children: React.ReactNode }) {
  const [items, setItems] = useState([]);
  
  const value = useMemo(
    () => ({
      items,
      addItem: (item) => setItems(prev => [...prev, item]),
      removeItem: (id) => setItems(prev => prev.filter(i => i.id !== id)),
    }),
    [items]
  );
  
  return <CartContext.Provider value={value}>{children}</CartContext.Provider>;
}

// 3. Theme context (changes occasionally)
const ThemeContext = createContext('light');

// Result:
// - Product list doesn't re-render when cart changes
// - Cart doesn't re-render when theme changes
// - User info doesn't re-render when cart changes
```

#### 7. Measuring Performance

**React DevTools Profiler:**
```typescript
import { Profiler } from 'react';

function onRenderCallback(
  id: string,
  phase: 'mount' | 'update',
  actualDuration: number,
  baseDuration: number,
  startTime: number,
  commitTime: number
) {
  console.log(`${id} (${phase})`, {
    actualDuration,
    baseDuration,
  });
}

function App() {
  return (
    <Profiler id="App" onRender={onRenderCallback}>
      <UserProvider>
        <YourComponents />
      </UserProvider>
    </Profiler>
  );
}
```

**Console Logging:**
```typescript
function Component() {
  const context = useContext(MyContext);
  
  console.log('Component rendered', {
    timestamp: Date.now(),
    contextValue: context,
  });
  
  return <div>...</div>;
}

// Check console for unexpected renders
```

#### 8. Best Practices

**Checklist:**
```typescript
/*
✅ Split contexts by concern (user, theme, settings)
✅ Memoize context values with useMemo
✅ Separate state and dispatch contexts
✅ Use React.memo for expensive components
✅ Consider alternatives for frequently changing values
✅ Profile performance with React DevTools
✅ Keep context close to consumers (not at root)
✅ Document when context causes re-renders
*/
```

**When to Use Context:**
```typescript
/*
✅ GOOD use cases:
- Theme (changes rarely)
- User authentication (changes on login/logout)
- Locale/language (changes rarely)
- Feature flags (static)
- Global settings (changes rarely)

❌ AVOID for:
- Frequently changing values (form inputs)
- High-frequency updates (animations, counters)
- Large component trees
- Performance-critical paths
*/
```

**When to Use Alternatives:**
```typescript
/*
Use state management library when:
- Many frequent updates
- Complex state logic
- Need devtools
- Need middleware
- Need selectors
- Large application

Library recommendations:
- Zustand: Simple, lightweight
- Jotai: Atomic state
- Redux Toolkit: Complex apps
- React Query: Server state
*/
```

#### Summary

**Performance Issues:**
- All consumers re-render when context changes
- Can't selectively subscribe to parts of context
- Reference equality causes unnecessary updates
- Not optimized for frequent changes

**Solutions:**
1. **Split contexts** by concern
2. **Memoize values** with useMemo
3. **Separate state and dispatch**
4. **Use selectors** (with libraries)
5. **React.memo** for expensive components
6. **Consider alternatives** for frequent updates

**Best Practices:**
- Use Context for rarely-changing global state
- Split contexts instead of one large context
- Memoize context values properly
- Profile and measure impact
- Consider Zustand/Jotai for complex state

**Key Insight:**
> "Context is great for avoiding prop drilling, but not optimized for frequent updates or large component trees."

**Golden Rule:**
> "If you're worried about Context performance, you probably need a state management library instead!"

</details>

---

### 155. How do you optimize rendering performance at scale?

<details>
<summary>View Answer</summary>

**Optimizing rendering performance at scale** requires a combination of techniques to handle large datasets, complex component trees, and frequent updates efficiently. This involves virtualization, memoization, batching, and architectural patterns.

#### 1. Virtual Lists and Windows

**React Window (Recommended):**
```typescript
import { FixedSizeList } from 'react-window';

// Render only visible rows
function VirtualList({ items }: { items: any[] }) {
  const Row = ({ index, style }: { index: number; style: React.CSSProperties }) => (
    <div style={style}>
      Item {index}: {items[index].name}
    </div>
  );
  
  return (
    <FixedSizeList
      height={600}        // Viewport height
      itemCount={items.length}  // Total items (10,000+)
      itemSize={50}       // Row height
      width="100%"
    >
      {Row}
    </FixedSizeList>
  );
}

// Without virtualization: Renders 10,000 DOM nodes
// With virtualization: Renders ~12 visible nodes
// Result: 99% fewer DOM nodes, instant render
```

**Variable Size Lists:**
```typescript
import { VariableSizeList } from 'react-window';

function DynamicList({ items }: { items: any[] }) {
  // Calculate row height dynamically
  const getItemSize = (index: number) => {
    return items[index].content.length > 100 ? 100 : 50;
  };
  
  const Row = ({ index, style }: { index: number; style: React.CSSProperties }) => (
    <div style={style}>
      <h3>{items[index].title}</h3>
      <p>{items[index].content}</p>
    </div>
  );
  
  return (
    <VariableSizeList
      height={600}
      itemCount={items.length}
      itemSize={getItemSize}
      width="100%"
    >
      {Row}
    </VariableSizeList>
  );
}
```

**React Virtuoso (Advanced):**
```typescript
import { Virtuoso } from 'react-virtuoso';

function InfiniteList() {
  const [items, setItems] = useState<string[]>([]);
  
  const loadMore = () => {
    // Load more items
    fetchItems().then(newItems => {
      setItems(prev => [...prev, ...newItems]);
    });
  };
  
  return (
    <Virtuoso
      data={items}
      endReached={loadMore}
      itemContent={(index, item) => (
        <div>
          Item {index}: {item}
        </div>
      )}
    />
  );
}
```

#### 2. Memoization Strategies

**React.memo for Components:**
```typescript
import { memo } from 'react';

// Expensive component
const ProductCard = memo(function ProductCard({
  product,
  onAddToCart,
}: {
  product: Product;
  onAddToCart: (id: string) => void;
}) {
  console.log('ProductCard rendered:', product.id);
  
  return (
    <div className="card">
      <img src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <button onClick={() => onAddToCart(product.id)}>
        Add to Cart
      </button>
    </div>
  );
});

// Custom comparison
const ProductCardOptimized = memo(
  ProductCard,
  (prevProps, nextProps) => {
    // Only re-render if product or callback changed
    return (
      prevProps.product.id === nextProps.product.id &&
      prevProps.product.price === nextProps.product.price &&
      prevProps.onAddToCart === nextProps.onAddToCart
    );
  }
);
```

**useMemo for Expensive Calculations:**
```typescript
import { useMemo } from 'react';

function DataAnalytics({ data }: { data: number[] }) {
  // Without memo: Recalculates on every render
  // const stats = calculateStatistics(data); // ❌ Slow!
  
  // With memo: Only recalculates when data changes
  const stats = useMemo(() => {
    console.log('Calculating statistics...');
    return {
      mean: data.reduce((a, b) => a + b, 0) / data.length,
      median: data.sort()[Math.floor(data.length / 2)],
      min: Math.min(...data),
      max: Math.max(...data),
    };
  }, [data]);
  
  return (
    <div>
      <p>Mean: {stats.mean}</p>
      <p>Median: {stats.median}</p>
      <p>Min: {stats.min}</p>
      <p>Max: {stats.max}</p>
    </div>
  );
}
```

**useCallback for Stable References:**
```typescript
import { useCallback, useState } from 'react';

function ParentComponent() {
  const [count, setCount] = useState(0);
  const [items, setItems] = useState<string[]>([]);
  
  // ❌ BAD: New function every render
  // const handleClick = (id: string) => {
  //   console.log('Clicked:', id);
  // };
  
  // ✅ GOOD: Stable function reference
  const handleClick = useCallback((id: string) => {
    console.log('Clicked:', id);
  }, []); // Never changes
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>
        Count: {count}
      </button>
      
      {/* Children don't re-render when count changes */}
      {items.map(item => (
        <MemoizedChild key={item} id={item} onClick={handleClick} />
      ))}
    </div>
  );
}
```

#### 3. Debouncing and Throttling

**Debounce Input:**
```typescript
import { useState, useEffect } from 'react';
import { debounce } from 'lodash';

function SearchComponent() {
  const [searchTerm, setSearchTerm] = useState('');
  const [results, setResults] = useState<any[]>([]);
  
  // Debounce search - only search after user stops typing
  useEffect(() => {
    const debouncedSearch = debounce(async (term: string) => {
      if (term.length > 2) {
        const data = await fetchSearchResults(term);
        setResults(data);
      }
    }, 300); // Wait 300ms after last keystroke
    
    debouncedSearch(searchTerm);
    
    return () => debouncedSearch.cancel();
  }, [searchTerm]);
  
  return (
    <div>
      <input
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        placeholder="Search..."
      />
      <SearchResults results={results} />
    </div>
  );
}

// Without debounce: API call on every keystroke
// With debounce: API call only after user stops typing
```

**Throttle Scroll Events:**
```typescript
import { useEffect } from 'react';
import { throttle } from 'lodash';

function InfiniteScroll() {
  useEffect(() => {
    const handleScroll = throttle(() => {
      const { scrollTop, scrollHeight, clientHeight } = document.documentElement;
      
      if (scrollTop + clientHeight >= scrollHeight - 100) {
        loadMoreItems();
      }
    }, 200); // Execute at most once every 200ms
    
    window.addEventListener('scroll', handleScroll);
    return () => {
      window.removeEventListener('scroll', handleScroll);
      handleScroll.cancel();
    };
  }, []);
  
  return <div>{/* Content */}</div>;
}
```

#### 4. Batch Updates

**React 18 Automatic Batching:**
```typescript
// React 18: Automatically batches all updates
function Component() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);
  
  const handleClick = () => {
    // These updates are batched automatically
    setCount(c => c + 1);
    setFlag(f => !f);
    // Only 1 re-render!
  };
  
  // Even in async functions
  const handleAsync = async () => {
    await fetchData();
    setCount(c => c + 1);  // Batched
    setFlag(f => !f);      // Batched
    // Only 1 re-render!
  };
  
  return <button onClick={handleClick}>Update</button>;
}
```

**Manual Batching (React 17):**
```typescript
import { unstable_batchedUpdates } from 'react-dom';

function Component() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);
  
  const handleAsync = async () => {
    const data = await fetchData();
    
    // Manually batch updates
    unstable_batchedUpdates(() => {
      setCount(data.count);
      setFlag(data.flag);
    });
    // Only 1 re-render
  };
  
  return <button onClick={handleAsync}>Fetch</button>;
}
```

#### 5. Code Splitting and Lazy Loading

**Route-Based Splitting:**
```typescript
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';

// Load routes on demand
const Home = lazy(() => import('./pages/Home'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Analytics = lazy(() => import('./pages/Analytics'));
const Reports = lazy(() => import('./pages/Reports'));

function App() {
  return (
    <Suspense fallback={<PageLoader />}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/analytics" element={<Analytics />} />
        <Route path="/reports" element={<Reports />} />
      </Routes>
    </Suspense>
  );
}

// Each route loads only when visited
// Reduces initial bundle size by 70%+
```

**Component-Level Splitting:**
```typescript
import { lazy, Suspense } from 'react';

// Heavy components loaded on demand
const RichTextEditor = lazy(() => import('./RichTextEditor'));
const VideoPlayer = lazy(() => import('./VideoPlayer'));
const PDFViewer = lazy(() => import('./PDFViewer'));

function DocumentEditor({ type }: { type: string }) {
  return (
    <Suspense fallback={<ComponentLoader />}>
      {type === 'text' && <RichTextEditor />}
      {type === 'video' && <VideoPlayer />}
      {type === 'pdf' && <PDFViewer />}
    </Suspense>
  );
}
```

#### 6. Optimize Large Lists

**Windowing with react-window:**
```typescript
import { FixedSizeList as List } from 'react-window';
import AutoSizer from 'react-virtualized-auto-sizer';

function LargeDataTable({ data }: { data: any[] }) {
  const Row = ({ index, style }: any) => (
    <div style={style} className="row">
      <span>{data[index].id}</span>
      <span>{data[index].name}</span>
      <span>{data[index].email}</span>
      <span>{data[index].status}</span>
    </div>
  );
  
  return (
    <AutoSizer>
      {({ height, width }) => (
        <List
          height={height}
          itemCount={data.length}
          itemSize={35}
          width={width}
        >
          {Row}
        </List>
      )}
    </AutoSizer>
  );
}

// Handles 100,000+ rows smoothly
// Only renders ~20 visible rows
```

**Pagination Instead of Infinite Scroll:**
```typescript
function PaginatedList({ pageSize = 20 }: { pageSize?: number }) {
  const [page, setPage] = useState(1);
  const { data, isLoading } = useQuery(
    ['items', page],
    () => fetchItems(page, pageSize)
  );
  
  return (
    <div>
      <ItemList items={data?.items || []} />
      
      <div className="pagination">
        <button
          disabled={page === 1}
          onClick={() => setPage(p => p - 1)}
        >
          Previous
        </button>
        <span>Page {page}</span>
        <button
          disabled={!data?.hasMore}
          onClick={() => setPage(p => p + 1)}
        >
          Next
        </button>
      </div>
    </div>
  );
}

// Only 20 items in DOM at a time
// Better performance than loading everything
```

#### 7. Concurrent Features (React 18)

**useTransition for Non-Urgent Updates:**
```typescript
import { useState, useTransition } from 'react';

function SearchResults() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState<any[]>([]);
  const [isPending, startTransition] = useTransition();
  
  const handleSearch = (value: string) => {
    // Urgent: Update input immediately
    setQuery(value);
    
    // Non-urgent: Mark as transition
    startTransition(() => {
      // Heavy filtering operation
      const filtered = heavyFilterOperation(value);
      setResults(filtered);
    });
  };
  
  return (
    <div>
      <input
        value={query}
        onChange={(e) => handleSearch(e.target.value)}
        placeholder="Search..."
      />
      
      {isPending && <div>Updating results...</div>}
      
      <ResultsList results={results} />
    </div>
  );
}

// Input stays responsive while filtering
// Smooth typing experience even with 10,000+ items
```

**useDeferredValue:**
```typescript
import { useState, useDeferredValue, useMemo } from 'react';

function FilteredList({ items }: { items: any[] }) {
  const [filter, setFilter] = useState('');
  const deferredFilter = useDeferredValue(filter);
  
  // Use deferred value for expensive operation
  const filteredItems = useMemo(
    () => items.filter(item => 
      item.name.toLowerCase().includes(deferredFilter.toLowerCase())
    ),
    [items, deferredFilter]
  );
  
  return (
    <div>
      <input
        value={filter}
        onChange={(e) => setFilter(e.target.value)}
      />
      
      {filter !== deferredFilter && <div>Updating...</div>}
      
      <ul>
        {filteredItems.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </div>
  );
}

// Input updates immediately
// List updates when React has time
```

#### 8. Optimize Context Usage

**Split Contexts:**
```typescript
// ❌ BAD: Single context with everything
const AppContext = createContext({ user, theme, settings, cart });
// Change cart → Everything re-renders

// ✅ GOOD: Separate contexts
const UserContext = createContext(null);
const ThemeContext = createContext(null);
const SettingsContext = createContext(null);
const CartContext = createContext(null);

// Change cart → Only cart consumers re-render
```

**Memoize Context Values:**
```typescript
function Provider({ children }: { children: React.ReactNode }) {
  const [state, setState] = useState(initialState);
  
  // ✅ Memoize to prevent unnecessary re-renders
  const value = useMemo(
    () => ({ state, setState }),
    [state]
  );
  
  return <Context.Provider value={value}>{children}</Context.Provider>;
}
```

#### 9. Web Workers for Heavy Computation

**Offload to Web Worker:**
```typescript
// worker.ts
self.addEventListener('message', (e) => {
  const { data, action } = e.data;
  
  if (action === 'process') {
    // Heavy computation
    const result = expensiveCalculation(data);
    self.postMessage({ result });
  }
});

// Component.tsx
import { useEffect, useState } from 'react';

function HeavyComponent({ data }: { data: number[] }) {
  const [result, setResult] = useState(null);
  const [worker, setWorker] = useState<Worker | null>(null);
  
  useEffect(() => {
    const w = new Worker(new URL('./worker.ts', import.meta.url));
    
    w.addEventListener('message', (e) => {
      setResult(e.data.result);
    });
    
    setWorker(w);
    
    return () => w.terminate();
  }, []);
  
  useEffect(() => {
    if (worker) {
      worker.postMessage({ action: 'process', data });
    }
  }, [data, worker]);
  
  return <div>{result}</div>;
}

// Heavy computation doesn't block UI
```

#### 10. Measure and Monitor

**React DevTools Profiler:**
```typescript
import { Profiler } from 'react';

function onRenderCallback(
  id: string,
  phase: 'mount' | 'update',
  actualDuration: number,
  baseDuration: number,
  startTime: number,
  commitTime: number
) {
  console.log(`${id} rendered in ${actualDuration}ms`);
  
  // Send to analytics
  analytics.track('component_render', {
    id,
    phase,
    duration: actualDuration,
  });
}

function App() {
  return (
    <Profiler id="App" onRender={onRenderCallback}>
      <YourComponents />
    </Profiler>
  );
}
```

**Performance Monitoring:**
```typescript
function ComponentWithMetrics() {
  useEffect(() => {
    // Measure component mount time
    const startTime = performance.now();
    
    return () => {
      const duration = performance.now() - startTime;
      console.log(`Component was mounted for ${duration}ms`);
    };
  }, []);
  
  return <div>{/* Content */}</div>;
}
```

#### 11. Architecture Patterns

**Component Composition:**
```typescript
// ❌ BAD: Props drilling
function App() {
  const [user, setUser] = useState(null);
  return <Layout user={user} setUser={setUser} />;
}

function Layout({ user, setUser }) {
  return <Sidebar user={user} setUser={setUser} />;
}

function Sidebar({ user, setUser }) {
  return <UserMenu user={user} setUser={setUser} />;
}

// ✅ GOOD: Composition
function App() {
  const [user, setUser] = useState(null);
  return (
    <Layout>
      <Sidebar>
        <UserMenu user={user} setUser={setUser} />
      </Sidebar>
    </Layout>
  );
}

function Layout({ children }) {
  return <div className="layout">{children}</div>;
}

function Sidebar({ children }) {
  return <aside>{children}</aside>;
}

// Layout and Sidebar don't re-render on user change
```

**State Colocation:**
```typescript
// ❌ BAD: State at app level
function App() {
  const [modalOpen, setModalOpen] = useState(false);
  const [formData, setFormData] = useState({});
  
  return (
    <div>
      <HomePage />
      {modalOpen && <Modal data={formData} />}
    </div>
  );
}

// ✅ GOOD: State close to where it's used
function App() {
  return <HomePage />;
}

function HomePage() {
  const [modalOpen, setModalOpen] = useState(false);
  const [formData, setFormData] = useState({});
  
  return (
    <div>
      {/* Content */}
      {modalOpen && <Modal data={formData} />}
    </div>
  );
}

// App doesn't re-render when modal state changes
```

#### 12. Best Practices Checklist

```typescript
/*
Rendering Performance Optimization:

✅ Use virtualization for large lists (react-window)
✅ Memoize expensive components (React.memo)
✅ Memoize expensive calculations (useMemo)
✅ Stabilize callback references (useCallback)
✅ Debounce/throttle frequent events
✅ Use code splitting (lazy + Suspense)
✅ Split contexts by concern
✅ Use useTransition for non-urgent updates
✅ Implement pagination for large datasets
✅ Colocate state close to usage
✅ Use composition over prop drilling
✅ Profile and measure performance
✅ Offload heavy computation to Web Workers
✅ Optimize images and assets
✅ Monitor production performance
*/
```

#### Summary

**Key Techniques:**

1. **Virtualization**
   - react-window for lists
   - Only render visible items
   - Handles 100,000+ items

2. **Memoization**
   - React.memo for components
   - useMemo for calculations
   - useCallback for functions

3. **Debouncing/Throttling**
   - Debounce input
   - Throttle scroll
   - Reduce update frequency

4. **Code Splitting**
   - Route-based splitting
   - Component-level splitting
   - Lazy loading

5. **Concurrent Features**
   - useTransition
   - useDeferredValue
   - Automatic batching

6. **Architecture**
   - State colocation
   - Component composition
   - Split contexts

**Performance Metrics:**
- First Contentful Paint (FCP): < 1.8s
- Time to Interactive (TTI): < 3.8s
- Total Blocking Time (TBT): < 200ms
- Largest Contentful Paint (LCP): < 2.5s

**Golden Rules:**
> "Measure first, optimize second. Don't guess, use the Profiler!"

> "Render less, render faster. Virtualize large lists, memoize expensive components."

</details>

---

### 156. What is the profiler API in React?

<details>
<summary>View Answer</summary>

**The Profiler API** is a React tool that measures the performance of your application by tracking how often components render and how long each render takes. It helps identify performance bottlenecks and optimize slow components.

#### 1. Basic Usage

**Simple Profiler:**
```typescript
import { Profiler, ProfilerOnRenderCallback } from 'react';

function onRenderCallback(
  id: string,                   // Profiler id
  phase: 'mount' | 'update',   // Mount or update phase
  actualDuration: number,       // Time spent rendering
  baseDuration: number,         // Estimated time without memoization
  startTime: number,            // When render started
  commitTime: number,           // When render committed
  interactions: Set<any>        // Interactions that caused render
) {
  console.log(`${id} (${phase}) took ${actualDuration}ms`);
}

function App() {
  return (
    <Profiler id="App" onRender={onRenderCallback}>
      <YourComponents />
    </Profiler>
  );
}

// Output:
// App (mount) took 45.2ms
// App (update) took 12.3ms
```

**Multiple Profilers:**
```typescript
function Dashboard() {
  return (
    <div>
      <Profiler id="Header" onRender={onRenderCallback}>
        <Header />
      </Profiler>
      
      <Profiler id="Sidebar" onRender={onRenderCallback}>
        <Sidebar />
      </Profiler>
      
      <Profiler id="MainContent" onRender={onRenderCallback}>
        <MainContent />
      </Profiler>
    </div>
  );
}

// Tracks each section independently
```

#### 2. Understanding the Metrics

**Parameter Explanation:**
```typescript
function onRenderCallback(
  id: string,
  phase: 'mount' | 'update',
  actualDuration: number,
  baseDuration: number,
  startTime: number,
  commitTime: number,
  interactions: Set<any>
) {
  /*
  id: 'MyComponent'
  - Identifier for the Profiler tree
  - Useful when using multiple Profilers
  
  phase: 'mount' | 'update'
  - 'mount': First render
  - 'update': Re-render
  
  actualDuration: 23.5
  - Actual time (ms) to render this update
  - Includes child components
  - Lower is better
  
  baseDuration: 45.2
  - Estimated time without optimizations
  - Time if all descendants rendered
  - Compare with actualDuration to see optimization impact
  
  startTime: 1234567.89
  - When React began rendering
  - Timestamp from performance.now()
  
  commitTime: 1234591.23
  - When React committed the update
  - Timestamp from performance.now()
  
  interactions: Set { { id: 'user-click', ... } }
  - User interactions that triggered this render
  - Requires React DevTools Profiler
  */
}
```

**Interpreting Results:**
```typescript
function analyzePerformance(
  id: string,
  phase: 'mount' | 'update',
  actualDuration: number,
  baseDuration: number
) {
  // Check if component is slow
  if (actualDuration > 16) {
    console.warn(`${id} took ${actualDuration}ms (> 16ms frame budget)`);
  }
  
  // Check optimization effectiveness
  if (phase === 'update') {
    const saved = baseDuration - actualDuration;
    const percentage = (saved / baseDuration) * 100;
    
    console.log(`${id} optimization saved ${saved.toFixed(2)}ms (${percentage.toFixed(1)}%)`);
    
    if (percentage < 20) {
      console.warn(`${id} has low optimization effectiveness`);
    }
  }
}
```

#### 3. Production Monitoring

**Send Metrics to Analytics:**
```typescript
import { Profiler } from 'react';

function onRenderCallback(
  id: string,
  phase: 'mount' | 'update',
  actualDuration: number,
  baseDuration: number,
  startTime: number,
  commitTime: number
) {
  // Send to analytics service
  if (actualDuration > 16) {
    analytics.track('slow_render', {
      componentId: id,
      phase,
      duration: actualDuration,
      timestamp: commitTime,
    });
  }
  
  // Send to monitoring service
  performance.measure(
    `${id}-${phase}`,
    {
      start: startTime,
      end: commitTime,
      detail: { actualDuration, baseDuration },
    }
  );
}

function App() {
  return (
    <Profiler id="App" onRender={onRenderCallback}>
      <YourApp />
    </Profiler>
  );
}
```

**Conditional Profiling:**
```typescript
import { Profiler } from 'react';

// Only profile in development or with flag
const ENABLE_PROFILING = 
  process.env.NODE_ENV === 'development' ||
  window.location.search.includes('profile=true');

function ConditionalProfiler({
  children,
  id,
}: {
  children: React.ReactNode;
  id: string;
}) {
  if (!ENABLE_PROFILING) {
    return <>{children}</>;
  }
  
  return (
    <Profiler id={id} onRender={onRenderCallback}>
      {children}
    </Profiler>
  );
}

// Usage
function App() {
  return (
    <ConditionalProfiler id="App">
      <YourApp />
    </ConditionalProfiler>
  );
}
```

#### 4. Advanced Profiling

**Track Render Count:**
```typescript
import { useRef } from 'react';

function ProfiledComponent({ id }: { id: string }) {
  const renderCountRef = useRef(0);
  
  const onRender = (
    id: string,
    phase: 'mount' | 'update',
    actualDuration: number
  ) => {
    renderCountRef.current++;
    
    console.log(`${id} render #${renderCountRef.current}: ${actualDuration}ms`);
    
    if (renderCountRef.current > 10) {
      console.warn(`${id} has rendered ${renderCountRef.current} times!`);
    }
  };
  
  return (
    <Profiler id={id} onRender={onRender}>
      <YourComponent />
    </Profiler>
  );
}
```

**Aggregate Metrics:**
```typescript
class PerformanceMonitor {
  private metrics: Map<string, number[]> = new Map();
  
  record(id: string, duration: number) {
    if (!this.metrics.has(id)) {
      this.metrics.set(id, []);
    }
    this.metrics.get(id)!.push(duration);
  }
  
  getStats(id: string) {
    const durations = this.metrics.get(id) || [];
    
    if (durations.length === 0) return null;
    
    const sorted = [...durations].sort((a, b) => a - b);
    
    return {
      count: durations.length,
      avg: durations.reduce((a, b) => a + b, 0) / durations.length,
      min: sorted[0],
      max: sorted[sorted.length - 1],
      p50: sorted[Math.floor(sorted.length * 0.5)],
      p95: sorted[Math.floor(sorted.length * 0.95)],
      p99: sorted[Math.floor(sorted.length * 0.99)],
    };
  }
  
  report() {
    for (const [id, durations] of this.metrics.entries()) {
      console.log(`\n${id}:`, this.getStats(id));
    }
  }
}

const monitor = new PerformanceMonitor();

function onRenderCallback(
  id: string,
  phase: 'mount' | 'update',
  actualDuration: number
) {
  monitor.record(id, actualDuration);
}

// Later, generate report
monitor.report();
/*
Output:
Header:
  count: 24
  avg: 3.2ms
  min: 1.1ms
  max: 12.5ms
  p50: 2.8ms
  p95: 8.1ms
  p99: 11.2ms
*/
```

#### 5. Nested Profilers

**Profile Subtrees:**
```typescript
function App() {
  return (
    <Profiler id="App" onRender={onRenderCallback}>
      <Header />
      
      <Profiler id="Dashboard" onRender={onRenderCallback}>
        <Sidebar />
        
        <Profiler id="MainContent" onRender={onRenderCallback}>
          <ContentArea />
        </Profiler>
      </Profiler>
      
      <Footer />
    </Profiler>
  );
}

// Tracks:
// - App (entire tree)
// - Dashboard (sidebar + content)
// - MainContent (just content)
```

**Selective Profiling:**
```typescript
function ProfileIfSlow({
  children,
  id,
  threshold = 16,
}: {
  children: React.ReactNode;
  id: string;
  threshold?: number;
}) {
  const onRender = (
    id: string,
    phase: 'mount' | 'update',
    actualDuration: number
  ) => {
    if (actualDuration > threshold) {
      console.warn(
        `⚠️ ${id} exceeded ${threshold}ms threshold: ${actualDuration}ms`
      );
    }
  };
  
  return (
    <Profiler id={id} onRender={onRender}>
      {children}
    </Profiler>
  );
}

// Usage
<ProfileIfSlow id="ExpensiveComponent" threshold={50}>
  <ExpensiveComponent />
</ProfileIfSlow>
```

#### 6. React DevTools Profiler

**Using DevTools:**
```typescript
/*
React DevTools Profiler Features:

1. Flame Graph:
   - Visual representation of render time
   - Color-coded by duration
   - Click to see details

2. Ranked Chart:
   - Components sorted by render time
   - Identify slowest components

3. Component Chart:
   - Individual component render history
   - See when and why component rendered

4. Interactions:
   - Track user interactions
   - See what triggered renders

How to use:
1. Open React DevTools
2. Click "Profiler" tab
3. Click record button (⚫)
4. Interact with app
5. Click stop button (⏹)
6. Analyze results
*/
```

**Enable Profiling in Production:**
```javascript
// webpack.config.js
module.exports = {
  resolve: {
    alias: {
      // Use profiling build in production
      'react-dom$': 'react-dom/profiling',
      'scheduler/tracing': 'scheduler/tracing-profiling',
    },
  },
};

// Note: Slightly larger bundle size
// Only enable when actively profiling production
```

#### 7. Practical Examples

**Identify Unnecessary Renders:**
```typescript
import { useRef } from 'react';

function ComponentWithTracking({ data }: { data: any }) {
  const renderCount = useRef(0);
  const prevData = useRef(data);
  
  const onRender = (
    id: string,
    phase: 'mount' | 'update',
    actualDuration: number
  ) => {
    renderCount.current++;
    
    if (phase === 'update') {
      const dataChanged = prevData.current !== data;
      
      if (!dataChanged) {
        console.warn(
          `${id} re-rendered without data change! ` +
          `Render #${renderCount.current}`
        );
      }
    }
    
    prevData.current = data;
  };
  
  return (
    <Profiler id="ComponentWithTracking" onRender={onRender}>
      <div>{JSON.stringify(data)}</div>
    </Profiler>
  );
}
```

**Compare Before/After Optimization:**
```typescript
const optimizationMetrics = {
  before: [] as number[],
  after: [] as number[],
};

function ProfiledComponent({ version }: { version: 'before' | 'after' }) {
  const onRender = (
    id: string,
    phase: 'mount' | 'update',
    actualDuration: number
  ) => {
    if (phase === 'update') {
      optimizationMetrics[version].push(actualDuration);
    }
  };
  
  return (
    <Profiler id={version} onRender={onRender}>
      {version === 'before' ? <UnoptimizedComponent /> : <OptimizedComponent />}
    </Profiler>
  );
}

// After testing:
function compareResults() {
  const beforeAvg = 
    optimizationMetrics.before.reduce((a, b) => a + b, 0) / 
    optimizationMetrics.before.length;
    
  const afterAvg = 
    optimizationMetrics.after.reduce((a, b) => a + b, 0) / 
    optimizationMetrics.after.length;
    
  const improvement = ((beforeAvg - afterAvg) / beforeAvg) * 100;
  
  console.log(`Before: ${beforeAvg.toFixed(2)}ms`);
  console.log(`After: ${afterAvg.toFixed(2)}ms`);
  console.log(`Improvement: ${improvement.toFixed(1)}%`);
}
```

#### 8. Performance Budget

**Set Thresholds:**
```typescript
const PERFORMANCE_BUDGET = {
  mount: 100,   // 100ms for initial mount
  update: 16,   // 16ms for updates (60fps)
  warning: 50,  // Warn if over 50ms
};

function onRenderCallback(
  id: string,
  phase: 'mount' | 'update',
  actualDuration: number
) {
  const budget = PERFORMANCE_BUDGET[phase];
  
  if (actualDuration > budget) {
    console.error(
      `❌ ${id} exceeded ${phase} budget: ` +
      `${actualDuration.toFixed(2)}ms > ${budget}ms`
    );
    
    // Send alert in production
    if (process.env.NODE_ENV === 'production') {
      alerting.send({
        type: 'performance_budget_exceeded',
        component: id,
        phase,
        duration: actualDuration,
        budget,
      });
    }
  } else if (actualDuration > PERFORMANCE_BUDGET.warning) {
    console.warn(
      `⚠️ ${id} is slow: ${actualDuration.toFixed(2)}ms`
    );
  } else {
    console.log(
      `✅ ${id} within budget: ${actualDuration.toFixed(2)}ms`
    );
  }
}
```

#### 9. Integration with Monitoring Services

**Sentry:**
```typescript
import * as Sentry from '@sentry/react';

function onRenderCallback(
  id: string,
  phase: 'mount' | 'update',
  actualDuration: number
) {
  // Send to Sentry
  Sentry.addBreadcrumb({
    category: 'performance',
    message: `${id} rendered`,
    level: 'info',
    data: {
      phase,
      duration: actualDuration,
    },
  });
  
  // Report slow renders
  if (actualDuration > 50) {
    Sentry.captureMessage(
      `Slow render: ${id}`,
      {
        level: 'warning',
        tags: { component: id, phase },
        extra: { duration: actualDuration },
      }
    );
  }
}
```

**Custom Analytics:**
```typescript
function onRenderCallback(
  id: string,
  phase: 'mount' | 'update',
  actualDuration: number,
  baseDuration: number
) {
  // Send to analytics
  analytics.track('component_render', {
    component_id: id,
    render_phase: phase,
    actual_duration: actualDuration,
    base_duration: baseDuration,
    optimization_percentage: 
      ((baseDuration - actualDuration) / baseDuration) * 100,
  });
}
```

#### 10. Best Practices

**Checklist:**
```typescript
/*
✅ Use Profiler in development to identify bottlenecks
✅ Profile after optimization to verify improvements
✅ Set performance budgets for components
✅ Monitor production performance selectively
✅ Use nested Profilers for granular tracking
✅ Aggregate metrics for statistical analysis
✅ Compare actualDuration vs baseDuration
✅ Send slow renders to monitoring service
✅ Use React DevTools Profiler for visual analysis
✅ Remove Profilers in production (or use conditionally)
*/
```

#### Summary

**Profiler API:**
- Measures component render performance
- Tracks mount and update phases
- Provides timing metrics
- Helps identify bottlenecks

**Key Metrics:**
- **actualDuration**: Actual render time
- **baseDuration**: Time without optimizations
- **phase**: 'mount' or 'update'
- **startTime/commitTime**: Timing info

**Use Cases:**
1. Identify slow components
2. Measure optimization impact
3. Monitor production performance
4. Set performance budgets
5. Track render frequency

**Tools:**
- Profiler component (API)
- React DevTools Profiler (visual)
- Chrome DevTools Performance
- Custom monitoring solutions

**Best Practices:**
- Profile during development
- Set performance thresholds
- Monitor critical paths
- Compare before/after optimizations
- Send metrics to analytics

**Key Insight:**
> "Don't guess where the bottlenecks are—use the Profiler to measure and identify them!"

**Golden Rule:**
> "A render under 16ms keeps the UI smooth. Profile, optimize, verify!"

</details>

---

### 157. How do you implement server-side caching strategies?

<details>
<summary>View Answer</summary>

**Server-side caching** stores rendered content or data on the server/CDN to avoid redundant processing. This dramatically reduces response times, server load, and costs while improving user experience.

#### 1. HTTP Cache Headers

**Cache-Control:**
```typescript
// Express.js
import express from 'express';

const app = express();

// Static assets - cache aggressively
app.use('/static', express.static('public', {
  maxAge: '1y',  // Cache for 1 year
  immutable: true,
}));

// API endpoint with caching
app.get('/api/products', (req, res) => {
  const products = getProducts();
  
  res.set({
    'Cache-Control': 'public, max-age=300, s-maxage=3600',
    // public: Can be cached by CDN
    // max-age=300: Browser caches 5 minutes
    // s-maxage=3600: CDN caches 1 hour
  });
  
  res.json(products);
});

// User-specific data - no cache
app.get('/api/user/profile', authenticateUser, (req, res) => {
  const profile = getUserProfile(req.userId);
  
  res.set({
    'Cache-Control': 'private, no-cache',
    // private: Only browser can cache
    // no-cache: Revalidate before using
  });
  
  res.json(profile);
});
```

**ETag for Conditional Requests:**
```typescript
import crypto from 'crypto';

app.get('/api/articles/:id', (req, res) => {
  const article = getArticle(req.params.id);
  
  // Generate ETag from content
  const etag = crypto
    .createHash('md5')
    .update(JSON.stringify(article))
    .digest('hex');
  
  // Check if client has current version
  if (req.headers['if-none-match'] === etag) {
    // Content hasn't changed
    return res.status(304).end();
  }
  
  res.set({
    'ETag': etag,
    'Cache-Control': 'public, max-age=300',
  });
  
  res.json(article);
});

// Client sends: If-None-Match: "abc123"
// Server responds: 304 Not Modified (no body)
// Saves bandwidth!
```

**Last-Modified:**
```typescript
app.get('/api/posts', (req, res) => {
  const posts = getPosts();
  const lastModified = getLastModifiedDate();
  
  // Check if client has current version
  const clientDate = req.headers['if-modified-since'];
  if (clientDate && new Date(clientDate) >= lastModified) {
    return res.status(304).end();
  }
  
  res.set({
    'Last-Modified': lastModified.toUTCString(),
    'Cache-Control': 'public, max-age=600',
  });
  
  res.json(posts);
});
```

#### 2. CDN Caching

**Cloudflare Configuration:**
```typescript
// Next.js API route with CDN caching
// pages/api/products/[id].ts
import { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const product = await fetchProduct(req.query.id as string);
  
  // Set cache headers for CDN
  res.setHeader(
    'Cache-Control',
    'public, s-maxage=3600, stale-while-revalidate=86400'
    // s-maxage=3600: CDN caches 1 hour
    // stale-while-revalidate=86400: Serve stale for 24h while revalidating
  );
  
  res.status(200).json(product);
}
```

**Vercel Edge Caching:**
```typescript
// Next.js page with ISR
// pages/products/[id].tsx
export async function getStaticProps({ params }) {
  const product = await fetchProduct(params.id);
  
  return {
    props: { product },
    revalidate: 300, // Revalidate every 5 minutes
  };
}

export async function getStaticPaths() {
  return {
    paths: [],
    fallback: 'blocking', // Generate on first request
  };
}

// Automatically cached at edge
// Fast global delivery
```

**Cloudflare Workers Cache API:**
```typescript
// Cloudflare Worker
export default {
  async fetch(request: Request): Promise<Response> {
    const cache = caches.default;
    const cacheKey = new Request(request.url, request);
    
    // Check cache
    let response = await cache.match(cacheKey);
    
    if (!response) {
      // Cache miss - fetch from origin
      response = await fetch(request);
      
      // Clone and cache response
      const clonedResponse = response.clone();
      
      // Cache for 1 hour
      const headers = new Headers(clonedResponse.headers);
      headers.set('Cache-Control', 'public, max-age=3600');
      
      const cachedResponse = new Response(clonedResponse.body, {
        status: clonedResponse.status,
        headers,
      });
      
      await cache.put(cacheKey, cachedResponse);
    }
    
    return response;
  },
};
```

#### 3. In-Memory Caching (Node.js)

**Simple In-Memory Cache:**
```typescript
class MemoryCache {
  private cache = new Map<string, { data: any; expires: number }>();
  
  set(key: string, data: any, ttlSeconds: number) {
    this.cache.set(key, {
      data,
      expires: Date.now() + ttlSeconds * 1000,
    });
  }
  
  get(key: string): any | null {
    const item = this.cache.get(key);
    
    if (!item) return null;
    
    if (Date.now() > item.expires) {
      this.cache.delete(key);
      return null;
    }
    
    return item.data;
  }
  
  delete(key: string) {
    this.cache.delete(key);
  }
  
  clear() {
    this.cache.clear();
  }
}

const cache = new MemoryCache();

// Usage in API route
app.get('/api/products', async (req, res) => {
  const cacheKey = 'products:all';
  
  // Check cache
  let products = cache.get(cacheKey);
  
  if (!products) {
    // Cache miss
    products = await fetchProductsFromDB();
    cache.set(cacheKey, products, 300); // Cache 5 minutes
  }
  
  res.json(products);
});
```

**LRU Cache (Eviction Strategy):**
```typescript
import LRU from 'lru-cache';

const cache = new LRU({
  max: 500,              // Max 500 items
  maxSize: 50000000,     // Max 50MB
  sizeCalculation: (value) => JSON.stringify(value).length,
  ttl: 1000 * 60 * 5,   // 5 minutes TTL
});

app.get('/api/user/:id', async (req, res) => {
  const cacheKey = `user:${req.params.id}`;
  
  let user = cache.get(cacheKey);
  
  if (!user) {
    user = await fetchUser(req.params.id);
    cache.set(cacheKey, user);
  }
  
  res.json(user);
});
```

#### 4. Redis Caching

**Basic Redis Cache:**
```typescript
import Redis from 'ioredis';

const redis = new Redis();

app.get('/api/posts', async (req, res) => {
  const cacheKey = 'posts:all';
  
  // Check Redis cache
  const cached = await redis.get(cacheKey);
  
  if (cached) {
    console.log('Cache hit');
    return res.json(JSON.parse(cached));
  }
  
  // Cache miss - fetch from DB
  console.log('Cache miss');
  const posts = await db.post.findMany();
  
  // Store in Redis for 5 minutes
  await redis.setex(cacheKey, 300, JSON.stringify(posts));
  
  res.json(posts);
});
```

**Cache Middleware:**
```typescript
import { Request, Response, NextFunction } from 'express';
import Redis from 'ioredis';

const redis = new Redis();

function cacheMiddleware(ttlSeconds: number) {
  return async (req: Request, res: Response, next: NextFunction) => {
    // Only cache GET requests
    if (req.method !== 'GET') {
      return next();
    }
    
    const cacheKey = `cache:${req.originalUrl}`;
    
    // Check cache
    const cached = await redis.get(cacheKey);
    
    if (cached) {
      return res.json(JSON.parse(cached));
    }
    
    // Override res.json to cache response
    const originalJson = res.json.bind(res);
    res.json = (data: any) => {
      redis.setex(cacheKey, ttlSeconds, JSON.stringify(data));
      return originalJson(data);
    };
    
    next();
  };
}

// Usage
app.get('/api/products', cacheMiddleware(300), async (req, res) => {
  const products = await fetchProducts();
  res.json(products);
});
```

**Cache Invalidation:**
```typescript
// Invalidate cache on update
app.post('/api/products', async (req, res) => {
  const product = await db.product.create({
    data: req.body,
  });
  
  // Invalidate related caches
  await redis.del('cache:/api/products');
  await redis.del(`cache:/api/products/${product.id}`);
  await redis.del('cache:/api/products/featured');
  
  res.json(product);
});

// Pattern-based invalidation
app.delete('/api/products/:id', async (req, res) => {
  await db.product.delete({ where: { id: req.params.id } });
  
  // Delete all product-related cache keys
  const keys = await redis.keys('cache:/api/products*');
  if (keys.length > 0) {
    await redis.del(...keys);
  }
  
  res.status(204).end();
});
```

#### 5. Next.js Incremental Static Regeneration

**ISR Implementation:**
```typescript
// pages/posts/[slug].tsx
export async function getStaticProps({ params }) {
  const post = await fetchPost(params.slug);
  
  return {
    props: { post },
    revalidate: 60, // Regenerate every 60 seconds
  };
}

export async function getStaticPaths() {
  // Pre-generate top 100 posts
  const topPosts = await fetchTopPosts(100);
  
  return {
    paths: topPosts.map(p => ({ params: { slug: p.slug } })),
    fallback: 'blocking', // Generate others on-demand
  };
}

// Result:
// - Top posts pre-rendered at build
// - Other posts rendered on first request
// - All posts revalidated every 60 seconds
// - Served from edge cache globally
```

**On-Demand Revalidation:**
```typescript
// pages/api/revalidate.ts
export default async function handler(req, res) {
  // Check secret
  if (req.query.secret !== process.env.REVALIDATE_SECRET) {
    return res.status(401).json({ message: 'Invalid token' });
  }
  
  try {
    // Revalidate specific path
    await res.revalidate(`/posts/${req.query.slug}`);
    return res.json({ revalidated: true });
  } catch (err) {
    return res.status(500).send('Error revalidating');
  }
}

// Call from CMS webhook:
// POST /api/revalidate?secret=TOKEN&slug=my-post
```

#### 6. Cache-Aside Pattern

**Lazy Loading Cache:**
```typescript
async function getUser(userId: string) {
  const cacheKey = `user:${userId}`;
  
  // 1. Try cache
  let user = await redis.get(cacheKey);
  
  if (user) {
    return JSON.parse(user);
  }
  
  // 2. Cache miss - query database
  user = await db.user.findUnique({ where: { id: userId } });
  
  if (!user) {
    throw new Error('User not found');
  }
  
  // 3. Store in cache
  await redis.setex(cacheKey, 3600, JSON.stringify(user));
  
  return user;
}
```

**Write-Through Cache:**
```typescript
async function updateUser(userId: string, data: any) {
  // 1. Update database
  const user = await db.user.update({
    where: { id: userId },
    data,
  });
  
  // 2. Update cache immediately
  const cacheKey = `user:${userId}`;
  await redis.setex(cacheKey, 3600, JSON.stringify(user));
  
  return user;
}
```

**Cache Warming:**
```typescript
// Pre-populate cache with frequently accessed data
async function warmCache() {
  console.log('Warming cache...');
  
  // Cache popular products
  const popularProducts = await db.product.findMany({
    where: { popular: true },
  });
  
  for (const product of popularProducts) {
    const cacheKey = `product:${product.id}`;
    await redis.setex(cacheKey, 3600, JSON.stringify(product));
  }
  
  console.log(`Cached ${popularProducts.length} products`);
}

// Run on server start
warmCache();
```

#### 7. Caching Strategies by Use Case

**Public Content (Aggressive Caching):**
```typescript
// Blog posts, marketing pages
res.setHeader(
  'Cache-Control',
  'public, max-age=3600, s-maxage=86400, stale-while-revalidate=604800'
);
// Browser: 1 hour
// CDN: 24 hours
// Serve stale: 7 days while revalidating
```

**User-Specific Content (Private Cache):**
```typescript
// User profile, dashboard
res.setHeader(
  'Cache-Control',
  'private, max-age=300, must-revalidate'
);
// Only browser caches (not CDN)
// 5 minutes
// Must revalidate when stale
```

**Frequently Updated (Short Cache):**
```typescript
// Live scores, stock prices
res.setHeader(
  'Cache-Control',
  'public, max-age=10, s-maxage=10'
);
// 10 seconds cache
// Balance between freshness and performance
```

**Sensitive Data (No Cache):**
```typescript
// Payment info, medical records
res.setHeader(
  'Cache-Control',
  'private, no-cache, no-store, must-revalidate'
);
// Never cache
// Always fetch fresh
```

#### 8. Cache Invalidation Strategies

**Time-Based (TTL):**
```typescript
// Expires after fixed duration
await redis.setex('key', 300, data); // 5 minutes
```

**Event-Based:**
```typescript
// Invalidate on data change
app.post('/api/products', async (req, res) => {
  const product = await createProduct(req.body);
  
  // Invalidate affected caches
  await invalidateProductCache(product.id);
  
  res.json(product);
});

async function invalidateProductCache(productId: string) {
  await redis.del(`product:${productId}`);
  await redis.del('products:all');
  await redis.del('products:featured');
}
```

**Cache Tags:**
```typescript
// Tag-based invalidation
const CACHE_TAGS = {
  PRODUCTS: 'products',
  USERS: 'users',
  POSTS: 'posts',
};

async function setCacheWithTags(
  key: string,
  data: any,
  ttl: number,
  tags: string[]
) {
  // Store data
  await redis.setex(key, ttl, JSON.stringify(data));
  
  // Track tags
  for (const tag of tags) {
    await redis.sadd(`tag:${tag}`, key);
  }
}

async function invalidateByTag(tag: string) {
  // Get all keys with this tag
  const keys = await redis.smembers(`tag:${tag}`);
  
  // Delete all
  if (keys.length > 0) {
    await redis.del(...keys);
  }
  
  // Clear tag set
  await redis.del(`tag:${tag}`);
}

// Usage
await setCacheWithTags(
  'product:123',
  product,
  3600,
  [CACHE_TAGS.PRODUCTS]
);

// Later: Invalidate all products
await invalidateByTag(CACHE_TAGS.PRODUCTS);
```

#### 9. Best Practices

**Checklist:**
```typescript
/*
✅ Use appropriate Cache-Control headers
✅ Implement ETag for conditional requests
✅ Cache at multiple levels (CDN, server, Redis)
✅ Set reasonable TTL values
✅ Invalidate cache on data changes
✅ Use cache keys wisely (include version)
✅ Monitor cache hit/miss ratios
✅ Handle cache failures gracefully
✅ Compress cached data
✅ Use Redis for distributed caching
✅ Warm cache for popular data
✅ Test cache behavior thoroughly
*/
```

**Cache Key Design:**
```typescript
// ✅ GOOD: Specific, versioned keys
const cacheKey = `v1:user:${userId}:profile`;
const cacheKey = `v2:products:${category}:page:${page}`;

// ❌ BAD: Ambiguous keys
const cacheKey = `data`;
const cacheKey = `${userId}`;
```

#### Summary

**Server-Side Caching Strategies:**

1. **HTTP Caching**
   - Cache-Control headers
   - ETag/Last-Modified
   - 304 Not Modified

2. **CDN Caching**
   - Edge caching
   - Global distribution
   - Stale-while-revalidate

3. **In-Memory Caching**
   - Fast access
   - LRU eviction
   - Limited to single server

4. **Redis Caching**
   - Distributed cache
   - Persistent
   - Pattern matching

5. **ISR (Next.js)**
   - Static generation
   - Automatic revalidation
   - Edge delivery

**Cache Levels:**
```
Browser Cache (max-age)
↓
CDN Cache (s-maxage)
↓
Server Cache (Redis)
↓
Database
```

**Key Principles:**
> "Cache aggressively, invalidate precisely."

> "The fastest request is the one you don't make!"

</details>

---

### 158. What is edge computing and how does it relate to React?

<details>
<summary>View Answer</summary>

**Edge computing** moves computation and data storage closer to end users by running code at CDN edge locations worldwide. For React apps, this means faster response times, reduced latency, and better performance through edge rendering, API routes, and middleware.

#### 1. What is Edge Computing?

**Concept:**
```typescript
/*
Traditional Architecture:
User (Tokyo) → Request → Server (US) → Response
- Latency: ~200ms
- Distance: ~10,000 km

Edge Architecture:
User (Tokyo) → Request → Edge (Tokyo) → Response
- Latency: ~10ms
- Distance: ~100 km

Benefits:
✅ Lower latency (20x faster)
✅ Better user experience
✅ Reduced origin server load
✅ Global distribution
✅ Auto-scaling
*/
```

**Edge Locations:**
```typescript
/*
Cloudflare: 300+ locations
Vercel Edge: 300+ locations
AWS CloudFront: 400+ locations
Fastly: 60+ locations

Coverage:
- North America
- Europe
- Asia Pacific
- South America
- Africa
- Middle East

Result: User always hits nearby edge
*/
```

#### 2. Edge Functions vs Serverless

**Comparison:**
```typescript
/*
Serverless Functions (AWS Lambda):
- Run in specific regions
- Cold start: 100-500ms
- Runtime: Node.js, Python, etc.
- Full Node.js APIs
- Database connections

Edge Functions (Cloudflare Workers, Vercel Edge):
- Run at 300+ global locations
- Cold start: <1ms
- Runtime: V8 isolates
- Limited APIs (Web Standards)
- No direct database access (use HTTP)

Use Serverless when:
✅ Need full Node.js features
✅ Database connections
✅ Heavy computation
✅ Long-running tasks

Use Edge when:
✅ Need low latency
✅ Lightweight logic
★ Global distribution
✅ High concurrency
*/
```

#### 3. Vercel Edge Functions

**Basic Edge Function:**
```typescript
// pages/api/hello.ts
import type { NextRequest } from 'next/server';

export const config = {
  runtime: 'edge', // Run at edge
};

export default async function handler(req: NextRequest) {
  // Get user location from request
  const country = req.geo?.country || 'Unknown';
  const city = req.geo?.city || 'Unknown';
  
  return new Response(
    JSON.stringify({
      message: 'Hello from the edge!',
      location: { country, city },
      timestamp: new Date().toISOString(),
    }),
    {
      headers: {
        'content-type': 'application/json',
      },
    }
  );
}

// Runs at edge nearest to user
// Response in <50ms globally
```

**Edge API with Geolocation:**
```typescript
// pages/api/geo.ts
import { NextRequest, NextResponse } from 'next/server';

export const config = { runtime: 'edge' };

export default async function handler(req: NextRequest) {
  const { geo, ip } = req;
  
  // Customize response based on location
  const content = {
    ip,
    country: geo?.country,
    city: geo?.city,
    region: geo?.region,
    latitude: geo?.latitude,
    longitude: geo?.longitude,
  };
  
  return NextResponse.json(content);
}

// Use for:
// - Geolocation-based content
// - Regional pricing
// - Language detection
// - Compliance (GDPR, etc.)
```

**Edge Middleware:**
```typescript
// middleware.ts
import { NextRequest, NextResponse } from 'next/server';

export function middleware(req: NextRequest) {
  const country = req.geo?.country;
  
  // Redirect based on country
  if (country === 'CN') {
    return NextResponse.redirect(new URL('/cn', req.url));
  }
  
  if (country === 'JP') {
    return NextResponse.redirect(new URL('/jp', req.url));
  }
  
  // Add custom header
  const response = NextResponse.next();
  response.headers.set('x-user-country', country || 'unknown');
  
  return response;
}

export const config = {
  matcher: '/:path*',
};

// Runs before page loads
// At edge, not origin
```

#### 4. Cloudflare Workers

**Worker Script:**
```typescript
// worker.ts
export default {
  async fetch(request: Request): Promise<Response> {
    const url = new URL(request.url);
    
    // Handle API request at edge
    if (url.pathname === '/api/data') {
      const data = {
        message: 'From Cloudflare Edge',
        location: request.cf?.city,
        timestamp: Date.now(),
      };
      
      return new Response(JSON.stringify(data), {
        headers: {
          'content-type': 'application/json',
          'cache-control': 'public, max-age=60',
        },
      });
    }
    
    // Proxy to origin for other requests
    return fetch(request);
  },
};
```

**Edge Caching:**
```typescript
export default {
  async fetch(request: Request): Promise<Response> {
    const cache = caches.default;
    
    // Check cache
    let response = await cache.match(request);
    
    if (!response) {
      // Cache miss - fetch from origin
      response = await fetch(request);
      
      // Clone and cache
      const clonedResponse = response.clone();
      
      // Only cache successful responses
      if (clonedResponse.status === 200) {
        await cache.put(request, clonedResponse);
      }
    }
    
    return response;
  },
};
```

**A/B Testing at Edge:**
```typescript
export default {
  async fetch(request: Request): Promise<Response> {
    const url = new URL(request.url);
    
    // Get or set A/B test variant
    const cookie = request.headers.get('cookie') || '';
    let variant = cookie.match(/variant=([AB])/)?.[1];
    
    if (!variant) {
      // Assign random variant
      variant = Math.random() < 0.5 ? 'A' : 'B';
    }
    
    // Fetch appropriate version
    url.searchParams.set('variant', variant);
    const response = await fetch(url);
    
    // Set cookie
    const newResponse = new Response(response.body, response);
    newResponse.headers.set(
      'Set-Cookie',
      `variant=${variant}; Max-Age=86400; Path=/`
    );
    
    return newResponse;
  },
};
```

#### 5. Edge Rendering (React Server Components)

**Next.js App Router with Edge:**
```typescript
// app/api/products/route.ts
import { NextRequest, NextResponse } from 'next/server';

export const runtime = 'edge';

export async function GET(request: NextRequest) {
  // Fetch data at edge
  const products = await fetch('https://api.example.com/products', {
    next: { revalidate: 60 }, // Cache 60 seconds
  }).then(r => r.json());
  
  return NextResponse.json(products);
}
```

**Server Component at Edge:**
```typescript
// app/products/page.tsx
export const runtime = 'edge';

async function getProducts() {
  const res = await fetch('https://api.example.com/products');
  return res.json();
}

export default async function ProductsPage() {
  const products = await getProducts();
  
  return (
    <div>
      <h1>Products</h1>
      {products.map((product: any) => (
        <div key={product.id}>
          <h2>{product.name}</h2>
          <p>${product.price}</p>
        </div>
      ))}
    </div>
  );
}

// Rendered at edge
// Fast response globally
```

#### 6. Edge Use Cases for React

**1. Authentication/Authorization:**
```typescript
// middleware.ts
import { NextRequest, NextResponse } from 'next/server';
import { verifyToken } from './lib/auth';

export async function middleware(req: NextRequest) {
  const token = req.cookies.get('token')?.value;
  
  // Verify token at edge
  if (!token || !(await verifyToken(token))) {
    return NextResponse.redirect(new URL('/login', req.url));
  }
  
  return NextResponse.next();
}

export const config = {
  matcher: ['/dashboard/:path*', '/profile/:path*'],
};

// Protects routes at edge
// No origin server hit
```

**2. Localization:**
```typescript
// middleware.ts
import { NextRequest, NextResponse } from 'next/server';

export function middleware(req: NextRequest) {
  const { pathname } = req.nextUrl;
  
  // Detect locale from header or cookie
  const locale = 
    req.cookies.get('locale')?.value ||
    req.headers.get('accept-language')?.split(',')[0] ||
    'en';
  
  // Redirect to localized path
  if (!pathname.startsWith(`/${locale}`)) {
    return NextResponse.redirect(
      new URL(`/${locale}${pathname}`, req.url)
    );
  }
  
  return NextResponse.next();
}
```

**3. Rate Limiting:**
```typescript
// Cloudflare Worker with rate limiting
const RATE_LIMIT = 100; // requests per minute

export default {
  async fetch(request: Request, env: any): Promise<Response> {
    const ip = request.headers.get('CF-Connecting-IP');
    const key = `ratelimit:${ip}`;
    
    // Check rate limit in KV store
    const count = await env.KV.get(key);
    
    if (count && parseInt(count) > RATE_LIMIT) {
      return new Response('Rate limit exceeded', { status: 429 });
    }
    
    // Increment counter
    await env.KV.put(key, String((count ? parseInt(count) : 0) + 1), {
      expirationTtl: 60, // 1 minute
    });
    
    return fetch(request);
  },
};
```

**4. Image Optimization:**
```typescript
// Edge function to optimize images
export const config = { runtime: 'edge' };

export default async function handler(req: NextRequest) {
  const { searchParams } = new URL(req.url);
  const imageUrl = searchParams.get('url');
  const width = parseInt(searchParams.get('w') || '800');
  const quality = parseInt(searchParams.get('q') || '75');
  
  // Fetch original image
  const response = await fetch(imageUrl!);
  
  // Transform at edge (if using Cloudflare Images)
  // Or return with cache headers
  return new Response(response.body, {
    headers: {
      'Content-Type': 'image/webp',
      'Cache-Control': 'public, max-age=31536000, immutable',
    },
  });
}
```

**5. Personalization:**
```typescript
// middleware.ts
export function middleware(req: NextRequest) {
  const country = req.geo?.country;
  const response = NextResponse.next();
  
  // Set personalization cookie
  if (country === 'US') {
    response.cookies.set('currency', 'USD');
    response.cookies.set('language', 'en-US');
  } else if (country === 'GB') {
    response.cookies.set('currency', 'GBP');
    response.cookies.set('language', 'en-GB');
  } else if (country === 'JP') {
    response.cookies.set('currency', 'JPY');
    response.cookies.set('language', 'ja');
  }
  
  return response;
}
```

#### 7. Edge Limitations

**What You Can't Do:**
```typescript
/*
❌ No Node.js built-in modules (fs, crypto, etc.)
❌ No native dependencies
❌ No direct database connections
❌ Limited execution time (typically 50-300ms)
❌ Limited memory (typically 128MB)
❌ No WebSockets (some platforms)
❌ No persistent storage

Workarounds:
✅ Use Web Standard APIs
✅ Use HTTP to access databases (Prisma Data Proxy)
✅ Use edge-compatible packages
✅ Keep logic lightweight
✅ Use KV stores for persistence
*/
```

**Edge-Compatible vs Not:**
```typescript
// ❌ Not edge-compatible
import fs from 'fs';
import crypto from 'crypto';
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();
const data = fs.readFileSync('./data.txt');

// ✅ Edge-compatible
import { createHash } from '@cloudflare/workers-crypto';
import { PrismaClient } from '@prisma/client/edge';

const prisma = new PrismaClient({
  datasources: {
    db: { url: process.env.DATABASE_URL },
  },
});

const hash = await createHash('SHA-256', data);
```

#### 8. Performance Benefits

**Latency Comparison:**
```typescript
/*
Scenario: User in Sydney accessing US-hosted app

Traditional (Sydney → US West):
- Network latency: ~180ms
- Server processing: ~50ms
- Total: ~230ms

Edge (Sydney → Sydney Edge):
- Network latency: ~10ms
- Edge processing: ~5ms
- Total: ~15ms

Improvement: 15x faster! (230ms → 15ms)

Cost:
- Reduced origin server load
- Lower bandwidth costs
- Better scalability
*/
```

#### 9. Best Practices

**Checklist:**
```typescript
/*
✅ Use edge for lightweight, latency-sensitive logic
✅ Keep edge functions small (<1MB)
✅ Cache aggressively at edge
✅ Use origin for heavy computation
✅ Handle edge function errors gracefully
✅ Test in multiple regions
✅ Monitor edge function performance
✅ Use Web Standard APIs
✅ Minimize external dependencies
✅ Leverage geolocation for personalization
*/
```

**When to Use Edge:**
```typescript
/*
✅ Authentication/authorization
✅ A/B testing
✅ Redirects and rewrites
✅ Rate limiting
✅ Header manipulation
✅ Geolocation-based routing
✅ API aggregation
✅ Lightweight data transformation
✅ Image optimization
✅ Static asset serving

❌ Avoid for:
- Heavy computation
- Long-running tasks
- Direct database queries (use HTTP instead)
- File system operations
- Complex business logic
*/
```

#### 10. Providers Comparison

**Feature Matrix:**
```typescript
/*
Cloudflare Workers:
- Network: 300+ locations
- Cold start: <1ms
- Free tier: 100K requests/day
- KV storage: Yes
- Durable Objects: Yes
- Best for: Global apps, APIs

Vercel Edge Functions:
- Network: 300+ locations
- Cold start: <1ms
- Free tier: 100K requests/month
- Integration: Next.js native
- Best for: Next.js apps

AWS Lambda@Edge:
- Network: 400+ CloudFront locations
- Cold start: ~50ms
- Best for: AWS ecosystem

Fastly Compute@Edge:
- Network: 60+ locations
- Runtime: WebAssembly
- Best for: High performance
*/
```

#### Summary

**Edge Computing for React:**

**Benefits:**
- ✅ Lower latency (10-20x faster)
- ✅ Global distribution
- ✅ Better performance
- ✅ Reduced origin load
- ✅ Auto-scaling

**Use Cases:**
1. Authentication/authorization
2. Geolocation-based routing
3. A/B testing
4. Rate limiting
5. API aggregation
6. Image optimization
7. Personalization

**Popular Platforms:**
- Cloudflare Workers
- Vercel Edge Functions
- AWS Lambda@Edge
- Fastly Compute@Edge

**Implementation:**
- Edge Functions (API routes)
- Edge Middleware (request handling)
- Edge Rendering (RSC)
- Edge Caching

**Key Principle:**
> "Move compute closer to users. Process at the edge, cache at the edge, serve from the edge."

**Impact:**
> "Edge computing can reduce latency from 200ms to 10ms, making apps feel instant globally!"

</details>

---

### 159. How do you optimize images in React applications?

<details>
<summary>View Answer</summary>

**Image optimization** is crucial for performance since images typically account for 50-70% of page weight. Proper optimization reduces load times, bandwidth usage, and improves user experience, especially on slow networks.

#### 1. Next.js Image Component

**Basic Usage:**
```typescript
import Image from 'next/image';

function ProductCard({ product }: { product: Product }) {
  return (
    <div>
      <Image
        src={product.image}
        alt={product.name}
        width={400}
        height={300}
        priority={false}  // Load lazily
        quality={75}      // Compress to 75%
      />
      <h3>{product.name}</h3>
    </div>
  );
}

// Automatic benefits:
// ✅ Automatic WebP/AVIF conversion
// ✅ Lazy loading
// ✅ Responsive images
// ✅ Blur placeholder
// ✅ Prevents layout shift
```

**Priority Loading:**
```typescript
// Hero image - load immediately
<Image
  src="/hero.jpg"
  alt="Hero"
  width={1920}
  height={1080}
  priority  // Don't lazy load
  quality={90}  // Higher quality
/>

// Below-the-fold images - lazy load
<Image
  src="/product.jpg"
  alt="Product"
  width={400}
  height={300}
  loading="lazy"  // Default
/>
```

**Blur Placeholder:**
```typescript
import Image from 'next/image';

function Card({ image }: { image: string }) {
  return (
    <Image
      src={image}
      alt="Product"
      width={400}
      height={300}
      placeholder="blur"
      blurDataURL="data:image/jpeg;base64,/9j/4AAQSkZJRg..." // Base64 thumbnail
    />
  );
}

// Or generate automatically (Next.js 13+)
// next.config.js
module.exports = {
  images: {
    formats: ['image/avif', 'image/webp'],
  },
};
```

**Responsive Images:**
```typescript
<Image
  src="/photo.jpg"
  alt="Photo"
  width={800}
  height={600}
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
/>

// Generates srcset:
// <img
//   srcset="
//     /_next/image?url=/photo.jpg&w=640 640w,
//     /_next/image?url=/photo.jpg&w=750 750w,
//     /_next/image?url=/photo.jpg&w=828 828w,
//     ...
//   "
//   sizes="(max-width: 768px) 100vw, ..."
// />
```

#### 2. Native Lazy Loading

**HTML Lazy Loading:**
```typescript
function ProductImage({ src, alt }: { src: string; alt: string }) {
  return (
    <img
      src={src}
      alt={alt}
      loading="lazy"  // Browser native lazy loading
      width="400"
      height="300"
    />
  );
}

// Supported by 95% of browsers
// No JavaScript required
```

**Intersection Observer (Custom):**
```typescript
import { useEffect, useRef, useState } from 'react';

function LazyImage({ src, alt }: { src: string; alt: string }) {
  const [isVisible, setIsVisible] = useState(false);
  const imgRef = useRef<HTMLImageElement>(null);
  
  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setIsVisible(true);
          observer.disconnect();
        }
      },
      { threshold: 0.1, rootMargin: '50px' }
    );
    
    if (imgRef.current) {
      observer.observe(imgRef.current);
    }
    
    return () => observer.disconnect();
  }, []);
  
  return (
    <img
      ref={imgRef}
      src={isVisible ? src : ''}
      data-src={src}
      alt={alt}
      className={isVisible ? 'loaded' : 'loading'}
    />
  );
}
```

#### 3. Modern Image Formats

**WebP and AVIF:**
```typescript
// HTML picture element
function OptimizedImage() {
  return (
    <picture>
      {/* AVIF: Best compression (50% smaller than JPEG) */}
      <source srcSet="/photo.avif" type="image/avif" />
      
      {/* WebP: Good compression (30% smaller than JPEG) */}
      <source srcSet="/photo.webp" type="image/webp" />
      
      {/* JPEG: Fallback */}
      <img src="/photo.jpg" alt="Photo" />
    </picture>
  );
}

// Browser automatically picks best supported format
```

**Component with Format Detection:**
```typescript
function SmartImage({ src, alt }: { src: string; alt: string }) {
  const basePath = src.replace(/\.[^.]+$/, '');
  const ext = src.match(/\.([^.]+)$/)?.[1] || 'jpg';
  
  return (
    <picture>
      <source srcSet={`${basePath}.avif`} type="image/avif" />
      <source srcSet={`${basePath}.webp`} type="image/webp" />
      <img src={src} alt={alt} loading="lazy" />
    </picture>
  );
}
```

#### 4. Responsive Images

**srcset and sizes:**
```typescript
function ResponsiveImage() {
  return (
    <img
      src="/photo-800.jpg"
      srcSet="
        /photo-400.jpg 400w,
        /photo-800.jpg 800w,
        /photo-1200.jpg 1200w,
        /photo-1600.jpg 1600w
      "
      sizes="
        (max-width: 640px) 100vw,
        (max-width: 1024px) 50vw,
        33vw
      "
      alt="Responsive"
      loading="lazy"
    />
  );
}

// Browser picks appropriate size based on:
// - Screen size
// - Device pixel ratio
// - Network conditions
```

**Art Direction:**
```typescript
function ArtDirectedImage() {
  return (
    <picture>
      {/* Mobile: Square crop */}
      <source
        media="(max-width: 640px)"
        srcSet="/photo-square-400.jpg 400w, /photo-square-800.jpg 800w"
      />
      
      {/* Tablet: 16:9 crop */}
      <source
        media="(max-width: 1024px)"
        srcSet="/photo-wide-800.jpg 800w, /photo-wide-1200.jpg 1200w"
      />
      
      {/* Desktop: Original */}
      <img
        src="/photo-1600.jpg"
        srcSet="/photo-1600.jpg 1600w, /photo-2400.jpg 2400w"
        alt="Art directed"
      />
    </picture>
  );
}
```

#### 5. Image Compression

**Sharp (Node.js):**
```typescript
import sharp from 'sharp';
import fs from 'fs';

async function optimizeImage(inputPath: string, outputPath: string) {
  await sharp(inputPath)
    .resize(800, 600, {
      fit: 'cover',
      position: 'center',
    })
    .webp({ quality: 80 })
    .toFile(outputPath);
}

// Reduce 2MB JPEG to 150KB WebP
```

**Image Optimization Script:**
```typescript
// scripts/optimize-images.ts
import sharp from 'sharp';
import { glob } from 'glob';
import path from 'path';

const SIZES = [400, 800, 1200, 1600];
const QUALITY = 80;

async function optimizeImages() {
  const images = await glob('public/images/**/*.{jpg,jpeg,png}');
  
  for (const image of images) {
    const basename = path.basename(image, path.extname(image));
    const dirname = path.dirname(image);
    
    for (const size of SIZES) {
      // WebP
      await sharp(image)
        .resize(size, null, { withoutEnlargement: true })
        .webp({ quality: QUALITY })
        .toFile(`${dirname}/${basename}-${size}.webp`);
      
      // AVIF
      await sharp(image)
        .resize(size, null, { withoutEnlargement: true })
        .avif({ quality: QUALITY })
        .toFile(`${dirname}/${basename}-${size}.avif`);
    }
    
    console.log(`Optimized: ${image}`);
  }
}

optimizeImages();
```

#### 6. CDN and Image Services

**Cloudinary:**
```typescript
function CloudinaryImage({ publicId }: { publicId: string }) {
  const cloudinaryUrl = `https://res.cloudinary.com/demo/image/upload/`;
  
  return (
    <img
      src={`${cloudinaryUrl}w_800,q_auto,f_auto/${publicId}`}
      srcSet={`
        ${cloudinaryUrl}w_400,q_auto,f_auto/${publicId} 400w,
        ${cloudinaryUrl}w_800,q_auto,f_auto/${publicId} 800w,
        ${cloudinaryUrl}w_1200,q_auto,f_auto/${publicId} 1200w
      `}
      sizes="(max-width: 768px) 100vw, 50vw"
      alt="Cloudinary"
      loading="lazy"
    />
  );
}

// Benefits:
// - Automatic format selection
// - Quality optimization
// - Responsive images
// - CDN delivery
```

**Imgix:**
```typescript
import Imgix from 'react-imgix';

function ImgixImage({ src }: { src: string }) {
  return (
    <Imgix
      src={src}
      sizes="(max-width: 768px) 100vw, 50vw"
      htmlAttributes={{
        alt: 'Optimized image',
        loading: 'lazy',
      }}
      imgixParams={{
        auto: 'format,compress',
        fit: 'crop',
        w: 800,
        h: 600,
      }}
    />
  );
}
```

#### 7. Progressive Loading

**Blur-Up Technique:**
```typescript
import { useState } from 'react';

function ProgressiveImage({
  src,
  placeholder,
  alt,
}: {
  src: string;
  placeholder: string;
  alt: string;
}) {
  const [loaded, setLoaded] = useState(false);
  
  return (
    <div className="progressive-image">
      <img
        src={placeholder}
        alt={alt}
        className="placeholder"
        style={{
          filter: loaded ? 'blur(0)' : 'blur(20px)',
          transition: 'filter 0.3s',
        }}
      />
      <img
        src={src}
        alt={alt}
        onLoad={() => setLoaded(true)}
        style={{
          opacity: loaded ? 1 : 0,
          transition: 'opacity 0.3s',
        }}
      />
    </div>
  );
}

// Show blurred thumbnail while loading full image
```

**Low Quality Image Placeholder (LQIP):**
```typescript
// Generate with Sharp
async function generateLQIP(imagePath: string): Promise<string> {
  const buffer = await sharp(imagePath)
    .resize(20) // Tiny 20px wide
    .blur()
    .toBuffer();
  
  return `data:image/jpeg;base64,${buffer.toString('base64')}`;
}

// Use in component
function ImageWithLQIP() {
  return (
    <Image
      src="/photo.jpg"
      alt="Photo"
      width={800}
      height={600}
      placeholder="blur"
      blurDataURL="data:image/jpeg;base64,/9j/4AAQ..."
    />
  );
}
```

#### 8. Background Images

**Lazy Load Background:**
```typescript
import { useEffect, useRef, useState } from 'react';

function LazyBackground({ imageUrl }: { imageUrl: string }) {
  const [loaded, setLoaded] = useState(false);
  const divRef = useRef<HTMLDivElement>(null);
  
  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setLoaded(true);
          observer.disconnect();
        }
      }
    );
    
    if (divRef.current) {
      observer.observe(divRef.current);
    }
    
    return () => observer.disconnect();
  }, []);
  
  return (
    <div
      ref={divRef}
      style={{
        backgroundImage: loaded ? `url(${imageUrl})` : 'none',
        backgroundColor: '#f0f0f0',
      }}
    />
  );
}
```

#### 9. SVG Optimization

**Inline SVG:**
```typescript
function Logo() {
  return (
    <svg width="100" height="100" viewBox="0 0 100 100">
      <circle cx="50" cy="50" r="40" fill="#007bff" />
    </svg>
  );
}

// Benefits:
// - No HTTP request
// - Styleable with CSS
// - Accessible
// - Small size
```

**SVGO Optimization:**
```bash
# Install
npm install -g svgo

# Optimize
svgo input.svg -o output.svg

# Result: 50-80% smaller
```

#### 10. Performance Monitoring

**Measure Image Load Time:**
```typescript
import { useEffect } from 'react';

function MonitoredImage({ src, alt }: { src: string; alt: string }) {
  useEffect(() => {
    const img = new window.Image();
    const startTime = performance.now();
    
    img.onload = () => {
      const loadTime = performance.now() - startTime;
      console.log(`Image loaded in ${loadTime}ms`);
      
      // Send to analytics
      analytics.track('image_load', {
        src,
        loadTime,
        size: img.naturalWidth * img.naturalHeight,
      });
    };
    
    img.src = src;
  }, [src]);
  
  return <img src={src} alt={alt} loading="lazy" />;
}
```

**Lighthouse Performance:**
```typescript
/*
Image Optimization Metrics:

✅ Properly sized images
✅ Efficient image formats (WebP, AVIF)
✅ Image dimensions specified
✅ Lazy loading offscreen images
✅ Avoid layout shift (width/height)
✅ Compress images
✅ Use CDN

Goals:
- Largest Contentful Paint (LCP): < 2.5s
- Cumulative Layout Shift (CLS): < 0.1
- Total image weight: < 1MB
*/
```

#### 11. Best Practices

**Checklist:**
```typescript
/*
✅ Use Next.js Image component (or similar)
✅ Convert to WebP/AVIF
✅ Compress images (80% quality)
✅ Use responsive images (srcset/sizes)
✅ Lazy load offscreen images
✅ Set explicit width/height
✅ Use blur placeholders
✅ Serve from CDN
✅ Use appropriate sizes
✅ Optimize SVGs
✅ Monitor with Lighthouse
✅ Test on slow networks
*/
```

**Image Size Guidelines:**
```typescript
/*
Recommended Sizes:

Thumbnails: 150x150 (10-20 KB)
Cards: 400x300 (30-50 KB)
Hero: 1920x1080 (100-200 KB)
Full width: 1600x900 (80-150 KB)

Format by Use Case:

Photos: WebP/AVIF (lossy)
Graphics: WebP/AVIF (lossless) or PNG
Icons: SVG
Animations: WebP (animated) or MP4

Quality Settings:

Hero images: 85-90%
Content images: 75-85%
Thumbnails: 70-80%
*/
```

#### Summary

**Image Optimization Techniques:**

1. **Next.js Image Component**
   - Automatic optimization
   - Lazy loading
   - Responsive images
   - Blur placeholders

2. **Modern Formats**
   - WebP (30% smaller)
   - AVIF (50% smaller)
   - Fallback to JPEG/PNG

3. **Lazy Loading**
   - Native `loading="lazy"`
   - Intersection Observer
   - Load offscreen images

4. **Responsive Images**
   - srcset/sizes
   - Art direction
   - Device-specific sizes

5. **Compression**
   - Sharp, Squoosh
   - 75-85% quality
   - Automated pipeline

6. **CDN & Services**
   - Cloudinary, Imgix
   - Automatic optimization
   - Global delivery

**Impact:**
> "Optimizing images can reduce page weight by 50-70%, cutting load times from 5s to <2s!"

**Key Rule:**
> "Right format, right size, lazy loaded, from CDN."

</details>

---

### 160. What is the impact of third-party libraries on performance?

<details>
<summary>View Answer</summary>

**Third-party libraries** can significantly impact performance by increasing bundle size, adding execution overhead, and introducing dependencies. A single poorly chosen library can add hundreds of kilobytes to your bundle and slow down your app.

#### 1. Bundle Size Impact

**The Problem:**
```typescript
/*
Common Library Sizes (uncompressed):

moment.js: 288 KB
lodash: 70 KB
react-icons (all): 900 KB
chartjs: 240 KB
jquery: 90 KB
axios: 13 KB

Example App:
React app: 50 KB
+ moment: 288 KB
+ lodash: 70 KB
+ chartjs: 240 KB
= 648 KB total

Result: 13x larger than necessary!

Optimized Alternative:
React app: 50 KB
+ date-fns: 13 KB
+ native JS: 0 KB
+ recharts: 93 KB
= 156 KB total (76% reduction)
*/
```

**Real-World Example:**
```typescript
// ❌ BAD: Importing entire library
import _ from 'lodash';  // 70 KB
import moment from 'moment';  // 288 KB
import * as Icons from 'react-icons/fa';  // 900 KB

function App() {
  const date = moment().format('YYYY-MM-DD');
  const unique = _.uniq([1, 2, 2, 3]);
  
  return (
    <div>
      <Icons.FaHome />
      <p>{date}</p>
    </div>
  );
}

// Bundle: ~1.3 MB
// Load time on 3G: ~10 seconds

// ✅ GOOD: Optimized alternatives
import { format } from 'date-fns';  // 13 KB (tree-shakeable)
import { FaHome } from 'react-icons/fa';  // 2 KB (single icon)

function App() {
  const date = format(new Date(), 'yyyy-MM-dd');
  const unique = [...new Set([1, 2, 2, 3])];  // Native
  
  return (
    <div>
      <FaHome />
      <p>{date}</p>
    </div>
  );
}

// Bundle: ~65 KB
// Load time on 3G: ~0.5 seconds
// 95% reduction!
```

#### 2. Analyze Library Impact

**Bundlephobia:**
```bash
# Check before installing
npx bundle-phobia moment

# Output:
# moment
# Size: 288.45 KB (gzipped: 71.56 KB)
# Download time (slow 3G): 1.4s
# ⚠️ Warning: Large bundle size

# Compare alternatives
npx bundle-phobia date-fns
# date-fns
# Size: 78.14 KB (gzipped: 13.28 KB)
# Download time (slow 3G): 0.3s
# ✅ Much smaller!
```

**Webpack Bundle Analyzer:**
```bash
npm install --save-dev webpack-bundle-analyzer

# Run
ANALYZE=true npm run build

# Visual breakdown:
# ┌──────────────────────┐
# │ moment.js: 288 KB    │
# │ (40% of bundle!)     │
# ├──────────────────────┤
# │ Your code: 200 KB    │
# └──────────────────────┘
```

**Import Cost (VS Code Extension):**
```typescript
import moment from 'moment';  // ⚠️ 288 KB
import _ from 'lodash';  // ⚠️ 70 KB
import { format } from 'date-fns';  // ✅ 13 KB

// Shows size inline as you code
```

#### 3. Tree Shaking Issues

**Non-Tree-Shakeable Libraries:**
```typescript
// ❌ lodash (CommonJS - not tree-shakeable)
import { uniq } from 'lodash';
// Entire lodash (70 KB) included!

// ✅ lodash-es (ES modules - tree-shakeable)
import { uniq } from 'lodash-es';
// Only uniq function (~5 KB) included

// ✅ Best: Individual imports
import uniq from 'lodash/uniq';
// Only uniq function included
```

**Barrel Exports Problem:**
```typescript
// components/index.ts (barrel export)
export { Button } from './Button';
export { Input } from './Input';
export { Modal } from './Modal';
// ... 50 more components

// app.tsx
import { Button } from './components';
// ⚠️ May import ALL components (depends on bundler)

// ✅ Better: Direct import
import { Button } from './components/Button';
// Only imports Button
```

#### 4. Lighter Alternatives

**Date Libraries:**
```typescript
// moment: 288 KB
import moment from 'moment';
moment().format('YYYY-MM-DD');

// date-fns: 13 KB (tree-shakeable)
import { format } from 'date-fns';
format(new Date(), 'yyyy-MM-dd');

// dayjs: 7 KB
import dayjs from 'dayjs';
dayjs().format('YYYY-MM-DD');

// Intl API: 0 KB (native)
new Intl.DateTimeFormat('en-US').format(new Date());
```

**Utility Libraries:**
```typescript
// lodash: 70 KB
import _ from 'lodash';
const result = _.uniq(array);

// Native JS: 0 KB
const result = [...new Set(array)];

// lodash-es: 5 KB (tree-shakeable)
import { uniq } from 'lodash-es';
const result = uniq(array);
```

**HTTP Clients:**
```typescript
// axios: 13 KB
import axios from 'axios';
const response = await axios.get('/api/data');

// fetch: 0 KB (native)
const response = await fetch('/api/data');
const data = await response.json();

// ky: 5 KB (fetch wrapper)
import ky from 'ky';
const data = await ky.get('/api/data').json();
```

**Chart Libraries:**
```typescript
// chartjs: 240 KB
import { Chart } from 'chart.js';

// recharts: 93 KB
import { LineChart, Line } from 'recharts';

// victory: 140 KB
import { VictoryChart, VictoryLine } from 'victory';

// Custom SVG: 5-10 KB
function SimpleChart({ data }) {
  return (
    <svg viewBox="0 0 400 200">
      {/* Custom chart implementation */}
    </svg>
  );
}
```

#### 5. Lazy Loading Libraries

**Dynamic Imports:**
```typescript
import { lazy, Suspense } from 'react';

// ❌ BAD: Load heavy library upfront
import { Editor } from 'react-draft-wysiwyg';

function App() {
  return <Editor />;
}

// ✅ GOOD: Load on demand
const Editor = lazy(() => 
  import('react-draft-wysiwyg').then(mod => ({ default: mod.Editor }))
);

function App() {
  const [showEditor, setShowEditor] = useState(false);
  
  return (
    <div>
      <button onClick={() => setShowEditor(true)}>Edit</button>
      
      {showEditor && (
        <Suspense fallback={<div>Loading editor...</div>}>
          <Editor />
        </Suspense>
      )}
    </div>
  );
}

// Editor only loads when needed
// Saves 200+ KB on initial load
```

**Load on Interaction:**
```typescript
import { useState } from 'react';

function ChartComponent({ data }) {
  const [Chart, setChart] = useState<any>(null);
  
  const loadChart = async () => {
    const { Chart: ChartJS } = await import('chart.js');
    setChart(() => ChartJS);
  };
  
  return (
    <div>
      {!Chart ? (
        <button onClick={loadChart}>Load Chart</button>
      ) : (
        <Chart data={data} />
      )}
    </div>
  );
}
```

#### 6. Performance Overhead

**Execution Time:**
```typescript
/*
Library Overhead:

moment.js:
- Parse time: 2-5ms per operation
- Memory: ~10 MB

date-fns:
- Parse time: 0.5-1ms per operation
- Memory: ~2 MB

Native Date:
- Parse time: 0.1ms per operation
- Memory: Minimal

Impact on 1000 operations:
- moment: 5 seconds
- date-fns: 1 second
- Native: 0.1 second
*/
```

**Benchmark Example:**
```typescript
function benchmarkDateLibraries() {
  const iterations = 10000;
  
  // moment.js
  console.time('moment');
  for (let i = 0; i < iterations; i++) {
    moment().format('YYYY-MM-DD');
  }
  console.timeEnd('moment');  // 850ms
  
  // date-fns
  console.time('date-fns');
  for (let i = 0; i < iterations; i++) {
    format(new Date(), 'yyyy-MM-dd');
  }
  console.timeEnd('date-fns');  // 320ms
  
  // Native
  console.time('native');
  for (let i = 0; i < iterations; i++) {
    new Date().toISOString().split('T')[0];
  }
  console.timeEnd('native');  // 45ms
}
```

#### 7. Dependency Hell

**Transitive Dependencies:**
```bash
# Install one library
npm install some-library

# Actually installs:
npm ls some-library
# some-library@1.0.0
# ├── dependency-a@2.0.0
# │   ├── dependency-b@3.0.0
# │   └── dependency-c@1.5.0
# └── dependency-d@4.0.0
#     └── dependency-e@2.1.0

# 6 packages instead of 1!
# 500 KB instead of 100 KB
```

**Check Dependencies:**
```bash
# See all dependencies
npm ls

# Find duplicate dependencies
npm dedupe

# Check for updates
npm outdated
```

#### 8. Monitoring in Production

**Track Library Performance:**
```typescript
import { useEffect } from 'react';

function trackLibraryLoad(libraryName: string) {
  useEffect(() => {
    const startTime = performance.now();
    
    import(libraryName).then(() => {
      const loadTime = performance.now() - startTime;
      
      analytics.track('library_load', {
        library: libraryName,
        loadTime,
      });
      
      if (loadTime > 1000) {
        console.warn(`${libraryName} took ${loadTime}ms to load`);
      }
    });
  }, []);
}
```

**Bundle Size Budget:**
```javascript
// webpack.config.js
module.exports = {
  performance: {
    maxEntrypointSize: 250000,  // 250 KB
    maxAssetSize: 250000,
    hints: 'error',  // Fail build if exceeded
  },
};
```

#### 9. Best Practices

**Evaluation Checklist:**
```typescript
/*
Before adding a library, check:

✅ Bundle size (use bundlephobia.com)
✅ Tree-shakeable? (ES modules)
✅ Dependencies count
✅ Last updated (active maintenance?)
✅ Weekly downloads (popular?)
✅ License (compatible?)
✅ Native alternatives exist?
✅ Can I implement it myself?
✅ TypeScript support?
✅ Performance benchmarks?

Red flags:
❌ > 100 KB size
❌ CommonJS only
❌ > 10 dependencies
❌ Not updated in 2+ years
❌ < 1000 weekly downloads
❌ No tests
*/
```

**Decision Matrix:**
```typescript
/*
Library Size → Decision:

< 10 KB: Usually OK
10-50 KB: Consider carefully
50-100 KB: Need strong justification
> 100 KB: Avoid or lazy load
> 500 KB: Find alternative

Examples:

✅ date-fns (13 KB): OK
⚠️ axios (13 KB): Consider native fetch
❌ moment (288 KB): Use alternative
❌ lodash (70 KB): Use native JS
✅ recharts (93 KB): OK if lazy loaded
*/
```

#### 10. Migration Strategies

**Gradual Replacement:**
```typescript
// Step 1: Add new library
import { format } from 'date-fns';
import moment from 'moment';

// Step 2: Use both (during migration)
const newCode = format(date, 'yyyy-MM-dd');
const oldCode = moment(date).format('YYYY-MM-DD');

// Step 3: Replace all usages
// Find: moment(.*).format
// Replace with date-fns equivalent

// Step 4: Remove old library
npm uninstall moment

// Result: 275 KB saved!
```

**Automated Migration:**
```bash
# Use codemod for automated refactoring
npx jscodeshift -t moment-to-date-fns.js src/

# Or use babel plugin
# .babelrc
{
  "plugins": [
    ["babel-plugin-transform-imports", {
      "lodash": {
        "transform": "lodash/${member}",
        "preventFullImport": true
      }
    }]
  ]
}
```

#### 11. Summary

**Impact of Third-Party Libraries:**

**Negative:**
- ❌ Increased bundle size
- ❌ Longer load times
- ❌ Execution overhead
- ❌ Dependency bloat
- ❌ Security vulnerabilities
- ❌ Maintenance burden

**Mitigation:**
1. **Evaluate before installing**
   - Check size with bundlephobia
   - Consider alternatives
   - Check if native solution exists

2. **Optimize usage**
   - Use tree-shakeable versions
   - Import only what you need
   - Lazy load heavy libraries

3. **Monitor continuously**
   - Bundle analyzer
   - Performance budgets
   - Lighthouse audits

4. **Replace when possible**
   - Use native APIs
   - Implement simple features yourself
   - Choose lighter alternatives

**Common Swaps:**
```typescript
moment (288 KB) → date-fns (13 KB) or native
lodash (70 KB) → native JS
axios (13 KB) → fetch (0 KB)
jquery (90 KB) → native DOM
chartjs (240 KB) → recharts (93 KB) or custom
```

**Key Principles:**
> "Every kilobyte counts. Choose libraries wisely."

> "The best library is the one you don't need!"

**Golden Rule:**
> "Bundle size < 200 KB. Any library > 100 KB needs strong justification or lazy loading."

</details>

---

## Completion Status

Topic 16: Performance & Optimization - **COMPLETE** ✅

- Q151: Bundle size optimization ✅
- Q152: Tree shaking ✅
- Q153: Dynamic imports ✅
- Q154: Context API performance ✅
- Q155: Rendering performance at scale ✅
- Q156: Profiler API ✅
- Q157: Server-side caching strategies ✅
- Q158: Edge computing ✅
- Q159: Image optimization ✅
- Q160: Third-party library impact ✅

**All 10 questions completed!** 🎉
