# Days 6-7: System Design Examples

## Introduction

Over these two days, we'll apply everything we've learned to design real-world systems. You'll learn how to approach system design interviews and practice designing scalable solutions for common applications. We'll walk through URL shorteners, real-time chat, and social media feeds.

## Learning Objectives

By the end of this lesson, you will be able to:
- Apply a structured approach to system design problems
- Estimate scale and capacity requirements
- Design systems with appropriate trade-offs
- Handle common system design interview questions
- Create architecture diagrams and explain decisions

---

## System Design Framework

```
┌─────────────────────────────────────────────────────────────────┐
│              SYSTEM DESIGN APPROACH (URES)                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. UNDERSTAND THE REQUIREMENTS (5 min)                          │
│     • What are the functional requirements?                      │
│     • What are the non-functional requirements?                  │
│     • What is the scale? (users, data, requests)                │
│     • What are the constraints?                                  │
│                                                                  │
│  2. ROUGH ESTIMATES (5 min)                                      │
│     • Daily active users                                         │
│     • Read vs write ratio                                        │
│     • Storage requirements                                       │
│     • Bandwidth needs                                            │
│                                                                  │
│  3. ESTABLISH HIGH-LEVEL DESIGN (10 min)                         │
│     • Core components                                            │
│     • Data flow                                                  │
│     • API design                                                 │
│     • Database schema                                            │
│                                                                  │
│  4. SCALE AND OPTIMIZE (15 min)                                  │
│     • Identify bottlenecks                                       │
│     • Add caching                                                │
│     • Scale components                                           │
│     • Handle edge cases                                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Example 1: URL Shortener (like bit.ly)

### 1. Requirements

```
┌─────────────────────────────────────────────────────────────────┐
│                    URL SHORTENER REQUIREMENTS                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  FUNCTIONAL:                                                     │
│  • Shorten a long URL to a short URL                            │
│  • Redirect short URL to original URL                           │
│  • Custom short links (optional)                                 │
│  • Analytics (click tracking)                                   │
│  • Link expiration (optional)                                   │
│                                                                  │
│  NON-FUNCTIONAL:                                                 │
│  • High availability (99.9% uptime)                             │
│  • Low latency redirection (< 100ms)                            │
│  • Short URLs should be as short as possible                    │
│  • Not predictable (security)                                   │
│                                                                  │
│  CONSTRAINTS:                                                    │
│  • 100M new URLs per day                                        │
│  • 10:1 read-to-write ratio (1B redirects/day)                  │
│  • URLs stored for 5 years                                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 2. Estimates

```
┌─────────────────────────────────────────────────────────────────┐
│                      CAPACITY ESTIMATION                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  TRAFFIC:                                                        │
│  • Writes: 100M/day = 1,157/sec                                 │
│  • Reads:  1B/day   = 11,574/sec                                │
│  • Peak:   ~30,000 reads/sec (3x average)                       │
│                                                                  │
│  STORAGE:                                                        │
│  • 100M URLs × 365 days × 5 years = 182.5B URLs                 │
│  • Each record: ~500 bytes                                      │
│  • Total: 182.5B × 500B = 91.25 TB                              │
│                                                                  │
│  URL LENGTH:                                                     │
│  • 182.5B unique URLs needed                                    │
│  • Base62 (a-z, A-Z, 0-9) = 62 characters                       │
│  • 62^7 = 3.5 trillion combinations ✓                           │
│  • Short URL: 7 characters                                       │
│                                                                  │
│  BANDWIDTH:                                                      │
│  • Write: 1,157/sec × 500B = 578 KB/s                           │
│  • Read:  11,574/sec × 500B = 5.8 MB/s                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 3. High-Level Design

```
┌─────────────────────────────────────────────────────────────────┐
│                  URL SHORTENER ARCHITECTURE                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────┐     ┌─────────────┐     ┌─────────────────────┐   │
│  │  Client  │────▶│Load Balancer│────▶│    API Servers      │   │
│  └──────────┘     └─────────────┘     │  (Stateless)        │   │
│                                       └──────────┬──────────┘   │
│                                                  │               │
│        ┌────────────────────┬───────────────────┼───────────┐   │
│        │                    │                   │           │   │
│        ▼                    ▼                   ▼           ▼   │
│  ┌───────────┐       ┌───────────┐       ┌───────────┐ ┌──────┐ │
│  │   Redis   │       │   Cache   │       │ Key Gen   │ │Analyt│ │
│  │Rate Limit │       │ (Hot URLs)│       │ Service   │ │ Queue│ │
│  └───────────┘       └───────────┘       └─────┬─────┘ └──────┘ │
│                            │                   │                 │
│                            │                   ▼                 │
│                            │           ┌───────────────┐        │
│                            └──────────▶│   Database    │        │
│                                        │  (Sharded)    │        │
│                                        └───────────────┘        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### API Design

```javascript
// API Endpoints
// POST /api/shorten
// Request: { "longUrl": "https://example.com/very/long/path" }
// Response: { "shortUrl": "https://short.ly/abc123" }

// GET /:shortCode
// Response: 301 Redirect to original URL

// GET /api/analytics/:shortCode
// Response: { "clicks": 1234, "referrers": {...}, "locations": {...} }

// Database Schema
const urlSchema = {
  id: 'string',           // Short code (primary key)
  longUrl: 'string',      // Original URL
  userId: 'string',       // Creator (optional)
  createdAt: 'timestamp',
  expiresAt: 'timestamp', // Optional expiration
  clicks: 'integer'       // Click counter
};

// Key Generation Approaches
// 1. Hash and Truncate
function generateShortCode(longUrl) {
  const hash = crypto.createHash('md5').update(longUrl).digest('base64');
  return hash.substring(0, 7).replace(/[+/=]/g, (c) => {
    return { '+': 'a', '/': 'b', '=': 'c' }[c];
  });
}

// 2. Pre-generated Keys (Better for scale)
class KeyGeneratorService {
  constructor(db) {
    this.db = db;
    this.keyBuffer = [];
    this.bufferSize = 1000;
  }

  async getKey() {
    if (this.keyBuffer.length === 0) {
      await this.refillBuffer();
    }
    return this.keyBuffer.pop();
  }

  async refillBuffer() {
    // Fetch unused keys from database
    const keys = await this.db.query(`
      UPDATE keys
      SET used = true
      WHERE id IN (
        SELECT id FROM keys WHERE used = false LIMIT $1
      )
      RETURNING key
    `, [this.bufferSize]);

    this.keyBuffer = keys.rows.map(r => r.key);
  }

  // Pre-generate keys (run as batch job)
  static generateKeys(count) {
    const keys = [];
    const chars = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';

    for (let i = 0; i < count; i++) {
      let key = '';
      for (let j = 0; j < 7; j++) {
        key += chars[Math.floor(Math.random() * chars.length)];
      }
      keys.push(key);
    }

    return keys;
  }
}
```

### 4. Detailed Design

```javascript
// url-shortener/src/services/url-service.js
class UrlShortenerService {
  constructor(db, cache, keyGenerator, analyticsQueue) {
    this.db = db;
    this.cache = cache;
    this.keyGenerator = keyGenerator;
    this.analyticsQueue = analyticsQueue;
  }

  async shorten(longUrl, userId, options = {}) {
    // Validate URL
    if (!this.isValidUrl(longUrl)) {
      throw new Error('Invalid URL');
    }

    // Check for existing URL (deduplication)
    const existing = await this.db.query(
      'SELECT id FROM urls WHERE long_url = $1 AND user_id = $2',
      [longUrl, userId]
    );

    if (existing.rows.length > 0) {
      return existing.rows[0].id;
    }

    // Get unique short code
    const shortCode = options.customCode || await this.keyGenerator.getKey();

    // Check custom code availability
    if (options.customCode) {
      const exists = await this.db.query(
        'SELECT id FROM urls WHERE id = $1',
        [shortCode]
      );
      if (exists.rows.length > 0) {
        throw new Error('Short code already taken');
      }
    }

    // Insert URL
    await this.db.query(
      `INSERT INTO urls (id, long_url, user_id, created_at, expires_at)
       VALUES ($1, $2, $3, NOW(), $4)`,
      [shortCode, longUrl, userId, options.expiresAt]
    );

    // Cache the mapping
    await this.cache.set(`url:${shortCode}`, longUrl, 86400); // 24 hours

    return shortCode;
  }

  async resolve(shortCode, requestInfo) {
    // Try cache first
    let longUrl = await this.cache.get(`url:${shortCode}`);

    if (!longUrl) {
      // Cache miss - query database
      const result = await this.db.query(
        'SELECT long_url, expires_at FROM urls WHERE id = $1',
        [shortCode]
      );

      if (result.rows.length === 0) {
        throw new Error('URL not found');
      }

      const url = result.rows[0];

      // Check expiration
      if (url.expires_at && new Date(url.expires_at) < new Date()) {
        throw new Error('URL expired');
      }

      longUrl = url.long_url;

      // Cache for next time
      await this.cache.set(`url:${shortCode}`, longUrl, 86400);
    }

    // Queue analytics (async, don't block redirect)
    this.analyticsQueue.add('click', {
      shortCode,
      timestamp: Date.now(),
      ip: requestInfo.ip,
      userAgent: requestInfo.userAgent,
      referrer: requestInfo.referrer
    });

    return longUrl;
  }

  isValidUrl(string) {
    try {
      new URL(string);
      return true;
    } catch {
      return false;
    }
  }
}

// API Route Handler
app.get('/:shortCode', async (req, res) => {
  try {
    const longUrl = await urlService.resolve(req.params.shortCode, {
      ip: req.ip,
      userAgent: req.headers['user-agent'],
      referrer: req.headers.referer
    });

    // 301 for permanent, 302 for temporary
    res.redirect(301, longUrl);
  } catch (error) {
    res.status(404).send('URL not found');
  }
});
```

### Scaling Considerations

```
┌─────────────────────────────────────────────────────────────────┐
│                    SCALING STRATEGIES                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  DATABASE SHARDING:                                              │
│  • Shard by first character of short code                       │
│  • 62 shards (a-z, A-Z, 0-9)                                    │
│  • Consistent hashing for even distribution                      │
│                                                                  │
│  CACHING:                                                        │
│  • Cache hot URLs in Redis                                      │
│  • 80/20 rule: 20% of URLs get 80% of traffic                   │
│  • LRU eviction policy                                          │
│                                                                  │
│  CDN:                                                            │
│  • Cache redirects at edge                                       │
│  • Lower latency for global users                                │
│                                                                  │
│  READ REPLICAS:                                                  │
│  • Handle read-heavy workload                                   │
│  • Write to primary, read from replicas                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Example 2: Real-Time Chat (like Slack/Discord)

### 1. Requirements

```
┌─────────────────────────────────────────────────────────────────┐
│                    CHAT SYSTEM REQUIREMENTS                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  FUNCTIONAL:                                                     │
│  • 1:1 messaging                                                 │
│  • Group chats (up to 1000 members)                             │
│  • Online presence indicators                                   │
│  • Message read receipts                                         │
│  • File/image sharing                                           │
│  • Message history and search                                   │
│  • Push notifications                                           │
│                                                                  │
│  NON-FUNCTIONAL:                                                 │
│  • Real-time delivery (< 100ms)                                 │
│  • Message ordering guaranteed                                   │
│  • 99.99% uptime                                                │
│  • Support offline messaging                                     │
│                                                                  │
│  SCALE:                                                          │
│  • 100M daily active users                                      │
│  • 1B messages per day                                          │
│  • Average 10 messages per user per day                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 2. Estimates

```
┌─────────────────────────────────────────────────────────────────┐
│                      CAPACITY ESTIMATION                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  CONNECTIONS:                                                    │
│  • 100M DAU, assume 10% concurrent = 10M connections            │
│  • Each server handles 50K connections                          │
│  • Need 200 WebSocket servers                                   │
│                                                                  │
│  MESSAGES:                                                       │
│  • 1B messages/day = 11,574 messages/sec                        │
│  • Average message: 100 bytes                                   │
│  • Daily: 1B × 100B = 100 GB                                    │
│  • Yearly: 36.5 TB                                              │
│                                                                  │
│  BANDWIDTH:                                                      │
│  • Outgoing: 11,574 msg/s × 100B × avg 10 recipients = 11.5 MB/s│
│  • Plus presence updates, typing indicators                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 3. High-Level Design

```
┌─────────────────────────────────────────────────────────────────┐
│                    CHAT SYSTEM ARCHITECTURE                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────┐                                                    │
│  │  Client  │◀────────── WebSocket ──────────▶┌──────────────┐  │
│  └──────────┘                                  │  WebSocket   │  │
│                                                │   Servers    │  │
│                                                └──────┬───────┘  │
│                                                       │          │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                        Message Broker                        │ │
│  │                    (Redis Pub/Sub or Kafka)                  │ │
│  └─────────────────────────────────────────────────────────────┘ │
│         │              │              │              │           │
│         ▼              ▼              ▼              ▼           │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐     │
│  │  Message  │  │ Presence  │  │   Push    │  │  Session  │     │
│  │  Service  │  │  Service  │  │  Service  │  │  Service  │     │
│  └─────┬─────┘  └─────┬─────┘  └───────────┘  └───────────┘     │
│        │              │                                          │
│        ▼              ▼                                          │
│  ┌───────────┐  ┌───────────┐                                   │
│  │ Cassandra │  │   Redis   │                                   │
│  │ (Messages)│  │(Presence) │                                   │
│  └───────────┘  └───────────┘                                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Implementation

```javascript
// WebSocket Server
const WebSocket = require('ws');
const Redis = require('ioredis');

class ChatServer {
  constructor() {
    this.wss = new WebSocket.Server({ port: 8080 });
    this.redis = new Redis();
    this.subscriber = new Redis();
    this.connections = new Map(); // userId -> WebSocket

    this.setupWebSocket();
    this.setupRedisSubscriber();
  }

  setupWebSocket() {
    this.wss.on('connection', async (ws, req) => {
      const userId = this.authenticateConnection(req);

      if (!userId) {
        ws.close(4001, 'Unauthorized');
        return;
      }

      // Store connection
      this.connections.set(userId, ws);

      // Update presence
      await this.updatePresence(userId, 'online');

      // Subscribe to user's channel
      await this.subscriber.subscribe(`user:${userId}`);

      // Handle messages
      ws.on('message', (data) => this.handleMessage(userId, data));

      // Handle disconnect
      ws.on('close', async () => {
        this.connections.delete(userId);
        await this.updatePresence(userId, 'offline');
        await this.subscriber.unsubscribe(`user:${userId}`);
      });

      // Send pending messages
      await this.deliverPendingMessages(userId);
    });
  }

  setupRedisSubscriber() {
    this.subscriber.on('message', (channel, message) => {
      const userId = channel.split(':')[1];
      const ws = this.connections.get(userId);

      if (ws && ws.readyState === WebSocket.OPEN) {
        ws.send(message);
      }
    });
  }

  async handleMessage(senderId, data) {
    const message = JSON.parse(data);

    switch (message.type) {
      case 'chat':
        await this.handleChatMessage(senderId, message);
        break;
      case 'typing':
        await this.handleTypingIndicator(senderId, message);
        break;
      case 'read':
        await this.handleReadReceipt(senderId, message);
        break;
    }
  }

  async handleChatMessage(senderId, message) {
    const { conversationId, content, recipients } = message;

    // Generate message ID and timestamp
    const messageId = this.generateMessageId();
    const timestamp = Date.now();

    // Store message
    await this.storeMessage({
      id: messageId,
      conversationId,
      senderId,
      content,
      timestamp,
      status: 'sent'
    });

    // Prepare message for delivery
    const outgoingMessage = JSON.stringify({
      type: 'chat',
      id: messageId,
      conversationId,
      senderId,
      content,
      timestamp
    });

    // Deliver to each recipient
    for (const recipientId of recipients) {
      // Try direct delivery via WebSocket
      const delivered = await this.deliverToUser(recipientId, outgoingMessage);

      if (!delivered) {
        // Store for offline delivery
        await this.storePendingMessage(recipientId, outgoingMessage);

        // Send push notification
        await this.sendPushNotification(recipientId, senderId, content);
      }
    }

    // Acknowledge to sender
    const senderWs = this.connections.get(senderId);
    if (senderWs) {
      senderWs.send(JSON.stringify({
        type: 'ack',
        messageId,
        status: 'sent'
      }));
    }
  }

  async deliverToUser(userId, message) {
    // Check if user is connected to this server
    const ws = this.connections.get(userId);
    if (ws && ws.readyState === WebSocket.OPEN) {
      ws.send(message);
      return true;
    }

    // Check if user is connected to another server
    const isOnline = await this.redis.get(`presence:${userId}`);
    if (isOnline) {
      // Publish to Redis for other servers
      await this.redis.publish(`user:${userId}`, message);
      return true;
    }

    return false;
  }

  async updatePresence(userId, status) {
    const key = `presence:${userId}`;

    if (status === 'online') {
      await this.redis.set(key, process.env.SERVER_ID, 'EX', 300); // 5 min TTL
    } else {
      await this.redis.del(key);
    }

    // Notify friends about presence change
    const friends = await this.getFriends(userId);
    const presenceUpdate = JSON.stringify({
      type: 'presence',
      userId,
      status
    });

    for (const friendId of friends) {
      await this.deliverToUser(friendId, presenceUpdate);
    }
  }

  generateMessageId() {
    // Snowflake-like ID: timestamp + server ID + sequence
    const timestamp = Date.now().toString(36);
    const serverId = process.env.SERVER_ID.padStart(4, '0');
    const sequence = (this.sequence++ % 1000).toString().padStart(3, '0');
    return `${timestamp}${serverId}${sequence}`;
  }
}

// Message Storage (Cassandra schema)
const messageSchema = `
  CREATE TABLE messages (
    conversation_id UUID,
    message_id TIMEUUID,
    sender_id UUID,
    content TEXT,
    created_at TIMESTAMP,
    PRIMARY KEY (conversation_id, message_id)
  ) WITH CLUSTERING ORDER BY (message_id DESC);
`;

// Conversation participants
const participantSchema = `
  CREATE TABLE conversation_participants (
    conversation_id UUID,
    user_id UUID,
    joined_at TIMESTAMP,
    last_read_at TIMESTAMP,
    PRIMARY KEY (conversation_id, user_id)
  );
`;

// User conversations index
const userConversationsSchema = `
  CREATE TABLE user_conversations (
    user_id UUID,
    conversation_id UUID,
    last_message_at TIMESTAMP,
    PRIMARY KEY (user_id, last_message_at, conversation_id)
  ) WITH CLUSTERING ORDER BY (last_message_at DESC);
`;
```

### Group Chat Optimization

```javascript
// Handling large groups (1000+ members)
class GroupChatService {
  async sendGroupMessage(groupId, senderId, content) {
    const messageId = generateMessageId();

    // Store message
    await this.storeMessage(groupId, messageId, senderId, content);

    // Get group members in batches
    const batchSize = 100;
    let offset = 0;
    let members;

    do {
      members = await this.getGroupMembers(groupId, offset, batchSize);

      // Process batch concurrently
      await Promise.all(members.map(memberId => {
        if (memberId !== senderId) {
          return this.deliverOrQueue(memberId, {
            type: 'group_message',
            groupId,
            messageId,
            senderId,
            content
          });
        }
      }));

      offset += batchSize;
    } while (members.length === batchSize);
  }

  // Fan-out on read for very large groups
  async getGroupMessages(groupId, userId, limit = 50) {
    // Get messages from group
    const messages = await cassandra.execute(
      'SELECT * FROM messages WHERE conversation_id = ? LIMIT ?',
      [groupId, limit]
    );

    // Get user's last read position
    const lastRead = await redis.get(`lastread:${userId}:${groupId}`);

    return messages.rows.map(msg => ({
      ...msg,
      isRead: msg.message_id <= lastRead
    }));
  }
}
```

---

## Example 3: Social Media Feed (like Twitter/Instagram)

### 1. Requirements

```
┌─────────────────────────────────────────────────────────────────┐
│                    SOCIAL FEED REQUIREMENTS                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  FUNCTIONAL:                                                     │
│  • Create posts (text, images, videos)                          │
│  • Follow/unfollow users                                        │
│  • View personalized feed                                       │
│  • Like and comment on posts                                    │
│  • Share/repost                                                 │
│  • Search posts and users                                       │
│                                                                  │
│  NON-FUNCTIONAL:                                                 │
│  • Feed generation < 500ms                                      │
│  • Eventually consistent (seconds, not minutes)                 │
│  • Handle viral content (millions of views)                     │
│  • Support celebrities (millions of followers)                  │
│                                                                  │
│  SCALE:                                                          │
│  • 500M daily active users                                      │
│  • Average user follows 200 people                              │
│  • 1M new posts per hour                                        │
│  • Average celebrity has 1M+ followers                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 2. Feed Generation Strategies

```
┌─────────────────────────────────────────────────────────────────┐
│                  FEED GENERATION STRATEGIES                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. PULL MODEL (Fan-out on Read)                                │
│     ┌────────────────────────────────────────────────────────┐  │
│     │ User opens app → Query all followed users' posts →     │  │
│     │ Merge and rank → Return feed                           │  │
│     └────────────────────────────────────────────────────────┘  │
│     ✓ Simple, no pre-computation                                │
│     ✓ Good for users who follow few people                     │
│     ✗ Slow for users following many people                     │
│     ✗ High read latency                                        │
│                                                                  │
│  2. PUSH MODEL (Fan-out on Write)                               │
│     ┌────────────────────────────────────────────────────────┐  │
│     │ User posts → Push to all followers' feeds →            │  │
│     │ Followers open app → Read pre-built feed               │  │
│     └────────────────────────────────────────────────────────┘  │
│     ✓ Fast reads (feed is pre-built)                           │
│     ✓ Good for users with few followers                        │
│     ✗ Expensive for celebrities (millions of writes)           │
│     ✗ Waste for inactive users                                 │
│                                                                  │
│  3. HYBRID MODEL (Best of Both)                                 │
│     ┌────────────────────────────────────────────────────────┐  │
│     │ Regular users → Push model                              │  │
│     │ Celebrities (>10K followers) → Pull model at read time │  │
│     │ Merge both at feed generation                          │  │
│     └────────────────────────────────────────────────────────┘  │
│     ✓ Balances write and read load                             │
│     ✓ Handles celebrities efficiently                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 3. Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    SOCIAL FEED ARCHITECTURE                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│                           ┌─────────────┐                       │
│                           │    CDN      │                       │
│                           │  (Images)   │                       │
│                           └──────┬──────┘                       │
│                                  │                               │
│  ┌──────────┐     ┌─────────────┴───────────────┐               │
│  │  Client  │────▶│       Load Balancer         │               │
│  └──────────┘     └─────────────┬───────────────┘               │
│                                 │                                │
│            ┌────────────────────┼────────────────────┐          │
│            ▼                    ▼                    ▼          │
│     ┌───────────┐        ┌───────────┐        ┌───────────┐    │
│     │   Post    │        │   Feed    │        │   User    │    │
│     │  Service  │        │  Service  │        │  Service  │    │
│     └─────┬─────┘        └─────┬─────┘        └─────┬─────┘    │
│           │                    │                    │           │
│           ▼                    ▼                    ▼           │
│     ┌───────────┐        ┌───────────┐        ┌───────────┐    │
│     │   Post    │        │Feed Cache │        │   User    │    │
│     │    DB     │        │  (Redis)  │        │    DB     │    │
│     └───────────┘        └───────────┘        └───────────┘    │
│           │                                                     │
│           ▼                                                     │
│     ┌───────────────────────────────────────────────────────┐  │
│     │              Fan-out Worker Service                    │  │
│     │         (Push posts to follower feeds)                │  │
│     └───────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Implementation

```javascript
// Post Service
class PostService {
  constructor(db, cache, fanoutQueue, searchIndex) {
    this.db = db;
    this.cache = cache;
    this.fanoutQueue = fanoutQueue;
    this.searchIndex = searchIndex;
  }

  async createPost(userId, content, mediaUrls = []) {
    // Generate snowflake ID for ordering
    const postId = this.generateSnowflakeId();

    // Store post
    const post = await this.db.posts.create({
      data: {
        id: postId,
        userId,
        content,
        mediaUrls,
        createdAt: new Date(),
        likesCount: 0,
        commentsCount: 0,
        sharesCount: 0
      }
    });

    // Cache the post
    await this.cache.set(`post:${postId}`, JSON.stringify(post), 86400);

    // Index for search
    await this.searchIndex.index('posts', {
      id: postId,
      content,
      userId,
      createdAt: post.createdAt
    });

    // Queue fan-out to followers
    await this.fanoutQueue.add('fanout', {
      postId,
      userId,
      createdAt: post.createdAt
    });

    return post;
  }
}

// Fan-out Worker
class FanoutWorker {
  constructor(db, cache) {
    this.db = db;
    this.cache = cache;
    this.celebrityThreshold = 10000;
  }

  async process(job) {
    const { postId, userId, createdAt } = job.data;

    // Get user's follower count
    const user = await this.db.users.findUnique({
      where: { id: userId },
      select: { followersCount: true }
    });

    // Skip fan-out for celebrities (use pull model)
    if (user.followersCount > this.celebrityThreshold) {
      // Mark post as celebrity post for pull-based retrieval
      await this.cache.sAdd('celebrity-posts', `${userId}:${postId}`);
      return;
    }

    // Fan-out to all followers
    const batchSize = 1000;
    let cursor = null;

    do {
      const followers = await this.db.follows.findMany({
        where: { followingId: userId },
        take: batchSize,
        cursor: cursor ? { id: cursor } : undefined,
        select: { followerId: true, id: true }
      });

      if (followers.length === 0) break;

      // Push to each follower's feed
      const pipeline = this.cache.pipeline();

      for (const follower of followers) {
        // Add to sorted set (score = timestamp for ordering)
        pipeline.zAdd(
          `feed:${follower.followerId}`,
          createdAt.getTime(),
          postId
        );

        // Trim to keep only recent posts
        pipeline.zRemRangeByRank(`feed:${follower.followerId}`, 0, -1001);
      }

      await pipeline.exec();

      cursor = followers[followers.length - 1].id;
    } while (true);
  }
}

// Feed Service
class FeedService {
  constructor(db, cache, postService) {
    this.db = db;
    this.cache = cache;
    this.postService = postService;
    this.feedSize = 100;
  }

  async getFeed(userId, cursor = null, limit = 20) {
    // Get pre-computed feed from cache
    const cachedPostIds = await this.cache.zRange(
      `feed:${userId}`,
      cursor ? cursor + 1 : 0,
      limit,
      { REV: true }
    );

    // Get celebrity posts (pull model)
    const celebrityPosts = await this.getCelebrityPosts(userId);

    // Merge and deduplicate
    const allPostIds = [...new Set([...cachedPostIds, ...celebrityPosts])];

    // Fetch full post data
    const posts = await this.getPostsByIds(allPostIds);

    // Sort by creation time
    posts.sort((a, b) => b.createdAt - a.createdAt);

    // Apply ranking algorithm
    const rankedPosts = this.rankPosts(posts, userId);

    return rankedPosts.slice(0, limit);
  }

  async getCelebrityPosts(userId) {
    // Get celebrities the user follows
    const following = await this.db.follows.findMany({
      where: {
        followerId: userId,
        following: {
          followersCount: { gt: 10000 }
        }
      },
      select: { followingId: true }
    });

    if (following.length === 0) return [];

    // Get recent posts from celebrities
    const recentPosts = await this.db.posts.findMany({
      where: {
        userId: { in: following.map(f => f.followingId) },
        createdAt: { gt: new Date(Date.now() - 24 * 60 * 60 * 1000) }
      },
      select: { id: true },
      orderBy: { createdAt: 'desc' },
      take: 50
    });

    return recentPosts.map(p => p.id);
  }

  async getPostsByIds(postIds) {
    const posts = [];

    // Try cache first
    const cacheKeys = postIds.map(id => `post:${id}`);
    const cached = await this.cache.mGet(cacheKeys);

    const missingIds = [];
    cached.forEach((post, index) => {
      if (post) {
        posts.push(JSON.parse(post));
      } else {
        missingIds.push(postIds[index]);
      }
    });

    // Fetch missing from database
    if (missingIds.length > 0) {
      const dbPosts = await this.db.posts.findMany({
        where: { id: { in: missingIds } },
        include: {
          user: {
            select: { id: true, username: true, avatarUrl: true }
          }
        }
      });

      // Cache and add to results
      const pipeline = this.cache.pipeline();
      for (const post of dbPosts) {
        pipeline.set(`post:${post.id}`, JSON.stringify(post), 'EX', 86400);
        posts.push(post);
      }
      await pipeline.exec();
    }

    return posts;
  }

  rankPosts(posts, userId) {
    // Simple ranking algorithm
    // In production, use ML models for personalization
    return posts.map(post => {
      let score = 0;

      // Recency
      const ageHours = (Date.now() - post.createdAt) / (1000 * 60 * 60);
      score += Math.max(0, 100 - ageHours * 2);

      // Engagement
      score += Math.log(post.likesCount + 1) * 10;
      score += Math.log(post.commentsCount + 1) * 15;
      score += Math.log(post.sharesCount + 1) * 20;

      // Relationship strength (would need more data)
      // score += relationshipScore * 20;

      return { ...post, score };
    }).sort((a, b) => b.score - a.score);
  }
}
```

### Database Schema

```sql
-- Posts table (sharded by user_id)
CREATE TABLE posts (
  id BIGINT PRIMARY KEY,  -- Snowflake ID
  user_id BIGINT NOT NULL,
  content TEXT,
  media_urls JSONB,
  likes_count INT DEFAULT 0,
  comments_count INT DEFAULT 0,
  shares_count INT DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW(),
  INDEX idx_user_posts (user_id, created_at DESC)
);

-- Follows table
CREATE TABLE follows (
  id BIGINT PRIMARY KEY,
  follower_id BIGINT NOT NULL,
  following_id BIGINT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE (follower_id, following_id),
  INDEX idx_followers (following_id),
  INDEX idx_following (follower_id)
);

-- Likes table
CREATE TABLE likes (
  post_id BIGINT,
  user_id BIGINT,
  created_at TIMESTAMP DEFAULT NOW(),
  PRIMARY KEY (post_id, user_id)
);

-- Comments table
CREATE TABLE comments (
  id BIGINT PRIMARY KEY,
  post_id BIGINT NOT NULL,
  user_id BIGINT NOT NULL,
  content TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  INDEX idx_post_comments (post_id, created_at DESC)
);
```

---

## System Design Interview Tips

```
┌─────────────────────────────────────────────────────────────────┐
│                    INTERVIEW TIPS                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  DO:                                                             │
│  ✓ Clarify requirements before designing                        │
│  ✓ Think out loud - explain your reasoning                      │
│  ✓ Start simple, then scale                                     │
│  ✓ Draw diagrams as you go                                      │
│  ✓ Discuss trade-offs explicitly                                │
│  ✓ Consider failure scenarios                                   │
│  ✓ Ask about priorities (latency vs consistency, etc.)          │
│                                                                  │
│  DON'T:                                                          │
│  ✗ Jump into implementation details too quickly                 │
│  ✗ Ignore non-functional requirements                           │
│  ✗ Design in silence                                             │
│  ✗ Forget about data storage                                    │
│  ✗ Overlook security and privacy                                │
│  ✗ Assume infinite resources                                    │
│                                                                  │
│  COMMON TOPICS:                                                  │
│  • URL Shortener                                                 │
│  • Chat/Messaging System                                         │
│  • Social Media Feed                                             │
│  • Rate Limiter                                                  │
│  • Search Autocomplete                                          │
│  • Notification System                                          │
│  • Video Streaming (Netflix)                                    │
│  • Ride Sharing (Uber)                                          │
│  • E-commerce (Amazon)                                          │
│  • File Storage (Dropbox)                                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Practice Exercises

### Exercise 1: Rate Limiter

Design a distributed rate limiter that:
- Supports multiple rate limit rules
- Works across multiple servers
- Handles 100K requests/second
- Returns remaining quota in headers

### Exercise 2: Notification System

Design a notification system that:
- Supports push, email, SMS
- Handles user preferences
- Batches notifications intelligently
- Scales to 1M users

### Exercise 3: Search Autocomplete

Design a search autocomplete that:
- Returns suggestions in < 100ms
- Personalizes based on user history
- Handles 10K queries/second
- Updates with trending searches

---

## Key Takeaways

1. **Start with requirements** - Understand before you design
2. **Estimate scale** - Numbers drive architecture decisions
3. **Design iteratively** - Simple first, then optimize
4. **Know the trade-offs** - Every decision has consequences
5. **Practice regularly** - System design is a skill that improves with practice
6. **Think about failure** - Systems fail; plan for it

---

## Conclusion

Congratulations! You've completed the System Design section. You now have the knowledge to:
- Scale applications horizontally and vertically
- Choose appropriate database strategies
- Implement effective caching
- Build event-driven architectures
- Design microservices systems
- Approach system design interviews with confidence

Remember: great system design comes from understanding trade-offs and making informed decisions based on specific requirements. Keep practicing and building!
