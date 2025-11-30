# Day 1: REST API Principles - Designing Clean APIs

## Introduction

REST (Representational State Transfer) is an architectural style for designing networked applications. Well-designed REST APIs are intuitive, consistent, and easy to use. Today, you'll learn the fundamental principles of REST and how to apply them when designing your APIs.

## Learning Objectives

By the end of this lesson, you will be able to:
- Understand REST architectural constraints
- Design resource-oriented APIs
- Use HTTP methods correctly
- Structure URLs following conventions
- Apply REST best practices

---

## REST Architectural Constraints

REST defines six constraints that guide API design:

### 1. Client-Server Separation

```
┌────────────┐                    ┌────────────┐
│   Client   │  HTTP Requests →   │   Server   │
│  (React)   │  ← JSON Responses  │  (Express) │
└────────────┘                    └────────────┘

- Client handles UI/UX
- Server handles data/business logic
- Each can evolve independently
```

### 2. Statelessness

Each request contains all information needed to process it:

```javascript
// GOOD: Stateless - Token contains user info
GET /api/orders
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

// BAD: Stateful - Server remembers user from previous requests
GET /api/orders
// Server: "Who is making this request again?"
```

### 3. Cacheability

Responses should indicate if they can be cached:

```javascript
// Cacheable response
app.get('/api/products/:id', (req, res) => {
  res.set({
    'Cache-Control': 'public, max-age=3600',  // Cache for 1 hour
    'ETag': '"product-v1"'
  });
  res.json(product);
});

// Non-cacheable response
app.get('/api/user/profile', (req, res) => {
  res.set('Cache-Control', 'no-store');  // Never cache
  res.json(profile);
});
```

### 4. Layered System

Client doesn't know if it's connected directly to server or intermediary:

```
Client → CDN → Load Balancer → API Gateway → Server
        ↓
      Cache
```

### 5. Uniform Interface

Consistent, predictable API design across all endpoints.

### 6. Code on Demand (Optional)

Server can extend client functionality by sending executable code.

---

## Resource-Oriented Design

### What is a Resource?

A resource is any concept that can be named and represented:

```
Resources:
- Users       → /users
- Products    → /products
- Orders      → /orders
- Cart Items  → /cart/items

Not Resources (Actions):
- /getUsers           ❌
- /createNewProduct   ❌
- /deleteOrder        ❌
```

### Resource Naming Conventions

```javascript
// USE: Nouns (plural for collections)
GET /users              // Collection of users
GET /users/123          // Single user
GET /products           // Collection of products
GET /products/456       // Single product

// AVOID: Verbs in URLs
GET /getUser/123        ❌
POST /createProduct     ❌
GET /deleteOrder/789    ❌

// USE: Lowercase, hyphen-separated
GET /user-profiles      ✓
GET /order-items        ✓

// AVOID: Camel case or underscores
GET /userProfiles       ❌
GET /order_items        ❌
```

### Nested Resources

```javascript
// Resources that belong to other resources
GET /users/123/orders           // Orders for user 123
GET /orders/456/items           // Items in order 456
GET /posts/789/comments         // Comments on post 789

// Avoid deep nesting (max 2-3 levels)
GET /users/123/orders/456/items/789/details  ❌ Too deep!

// Better: Use query parameters or flat structure
GET /items/789?order=456        ✓
GET /order-items/789            ✓
```

---

## HTTP Methods (CRUD Operations)

### The Standard Methods

| Method | Action | Description | Idempotent | Safe |
|--------|--------|-------------|------------|------|
| GET | Read | Retrieve resource(s) | Yes | Yes |
| POST | Create | Create new resource | No | No |
| PUT | Update | Replace entire resource | Yes | No |
| PATCH | Update | Partial update | Yes | No |
| DELETE | Delete | Remove resource | Yes | No |

### Idempotent vs Safe

```javascript
// SAFE: No side effects (GET)
GET /users/123  // Can call 1000 times, server state unchanged

// IDEMPOTENT: Same result regardless of times called
DELETE /users/123  // First call deletes, subsequent calls do nothing
PUT /users/123     // Replaces with same data each time

// NOT IDEMPOTENT: Different result each time
POST /orders       // Creates new order each time
```

### Implementation Examples

```javascript
// GET - Retrieve resources
app.get('/api/users', async (req, res) => {
  const users = await prisma.user.findMany();
  res.json(users);
});

app.get('/api/users/:id', async (req, res) => {
  const user = await prisma.user.findUnique({
    where: { id: req.params.id }
  });

  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }

  res.json(user);
});

// POST - Create new resource
app.post('/api/users', async (req, res) => {
  const user = await prisma.user.create({
    data: req.body
  });

  res.status(201)
    .location(`/api/users/${user.id}`)
    .json(user);
});

// PUT - Replace entire resource
app.put('/api/users/:id', async (req, res) => {
  const user = await prisma.user.update({
    where: { id: req.params.id },
    data: req.body  // Complete replacement
  });

  res.json(user);
});

// PATCH - Partial update
app.patch('/api/users/:id', async (req, res) => {
  const user = await prisma.user.update({
    where: { id: req.params.id },
    data: req.body  // Only update provided fields
  });

  res.json(user);
});

// DELETE - Remove resource
app.delete('/api/users/:id', async (req, res) => {
  await prisma.user.delete({
    where: { id: req.params.id }
  });

  res.status(204).end();  // No content
});
```

---

## HTTP Status Codes

### Success Codes (2xx)

```javascript
// 200 OK - Request succeeded
res.status(200).json({ user });

// 201 Created - Resource created
res.status(201).json({ user });

// 204 No Content - Success, no body (DELETE)
res.status(204).end();

// 202 Accepted - Request accepted for processing
res.status(202).json({ message: 'Processing started', jobId: '123' });
```

### Client Error Codes (4xx)

```javascript
// 400 Bad Request - Invalid request data
res.status(400).json({
  error: 'Validation Error',
  details: [
    { field: 'email', message: 'Invalid email format' }
  ]
});

// 401 Unauthorized - Authentication required
res.status(401).json({ error: 'Authentication required' });

// 403 Forbidden - Authenticated but not authorized
res.status(403).json({ error: 'Access denied' });

// 404 Not Found - Resource doesn't exist
res.status(404).json({ error: 'User not found' });

// 405 Method Not Allowed - Wrong HTTP method
res.status(405).json({ error: 'Method not allowed' });

// 409 Conflict - Resource conflict
res.status(409).json({ error: 'Email already registered' });

// 422 Unprocessable Entity - Valid syntax but semantic errors
res.status(422).json({
  error: 'Cannot process request',
  details: 'Order total cannot be negative'
});

// 429 Too Many Requests - Rate limited
res.status(429).json({
  error: 'Rate limit exceeded',
  retryAfter: 60
});
```

### Server Error Codes (5xx)

```javascript
// 500 Internal Server Error - Unexpected error
res.status(500).json({ error: 'Internal server error' });

// 502 Bad Gateway - Upstream server error
res.status(502).json({ error: 'Service temporarily unavailable' });

// 503 Service Unavailable - Server overloaded
res.status(503).json({
  error: 'Service unavailable',
  retryAfter: 30
});
```

---

## Request/Response Design

### Request Structure

```javascript
// Headers
{
  "Content-Type": "application/json",
  "Authorization": "Bearer token...",
  "Accept": "application/json",
  "Accept-Language": "en-US"
}

// Request body (POST/PUT/PATCH)
{
  "name": "John Doe",
  "email": "john@example.com",
  "role": "user"
}

// Query parameters (filtering, pagination)
GET /api/users?role=admin&status=active&page=1&limit=10
```

### Response Structure

```javascript
// Successful response
{
  "data": {
    "id": "123",
    "name": "John Doe",
    "email": "john@example.com",
    "createdAt": "2024-01-15T10:30:00Z"
  }
}

// Collection response with metadata
{
  "data": [
    { "id": "1", "name": "Product A" },
    { "id": "2", "name": "Product B" }
  ],
  "meta": {
    "total": 100,
    "page": 1,
    "limit": 10,
    "totalPages": 10
  },
  "links": {
    "self": "/api/products?page=1",
    "next": "/api/products?page=2",
    "prev": null,
    "first": "/api/products?page=1",
    "last": "/api/products?page=10"
  }
}

// Error response
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request data",
    "details": [
      {
        "field": "email",
        "message": "Must be a valid email address"
      }
    ]
  }
}
```

---

## Consistent API Design

### Response Wrapper Pattern

```javascript
// Utility function for consistent responses
const ApiResponse = {
  success(data, meta = null) {
    const response = { data };
    if (meta) response.meta = meta;
    return response;
  },

  error(code, message, details = null) {
    const response = {
      error: { code, message }
    };
    if (details) response.error.details = details;
    return response;
  },

  paginated(data, page, limit, total) {
    return {
      data,
      meta: {
        page,
        limit,
        total,
        totalPages: Math.ceil(total / limit)
      }
    };
  }
};

// Usage
app.get('/api/users/:id', async (req, res) => {
  const user = await prisma.user.findUnique({
    where: { id: req.params.id }
  });

  if (!user) {
    return res.status(404).json(
      ApiResponse.error('NOT_FOUND', 'User not found')
    );
  }

  res.json(ApiResponse.success(user));
});
```

### Field Naming Conventions

```javascript
// USE: camelCase for JSON
{
  "firstName": "John",
  "lastName": "Doe",
  "createdAt": "2024-01-15T10:30:00Z",
  "isActive": true
}

// AVOID: snake_case or PascalCase
{
  "first_name": "John",    // ❌
  "LastName": "Doe"        // ❌
}

// Date/Time: Use ISO 8601 format
{
  "createdAt": "2024-01-15T10:30:00Z",
  "expiresAt": "2024-01-15T11:30:00+05:30"
}
```

---

## HATEOAS (Hypermedia as the Engine of Application State)

Links in responses guide clients to related actions:

```javascript
// Order response with hypermedia links
{
  "data": {
    "id": "order-123",
    "status": "pending",
    "total": 99.99,
    "createdAt": "2024-01-15T10:30:00Z"
  },
  "links": {
    "self": { "href": "/api/orders/order-123" },
    "payment": { "href": "/api/orders/order-123/payment", "method": "POST" },
    "cancel": { "href": "/api/orders/order-123", "method": "DELETE" },
    "items": { "href": "/api/orders/order-123/items" },
    "customer": { "href": "/api/users/user-456" }
  },
  "actions": [
    {
      "name": "pay",
      "title": "Pay for order",
      "method": "POST",
      "href": "/api/orders/order-123/payment",
      "fields": [
        { "name": "paymentMethod", "type": "string", "required": true }
      ]
    }
  ]
}
```

---

## Actions on Resources

When you need to perform actions that don't fit CRUD:

```javascript
// Option 1: Use POST with action in URL
POST /api/orders/123/cancel
POST /api/users/456/activate
POST /api/posts/789/publish

// Option 2: Use status update via PATCH
PATCH /api/orders/123
{
  "status": "cancelled"
}

// Option 3: Create sub-resource
POST /api/orders/123/cancellation
{
  "reason": "Customer request",
  "refundAmount": 49.99
}
```

### Bulk Operations

```javascript
// Bulk update
PATCH /api/products
{
  "ids": ["1", "2", "3"],
  "update": {
    "status": "archived"
  }
}

// Bulk delete
DELETE /api/products?ids=1,2,3

// Or use POST for complex bulk operations
POST /api/products/bulk-delete
{
  "ids": ["1", "2", "3"]
}
```

---

## Express Router Organization

```javascript
// routes/users.js
const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');
const { authenticate, authorize } = require('../middleware/auth');
const { validateUser } = require('../middleware/validators');

// Collection routes
router.route('/')
  .get(authenticate, userController.list)
  .post(authenticate, authorize('admin'), validateUser, userController.create);

// Individual resource routes
router.route('/:id')
  .get(authenticate, userController.get)
  .put(authenticate, authorize('admin'), validateUser, userController.update)
  .patch(authenticate, userController.partialUpdate)
  .delete(authenticate, authorize('admin'), userController.delete);

// Nested resources
router.get('/:id/orders', authenticate, userController.getOrders);
router.get('/:id/addresses', authenticate, userController.getAddresses);

// Actions
router.post('/:id/activate', authenticate, authorize('admin'), userController.activate);
router.post('/:id/deactivate', authenticate, authorize('admin'), userController.deactivate);

module.exports = router;
```

### Controller Pattern

```javascript
// controllers/userController.js
const { prisma } = require('../lib/prisma');
const ApiResponse = require('../utils/apiResponse');

const userController = {
  async list(req, res) {
    const { page = 1, limit = 10, role, status } = req.query;

    const where = {};
    if (role) where.role = role;
    if (status) where.status = status;

    const [users, total] = await Promise.all([
      prisma.user.findMany({
        where,
        skip: (page - 1) * limit,
        take: parseInt(limit),
        select: {
          id: true,
          name: true,
          email: true,
          role: true,
          createdAt: true
        }
      }),
      prisma.user.count({ where })
    ]);

    res.json(ApiResponse.paginated(users, parseInt(page), parseInt(limit), total));
  },

  async get(req, res) {
    const user = await prisma.user.findUnique({
      where: { id: req.params.id },
      select: {
        id: true,
        name: true,
        email: true,
        role: true,
        createdAt: true
      }
    });

    if (!user) {
      return res.status(404).json(
        ApiResponse.error('NOT_FOUND', 'User not found')
      );
    }

    res.json(ApiResponse.success(user));
  },

  async create(req, res) {
    const user = await prisma.user.create({
      data: req.body,
      select: {
        id: true,
        name: true,
        email: true,
        role: true,
        createdAt: true
      }
    });

    res.status(201)
      .location(`/api/users/${user.id}`)
      .json(ApiResponse.success(user));
  },

  async update(req, res) {
    try {
      const user = await prisma.user.update({
        where: { id: req.params.id },
        data: req.body,
        select: {
          id: true,
          name: true,
          email: true,
          role: true,
          updatedAt: true
        }
      });

      res.json(ApiResponse.success(user));
    } catch (error) {
      if (error.code === 'P2025') {
        return res.status(404).json(
          ApiResponse.error('NOT_FOUND', 'User not found')
        );
      }
      throw error;
    }
  },

  async delete(req, res) {
    try {
      await prisma.user.delete({
        where: { id: req.params.id }
      });

      res.status(204).end();
    } catch (error) {
      if (error.code === 'P2025') {
        return res.status(404).json(
          ApiResponse.error('NOT_FOUND', 'User not found')
        );
      }
      throw error;
    }
  }
};

module.exports = userController;
```

---

## Practice Exercise

Design a REST API for an e-commerce platform:

1. **Identify resources**: Products, Categories, Users, Orders, Cart, Reviews
2. **Design endpoints**:
   ```
   Products:
   GET    /api/products
   GET    /api/products/:id
   POST   /api/products
   PUT    /api/products/:id
   DELETE /api/products/:id

   Categories:
   GET    /api/categories
   GET    /api/categories/:id/products

   Orders:
   GET    /api/orders
   GET    /api/orders/:id
   POST   /api/orders
   POST   /api/orders/:id/cancel

   Cart:
   GET    /api/cart
   POST   /api/cart/items
   PUT    /api/cart/items/:id
   DELETE /api/cart/items/:id
   ```

3. **Define response structures** for each endpoint
4. **Implement** at least 3 endpoints with proper status codes

---

## Key Takeaways

1. **Resources are nouns** - `/users`, not `/getUsers`
2. **HTTP methods define actions** - GET reads, POST creates, etc.
3. **Status codes communicate results** - 201 for created, 404 for not found
4. **Consistency is key** - Same patterns across all endpoints
5. **Stateless requests** - Each request is independent
6. **Use proper HTTP semantics** - Idempotency, cacheability

---

## What's Next?

Tomorrow, we'll learn about **Pagination and Filtering** - how to handle large datasets and give clients control over what data they receive.
