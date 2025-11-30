# Day 1: Core Web Vitals - Measuring User Experience

## Introduction

Core Web Vitals are Google's metrics for measuring real-world user experience on the web. They directly impact your search rankings and, more importantly, how users perceive your application. Today, you'll learn what these metrics mean, how to measure them, and strategies to optimize each one.

## Learning Objectives

By the end of this lesson, you will be able to:
- Understand Core Web Vitals and their thresholds
- Measure performance using various tools
- Identify and diagnose performance issues
- Implement improvements for each metric
- Set up continuous performance monitoring

---

## Core Web Vitals Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    CORE WEB VITALS                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  LCP - Largest Contentful Paint                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ "How fast does the main content load?"                      │ │
│  │                                                              │ │
│  │ Good: ≤ 2.5s    Needs Work: ≤ 4.0s    Poor: > 4.0s         │ │
│  │ ████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │ │
│  │ 0s        2.5s              4.0s                            │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  INP - Interaction to Next Paint (replaced FID in 2024)         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ "How responsive is the page to user input?"                 │ │
│  │                                                              │ │
│  │ Good: ≤ 200ms   Needs Work: ≤ 500ms   Poor: > 500ms        │ │
│  │ ████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │ │
│  │ 0ms       200ms             500ms                           │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  CLS - Cumulative Layout Shift                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ "How stable is the visual layout?"                          │ │
│  │                                                              │ │
│  │ Good: ≤ 0.1     Needs Work: ≤ 0.25    Poor: > 0.25         │ │
│  │ ████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │ │
│  │ 0         0.1               0.25                            │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Other Important Metrics

```
┌─────────────────────────────────────────────────────────────────┐
│                   ADDITIONAL METRICS                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  TTFB - Time to First Byte                                       │
│  • Time until first byte of response received                   │
│  • Target: < 800ms                                              │
│  • Affected by: Server, network, CDN                            │
│                                                                  │
│  FCP - First Contentful Paint                                    │
│  • Time until first content is rendered                          │
│  • Target: < 1.8s                                               │
│  • Affected by: Render-blocking resources, font loading         │
│                                                                  │
│  TTI - Time to Interactive                                       │
│  • Time until page is fully interactive                          │
│  • Target: < 3.8s                                               │
│  • Affected by: JavaScript execution, main thread blocking      │
│                                                                  │
│  TBT - Total Blocking Time                                       │
│  • Sum of blocking time between FCP and TTI                     │
│  • Target: < 200ms                                              │
│  • Affected by: Long tasks, JavaScript execution                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Measuring Performance

### Browser DevTools

```javascript
// Performance API - Programmatic measurement
// Get navigation timing
const navigation = performance.getEntriesByType('navigation')[0];
console.log('TTFB:', navigation.responseStart - navigation.requestStart);
console.log('DOM Content Loaded:', navigation.domContentLoadedEventEnd);
console.log('Load Complete:', navigation.loadEventEnd);

// Get paint timing
const paint = performance.getEntriesByType('paint');
paint.forEach(entry => {
  console.log(`${entry.name}: ${entry.startTime}ms`);
});

// Get LCP
new PerformanceObserver((list) => {
  const entries = list.getEntries();
  const lastEntry = entries[entries.length - 1];
  console.log('LCP:', lastEntry.startTime);
  console.log('LCP Element:', lastEntry.element);
}).observe({ type: 'largest-contentful-paint', buffered: true });

// Get CLS
let clsValue = 0;
new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (!entry.hadRecentInput) {
      clsValue += entry.value;
    }
  }
  console.log('CLS:', clsValue);
}).observe({ type: 'layout-shift', buffered: true });

// Get INP (Interaction to Next Paint)
new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log('Interaction:', entry.name, entry.duration);
  }
}).observe({ type: 'event', buffered: true, durationThreshold: 16 });
```

### Web Vitals Library

```javascript
// Install: npm install web-vitals

import { onCLS, onINP, onLCP, onFCP, onTTFB } from 'web-vitals';

// Report to analytics
function sendToAnalytics(metric) {
  const body = JSON.stringify({
    name: metric.name,
    value: metric.value,
    rating: metric.rating, // 'good', 'needs-improvement', 'poor'
    delta: metric.delta,
    id: metric.id,
    navigationType: metric.navigationType
  });

  // Use sendBeacon for reliability
  if (navigator.sendBeacon) {
    navigator.sendBeacon('/api/analytics', body);
  } else {
    fetch('/api/analytics', { body, method: 'POST', keepalive: true });
  }
}

// Track all metrics
onCLS(sendToAnalytics);
onINP(sendToAnalytics);
onLCP(sendToAnalytics);
onFCP(sendToAnalytics);
onTTFB(sendToAnalytics);

// React component for development
import { useReportWebVitals } from 'next/web-vitals';

export function WebVitalsReporter() {
  useReportWebVitals((metric) => {
    console.log(metric);

    // Only in development
    if (process.env.NODE_ENV === 'development') {
      const color = {
        good: 'green',
        'needs-improvement': 'orange',
        poor: 'red'
      }[metric.rating];

      console.log(
        `%c${metric.name}: ${metric.value.toFixed(2)} (${metric.rating})`,
        `color: ${color}; font-weight: bold;`
      );
    }
  });

  return null;
}
```

### Lighthouse CI

```javascript
// lighthouserc.js
module.exports = {
  ci: {
    collect: {
      url: ['http://localhost:3000/', 'http://localhost:3000/products'],
      numberOfRuns: 3,
      settings: {
        preset: 'desktop',
        // Or for mobile
        // preset: 'mobile',
      }
    },
    assert: {
      assertions: {
        'categories:performance': ['error', { minScore: 0.9 }],
        'categories:accessibility': ['warn', { minScore: 0.9 }],
        'first-contentful-paint': ['error', { maxNumericValue: 2000 }],
        'largest-contentful-paint': ['error', { maxNumericValue: 2500 }],
        'cumulative-layout-shift': ['error', { maxNumericValue: 0.1 }],
        'total-blocking-time': ['error', { maxNumericValue: 300 }]
      }
    },
    upload: {
      target: 'temporary-public-storage',
      // Or use your own server
      // target: 'lhci',
      // serverBaseUrl: 'https://your-lhci-server.com'
    }
  }
};

// package.json scripts
{
  "scripts": {
    "lighthouse": "lhci autorun",
    "lighthouse:ci": "lhci autorun --config=lighthouserc.js"
  }
}

// GitHub Actions integration
// .github/workflows/lighthouse.yml
/*
name: Lighthouse CI
on: [push]
jobs:
  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci && npm run build
      - run: npm run start &
      - run: npx lhci autorun
*/
```

---

## Optimizing LCP (Largest Contentful Paint)

### Common LCP Elements

```
┌─────────────────────────────────────────────────────────────────┐
│                    LCP ELEMENT TYPES                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  • <img> elements                                                │
│  • <image> inside <svg>                                          │
│  • <video> poster image                                          │
│  • Background images via url()                                   │
│  • Block-level text elements (<h1>, <p>, etc.)                  │
│                                                                  │
│  LCP OPTIMIZATION STRATEGIES:                                    │
│                                                                  │
│  1. Optimize the critical path                                   │
│  2. Preload LCP resources                                        │
│  3. Optimize server response time                               │
│  4. Use CDN for static assets                                   │
│  5. Optimize images                                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Preload Critical Resources

```html
<!-- Preload LCP image -->
<head>
  <link
    rel="preload"
    as="image"
    href="/hero-image.webp"
    fetchpriority="high"
  />

  <!-- Preload critical font -->
  <link
    rel="preload"
    as="font"
    type="font/woff2"
    href="/fonts/inter.woff2"
    crossorigin
  />

  <!-- Preconnect to external origins -->
  <link rel="preconnect" href="https://cdn.example.com" />
  <link rel="dns-prefetch" href="https://cdn.example.com" />
</head>

<!-- Priority hints for images -->
<img
  src="/hero.webp"
  alt="Hero"
  fetchpriority="high"
  loading="eager"
/>

<!-- Lower priority for below-fold images -->
<img
  src="/footer-image.webp"
  alt="Footer"
  fetchpriority="low"
  loading="lazy"
/>
```

### Next.js Image Optimization

```jsx
// components/HeroImage.jsx
import Image from 'next/image';

export function HeroImage() {
  return (
    <Image
      src="/hero.jpg"
      alt="Hero image"
      width={1200}
      height={600}
      priority  // Disables lazy loading, preloads the image
      sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
      placeholder="blur"
      blurDataURL="data:image/jpeg;base64,/9j/4AAQSkZJRg..."
    />
  );
}

// next.config.js - Image optimization settings
module.exports = {
  images: {
    formats: ['image/avif', 'image/webp'],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
    minimumCacheTTL: 60 * 60 * 24 * 365, // 1 year
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'cdn.example.com',
      },
    ],
  },
};
```

### Server-Side Optimization

```javascript
// Optimize TTFB with caching
// next.config.js
module.exports = {
  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          {
            key: 'Cache-Control',
            value: 'public, max-age=31536000, immutable',
          },
        ],
      },
    ];
  },
};

// Express with compression
const express = require('express');
const compression = require('compression');

const app = express();

// Enable gzip/brotli compression
app.use(compression({
  level: 6,
  threshold: 1024,
  filter: (req, res) => {
    if (req.headers['x-no-compression']) {
      return false;
    }
    return compression.filter(req, res);
  }
}));

// Static file caching
app.use(express.static('public', {
  maxAge: '1y',
  etag: true,
  lastModified: true
}));
```

---

## Optimizing INP (Interaction to Next Paint)

### Understanding INP

```
┌─────────────────────────────────────────────────────────────────┐
│                    INP BREAKDOWN                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  User clicks button                                              │
│       │                                                          │
│       ▼                                                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │Input Delay  │ +│ Processing  │ +│Presentation │ = INP        │
│  │  (wait for  │  │   Time      │  │   Delay     │              │
│  │ main thread)│  │ (handlers)  │  │  (render)   │              │
│  └─────────────┘  └─────────────┘  └─────────────┘              │
│                                                                  │
│  OPTIMIZATION TARGETS:                                           │
│  • Input Delay: Keep main thread free                           │
│  • Processing: Break up long tasks                              │
│  • Presentation: Minimize render work                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Break Up Long Tasks

```javascript
// Bad: Long synchronous task blocks main thread
function processLargeArray(items) {
  items.forEach(item => {
    // Heavy computation
    heavyComputation(item);
  });
}

// Good: Yield to main thread periodically
async function processLargeArrayAsync(items) {
  const CHUNK_SIZE = 100;

  for (let i = 0; i < items.length; i += CHUNK_SIZE) {
    const chunk = items.slice(i, i + CHUNK_SIZE);

    // Process chunk
    chunk.forEach(item => heavyComputation(item));

    // Yield to main thread
    await yieldToMain();
  }
}

// Yield function using scheduler API or setTimeout
function yieldToMain() {
  return new Promise(resolve => {
    // Use scheduler.yield() when available (Chrome 115+)
    if ('scheduler' in window && 'yield' in scheduler) {
      scheduler.yield().then(resolve);
    } else {
      // Fallback to setTimeout
      setTimeout(resolve, 0);
    }
  });
}

// Using requestIdleCallback for non-urgent work
function processInBackground(items) {
  let index = 0;

  function processChunk(deadline) {
    while (index < items.length && deadline.timeRemaining() > 0) {
      heavyComputation(items[index]);
      index++;
    }

    if (index < items.length) {
      requestIdleCallback(processChunk);
    }
  }

  requestIdleCallback(processChunk);
}
```

### Optimize Event Handlers

```javascript
// Bad: Heavy computation in click handler
button.addEventListener('click', () => {
  // This blocks the main thread
  const result = expensiveCalculation();
  updateUI(result);
});

// Good: Defer non-critical work
button.addEventListener('click', () => {
  // Immediate visual feedback
  button.classList.add('loading');

  // Defer heavy work
  requestAnimationFrame(() => {
    const result = expensiveCalculation();
    updateUI(result);
    button.classList.remove('loading');
  });
});

// Better: Use Web Workers for heavy computation
// worker.js
self.onmessage = function(e) {
  const result = expensiveCalculation(e.data);
  self.postMessage(result);
};

// main.js
const worker = new Worker('worker.js');

button.addEventListener('click', () => {
  button.classList.add('loading');

  worker.postMessage(data);
  worker.onmessage = (e) => {
    updateUI(e.result);
    button.classList.remove('loading');
  };
});
```

### React Optimization for INP

```jsx
// Use transitions for non-urgent updates
import { useTransition, useState } from 'react';

function SearchResults() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  function handleChange(e) {
    const value = e.target.value;

    // Urgent: Update input immediately
    setQuery(value);

    // Non-urgent: Update results with lower priority
    startTransition(() => {
      setResults(filterResults(value));
    });
  }

  return (
    <div>
      <input value={query} onChange={handleChange} />
      {isPending && <Spinner />}
      <ResultsList results={results} />
    </div>
  );
}

// Use useDeferredValue for expensive renders
import { useDeferredValue, useMemo } from 'react';

function ProductList({ products, filter }) {
  // Defer the filter value
  const deferredFilter = useDeferredValue(filter);

  // Expensive filtering only runs when deferred value updates
  const filteredProducts = useMemo(
    () => products.filter(p => p.name.includes(deferredFilter)),
    [products, deferredFilter]
  );

  return (
    <ul style={{ opacity: filter !== deferredFilter ? 0.5 : 1 }}>
      {filteredProducts.map(product => (
        <ProductItem key={product.id} product={product} />
      ))}
    </ul>
  );
}

// Virtualize long lists
import { FixedSizeList } from 'react-window';

function VirtualizedList({ items }) {
  const Row = ({ index, style }) => (
    <div style={style}>
      {items[index].name}
    </div>
  );

  return (
    <FixedSizeList
      height={400}
      width="100%"
      itemCount={items.length}
      itemSize={50}
    >
      {Row}
    </FixedSizeList>
  );
}
```

---

## Optimizing CLS (Cumulative Layout Shift)

### Common CLS Causes

```
┌─────────────────────────────────────────────────────────────────┐
│                    CLS CAUSES & SOLUTIONS                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  CAUSE                          SOLUTION                         │
│  ─────                          ────────                         │
│  Images without dimensions  →   Always set width/height         │
│  Ads/embeds without space   →   Reserve space with aspect-ratio │
│  Dynamically injected content → Use transforms, not layout      │
│  Web fonts causing FOUT      →   Use font-display: optional     │
│  Late-loading content        →   Use skeleton/placeholder       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Image Dimensions

```html
<!-- Always include width and height -->
<img
  src="image.jpg"
  alt="Description"
  width="800"
  height="600"
/>

<!-- Modern CSS approach with aspect-ratio -->
<style>
  .responsive-image {
    width: 100%;
    height: auto;
    aspect-ratio: 16 / 9;
    object-fit: cover;
  }
</style>

<img
  src="image.jpg"
  alt="Description"
  class="responsive-image"
/>
```

### Reserve Space for Dynamic Content

```css
/* Reserve space for ads */
.ad-container {
  min-height: 250px;
  width: 300px;
  background: #f0f0f0;
}

/* Aspect ratio for videos/embeds */
.video-container {
  aspect-ratio: 16 / 9;
  width: 100%;
  background: #000;
}

/* Skeleton loading */
.skeleton {
  background: linear-gradient(
    90deg,
    #f0f0f0 25%,
    #e0e0e0 50%,
    #f0f0f0 75%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
}

@keyframes shimmer {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}

.card-skeleton {
  height: 200px;
  border-radius: 8px;
}
```

### Font Loading Strategies

```css
/* Prevent FOUT (Flash of Unstyled Text) */
@font-face {
  font-family: 'CustomFont';
  src: url('/fonts/custom.woff2') format('woff2');
  font-display: swap; /* Show fallback, swap when loaded */
  /* Or use 'optional' to prevent layout shift entirely */
  /* font-display: optional; */
}

/* Size-adjust to match fallback font metrics */
@font-face {
  font-family: 'CustomFont';
  src: url('/fonts/custom.woff2') format('woff2');
  font-display: swap;
  size-adjust: 105%;
  ascent-override: 90%;
  descent-override: 20%;
  line-gap-override: 0%;
}

/* Fallback font stack with similar metrics */
body {
  font-family: 'CustomFont', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
}
```

### Animations Without Layout Shift

```css
/* Bad: Animating layout properties causes shifts */
.notification-bad {
  animation: slideDown 0.3s ease;
}

@keyframes slideDown {
  from {
    margin-top: -100px;  /* Causes layout shift! */
  }
  to {
    margin-top: 0;
  }
}

/* Good: Use transform (doesn't affect layout) */
.notification-good {
  animation: slideDownGood 0.3s ease;
}

@keyframes slideDownGood {
  from {
    transform: translateY(-100%);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}

/* Reserve space for notifications */
.notification-container {
  min-height: 60px;
  position: relative;
}

.notification {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
}
```

### React Component with CLS Prevention

```jsx
// components/Image.jsx - CLS-safe image component
import { useState } from 'react';

export function Image({ src, alt, width, height, className }) {
  const [loaded, setLoaded] = useState(false);
  const [error, setError] = useState(false);

  return (
    <div
      className={`image-container ${className}`}
      style={{
        aspectRatio: `${width} / ${height}`,
        backgroundColor: '#f0f0f0'
      }}
    >
      {!loaded && !error && (
        <div className="skeleton" style={{ width: '100%', height: '100%' }} />
      )}

      <img
        src={src}
        alt={alt}
        width={width}
        height={height}
        onLoad={() => setLoaded(true)}
        onError={() => setError(true)}
        style={{
          opacity: loaded ? 1 : 0,
          transition: 'opacity 0.3s'
        }}
      />

      {error && (
        <div className="error-placeholder">
          Failed to load image
        </div>
      )}
    </div>
  );
}

// components/DynamicContent.jsx - Content with reserved space
export function DynamicContent({ isLoading, children, height = 200 }) {
  return (
    <div
      style={{
        minHeight: height,
        transition: 'min-height 0.3s'
      }}
    >
      {isLoading ? (
        <div className="skeleton" style={{ height }} />
      ) : (
        children
      )}
    </div>
  );
}
```

---

## Performance Budget

```javascript
// performance-budget.js
const BUDGETS = {
  // Size budgets (compressed)
  javascript: 200 * 1024,     // 200 KB
  css: 50 * 1024,             // 50 KB
  images: 500 * 1024,         // 500 KB per page
  fonts: 100 * 1024,          // 100 KB
  total: 1000 * 1024,         // 1 MB total

  // Timing budgets
  lcp: 2500,                  // 2.5s
  fcp: 1800,                  // 1.8s
  tti: 3800,                  // 3.8s
  tbt: 200,                   // 200ms

  // Count budgets
  requests: 50,
  thirdPartyRequests: 10
};

// Webpack bundle analyzer integration
// webpack.config.js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: 'static',
      reportFilename: 'bundle-report.html',
      openAnalyzer: false
    })
  ],
  performance: {
    maxAssetSize: BUDGETS.javascript,
    maxEntrypointSize: BUDGETS.total,
    hints: 'error'
  }
};

// Next.js bundle analysis
// package.json
{
  "scripts": {
    "analyze": "ANALYZE=true next build"
  }
}

// next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true'
});

module.exports = withBundleAnalyzer({
  // config
});
```

---

## Continuous Monitoring

```javascript
// Real User Monitoring (RUM)
import { onCLS, onINP, onLCP } from 'web-vitals';

class PerformanceMonitor {
  constructor(endpoint) {
    this.endpoint = endpoint;
    this.metrics = {};
    this.setupObservers();
  }

  setupObservers() {
    onCLS(metric => this.record('CLS', metric));
    onINP(metric => this.record('INP', metric));
    onLCP(metric => this.record('LCP', metric));
  }

  record(name, metric) {
    this.metrics[name] = {
      value: metric.value,
      rating: metric.rating,
      delta: metric.delta
    };

    // Send to analytics
    this.send(name, metric);
  }

  send(name, metric) {
    const data = {
      name,
      value: metric.value,
      rating: metric.rating,
      page: window.location.pathname,
      userAgent: navigator.userAgent,
      connection: navigator.connection?.effectiveType,
      timestamp: Date.now()
    };

    // Use sendBeacon for reliability
    if (navigator.sendBeacon) {
      navigator.sendBeacon(this.endpoint, JSON.stringify(data));
    }
  }

  getReport() {
    return this.metrics;
  }
}

// Initialize monitoring
const monitor = new PerformanceMonitor('/api/performance');

// API endpoint to receive metrics
// pages/api/performance.js
export default async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).end();
  }

  const metrics = req.body;

  // Store in database
  await db.performanceMetrics.create({ data: metrics });

  // Alert if poor performance
  if (metrics.rating === 'poor') {
    await alertService.notify({
      type: 'performance',
      metric: metrics.name,
      value: metrics.value,
      page: metrics.page
    });
  }

  res.status(200).end();
}
```

---

## Practice Exercises

### Exercise 1: Audit a Page

Use Chrome DevTools and Lighthouse to:
- Identify the LCP element
- Find layout shifts
- Measure interaction responsiveness
- Create an optimization plan

### Exercise 2: Optimize Images

Take a page with unoptimized images:
- Add proper dimensions
- Implement lazy loading
- Use modern formats (WebP/AVIF)
- Add priority hints

### Exercise 3: Fix Layout Shifts

Find and fix CLS issues:
- Add aspect ratios to images
- Reserve space for dynamic content
- Optimize font loading
- Use skeleton loaders

---

## Key Takeaways

1. **Measure first** - Use Lighthouse, DevTools, and RUM data
2. **LCP** - Preload critical resources, optimize images
3. **INP** - Break up long tasks, use Web Workers
4. **CLS** - Reserve space, use transforms, optimize fonts
5. **Set budgets** - Define and enforce performance limits
6. **Monitor continuously** - Track real user experience

---

## What's Next?

Tomorrow, we'll dive deep into **Image Optimization** - learning advanced techniques for serving the right images to the right devices with the smallest possible file sizes.
