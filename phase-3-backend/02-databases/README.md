# Databases

**Duration:** 3 weeks

## Learning Objectives

By the end of this section, you will:
- Design relational database schemas
- Write SQL queries confidently
- Use Prisma ORM effectively
- Understand when to use SQL vs NoSQL
- Optimize database performance

---

## Week 1: SQL Fundamentals

### Day 1: Database Concepts
**Topics:**
- What is a database?
- Relational vs Non-relational
- Tables, rows, columns
- Primary and foreign keys
- Data types
- PostgreSQL setup

**Setup:**
```bash
# Install PostgreSQL
# macOS
brew install postgresql@15
brew services start postgresql@15

# Or use Docker
docker run --name postgres -e POSTGRES_PASSWORD=password -p 5432:5432 -d postgres:15

# Connect with psql
psql -U postgres
```

### Day 2: Basic SQL
**Topics:**
- CREATE TABLE
- INSERT, SELECT, UPDATE, DELETE
- WHERE clauses
- ORDER BY, LIMIT, OFFSET
- NULL handling

**Exercises:**
```sql
-- exercises/01-basic-sql.sql

-- Create tables
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  name VARCHAR(100) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  title VARCHAR(255) NOT NULL,
  content TEXT,
  published BOOLEAN DEFAULT false,
  user_id INTEGER REFERENCES users(id),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert data
INSERT INTO users (email, name) VALUES
  ('john@example.com', 'John Doe'),
  ('jane@example.com', 'Jane Smith');

-- Query data
SELECT * FROM users WHERE name LIKE 'J%';
SELECT * FROM posts WHERE published = true ORDER BY created_at DESC;
```

### Day 3: Joins and Relationships
**Topics:**
- One-to-one, one-to-many, many-to-many
- INNER JOIN
- LEFT/RIGHT JOIN
- Self joins
- Junction tables

**Exercises:**
```sql
-- exercises/02-joins.sql

-- One-to-many: Users -> Posts
SELECT u.name, p.title
FROM users u
INNER JOIN posts p ON u.id = p.user_id;

-- Many-to-many: Posts <-> Tags
CREATE TABLE tags (
  id SERIAL PRIMARY KEY,
  name VARCHAR(50) UNIQUE NOT NULL
);

CREATE TABLE post_tags (
  post_id INTEGER REFERENCES posts(id),
  tag_id INTEGER REFERENCES tags(id),
  PRIMARY KEY (post_id, tag_id)
);

-- Get posts with their tags
SELECT p.title, array_agg(t.name) as tags
FROM posts p
LEFT JOIN post_tags pt ON p.id = pt.post_id
LEFT JOIN tags t ON pt.tag_id = t.id
GROUP BY p.id;
```

### Day 4: Aggregations and Grouping
**Topics:**
- COUNT, SUM, AVG, MIN, MAX
- GROUP BY
- HAVING
- Subqueries
- Common table expressions (CTEs)

**Exercises:**
```sql
-- exercises/03-aggregations.sql

-- Count posts per user
SELECT u.name, COUNT(p.id) as post_count
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
GROUP BY u.id
HAVING COUNT(p.id) > 5;

-- CTE example
WITH active_users AS (
  SELECT user_id, COUNT(*) as post_count
  FROM posts
  WHERE created_at > NOW() - INTERVAL '30 days'
  GROUP BY user_id
)
SELECT u.name, au.post_count
FROM users u
JOIN active_users au ON u.id = au.user_id;
```

### Day 5: Indexes and Performance
**Topics:**
- What are indexes?
- Creating indexes
- EXPLAIN ANALYZE
- Index types (B-tree, Hash, GIN)
- When to index

**Exercises:**
```sql
-- exercises/04-indexes.sql

-- Create indexes
CREATE INDEX idx_posts_user_id ON posts(user_id);
CREATE INDEX idx_posts_created_at ON posts(created_at DESC);
CREATE INDEX idx_users_email ON users(email);

-- Analyze query performance
EXPLAIN ANALYZE
SELECT * FROM posts WHERE user_id = 1;
```

---

## Week 2: Prisma ORM

### Day 1: Prisma Setup
**Topics:**
- What is an ORM?
- Prisma vs other ORMs
- Installation and setup
- Schema definition
- Migrations

**Setup:**
```bash
npm install prisma @prisma/client
npx prisma init
```

**Schema Example:**
```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(uuid())
  email     String   @unique
  name      String
  password  String
  posts     Post[]
  comments  Comment[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Post {
  id        String    @id @default(uuid())
  title     String
  content   String?
  published Boolean   @default(false)
  author    User      @relation(fields: [authorId], references: [id])
  authorId  String
  comments  Comment[]
  tags      Tag[]
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
}

model Comment {
  id        String   @id @default(uuid())
  content   String
  author    User     @relation(fields: [authorId], references: [id])
  authorId  String
  post      Post     @relation(fields: [postId], references: [id])
  postId    String
  createdAt DateTime @default(now())
}

model Tag {
  id    String @id @default(uuid())
  name  String @unique
  posts Post[]
}
```

**Migrations:**
```bash
# Create migration
npx prisma migrate dev --name init

# Apply migrations in production
npx prisma migrate deploy

# Generate client
npx prisma generate

# Open Prisma Studio
npx prisma studio
```

### Day 2: CRUD Operations
**Topics:**
- Create operations
- Read operations
- Update operations
- Delete operations
- Selecting specific fields

**Examples:**
```typescript
// src/services/postService.ts
import { prisma } from '../config/database'

export const postService = {
  // Create
  async create(data: { title: string; content?: string; authorId: string }) {
    return prisma.post.create({
      data,
      include: { author: { select: { id: true, name: true } } },
    })
  },

  // Read one
  async findById(id: string) {
    return prisma.post.findUnique({
      where: { id },
      include: {
        author: { select: { id: true, name: true } },
        tags: true,
        _count: { select: { comments: true } },
      },
    })
  },

  // Read many with filters
  async findMany(params: {
    skip?: number
    take?: number
    published?: boolean
    authorId?: string
  }) {
    return prisma.post.findMany({
      where: {
        published: params.published,
        authorId: params.authorId,
      },
      skip: params.skip,
      take: params.take,
      orderBy: { createdAt: 'desc' },
      include: {
        author: { select: { id: true, name: true } },
        _count: { select: { comments: true } },
      },
    })
  },

  // Update
  async update(id: string, data: { title?: string; content?: string; published?: boolean }) {
    return prisma.post.update({
      where: { id },
      data,
    })
  },

  // Delete
  async delete(id: string) {
    return prisma.post.delete({
      where: { id },
    })
  },
}
```

### Day 3: Relations and Nested Writes
**Topics:**
- Include and select
- Nested creates
- Connecting existing records
- Disconnecting records
- Relation filters

**Examples:**
```typescript
// Create post with tags (connect existing and create new)
const post = await prisma.post.create({
  data: {
    title: 'My Post',
    content: 'Content here',
    author: { connect: { id: userId } },
    tags: {
      connectOrCreate: [
        { where: { name: 'javascript' }, create: { name: 'javascript' } },
        { where: { name: 'tutorial' }, create: { name: 'tutorial' } },
      ],
    },
  },
})

// Update with relation changes
await prisma.post.update({
  where: { id: postId },
  data: {
    tags: {
      disconnect: [{ id: tagToRemove }],
      connect: [{ id: tagToAdd }],
    },
  },
})

// Filter by relation
const posts = await prisma.post.findMany({
  where: {
    author: { name: { contains: 'John' } },
    tags: { some: { name: 'javascript' } },
  },
})
```

### Day 4: Transactions and Advanced Queries
**Topics:**
- Interactive transactions
- Sequential operations
- Raw queries
- Aggregations
- Pagination patterns

**Examples:**
```typescript
// Transaction
const result = await prisma.$transaction(async (tx) => {
  const user = await tx.user.create({
    data: { email: 'new@example.com', name: 'New User', password: hash },
  })

  const post = await tx.post.create({
    data: { title: 'Welcome!', authorId: user.id },
  })

  return { user, post }
})

// Aggregation
const stats = await prisma.post.aggregate({
  _count: { id: true },
  _avg: { views: true },
  where: { published: true },
})

// Cursor-based pagination
const posts = await prisma.post.findMany({
  take: 10,
  skip: 1, // Skip the cursor
  cursor: { id: lastPostId },
  orderBy: { createdAt: 'desc' },
})
```

### Day 5: Best Practices
**Topics:**
- Database seeding
- Soft deletes
- Timestamps
- Performance tips
- Connection pooling

**Seed Script:**
```typescript
// prisma/seed.ts
import { PrismaClient } from '@prisma/client'
import bcrypt from 'bcrypt'

const prisma = new PrismaClient()

async function main() {
  // Clear existing data
  await prisma.comment.deleteMany()
  await prisma.post.deleteMany()
  await prisma.user.deleteMany()

  // Create users
  const password = await bcrypt.hash('password123', 10)

  const user1 = await prisma.user.create({
    data: {
      email: 'john@example.com',
      name: 'John Doe',
      password,
      posts: {
        create: [
          { title: 'First Post', content: 'Hello World', published: true },
          { title: 'Second Post', content: 'More content', published: false },
        ],
      },
    },
  })

  console.log('Seeded database')
}

main()
  .catch(console.error)
  .finally(() => prisma.$disconnect())
```

```json
// package.json
{
  "prisma": {
    "seed": "tsx prisma/seed.ts"
  }
}
```

---

## Week 3: Advanced Database Topics

### Day 1: Database Design
**Topics:**
- Normalization (1NF, 2NF, 3NF)
- Denormalization when appropriate
- Schema design patterns
- Soft deletes pattern
- Audit trails

### Day 2: MongoDB Basics
**Topics:**
- Document databases
- When to use NoSQL
- MongoDB setup
- CRUD operations
- Mongoose ODM basics

```typescript
// Basic Mongoose setup
import mongoose from 'mongoose'

const userSchema = new mongoose.Schema({
  email: { type: String, required: true, unique: true },
  name: { type: String, required: true },
  posts: [{ type: mongoose.Schema.Types.ObjectId, ref: 'Post' }],
}, { timestamps: true })

const User = mongoose.model('User', userSchema)
```

### Day 3: Redis Basics
**Topics:**
- What is Redis?
- Data structures (strings, lists, sets, hashes)
- Caching patterns
- Session storage
- Pub/Sub basics

```typescript
import { createClient } from 'redis'

const redis = createClient({ url: process.env.REDIS_URL })
await redis.connect()

// String caching
await redis.set('user:123', JSON.stringify(userData), { EX: 3600 })
const cached = await redis.get('user:123')

// Hash
await redis.hSet('user:123', { name: 'John', email: 'john@example.com' })
const user = await redis.hGetAll('user:123')
```

### Day 4-5: Project Work
**Build: Complete Database Layer for E-commerce**
- Users, Products, Orders, OrderItems, Categories
- Complex relationships
- Full Prisma setup
- Seeding script
- Redis caching for products
- Pagination on all list endpoints

---

## Database Schema Patterns

### User Authentication Pattern
```prisma
model User {
  id            String    @id @default(uuid())
  email         String    @unique
  password      String
  emailVerified Boolean   @default(false)
  role          Role      @default(USER)
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  sessions      Session[]
}

model Session {
  id        String   @id @default(uuid())
  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  token     String   @unique
  expiresAt DateTime
  createdAt DateTime @default(now())
}

enum Role {
  USER
  ADMIN
}
```

### Soft Delete Pattern
```prisma
model Post {
  id        String    @id @default(uuid())
  title     String
  deletedAt DateTime?
  // ... other fields

  @@index([deletedAt])
}

// Usage
const posts = await prisma.post.findMany({
  where: { deletedAt: null }
})

// Soft delete
await prisma.post.update({
  where: { id },
  data: { deletedAt: new Date() }
})
```

---

## Resources

### Documentation
- [PostgreSQL Docs](https://www.postgresql.org/docs/)
- [Prisma Docs](https://www.prisma.io/docs)
- [MongoDB Docs](https://www.mongodb.com/docs/)

### Practice
- [SQLBolt](https://sqlbolt.com/)
- [PostgreSQL Exercises](https://pgexercises.com/)

---

## Checklist Before Moving On

- [ ] Can write complex SQL queries
- [ ] Understand database relationships
- [ ] Can design normalized schemas
- [ ] Comfortable with Prisma ORM
- [ ] Understand indexes and performance
- [ ] Know when to use SQL vs NoSQL
- [ ] Completed E-commerce database project

---

**Next:** [Authentication & Security](../03-auth-security/README.md)
