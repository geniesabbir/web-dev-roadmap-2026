# Day 5: Monitoring - Tracking Application Health and Performance

## Introduction

Monitoring is critical for maintaining reliable applications. Without proper monitoring, you're flying blind—unable to detect issues until users complain. Today, you'll learn to implement comprehensive monitoring including logging, metrics, error tracking, and alerting to keep your applications healthy.

## Learning Objectives

By the end of this lesson, you will be able to:
- Implement structured logging
- Set up error tracking with Sentry
- Monitor performance metrics
- Configure alerting systems
- Create dashboards for visibility

---

## Monitoring Overview

### The Three Pillars of Observability

```
┌─────────────────────────────────────────────────────────────────┐
│                    Observability Pillars                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │     LOGS        │  │    METRICS      │  │    TRACES       │ │
│  │                 │  │                 │  │                 │ │
│  │ What happened?  │  │ How much?       │  │ Where?          │ │
│  │ Event records   │  │ Numeric data    │  │ Request flow    │ │
│  │                 │  │                 │  │                 │ │
│  │ • Error logs    │  │ • CPU usage     │  │ • Request path  │ │
│  │ • Access logs   │  │ • Memory        │  │ • Service calls │ │
│  │ • Audit logs    │  │ • Response time │  │ • Latency       │ │
│  │ • Debug logs    │  │ • Error rate    │  │ • Bottlenecks   │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘ │
│                                                                  │
│  Together, they answer: What's happening in my application?     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Monitoring Stack

```
┌─────────────────────────────────────────────────────────────────┐
│                    Modern Monitoring Stack                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Application                                                     │
│      │                                                          │
│      ├── Logs ──────────► Logtail / CloudWatch / Datadog       │
│      │                                                          │
│      ├── Errors ────────► Sentry / Bugsnag                     │
│      │                                                          │
│      ├── Metrics ───────► Prometheus / Datadog / CloudWatch    │
│      │                                                          │
│      └── Traces ────────► Jaeger / Datadog APM                 │
│                                                                  │
│  All feed into → Dashboard (Grafana / Datadog)                 │
│              → Alerts (PagerDuty / Slack / Email)              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Logging

### Structured Logging with Pino

```javascript
// logger.js
const pino = require('pino');

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  transport: process.env.NODE_ENV === 'development'
    ? { target: 'pino-pretty' }
    : undefined,
  base: {
    env: process.env.NODE_ENV,
    version: process.env.APP_VERSION
  }
});

module.exports = logger;
```

```javascript
// Usage in application
const logger = require('./logger');

// Info log
logger.info({ userId: 123 }, 'User logged in');

// Error log
logger.error({ err: error, userId: 123 }, 'Failed to process payment');

// Debug log
logger.debug({ query: sql }, 'Database query');

// Child logger with context
const requestLogger = logger.child({ requestId: 'abc123' });
requestLogger.info('Processing request');
```

### Winston Logger

```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: { service: 'api' },
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});

// Development pretty printing
if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple()
  }));
}
```

### Request Logging Middleware

```javascript
const pinoHttp = require('pino-http');
const logger = require('./logger');

// Express middleware
app.use(pinoHttp({
  logger,
  customLogLevel: (req, res, err) => {
    if (res.statusCode >= 500 || err) return 'error';
    if (res.statusCode >= 400) return 'warn';
    return 'info';
  },
  customSuccessMessage: (req, res) => {
    return `${req.method} ${req.url} ${res.statusCode}`;
  },
  customProps: (req) => ({
    userId: req.user?.id,
    userAgent: req.headers['user-agent']
  })
}));
```

### Log Levels

```
┌─────────────────────────────────────────────────────────────────┐
│                        Log Levels                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Level     │ When to Use                                        │
│  ──────────┼───────────────────────────────────────────────────│
│  fatal     │ Application crash, unrecoverable                   │
│  error     │ Runtime errors, exceptions                         │
│  warn      │ Potential problems, deprecated features            │
│  info      │ Important events, state changes                    │
│  debug     │ Detailed debugging information                     │
│  trace     │ Very detailed tracing                              │
│                                                                  │
│  Production: info and above                                     │
│  Development: debug and above                                   │
│  Troubleshooting: trace                                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Error Tracking with Sentry

### Setup Sentry

```bash
npm install @sentry/node @sentry/tracing
```

```javascript
// sentry.js
const Sentry = require('@sentry/node');
const { ProfilingIntegration } = require('@sentry/profiling-node');

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  release: process.env.APP_VERSION,

  integrations: [
    new Sentry.Integrations.Http({ tracing: true }),
    new Sentry.Integrations.Express({ app }),
    new ProfilingIntegration()
  ],

  // Performance monitoring
  tracesSampleRate: 1.0,
  profilesSampleRate: 1.0,

  // Filter sensitive data
  beforeSend(event) {
    if (event.request?.headers) {
      delete event.request.headers.authorization;
    }
    return event;
  }
});
```

### Express Integration

```javascript
const express = require('express');
const Sentry = require('@sentry/node');

const app = express();

// Request handler must be first
app.use(Sentry.Handlers.requestHandler());

// Tracing handler
app.use(Sentry.Handlers.tracingHandler());

// Your routes
app.get('/', (req, res) => {
  res.send('Hello');
});

// Error handler must be before any other error middleware
app.use(Sentry.Handlers.errorHandler());

// Custom error handler
app.use((err, req, res, next) => {
  res.status(500).json({ error: 'Internal server error' });
});
```

### Manual Error Capture

```javascript
// Capture exception
try {
  await riskyOperation();
} catch (error) {
  Sentry.captureException(error, {
    tags: { operation: 'risky' },
    extra: { userId: user.id }
  });
}

// Capture message
Sentry.captureMessage('Something went wrong', 'warning');

// Add context
Sentry.setUser({ id: user.id, email: user.email });
Sentry.setTag('feature', 'checkout');
Sentry.setExtra('cart', cartItems);
```

### Next.js Integration

```javascript
// sentry.client.config.js
import * as Sentry from '@sentry/nextjs';

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  tracesSampleRate: 1.0
});

// sentry.server.config.js
import * as Sentry from '@sentry/nextjs';

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  tracesSampleRate: 1.0
});
```

---

## Application Metrics

### Custom Metrics with prom-client

```javascript
const client = require('prom-client');

// Enable default metrics
client.collectDefaultMetrics();

// Custom counter
const httpRequestsTotal = new client.Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'path', 'status']
});

// Custom histogram
const httpRequestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration',
  labelNames: ['method', 'path'],
  buckets: [0.1, 0.5, 1, 2, 5]
});

// Custom gauge
const activeConnections = new client.Gauge({
  name: 'active_connections',
  help: 'Number of active connections'
});

// Middleware
app.use((req, res, next) => {
  const end = httpRequestDuration.startTimer();

  res.on('finish', () => {
    httpRequestsTotal.inc({
      method: req.method,
      path: req.route?.path || 'unknown',
      status: res.statusCode
    });
    end({ method: req.method, path: req.route?.path || 'unknown' });
  });

  next();
});

// Metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', client.register.contentType);
  res.end(await client.register.metrics());
});
```

### Key Metrics to Track

```
┌─────────────────────────────────────────────────────────────────┐
│                    Essential Metrics                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  RED Method (Request-focused):                                  │
│  • Rate      - Requests per second                              │
│  • Errors    - Error rate                                       │
│  • Duration  - Response time                                    │
│                                                                  │
│  USE Method (Resource-focused):                                 │
│  • Utilization - CPU, memory usage                              │
│  • Saturation  - Queue depth, load                              │
│  • Errors      - Hardware/software errors                       │
│                                                                  │
│  Business Metrics:                                               │
│  • User signups per hour                                        │
│  • Orders processed                                             │
│  • Revenue per minute                                           │
│  • Active users                                                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Uptime Monitoring

### Health Check Endpoint

```javascript
// health.js
app.get('/health', async (req, res) => {
  const health = {
    status: 'healthy',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    checks: {}
  };

  // Database check
  try {
    await db.query('SELECT 1');
    health.checks.database = 'healthy';
  } catch (error) {
    health.checks.database = 'unhealthy';
    health.status = 'unhealthy';
  }

  // Redis check
  try {
    await redis.ping();
    health.checks.redis = 'healthy';
  } catch (error) {
    health.checks.redis = 'unhealthy';
    health.status = 'unhealthy';
  }

  // Memory check
  const memUsage = process.memoryUsage();
  health.checks.memory = {
    heapUsed: Math.round(memUsage.heapUsed / 1024 / 1024) + 'MB',
    heapTotal: Math.round(memUsage.heapTotal / 1024 / 1024) + 'MB'
  };

  const statusCode = health.status === 'healthy' ? 200 : 503;
  res.status(statusCode).json(health);
});
```

### External Monitoring Services

```bash
# Popular uptime monitors:
# - UptimeRobot (free tier available)
# - Pingdom
# - Better Uptime
# - Checkly

# Set up checks for:
# - Homepage (GET https://example.com)
# - API health (GET https://api.example.com/health)
# - Critical endpoints
```

---

## Alerting

### Alert Rules

```yaml
# Example Prometheus alert rules
groups:
  - name: application
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: High error rate detected

      - alert: SlowResponses
        expr: histogram_quantile(0.95, http_request_duration_seconds_bucket) > 2
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: 95th percentile response time > 2s

      - alert: HighMemoryUsage
        expr: process_resident_memory_bytes / 1024 / 1024 > 500
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: Memory usage > 500MB
```

### Slack Alerting

```javascript
// alerts.js
const axios = require('axios');

async function sendSlackAlert(message, severity = 'warning') {
  const color = severity === 'critical' ? '#FF0000' : '#FFA500';

  await axios.post(process.env.SLACK_WEBHOOK_URL, {
    attachments: [{
      color,
      title: `${severity.toUpperCase()} Alert`,
      text: message,
      footer: 'Monitoring System',
      ts: Math.floor(Date.now() / 1000)
    }]
  });
}

// Usage
sendSlackAlert('High error rate detected on API', 'critical');
```

### Email Alerts (SendGrid)

```javascript
const sgMail = require('@sendgrid/mail');
sgMail.setApiKey(process.env.SENDGRID_API_KEY);

async function sendAlertEmail(subject, message) {
  await sgMail.send({
    to: 'oncall@example.com',
    from: 'alerts@example.com',
    subject: `[ALERT] ${subject}`,
    html: `
      <h2>Alert: ${subject}</h2>
      <p>${message}</p>
      <p>Time: ${new Date().toISOString()}</p>
    `
  });
}
```

---

## Performance Monitoring

### Web Vitals (Frontend)

```javascript
// web-vitals.js
import { getCLS, getFID, getFCP, getLCP, getTTFB } from 'web-vitals';

function sendToAnalytics(metric) {
  const body = JSON.stringify({
    name: metric.name,
    value: metric.value,
    id: metric.id
  });

  // Send to your analytics endpoint
  navigator.sendBeacon('/api/analytics', body);
}

// Track Core Web Vitals
getCLS(sendToAnalytics);
getFID(sendToAnalytics);
getFCP(sendToAnalytics);
getLCP(sendToAnalytics);
getTTFB(sendToAnalytics);
```

### API Performance

```javascript
// Track slow queries
const start = Date.now();
const result = await db.query(sql);
const duration = Date.now() - start;

if (duration > 1000) {
  logger.warn({ sql, duration }, 'Slow query detected');
  metrics.slowQueries.inc({ query: 'users_list' });
}
```

### N+1 Query Detection

```javascript
// Using Prisma middleware
prisma.$use(async (params, next) => {
  const start = Date.now();
  const result = await next(params);
  const duration = Date.now() - start;

  if (duration > 100) {
    logger.warn({
      model: params.model,
      action: params.action,
      duration
    }, 'Slow database operation');
  }

  return result;
});
```

---

## Dashboard Setup

### Grafana Dashboard (JSON)

```json
{
  "title": "Application Dashboard",
  "panels": [
    {
      "title": "Request Rate",
      "type": "graph",
      "targets": [{
        "expr": "rate(http_requests_total[5m])"
      }]
    },
    {
      "title": "Error Rate",
      "type": "graph",
      "targets": [{
        "expr": "rate(http_requests_total{status=~\"5..\"}[5m])"
      }]
    },
    {
      "title": "Response Time (p95)",
      "type": "gauge",
      "targets": [{
        "expr": "histogram_quantile(0.95, http_request_duration_seconds_bucket)"
      }]
    }
  ]
}
```

### Vercel Analytics

```javascript
// Next.js with Vercel Analytics
import { Analytics } from '@vercel/analytics/react';

export default function App({ Component, pageProps }) {
  return (
    <>
      <Component {...pageProps} />
      <Analytics />
    </>
  );
}
```

---

## Complete Monitoring Setup

```javascript
// monitoring.js - Complete setup
const Sentry = require('@sentry/node');
const pino = require('pino');
const client = require('prom-client');

// Initialize Sentry
Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV
});

// Initialize logger
const logger = pino({
  level: process.env.LOG_LEVEL || 'info'
});

// Initialize metrics
client.collectDefaultMetrics();

const httpRequestsTotal = new client.Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'path', 'status']
});

// Export monitoring middleware
module.exports = function monitoringMiddleware(app) {
  // Sentry request handler
  app.use(Sentry.Handlers.requestHandler());

  // Request logging and metrics
  app.use((req, res, next) => {
    const start = Date.now();

    res.on('finish', () => {
      const duration = Date.now() - start;

      // Log request
      logger.info({
        method: req.method,
        url: req.url,
        status: res.statusCode,
        duration,
        userAgent: req.headers['user-agent']
      });

      // Record metric
      httpRequestsTotal.inc({
        method: req.method,
        path: req.route?.path || 'unknown',
        status: res.statusCode
      });
    });

    next();
  });

  // Health endpoint
  app.get('/health', async (req, res) => {
    res.json({ status: 'healthy' });
  });

  // Metrics endpoint
  app.get('/metrics', async (req, res) => {
    res.set('Content-Type', client.register.contentType);
    res.end(await client.register.metrics());
  });

  // Sentry error handler
  app.use(Sentry.Handlers.errorHandler());
};
```

---

## Practice Exercises

### Exercise 1: Logging Setup

```javascript
// 1. Set up Pino logger
// 2. Add request logging middleware
// 3. Log errors with context
// 4. View logs in development
```

### Exercise 2: Error Tracking

```bash
// 1. Create Sentry account
// 2. Integrate with Express/Next.js
// 3. Trigger test error
// 4. View in Sentry dashboard
```

### Exercise 3: Health Monitoring

```javascript
// 1. Create health check endpoint
// 2. Add database/redis checks
// 3. Set up UptimeRobot
// 4. Configure Slack alerts
```

---

## Quick Reference

### Tools

| Category | Tools |
|----------|-------|
| Logging | Pino, Winston, Logtail |
| Errors | Sentry, Bugsnag, Rollbar |
| Metrics | Prometheus, Datadog, CloudWatch |
| Uptime | UptimeRobot, Better Uptime |
| Dashboards | Grafana, Datadog |
| Alerts | PagerDuty, Slack, Email |

### Key Metrics

| Metric | Target |
|--------|--------|
| Error rate | < 1% |
| Response time (p95) | < 500ms |
| Uptime | > 99.9% |
| Apdex score | > 0.9 |

---

## Key Takeaways

1. **Structured logging** - JSON logs with context
2. **Error tracking** - Capture and alert on errors
3. **Health checks** - Verify dependencies
4. **Metrics** - Track RED/USE patterns
5. **Alerting** - Get notified before users complain
6. **Dashboards** - Visualize system health

---

## Congratulations!

You've completed the Cloud Deployment section and Phase 4: DevOps! You now know how to:
- Deploy to Vercel and Railway
- Configure domains and SSL
- Work with AWS services
- Monitor application health

Next up is **Phase 5: Advanced** - covering System Design, Performance, AI Integration, and Job Prep!
