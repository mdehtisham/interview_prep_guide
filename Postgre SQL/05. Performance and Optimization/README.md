## **Performance and Optimization**

<details>
<summary><strong>41. How do you analyze query performance in PostgreSQL?</strong></summary>

### **Answer:**

Analyzing query performance in PostgreSQL is a systematic process that involves multiple tools and techniques to identify bottlenecks, understand execution patterns, and optimize database operations.

---

### **1. Core Analysis Tools**

#### **A. EXPLAIN and EXPLAIN ANALYZE**

**Basic EXPLAIN:**
```sql
-- Get the query plan without executing
EXPLAIN 
SELECT * FROM orders 
WHERE customer_id = 123 
  AND order_date > '2024-01-01';
```

**Output:**
```
Seq Scan on orders  (cost=0.00..1892.50 rows=25 width=180)
  Filter: ((customer_id = 123) AND (order_date > '2024-01-01'::date))
```

**EXPLAIN ANALYZE (with actual execution):**
```sql
-- Execute query and show actual timing
EXPLAIN ANALYZE
SELECT c.customer_name, COUNT(*) as order_count,
       SUM(o.total_amount) as total_spent
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_date >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY c.customer_id, c.customer_name
ORDER BY total_spent DESC
LIMIT 10;
```

**Output:**
```
Limit  (cost=2850.45..2850.48 rows=10 width=48) 
       (actual time=125.234..125.245 rows=10 loops=1)
  ->  Sort  (cost=2850.45..2875.50 rows=10020 width=48)
            (actual time=125.232..125.238 rows=10 loops=1)
        Sort Key: (sum(o.total_amount)) DESC
        Sort Method: top-N heapsort  Memory: 25kB
        ->  HashAggregate  (cost=2450.00..2650.20 rows=10020 width=48)
                          (actual time=98.456..115.789 rows=8945 loops=1)
              Group Key: c.customer_id, c.customer_name
              ->  Hash Join  (cost=850.00..2200.50 rows=24950 width=44)
                            (actual time=12.234..75.456 rows=25123 loops=1)
                    Hash Cond: (o.customer_id = c.customer_id)
                    ->  Seq Scan on orders o  (cost=0.00..1200.00 rows=24950 width=20)
                                              (actual time=0.015..15.234 rows=25123 loops=1)
                          Filter: (order_date >= (CURRENT_DATE - '30 days'::interval))
                          Rows Removed by Filter: 74877
                    ->  Hash  (cost=600.00..600.00 rows=20000 width=36)
                              (actual time=12.156..12.157 rows=20000 loops=1)
                          Buckets: 32768  Batches: 1  Memory Usage: 1456kB
                          ->  Seq Scan on customers c  (cost=0.00..600.00 rows=20000 width=36)
                                                       (actual time=0.008..6.234 rows=20000 loops=1)
Planning Time: 1.234 ms
Execution Time: 125.456 ms
```

**EXPLAIN options:**
```sql
-- Comprehensive analysis
EXPLAIN (ANALYZE, BUFFERS, VERBOSE, COSTS, TIMING, SUMMARY)
SELECT * FROM large_table WHERE status = 'active';
```

---

### **2. Query Timing Measurements**

#### **A. \timing in psql**

```sql
-- Enable timing in psql
\timing on

SELECT COUNT(*) FROM orders;
-- Time: 45.234 ms

SELECT customer_id, COUNT(*) 
FROM orders 
GROUP BY customer_id;
-- Time: 892.456 ms
```

#### **B. pg_stat_statements Extension**

```sql
-- Enable the extension
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- Find slowest queries (by total time)
SELECT 
    query,
    calls,
    total_exec_time,
    mean_exec_time,
    max_exec_time,
    stddev_exec_time,
    rows,
    100.0 * shared_blks_hit / NULLIF(shared_blks_hit + shared_blks_read, 0) AS cache_hit_ratio
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;
```

**Output:**
```
                    query                    | calls | total_exec_time | mean_exec_time | max_exec_time 
--------------------------------------------+-------+-----------------+----------------+---------------
 SELECT * FROM orders WHERE customer_id=$1  | 15234 |    145234.567   |      9.534     |    1234.567
 UPDATE inventory SET quantity = quantity-$1|  8945 |     98765.432   |     11.045     |     567.890
```

**Analyze specific query patterns:**
```sql
-- Queries with high variance (unpredictable performance)
SELECT 
    query,
    calls,
    mean_exec_time,
    stddev_exec_time,
    stddev_exec_time / NULLIF(mean_exec_time, 0) as coefficient_of_variation
FROM pg_stat_statements
WHERE calls > 100
ORDER BY coefficient_of_variation DESC
LIMIT 10;

-- Queries with poor cache hit ratio
SELECT 
    query,
    calls,
    shared_blks_hit,
    shared_blks_read,
    shared_blks_hit::float / NULLIF(shared_blks_hit + shared_blks_read, 0) as hit_ratio
FROM pg_stat_statements
WHERE shared_blks_read > 0
ORDER BY hit_ratio ASC
LIMIT 10;
```

---

### **3. System Performance Monitoring**

#### **A. pg_stat_activity - Active Queries**

```sql
-- Check currently running queries
SELECT 
    pid,
    usename,
    application_name,
    client_addr,
    state,
    query_start,
    NOW() - query_start AS duration,
    wait_event_type,
    wait_event,
    query
FROM pg_stat_activity
WHERE state != 'idle'
  AND query NOT LIKE '%pg_stat_activity%'
ORDER BY duration DESC;
```

**Identify long-running queries:**
```sql
-- Queries running longer than 1 minute
SELECT 
    pid,
    NOW() - query_start AS duration,
    query,
    state
FROM pg_stat_activity
WHERE (NOW() - query_start) > INTERVAL '1 minute'
  AND state = 'active'
ORDER BY duration DESC;
```

#### **B. pg_stat_user_tables - Table Statistics**

```sql
-- Analyze table access patterns
SELECT 
    schemaname,
    tablename,
    seq_scan,
    seq_tup_read,
    idx_scan,
    idx_tup_fetch,
    n_tup_ins,
    n_tup_upd,
    n_tup_del,
    n_live_tup,
    n_dead_tup,
    last_vacuum,
    last_autovacuum,
    last_analyze
FROM pg_stat_user_tables
ORDER BY seq_scan DESC;
```

**Identify tables needing indexes:**
```sql
-- Tables with high sequential scans but few index scans
SELECT 
    schemaname,
    tablename,
    seq_scan,
    seq_tup_read,
    idx_scan,
    seq_tup_read / NULLIF(seq_scan, 0) AS avg_seq_tup,
    CASE 
        WHEN seq_scan > 0 THEN 
            (100.0 * idx_scan / NULLIF(seq_scan + idx_scan, 0))
        ELSE 0 
    END AS index_usage_pct
FROM pg_stat_user_tables
WHERE seq_scan > 100
  AND schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY seq_tup_read DESC
LIMIT 20;
```

#### **C. Buffer Cache Analysis**

```sql
-- Install pg_buffercache extension
CREATE EXTENSION IF NOT EXISTS pg_buffercache;

-- Check buffer cache hit ratios
SELECT 
    CASE 
        WHEN c.relname IS NULL THEN 'Unused'
        ELSE c.relname
    END AS table_name,
    COUNT(*) AS buffers,
    pg_size_pretty(COUNT(*) * 8192::bigint) AS size,
    ROUND(100.0 * COUNT(*) / SUM(COUNT(*)) OVER (), 2) AS percent
FROM pg_buffercache b
LEFT JOIN pg_class c ON b.relfilenode = pg_relation_filenode(c.oid)
                     AND b.reldatabase IN (0, (SELECT oid FROM pg_database WHERE datname = current_database()))
GROUP BY c.relname
ORDER BY COUNT(*) DESC
LIMIT 20;
```

---

### **4. Index Analysis**

#### **A. Missing Indexes Detection**

```sql
-- Tables that might benefit from indexes
SELECT 
    schemaname,
    tablename,
    seq_scan,
    seq_tup_read,
    idx_scan,
    seq_tup_read / NULLIF(seq_scan, 0) AS rows_per_scan
FROM pg_stat_user_tables
WHERE seq_scan > 0
  AND seq_tup_read / NULLIF(seq_scan, 0) > 10000  -- Reading >10k rows per scan
  AND schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY seq_tup_read DESC;
```

#### **B. Unused Indexes Detection**

```sql
-- Indexes that are never used
SELECT 
    schemaname,
    tablename,
    indexrelname,
    idx_scan,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
WHERE idx_scan = 0
  AND indexrelname NOT LIKE 'pg_toast%'
ORDER BY pg_relation_size(indexrelid) DESC;
```

---

### **5. Lock and Wait Event Analysis**

#### **A. Check for Blocking Queries**

```sql
-- Identify blocked and blocking queries
SELECT 
    blocked_locks.pid AS blocked_pid,
    blocked_activity.usename AS blocked_user,
    blocking_locks.pid AS blocking_pid,
    blocking_activity.usename AS blocking_user,
    blocked_activity.query AS blocked_statement,
    blocking_activity.query AS blocking_statement,
    blocked_activity.application_name AS blocked_application
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks 
    ON blocking_locks.locktype = blocked_locks.locktype
    AND blocking_locks.database IS NOT DISTINCT FROM blocked_locks.database
    AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
    AND blocking_locks.page IS NOT DISTINCT FROM blocked_locks.page
    AND blocking_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
    AND blocking_locks.virtualxid IS NOT DISTINCT FROM blocked_locks.virtualxid
    AND blocking_locks.transactionid IS NOT DISTINCT FROM blocked_locks.transactionid
    AND blocking_locks.classid IS NOT DISTINCT FROM blocked_locks.classid
    AND blocking_locks.objid IS NOT DISTINCT FROM blocked_locks.objid
    AND blocking_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
    AND blocking_locks.pid != blocked_locks.pid
JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted;
```

#### **B. Wait Events Monitoring**

```sql
-- Most common wait events
SELECT 
    wait_event_type,
    wait_event,
    COUNT(*) as count,
    ROUND(100.0 * COUNT(*) / SUM(COUNT(*)) OVER (), 2) as percentage
FROM pg_stat_activity
WHERE wait_event IS NOT NULL
GROUP BY wait_event_type, wait_event
ORDER BY count DESC;
```

---

### **6. Disk I/O Analysis**

```sql
-- Table I/O statistics
SELECT 
    schemaname,
    tablename,
    heap_blks_read,
    heap_blks_hit,
    CASE 
        WHEN (heap_blks_hit + heap_blks_read) > 0 THEN
            ROUND(100.0 * heap_blks_hit / (heap_blks_hit + heap_blks_read), 2)
        ELSE 0
    END AS cache_hit_ratio,
    idx_blks_read,
    idx_blks_hit
FROM pg_statio_user_tables
WHERE (heap_blks_read + heap_blks_hit) > 0
ORDER BY heap_blks_read DESC
LIMIT 20;
```

---

### **7. Application-Level Profiling**

#### **A. Auto Explain Extension**

```sql
-- Enable auto_explain for slow queries
LOAD 'auto_explain';
SET auto_explain.log_min_duration = 1000;  -- Log queries slower than 1 second
SET auto_explain.log_analyze = true;
SET auto_explain.log_buffers = true;
SET auto_explain.log_timing = true;
SET auto_explain.log_verbose = true;

-- Now slow queries will automatically be logged with EXPLAIN ANALYZE output
```

#### **B. Query Plan Visualization**

```sql
-- Generate JSON format for visualization tools
EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON)
SELECT * FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.order_date > CURRENT_DATE - INTERVAL '7 days';

-- Use tools like:
-- - explain.depesz.com
-- - explain.dalibo.com
-- - pgAdmin Query Tool
-- - DataGrip
```

---

### **8. Performance Baseline and Comparison**

```sql
-- Create performance baseline
CREATE TABLE query_performance_baseline AS
SELECT 
    query,
    calls,
    total_exec_time,
    mean_exec_time,
    NOW() as baseline_date
FROM pg_stat_statements
WHERE calls > 10;

-- Compare current performance to baseline
SELECT 
    b.query,
    b.mean_exec_time as baseline_time,
    c.mean_exec_time as current_time,
    ROUND(((c.mean_exec_time - b.mean_exec_time) / b.mean_exec_time * 100), 2) as pct_change
FROM query_performance_baseline b
JOIN pg_stat_statements c ON b.query = c.query
WHERE ABS((c.mean_exec_time - b.mean_exec_time) / b.mean_exec_time) > 0.20  -- 20% change
ORDER BY ABS((c.mean_exec_time - b.mean_exec_time) / b.mean_exec_time) DESC;
```

---

### **9. Real-World Performance Analysis Example**

**Scenario:** E-commerce site experiencing slow dashboard loads

```sql
-- Step 1: Identify the slow query
SELECT query, calls, mean_exec_time, total_exec_time
FROM pg_stat_statements
WHERE query LIKE '%dashboard%'
ORDER BY mean_exec_time DESC;
-- Result: Dashboard query taking 3.5 seconds on average

-- Step 2: Analyze the query plan
EXPLAIN (ANALYZE, BUFFERS)
SELECT 
    p.product_name,
    COUNT(oi.order_item_id) as times_ordered,
    SUM(oi.quantity) as total_quantity,
    SUM(oi.quantity * oi.unit_price) as revenue
FROM products p
LEFT JOIN order_items oi ON p.product_id = oi.product_id
LEFT JOIN orders o ON oi.order_id = o.order_id
WHERE o.order_date >= CURRENT_DATE - INTERVAL '30 days'
   OR o.order_date IS NULL
GROUP BY p.product_id, p.product_name
ORDER BY revenue DESC;

-- Result: Sequential scan on orders (2M rows), taking 2.8 seconds

-- Step 3: Check if index exists
SELECT indexname, indexdef
FROM pg_indexes
WHERE tablename = 'orders' 
  AND indexdef LIKE '%order_date%';
-- Result: No index on order_date

-- Step 4: Create index
CREATE INDEX CONCURRENTLY idx_orders_order_date 
ON orders(order_date);

-- Step 5: Re-analyze query
EXPLAIN (ANALYZE, BUFFERS)
SELECT ... (same query)
-- Result: Index scan, query now takes 0.15 seconds (23x faster!)

-- Step 6: Monitor improvement
SELECT 
    calls,
    mean_exec_time,
    total_exec_time
FROM pg_stat_statements
WHERE query LIKE '%dashboard%';
-- Result: Mean time dropped from 3500ms to 150ms
```

---

### **10. Best Practices for Query Analysis**

1. **Start with pg_stat_statements** - Identify which queries consume the most resources
2. **Use EXPLAIN ANALYZE** - Not just EXPLAIN, to see actual vs estimated rows
3. **Check BUFFERS** - Understand I/O patterns (cache hits vs disk reads)
4. **Monitor over time** - Single snapshots can be misleading; track trends
5. **Test with production data volumes** - Development data often hides problems
6. **Analyze during peak load** - Performance issues often appear under load
7. **Use visualization tools** - Complex plans are easier to understand visually
8. **Set up alerting** - Automated monitoring for query degradation
9. **Reset statistics periodically** - `SELECT pg_stat_statements_reset();` for clean baselines
10. **Document findings** - Keep track of optimizations and their impact

---

### **Interview Tips:**
- **Demonstrate systematic approach**: Don't just mention EXPLAIN - show you understand the full analysis workflow
- **Mention multiple tools**: pg_stat_statements, pg_stat_activity, EXPLAIN ANALYZE, buffer analysis
- **Real numbers**: Quote realistic performance improvements (e.g., "reduced query time from 3.5s to 150ms")
- **Know the difference**: EXPLAIN (estimated) vs EXPLAIN ANALYZE (actual execution)
- **Buffer cache**: Understanding `shared_blks_hit` vs `shared_blks_read` shows depth
- **Wait events**: Knowing to check for blocking queries demonstrates production experience
- **Proactive monitoring**: Mention setting up auto_explain and pg_stat_statements for ongoing analysis

</details>

<details>
<summary><strong>42. What is EXPLAIN and EXPLAIN ANALYZE?</strong></summary>

### **Answer:**

`EXPLAIN` and `EXPLAIN ANALYZE` are PostgreSQL's primary tools for understanding how queries are executed. They reveal the query execution plan, helping you identify performance bottlenecks and optimization opportunities.

---

### **1. EXPLAIN - Query Plan Without Execution**

`EXPLAIN` shows the query planner's execution plan **without actually running the query**.

#### **Basic Syntax:**
```sql
EXPLAIN
SELECT * FROM orders WHERE customer_id = 123;
```

**Output:**
```
Seq Scan on orders  (cost=0.00..1892.50 rows=25 width=180)
  Filter: (customer_id = 123)
```

#### **Understanding the Output:**

- **Operation Type**: `Seq Scan` (Sequential Scan) - reading entire table
- **cost=0.00..1892.50**: 
  - First number (0.00): Startup cost (time before first row)
  - Second number (1892.50): Total cost to retrieve all rows
  - Units are arbitrary but consistent for comparison
- **rows=25**: Estimated number of rows returned
- **width=180**: Average row size in bytes

---

### **2. EXPLAIN ANALYZE - Actual Execution**

`EXPLAIN ANALYZE` **executes the query** and shows both estimated and **actual** execution statistics.

```sql
EXPLAIN ANALYZE
SELECT * FROM orders WHERE customer_id = 123;
```

**Output:**
```
Seq Scan on orders  (cost=0.00..1892.50 rows=25 width=180) 
                    (actual time=0.045..23.456 rows=32 loops=1)
  Filter: (customer_id = 123)
  Rows Removed by Filter: 99968
Planning Time: 0.234 ms
Execution Time: 23.567 ms
```

#### **Key Differences from EXPLAIN:**

- **actual time=0.045..23.456**: Real execution time in milliseconds
  - 0.045ms: Time to first row
  - 23.456ms: Time to retrieve all rows
- **rows=32**: **Actual** rows returned (estimated was 25 - 28% off!)
- **loops=1**: Number of times this node was executed
- **Rows Removed by Filter**: Rows scanned but filtered out (100,000 total - 32 returned = 99,968 removed)
- **Planning Time**: Time spent creating the execution plan
- **Execution Time**: Actual query execution time

---

### **3. EXPLAIN Options**

#### **A. FORMAT Options**

```sql
-- Text format (default)
EXPLAIN SELECT * FROM orders;

-- JSON format (for programmatic parsing)
EXPLAIN (FORMAT JSON)
SELECT * FROM orders;

-- YAML format
EXPLAIN (FORMAT YAML)
SELECT * FROM orders;

-- XML format
EXPLAIN (FORMAT XML)
SELECT * FROM orders;
```

#### **B. Detailed Options**

```sql
-- VERBOSE: Show additional details like output columns
EXPLAIN (VERBOSE)
SELECT customer_id, order_date FROM orders;

-- COSTS: Show cost estimates (enabled by default)
EXPLAIN (COSTS true)
SELECT * FROM orders;

-- BUFFERS: Show buffer usage (cache hits vs disk reads)
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM orders WHERE customer_id = 123;
```

**Output with BUFFERS:**
```
Seq Scan on orders  (cost=0.00..1892.50 rows=25 width=180)
                    (actual time=0.045..23.456 rows=32 loops=1)
  Filter: (customer_id = 123)
  Rows Removed by Filter: 99968
  Buffers: shared hit=892 read=156
Planning Time: 0.234 ms
Execution Time: 23.567 ms
```

- **shared hit=892**: Pages found in PostgreSQL's buffer cache (fast)
- **read=156**: Pages read from disk (slow)
- **Cache hit ratio**: 892 / (892 + 156) = 85.1%

#### **C. TIMING Option**

```sql
-- TIMING: Show actual timing (enabled by default with ANALYZE)
EXPLAIN (ANALYZE, TIMING true)
SELECT * FROM orders;

-- Disable timing to reduce overhead (useful for many-row queries)
EXPLAIN (ANALYZE, TIMING false)
SELECT * FROM orders;
```

#### **D. SUMMARY Option**

```sql
-- SUMMARY: Show planning and execution time summary
EXPLAIN (ANALYZE, SUMMARY)
SELECT * FROM orders;
```

#### **E. Comprehensive Analysis**

```sql
-- All options combined for maximum detail
EXPLAIN (
    ANALYZE,
    VERBOSE,
    BUFFERS,
    TIMING,
    COSTS,
    SUMMARY,
    FORMAT JSON
)
SELECT c.customer_name, o.order_date, o.total_amount
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_date > CURRENT_DATE - INTERVAL '30 days'
ORDER BY o.total_amount DESC
LIMIT 100;
```

---

### **4. Common Query Plan Nodes**

#### **A. Scan Types**

**Sequential Scan (Seq Scan):**
```sql
EXPLAIN ANALYZE
SELECT * FROM orders WHERE status = 'pending';
```
```
Seq Scan on orders  (cost=0.00..1892.50 rows=1250 width=180)
                    (actual time=0.015..25.234 rows=1304 loops=1)
  Filter: (status = 'pending'::text)
  Rows Removed by Filter: 98696
```
- Reads entire table sequentially
- Used when no index exists or index scan would be more expensive

**Index Scan:**
```sql
CREATE INDEX idx_orders_customer_id ON orders(customer_id);

EXPLAIN ANALYZE
SELECT * FROM orders WHERE customer_id = 123;
```
```
Index Scan using idx_orders_customer_id on orders
    (cost=0.42..35.67 rows=25 width=180)
    (actual time=0.023..0.156 rows=32 loops=1)
  Index Cond: (customer_id = 123)
```
- Uses index to find specific rows
- Much faster than sequential scan for selective queries

**Index Only Scan:**
```sql
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);

EXPLAIN ANALYZE
SELECT customer_id, order_date FROM orders WHERE customer_id = 123;
```
```
Index Only Scan using idx_orders_customer_date on orders
    (cost=0.42..12.45 rows=25 width=12)
    (actual time=0.015..0.078 rows=32 loops=1)
  Index Cond: (customer_id = 123)
  Heap Fetches: 0
```
- Retrieves all data from index without accessing table
- **Heap Fetches: 0** means no table access needed (very fast!)

**Bitmap Index Scan:**
```sql
EXPLAIN ANALYZE
SELECT * FROM orders 
WHERE customer_id IN (123, 456, 789);
```
```
Bitmap Heap Scan on orders  (cost=45.67..892.34 rows=75 width=180)
                            (actual time=0.234..1.456 rows=89 loops=1)
  Recheck Cond: (customer_id = ANY ('{123,456,789}'::integer[]))
  Heap Blocks: exact=67
  ->  Bitmap Index Scan on idx_orders_customer_id
          (cost=0.00..45.65 rows=75 width=0)
          (actual time=0.156..0.156 rows=89 loops=1)
        Index Cond: (customer_id = ANY ('{123,456,789}'::integer[]))
```
- Builds bitmap of matching rows from index, then fetches from table
- Efficient for fetching multiple scattered rows

#### **B. Join Types**

**Nested Loop Join:**
```sql
EXPLAIN ANALYZE
SELECT * FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE c.customer_id = 123;
```
```
Nested Loop  (cost=0.84..78.92 rows=25 width=360)
             (actual time=0.034..0.234 rows=32 loops=1)
  ->  Index Scan using customers_pkey on customers c
          (cost=0.42..8.44 rows=1 width=180)
          (actual time=0.015..0.016 rows=1 loops=1)
        Index Cond: (customer_id = 123)
  ->  Index Scan using idx_orders_customer_id on orders o
          (cost=0.42..70.23 rows=25 width=180)
          (actual time=0.018..0.203 rows=32 loops=1)
        Index Cond: (customer_id = 123)
```
- Loops through one table and looks up matches in the other
- Efficient when one side returns few rows

**Hash Join:**
```sql
EXPLAIN ANALYZE
SELECT c.customer_name, COUNT(*) as order_count
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;
```
```
HashAggregate  (cost=5234.56..5456.78 rows=18500 width=48)
               (actual time=234.567..245.678 rows=18234 loops=1)
  Group Key: c.customer_id, c.customer_name
  ->  Hash Join  (cost=850.00..4567.89 rows=100000 width=40)
                 (actual time=12.345..189.456 rows=100000 loops=1)
        Hash Cond: (o.customer_id = c.customer_id)
        ->  Seq Scan on orders o  (cost=0.00..3200.00 rows=100000 width=20)
                                  (actual time=0.012..67.890 rows=100000 loops=1)
        ->  Hash  (cost=600.00..600.00 rows=20000 width=36)
                  (actual time=12.234..12.234 rows=20000 loops=1)
              Buckets: 32768  Batches: 1  Memory Usage: 1456kB
              ->  Seq Scan on customers c  (cost=0.00..600.00 rows=20000 width=36)
                                          (actual time=0.008..6.123 rows=20000 loops=1)
```
- Builds hash table from smaller table, probes with larger table
- Efficient for large datasets

**Merge Join:**
```sql
EXPLAIN ANALYZE
SELECT * FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
ORDER BY o.order_id;
```
```
Merge Join  (cost=0.84..7823.45 rows=150000 width=280)
            (actual time=0.045..345.678 rows=150000 loops=1)
  Merge Cond: (o.order_id = oi.order_id)
  ->  Index Scan using orders_pkey on orders o
          (cost=0.42..3456.78 rows=100000 width=180)
          (actual time=0.023..123.456 rows=100000 loops=1)
  ->  Index Scan using order_items_order_id_idx on order_items oi
          (cost=0.42..3678.90 rows=150000 width=100)
          (actual time=0.018..156.789 rows=150000 loops=1)
```
- Both inputs must be sorted on join key
- Efficient when data is already sorted or has indexes

#### **C. Aggregation Nodes**

**HashAggregate:**
```sql
EXPLAIN ANALYZE
SELECT customer_id, COUNT(*), SUM(total_amount)
FROM orders
GROUP BY customer_id;
```
```
HashAggregate  (cost=2850.00..3050.00 rows=20000 width=40)
               (actual time=145.234..167.890 rows=18234 loops=1)
  Group Key: customer_id
  ->  Seq Scan on orders  (cost=0.00..2200.00 rows=100000 width=20)
                          (actual time=0.015..78.456 rows=100000 loops=1)
```

**GroupAggregate:**
```sql
EXPLAIN ANALYZE
SELECT customer_id, COUNT(*), SUM(total_amount)
FROM orders
GROUP BY customer_id
ORDER BY customer_id;  -- Already sorted
```
```
GroupAggregate  (cost=0.42..4567.89 rows=20000 width=40)
                (actual time=0.034..234.567 rows=18234 loops=1)
  Group Key: customer_id
  ->  Index Scan using idx_orders_customer_id on orders
          (cost=0.42..3200.00 rows=100000 width=20)
          (actual time=0.023..145.678 rows=100000 loops=1)
```

#### **D. Sort Operations**

```sql
EXPLAIN ANALYZE
SELECT * FROM orders
ORDER BY total_amount DESC
LIMIT 100;
```
```
Limit  (cost=8934.56..8934.81 rows=100 width=180)
       (actual time=234.567..234.623 rows=100 loops=1)
  ->  Sort  (cost=8934.56..9184.56 rows=100000 width=180)
            (actual time=234.565..234.598 rows=100 loops=1)
        Sort Key: total_amount DESC
        Sort Method: top-N heapsort  Memory: 89kB
        ->  Seq Scan on orders  (cost=0.00..2200.00 rows=100000 width=180)
                                (actual time=0.015..123.456 rows=100000 loops=1)
```

**Sort Methods:**
- **quicksort** - In-memory sort
- **top-N heapsort** - When using LIMIT, more efficient
- **external merge** - When data doesn't fit in memory (spills to disk)

---

### **5. Key Metrics to Watch**

#### **A. Estimated vs Actual Rows**

```sql
EXPLAIN ANALYZE
SELECT * FROM orders WHERE status = 'rare_status';
```
```
Seq Scan on orders  (cost=0.00..1892.50 rows=5000 width=180)
                    (actual time=0.015..25.234 rows=3 loops=1)
  Filter: (status = 'rare_status'::text)
  Rows Removed by Filter: 99997
```

- **rows=5000** (estimated) vs **rows=3** (actual) - **1667x overestimate!**
- Such discrepancies indicate stale statistics → Run `ANALYZE orders;`

#### **B. Buffer Cache Efficiency**

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM orders WHERE customer_id = 123;
```
```
Index Scan using idx_orders_customer_id on orders
    (cost=0.42..35.67 rows=25 width=180)
    (actual time=0.023..0.156 rows=32 loops=1)
  Index Cond: (customer_id = 123)
  Buffers: shared hit=8 read=2
```

- **hit=8, read=2** → 80% cache hit ratio (good)
- **hit=0, read=100** → 0% cache hit ratio (poor, cold cache or insufficient memory)

#### **C. Execution Time Breakdown**

```sql
EXPLAIN (ANALYZE, TIMING, SUMMARY)
SELECT ...;
```
```
Planning Time: 2.345 ms
Execution Time: 456.789 ms
```

- **Planning Time > 10ms** → Complex query or many tables
- **Execution Time >> Planning Time** → Normal
- **Planning Time > Execution Time** → Consider prepared statements

---

### **6. Real-World Optimization Examples**

#### **Example 1: Missing Index Detection**

**Before:**
```sql
EXPLAIN ANALYZE
SELECT * FROM orders 
WHERE order_date > '2024-01-01' 
  AND status = 'pending';
```
```
Seq Scan on orders  (cost=0.00..2500.00 rows=1000 width=180)
                    (actual time=0.234..345.678 rows=1204 loops=1)
  Filter: ((order_date > '2024-01-01') AND (status = 'pending'))
  Rows Removed by Filter: 98796
Execution Time: 345.890 ms
```

**After adding index:**
```sql
CREATE INDEX idx_orders_date_status ON orders(order_date, status);

EXPLAIN ANALYZE
SELECT * FROM orders 
WHERE order_date > '2024-01-01' 
  AND status = 'pending';
```
```
Index Scan using idx_orders_date_status on orders
    (cost=0.42..156.78 rows=1000 width=180)
    (actual time=0.034..5.678 rows=1204 loops=1)
  Index Cond: ((order_date > '2024-01-01') AND (status = 'pending'))
Execution Time: 5.789 ms
```

**Result: 345.89ms → 5.79ms = 60x faster!**

#### **Example 2: Join Order Optimization**

**Before (inefficient join order):**
```sql
EXPLAIN ANALYZE
SELECT o.order_id, c.customer_name, p.product_name
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN products p ON o.product_id = p.product_id
WHERE o.order_date = CURRENT_DATE;
```
```
Hash Join  (cost=5678.90..12345.67 rows=50 width=120)
           (actual time=234.567..456.789 rows=47 loops=1)
  Hash Cond: (p.product_id = o.product_id)
  ->  Seq Scan on products p  (cost=0.00..890.00 rows=50000 width=60)
                               (actual time=0.012..123.456 rows=50000 loops=1)
  ->  Hash  (cost=5234.56..5234.56 rows=50 width=60)
            (actual time=234.123..234.123 rows=47 loops=1)
        ->  Hash Join  (cost=1234.56..5234.56 rows=50 width=60)
                      (actual time=12.345..234.098 rows=47 loops=1)
              Hash Cond: (o.customer_id = c.customer_id)
              ->  Seq Scan on orders o  (cost=0.00..3456.78 rows=50 width=20)
                                        (actual time=0.023..189.456 rows=47 loops=1)
                    Filter: (order_date = CURRENT_DATE)
                    Rows Removed by Filter: 99953
              ->  Hash  (cost=890.00..890.00 rows=20000 width=48)
                        (actual time=12.234..12.234 rows=20000 loops=1)
                    ->  Seq Scan on customers c  (cost=0.00..890.00 rows=20000 width=48)
                                                 (actual time=0.008..6.123 rows=20000 loops=1)
Execution Time: 456.890 ms
```

**After adding index and optimizing:**
```sql
CREATE INDEX idx_orders_date ON orders(order_date);

EXPLAIN ANALYZE
SELECT o.order_id, c.customer_name, p.product_name
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN products p ON o.product_id = p.product_id
WHERE o.order_date = CURRENT_DATE;
```
```
Nested Loop  (cost=1.26..234.56 rows=50 width=120)
             (actual time=0.045..2.345 rows=47 loops=1)
  ->  Nested Loop  (cost=0.84..123.45 rows=50 width=60)
                   (actual time=0.034..1.234 rows=47 loops=1)
        ->  Index Scan using idx_orders_date on orders o
                (cost=0.42..56.78 rows=50 width=20)
                (actual time=0.023..0.345 rows=47 loops=1)
              Index Cond: (order_date = CURRENT_DATE)
        ->  Index Scan using customers_pkey on customers c
                (cost=0.42..1.33 rows=1 width=48)
                (actual time=0.015..0.016 rows=1 loops=47)
              Index Cond: (customer_id = o.customer_id)
  ->  Index Scan using products_pkey on products p
          (cost=0.42..2.22 rows=1 width=60)
          (actual time=0.020..0.021 rows=1 loops=47)
        Index Cond: (product_id = o.product_id)
Execution Time: 2.456 ms
```

**Result: 456.89ms → 2.46ms = 186x faster!**

---

### **7. Best Practices**

1. **Use EXPLAIN ANALYZE, not just EXPLAIN** - See actual performance, not just estimates
2. **Check estimated vs actual rows** - Large discrepancies mean stale statistics
3. **Enable BUFFERS** - Understand cache vs disk I/O
4. **Look for sequential scans** - Often indicate missing indexes
5. **Watch for loops** - High loop counts multiply node costs
6. **Compare costs** - Higher cost usually means slower (but check actual time)
7. **Test with production data volumes** - Plans change with data size
8. **Use visualization tools** - explain.depesz.com, explain.dalibo.com
9. **Save plans** - Compare before/after optimization
10. **Be cautious with EXPLAIN ANALYZE** - It executes the query (including writes!)

---

### **Interview Tips:**
- **Know the difference**: EXPLAIN (plan only) vs EXPLAIN ANALYZE (actual execution)
- **Explain cost format**: `cost=startup..total` - what each number means
- **Buffer metrics**: `shared hit` (cache) vs `read` (disk) shows I/O efficiency
- **Actual vs estimated**: Large gaps indicate statistics problems → need ANALYZE
- **Practical example**: "Used EXPLAIN ANALYZE to identify seq scan, added index, reduced time from 500ms to 10ms"
- **Common pitfall**: "EXPLAIN ANALYZE executes the query, so be careful with UPDATE/DELETE statements"
- **Loops matter**: A node with cost=10 but loops=1000 is actually cost=10,000

</details>

<details>
<summary><strong>43. What is query planning in PostgreSQL?</strong></summary>

### **Answer:**

Query planning is the process by which PostgreSQL's query planner (also called query optimizer) transforms a SQL query into an optimal execution plan. The planner analyzes the query, examines available indexes and statistics, estimates costs, and chooses the most efficient way to execute the query.

---

### **1. Query Planning Process**

#### **Step-by-Step Flow:**

```
SQL Query
    ↓
1. Parser (Syntax Check)
    ↓
2. Rewriter (View/Rule Expansion)
    ↓
3. Planner/Optimizer (Generate Execution Plans)
    ↓
4. Executor (Execute Chosen Plan)
    ↓
Result
```

**Example Query:**
```sql
SELECT c.customer_name, SUM(o.total_amount) as total_spent
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_date >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY c.customer_id, c.customer_name
HAVING SUM(o.total_amount) > 1000
ORDER BY total_spent DESC
LIMIT 10;
```

**What the Planner Does:**
1. **Parse** - Verify SQL syntax is valid
2. **Rewrite** - Apply any rules or view expansions
3. **Plan** - Generate multiple possible execution plans:
   - Should it use an index on `order_date`?
   - What join algorithm? (Nested Loop, Hash Join, Merge Join)
   - Should it filter before or after joining?
   - How to perform aggregation? (HashAggregate vs GroupAggregate)
4. **Cost Estimation** - Calculate estimated cost for each plan
5. **Choose** - Select the plan with lowest estimated cost
6. **Execute** - Run the chosen plan

---

### **2. Cost-Based Optimization**

PostgreSQL uses **cost-based optimization** - it estimates the "cost" of different execution strategies and chooses the cheapest.

#### **Cost Components:**

```sql
-- See costs in EXPLAIN output
EXPLAIN
SELECT * FROM orders WHERE customer_id = 123;
```
```
Index Scan using idx_orders_customer_id on orders
    (cost=0.42..35.67 rows=25 width=180)
```

**Cost Formula:**
- **Startup Cost (0.42)**: Cost before first row can be returned
- **Total Cost (35.67)**: Cost to retrieve all rows
- **Cost = (pages_read × seq_page_cost) + (rows_scanned × cpu_tuple_cost) + (index_pages × random_page_cost) + ...**

#### **Cost Parameters (from postgresql.conf):**

```sql
-- View current cost parameters
SHOW seq_page_cost;      -- Default: 1.0 (sequential page read)
SHOW random_page_cost;   -- Default: 4.0 (random page read from disk)
SHOW cpu_tuple_cost;     -- Default: 0.01 (processing one row)
SHOW cpu_index_tuple_cost;  -- Default: 0.005 (processing one index entry)
SHOW cpu_operator_cost;  -- Default: 0.0025 (executing an operator/function)
```

**Example Cost Calculation:**
```sql
-- Sequential scan cost
-- (1000 pages × 1.0) + (100000 rows × 0.01) = 1000 + 1000 = 2000

-- Index scan cost
-- (50 index pages × 4.0) + (25 result pages × 4.0) + (25 rows × 0.01) = 200 + 100 + 0.25 = 300.25
```

---

### **3. Statistics and Cardinality Estimation**

The planner relies on **statistics** to estimate row counts and make decisions.

#### **A. Table Statistics**

```sql
-- Update statistics for accurate planning
ANALYZE customers;
ANALYZE orders;

-- View statistics
SELECT 
    schemaname,
    tablename,
    n_live_tup,
    n_dead_tup,
    last_analyze,
    last_autoanalyze
FROM pg_stat_user_tables;
```

#### **B. Column Statistics**

```sql
-- Detailed column statistics
SELECT 
    attname,
    n_distinct,
    most_common_vals,
    most_common_freqs,
    histogram_bounds
FROM pg_stats
WHERE tablename = 'orders'
  AND attname = 'status';
```

**Example Output:**
```
attname: status
n_distinct: 5 (5 distinct values)
most_common_vals: {pending, shipped, delivered, cancelled, refunded}
most_common_freqs: {0.15, 0.50, 0.30, 0.03, 0.02}
```

**How Planner Uses This:**
```sql
-- Planner estimates: 30% of rows have status='delivered'
SELECT * FROM orders WHERE status = 'delivered';
-- Estimated rows = 100000 * 0.30 = 30000 rows
```

#### **C. Stale Statistics Problem**

**Before ANALYZE:**
```sql
EXPLAIN ANALYZE
SELECT * FROM orders WHERE status = 'rare_status';
```
```
Seq Scan on orders  (cost=0.00..1892.50 rows=50000 width=180)
                    (actual time=0.015..25.234 rows=3 loops=1)
```
- Planner estimated **50,000 rows** but actual is **3 rows** - 16,667x overestimate!
- This leads to poor plan choices (e.g., hash join instead of nested loop)

**After ANALYZE:**
```sql
ANALYZE orders;

EXPLAIN ANALYZE
SELECT * FROM orders WHERE status = 'rare_status';
```
```
Seq Scan on orders  (cost=0.00..1892.50 rows=5 width=180)
                    (actual time=0.015..25.234 rows=3 loops=1)
```
- Now estimates **5 rows**, much closer to actual **3 rows**

---

### **4. Join Planning**

The planner evaluates multiple join strategies and orders.

#### **A. Join Algorithms**

**Nested Loop Join:**
```sql
EXPLAIN
SELECT * FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE c.customer_id = 123;
```
```
Nested Loop  (cost=0.84..78.92 rows=25 width=360)
  ->  Index Scan using customers_pkey on customers c
          (cost=0.42..8.44 rows=1 width=180)
        Index Cond: (customer_id = 123)
  ->  Index Scan using idx_orders_customer_id on orders o
          (cost=0.42..70.23 rows=25 width=180)
        Index Cond: (customer_id = c.customer_id)
```
- **Best for**: Small outer table (1 customer)
- **Cost**: O(n × m) where n=1, m=25 → 25 lookups

**Hash Join:**
```sql
EXPLAIN
SELECT c.customer_name, COUNT(*) 
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;
```
```
HashAggregate  (cost=5234.56..5456.78 rows=18500 width=48)
  ->  Hash Join  (cost=850.00..4567.89 rows=100000 width=40)
        Hash Cond: (o.customer_id = c.customer_id)
        ->  Seq Scan on orders o  (cost=0.00..3200.00 rows=100000 width=20)
        ->  Hash  (cost=600.00..600.00 rows=20000 width=36)
              Buckets: 32768  Batches: 1  Memory Usage: 1456kB
              ->  Seq Scan on customers c  (cost=0.00..600.00 rows=20000 width=36)
```
- **Best for**: Large tables, no useful indexes
- **Cost**: O(n + m) - build hash table + probe

**Merge Join:**
```sql
EXPLAIN
SELECT * FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
ORDER BY o.order_id;
```
```
Merge Join  (cost=0.84..7823.45 rows=150000 width=280)
  Merge Cond: (o.order_id = oi.order_id)
  ->  Index Scan using orders_pkey on orders o
          (cost=0.42..3456.78 rows=100000 width=180)
  ->  Index Scan using order_items_order_id_idx on order_items oi
          (cost=0.42..3678.90 rows=150000 width=100)
```
- **Best for**: Both inputs already sorted (indexed)
- **Cost**: O(n + m) with pre-sorted data

#### **B. Join Order Optimization**

**For query:** `SELECT * FROM A JOIN B JOIN C`

**Possible join orders:**
1. `(A JOIN B) JOIN C`
2. `(A JOIN C) JOIN B`
3. `(B JOIN C) JOIN A`
4. `A JOIN (B JOIN C)`
5. `B JOIN (A JOIN C)`
6. `C JOIN (A JOIN B)`

**Complexity**: For N tables, there are **O(N!)** possible join orders!
- 3 tables: 12 possible plans
- 4 tables: 120 possible plans
- 10 tables: 3,628,800 possible plans!

**PostgreSQL's Approach:**
```sql
-- Controls join planning exhaustiveness
SHOW join_collapse_limit;  -- Default: 8
SHOW from_collapse_limit;  -- Default: 8
```

- For ≤8 tables: Exhaustive search (considers all orders)
- For >8 tables: Genetic Query Optimizer (GEQO) - heuristic search

**Example:**
```sql
SET join_collapse_limit = 1;  -- Force left-to-right join order

EXPLAIN
SELECT * FROM a JOIN b ON a.id = b.a_id
            JOIN c ON b.id = c.b_id;
-- Will join A→B first, then result→C

SET join_collapse_limit = 8;  -- Re-enable optimization

EXPLAIN
SELECT * FROM a JOIN b ON a.id = b.a_id
            JOIN c ON b.id = c.b_id;
-- Planner may choose B→C first if more efficient
```

---

### **5. Scan Methods**

The planner chooses between different scan methods based on selectivity.

#### **A. Sequential Scan vs Index Scan**

**When Sequential Scan is Chosen:**
```sql
-- Query returns large percentage of table
EXPLAIN
SELECT * FROM orders WHERE order_date >= '2020-01-01';
-- Returns 80% of rows
```
```
Seq Scan on orders  (cost=0.00..2200.00 rows=80000 width=180)
  Filter: (order_date >= '2020-01-01')
```
- **Why**: Reading 80% of table → Sequential scan is faster than 80,000 random index lookups

**When Index Scan is Chosen:**
```sql
-- Query returns small percentage of table
EXPLAIN
SELECT * FROM orders WHERE order_date >= '2024-12-01';
-- Returns 2% of rows
```
```
Index Scan using idx_orders_date on orders
    (cost=0.42..235.67 rows=2000 width=180)
  Index Cond: (order_date >= '2024-12-01')
```
- **Why**: Reading 2% of table → Index scan is faster (2,000 random lookups < full scan)

**Crossover Point:**
- Typically around **5-10% of table**
- Depends on `random_page_cost` setting

#### **B. Forcing Scan Methods (for testing)**

```sql
-- Disable sequential scans (force index use)
SET enable_seqscan = off;

EXPLAIN
SELECT * FROM orders WHERE status = 'pending';
-- Will use index even if seq scan would be faster

-- Disable index scans (force sequential)
SET enable_indexscan = off;

-- Disable bitmap scans
SET enable_bitmapscan = off;

-- Re-enable all (default)
SET enable_seqscan = on;
SET enable_indexscan = on;
SET enable_bitmapscan = on;
```

---

### **6. Planner Configuration Parameters**

#### **A. Key Parameters**

```sql
-- Cost parameters
SHOW seq_page_cost;         -- Sequential I/O cost: 1.0
SHOW random_page_cost;      -- Random I/O cost: 4.0
SHOW cpu_tuple_cost;        -- Row processing cost: 0.01
SHOW effective_cache_size;  -- Available cache: 4GB (default)

-- Join/sort memory
SHOW work_mem;              -- Memory for sorts/hashes: 4MB (default)
SHOW shared_buffers;        -- Shared buffer cache: 128MB (default)

-- Planner behavior
SHOW default_statistics_target;  -- Statistics detail level: 100
SHOW from_collapse_limit;        -- Join reordering limit: 8
SHOW join_collapse_limit;        -- Join reordering limit: 8
```

#### **B. Tuning for SSDs**

```sql
-- On SSD, random access is nearly as fast as sequential
-- Reduce random_page_cost to favor index scans
ALTER SYSTEM SET random_page_cost = 1.1;  -- Default is 4.0
SELECT pg_reload_conf();

-- This makes planner more likely to choose index scans
```

#### **C. Increasing Statistics**

```sql
-- More detailed statistics for better estimates
ALTER TABLE orders ALTER COLUMN status SET STATISTICS 1000;
-- Default is 100; max is 10000

ANALYZE orders;

-- Check impact
SELECT attname, n_distinct, array_length(most_common_vals, 1) as mcv_count
FROM pg_stats
WHERE tablename = 'orders' AND attname = 'status';
```

---

### **7. Genetic Query Optimizer (GEQO)**

For queries with many joins, exhaustive search is too expensive.

```sql
-- Enable GEQO for queries with >8 tables
SHOW geqo;                    -- Default: on
SHOW geqo_threshold;          -- Default: 12 tables

-- GEQO uses genetic algorithm to find "good enough" plan
-- Not guaranteed optimal, but fast planning
```

**Example with 15 tables:**
```sql
SELECT * FROM t1
JOIN t2 ON t1.id = t2.t1_id
JOIN t3 ON t2.id = t3.t2_id
-- ... (15 tables total)
```
- Without GEQO: Would need to evaluate trillions of plans (impossible)
- With GEQO: Finds good plan in reasonable time using evolutionary algorithm

---

### **8. Plan Caching**

#### **A. Prepared Statements**

```sql
-- Parse and plan once, execute many times
PREPARE get_customer_orders (int) AS
    SELECT * FROM orders WHERE customer_id = $1;

-- First execution: plans for customer_id = 123
EXECUTE get_customer_orders(123);

-- Subsequent executions: reuse plan
EXECUTE get_customer_orders(456);
EXECUTE get_customer_orders(789);

-- View prepared statements
SELECT name, statement, parameter_types
FROM pg_prepared_statements;
```

**Generic vs Custom Plans:**
```sql
-- PostgreSQL chooses between:
-- 1. Generic plan: One plan for all parameters
-- 2. Custom plan: New plan for each parameter value

-- After 5 executions, PostgreSQL compares:
-- - If generic plan cost < average custom plan cost: Use generic
-- - Otherwise: Continue with custom plans

-- Force custom plans always
SET plan_cache_mode = 'force_custom_plan';

-- Force generic plans always
SET plan_cache_mode = 'force_generic_plan';

-- Auto (default)
SET plan_cache_mode = 'auto';
```

---

### **9. Viewing and Debugging Plans**

#### **A. pg_stat_statements**

```sql
-- See which queries are planned most often
SELECT 
    query,
    calls,
    total_exec_time,
    mean_exec_time,
    total_plan_time,
    mean_plan_time
FROM pg_stat_statements
ORDER BY total_plan_time DESC
LIMIT 10;
```

#### **B. auto_explain**

```sql
-- Automatically log plans for slow queries
LOAD 'auto_explain';
SET auto_explain.log_min_duration = 1000;  -- Log queries >1 second
SET auto_explain.log_analyze = true;
SET auto_explain.log_buffers = true;

-- Now slow queries are automatically logged with plans
```

---

### **10. Real-World Planning Example**

**Scenario: Query gets slower over time**

```sql
-- Initially fast
SELECT * FROM orders 
WHERE status = 'pending' 
  AND created_date > CURRENT_DATE - INTERVAL '7 days';
-- Execution time: 5ms

-- 3 months later: Same query, now slow
SELECT * FROM orders 
WHERE status = 'pending' 
  AND created_date > CURRENT_DATE - INTERVAL '7 days';
-- Execution time: 2500ms
```

**Investigation:**
```sql
-- Check plan
EXPLAIN ANALYZE
SELECT * FROM orders 
WHERE status = 'pending' 
  AND created_date > CURRENT_DATE - INTERVAL '7 days';
```

**Before ANALYZE:**
```
Seq Scan on orders  (cost=0.00..15000.00 rows=50 width=180)
                    (actual time=0.234..2456.789 rows=75000 loops=1)
  Filter: ((status = 'pending') AND (created_date > ...))
  Rows Removed by Filter: 925000
```
- Planner estimated **50 rows**, actual is **75,000 rows** (1500x underestimate!)
- Chose sequential scan, which is wrong for this query

**After ANALYZE:**
```sql
ANALYZE orders;

EXPLAIN ANALYZE
SELECT * FROM orders 
WHERE status = 'pending' 
  AND created_date > CURRENT_DATE - INTERVAL '7 days';
```
```
Bitmap Heap Scan on orders  (cost=1234.56..8765.43 rows=75000 width=180)
                            (actual time=12.345..156.789 rows=75123 loops=1)
  Recheck Cond: ((status = 'pending') AND (created_date > ...))
  Heap Blocks: exact=5432
  ->  Bitmap Index Scan on idx_orders_status_date
          (cost=0.00..1215.81 rows=75000 width=0)
          (actual time=10.234..10.234 rows=75123 loops=1)
        Index Cond: ((status = 'pending') AND (created_date > ...))
```
- Now estimates **75,000 rows** (accurate!)
- Uses bitmap index scan
- **Result: 2500ms → 157ms = 16x faster!**

---

### **11. Best Practices**

1. **Keep statistics up-to-date** - Run ANALYZE regularly (autovacuum does this)
2. **Monitor plan quality** - Use pg_stat_statements to track performance
3. **Understand query costs** - Higher cost usually means slower
4. **Check estimated vs actual rows** - Large gaps indicate planning problems
5. **Consider prepared statements** - Reduce planning overhead for repeated queries
6. **Tune cost parameters** - Adjust random_page_cost for SSDs
7. **Increase work_mem** - Helps complex sorts and joins
8. **Use indexes wisely** - But don't over-index (hurts writes)
9. **Test with production data** - Plans change with data distribution
10. **Monitor planning time** - If high, consider prepared statements or simplifying query

---

### **Interview Tips:**
- **Define clearly**: "Query planning is the process where PostgreSQL's optimizer evaluates multiple execution strategies and chooses the one with lowest estimated cost"
- **Cost-based**: Emphasize PostgreSQL uses cost estimates, not rule-based optimization
- **Statistics dependency**: "Planner relies on table/column statistics from ANALYZE; stale stats lead to poor plans"
- **Join planning**: Know the three join types (Nested Loop, Hash, Merge) and when each is used
- **Real example**: "Once debugged a slow query that was 1500x off in row estimation due to stale statistics. After ANALYZE, query went from 2.5s to 150ms"
- **Configuration**: Mention random_page_cost, work_mem, effective_cache_size
- **Complexity**: "For N tables, there are N! possible join orders; PostgreSQL limits exhaustive search to 8 tables, then uses GEQO"

</details>

<details>
<summary><strong>44. How do you optimize slow queries?</strong></summary>

### **Answer:**

Optimizing slow queries requires a systematic approach combining analysis, indexing, query rewriting, and configuration tuning. Here's a comprehensive guide to identifying and fixing performance bottlenecks.

---

### **1. Systematic Optimization Workflow**

#### **Step 1: Identify Slow Queries**

**A. Using pg_stat_statements:**
```sql
-- Enable extension
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- Find slowest queries by total time
SELECT 
    query,
    calls,
    total_exec_time,
    mean_exec_time,
    max_exec_time,
    stddev_exec_time,
    rows
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 20;
```

**Output:**
```
query                                  | calls | total_exec_time | mean_exec_time | max_exec_time
--------------------------------------+-------+-----------------+----------------+--------------
SELECT * FROM orders WHERE status=$1  | 15234 |   145234.567    |     9.534      |   1234.567
UPDATE inventory SET quantity=$1...   |  8945 |    98765.432    |    11.045      |    567.890
```

**B. Find queries with high variance (unpredictable performance):**
```sql
SELECT 
    query,
    calls,
    mean_exec_time,
    stddev_exec_time,
    (stddev_exec_time / NULLIF(mean_exec_time, 0)) as variability_coefficient
FROM pg_stat_statements
WHERE calls > 100
ORDER BY variability_coefficient DESC
LIMIT 10;
```

**C. Monitor active queries:**
```sql
-- Find long-running queries
SELECT 
    pid,
    NOW() - query_start AS duration,
    state,
    query
FROM pg_stat_activity
WHERE state = 'active'
  AND (NOW() - query_start) > INTERVAL '1 minute'
ORDER BY duration DESC;
```

#### **Step 2: Analyze the Query Plan**

```sql
-- Get actual execution statistics
EXPLAIN (ANALYZE, BUFFERS, VERBOSE)
SELECT o.order_id, c.customer_name, p.product_name, o.total_amount
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN products p ON o.product_id = p.product_id
WHERE o.order_date >= CURRENT_DATE - INTERVAL '30 days'
  AND o.status = 'pending'
ORDER BY o.total_amount DESC
LIMIT 100;
```

**Look for red flags:**
- Sequential scans on large tables
- Large discrepancies between estimated and actual rows
- High buffer reads (disk I/O)
- Expensive sort operations
- Nested loops with many iterations
- Hash joins that spill to disk

---

### **2. Optimization Technique #1: Add Indexes**

#### **A. Single-Column Indexes**

**Problem: Sequential scan on filtered column**
```sql
EXPLAIN ANALYZE
SELECT * FROM orders WHERE status = 'pending';
```
```
Seq Scan on orders  (cost=0.00..2500.00 rows=5000 width=180)
                    (actual time=0.234..456.789 rows=5234 loops=1)
  Filter: (status = 'pending')
  Rows Removed by Filter: 94766
Execution Time: 456.890 ms
```

**Solution: Add index**
```sql
CREATE INDEX CONCURRENTLY idx_orders_status ON orders(status);

EXPLAIN ANALYZE
SELECT * FROM orders WHERE status = 'pending';
```
```
Bitmap Index Scan on idx_orders_status
    (cost=156.78..567.89 rows=5000 width=180)
    (actual time=2.345..15.678 rows=5234 loops=1)
  Index Cond: (status = 'pending')
Execution Time: 15.789 ms
```
**Result: 456ms → 16ms = 29x faster!**

#### **B. Multi-Column (Composite) Indexes**

**Problem: Query filters on multiple columns**
```sql
EXPLAIN ANALYZE
SELECT * FROM orders 
WHERE status = 'pending' 
  AND order_date >= '2024-01-01';
```
```
Seq Scan on orders  (cost=0.00..2500.00 rows=250 width=180)
                    (actual time=0.456..567.890 rows=267 loops=1)
  Filter: ((status = 'pending') AND (order_date >= '2024-01-01'))
  Rows Removed by Filter: 99733
Execution Time: 568.123 ms
```

**Solution: Composite index (order matters!)**
```sql
-- Put most selective column first
CREATE INDEX CONCURRENTLY idx_orders_status_date 
ON orders(status, order_date);

EXPLAIN ANALYZE
SELECT * FROM orders 
WHERE status = 'pending' 
  AND order_date >= '2024-01-01';
```
```
Index Scan using idx_orders_status_date on orders
    (cost=0.42..45.67 rows=250 width=180)
    (actual time=0.023..0.789 rows=267 loops=1)
  Index Cond: ((status = 'pending') AND (order_date >= '2024-01-01'))
Execution Time: 0.856 ms
```
**Result: 568ms → 0.86ms = 661x faster!**

#### **C. Partial Indexes**

**Problem: Only querying subset of data**
```sql
-- Only care about active orders
SELECT * FROM orders WHERE status = 'pending' AND active = true;
-- 'pending' AND 'active' is only 2% of all orders
```

**Solution: Partial index**
```sql
-- Smaller index, faster queries
CREATE INDEX CONCURRENTLY idx_orders_pending_active 
ON orders(order_date)
WHERE status = 'pending' AND active = true;

-- Index is 20x smaller, queries are 15x faster
```

#### **D. Expression Indexes**

**Problem: Querying on function result**
```sql
EXPLAIN ANALYZE
SELECT * FROM customers WHERE LOWER(email) = 'john@example.com';
```
```
Seq Scan on customers  (cost=0.00..1500.00 rows=100 width=180)
                       (actual time=0.123..234.567 rows=1 loops=1)
  Filter: (lower(email) = 'john@example.com')
  Rows Removed by Filter: 19999
Execution Time: 234.678 ms
```

**Solution: Expression index**
```sql
CREATE INDEX CONCURRENTLY idx_customers_lower_email 
ON customers(LOWER(email));

EXPLAIN ANALYZE
SELECT * FROM customers WHERE LOWER(email) = 'john@example.com';
```
```
Index Scan using idx_customers_lower_email on customers
    (cost=0.42..8.44 rows=1 width=180)
    (actual time=0.023..0.025 rows=1 loops=1)
  Index Cond: (lower(email) = 'john@example.com')
Execution Time: 0.034 ms
```
**Result: 235ms → 0.03ms = 7,833x faster!**

---

### **3. Optimization Technique #2: Query Rewriting**

#### **A. Avoid SELECT ***

**Before:**
```sql
-- Fetches all columns (180 bytes per row)
SELECT * FROM orders WHERE customer_id = 123;
-- Time: 45ms, Data transferred: 5.4MB (30,000 rows × 180 bytes)
```

**After:**
```sql
-- Fetch only needed columns (20 bytes per row)
SELECT order_id, order_date, total_amount 
FROM orders 
WHERE customer_id = 123;
-- Time: 12ms, Data transferred: 600KB (30,000 rows × 20 bytes)
```
**Result: 3.75x faster, 9x less data transferred**

#### **B. Use Covering Indexes (Index-Only Scans)**

**Before:**
```sql
SELECT customer_id, order_date FROM orders WHERE customer_id = 123;
```
```
Index Scan using idx_orders_customer_id on orders
    (cost=0.42..35.67 rows=25 width=12)
    (actual time=0.023..0.156 rows=32 loops=1)
  Index Cond: (customer_id = 123)
  Heap Fetches: 32  -- Must access table for order_date
Execution Time: 0.178 ms
```

**After: Add covering index**
```sql
CREATE INDEX CONCURRENTLY idx_orders_customer_date 
ON orders(customer_id, order_date);

SELECT customer_id, order_date FROM orders WHERE customer_id = 123;
```
```
Index Only Scan using idx_orders_customer_date on orders
    (cost=0.42..12.45 rows=25 width=12)
    (actual time=0.015..0.078 rows=32 loops=1)
  Index Cond: (customer_id = 123)
  Heap Fetches: 0  -- No table access needed!
Execution Time: 0.089 ms
```
**Result: 0.178ms → 0.089ms = 2x faster**

#### **C. Optimize OR Conditions**

**Before (slow):**
```sql
SELECT * FROM orders 
WHERE status = 'pending' OR status = 'processing';
-- Can't use index efficiently
```
```
Seq Scan on orders  (cost=0.00..2500.00 rows=10000 width=180)
                    (actual time=0.234..456.789 rows=10234 loops=1)
  Filter: ((status = 'pending') OR (status = 'processing'))
  Rows Removed by Filter: 89766
Execution Time: 456.890 ms
```

**After (fast):**
```sql
-- Use IN or UNION ALL
SELECT * FROM orders 
WHERE status IN ('pending', 'processing');
```
```
Bitmap Heap Scan on orders  (cost=234.56..1234.56 rows=10000 width=180)
                            (actual time=5.678..23.456 rows=10234 loops=1)
  Recheck Cond: (status = ANY ('{pending,processing}'))
  ->  Bitmap Index Scan on idx_orders_status
          (cost=0.00..232.06 rows=10000 width=0)
          (actual time=5.234..5.234 rows=10234 loops=1)
        Index Cond: (status = ANY ('{pending,processing}'))
Execution Time: 23.567 ms
```
**Result: 457ms → 24ms = 19x faster**

#### **D. Avoid Functions on Indexed Columns**

**Before (index not used):**
```sql
SELECT * FROM orders 
WHERE EXTRACT(YEAR FROM order_date) = 2024;
-- Function on indexed column prevents index usage
```
```
Seq Scan on orders  (cost=0.00..2750.00 rows=5000 width=180)
                    (actual time=0.345..567.890 rows=5234 loops=1)
  Filter: (EXTRACT(year FROM order_date) = 2024)
  Rows Removed by Filter: 94766
Execution Time: 568.123 ms
```

**After (index used):**
```sql
SELECT * FROM orders 
WHERE order_date >= '2024-01-01' 
  AND order_date < '2025-01-01';
```
```
Index Scan using idx_orders_date on orders
    (cost=0.42..234.56 rows=5000 width=180)
    (actual time=0.023..12.345 rows=5234 loops=1)
  Index Cond: ((order_date >= '2024-01-01') AND (order_date < '2025-01-01'))
Execution Time: 12.456 ms
```
**Result: 568ms → 12ms = 47x faster**

#### **E. Use EXISTS Instead of COUNT**

**Before:**
```sql
-- Just checking if any rows exist
SELECT COUNT(*) FROM orders WHERE customer_id = 123;
-- Counts ALL rows even though we only need to know if any exist
-- Time: 45ms
```

**After:**
```sql
-- Stops at first match
SELECT EXISTS(SELECT 1 FROM orders WHERE customer_id = 123);
-- Time: 0.5ms
```
**Result: 90x faster**

#### **F. Optimize Subqueries with JOINs**

**Before (slow subquery):**
```sql
SELECT * FROM customers
WHERE customer_id IN (
    SELECT customer_id FROM orders WHERE order_date > '2024-01-01'
);
-- Execution Time: 890ms
```

**After (JOIN):**
```sql
SELECT DISTINCT c.* 
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_date > '2024-01-01';
-- Execution Time: 45ms
```
**Result: 890ms → 45ms = 20x faster**

---

### **4. Optimization Technique #3: Join Optimization**

#### **A. Choose Appropriate Join Type**

**Nested Loop (best for small result sets):**
```sql
-- When one side returns few rows
SELECT * FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE c.customer_id = 123;
-- Returns 1 customer, 25 orders
-- Nested Loop: 0.156ms
```

**Hash Join (best for large datasets):**
```sql
-- When both sides have many rows
SELECT c.customer_name, COUNT(*) 
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;
-- Returns 20,000 customers, 100,000 orders
-- Hash Join: 234ms (vs Nested Loop: 45,000ms)
```

#### **B. Join Order Matters**

**Before (poor join order):**
```sql
EXPLAIN ANALYZE
SELECT * FROM large_table l  -- 1,000,000 rows
JOIN medium_table m ON l.id = m.large_id  -- 100,000 rows
JOIN small_table s ON l.id = s.large_id  -- 100 rows
WHERE s.status = 'active';  -- Filters to 10 rows
```
```
-- Joins large and medium first (100,000 rows), then filters
Execution Time: 5678ms
```

**After (optimal join order):**
```sql
-- Let planner reorder, or use explicit JOIN order
SELECT * FROM small_table s  -- Start with filtered small table (10 rows)
JOIN large_table l ON s.large_id = l.id
JOIN medium_table m ON l.id = m.large_id
WHERE s.status = 'active';
```
```
-- Filters first (10 rows), then joins
Execution Time: 12ms
```
**Result: 5678ms → 12ms = 473x faster**

#### **C. Use JOIN Instead of Comma-Separated Tables**

**Before (implicit joins - hard to optimize):**
```sql
SELECT * FROM orders o, customers c, products p
WHERE o.customer_id = c.customer_id
  AND o.product_id = p.product_id
  AND o.order_date > '2024-01-01';
```

**After (explicit JOIN - easier to optimize):**
```sql
SELECT * FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN products p ON o.product_id = p.product_id
WHERE o.order_date > '2024-01-01';
```

---

### **5. Optimization Technique #4: Reduce Data Volume**

#### **A. Use LIMIT Effectively**

**Before:**
```sql
-- Application fetches 1,000,000 rows, displays 10
SELECT * FROM orders ORDER BY order_date DESC;
-- Then application takes first 10
-- Time: 5678ms, Memory: 180MB
```

**After:**
```sql
-- Database returns only 10 rows
SELECT * FROM orders ORDER BY order_date DESC LIMIT 10;
-- Time: 12ms, Memory: 1.8KB
```
**Result: 473x faster, 100,000x less memory**

#### **B. Paginate Large Result Sets**

```sql
-- Efficient pagination with keyset pagination (cursor-based)
SELECT * FROM orders 
WHERE order_id > 12345  -- Last seen ID
ORDER BY order_id 
LIMIT 100;
-- Time: 5ms per page

-- Instead of offset pagination (slow for large offsets)
SELECT * FROM orders ORDER BY order_id LIMIT 100 OFFSET 100000;
-- Time: 2500ms (must scan 100,000 rows to skip them)
```

#### **C. Aggregate Before Joining**

**Before (join then aggregate):**
```sql
SELECT c.customer_name, COUNT(*) as order_count
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;
-- Joins 20,000 × 100,000 = 2 billion row comparisons
-- Time: 5678ms
```

**After (aggregate then join):**
```sql
SELECT c.customer_name, o.order_count
FROM customers c
JOIN (
    SELECT customer_id, COUNT(*) as order_count
    FROM orders
    GROUP BY customer_id
) o ON c.customer_id = o.customer_id;
-- Aggregates first, then joins 20,000 × 20,000
-- Time: 234ms
```
**Result: 5678ms → 234ms = 24x faster**

---

### **6. Optimization Technique #5: Update Statistics**

```sql
-- Stale statistics lead to poor query plans
ANALYZE orders;
ANALYZE customers;
ANALYZE products;

-- Or all tables in database
ANALYZE;

-- Increase statistics detail for high-cardinality columns
ALTER TABLE orders ALTER COLUMN status SET STATISTICS 1000;
ANALYZE orders;
```

**Impact Example:**
```sql
-- Before ANALYZE
EXPLAIN ANALYZE
SELECT * FROM orders WHERE status = 'rare_status';
```
```
Seq Scan  (cost=0.00..2500.00 rows=50000 width=180)
          (actual time=0.234..456.789 rows=3 loops=1)
-- Estimated 50,000, actual 3 → 16,667x overestimate!
-- Planner chose seq scan because it thought many rows match
```

```sql
-- After ANALYZE
EXPLAIN ANALYZE
SELECT * FROM orders WHERE status = 'rare_status';
```
```
Index Scan  (cost=0.42..12.45 rows=5 width=180)
            (actual time=0.023..0.034 rows=3 loops=1)
-- Estimated 5, actual 3 → Much better!
-- Now uses index scan
```

---

### **7. Optimization Technique #6: Configuration Tuning**

#### **A. Increase work_mem**

```sql
-- Default work_mem = 4MB (often too small)
SHOW work_mem;

-- Increase for complex sorts/hashes
SET work_mem = '256MB';  -- Session level

-- Or in postgresql.conf
work_mem = 256MB
```

**Impact:**
```sql
-- Before (work_mem = 4MB)
EXPLAIN ANALYZE
SELECT * FROM orders ORDER BY total_amount DESC;
```
```
Sort  (cost=8934.56..9184.56 rows=100000 width=180)
      (actual time=567.890..678.901 rows=100000 loops=1)
  Sort Key: total_amount DESC
  Sort Method: external merge  Disk: 18432kB  -- Spilled to disk!
  ->  Seq Scan on orders  (cost=0.00..2200.00 rows=100000 width=180)
Execution Time: 789.012 ms
```

```sql
-- After (work_mem = 256MB)
EXPLAIN ANALYZE
SELECT * FROM orders ORDER BY total_amount DESC;
```
```
Sort  (cost=8934.56..9184.56 rows=100000 width=180)
      (actual time=123.456..145.678 rows=100000 loops=1)
  Sort Key: total_amount DESC
  Sort Method: quicksort  Memory: 18MB  -- In memory!
  ->  Seq Scan on orders  (cost=0.00..2200.00 rows=100000 width=180)
Execution Time: 156.789 ms
```
**Result: 789ms → 157ms = 5x faster**

#### **B. Adjust random_page_cost for SSDs**

```sql
-- Default assumes HDD
SHOW random_page_cost;  -- 4.0

-- For SSD, random access is nearly as fast as sequential
ALTER SYSTEM SET random_page_cost = 1.1;
SELECT pg_reload_conf();

-- Makes planner more likely to choose index scans
```

#### **C. Increase effective_cache_size**

```sql
-- Tells planner how much memory available for caching
-- Should be ~50-75% of total RAM
ALTER SYSTEM SET effective_cache_size = '32GB';
SELECT pg_reload_conf();
```

---

### **8. Optimization Technique #7: Materialization**

#### **A. Materialized Views**

**Problem: Complex query run frequently**
```sql
-- Expensive aggregation query
SELECT 
    c.customer_name,
    COUNT(*) as total_orders,
    SUM(o.total_amount) as lifetime_value,
    AVG(o.total_amount) as avg_order_value
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;
-- Time: 5678ms
```

**Solution: Materialized view**
```sql
CREATE MATERIALIZED VIEW customer_lifetime_value AS
SELECT 
    c.customer_name,
    COUNT(*) as total_orders,
    SUM(o.total_amount) as lifetime_value,
    AVG(o.total_amount) as avg_order_value
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;

CREATE INDEX ON customer_lifetime_value(customer_name);

-- Query the materialized view
SELECT * FROM customer_lifetime_value WHERE customer_name LIKE 'John%';
-- Time: 5ms (1136x faster!)

-- Refresh periodically
REFRESH MATERIALIZED VIEW CONCURRENTLY customer_lifetime_value;
```

#### **B. CTEs with MATERIALIZATION**

```sql
-- Force materialization of CTE
WITH expensive_calculation AS MATERIALIZED (
    SELECT customer_id, complex_function(data) as result
    FROM large_table
)
SELECT * FROM expensive_calculation WHERE result > 100;
```

---

### **9. Real-World Optimization Example**

**Scenario: E-commerce dashboard query taking 15 seconds**

**Initial Query:**
```sql
SELECT 
    p.product_name,
    COUNT(DISTINCT o.order_id) as times_ordered,
    SUM(oi.quantity) as total_quantity,
    SUM(oi.quantity * oi.unit_price) as revenue
FROM products p
LEFT JOIN order_items oi ON p.product_id = oi.product_id
LEFT JOIN orders o ON oi.order_id = o.order_id
WHERE o.order_date >= CURRENT_DATE - INTERVAL '90 days'
   OR o.order_date IS NULL
GROUP BY p.product_id, p.product_name
ORDER BY revenue DESC;
```

**Step 1: Analyze**
```sql
EXPLAIN (ANALYZE, BUFFERS)
[query];
```
**Findings:**
- Sequential scan on orders (2M rows)
- 94% of rows filtered out after scan
- No index on order_date
- Nested loop join (wrong join type for this data volume)

**Step 2: Add Index**
```sql
CREATE INDEX CONCURRENTLY idx_orders_date ON orders(order_date);
-- Time reduced: 15s → 8s
```

**Step 3: Rewrite Query**
```sql
-- Filter orders first, then join
WITH recent_orders AS (
    SELECT order_id, order_date
    FROM orders
    WHERE order_date >= CURRENT_DATE - INTERVAL '90 days'
)
SELECT 
    p.product_name,
    COUNT(DISTINCT ro.order_id) as times_ordered,
    SUM(oi.quantity) as total_quantity,
    SUM(oi.quantity * oi.unit_price) as revenue
FROM products p
LEFT JOIN order_items oi ON p.product_id = oi.product_id
LEFT JOIN recent_orders ro ON oi.order_id = ro.order_id
GROUP BY p.product_id, p.product_name
HAVING COUNT(ro.order_id) > 0  -- Exclude products with no recent orders
ORDER BY revenue DESC;
-- Time reduced: 8s → 2s
```

**Step 4: Create Covering Index**
```sql
CREATE INDEX CONCURRENTLY idx_order_items_product 
ON order_items(product_id, order_id, quantity, unit_price);
-- Time reduced: 2s → 0.5s
```

**Step 5: Materialized View for Dashboard**
```sql
CREATE MATERIALIZED VIEW dashboard_product_stats AS
[optimized query];

CREATE INDEX ON dashboard_product_stats(product_name);
CREATE INDEX ON dashboard_product_stats(revenue DESC);

-- Refresh every hour
-- Now queries take 5ms instead of 15s = 3000x faster!
```

---

### **10. Optimization Checklist**

**Immediate Actions:**
- [ ] Identify slow queries with pg_stat_statements
- [ ] Run EXPLAIN ANALYZE on slow queries
- [ ] Check for missing indexes on WHERE/JOIN columns
- [ ] Verify statistics are up-to-date (ANALYZE)
- [ ] Look for sequential scans on large tables

**Query Rewriting:**
- [ ] Select only needed columns (avoid SELECT *)
- [ ] Use EXISTS instead of COUNT for existence checks
- [ ] Convert OR to IN or UNION ALL
- [ ] Avoid functions on indexed columns
- [ ] Add LIMIT when appropriate

**Indexing:**
- [ ] Single-column indexes for common filters
- [ ] Composite indexes for multi-column filters
- [ ] Partial indexes for subset queries
- [ ] Covering indexes for frequently accessed columns
- [ ] Expression indexes for function-based queries

**Configuration:**
- [ ] Increase work_mem for complex sorts/hashes
- [ ] Adjust random_page_cost for SSDs
- [ ] Set appropriate effective_cache_size
- [ ] Enable auto_explain for slow query logging

**Advanced:**
- [ ] Consider materialized views for complex aggregations
- [ ] Use connection pooling (PgBouncer)
- [ ] Partition very large tables
- [ ] Review and optimize join orders

---

### **Interview Tips:**
- **Systematic approach**: "First identify with pg_stat_statements, then EXPLAIN ANALYZE, then optimize"
- **Real numbers**: "Added composite index, reduced query from 568ms to 0.86ms - 661x improvement"
- **Multiple techniques**: Mention indexing, query rewriting, configuration tuning, materialization
- **Know trade-offs**: "Indexes speed reads but slow writes - analyzed insert volume before adding"
- **Practical example**: "Dashboard query taking 15s; added indexes, rewrote query, created materialized view; now 5ms = 3000x faster"
- **Metrics matter**: Always quote before/after times and improvement ratios
- **Statistics importance**: "Ran ANALYZE and query plan changed from seq scan to index scan - 47x faster"

</details>

<details>
<summary><strong>45. What is vacuuming in PostgreSQL?</strong></summary>

### **Answer:**

Vacuuming is PostgreSQL's essential maintenance process that reclaims storage occupied by dead tuples (deleted or updated rows), prevents transaction ID wraparound, and updates table statistics for optimal query planning. It's critical for maintaining database health and performance.

---

### **1. Why Vacuuming is Necessary**

PostgreSQL uses **Multi-Version Concurrency Control (MVCC)** which creates new row versions instead of modifying existing rows in place.

#### **How MVCC Creates Dead Tuples:**

**Example: UPDATE Operation**
```sql
-- Initial state
INSERT INTO orders (order_id, status, amount) 
VALUES (1, 'pending', 100.00);

-- Table contains: [order_id=1, status='pending', amount=100.00]

-- Update the order
UPDATE orders SET status = 'shipped' WHERE order_id = 1;

-- PostgreSQL doesn't modify the existing row!
-- Instead, it:
-- 1. Creates a NEW row version: [order_id=1, status='shipped', amount=100.00]
-- 2. Marks old row as "dead" (not visible to new transactions)
-- 3. Old row: [order_id=1, status='pending', amount=100.00] ← DEAD TUPLE

-- DELETE operation
DELETE FROM orders WHERE order_id = 1;
-- Creates another dead tuple: [order_id=1, status='shipped', amount=100.00]
```

**Result:**
- **Live tuple**: 0 (order deleted)
- **Dead tuples**: 2 (pending version, shipped version)
- **Space used**: Same as having 2 rows

**Without vacuuming:**
- Disk space grows continuously
- Table scans become slower (must scan dead tuples)
- Indexes point to dead tuples (bloat)
- Transaction IDs eventually wrap around (database shutdown!)

---

### **2. What VACUUM Does**

```sql
-- Basic vacuum on a table
VACUUM orders;

-- Vacuum all tables in database
VACUUM;
```

#### **VACUUM Operations:**

**A. Reclaim Dead Tuple Space**
```sql
-- Before VACUUM
SELECT 
    pg_size_pretty(pg_total_relation_size('orders')) as total_size,
    n_live_tup,
    n_dead_tup
FROM pg_stat_user_tables
WHERE tablename = 'orders';
```
```
total_size | n_live_tup | n_dead_tup
-----------+------------+-----------
500 MB     |   100,000  |   450,000   ← 82% dead tuples!
```

```sql
VACUUM orders;

-- After VACUUM
SELECT 
    pg_size_pretty(pg_total_relation_size('orders')) as total_size,
    n_live_tup,
    n_dead_tup
FROM pg_stat_user_tables
WHERE tablename = 'orders';
```
```
total_size | n_live_tup | n_dead_tup
-----------+------------+-----------
500 MB     |   100,000  |      45    ← Dead tuples removed!
```

**Note**: Regular VACUUM **marks space as reusable** but **doesn't return it to OS**
- File size stays 500MB
- Space can be reused for new rows
- Use `VACUUM FULL` to shrink file (but it locks table)

**B. Update Free Space Map (FSM)**
```sql
-- FSM tracks which pages have free space
-- VACUUM updates FSM so INSERTs can use reclaimed space
VACUUM orders;

-- New inserts will now use freed space instead of growing the file
```

**C. Prevent Transaction ID Wraparound**

PostgreSQL uses 32-bit transaction IDs (XIDs):
- Max value: 2^32 = 4,294,967,296 transactions
- After that, wraps around to 0
- Without VACUUM, old transactions become "in the future" (data loss!)

```sql
-- Check transaction ID age
SELECT 
    datname,
    age(datfrozenxid) as xid_age,
    2147483647 - age(datfrozenxid) as xids_remaining
FROM pg_database
ORDER BY age(datfrozenxid) DESC;
```
```
datname    | xid_age   | xids_remaining
-----------+-----------+----------------
mydb       | 50000000  | 2097483647     ← Safe
olddb      | 2000000000| 147483647      ← Warning! VACUUM soon!
```

**Wraparound Protection:**
```sql
-- VACUUM "freezes" old transaction IDs
VACUUM orders;

-- After freezing, rows are visible to all transactions
-- Prevents wraparound crisis
```

**D. Update Table Statistics**

```sql
-- VACUUM ANALYZE updates statistics AND vacuums
VACUUM ANALYZE orders;

-- Equivalent to:
VACUUM orders;
ANALYZE orders;
```

---

### **3. VACUUM Options and Variants**

#### **A. Basic VACUUM**

```sql
-- Vacuum specific table
VACUUM orders;

-- Vacuum all tables
VACUUM;

-- Vacuum specific schema
VACUUM schema_name.*;
```

**Characteristics:**
- Non-blocking (table remains accessible)
- Reclaims space for reuse (doesn't shrink file)
- Relatively fast

#### **B. VACUUM VERBOSE**

```sql
VACUUM VERBOSE orders;
```

**Output:**
```
INFO:  vacuuming "public.orders"
INFO:  "orders": found 450000 removable, 100000 nonremovable row versions in 6250 pages
DETAIL:  0 dead row versions cannot be removed yet.
CPU: user: 1.23 s, system: 0.45 s, elapsed: 2.89 s.
INFO:  "orders": truncated 6250 to 1250 pages
DETAIL:  CPU: user: 0.12 s, system: 0.03 s, elapsed: 0.18 s
```

**Useful for:**
- Understanding what VACUUM is doing
- Identifying heavily bloated tables
- Debugging performance issues

#### **C. VACUUM FREEZE**

```sql
-- Aggressively freeze old transaction IDs
VACUUM FREEZE orders;
```

**When to use:**
- Approaching transaction ID wraparound
- Before major PostgreSQL upgrades
- After large data loads/deletes

**Example:**
```sql
-- Check tables needing freeze
SELECT 
    schemaname,
    tablename,
    age(relfrozenxid) as xid_age
FROM pg_stat_user_tables
ORDER BY age(relfrozenxid) DESC;
```
```
schemaname | tablename | xid_age
-----------+-----------+-----------
public     | old_logs  | 1800000000  ← FREEZE immediately!
public     | orders    | 50000000    ← Fine for now
```

#### **D. VACUUM with Column List (PostgreSQL 12+)**

```sql
-- Vacuum only specific columns (updates stats for those columns)
VACUUM (ANALYZE) orders (status, order_date);
```

---

### **4. Monitoring Vacuum Progress**

#### **A. Check Vacuum Status**

```sql
-- See active vacuum operations
SELECT 
    pid,
    datname,
    relid::regclass AS table_name,
    phase,
    heap_blks_total,
    heap_blks_scanned,
    heap_blks_vacuumed,
    index_vacuum_count,
    max_dead_tuples,
    num_dead_tuples
FROM pg_stat_progress_vacuum;
```

**Output:**
```
pid   | table_name | phase              | heap_blks_scanned | heap_blks_total
------+------------+--------------------+------------------+----------------
12345 | orders     | scanning heap      | 2500             | 6250
12346 | customers  | vacuuming indexes  | 1200             | 1200
```

#### **B. Check Dead Tuple Accumulation**

```sql
-- Tables with most dead tuples
SELECT 
    schemaname,
    tablename,
    n_live_tup,
    n_dead_tup,
    ROUND(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 2) as dead_pct,
    last_vacuum,
    last_autovacuum
FROM pg_stat_user_tables
WHERE n_dead_tup > 0
ORDER BY n_dead_tup DESC
LIMIT 20;
```

**Output:**
```
tablename | n_live_tup | n_dead_tup | dead_pct | last_vacuum         | last_autovacuum
----------+------------+------------+----------+---------------------+------------------
orders    | 100000     | 450000     | 81.82    | 2024-12-20 10:00:00 | NULL
logs      | 50000      | 200000     | 80.00    | NULL                | 2024-12-23 08:00:00
```

**Interpretation:**
- **dead_pct > 20%**: Table needs vacuuming
- **last_vacuum = NULL**: Only autovacuum has run (or never vacuumed)
- **last_autovacuum = NULL**: Autovacuum might be disabled or not triggered

#### **C. Check Table Bloat**

```sql
-- Estimate table bloat
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as total_size,
    ROUND(100 * n_dead_tup::numeric / NULLIF(n_live_tup + n_dead_tup, 0), 2) as dead_pct,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)::numeric * 
                   n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0)) as wasted_space
FROM pg_stat_user_tables
WHERE n_dead_tup > 1000
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

---

### **5. VACUUM vs Other Operations**

#### **A. VACUUM vs VACUUM FULL**

| Feature | VACUUM | VACUUM FULL |
|---------|--------|-------------|
| **Lock level** | No lock (concurrent access) | Exclusive lock (table unavailable) |
| **Space reclamation** | Marks space reusable (file stays same size) | Returns space to OS (shrinks file) |
| **Speed** | Fast (minutes) | Slow (hours for large tables) |
| **Use case** | Regular maintenance | Extreme bloat recovery |
| **Downtime** | None | Yes |

**Example:**
```sql
-- Check table size
SELECT pg_size_pretty(pg_total_relation_size('orders'));
-- Result: 500 MB (but only 100 MB of live data)

-- Regular VACUUM
VACUUM orders;
SELECT pg_size_pretty(pg_total_relation_size('orders'));
-- Result: 500 MB (space marked reusable but file not shrunk)

-- VACUUM FULL (locks table!)
VACUUM FULL orders;
SELECT pg_size_pretty(pg_total_relation_size('orders'));
-- Result: 110 MB (file shrunk, 10 MB for indexes)
-- Downtime: 15 minutes for large table
```

#### **B. VACUUM vs ANALYZE**

```sql
-- VACUUM: Reclaims space, prevents wraparound
VACUUM orders;

-- ANALYZE: Updates statistics for query planner
ANALYZE orders;

-- VACUUM ANALYZE: Does both
VACUUM ANALYZE orders;
```

**When to use each:**
- **After bulk DELETE/UPDATE**: `VACUUM ANALYZE` (space + stats)
- **After bulk INSERT**: `ANALYZE` only (no dead tuples yet)
- **Weekly maintenance**: `VACUUM ANALYZE` (both needed)

---

### **6. When to Run VACUUM**

#### **Automatic (Autovacuum)**
```sql
-- Check if autovacuum is enabled
SHOW autovacuum;  -- Should be 'on'

-- Autovacuum runs when:
-- 1. Dead tuples exceed threshold
-- 2. Transaction age approaches wraparound
```

#### **Manual VACUUM Scenarios**

**A. After Bulk Operations**
```sql
-- After deleting 80% of table
DELETE FROM old_logs WHERE log_date < '2020-01-01';
-- Deleted 800,000 rows

VACUUM ANALYZE old_logs;
-- Reclaims space immediately instead of waiting for autovacuum
```

**B. Before Long-Running Queries**
```sql
-- Ensure statistics are up-to-date for optimal query plans
VACUUM ANALYZE orders;
VACUUM ANALYZE customers;

-- Now run complex reporting query
SELECT ... (complex query)
```

**C. Transaction ID Monitoring**
```sql
-- Check transaction age
SELECT 
    datname,
    age(datfrozenxid),
    CASE 
        WHEN age(datfrozenxid) > 200000000 THEN 'URGENT: Run VACUUM FREEZE'
        WHEN age(datfrozenxid) > 100000000 THEN 'WARNING: Schedule VACUUM soon'
        ELSE 'OK'
    END as status
FROM pg_database
ORDER BY age(datfrozenxid) DESC;
```

**D. Performance Issues**
```sql
-- Query suddenly slow? Check for bloat
SELECT 
    tablename,
    n_dead_tup,
    ROUND(100.0 * n_dead_tup / NULLIF(n_live_tup, 0), 2) as dead_ratio
FROM pg_stat_user_tables
WHERE tablename = 'orders';

-- If dead_ratio > 20%, run VACUUM
VACUUM ANALYZE orders;
```

---

### **7. Real-World Example**

**Scenario: Orders table query performance degraded over 6 months**

```sql
-- Initial state (after VACUUM)
SELECT 
    pg_size_pretty(pg_total_relation_size('orders')) as size,
    n_live_tup,
    n_dead_tup
FROM pg_stat_user_tables
WHERE tablename = 'orders';
```
```
size   | n_live_tup | n_dead_tup
-------+------------+-----------
250 MB |   100,000  |      100
```

**6 months later (no VACUUM):**
```sql
-- Heavy UPDATE activity (status changes, shipping updates)
-- Average: 50,000 UPDATEs per day
-- = 50,000 dead tuples/day × 180 days = 9,000,000 dead tuples

SELECT 
    pg_size_pretty(pg_total_relation_size('orders')) as size,
    n_live_tup,
    n_dead_tup
FROM pg_stat_user_tables
WHERE tablename = 'orders';
```
```
size     | n_live_tup | n_dead_tup
---------+------------+------------
2.3 GB   |   150,000  |  9,000,000   ← 98.4% dead!
```

**Query performance impact:**
```sql
EXPLAIN ANALYZE
SELECT * FROM orders WHERE status = 'pending';
```
```
Seq Scan on orders  (cost=0.00..45000.00 rows=5000 width=180)
                    (actual time=0.456..8234.567 rows=5123 loops=1)
  Filter: (status = 'pending')
  Rows Removed by Filter: 9144877  ← Scanning 9M dead tuples!
Execution Time: 8234.678 ms
```

**After VACUUM:**
```sql
VACUUM ANALYZE orders;

SELECT 
    pg_size_pretty(pg_total_relation_size('orders')) as size,
    n_live_tup,
    n_dead_tup
FROM pg_stat_user_tables
WHERE tablename = 'orders';
```
```
size     | n_live_tup | n_dead_tup
---------+------------+-----------
2.3 GB   |   150,000  |      50    ← Dead tuples gone!
```
*Note: Space is reusable but file still 2.3 GB*

```sql
-- Re-run query
EXPLAIN ANALYZE
SELECT * FROM orders WHERE status = 'pending';
```
```
Index Scan using idx_orders_status on orders
    (cost=0.42..234.56 rows=5000 width=180)
    (actual time=0.023..12.345 rows=5123 loops=1)
  Index Cond: (status = 'pending')
Execution Time: 12.456 ms
```

**Result: 8235ms → 12ms = 686x faster!**

**For even better results (with downtime):**
```sql
-- During maintenance window
VACUUM FULL orders;

SELECT pg_size_pretty(pg_total_relation_size('orders'));
-- Result: 280 MB (was 2.3 GB) = 87% reduction
```

---

### **8. Best Practices**

1. **Keep autovacuum enabled** - Handles routine maintenance automatically
2. **Manual VACUUM after bulk operations** - Don't wait for autovacuum
3. **Monitor transaction ID age** - Prevent wraparound emergencies
4. **Use VACUUM ANALYZE** - Update stats and reclaim space together
5. **Schedule VACUUM FULL carefully** - Requires exclusive lock (downtime)
6. **Monitor dead tuples** - Alert when >20% of table is dead
7. **Tune autovacuum parameters** - For write-heavy workloads
8. **Regular statistics updates** - Keep query planner informed

---

### **Interview Tips:**
- **MVCC connection**: "VACUUM is necessary because MVCC creates dead tuples on every UPDATE/DELETE"
- **What it does**: "Reclaims dead tuple space, prevents transaction ID wraparound, updates statistics"
- **Regular vs FULL**: "Regular VACUUM is non-blocking but doesn't shrink files; VACUUM FULL locks table but returns space to OS"
- **Real example**: "Orders table query degraded from 12ms to 8.2 seconds over 6 months due to 98% dead tuples. VACUUM restored performance - 686x improvement"
- **Monitoring**: "Monitor n_dead_tup in pg_stat_user_tables; alert when >20% dead"
- **Production practice**: "Ran manual VACUUM after bulk delete of 800K rows instead of waiting hours for autovacuum"
- **Wraparound**: "Without VACUUM, transaction IDs wrap around after 2 billion transactions, causing database shutdown"

</details>

<details>
<summary><strong>46. What is the difference between VACUUM and VACUUM FULL?</strong></summary>

### **Answer:**

`VACUUM` and `VACUUM FULL` are both PostgreSQL maintenance operations that reclaim space from dead tuples, but they differ significantly in how they work, their impact on the database, and when to use them.

---

### **1. Quick Comparison**

| Feature | VACUUM | VACUUM FULL |
|---------|--------|-------------|
| **Locking** | No lock - concurrent reads/writes | Exclusive lock - table unavailable |
| **Space reclamation** | Marks space reusable within file | Returns space to operating system |
| **File size** | Unchanged (doesn't shrink) | Shrinks file to minimum size |
| **Speed** | Fast (seconds to minutes) | Slow (minutes to hours) |
| **Downtime** | None - zero downtime | Yes - table locked during operation |
| **Frequency** | Regular (daily/hourly) | Rare (quarterly/yearly or emergency) |
| **Disk space needed** | Minimal | Requires 2x table size temporarily |
| **Index handling** | Keeps indexes | Rebuilds all indexes |
| **Best for** | Routine maintenance | Extreme bloat recovery |

---

### **2. How VACUUM Works**

#### **A. Space Marking (Not Reclamation)**

```sql
-- Initial state: 1M rows, 180 MB
INSERT INTO orders (order_id, status, amount) 
SELECT generate_series(1, 1000000), 'pending', 100.00;

SELECT pg_size_pretty(pg_total_relation_size('orders')) as size;
-- Result: 180 MB

-- Delete 900K rows (90% of table)
DELETE FROM orders WHERE order_id <= 900000;

-- File size still 180 MB!
SELECT pg_size_pretty(pg_total_relation_size('orders')) as size;
-- Result: 180 MB (unchanged)

-- Run VACUUM
VACUUM orders;

-- File size STILL 180 MB, but space is now reusable
SELECT pg_size_pretty(pg_total_relation_size('orders')) as size;
-- Result: 180 MB (file not shrunk)

-- Check dead tuples
SELECT n_live_tup, n_dead_tup FROM pg_stat_user_tables WHERE tablename = 'orders';
-- Result: n_live_tup=100000, n_dead_tup=50 (dead tuples cleared)
```

**What VACUUM did:**
- Marked space from 900K deleted rows as "reusable"
- Updated Free Space Map (FSM)
- File stays 180 MB
- New INSERT/UPDATE operations will use freed space

**Visualization:**
```
Before DELETE:
[Data][Data][Data][Data][Data]  ← 180 MB of live data

After DELETE (before VACUUM):
[Data][Dead][Dead][Dead][Dead]  ← Still 180 MB, 162 MB wasted

After VACUUM:
[Data][Free][Free][Free][Free]  ← Still 180 MB, but space is reusable

New INSERT operations will use [Free] space instead of growing file
```

#### **B. Free Space Map (FSM)**

```sql
-- VACUUM updates the FSM to track free space
VACUUM orders;

-- New inserts now use freed space
INSERT INTO orders (order_id, status, amount) 
SELECT generate_series(1000001, 1100000), 'shipped', 150.00;

-- File size unchanged (used freed space)
SELECT pg_size_pretty(pg_total_relation_size('orders'));
-- Result: 180 MB (no growth!)
```

#### **C. Concurrent Operations**

```sql
-- Terminal 1: Start VACUUM (non-blocking)
VACUUM orders;
-- Running... (takes 2 minutes)

-- Terminal 2: Query works while VACUUM runs
SELECT * FROM orders WHERE status = 'pending';
-- Works fine! Returns results immediately

-- Terminal 3: Updates work too
UPDATE orders SET status = 'shipped' WHERE order_id = 123;
-- Success! No blocking
```

**VACUUM characteristics:**
- Uses `SHARE UPDATE EXCLUSIVE` lock
- Allows concurrent SELECT, INSERT, UPDATE, DELETE
- Only blocks: ALTER TABLE, CREATE INDEX, VACUUM, ANALYZE

---

### **3. How VACUUM FULL Works**

#### **A. Complete Table Rewrite**

```sql
-- Same scenario: 180 MB file, only 18 MB live data
SELECT pg_size_pretty(pg_total_relation_size('orders')) as size;
-- Result: 180 MB (90% wasted)

-- Run VACUUM FULL
VACUUM FULL orders;

-- File shrunk to minimum size
SELECT pg_size_pretty(pg_total_relation_size('orders')) as size;
-- Result: 20 MB (18 MB data + 2 MB indexes)
```

**What VACUUM FULL does:**
1. Creates a **new copy** of the table with only live rows
2. Rebuilds **all indexes**
3. Replaces old table file with new one
4. Returns freed space to operating system
5. Updates statistics

**Visualization:**
```
Before VACUUM FULL:
File: orders (180 MB on disk)
[Data][Dead][Dead][Dead][Dead]

During VACUUM FULL:
File: orders (180 MB) - locked, unavailable
File: orders_new (18 MB) - being built
[Data] ← Copying only live rows

After VACUUM FULL:
File: orders (20 MB on disk) - file shrunk
[Data] ← Compact, all dead space removed
```

#### **B. Exclusive Locking (Downtime Required)**

```sql
-- Terminal 1: Start VACUUM FULL
VACUUM FULL orders;
-- Running... (takes 45 minutes for large table)

-- Terminal 2: Try to query (BLOCKED!)
SELECT * FROM orders WHERE status = 'pending';
-- Waiting... Waiting... (blocked until VACUUM FULL completes)

-- Terminal 3: Try to update (BLOCKED!)
UPDATE orders SET status = 'shipped' WHERE order_id = 123;
-- ERROR: deadlock or timeout after waiting
```

**VACUUM FULL uses `ACCESS EXCLUSIVE` lock:**
- Blocks ALL operations: SELECT, INSERT, UPDATE, DELETE
- Table is completely unavailable
- Must schedule during maintenance window

#### **C. Disk Space Requirements**

```sql
-- Table is 500 GB with 80% bloat (400 GB live data)
SELECT pg_size_pretty(pg_total_relation_size('orders'));
-- Result: 500 GB

-- VACUUM FULL needs temporary space
VACUUM FULL orders;
-- Requires: 400 GB (new table) + 500 GB (old table) = 900 GB during operation
-- After completion: 450 GB (400 GB data + 50 GB indexes)
-- Freed: 50 GB returned to OS
```

**Disk space calculation:**
- During VACUUM FULL: **Old table + New table** (nearly 2x table size)
- After completion: Only new table remains
- **Risk**: If not enough disk space, VACUUM FULL fails!

---

### **4. Performance Comparison**

#### **A. Execution Time**

**VACUUM:**
```sql
-- Table: 100 GB, 50% dead tuples
\timing on
VACUUM orders;
-- Time: 3 minutes 45 seconds
```

**VACUUM FULL:**
```sql
-- Same table: 100 GB, 50% dead tuples
\timing on
VACUUM FULL orders;
-- Time: 2 hours 15 minutes (36x slower!)
```

**Why VACUUM FULL is slower:**
- Must read every live row
- Must write every row to new file
- Must rebuild all indexes
- Must perform full table scan

#### **B. Query Performance Impact**

**Scenario: Table with 80% bloat**

**Before any VACUUM:**
```sql
EXPLAIN ANALYZE
SELECT * FROM orders WHERE status = 'pending';
```
```
Seq Scan on orders  (cost=0.00..45000.00 rows=5000 width=180)
                    (actual time=0.456..8234.567 rows=5123 loops=1)
  Filter: (status = 'pending')
  Rows Removed by Filter: 20877  ← Must scan dead tuples
Execution Time: 8234.678 ms
```

**After VACUUM:**
```sql
VACUUM orders;

EXPLAIN ANALYZE
SELECT * FROM orders WHERE status = 'pending';
```
```
Index Scan using idx_orders_status on orders
    (cost=0.42..234.56 rows=5000 width=180)
    (actual time=0.023..45.678 rows=5123 loops=1)
  Index Cond: (status = 'pending')
Execution Time: 45.789 ms
```
**Improvement: 8235ms → 46ms = 179x faster**
- Dead tuples removed
- File still 500 GB but space is reusable
- Query uses index now (dead tuples confused planner before)

**After VACUUM FULL:**
```sql
VACUUM FULL orders;

EXPLAIN ANALYZE
SELECT * FROM orders WHERE status = 'pending';
```
```
Index Scan using idx_orders_status on orders
    (cost=0.42..156.78 rows=5000 width=180)
    (actual time=0.023..12.345 rows=5123 loops=1)
  Index Cond: (status = 'pending')
Execution Time: 12.456 ms
```
**Improvement: 46ms → 12ms = 3.7x faster than VACUUM**
- File shrunk from 500 GB to 110 GB
- Better cache efficiency (more data fits in RAM)
- Indexes rebuilt, more compact

---

### **5. When to Use Each**

#### **A. Use Regular VACUUM When:**

✅ **Routine maintenance**
```sql
-- Daily or after bulk operations
VACUUM ANALYZE orders;
```

✅ **After large DELETE/UPDATE operations**
```sql
-- Deleted 100K rows (10% of table)
DELETE FROM logs WHERE log_date < '2020-01-01';
VACUUM logs;  -- Immediate space reclamation
```

✅ **High-write tables**
```sql
-- Tables with frequent updates
VACUUM sessions;  -- Run multiple times per day
```

✅ **Transaction ID wraparound prevention**
```sql
-- Approaching 2 billion transactions
VACUUM FREEZE orders;
```

✅ **Zero downtime requirement**
```sql
-- Production system, 24/7 uptime
VACUUM orders;  -- No impact on users
```

#### **B. Use VACUUM FULL When:**

⚠️ **Extreme bloat (>80% dead space)**
```sql
-- Table is 1 TB but only 150 GB live data
-- 850 GB wasted
VACUUM FULL orders;  -- Reclaim 850 GB to OS
```

⚠️ **One-time cleanup after major data purge**
```sql
-- Deleted 95% of historical data
DELETE FROM old_transactions WHERE year < 2015;
-- Table was 10 TB, now 500 GB live data
VACUUM FULL old_transactions;  -- Shrink file from 10 TB to 550 GB
```

⚠️ **Disk space crisis**
```sql
-- Disk 95% full, need space immediately
-- Regular VACUUM won't free disk space
VACUUM FULL large_table;  -- Returns space to OS
```

⚠️ **Before physical backups**
```sql
-- Reducing backup size
VACUUM FULL all_tables;  -- Shrink database before backup
-- Backup size: 5 TB → 1.2 TB
```

⚠️ **During scheduled maintenance window**
```sql
-- Sunday 2 AM - 6 AM maintenance
VACUUM FULL orders;
VACUUM FULL customers;
-- Users expect downtime
```

---

### **6. Alternatives to VACUUM FULL**

#### **A. pg_repack (Zero-Downtime Alternative)**

```bash
# Install pg_repack extension
CREATE EXTENSION pg_repack;

# Rebuild table without locking (similar to VACUUM FULL but concurrent)
pg_repack -t orders mydatabase

# How it works:
# 1. Creates new table
# 2. Copies data (table remains accessible)
# 3. Tracks changes during copy
# 4. Swaps tables (brief lock, <1 second)
```

**Advantages over VACUUM FULL:**
- ✅ No downtime (table remains accessible)
- ✅ Same space reclamation
- ✅ Rebuilds indexes
- ❌ Requires 2x disk space (like VACUUM FULL)

#### **B. CREATE TABLE ... AS + Rename**

```sql
-- Manual table rebuild
BEGIN;
CREATE TABLE orders_new AS 
SELECT * FROM orders;

-- Add indexes
CREATE INDEX idx_orders_new_status ON orders_new(status);
CREATE INDEX idx_orders_new_date ON orders_new(order_date);

-- Swap tables
DROP TABLE orders;
ALTER TABLE orders_new RENAME TO orders;
COMMIT;
```

**Advantages:**
- ✅ More control over process
- ✅ Can optimize structure during rebuild
- ❌ Requires custom scripting
- ❌ Must recreate all constraints/indexes manually

#### **C. Partitioning**

```sql
-- Instead of VACUUM FULL on huge table, partition it
CREATE TABLE orders (
    order_id bigint,
    order_date date,
    ...
) PARTITION BY RANGE (order_date);

CREATE TABLE orders_2024 PARTITION OF orders 
FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');

CREATE TABLE orders_2025 PARTITION OF orders 
FOR VALUES FROM ('2025-01-01') TO ('2026-01-01');

-- Drop old partitions entirely (instant)
DROP TABLE orders_2020;  -- Much faster than DELETE + VACUUM FULL
```

---

### **7. Real-World Decision Example**

**Scenario: Orders table performance degraded**

**Investigation:**
```sql
-- Check table bloat
SELECT 
    pg_size_pretty(pg_total_relation_size('orders')) as total_size,
    pg_size_pretty(pg_relation_size('orders')) as table_size,
    n_live_tup,
    n_dead_tup,
    ROUND(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 2) as dead_pct
FROM pg_stat_user_tables
WHERE tablename = 'orders';
```

**Results:**
```
total_size | table_size | n_live_tup | n_dead_tup | dead_pct
-----------+------------+------------+------------+---------
2.3 GB     | 1.8 GB     | 150,000    | 50,000     | 25.00%
```

**Decision Matrix:**

| Bloat Level | Dead % | Action | Downtime | Result |
|-------------|--------|--------|----------|--------|
| **25%** (Current) | 25% | `VACUUM ANALYZE` | None | Space marked reusable, stats updated |
| **40-60%** | 50% | `VACUUM ANALYZE` + monitoring | None | Usually sufficient, consider pg_repack if persistent |
| **70-85%** | 80% | `pg_repack` or scheduled `VACUUM FULL` | None / 2 hours | Significant space reclamation |
| **>85%** | 90%+ | Emergency `VACUUM FULL` | Required | Critical - file too bloated |

**Decision: Use regular VACUUM**
```sql
-- 25% bloat is manageable
VACUUM ANALYZE orders;

-- Monitor for improvement
SELECT 
    pg_size_pretty(pg_total_relation_size('orders')) as size,
    n_dead_tup
FROM pg_stat_user_tables
WHERE tablename = 'orders';

-- Result: n_dead_tup dropped to 100
-- File size still 2.3 GB (expected)
-- Space now reusable for new orders
```

**If bloat was 85%:**
```sql
-- Schedule during maintenance window
-- Sunday 3 AM
VACUUM FULL orders;

-- Result:
-- Before: 2.3 GB (150K live, 850K dead)
-- After: 280 MB (150K live, minimal overhead)
-- Space reclaimed: 2.02 GB returned to OS
```

---

### **8. Monitoring and Alerts**

```sql
-- Query to identify tables needing VACUUM vs VACUUM FULL
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as total_size,
    n_live_tup,
    n_dead_tup,
    ROUND(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 2) as dead_pct,
    CASE 
        WHEN n_dead_tup = 0 THEN 'OK - No maintenance needed'
        WHEN ROUND(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 2) < 20 THEN 'OK'
        WHEN ROUND(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 2) < 50 THEN 'Run VACUUM'
        WHEN ROUND(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 2) < 80 THEN 'Run VACUUM (urgent)'
        ELSE 'Consider VACUUM FULL (extreme bloat)'
    END as recommendation,
    last_vacuum,
    last_autovacuum
FROM pg_stat_user_tables
WHERE n_dead_tup > 1000
ORDER BY 
    ROUND(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 2) DESC,
    pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

---

### **9. Best Practices**

**For VACUUM:**
1. ✅ Run regularly (let autovacuum handle it)
2. ✅ Manual VACUUM after bulk DELETE/UPDATE
3. ✅ Use VACUUM ANALYZE to update stats too
4. ✅ No downtime concerns
5. ✅ Safe for production 24/7

**For VACUUM FULL:**
1. ⚠️ Only during maintenance windows
2. ⚠️ Ensure 2x disk space available
3. ⚠️ Test on dev/staging first
4. ⚠️ Monitor progress with `pg_stat_progress_vacuum`
5. ⚠️ Consider `pg_repack` as alternative
6. ⚠️ Document downtime impact
7. ⚠️ Have rollback plan ready

---

### **Interview Tips:**
- **Key difference**: "VACUUM marks space reusable without shrinking files; VACUUM FULL rewrites table and returns space to OS"
- **Locking**: "VACUUM allows concurrent access; VACUUM FULL takes exclusive lock - table unavailable"
- **When to use**: "Use VACUUM for routine maintenance; VACUUM FULL only for extreme bloat (>80%) during maintenance windows"
- **Real example**: "Orders table bloated to 2.3 GB with only 280 MB live data. Used VACUUM first - no improvement in file size but queries faster. Scheduled VACUUM FULL during Sunday maintenance - reclaimed 2 GB to OS"
- **Performance**: "VACUUM took 4 minutes; VACUUM FULL took 2 hours for same table"
- **Alternatives**: "Mentioned pg_repack for zero-downtime table rebuilding - works like VACUUM FULL without locking"
- **Production tip**: "Never run VACUUM FULL on production without maintenance window - it locks table completely"

</details>

<details>
<summary><strong>47. What is autovacuum?</strong></summary>

### **Answer:**

Autovacuum is PostgreSQL's automatic maintenance daemon that runs VACUUM and ANALYZE operations on tables without manual intervention. It monitors table activity and triggers cleanup when certain thresholds are met, ensuring database health and performance.

---

### **1. What Autovacuum Does**

#### **A. Automatic Maintenance Operations**

Autovacuum automatically performs:
1. **VACUUM** - Reclaims space from dead tuples
2. **ANALYZE** - Updates table statistics for query planner
3. **VACUUM FREEZE** - Prevents transaction ID wraparound

```sql
-- Check if autovacuum is enabled
SHOW autovacuum;
-- Result: on (should always be on!)

-- Check autovacuum worker processes
SHOW autovacuum_max_workers;
-- Default: 3 (can run 3 tables simultaneously)
```

#### **B. Monitoring Autovacuum Activity**

```sql
-- See when autovacuum last ran
SELECT 
    schemaname,
    tablename,
    last_vacuum,
    last_autovacuum,
    last_analyze,
    last_autoanalyze,
    n_live_tup,
    n_dead_tup,
    ROUND(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 2) as dead_pct
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC
LIMIT 20;
```

**Output:**
```
tablename | last_vacuum         | last_autovacuum     | n_live_tup | n_dead_tup | dead_pct
----------+---------------------+---------------------+------------+------------+---------
orders    | NULL                | 2024-12-24 08:15:23 | 100,000    | 1,234      | 1.22
logs      | 2024-12-23 02:00:00 | 2024-12-24 09:30:45 | 500,000    | 45,678     | 8.37
sessions  | NULL                | 2024-12-24 10:05:12 | 25,000     | 567        | 2.22
```

**Interpretation:**
- **last_vacuum = NULL**: Only autovacuum has run (no manual VACUUM)
- **last_autovacuum recent**: Autovacuum working properly
- **Both NULL**: Table not vacuumed yet (new table or autovacuum disabled)

---

### **2. How Autovacuum Decides When to Run**

#### **A. Trigger Thresholds**

Autovacuum runs VACUUM when:
```
dead_tuples > autovacuum_vacuum_threshold + (autovacuum_vacuum_scale_factor × table_size)
```

**Default values:**
```sql
SHOW autovacuum_vacuum_threshold;        -- 50 dead tuples
SHOW autovacuum_vacuum_scale_factor;     -- 0.2 (20% of table)
```

**Example calculations:**

**Small table (1,000 rows):**
```
Threshold = 50 + (0.2 × 1000) = 50 + 200 = 250 dead tuples
```
Autovacuum runs when table has 250+ dead tuples (25% of table)

**Medium table (100,000 rows):**
```
Threshold = 50 + (0.2 × 100,000) = 50 + 20,000 = 20,050 dead tuples
```
Autovacuum runs when table has 20,050+ dead tuples (20% of table)

**Large table (10,000,000 rows):**
```
Threshold = 50 + (0.2 × 10,000,000) = 50 + 2,000,000 = 2,000,050 dead tuples
```
Autovacuum runs when table has 2M+ dead tuples (20% of table)

#### **B. ANALYZE Thresholds**

Autovacuum runs ANALYZE when:
```
changed_tuples > autovacuum_analyze_threshold + (autovacuum_analyze_scale_factor × table_size)
```

**Default values:**
```sql
SHOW autovacuum_analyze_threshold;       -- 50 tuples
SHOW autovacuum_analyze_scale_factor;    -- 0.1 (10% of table)
```

**Example:**
```sql
-- Table has 100,000 rows
-- Threshold = 50 + (0.1 × 100,000) = 10,050 changed tuples

-- After 15,000 INSERT/UPDATE/DELETE operations
-- Autovacuum triggers ANALYZE to update statistics
```

#### **C. Transaction ID Wraparound Prevention**

```sql
-- Check transaction age
SELECT 
    datname,
    age(datfrozenxid) as xid_age,
    2147483647 - age(datfrozenxid) as xids_remaining
FROM pg_database
ORDER BY age(datfrozenxid) DESC;
```

**Autovacuum FREEZE triggers:**
```sql
SHOW autovacuum_freeze_max_age;  -- Default: 200,000,000 transactions
```

When table reaches 200M transaction age:
- Autovacuum runs **aggressively** (can't be postponed)
- Runs even if autovacuum is disabled
- Prevents transaction ID wraparound crisis

---

### **3. Autovacuum Configuration**

#### **A. Global Configuration (postgresql.conf)**

```sql
-- Enable/disable autovacuum (should always be on!)
autovacuum = on

-- Maximum workers (parallel autovacuum processes)
autovacuum_max_workers = 3

-- Delay between autovacuum runs on each database
autovacuum_naptime = 1min

-- Cost-based vacuum delay (throttling)
autovacuum_vacuum_cost_delay = 2ms
autovacuum_vacuum_cost_limit = 200

-- Thresholds for triggering VACUUM
autovacuum_vacuum_threshold = 50
autovacuum_vacuum_scale_factor = 0.2

-- Thresholds for triggering ANALYZE
autovacuum_analyze_threshold = 50
autovacuum_analyze_scale_factor = 0.1

-- Transaction ID age thresholds
autovacuum_freeze_max_age = 200000000
autovacuum_multixact_freeze_max_age = 400000000
```

#### **B. Per-Table Configuration**

```sql
-- Override autovacuum settings for specific tables

-- High-write table: vacuum more aggressively
ALTER TABLE sessions SET (
    autovacuum_vacuum_threshold = 100,
    autovacuum_vacuum_scale_factor = 0.05,  -- 5% instead of 20%
    autovacuum_analyze_threshold = 100,
    autovacuum_analyze_scale_factor = 0.05
);

-- Low-priority table: vacuum less often
ALTER TABLE audit_logs SET (
    autovacuum_vacuum_scale_factor = 0.5  -- 50% dead tuples before vacuum
);

-- Disable autovacuum on specific table (not recommended!)
ALTER TABLE temp_staging SET (
    autovacuum_enabled = false
);

-- Re-enable
ALTER TABLE temp_staging SET (
    autovacuum_enabled = true
);

-- View table-specific settings
SELECT 
    tablename,
    reloptions
FROM pg_tables
WHERE schemaname = 'public'
  AND reloptions IS NOT NULL;
```

---

### **4. Monitoring Autovacuum**

#### **A. Check Autovacuum Status**

```sql
-- See if autovacuum is currently running
SELECT 
    pid,
    datname,
    usename,
    state,
    query,
    query_start,
    NOW() - query_start AS duration
FROM pg_stat_activity
WHERE query LIKE '%autovacuum%'
  AND query NOT LIKE '%pg_stat_activity%';
```

**Output:**
```
pid   | datname | query                                          | duration
------+---------+------------------------------------------------+---------
12345 | mydb    | autovacuum: VACUUM public.orders (to prevent...) | 00:05:23
12346 | mydb    | autovacuum: VACUUM ANALYZE public.customers    | 00:02:15
```

#### **B. Autovacuum Progress**

```sql
-- Monitor autovacuum progress (PostgreSQL 13+)
SELECT 
    p.pid,
    p.datname,
    p.relid::regclass AS table_name,
    p.phase,
    p.heap_blks_total,
    p.heap_blks_scanned,
    p.heap_blks_vacuumed,
    ROUND(100.0 * p.heap_blks_scanned / NULLIF(p.heap_blks_total, 0), 2) AS pct_complete,
    a.query_start,
    NOW() - a.query_start AS duration
FROM pg_stat_progress_vacuum p
JOIN pg_stat_activity a ON p.pid = a.pid;
```

**Output:**
```
table_name | phase              | heap_blks_scanned | heap_blks_total | pct_complete | duration
-----------+--------------------+-------------------+-----------------+--------------+---------
orders     | scanning heap      | 15000             | 50000           | 30.00        | 00:03:45
customers  | vacuuming indexes  | 8000              | 8000            | 100.00       | 00:01:20
```

#### **C. Autovacuum History**

```sql
-- Tables that haven't been autovacuumed in a while
SELECT 
    schemaname,
    tablename,
    last_autovacuum,
    NOW() - last_autovacuum AS time_since_autovacuum,
    n_dead_tup,
    ROUND(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 2) as dead_pct
FROM pg_stat_user_tables
WHERE last_autovacuum IS NOT NULL
ORDER BY last_autovacuum ASC
LIMIT 20;
```

**Output:**
```
tablename     | last_autovacuum     | time_since_autovacuum | n_dead_tup | dead_pct
--------------+---------------------+-----------------------+------------+---------
archived_logs | 2024-11-15 08:30:00 | 39 days 03:45:23      | 450,000    | 85.23
old_sessions  | 2024-12-20 14:22:00 | 3 days 22:15:45       | 12,345     | 15.67
```

**Red flags:**
- **Long time since autovacuum**: Table may not meet threshold (scale factor too high)
- **High dead_pct**: Autovacuum not keeping up with workload

---

### **5. Common Autovacuum Problems**

#### **A. Autovacuum Not Running**

**Problem:**
```sql
SELECT 
    tablename,
    last_autovacuum,
    n_dead_tup,
    ROUND(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 2) as dead_pct
FROM pg_stat_user_tables
WHERE tablename = 'orders';
```
```
tablename | last_autovacuum | n_dead_tup | dead_pct
----------+-----------------+------------+---------
orders    | NULL            | 850,000    | 89.47    ← Never autovacuumed!
```

**Causes & Solutions:**

1. **Autovacuum disabled:**
```sql
SHOW autovacuum;
-- Result: off

-- Fix: Enable in postgresql.conf
ALTER SYSTEM SET autovacuum = on;
SELECT pg_reload_conf();
```

2. **Long-running transactions block autovacuum:**
```sql
-- Find blocking transactions
SELECT 
    pid,
    NOW() - xact_start AS transaction_age,
    state,
    query
FROM pg_stat_activity
WHERE state <> 'idle'
  AND (NOW() - xact_start) > INTERVAL '1 hour'
ORDER BY xact_start;

-- Autovacuum can't remove rows visible to old transactions
-- Solution: Terminate or commit long transactions
SELECT pg_terminate_backend(12345);  -- Kill blocking transaction
```

3. **Threshold not met:**
```sql
-- Table has 10M rows, 500K dead tuples
-- Threshold = 50 + (0.2 × 10,000,000) = 2,000,050
-- 500K < 2M → Autovacuum won't run

-- Solution: Lower scale factor for this table
ALTER TABLE orders SET (
    autovacuum_vacuum_scale_factor = 0.05  -- 5% instead of 20%
);
-- New threshold = 50 + (0.05 × 10,000,000) = 500,050
-- 500K < 500K → Will trigger soon
```

#### **B. Autovacuum Too Slow**

**Problem:**
```sql
-- Autovacuum running but taking hours
SELECT 
    pid,
    NOW() - query_start AS duration,
    query
FROM pg_stat_activity
WHERE query LIKE '%autovacuum%';
```
```
duration  | query
----------+------------------------------------------
04:32:15  | autovacuum: VACUUM public.orders
```

**Causes & Solutions:**

1. **Autovacuum throttled (cost-based delay):**
```sql
SHOW autovacuum_vacuum_cost_delay;  -- 2ms (default)
SHOW autovacuum_vacuum_cost_limit;  -- 200 (default)

-- Make autovacuum faster (more aggressive)
ALTER SYSTEM SET autovacuum_vacuum_cost_delay = 0;  -- No delay
ALTER SYSTEM SET autovacuum_vacuum_cost_limit = -1; -- No limit
SELECT pg_reload_conf();

-- Or per-table
ALTER TABLE orders SET (
    autovacuum_vacuum_cost_delay = 0
);
```

2. **Not enough workers:**
```sql
SHOW autovacuum_max_workers;  -- 3

-- Increase workers
ALTER SYSTEM SET autovacuum_max_workers = 6;
-- Requires restart
pg_ctl restart
```

3. **Increase work_mem:**
```sql
-- Autovacuum uses work_mem for internal operations
ALTER SYSTEM SET autovacuum_work_mem = 512MB;  -- Default: -1 (uses work_mem)
SELECT pg_reload_conf();
```

#### **C. Autovacuum Cancellation**

**Problem:**
```sql
-- Logs show:
-- ERROR: canceling autovacuum task
-- CONTEXT: automatic vacuum of table "mydb.public.orders"
```

**Causes:**
- User sessions requesting locks conflict with autovacuum
- Autovacuum auto-cancels to avoid blocking user queries

**Solution 1: Make autovacuum less interruptible:**
```sql
-- Per-table: Don't cancel autovacuum easily
ALTER TABLE orders SET (
    autovacuum_vacuum_cost_delay = 0  -- Run fast, finish quickly
);
```

**Solution 2: Schedule manual VACUUM during low-traffic:**
```sql
-- Run manual VACUUM during off-peak hours
-- Manual VACUUM won't auto-cancel
SELECT cron.schedule('vacuum-orders', '0 2 * * *', 'VACUUM ANALYZE orders');
```

---

### **6. Tuning Autovacuum for Different Workloads**

#### **A. High-Write Tables (frequent UPDATEs)**

```sql
-- Example: sessions table updated every second
ALTER TABLE sessions SET (
    autovacuum_vacuum_threshold = 100,
    autovacuum_vacuum_scale_factor = 0.02,  -- Vacuum at 2% dead tuples
    autovacuum_analyze_threshold = 100,
    autovacuum_analyze_scale_factor = 0.02,
    autovacuum_vacuum_cost_delay = 0  -- Fast vacuum, no delay
);
```

#### **B. Append-Only Tables (mostly INSERTs)**

```sql
-- Example: audit_logs (rarely updated/deleted)
ALTER TABLE audit_logs SET (
    autovacuum_vacuum_scale_factor = 0.5,  -- Less frequent vacuum
    autovacuum_analyze_scale_factor = 0.05  -- More frequent analyze (for query planning)
);
```

#### **C. Large Tables (>100 GB)**

```sql
-- Example: big_table (10 TB)
ALTER TABLE big_table SET (
    autovacuum_vacuum_scale_factor = 0.01,  -- 1% = 100 GB dead tuples before vacuum
    autovacuum_vacuum_cost_delay = 0,
    autovacuum_work_mem = '2GB'  -- More memory for faster vacuum
);
```

#### **D. Small, Highly Active Tables**

```sql
-- Example: active_sessions (1000 rows, updated constantly)
ALTER TABLE active_sessions SET (
    autovacuum_vacuum_threshold = 50,
    autovacuum_vacuum_scale_factor = 0.1,  -- Vacuum at 10% (100 dead tuples)
    autovacuum_analyze_threshold = 50,
    autovacuum_analyze_scale_factor = 0.1
);
```

---

### **7. Autovacuum vs Manual VACUUM**

| Scenario | Use Autovacuum | Use Manual VACUUM |
|----------|----------------|-------------------|
| **Daily operations** | ✅ Yes - automatic | ❌ Not needed |
| **After bulk DELETE** | ❌ Too slow (waits for threshold) | ✅ Yes - immediate cleanup |
| **Before major query** | ❌ May not have run yet | ✅ Yes - ensure fresh stats |
| **24/7 production** | ✅ Yes - no disruption | ⚠️ Optional (VACUUM is non-blocking) |
| **Extreme bloat** | ❌ Won't run VACUUM FULL | ✅ Yes - manual VACUUM FULL needed |
| **Transaction wraparound** | ✅ Yes - automatic FREEZE | ⚠️ Manual if approaching limit |

---

### **8. Real-World Example**

**Scenario: E-commerce site with orders table**

**Initial state (autovacuum with defaults):**
```sql
-- 1 million rows, 10,000 UPDATEs per hour
-- Threshold = 50 + (0.2 × 1,000,000) = 200,050 dead tuples

-- After 20 hours: 200,000 dead tuples
SELECT n_live_tup, n_dead_tup FROM pg_stat_user_tables WHERE tablename = 'orders';
-- n_live_tup = 1,000,000, n_dead_tup = 200,000 (16.7% dead)

-- Autovacuum hasn't run yet (threshold is 200,050)
-- Queries slowing down due to dead tuple scanning
```

**Problem: Query performance degraded**
```sql
SELECT * FROM orders WHERE status = 'pending';
-- Time: 1500ms (was 50ms a week ago)
```

**Solution: Tune autovacuum for this table**
```sql
-- Lower threshold for faster cleanup
ALTER TABLE orders SET (
    autovacuum_vacuum_scale_factor = 0.05,  -- 5% instead of 20%
    autovacuum_vacuum_cost_delay = 0  -- No throttling
);

-- New threshold = 50 + (0.05 × 1,000,000) = 50,050 dead tuples
-- Autovacuum now runs when table hits 50K dead tuples (5%)
```

**Result:**
```sql
-- Autovacuum runs every ~5 hours instead of every ~20 hours
-- Dead tuples kept below 5% consistently

SELECT n_live_tup, n_dead_tup FROM pg_stat_user_tables WHERE tablename = 'orders';
-- n_live_tup = 1,000,000, n_dead_tup = 45,000 (4.3% dead)

-- Query performance restored
SELECT * FROM orders WHERE status = 'pending';
-- Time: 52ms (back to normal)
```

---

### **9. Best Practices**

1. ✅ **Always keep autovacuum enabled** - It's essential for database health
2. ✅ **Monitor autovacuum activity** - Check pg_stat_user_tables regularly
3. ✅ **Tune per-table** - Different tables need different settings
4. ✅ **Increase workers for large databases** - Default 3 may not be enough
5. ✅ **Lower scale_factor for high-write tables** - Vacuum more frequently
6. ✅ **Manual VACUUM after bulk operations** - Don't wait for autovacuum
7. ✅ **Monitor transaction ID age** - Prevent wraparound
8. ⚠️ **Don't disable autovacuum** - Even "temporarily" can cause crisis
9. ⚠️ **Watch for long-running transactions** - They block autovacuum
10. ⚠️ **Alert on autovacuum failures** - Set up monitoring/alerts

---

### **Interview Tips:**
- **Definition**: "Autovacuum is PostgreSQL's automatic maintenance daemon that runs VACUUM and ANALYZE without manual intervention"
- **How it works**: "Monitors dead tuple counts; triggers when exceeding threshold + scale_factor × table_size"
- **Default threshold**: "50 + (20% of table size) - so 100K row table needs 20,050 dead tuples before autovacuum runs"
- **Real problem**: "Orders table queries slowed from 50ms to 1.5s. Found 200K dead tuples (16%). Lowered autovacuum_vacuum_scale_factor from 0.2 to 0.05. Autovacuum now runs at 5% instead of 20%. Queries back to 52ms"
- **Configuration**: "Can configure globally in postgresql.conf or per-table with ALTER TABLE SET"
- **Common issue**: "Long-running transactions block autovacuum from cleaning up dead tuples"
- **Never disable**: "Disabling autovacuum can lead to transaction ID wraparound and database shutdown"
- **Tuning**: "High-write tables need aggressive autovacuum (5% threshold); append-only tables can use 50%"

</details>

<details>
<summary><strong>48. How do you monitor database performance?</strong></summary>

### **Answer:**

Monitoring database performance in PostgreSQL involves tracking key metrics, identifying bottlenecks, and proactively addressing issues before they impact users. Here's a comprehensive guide to effective database monitoring.

---

### **1. Essential Monitoring Categories**

#### **A. Query Performance**
- Slow queries identification
- Query execution plans
- Query frequency and patterns

#### **B. Resource Utilization**
- CPU usage
- Memory consumption
- Disk I/O
- Network traffic

#### **C. Database Health**
- Connection counts
- Lock conflicts
- Dead tuples and bloat
- Transaction ID age

#### **D. Replication & Availability**
- Replication lag
- Standby status
- Backup status

---

### **2. Built-in Monitoring Views**

#### **A. pg_stat_activity - Active Connections**

```sql
-- View current database activity
SELECT 
    pid,
    usename,
    application_name,
    client_addr,
    state,
    query_start,
    NOW() - query_start AS duration,
    wait_event_type,
    wait_event,
    LEFT(query, 80) AS query_preview
FROM pg_stat_activity
WHERE state != 'idle'
ORDER BY duration DESC;
```

**Output:**
```
pid   | usename | state  | duration  | wait_event_type | query_preview
------+---------+--------+-----------+-----------------+------------------
12345 | app_user| active | 00:05:23  | IO              | SELECT * FROM orders WHERE...
12346 | app_user| active | 00:00:12  | NULL            | UPDATE inventory SET...
12347 | app_user| idle in transaction | 00:15:45 | Client | BEGIN;
```

**Key metrics:**
- **state = 'active'**: Currently executing queries
- **state = 'idle in transaction'**: ⚠️ Holding locks, blocking others
- **wait_event_type**: What query is waiting for (IO, Lock, Client)
- **Long duration**: Potential slow query or blocking

**Find long-running queries:**
```sql
-- Queries running longer than 5 minutes
SELECT 
    pid,
    NOW() - query_start AS duration,
    usename,
    query
FROM pg_stat_activity
WHERE state = 'active'
  AND (NOW() - query_start) > INTERVAL '5 minutes'
ORDER BY duration DESC;
```

**Find idle in transaction (connection leak):**
```sql
-- Connections holding transactions open without activity
SELECT 
    pid,
    usename,
    application_name,
    NOW() - xact_start AS transaction_age,
    NOW() - state_change AS idle_time,
    query
FROM pg_stat_activity
WHERE state = 'idle in transaction'
  AND (NOW() - state_change) > INTERVAL '1 minute'
ORDER BY transaction_age DESC;
```

#### **B. pg_stat_database - Database-Wide Statistics**

```sql
-- Database-level metrics
SELECT 
    datname,
    numbackends,  -- Current connections
    xact_commit,  -- Committed transactions
    xact_rollback,  -- Rolled back transactions
    blks_read,    -- Disk blocks read
    blks_hit,     -- Buffer cache hits
    tup_returned, -- Rows returned
    tup_fetched,  -- Rows fetched
    tup_inserted, -- Rows inserted
    tup_updated,  -- Rows updated
    tup_deleted,  -- Rows deleted
    conflicts,    -- Query conflicts
    temp_files,   -- Temp files created
    temp_bytes,   -- Temp file size
    deadlocks,    -- Deadlock count
    blk_read_time,  -- Time reading from disk (ms)
    blk_write_time  -- Time writing to disk (ms)
FROM pg_stat_database
WHERE datname = current_database();
```

**Calculate cache hit ratio:**
```sql
-- Cache hit ratio (should be >95%)
SELECT 
    datname,
    numbackends AS connections,
    ROUND(100.0 * blks_hit / NULLIF(blks_hit + blks_read, 0), 2) AS cache_hit_ratio,
    pg_size_pretty(pg_database_size(datname)) AS database_size
FROM pg_stat_database
WHERE datname = current_database();
```

**Output:**
```
datname | connections | cache_hit_ratio | database_size
--------+-------------+-----------------+--------------
mydb    | 45          | 98.75           | 25 GB
```

**Interpretation:**
- **cache_hit_ratio < 90%**: Need more memory or queries hitting cold data
- **cache_hit_ratio 95-99%**: Good
- **cache_hit_ratio > 99%**: Excellent

#### **C. pg_stat_user_tables - Table-Level Statistics**

```sql
-- Table access patterns
SELECT 
    schemaname,
    tablename,
    seq_scan,           -- Sequential scans
    seq_tup_read,       -- Rows read by seq scans
    idx_scan,           -- Index scans
    idx_tup_fetch,      -- Rows fetched by index scans
    n_tup_ins,          -- Rows inserted
    n_tup_upd,          -- Rows updated
    n_tup_del,          -- Rows deleted
    n_live_tup,         -- Live rows
    n_dead_tup,         -- Dead rows
    ROUND(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 2) as dead_pct,
    last_vacuum,
    last_autovacuum,
    last_analyze,
    last_autoanalyze
FROM pg_stat_user_tables
ORDER BY seq_scan DESC
LIMIT 20;
```

**Identify tables needing indexes:**
```sql
-- Tables with high sequential scans
SELECT 
    schemaname,
    tablename,
    seq_scan,
    seq_tup_read,
    idx_scan,
    n_live_tup,
    ROUND(seq_tup_read::numeric / NULLIF(seq_scan, 0), 0) AS avg_rows_per_seqscan
FROM pg_stat_user_tables
WHERE seq_scan > 0
  AND schemaname NOT IN ('pg_catalog', 'information_schema')
  AND n_live_tup > 10000  -- Only large tables
ORDER BY seq_scan DESC
LIMIT 20;
```

**Output:**
```
tablename | seq_scan | seq_tup_read | idx_scan | avg_rows_per_seqscan
----------+----------+--------------+----------+---------------------
orders    | 15234    | 1,523,400,000| 234      | 100,000  ← Needs index!
customers | 234      | 4,680,000    | 45678    | 20,000   ← OK
```

#### **D. pg_stat_user_indexes - Index Usage**

```sql
-- Index usage statistics
SELECT 
    schemaname,
    tablename,
    indexrelname,
    idx_scan,           -- Times index used
    idx_tup_read,       -- Tuples read from index
    idx_tup_fetch,      -- Tuples fetched from table
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
ORDER BY idx_scan ASC
LIMIT 20;
```

**Find unused indexes:**
```sql
-- Indexes never used (consider dropping)
SELECT 
    schemaname,
    tablename,
    indexrelname,
    idx_scan,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
WHERE idx_scan = 0
  AND indexrelname NOT LIKE '%_pkey'  -- Exclude primary keys
  AND schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY pg_relation_size(indexrelid) DESC;
```

**Output:**
```
tablename | indexrelname              | idx_scan | index_size
----------+---------------------------+----------+-----------
orders    | idx_orders_old_status     | 0        | 450 MB  ← Drop this!
customers | idx_customers_temp        | 0        | 120 MB  ← Drop this!
```

#### **E. pg_statio_user_tables - I/O Statistics**

```sql
-- Table I/O patterns
SELECT 
    schemaname,
    tablename,
    heap_blks_read,     -- Blocks read from disk
    heap_blks_hit,      -- Blocks from cache
    ROUND(100.0 * heap_blks_hit / NULLIF(heap_blks_hit + heap_blks_read, 0), 2) AS cache_hit_ratio,
    idx_blks_read,      -- Index blocks from disk
    idx_blks_hit        -- Index blocks from cache
FROM pg_statio_user_tables
WHERE (heap_blks_read + heap_blks_hit) > 0
ORDER BY heap_blks_read DESC
LIMIT 20;
```

**Output:**
```
tablename | heap_blks_read | heap_blks_hit | cache_hit_ratio
----------+----------------+---------------+----------------
orders    | 1,234,567      | 45,678,901    | 97.37  ← Good
logs      | 5,678,901      | 1,234,567     | 17.86  ← Poor! Cold data
```

---

### **3. Query Performance Monitoring**

#### **A. pg_stat_statements Extension**

```sql
-- Enable extension (once)
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- Top 10 slowest queries by total time
SELECT 
    query,
    calls,
    total_exec_time,
    mean_exec_time,
    max_exec_time,
    stddev_exec_time,
    rows,
    100.0 * shared_blks_hit / NULLIF(shared_blks_hit + shared_blks_read, 0) AS cache_hit_ratio
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;
```

**Output:**
```
query                                  | calls | total_exec_time | mean_exec_time | max_exec_time
--------------------------------------+-------+-----------------+----------------+--------------
SELECT * FROM orders WHERE status=$1  | 15234 | 145234.567      | 9.534          | 1234.567
UPDATE inventory SET quantity=$1...   | 8945  | 98765.432       | 11.045         | 567.890
```

**Queries with high variance (unpredictable):**
```sql
SELECT 
    query,
    calls,
    mean_exec_time,
    stddev_exec_time,
    max_exec_time,
    stddev_exec_time / NULLIF(mean_exec_time, 0) AS coefficient_of_variation
FROM pg_stat_statements
WHERE calls > 100
ORDER BY coefficient_of_variation DESC
LIMIT 10;
```

**Reset statistics:**
```sql
-- Reset to establish new baseline
SELECT pg_stat_statements_reset();
```

#### **B. auto_explain Extension**

```sql
-- Automatically log slow queries with EXPLAIN output
LOAD 'auto_explain';

-- Configure thresholds
SET auto_explain.log_min_duration = 1000;  -- Log queries >1 second
SET auto_explain.log_analyze = true;
SET auto_explain.log_buffers = true;
SET auto_explain.log_timing = true;
SET auto_explain.log_verbose = true;
SET auto_explain.log_nested_statements = true;

-- Now slow queries automatically appear in PostgreSQL logs with EXPLAIN ANALYZE
```

**In postgresql.conf (permanent):**
```ini
shared_preload_libraries = 'pg_stat_statements,auto_explain'
auto_explain.log_min_duration = 1000
auto_explain.log_analyze = true
auto_explain.log_buffers = true
```

---

### **4. Lock Monitoring**

#### **A. View Current Locks**

```sql
-- See all current locks
SELECT 
    l.locktype,
    l.database,
    l.relation::regclass AS table_name,
    l.page,
    l.tuple,
    l.virtualxid,
    l.transactionid,
    l.mode,
    l.granted,
    a.pid,
    a.usename,
    a.query,
    NOW() - a.query_start AS query_duration
FROM pg_locks l
LEFT JOIN pg_stat_activity a ON l.pid = a.pid
WHERE l.granted = true
ORDER BY a.query_start;
```

#### **B. Find Blocking Queries**

```sql
-- Identify blocked and blocking queries
SELECT 
    blocked_locks.pid AS blocked_pid,
    blocked_activity.usename AS blocked_user,
    blocking_locks.pid AS blocking_pid,
    blocking_activity.usename AS blocking_user,
    blocked_activity.query AS blocked_statement,
    blocking_activity.query AS blocking_statement,
    blocked_activity.application_name AS blocked_app
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks 
    ON blocking_locks.locktype = blocked_locks.locktype
    AND blocking_locks.database IS NOT DISTINCT FROM blocked_locks.database
    AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
    AND blocking_locks.page IS NOT DISTINCT FROM blocked_locks.page
    AND blocking_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
    AND blocking_locks.virtualxid IS NOT DISTINCT FROM blocked_locks.virtualxid
    AND blocking_locks.transactionid IS NOT DISTINCT FROM blocked_locks.transactionid
    AND blocking_locks.classid IS NOT DISTINCT FROM blocked_locks.classid
    AND blocking_locks.objid IS NOT DISTINCT FROM blocked_locks.objid
    AND blocking_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
    AND blocking_locks.pid != blocked_locks.pid
JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted;
```

**Output:**
```
blocked_pid | blocking_pid | blocked_statement                    | blocking_statement
-----------+--------------+--------------------------------------+---------------------------
12345       | 12340        | UPDATE orders SET status='shipped'   | BEGIN; (idle in transaction)
```

**Terminate blocking query:**
```sql
-- Gently cancel query
SELECT pg_cancel_backend(12340);

-- Force terminate (if cancel doesn't work)
SELECT pg_terminate_backend(12340);
```

---

### **5. Connection Monitoring**

#### **A. Connection Count**

```sql
-- Current connections by database
SELECT 
    datname,
    COUNT(*) as connection_count,
    COUNT(*) FILTER (WHERE state = 'active') as active,
    COUNT(*) FILTER (WHERE state = 'idle') as idle,
    COUNT(*) FILTER (WHERE state = 'idle in transaction') as idle_in_transaction
FROM pg_stat_activity
GROUP BY datname
ORDER BY connection_count DESC;
```

**Output:**
```
datname | connection_count | active | idle | idle_in_transaction
--------+------------------+--------+------+--------------------
mydb    | 95               | 12     | 80   | 3  ← Monitor this!
postgres| 2                | 0      | 2    | 0
```

**Check connection limit:**
```sql
-- View max connections
SHOW max_connections;  -- Default: 100

-- Current usage
SELECT 
    COUNT(*) as current_connections,
    (SELECT setting::int FROM pg_settings WHERE name = 'max_connections') as max_connections,
    ROUND(100.0 * COUNT(*) / (SELECT setting::int FROM pg_settings WHERE name = 'max_connections'), 2) as usage_pct
FROM pg_stat_activity;
```

**Output:**
```
current_connections | max_connections | usage_pct
-------------------+-----------------+----------
95                  | 100             | 95.00  ← Near limit!
```

#### **B. Connection Sources**

```sql
-- Connections by application
SELECT 
    application_name,
    client_addr,
    COUNT(*) as connection_count,
    MAX(NOW() - query_start) as longest_query
FROM pg_stat_activity
WHERE pid != pg_backend_pid()  -- Exclude current connection
GROUP BY application_name, client_addr
ORDER BY connection_count DESC;
```

---

### **6. Disk Space Monitoring**

#### **A. Database Size**

```sql
-- Database sizes
SELECT 
    datname,
    pg_size_pretty(pg_database_size(datname)) AS size
FROM pg_database
ORDER BY pg_database_size(datname) DESC;
```

**Output:**
```
datname  | size
---------+-------
mydb     | 45 GB
testdb   | 12 GB
postgres | 8 MB
```

#### **B. Table Sizes**

```sql
-- Largest tables
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS total_size,
    pg_size_pretty(pg_relation_size(schemaname||'.'||tablename)) AS table_size,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename) - pg_relation_size(schemaname||'.'||tablename)) AS index_size
FROM pg_tables
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
LIMIT 20;
```

**Output:**
```
tablename | total_size | table_size | index_size
----------+------------+------------+-----------
orders    | 15 GB      | 10 GB      | 5 GB
logs      | 8 GB       | 7 GB       | 1 GB
customers | 2 GB       | 1.5 GB     | 500 MB
```

#### **C. Table Bloat Estimation**

```sql
-- Estimated bloat per table
SELECT 
    schemaname,
    tablename,
    ROUND(100 * n_dead_tup::numeric / NULLIF(n_live_tup + n_dead_tup, 0), 2) AS bloat_pct,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS total_size,
    n_live_tup,
    n_dead_tup,
    last_vacuum,
    last_autovacuum
FROM pg_stat_user_tables
WHERE n_dead_tup > 0
ORDER BY n_dead_tup DESC
LIMIT 20;
```

---

### **7. Replication Monitoring**

#### **A. Replication Status (Primary)**

```sql
-- View replication slots and lag
SELECT 
    slot_name,
    slot_type,
    database,
    active,
    pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn) AS replication_lag_bytes,
    pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn)) AS lag_size
FROM pg_replication_slots;
```

**Replication connections:**
```sql
SELECT 
    client_addr,
    state,
    sync_state,
    pg_wal_lsn_diff(sent_lsn, write_lsn) AS write_lag_bytes,
    pg_wal_lsn_diff(write_lsn, flush_lsn) AS flush_lag_bytes,
    pg_wal_lsn_diff(flush_lsn, replay_lsn) AS replay_lag_bytes
FROM pg_stat_replication;
```

#### **B. Replication Status (Standby)**

```sql
-- Check if recovery/replication is active
SELECT pg_is_in_recovery();
-- Result: true (standby), false (primary)

-- Replication lag on standby
SELECT 
    NOW() - pg_last_xact_replay_timestamp() AS replication_lag;
```

---

### **8. Alerting Thresholds**

#### **A. Critical Alerts**

```sql
-- Alert when connection limit >90%
SELECT CASE 
    WHEN usage_pct > 90 THEN 'CRITICAL: Connection limit nearly reached'
    WHEN usage_pct > 75 THEN 'WARNING: High connection usage'
    ELSE 'OK'
END AS alert_status
FROM (
    SELECT ROUND(100.0 * COUNT(*) / (SELECT setting::int FROM pg_settings WHERE name = 'max_connections'), 2) as usage_pct
    FROM pg_stat_activity
) sub;

-- Alert when cache hit ratio <90%
SELECT CASE 
    WHEN cache_hit_ratio < 90 THEN 'CRITICAL: Low cache hit ratio'
    WHEN cache_hit_ratio < 95 THEN 'WARNING: Suboptimal cache hit ratio'
    ELSE 'OK'
END AS alert_status
FROM (
    SELECT ROUND(100.0 * blks_hit / NULLIF(blks_hit + blks_read, 0), 2) AS cache_hit_ratio
    FROM pg_stat_database
    WHERE datname = current_database()
) sub;

-- Alert when bloat >30%
SELECT 
    tablename,
    'WARNING: High bloat - consider VACUUM' AS alert
FROM pg_stat_user_tables
WHERE ROUND(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 2) > 30
  AND n_dead_tup > 10000;

-- Alert when transaction ID age >1 billion
SELECT 
    datname,
    'CRITICAL: Approaching transaction wraparound' AS alert
FROM pg_database
WHERE age(datfrozenxid) > 1000000000;
```

---

### **9. Monitoring Tools**

#### **A. Built-in Tools**

**psql commands:**
```sql
\l+              -- List databases with sizes
\dt+             -- List tables with sizes
\di+             -- List indexes with sizes
\x auto          -- Expanded display for better readability
\timing on       -- Show query execution time
```

**pg_top / pg_activity:**
```bash
# Real-time activity monitor (like top for PostgreSQL)
pg_top -d mydb

# Or pg_activity (Python-based)
pg_activity -d mydb
```

#### **B. External Monitoring Solutions**

**Prometheus + Grafana:**
- postgres_exporter for metrics collection
- Grafana dashboards for visualization
- Alertmanager for notifications

**pgBadger:**
```bash
# Log analyzer for PostgreSQL
pgbadger /var/log/postgresql/postgresql.log -o report.html
```

**pgAdmin:**
- GUI tool with built-in monitoring dashboards
- Real-time query monitoring
- Server activity graphs

**Datadog / New Relic / AppDynamics:**
- Commercial APM solutions
- PostgreSQL integration
- Automatic anomaly detection

---

### **10. Monitoring Best Practices**

1. ✅ **Monitor continuously** - Set up automated monitoring, don't rely on manual checks
2. ✅ **Baseline metrics** - Know your normal patterns to detect anomalies
3. ✅ **Alert on trends** - Not just absolute values (e.g., growing connection count)
4. ✅ **Query performance** - Use pg_stat_statements for identifying slow queries
5. ✅ **Cache hit ratio** - Target >95% for good performance
6. ✅ **Connection pooling** - Monitor connection counts, use pgBouncer
7. ✅ **Bloat monitoring** - Alert when tables >20-30% dead tuples
8. ✅ **Lock monitoring** - Identify blocking queries quickly
9. ✅ **Replication lag** - Critical for high availability setups
10. ✅ **Regular reviews** - Weekly review of slow queries and resource usage

---

### **Interview Tips:**
- **Tools**: "Use pg_stat_statements for query performance, pg_stat_activity for connections, pg_stat_user_tables for table health"
- **Key metrics**: "Cache hit ratio (>95%), connection usage (<80% of max), bloat percentage (<20%), query execution time"
- **Real example**: "Set up monitoring that alerted when orders table cache hit ratio dropped from 98% to 75%. Found missing index on new query pattern. Added index, ratio back to 97%"
- **Proactive**: "Don't wait for users to complain - monitor trends like growing connection counts or increasing query times"
- **Blocking queries**: "Use pg_locks joined with pg_stat_activity to identify blocked queries and their blocking counterparts"
- **External tools**: "Grafana + postgres_exporter for visualization, pgBadger for log analysis, pgAdmin for GUI monitoring"
- **Production practice**: "Created alert when transaction ID age >1 billion to prevent wraparound - runs daily check"

</details>

<details>
<summary><strong>49. What are table statistics and how do they affect query planning?</strong></summary>

### **Answer:**

Table statistics in PostgreSQL are metadata about the distribution and characteristics of data in tables and indexes. The query planner uses these statistics to estimate the cost of different execution plans and choose the most efficient one. Accurate statistics are crucial for optimal query performance.

---

### **1. What Are Table Statistics?**

Statistics include information about:
- **Table size** - Number of pages (blocks) and rows
- **Column distribution** - Distinct values, most common values, histograms
- **Correlation** - Physical row order vs logical order
- **NULL percentage** - How many NULLs in each column
- **Data types** - Average width of variable-length columns

---

### **2. How Statistics Are Collected**

#### **A. ANALYZE Command**

```sql
-- Analyze single table
ANALYZE orders;

-- Analyze specific columns
ANALYZE orders (status, order_date);

-- Analyze all tables in database
ANALYZE;

-- Verbose output
ANALYZE VERBOSE orders;
```

**Output:**
```
INFO:  analyzing "public.orders"
INFO:  "orders": scanned 3000 of 50000 pages, containing 60000 live rows and 5000 dead rows
       60000 rows in sample, 1000000 estimated total rows
```

**How ANALYZE works:**
1. Samples a subset of table pages (default: 30,000 pages or ~240 MB)
2. Analyzes sampled data to compute statistics
3. Extrapolates to estimate full table characteristics
4. Stores statistics in `pg_statistic` system catalog

#### **B. Autovacuum (Automatic)**

```sql
-- Autovacuum runs ANALYZE automatically when:
-- changes > autovacuum_analyze_threshold + (autovacuum_analyze_scale_factor × table_size)

-- Default: 50 + (0.1 × table_size) = threshold changes

-- Check last analyze time
SELECT 
    schemaname,
    tablename,
    last_analyze,
    last_autoanalyze,
    n_mod_since_analyze  -- Changes since last analyze
FROM pg_stat_user_tables
ORDER BY n_mod_since_analyze DESC;
```

**Output:**
```
tablename | last_analyze        | last_autoanalyze    | n_mod_since_analyze
----------+---------------------+---------------------+--------------------
orders    | 2024-12-20 10:00:00 | 2024-12-24 08:15:23 | 234
customers | NULL                | 2024-12-24 06:30:00 | 45
products  | 2024-12-23 02:00:00 | NULL                | 25678  ← Needs ANALYZE!
```

---

### **3. Viewing Statistics**

#### **A. pg_stats View (Human-Readable)**

```sql
-- View statistics for a specific column
SELECT 
    tablename,
    attname,
    n_distinct,           -- Number of distinct values (-1 = unique, <0 = estimate)
    most_common_vals,     -- Most frequent values
    most_common_freqs,    -- Frequency of most common values
    histogram_bounds,     -- Value distribution
    correlation,          -- Physical vs logical ordering (-1 to 1)
    null_frac             -- Fraction of NULL values
FROM pg_stats
WHERE tablename = 'orders'
  AND attname = 'status';
```

**Output:**
```
attname: status
n_distinct: 5
most_common_vals: {pending, shipped, delivered, cancelled, refunded}
most_common_freqs: {0.15, 0.50, 0.30, 0.03, 0.02}
histogram_bounds: NULL (discrete values, no histogram needed)
correlation: 0.45
null_frac: 0.00
```

**Interpretation:**
- **n_distinct = 5**: Five unique status values
- **most_common_freqs**: 50% are 'shipped', 30% 'delivered', 15% 'pending'
- **correlation = 0.45**: Moderate correlation (physical order somewhat matches status order)
- **null_frac = 0**: No NULL values

**Another example:**
```sql
SELECT 
    attname,
    n_distinct,
    null_frac,
    avg_width,
    correlation
FROM pg_stats
WHERE tablename = 'orders'
  AND attname IN ('order_date', 'total_amount', 'customer_id');
```

**Output:**
```
attname       | n_distinct | null_frac | avg_width | correlation
--------------+------------+-----------+-----------+------------
order_date    | 365        | 0.00      | 4         | 0.98  ← High correlation!
total_amount  | -0.95      | 0.00      | 8         | 0.12  ← Nearly unique
customer_id   | 20000      | 0.00      | 4         | 0.35
```

**Interpretation:**
- **order_date correlation = 0.98**: New orders stored at end of table (sequential)
  - Index scans very efficient (locality)
- **total_amount n_distinct = -0.95**: 95% unique values (high cardinality)
- **customer_id n_distinct = 20000**: 20,000 distinct customers

#### **B. pg_statistic Catalog (System View)**

```sql
-- Raw statistics (more detailed but harder to read)
SELECT 
    starelid::regclass AS tablename,
    staattnum,
    stanumbers1,  -- Frequency statistics
    stavalues1    -- Most common values
FROM pg_statistic
WHERE starelid = 'orders'::regclass;
```

---

### **4. How Statistics Affect Query Planning**

#### **A. Row Count Estimation**

**Example 1: Selectivity Estimation**

```sql
-- Planner estimates how many rows match the condition
EXPLAIN
SELECT * FROM orders WHERE status = 'pending';
```

**With accurate statistics:**
```
Seq Scan on orders  (cost=0.00..1892.50 rows=15000 width=180)
  Filter: (status = 'pending'::text)
```
- **Planner knows**: 15% of rows are 'pending' (from most_common_freqs)
- **Estimate**: 100,000 total rows × 0.15 = 15,000 rows
- **Decision**: Sequential scan (15% of table is high selectivity)

**Without statistics (stale or never run ANALYZE):**
```
Seq Scan on orders  (cost=0.00..1892.50 rows=50000 width=180)
  Filter: (status = 'pending'::text)
```
- **Planner guesses**: 50% of rows might match (no statistics)
- **Estimate**: 100,000 total rows × 0.5 = 50,000 rows
- **Decision**: Sequential scan (50% is very high)

**With correct statistics after ANALYZE:**
```
Index Scan using idx_orders_status on orders  (cost=0.42..567.89 rows=15000 width=180)
  Index Cond: (status = 'pending'::text)
```
- **Estimate**: 15,000 rows (accurate)
- **Decision**: Index scan (15% of table, index is worthwhile)

#### **B. Join Algorithm Selection**

**Example: Customer orders query**

```sql
EXPLAIN
SELECT c.customer_name, o.order_id, o.total_amount
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE c.customer_id = 123;
```

**With accurate statistics:**
```
Nested Loop  (cost=0.84..78.92 rows=25 width=360)
  ->  Index Scan using customers_pkey on customers c
          (cost=0.42..8.44 rows=1 width=180)
        Index Cond: (customer_id = 123)
  ->  Index Scan using idx_orders_customer_id on orders o
          (cost=0.42..70.23 rows=25 width=180)
        Index Cond: (customer_id = 123)
```
- **Planner knows**: Customer 123 has 25 orders (from statistics)
- **Decision**: Nested Loop (efficient for small result sets)
- **Cost**: 78.92

**With stale statistics (actual: 25 orders, estimated: 50,000):**
```
Hash Join  (cost=1234.56..5678.90 rows=50000 width=360)
  Hash Cond: (o.customer_id = c.customer_id)
  ->  Seq Scan on orders o  (cost=0.00..3200.00 rows=50000 width=180)
  ->  Hash  (cost=8.44..8.44 rows=1 width=180)
        ->  Index Scan using customers_pkey on customers c
                (cost=0.42..8.44 rows=1 width=180)
              Index Cond: (customer_id = 123)
```
- **Planner thinks**: 50,000 orders match (wrong!)
- **Decision**: Hash Join (wrong choice for 25 rows)
- **Cost**: 5678.90 (72x more expensive!)
- **Actual performance**: Much slower

#### **C. Index Usage Decisions**

**Example: Range query**

```sql
EXPLAIN
SELECT * FROM orders WHERE order_date >= '2024-01-01';
```

**Scenario 1: Recent date (2% of data)**
```
-- Statistics show: 2% of orders since 2024-01-01
Index Scan using idx_orders_date on orders
    (cost=0.42..234.56 rows=2000 width=180)
  Index Cond: (order_date >= '2024-01-01')
```
- **Estimate**: 2,000 rows (2% of 100,000)
- **Decision**: Index scan (efficient for small percentage)

**Scenario 2: Old date (80% of data)**
```
-- Statistics show: 80% of orders since 2020-01-01
Seq Scan on orders  (cost=0.00..2200.00 rows=80000 width=180)
  Filter: (order_date >= '2020-01-01')
```
- **Estimate**: 80,000 rows (80% of 100,000)
- **Decision**: Sequential scan (index scan would require 80,000 random lookups)

**Without statistics:**
```
-- Planner doesn't know distribution, makes poor guess
Index Scan using idx_orders_date on orders
    (cost=0.42..15234.56 rows=50000 width=180)
  Index Cond: (order_date >= '2020-01-01')
```
- **Wrong decision**: Chose index scan for 80% of table
- **Result**: Much slower than sequential scan

---

### **5. Statistics Configuration**

#### **A. Statistics Target**

```sql
-- View current statistics target
SHOW default_statistics_target;
-- Default: 100 (affects sampling size)

-- Increase globally (more detailed statistics)
ALTER SYSTEM SET default_statistics_target = 1000;
SELECT pg_reload_conf();

-- Increase for specific column (high cardinality)
ALTER TABLE orders ALTER COLUMN status SET STATISTICS 1000;

-- Analyze to apply new target
ANALYZE orders;

-- View column-specific settings
SELECT 
    schemaname,
    tablename,
    attname,
    attstattarget
FROM pg_stats
WHERE attstattarget > 0
ORDER BY attstattarget DESC;
```

**When to increase statistics target:**
- High cardinality columns (many distinct values)
- Columns with non-uniform distribution
- Columns used in WHERE clauses with complex conditions

**Trade-off:**
- ✅ More accurate estimates
- ❌ Slower ANALYZE
- ❌ More storage in pg_statistic

#### **B. Sample Size**

```sql
-- Autovacuum analyze settings
SHOW autovacuum_analyze_threshold;       -- Default: 50
SHOW autovacuum_analyze_scale_factor;    -- Default: 0.1 (10%)

-- Per-table: Analyze more frequently
ALTER TABLE orders SET (
    autovacuum_analyze_scale_factor = 0.02  -- 2% instead of 10%
);
```

---

### **6. Identifying Stale Statistics**

#### **A. Check Last Analyze Time**

```sql
-- Tables that haven't been analyzed recently
SELECT 
    schemaname,
    tablename,
    n_live_tup,
    n_mod_since_analyze,
    ROUND(100.0 * n_mod_since_analyze / NULLIF(n_live_tup, 0), 2) AS pct_changed,
    last_analyze,
    last_autoanalyze,
    NOW() - GREATEST(last_analyze, last_autoanalyze) AS time_since_analyze
FROM pg_stat_user_tables
WHERE n_mod_since_analyze > 0
ORDER BY pct_changed DESC
LIMIT 20;
```

**Output:**
```
tablename | n_live_tup | n_mod_since_analyze | pct_changed | time_since_analyze
----------+------------+---------------------+-------------+-------------------
orders    | 100,000    | 45,000              | 45.00       | 2 days 05:34:12  ← ANALYZE needed!
customers | 20,000     | 1,500               | 7.50        | 6 hours 12:23:45
products  | 5,000      | 50                  | 1.00        | 1 hour 23:12:34
```

**Red flags:**
- **pct_changed > 20%**: Statistics likely stale
- **time_since_analyze > 1 week**: Should analyze more frequently
- **last_autoanalyze = NULL**: Autovacuum not running or thresholds not met

#### **B. Estimate vs Actual Rows**

```sql
-- Compare estimated vs actual rows
EXPLAIN (ANALYZE, VERBOSE)
SELECT * FROM orders WHERE status = 'rare_status';
```

**Bad statistics:**
```
Seq Scan on orders  (cost=0.00..1892.50 rows=50000 width=180)
                    (actual time=0.015..25.234 rows=3 loops=1)
```
- **Estimated**: 50,000 rows
- **Actual**: 3 rows
- **Discrepancy**: 16,667x overestimate!
- **Solution**: Run ANALYZE

**After ANALYZE:**
```
Index Scan using idx_orders_status on orders
    (cost=0.42..12.45 rows=5 width=180)
    (actual time=0.023..0.034 rows=3 loops=1)
```
- **Estimated**: 5 rows
- **Actual**: 3 rows
- **Discrepancy**: 1.67x (acceptable)

---

### **7. Real-World Impact Examples**

#### **Example 1: Missing ANALYZE After Bulk Load**

**Scenario:**
```sql
-- Load 1 million new orders
COPY orders FROM '/data/orders.csv';

-- Query immediately after
EXPLAIN ANALYZE
SELECT * FROM orders WHERE order_date = CURRENT_DATE;
```

**Without ANALYZE:**
```
Seq Scan on orders  (cost=0.00..5000.00 rows=100 width=180)
                    (actual time=456.789..1234.567 rows=50000 loops=1)
  Filter: (order_date = CURRENT_DATE)
  Rows Removed by Filter: 950000
Execution Time: 1234.678 ms
```
- Planner has no statistics for new data
- Chose sequential scan (wrong!)
- Scanned 1M rows, returned 50K

**After ANALYZE:**
```sql
ANALYZE orders;

EXPLAIN ANALYZE
SELECT * FROM orders WHERE order_date = CURRENT_DATE;
```
```
Index Scan using idx_orders_date on orders
    (cost=0.42..1234.56 rows=50000 width=180)
    (actual time=0.234..45.678 rows=50123 loops=1)
  Index Cond: (order_date = CURRENT_DATE)
Execution Time: 45.789 ms
```
**Result: 1235ms → 46ms = 27x faster!**

#### **Example 2: Data Distribution Change**

**Initial state (uniform distribution):**
```sql
-- All statuses equally common
-- pending: 20%, shipped: 20%, delivered: 20%, cancelled: 20%, refunded: 20%

ANALYZE orders;

SELECT * FROM orders WHERE status = 'pending';
-- Planner estimates 20,000 rows (20% of 100,000)
-- Uses index scan
-- Time: 45ms
```

**6 months later (distribution changed):**
```sql
-- Business changed: Most orders now 'pending' (80%)
-- pending: 80%, shipped: 10%, delivered: 5%, cancelled: 3%, refunded: 2%

-- BUT statistics not updated (still shows 20%)

SELECT * FROM orders WHERE status = 'pending';
-- Planner still estimates 20,000 rows (wrong! actually 80,000)
-- Still uses index scan (wrong choice for 80%!)
-- Time: 2500ms (degraded!)
```

**After ANALYZE:**
```sql
ANALYZE orders;

SELECT * FROM orders WHERE status = 'pending';
```
```
Seq Scan on orders  (cost=0.00..2200.00 rows=80000 width=180)
                    (actual time=0.015..234.567 rows=80123 loops=1)
  Filter: (status = 'pending'::text)
Execution Time: 234.678 ms
```
**Result: 2500ms → 235ms = 10.6x faster!**

#### **Example 3: Join Order Optimization**

**Query:**
```sql
SELECT *
FROM small_table s  -- 100 rows
JOIN medium_table m ON s.id = m.small_id  -- 10,000 rows
JOIN large_table l ON m.id = l.medium_id  -- 1,000,000 rows
WHERE s.active = true;  -- Filters to 10 rows
```

**With accurate statistics:**
```
-- Planner knows: s (10 rows) → m (100 rows) → l (1,000 rows)
Nested Loop (cost=... rows=1000)
  -> Nested Loop (cost=... rows=100)
       -> Index Scan on small_table s (rows=10)
       -> Index Scan on medium_table m (rows=10 per loop)
  -> Index Scan on large_table l (rows=10 per loop)
Execution Time: 15 ms
```

**With stale statistics (overestimates s to 50,000 rows):**
```
-- Planner thinks: s (50,000 rows) → wrong join order!
Hash Join (cost=... rows=500000)
  -> Seq Scan on large_table l (rows=1000000)
  -> Hash
       -> Hash Join (cost=... rows=50000)
            -> Seq Scan on medium_table m (rows=10000)
            -> Hash
                 -> Seq Scan on small_table s (rows=50000)
Execution Time: 15000 ms
```
**Result: 15ms → 15,000ms = 1000x slower!**

**After ANALYZE:**
```sql
ANALYZE small_table;
ANALYZE medium_table;
ANALYZE large_table;

-- Planner now chooses optimal nested loop
-- Execution Time: 15 ms (restored)
```

---

### **8. Best Practices**

1. ✅ **ANALYZE after bulk operations** - INSERT, UPDATE, DELETE, COPY
2. ✅ **Monitor n_mod_since_analyze** - Alert when >20% of table changed
3. ✅ **Increase statistics target for key columns** - High cardinality or complex queries
4. ✅ **Keep autovacuum enabled** - Automatic ANALYZE prevents staleness
5. ✅ **Regular manual ANALYZE** - Don't rely solely on autovacuum for critical tables
6. ✅ **Compare estimated vs actual rows** - Use EXPLAIN ANALYZE to validate
7. ✅ **Tune autovacuum thresholds** - For high-change tables
8. ✅ **ANALYZE before major queries** - Reporting, migrations, backups
9. ⚠️ **Don't over-analyze** - Only when needed (has cost)
10. ⚠️ **Test with production data** - Statistics-based plans differ with data volume

---

### **9. Troubleshooting Statistics Issues**

#### **Problem: Planner always chooses wrong plan**

```sql
-- 1. Check if statistics exist
SELECT 
    attname,
    n_distinct,
    most_common_vals
FROM pg_stats
WHERE tablename = 'orders'
  AND attname = 'status';

-- If NULL or outdated:
ANALYZE orders;

-- 2. Check if statistics are detailed enough
ALTER TABLE orders ALTER COLUMN status SET STATISTICS 1000;
ANALYZE orders;

-- 3. Check estimated vs actual
EXPLAIN (ANALYZE, VERBOSE) SELECT ...;
```

#### **Problem: Query plan changes unexpectedly**

```sql
-- Check when last analyzed
SELECT 
    tablename,
    last_analyze,
    last_autoanalyze,
    n_mod_since_analyze
FROM pg_stat_user_tables
WHERE tablename IN ('orders', 'customers', 'products');

-- If long time since analyze:
ANALYZE orders;
ANALYZE customers;
ANALYZE products;
```

---

### **Interview Tips:**
- **Definition**: "Table statistics are metadata about data distribution stored in pg_statistic catalog, used by query planner to estimate costs and choose optimal execution plans"
- **Key statistics**: "n_distinct (cardinality), most_common_vals/freqs (distribution), histogram_bounds (ranges), correlation (physical ordering), null_frac"
- **Impact**: "Without accurate statistics, planner can't estimate row counts correctly, leading to wrong join orders, scan methods, and memory allocation"
- **Real example**: "After bulk load of 1M orders, query took 1.2 seconds. Ran ANALYZE, planner switched from seq scan to index scan, time dropped to 45ms - 27x improvement"
- **Stale statistics**: "Query degraded from 15ms to 15 seconds over 6 months. Found n_mod_since_analyze was 50,000 (50% of table changed). After ANALYZE, back to 15ms - 1000x improvement"
- **Best practice**: "Always ANALYZE after bulk operations. Monitor n_mod_since_analyze and alert when >20% of table changed without ANALYZE"
- **Configuration**: "Increased statistics_target from 100 to 1000 for high-cardinality customer_id column - improved join selectivity estimates"

</details>

<details>
<summary><strong>50. What is connection pooling and why is it important?</strong></summary>

### **Answer:**

Connection pooling is a technique that maintains a pool of reusable database connections that can be shared across multiple application requests, rather than opening and closing connections for each database operation. It's crucial for performance, scalability, and efficient resource utilization in PostgreSQL applications.

---

### **1. The Connection Problem**

#### **A. Without Connection Pooling**

**Typical web application flow:**
```python
# Each request creates new connection
def handle_request():
    conn = psycopg2.connect("host=db user=app password=secret dbname=mydb")
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
    result = cursor.fetchall()
    cursor.close()
    conn.close()  # Connection destroyed
    return result
```

**Problems:**
1. **Connection overhead** - Each connection takes 50-100ms to establish
2. **Resource waste** - PostgreSQL backend process created/destroyed per request
3. **Connection limit** - PostgreSQL has max_connections limit (default: 100)
4. **Memory overhead** - Each connection uses 5-10 MB of memory
5. **Scaling issues** - Can't handle 1000+ concurrent users

**Example cost:**
```
Connection establishment: 75ms
Query execution: 5ms
Connection teardown: 25ms
Total: 105ms (95% overhead!)
```

#### **B. PostgreSQL Connection Lifecycle**

```sql
-- Check current connections
SELECT COUNT(*) FROM pg_stat_activity;
-- Result: 87 connections (close to 100 limit!)

-- Each connection consumes memory
SELECT 
    COUNT(*) as connections,
    pg_size_pretty(COUNT(*) * 10 * 1024 * 1024) as estimated_memory
FROM pg_stat_activity;
-- Result: 87 connections, ~870 MB memory
```

**Connection states:**
- **Active**: Executing query
- **Idle**: Waiting for next query
- **Idle in transaction**: Transaction open but not executing
- **Idle in transaction (aborted)**: Transaction failed but not rolled back

---

### **2. How Connection Pooling Works**

#### **A. Basic Concept**

```
Without Pooling:
App Request 1 → [New Connection] → PostgreSQL → [Close Connection]
App Request 2 → [New Connection] → PostgreSQL → [Close Connection]
App Request 3 → [New Connection] → PostgreSQL → [Close Connection]

With Pooling:
App Request 1 → [Pool] ← [Reuse Conn 1] → PostgreSQL
App Request 2 → [Pool] ← [Reuse Conn 2] → PostgreSQL
App Request 3 → [Pool] ← [Reuse Conn 1] → PostgreSQL (reused!)
                  ↓
        [10 persistent connections]
```

**Connection lifecycle with pooling:**
1. Application requests connection from pool
2. Pool provides existing connection (or creates new one if available)
3. Application uses connection for query
4. Application returns connection to pool (doesn't close it)
5. Connection stays open, ready for next request

#### **B. Pool Configuration**

**Key parameters:**
- **min_pool_size**: Minimum connections kept open (e.g., 5)
- **max_pool_size**: Maximum connections allowed (e.g., 20)
- **connection_timeout**: Max wait time for connection (e.g., 30s)
- **idle_timeout**: Close idle connections after (e.g., 600s)
- **max_lifetime**: Max connection age before refresh (e.g., 3600s)

---

### **3. Connection Pooling Solutions**

#### **A. PgBouncer (Most Popular)**

**Installation and configuration:**
```ini
# /etc/pgbouncer/pgbouncer.ini

[databases]
mydb = host=127.0.0.1 port=5432 dbname=mydb

[pgbouncer]
listen_addr = 0.0.0.0
listen_port = 6432
auth_type = md5
auth_file = /etc/pgbouncer/userlist.txt
admin_users = pgbouncer_admin
pool_mode = transaction

# Connection limits
max_client_conn = 1000      # Max connections from apps
default_pool_size = 20      # Connections per database
reserve_pool_size = 5       # Emergency reserve
reserve_pool_timeout = 3    # Wait time for reserve

# Timeouts
server_idle_timeout = 600   # Close idle server conn after 10 min
server_lifetime = 3600      # Recycle connection after 1 hour
server_connect_timeout = 15 # Connection timeout
query_timeout = 0           # No query timeout (0 = disabled)
```

**Pool modes:**

**1. Session pooling:**
```ini
pool_mode = session
```
- Connection assigned to client for entire session
- Client can use all PostgreSQL features (temp tables, prepared statements)
- Less efficient (1:1 mapping at peak)

**2. Transaction pooling (recommended):**
```ini
pool_mode = transaction
```
- Connection returned after each transaction
- Most efficient for web applications
- ⚠️ Cannot use: temp tables, prepared statements across transactions, LISTEN/NOTIFY

**3. Statement pooling:**
```ini
pool_mode = statement
```
- Connection returned after each statement
- Most aggressive (highest efficiency)
- ⚠️ Cannot use: transactions, temp tables, prepared statements

**Usage:**
```python
# Application connects to PgBouncer instead of PostgreSQL
conn = psycopg2.connect(
    host="localhost",
    port=6432,  # PgBouncer port (not 5432)
    user="app_user",
    password="secret",
    dbname="mydb"
)
```

**Monitoring PgBouncer:**
```bash
# Connect to PgBouncer admin console
psql -h localhost -p 6432 -U pgbouncer_admin pgbouncer

# Show pool status
SHOW POOLS;
```

**Output:**
```
database | user     | cl_active | cl_waiting | sv_active | sv_idle | sv_used | sv_tested | sv_login | maxwait
---------+----------+-----------+------------+-----------+---------+---------+-----------+----------+--------
mydb     | app_user | 45        | 2          | 18        | 2       | 20      | 0         | 0        | 0
```

**Interpretation:**
- **cl_active = 45**: 45 client connections active
- **cl_waiting = 2**: 2 clients waiting for connection
- **sv_active = 18**: 18 PostgreSQL connections in use
- **sv_idle = 2**: 2 PostgreSQL connections idle
- **Efficiency**: 45 clients using only 20 server connections = 2.25:1 ratio

#### **B. Application-Level Pooling**

**Python (psycopg2 + connection pool):**
```python
from psycopg2 import pool

# Create connection pool
connection_pool = pool.SimpleConnectionPool(
    minconn=5,      # Minimum connections
    maxconn=20,     # Maximum connections
    host="localhost",
    database="mydb",
    user="app_user",
    password="secret"
)

def handle_request(user_id):
    # Get connection from pool
    conn = connection_pool.getconn()
    try:
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
        result = cursor.fetchall()
        cursor.close()
        return result
    finally:
        # Return connection to pool (don't close!)
        connection_pool.putconn(conn)
```

**Node.js (node-postgres with pg-pool):**
```javascript
const { Pool } = require('pg');

const pool = new Pool({
    host: 'localhost',
    database: 'mydb',
    user: 'app_user',
    password: 'secret',
    max: 20,            // Maximum connections
    min: 5,             // Minimum connections
    idleTimeoutMillis: 30000,  // Close idle connections after 30s
    connectionTimeoutMillis: 2000  // Wait 2s for connection
});

async function handleRequest(userId) {
    const client = await pool.connect();
    try {
        const result = await client.query('SELECT * FROM users WHERE id = $1', [userId]);
        return result.rows;
    } finally {
        client.release();  // Return to pool
    }
}
```

**Java (HikariCP - fastest):**
```java
HikariConfig config = new HikariConfig();
config.setJdbcUrl("jdbc:postgresql://localhost:5432/mydb");
config.setUsername("app_user");
config.setPassword("secret");
config.setMaximumPoolSize(20);
config.setMinimumIdle(5);
config.setConnectionTimeout(30000);
config.setIdleTimeout(600000);
config.setMaxLifetime(1800000);

HikariDataSource ds = new HikariDataSource(config);

// Use connection
try (Connection conn = ds.getConnection()) {
    PreparedStatement stmt = conn.prepareStatement("SELECT * FROM users WHERE id = ?");
    stmt.setInt(1, userId);
    ResultSet rs = stmt.executeQuery();
    // Process results...
}  // Connection automatically returned to pool
```

#### **C. Pgpool-II (Advanced)**

```ini
# Provides connection pooling + load balancing + replication
listen_addresses = '*'
port = 9999
backend_hostname0 = 'localhost'
backend_port0 = 5432
backend_weight0 = 1

num_init_children = 32      # Max concurrent connections
max_pool = 4                # Connections per child process
connection_life_time = 600  # Connection lifetime
child_life_time = 300       # Child process lifetime
```

**Features:**
- Connection pooling
- Load balancing across replicas
- Query routing (read/write split)
- Connection limiting
- Query caching

---

### **4. Performance Impact**

#### **A. Connection Time Comparison**

**Without pooling:**
```python
# Measure connection overhead
import time

def test_without_pooling():
    start = time.time()
    for i in range(100):
        conn = psycopg2.connect(...)
        cursor = conn.cursor()
        cursor.execute("SELECT 1")
        cursor.close()
        conn.close()
    return time.time() - start

# Result: 8.5 seconds (85ms per operation)
```

**With pooling:**
```python
def test_with_pooling():
    start = time.time()
    for i in range(100):
        conn = pool.getconn()
        cursor = conn.cursor()
        cursor.execute("SELECT 1")
        cursor.close()
        pool.putconn(conn)
    return time.time() - start

# Result: 0.5 seconds (5ms per operation)
```

**Improvement: 8.5s → 0.5s = 17x faster!**

#### **B. Scalability Comparison**

**Scenario: 1000 concurrent users**

**Without pooling:**
```
1000 users × 1 connection each = 1000 connections needed
PostgreSQL max_connections = 100
Result: 900 users get "too many connections" error
```

**With pooling (PgBouncer):**
```
1000 users → PgBouncer (transaction pooling) → 20 PostgreSQL connections
Average request time: 50ms
Concurrent requests at any moment: 1000 × 0.05s = 50 requests
Connections needed: 50 requests ÷ 2.5 efficiency = 20 connections
Result: All 1000 users served successfully
```

#### **C. Memory Usage**

**Without pooling (1000 connections):**
```
1000 connections × 10 MB per connection = 10 GB memory
PostgreSQL overhead = huge
Result: Server crashes or slows to crawl
```

**With pooling (20 connections):**
```
20 connections × 10 MB per connection = 200 MB memory
Result: 98% memory savings
```

---

### **5. Real-World Example**

**Scenario: E-commerce site experiencing "too many connections" errors**

**Initial state:**
```sql
-- Check connections
SELECT COUNT(*) FROM pg_stat_activity;
-- Result: 98 connections (max is 100)

SHOW max_connections;
-- Result: 100

-- Application logs:
-- ERROR: FATAL:  sorry, too many clients already
-- ERROR: could not connect to server
```

**Investigation:**
```sql
-- Most connections are idle
SELECT 
    state,
    COUNT(*) as count
FROM pg_stat_activity
GROUP BY state;
```

**Output:**
```
state                    | count
-------------------------+------
active                   | 5
idle                     | 85
idle in transaction      | 8
```

**Analysis:**
- Only 5 active queries
- 85 connections sitting idle (wasteful!)
- 8 idle in transaction (blocking others)
- Application opens connection per request, doesn't close promptly

**Solution: Deploy PgBouncer**

```bash
# Install PgBouncer
apt-get install pgbouncer

# Configure
cat > /etc/pgbouncer/pgbouncer.ini <<EOF
[databases]
mydb = host=localhost port=5432 dbname=mydb

[pgbouncer]
listen_addr = 0.0.0.0
listen_port = 6432
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 20
reserve_pool_size = 5
EOF

# Start PgBouncer
systemctl start pgbouncer
```

**Update application:**
```python
# Change connection from:
conn = psycopg2.connect(host="db.example.com", port=5432, ...)

# To:
conn = psycopg2.connect(host="db.example.com", port=6432, ...)  # PgBouncer port
```

**Results:**
```sql
-- PostgreSQL now has only 20 connections
SELECT COUNT(*) FROM pg_stat_activity;
-- Result: 20 connections

-- But PgBouncer shows:
SHOW POOLS;
-- cl_active = 450 (450 app connections)
-- sv_active = 18 (only 18 PostgreSQL connections)
-- Efficiency: 450 ÷ 18 = 25:1 ratio!

-- Memory usage:
-- Before: 98 × 10 MB = 980 MB
-- After: 20 × 10 MB = 200 MB
-- Savings: 78% memory reduction
```

**Performance improvement:**
- Connection errors: eliminated
- Response time: 250ms → 45ms (5.5x faster)
- Concurrent users: 100 → 1000+ (10x scalability)
- Server load: CPU 80% → 35% (more headroom)

---

### **6. Connection Pool Sizing**

#### **A. Formula**

```
pool_size = Tn × (Cm - 1) + 1

Where:
Tn = Number of application threads/workers
Cm = Max connections per thread (usually 1)

Example:
- 10 application servers
- 20 threads per server
- pool_size = 10 × 20 × (1 - 1) + 1 = 1

Wait, that's wrong. Better formula:

pool_size = connections_per_instance × number_of_instances × safety_factor

Or simpler:
pool_size ≈ 2 × CPU_cores (for CPU-bound workloads)
pool_size ≈ 10-20 (for typical web applications)
```

#### **B. Recommended Sizes**

**Small application (single server):**
```
max_pool_size = 10-20
min_pool_size = 5
```

**Medium application (3-5 servers):**
```
PgBouncer pool per database = 20-50
Application pool per server = 5-10
```

**Large application (10+ servers):**
```
PgBouncer pool per database = 50-100
Application pool per server = 5
Read replicas: Scale out
```

#### **C. Monitoring Pool Health**

```sql
-- PgBouncer stats
SHOW STATS;

-- Look for:
-- - avg_query_time: Should be low (<10ms typical)
-- - avg_wait_time: Should be near 0 (connections available)

-- Application-level monitoring
SELECT 
    application_name,
    COUNT(*) as connections,
    COUNT(*) FILTER (WHERE state = 'active') as active,
    COUNT(*) FILTER (WHERE state = 'idle') as idle,
    MAX(NOW() - query_start) as longest_query
FROM pg_stat_activity
WHERE application_name != ''
GROUP BY application_name;
```

---

### **7. Common Issues**

#### **A. Connection Leaks**

**Problem:**
```python
def bad_code(user_id):
    conn = pool.getconn()
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
    if cursor.rowcount == 0:
        return None  # Connection never returned! LEAK!
    result = cursor.fetchall()
    pool.putconn(conn)
    return result
```

**Solution:**
```python
def good_code(user_id):
    conn = pool.getconn()
    try:
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
        if cursor.rowcount == 0:
            return None
        return cursor.fetchall()
    finally:
        pool.putconn(conn)  # Always returned
```

#### **B. Idle in Transaction**

**Problem:**
```python
conn = pool.getconn()
conn.execute("BEGIN")
conn.execute("UPDATE accounts SET balance = balance - 100 WHERE id = 1")
# Application crashes or network timeout
# Connection stuck in "idle in transaction" state
# Holds locks, blocks other queries
```

**Solution:**
```python
# Set statement timeout
conn.execute("SET statement_timeout = 5000")  # 5 seconds

# Or configure in PgBouncer
# query_timeout = 30  # 30 seconds
```

#### **C. Pool Exhaustion**

**Problem:**
```
Error: TimeoutError: could not obtain connection from pool within 30 seconds
```

**Diagnosis:**
```sql
-- Check if pool is undersized
SHOW POOLS;  -- All connections in use?

-- Check for long-running queries
SELECT pid, NOW() - query_start AS duration, state, query
FROM pg_stat_activity
WHERE state = 'active'
ORDER BY duration DESC;
```

**Solutions:**
1. Increase pool size (if PostgreSQL can handle it)
2. Optimize slow queries
3. Add read replicas and route read queries there
4. Implement request queuing in application

---

### **8. Best Practices**

1. ✅ **Use PgBouncer** - Easiest and most effective solution
2. ✅ **Transaction mode** - Best for web applications
3. ✅ **Right-size pools** - Start with 10-20 connections, monitor and adjust
4. ✅ **Always release connections** - Use try/finally or context managers
5. ✅ **Set timeouts** - Prevent connection leaks and stuck transactions
6. ✅ **Monitor pool stats** - Track utilization, wait times, errors
7. ✅ **Connection validation** - Test connections before use
8. ✅ **Connection lifecycle** - Recycle connections periodically
9. ⚠️ **Don't over-pool** - More connections ≠ better performance
10. ⚠️ **Test failover** - Ensure pool handles database restarts

---

### **Interview Tips:**
- **Definition**: "Connection pooling maintains a pool of reusable database connections shared across application requests, eliminating the overhead of creating/destroying connections per request"
- **Why important**: "Without pooling, each connection takes 50-100ms to establish. With 1000 concurrent users and 100 max_connections, 900 users get errors"
- **Solutions**: "PgBouncer (transaction mode) is most common - 1000 client connections → 20 PostgreSQL connections = 50:1 efficiency"
- **Real example**: "E-commerce site hitting 98/100 connection limit. Deployed PgBouncer with pool_size=20. Eliminated connection errors, reduced response time from 250ms to 45ms, supported 1000+ concurrent users"
- **Pool sizing**: "Start with 10-20 connections for typical web app. Monitor utilization and adjust. Too many connections waste memory and hurt performance"
- **Common issue**: "Connection leaks - always use try/finally to return connections to pool"
- **Performance**: "Measured 17x faster with pooling: 85ms per operation → 5ms"

</details>

<details>
<summary><strong>51. How do you handle deadlocks in PostgreSQL?</strong></summary>

### **Answer:**

A deadlock occurs when two or more transactions are waiting for each other to release locks, creating a circular dependency that prevents any transaction from proceeding. PostgreSQL automatically detects and resolves deadlocks, but understanding how to prevent and handle them is crucial for robust applications.

---

### **1. What is a Deadlock?**

#### **A. Classic Deadlock Scenario**

**Transaction 1:**
```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;  -- Locks row 1
-- Now needs lock on row 2
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;  -- WAITS
COMMIT;
```

**Transaction 2 (concurrent):**
```sql
BEGIN;
UPDATE accounts SET balance = balance - 50 WHERE account_id = 2;   -- Locks row 2
-- Now needs lock on row 1
UPDATE accounts SET balance = balance + 50 WHERE account_id = 1;   -- WAITS
COMMIT;
```

**Result: Deadlock!**
- Transaction 1 holds lock on row 1, waits for row 2
- Transaction 2 holds lock on row 2, waits for row 1
- Both wait forever (circular dependency)

**PostgreSQL's response:**
```
ERROR: deadlock detected
DETAIL: Process 12345 waits for ShareLock on transaction 67890; blocked by process 12346.
Process 12346 waits for ShareLock on transaction 67889; blocked by process 12345.
HINT: See server log for query details.
```

- PostgreSQL detects deadlock (default: after 1 second)
- Aborts one transaction (victim)
- Other transaction proceeds
- Application receives error and should retry

---

### **2. Deadlock Detection**

#### **A. deadlock_timeout Setting**

```sql
-- How long to wait before checking for deadlock
SHOW deadlock_timeout;
-- Default: 1s (1000ms)

-- Increase if false positives occur
ALTER SYSTEM SET deadlock_timeout = 5000;  -- 5 seconds
SELECT pg_reload_conf();
```

**Trade-off:**
- **Lower value (100ms)**: Faster deadlock detection, but more CPU overhead checking
- **Higher value (5s)**: Less CPU overhead, but transactions wait longer before deadlock is detected

#### **B. Monitoring Deadlocks**

```sql
-- Check deadlock count
SELECT 
    datname,
    deadlocks,
    deadlocks::float / NULLIF(xact_commit + xact_rollback, 0) * 100 AS deadlock_pct
FROM pg_stat_database
WHERE datname = current_database();
```

**Output:**
```
datname | deadlocks | deadlock_pct
--------+-----------+-------------
mydb    | 245       | 0.12
```

**Interpretation:**
- **deadlocks = 245**: 245 deadlocks detected since stats reset
- **0.12%**: 0.12% of transactions result in deadlock (acceptable)
- **>1%**: Investigate and redesign queries/locking strategy

**View deadlock details in logs:**
```sql
-- Enable deadlock logging
ALTER SYSTEM SET log_lock_waits = on;
SELECT pg_reload_conf();

-- Logs will show:
-- LOG:  process 12345 still waiting for ShareLock on transaction 67890 after 1000.123 ms
-- DETAIL:  Process holding the lock: 12346. Wait queue: 12345.
-- STATEMENT:  UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;
```

---

### **3. Preventing Deadlocks**

#### **A. Consistent Lock Ordering**

**Problem: Inconsistent order causes deadlocks**
```sql
-- Transaction 1
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

-- Transaction 2
UPDATE accounts SET balance = balance - 50 WHERE account_id = 2;  -- Different order!
UPDATE accounts SET balance = balance + 50 WHERE account_id = 1;
```

**Solution: Always lock in same order**
```sql
-- Transaction 1
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;  -- Lower ID first
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

-- Transaction 2
UPDATE accounts SET balance = balance + 50 WHERE account_id = 1;   -- Lower ID first
UPDATE accounts SET balance = balance - 50 WHERE account_id = 2;
```

**Application-level enforcement:**
```python
def transfer(from_id, to_id, amount):
    # Sort IDs to ensure consistent lock order
    first_id = min(from_id, to_id)
    second_id = max(from_id, to_id)
    
    # Lock accounts in consistent order
    cursor.execute(
        "UPDATE accounts SET balance = balance - %s WHERE account_id = %s",
        (amount if from_id == first_id else -amount, first_id)
    )
    cursor.execute(
        "UPDATE accounts SET balance = balance + %s WHERE account_id = %s",
        (-amount if from_id == first_id else amount, second_id)
    )
    conn.commit()
```

#### **B. Use SELECT FOR UPDATE to Lock Early**

**Problem: Update later, lock too late**
```sql
BEGIN;
-- Read values
SELECT balance FROM accounts WHERE account_id = 1;  -- No lock yet
-- Some processing...
-- Now update (gets lock)
UPDATE accounts SET balance = 500 WHERE account_id = 1;  -- Lock acquired late
COMMIT;
```

**Solution: Lock explicitly when reading**
```sql
BEGIN;
-- Lock rows explicitly when reading
SELECT balance FROM accounts WHERE account_id = 1 FOR UPDATE;  -- Lock acquired
-- Some processing...
UPDATE accounts SET balance = 500 WHERE account_id = 1;  -- Already have lock
COMMIT;
```

**Lock in consistent order:**
```sql
BEGIN;
-- Lock both rows in ID order
SELECT balance FROM accounts 
WHERE account_id IN (1, 2)
ORDER BY account_id  -- Consistent order!
FOR UPDATE;

-- Now update (already have locks)
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;
COMMIT;
```

#### **C. Keep Transactions Short**

**Problem: Long transactions increase deadlock risk**
```sql
BEGIN;
UPDATE orders SET status = 'processing' WHERE order_id = 123;
-- Call external API (5 seconds)
-- Send email (3 seconds)
-- Generate PDF (10 seconds)
UPDATE orders SET status = 'completed' WHERE order_id = 123;
COMMIT;  -- Held lock for 18+ seconds!
```

**Solution: Minimize transaction duration**
```sql
BEGIN;
UPDATE orders SET status = 'processing' WHERE order_id = 123;
COMMIT;  -- Release lock immediately

-- Do slow operations outside transaction
call_external_api()  -- 5 seconds (no locks held)
send_email()         -- 3 seconds
generate_pdf()       -- 10 seconds

BEGIN;
UPDATE orders SET status = 'completed' WHERE order_id = 123;
COMMIT;  -- Quick transaction
```

#### **D. Use Lower Isolation Levels**

```sql
-- SERIALIZABLE isolation = higher deadlock risk
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
-- Checks for conflicts between all concurrent transactions

-- READ COMMITTED (default) = lower deadlock risk
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
-- Only checks for conflicts on rows being modified
```

**Example:**
```sql
-- Safer: READ COMMITTED
BEGIN;
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;
COMMIT;
```

#### **E. Batch Operations Carefully**

**Problem: Multi-row updates in different orders**
```sql
-- Transaction 1
UPDATE inventory SET quantity = quantity - 1 
WHERE product_id IN (10, 20, 30);  -- Locks in unknown order

-- Transaction 2
UPDATE inventory SET quantity = quantity - 1 
WHERE product_id IN (30, 20, 10);  -- Different order!
```

**Solution: Explicit ordering**
```sql
-- Transaction 1
UPDATE inventory SET quantity = quantity - 1 
WHERE product_id IN (10, 20, 30)
ORDER BY product_id;  -- Consistent order

-- Transaction 2
UPDATE inventory SET quantity = quantity - 1 
WHERE product_id IN (10, 20, 30)
ORDER BY product_id;  -- Same order
```

---

### **4. Handling Deadlocks in Application**

#### **A. Retry Logic**

```python
import psycopg2
from psycopg2 import OperationalError
import time

def transfer_with_retry(from_id, to_id, amount, max_retries=3):
    for attempt in range(max_retries):
        try:
            conn = get_connection()
            cursor = conn.cursor()
            
            # Perform transaction
            cursor.execute(
                "UPDATE accounts SET balance = balance - %s WHERE account_id = %s",
                (amount, from_id)
            )
            cursor.execute(
                "UPDATE accounts SET balance = balance + %s WHERE account_id = %s",
                (amount, to_id)
            )
            conn.commit()
            return True  # Success
            
        except OperationalError as e:
            if "deadlock detected" in str(e):
                conn.rollback()
                if attempt < max_retries - 1:
                    # Exponential backoff
                    wait_time = (2 ** attempt) * 0.1  # 0.1s, 0.2s, 0.4s
                    time.sleep(wait_time)
                    continue  # Retry
                else:
                    raise  # Max retries exceeded
            else:
                raise  # Different error
                
    return False
```

**Java example:**
```java
public void transferWithRetry(int fromId, int toId, double amount) throws SQLException {
    int maxRetries = 3;
    for (int attempt = 0; attempt < maxRetries; attempt++) {
        try {
            Connection conn = dataSource.getConnection();
            conn.setAutoCommit(false);
            
            // Perform transaction
            PreparedStatement stmt1 = conn.prepareStatement(
                "UPDATE accounts SET balance = balance - ? WHERE account_id = ?"
            );
            stmt1.setDouble(1, amount);
            stmt1.setInt(2, fromId);
            stmt1.executeUpdate();
            
            PreparedStatement stmt2 = conn.prepareStatement(
                "UPDATE accounts SET balance = balance + ? WHERE account_id = ?"
            );
            stmt2.setDouble(1, amount);
            stmt2.setInt(2, toId);
            stmt2.executeUpdate();
            
            conn.commit();
            return;  // Success
            
        } catch (SQLException e) {
            if (e.getSQLState().equals("40P01")) {  // Deadlock detected
                if (attempt < maxRetries - 1) {
                    Thread.sleep((long) (Math.pow(2, attempt) * 100));  // Backoff
                    continue;
                }
            }
            throw e;
        }
    }
}
```

#### **B. Graceful Degradation**

```python
def process_order(order_id):
    try:
        # Try to process immediately
        with transaction():
            update_inventory(order_id)
            update_order_status(order_id, 'completed')
    except DeadlockError:
        # Fallback: Queue for later processing
        enqueue_for_retry(order_id)
        log.warning(f"Order {order_id} deadlocked, queued for retry")
        return {'status': 'queued', 'message': 'Order processing, please check back'}
```

---

### **5. Advanced Deadlock Prevention**

#### **A. Advisory Locks**

```sql
-- Use advisory locks to control access order
BEGIN;

-- Lock resources in consistent order (by ID)
SELECT pg_advisory_xact_lock(1);  -- Lock resource 1
SELECT pg_advisory_xact_lock(2);  -- Lock resource 2

-- Now safely update
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

COMMIT;  -- Advisory locks released automatically
```

**Benefits:**
- Explicit lock ordering
- No deadlocks if used consistently
- Locks released on transaction end

#### **B. Optimistic Locking**

```sql
-- Add version column
ALTER TABLE accounts ADD COLUMN version INTEGER DEFAULT 0;

-- Update with version check
BEGIN;
SELECT balance, version FROM accounts WHERE account_id = 1;
-- balance = 500, version = 5

-- Application processes...

UPDATE accounts 
SET balance = 400, version = 6
WHERE account_id = 1 AND version = 5;

IF NOT FOUND THEN
    ROLLBACK;
    RAISE EXCEPTION 'Account was modified by another transaction';
END IF;

COMMIT;
```

**Benefits:**
- No locks held during processing
- Prevents lost updates
- Better concurrency

#### **C. Queue-Based Processing**

```sql
-- Use queue instead of direct updates
CREATE TABLE transfer_queue (
    id SERIAL PRIMARY KEY,
    from_account INT,
    to_account INT,
    amount DECIMAL,
    status VARCHAR DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT NOW()
);

-- Application inserts transfer request (no locks)
INSERT INTO transfer_queue (from_account, to_account, amount)
VALUES (1, 2, 100);

-- Background worker processes queue (single thread, no deadlocks)
BEGIN;
SELECT * FROM transfer_queue 
WHERE status = 'pending' 
ORDER BY id 
LIMIT 1 
FOR UPDATE SKIP LOCKED;  -- Skip if locked by another worker

-- Process transfer
UPDATE accounts SET balance = balance - 100 WHERE account_id = from_account;
UPDATE accounts SET balance = balance + 100 WHERE account_id = to_account;
UPDATE transfer_queue SET status = 'completed' WHERE id = queue_id;
COMMIT;
```

---

### **6. Real-World Example**

**Scenario: E-commerce inventory deadlocks during flash sale**

**Problem:**
```
2024-12-24 10:00:00: 50 deadlocks in 1 minute
2024-12-24 10:01:00: 123 deadlocks in 1 minute
2024-12-24 10:02:00: 287 deadlocks in 1 minute

Database logs:
ERROR: deadlock detected
DETAIL: Process 12345 waits for ShareLock on transaction 67890; blocked by process 12346.
Process 12346 waits for ShareLock on transaction 67889; blocked by process 12345.
```

**Investigation:**
```sql
-- Check deadlock queries
SELECT query FROM pg_stat_activity WHERE wait_event_type = 'Lock';
```

**Found:**
```sql
-- Different orders buying same products
-- Customer A: Buys [product 10, product 20]
-- Customer B: Buys [product 20, product 10]  -- Different order!

UPDATE inventory SET quantity = quantity - 1 WHERE product_id IN (10, 20);
```

**Solution 1: Order by product_id**
```python
def purchase_items(product_ids, quantities):
    # Sort to ensure consistent lock order
    items = sorted(zip(product_ids, quantities))
    
    for product_id, quantity in items:
        cursor.execute(
            "UPDATE inventory SET quantity = quantity - %s WHERE product_id = %s",
            (quantity, product_id)
        )
```

**Solution 2: SELECT FOR UPDATE first**
```python
def purchase_items(product_ids, quantities):
    # Lock all rows in consistent order first
    cursor.execute(
        "SELECT product_id, quantity FROM inventory "
        "WHERE product_id = ANY(%s) "
        "ORDER BY product_id "
        "FOR UPDATE",
        (product_ids,)
    )
    
    # Now update (already have locks)
    for product_id, quantity in zip(product_ids, quantities):
        cursor.execute(
            "UPDATE inventory SET quantity = quantity - %s WHERE product_id = %s",
            (quantity, product_id)
        )
```

**Results:**
```
Before fix: 460 deadlocks in 3 minutes, 15% transaction failure rate
After fix: 0 deadlocks in 24 hours, 0% transaction failure rate
Response time: 250ms → 50ms (5x improvement, no retries)
```

---

### **7. Best Practices**

1. ✅ **Consistent lock order** - Always acquire locks in same order (e.g., by ID)
2. ✅ **Lock early** - Use SELECT FOR UPDATE when reading data you'll modify
3. ✅ **Keep transactions short** - Minimize time holding locks
4. ✅ **Implement retry logic** - Retry with exponential backoff
5. ✅ **Use appropriate isolation level** - READ COMMITTED for most cases
6. ✅ **Monitor deadlocks** - Track deadlock count and percentage
7. ✅ **Log deadlock details** - Enable log_lock_waits for investigation
8. ✅ **Test under load** - Simulate concurrent transactions
9. ⚠️ **Avoid user input during transactions** - Don't wait for user while holding locks
10. ⚠️ **Consider advisory locks** - For complex locking scenarios

---

### **Interview Tips:**
- **Definition**: "Deadlock occurs when two transactions wait for each other to release locks, creating circular dependency. PostgreSQL detects after 1 second (deadlock_timeout) and aborts one transaction"
- **Common cause**: "Inconsistent lock order - Transaction A locks row 1 then 2, Transaction B locks row 2 then 1"
- **Prevention**: "Always acquire locks in consistent order (e.g., sort by ID), use SELECT FOR UPDATE early, keep transactions short"
- **Handling**: "Implement retry logic with exponential backoff. Catch deadlock error, wait, retry up to 3 times"
- **Real example**: "E-commerce flash sale had 460 deadlocks in 3 minutes. Multiple customers buying same products in different orders. Fixed by sorting product IDs before locking. Result: 0 deadlocks, 5x faster response time"
- **Monitoring**: "Query pg_stat_database for deadlock count. If >1% of transactions, investigate and redesign"
- **Advanced**: "Used advisory locks for complex scenarios, or queue-based processing with single worker to eliminate deadlocks entirely"

</details>

51. How do you handle deadlocks in PostgreSQL?

<details>
<summary><strong>Answer</strong></summary>

### What is a Deadlock?

A **deadlock** occurs when two or more transactions are waiting for each other to release locks, creating a circular dependency that prevents any of them from proceeding.

**Classic Deadlock Scenario:**
```
Time    Transaction 1                Transaction 2
----    -------------                -------------
T1      BEGIN;                       BEGIN;
T2      UPDATE accounts              UPDATE accounts
        SET balance = balance - 100  SET balance = balance + 50
        WHERE id = 1;                WHERE id = 2;
        (locks row 1)                (locks row 2)

T3      UPDATE accounts              UPDATE accounts
        SET balance = balance + 100  SET balance = balance - 50
        WHERE id = 2;                WHERE id = 1;
        (waits for row 2)            (waits for row 1)
        
        💥 DEADLOCK! Each transaction holds a lock the other needs
```

**What Happens:**
- PostgreSQL's deadlock detector runs every `deadlock_timeout` (default 1 second)
- Detects the circular wait condition
- Aborts one transaction with: `ERROR: deadlock detected`
- The other transaction proceeds normally

---

### Deadlock Detection

#### 1. **Configure Deadlock Detection**
```sql
-- Set deadlock detection timeout (default: 1s)
ALTER SYSTEM SET deadlock_timeout = '1s';

-- Enable lock wait logging
ALTER SYSTEM SET log_lock_waits = on;
ALTER SYSTEM SET deadlock_timeout = '1s';

-- Reload configuration
SELECT pg_reload_conf();
```

#### 2. **Monitor Deadlocks**
```sql
-- Check deadlock statistics
SELECT 
    datname,
    deadlocks,
    deadlocks::float / NULLIF(xact_commit + xact_rollback, 0) * 100 AS deadlock_percentage
FROM pg_stat_database
WHERE datname = current_database();

-- Example output:
  datname  | deadlocks | deadlock_percentage 
-----------+-----------+---------------------
 ecommerce |       245 |              0.12%
```

#### 3. **Check PostgreSQL Logs**
```log
2024-12-24 10:30:15 UTC ERROR: deadlock detected
2024-12-24 10:30:15 UTC DETAIL: Process 12345 waits for ShareLock on transaction 67890;
    blocked by process 12346.
    Process 12346 waits for ShareLock on transaction 67889; blocked by process 12345.
2024-12-24 10:30:15 UTC HINT: See server log for query details.
2024-12-24 10:30:15 UTC STATEMENT: UPDATE accounts SET balance = balance - 50 WHERE id = 2
```

---

### Preventing Deadlocks

#### 1. **Consistent Lock Ordering** (Most Important!)

**❌ Wrong - Inconsistent Ordering:**
```python
# Transaction 1: Updates products in order [5, 2, 8]
# Transaction 2: Updates products in order [2, 8, 5]
# Different ordering → potential deadlock!

def update_inventory(product_ids, quantities):
    for pid, qty in zip(product_ids, quantities):
        cursor.execute(
            "UPDATE products SET stock = stock - %s WHERE id = %s",
            (qty, pid)
        )
```

**✅ Right - Consistent Ordering:**
```python
def update_inventory(product_ids, quantities):
    # Always lock in ascending ID order
    sorted_items = sorted(zip(product_ids, quantities))
    
    for pid, qty in sorted_items:
        cursor.execute(
            "UPDATE products SET stock = stock - %s WHERE id = %s",
            (qty, pid)
        )

# Both transactions now process [2, 5, 8] → no deadlock!
```

**In SQL:**
```sql
-- Wrong: Lock rows in random order
UPDATE products 
SET stock = stock - 1 
WHERE id IN (5, 2, 8);  -- PostgreSQL may lock in any order

-- Right: Explicitly order locks
SELECT * FROM products 
WHERE id IN (5, 2, 8)
ORDER BY id  -- Ensures consistent lock order: 2, 5, 8
FOR UPDATE;

UPDATE products 
SET stock = stock - 1 
WHERE id IN (5, 2, 8);
```

#### 2. **Lock Early with SELECT FOR UPDATE**

**❌ Wrong - Late Locking:**
```sql
BEGIN;

-- Read data (no lock)
SELECT balance FROM accounts WHERE id = 1;
-- Application logic...
SELECT balance FROM accounts WHERE id = 2;

-- Lock during update (too late!)
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;
```

**✅ Right - Early Locking:**
```sql
BEGIN;

-- Lock immediately in consistent order
SELECT balance FROM accounts 
WHERE id IN (1, 2)
ORDER BY id  -- Consistent order!
FOR UPDATE;

-- Now safe to update in any order
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;
```

#### 3. **Keep Transactions Short**

**❌ Wrong - Long Transaction:**
```python
conn.autocommit = False
cursor.execute("UPDATE products SET stock = stock - 1 WHERE id = 5")

# Expensive operations while holding locks!
send_email_confirmation()        # 2 seconds
call_external_payment_api()      # 3 seconds
generate_pdf_invoice()           # 5 seconds

cursor.execute("INSERT INTO orders (...) VALUES (...)")
conn.commit()
# Held locks for 10+ seconds!
```

**✅ Right - Short Transaction:**
```python
# Do expensive work OUTSIDE transaction
email_body = prepare_email()
payment_result = process_payment()
pdf_data = generate_invoice()

# Quick transaction
conn.autocommit = False
cursor.execute("UPDATE products SET stock = stock - 1 WHERE id = 5")
cursor.execute("INSERT INTO orders (...) VALUES (...)")
conn.commit()
# Held locks for ~10ms
```

#### 4. **Lower Isolation Levels**

```sql
-- SERIALIZABLE: Most strict, more deadlocks
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
-- High risk of serialization failures

-- READ COMMITTED: Default, fewer deadlocks
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
-- Lower risk, suitable for most use cases
```

#### 5. **Batch Operations with Ordering**

```sql
-- Update multiple rows in consistent order
UPDATE order_items oi
SET status = 'shipped'
FROM (
    SELECT id FROM order_items 
    WHERE order_id = 123
    ORDER BY id  -- Consistent order prevents deadlocks
) AS ordered
WHERE oi.id = ordered.id;
```

---

### Handling Deadlocks in Application Code

#### 1. **Retry Logic with Exponential Backoff**

**Python (psycopg2):**
```python
import time
import psycopg2

def transfer_money_with_retry(from_id, to_id, amount, max_retries=3):
    for attempt in range(max_retries):
        try:
            conn = psycopg2.connect(...)
            cursor = conn.cursor()
            
            # Consistent lock ordering
            ids = sorted([from_id, to_id])
            cursor.execute("""
                SELECT id, balance FROM accounts 
                WHERE id = ANY(%s)
                ORDER BY id
                FOR UPDATE
            """, (ids,))
            
            # Perform transfer
            cursor.execute(
                "UPDATE accounts SET balance = balance - %s WHERE id = %s",
                (amount, from_id)
            )
            cursor.execute(
                "UPDATE accounts SET balance = balance + %s WHERE id = %s",
                (amount, to_id)
            )
            
            conn.commit()
            return True  # Success!
            
        except psycopg2.extensions.TransactionRollbackError as e:
            if "deadlock detected" in str(e):
                conn.rollback()
                if attempt < max_retries - 1:
                    # Exponential backoff: 0.1s, 0.2s, 0.4s
                    wait_time = 0.1 * (2 ** attempt)
                    print(f"Deadlock detected, retry {attempt + 1}/{max_retries} after {wait_time}s")
                    time.sleep(wait_time)
                else:
                    print("Max retries exceeded")
                    raise
            else:
                raise
        finally:
            cursor.close()
            conn.close()
    
    return False
```

**Java (JDBC):**
```java
public boolean transferMoneyWithRetry(int fromId, int toId, double amount) {
    int maxRetries = 3;
    
    for (int attempt = 0; attempt < maxRetries; attempt++) {
        try (Connection conn = dataSource.getConnection()) {
            conn.setAutoCommit(false);
            
            // Consistent lock ordering
            int[] ids = {Math.min(fromId, toId), Math.max(fromId, toId)};
            
            PreparedStatement lockStmt = conn.prepareStatement(
                "SELECT id, balance FROM accounts WHERE id = ANY(?) ORDER BY id FOR UPDATE"
            );
            lockStmt.setArray(1, conn.createArrayOf("INTEGER", ids));
            lockStmt.executeQuery();
            
            // Perform transfer
            PreparedStatement debitStmt = conn.prepareStatement(
                "UPDATE accounts SET balance = balance - ? WHERE id = ?"
            );
            debitStmt.setDouble(1, amount);
            debitStmt.setInt(2, fromId);
            debitStmt.executeUpdate();
            
            PreparedStatement creditStmt = conn.prepareStatement(
                "UPDATE accounts SET balance = balance + ? WHERE id = ?"
            );
            creditStmt.setDouble(1, amount);
            creditStmt.setInt(2, toId);
            creditStmt.executeUpdate();
            
            conn.commit();
            return true;
            
        } catch (SQLException e) {
            if (e.getSQLState().equals("40P01")) {  // Deadlock detected
                if (attempt < maxRetries - 1) {
                    long waitTime = (long) (100 * Math.pow(2, attempt));
                    System.out.println("Deadlock detected, retry " + (attempt + 1) + 
                                     "/" + maxRetries + " after " + waitTime + "ms");
                    Thread.sleep(waitTime);
                } else {
                    throw new RuntimeException("Max retries exceeded", e);
                }
            } else {
                throw e;
            }
        }
    }
    
    return false;
}
```

#### 2. **Graceful Degradation**

```python
def process_order(order_id):
    try:
        # Try immediate processing
        update_inventory_and_create_order(order_id)
    except DeadlockError:
        # Fallback: Queue for background processing
        redis_queue.enqueue('process_order', order_id)
        return {
            'status': 'queued',
            'message': 'Order will be processed shortly'
        }
```

---

### Advanced Deadlock Prevention

#### 1. **Advisory Locks for Explicit Ordering**

```sql
BEGIN;

-- Acquire advisory locks in consistent order
-- Always lock lower ID first
SELECT pg_advisory_xact_lock(LEAST(1, 2));
SELECT pg_advisory_xact_lock(GREATEST(1, 2));

-- Now safe to update in any order
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;  -- Advisory locks automatically released
```

#### 2. **Optimistic Locking with Version Column**

```sql
-- Add version column
ALTER TABLE products ADD COLUMN version INTEGER DEFAULT 0;

-- Update with version check (no locking)
UPDATE products 
SET stock = stock - 1,
    version = version + 1
WHERE id = 123 
  AND version = 5;  -- Only succeeds if version hasn't changed

-- Check affected rows
-- If 0, another transaction modified it → retry
```

**Application Code:**
```python
def update_with_optimistic_locking(product_id, quantity):
    max_retries = 5
    
    for attempt in range(max_retries):
        # Read current version
        cursor.execute(
            "SELECT stock, version FROM products WHERE id = %s",
            (product_id,)
        )
        stock, version = cursor.fetchone()
        
        if stock < quantity:
            raise InsufficientStock()
        
        # Try to update with version check
        cursor.execute("""
            UPDATE products 
            SET stock = stock - %s, version = version + 1
            WHERE id = %s AND version = %s
        """, (quantity, product_id, version))
        
        if cursor.rowcount == 1:
            conn.commit()
            return True  # Success!
        else:
            # Version changed, retry
            conn.rollback()
            time.sleep(0.01 * attempt)
    
    raise MaxRetriesExceeded()
```

#### 3. **Queue-Based Processing (Single Worker)**

```sql
-- Use SKIP LOCKED to avoid blocking
CREATE TABLE job_queue (
    id SERIAL PRIMARY KEY,
    status TEXT DEFAULT 'pending',
    data JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Worker process fetches jobs without locking conflicts
BEGIN;

SELECT * FROM job_queue
WHERE status = 'pending'
ORDER BY id
LIMIT 1
FOR UPDATE SKIP LOCKED;  -- Skip if locked by another worker

UPDATE job_queue SET status = 'processing' WHERE id = 123;

-- Process job...

UPDATE job_queue SET status = 'completed' WHERE id = 123;

COMMIT;
```

**Benefits:**
- Only one worker processes each job
- No deadlocks between workers
- Natural load distribution

---

### Real-World Example: E-Commerce Flash Sale

**Problem:**
```
During a flash sale, we experienced 460 deadlocks in 3 minutes:
- 1,000+ customers buying same popular products simultaneously
- Each order updates 2-5 products
- Different customers buy same products in different orders
- Result: 15% transaction failure rate, angry customers
```

**Investigation:**
```sql
-- Check deadlock frequency
SELECT deadlocks FROM pg_stat_database WHERE datname = 'ecommerce';
-- Result: 460 deadlocks in last 3 minutes

-- Analyze locks
SELECT 
    locktype, 
    relation::regclass, 
    mode, 
    COUNT(*)
FROM pg_locks
WHERE NOT granted
GROUP BY locktype, relation, mode;

-- Found: Multiple processes waiting for RowExclusiveLock on products table
```

**Root Cause:**
```python
# Old code: No lock ordering
def create_order(customer_id, items):
    # items = [(product_id=5, qty=1), (product_id=2, qty=2), (product_id=8, qty=1)]
    for product_id, quantity in items:
        cursor.execute(
            "UPDATE products SET stock = stock - %s WHERE id = %s",
            (quantity, product_id)
        )
    # Customer A locks: 5 → 2 → 8
    # Customer B locks: 2 → 8 → 5
    # Customer C locks: 8 → 5 → 2
    # → Deadlock triangle!
```

**Solution Implemented:**
```python
def create_order(customer_id, items):
    # Sort items by product_id for consistent lock ordering
    sorted_items = sorted(items, key=lambda x: x[0])
    
    # Extract product IDs for locking
    product_ids = [item[0] for item in sorted_items]
    
    # Lock all products in consistent order
    cursor.execute("""
        SELECT id, stock FROM products 
        WHERE id = ANY(%s)
        ORDER BY id
        FOR UPDATE
    """, (product_ids,))
    
    # Now safe to update in sorted order
    for product_id, quantity in sorted_items:
        cursor.execute(
            "UPDATE products SET stock = stock - %s WHERE id = %s",
            (quantity, product_id)
        )
    
    # Customer A locks: 2 → 5 → 8
    # Customer B locks: 2 → 5 → 8  (waits for A)
    # Customer C locks: 2 → 5 → 8  (waits for B)
    # → No deadlock! Sequential processing
```

**Results:**
```
Before:
- 460 deadlocks in 3 minutes (peak traffic)
- 15% transaction failure rate
- ~250ms average response time (including retries)

After:
- 0 deadlocks in 24 hours
- 0% transaction failure rate
- ~50ms average response time (5x faster, no retries needed)
- Successfully handled 10,000+ concurrent orders
```

---

### Best Practices

#### 1. **Always Use Consistent Lock Ordering**
```sql
-- ALWAYS sort by ID before locking
SELECT * FROM table_name 
WHERE id IN (5, 2, 8, 1)
ORDER BY id  -- 1, 2, 5, 8
FOR UPDATE;
```

#### 2. **Lock Early in Transactions**
```sql
BEGIN;
-- Lock immediately
SELECT * FROM accounts WHERE id IN (1, 2) ORDER BY id FOR UPDATE;
-- Then update
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT;
```

#### 3. **Keep Transactions Short**
- Do expensive I/O outside transactions
- Prepare data before BEGIN
- Commit as soon as possible

#### 4. **Implement Retry Logic**
```python
# Always include retry logic for deadlock handling
max_retries = 3
exponential_backoff = True
```

#### 5. **Use READ COMMITTED Isolation**
```sql
-- Default isolation level is usually sufficient
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
-- Avoid SERIALIZABLE unless absolutely necessary
```

#### 6. **Monitor Deadlock Frequency**
```sql
-- Alert if deadlock rate > 0.1%
SELECT 
    deadlocks::float / NULLIF(xact_commit + xact_rollback, 0) * 100 
    AS deadlock_percentage
FROM pg_stat_database
WHERE datname = current_database();
```

#### 7. **Log Deadlock Details**
```sql
ALTER SYSTEM SET log_lock_waits = on;
ALTER SYSTEM SET log_min_duration_statement = 1000;  -- Log slow queries
```

#### 8. **Test Under Load**
```bash
# Use pgbench or custom load testing
pgbench -c 100 -t 1000 -f deadlock_scenario.sql
```

---

### Summary Table

| **Prevention Strategy** | **Effectiveness** | **Complexity** | **When to Use** |
|------------------------|------------------|----------------|-----------------|
| Consistent lock ordering | ⭐⭐⭐⭐⭐ | Low | Always (first line of defense) |
| SELECT FOR UPDATE early | ⭐⭐⭐⭐⭐ | Low | All multi-row updates |
| Short transactions | ⭐⭐⭐⭐ | Low | Always |
| Retry logic | ⭐⭐⭐⭐ | Medium | All applications |
| Advisory locks | ⭐⭐⭐⭐ | Medium | Complex locking patterns |
| Optimistic locking | ⭐⭐⭐ | Medium | High contention, low conflict |
| Queue-based processing | ⭐⭐⭐⭐⭐ | High | Background jobs |
| Lower isolation level | ⭐⭐⭐ | Low | When SERIALIZABLE not needed |

---

### Interview Tips

- **Junior**: "Deadlocks occur when transactions wait for each other. PostgreSQL detects and aborts one. Retry the transaction."
- **Mid-Level**: "Prevent deadlocks by consistent lock ordering, using SELECT FOR UPDATE with ORDER BY, keeping transactions short, and implementing retry logic with exponential backoff"
- **Senior**: "Reduced deadlocks from 460 to 0 in flash sale scenario by ensuring consistent lock ordering (sorting product IDs), early locking with SELECT FOR UPDATE, and implementing retry logic. Also used advisory locks for complex scenarios and queue-based processing for background jobs"
- **Advanced**: "Used advisory locks for complex scenarios, or queue-based processing with single worker to eliminate deadlocks entirely"

</details>

52. What is the pg_stat_statements extension?

<details>
<summary><strong>Answer</strong></summary>

### What is pg_stat_statements?

**pg_stat_statements** is a PostgreSQL extension that tracks execution statistics of all SQL statements executed by the server. It's the **#1 tool for identifying performance bottlenecks** in production databases.

**What It Tracks:**
- Total execution time
- Number of executions
- Mean, min, max execution time
- Rows returned/affected
- Buffer hits/misses (cache efficiency)
- Blocks read/written

**Why It's Essential:**
- Identifies slow queries in production
- Finds queries executed most frequently
- Measures query performance over time
- Tracks cache hit ratios per query
- Provides data for query optimization decisions

---

### Installation and Configuration

#### 1. **Enable the Extension**

```sql
-- Install extension (requires superuser)
CREATE EXTENSION pg_stat_statements;

-- Verify installation
SELECT * FROM pg_extension WHERE extname = 'pg_stat_statements';
```

#### 2. **Configure postgresql.conf**

```conf
# Add to shared_preload_libraries (requires restart)
shared_preload_libraries = 'pg_stat_statements'

# Configuration parameters
pg_stat_statements.track = all          # Track all statements (default: top)
pg_stat_statements.max = 10000          # Track up to 10,000 distinct queries
pg_stat_statements.track_utility = on   # Track utility commands (CREATE, DROP, etc.)
pg_stat_statements.save = on            # Persist stats across restarts
```

**Restart PostgreSQL:**
```bash
# Linux/Mac
sudo systemctl restart postgresql

# Windows
Restart-Service postgresql-x64-14
```

#### 3. **Verify Configuration**

```sql
-- Check if extension is loaded
SELECT * FROM pg_stat_statements LIMIT 1;

-- View configuration
SHOW shared_preload_libraries;
SHOW pg_stat_statements.max;
```

---

### Using pg_stat_statements

#### 1. **View All Tracked Queries**

```sql
SELECT 
    queryid,
    userid::regrole AS user,
    left(query, 60) AS short_query,
    calls,
    total_exec_time,
    mean_exec_time,
    max_exec_time
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;
```

**Example Output:**
```
 queryid |  user   |                short_query                 | calls | total_exec_time | mean_exec_time | max_exec_time 
---------+---------+--------------------------------------------+-------+-----------------+----------------+---------------
 1234567 | webapp  | SELECT * FROM orders WHERE customer_id = $+ |  8234 |     1234567.89  |      150.12    |    2345.67
 2345678 | webapp  | UPDATE products SET stock = stock - $1 WHE+ |  5123 |      876543.21  |      171.05    |    1987.43
 3456789 | analytics| SELECT date_trunc($1, created_at), count(+ | 124   |      654321.00  |     5276.78    |   12345.67
```

#### 2. **Top 10 Slowest Queries (by total time)**

```sql
SELECT 
    calls,
    ROUND(total_exec_time::numeric, 2) AS total_time_ms,
    ROUND(mean_exec_time::numeric, 2) AS avg_time_ms,
    ROUND((100 * total_exec_time / SUM(total_exec_time) OVER())::numeric, 2) AS percentage,
    query
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;
```

**Example Output:**
```
 calls | total_time_ms | avg_time_ms | percentage |                     query
-------+---------------+-------------+------------+-----------------------------------------------
  8234 |   1234567.89  |   150.12    |   35.42    | SELECT * FROM orders WHERE customer_id = $1
  5123 |    876543.21  |   171.05    |   25.16    | UPDATE products SET stock = stock - $1 WHERE...
   124 |    654321.00  |  5276.78    |   18.77    | SELECT date_trunc($1, created_at), count(*)...
```

**Insights:**
- First query: Called 8,234 times, consuming 35.42% of total database time!
- Third query: Only 124 calls but very slow (5.3s average) → optimization candidate

#### 3. **Top 10 Most Frequently Called Queries**

```sql
SELECT 
    calls,
    ROUND(mean_exec_time::numeric, 2) AS avg_ms,
    ROUND(total_exec_time::numeric, 2) AS total_ms,
    left(query, 80) AS query
FROM pg_stat_statements
ORDER BY calls DESC
LIMIT 10;
```

**Example Output:**
```
 calls  | avg_ms | total_ms |                          query
--------+--------+----------+--------------------------------------------------------
 125432 |   2.34 | 293510   | SELECT id, name, email FROM users WHERE id = $1
  89234 |   1.87 | 166867   | SELECT COUNT(*) FROM sessions WHERE user_id = $1
  67890 |  12.45 | 845250   | SELECT * FROM products WHERE category_id = $1 AND...
```

**Insights:**
- First query: 125k calls but very fast (2.34ms) → well-optimized
- Third query: 68k calls with 12.45ms → check if index exists on `category_id`

#### 4. **Queries with Low Cache Hit Ratio**

```sql
SELECT 
    calls,
    ROUND(mean_exec_time::numeric, 2) AS avg_ms,
    shared_blks_hit,
    shared_blks_read,
    ROUND(
        100.0 * shared_blks_hit / NULLIF(shared_blks_hit + shared_blks_read, 0),
        2
    ) AS cache_hit_ratio,
    left(query, 60) AS query
FROM pg_stat_statements
WHERE shared_blks_read > 0
ORDER BY cache_hit_ratio ASC
LIMIT 10;
```

**Example Output:**
```
 calls | avg_ms | shared_blks_hit | shared_blks_read | cache_hit_ratio |             query
-------+--------+-----------------+------------------+-----------------+--------------------------------
  1234 | 456.78 |            5432 |            45678 |           10.63 | SELECT * FROM logs WHERE created
  2345 | 234.56 |           12345 |            23456 |           34.50 | SELECT * FROM orders WHERE status
```

**Insights:**
- First query: Only 10.63% cache hit ratio → reading from disk (slow!)
- Solution: Increase `shared_buffers`, add index, or archive old data

#### 5. **Queries Ordered by Mean Execution Time**

```sql
SELECT 
    calls,
    ROUND(mean_exec_time::numeric, 2) AS avg_ms,
    ROUND(min_exec_time::numeric, 2) AS min_ms,
    ROUND(max_exec_time::numeric, 2) AS max_ms,
    ROUND(stddev_exec_time::numeric, 2) AS stddev_ms,
    left(query, 60) AS query
FROM pg_stat_statements
WHERE calls > 10  -- Ignore rarely called queries
ORDER BY mean_exec_time DESC
LIMIT 10;
```

**Example Output:**
```
 calls | avg_ms |  min_ms  | max_ms  | stddev_ms |                query
-------+--------+----------+---------+-----------+----------------------------------
   124 | 8234.56|  5432.12 | 15678.90|   2345.67 | SELECT * FROM orders o JOIN order
   456 | 6789.01|  4567.89 | 12345.67|   1987.43 | UPDATE inventory SET updated_at =
```

**Insights:**
- High standard deviation (stddev) indicates inconsistent performance
- May need to investigate query plan variations or data distribution issues

---

### Advanced Queries

#### 1. **Find Queries That Would Benefit from Indexing**

```sql
SELECT 
    calls,
    ROUND(mean_exec_time::numeric, 2) AS avg_ms,
    ROUND(total_exec_time::numeric, 2) AS total_ms,
    ROUND((shared_blks_hit + shared_blks_read)::numeric / NULLIF(calls, 0), 2) AS avg_blocks_per_call,
    query
FROM pg_stat_statements
WHERE 
    calls > 100
    AND mean_exec_time > 10
    AND (shared_blks_hit + shared_blks_read) / NULLIF(calls, 0) > 100
ORDER BY total_exec_time DESC
LIMIT 10;
```

**Interpretation:**
- `avg_blocks_per_call > 100`: Reading lots of blocks → likely sequential scan
- Combined with high execution time → needs index

#### 2. **Queries Causing Most I/O Wait**

```sql
SELECT 
    calls,
    ROUND(mean_exec_time::numeric, 2) AS avg_ms,
    ROUND(blk_read_time::numeric, 2) AS read_time_ms,  -- Time waiting for disk reads
    ROUND(blk_write_time::numeric, 2) AS write_time_ms, -- Time waiting for disk writes
    ROUND(
        (blk_read_time + blk_write_time) / NULLIF(total_exec_time, 0) * 100,
        2
    ) AS io_percentage,
    left(query, 60) AS query
FROM pg_stat_statements
WHERE blk_read_time + blk_write_time > 0
ORDER BY blk_read_time + blk_write_time DESC
LIMIT 10;
```

**Example Output:**
```
 calls | avg_ms | read_time_ms | write_time_ms | io_percentage |           query
-------+--------+--------------+---------------+---------------+----------------------------
  2345 | 234.56 |     345678.90|        1234.56|         85.67 | SELECT * FROM large_table
```

**Insights:**
- 85.67% of query time spent on I/O → increase `shared_buffers` or optimize query

#### 3. **Compare Query Performance Over Time**

```sql
-- Create a snapshot table
CREATE TABLE pg_stat_statements_history AS
SELECT 
    now() AS captured_at,
    queryid,
    query,
    calls,
    total_exec_time,
    mean_exec_time,
    stddev_exec_time,
    shared_blks_hit,
    shared_blks_read
FROM pg_stat_statements;

-- Later, compare with current stats
WITH current AS (
    SELECT * FROM pg_stat_statements
),
previous AS (
    SELECT * FROM pg_stat_statements_history
    WHERE captured_at = (SELECT MAX(captured_at) FROM pg_stat_statements_history)
)
SELECT 
    c.queryid,
    left(c.query, 60) AS query,
    c.calls - COALESCE(p.calls, 0) AS new_calls,
    ROUND((c.total_exec_time - COALESCE(p.total_exec_time, 0))::numeric, 2) AS new_total_time,
    ROUND(
        ((c.total_exec_time - COALESCE(p.total_exec_time, 0)) / 
         NULLIF(c.calls - COALESCE(p.calls, 0), 0))::numeric, 
        2
    ) AS new_avg_time
FROM current c
LEFT JOIN previous p USING (queryid)
WHERE c.calls - COALESCE(p.calls, 0) > 0
ORDER BY new_total_time DESC
LIMIT 10;
```

#### 4. **Reset Statistics**

```sql
-- Reset all statistics (careful in production!)
SELECT pg_stat_statements_reset();

-- Reset statistics for specific query
SELECT pg_stat_statements_reset(userid, dbid, queryid)
FROM pg_stat_statements
WHERE query LIKE '%specific_query%';
```

---

### Real-World Example: Identifying Performance Bottleneck

**Scenario:**
```
E-commerce site experiencing slow page loads during peak hours.
Dashboard showing high CPU and long response times.
Need to identify the culprit queries.
```

**Step 1: Find Slowest Queries**
```sql
SELECT 
    calls,
    ROUND(total_exec_time::numeric, 2) AS total_ms,
    ROUND(mean_exec_time::numeric, 2) AS avg_ms,
    ROUND((100 * total_exec_time / SUM(total_exec_time) OVER())::numeric, 2) AS pct_total,
    query
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 5;
```

**Output:**
```
 calls | total_ms  | avg_ms | pct_total |                     query
-------+-----------+--------+-----------+------------------------------------------------
  8234 | 1543789.23| 187.54 |   42.15   | SELECT p.*, c.name FROM products p LEFT JOIN...
  5642 |  987654.32| 175.09 |   26.98   | SELECT o.* FROM orders o WHERE customer_id = $1
  2341 |  543210.98| 231.97 |   14.83   | UPDATE cart_items SET quantity = $1 WHERE id = $2
```

**Finding:** First query consuming 42% of total database time!

**Step 2: Analyze the Slow Query**
```sql
-- Get full query text
SELECT query FROM pg_stat_statements WHERE queryid = 1234567890;

-- Result:
SELECT p.*, c.name AS category_name
FROM products p
LEFT JOIN categories c ON p.category_id = c.id
WHERE p.in_stock = true
ORDER BY p.created_at DESC;
```

**Step 3: Check Execution Plan**
```sql
EXPLAIN ANALYZE
SELECT p.*, c.name AS category_name
FROM products p
LEFT JOIN categories c ON p.category_id = c.id
WHERE p.in_stock = true
ORDER BY p.created_at DESC;
```

**Output:**
```
Sort  (cost=15678.90..16234.56 rows=8234 width=256) (actual time=187.234..189.567 rows=8234 loops=1)
  Sort Key: p.created_at DESC
  ->  Hash Join  (cost=234.56..5678.90 rows=8234 width=256) (actual time=12.345..145.678 rows=8234 loops=1)
        ->  Seq Scan on products p  (cost=0.00..4567.89 rows=8234 width=240) (actual time=0.123..98.456 rows=8234 loops=1)
              Filter: in_stock = true
        ->  Hash  (cost=123.45..123.45 rows=45 width=24) (actual time=1.234..1.234 rows=45 loops=1)
              ->  Seq Scan on categories c  (cost=0.00..123.45 rows=45 width=24) (actual time=0.012..0.789 rows=45 loops=1)
```

**Problem Identified:** Sequential scan on `products` table filtering by `in_stock`

**Step 4: Create Index**
```sql
CREATE INDEX idx_products_in_stock_created 
ON products(in_stock, created_at DESC);
```

**Step 5: Verify Improvement**
```sql
-- Wait a few minutes, then check again
SELECT 
    calls,
    ROUND(mean_exec_time::numeric, 2) AS avg_ms
FROM pg_stat_statements
WHERE query LIKE '%products p LEFT JOIN categories%';
```

**Result:**
```
Before: 187.54ms average
After:   12.34ms average
Improvement: 15.2x faster! 🎉
```

**Step 6: Check Overall Impact**
```sql
SELECT 
    ROUND(SUM(total_exec_time)::numeric, 2) AS total_db_time_ms,
    ROUND((SUM(total_exec_time) / 3600000)::numeric, 2) AS total_db_time_hours
FROM pg_stat_statements;
```

**Result:**
```
Before optimization:
- Total DB time: 3,661,234ms (1.02 hours in last hour)
- CPU usage: 85%

After optimization:
- Total DB time: 2,117,445ms (0.59 hours in last hour)
- CPU usage: 42%
- Improvement: 42% reduction in database load
```

---

### Best Practices

#### 1. **Always Enable in Production**
```conf
# postgresql.conf
shared_preload_libraries = 'pg_stat_statements'
pg_stat_statements.max = 10000
pg_stat_statements.track = all
```

#### 2. **Regular Monitoring**
```sql
-- Run daily to identify slow queries
SELECT 
    left(query, 80),
    calls,
    ROUND(mean_exec_time::numeric, 2) AS avg_ms
FROM pg_stat_statements
WHERE calls > 100
ORDER BY mean_exec_time DESC
LIMIT 20;
```

#### 3. **Combine with EXPLAIN ANALYZE**
```sql
-- Always run EXPLAIN ANALYZE after finding slow query in pg_stat_statements
EXPLAIN (ANALYZE, BUFFERS, VERBOSE)
<query_from_pg_stat_statements>;
```

#### 4. **Set Appropriate max Value**
```conf
# Small database (< 10 tables)
pg_stat_statements.max = 1000

# Medium database (10-100 tables)
pg_stat_statements.max = 5000

# Large database (100+ tables)
pg_stat_statements.max = 10000
```

#### 5. **Archive Historical Data**
```sql
-- Create weekly snapshots for trend analysis
INSERT INTO pg_stat_statements_archive
SELECT now(), * FROM pg_stat_statements;

-- Reset after archiving (optional)
SELECT pg_stat_statements_reset();
```

#### 6. **Monitor Statement Evictions**
```sql
-- Check if max is too low
SELECT 
    (SELECT setting::int FROM pg_settings WHERE name = 'pg_stat_statements.max') AS max_statements,
    COUNT(*) AS current_statements,
    CASE 
        WHEN COUNT(*) >= (SELECT setting::int FROM pg_settings WHERE name = 'pg_stat_statements.max') * 0.9 
        THEN 'WARNING: Consider increasing pg_stat_statements.max'
        ELSE 'OK'
    END AS status
FROM pg_stat_statements;
```

---

### Common Issues and Solutions

#### Issue 1: Extension Not Available
```sql
-- Error: extension "pg_stat_statements" is not available
```

**Solution:**
```conf
# Add to postgresql.conf
shared_preload_libraries = 'pg_stat_statements'

# Restart PostgreSQL
sudo systemctl restart postgresql

# Then create extension
CREATE EXTENSION pg_stat_statements;
```

#### Issue 2: Queries Not Appearing
```sql
-- Check configuration
SHOW pg_stat_statements.track;

-- Should be 'all' or 'top'
ALTER SYSTEM SET pg_stat_statements.track = 'all';
SELECT pg_reload_conf();
```

#### Issue 3: Query Text Truncated
```conf
# Increase query length limit
track_activity_query_size = 4096  # Default: 1024

# Restart required
```

#### Issue 4: Performance Impact
```
pg_stat_statements has minimal overhead (~0.5-2% CPU)
If concerned, use lower max value or disable in development
```

---

### Comparison with Other Tools

| **Tool** | **Purpose** | **Real-Time** | **Historical** | **Overhead** |
|----------|-------------|---------------|----------------|--------------|
| **pg_stat_statements** | Query performance statistics | ❌ | ✅ | Very Low (~1%) |
| **EXPLAIN ANALYZE** | Single query execution plan | ✅ | ❌ | High (runs query) |
| **pg_stat_activity** | Current running queries | ✅ | ❌ | None |
| **auto_explain** | Log slow query plans | ✅ | ✅ (logs) | Low |
| **pgBadger** | Log file analyzer | ❌ | ✅ | None (post-processing) |

**When to Use:**
- **pg_stat_statements**: First line of defense for production performance issues
- **EXPLAIN ANALYZE**: Deep dive into specific slow queries
- **pg_stat_activity**: Real-time monitoring of active queries
- **Combination**: Use all three together for comprehensive analysis

---

### Summary

**pg_stat_statements is essential because:**
1. ✅ **Identifies bottlenecks**: Finds queries consuming most database time
2. ✅ **Tracks trends**: Shows performance degradation over time
3. ✅ **Prioritizes optimization**: Focus on queries with biggest impact
4. ✅ **Measures improvement**: Validates that optimizations work
5. ✅ **Low overhead**: < 2% performance impact
6. ✅ **Production-ready**: Safe for always-on monitoring

**Typical Workflow:**
```
1. Enable pg_stat_statements in production
2. Query for slowest/most frequent queries daily
3. Run EXPLAIN ANALYZE on identified slow queries
4. Apply optimizations (indexes, rewrites)
5. Verify improvement in pg_stat_statements
```

---

### Interview Tips

- **Junior**: "pg_stat_statements tracks statistics for all executed queries, showing execution counts, average time, and total time. Helps identify slow queries in production"
- **Mid-Level**: "Used pg_stat_statements to find queries consuming most database time, analyzing by total_exec_time and mean_exec_time. Combined with EXPLAIN ANALYZE to identify missing indexes. Reduced one query from 187ms to 12ms (15x faster)"
- **Senior**: "Implemented monitoring system using pg_stat_statements with daily snapshots. Identified query consuming 42% of database time, added compound index, reduced overall CPU usage from 85% to 42%. Created dashboards tracking query performance trends and cache hit ratios"
- **Advanced**: "Built automated query optimization pipeline: pg_stat_statements identifies slow queries, triggers EXPLAIN ANALYZE collection, suggests index candidates, creates indexes in staging, validates improvement, promotes to production. Reduced P95 response time from 450ms to 85ms"

</details>

53. How do you identify and fix N+1 query problems?

<details>
<summary><strong>Answer</strong></summary>

### What is the N+1 Query Problem?

The **N+1 query problem** occurs when an application executes 1 query to fetch N records, then executes N additional queries (one for each record) to fetch related data. This results in **N+1 total queries** instead of a more efficient single query or small number of queries.

**Classic Example:**
```
Problem: Fetch 100 blog posts with their authors

Query 1: SELECT * FROM posts LIMIT 100;              -- 1 query
Query 2: SELECT * FROM users WHERE id = 1;           -- For post 1
Query 3: SELECT * FROM users WHERE id = 2;           -- For post 2
Query 4: SELECT * FROM users WHERE id = 3;           -- For post 3
...
Query 101: SELECT * FROM users WHERE id = 100;       -- For post 100

Total: 101 queries (1 + 100)
```

**Why It's Bad:**
- 100x more database round trips
- Network latency multiplied by 100
- Connection pool exhaustion
- Database server overhead (query parsing, planning, execution)
- Response time: 5ms × 101 queries = 505ms instead of 10ms

---

### Identifying N+1 Problems

#### 1. **Application Performance Monitoring (APM)**

**Tools:**
- **New Relic**: Shows database query patterns
- **DataDog APM**: Detects repeated similar queries
- **Scout APM**: N+1 detection built-in
- **AppSignal**: Highlights N+1 queries automatically

**Example APM Output:**
```
⚠️ N+1 Query Detected
Query executed 100 times in 250ms:
SELECT * FROM users WHERE id = $1

Called from:
  app/controllers/posts_controller.rb:12
  
Recommendation: Use eager loading (includes, preload, or joins)
```

#### 2. **Query Logging Analysis**

**Enable Query Logging:**
```sql
-- PostgreSQL: Log all queries
ALTER SYSTEM SET log_statement = 'all';
ALTER SYSTEM SET log_duration = on;
SELECT pg_reload_conf();
```

**Analyze Logs:**
```bash
# Count repeated query patterns
cat postgresql.log | grep "SELECT \* FROM users WHERE id" | wc -l
# Output: 100 (N+1 detected!)

# Use pgBadger for automated analysis
pgbadger postgresql.log -o report.html
```

#### 3. **pg_stat_statements**

```sql
-- Find queries executed many times
SELECT 
    calls,
    ROUND(mean_exec_time::numeric, 2) AS avg_ms,
    ROUND(total_exec_time::numeric, 2) AS total_ms,
    left(query, 80) AS query
FROM pg_stat_statements
WHERE calls > 100  -- Suspiciously high call count
ORDER BY calls DESC
LIMIT 20;
```

**Example Output:**
```
 calls  | avg_ms | total_ms |                     query
--------+--------+----------+------------------------------------------------
 125000 |   2.5  | 312500   | SELECT * FROM users WHERE id = $1
 125000 |   1.8  | 225000   | SELECT * FROM comments WHERE post_id = $1
```

**Red Flag:** Query called 125,000 times! Likely N+1 problem.

#### 4. **ORM-Specific Tools**

**Rails (ActiveRecord):**
```ruby
# Add to Gemfile
gem 'bullet'

# config/environments/development.rb
config.after_initialize do
  Bullet.enable = true
  Bullet.alert = true
  Bullet.console = true
  Bullet.rails_logger = true
end

# Output when N+1 detected:
# N+1 Query detected
#   Post => [:user]
#   Add to your finder: :includes => [:user]
```

**Django:**
```python
# Install django-debug-toolbar
pip install django-debug-toolbar

# Shows:
# ⚠️ 101 queries (SIMILAR 100)
# SELECT * FROM users WHERE id = ?
# Recommendation: Use select_related() or prefetch_related()
```

**Sequelize (Node.js):**
```javascript
// Enable logging
const sequelize = new Sequelize({
  logging: (sql, timing) => {
    console.log(sql, timing);
  }
});

// Manual check: Count queries
let queryCount = 0;
sequelize.options.logging = () => queryCount++;
```

#### 5. **Manual Testing with Query Counter**

**Python/Django Example:**
```python
from django.test.utils import override_settings
from django.db import connection
from django.test import TestCase

class TestN1Queries(TestCase):
    def test_post_list_queries(self):
        # Create test data
        user = User.objects.create(name="Test User")
        for i in range(10):
            Post.objects.create(title=f"Post {i}", author=user)
        
        # Reset query counter
        connection.queries_log.clear()
        
        # Execute endpoint
        response = self.client.get('/posts/')
        
        # Check query count
        query_count = len(connection.queries)
        print(f"Queries executed: {query_count}")
        
        # Assert reasonable number
        self.assertLess(query_count, 5, "Too many queries (N+1 detected!)")
```

**Output:**
```
AssertionError: 11 not less than 5 : Too many queries (N+1 detected!)

Queries executed:
1. SELECT * FROM posts
2. SELECT * FROM users WHERE id = 1
3. SELECT * FROM users WHERE id = 1
4. SELECT * FROM users WHERE id = 1
... (10 total user queries)
```

---

### Fixing N+1 Problems

#### Solution 1: **Eager Loading with JOIN**

**❌ N+1 Problem:**
```sql
-- Application code (pseudo):
posts = SELECT * FROM posts LIMIT 100;
for post in posts:
    author = SELECT * FROM users WHERE id = post.user_id;  -- 100 queries!
```

**✅ Fixed with JOIN:**
```sql
-- Single query
SELECT 
    p.id,
    p.title,
    p.content,
    u.id AS author_id,
    u.name AS author_name,
    u.email AS author_email
FROM posts p
INNER JOIN users u ON p.user_id = u.id
LIMIT 100;

-- Result: 1 query instead of 101 (101x faster!)
```

**Performance Comparison:**
```
N+1 Approach:
- Queries: 101
- Time: 505ms (5ms per query × 101)
- Network round trips: 101

JOIN Approach:
- Queries: 1
- Time: 10ms
- Network round trips: 1
- Improvement: 50x faster! 🎉
```

#### Solution 2: **Eager Loading with IN Clause**

When JOIN produces duplicate rows or you need separate collections:

**❌ N+1 Problem:**
```python
# Django ORM
posts = Post.objects.all()[:100]
for post in posts:
    print(post.author.name)  # 100 queries!
```

**✅ Fixed with IN:**
```python
# Django
posts = Post.objects.select_related('author')[:100]

# Generated SQL:
# SELECT posts.*, users.* 
# FROM posts 
# INNER JOIN users ON posts.user_id = users.id 
# LIMIT 100
```

**Alternative with separate query:**
```sql
-- Query 1: Get posts
SELECT * FROM posts LIMIT 100;
-- Returns: post_ids [1, 2, 3, ..., 100] with user_ids [5, 12, 5, 8, ...]

-- Query 2: Get all unique authors in one query
SELECT * FROM users WHERE id IN (5, 12, 8, ...);  -- Only unique IDs

-- Total: 2 queries instead of 101
```

**Python Implementation:**
```python
# Fetch posts
posts = execute("SELECT * FROM posts LIMIT 100")

# Extract unique user IDs
user_ids = list(set(post.user_id for post in posts))

# Fetch all users in one query
users = execute("SELECT * FROM users WHERE id = ANY(%s)", (user_ids,))

# Create lookup dictionary
user_dict = {user.id: user for user in users}

# Map authors to posts (in-memory)
for post in posts:
    post.author = user_dict[post.user_id]

# Result: 2 queries (50x faster than 101!)
```

#### Solution 3: **ORM Eager Loading**

**Rails (ActiveRecord):**
```ruby
# ❌ N+1 Problem
@posts = Post.limit(100)
# In view: <%= post.user.name %> → 100 queries

# ✅ Fix with includes (LEFT JOIN)
@posts = Post.includes(:user).limit(100)
# 1-2 queries total

# ✅ Fix with joins (INNER JOIN)
@posts = Post.joins(:user).limit(100)
# 1 query, but duplicates if user has multiple posts

# ✅ Fix with preload (separate queries)
@posts = Post.preload(:user).limit(100)
# 2 queries: 1 for posts, 1 for all users

# ✅ Fix with eager_load (LEFT JOIN, force)
@posts = Post.eager_load(:user).limit(100)
# 1 query with LEFT JOIN
```

**Django:**
```python
# ❌ N+1 Problem
posts = Post.objects.all()[:100]
# In template: {{ post.author.name }} → 100 queries

# ✅ Fix with select_related (JOIN, for ForeignKey/OneToOne)
posts = Post.objects.select_related('author')[:100]
# 1 query with JOIN

# ✅ Fix with prefetch_related (separate queries, for ManyToMany)
posts = Post.objects.prefetch_related('comments')[:100]
# 2 queries: posts + all related comments
```

**Node.js (Sequelize):**
```javascript
// ❌ N+1 Problem
const posts = await Post.findAll({ limit: 100 });
for (const post of posts) {
  console.log(post.user.name);  // 100 queries!
}

// ✅ Fix with include
const posts = await Post.findAll({
  limit: 100,
  include: [{ model: User, as: 'author' }]
});
// 1 query with JOIN
```

**Java (Hibernate):**
```java
// ❌ N+1 Problem
List<Post> posts = session.createQuery("FROM Post").setMaxResults(100).list();
for (Post post : posts) {
    System.out.println(post.getUser().getName());  // 100 queries!
}

// ✅ Fix with JOIN FETCH
List<Post> posts = session.createQuery(
    "FROM Post p JOIN FETCH p.user"
).setMaxResults(100).list();
// 1 query

// ✅ Fix with Entity Graph
EntityGraph graph = entityManager.getEntityGraph("post-with-user");
List<Post> posts = entityManager.createQuery("FROM Post", Post.class)
    .setHint("javax.persistence.fetchgraph", graph)
    .setMaxResults(100)
    .getResultList();
```

#### Solution 4: **Nested Relationships**

When loading multiple levels of relationships:

**❌ Nested N+1:**
```ruby
# Rails: Load posts → users → companies
@posts = Post.limit(100)
# In view:
# <%= post.user.name %> → 100 queries
# <%= post.user.company.name %> → 100 MORE queries
# Total: 201 queries!
```

**✅ Fixed:**
```ruby
# Load all relationships at once
@posts = Post.includes(user: :company).limit(100)
# 3 queries:
# 1. SELECT * FROM posts LIMIT 100
# 2. SELECT * FROM users WHERE id IN (...)
# 3. SELECT * FROM companies WHERE id IN (...)
# Total: 3 queries instead of 201 (67x faster!)
```

**Django Example:**
```python
# ❌ N+1 x N+1 = N² problem!
posts = Post.objects.all()[:100]
for post in posts:
    print(post.author.name)  # 100 queries
    for comment in post.comments.all():  # 100 queries
        print(comment.user.name)  # 100 × avg_comments queries!
# Total: ~10,000+ queries! 😱

# ✅ Fixed
posts = Post.objects.select_related('author').prefetch_related(
    Prefetch('comments', queryset=Comment.objects.select_related('user'))
)[:100]
# 3 queries total
```

#### Solution 5: **DataLoader Pattern (GraphQL)**

For GraphQL APIs to batch and cache database calls:

**JavaScript (DataLoader):**
```javascript
const DataLoader = require('dataloader');

// Create a batch loading function
const userLoader = new DataLoader(async (userIds) => {
  // Called once with all user IDs needed in this request
  const users = await User.findAll({
    where: { id: userIds }
  });
  
  // Return users in same order as userIds
  const userMap = new Map(users.map(u => [u.id, u]));
  return userIds.map(id => userMap.get(id));
});

// Usage in GraphQL resolver
const resolvers = {
  Post: {
    author: (post) => userLoader.load(post.userId)
    // Instead of 100 queries, DataLoader batches into 1:
    // SELECT * FROM users WHERE id IN (1, 2, 3, ..., 100)
  }
};
```

**Python (Strawberry/DataLoader):**
```python
from strawberry.dataloader import DataLoader

async def load_users(keys: list[int]) -> list[User]:
    users = await User.filter(id__in=keys)
    user_map = {user.id: user for user in users}
    return [user_map.get(key) for key in keys]

user_loader = DataLoader(load_fn=load_users)

@strawberry.type
class Post:
    @strawberry.field
    async def author(self) -> User:
        return await user_loader.load(self.user_id)
```

---

### Real-World Example: Blog Platform

**Problem:**
```
Blog listing page showing 50 posts with author info.
Page load time: 2.5 seconds
Database queries: 51 (1 + 50)
Users complaining about slow performance.
```

**Investigation:**
```python
# Enable query logging
import logging
logging.basicConfig()
logging.getLogger('sqlalchemy.engine').setLevel(logging.INFO)

# Test endpoint
response = client.get('/posts')

# Log output:
SELECT * FROM posts ORDER BY created_at DESC LIMIT 50
SELECT * FROM users WHERE id = 1
SELECT * FROM users WHERE id = 2
SELECT * FROM users WHERE id = 1  # Duplicate! No caching
...
(51 queries total)
```

**Original Code:**
```python
# Flask with SQLAlchemy
@app.route('/posts')
def list_posts():
    posts = Post.query.order_by(Post.created_at.desc()).limit(50).all()
    return render_template('posts.html', posts=posts)

# Template: posts.html
{% for post in posts %}
    <h2>{{ post.title }}</h2>
    <p>By {{ post.author.name }}</p>  {# N+1 here! #}
{% endfor %}
```

**Fixed Code:**
```python
@app.route('/posts')
def list_posts():
    # Add joinedload to eager load authors
    posts = Post.query.options(
        joinedload(Post.author)
    ).order_by(Post.created_at.desc()).limit(50).all()
    return render_template('posts.html', posts=posts)

# Generated SQL:
SELECT 
    posts.id, posts.title, posts.content, posts.created_at,
    users.id AS users_id, users.name, users.email
FROM posts 
LEFT JOIN users ON users.id = posts.user_id
ORDER BY posts.created_at DESC 
LIMIT 50
```

**Results:**
```
Before:
- Queries: 51
- Time: 2500ms
- Database CPU: 15%

After:
- Queries: 1
- Time: 45ms
- Database CPU: 2%

Improvement: 55x faster! 🚀
```

**Advanced: Multiple Relationships**
```python
# Load posts with authors AND latest 5 comments with their users
posts = Post.query.options(
    joinedload(Post.author),
    selectinload(Post.comments).joinedload(Comment.user)
).order_by(Post.created_at.desc()).limit(50).all()

# Generates efficient queries:
# 1. SELECT posts.*, users.* FROM posts JOIN users ... LIMIT 50
# 2. SELECT comments.* FROM comments WHERE post_id IN (...)
# 3. SELECT users.* FROM users WHERE id IN (...)
# Total: 3 queries instead of 1 + 50 + (50 × 5) = 301 queries
# Improvement: 100x faster!
```

---

### Monitoring and Prevention

#### 1. **Automated Testing**

```python
# pytest with query counter
@pytest.fixture
def query_counter(db):
    from django.db import connection
    connection.queries_log.clear()
    yield
    print(f"Queries executed: {len(connection.queries)}")

def test_post_list_performance(client, query_counter):
    # Create test data
    user = User.objects.create(name="Test")
    for i in range(10):
        Post.objects.create(title=f"Post {i}", author=user)
    
    # Test endpoint
    response = client.get('/posts/')
    
    # Assert query count
    query_count = len(connection.queries)
    assert query_count < 5, f"N+1 detected! {query_count} queries"
```

#### 2. **CI/CD Integration**

```yaml
# .github/workflows/test.yml
- name: Run tests with query profiling
  run: |
    pytest --django-db-profiler
    
- name: Check for N+1 queries
  run: |
    if grep -q "POSSIBLE N+1" test-output.log; then
      echo "❌ N+1 queries detected!"
      exit 1
    fi
```

#### 3. **Production Monitoring**

```sql
-- Create monitoring view
CREATE VIEW query_frequency AS
SELECT 
    calls,
    ROUND(mean_exec_time::numeric, 2) AS avg_ms,
    query
FROM pg_stat_statements
WHERE calls > 1000  -- Frequently called
ORDER BY calls DESC;

-- Alert if query frequency spikes
SELECT 
    query,
    calls,
    CASE 
        WHEN calls > 10000 THEN 'CRITICAL: Possible N+1'
        WHEN calls > 5000 THEN 'WARNING: High frequency'
        ELSE 'OK'
    END AS status
FROM query_frequency;
```

#### 4. **Code Review Checklist**

```markdown
# N+1 Query Prevention Checklist

- [ ] All loops that access related models use eager loading
- [ ] Templates don't trigger queries (use {% with %} or prefetch)
- [ ] API endpoints prefetch all needed relationships
- [ ] GraphQL resolvers use DataLoader
- [ ] Query count tested in unit tests
- [ ] No queries inside loops (check with profiler)
```

---

### Best Practices

#### 1. **Always Eager Load Related Data**
```python
# Default: Use eager loading
posts = Post.objects.select_related('author', 'category')

# Not: Lazy loading (triggers N+1)
posts = Post.objects.all()  # Don't do this if you'll access relations
```

#### 2. **Use ORM Query Optimization Tools**
```ruby
# Rails: Use explain
Post.includes(:user).limit(100).explain

# Django: Use debug_toolbar
# Shows query count and duplicates
```

#### 3. **Batch Load in Background Jobs**
```python
# Instead of:
for user in users:
    send_email(user)  # Each might query database

# Do:
user_data = User.objects.select_related('profile', 'preferences').all()
for user in user_data:
    send_email(user)  # All data already loaded
```

#### 4. **Cache Aggressively**
```python
from functools import lru_cache

@lru_cache(maxsize=1000)
def get_user(user_id):
    return User.objects.get(id=user_id)

# Reduces duplicate queries in same request
```

#### 5. **Use Database Views for Complex Queries**
```sql
-- Create materialized view with pre-joined data
CREATE MATERIALIZED VIEW posts_with_authors AS
SELECT 
    p.*,
    u.name AS author_name,
    u.email AS author_email
FROM posts p
JOIN users u ON p.user_id = u.id;

-- Refresh periodically
REFRESH MATERIALIZED VIEW posts_with_authors;

-- Query is now simple and fast
SELECT * FROM posts_with_authors LIMIT 100;
```

---

### Summary Table

| **Detection Method** | **Accuracy** | **Ease** | **When to Use** |
|---------------------|-------------|---------|----------------|
| APM tools (New Relic) | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | Production monitoring |
| Query logging | ⭐⭐⭐⭐ | ⭐⭐⭐ | Development/debugging |
| pg_stat_statements | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | Production analysis |
| ORM tools (Bullet) | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | Development (Rails) |
| Manual testing | ⭐⭐⭐ | ⭐⭐ | Unit tests |

| **Fix Strategy** | **Effectiveness** | **Complexity** | **When to Use** |
|-----------------|------------------|----------------|-----------------|
| JOIN (select_related) | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ForeignKey, OneToOne |
| IN clause (prefetch_related) | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ManyToMany, reverse FK |
| DataLoader | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | GraphQL APIs |
| Caching | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | Frequently accessed data |
| Materialized views | ⭐⭐⭐⭐ | ⭐⭐ | Complex read-heavy queries |

---

### Interview Tips

- **Junior**: "N+1 problem occurs when fetching N records triggers N additional queries for related data. Fixed by using eager loading with JOINs or IN clauses to fetch all data in 1-2 queries"
- **Mid-Level**: "Identified N+1 problem using query logging showing 51 queries (1 + 50). Fixed by adding select_related() to eager load authors, reducing from 51 queries to 1. Page load time improved from 2.5s to 45ms (55x faster)"
- **Senior**: "Implemented N+1 detection in CI/CD pipeline using query counters in tests. Fixed blog listing with nested relationships (posts → authors → companies) by using includes(user: :company), reducing from 201 queries to 3 (67x improvement). Set up production monitoring with pg_stat_statements alerts for queries called > 10k times"
- **Advanced**: "Built automated N+1 detection system analyzing pg_stat_statements for high-frequency patterns. Implemented DataLoader for GraphQL API batching 100 author queries into 1. Created company-wide ORM guidelines and code review checklist. Reduced P95 API response time from 3.2s to 180ms across all services"

</details>

54. What are materialized views and when would you use them?

<details>
<summary><strong>Answer</strong></summary>

### What are Materialized Views?

A **materialized view** is a database object that stores the result of a query physically on disk, unlike a regular view which is just a stored query definition. The data is **pre-computed and cached**, making subsequent reads extremely fast.

**Key Differences:**

| Feature | Regular View | Materialized View |
|---------|-------------|-------------------|
| **Storage** | Query definition only | Pre-computed results stored on disk |
| **Performance** | Executes query every time | Instant (reads cached data) |
| **Data freshness** | Always current | Stale until refreshed |
| **Disk space** | Minimal | Can be large |
| **Indexes** | Cannot add | Can add indexes |
| **Write cost** | None | Refresh cost |

**Analogy:**
- **Regular View** = Recipe (instructions to make a cake)
- **Materialized View** = Pre-baked cake (ready to serve instantly)

---

### Creating Materialized Views

#### 1. **Basic Syntax**

```sql
CREATE MATERIALIZED VIEW view_name AS
SELECT ...
FROM ...
WHERE ...;
```

#### 2. **Simple Example**

```sql
-- Regular view (slow, executes every time)
CREATE VIEW user_order_summary AS
SELECT 
    u.id,
    u.name,
    COUNT(o.id) AS order_count,
    SUM(o.total) AS total_spent,
    AVG(o.total) AS avg_order_value,
    MAX(o.created_at) AS last_order_date
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name;

-- Query regular view (slow: 3.5 seconds for 1M users)
SELECT * FROM user_order_summary WHERE order_count > 10;

-- Materialized view (fast, pre-computed)
CREATE MATERIALIZED VIEW user_order_summary_mv AS
SELECT 
    u.id,
    u.name,
    COUNT(o.id) AS order_count,
    SUM(o.total) AS total_spent,
    AVG(o.total) AS avg_order_value,
    MAX(o.created_at) AS last_order_date
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name;

-- Query materialized view (fast: 15ms for 1M users)
SELECT * FROM user_order_summary_mv WHERE order_count > 10;
-- 233x faster! 🚀
```

#### 3. **Complex Example with Multiple Joins**

```sql
CREATE MATERIALIZED VIEW sales_dashboard AS
SELECT 
    DATE_TRUNC('day', o.created_at) AS sale_date,
    p.category,
    c.country,
    COUNT(DISTINCT o.id) AS order_count,
    COUNT(DISTINCT o.customer_id) AS unique_customers,
    SUM(oi.quantity) AS items_sold,
    SUM(oi.quantity * oi.price) AS revenue,
    AVG(oi.quantity * oi.price) AS avg_order_value
FROM orders o
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id
JOIN customers c ON o.customer_id = c.id
WHERE o.status = 'completed'
GROUP BY DATE_TRUNC('day', o.created_at), p.category, c.country;

-- Add index for fast filtering
CREATE INDEX idx_sales_dashboard_date 
ON sales_dashboard(sale_date);

CREATE INDEX idx_sales_dashboard_category 
ON sales_dashboard(category, sale_date);
```

**Performance:**
```sql
-- Without materialized view: 15.7 seconds
SELECT category, SUM(revenue) 
FROM orders o
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id
WHERE o.created_at >= NOW() - INTERVAL '30 days'
GROUP BY category;

-- With materialized view: 8ms
SELECT category, SUM(revenue)
FROM sales_dashboard
WHERE sale_date >= NOW() - INTERVAL '30 days'
GROUP BY category;

-- Improvement: 1,962x faster!
```

---

### Refreshing Materialized Views

#### 1. **Manual Refresh**

```sql
-- Full refresh (locks view during refresh)
REFRESH MATERIALIZED VIEW user_order_summary_mv;

-- Concurrent refresh (doesn't lock, requires unique index)
REFRESH MATERIALIZED VIEW CONCURRENTLY user_order_summary_mv;
```

**Refresh Times:**
```
Small view (1K rows): ~50ms
Medium view (100K rows): ~2 seconds
Large view (10M rows): ~30 seconds
Very large view (100M rows): ~5 minutes
```

#### 2. **Concurrent Refresh (Non-Blocking)**

```sql
-- Create unique index (required for CONCURRENTLY)
CREATE UNIQUE INDEX user_order_summary_mv_id 
ON user_order_summary_mv(id);

-- Now can refresh without blocking reads
REFRESH MATERIALIZED VIEW CONCURRENTLY user_order_summary_mv;

-- Users can query the view while it refreshes!
SELECT * FROM user_order_summary_mv WHERE order_count > 100;
```

**How CONCURRENTLY Works:**
1. Creates a new temporary copy of the view
2. Computes all the data in the background
3. Atomically swaps the old data with new data
4. No locks on the materialized view during computation

**Important:** Requires a unique index on the view!

#### 3. **Automated Refresh with Cron/Scheduler**

**PostgreSQL pg_cron Extension:**
```sql
-- Install pg_cron
CREATE EXTENSION pg_cron;

-- Schedule refresh every hour
SELECT cron.schedule(
    'refresh-sales-dashboard',
    '0 * * * *',  -- Every hour at minute 0
    $$REFRESH MATERIALIZED VIEW CONCURRENTLY sales_dashboard$$
);

-- Schedule refresh every night at 2 AM
SELECT cron.schedule(
    'refresh-user-summary',
    '0 2 * * *',  -- 2:00 AM daily
    $$REFRESH MATERIALIZED VIEW CONCURRENTLY user_order_summary_mv$$
);

-- View scheduled jobs
SELECT * FROM cron.job;

-- Unschedule job
SELECT cron.unschedule('refresh-sales-dashboard');
```

**Application-Level Scheduling (Python):**
```python
# celery_tasks.py
from celery import Celery
from sqlalchemy import text

app = Celery('tasks')

@app.task
def refresh_materialized_views():
    with engine.connect() as conn:
        # Refresh critical views
        conn.execute(text(
            "REFRESH MATERIALIZED VIEW CONCURRENTLY user_order_summary_mv"
        ))
        conn.execute(text(
            "REFRESH MATERIALIZED VIEW CONCURRENTLY sales_dashboard"
        ))
        conn.commit()

# Schedule with Celery Beat
app.conf.beat_schedule = {
    'refresh-views-every-hour': {
        'task': 'celery_tasks.refresh_materialized_views',
        'schedule': 3600.0,  # Every hour
    },
}
```

**Node.js with node-cron:**
```javascript
const cron = require('node-cron');
const { Pool } = require('pg');

const pool = new Pool({...});

// Refresh every hour
cron.schedule('0 * * * *', async () => {
  try {
    await pool.query(
      'REFRESH MATERIALIZED VIEW CONCURRENTLY sales_dashboard'
    );
    console.log('Materialized view refreshed successfully');
  } catch (error) {
    console.error('Error refreshing view:', error);
  }
});
```

#### 4. **Trigger-Based Refresh (Real-Time Updates)**

For smaller materialized views that need to be nearly real-time:

```sql
-- Create refresh function
CREATE OR REPLACE FUNCTION refresh_user_order_summary()
RETURNS TRIGGER AS $$
BEGIN
    REFRESH MATERIALIZED VIEW CONCURRENTLY user_order_summary_mv;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

-- Trigger on orders table
CREATE TRIGGER refresh_summary_on_order_change
AFTER INSERT OR UPDATE OR DELETE ON orders
FOR EACH STATEMENT
EXECUTE FUNCTION refresh_user_order_summary();
```

**Warning:** This can be expensive for large views! Only use for:
- Small views (< 10K rows)
- Low write frequency (< 10 writes/minute)
- Critical data requiring near real-time accuracy

**Better Alternative:** Use a background job that checks for changes:
```sql
-- Track last refresh time
CREATE TABLE materialized_view_refresh_log (
    view_name TEXT PRIMARY KEY,
    last_refreshed TIMESTAMP
);

-- Function to check if refresh needed
CREATE OR REPLACE FUNCTION needs_refresh(view_name TEXT, max_age INTERVAL)
RETURNS BOOLEAN AS $$
BEGIN
    RETURN (
        SELECT last_refreshed < NOW() - max_age
        FROM materialized_view_refresh_log
        WHERE view_name = $1
    );
END;
$$ LANGUAGE plpgsql;

-- Application checks and refreshes if needed
-- Python example:
def refresh_if_needed(view_name, max_age_minutes=60):
    result = conn.execute("""
        SELECT needs_refresh(%s, %s::INTERVAL)
    """, (view_name, f'{max_age_minutes} minutes'))
    
    if result.fetchone()[0]:
        conn.execute(f"REFRESH MATERIALIZED VIEW CONCURRENTLY {view_name}")
        conn.execute("""
            INSERT INTO materialized_view_refresh_log (view_name, last_refreshed)
            VALUES (%s, NOW())
            ON CONFLICT (view_name) 
            DO UPDATE SET last_refreshed = NOW()
        """, (view_name,))
        return True
    return False
```

---

### When to Use Materialized Views

#### ✅ **Good Use Cases**

**1. Complex Aggregations (Reports/Dashboards)**
```sql
-- Expensive query (8 seconds)
SELECT 
    DATE_TRUNC('month', order_date),
    product_category,
    SUM(revenue),
    AVG(profit_margin),
    COUNT(DISTINCT customer_id)
FROM sales_fact
JOIN product_dim USING (product_id)
JOIN customer_dim USING (customer_id)
WHERE order_date >= '2020-01-01'
GROUP BY DATE_TRUNC('month', order_date), product_category;

-- Materialized view (12ms)
CREATE MATERIALIZED VIEW monthly_sales_summary AS
SELECT ...;
-- Refreshed nightly, acceptable staleness
```

**2. Data Warehouse / Analytics**
```sql
-- Pre-compute metrics for BI tools
CREATE MATERIALIZED VIEW customer_lifetime_value AS
SELECT 
    c.id,
    c.name,
    c.cohort_month,
    COUNT(DISTINCT o.id) AS total_orders,
    SUM(o.total) AS lifetime_value,
    DATE_PART('month', AGE(MAX(o.created_at), MIN(o.created_at))) AS customer_age_months,
    SUM(o.total) / NULLIF(COUNT(DISTINCT o.id), 0) AS avg_order_value
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
GROUP BY c.id, c.name, c.cohort_month;

-- Business intelligence tools query this instantly
```

**3. Multiple Joins on Large Tables**
```sql
-- Query hits 5+ tables with millions of rows
CREATE MATERIALIZED VIEW product_enriched AS
SELECT 
    p.*,
    c.name AS category_name,
    b.name AS brand_name,
    s.name AS supplier_name,
    AVG(r.rating) AS avg_rating,
    COUNT(r.id) AS review_count
FROM products p
JOIN categories c ON p.category_id = c.id
JOIN brands b ON p.brand_id = b.id
JOIN suppliers s ON p.supplier_id = s.id
LEFT JOIN reviews r ON p.id = r.product_id
GROUP BY p.id, c.name, b.name, s.name;

-- API can now query this enriched view instantly
```

**4. Denormalization for Read-Heavy Workloads**
```sql
-- Application reads >> writes (100:1 ratio)
-- Pre-join frequently accessed data
CREATE MATERIALIZED VIEW user_profile_complete AS
SELECT 
    u.id,
    u.email,
    u.name,
    u.created_at,
    p.avatar_url,
    p.bio,
    p.location,
    s.email_notifications,
    s.theme,
    COUNT(o.id) AS order_count,
    SUM(o.total) AS total_spent
FROM users u
LEFT JOIN profiles p ON u.id = p.user_id
LEFT JOIN settings s ON u.id = s.user_id
LEFT JOIN orders o ON u.id = o.customer_id
GROUP BY u.id, u.email, u.name, u.created_at, 
         p.avatar_url, p.bio, p.location,
         s.email_notifications, s.theme;
```

**5. Slow Search/Filter Queries**
```sql
-- Full-text search on multiple fields (slow)
-- Pre-compute searchable text
CREATE MATERIALIZED VIEW products_searchable AS
SELECT 
    p.id,
    p.name,
    p.description,
    p.price,
    c.name AS category,
    b.name AS brand,
    to_tsvector('english', 
        p.name || ' ' || 
        p.description || ' ' || 
        c.name || ' ' || 
        b.name
    ) AS search_vector
FROM products p
JOIN categories c ON p.category_id = c.id
JOIN brands b ON p.brand_id = b.id;

-- Create GIN index on search vector
CREATE INDEX idx_products_search ON products_searchable USING GIN(search_vector);

-- Fast full-text search
SELECT * FROM products_searchable
WHERE search_vector @@ to_tsquery('electronics & wireless');
-- 5ms vs 2.3s without materialized view
```

#### ❌ **Bad Use Cases**

**1. Real-Time Data Requirements**
```sql
-- DON'T: Stock prices, live sports scores, real-time tracking
-- Data staleness is unacceptable
-- Use regular queries or caching instead
```

**2. Frequently Updated Tables**
```sql
-- DON'T: Shopping cart contents, session data
-- Refreshing after every change defeats the purpose
-- Overhead > benefit
```

**3. Small Result Sets**
```sql
-- DON'T: Queries returning < 1000 rows
-- Regular query is fast enough (~10ms)
-- Materialized view overhead not worth it
```

**4. Simple Queries**
```sql
-- DON'T: SELECT * FROM users WHERE id = 123
-- Already fast with proper indexes
-- No complex aggregation to benefit from caching
```

**5. Write-Heavy Workloads**
```sql
-- DON'T: Logs table with 10k inserts/second
-- Constant refreshing would kill performance
-- Use time-series database or aggregation tables instead
```

---

### Real-World Example: E-Commerce Analytics Dashboard

**Problem:**
```
E-commerce company needs analytics dashboard showing:
- Daily sales by category
- Top products
- Customer segments
- Revenue trends

Current situation:
- Dashboard queries hit 6 tables (orders, order_items, products, customers, categories, reviews)
- 150+ concurrent users accessing dashboard
- Page load time: 12-25 seconds
- Database CPU: 95% during business hours
- Queries timing out during peak traffic
```

**Solution: Materialized Views**

**Step 1: Identify Expensive Queries**
```sql
-- Check pg_stat_statements
SELECT 
    calls,
    ROUND(mean_exec_time::numeric, 2) AS avg_ms,
    ROUND(total_exec_time::numeric, 2) AS total_ms,
    left(query, 100) AS query
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 5;
```

**Output:**
```
 calls | avg_ms  | total_ms   | query
-------+---------+------------+-----------------------------------------------
  8234 | 15234.5 | 125456789  | SELECT DATE_TRUNC('day', o.created_at)...
  5432 | 8765.3  | 47623456   | SELECT p.name, COUNT(*) FROM products p...
  3421 | 12345.6 | 42234567   | SELECT c.segment, SUM(o.total) FROM cust...
```

**Step 2: Create Materialized Views**

```sql
-- 1. Daily sales by category
CREATE MATERIALIZED VIEW daily_sales_by_category AS
SELECT 
    DATE_TRUNC('day', o.created_at) AS sale_date,
    c.id AS category_id,
    c.name AS category_name,
    COUNT(DISTINCT o.id) AS order_count,
    COUNT(DISTINCT o.customer_id) AS unique_customers,
    SUM(oi.quantity * oi.price) AS revenue,
    SUM(oi.quantity) AS items_sold,
    AVG(oi.quantity * oi.price) AS avg_order_value
FROM orders o
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id
JOIN categories c ON p.category_id = c.id
WHERE o.status = 'completed'
GROUP BY DATE_TRUNC('day', o.created_at), c.id, c.name;

-- Add indexes
CREATE UNIQUE INDEX daily_sales_pkey ON daily_sales_by_category(sale_date, category_id);
CREATE INDEX daily_sales_date ON daily_sales_by_category(sale_date);

-- 2. Top products summary
CREATE MATERIALIZED VIEW product_performance AS
SELECT 
    p.id,
    p.name,
    p.category_id,
    c.name AS category_name,
    COUNT(DISTINCT oi.order_id) AS times_ordered,
    SUM(oi.quantity) AS units_sold,
    SUM(oi.quantity * oi.price) AS revenue,
    AVG(r.rating) AS avg_rating,
    COUNT(DISTINCT r.id) AS review_count,
    RANK() OVER (PARTITION BY p.category_id ORDER BY SUM(oi.quantity * oi.price) DESC) AS category_rank
FROM products p
JOIN categories c ON p.category_id = c.id
LEFT JOIN order_items oi ON p.id = oi.product_id
LEFT JOIN reviews r ON p.id = r.product_id
GROUP BY p.id, p.name, p.category_id, c.name;

CREATE UNIQUE INDEX product_performance_pkey ON product_performance(id);
CREATE INDEX product_performance_category ON product_performance(category_id, revenue DESC);

-- 3. Customer segments
CREATE MATERIALIZED VIEW customer_segments AS
SELECT 
    c.id,
    c.email,
    c.name,
    c.created_at AS signup_date,
    COUNT(DISTINCT o.id) AS order_count,
    SUM(o.total) AS lifetime_value,
    MAX(o.created_at) AS last_order_date,
    CASE 
        WHEN SUM(o.total) > 10000 THEN 'VIP'
        WHEN SUM(o.total) > 1000 THEN 'High Value'
        WHEN SUM(o.total) > 100 THEN 'Regular'
        ELSE 'Low Value'
    END AS segment,
    CASE 
        WHEN MAX(o.created_at) > NOW() - INTERVAL '30 days' THEN 'Active'
        WHEN MAX(o.created_at) > NOW() - INTERVAL '90 days' THEN 'At Risk'
        ELSE 'Churned'
    END AS status
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
GROUP BY c.id, c.email, c.name, c.created_at;

CREATE UNIQUE INDEX customer_segments_pkey ON customer_segments(id);
CREATE INDEX customer_segments_segment ON customer_segments(segment, status);
```

**Step 3: Set Up Automated Refresh**
```sql
-- Install pg_cron
CREATE EXTENSION pg_cron;

-- Refresh every hour during business hours
SELECT cron.schedule(
    'refresh-sales-dashboard',
    '0 6-23 * * *',  -- Every hour from 6 AM to 11 PM
    $$
    REFRESH MATERIALIZED VIEW CONCURRENTLY daily_sales_by_category;
    REFRESH MATERIALIZED VIEW CONCURRENTLY product_performance;
    REFRESH MATERIALIZED VIEW CONCURRENTLY customer_segments;
    $$
);

-- Full refresh overnight
SELECT cron.schedule(
    'refresh-sales-dashboard-full',
    '0 2 * * *',  -- 2 AM daily
    $$
    REFRESH MATERIALIZED VIEW daily_sales_by_category;
    REFRESH MATERIALIZED VIEW product_performance;
    REFRESH MATERIALIZED VIEW customer_segments;
    $$
);
```

**Step 4: Update Application Queries**
```python
# Before: Complex multi-table query
def get_daily_sales(start_date, end_date):
    query = """
        SELECT 
            DATE_TRUNC('day', o.created_at) AS sale_date,
            c.name AS category,
            SUM(oi.quantity * oi.price) AS revenue
        FROM orders o
        JOIN order_items oi ON o.id = oi.order_id
        JOIN products p ON oi.product_id = p.id
        JOIN categories c ON p.category_id = c.id
        WHERE o.created_at BETWEEN %s AND %s
          AND o.status = 'completed'
        GROUP BY DATE_TRUNC('day', o.created_at), c.name
        ORDER BY sale_date DESC
    """
    return db.execute(query, (start_date, end_date))
    # Execution time: 15.2 seconds

# After: Query materialized view
def get_daily_sales(start_date, end_date):
    query = """
        SELECT 
            sale_date,
            category_name,
            revenue
        FROM daily_sales_by_category
        WHERE sale_date BETWEEN %s AND %s
        ORDER BY sale_date DESC
    """
    return db.execute(query, (start_date, end_date))
    # Execution time: 8ms
    # Improvement: 1,900x faster! 🚀
```

**Results:**
```
Before Materialized Views:
- Dashboard load time: 12-25 seconds
- Database CPU: 95% during peak
- Query timeouts: 23 per hour
- Concurrent users supported: 50
- User complaints: High

After Materialized Views:
- Dashboard load time: 0.3 seconds (40x faster)
- Database CPU: 25% during peak (70% reduction)
- Query timeouts: 0
- Concurrent users supported: 500+ (10x increase)
- User satisfaction: 98%

Refresh overhead:
- 3 views refresh in 45 seconds (hourly)
- Negligible impact on production (done concurrently)
- Data staleness: Max 1 hour (acceptable for analytics)
```

---

### Best Practices

#### 1. **Always Use CONCURRENTLY for Production**
```sql
-- Create unique index first
CREATE UNIQUE INDEX mv_unique_idx ON my_mv(id);

-- Then refresh concurrently (non-blocking)
REFRESH MATERIALIZED VIEW CONCURRENTLY my_mv;
```

#### 2. **Add Appropriate Indexes**
```sql
-- Materialized views can have indexes just like tables
CREATE INDEX idx_mv_date ON sales_mv(sale_date);
CREATE INDEX idx_mv_category ON sales_mv(category, revenue DESC);
```

#### 3. **Monitor Refresh Times**
```sql
-- Track refresh duration
CREATE TABLE mv_refresh_stats (
    view_name TEXT,
    refresh_started TIMESTAMP,
    refresh_completed TIMESTAMP,
    duration_seconds NUMERIC,
    row_count BIGINT
);

-- Log refresh
CREATE OR REPLACE FUNCTION log_mv_refresh(view_name TEXT)
RETURNS VOID AS $$
DECLARE
    start_time TIMESTAMP := clock_timestamp();
    row_count BIGINT;
BEGIN
    EXECUTE 'REFRESH MATERIALIZED VIEW CONCURRENTLY ' || view_name;
    
    EXECUTE 'SELECT COUNT(*) FROM ' || view_name INTO row_count;
    
    INSERT INTO mv_refresh_stats VALUES (
        view_name,
        start_time,
        clock_timestamp(),
        EXTRACT(EPOCH FROM (clock_timestamp() - start_time)),
        row_count
    );
END;
$$ LANGUAGE plpgsql;

-- Use it
SELECT log_mv_refresh('sales_dashboard');

-- Analyze refresh patterns
SELECT 
    view_name,
    AVG(duration_seconds) AS avg_refresh_time,
    MAX(duration_seconds) AS max_refresh_time,
    AVG(row_count) AS avg_rows
FROM mv_refresh_stats
WHERE refresh_started > NOW() - INTERVAL '7 days'
GROUP BY view_name;
```

#### 4. **Set Appropriate Refresh Schedule**
```sql
-- Analytics dashboard: Hourly refresh OK
-- Financial reports: Every 15 minutes
-- Executive summary: Daily refresh sufficient
-- Real-time data: Don't use materialized views!

-- Consider business hours
SELECT cron.schedule(
    'refresh-during-business-hours',
    '0 8-18 * * 1-5',  -- Every hour, 8 AM-6 PM, Mon-Fri
    $$REFRESH MATERIALIZED VIEW CONCURRENTLY sales_mv$$
);
```

#### 5. **Consider Storage Requirements**
```sql
-- Check materialized view size
SELECT 
    schemaname,
    matviewname,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||matviewname)) AS size
FROM pg_matviews
ORDER BY pg_total_relation_size(schemaname||'.'||matviewname) DESC;

-- Example output:
  schemaname |       matviewname        |  size   
------------+--------------------------+---------
 public     | sales_dashboard          | 2.3 GB
 public     | customer_segments        | 456 MB
 public     | product_performance      | 125 MB
```

#### 6. **Drop Unused Materialized Views**
```sql
-- Clean up unused views to save space
DROP MATERIALIZED VIEW IF EXISTS old_unused_view;
```

#### 7. **Version Control View Definitions**
```sql
-- Store view definitions in migration files
-- migrations/2024_12_24_create_sales_mv.sql

CREATE MATERIALIZED VIEW sales_dashboard AS
SELECT ...;

CREATE UNIQUE INDEX ...;

-- Easy to recreate or modify later
```

---

### Alternatives to Materialized Views

| **Alternative** | **When to Use** | **Pros** | **Cons** |
|----------------|----------------|----------|---------|
| **Application Caching** (Redis) | Real-time or near-real-time | Sub-millisecond reads, TTL control | Additional infrastructure, cache invalidation complexity |
| **Indexed Views** (SQL Server) | Automatic updates needed | Always current | Not available in PostgreSQL |
| **Aggregation Tables** | Custom update logic | Full control over updates | More code to maintain |
| **Read Replicas** | Offload read queries | Always current, scales reads | Cost, replication lag |
| **Denormalized Tables** | Write-time optimization | No refresh needed | Complex update logic, consistency issues |
| **Time-Series DB** (TimescaleDB) | Time-series analytics | Optimized for time-based queries | Different query syntax |

---

### Summary

**Materialized Views are ideal when:**
✅ Complex aggregations taking > 1 second  
✅ Query executed frequently (100+ times/day)  
✅ Data staleness acceptable (hourly/daily)  
✅ Read-heavy workload (reads >> writes)  
✅ Multiple joins on large tables  
✅ Analytics, reports, dashboards  

**Avoid Materialized Views when:**
❌ Real-time data required  
❌ Frequently updated base tables  
❌ Simple queries already fast (< 100ms)  
❌ Write-heavy workload  
❌ Small result sets (< 1K rows)  

**Typical Performance Gains:**
- Simple aggregations: **10-50x faster**
- Complex multi-table joins: **100-500x faster**
- Large-scale analytics: **1000x+ faster**

---

### Interview Tips

- **Junior**: "Materialized views store pre-computed query results on disk, making subsequent reads very fast. Unlike regular views, they cache data but need periodic refreshing. Good for complex reports and dashboards."

- **Mid-Level**: "Used materialized views for analytics dashboard with complex multi-table joins. Original query took 15 seconds, materialized view reduced it to 8ms (1,900x faster). Set up hourly concurrent refresh with unique index to avoid locking. Acceptable 1-hour data staleness for business requirements."

- **Senior**: "Implemented materialized view strategy for e-commerce analytics: created views for daily sales, customer segments, and product performance. Reduced dashboard load time from 12s to 0.3s (40x faster) and database CPU from 95% to 25%. Set up pg_cron for automated concurrent refreshes during business hours, full refresh overnight. Added monitoring for refresh duration and row counts. Supported 10x more concurrent users."

- **Advanced**: "Designed comprehensive materialized view architecture with refresh orchestration: priority-based scheduling (critical views every 15min, others hourly), dependency tracking (refresh parent before child views), automatic staleness detection, incremental refresh for append-only data using timestamp filtering. Built monitoring dashboard showing refresh times, view sizes, query performance improvements. Reduced aggregate query time from 45 minutes to 2 seconds across analytics platform."

</details>