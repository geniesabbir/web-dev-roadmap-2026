# Day 6: Error Handling in Express

## Introduction

Robust error handling is critical for building production-ready applications. This lesson covers comprehensive strategies for handling errors, creating custom error classes, building global error handlers, and providing meaningful error responses.

## Learning Objectives

By the end of this lesson, you will:
- Understand Express error handling patterns
- Create custom error classes
- Build global error handling middleware
- Handle async errors properly
- Implement environment-specific error responses
- Log errors effectively

---

## Express Error Handling Basics

### Synchronous Errors

```javascript
// Express catches sync errors automatically
app.get('/sync-error', (req, res) => {
  throw new Error('Sync error!');
  // Express catches this and passes to error handler
});
```

### Asynchronous Errors

```javascript
// Async errors MUST be passed to next()
app.get('/async-error', async (req, res, next) => {
  try {
    await someAsyncOperation();
    res.json({ success: true });
  } catch (error) {
    next(error); // Pass to error handler
  }
});

// Without try/catch, error is NOT caught!
app.get('/unhandled', async (req, res) => {
  await someFailingOperation(); // UNCAUGHT! Server crashes
});
```

### Error Handling Middleware

```javascript
// Must have 4 parameters
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Something went wrong!' });
});

// Order matters - must be after all routes
app.get('/route1', handler1);
app.get('/route2', handler2);
app.use(errorHandler); // Last
```

---

## Custom Error Classes

### Base Application Error

```javascript
// errors/AppError.js
class AppError extends Error {
  constructor(message, statusCode, errorCode = 'INTERNAL_ERROR') {
    super(message);

    this.statusCode = statusCode;
    this.status = `${statusCode}`.startsWith('4') ? 'fail' : 'error';
    this.errorCode = errorCode;
    this.isOperational = true;

    Error.captureStackTrace(this, this.constructor);
  }
}

module.exports = AppError;
```

### Specific Error Classes

```javascript
// errors/index.js
const AppError = require('./AppError');

class NotFoundError extends AppError {
  constructor(resource = 'Resource') {
    super(`${resource} not found`, 404, 'NOT_FOUND');
  }
}

class ValidationError extends AppError {
  constructor(message, errors = []) {
    super(message, 400, 'VALIDATION_ERROR');
    this.errors = errors;
  }
}

class UnauthorizedError extends AppError {
  constructor(message = 'Not authenticated') {
    super(message, 401, 'UNAUTHORIZED');
  }
}

class ForbiddenError extends AppError {
  constructor(message = 'Not authorized to access this resource') {
    super(message, 403, 'FORBIDDEN');
  }
}

class ConflictError extends AppError {
  constructor(message = 'Resource already exists') {
    super(message, 409, 'CONFLICT');
  }
}

class RateLimitError extends AppError {
  constructor(retryAfter) {
    super('Too many requests', 429, 'RATE_LIMIT_EXCEEDED');
    this.retryAfter = retryAfter;
  }
}

module.exports = {
  AppError,
  NotFoundError,
  ValidationError,
  UnauthorizedError,
  ForbiddenError,
  ConflictError,
  RateLimitError,
};
```

### Using Custom Errors

```javascript
const { NotFoundError, ValidationError, ForbiddenError } = require('./errors');

// In route handlers
app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);

    if (!user) {
      throw new NotFoundError('User');
    }

    res.json(user);
  } catch (err) {
    next(err);
  }
});

app.post('/users', async (req, res, next) => {
  try {
    const errors = validateUser(req.body);

    if (errors.length > 0) {
      throw new ValidationError('Invalid input', errors);
    }

    const user = await User.create(req.body);
    res.status(201).json(user);
  } catch (err) {
    next(err);
  }
});

app.delete('/posts/:id', async (req, res, next) => {
  try {
    const post = await Post.findById(req.params.id);

    if (!post) {
      throw new NotFoundError('Post');
    }

    if (post.author.toString() !== req.user.id) {
      throw new ForbiddenError('You can only delete your own posts');
    }

    await post.remove();
    res.status(204).send();
  } catch (err) {
    next(err);
  }
});
```

---

## Async Handler Wrapper

### The catchAsync Pattern

```javascript
// utils/catchAsync.js
const catchAsync = (fn) => {
  return (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
};

module.exports = catchAsync;
```

### Usage

```javascript
const catchAsync = require('./utils/catchAsync');
const { NotFoundError } = require('./errors');

// No try/catch needed!
app.get('/users/:id', catchAsync(async (req, res) => {
  const user = await User.findById(req.params.id);

  if (!user) {
    throw new NotFoundError('User');
  }

  res.json(user);
}));

app.post('/users', catchAsync(async (req, res) => {
  const user = await User.create(req.body);
  res.status(201).json(user);
}));

// Any error thrown will be caught and passed to error handler
```

### Alternative: Express 5

Express 5 (beta) handles async errors automatically:

```javascript
// Express 5 (no wrapper needed)
app.get('/users/:id', async (req, res) => {
  const user = await User.findById(req.params.id);
  res.json(user);
});
// Errors automatically passed to error handler
```

---

## Global Error Handler

### Comprehensive Error Handler

```javascript
// middleware/errorHandler.js
const { AppError } = require('../errors');

// Handle specific error types
const handleCastErrorDB = (err) => {
  const message = `Invalid ${err.path}: ${err.value}`;
  return new AppError(message, 400, 'INVALID_ID');
};

const handleDuplicateFieldsDB = (err) => {
  const field = Object.keys(err.keyValue)[0];
  const message = `${field} already exists`;
  return new AppError(message, 409, 'DUPLICATE_FIELD');
};

const handleValidationErrorDB = (err) => {
  const errors = Object.values(err.errors).map(e => ({
    field: e.path,
    message: e.message,
  }));
  return new AppError('Validation failed', 400, 'VALIDATION_ERROR', errors);
};

const handleJWTError = () =>
  new AppError('Invalid token', 401, 'INVALID_TOKEN');

const handleJWTExpiredError = () =>
  new AppError('Token expired', 401, 'TOKEN_EXPIRED');

// Development error response
const sendErrorDev = (err, req, res) => {
  res.status(err.statusCode).json({
    status: err.status,
    error: err,
    message: err.message,
    stack: err.stack,
  });
};

// Production error response
const sendErrorProd = (err, req, res) => {
  // Operational error: send message to client
  if (err.isOperational) {
    const response = {
      status: err.status,
      code: err.errorCode,
      message: err.message,
    };

    // Include validation errors if present
    if (err.errors) {
      response.errors = err.errors;
    }

    return res.status(err.statusCode).json(response);
  }

  // Programming or unknown error: don't leak details
  console.error('ERROR:', err);

  res.status(500).json({
    status: 'error',
    code: 'INTERNAL_ERROR',
    message: 'Something went wrong',
  });
};

// Main error handler
const errorHandler = (err, req, res, next) => {
  err.statusCode = err.statusCode || 500;
  err.status = err.status || 'error';

  if (process.env.NODE_ENV === 'development') {
    sendErrorDev(err, req, res);
  } else {
    let error = Object.assign(Object.create(Object.getPrototypeOf(err)), err);

    // Handle specific error types
    if (err.name === 'CastError') error = handleCastErrorDB(err);
    if (err.code === 11000) error = handleDuplicateFieldsDB(err);
    if (err.name === 'ValidationError') error = handleValidationErrorDB(err);
    if (err.name === 'JsonWebTokenError') error = handleJWTError();
    if (err.name === 'TokenExpiredError') error = handleJWTExpiredError();

    sendErrorProd(error, req, res);
  }
};

module.exports = errorHandler;
```

### 404 Handler

```javascript
// Must be before error handler, after all routes
app.use((req, res, next) => {
  next(new NotFoundError(`Cannot ${req.method} ${req.originalUrl}`));
});

app.use(errorHandler);
```

---

## Error Logging

### Winston Logger Setup

```bash
npm install winston
```

```javascript
// utils/logger.js
const winston = require('winston');
const path = require('path');

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: { service: 'api' },
  transports: [
    // Error logs
    new winston.transports.File({
      filename: path.join('logs', 'error.log'),
      level: 'error',
    }),
    // All logs
    new winston.transports.File({
      filename: path.join('logs', 'combined.log'),
    }),
  ],
});

// Console logging in development
if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.combine(
      winston.format.colorize(),
      winston.format.simple()
    ),
  }));
}

module.exports = logger;
```

### Error Logging Middleware

```javascript
const logger = require('./utils/logger');

const errorHandler = (err, req, res, next) => {
  // Log error
  logger.error({
    message: err.message,
    stack: err.stack,
    statusCode: err.statusCode,
    url: req.originalUrl,
    method: req.method,
    ip: req.ip,
    userId: req.user?.id,
    body: req.body,
    query: req.query,
  });

  // Send response...
};
```

### Request ID for Tracing

```javascript
const { v4: uuidv4 } = require('uuid');

// Add request ID middleware
app.use((req, res, next) => {
  req.id = req.headers['x-request-id'] || uuidv4();
  res.set('X-Request-Id', req.id);
  next();
});

// Include in logs
logger.error({
  requestId: req.id,
  message: err.message,
  // ...
});
```

---

## Error Response Standards

### Consistent Error Format

```javascript
// Success response
{
  "status": "success",
  "data": { ... }
}

// Error response
{
  "status": "fail",     // Client error (4xx)
  "code": "VALIDATION_ERROR",
  "message": "Invalid input data",
  "errors": [
    { "field": "email", "message": "Invalid email format" },
    { "field": "password", "message": "Password too short" }
  ]
}

// Server error response
{
  "status": "error",    // Server error (5xx)
  "code": "INTERNAL_ERROR",
  "message": "Something went wrong"
}
```

### Response Helper

```javascript
// utils/response.js
const success = (res, data, statusCode = 200) => {
  res.status(statusCode).json({
    status: 'success',
    data,
  });
};

const created = (res, data) => success(res, data, 201);

const noContent = (res) => res.status(204).send();

const paginated = (res, data, pagination) => {
  res.status(200).json({
    status: 'success',
    data,
    pagination,
  });
};

module.exports = { success, created, noContent, paginated };
```

---

## Handling Specific Scenarios

### Database Connection Errors

```javascript
const mongoose = require('mongoose');

mongoose.connection.on('error', (err) => {
  logger.error('MongoDB connection error:', err);
});

mongoose.connection.on('disconnected', () => {
  logger.warn('MongoDB disconnected');
});

// Graceful shutdown
process.on('SIGINT', async () => {
  await mongoose.connection.close();
  process.exit(0);
});
```

### Unhandled Rejections and Exceptions

```javascript
// server.js
process.on('uncaughtException', (err) => {
  logger.error('UNCAUGHT EXCEPTION:', err);
  process.exit(1);
});

process.on('unhandledRejection', (reason, promise) => {
  logger.error('UNHANDLED REJECTION:', reason);
  // Close server gracefully
  server.close(() => {
    process.exit(1);
  });
});

const server = app.listen(PORT, () => {
  logger.info(`Server running on port ${PORT}`);
});
```

### Request Timeout

```javascript
const timeout = require('connect-timeout');

// Set timeout for all routes
app.use(timeout('30s'));

// Check timeout after slow operations
app.use((req, res, next) => {
  if (!req.timedout) next();
});

// Handle timeout
app.use((err, req, res, next) => {
  if (req.timedout) {
    return res.status(503).json({
      status: 'error',
      code: 'TIMEOUT',
      message: 'Request timed out',
    });
  }
  next(err);
});
```

---

## Complete Error Handling Setup

```javascript
// app.js
const express = require('express');
const { NotFoundError } = require('./errors');
const errorHandler = require('./middleware/errorHandler');
const catchAsync = require('./utils/catchAsync');
const logger = require('./utils/logger');

const app = express();

// Middleware
app.use(express.json());
app.use((req, res, next) => {
  req.id = require('uuid').v4();
  res.set('X-Request-Id', req.id);
  next();
});

// Request logging
app.use((req, res, next) => {
  logger.info({
    requestId: req.id,
    method: req.method,
    url: req.url,
    ip: req.ip,
  });
  next();
});

// Routes
app.use('/api/users', require('./routes/users'));
app.use('/api/posts', require('./routes/posts'));

// 404 handler
app.use((req, res, next) => {
  next(new NotFoundError(`Cannot ${req.method} ${req.originalUrl}`));
});

// Global error handler
app.use(errorHandler);

module.exports = app;
```

```javascript
// server.js
require('dotenv').config();
const app = require('./app');
const logger = require('./utils/logger');
const mongoose = require('mongoose');

// Handle uncaught exceptions
process.on('uncaughtException', (err) => {
  logger.error('UNCAUGHT EXCEPTION! Shutting down...', err);
  process.exit(1);
});

// Connect to database
mongoose.connect(process.env.DATABASE_URL)
  .then(() => logger.info('Database connected'))
  .catch((err) => {
    logger.error('Database connection failed:', err);
    process.exit(1);
  });

// Start server
const PORT = process.env.PORT || 3000;
const server = app.listen(PORT, () => {
  logger.info(`Server running on port ${PORT}`);
});

// Handle unhandled promise rejections
process.on('unhandledRejection', (err) => {
  logger.error('UNHANDLED REJECTION! Shutting down...', err);
  server.close(() => process.exit(1));
});

// Graceful shutdown
process.on('SIGTERM', () => {
  logger.info('SIGTERM received. Shutting down gracefully...');
  server.close(() => {
    logger.info('Process terminated');
  });
});
```

---

## Exercise: Implement Error Handling

Create a complete error handling system for a user authentication API:
1. Custom error classes
2. Async handler wrapper
3. Global error handler
4. Environment-specific responses

---

## Solution

```javascript
// errors/index.js
class AppError extends Error {
  constructor(message, statusCode, code = 'ERROR') {
    super(message);
    this.statusCode = statusCode;
    this.status = statusCode < 500 ? 'fail' : 'error';
    this.code = code;
    this.isOperational = true;
    Error.captureStackTrace(this, this.constructor);
  }
}

class BadRequestError extends AppError {
  constructor(message = 'Bad request', errors = null) {
    super(message, 400, 'BAD_REQUEST');
    this.errors = errors;
  }
}

class UnauthorizedError extends AppError {
  constructor(message = 'Unauthorized') {
    super(message, 401, 'UNAUTHORIZED');
  }
}

class NotFoundError extends AppError {
  constructor(resource = 'Resource') {
    super(`${resource} not found`, 404, 'NOT_FOUND');
  }
}

class ConflictError extends AppError {
  constructor(message = 'Conflict') {
    super(message, 409, 'CONFLICT');
  }
}

module.exports = {
  AppError,
  BadRequestError,
  UnauthorizedError,
  NotFoundError,
  ConflictError,
};

// utils/catchAsync.js
module.exports = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

// middleware/errorHandler.js
const errorHandler = (err, req, res, next) => {
  err.statusCode = err.statusCode || 500;
  err.status = err.status || 'error';

  // Development: full error details
  if (process.env.NODE_ENV === 'development') {
    return res.status(err.statusCode).json({
      status: err.status,
      code: err.code || 'INTERNAL_ERROR',
      message: err.message,
      errors: err.errors,
      stack: err.stack,
    });
  }

  // Production: safe error response
  if (err.isOperational) {
    return res.status(err.statusCode).json({
      status: err.status,
      code: err.code,
      message: err.message,
      ...(err.errors && { errors: err.errors }),
    });
  }

  // Unknown error
  console.error('ERROR:', err);
  res.status(500).json({
    status: 'error',
    code: 'INTERNAL_ERROR',
    message: 'An unexpected error occurred',
  });
};

module.exports = errorHandler;

// routes/auth.js
const express = require('express');
const router = express.Router();
const catchAsync = require('../utils/catchAsync');
const {
  BadRequestError,
  UnauthorizedError,
  NotFoundError,
  ConflictError,
} = require('../errors');

// Simulated user store
const users = new Map();

router.post('/register', catchAsync(async (req, res) => {
  const { email, password, name } = req.body;

  // Validation
  const errors = [];
  if (!email) errors.push({ field: 'email', message: 'Email is required' });
  if (!password) errors.push({ field: 'password', message: 'Password is required' });
  if (password && password.length < 8) {
    errors.push({ field: 'password', message: 'Password must be 8+ characters' });
  }

  if (errors.length > 0) {
    throw new BadRequestError('Validation failed', errors);
  }

  // Check if user exists
  if (users.has(email)) {
    throw new ConflictError('Email already registered');
  }

  // Create user
  const user = { id: Date.now(), email, name, password };
  users.set(email, user);

  res.status(201).json({
    status: 'success',
    data: { id: user.id, email: user.email, name: user.name },
  });
}));

router.post('/login', catchAsync(async (req, res) => {
  const { email, password } = req.body;

  if (!email || !password) {
    throw new BadRequestError('Email and password are required');
  }

  const user = users.get(email);

  if (!user || user.password !== password) {
    throw new UnauthorizedError('Invalid credentials');
  }

  // Generate token (simplified)
  const token = Buffer.from(`${user.id}:${Date.now()}`).toString('base64');

  res.json({
    status: 'success',
    data: { token },
  });
}));

router.get('/profile/:id', catchAsync(async (req, res) => {
  const user = Array.from(users.values()).find(
    u => u.id === parseInt(req.params.id)
  );

  if (!user) {
    throw new NotFoundError('User');
  }

  res.json({
    status: 'success',
    data: { id: user.id, email: user.email, name: user.name },
  });
}));

module.exports = router;

// app.js
const express = require('express');
const { NotFoundError } = require('./errors');
const errorHandler = require('./middleware/errorHandler');
const authRoutes = require('./routes/auth');

const app = express();

app.use(express.json());
app.use('/api/auth', authRoutes);

app.use((req, res, next) => {
  next(new NotFoundError('Endpoint'));
});

app.use(errorHandler);

app.listen(3000, () => console.log('Server running on port 3000'));
```

---

## Key Takeaways

1. **Custom error classes** - Create meaningful, categorized errors
2. **catchAsync wrapper** - Eliminates try/catch boilerplate
3. **Global error handler** - Centralized error processing
4. **Environment-aware responses** - Full details in dev, safe in prod
5. **Proper logging** - Log errors with context for debugging
6. **Graceful shutdown** - Handle uncaught errors properly
7. **Consistent format** - Use standard error response structure

---

## What's Next?

Tomorrow we'll learn about **File Uploads** in Express - handling multipart forms, storing files, and working with cloud storage!
