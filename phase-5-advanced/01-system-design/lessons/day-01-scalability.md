# Day 1: Scalability Fundamentals - Building Systems That Grow

## Introduction

Scalability is the ability of a system to handle increased load by adding resources. As your application grows from hundreds to millions of users, understanding scalability becomes critical. Today, you'll learn the fundamental concepts, patterns, and strategies for building scalable systems.

## Learning Objectives

By the end of this lesson, you will be able to:
- Understand vertical vs horizontal scaling
- Identify scalability bottlenecks
- Design stateless applications
- Implement load balancing strategies
- Apply scalability patterns

---

## Scalability Basics

### What is Scalability?

```
┌─────────────────────────────────────────────────────────────────┐
│                    Scalability Dimensions                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Load Scalability                                               │
│  └── Handle more requests per second                            │
│                                                                  │
│  Data Scalability                                               │
│  └── Store and process more data                                │
│                                                                  │
│  Geographic Scalability                                         │
│  └── Serve users across regions                                 │
│                                                                  │
│  Administrative Scalability                                     │
│  └── Manage growing teams and services                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Vertical vs Horizontal Scaling

```
┌─────────────────────────────────────────────────────────────────┐
│              Vertical Scaling (Scale Up)                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Before:           After:                                        │
│  ┌─────────┐       ┌─────────────┐                              │
│  │ 4 CPU   │  →    │ 16 CPU      │                              │
│  │ 8GB RAM │       │ 64GB RAM    │                              │
│  │ 100GB   │       │ 1TB SSD     │                              │
│  └─────────┘       └─────────────┘                              │
│                                                                  │
│  Pros:                          Cons:                           │
│  • Simple                       • Hardware limits               │
│  • No code changes              • Single point of failure       │
│  • Works for any app            • Expensive at high end         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│              Horizontal Scaling (Scale Out)                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Before:           After:                                        │
│  ┌─────────┐       ┌─────────┐ ┌─────────┐ ┌─────────┐         │
│  │ Server  │  →    │ Server  │ │ Server  │ │ Server  │         │
│  └─────────┘       └─────────┘ └─────────┘ └─────────┘         │
│                           │         │         │                  │
│                           └─────────┴─────────┘                  │
│                                  │                               │
│                          Load Balancer                          │
│                                                                  │
│  Pros:                          Cons:                           │
│  • Unlimited scaling            • Complex architecture          │
│  • Redundancy/failover          • State management              │
│  • Cost-effective               • Data consistency              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### When to Scale

```
┌─────────────────────────────────────────────────────────────────┐
│                    Scaling Indicators                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Response Time                                                   │
│  └── P95 latency > 500ms                                        │
│                                                                  │
│  CPU Usage                                                       │
│  └── Sustained > 70%                                            │
│                                                                  │
│  Memory Usage                                                    │
│  └── Approaching limits, frequent GC                            │
│                                                                  │
│  Error Rate                                                      │
│  └── Increasing 5xx errors                                      │
│                                                                  │
│  Queue Depth                                                     │
│  └── Growing backlog                                            │
│                                                                  │
│  Database                                                        │
│  └── Connection pool exhaustion                                 │
│  └── Slow queries increasing                                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Stateless Architecture

### Why Stateless?

```
┌─────────────────────────────────────────────────────────────────┐
│                    Stateful vs Stateless                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  STATEFUL SERVER                                                │
│  ┌─────────────────┐                                            │
│  │  Server 1       │ ← User A's session stored here            │
│  │  [Session A]    │                                            │
│  └─────────────────┘                                            │
│         ↑                                                        │
│         │ Must route to same server (sticky sessions)           │
│         │                                                        │
│      User A                                                      │
│                                                                  │
│  Problems:                                                       │
│  • Server dies = session lost                                   │
│  • Can't easily add/remove servers                              │
│  • Uneven load distribution                                     │
│                                                                  │
│  ─────────────────────────────────────────────────────────────  │
│                                                                  │
│  STATELESS SERVER                                               │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐                           │
│  │Server 1 │ │Server 2 │ │Server 3 │  ← Any server works       │
│  └─────────┘ └─────────┘ └─────────┘                           │
│       │           │           │                                  │
│       └───────────┼───────────┘                                  │
│                   │                                              │
│            ┌──────┴──────┐                                      │
│            │   Redis     │  ← Shared session store              │
│            │  (Sessions) │                                      │
│            └─────────────┘                                      │
│                                                                  │
│  Benefits:                                                       │
│  • Any server handles any request                               │
│  • Easy to add/remove servers                                   │
│  • Better fault tolerance                                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Making Applications Stateless

```javascript
// BAD: Storing state in memory
const sessions = new Map();

app.post('/login', (req, res) => {
  const sessionId = uuid();
  sessions.set(sessionId, { userId: user.id }); // Lost on restart!
  res.cookie('sessionId', sessionId);
});

// GOOD: External session store (Redis)
const session = require('express-session');
const RedisStore = require('connect-redis').default;
const redis = require('redis');

const redisClient = redis.createClient({ url: process.env.REDIS_URL });

app.use(session({
  store: new RedisStore({ client: redisClient }),
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false
}));
```

```javascript
// BAD: In-memory cache
const cache = new Map();

async function getUser(id) {
  if (cache.has(id)) return cache.get(id);
  const user = await db.user.findUnique({ where: { id } });
  cache.set(id, user);
  return user;
}

// GOOD: Redis cache
const Redis = require('ioredis');
const redis = new Redis(process.env.REDIS_URL);

async function getUser(id) {
  const cached = await redis.get(`user:${id}`);
  if (cached) return JSON.parse(cached);

  const user = await db.user.findUnique({ where: { id } });
  await redis.setex(`user:${id}`, 3600, JSON.stringify(user));
  return user;
}
```

---

## Load Balancing

### Load Balancer Types

```
┌─────────────────────────────────────────────────────────────────┐
│                    Load Balancer Types                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Layer 4 (Transport)                                            │
│  └── Routes based on IP/port                                    │
│  └── Very fast, no content inspection                           │
│  └── Example: AWS NLB, HAProxy (TCP mode)                       │
│                                                                  │
│  Layer 7 (Application)                                          │
│  └── Routes based on HTTP content                               │
│  └── URL path, headers, cookies                                 │
│  └── SSL termination, caching                                   │
│  └── Example: Nginx, AWS ALB, HAProxy (HTTP mode)               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Load Balancing Algorithms

```
┌─────────────────────────────────────────────────────────────────┐
│                  Load Balancing Algorithms                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Round Robin                                                     │
│  └── Requests: 1→A, 2→B, 3→C, 4→A, 5→B...                      │
│  └── Simple, even distribution                                  │
│                                                                  │
│  Weighted Round Robin                                            │
│  └── Server A (weight 3): gets 3x traffic                       │
│  └── Server B (weight 1): gets 1x traffic                       │
│                                                                  │
│  Least Connections                                               │
│  └── Route to server with fewest active connections             │
│  └── Good for varying request duration                          │
│                                                                  │
│  IP Hash                                                         │
│  └── Same client IP → same server                               │
│  └── Useful for session affinity (if needed)                    │
│                                                                  │
│  Random                                                          │
│  └── Randomly select server                                     │
│  └── Simple, statistically even                                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Nginx Load Balancer Configuration

```nginx
# /etc/nginx/nginx.conf

upstream api_servers {
    # Round robin (default)
    server api1.example.com:3000;
    server api2.example.com:3000;
    server api3.example.com:3000;

    # Weighted
    server api1.example.com:3000 weight=3;
    server api2.example.com:3000 weight=1;

    # Least connections
    least_conn;

    # IP hash (sticky sessions)
    ip_hash;

    # Health checks
    server api1.example.com:3000 max_fails=3 fail_timeout=30s;
}

server {
    listen 80;

    location /api {
        proxy_pass http://api_servers;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

---

## Scalability Patterns

### Pattern 1: Service Replication

```
┌─────────────────────────────────────────────────────────────────┐
│                   Service Replication                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│                    Load Balancer                                │
│                         │                                        │
│         ┌───────────────┼───────────────┐                       │
│         │               │               │                        │
│         ▼               ▼               ▼                        │
│    ┌─────────┐    ┌─────────┐    ┌─────────┐                   │
│    │ App v1  │    │ App v1  │    │ App v1  │                   │
│    │ (copy)  │    │ (copy)  │    │ (copy)  │                   │
│    └─────────┘    └─────────┘    └─────────┘                   │
│                                                                  │
│  • All instances run same code                                  │
│  • Stateless (shared external state)                            │
│  • Easy to add/remove instances                                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Pattern 2: Functional Decomposition

```
┌─────────────────────────────────────────────────────────────────┐
│                Functional Decomposition                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Before (Monolith):                                             │
│  ┌─────────────────────────────────────────┐                   │
│  │  Users + Orders + Products + Payments   │                   │
│  └─────────────────────────────────────────┘                   │
│                                                                  │
│  After (Decomposed):                                            │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐              │
│  │ Users   │ │ Orders  │ │Products │ │Payments │              │
│  │ Service │ │ Service │ │ Service │ │ Service │              │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘              │
│       │           │           │           │                     │
│       └───────────┴───────────┴───────────┘                     │
│                       │                                          │
│               API Gateway                                       │
│                                                                  │
│  • Scale services independently                                 │
│  • Different tech stacks possible                               │
│  • Team autonomy                                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Pattern 3: Data Partitioning (Sharding)

```
┌─────────────────────────────────────────────────────────────────┐
│                    Data Partitioning                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Application                                                     │
│       │                                                          │
│       ▼                                                          │
│  Partition Router                                               │
│       │                                                          │
│       ├── Users A-M → Shard 1                                   │
│       ├── Users N-Z → Shard 2                                   │
│       │                                                          │
│  ┌─────────┐    ┌─────────┐                                    │
│  │ Shard 1 │    │ Shard 2 │                                    │
│  │  A-M    │    │  N-Z    │                                    │
│  └─────────┘    └─────────┘                                    │
│                                                                  │
│  Strategies:                                                     │
│  • Range-based (A-M, N-Z)                                       │
│  • Hash-based (hash(userId) % numShards)                        │
│  • Geographic (US → Shard 1, EU → Shard 2)                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Auto Scaling

### Auto Scaling Concepts

```
┌─────────────────────────────────────────────────────────────────┐
│                    Auto Scaling                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Metrics Watched:                                               │
│  • CPU utilization                                              │
│  • Memory usage                                                 │
│  • Request count                                                │
│  • Custom metrics                                               │
│                                                                  │
│  Scale Out (Add instances) when:                                │
│  └── CPU > 70% for 5 minutes                                    │
│                                                                  │
│  Scale In (Remove instances) when:                              │
│  └── CPU < 30% for 10 minutes                                   │
│                                                                  │
│  Cooldown Period:                                               │
│  └── Wait 5 minutes between scaling actions                     │
│                                                                  │
│  Instance Limits:                                               │
│  └── Min: 2, Max: 10, Desired: 4                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Kubernetes HPA Example

```yaml
# horizontal-pod-autoscaler.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
```

---

## Identifying Bottlenecks

### Common Bottlenecks

```
┌─────────────────────────────────────────────────────────────────┐
│                   Common Bottlenecks                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Database                                                        │
│  • Slow queries                                                 │
│  • Connection pool exhaustion                                   │
│  • Lock contention                                              │
│  Solution: Indexing, connection pooling, read replicas          │
│                                                                  │
│  Network                                                         │
│  • Bandwidth limits                                             │
│  • High latency                                                 │
│  Solution: CDN, compression, regional deployment                │
│                                                                  │
│  CPU                                                             │
│  • Heavy computation                                            │
│  • Inefficient algorithms                                       │
│  Solution: Optimize code, caching, horizontal scaling           │
│                                                                  │
│  Memory                                                          │
│  • Memory leaks                                                 │
│  • Large data in memory                                         │
│  Solution: Fix leaks, streaming, pagination                     │
│                                                                  │
│  I/O                                                             │
│  • Disk reads/writes                                            │
│  • External API calls                                           │
│  Solution: Caching, async processing, SSDs                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Profiling Tools

```javascript
// Node.js profiling
const v8 = require('v8');
const fs = require('fs');

// Heap snapshot
const heapSnapshot = v8.writeHeapSnapshot();
console.log(`Heap snapshot written to ${heapSnapshot}`);

// CPU profiling
const inspector = require('inspector');
const session = new inspector.Session();
session.connect();

session.post('Profiler.enable', () => {
  session.post('Profiler.start', () => {
    // Run your code

    session.post('Profiler.stop', (err, { profile }) => {
      fs.writeFileSync('profile.cpuprofile', JSON.stringify(profile));
    });
  });
});
```

---

## Practice Exercises

### Exercise 1: Design Stateless App

```
Design a stateless architecture for:
- User authentication
- Shopping cart
- Real-time notifications

What external stores would you use?
```

### Exercise 2: Load Balancer Config

```nginx
# Configure Nginx load balancer with:
# - 3 backend servers
# - Health checks
# - Least connections algorithm
# - SSL termination
```

### Exercise 3: Identify Bottlenecks

```
Given these symptoms, identify the bottleneck:
1. Response time increases linearly with users
2. Database connections maxed at 100
3. CPU at 30%, Memory at 40%

What's the bottleneck? How would you fix it?
```

---

## Quick Reference

### Scaling Comparison

| Type | Pros | Cons |
|------|------|------|
| Vertical | Simple, no code changes | Limited, expensive, SPOF |
| Horizontal | Unlimited, fault tolerant | Complex, state management |

### Checklist for Scalability

- [ ] Application is stateless
- [ ] Sessions in external store
- [ ] Cache in Redis/Memcached
- [ ] Database connection pooling
- [ ] Load balancer configured
- [ ] Health checks enabled
- [ ] Auto-scaling rules defined
- [ ] Monitoring in place

---

## Key Takeaways

1. **Prefer horizontal scaling** for web applications
2. **Make applications stateless** - store state externally
3. **Use load balancers** to distribute traffic
4. **Identify bottlenecks** before scaling
5. **Auto-scale** based on metrics
6. **Monitor** to know when to scale

---

## What's Next?

Tomorrow, we'll dive into **Database Scaling** - read replicas, sharding, and strategies for handling massive amounts of data.
