# System Design

**Duration:** 2-3 weeks

## Learning Objectives

By the end of this section, you will:
- Understand scalability concepts
- Design systems that handle millions of users
- Make informed architectural decisions
- Communicate designs effectively

---

## Week 1: Fundamentals

### Scalability Basics

**Vertical Scaling (Scale Up)**
- Add more power to existing machine
- Limited by hardware
- Simple but has ceiling

**Horizontal Scaling (Scale Out)**
- Add more machines
- More complex but unlimited
- Requires stateless design

### Key Concepts

**Load Balancer**
```
                    ┌─────────────┐
                    │   Client    │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │Load Balancer│
                    └──────┬──────┘
              ┌────────────┼────────────┐
              │            │            │
        ┌─────▼─────┐┌─────▼─────┐┌─────▼─────┐
        │  Server 1 ││  Server 2 ││  Server 3 │
        └───────────┘└───────────┘└───────────┘
```

**Database Replication**
```
        ┌─────────────┐
        │   Primary   │ (writes)
        └──────┬──────┘
        ┌──────┴──────┐
        │ Replication │
        ├──────┬──────┤
   ┌────▼───┐ ┌▼────┐ ┌▼────┐
   │Replica1│ │Rep 2│ │Rep 3│ (reads)
   └────────┘ └─────┘ └─────┘
```

**Caching Layers**
```
Client → CDN → Application Cache → Database Cache → Database
```

### Database Scaling

**Read Replicas**
- Primary for writes
- Replicas for reads
- Eventually consistent

**Database Sharding**
```
User ID 1-1000     → Shard 1
User ID 1001-2000  → Shard 2
User ID 2001-3000  → Shard 3
```

**Partitioning Strategies**
- Range-based (by date, ID range)
- Hash-based (consistent hashing)
- Geographic (by region)

---

## Week 2: Architecture Patterns

### Monolith vs Microservices

**Monolith**
```
┌────────────────────────────┐
│        Application         │
│  ┌────┐ ┌────┐ ┌────────┐ │
│  │Auth│ │Cart│ │Products│  │
│  └────┘ └────┘ └────────┘ │
│         ┌──────┐          │
│         │  DB  │          │
│         └──────┘          │
└────────────────────────────┘

Pros: Simple, easy to deploy
Cons: Hard to scale independently
```

**Microservices**
```
┌──────┐ ┌──────┐ ┌──────────┐
│ Auth │ │ Cart │ │ Products │
│  DB  │ │  DB  │ │    DB    │
└──┬───┘ └──┬───┘ └────┬─────┘
   │        │          │
   └────────┼──────────┘
            │
      ┌─────▼─────┐
      │  Gateway  │
      └───────────┘

Pros: Scale independently, tech flexibility
Cons: Complex, network overhead
```

### Message Queues

**Use Cases**
- Background jobs
- Decoupling services
- Rate limiting
- Event-driven architecture

**Popular Options**
- Redis (simple)
- RabbitMQ (complex routing)
- Kafka (high throughput)

```
┌─────────┐    ┌───────┐    ┌──────────┐
│Producer │───▶│ Queue │───▶│ Consumer │
└─────────┘    └───────┘    └──────────┘

Example: Order Processing
User → API → Queue → Email Service
                  → Inventory Service
                  → Analytics Service
```

### Caching Strategies

**Cache-Aside (Lazy Loading)**
```typescript
async function getUser(id: string) {
  // Check cache first
  const cached = await redis.get(`user:${id}`)
  if (cached) return JSON.parse(cached)

  // Cache miss - fetch from DB
  const user = await db.user.findUnique({ where: { id } })

  // Store in cache
  await redis.set(`user:${id}`, JSON.stringify(user), 'EX', 3600)

  return user
}
```

**Write-Through**
```typescript
async function updateUser(id: string, data: UserData) {
  // Update DB
  const user = await db.user.update({ where: { id }, data })

  // Update cache
  await redis.set(`user:${id}`, JSON.stringify(user), 'EX', 3600)

  return user
}
```

**Cache Invalidation**
```typescript
async function updateUser(id: string, data: UserData) {
  const user = await db.user.update({ where: { id }, data })

  // Invalidate cache
  await redis.del(`user:${id}`)

  return user
}
```

---

## Week 3: Design Practice

### URL Shortener Design

**Requirements**
- Shorten long URLs
- Redirect to original
- Analytics (optional)

**Design**
```
┌────────┐   ┌─────────┐   ┌────────┐
│ Client │──▶│   API   │──▶│  Redis │ (cache)
└────────┘   └────┬────┘   └────────┘
                  │
             ┌────▼────┐
             │   DB    │
             └─────────┘

Tables:
- urls (id, short_code, original_url, created_at)
- clicks (id, url_id, timestamp, ip, user_agent)

Short Code Generation:
- Base62 encoding (a-z, A-Z, 0-9)
- 6 characters = 62^6 = 56 billion combinations
```

### Chat Application Design

**Requirements**
- Real-time messaging
- Group chats
- Message history
- Online status

**Design**
```
┌────────┐   ┌──────────────┐   ┌─────────┐
│ Client │◀─▶│ WebSocket    │──▶│  Redis  │ (pub/sub)
└────────┘   │   Server     │   └─────────┘
             └──────┬───────┘
                    │
             ┌──────▼───────┐
             │  PostgreSQL  │ (messages)
             └──────────────┘

Components:
- WebSocket for real-time
- Redis Pub/Sub for scaling
- PostgreSQL for persistence
- CDN for media
```

### E-commerce Design

**Requirements**
- Product catalog
- Shopping cart
- Order processing
- Inventory management

**Design**
```
                    ┌───────────┐
                    │    CDN    │ (images)
                    └─────┬─────┘
                          │
┌────────┐   ┌────────────▼────────────┐
│ Client │──▶│      API Gateway        │
└────────┘   └───┬────────┬───────┬────┘
                 │        │       │
          ┌──────▼──┐ ┌───▼───┐ ┌─▼─────┐
          │Products │ │ Cart  │ │Orders │
          │   DB    │ │ Redis │ │  DB   │
          └─────────┘ └───────┘ └───────┘
                                    │
                              ┌─────▼─────┐
                              │   Queue   │
                              └─────┬─────┘
                      ┌─────────────┼─────────────┐
                      │             │             │
                ┌─────▼────┐ ┌─────▼────┐ ┌──────▼─────┐
                │ Payment  │ │Inventory │ │   Email    │
                └──────────┘ └──────────┘ └────────────┘
```

---

## Design Interview Template

### 1. Clarify Requirements (5 min)
- What's the scale? (users, requests/sec)
- What features are must-have?
- What can be eventual consistent?

### 2. High-Level Design (10 min)
- Draw main components
- Show data flow
- Identify databases

### 3. Deep Dive (15 min)
- Database schema
- API design
- Scaling strategies
- Bottlenecks and solutions

### 4. Wrap Up (5 min)
- Trade-offs made
- Future improvements
- Monitoring strategy

---

## Resources

- [System Design Primer](https://github.com/donnemartin/system-design-primer)
- [Designing Data-Intensive Applications](https://dataintensive.net/)
- [ByteByteGo Newsletter](https://blog.bytebytego.com/)

---

## Checklist Before Moving On

- [ ] Understand horizontal vs vertical scaling
- [ ] Know database scaling strategies
- [ ] Understand caching patterns
- [ ] Can design basic systems
- [ ] Can communicate designs clearly

---

**Next:** [Performance Optimization](../02-performance/README.md)
