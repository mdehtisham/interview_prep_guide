## **Queries and Operations**

<details>
<summary>11. What is the difference between DELETE, TRUNCATE, and DROP?</summary>

### Answer:

**DELETE**, **TRUNCATE**, and **DROP** are three different SQL commands for removing data, but they have distinct purposes, behaviors, and performance characteristics.

---

### **Quick Comparison:**

| Feature | DELETE | TRUNCATE | DROP |
|---------|--------|----------|------|
| **Purpose** | Remove specific rows | Remove all rows | Remove entire table |
| **WHERE clause** | Yes | No | No |
| **Speed** | Slower (row-by-row) | Fast (bulk operation) | Instant |
| **Rollback** | Yes (in transaction) | Yes (in transaction) | Yes (in transaction) |
| **Triggers** | Fires DELETE triggers | No triggers | No triggers |
| **Identity/Serial** | Doesn't reset | Resets to initial | N/A (table gone) |
| **Space** | Doesn't reclaim immediately | Reclaims immediately | Reclaims immediately |
| **Locks** | Row-level locks | Table-level lock | Table-level lock |
| **Foreign Keys** | Respects FK constraints | Can cascade if configured | Must drop dependents first |
| **Log** | Logs each row | Minimal logging | Logs table removal |

---

### **1. DELETE - Selective Row Removal:**

Removes specific rows based on a condition. Can delete all or some rows.

```sql
-- Create sample table
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    department VARCHAR(50),
    salary NUMERIC(10,2),
    hire_date DATE
);

INSERT INTO employees (name, department, salary, hire_date) VALUES
    ('John Doe', 'IT', 75000, '2020-01-15'),
    ('Jane Smith', 'HR', 65000, '2019-03-20'),
    ('Bob Johnson', 'IT', 80000, '2021-06-10'),
    ('Alice Brown', 'Sales', 70000, '2018-11-05');

-- Delete specific rows (with WHERE clause)
DELETE FROM employees WHERE department = 'IT';
-- Removes: John Doe, Bob Johnson

-- Delete with complex condition
DELETE FROM employees 
WHERE salary < 70000 AND hire_date < '2020-01-01';
-- Removes: Jane Smith (if salary < 70000)

-- Delete all rows (but keeps table structure)
DELETE FROM employees;
-- Table still exists, but empty

-- Delete with RETURNING (PostgreSQL feature)
DELETE FROM employees 
WHERE department = 'Sales'
RETURNING *;
-- Deletes and returns deleted rows
```

**Transaction & Rollback:**
```sql
BEGIN;
    DELETE FROM employees WHERE id = 1;
    SELECT * FROM employees;  -- Row is gone
ROLLBACK;
-- Row is back!
SELECT * FROM employees;
```

**Performance Characteristics:**
```sql
-- DELETE is slow for large datasets
-- Processes row-by-row, creates WAL entries for each row

-- Example: Delete 1 million rows
DELETE FROM large_table WHERE category = 'old';
-- Takes time: Scans table, deletes each row, logs each deletion

-- View delete progress (in another session)
SELECT 
    query,
    state,
    query_start,
    NOW() - query_start AS duration
FROM pg_stat_activity
WHERE query LIKE 'DELETE%';
```

**Triggers are Fired:**
```sql
-- Create audit trigger
CREATE TABLE employees_audit (
    id SERIAL PRIMARY KEY,
    employee_id INTEGER,
    action VARCHAR(10),
    deleted_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE OR REPLACE FUNCTION audit_employee_delete()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO employees_audit (employee_id, action)
    VALUES (OLD.id, 'DELETE');
    RETURN OLD;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER employee_delete_trigger
AFTER DELETE ON employees
FOR EACH ROW
EXECUTE FUNCTION audit_employee_delete();

-- DELETE fires the trigger
DELETE FROM employees WHERE id = 1;
-- Trigger executes, audit record created

SELECT * FROM employees_audit;
-- Shows delete audit entry
```

---

### **2. TRUNCATE - Fast Bulk Removal:**

Removes all rows from a table quickly without scanning rows. Resets sequences.

```sql
-- Truncate table (removes all rows)
TRUNCATE TABLE employees;
-- Fast: Doesn't scan rows, just deallocates data pages

-- Truncate multiple tables at once
TRUNCATE TABLE employees, departments, projects;

-- Truncate with CASCADE (handles foreign keys)
CREATE TABLE departments (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE employees_fk (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    dept_id INTEGER REFERENCES departments(id)
);

INSERT INTO departments VALUES (1, 'IT'), (2, 'HR');
INSERT INTO employees_fk VALUES (1, 'John', 1), (2, 'Jane', 2);

-- This fails (foreign key constraint)
TRUNCATE TABLE departments;
-- ERROR: cannot truncate a table referenced in a foreign key constraint

-- This works (cascades to dependent tables)
TRUNCATE TABLE departments CASCADE;
-- Truncates both departments AND employees_fk

-- Truncate with RESTART IDENTITY (resets sequences)
INSERT INTO employees (name) VALUES ('John'), ('Jane'), ('Bob');
-- IDs: 1, 2, 3

DELETE FROM employees;
INSERT INTO employees (name) VALUES ('Alice');
-- ID: 4 (sequence continues)

TRUNCATE TABLE employees;
INSERT INTO employees (name) VALUES ('Alice');
-- ID: 5 (sequence still continues)

TRUNCATE TABLE employees RESTART IDENTITY;
INSERT INTO employees (name) VALUES ('Alice');
-- ID: 1 (sequence reset!)
```

**Performance Benefits:**
```sql
-- Compare DELETE vs TRUNCATE on large table
CREATE TABLE test_large (id SERIAL, data TEXT);
INSERT INTO test_large (data) 
SELECT 'data_' || i FROM generate_series(1, 1000000) i;

-- Method 1: DELETE (slow)
\timing on
DELETE FROM test_large;
-- Time: 5000+ ms (processes each row)

-- Method 2: TRUNCATE (fast)
\timing on
TRUNCATE TABLE test_large;
-- Time: 50 ms (instant)
```

**Transaction Support:**
```sql
BEGIN;
    TRUNCATE TABLE employees;
    SELECT COUNT(*) FROM employees;  -- 0 rows
ROLLBACK;
-- Rolled back!
SELECT COUNT(*) FROM employees;  -- Original rows back
```

**No Triggers:**
```sql
-- TRUNCATE does NOT fire DELETE triggers
TRUNCATE TABLE employees;
-- audit_employee_delete() trigger is NOT executed
-- No audit entries created
```

---

### **3. DROP - Complete Table Removal:**

Removes the entire table structure and data. Table no longer exists.

```sql
-- Drop table
DROP TABLE employees;
-- Table completely removed (structure + data)

-- Try to query dropped table
SELECT * FROM employees;
-- ERROR: relation "employees" does not exist

-- Drop with IF EXISTS (no error if table doesn't exist)
DROP TABLE IF EXISTS employees;
-- Safe: No error if already dropped

-- Drop with CASCADE (removes dependent objects)
CREATE TABLE departments_v2 (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE employees_v2 (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    dept_id INTEGER REFERENCES departments_v2(id)
);

CREATE VIEW dept_summary AS
    SELECT d.name, COUNT(e.id) AS employee_count
    FROM departments_v2 d
    LEFT JOIN employees_v2 e ON d.id = e.dept_id
    GROUP BY d.name;

-- This fails (dependencies exist)
DROP TABLE departments_v2;
-- ERROR: cannot drop table departments_v2 because other objects depend on it

-- This works (drops all dependent objects)
DROP TABLE departments_v2 CASCADE;
-- Drops: departments_v2, employees_v2 (FK), dept_summary (view)

-- Drop multiple tables
DROP TABLE table1, table2, table3;
```

**Recovering Dropped Tables:**
```sql
-- DROP in transaction can be rolled back
BEGIN;
    DROP TABLE employees;
    SELECT * FROM employees;  -- ERROR: table doesn't exist
ROLLBACK;
-- Table is back!
SELECT * FROM employees;  -- Works again

-- But if committed, cannot rollback
BEGIN;
    DROP TABLE employees;
COMMIT;
-- Table is permanently gone (unless you have backups)
```

---

### **Practical Decision Matrix:**

#### **Use DELETE when:**

```sql
-- ✅ Need to remove specific rows
DELETE FROM orders WHERE status = 'cancelled';

-- ✅ Need to maintain audit trail (triggers fire)
DELETE FROM users WHERE inactive_days > 365;
-- Audit trigger logs each deletion

-- ✅ Need to use WHERE clause
DELETE FROM logs WHERE created_at < NOW() - INTERVAL '90 days';

-- ✅ Working with small datasets
DELETE FROM temp_table WHERE processed = true;
```

#### **Use TRUNCATE when:**

```sql
-- ✅ Need to remove ALL rows quickly
TRUNCATE TABLE staging_data;

-- ✅ Need to reset identity/sequence
TRUNCATE TABLE test_data RESTART IDENTITY;

-- ✅ Clearing large tables for reload
TRUNCATE TABLE import_staging;
INSERT INTO import_staging SELECT * FROM external_source;

-- ✅ Development/testing (quick cleanup)
TRUNCATE TABLE test_users, test_orders, test_products RESTART IDENTITY CASCADE;
```

#### **Use DROP when:**

```sql
-- ✅ Table no longer needed
DROP TABLE old_archive_2020;

-- ✅ Restructuring database schema
DROP TABLE IF EXISTS legacy_customers;

-- ✅ Removing temporary tables
DROP TABLE IF EXISTS temp_calculations;

-- ✅ Cleanup after migration
DROP TABLE old_format_data CASCADE;
```

---

### **Production Examples:**

#### **Example 1: Data Cleanup Job**

```sql
-- Scenario: Clean up old logs

-- Option 1: DELETE (with condition, keeps some data)
DO $$
BEGIN
    DELETE FROM application_logs 
    WHERE created_at < NOW() - INTERVAL '30 days'
      AND level != 'ERROR';  -- Keep errors longer
    
    RAISE NOTICE 'Deleted % old log entries', ROW_COUNT;
END $$;

-- Option 2: TRUNCATE (remove all, start fresh)
-- Use case: Daily import that replaces all data
BEGIN;
    TRUNCATE TABLE daily_metrics;
    INSERT INTO daily_metrics SELECT * FROM external_metrics_feed;
COMMIT;
```

#### **Example 2: Development Environment Reset**

```sql
-- Reset test database (fast, preserves structure)
TRUNCATE TABLE 
    users, 
    posts, 
    comments, 
    sessions 
RESTART IDENTITY CASCADE;

-- Reload test data
\i test_data.sql
```

#### **Example 3: Schema Migration**

```sql
-- Remove obsolete tables during migration
DO $$
DECLARE
    tbl TEXT;
BEGIN
    FOR tbl IN 
        SELECT tablename 
        FROM pg_tables 
        WHERE schemaname = 'public' 
          AND tablename LIKE 'legacy_%'
    LOOP
        EXECUTE 'DROP TABLE IF EXISTS ' || quote_ident(tbl) || ' CASCADE';
        RAISE NOTICE 'Dropped table: %', tbl;
    END LOOP;
END $$;
```

---

### **Storage & Performance Impact:**

```sql
-- DELETE: Marks rows as dead, requires VACUUM
CREATE TABLE test_delete (id SERIAL, data TEXT);
INSERT INTO test_delete (data) SELECT 'data' FROM generate_series(1, 100000);

DELETE FROM test_delete WHERE id < 50000;

-- Check dead tuples
SELECT 
    n_live_tup,
    n_dead_tup,
    pg_size_pretty(pg_relation_size('test_delete')) AS size
FROM pg_stat_user_tables
WHERE relname = 'test_delete';
-- Shows 50000 live, 50000 dead tuples
-- Size hasn't decreased (dead tuples still occupy space)

-- Reclaim space
VACUUM test_delete;  -- Marks space as reusable
-- Or
VACUUM FULL test_delete;  -- Physically reclaims space (locks table)

-- TRUNCATE: Immediately reclaims space
TRUNCATE TABLE test_delete;
SELECT pg_size_pretty(pg_relation_size('test_delete'));
-- Size: 0 bytes (space immediately reclaimed)
```

---

### **Common Pitfalls:**

```sql
-- ❌ Using DELETE without WHERE (slow)
DELETE FROM large_table;
-- Takes forever, logs every row

-- ✅ Use TRUNCATE instead
TRUNCATE TABLE large_table;

-- ❌ Forgetting CASCADE on TRUNCATE
TRUNCATE TABLE parent_table;
-- ERROR: foreign key constraint

-- ✅ Use CASCADE when needed
TRUNCATE TABLE parent_table CASCADE;

-- ❌ Dropping table without backup
DROP TABLE important_data;
-- Oops, needed that!

-- ✅ Backup first, or use transaction
BEGIN;
    CREATE TABLE important_data_backup AS SELECT * FROM important_data;
    DROP TABLE important_data;
-- Review, then COMMIT or ROLLBACK

-- ❌ Using TRUNCATE when you need audit trail
TRUNCATE TABLE users;
-- No trigger fired, no audit log

-- ✅ Use DELETE for auditing
DELETE FROM users WHERE inactive = true;
-- Triggers fire, audit complete
```

---

### **Interview Tips:**
- Emphasize speed: DELETE (slow), TRUNCATE (fast), DROP (instant)
- Mention WHERE clause: only DELETE supports it
- Discuss triggers: only DELETE fires them
- Explain sequence reset: TRUNCATE resets, DELETE doesn't
- Show understanding of use cases (selective vs all rows vs table removal)
- Mention transaction support (all three can be rolled back)
- Discuss foreign key handling (CASCADE option)

</details>

<details>
<summary>12. How do you find duplicate records in a table?</summary>

### Answer:

Finding duplicate records is a common database task for data quality, deduplication, and cleanup operations. PostgreSQL offers multiple approaches depending on what constitutes a "duplicate" and what you want to do with them.

---

### **Basic Duplicate Detection:**

#### **1. Find Duplicates by Single Column:**

```sql
-- Create sample table with duplicates
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255),
    name VARCHAR(100),
    created_at TIMESTAMPTZ DEFAULT NOW()
);

INSERT INTO users (email, name) VALUES
    ('john@example.com', 'John Doe'),
    ('jane@example.com', 'Jane Smith'),
    ('john@example.com', 'John Duplicate'),  -- Duplicate email
    ('bob@example.com', 'Bob Johnson'),
    ('jane@example.com', 'Jane Duplicate');  -- Duplicate email

-- Find duplicate emails
SELECT 
    email,
    COUNT(*) AS duplicate_count
FROM users
GROUP BY email
HAVING COUNT(*) > 1;

-- Result:
-- email             | duplicate_count
-- ------------------|----------------
-- john@example.com  | 2
-- jane@example.com  | 2
```

#### **2. Find All Duplicate Rows (with details):**

```sql
-- Show all rows that have duplicate emails
SELECT 
    u.*
FROM users u
INNER JOIN (
    SELECT email
    FROM users
    GROUP BY email
    HAVING COUNT(*) > 1
) duplicates ON u.email = duplicates.email
ORDER BY u.email, u.id;

-- Result shows both John and Jane entries with their details
```

---

### **Advanced Duplicate Detection:**

#### **3. Find Duplicates by Multiple Columns:**

```sql
-- Create sample table
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    category TEXT,
    price NUMERIC(10,2),
    supplier TEXT
);

INSERT INTO products (name, category, price, supplier) VALUES
    ('Laptop', 'Electronics', 999.99, 'Supplier A'),
    ('Laptop', 'Electronics', 999.99, 'Supplier A'),  -- Exact duplicate
    ('Laptop', 'Electronics', 899.99, 'Supplier B'),  -- Different price
    ('Mouse', 'Electronics', 29.99, 'Supplier A'),
    ('Mouse', 'Electronics', 29.99, 'Supplier A');    -- Exact duplicate

-- Find duplicates based on name + category + price
SELECT 
    name,
    category,
    price,
    COUNT(*) AS duplicate_count
FROM products
GROUP BY name, category, price
HAVING COUNT(*) > 1;

-- Result:
-- name   | category    | price  | duplicate_count
-- -------|-------------|--------|----------------
-- Laptop | Electronics | 999.99 | 2
-- Mouse  | Electronics | 29.99  | 2
```

#### **4. Using Window Functions (Show Duplicate Number):**

```sql
-- Assign row numbers to duplicates
SELECT 
    id,
    email,
    name,
    ROW_NUMBER() OVER (PARTITION BY email ORDER BY id) AS duplicate_number
FROM users;

-- Result:
-- id | email             | name           | duplicate_number
-- ---|-------------------|----------------|------------------
-- 1  | john@example.com  | John Doe       | 1
-- 3  | john@example.com  | John Duplicate | 2
-- 2  | jane@example.com  | Jane Smith     | 1
-- 5  | jane@example.com  | Jane Duplicate | 2
-- 4  | bob@example.com   | Bob Johnson    | 1

-- Find only the duplicates (not the first occurrence)
SELECT 
    id,
    email,
    name
FROM (
    SELECT 
        *,
        ROW_NUMBER() OVER (PARTITION BY email ORDER BY id) AS rn
    FROM users
) t
WHERE rn > 1;

-- Result shows only John Duplicate and Jane Duplicate
```

---

### **Finding and Removing Duplicates:**

#### **5. Keep First, Delete Rest:**

```sql
-- Delete duplicates, keep the oldest record (lowest ID)
DELETE FROM users
WHERE id NOT IN (
    SELECT MIN(id)
    FROM users
    GROUP BY email
);

-- More efficient using window function
DELETE FROM users
WHERE id IN (
    SELECT id
    FROM (
        SELECT 
            id,
            ROW_NUMBER() OVER (PARTITION BY email ORDER BY id) AS rn
        FROM users
    ) t
    WHERE rn > 1
);
```

#### **6. Keep Most Recent, Delete Rest:**

```sql
-- Delete duplicates, keep the newest record (highest created_at)
DELETE FROM users
WHERE id IN (
    SELECT id
    FROM (
        SELECT 
            id,
            ROW_NUMBER() OVER (
                PARTITION BY email 
                ORDER BY created_at DESC
            ) AS rn
        FROM users
    ) t
    WHERE rn > 1
);
```

#### **7. Safe Approach - Move to Archive First:**

```sql
-- Create archive table
CREATE TABLE users_duplicates AS SELECT * FROM users WHERE false;

-- Move duplicates to archive (before deletion)
WITH duplicates AS (
    SELECT id
    FROM (
        SELECT 
            id,
            ROW_NUMBER() OVER (PARTITION BY email ORDER BY id) AS rn
        FROM users
    ) t
    WHERE rn > 1
)
INSERT INTO users_duplicates
SELECT u.* FROM users u
INNER JOIN duplicates d ON u.id = d.id;

-- Now safe to delete
DELETE FROM users
WHERE id IN (SELECT id FROM users_duplicates);
```

---

### **Preventing Duplicates:**

#### **8. Add UNIQUE Constraint:**

```sql
-- Prevent future duplicates
ALTER TABLE users 
ADD CONSTRAINT unique_email UNIQUE (email);

-- Or create unique index
CREATE UNIQUE INDEX unique_users_email ON users(email);

-- Now duplicates are prevented
INSERT INTO users (email, name) VALUES ('john@example.com', 'Another John');
-- ERROR: duplicate key value violates unique constraint "unique_email"
```

#### **9. Unique Constraint on Multiple Columns:**

```sql
-- Prevent duplicate combinations
ALTER TABLE products
ADD CONSTRAINT unique_product UNIQUE (name, category, supplier);

-- Or partial unique index (only active products)
CREATE UNIQUE INDEX unique_active_products 
ON products(name, category) 
WHERE status = 'active';
```

---

### **Production Examples:**

#### **Example 1: Customer Data Deduplication:**

```sql
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255),
    phone VARCHAR(20),
    name VARCHAR(100),
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Find duplicates by email OR phone
WITH duplicate_emails AS (
    SELECT email
    FROM customers
    WHERE email IS NOT NULL
    GROUP BY email
    HAVING COUNT(*) > 1
),
duplicate_phones AS (
    SELECT phone
    FROM customers
    WHERE phone IS NOT NULL
    GROUP BY phone
    HAVING COUNT(*) > 1
)
SELECT 
    c.*,
    CASE 
        WHEN de.email IS NOT NULL THEN 'Email Duplicate'
        WHEN dp.phone IS NOT NULL THEN 'Phone Duplicate'
    END AS duplicate_type
FROM customers c
LEFT JOIN duplicate_emails de ON c.email = de.email
LEFT JOIN duplicate_phones dp ON c.phone = dp.phone
WHERE de.email IS NOT NULL OR dp.phone IS NOT NULL
ORDER BY c.email, c.phone, c.id;
```

#### **Example 2: Merge Duplicates (Keep Best Data):**

```sql
-- Merge duplicate users, keeping most complete data
WITH ranked_users AS (
    SELECT 
        *,
        ROW_NUMBER() OVER (
            PARTITION BY email 
            ORDER BY 
                -- Prefer records with more data
                (CASE WHEN phone IS NOT NULL THEN 1 ELSE 0 END) +
                (CASE WHEN address IS NOT NULL THEN 1 ELSE 0 END) +
                (CASE WHEN name IS NOT NULL THEN 1 ELSE 0 END) DESC,
                created_at DESC  -- Then most recent
        ) AS rn
    FROM users
)
DELETE FROM users
WHERE id IN (
    SELECT id FROM ranked_users WHERE rn > 1
);
```

#### **Example 3: Report Duplicate Statistics:**

```sql
-- Comprehensive duplicate analysis
SELECT 
    'Email' AS field,
    COUNT(DISTINCT email) AS total_unique,
    SUM(CASE WHEN cnt > 1 THEN cnt - 1 ELSE 0 END) AS total_duplicates,
    SUM(CASE WHEN cnt > 1 THEN 1 ELSE 0 END) AS affected_values
FROM (
    SELECT email, COUNT(*) AS cnt
    FROM users
    WHERE email IS NOT NULL
    GROUP BY email
) t

UNION ALL

SELECT 
    'Phone' AS field,
    COUNT(DISTINCT phone) AS total_unique,
    SUM(CASE WHEN cnt > 1 THEN cnt - 1 ELSE 0 END) AS total_duplicates,
    SUM(CASE WHEN cnt > 1 THEN 1 ELSE 0 END) AS affected_values
FROM (
    SELECT phone, COUNT(*) AS cnt
    FROM users
    WHERE phone IS NOT NULL
    GROUP BY phone
) t;

-- Result:
-- field | total_unique | total_duplicates | affected_values
-- ------|--------------|------------------|----------------
-- Email | 1000         | 50               | 25
-- Phone | 950          | 30               | 15
```

---

### **Performance Optimization:**

```sql
-- For large tables, create indexes first
CREATE INDEX idx_users_email ON users(email);

-- Use EXPLAIN to check query performance
EXPLAIN ANALYZE
SELECT email, COUNT(*) 
FROM users 
GROUP BY email 
HAVING COUNT(*) > 1;

-- For very large tables, use approximate methods
-- Sample-based duplicate detection
SELECT 
    email,
    COUNT(*) AS duplicate_count
FROM users TABLESAMPLE BERNOULLI(10)  -- 10% sample
GROUP BY email
HAVING COUNT(*) > 1;
```

---

### **Using DISTINCT vs GROUP BY:**

```sql
-- DISTINCT: Get unique rows
SELECT DISTINCT email FROM users;

-- GROUP BY: Count duplicates
SELECT email, COUNT(*) FROM users GROUP BY email;

-- DISTINCT ON: Get first row per group (PostgreSQL specific)
SELECT DISTINCT ON (email)
    id, email, name, created_at
FROM users
ORDER BY email, created_at DESC;
-- Returns: One row per email (most recent)
```

---

### **Common Table Expression (CTE) Approach:**

```sql
-- Complex deduplication with multiple criteria
WITH 
-- Step 1: Identify duplicates
duplicates AS (
    SELECT email
    FROM users
    GROUP BY email
    HAVING COUNT(*) > 1
),
-- Step 2: Rank duplicates
ranked AS (
    SELECT 
        u.*,
        ROW_NUMBER() OVER (
            PARTITION BY u.email 
            ORDER BY u.created_at DESC
        ) AS rn
    FROM users u
    INNER JOIN duplicates d ON u.email = d.email
),
-- Step 3: Mark for deletion
to_delete AS (
    SELECT id FROM ranked WHERE rn > 1
)
-- Step 4: Delete (or SELECT to preview)
SELECT * FROM users WHERE id IN (SELECT id FROM to_delete);
-- Change SELECT to DELETE when ready
```

---

### **Interview Tips:**
- Start with simple GROUP BY / HAVING COUNT(*) > 1
- Show window functions for more control (ROW_NUMBER)
- Discuss keeping first vs most recent vs most complete record
- Mention unique constraints for prevention
- Show awareness of performance (indexes on duplicate-check columns)
- Provide safe approach (archive before delete)
- Discuss multi-column duplicates

</details>

<details>
<summary>13. What is the difference between WHERE and HAVING clause?</summary>

### Answer:

**WHERE** and **HAVING** are both used to filter data, but they operate at different stages of query execution and serve distinct purposes. Understanding when to use each is fundamental to writing correct and efficient SQL queries.

---

### **Key Differences:**

| Aspect | WHERE | HAVING |
|--------|-------|--------|
| **Purpose** | Filter rows before grouping | Filter groups after aggregation |
| **Used With** | Any SELECT statement | Only with GROUP BY |
| **Operates On** | Individual rows | Aggregated groups |
| **Can Use** | Column names, basic conditions | Aggregate functions (COUNT, SUM, etc.) |
| **Execution Order** | Before GROUP BY | After GROUP BY |
| **Performance** | Faster (reduces rows early) | Slower (processes more data) |
| **Aggregates** | Cannot use (SUM, COUNT, etc.) | Can use aggregate functions |

---

### **Basic Examples:**

#### **1. WHERE - Filter Rows:**

```sql
-- Create sample table
CREATE TABLE sales (
    id SERIAL PRIMARY KEY,
    product VARCHAR(100),
    category VARCHAR(50),
    amount NUMERIC(10,2),
    sale_date DATE,
    region VARCHAR(50)
);

INSERT INTO sales (product, category, amount, sale_date, region) VALUES
    ('Laptop', 'Electronics', 1200, '2024-01-15', 'North'),
    ('Mouse', 'Electronics', 25, '2024-01-16', 'North'),
    ('Desk', 'Furniture', 350, '2024-01-17', 'South'),
    ('Chair', 'Furniture', 200, '2024-01-18', 'South'),
    ('Monitor', 'Electronics', 400, '2024-01-19', 'North'),
    ('Keyboard', 'Electronics', 75, '2024-01-20', 'East');

-- WHERE: Filter individual rows
SELECT *
FROM sales
WHERE amount > 100;

-- Result: Returns rows where amount > 100
-- (Laptop, Desk, Chair, Monitor)

-- WHERE with multiple conditions
SELECT *
FROM sales
WHERE category = 'Electronics' AND amount > 50;

-- Result: (Laptop, Monitor, Keyboard)
```

#### **2. HAVING - Filter Groups:**

```sql
-- HAVING: Filter aggregated results
SELECT 
    category,
    COUNT(*) AS product_count,
    SUM(amount) AS total_sales
FROM sales
GROUP BY category
HAVING SUM(amount) > 500;

-- Result:
-- category    | product_count | total_sales
-- ------------|---------------|-------------
-- Electronics | 4             | 1700
-- (Furniture total is 550, but not > 500 if we had different data)

-- HAVING with COUNT
SELECT 
    region,
    COUNT(*) AS sale_count
FROM sales
GROUP BY region
HAVING COUNT(*) >= 3;

-- Result:
-- region | sale_count
-- -------|------------
-- North  | 3
-- (Only North has 3+ sales)
```

---

### **Query Execution Order:**

```sql
-- SQL Query Execution Order:
-- 1. FROM       - Get table
-- 2. WHERE      - Filter rows
-- 3. GROUP BY   - Group rows
-- 4. HAVING     - Filter groups
-- 5. SELECT     - Select columns
-- 6. ORDER BY   - Sort results
-- 7. LIMIT      - Limit results

-- Example demonstrating order:
SELECT 
    category,
    COUNT(*) AS product_count,
    AVG(amount) AS avg_amount
FROM sales
WHERE region = 'North'           -- Step 2: Filter to North region only
GROUP BY category                -- Step 3: Group by category
HAVING COUNT(*) > 1              -- Step 4: Filter groups with 2+ products
ORDER BY avg_amount DESC         -- Step 6: Sort by average
LIMIT 5;                        -- Step 7: Limit to 5 results

-- Execution flow:
-- 1. Start with sales table
-- 2. WHERE filters to North region (3 rows)
-- 3. GROUP BY creates groups per category
-- 4. HAVING filters to groups with 2+ products
-- 5. SELECT calculates COUNT and AVG
-- 6. ORDER BY sorts results
-- 7. LIMIT takes top 5
```

---

### **WHERE vs HAVING - Side by Side:**

```sql
-- Example 1: Filter before grouping (WHERE)
SELECT 
    category,
    SUM(amount) AS total_sales
FROM sales
WHERE amount > 100               -- Filter individual rows first
GROUP BY category;

-- Result: Only processes rows where amount > 100
-- Electronics: 1600 (Laptop + Monitor)
-- Furniture: 550 (Desk + Chair)

-- Example 2: Filter after grouping (HAVING)
SELECT 
    category,
    SUM(amount) AS total_sales
FROM sales
GROUP BY category
HAVING SUM(amount) > 500;        -- Filter groups after aggregation

-- Result: Processes all rows, then filters categories
-- Electronics: 1700
-- Furniture: 550

-- Example 3: Both together (most common)
SELECT 
    category,
    COUNT(*) AS product_count,
    SUM(amount) AS total_sales
FROM sales
WHERE sale_date >= '2024-01-15'  -- First: Filter recent sales
GROUP BY category
HAVING SUM(amount) > 300;        -- Then: Filter categories with good sales

-- Execution:
-- 1. WHERE reduces dataset (only recent sales)
-- 2. GROUP BY aggregates remaining rows
-- 3. HAVING filters aggregated results
```

---

### **Common Use Cases:**

#### **Use Case 1: Sales Analysis**

```sql
-- Find high-value customers (orders > $1000 total)
SELECT 
    customer_id,
    COUNT(*) AS order_count,
    SUM(order_total) AS total_spent,
    AVG(order_total) AS avg_order
FROM orders
WHERE status = 'completed'       -- Only completed orders
GROUP BY customer_id
HAVING SUM(order_total) > 1000   -- High-value customers only
ORDER BY total_spent DESC;
```

#### **Use Case 2: Product Performance**

```sql
-- Products with consistently high sales (avg > $500, sold 10+ times)
SELECT 
    product_name,
    COUNT(*) AS times_sold,
    AVG(sale_price) AS avg_price,
    SUM(sale_price) AS total_revenue
FROM product_sales
WHERE sale_date >= CURRENT_DATE - INTERVAL '90 days'  -- Last 3 months
GROUP BY product_name
HAVING 
    COUNT(*) >= 10 AND           -- Sold at least 10 times
    AVG(sale_price) > 500;       -- Average price > $500
```

#### **Use Case 3: Data Quality Check**

```sql
-- Find duplicate emails (users with same email)
SELECT 
    email,
    COUNT(*) AS duplicate_count
FROM users
WHERE email IS NOT NULL          -- Exclude null emails
GROUP BY email
HAVING COUNT(*) > 1              -- Only show duplicates
ORDER BY duplicate_count DESC;
```

---

### **Errors and Misconceptions:**

```sql
-- ❌ ERROR: Can't use aggregate in WHERE
SELECT category, SUM(amount)
FROM sales
WHERE SUM(amount) > 500          -- ERROR!
GROUP BY category;
-- ERROR: aggregate functions are not allowed in WHERE

-- ✅ CORRECT: Use HAVING for aggregates
SELECT category, SUM(amount)
FROM sales
GROUP BY category
HAVING SUM(amount) > 500;        -- Correct!

-- ❌ ERROR: Can't use HAVING without GROUP BY (in most cases)
SELECT *
FROM sales
HAVING amount > 100;             -- ERROR! (No GROUP BY)

-- ✅ CORRECT: Use WHERE for non-aggregated filters
SELECT *
FROM sales
WHERE amount > 100;

-- ⚠️ SPECIAL CASE: HAVING without GROUP BY (entire table as one group)
SELECT 
    COUNT(*) AS total_count,
    SUM(amount) AS total_amount
FROM sales
HAVING SUM(amount) > 2000;
-- Valid! Treats entire table as one group
-- Returns row only if total sales > 2000, otherwise empty result
```

---

### **Performance Considerations:**

```sql
-- Better Performance: Filter with WHERE first
SELECT 
    category,
    COUNT(*) AS product_count
FROM sales
WHERE sale_date >= '2024-01-01'  -- Reduces dataset early
GROUP BY category
HAVING COUNT(*) > 2;

-- Worse Performance: Filter only with HAVING
SELECT 
    category,
    COUNT(*) AS product_count
FROM sales
WHERE true  -- No filtering
GROUP BY category
HAVING 
    COUNT(*) > 2 AND
    MIN(sale_date) >= '2024-01-01';  -- Late filtering

-- Why? First query processes fewer rows during GROUP BY

-- Use EXPLAIN to see difference
EXPLAIN ANALYZE
SELECT category, COUNT(*)
FROM large_sales_table
WHERE region = 'North'           -- Early filter
GROUP BY category
HAVING COUNT(*) > 10;

-- vs

EXPLAIN ANALYZE
SELECT category, COUNT(*)
FROM large_sales_table
GROUP BY category
HAVING COUNT(*) > 10 AND
       MAX(CASE WHEN region = 'North' THEN 1 ELSE 0 END) = 1;  -- Late filter
```

---

### **Complex Example - Combining Both:**

```sql
-- E-commerce: Find popular products in specific category
SELECT 
    p.product_name,
    p.category,
    COUNT(o.id) AS order_count,
    SUM(o.quantity) AS total_sold,
    AVG(o.price) AS avg_price,
    SUM(o.quantity * o.price) AS total_revenue
FROM products p
JOIN order_items o ON p.id = o.product_id
JOIN orders ord ON o.order_id = ord.id
WHERE 
    ord.status = 'delivered' AND              -- WHERE: Filter individual rows
    ord.order_date >= CURRENT_DATE - INTERVAL '6 months' AND
    p.category IN ('Electronics', 'Books')
GROUP BY p.product_name, p.category
HAVING 
    COUNT(o.id) >= 50 AND                     -- HAVING: Filter groups
    SUM(o.quantity * o.price) > 10000
ORDER BY total_revenue DESC
LIMIT 20;

-- Execution:
-- 1. JOIN tables
-- 2. WHERE filters: delivered orders, last 6 months, specific categories
-- 3. GROUP BY: Aggregate by product
-- 4. HAVING filters: Products with 50+ orders AND $10K+ revenue
-- 5. ORDER BY: Sort by revenue
-- 6. LIMIT: Top 20 results
```

---

### **Advanced Patterns:**

```sql
-- Pattern 1: Filter groups by percentage
SELECT 
    region,
    COUNT(*) AS sale_count,
    SUM(amount) AS total_sales,
    ROUND(SUM(amount) * 100.0 / (SELECT SUM(amount) FROM sales), 2) AS pct_of_total
FROM sales
GROUP BY region
HAVING SUM(amount) > (SELECT SUM(amount) * 0.1 FROM sales);  -- Regions with >10% of total

-- Pattern 2: Multiple HAVING conditions
SELECT 
    category,
    COUNT(*) AS product_count,
    AVG(amount) AS avg_amount,
    MIN(amount) AS min_amount,
    MAX(amount) AS max_amount
FROM sales
GROUP BY category
HAVING 
    COUNT(*) >= 3 AND
    AVG(amount) > 100 AND
    MAX(amount) < 2000;

-- Pattern 3: HAVING with subquery
SELECT 
    region,
    SUM(amount) AS total_sales
FROM sales
GROUP BY region
HAVING SUM(amount) > (
    SELECT AVG(region_total) 
    FROM (
        SELECT region, SUM(amount) AS region_total
        FROM sales
        GROUP BY region
    ) t
);
-- Returns regions with above-average total sales
```

---

### **Interview Tips:**
- Emphasize execution order: WHERE → GROUP BY → HAVING
- WHERE filters rows, HAVING filters groups
- WHERE cannot use aggregates, HAVING can
- Performance: WHERE is faster (filters early)
- Show both clauses together in queries
- Mention HAVING can work without GROUP BY (special case)
- Provide real-world examples (sales analysis, duplicates)

</details>

<details>
<summary>14. Explain the LIMIT and OFFSET clauses.</summary>

### Answer:

**LIMIT** and **OFFSET** are clauses used to control the number of rows returned by a query and to implement pagination. They're essential for handling large result sets efficiently and building paginated interfaces.

---

### **Basic Syntax:**

```sql
SELECT columns
FROM table
ORDER BY column
LIMIT number_of_rows OFFSET starting_position;

-- Or using FETCH (SQL standard alternative)
SELECT columns
FROM table
ORDER BY column
OFFSET starting_position ROWS
FETCH FIRST number_of_rows ROWS ONLY;
```

---

### **LIMIT - Restrict Number of Rows:**

```sql
-- Create sample table
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    price NUMERIC(10,2),
    category TEXT,
    rating NUMERIC(3,2)
);

INSERT INTO products (name, price, category, rating) VALUES
    ('Laptop Pro', 1299.99, 'Electronics', 4.5),
    ('Mouse Wireless', 29.99, 'Electronics', 4.2),
    ('Desk Chair', 199.99, 'Furniture', 4.7),
    ('Monitor 27"', 349.99, 'Electronics', 4.6),
    ('Keyboard Mechanical', 89.99, 'Electronics', 4.4),
    ('Standing Desk', 449.99, 'Furniture', 4.8),
    ('Webcam HD', 79.99, 'Electronics', 4.3),
    ('Desk Lamp', 39.99, 'Furniture', 4.1),
    ('USB Hub', 24.99, 'Electronics', 4.0),
    ('Bookshelf', 129.99, 'Furniture', 4.5);

-- Get first 5 products
SELECT * FROM products
ORDER BY id
LIMIT 5;

-- Result: Returns products with id 1-5

-- Get top 3 most expensive products
SELECT name, price
FROM products
ORDER BY price DESC
LIMIT 3;

-- Result:
-- name          | price
-- --------------|--------
-- Laptop Pro    | 1299.99
-- Standing Desk | 449.99
-- Monitor 27"   | 349.99

-- Get highest rated product
SELECT name, rating
FROM products
ORDER BY rating DESC
LIMIT 1;

-- Result: Standing Desk (4.8)
```

---

### **OFFSET - Skip Rows:**

```sql
-- Skip first 3 products, get next 5
SELECT name, price
FROM products
ORDER BY id
LIMIT 5 OFFSET 3;

-- Result: Returns products 4-8 (skips 1-3)

-- Alternative syntax (skip first 3)
SELECT name, price
FROM products
ORDER BY id
OFFSET 3;
-- Returns all rows starting from row 4

-- Get second page (rows 6-10)
SELECT name, price
FROM products
ORDER BY id
LIMIT 5 OFFSET 5;

-- Explanation:
-- OFFSET 5 means skip first 5 rows
-- LIMIT 5 means return next 5 rows
-- So returns rows 6-10
```

---

### **Pagination Pattern:**

```sql
-- Pagination formula:
-- LIMIT = page_size
-- OFFSET = (page_number - 1) * page_size

-- Page 1 (items 1-10)
SELECT * FROM products
ORDER BY id
LIMIT 10 OFFSET 0;

-- Page 2 (items 11-20)
SELECT * FROM products
ORDER BY id
LIMIT 10 OFFSET 10;

-- Page 3 (items 21-30)
SELECT * FROM products
ORDER BY id
LIMIT 10 OFFSET 20;

-- Generic pagination query
DO $$
DECLARE
    page_size INTEGER := 5;
    page_number INTEGER := 2;
    offset_value INTEGER;
BEGIN
    offset_value := (page_number - 1) * page_size;
    
    RAISE NOTICE 'Page %, Size %, Offset %', page_number, page_size, offset_value;
    
    -- Execute query
    PERFORM * FROM products
    ORDER BY id
    LIMIT page_size OFFSET offset_value;
END $$;
```

---

### **Practical Examples:**

#### **Example 1: E-commerce Product Listing**

```sql
-- Product listing with pagination
CREATE OR REPLACE FUNCTION get_products_page(
    p_page INTEGER,
    p_page_size INTEGER,
    p_category TEXT DEFAULT NULL
)
RETURNS TABLE(
    product_id INTEGER,
    product_name TEXT,
    price NUMERIC,
    rating NUMERIC,
    total_count BIGINT
) AS $$
DECLARE
    offset_val INTEGER;
BEGIN
    offset_val := (p_page - 1) * p_page_size;
    
    RETURN QUERY
    SELECT 
        p.id,
        p.name,
        p.price,
        p.rating,
        COUNT(*) OVER() AS total_count  -- Total for pagination info
    FROM products p
    WHERE p_category IS NULL OR p.category = p_category
    ORDER BY p.rating DESC, p.name
    LIMIT p_page_size
    OFFSET offset_val;
END;
$$ LANGUAGE plpgsql;

-- Usage
SELECT * FROM get_products_page(1, 5, 'Electronics');
-- Returns page 1, 5 items, Electronics only

SELECT * FROM get_products_page(2, 5, NULL);
-- Returns page 2, 5 items, all categories
```

#### **Example 2: API Response with Metadata**

```sql
-- Complete pagination response
WITH paginated_results AS (
    SELECT 
        *,
        COUNT(*) OVER() AS total_count
    FROM products
    WHERE category = 'Electronics'
    ORDER BY price DESC
    LIMIT 5 OFFSET 0
)
SELECT 
    json_build_object(
        'data', json_agg(
            json_build_object(
                'id', id,
                'name', name,
                'price', price,
                'rating', rating
            )
        ),
        'pagination', json_build_object(
            'page', 1,
            'page_size', 5,
            'total_items', MAX(total_count),
            'total_pages', CEIL(MAX(total_count) / 5.0)
        )
    ) AS response
FROM paginated_results;

-- Returns JSON:
-- {
--   "data": [...products...],
--   "pagination": {
--     "page": 1,
--     "page_size": 5,
--     "total_items": 15,
--     "total_pages": 3
--   }
-- }
```

#### **Example 3: Leaderboard (Top Performers)**

```sql
-- Top 10 users by score
SELECT 
    rank() OVER (ORDER BY score DESC) AS rank,
    username,
    score,
    level
FROM users
ORDER BY score DESC
LIMIT 10;

-- Get user's position in leaderboard
WITH ranked_users AS (
    SELECT 
        id,
        username,
        score,
        ROW_NUMBER() OVER (ORDER BY score DESC) AS position
    FROM users
)
SELECT 
    username,
    score,
    position,
    (SELECT COUNT(*) FROM users) AS total_users
FROM ranked_users
WHERE id = 12345;  -- Specific user
```

---

### **Performance Considerations:**

```sql
-- ⚠️ OFFSET is inefficient for large offsets
-- Problem: Database must scan and skip all offset rows

-- Bad: Deep pagination (page 1000)
SELECT * FROM large_table
ORDER BY created_at
LIMIT 20 OFFSET 20000;
-- Scans 20,020 rows to return 20!

-- ✅ Better: Keyset pagination (cursor-based)
-- Instead of OFFSET, use WHERE with last seen value

-- First page
SELECT * FROM large_table
ORDER BY created_at, id
LIMIT 20;

-- Next page (use last_created_at and last_id from previous result)
SELECT * FROM large_table
WHERE (created_at, id) > ('2024-01-15 10:30:00', 12345)
ORDER BY created_at, id
LIMIT 20;

-- Benefits:
-- - Consistent performance regardless of page
-- - No skipping rows
-- - Works well with real-time data
```

**Performance comparison:**
```sql
-- Create large table
CREATE TABLE large_products AS
SELECT 
    i AS id,
    'Product ' || i AS name,
    (random() * 1000)::numeric(10,2) AS price
FROM generate_series(1, 1000000) i;

CREATE INDEX idx_large_products_id ON large_products(id);

-- Slow: Large OFFSET
EXPLAIN ANALYZE
SELECT * FROM large_products
ORDER BY id
LIMIT 20 OFFSET 500000;
-- Execution Time: ~200ms (scans 500,020 rows)

-- Fast: Keyset pagination
EXPLAIN ANALYZE
SELECT * FROM large_products
WHERE id > 500000
ORDER BY id
LIMIT 20;
-- Execution Time: ~1ms (uses index, scans only 20 rows)
```

---

### **FETCH - SQL Standard Alternative:**

```sql
-- PostgreSQL supports SQL standard FETCH syntax

-- LIMIT 5
SELECT * FROM products
ORDER BY price
FETCH FIRST 5 ROWS ONLY;

-- LIMIT 5 OFFSET 10
SELECT * FROM products
ORDER BY price
OFFSET 10 ROWS
FETCH NEXT 5 ROWS ONLY;

-- Equivalent to LIMIT/OFFSET but more verbose
-- Use LIMIT/OFFSET for brevity in PostgreSQL
```

---

### **Edge Cases and Special Scenarios:**

```sql
-- NULL in LIMIT/OFFSET
SELECT * FROM products
LIMIT NULL;
-- Returns all rows (same as no LIMIT)

SELECT * FROM products
LIMIT NULL OFFSET 5;
-- Skips first 5, returns all remaining

-- LIMIT 0 (useful for schema inspection)
SELECT * FROM products
LIMIT 0;
-- Returns no rows, but defines result structure

-- OFFSET larger than row count
SELECT * FROM products
LIMIT 5 OFFSET 1000;
-- Returns empty result (no error)

-- Negative LIMIT/OFFSET (not allowed)
SELECT * FROM products
LIMIT -5;
-- ERROR: LIMIT must not be negative

-- LIMIT with expressions
SELECT * FROM products
LIMIT (SELECT COUNT(*) / 2 FROM products);
-- Returns half the rows

-- Dynamic LIMIT from parameter
PREPARE get_top_n (INTEGER) AS
    SELECT * FROM products
    ORDER BY rating DESC
    LIMIT $1;

EXECUTE get_top_n(5);  -- Get top 5
EXECUTE get_top_n(10); -- Get top 10
```

---

### **Combining with Other Clauses:**

```sql
-- LIMIT with WHERE
SELECT name, price
FROM products
WHERE category = 'Electronics'
ORDER BY price DESC
LIMIT 3;

-- LIMIT with JOIN
SELECT 
    c.name AS customer,
    COUNT(o.id) AS order_count
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
GROUP BY c.id, c.name
ORDER BY order_count DESC
LIMIT 10;
-- Top 10 customers by order count

-- LIMIT with subquery
SELECT *
FROM (
    SELECT 
        category,
        name,
        price,
        ROW_NUMBER() OVER (PARTITION BY category ORDER BY price DESC) AS rn
    FROM products
) t
WHERE rn <= 3;
-- Top 3 products per category

-- LIMIT with UNION
(SELECT 'High' AS price_range, * FROM products WHERE price > 500 ORDER BY price DESC LIMIT 5)
UNION ALL
(SELECT 'Low' AS price_range, * FROM products WHERE price <= 100 ORDER BY price LIMIT 5);
-- Top 5 from each price range
```

---

### **Random Sample with LIMIT:**

```sql
-- Get random products
SELECT * FROM products
ORDER BY RANDOM()
LIMIT 5;

-- More efficient for large tables
SELECT * FROM products
WHERE id IN (
    SELECT id FROM products
    ORDER BY RANDOM()
    LIMIT 5
);

-- Reservoir sampling for large datasets
SELECT * FROM products
TABLESAMPLE BERNOULLI(10)  -- 10% sample
LIMIT 100;
```

---

### **Common Pitfalls:**

```sql
-- ❌ LIMIT without ORDER BY (unpredictable results)
SELECT * FROM products
LIMIT 5;
-- Returns 5 rows, but which ones? Undefined!
-- Results may differ between queries

-- ✅ Always use ORDER BY with LIMIT
SELECT * FROM products
ORDER BY id
LIMIT 5;
-- Deterministic results

-- ❌ Using OFFSET for deep pagination
SELECT * FROM products
ORDER BY created_at
LIMIT 20 OFFSET 1000000;
-- Very slow! Scans 1,000,020 rows

-- ✅ Use keyset pagination
SELECT * FROM products
WHERE created_at > '2024-01-15 10:00:00'
ORDER BY created_at
LIMIT 20;

-- ❌ Not considering total count for pagination
SELECT * FROM products
LIMIT 10 OFFSET 20;
-- How many total pages? Unknown!

-- ✅ Include total count
SELECT 
    *,
    COUNT(*) OVER() AS total_count
FROM products
ORDER BY id
LIMIT 10 OFFSET 20;
```

---

### **Real-World Application Pattern:**

```sql
-- Complete pagination helper view
CREATE OR REPLACE VIEW paginated_products AS
WITH product_data AS (
    SELECT 
        p.*,
        COUNT(*) OVER() AS total_count,
        ROW_NUMBER() OVER(ORDER BY p.rating DESC, p.name) AS row_num
    FROM products p
)
SELECT 
    id,
    name,
    price,
    category,
    rating,
    row_num,
    total_count,
    CEIL(total_count / 10.0) AS total_pages,
    CEIL(row_num / 10.0) AS current_page
FROM product_data;

-- Query specific page
SELECT * FROM paginated_products
WHERE current_page = 2;

-- Or use traditional LIMIT/OFFSET
SELECT 
    id, name, price, category, rating,
    total_count,
    total_pages,
    2 AS current_page
FROM paginated_products
LIMIT 10 OFFSET 10;
```

---

### **Interview Tips:**
- Explain LIMIT restricts rows, OFFSET skips rows
- Emphasize importance of ORDER BY for deterministic results
- Discuss pagination formula: OFFSET = (page - 1) * page_size
- Mention performance issues with large OFFSET (O(n+offset))
- Show keyset pagination as better alternative for deep pages
- Provide complete pagination with total count
- Mention SQL standard FETCH as alternative

</details>

<details>
<summary>15. What is the difference between UNION and UNION ALL?</summary>

### Answer:

**UNION** and **UNION ALL** are set operators that combine results from multiple SELECT statements. The key difference is that **UNION** removes duplicates while **UNION ALL** keeps all rows including duplicates.

---

### **Key Differences:**

| Feature | UNION | UNION ALL |
|---------|-------|-----------|
| **Duplicates** | Removes duplicates | Keeps all duplicates |
| **Performance** | Slower (sorting required) | Faster (no deduplication) |
| **Use Case** | Need unique results | Need all results |
| **Sorting** | Implicit sort for dedup | No sorting |
| **Memory** | More memory usage | Less memory usage |
| **Result Count** | ≤ sum of all queries | = sum of all queries |

---

### **Basic Examples:**

```sql
-- Create sample tables
CREATE TABLE customers_north (
    id INTEGER,
    name VARCHAR(100),
    region VARCHAR(50)
);

CREATE TABLE customers_south (
    id INTEGER,
    name VARCHAR(100),
    region VARCHAR(50)
);

INSERT INTO customers_north VALUES
    (1, 'John Doe', 'North'),
    (2, 'Jane Smith', 'North'),
    (3, 'Bob Johnson', 'North');

INSERT INTO customers_south VALUES
    (3, 'Bob Johnson', 'South'),    -- Duplicate person
    (4, 'Alice Brown', 'South'),
    (5, 'Charlie Davis', 'South');

-- UNION: Removes duplicates
SELECT id, name FROM customers_north
UNION
SELECT id, name FROM customers_south
ORDER BY id;

-- Result: 5 rows (Bob Johnson appears once)
-- id | name
-- ---|-------------
-- 1  | John Doe
-- 2  | Jane Smith
-- 3  | Bob Johnson
-- 4  | Alice Brown
-- 5  | Charlie Davis

-- UNION ALL: Keeps duplicates
SELECT id, name FROM customers_north
UNION ALL
SELECT id, name FROM customers_south
ORDER BY id;

-- Result: 6 rows (Bob Johnson appears twice)
-- id | name
-- ---|-------------
-- 1  | John Doe
-- 2  | Jane Smith
-- 3  | Bob Johnson
-- 3  | Bob Johnson  ← Duplicate
-- 4  | Alice Brown
-- 5  | Charlie Davis
```

---

### **Performance Comparison:**

```sql
-- Create large test tables
CREATE TABLE table_a AS
SELECT i AS id, 'Name_' || i AS name
FROM generate_series(1, 100000) i;

CREATE TABLE table_b AS
SELECT i AS id, 'Name_' || i AS name
FROM generate_series(50000, 150000) i;  -- 50% overlap

-- UNION (slower - removes duplicates)
EXPLAIN ANALYZE
SELECT * FROM table_a
UNION
SELECT * FROM table_b;

-- Execution time: ~500ms
-- Steps: 1. Scan both tables
--        2. Sort and remove duplicates
--        3. Return unique results

-- UNION ALL (faster - keeps all)
EXPLAIN ANALYZE
SELECT * FROM table_a
UNION ALL
SELECT * FROM table_b;

-- Execution time: ~100ms
-- Steps: 1. Scan both tables
--        2. Concatenate results
--        3. Return all rows

-- Performance difference increases with:
-- - Larger datasets
-- - More overlapping data
-- - Wider row structures
```

---

### **When to Use Each:**

#### **Use UNION when:**

```sql
-- ✅ Need unique results across multiple sources
-- Example: Combined mailing list (no duplicates)
SELECT email FROM newsletter_subscribers
UNION
SELECT email FROM customers
UNION
SELECT email FROM event_attendees;

-- ✅ Merging data from different time periods/tables
SELECT * FROM orders_2023
UNION
SELECT * FROM orders_2024;
-- Ensures no duplicate orders if data overlaps

-- ✅ Combining similar data with potential duplicates
SELECT product_id FROM cart_items
UNION
SELECT product_id FROM wishlist_items;
-- Unique products across cart and wishlist
```

#### **Use UNION ALL when:**

```sql
-- ✅ Know there are no duplicates
SELECT * FROM orders_2023  -- Jan-Dec 2023
UNION ALL
SELECT * FROM orders_2024;  -- Jan-Dec 2024
-- No overlap possible, faster

-- ✅ Want to keep all occurrences (even duplicates)
SELECT 'Error' AS type, message FROM error_logs
UNION ALL
SELECT 'Warning' AS type, message FROM warning_logs;
-- Keep all logs, even if message is same

-- ✅ Aggregating data from multiple sources
SELECT user_id, 'Login' AS action FROM login_events
UNION ALL
SELECT user_id, 'Purchase' AS action FROM purchase_events
UNION ALL
SELECT user_id, 'Logout' AS action FROM logout_events;
-- All events needed for timeline

-- ✅ Performance-critical queries (no deduplication overhead)
SELECT * FROM large_table_partition_1
UNION ALL
SELECT * FROM large_table_partition_2
UNION ALL
SELECT * FROM large_table_partition_3;
```

---

### **Rules and Requirements:**

```sql
-- 1. Same number of columns
-- ❌ Error: different column counts
SELECT id, name FROM customers_north
UNION
SELECT id FROM customers_south;
-- ERROR: each UNION query must have same number of columns

-- ✅ Correct: same column count
SELECT id, name FROM customers_north
UNION
SELECT id, name FROM customers_south;

-- 2. Compatible data types
-- ❌ Error: incompatible types
SELECT id, name FROM customers
UNION
SELECT name, id FROM employees;  -- Reversed order
-- ERROR: data type mismatch

-- ✅ Correct: compatible types (cast if needed)
SELECT id::TEXT, name FROM customers
UNION
SELECT employee_code, full_name FROM employees;

-- 3. Column names from first query
SELECT id AS customer_id, name AS customer_name FROM customers_north
UNION
SELECT id, name FROM customers_south;
-- Result columns: customer_id, customer_name

-- 4. ORDER BY at the end (applies to final result)
SELECT id, name FROM customers_north
UNION
SELECT id, name FROM customers_south
ORDER BY name;  -- Sorts combined result

-- ❌ Can't order individual queries
SELECT id, name FROM customers_north ORDER BY id  -- Not allowed
UNION
SELECT id, name FROM customers_south;

-- ✅ Use subqueries if needed
(SELECT id, name FROM customers_north ORDER BY id LIMIT 5)
UNION ALL
(SELECT id, name FROM customers_south ORDER BY id LIMIT 5);
```

---

### **Practical Examples:**

#### **Example 1: Comprehensive Customer List**

```sql
-- Get all customer contacts (no duplicates)
SELECT 
    email,
    'Newsletter' AS source,
    subscribed_at AS date
FROM newsletter_subscribers
WHERE active = true

UNION

SELECT 
    email,
    'Purchase' AS source,
    MAX(order_date) AS date
FROM orders
GROUP BY email

UNION

SELECT 
    email,
    'Support' AS source,
    MAX(ticket_date) AS date
FROM support_tickets
GROUP BY email

ORDER BY email;
-- Result: Unique emails across all sources
```

#### **Example 2: Activity Timeline**

```sql
-- User activity timeline (keep all events)
SELECT 
    user_id,
    'Login' AS event_type,
    login_time AS event_time,
    ip_address AS details
FROM user_logins
WHERE user_id = 12345

UNION ALL

SELECT 
    user_id,
    'Page View' AS event_type,
    view_time AS event_time,
    page_url AS details
FROM page_views
WHERE user_id = 12345

UNION ALL

SELECT 
    user_id,
    'Purchase' AS event_type,
    purchase_time AS event_time,
    order_id::TEXT AS details
FROM purchases
WHERE user_id = 12345

ORDER BY event_time DESC;
-- Chronological activity log (all events)
```

#### **Example 3: Search Across Multiple Tables**

```sql
-- Search for "laptop" across products, articles, and FAQs
SELECT 
    'Product' AS result_type,
    id,
    name AS title,
    price::TEXT AS extra_info
FROM products
WHERE name ILIKE '%laptop%'

UNION ALL

SELECT 
    'Article' AS result_type,
    id,
    title,
    author AS extra_info
FROM articles
WHERE content ILIKE '%laptop%'

UNION ALL

SELECT 
    'FAQ' AS result_type,
    id,
    question AS title,
    NULL AS extra_info
FROM faqs
WHERE question ILIKE '%laptop%' OR answer ILIKE '%laptop%'

ORDER BY result_type, id
LIMIT 50;
```

#### **Example 4: Reporting from Partitioned Tables**

```sql
-- Query across time-partitioned tables
SELECT * FROM sales_q1_2024
UNION ALL
SELECT * FROM sales_q2_2024
UNION ALL
SELECT * FROM sales_q3_2024
UNION ALL
SELECT * FROM sales_q4_2024;

-- Faster than UNION (no duplicate check needed)
-- Partitions don't overlap by design
```

---

### **Multiple UNION Operations:**

```sql
-- Chain multiple UNIONs
SELECT id, name FROM customers_north
UNION
SELECT id, name FROM customers_south
UNION
SELECT id, name FROM customers_east
UNION
SELECT id, name FROM customers_west;
-- All unique customers from all regions

-- Mix UNION and UNION ALL
SELECT id, name FROM active_customers
UNION ALL  -- Keep all active
SELECT id, name FROM inactive_customers
UNION      -- Remove duplicates between inactive and archived
SELECT id, name FROM archived_customers;

-- Precedence: Evaluated left to right
-- Use parentheses for clarity
(
    SELECT id FROM table_a
    UNION
    SELECT id FROM table_b
)
UNION ALL
SELECT id FROM table_c;
```

---

### **UNION with Aggregates:**

```sql
-- Combine aggregated results
SELECT 
    'Q1' AS quarter,
    SUM(revenue) AS total_revenue,
    COUNT(*) AS order_count
FROM orders_q1
UNION ALL
SELECT 
    'Q2' AS quarter,
    SUM(revenue) AS total_revenue,
    COUNT(*) AS order_count
FROM orders_q2
UNION ALL
SELECT 
    'Q3' AS quarter,
    SUM(revenue) AS total_revenue,
    COUNT(*) AS order_count
FROM orders_q3
UNION ALL
SELECT 
    'Q4' AS quarter,
    SUM(revenue) AS total_revenue,
    COUNT(*) AS order_count
FROM orders_q4;

-- Result: Quarterly summary
-- quarter | total_revenue | order_count
-- --------|---------------|-------------
-- Q1      | 150000.00     | 1200
-- Q2      | 175000.00     | 1350
-- Q3      | 200000.00     | 1500
-- Q4      | 225000.00     | 1600
```

---

### **Common Mistakes:**

```sql
-- ❌ Using UNION when UNION ALL is appropriate
SELECT * FROM large_table_part1
UNION  -- Unnecessary deduplication overhead
SELECT * FROM large_table_part2;

-- ✅ Use UNION ALL for partitioned tables
SELECT * FROM large_table_part1
UNION ALL
SELECT * FROM large_table_part2;

-- ❌ Forgetting column count/type compatibility
SELECT id, name, email FROM table1
UNION
SELECT id, name FROM table2;  -- Error!

-- ✅ Add NULL placeholders
SELECT id, name, email FROM table1
UNION
SELECT id, name, NULL AS email FROM table2;

-- ❌ Inefficient duplicate removal
SELECT DISTINCT * FROM (
    SELECT * FROM table1
    UNION ALL
    SELECT * FROM table2
) t;
-- Just use UNION directly!

-- ✅ Use UNION
SELECT * FROM table1
UNION
SELECT * FROM table2;
```

---

### **Alternative Set Operators:**

```sql
-- INTERSECT: Common rows only
SELECT id FROM customers_2023
INTERSECT
SELECT id FROM customers_2024;
-- Customers in both years

-- EXCEPT: Rows in first, not in second
SELECT id FROM customers_2023
EXCEPT
SELECT id FROM customers_2024;
-- Customers only in 2023 (churned)

-- INTERSECT ALL and EXCEPT ALL (keep duplicates)
SELECT id FROM orders_batch1
INTERSECT ALL
SELECT id FROM orders_batch2;
```

---

### **Interview Tips:**
- Emphasize UNION removes duplicates, UNION ALL keeps all
- Mention UNION ALL is faster (no deduplication overhead)
- Explain when to use each (unique vs all data)
- Discuss requirements: same column count, compatible types
- Show performance impact on large datasets
- Mention ORDER BY applies to final combined result
- Provide practical examples (mailing lists, partitioned tables, activity logs)

</details>

<details>
<summary>16. How do you perform case-insensitive searches in PostgreSQL?</summary>

### Answer:

PostgreSQL provides multiple methods for case-insensitive string matching, each with different performance characteristics and use cases. Choosing the right approach depends on data size, query frequency, and specific requirements.

---

### **Methods for Case-Insensitive Search:**

---

### **1. ILIKE Operator (PostgreSQL-specific):**

Most straightforward method for case-insensitive pattern matching.

```sql
-- Create sample table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(100),
    email VARCHAR(255),
    full_name TEXT
);

INSERT INTO users (username, email, full_name) VALUES
    ('JohnDoe', 'john@example.com', 'John Doe'),
    ('JANEDOE', 'jane@EXAMPLE.com', 'Jane Doe'),
    ('bobsmith', 'Bob@Example.COM', 'Bob Smith'),
    ('AliceW', 'alice@example.com', 'Alice Williams');

-- ILIKE: Case-insensitive LIKE
SELECT * FROM users
WHERE username ILIKE 'johndoe';
-- Finds: JohnDoe

SELECT * FROM users
WHERE email ILIKE '%EXAMPLE%';
-- Finds: all emails with 'example' (any case)

-- Pattern matching with ILIKE
SELECT * FROM users
WHERE full_name ILIKE 'j%';
-- Finds: John Doe, Jane Doe

SELECT * FROM users
WHERE username ILIKE '%doe';
-- Finds: JohnDoe, JANEDOE
```

---

### **2. LOWER() / UPPER() Functions:**

Convert both sides to same case for comparison.

```sql
-- Using LOWER()
SELECT * FROM users
WHERE LOWER(username) = LOWER('JohnDoe');
-- Finds: JohnDoe

SELECT * FROM users
WHERE LOWER(email) = 'john@example.com';
-- Finds: john@example.com

-- Using UPPER()
SELECT * FROM users
WHERE UPPER(full_name) = UPPER('bob smith');
-- Finds: Bob Smith

-- Pattern matching with LOWER()
SELECT * FROM users
WHERE LOWER(username) LIKE '%doe%';
-- Finds: JohnDoe, JANEDOE
```

---

### **3. Case-Insensitive Collation:**

Use collations for linguistic-aware comparisons.

```sql
-- Using COLLATE (recommended for regular comparisons)
SELECT * FROM users
WHERE username = 'johndoe' COLLATE "en_US.utf8";
-- Case-insensitive in many locales

-- More explicit case-insensitive collation (PostgreSQL 12+)
SELECT * FROM users
WHERE username COLLATE "en-US-x-icu" = 'JOHNDOE';

-- Create column with case-insensitive collation
CREATE TABLE users_ci (
    id SERIAL PRIMARY KEY,
    username VARCHAR(100) COLLATE "case_insensitive",
    email VARCHAR(255)
);

-- Note: Collation support varies by PostgreSQL version and locale
```

---

### **4. CITEXT Extension (Best for frequent searches):**

Dedicated case-insensitive text type.

```sql
-- Enable extension
CREATE EXTENSION IF NOT EXISTS citext;

-- Create table with citext
CREATE TABLE users_citext (
    id SERIAL PRIMARY KEY,
    username CITEXT,
    email CITEXT,
    full_name CITEXT
);

INSERT INTO users_citext (username, email, full_name) VALUES
    ('JohnDoe', 'john@example.com', 'John Doe'),
    ('JANEDOE', 'jane@EXAMPLE.com', 'Jane Doe');

-- Now all comparisons are automatically case-insensitive
SELECT * FROM users_citext
WHERE username = 'johndoe';  -- Finds: JohnDoe

SELECT * FROM users_citext
WHERE email = 'JOHN@EXAMPLE.COM';  -- Finds: john@example.com

-- Pattern matching also case-insensitive
SELECT * FROM users_citext
WHERE username LIKE 'john%';  -- Finds: JohnDoe

-- Create indexes on citext (works normally)
CREATE INDEX idx_users_citext_email ON users_citext(email);
```

---

### **5. Regular Expressions (~* operator):**

Case-insensitive regex matching.

```sql
-- ~*: Case-insensitive regex match
SELECT * FROM users
WHERE username ~* '^john';
-- Finds: usernames starting with 'john' (any case)

SELECT * FROM users
WHERE email ~* '^[a-z]+@example\.com$';
-- Finds: emails matching pattern (case-insensitive)

-- More complex patterns
SELECT * FROM users
WHERE full_name ~* 'doe|smith';
-- Finds: names containing 'doe' or 'smith' (any case)

-- Match any position
SELECT * FROM users
WHERE username ~* 'doe';
-- Finds: JohnDoe, JANEDOE
```

---

### **Performance Comparison:**

```sql
-- Create large test table
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    name_citext CITEXT
);

-- Insert 100,000 products
INSERT INTO products (name, name_citext)
SELECT 
    'Product ' || i,
    'Product ' || i
FROM generate_series(1, 100000) i;

-- Method 1: ILIKE (no index optimization)
EXPLAIN ANALYZE
SELECT * FROM products
WHERE name ILIKE 'product 50000';
-- Seq Scan, ~100ms (full table scan)

-- Method 2: LOWER with expression index
CREATE INDEX idx_products_name_lower ON products(LOWER(name));

EXPLAIN ANALYZE
SELECT * FROM products
WHERE LOWER(name) = LOWER('Product 50000');
-- Index Scan, ~1ms (uses index)

-- Method 3: CITEXT with regular index
CREATE INDEX idx_products_name_citext ON products(name_citext);

EXPLAIN ANALYZE
SELECT * FROM products
WHERE name_citext = 'PRODUCT 50000';
-- Index Scan, ~1ms (uses index, automatically case-insensitive)
```

---

### **Indexing Strategies:**

#### **Expression Index for LOWER():**

```sql
-- Create expression index
CREATE INDEX idx_users_email_lower 
ON users(LOWER(email));

-- Query must match expression exactly
SELECT * FROM users
WHERE LOWER(email) = 'john@example.com';
-- Uses index

-- This won't use index
SELECT * FROM users
WHERE UPPER(email) = 'JOHN@EXAMPLE.COM';
-- Different expression, no index match
```

#### **GIN Index for Pattern Matching:**

```sql
-- Install extension
CREATE EXTENSION IF NOT EXISTS pg_trgm;

-- Create GIN index for ILIKE
CREATE INDEX idx_users_username_trgm 
ON users USING GIN(username gin_trgm_ops);

-- Now ILIKE uses index
EXPLAIN ANALYZE
SELECT * FROM users
WHERE username ILIKE '%doe%';
-- Uses GIN index with trigram matching
```

#### **CITEXT with B-tree Index:**

```sql
-- Regular index on citext
CREATE INDEX idx_users_citext_username 
ON users_citext(username);

-- All queries automatically use index
SELECT * FROM users_citext WHERE username = 'johndoe';
SELECT * FROM users_citext WHERE username = 'JOHNDOE';
SELECT * FROM users_citext WHERE username = 'JohnDoe';
-- All use the same index
```

---

### **Practical Examples:**

#### **Example 1: User Login (Case-Insensitive Email)**

```sql
-- Login function with case-insensitive email
CREATE OR REPLACE FUNCTION authenticate_user(
    p_email TEXT,
    p_password TEXT
)
RETURNS TABLE(user_id INTEGER, username TEXT) AS $$
BEGIN
    RETURN QUERY
    SELECT id, username
    FROM users
    WHERE LOWER(email) = LOWER(p_email)
      AND password_hash = crypt(p_password, password_hash);
END;
$$ LANGUAGE plpgsql;

-- Usage (email in any case works)
SELECT * FROM authenticate_user('JOHN@EXAMPLE.COM', 'password123');
SELECT * FROM authenticate_user('john@example.com', 'password123');
-- Both work
```

#### **Example 2: Product Search**

```sql
-- Multi-field case-insensitive search
SELECT 
    id,
    name,
    description,
    price
FROM products
WHERE 
    LOWER(name) LIKE LOWER('%laptop%') OR
    LOWER(description) LIKE LOWER('%laptop%') OR
    LOWER(category) LIKE LOWER('%laptop%')
ORDER BY 
    CASE 
        WHEN LOWER(name) LIKE LOWER('%laptop%') THEN 1
        WHEN LOWER(description) LIKE LOWER('%laptop%') THEN 2
        ELSE 3
    END,
    price;
-- Prioritizes name matches, then description
```

#### **Example 3: Unique Constraint (Case-Insensitive)**

```sql
-- Prevent duplicate emails (case-insensitive)
CREATE UNIQUE INDEX unique_email_ci 
ON users(LOWER(email));

-- Now these fail (duplicate)
INSERT INTO users (email) VALUES ('john@example.com');
INSERT INTO users (email) VALUES ('JOHN@EXAMPLE.COM');
-- ERROR: duplicate key value

-- Alternative: Use CITEXT
CREATE TABLE users_ci (
    id SERIAL PRIMARY KEY,
    email CITEXT UNIQUE
);

INSERT INTO users_ci (email) VALUES ('john@example.com');
INSERT INTO users_ci (email) VALUES ('JOHN@EXAMPLE.COM');
-- ERROR: duplicate key value
```

#### **Example 4: Full-Text Search (Case-Insensitive)**

```sql
-- Full-text search is naturally case-insensitive
CREATE TABLE articles (
    id SERIAL PRIMARY KEY,
    title TEXT,
    content TEXT,
    search_vector TSVECTOR
);

-- Update search vector
UPDATE articles
SET search_vector = to_tsvector('english', title || ' ' || content);

CREATE INDEX idx_articles_fts ON articles USING GIN(search_vector);

-- Search (case-insensitive by default)
SELECT title
FROM articles
WHERE search_vector @@ to_tsquery('english', 'PostgreSQL & Database');
-- Finds 'postgresql', 'PostgreSQL', 'POSTGRESQL', etc.
```

---

### **Choosing the Right Method:**

```sql
-- Decision Matrix:

-- Use ILIKE when:
-- ✅ Simple one-off queries
-- ✅ Small tables
-- ✅ Pattern matching needed
SELECT * FROM users WHERE email ILIKE '%@example.com';

-- Use LOWER() with expression index when:
-- ✅ Exact match queries
-- ✅ Frequent searches on existing tables
-- ✅ Can't modify schema
CREATE INDEX idx_email_lower ON users(LOWER(email));
SELECT * FROM users WHERE LOWER(email) = LOWER('john@example.com');

-- Use CITEXT when:
-- ✅ New schema design
-- ✅ All comparisons should be case-insensitive
-- ✅ Want automatic case-insensitivity
CREATE TABLE users (email CITEXT UNIQUE);

-- Use regex (~*) when:
-- ✅ Complex pattern matching
-- ✅ Multiple alternatives
SELECT * FROM users WHERE username ~* '^(john|jane)';

-- Use full-text search when:
-- ✅ Search in large text fields
-- ✅ Need ranking
-- ✅ Linguistic features needed
SELECT * FROM articles WHERE search_vector @@ to_tsquery('database');
```

---

### **Common Pitfalls:**

```sql
-- ❌ ILIKE without index (slow on large tables)
SELECT * FROM large_table
WHERE name ILIKE '%search%';
-- Full table scan

-- ✅ Create trigram index
CREATE EXTENSION pg_trgm;
CREATE INDEX idx_name_trgm ON large_table USING GIN(name gin_trgm_ops);

-- ❌ Inconsistent use of LOWER()
SELECT * FROM users
WHERE LOWER(email) = 'john@example.com';  -- email not lowercased
-- Still works but inconsistent

-- ✅ Consistent lowercasing
SELECT * FROM users
WHERE LOWER(email) = LOWER('john@example.com');

-- ❌ Using LIKE with case-insensitive search
SELECT * FROM users
WHERE LOWER(username) LIKE LOWER('%John%');
-- Works but verbose

-- ✅ Use ILIKE directly
SELECT * FROM users
WHERE username ILIKE '%john%';

-- ❌ Mixing case types
CREATE TABLE users (
    email1 TEXT,
    email2 CITEXT
);
-- Comparison between TEXT and CITEXT may be unexpected
SELECT * FROM users WHERE email1 = email2;
```

---

### **Interview Tips:**
- Start with ILIKE as simplest PostgreSQL-specific solution
- Mention LOWER()/UPPER() with expression indexes for performance
- Discuss CITEXT extension for automatic case-insensitivity
- Show regex ~* operator for complex patterns
- Emphasize indexing strategies (expression index, GIN with pg_trgm)
- Provide practical examples (login, search, unique constraints)
- Mention full-text search for large text fields

</details>

<details>
<summary>17. What are CTEs (Common Table Expressions)?</summary>

### Answer:

**CTEs (Common Table Expressions)** are temporary named result sets defined using the `WITH` clause. They make complex queries more readable, maintainable, and can be referenced multiple times within a query. Think of them as temporary views that exist only for the duration of a query.

---

### **Basic Syntax:**

```sql
WITH cte_name AS (
    SELECT columns
    FROM table
    WHERE conditions
)
SELECT *
FROM cte_name;
```

---

### **Simple CTE Examples:**

```sql
-- Create sample tables
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    department VARCHAR(50),
    salary NUMERIC(10,2),
    manager_id INTEGER,
    hire_date DATE
);

INSERT INTO employees (name, department, salary, manager_id, hire_date) VALUES
    ('Alice', 'Engineering', 120000, NULL, '2018-01-15'),
    ('Bob', 'Engineering', 95000, 1, '2019-03-20'),
    ('Charlie', 'Engineering', 85000, 1, '2020-06-10'),
    ('David', 'Sales', 80000, NULL, '2019-08-05'),
    ('Eve', 'Sales', 75000, 4, '2020-11-12'),
    ('Frank', 'HR', 70000, NULL, '2021-02-28');

-- Basic CTE: High earners
WITH high_earners AS (
    SELECT name, salary, department
    FROM employees
    WHERE salary > 80000
)
SELECT * FROM high_earners
ORDER BY salary DESC;

-- Result: Alice, Bob, Charlie, David
```

---

### **Multiple CTEs:**

```sql
-- Define multiple CTEs in one query
WITH 
dept_avg AS (
    SELECT 
        department,
        AVG(salary) AS avg_salary,
        COUNT(*) AS employee_count
    FROM employees
    GROUP BY department
),
high_earners AS (
    SELECT 
        e.name,
        e.salary,
        e.department
    FROM employees e
    WHERE e.salary > 85000
)
SELECT 
    h.name,
    h.salary,
    h.department,
    d.avg_salary,
    d.employee_count
FROM high_earners h
JOIN dept_avg d ON h.department = d.department
ORDER BY h.salary DESC;

-- CTEs can reference each other
WITH 
sales_data AS (
    SELECT 
        product_id,
        SUM(quantity) AS total_sold,
        SUM(revenue) AS total_revenue
    FROM sales
    GROUP BY product_id
),
top_products AS (
    SELECT *
    FROM sales_data
    WHERE total_sold > 100
)
SELECT * FROM top_products
ORDER BY total_revenue DESC;
```

---

### **Recursive CTEs:**

One of the most powerful CTE features - allows queries to reference themselves.

```sql
-- Recursive CTE structure:
WITH RECURSIVE cte_name AS (
    -- Base case (non-recursive term)
    SELECT initial_values
    
    UNION ALL
    
    -- Recursive term (references cte_name)
    SELECT recursive_values
    FROM cte_name
    WHERE termination_condition
)
SELECT * FROM cte_name;
```

#### **Example 1: Generate Series**

```sql
-- Generate numbers 1 to 10
WITH RECURSIVE numbers AS (
    -- Base case
    SELECT 1 AS n
    
    UNION ALL
    
    -- Recursive case
    SELECT n + 1
    FROM numbers
    WHERE n < 10
)
SELECT * FROM numbers;

-- Result: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10
```

#### **Example 2: Organizational Hierarchy**

```sql
-- Find all employees under a manager (including sub-managers)
WITH RECURSIVE employee_hierarchy AS (
    -- Base case: Start with top-level manager
    SELECT 
        id,
        name,
        manager_id,
        1 AS level,
        name::TEXT AS path
    FROM employees
    WHERE manager_id IS NULL  -- Top-level managers
    
    UNION ALL
    
    -- Recursive case: Find direct reports
    SELECT 
        e.id,
        e.name,
        e.manager_id,
        eh.level + 1,
        eh.path || ' -> ' || e.name
    FROM employees e
    INNER JOIN employee_hierarchy eh ON e.manager_id = eh.id
)
SELECT 
    level,
    REPEAT('  ', level - 1) || name AS employee,
    path AS reporting_chain
FROM employee_hierarchy
ORDER BY path;

-- Result shows organizational tree:
-- level | employee        | reporting_chain
-- ------|-----------------|------------------
-- 1     | Alice           | Alice
-- 2     |   Bob           | Alice -> Bob
-- 2     |   Charlie       | Alice -> Charlie
-- 1     | David           | David
-- 2     |   Eve           | David -> Eve
-- 1     | Frank           | Frank
```

#### **Example 3: Bill of Materials (BOM)**

```sql
-- Create parts table
CREATE TABLE parts (
    part_id INTEGER PRIMARY KEY,
    part_name TEXT,
    parent_part_id INTEGER
);

INSERT INTO parts VALUES
    (1, 'Bicycle', NULL),
    (2, 'Frame', 1),
    (3, 'Wheel', 1),
    (4, 'Tire', 3),
    (5, 'Rim', 3),
    (6, 'Spoke', 5);

-- Find all components of a bicycle
WITH RECURSIVE bom AS (
    -- Base: Top-level product
    SELECT 
        part_id,
        part_name,
        parent_part_id,
        1 AS level
    FROM parts
    WHERE part_id = 1  -- Bicycle
    
    UNION ALL
    
    -- Recursive: Find sub-components
    SELECT 
        p.part_id,
        p.part_name,
        p.parent_part_id,
        b.level + 1
    FROM parts p
    INNER JOIN bom b ON p.parent_part_id = b.part_id
)
SELECT 
    level,
    REPEAT('  ', level - 1) || part_name AS component
FROM bom
ORDER BY level, part_name;

-- Result:
-- level | component
-- ------|-------------
-- 1     | Bicycle
-- 2     |   Frame
-- 2     |   Wheel
-- 3     |     Rim
-- 3     |     Tire
-- 4     |       Spoke
```

---

### **CTE Advantages:**

#### **1. Improved Readability:**

```sql
-- Without CTE (hard to read)
SELECT 
    e.name,
    e.salary,
    (SELECT AVG(salary) FROM employees WHERE department = e.department) AS dept_avg
FROM employees e
WHERE e.salary > (
    SELECT AVG(salary) 
    FROM employees 
    WHERE department = e.department
);

-- With CTE (much clearer)
WITH dept_stats AS (
    SELECT 
        department,
        AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
)
SELECT 
    e.name,
    e.salary,
    d.avg_salary AS dept_avg
FROM employees e
JOIN dept_stats d ON e.department = d.department
WHERE e.salary > d.avg_salary;
```

#### **2. Reusability:**

```sql
-- Reference CTE multiple times
WITH sales_summary AS (
    SELECT 
        product_id,
        SUM(quantity) AS total_quantity,
        SUM(revenue) AS total_revenue
    FROM sales
    GROUP BY product_id
)
SELECT 
    'Top by Quantity' AS metric,
    product_id,
    total_quantity AS value
FROM sales_summary
ORDER BY total_quantity DESC
LIMIT 5

UNION ALL

SELECT 
    'Top by Revenue' AS metric,
    product_id,
    total_revenue AS value
FROM sales_summary
ORDER BY total_revenue DESC
LIMIT 5;
```

#### **3. Breaking Down Complex Queries:**

```sql
-- Multi-step analysis made readable
WITH 
-- Step 1: Get active customers
active_customers AS (
    SELECT DISTINCT customer_id
    FROM orders
    WHERE order_date >= CURRENT_DATE - INTERVAL '90 days'
),
-- Step 2: Calculate customer metrics
customer_metrics AS (
    SELECT 
        c.customer_id,
        c.name,
        COUNT(o.id) AS order_count,
        SUM(o.total) AS total_spent,
        AVG(o.total) AS avg_order_value
    FROM active_customers ac
    JOIN customers c ON ac.customer_id = c.id
    LEFT JOIN orders o ON c.id = o.customer_id
    GROUP BY c.customer_id, c.name
),
-- Step 3: Segment customers
customer_segments AS (
    SELECT 
        *,
        CASE 
            WHEN total_spent >= 10000 THEN 'VIP'
            WHEN total_spent >= 5000 THEN 'Premium'
            WHEN total_spent >= 1000 THEN 'Regular'
            ELSE 'New'
        END AS segment
    FROM customer_metrics
)
SELECT 
    segment,
    COUNT(*) AS customer_count,
    AVG(total_spent) AS avg_spent,
    SUM(total_spent) AS segment_revenue
FROM customer_segments
GROUP BY segment
ORDER BY segment_revenue DESC;
```

---

### **CTE Materialization (PostgreSQL 12+):**

```sql
-- By default, CTEs are optimization fence (always materialized)
-- Can control with MATERIALIZED / NOT MATERIALIZED hints

-- Force materialization (default before PG12)
WITH expensive_calc AS MATERIALIZED (
    SELECT 
        product_id,
        complex_calculation(data) AS result
    FROM large_table
)
SELECT * FROM expensive_calc
WHERE result > 100;

-- Prevent materialization (inline CTE)
WITH simple_filter AS NOT MATERIALIZED (
    SELECT * FROM products
    WHERE active = true
)
SELECT * FROM simple_filter
WHERE price > 100;
-- Optimizer can push down the price filter

-- When to use MATERIALIZED:
-- ✅ CTE is used multiple times
-- ✅ CTE computation is expensive
-- ✅ Want to calculate once, reuse result

-- When to use NOT MATERIALIZED:
-- ✅ Simple filters
-- ✅ CTE used only once
-- ✅ Want optimizer to inline and optimize together
```

---

### **Practical Production Examples:**

#### **Example 1: Date Series Generation**

```sql
-- Generate date range for reports (fill gaps)
WITH RECURSIVE date_series AS (
    SELECT DATE '2024-01-01' AS date
    
    UNION ALL
    
    SELECT date + INTERVAL '1 day'
    FROM date_series
    WHERE date < '2024-12-31'
)
SELECT 
    d.date,
    COALESCE(COUNT(o.id), 0) AS order_count,
    COALESCE(SUM(o.total), 0) AS daily_revenue
FROM date_series d
LEFT JOIN orders o ON DATE(o.order_date) = d.date
GROUP BY d.date
ORDER BY d.date;
-- Shows all days, even days with no orders (0 count)
```

#### **Example 2: Running Totals**

```sql
-- Calculate running totals with CTE
WITH daily_sales AS (
    SELECT 
        DATE(order_date) AS sale_date,
        SUM(total) AS daily_total
    FROM orders
    GROUP BY DATE(order_date)
)
SELECT 
    sale_date,
    daily_total,
    SUM(daily_total) OVER (ORDER BY sale_date) AS running_total
FROM daily_sales
ORDER BY sale_date;
```

#### **Example 3: Permission Checking (Recursive)**

```sql
-- Check if user has permission (through roles/groups)
CREATE TABLE user_roles (
    user_id INTEGER,
    role_id INTEGER
);

CREATE TABLE role_hierarchy (
    role_id INTEGER,
    parent_role_id INTEGER,
    role_name TEXT
);

-- Check all roles user has (including inherited)
WITH RECURSIVE user_permissions AS (
    -- Base: Direct roles
    SELECT 
        ur.user_id,
        rh.role_id,
        rh.role_name,
        1 AS depth
    FROM user_roles ur
    JOIN role_hierarchy rh ON ur.role_id = rh.role_id
    WHERE ur.user_id = 123
    
    UNION
    
    -- Recursive: Parent roles
    SELECT 
        up.user_id,
        rh.role_id,
        rh.role_name,
        up.depth + 1
    FROM user_permissions up
    JOIN role_hierarchy rh ON up.role_id = rh.parent_role_id
)
SELECT DISTINCT role_name
FROM user_permissions
ORDER BY role_name;
```

#### **Example 4: Data Cleaning Pipeline**

```sql
-- Multi-stage data cleaning
WITH 
-- Stage 1: Remove nulls
cleaned_data AS (
    SELECT *
    FROM raw_data
    WHERE email IS NOT NULL 
      AND name IS NOT NULL
),
-- Stage 2: Standardize format
standardized_data AS (
    SELECT 
        id,
        LOWER(TRIM(email)) AS email,
        INITCAP(name) AS name,
        phone
    FROM cleaned_data
),
-- Stage 3: Remove duplicates (keep first)
deduped_data AS (
    SELECT DISTINCT ON (email)
        *
    FROM standardized_data
    ORDER BY email, id
)
SELECT * FROM deduped_data;
```

---

### **Common Mistakes:**

```sql
-- ❌ Forgetting RECURSIVE keyword for recursive CTEs
WITH employee_tree AS (
    SELECT * FROM employees WHERE manager_id IS NULL
    UNION ALL
    SELECT e.* FROM employees e
    JOIN employee_tree et ON e.manager_id = et.id
);
-- ERROR: Must use WITH RECURSIVE

-- ✅ Correct
WITH RECURSIVE employee_tree AS (
    SELECT * FROM employees WHERE manager_id IS NULL
    UNION ALL
    SELECT e.* FROM employees e
    JOIN employee_tree et ON e.manager_id = et.id
)
SELECT * FROM employee_tree;

-- ❌ Infinite recursion (no termination)
WITH RECURSIVE infinite AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1 FROM infinite  -- No WHERE clause!
)
SELECT * FROM infinite;
-- Runs until max recursion depth

-- ✅ Always have termination condition
WITH RECURSIVE numbers AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1 FROM numbers WHERE n < 100
)
SELECT * FROM numbers;

-- ❌ Using CTEs when not needed (unnecessary materialization)
WITH simple_cte AS (
    SELECT * FROM table WHERE active = true
)
SELECT * FROM simple_cte WHERE price > 100;

-- ✅ Just use WHERE directly (simpler, optimizer-friendly)
SELECT * FROM table 
WHERE active = true AND price > 100;
```

---

### **CTE vs Temporary Tables:**

```sql
-- CTE: Query-scoped, no cleanup needed
WITH temp_data AS (
    SELECT * FROM large_table WHERE condition
)
SELECT * FROM temp_data;
-- Automatically gone after query

-- Temporary Table: Session-scoped, explicit cleanup
CREATE TEMP TABLE temp_data AS
SELECT * FROM large_table WHERE condition;

SELECT * FROM temp_data;  -- Can use in multiple queries
SELECT * FROM temp_data WHERE another_condition;

DROP TABLE temp_data;  -- Manual cleanup

-- Use CTE when:
-- ✅ Single query
-- ✅ Readable, organized code
-- ✅ No indexes needed

-- Use Temp Table when:
-- ✅ Multiple queries need same data
-- ✅ Need indexes on intermediate result
-- ✅ Very large intermediate result
```

---

### **Interview Tips:**
- Define CTE as temporary named result set using WITH clause
- Explain readability benefits (break down complex queries)
- Show recursive CTEs for hierarchical data (trees, graphs)
- Mention CTE can be referenced multiple times in query
- Discuss MATERIALIZED vs NOT MATERIALIZED (PG12+)
- Provide practical examples (org charts, date series, BOM)
- Compare to subqueries and temp tables

</details>

<details>
<summary>18. What is the difference between a subquery and a CTE?</summary>

### Answer:

**Subqueries** and **CTEs (Common Table Expressions)** both allow you to nest queries, but they differ in syntax, readability, performance characteristics, and capabilities. Understanding when to use each is important for writing efficient, maintainable SQL.

---

### **Key Differences:**

| Aspect | Subquery | CTE |
|--------|----------|-----|
| **Syntax** | Inline in FROM, WHERE, SELECT | WITH clause at top |
| **Readability** | Can be hard to read (nested) | More readable (named, sequential) |
| **Reusability** | Must repeat if used multiple times | Can reference multiple times |
| **Recursion** | Not supported | Supports recursive queries |
| **Naming** | Anonymous (no name) | Named (explicit alias) |
| **Execution** | Depends on optimizer | Materialized by default (PG<12) |
| **Debugging** | Harder to debug | Easier to debug (test each CTE) |
| **Scope** | Limited to that part of query | Available to entire query |

---

### **Syntax Comparison:**

#### **Subquery:**

```sql
-- Inline subquery
SELECT *
FROM (
    SELECT 
        name,
        salary,
        department
    FROM employees
    WHERE salary > 80000
) AS high_earners
WHERE department = 'Engineering';

-- Subquery in WHERE
SELECT name, salary
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);

-- Subquery in SELECT
SELECT 
    name,
    salary,
    (SELECT AVG(salary) FROM employees WHERE department = e.department) AS dept_avg
FROM employees e;
```

#### **CTE:**

```sql
-- Equivalent CTE
WITH high_earners AS (
    SELECT 
        name,
        salary,
        department
    FROM employees
    WHERE salary > 80000
)
SELECT *
FROM high_earners
WHERE department = 'Engineering';

-- Multiple CTEs
WITH 
avg_salary AS (
    SELECT AVG(salary) AS avg_sal
    FROM employees
),
dept_avg AS (
    SELECT 
        department,
        AVG(salary) AS dept_avg_sal
    FROM employees
    GROUP BY department
)
SELECT 
    e.name,
    e.salary,
    a.avg_sal,
    d.dept_avg_sal
FROM employees e
CROSS JOIN avg_salary a
JOIN dept_avg d ON e.department = d.department;
```

---

### **Readability Comparison:**

#### **Complex Nested Subqueries (Hard to Read):**

```sql
-- Deeply nested subqueries
SELECT 
    dept,
    avg_salary,
    (SELECT AVG(revenue) 
     FROM sales s
     WHERE s.department = d.dept) AS avg_revenue
FROM (
    SELECT 
        department AS dept,
        AVG(salary) AS avg_salary
    FROM employees
    WHERE salary > (
        SELECT AVG(salary)
        FROM employees
        WHERE hire_date > '2020-01-01'
    )
    GROUP BY department
) d
WHERE avg_salary > (
    SELECT AVG(salary) * 0.8
    FROM employees
);
```

#### **Same Logic with CTEs (Much Clearer):**

```sql
-- Step-by-step with CTEs
WITH 
recent_avg AS (
    SELECT AVG(salary) AS avg_sal
    FROM employees
    WHERE hire_date > '2020-01-01'
),
high_salary_emps AS (
    SELECT department, AVG(salary) AS avg_salary
    FROM employees
    WHERE salary > (SELECT avg_sal FROM recent_avg)
    GROUP BY department
),
company_avg AS (
    SELECT AVG(salary) * 0.8 AS threshold
    FROM employees
),
dept_revenue AS (
    SELECT 
        department,
        AVG(revenue) AS avg_revenue
    FROM sales
    GROUP BY department
)
SELECT 
    h.department AS dept,
    h.avg_salary,
    d.avg_revenue
FROM high_salary_emps h
JOIN dept_revenue d ON h.department = d.department
WHERE h.avg_salary > (SELECT threshold FROM company_avg);
```

---

### **Reusability:**

#### **Subquery - Must Repeat:**

```sql
-- Same subquery repeated multiple times
SELECT 
    'Top Performers' AS category,
    name,
    salary
FROM (
    SELECT name, salary
    FROM employees
    WHERE salary > (SELECT AVG(salary) FROM employees)
) high_earners
WHERE salary > 100000

UNION ALL

SELECT 
    'High Earners Count' AS category,
    COUNT(*)::TEXT AS name,
    NULL::NUMERIC AS salary
FROM (
    SELECT name, salary
    FROM employees
    WHERE salary > (SELECT AVG(salary) FROM employees)
) high_earners;
-- Subquery repeated - not DRY (Don't Repeat Yourself)
```

#### **CTE - Define Once, Use Multiple Times:**

```sql
-- CTE defined once, referenced multiple times
WITH high_earners AS (
    SELECT name, salary
    FROM employees
    WHERE salary > (SELECT AVG(salary) FROM employees)
)
SELECT 
    'Top Performers' AS category,
    name,
    salary
FROM high_earners
WHERE salary > 100000

UNION ALL

SELECT 
    'High Earners Count' AS category,
    COUNT(*)::TEXT AS name,
    NULL::NUMERIC AS salary
FROM high_earners;
-- high_earners CTE used twice
```

---

### **Recursion:**

#### **Subqueries Cannot Be Recursive:**

```sql
-- ❌ Subqueries cannot reference themselves
-- This is not possible with subqueries alone

-- Hierarchical data requires recursive approach
```

#### **CTEs Support Recursion:**

```sql
-- ✅ Recursive CTE for hierarchical data
WITH RECURSIVE employee_chain AS (
    -- Base case
    SELECT id, name, manager_id, 1 AS level
    FROM employees
    WHERE id = 5  -- Start with employee ID 5
    
    UNION ALL
    
    -- Recursive case
    SELECT 
        e.id,
        e.name,
        e.manager_id,
        ec.level + 1
    FROM employees e
    JOIN employee_chain ec ON e.manager_id = ec.id
)
SELECT * FROM employee_chain;
-- Shows employee 5 and all their managers up the chain
```

---

### **Performance Considerations:**

#### **Subquery Performance:**

```sql
-- Correlated subquery (can be slow)
SELECT 
    e.name,
    e.salary,
    (
        SELECT AVG(salary)
        FROM employees e2
        WHERE e2.department = e.department
    ) AS dept_avg
FROM employees e;
-- Subquery executes for EACH row (N times)
-- Can be very slow on large tables

-- Optimizer may convert to join, but not guaranteed
```

#### **CTE Performance:**

```sql
-- CTE approach (typically better)
WITH dept_avg AS (
    SELECT 
        department,
        AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
)
SELECT 
    e.name,
    e.salary,
    d.avg_salary AS dept_avg
FROM employees e
JOIN dept_avg d ON e.department = d.department;
-- Calculated once, joined to main query

-- Note: In PostgreSQL < 12, CTEs are optimization fence
-- In PostgreSQL 12+, can use NOT MATERIALIZED for optimization
WITH dept_avg AS NOT MATERIALIZED (
    SELECT department, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
)
SELECT e.name, e.salary, d.avg_salary
FROM employees e
JOIN dept_avg d ON e.department = d.department;
```

---

### **When to Use Subqueries:**

```sql
-- ✅ Simple one-time filters
SELECT *
FROM products
WHERE price > (SELECT AVG(price) FROM products);

-- ✅ EXISTS / NOT EXISTS checks
SELECT *
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
      AND o.order_date >= CURRENT_DATE - INTERVAL '30 days'
);

-- ✅ IN / NOT IN with small lists
SELECT *
FROM employees
WHERE department_id IN (
    SELECT id FROM departments WHERE location = 'New York'
);

-- ✅ Single value in SELECT
SELECT 
    product_name,
    price,
    (SELECT MAX(price) FROM products) AS max_price
FROM products;
```

---

### **When to Use CTEs:**

```sql
-- ✅ Complex multi-step queries
WITH 
step1 AS (SELECT ...),
step2 AS (SELECT ... FROM step1),
step3 AS (SELECT ... FROM step2)
SELECT * FROM step3;

-- ✅ Recursive queries
WITH RECURSIVE hierarchy AS (
    SELECT ... -- Base
    UNION ALL
    SELECT ... FROM hierarchy -- Recursive
)
SELECT * FROM hierarchy;

-- ✅ Reusing same subquery multiple times
WITH data_subset AS (
    SELECT ... FROM large_table WHERE complex_conditions
)
SELECT ... FROM data_subset WHERE condition1
UNION ALL
SELECT ... FROM data_subset WHERE condition2;

-- ✅ Improving readability
WITH 
active_users AS (...),
user_stats AS (...),
final_report AS (...)
SELECT * FROM final_report;
```

---

### **Practical Examples:**

#### **Example 1: Data Analysis Pipeline**

```sql
-- Subquery version (harder to follow)
SELECT 
    category,
    total_sales,
    pct_of_total
FROM (
    SELECT 
        category,
        total_sales,
        total_sales / (SELECT SUM(total_sales) FROM (
            SELECT 
                category,
                SUM(revenue) AS total_sales
            FROM sales
            GROUP BY category
        ) s) * 100 AS pct_of_total
    FROM (
        SELECT 
            category,
            SUM(revenue) AS total_sales
        FROM sales
        GROUP BY category
    ) cat_sales
) final
WHERE pct_of_total > 10;

-- CTE version (clear steps)
WITH 
category_sales AS (
    SELECT 
        category,
        SUM(revenue) AS total_sales
    FROM sales
    GROUP BY category
),
total_sales_sum AS (
    SELECT SUM(total_sales) AS company_total
    FROM category_sales
),
category_pct AS (
    SELECT 
        cs.category,
        cs.total_sales,
        (cs.total_sales / ts.company_total * 100) AS pct_of_total
    FROM category_sales cs
    CROSS JOIN total_sales_sum ts
)
SELECT *
FROM category_pct
WHERE pct_of_total > 10;
```

#### **Example 2: Finding Top Performers per Department**

```sql
-- Subquery version
SELECT *
FROM (
    SELECT 
        name,
        department,
        salary,
        ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rank
    FROM employees
) ranked
WHERE rank <= 3;

-- CTE version (same readability in this case)
WITH ranked_employees AS (
    SELECT 
        name,
        department,
        salary,
        ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rank
    FROM employees
)
SELECT *
FROM ranked_employees
WHERE rank <= 3;
-- For simple window function queries, both are fine
```

---

### **Conversion Examples:**

#### **Correlated Subquery → CTE:**

```sql
-- Before: Correlated subquery
SELECT 
    p.product_name,
    p.price,
    (
        SELECT AVG(price)
        FROM products p2
        WHERE p2.category = p.category
    ) AS category_avg_price
FROM products p;

-- After: CTE (more efficient)
WITH category_averages AS (
    SELECT 
        category,
        AVG(price) AS avg_price
    FROM products
    GROUP BY category
)
SELECT 
    p.product_name,
    p.price,
    ca.avg_price AS category_avg_price
FROM products p
JOIN category_averages ca ON p.category = ca.category;
```

---

### **Mixed Approach (Both Together):**

```sql
-- You can use both in same query
WITH recent_orders AS (
    SELECT *
    FROM orders
    WHERE order_date >= CURRENT_DATE - INTERVAL '30 days'
)
SELECT 
    customer_id,
    COUNT(*) AS order_count,
    SUM(total) AS total_spent
FROM recent_orders
WHERE total > (
    SELECT AVG(total)  -- Subquery within CTE query
    FROM recent_orders
)
GROUP BY customer_id;
```

---

### **Debugging:**

```sql
-- CTEs are easier to debug (test each step)
WITH 
step1 AS (
    SELECT * FROM orders WHERE status = 'completed'
),
step2 AS (
    SELECT customer_id, SUM(total) AS total_spent
    FROM step1
    GROUP BY customer_id
),
step3 AS (
    SELECT *
    FROM step2
    WHERE total_spent > 1000
)
SELECT * FROM step1;  -- Test step 1
-- SELECT * FROM step2;  -- Test step 2
-- SELECT * FROM step3;  -- Test final step

-- Subqueries are harder to isolate and test
```

---

### **Interview Tips:**
- Emphasize readability: CTEs are more readable for complex queries
- Mention reusability: CTEs can be referenced multiple times
- Highlight recursion: Only CTEs support recursive queries
- Discuss performance: correlated subqueries can be slow
- Show when to use each: simple filters (subquery), complex multi-step (CTE)
- Mention materialization: CTE behavior changed in PostgreSQL 12
- Provide conversion example: correlated subquery → CTE with JOIN

</details>

<details>
<summary>19. How do you use window functions in PostgreSQL?</summary>

### Answer:

**Window functions** perform calculations across a set of rows related to the current row, without collapsing rows like aggregate functions do. They're essential for analytics, rankings, running totals, and comparing rows within partitions.

---

### **Basic Syntax:**

```sql
function_name([arguments]) OVER (
    [PARTITION BY partition_expression]
    [ORDER BY sort_expression]
    [frame_clause]
)
```

---

### **Window Function Categories:**

1. **Ranking**: ROW_NUMBER(), RANK(), DENSE_RANK(), NTILE()
2. **Aggregate**: SUM(), AVG(), COUNT(), MIN(), MAX()
3. **Value**: LAG(), LEAD(), FIRST_VALUE(), LAST_VALUE(), NTH_VALUE()
4. **Statistical**: PERCENT_RANK(), CUME_DIST()

---

### **Setup Sample Data:**

```sql
CREATE TABLE sales (
    id SERIAL PRIMARY KEY,
    employee_name VARCHAR(100),
    department VARCHAR(50),
    sale_date DATE,
    amount NUMERIC(10,2)
);

INSERT INTO sales (employee_name, department, sale_date, amount) VALUES
    ('Alice', 'Electronics', '2024-01-15', 1200),
    ('Bob', 'Electronics', '2024-01-16', 800),
    ('Charlie', 'Electronics', '2024-01-17', 1500),
    ('David', 'Furniture', '2024-01-15', 600),
    ('Eve', 'Furniture', '2024-01-16', 900),
    ('Frank', 'Furniture', '2024-01-17', 750),
    ('Alice', 'Electronics', '2024-01-18', 1300),
    ('Bob', 'Electronics', '2024-01-19', 950);
```

---

### **1. ROW_NUMBER() - Sequential Number:**

```sql
-- Assign unique row number to each row
SELECT 
    employee_name,
    department,
    amount,
    ROW_NUMBER() OVER (ORDER BY amount DESC) AS overall_rank
FROM sales;

-- Result:
-- employee_name | department   | amount | overall_rank
-- --------------|--------------|--------|-------------
-- Charlie       | Electronics  | 1500   | 1
-- Alice         | Electronics  | 1300   | 2
-- Alice         | Electronics  | 1200   | 3
-- Bob           | Electronics  | 950    | 4
-- ...

-- Row number within each department
SELECT 
    employee_name,
    department,
    amount,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY amount DESC) AS dept_rank
FROM sales;

-- Result shows ranking within each department separately
```

---

### **2. Aggregate Window Functions:**

```sql
-- Running total
SELECT 
    employee_name,
    sale_date,
    amount,
    SUM(amount) OVER (ORDER BY sale_date) AS running_total
FROM sales
ORDER BY sale_date;

-- Result:
-- employee_name | sale_date   | amount | running_total
-- --------------|-------------|--------|---------------
-- Alice         | 2024-01-15  | 1200   | 1800  (1200+600)
-- David         | 2024-01-15  | 600    | 1800
-- Bob           | 2024-01-16  | 800    | 3500  (+800+900)
-- Eve           | 2024-01-16  | 900    | 3500
-- ...

-- Department totals (without collapsing rows)
SELECT 
    employee_name,
    department,
    amount,
    SUM(amount) OVER (PARTITION BY department) AS dept_total,
    AVG(amount) OVER (PARTITION BY department) AS dept_avg,
    COUNT(*) OVER (PARTITION BY department) AS dept_count
FROM sales;

-- Result: Each row shows individual amount + department aggregates
```

---

### **3. LAG() and LEAD() - Previous/Next Row Values:**

```sql
-- Compare with previous sale
SELECT 
    employee_name,
    sale_date,
    amount,
    LAG(amount) OVER (ORDER BY sale_date) AS previous_sale,
    amount - LAG(amount) OVER (ORDER BY sale_date) AS change
FROM sales
ORDER BY sale_date;

-- Result:
-- employee_name | sale_date  | amount | previous_sale | change
-- --------------|------------|--------|---------------|--------
-- Alice         | 2024-01-15 | 1200   | NULL          | NULL
-- David         | 2024-01-15 | 600    | 1200          | -600
-- Bob           | 2024-01-16 | 800    | 600           | 200
-- ...

-- Compare employee's own sales over time
SELECT 
    employee_name,
    sale_date,
    amount,
    LAG(amount) OVER (PARTITION BY employee_name ORDER BY sale_date) AS previous_amount,
    LEAD(amount) OVER (PARTITION BY employee_name ORDER BY sale_date) AS next_amount
FROM sales
ORDER BY employee_name, sale_date;

-- With default value for NULL
SELECT 
    employee_name,
    sale_date,
    amount,
    LAG(amount, 1, 0) OVER (PARTITION BY employee_name ORDER BY sale_date) AS previous_amount
FROM sales;
-- LAG(column, offset, default)
```

---

### **4. FIRST_VALUE() and LAST_VALUE():**

```sql
-- Compare to first sale of the day
SELECT 
    employee_name,
    sale_date,
    amount,
    FIRST_VALUE(amount) OVER (PARTITION BY sale_date ORDER BY amount) AS lowest_sale_today,
    LAST_VALUE(amount) OVER (
        PARTITION BY sale_date 
        ORDER BY amount
        RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS highest_sale_today
FROM sales
ORDER BY sale_date, amount;

-- Note: LAST_VALUE requires frame clause to work as expected
-- Default frame is RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

---

### **5. Frame Clauses (ROWS vs RANGE):**

```sql
-- Moving average (last 3 sales)
SELECT 
    employee_name,
    sale_date,
    amount,
    AVG(amount) OVER (
        ORDER BY sale_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg_3
FROM sales
ORDER BY sale_date;

-- ROWS: Physical row offsets
-- RANGE: Logical value-based ranges

-- Example showing difference
CREATE TABLE values (id INT, val INT);
INSERT INTO values VALUES (1, 10), (2, 10), (3, 20), (4, 30);

-- ROWS: Counts physical rows
SELECT 
    val,
    SUM(val) OVER (ORDER BY val ROWS BETWEEN 1 PRECEDING AND CURRENT ROW) AS rows_sum
FROM values;
-- Result: First row uses 1 row, second uses 2 rows

-- RANGE: Includes all rows with same value
SELECT 
    val,
    SUM(val) OVER (ORDER BY val RANGE BETWEEN 1 PRECEDING AND CURRENT ROW) AS range_sum
FROM values;
-- Result: Both rows with val=10 included together
```

---

### **Practical Examples:**

#### **Example 1: Sales Leaderboard**

```sql
-- Top 3 salespeople per department
WITH ranked_sales AS (
    SELECT 
        employee_name,
        department,
        SUM(amount) AS total_sales,
        RANK() OVER (PARTITION BY department ORDER BY SUM(amount) DESC) AS rank
    FROM sales
    GROUP BY employee_name, department
)
SELECT *
FROM ranked_sales
WHERE rank <= 3
ORDER BY department, rank;
```

#### **Example 2: Running Total by Department**

```sql
-- Running total within each department
SELECT 
    employee_name,
    department,
    sale_date,
    amount,
    SUM(amount) OVER (
        PARTITION BY department 
        ORDER BY sale_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS dept_running_total
FROM sales
ORDER BY department, sale_date;
```

#### **Example 3: Percentage of Total**

```sql
-- Each sale as percentage of department total
SELECT 
    employee_name,
    department,
    amount,
    SUM(amount) OVER (PARTITION BY department) AS dept_total,
    ROUND(
        amount * 100.0 / SUM(amount) OVER (PARTITION BY department),
        2
    ) AS pct_of_dept_total
FROM sales
ORDER BY department, amount DESC;
```

#### **Example 4: Year-over-Year Comparison**

```sql
-- Compare sales to same period last year
WITH monthly_sales AS (
    SELECT 
        DATE_TRUNC('month', sale_date) AS month,
        department,
        SUM(amount) AS monthly_total
    FROM sales
    GROUP BY DATE_TRUNC('month', sale_date), department
)
SELECT 
    month,
    department,
    monthly_total,
    LAG(monthly_total, 12) OVER (
        PARTITION BY department 
        ORDER BY month
    ) AS same_month_last_year,
    monthly_total - LAG(monthly_total, 12) OVER (
        PARTITION BY department 
        ORDER BY month
    ) AS yoy_change
FROM monthly_sales
ORDER BY department, month;
```

#### **Example 5: Gap Analysis**

```sql
-- Find gaps between consecutive sales
SELECT 
    employee_name,
    sale_date,
    LAG(sale_date) OVER (PARTITION BY employee_name ORDER BY sale_date) AS previous_sale_date,
    sale_date - LAG(sale_date) OVER (
        PARTITION BY employee_name 
        ORDER BY sale_date
    ) AS days_since_last_sale
FROM sales
WHERE employee_name = 'Alice'
ORDER BY sale_date;
```

#### **Example 6: Percentiles**

```sql
-- Assign sales to quartiles
SELECT 
    employee_name,
    amount,
    NTILE(4) OVER (ORDER BY amount) AS quartile,
    PERCENT_RANK() OVER (ORDER BY amount) AS percent_rank,
    CUME_DIST() OVER (ORDER BY amount) AS cumulative_dist
FROM sales
ORDER BY amount;

-- quartile: 1=bottom 25%, 2=25-50%, 3=50-75%, 4=top 25%
-- percent_rank: Relative rank (0 to 1)
-- cume_dist: Cumulative distribution (0 to 1)
```

---

### **Named Windows (Reusable):**

```sql
-- Define window once, reuse multiple times
SELECT 
    employee_name,
    department,
    amount,
    ROW_NUMBER() OVER w AS row_num,
    RANK() OVER w AS rank,
    AVG(amount) OVER w AS avg_amount
FROM sales
WINDOW w AS (PARTITION BY department ORDER BY amount DESC)
ORDER BY department, amount DESC;
-- More DRY (Don't Repeat Yourself)
```

---

### **Performance Optimization:**

```sql
-- Window functions can be expensive on large datasets
-- Create indexes to help sorting

-- Index for PARTITION BY + ORDER BY
CREATE INDEX idx_sales_dept_date ON sales(department, sale_date);

-- Index for just ORDER BY
CREATE INDEX idx_sales_amount ON sales(amount DESC);

-- Use EXPLAIN to check query plan
EXPLAIN ANALYZE
SELECT 
    employee_name,
    amount,
    AVG(amount) OVER (PARTITION BY department ORDER BY sale_date)
FROM sales;
```

---

### **Common Pitfalls:**

```sql
-- ❌ Forgetting ORDER BY (results undefined)
SELECT 
    employee_name,
    ROW_NUMBER() OVER (PARTITION BY department) AS row_num
FROM sales;
-- row_num order is random!

-- ✅ Always specify ORDER BY for ranking functions
SELECT 
    employee_name,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY amount DESC) AS row_num
FROM sales;

-- ❌ LAST_VALUE without proper frame
SELECT 
    amount,
    LAST_VALUE(amount) OVER (ORDER BY sale_date) AS last_val
FROM sales;
-- Returns current row, not last row!

-- ✅ Specify frame explicitly
SELECT 
    amount,
    LAST_VALUE(amount) OVER (
        ORDER BY sale_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS last_val
FROM sales;
```

---

### **Interview Tips:**
- Explain window functions don't collapse rows (unlike GROUP BY)
- Show PARTITION BY (groups) and ORDER BY (sorting within groups)
- Demonstrate ROW_NUMBER, RANK, DENSE_RANK differences
- Provide LAG/LEAD for row comparisons
- Mention frame clauses (ROWS vs RANGE)
- Give practical examples (running totals, rankings, percentages)
- Discuss performance (indexes on PARTITION BY and ORDER BY columns)

</details>

<details>
<summary>20. What is the difference between RANK(), DENSE_RANK(), and ROW_NUMBER()?</summary>

### Answer:

**ROW_NUMBER()**, **RANK()**, and **DENSE_RANK()** are window functions used for ranking rows, but they handle ties (duplicate values) differently. Understanding these differences is crucial for leaderboards, pagination, and ranking scenarios.

---

### **Quick Comparison:**

| Function | Behavior | Gaps After Ties | Use Case |
|----------|----------|-----------------|----------|
| **ROW_NUMBER()** | Always unique | N/A (no ties) | Pagination, unique identifiers |
| **RANK()** | Same rank for ties | Yes (skips numbers) | Competition ranking (1, 2, 2, 4) |
| **DENSE_RANK()** | Same rank for ties | No (consecutive) | Dense ranking (1, 2, 2, 3) |

---

### **Visual Example:**

```sql
-- Sample data with ties
CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    score INTEGER
);

INSERT INTO students (name, score) VALUES
    ('Alice', 95),
    ('Bob', 90),
    ('Charlie', 90),    -- Tie with Bob
    ('David', 85),
    ('Eve', 85),        -- Tie with David
    ('Frank', 80);

-- Compare all three ranking functions
SELECT 
    name,
    score,
    ROW_NUMBER() OVER (ORDER BY score DESC) AS row_num,
    RANK() OVER (ORDER BY score DESC) AS rank,
    DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rank
FROM students
ORDER BY score DESC;

-- Result:
-- name    | score | row_num | rank | dense_rank
-- --------|-------|---------|------|------------
-- Alice   | 95    | 1       | 1    | 1
-- Bob     | 90    | 2       | 2    | 2
-- Charlie | 90    | 3       | 2    | 2  ← Same rank as Bob
-- David   | 85    | 4       | 4    | 3  ← RANK skips 3, DENSE_RANK doesn't
-- Eve     | 85    | 5       | 4    | 3
-- Frank   | 80    | 6       | 6    | 4  ← RANK skips 5, DENSE_RANK doesn't
```

---

### **Detailed Breakdown:**

#### **1. ROW_NUMBER() - Always Unique:**

```sql
-- ROW_NUMBER: Sequential numbering (1, 2, 3, 4, ...)
-- Even with ties, assigns unique numbers
SELECT 
    name,
    score,
    ROW_NUMBER() OVER (ORDER BY score DESC) AS row_num
FROM students
ORDER BY score DESC;

-- Result:
-- name    | score | row_num
-- --------|-------|--------
-- Alice   | 95    | 1
-- Bob     | 90    | 2
-- Charlie | 90    | 3       ← Different from Bob (arbitrary order)
-- David   | 85    | 4
-- Eve     | 85    | 5       ← Different from David (arbitrary order)
-- Frank   | 80    | 6

-- With additional ORDER BY to make deterministic
SELECT 
    name,
    score,
    ROW_NUMBER() OVER (ORDER BY score DESC, name) AS row_num
FROM students
ORDER BY score DESC, name;
-- Now ties are broken by name alphabetically
```

**Use Cases for ROW_NUMBER():**
```sql
-- ✅ Pagination (need unique row numbers)
SELECT * FROM (
    SELECT 
        *,
        ROW_NUMBER() OVER (ORDER BY created_at DESC) AS rn
    FROM products
) t
WHERE rn BETWEEN 11 AND 20;  -- Page 2 (items 11-20)

-- ✅ Deduplication (keep first occurrence)
DELETE FROM users
WHERE id IN (
    SELECT id FROM (
        SELECT 
            id,
            ROW_NUMBER() OVER (PARTITION BY email ORDER BY created_at) AS rn
        FROM users
    ) t
    WHERE rn > 1
);

-- ✅ Unique identifiers for each row
SELECT 
    'ORD-' || ROW_NUMBER() OVER (ORDER BY order_date) AS order_number,
    customer_name,
    total
FROM orders;
```

---

#### **2. RANK() - Competition Ranking (Gaps):**

```sql
-- RANK: Same rank for ties, skips next numbers
-- Like Olympic medals: If two silver medals, no bronze
SELECT 
    name,
    score,
    RANK() OVER (ORDER BY score DESC) AS rank
FROM students
ORDER BY score DESC;

-- Result:
-- name    | score | rank
-- --------|-------|-----
-- Alice   | 95    | 1
-- Bob     | 90    | 2
-- Charlie | 90    | 2     ← Same rank as Bob
-- David   | 85    | 4     ← Skips 3 (because two people at rank 2)
-- Eve     | 85    | 4
-- Frank   | 80    | 6     ← Skips 5
```

**Use Cases for RANK():**
```sql
-- ✅ Sports/competition rankings
WITH game_scores AS (
    SELECT 
        player_name,
        score,
        RANK() OVER (ORDER BY score DESC) AS position
    FROM tournament_results
)
SELECT 
    CASE position
        WHEN 1 THEN '🥇 Gold'
        WHEN 2 THEN '🥈 Silver'
        WHEN 3 THEN '🥉 Bronze'
        ELSE position || 'th Place'
    END AS medal,
    player_name,
    score
FROM game_scores
ORDER BY position;

-- ✅ Top N with ties
-- "Get top 3 scores" - if tie at 3rd, includes all tied
WITH ranked_students AS (
    SELECT 
        name,
        score,
        RANK() OVER (ORDER BY score DESC) AS rank
    FROM students
)
SELECT *
FROM ranked_students
WHERE rank <= 3;
-- If two students tied for 3rd place, both included

-- ✅ Percentile calculations
SELECT 
    employee_name,
    salary,
    RANK() OVER (ORDER BY salary DESC) AS salary_rank,
    COUNT(*) OVER () AS total_employees,
    ROUND(100.0 * RANK() OVER (ORDER BY salary DESC) / COUNT(*) OVER (), 2) AS percentile
FROM employees;
```

---

#### **3. DENSE_RANK() - Consecutive Ranking (No Gaps):**

```sql
-- DENSE_RANK: Same rank for ties, but NO gaps
-- Always consecutive numbers
SELECT 
    name,
    score,
    DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rank
FROM students
ORDER BY score DESC;

-- Result:
-- name    | score | dense_rank
-- --------|-------|------------
-- Alice   | 95    | 1
-- Bob     | 90    | 2
-- Charlie | 90    | 2           ← Same rank as Bob
-- David   | 85    | 3           ← No gap (not 4)
-- Eve     | 85    | 3
-- Frank   | 80    | 4           ← No gap (not 6)
```

**Use Cases for DENSE_RANK():**
```sql
-- ✅ Category rankings (no gaps preferred)
SELECT 
    product_name,
    category,
    price,
    DENSE_RANK() OVER (PARTITION BY category ORDER BY price DESC) AS price_rank
FROM products;
-- Want ranks 1, 2, 3, 4... not 1, 2, 2, 4, 6...

-- ✅ Level/tier assignments
WITH user_ranks AS (
    SELECT 
        username,
        total_points,
        DENSE_RANK() OVER (ORDER BY total_points DESC) AS tier
    FROM users
)
SELECT 
    username,
    total_points,
    CASE 
        WHEN tier <= 10 THEN 'Diamond'
        WHEN tier <= 50 THEN 'Platinum'
        WHEN tier <= 200 THEN 'Gold'
        ELSE 'Silver'
    END AS membership_tier
FROM user_ranks;

-- ✅ Consecutive grouping
SELECT 
    order_date,
    total_orders,
    DENSE_RANK() OVER (ORDER BY total_orders DESC) AS performance_group
FROM daily_order_summary;
-- Groups by performance level (1=best, 2=second best, etc.)
```

---

### **Side-by-Side Comparison:**

```sql
-- Complete comparison with multiple ties
CREATE TABLE exam_scores (
    student VARCHAR(50),
    score INTEGER
);

INSERT INTO exam_scores VALUES
    ('Alice', 100),
    ('Bob', 95),
    ('Charlie', 95),
    ('David', 95),      -- Three-way tie
    ('Eve', 90),
    ('Frank', 90),      -- Two-way tie
    ('Grace', 85);

SELECT 
    student,
    score,
    ROW_NUMBER() OVER w AS row_num,
    RANK() OVER w AS rank,
    DENSE_RANK() OVER w AS dense_rank,
    -- Show the gaps
    RANK() OVER w - DENSE_RANK() OVER w AS gap
FROM exam_scores
WINDOW w AS (ORDER BY score DESC)
ORDER BY score DESC;

-- Result:
-- student | score | row_num | rank | dense_rank | gap
-- --------|-------|---------|------|------------|----
-- Alice   | 100   | 1       | 1    | 1          | 0
-- Bob     | 95    | 2       | 2    | 2          | 0
-- Charlie | 95    | 3       | 2    | 2          | 0
-- David   | 95    | 4       | 2    | 2          | 0
-- Eve     | 90    | 5       | 5    | 3          | 2  ← RANK skipped 3,4
-- Frank   | 90    | 6       | 5    | 3          | 2
-- Grace   | 85    | 7       | 7    | 4          | 3  ← RANK skipped 6
```

---

### **With PARTITION BY:**

```sql
-- All three functions work with partitions
SELECT 
    department,
    employee_name,
    salary,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS row_num,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank,
    DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dense_rank
FROM employees
ORDER BY department, salary DESC;

-- Result: Ranking resets for each department
-- department   | employee_name | salary | row_num | rank | dense_rank
-- -------------|---------------|--------|---------|------|------------
-- Engineering  | Alice         | 120000 | 1       | 1    | 1
-- Engineering  | Bob           | 95000  | 2       | 2    | 2
-- Engineering  | Charlie       | 95000  | 3       | 2    | 2
-- Engineering  | Dan           | 85000  | 4       | 4    | 3
-- Sales        | Eve           | 90000  | 1       | 1    | 1  ← Resets
-- Sales        | Frank         | 80000  | 2       | 2    | 2
```

---

### **Practical Examples:**

#### **Example 1: E-commerce Product Rankings**

```sql
-- Top 3 products per category (including ties at position 3)
WITH product_ranks AS (
    SELECT 
        category,
        product_name,
        sales_count,
        RANK() OVER (PARTITION BY category ORDER BY sales_count DESC) AS rank
    FROM products
)
SELECT 
    category,
    product_name,
    sales_count
FROM product_ranks
WHERE rank <= 3
ORDER BY category, rank;
-- If two products tied for 3rd place, both included
```

#### **Example 2: Salary Bands**

```sql
-- Assign salary bands using DENSE_RANK
WITH salary_ranks AS (
    SELECT 
        employee_name,
        salary,
        DENSE_RANK() OVER (ORDER BY salary DESC) AS salary_band
    FROM employees
)
SELECT 
    employee_name,
    salary,
    salary_band,
    CASE 
        WHEN salary_band <= 5 THEN 'Executive'
        WHEN salary_band <= 15 THEN 'Senior'
        WHEN salary_band <= 30 THEN 'Mid-Level'
        ELSE 'Entry-Level'
    END AS level
FROM salary_ranks;
```

#### **Example 3: Leaderboard with Tie-Breaking**

```sql
-- Game leaderboard: Same score = same rank, but show order for ties
SELECT 
    player_name,
    score,
    RANK() OVER (ORDER BY score DESC) AS rank,
    ROW_NUMBER() OVER (ORDER BY score DESC, kills DESC, deaths ASC) AS display_order
FROM game_results
ORDER BY display_order;
-- RANK shows true rank, ROW_NUMBER for display order (breaks ties)
```

#### **Example 4: Top N Per Group**

```sql
-- Top 2 highest-paid employees per department
-- Want exactly 2 rows per department (use ROW_NUMBER)
WITH ranked_employees AS (
    SELECT 
        department,
        employee_name,
        salary,
        ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rn
    FROM employees
)
SELECT 
    department,
    employee_name,
    salary
FROM ranked_employees
WHERE rn <= 2;
-- Guaranteed 2 rows per department (even if salaries tied)

-- vs including ties (use RANK)
WITH ranked_employees AS (
    SELECT 
        department,
        employee_name,
        salary,
        RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank
    FROM employees
)
SELECT 
    department,
    employee_name,
    salary
FROM ranked_employees
WHERE rank <= 2;
-- May get more than 2 rows if salaries tied at position 2
```

---

### **Performance Considerations:**

```sql
-- All three have similar performance
-- Optimization tips:

-- 1. Create index on ORDER BY columns
CREATE INDEX idx_students_score ON students(score DESC);

-- 2. Create index for PARTITION BY + ORDER BY
CREATE INDEX idx_employees_dept_salary 
ON employees(department, salary DESC);

-- 3. Use EXPLAIN to check query plan
EXPLAIN ANALYZE
SELECT 
    employee_name,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC)
FROM employees;

-- 4. Consider materializing for complex queries
CREATE MATERIALIZED VIEW employee_rankings AS
SELECT 
    department,
    employee_name,
    salary,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank
FROM employees;

CREATE INDEX idx_emp_rank_dept ON employee_rankings(department, rank);
```

---

### **Common Mistakes:**

```sql
-- ❌ Using wrong function for use case
-- Want unique pagination numbers? Don't use RANK()
SELECT * FROM (
    SELECT *, RANK() OVER (ORDER BY created_at) AS page_num
    FROM products
) t
WHERE page_num BETWEEN 11 AND 20;
-- May skip rows if ties exist!

-- ✅ Use ROW_NUMBER for pagination
SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER (ORDER BY created_at, id) AS page_num
    FROM products
) t
WHERE page_num BETWEEN 11 AND 20;

-- ❌ Expecting DENSE_RANK when using RANK
SELECT COUNT(DISTINCT rank) FROM (
    SELECT RANK() OVER (ORDER BY score DESC) AS rank
    FROM students
) t;
-- Returns 5 (with gaps: 1,2,4,6)

-- ✅ Use DENSE_RANK for consecutive count
SELECT COUNT(DISTINCT dense_rank) FROM (
    SELECT DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rank
    FROM students
) t;
-- Returns 4 (no gaps: 1,2,3,4)

-- ❌ Not specifying ORDER BY (undefined behavior)
SELECT ROW_NUMBER() OVER () FROM students;
-- Random order!

-- ✅ Always specify ORDER BY
SELECT ROW_NUMBER() OVER (ORDER BY score DESC) FROM students;
```

---

### **Decision Matrix:**

```sql
-- Use ROW_NUMBER() when:
-- ✅ Need unique sequential numbers (pagination)
-- ✅ Deduplication (keep one, delete rest)
-- ✅ Deterministic ordering required

-- Use RANK() when:
-- ✅ Competition/sports rankings (Olympic-style)
-- ✅ Want to honor ties but show "true" position
-- ✅ "Top 3" should include all tied at 3rd

-- Use DENSE_RANK() when:
-- ✅ Want consecutive rankings (1,2,3,4...)
-- ✅ Categorizing/grouping by rank level
-- ✅ Gaps in ranking would be confusing
```

---

### **Combining Multiple Rankings:**

```sql
-- Use multiple ranking functions together
SELECT 
    student,
    score,
    -- Overall unique row number
    ROW_NUMBER() OVER (ORDER BY score DESC) AS row_num,
    
    -- Rank with gaps (competition style)
    RANK() OVER (ORDER BY score DESC) AS rank,
    
    -- Rank without gaps (level assignment)
    DENSE_RANK() OVER (ORDER BY score DESC) AS level,
    
    -- Percentile rank (0 to 1)
    PERCENT_RANK() OVER (ORDER BY score DESC) AS percent_rank,
    
    -- Cumulative distribution (0 to 1)
    CUME_DIST() OVER (ORDER BY score DESC) AS cume_dist
FROM exam_scores
ORDER BY score DESC;
```

---

### **Interview Tips:**
- Clearly explain: ROW_NUMBER (unique), RANK (gaps), DENSE_RANK (no gaps)
- Use visual example showing ties and how each function handles them
- Mention use cases: pagination (ROW_NUMBER), competitions (RANK), levels (DENSE_RANK)
- Show PARTITION BY works with all three
- Discuss performance (all similar, indexes help)
- Emphasize importance of ORDER BY
- Provide practical examples (leaderboards, top N per group)

</details>