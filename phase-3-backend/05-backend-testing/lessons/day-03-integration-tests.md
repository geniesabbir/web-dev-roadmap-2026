# Day 3: Integration Testing - Testing Components Together

## Introduction

Integration tests verify that multiple components work together correctly. Unlike unit tests that test functions in isolation, integration tests check the interaction between your API routes, controllers, services, and database. Today, you'll learn how to write comprehensive integration tests for your Express API.

## Learning Objectives

By the end of this lesson, you will be able to:
- Write integration tests for API endpoints
- Test database operations end-to-end
- Handle authentication in tests
- Test error responses and edge cases
- Structure integration test suites effectively

---

## Integration vs Unit Tests

```
Unit Tests:
┌──────────────┐
│   Function   │  ← Test in isolation with mocks
└──────────────┘

Integration Tests:
┌──────────┐     ┌────────────┐     ┌──────────┐     ┌──────────┐
│  Route   │ ──► │ Controller │ ──► │ Service  │ ──► │ Database │
└──────────┘     └────────────┘     └──────────┘     └──────────┘
                  ↑
                  └── Test the full flow
```

### When to Use Each

| Aspect | Unit Tests | Integration Tests |
|--------|-----------|-------------------|
| Speed | Fast (ms) | Slower (seconds) |
| Scope | Single function | Multiple components |
| Database | Mocked | Real test database |
| Purpose | Logic correctness | System behavior |
| Quantity | Many | Fewer, focused |

---

## Setting Up Integration Tests

### Test Database Configuration

```javascript
// tests/integration/setup.ts
import { prisma } from '../helpers/prisma';

beforeAll(async () => {
  // Ensure test database is ready
  await prisma.$connect();
});

afterEach(async () => {
  // Clean all tables after each test
  const tableNames = ['Comment', 'Post', 'Session', 'User'];

  for (const table of tableNames) {
    await prisma.$executeRawUnsafe(
      `TRUNCATE TABLE "${table}" CASCADE;`
    );
  }
});

afterAll(async () => {
  await prisma.$disconnect();
});
```

### Test Helpers

```javascript
// tests/helpers/testApp.ts
import request from 'supertest';
import app from '../../src/app';

export const api = request(app);

export async function loginAs(email: string, password: string) {
  const response = await api
    .post('/api/auth/login')
    .send({ email, password });

  return response.body.token;
}

export function authRequest(token: string) {
  return {
    get: (url: string) =>
      api.get(url).set('Authorization', `Bearer ${token}`),
    post: (url: string) =>
      api.post(url).set('Authorization', `Bearer ${token}`),
    put: (url: string) =>
      api.put(url).set('Authorization', `Bearer ${token}`),
    patch: (url: string) =>
      api.patch(url).set('Authorization', `Bearer ${token}`),
    delete: (url: string) =>
      api.delete(url).set('Authorization', `Bearer ${token}`)
  };
}
```

---

## Testing CRUD Endpoints

### Complete User API Tests

```javascript
// tests/integration/users.test.ts
import { api, authRequest } from '../helpers/testApp';
import { createUser, createAuthenticatedUser } from '../helpers/factories';

describe('Users API', () => {
  describe('GET /api/users', () => {
    it('should return empty array when no users', async () => {
      const { token } = await createAuthenticatedUser({ role: 'ADMIN' });

      const response = await authRequest(token)
        .get('/api/users')
        .expect(200);

      expect(response.body.data).toEqual([]);
    });

    it('should return all users for admin', async () => {
      const { token } = await createAuthenticatedUser({ role: 'ADMIN' });
      await createUser({ email: 'user1@example.com' });
      await createUser({ email: 'user2@example.com' });

      const response = await authRequest(token)
        .get('/api/users')
        .expect(200);

      expect(response.body.data).toHaveLength(3); // Including admin
    });

    it('should support pagination', async () => {
      const { token } = await createAuthenticatedUser({ role: 'ADMIN' });

      // Create 15 users
      for (let i = 0; i < 15; i++) {
        await createUser({ email: `user${i}@example.com` });
      }

      const response = await authRequest(token)
        .get('/api/users?page=1&limit=10')
        .expect(200);

      expect(response.body.data).toHaveLength(10);
      expect(response.body.meta).toMatchObject({
        page: 1,
        limit: 10,
        total: 16,
        totalPages: 2
      });
    });

    it('should return 401 without authentication', async () => {
      await api
        .get('/api/users')
        .expect(401);
    });

    it('should return 403 for non-admin users', async () => {
      const { token } = await createAuthenticatedUser({ role: 'USER' });

      await authRequest(token)
        .get('/api/users')
        .expect(403);
    });
  });

  describe('GET /api/users/:id', () => {
    it('should return user by id', async () => {
      const { user, token } = await createAuthenticatedUser();

      const response = await authRequest(token)
        .get(`/api/users/${user.id}`)
        .expect(200);

      expect(response.body.data).toMatchObject({
        id: user.id,
        email: user.email,
        name: user.name
      });
    });

    it('should return 404 for non-existent user', async () => {
      const { token } = await createAuthenticatedUser();

      const response = await authRequest(token)
        .get('/api/users/non-existent-id')
        .expect(404);

      expect(response.body.error).toContain('not found');
    });

    it('should not return password field', async () => {
      const { user, token } = await createAuthenticatedUser();

      const response = await authRequest(token)
        .get(`/api/users/${user.id}`)
        .expect(200);

      expect(response.body.data).not.toHaveProperty('password');
    });
  });

  describe('PATCH /api/users/:id', () => {
    it('should update user name', async () => {
      const { user, token } = await createAuthenticatedUser();

      const response = await authRequest(token)
        .patch(`/api/users/${user.id}`)
        .send({ name: 'Updated Name' })
        .expect(200);

      expect(response.body.data.name).toBe('Updated Name');
    });

    it('should not allow updating other users profile', async () => {
      const otherUser = await createUser();
      const { token } = await createAuthenticatedUser();

      await authRequest(token)
        .patch(`/api/users/${otherUser.id}`)
        .send({ name: 'Hacked' })
        .expect(403);
    });

    it('should allow admin to update any user', async () => {
      const user = await createUser();
      const { token } = await createAuthenticatedUser({ role: 'ADMIN' });

      const response = await authRequest(token)
        .patch(`/api/users/${user.id}`)
        .send({ name: 'Admin Updated' })
        .expect(200);

      expect(response.body.data.name).toBe('Admin Updated');
    });
  });

  describe('DELETE /api/users/:id', () => {
    it('should delete user (admin only)', async () => {
      const user = await createUser();
      const { token } = await createAuthenticatedUser({ role: 'ADMIN' });

      await authRequest(token)
        .delete(`/api/users/${user.id}`)
        .expect(204);

      // Verify deletion
      await authRequest(token)
        .get(`/api/users/${user.id}`)
        .expect(404);
    });

    it('should return 403 for non-admin', async () => {
      const user = await createUser();
      const { token } = await createAuthenticatedUser({ role: 'USER' });

      await authRequest(token)
        .delete(`/api/users/${user.id}`)
        .expect(403);
    });
  });
});
```

---

## Testing Authentication Flow

```javascript
// tests/integration/auth.test.ts
import { api } from '../helpers/testApp';
import { createUser } from '../helpers/factories';
import bcrypt from 'bcrypt';

describe('Authentication', () => {
  describe('POST /api/auth/register', () => {
    const validUser = {
      email: 'newuser@example.com',
      password: 'SecurePass123!',
      name: 'New User'
    };

    it('should register new user and return token', async () => {
      const response = await api
        .post('/api/auth/register')
        .send(validUser)
        .expect(201);

      expect(response.body).toHaveProperty('token');
      expect(response.body.user).toMatchObject({
        email: validUser.email,
        name: validUser.name
      });
      expect(response.body.user).not.toHaveProperty('password');
    });

    it('should set refresh token cookie', async () => {
      const response = await api
        .post('/api/auth/register')
        .send(validUser)
        .expect(201);

      expect(response.headers['set-cookie']).toBeDefined();
      expect(response.headers['set-cookie'][0]).toMatch(/refreshToken=/);
    });

    it('should reject duplicate email', async () => {
      await createUser({ email: validUser.email });

      const response = await api
        .post('/api/auth/register')
        .send(validUser)
        .expect(409);

      expect(response.body.error).toContain('already exists');
    });

    it('should validate email format', async () => {
      const response = await api
        .post('/api/auth/register')
        .send({ ...validUser, email: 'invalid-email' })
        .expect(400);

      expect(response.body.errors).toContainEqual(
        expect.objectContaining({ field: 'email' })
      );
    });

    it('should validate password strength', async () => {
      const response = await api
        .post('/api/auth/register')
        .send({ ...validUser, password: 'weak' })
        .expect(400);

      expect(response.body.errors).toContainEqual(
        expect.objectContaining({ field: 'password' })
      );
    });
  });

  describe('POST /api/auth/login', () => {
    beforeEach(async () => {
      await createUser({
        email: 'existing@example.com',
        password: await bcrypt.hash('CorrectPassword123!', 10)
      });
    });

    it('should login with valid credentials', async () => {
      const response = await api
        .post('/api/auth/login')
        .send({
          email: 'existing@example.com',
          password: 'CorrectPassword123!'
        })
        .expect(200);

      expect(response.body).toHaveProperty('token');
      expect(response.body.user.email).toBe('existing@example.com');
    });

    it('should reject invalid password', async () => {
      const response = await api
        .post('/api/auth/login')
        .send({
          email: 'existing@example.com',
          password: 'WrongPassword'
        })
        .expect(401);

      expect(response.body.error).toContain('Invalid');
    });

    it('should reject non-existent email', async () => {
      const response = await api
        .post('/api/auth/login')
        .send({
          email: 'nonexistent@example.com',
          password: 'AnyPassword123!'
        })
        .expect(401);

      // Same error message to prevent email enumeration
      expect(response.body.error).toContain('Invalid');
    });
  });

  describe('POST /api/auth/refresh', () => {
    it('should refresh access token with valid refresh token', async () => {
      // First, register to get tokens
      const registerResponse = await api
        .post('/api/auth/register')
        .send({
          email: 'refresh@example.com',
          password: 'Password123!',
          name: 'Test'
        });

      const cookies = registerResponse.headers['set-cookie'];

      // Then refresh
      const response = await api
        .post('/api/auth/refresh')
        .set('Cookie', cookies)
        .expect(200);

      expect(response.body).toHaveProperty('accessToken');
    });

    it('should reject without refresh token', async () => {
      await api
        .post('/api/auth/refresh')
        .expect(401);
    });
  });

  describe('POST /api/auth/logout', () => {
    it('should clear refresh token cookie', async () => {
      const registerResponse = await api
        .post('/api/auth/register')
        .send({
          email: 'logout@example.com',
          password: 'Password123!',
          name: 'Test'
        });

      const cookies = registerResponse.headers['set-cookie'];

      const response = await api
        .post('/api/auth/logout')
        .set('Cookie', cookies)
        .expect(200);

      expect(response.headers['set-cookie'][0]).toMatch(/refreshToken=;/);
    });
  });
});
```

---

## Testing Complex Workflows

### Order Creation Flow

```javascript
// tests/integration/orders.test.ts
import { api, authRequest } from '../helpers/testApp';
import { createAuthenticatedUser, createProduct } from '../helpers/factories';
import { prisma } from '../helpers/prisma';

describe('Orders API', () => {
  describe('POST /api/orders', () => {
    it('should create order and update product stock', async () => {
      const { token, user } = await createAuthenticatedUser();
      const product = await createProduct({ stock: 10, price: 25.00 });

      const response = await authRequest(token)
        .post('/api/orders')
        .send({
          items: [
            { productId: product.id, quantity: 3 }
          ],
          shippingAddress: {
            street: '123 Main St',
            city: 'Test City',
            zipCode: '12345'
          }
        })
        .expect(201);

      // Verify order created
      expect(response.body.data).toMatchObject({
        userId: user.id,
        status: 'PENDING',
        total: 75.00
      });
      expect(response.body.data.items).toHaveLength(1);

      // Verify stock updated
      const updatedProduct = await prisma.product.findUnique({
        where: { id: product.id }
      });
      expect(updatedProduct?.stock).toBe(7);
    });

    it('should reject order with insufficient stock', async () => {
      const { token } = await createAuthenticatedUser();
      const product = await createProduct({ stock: 2 });

      const response = await authRequest(token)
        .post('/api/orders')
        .send({
          items: [{ productId: product.id, quantity: 5 }],
          shippingAddress: { street: '123 Main St', city: 'City', zipCode: '12345' }
        })
        .expect(400);

      expect(response.body.error).toContain('Insufficient stock');
    });

    it('should reject order with invalid product', async () => {
      const { token } = await createAuthenticatedUser();

      const response = await authRequest(token)
        .post('/api/orders')
        .send({
          items: [{ productId: 'non-existent', quantity: 1 }],
          shippingAddress: { street: '123 Main St', city: 'City', zipCode: '12345' }
        })
        .expect(404);

      expect(response.body.error).toContain('not found');
    });

    it('should apply discount code', async () => {
      const { token } = await createAuthenticatedUser();
      const product = await createProduct({ price: 100 });

      // Create discount code
      await prisma.discountCode.create({
        data: {
          code: 'SAVE10',
          type: 'PERCENTAGE',
          value: 10,
          active: true
        }
      });

      const response = await authRequest(token)
        .post('/api/orders')
        .send({
          items: [{ productId: product.id, quantity: 1 }],
          discountCode: 'SAVE10',
          shippingAddress: { street: '123 Main St', city: 'City', zipCode: '12345' }
        })
        .expect(201);

      expect(response.body.data.discount).toBe(10);
      expect(response.body.data.total).toBe(90);
    });
  });

  describe('Order Lifecycle', () => {
    it('should handle complete order flow', async () => {
      const { token } = await createAuthenticatedUser();
      const product = await createProduct({ stock: 10, price: 50 });

      // 1. Create order
      const createResponse = await authRequest(token)
        .post('/api/orders')
        .send({
          items: [{ productId: product.id, quantity: 2 }],
          shippingAddress: { street: '123 Main St', city: 'City', zipCode: '12345' }
        })
        .expect(201);

      const orderId = createResponse.body.data.id;
      expect(createResponse.body.data.status).toBe('PENDING');

      // 2. Process payment
      await authRequest(token)
        .post(`/api/orders/${orderId}/pay`)
        .send({ paymentMethod: 'card' })
        .expect(200);

      // 3. Check status updated
      const orderResponse = await authRequest(token)
        .get(`/api/orders/${orderId}`)
        .expect(200);

      expect(orderResponse.body.data.status).toBe('PAID');

      // 4. Ship order (admin)
      const { token: adminToken } = await createAuthenticatedUser({ role: 'ADMIN' });

      await authRequest(adminToken)
        .post(`/api/orders/${orderId}/ship`)
        .send({ trackingNumber: 'TRACK123' })
        .expect(200);

      // 5. Verify final status
      const finalResponse = await authRequest(token)
        .get(`/api/orders/${orderId}`)
        .expect(200);

      expect(finalResponse.body.data.status).toBe('SHIPPED');
      expect(finalResponse.body.data.trackingNumber).toBe('TRACK123');
    });
  });
});
```

---

## Testing File Uploads

```javascript
// tests/integration/uploads.test.ts
import path from 'path';
import { api, authRequest } from '../helpers/testApp';
import { createAuthenticatedUser } from '../helpers/factories';

describe('File Uploads', () => {
  describe('POST /api/users/:id/avatar', () => {
    it('should upload avatar image', async () => {
      const { user, token } = await createAuthenticatedUser();
      const testImagePath = path.join(__dirname, '../fixtures/test-avatar.jpg');

      const response = await authRequest(token)
        .post(`/api/users/${user.id}/avatar`)
        .attach('avatar', testImagePath)
        .expect(200);

      expect(response.body.avatarUrl).toMatch(/^https?:\/\/.+\.(jpg|png)$/);
    });

    it('should reject non-image files', async () => {
      const { user, token } = await createAuthenticatedUser();
      const testFilePath = path.join(__dirname, '../fixtures/document.pdf');

      await authRequest(token)
        .post(`/api/users/${user.id}/avatar`)
        .attach('avatar', testFilePath)
        .expect(400);
    });

    it('should reject files over size limit', async () => {
      const { user, token } = await createAuthenticatedUser();
      const largeFilePath = path.join(__dirname, '../fixtures/large-image.jpg');

      const response = await authRequest(token)
        .post(`/api/users/${user.id}/avatar`)
        .attach('avatar', largeFilePath)
        .expect(413);

      expect(response.body.error).toContain('too large');
    });
  });
});
```

---

## Testing Error Scenarios

```javascript
// tests/integration/errors.test.ts
import { api, authRequest } from '../helpers/testApp';
import { createAuthenticatedUser } from '../helpers/factories';

describe('Error Handling', () => {
  describe('404 Not Found', () => {
    it('should return 404 for non-existent routes', async () => {
      const response = await api
        .get('/api/non-existent-endpoint')
        .expect(404);

      expect(response.body.error).toBe('Not Found');
    });

    it('should return 404 for non-existent resources', async () => {
      const { token } = await createAuthenticatedUser();

      await authRequest(token)
        .get('/api/products/non-existent-id')
        .expect(404);
    });
  });

  describe('400 Bad Request', () => {
    it('should return validation errors', async () => {
      const response = await api
        .post('/api/auth/register')
        .send({})
        .expect(400);

      expect(response.body.errors).toBeInstanceOf(Array);
      expect(response.body.errors.length).toBeGreaterThan(0);
    });

    it('should return 400 for invalid JSON', async () => {
      await api
        .post('/api/auth/login')
        .set('Content-Type', 'application/json')
        .send('{ invalid json }')
        .expect(400);
    });
  });

  describe('401 Unauthorized', () => {
    it('should return 401 for missing token', async () => {
      await api
        .get('/api/users/me')
        .expect(401);
    });

    it('should return 401 for invalid token', async () => {
      await api
        .get('/api/users/me')
        .set('Authorization', 'Bearer invalid-token')
        .expect(401);
    });

    it('should return 401 for expired token', async () => {
      const expiredToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOiIxIiwiZXhwIjoxfQ.xxx';

      await api
        .get('/api/users/me')
        .set('Authorization', `Bearer ${expiredToken}`)
        .expect(401);
    });
  });

  describe('403 Forbidden', () => {
    it('should return 403 for unauthorized access', async () => {
      const { token } = await createAuthenticatedUser({ role: 'USER' });

      await authRequest(token)
        .get('/api/admin/dashboard')
        .expect(403);
    });
  });

  describe('500 Internal Server Error', () => {
    it('should not expose internal error details', async () => {
      // Trigger an internal error somehow
      const response = await api
        .get('/api/test/trigger-error')
        .expect(500);

      expect(response.body.error).toBe('Internal Server Error');
      expect(response.body).not.toHaveProperty('stack');
    });
  });
});
```

---

## Test Organization Best Practices

### Group by Feature

```
tests/
├── integration/
│   ├── auth/
│   │   ├── register.test.ts
│   │   ├── login.test.ts
│   │   └── refresh.test.ts
│   ├── users/
│   │   ├── crud.test.ts
│   │   ├── profile.test.ts
│   │   └── avatar.test.ts
│   ├── products/
│   │   ├── crud.test.ts
│   │   ├── search.test.ts
│   │   └── categories.test.ts
│   └── orders/
│       ├── create.test.ts
│       ├── payment.test.ts
│       └── lifecycle.test.ts
```

### Shared Setup

```javascript
// tests/integration/products/setup.ts
import { createAuthenticatedUser, createProduct } from '../../helpers/factories';

export async function setupProductTests() {
  const { user, token } = await createAuthenticatedUser({ role: 'ADMIN' });

  const products = await Promise.all([
    createProduct({ name: 'Product A', price: 10 }),
    createProduct({ name: 'Product B', price: 20 }),
    createProduct({ name: 'Product C', price: 30 })
  ]);

  return { user, token, products };
}
```

---

## Practice Exercise

Write integration tests for a blog API:

1. **Test post CRUD**:
   - Create post (authenticated)
   - Read posts (public, with pagination)
   - Update post (author only)
   - Delete post (author or admin)

2. **Test comments**:
   - Add comment to post
   - Delete own comment
   - List comments for post

3. **Test edge cases**:
   - Comment on non-existent post
   - Update non-existent post
   - Access without authentication

---

## Key Takeaways

1. **Test real database** - Use test database, not mocks
2. **Clean between tests** - Ensure isolation
3. **Test full flows** - Request to database to response
4. **Include auth tests** - Test protected routes
5. **Test error cases** - 400, 401, 403, 404, 500
6. **Use factories** - Quickly create test data

---

## What's Next?

Tomorrow, we'll dive deep into **Mocking** - how to isolate components by mocking dependencies like databases, APIs, and external services.
