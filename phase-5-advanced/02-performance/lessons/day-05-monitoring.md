# Day 5: Performance Monitoring - Tracking What Matters

## Introduction

You can't improve what you don't measure. Performance monitoring provides the data you need to identify issues, track improvements, and ensure your application stays fast as it evolves. Today, you'll learn how to set up comprehensive monitoring for both frontend and backend performance.

## Learning Objectives

By the end of this lesson, you will be able to:
- Set up Real User Monitoring (RUM)
- Implement Application Performance Monitoring (APM)
- Create performance dashboards
- Configure alerts for performance regressions
- Use monitoring data to drive optimizations

---

## Monitoring Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                  MONITORING ECOSYSTEM                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  FRONTEND (RUM)              BACKEND (APM)                       │
│  ┌─────────────────┐        ┌─────────────────┐                 │
│  │ Core Web Vitals │        │ Response Times  │                 │
│  │ Page Load Times │        │ Error Rates     │                 │
│  │ JS Errors       │        │ Throughput      │                 │
│  │ User Sessions   │        │ Database Perf   │                 │
│  └────────┬────────┘        └────────┬────────┘                 │
│           │                          │                           │
│           └──────────┬───────────────┘                           │
│                      ▼                                           │
│           ┌─────────────────────┐                               │
│           │   Monitoring Stack  │                               │
│           │ ┌─────────────────┐ │                               │
│           │ │ Collection      │ │                               │
│           │ │ (Agents/SDKs)   │ │                               │
│           │ └────────┬────────┘ │                               │
│           │          ▼          │                               │
│           │ ┌─────────────────┐ │                               │
│           │ │ Storage         │ │                               │
│           │ │ (TimeSeries DB) │ │                               │
│           │ └────────┬────────┘ │                               │
│           │          ▼          │                               │
│           │ ┌─────────────────┐ │                               │
│           │ │ Visualization   │ │                               │
│           │ │ (Dashboards)    │ │                               │
│           │ └────────┬────────┘ │                               │
│           │          ▼          │                               │
│           │ ┌─────────────────┐ │                               │
│           │ │ Alerting        │ │                               │
│           │ │ (PagerDuty/Slack│ │                               │
│           │ └─────────────────┘ │                               │
│           └─────────────────────┘                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Real User Monitoring (RUM)

### Web Vitals Collection

```javascript
// lib/rum.js
import { onCLS, onINP, onLCP, onFCP, onTTFB } from 'web-vitals';

class RealUserMonitoring {
  constructor(options = {}) {
    this.endpoint = options.endpoint || '/api/rum';
    this.sampleRate = options.sampleRate || 1.0;
    this.metrics = {};
    this.sessionId = this.generateSessionId();

    this.init();
  }

  init() {
    // Only sample a percentage of users
    if (Math.random() > this.sampleRate) return;

    // Collect Core Web Vitals
    onCLS(metric => this.recordMetric('CLS', metric));
    onINP(metric => this.recordMetric('INP', metric));
    onLCP(metric => this.recordMetric('LCP', metric));
    onFCP(metric => this.recordMetric('FCP', metric));
    onTTFB(metric => this.recordMetric('TTFB', metric));

    // Navigation timing
    this.recordNavigationTiming();

    // Resource timing
    this.recordResourceTiming();

    // Long tasks
    this.observeLongTasks();

    // JS errors
    this.observeErrors();

    // Send on page unload
    window.addEventListener('visibilitychange', () => {
      if (document.visibilityState === 'hidden') {
        this.flush();
      }
    });
  }

  recordMetric(name, metric) {
    this.metrics[name] = {
      value: metric.value,
      rating: metric.rating,
      delta: metric.delta,
      id: metric.id,
      navigationType: metric.navigationType
    };

    // Send immediately for important metrics
    if (name === 'LCP' || name === 'CLS') {
      this.send({ type: 'vital', name, ...this.metrics[name] });
    }
  }

  recordNavigationTiming() {
    const observer = new PerformanceObserver((list) => {
      const navigation = list.getEntries()[0];

      this.send({
        type: 'navigation',
        dns: navigation.domainLookupEnd - navigation.domainLookupStart,
        tcp: navigation.connectEnd - navigation.connectStart,
        ttfb: navigation.responseStart - navigation.requestStart,
        download: navigation.responseEnd - navigation.responseStart,
        domParsing: navigation.domInteractive - navigation.responseEnd,
        domComplete: navigation.domComplete - navigation.domInteractive,
        load: navigation.loadEventEnd - navigation.loadEventStart
      });
    });

    observer.observe({ type: 'navigation', buffered: true });
  }

  recordResourceTiming() {
    const observer = new PerformanceObserver((list) => {
      const resources = list.getEntries();

      // Group by type
      const byType = resources.reduce((acc, r) => {
        const type = r.initiatorType;
        if (!acc[type]) acc[type] = [];
        acc[type].push({
          name: r.name,
          duration: r.duration,
          size: r.transferSize
        });
        return acc;
      }, {});

      // Find slow resources
      const slowResources = resources
        .filter(r => r.duration > 500)
        .map(r => ({
          name: r.name,
          duration: r.duration,
          type: r.initiatorType
        }));

      if (slowResources.length > 0) {
        this.send({ type: 'slow-resources', resources: slowResources });
      }
    });

    observer.observe({ type: 'resource', buffered: true });
  }

  observeLongTasks() {
    if (!('PerformanceLongTaskTiming' in window)) return;

    const observer = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        if (entry.duration > 50) {
          this.send({
            type: 'long-task',
            duration: entry.duration,
            startTime: entry.startTime
          });
        }
      }
    });

    observer.observe({ type: 'longtask', buffered: true });
  }

  observeErrors() {
    window.addEventListener('error', (event) => {
      this.send({
        type: 'error',
        message: event.message,
        filename: event.filename,
        lineno: event.lineno,
        colno: event.colno,
        stack: event.error?.stack
      });
    });

    window.addEventListener('unhandledrejection', (event) => {
      this.send({
        type: 'unhandled-rejection',
        reason: String(event.reason),
        stack: event.reason?.stack
      });
    });
  }

  send(data) {
    const payload = {
      ...data,
      sessionId: this.sessionId,
      url: window.location.href,
      timestamp: Date.now(),
      userAgent: navigator.userAgent,
      connection: navigator.connection?.effectiveType,
      deviceMemory: navigator.deviceMemory,
      hardwareConcurrency: navigator.hardwareConcurrency
    };

    // Use sendBeacon for reliability
    if (navigator.sendBeacon) {
      navigator.sendBeacon(this.endpoint, JSON.stringify(payload));
    } else {
      fetch(this.endpoint, {
        method: 'POST',
        body: JSON.stringify(payload),
        keepalive: true
      });
    }
  }

  flush() {
    this.send({
      type: 'session-end',
      metrics: this.metrics
    });
  }

  generateSessionId() {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }
}

// Initialize
export const rum = new RealUserMonitoring({
  endpoint: '/api/rum',
  sampleRate: 0.1 // 10% of users
});
```

### API Endpoint for RUM Data

```javascript
// pages/api/rum.js
import { prisma } from '@/lib/prisma';

export default async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).end();
  }

  try {
    const data = req.body;

    // Store in database
    await prisma.performanceMetric.create({
      data: {
        type: data.type,
        sessionId: data.sessionId,
        url: data.url,
        value: data.value || null,
        rating: data.rating || null,
        payload: data,
        userAgent: data.userAgent,
        connection: data.connection,
        timestamp: new Date(data.timestamp)
      }
    });

    // Check for alerts
    if (data.type === 'vital' && data.rating === 'poor') {
      await sendAlert({
        metric: data.name,
        value: data.value,
        url: data.url
      });
    }

    res.status(204).end();
  } catch (error) {
    console.error('RUM ingestion error:', error);
    res.status(500).end();
  }
}

// Aggregate metrics for dashboard
export async function getMetricsSummary(timeRange = '24h') {
  const since = new Date(Date.now() - parseTimeRange(timeRange));

  const metrics = await prisma.$queryRaw`
    SELECT
      type,
      PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY value) as p50,
      PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY value) as p75,
      PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY value) as p95,
      COUNT(*) as count,
      AVG(CASE WHEN rating = 'good' THEN 1 ELSE 0 END) * 100 as good_percent
    FROM performance_metrics
    WHERE timestamp > ${since}
      AND type = 'vital'
    GROUP BY type
  `;

  return metrics;
}
```

---

## Application Performance Monitoring (APM)

### Express APM Middleware

```javascript
// middleware/apm.js
const { AsyncLocalStorage } = require('async_hooks');

const traceStorage = new AsyncLocalStorage();

class APM {
  constructor(options = {}) {
    this.serviceName = options.serviceName || 'api';
    this.endpoint = options.endpoint || '/api/apm';
    this.sampleRate = options.sampleRate || 1.0;
  }

  middleware() {
    return (req, res, next) => {
      // Skip based on sample rate
      if (Math.random() > this.sampleRate) return next();

      const trace = {
        traceId: this.generateTraceId(),
        spanId: this.generateSpanId(),
        startTime: process.hrtime.bigint(),
        method: req.method,
        path: req.path,
        spans: []
      };

      // Run in async context
      traceStorage.run(trace, () => {
        // Capture response
        res.on('finish', () => {
          trace.endTime = process.hrtime.bigint();
          trace.duration = Number(trace.endTime - trace.startTime) / 1e6;
          trace.statusCode = res.statusCode;

          this.send(trace);
        });

        next();
      });
    };
  }

  // Create a span for tracking operations
  span(name) {
    const trace = traceStorage.getStore();
    if (!trace) return { end: () => {} };

    const span = {
      id: this.generateSpanId(),
      name,
      startTime: process.hrtime.bigint()
    };

    trace.spans.push(span);

    return {
      end: (metadata = {}) => {
        span.endTime = process.hrtime.bigint();
        span.duration = Number(span.endTime - span.startTime) / 1e6;
        span.metadata = metadata;
      },
      setError: (error) => {
        span.error = {
          message: error.message,
          stack: error.stack
        };
      }
    };
  }

  // Wrap database client
  wrapPrisma(prisma) {
    return prisma.$extends({
      query: {
        $allModels: {
          async $allOperations({ model, operation, args, query }) {
            const span = apm.span(`prisma.${model}.${operation}`);
            try {
              const result = await query(args);
              span.end({ model, operation });
              return result;
            } catch (error) {
              span.setError(error);
              span.end({ model, operation, error: true });
              throw error;
            }
          }
        }
      }
    });
  }

  // Wrap fetch calls
  wrapFetch() {
    const originalFetch = global.fetch;

    global.fetch = async (url, options = {}) => {
      const span = this.span(`fetch:${new URL(url).hostname}`);
      try {
        const response = await originalFetch(url, options);
        span.end({
          url,
          method: options.method || 'GET',
          status: response.status
        });
        return response;
      } catch (error) {
        span.setError(error);
        span.end({ url, error: true });
        throw error;
      }
    };
  }

  send(trace) {
    // Async send to avoid blocking
    setImmediate(() => {
      fetch(this.endpoint, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          ...trace,
          service: this.serviceName,
          timestamp: Date.now()
        })
      }).catch(err => console.error('APM send failed:', err));
    });
  }

  generateTraceId() {
    return `${Date.now().toString(36)}-${Math.random().toString(36).substr(2, 16)}`;
  }

  generateSpanId() {
    return Math.random().toString(36).substr(2, 16);
  }
}

// Export singleton
const apm = new APM({
  serviceName: process.env.SERVICE_NAME || 'api',
  sampleRate: parseFloat(process.env.APM_SAMPLE_RATE) || 0.1
});

module.exports = { apm, traceStorage };
```

### Using APM in Routes

```javascript
const express = require('express');
const { apm } = require('./middleware/apm');
const prisma = require('./lib/prisma');

const app = express();

// Apply APM middleware
app.use(apm.middleware());

// Wrap Prisma for automatic tracing
const db = apm.wrapPrisma(prisma);

// Wrap fetch globally
apm.wrapFetch();

app.get('/api/products/:id', async (req, res) => {
  // Automatic tracing for database queries
  const product = await db.product.findUnique({
    where: { id: req.params.id }
  });

  // Manual span for custom operations
  const enrichSpan = apm.span('enrichProduct');
  try {
    const enrichedProduct = await enrichWithExternalData(product);
    enrichSpan.end({ success: true });
    res.json(enrichedProduct);
  } catch (error) {
    enrichSpan.setError(error);
    enrichSpan.end({ success: false });
    throw error;
  }
});

async function enrichWithExternalData(product) {
  // fetch is automatically traced
  const response = await fetch(`https://api.example.com/enrich/${product.sku}`);
  return { ...product, ...(await response.json()) };
}
```

---

## Metrics and Dashboards

### Prometheus Metrics

```javascript
// metrics/prometheus.js
const client = require('prom-client');

// Create registry
const register = new client.Registry();

// Add default metrics
client.collectDefaultMetrics({ register });

// Custom metrics
const httpRequestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.01, 0.05, 0.1, 0.5, 1, 5]
});
register.registerMetric(httpRequestDuration);

const httpRequestsTotal = new client.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code']
});
register.registerMetric(httpRequestsTotal);

const activeConnections = new client.Gauge({
  name: 'http_active_connections',
  help: 'Number of active HTTP connections'
});
register.registerMetric(activeConnections);

const dbQueryDuration = new client.Histogram({
  name: 'db_query_duration_seconds',
  help: 'Duration of database queries',
  labelNames: ['operation', 'model'],
  buckets: [0.001, 0.01, 0.05, 0.1, 0.5, 1]
});
register.registerMetric(dbQueryDuration);

// Middleware to collect HTTP metrics
function metricsMiddleware(req, res, next) {
  activeConnections.inc();

  const start = process.hrtime.bigint();

  res.on('finish', () => {
    activeConnections.dec();

    const duration = Number(process.hrtime.bigint() - start) / 1e9;
    const labels = {
      method: req.method,
      route: req.route?.path || req.path,
      status_code: res.statusCode
    };

    httpRequestDuration.observe(labels, duration);
    httpRequestsTotal.inc(labels);
  });

  next();
}

// Metrics endpoint
function metricsHandler(req, res) {
  res.set('Content-Type', register.contentType);
  register.metrics().then(data => res.end(data));
}

module.exports = {
  register,
  metricsMiddleware,
  metricsHandler,
  httpRequestDuration,
  dbQueryDuration
};
```

### Grafana Dashboard Configuration

```json
{
  "dashboard": {
    "title": "API Performance",
    "panels": [
      {
        "title": "Request Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])",
            "legendFormat": "{{method}} {{route}}"
          }
        ]
      },
      {
        "title": "Response Time (p95)",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "p95"
          },
          {
            "expr": "histogram_quantile(0.50, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "p50"
          }
        ]
      },
      {
        "title": "Error Rate",
        "type": "singlestat",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total{status_code=~\"5..\"}[5m])) / sum(rate(http_requests_total[5m])) * 100"
          }
        ]
      },
      {
        "title": "Database Query Time",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(db_query_duration_seconds_bucket[5m]))",
            "legendFormat": "{{operation}} {{model}}"
          }
        ]
      }
    ]
  }
}
```

---

## Alerting

### Alert Rules

```javascript
// alerts/rules.js
const alertRules = [
  {
    name: 'HighErrorRate',
    condition: (metrics) => {
      const errorRate = metrics.errors5xx / metrics.totalRequests;
      return errorRate > 0.01; // > 1% error rate
    },
    severity: 'critical',
    message: (metrics) =>
      `Error rate is ${(metrics.errors5xx / metrics.totalRequests * 100).toFixed(2)}%`
  },
  {
    name: 'SlowResponseTime',
    condition: (metrics) => metrics.p95ResponseTime > 1000,
    severity: 'warning',
    message: (metrics) =>
      `P95 response time is ${metrics.p95ResponseTime}ms (threshold: 1000ms)`
  },
  {
    name: 'HighMemoryUsage',
    condition: (metrics) => metrics.memoryUsage > 0.9,
    severity: 'warning',
    message: (metrics) =>
      `Memory usage is at ${(metrics.memoryUsage * 100).toFixed(1)}%`
  },
  {
    name: 'PoorCoreWebVitals',
    condition: (metrics) => {
      return metrics.lcp?.rating === 'poor' ||
             metrics.cls?.rating === 'poor' ||
             metrics.inp?.rating === 'poor';
    },
    severity: 'warning',
    message: (metrics) => {
      const poor = [];
      if (metrics.lcp?.rating === 'poor') poor.push(`LCP: ${metrics.lcp.value}ms`);
      if (metrics.cls?.rating === 'poor') poor.push(`CLS: ${metrics.cls.value}`);
      if (metrics.inp?.rating === 'poor') poor.push(`INP: ${metrics.inp.value}ms`);
      return `Poor Core Web Vitals: ${poor.join(', ')}`;
    }
  }
];

// Alert checker
class AlertManager {
  constructor(notifiers = []) {
    this.notifiers = notifiers;
    this.activeAlerts = new Map();
    this.cooldownMs = 5 * 60 * 1000; // 5 minutes
  }

  async check(metrics) {
    for (const rule of alertRules) {
      try {
        if (rule.condition(metrics)) {
          await this.fire(rule, metrics);
        } else {
          this.resolve(rule.name);
        }
      } catch (error) {
        console.error(`Error checking alert ${rule.name}:`, error);
      }
    }
  }

  async fire(rule, metrics) {
    const lastFired = this.activeAlerts.get(rule.name);

    // Cooldown check
    if (lastFired && Date.now() - lastFired < this.cooldownMs) {
      return;
    }

    this.activeAlerts.set(rule.name, Date.now());

    const alert = {
      name: rule.name,
      severity: rule.severity,
      message: rule.message(metrics),
      timestamp: new Date().toISOString()
    };

    console.log(`Alert fired: ${rule.name}`);

    for (const notifier of this.notifiers) {
      try {
        await notifier.send(alert);
      } catch (error) {
        console.error(`Notifier error:`, error);
      }
    }
  }

  resolve(name) {
    if (this.activeAlerts.has(name)) {
      this.activeAlerts.delete(name);
      console.log(`Alert resolved: ${name}`);
    }
  }
}

module.exports = { AlertManager, alertRules };
```

### Notification Integrations

```javascript
// alerts/notifiers.js

// Slack notifier
class SlackNotifier {
  constructor(webhookUrl) {
    this.webhookUrl = webhookUrl;
  }

  async send(alert) {
    const color = {
      critical: '#ff0000',
      warning: '#ffaa00',
      info: '#0000ff'
    }[alert.severity] || '#808080';

    await fetch(this.webhookUrl, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        attachments: [{
          color,
          title: `🚨 ${alert.name}`,
          text: alert.message,
          footer: `Severity: ${alert.severity}`,
          ts: Math.floor(Date.now() / 1000)
        }]
      })
    });
  }
}

// PagerDuty notifier
class PagerDutyNotifier {
  constructor(integrationKey) {
    this.integrationKey = integrationKey;
  }

  async send(alert) {
    if (alert.severity !== 'critical') return;

    await fetch('https://events.pagerduty.com/v2/enqueue', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        routing_key: this.integrationKey,
        event_action: 'trigger',
        dedup_key: alert.name,
        payload: {
          summary: alert.message,
          severity: 'critical',
          source: 'performance-monitoring',
          timestamp: alert.timestamp
        }
      })
    });
  }
}

// Email notifier
class EmailNotifier {
  constructor(config) {
    this.transporter = nodemailer.createTransport(config);
    this.recipients = config.recipients;
  }

  async send(alert) {
    await this.transporter.sendMail({
      from: 'alerts@myapp.com',
      to: this.recipients.join(','),
      subject: `[${alert.severity.toUpperCase()}] ${alert.name}`,
      text: `
Alert: ${alert.name}
Severity: ${alert.severity}
Time: ${alert.timestamp}

${alert.message}
      `.trim()
    });
  }
}

module.exports = { SlackNotifier, PagerDutyNotifier, EmailNotifier };
```

---

## Performance Regression Detection

```javascript
// regression/detector.js
class RegressionDetector {
  constructor(db) {
    this.db = db;
    this.thresholds = {
      LCP: { warning: 0.1, critical: 0.2 },    // 10%, 20% regression
      INP: { warning: 0.1, critical: 0.2 },
      CLS: { warning: 0.15, critical: 0.3 },
      TTFB: { warning: 0.15, critical: 0.3 },
      responseTime: { warning: 0.1, critical: 0.2 }
    };
  }

  async detect() {
    const regressions = [];

    // Compare current metrics to baseline
    const current = await this.getCurrentMetrics();
    const baseline = await this.getBaselineMetrics();

    for (const [metric, values] of Object.entries(current)) {
      const baselineValue = baseline[metric];
      if (!baselineValue) continue;

      const change = (values.p75 - baselineValue.p75) / baselineValue.p75;

      const threshold = this.thresholds[metric];
      if (!threshold) continue;

      if (change > threshold.critical) {
        regressions.push({
          metric,
          severity: 'critical',
          change: change * 100,
          current: values.p75,
          baseline: baselineValue.p75
        });
      } else if (change > threshold.warning) {
        regressions.push({
          metric,
          severity: 'warning',
          change: change * 100,
          current: values.p75,
          baseline: baselineValue.p75
        });
      }
    }

    return regressions;
  }

  async getCurrentMetrics() {
    // Get metrics from last 24 hours
    return this.db.$queryRaw`
      SELECT
        type as metric,
        PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY value) as p75
      FROM performance_metrics
      WHERE timestamp > NOW() - INTERVAL '24 hours'
        AND type IN ('LCP', 'INP', 'CLS', 'TTFB')
      GROUP BY type
    `;
  }

  async getBaselineMetrics() {
    // Get baseline from last 7-30 days (excluding last 24h)
    return this.db.$queryRaw`
      SELECT
        type as metric,
        PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY value) as p75
      FROM performance_metrics
      WHERE timestamp BETWEEN NOW() - INTERVAL '30 days' AND NOW() - INTERVAL '24 hours'
        AND type IN ('LCP', 'INP', 'CLS', 'TTFB')
      GROUP BY type
    `;
  }
}

// Run regression detection periodically
const detector = new RegressionDetector(db);

async function checkForRegressions() {
  const regressions = await detector.detect();

  for (const regression of regressions) {
    await alertManager.fire({
      name: 'PerformanceRegression',
      severity: regression.severity,
      message: () =>
        `${regression.metric} regressed by ${regression.change.toFixed(1)}% ` +
        `(${regression.baseline.toFixed(2)} → ${regression.current.toFixed(2)})`
    }, {});
  }
}

// Check every hour
setInterval(checkForRegressions, 60 * 60 * 1000);
```

---

## Synthetic Monitoring

```javascript
// synthetic/monitor.js
const { chromium } = require('playwright');

class SyntheticMonitor {
  constructor(config) {
    this.config = config;
    this.results = [];
  }

  async runChecks() {
    const browser = await chromium.launch();
    const context = await browser.newContext();

    for (const check of this.config.checks) {
      try {
        const result = await this.runCheck(context, check);
        this.results.push(result);
      } catch (error) {
        this.results.push({
          name: check.name,
          success: false,
          error: error.message
        });
      }
    }

    await browser.close();
    return this.results;
  }

  async runCheck(context, check) {
    const page = await context.newPage();

    // Enable performance metrics
    await page.coverage.startJSCoverage();

    const startTime = Date.now();
    await page.goto(check.url, { waitUntil: 'networkidle' });
    const loadTime = Date.now() - startTime;

    // Get Core Web Vitals
    const vitals = await page.evaluate(() => {
      return new Promise((resolve) => {
        new PerformanceObserver((list) => {
          const lcp = list.getEntries().pop();
          resolve({ lcp: lcp?.startTime });
        }).observe({ type: 'largest-contentful-paint', buffered: true });
      });
    });

    // Run assertions
    const assertions = [];
    for (const assertion of check.assertions || []) {
      const passed = await this.runAssertion(page, assertion);
      assertions.push({ ...assertion, passed });
    }

    await page.close();

    return {
      name: check.name,
      url: check.url,
      success: assertions.every(a => a.passed),
      loadTime,
      vitals,
      assertions
    };
  }

  async runAssertion(page, assertion) {
    switch (assertion.type) {
      case 'element-visible':
        return page.isVisible(assertion.selector);

      case 'response-time':
        const timing = await page.evaluate(() =>
          performance.timing.loadEventEnd - performance.timing.navigationStart
        );
        return timing < assertion.maxMs;

      case 'text-content':
        const text = await page.textContent(assertion.selector);
        return text?.includes(assertion.contains);

      default:
        return false;
    }
  }
}

// Configuration
const monitorConfig = {
  checks: [
    {
      name: 'Homepage Load',
      url: 'https://myapp.com',
      assertions: [
        { type: 'response-time', maxMs: 3000 },
        { type: 'element-visible', selector: '[data-testid="hero"]' },
        { type: 'text-content', selector: 'h1', contains: 'Welcome' }
      ]
    },
    {
      name: 'Product Page',
      url: 'https://myapp.com/products/123',
      assertions: [
        { type: 'response-time', maxMs: 2000 },
        { type: 'element-visible', selector: '[data-testid="add-to-cart"]' }
      ]
    }
  ]
};

// Run every 5 minutes
const monitor = new SyntheticMonitor(monitorConfig);

setInterval(async () => {
  const results = await monitor.runChecks();

  // Report results
  for (const result of results) {
    if (!result.success) {
      await alertManager.fire({
        name: 'SyntheticCheckFailed',
        severity: 'critical',
        message: () => `Synthetic check "${result.name}" failed: ${result.error || 'Assertions failed'}`
      }, {});
    }
  }
}, 5 * 60 * 1000);
```

---

## Practice Exercises

### Exercise 1: RUM Setup

Implement Real User Monitoring:
- Collect Core Web Vitals
- Send to analytics endpoint
- Build dashboard showing p50, p75, p95

### Exercise 2: APM Integration

Add APM to an existing API:
- Add request tracing
- Track database queries
- Identify slow endpoints

### Exercise 3: Alerting

Set up performance alerts:
- Define thresholds for key metrics
- Integrate with Slack
- Create escalation rules

---

## Key Takeaways

1. **Measure real users** - RUM gives you actual user experience data
2. **Trace requests** - APM helps identify bottlenecks
3. **Set baselines** - Know your normal to detect regressions
4. **Alert wisely** - Too many alerts leads to alert fatigue
5. **Automate checks** - Synthetic monitoring catches issues early
6. **Act on data** - Monitoring without action is pointless

---

## Conclusion

You've completed the Performance section! You now know how to:
- Measure and improve Core Web Vitals
- Optimize images for the web
- Split code for faster loading
- Speed up backend responses
- Monitor performance in production

Performance is an ongoing practice. Set up your monitoring, establish baselines, and continuously improve based on real data.
