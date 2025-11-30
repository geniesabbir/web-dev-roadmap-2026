# Day 4: Backend Performance - Fast APIs and Servers

## Introduction

A fast frontend means nothing if your API takes seconds to respond. Backend performance encompasses database optimization, efficient data handling, caching strategies, and Node.js runtime optimization. Today, you'll learn how to identify bottlenecks and implement solutions that dramatically improve API response times.

## Learning Objectives

By the end of this lesson, you will be able to:
- Profile and identify backend bottlenecks
- Optimize database queries and connections
- Implement effective caching strategies
- Handle concurrent requests efficiently
- Optimize Node.js runtime performance

---

## Backend Performance Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                  API REQUEST LIFECYCLE                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Client Request                                                  │
│       │                                                          │
│       ▼                                                          │
│  ┌─────────────┐  Network latency affects all requests          │
│  │   Network   │  Target: < 50ms for same region               │
│  └──────┬──────┘                                                │
│         ▼                                                        │
│  ┌─────────────┐  TLS handshake, HTTP parsing                   │
│  │   Server    │  Target: < 10ms                                │
│  └──────┬──────┘                                                │
│         ▼                                                        │
│  ┌─────────────┐  Auth, validation, parsing                     │
│  │ Middleware  │  Target: < 5ms                                 │
│  └──────┬──────┘                                                │
│         ▼                                                        │
│  ┌─────────────┐  Business logic                                │
│  │  Handler    │  Target: < 10ms (excluding I/O)                │
│  └──────┬──────┘                                                │
│         ▼                                                        │
│  ┌─────────────┐  Queries, external APIs                        │
│  │    I/O      │  Target: < 100ms (most impact!)                │
│  └──────┬──────┘                                                │
│         ▼                                                        │
│  ┌─────────────┐  JSON stringify, compression                   │
│  │  Response   │  Target: < 5ms                                 │
│  └─────────────┘                                                │
│                                                                  │
│  Total Target: < 200ms for most APIs                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Profiling and Measuring

### Request Timing Middleware

```javascript
// middleware/timing.js
function timingMiddleware(req, res, next) {
  const start = process.hrtime.bigint();

  // Track response
  res.on('finish', () => {
    const end = process.hrtime.bigint();
    const durationMs = Number(end - start) / 1e6;

    // Log slow requests
    if (durationMs > 200) {
      console.warn(`Slow request: ${req.method} ${req.path} - ${durationMs.toFixed(2)}ms`);
    }

    // Add timing header
    res.setHeader('Server-Timing', `total;dur=${durationMs.toFixed(2)}`);
  });

  next();
}

// Detailed timing with phases
function detailedTimingMiddleware(req, res, next) {
  const timings = {
    start: process.hrtime.bigint(),
    dbStart: null,
    dbEnd: null,
    cacheStart: null,
    cacheEnd: null
  };

  // Expose timing functions
  req.timing = {
    startDb: () => { timings.dbStart = process.hrtime.bigint(); },
    endDb: () => { timings.dbEnd = process.hrtime.bigint(); },
    startCache: () => { timings.cacheStart = process.hrtime.bigint(); },
    endCache: () => { timings.cacheEnd = process.hrtime.bigint(); }
  };

  res.on('finish', () => {
    const end = process.hrtime.bigint();
    const total = Number(end - timings.start) / 1e6;

    const phases = [];

    if (timings.dbStart && timings.dbEnd) {
      const dbTime = Number(timings.dbEnd - timings.dbStart) / 1e6;
      phases.push(`db;dur=${dbTime.toFixed(2)}`);
    }

    if (timings.cacheStart && timings.cacheEnd) {
      const cacheTime = Number(timings.cacheEnd - timings.cacheStart) / 1e6;
      phases.push(`cache;dur=${cacheTime.toFixed(2)}`);
    }

    phases.push(`total;dur=${total.toFixed(2)}`);

    res.setHeader('Server-Timing', phases.join(', '));
  });

  next();
}

// Usage in route
app.get('/api/products', detailedTimingMiddleware, async (req, res) => {
  req.timing.startCache();
  const cached = await cache.get('products');
  req.timing.endCache();

  if (cached) {
    return res.json(cached);
  }

  req.timing.startDb();
  const products = await db.products.findMany();
  req.timing.endDb();

  await cache.set('products', products, 300);
  res.json(products);
});
```

### Node.js Profiling

```javascript
// CPU Profiling with --inspect
// node --inspect server.js
// Open chrome://inspect in Chrome

// Programmatic profiling
const v8 = require('v8');
const fs = require('fs');

// Memory snapshot
function takeHeapSnapshot() {
  const snapshotPath = `heap-${Date.now()}.heapsnapshot`;
  const snapshotStream = v8.writeHeapSnapshot(snapshotPath);
  console.log(`Heap snapshot written to: ${snapshotPath}`);
}

// Memory usage logging
function logMemoryUsage() {
  const usage = process.memoryUsage();
  console.log({
    rss: `${(usage.rss / 1024 / 1024).toFixed(2)} MB`,
    heapTotal: `${(usage.heapTotal / 1024 / 1024).toFixed(2)} MB`,
    heapUsed: `${(usage.heapUsed / 1024 / 1024).toFixed(2)} MB`,
    external: `${(usage.external / 1024 / 1024).toFixed(2)} MB`
  });
}

// Log memory every minute
setInterval(logMemoryUsage, 60000);

// Using clinic.js for profiling
// npm install -g clinic
// clinic doctor -- node server.js
// clinic flame -- node server.js
// clinic bubbleprof -- node server.js
```

---

## Database Optimization

### Query Optimization

```javascript
// Prisma query optimization
// Bad: N+1 query problem
async function getUsersWithPosts() {
  const users = await prisma.user.findMany();

  // N additional queries!
  for (const user of users) {
    user.posts = await prisma.post.findMany({
      where: { authorId: user.id }
    });
  }

  return users;
}

// Good: Include relation (single query with JOIN)
async function getUsersWithPosts() {
  return prisma.user.findMany({
    include: {
      posts: {
        take: 10,
        orderBy: { createdAt: 'desc' }
      }
    }
  });
}

// Good: Select only needed fields
async function getUsersWithPosts() {
  return prisma.user.findMany({
    select: {
      id: true,
      name: true,
      email: true,
      posts: {
        select: {
          id: true,
          title: true
        },
        take: 10
      }
    }
  });
}

// Pagination with cursor (more efficient than offset)
async function getPaginatedPosts(cursor, limit = 20) {
  return prisma.post.findMany({
    take: limit,
    skip: cursor ? 1 : 0,
    cursor: cursor ? { id: cursor } : undefined,
    orderBy: { createdAt: 'desc' }
  });
}

// Batch operations
async function updateManyPosts(updates) {
  // Bad: Individual updates
  for (const update of updates) {
    await prisma.post.update({
      where: { id: update.id },
      data: update.data
    });
  }

  // Good: Transaction with batch
  await prisma.$transaction(
    updates.map(update =>
      prisma.post.update({
        where: { id: update.id },
        data: update.data
      })
    )
  );
}
```

### Database Indexes

```sql
-- Identify slow queries
EXPLAIN ANALYZE SELECT * FROM posts WHERE author_id = 1 ORDER BY created_at DESC;

-- Add indexes for common queries
CREATE INDEX idx_posts_author_id ON posts(author_id);
CREATE INDEX idx_posts_created_at ON posts(created_at DESC);

-- Composite index for multiple columns
CREATE INDEX idx_posts_author_created ON posts(author_id, created_at DESC);

-- Partial index for common filters
CREATE INDEX idx_posts_published ON posts(created_at DESC)
WHERE status = 'published';

-- Prisma schema with indexes
model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String
  authorId  Int
  status    String   @default("draft")
  createdAt DateTime @default(now())

  author    User     @relation(fields: [authorId], references: [id])

  @@index([authorId])
  @@index([createdAt(sort: Desc)])
  @@index([authorId, createdAt(sort: Desc)])
  @@index([status, createdAt(sort: Desc)])
}
```

### Connection Pooling

```javascript
// Prisma with connection pool
// prisma/schema.prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
  // Connection pool settings in URL:
  // postgresql://user:pass@host/db?connection_limit=10&pool_timeout=10
}

// pg (node-postgres) connection pool
const { Pool } = require('pg');

const pool = new Pool({
  host: process.env.DB_HOST,
  port: 5432,
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,

  // Pool configuration
  max: 20,                  // Maximum connections
  min: 5,                   // Minimum connections
  idleTimeoutMillis: 30000, // Close idle connections after 30s
  connectionTimeoutMillis: 2000, // Error if can't connect in 2s

  // Connection validation
  query_timeout: 10000,     // Query timeout 10s
});

// Monitor pool health
pool.on('connect', () => console.log('Pool: new connection'));
pool.on('remove', () => console.log('Pool: connection removed'));
pool.on('error', (err) => console.error('Pool error:', err));

// Query with connection reuse
async function query(text, params) {
  const start = Date.now();
  const result = await pool.query(text, params);
  const duration = Date.now() - start;

  if (duration > 100) {
    console.warn(`Slow query (${duration}ms):`, text);
  }

  return result;
}

// Transaction helper
async function transaction(callback) {
  const client = await pool.connect();
  try {
    await client.query('BEGIN');
    const result = await callback(client);
    await client.query('COMMIT');
    return result;
  } catch (error) {
    await client.query('ROLLBACK');
    throw error;
  } finally {
    client.release();
  }
}
```

---

## API Response Optimization

### Response Compression

```javascript
const express = require('express');
const compression = require('compression');

const app = express();

// Enable compression for all responses
app.use(compression({
  level: 6,           // Compression level (1-9)
  threshold: 1024,    // Only compress responses > 1KB
  filter: (req, res) => {
    // Don't compress if client doesn't accept it
    if (req.headers['x-no-compression']) {
      return false;
    }
    return compression.filter(req, res);
  }
}));

// Brotli compression (better than gzip)
// nginx.conf
// brotli on;
// brotli_types text/plain text/css application/json application/javascript;
// brotli_comp_level 6;
```

### Pagination and Limiting

```javascript
// Efficient pagination
app.get('/api/posts', async (req, res) => {
  const {
    cursor,
    limit = 20,
    fields
  } = req.query;

  // Validate and cap limit
  const safeLimit = Math.min(parseInt(limit) || 20, 100);

  // Parse field selection
  const select = fields
    ? fields.split(',').reduce((acc, f) => ({ ...acc, [f]: true }), {})
    : undefined;

  const posts = await prisma.post.findMany({
    take: safeLimit + 1, // Fetch one extra to check if more exist
    skip: cursor ? 1 : 0,
    cursor: cursor ? { id: cursor } : undefined,
    orderBy: { createdAt: 'desc' },
    select
  });

  const hasMore = posts.length > safeLimit;
  const data = hasMore ? posts.slice(0, -1) : posts;
  const nextCursor = hasMore ? data[data.length - 1].id : null;

  res.json({
    data,
    pagination: {
      hasMore,
      nextCursor
    }
  });
});

// Field filtering
app.get('/api/users/:id', async (req, res) => {
  const { fields } = req.query;

  // Only select requested fields
  const select = fields
    ? fields.split(',').reduce((acc, f) => ({ ...acc, [f]: true }), { id: true })
    : undefined;

  const user = await prisma.user.findUnique({
    where: { id: req.params.id },
    select
  });

  res.json(user);
});
```

### Streaming Large Responses

```javascript
const { Transform } = require('stream');

// Stream JSON array
app.get('/api/export/users', async (req, res) => {
  res.setHeader('Content-Type', 'application/json');
  res.write('[');

  let first = true;
  const cursor = prisma.user.findMany({
    cursor: true // Enable cursor-based iteration
  });

  for await (const user of cursor) {
    if (!first) res.write(',');
    res.write(JSON.stringify(user));
    first = false;
  }

  res.write(']');
  res.end();
});

// Stream CSV export
const { stringify } = require('csv-stringify');

app.get('/api/export/users.csv', async (req, res) => {
  res.setHeader('Content-Type', 'text/csv');
  res.setHeader('Content-Disposition', 'attachment; filename=users.csv');

  const stringifier = stringify({
    header: true,
    columns: ['id', 'name', 'email', 'createdAt']
  });

  stringifier.pipe(res);

  const batchSize = 1000;
  let cursor = undefined;

  while (true) {
    const users = await prisma.user.findMany({
      take: batchSize,
      skip: cursor ? 1 : 0,
      cursor: cursor ? { id: cursor } : undefined,
      orderBy: { id: 'asc' }
    });

    if (users.length === 0) break;

    for (const user of users) {
      stringifier.write(user);
    }

    cursor = users[users.length - 1].id;
  }

  stringifier.end();
});
```

---

## Caching Strategies

### Multi-Layer Caching

```javascript
// cache/multi-layer.js
class MultiLayerCache {
  constructor(memoryCache, redisCache) {
    this.l1 = memoryCache;  // In-memory (fastest)
    this.l2 = redisCache;   // Redis (shared across instances)
  }

  async get(key) {
    // Try L1 first
    let value = this.l1.get(key);
    if (value !== undefined) {
      return { value, source: 'memory' };
    }

    // Try L2
    value = await this.l2.get(key);
    if (value !== null) {
      // Populate L1 for next request
      this.l1.set(key, value);
      return { value, source: 'redis' };
    }

    return { value: null, source: 'miss' };
  }

  async set(key, value, ttl) {
    this.l1.set(key, value, ttl);
    await this.l2.set(key, value, ttl);
  }

  async del(key) {
    this.l1.del(key);
    await this.l2.del(key);
  }
}

// Usage
const NodeCache = require('node-cache');
const Redis = require('ioredis');

const memoryCache = new NodeCache({ stdTTL: 60, checkperiod: 120 });
const redisCache = new Redis(process.env.REDIS_URL);

const cache = new MultiLayerCache(memoryCache, redisCache);

// In route handler
app.get('/api/products/:id', async (req, res) => {
  const key = `product:${req.params.id}`;

  const { value: cached, source } = await cache.get(key);
  if (cached) {
    res.setHeader('X-Cache', source);
    return res.json(cached);
  }

  const product = await prisma.product.findUnique({
    where: { id: req.params.id }
  });

  if (product) {
    await cache.set(key, product, 300);
  }

  res.setHeader('X-Cache', 'miss');
  res.json(product);
});
```

### Request Deduplication

```javascript
// Prevent duplicate concurrent requests
class RequestDeduplicator {
  constructor() {
    this.pending = new Map();
  }

  async dedupe(key, fetchFn) {
    // If request already in flight, return same promise
    if (this.pending.has(key)) {
      return this.pending.get(key);
    }

    // Create new request
    const promise = fetchFn()
      .finally(() => {
        this.pending.delete(key);
      });

    this.pending.set(key, promise);
    return promise;
  }
}

const deduplicator = new RequestDeduplicator();

// Even if 100 requests come in simultaneously for same product,
// only one database query is made
app.get('/api/products/:id', async (req, res) => {
  const product = await deduplicator.dedupe(
    `product:${req.params.id}`,
    () => prisma.product.findUnique({
      where: { id: req.params.id }
    })
  );

  res.json(product);
});
```

### Cache Warming

```javascript
// Warm cache on startup and periodically
class CacheWarmer {
  constructor(cache, prisma) {
    this.cache = cache;
    this.prisma = prisma;
  }

  async warmAll() {
    console.log('Warming cache...');

    await Promise.all([
      this.warmProducts(),
      this.warmCategories(),
      this.warmPopularItems()
    ]);

    console.log('Cache warming complete');
  }

  async warmProducts() {
    const products = await this.prisma.product.findMany({
      take: 100,
      orderBy: { viewCount: 'desc' }
    });

    await Promise.all(
      products.map(p =>
        this.cache.set(`product:${p.id}`, p, 3600)
      )
    );

    console.log(`Warmed ${products.length} products`);
  }

  async warmCategories() {
    const categories = await this.prisma.category.findMany({
      include: { _count: { select: { products: true } } }
    });

    await this.cache.set('categories:all', categories, 86400);
    console.log(`Warmed ${categories.length} categories`);
  }

  async warmPopularItems() {
    const popular = await this.prisma.product.findMany({
      take: 20,
      orderBy: { salesCount: 'desc' }
    });

    await this.cache.set('products:popular', popular, 1800);
    console.log('Warmed popular products');
  }

  scheduleRewarming(intervalMs = 3600000) {
    setInterval(() => this.warmAll(), intervalMs);
  }
}

// On app startup
const warmer = new CacheWarmer(cache, prisma);
await warmer.warmAll();
warmer.scheduleRewarming(60 * 60 * 1000); // Rewarm hourly
```

---

## Node.js Optimization

### Clustering

```javascript
// cluster.js
const cluster = require('cluster');
const os = require('os');

if (cluster.isPrimary) {
  const numCPUs = os.cpus().length;
  console.log(`Primary ${process.pid} starting ${numCPUs} workers`);

  // Fork workers
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  // Handle worker crashes
  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died. Restarting...`);
    cluster.fork();
  });

  // Graceful shutdown
  process.on('SIGTERM', () => {
    console.log('SIGTERM received, shutting down gracefully');
    for (const id in cluster.workers) {
      cluster.workers[id].send('shutdown');
    }
  });
} else {
  // Worker process
  require('./server');

  process.on('message', (msg) => {
    if (msg === 'shutdown') {
      console.log(`Worker ${process.pid} shutting down`);
      process.exit(0);
    }
  });
}

// Or use PM2
// pm2 start server.js -i max
```

### Worker Threads for CPU-Intensive Tasks

```javascript
// workers/heavy-computation.js
const { parentPort, workerData } = require('worker_threads');

function heavyComputation(data) {
  // CPU-intensive work here
  let result = 0;
  for (let i = 0; i < data.iterations; i++) {
    result += Math.sqrt(i);
  }
  return result;
}

const result = heavyComputation(workerData);
parentPort.postMessage(result);

// main.js
const { Worker } = require('worker_threads');

function runWorker(data) {
  return new Promise((resolve, reject) => {
    const worker = new Worker('./workers/heavy-computation.js', {
      workerData: data
    });

    worker.on('message', resolve);
    worker.on('error', reject);
    worker.on('exit', (code) => {
      if (code !== 0) {
        reject(new Error(`Worker stopped with exit code ${code}`));
      }
    });
  });
}

// Usage in route
app.post('/api/compute', async (req, res) => {
  const result = await runWorker({
    iterations: req.body.iterations
  });

  res.json({ result });
});

// Worker pool for efficiency
const { StaticPool } = require('node-worker-threads-pool');

const pool = new StaticPool({
  size: 4,
  task: './workers/heavy-computation.js'
});

app.post('/api/compute', async (req, res) => {
  const result = await pool.exec(req.body);
  res.json({ result });
});
```

### Memory Management

```javascript
// Avoid memory leaks
// Bad: Growing array
const cache = [];
app.get('/api/data', (req, res) => {
  cache.push(fetchData()); // Memory leak!
  res.json(cache[cache.length - 1]);
});

// Good: Use LRU cache with size limit
const LRU = require('lru-cache');
const cache = new LRU({
  max: 500,              // Maximum items
  maxSize: 50 * 1024 * 1024, // 50MB max size
  sizeCalculation: (value) => JSON.stringify(value).length,
  ttl: 1000 * 60 * 5     // 5 minutes
});

// Stream large data instead of loading into memory
app.get('/api/export', async (req, res) => {
  // Bad: Load all into memory
  // const data = await db.query('SELECT * FROM large_table');
  // res.json(data);

  // Good: Stream
  const cursor = db.query('SELECT * FROM large_table').cursor();
  res.setHeader('Content-Type', 'application/json');
  res.write('[');

  let first = true;
  for await (const row of cursor) {
    if (!first) res.write(',');
    res.write(JSON.stringify(row));
    first = false;
  }

  res.write(']');
  res.end();
});

// Garbage collection monitoring
const v8 = require('v8');

setInterval(() => {
  const heapStats = v8.getHeapStatistics();
  const usedPercent = (heapStats.used_heap_size / heapStats.heap_size_limit) * 100;

  if (usedPercent > 90) {
    console.warn(`High memory usage: ${usedPercent.toFixed(1)}%`);
    // Could trigger graceful restart
  }
}, 30000);
```

---

## Rate Limiting and Throttling

```javascript
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');

// Basic rate limiter
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100,                  // 100 requests per window
  message: { error: 'Too many requests, please try again later' },
  standardHeaders: true,
  legacyHeaders: false
});

// Redis-backed rate limiter (for clustered apps)
const redisLimiter = rateLimit({
  store: new RedisStore({
    client: redisClient,
    prefix: 'rl:'
  }),
  windowMs: 15 * 60 * 1000,
  max: 100
});

// Tiered rate limits
const tierLimits = {
  free: { windowMs: 60000, max: 10 },
  basic: { windowMs: 60000, max: 100 },
  pro: { windowMs: 60000, max: 1000 }
};

function tieredRateLimiter(req, res, next) {
  const userTier = req.user?.tier || 'free';
  const limits = tierLimits[userTier];

  return rateLimit({
    windowMs: limits.windowMs,
    max: limits.max,
    keyGenerator: (req) => req.user?.id || req.ip
  })(req, res, next);
}

app.use('/api', tieredRateLimiter);
```

---

## Practice Exercises

### Exercise 1: Query Optimization

Optimize a slow API endpoint:
- Add request timing
- Identify N+1 queries
- Add appropriate indexes
- Implement caching

### Exercise 2: Load Testing

Use k6 or autocannon to:
- Test API under load
- Identify bottlenecks
- Measure throughput
- Find breaking point

### Exercise 3: Caching Layer

Implement multi-layer caching:
- In-memory for hot data
- Redis for shared cache
- Cache invalidation on updates
- Cache warming on startup

---

## Key Takeaways

1. **Measure first** - Profile before optimizing
2. **Fix queries** - N+1 problems are the #1 issue
3. **Index strategically** - Cover your common queries
4. **Cache aggressively** - But invalidate correctly
5. **Stream large data** - Don't load everything into memory
6. **Use workers** - Offload CPU work from event loop

---

## What's Next?

Tomorrow, we'll explore **Performance Monitoring** - setting up comprehensive monitoring to track performance in production and catch regressions before users notice.
