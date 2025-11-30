# Day 2: Database Scaling - Handling Massive Data

## Introduction

The database is often the hardest component to scale. Unlike stateless application servers that can be easily replicated, databases hold state and must maintain consistency. Today, you'll learn strategies for scaling databases including replication, sharding, and choosing the right database for your needs.

## Learning Objectives

By the end of this lesson, you will be able to:
- Implement read replicas for scaling reads
- Design sharding strategies
- Understand CAP theorem trade-offs
- Choose between SQL and NoSQL
- Optimize database performance

---

## Database Scaling Overview

### The Database Bottleneck

```
┌─────────────────────────────────────────────────────────────────┐
│                    Why Databases Are Hard to Scale               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Application Servers              Database                      │
│  ┌───┐ ┌───┐ ┌───┐               ┌─────────┐                   │
│  │ A │ │ A │ │ A │  ────────────►│   DB    │◄── Single point  │
│  └───┘ └───┘ └───┘               │ (state) │    of failure    │
│     ↑     ↑     ↑                └─────────┘                   │
│     Stateless (easy)              Stateful (hard)              │
│                                                                  │
│  Challenges:                                                     │
│  • Data consistency across copies                               │
│  • ACID transactions                                            │
│  • Joins across partitions                                      │
│  • Schema changes                                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Scaling Strategies

```
┌─────────────────────────────────────────────────────────────────┐
│                   Database Scaling Strategies                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Optimize First                                              │
│     └── Indexes, query optimization, connection pooling         │
│                                                                  │
│  2. Vertical Scaling                                            │
│     └── Bigger server, more RAM, faster SSD                     │
│                                                                  │
│  3. Read Replicas                                               │
│     └── Offload reads to replica servers                        │
│                                                                  │
│  4. Caching                                                      │
│     └── Redis/Memcached for frequent reads                      │
│                                                                  │
│  5. Sharding (Horizontal Partitioning)                          │
│     └── Split data across multiple databases                    │
│                                                                  │
│  6. Different Database                                          │
│     └── NoSQL for specific use cases                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Read Replicas

### Master-Replica Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Read Replica Pattern                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│              Application                                        │
│                  │                                               │
│          ┌──────┴──────┐                                        │
│          │             │                                         │
│       Writes        Reads                                       │
│          │             │                                         │
│          ▼             ▼                                         │
│    ┌─────────┐   ┌─────────────────────────┐                   │
│    │ Primary │   │    Read Replicas        │                   │
│    │ (Write) │──►│ ┌───────┐ ┌───────┐    │                   │
│    └─────────┘   │ │Replica│ │Replica│    │                   │
│         │        │ │   1   │ │   2   │    │                   │
│         │        │ └───────┘ └───────┘    │                   │
│         │        └─────────────────────────┘                   │
│         │                                                       │
│    Replication (async)                                         │
│                                                                  │
│  Benefits:                                                       │
│  • Scale read capacity                                          │
│  • Read from nearest replica                                    │
│  • Primary failover to replica                                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Implementation with Prisma

```javascript
// prisma/schema.prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// Read replica connection
const { PrismaClient } = require('@prisma/client');

// Primary for writes
const primaryDb = new PrismaClient({
  datasources: {
    db: { url: process.env.DATABASE_URL }
  }
});

// Replica for reads
const replicaDb = new PrismaClient({
  datasources: {
    db: { url: process.env.DATABASE_REPLICA_URL }
  }
});

// Usage
async function getUsers() {
  // Read from replica
  return replicaDb.user.findMany();
}

async function createUser(data) {
  // Write to primary
  return primaryDb.user.create({ data });
}
```

### Replication Lag

```
┌─────────────────────────────────────────────────────────────────┐
│                    Replication Lag                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Primary                Replica                                 │
│  ┌───────────┐         ┌───────────┐                           │
│  │ Insert X  │ ──────► │           │  Lag: 100ms               │
│  │ at T=0    │  async  │ Insert X  │                           │
│  └───────────┘         │ at T=100  │                           │
│                        └───────────┘                           │
│                                                                  │
│  Problem: Read-your-writes inconsistency                        │
│  ─────────────────────────────────────────                      │
│  1. User creates post (writes to primary)                       │
│  2. User refreshes page (reads from replica)                    │
│  3. Post not visible yet (replication lag)                      │
│                                                                  │
│  Solutions:                                                      │
│  • Read from primary for user's own data                        │
│  • Include timestamp, only read from replica if caught up       │
│  • Accept eventual consistency for some reads                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Database Sharding

### Sharding Concept

```
┌─────────────────────────────────────────────────────────────────┐
│                    Database Sharding                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Single Database (100M users)                                   │
│  ┌─────────────────────────────────────────┐                   │
│  │              All Users                   │  ← Too big!      │
│  └─────────────────────────────────────────┘                   │
│                                                                  │
│                       ↓ Shard                                   │
│                                                                  │
│  Shard 1          Shard 2          Shard 3                     │
│  ┌───────────┐   ┌───────────┐   ┌───────────┐                │
│  │ Users 1-33M│  │Users 33-66M│  │Users 66-100M│              │
│  └───────────┘   └───────────┘   └───────────┘                │
│                                                                  │
│  Each shard:                                                     │
│  • Independent database                                         │
│  • Holds subset of data                                         │
│  • Can have own replicas                                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Sharding Strategies

```
┌─────────────────────────────────────────────────────────────────┐
│                    Sharding Strategies                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Range-Based Sharding                                        │
│     ────────────────────────────                                │
│     userId 1-1M     → Shard A                                   │
│     userId 1M-2M    → Shard B                                   │
│     userId 2M-3M    → Shard C                                   │
│                                                                  │
│     Pros: Range queries easy                                    │
│     Cons: Hotspots (new users → last shard)                    │
│                                                                  │
│  2. Hash-Based Sharding                                         │
│     ────────────────────────────                                │
│     hash(userId) % 3 = 0 → Shard A                             │
│     hash(userId) % 3 = 1 → Shard B                             │
│     hash(userId) % 3 = 2 → Shard C                             │
│                                                                  │
│     Pros: Even distribution                                     │
│     Cons: Range queries span all shards                        │
│                                                                  │
│  3. Directory-Based Sharding                                    │
│     ────────────────────────────                                │
│     Lookup table: userId → Shard                               │
│                                                                  │
│     Pros: Flexible placement                                    │
│     Cons: Lookup overhead, SPOF                                │
│                                                                  │
│  4. Geographic Sharding                                         │
│     ────────────────────────────                                │
│     US users → US shard                                        │
│     EU users → EU shard                                        │
│                                                                  │
│     Pros: Low latency, data locality                           │
│     Cons: Cross-region queries                                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Sharding Implementation

```javascript
// Simple hash-based sharding
class ShardRouter {
  constructor(shards) {
    this.shards = shards;
    this.numShards = shards.length;
  }

  getShard(userId) {
    const hash = this.hashCode(userId.toString());
    const shardIndex = Math.abs(hash) % this.numShards;
    return this.shards[shardIndex];
  }

  hashCode(str) {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      const char = str.charCodeAt(i);
      hash = ((hash << 5) - hash) + char;
      hash = hash & hash;
    }
    return hash;
  }
}

// Usage
const shards = [
  new PrismaClient({ datasources: { db: { url: process.env.SHARD_0_URL } } }),
  new PrismaClient({ datasources: { db: { url: process.env.SHARD_1_URL } } }),
  new PrismaClient({ datasources: { db: { url: process.env.SHARD_2_URL } } })
];

const router = new ShardRouter(shards);

async function getUser(userId) {
  const shard = router.getShard(userId);
  return shard.user.findUnique({ where: { id: userId } });
}
```

### Sharding Challenges

```
┌─────────────────────────────────────────────────────────────────┐
│                    Sharding Challenges                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Cross-Shard Queries                                            │
│  ──────────────────                                             │
│  "Find all users with orders > $1000"                          │
│  Must query ALL shards, merge results                          │
│                                                                  │
│  Cross-Shard Joins                                              │
│  ─────────────────                                              │
│  User in Shard A, Orders in Shard B                            │
│  Join at application level (slow)                              │
│                                                                  │
│  Cross-Shard Transactions                                       │
│  ────────────────────────                                       │
│  ACID across shards = distributed transaction                  │
│  Very complex, consider eventual consistency                    │
│                                                                  │
│  Resharding                                                      │
│  ──────────                                                     │
│  Adding/removing shards requires data migration                │
│  Use consistent hashing to minimize movement                   │
│                                                                  │
│  Hotspots                                                        │
│  ────────                                                       │
│  One shard gets more traffic (celebrity user)                  │
│  Solution: Further split hot shard                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## CAP Theorem

### Understanding CAP

```
┌─────────────────────────────────────────────────────────────────┐
│                      CAP Theorem                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│                    Consistency                                   │
│                        /\                                        │
│                       /  \                                       │
│                      /    \                                      │
│                     / CA   \                                     │
│                    /        \                                    │
│                   /──────────\                                   │
│                  /            \                                  │
│                 / CP        AP \                                 │
│                /                \                                │
│               ────────────────────                              │
│         Partition          Availability                         │
│         Tolerance                                               │
│                                                                  │
│  C (Consistency): All nodes see same data at same time         │
│  A (Availability): Every request gets a response               │
│  P (Partition Tolerance): System works despite network splits  │
│                                                                  │
│  In distributed systems, P is required.                         │
│  Choose between C and A during partitions.                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Database Classifications

```
┌─────────────────────────────────────────────────────────────────┐
│                   Database CAP Trade-offs                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  CP (Consistency + Partition Tolerance)                         │
│  ──────────────────────────────────────                         │
│  • PostgreSQL, MySQL (with sync replication)                    │
│  • MongoDB (with majority write concern)                        │
│  • HBase, Redis Cluster                                         │
│  → Unavailable during partition                                 │
│                                                                  │
│  AP (Availability + Partition Tolerance)                        │
│  ──────────────────────────────────────                         │
│  • Cassandra, DynamoDB                                          │
│  • CouchDB                                                      │
│  • Eventual consistency                                         │
│  → May return stale data during partition                       │
│                                                                  │
│  CA (Consistency + Availability)                                │
│  ──────────────────────────────                                 │
│  • Traditional single-node RDBMS                                │
│  → Not partition tolerant (single point of failure)            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## SQL vs NoSQL

### Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                    SQL vs NoSQL                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  SQL (Relational)              NoSQL (Non-relational)           │
│  ─────────────────             ────────────────────             │
│  Fixed schema                  Flexible schema                   │
│  ACID transactions             BASE (eventual consistency)       │
│  Complex queries (JOIN)        Simple queries                    │
│  Vertical scaling              Horizontal scaling                │
│  Strong consistency            Eventual consistency              │
│                                                                  │
│  Use SQL for:                  Use NoSQL for:                   │
│  • Complex relationships       • High write throughput           │
│  • ACID requirements           • Flexible/evolving schema        │
│  • Complex queries             • Horizontal scaling              │
│  • Structured data             • Document/key-value data        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### NoSQL Types

```
┌─────────────────────────────────────────────────────────────────┐
│                    NoSQL Database Types                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Document Stores                                                │
│  ───────────────                                                │
│  MongoDB, CouchDB                                               │
│  { "name": "John", "orders": [...] }                           │
│  Use case: Content management, user profiles                    │
│                                                                  │
│  Key-Value Stores                                               │
│  ────────────────                                               │
│  Redis, DynamoDB                                                │
│  "user:123" → serialized data                                  │
│  Use case: Sessions, caching, real-time data                    │
│                                                                  │
│  Wide-Column Stores                                             │
│  ──────────────────                                             │
│  Cassandra, HBase                                               │
│  Row key → column families                                      │
│  Use case: Time-series, IoT, analytics                         │
│                                                                  │
│  Graph Databases                                                │
│  ───────────────                                                │
│  Neo4j, Amazon Neptune                                          │
│  Nodes and edges                                                │
│  Use case: Social networks, recommendations                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Database Optimization

### Indexing

```sql
-- Find slow queries
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 123;

-- Create index
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- Composite index
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

-- Partial index
CREATE INDEX idx_active_users ON users(email) WHERE active = true;

-- Covering index (includes all needed columns)
CREATE INDEX idx_orders_user_id_amount ON orders(user_id) INCLUDE (amount, status);
```

### Connection Pooling

```javascript
// Without pooling: New connection per request (slow)
// With pooling: Reuse connections (fast)

// PgBouncer config
[databases]
mydb = host=localhost dbname=mydb

[pgbouncer]
listen_port = 6432
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 20
```

```javascript
// Prisma connection pooling
const prisma = new PrismaClient({
  datasources: {
    db: {
      url: process.env.DATABASE_URL + '?connection_limit=20&pool_timeout=10'
    }
  }
});
```

### Query Optimization

```javascript
// BAD: N+1 query
const users = await prisma.user.findMany();
for (const user of users) {
  const orders = await prisma.order.findMany({
    where: { userId: user.id }
  });
}

// GOOD: Include related data
const users = await prisma.user.findMany({
  include: { orders: true }
});

// GOOD: Select only needed fields
const users = await prisma.user.findMany({
  select: {
    id: true,
    name: true,
    email: true
  }
});

// GOOD: Pagination
const users = await prisma.user.findMany({
  take: 20,
  skip: 40,
  orderBy: { createdAt: 'desc' }
});
```

---

## Practice Exercises

### Exercise 1: Design Read Replicas

```
Design a read replica setup for an e-commerce app:
- Product catalog (read-heavy)
- Order processing (write-heavy)
- User sessions

Which queries go to primary vs replica?
```

### Exercise 2: Sharding Strategy

```
Design sharding for a social media app with:
- 100M users
- Users have posts, likes, follows
- Must support: User profile, User's posts, News feed

What's your sharding key? Why?
```

### Exercise 3: Database Selection

```
Choose the right database for:
1. User sessions (fast lookup by session ID)
2. Product catalog (complex queries, relationships)
3. Activity feed (high write volume, time-ordered)
4. Social graph (friends of friends queries)
```

---

## Quick Reference

### Scaling Strategies

| Strategy | Scales | Complexity |
|----------|--------|------------|
| Optimize | 2-3x | Low |
| Vertical | 5-10x | Low |
| Read replicas | 10-20x reads | Medium |
| Caching | 50-100x reads | Medium |
| Sharding | Unlimited | High |

### Database Selection Guide

| Use Case | Recommended |
|----------|-------------|
| Complex relationships | PostgreSQL |
| High write throughput | Cassandra, DynamoDB |
| Sessions/cache | Redis |
| Documents | MongoDB |
| Time series | TimescaleDB, InfluxDB |
| Graph queries | Neo4j |

---

## Key Takeaways

1. **Optimize before scaling** - indexes, queries, pooling
2. **Read replicas** for read-heavy workloads
3. **Sharding** for write scaling and data size
4. **CAP theorem** - can't have all three
5. **Right database** for the job
6. **Accept trade-offs** - consistency vs availability

---

## What's Next?

Tomorrow, we'll explore **Caching** - using Redis and CDNs to dramatically reduce database load and improve response times.
