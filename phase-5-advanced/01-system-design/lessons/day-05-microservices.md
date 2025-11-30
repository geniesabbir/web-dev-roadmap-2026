# Day 5: Microservices Architecture

## Introduction

Microservices architecture structures an application as a collection of loosely coupled, independently deployable services. Each service is owned by a small team and focuses on a specific business capability. Today, you'll learn when to use microservices, how to design service boundaries, and patterns for building distributed systems.

## Learning Objectives

By the end of this lesson, you will be able to:
- Understand microservices vs monolith trade-offs
- Design service boundaries using Domain-Driven Design
- Implement inter-service communication patterns
- Handle distributed system challenges
- Deploy and manage microservices

---

## Monolith vs Microservices

```
┌─────────────────────────────────────────────────────────────────┐
│                ARCHITECTURE COMPARISON                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  MONOLITH                          MICROSERVICES                 │
│  ┌────────────────────┐            ┌─────┐ ┌─────┐ ┌─────┐      │
│  │     Application    │            │User │ │Order│ │Inv. │      │
│  │  ┌──────────────┐  │            │ Svc │ │ Svc │ │ Svc │      │
│  │  │    Users     │  │            └──┬──┘ └──┬──┘ └──┬──┘      │
│  │  ├──────────────┤  │               │       │       │         │
│  │  │    Orders    │  │            ┌──┴───────┴───────┴──┐      │
│  │  ├──────────────┤  │            │    Message Bus      │      │
│  │  │   Inventory  │  │            └──┬───────┬───────┬──┘      │
│  │  ├──────────────┤  │               │       │       │         │
│  │  │   Payments   │  │            ┌──┴──┐ ┌──┴──┐ ┌──┴──┐      │
│  │  └──────────────┘  │            │Pay  │ │Notif│ │Anlyt│      │
│  │  ┌──────────────┐  │            │ Svc │ │ Svc │ │ Svc │      │
│  │  │   Database   │  │            └─────┘ └─────┘ └─────┘      │
│  │  └──────────────┘  │               │       │       │         │
│  └────────────────────┘            ┌──┴──┐ ┌──┴──┐ ┌──┴──┐      │
│                                    │ DB  │ │ DB  │ │ DB  │      │
│                                    └─────┘ └─────┘ └─────┘      │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  MONOLITH PROS                     MICROSERVICES PROS           │
│  ✓ Simple development              ✓ Independent deployment     │
│  ✓ Easy debugging                  ✓ Technology flexibility     │
│  ✓ Single deployment               ✓ Scalability per service    │
│  ✓ No network latency              ✓ Team autonomy              │
│  ✓ ACID transactions               ✓ Fault isolation            │
│                                                                  │
│  MONOLITH CONS                     MICROSERVICES CONS           │
│  ✗ Scaling is all-or-nothing       ✗ Distributed complexity     │
│  ✗ One failure breaks all          ✗ Network latency            │
│  ✗ Technology lock-in              ✗ Data consistency           │
│  ✗ Large codebase = slow dev       ✗ Operational overhead       │
│  ✗ Coordinated deploys             ✗ Testing complexity         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### When to Use Each

```
┌─────────────────────────────────────────────────────────────────┐
│                    DECISION GUIDE                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  START WITH MONOLITH WHEN:                                       │
│  • New product/startup (MVP stage)                               │
│  • Small team (< 10 developers)                                  │
│  • Domain is not well understood                                 │
│  • Simple scaling requirements                                   │
│  • Time to market is critical                                    │
│                                                                  │
│  CONSIDER MICROSERVICES WHEN:                                    │
│  • Large team (10+ developers)                                   │
│  • Need independent deployments                                  │
│  • Different scaling requirements per component                  │
│  • Polyglot technology needs                                     │
│  • Domain is well understood                                     │
│  • DevOps maturity is high                                       │
│                                                                  │
│  ⚠️ WARNING: Premature microservices is worse than monolith!    │
│                                                                  │
│  MIGRATION PATH:                                                 │
│  Monolith → Modular Monolith → Microservices                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Service Design

### Domain-Driven Design (DDD)

```
┌─────────────────────────────────────────────────────────────────┐
│                    BOUNDED CONTEXTS                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  E-Commerce Domain                                               │
│                                                                  │
│  ┌───────────────┐    ┌───────────────┐    ┌───────────────┐    │
│  │   IDENTITY    │    │    CATALOG    │    │    ORDERS     │    │
│  │   CONTEXT     │    │    CONTEXT    │    │    CONTEXT    │    │
│  │ ┌───────────┐ │    │ ┌───────────┐ │    │ ┌───────────┐ │    │
│  │ │   User    │ │    │ │  Product  │ │    │ │   Order   │ │    │
│  │ │  Account  │ │    │ │  Category │ │    │ │ OrderItem │ │    │
│  │ │   Role    │ │    │ │  Pricing  │ │    │ │  Payment  │ │    │
│  │ └───────────┘ │    │ └───────────┘ │    │ └───────────┘ │    │
│  └───────────────┘    └───────────────┘    └───────────────┘    │
│                                                                  │
│  ┌───────────────┐    ┌───────────────┐    ┌───────────────┐    │
│  │   INVENTORY   │    │   SHIPPING    │    │  NOTIFICATION │    │
│  │   CONTEXT     │    │   CONTEXT     │    │    CONTEXT    │    │
│  │ ┌───────────┐ │    │ ┌───────────┐ │    │ ┌───────────┐ │    │
│  │ │   Stock   │ │    │ │  Shipment │ │    │ │   Email   │ │    │
│  │ │ Warehouse │ │    │ │  Carrier  │ │    │ │    SMS    │ │    │
│  │ │ Supplier  │ │    │ │  Tracking │ │    │ │   Push    │ │    │
│  │ └───────────┘ │    │ └───────────┘ │    │ └───────────┘ │    │
│  └───────────────┘    └───────────────┘    └───────────────┘    │
│                                                                  │
│  Each bounded context = potential microservice                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Service Boundaries

```javascript
// Good service boundary - Cohesive, single responsibility
// user-service/

// ✓ All user-related operations in one service
const userService = {
  createUser: (userData) => { /* ... */ },
  updateUser: (userId, updates) => { /* ... */ },
  getUser: (userId) => { /* ... */ },
  authenticate: (credentials) => { /* ... */ },
  changePassword: (userId, newPassword) => { /* ... */ },
  getUserPreferences: (userId) => { /* ... */ }
};

// Bad service boundary - Too granular
// ✗ Separate services for user CRUD and preferences
// user-crud-service/
// user-preferences-service/
// user-auth-service/
// This creates unnecessary network calls between tightly coupled data

// Bad service boundary - Too large
// ✗ One service for users, orders, and payments
// user-order-payment-service/
// This is just a distributed monolith
```

### Service Template

```javascript
// service-template/
// ├── src/
// │   ├── api/           # HTTP/gRPC handlers
// │   ├── domain/        # Business logic
// │   ├── infrastructure/# Database, external services
// │   ├── events/        # Event publishing/subscribing
// │   └── index.js
// ├── tests/
// ├── Dockerfile
// └── package.json

// src/index.js
const express = require('express');
const { connectDatabase } = require('./infrastructure/database');
const { setupEventHandlers } = require('./events');
const routes = require('./api/routes');
const { healthCheck } = require('./api/health');

async function startService() {
  const app = express();

  // Middleware
  app.use(express.json());

  // Health check endpoint
  app.get('/health', healthCheck);

  // API routes
  app.use('/api', routes);

  // Connect to database
  await connectDatabase();

  // Setup event handlers
  await setupEventHandlers();

  // Start server
  const port = process.env.PORT || 3000;
  app.listen(port, () => {
    console.log(`Service started on port ${port}`);
  });
}

startService().catch(console.error);
```

---

## Inter-Service Communication

### Synchronous Communication (HTTP/gRPC)

```javascript
// http-client.js - Service-to-service HTTP calls
const axios = require('axios');

class ServiceClient {
  constructor(baseURL, options = {}) {
    this.client = axios.create({
      baseURL,
      timeout: options.timeout || 5000,
      headers: {
        'Content-Type': 'application/json'
      }
    });

    // Add retry logic
    this.setupRetry(options.retries || 3);

    // Add circuit breaker
    this.circuitBreaker = new CircuitBreaker(options.circuitBreaker);
  }

  setupRetry(maxRetries) {
    this.client.interceptors.response.use(
      response => response,
      async error => {
        const config = error.config;

        if (!config || !config.retry) {
          config.retry = 0;
        }

        if (config.retry >= maxRetries) {
          throw error;
        }

        // Only retry on network errors or 5xx
        if (!error.response || error.response.status >= 500) {
          config.retry++;
          const delay = Math.pow(2, config.retry) * 100;
          await new Promise(r => setTimeout(r, delay));
          return this.client(config);
        }

        throw error;
      }
    );
  }

  async get(path, options = {}) {
    return this.circuitBreaker.execute(() =>
      this.client.get(path, options)
    );
  }

  async post(path, data, options = {}) {
    return this.circuitBreaker.execute(() =>
      this.client.post(path, data, options)
    );
  }
}

// Service clients
const userService = new ServiceClient(
  process.env.USER_SERVICE_URL || 'http://user-service:3001'
);

const orderService = new ServiceClient(
  process.env.ORDER_SERVICE_URL || 'http://order-service:3002'
);

// Usage in order-service
async function createOrder(orderData) {
  // Call user service to validate user
  const { data: user } = await userService.get(`/users/${orderData.userId}`);

  if (!user.active) {
    throw new Error('User account is not active');
  }

  // Create order
  const order = await db.orders.create(orderData);

  return order;
}
```

### Asynchronous Communication (Events)

```javascript
// event-bus.js
const { Queue, Worker } = require('bullmq');

class EventBus {
  constructor(connection) {
    this.connection = connection;
    this.publishers = new Map();
    this.subscribers = new Map();
  }

  async publish(eventName, data) {
    if (!this.publishers.has(eventName)) {
      this.publishers.set(eventName, new Queue(eventName, {
        connection: this.connection
      }));
    }

    const queue = this.publishers.get(eventName);
    await queue.add(eventName, {
      type: eventName,
      data,
      timestamp: Date.now(),
      source: process.env.SERVICE_NAME
    });
  }

  subscribe(eventName, handler) {
    if (this.subscribers.has(eventName)) {
      throw new Error(`Already subscribed to ${eventName}`);
    }

    const worker = new Worker(eventName, async (job) => {
      await handler(job.data.data, job.data);
    }, {
      connection: this.connection
    });

    this.subscribers.set(eventName, worker);
  }
}

// Usage - Order Service publishes events
const eventBus = new EventBus(redisConnection);

async function createOrder(orderData) {
  const order = await db.orders.create(orderData);

  // Publish event for other services
  await eventBus.publish('order.created', {
    orderId: order.id,
    userId: order.userId,
    items: order.items,
    total: order.total
  });

  return order;
}

// Usage - Inventory Service subscribes to events
eventBus.subscribe('order.created', async (orderData) => {
  console.log('Received order.created event:', orderData.orderId);

  // Reserve inventory for order items
  for (const item of orderData.items) {
    await reserveStock(item.productId, item.quantity);
  }
});

// Usage - Notification Service subscribes to events
eventBus.subscribe('order.created', async (orderData) => {
  // Send order confirmation email
  await sendOrderConfirmation(orderData.userId, orderData.orderId);
});
```

### API Gateway Pattern

```javascript
// api-gateway/src/index.js
const express = require('express');
const httpProxy = require('http-proxy');
const rateLimit = require('express-rate-limit');
const jwt = require('jsonwebtoken');

const app = express();
const proxy = httpProxy.createProxyServer();

// Service registry
const services = {
  users: process.env.USER_SERVICE_URL || 'http://user-service:3001',
  orders: process.env.ORDER_SERVICE_URL || 'http://order-service:3002',
  products: process.env.PRODUCT_SERVICE_URL || 'http://product-service:3003',
  payments: process.env.PAYMENT_SERVICE_URL || 'http://payment-service:3004'
};

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100
});

app.use(limiter);

// Authentication middleware
function authenticate(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1];

  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    // Forward user info to services
    req.headers['x-user-id'] = decoded.userId;
    req.headers['x-user-role'] = decoded.role;
    next();
  } catch (error) {
    res.status(401).json({ error: 'Invalid token' });
  }
}

// Public routes (no auth required)
app.post('/api/auth/login', (req, res) => {
  proxy.web(req, res, { target: services.users });
});

app.post('/api/auth/register', (req, res) => {
  proxy.web(req, res, { target: services.users });
});

// Protected routes
app.use('/api/users', authenticate, (req, res) => {
  proxy.web(req, res, { target: services.users });
});

app.use('/api/orders', authenticate, (req, res) => {
  proxy.web(req, res, { target: services.orders });
});

app.use('/api/products', (req, res) => {
  // Products can be public for browsing
  proxy.web(req, res, { target: services.products });
});

app.use('/api/payments', authenticate, (req, res) => {
  proxy.web(req, res, { target: services.payments });
});

// Error handling
proxy.on('error', (err, req, res) => {
  console.error('Proxy error:', err);
  res.status(502).json({ error: 'Service unavailable' });
});

app.listen(3000, () => {
  console.log('API Gateway running on port 3000');
});
```

---

## Distributed System Patterns

### Circuit Breaker

```javascript
// circuit-breaker.js
class CircuitBreaker {
  constructor(options = {}) {
    this.failureThreshold = options.failureThreshold || 5;
    this.successThreshold = options.successThreshold || 2;
    this.timeout = options.timeout || 30000;

    this.state = 'CLOSED';
    this.failures = 0;
    this.successes = 0;
    this.lastFailureTime = null;
  }

  async execute(fn) {
    if (this.state === 'OPEN') {
      // Check if timeout has passed
      if (Date.now() - this.lastFailureTime >= this.timeout) {
        this.state = 'HALF-OPEN';
      } else {
        throw new Error('Circuit breaker is OPEN');
      }
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  onSuccess() {
    if (this.state === 'HALF-OPEN') {
      this.successes++;
      if (this.successes >= this.successThreshold) {
        this.reset();
      }
    } else {
      this.failures = 0;
    }
  }

  onFailure() {
    this.failures++;
    this.lastFailureTime = Date.now();

    if (this.failures >= this.failureThreshold) {
      this.state = 'OPEN';
      console.log('Circuit breaker OPENED');
    }
  }

  reset() {
    this.state = 'CLOSED';
    this.failures = 0;
    this.successes = 0;
    console.log('Circuit breaker RESET');
  }

  getState() {
    return {
      state: this.state,
      failures: this.failures,
      successes: this.successes
    };
  }
}

// Usage
const paymentCircuit = new CircuitBreaker({
  failureThreshold: 3,
  timeout: 60000
});

async function processPayment(paymentData) {
  return paymentCircuit.execute(async () => {
    return await paymentGateway.charge(paymentData);
  });
}
```

### Saga Pattern (Distributed Transactions)

```javascript
// saga-orchestrator.js
class SagaOrchestrator {
  constructor() {
    this.steps = [];
  }

  addStep(name, execute, compensate) {
    this.steps.push({ name, execute, compensate });
    return this;
  }

  async execute(context) {
    const completedSteps = [];

    try {
      for (const step of this.steps) {
        console.log(`Executing step: ${step.name}`);
        const result = await step.execute(context);
        completedSteps.push({ step, result });
        context = { ...context, [step.name]: result };
      }

      return { success: true, context };
    } catch (error) {
      console.error(`Step failed: ${error.message}`);

      // Compensate in reverse order
      for (const { step, result } of completedSteps.reverse()) {
        try {
          console.log(`Compensating step: ${step.name}`);
          await step.compensate(context, result);
        } catch (compensateError) {
          console.error(`Compensation failed for ${step.name}:`, compensateError);
          // Log for manual intervention
        }
      }

      return { success: false, error: error.message };
    }
  }
}

// Order creation saga
const createOrderSaga = new SagaOrchestrator()
  .addStep(
    'reserveInventory',
    async (ctx) => {
      const reservation = await inventoryService.reserve(ctx.items);
      return reservation;
    },
    async (ctx, result) => {
      await inventoryService.releaseReservation(result.reservationId);
    }
  )
  .addStep(
    'processPayment',
    async (ctx) => {
      const payment = await paymentService.charge({
        userId: ctx.userId,
        amount: ctx.total
      });
      return payment;
    },
    async (ctx, result) => {
      await paymentService.refund(result.paymentId);
    }
  )
  .addStep(
    'createOrder',
    async (ctx) => {
      const order = await orderService.create({
        userId: ctx.userId,
        items: ctx.items,
        paymentId: ctx.processPayment.paymentId,
        reservationId: ctx.reserveInventory.reservationId
      });
      return order;
    },
    async (ctx, result) => {
      await orderService.cancel(result.orderId);
    }
  )
  .addStep(
    'sendConfirmation',
    async (ctx) => {
      await notificationService.sendOrderConfirmation({
        userId: ctx.userId,
        orderId: ctx.createOrder.orderId
      });
      return { sent: true };
    },
    async () => {
      // No compensation needed for notifications
    }
  );

// Usage
async function placeOrder(orderRequest) {
  const result = await createOrderSaga.execute({
    userId: orderRequest.userId,
    items: orderRequest.items,
    total: orderRequest.total
  });

  if (!result.success) {
    throw new Error(`Order failed: ${result.error}`);
  }

  return result.context.createOrder;
}
```

### Service Discovery

```javascript
// service-discovery.js
const Consul = require('consul');

class ServiceRegistry {
  constructor() {
    this.consul = new Consul({
      host: process.env.CONSUL_HOST || 'localhost',
      port: process.env.CONSUL_PORT || 8500
    });
    this.cache = new Map();
    this.cacheTimeout = 30000; // 30 seconds
  }

  async register(serviceName, port, healthCheck) {
    const serviceId = `${serviceName}-${process.env.HOSTNAME || 'local'}-${port}`;

    await this.consul.agent.service.register({
      id: serviceId,
      name: serviceName,
      port,
      check: {
        http: healthCheck,
        interval: '10s',
        timeout: '5s'
      }
    });

    // Deregister on shutdown
    process.on('SIGINT', async () => {
      await this.consul.agent.service.deregister(serviceId);
      process.exit();
    });

    console.log(`Registered ${serviceName} with Consul`);
  }

  async discover(serviceName) {
    // Check cache first
    const cached = this.cache.get(serviceName);
    if (cached && Date.now() - cached.timestamp < this.cacheTimeout) {
      return cached.instances;
    }

    // Fetch from Consul
    const result = await this.consul.health.service({
      service: serviceName,
      passing: true
    });

    const instances = result.map(entry => ({
      id: entry.Service.ID,
      address: entry.Service.Address || entry.Node.Address,
      port: entry.Service.Port
    }));

    // Update cache
    this.cache.set(serviceName, {
      instances,
      timestamp: Date.now()
    });

    return instances;
  }

  // Simple round-robin load balancing
  async getServiceUrl(serviceName) {
    const instances = await this.discover(serviceName);

    if (instances.length === 0) {
      throw new Error(`No instances found for ${serviceName}`);
    }

    // Round-robin selection
    if (!this.roundRobinIndex) {
      this.roundRobinIndex = new Map();
    }

    const currentIndex = this.roundRobinIndex.get(serviceName) || 0;
    const instance = instances[currentIndex % instances.length];
    this.roundRobinIndex.set(serviceName, currentIndex + 1);

    return `http://${instance.address}:${instance.port}`;
  }
}

// Usage
const registry = new ServiceRegistry();

// Register this service
await registry.register('order-service', 3002, 'http://localhost:3002/health');

// Discover and call another service
async function callUserService(userId) {
  const baseUrl = await registry.getServiceUrl('user-service');
  const response = await fetch(`${baseUrl}/api/users/${userId}`);
  return response.json();
}
```

---

## Data Management

### Database per Service

```
┌─────────────────────────────────────────────────────────────────┐
│                  DATABASE PER SERVICE                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐          │
│  │User Service │    │Order Service│    │Product Svc  │          │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘          │
│         │                  │                  │                  │
│         ▼                  ▼                  ▼                  │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐          │
│  │ PostgreSQL  │    │  MongoDB    │    │ PostgreSQL  │          │
│  │   Users     │    │   Orders    │    │  Products   │          │
│  └─────────────┘    └─────────────┘    └─────────────┘          │
│                                                                  │
│  Benefits:                                                       │
│  ✓ Independent scaling                                          │
│  ✓ Technology flexibility                                       │
│  ✓ Isolated failures                                            │
│  ✓ Clear ownership                                              │
│                                                                  │
│  Challenges:                                                     │
│  ✗ Cross-service queries                                        │
│  ✗ Data consistency                                             │
│  ✗ Distributed transactions                                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Event Sourcing

```javascript
// event-store.js
class EventStore {
  constructor(db) {
    this.db = db;
  }

  async append(streamId, events, expectedVersion) {
    const client = await this.db.pool.connect();

    try {
      await client.query('BEGIN');

      // Check current version
      const result = await client.query(
        'SELECT MAX(version) as version FROM events WHERE stream_id = $1',
        [streamId]
      );

      const currentVersion = result.rows[0].version || 0;

      if (expectedVersion !== -1 && currentVersion !== expectedVersion) {
        throw new Error('Concurrency conflict');
      }

      // Append events
      let version = currentVersion;
      for (const event of events) {
        version++;
        await client.query(
          `INSERT INTO events (stream_id, version, type, data, timestamp)
           VALUES ($1, $2, $3, $4, NOW())`,
          [streamId, version, event.type, JSON.stringify(event.data)]
        );
      }

      await client.query('COMMIT');
      return version;
    } catch (error) {
      await client.query('ROLLBACK');
      throw error;
    } finally {
      client.release();
    }
  }

  async getEvents(streamId, fromVersion = 0) {
    const result = await this.db.pool.query(
      `SELECT * FROM events
       WHERE stream_id = $1 AND version > $2
       ORDER BY version`,
      [streamId, fromVersion]
    );

    return result.rows.map(row => ({
      type: row.type,
      data: row.data,
      version: row.version,
      timestamp: row.timestamp
    }));
  }
}

// Order aggregate with event sourcing
class Order {
  constructor(id) {
    this.id = id;
    this.status = null;
    this.items = [];
    this.total = 0;
    this.version = 0;
  }

  static async load(eventStore, orderId) {
    const order = new Order(orderId);
    const events = await eventStore.getEvents(`order-${orderId}`);

    for (const event of events) {
      order.apply(event);
    }

    return order;
  }

  apply(event) {
    switch (event.type) {
      case 'OrderCreated':
        this.status = 'created';
        this.items = event.data.items;
        this.total = event.data.total;
        break;
      case 'OrderPaid':
        this.status = 'paid';
        break;
      case 'OrderShipped':
        this.status = 'shipped';
        this.trackingNumber = event.data.trackingNumber;
        break;
      case 'OrderCancelled':
        this.status = 'cancelled';
        break;
    }
    this.version = event.version;
  }

  // Command handlers
  static create(orderId, items, total) {
    return [{
      type: 'OrderCreated',
      data: { orderId, items, total }
    }];
  }

  pay() {
    if (this.status !== 'created') {
      throw new Error('Order cannot be paid');
    }
    return [{ type: 'OrderPaid', data: {} }];
  }

  ship(trackingNumber) {
    if (this.status !== 'paid') {
      throw new Error('Order must be paid before shipping');
    }
    return [{
      type: 'OrderShipped',
      data: { trackingNumber }
    }];
  }
}

// Usage
const eventStore = new EventStore(db);

// Create order
const orderId = 'order-123';
const events = Order.create(orderId, items, total);
await eventStore.append(`order-${orderId}`, events, -1);

// Load and update order
const order = await Order.load(eventStore, orderId);
const payEvents = order.pay();
await eventStore.append(`order-${orderId}`, payEvents, order.version);
```

### CQRS (Command Query Responsibility Segregation)

```javascript
// cqrs-example.js

// Commands - Write side
class OrderCommandHandler {
  constructor(eventStore, eventBus) {
    this.eventStore = eventStore;
    this.eventBus = eventBus;
  }

  async handle(command) {
    switch (command.type) {
      case 'CreateOrder':
        return this.createOrder(command);
      case 'PayOrder':
        return this.payOrder(command);
      case 'ShipOrder':
        return this.shipOrder(command);
    }
  }

  async createOrder(command) {
    const events = Order.create(
      command.orderId,
      command.items,
      command.total
    );

    await this.eventStore.append(
      `order-${command.orderId}`,
      events,
      -1
    );

    // Publish events for read side
    for (const event of events) {
      await this.eventBus.publish('order.events', event);
    }
  }

  async payOrder(command) {
    const order = await Order.load(this.eventStore, command.orderId);
    const events = order.pay();

    await this.eventStore.append(
      `order-${command.orderId}`,
      events,
      order.version
    );

    for (const event of events) {
      await this.eventBus.publish('order.events', event);
    }
  }
}

// Queries - Read side (optimized read model)
class OrderQueryHandler {
  constructor(readDb) {
    this.db = readDb;
  }

  async getOrder(orderId) {
    return this.db.orders.findUnique({
      where: { id: orderId },
      include: { items: true }
    });
  }

  async getUserOrders(userId, pagination) {
    return this.db.orders.findMany({
      where: { userId },
      orderBy: { createdAt: 'desc' },
      take: pagination.limit,
      skip: pagination.offset
    });
  }

  async getOrderStats() {
    return this.db.orderStats.findFirst();
  }
}

// Read model updater (projection)
class OrderProjection {
  constructor(readDb, eventBus) {
    this.db = readDb;
    this.setupEventHandlers(eventBus);
  }

  setupEventHandlers(eventBus) {
    eventBus.subscribe('order.events', async (event) => {
      switch (event.type) {
        case 'OrderCreated':
          await this.onOrderCreated(event);
          break;
        case 'OrderPaid':
          await this.onOrderPaid(event);
          break;
        case 'OrderShipped':
          await this.onOrderShipped(event);
          break;
      }
    });
  }

  async onOrderCreated(event) {
    await this.db.orders.create({
      data: {
        id: event.data.orderId,
        status: 'created',
        items: event.data.items,
        total: event.data.total,
        createdAt: new Date()
      }
    });

    // Update stats
    await this.db.orderStats.upsert({
      where: { id: 'stats' },
      update: {
        totalOrders: { increment: 1 },
        totalRevenue: { increment: event.data.total }
      },
      create: {
        id: 'stats',
        totalOrders: 1,
        totalRevenue: event.data.total
      }
    });
  }

  async onOrderPaid(event) {
    await this.db.orders.update({
      where: { id: event.data.orderId },
      data: { status: 'paid', paidAt: new Date() }
    });
  }

  async onOrderShipped(event) {
    await this.db.orders.update({
      where: { id: event.data.orderId },
      data: {
        status: 'shipped',
        trackingNumber: event.data.trackingNumber,
        shippedAt: new Date()
      }
    });
  }
}
```

---

## Deployment

### Docker Compose for Development

```yaml
# docker-compose.yml
version: '3.8'

services:
  # API Gateway
  gateway:
    build: ./gateway
    ports:
      - "3000:3000"
    environment:
      - USER_SERVICE_URL=http://user-service:3001
      - ORDER_SERVICE_URL=http://order-service:3002
      - PRODUCT_SERVICE_URL=http://product-service:3003
    depends_on:
      - user-service
      - order-service
      - product-service

  # User Service
  user-service:
    build: ./user-service
    ports:
      - "3001:3001"
    environment:
      - DATABASE_URL=postgres://postgres:password@user-db:5432/users
      - REDIS_URL=redis://redis:6379
    depends_on:
      - user-db
      - redis

  user-db:
    image: postgres:15
    environment:
      POSTGRES_DB: users
      POSTGRES_PASSWORD: password
    volumes:
      - user-data:/var/lib/postgresql/data

  # Order Service
  order-service:
    build: ./order-service
    ports:
      - "3002:3002"
    environment:
      - DATABASE_URL=postgres://postgres:password@order-db:5432/orders
      - REDIS_URL=redis://redis:6379
      - USER_SERVICE_URL=http://user-service:3001
    depends_on:
      - order-db
      - redis

  order-db:
    image: postgres:15
    environment:
      POSTGRES_DB: orders
      POSTGRES_PASSWORD: password
    volumes:
      - order-data:/var/lib/postgresql/data

  # Product Service
  product-service:
    build: ./product-service
    ports:
      - "3003:3003"
    environment:
      - DATABASE_URL=postgres://postgres:password@product-db:5432/products
      - REDIS_URL=redis://redis:6379

  product-db:
    image: postgres:15
    environment:
      POSTGRES_DB: products
      POSTGRES_PASSWORD: password
    volumes:
      - product-data:/var/lib/postgresql/data

  # Shared Redis
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  user-data:
  order-data:
  product-data:
```

### Kubernetes Deployment

```yaml
# k8s/user-service.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
  labels:
    app: user-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: user-service
  template:
    metadata:
      labels:
        app: user-service
    spec:
      containers:
        - name: user-service
          image: myregistry/user-service:v1.0.0
          ports:
            - containerPort: 3001
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: user-service-secrets
                  key: database-url
            - name: REDIS_URL
              valueFrom:
                configMapKeyRef:
                  name: shared-config
                  key: redis-url
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /health
              port: 3001
            initialDelaySeconds: 10
            periodSeconds: 5
          readinessProbe:
            httpGet:
              path: /ready
              port: 3001
            initialDelaySeconds: 5
            periodSeconds: 3

---
apiVersion: v1
kind: Service
metadata:
  name: user-service
spec:
  selector:
    app: user-service
  ports:
    - port: 3001
      targetPort: 3001
  type: ClusterIP

---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: user-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: user-service
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

---

## Practice Exercises

### Exercise 1: Service Decomposition

Take a monolith e-commerce app and:
- Identify bounded contexts
- Design service APIs
- Plan data ownership
- Define integration events

### Exercise 2: Event-Driven Integration

Build two services that communicate via events:
- Order service publishes order events
- Inventory service subscribes and updates stock
- Handle failure scenarios

### Exercise 3: API Gateway

Create an API gateway that:
- Routes to multiple services
- Handles authentication
- Implements rate limiting
- Aggregates responses

---

## Key Takeaways

1. **Start with monolith** - Decompose when you understand the domain
2. **Design around business capabilities** - Not technical layers
3. **Own your data** - Each service owns its database
4. **Prefer events over calls** - Loose coupling, better resilience
5. **Embrace eventual consistency** - Not everything needs transactions
6. **Invest in observability** - Distributed tracing, centralized logging

---

## What's Next?

Tomorrow and the day after, we'll work through **System Design Examples** - applying everything we've learned to design real-world systems like URL shorteners, chat applications, and social media feeds.
