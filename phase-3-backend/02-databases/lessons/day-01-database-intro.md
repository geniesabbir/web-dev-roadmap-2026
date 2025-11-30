# Day 1: Introduction to Databases

## Introduction

Databases are the backbone of almost every web application. They store, organize, and manage data so your application can retrieve and manipulate it efficiently. This lesson introduces database concepts, compares different database types, and helps you understand when to use each.

## Learning Objectives

By the end of this lesson, you will:
- Understand what databases are and why they're essential
- Know the difference between SQL and NoSQL databases
- Understand basic database terminology
- Learn about ACID properties and data integrity
- Set up PostgreSQL for local development
- Connect to a database from Node.js

---

## What is a Database?

### Definition

A database is an organized collection of structured data stored electronically. A Database Management System (DBMS) is software that manages the database and provides:

- **Data storage** - Efficient storage of large amounts of data
- **Data retrieval** - Fast querying and fetching of data
- **Data manipulation** - Insert, update, delete operations
- **Data security** - Access control and encryption
- **Data integrity** - Ensuring data accuracy and consistency
- **Concurrent access** - Multiple users accessing data simultaneously

### Why Not Just Use Files?

```
Files vs Databases
─────────────────────────────────────────────────────
Feature          │ Files           │ Databases
─────────────────────────────────────────────────────
Query Speed      │ Slow (scan all) │ Fast (indexes)
Concurrent Access│ Problematic     │ Built-in
Data Integrity   │ Manual          │ Automatic
Relationships    │ Complex         │ Native support
Backup/Recovery  │ Manual          │ Built-in
Scalability      │ Limited         │ Horizontal/Vertical
─────────────────────────────────────────────────────
```

---

## Database Types

### Relational Databases (SQL)

Store data in structured tables with rows and columns. Use SQL (Structured Query Language) for querying.

```
┌─────────────────────────────────────────────────────┐
│                    users table                       │
├────────┬───────────┬──────────────────┬────────────┤
│ id     │ name      │ email            │ created_at │
├────────┼───────────┼──────────────────┼────────────┤
│ 1      │ John Doe  │ john@example.com │ 2024-01-15 │
│ 2      │ Jane Doe  │ jane@example.com │ 2024-01-16 │
└────────┴───────────┴──────────────────┴────────────┘
         │
         │ Foreign Key Relationship
         ▼
┌─────────────────────────────────────────────────────┐
│                    posts table                       │
├────────┬────────────────────┬───────────┬──────────┤
│ id     │ title              │ user_id   │ content  │
├────────┼────────────────────┼───────────┼──────────┤
│ 1      │ Hello World        │ 1         │ ...      │
│ 2      │ My Second Post     │ 1         │ ...      │
│ 3      │ Jane's Post        │ 2         │ ...      │
└────────┴────────────────────┴───────────┴──────────┘
```

**Popular SQL Databases:**
- **PostgreSQL** - Feature-rich, open source, great for complex queries
- **MySQL** - Popular, widely supported, good for web applications
- **SQLite** - File-based, embedded, perfect for local/mobile apps
- **Microsoft SQL Server** - Enterprise, Windows ecosystem
- **Oracle** - Enterprise, high performance

**Best For:**
- Complex relationships between data
- Transactions requiring ACID compliance
- Structured, consistent data
- Complex queries and reporting
- Financial applications

### NoSQL Databases

Store data in flexible, non-tabular formats. Different types for different use cases.

#### Document Databases (MongoDB)

Store data as JSON-like documents:

```javascript
// MongoDB Document
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "name": "John Doe",
  "email": "john@example.com",
  "posts": [
    {
      "title": "Hello World",
      "content": "My first post...",
      "tags": ["intro", "welcome"],
      "comments": [
        { "user": "Jane", "text": "Nice post!" }
      ]
    }
  ],
  "preferences": {
    "theme": "dark",
    "notifications": true
  }
}
```

**Best For:**
- Flexible, evolving schemas
- Hierarchical data
- Content management
- Real-time analytics
- Rapid development

#### Key-Value Stores (Redis)

Simple key-value pairs, extremely fast:

```
Key                          Value
────────────────────────────────────────────
user:1:name                  "John Doe"
user:1:email                 "john@example.com"
session:abc123               {"userId": 1, "expires": "..."}
cache:posts:recent           "[{...}, {...}, {...}]"
```

**Best For:**
- Caching
- Session management
- Real-time leaderboards
- Pub/sub messaging
- Rate limiting

#### Wide-Column Stores (Cassandra)

Optimized for large-scale distributed data:

```
Row Key    │ Column Family: profile
───────────┼──────────────────────────────────
user:1     │ name: "John"  │ email: "j@e.com"  │ age: 30
user:2     │ name: "Jane"  │ city: "NYC"       │
```

**Best For:**
- Time-series data
- IoT applications
- High write throughput
- Geographic distribution

#### Graph Databases (Neo4j)

Store relationships as first-class citizens:

```
    ┌─────────┐         ┌─────────┐
    │  John   │─FOLLOWS─▶│  Jane   │
    └─────────┘         └─────────┘
         │                   │
      WROTE              LIKES
         │                   │
         ▼                   ▼
    ┌─────────┐         ┌─────────┐
    │  Post1  │◀─LIKES──│  Post2  │
    └─────────┘         └─────────┘
```

**Best For:**
- Social networks
- Recommendation engines
- Fraud detection
- Knowledge graphs

---

## SQL vs NoSQL Comparison

```
┌──────────────────────────────────────────────────────────────┐
│              SQL vs NoSQL Comparison                          │
├────────────────────┬─────────────────┬───────────────────────┤
│ Aspect             │ SQL             │ NoSQL                 │
├────────────────────┼─────────────────┼───────────────────────┤
│ Schema             │ Fixed, strict   │ Flexible, dynamic     │
│ Relationships      │ Joins           │ Embedded/References   │
│ Scaling            │ Vertical        │ Horizontal            │
│ ACID               │ Full support    │ Varies                │
│ Query Language     │ SQL (standard)  │ Varies by DB          │
│ Data Model         │ Tables          │ Documents/Key-Value   │
│ Best For           │ Complex queries │ Simple queries at     │
│                    │ Transactions    │ massive scale         │
└────────────────────┴─────────────────┴───────────────────────┘
```

### When to Use SQL

- Data has clear relationships
- Need complex queries with JOINs
- Require transactions (banking, inventory)
- Data structure is known and stable
- Need strong consistency

### When to Use NoSQL

- Flexible or evolving schema
- High velocity data ingestion
- Horizontal scaling is priority
- Simple query patterns
- Real-time applications

### Modern Approach: Polyglot Persistence

Many applications use multiple databases:

```
┌─────────────────────────────────────────────────────────┐
│                    Application                           │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐           │
│  │PostgreSQL │  │  MongoDB  │  │   Redis   │           │
│  │           │  │           │  │           │           │
│  │ Users     │  │ Posts     │  │ Cache     │           │
│  │ Orders    │  │ Comments  │  │ Sessions  │           │
│  │ Payments  │  │ Logs      │  │ Real-time │           │
│  └───────────┘  └───────────┘  └───────────┘           │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## Database Terminology

### Common Terms

| Term | Definition |
|------|------------|
| **Table/Collection** | A set of related data (like a spreadsheet) |
| **Row/Document** | A single record in a table |
| **Column/Field** | A single attribute of a record |
| **Primary Key** | Unique identifier for each row |
| **Foreign Key** | Reference to a primary key in another table |
| **Index** | Data structure for faster queries |
| **Schema** | Structure/blueprint of the database |
| **Query** | Request to retrieve or modify data |
| **Transaction** | Group of operations that succeed or fail together |

### Primary Keys

Every table needs a way to uniquely identify each row:

```sql
-- Auto-incrementing integer (traditional)
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100)
);

-- UUID (distributed systems)
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(100)
);
```

**Integer IDs:**
- Pros: Small, fast, sequential
- Cons: Predictable, limited range, distributed ID generation complex

**UUIDs:**
- Pros: Globally unique, no coordination needed
- Cons: Larger, slower indexes, not sequential

### Indexes

Indexes speed up queries but slow down writes:

```sql
-- Without index: Full table scan O(n)
SELECT * FROM users WHERE email = 'john@example.com';

-- With index: B-tree lookup O(log n)
CREATE INDEX idx_users_email ON users(email);
```

```
Without Index          With Index (B-tree)
──────────────         ─────────────────────
┌─────────────┐              [M]
│ Scan all    │             /   \
│ 1000 rows   │          [D]     [T]
│ to find     │         /   \   /   \
│ match       │       [A]  [H] [P]  [Z]
└─────────────┘        ↓
                    Direct lookup
```

---

## ACID Properties

ACID ensures reliable database transactions:

### Atomicity

All operations in a transaction succeed, or none do:

```sql
-- Transfer $100 from Account A to Account B
BEGIN TRANSACTION;
  UPDATE accounts SET balance = balance - 100 WHERE id = 'A';
  UPDATE accounts SET balance = balance + 100 WHERE id = 'B';
COMMIT;
-- If either fails, both are rolled back
```

### Consistency

Database moves from one valid state to another:

```sql
-- Constraint ensures balance never goes negative
ALTER TABLE accounts ADD CONSTRAINT positive_balance CHECK (balance >= 0);

-- This will fail if balance would become negative
UPDATE accounts SET balance = balance - 1000 WHERE id = 'A';
```

### Isolation

Concurrent transactions don't interfere:

```
Transaction A          Transaction B
─────────────          ─────────────
Read balance: $100
                       Read balance: $100
Deduct $50
                       Deduct $30
Write balance: $50
                       Write balance: $70  ← Wrong! Should be $20

With Isolation:
Transaction A          Transaction B
─────────────          ─────────────
Lock row
Read balance: $100
Deduct $50
Write balance: $50
Unlock row
                       Lock row
                       Read balance: $50
                       Deduct $30
                       Write balance: $20  ← Correct!
                       Unlock row
```

### Durability

Committed transactions persist even after system failure:

```
1. Transaction commits
2. Changes written to transaction log
3. Log flushed to disk
4. Even if server crashes, data recovers from log
```

---

## Setting Up PostgreSQL

### Installation

```bash
# macOS with Homebrew
brew install postgresql@16
brew services start postgresql@16

# Ubuntu/Debian
sudo apt update
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql

# Windows
# Download installer from https://www.postgresql.org/download/windows/

# Docker (recommended for development)
docker run --name postgres \
  -e POSTGRES_PASSWORD=password \
  -e POSTGRES_DB=myapp \
  -p 5432:5432 \
  -d postgres:16
```

### Basic PostgreSQL Commands

```bash
# Connect to PostgreSQL
psql -U postgres

# Create a database
createdb myapp

# Connect to a database
psql -d myapp

# Common psql commands
\l          # List databases
\c dbname   # Connect to database
\dt         # List tables
\d table    # Describe table
\q          # Quit
```

### Create Your First Database

```sql
-- Connect as postgres user
psql -U postgres

-- Create a new database
CREATE DATABASE blog_db;

-- Connect to it
\c blog_db

-- Create a users table
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create a posts table
CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  title VARCHAR(200) NOT NULL,
  content TEXT,
  user_id INTEGER REFERENCES users(id),
  published BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert sample data
INSERT INTO users (name, email, password_hash) VALUES
  ('John Doe', 'john@example.com', 'hash1'),
  ('Jane Doe', 'jane@example.com', 'hash2');

INSERT INTO posts (title, content, user_id, published) VALUES
  ('Hello World', 'My first post content', 1, true),
  ('Draft Post', 'Work in progress...', 1, false),
  ('Jane''s Post', 'Another post here', 2, true);

-- Query the data
SELECT * FROM users;
SELECT * FROM posts WHERE published = true;

-- Join query
SELECT posts.title, users.name as author
FROM posts
JOIN users ON posts.user_id = users.id
WHERE posts.published = true;
```

---

## Connecting from Node.js

### Using pg (node-postgres)

```bash
npm install pg
```

```javascript
// db.js
const { Pool } = require('pg');

const pool = new Pool({
  host: process.env.DB_HOST || 'localhost',
  port: process.env.DB_PORT || 5432,
  database: process.env.DB_NAME || 'blog_db',
  user: process.env.DB_USER || 'postgres',
  password: process.env.DB_PASSWORD || 'password',
  max: 20, // Maximum connections in pool
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

// Test connection
pool.query('SELECT NOW()', (err, res) => {
  if (err) {
    console.error('Database connection failed:', err.message);
  } else {
    console.log('Database connected:', res.rows[0].now);
  }
});

module.exports = pool;
```

### Basic Queries

```javascript
const pool = require('./db');

// Simple query
async function getUsers() {
  const result = await pool.query('SELECT * FROM users');
  return result.rows;
}

// Parameterized query (prevents SQL injection!)
async function getUserById(id) {
  const result = await pool.query(
    'SELECT * FROM users WHERE id = $1',
    [id]
  );
  return result.rows[0];
}

// Insert and return
async function createUser(name, email, passwordHash) {
  const result = await pool.query(
    `INSERT INTO users (name, email, password_hash)
     VALUES ($1, $2, $3)
     RETURNING *`,
    [name, email, passwordHash]
  );
  return result.rows[0];
}

// Update
async function updateUser(id, name) {
  const result = await pool.query(
    `UPDATE users SET name = $1, updated_at = NOW()
     WHERE id = $2
     RETURNING *`,
    [name, id]
  );
  return result.rows[0];
}

// Delete
async function deleteUser(id) {
  const result = await pool.query(
    'DELETE FROM users WHERE id = $1 RETURNING *',
    [id]
  );
  return result.rows[0];
}

// Transaction
async function transferFunds(fromId, toId, amount) {
  const client = await pool.connect();

  try {
    await client.query('BEGIN');

    await client.query(
      'UPDATE accounts SET balance = balance - $1 WHERE id = $2',
      [amount, fromId]
    );

    await client.query(
      'UPDATE accounts SET balance = balance + $1 WHERE id = $2',
      [amount, toId]
    );

    await client.query('COMMIT');
  } catch (err) {
    await client.query('ROLLBACK');
    throw err;
  } finally {
    client.release();
  }
}
```

### Express Integration

```javascript
// routes/users.js
const express = require('express');
const router = express.Router();
const pool = require('../db');

// Get all users
router.get('/', async (req, res, next) => {
  try {
    const result = await pool.query(
      'SELECT id, name, email, created_at FROM users ORDER BY created_at DESC'
    );
    res.json(result.rows);
  } catch (err) {
    next(err);
  }
});

// Get user by ID
router.get('/:id', async (req, res, next) => {
  try {
    const { id } = req.params;
    const result = await pool.query(
      'SELECT id, name, email, created_at FROM users WHERE id = $1',
      [id]
    );

    if (result.rows.length === 0) {
      return res.status(404).json({ error: 'User not found' });
    }

    res.json(result.rows[0]);
  } catch (err) {
    next(err);
  }
});

// Create user
router.post('/', async (req, res, next) => {
  try {
    const { name, email, password } = req.body;

    // Hash password (use bcrypt in real app)
    const passwordHash = 'hashed_' + password;

    const result = await pool.query(
      `INSERT INTO users (name, email, password_hash)
       VALUES ($1, $2, $3)
       RETURNING id, name, email, created_at`,
      [name, email, passwordHash]
    );

    res.status(201).json(result.rows[0]);
  } catch (err) {
    if (err.code === '23505') { // Unique violation
      return res.status(409).json({ error: 'Email already exists' });
    }
    next(err);
  }
});

module.exports = router;
```

---

## SQL Injection Prevention

**NEVER** concatenate user input into SQL queries:

```javascript
// DANGEROUS - SQL Injection vulnerability!
const query = `SELECT * FROM users WHERE email = '${email}'`;
// Attacker input: ' OR '1'='1
// Result: SELECT * FROM users WHERE email = '' OR '1'='1'
// Returns ALL users!

// SAFE - Use parameterized queries
const result = await pool.query(
  'SELECT * FROM users WHERE email = $1',
  [email]
);
```

---

## Exercise: Build a Basic Data Layer

Create a complete data access layer for a todo application:

```javascript
// models/todo.js
const pool = require('../db');

class TodoModel {
  static async create(userId, title, description = null) {
    const result = await pool.query(
      `INSERT INTO todos (user_id, title, description)
       VALUES ($1, $2, $3)
       RETURNING *`,
      [userId, title, description]
    );
    return result.rows[0];
  }

  static async findByUser(userId, { completed, limit = 50, offset = 0 } = {}) {
    let query = 'SELECT * FROM todos WHERE user_id = $1';
    const params = [userId];

    if (completed !== undefined) {
      query += ' AND completed = $2';
      params.push(completed);
    }

    query += ' ORDER BY created_at DESC LIMIT $' + (params.length + 1);
    params.push(limit);

    query += ' OFFSET $' + (params.length + 1);
    params.push(offset);

    const result = await pool.query(query, params);
    return result.rows;
  }

  static async findById(id, userId) {
    const result = await pool.query(
      'SELECT * FROM todos WHERE id = $1 AND user_id = $2',
      [id, userId]
    );
    return result.rows[0];
  }

  static async update(id, userId, updates) {
    const { title, description, completed } = updates;

    const result = await pool.query(
      `UPDATE todos
       SET title = COALESCE($1, title),
           description = COALESCE($2, description),
           completed = COALESCE($3, completed),
           updated_at = NOW()
       WHERE id = $4 AND user_id = $5
       RETURNING *`,
      [title, description, completed, id, userId]
    );
    return result.rows[0];
  }

  static async delete(id, userId) {
    const result = await pool.query(
      'DELETE FROM todos WHERE id = $1 AND user_id = $2 RETURNING *',
      [id, userId]
    );
    return result.rows[0];
  }

  static async getStats(userId) {
    const result = await pool.query(
      `SELECT
         COUNT(*) as total,
         COUNT(*) FILTER (WHERE completed = true) as completed,
         COUNT(*) FILTER (WHERE completed = false) as pending
       FROM todos
       WHERE user_id = $1`,
      [userId]
    );
    return result.rows[0];
  }
}

module.exports = TodoModel;
```

---

## Key Takeaways

1. **Databases are essential** - They provide structured, reliable data storage
2. **SQL for relationships** - Use PostgreSQL/MySQL for complex, related data
3. **NoSQL for flexibility** - Use MongoDB/Redis for specific use cases
4. **ACID matters** - Understand when you need transaction guarantees
5. **Always parameterize** - Never concatenate user input into queries
6. **Indexes optimize reads** - But slow down writes
7. **Connection pooling** - Reuse database connections for performance

---

## What's Next?

Tomorrow we'll dive deep into **SQL Basics** - learning SELECT, INSERT, UPDATE, DELETE, and how to write efficient queries with WHERE, ORDER BY, GROUP BY, and more!
