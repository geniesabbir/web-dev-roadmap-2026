# Day 3: JWT Implementation - Building Authentication in Node.js

## Introduction

Now that you understand how JWTs work, it's time to implement a complete authentication system. Today, you'll build a production-ready JWT authentication system with access tokens, refresh tokens, and protected routes.

## Learning Objectives

By the end of this lesson, you will be able to:
- Set up JWT authentication in Express
- Implement user registration and login endpoints
- Create authentication middleware to protect routes
- Handle token refresh flow
- Implement secure logout functionality

---

## Project Setup

### Install Dependencies

```bash
npm install jsonwebtoken bcrypt express-validator cookie-parser
npm install -D @types/jsonwebtoken @types/bcrypt
```

### Environment Configuration

```env
# .env
NODE_ENV=development
PORT=3000

# JWT Configuration
JWT_ACCESS_SECRET=your-super-secret-access-key-min-32-chars
JWT_REFRESH_SECRET=your-super-secret-refresh-key-min-32-chars
JWT_ACCESS_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d

# Database
DATABASE_URL="postgresql://user:password@localhost:5432/myapp"
```

### Token Configuration Module

```javascript
// config/jwt.js
require('dotenv').config();

module.exports = {
  accessToken: {
    secret: process.env.JWT_ACCESS_SECRET,
    expiresIn: process.env.JWT_ACCESS_EXPIRES_IN || '15m'
  },
  refreshToken: {
    secret: process.env.JWT_REFRESH_SECRET,
    expiresIn: process.env.JWT_REFRESH_EXPIRES_IN || '7d'
  },
  cookie: {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict',
    maxAge: 7 * 24 * 60 * 60 * 1000 // 7 days
  }
};
```

---

## Token Service

### Generate and Verify Tokens

```javascript
// services/tokenService.js
const jwt = require('jsonwebtoken');
const crypto = require('crypto');
const config = require('../config/jwt');
const { prisma } = require('../lib/prisma');

const TokenService = {
  // Generate access token
  generateAccessToken(userId, role) {
    return jwt.sign(
      {
        userId,
        role,
        type: 'access'
      },
      config.accessToken.secret,
      {
        expiresIn: config.accessToken.expiresIn,
        issuer: 'myapp',
        audience: 'myapp-api'
      }
    );
  },

  // Generate refresh token
  generateRefreshToken(userId) {
    return jwt.sign(
      {
        userId,
        type: 'refresh',
        jti: crypto.randomUUID() // Unique token ID for revocation
      },
      config.refreshToken.secret,
      {
        expiresIn: config.refreshToken.expiresIn,
        issuer: 'myapp'
      }
    );
  },

  // Generate both tokens
  generateTokenPair(userId, role) {
    return {
      accessToken: this.generateAccessToken(userId, role),
      refreshToken: this.generateRefreshToken(userId)
    };
  },

  // Verify access token
  verifyAccessToken(token) {
    try {
      return jwt.verify(token, config.accessToken.secret, {
        issuer: 'myapp',
        audience: 'myapp-api',
        algorithms: ['HS256']
      });
    } catch (error) {
      return null;
    }
  },

  // Verify refresh token
  verifyRefreshToken(token) {
    try {
      return jwt.verify(token, config.refreshToken.secret, {
        issuer: 'myapp',
        algorithms: ['HS256']
      });
    } catch (error) {
      return null;
    }
  },

  // Store refresh token in database (for revocation)
  async storeRefreshToken(userId, token, userAgent, ipAddress) {
    const decoded = jwt.decode(token);

    await prisma.refreshToken.create({
      data: {
        userId,
        tokenId: decoded.jti,
        userAgent,
        ipAddress,
        expiresAt: new Date(decoded.exp * 1000)
      }
    });
  },

  // Check if refresh token is valid (not revoked)
  async isRefreshTokenValid(tokenId, userId) {
    const token = await prisma.refreshToken.findFirst({
      where: {
        tokenId,
        userId,
        revokedAt: null,
        expiresAt: { gt: new Date() }
      }
    });

    return !!token;
  },

  // Revoke a specific refresh token
  async revokeRefreshToken(tokenId) {
    await prisma.refreshToken.updateMany({
      where: { tokenId },
      data: { revokedAt: new Date() }
    });
  },

  // Revoke all refresh tokens for a user
  async revokeAllUserTokens(userId) {
    await prisma.refreshToken.updateMany({
      where: { userId, revokedAt: null },
      data: { revokedAt: new Date() }
    });
  },

  // Clean up expired tokens (run periodically)
  async cleanupExpiredTokens() {
    const result = await prisma.refreshToken.deleteMany({
      where: {
        expiresAt: { lt: new Date() }
      }
    });

    return result.count;
  }
};

module.exports = TokenService;
```

---

## Database Schema

### Prisma Schema for Tokens

```prisma
// prisma/schema.prisma
model User {
  id            String         @id @default(uuid())
  email         String         @unique
  password      String
  name          String
  role          Role           @default(USER)
  isActive      Boolean        @default(true)
  createdAt     DateTime       @default(now())
  updatedAt     DateTime       @updatedAt
  refreshTokens RefreshToken[]
}

enum Role {
  USER
  ADMIN
  MODERATOR
}

model RefreshToken {
  id        String    @id @default(uuid())
  tokenId   String    @unique  // JWT jti claim
  userId    String
  userAgent String?
  ipAddress String?
  expiresAt DateTime
  revokedAt DateTime?
  createdAt DateTime  @default(now())
  user      User      @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
  @@index([tokenId])
  @@index([expiresAt])
}
```

---

## Authentication Controller

```javascript
// controllers/authController.js
const bcrypt = require('bcrypt');
const { validationResult } = require('express-validator');
const { prisma } = require('../lib/prisma');
const TokenService = require('../services/tokenService');
const config = require('../config/jwt');

const SALT_ROUNDS = 12;

const authController = {
  // User Registration
  async register(req, res) {
    try {
      const errors = validationResult(req);
      if (!errors.isEmpty()) {
        return res.status(400).json({ errors: errors.array() });
      }

      const { email, password, name } = req.body;

      // Check if user exists
      const existingUser = await prisma.user.findUnique({
        where: { email }
      });

      if (existingUser) {
        return res.status(409).json({
          error: 'User with this email already exists'
        });
      }

      // Hash password
      const hashedPassword = await bcrypt.hash(password, SALT_ROUNDS);

      // Create user
      const user = await prisma.user.create({
        data: {
          email,
          password: hashedPassword,
          name
        },
        select: {
          id: true,
          email: true,
          name: true,
          role: true,
          createdAt: true
        }
      });

      // Generate tokens
      const tokens = TokenService.generateTokenPair(user.id, user.role);

      // Store refresh token
      await TokenService.storeRefreshToken(
        user.id,
        tokens.refreshToken,
        req.headers['user-agent'],
        req.ip
      );

      // Set refresh token as HttpOnly cookie
      res.cookie('refreshToken', tokens.refreshToken, config.cookie);

      res.status(201).json({
        message: 'Registration successful',
        user,
        accessToken: tokens.accessToken
      });
    } catch (error) {
      console.error('Registration error:', error);
      res.status(500).json({ error: 'Registration failed' });
    }
  },

  // User Login
  async login(req, res) {
    try {
      const { email, password } = req.body;

      // Find user
      const user = await prisma.user.findUnique({
        where: { email }
      });

      // Constant-time comparison (prevent timing attacks)
      const isValid = user
        ? await bcrypt.compare(password, user.password)
        : await bcrypt.compare(password, '$2b$12$invalidhash');

      if (!user || !isValid) {
        return res.status(401).json({
          error: 'Invalid email or password'
        });
      }

      // Check if user is active
      if (!user.isActive) {
        return res.status(403).json({
          error: 'Account is deactivated'
        });
      }

      // Generate tokens
      const tokens = TokenService.generateTokenPair(user.id, user.role);

      // Store refresh token
      await TokenService.storeRefreshToken(
        user.id,
        tokens.refreshToken,
        req.headers['user-agent'],
        req.ip
      );

      // Set refresh token as HttpOnly cookie
      res.cookie('refreshToken', tokens.refreshToken, config.cookie);

      res.json({
        message: 'Login successful',
        user: {
          id: user.id,
          email: user.email,
          name: user.name,
          role: user.role
        },
        accessToken: tokens.accessToken
      });
    } catch (error) {
      console.error('Login error:', error);
      res.status(500).json({ error: 'Login failed' });
    }
  },

  // Refresh Access Token
  async refresh(req, res) {
    try {
      const refreshToken = req.cookies.refreshToken;

      if (!refreshToken) {
        return res.status(401).json({
          error: 'Refresh token not found'
        });
      }

      // Verify refresh token
      const decoded = TokenService.verifyRefreshToken(refreshToken);

      if (!decoded || decoded.type !== 'refresh') {
        res.clearCookie('refreshToken');
        return res.status(401).json({
          error: 'Invalid refresh token'
        });
      }

      // Check if token is revoked
      const isValid = await TokenService.isRefreshTokenValid(
        decoded.jti,
        decoded.userId
      );

      if (!isValid) {
        res.clearCookie('refreshToken');
        return res.status(401).json({
          error: 'Refresh token has been revoked'
        });
      }

      // Get user
      const user = await prisma.user.findUnique({
        where: { id: decoded.userId },
        select: { id: true, role: true, isActive: true }
      });

      if (!user || !user.isActive) {
        res.clearCookie('refreshToken');
        return res.status(401).json({
          error: 'User not found or inactive'
        });
      }

      // Generate new access token
      const accessToken = TokenService.generateAccessToken(user.id, user.role);

      res.json({ accessToken });
    } catch (error) {
      console.error('Token refresh error:', error);
      res.status(500).json({ error: 'Token refresh failed' });
    }
  },

  // Logout (revoke current refresh token)
  async logout(req, res) {
    try {
      const refreshToken = req.cookies.refreshToken;

      if (refreshToken) {
        const decoded = TokenService.verifyRefreshToken(refreshToken);

        if (decoded) {
          await TokenService.revokeRefreshToken(decoded.jti);
        }
      }

      res.clearCookie('refreshToken');
      res.json({ message: 'Logged out successfully' });
    } catch (error) {
      console.error('Logout error:', error);
      // Still clear cookie even if error
      res.clearCookie('refreshToken');
      res.json({ message: 'Logged out' });
    }
  },

  // Logout from all devices
  async logoutAll(req, res) {
    try {
      await TokenService.revokeAllUserTokens(req.user.userId);
      res.clearCookie('refreshToken');

      res.json({ message: 'Logged out from all devices' });
    } catch (error) {
      console.error('Logout all error:', error);
      res.status(500).json({ error: 'Logout failed' });
    }
  },

  // Get current user
  async me(req, res) {
    try {
      const user = await prisma.user.findUnique({
        where: { id: req.user.userId },
        select: {
          id: true,
          email: true,
          name: true,
          role: true,
          createdAt: true
        }
      });

      if (!user) {
        return res.status(404).json({ error: 'User not found' });
      }

      res.json({ user });
    } catch (error) {
      console.error('Get user error:', error);
      res.status(500).json({ error: 'Failed to get user' });
    }
  },

  // Get active sessions
  async getSessions(req, res) {
    try {
      const sessions = await prisma.refreshToken.findMany({
        where: {
          userId: req.user.userId,
          revokedAt: null,
          expiresAt: { gt: new Date() }
        },
        select: {
          id: true,
          userAgent: true,
          ipAddress: true,
          createdAt: true,
          expiresAt: true
        },
        orderBy: { createdAt: 'desc' }
      });

      res.json({ sessions });
    } catch (error) {
      console.error('Get sessions error:', error);
      res.status(500).json({ error: 'Failed to get sessions' });
    }
  },

  // Revoke specific session
  async revokeSession(req, res) {
    try {
      const { sessionId } = req.params;

      const session = await prisma.refreshToken.findFirst({
        where: {
          id: sessionId,
          userId: req.user.userId
        }
      });

      if (!session) {
        return res.status(404).json({ error: 'Session not found' });
      }

      await prisma.refreshToken.update({
        where: { id: sessionId },
        data: { revokedAt: new Date() }
      });

      res.json({ message: 'Session revoked' });
    } catch (error) {
      console.error('Revoke session error:', error);
      res.status(500).json({ error: 'Failed to revoke session' });
    }
  }
};

module.exports = authController;
```

---

## Authentication Middleware

```javascript
// middleware/auth.js
const TokenService = require('../services/tokenService');

// Authenticate user (required)
const authenticate = (req, res, next) => {
  try {
    const authHeader = req.headers.authorization;

    if (!authHeader || !authHeader.startsWith('Bearer ')) {
      return res.status(401).json({
        error: 'Access token required'
      });
    }

    const token = authHeader.split(' ')[1];
    const decoded = TokenService.verifyAccessToken(token);

    if (!decoded) {
      return res.status(401).json({
        error: 'Invalid or expired access token'
      });
    }

    if (decoded.type !== 'access') {
      return res.status(401).json({
        error: 'Invalid token type'
      });
    }

    // Attach user info to request
    req.user = {
      userId: decoded.userId,
      role: decoded.role
    };

    next();
  } catch (error) {
    console.error('Auth middleware error:', error);
    res.status(401).json({ error: 'Authentication failed' });
  }
};

// Optional authentication (user info if authenticated)
const optionalAuth = (req, res, next) => {
  try {
    const authHeader = req.headers.authorization;

    if (authHeader && authHeader.startsWith('Bearer ')) {
      const token = authHeader.split(' ')[1];
      const decoded = TokenService.verifyAccessToken(token);

      if (decoded && decoded.type === 'access') {
        req.user = {
          userId: decoded.userId,
          role: decoded.role
        };
      }
    }

    next();
  } catch (error) {
    // Continue without authentication
    next();
  }
};

// Role-based authorization
const authorize = (...allowedRoles) => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({
        error: 'Authentication required'
      });
    }

    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({
        error: 'Insufficient permissions'
      });
    }

    next();
  };
};

// Resource ownership check
const authorizeOwner = (getResourceOwnerId) => {
  return async (req, res, next) => {
    try {
      const ownerId = await getResourceOwnerId(req);

      if (ownerId !== req.user.userId && req.user.role !== 'ADMIN') {
        return res.status(403).json({
          error: 'Access denied'
        });
      }

      next();
    } catch (error) {
      console.error('Authorization error:', error);
      res.status(500).json({ error: 'Authorization failed' });
    }
  };
};

module.exports = {
  authenticate,
  optionalAuth,
  authorize,
  authorizeOwner
};
```

---

## Input Validation

```javascript
// middleware/validators.js
const { body, param } = require('express-validator');

const registerValidation = [
  body('email')
    .isEmail()
    .normalizeEmail()
    .withMessage('Valid email is required'),

  body('password')
    .isLength({ min: 8 })
    .withMessage('Password must be at least 8 characters')
    .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/)
    .withMessage('Password must include uppercase, lowercase, number, and special character'),

  body('name')
    .trim()
    .isLength({ min: 2, max: 100 })
    .withMessage('Name must be 2-100 characters')
    .escape()
];

const loginValidation = [
  body('email')
    .isEmail()
    .normalizeEmail()
    .withMessage('Valid email is required'),

  body('password')
    .notEmpty()
    .withMessage('Password is required')
];

const sessionIdValidation = [
  param('sessionId')
    .isUUID()
    .withMessage('Invalid session ID')
];

module.exports = {
  registerValidation,
  loginValidation,
  sessionIdValidation
};
```

---

## Routes Setup

```javascript
// routes/auth.js
const express = require('express');
const router = express.Router();
const authController = require('../controllers/authController');
const { authenticate } = require('../middleware/auth');
const {
  registerValidation,
  loginValidation,
  sessionIdValidation
} = require('../middleware/validators');

// Public routes
router.post('/register', registerValidation, authController.register);
router.post('/login', loginValidation, authController.login);
router.post('/refresh', authController.refresh);
router.post('/logout', authController.logout);

// Protected routes
router.get('/me', authenticate, authController.me);
router.post('/logout-all', authenticate, authController.logoutAll);
router.get('/sessions', authenticate, authController.getSessions);
router.delete(
  '/sessions/:sessionId',
  authenticate,
  sessionIdValidation,
  authController.revokeSession
);

module.exports = router;
```

---

## Express App Setup

```javascript
// app.js
const express = require('express');
const cookieParser = require('cookie-parser');
const cors = require('cors');
const helmet = require('helmet');
const authRoutes = require('./routes/auth');

const app = express();

// Security middleware
app.use(helmet());
app.use(cors({
  origin: process.env.CLIENT_URL || 'http://localhost:3000',
  credentials: true  // Required for cookies
}));

// Body parsing
app.use(express.json());
app.use(cookieParser());

// Trust proxy (for req.ip behind load balancer)
app.set('trust proxy', 1);

// Routes
app.use('/api/auth', authRoutes);

// Error handling
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Something went wrong' });
});

module.exports = app;
```

---

## Protected Route Examples

```javascript
// routes/users.js
const express = require('express');
const router = express.Router();
const { authenticate, authorize, authorizeOwner } = require('../middleware/auth');
const { prisma } = require('../lib/prisma');

// Get all users (admin only)
router.get(
  '/',
  authenticate,
  authorize('ADMIN'),
  async (req, res) => {
    const users = await prisma.user.findMany({
      select: {
        id: true,
        email: true,
        name: true,
        role: true,
        createdAt: true
      }
    });

    res.json({ users });
  }
);

// Get user profile
router.get(
  '/:userId',
  authenticate,
  authorizeOwner(async (req) => req.params.userId),
  async (req, res) => {
    const user = await prisma.user.findUnique({
      where: { id: req.params.userId },
      select: {
        id: true,
        email: true,
        name: true,
        role: true,
        createdAt: true
      }
    });

    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }

    res.json({ user });
  }
);

// Update user profile (owner only)
router.patch(
  '/:userId',
  authenticate,
  authorizeOwner(async (req) => req.params.userId),
  async (req, res) => {
    const { name } = req.body;

    const user = await prisma.user.update({
      where: { id: req.params.userId },
      data: { name },
      select: {
        id: true,
        email: true,
        name: true,
        role: true
      }
    });

    res.json({ user });
  }
);

// Delete user (admin only)
router.delete(
  '/:userId',
  authenticate,
  authorize('ADMIN'),
  async (req, res) => {
    await prisma.user.delete({
      where: { id: req.params.userId }
    });

    res.json({ message: 'User deleted' });
  }
);

module.exports = router;
```

---

## Client-Side Token Management

```javascript
// client/auth.js

// Token storage
let accessToken = null;

// Set access token
export function setAccessToken(token) {
  accessToken = token;
}

// Get access token
export function getAccessToken() {
  return accessToken;
}

// Clear access token
export function clearAccessToken() {
  accessToken = null;
}

// API request with automatic token refresh
export async function authFetch(url, options = {}) {
  // Add authorization header
  const headers = {
    ...options.headers,
    'Authorization': `Bearer ${accessToken}`
  };

  let response = await fetch(url, { ...options, headers });

  // If token expired, try to refresh
  if (response.status === 401) {
    const refreshed = await refreshAccessToken();

    if (refreshed) {
      // Retry with new token
      headers['Authorization'] = `Bearer ${accessToken}`;
      response = await fetch(url, { ...options, headers });
    } else {
      // Refresh failed, redirect to login
      clearAccessToken();
      window.location.href = '/login';
    }
  }

  return response;
}

// Refresh access token
async function refreshAccessToken() {
  try {
    const response = await fetch('/api/auth/refresh', {
      method: 'POST',
      credentials: 'include'  // Include cookies
    });

    if (!response.ok) {
      return false;
    }

    const data = await response.json();
    setAccessToken(data.accessToken);
    return true;
  } catch {
    return false;
  }
}

// Login
export async function login(email, password) {
  const response = await fetch('/api/auth/login', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    credentials: 'include',
    body: JSON.stringify({ email, password })
  });

  if (!response.ok) {
    throw new Error('Login failed');
  }

  const data = await response.json();
  setAccessToken(data.accessToken);
  return data.user;
}

// Logout
export async function logout() {
  await fetch('/api/auth/logout', {
    method: 'POST',
    credentials: 'include'
  });

  clearAccessToken();
}

// Initialize (check for existing session on page load)
export async function initAuth() {
  const refreshed = await refreshAccessToken();
  return refreshed;
}
```

---

## Token Cleanup Job

```javascript
// jobs/tokenCleanup.js
const cron = require('node-cron');
const TokenService = require('../services/tokenService');

// Run every day at 3 AM
cron.schedule('0 3 * * *', async () => {
  try {
    const deletedCount = await TokenService.cleanupExpiredTokens();
    console.log(`Cleaned up ${deletedCount} expired tokens`);
  } catch (error) {
    console.error('Token cleanup failed:', error);
  }
});

console.log('Token cleanup job scheduled');
```

---

## Testing Authentication

```javascript
// tests/auth.test.js
const request = require('supertest');
const app = require('../app');
const { prisma } = require('../lib/prisma');

describe('Authentication', () => {
  beforeEach(async () => {
    await prisma.refreshToken.deleteMany();
    await prisma.user.deleteMany();
  });

  describe('POST /api/auth/register', () => {
    it('should register a new user', async () => {
      const res = await request(app)
        .post('/api/auth/register')
        .send({
          email: 'test@example.com',
          password: 'Password123!',
          name: 'Test User'
        });

      expect(res.status).toBe(201);
      expect(res.body.user.email).toBe('test@example.com');
      expect(res.body.accessToken).toBeDefined();
      expect(res.headers['set-cookie']).toBeDefined();
    });

    it('should reject weak password', async () => {
      const res = await request(app)
        .post('/api/auth/register')
        .send({
          email: 'test@example.com',
          password: 'weak',
          name: 'Test User'
        });

      expect(res.status).toBe(400);
    });
  });

  describe('POST /api/auth/login', () => {
    beforeEach(async () => {
      await request(app)
        .post('/api/auth/register')
        .send({
          email: 'test@example.com',
          password: 'Password123!',
          name: 'Test User'
        });
    });

    it('should login with valid credentials', async () => {
      const res = await request(app)
        .post('/api/auth/login')
        .send({
          email: 'test@example.com',
          password: 'Password123!'
        });

      expect(res.status).toBe(200);
      expect(res.body.accessToken).toBeDefined();
    });

    it('should reject invalid password', async () => {
      const res = await request(app)
        .post('/api/auth/login')
        .send({
          email: 'test@example.com',
          password: 'wrongpassword'
        });

      expect(res.status).toBe(401);
    });
  });

  describe('GET /api/auth/me', () => {
    it('should return user with valid token', async () => {
      const registerRes = await request(app)
        .post('/api/auth/register')
        .send({
          email: 'test@example.com',
          password: 'Password123!',
          name: 'Test User'
        });

      const res = await request(app)
        .get('/api/auth/me')
        .set('Authorization', `Bearer ${registerRes.body.accessToken}`);

      expect(res.status).toBe(200);
      expect(res.body.user.email).toBe('test@example.com');
    });

    it('should reject without token', async () => {
      const res = await request(app).get('/api/auth/me');

      expect(res.status).toBe(401);
    });
  });
});
```

---

## Practice Exercise

Build a complete authentication system:

1. **Set up the project**:
   - Initialize Express with all middleware
   - Configure Prisma with User and RefreshToken models
   - Set up environment variables

2. **Implement all endpoints**:
   - Register, Login, Logout
   - Token refresh
   - Get current user
   - Session management

3. **Add features**:
   - Password change endpoint
   - Email verification (bonus)
   - Account deactivation

4. **Test with Postman/Thunder Client**:
   - Register a user
   - Login and observe tokens
   - Access protected routes
   - Refresh token when expired
   - Logout and verify token is revoked

---

## Key Takeaways

1. **Separate token types** - Access tokens for API, refresh tokens for renewal
2. **Store refresh tokens** - Enable revocation and session management
3. **HttpOnly cookies** - Protect refresh tokens from XSS
4. **Constant-time comparison** - Prevent timing attacks in login
5. **Token cleanup** - Regularly remove expired tokens from database
6. **Generic error messages** - Don't reveal user existence

---

## What's Next?

Tomorrow, we'll implement **OAuth 2.0** authentication with social login providers like Google, GitHub, and others.
