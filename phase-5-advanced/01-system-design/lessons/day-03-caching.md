# Day 3: Caching - Speed Up Your Applications

## Introduction

Caching is one of the most powerful techniques for improving application performance. By storing frequently accessed data closer to where it's needed, you can dramatically reduce response times, database load, and infrastructure costs. Today, you'll learn caching strategies that can make your applications 10-100x faster.

## Learning Objectives

By the end of this lesson, you will be able to:
- Understand different caching layers and strategies
- Implement Redis caching in Node.js applications
- Use CDNs for static asset and edge caching
- Apply cache invalidation strategies
- Design effective caching architectures

---

## The Caching Pyramid

```
┌─────────────────────────────────────────────────────────────────┐
│                      CACHING LAYERS                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  CLOSEST TO USER ──────────────────────────── FARTHEST          │
│                                                                  │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌────────┐ │
│  │ Browser │→ │   CDN   │→ │  Nginx  │→ │  Redis  │→ │Database│ │
│  │  Cache  │  │  Edge   │  │  Cache  │  │ In-Mem  │  │ Cache  │ │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘  └────────┘ │
│                                                                  │
│  Speed:  Fastest ◀──────────────────────────────▶ Slowest      │
│  Size:   Smallest ◀─────────────────────────────▶ Largest      │
│  Cost:   Cheapest ◀─────────────────────────────▶ Expensive    │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Response Time Comparison:                                 │   │
│  │ • Browser Cache:    ~0ms (instant)                       │   │
│  │ • CDN Edge:         ~10-50ms                             │   │
│  │ • Redis Cache:      ~1-5ms                               │   │
│  │ • Database Query:   ~50-500ms                            │   │
│  │ • External API:     ~100-1000ms                          │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Caching Strategies

### Cache-Aside (Lazy Loading)

```javascript
// cache-aside.js - Most common pattern
const redis = require('redis');
const client = redis.createClient();

class CacheAside {
  constructor(redisClient, ttlSeconds = 3600) {
    this.redis = redisClient;
    this.ttl = ttlSeconds;
  }

  async get(key, fetchFunction) {
    // 1. Check cache first
    const cached = await this.redis.get(key);

    if (cached) {
      console.log(`Cache HIT: ${key}`);
      return JSON.parse(cached);
    }

    // 2. Cache miss - fetch from source
    console.log(`Cache MISS: ${key}`);
    const data = await fetchFunction();

    // 3. Store in cache for next time
    await this.redis.setEx(key, this.ttl, JSON.stringify(data));

    return data;
  }

  async invalidate(key) {
    await this.redis.del(key);
  }

  async invalidatePattern(pattern) {
    const keys = await this.redis.keys(pattern);
    if (keys.length > 0) {
      await this.redis.del(keys);
    }
  }
}

// Usage
const cache = new CacheAside(client);

async function getUser(userId) {
  return cache.get(`user:${userId}`, async () => {
    // This only runs on cache miss
    const user = await db.query('SELECT * FROM users WHERE id = $1', [userId]);
    return user;
  });
}

async function updateUser(userId, data) {
  await db.query('UPDATE users SET ... WHERE id = $1', [userId]);
  // Invalidate cache after update
  await cache.invalidate(`user:${userId}`);
}
```

### Write-Through Cache

```javascript
// write-through.js - Write to cache and database together
class WriteThroughCache {
  constructor(redisClient, ttlSeconds = 3600) {
    this.redis = redisClient;
    this.ttl = ttlSeconds;
  }

  async write(key, data, writeFunction) {
    // 1. Write to database first
    await writeFunction(data);

    // 2. Write to cache
    await this.redis.setEx(key, this.ttl, JSON.stringify(data));

    return data;
  }

  async read(key, fetchFunction) {
    const cached = await this.redis.get(key);

    if (cached) {
      return JSON.parse(cached);
    }

    const data = await fetchFunction();
    await this.redis.setEx(key, this.ttl, JSON.stringify(data));
    return data;
  }
}

// Usage
const cache = new WriteThroughCache(client);

async function createProduct(productData) {
  return cache.write(
    `product:${productData.id}`,
    productData,
    async (data) => {
      await db.query(
        'INSERT INTO products (id, name, price) VALUES ($1, $2, $3)',
        [data.id, data.name, data.price]
      );
    }
  );
}
```

### Write-Behind (Write-Back) Cache

```javascript
// write-behind.js - Write to cache, async write to database
class WriteBehindCache {
  constructor(redisClient) {
    this.redis = redisClient;
    this.writeQueue = [];
    this.flushInterval = 5000; // 5 seconds

    // Periodically flush writes to database
    setInterval(() => this.flushWrites(), this.flushInterval);
  }

  async write(key, data) {
    // 1. Write to cache immediately (fast)
    await this.redis.set(key, JSON.stringify(data));

    // 2. Queue for database write
    this.writeQueue.push({ key, data, timestamp: Date.now() });

    return data;
  }

  async flushWrites() {
    if (this.writeQueue.length === 0) return;

    const batch = [...this.writeQueue];
    this.writeQueue = [];

    try {
      // Batch write to database
      await this.batchInsert(batch);
      console.log(`Flushed ${batch.length} writes to database`);
    } catch (error) {
      // Re-queue failed writes
      this.writeQueue = [...batch, ...this.writeQueue];
      console.error('Batch write failed, will retry');
    }
  }

  async batchInsert(items) {
    // Use database transaction for batch insert
    const client = await db.pool.connect();
    try {
      await client.query('BEGIN');
      for (const item of items) {
        await client.query(
          'INSERT INTO events (key, data) VALUES ($1, $2) ON CONFLICT (key) DO UPDATE SET data = $2',
          [item.key, item.data]
        );
      }
      await client.query('COMMIT');
    } catch (error) {
      await client.query('ROLLBACK');
      throw error;
    } finally {
      client.release();
    }
  }
}

// Good for: High-write scenarios like analytics, logging
```

---

## Redis Caching in Node.js

### Basic Setup

```javascript
// redis-setup.js
const { createClient } = require('redis');

class RedisCache {
  constructor() {
    this.client = null;
  }

  async connect() {
    this.client = createClient({
      url: process.env.REDIS_URL || 'redis://localhost:6379',
      socket: {
        reconnectStrategy: (retries) => {
          if (retries > 10) {
            return new Error('Max reconnection attempts reached');
          }
          return Math.min(retries * 100, 3000);
        }
      }
    });

    this.client.on('error', (err) => console.error('Redis Error:', err));
    this.client.on('connect', () => console.log('Redis connected'));
    this.client.on('reconnecting', () => console.log('Redis reconnecting...'));

    await this.client.connect();
    return this;
  }

  // String operations
  async set(key, value, ttlSeconds) {
    const stringValue = typeof value === 'object'
      ? JSON.stringify(value)
      : String(value);

    if (ttlSeconds) {
      await this.client.setEx(key, ttlSeconds, stringValue);
    } else {
      await this.client.set(key, stringValue);
    }
  }

  async get(key) {
    const value = await this.client.get(key);
    if (!value) return null;

    try {
      return JSON.parse(value);
    } catch {
      return value;
    }
  }

  // Hash operations (for objects)
  async hSet(key, field, value) {
    await this.client.hSet(key, field, JSON.stringify(value));
  }

  async hGet(key, field) {
    const value = await this.client.hGet(key, field);
    return value ? JSON.parse(value) : null;
  }

  async hGetAll(key) {
    const data = await this.client.hGetAll(key);
    const result = {};
    for (const [field, value] of Object.entries(data)) {
      result[field] = JSON.parse(value);
    }
    return result;
  }

  // List operations (for queues)
  async lPush(key, value) {
    await this.client.lPush(key, JSON.stringify(value));
  }

  async rPop(key) {
    const value = await this.client.rPop(key);
    return value ? JSON.parse(value) : null;
  }

  // Set operations (for unique collections)
  async sAdd(key, ...members) {
    await this.client.sAdd(key, members.map(m => JSON.stringify(m)));
  }

  async sMembers(key) {
    const members = await this.client.sMembers(key);
    return members.map(m => JSON.parse(m));
  }

  // Sorted set (for leaderboards, rankings)
  async zAdd(key, score, member) {
    await this.client.zAdd(key, { score, value: JSON.stringify(member) });
  }

  async zRange(key, start, stop, withScores = false) {
    const options = withScores ? { REV: true, WITHSCORES: true } : { REV: true };
    const results = await this.client.zRange(key, start, stop, options);
    return results.map(r =>
      typeof r === 'object' ? { ...r, value: JSON.parse(r.value) } : JSON.parse(r)
    );
  }

  // Utility
  async del(...keys) {
    await this.client.del(keys);
  }

  async exists(key) {
    return await this.client.exists(key);
  }

  async ttl(key) {
    return await this.client.ttl(key);
  }

  async keys(pattern) {
    return await this.client.keys(pattern);
  }

  async flushAll() {
    await this.client.flushAll();
  }
}

module.exports = new RedisCache();
```

### Express Middleware Cache

```javascript
// cache-middleware.js
const cache = require('./redis-setup');

function cacheMiddleware(options = {}) {
  const {
    ttl = 300,
    keyPrefix = 'api:',
    keyGenerator = (req) => `${req.method}:${req.originalUrl}`,
    condition = () => true,
    invalidateOn = []
  } = options;

  return async (req, res, next) => {
    // Skip caching for non-GET requests (unless specified)
    if (req.method !== 'GET' && !condition(req)) {
      return next();
    }

    const cacheKey = `${keyPrefix}${keyGenerator(req)}`;

    try {
      // Check cache
      const cached = await cache.get(cacheKey);

      if (cached) {
        res.set('X-Cache', 'HIT');
        return res.json(cached);
      }

      // Store original json method
      const originalJson = res.json.bind(res);

      // Override json to cache response
      res.json = async (data) => {
        res.set('X-Cache', 'MISS');

        // Cache successful responses only
        if (res.statusCode >= 200 && res.statusCode < 300) {
          await cache.set(cacheKey, data, ttl);
        }

        return originalJson(data);
      };

      next();
    } catch (error) {
      console.error('Cache middleware error:', error);
      next(); // Continue without cache on error
    }
  };
}

// Cache invalidation middleware
function invalidateCache(patterns) {
  return async (req, res, next) => {
    const originalJson = res.json.bind(res);

    res.json = async (data) => {
      // Invalidate after successful mutation
      if (res.statusCode >= 200 && res.statusCode < 300) {
        for (const pattern of patterns) {
          const resolvedPattern = typeof pattern === 'function'
            ? pattern(req, data)
            : pattern;
          await cache.invalidatePattern(resolvedPattern);
        }
      }
      return originalJson(data);
    };

    next();
  };
}

// Usage in routes
const express = require('express');
const app = express();

// Cache all GET /api/products requests for 5 minutes
app.get('/api/products',
  cacheMiddleware({ ttl: 300, keyPrefix: 'products:' }),
  async (req, res) => {
    const products = await db.query('SELECT * FROM products');
    res.json(products);
  }
);

// Cache individual product for 10 minutes
app.get('/api/products/:id',
  cacheMiddleware({
    ttl: 600,
    keyGenerator: (req) => `product:${req.params.id}`
  }),
  async (req, res) => {
    const product = await db.query('SELECT * FROM products WHERE id = $1', [req.params.id]);
    res.json(product);
  }
);

// Invalidate cache on product update
app.put('/api/products/:id',
  invalidateCache([
    (req) => `products:*`,
    (req) => `product:${req.params.id}`
  ]),
  async (req, res) => {
    await db.query('UPDATE products SET ... WHERE id = $1', [req.params.id]);
    res.json({ success: true });
  }
);
```

---

## CDN and Edge Caching

### CDN Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     CDN ARCHITECTURE                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  User (Tokyo)      User (London)     User (New York)            │
│       │                 │                  │                     │
│       ▼                 ▼                  ▼                     │
│  ┌─────────┐       ┌─────────┐        ┌─────────┐               │
│  │  Edge   │       │  Edge   │        │  Edge   │               │
│  │  Tokyo  │       │ London  │        │   NYC   │               │
│  └────┬────┘       └────┬────┘        └────┬────┘               │
│       │                 │                  │                     │
│       │    Cache Miss   │    Cache Hit     │                     │
│       │                 │    (instant)     │                     │
│       ▼                 │                  │                     │
│  ┌─────────────────────────────────────────────────┐            │
│  │              ORIGIN SERVER (US-East)            │            │
│  │          (Your actual application)              │            │
│  └─────────────────────────────────────────────────┘            │
│                                                                  │
│  Cache Flow:                                                     │
│  1. User requests asset                                          │
│  2. Edge server checks local cache                              │
│  3. If HIT → Return immediately (~10-50ms)                      │
│  4. If MISS → Fetch from origin, cache, return (~200-500ms)     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Cache-Control Headers

```javascript
// cache-headers.js
const express = require('express');
const app = express();

// Static assets - Long cache
app.use('/static', express.static('public', {
  maxAge: '1y',  // Cache for 1 year
  immutable: true,
  setHeaders: (res, path) => {
    // Add version hash to filename for cache busting
    res.set('Cache-Control', 'public, max-age=31536000, immutable');
  }
}));

// API responses - Short cache with revalidation
app.get('/api/products', (req, res) => {
  res.set({
    'Cache-Control': 'public, max-age=60, stale-while-revalidate=300',
    'Vary': 'Accept-Encoding',
    'ETag': generateETag(products)
  });
  res.json(products);
});

// Private data - No CDN cache
app.get('/api/user/profile', authMiddleware, (req, res) => {
  res.set({
    'Cache-Control': 'private, no-cache, no-store, must-revalidate',
    'Pragma': 'no-cache',
    'Expires': '0'
  });
  res.json(req.user);
});

// Conditional caching based on freshness
app.get('/api/articles/:id', async (req, res) => {
  const article = await getArticle(req.params.id);
  const etag = `"${article.updatedAt.getTime()}"`;

  // Check if client has fresh version
  if (req.headers['if-none-match'] === etag) {
    return res.status(304).end();
  }

  res.set({
    'Cache-Control': 'public, max-age=300',
    'ETag': etag,
    'Last-Modified': article.updatedAt.toUTCString()
  });
  res.json(article);
});

// Cache-Control Directives Reference:
// public       - CDN can cache
// private      - Only browser can cache
// max-age      - Seconds until stale
// s-maxage     - CDN-specific max-age
// no-cache     - Must revalidate before use
// no-store     - Don't cache at all
// must-revalidate - Must check origin when stale
// stale-while-revalidate - Serve stale, update in background
// immutable    - Never changes, don't revalidate
```

### Vercel Edge Caching

```javascript
// Next.js API route with edge caching
// pages/api/products.js
export default async function handler(req, res) {
  const products = await fetchProducts();

  // Cache at Vercel Edge for 60 seconds
  // Stale-while-revalidate for 600 seconds
  res.setHeader(
    'Cache-Control',
    's-maxage=60, stale-while-revalidate=600'
  );

  res.json(products);
}

// Next.js App Router with caching
// app/api/products/route.ts
import { NextResponse } from 'next/server';

export async function GET() {
  const products = await fetchProducts();

  return NextResponse.json(products, {
    headers: {
      'Cache-Control': 's-maxage=60, stale-while-revalidate=600'
    }
  });
}

// Edge runtime for global distribution
export const runtime = 'edge';

// Revalidation in Next.js App Router
// app/products/page.tsx
export const revalidate = 60; // Revalidate every 60 seconds

async function ProductsPage() {
  const products = await fetch('https://api.example.com/products', {
    next: { revalidate: 60 }
  }).then(r => r.json());

  return <ProductList products={products} />;
}
```

### CloudFront Configuration

```yaml
# cloudfront-config.yml
Resources:
  Distribution:
    Type: AWS::CloudFront::Distribution
    Properties:
      DistributionConfig:
        Origins:
          - DomainName: api.myapp.com
            Id: APIOrigin
            CustomOriginConfig:
              HTTPSPort: 443
              OriginProtocolPolicy: https-only

        DefaultCacheBehavior:
          TargetOriginId: APIOrigin
          ViewerProtocolPolicy: redirect-to-https

          # Cache policy
          CachePolicyId: !Ref APICachePolicy

          # Compress responses
          Compress: true

          # Forward headers needed for API
          ForwardedValues:
            QueryString: true
            Headers:
              - Authorization
              - Accept

  APICachePolicy:
    Type: AWS::CloudFront::CachePolicy
    Properties:
      CachePolicyConfig:
        Name: APICachePolicy
        DefaultTTL: 60
        MaxTTL: 3600
        MinTTL: 0
        ParametersInCacheKeyAndForwardedToOrigin:
          EnableAcceptEncodingGzip: true
          HeadersConfig:
            HeaderBehavior: whitelist
            Headers:
              - Accept-Language
          QueryStringsConfig:
            QueryStringBehavior: all
```

---

## Cache Invalidation Strategies

### Time-Based (TTL)

```javascript
// ttl-strategies.js
const CACHE_TTLS = {
  // Very short - Frequently changing data
  realtime: 5,           // 5 seconds

  // Short - User-specific, semi-dynamic
  userSession: 300,      // 5 minutes

  // Medium - API responses, lists
  apiResponse: 3600,     // 1 hour

  // Long - Rarely changing
  staticContent: 86400,  // 1 day

  // Very long - Immutable content
  immutable: 31536000    // 1 year
};

// Adaptive TTL based on data characteristics
function calculateTTL(dataType, lastModified) {
  const age = Date.now() - lastModified.getTime();
  const dayInMs = 86400000;

  // Older data changes less frequently
  if (age > 30 * dayInMs) return CACHE_TTLS.staticContent;
  if (age > 7 * dayInMs) return CACHE_TTLS.apiResponse;
  if (age > dayInMs) return CACHE_TTLS.userSession;
  return CACHE_TTLS.realtime;
}
```

### Event-Based Invalidation

```javascript
// event-invalidation.js
const EventEmitter = require('events');

class CacheInvalidator extends EventEmitter {
  constructor(cache) {
    super();
    this.cache = cache;
    this.setupListeners();
  }

  setupListeners() {
    // User events
    this.on('user:updated', async ({ userId }) => {
      await this.cache.del(`user:${userId}`);
      await this.cache.del(`user:${userId}:profile`);
      await this.cache.del(`user:${userId}:preferences`);
    });

    this.on('user:deleted', async ({ userId }) => {
      const keys = await this.cache.keys(`user:${userId}:*`);
      if (keys.length) await this.cache.del(...keys);
    });

    // Product events
    this.on('product:updated', async ({ productId, categoryId }) => {
      await this.cache.del(`product:${productId}`);
      await this.cache.del(`category:${categoryId}:products`);
      await this.cache.del('products:featured');
      await this.cache.del('products:latest');
    });

    // Bulk invalidation
    this.on('catalog:refreshed', async () => {
      const keys = await this.cache.keys('product:*');
      const categoryKeys = await this.cache.keys('category:*');
      await this.cache.del(...keys, ...categoryKeys);
    });
  }
}

// Usage in service
class ProductService {
  constructor(db, cache, invalidator) {
    this.db = db;
    this.cache = cache;
    this.invalidator = invalidator;
  }

  async updateProduct(productId, data) {
    const product = await this.db.updateProduct(productId, data);

    // Emit event for cache invalidation
    this.invalidator.emit('product:updated', {
      productId,
      categoryId: product.categoryId
    });

    return product;
  }
}
```

### Tag-Based Invalidation

```javascript
// tag-based-cache.js
class TaggedCache {
  constructor(cache) {
    this.cache = cache;
  }

  async set(key, value, tags = [], ttl = 3600) {
    // Store the value
    await this.cache.set(key, value, ttl);

    // Associate key with tags
    for (const tag of tags) {
      await this.cache.sAdd(`tag:${tag}`, key);
    }

    // Store tags for this key
    await this.cache.set(`${key}:tags`, tags, ttl);
  }

  async get(key) {
    return this.cache.get(key);
  }

  async invalidateByTag(tag) {
    // Get all keys with this tag
    const keys = await this.cache.sMembers(`tag:${tag}`);

    if (keys.length === 0) return;

    // Delete all keys
    for (const key of keys) {
      // Get and clean up other tag associations
      const keyTags = await this.cache.get(`${key}:tags`) || [];
      for (const otherTag of keyTags) {
        await this.cache.sRem(`tag:${otherTag}`, key);
      }
      await this.cache.del(key, `${key}:tags`);
    }

    // Clean up the tag set
    await this.cache.del(`tag:${tag}`);

    console.log(`Invalidated ${keys.length} keys with tag: ${tag}`);
  }
}

// Usage
const taggedCache = new TaggedCache(cache);

// Cache product with multiple tags
await taggedCache.set(
  `product:123`,
  productData,
  ['products', 'category:electronics', 'brand:apple']
);

// Invalidate all electronics products
await taggedCache.invalidateByTag('category:electronics');
```

---

## Caching Patterns in Practice

### Request Deduplication

```javascript
// request-dedup.js
class RequestDeduplicator {
  constructor() {
    this.pending = new Map();
  }

  async dedupe(key, fetchFn) {
    // Check if request is already in flight
    if (this.pending.has(key)) {
      console.log(`Deduping request: ${key}`);
      return this.pending.get(key);
    }

    // Create new request promise
    const promise = fetchFn()
      .finally(() => {
        // Clean up after completion
        this.pending.delete(key);
      });

    this.pending.set(key, promise);
    return promise;
  }
}

// Usage - Multiple simultaneous requests get same result
const dedup = new RequestDeduplicator();

async function getUser(userId) {
  return dedup.dedupe(`user:${userId}`, async () => {
    // This only runs once even if called 100 times simultaneously
    return await fetch(`/api/users/${userId}`).then(r => r.json());
  });
}

// Even if called 100 times at once, only 1 API request is made
await Promise.all(
  Array(100).fill().map(() => getUser(123))
);
```

### Multi-Level Cache

```javascript
// multi-level-cache.js
class MultiLevelCache {
  constructor(l1Cache, l2Cache) {
    this.l1 = l1Cache; // In-memory (fast, small)
    this.l2 = l2Cache; // Redis (slower, larger)
  }

  async get(key) {
    // Try L1 first (in-memory)
    let value = this.l1.get(key);
    if (value !== undefined) {
      return { value, source: 'L1' };
    }

    // Try L2 (Redis)
    value = await this.l2.get(key);
    if (value !== null) {
      // Populate L1 for next time
      this.l1.set(key, value);
      return { value, source: 'L2' };
    }

    return { value: null, source: 'MISS' };
  }

  async set(key, value, ttl) {
    // Write to both levels
    this.l1.set(key, value, ttl);
    await this.l2.set(key, value, ttl);
  }

  async del(key) {
    this.l1.del(key);
    await this.l2.del(key);
  }
}

// Simple in-memory L1 cache
class InMemoryCache {
  constructor(maxSize = 1000) {
    this.cache = new Map();
    this.maxSize = maxSize;
  }

  get(key) {
    const item = this.cache.get(key);
    if (!item) return undefined;

    if (Date.now() > item.expires) {
      this.cache.delete(key);
      return undefined;
    }

    return item.value;
  }

  set(key, value, ttlSeconds = 60) {
    // Evict oldest if at capacity
    if (this.cache.size >= this.maxSize) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }

    this.cache.set(key, {
      value,
      expires: Date.now() + (ttlSeconds * 1000)
    });
  }

  del(key) {
    this.cache.delete(key);
  }
}

// Usage
const multiCache = new MultiLevelCache(
  new InMemoryCache(100),
  redisCache
);
```

### Cache Warming

```javascript
// cache-warming.js
class CacheWarmer {
  constructor(cache, db) {
    this.cache = cache;
    this.db = db;
  }

  async warmOnStartup() {
    console.log('Warming cache...');

    await Promise.all([
      this.warmProducts(),
      this.warmCategories(),
      this.warmConfigs()
    ]);

    console.log('Cache warming complete');
  }

  async warmProducts() {
    // Load frequently accessed products
    const topProducts = await this.db.query(`
      SELECT * FROM products
      ORDER BY view_count DESC
      LIMIT 100
    `);

    for (const product of topProducts) {
      await this.cache.set(`product:${product.id}`, product, 3600);
    }

    console.log(`Warmed ${topProducts.length} products`);
  }

  async warmCategories() {
    const categories = await this.db.query('SELECT * FROM categories');

    for (const category of categories) {
      await this.cache.set(`category:${category.id}`, category, 86400);
    }

    // Also cache the full list
    await this.cache.set('categories:all', categories, 86400);
  }

  async warmConfigs() {
    const configs = await this.db.query('SELECT * FROM app_configs');

    const configMap = {};
    for (const config of configs) {
      configMap[config.key] = config.value;
    }

    await this.cache.set('app:config', configMap, 3600);
  }

  // Scheduled re-warming
  scheduleRewarming(intervalMs = 3600000) {
    setInterval(() => {
      this.warmOnStartup().catch(console.error);
    }, intervalMs);
  }
}

// On application start
const warmer = new CacheWarmer(cache, db);
await warmer.warmOnStartup();
warmer.scheduleRewarming(60 * 60 * 1000); // Re-warm hourly
```

---

## Cache Monitoring

```javascript
// cache-metrics.js
class CacheMetrics {
  constructor() {
    this.hits = 0;
    this.misses = 0;
    this.errors = 0;
    this.latencies = [];
  }

  recordHit(latencyMs) {
    this.hits++;
    this.latencies.push(latencyMs);
  }

  recordMiss(latencyMs) {
    this.misses++;
    this.latencies.push(latencyMs);
  }

  recordError() {
    this.errors++;
  }

  getStats() {
    const total = this.hits + this.misses;
    const hitRate = total > 0 ? (this.hits / total * 100).toFixed(2) : 0;

    const sortedLatencies = [...this.latencies].sort((a, b) => a - b);
    const p50 = sortedLatencies[Math.floor(sortedLatencies.length * 0.5)] || 0;
    const p95 = sortedLatencies[Math.floor(sortedLatencies.length * 0.95)] || 0;
    const p99 = sortedLatencies[Math.floor(sortedLatencies.length * 0.99)] || 0;

    return {
      hits: this.hits,
      misses: this.misses,
      hitRate: `${hitRate}%`,
      errors: this.errors,
      latency: {
        p50: `${p50.toFixed(2)}ms`,
        p95: `${p95.toFixed(2)}ms`,
        p99: `${p99.toFixed(2)}ms`
      }
    };
  }

  reset() {
    this.hits = 0;
    this.misses = 0;
    this.errors = 0;
    this.latencies = [];
  }
}

// Metrics endpoint
app.get('/metrics/cache', (req, res) => {
  res.json(cacheMetrics.getStats());
});

// Example output:
// {
//   "hits": 15234,
//   "misses": 1523,
//   "hitRate": "90.92%",
//   "errors": 12,
//   "latency": {
//     "p50": "0.45ms",
//     "p95": "2.34ms",
//     "p99": "5.67ms"
//   }
// }
```

---

## Practice Exercises

### Exercise 1: API Response Cache

Build a caching layer for an API:
- Cache GET requests with configurable TTL
- Automatically invalidate on POST/PUT/DELETE
- Add cache hit/miss headers

### Exercise 2: Session Store

Implement Redis session management:
- Store user sessions in Redis
- Handle session expiration
- Support session refresh on activity

### Exercise 3: Rate Limiter with Cache

Create a rate limiter using Redis:
- Track request counts per IP/user
- Implement sliding window algorithm
- Return remaining requests in headers

---

## Key Takeaways

1. **Cache at multiple levels** - Browser, CDN, application, database
2. **Choose the right strategy** - Cache-aside for reads, write-through for consistency
3. **Set appropriate TTLs** - Balance freshness vs performance
4. **Invalidate smartly** - Event-based is more accurate than time-based alone
5. **Monitor cache health** - Track hit rates and latencies
6. **Warm critical data** - Pre-populate cache on startup

---

## What's Next?

Tomorrow, we'll explore **Message Queues** - learning how to use queues like Redis, RabbitMQ, and SQS to handle background jobs, decouple services, and build resilient distributed systems.
