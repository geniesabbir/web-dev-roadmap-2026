# Day 4: Message Queues - Asynchronous Communication

## Introduction

Message queues are essential for building scalable, resilient distributed systems. They enable asynchronous communication between services, handle traffic spikes gracefully, and ensure reliable message delivery even when components fail. Today, you'll learn how to implement queues with Redis, understand queue patterns, and know when to use different messaging technologies.

## Learning Objectives

By the end of this lesson, you will be able to:
- Understand message queue concepts and patterns
- Implement queues with Redis (Bull/BullMQ)
- Design reliable job processing systems
- Handle failures and retries gracefully
- Choose the right queue technology for your needs

---

## Why Message Queues?

```
┌─────────────────────────────────────────────────────────────────┐
│              SYNCHRONOUS VS ASYNCHRONOUS                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  SYNCHRONOUS (Without Queue):                                    │
│  ┌────────┐  ──────────▶  ┌────────────┐  ──────▶  ┌──────────┐ │
│  │ Client │   Request    │   Server    │  Email   │Email Svc │ │
│  └────────┘  ◀──────────  └────────────┘  ◀──────  └──────────┘ │
│                Response     2-5 seconds                          │
│                                                                  │
│  Problems: ❌ Slow response, ❌ Tight coupling, ❌ No fault tolerance │
│                                                                  │
│  ─────────────────────────────────────────────────────────────── │
│                                                                  │
│  ASYNCHRONOUS (With Queue):                                      │
│  ┌────────┐  ──────▶  ┌────────┐  ──────▶  ┌───────┐            │
│  │ Client │  Request │ Server │   Job    │ Queue │             │
│  └────────┘  ◀──────  └────────┘          └───┬───┘            │
│               ~50ms                            │                 │
│                                                ▼                 │
│                              ┌──────────┐  ┌──────────┐         │
│                              │ Worker 1 │  │ Worker 2 │         │
│                              └──────────┘  └──────────┘         │
│                                    │                             │
│                                    ▼                             │
│                              ┌──────────┐                        │
│                              │Email Svc │                        │
│                              └──────────┘                        │
│                                                                  │
│  Benefits: ✓ Fast response, ✓ Loose coupling, ✓ Fault tolerant  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Use Cases for Message Queues

```
┌─────────────────────────────────────────────────────────────────┐
│                    COMMON USE CASES                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  📧 EMAIL NOTIFICATIONS                                          │
│  User signs up → Queue email job → Worker sends welcome email    │
│                                                                  │
│  📸 IMAGE PROCESSING                                             │
│  Upload image → Queue resize jobs → Workers create thumbnails    │
│                                                                  │
│  📊 DATA PROCESSING                                              │
│  Import CSV → Queue row processing → Workers process in parallel │
│                                                                  │
│  💳 PAYMENT PROCESSING                                           │
│  Order placed → Queue payment → Worker processes with retry      │
│                                                                  │
│  🔔 PUSH NOTIFICATIONS                                           │
│  Event occurs → Queue notification → Workers send to devices     │
│                                                                  │
│  📈 ANALYTICS                                                    │
│  User action → Queue event → Workers aggregate and store         │
│                                                                  │
│  🔄 WEBHOOKS                                                     │
│  Event triggered → Queue webhook → Worker delivers with retry    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Queue Patterns

### Simple Queue (Point-to-Point)

```
┌─────────────────────────────────────────────────────────────────┐
│                     SIMPLE QUEUE                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Producer                Queue                Consumer           │
│  ┌────────┐           ┌─────────┐           ┌────────┐          │
│  │ API    │──────────▶│ ████████│──────────▶│ Worker │          │
│  │ Server │   Job     │ ███████ │   Job     │        │          │
│  └────────┘           │ ██████  │           └────────┘          │
│                       └─────────┘                                │
│                                                                  │
│  • One producer, one consumer per message                        │
│  • First-in, first-out (FIFO)                                    │
│  • Message deleted after processing                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Competing Consumers

```
┌─────────────────────────────────────────────────────────────────┐
│                  COMPETING CONSUMERS                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Producer                Queue                Consumers          │
│  ┌────────┐           ┌─────────┐         ┌──────────┐          │
│  │ API    │──────────▶│ ████████│────────▶│ Worker 1 │          │
│  │ Server │   Jobs    │ ███████ │         └──────────┘          │
│  └────────┘           │ ██████  │         ┌──────────┐          │
│                       │ █████   │────────▶│ Worker 2 │          │
│                       └─────────┘         └──────────┘          │
│                            │              ┌──────────┐          │
│                            └─────────────▶│ Worker 3 │          │
│                                           └──────────┘          │
│                                                                  │
│  • Multiple workers process jobs in parallel                     │
│  • Each job processed by exactly one worker                      │
│  • Horizontal scaling by adding workers                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Pub/Sub (Publish-Subscribe)

```
┌─────────────────────────────────────────────────────────────────┐
│                    PUBLISH-SUBSCRIBE                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Publisher              Topic               Subscribers          │
│  ┌────────┐           ┌─────────┐         ┌──────────┐          │
│  │ Order  │──────────▶│  order  │────────▶│ Email    │          │
│  │ Service│  Event    │ created │         │ Service  │          │
│  └────────┘           │         │         └──────────┘          │
│                       │         │         ┌──────────┐          │
│                       │         │────────▶│ Inventory│          │
│                       │         │         │ Service  │          │
│                       │         │         └──────────┘          │
│                       │         │         ┌──────────┐          │
│                       │         │────────▶│Analytics │          │
│                       └─────────┘         │ Service  │          │
│                                           └──────────┘          │
│                                                                  │
│  • One event delivered to ALL subscribers                        │
│  • Services are decoupled                                        │
│  • Easy to add new subscribers                                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## BullMQ with Redis

### Setup

```bash
# Install dependencies
npm install bullmq ioredis

# Or with Bull (older, still popular)
npm install bull
```

### Basic Queue Implementation

```javascript
// queue-setup.js
const { Queue, Worker, QueueScheduler } = require('bullmq');
const IORedis = require('ioredis');

// Redis connection
const connection = new IORedis({
  host: process.env.REDIS_HOST || 'localhost',
  port: process.env.REDIS_PORT || 6379,
  maxRetriesPerRequest: null
});

// Create queue
const emailQueue = new Queue('email', { connection });

// Queue scheduler (handles delayed jobs and retries)
const scheduler = new QueueScheduler('email', { connection });

// Producer: Add jobs to queue
async function sendWelcomeEmail(userId, email) {
  const job = await emailQueue.add(
    'welcome-email',  // Job name
    {                 // Job data
      userId,
      email,
      template: 'welcome'
    },
    {                 // Job options
      attempts: 3,
      backoff: {
        type: 'exponential',
        delay: 1000  // 1s, 2s, 4s
      },
      removeOnComplete: 100,  // Keep last 100 completed
      removeOnFail: 1000      // Keep last 1000 failed
    }
  );

  console.log(`Created job ${job.id}`);
  return job;
}

// Consumer: Process jobs
const emailWorker = new Worker('email', async (job) => {
  console.log(`Processing job ${job.id}: ${job.name}`);

  const { userId, email, template } = job.data;

  // Update progress
  await job.updateProgress(10);

  // Simulate email sending
  await sendEmail(email, template);

  await job.updateProgress(100);

  return { sent: true, timestamp: new Date() };
}, {
  connection,
  concurrency: 5  // Process 5 jobs at once
});

// Event handlers
emailWorker.on('completed', (job, result) => {
  console.log(`Job ${job.id} completed:`, result);
});

emailWorker.on('failed', (job, error) => {
  console.error(`Job ${job.id} failed:`, error.message);
});

emailWorker.on('progress', (job, progress) => {
  console.log(`Job ${job.id} progress: ${progress}%`);
});

module.exports = { emailQueue, sendWelcomeEmail };
```

### Advanced Job Options

```javascript
// advanced-jobs.js
const { Queue } = require('bullmq');

const taskQueue = new Queue('tasks', { connection });

// Delayed job
await taskQueue.add('reminder', { userId: 123 }, {
  delay: 24 * 60 * 60 * 1000  // 24 hours from now
});

// Scheduled/Cron job
await taskQueue.add('daily-report', {}, {
  repeat: {
    cron: '0 9 * * *'  // Every day at 9 AM
  }
});

// Priority job (lower = higher priority)
await taskQueue.add('urgent', { type: 'critical' }, {
  priority: 1  // Will be processed before priority 2, 3, etc.
});

// Job with custom ID (prevents duplicates)
await taskQueue.add('user-sync', { userId: 123 }, {
  jobId: `user-sync-${123}`  // Same ID = same job
});

// Rate limited job
await taskQueue.add('api-call', { endpoint: '/data' }, {
  limiter: {
    max: 10,       // Max 10 jobs
    duration: 1000 // Per second
  }
});

// Job with dependencies
const job1 = await taskQueue.add('step1', { data: 'first' });
await taskQueue.add('step2', { data: 'second' }, {
  parent: {
    id: job1.id,
    queue: 'tasks'
  }
});

// Bulk add jobs
await taskQueue.addBulk([
  { name: 'email', data: { to: 'a@example.com' } },
  { name: 'email', data: { to: 'b@example.com' } },
  { name: 'email', data: { to: 'c@example.com' } }
]);
```

### Job Flows (Workflows)

```javascript
// job-flows.js
const { FlowProducer } = require('bullmq');

const flowProducer = new FlowProducer({ connection });

// Create a workflow: Process order
const flow = await flowProducer.add({
  name: 'complete-order',
  queueName: 'orders',
  data: { orderId: '12345' },
  children: [
    {
      name: 'charge-payment',
      queueName: 'payments',
      data: { orderId: '12345', amount: 99.99 }
    },
    {
      name: 'reserve-inventory',
      queueName: 'inventory',
      data: { orderId: '12345', items: ['SKU-001', 'SKU-002'] }
    },
    {
      name: 'send-confirmation',
      queueName: 'email',
      data: { orderId: '12345', email: 'customer@example.com' },
      opts: {
        delay: 1000  // Wait 1 second after parent jobs
      }
    }
  ]
});

// Parent job waits for all children to complete
// If any child fails, parent also fails
```

---

## Queue Architecture Patterns

### Worker Service

```javascript
// worker-service.js
const { Worker } = require('bullmq');

class WorkerService {
  constructor(queueName, processor, options = {}) {
    this.queueName = queueName;
    this.processor = processor;
    this.options = options;
    this.worker = null;
  }

  start() {
    this.worker = new Worker(
      this.queueName,
      this.processor,
      {
        connection,
        concurrency: this.options.concurrency || 1,
        limiter: this.options.limiter
      }
    );

    this.setupEventHandlers();
    console.log(`Worker started for queue: ${this.queueName}`);
  }

  setupEventHandlers() {
    this.worker.on('completed', (job, result) => {
      console.log(`✓ ${this.queueName}:${job.id} completed`);
      this.options.onCompleted?.(job, result);
    });

    this.worker.on('failed', (job, error) => {
      console.error(`✗ ${this.queueName}:${job.id} failed: ${error.message}`);
      this.options.onFailed?.(job, error);
    });

    this.worker.on('error', (error) => {
      console.error(`Worker error: ${error.message}`);
    });
  }

  async stop() {
    await this.worker?.close();
    console.log(`Worker stopped for queue: ${this.queueName}`);
  }
}

// Usage
const emailWorker = new WorkerService(
  'email',
  async (job) => {
    await sendEmail(job.data.to, job.data.subject, job.data.body);
    return { sent: true };
  },
  {
    concurrency: 10,
    onCompleted: (job) => {
      metrics.increment('emails.sent');
    },
    onFailed: (job, error) => {
      alerting.notify(`Email failed: ${error.message}`);
    }
  }
);

emailWorker.start();

// Graceful shutdown
process.on('SIGTERM', async () => {
  await emailWorker.stop();
  process.exit(0);
});
```

### Queue Manager

```javascript
// queue-manager.js
const { Queue, QueueScheduler, QueueEvents } = require('bullmq');

class QueueManager {
  constructor(connection) {
    this.connection = connection;
    this.queues = new Map();
    this.schedulers = new Map();
    this.events = new Map();
  }

  createQueue(name, options = {}) {
    if (this.queues.has(name)) {
      return this.queues.get(name);
    }

    const queue = new Queue(name, {
      connection: this.connection,
      defaultJobOptions: {
        attempts: 3,
        backoff: { type: 'exponential', delay: 1000 },
        removeOnComplete: 100,
        removeOnFail: 1000,
        ...options.defaultJobOptions
      }
    });

    // Create scheduler for delayed jobs
    const scheduler = new QueueScheduler(name, {
      connection: this.connection
    });

    // Create events listener
    const events = new QueueEvents(name, {
      connection: this.connection
    });

    this.queues.set(name, queue);
    this.schedulers.set(name, scheduler);
    this.events.set(name, events);

    return queue;
  }

  getQueue(name) {
    return this.queues.get(name);
  }

  async getQueueStats(name) {
    const queue = this.queues.get(name);
    if (!queue) return null;

    const [waiting, active, completed, failed, delayed] = await Promise.all([
      queue.getWaitingCount(),
      queue.getActiveCount(),
      queue.getCompletedCount(),
      queue.getFailedCount(),
      queue.getDelayedCount()
    ]);

    return { waiting, active, completed, failed, delayed };
  }

  async getAllStats() {
    const stats = {};
    for (const name of this.queues.keys()) {
      stats[name] = await this.getQueueStats(name);
    }
    return stats;
  }

  async closeAll() {
    for (const queue of this.queues.values()) {
      await queue.close();
    }
    for (const scheduler of this.schedulers.values()) {
      await scheduler.close();
    }
    for (const events of this.events.values()) {
      await events.close();
    }
  }
}

// Usage
const manager = new QueueManager(connection);

const emailQueue = manager.createQueue('email');
const imageQueue = manager.createQueue('image-processing', {
  defaultJobOptions: { attempts: 5 }
});

// Dashboard endpoint
app.get('/api/queues/stats', async (req, res) => {
  const stats = await manager.getAllStats();
  res.json(stats);
});
```

### Error Handling and Retries

```javascript
// error-handling.js
const { Worker } = require('bullmq');

// Custom error types
class RetryableError extends Error {
  constructor(message) {
    super(message);
    this.name = 'RetryableError';
  }
}

class PermanentError extends Error {
  constructor(message) {
    super(message);
    this.name = 'PermanentError';
  }
}

// Worker with error handling
const worker = new Worker('payments', async (job) => {
  try {
    const result = await processPayment(job.data);
    return result;
  } catch (error) {
    if (error.code === 'NETWORK_ERROR') {
      // Will be retried
      throw new RetryableError('Network temporarily unavailable');
    }

    if (error.code === 'INVALID_CARD') {
      // Move to dead letter queue, don't retry
      await deadLetterQueue.add('failed-payment', {
        originalJob: job.data,
        error: error.message,
        failedAt: new Date()
      });
      throw new PermanentError('Invalid card - not retrying');
    }

    throw error;
  }
}, {
  connection,
  settings: {
    backoffStrategies: {
      custom: (attemptsMade) => {
        // Custom backoff: 1s, 5s, 30s, 2min, 5min
        const delays = [1000, 5000, 30000, 120000, 300000];
        return delays[attemptsMade - 1] || 300000;
      }
    }
  }
});

// Job with custom backoff
await paymentQueue.add('process', paymentData, {
  attempts: 5,
  backoff: {
    type: 'custom'
  }
});

// Dead letter queue processing
const dlqWorker = new Worker('dead-letter', async (job) => {
  // Alert team about failed job
  await alertService.notify({
    type: 'failed-job',
    queue: job.data.originalQueue,
    data: job.data,
    error: job.data.error
  });

  // Store for manual review
  await db.failedJobs.create({
    data: job.data,
    createdAt: new Date()
  });
}, { connection });
```

---

## Pub/Sub with Redis

```javascript
// pubsub.js
const Redis = require('ioredis');

class PubSubService {
  constructor() {
    this.publisher = new Redis();
    this.subscriber = new Redis();
    this.handlers = new Map();
  }

  async publish(channel, message) {
    const payload = JSON.stringify({
      data: message,
      timestamp: Date.now(),
      id: crypto.randomUUID()
    });

    await this.publisher.publish(channel, payload);
    console.log(`Published to ${channel}`);
  }

  subscribe(channel, handler) {
    // Store handler
    if (!this.handlers.has(channel)) {
      this.handlers.set(channel, []);
    }
    this.handlers.get(channel).push(handler);

    // Subscribe if first handler
    if (this.handlers.get(channel).length === 1) {
      this.subscriber.subscribe(channel);
    }
  }

  start() {
    this.subscriber.on('message', async (channel, message) => {
      const handlers = this.handlers.get(channel) || [];
      const payload = JSON.parse(message);

      for (const handler of handlers) {
        try {
          await handler(payload.data, payload);
        } catch (error) {
          console.error(`Handler error for ${channel}:`, error);
        }
      }
    });
  }
}

// Usage
const pubsub = new PubSubService();
pubsub.start();

// Subscribe to events
pubsub.subscribe('order:created', async (order) => {
  console.log('New order:', order.id);
  await sendOrderConfirmation(order);
});

pubsub.subscribe('order:created', async (order) => {
  console.log('Updating inventory for order:', order.id);
  await updateInventory(order.items);
});

// Publish event
await pubsub.publish('order:created', {
  id: 'order-123',
  userId: 'user-456',
  items: ['SKU-001', 'SKU-002'],
  total: 99.99
});
```

---

## Real-World Examples

### Email Service

```javascript
// email-service.js
const { Queue, Worker } = require('bullmq');
const nodemailer = require('nodemailer');

// Email queue
const emailQueue = new Queue('email', {
  connection,
  defaultJobOptions: {
    attempts: 3,
    backoff: { type: 'exponential', delay: 5000 }
  }
});

// Email templates
const templates = {
  welcome: (data) => ({
    subject: `Welcome to ${data.appName}!`,
    html: `<h1>Welcome ${data.name}!</h1><p>Thanks for joining...</p>`
  }),
  passwordReset: (data) => ({
    subject: 'Reset your password',
    html: `<p>Click <a href="${data.resetLink}">here</a> to reset...</p>`
  }),
  orderConfirmation: (data) => ({
    subject: `Order #${data.orderId} confirmed`,
    html: `<h1>Thanks for your order!</h1>...`
  })
};

// Producer functions
const EmailService = {
  async sendWelcome(userId, email, name) {
    await emailQueue.add('welcome', {
      to: email,
      template: 'welcome',
      data: { name, appName: 'MyApp' }
    });
  },

  async sendPasswordReset(email, resetToken) {
    await emailQueue.add('password-reset', {
      to: email,
      template: 'passwordReset',
      data: {
        resetLink: `https://app.com/reset?token=${resetToken}`
      }
    }, {
      priority: 1  // High priority
    });
  },

  async sendOrderConfirmation(order) {
    await emailQueue.add('order-confirmation', {
      to: order.email,
      template: 'orderConfirmation',
      data: order
    });
  }
};

// Worker
const transporter = nodemailer.createTransporter({
  host: process.env.SMTP_HOST,
  port: 587,
  auth: {
    user: process.env.SMTP_USER,
    pass: process.env.SMTP_PASS
  }
});

const emailWorker = new Worker('email', async (job) => {
  const { to, template, data } = job.data;
  const { subject, html } = templates[template](data);

  await transporter.sendMail({
    from: 'noreply@myapp.com',
    to,
    subject,
    html
  });

  return { sent: true, to };
}, {
  connection,
  concurrency: 10
});

module.exports = EmailService;
```

### Image Processing Pipeline

```javascript
// image-processor.js
const { Queue, Worker, FlowProducer } = require('bullmq');
const sharp = require('sharp');
const { S3Client, PutObjectCommand } = require('@aws-sdk/client-s3');

const imageQueue = new Queue('images', { connection });
const flowProducer = new FlowProducer({ connection });

// Start image processing pipeline
async function processUploadedImage(imageBuffer, fileName, userId) {
  // Create processing flow
  await flowProducer.add({
    name: 'complete-processing',
    queueName: 'images',
    data: { fileName, userId, status: 'completed' },
    children: [
      {
        name: 'create-thumbnail',
        queueName: 'images',
        data: {
          buffer: imageBuffer.toString('base64'),
          size: { width: 150, height: 150 },
          output: `thumbnails/${fileName}`
        }
      },
      {
        name: 'create-medium',
        queueName: 'images',
        data: {
          buffer: imageBuffer.toString('base64'),
          size: { width: 800, height: 600 },
          output: `medium/${fileName}`
        }
      },
      {
        name: 'create-optimized',
        queueName: 'images',
        data: {
          buffer: imageBuffer.toString('base64'),
          quality: 80,
          output: `optimized/${fileName}`
        }
      }
    ]
  });
}

// S3 client
const s3 = new S3Client({ region: process.env.AWS_REGION });

// Image processing worker
const imageWorker = new Worker('images', async (job) => {
  const { buffer, size, quality, output } = job.data;

  if (job.name === 'complete-processing') {
    // Final step - update database
    await db.images.update({
      where: { fileName: job.data.fileName },
      data: { status: 'processed', processedAt: new Date() }
    });
    return { completed: true };
  }

  // Process image
  const imageBuffer = Buffer.from(buffer, 'base64');
  let processed = sharp(imageBuffer);

  if (size) {
    processed = processed.resize(size.width, size.height, {
      fit: 'cover'
    });
  }

  if (quality) {
    processed = processed.jpeg({ quality });
  }

  const outputBuffer = await processed.toBuffer();

  // Upload to S3
  await s3.send(new PutObjectCommand({
    Bucket: process.env.S3_BUCKET,
    Key: output,
    Body: outputBuffer,
    ContentType: 'image/jpeg'
  }));

  return { uploaded: output, size: outputBuffer.length };
}, {
  connection,
  concurrency: 3  // Limit concurrent processing
});

module.exports = { processUploadedImage };
```

### Webhook Delivery

```javascript
// webhook-service.js
const { Queue, Worker } = require('bullmq');
const crypto = require('crypto');

const webhookQueue = new Queue('webhooks', {
  connection,
  defaultJobOptions: {
    attempts: 5,
    backoff: {
      type: 'exponential',
      delay: 60000  // 1 min, 2 min, 4 min, 8 min, 16 min
    }
  }
});

// Queue a webhook delivery
async function deliverWebhook(subscription, event, payload) {
  const timestamp = Date.now();
  const signature = generateSignature(
    subscription.secret,
    timestamp,
    payload
  );

  await webhookQueue.add('deliver', {
    url: subscription.url,
    event,
    payload,
    timestamp,
    signature,
    subscriptionId: subscription.id
  }, {
    jobId: `${subscription.id}-${event}-${timestamp}`
  });
}

function generateSignature(secret, timestamp, payload) {
  const data = `${timestamp}.${JSON.stringify(payload)}`;
  return crypto.createHmac('sha256', secret).update(data).digest('hex');
}

// Webhook worker
const webhookWorker = new Worker('webhooks', async (job) => {
  const { url, event, payload, timestamp, signature } = job.data;

  const response = await fetch(url, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-Webhook-Event': event,
      'X-Webhook-Timestamp': timestamp.toString(),
      'X-Webhook-Signature': signature
    },
    body: JSON.stringify(payload),
    signal: AbortSignal.timeout(30000)  // 30 second timeout
  });

  if (!response.ok) {
    const error = new Error(`Webhook failed: ${response.status}`);
    error.statusCode = response.statusCode;

    // Don't retry 4xx errors (except 429)
    if (response.status >= 400 && response.status < 500 && response.status !== 429) {
      error.unrecoverable = true;
    }

    throw error;
  }

  // Log successful delivery
  await db.webhookDeliveries.create({
    data: {
      subscriptionId: job.data.subscriptionId,
      event,
      status: 'delivered',
      deliveredAt: new Date()
    }
  });

  return { status: response.status };
}, {
  connection,
  concurrency: 20
});

// Handle unrecoverable errors
webhookWorker.on('failed', async (job, error) => {
  if (job.attemptsMade >= job.opts.attempts || error.unrecoverable) {
    // Disable webhook after too many failures
    await db.webhookSubscriptions.update({
      where: { id: job.data.subscriptionId },
      data: { status: 'disabled', disabledReason: error.message }
    });

    // Notify subscription owner
    await notifyWebhookDisabled(job.data.subscriptionId);
  }
});

module.exports = { deliverWebhook };
```

---

## Queue Monitoring

```javascript
// queue-dashboard.js
const { Queue } = require('bullmq');

class QueueDashboard {
  constructor(queues) {
    this.queues = queues;
  }

  async getOverview() {
    const stats = await Promise.all(
      this.queues.map(async (queue) => {
        const [waiting, active, completed, failed, delayed, paused] =
          await Promise.all([
            queue.getWaitingCount(),
            queue.getActiveCount(),
            queue.getCompletedCount(),
            queue.getFailedCount(),
            queue.getDelayedCount(),
            queue.isPaused()
          ]);

        return {
          name: queue.name,
          waiting,
          active,
          completed,
          failed,
          delayed,
          paused,
          health: this.calculateHealth(failed, completed)
        };
      })
    );

    return stats;
  }

  calculateHealth(failed, completed) {
    if (completed === 0) return 'unknown';
    const failRate = failed / (failed + completed);
    if (failRate < 0.01) return 'healthy';
    if (failRate < 0.05) return 'warning';
    return 'critical';
  }

  async getFailedJobs(queueName, limit = 10) {
    const queue = this.queues.find(q => q.name === queueName);
    if (!queue) return [];

    const jobs = await queue.getFailed(0, limit - 1);
    return jobs.map(job => ({
      id: job.id,
      name: job.name,
      data: job.data,
      failedReason: job.failedReason,
      attemptsMade: job.attemptsMade,
      timestamp: job.timestamp
    }));
  }

  async retryFailed(queueName, jobId) {
    const queue = this.queues.find(q => q.name === queueName);
    if (!queue) throw new Error('Queue not found');

    const job = await queue.getJob(jobId);
    if (!job) throw new Error('Job not found');

    await job.retry();
    return { retried: true };
  }

  async pauseQueue(queueName) {
    const queue = this.queues.find(q => q.name === queueName);
    await queue?.pause();
  }

  async resumeQueue(queueName) {
    const queue = this.queues.find(q => q.name === queueName);
    await queue?.resume();
  }
}

// Express routes
const dashboard = new QueueDashboard([emailQueue, imageQueue, webhookQueue]);

app.get('/admin/queues', async (req, res) => {
  const stats = await dashboard.getOverview();
  res.json(stats);
});

app.get('/admin/queues/:name/failed', async (req, res) => {
  const jobs = await dashboard.getFailedJobs(req.params.name);
  res.json(jobs);
});

app.post('/admin/queues/:name/jobs/:id/retry', async (req, res) => {
  await dashboard.retryFailed(req.params.name, req.params.id);
  res.json({ success: true });
});
```

---

## Queue Technology Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                  QUEUE TECHNOLOGY COMPARISON                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  REDIS (BullMQ)                                                  │
│  ├─ Best for: Most Node.js apps, simple to complex workflows    │
│  ├─ Pros: Fast, flexible, good ecosystem                        │
│  ├─ Cons: In-memory (need persistence config)                   │
│  └─ Scale: Thousands of jobs/second                             │
│                                                                  │
│  RabbitMQ                                                        │
│  ├─ Best for: Complex routing, multiple consumers               │
│  ├─ Pros: Mature, reliable, feature-rich                        │
│  ├─ Cons: More complex setup                                    │
│  └─ Scale: Tens of thousands of messages/second                 │
│                                                                  │
│  AWS SQS                                                         │
│  ├─ Best for: AWS ecosystem, serverless                         │
│  ├─ Pros: Fully managed, scales automatically                   │
│  ├─ Cons: AWS lock-in, eventual consistency                     │
│  └─ Scale: Virtually unlimited                                  │
│                                                                  │
│  Kafka                                                           │
│  ├─ Best for: Event streaming, high throughput                  │
│  ├─ Pros: Extremely fast, durable, replayable                   │
│  ├─ Cons: Complex, overkill for simple queues                   │
│  └─ Scale: Millions of messages/second                          │
│                                                                  │
│  RECOMMENDATIONS:                                                │
│  • Starting out? Use Redis + BullMQ                              │
│  • Need managed? Use AWS SQS                                     │
│  • High throughput streaming? Consider Kafka                    │
│  • Complex routing? Consider RabbitMQ                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Practice Exercises

### Exercise 1: Email Queue

Build an email notification system:
- Queue different email types (welcome, password reset, etc.)
- Implement retry with exponential backoff
- Add rate limiting per email provider

### Exercise 2: Background Jobs

Create a file processing system:
- Upload files to queue for processing
- Process in parallel with concurrency limits
- Handle failures and notify users

### Exercise 3: Event System

Build a pub/sub event system:
- Publish events from API endpoints
- Multiple services subscribe to events
- Track event delivery status

---

## Key Takeaways

1. **Decouple with queues** - Separate producers from consumers
2. **Choose the right pattern** - Point-to-point, pub/sub, or flows
3. **Handle failures gracefully** - Implement retries with backoff
4. **Monitor queue health** - Track waiting, active, and failed jobs
5. **Scale horizontally** - Add workers to increase throughput
6. **Use dead letter queues** - Capture and analyze failed jobs

---

## What's Next?

Tomorrow, we'll explore **Microservices Architecture** - learning how to design, build, and deploy distributed systems with multiple independent services communicating over networks.
