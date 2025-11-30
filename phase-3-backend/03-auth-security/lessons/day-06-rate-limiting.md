# Day 6: Rate Limiting - Protecting Your API from Abuse

## Introduction

Rate limiting controls how many requests a client can make to your API within a given time period. It protects your application from brute force attacks, denial of service, and resource exhaustion. Today, you'll learn how to implement effective rate limiting strategies using Redis and express-rate-limit.

## Learning Objectives

By the end of this lesson, you will be able to:
- Understand different rate limiting algorithms
- Implement rate limiting with express-rate-limit
- Use Redis for distributed rate limiting
- Create tiered rate limits for different user types
- Handle rate limit responses properly

---

## Why Rate Limiting Matters

### Threats Mitigated by Rate Limiting

| Threat | Description | Rate Limit Strategy |
|--------|-------------|---------------------|
| Brute Force | Password guessing attacks | Strict limits on auth endpoints |
| DDoS | Overwhelming server with requests | Global request limits |
| Scraping | Harvesting data from your API | Per-IP limits |
| API Abuse | Excessive usage by single user | Per-user limits |
| Resource Exhaustion | Depleting server resources | Endpoint-specific limits |

---

## Rate Limiting Algorithms

### 1. Fixed Window

```
Time:    |----Window 1----|----Window 2----|
Requests: xxxxxxxx         xxxxxxx

Limit: 10 requests per minute
Window 1 (00:00-00:59): 8 requests ✓
Window 2 (01:00-01:59): 7 requests ✓

Problem: 8 requests at 00:59 + 7 at 01:00 = 15 in 1 minute span
```

### 2. Sliding Window

```
Current time: 01:30
Looking back 60 seconds: 00:30 - 01:30
Count requests in this sliding window

More accurate but more memory intensive
```

### 3. Token Bucket

```
Bucket holds N tokens
Refills at rate of X tokens/second
Each request consumes 1 token
Request denied if bucket empty

Allows bursts while maintaining average rate
```

### 4. Leaky Bucket

```
Requests queue like water in bucket
Processed at constant rate
Bucket overflows = requests dropped

Smooths traffic spikes
```

---

## Basic Rate Limiting with express-rate-limit

### Installation

```bash
npm install express-rate-limit
```

### Simple Configuration

```javascript
// middleware/rateLimiter.js
const rateLimit = require('express-rate-limit');

// Global rate limiter
const globalLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100,                   // 100 requests per window
  message: {
    error: 'Too many requests',
    message: 'You have exceeded the rate limit. Please try again later.',
    retryAfter: 15 * 60       // seconds
  },
  standardHeaders: true,      // Return rate limit info in headers
  legacyHeaders: false        // Disable X-RateLimit-* headers
});

// Apply to all routes
app.use(globalLimiter);
```

### Response Headers

```http
RateLimit-Limit: 100
RateLimit-Remaining: 95
RateLimit-Reset: 1640000000
Retry-After: 900
```

---

## Endpoint-Specific Rate Limits

```javascript
// middleware/rateLimiters.js
const rateLimit = require('express-rate-limit');

// Strict limiter for authentication endpoints
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 5,                     // 5 attempts
  message: {
    error: 'Too many login attempts',
    message: 'Account temporarily locked. Try again in 15 minutes.',
    code: 'AUTH_RATE_LIMITED'
  },
  skipSuccessfulRequests: true  // Don't count successful logins
});

// Password reset limiter
const passwordResetLimiter = rateLimit({
  windowMs: 60 * 60 * 1000,  // 1 hour
  max: 3,                     // 3 reset requests
  message: {
    error: 'Too many password reset requests',
    message: 'Please wait before requesting another reset.'
  }
});

// API endpoints limiter
const apiLimiter = rateLimit({
  windowMs: 60 * 1000,       // 1 minute
  max: 60,                    // 60 requests per minute
  message: {
    error: 'API rate limit exceeded',
    message: 'Slow down! You are making too many requests.'
  }
});

// File upload limiter
const uploadLimiter = rateLimit({
  windowMs: 60 * 60 * 1000,  // 1 hour
  max: 10,                    // 10 uploads per hour
  message: {
    error: 'Upload limit exceeded',
    message: 'You can upload a maximum of 10 files per hour.'
  }
});

module.exports = {
  authLimiter,
  passwordResetLimiter,
  apiLimiter,
  uploadLimiter
};
```

### Apply to Routes

```javascript
// routes/auth.js
const express = require('express');
const router = express.Router();
const { authLimiter, passwordResetLimiter } = require('../middleware/rateLimiters');

router.post('/login', authLimiter, authController.login);
router.post('/register', authLimiter, authController.register);
router.post('/forgot-password', passwordResetLimiter, authController.forgotPassword);

module.exports = router;
```

---

## Redis-Based Rate Limiting (Distributed)

For applications running on multiple servers, you need a shared rate limit store.

### Installation

```bash
npm install rate-limit-redis ioredis
```

### Configuration

```javascript
// config/redis.js
const Redis = require('ioredis');

const redis = new Redis({
  host: process.env.REDIS_HOST || 'localhost',
  port: process.env.REDIS_PORT || 6379,
  password: process.env.REDIS_PASSWORD,
  maxRetriesPerRequest: 3,
  retryDelayOnFailover: 100,
  enableReadyCheck: true
});

redis.on('error', (err) => {
  console.error('Redis connection error:', err);
});

redis.on('connect', () => {
  console.log('Connected to Redis');
});

module.exports = redis;
```

### Redis Rate Limiter

```javascript
// middleware/redisRateLimiter.js
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');
const redis = require('../config/redis');

// Create Redis store
const createRedisStore = (prefix) => {
  return new RedisStore({
    sendCommand: (...args) => redis.call(...args),
    prefix: `rl:${prefix}:`
  });
};

// Global limiter with Redis
const globalLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  store: createRedisStore('global'),
  standardHeaders: true,
  legacyHeaders: false
});

// Auth limiter with Redis
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  store: createRedisStore('auth'),
  skipSuccessfulRequests: true,
  message: {
    error: 'Too many attempts',
    retryAfter: 900
  }
});

// API limiter with Redis
const apiLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 60,
  store: createRedisStore('api'),
  standardHeaders: true
});

module.exports = {
  globalLimiter,
  authLimiter,
  apiLimiter
};
```

---

## Custom Key Generator

### Rate Limit by User ID (Authenticated Users)

```javascript
const rateLimit = require('express-rate-limit');

const userBasedLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 100,
  keyGenerator: (req) => {
    // Use user ID for authenticated requests, IP for anonymous
    return req.user?.userId || req.ip;
  },
  skip: (req) => {
    // Skip rate limiting for admins
    return req.user?.role === 'ADMIN';
  }
});
```

### Rate Limit by API Key

```javascript
const apiKeyLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 1000,
  keyGenerator: (req) => {
    // Rate limit by API key
    return req.headers['x-api-key'] || req.ip;
  },
  handler: (req, res) => {
    res.status(429).json({
      error: 'Rate limit exceeded',
      apiKey: req.headers['x-api-key']?.substring(0, 8) + '...',
      documentation: 'https://api.example.com/docs/rate-limits'
    });
  }
});
```

---

## Tiered Rate Limiting

Different limits for different user tiers:

```javascript
// middleware/tieredRateLimiter.js
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');
const redis = require('../config/redis');

// Rate limits by tier
const TIER_LIMITS = {
  free: { requests: 100, window: 60 * 60 * 1000 },      // 100/hour
  basic: { requests: 1000, window: 60 * 60 * 1000 },    // 1000/hour
  pro: { requests: 10000, window: 60 * 60 * 1000 },     // 10000/hour
  enterprise: { requests: 100000, window: 60 * 60 * 1000 } // 100000/hour
};

// Dynamic rate limiter based on user tier
const tieredLimiter = async (req, res, next) => {
  // Get user tier (from database or token)
  const tier = req.user?.tier || 'free';
  const limits = TIER_LIMITS[tier];

  // Create dynamic limiter
  const limiter = rateLimit({
    windowMs: limits.window,
    max: limits.requests,
    keyGenerator: (req) => `${tier}:${req.user?.userId || req.ip}`,
    store: new RedisStore({
      sendCommand: (...args) => redis.call(...args),
      prefix: 'rl:tiered:'
    }),
    handler: (req, res) => {
      res.status(429).json({
        error: 'Rate limit exceeded',
        tier,
        limit: limits.requests,
        window: `${limits.window / 1000 / 60} minutes`,
        upgrade: tier !== 'enterprise'
          ? 'Upgrade your plan for higher limits'
          : null
      });
    }
  });

  limiter(req, res, next);
};

module.exports = tieredLimiter;
```

---

## Sliding Window with Redis

Custom implementation for more accurate rate limiting:

```javascript
// middleware/slidingWindowLimiter.js
const redis = require('../config/redis');

const slidingWindowLimiter = (options) => {
  const {
    windowMs = 60000,
    max = 100,
    keyPrefix = 'rl:sliding:'
  } = options;

  return async (req, res, next) => {
    const key = `${keyPrefix}${req.user?.userId || req.ip}`;
    const now = Date.now();
    const windowStart = now - windowMs;

    try {
      // Use Redis sorted set for sliding window
      const pipeline = redis.pipeline();

      // Remove old entries outside window
      pipeline.zremrangebyscore(key, '-inf', windowStart);

      // Add current request
      pipeline.zadd(key, now, `${now}-${Math.random()}`);

      // Count requests in window
      pipeline.zcard(key);

      // Set expiry on key
      pipeline.pexpire(key, windowMs);

      const results = await pipeline.exec();
      const requestCount = results[2][1];

      // Set headers
      res.set({
        'RateLimit-Limit': max,
        'RateLimit-Remaining': Math.max(0, max - requestCount),
        'RateLimit-Reset': Math.ceil((now + windowMs) / 1000)
      });

      if (requestCount > max) {
        return res.status(429).json({
          error: 'Rate limit exceeded',
          retryAfter: Math.ceil(windowMs / 1000)
        });
      }

      next();
    } catch (error) {
      console.error('Rate limiter error:', error);
      // Fail open - allow request if rate limiter fails
      next();
    }
  };
};

module.exports = slidingWindowLimiter;
```

---

## Handling Rate Limit Responses

### Client-Side Handling

```javascript
// client/api.js
async function fetchWithRetry(url, options = {}, maxRetries = 3) {
  let retryCount = 0;

  while (retryCount < maxRetries) {
    const response = await fetch(url, options);

    if (response.status === 429) {
      const retryAfter = response.headers.get('Retry-After');
      const waitTime = retryAfter
        ? parseInt(retryAfter) * 1000
        : Math.pow(2, retryCount) * 1000; // Exponential backoff

      console.log(`Rate limited. Retrying in ${waitTime}ms...`);

      await new Promise(resolve => setTimeout(resolve, waitTime));
      retryCount++;
      continue;
    }

    return response;
  }

  throw new Error('Max retries exceeded');
}
```

### React Hook for Rate Limiting

```jsx
// hooks/useRateLimitedFetch.js
import { useState, useCallback } from 'react';

export function useRateLimitedFetch() {
  const [rateLimited, setRateLimited] = useState(false);
  const [retryAfter, setRetryAfter] = useState(null);

  const fetchWithRateLimit = useCallback(async (url, options) => {
    try {
      const response = await fetch(url, options);

      if (response.status === 429) {
        const retry = response.headers.get('Retry-After');
        setRateLimited(true);
        setRetryAfter(parseInt(retry) || 60);

        // Auto-reset after retry period
        setTimeout(() => {
          setRateLimited(false);
          setRetryAfter(null);
        }, (parseInt(retry) || 60) * 1000);

        throw new Error('Rate limited');
      }

      return response;
    } catch (error) {
      throw error;
    }
  }, []);

  return { fetchWithRateLimit, rateLimited, retryAfter };
}

// Usage
function MyComponent() {
  const { fetchWithRateLimit, rateLimited, retryAfter } = useRateLimitedFetch();

  if (rateLimited) {
    return <div>Please wait {retryAfter} seconds before trying again.</div>;
  }

  // ...
}
```

---

## Advanced Patterns

### IP Whitelist

```javascript
const rateLimit = require('express-rate-limit');

const WHITELISTED_IPS = [
  '127.0.0.1',
  '::1',
  '10.0.0.0/8',
  '192.168.0.0/16'
];

const limiter = rateLimit({
  windowMs: 60 * 1000,
  max: 60,
  skip: (req) => {
    const ip = req.ip;
    return WHITELISTED_IPS.some(whitelisted => {
      if (whitelisted.includes('/')) {
        // CIDR notation - would need ip-range-check library
        return isIpInRange(ip, whitelisted);
      }
      return ip === whitelisted;
    });
  }
});
```

### Cost-Based Rate Limiting

```javascript
// Different endpoints have different "costs"
const ENDPOINT_COSTS = {
  'GET /api/users': 1,
  'GET /api/posts': 1,
  'POST /api/posts': 5,
  'POST /api/upload': 10,
  'GET /api/search': 3
};

const costBasedLimiter = async (req, res, next) => {
  const endpoint = `${req.method} ${req.route?.path || req.path}`;
  const cost = ENDPOINT_COSTS[endpoint] || 1;

  const key = `rl:cost:${req.user?.userId || req.ip}`;
  const limit = 100; // Total cost limit per minute

  const current = await redis.incrbyfloat(key, cost);

  if (current === cost) {
    await redis.expire(key, 60);
  }

  res.set({
    'RateLimit-Cost': cost,
    'RateLimit-Total': current,
    'RateLimit-Limit': limit
  });

  if (current > limit) {
    return res.status(429).json({
      error: 'Rate limit exceeded',
      cost,
      totalCost: current,
      limit
    });
  }

  next();
};
```

### Slow Down Middleware

Instead of blocking, gradually slow down responses:

```bash
npm install express-slow-down
```

```javascript
const slowDown = require('express-slow-down');

const speedLimiter = slowDown({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  delayAfter: 50,             // Allow 50 requests per window without delay
  delayMs: (hits) => hits * 100,  // Add 100ms delay per request above 50
  maxDelayMs: 5000,           // Maximum 5 second delay
  skipFailedRequests: true
});

// Requests 1-50: no delay
// Request 51: 100ms delay
// Request 52: 200ms delay
// Request 100: 5000ms delay (max)
```

---

## Monitoring Rate Limits

```javascript
// middleware/rateLimitMonitor.js
const rateLimit = require('express-rate-limit');

const monitoredLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 100,
  handler: (req, res, next, options) => {
    // Log rate limit hit
    console.log({
      event: 'rate_limit_exceeded',
      ip: req.ip,
      userId: req.user?.userId,
      path: req.path,
      timestamp: new Date().toISOString()
    });

    // Emit metric for monitoring
    // metrics.increment('rate_limit.exceeded', { path: req.path });

    res.status(429).json({
      error: 'Too many requests'
    });
  },
  onLimitReached: (req, res, options) => {
    // Called when limit is first reached
    console.log(`Rate limit reached for ${req.ip}`);
  }
});
```

---

## Testing Rate Limiters

```javascript
// tests/rateLimiter.test.js
const request = require('supertest');
const app = require('../app');
const redis = require('../config/redis');

describe('Rate Limiter', () => {
  beforeEach(async () => {
    // Clear rate limit keys
    const keys = await redis.keys('rl:*');
    if (keys.length > 0) {
      await redis.del(...keys);
    }
  });

  it('should allow requests under limit', async () => {
    for (let i = 0; i < 5; i++) {
      const res = await request(app).get('/api/test');
      expect(res.status).toBe(200);
    }
  });

  it('should block requests over limit', async () => {
    // Make requests up to limit
    for (let i = 0; i < 10; i++) {
      await request(app).get('/api/test');
    }

    // Next request should be blocked
    const res = await request(app).get('/api/test');
    expect(res.status).toBe(429);
    expect(res.body.error).toContain('Too many requests');
  });

  it('should include rate limit headers', async () => {
    const res = await request(app).get('/api/test');

    expect(res.headers['ratelimit-limit']).toBeDefined();
    expect(res.headers['ratelimit-remaining']).toBeDefined();
    expect(res.headers['ratelimit-reset']).toBeDefined();
  });

  it('should track per-user limits', async () => {
    // User 1 makes 10 requests
    for (let i = 0; i < 10; i++) {
      await request(app)
        .get('/api/test')
        .set('Authorization', 'Bearer user1-token');
    }

    // User 2 should still be able to make requests
    const res = await request(app)
      .get('/api/test')
      .set('Authorization', 'Bearer user2-token');

    expect(res.status).toBe(200);
  });
});
```

---

## Practice Exercise

Implement comprehensive rate limiting:

1. **Set up Redis-based rate limiting**
2. **Create endpoint-specific limits**:
   - `/api/auth/*` - 5 requests per 15 minutes
   - `/api/upload` - 10 requests per hour
   - `/api/*` - 100 requests per minute
3. **Implement tiered limits** for free/paid users
4. **Add slow-down middleware** for gradual response delay
5. **Create monitoring** for rate limit events

---

## Key Takeaways

1. **Rate limiting protects your API** - From abuse, attacks, and overload
2. **Use Redis for distributed systems** - Shared state across servers
3. **Different endpoints need different limits** - Auth stricter than general
4. **Include headers** - Let clients know their limit status
5. **Fail open** - Allow requests if rate limiter fails
6. **Monitor rate limit events** - Detect attacks and adjust limits

---

## What's Next?

Tomorrow, we'll explore the **OWASP Top 10** - the most critical security risks for web applications and how to protect against them.
