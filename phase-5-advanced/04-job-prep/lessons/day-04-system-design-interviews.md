# Day 4: System Design Interviews

## Introduction

System design interviews test your ability to design large-scale distributed systems. While traditionally reserved for senior roles, they're increasingly common at all levels. Even junior developers should understand basic system design concepts. Today, we'll cover the framework, core concepts, and practice with real design problems.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    System Design Interview Structure                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  45-60 MINUTE INTERVIEW                                                 │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │  0-5 min    │  Clarify requirements and scope                    │  │
│  ├─────────────┼────────────────────────────────────────────────────┤  │
│  │  5-15 min   │  High-level design / API design                    │  │
│  ├─────────────┼────────────────────────────────────────────────────┤  │
│  │  15-35 min  │  Deep dive into components                         │  │
│  ├─────────────┼────────────────────────────────────────────────────┤  │
│  │  35-45 min  │  Handle scale / discuss tradeoffs                  │  │
│  ├─────────────┼────────────────────────────────────────────────────┤  │
│  │  45-50 min  │  Wrap up and questions                             │  │
│  └─────────────┴────────────────────────────────────────────────────┘  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## Learning Objectives

By the end of this lesson, you will:
- Understand the system design interview framework
- Know essential system design concepts and components
- Practice with common system design questions
- Learn how to handle tradeoffs and scale

---

## 1. The RESHADE Framework

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    RESHADE Framework                                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  R - REQUIREMENTS (5 minutes)                                           │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Functional Requirements:                                        │   │
│  │  • What features must the system support?                        │   │
│  │  • What does the user journey look like?                         │   │
│  │                                                                  │   │
│  │  Non-Functional Requirements:                                    │   │
│  │  • Scale: How many users? How much data?                         │   │
│  │  • Performance: What latency is acceptable?                      │   │
│  │  • Availability: What's the uptime requirement?                  │   │
│  │  • Consistency: Is eventual consistency acceptable?              │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  E - ESTIMATION (3 minutes)                                             │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • Daily/Monthly Active Users                                    │   │
│  │  • Requests per second (read/write ratio)                        │   │
│  │  • Storage requirements                                          │   │
│  │  • Bandwidth requirements                                        │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  S - STORAGE (5 minutes)                                                │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • Define data models and schemas                                │   │
│  │  • Choose database type (SQL vs NoSQL)                           │   │
│  │  • Consider caching strategy                                     │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  H - HIGH-LEVEL DESIGN (10 minutes)                                     │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • Draw main components                                          │   │
│  │  • Show data flow                                                │   │
│  │  • Identify APIs                                                 │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  A - API DESIGN (5 minutes)                                             │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • Define main endpoints                                         │   │
│  │  • Specify request/response formats                              │   │
│  │  • Consider authentication                                       │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  D - DEEP DIVE (15 minutes)                                             │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • Focus on most complex/interesting components                  │   │
│  │  • Discuss algorithms and data structures                        │   │
│  │  • Address bottlenecks                                           │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  E - EVALUATE & EXTEND (5 minutes)                                      │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  • Discuss tradeoffs made                                        │   │
│  │  • How would you scale further?                                  │   │
│  │  • What would you do differently?                                │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Essential Building Blocks

### Load Balancers

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Load Balancer Concepts                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  PURPOSE: Distribute traffic across multiple servers                    │
│                                                                         │
│  ALGORITHMS:                                                            │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Round Robin      │  Requests distributed sequentially          │   │
│  │  Weighted RR      │  More traffic to powerful servers           │   │
│  │  Least Connections│  Send to server with fewest active conn     │   │
│  │  IP Hash          │  Same client IP → same server (stickiness)  │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  LAYERS:                                                                │
│  • L4 (Transport): Based on IP/port, faster but less flexible          │
│  • L7 (Application): Based on content, can route by URL/headers         │
│                                                                         │
│  DIAGRAM:                                                               │
│                                                                         │
│       Clients                                                           │
│         │                                                               │
│         ▼                                                               │
│   ┌───────────────┐                                                    │
│   │ Load Balancer │                                                    │
│   └───────┬───────┘                                                    │
│           │                                                             │
│     ┌─────┼─────┐                                                      │
│     ▼     ▼     ▼                                                      │
│   ┌───┐ ┌───┐ ┌───┐                                                    │
│   │S1 │ │S2 │ │S3 │  Application Servers                               │
│   └───┘ └───┘ └───┘                                                    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Caching

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Caching Strategies                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  CACHE PLACEMENT:                                                       │
│                                                                         │
│  Client ──▶ CDN ──▶ Load Balancer ──▶ App Server ──▶ Cache ──▶ DB      │
│    │         │           │               │           │                  │
│    └─────────┴───────────┴───────────────┴───────────┘                 │
│         Every layer can have caching                                    │
│                                                                         │
│  CACHING PATTERNS:                                                      │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Cache-Aside (Lazy Loading)                                      │   │
│  │  1. Check cache                                                  │   │
│  │  2. If miss, query DB                                            │   │
│  │  3. Store in cache                                               │   │
│  │  4. Return data                                                  │   │
│  │                                                                  │   │
│  │  Pros: Only requested data cached                                │   │
│  │  Cons: Cache miss = slower first request                         │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Write-Through                                                   │   │
│  │  1. Write to cache AND DB simultaneously                         │   │
│  │                                                                  │   │
│  │  Pros: Cache always consistent                                   │   │
│  │  Cons: Higher write latency                                      │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Write-Behind (Write-Back)                                       │   │
│  │  1. Write to cache                                               │   │
│  │  2. Asynchronously write to DB                                   │   │
│  │                                                                  │   │
│  │  Pros: Fast writes                                               │   │
│  │  Cons: Risk of data loss if cache fails                          │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  EVICTION POLICIES:                                                     │
│  • LRU (Least Recently Used) - Most common                              │
│  • LFU (Least Frequently Used)                                          │
│  • TTL (Time To Live) - Expiration-based                                │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Databases

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Database Selection                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  SQL (Relational)                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Use When:                                                       │   │
│  │  • Complex queries and joins                                     │   │
│  │  • ACID transactions required                                    │   │
│  │  • Data has clear relationships                                  │   │
│  │  • Schema is stable                                              │   │
│  │                                                                  │   │
│  │  Examples: PostgreSQL, MySQL                                     │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  NoSQL (Non-Relational)                                                 │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Document (MongoDB, DynamoDB):                                   │   │
│  │  • Flexible schema                                               │   │
│  │  • Nested data structures                                        │   │
│  │  • Use for: User profiles, content management                    │   │
│  │                                                                  │   │
│  │  Key-Value (Redis, DynamoDB):                                    │   │
│  │  • Simple get/set operations                                     │   │
│  │  • Ultra-fast reads/writes                                       │   │
│  │  • Use for: Caching, sessions, rate limiting                     │   │
│  │                                                                  │   │
│  │  Wide-Column (Cassandra, HBase):                                 │   │
│  │  • Time-series data                                              │   │
│  │  • Write-heavy workloads                                         │   │
│  │  • Use for: Analytics, logs, IoT data                            │   │
│  │                                                                  │   │
│  │  Graph (Neo4j):                                                  │   │
│  │  • Complex relationships                                         │   │
│  │  • Use for: Social networks, recommendations                     │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  SCALING STRATEGIES:                                                    │
│  • Vertical: Add more CPU/RAM (limited)                                │
│  • Horizontal: Add more machines (sharding/partitioning)               │
│  • Read replicas: Offload read traffic                                  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Message Queues

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Message Queues                                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  PURPOSE: Asynchronous communication between services                   │
│                                                                         │
│  BENEFITS:                                                              │
│  • Decouple producers and consumers                                     │
│  • Handle traffic spikes (buffer)                                       │
│  • Retry failed operations                                              │
│  • Scale independently                                                  │
│                                                                         │
│  PATTERNS:                                                              │
│                                                                         │
│  Point-to-Point:                                                        │
│  ┌──────────┐     ┌───────┐     ┌──────────┐                           │
│  │ Producer │────▶│ Queue │────▶│ Consumer │                           │
│  └──────────┘     └───────┘     └──────────┘                           │
│                                                                         │
│  Pub/Sub:                                                               │
│  ┌──────────┐     ┌───────┐     ┌──────────┐                           │
│  │ Publisher│────▶│ Topic │────▶│Consumer 1│                           │
│  └──────────┘     └───┬───┘     └──────────┘                           │
│                       │         ┌──────────┐                           │
│                       └────────▶│Consumer 2│                           │
│                                 └──────────┘                           │
│                                                                         │
│  USE CASES:                                                             │
│  • Email/notification sending                                           │
│  • Order processing                                                     │
│  • Image/video processing                                               │
│  • Log aggregation                                                      │
│                                                                         │
│  TOOLS: RabbitMQ, Apache Kafka, AWS SQS, Redis Pub/Sub                 │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### CDN (Content Delivery Network)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    CDN Architecture                                     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  PURPOSE: Serve static content from edge locations close to users       │
│                                                                         │
│                        ┌─────────────────────────────────┐              │
│                        │         Origin Server          │              │
│                        └─────────────────────────────────┘              │
│                                      ▲                                  │
│                                      │ (Cache miss)                     │
│                                      │                                  │
│   ┌─────────────────────────────────────────────────────────────┐      │
│   │                         CDN Network                          │      │
│   │                                                              │      │
│   │  ┌──────────┐    ┌──────────┐    ┌──────────┐              │      │
│   │  │ Edge EU  │    │ Edge US  │    │ Edge Asia│              │      │
│   │  └────┬─────┘    └────┬─────┘    └────┬─────┘              │      │
│   │       │               │               │                     │      │
│   └───────┼───────────────┼───────────────┼─────────────────────┘      │
│           │               │               │                             │
│           ▼               ▼               ▼                             │
│        Users           Users           Users                           │
│        (EU)            (US)            (Asia)                          │
│                                                                         │
│  WHAT TO CACHE:                                                         │
│  • Images, videos, CSS, JavaScript                                      │
│  • HTML pages (with proper cache headers)                               │
│  • API responses (with caution)                                         │
│                                                                         │
│  PROVIDERS: Cloudflare, AWS CloudFront, Fastly, Akamai                 │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Back-of-the-Envelope Calculations

### Common Numbers to Know

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Numbers Every Engineer Should Know                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  LATENCIES:                                                             │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  L1 cache reference                    0.5 ns                    │   │
│  │  L2 cache reference                    7 ns                      │   │
│  │  Main memory reference                 100 ns                    │   │
│  │  SSD random read                       150 μs                    │   │
│  │  HDD seek                              10 ms                     │   │
│  │  Round trip within datacenter          0.5 ms                    │   │
│  │  Round trip US East to West            40 ms                     │   │
│  │  Round trip US to Europe               80 ms                     │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  THROUGHPUT:                                                            │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Read 1 MB from RAM                    250 μs                    │   │
│  │  Read 1 MB from SSD                    1 ms                      │   │
│  │  Read 1 MB from network                10 ms                     │   │
│  │  Read 1 MB from HDD                    20 ms                     │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  STORAGE:                                                               │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  1 KB = 1,000 bytes (text, small JSON)                           │   │
│  │  1 MB = 1,000 KB (image, short audio)                            │   │
│  │  1 GB = 1,000 MB (video, database table)                         │   │
│  │  1 TB = 1,000 GB (large database)                                │   │
│  │  1 PB = 1,000 TB (big data)                                      │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  QUICK ESTIMATES:                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Seconds in a day                      ~100,000 (86,400)         │   │
│  │  Seconds in a month                    ~2.5 million              │   │
│  │  1 million requests/day                ~12 requests/second       │   │
│  │  100 million users, 10% DAU            ~10 million daily users   │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Estimation Example: Twitter-like Service

```
REQUIREMENTS:
- 500 million users
- 200 million daily active users (DAU)
- Average user reads 100 tweets/day
- Average user posts 2 tweets/day
- 10% of tweets have media (1 MB avg)

CALCULATIONS:

Read Traffic:
- 200M users × 100 tweets = 20 billion reads/day
- 20B / 100K seconds = 200,000 reads/second
- Peak (2x average) = 400,000 reads/second

Write Traffic:
- 200M users × 2 tweets = 400 million writes/day
- 400M / 100K seconds = 4,000 writes/second
- Peak = 8,000 writes/second

Storage (per day):
- Tweet text: 400M × 280 bytes = ~112 GB/day
- Media: 40M × 1 MB = ~40 TB/day
- Total: ~40 TB/day → ~15 PB/year

Bandwidth:
- Reads: 200K req/sec × 280 bytes = ~56 MB/sec (text only)
- With media: Much higher, need CDN

CONCLUSIONS:
- Read-heavy system (50:1 ratio)
- Need caching layer for hot tweets
- CDN for media content
- Database sharding required
```

---

## 4. Common System Design Problems

### Design a URL Shortener (like bit.ly)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    URL Shortener Design                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  REQUIREMENTS:                                                          │
│  • Generate short URL from long URL                                     │
│  • Redirect short URL to original                                       │
│  • Custom short URLs (optional)                                         │
│  • Analytics (optional)                                                 │
│  • Scale: 100M URLs/day created, 10B redirects/day                     │
│                                                                         │
│  API DESIGN:                                                            │
│  POST /api/shorten                                                      │
│    Body: { "long_url": "https://...", "custom_alias": "optional" }     │
│    Response: { "short_url": "https://short.ly/abc123" }                │
│                                                                         │
│  GET /{short_code}                                                      │
│    Response: 301 Redirect to original URL                              │
│                                                                         │
│  KEY DESIGN DECISIONS:                                                  │
│                                                                         │
│  1. Short Code Generation:                                              │
│     • Base62 encoding (a-z, A-Z, 0-9)                                   │
│     • 7 characters = 62^7 = 3.5 trillion combinations                   │
│     • Options:                                                          │
│       - Counter-based: Unique ID → Base62                               │
│       - Hash-based: MD5(long_url) → Take first 7 chars                 │
│       - Random: Generate random, check for collision                    │
│                                                                         │
│  2. Database Schema:                                                    │
│     urls                                                                │
│     ├── short_code (PK, indexed)                                       │
│     ├── long_url                                                        │
│     ├── user_id                                                         │
│     ├── created_at                                                      │
│     └── expires_at                                                      │
│                                                                         │
│  ARCHITECTURE:                                                          │
│                                                                         │
│  ┌─────────┐     ┌────────────┐     ┌─────────────┐                    │
│  │ Client  │────▶│    LB      │────▶│ App Servers │                    │
│  └─────────┘     └────────────┘     └──────┬──────┘                    │
│                                            │                            │
│                        ┌───────────────────┼───────────────────┐       │
│                        ▼                   ▼                   ▼       │
│                  ┌──────────┐        ┌──────────┐        ┌──────────┐ │
│                  │  Cache   │        │ Database │        │ Counter  │ │
│                  │ (Redis)  │        │ (NoSQL)  │        │ Service  │ │
│                  └──────────┘        └──────────┘        └──────────┘ │
│                                                                         │
│  SCALING CONSIDERATIONS:                                                │
│  • Cache popular URLs (high hit rate expected)                          │
│  • Shard database by short_code hash                                    │
│  • Use CDN for geographic distribution                                  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Design a Rate Limiter

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Rate Limiter Design                                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  REQUIREMENTS:                                                          │
│  • Limit requests per user/IP                                           │
│  • Different limits for different endpoints                             │
│  • Distributed across multiple servers                                  │
│  • Low latency overhead                                                 │
│                                                                         │
│  ALGORITHMS:                                                            │
│                                                                         │
│  1. Token Bucket:                                                       │
│     ┌────────────────────────────────────────────────────────────┐     │
│     │  • Bucket holds tokens (max = bucket size)                 │     │
│     │  • Tokens added at fixed rate                              │     │
│     │  • Each request consumes a token                           │     │
│     │  • Request rejected if no tokens                           │     │
│     │                                                            │     │
│     │  Pros: Allows bursts up to bucket size                     │     │
│     │  Cons: More memory (store token count + timestamp)         │     │
│     └────────────────────────────────────────────────────────────┘     │
│                                                                         │
│  2. Sliding Window Counter:                                             │
│     ┌────────────────────────────────────────────────────────────┐     │
│     │  • Track request counts in time windows                    │     │
│     │  • Interpolate between windows for smoothness              │     │
│     │                                                            │     │
│     │  Current window: 5 requests                                │     │
│     │  Previous window: 10 requests                              │     │
│     │  Position: 70% through current window                      │     │
│     │  Effective count: 5 + (10 × 0.3) = 8 requests              │     │
│     └────────────────────────────────────────────────────────────┘     │
│                                                                         │
│  IMPLEMENTATION (Token Bucket with Redis):                              │
│                                                                         │
│  function checkRateLimit(userId) {                                      │
│    const key = `rate_limit:${userId}`;                                 │
│    const now = Date.now();                                              │
│    const windowMs = 60000; // 1 minute                                  │
│    const maxRequests = 100;                                             │
│                                                                         │
│    // Lua script for atomic operation                                   │
│    const script = `                                                    │
│      local tokens = redis.call('GET', KEYS[1])                         │
│      local lastRefill = redis.call('GET', KEYS[2])                     │
│      -- Refill logic...                                                │
│      -- Check and decrement...                                         │
│      return allowed                                                     │
│    `;                                                                   │
│                                                                         │
│    return redis.eval(script, [key, `${key}:time`], [now, ...]);       │
│  }                                                                      │
│                                                                         │
│  ARCHITECTURE:                                                          │
│                                                                         │
│   Request → Rate Limiter → App Server                                   │
│                 │                                                       │
│                 ▼                                                       │
│         ┌─────────────┐                                                │
│         │    Redis    │  (Distributed counter store)                   │
│         │   Cluster   │                                                │
│         └─────────────┘                                                │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Design a News Feed (like Twitter/Facebook)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    News Feed Design                                     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  REQUIREMENTS:                                                          │
│  • Users can post content                                               │
│  • Users see posts from people they follow                              │
│  • Feed sorted by relevance/time                                        │
│  • Support millions of users                                            │
│                                                                         │
│  TWO APPROACHES:                                                        │
│                                                                         │
│  1. Pull Model (Fan-out on Read):                                       │
│     ┌────────────────────────────────────────────────────────────┐     │
│     │  When user requests feed:                                   │     │
│     │  1. Get list of followed users                              │     │
│     │  2. Query posts from each user                              │     │
│     │  3. Merge and sort                                          │     │
│     │  4. Return to user                                          │     │
│     │                                                             │     │
│     │  Pros: No extra storage, always fresh                       │     │
│     │  Cons: Slow for users following many people                 │     │
│     └────────────────────────────────────────────────────────────┘     │
│                                                                         │
│  2. Push Model (Fan-out on Write):                                      │
│     ┌────────────────────────────────────────────────────────────┐     │
│     │  When user posts:                                           │     │
│     │  1. Get list of followers                                   │     │
│     │  2. Push post to each follower's feed cache                 │     │
│     │                                                             │     │
│     │  When user requests feed:                                   │     │
│     │  1. Read from pre-computed feed cache                       │     │
│     │                                                             │     │
│     │  Pros: Fast reads                                           │     │
│     │  Cons: Slow writes for celebrities, storage overhead        │     │
│     └────────────────────────────────────────────────────────────┘     │
│                                                                         │
│  HYBRID APPROACH (Best of Both):                                        │
│  • Normal users: Push model                                             │
│  • Celebrities (>1M followers): Pull model on read                      │
│                                                                         │
│  ARCHITECTURE:                                                          │
│                                                                         │
│  ┌──────────┐      ┌──────────────┐      ┌──────────────────┐          │
│  │  Client  │─────▶│  API Gateway │─────▶│  Feed Service    │          │
│  └──────────┘      └──────────────┘      └────────┬─────────┘          │
│                                                   │                     │
│           ┌───────────────────────────────────────┼──────────┐         │
│           │                                       │          │         │
│           ▼                                       ▼          ▼         │
│    ┌─────────────┐                    ┌──────────────┐ ┌──────────┐   │
│    │ Post Service│                    │ Feed Cache   │ │ User DB  │   │
│    │             │                    │ (Redis)      │ └──────────┘   │
│    └──────┬──────┘                    └──────────────┘                │
│           │                                                            │
│           ▼                                                            │
│    ┌─────────────┐      ┌──────────────────┐                          │
│    │  Posts DB   │      │  Fan-out Worker  │                          │
│    │             │◀────▶│  (Async queue)   │                          │
│    └─────────────┘      └──────────────────┘                          │
│                                                                         │
│  DATA MODEL:                                                            │
│                                                                         │
│  posts: { id, user_id, content, media_urls, created_at }               │
│  followers: { user_id, follower_id }                                   │
│  feed_cache: { user_id: [post_ids...] }  // In Redis                   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Handling Tradeoffs

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Common Tradeoffs in System Design                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  CONSISTENCY vs AVAILABILITY (CAP Theorem)                              │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  You can only guarantee 2 of 3:                                  │   │
│  │  • Consistency: All nodes see same data                          │   │
│  │  • Availability: System always responds                          │   │
│  │  • Partition tolerance: Works despite network failures           │   │
│  │                                                                  │   │
│  │  Choose CP: Banking, inventory (consistency critical)            │   │
│  │  Choose AP: Social media, caching (availability critical)        │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  LATENCY vs THROUGHPUT                                                  │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Optimize for latency: Cache aggressively, fewer hops            │   │
│  │  Optimize for throughput: Batch operations, async processing     │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  SQL vs NoSQL                                                           │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  SQL: Strong consistency, complex queries, ACID                  │   │
│  │  NoSQL: Flexible schema, horizontal scaling, eventual consistency│   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  PUSH vs PULL                                                           │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Push: Fast reads, storage overhead, write amplification         │   │
│  │  Pull: Storage efficient, slower reads, compute on read          │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
│  SYNC vs ASYNC                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Sync: Simpler, immediate feedback, blocks caller                │   │
│  │  Async: Better throughput, complex, harder to debug              │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Common Design Questions

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    System Design Question Categories                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  SOCIAL / CONTENT                                                       │
│  • Design Twitter / X                                                   │
│  • Design Facebook News Feed                                            │
│  • Design Instagram                                                     │
│  • Design TikTok / YouTube                                              │
│                                                                         │
│  MESSAGING                                                              │
│  • Design WhatsApp / Messenger                                          │
│  • Design Slack                                                         │
│  • Design Notification System                                           │
│                                                                         │
│  INFRASTRUCTURE                                                         │
│  • Design URL Shortener                                                 │
│  • Design Rate Limiter                                                  │
│  • Design Distributed Cache                                             │
│  • Design Search Autocomplete                                           │
│                                                                         │
│  E-COMMERCE                                                             │
│  • Design Amazon                                                        │
│  • Design Uber / Lyft                                                   │
│  • Design Food Delivery (DoorDash)                                      │
│  • Design Booking System (Airbnb)                                       │
│                                                                         │
│  STORAGE / FILE                                                         │
│  • Design Dropbox / Google Drive                                        │
│  • Design Google Photos                                                 │
│  • Design Distributed File System                                       │
│                                                                         │
│  SPECIALIZED                                                            │
│  • Design Web Crawler                                                   │
│  • Design Typeahead / Autocomplete                                      │
│  • Design Key-Value Store                                               │
│  • Design Unique ID Generator                                           │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Practice Tips

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    System Design Preparation Tips                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. PRACTICE OUT LOUD                                                   │
│     • Draw diagrams while explaining                                    │
│     • Practice with a timer (45 minutes)                                │
│     • Record yourself and review                                        │
│                                                                         │
│  2. STUDY REAL SYSTEMS                                                  │
│     • Read engineering blogs (Netflix, Uber, Airbnb)                    │
│     • Understand how real systems are built                             │
│     • Learn from postmortems                                            │
│                                                                         │
│  3. KNOW YOUR NUMBERS                                                   │
│     • Memorize latency numbers                                          │
│     • Practice back-of-envelope calculations                            │
│                                                                         │
│  4. FOCUS ON TRADEOFFS                                                  │
│     • There's no "right" answer                                         │
│     • Explain why you chose each approach                               │
│     • Acknowledge limitations                                           │
│                                                                         │
│  5. START SIMPLE                                                        │
│     • Begin with basic architecture                                     │
│     • Add complexity when asked                                         │
│     • Don't over-engineer initially                                     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    System Design Success Checklist                      │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ✓ Always clarify requirements first                                   │
│  ✓ Use the RESHADE framework consistently                              │
│  ✓ Draw diagrams as you explain                                        │
│  ✓ Do back-of-envelope calculations                                    │
│  ✓ Choose appropriate data stores for the use case                     │
│  ✓ Discuss tradeoffs explicitly                                        │
│  ✓ Start simple, then scale                                            │
│  ✓ Know common building blocks (LB, cache, queue, CDN)                 │
│  ✓ Practice 2-3 designs thoroughly rather than many superficially      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Additional Resources

- [System Design Primer](https://github.com/donnemartin/system-design-primer)
- [Grokking System Design](https://www.designgurus.io/course/grokking-the-system-design-interview)
- [ByteByteGo](https://bytebytego.com/)
- [Alex Xu's System Design Books](https://www.amazon.com/System-Design-Interview-insiders-Second/dp/B08CMF2CQF)
- [High Scalability Blog](http://highscalability.com/)

---

Tomorrow, we'll cover **Behavioral Interviews** - equally important as technical skills for landing offers.
