# Day 4: Express Middleware

## Introduction

Middleware is the backbone of Express applications. It's a powerful pattern that allows you to process requests through a pipeline of functions. Understanding middleware is essential for building robust, maintainable Express applications.

## Learning Objectives

By the end of this lesson, you will:
- Understand what middleware is and how it works
- Create custom middleware functions
- Use built-in and third-party middleware
- Implement authentication and authorization middleware
- Handle errors with middleware
- Organize middleware in your application

---

## What is Middleware?

Middleware functions are functions that have access to:
- The request object (`req`)
- The response object (`res`)
- The next middleware function (`next`)

```javascript
// Basic middleware structure
function middleware(req, res, next) {
  // Do something with req/res
  console.log('Middleware executed');

  // Call next() to continue to next middleware
  next();

  // Or send a response to end the cycle
  // res.send('Done');
}
```

### The Request-Response Cycle

```
Request → Middleware 1 → Middleware 2 → Route Handler → Response
              ↓              ↓
           next()         next()
```

### Middleware Can:

1. **Execute any code**
2. **Modify request and response objects**
3. **End the request-response cycle**
4. **Call the next middleware function**

---

## Types of Middleware

### 1. Application-Level Middleware

```javascript
const express = require('express');
const app = express();

// Runs for ALL requests
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});

// Runs for specific path
app.use('/api', (req, res, next) => {
  console.log('API request');
  next();
});

// Runs for specific path and method
app.get('/users', (req, res, next) => {
  console.log('Getting users');
  next();
});
```

### 2. Router-Level Middleware

```javascript
const express = require('express');
const router = express.Router();

// Middleware for all routes in this router
router.use((req, res, next) => {
  console.log('Router middleware');
  next();
});

// Middleware for specific route
router.get('/:id', validateId, (req, res) => {
  res.json({ id: req.params.id });
});

module.exports = router;
```

### 3. Error-Handling Middleware

```javascript
// Error middleware has 4 parameters
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Something went wrong!' });
});
```

### 4. Built-in Middleware

```javascript
// Parse JSON bodies
app.use(express.json());

// Parse URL-encoded bodies
app.use(express.urlencoded({ extended: true }));

// Serve static files
app.use(express.static('public'));
```

### 5. Third-Party Middleware

```javascript
const morgan = require('morgan');
const cors = require('cors');
const helmet = require('helmet');

app.use(morgan('dev'));
app.use(cors());
app.use(helmet());
```

---

## Creating Custom Middleware

### Basic Logger

```javascript
function logger(req, res, next) {
  const start = Date.now();

  // Listen for response finish
  res.on('finish', () => {
    const duration = Date.now() - start;
    console.log(`${req.method} ${req.url} ${res.statusCode} - ${duration}ms`);
  });

  next();
}

app.use(logger);
```

### Request ID Middleware

```javascript
const crypto = require('crypto');

function requestId(req, res, next) {
  req.id = crypto.randomUUID();
  res.set('X-Request-Id', req.id);
  next();
}

app.use(requestId);

app.get('/test', (req, res) => {
  res.json({ requestId: req.id });
});
```

### Rate Limiter

```javascript
function createRateLimiter({ windowMs = 60000, max = 100 } = {}) {
  const requests = new Map();

  return (req, res, next) => {
    const ip = req.ip;
    const now = Date.now();
    const windowStart = now - windowMs;

    // Get existing requests for this IP
    const userRequests = requests.get(ip) || [];

    // Filter to only requests within window
    const recentRequests = userRequests.filter(time => time > windowStart);

    if (recentRequests.length >= max) {
      return res.status(429).json({
        error: 'Too many requests',
        retryAfter: Math.ceil((recentRequests[0] + windowMs - now) / 1000),
      });
    }

    // Add current request
    recentRequests.push(now);
    requests.set(ip, recentRequests);

    // Add rate limit headers
    res.set('X-RateLimit-Limit', max);
    res.set('X-RateLimit-Remaining', max - recentRequests.length);

    next();
  };
}

// Usage
app.use('/api', createRateLimiter({ windowMs: 60000, max: 100 }));
```

### Validation Middleware

```javascript
function validateBody(schema) {
  return (req, res, next) => {
    const errors = [];

    for (const [field, rules] of Object.entries(schema)) {
      const value = req.body[field];

      if (rules.required && !value) {
        errors.push(`${field} is required`);
        continue;
      }

      if (value) {
        if (rules.type && typeof value !== rules.type) {
          errors.push(`${field} must be a ${rules.type}`);
        }

        if (rules.minLength && value.length < rules.minLength) {
          errors.push(`${field} must be at least ${rules.minLength} characters`);
        }

        if (rules.maxLength && value.length > rules.maxLength) {
          errors.push(`${field} must be at most ${rules.maxLength} characters`);
        }

        if (rules.pattern && !rules.pattern.test(value)) {
          errors.push(`${field} format is invalid`);
        }
      }
    }

    if (errors.length > 0) {
      return res.status(400).json({ errors });
    }

    next();
  };
}

// Usage
const userSchema = {
  name: { required: true, type: 'string', minLength: 2 },
  email: { required: true, pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/ },
  age: { type: 'number' },
};

app.post('/users', validateBody(userSchema), (req, res) => {
  res.status(201).json(req.body);
});
```

---

## Authentication Middleware

### Basic Token Auth

```javascript
function authenticate(req, res, next) {
  const authHeader = req.headers.authorization;

  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'No token provided' });
  }

  const token = authHeader.split(' ')[1];

  try {
    // Verify token (example with JWT)
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    res.status(401).json({ error: 'Invalid token' });
  }
}

// Usage
app.use('/api/protected', authenticate);

app.get('/api/protected/profile', (req, res) => {
  res.json({ user: req.user });
});
```

### Role-Based Authorization

```javascript
function authorize(...roles) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Not authenticated' });
    }

    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Not authorized' });
    }

    next();
  };
}

// Usage
app.get('/api/admin/users', authenticate, authorize('admin'), (req, res) => {
  res.json({ message: 'Admin only content' });
});

app.get('/api/moderator', authenticate, authorize('admin', 'moderator'), (req, res) => {
  res.json({ message: 'Admin or moderator content' });
});
```

### API Key Authentication

```javascript
function apiKeyAuth(req, res, next) {
  const apiKey = req.headers['x-api-key'] || req.query.api_key;

  if (!apiKey) {
    return res.status(401).json({ error: 'API key required' });
  }

  // Validate API key (check database in real app)
  const validKeys = ['key-1234', 'key-5678'];

  if (!validKeys.includes(apiKey)) {
    return res.status(401).json({ error: 'Invalid API key' });
  }

  // Optionally attach user/app info to request
  req.apiClient = { key: apiKey };
  next();
}

app.use('/api/v1', apiKeyAuth);
```

---

## Error Handling Middleware

### Comprehensive Error Handler

```javascript
// Custom error class
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.status = `${statusCode}`.startsWith('4') ? 'fail' : 'error';
    this.isOperational = true;

    Error.captureStackTrace(this, this.constructor);
  }
}

// Async wrapper
const catchAsync = (fn) => (req, res, next) => {
  fn(req, res, next).catch(next);
};

// Error handler middleware
const errorHandler = (err, req, res, next) => {
  err.statusCode = err.statusCode || 500;
  err.status = err.status || 'error';

  if (process.env.NODE_ENV === 'development') {
    return res.status(err.statusCode).json({
      status: err.status,
      error: err,
      message: err.message,
      stack: err.stack,
    });
  }

  // Production: Don't leak error details
  if (err.isOperational) {
    return res.status(err.statusCode).json({
      status: err.status,
      message: err.message,
    });
  }

  // Programming or unknown error
  console.error('ERROR:', err);
  res.status(500).json({
    status: 'error',
    message: 'Something went wrong!',
  });
};

// Usage
app.get('/users/:id', catchAsync(async (req, res) => {
  const user = await User.findById(req.params.id);

  if (!user) {
    throw new AppError('User not found', 404);
  }

  res.json(user);
}));

// Must be last
app.use(errorHandler);
```

### Handle Specific Error Types

```javascript
const errorHandler = (err, req, res, next) => {
  let error = { ...err };
  error.message = err.message;

  // MongoDB CastError (invalid ObjectId)
  if (err.name === 'CastError') {
    error = new AppError(`Invalid ${err.path}: ${err.value}`, 400);
  }

  // MongoDB Duplicate Key Error
  if (err.code === 11000) {
    const field = Object.keys(err.keyValue)[0];
    error = new AppError(`${field} already exists`, 400);
  }

  // MongoDB Validation Error
  if (err.name === 'ValidationError') {
    const messages = Object.values(err.errors).map(e => e.message);
    error = new AppError(messages.join('. '), 400);
  }

  // JWT Errors
  if (err.name === 'JsonWebTokenError') {
    error = new AppError('Invalid token', 401);
  }

  if (err.name === 'TokenExpiredError') {
    error = new AppError('Token expired', 401);
  }

  res.status(error.statusCode || 500).json({
    status: error.status || 'error',
    message: error.message || 'Something went wrong',
  });
};
```

---

## Popular Third-Party Middleware

### Morgan (Logging)

```javascript
const morgan = require('morgan');

// Predefined formats
app.use(morgan('dev'));      // Colored status for development
app.use(morgan('combined')); // Apache combined format
app.use(morgan('common'));   // Apache common format
app.use(morgan('tiny'));     // Minimal output

// Custom format
morgan.token('id', req => req.id);
app.use(morgan(':id :method :url :status :response-time ms'));

// Write to file
const fs = require('fs');
const path = require('path');
const accessLogStream = fs.createWriteStream(
  path.join(__dirname, 'access.log'),
  { flags: 'a' }
);
app.use(morgan('combined', { stream: accessLogStream }));
```

### CORS

```javascript
const cors = require('cors');

// Allow all origins
app.use(cors());

// Allow specific origin
app.use(cors({ origin: 'http://localhost:3000' }));

// Allow multiple origins
app.use(cors({
  origin: ['http://localhost:3000', 'https://myapp.com'],
}));

// Dynamic origin
app.use(cors({
  origin: (origin, callback) => {
    const allowedOrigins = ['http://localhost:3000'];

    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
}));

// Full configuration
app.use(cors({
  origin: 'http://localhost:3000',
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  exposedHeaders: ['X-Request-Id'],
  credentials: true,
  maxAge: 86400, // 24 hours
}));
```

### Helmet (Security Headers)

```javascript
const helmet = require('helmet');

// Use all default protections
app.use(helmet());

// Or configure individually
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'", "'unsafe-inline'"],
    styleSrc: ["'self'", "'unsafe-inline'"],
    imgSrc: ["'self'", 'data:', 'https:'],
  },
}));

app.use(helmet.hsts({ maxAge: 31536000 }));
app.use(helmet.noSniff());
app.use(helmet.xssFilter());
```

### Compression

```javascript
const compression = require('compression');

// Compress all responses
app.use(compression());

// With options
app.use(compression({
  level: 6,        // Compression level (0-9)
  threshold: 1024, // Only compress responses > 1KB
  filter: (req, res) => {
    // Don't compress if 'x-no-compression' header is present
    if (req.headers['x-no-compression']) {
      return false;
    }
    return compression.filter(req, res);
  },
}));
```

### Cookie Parser

```javascript
const cookieParser = require('cookie-parser');

app.use(cookieParser());
app.use(cookieParser('secret-key')); // For signed cookies

app.get('/test', (req, res) => {
  // Read cookies
  console.log(req.cookies);       // { name: 'value' }
  console.log(req.signedCookies); // Signed cookies

  // Set cookies
  res.cookie('name', 'value', {
    httpOnly: true,
    secure: true,
    maxAge: 24 * 60 * 60 * 1000, // 1 day
  });

  res.send('Cookie set');
});
```

---

## Middleware Execution Order

### Understanding Order

```javascript
// Middleware runs in the order it's defined

app.use((req, res, next) => {
  console.log('1 - First middleware');
  next();
});

app.use((req, res, next) => {
  console.log('2 - Second middleware');
  next();
});

app.get('/test', (req, res, next) => {
  console.log('3 - Route handler');
  next();
});

app.use((req, res, next) => {
  console.log('4 - After route middleware');
  res.send('Done');
});

// Output:
// 1 - First middleware
// 2 - Second middleware
// 3 - Route handler
// 4 - After route middleware
```

### Skipping Middleware

```javascript
app.use((req, res, next) => {
  if (req.query.skip) {
    return next('route'); // Skip remaining route middleware
  }
  next();
});
```

### Conditional Middleware

```javascript
// Only apply in production
if (process.env.NODE_ENV === 'production') {
  app.use(compression());
  app.use(helmet());
}

// Only apply for specific paths
app.use('/api', authenticate);

// Conditional within middleware
const conditionalMiddleware = (req, res, next) => {
  if (req.path === '/health') {
    return next(); // Skip for health check
  }
  // Apply logic
  next();
};
```

---

## Organizing Middleware

### Middleware Factory Pattern

```javascript
// middleware/index.js
const morgan = require('morgan');
const cors = require('cors');
const helmet = require('helmet');
const compression = require('compression');
const rateLimit = require('express-rate-limit');

const setupMiddleware = (app) => {
  // Security
  app.use(helmet());
  app.use(cors({
    origin: process.env.CORS_ORIGIN,
    credentials: true,
  }));

  // Rate limiting
  app.use('/api', rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 100,
  }));

  // Logging
  if (process.env.NODE_ENV === 'development') {
    app.use(morgan('dev'));
  } else {
    app.use(morgan('combined'));
  }

  // Body parsing
  app.use(express.json({ limit: '10kb' }));
  app.use(express.urlencoded({ extended: true }));

  // Compression
  app.use(compression());
};

module.exports = setupMiddleware;
```

```javascript
// app.js
const express = require('express');
const setupMiddleware = require('./middleware');

const app = express();

setupMiddleware(app);

// Routes...
```

---

## Exercise: Build Middleware Suite

Create a set of reusable middleware:
1. Request timing middleware
2. Request body logger (for debugging)
3. Simple authentication check
4. Response formatting middleware

---

## Solution

```javascript
// middleware/timing.js
function timing(req, res, next) {
  const start = process.hrtime();

  res.on('finish', () => {
    const [seconds, nanoseconds] = process.hrtime(start);
    const duration = (seconds * 1000 + nanoseconds / 1000000).toFixed(2);

    console.log(`${req.method} ${req.url} - ${duration}ms`);
  });

  next();
}

// middleware/bodyLogger.js
function bodyLogger(req, res, next) {
  if (process.env.NODE_ENV !== 'development') {
    return next();
  }

  if (['POST', 'PUT', 'PATCH'].includes(req.method) && req.body) {
    console.log('Request Body:', JSON.stringify(req.body, null, 2));
  }

  next();
}

// middleware/auth.js
function simpleAuth(req, res, next) {
  const token = req.headers['x-auth-token'];

  if (!token) {
    return res.status(401).json({
      success: false,
      error: 'Authentication required',
    });
  }

  // Simple token validation (use JWT in production)
  if (token === 'valid-token-123') {
    req.user = { id: 1, name: 'John' };
    return next();
  }

  res.status(401).json({
    success: false,
    error: 'Invalid token',
  });
}

// middleware/responseFormat.js
function responseFormat(req, res, next) {
  // Override res.json
  const originalJson = res.json.bind(res);

  res.json = (data) => {
    const formatted = {
      success: res.statusCode < 400,
      timestamp: new Date().toISOString(),
      path: req.originalUrl,
      data: res.statusCode < 400 ? data : undefined,
      error: res.statusCode >= 400 ? data : undefined,
    };

    return originalJson(formatted);
  };

  next();
}

// app.js
const express = require('express');
const app = express();

// Apply middleware
app.use(express.json());
app.use(timing);
app.use(bodyLogger);
app.use(responseFormat);

// Public routes
app.get('/health', (req, res) => {
  res.json({ status: 'ok' });
});

// Protected routes
app.use('/api', simpleAuth);

app.get('/api/profile', (req, res) => {
  res.json({ user: req.user });
});

app.post('/api/data', (req, res) => {
  res.status(201).json({ received: req.body });
});

// Error handling
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).json({ error: 'Internal server error' });
});

app.listen(3000);
```

Test:

```bash
# Health check (no auth)
curl http://localhost:3000/health

# Protected route without token
curl http://localhost:3000/api/profile
# Returns: 401 error

# Protected route with token
curl -H "x-auth-token: valid-token-123" http://localhost:3000/api/profile

# POST with body
curl -X POST http://localhost:3000/api/data \
  -H "Content-Type: application/json" \
  -H "x-auth-token: valid-token-123" \
  -d '{"message": "Hello"}'
```

---

## Key Takeaways

1. **Middleware is a pipeline** - Requests flow through middleware in order
2. **Always call next()** - Unless you send a response
3. **Order matters** - Define middleware before routes
4. **Error middleware has 4 params** - (err, req, res, next)
5. **Use third-party middleware** - Don't reinvent the wheel
6. **Organize middleware** - Keep it modular and reusable
7. **Conditional application** - Apply middleware where needed

---

## What's Next?

Tomorrow we'll explore **Advanced Routing** - route parameters, query strings, route groups, and building RESTful routes!
