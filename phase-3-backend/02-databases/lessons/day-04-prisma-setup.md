# Day 4: Prisma Setup

## Introduction

Prisma is a modern database toolkit that makes working with databases type-safe and developer-friendly. It includes an ORM (Object-Relational Mapper), a migration system, and a powerful query builder. This lesson covers installing Prisma, defining schemas, and running your first migrations.

## Learning Objectives

By the end of this lesson, you will:
- Understand what Prisma is and its components
- Set up Prisma in a Node.js/TypeScript project
- Define database models in the Prisma schema
- Run migrations to create database tables
- Use Prisma Studio for database exploration
- Configure Prisma Client for your application

---

## What is Prisma?

### Prisma Components

```
┌─────────────────────────────────────────────────────────────────┐
│                         Prisma                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │ Prisma       │  │ Prisma       │  │ Prisma       │          │
│  │ Client       │  │ Migrate      │  │ Studio       │          │
│  │              │  │              │  │              │          │
│  │ Type-safe    │  │ Database     │  │ Visual DB    │          │
│  │ query        │  │ migrations   │  │ browser      │          │
│  │ builder      │  │ & schema     │  │ & editor     │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Prisma vs Other ORMs

```
┌──────────────────┬─────────────┬─────────────┬─────────────────┐
│ Feature          │ Prisma      │ TypeORM     │ Sequelize       │
├──────────────────┼─────────────┼─────────────┼─────────────────┤
│ Type Safety      │ Excellent   │ Good        │ Limited         │
│ Auto-completion  │ Full        │ Partial     │ Limited         │
│ Schema           │ Prisma DSL  │ Decorators  │ JavaScript      │
│ Migrations       │ Built-in    │ Built-in    │ Built-in        │
│ Learning Curve   │ Low         │ Medium      │ Medium          │
│ Performance      │ Excellent   │ Good        │ Good            │
│ Database Support │ Many        │ Many        │ Many            │
└──────────────────┴─────────────┴─────────────┴─────────────────┘
```

### Supported Databases

- **PostgreSQL** (recommended)
- **MySQL**
- **SQLite**
- **SQL Server**
- **MongoDB**
- **CockroachDB**

---

## Project Setup

### Create a New Project

```bash
# Create project directory
mkdir prisma-blog && cd prisma-blog

# Initialize Node.js project
npm init -y

# Install dependencies
npm install typescript ts-node @types/node --save-dev
npm install prisma --save-dev
npm install @prisma/client

# Initialize TypeScript
npx tsc --init
```

### Configure TypeScript

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### Initialize Prisma

```bash
# Initialize Prisma with PostgreSQL
npx prisma init --datasource-provider postgresql

# Or with SQLite (easier for learning)
npx prisma init --datasource-provider sqlite
```

This creates:
- `prisma/schema.prisma` - Your database schema
- `.env` - Environment variables

### Project Structure

```
prisma-blog/
├── prisma/
│   └── schema.prisma      # Database schema
├── src/
│   ├── index.ts           # Application entry
│   └── lib/
│       └── prisma.ts      # Prisma client instance
├── .env                   # Environment variables
├── package.json
└── tsconfig.json
```

---

## Database Connection

### Environment Variables

```bash
# .env for PostgreSQL
DATABASE_URL="postgresql://user:password@localhost:5432/blog_db?schema=public"

# .env for SQLite (simpler for development)
DATABASE_URL="file:./dev.db"

# .env for MySQL
DATABASE_URL="mysql://user:password@localhost:3306/blog_db"
```

### Connection String Format

```
postgresql://USER:PASSWORD@HOST:PORT/DATABASE?schema=SCHEMA

Examples:
# Local PostgreSQL
postgresql://postgres:password@localhost:5432/mydb?schema=public

# Docker PostgreSQL
postgresql://postgres:postgres@localhost:5432/mydb

# Remote (e.g., Supabase, Neon, Railway)
postgresql://user:pass@db.example.com:5432/mydb?sslmode=require
```

---

## Prisma Schema

### Schema File Structure

```prisma
// prisma/schema.prisma

// 1. Generator - Generates Prisma Client
generator client {
  provider = "prisma-client-js"
}

// 2. Datasource - Database connection
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// 3. Models - Your database tables
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  posts     Post[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String?
  published Boolean  @default(false)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  Int
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

### Field Types

```prisma
model Example {
  // Scalar types
  id          Int       @id @default(autoincrement())
  bigId       BigInt    @id @default(autoincrement())
  uuid        String    @id @default(uuid())
  cuid        String    @id @default(cuid())

  // String
  name        String              // VARCHAR(191) by default
  bio         String?             // Nullable
  email       String    @unique   // Unique constraint
  content     String    @db.Text  // TEXT type

  // Numbers
  age         Int
  price       Float
  amount      Decimal   @db.Decimal(10, 2)

  // Boolean
  isActive    Boolean   @default(true)

  // DateTime
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
  publishedAt DateTime?

  // JSON (PostgreSQL, MySQL)
  metadata    Json?

  // Enum
  role        Role      @default(USER)

  // Bytes (for binary data)
  avatar      Bytes?
}

enum Role {
  USER
  ADMIN
  MODERATOR
}
```

### Field Attributes

```prisma
model User {
  // @id - Primary key
  id Int @id @default(autoincrement())

  // @unique - Unique constraint
  email String @unique

  // @default - Default value
  role      String   @default("user")
  isActive  Boolean  @default(true)
  createdAt DateTime @default(now())
  uuid      String   @default(uuid())
  cuid      String   @default(cuid())

  // @updatedAt - Auto-update timestamp
  updatedAt DateTime @updatedAt

  // @map - Map to different column name
  firstName String @map("first_name")

  // @db - Database-specific type
  bio String? @db.Text
  amount Decimal @db.Decimal(10, 2)

  // @ignore - Ignore field in Prisma Client
  internalField String? @ignore

  // @@map - Map to different table name
  @@map("users")
}
```

### Model Attributes

```prisma
model Post {
  id       Int    @id @default(autoincrement())
  title    String
  authorId Int

  // Compound unique constraint
  @@unique([title, authorId])

  // Index for faster queries
  @@index([authorId])
  @@index([title, authorId])

  // Compound primary key
  // @@id([authorId, title])

  // Map to different table name
  @@map("blog_posts")
}
```

---

## Complete Blog Schema

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// User model
model User {
  id        Int       @id @default(autoincrement())
  email     String    @unique
  name      String
  password  String
  bio       String?   @db.Text
  avatar    String?
  role      Role      @default(USER)
  isActive  Boolean   @default(true)

  // Relations
  posts     Post[]
  comments  Comment[]
  likes     Like[]

  // Timestamps
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt

  @@map("users")
}

// Post model
model Post {
  id          Int       @id @default(autoincrement())
  title       String    @db.VarChar(200)
  slug        String    @unique
  content     String    @db.Text
  excerpt     String?   @db.VarChar(500)
  coverImage  String?
  published   Boolean   @default(false)
  publishedAt DateTime?
  views       Int       @default(0)

  // Relations
  author      User      @relation(fields: [authorId], references: [id], onDelete: Cascade)
  authorId    Int
  category    Category? @relation(fields: [categoryId], references: [id])
  categoryId  Int?
  tags        Tag[]
  comments    Comment[]
  likes       Like[]

  // Timestamps
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt

  // Indexes
  @@index([authorId])
  @@index([categoryId])
  @@index([published, publishedAt])
  @@map("posts")
}

// Category model
model Category {
  id          Int      @id @default(autoincrement())
  name        String   @unique
  slug        String   @unique
  description String?

  // Relations
  posts       Post[]

  @@map("categories")
}

// Tag model (many-to-many with Post)
model Tag {
  id    Int    @id @default(autoincrement())
  name  String @unique
  slug  String @unique

  // Relations
  posts Post[]

  @@map("tags")
}

// Comment model (self-referential for replies)
model Comment {
  id        Int       @id @default(autoincrement())
  content   String    @db.Text

  // Relations
  post      Post      @relation(fields: [postId], references: [id], onDelete: Cascade)
  postId    Int
  author    User      @relation(fields: [authorId], references: [id], onDelete: Cascade)
  authorId  Int

  // Self-referential (replies)
  parent    Comment?  @relation("CommentReplies", fields: [parentId], references: [id])
  parentId  Int?
  replies   Comment[] @relation("CommentReplies")

  // Timestamps
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt

  @@index([postId])
  @@index([authorId])
  @@map("comments")
}

// Like model (many-to-many junction)
model Like {
  id        Int      @id @default(autoincrement())
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  userId    Int
  post      Post     @relation(fields: [postId], references: [id], onDelete: Cascade)
  postId    Int
  createdAt DateTime @default(now())

  // Unique constraint (user can only like a post once)
  @@unique([userId, postId])
  @@map("likes")
}

// Enums
enum Role {
  USER
  ADMIN
  AUTHOR
}
```

---

## Running Migrations

### Create and Apply Migration

```bash
# Create migration from schema changes
npx prisma migrate dev --name init

# This will:
# 1. Create SQL migration file in prisma/migrations/
# 2. Apply migration to database
# 3. Generate Prisma Client
```

### Migration Commands

```bash
# Create and apply migration (development)
npx prisma migrate dev --name add_user_role

# Apply pending migrations (production)
npx prisma migrate deploy

# Reset database (delete all data, re-run migrations)
npx prisma migrate reset

# Generate migration without applying
npx prisma migrate dev --create-only

# Check migration status
npx prisma migrate status

# Mark migration as applied (without running)
npx prisma migrate resolve --applied "migration_name"
```

### Migration File Example

```sql
-- prisma/migrations/20240115000000_init/migration.sql

-- CreateEnum
CREATE TYPE "Role" AS ENUM ('USER', 'ADMIN', 'AUTHOR');

-- CreateTable
CREATE TABLE "users" (
    "id" SERIAL NOT NULL,
    "email" TEXT NOT NULL,
    "name" TEXT NOT NULL,
    "password" TEXT NOT NULL,
    "bio" TEXT,
    "avatar" TEXT,
    "role" "Role" NOT NULL DEFAULT 'USER',
    "isActive" BOOLEAN NOT NULL DEFAULT true,
    "createdAt" TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP,
    "updatedAt" TIMESTAMP(3) NOT NULL,

    CONSTRAINT "users_pkey" PRIMARY KEY ("id")
);

-- CreateTable
CREATE TABLE "posts" (
    "id" SERIAL NOT NULL,
    "title" VARCHAR(200) NOT NULL,
    "slug" TEXT NOT NULL,
    "content" TEXT NOT NULL,
    ...
);

-- CreateIndex
CREATE UNIQUE INDEX "users_email_key" ON "users"("email");
```

---

## Prisma Client Setup

### Create Prisma Client Instance

```typescript
// src/lib/prisma.ts
import { PrismaClient } from '@prisma/client';

// Prevent multiple instances in development (hot reload)
const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: ['query', 'info', 'warn', 'error'],
  });

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}

export default prisma;
```

### Prisma Client Options

```typescript
// With logging
const prisma = new PrismaClient({
  log: [
    { level: 'query', emit: 'event' },
    { level: 'info', emit: 'stdout' },
    { level: 'warn', emit: 'stdout' },
    { level: 'error', emit: 'stdout' },
  ],
});

// Listen to query events
prisma.$on('query', (e) => {
  console.log('Query:', e.query);
  console.log('Params:', e.params);
  console.log('Duration:', e.duration + 'ms');
});

// With error formatting
const prisma = new PrismaClient({
  errorFormat: 'pretty', // 'minimal' | 'colorless' | 'pretty'
});
```

### Generate Prisma Client

```bash
# Generate client (run after schema changes)
npx prisma generate

# The client is generated to:
# node_modules/.prisma/client/
# node_modules/@prisma/client/
```

---

## Prisma Studio

### Launch Studio

```bash
# Open Prisma Studio (visual database browser)
npx prisma studio

# Opens at http://localhost:5555
```

### Features

- Browse all tables and data
- Add, edit, delete records
- Filter and sort data
- View relations
- Export data

---

## Other Prisma Commands

### Database Commands

```bash
# Push schema to database (without migrations - good for prototyping)
npx prisma db push

# Pull schema from existing database
npx prisma db pull

# Seed database
npx prisma db seed

# Execute raw SQL
npx prisma db execute --file ./script.sql
```

### Schema Validation

```bash
# Validate schema file
npx prisma validate

# Format schema file
npx prisma format
```

### Introspection

```bash
# Generate schema from existing database
npx prisma db pull

# This is useful when:
# - Working with existing databases
# - Migrating from another ORM
# - Database-first development
```

---

## Database Seeding

### Setup Seed Script

```json
// package.json
{
  "prisma": {
    "seed": "ts-node prisma/seed.ts"
  }
}
```

### Create Seed File

```typescript
// prisma/seed.ts
import { PrismaClient, Role } from '@prisma/client';

const prisma = new PrismaClient();

async function main() {
  console.log('Start seeding...');

  // Clean existing data
  await prisma.like.deleteMany();
  await prisma.comment.deleteMany();
  await prisma.post.deleteMany();
  await prisma.tag.deleteMany();
  await prisma.category.deleteMany();
  await prisma.user.deleteMany();

  // Create categories
  const categories = await Promise.all([
    prisma.category.create({
      data: { name: 'Technology', slug: 'technology', description: 'Tech articles' },
    }),
    prisma.category.create({
      data: { name: 'Lifestyle', slug: 'lifestyle', description: 'Lifestyle content' },
    }),
    prisma.category.create({
      data: { name: 'Travel', slug: 'travel', description: 'Travel stories' },
    }),
  ]);

  // Create tags
  const tags = await Promise.all([
    prisma.tag.create({ data: { name: 'JavaScript', slug: 'javascript' } }),
    prisma.tag.create({ data: { name: 'TypeScript', slug: 'typescript' } }),
    prisma.tag.create({ data: { name: 'React', slug: 'react' } }),
    prisma.tag.create({ data: { name: 'Node.js', slug: 'nodejs' } }),
  ]);

  // Create users
  const admin = await prisma.user.create({
    data: {
      email: 'admin@example.com',
      name: 'Admin User',
      password: 'hashedpassword123', // Use bcrypt in real app
      role: Role.ADMIN,
      bio: 'Site administrator',
    },
  });

  const author = await prisma.user.create({
    data: {
      email: 'author@example.com',
      name: 'Jane Author',
      password: 'hashedpassword456',
      role: Role.AUTHOR,
      bio: 'Technical writer and developer',
    },
  });

  const user = await prisma.user.create({
    data: {
      email: 'user@example.com',
      name: 'John User',
      password: 'hashedpassword789',
      role: Role.USER,
    },
  });

  // Create posts
  const post1 = await prisma.post.create({
    data: {
      title: 'Getting Started with Prisma',
      slug: 'getting-started-with-prisma',
      content: 'Prisma is a modern database toolkit...',
      excerpt: 'Learn how to use Prisma with Node.js and TypeScript.',
      published: true,
      publishedAt: new Date(),
      authorId: author.id,
      categoryId: categories[0].id,
      tags: {
        connect: [{ id: tags[1].id }, { id: tags[3].id }], // TypeScript, Node.js
      },
    },
  });

  const post2 = await prisma.post.create({
    data: {
      title: 'Building APIs with Express',
      slug: 'building-apis-with-express',
      content: 'Express.js is a minimal and flexible...',
      excerpt: 'Create RESTful APIs with Express and Prisma.',
      published: true,
      publishedAt: new Date(),
      authorId: author.id,
      categoryId: categories[0].id,
      tags: {
        connect: [{ id: tags[0].id }, { id: tags[3].id }], // JavaScript, Node.js
      },
    },
  });

  // Create comments
  await prisma.comment.create({
    data: {
      content: 'Great article! Very helpful.',
      postId: post1.id,
      authorId: user.id,
    },
  });

  await prisma.comment.create({
    data: {
      content: 'Thanks for this detailed guide.',
      postId: post1.id,
      authorId: admin.id,
    },
  });

  // Create likes
  await prisma.like.createMany({
    data: [
      { userId: user.id, postId: post1.id },
      { userId: admin.id, postId: post1.id },
      { userId: user.id, postId: post2.id },
    ],
  });

  console.log('Seeding finished.');
}

main()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

### Run Seed

```bash
# Run seed script
npx prisma db seed

# Seed runs automatically after:
npx prisma migrate reset
npx prisma migrate dev (if seed exists)
```

---

## Testing Your Setup

### Basic Test Script

```typescript
// src/index.ts
import prisma from './lib/prisma';

async function main() {
  // Test connection
  await prisma.$connect();
  console.log('Connected to database');

  // Count users
  const userCount = await prisma.user.count();
  console.log(`Total users: ${userCount}`);

  // Get all users with their posts
  const users = await prisma.user.findMany({
    include: {
      posts: true,
    },
  });

  console.log('Users:', JSON.stringify(users, null, 2));
}

main()
  .catch(console.error)
  .finally(() => prisma.$disconnect());
```

### Run the Test

```bash
# Run with ts-node
npx ts-node src/index.ts

# Or add script to package.json
{
  "scripts": {
    "dev": "ts-node src/index.ts"
  }
}

npm run dev
```

---

## Key Takeaways

1. **Prisma Schema is the source of truth** - Define models, relations, and constraints
2. **Migrations track changes** - Version control for your database
3. **Prisma Client is type-safe** - Full autocomplete and type checking
4. **Use a singleton pattern** - Avoid multiple Prisma Client instances
5. **Seed your database** - Start with consistent test data
6. **Prisma Studio helps visualization** - Browse and edit data visually

---

## What's Next?

Tomorrow we'll dive into **Prisma CRUD Operations** - learning how to create, read, update, and delete data with Prisma's powerful query API!
