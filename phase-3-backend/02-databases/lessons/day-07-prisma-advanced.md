# Day 7: Advanced Prisma

## Introduction

Now that you've mastered CRUD operations and relations, it's time to explore advanced Prisma features. This lesson covers transactions, raw SQL queries, middleware, performance optimization, and error handling patterns.

## Learning Objectives

By the end of this lesson, you will:
- Execute transactions for data integrity
- Use raw SQL when needed
- Implement Prisma middleware
- Optimize query performance
- Handle errors gracefully
- Implement common advanced patterns

---

## Transactions

### Sequential Transactions ($transaction)

Run multiple operations as a single atomic transaction.

```typescript
// Array syntax - sequential operations
const [user, post] = await prisma.$transaction([
  prisma.user.create({
    data: { email: 'alice@example.com', name: 'Alice' },
  }),
  prisma.post.create({
    data: {
      title: 'First Post',
      authorId: 1, // Must know ID beforehand
    },
  }),
]);

// All succeed or all fail
```

### Interactive Transactions

For operations that depend on each other.

```typescript
// Interactive transaction with callback
const result = await prisma.$transaction(async (tx) => {
  // Create user
  const user = await tx.user.create({
    data: { email: 'bob@example.com', name: 'Bob' },
  });

  // Use user.id for post
  const post = await tx.post.create({
    data: {
      title: 'Bob\'s Post',
      authorId: user.id,
    },
  });

  // Can add business logic
  if (someCondition) {
    throw new Error('Transaction cancelled');
    // Both user and post will be rolled back
  }

  return { user, post };
});
```

### Transaction Options

```typescript
const result = await prisma.$transaction(
  async (tx) => {
    // Operations here
  },
  {
    maxWait: 5000,    // Max time to wait for transaction slot (ms)
    timeout: 10000,   // Max transaction duration (ms)
    isolationLevel: 'Serializable', // Transaction isolation
  }
);

// Isolation levels:
// - ReadUncommitted (fastest, least safe)
// - ReadCommitted (default for most DBs)
// - RepeatableRead
// - Serializable (slowest, most safe)
```

### Common Transaction Patterns

```typescript
// Transfer money between accounts
async function transferFunds(fromId: number, toId: number, amount: number) {
  return prisma.$transaction(async (tx) => {
    // Deduct from sender
    const sender = await tx.account.update({
      where: { id: fromId },
      data: { balance: { decrement: amount } },
    });

    // Check sufficient funds
    if (sender.balance < 0) {
      throw new Error('Insufficient funds');
    }

    // Add to receiver
    const receiver = await tx.account.update({
      where: { id: toId },
      data: { balance: { increment: amount } },
    });

    // Create transaction record
    await tx.transaction.create({
      data: {
        fromAccountId: fromId,
        toAccountId: toId,
        amount,
        type: 'TRANSFER',
      },
    });

    return { sender, receiver };
  });
}

// Order with inventory check
async function createOrder(userId: number, items: Array<{ productId: number; quantity: number }>) {
  return prisma.$transaction(async (tx) => {
    // Check and update inventory
    for (const item of items) {
      const product = await tx.product.update({
        where: { id: item.productId },
        data: { stock: { decrement: item.quantity } },
      });

      if (product.stock < 0) {
        throw new Error(`Insufficient stock for product ${product.name}`);
      }
    }

    // Calculate total
    const products = await tx.product.findMany({
      where: { id: { in: items.map(i => i.productId) } },
    });

    const total = items.reduce((sum, item) => {
      const product = products.find(p => p.id === item.productId)!;
      return sum + product.price * item.quantity;
    }, 0);

    // Create order
    const order = await tx.order.create({
      data: {
        userId,
        total,
        items: {
          create: items.map(item => ({
            productId: item.productId,
            quantity: item.quantity,
            price: products.find(p => p.id === item.productId)!.price,
          })),
        },
      },
      include: { items: true },
    });

    return order;
  });
}
```

---

## Raw SQL Queries

### queryRaw - SELECT Queries

```typescript
// Simple raw query
const users = await prisma.$queryRaw`
  SELECT * FROM users WHERE role = 'ADMIN'
`;

// With parameters (safe from SQL injection)
const role = 'ADMIN';
const users = await prisma.$queryRaw`
  SELECT * FROM users WHERE role = ${role}
`;

// Complex queries
const stats = await prisma.$queryRaw`
  SELECT
    DATE_TRUNC('month', created_at) as month,
    COUNT(*) as count,
    SUM(total) as revenue
  FROM orders
  WHERE created_at >= ${startDate}
  GROUP BY DATE_TRUNC('month', created_at)
  ORDER BY month DESC
`;

// Type the result
interface UserStats {
  month: Date;
  count: bigint;
  revenue: number;
}

const stats = await prisma.$queryRaw<UserStats[]>`
  SELECT ...
`;
```

### executeRaw - INSERT, UPDATE, DELETE

```typescript
// Execute raw SQL (returns affected row count)
const count = await prisma.$executeRaw`
  UPDATE users SET is_active = false
  WHERE last_login < ${cutoffDate}
`;

console.log(`Deactivated ${count} users`);

// Bulk operations
const result = await prisma.$executeRaw`
  INSERT INTO audit_logs (action, user_id, timestamp)
  SELECT 'INACTIVE', id, NOW()
  FROM users
  WHERE last_login < ${cutoffDate}
`;
```

### queryRawUnsafe and executeRawUnsafe

```typescript
// WARNING: Only use with trusted input!
// Vulnerable to SQL injection

// Dynamic table name (can't parameterize table names)
const tableName = 'users'; // Must be validated!
const users = await prisma.$queryRawUnsafe(
  `SELECT * FROM ${tableName} WHERE id = $1`,
  userId
);

// Dynamic column selection
const columns = ['id', 'name', 'email']; // Must be validated!
const users = await prisma.$queryRawUnsafe(
  `SELECT ${columns.join(', ')} FROM users`
);
```

### SQL Template Tag

```typescript
import { Prisma } from '@prisma/client';

// Build dynamic queries safely
function buildUserQuery(filters: { role?: string; active?: boolean }) {
  const conditions: Prisma.Sql[] = [];

  if (filters.role) {
    conditions.push(Prisma.sql`role = ${filters.role}`);
  }
  if (filters.active !== undefined) {
    conditions.push(Prisma.sql`is_active = ${filters.active}`);
  }

  const whereClause = conditions.length > 0
    ? Prisma.sql`WHERE ${Prisma.join(conditions, ' AND ')}`
    : Prisma.empty;

  return prisma.$queryRaw`
    SELECT * FROM users ${whereClause}
  `;
}
```

---

## Prisma Middleware

### Logging Middleware

```typescript
const prisma = new PrismaClient();

// Log all queries
prisma.$use(async (params, next) => {
  const before = Date.now();
  const result = await next(params);
  const after = Date.now();

  console.log(`${params.model}.${params.action} - ${after - before}ms`);
  return result;
});
```

### Soft Delete Middleware

```typescript
// Automatically filter deleted records
prisma.$use(async (params, next) => {
  // Add deletedAt filter to all reads
  if (params.action === 'findUnique' || params.action === 'findFirst') {
    params.action = 'findFirst';
    params.args.where = {
      ...params.args.where,
      deletedAt: null,
    };
  }

  if (params.action === 'findMany') {
    if (params.args.where) {
      if (params.args.where.deletedAt === undefined) {
        params.args.where.deletedAt = null;
      }
    } else {
      params.args.where = { deletedAt: null };
    }
  }

  // Convert delete to soft delete
  if (params.action === 'delete') {
    params.action = 'update';
    params.args.data = { deletedAt: new Date() };
  }

  if (params.action === 'deleteMany') {
    params.action = 'updateMany';
    if (params.args.data) {
      params.args.data.deletedAt = new Date();
    } else {
      params.args.data = { deletedAt: new Date() };
    }
  }

  return next(params);
});
```

### Audit Log Middleware

```typescript
prisma.$use(async (params, next) => {
  const result = await next(params);

  // Log write operations
  if (['create', 'update', 'delete'].includes(params.action)) {
    await prisma.auditLog.create({
      data: {
        model: params.model!,
        action: params.action,
        recordId: result?.id?.toString(),
        data: JSON.stringify(params.args),
        timestamp: new Date(),
      },
    });
  }

  return result;
});
```

---

## Prisma Client Extensions

```typescript
import { Prisma, PrismaClient } from '@prisma/client';

// Extend Prisma Client with custom methods
const prisma = new PrismaClient().$extends({
  model: {
    user: {
      // Custom method on User model
      async findByEmail(email: string) {
        return prisma.user.findUnique({ where: { email } });
      },

      async softDelete(id: number) {
        return prisma.user.update({
          where: { id },
          data: { deletedAt: new Date(), isActive: false },
        });
      },
    },

    post: {
      async publish(id: number) {
        return prisma.post.update({
          where: { id },
          data: { published: true, publishedAt: new Date() },
        });
      },

      async findPublished() {
        return prisma.post.findMany({
          where: { published: true },
          orderBy: { publishedAt: 'desc' },
        });
      },
    },
  },

  // Custom result fields
  result: {
    user: {
      fullName: {
        needs: { firstName: true, lastName: true },
        compute(user) {
          return `${user.firstName} ${user.lastName}`;
        },
      },
    },
  },

  // Query modifications
  query: {
    user: {
      async findMany({ args, query }) {
        // Always exclude password
        args.select = { ...args.select, password: false };
        return query(args);
      },
    },
  },
});

// Usage
const user = await prisma.user.findByEmail('alice@example.com');
await prisma.user.softDelete(1);
await prisma.post.publish(1);
const posts = await prisma.post.findPublished();
```

---

## Performance Optimization

### Select Only What You Need

```typescript
// Bad - fetches all columns
const users = await prisma.user.findMany();

// Good - only needed columns
const users = await prisma.user.findMany({
  select: { id: true, name: true, email: true },
});
```

### Avoid N+1 Queries

```typescript
// Bad - N+1 problem
const posts = await prisma.post.findMany();
for (const post of posts) {
  const author = await prisma.user.findUnique({
    where: { id: post.authorId },
  });
  console.log(post.title, author?.name);
}

// Good - single query with include
const posts = await prisma.post.findMany({
  include: { author: true },
});
for (const post of posts) {
  console.log(post.title, post.author.name);
}
```

### Batch Operations

```typescript
// Bad - individual creates
for (const userData of usersData) {
  await prisma.user.create({ data: userData });
}

// Good - batch create
await prisma.user.createMany({
  data: usersData,
  skipDuplicates: true,
});
```

### Use Database Indexes

```prisma
model Post {
  id          Int      @id @default(autoincrement())
  title       String
  authorId    Int
  categoryId  Int?
  published   Boolean  @default(false)
  publishedAt DateTime?

  // Single column indexes
  @@index([authorId])
  @@index([categoryId])

  // Composite index for common queries
  @@index([published, publishedAt])
  @@index([authorId, published])
}
```

### Cursor Pagination for Large Datasets

```typescript
// Offset pagination is slow for large offsets
const page1000 = await prisma.post.findMany({
  skip: 10000, // Slow!
  take: 10,
});

// Cursor pagination is efficient
const posts = await prisma.post.findMany({
  take: 10,
  skip: 1, // Skip cursor
  cursor: { id: lastSeenId },
  orderBy: { id: 'asc' },
});
```

### Connection Pooling

```typescript
// Configure pool size based on your needs
const prisma = new PrismaClient({
  datasources: {
    db: {
      url: process.env.DATABASE_URL,
    },
  },
});

// In DATABASE_URL, configure pool:
// postgresql://user:pass@host:5432/db?connection_limit=20&pool_timeout=30
```

---

## Error Handling

### Prisma Error Types

```typescript
import { PrismaClientKnownRequestError, PrismaClientValidationError } from '@prisma/client/runtime/library';

try {
  await prisma.user.create({
    data: { email: 'duplicate@example.com' },
  });
} catch (error) {
  if (error instanceof PrismaClientKnownRequestError) {
    // Known Prisma errors
    switch (error.code) {
      case 'P2002':
        // Unique constraint violation
        console.log('Email already exists');
        break;
      case 'P2025':
        // Record not found
        console.log('Record not found');
        break;
      case 'P2003':
        // Foreign key constraint failed
        console.log('Related record not found');
        break;
      default:
        console.log('Database error:', error.message);
    }
  } else if (error instanceof PrismaClientValidationError) {
    // Invalid query
    console.log('Invalid query:', error.message);
  } else {
    throw error;
  }
}
```

### Common Error Codes

```typescript
const PrismaErrorCodes = {
  // Unique constraint
  P2002: 'Unique constraint failed',

  // Foreign key constraint
  P2003: 'Foreign key constraint failed',

  // Record not found
  P2025: 'Record not found',

  // Required field missing
  P2012: 'Missing required value',

  // Invalid ID
  P2023: 'Inconsistent column data',
};
```

### Error Handling Middleware

```typescript
// Express error handler for Prisma errors
import { PrismaClientKnownRequestError } from '@prisma/client/runtime/library';

function prismaErrorHandler(err: Error, req: Request, res: Response, next: NextFunction) {
  if (err instanceof PrismaClientKnownRequestError) {
    switch (err.code) {
      case 'P2002':
        return res.status(409).json({
          error: 'Conflict',
          message: `A record with this ${err.meta?.target} already exists`,
        });
      case 'P2025':
        return res.status(404).json({
          error: 'Not Found',
          message: 'The requested resource was not found',
        });
      case 'P2003':
        return res.status(400).json({
          error: 'Bad Request',
          message: 'Referenced record does not exist',
        });
      default:
        return res.status(500).json({
          error: 'Database Error',
          message: 'An unexpected database error occurred',
        });
    }
  }

  next(err);
}
```

---

## Common Patterns

### Repository Pattern

```typescript
// src/repositories/user.repository.ts
import prisma from '../lib/prisma';
import { Prisma, User } from '@prisma/client';

export class UserRepository {
  async findById(id: number): Promise<User | null> {
    return prisma.user.findUnique({ where: { id } });
  }

  async findByEmail(email: string): Promise<User | null> {
    return prisma.user.findUnique({ where: { email } });
  }

  async findMany(params: {
    where?: Prisma.UserWhereInput;
    orderBy?: Prisma.UserOrderByWithRelationInput;
    skip?: number;
    take?: number;
  }): Promise<User[]> {
    return prisma.user.findMany(params);
  }

  async create(data: Prisma.UserCreateInput): Promise<User> {
    return prisma.user.create({ data });
  }

  async update(id: number, data: Prisma.UserUpdateInput): Promise<User> {
    return prisma.user.update({ where: { id }, data });
  }

  async delete(id: number): Promise<User> {
    return prisma.user.delete({ where: { id } });
  }

  async count(where?: Prisma.UserWhereInput): Promise<number> {
    return prisma.user.count({ where });
  }
}
```

### Service Layer Pattern

```typescript
// src/services/user.service.ts
import { UserRepository } from '../repositories/user.repository';
import { hash, compare } from 'bcrypt';

export class UserService {
  private userRepo = new UserRepository();

  async createUser(data: { email: string; password: string; name: string }) {
    const existingUser = await this.userRepo.findByEmail(data.email);
    if (existingUser) {
      throw new Error('Email already registered');
    }

    const hashedPassword = await hash(data.password, 12);
    return this.userRepo.create({
      ...data,
      password: hashedPassword,
    });
  }

  async verifyCredentials(email: string, password: string) {
    const user = await this.userRepo.findByEmail(email);
    if (!user) {
      return null;
    }

    const isValid = await compare(password, user.password);
    return isValid ? user : null;
  }

  async updateProfile(userId: number, data: { name?: string; bio?: string }) {
    return this.userRepo.update(userId, data);
  }
}
```

### Pagination Helper

```typescript
// src/utils/pagination.ts
import { Prisma } from '@prisma/client';

export interface PaginationParams {
  page?: number;
  pageSize?: number;
  cursor?: number;
}

export interface PaginatedResult<T> {
  data: T[];
  meta: {
    total: number;
    page: number;
    pageSize: number;
    totalPages: number;
    hasMore: boolean;
  };
}

export function getPaginationParams(params: PaginationParams) {
  const page = Math.max(1, params.page || 1);
  const pageSize = Math.min(100, Math.max(1, params.pageSize || 10));
  const skip = (page - 1) * pageSize;

  return { page, pageSize, skip, take: pageSize };
}

export function buildPaginatedResult<T>(
  data: T[],
  total: number,
  params: ReturnType<typeof getPaginationParams>
): PaginatedResult<T> {
  const totalPages = Math.ceil(total / params.pageSize);

  return {
    data,
    meta: {
      total,
      page: params.page,
      pageSize: params.pageSize,
      totalPages,
      hasMore: params.page < totalPages,
    },
  };
}
```

---

## Key Takeaways

1. **Use transactions for data integrity** - Multiple related operations should be atomic
2. **Raw SQL when needed** - Prisma supports complex SQL when the ORM isn't enough
3. **Middleware for cross-cutting concerns** - Logging, soft delete, audit trails
4. **Extensions for custom methods** - Keep model-specific logic organized
5. **Select only needed fields** - Avoid fetching unnecessary data
6. **Avoid N+1** - Use include/select for related data
7. **Handle errors gracefully** - Different error codes need different handling

---

## What's Next?

Tomorrow we'll explore **MongoDB Basics** - learning a popular NoSQL document database!
