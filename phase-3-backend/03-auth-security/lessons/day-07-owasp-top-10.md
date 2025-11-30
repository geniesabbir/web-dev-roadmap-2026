# Day 7: OWASP Top 10 - Critical Web Application Security Risks

## Introduction

The OWASP (Open Web Application Security Project) Top 10 is the definitive list of the most critical security risks facing web applications. Understanding and protecting against these vulnerabilities is essential for any web developer. Today, we'll explore each risk and learn practical prevention techniques.

## Learning Objectives

By the end of this lesson, you will be able to:
- Identify the OWASP Top 10 security risks
- Understand how each vulnerability works
- Implement prevention measures in Node.js/Express
- Test your application for common vulnerabilities
- Apply security best practices throughout development

---

## OWASP Top 10 (2021)

| Rank | Risk | Description |
|------|------|-------------|
| A01 | Broken Access Control | Unauthorized access to resources |
| A02 | Cryptographic Failures | Weak encryption, exposed data |
| A03 | Injection | SQL, NoSQL, command injection |
| A04 | Insecure Design | Flawed architecture |
| A05 | Security Misconfiguration | Default/insecure settings |
| A06 | Vulnerable Components | Outdated dependencies |
| A07 | Authentication Failures | Weak auth mechanisms |
| A08 | Data Integrity Failures | Untrusted data, insecure CI/CD |
| A09 | Security Logging Failures | Missing audit trails |
| A10 | SSRF | Server-side request forgery |

---

## A01: Broken Access Control

Users acting outside their intended permissions.

### Vulnerable Code

```javascript
// VULNERABLE: No authorization check
app.get('/api/users/:userId/profile', async (req, res) => {
  const user = await prisma.user.findUnique({
    where: { id: req.params.userId }
  });
  res.json(user);  // Anyone can view any profile!
});

// VULNERABLE: Insecure Direct Object Reference (IDOR)
app.get('/api/documents/:documentId', async (req, res) => {
  const doc = await prisma.document.findUnique({
    where: { id: req.params.documentId }
  });
  res.json(doc);  // User can access any document by guessing ID!
});
```

### Prevention

```javascript
// SECURE: Check ownership
app.get('/api/users/:userId/profile', authenticate, async (req, res) => {
  // Only allow access to own profile or if admin
  if (req.user.userId !== req.params.userId && req.user.role !== 'ADMIN') {
    return res.status(403).json({ error: 'Access denied' });
  }

  const user = await prisma.user.findUnique({
    where: { id: req.params.userId },
    select: { id: true, name: true, email: true }  // Don't expose sensitive fields
  });

  res.json(user);
});

// SECURE: Authorization middleware
const authorizeOwnership = (getResourceOwnerId) => {
  return async (req, res, next) => {
    const ownerId = await getResourceOwnerId(req);

    if (ownerId !== req.user.userId && req.user.role !== 'ADMIN') {
      return res.status(403).json({ error: 'Access denied' });
    }

    next();
  };
};

// Usage
app.get('/api/documents/:documentId',
  authenticate,
  authorizeOwnership(async (req) => {
    const doc = await prisma.document.findUnique({
      where: { id: req.params.documentId },
      select: { ownerId: true }
    });
    return doc?.ownerId;
  }),
  documentController.get
);
```

### Access Control Best Practices

```javascript
// 1. Deny by default
const isAuthorized = (user, resource, action) => {
  // Explicitly define allowed actions
  const permissions = {
    'ADMIN': ['*'],
    'USER': ['read:own', 'write:own'],
    'GUEST': ['read:public']
  };

  // Check if action is allowed
  const allowed = permissions[user.role] || [];
  return allowed.includes('*') || allowed.includes(action);
};

// 2. Use UUIDs instead of sequential IDs
// Bad: /api/orders/1, /api/orders/2 (easy to guess)
// Good: /api/orders/550e8400-e29b-41d4-a716-446655440000

// 3. Implement RBAC (Role-Based Access Control)
const authorize = (...allowedRoles) => {
  return (req, res, next) => {
    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }
    next();
  };
};

app.delete('/api/users/:id', authenticate, authorize('ADMIN'), deleteUser);
```

---

## A02: Cryptographic Failures

Exposure of sensitive data due to weak or missing encryption.

### Vulnerable Code

```javascript
// VULNERABLE: Storing passwords in plain text
const user = await prisma.user.create({
  data: {
    email,
    password: password  // Plain text!
  }
});

// VULNERABLE: Weak hashing
const hash = crypto.createHash('md5').update(password).digest('hex');

// VULNERABLE: Hardcoded secrets
const JWT_SECRET = 'my-super-secret-key';

// VULNERABLE: Exposing sensitive data in responses
app.get('/api/users/:id', async (req, res) => {
  const user = await prisma.user.findUnique({ where: { id: req.params.id } });
  res.json(user);  // Includes password hash, SSN, etc.
});
```

### Prevention

```javascript
// SECURE: Use bcrypt for passwords
const bcrypt = require('bcrypt');
const SALT_ROUNDS = 12;

const hashedPassword = await bcrypt.hash(password, SALT_ROUNDS);

// SECURE: Use environment variables for secrets
const JWT_SECRET = process.env.JWT_SECRET;
if (!JWT_SECRET || JWT_SECRET.length < 32) {
  throw new Error('JWT_SECRET must be at least 32 characters');
}

// SECURE: Select only needed fields
app.get('/api/users/:id', authenticate, async (req, res) => {
  const user = await prisma.user.findUnique({
    where: { id: req.params.id },
    select: {
      id: true,
      name: true,
      email: true,
      createdAt: true
      // Never include: password, ssn, creditCard, etc.
    }
  });
  res.json(user);
});

// SECURE: Encrypt sensitive data at rest
const crypto = require('crypto');

const ENCRYPTION_KEY = Buffer.from(process.env.ENCRYPTION_KEY, 'hex');
const IV_LENGTH = 16;

function encrypt(text) {
  const iv = crypto.randomBytes(IV_LENGTH);
  const cipher = crypto.createCipheriv('aes-256-gcm', ENCRYPTION_KEY, iv);
  const encrypted = Buffer.concat([cipher.update(text, 'utf8'), cipher.final()]);
  const authTag = cipher.getAuthTag();

  return Buffer.concat([iv, authTag, encrypted]).toString('base64');
}

function decrypt(encryptedText) {
  const buffer = Buffer.from(encryptedText, 'base64');
  const iv = buffer.subarray(0, IV_LENGTH);
  const authTag = buffer.subarray(IV_LENGTH, IV_LENGTH + 16);
  const encrypted = buffer.subarray(IV_LENGTH + 16);

  const decipher = crypto.createDecipheriv('aes-256-gcm', ENCRYPTION_KEY, iv);
  decipher.setAuthTag(authTag);

  return decipher.update(encrypted) + decipher.final('utf8');
}

// Encrypt SSN before storing
const encryptedSSN = encrypt(ssn);
```

---

## A03: Injection

Untrusted data sent as part of a command or query.

### SQL Injection

```javascript
// VULNERABLE: String concatenation in SQL
app.get('/api/search', async (req, res) => {
  const { query } = req.query;
  const results = await prisma.$queryRawUnsafe(
    `SELECT * FROM users WHERE name LIKE '%${query}%'`
  );
  res.json(results);
});

// Attack: ?query='; DROP TABLE users; --
```

### Prevention

```javascript
// SECURE: Parameterized queries
app.get('/api/search', async (req, res) => {
  const { query } = req.query;

  // Using Prisma (safe by default)
  const results = await prisma.user.findMany({
    where: {
      name: { contains: query }
    }
  });

  // Or with raw queries using parameters
  const results = await prisma.$queryRaw`
    SELECT * FROM users WHERE name LIKE ${`%${query}%`}
  `;

  res.json(results);
});
```

### NoSQL Injection

```javascript
// VULNERABLE: MongoDB query injection
app.post('/api/login', async (req, res) => {
  const user = await db.collection('users').findOne({
    email: req.body.email,
    password: req.body.password
  });
  // Attack: { "email": "admin@example.com", "password": { "$ne": "" } }
});

// SECURE: Validate and sanitize input
const { body, validationResult } = require('express-validator');

app.post('/api/login',
  body('email').isEmail().normalizeEmail(),
  body('password').isString().isLength({ min: 1 }),
  async (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }

    // Now safe to use
    const { email, password } = req.body;
  }
);
```

### Command Injection

```javascript
// VULNERABLE: Executing user input as command
const { exec } = require('child_process');

app.get('/api/ping', (req, res) => {
  const { host } = req.query;
  exec(`ping -c 4 ${host}`, (error, stdout) => {
    res.send(stdout);
  });
  // Attack: ?host=google.com; rm -rf /
});

// SECURE: Use execFile with arguments array
const { execFile } = require('child_process');

app.get('/api/ping', (req, res) => {
  const { host } = req.query;

  // Validate input
  const hostRegex = /^[a-zA-Z0-9.-]+$/;
  if (!hostRegex.test(host)) {
    return res.status(400).json({ error: 'Invalid hostname' });
  }

  execFile('ping', ['-c', '4', host], (error, stdout) => {
    if (error) {
      return res.status(500).json({ error: 'Ping failed' });
    }
    res.send(stdout);
  });
});
```

---

## A04: Insecure Design

Flaws in the application's fundamental design.

### Examples of Insecure Design

```javascript
// INSECURE: No rate limiting on password reset
app.post('/api/forgot-password', async (req, res) => {
  // Attacker can enumerate valid emails by response time
  const user = await prisma.user.findUnique({
    where: { email: req.body.email }
  });

  if (!user) {
    return res.status(404).json({ error: 'User not found' });  // Information leak!
  }

  // Send reset email...
});

// INSECURE: Predictable resource locations
// /uploads/invoice_001.pdf, invoice_002.pdf  // Easy to enumerate
```

### Secure Design Patterns

```javascript
// SECURE: Constant-time responses
app.post('/api/forgot-password',
  rateLimit({ windowMs: 60 * 60 * 1000, max: 3 }),
  async (req, res) => {
    const { email } = req.body;

    // Always perform lookup and respond with same message
    const user = await prisma.user.findUnique({ where: { email } });

    if (user) {
      await sendPasswordResetEmail(user);
    }

    // Same response regardless of user existence
    res.json({
      message: 'If an account exists, a reset email has been sent'
    });
  }
);

// SECURE: Unpredictable resource names
const crypto = require('crypto');

function generateSecureFilename(originalName) {
  const ext = path.extname(originalName);
  const randomName = crypto.randomBytes(16).toString('hex');
  return `${randomName}${ext}`;
}
// Result: 7f3a8b2c1d4e5f6a7b8c9d0e1f2a3b4c.pdf

// SECURE: Defense in depth
// Multiple layers of security
app.get('/api/admin/data',
  authenticate,              // Layer 1: Must be logged in
  authorize('ADMIN'),        // Layer 2: Must be admin
  rateLimit({ max: 100 }),   // Layer 3: Rate limited
  validateInput,             // Layer 4: Input validation
  auditLog,                  // Layer 5: Logging
  adminController.getData
);
```

---

## A05: Security Misconfiguration

Insecure default configurations and missing security settings.

### Common Misconfigurations

```javascript
// INSECURE: Debug mode in production
app.use(express.errorHandler({ dumpExceptions: true, showStack: true }));

// INSECURE: Exposed headers
// X-Powered-By: Express (reveals technology stack)

// INSECURE: CORS allowing all origins
app.use(cors({ origin: '*', credentials: true }));

// INSECURE: Missing security headers
// No CSP, HSTS, X-Frame-Options, etc.

// INSECURE: Default credentials
const config = {
  database: {
    user: 'admin',
    password: 'admin123'
  }
};
```

### Secure Configuration

```javascript
// SECURE: Environment-based configuration
const config = {
  isProduction: process.env.NODE_ENV === 'production',
  port: parseInt(process.env.PORT, 10) || 3000,
  database: {
    url: process.env.DATABASE_URL  // From environment
  }
};

// SECURE: Helmet for security headers
const helmet = require('helmet');
app.use(helmet());

// SECURE: Remove identifying headers
app.disable('x-powered-by');

// SECURE: Proper error handling
app.use((err, req, res, next) => {
  console.error(err);  // Log the full error

  // Don't expose error details in production
  res.status(500).json({
    error: config.isProduction
      ? 'Internal server error'
      : err.message
  });
});

// SECURE: Strict CORS
app.use(cors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') || [],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE']
}));
```

---

## A06: Vulnerable and Outdated Components

Using dependencies with known vulnerabilities.

### Detection

```bash
# Check for vulnerabilities
npm audit

# Check for outdated packages
npm outdated

# Use tools like Snyk
npx snyk test
```

### Prevention

```javascript
// package.json - Pin versions
{
  "dependencies": {
    "express": "4.18.2",      // Pinned version
    "lodash": "^4.17.21"      // Minor updates only
  }
}

// Use lock files
// package-lock.json ensures consistent installs

// Automated scanning in CI/CD
// .github/workflows/security.yml
```

```yaml
# .github/workflows/security.yml
name: Security Scan

on:
  push:
    branches: [main]
  schedule:
    - cron: '0 0 * * *'  # Daily

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm audit --audit-level=high
      - name: Snyk Security Scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

---

## A07: Identification and Authentication Failures

Weak authentication mechanisms.

### Vulnerabilities

```javascript
// INSECURE: Weak password requirements
if (password.length < 6) {
  return res.status(400).json({ error: 'Password too short' });
}

// INSECURE: No brute force protection
app.post('/api/login', async (req, res) => {
  // Unlimited login attempts!
});

// INSECURE: Session fixation
app.post('/api/login', (req, res) => {
  // Using same session ID after login
  req.session.userId = user.id;
});

// INSECURE: Exposing session in URL
// https://example.com/dashboard?sessionid=abc123
```

### Secure Authentication

```javascript
// SECURE: Strong password policy
const passwordSchema = require('password-validator');

const schema = new passwordSchema();
schema
  .is().min(12)
  .is().max(128)
  .has().uppercase()
  .has().lowercase()
  .has().digits()
  .has().symbols()
  .has().not().spaces()
  .is().not().oneOf(['Password123!', 'Qwerty123!']);

// SECURE: Rate limiting on auth
app.post('/api/login',
  rateLimit({ windowMs: 15 * 60 * 1000, max: 5 }),
  loginController
);

// SECURE: Regenerate session after login
app.post('/api/login', (req, res) => {
  // ... validate credentials

  req.session.regenerate((err) => {
    req.session.userId = user.id;
    res.json({ message: 'Logged in' });
  });
});

// SECURE: Multi-factor authentication
const speakeasy = require('speakeasy');

function verifyTOTP(secret, token) {
  return speakeasy.totp.verify({
    secret,
    encoding: 'base32',
    token,
    window: 1
  });
}
```

---

## A08: Software and Data Integrity Failures

Not verifying integrity of software updates or data.

### Vulnerabilities

```javascript
// INSECURE: Loading scripts from untrusted CDN without integrity
<script src="https://cdn.example.com/lib.js"></script>

// INSECURE: Deserializing untrusted data
const data = JSON.parse(req.body.data);
eval(data.code);  // Arbitrary code execution!

// INSECURE: No signature verification on webhooks
app.post('/webhooks/payment', (req, res) => {
  processPayment(req.body);  // Anyone can call this!
});
```

### Secure Integrity Checks

```javascript
// SECURE: Subresource Integrity (SRI)
<script
  src="https://cdn.example.com/lib.js"
  integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/..."
  crossorigin="anonymous"
></script>

// SECURE: Webhook signature verification (Stripe example)
const stripe = require('stripe');

app.post('/webhooks/stripe',
  express.raw({ type: 'application/json' }),
  (req, res) => {
    const sig = req.headers['stripe-signature'];

    try {
      const event = stripe.webhooks.constructEvent(
        req.body,
        sig,
        process.env.STRIPE_WEBHOOK_SECRET
      );

      // Event is verified
      handleStripeEvent(event);
      res.json({ received: true });
    } catch (err) {
      res.status(400).json({ error: 'Invalid signature' });
    }
  }
);

// SECURE: Safe deserialization
function safeParseJSON(json, allowedKeys) {
  const data = JSON.parse(json);

  // Only allow specific keys
  const safe = {};
  for (const key of allowedKeys) {
    if (key in data) {
      safe[key] = data[key];
    }
  }

  return safe;
}
```

---

## A09: Security Logging and Monitoring Failures

Insufficient logging for security events.

### What to Log

```javascript
// Security events to log
const securityEvents = [
  'login_success',
  'login_failure',
  'password_change',
  'password_reset_request',
  'account_lockout',
  'permission_denied',
  'suspicious_activity',
  'data_export',
  'admin_action'
];

// Logging implementation
const winston = require('winston');

const securityLogger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.File({ filename: 'security.log' }),
    new winston.transports.Console()
  ]
});

// Log security events
function logSecurityEvent(event, details) {
  securityLogger.info({
    event,
    timestamp: new Date().toISOString(),
    ...details
  });
}

// Usage
app.post('/api/login', async (req, res) => {
  const { email, password } = req.body;

  try {
    const user = await authenticate(email, password);

    logSecurityEvent('login_success', {
      userId: user.id,
      email,
      ip: req.ip,
      userAgent: req.headers['user-agent']
    });

    res.json({ token: generateToken(user) });
  } catch (error) {
    logSecurityEvent('login_failure', {
      email,
      ip: req.ip,
      reason: error.message,
      userAgent: req.headers['user-agent']
    });

    res.status(401).json({ error: 'Invalid credentials' });
  }
});
```

### Alert on Suspicious Activity

```javascript
// Detect and alert on suspicious patterns
const suspiciousPatterns = {
  bruteForce: async (ip) => {
    const failures = await redis.get(`login_failures:${ip}`);
    return parseInt(failures) >= 10;
  },

  accountEnumeration: async (ip) => {
    const attempts = await redis.get(`email_checks:${ip}`);
    return parseInt(attempts) >= 20;
  }
};

async function checkSuspiciousActivity(req) {
  const ip = req.ip;

  if (await suspiciousPatterns.bruteForce(ip)) {
    logSecurityEvent('suspicious_activity', {
      type: 'brute_force_attempt',
      ip,
      action: 'blocked'
    });

    // Alert security team
    await sendSecurityAlert({
      type: 'brute_force',
      ip,
      severity: 'high'
    });

    return true;
  }

  return false;
}
```

---

## A10: Server-Side Request Forgery (SSRF)

Server making requests to unintended locations.

### Vulnerability

```javascript
// VULNERABLE: User-controlled URL
app.get('/api/fetch', async (req, res) => {
  const { url } = req.query;
  const response = await fetch(url);  // Can access internal services!
  res.json(await response.json());
});

// Attack: ?url=http://169.254.169.254/latest/meta-data/
// (AWS metadata endpoint - can expose credentials)
```

### Prevention

```javascript
const { URL } = require('url');
const dns = require('dns').promises;

// Validate and sanitize URLs
async function isUrlSafe(urlString) {
  try {
    const url = new URL(urlString);

    // Only allow HTTPS
    if (url.protocol !== 'https:') {
      return false;
    }

    // Block private/internal IPs
    const addresses = await dns.lookup(url.hostname, { all: true });

    for (const addr of addresses) {
      if (isPrivateIP(addr.address)) {
        return false;
      }
    }

    // Allowlist of permitted domains
    const allowedDomains = ['api.example.com', 'cdn.example.com'];
    if (!allowedDomains.includes(url.hostname)) {
      return false;
    }

    return true;
  } catch {
    return false;
  }
}

function isPrivateIP(ip) {
  const privateRanges = [
    /^127\./,                    // Loopback
    /^10\./,                     // Private Class A
    /^172\.(1[6-9]|2\d|3[01])\./, // Private Class B
    /^192\.168\./,               // Private Class C
    /^169\.254\./,               // Link-local
    /^0\./,                      // Current network
    /^::1$/,                     // IPv6 loopback
    /^fe80:/i                    // IPv6 link-local
  ];

  return privateRanges.some(range => range.test(ip));
}

// SECURE: Validate before fetching
app.get('/api/fetch', async (req, res) => {
  const { url } = req.query;

  if (!await isUrlSafe(url)) {
    return res.status(400).json({ error: 'URL not allowed' });
  }

  const response = await fetch(url, {
    timeout: 5000,
    follow: 0  // Don't follow redirects
  });

  res.json(await response.json());
});
```

---

## Security Checklist

```markdown
## Pre-Deployment Security Checklist

### Authentication & Authorization
- [ ] Passwords hashed with bcrypt (cost 12+)
- [ ] JWTs signed with strong secret
- [ ] Access control on all endpoints
- [ ] Rate limiting on auth endpoints
- [ ] Session regeneration on login

### Input Validation
- [ ] All user input validated
- [ ] Parameterized queries only
- [ ] File upload restrictions
- [ ] Content-Type validation

### Data Protection
- [ ] HTTPS enforced
- [ ] Sensitive data encrypted at rest
- [ ] No sensitive data in logs
- [ ] PII anonymized where possible

### Security Headers
- [ ] CSP configured
- [ ] HSTS enabled
- [ ] X-Frame-Options set
- [ ] X-Content-Type-Options: nosniff

### Dependencies
- [ ] npm audit clean
- [ ] Dependencies up to date
- [ ] No unused dependencies

### Logging & Monitoring
- [ ] Security events logged
- [ ] Alerts configured
- [ ] Error handling doesn't leak info
```

---

## Practice Exercise

Conduct a security audit:

1. **Review your codebase** for each OWASP Top 10 risk
2. **Run automated scans**: `npm audit`, OWASP ZAP
3. **Test manually**:
   - Try SQL injection in input fields
   - Test IDOR by changing IDs
   - Check if you can bypass authorization
4. **Fix identified vulnerabilities**
5. **Document security measures** implemented

---

## Key Takeaways

1. **Defense in depth** - Multiple layers of security
2. **Validate all input** - Never trust user data
3. **Principle of least privilege** - Minimum required permissions
4. **Keep dependencies updated** - Regularly audit packages
5. **Log security events** - Enable incident response
6. **Test regularly** - Automated and manual security testing
7. **Security is ongoing** - Not a one-time task

---

## Resources

- [OWASP Top 10](https://owasp.org/Top10/)
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)
- [Node.js Security Checklist](https://blog.risingstack.com/node-js-security-checklist/)
- [OWASP ZAP](https://www.zaproxy.org/) - Free security testing tool

---

## Congratulations!

You've completed the Auth & Security section! You now have a solid foundation in:
- Password hashing and secure storage
- JWT authentication and authorization
- OAuth social login
- Security headers and CORS
- Rate limiting
- Common vulnerabilities and prevention

In the next section, we'll explore **API Design** best practices to build well-structured, maintainable APIs.
