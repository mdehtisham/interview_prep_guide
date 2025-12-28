
## **Indexes**

<details>
<summary>29. What is an index and why is it important?</summary>

### Answer:

An **index** is a database structure that improves the speed of data retrieval operations at the cost of additional storage space and slower write operations. Think of it like a book's index: instead of reading every page to find a topic, you check the index to jump directly to the relevant pages.

---

### **How Indexes Work:**

```sql
-- Without index: Sequential scan (reads every row)
SELECT * FROM employees WHERE employee_id = 12345;
-- Scans all rows: Row 1, Row 2, Row 3, ... Row 1,000,000
-- Time: O(n) - Linear

-- With index: Index scan (jumps directly to target)
CREATE INDEX idx_employee_id ON employees(employee_id);

SELECT * FROM employees WHERE employee_id = 12345;
-- Index lookup: Jumps directly to row 12345
-- Time: O(log n) - Logarithmic (much faster!)
```

---

### **Visual Example:**

```sql
-- Sample table
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    email VARCHAR(100),
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    salary INTEGER,
    department VARCHAR(50),
    hire_date DATE
);

-- Insert 1 million rows
INSERT INTO employees (email, first_name, last_name, salary, department, hire_date)
SELECT 
    'user' || i || '@example.com',
    'First' || i,
    'Last' || i,
    50000 + (random() * 50000)::INTEGER,
    CASE (random() * 4)::INTEGER
        WHEN 0 THEN 'Engineering'
        WHEN 1 THEN 'Sales'
        WHEN 2 THEN 'Marketing'
        ELSE 'HR'
    END,
    CURRENT_DATE - (random() * 3650)::INTEGER
FROM generate_series(1, 1000000) AS i;

-- Query WITHOUT index
EXPLAIN ANALYZE
SELECT * FROM employees WHERE email = 'user500000@example.com';

/*
Seq Scan on employees  (cost=0.00..25000.00 rows=1 width=100) (actual time=250.123..500.456 rows=1 loops=1)
  Filter: (email = 'user500000@example.com')
  Rows Removed by Filter: 999999
Planning Time: 0.123 ms
Execution Time: 500.789 ms  ← Slow! (500ms)
*/

-- Create index
CREATE INDEX idx_employees_email ON employees(email);

-- Query WITH index
EXPLAIN ANALYZE
SELECT * FROM employees WHERE email = 'user500000@example.com';

/*
Index Scan using idx_employees_email on employees  (cost=0.42..8.44 rows=1 width=100) (actual time=0.025..0.026 rows=1 loops=1)
  Index Cond: (email = 'user500000@example.com')
Planning Time: 0.098 ms
Execution Time: 0.045 ms  ← Fast! (0.045ms - 11,000x faster!)
*/
```

---

### **Why Indexes Are Important:**

#### **1. Dramatically Faster Queries:**

```sql
-- Performance comparison
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    sku VARCHAR(50),
    product_name VARCHAR(200),
    category VARCHAR(50),
    price DECIMAL(10,2)
);

-- Insert 10 million products
INSERT INTO products (sku, product_name, category, price)
SELECT 
    'SKU-' || i,
    'Product ' || i,
    'Category ' || (i % 100),
    (random() * 1000)::DECIMAL(10,2)
FROM generate_series(1, 10000000) AS i;

-- WITHOUT index on category
\timing on
SELECT COUNT(*) FROM products WHERE category = 'Category 42';
-- Time: 2500 ms (2.5 seconds)

-- WITH index on category
CREATE INDEX idx_products_category ON products(category);

SELECT COUNT(*) FROM products WHERE category = 'Category 42';
-- Time: 15 ms (0.015 seconds - 166x faster!)
```

#### **2. Efficient Sorting:**

```sql
-- Without index: Full table scan + sort
EXPLAIN ANALYZE
SELECT * FROM employees ORDER BY hire_date LIMIT 10;

/*
Limit  (cost=45000.00..45000.25 rows=10 width=100)
  ->  Sort  (cost=45000.00..47500.00 rows=1000000 width=100)
        Sort Key: hire_date
        Sort Method: external merge  Disk: 85000kB  ← Expensive!
        ->  Seq Scan on employees  (cost=0.00..25000.00 rows=1000000 width=100)
*/

-- With index: Already sorted, no sort needed
CREATE INDEX idx_employees_hire_date ON employees(hire_date);

EXPLAIN ANALYZE
SELECT * FROM employees ORDER BY hire_date LIMIT 10;

/*
Limit  (cost=0.42..0.86 rows=10 width=100)
  ->  Index Scan using idx_employees_hire_date on employees  (cost=0.42..44000.42 rows=1000000 width=100)
        ← No sort! Already sorted by index
*/
```

#### **3. Fast JOIN Operations:**

```sql
-- Without indexes: Slow nested loop or hash join
SELECT 
    c.customer_name,
    o.order_id,
    o.total
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id;
-- Sequential scan on orders for each customer (slow)

-- With indexes: Fast index lookups
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
CREATE INDEX idx_customers_customer_id ON customers(customer_id);

SELECT 
    c.customer_name,
    o.order_id,
    o.total
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id;
-- Index scan for each customer (fast)
```

#### **4. Unique Constraints:**

```sql
-- Indexes enforce uniqueness
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,  -- Automatically creates unique index
    email VARCHAR(100) UNIQUE    -- Automatically creates unique index
);

-- Try to insert duplicate email
INSERT INTO users (email) VALUES ('john@example.com');
INSERT INTO users (email) VALUES ('john@example.com');
-- ERROR: duplicate key value violates unique constraint "users_email_key"
-- Index prevents duplicates instantly (no table scan needed)
```

#### **5. Foreign Key Performance:**

```sql
-- Foreign keys benefit from indexes
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER REFERENCES customers(customer_id),
    order_date DATE
);

-- Without index on customer_id: Slow foreign key checks
-- Every INSERT/UPDATE on orders requires scanning customers table

-- With index: Fast foreign key validation
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
-- Now INSERT/UPDATE is fast (index lookup instead of scan)
```

---

### **Index Trade-offs:**

#### **Benefits:**
- ✅ **Faster SELECT queries** (10x to 1000x speedup)
- ✅ **Efficient sorting** (ORDER BY)
- ✅ **Fast JOINs** (index lookups)
- ✅ **Quick uniqueness checks** (UNIQUE constraints)
- ✅ **Reduced I/O** (less disk reads)

#### **Costs:**
- ❌ **Extra storage space** (indexes can be 20-50% of table size)
- ❌ **Slower INSERT/UPDATE/DELETE** (must update index too)
- ❌ **Maintenance overhead** (vacuum, reindex)
- ❌ **More complex query planning** (optimizer has more options)

```sql
-- Measure index overhead
SELECT 
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS total_size,
    pg_size_pretty(pg_relation_size(schemaname||'.'||tablename)) AS table_size,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename) - pg_relation_size(schemaname||'.'||tablename)) AS indexes_size
FROM pg_tables
WHERE schemaname = 'public'
  AND tablename = 'employees';

-- Result:
-- tablename | total_size | table_size | indexes_size
-- ----------|------------|------------|-------------
-- employees | 150 MB     | 100 MB     | 50 MB (indexes are 50% of table size)
```

---

### **When to Create Indexes:**

```sql
-- ✅ CREATE indexes for:

-- 1. Columns in WHERE clauses
CREATE INDEX idx_users_status ON users(status);
SELECT * FROM users WHERE status = 'active';

-- 2. Columns in JOIN conditions
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
SELECT * FROM customers c JOIN orders o ON c.customer_id = o.customer_id;

-- 3. Columns in ORDER BY
CREATE INDEX idx_products_price ON products(price);
SELECT * FROM products ORDER BY price DESC;

-- 4. Columns in GROUP BY
CREATE INDEX idx_sales_region ON sales(region);
SELECT region, SUM(amount) FROM sales GROUP BY region;

-- 5. Foreign key columns
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
CREATE INDEX idx_order_items_product_id ON order_items(product_id);

-- 6. Columns used for uniqueness
CREATE UNIQUE INDEX idx_users_email ON users(email);
```

---

### **Real-World Example:**

```sql
-- E-commerce order search
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER,
    order_date DATE,
    status VARCHAR(20),
    total DECIMAL(10,2)
);

-- Common queries:
-- 1. Find orders by customer
SELECT * FROM orders WHERE customer_id = 12345;

-- 2. Find recent orders
SELECT * FROM orders WHERE order_date >= CURRENT_DATE - 7;

-- 3. Find pending orders
SELECT * FROM orders WHERE status = 'pending';

-- 4. Find orders in date range
SELECT * FROM orders 
WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31'
ORDER BY order_date DESC;

-- Create indexes for these queries
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
CREATE INDEX idx_orders_order_date ON orders(order_date);
CREATE INDEX idx_orders_status ON orders(status);

-- Composite index for common filter + sort pattern
CREATE INDEX idx_orders_status_date ON orders(status, order_date DESC);
-- Perfect for: WHERE status = 'pending' ORDER BY order_date DESC

-- Check index usage
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan AS index_scans,
    idx_tup_read AS tuples_read,
    idx_tup_fetch AS tuples_fetched
FROM pg_stat_user_indexes
WHERE tablename = 'orders'
ORDER BY idx_scan DESC;
```

---

### **Index Limitations:**

```sql
-- ❌ Indexes DON'T help with:

-- 1. Functions on columns (without expression index)
CREATE INDEX idx_users_email ON users(email);
SELECT * FROM users WHERE LOWER(email) = 'john@example.com';
-- Index not used! (LOWER function)

-- Solution: Expression index
CREATE INDEX idx_users_email_lower ON users(LOWER(email));
SELECT * FROM users WHERE LOWER(email) = 'john@example.com';
-- Now index is used!

-- 2. Leading wildcards
CREATE INDEX idx_products_name ON products(product_name);
SELECT * FROM products WHERE product_name LIKE '%shoes%';
-- Index not used! (leading %)

SELECT * FROM products WHERE product_name LIKE 'shoes%';
-- Index CAN be used (no leading %)

-- 3. OR conditions across different columns
SELECT * FROM employees WHERE first_name = 'John' OR last_name = 'Smith';
-- Even with indexes on both columns, may not use them efficiently

-- 4. Small tables
SELECT * FROM small_lookup_table WHERE code = 'ABC';
-- Sequential scan might be faster for tables with < 1000 rows

-- 5. Retrieving large portions of table
SELECT * FROM employees WHERE salary > 30000;
-- If 90% of employees earn > 30000, sequential scan is faster
```

---

### **Monitoring Index Performance:**

```sql
-- 1. Check if indexes are being used
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
ORDER BY idx_scan ASC;  -- Low idx_scan = unused index

-- 2. Find unused indexes (candidates for removal)
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
WHERE idx_scan = 0
  AND indexrelid NOT IN (
      SELECT indexrelid FROM pg_index WHERE indisunique OR indisprimary
  );
-- These indexes are never used and waste space

-- 3. Check query performance with EXPLAIN
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM employees WHERE department = 'Engineering';
-- Look for "Seq Scan" vs "Index Scan"
```

---

### **Best Practices:**

```sql
-- 1. Index foreign keys
CREATE INDEX idx_orders_customer_id ON orders(customer_id);

-- 2. Index frequently queried columns
CREATE INDEX idx_users_email ON users(email);

-- 3. Use composite indexes for common query patterns
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);
-- Good for: WHERE customer_id = X ORDER BY order_date

-- 4. Don't over-index
-- ❌ Bad: 10 single-column indexes
-- ✅ Better: 3-4 well-chosen composite indexes

-- 5. Monitor and remove unused indexes
DROP INDEX IF EXISTS idx_rarely_used;

-- 6. Rebuild indexes periodically
REINDEX TABLE employees;

-- 7. Use partial indexes for filtered queries
CREATE INDEX idx_orders_pending ON orders(order_date) 
WHERE status = 'pending';
-- Smaller index, faster for this specific query
```

---

### **Interview Tips:**
- Define index: Data structure that speeds up queries (like book index)
- Explain how: Provides fast lookup path (O(log n) instead of O(n))
- State benefits: Faster SELECT, efficient sorting/JOINs, uniqueness enforcement
- Mention trade-offs: Extra storage, slower writes, maintenance overhead
- Show dramatic example: 1000x speedup with index vs without
- Discuss when to index: WHERE, JOIN, ORDER BY, foreign keys
- Note limitations: Functions on columns, leading wildcards, small tables
- Emphasize monitoring: Check index usage, remove unused indexes
- Provide real numbers: Sequential scan 500ms vs Index scan 0.045ms

</details>

<details>
<summary>30. What are the different types of indexes in PostgreSQL?</summary>

### Answer:

PostgreSQL supports several index types, each optimized for different data types and query patterns: **B-tree** (default), **Hash**, **GiST**, **GIN**, **BRIN**, and **SP-GiST**. Choosing the right index type can significantly improve query performance.

---

### **1. B-tree Index (Default):**

#### **Overview:**
- **Most common** index type (default when you CREATE INDEX)
- **Balanced tree** structure
- **Best for:** Equality and range queries (<, <=, =, >=, >)
- **Supports:** Sorting (ORDER BY)

#### **When to Use:**

```sql
-- Automatically created for PRIMARY KEY and UNIQUE
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,  -- B-tree index created automatically
    email VARCHAR(100) UNIQUE    -- B-tree index created automatically
);

-- Explicit B-tree index (default type)
CREATE INDEX idx_employees_salary ON employees(salary);
-- Same as:
CREATE INDEX idx_employees_salary ON employees USING btree(salary);

-- Perfect for range queries
SELECT * FROM employees WHERE salary BETWEEN 50000 AND 100000;
-- Uses B-tree index efficiently

SELECT * FROM employees WHERE salary > 75000;
-- Uses B-tree index efficiently

-- Perfect for sorting
SELECT * FROM employees ORDER BY salary DESC LIMIT 10;
-- B-tree index provides sorted order (no sort needed)

-- Pattern matching with leading characters
SELECT * FROM users WHERE email LIKE 'john%';
-- B-tree can optimize this (no leading wildcard)
```

#### **Examples:**

```sql
-- Single column B-tree
CREATE INDEX idx_products_price ON products(price);

-- Multi-column B-tree (composite)
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);
-- Good for:
-- WHERE customer_id = X
-- WHERE customer_id = X AND order_date = Y
-- WHERE customer_id = X ORDER BY order_date

-- Unique B-tree
CREATE UNIQUE INDEX idx_users_username ON users(username);

-- Partial B-tree (filtered)
CREATE INDEX idx_orders_pending ON orders(order_date) 
WHERE status = 'pending';
-- Smaller index, only indexes pending orders

-- Expression B-tree
CREATE INDEX idx_users_email_lower ON users(LOWER(email));
SELECT * FROM users WHERE LOWER(email) = 'john@example.com';
-- Handles case-insensitive searches
```

---

### **2. Hash Index:**

#### **Overview:**
- **Hash table** structure
- **Best for:** Simple equality comparisons (=) only
- **NOT good for:** Range queries, sorting, pattern matching
- **WAL-logged** since PostgreSQL 10 (safe for replication)

#### **When to Use:**

```sql
-- Hash index for equality-only queries
CREATE INDEX idx_sessions_token ON sessions USING hash(session_token);

-- Good for exact matches
SELECT * FROM sessions WHERE session_token = 'abc123xyz';
-- Uses hash index efficiently

-- ❌ NOT good for ranges
SELECT * FROM sessions WHERE session_token > 'abc';
-- Cannot use hash index (falls back to sequential scan)

-- ❌ NOT good for sorting
SELECT * FROM sessions ORDER BY session_token;
-- Cannot use hash index for sorting
```

#### **Hash vs B-tree:**

```sql
-- For equality queries, hash is slightly faster
-- But B-tree is more versatile (also handles ranges)

-- Hash index
CREATE INDEX idx_hash_id ON large_table USING hash(id);
SELECT * FROM large_table WHERE id = 12345;
-- Lookup: O(1) average case

-- B-tree index
CREATE INDEX idx_btree_id ON large_table USING btree(id);
SELECT * FROM large_table WHERE id = 12345;
-- Lookup: O(log n)

-- Verdict: B-tree is usually better (more versatile, minimal speed difference)
```

---

### **3. GiST Index (Generalized Search Tree):**

#### **Overview:**
- **Generalized** search tree (framework for custom index types)
- **Best for:** Geometric data, full-text search, custom types
- **Supports:** Overlaps, contains, is-contained-by, distance

#### **When to Use:**

```sql
-- 1. Geometric/Spatial data (PostGIS)
CREATE TABLE locations (
    location_id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    coordinates POINT
);

CREATE INDEX idx_locations_coords ON locations USING gist(coordinates);

-- Find nearby points
SELECT * FROM locations 
WHERE coordinates <-> point '(40.7128, -74.0060)' < 10;
-- Uses GiST index for distance queries

-- 2. Range types
CREATE TABLE reservations (
    reservation_id SERIAL PRIMARY KEY,
    room_id INTEGER,
    during TSRANGE  -- Timestamp range
);

CREATE INDEX idx_reservations_during ON reservations USING gist(during);

-- Find overlapping reservations
SELECT * FROM reservations 
WHERE during && '[2024-01-01 14:00, 2024-01-01 16:00)'::TSRANGE;
-- Uses GiST index for range overlaps

-- 3. Full-text search (basic)
CREATE INDEX idx_documents_content ON documents USING gist(to_tsvector('english', content));

SELECT * FROM documents 
WHERE to_tsvector('english', content) @@ to_tsquery('postgresql & index');
-- Uses GiST for text search
```

#### **Common Operators:**

```sql
-- && (overlaps)
SELECT * FROM events WHERE time_range && '[2024-01-01, 2024-01-31]'::DATERANGE;

-- <-> (distance)
SELECT * FROM locations ORDER BY coordinates <-> point '(0,0)' LIMIT 10;

-- @> (contains)
SELECT * FROM ranges WHERE range_column @> 50;

-- <@ (is contained by)
SELECT * FROM ranges WHERE range_column <@ '[0, 100]'::INT4RANGE;
```

---

### **4. GIN Index (Generalized Inverted Index):**

#### **Overview:**
- **Inverted index** (like book index: word → pages)
- **Best for:** Array, JSONB, full-text search, multiple values per row
- **Larger** than B-tree but very fast for containment queries

#### **When to Use:**

```sql
-- 1. JSONB columns
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    name VARCHAR(200),
    attributes JSONB
);

CREATE INDEX idx_products_attributes ON products USING gin(attributes);

-- Search in JSONB
SELECT * FROM products 
WHERE attributes @> '{"color": "red", "size": "large"}';
-- Uses GIN index efficiently

SELECT * FROM products 
WHERE attributes ? 'brand';  -- Key exists
-- Uses GIN index

-- 2. Array columns
CREATE TABLE articles (
    article_id SERIAL PRIMARY KEY,
    title VARCHAR(200),
    tags TEXT[]
);

CREATE INDEX idx_articles_tags ON articles USING gin(tags);

-- Find articles with specific tag
SELECT * FROM articles WHERE tags @> ARRAY['postgresql'];
-- Uses GIN index

SELECT * FROM articles WHERE tags && ARRAY['database', 'index'];
-- Uses GIN index (overlaps)

-- 3. Full-text search (preferred over GiST)
CREATE TABLE documents (
    document_id SERIAL PRIMARY KEY,
    title VARCHAR(200),
    content TEXT,
    search_vector TSVECTOR
);

CREATE INDEX idx_documents_search ON documents USING gin(search_vector);

-- Full-text search
SELECT * FROM documents 
WHERE search_vector @@ to_tsquery('english', 'postgresql & performance');
-- Uses GIN index (faster than GiST for text search)

-- 4. Multiple columns with GIN
CREATE INDEX idx_products_multi ON products USING gin(tags, attributes);
-- Can search either or both
```

#### **GIN vs GiST for Text Search:**

```sql
-- GIN: Faster searches, slower updates, larger size
CREATE INDEX idx_gin ON documents USING gin(to_tsvector('english', content));
-- Best for: Read-heavy workloads, large text corpus

-- GiST: Slower searches, faster updates, smaller size
CREATE INDEX idx_gist ON documents USING gist(to_tsvector('english', content));
-- Best for: Write-heavy workloads, frequent updates
```

---

### **5. BRIN Index (Block Range Index):**

#### **Overview:**
- **Block range** index (stores min/max per block range)
- **Very small** size (1000x smaller than B-tree)
- **Best for:** Very large tables with natural correlation (time series)
- **Good for:** Range queries on correlated data

#### **When to Use:**

```sql
-- Time-series data (naturally ordered by time)
CREATE TABLE sensor_readings (
    reading_id BIGSERIAL PRIMARY KEY,
    sensor_id INTEGER,
    reading_time TIMESTAMP,
    temperature DECIMAL(5,2)
);

-- BRIN index on time column
CREATE INDEX idx_readings_time ON sensor_readings USING brin(reading_time);

-- Efficient for range queries on large, ordered data
SELECT * FROM sensor_readings 
WHERE reading_time BETWEEN '2024-01-01' AND '2024-01-31';
-- Uses BRIN index (very small, but effective)

-- Check index size
SELECT 
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
WHERE tablename = 'sensor_readings';

-- Result:
-- indexname            | index_size
-- ---------------------|------------
-- idx_readings_time    | 48 kB      (BRIN)
-- vs B-tree would be:  | 50 MB      (B-tree)
-- 1000x smaller!
```

#### **BRIN Characteristics:**

```sql
-- ✅ Good for:
-- - Large tables (> 10 GB)
-- - Naturally ordered data (timestamps, IDs)
-- - Range queries
-- - Low-cardinality columns (few distinct values)

-- ❌ Bad for:
-- - Randomly ordered data
-- - Exact equality searches
-- - Small tables
-- - High-cardinality columns

-- Example: Log files (naturally ordered by time)
CREATE TABLE application_logs (
    log_id BIGSERIAL PRIMARY KEY,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    level VARCHAR(10),
    message TEXT
);

CREATE INDEX idx_logs_timestamp ON application_logs USING brin(timestamp);
-- Perfect: logs naturally ordered by time, table is huge

-- Bad example: Random UUIDs
CREATE TABLE random_data (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    value TEXT
);

CREATE INDEX idx_random_id ON random_data USING brin(id);
-- Bad: UUIDs are random, no correlation, BRIN won't help
```

---

### **6. SP-GiST Index (Space-Partitioned GiST):**

#### **Overview:**
- **Space-partitioned** generalized search tree
- **Best for:** Non-balanced data structures (quad-trees, tries)
- **Good for:** Phone numbers, IP addresses, geographic data

#### **When to Use:**

```sql
-- 1. IP addresses
CREATE TABLE ip_logs (
    log_id SERIAL PRIMARY KEY,
    ip_address INET,
    timestamp TIMESTAMP
);

CREATE INDEX idx_ip_logs_address ON ip_logs USING spgist(ip_address);

SELECT * FROM ip_logs WHERE ip_address << '192.168.1.0/24'::INET;
-- Uses SP-GiST for subnet queries

-- 2. Phone numbers (trie structure)
CREATE TABLE phone_directory (
    phone_id SERIAL PRIMARY KEY,
    phone_number VARCHAR(20)
);

CREATE INDEX idx_phone_numbers ON phone_directory USING spgist(phone_number);

SELECT * FROM phone_directory WHERE phone_number LIKE '555%';
-- Efficient prefix search with SP-GiST

-- 3. Geographic data (quad-tree)
CREATE INDEX idx_locations_point ON locations USING spgist(coordinates);
```

---

### **Index Type Comparison Table:**

| Index Type | Best For | Operators | Size | Update Speed |
|------------|----------|-----------|------|-------------|
| **B-tree** | General purpose, ranges, sorting | =, <, <=, >, >=, BETWEEN, LIKE 'abc%' | Medium | Fast |
| **Hash** | Equality only | = | Medium | Fast |
| **GiST** | Geometric, ranges, full-text | &&, <->, @>, <@ | Medium | Medium |
| **GIN** | JSONB, arrays, full-text | @>, ?, @@, && | Large | Slow |
| **BRIN** | Large ordered tables | =, <, <=, >, >=, BETWEEN | Very Small | Very Fast |
| **SP-GiST** | Non-balanced trees, IPs | <<, >>, &&, ~ | Medium | Medium |

---

### **Choosing the Right Index Type:**

```sql
-- Decision flowchart:

-- INTEGER/NUMERIC/DATE/TEXT column?
--   └─> Range queries, sorting needed?
--       ├─> Yes → B-tree (default)
--       └─> No, only equality → Hash (but B-tree still good)

-- JSONB column?
--   └─> GIN index

-- Array column?
--   └─> GIN index

-- Full-text search?
--   └─> Read-heavy → GIN
--   └─> Write-heavy → GiST

-- Geometric/spatial data?
--   └─> Complex queries → GiST
--   └─> Simple spatial → SP-GiST

-- Large table (> 10 GB) with natural ordering?
--   └─> BRIN index

-- IP addresses, phone numbers?
--   └─> SP-GiST

-- When in doubt?
--   └─> B-tree (works for 90% of cases)
```

---

### **Practical Examples by Use Case:**

```sql
-- Use Case 1: E-commerce product search
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,          -- B-tree (automatic)
    sku VARCHAR(50) UNIQUE,                 -- B-tree (automatic)
    name VARCHAR(200),
    price DECIMAL(10,2),
    category VARCHAR(50),
    tags TEXT[],
    attributes JSONB,
    created_at TIMESTAMP
);

CREATE INDEX idx_products_price ON products(price);           -- B-tree for ranges
CREATE INDEX idx_products_category ON products(category);     -- B-tree for equality
CREATE INDEX idx_products_tags ON products USING gin(tags);   -- GIN for array search
CREATE INDEX idx_products_attrs ON products USING gin(attributes);  -- GIN for JSONB
CREATE INDEX idx_products_name ON products USING gin(to_tsvector('english', name));  -- GIN for search

-- Use Case 2: Time-series sensor data
CREATE TABLE sensor_data (
    sensor_id INTEGER,
    timestamp TIMESTAMP,
    temperature DECIMAL(5,2),
    humidity DECIMAL(5,2)
);

CREATE INDEX idx_sensor_timestamp ON sensor_data USING brin(timestamp);  -- BRIN for huge time-series
CREATE INDEX idx_sensor_id ON sensor_data(sensor_id);  -- B-tree for exact matches

-- Use Case 3: Geographic application
CREATE TABLE restaurants (
    restaurant_id SERIAL PRIMARY KEY,
    name VARCHAR(200),
    location POINT,
    address JSONB
);

CREATE INDEX idx_restaurants_location ON restaurants USING gist(location);  -- GiST for spatial
CREATE INDEX idx_restaurants_address ON restaurants USING gin(address);     -- GIN for JSONB

-- Use Case 4: Session management
CREATE TABLE sessions (
    session_id SERIAL PRIMARY KEY,
    session_token VARCHAR(64) UNIQUE,  -- B-tree unique index
    user_id INTEGER,
    expires_at TIMESTAMP
);

CREATE INDEX idx_sessions_user ON sessions(user_id);         -- B-tree
CREATE INDEX idx_sessions_expires ON sessions(expires_at);   -- B-tree
```

---

### **Monitoring Index Types:**

```sql
-- Check index types in your database
SELECT 
    schemaname,
    tablename,
    indexname,
    pg_get_indexdef(indexrelid) AS index_definition,
    CASE 
        WHEN pg_get_indexdef(indexrelid) LIKE '%USING btree%' THEN 'B-tree'
        WHEN pg_get_indexdef(indexrelid) LIKE '%USING hash%' THEN 'Hash'
        WHEN pg_get_indexdef(indexrelid) LIKE '%USING gist%' THEN 'GiST'
        WHEN pg_get_indexdef(indexrelid) LIKE '%USING gin%' THEN 'GIN'
        WHEN pg_get_indexdef(indexrelid) LIKE '%USING brin%' THEN 'BRIN'
        WHEN pg_get_indexdef(indexrelid) LIKE '%USING spgist%' THEN 'SP-GiST'
        ELSE 'B-tree'  -- Default
    END AS index_type,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
ORDER BY tablename, indexname;
```

---

### **Interview Tips:**
- List all 6 types: B-tree, Hash, GiST, GIN, BRIN, SP-GiST
- Emphasize B-tree as default (90% of cases)
- Explain each type's specialty: GIN for JSONB/arrays, GiST for spatial, BRIN for huge time-series
- Show practical examples: JSONB → GIN, timestamps → BRIN, ranges → B-tree
- Mention trade-offs: GIN (large but fast for reads), BRIN (tiny but needs correlation)
- Provide decision guide: "When in doubt, use B-tree"
- Demonstrate with CREATE INDEX syntax for each type
- Discuss real use cases: e-commerce (B-tree + GIN), sensors (BRIN), maps (GiST)

</details>

<details>
<summary>31. What is the difference between B-tree and Hash indexes?</summary>

### Answer:

**B-tree** and **Hash** indexes serve different purposes: B-tree is a versatile, sorted tree structure perfect for range queries and sorting, while Hash is optimized for equality-only lookups. In practice, B-tree is almost always the better choice due to its versatility.

---

### **Key Differences:**

| Feature | B-tree Index | Hash Index |
|---------|--------------|------------|
| **Structure** | Balanced tree (sorted) | Hash table (unsorted) |
| **Default** | Yes (default type) | No (must specify USING hash) |
| **Equality (=)** | ✅ Fast O(log n) | ✅ Slightly faster O(1) |
| **Range (<, >, BETWEEN)** | ✅ Yes | ❌ No |
| **Sorting (ORDER BY)** | ✅ Yes | ❌ No |
| **Pattern (LIKE 'abc%')** | ✅ Yes | ❌ No |
| **Min/Max queries** | ✅ Fast | ❌ No |
| **Multi-column** | ✅ Yes | ✅ Yes (limited) |
| **Unique constraints** | ✅ Yes | ✅ Yes |
| **WAL-logged** | ✅ Yes | ✅ Yes (since PG 10) |
| **Size** | Medium | Medium (similar) |
| **Use case** | 90% of scenarios | Rare (equality only) |

---

### **1. B-tree Index (Balanced Tree):**

#### **Structure:**

```sql
-- B-tree maintains sorted order
-- Example: Index on employee_id

     [50]
    /    \
  [25]    [75]
  /  \    /  \
[10][40][60][90]

-- Tree is balanced: all leaves at same depth
-- Each node stores sorted keys
-- Fast lookup: O(log n) comparisons
```

#### **Capabilities:**

```sql
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    email VARCHAR(100),
    salary INTEGER,
    hire_date DATE
);

-- B-tree index (default)
CREATE INDEX idx_employees_salary ON employees(salary);

-- ✅ Equality queries (fast)
SELECT * FROM employees WHERE salary = 75000;
-- Index Scan: O(log n)

-- ✅ Range queries (efficient)
SELECT * FROM employees WHERE salary BETWEEN 50000 AND 100000;
-- Index Range Scan: finds start point, scans in order

SELECT * FROM employees WHERE salary > 60000;
-- Index Range Scan: efficient

-- ✅ Sorting (no sort needed)
SELECT * FROM employees ORDER BY salary DESC LIMIT 10;
-- Index scan in reverse order (already sorted)

-- ✅ Min/Max queries (instant)
SELECT MIN(salary) FROM employees;
SELECT MAX(salary) FROM employees;
-- Just read first/last leaf in index

-- ✅ Pattern matching (with leading characters)
CREATE INDEX idx_employees_email ON employees(email);
SELECT * FROM employees WHERE email LIKE 'john%';
-- Index scan: can use sorted order

-- ✅ Multi-column indexes
CREATE INDEX idx_employees_date_salary ON employees(hire_date, salary);
SELECT * FROM employees WHERE hire_date = '2024-01-01' ORDER BY salary;
-- Perfect: uses composite index for both filter and sort
```

---

### **2. Hash Index:**

#### **Structure:**

```sql
-- Hash index uses hash function
-- Example: Index on session_token

Hash Function: token → hash → bucket
'abc123' → hash('abc123') → bucket[42]
'xyz789' → hash('xyz789') → bucket[17]

Bucket Array:
[0] → NULL
[1] → NULL
...
[17] → 'xyz789' → row_pointer
...
[42] → 'abc123' → row_pointer
...

-- No sorting: buckets are unordered
-- Fast direct lookup: O(1) average case
-- Cannot do range scans (no order)
```

#### **Capabilities:**

```sql
CREATE TABLE sessions (
    session_id SERIAL PRIMARY KEY,
    session_token VARCHAR(64) UNIQUE,
    user_id INTEGER,
    created_at TIMESTAMP
);

-- Hash index (must specify)
CREATE INDEX idx_sessions_token ON sessions USING hash(session_token);

-- ✅ Equality queries (slightly faster than B-tree)
SELECT * FROM sessions WHERE session_token = 'abc123xyz';
-- Hash lookup: O(1) average

-- ❌ Range queries (NOT supported)
SELECT * FROM sessions WHERE session_token > 'abc';
-- ERROR or Sequential Scan (cannot use hash index)

-- ❌ Sorting (NOT supported)
SELECT * FROM sessions ORDER BY session_token;
-- Cannot use hash index for sorting (no order)

-- ❌ Pattern matching (NOT supported)
SELECT * FROM sessions WHERE session_token LIKE 'abc%';
-- Cannot use hash index

-- ❌ Min/Max queries (NOT supported)
SELECT MIN(session_token) FROM sessions;
-- Cannot use hash index (no order concept)
```

---

### **Performance Comparison:**

```sql
-- Create test table with 10 million rows
CREATE TABLE performance_test (
    id SERIAL PRIMARY KEY,
    lookup_key VARCHAR(50),
    data TEXT
);

INSERT INTO performance_test (lookup_key, data)
SELECT 
    'KEY-' || i,
    'Data for key ' || i
FROM generate_series(1, 10000000) AS i;

-- Test 1: Equality queries
-- B-tree index
CREATE INDEX idx_btree_key ON performance_test(lookup_key);

EXPLAIN ANALYZE
SELECT * FROM performance_test WHERE lookup_key = 'KEY-5000000';
/*
Index Scan using idx_btree_key
  (cost=0.43..8.45 rows=1)
  (actual time=0.035..0.036 rows=1)
Execution Time: 0.055 ms
*/

-- Hash index
DROP INDEX idx_btree_key;
CREATE INDEX idx_hash_key ON performance_test USING hash(lookup_key);

EXPLAIN ANALYZE
SELECT * FROM performance_test WHERE lookup_key = 'KEY-5000000';
/*
Index Scan using idx_hash_key
  (cost=0.00..8.02 rows=1)
  (actual time=0.028..0.029 rows=1)
Execution Time: 0.048 ms
*/

-- Result: Hash slightly faster (0.048ms vs 0.055ms)
-- But difference is negligible (0.007ms = 7 microseconds)

-- Test 2: Range queries
-- B-tree (works)
CREATE INDEX idx_btree_key ON performance_test(lookup_key);

EXPLAIN ANALYZE
SELECT * FROM performance_test 
WHERE lookup_key BETWEEN 'KEY-5000000' AND 'KEY-5001000';
/*
Index Scan using idx_btree_key
  (cost=0.43..50.45 rows=1000)
  (actual time=0.035..1.250 rows=1000)
Execution Time: 1.350 ms  ← Fast with B-tree
*/

-- Hash (fails)
DROP INDEX idx_btree_key;
CREATE INDEX idx_hash_key ON performance_test USING hash(lookup_key);

EXPLAIN ANALYZE
SELECT * FROM performance_test 
WHERE lookup_key BETWEEN 'KEY-5000000' AND 'KEY-5001000';
/*
Seq Scan on performance_test
  Filter: (lookup_key >= 'KEY-5000000' AND lookup_key <= 'KEY-5001000')
  Rows Removed by Filter: 9999000
  (actual time=2500..5000 rows=1000)
Execution Time: 5250 ms  ← Slow! Hash index not used
*/

-- Verdict: B-tree 1.35ms vs Hash 5250ms (3888x faster!)
```

---

### **When to Use Each:**

#### **B-tree: Use for almost everything (default)**

```sql
-- ✅ Use B-tree when you need:

-- 1. Range queries
CREATE INDEX idx_products_price ON products(price);
SELECT * FROM products WHERE price BETWEEN 10 AND 50;

-- 2. Sorting
CREATE INDEX idx_employees_hire_date ON employees(hire_date);
SELECT * FROM employees ORDER BY hire_date DESC;

-- 3. Min/Max aggregates
SELECT MIN(price), MAX(price) FROM products;

-- 4. Pattern matching (leading characters)
CREATE INDEX idx_users_email ON users(email);
SELECT * FROM users WHERE email LIKE 'john%';

-- 5. Composite indexes
CREATE INDEX idx_orders_cust_date ON orders(customer_id, order_date);
SELECT * FROM orders WHERE customer_id = 123 ORDER BY order_date;

-- 6. Inequality comparisons
SELECT * FROM products WHERE price > 100;

-- 7. General purpose (when unsure)
CREATE INDEX idx_any_column ON table_name(column_name);
-- B-tree handles everything
```

#### **Hash: Use only for specific equality-only cases (rare)**

```sql
-- ⚠️ Use Hash ONLY when:

-- 1. ONLY equality comparisons (no ranges ever)
CREATE INDEX idx_sessions_token ON sessions USING hash(session_token);
SELECT * FROM sessions WHERE session_token = 'abc123';
-- And you NEVER do:
-- - WHERE token > 'abc'  ❌
-- - WHERE token LIKE 'abc%'  ❌
-- - ORDER BY token  ❌

-- 2. Very large equality-heavy tables (minimal benefit)
CREATE INDEX idx_users_uuid ON users USING hash(user_uuid);
-- Only if you never need ranges/sorting

-- Reality: Hash rarely needed
-- B-tree handles equality just as well (0.007ms difference)
-- And B-tree gives you versatility for future queries
```

---

### **Real-World Examples:**

#### **Example 1: User Authentication**

```sql
-- Scenario: Look up users by email for login
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    email VARCHAR(100) UNIQUE,
    password_hash VARCHAR(255),
    created_at TIMESTAMP
);

-- Option 1: Hash index (equality only)
CREATE INDEX idx_users_email ON users USING hash(email);
SELECT * FROM users WHERE email = 'john@example.com';  -- ✅ Works

-- But later you need:
SELECT * FROM users WHERE email LIKE '%@example.com';  -- ❌ Can't use index
SELECT * FROM users ORDER BY email;  -- ❌ Can't use index
SELECT MIN(email), MAX(email) FROM users;  -- ❌ Can't use index

-- Option 2: B-tree index (versatile)
CREATE INDEX idx_users_email ON users USING btree(email);
SELECT * FROM users WHERE email = 'john@example.com';  -- ✅ Works
SELECT * FROM users WHERE email LIKE 'john%';  -- ✅ Works
SELECT * FROM users ORDER BY email;  -- ✅ Works
SELECT MIN(email), MAX(email) FROM users;  -- ✅ Works instantly

-- Verdict: B-tree is better (handles everything)
```

#### **Example 2: Session Tokens**

```sql
-- Scenario: Session lookup (only equality)
CREATE TABLE sessions (
    session_id BIGSERIAL PRIMARY KEY,
    session_token VARCHAR(64) UNIQUE,
    user_id INTEGER,
    expires_at TIMESTAMP
);

-- Hash index candidate (only = lookups)
CREATE INDEX idx_sessions_token ON sessions USING hash(session_token);
SELECT * FROM sessions WHERE session_token = 'abc123xyz';  -- ✅ Works

-- But B-tree also works great:
CREATE INDEX idx_sessions_token ON sessions(session_token);  -- B-tree
SELECT * FROM sessions WHERE session_token = 'abc123xyz';  -- ✅ Works (0.007ms slower)

-- Verdict: B-tree still better (minimal speed difference, more versatile)
```

#### **Example 3: Product Catalog**

```sql
-- Scenario: E-commerce products
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    sku VARCHAR(50) UNIQUE,
    name VARCHAR(200),
    price DECIMAL(10,2),
    category VARCHAR(50)
);

-- ❌ BAD: Hash index on price
CREATE INDEX idx_products_price ON products USING hash(price);
-- Problem: Users search price ranges
SELECT * FROM products WHERE price BETWEEN 10 AND 50;  -- Can't use hash!

-- ✅ GOOD: B-tree on price
CREATE INDEX idx_products_price ON products(price);
SELECT * FROM products WHERE price BETWEEN 10 AND 50;  -- Fast!
SELECT * FROM products WHERE price > 20;  -- Fast!
SELECT * FROM products ORDER BY price DESC;  -- Fast!
SELECT MIN(price), MAX(price) FROM products;  -- Instant!
```

---

### **Hash Index Limitations:**

```sql
-- Historical issues (fixed in PostgreSQL 10+)
-- Before PostgreSQL 10:
-- - Not WAL-logged (lost on crash)
-- - Not replicated to standby servers
-- - Required REINDEX after crash recovery

-- Since PostgreSQL 10:
-- ✅ WAL-logged (crash-safe)
-- ✅ Replicated properly
-- ✅ No special maintenance needed

-- But still limited:
-- ❌ No range queries
-- ❌ No sorting
-- ❌ No pattern matching
-- ❌ No min/max
-- ❌ Minimal performance advantage over B-tree

-- Conclusion: Use B-tree unless you have a very specific reason
```

---

### **Index Size Comparison:**

```sql
-- Check index sizes
SELECT 
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
    idx_scan AS times_used
FROM pg_stat_user_indexes
WHERE tablename = 'performance_test'
ORDER BY indexname;

-- Result:
-- indexname        | index_size | times_used
-- -----------------|------------|------------
-- idx_btree_key    | 214 MB     | 1,250,000
-- idx_hash_key     | 218 MB     | 1,000,000

-- Similar size, but B-tree more versatile
```

---

### **Migration Example:**

```sql
-- If you have a Hash index and need ranges:

-- Before: Hash index (equality only)
CREATE INDEX idx_orders_status ON orders USING hash(status);
SELECT * FROM orders WHERE status = 'pending';  -- Works

-- New requirement: Filter multiple statuses
SELECT * FROM orders WHERE status IN ('pending', 'processing');  -- Slow!

-- Solution: Switch to B-tree
DROP INDEX idx_orders_status;
CREATE INDEX idx_orders_status ON orders(status);  -- B-tree

SELECT * FROM orders WHERE status = 'pending';  -- Still fast
SELECT * FROM orders WHERE status IN ('pending', 'processing');  -- Now fast!
SELECT * FROM orders ORDER BY status;  -- Now possible!
```

---

### **Best Practices:**

```sql
-- ✅ DO: Use B-tree by default
CREATE INDEX idx_column ON table_name(column_name);
-- Handles 90%+ of use cases

-- ✅ DO: Use B-tree for versatility
-- Even if you only need equality now, future queries may need ranges

-- ❌ DON'T: Use Hash unless you're certain
-- Only if you're 100% sure you'll NEVER need:
-- - Range queries
-- - Sorting
-- - Pattern matching
-- - Min/Max queries

-- ⚠️ CONSIDER: Hash only for legacy compatibility
-- Or very specific high-throughput equality-only workloads

-- 💡 REALITY: B-tree is almost always the answer
CREATE INDEX idx_any_use_case ON table_name(column_name);  -- Just use B-tree
```

---

### **Summary Table:**

| Scenario | B-tree | Hash | Winner |
|----------|--------|------|--------|
| Equality queries (=) | ✅ Fast | ✅ Slightly faster | **Tie** (0.007ms diff) |
| Range queries | ✅ Fast | ❌ No support | **B-tree** |
| Sorting | ✅ Fast | ❌ No support | **B-tree** |
| Pattern matching | ✅ Works | ❌ No support | **B-tree** |
| Min/Max | ✅ Instant | ❌ No support | **B-tree** |
| Versatility | ✅ High | ❌ Low | **B-tree** |
| Future-proof | ✅ Yes | ❌ Limited | **B-tree** |
| **Overall** | | | **B-tree wins** |

---

### **Interview Tips:**
- State default: B-tree is default, Hash requires explicit USING hash
- Key difference: B-tree is sorted (ranges), Hash is unsorted (equality only)
- Performance: Hash slightly faster for equality (negligible: ~0.007ms)
- Capabilities: B-tree supports ranges/sorting/patterns, Hash doesn't
- Recommendation: Use B-tree 99% of the time (more versatile)
- Show example: Range query works with B-tree (1.35ms), fails with Hash (5250ms = 3888x slower)
- Mention history: Hash had issues before PG 10, now safe but still limited
- Practical advice: "When in doubt, use B-tree"

</details>

<details>
<summary>32. What is a GiST index?</summary>

### Answer:

**GiST (Generalized Search Tree)** is a balanced tree structure that provides a framework for implementing custom index types. It's perfect for **geometric data**, **range types**, **full-text search**, and other complex data types that don't fit the standard B-tree model.

---

### **GiST Overview:**

```sql
-- GiST = Generalized Search Tree
-- Think of it as a "plugin framework" for custom indexes

Key Features:
- Balanced tree (like B-tree)
- Extensible (custom operators and data types)
- Supports overlaps, contains, distance, intersects
- Used by PostGIS for spatial queries
- Good for full-text search (though GIN is better)
```

---

### **1. Geometric/Spatial Data (Most Common Use):**

#### **Point Data:**

```sql
-- Store locations as points
CREATE TABLE restaurants (
    restaurant_id SERIAL PRIMARY KEY,
    name VARCHAR(200),
    location POINT,  -- (latitude, longitude)
    cuisine VARCHAR(50)
);

-- Insert sample data
INSERT INTO restaurants (name, location, cuisine) VALUES
    ('Pizza Palace', point '(40.7589, -73.9851)', 'Italian'),     -- Times Square
    ('Sushi World', point '(40.7614, -73.9776)', 'Japanese'),     -- Central Park
    ('Burger House', point '(40.7128, -74.0060)', 'American'),    -- Downtown
    ('Taco Stand', point '(40.7282, -73.7949)', 'Mexican');       -- Queens

-- Create GiST index on location
CREATE INDEX idx_restaurants_location ON restaurants USING gist(location);

-- Query 1: Find nearby restaurants (distance query)
SELECT 
    name,
    location,
    location <-> point '(40.7580, -73.9855)' AS distance  -- Distance from Times Square
FROM restaurants
ORDER BY location <-> point '(40.7580, -73.9855)'
LIMIT 5;

/*
Result:
name          | location                | distance
--------------|-------------------------|----------
Pizza Palace  | (40.7589,-73.9851)      | 0.001
Sushi World   | (40.7614,-73.9776)      | 0.008
Burger House  | (40.7128,-74.0060)      | 0.053
Taco Stand    | (40.7282,-73.7949)      | 0.192

Uses GiST index for fast distance calculation
*/

-- Query 2: Find restaurants within radius
SELECT name, location
FROM restaurants
WHERE location <-> point '(40.7580, -73.9855)' < 0.01;  -- Within 0.01 degrees (~1km)
-- Uses GiST index efficiently

-- Query 3: Bounding box query
SELECT name, location
FROM restaurants
WHERE location <@ box '((40.75, -74.00), (40.77, -73.97))';  -- Within box
-- Uses GiST index for containment
```

#### **Geometric Shapes:**

```sql
-- Store regions as polygons, circles, boxes
CREATE TABLE delivery_zones (
    zone_id SERIAL PRIMARY KEY,
    zone_name VARCHAR(100),
    area POLYGON
);

-- Create GiST index
CREATE INDEX idx_zones_area ON delivery_zones USING gist(area);

-- Insert zones
INSERT INTO delivery_zones (zone_name, area) VALUES
    ('Downtown', polygon '((40.70,-74.02),(40.70,-73.98),(40.72,-73.98),(40.72,-74.02))'),
    ('Midtown', polygon '((40.75,-74.00),(40.75,-73.97),(40.78,-73.97),(40.78,-74.00))');

-- Check if point is within delivery zone
SELECT zone_name
FROM delivery_zones
WHERE area @> point '(40.7128, -74.0060)';  -- @> means "contains"
-- Uses GiST index

-- Find overlapping zones
SELECT z1.zone_name, z2.zone_name
FROM delivery_zones z1
JOIN delivery_zones z2 ON z1.area && z2.area  -- && means "overlaps"
WHERE z1.zone_id < z2.zone_id;
-- Uses GiST index for overlap detection
```

---

### **2. Range Types:**

#### **Date/Time Ranges:**

```sql
-- Hotel room reservations
CREATE TABLE reservations (
    reservation_id SERIAL PRIMARY KEY,
    room_id INTEGER,
    guest_name VARCHAR(100),
    during TSRANGE  -- Timestamp range: [check_in, check_out)
);

-- Create GiST index on range
CREATE INDEX idx_reservations_during ON reservations USING gist(during);

-- Insert reservations
INSERT INTO reservations (room_id, guest_name, during) VALUES
    (101, 'John Doe', '[2024-01-10 14:00, 2024-01-15 11:00)'),
    (101, 'Jane Smith', '[2024-01-16 15:00, 2024-01-20 10:00)'),
    (102, 'Bob Johnson', '[2024-01-12 14:00, 2024-01-18 11:00)');

-- Query 1: Find overlapping reservations (double-booking check)
SELECT 
    r1.guest_name AS guest1,
    r2.guest_name AS guest2,
    r1.during,
    r2.during
FROM reservations r1
JOIN reservations r2 ON r1.room_id = r2.room_id 
    AND r1.during && r2.during  -- && means "overlaps"
    AND r1.reservation_id < r2.reservation_id;

-- Uses GiST index for fast overlap detection

-- Query 2: Check availability for new reservation
SELECT room_id
FROM reservations
WHERE room_id = 101
  AND during && '[2024-01-17 14:00, 2024-01-22 11:00)'::TSRANGE;
  
-- Empty result = room available, else conflict exists
-- Uses GiST index

-- Query 3: Find reservations containing specific date
SELECT guest_name, during
FROM reservations
WHERE during @> '2024-01-14 10:00:00'::TIMESTAMP;  -- @> means "contains"
-- Returns reservations active at that time

-- Query 4: Find reservations within date range
SELECT guest_name, during
FROM reservations
WHERE during <@ '[2024-01-01, 2024-02-01)'::TSRANGE;  -- <@ means "contained by"
-- Returns reservations completely within January 2024
```

#### **Numeric Ranges:**

```sql
-- Product price ranges
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    product_name VARCHAR(200),
    price_range INT4RANGE  -- Integer range [min_price, max_price)
);

-- Create GiST index
CREATE INDEX idx_products_price_range ON products USING gist(price_range);

-- Insert products
INSERT INTO products (product_name, price_range) VALUES
    ('Budget Laptop', '[300, 500)'),
    ('Premium Laptop', '[1000, 2000)'),
    ('Mid-Range Laptop', '[600, 900)');

-- Find products with price overlapping target range
SELECT product_name, price_range
FROM products
WHERE price_range && '[400, 700)'::INT4RANGE;
-- Returns: Budget Laptop, Mid-Range Laptop

-- Find products within budget
SELECT product_name, price_range
FROM products
WHERE price_range <@ '[0, 600)'::INT4RANGE;
-- Returns: Budget Laptop (entirely within budget)
```

---

### **3. Full-Text Search:**

```sql
-- Document search (GiST for text search)
CREATE TABLE documents (
    document_id SERIAL PRIMARY KEY,
    title VARCHAR(200),
    content TEXT,
    search_vector TSVECTOR  -- Pre-computed search vector
);

-- Create GiST index for full-text search
CREATE INDEX idx_documents_search ON documents USING gist(search_vector);

-- Insert documents
INSERT INTO documents (title, content, search_vector) VALUES
    ('PostgreSQL Tutorial', 
     'Learn about indexes and query optimization',
     to_tsvector('english', 'PostgreSQL Tutorial Learn about indexes and query optimization')),
    ('Database Design',
     'Best practices for schema design and normalization',
     to_tsvector('english', 'Database Design Best practices for schema design and normalization'));

-- Full-text search query
SELECT title, content
FROM documents
WHERE search_vector @@ to_tsquery('english', 'postgresql & index');
-- Uses GiST index

-- Note: GIN is usually better for full-text search
-- GiST: Smaller index, faster updates, slower searches
-- GIN: Larger index, slower updates, faster searches
-- Use GiST if write-heavy, GIN if read-heavy
```

---

### **4. Network Addresses (INET/CIDR):**

```sql
-- IP address ranges
CREATE TABLE ip_whitelist (
    rule_id SERIAL PRIMARY KEY,
    description VARCHAR(200),
    ip_range INET
);

-- Create GiST index
CREATE INDEX idx_whitelist_range ON ip_whitelist USING gist(ip_range);

-- Insert IP ranges
INSERT INTO ip_whitelist (description, ip_range) VALUES
    ('Office Network', '192.168.1.0/24'),
    ('VPN Range', '10.0.0.0/8'),
    ('Partner Network', '172.16.0.0/16');

-- Check if specific IP is whitelisted
SELECT description
FROM ip_whitelist
WHERE ip_range >> '192.168.1.100'::INET;  -- >> means "contains"
-- Returns: Office Network

-- Find overlapping IP ranges
SELECT w1.description, w2.description
FROM ip_whitelist w1
JOIN ip_whitelist w2 ON w1.ip_range && w2.ip_range
WHERE w1.rule_id < w2.rule_id;
```

---

### **GiST Operators:**

```sql
-- Common GiST operators:

-- && (overlaps / intersects)
SELECT * FROM reservations WHERE during && '[2024-01-15, 2024-01-20)'::TSRANGE;

-- @> (contains)
SELECT * FROM delivery_zones WHERE area @> point '(40.7128, -74.0060)';

-- <@ (is contained by / within)
SELECT * FROM reservations WHERE during <@ '[2024-01-01, 2024-02-01)'::TSRANGE;

-- <-> (distance - for ordering)
SELECT * FROM restaurants ORDER BY location <-> point '(40.7580, -73.9855)' LIMIT 10;

-- << (strictly left of)
-- >> (strictly right of)
-- &< (does not extend to right of)
-- &> (does not extend to left of)

-- Full list depends on data type (geometric, range, etc.)
```

---

### **GiST vs B-tree:**

```sql
-- Comparison:

-- B-tree: Good for scalar values
CREATE INDEX idx_btree_date ON reservations(check_in_date);
SELECT * FROM reservations WHERE check_in_date = '2024-01-15';  -- Works
SELECT * FROM reservations WHERE check_in_date BETWEEN '2024-01-01' AND '2024-01-31';  -- Works

-- But: Cannot handle range overlaps
-- This query cannot use B-tree efficiently:
SELECT * FROM reservations 
WHERE check_in_date <= '2024-01-20' AND check_out_date >= '2024-01-15';
-- Needs to check two columns separately (inefficient)

-- GiST: Perfect for ranges
CREATE INDEX idx_gist_during ON reservations USING gist(during);
SELECT * FROM reservations WHERE during && '[2024-01-15, 2024-01-20)'::TSRANGE;  -- Efficient!
-- Single index operation checks overlap
```

---

### **Performance Example:**

```sql
-- Create large dataset: 1 million reservations
CREATE TABLE large_reservations (
    reservation_id SERIAL PRIMARY KEY,
    room_id INTEGER,
    during TSRANGE
);

INSERT INTO large_reservations (room_id, during)
SELECT 
    (random() * 1000)::INTEGER + 1,
    tsrange(
        CURRENT_DATE + (random() * 365)::INTEGER * INTERVAL '1 day',
        CURRENT_DATE + (random() * 365)::INTEGER * INTERVAL '1 day' + INTERVAL '3 days'
    )
FROM generate_series(1, 1000000);

-- Without GiST index
EXPLAIN ANALYZE
SELECT COUNT(*) FROM large_reservations
WHERE during && '[2024-06-01, 2024-06-30)'::TSRANGE;

/*
Seq Scan on large_reservations  
  Filter: (during && '[2024-06-01, 2024-06-30)'::TSRANGE)
  Rows Removed by Filter: 950000
Planning Time: 0.1 ms
Execution Time: 1250 ms  ← Slow (full table scan)
*/

-- With GiST index
CREATE INDEX idx_large_during ON large_reservations USING gist(during);

EXPLAIN ANALYZE
SELECT COUNT(*) FROM large_reservations
WHERE during && '[2024-06-01, 2024-06-30)'::TSRANGE;

/*
Bitmap Index Scan on idx_large_during
  Index Cond: (during && '[2024-06-01, 2024-06-30)'::TSRANGE)
Planning Time: 0.2 ms
Execution Time: 45 ms  ← Fast! (27x speedup)
*/
```

---

### **GiST Limitations:**

```sql
-- ❌ Not good for simple equality
CREATE INDEX idx_gist_status ON orders USING gist(status);
SELECT * FROM orders WHERE status = 'pending';
-- Better with B-tree (GiST is overkill for simple values)

-- ❌ Not good for high-cardinality scalar values
CREATE INDEX idx_gist_email ON users USING gist(email);
-- Better with B-tree (email is scalar, not geometric/range)

-- ❌ Larger than B-tree for simple data
-- GiST has overhead for supporting complex operators

-- ✅ Use GiST for:
-- - Geometric/spatial data
-- - Range types (overlaps, contains)
-- - Full-text search (if write-heavy)
-- - Custom data types with complex operators
```

---

### **Real-World Use Cases:**

#### **Use Case 1: Restaurant Finder (Geospatial)**

```sql
CREATE TABLE restaurants (
    restaurant_id SERIAL PRIMARY KEY,
    name VARCHAR(200),
    location POINT,
    rating DECIMAL(2,1)
);

CREATE INDEX idx_restaurants_location ON restaurants USING gist(location);

-- Find top 10 restaurants near user
SELECT name, rating, location <-> point '(40.7580, -73.9855)' AS distance
FROM restaurants
WHERE rating >= 4.0
ORDER BY location <-> point '(40.7580, -73.9855)'
LIMIT 10;
-- Uses GiST index for distance ordering
```

#### **Use Case 2: Meeting Room Scheduler**

```sql
CREATE TABLE bookings (
    booking_id SERIAL PRIMARY KEY,
    room_id INTEGER,
    meeting_title VARCHAR(200),
    time_slot TSRANGE
);

CREATE INDEX idx_bookings_slot ON bookings USING gist(time_slot);

-- Check room availability
SELECT room_id
FROM bookings
WHERE room_id = 5
  AND time_slot && '[2024-12-23 14:00, 2024-12-23 16:00)'::TSRANGE;
  
-- Empty result = available, otherwise conflict
```

#### **Use Case 3: IP Geolocation**

```sql
CREATE TABLE ip_locations (
    range_id SERIAL PRIMARY KEY,
    ip_range INET,
    country VARCHAR(100),
    city VARCHAR(100)
);

CREATE INDEX idx_ip_range ON ip_locations USING gist(ip_range);

-- Look up location for IP
SELECT country, city
FROM ip_locations
WHERE ip_range >> '203.0.113.45'::INET
LIMIT 1;
-- Uses GiST for fast subnet lookup
```

---

### **Maintenance:**

```sql
-- Check GiST index size and usage
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan AS times_used,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
WHERE indexname LIKE '%gist%'
ORDER BY idx_scan DESC;

-- Rebuild GiST index if needed
REINDEX INDEX idx_restaurants_location;

-- Or rebuild all GiST indexes on table
REINDEX TABLE restaurants;
```

---

### **Interview Tips:**
- Define: GiST = Generalized Search Tree, framework for custom indexes
- Key use: Geometric/spatial data, range types, full-text search
- Show operators: && (overlaps), @> (contains), <-> (distance), <@ (contained by)
- Provide example: Hotel reservations with TSRANGE for overlap detection
- Compare to B-tree: B-tree for scalars, GiST for complex types
- Mention PostGIS: GiST powers spatial queries in PostGIS extension
- Performance: Show ~27x speedup for range overlap queries
- When to use: Ranges, geometry, spatial data, overlaps, distance queries
- Practical: Meeting room conflicts, nearby restaurants, IP ranges

</details>

<details>
<summary>33. What is a GIN index and when would you use it?</summary>

### Answer:

**GIN (Generalized Inverted Index)** is an inverted index structure optimized for indexing composite values where each element can appear in multiple rows. It's perfect for **JSONB**, **arrays**, **full-text search**, and other multi-valued columns. Think of it like a book's index: each word points to all pages where it appears.

---

### **GIN Overview:**

```sql
-- GIN = Generalized Inverted Index
-- Structure: value → list of rows containing that value

Example - Tags array:
Row 1: tags = ['postgresql', 'database', 'sql']
Row 2: tags = ['mongodb', 'database', 'nosql']
Row 3: tags = ['postgresql', 'performance']

GIN Index Structure:
'postgresql' → [Row 1, Row 3]
'database'   → [Row 1, Row 2]
'sql'        → [Row 1]
'mongodb'    → [Row 2]
'nosql'      → [Row 2]
'performance'→ [Row 3]

Query: WHERE tags @> ARRAY['postgresql']
→ GIN lookup: 'postgresql' → [Row 1, Row 3]
→ Returns rows instantly (no sequential scan)
```

---

### **1. Array Columns (Most Common Use):**

#### **Basic Array Search:**

```sql
-- Articles with tags
CREATE TABLE articles (
    article_id SERIAL PRIMARY KEY,
    title VARCHAR(200),
    content TEXT,
    tags TEXT[],  -- Array of tags
    categories TEXT[]
);

-- Create GIN index on tags array
CREATE INDEX idx_articles_tags ON articles USING gin(tags);

-- Insert sample data
INSERT INTO articles (title, content, tags, categories) VALUES
    ('PostgreSQL Indexes', 'Learn about B-tree and GIN...', 
     ARRAY['postgresql', 'database', 'indexes', 'performance'], 
     ARRAY['tutorial', 'database']),
    ('MongoDB Basics', 'Introduction to NoSQL...', 
     ARRAY['mongodb', 'nosql', 'database'], 
     ARRAY['tutorial', 'nosql']),
    ('SQL Optimization', 'Query optimization techniques...', 
     ARRAY['sql', 'postgresql', 'optimization', 'performance'], 
     ARRAY['advanced', 'database']);

-- Query 1: Find articles with specific tag (contains)
SELECT title, tags
FROM articles
WHERE tags @> ARRAY['postgresql'];
-- @> means "contains"
-- Uses GIN index efficiently

/*
Result:
title                | tags
---------------------|----------------------------------------
PostgreSQL Indexes   | {postgresql,database,indexes,performance}
SQL Optimization     | {sql,postgresql,optimization,performance}

Execution: Index Scan using idx_articles_tags
*/

-- Query 2: Find articles with ANY of these tags (overlap)
SELECT title, tags
FROM articles
WHERE tags && ARRAY['mongodb', 'postgresql'];
-- && means "overlaps" (has at least one element in common)
-- Uses GIN index

/*
Result: All three articles (all have either mongodb or postgresql)
*/

-- Query 3: Find articles with ALL of these tags (contains all)
SELECT title, tags
FROM articles
WHERE tags @> ARRAY['postgresql', 'performance'];
-- Must contain BOTH tags
-- Uses GIN index

/*
Result:
PostgreSQL Indexes
SQL Optimization
*/

-- Query 4: Check if specific element exists
SELECT title
FROM articles
WHERE 'database' = ANY(tags);
-- Alternative syntax, but GIN index still helps
```

#### **Performance Comparison:**

```sql
-- Create large dataset: 1 million articles
CREATE TABLE large_articles (
    article_id SERIAL PRIMARY KEY,
    title VARCHAR(200),
    tags TEXT[]
);

INSERT INTO large_articles (title, tags)
SELECT 
    'Article ' || i,
    ARRAY[
        'tag' || ((random() * 100)::INTEGER),
        'tag' || ((random() * 100)::INTEGER),
        'tag' || ((random() * 100)::INTEGER)
    ]
FROM generate_series(1, 1000000) AS i;

-- Without GIN index
EXPLAIN ANALYZE
SELECT COUNT(*) FROM large_articles
WHERE tags @> ARRAY['tag42'];

/*
Seq Scan on large_articles
  Filter: (tags @> '{tag42}'::text[])
  Rows Removed by Filter: 990000
Execution Time: 2500 ms  ← Slow (full table scan)
*/

-- With GIN index
CREATE INDEX idx_large_tags ON large_articles USING gin(tags);

EXPLAIN ANALYZE
SELECT COUNT(*) FROM large_articles
WHERE tags @> ARRAY['tag42'];

/*
Bitmap Index Scan on idx_large_tags
  Index Cond: (tags @> '{tag42}'::text[])
Execution Time: 12 ms  ← Fast! (208x speedup)
*/
```

---

### **2. JSONB Columns (Very Common):**

#### **JSONB Search:**

```sql
-- Products with flexible attributes
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    name VARCHAR(200),
    price DECIMAL(10,2),
    attributes JSONB  -- Flexible attributes
);

-- Create GIN index on JSONB column
CREATE INDEX idx_products_attributes ON products USING gin(attributes);

-- Insert sample products
INSERT INTO products (name, price, attributes) VALUES
    ('Laptop Pro', 1299.99, '{
        "brand": "TechCorp",
        "color": "silver",
        "specs": {
            "cpu": "Intel i7",
            "ram": "16GB",
            "storage": "512GB SSD"
        },
        "features": ["backlit keyboard", "touchscreen", "fingerprint"]
    }'),
    ('Laptop Basic', 499.99, '{
        "brand": "BudgetTech",
        "color": "black",
        "specs": {
            "cpu": "Intel i3",
            "ram": "8GB",
            "storage": "256GB SSD"
        },
        "features": ["lightweight"]
    }'),
    ('Desktop Pro', 1899.99, '{
        "brand": "TechCorp",
        "color": "black",
        "specs": {
            "cpu": "Intel i9",
            "ram": "32GB",
            "storage": "1TB SSD"
        },
        "features": ["RGB lighting", "liquid cooling"]
    }');

-- Query 1: Find products by top-level key-value (containment)
SELECT name, attributes->>'brand' AS brand
FROM products
WHERE attributes @> '{"brand": "TechCorp"}';
-- @> means JSONB contains
-- Uses GIN index

/*
Result:
name        | brand
------------|----------
Laptop Pro  | TechCorp
Desktop Pro | TechCorp
*/

-- Query 2: Find products by nested key-value
SELECT name, attributes->'specs'->>'ram' AS ram
FROM products
WHERE attributes @> '{"specs": {"ram": "16GB"}}';
-- Works with nested JSONB
-- Uses GIN index

/*
Result:
name       | ram
-----------|-----
Laptop Pro | 16GB
*/

-- Query 3: Check if key exists
SELECT name
FROM products
WHERE attributes ? 'color';
-- ? means "key exists"
-- Uses GIN index

/*
Result: All products (all have color key)
*/

-- Query 4: Check if any of these keys exist
SELECT name
FROM products
WHERE attributes ?| ARRAY['warranty', 'color'];
-- ?| means "any of these keys exist"
-- Uses GIN index

-- Query 5: Check if all of these keys exist
SELECT name
FROM products
WHERE attributes ?& ARRAY['brand', 'color', 'price'];
-- ?& means "all of these keys exist"
-- Uses GIN index

-- Query 6: Find products with specific array element
SELECT name
FROM products
WHERE attributes->'features' @> '["touchscreen"]';
-- Search within JSONB array
-- Uses GIN index

/*
Result:
Laptop Pro
*/

-- Query 7: Complex query (multiple conditions)
SELECT name, price, attributes
FROM products
WHERE attributes @> '{"brand": "TechCorp"}'
  AND attributes->'specs' @> '{"ram": "16GB"}';
-- Uses GIN index for both conditions
```

#### **JSONB Path Queries:**

```sql
-- Advanced JSONB queries
SELECT name, attributes
FROM products
WHERE attributes @@ '$.specs.ram == "16GB"';
-- JSON path syntax (PostgreSQL 12+)
-- Can use GIN index with jsonb_path_ops

-- Extract and search
SELECT 
    name,
    jsonb_array_elements_text(attributes->'features') AS feature
FROM products
WHERE attributes->'features' @> '["touchscreen"]';
-- Expand array and filter
```

---

### **3. Full-Text Search (Preferred for Read-Heavy):**

#### **Full-Text Search with GIN:**

```sql
-- Document search
CREATE TABLE documents (
    document_id SERIAL PRIMARY KEY,
    title VARCHAR(200),
    content TEXT,
    search_vector TSVECTOR  -- Pre-computed search vector
);

-- Create GIN index for full-text search
CREATE INDEX idx_documents_search ON documents USING gin(search_vector);

-- Insert documents
INSERT INTO documents (title, content, search_vector) VALUES
    ('PostgreSQL Performance', 
     'Learn how to optimize PostgreSQL queries using indexes and EXPLAIN',
     to_tsvector('english', 'PostgreSQL Performance Learn how to optimize PostgreSQL queries using indexes and EXPLAIN')),
    ('Database Design Patterns',
     'Best practices for designing scalable database schemas',
     to_tsvector('english', 'Database Design Patterns Best practices for designing scalable database schemas')),
    ('Advanced SQL Techniques',
     'Master complex SQL queries with CTEs, window functions, and optimization',
     to_tsvector('english', 'Advanced SQL Techniques Master complex SQL queries with CTEs, window functions, and optimization'));

-- Full-text search query
SELECT 
    title,
    ts_rank(search_vector, query) AS rank
FROM documents, to_tsquery('english', 'postgresql & optimize') AS query
WHERE search_vector @@ query
ORDER BY rank DESC;
-- Uses GIN index for fast text search

/*
Result:
title                      | rank
---------------------------|--------
PostgreSQL Performance     | 0.0607927
Advanced SQL Techniques    | 0.0303964
*/

-- Search with multiple terms (OR)
SELECT title
FROM documents
WHERE search_vector @@ to_tsquery('english', 'postgresql | database | sql');
-- Matches documents with ANY of these terms
-- Uses GIN index

-- Search with phrase
SELECT title
FROM documents
WHERE search_vector @@ phraseto_tsquery('english', 'database design');
-- Matches phrase "database design"
-- Uses GIN index
```

#### **Auto-updating Search Vector:**

```sql
-- Automatically maintain search vector
CREATE TABLE articles_with_search (
    article_id SERIAL PRIMARY KEY,
    title VARCHAR(200),
    content TEXT,
    search_vector TSVECTOR GENERATED ALWAYS AS (
        to_tsvector('english', coalesce(title, '') || ' ' || coalesce(content, ''))
    ) STORED
);

CREATE INDEX idx_articles_search ON articles_with_search USING gin(search_vector);

-- Insert data (search_vector auto-generated)
INSERT INTO articles_with_search (title, content) VALUES
    ('GIN Index Tutorial', 'Learn about GIN indexes in PostgreSQL'),
    ('Full-Text Search', 'Implementing search functionality with tsvector');

-- Search automatically uses generated column
SELECT title
FROM articles_with_search
WHERE search_vector @@ to_tsquery('english', 'gin & postgresql');
```

---

### **4. Multi-Column GIN Index:**

```sql
-- Index multiple columns
CREATE TABLE products_multi (
    product_id SERIAL PRIMARY KEY,
    name VARCHAR(200),
    tags TEXT[],
    attributes JSONB
);

-- Multi-column GIN index
CREATE INDEX idx_products_multi ON products_multi USING gin(tags, attributes);

-- Can search either or both columns
SELECT name FROM products_multi WHERE tags @> ARRAY['electronics'];
SELECT name FROM products_multi WHERE attributes @> '{"brand": "Sony"}';
SELECT name FROM products_multi 
WHERE tags @> ARRAY['electronics'] 
  AND attributes @> '{"brand": "Sony"}';
-- All queries can use the same index
```

---

### **GIN vs GiST Comparison:**

```sql
-- GIN vs GiST for full-text search

-- Scenario: 1 million documents
CREATE TABLE comparison_docs (
    doc_id SERIAL PRIMARY KEY,
    content TEXT,
    search_vector TSVECTOR
);

-- Insert 1 million documents
INSERT INTO comparison_docs (content, search_vector)
SELECT 
    'Document content with words ' || i || ' and more text',
    to_tsvector('english', 'Document content with words ' || i || ' and more text')
FROM generate_series(1, 1000000) AS i;

-- GIN index
CREATE INDEX idx_gin ON comparison_docs USING gin(search_vector);

-- GiST index
CREATE INDEX idx_gist ON comparison_docs USING gist(search_vector);

-- Performance comparison:

-- Search speed (GIN wins)
-- GIN:  15 ms per search
-- GiST: 45 ms per search (3x slower)

-- Index size (GiST wins)
-- GIN:  250 MB
-- GiST: 180 MB (28% smaller)

-- Insert speed (GiST wins)
-- GIN:  0.5 ms per insert
-- GiST: 0.2 ms per insert (2.5x faster)

-- Update speed (GiST wins)
-- GIN:  1.2 ms per update
-- GiST: 0.4 ms per update (3x faster)
```

**Verdict:**
- **GIN**: Best for read-heavy workloads (more searches than updates)
- **GiST**: Best for write-heavy workloads (frequent updates)

---

### **GIN Operators:**

```sql
-- Array operators:
@>   -- Contains (array/jsonb contains elements)
<@   -- Contained by
&&   -- Overlaps (has common elements)
=    -- Equals

-- JSONB operators:
@>   -- Contains (JSONB contains key-value pairs)
<@   -- Contained by
?    -- Key exists
?|   -- Any of these keys exist
?&   -- All of these keys exist
@@   -- JSON path match

-- Full-text search operators:
@@   -- Matches tsquery
@@@  -- Matches (deprecated, use @@)

-- Examples:
SELECT * FROM articles WHERE tags @> ARRAY['postgresql'];         -- Contains
SELECT * FROM articles WHERE tags && ARRAY['sql', 'nosql'];       -- Overlaps
SELECT * FROM products WHERE attributes ? 'brand';                 -- Key exists
SELECT * FROM products WHERE attributes @> '{"color": "red"}';     -- JSONB contains
SELECT * FROM documents WHERE search_vector @@ to_tsquery('word'); -- Text search
```

---

### **GIN Index Options:**

#### **Default GIN vs GIN (jsonb_path_ops):**

```sql
-- Default GIN (indexes all keys and values)
CREATE INDEX idx_jsonb_default ON products USING gin(attributes);
-- Supports: @>, ?, ?|, ?&, <@

-- GIN with jsonb_path_ops (indexes only values for containment)
CREATE INDEX idx_jsonb_path ON products USING gin(attributes jsonb_path_ops);
-- Supports: @> only (faster, smaller, but limited)

-- Performance comparison:
-- jsonb_path_ops: 30% smaller, 20% faster for @> queries
-- Default: More versatile (supports all operators)

-- Use jsonb_path_ops if you ONLY use @> operator
CREATE INDEX idx_attrs_optimized ON products USING gin(attributes jsonb_path_ops);
SELECT * FROM products WHERE attributes @> '{"brand": "Sony"}';  -- ✅ Works

-- But this won't work:
SELECT * FROM products WHERE attributes ? 'brand';  -- ❌ Can't use jsonb_path_ops index
```

---

### **Real-World Use Cases:**

#### **Use Case 1: E-commerce Product Search**

```sql
CREATE TABLE ecommerce_products (
    product_id SERIAL PRIMARY KEY,
    name VARCHAR(200),
    description TEXT,
    tags TEXT[],
    attributes JSONB,
    search_vector TSVECTOR
);

-- Multiple GIN indexes
CREATE INDEX idx_prod_tags ON ecommerce_products USING gin(tags);
CREATE INDEX idx_prod_attrs ON ecommerce_products USING gin(attributes);
CREATE INDEX idx_prod_search ON ecommerce_products USING gin(search_vector);

-- Search by tags
SELECT name FROM ecommerce_products 
WHERE tags @> ARRAY['electronics', 'sale'];

-- Search by attributes
SELECT name FROM ecommerce_products
WHERE attributes @> '{"brand": "Samsung", "color": "black"}';

-- Full-text search
SELECT name FROM ecommerce_products
WHERE search_vector @@ to_tsquery('english', 'wireless & headphones');

-- Combined search
SELECT name, tags, attributes
FROM ecommerce_products
WHERE tags && ARRAY['electronics']
  AND attributes @> '{"price_range": "budget"}'
  AND search_vector @@ to_tsquery('english', 'bluetooth');
```

#### **Use Case 2: Social Media Tags**

```sql
CREATE TABLE posts (
    post_id SERIAL PRIMARY KEY,
    user_id INTEGER,
    content TEXT,
    tags TEXT[],
    created_at TIMESTAMP
);

CREATE INDEX idx_posts_tags ON posts USING gin(tags);

-- Find posts with specific hashtags
SELECT post_id, content
FROM posts
WHERE tags @> ARRAY['#postgresql', '#database'];

-- Find posts with any of these tags
SELECT post_id, content
FROM posts
WHERE tags && ARRAY['#tech', '#programming', '#coding'];

-- Trending tags (with GIN index support)
SELECT unnest(tags) AS tag, COUNT(*) AS count
FROM posts
WHERE created_at > CURRENT_DATE - INTERVAL '7 days'
GROUP BY tag
ORDER BY count DESC
LIMIT 10;
```

#### **Use Case 3: Configuration Management**

```sql
CREATE TABLE server_configs (
    server_id SERIAL PRIMARY KEY,
    hostname VARCHAR(100),
    config JSONB
);

CREATE INDEX idx_configs ON server_configs USING gin(config);

-- Find servers with specific configuration
SELECT hostname
FROM server_configs
WHERE config @> '{"monitoring": {"enabled": true}}';

-- Find servers with specific feature enabled
SELECT hostname
FROM server_configs
WHERE config->'features' @> '["auto_scaling"]';

-- Find servers with any of these settings
SELECT hostname
FROM server_configs
WHERE config ?| ARRAY['backup_enabled', 'monitoring_enabled'];
```

---

### **Performance and Maintenance:**

```sql
-- Check GIN index size and usage
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan AS times_used,
    idx_tup_read AS tuples_read,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
WHERE indexname LIKE '%gin%'
ORDER BY pg_relation_size(indexrelid) DESC;

-- GIN indexes can be large
-- Monitor size vs benefit

-- Find unused GIN indexes
SELECT 
    schemaname,
    tablename,
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS wasted_space
FROM pg_stat_user_indexes
WHERE indexname LIKE '%gin%'
  AND idx_scan = 0;
-- Consider dropping if never used

-- Rebuild GIN index (if fragmented)
REINDEX INDEX idx_products_attributes;
```

---

### **GIN Limitations:**

```sql
-- ❌ Not good for simple scalar values
CREATE INDEX idx_status ON orders USING gin(status);  -- Bad idea
-- Use B-tree instead

-- ❌ Slower for updates/inserts (builds inverted index)
-- Every insert/update must update index for each array element/JSONB key

-- ❌ Large index size
-- GIN indexes are typically 2-3x larger than B-tree

-- ✅ Perfect for:
-- - Arrays (tags, categories)
-- - JSONB (flexible attributes)
-- - Full-text search (read-heavy)
-- - Multi-valued data
```

---

### **Interview Tips:**
- Define: GIN = Generalized Inverted Index (value → rows containing that value)
- Primary use: Arrays, JSONB, full-text search
- Show operators: @> (contains), && (overlaps), ? (key exists), @@ (text search)
- Demonstrate: Array search with 208x speedup example
- Compare to GiST: GIN faster for reads, slower for writes, larger size
- JSONB examples: Product attributes, flexible schemas
- Mention jsonb_path_ops: 30% smaller, faster for containment-only queries
- Real-world: E-commerce (tags, attributes), social media (hashtags), search
- Trade-off: Slower writes but much faster multi-value searches

</details>

<details>
<summary>34. What is a partial index?</summary>

### Answer:

A **partial index** (also called a **filtered index**) indexes only a subset of rows in a table based on a WHERE condition. This creates a smaller, faster, more efficient index that's perfect for queries targeting specific data subsets.

---

### **Partial Index Overview:**

```sql
-- Regular index: Indexes ALL rows
CREATE INDEX idx_orders_status ON orders(status);
-- Indexes every status value for every row

-- Partial index: Indexes ONLY rows matching condition
CREATE INDEX idx_orders_pending ON orders(order_date) 
WHERE status = 'pending';
-- Only indexes rows where status = 'pending'
-- Much smaller index, faster queries on pending orders
```

**Benefits:**
- ✅ **Smaller index size** (only relevant rows)
- ✅ **Faster queries** (smaller index = faster scans)
- ✅ **Faster updates** (fewer index entries to maintain)
- ✅ **Less disk I/O** (smaller footprint)
- ✅ **Focused on hot data** (index what matters)

---

### **1. Basic Partial Index:**

#### **Status Filtering:**

```sql
-- E-commerce orders table
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER,
    order_date TIMESTAMP,
    status VARCHAR(20),  -- 'pending', 'processing', 'shipped', 'delivered', 'cancelled'
    total DECIMAL(10,2)
);

-- Insert sample data (1 million orders)
INSERT INTO orders (customer_id, order_date, status, total)
SELECT 
    (random() * 10000)::INTEGER + 1,
    CURRENT_TIMESTAMP - (random() * 365)::INTEGER * INTERVAL '1 day',
    CASE (random() * 100)::INTEGER
        WHEN 0..5 THEN 'pending'      -- 5% pending
        WHEN 6..10 THEN 'processing'  -- 5% processing
        ELSE 'delivered'               -- 90% delivered
    END,
    (random() * 500 + 10)::DECIMAL(10,2)
FROM generate_series(1, 1000000);

-- Regular full index (indexes all 1 million rows)
CREATE INDEX idx_orders_date_full ON orders(order_date);

-- Check size
SELECT pg_size_pretty(pg_relation_size('idx_orders_date_full'));
-- Result: 21 MB

-- Partial index (indexes only pending orders = 50,000 rows)
CREATE INDEX idx_orders_pending ON orders(order_date) 
WHERE status = 'pending';

-- Check size
SELECT pg_size_pretty(pg_relation_size('idx_orders_pending'));
-- Result: 1.1 MB (20x smaller!)

-- Query for pending orders
EXPLAIN ANALYZE
SELECT * FROM orders 
WHERE status = 'pending' 
  AND order_date > CURRENT_DATE - INTERVAL '7 days';

/*
Index Scan using idx_orders_pending
  Index Cond: (order_date > (CURRENT_DATE - '7 days'::interval))
  Filter: (status = 'pending')
Planning Time: 0.05 ms
Execution Time: 0.8 ms  ← Fast! (uses partial index)

Regular index would take: 3.5 ms
Speedup: 4.3x faster
*/
```

#### **Multiple Partial Indexes:**

```sql
-- Create partial indexes for different statuses
CREATE INDEX idx_orders_pending ON orders(order_date) WHERE status = 'pending';
CREATE INDEX idx_orders_processing ON orders(order_date) WHERE status = 'processing';
CREATE INDEX idx_orders_shipped ON orders(order_date) WHERE status = 'shipped';

-- Don't index 'delivered' (90% of data, use seq scan instead)

-- Query automatically picks correct partial index
SELECT * FROM orders WHERE status = 'pending' AND order_date > '2024-01-01';
-- Uses: idx_orders_pending

SELECT * FROM orders WHERE status = 'processing' AND order_date > '2024-01-01';
-- Uses: idx_orders_processing

-- Total size of 3 partial indexes: 3.3 MB
-- vs full index on all rows: 21 MB
-- Saving: 85% disk space!
```

---

### **2. Null Value Partial Indexes:**

#### **Index Only Non-Null Values:**

```sql
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    username VARCHAR(100),
    email VARCHAR(100),
    deleted_at TIMESTAMP  -- NULL for active users, timestamp for deleted
);

-- Most users are active (deleted_at IS NULL)
-- Only 2% are deleted

-- Bad: Full index (indexes NULLs too)
CREATE INDEX idx_users_deleted_full ON users(deleted_at);

-- Good: Partial index (indexes only deleted users)
CREATE INDEX idx_users_deleted ON users(deleted_at) 
WHERE deleted_at IS NOT NULL;
-- Much smaller, faster for deleted user queries

-- Even better: Index active users (if that's what you query)
CREATE INDEX idx_users_active ON users(user_id) 
WHERE deleted_at IS NULL;
-- Indexes only active users

-- Query active users
SELECT * FROM users WHERE deleted_at IS NULL;
-- Uses: idx_users_active
-- Fast! Index is 98% smaller
```

#### **Email Verification Example:**

```sql
CREATE TABLE accounts (
    account_id SERIAL PRIMARY KEY,
    email VARCHAR(100),
    email_verified BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP
);

-- Most accounts are verified
-- Only 5% are unverified

-- Partial index: Only unverified accounts
CREATE INDEX idx_accounts_unverified ON accounts(created_at) 
WHERE email_verified = FALSE;

-- Find old unverified accounts (for cleanup)
SELECT * FROM accounts 
WHERE email_verified = FALSE 
  AND created_at < CURRENT_DATE - INTERVAL '30 days';
-- Uses partial index (very fast on small subset)
```

---

### **3. Value Range Partial Indexes:**

#### **High-Value Transactions:**

```sql
CREATE TABLE transactions (
    transaction_id SERIAL PRIMARY KEY,
    user_id INTEGER,
    amount DECIMAL(10,2),
    transaction_date TIMESTAMP
);

-- Most transactions are small (< $100)
-- Only 1% are large (> $10,000)

-- Partial index: Only high-value transactions
CREATE INDEX idx_transactions_high_value ON transactions(transaction_date) 
WHERE amount > 10000;
-- Much smaller index for fraud detection/reporting on large transactions

-- Query high-value transactions
SELECT * FROM transactions 
WHERE amount > 10000 
  AND transaction_date > CURRENT_DATE - INTERVAL '30 days';
-- Uses partial index (fast!)

-- Fraud detection query
SELECT user_id, COUNT(*), SUM(amount)
FROM transactions
WHERE amount > 10000
  AND transaction_date > CURRENT_DATE - INTERVAL '1 day'
GROUP BY user_id
HAVING COUNT(*) > 5;
-- Detects users with >5 large transactions today
-- Uses partial index
```

#### **Date Range Partial Index:**

```sql
CREATE TABLE events (
    event_id SERIAL PRIMARY KEY,
    event_name VARCHAR(200),
    event_date DATE,
    status VARCHAR(20)
);

-- Partial index: Only future events
CREATE INDEX idx_events_future ON events(event_date) 
WHERE event_date >= CURRENT_DATE;
-- Past events rarely queried, don't waste index space

-- Query upcoming events
SELECT * FROM events 
WHERE event_date >= CURRENT_DATE 
  AND event_date <= CURRENT_DATE + INTERVAL '7 days';
-- Uses partial index

-- Old data doesn't bloat the index
-- Index stays small and fast
```

---

### **4. Complex Partial Index Conditions:**

#### **Multi-Condition Filtering:**

```sql
-- Payment processing
CREATE TABLE payments (
    payment_id SERIAL PRIMARY KEY,
    order_id INTEGER,
    amount DECIMAL(10,2),
    status VARCHAR(20),
    payment_method VARCHAR(50),
    retry_count INTEGER,
    created_at TIMESTAMP
);

-- Partial index: Failed payments that need retry
CREATE INDEX idx_payments_retry ON payments(created_at) 
WHERE status = 'failed' 
  AND retry_count < 3;
-- Only index payments eligible for retry

-- Retry failed payments
SELECT * FROM payments
WHERE status = 'failed'
  AND retry_count < 3
  AND created_at > CURRENT_TIMESTAMP - INTERVAL '1 hour'
ORDER BY created_at
LIMIT 100;
-- Uses partial index (very targeted)

-- Partial index: High-value credit card payments
CREATE INDEX idx_payments_cc_high ON payments(order_id)
WHERE payment_method = 'credit_card' 
  AND amount > 1000;
-- For fraud monitoring on expensive credit card transactions
```

#### **Compound Conditions:**

```sql
CREATE TABLE subscriptions (
    subscription_id SERIAL PRIMARY KEY,
    user_id INTEGER,
    plan_type VARCHAR(50),
    status VARCHAR(20),
    trial_ends_at TIMESTAMP,
    next_billing_date DATE
);

-- Partial index: Expiring trials (for conversion campaigns)
CREATE INDEX idx_subs_trial_expiring ON subscriptions(trial_ends_at)
WHERE status = 'trial' 
  AND trial_ends_at BETWEEN CURRENT_DATE AND CURRENT_DATE + INTERVAL '7 days';
-- Only indexes trials expiring in next 7 days

-- Partial index: Premium subscriptions due for billing
CREATE INDEX idx_subs_premium_billing ON subscriptions(next_billing_date)
WHERE plan_type IN ('premium', 'enterprise') 
  AND status = 'active';
-- Focus on high-value customers
```

---

### **5. Unique Partial Indexes:**

#### **Conditional Uniqueness:**

```sql
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    sku VARCHAR(50),
    name VARCHAR(200),
    deleted_at TIMESTAMP
);

-- Requirement: SKU must be unique among active products
-- But deleted products can have duplicate SKUs (for historical data)

-- Unique partial index: SKU unique only for non-deleted products
CREATE UNIQUE INDEX idx_products_sku_active ON products(sku) 
WHERE deleted_at IS NULL;

-- Insert active products (works)
INSERT INTO products (sku, name, deleted_at) VALUES
    ('SKU-001', 'Product A', NULL),
    ('SKU-002', 'Product B', NULL);

-- Try to insert duplicate active SKU (fails)
INSERT INTO products (sku, name, deleted_at) VALUES
    ('SKU-001', 'Product C', NULL);
-- ERROR: duplicate key value violates unique constraint

-- Soft-delete product
UPDATE products SET deleted_at = CURRENT_TIMESTAMP WHERE sku = 'SKU-001';

-- Now can insert new product with same SKU (works!)
INSERT INTO products (sku, name, deleted_at) VALUES
    ('SKU-001', 'Product A v2', NULL);
-- Success! Uniqueness only enforced on active products
```

#### **One Active Session Per User:**

```sql
CREATE TABLE sessions (
    session_id SERIAL PRIMARY KEY,
    user_id INTEGER,
    session_token VARCHAR(64),
    is_active BOOLEAN,
    created_at TIMESTAMP,
    expires_at TIMESTAMP
);

-- Requirement: Each user can have only ONE active session

-- Unique partial index
CREATE UNIQUE INDEX idx_sessions_one_active ON sessions(user_id) 
WHERE is_active = TRUE;

-- User logs in (creates active session)
INSERT INTO sessions (user_id, session_token, is_active, expires_at) VALUES
    (123, 'token-abc', TRUE, CURRENT_TIMESTAMP + INTERVAL '1 hour');

-- User tries to log in from another device (fails)
INSERT INTO sessions (user_id, session_token, is_active, expires_at) VALUES
    (123, 'token-xyz', TRUE, CURRENT_TIMESTAMP + INTERVAL '1 hour');
-- ERROR: duplicate key value violates unique constraint

-- To allow new login, deactivate old session first
UPDATE sessions SET is_active = FALSE WHERE user_id = 123 AND is_active = TRUE;

-- Now new login works
INSERT INTO sessions (user_id, session_token, is_active, expires_at) VALUES
    (123, 'token-xyz', TRUE, CURRENT_TIMESTAMP + INTERVAL '1 hour');
-- Success!
```

---

### **6. Performance Comparison:**

```sql
-- Create test table with 10 million rows
CREATE TABLE performance_test (
    id SERIAL PRIMARY KEY,
    category VARCHAR(20),
    value INTEGER,
    created_at TIMESTAMP
);

-- 95% category = 'standard', 5% category = 'premium'
INSERT INTO performance_test (category, value, created_at)
SELECT 
    CASE WHEN random() < 0.95 THEN 'standard' ELSE 'premium' END,
    (random() * 1000)::INTEGER,
    CURRENT_TIMESTAMP - (random() * 365)::INTEGER * INTERVAL '1 day'
FROM generate_series(1, 10000000);

-- Test 1: Full index
CREATE INDEX idx_full ON performance_test(created_at);

SELECT pg_size_pretty(pg_relation_size('idx_full'));
-- Size: 214 MB

EXPLAIN ANALYZE
SELECT * FROM performance_test 
WHERE category = 'premium' 
  AND created_at > CURRENT_DATE - INTERVAL '30 days';
-- Execution Time: 150 ms

-- Test 2: Partial index (only premium)
DROP INDEX idx_full;
CREATE INDEX idx_partial ON performance_test(created_at) 
WHERE category = 'premium';

SELECT pg_size_pretty(pg_relation_size('idx_partial'));
-- Size: 11 MB (19x smaller!)

EXPLAIN ANALYZE
SELECT * FROM performance_test 
WHERE category = 'premium' 
  AND created_at > CURRENT_DATE - INTERVAL '30 days';
-- Execution Time: 25 ms (6x faster!)

-- Savings: 203 MB disk space, 6x faster queries
```

---

### **When to Use Partial Indexes:**

```sql
-- ✅ Use partial indexes when:

-- 1. Querying small subset of table
WHERE status IN ('pending', 'processing')  -- 10% of data

-- 2. Filtering by boolean flags
WHERE deleted_at IS NULL  -- Active records
WHERE email_verified = FALSE  -- Unverified accounts

-- 3. Focusing on value ranges
WHERE amount > 10000  -- High-value transactions
WHERE priority = 'urgent'  -- Urgent items

-- 4. Time-based filtering
WHERE created_at >= CURRENT_DATE - INTERVAL '30 days'  -- Recent data

-- 5. Enforcing conditional uniqueness
CREATE UNIQUE INDEX idx_unique_active ON table(column) WHERE is_active = TRUE;

-- ❌ Don't use partial indexes when:

-- 1. Queries target most rows
WHERE status = 'completed'  -- 95% of data (use seq scan or full index)

-- 2. Filter conditions change frequently
WHERE created_at > '2024-01-01'  -- Date changes, index becomes stale

-- 3. Multiple different conditions
-- Can't create partial index for every possible query combination
```

---

### **Monitoring Partial Indexes:**

```sql
-- Check partial index usage
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan AS times_used,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
    pg_get_indexdef(indexrelid) AS definition
FROM pg_stat_user_indexes
WHERE pg_get_indexdef(indexrelid) LIKE '%WHERE%'  -- Partial indexes have WHERE
ORDER BY idx_scan DESC;

-- Find unused partial indexes
SELECT 
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS wasted_space,
    pg_get_indexdef(indexrelid) AS definition
FROM pg_stat_user_indexes
WHERE pg_get_indexdef(indexrelid) LIKE '%WHERE%'
  AND idx_scan = 0;
-- Consider dropping if never used
```

---

### **Best Practices:**

```sql
-- ✅ DO: Use partial indexes for frequently queried subsets
CREATE INDEX idx_recent_orders ON orders(order_date) 
WHERE created_at > CURRENT_DATE - INTERVAL '90 days';

-- ✅ DO: Combine with other index types
CREATE INDEX idx_orders_pending_gin ON orders USING gin(tags) 
WHERE status = 'pending';

-- ✅ DO: Use for conditional uniqueness
CREATE UNIQUE INDEX idx_active_username ON users(username) 
WHERE deleted_at IS NULL;

-- ✅ DO: Keep WHERE condition simple
WHERE status = 'pending'  -- Good
WHERE status = 'pending' AND created_at > NOW() - INTERVAL '1 day'  -- Too specific

-- ❌ DON'T: Create too many partial indexes
-- Don't create 10 different partial indexes for 10 different statuses
-- Create 2-3 for common queries, let other queries use full index

-- ❌ DON'T: Use complex expressions in WHERE
WHERE EXTRACT(MONTH FROM created_at) = 12  -- Bad (complex)
WHERE created_at >= '2024-12-01' AND created_at < '2025-01-01'  -- Better

-- 💡 TIP: Combine partial indexes with specific queries
-- Make sure query WHERE clause includes partial index condition
-- SELECT ... WHERE status = 'pending' AND order_date > X  ✅
-- SELECT ... WHERE order_date > X  ❌ (missing status condition)
```

---

### **Interview Tips:**
- Define: Partial index = filtered index, only indexes rows matching WHERE condition
- Key benefit: Smaller size, faster queries, less maintenance overhead
- Show example: 1 million rows → 50k rows indexed = 20x smaller index
- Demonstrate: 6x faster queries with partial index vs full index
- Use cases: Status filtering, NULL exclusion, value ranges, date ranges
- Unique partial: Enforce conditional uniqueness (e.g., active SKUs only)
- Practical: Pending orders, unverified accounts, high-value transactions, future events
- Performance: 19x smaller index, 6x faster queries, 203 MB saved
- Best practice: Use for frequently queried small subsets (5-20% of data)

</details>

<details>
<summary>35. What is a unique index?</summary>

### Answer:

A **unique index** ensures that all values in the indexed column(s) are distinct, preventing duplicate entries. It's automatically created for PRIMARY KEY and UNIQUE constraints and can also be created explicitly to enforce data integrity.

---

### **Unique Index Overview:**

```sql
-- Unique index prevents duplicates
CREATE UNIQUE INDEX idx_users_email ON users(email);

-- Try to insert duplicate
INSERT INTO users (email) VALUES ('john@example.com');
INSERT INTO users (email) VALUES ('john@example.com');
-- ERROR: duplicate key value violates unique constraint "idx_users_email"

-- Unique index enforces uniqueness instantly (no full table scan)
```

**Key Features:**
- ✅ **Prevents duplicates** (enforces uniqueness constraint)
- ✅ **Fast lookups** (like regular indexes)
- ✅ **Data integrity** (database-level validation)
- ✅ **Automatically created** for PRIMARY KEY and UNIQUE constraints
- ✅ **Allows NULLs** (multiple NULLs permitted by default)

---

### **1. Automatic Unique Indexes:**

#### **PRIMARY KEY (Automatic Unique Index):**

```sql
-- Primary key automatically creates unique index
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,  -- Creates unique index automatically
    username VARCHAR(100),
    email VARCHAR(100)
);

-- Check indexes
\d users

/*
Indexes:
    "users_pkey" PRIMARY KEY, btree (user_id)  ← Automatic unique index
*/

-- Try to insert duplicate primary key
INSERT INTO users (user_id, username, email) VALUES (1, 'john', 'john@example.com');
INSERT INTO users (user_id, username, email) VALUES (1, 'jane', 'jane@example.com');
-- ERROR: duplicate key value violates unique constraint "users_pkey"
```

#### **UNIQUE Constraint (Automatic Unique Index):**

```sql
-- UNIQUE constraint automatically creates unique index
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    sku VARCHAR(50) UNIQUE,  -- Creates unique index automatically
    name VARCHAR(200)
);

-- Check indexes
\d products

/*
Indexes:
    "products_pkey" PRIMARY KEY, btree (product_id)
    "products_sku_key" UNIQUE CONSTRAINT, btree (sku)  ← Automatic unique index
*/

-- Try to insert duplicate SKU
INSERT INTO products (sku, name) VALUES ('SKU-001', 'Product A');
INSERT INTO products (sku, name) VALUES ('SKU-001', 'Product B');
-- ERROR: duplicate key value violates unique constraint "products_sku_key"
```

---

### **2. Explicit Unique Indexes:**

#### **Create Unique Index Manually:**

```sql
-- Create table without UNIQUE constraint
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    email VARCHAR(100),
    employee_code VARCHAR(20),
    department VARCHAR(50)
);

-- Add unique index explicitly
CREATE UNIQUE INDEX idx_employees_email ON employees(email);

-- Now email must be unique
INSERT INTO employees (email, employee_code, department) VALUES
    ('alice@company.com', 'EMP001', 'Engineering');

-- Duplicate email fails
INSERT INTO employees (email, employee_code, department) VALUES
    ('alice@company.com', 'EMP002', 'Sales');
-- ERROR: duplicate key value violates unique constraint "idx_employees_email"

-- Different email works
INSERT INTO employees (email, employee_code, department) VALUES
    ('bob@company.com', 'EMP002', 'Sales');
-- Success!
```

#### **Unique Index on Expression:**

```sql
-- Case-insensitive unique email
CREATE UNIQUE INDEX idx_users_email_lower ON users(LOWER(email));

-- Now email is unique regardless of case
INSERT INTO users (username, email) VALUES ('user1', 'John@Example.com');
INSERT INTO users (username, email) VALUES ('user2', 'john@example.com');
-- ERROR: duplicate key value (case-insensitive)

-- This is different from regular unique index
CREATE UNIQUE INDEX idx_users_email_case ON users(email);
-- This allows: 'John@Example.com' and 'john@example.com' (different case)
```

---

### **3. Multi-Column Unique Indexes:**

#### **Composite Uniqueness:**

```sql
-- Flight bookings: unique combination of flight + seat
CREATE TABLE bookings (
    booking_id SERIAL PRIMARY KEY,
    flight_number VARCHAR(10),
    seat_number VARCHAR(5),
    passenger_name VARCHAR(100),
    booking_date TIMESTAMP
);

-- Unique index on flight + seat combination
CREATE UNIQUE INDEX idx_bookings_flight_seat ON bookings(flight_number, seat_number);

-- Book seat 12A on flight AA100
INSERT INTO bookings (flight_number, seat_number, passenger_name) VALUES
    ('AA100', '12A', 'John Doe');

-- Try to book same seat on same flight (fails)
INSERT INTO bookings (flight_number, seat_number, passenger_name) VALUES
    ('AA100', '12A', 'Jane Smith');
-- ERROR: duplicate key value

-- Same seat on different flight (works)
INSERT INTO bookings (flight_number, seat_number, passenger_name) VALUES
    ('AA200', '12A', 'Jane Smith');
-- Success! Different flight_number

-- Different seat on same flight (works)
INSERT INTO bookings (flight_number, seat_number, passenger_name) VALUES
    ('AA100', '13B', 'Jane Smith');
-- Success! Different seat_number
```

#### **Real-World Example: Event Registration:**

```sql
CREATE TABLE event_registrations (
    registration_id SERIAL PRIMARY KEY,
    event_id INTEGER,
    user_id INTEGER,
    registration_date TIMESTAMP
);

-- Ensure each user can register for each event only once
CREATE UNIQUE INDEX idx_registrations_event_user ON event_registrations(event_id, user_id);

-- User 123 registers for event 5
INSERT INTO event_registrations (event_id, user_id) VALUES (5, 123);

-- Same user tries to register again for same event (fails)
INSERT INTO event_registrations (event_id, user_id) VALUES (5, 123);
-- ERROR: duplicate key value

-- Same user registers for different event (works)
INSERT INTO event_registrations (event_id, user_id) VALUES (6, 123);
-- Success!

-- Different user registers for same event (works)
INSERT INTO event_registrations (event_id, user_id) VALUES (5, 456);
-- Success!
```

---

### **4. NULLs in Unique Indexes:**

#### **Default Behavior (Multiple NULLs Allowed):**

```sql
CREATE TABLE contacts (
    contact_id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    phone VARCHAR(20),
    email VARCHAR(100)
);

-- Unique index on email
CREATE UNIQUE INDEX idx_contacts_email ON contacts(email);

-- Insert contacts with NULL emails
INSERT INTO contacts (name, phone, email) VALUES ('John', '555-0001', NULL);
INSERT INTO contacts (name, phone, email) VALUES ('Jane', '555-0002', NULL);
INSERT INTO contacts (name, phone, email) VALUES ('Bob', '555-0003', NULL);
-- All succeed! Multiple NULLs are allowed

-- But duplicate non-NULL email fails
INSERT INTO contacts (name, phone, email) VALUES ('Alice', '555-0004', 'test@example.com');
INSERT INTO contacts (name, phone, email) VALUES ('Charlie', '555-0005', 'test@example.com');
-- ERROR: duplicate key value (non-NULL duplicates not allowed)

/*
Explanation: In PostgreSQL, NULL is "not equal" to NULL
So multiple NULLs don't violate uniqueness
*/
```

#### **Prevent Multiple NULLs (Partial Unique Index):**

```sql
-- If you want to allow only ONE NULL
CREATE UNIQUE INDEX idx_contacts_email_one_null ON contacts(email)
WHERE email IS NOT NULL;

-- Add NOT NULL constraint to prevent NULLs entirely
ALTER TABLE contacts ALTER COLUMN email SET NOT NULL;

-- Or combine at table creation
CREATE TABLE strict_contacts (
    contact_id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100) UNIQUE NOT NULL  -- No NULLs, no duplicates
);
```

---

### **5. Unique Partial Indexes (Conditional Uniqueness):**

#### **Unique Among Active Records:**

```sql
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    sku VARCHAR(50),
    name VARCHAR(200),
    is_active BOOLEAN DEFAULT TRUE
);

-- SKU must be unique only among active products
CREATE UNIQUE INDEX idx_products_sku_active ON products(sku)
WHERE is_active = TRUE;

-- Add active products (unique SKUs)
INSERT INTO products (sku, name, is_active) VALUES
    ('SKU-001', 'Product A', TRUE),
    ('SKU-002', 'Product B', TRUE);

-- Try duplicate active SKU (fails)
INSERT INTO products (sku, name, is_active) VALUES
    ('SKU-001', 'Product C', TRUE);
-- ERROR: duplicate key value

-- Deactivate product
UPDATE products SET is_active = FALSE WHERE sku = 'SKU-001';

-- Now can reuse SKU for new active product (works!)
INSERT INTO products (sku, name, is_active) VALUES
    ('SKU-001', 'Product A v2', TRUE);
-- Success! Old product is inactive, uniqueness only enforced on active

-- Can have multiple inactive products with same SKU
INSERT INTO products (sku, name, is_active) VALUES
    ('SKU-999', 'Old Product', FALSE);
INSERT INTO products (sku, name, is_active) VALUES
    ('SKU-999', 'Another Old', FALSE);
-- Both succeed! Not active, so not in unique index
```

#### **Unique Email for Non-Deleted Users:**

```sql
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    username VARCHAR(100),
    email VARCHAR(100),
    deleted_at TIMESTAMP
);

-- Email unique only for active (non-deleted) users
CREATE UNIQUE INDEX idx_users_email_active ON users(email)
WHERE deleted_at IS NULL;

-- Add users
INSERT INTO users (username, email) VALUES
    ('john', 'john@example.com');

-- Duplicate email fails (both active)
INSERT INTO users (username, email) VALUES
    ('john2', 'john@example.com');
-- ERROR: duplicate key value

-- Soft-delete first user
UPDATE users SET deleted_at = CURRENT_TIMESTAMP WHERE username = 'john';

-- Now can reuse email (works!)
INSERT INTO users (username, email) VALUES
    ('john_new', 'john@example.com');
-- Success! Old user deleted, uniqueness only on active users
```

---

### **6. Performance Benefits:**

#### **Fast Duplicate Detection:**

```sql
-- Without unique index
CREATE TABLE test_no_index (
    id SERIAL PRIMARY KEY,
    value VARCHAR(50)
);

-- Insert 1 million rows
INSERT INTO test_no_index (value)
SELECT 'VALUE-' || i
FROM generate_series(1, 1000000) AS i;

-- Check for duplicate (slow)
SELECT value, COUNT(*)
FROM test_no_index
GROUP BY value
HAVING COUNT(*) > 1;
-- Time: 1500 ms (full table scan + grouping)

-- With unique index
CREATE TABLE test_with_index (
    id SERIAL PRIMARY KEY,
    value VARCHAR(50)
);

CREATE UNIQUE INDEX idx_test_value ON test_with_index(value);

-- Insert 1 million rows
INSERT INTO test_with_index (value)
SELECT 'VALUE-' || i
FROM generate_series(1, 1000000) AS i;

-- Try to insert duplicate (instant detection)
INSERT INTO test_with_index (value) VALUES ('VALUE-500000');
-- ERROR: duplicate key value
-- Detection Time: 0.05 ms (instant index lookup)
-- 30,000x faster than full table scan!
```

#### **Fast Lookup on Unique Values:**

```sql
-- Unique index provides fast lookups
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    email VARCHAR(100)
);

CREATE UNIQUE INDEX idx_users_email ON users(email);

-- Insert 10 million users
INSERT INTO users (email)
SELECT 'user' || i || '@example.com'
FROM generate_series(1, 10000000) AS i;

-- Lookup by email (fast)
EXPLAIN ANALYZE
SELECT * FROM users WHERE email = 'user5000000@example.com';

/*
Index Scan using idx_users_email
  Index Cond: (email = 'user5000000@example.com')
Execution Time: 0.035 ms  ← Very fast!

Unique index: Optimizer knows max 1 row can match
Stops after finding first match (even faster than regular index)
*/
```

---

### **7. Unique Index vs CHECK Constraint:**

```sql
-- CHECK constraint: Application-level validation (slower)
CREATE TABLE products_check (
    product_id SERIAL PRIMARY KEY,
    sku VARCHAR(50),
    CONSTRAINT check_unique_sku CHECK (
        sku NOT IN (SELECT sku FROM products_check WHERE sku IS NOT NULL)
    )
);
-- Problem: Subquery runs on every INSERT (very slow)

-- Unique index: Database-level enforcement (fast)
CREATE TABLE products_unique (
    product_id SERIAL PRIMARY KEY,
    sku VARCHAR(50)
);

CREATE UNIQUE INDEX idx_sku ON products_unique(sku);
-- Fast: Index lookup, no subquery needed
```

---

### **8. Deferrable Unique Constraints:**

#### **Immediate vs Deferred Checking:**

```sql
-- Immediate unique constraint (default)
CREATE TABLE accounts (
    account_id SERIAL PRIMARY KEY,
    account_number INTEGER UNIQUE
);

-- Swap account numbers (fails)
BEGIN;
UPDATE accounts SET account_number = 2000 WHERE account_number = 1000;  -- OK
UPDATE accounts SET account_number = 1000 WHERE account_number = 2000;  -- ERROR!
-- Second update fails: 2000 already exists (from first update)
ROLLBACK;

-- Deferrable unique constraint
CREATE TABLE accounts_deferred (
    account_id SERIAL PRIMARY KEY,
    account_number INTEGER,
    CONSTRAINT unique_account_number UNIQUE (account_number) DEFERRABLE INITIALLY DEFERRED
);

-- Swap account numbers (works!)
BEGIN;
UPDATE accounts_deferred SET account_number = 2000 WHERE account_number = 1000;
UPDATE accounts_deferred SET account_number = 1000 WHERE account_number = 2000;
COMMIT;  -- Uniqueness checked at commit time (no violation)
-- Success!
```

---

### **9. Monitoring and Maintenance:**

```sql
-- Find all unique indexes
SELECT 
    schemaname,
    tablename,
    indexname,
    pg_get_indexdef(indexrelid) AS definition
FROM pg_stat_user_indexes
WHERE indexrelid IN (
    SELECT indexrelid 
    FROM pg_index 
    WHERE indisunique = TRUE
)
ORDER BY tablename, indexname;

-- Check index size
SELECT 
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
WHERE indexrelid IN (
    SELECT indexrelid 
    FROM pg_index 
    WHERE indisunique = TRUE
)
ORDER BY pg_relation_size(indexrelid) DESC;

-- Rebuild unique index
REINDEX INDEX idx_users_email;
```

---

### **10. Best Practices:**

```sql
-- ✅ DO: Use UNIQUE constraint for table-level uniqueness
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    email VARCHAR(100) UNIQUE  -- Preferred: Clear intent
);

-- ✅ DO: Use unique index for expression-based uniqueness
CREATE UNIQUE INDEX idx_users_email_lower ON users(LOWER(email));

-- ✅ DO: Use partial unique index for conditional uniqueness
CREATE UNIQUE INDEX idx_active_sku ON products(sku) WHERE is_active = TRUE;

-- ✅ DO: Combine with NOT NULL when appropriate
CREATE TABLE strict_users (
    user_id SERIAL PRIMARY KEY,
    email VARCHAR(100) UNIQUE NOT NULL  -- No NULLs, no duplicates
);

-- ✅ DO: Use multi-column unique for composite keys
CREATE UNIQUE INDEX idx_bookings ON bookings(flight_number, seat_number);

-- ❌ DON'T: Rely on application-level uniqueness checks
-- Always enforce uniqueness at database level (race conditions!)

-- ❌ DON'T: Create redundant unique indexes
CREATE TABLE redundant (
    id SERIAL PRIMARY KEY,
    email VARCHAR(100) UNIQUE
);
-- Don't add: CREATE UNIQUE INDEX idx_email ON redundant(email);
-- Already have unique constraint!

-- 💡 TIP: Unique indexes also speed up lookups
-- They serve dual purpose: enforce uniqueness + fast searches
```

---

### **11. Common Patterns:**

#### **Username + Email Unique:**

```sql
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    username VARCHAR(100) UNIQUE,
    email VARCHAR(100) UNIQUE,
    created_at TIMESTAMP
);
-- Both username and email must be unique independently
```

#### **Slug/URL Unique:**

```sql
CREATE TABLE articles (
    article_id SERIAL PRIMARY KEY,
    title VARCHAR(200),
    slug VARCHAR(200) UNIQUE,  -- URL-friendly unique identifier
    content TEXT
);
-- Each article has unique slug for SEO-friendly URLs
```

#### **Natural Key Unique:**

```sql
CREATE TABLE countries (
    country_id SERIAL PRIMARY KEY,
    iso_code CHAR(2) UNIQUE,  -- ISO 3166-1 alpha-2 code
    country_name VARCHAR(100)
);
-- ISO code must be unique (natural key)
```

---

### **Interview Tips:**
- Define: Unique index ensures all indexed values are distinct
- Automatic creation: PRIMARY KEY and UNIQUE constraints create unique indexes
- Uniqueness enforcement: Database-level validation (instant duplicate detection)
- NULLs allowed: Multiple NULLs permitted by default (NULL ≠ NULL)
- Performance: 30,000x faster duplicate detection vs full table scan
- Dual purpose: Enforce uniqueness + fast lookups
- Partial unique: Conditional uniqueness (active records only)
- Multi-column: Composite uniqueness (flight + seat)
- Expression-based: Case-insensitive email uniqueness
- Best practice: Use UNIQUE constraint for clarity, unique index for complex cases

</details>

<details>
<summary>36. How do you create a multi-column index?</summary>

### Answer:

A **multi-column index** (also called a **composite index** or **concatenated index**) indexes multiple columns together, enabling efficient queries that filter or sort by those columns. The order of columns in the index is crucial for performance.

---

### **Multi-Column Index Basics:**

```sql
-- Single-column indexes (separate)
CREATE INDEX idx_orders_customer ON orders(customer_id);
CREATE INDEX idx_orders_date ON orders(order_date);

-- Multi-column index (composite)
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);
-- Indexes customer_id + order_date together
-- More efficient for queries using both columns
```

**Key Concept:**
- Column order matters: `(customer_id, order_date)` ≠ `(order_date, customer_id)`
- Index can be used for **leftmost columns** (prefix)
- Most selective column should typically go first

---

### **1. Creating Multi-Column Indexes:**

#### **Basic Syntax:**

```sql
-- Syntax: CREATE INDEX index_name ON table_name(column1, column2, column3, ...);

-- Example: E-commerce orders
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER,
    order_date DATE,
    status VARCHAR(20),
    total DECIMAL(10,2)
);

-- Multi-column index on customer + date
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);

-- Multi-column index on customer + status + date
CREATE INDEX idx_orders_customer_status_date ON orders(customer_id, status, order_date);

-- Multi-column index with DESC ordering
CREATE INDEX idx_orders_customer_date_desc ON orders(customer_id, order_date DESC);
```

---

### **2. Column Order Rules:**

#### **Leftmost Prefix Rule:**

```sql
-- Create multi-column index
CREATE INDEX idx_orders_cust_status_date ON orders(customer_id, status, order_date);

-- ✅ Can use index (leftmost column)
SELECT * FROM orders WHERE customer_id = 123;
-- Uses index efficiently

-- ✅ Can use index (leftmost 2 columns)
SELECT * FROM orders WHERE customer_id = 123 AND status = 'pending';
-- Uses index efficiently

-- ✅ Can use index (all 3 columns)
SELECT * FROM orders WHERE customer_id = 123 AND status = 'pending' AND order_date > '2024-01-01';
-- Uses index efficiently

-- ⚠️ Limited use (skips first column)
SELECT * FROM orders WHERE status = 'pending' AND order_date > '2024-01-01';
-- May not use index efficiently (doesn't start with customer_id)

-- ❌ Cannot use index effectively (skips leftmost columns)
SELECT * FROM orders WHERE order_date > '2024-01-01';
-- Likely won't use index (doesn't include customer_id or status)

-- Visualization:
-- Index: (customer_id, status, order_date)
-- Think of it like a phonebook: sorted by last name, then first name
-- Can find "Smith, John" but not "John" without last name
```

---

### **3. Choosing Column Order:**

#### **Most Selective Column First:**

```sql
-- Scenario: Employee database
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    department VARCHAR(50),      -- 10 departments (low selectivity)
    job_title VARCHAR(100),      -- 50 job titles (medium selectivity)
    email VARCHAR(100),          -- 10,000 unique emails (high selectivity)
    salary INTEGER
);

-- ❌ Bad: Low selectivity first
CREATE INDEX idx_emp_bad ON employees(department, job_title, email);
-- department has only 10 values, not very selective

-- ✅ Good: High selectivity first
CREATE INDEX idx_emp_good ON employees(email, job_title, department);
-- email is unique, very selective

-- ✅ Best: Match query patterns
-- If most queries are: WHERE department = X AND job_title = Y
CREATE INDEX idx_emp_dept_title ON employees(department, job_title);
-- Order matches query pattern
```

#### **Most Frequent Column First:**

```sql
-- Consider query frequency

-- Common query pattern (90% of queries)
SELECT * FROM orders WHERE customer_id = X AND order_date > Y;

-- Less common query (10% of queries)
SELECT * FROM orders WHERE order_date > Y;

-- Best index: customer_id first (matches 90% of queries)
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);

-- If you also need date-only queries, create separate index
CREATE INDEX idx_orders_date ON orders(order_date);
```

---

### **4. Practical Examples:**

#### **Example 1: Customer Orders with Date Range:**

```sql
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER,
    order_date TIMESTAMP,
    status VARCHAR(20),
    total DECIMAL(10,2)
);

-- Multi-column index
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);

-- Query 1: Orders for specific customer (uses index)
SELECT * FROM orders WHERE customer_id = 123;
-- Index Scan on idx_orders_customer_date
-- Uses leftmost column

-- Query 2: Recent orders for customer (uses index)
SELECT * FROM orders 
WHERE customer_id = 123 
  AND order_date > CURRENT_DATE - INTERVAL '30 days';
-- Index Scan on idx_orders_customer_date
-- Uses both columns efficiently

-- Query 3: Customer orders sorted by date (uses index)
SELECT * FROM orders 
WHERE customer_id = 123 
ORDER BY order_date DESC 
LIMIT 10;
-- Index Scan on idx_orders_customer_date
-- Index provides both filtering and sorting (no extra sort needed!)
```

#### **Example 2: Multi-Tenant Application:**

```sql
CREATE TABLE documents (
    document_id SERIAL PRIMARY KEY,
    tenant_id INTEGER,          -- Which customer/organization
    category VARCHAR(50),
    created_at TIMESTAMP,
    title VARCHAR(200)
);

-- Multi-column index for tenant queries
CREATE INDEX idx_docs_tenant_category_date ON documents(tenant_id, category, created_at);

-- Query: Recent documents in category for tenant
SELECT * FROM documents
WHERE tenant_id = 42
  AND category = 'invoices'
  AND created_at > CURRENT_DATE - INTERVAL '90 days'
ORDER BY created_at DESC;

/*
EXPLAIN ANALYZE output:
Index Scan using idx_docs_tenant_category_date
  Index Cond: (tenant_id = 42 AND category = 'invoices' AND created_at > ...)
  
All filtering and sorting from index (very efficient!)
*/
```

#### **Example 3: Event Logging:**

```sql
CREATE TABLE application_logs (
    log_id BIGSERIAL PRIMARY KEY,
    application_name VARCHAR(50),
    severity VARCHAR(10),  -- DEBUG, INFO, WARN, ERROR
    timestamp TIMESTAMP,
    message TEXT
);

-- Multi-column index for log queries
CREATE INDEX idx_logs_app_severity_time ON application_logs(application_name, severity, timestamp DESC);

-- Query: Recent errors for specific app
SELECT * FROM application_logs
WHERE application_name = 'web-api'
  AND severity = 'ERROR'
  AND timestamp > CURRENT_TIMESTAMP - INTERVAL '1 hour'
ORDER BY timestamp DESC
LIMIT 100;

-- Uses index for all operations:
-- - Filter by app (leftmost column)
-- - Filter by severity (2nd column)
-- - Filter by timestamp (3rd column)
-- - Sort by timestamp (DESC matches index order)
```

---

### **5. Sorting with Multi-Column Indexes:**

#### **Index Order Matches Query Order:**

```sql
-- Index with specific column order
CREATE INDEX idx_orders_cust_date_desc ON orders(customer_id, order_date DESC);

-- ✅ Query matches index order (fast)
SELECT * FROM orders 
WHERE customer_id = 123 
ORDER BY order_date DESC;
-- Index provides sorted order (no sort operation needed)

-- ❌ Query sorts opposite direction (slower)
SELECT * FROM orders 
WHERE customer_id = 123 
ORDER BY order_date ASC;
-- Must reverse index scan or sort separately

-- Solution: Create index matching most common sort order
-- Or PostgreSQL can scan index backwards (still efficient)
```

#### **Multi-Column Sorting:**

```sql
-- Index for complex sorting
CREATE INDEX idx_orders_status_date_total ON orders(status, order_date DESC, total DESC);

-- Query matches index order
SELECT * FROM orders
WHERE status = 'pending'
ORDER BY order_date DESC, total DESC
LIMIT 20;
-- Index provides filtering and sorting (perfect match!)
```

---

### **6. Include Columns (PostgreSQL 11+):**

#### **INCLUDE Clause:**

```sql
-- Traditional multi-column index (all columns in tree)
CREATE INDEX idx_orders_traditional ON orders(customer_id, order_date, status, total);
-- All columns stored in B-tree structure (larger index)

-- Index with INCLUDE (only key columns in tree)
CREATE INDEX idx_orders_include ON orders(customer_id, order_date) 
INCLUDE (status, total);
-- Only customer_id and order_date in B-tree
-- status and total stored in leaf nodes (smaller index, faster searches)

-- Benefits:
-- ✅ Smaller index (faster lookups)
-- ✅ Supports index-only scans
-- ✅ Non-key columns don't affect sort order

-- Query covered by index (index-only scan)
SELECT customer_id, order_date, status, total
FROM orders
WHERE customer_id = 123 AND order_date > '2024-01-01';

/*
Index Only Scan using idx_orders_include
  All data retrieved from index (no table access needed)
  Heap Fetches: 0  ← Perfect!
*/
```

---

### **7. Multi-Column Index Types:**

#### **B-tree (Default):**

```sql
-- B-tree multi-column index (most common)
CREATE INDEX idx_orders_btree ON orders(customer_id, order_date);
-- Perfect for equality, ranges, sorting
```

#### **GIN (for arrays/JSONB):**

```sql
-- GIN multi-column index
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    tags TEXT[],
    attributes JSONB
);

CREATE INDEX idx_products_gin ON products USING gin(tags, attributes);

-- Search either or both columns
SELECT * FROM products WHERE tags @> ARRAY['electronics'];
SELECT * FROM products WHERE attributes @> '{"brand": "Sony"}';
SELECT * FROM products WHERE tags @> ARRAY['electronics'] AND attributes @> '{"brand": "Sony"}';
-- All queries can use the same multi-column GIN index
```

#### **GiST (for geometric/range):**

```sql
-- GiST multi-column index
CREATE TABLE events (
    event_id SERIAL PRIMARY KEY,
    location POINT,
    time_range TSRANGE
);

CREATE INDEX idx_events_gist ON events USING gist(location, time_range);

-- Spatial and temporal queries
SELECT * FROM events 
WHERE location <-> point '(40.7128, -74.0060)' < 10
  AND time_range && '[2024-01-01, 2024-01-31)'::TSRANGE;
```

---

### **8. Performance Comparison:**

```sql
-- Setup test table
CREATE TABLE test_orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER,
    order_date DATE,
    status VARCHAR(20)
);

-- Insert 10 million rows
INSERT INTO test_orders (customer_id, order_date, status)
SELECT 
    (random() * 10000)::INTEGER + 1,
    CURRENT_DATE - (random() * 365)::INTEGER,
    CASE (random() * 3)::INTEGER
        WHEN 0 THEN 'pending'
        WHEN 1 THEN 'shipped'
        ELSE 'delivered'
    END
FROM generate_series(1, 10000000);

-- Test 1: No index
EXPLAIN ANALYZE
SELECT * FROM test_orders 
WHERE customer_id = 5000 AND order_date > '2024-01-01';
-- Seq Scan: 3500 ms

-- Test 2: Two separate indexes
CREATE INDEX idx_customer ON test_orders(customer_id);
CREATE INDEX idx_date ON test_orders(order_date);

EXPLAIN ANALYZE
SELECT * FROM test_orders 
WHERE customer_id = 5000 AND order_date > '2024-01-01';
-- Bitmap Index Scan (combines both indexes): 180 ms

-- Test 3: Multi-column index
DROP INDEX idx_customer;
DROP INDEX idx_date;
CREATE INDEX idx_customer_date ON test_orders(customer_id, order_date);

EXPLAIN ANALYZE
SELECT * FROM test_orders 
WHERE customer_id = 5000 AND order_date > '2024-01-01';
-- Index Scan using idx_customer_date: 25 ms
-- 7x faster than separate indexes!
-- 140x faster than no indexes!
```

---

### **9. When to Use Multi-Column vs Separate Indexes:**

```sql
-- ✅ Use multi-column index when:

-- 1. Columns frequently queried together
WHERE customer_id = X AND order_date > Y  -- Perfect for multi-column

-- 2. Need to sort by multiple columns
ORDER BY customer_id, order_date DESC

-- 3. Want index-only scans (with INCLUDE)
SELECT customer_id, order_date, total  -- All in index

-- ❌ Use separate indexes when:

-- 1. Columns queried independently
WHERE customer_id = X  (50% of queries)
WHERE order_date > Y   (50% of queries)
-- Better: Two separate indexes

-- 2. OR conditions
WHERE customer_id = X OR order_date > Y
-- Separate indexes can be combined with Bitmap OR

-- 3. Different query patterns
-- Some queries use customer_id only
-- Other queries use order_date only
-- Create separate indexes for each
```

---

### **10. Monitoring and Optimization:**

```sql
-- Check multi-column index usage
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan AS times_used,
    pg_get_indexdef(indexrelid) AS definition,
    pg_size_pretty(pg_relation_size(indexrelid)) AS size
FROM pg_stat_user_indexes
WHERE pg_get_indexdef(indexrelid) LIKE '%,%'  -- Multi-column indexes have comma
ORDER BY idx_scan DESC;

-- Analyze query to see if index is used
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM orders 
WHERE customer_id = 123 AND order_date > '2024-01-01';

-- Check index bloat
SELECT 
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS size,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch
FROM pg_stat_user_indexes
WHERE tablename = 'orders';
```

---

### **11. Best Practices:**

```sql
-- ✅ DO: Order columns by query frequency
-- Most queried column first
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);

-- ✅ DO: Match index order to sort order
CREATE INDEX idx_orders_cust_date_desc ON orders(customer_id, order_date DESC);

-- ✅ DO: Use INCLUDE for covering indexes
CREATE INDEX idx_orders_cover ON orders(customer_id, order_date) 
INCLUDE (status, total);

-- ✅ DO: Keep index size reasonable
-- Don't add too many columns (diminishing returns)
CREATE INDEX idx_reasonable ON orders(col1, col2, col3);  -- OK
-- CREATE INDEX idx_too_many ON orders(col1, col2, col3, col4, col5, col6);  -- Too many!

-- ✅ DO: Monitor and remove unused indexes
-- If idx_scan = 0, consider dropping

-- ❌ DON'T: Create redundant indexes
CREATE INDEX idx_customer_date ON orders(customer_id, order_date);
-- DON'T ALSO CREATE:
-- CREATE INDEX idx_customer ON orders(customer_id);  -- Redundant!
-- Multi-column index can handle customer_id queries

-- ❌ DON'T: Ignore column order
-- Wrong order can make index useless for your queries

-- 💡 TIP: Analyze actual query patterns
-- Use pg_stat_statements to see what queries run most often
-- Design indexes to match real usage
```

---

### **Interview Tips:**
- Define: Multi-column index indexes multiple columns together
- Column order matters: Leftmost prefix rule (can use left columns only)
- Most selective first: Usually put highly selective column first
- Query matching: Order columns to match most common query patterns
- Sorting benefit: Index provides both filtering and sorting
- INCLUDE clause: Add non-key columns for index-only scans (PG 11+)
- Performance: 7x faster than separate indexes, 140x faster than no index
- Examples: customer_id + order_date, tenant_id + category + date
- Best practice: Match query patterns, monitor usage, avoid redundancy
- Tradeoff: Larger index size vs fewer indexes overall

</details>

<details>
<summary>37. What is index bloat and how do you handle it?</summary>

### Answer:

**Index bloat** occurs when an index contains significant amounts of dead space from deleted or updated rows, making the index larger and slower than necessary. This happens because PostgreSQL's MVCC (Multi-Version Concurrency Control) creates dead tuples that aren't immediately removed from indexes.

---

### **Understanding Index Bloat:**

```sql
-- How index bloat happens:

-- 1. Initial state: Table with 1000 rows, index is compact
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    email VARCHAR(100),
    status VARCHAR(20)
);

CREATE INDEX idx_users_email ON users(email);
-- Index size: 100 KB

-- 2. Heavy UPDATE activity
UPDATE users SET status = 'active' WHERE status = 'pending';
-- PostgreSQL MVCC creates new row versions
-- Old index entries remain (marked as dead)
-- Index size: 150 KB (50% bloat!)

-- 3. Heavy DELETE activity
DELETE FROM users WHERE status = 'deleted';
-- Rows deleted but index entries remain (dead tuples)
-- Index size: 180 KB (80% bloat!)

-- 4. Over time
-- Multiple updates/deletes create more dead space
-- Index size: 500 KB (400% bloat!)
-- Performance degraded (more pages to scan)
```

---

### **1. Detecting Index Bloat:**

#### **Method 1: pgstattuple Extension:**

```sql
-- Enable pgstattuple extension
CREATE EXTENSION IF NOT EXISTS pgstattuple;

-- Check specific index bloat
SELECT 
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
    pgstatindex(indexrelid).*
FROM pg_stat_user_indexes
WHERE indexname = 'idx_users_email';

/*
Result:
indexname         | index_size | version | tree_level | leaf_pages | dead_leaf_pages | avg_leaf_density
------------------|------------|---------|------------|------------|-----------------|------------------
idx_users_email   | 150 MB     | 4       | 3          | 18000      | 4500            | 65.5

dead_leaf_pages = 4500 out of 18000 = 25% bloat!
avg_leaf_density = 65.5% (should be 90%+)
*/

-- Check all indexes
SELECT 
    schemaname,
    tablename,
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
    ROUND((pgstatindex(indexrelid)).avg_leaf_density) AS leaf_density,
    pg_size_pretty(
        pg_relation_size(indexrelid) * 
        (100 - (pgstatindex(indexrelid)).avg_leaf_density) / 100
    ) AS estimated_bloat
FROM pg_stat_user_indexes
WHERE schemaname = 'public'
ORDER BY pg_relation_size(indexrelid) DESC;
```

#### **Method 2: Manual Calculation Query:**

```sql
-- Estimate index bloat percentage
SELECT 
    schemaname,
    tablename,
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
    CASE 
        WHEN idx_tup_read = 0 THEN 0
        ELSE ROUND((idx_tup_read - idx_tup_fetch)::NUMERIC / idx_tup_read * 100, 2)
    END AS potential_bloat_pct,
    idx_scan AS times_used
FROM pg_stat_user_indexes
WHERE idx_scan > 0
ORDER BY pg_relation_size(indexrelid) DESC;
```

#### **Method 3: Dead Tuples Query:**

```sql
-- Check dead tuples in table (affects indexes)
SELECT 
    schemaname,
    relname AS table_name,
    n_live_tup AS live_rows,
    n_dead_tup AS dead_rows,
    ROUND(n_dead_tup::NUMERIC / NULLIF(n_live_tup, 0) * 100, 2) AS dead_row_pct,
    last_vacuum,
    last_autovacuum
FROM pg_stat_user_tables
WHERE n_dead_tup > 0
ORDER BY n_dead_tup DESC;

/*
Result:
table_name | live_rows | dead_rows | dead_row_pct | last_vacuum | last_autovacuum
-----------|-----------|-----------|--------------|-------------|------------------
orders     | 1000000   | 250000    | 25.00        | NULL        | 2024-12-20
users      | 500000    | 150000    | 30.00        | NULL        | 2024-12-21

High dead_row_pct indicates bloat!
*/
```

---

### **2. Causes of Index Bloat:**

#### **Frequent UPDATEs:**

```sql
-- Scenario: User session tracking
CREATE TABLE sessions (
    session_id SERIAL PRIMARY KEY,
    user_id INTEGER,
    last_activity TIMESTAMP,
    page_views INTEGER
);

CREATE INDEX idx_sessions_user ON sessions(user_id);

-- Initial insert: 1 million sessions
INSERT INTO sessions (user_id, last_activity, page_views)
SELECT 
    (random() * 100000)::INTEGER,
    CURRENT_TIMESTAMP,
    1
FROM generate_series(1, 1000000);

-- Index size: 21 MB

-- Frequent updates (every page view)
UPDATE sessions SET 
    last_activity = CURRENT_TIMESTAMP,
    page_views = page_views + 1
WHERE session_id = 12345;

-- After 1 million updates:
-- Index size: 85 MB (4x bloat!)
-- Each UPDATE creates new index entry
-- Old entries marked as dead but not removed
```

#### **Heavy DELETEs:**

```sql
-- Scenario: Old data cleanup
CREATE TABLE log_entries (
    log_id BIGSERIAL PRIMARY KEY,
    timestamp TIMESTAMP,
    level VARCHAR(10),
    message TEXT
);

CREATE INDEX idx_logs_timestamp ON log_entries(timestamp);

-- Initial: 10 million log entries
-- Index size: 214 MB

-- Delete old logs
DELETE FROM log_entries WHERE timestamp < CURRENT_DATE - INTERVAL '90 days';
-- Deleted 8 million rows (80% of data)

-- Index size: 210 MB (still large!)
-- Dead entries not reclaimed
-- Index bloated with deleted entries
```

#### **HOT Updates Not Working:**

```sql
-- HOT = Heap Only Tuple (updates without index update)

-- Bad: UPDATE indexed column (creates index entry)
UPDATE users SET email = 'newemail@example.com' WHERE user_id = 123;
-- Index must be updated (causes bloat over time)

-- Good: UPDATE non-indexed column (HOT update)
UPDATE users SET last_login = CURRENT_TIMESTAMP WHERE user_id = 123;
-- Table updated, index unchanged (no bloat)

-- Check HOT updates
SELECT 
    schemaname,
    relname,
    n_tup_upd AS total_updates,
    n_tup_hot_upd AS hot_updates,
    ROUND(n_tup_hot_upd::NUMERIC / NULLIF(n_tup_upd, 0) * 100, 2) AS hot_update_pct
FROM pg_stat_user_tables
WHERE n_tup_upd > 0
ORDER BY hot_update_pct ASC;

/*
Result:
relname | total_updates | hot_updates | hot_update_pct
--------|---------------|-------------|----------------
users   | 1000000       | 100000      | 10.00
sessions| 5000000       | 4500000     | 90.00

Low hot_update_pct = More index bloat
*/
```

---

### **3. Handling Index Bloat:**

#### **Solution 1: REINDEX (Simple, Locks Table):**

```sql
-- Rebuild single index (exclusive lock during rebuild)
REINDEX INDEX idx_users_email;
-- Blocks reads/writes to table during rebuild

-- Rebuild all indexes on table
REINDEX TABLE users;

-- Rebuild all indexes in database
REINDEX DATABASE mydb;

-- Check size before and after
SELECT 
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS size_before
FROM pg_stat_user_indexes
WHERE indexname = 'idx_users_email';
-- Before: 150 MB

REINDEX INDEX idx_users_email;

SELECT 
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS size_after
FROM pg_stat_user_indexes
WHERE indexname = 'idx_users_email';
-- After: 85 MB (43% reduction!)
```

#### **Solution 2: REINDEX CONCURRENTLY (No Locks, PostgreSQL 12+):**

```sql
-- Rebuild index without blocking reads/writes
REINDEX INDEX CONCURRENTLY idx_users_email;
-- Table remains fully accessible during rebuild
-- Takes longer but no downtime

-- Limitation: Cannot use inside transaction
BEGIN;
REINDEX INDEX CONCURRENTLY idx_users_email;  -- ERROR!
ROLLBACK;

-- Must run outside transaction
REINDEX INDEX CONCURRENTLY idx_users_email;  -- OK

-- Check progress
SELECT 
    pid,
    query,
    state,
    wait_event_type,
    wait_event
FROM pg_stat_activity
WHERE query LIKE '%REINDEX%';
```

#### **Solution 3: CREATE INDEX CONCURRENTLY + DROP (Manual):**

```sql
-- Alternative to REINDEX (more control)

-- Step 1: Create new index concurrently
CREATE INDEX CONCURRENTLY idx_users_email_new ON users(email);
-- Table accessible during creation

-- Step 2: Drop old bloated index
DROP INDEX idx_users_email;

-- Step 3: Rename new index
ALTER INDEX idx_users_email_new RENAME TO idx_users_email;

-- Benefit: Full control, can monitor progress
-- Drawback: More manual steps
```

#### **Solution 4: VACUUM FULL (Extreme, Locks Table):**

```sql
-- VACUUM FULL rebuilds entire table and all indexes
VACUUM FULL users;
-- Locks table exclusively (no reads/writes)
-- Reclaims all dead space
-- Very slow for large tables

-- Only use when:
-- - Severe bloat (>50%)
-- - Can afford downtime
-- - Need to reclaim disk space

-- Check size before/after
SELECT pg_size_pretty(pg_total_relation_size('users'));
-- Before: 500 MB

VACUUM FULL users;

SELECT pg_size_pretty(pg_total_relation_size('users'));
-- After: 280 MB (44% reduction)
```

#### **Solution 5: Regular VACUUM (Preventive):**

```sql
-- Regular VACUUM marks dead tuples for reuse (doesn't shrink)
VACUUM users;
-- No locks, safe to run anytime
-- Prevents bloat from getting worse

-- Aggressive VACUUM
VACUUM (ANALYZE, VERBOSE) users;

-- Check autovacuum settings
SELECT 
    relname,
    reloptions
FROM pg_class
WHERE relname = 'users';

-- Tune autovacuum for high-churn tables
ALTER TABLE users SET (
    autovacuum_vacuum_scale_factor = 0.05,  -- Vacuum when 5% dead (default 20%)
    autovacuum_vacuum_threshold = 1000      -- Minimum dead tuples
);

-- Check last vacuum
SELECT 
    schemaname,
    relname,
    last_vacuum,
    last_autovacuum,
    n_dead_tup
FROM pg_stat_user_tables
WHERE relname = 'users';
```

---

### **4. Preventing Index Bloat:**

#### **Strategy 1: Optimize UPDATEs (HOT Updates):**

```sql
-- Minimize indexed column updates

-- Bad: Update indexed email frequently
UPDATE users SET 
    email = new_email,
    last_login = CURRENT_TIMESTAMP
WHERE user_id = 123;
-- Updates index (causes bloat)

-- Good: Separate frequently updated data
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    email VARCHAR(100) UNIQUE,  -- Rarely updated
    username VARCHAR(100)
);

CREATE TABLE user_sessions (
    user_id INTEGER REFERENCES users(user_id),
    last_login TIMESTAMP,       -- Frequently updated
    login_count INTEGER
);

-- Now frequent updates don't bloat email index
UPDATE user_sessions SET 
    last_login = CURRENT_TIMESTAMP,
    login_count = login_count + 1
WHERE user_id = 123;
```

#### **Strategy 2: Tune Autovacuum:**

```sql
-- Global autovacuum settings (postgresql.conf)
-- autovacuum = on
-- autovacuum_vacuum_scale_factor = 0.1  (10% dead tuples)
-- autovacuum_vacuum_threshold = 50

-- Per-table autovacuum for high-churn tables
ALTER TABLE high_traffic_table SET (
    autovacuum_enabled = true,
    autovacuum_vacuum_scale_factor = 0.02,  -- Vacuum at 2% dead
    autovacuum_analyze_scale_factor = 0.01,  -- Analyze at 1% changes
    autovacuum_vacuum_cost_delay = 10,       -- Faster vacuum
    autovacuum_vacuum_cost_limit = 1000      -- More aggressive
);

-- Monitor autovacuum activity
SELECT 
    schemaname,
    relname,
    last_autovacuum,
    autovacuum_count,
    n_dead_tup,
    n_live_tup,
    ROUND(n_dead_tup::NUMERIC / NULLIF(n_live_tup, 0) * 100, 2) AS bloat_pct
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC;
```

#### **Strategy 3: Fillfactor Tuning:**

```sql
-- Fillfactor: Percentage of page to fill (default 90%)
-- Lower fillfactor = More room for HOT updates

-- For frequently updated tables
CREATE TABLE frequently_updated (
    id SERIAL PRIMARY KEY,
    data VARCHAR(100),
    updated_at TIMESTAMP
) WITH (fillfactor = 70);  -- Leave 30% free space

CREATE INDEX idx_data ON frequently_updated(data) WITH (fillfactor = 70);

-- Benefit: More HOT updates possible
-- Tradeoff: Larger initial size
-- Use when: Many updates on same rows
```

#### **Strategy 4: Partitioning:**

```sql
-- Partition large tables to limit bloat scope
CREATE TABLE log_entries (
    log_id BIGSERIAL,
    timestamp TIMESTAMP,
    message TEXT
) PARTITION BY RANGE (timestamp);

-- Monthly partitions
CREATE TABLE log_entries_2024_01 PARTITION OF log_entries
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE log_entries_2024_02 PARTITION OF log_entries
    FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');

-- Indexes per partition (smaller, easier to rebuild)
CREATE INDEX idx_log_2024_01 ON log_entries_2024_01(timestamp);
CREATE INDEX idx_log_2024_02 ON log_entries_2024_02(timestamp);

-- Drop old partitions (instead of DELETE)
DROP TABLE log_entries_2023_01;  -- Instant, reclaims all space
-- vs DELETE (leaves dead tuples, causes bloat)
```

---

### **5. Monitoring and Alerting:**

#### **Bloat Monitoring Query:**

```sql
-- Complete bloat monitoring query
WITH index_stats AS (
    SELECT 
        schemaname,
        tablename,
        indexname,
        pg_relation_size(indexrelid) AS index_bytes,
        idx_scan,
        idx_tup_read,
        idx_tup_fetch
    FROM pg_stat_user_indexes
),
bloat_estimates AS (
    SELECT 
        schemaname,
        tablename,
        indexname,
        pg_size_pretty(index_bytes) AS index_size,
        idx_scan,
        CASE 
            WHEN idx_tup_read = 0 THEN 0
            ELSE ROUND((idx_tup_read - idx_tup_fetch)::NUMERIC / idx_tup_read * 100, 2)
        END AS estimated_bloat_pct
    FROM index_stats
    WHERE index_bytes > 10 * 1024 * 1024  -- Only indexes > 10 MB
)
SELECT 
    schemaname,
    tablename,
    indexname,
    index_size,
    estimated_bloat_pct,
    idx_scan,
    CASE 
        WHEN estimated_bloat_pct > 50 THEN '🔴 Critical - Rebuild urgently'
        WHEN estimated_bloat_pct > 30 THEN '🟡 Warning - Consider rebuild'
        WHEN estimated_bloat_pct > 20 THEN '🟢 Minor bloat'
        ELSE '✅ Healthy'
    END AS status
FROM bloat_estimates
WHERE estimated_bloat_pct > 20
ORDER BY estimated_bloat_pct DESC;
```

#### **Automated Maintenance Script:**

```sql
-- Weekly maintenance job
DO $$
DECLARE
    idx RECORD;
BEGIN
    -- Reindex bloated indexes
    FOR idx IN 
        SELECT indexname
        FROM pg_stat_user_indexes psi
        JOIN pg_index pi ON psi.indexrelid = pi.indexrelid
        WHERE NOT pi.indisunique  -- Don't reindex unique/primary keys during peak
          AND pg_relation_size(psi.indexrelid) > 100 * 1024 * 1024  -- > 100 MB
    LOOP
        EXECUTE format('REINDEX INDEX CONCURRENTLY %I', idx.indexname);
        RAISE NOTICE 'Reindexed: %', idx.indexname;
    END LOOP;
END $$;
```

---

### **6. Best Practices:**

```sql
-- ✅ DO: Monitor bloat regularly
-- Schedule weekly bloat checks

-- ✅ DO: Use REINDEX CONCURRENTLY for production
REINDEX INDEX CONCURRENTLY idx_users_email;
-- No downtime

-- ✅ DO: Tune autovacuum for high-churn tables
ALTER TABLE sessions SET (autovacuum_vacuum_scale_factor = 0.05);

-- ✅ DO: Minimize updates to indexed columns
-- Separate frequently updated columns

-- ✅ DO: Use partitioning for large tables
-- Easier to maintain smaller indexes

-- ✅ DO: Schedule maintenance during low-traffic periods
-- Even REINDEX CONCURRENTLY has some overhead

-- ❌ DON'T: Use VACUUM FULL on large tables in production
-- Too disruptive, exclusive lock

-- ❌ DON'T: Ignore bloat warnings
-- Bloat compounds over time

-- ❌ DON'T: Disable autovacuum
-- Prevents bloat prevention

-- 💡 TIP: Monitor autovacuum activity
SELECT * FROM pg_stat_progress_vacuum;
-- See vacuum progress in real-time
```

---

### **7. Real-World Example:**

```sql
-- Production scenario: E-commerce orders table

-- Problem: Order status updated frequently
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER,
    status VARCHAR(20),  -- Updated many times: pending → processing → shipped → delivered
    order_date TIMESTAMP,
    total DECIMAL(10,2)
);

CREATE INDEX idx_orders_status ON orders(status);

-- After 6 months: Index bloated to 500 MB (should be 150 MB)

-- Solution 1: Immediate fix (off-peak hours)
REINDEX INDEX CONCURRENTLY idx_orders_status;
-- Reduces to 150 MB

-- Solution 2: Long-term prevention
-- Separate hot columns
CREATE TABLE order_tracking (
    order_id INTEGER PRIMARY KEY REFERENCES orders(order_id),
    status VARCHAR(20),
    updated_at TIMESTAMP
);

-- Move status to tracking table
-- Now status updates don't bloat orders indexes

-- Solution 3: Tune autovacuum
ALTER TABLE orders SET (
    autovacuum_vacuum_scale_factor = 0.05,
    autovacuum_vacuum_threshold = 5000
);

-- Result: Bloat stays under 10%
```

---

### **Interview Tips:**
- Define: Index bloat = dead space from deleted/updated rows (MVCC creates dead tuples)
- Cause: Frequent UPDATEs/DELETEs, HOT updates not working, insufficient vacuuming
- Detection: pgstattuple extension, dead tuples query, manual bloat calculation
- Symptoms: Large index size, slow queries, high disk usage
- Solutions: REINDEX (locks), REINDEX CONCURRENTLY (no locks, PG 12+), regular VACUUM
- Prevention: Tune autovacuum, minimize indexed column updates, use fillfactor, partitioning
- Performance impact: 3-4x larger index, slower scans, more I/O
- Example: 150 MB index bloated to 500 MB, REINDEX reduces to 150 MB (70% savings)
- Best practice: Monitor regularly, REINDEX CONCURRENTLY during low traffic
- Real-world: Separate frequently updated columns to minimize index bloat

</details>

<details>
<summary>38. When should you avoid using indexes?</summary>

### Answer:

While indexes dramatically improve read performance, there are specific scenarios where **not creating an index** or **removing an existing index** is the better choice. Indexes have costs (storage, write overhead, maintenance) that can outweigh their benefits in certain situations.

---

### **When to Avoid Indexes:**

---

### **1. Small Tables:**

#### **Tables with Few Rows:**

```sql
-- Small lookup table (100 rows)
CREATE TABLE status_codes (
    code VARCHAR(20) PRIMARY KEY,
    description VARCHAR(200)
);

-- Only 100 rows total
INSERT INTO status_codes VALUES
    ('pending', 'Order pending'),
    ('processing', 'Order processing'),
    ('shipped', 'Order shipped'),
    ('delivered', 'Order delivered');

-- Index on description? NO!
-- Sequential scan is faster for small tables
SELECT * FROM status_codes WHERE description LIKE '%pending%';

-- Explanation:
-- Sequential scan: Read 100 rows in 1-2 disk blocks
-- Index scan: Read index pages + table pages (more overhead)
-- 
-- Rule of thumb: Tables < 1000 rows rarely benefit from secondary indexes
```

#### **Performance Comparison:**

```sql
-- Test with small table
CREATE TABLE small_test (
    id SERIAL PRIMARY KEY,
    value VARCHAR(50)
);

-- Insert 500 rows
INSERT INTO small_test (value)
SELECT 'VALUE-' || i FROM generate_series(1, 500) AS i;

-- Query without index
EXPLAIN ANALYZE
SELECT * FROM small_test WHERE value = 'VALUE-250';
/*
Seq Scan on small_test  (cost=0.00..12.50 rows=1)
  Filter: (value = 'VALUE-250')
Execution Time: 0.05 ms  ← Fast enough!
*/

-- Create index
CREATE INDEX idx_small_value ON small_test(value);

-- Query with index
EXPLAIN ANALYZE
SELECT * FROM small_test WHERE value = 'VALUE-250';
/*
Index Scan using idx_small_value  (cost=0.27..8.29 rows=1)
  Index Cond: (value = 'VALUE-250')
Execution Time: 0.04 ms  ← Barely faster (0.01ms improvement)
*/

-- Verdict: Index adds minimal benefit, wastes 24 KB storage
```

---

### **2. Low Selectivity Columns:**

#### **Few Distinct Values:**

```sql
-- Table with 1 million employees
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    gender CHAR(1),      -- Only 2 values: 'M', 'F' (low selectivity)
    is_active BOOLEAN,   -- Only 2 values: true, false (low selectivity)
    department VARCHAR(50),
    salary INTEGER
);

-- Insert 1 million rows
INSERT INTO employees (name, gender, is_active, department, salary)
SELECT 
    'Employee ' || i,
    CASE WHEN random() < 0.5 THEN 'M' ELSE 'F' END,
    random() < 0.95,
    'Dept-' || ((random() * 10)::INTEGER),
    50000 + (random() * 50000)::INTEGER
FROM generate_series(1, 1000000) AS i;

-- Bad: Index on gender (returns ~50% of rows)
CREATE INDEX idx_employees_gender ON employees(gender);

-- Query
EXPLAIN ANALYZE
SELECT * FROM employees WHERE gender = 'M';
/*
Seq Scan on employees  (cost=0.00..20000.00 rows=500000)
  Filter: (gender = 'M')
Execution Time: 250 ms

Index NOT used! PostgreSQL chose sequential scan
Why? Query returns 50% of table, seq scan is faster
*/

-- Rule: If query returns > 5-10% of table, index is rarely used
-- Gender column: Each value represents 50% of data
-- Index wastes space without providing benefit
```

#### **Boolean Columns:**

```sql
-- Bad: Index on boolean (95% true, 5% false)
CREATE INDEX idx_employees_active ON employees(is_active);

-- Query for active employees (95% of data)
SELECT * FROM employees WHERE is_active = true;
-- Sequential scan used (returns too many rows)

-- Solution: Partial index for minority case
DROP INDEX idx_employees_active;
CREATE INDEX idx_employees_inactive ON employees(employee_id) 
WHERE is_active = false;
-- Only indexes 5% of data (inactive employees)

-- Now this query is fast:
SELECT * FROM employees WHERE is_active = false;
-- Uses partial index (small, efficient)
```

---

### **3. Write-Heavy Tables:**

#### **High INSERT/UPDATE/DELETE Rate:**

```sql
-- Scenario: Real-time sensor data (millions of inserts/hour)
CREATE TABLE sensor_readings (
    reading_id BIGSERIAL PRIMARY KEY,
    sensor_id INTEGER,
    timestamp TIMESTAMP,
    temperature DECIMAL(5,2),
    humidity DECIMAL(5,2),
    pressure DECIMAL(6,2)
);

-- Bad: Create indexes on every column
CREATE INDEX idx_sensor_id ON sensor_readings(sensor_id);
CREATE INDEX idx_timestamp ON sensor_readings(timestamp);
CREATE INDEX idx_temperature ON sensor_readings(temperature);
CREATE INDEX idx_humidity ON sensor_readings(humidity);
CREATE INDEX idx_pressure ON sensor_readings(pressure);

-- Insert performance test
EXPLAIN ANALYZE
INSERT INTO sensor_readings (sensor_id, timestamp, temperature, humidity, pressure)
SELECT 
    (random() * 1000)::INTEGER,
    CURRENT_TIMESTAMP + (i * INTERVAL '1 second'),
    (random() * 40 - 10)::DECIMAL(5,2),
    (random() * 100)::DECIMAL(5,2),
    (random() * 200 + 900)::DECIMAL(6,2)
FROM generate_series(1, 10000) AS i;

/*
With 5 indexes: 2500 ms
Without indexes: 150 ms
16x slower with indexes!
*/

-- Each INSERT must update 6 indexes (PRIMARY KEY + 5 secondary)
-- Overhead: Write to index pages, maintain B-tree balance
-- Solution: Only create indexes for frequent queries

-- Keep only necessary indexes
DROP INDEX idx_temperature;
DROP INDEX idx_humidity;
DROP INDEX idx_pressure;

-- Keep only commonly queried indexes
-- idx_sensor_id (for per-sensor queries)
-- idx_timestamp (for time-range queries)
```

#### **Batch Loading:**

```sql
-- Data warehouse loading scenario
CREATE TABLE sales_facts (
    sale_id BIGSERIAL PRIMARY KEY,
    date_key INTEGER,
    product_key INTEGER,
    customer_key INTEGER,
    amount DECIMAL(10,2),
    quantity INTEGER
);

-- Strategy: Drop indexes before bulk load
DROP INDEX IF EXISTS idx_sales_date;
DROP INDEX IF EXISTS idx_sales_product;
DROP INDEX IF EXISTS idx_sales_customer;

-- Bulk load (10 million rows)
INSERT INTO sales_facts SELECT ... FROM source_data;
-- Fast: No index maintenance

-- Recreate indexes after load
CREATE INDEX idx_sales_date ON sales_facts(date_key);
CREATE INDEX idx_sales_product ON sales_facts(product_key);
CREATE INDEX idx_sales_customer ON sales_facts(customer_key);
-- More efficient to build indexes once than maintain during inserts

-- Speedup: 5-10x faster for large bulk loads
```

---

### **4. Volatile/Temporary Data:**

#### **Short-Lived Tables:**

```sql
-- Temporary processing table
CREATE TEMP TABLE processing_temp (
    id SERIAL,
    data TEXT,
    result TEXT
);

-- Bad: Create indexes on temp table
CREATE INDEX idx_temp_data ON processing_temp(data);

-- Why avoid:
-- - Table exists briefly (minutes)
-- - Data loaded, processed, dropped
-- - Index creation overhead > benefit

-- Good: Use temp table without indexes (if small)
-- Or create index only if temp table is large (> 100K rows) and queried multiple times
```

#### **Staging Tables:**

```sql
-- ETL staging table
CREATE TABLE staging_customers (
    customer_id INTEGER,
    name VARCHAR(100),
    email VARCHAR(100),
    address TEXT
);

-- Data flow:
-- 1. Load from CSV (no indexes needed)
-- 2. Transform/clean data
-- 3. Insert to production table (which has indexes)
-- 4. Truncate staging

-- Don't index staging tables (unnecessary overhead)
```

---

### **5. Columns Never Used in WHERE/JOIN/ORDER BY:**

#### **Display-Only Columns:**

```sql
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    sku VARCHAR(50),           -- Used in WHERE: needs index
    name VARCHAR(200),         -- Display only: no index
    description TEXT,          -- Display only: no index
    long_specs TEXT,           -- Display only: no index
    price DECIMAL(10,2),       -- Used in WHERE/ORDER BY: needs index
    created_at TIMESTAMP       -- Used in WHERE/ORDER BY: needs index
);

-- ✅ Index these (used in queries)
CREATE INDEX idx_products_sku ON products(sku);
CREATE INDEX idx_products_price ON products(price);
CREATE INDEX idx_products_created ON products(created_at);

-- ❌ Don't index these (only displayed)
-- name, description, long_specs
-- Never used in WHERE, JOIN, or ORDER BY clauses
```

#### **Audit Trail Columns:**

```sql
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER,
    order_date DATE,
    total DECIMAL(10,2),
    created_at TIMESTAMP,
    created_by VARCHAR(100),   -- Audit: rarely queried
    updated_at TIMESTAMP,      -- Audit: rarely queried
    updated_by VARCHAR(100),   -- Audit: rarely queried
    notes TEXT                 -- Audit: rarely queried
);

-- Index queried columns only
CREATE INDEX idx_orders_customer ON orders(customer_id);
CREATE INDEX idx_orders_date ON orders(order_date);

-- Don't index audit columns (rarely queried)
-- created_by, updated_by, notes
```

---

### **6. Large Text/Binary Columns:**

#### **TOAST Columns:**

```sql
-- Large text columns (stored in TOAST)
CREATE TABLE documents (
    document_id SERIAL PRIMARY KEY,
    title VARCHAR(200),        -- Indexable
    content TEXT,              -- Large: don't index directly
    file_data BYTEA,           -- Binary: don't index
    created_at TIMESTAMP
);

-- ❌ Bad: Index large text column
CREATE INDEX idx_documents_content ON documents(content);
-- Index will be huge, slow to build/maintain

-- ✅ Good: Full-text search index (if needed)
ALTER TABLE documents ADD COLUMN search_vector TSVECTOR;
CREATE INDEX idx_documents_search ON documents USING gin(search_vector);

-- ✅ Good: Index prefix only (if needed)
CREATE INDEX idx_documents_content_prefix ON documents(substring(content, 1, 50));
-- Or use expression index
CREATE INDEX idx_documents_content_start ON documents(left(content, 100));
```

---

### **7. Heavily Duplicated Data:**

#### **Redundant Indexes:**

```sql
-- Bad: Redundant indexes
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER,
    order_date DATE,
    status VARCHAR(20)
);

-- ❌ Redundant
CREATE INDEX idx_customer ON orders(customer_id);
CREATE INDEX idx_customer_date ON orders(customer_id, order_date);
-- idx_customer_date can handle queries on customer_id alone

-- ✅ Better: Keep only composite index
DROP INDEX idx_customer;
-- idx_customer_date covers both use cases (leftmost prefix rule)

-- Exception: Keep single-column index if:
-- - Much smaller than composite
-- - Used independently very frequently
-- - Composite index is very large
```

---

### **8. Performance Analysis:**

```sql
-- Identify unused indexes (candidates for removal)
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan AS times_used,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
    pg_get_indexdef(indexrelid) AS definition
FROM pg_stat_user_indexes
WHERE idx_scan = 0                          -- Never used
  AND indexrelid NOT IN (
      SELECT indexrelid FROM pg_index WHERE indisunique OR indisprimary
  )                                          -- Not unique/primary key
ORDER BY pg_relation_size(indexrelid) DESC;

/*
Result:
indexname                  | times_used | index_size | definition
---------------------------|------------|------------|-------------
idx_employees_gender       | 0          | 21 MB      | CREATE INDEX ...
idx_temp_processing_data   | 0          | 15 MB      | CREATE INDEX ...
idx_products_description   | 0          | 45 MB      | CREATE INDEX ...

Total wasted space: 81 MB
Recommendation: DROP these indexes
*/

-- Check index cost vs benefit
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan,
    pg_size_pretty(pg_relation_size(indexrelid)) AS size,
    CASE 
        WHEN idx_scan = 0 THEN '❌ Never used - DROP'
        WHEN idx_scan < 10 THEN '⚠️ Rarely used - Consider DROP'
        WHEN idx_scan < 100 THEN '🟡 Low usage'
        ELSE '✅ Actively used'
    END AS recommendation
FROM pg_stat_user_indexes
WHERE schemaname = 'public'
ORDER BY idx_scan ASC, pg_relation_size(indexrelid) DESC;
```

---

### **9. Decision Framework:**

```sql
-- Should I create an index? Checklist:

-- ✅ Create index if:
-- 1. Table has > 1000 rows
-- 2. Column used in WHERE, JOIN, or ORDER BY
-- 3. Column has high selectivity (many distinct values)
-- 4. Queries return < 5-10% of table rows
-- 5. Table is read-heavy (more SELECTs than UPDATEs)

-- ❌ Don't create index if:
-- 1. Table has < 1000 rows
-- 2. Column never used in queries
-- 3. Column has low selectivity (few distinct values)
-- 4. Queries return > 10% of table rows
-- 5. Table is write-heavy (frequent INSERT/UPDATE/DELETE)
-- 6. Temporary/staging table
-- 7. Large text/binary columns
-- 8. Index would be redundant

-- 🤔 Consider alternatives:
-- - Partial index (for specific conditions)
-- - Expression index (for functions on columns)
-- - GIN/GiST (for arrays/JSONB)
-- - Covering index with INCLUDE
-- - Partitioning (for very large tables)
```

---

### **10. Best Practices:**

```sql
-- ✅ DO: Regularly review index usage
SELECT * FROM pg_stat_user_indexes WHERE idx_scan = 0;

-- ✅ DO: Drop unused indexes
DROP INDEX idx_never_used;

-- ✅ DO: Use partial indexes for low-selectivity columns
CREATE INDEX idx_inactive_only ON users(user_id) WHERE is_active = false;

-- ✅ DO: Consolidate redundant indexes
-- Keep multi-column index, drop single-column if redundant

-- ✅ DO: Profile queries before creating indexes
EXPLAIN ANALYZE SELECT ...
-- Ensure index will actually be used

-- ❌ DON'T: Create indexes "just in case"
-- Every index has a cost (storage, writes, maintenance)

-- ❌ DON'T: Index every column
-- Only index columns used in queries

-- ❌ DON'T: Keep indexes on small tables
-- Sequential scan is faster

-- 💡 TIP: Monitor index size vs benefit
SELECT 
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS size,
    idx_scan AS usage_count,
    ROUND(pg_relation_size(indexrelid)::NUMERIC / NULLIF(idx_scan, 0)) AS bytes_per_use
FROM pg_stat_user_indexes
ORDER BY bytes_per_use DESC NULLS FIRST;
-- High bytes_per_use = wasteful index
```

---

### **Interview Tips:**
- Small tables: < 1000 rows, sequential scan faster than index
- Low selectivity: Boolean, gender columns (few distinct values, returns >10% rows)
- Write-heavy: Indexes slow INSERT/UPDATE/DELETE (16x slower with many indexes)
- Temporary data: Staging tables, temp tables don't need indexes
- Unused columns: Display-only, audit columns never queried
- Large columns: TEXT, BYTEA - use full-text search or prefix indexes instead
- Redundant: Multi-column index covers single-column queries (leftmost prefix)
- Detection: Query pg_stat_user_indexes for idx_scan = 0
- Rule of thumb: Create indexes only for columns in WHERE/JOIN/ORDER BY
- Cost-benefit: Every index adds storage + write overhead, ensure it provides value

</details>

<details>
<summary>39. How do you check if an index is being used?</summary>

### Answer:

You can check if an index is being used through **EXPLAIN/EXPLAIN ANALYZE**, **pg_stat_user_indexes** system view, and **pg_stat_statements** extension. These tools help identify whether PostgreSQL's query planner is utilizing your indexes and how frequently they're accessed.

---

### **1. EXPLAIN and EXPLAIN ANALYZE:**

#### **Basic EXPLAIN (Query Plan):**

```sql
-- Create test table and index
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER,
    order_date DATE,
    status VARCHAR(20),
    total DECIMAL(10,2)
);

CREATE INDEX idx_orders_customer ON orders(customer_id);
CREATE INDEX idx_orders_date ON orders(order_date);

-- Insert sample data
INSERT INTO orders (customer_id, order_date, status, total)
SELECT 
    (random() * 10000)::INTEGER + 1,
    CURRENT_DATE - (random() * 365)::INTEGER,
    'completed',
    (random() * 1000)::DECIMAL(10,2)
FROM generate_series(1, 100000);

-- Check if index is used
EXPLAIN
SELECT * FROM orders WHERE customer_id = 5000;

/*
QUERY PLAN
------------------------------------------------------------------------
Index Scan using idx_orders_customer on orders  ← Index IS being used!
  Index Cond: (customer_id = 5000)
  
Key indicators:
- "Index Scan using idx_orders_customer" = Index is used
- "Index Cond" shows the condition that triggers index usage
*/

-- Query without index usage
EXPLAIN
SELECT * FROM orders WHERE status = 'completed';

/*
QUERY PLAN
------------------------------------------------------------------------
Seq Scan on orders  ← Index NOT used (no index on status)
  Filter: ((status)::text = 'completed'::text)
  
"Seq Scan" = Sequential scan (full table scan, no index)
*/
```

#### **EXPLAIN ANALYZE (Actual Execution):**

```sql
-- EXPLAIN ANALYZE runs the query and shows actual performance
EXPLAIN ANALYZE
SELECT * FROM orders WHERE customer_id = 5000;

/*
QUERY PLAN
------------------------------------------------------------------------
Index Scan using idx_orders_customer on orders
  (cost=0.29..12.31 rows=10 width=35)
  (actual time=0.025..0.045 rows=10 loops=1)
  Index Cond: (customer_id = 5000)
Planning Time: 0.123 ms
Execution Time: 0.067 ms  ← Actual execution time

Key information:
- cost=0.29..12.31: Estimated cost range
- rows=10: Estimated rows (vs actual rows=10)
- actual time=0.025..0.045: Actual time to retrieve rows
- Index Scan confirms index usage
*/

-- Compare with sequential scan
EXPLAIN ANALYZE
SELECT * FROM orders WHERE status = 'completed';

/*
QUERY PLAN
------------------------------------------------------------------------
Seq Scan on orders
  (cost=0.00..2179.00 rows=100000 width=35)
  (actual time=0.015..35.234 rows=100000 loops=1)
  Filter: ((status)::text = 'completed'::text)
Planning Time: 0.098 ms
Execution Time: 42.567 ms  ← Much slower (no index)

Seq Scan = No index used
Execution: 42.567 ms vs 0.067 ms with index (635x slower!)
*/
```

#### **EXPLAIN Options:**

```sql
-- Detailed EXPLAIN with all options
EXPLAIN (ANALYZE, BUFFERS, VERBOSE, COSTS, TIMING)
SELECT * FROM orders WHERE customer_id = 5000;

/*
QUERY PLAN
------------------------------------------------------------------------
Index Scan using idx_orders_customer on public.orders
  Output: order_id, customer_id, order_date, status, total
  Index Cond: (orders.customer_id = 5000)
  Buffers: shared hit=4  ← Cache hits (index pages read from cache)
  (actual time=0.025..0.045 rows=10 loops=1)
Planning Time: 0.123 ms
Execution Time: 0.067 ms

Key metrics:
- Buffers: shared hit=4 means 4 pages read from cache
- Output: Shows returned columns
- Verbose: Shows schema names
*/
```

---

### **2. pg_stat_user_indexes (Index Statistics):**

#### **Check Index Usage Statistics:**

```sql
-- View all indexes and their usage
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan AS index_scans,         -- Number of times index was used
    idx_tup_read AS tuples_read,     -- Total tuples read from index
    idx_tup_fetch AS tuples_fetched  -- Tuples fetched from table using index
FROM pg_stat_user_indexes
WHERE schemaname = 'public'
ORDER BY idx_scan DESC;

/*
Result:
schemaname | tablename | indexname            | index_scans | tuples_read | tuples_fetched
-----------|-----------|----------------------|-------------|-------------|----------------
public     | orders    | idx_orders_customer  | 15,432      | 154,320     | 154,320
public     | orders    | idx_orders_date      | 8,765       | 87,650      | 87,650
public     | orders    | idx_orders_status    | 0           | 0           | 0  ← Never used!
public     | users     | users_pkey           | 125,678     | 125,678     | 125,678

Interpretation:
- idx_orders_customer: Heavily used (15,432 scans)
- idx_orders_status: Never used (0 scans) → Candidate for removal
*/
```

#### **Find Unused Indexes:**

```sql
-- Indexes that are never used
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan AS times_used,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
    pg_get_indexdef(indexrelid) AS definition
FROM pg_stat_user_indexes
WHERE idx_scan = 0                          -- Never scanned
  AND indexrelid NOT IN (
      SELECT indexrelid 
      FROM pg_index 
      WHERE indisunique = TRUE OR indisprimary = TRUE
  )                                          -- Exclude unique/primary key indexes
ORDER BY pg_relation_size(indexrelid) DESC;

/*
Result:
indexname              | times_used | index_size | definition
-----------------------|------------|------------|-----------------------------
idx_orders_status      | 0          | 2208 kB    | CREATE INDEX idx_orders_...
idx_products_category  | 0          | 1536 kB    | CREATE INDEX idx_products...

Action: Consider dropping these unused indexes (save 3.7 MB)
*/
```

#### **Index Usage Efficiency:**

```sql
-- Calculate index usage efficiency
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
    CASE 
        WHEN idx_scan = 0 THEN 'Never used - DROP'
        WHEN idx_scan < 100 THEN 'Rarely used'
        WHEN idx_scan < 1000 THEN 'Moderately used'
        WHEN idx_scan < 10000 THEN 'Frequently used'
        ELSE 'Heavily used'
    END AS usage_category,
    ROUND(
        pg_relation_size(indexrelid)::NUMERIC / NULLIF(idx_scan, 0) / 1024, 2
    ) AS kb_per_scan
FROM pg_stat_user_indexes
WHERE schemaname = 'public'
ORDER BY idx_scan ASC, pg_relation_size(indexrelid) DESC;

/*
Result:
indexname              | idx_scan | size    | usage_category | kb_per_scan
-----------------------|----------|---------|----------------|-------------
idx_rarely_used        | 5        | 1 MB    | Rarely used    | 204.8
idx_orders_customer    | 15432    | 2.2 MB  | Heavily used   | 0.15
idx_orders_date        | 8765     | 1.8 MB  | Frequently used| 0.21

Interpretation:
- Low kb_per_scan = Efficient (worth the space)
- High kb_per_scan = Wasteful (large index, rarely used)
*/
```

---

### **3. pg_stat_statements (Query-Level Analysis):**

#### **Enable pg_stat_statements:**

```sql
-- Enable extension (requires postgresql.conf change and restart)
-- In postgresql.conf:
-- shared_preload_libraries = 'pg_stat_statements'

-- Create extension
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- Reset statistics
SELECT pg_stat_statements_reset();
```

#### **Analyze Query Patterns:**

```sql
-- Find queries and whether they use indexes
SELECT 
    query,
    calls,
    total_exec_time / calls AS avg_time_ms,
    rows / calls AS avg_rows,
    100.0 * shared_blks_hit / NULLIF(shared_blks_hit + shared_blks_read, 0) AS cache_hit_ratio
FROM pg_stat_statements
WHERE query LIKE '%orders%'
  AND calls > 10
ORDER BY total_exec_time DESC
LIMIT 10;

/*
Result:
query                                      | calls | avg_time_ms | avg_rows | cache_hit_ratio
-------------------------------------------|-------|-------------|----------|----------------
SELECT * FROM orders WHERE customer_id=?  | 5000  | 0.25        | 10       | 99.8
SELECT * FROM orders WHERE status=?       | 3000  | 45.5        | 50000    | 85.2

Interpretation:
- First query: Fast (0.25ms), high cache hit = Using index
- Second query: Slow (45.5ms), lower cache hit = No index (seq scan)
*/

-- Find slow queries that might benefit from indexes
SELECT 
    LEFT(query, 60) AS query_preview,
    calls,
    mean_exec_time AS avg_ms,
    max_exec_time AS max_ms,
    stddev_exec_time AS stddev_ms
FROM pg_stat_statements
WHERE mean_exec_time > 10  -- Slower than 10ms average
ORDER BY mean_exec_time DESC
LIMIT 10;

-- These queries are candidates for new indexes
```

---

### **4. Index Scan Types:**

#### **Understanding Scan Types:**

```sql
-- Different index scan methods in EXPLAIN

-- 1. Index Scan (most common)
EXPLAIN
SELECT * FROM orders WHERE customer_id = 5000;
/*
Index Scan using idx_orders_customer
- Directly uses index to find rows
- Best for small result sets
*/

-- 2. Index Only Scan (most efficient)
EXPLAIN
SELECT customer_id FROM orders WHERE customer_id = 5000;
/*
Index Only Scan using idx_orders_customer
- All needed data in index (no table access)
- Fastest: reads only index, not table
*/

-- 3. Bitmap Index Scan (multiple indexes or large results)
EXPLAIN
SELECT * FROM orders WHERE customer_id = 5000 OR order_date > '2024-01-01';
/*
Bitmap Heap Scan on orders
  Recheck Cond: (customer_id = 5000) OR (order_date > '2024-01-01')
  ->  BitmapOr
        ->  Bitmap Index Scan on idx_orders_customer
        ->  Bitmap Index Scan on idx_orders_date
        
- Combines multiple indexes
- Good for larger result sets
*/

-- 4. Sequential Scan (no index used)
EXPLAIN
SELECT * FROM orders WHERE status = 'pending';
/*
Seq Scan on orders
  Filter: (status = 'pending')
  
- No index available or not beneficial
- Reads entire table
*/
```

---

### **5. Real-Time Monitoring:**

#### **Monitor Active Index Scans:**

```sql
-- See currently running queries and their index usage
SELECT 
    pid,
    usename,
    LEFT(query, 80) AS query_preview,
    state,
    query_start,
    NOW() - query_start AS duration
FROM pg_stat_activity
WHERE state = 'active'
  AND query NOT LIKE '%pg_stat_activity%'
ORDER BY query_start;

-- Then EXPLAIN the slow queries to check index usage
```

#### **Index Cache Hit Ratio:**

```sql
-- Check if indexes are cached (in memory)
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan,
    pg_size_pretty(pg_relation_size(indexrelid)) AS size,
    CASE 
        WHEN idx_blks_hit + idx_blks_read = 0 THEN 0
        ELSE ROUND(100.0 * idx_blks_hit / (idx_blks_hit + idx_blks_read), 2)
    END AS cache_hit_ratio
FROM pg_statio_user_indexes
WHERE schemaname = 'public'
ORDER BY idx_scan DESC;

/*
Result:
indexname              | idx_scan | size    | cache_hit_ratio
-----------------------|----------|---------|----------------
idx_orders_customer    | 15432    | 2.2 MB  | 99.85  ← Excellent (cached)
idx_orders_date        | 8765     | 1.8 MB  | 98.12  ← Good
idx_rarely_used        | 5        | 5 MB    | 45.23  ← Poor (too large, not cached)

High cache_hit_ratio = Index frequently accessed and cached
Low cache_hit_ratio = Index too large or rarely used
*/
```

---

### **6. Testing Index Usage:**

#### **Force or Disable Index Usage:**

```sql
-- Disable sequential scan to force index use (testing only)
SET enable_seqscan = OFF;

EXPLAIN ANALYZE
SELECT * FROM orders WHERE status = 'pending';
-- Forces index scan even if inefficient

-- Re-enable
SET enable_seqscan = ON;

-- Disable specific index scan types
SET enable_indexscan = OFF;    -- Disable regular index scans
SET enable_bitmapscan = OFF;   -- Disable bitmap scans
SET enable_indexonlyscan = OFF; -- Disable index-only scans

-- Reset all
RESET enable_seqscan;
RESET enable_indexscan;
RESET enable_bitmapscan;
RESET enable_indexonlyscan;

-- Test with different cost settings
SET random_page_cost = 1.1;  -- SSD (default 4.0 for HDD)
EXPLAIN ANALYZE
SELECT * FROM orders WHERE customer_id = 5000;
-- May change planner decisions

RESET random_page_cost;
```

---

### **7. Comprehensive Index Report:**

#### **Complete Index Health Check:**

```sql
-- Full index analysis query
WITH index_usage AS (
    SELECT 
        schemaname,
        tablename,
        indexname,
        idx_scan,
        pg_relation_size(indexrelid) AS index_bytes,
        pg_relation_size(relid) AS table_bytes
    FROM pg_stat_user_indexes psu
    JOIN pg_stat_user_tables pst ON psu.tablename = pst.relname
),
index_details AS (
    SELECT 
        schemaname,
        tablename,
        indexname,
        idx_scan,
        pg_size_pretty(index_bytes) AS index_size,
        pg_size_pretty(table_bytes) AS table_size,
        ROUND(100.0 * index_bytes / NULLIF(table_bytes, 0), 2) AS index_table_ratio,
        CASE 
            WHEN idx_scan = 0 THEN '❌ Never used'
            WHEN idx_scan < 100 THEN '⚠️ Rarely used'
            WHEN idx_scan < 1000 THEN '🟡 Moderate'
            WHEN idx_scan < 10000 THEN '🟢 Frequent'
            ELSE '✅ Heavy'
        END AS usage_status
    FROM index_usage
)
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan AS scans,
    index_size,
    table_size,
    index_table_ratio || '%' AS size_ratio,
    usage_status,
    CASE 
        WHEN idx_scan = 0 THEN 'Consider dropping'
        WHEN idx_scan < 100 AND index_bytes > 10485760 THEN 'Review necessity'
        ELSE 'Keep'
    END AS recommendation
FROM index_details
ORDER BY 
    CASE 
        WHEN idx_scan = 0 THEN 0
        WHEN idx_scan < 100 THEN 1
        ELSE 2
    END,
    index_bytes DESC;
```

---

### **8. Automated Monitoring Script:**

```sql
-- Daily index usage report
DO $$
DECLARE
    unused_count INTEGER;
    total_waste BIGINT;
BEGIN
    -- Count unused indexes
    SELECT COUNT(*), SUM(pg_relation_size(indexrelid))
    INTO unused_count, total_waste
    FROM pg_stat_user_indexes
    WHERE idx_scan = 0
      AND indexrelid NOT IN (
          SELECT indexrelid FROM pg_index WHERE indisunique OR indisprimary
      );
    
    RAISE NOTICE '===== Index Usage Report =====';
    RAISE NOTICE 'Unused indexes: %', unused_count;
    RAISE NOTICE 'Wasted space: %', pg_size_pretty(total_waste);
    
    -- List unused indexes
    FOR rec IN 
        SELECT indexname, pg_size_pretty(pg_relation_size(indexrelid)) AS size
        FROM pg_stat_user_indexes
        WHERE idx_scan = 0
          AND indexrelid NOT IN (
              SELECT indexrelid FROM pg_index WHERE indisunique OR indisprimary
          )
        ORDER BY pg_relation_size(indexrelid) DESC
    LOOP
        RAISE NOTICE 'Unused: % (size: %)', rec.indexname, rec.size;
    END LOOP;
END $$;
```

---

### **9. Best Practices:**

```sql
-- ✅ DO: Use EXPLAIN ANALYZE before creating indexes
EXPLAIN ANALYZE SELECT * FROM orders WHERE customer_id = 5000;
-- Check if existing indexes are used

-- ✅ DO: Monitor index usage regularly
SELECT * FROM pg_stat_user_indexes WHERE idx_scan = 0;
-- Weekly review

-- ✅ DO: Check query plans for slow queries
-- Enable auto_explain for production monitoring
-- In postgresql.conf:
-- session_preload_libraries = 'auto_explain'
-- auto_explain.log_min_duration = 1000  -- Log queries > 1 second

-- ✅ DO: Reset statistics after major changes
SELECT pg_stat_reset();
-- Then monitor fresh usage patterns

-- ✅ DO: Test indexes in dev before production
-- Create index, run typical queries, check EXPLAIN

-- ❌ DON'T: Drop indexes without checking
-- Always verify idx_scan = 0 for sustained period

-- ❌ DON'T: Rely solely on EXPLAIN (no ANALYZE)
-- EXPLAIN shows plan, ANALYZE shows actual execution

-- ❌ DON'T: Ignore cache hit ratios
-- Low cache hits indicate index too large or rarely used

-- 💡 TIP: Use pgAdmin or pgBadger for visual analysis
-- Easier to spot patterns than raw SQL queries
```

---

### **10. Common Scenarios:**

#### **Scenario 1: Index Exists But Not Used:**

```sql
-- Index exists
CREATE INDEX idx_orders_status ON orders(status);

-- Query doesn't use it
EXPLAIN SELECT * FROM orders WHERE status = 'completed';
-- Seq Scan (why?)

-- Reasons:
-- 1. Query returns too many rows (> 10% of table)
SELECT COUNT(*) FROM orders WHERE status = 'completed';
-- Result: 950,000 out of 1,000,000 (95%)
-- Solution: Index not beneficial, sequential scan faster

-- 2. Statistics outdated
ANALYZE orders;
-- Updates table statistics

-- 3. Index bloated
REINDEX INDEX idx_orders_status;

-- 4. Wrong data type
-- If status is TEXT but querying with VARCHAR, might not use index
```

#### **Scenario 2: Need Index-Only Scan:**

```sql
-- Current: Index scan + table access
EXPLAIN ANALYZE
SELECT customer_id, order_date FROM orders WHERE customer_id = 5000;
/*
Index Scan using idx_orders_customer
  → Reads index, then fetches rows from table
*/

-- Solution: Covering index with INCLUDE
CREATE INDEX idx_orders_customer_covering ON orders(customer_id) 
INCLUDE (order_date);

EXPLAIN ANALYZE
SELECT customer_id, order_date FROM orders WHERE customer_id = 5000;
/*
Index Only Scan using idx_orders_customer_covering
  Heap Fetches: 0  ← Perfect! No table access
  → All data from index
*/
```

---

### **Interview Tips:**
- Primary method: EXPLAIN ANALYZE shows actual query execution with index usage
- Statistics view: pg_stat_user_indexes shows idx_scan (usage count)
- Unused indexes: Query where idx_scan = 0 (candidates for removal)
- Scan types: Index Scan (direct), Index Only Scan (fastest), Bitmap Index Scan (multiple indexes), Seq Scan (no index)
- Efficiency metric: kb_per_scan (index size / usage count)
- Cache hit ratio: High ratio (>95%) = well-utilized index
- Real-time: pg_stat_activity shows running queries, EXPLAIN them
- Testing: Use enable_seqscan = OFF to force index usage (development only)
- Common issue: Index exists but not used → query returns >10% rows or statistics outdated
- Best practice: Weekly review with idx_scan = 0 query

</details>

<details>
<summary>40. What is REINDEX and when should you use it?</summary>

### Answer:

**REINDEX** rebuilds an index from scratch, removing bloat and corruption while restoring optimal performance. It's essential for maintaining index health in high-write workloads, though it locks the table during rebuild (except when using CONCURRENTLY option in PostgreSQL 12+).

---

### **REINDEX Overview:**

```sql
-- REINDEX rebuilds index completely
REINDEX INDEX idx_users_email;
-- Creates new index structure
-- Drops old index
-- Removes all bloat

-- Before REINDEX: 150 MB (bloated)
-- After REINDEX: 85 MB (compact)
-- Performance restored
```

---

### **1. REINDEX Syntax:**

#### **Basic Forms:**

```sql
-- Reindex a single index
REINDEX INDEX idx_users_email;
-- Locks table during rebuild

-- Reindex all indexes on a table
REINDEX TABLE users;
-- Rebuilds all indexes including primary key

-- Reindex all indexes in a schema
REINDEX SCHEMA public;
-- Rebuilds all indexes in schema

-- Reindex entire database
REINDEX DATABASE mydb;
-- Rebuilds all indexes in database (very slow)

-- Reindex system catalogs
REINDEX SYSTEM mydb;
-- Rebuilds system catalog indexes
```

#### **REINDEX CONCURRENTLY (PostgreSQL 12+):**

```sql
-- Non-blocking reindex (production-safe)
REINDEX INDEX CONCURRENTLY idx_users_email;
-- Table remains accessible during rebuild
-- Reads and writes continue normally

-- Table-level concurrent reindex
REINDEX TABLE CONCURRENTLY users;
-- Rebuilds all indexes without blocking

-- Limitations:
-- 1. Cannot run inside transaction
BEGIN;
REINDEX INDEX CONCURRENTLY idx_users_email;  -- ERROR
ROLLBACK;

-- 2. Takes longer than regular REINDEX
-- Regular: 30 seconds, Concurrent: 45-60 seconds

-- 3. Requires more disk space temporarily
-- Old index + new index exist simultaneously
```

---

### **2. When to Use REINDEX:**

#### **Reason 1: Index Bloat:**

```sql
-- Scenario: Heavy UPDATE workload
CREATE TABLE sessions (
    session_id SERIAL PRIMARY KEY,
    user_id INTEGER,
    last_activity TIMESTAMP,
    data JSONB
);

CREATE INDEX idx_sessions_user ON sessions(user_id);

-- After millions of updates
SELECT 
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS current_size
FROM pg_stat_user_indexes
WHERE indexname = 'idx_sessions_user';
-- Result: 250 MB (bloated)

-- Check bloat using pgstattuple
SELECT 
    avg_leaf_density,
    leaf_pages,
    deleted_pages
FROM pgstatindex('idx_sessions_user');
/*
avg_leaf_density | leaf_pages | deleted_pages
-----------------|------------|---------------
62.5             | 30000      | 8000

Low density (< 70%) and many deleted pages = bloat!
*/

-- REINDEX to fix
REINDEX INDEX CONCURRENTLY idx_sessions_user;

-- After REINDEX
SELECT 
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS new_size
FROM pg_stat_user_indexes
WHERE indexname = 'idx_sessions_user';
-- Result: 140 MB (44% reduction!)
```

#### **Reason 2: Index Corruption:**

```sql
-- Symptoms of corruption:
-- - Queries return incorrect results
-- - Index scans fail with errors
-- - Constraint violations not detected

-- Error message example:
-- ERROR: index "idx_users_email" contains unexpected zero page at block 42
-- HINT: Please REINDEX it.

-- Fix corruption
REINDEX INDEX idx_users_email;
-- Rebuilds from table data (source of truth)

-- For critical tables, reindex all
REINDEX TABLE users;
```

#### **Reason 3: Performance Degradation:**

```sql
-- Query performance degraded over time
-- Before (6 months ago): 5ms
-- Now: 50ms (10x slower)

-- Check query plan
EXPLAIN ANALYZE
SELECT * FROM orders WHERE customer_id = 5000;
/*
Index Scan using idx_orders_customer
  Buffers: shared hit=150  ← High buffer reads (bloated index)
Execution Time: 50 ms
*/

-- REINDEX
REINDEX INDEX CONCURRENTLY idx_orders_customer;

-- After REINDEX
EXPLAIN ANALYZE
SELECT * FROM orders WHERE customer_id = 5000;
/*
Index Scan using idx_orders_customer
  Buffers: shared hit=15  ← Much fewer buffer reads
Execution Time: 5 ms  ← 10x faster!
*/
```

#### **Reason 4: After Bulk Operations:**

```sql
-- Scenario: Bulk delete removes 80% of table
DELETE FROM log_entries WHERE created_at < CURRENT_DATE - INTERVAL '90 days';
-- Deleted 8 million of 10 million rows

-- Indexes still contain dead entries
SELECT 
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS size
FROM pg_stat_user_indexes
WHERE tablename = 'log_entries';
-- Result: Indexes still 200 MB (should be 40 MB)

-- REINDEX to reclaim space
REINDEX TABLE CONCURRENTLY log_entries;

-- After REINDEX
SELECT 
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS size
FROM pg_stat_user_indexes
WHERE tablename = 'log_entries';
-- Result: 42 MB (79% space reclaimed)
```

#### **Reason 5: Changing Storage Parameters:**

```sql
-- Change index fillfactor
CREATE INDEX idx_users_status ON users(status) WITH (fillfactor = 90);

-- Want to change to 70 for more HOT updates
-- Can't ALTER, must REINDEX with new parameters

-- Drop and recreate with new fillfactor
DROP INDEX idx_users_status;
CREATE INDEX idx_users_status ON users(status) WITH (fillfactor = 70);

-- Or REINDEX (PostgreSQL 14+)
REINDEX (FILLFACTOR = 70) INDEX idx_users_status;
```

---

### **3. REINDEX vs Alternatives:**

#### **REINDEX vs VACUUM:**

```sql
-- VACUUM: Marks dead tuples for reuse (doesn't shrink)
VACUUM users;
-- Pros: No locks, fast, safe anytime
-- Cons: Doesn't reclaim disk space

-- VACUUM FULL: Rebuilds table and indexes (shrinks)
VACUUM FULL users;
-- Pros: Reclaims all space
-- Cons: Exclusive lock, very slow, blocks all access

-- REINDEX: Rebuilds only indexes
REINDEX INDEX CONCURRENTLY idx_users_email;
-- Pros: Fixes index bloat, no table lock (CONCURRENTLY)
-- Cons: Only fixes indexes, not table

-- When to use what:
-- - Regular maintenance: VACUUM (autovacuum handles this)
-- - Index bloat: REINDEX CONCURRENTLY
-- - Extreme bloat (table + indexes): VACUUM FULL (with downtime)
```

#### **REINDEX vs DROP + CREATE:**

```sql
-- Method 1: REINDEX
REINDEX INDEX CONCURRENTLY idx_users_email;
-- Pros: Simple, one command
-- Cons: None (best choice)

-- Method 2: DROP + CREATE
DROP INDEX idx_users_email;
CREATE INDEX CONCURRENTLY idx_users_email_new ON users(email);
-- Pros: More control, can rename
-- Cons: More steps, queries fail between DROP and CREATE

-- Method 3: CREATE + DROP (safer)
CREATE INDEX CONCURRENTLY idx_users_email_new ON users(email);
DROP INDEX idx_users_email;
ALTER INDEX idx_users_email_new RENAME TO idx_users_email;
-- Pros: No downtime, old index available until new one ready
-- Cons: Requires 2x disk space temporarily

-- Winner: REINDEX CONCURRENTLY (simplest, safest)
```

---

### **4. Performance Impact:**

#### **Timing and Resources:**

```sql
-- Test REINDEX time on different sizes
CREATE TABLE test_reindex (
    id SERIAL PRIMARY KEY,
    value VARCHAR(100),
    data TEXT
);

-- Test 1: Small index (1 MB)
INSERT INTO test_reindex SELECT i, 'value-'||i, 'data' FROM generate_series(1, 10000) i;
CREATE INDEX idx_test_value ON test_reindex(value);

\timing
REINDEX INDEX idx_test_value;
-- Time: 50 ms

-- Test 2: Medium index (100 MB)
INSERT INTO test_reindex SELECT i, 'value-'||i, 'data' FROM generate_series(1, 1000000) i;
\timing
REINDEX INDEX idx_test_value;
-- Time: 5 seconds

-- Test 3: Large index (5 GB)
-- Time: 5-10 minutes

-- CONCURRENTLY is slower
REINDEX INDEX CONCURRENTLY idx_test_value;
-- Time: 8 seconds (60% slower, but no locks)
```

#### **Blocking vs Non-Blocking:**

```sql
-- Regular REINDEX (blocks writes)
REINDEX INDEX idx_users_email;
-- Acquires AccessExclusiveLock
-- Blocks: SELECT, INSERT, UPDATE, DELETE
-- Duration: Fast, but disruptive

-- REINDEX CONCURRENTLY (no blocks)
REINDEX INDEX CONCURRENTLY idx_users_email;
-- Acquires ShareUpdateExclusiveLock
-- Allows: SELECT, INSERT, UPDATE, DELETE
-- Blocks: Other REINDEXes, schema changes
-- Duration: Slower, but production-safe

-- Check locks during REINDEX
SELECT 
    pid,
    mode,
    granted,
    query
FROM pg_locks l
JOIN pg_stat_activity a ON l.pid = a.pid
WHERE relation = 'idx_users_email'::regclass;
```

---

### **5. Monitoring REINDEX Progress:**

#### **Track REINDEX Status:**

```sql
-- View active REINDEX operations (PostgreSQL 12+)
SELECT 
    pid,
    phase,
    blocks_total,
    blocks_done,
    ROUND(100.0 * blocks_done / NULLIF(blocks_total, 0), 2) AS percent_complete,
    tuples_total,
    tuples_done
FROM pg_stat_progress_create_index
WHERE command = 'REINDEX';

/*
Result (during REINDEX CONCURRENTLY):
pid   | phase              | blocks_total | blocks_done | percent_complete | tuples_total | tuples_done
------|--------------------|--------------| ------------|------------------|--------------|-------------
12345 | building index     | 50000        | 25000       | 50.00            | 5000000      | 2500000

Phases:
1. initializing
2. building index (main work)
3. waiting for old snapshots
4. finalizing
*/
```

#### **Monitor Space Usage:**

```sql
-- Check disk space before REINDEX CONCURRENTLY
SELECT 
    pg_size_pretty(pg_total_relation_size('users')) AS table_total_size,
    pg_size_pretty(sum(pg_relation_size(indexrelid))) AS indexes_size
FROM pg_stat_user_indexes
WHERE tablename = 'users';

/*
Result:
table_total_size | indexes_size
-----------------|-------------
500 MB           | 200 MB

REINDEX CONCURRENTLY needs: ~200 MB extra during rebuild
Ensure sufficient disk space!
*/
```

---

### **6. Automated REINDEX Strategy:**

#### **Scheduled Maintenance:**

```sql
-- Weekly REINDEX script for bloated indexes
DO $$
DECLARE
    idx RECORD;
    bloat_pct NUMERIC;
BEGIN
    FOR idx IN 
        SELECT 
            schemaname,
            tablename,
            indexname,
            pg_relation_size(indexrelid) AS index_size
        FROM pg_stat_user_indexes
        WHERE schemaname = 'public'
          AND idx_scan > 0  -- Only used indexes
          AND pg_relation_size(indexrelid) > 10 * 1024 * 1024  -- > 10 MB
    LOOP
        -- Check bloat (simplified)
        SELECT 
            CASE 
                WHEN idx_tup_read = 0 THEN 0
                ELSE ROUND((idx_tup_read - idx_tup_fetch)::NUMERIC / idx_tup_read * 100, 2)
            END
        INTO bloat_pct
        FROM pg_stat_user_indexes
        WHERE indexname = idx.indexname;
        
        -- REINDEX if bloat > 30%
        IF bloat_pct > 30 THEN
            RAISE NOTICE 'Reindexing % (bloat: %)', idx.indexname, bloat_pct;
            EXECUTE format('REINDEX INDEX CONCURRENTLY %I', idx.indexname);
        END IF;
    END LOOP;
END $$;
```

#### **Conditional REINDEX:**

```sql
-- REINDEX based on conditions
CREATE OR REPLACE FUNCTION auto_reindex_if_needed(
    idx_name TEXT,
    bloat_threshold NUMERIC DEFAULT 30.0
) RETURNS VOID AS $$
DECLARE
    current_bloat NUMERIC;
    index_size BIGINT;
BEGIN
    -- Get index size
    SELECT pg_relation_size(idx_name::regclass) INTO index_size;
    
    -- Skip small indexes (< 50 MB)
    IF index_size < 50 * 1024 * 1024 THEN
        RAISE NOTICE 'Index % too small, skipping', idx_name;
        RETURN;
    END IF;
    
    -- Check bloat using pgstattuple
    SELECT 100 - avg_leaf_density INTO current_bloat
    FROM pgstatindex(idx_name);
    
    -- REINDEX if bloated
    IF current_bloat > bloat_threshold THEN
        RAISE NOTICE 'Reindexing % (bloat: %)', idx_name, current_bloat;
        EXECUTE format('REINDEX INDEX CONCURRENTLY %I', idx_name);
    ELSE
        RAISE NOTICE 'Index % healthy (bloat: %)', idx_name, current_bloat;
    END IF;
END;
$$ LANGUAGE plpgsql;

-- Use function
SELECT auto_reindex_if_needed('idx_users_email');
SELECT auto_reindex_if_needed('idx_orders_customer', 25.0);  -- Custom threshold
```

---

### **7. REINDEX Best Practices:**

```sql
-- ✅ DO: Use REINDEX CONCURRENTLY in production
REINDEX INDEX CONCURRENTLY idx_users_email;
-- No downtime

-- ✅ DO: Schedule during low-traffic periods
-- Even CONCURRENTLY has overhead
-- Run at night or weekends

-- ✅ DO: Monitor bloat regularly
SELECT * FROM pgstatindex('idx_users_email');
-- Weekly check

-- ✅ DO: REINDEX after bulk operations
DELETE FROM large_table WHERE ...;  -- Delete 80% of rows
REINDEX TABLE CONCURRENTLY large_table;  -- Reclaim space

-- ✅ DO: Check disk space before CONCURRENTLY
-- Needs 2x index size temporarily

-- ✅ DO: Test in development first
-- Verify timing and impact

-- ❌ DON'T: Use regular REINDEX in production
REINDEX INDEX idx_users_email;  -- Locks table!
-- Use CONCURRENTLY instead

-- ❌ DON'T: REINDEX everything unnecessarily
REINDEX DATABASE mydb;  -- Overkill, locks everything
-- Target specific bloated indexes

-- ❌ DON'T: REINDEX inside transactions
BEGIN;
REINDEX INDEX CONCURRENTLY idx_users_email;  -- ERROR
COMMIT;

-- ❌ DON'T: Forget autovacuum tuning
-- REINDEX is reactive, autovacuum is preventive
-- Tune autovacuum to prevent bloat

-- 💡 TIP: Combine with ANALYZE
REINDEX INDEX CONCURRENTLY idx_users_email;
ANALYZE users;  -- Update statistics after REINDEX
```

---

### **8. Troubleshooting:**

#### **REINDEX Failures:**

```sql
-- Error: Out of disk space
-- REINDEX CONCURRENTLY failed: no space left on device
-- Solution: Free space or use regular REINDEX (smaller temp space)

-- Error: Another REINDEX in progress
-- ERROR: cannot reindex concurrently table "users" because it has already been marked for reindex
-- Solution: Wait for existing REINDEX to complete or cancel it
SELECT pg_cancel_backend(pid) FROM pg_stat_activity WHERE query LIKE '%REINDEX%';

-- Error: Invalid index
-- ERROR: could not create unique index "idx_users_email"
-- DETAIL: Key (email)=(john@example.com) is duplicated.
-- Solution: Fix data duplicates before REINDEX UNIQUE indexes
```

#### **Performance Issues:**

```sql
-- REINDEX taking too long
-- Check system resources
SELECT 
    pid,
    wait_event_type,
    wait_event,
    state
FROM pg_stat_activity
WHERE query LIKE '%REINDEX%';

-- If wait_event = 'IO', disk is slow
-- If wait_event = 'Lock', waiting for locks
-- If wait_event = NULL and state = 'active', working normally

-- Speed up REINDEX (development only)
SET maintenance_work_mem = '2GB';  -- More memory for index build
REINDEX INDEX idx_large_table;
-- Faster but uses more memory
```

---

### **9. Real-World Example:**

```sql
-- Production scenario: E-commerce orders table

-- Problem: Slow order queries after 6 months
SELECT * FROM orders WHERE customer_id = 5000;
-- Before: 5ms
-- Now: 50ms (10x slower)

-- Step 1: Check bloat
SELECT 
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS size,
    idx_scan,
    (SELECT avg_leaf_density FROM pgstatindex(indexname)) AS density
FROM pg_stat_user_indexes
WHERE tablename = 'orders';

/*
Result:
indexname              | size    | idx_scan | density
-----------------------|---------|----------|--------
idx_orders_customer    | 250 MB  | 500000   | 62.5    ← Low density!
idx_orders_date        | 180 MB  | 300000   | 85.2    ← Healthy
*/

-- Step 2: REINDEX bloated index (off-peak hours: 2 AM)
REINDEX INDEX CONCURRENTLY idx_orders_customer;
-- Duration: 8 minutes
-- No downtime

-- Step 3: Verify improvement
SELECT 
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS size,
    (SELECT avg_leaf_density FROM pgstatindex(indexname)) AS density
FROM pg_stat_user_indexes
WHERE indexname = 'idx_orders_customer';

/*
Result:
indexname              | size    | density
-----------------------|---------|--------
idx_orders_customer    | 145 MB  | 89.8    ← Improved!
*/

-- Step 4: Test query performance
EXPLAIN ANALYZE
SELECT * FROM orders WHERE customer_id = 5000;
-- Execution Time: 5 ms ← Back to normal!

-- Step 5: Update statistics
ANALYZE orders;

-- Result: Performance restored, 42% space saved
```

---

### **Interview Tips:**
- Define: REINDEX rebuilds index from scratch, removing bloat and corruption
- When to use: Index bloat (>30%), corruption errors, performance degradation, after bulk deletes
- CONCURRENTLY: PostgreSQL 12+, no table locks, production-safe (takes 50% longer)
- Regular REINDEX: Faster but locks table (blocks all operations)
- vs VACUUM: VACUUM marks dead tuples reuse, REINDEX rebuilds entirely
- Performance: Small index (1 MB) = 50ms, large (5 GB) = 5-10 minutes
- Monitoring: pg_stat_progress_create_index shows progress
- Best practice: Use CONCURRENTLY during low-traffic, schedule weekly for high-churn indexes
- Real example: 250 MB bloated to 145 MB, query 50ms → 5ms (10x faster)
- Automation: Check bloat weekly, REINDEX if >30% bloated

</details>
