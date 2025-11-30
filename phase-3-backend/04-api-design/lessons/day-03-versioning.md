# Day 3: API Versioning - Evolving Your API

## Introduction

APIs evolve over time. New features are added, old ones are deprecated, and breaking changes become necessary. API versioning allows you to make these changes without breaking existing clients. Today, you'll learn different versioning strategies and how to implement them effectively.

## Learning Objectives

By the end of this lesson, you will be able to:
- Understand when and why to version your API
- Compare different versioning strategies
- Implement URL-based and header-based versioning
- Handle deprecation gracefully
- Manage multiple API versions

---

## When to Version

### Breaking Changes Require New Version

```javascript
// Breaking changes (require new version):

// 1. Removing a field
// v1: { id, name, email, phone }
// v2: { id, name, email }  // phone removed

// 2. Changing field type
// v1: { price: "19.99" }    // string
// v2: { price: 19.99 }      // number

// 3. Changing field name
// v1: { user_name: "john" }
// v2: { username: "john" }

// 4. Changing response structure
// v1: { user: {...} }
// v2: { data: { user: {...} } }

// 5. Changing status codes
// v1: 200 for created
// v2: 201 for created

// 6. Changing authentication method
// v1: API key in query string
// v2: Bearer token in header
```

### Non-Breaking Changes (No Version Needed)

```javascript
// Safe changes (backward compatible):

// 1. Adding new optional fields
// { id, name } → { id, name, avatar }

// 2. Adding new endpoints
// Adding GET /api/users/stats

// 3. Adding new query parameters
// Adding ?include=posts

// 4. Adding new optional request body fields
// { name } → { name, nickname? }

// 5. Relaxing validation
// Required field → Optional field

// 6. Adding new response headers
// Adding X-Request-Id
```

---

## Versioning Strategies

### 1. URL Path Versioning

Most common and visible approach:

```
https://api.example.com/v1/users
https://api.example.com/v2/users
```

**Pros:**
- Easy to understand and use
- Clear which version is being used
- Easy to route and cache
- Browser-friendly (can test in address bar)

**Cons:**
- Not truly RESTful (version isn't a resource)
- URL changes between versions

### 2. Query Parameter Versioning

```
https://api.example.com/users?version=1
https://api.example.com/users?api-version=2
```

**Pros:**
- Optional, can have default version
- URL structure stays same

**Cons:**
- Easy to forget
- Harder to cache
- Not as visible

### 3. Header Versioning

```
GET /users
Accept: application/vnd.api+json; version=1
```

Or custom header:

```
GET /users
X-API-Version: 2
```

**Pros:**
- Clean URLs
- More RESTful
- Flexible content negotiation

**Cons:**
- Hidden, harder to test
- Not browser-friendly
- Requires header configuration in clients

### 4. Media Type Versioning

```
Accept: application/vnd.company.api.v1+json
Accept: application/vnd.company.api.v2+json
```

**Pros:**
- Most RESTful approach
- Version is part of representation

**Cons:**
- Complex to implement
- Harder for clients to use

---

## URL Path Versioning Implementation

### Router Setup

```javascript
// routes/index.js
const express = require('express');
const router = express.Router();

const v1Routes = require('./v1');
const v2Routes = require('./v2');

// Mount versioned routes
router.use('/v1', v1Routes);
router.use('/v2', v2Routes);

// Redirect unversioned to latest
router.use('/users', (req, res) => {
  res.redirect(307, `/v2${req.url}`);
});

module.exports = router;
```

### V1 Routes

```javascript
// routes/v1/index.js
const express = require('express');
const router = express.Router();
const usersRouter = require('./users');
const productsRouter = require('./products');

router.use('/users', usersRouter);
router.use('/products', productsRouter);

module.exports = router;
```

```javascript
// routes/v1/users.js
const express = require('express');
const router = express.Router();

router.get('/', async (req, res) => {
  const users = await prisma.user.findMany({
    select: {
      id: true,
      name: true,
      email: true,
      phone: true  // Included in v1
    }
  });

  // v1 response format
  res.json(users);
});

router.get('/:id', async (req, res) => {
  const user = await prisma.user.findUnique({
    where: { id: req.params.id },
    select: {
      id: true,
      name: true,
      email: true,
      phone: true
    }
  });

  res.json(user);
});

module.exports = router;
```

### V2 Routes (with changes)

```javascript
// routes/v2/users.js
const express = require('express');
const router = express.Router();

router.get('/', async (req, res) => {
  const users = await prisma.user.findMany({
    select: {
      id: true,
      name: true,
      email: true,
      avatar: true,    // New in v2
      createdAt: true  // New in v2
      // phone removed in v2
    }
  });

  // v2 response format with wrapper
  res.json({
    data: users,
    meta: {
      version: 'v2'
    }
  });
});

router.get('/:id', async (req, res) => {
  const user = await prisma.user.findUnique({
    where: { id: req.params.id },
    select: {
      id: true,
      name: true,
      email: true,
      avatar: true,
      createdAt: true
    }
  });

  res.json({
    data: user,
    links: {
      self: `/v2/users/${user.id}`,
      posts: `/v2/users/${user.id}/posts`
    }
  });
});

module.exports = router;
```

---

## Header-Based Versioning Implementation

### Version Middleware

```javascript
// middleware/apiVersion.js
const DEFAULT_VERSION = '2';
const SUPPORTED_VERSIONS = ['1', '2'];

const apiVersion = (req, res, next) => {
  // Check various version sources
  let version = req.headers['x-api-version']
    || req.headers['api-version']
    || extractFromAcceptHeader(req.headers.accept)
    || DEFAULT_VERSION;

  // Validate version
  if (!SUPPORTED_VERSIONS.includes(version)) {
    return res.status(400).json({
      error: 'Unsupported API version',
      supportedVersions: SUPPORTED_VERSIONS
    });
  }

  req.apiVersion = version;

  // Add version to response headers
  res.set('X-API-Version', version);

  next();
};

function extractFromAcceptHeader(accept) {
  if (!accept) return null;

  // Parse: application/vnd.api.v2+json
  const match = accept.match(/application\/vnd\.api\.v(\d+)\+json/);
  return match ? match[1] : null;
}

module.exports = apiVersion;
```

### Version-Aware Controller

```javascript
// controllers/userController.js
const userController = {
  async list(req, res) {
    const { apiVersion } = req;

    const users = await prisma.user.findMany();

    // Different response based on version
    if (apiVersion === '1') {
      res.json(users.map(u => ({
        id: u.id,
        name: u.name,
        email: u.email,
        phone: u.phone
      })));
    } else {
      res.json({
        data: users.map(u => ({
          id: u.id,
          name: u.name,
          email: u.email,
          avatar: u.avatar,
          createdAt: u.createdAt
        })),
        meta: { version: apiVersion }
      });
    }
  }
};
```

### Router with Version Middleware

```javascript
// routes/users.js
const express = require('express');
const router = express.Router();
const apiVersion = require('../middleware/apiVersion');
const userController = require('../controllers/userController');

router.use(apiVersion);

router.get('/', userController.list);
router.get('/:id', userController.get);

module.exports = router;
```

---

## Shared Code Between Versions

### Service Layer Pattern

```javascript
// services/userService.js
const userService = {
  async findAll(options = {}) {
    return prisma.user.findMany({
      where: options.where,
      orderBy: options.orderBy,
      skip: options.skip,
      take: options.take
    });
  },

  async findById(id) {
    return prisma.user.findUnique({
      where: { id }
    });
  },

  async create(data) {
    return prisma.user.create({ data });
  }
};

module.exports = userService;
```

### Version-Specific Transformers

```javascript
// transformers/userTransformer.js
const userTransformer = {
  v1: {
    single(user) {
      return {
        id: user.id,
        name: user.name,
        email: user.email,
        phone: user.phone
      };
    },

    collection(users) {
      return users.map(this.single);
    }
  },

  v2: {
    single(user) {
      return {
        id: user.id,
        name: user.name,
        email: user.email,
        avatar: user.avatar,
        createdAt: user.createdAt,
        links: {
          self: `/v2/users/${user.id}`,
          posts: `/v2/users/${user.id}/posts`
        }
      };
    },

    collection(users) {
      return {
        data: users.map(u => this.single(u)),
        meta: {
          count: users.length
        }
      };
    }
  }
};

// Usage
const transform = userTransformer[`v${req.apiVersion}`];
res.json(transform.collection(users));
```

---

## Deprecation Strategy

### Deprecation Headers

```javascript
// middleware/deprecation.js
const deprecatedEndpoints = {
  '/v1/users': {
    sunset: '2025-06-01',
    replacement: '/v2/users',
    message: 'v1 API will be removed on June 1, 2025'
  },
  '/v1/products': {
    sunset: '2025-06-01',
    replacement: '/v2/products'
  }
};

const deprecationWarning = (req, res, next) => {
  const path = req.path;

  for (const [deprecatedPath, info] of Object.entries(deprecatedEndpoints)) {
    if (path.startsWith(deprecatedPath)) {
      res.set({
        'Deprecation': `date="${info.sunset}"`,
        'Sunset': new Date(info.sunset).toUTCString(),
        'Link': `<${info.replacement}>; rel="successor-version"`
      });

      if (info.message) {
        res.set('X-Deprecation-Notice', info.message);
      }

      break;
    }
  }

  next();
};

module.exports = deprecationWarning;
```

### Deprecation Logging

```javascript
// middleware/deprecationLogger.js
const deprecationLogger = (req, res, next) => {
  if (req.path.includes('/v1/')) {
    console.log({
      type: 'deprecated_api_call',
      path: req.path,
      method: req.method,
      clientId: req.headers['x-client-id'],
      ip: req.ip,
      timestamp: new Date().toISOString()
    });

    // Track in analytics
    // analytics.track('deprecated_api_call', { ... });
  }

  next();
};
```

### Client Communication

```javascript
// Deprecation response format
{
  "data": [...],
  "warnings": [
    {
      "code": "DEPRECATED",
      "message": "This endpoint is deprecated and will be removed on 2025-06-01",
      "documentation": "https://api.example.com/docs/migration/v1-to-v2"
    }
  ]
}
```

---

## Version Migration Guide

### Documentation Template

```markdown
# API v1 to v2 Migration Guide

## Overview
API v2 introduces improvements in response structure and removes deprecated fields.

## Timeline
- v2 released: January 1, 2025
- v1 deprecated: March 1, 2025
- v1 sunset: June 1, 2025

## Breaking Changes

### Response Structure
v1:
```json
[{ "id": 1, "name": "John" }]
```

v2:
```json
{ "data": [{ "id": 1, "name": "John" }], "meta": {...} }
```

### Removed Fields
| Endpoint | Removed Field | Alternative |
|----------|--------------|-------------|
| /users | phone | Use /users/{id}/contact |

### Changed Fields
| Endpoint | v1 | v2 |
|----------|----|----|
| /users | user_name | username |

## Migration Steps
1. Update response parsing to handle wrapper object
2. Update field names in your code
3. Test with v2 endpoints
4. Update API version in your client
```

---

## Automated Version Detection

```javascript
// For gradual migration, detect client capability
const detectClientVersion = (req) => {
  // Check if client sends v2 request body format
  if (req.body && req.body.data) {
    return '2';
  }

  // Check user agent for known v2 clients
  const ua = req.headers['user-agent'] || '';
  if (ua.includes('MyApp/2.')) {
    return '2';
  }

  // Default to v1 for backward compatibility
  return '1';
};

// Auto-negotiate version
app.use((req, res, next) => {
  if (!req.headers['x-api-version']) {
    req.apiVersion = detectClientVersion(req);
  }
  next();
});
```

---

## Testing Multiple Versions

```javascript
// tests/api-versions.test.js
const request = require('supertest');
const app = require('../app');

describe('API Versioning', () => {
  describe('v1 Users API', () => {
    it('should return v1 format', async () => {
      const res = await request(app)
        .get('/v1/users')
        .expect(200);

      expect(Array.isArray(res.body)).toBe(true);
      expect(res.body[0]).toHaveProperty('phone');
    });

    it('should include deprecation headers', async () => {
      const res = await request(app)
        .get('/v1/users')
        .expect(200);

      expect(res.headers['deprecation']).toBeDefined();
      expect(res.headers['sunset']).toBeDefined();
    });
  });

  describe('v2 Users API', () => {
    it('should return v2 format with wrapper', async () => {
      const res = await request(app)
        .get('/v2/users')
        .expect(200);

      expect(res.body).toHaveProperty('data');
      expect(res.body).toHaveProperty('meta');
      expect(res.body.data[0]).not.toHaveProperty('phone');
    });
  });

  describe('Header-based versioning', () => {
    it('should respect X-API-Version header', async () => {
      const res = await request(app)
        .get('/users')
        .set('X-API-Version', '1')
        .expect(200);

      expect(res.headers['x-api-version']).toBe('1');
    });
  });
});
```

---

## Best Practices

### 1. Version Early

```javascript
// Start with versioning from day one
// Even if you only have v1
app.use('/api/v1', routes);
```

### 2. Support Multiple Versions

```javascript
// Maintain at least 2 versions
// Current + previous
const ACTIVE_VERSIONS = ['v1', 'v2'];
const SUNSET_VERSIONS = ['v0'];  // Read-only, deprecated
```

### 3. Version the API, Not Resources

```javascript
// Good: Version at API level
/v1/users
/v2/users

// Avoid: Version at resource level
/users/v1
/users/v2
```

### 4. Don't Version Too Often

```javascript
// Release new version only for breaking changes
// Use feature flags for non-breaking features
app.get('/v1/users', (req, res) => {
  const users = getUsers();

  // Feature flag, not new version
  if (req.query.include === 'posts') {
    users.forEach(u => u.posts = getPosts(u.id));
  }

  res.json(users);
});
```

---

## Practice Exercise

Implement API versioning:

1. **Create v1 and v2** of a products API
2. **v1 → v2 changes**:
   - Add wrapper object `{ data: [...] }`
   - Rename `product_name` to `name`
   - Remove `sku` field
   - Add `createdAt` and `updatedAt`
3. **Implement deprecation** headers for v1
4. **Create migration** documentation
5. **Write tests** for both versions

---

## Key Takeaways

1. **Version for breaking changes only** - Additive changes don't need versions
2. **URL versioning is simplest** - Easy to use and understand
3. **Support graceful deprecation** - Give clients time to migrate
4. **Share code between versions** - Use transformers for differences
5. **Document migrations clearly** - Make upgrades easy
6. **Test all versions** - Maintain quality across versions

---

## What's Next?

Tomorrow, we'll learn about **API Documentation** - how to document your API effectively using OpenAPI/Swagger.
