# Day 3: Code Splitting - Load Only What You Need

## Introduction

Modern web applications can have JavaScript bundles measured in megabytes. Code splitting breaks your bundle into smaller chunks that can be loaded on demand, dramatically improving initial load time and Time to Interactive. Today, you'll learn various code splitting strategies and how to implement them effectively.

## Learning Objectives

By the end of this lesson, you will be able to:
- Understand when and why to code split
- Implement route-based splitting in React and Next.js
- Use dynamic imports for component-level splitting
- Configure webpack and Vite for optimal chunking
- Analyze and optimize your bundle

---

## Why Code Splitting?

```
┌─────────────────────────────────────────────────────────────────┐
│                  CODE SPLITTING BENEFITS                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  WITHOUT CODE SPLITTING:                                         │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                     bundle.js (2MB)                       │   │
│  │  [Home][Products][Cart][Admin][Charts][PDF][Maps]...     │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
│  User visits homepage → Downloads ALL 2MB → Slow initial load   │
│                                                                  │
│  ─────────────────────────────────────────────────────────────  │
│                                                                  │
│  WITH CODE SPLITTING:                                            │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐             │
│  │ main.js     │ │ products.js  │ │ admin.js     │             │
│  │ (100KB)     │ │ (150KB)      │ │ (300KB)      │             │
│  └──────────────┘ └──────────────┘ └──────────────┘             │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐             │
│  │ charts.js   │ │ pdf.js       │ │ maps.js      │             │
│  │ (200KB)     │ │ (400KB)      │ │ (250KB)      │             │
│  └──────────────┘ └──────────────┘ └──────────────┘             │
│                                                                  │
│  User visits homepage → Downloads only 100KB → Fast!            │
│  User visits products → Downloads products.js on demand         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Splitting Strategies

```
┌─────────────────────────────────────────────────────────────────┐
│                  SPLITTING STRATEGIES                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. ROUTE-BASED SPLITTING                                        │
│     Split by page/route - most common and effective             │
│     /home → home.js                                              │
│     /products → products.js                                      │
│     /admin → admin.js                                            │
│                                                                  │
│  2. COMPONENT-BASED SPLITTING                                    │
│     Split heavy components that aren't immediately needed        │
│     Modal → modal.js (load when opened)                          │
│     Chart → chart.js (load when scrolled into view)             │
│                                                                  │
│  3. VENDOR SPLITTING                                             │
│     Separate third-party libraries (better caching)             │
│     React, lodash → vendors.js                                   │
│     Your code → app.js                                           │
│                                                                  │
│  4. CONDITIONAL SPLITTING                                        │
│     Based on user features or device                            │
│     Admin features → admin.js (only for admins)                 │
│     Mobile-specific → mobile.js                                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Dynamic Imports

### Basic Dynamic Import

```javascript
// Static import - always loaded
import { heavyFunction } from './heavy-module';

// Dynamic import - loaded on demand
async function loadHeavyModule() {
  const { heavyFunction } = await import('./heavy-module');
  heavyFunction();
}

// With error handling
async function loadModule() {
  try {
    const module = await import('./feature-module');
    module.init();
  } catch (error) {
    console.error('Failed to load module:', error);
    // Show fallback or error UI
  }
}

// Named exports
async function loadChartLibrary() {
  const { Chart, LineChart, BarChart } = await import('chart-library');
  return new Chart();
}

// Default export
async function loadComponent() {
  const { default: Component } = await import('./Component');
  return Component;
}
```

### Webpack Magic Comments

```javascript
// Name the chunk for better debugging
const AdminPanel = () => import(
  /* webpackChunkName: "admin" */
  './AdminPanel'
);

// Prefetch - load in idle time (likely to be needed soon)
const ProductDetails = () => import(
  /* webpackChunkName: "product-details" */
  /* webpackPrefetch: true */
  './ProductDetails'
);

// Preload - load immediately (definitely needed soon)
const CheckoutFlow = () => import(
  /* webpackChunkName: "checkout" */
  /* webpackPreload: true */
  './CheckoutFlow'
);

// Disable chunk splitting for this import
const CriticalComponent = () => import(
  /* webpackMode: "eager" */
  './CriticalComponent'
);

// Multiple possible imports (generates chunks for each)
async function loadLocale(locale) {
  return import(
    /* webpackChunkName: "locale-[request]" */
    `./locales/${locale}.json`
  );
}
```

---

## React Code Splitting

### React.lazy and Suspense

```jsx
import React, { Suspense, lazy } from 'react';

// Lazy load components
const Dashboard = lazy(() => import('./Dashboard'));
const Settings = lazy(() => import('./Settings'));
const Analytics = lazy(() => import('./Analytics'));

// Loading fallback component
function LoadingSpinner() {
  return (
    <div className="loading-spinner">
      <div className="spinner" />
      <p>Loading...</p>
    </div>
  );
}

// Error boundary for lazy components
class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error">
          <h2>Something went wrong</h2>
          <button onClick={() => window.location.reload()}>
            Reload page
          </button>
        </div>
      );
    }
    return this.props.children;
  }
}

// App with lazy routes
function App() {
  return (
    <ErrorBoundary>
      <Suspense fallback={<LoadingSpinner />}>
        <Routes>
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/settings" element={<Settings />} />
          <Route path="/analytics" element={<Analytics />} />
        </Routes>
      </Suspense>
    </ErrorBoundary>
  );
}
```

### Component-Level Splitting

```jsx
import { lazy, Suspense, useState } from 'react';

// Heavy components loaded on demand
const HeavyChart = lazy(() => import('./HeavyChart'));
const PDFViewer = lazy(() => import('./PDFViewer'));
const RichTextEditor = lazy(() => import('./RichTextEditor'));

function Dashboard() {
  const [showChart, setShowChart] = useState(false);
  const [showPDF, setShowPDF] = useState(false);

  return (
    <div>
      <h1>Dashboard</h1>

      {/* Chart loaded only when button clicked */}
      <button onClick={() => setShowChart(true)}>
        Show Analytics
      </button>

      {showChart && (
        <Suspense fallback={<div>Loading chart...</div>}>
          <HeavyChart data={analyticsData} />
        </Suspense>
      )}

      {/* PDF viewer loaded on demand */}
      <button onClick={() => setShowPDF(true)}>
        View Report
      </button>

      {showPDF && (
        <Suspense fallback={<div>Loading PDF viewer...</div>}>
          <PDFViewer url="/reports/monthly.pdf" />
        </Suspense>
      )}
    </div>
  );
}

// Modal that loads content lazily
const ModalContent = lazy(() => import('./HeavyModalContent'));

function Modal({ isOpen, onClose }) {
  if (!isOpen) return null;

  return (
    <div className="modal-overlay">
      <div className="modal">
        <Suspense fallback={<div>Loading...</div>}>
          <ModalContent />
        </Suspense>
        <button onClick={onClose}>Close</button>
      </div>
    </div>
  );
}
```

### Preloading Components

```jsx
import { lazy, Suspense, useEffect } from 'react';

// Define lazy component
const HeavyComponent = lazy(() => import('./HeavyComponent'));

// Preload function
const preloadHeavyComponent = () => import('./HeavyComponent');

function App() {
  // Preload on hover
  const handleMouseEnter = () => {
    preloadHeavyComponent();
  };

  // Preload after initial render
  useEffect(() => {
    // Wait for main content to load, then preload
    const timer = setTimeout(() => {
      preloadHeavyComponent();
    }, 2000);

    return () => clearTimeout(timer);
  }, []);

  return (
    <div>
      <button onMouseEnter={handleMouseEnter}>
        Show Heavy Feature
      </button>
    </div>
  );
}

// Custom hook for preloading
function usePreload(importFn, delay = 0) {
  useEffect(() => {
    const timer = setTimeout(() => {
      importFn();
    }, delay);

    return () => clearTimeout(timer);
  }, [importFn, delay]);
}

// Usage
const Dashboard = lazy(() => import('./Dashboard'));
const preloadDashboard = () => import('./Dashboard');

function Nav() {
  // Preload dashboard 1 second after nav renders
  usePreload(preloadDashboard, 1000);

  return <nav>...</nav>;
}
```

---

## Next.js Code Splitting

### Automatic Route Splitting

```jsx
// Next.js automatically splits by route
// pages/index.js → chunk for home
// pages/products.js → chunk for products
// pages/admin/dashboard.js → chunk for admin

// App Router (app directory) - same automatic splitting
// app/page.js → home chunk
// app/products/page.js → products chunk
// app/admin/dashboard/page.js → admin chunk
```

### Dynamic Imports in Next.js

```jsx
// next/dynamic with SSR disabled
import dynamic from 'next/dynamic';

// Client-only component (no SSR)
const Chart = dynamic(
  () => import('../components/Chart'),
  {
    ssr: false,
    loading: () => <div>Loading chart...</div>
  }
);

// With custom loading component
const Editor = dynamic(
  () => import('../components/RichTextEditor'),
  {
    loading: () => <EditorSkeleton />,
    ssr: false
  }
);

// Named exports
const DynamicComponent = dynamic(
  () => import('../components/MultiExport').then(mod => mod.SpecificComponent)
);

// With suspense (React 18+)
const SuspenseComponent = dynamic(
  () => import('../components/HeavyComponent'),
  { suspense: true }
);

function Page() {
  return (
    <Suspense fallback={<Loading />}>
      <SuspenseComponent />
    </Suspense>
  );
}
```

### Conditional Loading

```jsx
import dynamic from 'next/dynamic';
import { useSession } from 'next-auth/react';

// Admin panel - only load for admins
const AdminPanel = dynamic(
  () => import('../components/AdminPanel'),
  { ssr: false }
);

// Mobile-specific component
const MobileNav = dynamic(
  () => import('../components/MobileNav'),
  { ssr: false }
);

function Dashboard() {
  const { data: session } = useSession();
  const isMobile = useMediaQuery('(max-width: 768px)');

  return (
    <div>
      {session?.user?.role === 'admin' && <AdminPanel />}
      {isMobile && <MobileNav />}
    </div>
  );
}

// Feature flag based loading
const ExperimentalFeature = dynamic(
  () => import('../components/ExperimentalFeature'),
  {
    ssr: false,
    loading: () => null
  }
);

function App() {
  const { isEnabled } = useFeatureFlag('experimental-feature');

  return (
    <div>
      {isEnabled && <ExperimentalFeature />}
    </div>
  );
}
```

---

## Webpack Configuration

### Manual Chunk Configuration

```javascript
// webpack.config.js
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      minSize: 20000,      // Minimum chunk size (20KB)
      maxSize: 244000,     // Maximum chunk size (244KB)
      minChunks: 1,        // Minimum times a module must be shared
      maxAsyncRequests: 30,
      maxInitialRequests: 30,
      cacheGroups: {
        // Vendor chunk for node_modules
        vendors: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
          priority: 10
        },
        // Separate chunk for React
        react: {
          test: /[\\/]node_modules[\\/](react|react-dom|scheduler)[\\/]/,
          name: 'react',
          chunks: 'all',
          priority: 20
        },
        // Separate chunk for large libraries
        charts: {
          test: /[\\/]node_modules[\\/](chart\.js|recharts|d3)[\\/]/,
          name: 'charts',
          chunks: 'async',
          priority: 15
        },
        // Common modules shared between chunks
        common: {
          name: 'common',
          minChunks: 2,
          chunks: 'all',
          priority: 5,
          reuseExistingChunk: true
        }
      }
    }
  }
};
```

### Module Federation

```javascript
// webpack.config.js - Host App
const { ModuleFederationPlugin } = require('webpack').container;

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'host',
      remotes: {
        remoteApp: 'remoteApp@http://localhost:3001/remoteEntry.js'
      },
      shared: {
        react: { singleton: true, eager: true },
        'react-dom': { singleton: true, eager: true }
      }
    })
  ]
};

// webpack.config.js - Remote App
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'remoteApp',
      filename: 'remoteEntry.js',
      exposes: {
        './Button': './src/Button',
        './Card': './src/Card'
      },
      shared: {
        react: { singleton: true },
        'react-dom': { singleton: true }
      }
    })
  ]
};

// Usage in Host App
const RemoteButton = lazy(() => import('remoteApp/Button'));

function App() {
  return (
    <Suspense fallback="Loading...">
      <RemoteButton />
    </Suspense>
  );
}
```

---

## Vite Configuration

```javascript
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          // Vendor chunks
          'react-vendor': ['react', 'react-dom'],
          'router': ['react-router-dom'],
          'ui': ['@chakra-ui/react', '@emotion/react'],

          // Feature chunks
          'charts': ['recharts', 'd3'],
          'forms': ['react-hook-form', 'zod'],

          // Or use a function for more control
          // manualChunks(id) {
          //   if (id.includes('node_modules')) {
          //     if (id.includes('react')) return 'react-vendor';
          //     if (id.includes('lodash')) return 'lodash';
          //     return 'vendor';
          //   }
          // }
        }
      }
    },
    // Chunk size warnings
    chunkSizeWarningLimit: 500,

    // Minification
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true,
        drop_debugger: true
      }
    }
  }
});
```

---

## Bundle Analysis

### Webpack Bundle Analyzer

```javascript
// webpack.config.js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: 'static',
      reportFilename: 'bundle-report.html',
      openAnalyzer: false,
      generateStatsFile: true,
      statsFilename: 'bundle-stats.json'
    })
  ]
};

// package.json scripts
{
  "scripts": {
    "analyze": "webpack --profile --json > stats.json && webpack-bundle-analyzer stats.json",
    "build:analyze": "ANALYZE=true npm run build"
  }
}
```

### Next.js Bundle Analysis

```javascript
// next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
  openAnalyzer: true
});

module.exports = withBundleAnalyzer({
  // your config
});

// Run analysis
// ANALYZE=true npm run build
```

### Source Map Explorer

```bash
# Install
npm install -D source-map-explorer

# Generate source maps and analyze
npm run build
npx source-map-explorer 'build/static/js/*.js'
```

### Import Cost (VS Code Extension)

```javascript
// Shows inline size of imports in VS Code
import moment from 'moment';        // 288KB
import dayjs from 'dayjs';          // 6.5KB  ✓
import { format } from 'date-fns';  // 5KB    ✓

import _ from 'lodash';             // 71KB
import groupBy from 'lodash/groupBy'; // 5KB  ✓
```

---

## Optimization Patterns

### Tree Shaking

```javascript
// Bad - imports entire library
import _ from 'lodash';
_.map(arr, fn);

// Good - tree-shakeable import
import map from 'lodash/map';
map(arr, fn);

// Or use lodash-es for better tree shaking
import { map } from 'lodash-es';

// Named exports are tree-shakeable
// utils.js
export function usedFunction() { }
export function unusedFunction() { } // Will be removed

// import { usedFunction } from './utils';

// Default exports are NOT tree-shakeable
// utils.js
export default {
  usedFunction() { },
  unusedFunction() { } // Won't be removed!
};
```

### Lazy Loading Third-Party Libraries

```jsx
// Instead of importing at top level
import Chart from 'chart.js/auto';

// Load on demand
async function renderChart(canvas, data) {
  const { Chart } = await import('chart.js/auto');

  new Chart(canvas, {
    type: 'line',
    data: data
  });
}

// React component with lazy library
function ChartComponent({ data }) {
  const canvasRef = useRef();

  useEffect(() => {
    let chart;

    async function initChart() {
      const { Chart } = await import('chart.js/auto');
      chart = new Chart(canvasRef.current, {
        type: 'line',
        data
      });
    }

    initChart();

    return () => chart?.destroy();
  }, [data]);

  return <canvas ref={canvasRef} />;
}
```

### Route-Based Preloading

```jsx
import { lazy, Suspense } from 'react';
import { Link, useLocation } from 'react-router-dom';

// Define routes with preload functions
const routes = {
  '/dashboard': {
    component: lazy(() => import('./pages/Dashboard')),
    preload: () => import('./pages/Dashboard')
  },
  '/products': {
    component: lazy(() => import('./pages/Products')),
    preload: () => import('./pages/Products')
  },
  '/settings': {
    component: lazy(() => import('./pages/Settings')),
    preload: () => import('./pages/Settings')
  }
};

// Preloading Link component
function PreloadLink({ to, children, ...props }) {
  const handleMouseEnter = () => {
    const route = routes[to];
    if (route?.preload) {
      route.preload();
    }
  };

  return (
    <Link
      to={to}
      onMouseEnter={handleMouseEnter}
      onFocus={handleMouseEnter}
      {...props}
    >
      {children}
    </Link>
  );
}

// Navigation with preloading
function Nav() {
  return (
    <nav>
      <PreloadLink to="/dashboard">Dashboard</PreloadLink>
      <PreloadLink to="/products">Products</PreloadLink>
      <PreloadLink to="/settings">Settings</PreloadLink>
    </nav>
  );
}
```

---

## Performance Budget for JavaScript

```javascript
// webpack.config.js
module.exports = {
  performance: {
    maxEntrypointSize: 250000,  // 250 KB
    maxAssetSize: 200000,       // 200 KB
    hints: 'error',             // Fail build if exceeded
    assetFilter: (assetFilename) => {
      return assetFilename.endsWith('.js');
    }
  }
};

// Custom budget checker
// scripts/check-bundle-size.js
const fs = require('fs');
const path = require('path');

const BUDGET = {
  'main': 150000,      // 150 KB
  'vendors': 200000,   // 200 KB
  'total': 400000      // 400 KB
};

function checkBundleSize() {
  const buildDir = path.join(__dirname, '../build/static/js');
  const files = fs.readdirSync(buildDir);

  let totalSize = 0;
  const violations = [];

  files.forEach(file => {
    if (!file.endsWith('.js')) return;

    const filePath = path.join(buildDir, file);
    const stats = fs.statSync(filePath);
    totalSize += stats.size;

    // Check individual budgets
    const budgetKey = Object.keys(BUDGET).find(key =>
      file.includes(key)
    );

    if (budgetKey && stats.size > BUDGET[budgetKey]) {
      violations.push({
        file,
        size: stats.size,
        budget: BUDGET[budgetKey]
      });
    }
  });

  // Check total budget
  if (totalSize > BUDGET.total) {
    violations.push({
      file: 'TOTAL',
      size: totalSize,
      budget: BUDGET.total
    });
  }

  if (violations.length > 0) {
    console.error('Bundle size violations:');
    violations.forEach(v => {
      console.error(`  ${v.file}: ${(v.size / 1024).toFixed(1)}KB > ${(v.budget / 1024).toFixed(1)}KB`);
    });
    process.exit(1);
  }

  console.log(`Total bundle size: ${(totalSize / 1024).toFixed(1)}KB`);
}

checkBundleSize();
```

---

## Practice Exercises

### Exercise 1: Route-Based Splitting

Take an existing React app and:
- Convert static imports to lazy imports
- Add Suspense boundaries
- Implement error boundaries
- Add preloading on hover

### Exercise 2: Component Splitting

Identify and split heavy components:
- Find components over 50KB
- Convert to lazy loading
- Add appropriate loading states
- Measure before/after bundle size

### Exercise 3: Bundle Analysis

Analyze an existing project:
- Generate bundle report
- Identify largest dependencies
- Find duplicate modules
- Create optimization plan

---

## Key Takeaways

1. **Route-split first** - Biggest impact with least effort
2. **Lazy load below-fold** - Don't load what users don't see
3. **Preload on intent** - Hover/focus preloading for smooth UX
4. **Watch your vendors** - Third-party code adds up fast
5. **Measure continuously** - Use bundle analyzer in CI
6. **Set budgets** - Prevent regressions with size limits

---

## What's Next?

Tomorrow, we'll explore **Backend Performance** - learning how to optimize Node.js and API performance with caching, database optimization, and efficient data handling.
