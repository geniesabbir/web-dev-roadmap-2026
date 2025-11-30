# Day 5: Security Headers - Protecting Your Application

## Introduction

HTTP security headers are your first line of defense against many common web attacks. These headers instruct browsers how to behave when handling your site's content, preventing attacks like XSS, clickjacking, and data injection. Today, you'll learn how to implement essential security headers using Helmet.js and understand what each header protects against.

## Learning Objectives

By the end of this lesson, you will be able to:
- Understand common web vulnerabilities that headers prevent
- Configure Helmet.js for comprehensive protection
- Implement Content Security Policy (CSP)
- Set up CORS properly
- Test and validate security headers

---

## Why Security Headers Matter

### Common Attacks Prevented by Headers

| Attack | Description | Prevention Header |
|--------|-------------|-------------------|
| XSS | Injecting malicious scripts | Content-Security-Policy |
| Clickjacking | Embedding your site in iframe | X-Frame-Options |
| MIME Sniffing | Executing files as wrong type | X-Content-Type-Options |
| Man-in-the-Middle | Downgrading HTTPS to HTTP | Strict-Transport-Security |
| Information Leakage | Exposing referrer data | Referrer-Policy |

---

## Setting Up Helmet.js

### Installation

```bash
npm install helmet
```

### Basic Configuration

```javascript
// app.js
const express = require('express');
const helmet = require('helmet');

const app = express();

// Apply all default security headers
app.use(helmet());

// Your routes...
app.get('/', (req, res) => {
  res.send('Secured with Helmet!');
});
```

### Default Headers Applied by Helmet

```http
Content-Security-Policy: default-src 'self';base-uri 'self';...
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Resource-Policy: same-origin
Origin-Agent-Cluster: ?1
Referrer-Policy: no-referrer
Strict-Transport-Security: max-age=15552000; includeSubDomains
X-Content-Type-Options: nosniff
X-DNS-Prefetch-Control: off
X-Download-Options: noopen
X-Frame-Options: SAMEORIGIN
X-Permitted-Cross-Domain-Policies: none
X-XSS-Protection: 0
```

---

## Content Security Policy (CSP)

CSP is the most powerful security header. It tells the browser which sources of content are trusted.

### Understanding CSP Directives

```javascript
// CSP Directive Reference
const cspDirectives = {
  'default-src': 'Fallback for other directives',
  'script-src': 'Where scripts can be loaded from',
  'style-src': 'Where stylesheets can be loaded from',
  'img-src': 'Where images can be loaded from',
  'font-src': 'Where fonts can be loaded from',
  'connect-src': 'Where fetch/XHR can connect to',
  'frame-src': 'Where iframes can embed from',
  'object-src': 'Where plugins (Flash, etc.) can load from',
  'media-src': 'Where audio/video can load from',
  'form-action': 'Where forms can submit to',
  'frame-ancestors': 'Who can embed this page in iframe',
  'base-uri': 'Restricts <base> tag URLs',
  'report-uri': 'Where to send violation reports'
};
```

### CSP Source Values

```javascript
// CSP Source Values
const sourceValues = {
  "'self'": 'Same origin only',
  "'none'": 'Block all',
  "'unsafe-inline'": 'Allow inline scripts/styles (avoid!)',
  "'unsafe-eval'": 'Allow eval() (avoid!)',
  "'strict-dynamic'": 'Trust scripts loaded by trusted scripts',
  "'nonce-{random}'": 'Allow specific inline elements',
  "'sha256-{hash}'": 'Allow specific inline by hash',
  'https:': 'Any HTTPS source',
  'data:': 'Data URLs (for inline images)',
  '*.example.com': 'Wildcard subdomain'
};
```

### Configuring CSP with Helmet

```javascript
// Strict CSP Configuration
app.use(
  helmet.contentSecurityPolicy({
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: [
        "'self'",
        "https://cdn.jsdelivr.net",
        "https://www.google-analytics.com"
      ],
      styleSrc: [
        "'self'",
        "https://fonts.googleapis.com",
        "'unsafe-inline'"  // Often needed for CSS-in-JS
      ],
      fontSrc: [
        "'self'",
        "https://fonts.gstatic.com"
      ],
      imgSrc: [
        "'self'",
        "data:",  // For inline images
        "https:",  // Any HTTPS image
        "blob:"   // For dynamically created images
      ],
      connectSrc: [
        "'self'",
        "https://api.myapp.com",
        "wss://ws.myapp.com"  // WebSocket
      ],
      frameSrc: ["'none'"],  // No iframes allowed
      objectSrc: ["'none'"], // No Flash/plugins
      baseUri: ["'self'"],
      formAction: ["'self'"],
      frameAncestors: ["'none'"],  // Prevent clickjacking
      upgradeInsecureRequests: []  // Upgrade HTTP to HTTPS
    }
  })
);
```

### Using Nonces for Inline Scripts

```javascript
const crypto = require('crypto');

// Generate nonce per request
app.use((req, res, next) => {
  res.locals.nonce = crypto.randomBytes(16).toString('base64');
  next();
});

// Apply CSP with nonce
app.use((req, res, next) => {
  helmet.contentSecurityPolicy({
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: [
        "'self'",
        `'nonce-${res.locals.nonce}'`
      ],
      styleSrc: [
        "'self'",
        `'nonce-${res.locals.nonce}'`
      ]
    }
  })(req, res, next);
});

// Use nonce in HTML
app.get('/', (req, res) => {
  res.send(`
    <!DOCTYPE html>
    <html>
      <head>
        <script nonce="${res.locals.nonce}">
          // This inline script will execute
          console.log('Allowed by nonce');
        </script>
      </head>
      <body>
        <h1>Nonce-protected page</h1>
      </body>
    </html>
  `);
});
```

### CSP Reporting

```javascript
// Report-only mode (for testing)
app.use(
  helmet.contentSecurityPolicy({
    directives: {
      defaultSrc: ["'self'"],
      reportUri: '/csp-report'
    },
    reportOnly: true  // Don't block, just report
  })
);

// CSP violation endpoint
app.post('/csp-report', express.json({ type: 'application/csp-report' }), (req, res) => {
  console.log('CSP Violation:', req.body);

  // Log to monitoring service
  // logService.logCSPViolation(req.body);

  res.status(204).end();
});
```

---

## Strict Transport Security (HSTS)

Forces browsers to only use HTTPS for your domain.

```javascript
// HSTS Configuration
app.use(
  helmet.hsts({
    maxAge: 31536000,         // 1 year in seconds
    includeSubDomains: true,  // Apply to subdomains
    preload: true             // Allow HSTS preload list submission
  })
);

// Equivalent header:
// Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

### HSTS Considerations

```javascript
// IMPORTANT: Start with a short max-age when testing
app.use(
  helmet.hsts({
    maxAge: process.env.NODE_ENV === 'production'
      ? 31536000  // 1 year in production
      : 86400     // 1 day in development
  })
);

// WARNING: Once set, browsers will ONLY use HTTPS
// Make sure your SSL is properly configured before enabling!
```

---

## X-Frame-Options (Clickjacking Protection)

Prevents your site from being embedded in iframes.

```javascript
// Prevent all framing
app.use(
  helmet.frameguard({ action: 'deny' })
);

// Allow same-origin framing only
app.use(
  helmet.frameguard({ action: 'sameorigin' })
);

// CSP frame-ancestors is more flexible (recommended)
app.use(
  helmet.contentSecurityPolicy({
    directives: {
      frameAncestors: ["'self'", "https://trusted-site.com"]
    }
  })
);
```

---

## X-Content-Type-Options

Prevents MIME type sniffing.

```javascript
app.use(helmet.noSniff());
// Header: X-Content-Type-Options: nosniff

// Without this, browsers might execute:
// - HTML files as JavaScript
// - Images as scripts
// - Text files as executables
```

---

## Referrer Policy

Controls how much referrer information is sent with requests.

```javascript
// Options from most to least restrictive
app.use(
  helmet.referrerPolicy({
    policy: 'no-referrer'  // Never send referrer
  })
);

// Other options:
// 'no-referrer' - Never send
// 'same-origin' - Only for same-origin requests
// 'strict-origin' - Send origin only, HTTPS→HTTPS
// 'strict-origin-when-cross-origin' - Full for same-origin, origin for cross
// 'origin' - Only send origin, not full URL
// 'origin-when-cross-origin' - Full for same-origin, origin for cross
// 'unsafe-url' - Always send full URL (avoid!)
```

---

## CORS (Cross-Origin Resource Sharing)

### Understanding CORS

```
┌─────────────────────────────────────────────────────────────────┐
│                         CORS Flow                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Browser                    Server (api.example.com)            │
│     │                              │                            │
│     │  Preflight (OPTIONS)         │                            │
│     │  Origin: https://app.com     │                            │
│     │  Access-Control-Request-*    │                            │
│     │────────────────────────────>│                             │
│     │                              │                            │
│     │  Access-Control-Allow-*      │                            │
│     │<────────────────────────────│                             │
│     │                              │                            │
│     │  Actual Request (GET/POST)   │                            │
│     │────────────────────────────>│                             │
│     │                              │                            │
│     │  Response + CORS Headers     │                            │
│     │<────────────────────────────│                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Installing and Configuring CORS

```bash
npm install cors
```

```javascript
const cors = require('cors');

// Allow all origins (development only!)
app.use(cors());

// Production configuration
const corsOptions = {
  origin: function (origin, callback) {
    const allowedOrigins = [
      'https://myapp.com',
      'https://www.myapp.com',
      'https://admin.myapp.com'
    ];

    // Allow requests with no origin (mobile apps, curl, etc.)
    if (!origin) return callback(null, true);

    if (allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true,  // Allow cookies
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  exposedHeaders: ['X-Total-Count'],  // Custom headers client can access
  maxAge: 86400  // Preflight cache (24 hours)
};

app.use(cors(corsOptions));
```

### Per-Route CORS

```javascript
// Different CORS for different routes
app.get('/api/public', cors(), (req, res) => {
  // Open to all origins
  res.json({ message: 'Public data' });
});

app.get('/api/private', cors({
  origin: 'https://myapp.com',
  credentials: true
}), (req, res) => {
  // Restricted to specific origin
  res.json({ message: 'Private data' });
});

// Webhook endpoint - accept from specific service
app.post('/webhooks/stripe', cors({
  origin: 'https://api.stripe.com'
}), (req, res) => {
  // Handle webhook
});
```

---

## Complete Security Headers Configuration

```javascript
// config/security.js
const helmet = require('helmet');
const cors = require('cors');
const crypto = require('crypto');

const isDev = process.env.NODE_ENV !== 'production';

// Generate nonce middleware
const nonceMiddleware = (req, res, next) => {
  res.locals.nonce = crypto.randomBytes(16).toString('base64');
  next();
};

// Helmet configuration
const helmetConfig = (req, res) => ({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: [
        "'self'",
        `'nonce-${res.locals.nonce}'`,
        ...(isDev ? ["'unsafe-eval'"] : []),  // For HMR in dev
        'https://cdn.jsdelivr.net'
      ],
      styleSrc: [
        "'self'",
        `'nonce-${res.locals.nonce}'`,
        'https://fonts.googleapis.com'
      ],
      fontSrc: ["'self'", 'https://fonts.gstatic.com'],
      imgSrc: ["'self'", 'data:', 'https:'],
      connectSrc: [
        "'self'",
        process.env.API_URL,
        ...(isDev ? ['ws://localhost:*'] : [])
      ],
      frameSrc: ["'none'"],
      objectSrc: ["'none'"],
      baseUri: ["'self'"],
      formAction: ["'self'"],
      frameAncestors: ["'none'"],
      ...(isDev ? {} : { upgradeInsecureRequests: [] })
    },
    reportOnly: isDev
  },
  hsts: {
    maxAge: isDev ? 86400 : 31536000,
    includeSubDomains: true,
    preload: !isDev
  },
  referrerPolicy: {
    policy: 'strict-origin-when-cross-origin'
  }
});

// CORS configuration
const corsConfig = {
  origin: function (origin, callback) {
    const allowedOrigins = process.env.ALLOWED_ORIGINS?.split(',') || [];

    if (isDev) {
      allowedOrigins.push('http://localhost:3000');
      allowedOrigins.push('http://localhost:5173');
    }

    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error(`Origin ${origin} not allowed by CORS`));
    }
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization', 'X-Requested-With'],
  exposedHeaders: ['X-Total-Count', 'X-Page', 'X-Per-Page'],
  maxAge: 86400
};

module.exports = { nonceMiddleware, helmetConfig, corsConfig };
```

### Apply in Express

```javascript
// app.js
const express = require('express');
const { nonceMiddleware, helmetConfig, corsConfig } = require('./config/security');
const helmet = require('helmet');
const cors = require('cors');

const app = express();

// Security headers
app.use(nonceMiddleware);
app.use((req, res, next) => {
  helmet(helmetConfig(req, res))(req, res, next);
});
app.use(cors(corsConfig));

// Additional security measures
app.disable('x-powered-by');  // Don't reveal Express

// Rate limiting (covered in next lesson)
// app.use(rateLimit(...));

module.exports = app;
```

---

## Testing Security Headers

### Using SecurityHeaders.com

```javascript
// Test your production site at:
// https://securityheaders.com/?q=https://yoursite.com

// Goal: A+ rating
```

### Automated Testing

```javascript
// tests/security-headers.test.js
const request = require('supertest');
const app = require('../app');

describe('Security Headers', () => {
  it('should have Content-Security-Policy', async () => {
    const res = await request(app).get('/');
    expect(res.headers['content-security-policy']).toBeDefined();
  });

  it('should have X-Frame-Options', async () => {
    const res = await request(app).get('/');
    expect(res.headers['x-frame-options']).toBe('SAMEORIGIN');
  });

  it('should have X-Content-Type-Options', async () => {
    const res = await request(app).get('/');
    expect(res.headers['x-content-type-options']).toBe('nosniff');
  });

  it('should have Strict-Transport-Security', async () => {
    const res = await request(app).get('/');
    expect(res.headers['strict-transport-security']).toMatch(/max-age=\d+/);
  });

  it('should have Referrer-Policy', async () => {
    const res = await request(app).get('/');
    expect(res.headers['referrer-policy']).toBeDefined();
  });

  it('should not expose X-Powered-By', async () => {
    const res = await request(app).get('/');
    expect(res.headers['x-powered-by']).toBeUndefined();
  });
});

describe('CORS', () => {
  it('should allow requests from allowed origin', async () => {
    const res = await request(app)
      .get('/api/test')
      .set('Origin', 'https://myapp.com');

    expect(res.headers['access-control-allow-origin']).toBe('https://myapp.com');
  });

  it('should reject requests from unknown origin', async () => {
    const res = await request(app)
      .get('/api/test')
      .set('Origin', 'https://evil.com');

    expect(res.headers['access-control-allow-origin']).toBeUndefined();
  });

  it('should handle preflight requests', async () => {
    const res = await request(app)
      .options('/api/test')
      .set('Origin', 'https://myapp.com')
      .set('Access-Control-Request-Method', 'POST');

    expect(res.status).toBe(204);
    expect(res.headers['access-control-allow-methods']).toBeDefined();
  });
});
```

---

## Browser DevTools Check

```javascript
// Check headers in browser DevTools:
// 1. Open DevTools (F12)
// 2. Go to Network tab
// 3. Click on any request
// 4. Check Response Headers

// Common issues to look for:
// - Missing CSP header
// - X-Powered-By: Express (should be removed)
// - Missing HSTS on HTTPS sites
// - Overly permissive CORS
```

---

## Security Headers Cheat Sheet

```javascript
// Essential headers for any production app
const essentialHeaders = {
  // Prevent XSS
  'Content-Security-Policy': "default-src 'self'",

  // Prevent clickjacking
  'X-Frame-Options': 'DENY',

  // Prevent MIME sniffing
  'X-Content-Type-Options': 'nosniff',

  // Force HTTPS
  'Strict-Transport-Security': 'max-age=31536000; includeSubDomains',

  // Control referrer
  'Referrer-Policy': 'strict-origin-when-cross-origin',

  // Prevent some browser features
  'Permissions-Policy': 'geolocation=(), microphone=(), camera=()',

  // Cross-origin isolation
  'Cross-Origin-Opener-Policy': 'same-origin',
  'Cross-Origin-Resource-Policy': 'same-origin'
};
```

---

## Practice Exercise

Configure comprehensive security headers:

1. **Set up Helmet**:
   - Apply default configuration
   - Customize CSP for your app's needs

2. **Configure CSP**:
   - Allow your CDN sources
   - Use nonces for inline scripts
   - Set up CSP reporting

3. **Configure CORS**:
   - Allow specific origins
   - Enable credentials
   - Set appropriate methods

4. **Test your configuration**:
   - Use securityheaders.com
   - Write automated tests
   - Check browser DevTools

5. **Bonus**: Implement CSP violation reporting and monitoring

---

## Key Takeaways

1. **Helmet provides sensible defaults** - Start with `app.use(helmet())`
2. **CSP is your strongest tool** - Prevents XSS and injection attacks
3. **Use nonces for inline scripts** - Don't use `unsafe-inline`
4. **HSTS forces HTTPS** - Prevents downgrade attacks
5. **CORS is not security** - It's browser-enforced, not server-enforced
6. **Test your headers** - Use automated tools and manual verification
7. **Start strict, loosen carefully** - Easier than fixing vulnerabilities

---

## What's Next?

Tomorrow, we'll implement **Rate Limiting** - protecting your API from abuse, brute force attacks, and DDoS attempts.
