# Day 3: JOINs and Relations

## Introduction

Real-world data rarely lives in a single table. Relational databases shine when connecting related data across tables. This lesson covers database relationships, foreign keys, and the various types of JOINs that let you query related data efficiently.

## Learning Objectives

By the end of this lesson, you will:
- Understand different types of table relationships
- Design schemas with proper foreign keys
- Write queries using all types of JOINs
- Choose the right JOIN for your use case
- Avoid common JOIN pitfalls
- Optimize JOIN performance

---

## Database Relationships

### One-to-Many (1:N)

The most common relationship. One record in Table A relates to many records in Table B.

```
┌─────────────┐       ┌─────────────────┐
│   users     │       │     posts       │
├─────────────┤       ├─────────────────┤
│ id (PK)     │───┐   │ id (PK)         │
│ name        │   │   │ title           │
│ email       │   └──▶│ user_id (FK)    │
└─────────────┘       │ content         │
                      └─────────────────┘

One user → Many posts
```

```sql
-- Create tables
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL
);

CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  title VARCHAR(200) NOT NULL,
  content TEXT,
  user_id INTEGER NOT NULL REFERENCES users(id),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Insert data
INSERT INTO users (name, email) VALUES
  ('Alice', 'alice@example.com'),
  ('Bob', 'bob@example.com');

INSERT INTO posts (title, content, user_id) VALUES
  ('First Post', 'Hello World', 1),
  ('Second Post', 'More content', 1),
  ('Bob''s Post', 'Bob writes', 2);
```

### One-to-One (1:1)

One record in Table A relates to exactly one record in Table B.

```
┌─────────────┐       ┌─────────────────────┐
│   users     │       │   user_profiles     │
├─────────────┤       ├─────────────────────┤
│ id (PK)     │◀──────│ user_id (PK, FK)    │
│ name        │       │ bio                 │
│ email       │       │ avatar_url          │
└─────────────┘       │ phone               │
                      └─────────────────────┘

One user → One profile
```

```sql
CREATE TABLE user_profiles (
  user_id INTEGER PRIMARY KEY REFERENCES users(id),
  bio TEXT,
  avatar_url VARCHAR(500),
  phone VARCHAR(20),
  birth_date DATE
);

-- Alternative: Store in same table if always needed together
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  -- Profile fields (one-to-one relationship embedded)
  bio TEXT,
  avatar_url VARCHAR(500)
);
```

### Many-to-Many (M:N)

Many records in Table A relate to many records in Table B. Requires a junction/join table.

```
┌─────────────┐       ┌───────────────────┐       ┌─────────────┐
│   posts     │       │   posts_tags      │       │    tags     │
├─────────────┤       ├───────────────────┤       ├─────────────┤
│ id (PK)     │◀──────│ post_id (FK)      │──────▶│ id (PK)     │
│ title       │       │ tag_id (FK)       │       │ name        │
│ content     │       │ (composite PK)    │       │             │
└─────────────┘       └───────────────────┘       └─────────────┘

Many posts ↔ Many tags
```

```sql
CREATE TABLE tags (
  id SERIAL PRIMARY KEY,
  name VARCHAR(50) UNIQUE NOT NULL
);

-- Junction table
CREATE TABLE posts_tags (
  post_id INTEGER REFERENCES posts(id) ON DELETE CASCADE,
  tag_id INTEGER REFERENCES tags(id) ON DELETE CASCADE,
  PRIMARY KEY (post_id, tag_id)
);

-- Insert tags
INSERT INTO tags (name) VALUES ('javascript'), ('tutorial'), ('beginner');

-- Tag posts
INSERT INTO posts_tags (post_id, tag_id) VALUES
  (1, 1), -- Post 1 has javascript tag
  (1, 2), -- Post 1 has tutorial tag
  (2, 1), -- Post 2 has javascript tag
  (2, 3); -- Post 2 has beginner tag
```

### Self-Referential Relationships

A table that references itself.

```
┌───────────────────────────────────┐
│           employees               │
├───────────────────────────────────┤
│ id (PK)                           │
│ name                              │
│ manager_id (FK → employees.id)    │──┐
└───────────────────────────────────┘  │
           ▲                            │
           └────────────────────────────┘
```

```sql
CREATE TABLE employees (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  manager_id INTEGER REFERENCES employees(id)
);

-- CEO has no manager
INSERT INTO employees (name, manager_id) VALUES ('CEO', NULL);

-- Managers report to CEO
INSERT INTO employees (name, manager_id) VALUES ('CTO', 1);
INSERT INTO employees (name, manager_id) VALUES ('CFO', 1);

-- Developers report to CTO
INSERT INTO employees (name, manager_id) VALUES ('Developer 1', 2);
INSERT INTO employees (name, manager_id) VALUES ('Developer 2', 2);
```

---

## Foreign Keys

### Creating Foreign Keys

```sql
-- Inline (column level)
CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id)
);

-- Named constraint
CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  user_id INTEGER,
  CONSTRAINT fk_posts_user FOREIGN KEY (user_id) REFERENCES users(id)
);

-- Add to existing table
ALTER TABLE posts
ADD CONSTRAINT fk_posts_user
FOREIGN KEY (user_id) REFERENCES users(id);
```

### Referential Actions

What happens when referenced row is updated/deleted?

```sql
-- CASCADE: Delete/update related rows
CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id) ON DELETE CASCADE
);
-- Deleting user deletes all their posts

-- SET NULL: Set foreign key to NULL
CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id) ON DELETE SET NULL
);
-- Deleting user sets posts.user_id to NULL

-- SET DEFAULT: Set to default value
CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  user_id INTEGER DEFAULT 1 REFERENCES users(id) ON DELETE SET DEFAULT
);

-- RESTRICT: Prevent deletion if references exist (default)
CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id) ON DELETE RESTRICT
);
-- Cannot delete user if they have posts

-- NO ACTION: Same as RESTRICT but checked at end of transaction
CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id) ON DELETE NO ACTION
);
```

### Best Practices for Foreign Keys

```sql
-- Always name your constraints
CONSTRAINT fk_table_column FOREIGN KEY (column) REFERENCES other(id)

-- Index foreign keys for performance
CREATE INDEX idx_posts_user_id ON posts(user_id);

-- Consider ON DELETE behavior carefully
-- CASCADE for owned entities (user → posts)
-- SET NULL for optional relationships
-- RESTRICT for important references (orders → products)
```

---

## JOIN Types

### Sample Data Setup

```sql
-- Create tables
CREATE TABLE departments (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL
);

CREATE TABLE employees (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  department_id INTEGER REFERENCES departments(id),
  salary DECIMAL(10, 2)
);

-- Insert data
INSERT INTO departments (name) VALUES
  ('Engineering'),
  ('Marketing'),
  ('Sales'),
  ('HR');  -- No employees

INSERT INTO employees (name, department_id, salary) VALUES
  ('Alice', 1, 90000),    -- Engineering
  ('Bob', 1, 85000),      -- Engineering
  ('Carol', 2, 75000),    -- Marketing
  ('Dave', 3, 80000),     -- Sales
  ('Eve', NULL, 70000);   -- No department
```

### INNER JOIN

Returns only matching rows from both tables.

```
departments              employees              INNER JOIN Result
┌────┬─────────────┐    ┌────┬───────┬────┐   ┌─────────────┬───────┐
│ id │ name        │    │ id │ name  │dept│   │ dept_name   │ name  │
├────┼─────────────┤    ├────┼───────┼────┤   ├─────────────┼───────┤
│ 1  │ Engineering │    │ 1  │ Alice │ 1  │   │ Engineering │ Alice │
│ 2  │ Marketing   │    │ 2  │ Bob   │ 1  │   │ Engineering │ Bob   │
│ 3  │ Sales       │    │ 3  │ Carol │ 2  │   │ Marketing   │ Carol │
│ 4  │ HR          │    │ 4  │ Dave  │ 3  │   │ Sales       │ Dave  │
└────┴─────────────┘    │ 5  │ Eve   │NULL│   └─────────────┴───────┘
                        └────┴───────┴────┘
                                              HR excluded (no employees)
                                              Eve excluded (no department)
```

```sql
-- INNER JOIN (explicit)
SELECT d.name AS department, e.name AS employee
FROM departments d
INNER JOIN employees e ON d.id = e.department_id;

-- JOIN is same as INNER JOIN
SELECT d.name AS department, e.name AS employee
FROM departments d
JOIN employees e ON d.id = e.department_id;
```

### LEFT JOIN (LEFT OUTER JOIN)

Returns all rows from left table, matched rows from right table (NULL if no match).

```
LEFT JOIN Result
┌─────────────┬───────┐
│ dept_name   │ name  │
├─────────────┼───────┤
│ Engineering │ Alice │
│ Engineering │ Bob   │
│ Marketing   │ Carol │
│ Sales       │ Dave  │
│ HR          │ NULL  │  ← HR included, no employee
└─────────────┴───────┘
```

```sql
-- LEFT JOIN
SELECT d.name AS department, e.name AS employee
FROM departments d
LEFT JOIN employees e ON d.id = e.department_id;

-- Find departments with no employees
SELECT d.name AS department
FROM departments d
LEFT JOIN employees e ON d.id = e.department_id
WHERE e.id IS NULL;
```

### RIGHT JOIN (RIGHT OUTER JOIN)

Returns all rows from right table, matched rows from left table.

```
RIGHT JOIN Result
┌─────────────┬───────┐
│ dept_name   │ name  │
├─────────────┼───────┤
│ Engineering │ Alice │
│ Engineering │ Bob   │
│ Marketing   │ Carol │
│ Sales       │ Dave  │
│ NULL        │ Eve   │  ← Eve included, no department
└─────────────┴───────┘
```

```sql
-- RIGHT JOIN
SELECT d.name AS department, e.name AS employee
FROM departments d
RIGHT JOIN employees e ON d.id = e.department_id;

-- Usually rewritten as LEFT JOIN (more readable)
SELECT d.name AS department, e.name AS employee
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;
```

### FULL OUTER JOIN

Returns all rows from both tables, with NULL where no match.

```
FULL OUTER JOIN Result
┌─────────────┬───────┐
│ dept_name   │ name  │
├─────────────┼───────┤
│ Engineering │ Alice │
│ Engineering │ Bob   │
│ Marketing   │ Carol │
│ Sales       │ Dave  │
│ HR          │ NULL  │  ← HR with no employees
│ NULL        │ Eve   │  ← Eve with no department
└─────────────┴───────┘
```

```sql
-- FULL OUTER JOIN
SELECT d.name AS department, e.name AS employee
FROM departments d
FULL OUTER JOIN employees e ON d.id = e.department_id;

-- Find unmatched rows from both sides
SELECT d.name AS department, e.name AS employee
FROM departments d
FULL OUTER JOIN employees e ON d.id = e.department_id
WHERE d.id IS NULL OR e.id IS NULL;
```

### CROSS JOIN

Returns Cartesian product (all combinations).

```sql
-- CROSS JOIN
SELECT d.name AS department, e.name AS employee
FROM departments d
CROSS JOIN employees e;
-- Returns 4 departments × 5 employees = 20 rows

-- Implicit cross join (old syntax)
SELECT d.name, e.name
FROM departments d, employees e;

-- Useful for generating combinations
SELECT
  DATE_TRUNC('day', generate_series('2024-01-01', '2024-01-31', '1 day')) AS date,
  p.name AS product
FROM products p;
```

### Self JOIN

Join a table to itself.

```sql
-- Find employees and their managers
SELECT
  e.name AS employee,
  m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;

-- Find employees who earn more than their manager
SELECT
  e.name AS employee,
  e.salary AS employee_salary,
  m.name AS manager,
  m.salary AS manager_salary
FROM employees e
JOIN employees m ON e.manager_id = m.id
WHERE e.salary > m.salary;
```

---

## Complex JOIN Queries

### Multiple JOINs

```sql
-- Posts with author and tags
SELECT
  p.id,
  p.title,
  u.name AS author,
  STRING_AGG(t.name, ', ') AS tags
FROM posts p
JOIN users u ON p.user_id = u.id
LEFT JOIN posts_tags pt ON p.id = pt.post_id
LEFT JOIN tags t ON pt.tag_id = t.id
GROUP BY p.id, p.title, u.name;
```

### JOINs with Aggregation

```sql
-- Department statistics
SELECT
  d.name AS department,
  COUNT(e.id) AS employee_count,
  ROUND(AVG(e.salary), 2) AS avg_salary,
  SUM(e.salary) AS total_salary
FROM departments d
LEFT JOIN employees e ON d.id = e.department_id
GROUP BY d.id, d.name
ORDER BY employee_count DESC;

-- Users with post counts
SELECT
  u.id,
  u.name,
  COUNT(p.id) AS post_count,
  MAX(p.created_at) AS last_post_date
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
GROUP BY u.id, u.name
ORDER BY post_count DESC;
```

### Filtered JOINs

```sql
-- Join with condition in ON clause vs WHERE clause
-- ON: Filter before joining (affects outer join results)
SELECT d.name, e.name
FROM departments d
LEFT JOIN employees e ON d.id = e.department_id AND e.salary > 80000;
-- Shows all departments, only high-salary employees

-- WHERE: Filter after joining
SELECT d.name, e.name
FROM departments d
LEFT JOIN employees e ON d.id = e.department_id
WHERE e.salary > 80000 OR e.salary IS NULL;
-- Different result!
```

### Subqueries in JOINs

```sql
-- Join with derived table
SELECT
  u.name,
  ps.post_count,
  ps.avg_views
FROM users u
JOIN (
  SELECT
    user_id,
    COUNT(*) AS post_count,
    AVG(views) AS avg_views
  FROM posts
  GROUP BY user_id
) ps ON u.id = ps.user_id;

-- Lateral join (correlated subquery)
SELECT
  d.name AS department,
  top_earner.name AS top_earner,
  top_earner.salary
FROM departments d
LEFT JOIN LATERAL (
  SELECT name, salary
  FROM employees
  WHERE department_id = d.id
  ORDER BY salary DESC
  LIMIT 1
) top_earner ON true;
```

---

## JOIN vs Subquery

### When to Use JOIN

```sql
-- Need columns from multiple tables
SELECT o.id, u.name, o.total
FROM orders o
JOIN users u ON o.user_id = u.id;

-- Aggregating related data
SELECT u.name, COUNT(p.id) AS posts
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
GROUP BY u.id, u.name;
```

### When to Use Subquery

```sql
-- Existence check
SELECT * FROM users u
WHERE EXISTS (
  SELECT 1 FROM orders o
  WHERE o.user_id = u.id AND o.total > 1000
);

-- Filtering by aggregate
SELECT * FROM users
WHERE id IN (
  SELECT user_id FROM orders
  GROUP BY user_id
  HAVING SUM(total) > 5000
);

-- Single value comparison
SELECT * FROM products
WHERE price > (SELECT AVG(price) FROM products);
```

### Performance Comparison

```sql
-- These are often equivalent (optimizer may convert)

-- Subquery with IN
SELECT * FROM users
WHERE id IN (SELECT user_id FROM orders);

-- JOIN (often faster)
SELECT DISTINCT u.*
FROM users u
JOIN orders o ON u.id = o.user_id;

-- EXISTS (often fastest for checking existence)
SELECT * FROM users u
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);
```

---

## Practical Examples

### E-commerce Order Report

```sql
-- Full order details with customer and items
SELECT
  o.id AS order_id,
  o.created_at AS order_date,
  c.name AS customer_name,
  c.email AS customer_email,
  p.name AS product_name,
  oi.quantity,
  oi.price AS unit_price,
  oi.quantity * oi.price AS line_total,
  o.total AS order_total
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id
WHERE o.created_at >= '2024-01-01'
ORDER BY o.created_at DESC, o.id, oi.id;
```

### Blog with Categories and Comments

```sql
-- Posts with category, author, and comment count
SELECT
  p.id,
  p.title,
  p.slug,
  u.name AS author,
  c.name AS category,
  COUNT(DISTINCT cm.id) AS comment_count,
  STRING_AGG(DISTINCT t.name, ', ' ORDER BY t.name) AS tags,
  p.published_at
FROM posts p
JOIN users u ON p.author_id = u.id
LEFT JOIN categories c ON p.category_id = c.id
LEFT JOIN posts_tags pt ON p.id = pt.post_id
LEFT JOIN tags t ON pt.tag_id = t.id
LEFT JOIN comments cm ON p.id = cm.post_id
WHERE p.status = 'published'
GROUP BY p.id, p.title, p.slug, u.name, c.name, p.published_at
ORDER BY p.published_at DESC;
```

### Organizational Hierarchy

```sql
-- Full organizational tree with recursive CTE
WITH RECURSIVE org_tree AS (
  -- Base case: top-level (no manager)
  SELECT
    id,
    name,
    manager_id,
    1 AS level,
    name AS path
  FROM employees
  WHERE manager_id IS NULL

  UNION ALL

  -- Recursive case: employees with managers
  SELECT
    e.id,
    e.name,
    e.manager_id,
    ot.level + 1,
    ot.path || ' → ' || e.name
  FROM employees e
  JOIN org_tree ot ON e.manager_id = ot.id
)
SELECT
  REPEAT('  ', level - 1) || name AS employee,
  level,
  path
FROM org_tree
ORDER BY path;
```

### User Activity Dashboard

```sql
-- User activity summary
SELECT
  u.id,
  u.name,
  u.created_at AS member_since,
  COUNT(DISTINCT p.id) AS total_posts,
  COUNT(DISTINCT c.id) AS total_comments,
  COUNT(DISTINCT l.post_id) AS posts_liked,
  COALESCE(MAX(p.created_at), MAX(c.created_at)) AS last_activity
FROM users u
LEFT JOIN posts p ON u.id = p.author_id
LEFT JOIN comments c ON u.id = c.author_id
LEFT JOIN likes l ON u.id = l.user_id
GROUP BY u.id, u.name, u.created_at
ORDER BY last_activity DESC NULLS LAST;
```

---

## JOIN Performance Tips

### Always Index Foreign Keys

```sql
-- Without index: Full table scan for each join
-- With index: Fast index lookup

CREATE INDEX idx_posts_user_id ON posts(user_id);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
CREATE INDEX idx_order_items_product_id ON order_items(product_id);
```

### Use EXPLAIN ANALYZE

```sql
EXPLAIN ANALYZE
SELECT u.name, COUNT(p.id)
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
GROUP BY u.id, u.name;

-- Look for:
-- - Seq Scan (sequential scan) → Consider adding index
-- - Nested Loop → OK for small tables
-- - Hash Join → Good for larger tables
-- - Merge Join → Good for sorted data
```

### Limit Data Before Joining

```sql
-- Bad: Join everything, then filter
SELECT o.*, u.name
FROM orders o
JOIN users u ON o.user_id = u.id
WHERE o.created_at > '2024-01-01';

-- Better: Filter early with index
SELECT o.*, u.name
FROM (
  SELECT * FROM orders
  WHERE created_at > '2024-01-01'
) o
JOIN users u ON o.user_id = u.id;

-- Or use proper indexing so optimizer can push down
CREATE INDEX idx_orders_created_at ON orders(created_at);
```

### Avoid SELECT *

```sql
-- Bad: Fetches all columns from all tables
SELECT *
FROM orders o
JOIN users u ON o.user_id = u.id
JOIN order_items oi ON o.id = oi.order_id;

-- Good: Select only needed columns
SELECT
  o.id,
  o.total,
  u.name,
  oi.product_id,
  oi.quantity
FROM orders o
JOIN users u ON o.user_id = u.id
JOIN order_items oi ON o.id = oi.order_id;
```

---

## Key Takeaways

1. **Choose the right relationship** - 1:1, 1:N, or M:N based on your data model
2. **Foreign keys ensure integrity** - Use appropriate ON DELETE actions
3. **INNER JOIN for matches** - Only rows that exist in both tables
4. **LEFT JOIN for optional** - All from left, matches from right
5. **Index your foreign keys** - JOINs are only fast with proper indexes
6. **Filter early** - Apply WHERE conditions before large JOINs
7. **Use EXPLAIN** - Understand your query execution plan

---

## What's Next?

Tomorrow we'll dive into **Prisma Setup** - a modern TypeScript ORM that makes database operations type-safe and developer-friendly!
