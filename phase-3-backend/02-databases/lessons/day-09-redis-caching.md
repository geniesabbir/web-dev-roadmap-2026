# Day 9: Redis Caching

## Introduction

Redis is an in-memory data store that excels at caching, session management, real-time analytics, and message queuing. Its speed and versatility make it an essential tool for building high-performance applications. This lesson covers Redis fundamentals, caching patterns, and Node.js integration.

## Learning Objectives

By the end of this lesson, you will:
- Understand Redis data structures and use cases
- Set up Redis locally and in the cloud
- Implement common caching patterns
- Use Redis for sessions and rate limiting
- Build a caching layer for your API
- Handle cache invalidation strategies

---

## What is Redis?

### Key Features

```
┌─────────────────────────────────────────────────────────────────┐
│                         Redis                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  • In-memory storage (microsecond latency)                      │
│  • Rich data structures (strings, lists, sets, hashes, etc.)   │
│  • Persistence options (RDB snapshots, AOF logs)                │
│  • Pub/Sub messaging                                            │
│  • Lua scripting                                                │
│  • Clustering and replication                                   │
│  • TTL (Time-To-Live) for automatic expiration                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Common Use Cases

- **Caching** - Store frequently accessed data
- **Session storage** - User sessions with auto-expiry
- **Rate limiting** - API request throttling
- **Real-time leaderboards** - Sorted sets for rankings
- **Pub/Sub** - Real-time messaging
- **Job queues** - Background task processing
- **Real-time analytics** - Counters and metrics

---

## Setup

### Local Installation

```bash
# macOS with Homebrew
brew install redis
brew services start redis

# Docker (recommended)
docker run -d --name redis \
  -p 6379:6379 \
  redis:7-alpine

# With persistence
docker run -d --name redis \
  -p 6379:6379 \
  -v redis_data:/data \
  redis:7-alpine redis-server --appendonly yes

# Ubuntu
sudo apt-get install redis-server
sudo systemctl start redis
```

### Redis Cloud

- **Redis Cloud** (redis.com) - Managed Redis
- **Upstash** - Serverless Redis
- **AWS ElastiCache** - AWS managed Redis
- **Railway/Render** - Easy deployment

### Redis CLI

```bash
# Connect to local Redis
redis-cli

# Connect to remote Redis
redis-cli -h hostname -p 6379 -a password

# Basic commands
PING                    # Test connection (returns PONG)
INFO                    # Server information
DBSIZE                  # Number of keys
FLUSHDB                 # Clear current database
FLUSHALL                # Clear all databases
QUIT                    # Exit CLI
```

---

## Data Structures

### Strings

The most basic type. Can store text, numbers, or binary data.

```bash
# Set and get
SET user:1:name "Alice"
GET user:1:name          # "Alice"

# Set with expiration
SET session:abc123 "user_data" EX 3600  # Expires in 1 hour
SETEX session:abc123 3600 "user_data"   # Same as above

# Set if not exists
SETNX lock:resource "owner"  # Returns 1 if set, 0 if exists

# Increment/Decrement
SET counter 10
INCR counter             # 11
INCRBY counter 5         # 16
DECR counter             # 15

# Multiple operations
MSET key1 "value1" key2 "value2"
MGET key1 key2           # ["value1", "value2"]
```

### Hashes

Key-value pairs within a key. Perfect for objects.

```bash
# Set hash fields
HSET user:1 name "Alice" email "alice@example.com" age 25

# Get hash fields
HGET user:1 name         # "Alice"
HGETALL user:1           # {"name": "Alice", "email": "...", "age": "25"}
HMGET user:1 name email  # ["Alice", "alice@example.com"]

# Check field existence
HEXISTS user:1 name      # 1 (true)

# Increment hash field
HINCRBY user:1 age 1     # 26

# Delete field
HDEL user:1 email

# Get all keys/values
HKEYS user:1
HVALS user:1
```

### Lists

Ordered collection of strings. Good for queues and recent items.

```bash
# Push to list
LPUSH notifications "msg1"  # Push to left (front)
RPUSH notifications "msg2"  # Push to right (back)

# Pop from list
LPOP notifications          # Remove from left
RPOP notifications          # Remove from right

# Get range
LRANGE notifications 0 -1   # All items
LRANGE notifications 0 9    # First 10 items

# Get length
LLEN notifications

# Blocking pop (for queues)
BLPOP queue:jobs 30         # Wait up to 30 seconds for item
```

### Sets

Unordered collection of unique strings.

```bash
# Add members
SADD tags "javascript" "nodejs" "redis"

# Get all members
SMEMBERS tags

# Check membership
SISMEMBER tags "javascript"  # 1 (true)

# Remove member
SREM tags "redis"

# Set operations
SADD tags2 "python" "javascript"
SUNION tags tags2           # All unique tags from both
SINTER tags tags2           # Common tags ("javascript")
SDIFF tags tags2            # Tags in tags but not tags2

# Random member
SRANDMEMBER tags
SPOP tags                   # Remove and return random
```

### Sorted Sets

Sets with a score for ordering. Perfect for leaderboards.

```bash
# Add members with scores
ZADD leaderboard 100 "alice" 85 "bob" 92 "carol"

# Get range by rank
ZRANGE leaderboard 0 -1              # All, lowest to highest
ZREVRANGE leaderboard 0 2            # Top 3, highest to lowest
ZREVRANGE leaderboard 0 2 WITHSCORES # With scores

# Get range by score
ZRANGEBYSCORE leaderboard 80 100     # Scores between 80-100

# Get rank
ZRANK leaderboard "alice"            # 2 (0-indexed, lowest first)
ZREVRANK leaderboard "alice"         # 0 (highest first)

# Increment score
ZINCRBY leaderboard 10 "bob"         # Bob's new score: 95

# Get score
ZSCORE leaderboard "alice"           # 100

# Remove members
ZREM leaderboard "bob"
```

---

## Expiration (TTL)

```bash
# Set expiration
SET key "value"
EXPIRE key 3600           # Expire in 3600 seconds
EXPIREAT key 1704067200   # Expire at Unix timestamp

# Set with expiration
SET key "value" EX 3600   # Seconds
SET key "value" PX 3600000 # Milliseconds

# Check TTL
TTL key                   # Seconds remaining (-1 = no expiry, -2 = doesn't exist)
PTTL key                  # Milliseconds remaining

# Remove expiration
PERSIST key

# Check if key exists
EXISTS key                # 1 if exists, 0 if not
```

---

## Node.js Integration

### Using ioredis

```bash
npm install ioredis
```

```javascript
// src/lib/redis.js
const Redis = require('ioredis');

const redis = new Redis({
  host: process.env.REDIS_HOST || 'localhost',
  port: process.env.REDIS_PORT || 6379,
  password: process.env.REDIS_PASSWORD,
  maxRetriesPerRequest: 3,
  retryStrategy(times) {
    const delay = Math.min(times * 50, 2000);
    return delay;
  },
});

redis.on('connect', () => {
  console.log('Redis connected');
});

redis.on('error', (err) => {
  console.error('Redis error:', err);
});

module.exports = redis;
```

### Basic Operations

```javascript
const redis = require('./lib/redis');

// Strings
await redis.set('key', 'value');
await redis.set('key', 'value', 'EX', 3600); // With expiration
const value = await redis.get('key');

// Delete
await redis.del('key');
await redis.del('key1', 'key2', 'key3'); // Multiple keys

// Check exists
const exists = await redis.exists('key'); // 1 or 0

// Hashes
await redis.hset('user:1', { name: 'Alice', email: 'alice@example.com' });
const user = await redis.hgetall('user:1');
const name = await redis.hget('user:1', 'name');

// Lists
await redis.lpush('queue', 'item1', 'item2');
const items = await redis.lrange('queue', 0, -1);

// Sets
await redis.sadd('tags', 'js', 'node', 'redis');
const tags = await redis.smembers('tags');

// Sorted sets
await redis.zadd('leaderboard', 100, 'alice', 85, 'bob');
const top10 = await redis.zrevrange('leaderboard', 0, 9, 'WITHSCORES');

// Pipeline (batch operations)
const pipeline = redis.pipeline();
pipeline.set('key1', 'value1');
pipeline.set('key2', 'value2');
pipeline.get('key1');
const results = await pipeline.exec();

// Transaction
const multi = redis.multi();
multi.incr('counter');
multi.incr('counter');
const results = await multi.exec();
```

---

## Caching Patterns

### Cache-Aside (Lazy Loading)

```javascript
// src/services/cache.js
const redis = require('../lib/redis');

class CacheService {
  constructor(defaultTTL = 3600) {
    this.defaultTTL = defaultTTL;
  }

  async get(key) {
    const cached = await redis.get(key);
    return cached ? JSON.parse(cached) : null;
  }

  async set(key, value, ttl = this.defaultTTL) {
    await redis.set(key, JSON.stringify(value), 'EX', ttl);
  }

  async del(key) {
    await redis.del(key);
  }

  async delPattern(pattern) {
    const keys = await redis.keys(pattern);
    if (keys.length > 0) {
      await redis.del(...keys);
    }
  }

  // Cache-aside pattern
  async getOrSet(key, fetchFn, ttl = this.defaultTTL) {
    // Try cache first
    const cached = await this.get(key);
    if (cached !== null) {
      return cached;
    }

    // Fetch from source
    const data = await fetchFn();

    // Store in cache
    if (data !== null && data !== undefined) {
      await this.set(key, data, ttl);
    }

    return data;
  }
}

module.exports = new CacheService();
```

### Using Cache Service

```javascript
// src/services/user.service.js
const cache = require('./cache');
const prisma = require('../lib/prisma');

class UserService {
  async findById(id) {
    const cacheKey = `user:${id}`;

    return cache.getOrSet(
      cacheKey,
      async () => {
        return prisma.user.findUnique({
          where: { id },
          select: { id: true, name: true, email: true, avatar: true },
        });
      },
      3600 // 1 hour TTL
    );
  }

  async findAll(page = 1, pageSize = 10) {
    const cacheKey = `users:page:${page}:size:${pageSize}`;

    return cache.getOrSet(
      cacheKey,
      async () => {
        return prisma.user.findMany({
          skip: (page - 1) * pageSize,
          take: pageSize,
          orderBy: { createdAt: 'desc' },
        });
      },
      300 // 5 minutes TTL
    );
  }

  async update(id, data) {
    const user = await prisma.user.update({
      where: { id },
      data,
    });

    // Invalidate cache
    await cache.del(`user:${id}`);
    await cache.delPattern('users:page:*'); // Invalidate list caches

    return user;
  }

  async delete(id) {
    await prisma.user.delete({ where: { id } });
    await cache.del(`user:${id}`);
    await cache.delPattern('users:page:*');
  }
}
```

### Write-Through Cache

```javascript
// Update cache when writing to database
async function updateUser(id, data) {
  // Update database
  const user = await prisma.user.update({
    where: { id },
    data,
  });

  // Update cache immediately
  await cache.set(`user:${id}`, user, 3600);

  return user;
}
```

### Cache Decorator

```javascript
// Decorator pattern for caching
function cached(keyGenerator, ttl = 3600) {
  return function (target, propertyKey, descriptor) {
    const originalMethod = descriptor.value;

    descriptor.value = async function (...args) {
      const key = keyGenerator(...args);
      const cached = await cache.get(key);

      if (cached !== null) {
        return cached;
      }

      const result = await originalMethod.apply(this, args);

      if (result !== null) {
        await cache.set(key, result, ttl);
      }

      return result;
    };

    return descriptor;
  };
}

// Usage
class PostService {
  @cached((id) => `post:${id}`, 3600)
  async findById(id) {
    return prisma.post.findUnique({ where: { id } });
  }
}
```

---

## Session Management

```javascript
// src/middleware/session.js
const redis = require('../lib/redis');
const { v4: uuidv4 } = require('uuid');

const SESSION_TTL = 24 * 60 * 60; // 24 hours

class SessionManager {
  async create(userId, data = {}) {
    const sessionId = uuidv4();
    const session = {
      userId,
      ...data,
      createdAt: Date.now(),
    };

    await redis.hset(`session:${sessionId}`, session);
    await redis.expire(`session:${sessionId}`, SESSION_TTL);

    return sessionId;
  }

  async get(sessionId) {
    const session = await redis.hgetall(`session:${sessionId}`);
    return Object.keys(session).length > 0 ? session : null;
  }

  async refresh(sessionId) {
    const exists = await redis.exists(`session:${sessionId}`);
    if (exists) {
      await redis.expire(`session:${sessionId}`, SESSION_TTL);
      return true;
    }
    return false;
  }

  async destroy(sessionId) {
    await redis.del(`session:${sessionId}`);
  }

  async destroyUserSessions(userId) {
    const keys = await redis.keys('session:*');
    for (const key of keys) {
      const session = await redis.hgetall(key);
      if (session.userId === String(userId)) {
        await redis.del(key);
      }
    }
  }
}

// Middleware
const sessionMiddleware = async (req, res, next) => {
  const sessionId = req.cookies?.sessionId || req.headers['x-session-id'];

  if (sessionId) {
    const session = await sessionManager.get(sessionId);
    if (session) {
      req.session = session;
      await sessionManager.refresh(sessionId);
    }
  }

  next();
};
```

---

## Rate Limiting

### Fixed Window

```javascript
// src/middleware/rateLimit.js
const redis = require('../lib/redis');

function rateLimiter(options = {}) {
  const {
    windowMs = 60000,  // 1 minute
    max = 100,         // Max requests per window
    keyGenerator = (req) => req.ip,
    message = 'Too many requests',
  } = options;

  return async (req, res, next) => {
    const key = `ratelimit:${keyGenerator(req)}`;
    const windowSeconds = Math.ceil(windowMs / 1000);

    const current = await redis.incr(key);

    if (current === 1) {
      await redis.expire(key, windowSeconds);
    }

    const ttl = await redis.ttl(key);

    res.set({
      'X-RateLimit-Limit': max,
      'X-RateLimit-Remaining': Math.max(0, max - current),
      'X-RateLimit-Reset': Math.ceil(Date.now() / 1000) + ttl,
    });

    if (current > max) {
      return res.status(429).json({
        error: message,
        retryAfter: ttl,
      });
    }

    next();
  };
}

// Usage
app.use('/api', rateLimiter({ max: 100, windowMs: 60000 }));
app.use('/api/auth', rateLimiter({ max: 5, windowMs: 60000 }));
```

### Sliding Window (More Accurate)

```javascript
async function slidingWindowRateLimit(key, limit, windowSeconds) {
  const now = Date.now();
  const windowStart = now - (windowSeconds * 1000);

  // Remove old entries
  await redis.zremrangebyscore(key, 0, windowStart);

  // Count current entries
  const count = await redis.zcard(key);

  if (count >= limit) {
    return { allowed: false, count };
  }

  // Add current request
  await redis.zadd(key, now, `${now}-${Math.random()}`);
  await redis.expire(key, windowSeconds);

  return { allowed: true, count: count + 1 };
}
```

---

## Pub/Sub

### Publisher

```javascript
const redis = require('./lib/redis');

// Publish events
async function publishEvent(channel, data) {
  await redis.publish(channel, JSON.stringify(data));
}

// Usage
publishEvent('notifications', {
  type: 'NEW_MESSAGE',
  userId: 123,
  message: 'Hello!',
});
```

### Subscriber

```javascript
const Redis = require('ioredis');
const subscriber = new Redis(); // Separate connection for subscribing

subscriber.subscribe('notifications', (err, count) => {
  if (err) {
    console.error('Subscribe error:', err);
    return;
  }
  console.log(`Subscribed to ${count} channel(s)`);
});

subscriber.on('message', (channel, message) => {
  const data = JSON.parse(message);
  console.log(`Received on ${channel}:`, data);

  // Handle different event types
  switch (data.type) {
    case 'NEW_MESSAGE':
      // Send push notification
      break;
    case 'USER_UPDATED':
      // Update cache
      break;
  }
});
```

---

## Cache Invalidation Strategies

### Time-Based (TTL)

```javascript
// Simple: Set appropriate TTL
await cache.set('data', value, 300); // 5 minutes
```

### Event-Based

```javascript
// Invalidate on data changes
async function updatePost(id, data) {
  const post = await prisma.post.update({ where: { id }, data });

  // Invalidate related caches
  await Promise.all([
    cache.del(`post:${id}`),
    cache.del(`post:slug:${post.slug}`),
    cache.delPattern(`posts:author:${post.authorId}:*`),
    cache.delPattern('posts:list:*'),
  ]);

  return post;
}
```

### Tag-Based Invalidation

```javascript
// Store tags with cached items
async function cacheWithTags(key, value, tags, ttl) {
  await redis.set(key, JSON.stringify(value), 'EX', ttl);

  for (const tag of tags) {
    await redis.sadd(`tag:${tag}`, key);
  }
}

async function invalidateTag(tag) {
  const keys = await redis.smembers(`tag:${tag}`);
  if (keys.length > 0) {
    await redis.del(...keys);
    await redis.del(`tag:${tag}`);
  }
}

// Usage
await cacheWithTags('post:1', post, ['posts', `user:${post.authorId}`], 3600);
await invalidateTag(`user:${userId}`); // Invalidate all user's cached data
```

---

## Complete Caching Middleware

```javascript
// src/middleware/cache.js
const redis = require('../lib/redis');
const crypto = require('crypto');

function cacheMiddleware(options = {}) {
  const {
    ttl = 300,
    keyGenerator = (req) => {
      const hash = crypto
        .createHash('md5')
        .update(req.originalUrl + JSON.stringify(req.query))
        .digest('hex');
      return `cache:${req.method}:${hash}`;
    },
    condition = (req) => req.method === 'GET',
  } = options;

  return async (req, res, next) => {
    if (!condition(req)) {
      return next();
    }

    const key = keyGenerator(req);

    try {
      const cached = await redis.get(key);

      if (cached) {
        const data = JSON.parse(cached);
        res.set('X-Cache', 'HIT');
        return res.json(data);
      }

      // Store original json method
      const originalJson = res.json.bind(res);

      // Override json method to cache response
      res.json = async (body) => {
        res.set('X-Cache', 'MISS');

        // Cache successful responses only
        if (res.statusCode >= 200 && res.statusCode < 300) {
          await redis.set(key, JSON.stringify(body), 'EX', ttl);
        }

        return originalJson(body);
      };

      next();
    } catch (error) {
      console.error('Cache error:', error);
      next();
    }
  };
}

// Usage
app.get('/api/posts', cacheMiddleware({ ttl: 60 }), postsController.list);
app.get('/api/posts/:id', cacheMiddleware({ ttl: 300 }), postsController.show);
```

---

## Key Takeaways

1. **Redis is fast** - In-memory, microsecond latency
2. **Use the right data structure** - Strings, hashes, lists, sets, sorted sets
3. **Set TTL on everything** - Prevent stale data and memory issues
4. **Cache-aside is most common** - Check cache, fetch if miss, store result
5. **Invalidate proactively** - Clear cache when data changes
6. **Use separate connections** - For pub/sub, use dedicated subscribers
7. **Monitor memory** - Redis data must fit in RAM

---

## What's Next?

Tomorrow we'll build a **Complete Database Project** - bringing together PostgreSQL, Prisma, and Redis in a production-ready application!
