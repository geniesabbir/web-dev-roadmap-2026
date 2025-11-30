# Day 1: Backend Testing Setup - Jest & Testing Environment

## Introduction

Testing is essential for building reliable backend applications. A well-tested API gives you confidence that your code works correctly and allows you to refactor without fear of breaking things. Today, you'll set up a comprehensive testing environment for your Node.js/Express application.

## Learning Objectives

By the end of this lesson, you will be able to:
- Set up Jest for backend testing
- Configure test environments and databases
- Structure tests for Express applications
- Use Supertest for HTTP testing
- Understand testing best practices

---

## Why Test Your Backend?

### Benefits of Testing

```
Without Tests:
- Fear of refactoring
- Manual testing after every change
- Bugs discovered in production
- Difficult onboarding for new developers

With Tests:
- Confidence to refactor
- Automated verification
- Bugs caught early
- Living documentation of behavior
```

### Types of Backend Tests

| Type | Scope | Speed | Purpose |
|------|-------|-------|---------|
| Unit | Single function | Fast | Logic correctness |
| Integration | Multiple components | Medium | Component interaction |
| E2E | Entire system | Slow | User flows |

---

## Setting Up Jest

### Installation

```bash
npm install -D jest @types/jest ts-jest
```

### Jest Configuration

```javascript
// jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/src', '<rootDir>/tests'],
  testMatch: [
    '**/__tests__/**/*.ts',
    '**/*.test.ts',
    '**/*.spec.ts'
  ],
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1'
  },
  setupFilesAfterEnv: ['<rootDir>/tests/setup.ts'],
  coverageDirectory: 'coverage',
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.d.ts',
    '!src/index.ts'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  },
  verbose: true,
  testTimeout: 10000
};
```

### Package.json Scripts

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:ci": "jest --ci --coverage --runInBand"
  }
}
```

---

## Test Environment Setup

### Environment Configuration

```javascript
// tests/setup.ts
import dotenv from 'dotenv';

// Load test environment variables
dotenv.config({ path: '.env.test' });

// Global test setup
beforeAll(async () => {
  // Setup code that runs once before all tests
  console.log('🧪 Starting test suite');
});

afterAll(async () => {
  // Cleanup code that runs once after all tests
  console.log('✅ Test suite complete');
});

// Reset mocks between tests
beforeEach(() => {
  jest.clearAllMocks();
});
```

### Test Environment Variables

```env
# .env.test
NODE_ENV=test
PORT=3001
DATABASE_URL="postgresql://user:password@localhost:5432/myapp_test"
JWT_SECRET=test-jwt-secret-key
REDIS_URL="redis://localhost:6379/1"
```

---

## Test Database Setup

### Prisma Test Configuration

```javascript
// tests/helpers/prisma.ts
import { PrismaClient } from '@prisma/client';
import { execSync } from 'child_process';

const prisma = new PrismaClient();

export async function setupTestDatabase() {
  // Reset database
  execSync('npx prisma migrate reset --force --skip-seed', {
    env: { ...process.env, DATABASE_URL: process.env.DATABASE_URL }
  });
}

export async function cleanupTestDatabase() {
  // Delete all data in correct order (respect foreign keys)
  const tablenames = await prisma.$queryRaw<Array<{ tablename: string }>>`
    SELECT tablename FROM pg_tables WHERE schemaname='public'
  `;

  for (const { tablename } of tablenames) {
    if (tablename !== '_prisma_migrations') {
      await prisma.$executeRawUnsafe(
        `TRUNCATE TABLE "public"."${tablename}" CASCADE;`
      );
    }
  }
}

export async function disconnectDatabase() {
  await prisma.$disconnect();
}

export { prisma };
```

### Database Setup in Tests

```javascript
// tests/setup.ts
import {
  setupTestDatabase,
  cleanupTestDatabase,
  disconnectDatabase
} from './helpers/prisma';

beforeAll(async () => {
  await setupTestDatabase();
});

afterEach(async () => {
  await cleanupTestDatabase();
});

afterAll(async () => {
  await disconnectDatabase();
});
```

---

## Supertest for HTTP Testing

### Installation

```bash
npm install -D supertest @types/supertest
```

### App Export for Testing

```javascript
// src/app.ts
import express from 'express';
import cors from 'cors';
import helmet from 'helmet';
import routes from './routes';
import { errorHandler } from './middleware/errorHandler';

const app = express();

app.use(helmet());
app.use(cors());
app.use(express.json());

app.use('/api', routes);
app.use(errorHandler);

// Export app separately from server start
export default app;
```

```javascript
// src/index.ts
import app from './app';

const PORT = process.env.PORT || 3000;

// Only start server if not in test mode
if (process.env.NODE_ENV !== 'test') {
  app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
  });
}

export default app;
```

### Basic Supertest Usage

```javascript
// tests/routes/health.test.ts
import request from 'supertest';
import app from '../../src/app';

describe('Health Check', () => {
  it('should return 200 OK', async () => {
    const response = await request(app)
      .get('/api/health')
      .expect(200);

    expect(response.body).toEqual({
      status: 'ok',
      timestamp: expect.any(String)
    });
  });
});
```

---

## Test Utilities

### Request Helper

```javascript
// tests/helpers/request.ts
import request from 'supertest';
import app from '../../src/app';

export const api = request(app);

// Helper for authenticated requests
export function authenticatedRequest(token: string) {
  return {
    get: (url: string) =>
      request(app).get(url).set('Authorization', `Bearer ${token}`),
    post: (url: string) =>
      request(app).post(url).set('Authorization', `Bearer ${token}`),
    put: (url: string) =>
      request(app).put(url).set('Authorization', `Bearer ${token}`),
    patch: (url: string) =>
      request(app).patch(url).set('Authorization', `Bearer ${token}`),
    delete: (url: string) =>
      request(app).delete(url).set('Authorization', `Bearer ${token}`)
  };
}
```

### Factory Functions

```javascript
// tests/helpers/factories.ts
import { prisma } from './prisma';
import bcrypt from 'bcrypt';
import jwt from 'jsonwebtoken';

export async function createUser(overrides = {}) {
  const defaultUser = {
    email: `test-${Date.now()}@example.com`,
    name: 'Test User',
    password: await bcrypt.hash('password123', 10),
    role: 'USER'
  };

  return prisma.user.create({
    data: { ...defaultUser, ...overrides }
  });
}

export async function createAuthenticatedUser(overrides = {}) {
  const user = await createUser(overrides);

  const token = jwt.sign(
    { userId: user.id, role: user.role },
    process.env.JWT_SECRET!,
    { expiresIn: '1h' }
  );

  return { user, token };
}

export async function createPost(authorId: string, overrides = {}) {
  const defaultPost = {
    title: 'Test Post',
    content: 'Test content',
    published: true,
    authorId
  };

  return prisma.post.create({
    data: { ...defaultPost, ...overrides }
  });
}

export async function createProduct(overrides = {}) {
  const defaultProduct = {
    name: 'Test Product',
    price: 99.99,
    description: 'Test description',
    inStock: true
  };

  return prisma.product.create({
    data: { ...defaultProduct, ...overrides }
  });
}
```

### Test Data Builders

```javascript
// tests/helpers/builders.ts

class UserBuilder {
  private data: any = {
    email: 'test@example.com',
    name: 'Test User',
    password: 'password123',
    role: 'USER'
  };

  withEmail(email: string) {
    this.data.email = email;
    return this;
  }

  withName(name: string) {
    this.data.name = name;
    return this;
  }

  withRole(role: string) {
    this.data.role = role;
    return this;
  }

  asAdmin() {
    this.data.role = 'ADMIN';
    return this;
  }

  build() {
    return { ...this.data };
  }

  async create() {
    const hashedPassword = await bcrypt.hash(this.data.password, 10);
    return prisma.user.create({
      data: { ...this.data, password: hashedPassword }
    });
  }
}

export const userBuilder = () => new UserBuilder();

// Usage
const user = await userBuilder()
  .withEmail('admin@example.com')
  .asAdmin()
  .create();
```

---

## Project Structure

```
project/
├── src/
│   ├── controllers/
│   ├── middleware/
│   ├── routes/
│   ├── services/
│   ├── app.ts
│   └── index.ts
├── tests/
│   ├── __mocks__/           # Manual mocks
│   │   └── prisma.ts
│   ├── helpers/             # Test utilities
│   │   ├── factories.ts
│   │   ├── prisma.ts
│   │   └── request.ts
│   ├── unit/                # Unit tests
│   │   ├── services/
│   │   └── utils/
│   ├── integration/         # Integration tests
│   │   ├── auth.test.ts
│   │   ├── users.test.ts
│   │   └── products.test.ts
│   └── setup.ts             # Global setup
├── jest.config.js
├── .env.test
└── package.json
```

---

## Writing Your First Test

### Simple Unit Test

```javascript
// tests/unit/utils/formatters.test.ts
import { formatCurrency, formatDate } from '../../../src/utils/formatters';

describe('formatCurrency', () => {
  it('should format number as USD currency', () => {
    expect(formatCurrency(99.99)).toBe('$99.99');
  });

  it('should handle zero', () => {
    expect(formatCurrency(0)).toBe('$0.00');
  });

  it('should round to 2 decimal places', () => {
    expect(formatCurrency(99.999)).toBe('$100.00');
  });
});

describe('formatDate', () => {
  it('should format ISO date string', () => {
    const date = '2024-01-15T10:30:00Z';
    expect(formatDate(date)).toBe('January 15, 2024');
  });
});
```

### API Endpoint Test

```javascript
// tests/integration/health.test.ts
import request from 'supertest';
import app from '../../src/app';

describe('GET /api/health', () => {
  it('should return health status', async () => {
    const response = await request(app)
      .get('/api/health')
      .expect('Content-Type', /json/)
      .expect(200);

    expect(response.body).toMatchObject({
      status: 'ok'
    });
  });
});
```

### Authentication Test

```javascript
// tests/integration/auth.test.ts
import request from 'supertest';
import app from '../../src/app';
import { createUser } from '../helpers/factories';

describe('Auth Endpoints', () => {
  describe('POST /api/auth/register', () => {
    it('should register a new user', async () => {
      const response = await request(app)
        .post('/api/auth/register')
        .send({
          email: 'newuser@example.com',
          password: 'Password123!',
          name: 'New User'
        })
        .expect(201);

      expect(response.body).toMatchObject({
        user: {
          email: 'newuser@example.com',
          name: 'New User'
        },
        token: expect.any(String)
      });
      expect(response.body.user).not.toHaveProperty('password');
    });

    it('should return 409 for duplicate email', async () => {
      await createUser({ email: 'existing@example.com' });

      const response = await request(app)
        .post('/api/auth/register')
        .send({
          email: 'existing@example.com',
          password: 'Password123!',
          name: 'New User'
        })
        .expect(409);

      expect(response.body.error).toContain('already exists');
    });

    it('should return 400 for invalid email', async () => {
      const response = await request(app)
        .post('/api/auth/register')
        .send({
          email: 'invalid-email',
          password: 'Password123!',
          name: 'New User'
        })
        .expect(400);

      expect(response.body.errors).toEqual(
        expect.arrayContaining([
          expect.objectContaining({ field: 'email' })
        ])
      );
    });
  });

  describe('POST /api/auth/login', () => {
    beforeEach(async () => {
      await createUser({
        email: 'test@example.com',
        password: await bcrypt.hash('Password123!', 10)
      });
    });

    it('should login with valid credentials', async () => {
      const response = await request(app)
        .post('/api/auth/login')
        .send({
          email: 'test@example.com',
          password: 'Password123!'
        })
        .expect(200);

      expect(response.body).toHaveProperty('token');
      expect(response.body.user.email).toBe('test@example.com');
    });

    it('should return 401 for invalid password', async () => {
      await request(app)
        .post('/api/auth/login')
        .send({
          email: 'test@example.com',
          password: 'wrongpassword'
        })
        .expect(401);
    });
  });
});
```

---

## Running Tests

```bash
# Run all tests
npm test

# Run tests in watch mode
npm run test:watch

# Run specific test file
npm test -- tests/integration/auth.test.ts

# Run tests matching pattern
npm test -- --testNamePattern="register"

# Run with coverage
npm run test:coverage

# Run tests for CI
npm run test:ci
```

---

## Jest Matchers Quick Reference

```javascript
// Equality
expect(value).toBe(expected);           // Strict equality
expect(value).toEqual(expected);        // Deep equality
expect(value).toStrictEqual(expected);  // Deep + type

// Truthiness
expect(value).toBeTruthy();
expect(value).toBeFalsy();
expect(value).toBeNull();
expect(value).toBeUndefined();
expect(value).toBeDefined();

// Numbers
expect(value).toBeGreaterThan(3);
expect(value).toBeGreaterThanOrEqual(3);
expect(value).toBeLessThan(5);
expect(value).toBeCloseTo(0.3);

// Strings
expect(value).toMatch(/pattern/);
expect(value).toContain('substring');

// Arrays
expect(array).toContain(item);
expect(array).toHaveLength(3);
expect(array).toContainEqual({ name: 'test' });

// Objects
expect(object).toHaveProperty('key');
expect(object).toHaveProperty('key', 'value');
expect(object).toMatchObject({ key: 'value' });

// Exceptions
expect(() => fn()).toThrow();
expect(() => fn()).toThrow('error message');
expect(() => fn()).toThrow(ErrorClass);

// Async
await expect(promise).resolves.toBe(value);
await expect(promise).rejects.toThrow();

// Snapshots
expect(value).toMatchSnapshot();
expect(value).toMatchInlineSnapshot(`"expected"`);
```

---

## Practice Exercise

Set up testing for your Express application:

1. **Install dependencies**: Jest, Supertest, ts-jest
2. **Create jest.config.js** with proper configuration
3. **Set up test database** connection
4. **Create helper functions**: factories, request helpers
5. **Write first tests**:
   - Health check endpoint
   - User registration
   - User login
6. **Run tests and verify** they pass

---

## Key Takeaways

1. **Separate app from server** - Export app for testing
2. **Use test database** - Never test against production
3. **Clean between tests** - Ensure test isolation
4. **Create helper functions** - DRY test code
5. **Factory functions** - Quickly create test data
6. **Match error cases** - Test unhappy paths too

---

## What's Next?

Tomorrow, we'll dive deep into **Unit Testing** - testing individual functions and services in isolation with mocks.
