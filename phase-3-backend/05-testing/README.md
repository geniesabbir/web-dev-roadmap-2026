# Backend Testing

**Duration:** 1 week

## Learning Objectives

By the end of this section, you will:
- Write unit tests for services
- Write integration tests for APIs
- Set up test databases
- Mock external dependencies
- Achieve good test coverage

---

## Day 1: Testing Setup

### Vitest Setup (Recommended)
```bash
npm install -D vitest @vitest/coverage-v8
npm install -D supertest @types/supertest
```

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    setupFiles: ['./src/test/setup.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html'],
    },
  },
})

// src/test/setup.ts
import { beforeAll, afterAll, beforeEach } from 'vitest'
import { prisma } from '../config/database'

beforeAll(async () => {
  // Connect to test database
  await prisma.$connect()
})

afterAll(async () => {
  await prisma.$disconnect()
})

beforeEach(async () => {
  // Clean database before each test
  await prisma.comment.deleteMany()
  await prisma.post.deleteMany()
  await prisma.user.deleteMany()
})
```

### Test Database
```bash
# .env.test
DATABASE_URL="postgresql://user:pass@localhost:5432/myapp_test"
```

```json
// package.json
{
  "scripts": {
    "test": "dotenv -e .env.test -- vitest",
    "test:coverage": "dotenv -e .env.test -- vitest run --coverage",
    "test:watch": "dotenv -e .env.test -- vitest --watch"
  }
}
```

---

## Day 2: Unit Testing Services

### Testing Pure Functions
```typescript
// src/utils/formatters.test.ts
import { describe, it, expect } from 'vitest'
import { formatPrice, slugify } from './formatters'

describe('formatPrice', () => {
  it('formats cents to dollars', () => {
    expect(formatPrice(1000)).toBe('$10.00')
    expect(formatPrice(1234)).toBe('$12.34')
  })

  it('handles zero', () => {
    expect(formatPrice(0)).toBe('$0.00')
  })
})

describe('slugify', () => {
  it('converts to lowercase', () => {
    expect(slugify('Hello World')).toBe('hello-world')
  })

  it('removes special characters', () => {
    expect(slugify('Hello! World?')).toBe('hello-world')
  })
})
```

### Testing Services with Database
```typescript
// src/services/userService.test.ts
import { describe, it, expect, beforeEach } from 'vitest'
import { userService } from './userService'
import { prisma } from '../config/database'

describe('userService', () => {
  describe('create', () => {
    it('creates a new user', async () => {
      const userData = {
        email: 'test@example.com',
        password: 'password123',
        name: 'Test User',
      }

      const user = await userService.create(userData)

      expect(user).toMatchObject({
        email: 'test@example.com',
        name: 'Test User',
      })
      expect(user.id).toBeDefined()
      expect(user.password).toBeUndefined() // Should not return password
    })

    it('throws error for duplicate email', async () => {
      const userData = {
        email: 'test@example.com',
        password: 'password123',
        name: 'Test User',
      }

      await userService.create(userData)

      await expect(userService.create(userData)).rejects.toThrow(
        'Email already exists'
      )
    })
  })

  describe('findById', () => {
    it('returns user when found', async () => {
      const created = await prisma.user.create({
        data: {
          email: 'test@example.com',
          password: 'hashed',
          name: 'Test',
        },
      })

      const user = await userService.findById(created.id)

      expect(user).toMatchObject({
        id: created.id,
        email: 'test@example.com',
      })
    })

    it('throws error when not found', async () => {
      await expect(userService.findById('non-existent-id')).rejects.toThrow(
        'User not found'
      )
    })
  })
})
```

---

## Day 3: Integration Testing APIs

### Testing with Supertest
```typescript
// src/test/helpers.ts
import { prisma } from '../config/database'
import bcrypt from 'bcrypt'
import { generateAccessToken } from '../utils/jwt'

export async function createTestUser(data?: Partial<{
  email: string
  name: string
  password: string
}>) {
  const password = await bcrypt.hash(data?.password || 'password123', 10)

  return prisma.user.create({
    data: {
      email: data?.email || `test-${Date.now()}@example.com`,
      name: data?.name || 'Test User',
      password,
    },
  })
}

export function getAuthToken(user: { id: string; email: string }) {
  return generateAccessToken({ userId: user.id, email: user.email })
}

// src/routes/users.test.ts
import { describe, it, expect, beforeEach } from 'vitest'
import request from 'supertest'
import { app } from '../app'
import { createTestUser, getAuthToken } from '../test/helpers'

describe('GET /api/v1/users', () => {
  it('returns 401 without authentication', async () => {
    const response = await request(app).get('/api/v1/users')

    expect(response.status).toBe(401)
  })

  it('returns users when authenticated', async () => {
    const user = await createTestUser()
    const token = getAuthToken(user)

    const response = await request(app)
      .get('/api/v1/users')
      .set('Authorization', `Bearer ${token}`)

    expect(response.status).toBe(200)
    expect(response.body.status).toBe('success')
    expect(response.body.data).toHaveLength(1)
  })
})

describe('POST /api/v1/users', () => {
  it('creates a new user', async () => {
    const response = await request(app)
      .post('/api/v1/users')
      .send({
        email: 'new@example.com',
        password: 'Password123!',
        name: 'New User',
      })

    expect(response.status).toBe(201)
    expect(response.body.data.email).toBe('new@example.com')
  })

  it('validates required fields', async () => {
    const response = await request(app)
      .post('/api/v1/users')
      .send({ email: 'invalid' })

    expect(response.status).toBe(400)
    expect(response.body.errors).toBeDefined()
  })
})

describe('GET /api/v1/users/:id', () => {
  it('returns user by id', async () => {
    const user = await createTestUser()
    const token = getAuthToken(user)

    const response = await request(app)
      .get(`/api/v1/users/${user.id}`)
      .set('Authorization', `Bearer ${token}`)

    expect(response.status).toBe(200)
    expect(response.body.data.id).toBe(user.id)
  })

  it('returns 404 for non-existent user', async () => {
    const user = await createTestUser()
    const token = getAuthToken(user)

    const response = await request(app)
      .get('/api/v1/users/non-existent-id')
      .set('Authorization', `Bearer ${token}`)

    expect(response.status).toBe(404)
  })
})
```

---

## Day 4: Testing Authentication

```typescript
// src/routes/auth.test.ts
import { describe, it, expect } from 'vitest'
import request from 'supertest'
import { app } from '../app'
import { createTestUser } from '../test/helpers'

describe('POST /api/v1/auth/login', () => {
  it('returns tokens for valid credentials', async () => {
    await createTestUser({
      email: 'test@example.com',
      password: 'password123',
    })

    const response = await request(app)
      .post('/api/v1/auth/login')
      .send({
        email: 'test@example.com',
        password: 'password123',
      })

    expect(response.status).toBe(200)
    expect(response.body.accessToken).toBeDefined()
    expect(response.body.refreshToken).toBeDefined()
  })

  it('returns 401 for invalid credentials', async () => {
    await createTestUser({
      email: 'test@example.com',
      password: 'password123',
    })

    const response = await request(app)
      .post('/api/v1/auth/login')
      .send({
        email: 'test@example.com',
        password: 'wrongpassword',
      })

    expect(response.status).toBe(401)
  })

  it('returns 401 for non-existent user', async () => {
    const response = await request(app)
      .post('/api/v1/auth/login')
      .send({
        email: 'nonexistent@example.com',
        password: 'password123',
      })

    expect(response.status).toBe(401)
  })
})

describe('POST /api/v1/auth/refresh', () => {
  it('returns new access token', async () => {
    const user = await createTestUser({
      email: 'test@example.com',
      password: 'password123',
    })

    // Login first
    const loginResponse = await request(app)
      .post('/api/v1/auth/login')
      .send({
        email: 'test@example.com',
        password: 'password123',
      })

    // Refresh
    const response = await request(app)
      .post('/api/v1/auth/refresh')
      .send({ refreshToken: loginResponse.body.refreshToken })

    expect(response.status).toBe(200)
    expect(response.body.accessToken).toBeDefined()
  })
})
```

---

## Day 5: Mocking External Services

### Mocking with Vitest
```typescript
// src/services/emailService.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { emailService } from './emailService'

// Mock the email provider
vi.mock('../config/email', () => ({
  sendEmail: vi.fn(),
}))

import { sendEmail } from '../config/email'

describe('emailService', () => {
  beforeEach(() => {
    vi.clearAllMocks()
  })

  it('sends welcome email', async () => {
    (sendEmail as any).mockResolvedValue({ success: true })

    await emailService.sendWelcomeEmail('test@example.com', 'Test User')

    expect(sendEmail).toHaveBeenCalledWith({
      to: 'test@example.com',
      subject: 'Welcome to Our App!',
      template: 'welcome',
      data: { name: 'Test User' },
    })
  })

  it('handles email failure gracefully', async () => {
    (sendEmail as any).mockRejectedValue(new Error('SMTP Error'))

    // Should not throw, just log
    await expect(
      emailService.sendWelcomeEmail('test@example.com', 'Test')
    ).resolves.not.toThrow()
  })
})
```

### Mocking HTTP Requests
```typescript
// Using MSW for HTTP mocking
import { setupServer } from 'msw/node'
import { rest } from 'msw'

const server = setupServer(
  rest.get('https://api.stripe.com/v1/customers/:id', (req, res, ctx) => {
    return res(
      ctx.json({
        id: req.params.id,
        email: 'customer@example.com',
      })
    )
  })
)

beforeAll(() => server.listen())
afterEach(() => server.resetHandlers())
afterAll(() => server.close())

it('fetches customer from Stripe', async () => {
  const customer = await paymentService.getCustomer('cus_123')
  expect(customer.email).toBe('customer@example.com')
})
```

---

## Test Patterns

### Factory Pattern
```typescript
// src/test/factories/userFactory.ts
import { faker } from '@faker-js/faker'
import { prisma } from '../../config/database'
import bcrypt from 'bcrypt'

interface UserOverrides {
  email?: string
  name?: string
  password?: string
  role?: 'USER' | 'ADMIN'
}

export async function createUser(overrides: UserOverrides = {}) {
  const password = await bcrypt.hash(overrides.password || 'password123', 10)

  return prisma.user.create({
    data: {
      email: overrides.email || faker.internet.email(),
      name: overrides.name || faker.person.fullName(),
      password,
      role: overrides.role || 'USER',
    },
  })
}

export async function createPost(authorId: string, overrides = {}) {
  return prisma.post.create({
    data: {
      title: faker.lorem.sentence(),
      content: faker.lorem.paragraphs(3),
      published: false,
      authorId,
      ...overrides,
    },
  })
}
```

### Test Coverage
```bash
# Run with coverage
npm run test:coverage

# Aim for:
# - Statements: 80%+
# - Branches: 75%+
# - Functions: 80%+
# - Lines: 80%+
```

---

## Checklist Before Moving On

- [ ] Can write unit tests for services
- [ ] Can write integration tests for APIs
- [ ] Can set up test databases
- [ ] Can mock external dependencies
- [ ] Have good test coverage on projects

---

**Phase 3 Complete!**

Before moving to Phase 4, ensure you:
- [ ] Completed all Phase 3 sections
- [ ] Built Blog API with full test coverage
- [ ] Built E-commerce Backend
- [ ] Implemented authentication properly
- [ ] Have well-documented APIs

**Next:** [Phase 4: DevOps & Deployment](../../phase-4-devops/README.md)
