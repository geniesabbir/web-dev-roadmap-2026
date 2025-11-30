# Day 2: SQL Basics

## Introduction

SQL (Structured Query Language) is the standard language for interacting with relational databases. This lesson covers the fundamental SQL operations you'll use daily: creating tables, inserting data, querying with SELECT, and modifying data with UPDATE and DELETE.

## Learning Objectives

By the end of this lesson, you will:
- Understand SQL syntax and conventions
- Create tables with proper data types and constraints
- Insert, update, and delete data
- Query data with SELECT, WHERE, and ORDER BY
- Use aggregate functions and GROUP BY
- Filter data with various operators

---

## SQL Basics

### SQL Statement Categories

```
┌─────────────────────────────────────────────────────────────┐
│                    SQL Categories                            │
├─────────────────┬───────────────────────────────────────────┤
│ DDL             │ Data Definition Language                   │
│ (Structure)     │ CREATE, ALTER, DROP, TRUNCATE             │
├─────────────────┼───────────────────────────────────────────┤
│ DML             │ Data Manipulation Language                 │
│ (Data)          │ SELECT, INSERT, UPDATE, DELETE            │
├─────────────────┼───────────────────────────────────────────┤
│ DCL             │ Data Control Language                      │
│ (Permissions)   │ GRANT, REVOKE                             │
├─────────────────┼───────────────────────────────────────────┤
│ TCL             │ Transaction Control Language               │
│ (Transactions)  │ BEGIN, COMMIT, ROLLBACK                   │
└─────────────────┴───────────────────────────────────────────┘
```

### SQL Conventions

```sql
-- SQL is case-insensitive for keywords
SELECT * FROM users;  -- Works
select * from USERS;  -- Also works

-- Convention: UPPERCASE keywords, lowercase identifiers
SELECT id, name FROM users WHERE active = true;

-- Use snake_case for identifiers
user_id, created_at, first_name

-- End statements with semicolon
SELECT * FROM users;

-- Comments
-- Single line comment
/* Multi-line
   comment */
```

---

## Data Types

### Common PostgreSQL Data Types

```sql
-- Numeric Types
INTEGER          -- Whole numbers (-2147483648 to 2147483647)
BIGINT           -- Large whole numbers
SERIAL           -- Auto-incrementing integer
BIGSERIAL        -- Auto-incrementing big integer
DECIMAL(10,2)    -- Exact decimal (10 digits, 2 after decimal)
NUMERIC(10,2)    -- Same as DECIMAL
REAL             -- Floating point (6 decimal precision)
DOUBLE PRECISION -- Floating point (15 decimal precision)

-- Text Types
CHAR(n)          -- Fixed-length string
VARCHAR(n)       -- Variable-length string (up to n chars)
TEXT             -- Unlimited length string

-- Boolean
BOOLEAN          -- true/false/null

-- Date/Time Types
DATE             -- Date only (2024-01-15)
TIME             -- Time only (14:30:00)
TIMESTAMP        -- Date and time (2024-01-15 14:30:00)
TIMESTAMPTZ      -- Timestamp with timezone (recommended)
INTERVAL         -- Time span

-- Other Common Types
UUID             -- Universally unique identifier
JSON             -- JSON data
JSONB            -- Binary JSON (faster queries)
ARRAY            -- Array of any type
BYTEA            -- Binary data
```

### Choosing the Right Type

```sql
-- User table with appropriate types
CREATE TABLE users (
  id              SERIAL PRIMARY KEY,           -- Auto-increment
  email           VARCHAR(255) NOT NULL UNIQUE, -- Email has max length
  name            VARCHAR(100) NOT NULL,        -- Names are bounded
  bio             TEXT,                         -- Bio can be long
  age             INTEGER,                      -- Whole numbers
  balance         DECIMAL(10, 2),               -- Money needs precision
  is_active       BOOLEAN DEFAULT true,         -- True/false
  preferences     JSONB,                        -- Flexible settings
  created_at      TIMESTAMPTZ DEFAULT NOW(),    -- With timezone
  last_login      TIMESTAMPTZ
);
```

---

## Creating Tables (DDL)

### CREATE TABLE

```sql
-- Basic table
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name VARCHAR(200) NOT NULL,
  price DECIMAL(10, 2) NOT NULL,
  description TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- With more constraints
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  user_id INTEGER NOT NULL REFERENCES users(id),
  total DECIMAL(10, 2) NOT NULL CHECK (total >= 0),
  status VARCHAR(20) DEFAULT 'pending',
  notes TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),

  -- Table-level constraint
  CONSTRAINT valid_status CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled'))
);
```

### Constraints

```sql
-- NOT NULL - Value required
name VARCHAR(100) NOT NULL

-- UNIQUE - No duplicate values
email VARCHAR(255) UNIQUE

-- PRIMARY KEY - Unique identifier (NOT NULL + UNIQUE)
id SERIAL PRIMARY KEY

-- FOREIGN KEY - Reference another table
user_id INTEGER REFERENCES users(id)

-- CHECK - Custom validation
age INTEGER CHECK (age >= 0 AND age <= 150)
price DECIMAL CHECK (price > 0)

-- DEFAULT - Default value if not provided
created_at TIMESTAMPTZ DEFAULT NOW()
status VARCHAR(20) DEFAULT 'active'
is_admin BOOLEAN DEFAULT false
```

### ALTER TABLE

```sql
-- Add column
ALTER TABLE users ADD COLUMN phone VARCHAR(20);

-- Remove column
ALTER TABLE users DROP COLUMN phone;

-- Rename column
ALTER TABLE users RENAME COLUMN name TO full_name;

-- Change column type
ALTER TABLE users ALTER COLUMN age TYPE SMALLINT;

-- Add constraint
ALTER TABLE users ADD CONSTRAINT email_format
  CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$');

-- Drop constraint
ALTER TABLE users DROP CONSTRAINT email_format;

-- Add foreign key
ALTER TABLE posts ADD CONSTRAINT fk_author
  FOREIGN KEY (author_id) REFERENCES users(id);

-- Rename table
ALTER TABLE users RENAME TO customers;
```

### DROP TABLE

```sql
-- Drop table (fails if table doesn't exist)
DROP TABLE products;

-- Drop if exists (safer)
DROP TABLE IF EXISTS products;

-- Drop with cascade (also drops dependent objects)
DROP TABLE users CASCADE;

-- Truncate (delete all rows, keep structure)
TRUNCATE TABLE logs;
TRUNCATE TABLE orders RESTART IDENTITY; -- Reset serial
```

---

## Inserting Data (INSERT)

### Basic INSERT

```sql
-- Insert single row
INSERT INTO users (name, email, age)
VALUES ('John Doe', 'john@example.com', 30);

-- Insert with all columns (not recommended)
INSERT INTO users
VALUES (1, 'john@example.com', 'John Doe', NULL, 30, ...);

-- Insert multiple rows
INSERT INTO users (name, email, age) VALUES
  ('John Doe', 'john@example.com', 30),
  ('Jane Doe', 'jane@example.com', 25),
  ('Bob Smith', 'bob@example.com', 35);
```

### INSERT with RETURNING

```sql
-- Get inserted row back
INSERT INTO users (name, email)
VALUES ('John Doe', 'john@example.com')
RETURNING *;

-- Get specific columns
INSERT INTO users (name, email)
VALUES ('John Doe', 'john@example.com')
RETURNING id, created_at;
```

### INSERT from SELECT

```sql
-- Copy data from another table
INSERT INTO archived_orders (id, user_id, total, created_at)
SELECT id, user_id, total, created_at
FROM orders
WHERE status = 'delivered' AND created_at < '2024-01-01';
```

### Handling Conflicts

```sql
-- Insert or update (upsert)
INSERT INTO users (email, name, login_count)
VALUES ('john@example.com', 'John Doe', 1)
ON CONFLICT (email)
DO UPDATE SET
  name = EXCLUDED.name,
  login_count = users.login_count + 1;

-- Insert or ignore
INSERT INTO users (email, name)
VALUES ('john@example.com', 'John Doe')
ON CONFLICT (email) DO NOTHING;
```

---

## Querying Data (SELECT)

### Basic SELECT

```sql
-- Select all columns
SELECT * FROM users;

-- Select specific columns
SELECT id, name, email FROM users;

-- Select with alias
SELECT
  id,
  name AS full_name,
  email AS email_address
FROM users;

-- Select with expressions
SELECT
  name,
  price,
  price * 0.1 AS tax,
  price * 1.1 AS total
FROM products;
```

### DISTINCT

```sql
-- Unique values only
SELECT DISTINCT status FROM orders;

-- Distinct combinations
SELECT DISTINCT city, state FROM addresses;
```

### WHERE Clause

```sql
-- Basic comparison
SELECT * FROM users WHERE age > 18;
SELECT * FROM users WHERE name = 'John Doe';
SELECT * FROM users WHERE is_active = true;

-- Multiple conditions (AND)
SELECT * FROM users
WHERE age >= 18 AND is_active = true;

-- Multiple conditions (OR)
SELECT * FROM users
WHERE role = 'admin' OR role = 'moderator';

-- Combining AND/OR (use parentheses!)
SELECT * FROM orders
WHERE status = 'pending'
  AND (total > 100 OR user_id = 1);
```

### Comparison Operators

```sql
-- Equality and inequality
WHERE age = 30        -- Equal
WHERE age != 30       -- Not equal
WHERE age <> 30       -- Not equal (SQL standard)

-- Greater/Less than
WHERE age > 18        -- Greater than
WHERE age >= 18       -- Greater than or equal
WHERE age < 65        -- Less than
WHERE age <= 65       -- Less than or equal

-- BETWEEN (inclusive)
WHERE age BETWEEN 18 AND 65
-- Same as: WHERE age >= 18 AND age <= 65

-- IN (match any in list)
WHERE status IN ('pending', 'processing', 'shipped')
-- Same as: WHERE status = 'pending' OR status = 'processing' OR status = 'shipped'

-- NOT IN
WHERE status NOT IN ('cancelled', 'refunded')
```

### Pattern Matching

```sql
-- LIKE (case-sensitive)
WHERE name LIKE 'John%'      -- Starts with 'John'
WHERE name LIKE '%Doe'       -- Ends with 'Doe'
WHERE name LIKE '%oh%'       -- Contains 'oh'
WHERE name LIKE 'J_hn'       -- J, any char, hn (John, Jahn, etc.)

-- ILIKE (case-insensitive, PostgreSQL)
WHERE name ILIKE '%john%'    -- john, John, JOHN, etc.

-- Regular expressions (PostgreSQL)
WHERE email ~ '^[a-z]+@'     -- Starts with lowercase letters before @
WHERE email ~* '^[a-z]+@'    -- Case-insensitive regex
```

### NULL Handling

```sql
-- Check for NULL
WHERE phone IS NULL
WHERE phone IS NOT NULL

-- COALESCE - first non-null value
SELECT COALESCE(phone, 'N/A') AS phone FROM users;
SELECT COALESCE(nickname, name, 'Anonymous') AS display_name FROM users;

-- NULLIF - return NULL if values equal
SELECT NULLIF(discount, 0) AS discount FROM orders;
-- Returns NULL instead of 0 (useful for division)
```

### ORDER BY

```sql
-- Ascending (default)
SELECT * FROM users ORDER BY name;
SELECT * FROM users ORDER BY name ASC;

-- Descending
SELECT * FROM users ORDER BY created_at DESC;

-- Multiple columns
SELECT * FROM users ORDER BY last_name ASC, first_name ASC;

-- Order by expression
SELECT * FROM products ORDER BY price * quantity DESC;

-- NULLS handling
SELECT * FROM users ORDER BY last_login DESC NULLS LAST;
SELECT * FROM users ORDER BY last_login DESC NULLS FIRST;
```

### LIMIT and OFFSET

```sql
-- Get first 10 rows
SELECT * FROM users LIMIT 10;

-- Pagination (page 2, 10 per page)
SELECT * FROM users
ORDER BY created_at DESC
LIMIT 10 OFFSET 10;

-- Common pagination pattern
SELECT * FROM users
ORDER BY id
LIMIT :page_size OFFSET (:page - 1) * :page_size;
```

---

## Aggregate Functions

### Basic Aggregates

```sql
-- COUNT - number of rows
SELECT COUNT(*) FROM users;
SELECT COUNT(phone) FROM users;  -- Excludes NULLs
SELECT COUNT(DISTINCT city) FROM users;

-- SUM - total
SELECT SUM(total) FROM orders;

-- AVG - average
SELECT AVG(age) FROM users;

-- MIN/MAX
SELECT MIN(price), MAX(price) FROM products;

-- Combining aggregates
SELECT
  COUNT(*) AS total_orders,
  SUM(total) AS revenue,
  AVG(total) AS avg_order,
  MIN(total) AS smallest_order,
  MAX(total) AS largest_order
FROM orders;
```

### GROUP BY

```sql
-- Count orders by status
SELECT status, COUNT(*) AS count
FROM orders
GROUP BY status;

-- Total revenue by month
SELECT
  DATE_TRUNC('month', created_at) AS month,
  SUM(total) AS revenue
FROM orders
GROUP BY DATE_TRUNC('month', created_at)
ORDER BY month;

-- Multiple grouping columns
SELECT
  EXTRACT(YEAR FROM created_at) AS year,
  EXTRACT(MONTH FROM created_at) AS month,
  status,
  COUNT(*) AS count,
  SUM(total) AS total
FROM orders
GROUP BY
  EXTRACT(YEAR FROM created_at),
  EXTRACT(MONTH FROM created_at),
  status
ORDER BY year, month, status;
```

### HAVING

Filter groups (like WHERE for aggregates):

```sql
-- Find customers with more than 5 orders
SELECT user_id, COUNT(*) AS order_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 5;

-- Find products with average rating below 3
SELECT product_id, AVG(rating) AS avg_rating
FROM reviews
GROUP BY product_id
HAVING AVG(rating) < 3;

-- WHERE vs HAVING
SELECT status, COUNT(*) AS count
FROM orders
WHERE created_at > '2024-01-01'     -- Filter rows BEFORE grouping
GROUP BY status
HAVING COUNT(*) > 10;              -- Filter groups AFTER grouping
```

---

## Updating Data (UPDATE)

### Basic UPDATE

```sql
-- Update single row
UPDATE users
SET name = 'John Smith'
WHERE id = 1;

-- Update multiple columns
UPDATE users
SET
  name = 'John Smith',
  email = 'johnsmith@example.com',
  updated_at = NOW()
WHERE id = 1;

-- Update multiple rows
UPDATE products
SET price = price * 1.1
WHERE category = 'electronics';
```

### UPDATE with RETURNING

```sql
-- Return updated rows
UPDATE users
SET is_active = false
WHERE last_login < NOW() - INTERVAL '1 year'
RETURNING id, email;
```

### UPDATE with Subquery

```sql
-- Update based on another table
UPDATE orders
SET status = 'vip'
WHERE user_id IN (
  SELECT id FROM users WHERE total_spent > 10000
);

-- Update with JOIN (PostgreSQL)
UPDATE orders o
SET discount = 0.1
FROM users u
WHERE o.user_id = u.id AND u.is_premium = true;
```

### Conditional UPDATE

```sql
-- CASE expression
UPDATE products
SET price = CASE
  WHEN category = 'electronics' THEN price * 1.1
  WHEN category = 'clothing' THEN price * 1.05
  ELSE price
END;

-- Update only if different
UPDATE users
SET
  name = COALESCE(NULLIF(:new_name, ''), name),
  email = COALESCE(NULLIF(:new_email, ''), email)
WHERE id = :id;
```

---

## Deleting Data (DELETE)

### Basic DELETE

```sql
-- Delete single row
DELETE FROM users WHERE id = 1;

-- Delete multiple rows
DELETE FROM orders WHERE status = 'cancelled';

-- Delete with complex condition
DELETE FROM sessions
WHERE user_id = 1 AND expires_at < NOW();
```

### DELETE with RETURNING

```sql
-- Return deleted rows
DELETE FROM users
WHERE is_active = false
RETURNING id, email;
```

### DELETE with Subquery

```sql
-- Delete based on another table
DELETE FROM orders
WHERE user_id IN (
  SELECT id FROM users WHERE is_banned = true
);
```

### Safe Deletion Practices

```sql
-- Always use WHERE (avoid deleting all rows!)
DELETE FROM users; -- DANGEROUS!

-- Preview what will be deleted
SELECT * FROM users WHERE last_login < '2023-01-01';
-- Then delete
DELETE FROM users WHERE last_login < '2023-01-01';

-- Soft delete pattern (recommended)
UPDATE users
SET deleted_at = NOW(), is_active = false
WHERE id = 1;
```

---

## Practical Examples

### E-commerce Queries

```sql
-- Create sample schema
CREATE TABLE customers (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  tier VARCHAR(20) DEFAULT 'standard',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name VARCHAR(200) NOT NULL,
  price DECIMAL(10, 2) NOT NULL,
  stock INTEGER DEFAULT 0,
  category VARCHAR(50)
);

CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  customer_id INTEGER REFERENCES customers(id),
  total DECIMAL(10, 2) NOT NULL,
  status VARCHAR(20) DEFAULT 'pending',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE order_items (
  id SERIAL PRIMARY KEY,
  order_id INTEGER REFERENCES orders(id),
  product_id INTEGER REFERENCES products(id),
  quantity INTEGER NOT NULL,
  price DECIMAL(10, 2) NOT NULL
);

-- Insert sample data
INSERT INTO customers (name, email, tier) VALUES
  ('Alice Johnson', 'alice@example.com', 'premium'),
  ('Bob Williams', 'bob@example.com', 'standard'),
  ('Carol Davis', 'carol@example.com', 'premium');

INSERT INTO products (name, price, stock, category) VALUES
  ('Laptop', 999.99, 50, 'electronics'),
  ('Mouse', 29.99, 200, 'electronics'),
  ('Desk Chair', 199.99, 30, 'furniture'),
  ('Notebook', 9.99, 500, 'office');

-- Query: Find total revenue by category
SELECT
  p.category,
  SUM(oi.quantity * oi.price) AS revenue
FROM order_items oi
JOIN products p ON oi.product_id = p.id
GROUP BY p.category
ORDER BY revenue DESC;

-- Query: Find top customers by total spent
SELECT
  c.id,
  c.name,
  c.email,
  COUNT(o.id) AS order_count,
  SUM(o.total) AS total_spent
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
GROUP BY c.id, c.name, c.email
ORDER BY total_spent DESC NULLS LAST
LIMIT 10;

-- Query: Products low in stock
SELECT name, stock, category
FROM products
WHERE stock < 20
ORDER BY stock ASC;

-- Query: Orders pending for more than 24 hours
SELECT
  o.id,
  c.name AS customer,
  o.total,
  o.created_at,
  NOW() - o.created_at AS pending_duration
FROM orders o
JOIN customers c ON o.customer_id = c.id
WHERE o.status = 'pending'
  AND o.created_at < NOW() - INTERVAL '24 hours'
ORDER BY o.created_at ASC;
```

### User Management Queries

```sql
-- Active users who logged in this month
SELECT id, name, email, last_login
FROM users
WHERE is_active = true
  AND last_login >= DATE_TRUNC('month', CURRENT_DATE)
ORDER BY last_login DESC;

-- Users by registration month
SELECT
  DATE_TRUNC('month', created_at) AS month,
  COUNT(*) AS new_users
FROM users
GROUP BY DATE_TRUNC('month', created_at)
ORDER BY month DESC;

-- Find duplicate emails (data quality check)
SELECT email, COUNT(*) AS count
FROM users
GROUP BY email
HAVING COUNT(*) > 1;

-- User activity summary
SELECT
  CASE
    WHEN last_login >= NOW() - INTERVAL '7 days' THEN 'active'
    WHEN last_login >= NOW() - INTERVAL '30 days' THEN 'recent'
    WHEN last_login >= NOW() - INTERVAL '90 days' THEN 'inactive'
    ELSE 'dormant'
  END AS activity_status,
  COUNT(*) AS user_count
FROM users
GROUP BY 1
ORDER BY user_count DESC;
```

---

## Common SQL Patterns

### Pagination

```sql
-- Offset pagination (simple but slow for large offsets)
SELECT * FROM products
ORDER BY created_at DESC
LIMIT 20 OFFSET 40;

-- Cursor pagination (more efficient)
SELECT * FROM products
WHERE created_at < :last_seen_created_at
ORDER BY created_at DESC
LIMIT 20;

-- Keyset pagination (best for large datasets)
SELECT * FROM products
WHERE (created_at, id) < (:last_created_at, :last_id)
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

### Search

```sql
-- Simple search
SELECT * FROM products
WHERE name ILIKE '%keyboard%';

-- Multi-column search
SELECT * FROM products
WHERE name ILIKE '%search%'
   OR description ILIKE '%search%';

-- Full-text search (PostgreSQL)
SELECT * FROM products
WHERE to_tsvector('english', name || ' ' || description)
   @@ plainto_tsquery('english', 'wireless keyboard');
```

### Existence Checks

```sql
-- Check if user exists
SELECT EXISTS (
  SELECT 1 FROM users WHERE email = 'test@example.com'
) AS exists;

-- In application code
SELECT 1 FROM users WHERE email = $1 LIMIT 1;
```

### Conditional Aggregation

```sql
-- Count by status in one query
SELECT
  COUNT(*) AS total,
  COUNT(*) FILTER (WHERE status = 'pending') AS pending,
  COUNT(*) FILTER (WHERE status = 'processing') AS processing,
  COUNT(*) FILTER (WHERE status = 'shipped') AS shipped,
  COUNT(*) FILTER (WHERE status = 'delivered') AS delivered
FROM orders;
```

---

## Key Takeaways

1. **Use appropriate data types** - VARCHAR for bounded text, TEXT for unlimited, DECIMAL for money
2. **Add constraints** - NOT NULL, UNIQUE, CHECK prevent bad data
3. **Parameterize queries** - Never concatenate user input
4. **Use RETURNING** - Get inserted/updated/deleted rows without extra query
5. **GROUP BY with HAVING** - Aggregate then filter groups
6. **ORDER BY before LIMIT** - Always order for consistent pagination
7. **Preview before DELETE** - Use SELECT first to verify affected rows

---

## What's Next?

Tomorrow we'll learn about **JOINs and Relations** - connecting tables together to query related data across your database!
