# Day 10: Capstone Project - Production-Ready API

## Introduction

Today we'll take the Blog API from Days 8-9 and make it truly production-ready. We'll add essential features like rate limiting, request logging, API documentation, health checks, graceful shutdown, and deployment preparation. This project consolidates everything you've learned about Node.js and Express.

## Learning Objectives

By the end of this lesson, you will:
- Implement rate limiting and security hardening
- Add comprehensive request logging with Winston
- Generate API documentation with Swagger/OpenAPI
- Implement health check endpoints
- Add graceful shutdown handling
- Configure environment-based settings
- Prepare the application for deployment
- Write a complete production deployment checklist

---

## Project Enhancement Overview

We'll enhance the Blog API with:

```
Production Features:
├── Security
│   ├── Rate limiting
│   ├── Request sanitization
│   ├── Security headers (Helmet)
│   └── CORS configuration
├── Monitoring
│   ├── Request logging (Winston)
│   ├── Health checks
│   ├── Error tracking
│   └── Performance metrics
├── Documentation
│   ├── Swagger/OpenAPI spec
│   └── API versioning
├── Reliability
│   ├── Graceful shutdown
│   ├── Process management
│   └── Environment configuration
└── Deployment
    ├── Docker setup
    ├── Environment variables
    └── Production checklist
```

---

## Step 1: Enhanced Project Structure

```
blog-api/
├── src/
│   ├── config/
│   │   ├── index.js
│   │   ├── database.js
│   │   ├── logger.js
│   │   └── swagger.js
│   ├── middleware/
│   │   ├── auth.js
│   │   ├── validate.js
│   │   ├── errorHandler.js
│   │   ├── notFound.js
│   │   ├── rateLimiter.js
│   │   └── requestLogger.js
│   ├── modules/
│   │   ├── auth/
│   │   ├── users/
│   │   ├── posts/
│   │   └── health/
│   │       ├── health.routes.js
│   │       └── health.controller.js
│   ├── utils/
│   │   ├── AppError.js
│   │   ├── catchAsync.js
│   │   ├── response.js
│   │   └── pagination.js
│   ├── app.js
│   └── server.js
├── logs/
├── .env
├── .env.example
├── .env.production
├── Dockerfile
├── docker-compose.yml
├── package.json
└── README.md
```

---

## Step 2: Advanced Configuration

### Environment Configuration

```javascript
// src/config/index.js
require('dotenv').config();

const config = {
  env: process.env.NODE_ENV || 'development',
  port: parseInt(process.env.PORT, 10) || 3000,

  // Database
  mongoUri: process.env.MONGODB_URI || 'mongodb://localhost:27017/blog-api',

  // JWT
  jwt: {
    secret: process.env.JWT_SECRET,
    expiresIn: process.env.JWT_EXPIRES_IN || '7d',
    refreshExpiresIn: process.env.JWT_REFRESH_EXPIRES_IN || '30d',
  },

  // Rate Limiting
  rateLimit: {
    windowMs: parseInt(process.env.RATE_LIMIT_WINDOW_MS, 10) || 15 * 60 * 1000,
    max: parseInt(process.env.RATE_LIMIT_MAX, 10) || 100,
  },

  // CORS
  cors: {
    origin: process.env.CORS_ORIGIN || '*',
    credentials: true,
  },

  // Logging
  logging: {
    level: process.env.LOG_LEVEL || 'info',
    format: process.env.LOG_FORMAT || 'combined',
  },

  // API
  api: {
    prefix: '/api',
    version: 'v1',
  },
};

// Validate required environment variables
const requiredEnvVars = ['JWT_SECRET'];

if (config.env === 'production') {
  requiredEnvVars.push('MONGODB_URI');
}

const missingEnvVars = requiredEnvVars.filter(envVar => !process.env[envVar]);

if (missingEnvVars.length > 0) {
  throw new Error(`Missing required environment variables: ${missingEnvVars.join(', ')}`);
}

module.exports = config;
```

### Environment Files

```bash
# .env.example
NODE_ENV=development
PORT=3000

# Database
MONGODB_URI=mongodb://localhost:27017/blog-api

# JWT
JWT_SECRET=your-secret-key-min-32-characters-long
JWT_EXPIRES_IN=7d
JWT_REFRESH_EXPIRES_IN=30d

# Rate Limiting
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX=100

# CORS
CORS_ORIGIN=http://localhost:3000

# Logging
LOG_LEVEL=debug
LOG_FORMAT=dev
```

```bash
# .env.production
NODE_ENV=production
PORT=3000

# Database (set in production environment)
# MONGODB_URI=

# JWT (set in production environment)
# JWT_SECRET=

# Rate Limiting
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX=50

# CORS
CORS_ORIGIN=https://yourdomain.com

# Logging
LOG_LEVEL=info
LOG_FORMAT=combined
```

---

## Step 3: Logging with Winston

### Logger Configuration

```javascript
// src/config/logger.js
const winston = require('winston');
const path = require('path');
const config = require('./index');

// Define log format
const logFormat = winston.format.combine(
  winston.format.timestamp({ format: 'YYYY-MM-DD HH:mm:ss' }),
  winston.format.errors({ stack: true }),
  winston.format.printf(({ timestamp, level, message, stack, ...meta }) => {
    let log = `${timestamp} [${level.toUpperCase()}]: ${message}`;
    if (Object.keys(meta).length > 0) {
      log += ` ${JSON.stringify(meta)}`;
    }
    if (stack) {
      log += `\n${stack}`;
    }
    return log;
  })
);

// Define transports based on environment
const transports = [
  // Console transport
  new winston.transports.Console({
    format: winston.format.combine(
      winston.format.colorize(),
      logFormat
    ),
  }),
];

// Add file transports in production
if (config.env === 'production') {
  transports.push(
    // Error logs
    new winston.transports.File({
      filename: path.join('logs', 'error.log'),
      level: 'error',
      format: logFormat,
      maxsize: 5242880, // 5MB
      maxFiles: 5,
    }),
    // Combined logs
    new winston.transports.File({
      filename: path.join('logs', 'combined.log'),
      format: logFormat,
      maxsize: 5242880, // 5MB
      maxFiles: 5,
    })
  );
}

// Create logger
const logger = winston.createLogger({
  level: config.logging.level,
  format: logFormat,
  transports,
  // Don't exit on error
  exitOnError: false,
});

// Create a stream for Morgan
logger.stream = {
  write: (message) => {
    logger.info(message.trim());
  },
};

module.exports = logger;
```

### Request Logger Middleware

```javascript
// src/middleware/requestLogger.js
const morgan = require('morgan');
const logger = require('../config/logger');
const config = require('../config');

// Custom token for response time in ms
morgan.token('response-time-ms', (req, res) => {
  const time = morgan['response-time'](req, res);
  return time ? `${time}ms` : '-';
});

// Custom token for user ID
morgan.token('user-id', (req) => {
  return req.user?.id || 'anonymous';
});

// Custom token for request body (sanitized)
morgan.token('body', (req) => {
  if (!req.body || Object.keys(req.body).length === 0) return '-';

  // Sanitize sensitive fields
  const sanitized = { ...req.body };
  const sensitiveFields = ['password', 'token', 'secret', 'authorization'];

  sensitiveFields.forEach(field => {
    if (sanitized[field]) {
      sanitized[field] = '[REDACTED]';
    }
  });

  return JSON.stringify(sanitized);
});

// Development format
const devFormat = ':method :url :status :response-time-ms - :user-id';

// Production format (JSON for log aggregation)
const prodFormat = JSON.stringify({
  method: ':method',
  url: ':url',
  status: ':status',
  responseTime: ':response-time-ms',
  contentLength: ':res[content-length]',
  userAgent: ':user-agent',
  ip: ':remote-addr',
  userId: ':user-id',
});

const requestLogger = morgan(
  config.env === 'production' ? prodFormat : devFormat,
  { stream: logger.stream }
);

module.exports = requestLogger;
```

---

## Step 4: Rate Limiting

```javascript
// src/middleware/rateLimiter.js
const rateLimit = require('express-rate-limit');
const config = require('../config');
const AppError = require('../utils/AppError');

// General API rate limiter
const apiLimiter = rateLimit({
  windowMs: config.rateLimit.windowMs,
  max: config.rateLimit.max,
  message: {
    status: 'error',
    message: 'Too many requests, please try again later.',
  },
  standardHeaders: true,
  legacyHeaders: false,
  handler: (req, res, next, options) => {
    throw new AppError(options.message.message, 429);
  },
});

// Stricter limiter for auth routes
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 attempts
  message: {
    status: 'error',
    message: 'Too many login attempts, please try again after 15 minutes.',
  },
  standardHeaders: true,
  legacyHeaders: false,
  skipSuccessfulRequests: true, // Only count failed requests
});

// Limiter for account creation
const createAccountLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 5, // 5 accounts per hour per IP
  message: {
    status: 'error',
    message: 'Too many accounts created from this IP, please try again after an hour.',
  },
});

// Custom limiter factory
const createLimiter = (options) => {
  return rateLimit({
    windowMs: options.windowMs || 15 * 60 * 1000,
    max: options.max || 100,
    message: {
      status: 'error',
      message: options.message || 'Too many requests.',
    },
    standardHeaders: true,
    legacyHeaders: false,
    ...options,
  });
};

module.exports = {
  apiLimiter,
  authLimiter,
  createAccountLimiter,
  createLimiter,
};
```

---

## Step 5: Health Checks

### Health Controller

```javascript
// src/modules/health/health.controller.js
const mongoose = require('mongoose');
const os = require('os');

const getHealth = async (req, res) => {
  const healthcheck = {
    status: 'ok',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    environment: process.env.NODE_ENV,
  };

  res.json(healthcheck);
};

const getReadiness = async (req, res) => {
  const checks = {
    database: await checkDatabase(),
  };

  const isReady = Object.values(checks).every(check => check.status === 'ok');

  res.status(isReady ? 200 : 503).json({
    status: isReady ? 'ready' : 'not ready',
    timestamp: new Date().toISOString(),
    checks,
  });
};

const getLiveness = async (req, res) => {
  res.json({
    status: 'alive',
    timestamp: new Date().toISOString(),
    pid: process.pid,
  });
};

const getMetrics = async (req, res) => {
  const metrics = {
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    memory: {
      used: process.memoryUsage(),
      total: os.totalmem(),
      free: os.freemem(),
    },
    cpu: {
      loadAverage: os.loadavg(),
      cores: os.cpus().length,
    },
    process: {
      pid: process.pid,
      version: process.version,
      platform: process.platform,
    },
  };

  res.json(metrics);
};

// Helper function to check database connection
async function checkDatabase() {
  try {
    const state = mongoose.connection.readyState;
    const states = {
      0: 'disconnected',
      1: 'connected',
      2: 'connecting',
      3: 'disconnecting',
    };

    if (state === 1) {
      // Ping the database
      await mongoose.connection.db.admin().ping();
      return { status: 'ok', state: states[state] };
    }

    return { status: 'error', state: states[state] };
  } catch (error) {
    return { status: 'error', message: error.message };
  }
}

module.exports = {
  getHealth,
  getReadiness,
  getLiveness,
  getMetrics,
};
```

### Health Routes

```javascript
// src/modules/health/health.routes.js
const express = require('express');
const router = express.Router();
const controller = require('./health.controller');

// Basic health check
router.get('/', controller.getHealth);

// Kubernetes-style probes
router.get('/ready', controller.getReadiness);
router.get('/live', controller.getLiveness);

// Detailed metrics (protect in production)
router.get('/metrics', controller.getMetrics);

module.exports = router;
```

---

## Step 6: API Documentation with Swagger

### Swagger Configuration

```bash
npm install swagger-jsdoc swagger-ui-express
```

```javascript
// src/config/swagger.js
const swaggerJsdoc = require('swagger-jsdoc');
const config = require('./index');

const options = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'Blog API',
      version: '1.0.0',
      description: 'A production-ready Blog API built with Node.js and Express',
      contact: {
        name: 'API Support',
        email: 'support@example.com',
      },
      license: {
        name: 'MIT',
        url: 'https://opensource.org/licenses/MIT',
      },
    },
    servers: [
      {
        url: `http://localhost:${config.port}`,
        description: 'Development server',
      },
      {
        url: 'https://api.yourdomain.com',
        description: 'Production server',
      },
    ],
    components: {
      securitySchemes: {
        bearerAuth: {
          type: 'http',
          scheme: 'bearer',
          bearerFormat: 'JWT',
        },
      },
      schemas: {
        User: {
          type: 'object',
          properties: {
            id: { type: 'string', example: '507f1f77bcf86cd799439011' },
            name: { type: 'string', example: 'John Doe' },
            email: { type: 'string', format: 'email', example: 'john@example.com' },
            role: { type: 'string', enum: ['user', 'admin'], example: 'user' },
            bio: { type: 'string', example: 'Software developer' },
            createdAt: { type: 'string', format: 'date-time' },
          },
        },
        Post: {
          type: 'object',
          properties: {
            id: { type: 'string', example: '507f1f77bcf86cd799439011' },
            title: { type: 'string', example: 'My First Post' },
            slug: { type: 'string', example: 'my-first-post' },
            content: { type: 'string', example: 'Post content here...' },
            excerpt: { type: 'string', example: 'Short excerpt...' },
            author: { $ref: '#/components/schemas/User' },
            status: { type: 'string', enum: ['draft', 'published'], example: 'published' },
            createdAt: { type: 'string', format: 'date-time' },
          },
        },
        Error: {
          type: 'object',
          properties: {
            status: { type: 'string', example: 'error' },
            message: { type: 'string', example: 'Error message' },
          },
        },
        ValidationError: {
          type: 'object',
          properties: {
            status: { type: 'string', example: 'error' },
            message: { type: 'string', example: 'Validation failed' },
            errors: {
              type: 'array',
              items: {
                type: 'object',
                properties: {
                  field: { type: 'string' },
                  message: { type: 'string' },
                },
              },
            },
          },
        },
      },
      responses: {
        NotFound: {
          description: 'Resource not found',
          content: {
            'application/json': {
              schema: { $ref: '#/components/schemas/Error' },
            },
          },
        },
        Unauthorized: {
          description: 'Authentication required',
          content: {
            'application/json': {
              schema: { $ref: '#/components/schemas/Error' },
            },
          },
        },
        ValidationError: {
          description: 'Validation error',
          content: {
            'application/json': {
              schema: { $ref: '#/components/schemas/ValidationError' },
            },
          },
        },
      },
    },
    tags: [
      { name: 'Auth', description: 'Authentication endpoints' },
      { name: 'Users', description: 'User management' },
      { name: 'Posts', description: 'Blog posts' },
      { name: 'Health', description: 'Health check endpoints' },
    ],
  },
  apis: ['./src/modules/**/*.routes.js', './src/modules/**/*.controller.js'],
};

const swaggerSpec = swaggerJsdoc(options);

module.exports = swaggerSpec;
```

### Add Swagger Annotations to Routes

```javascript
// src/modules/auth/auth.routes.js
const express = require('express');
const router = express.Router();
const controller = require('./auth.controller');
const { createUser } = require('../users/user.validation');
const validate = require('../../middleware/validate');
const { authLimiter, createAccountLimiter } = require('../../middleware/rateLimiter');

/**
 * @swagger
 * /api/auth/register:
 *   post:
 *     summary: Register a new user
 *     tags: [Auth]
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             required:
 *               - name
 *               - email
 *               - password
 *             properties:
 *               name:
 *                 type: string
 *                 minLength: 2
 *                 maxLength: 50
 *                 example: John Doe
 *               email:
 *                 type: string
 *                 format: email
 *                 example: john@example.com
 *               password:
 *                 type: string
 *                 minLength: 8
 *                 example: password123
 *     responses:
 *       201:
 *         description: User registered successfully
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 status:
 *                   type: string
 *                   example: success
 *                 data:
 *                   type: object
 *                   properties:
 *                     token:
 *                       type: string
 *                     user:
 *                       $ref: '#/components/schemas/User'
 *       400:
 *         $ref: '#/components/responses/ValidationError'
 *       409:
 *         description: Email already exists
 */
router.post('/register', createAccountLimiter, validate(createUser), controller.register);

/**
 * @swagger
 * /api/auth/login:
 *   post:
 *     summary: Login user
 *     tags: [Auth]
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             required:
 *               - email
 *               - password
 *             properties:
 *               email:
 *                 type: string
 *                 format: email
 *               password:
 *                 type: string
 *     responses:
 *       200:
 *         description: Login successful
 *       401:
 *         description: Invalid credentials
 */
router.post('/login', authLimiter, controller.login);

module.exports = router;
```

```javascript
// src/modules/posts/post.routes.js
const express = require('express');
const router = express.Router();
const controller = require('./post.controller');
const { protect } = require('../../middleware/auth');

/**
 * @swagger
 * /api/posts:
 *   get:
 *     summary: Get all published posts
 *     tags: [Posts]
 *     parameters:
 *       - in: query
 *         name: page
 *         schema:
 *           type: integer
 *           default: 1
 *         description: Page number
 *       - in: query
 *         name: limit
 *         schema:
 *           type: integer
 *           default: 10
 *         description: Items per page
 *       - in: query
 *         name: sort
 *         schema:
 *           type: string
 *           default: -createdAt
 *         description: Sort field (prefix with - for descending)
 *       - in: query
 *         name: search
 *         schema:
 *           type: string
 *         description: Search in title and content
 *     responses:
 *       200:
 *         description: List of posts
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 status:
 *                   type: string
 *                 results:
 *                   type: integer
 *                 pagination:
 *                   type: object
 *                   properties:
 *                     page:
 *                       type: integer
 *                     limit:
 *                       type: integer
 *                     total:
 *                       type: integer
 *                     pages:
 *                       type: integer
 *                 data:
 *                   type: array
 *                   items:
 *                     $ref: '#/components/schemas/Post'
 */
router.get('/', controller.getAll);

/**
 * @swagger
 * /api/posts:
 *   post:
 *     summary: Create a new post
 *     tags: [Posts]
 *     security:
 *       - bearerAuth: []
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             required:
 *               - title
 *               - content
 *             properties:
 *               title:
 *                 type: string
 *                 maxLength: 200
 *               content:
 *                 type: string
 *               excerpt:
 *                 type: string
 *                 maxLength: 500
 *               tags:
 *                 type: array
 *                 items:
 *                   type: string
 *     responses:
 *       201:
 *         description: Post created
 *       401:
 *         $ref: '#/components/responses/Unauthorized'
 */
router.post('/', protect, controller.create);

module.exports = router;
```

---

## Step 7: Graceful Shutdown

```javascript
// src/server.js
const app = require('./app');
const config = require('./config');
const connectDB = require('./config/database');
const logger = require('./config/logger');

// Track server and connections
let server;
const connections = new Set();

// Handle uncaught exceptions
process.on('uncaughtException', (err) => {
  logger.error('UNCAUGHT EXCEPTION! Shutting down...', { error: err.message, stack: err.stack });
  process.exit(1);
});

// Connect to database and start server
connectDB().then(() => {
  server = app.listen(config.port, () => {
    logger.info(`Server running on port ${config.port} in ${config.env} mode`);
  });

  // Track connections for graceful shutdown
  server.on('connection', (connection) => {
    connections.add(connection);
    connection.on('close', () => {
      connections.delete(connection);
    });
  });
});

// Handle unhandled promise rejections
process.on('unhandledRejection', (err) => {
  logger.error('UNHANDLED REJECTION! Shutting down...', { error: err.message, stack: err.stack });
  gracefulShutdown('UNHANDLED REJECTION');
});

// Handle SIGTERM (Kubernetes, Docker, etc.)
process.on('SIGTERM', () => {
  logger.info('SIGTERM received. Performing graceful shutdown...');
  gracefulShutdown('SIGTERM');
});

// Handle SIGINT (Ctrl+C)
process.on('SIGINT', () => {
  logger.info('SIGINT received. Performing graceful shutdown...');
  gracefulShutdown('SIGINT');
});

// Graceful shutdown function
async function gracefulShutdown(signal) {
  logger.info(`${signal} signal received. Starting graceful shutdown...`);

  // Stop accepting new connections
  if (server) {
    server.close(() => {
      logger.info('HTTP server closed');
    });
  }

  // Set a timeout for forceful shutdown
  const shutdownTimeout = setTimeout(() => {
    logger.error('Could not close connections in time, forcefully shutting down');
    process.exit(1);
  }, 30000); // 30 seconds

  // Close existing connections
  for (const connection of connections) {
    connection.end();
  }

  // Close database connection
  try {
    const mongoose = require('mongoose');
    await mongoose.connection.close();
    logger.info('MongoDB connection closed');
  } catch (err) {
    logger.error('Error closing MongoDB connection', { error: err.message });
  }

  clearTimeout(shutdownTimeout);
  logger.info('Graceful shutdown completed');
  process.exit(0);
}

module.exports = server;
```

---

## Step 8: Complete App Configuration

```javascript
// src/app.js
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const mongoSanitize = require('express-mongo-sanitize');
const hpp = require('hpp');
const compression = require('compression');
const swaggerUi = require('swagger-ui-express');

const config = require('./config');
const logger = require('./config/logger');
const swaggerSpec = require('./config/swagger');

// Routes
const authRoutes = require('./modules/auth/auth.routes');
const userRoutes = require('./modules/users/user.routes');
const postRoutes = require('./modules/posts/post.routes');
const healthRoutes = require('./modules/health/health.routes');

// Middleware
const { apiLimiter } = require('./middleware/rateLimiter');
const requestLogger = require('./middleware/requestLogger');
const errorHandler = require('./middleware/errorHandler');
const AppError = require('./utils/AppError');

const app = express();

// Trust proxy (for rate limiting behind reverse proxy)
app.set('trust proxy', 1);

// Security middleware
app.use(helmet({
  contentSecurityPolicy: config.env === 'production',
  crossOriginEmbedderPolicy: config.env === 'production',
}));

// CORS
app.use(cors(config.cors));

// Request logging
app.use(requestLogger);

// Body parsing
app.use(express.json({ limit: '10kb' }));
app.use(express.urlencoded({ extended: true, limit: '10kb' }));

// Data sanitization against NoSQL injection
app.use(mongoSanitize());

// Prevent parameter pollution
app.use(hpp({
  whitelist: ['sort', 'fields', 'page', 'limit'],
}));

// Compression
app.use(compression());

// Rate limiting (apply to API routes)
app.use('/api', apiLimiter);

// API Documentation
if (config.env !== 'production') {
  app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec, {
    explorer: true,
    customCss: '.swagger-ui .topbar { display: none }',
    customSiteTitle: 'Blog API Documentation',
  }));

  // Serve OpenAPI spec as JSON
  app.get('/api-docs.json', (req, res) => {
    res.setHeader('Content-Type', 'application/json');
    res.send(swaggerSpec);
  });
}

// Health check routes (no rate limiting)
app.use('/health', healthRoutes);

// API routes
const apiPrefix = config.api.prefix;
app.use(`${apiPrefix}/auth`, authRoutes);
app.use(`${apiPrefix}/users`, userRoutes);
app.use(`${apiPrefix}/posts`, postRoutes);

// Root endpoint
app.get('/', (req, res) => {
  res.json({
    name: 'Blog API',
    version: '1.0.0',
    documentation: '/api-docs',
    health: '/health',
  });
});

// 404 handler
app.use((req, res, next) => {
  next(new AppError(`Cannot ${req.method} ${req.originalUrl}`, 404));
});

// Global error handler
app.use(errorHandler);

module.exports = app;
```

---

## Step 9: Docker Configuration

### Dockerfile

```dockerfile
# Dockerfile
FROM node:20-alpine AS base

# Install dependencies only when needed
FROM base AS deps
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Development image
FROM base AS development
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "run", "dev"]

# Production image
FROM base AS production
WORKDIR /app

# Set environment
ENV NODE_ENV=production

# Create non-root user
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 expressjs

# Copy dependencies from deps stage
COPY --from=deps /app/node_modules ./node_modules

# Copy source code
COPY . .

# Create logs directory
RUN mkdir -p logs && chown -R expressjs:nodejs logs

# Switch to non-root user
USER expressjs

EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

CMD ["node", "src/server.js"]
```

### Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  api:
    build:
      context: .
      target: production
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - PORT=3000
      - MONGODB_URI=mongodb://mongo:27017/blog-api
      - JWT_SECRET=${JWT_SECRET}
    depends_on:
      mongo:
        condition: service_healthy
    restart: unless-stopped
    networks:
      - app-network

  mongo:
    image: mongo:7
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db
    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped
    networks:
      - app-network

volumes:
  mongo-data:

networks:
  app-network:
    driver: bridge
```

### Development Docker Compose

```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  api:
    build:
      context: .
      target: development
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - PORT=3000
      - MONGODB_URI=mongodb://mongo:27017/blog-api-dev
      - JWT_SECRET=dev-secret-key-at-least-32-chars
    volumes:
      - .:/app
      - /app/node_modules
    depends_on:
      - mongo
    networks:
      - dev-network

  mongo:
    image: mongo:7
    ports:
      - "27017:27017"
    volumes:
      - mongo-dev-data:/data/db
    networks:
      - dev-network

volumes:
  mongo-dev-data:

networks:
  dev-network:
    driver: bridge
```

---

## Step 10: Package.json Scripts

```json
{
  "name": "blog-api",
  "version": "1.0.0",
  "description": "Production-ready Blog API",
  "main": "src/server.js",
  "scripts": {
    "start": "node src/server.js",
    "dev": "nodemon src/server.js",
    "lint": "eslint src/",
    "lint:fix": "eslint src/ --fix",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "docker:dev": "docker-compose -f docker-compose.dev.yml up",
    "docker:dev:build": "docker-compose -f docker-compose.dev.yml up --build",
    "docker:prod": "docker-compose up -d",
    "docker:prod:build": "docker-compose up -d --build",
    "docker:down": "docker-compose down"
  },
  "dependencies": {
    "bcryptjs": "^2.4.3",
    "compression": "^1.7.4",
    "cors": "^2.8.5",
    "dotenv": "^16.3.1",
    "express": "^4.18.2",
    "express-mongo-sanitize": "^2.2.0",
    "express-rate-limit": "^7.1.5",
    "express-validator": "^7.0.1",
    "helmet": "^7.1.0",
    "hpp": "^0.2.3",
    "jsonwebtoken": "^9.0.2",
    "mongoose": "^8.0.3",
    "morgan": "^1.10.0",
    "slugify": "^1.6.6",
    "swagger-jsdoc": "^6.2.8",
    "swagger-ui-express": "^5.0.0",
    "winston": "^3.11.0"
  },
  "devDependencies": {
    "eslint": "^8.56.0",
    "jest": "^29.7.0",
    "nodemon": "^3.0.2",
    "supertest": "^6.3.3"
  },
  "engines": {
    "node": ">=18.0.0"
  }
}
```

---

## Step 11: Production Deployment Checklist

### Pre-Deployment

```markdown
## Environment
- [ ] All environment variables are set in production
- [ ] JWT_SECRET is at least 32 characters and securely generated
- [ ] MONGODB_URI points to production database
- [ ] NODE_ENV is set to 'production'
- [ ] CORS_ORIGIN is set to your frontend domain

## Security
- [ ] Rate limiting is configured appropriately
- [ ] All secrets are stored in environment variables
- [ ] HTTPS is enforced (via reverse proxy)
- [ ] Security headers are enabled (Helmet)
- [ ] Input validation is in place
- [ ] MongoDB sanitization is enabled

## Performance
- [ ] Compression is enabled
- [ ] Database indexes are created
- [ ] Connection pooling is configured
- [ ] Response caching headers are set

## Monitoring
- [ ] Health check endpoints are working
- [ ] Logging is configured for production
- [ ] Error tracking service is integrated (optional)
- [ ] Metrics collection is set up (optional)

## Database
- [ ] MongoDB replica set is configured (recommended)
- [ ] Database backups are scheduled
- [ ] Connection string uses SSL

## Docker
- [ ] Docker image builds successfully
- [ ] Health checks are passing
- [ ] Non-root user is configured
- [ ] Log volumes are mounted
```

### Deployment Commands

```bash
# Build and deploy with Docker
docker-compose up -d --build

# View logs
docker-compose logs -f api

# Scale horizontally
docker-compose up -d --scale api=3

# Health check
curl http://localhost:3000/health
curl http://localhost:3000/health/ready
curl http://localhost:3000/health/live
```

---

## Complete Testing Guide

```bash
# Start the server
npm run dev

# Test health endpoints
curl http://localhost:3000/health
curl http://localhost:3000/health/ready
curl http://localhost:3000/health/live
curl http://localhost:3000/health/metrics

# Test API documentation
open http://localhost:3000/api-docs

# Register a user
curl -X POST http://localhost:3000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"name":"John","email":"john@example.com","password":"password123"}'

# Login
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"john@example.com","password":"password123"}'

# Create a post (use token from login)
curl -X POST http://localhost:3000/api/posts \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{"title":"My Post","content":"Content here"}'

# Get posts with pagination
curl "http://localhost:3000/api/posts?page=1&limit=10"

# Test rate limiting (run many times quickly)
for i in {1..20}; do curl http://localhost:3000/api/posts; done
```

---

## Key Takeaways

1. **Security is paramount** - Use Helmet, rate limiting, input sanitization, and CORS
2. **Logging matters** - Structured logging with Winston helps debugging in production
3. **Health checks enable reliability** - Kubernetes and load balancers need them
4. **Documentation is essential** - Swagger/OpenAPI makes APIs usable
5. **Graceful shutdown prevents data loss** - Handle SIGTERM and SIGINT properly
6. **Environment configuration** - Keep secrets out of code
7. **Docker simplifies deployment** - Consistent environments across dev and production

---

## What's Next?

Congratulations on completing the Node.js & Express module! You've built a production-ready API from scratch.

**Next up: Databases** - We'll dive deep into:
- SQL fundamentals
- Prisma ORM
- MongoDB basics
- Redis caching
- Database design patterns

You'll learn how to properly persist data and build robust database-backed applications!
