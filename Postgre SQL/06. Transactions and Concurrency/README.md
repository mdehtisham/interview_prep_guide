## **Transactions and Concurrency**

55. What is a transaction in PostgreSQL?

<details>
<summary><strong>Answer</strong></summary>

### What is a Transaction?

A **transaction** is a sequence of one or more SQL operations that are executed as a single, atomic unit of work. It follows the **ACID** properties to ensure data integrity and consistency.

**Key Concept:** Transactions ensure that either **all operations succeed** or **none of them do** (all-or-nothing principle).

---

### ACID Properties

| Property | Description | Example |
|----------|-------------|---------|
| **Atomicity** | All operations succeed or all fail (no partial completion) | Transfer money: debit and credit must both succeed |
| **Consistency** | Database moves from one valid state to another | Account balances never go negative |
| **Isolation** | Concurrent transactions don't interfere with each other | Two users updating same record see consistent data |
| **Durability** | Committed changes persist even after system failure | After COMMIT, data survives crashes/restarts |

---

### Transaction Basics

#### 1. **Starting a Transaction**

```sql
-- Explicit transaction
BEGIN;
-- or
START TRANSACTION;

-- SQL statements here...

COMMIT;  -- Save changes
-- or
ROLLBACK;  -- Discard changes
```

#### 2. **Autocommit Mode (Default)**

```sql
-- In PostgreSQL, each statement is auto-wrapped in a transaction
UPDATE accounts SET balance = 1000 WHERE id = 1;
-- Automatically committed after execution

-- Equivalent to:
BEGIN;
UPDATE accounts SET balance = 1000 WHERE id = 1;
COMMIT;
```

#### 3. **Explicit Transaction Example**

```sql
BEGIN;

-- Debit $100 from account 1
UPDATE accounts 
SET balance = balance - 100 
WHERE id = 1;

-- Credit $100 to account 2
UPDATE accounts 
SET balance = balance + 100 
WHERE id = 2;

-- Both succeed → COMMIT
COMMIT;

-- If any fails → ROLLBACK
-- ROLLBACK;
```

---

### ACID in Action

#### Example 1: **Atomicity** (All or Nothing)

**Scenario:** Transfer $500 from Alice to Bob

```sql
BEGIN;

-- Step 1: Debit Alice's account
UPDATE accounts 
SET balance = balance - 500 
WHERE user_id = 1 AND balance >= 500;

-- Check if debit succeeded
SELECT balance FROM accounts WHERE user_id = 1;
-- balance = 500 (was 1000)

-- Step 2: Credit Bob's account
UPDATE accounts 
SET balance = balance + 500 
WHERE user_id = 2;

-- Something goes wrong here...
-- Power outage, network error, constraint violation, etc.

ROLLBACK;  -- Both changes undone!

-- Result: Alice still has $1000, Bob still has original balance
-- No partial transfer! ✅ Atomicity preserved
```

**Without Transaction (Dangerous!):**
```sql
-- Step 1: Debit Alice (succeeds)
UPDATE accounts SET balance = balance - 500 WHERE user_id = 1;

-- Step 2: Credit Bob (FAILS due to error)
UPDATE accounts SET balance = balance + 500 WHERE user_id = 2;
-- ERROR: some error occurs

-- Result: Alice lost $500, Bob gained nothing! 
-- Money disappeared! ❌ Data corruption
```

#### Example 2: **Consistency** (Valid State)

```sql
-- Constraint: balance cannot be negative
ALTER TABLE accounts ADD CONSTRAINT balance_non_negative 
CHECK (balance >= 0);

BEGIN;

-- Try to withdraw more than available
UPDATE accounts 
SET balance = balance - 1500 
WHERE user_id = 1 AND balance = 1000;

-- ERROR: new row for relation "accounts" violates check constraint
-- Transaction automatically rolled back

ROLLBACK;

-- Database remains in valid state (no negative balances) ✅
```

#### Example 3: **Isolation** (Concurrent Access)

```sql
-- Transaction 1 (User A)
BEGIN;
SELECT balance FROM accounts WHERE user_id = 1;  -- Reads: $1000
UPDATE accounts SET balance = balance - 100 WHERE user_id = 1;
-- Balance now: $900 (not yet committed)

-- Transaction 2 (User B) - runs concurrently
BEGIN;
SELECT balance FROM accounts WHERE user_id = 1;  
-- What does User B see?
-- Depends on isolation level:
--   READ COMMITTED: Still sees $1000 (old value)
--   REPEATABLE READ: Sees $1000 (snapshot)
--   SERIALIZABLE: May see $1000 or block/error

-- Transaction 1 completes
COMMIT;

-- Now User B sees: $900 (after Transaction 1 committed)
```

#### Example 4: **Durability** (Persistence)

```sql
BEGIN;

INSERT INTO orders (customer_id, total, status) 
VALUES (123, 299.99, 'completed');

COMMIT;  -- Transaction committed successfully

-- Immediately after: Server crashes/power outage! 💥
-- System restarts...

-- Query database after restart:
SELECT * FROM orders WHERE customer_id = 123;
-- Order is still there! ✅ Durability guaranteed

-- PostgreSQL uses Write-Ahead Logging (WAL) to ensure durability
```

---

### Transaction Commands

#### 1. **BEGIN / START TRANSACTION**

```sql
-- Start transaction
BEGIN;
-- or
START TRANSACTION;

-- With options
BEGIN ISOLATION LEVEL SERIALIZABLE;
BEGIN READ ONLY;
BEGIN READ WRITE;
BEGIN ISOLATION LEVEL REPEATABLE READ READ ONLY;
```

#### 2. **COMMIT**

```sql
-- Save all changes permanently
COMMIT;
-- or
END;
```

**What happens on COMMIT:**
1. All changes are written to WAL (Write-Ahead Log)
2. Changes become visible to other transactions
3. Locks are released
4. Transaction completes successfully

#### 3. **ROLLBACK**

```sql
-- Discard all changes
ROLLBACK;
-- or
ABORT;
```

**When ROLLBACK happens:**
- Explicit `ROLLBACK` command
- Error occurs in transaction
- Constraint violation
- Deadlock detected
- Connection lost

#### 4. **SAVEPOINT** (Partial Rollback)

```sql
BEGIN;

INSERT INTO orders (id, total) VALUES (1, 100);

SAVEPOINT my_savepoint;

INSERT INTO orders (id, total) VALUES (2, 200);
-- Error occurs here

ROLLBACK TO SAVEPOINT my_savepoint;
-- Only second INSERT is rolled back
-- First INSERT still active in transaction

COMMIT;
-- Only order #1 is saved
```

---

### Real-World Examples

#### Example 1: **E-Commerce Order Processing**

```sql
BEGIN;

-- 1. Create order
INSERT INTO orders (customer_id, total, status, created_at)
VALUES (456, 299.99, 'pending', NOW())
RETURNING id INTO order_id;

-- 2. Add order items
INSERT INTO order_items (order_id, product_id, quantity, price)
VALUES 
    (order_id, 101, 2, 49.99),
    (order_id, 205, 1, 199.99);

-- 3. Update inventory (decrease stock)
UPDATE products 
SET stock = stock - 2 
WHERE id = 101 AND stock >= 2;

UPDATE products 
SET stock = stock - 1 
WHERE id = 205 AND stock >= 1;

-- 4. Check if all updates succeeded
IF (SELECT ROW_COUNT()) = 0 THEN
    ROLLBACK;  -- Insufficient stock
    RAISE EXCEPTION 'Insufficient stock';
END IF;

-- 5. Update customer stats
UPDATE customers 
SET total_orders = total_orders + 1,
    total_spent = total_spent + 299.99
WHERE id = 456;

-- 6. Mark order as completed
UPDATE orders 
SET status = 'completed' 
WHERE id = order_id;

COMMIT;
-- All 6 operations succeed together or all fail ✅
```

**Without Transaction (Dangerous!):**
```sql
-- Order created ✓
INSERT INTO orders ...;

-- Items added ✓
INSERT INTO order_items ...;

-- Stock updated ✓
UPDATE products SET stock = stock - 2 WHERE id = 101;

-- ❌ ERROR: Server crashes here

-- Customer stats NOT updated ✗
-- Order status still 'pending' ✗
-- Result: Inconsistent state! Stock reduced but order not completed
```

#### Example 2: **Bank Transfer**

```sql
CREATE OR REPLACE FUNCTION transfer_money(
    from_account INT,
    to_account INT,
    amount NUMERIC
) RETURNS BOOLEAN AS $$
BEGIN
    -- Start transaction (implicit in function)
    
    -- Validate from_account has sufficient balance
    IF (SELECT balance FROM accounts WHERE id = from_account) < amount THEN
        RAISE EXCEPTION 'Insufficient funds';
    END IF;
    
    -- Debit from_account
    UPDATE accounts 
    SET balance = balance - amount,
        updated_at = NOW()
    WHERE id = from_account;
    
    -- Credit to_account
    UPDATE accounts 
    SET balance = balance + amount,
        updated_at = NOW()
    WHERE id = to_account;
    
    -- Log transaction
    INSERT INTO transaction_log (from_account, to_account, amount, timestamp)
    VALUES (from_account, to_account, amount, NOW());
    
    RETURN TRUE;
    
EXCEPTION
    WHEN OTHERS THEN
        -- Any error → automatic ROLLBACK
        RAISE NOTICE 'Transfer failed: %', SQLERRM;
        RETURN FALSE;
END;
$$ LANGUAGE plpgsql;

-- Usage
SELECT transfer_money(1, 2, 500.00);
-- Either all 3 operations succeed or none do ✅
```

#### Example 3: **User Registration with Profile**

```sql
BEGIN;

-- 1. Create user account
INSERT INTO users (email, password_hash, created_at)
VALUES ('john@example.com', 'hashed_password', NOW())
RETURNING id INTO user_id;

-- 2. Create user profile
INSERT INTO user_profiles (user_id, first_name, last_name, phone)
VALUES (user_id, 'John', 'Doe', '555-1234');

-- 3. Assign default role
INSERT INTO user_roles (user_id, role_id)
VALUES (user_id, 1);  -- role_id 1 = 'customer'

-- 4. Create default settings
INSERT INTO user_settings (user_id, email_notifications, theme)
VALUES (user_id, true, 'light');

-- 5. Send welcome email (external API call - happens after COMMIT)
-- Note: Don't call external APIs inside transactions!

COMMIT;

-- Now safe to send email
-- send_welcome_email('john@example.com');
```

---

### Transaction Boundaries

#### What Should Be in a Transaction?

**✅ Include:**
- Multiple related database operations
- Operations that must succeed/fail together
- Updates requiring consistency checks

**❌ Exclude:**
- External API calls (slow, unreliable)
- File I/O operations
- Email sending
- Long-running computations

**Bad Example:**
```sql
BEGIN;

UPDATE orders SET status = 'shipped' WHERE id = 123;

-- ❌ DON'T DO THIS: External API call inside transaction
-- call_shipping_api(order_123);  -- Takes 3 seconds
-- call_email_service('Order shipped');  -- Takes 2 seconds

COMMIT;
-- Transaction held open for 5+ seconds! Locks held, blocking others
```

**Good Example:**
```sql
BEGIN;
UPDATE orders SET status = 'shipped' WHERE id = 123;
COMMIT;
-- Transaction completed in <10ms, locks released

-- Then call external services
call_shipping_api(order_123);
call_email_service('Order shipped');
```

---

### Transaction Performance

#### 1. **Keep Transactions Short**

```sql
-- ❌ Bad: Long transaction
BEGIN;
UPDATE large_table SET status = 'processed';  -- 10 seconds
-- Holds locks for entire duration

-- ✅ Good: Process in batches
DO $$
BEGIN
    LOOP
        BEGIN
            UPDATE large_table 
            SET status = 'processed' 
            WHERE id IN (
                SELECT id FROM large_table 
                WHERE status = 'pending' 
                LIMIT 1000
            );
            COMMIT;
            
            EXIT WHEN NOT FOUND;
        END;
    END LOOP;
END $$;
-- Each batch commits quickly, locks released frequently
```

#### 2. **Transaction Overhead**

```sql
-- Benchmark: Single transaction vs multiple
-- Test: Insert 10,000 rows

-- Approach 1: One transaction (fast)
BEGIN;
INSERT INTO test_table (value) VALUES (1);
INSERT INTO test_table (value) VALUES (2);
-- ... 10,000 inserts
COMMIT;
-- Time: 1.2 seconds

-- Approach 2: 10,000 transactions (slow)
BEGIN; INSERT INTO test_table (value) VALUES (1); COMMIT;
BEGIN; INSERT INTO test_table (value) VALUES (2); COMMIT;
-- ... 10,000 times
-- Time: 45 seconds

-- Improvement: 37.5x faster with batching!
```

#### 3. **Read-Only Transactions**

```sql
-- Optimize read-only queries
BEGIN TRANSACTION READ ONLY;

SELECT * FROM large_table WHERE ...;
SELECT * FROM another_table WHERE ...;

COMMIT;

-- Benefits:
-- - No transaction ID assigned (faster)
-- - Can read from standby replicas
-- - Query planner optimizations
```

---

### Checking Transaction State

#### 1. **Current Transaction Status**

```sql
-- Check if in transaction
SELECT current_setting('transaction_isolation');

-- Check transaction state
SHOW transaction_read_only;

-- Get transaction ID (only if transaction started)
SELECT txid_current();
```

#### 2. **View Active Transactions**

```sql
SELECT 
    pid,
    usename,
    state,
    query_start,
    state_change,
    query
FROM pg_stat_activity
WHERE state IN ('active', 'idle in transaction')
ORDER BY query_start;
```

**Output:**
```
  pid  | usename | state              | query_start         | query
-------+---------+--------------------+---------------------+------------------
 12345 | webapp  | idle in transaction| 2024-12-26 10:30:15 | UPDATE accounts...
 12346 | webapp  | active             | 2024-12-26 10:31:42 | SELECT * FROM...
```

**Warning:** `idle in transaction` means transaction open but not executing queries. This holds locks and can cause problems!

#### 3. **Terminate Idle Transactions**

```sql
-- Kill transactions idle for > 5 minutes
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE state = 'idle in transaction'
  AND NOW() - state_change > INTERVAL '5 minutes';
```

---

### Common Transaction Pitfalls

#### 1. **Long-Running Transactions**

```sql
-- ❌ Problem
BEGIN;
SELECT * FROM products;  -- User reviews results for 10 minutes
UPDATE products SET price = 99.99 WHERE id = 1;
COMMIT;
-- Holds transaction open for 10+ minutes!

-- ✅ Solution: Close transaction quickly
SELECT * FROM products;  -- No transaction
-- User reviews results
BEGIN;
UPDATE products SET price = 99.99 WHERE id = 1;
COMMIT;
```

#### 2. **Forgetting COMMIT**

```sql
-- ❌ Problem
BEGIN;
UPDATE accounts SET balance = 1000 WHERE id = 1;
-- User closes terminal/connection lost
-- Transaction never committed!

-- ✅ Solution: Use timeout
SET statement_timeout = '30s';
SET idle_in_transaction_session_timeout = '5min';
```

#### 3. **Nested Transactions (Not Supported)**

```sql
-- ❌ PostgreSQL doesn't support nested BEGIN/COMMIT
BEGIN;
    BEGIN;  -- Ignored (warning)
        UPDATE accounts SET balance = 1000;
    COMMIT;  -- Commits entire transaction!
COMMIT;  -- Transaction already committed!

-- ✅ Use SAVEPOINTs instead
BEGIN;
    UPDATE accounts SET balance = 1000;
    
    SAVEPOINT my_savepoint;
    UPDATE accounts SET balance = 2000;
    ROLLBACK TO SAVEPOINT my_savepoint;
    
COMMIT;
```

---

### Best Practices

1. **Keep transactions short** (< 1 second when possible)
2. **Don't call external services** inside transactions
3. **Use explicit BEGIN/COMMIT** for multi-statement operations
4. **Set timeouts** to prevent stuck transactions
5. **Handle errors** with proper ROLLBACK
6. **Batch large operations** (commit every N rows)
7. **Use read-only transactions** for reporting queries
8. **Monitor idle transactions** in production
9. **Test error scenarios** (what if step 3 fails?)
10. **Log transaction start/end** for debugging

---

### Summary Table

| **Aspect** | **Details** |
|------------|-------------|
| **Purpose** | Group multiple operations into atomic unit |
| **Commands** | BEGIN, COMMIT, ROLLBACK, SAVEPOINT |
| **Properties** | ACID (Atomicity, Consistency, Isolation, Durability) |
| **Default** | Autocommit (each statement auto-wrapped) |
| **Isolation Levels** | READ COMMITTED (default), REPEATABLE READ, SERIALIZABLE |
| **Performance** | Batch operations in single transaction (37x faster) |
| **Monitoring** | pg_stat_activity, txid_current() |
| **Common Issues** | Long transactions, idle in transaction, forgetting COMMIT |

---

### Interview Tips

- **Junior**: "Transaction is a group of SQL operations that execute as a single unit following ACID properties. Use BEGIN to start, COMMIT to save changes, ROLLBACK to discard changes. Ensures all operations succeed together or all fail."

- **Mid-Level**: "Transactions provide ACID guarantees: Atomicity (all-or-nothing), Consistency (valid states), Isolation (concurrent safety), Durability (crash-proof). Used transactions for bank transfer ensuring debit and credit both succeed. Kept transactions short (< 1 second) and avoided external API calls inside transactions to prevent lock contention."

- **Senior**: "Implemented transaction strategy for e-commerce order processing: order creation, inventory updates, and customer stats in single transaction ensuring consistency. Used SAVEPOINT for partial rollbacks in complex workflows. Monitored long-running transactions with pg_stat_activity and set idle_in_transaction_session_timeout to 5 minutes. Batched large operations committing every 1000 rows for 37x performance improvement over individual transactions."

- **Advanced**: "Designed distributed transaction system with two-phase commit for cross-database consistency. Implemented retry logic with exponential backoff for serialization failures. Used read-only transactions on replicas for reporting queries reducing primary load by 60%. Built transaction monitoring dashboard tracking duration, lock waits, and idle-in-transaction states. Optimized batch processing from 45 seconds to 1.2 seconds by batching 10k inserts in single transaction."

</details>

56. What are transaction isolation levels?

<details>
<summary><strong>Answer</strong></summary>

### What are Transaction Isolation Levels?

**Transaction isolation levels** define how and when changes made by one transaction become visible to other concurrent transactions. They balance **data consistency** against **concurrency performance**.

**The Trade-off:**
- **Higher isolation** = More consistency, fewer anomalies, but **slower** (more locking)
- **Lower isolation** = Better performance, more concurrency, but **potential anomalies**

---

### Four Standard Isolation Levels

| Level | Dirty Reads | Non-Repeatable Reads | Phantom Reads | Performance |
|-------|-------------|---------------------|---------------|-------------|
| **READ UNCOMMITTED** | ✅ Possible | ✅ Possible | ✅ Possible | Fastest |
| **READ COMMITTED** | ❌ Prevented | ✅ Possible | ✅ Possible | Fast ⭐ Default |
| **REPEATABLE READ** | ❌ Prevented | ❌ Prevented | ✅ Possible\* | Slower |
| **SERIALIZABLE** | ❌ Prevented | ❌ Prevented | ❌ Prevented | Slowest |

\* PostgreSQL prevents phantom reads even in REPEATABLE READ

**PostgreSQL Specifics:**
- Does NOT support READ UNCOMMITTED (uses READ COMMITTED instead)
- REPEATABLE READ uses **snapshot isolation** (prevents phantoms)
- SERIALIZABLE uses **Serializable Snapshot Isolation (SSI)**

---

### Setting Isolation Levels

#### 1. **Per Transaction**

```sql
-- Set for current transaction
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
-- or
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- SQL statements...

COMMIT;
```

#### 2. **Per Session**

```sql
-- Set for entire session
SET SESSION CHARACTERISTICS AS TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- All subsequent transactions use SERIALIZABLE
BEGIN;
-- ...
COMMIT;
```

#### 3. **Check Current Level**

```sql
-- Show current isolation level
SHOW transaction_isolation;

-- Output: read committed (default)
```

#### 4. **Database Default (postgresql.conf)**

```conf
# Set default for all connections
default_transaction_isolation = 'read committed'
```

---

### Understanding Read Anomalies

#### 1. **Dirty Read** (Reading Uncommitted Data)

**Definition:** Transaction A reads data modified by Transaction B before B commits.

**Example:**
```sql
-- Transaction A (Bank Transfer)
BEGIN;
UPDATE accounts SET balance = balance - 500 WHERE id = 1;
-- Balance: $1000 → $500 (NOT committed yet)

-- Transaction B (Auditor)
BEGIN TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;  -- If supported
SELECT balance FROM accounts WHERE id = 1;
-- Sees: $500 (DIRTY READ - uncommitted value)

-- Transaction A fails and rolls back
ROLLBACK;  -- Balance returns to $1000

-- Transaction B's reading was wrong! Account never actually had $500
```

**PostgreSQL Prevention:** 
- PostgreSQL does NOT allow dirty reads
- READ UNCOMMITTED behaves as READ COMMITTED
- Always reads committed data only ✅

#### 2. **Non-Repeatable Read** (Data Changes Between Reads)

**Definition:** Transaction reads same row twice and gets different values because another transaction modified it.

**Example:**
```sql
-- Transaction A (Report Generation)
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;

SELECT balance FROM accounts WHERE id = 1;
-- Reads: $1000

-- ... processing for 5 seconds ...

-- Transaction B (Deposit)
BEGIN;
UPDATE accounts SET balance = balance + 500 WHERE id = 1;
COMMIT;  -- Balance now: $1500

-- Transaction A continues
SELECT balance FROM accounts WHERE id = 1;
-- Reads: $1500 (DIFFERENT VALUE - non-repeatable read)

COMMIT;

-- Report shows account had $1000, then $1500 in same transaction!
-- Inconsistent snapshot ❌
```

**Prevention:** Use REPEATABLE READ or SERIALIZABLE

```sql
-- Transaction A with REPEATABLE READ
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

SELECT balance FROM accounts WHERE id = 1;
-- Reads: $1000

-- Transaction B updates and commits (as before)
-- ...

SELECT balance FROM accounts WHERE id = 1;
-- Still reads: $1000 (CONSISTENT SNAPSHOT ✅)

COMMIT;
```

#### 3. **Phantom Read** (New Rows Appear)

**Definition:** Transaction reads set of rows twice and gets different results because another transaction inserted/deleted rows.

**Example:**
```sql
-- Transaction A (Calculate Average Salary)
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;

SELECT AVG(salary) FROM employees WHERE department = 'Engineering';
-- Result: $75,000 (based on 10 employees)

-- Transaction B (Hire New Employee)
BEGIN;
INSERT INTO employees (name, department, salary) 
VALUES ('Alice', 'Engineering', 150000);
COMMIT;

-- Transaction A continues
SELECT AVG(salary) FROM employees WHERE department = 'Engineering';
-- Result: $81,818 (based on 11 employees - PHANTOM READ)
-- New row "appeared" ❌

COMMIT;
```

**Prevention:** Use REPEATABLE READ (PostgreSQL) or SERIALIZABLE

```sql
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

SELECT AVG(salary) FROM employees WHERE department = 'Engineering';
-- Result: $75,000

-- Transaction B inserts and commits (as before)

SELECT AVG(salary) FROM employees WHERE department = 'Engineering';
-- Still result: $75,000 (No phantom - consistent snapshot ✅)

COMMIT;
```

---

### PostgreSQL Isolation Levels in Detail

#### 1. **READ COMMITTED (Default)**

**How it works:**
- Each **query** sees a snapshot of data as of the start of that query
- New queries see recently committed changes from other transactions
- No dirty reads

**Use case:** Most applications (95%+ of use cases)

**Example:**
```sql
-- Transaction A
BEGIN;  -- Default: READ COMMITTED

SELECT * FROM products WHERE id = 1;
-- Sees: price = $100

-- Wait 5 seconds...

-- Transaction B (in parallel)
BEGIN;
UPDATE products SET price = 150 WHERE id = 1;
COMMIT;  -- Price now $150 in database

-- Transaction A continues (new query)
SELECT * FROM products WHERE id = 1;
-- Sees: price = $150 (COMMITTED change visible)

COMMIT;
```

**Characteristics:**
- ✅ Best performance
- ✅ Highest concurrency
- ✅ Prevents dirty reads
- ❌ Allows non-repeatable reads
- ❌ Allows phantom reads
- ⭐ **Recommended for most OLTP applications**

**When to use:**
- Web applications with independent requests
- APIs handling single operations
- Most business applications
- When you read fresh data for each query

#### 2. **REPEATABLE READ**

**How it works:**
- **Transaction** sees a snapshot of data as of its first query
- All queries in transaction see the same snapshot
- Other transactions' commits invisible until current transaction ends
- Uses **MVCC** (Multi-Version Concurrency Control)

**Example:**
```sql
-- Transaction A
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

SELECT price FROM products WHERE id = 1;
-- Sees: $100 (snapshot taken)

-- Transaction B
BEGIN;
UPDATE products SET price = 150 WHERE id = 1;
COMMIT;

-- Transaction A continues
SELECT price FROM products WHERE id = 1;
-- Still sees: $100 (consistent snapshot)

UPDATE products SET price = 200 WHERE id = 1;
-- ERROR! serialization failure:
-- "could not serialize access due to concurrent update"

ROLLBACK;
```

**Characteristics:**
- ✅ Prevents non-repeatable reads
- ✅ Prevents phantom reads (PostgreSQL only!)
- ✅ Consistent snapshot throughout transaction
- ❌ Can cause serialization errors (need retry logic)
- ❌ Slower than READ COMMITTED

**When to use:**
- Reports requiring consistent data snapshot
- Batch processing with multiple queries
- Financial calculations across multiple tables
- When you need same data throughout transaction

**Serialization Errors:**
```sql
-- Transaction A
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT * FROM accounts WHERE id = 1;  -- balance = $1000

-- Transaction B
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
UPDATE accounts SET balance = balance + 500 WHERE id = 1;
COMMIT;  -- Success

-- Transaction A tries to update
UPDATE accounts SET balance = balance - 200 WHERE id = 1;
-- ERROR: could not serialize access due to concurrent update

ROLLBACK;

-- ✅ Must retry transaction A
```

#### 3. **SERIALIZABLE**

**How it works:**
- **Strongest** isolation level
- Guarantees transactions execute as if they ran serially (one after another)
- Uses **Serializable Snapshot Isolation (SSI)**
- Detects read/write conflicts and aborts transactions

**Example:**
```sql
-- Two transactions reading and updating based on reads

-- Transaction A (Check and Update)
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

SELECT SUM(balance) FROM accounts WHERE user_id = 1;
-- Sees: $5000

-- Transaction B (Concurrent)
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

SELECT SUM(balance) FROM accounts WHERE user_id = 1;
-- Sees: $5000

-- Transaction A continues
INSERT INTO accounts (user_id, balance) VALUES (1, 1000);
COMMIT;  -- Success

-- Transaction B tries to commit
INSERT INTO accounts (user_id, balance) VALUES (1, 1000);
COMMIT;
-- ERROR: could not serialize access due to read/write dependencies
-- PostgreSQL detected conflict and aborted Transaction B

-- ✅ Must retry Transaction B
```

**Characteristics:**
- ✅ Prevents ALL anomalies
- ✅ Safest isolation level
- ✅ Equivalent to serial execution
- ❌ Lowest performance
- ❌ Highest chance of serialization failures
- ❌ Requires robust retry logic

**When to use:**
- Financial systems requiring absolute consistency
- Inventory management (prevent overselling)
- Booking systems (prevent double-booking)
- When business logic depends on multiple reads/writes
- Critical operations where data integrity is paramount

**Real-World Example: Concert Ticket Booking**
```sql
-- Prevent double-booking same seat

-- User A
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Check if seat available
SELECT status FROM seats WHERE id = 123;
-- Sees: 'available'

-- User B (concurrent)
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SELECT status FROM seats WHERE id = 123;
-- Sees: 'available'

-- User A books seat
UPDATE seats SET status = 'booked', user_id = 1 WHERE id = 123;
COMMIT;  -- Success ✅

-- User B tries to book
UPDATE seats SET status = 'booked', user_id = 2 WHERE id = 123;
COMMIT;
-- ERROR: could not serialize ❌
-- Seat already booked by User A

ROLLBACK;
-- User B must be shown "Seat no longer available"
```

---

### Comparison with Real-World Scenarios

#### Scenario 1: **Banking Transfer**

**READ COMMITTED:**
```sql
-- Works fine for simple transfer
BEGIN;  -- READ COMMITTED

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;
-- ✅ Good enough for most cases
```

**REPEATABLE READ:**
```sql
-- Better for complex check + update
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Check balance
SELECT balance FROM accounts WHERE id = 1;  -- $1000

-- Business logic...

-- Update based on read
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- If another transaction modified id=1, this fails with serialization error
COMMIT;
```

**SERIALIZABLE:**
```sql
-- Required for complex business rules
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Check total across multiple accounts
SELECT SUM(balance) FROM accounts WHERE user_id = 1;  -- $5000

-- Business rule: Total must stay above $1000
IF sum < 1100 THEN
    RAISE EXCEPTION 'Insufficient total balance';
END IF;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;

COMMIT;
-- ✅ Ensures no concurrent transaction violates rule
```

#### Scenario 2: **E-Commerce Inventory**

**READ COMMITTED:**
```sql
-- ❌ Race condition possible!
BEGIN;

SELECT stock FROM products WHERE id = 101;  -- stock = 5

-- Another transaction might reduce stock here

UPDATE products SET stock = stock - 3 WHERE id = 101;
-- Might go negative if concurrent updates!

COMMIT;
```

**REPEATABLE READ:**
```sql
-- ✅ Better, but needs retry logic
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

SELECT stock FROM products WHERE id = 101;  -- stock = 5

IF stock >= 3 THEN
    UPDATE products SET stock = stock - 3 WHERE id = 101;
ELSE
    RAISE EXCEPTION 'Insufficient stock';
END IF;

COMMIT;
-- Might fail with serialization error if concurrent update
-- Must retry transaction
```

**SERIALIZABLE:**
```sql
-- ✅ Safest, prevents overselling
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

SELECT stock FROM products WHERE id = 101;  -- stock = 5

UPDATE products SET stock = stock - 3 WHERE id = 101;

COMMIT;
-- PostgreSQL guarantees no overselling even with concurrent transactions
```

---

### Performance Impact

**Benchmark: 1000 Concurrent Transactions**

| Isolation Level | Throughput (TPS) | Errors | Avg Latency |
|-----------------|------------------|--------|-------------|
| READ COMMITTED | 5000 TPS | 0% | 20ms |
| REPEATABLE READ | 3500 TPS | 5% serialization | 35ms |
| SERIALIZABLE | 1200 TPS | 15% serialization | 80ms |

**Key Takeaways:**
- READ COMMITTED: 4x faster than SERIALIZABLE
- SERIALIZABLE: 15% retry rate (needs retry logic)
- Most applications use READ COMMITTED (default)

---

### Best Practices

#### 1. **Use Default (READ COMMITTED) Unless You Need More**

```sql
-- 95% of applications
BEGIN;  -- READ COMMITTED (default)
-- Most OLTP operations work fine
```

#### 2. **Use REPEATABLE READ for Reports/Analytics**

```sql
-- Consistent snapshot for entire report
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

SELECT ... -- Multiple complex queries
SELECT ...
SELECT ...

COMMIT;
```

#### 3. **Use SERIALIZABLE for Critical Operations**

```sql
-- Ticket booking, inventory, financial operations
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Complex business logic with read-then-write
SELECT ...
UPDATE ...

COMMIT;
```

#### 4. **Always Implement Retry Logic for Higher Isolation**

```python
def execute_with_retry(func, max_retries=3):
    for attempt in range(max_retries):
        try:
            return func()
        except SerializationError:
            if attempt == max_retries - 1:
                raise
            time.sleep(0.1 * (2 ** attempt))  # Exponential backoff
```

#### 5. **Monitor Serialization Failures**

```sql
-- Check rollback rate
SELECT 
    datname,
    xact_commit,
    xact_rollback,
    ROUND(100.0 * xact_rollback / NULLIF(xact_commit + xact_rollback, 0), 2) 
        AS rollback_percentage
FROM pg_stat_database;
```

---

### Summary Table

| **Isolation Level** | **Use Case** | **Pros** | **Cons** |
|---------------------|-------------|----------|----------|
| **READ COMMITTED** | Most web apps, APIs | Fast, high concurrency | Non-repeatable reads |
| **REPEATABLE READ** | Reports, batch jobs | Consistent snapshot | Serialization errors |
| **SERIALIZABLE** | Financial, inventory | Absolute consistency | Slow, many retries |

---

### Interview Tips

- **Junior**: "PostgreSQL has three isolation levels: READ COMMITTED (default, sees committed changes), REPEATABLE READ (consistent snapshot), and SERIALIZABLE (strongest, prevents all anomalies). Higher isolation = more consistency but slower performance."

- **Mid-Level**: "Used READ COMMITTED for most OLTP operations providing good balance. Implemented REPEATABLE READ for financial reports requiring consistent data snapshot across multiple queries. Handled serialization errors with exponential backoff retry logic."

- **Senior**: "Selected isolation levels based on requirements: READ COMMITTED for 95% of operations, REPEATABLE READ for analytics with 30+ minute queries requiring consistent snapshot, SERIALIZABLE for inventory management preventing overselling. Monitored rollback rates showing 5% serialization failures in REPEATABLE READ, implemented automatic retry with 3 attempts reducing user-facing errors from 5% to 0.1%."

- **Advanced**: "Designed tiered isolation strategy: READ COMMITTED for real-time APIs (5000 TPS), REPEATABLE READ for reporting (consistent dashboards), SERIALIZABLE for critical paths (ticket booking, financial transfers). Built retry framework with adaptive backoff based on contention metrics. Reduced SERIALIZABLE rollback rate from 15% to 4% by reordering operations and using FOR UPDATE SKIP LOCKED for queue processing. Improved throughput by 3x while maintaining consistency guarantees."

</details>

57. What is the difference between READ COMMITTED and REPEATABLE READ?

<details>
<summary><strong>Answer</strong></summary>

### Key Differences Overview

| Aspect | READ COMMITTED | REPEATABLE READ |
|--------|----------------|-----------------|
| **Snapshot** | Per query | Per transaction |
| **Visibility** | Sees recent commits between queries | Sees snapshot from first query |
| **Non-repeatable reads** | Possible | Prevented |
| **Phantom reads** | Possible | Prevented (PostgreSQL) |
| **Performance** | Faster | Slightly slower |
| **Concurrency** | Higher | Lower |
| **Serialization errors** | Rare | More common |
| **Default** | ✅ Yes | No |
| **Use case** | Most OLTP apps | Reports, analytics |

---

### READ COMMITTED (Default)

**How it works:**
- Each **individual query** sees a snapshot as of its start
- New queries see commits from other transactions
- Fresh data for each SQL statement

**Visual Example:**
```
Timeline:
T1: Transaction A: BEGIN
T2: Transaction A: SELECT balance (sees $1000)
T3: Transaction B: UPDATE balance to $1500, COMMIT
T4: Transaction A: SELECT balance (sees $1500) ← NEW value visible
T5: Transaction A: COMMIT
```

**Code Example:**
```sql
-- Transaction A (READ COMMITTED - Default)
BEGIN;

-- Query 1: Read product price
SELECT price FROM products WHERE id = 101;
-- Result: $100

-- ... 5 seconds pass ...

-- Transaction B (concurrent)
BEGIN;
UPDATE products SET price = 150 WHERE id = 101;
COMMIT;  -- Price now $150

-- Transaction A: Query 2 (new query, new snapshot)
SELECT price FROM products WHERE id = 101;
-- Result: $150 (sees committed change!)

-- Query 3: Calculate discount
SELECT price * 0.9 FROM products WHERE id = 101;
-- Result: $135 (based on $150)

COMMIT;

-- ⚠️ Different queries saw different prices!
-- Query 1: $100
-- Query 2: $150
-- Query 3: $135
```

**Characteristics:**
- ✅ **Best performance** - minimal locking overhead
- ✅ **Always sees latest committed data**
- ✅ **High concurrency** - fewer conflicts
- ✅ **Simple to reason about** - each query is fresh
- ❌ **Non-repeatable reads** - same query can return different results
- ❌ **Phantom reads** - rows can appear/disappear

**Real-World Scenario: Banking Dashboard**
```sql
BEGIN;  -- READ COMMITTED

-- Show account summary at 10:00:01 AM
SELECT account_id, balance FROM accounts WHERE user_id = 123;
-- Account 1: $1,000
-- Account 2: $500
-- Total: $1,500

-- User deposits $500 to Account 1 (10:00:05 AM)
-- Another transaction: UPDATE accounts SET balance = 1500 WHERE account_id = 1; COMMIT;

-- Refresh page at 10:00:10 AM
SELECT account_id, balance FROM accounts WHERE user_id = 123;
-- Account 1: $1,500 ← Updated!
-- Account 2: $500
-- Total: $2,000 ← Different total!

COMMIT;

-- ✅ Good: Always shows latest data
-- ❌ Issue: Same report shows different totals (non-repeatable)
```

---

### REPEATABLE READ

**How it works:**
- **Entire transaction** sees a snapshot as of first query
- All queries see the same consistent data
- Commits from other transactions invisible

**Visual Example:**
```
Timeline:
T1: Transaction A: BEGIN (REPEATABLE READ)
T2: Transaction A: SELECT balance (sees $1000, snapshot frozen)
T3: Transaction B: UPDATE balance to $1500, COMMIT
T4: Transaction A: SELECT balance (still sees $1000) ← OLD value
T5: Transaction A: COMMIT
```

**Code Example:**
```sql
-- Transaction A (REPEATABLE READ)
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Query 1: Read product price (snapshot starts here)
SELECT price FROM products WHERE id = 101;
-- Result: $100

-- ... 5 seconds pass ...

-- Transaction B (concurrent)
BEGIN;
UPDATE products SET price = 150 WHERE id = 101;
COMMIT;  -- Price now $150 in database

-- Transaction A: Query 2 (uses same snapshot)
SELECT price FROM products WHERE id = 101;
-- Result: $100 (still sees old value!)

-- Query 3: Calculate discount
SELECT price * 0.9 FROM products WHERE id = 101;
-- Result: $90 (based on $100)

COMMIT;

-- ✅ All queries saw consistent $100 price
-- Even though database had $150 after Transaction B committed
```

**Characteristics:**
- ✅ **Consistent snapshot** - all queries see same data
- ✅ **No non-repeatable reads** - same query always returns same result
- ✅ **No phantom reads** (PostgreSQL) - row count stays same
- ✅ **Predictable** - transaction sees frozen point in time
- ❌ **Serialization errors** - UPDATE may fail if row changed
- ❌ **Slightly slower** - more snapshot management
- ❌ **Requires retry logic** - handle serialization failures

**Real-World Scenario: Financial Report**
```sql
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Generate monthly report at 10:00 AM
SELECT account_id, balance FROM accounts WHERE user_id = 123;
-- Account 1: $1,000
-- Account 2: $500
-- Total: $1,500

-- User deposits $500 to Account 1 (10:05 AM)
-- Another transaction: UPDATE accounts SET balance = 1500 WHERE account_id = 1; COMMIT;

-- Continue report calculations at 10:10 AM
SELECT account_id, balance FROM accounts WHERE user_id = 123;
-- Account 1: $1,000 ← Still old value!
-- Account 2: $500
-- Total: $1,500 ← Same total! Consistent!

-- Calculate averages, aggregates...
SELECT AVG(balance) FROM accounts WHERE user_id = 123;
-- Average: $750 (consistent with snapshot)

COMMIT;

-- ✅ Good: Entire report uses consistent data from 10:00 AM
-- ✅ No contradictions in report
```

---

### Side-by-Side Comparison

#### Scenario 1: Reading Same Row Multiple Times

**READ COMMITTED:**
```sql
BEGIN;  -- READ COMMITTED (default)

-- Time 1: Read order status
SELECT status FROM orders WHERE id = 123;
-- Result: 'pending'

-- (Another transaction updates: status → 'shipped')

-- Time 2: Read again
SELECT status FROM orders WHERE id = 123;
-- Result: 'shipped' (CHANGED!)

-- Time 3: Check payment
SELECT payment_status FROM orders WHERE id = 123;
-- Might see yet another state if updated again

COMMIT;

-- Different queries saw different states ❌
```

**REPEATABLE READ:**
```sql
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Time 1: Read order status
SELECT status FROM orders WHERE id = 123;
-- Result: 'pending'

-- (Another transaction updates: status → 'shipped')

-- Time 2: Read again
SELECT status FROM orders WHERE id = 123;
-- Result: 'pending' (SAME!)

-- Time 3: Check payment
SELECT payment_status FROM orders WHERE id = 123;
-- Sees same consistent snapshot

COMMIT;

-- All queries saw consistent state ✅
```

#### Scenario 2: Aggregation Queries

**READ COMMITTED:**
```sql
BEGIN;

-- Calculate total sales
SELECT SUM(total) FROM orders WHERE status = 'completed';
-- Result: $10,000 (100 orders)

-- (Another transaction adds new order: $150)

-- Recalculate to verify
SELECT SUM(total) FROM orders WHERE status = 'completed';
-- Result: $10,150 (101 orders) ← DIFFERENT!

-- Calculate average
SELECT AVG(total) FROM orders WHERE status = 'completed';
-- Average based on 101 orders (inconsistent with first query)

COMMIT;

-- Report has inconsistent numbers! ❌
```

**REPEATABLE READ:**
```sql
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Calculate total sales
SELECT SUM(total) FROM orders WHERE status = 'completed';
-- Result: $10,000 (100 orders)

-- (Another transaction adds new order: $150)

-- Recalculate to verify
SELECT SUM(total) FROM orders WHERE status = 'completed';
-- Result: $10,000 (100 orders) ← SAME!

-- Calculate average
SELECT AVG(total) FROM orders WHERE status = 'completed';
-- Average: $100 (consistent: $10,000 / 100)

COMMIT;

-- All calculations consistent! ✅
```

#### Scenario 3: UPDATE Based on Read

**READ COMMITTED:**
```sql
BEGIN;

-- Read current stock
SELECT stock FROM products WHERE id = 101;
-- Result: 50

-- (Another transaction reduces: stock → 5)

-- Update based on read (assuming 50)
UPDATE products SET stock = stock - 10 WHERE id = 101;
-- Succeeds! stock becomes -5 ❌ NEGATIVE STOCK!

COMMIT;

-- No error, but business logic violated!
```

**REPEATABLE READ:**
```sql
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Read current stock
SELECT stock FROM products WHERE id = 101;
-- Result: 50

-- (Another transaction reduces: stock → 5, COMMITS)

-- Update based on read
UPDATE products SET stock = stock - 10 WHERE id = 101;
-- ERROR: could not serialize access due to concurrent update

ROLLBACK;

-- ✅ Transaction fails, preventing negative stock!
-- Must retry with fresh data
```

---

### When to Use Each

#### Use READ COMMITTED When:

**✅ Most Web Applications**
```sql
-- Handle single user request
BEGIN;

-- User updates profile
UPDATE users SET name = 'John Doe' WHERE id = 123;

-- Update timestamp
UPDATE users SET updated_at = NOW() WHERE id = 123;

COMMIT;

-- Simple, independent operations - READ COMMITTED perfect
```

**✅ CRUD Operations**
```sql
-- Create order
BEGIN;
INSERT INTO orders (customer_id, total) VALUES (456, 99.99);
COMMIT;

-- Each operation independent - no need for snapshot
```

**✅ High-Concurrency APIs**
```sql
-- Thousands of concurrent users
-- Each operation standalone
-- Need maximum throughput
-- READ COMMITTED gives best performance
```

**✅ Real-Time Data Display**
```sql
-- Dashboard showing live metrics
BEGIN;

SELECT COUNT(*) FROM active_sessions;
SELECT AVG(response_time) FROM requests WHERE timestamp > NOW() - INTERVAL '5 minutes';

COMMIT;

-- Want to see latest data - READ COMMITTED appropriate
```

#### Use REPEATABLE READ When:

**✅ Reports and Analytics**
```sql
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Monthly sales report (runs for 30 seconds)
SELECT 
    DATE_TRUNC('day', created_at) AS date,
    SUM(total) AS daily_sales,
    COUNT(*) AS order_count
FROM orders
WHERE created_at >= '2024-12-01'
GROUP BY date;

-- Calculate totals
SELECT SUM(total), AVG(total) FROM orders WHERE created_at >= '2024-12-01';

-- All queries use consistent snapshot ✅
COMMIT;
```

**✅ Batch Processing**
```sql
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Process all pending orders
SELECT id FROM orders WHERE status = 'pending';

-- For each order...
UPDATE orders SET status = 'processing' WHERE id = ...;
UPDATE inventory SET stock = stock - ... WHERE product_id = ...;

-- All operations see consistent pending orders list
COMMIT;
```

**✅ Multi-Step Business Logic**
```sql
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Step 1: Check user tier
SELECT tier FROM customers WHERE id = 123;  -- 'gold'

-- Step 2: Calculate discount based on tier
-- (5 seconds of business logic)

-- Step 3: Apply discount
UPDATE orders SET discount = 0.20 WHERE customer_id = 123;

-- Even if user tier changed, we use consistent snapshot ✅
COMMIT;
```

**✅ Financial Calculations**
```sql
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Calculate portfolio value (multiple queries)
SELECT SUM(shares * current_price) FROM holdings WHERE user_id = 123;
SELECT cash_balance FROM accounts WHERE user_id = 123;

-- Generate tax report
SELECT SUM(gain) FROM transactions WHERE user_id = 123 AND year = 2024;

-- All queries see consistent data at same point in time ✅
COMMIT;
```

---

### Handling Serialization Errors (REPEATABLE READ)

**The Problem:**
```sql
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

SELECT stock FROM products WHERE id = 101;
-- Sees: 50

-- Another transaction updates stock to 45 and commits

UPDATE products SET stock = stock - 10 WHERE id = 101;
-- ERROR: could not serialize access due to concurrent update

ROLLBACK;
```

**Solution: Retry Logic**

**Python Example:**
```python
import psycopg2
from psycopg2 import extensions

def process_order_with_retry(order_id, max_retries=3):
    for attempt in range(max_retries):
        try:
            conn = get_connection()
            conn.set_isolation_level(extensions.ISOLATION_LEVEL_REPEATABLE_READ)
            
            cursor = conn.cursor()
            
            # Read and update logic
            cursor.execute("SELECT stock FROM products WHERE id = %s", (product_id,))
            stock = cursor.fetchone()[0]
            
            if stock < quantity:
                raise InsufficientStock()
            
            cursor.execute(
                "UPDATE products SET stock = stock - %s WHERE id = %s",
                (quantity, product_id)
            )
            
            conn.commit()
            return True  # Success!
            
        except psycopg2.extensions.TransactionRollbackError as e:
            if "could not serialize" in str(e):
                conn.rollback()
                if attempt < max_retries - 1:
                    time.sleep(0.1 * (2 ** attempt))  # Exponential backoff
                    continue
                else:
                    raise Exception("Max retries exceeded")
            else:
                raise
        finally:
            conn.close()
    
    return False
```

**Node.js Example:**
```javascript
async function processOrderWithRetry(orderId, maxRetries = 3) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    const client = await pool.connect();
    
    try {
      await client.query('BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ');
      
      // Read and update logic
      const result = await client.query(
        'SELECT stock FROM products WHERE id = $1',
        [productId]
      );
      
      if (result.rows[0].stock < quantity) {
        throw new Error('Insufficient stock');
      }
      
      await client.query(
        'UPDATE products SET stock = stock - $1 WHERE id = $2',
        [quantity, productId]
      );
      
      await client.query('COMMIT');
      return true;  // Success!
      
    } catch (error) {
      await client.query('ROLLBACK');
      
      if (error.code === '40001' && attempt < maxRetries - 1) {
        // Serialization failure - retry with exponential backoff
        await new Promise(resolve => 
          setTimeout(resolve, 100 * Math.pow(2, attempt))
        );
        continue;
      }
      
      throw error;
    } finally {
      client.release();
    }
  }
}
```

---

### Performance Comparison

**Benchmark: 1000 Concurrent Transactions**

```sql
-- Test setup
CREATE TABLE test_accounts (
    id SERIAL PRIMARY KEY,
    balance NUMERIC(10,2)
);

INSERT INTO test_accounts (balance)
SELECT 1000 FROM generate_series(1, 10000);
```

**READ COMMITTED Results:**
```
Throughput: 5,200 transactions/second
Success rate: 100%
Average latency: 19ms
P99 latency: 45ms
Serialization errors: 0
```

**REPEATABLE READ Results:**
```
Throughput: 3,800 transactions/second (27% slower)
Success rate: 95% (5% serialization errors)
Average latency: 26ms (37% higher)
P99 latency: 180ms (4x higher)
Serialization errors: 50 (5%)
```

**With Retry Logic (REPEATABLE READ):**
```
Effective throughput: 3,600 transactions/second
User-facing success rate: 99.9%
Average latency: 28ms
P99 latency: 200ms
```

---

### Real-World Example: E-Commerce Platform

**Problem:**
```
Need to generate daily sales reports at 11 PM
Reports take 5 minutes to run
Thousands of new orders during report generation
Reports showing inconsistent numbers (non-repeatable reads)
```

**Original Code (READ COMMITTED):**
```sql
BEGIN;  -- READ COMMITTED

-- Query 1: Count orders (11:00:00 PM)
SELECT COUNT(*) FROM orders WHERE DATE(created_at) = CURRENT_DATE;
-- Result: 5,000 orders

-- Query 2: Calculate revenue (11:02:30 PM)
SELECT SUM(total) FROM orders WHERE DATE(created_at) = CURRENT_DATE;
-- Result: $125,000 (based on 5,250 orders - 250 new orders added!)

-- Query 3: Calculate average (11:04:00 PM)
SELECT AVG(total) FROM orders WHERE DATE(created_at) = CURRENT_DATE;
-- Result: $23.50 (based on 5,500 orders - 250 more added!)

COMMIT;

-- Report shows:
-- Orders: 5,000
-- Revenue: $125,000 (but should be $120,000 for 5,000 orders @ $24 avg)
-- Average: $23.50 (inconsistent - implies $129,250 revenue for 5,500 orders)
-- ❌ Numbers don't add up!
```

**Fixed Code (REPEATABLE READ):**
```sql
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Snapshot frozen at 11:00:00 PM

-- Query 1: Count orders
SELECT COUNT(*) FROM orders WHERE DATE(created_at) = CURRENT_DATE;
-- Result: 5,000 orders

-- Query 2: Calculate revenue (2.5 minutes later)
SELECT SUM(total) FROM orders WHERE DATE(created_at) = CURRENT_DATE;
-- Result: $120,000 (same 5,000 orders)

-- Query 3: Calculate average (4 minutes later)
SELECT AVG(total) FROM orders WHERE DATE(created_at) = CURRENT_DATE;
-- Result: $24.00 (same 5,000 orders)

COMMIT;

-- Report shows:
-- Orders: 5,000
-- Revenue: $120,000
-- Average: $24.00
-- ✅ All numbers consistent! (5,000 × $24 = $120,000)
```

**Results:**
```
Before (READ COMMITTED):
- Report generation time: 5 minutes
- Accuracy issues: 15% of reports had inconsistent numbers
- Customer complaints: High (CFO questioning data quality)

After (REPEATABLE READ):
- Report generation time: 5 minutes (same)
- Accuracy issues: 0% (all reports consistent)
- Customer satisfaction: 100%
- Additional overhead: Negligible (<2%)
```

---

### Summary

**Quick Decision Guide:**

```
Is this a report or analytics query running multiple SELECT statements?
├─ YES → Use REPEATABLE READ
└─ NO
    ├─ Does business logic depend on multiple reads being consistent?
    │   ├─ YES → Use REPEATABLE READ (implement retry logic)
    │   └─ NO → Use READ COMMITTED (default)
    └─ Is this a simple CRUD operation (single read/write)?
        └─ YES → Use READ COMMITTED (default)
```

**Key Takeaways:**
1. **READ COMMITTED** = Fast, fresh data, possible inconsistencies
2. **REPEATABLE READ** = Slower, consistent snapshot, needs retry logic
3. **Default to READ COMMITTED** for 95% of use cases
4. **Use REPEATABLE READ** for reports and multi-query business logic
5. **Always implement retry logic** with REPEATABLE READ

---

### Interview Tips

- **Junior**: "READ COMMITTED is default, each query sees latest commits. REPEATABLE READ sees consistent snapshot for entire transaction. READ COMMITTED is faster but may have non-repeatable reads."

- **Mid-Level**: "Used READ COMMITTED for OLTP operations (5,200 TPS) and REPEATABLE READ for reports (3,800 TPS). REPEATABLE READ prevents non-repeatable reads by maintaining consistent snapshot throughout transaction. Implemented retry logic for 5% serialization errors using exponential backoff."

- **Senior**: "Selected READ COMMITTED for web APIs handling independent requests (99% of operations), REPEATABLE READ for financial reports requiring data consistency across multiple queries. Reduced report inconsistencies from 15% to 0% by switching to REPEATABLE READ. Implemented automatic retry framework handling serialization errors, achieving 99.9% effective success rate. Report accuracy improved dramatically with negligible 2% performance overhead."

- **Advanced**: "Architected isolation level strategy: READ COMMITTED for real-time APIs (5,200 TPS), REPEATABLE READ for analytics/reporting (3,800 TPS with retry logic). Built smart retry system detecting serialization failure patterns and adjusting backoff dynamically. Reduced retry attempts by 40% through operation reordering and early locking with FOR UPDATE. Created monitoring showing isolation level distribution, serialization error rates, and retry success rates. Maintained 99.95% user-facing success rate while ensuring data consistency for 1M+ daily reports."

</details>

58. What is SERIALIZABLE isolation level?

<details>
<summary><strong>Answer</strong></summary>

### What is SERIALIZABLE?

**SERIALIZABLE** is the **strongest** transaction isolation level in PostgreSQL. It guarantees that concurrent transactions execute as if they ran **serially** (one after another), even though they actually run in parallel.

**Key Concept:** If transactions would produce different results when run concurrently versus serially, PostgreSQL detects this and aborts one of them with a serialization error.

---

### How SERIALIZABLE Works

**PostgreSQL Implementation: Serializable Snapshot Isolation (SSI)**

PostgreSQL uses an advanced technique called **SSI** that:
1. Tracks read and write dependencies between transactions
2. Detects **read-write conflicts** that could lead to anomalies
3. Aborts transactions when conflicts detected
4. More efficient than traditional lock-based serialization

**Visual Example:**
```
Serial Execution (One-at-a-Time):
T1: ──────────────────────────────────────── (completes)
T2:                                         ──────────────────────── (starts after T1)

Concurrent Execution with SERIALIZABLE:
T1: ──────────────────────────────────────── (completes)
T2:     ──────────────────────────────────── (runs concurrently)
            ↑
            If T2 would produce different result than serial execution,
            PostgreSQL aborts T2 with serialization error
```

---

### SERIALIZABLE vs Other Levels

| Isolation Level | Prevents All Anomalies? | Performance | Retry Needed? |
|----------------|------------------------|-------------|---------------|
| READ COMMITTED | ❌ No | Fast (5000 TPS) | Rarely |
| REPEATABLE READ | ⚠️ Most | Medium (3800 TPS) | Sometimes (5%) |
| SERIALIZABLE | ✅ Yes | Slow (1200 TPS) | Often (15-25%) |

---

### Setting SERIALIZABLE

```sql
-- Per transaction
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
-- SQL statements
COMMIT;

-- Per session
SET SESSION CHARACTERISTICS AS TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Check current level
SHOW transaction_isolation;
```

---

### Anomalies Prevented by SERIALIZABLE

#### 1. **Write Skew Anomaly**

**Problem:** Two transactions read overlapping data and make decisions based on those reads, but their combined effect violates business rules.

**Example: Doctor On-Call Schedule**

**Business Rule:** At least one doctor must be on-call at all times

```sql
-- Initial state: 2 doctors on-call
doctors table:
id | name    | on_call
---+---------+--------
1  | Dr. A   | true
2  | Dr. B   | true
```

**With REPEATABLE READ (FAILS):**
```sql
-- Transaction 1 (Dr. A going off-call)
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Check if another doctor is on-call
SELECT COUNT(*) FROM doctors WHERE on_call = true AND id != 1;
-- Result: 1 (Dr. B is on-call, so safe to go off)

-- Transaction 2 (Dr. B going off-call) - runs concurrently
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Check if another doctor is on-call
SELECT COUNT(*) FROM doctors WHERE on_call = true AND id != 2;
-- Result: 1 (Dr. A is on-call, so safe to go off)

-- Transaction 1 commits
UPDATE doctors SET on_call = false WHERE id = 1;
COMMIT;  -- Success!

-- Transaction 2 commits
UPDATE doctors SET on_call = false WHERE id = 2;
COMMIT;  -- Success!

-- Final state: NO doctors on-call! ❌
-- Business rule violated!
```

**With SERIALIZABLE (SAFE):**
```sql
-- Transaction 1 (Dr. A going off-call)
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

SELECT COUNT(*) FROM doctors WHERE on_call = true AND id != 1;
-- Result: 1

UPDATE doctors SET on_call = false WHERE id = 1;
COMMIT;  -- Success!

-- Transaction 2 (Dr. B going off-call)
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

SELECT COUNT(*) FROM doctors WHERE on_call = true AND id != 2;
-- Result: 1

UPDATE doctors SET on_call = false WHERE id = 2;
COMMIT;
-- ERROR: could not serialize access due to read/write dependencies
-- ✅ Transaction aborted! Business rule protected!

ROLLBACK;
-- Must retry, will see Dr. A is off-call, won't allow Dr. B to go off
```

#### 2. **Read-Only Transaction Anomaly**

**Problem:** Read-only transaction sees inconsistent state that never existed serially.

**Example: Bank Account Constraint**

**Business Rule:** Sum of all accounts must equal total deposits

```sql
-- Initial state
accounts table:
account_id | balance
-----------+--------
1          | 1000
2          | 500
Total: 1500

deposits_log:
total_deposits = 1500
```

**With REPEATABLE READ (INCONSISTENT):**
```sql
-- Transaction 1 (Transfer $200: Account 1 → Account 2)
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

UPDATE accounts SET balance = balance - 200 WHERE account_id = 1;
-- Account 1: 800

-- Transaction 2 (Audit - read-only)
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

SELECT SUM(balance) FROM accounts;
-- Reads: Account 1 = 800 (sees T1's write, even though not committed)
--        Account 2 = 500 (doesn't see T1's write yet)
-- Total: 1300 ❌

SELECT total_deposits FROM deposits_log;
-- Total: 1500

-- Discrepancy: 1300 vs 1500! (200 missing)
COMMIT;

-- Transaction 1 completes
UPDATE accounts SET balance = balance + 200 WHERE account_id = 2;
COMMIT;

-- Audit saw inconsistent state that never existed!
```

**With SERIALIZABLE (CONSISTENT):**
```sql
-- Transaction 1 (Transfer)
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

UPDATE accounts SET balance = balance - 200 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 200 WHERE account_id = 2;
COMMIT;

-- Transaction 2 (Audit)
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

SELECT SUM(balance) FROM accounts;  -- 1500 ✅
SELECT total_deposits FROM deposits_log;  -- 1500 ✅

COMMIT;

-- Always sees consistent state!
-- Either sees transfer before it started, or after it completed
```

#### 3. **Serialization Anomaly (Complex)**

**Example: Inventory Management**

**Business Rule:** Total items sold cannot exceed total items purchased

```sql
-- Transaction 1: Check and sell
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

SELECT SUM(quantity) FROM purchases WHERE product_id = 101;
-- Total purchased: 100

SELECT SUM(quantity) FROM sales WHERE product_id = 101;
-- Total sold: 80

-- Available: 20, can sell 15
INSERT INTO sales (product_id, quantity) VALUES (101, 15);

COMMIT;

-- Transaction 2: Check and sell (concurrent)
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

SELECT SUM(quantity) FROM purchases WHERE product_id = 101;
-- Total purchased: 100

SELECT SUM(quantity) FROM sales WHERE product_id = 101;
-- Total sold: 80 (doesn't see T1's sale yet)

-- Available: 20, can sell 15
INSERT INTO sales (product_id, quantity) VALUES (101, 15);

COMMIT;
-- ERROR: could not serialize access
-- ✅ Prevents overselling! (would have sold 110 total from 100 purchased)
```

---

### Real-World Use Cases

#### 1. **Concert Ticket Booking (Prevent Double-Booking)**

```sql
-- Business Rule: Each seat can only be booked once

CREATE TABLE seats (
    id SERIAL PRIMARY KEY,
    concert_id INT,
    seat_number VARCHAR(10),
    status VARCHAR(20),
    booked_by INT,
    UNIQUE(concert_id, seat_number)
);

-- User A tries to book seat A1
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Check if seat available
SELECT status FROM seats 
WHERE concert_id = 123 AND seat_number = 'A1';
-- Result: 'available'

-- User B tries same seat (concurrent)
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

SELECT status FROM seats 
WHERE concert_id = 123 AND seat_number = 'A1';
-- Result: 'available' (snapshot before User A's update)

-- User A books
UPDATE seats 
SET status = 'booked', booked_by = 1001
WHERE concert_id = 123 AND seat_number = 'A1';

COMMIT;  -- Success!

-- User B tries to book
UPDATE seats 
SET status = 'booked', booked_by = 1002
WHERE concert_id = 123 AND seat_number = 'A1';

COMMIT;
-- ERROR: could not serialize access
-- ✅ Double-booking prevented!

ROLLBACK;
-- Show User B: "Seat no longer available"
```

#### 2. **Flash Sale (Limited Quantity)**

```sql
-- Business Rule: Only 100 items available

CREATE TABLE flash_sale_items (
    id SERIAL PRIMARY KEY,
    product_id INT,
    quantity_available INT,
    quantity_sold INT DEFAULT 0
);

INSERT INTO flash_sale_items (product_id, quantity_available)
VALUES (501, 100);

-- 1000 users trying to buy simultaneously

-- User 1
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

SELECT quantity_available, quantity_sold 
FROM flash_sale_items WHERE product_id = 501;
-- Available: 100, Sold: 0

-- Check if can buy
IF (available - sold) >= 1 THEN
    UPDATE flash_sale_items 
    SET quantity_sold = quantity_sold + 1
    WHERE product_id = 501;
    
    INSERT INTO orders (user_id, product_id, quantity)
    VALUES (user_id, 501, 1);
END IF;

COMMIT;

-- Users 2-1000 do same thing concurrently
-- SERIALIZABLE ensures exactly 100 orders are created
-- Remaining 900 users get serialization error → show "sold out"
```

#### 3. **Accounting (Double-Entry Bookkeeping)**

```sql
-- Business Rule: Debits must equal credits

CREATE TABLE journal_entries (
    id SERIAL PRIMARY KEY,
    transaction_id INT,
    account_id INT,
    amount NUMERIC(10,2),
    type VARCHAR(10)  -- 'debit' or 'credit'
);

-- Transfer $500 between accounts
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Debit checking account
INSERT INTO journal_entries (transaction_id, account_id, amount, type)
VALUES (123, 1001, 500, 'debit');

-- Credit savings account
INSERT INTO journal_entries (transaction_id, account_id, amount, type)
VALUES (123, 2001, 500, 'credit');

-- Verify balance
SELECT 
    SUM(CASE WHEN type = 'debit' THEN amount ELSE 0 END) AS total_debit,
    SUM(CASE WHEN type = 'credit' THEN amount ELSE 0 END) AS total_credit
FROM journal_entries
WHERE transaction_id = 123;

-- If debit != credit, rollback
IF total_debit != total_credit THEN
    RAISE EXCEPTION 'Debits must equal credits';
END IF;

COMMIT;

-- SERIALIZABLE ensures concurrent transactions don't create imbalance
```

---

### Handling Serialization Errors

**Error Code:** `40001` (serialization_failure)

**Error Message:**
```
ERROR: could not serialize access due to read/write dependencies among transactions
DETAIL: Reason code: Canceled on identification as a pivot, during commit attempt.
HINT: The transaction might succeed if retried.
```

**Retry Pattern (Python):**
```python
import psycopg2
from psycopg2 import extensions

def execute_serializable_transaction(func, max_retries=5):
    """
    Execute function in SERIALIZABLE transaction with automatic retry
    """
    for attempt in range(max_retries):
        conn = None
        try:
            conn = get_connection()
            conn.set_isolation_level(extensions.ISOLATION_LEVEL_SERIALIZABLE)
            
            cursor = conn.cursor()
            
            # Execute user's transaction logic
            result = func(cursor)
            
            conn.commit()
            return result  # Success!
            
        except psycopg2.extensions.TransactionRollbackError as e:
            if conn:
                conn.rollback()
            
            if attempt < max_retries - 1:
                # Exponential backoff
                wait_time = 0.1 * (2 ** attempt) + random.uniform(0, 0.1)
                time.sleep(wait_time)
                print(f"Serialization error, retry {attempt + 1}/{max_retries}")
                continue
            else:
                raise Exception(f"Failed after {max_retries} retries")
        
        except Exception as e:
            if conn:
                conn.rollback()
            raise
        
        finally:
            if conn:
                conn.close()
    
    raise Exception("Should not reach here")

# Usage
def book_ticket(cursor):
    cursor.execute("""
        SELECT status FROM seats 
        WHERE concert_id = %s AND seat_number = %s
    """, (concert_id, seat_number))
    
    status = cursor.fetchone()[0]
    
    if status != 'available':
        raise Exception('Seat not available')
    
    cursor.execute("""
        UPDATE seats 
        SET status = 'booked', booked_by = %s
        WHERE concert_id = %s AND seat_number = %s
    """, (user_id, concert_id, seat_number))
    
    return True

# Automatically retries on serialization failure
result = execute_serializable_transaction(book_ticket, max_retries=5)
```

**Node.js Example:**
```javascript
async function executeSerializableTransaction(func, maxRetries = 5) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    const client = await pool.connect();
    
    try {
      await client.query('BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE');
      
      const result = await func(client);
      
      await client.query('COMMIT');
      return result;  // Success!
      
    } catch (error) {
      await client.query('ROLLBACK');
      
      if (error.code === '40001' && attempt < maxRetries - 1) {
        // Serialization failure - retry
        const waitTime = 100 * Math.pow(2, attempt) + Math.random() * 100;
        await new Promise(resolve => setTimeout(resolve, waitTime));
        console.log(`Serialization error, retry ${attempt + 1}/${maxRetries}`);
        continue;
      }
      
      throw error;
    } finally {
      client.release();
    }
  }
  
  throw new Error(`Failed after ${maxRetries} retries`);
}

// Usage
await executeSerializableTransaction(async (client) => {
  const result = await client.query(
    'SELECT status FROM seats WHERE concert_id = $1 AND seat_number = $2',
    [concertId, seatNumber]
  );
  
  if (result.rows[0].status !== 'available') {
    throw new Error('Seat not available');
  }
  
  await client.query(
    'UPDATE seats SET status = $1, booked_by = $2 WHERE concert_id = $3 AND seat_number = $4',
    ['booked', userId, concertId, seatNumber]
  );
  
  return true;
});
```

---

### Performance Considerations

#### Benchmark Results

**Test:** 1000 concurrent transactions updating shared resources

| Isolation Level | TPS | Success Rate | Retries Needed | Avg Latency |
|----------------|-----|--------------|----------------|-------------|
| READ COMMITTED | 5000 | 100% | 0% | 20ms |
| REPEATABLE READ | 3800 | 95% | 5% | 35ms |
| **SERIALIZABLE** | **1200** | **75%** | **25%** | **85ms** |
| SERIALIZABLE (with retry) | 1000 | 99.9% | 40% total | 120ms |

**Key Insights:**
- SERIALIZABLE is **4x slower** than READ COMMITTED
- **25% of transactions fail** and need retry (high contention)
- With retry logic, effective success rate: 99.9%
- Total latency including retries: **6x higher** than READ COMMITTED

#### Reducing Serialization Failures

**1. Keep Transactions Short**
```sql
-- ❌ Bad: Long transaction
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

SELECT ... -- 2 seconds
-- Business logic in application
UPDATE ... -- 1 second

COMMIT;
-- High chance of conflict over 3+ seconds

-- ✅ Good: Short transaction
-- Do business logic OUTSIDE transaction
SELECT ... -- (outside transaction)
-- Process data in application

BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
UPDATE ... -- 10ms
COMMIT;
-- Minimal conflict window
```

**2. Order Operations Consistently**
```sql
-- ❌ Bad: Random order
UPDATE accounts SET balance = balance - 100 WHERE id IN (5, 2, 8);

-- ✅ Good: Sorted order
UPDATE accounts SET balance = balance - 100 WHERE id IN (2, 5, 8) ORDER BY id;
```

**3. Use FOR UPDATE to Lock Early**
```sql
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Lock rows immediately
SELECT * FROM accounts WHERE id IN (1, 2) ORDER BY id FOR UPDATE;

-- Now update
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;
```

---

### When to Use SERIALIZABLE

#### ✅ **Use SERIALIZABLE When:**

1. **Financial Transactions**
   - Money transfers
   - Double-entry bookkeeping
   - Currency exchanges

2. **Inventory Management**
   - Prevent overselling
   - Stock allocation
   - Limited quantity flash sales

3. **Booking Systems**
   - Seat reservations
   - Hotel rooms
   - Appointment scheduling

4. **Business Rules Enforcement**
   - At least N on-call doctors
   - Maximum credit limit
   - Quota management

5. **Audit Requirements**
   - Regulatory compliance
   - Financial reporting
   - Data integrity critical

#### ❌ **Don't Use SERIALIZABLE When:**

1. **High-Volume OLTP**
   - Need > 3000 TPS
   - Can't afford retry overhead

2. **Read-Heavy Workloads**
   - Mostly SELECT queries
   - READ COMMITTED sufficient

3. **Independent Operations**
   - No business logic dependencies
   - Single-row updates

4. **Real-Time Systems**
   - Can't tolerate retry delays
   - Need consistent low latency

---

### Monitoring SERIALIZABLE Transactions

```sql
-- Check serialization failure rate
SELECT 
    datname,
    xact_commit,
    xact_rollback,
    ROUND(100.0 * xact_rollback / NULLIF(xact_commit + xact_rollback, 0), 2) 
        AS rollback_pct
FROM pg_stat_database
WHERE datname = current_database();

-- View currently blocked transactions
SELECT 
    blocked_locks.pid AS blocked_pid,
    blocking_locks.pid AS blocking_pid,
    blocked_activity.query AS blocked_query,
    blocking_activity.query AS blocking_query
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks ON blocking_locks.locktype = blocked_locks.locktype
JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted;

-- Check for long-running SERIALIZABLE transactions
SELECT 
    pid,
    now() - xact_start AS duration,
    state,
    query
FROM pg_stat_activity
WHERE state = 'active' 
  AND now() - xact_start > INTERVAL '5 seconds'
ORDER BY duration DESC;
```

---

### Summary

**SERIALIZABLE Characteristics:**
- ✅ **Strongest isolation** - prevents all anomalies
- ✅ **Equivalent to serial execution** - as if transactions ran one-at-a-time
- ✅ **Critical for business rules** - prevents logic violations
- ❌ **Slowest performance** - 4x slower than READ COMMITTED
- ❌ **High retry rate** - 15-25% serialization failures
- ❌ **Requires retry logic** - must handle and retry failures

**When to Use:**
- Financial systems
- Inventory/booking systems
- Enforcing complex business rules
- Regulatory compliance

**When NOT to Use:**
- High-volume OLTP (> 3000 TPS)
- Read-heavy workloads
- Independent operations
- Real-time systems

---

### Interview Tips

- **Junior**: "SERIALIZABLE is the strongest isolation level. It ensures transactions execute as if run serially, preventing all anomalies. Slower than other levels and may cause serialization errors requiring retries."

- **Mid-Level**: "Used SERIALIZABLE for ticket booking system preventing double-booking. Transactions check seat availability then book. SERIALIZABLE detects conflicts and aborts one transaction. Implemented retry logic with exponential backoff handling 25% serialization failure rate. Performance: 1200 TPS vs 5000 TPS for READ COMMITTED."

- **Senior**: "Implemented SERIALIZABLE for flash sale with 100 items and 1000 concurrent users. Prevented overselling by detecting read-write dependencies. Built automatic retry framework with 5 attempts and exponential backoff, achieving 99.9% success rate. Optimized by keeping transactions short (< 50ms), using consistent lock ordering, and FOR UPDATE for early locking. Reduced serialization failures from 25% to 12% through optimization. Monitored rollback rates and transaction duration."

- **Advanced**: "Designed hybrid isolation strategy: SERIALIZABLE for critical paths (booking, financial) representing 5% of transactions, READ COMMITTED for rest. Built intelligent retry system analyzing contention patterns and adaptively adjusting backoff. Implemented transaction profiling identifying operations causing most conflicts, reordered them reducing failures by 40%. Used predicate locking analysis to minimize false positives. Created monitoring dashboard tracking serialization failure rates by transaction type, retry distribution, and conflict patterns. Achieved 99.95% user-facing success rate while maintaining strict consistency for 50K daily critical transactions."

</details>

59. How do you handle concurrent updates in PostgreSQL?

<details>
<summary><strong>Answer</strong></summary>

### Overview of Concurrent Updates

When multiple transactions try to modify the same data simultaneously, PostgreSQL provides several mechanisms to handle these conflicts safely while maintaining data consistency.

**Key Challenge:** Prevent **lost updates** where one transaction's changes overwrite another's without detection.

---

### Common Concurrent Update Problems

#### 1. **Lost Update Problem**

```sql
-- Initial: account balance = $1000

-- Transaction A (User withdraws $100)
BEGIN;
SELECT balance FROM accounts WHERE id = 1;  -- Reads: $1000
-- balance_new = 1000 - 100 = 900

-- Transaction B (User deposits $200) - concurrent
BEGIN;
SELECT balance FROM accounts WHERE id = 1;  -- Reads: $1000
-- balance_new = 1000 + 200 = 1200

-- Transaction A updates
UPDATE accounts SET balance = 900 WHERE id = 1;
COMMIT;

-- Transaction B updates (OVERWRITES Transaction A's change!)
UPDATE accounts SET balance = 1200 WHERE id = 1;
COMMIT;

-- Final balance: $1200
-- Expected: $1100 ($1000 - $100 + $200)
-- Lost Update: $100 withdrawal was lost! ❌
```

---

### Solutions for Concurrent Updates

### Solution 1: **Optimistic Locking (Version Column)**

**How it works:** Add a version column, increment it on each update, and check version hasn't changed.

**Implementation:**
```sql
-- Add version column
ALTER TABLE accounts ADD COLUMN version INTEGER DEFAULT 1;

-- Transaction A (Optimistic locking)
BEGIN;

-- Read current state
SELECT balance, version FROM accounts WHERE id = 1;
-- balance = 1000, version = 1

-- Calculate new balance
new_balance = 1000 - 100  -- 900

-- Update only if version hasn't changed
UPDATE accounts 
SET balance = 900, version = version + 1
WHERE id = 1 AND version = 1;

-- Check if update succeeded
GET DIAGNOSTICS affected_rows = ROW_COUNT;

IF affected_rows = 0 THEN
    ROLLBACK;
    RAISE EXCEPTION 'Concurrent modification detected, please retry';
END IF;

COMMIT;

-- Transaction B (concurrent)
BEGIN;

SELECT balance, version FROM accounts WHERE id = 1;
-- balance = 1000, version = 1

new_balance = 1000 + 200  -- 1200

-- Tries to update
UPDATE accounts 
SET balance = 1200, version = version + 1
WHERE id = 1 AND version = 1;

-- If Transaction A committed first:
-- affected_rows = 0 (version is now 2, not 1)
-- Transaction B detects conflict and retries ✅

ROLLBACK;
-- Must retry with fresh data
```

**Application Code (Python):**
```python
def update_account_optimistic(account_id, amount, max_retries=3):
    """
    Update account with optimistic locking
    """
    for attempt in range(max_retries):
        try:
            conn = get_connection()
            cursor = conn.cursor()
            
            # Read current state
            cursor.execute(
                "SELECT balance, version FROM accounts WHERE id = %s",
                (account_id,)
            )
            balance, version = cursor.fetchone()
            
            # Calculate new balance
            new_balance = balance + amount
            
            if new_balance < 0:
                raise InsufficientFunds()
            
            # Update with version check
            cursor.execute("""
                UPDATE accounts 
                SET balance = %s, version = version + 1
                WHERE id = %s AND version = %s
            """, (new_balance, account_id, version))
            
            if cursor.rowcount == 0:
                # Concurrent modification detected
                conn.rollback()
                if attempt < max_retries - 1:
                    time.sleep(0.1 * (2 ** attempt))  # Exponential backoff
                    continue
                else:
                    raise Exception("Max retries exceeded")
            
            conn.commit()
            return new_balance
            
        except Exception as e:
            conn.rollback()
            raise
        finally:
            conn.close()
    
    raise Exception("Should not reach here")

# Usage
new_balance = update_account_optimistic(account_id=1, amount=-100)
```

**Pros:**
- ✅ No locks held during read
- ✅ Great for low-contention scenarios
- ✅ High concurrency
- ✅ Works with any isolation level

**Cons:**
- ❌ Requires retry logic
- ❌ May have high retry rate under contention
- ❌ Needs application code changes

---

### Solution 2: **Pessimistic Locking (SELECT FOR UPDATE)**

**How it works:** Lock rows immediately when reading, preventing other transactions from modifying them.

**Implementation:**
```sql
-- Transaction A
BEGIN;

-- Lock the row (other transactions must wait)
SELECT balance FROM accounts WHERE id = 1 FOR UPDATE;
-- balance = 1000

-- Calculate new balance
new_balance = 1000 - 100  -- 900

-- Update (no one else can modify this row)
UPDATE accounts SET balance = 900 WHERE id = 1;

COMMIT;  -- Releases lock

-- Transaction B (concurrent) - BLOCKS until Transaction A commits
BEGIN;

-- Waits here until Transaction A releases lock
SELECT balance FROM accounts WHERE id = 1 FOR UPDATE;
-- Now reads: 900 (after Transaction A committed)

new_balance = 900 + 200  -- 1100

UPDATE accounts SET balance = 1100 WHERE id = 1;

COMMIT;

-- Final balance: $1100 ✅ Correct!
-- No lost updates!
```

**Lock Modes:**
```sql
-- FOR UPDATE: Exclusive lock (blocks other FOR UPDATE, UPDATE, DELETE)
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;

-- FOR NO KEY UPDATE: Like FOR UPDATE but allows foreign key checks
SELECT * FROM accounts WHERE id = 1 FOR NO KEY UPDATE;

-- FOR SHARE: Shared lock (blocks UPDATE/DELETE but allows other FOR SHARE)
SELECT * FROM accounts WHERE id = 1 FOR SHARE;

-- FOR KEY SHARE: Weakest lock (blocks DELETE but allows UPDATE)
SELECT * FROM accounts WHERE id = 1 FOR KEY SHARE;
```

**NOWAIT and SKIP LOCKED:**
```sql
-- NOWAIT: Error immediately if row locked
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE NOWAIT;
-- ERROR: could not obtain lock on row in relation "accounts"
ROLLBACK;

-- SKIP LOCKED: Skip locked rows, process available ones
BEGIN;
SELECT * FROM jobs 
WHERE status = 'pending' 
ORDER BY id 
LIMIT 10
FOR UPDATE SKIP LOCKED;
-- Returns only unlocked rows (great for job queues!)
```

**Application Code (Python):**
```python
def update_account_pessimistic(account_id, amount):
    """
    Update account with pessimistic locking
    """
    conn = get_connection()
    cursor = conn.cursor()
    
    try:
        cursor.execute("BEGIN")
        
        # Lock row immediately
        cursor.execute(
            "SELECT balance FROM accounts WHERE id = %s FOR UPDATE",
            (account_id,)
        )
        balance = cursor.fetchone()[0]
        
        # Calculate new balance
        new_balance = balance + amount
        
        if new_balance < 0:
            raise InsufficientFunds()
        
        # Update (row is locked, safe)
        cursor.execute(
            "UPDATE accounts SET balance = %s WHERE id = %s",
            (new_balance, account_id)
        )
        
        conn.commit()
        return new_balance
        
    except Exception as e:
        conn.rollback()
        raise
    finally:
        conn.close()

# Usage
new_balance = update_account_pessimistic(account_id=1, amount=-100)
# Other transactions wait until this completes
```

**Pros:**
- ✅ No retry logic needed
- ✅ Guaranteed to work
- ✅ Simple to understand
- ✅ Good for high-contention scenarios

**Cons:**
- ❌ Locks held during entire transaction
- ❌ Potential for deadlocks
- ❌ Lower concurrency
- ❌ Can cause blocking

---

### Solution 3: **Atomic Operations (No Read)**

**How it works:** Use atomic UPDATE without reading first.

**Implementation:**
```sql
-- Instead of read-then-update
BEGIN;
SELECT balance FROM accounts WHERE id = 1;  -- Read
-- Calculate...
UPDATE accounts SET balance = 900 WHERE id = 1;  -- Update
COMMIT;

-- Use atomic operation
UPDATE accounts 
SET balance = balance - 100 
WHERE id = 1 
  AND balance >= 100;  -- Constraint check in same operation

-- Returns affected rows (0 if insufficient balance)
```

**Application Code:**
```python
def update_account_atomic(account_id, amount):
    """
    Atomic update without reading first
    """
    conn = get_connection()
    cursor = conn.cursor()
    
    try:
        # Atomic update with constraint
        cursor.execute("""
            UPDATE accounts 
            SET balance = balance + %s,
                updated_at = NOW()
            WHERE id = %s 
              AND balance + %s >= 0
        """, (amount, account_id, amount))
        
        if cursor.rowcount == 0:
            # Either account doesn't exist or insufficient funds
            cursor.execute(
                "SELECT balance FROM accounts WHERE id = %s",
                (account_id,)
            )
            result = cursor.fetchone()
            if result is None:
                raise AccountNotFound()
            else:
                raise InsufficientFunds()
        
        conn.commit()
        
        # Get new balance
        cursor.execute(
            "SELECT balance FROM accounts WHERE id = %s",
            (account_id,)
        )
        return cursor.fetchone()[0]
        
    except Exception as e:
        conn.rollback()
        raise
    finally:
        conn.close()

# Usage
new_balance = update_account_atomic(account_id=1, amount=-100)
```

**Pros:**
- ✅ Simplest solution
- ✅ Best performance
- ✅ No locking overhead
- ✅ No retry logic needed

**Cons:**
- ❌ Limited to simple operations
- ❌ Can't use for complex business logic
- ❌ Need to handle insufficient funds separately

---

### Solution 4: **Serializable Isolation Level**

**How it works:** PostgreSQL detects conflicts and aborts transactions.

```sql
-- Transaction A
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

SELECT balance FROM accounts WHERE id = 1;  -- 1000
-- Calculate...
UPDATE accounts SET balance = 900 WHERE id = 1;

COMMIT;  -- Success

-- Transaction B (concurrent)
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

SELECT balance FROM accounts WHERE id = 1;  -- 1000 (snapshot)
-- Calculate...
UPDATE accounts SET balance = 1200 WHERE id = 1;

COMMIT;
-- ERROR: could not serialize access due to concurrent update
-- Must retry ✅
```

**Application Code:** (See Q58 for comprehensive retry logic)

**Pros:**
- ✅ Strongest guarantee
- ✅ No manual locking needed
- ✅ Prevents all anomalies

**Cons:**
- ❌ Requires retry logic
- ❌ High failure rate under contention
- ❌ Slower performance

---

### Solution 5: **Advisory Locks**

**How it works:** Application-level locks using PostgreSQL's advisory lock functions.

```sql
-- Transaction A
BEGIN;

-- Acquire advisory lock (blocks if held by another transaction)
SELECT pg_advisory_xact_lock(1);  -- Lock ID: 1

-- Now safe to read and update
SELECT balance FROM accounts WHERE id = 1;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;

COMMIT;  -- Automatically releases advisory lock

-- Transaction B (concurrent)
BEGIN;

-- Waits here until Transaction A releases lock
SELECT pg_advisory_xact_lock(1);

-- Now can safely proceed
SELECT balance FROM accounts WHERE id = 1;
UPDATE accounts SET balance = balance + 200 WHERE id = 1;

COMMIT;
```

**Try variant (non-blocking):**
```sql
BEGIN;

-- Try to acquire lock, returns false if unavailable
SELECT pg_try_advisory_xact_lock(1);
-- Returns: true (acquired) or false (already locked)

IF lock_acquired THEN
    -- Process...
ELSE
    -- Lock not available, handle accordingly
    RAISE NOTICE 'Resource busy, try again later';
END IF;

COMMIT;
```

**Pros:**
- ✅ Application-level coordination
- ✅ Can lock logical resources (not just rows)
- ✅ Flexible

**Cons:**
- ❌ Manual lock management
- ❌ Potential for deadlocks
- ❌ More complex code

---

### Comparison of Solutions

| Solution | Performance | Complexity | Retries | Best For |
|----------|------------|------------|---------|----------|
| **Optimistic Locking** | ⭐⭐⭐⭐⭐ | Medium | Yes (5-15%) | Low contention |
| **Pessimistic (FOR UPDATE)** | ⭐⭐⭐ | Low | No | High contention |
| **Atomic Operations** | ⭐⭐⭐⭐⭐ | Low | No | Simple updates |
| **SERIALIZABLE** | ⭐⭐ | High | Yes (15-25%) | Complex logic |
| **Advisory Locks** | ⭐⭐⭐⭐ | High | No | Cross-table coordination |

---

### Real-World Example: Inventory Management

**Problem:**
```
E-commerce site with 1000 concurrent users
Users can purchase products (decrease inventory)
Must prevent overselling (negative stock)
High contention on popular products
```

**Solution 1: Optimistic Locking**
```sql
ALTER TABLE products ADD COLUMN version INTEGER DEFAULT 1;

-- Purchase attempt
BEGIN;

SELECT stock, version FROM products WHERE id = 101;
-- stock = 5, version = 42

-- Check sufficient stock
IF stock < quantity THEN
    RAISE EXCEPTION 'Insufficient stock';
END IF;

-- Update with version check
UPDATE products 
SET stock = stock - 3,
    version = version + 1
WHERE id = 101 AND version = 42;

IF ROW_COUNT() = 0 THEN
    ROLLBACK;
    RAISE EXCEPTION 'Concurrent modification, please retry';
END IF;

INSERT INTO orders (product_id, quantity) VALUES (101, 3);

COMMIT;
```

**Results:**
- Success rate: 85% on first attempt
- 15% retry rate during peak
- Average: 1.2 attempts per transaction
- Response time: 25ms average
- Throughput: 4500 orders/second

**Solution 2: Pessimistic Locking (FOR UPDATE)**
```sql
-- Purchase attempt
BEGIN;

-- Lock product row immediately
SELECT stock FROM products WHERE id = 101 FOR UPDATE;
-- Other transactions wait here

-- Check sufficient stock
IF stock < quantity THEN
    RAISE EXCEPTION 'Insufficient stock';
END IF;

-- Update (exclusive access)
UPDATE products 
SET stock = stock - 3
WHERE id = 101;

INSERT INTO orders (product_id, quantity) VALUES (101, 3);

COMMIT;  -- Release lock
```

**Results:**
- Success rate: 100% (no retries)
- Blocking: Transactions wait in queue
- Response time: 45ms average (includes wait time)
- Throughput: 2800 orders/second
- No retry logic needed

**Solution 3: Atomic Operation**
```sql
-- Purchase attempt (no transaction needed!)
UPDATE products 
SET stock = stock - 3
WHERE id = 101 
  AND stock >= 3;

-- Check if succeeded
IF ROW_COUNT() = 0 THEN
    -- Either insufficient stock or product doesn't exist
    SELECT stock FROM products WHERE id = 101;
    IF stock < 3 THEN
        RAISE EXCEPTION 'Insufficient stock';
    END IF;
END IF;

-- Insert order (in separate transaction if needed)
INSERT INTO orders (product_id, quantity) VALUES (101, 3);
```

**Results:**
- Success rate: 100%
- No blocking or retries
- Response time: 8ms average
- Throughput: 6200 orders/second
- ⭐ **Best performance**

---

### Job Queue Pattern (SKIP LOCKED)

**Problem:** Multiple workers processing jobs from a queue without conflicts.

```sql
-- Traditional approach (problematic)
BEGIN;

SELECT id FROM jobs WHERE status = 'pending' LIMIT 1;
UPDATE jobs SET status = 'processing' WHERE id = ...;

COMMIT;
-- Problem: Multiple workers select same job before updating!
```

**Solution: FOR UPDATE SKIP LOCKED**
```sql
-- Worker 1
BEGIN;

SELECT id, data 
FROM jobs 
WHERE status = 'pending'
ORDER BY priority DESC, created_at
LIMIT 1
FOR UPDATE SKIP LOCKED;  -- Skip jobs locked by other workers

-- If row returned, process it
UPDATE jobs SET status = 'processing' WHERE id = ...;

-- Process job...

UPDATE jobs SET status = 'completed' WHERE id = ...;

COMMIT;

-- Worker 2 (concurrent) - gets different job
BEGIN;

SELECT id, data 
FROM jobs 
WHERE status = 'pending'
ORDER BY priority DESC, created_at
LIMIT 1
FOR UPDATE SKIP LOCKED;  -- Skips job locked by Worker 1

-- Gets next available job ✅
```

**Benefits:**
- ✅ No conflicts between workers
- ✅ High throughput
- ✅ Natural load distribution
- ✅ No retry logic needed

---

### Best Practices

#### 1. **Choose Right Strategy Based on Contention**

```python
# Low contention (< 5% conflicts): Optimistic locking
if expected_conflict_rate < 0.05:
    use_optimistic_locking()

# High contention (> 20% conflicts): Pessimistic locking
elif expected_conflict_rate > 0.20:
    use_pessimistic_locking()

# Simple operations: Atomic updates
else:
    use_atomic_operations()
```

#### 2. **Always Have Retry Logic**

```python
def with_retry(func, max_retries=3):
    for attempt in range(max_retries):
        try:
            return func()
        except (OptimisticLockError, SerializationError) as e:
            if attempt == max_retries - 1:
                raise
            time.sleep(0.1 * (2 ** attempt))
```

#### 3. **Lock in Consistent Order**

```sql
-- ❌ Bad: Random order (deadlock risk)
UPDATE accounts SET balance = balance - 100 WHERE id IN (5, 2, 8);

-- ✅ Good: Sorted order
SELECT * FROM accounts WHERE id IN (2, 5, 8) ORDER BY id FOR UPDATE;
UPDATE accounts SET balance = balance - 100 WHERE id IN (2, 5, 8);
```

#### 4. **Keep Transactions Short**

```sql
-- ❌ Bad: Long transaction holding locks
BEGIN;
SELECT * FROM products WHERE id = 101 FOR UPDATE;
-- User thinks for 5 minutes...
UPDATE products SET stock = stock - 1 WHERE id = 101;
COMMIT;

-- ✅ Good: Short transaction
-- Do thinking outside transaction
BEGIN;
SELECT * FROM products WHERE id = 101 FOR UPDATE;
UPDATE products SET stock = stock - 1 WHERE id = 101;
COMMIT;  -- Release locks quickly
```

#### 5. **Monitor Contention**

```sql
-- Check lock waits
SELECT 
    COUNT(*) as waiting_transactions,
    locktype,
    mode
FROM pg_locks
WHERE NOT granted
GROUP BY locktype, mode;

-- Check blocking queries
SELECT 
    blocked.pid AS blocked_pid,
    blocked.query AS blocked_query,
    blocking.pid AS blocking_pid,
    blocking.query AS blocking_query
FROM pg_stat_activity blocked
JOIN pg_locks blocked_locks ON blocked.pid = blocked_locks.pid
JOIN pg_locks blocking_locks ON blocked_locks.locktype = blocking_locks.locktype
JOIN pg_stat_activity blocking ON blocking.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted AND blocking_locks.granted;
```

---

### Summary

**Quick Decision Guide:**

```
Simple increment/decrement?
└─ YES → Use Atomic Operations (UPDATE ... SET x = x + 1)

Need complex business logic with reads?
├─ Low contention (< 5% conflicts)?
│   └─ YES → Optimistic Locking (version column)
└─ High contention (> 20% conflicts)?
    └─ YES → Pessimistic Locking (FOR UPDATE)

Need strongest guarantees?
└─ YES → SERIALIZABLE isolation level

Job queue / worker pattern?
└─ YES → FOR UPDATE SKIP LOCKED
```

---

### Interview Tips

- **Junior**: "Use SELECT FOR UPDATE to lock rows before updating, preventing concurrent modifications. FOR UPDATE blocks other transactions until current transaction commits. Alternatively, use optimistic locking with version column to detect conflicts and retry."

- **Mid-Level**: "Implemented optimistic locking for inventory system using version column. Transactions read stock and version, update only if version matches. 15% retry rate during peak traffic. Used exponential backoff for retries. For high-contention operations, switched to FOR UPDATE for guaranteed success without retries. Atomic operations (UPDATE SET stock = stock - 1 WHERE stock >= 1) provided best performance at 6200 TPS."

- **Senior**: "Designed concurrent update strategy based on contention: atomic operations for simple updates (6200 TPS), optimistic locking for medium contention (4500 TPS, 15% retry rate), pessimistic locking (FOR UPDATE) for high contention (2800 TPS, no retries). Implemented job queue using FOR UPDATE SKIP LOCKED allowing multiple workers to process jobs without conflicts. Built monitoring tracking lock waits and blocking queries, identified deadlock patterns and fixed by consistent lock ordering. Reduced average response time from 45ms to 8ms by choosing appropriate locking strategy per operation type."

- **Advanced**: "Architected hybrid locking system: analyzed transaction patterns and automatically selected locking strategy (optimistic/pessimistic/atomic) based on historical contention rates and data access patterns. Built adaptive retry logic adjusting backoff based on real-time conflict rates. Implemented lock escalation prevention through early consistent ordering and batch size optimization. Created contention monitoring dashboard showing hot rows, lock wait times, and deadlock frequency. Optimized high-contention product updates from 2800 TPS to 5200 TPS by implementing row-level sharding and pre-allocated inventory buckets. Maintained 99.99% success rate while handling 50K concurrent updates."

</details>

60. What are locks in PostgreSQL?

<details>
<summary><strong>Answer</strong></summary>

### What are Locks?

**Locks** are PostgreSQL's mechanism for controlling concurrent access to data, ensuring that multiple transactions can work together safely without corrupting data or violating consistency.

**Purpose:**
- Prevent concurrent transactions from interfering with each other
- Ensure ACID properties are maintained
- Allow maximum safe concurrency

**Key Concept:** PostgreSQL uses **Multi-Version Concurrency Control (MVCC)** which minimizes locking compared to traditional databases. Most reads don't block writes, and writes don't block reads!

---

### Types of Locks

PostgreSQL has multiple lock types operating at different levels:

1. **Table-level locks**
2. **Row-level locks**
3. **Page-level locks**
4. **Advisory locks**
5. **Deadlock detection**

---

### Table-Level Locks

**8 different modes** from least to most restrictive:

| Lock Mode | Conflicts With | Acquired By | Blocks |
|-----------|----------------|-------------|---------|
| **ACCESS SHARE** | ACCESS EXCLUSIVE | SELECT | Only DROP TABLE |
| **ROW SHARE** | EXCLUSIVE, ACCESS EXCLUSIVE | SELECT FOR UPDATE | ALTER TABLE, DROP |
| **ROW EXCLUSIVE** | SHARE, EXCLUSIVE, ACCESS EXCLUSIVE | INSERT, UPDATE, DELETE | Most ALTER TABLE |
| **SHARE UPDATE EXCLUSIVE** | Self, SHARE, EXCLUSIVE, ACCESS EXCLUSIVE | VACUUM, CREATE INDEX CONCURRENTLY | Concurrent VACUUM |
| **SHARE** | ROW EXCLUSIVE, EXCLUSIVE, ACCESS EXCLUSIVE | CREATE INDEX | INSERT, UPDATE, DELETE |
| **SHARE ROW EXCLUSIVE** | ROW EXCLUSIVE, SHARE, EXCLUSIVE, ACCESS EXCLUSIVE | - | INSERT, UPDATE, DELETE |
| **EXCLUSIVE** | ROW SHARE, ROW EXCLUSIVE, SHARE, SHARE ROW EXCLUSIVE, EXCLUSIVE, ACCESS EXCLUSIVE | REFRESH MATERIALIZED VIEW | All except SELECT |
| **ACCESS EXCLUSIVE** | ALL | DROP TABLE, TRUNCATE, ALTER TABLE | Everything |

#### **Common Table Locks in Practice:**

**1. ACCESS SHARE (SELECT)**
```sql
-- Reading data
SELECT * FROM products;
-- Acquires: ACCESS SHARE lock

-- Multiple SELECTs can run concurrently ✅
-- Does NOT block: INSERT, UPDATE, DELETE ✅
-- Blocks: DROP TABLE, TRUNCATE ❌
```

**2. ROW EXCLUSIVE (INSERT, UPDATE, DELETE)**
```sql
-- Modifying data
INSERT INTO products (name, price) VALUES ('Widget', 9.99);
UPDATE products SET price = 10.99 WHERE id = 1;
DELETE FROM products WHERE id = 2;
-- Acquires: ROW EXCLUSIVE lock

-- Multiple writes can run concurrently ✅
-- Does NOT block: SELECT ✅
-- Does NOT block: Other INSERT/UPDATE/DELETE ✅
-- Blocks: LOCK TABLE ... IN SHARE MODE, DROP TABLE ❌
```

**3. ACCESS EXCLUSIVE (DROP, TRUNCATE, REINDEX)**
```sql
-- Structure changes
DROP TABLE products;
TRUNCATE products;
ALTER TABLE products ADD COLUMN category VARCHAR(50);
REINDEX TABLE products;
-- Acquires: ACCESS EXCLUSIVE lock

-- Blocks: EVERYTHING ❌
-- No reads or writes until complete
```

#### **Viewing Table Locks:**

```sql
-- Check current table locks
SELECT 
    l.locktype,
    l.relation::regclass AS table_name,
    l.mode,
    l.granted,
    a.pid,
    a.query
FROM pg_locks l
JOIN pg_stat_activity a ON l.pid = a.pid
WHERE l.relation IS NOT NULL
ORDER BY l.relation;
```

**Example Output:**
```
 locktype |  table_name  |        mode         | granted |  pid  |           query
----------+--------------+---------------------+---------+-------+---------------------------
 relation | products     | AccessShareLock     | t       | 12345 | SELECT * FROM products
 relation | products     | RowExclusiveLock    | t       | 12346 | UPDATE products SET...
 relation | orders       | AccessExclusiveLock | f       | 12347 | ALTER TABLE orders...
                                                   ↑ Waiting (not granted)
```

---

### Row-Level Locks

**More granular** than table locks. Only lock specific rows, allowing higher concurrency.

#### **Four Row Lock Modes:**

| Lock Mode | Acquired By | Conflicts With | Use Case |
|-----------|-------------|----------------|----------|
| **FOR UPDATE** | SELECT FOR UPDATE | FOR UPDATE, FOR NO KEY UPDATE, FOR SHARE, FOR KEY SHARE, UPDATE, DELETE | Exclusive write access |
| **FOR NO KEY UPDATE** | SELECT FOR NO KEY UPDATE | FOR UPDATE, FOR NO KEY UPDATE, FOR SHARE, UPDATE, DELETE | Update without changing key |
| **FOR SHARE** | SELECT FOR SHARE | FOR UPDATE, FOR NO KEY UPDATE, UPDATE, DELETE | Prevent modifications |
| **FOR KEY SHARE** | SELECT FOR KEY SHARE | FOR UPDATE, DELETE | Weakest, allow updates |

#### **1. FOR UPDATE (Exclusive Lock)**

```sql
BEGIN;

-- Lock rows exclusively
SELECT * FROM accounts WHERE id IN (1, 2) FOR UPDATE;
-- Other transactions CANNOT:
-- - SELECT FOR UPDATE (blocks)
-- - UPDATE (blocks)
-- - DELETE (blocks)
-- Other transactions CAN:
-- - SELECT (without FOR UPDATE) ✅

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;  -- Releases locks
```

**Example: Money Transfer**
```sql
-- Transaction A
BEGIN;
SELECT balance FROM accounts WHERE id = 1 FOR UPDATE;
-- Locks row 1

-- Transaction B (concurrent)
BEGIN;
SELECT balance FROM accounts WHERE id = 1 FOR UPDATE;
-- BLOCKS here, waiting for Transaction A

-- Transaction A completes
UPDATE accounts SET balance = 900 WHERE id = 1;
COMMIT;  -- Releases lock

-- Transaction B now proceeds
-- Sees new balance: 900
```

#### **2. FOR SHARE (Shared Lock)**

```sql
BEGIN;

-- Shared lock (multiple transactions can hold simultaneously)
SELECT * FROM products WHERE id = 101 FOR SHARE;

-- Other transactions CAN:
-- - SELECT FOR SHARE ✅ (multiple shared locks OK)
-- - SELECT (without FOR SHARE) ✅
-- Other transactions CANNOT:
-- - UPDATE (blocks) ❌
-- - DELETE (blocks) ❌
-- - SELECT FOR UPDATE (blocks) ❌

COMMIT;
```

**Use Case: Prevent deletion while referencing**
```sql
-- Transaction A: Creating order
BEGIN;

-- Lock products (prevent deletion/update)
SELECT * FROM products WHERE id IN (101, 102) FOR SHARE;

-- Safe to create order items now
INSERT INTO order_items (product_id, quantity) VALUES (101, 2);
INSERT INTO order_items (product_id, quantity) VALUES (102, 1);

COMMIT;

-- Transaction B (concurrent): Try to delete product
BEGIN;
DELETE FROM products WHERE id = 101;
-- BLOCKS until Transaction A commits (prevents FK violation)
```

#### **3. FOR UPDATE NOWAIT**

```sql
BEGIN;

-- Try to lock, error immediately if unavailable
SELECT * FROM accounts WHERE id = 1 FOR UPDATE NOWAIT;
-- If locked by another transaction:
-- ERROR: could not obtain lock on row in relation "accounts"

-- Handle in application:
-- "Resource busy, please try again"

ROLLBACK;
```

#### **4. FOR UPDATE SKIP LOCKED**

```sql
-- Get available (unlocked) rows
SELECT * FROM jobs 
WHERE status = 'pending'
ORDER BY priority DESC
LIMIT 10
FOR UPDATE SKIP LOCKED;

-- Returns only rows NOT locked by other transactions
-- Perfect for job queues with multiple workers!
```

**Job Queue Example:**
```sql
-- Worker 1
BEGIN;
SELECT id FROM jobs WHERE status = 'pending' LIMIT 1 FOR UPDATE SKIP LOCKED;
-- Gets job ID: 123

UPDATE jobs SET status = 'processing' WHERE id = 123;
-- Process job...
UPDATE jobs SET status = 'completed' WHERE id = 123;
COMMIT;

-- Worker 2 (concurrent)
BEGIN;
SELECT id FROM jobs WHERE status = 'pending' LIMIT 1 FOR UPDATE SKIP LOCKED;
-- Gets job ID: 124 (skips 123 which is locked by Worker 1)

-- No conflicts! ✅
```

---

### Viewing Row Locks

```sql
-- Check row-level locks
SELECT 
    l.locktype,
    l.relation::regclass AS table_name,
    l.page,
    l.tuple,
    l.mode,
    l.granted,
    a.query
FROM pg_locks l
JOIN pg_stat_activity a ON l.pid = a.pid
WHERE l.locktype = 'tuple'
ORDER BY l.relation;
```

---

### Advisory Locks

**Application-level locks** for coordinating access to logical resources.

#### **Session-Level Advisory Locks:**

```sql
-- Acquire lock (blocks if unavailable)
SELECT pg_advisory_lock(123);

-- Critical section...

-- Release lock
SELECT pg_advisory_unlock(123);
```

#### **Transaction-Level Advisory Locks (Auto-Release):**

```sql
BEGIN;

-- Acquire lock (auto-released on commit/rollback)
SELECT pg_advisory_xact_lock(123);

-- Critical section...

COMMIT;  -- Automatically releases lock
```

#### **Try Lock (Non-Blocking):**

```sql
-- Try to acquire, returns true/false
SELECT pg_try_advisory_lock(123);
-- Returns: true (acquired) or false (unavailable)

IF lock_acquired THEN
    -- Proceed with critical section
ELSE
    -- Handle "resource busy"
END IF;

SELECT pg_advisory_unlock(123);
```

**Use Cases:**
```sql
-- 1. Ensure only one instance of job runs
BEGIN;

IF pg_try_advisory_xact_lock(hash('daily-report-job')) THEN
    -- Run report...
ELSE
    RAISE NOTICE 'Job already running, skipping';
END IF;

COMMIT;

-- 2. Coordinate across multiple tables
BEGIN;

SELECT pg_advisory_xact_lock(hash('user-' || user_id));

-- Multiple operations on user data
UPDATE users SET ... WHERE id = user_id;
UPDATE profiles SET ... WHERE user_id = user_id;
UPDATE settings SET ... WHERE user_id = user_id;

COMMIT;  -- All user data operations atomic and coordinated
```

---

### Lock Conflicts and Waiting

#### **What Happens When Lock Conflicts:**

```sql
-- Transaction A
BEGIN;
UPDATE products SET stock = stock - 1 WHERE id = 101;
-- Holds ROW EXCLUSIVE lock on row 101

-- Transaction B (concurrent)
BEGIN;
UPDATE products SET stock = stock - 2 WHERE id = 101;
-- BLOCKS here, waiting for Transaction A to release lock

-- After Transaction A commits/rollbacks:
-- Transaction B proceeds
```

#### **Check Blocking Queries:**

```sql
SELECT 
    blocked_locks.pid AS blocked_pid,
    blocked_activity.usename AS blocked_user,
    blocking_locks.pid AS blocking_pid,
    blocking_activity.usename AS blocking_user,
    blocked_activity.query AS blocked_query,
    blocking_activity.query AS current_query_in_blocking_process,
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

---

### Lock Timeouts

**Prevent transactions from waiting forever:**

```sql
-- Set lock timeout (statement fails if can't acquire lock within timeout)
SET lock_timeout = '5s';

BEGIN;
UPDATE products SET stock = stock - 1 WHERE id = 101;
-- If locked by another transaction for > 5s:
-- ERROR: canceling statement due to lock timeout
COMMIT;

-- Set statement timeout (entire statement must complete within timeout)
SET statement_timeout = '10s';

-- Set idle in transaction timeout
SET idle_in_transaction_session_timeout = '5min';
```

**postgresql.conf defaults:**
```conf
lock_timeout = 0  # 0 = wait forever (default)
statement_timeout = 0  # 0 = wait forever (default)
idle_in_transaction_session_timeout = 0  # 0 = disabled
deadlock_timeout = 1s  # How long to wait before checking for deadlock
```

---

### Lock Monitoring Queries

#### **1. Show All Locks:**

```sql
SELECT 
    l.locktype,
    l.database,
    l.relation::regclass AS table,
    l.page,
    l.tuple,
    l.virtualxid,
    l.transactionid,
    l.mode,
    l.granted,
    a.pid,
    a.usename,
    a.application_name,
    a.client_addr,
    a.query_start,
    a.query
FROM pg_locks l
LEFT JOIN pg_stat_activity a ON l.pid = a.pid
ORDER BY l.granted, a.query_start;
```

#### **2. Show Lock Waits:**

```sql
SELECT 
    l.locktype,
    l.relation::regclass,
    l.mode,
    l.pid,
    a.usename,
    a.query,
    a.wait_event_type,
    a.wait_event,
    now() - a.query_start AS waiting_duration
FROM pg_locks l
JOIN pg_stat_activity a ON l.pid = a.pid
WHERE NOT l.granted
ORDER BY waiting_duration DESC;
```

#### **3. Count Locks by Type:**

```sql
SELECT 
    locktype,
    mode,
    COUNT(*) AS count,
    SUM(CASE WHEN granted THEN 1 ELSE 0 END) AS granted,
    SUM(CASE WHEN NOT granted THEN 1 ELSE 0 END) AS waiting
FROM pg_locks
GROUP BY locktype, mode
ORDER BY count DESC;
```

---

### Real-World Example: E-Commerce Flash Sale

**Problem:**
```
Flash sale: 100 units of popular product
10,000 users trying to buy simultaneously
Must prevent overselling
High contention on single row
```

**Solution 1: Row Lock (Basic)**
```sql
BEGIN;

-- Lock product row
SELECT stock FROM products WHERE id = 101 FOR UPDATE;

IF stock > 0 THEN
    UPDATE products SET stock = stock - 1 WHERE id = 101;
    INSERT INTO orders (product_id, user_id) VALUES (101, 1234);
    COMMIT;
ELSE
    ROLLBACK;
    RAISE NOTICE 'Out of stock';
END IF;
```

**Problem:** 
- All 10,000 users queue on single row lock
- Very slow (100 TPS)
- Most users timeout

**Solution 2: Optimistic Locking**
```sql
-- (See previous question for details)
-- Better: 4500 TPS with retries
```

**Solution 3: Inventory Sharding** (Best)
```sql
-- Pre-allocate inventory into 10 buckets
CREATE TABLE inventory_buckets (
    product_id INT,
    bucket_id INT,
    stock INT,
    PRIMARY KEY (product_id, bucket_id)
);

-- Initial setup: 10 buckets with 10 units each
INSERT INTO inventory_buckets (product_id, bucket_id, stock)
SELECT 101, bucket, 10 FROM generate_series(1, 10) AS bucket;

-- User purchase
BEGIN;

-- Randomly select bucket (distribute load)
bucket_id = user_id % 10;

-- Lock only one bucket (10x less contention!)
SELECT stock FROM inventory_buckets 
WHERE product_id = 101 AND bucket_id = bucket_id
FOR UPDATE;

IF stock > 0 THEN
    UPDATE inventory_buckets 
    SET stock = stock - 1 
    WHERE product_id = 101 AND bucket_id = bucket_id;
    
    INSERT INTO orders (product_id, user_id) VALUES (101, user_id);
    COMMIT;
ELSE
    -- Try next bucket
    ROLLBACK;
END IF;
```

**Results:**
- 10x less contention (10 separate locks instead of 1)
- Throughput: 8500 TPS (85x improvement!)
- Average latency: 12ms (down from 1000ms)
- 99.9% success rate

---

### Best Practices

#### 1. **Use Appropriate Lock Level**
- SELECT: No explicit lock needed (ACCESS SHARE auto)
- Read-then-update: `FOR UPDATE`
- Prevent deletion: `FOR SHARE`
- Job queue: `FOR UPDATE SKIP LOCKED`

#### 2. **Lock in Consistent Order**
```sql
-- ✅ Always lock in ascending ID order
SELECT * FROM accounts WHERE id IN (2, 5, 8) ORDER BY id FOR UPDATE;
```

#### 3. **Keep Transactions Short**
```sql
-- ✅ Short transaction
BEGIN;
SELECT ... FOR UPDATE;
UPDATE ...;
COMMIT;  -- Release locks ASAP
```

#### 4. **Set Timeouts**
```sql
SET lock_timeout = '30s';
SET statement_timeout = '1min';
SET idle_in_transaction_session_timeout = '5min';
```

#### 5. **Monitor Lock Waits**
```sql
-- Alert if too many lock waits
SELECT COUNT(*) FROM pg_locks WHERE NOT granted;
```

---

### Summary

| Lock Type | Granularity | Use Case | Performance |
|-----------|-------------|----------|-------------|
| **Table locks** | Entire table | DDL operations | Low contention |
| **Row locks** | Individual rows | DML operations | High concurrency |
| **Advisory locks** | Logical resources | App coordination | Flexible |

**Key Takeaways:**
- PostgreSQL's MVCC minimizes locking (reads don't block writes!)
- Row locks provide high concurrency
- Use `FOR UPDATE` for exclusive access
- Use `SKIP LOCKED` for job queues
- Always monitor and set lock timeouts

---

### Interview Tips

- **Junior**: "PostgreSQL has table-level and row-level locks. Row locks are acquired by UPDATE/DELETE or SELECT FOR UPDATE. FOR UPDATE locks rows exclusively, preventing other transactions from modifying them. FOR SHARE allows multiple reads but prevents writes."

- **Mid-Level**: "Used FOR UPDATE to lock rows before updating in money transfer operations, ensuring no lost updates. Implemented job queue using FOR UPDATE SKIP LOCKED allowing multiple workers to process jobs without conflicts. Set lock_timeout to 30s preventing indefinite waits. Monitored lock waits with pg_locks showing blocked queries and blocking processes."

- **Senior**: "Optimized high-contention inventory updates from 100 TPS to 8500 TPS using inventory sharding (10 buckets) reducing lock contention 10x. Used consistent lock ordering (ORDER BY id) to prevent deadlocks. Implemented adaptive locking strategy: FOR UPDATE for high-contention rows, optimistic locking for low-contention. Built monitoring dashboard tracking lock waits, blocking queries, and deadlock frequency. Reduced average lock wait time from 1000ms to 12ms."

- **Advanced**: "Designed distributed locking system using advisory locks for cross-database coordination. Implemented lock contention analyzer identifying hot rows and automatically suggesting sharding strategies. Built predictive lock timeout system adjusting timeouts based on query patterns and historical wait times. Used lock escalation prevention through batch size optimization and pre-allocated resource pools. Created lock visualization dashboard showing real-time lock dependency graphs and bottleneck identification. Achieved 99.99% lock acquisition success rate while handling 100K concurrent transactions."

</details>

61. What is the difference between row-level and table-level locks?

<details>
<summary><strong>Answer</strong></summary>

### Key Differences Overview

| Aspect | Row-Level Locks | Table-Level Locks |
|--------|----------------|-------------------|
| **Granularity** | Individual rows | Entire table |
| **Concurrency** | High (many transactions can work simultaneously) | Low (blocks entire table) |
| **Overhead** | Higher (more lock records) | Lower (single lock) |
| **Acquired by** | DML (INSERT, UPDATE, DELETE, SELECT FOR UPDATE) | DDL (ALTER, DROP, TRUNCATE, explicit LOCK TABLE) |
| **Typical use** | OLTP, concurrent updates | DDL operations, bulk operations |
| **Performance** | Better for small updates | Better for bulk operations |
| **Lock contention** | Only on specific rows | Entire table |

---

### Table-Level Locks

**Characteristics:**
- Lock the **entire table**
- **Lower overhead** (single lock record)
- **Less concurrency** (blocks more operations)
- **Automatic** for DDL operations
- Can be explicit with `LOCK TABLE` command

#### **8 Table Lock Modes (from least to most restrictive):**

```sql
-- 1. ACCESS SHARE (least restrictive)
SELECT * FROM products;
-- Blocks: DROP TABLE, TRUNCATE, ALTER TABLE
-- Allows: Everything else (INSERT, UPDATE, DELETE, other SELECT)

-- 2. ROW SHARE
SELECT * FROM products WHERE id = 1 FOR UPDATE;
-- Blocks: EXCLUSIVE, ACCESS EXCLUSIVE locks
-- Allows: Most operations

-- 3. ROW EXCLUSIVE (most DML)
INSERT INTO products (name) VALUES ('Widget');
UPDATE products SET price = 9.99 WHERE id = 1;
DELETE FROM products WHERE id = 2;
-- Blocks: SHARE, EXCLUSIVE, ACCESS EXCLUSIVE locks
-- Allows: SELECT, other INSERT/UPDATE/DELETE

-- 4. SHARE UPDATE EXCLUSIVE
VACUUM products;
CREATE INDEX CONCURRENTLY idx_name ON products(name);
-- Blocks: Another VACUUM, SHARE, EXCLUSIVE locks

-- 5. SHARE
CREATE INDEX idx_price ON products(price);
-- Blocks: INSERT, UPDATE, DELETE
-- Allows: SELECT

-- 6. SHARE ROW EXCLUSIVE
-- Rarely used explicitly

-- 7. EXCLUSIVE
REFRESH MATERIALIZED VIEW product_summary;
-- Blocks: Everything except SELECT

-- 8. ACCESS EXCLUSIVE (most restrictive)
DROP TABLE products;
TRUNCATE products;
ALTER TABLE products ADD COLUMN category VARCHAR(50);
LOCK TABLE products IN ACCESS EXCLUSIVE MODE;
-- Blocks: EVERYTHING (including SELECT)
```

#### **Impact on Concurrency:**

```sql
-- Transaction A: ALTER TABLE (ACCESS EXCLUSIVE lock)
BEGIN;
ALTER TABLE products ADD COLUMN description TEXT;
-- Holds ACCESS EXCLUSIVE lock for entire duration (could be minutes!)

-- Transaction B: SELECT (BLOCKS!)
SELECT * FROM products WHERE id = 1;
-- Waits for ALTER TABLE to complete

-- Transaction C: INSERT (BLOCKS!)
INSERT INTO products (name) VALUES ('New Product');
-- Waits

-- Transaction D: UPDATE (BLOCKS!)
UPDATE products SET price = 10.99 WHERE id = 5;
-- Waits

-- All operations blocked by single ALTER TABLE! ❌
```

#### **Explicit Table Locking:**

```sql
BEGIN;

-- Explicitly lock entire table
LOCK TABLE products IN EXCLUSIVE MODE;

-- Now can perform multiple operations knowing table won't change
UPDATE products SET price = price * 1.1 WHERE category = 'Electronics';
UPDATE products SET price = price * 1.05 WHERE category = 'Books';

COMMIT;
```

**Use Cases for Explicit Table Locks:**
- Bulk updates across entire table
- Ensuring table structure doesn't change
- Coordinating complex multi-step operations
- Data migrations

---

### Row-Level Locks

**Characteristics:**
- Lock **individual rows only**
- **Higher concurrency** (other rows freely accessible)
- **More overhead** (lock record per row)
- **Automatic** for DML operations
- Acquired with `FOR UPDATE`, `FOR SHARE`, etc.

#### **4 Row Lock Modes:**

```sql
-- 1. FOR UPDATE (Exclusive)
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
-- Locks only row with id=1
-- Blocks: FOR UPDATE, FOR NO KEY UPDATE, FOR SHARE, UPDATE, DELETE on row 1
-- Allows: Operations on ALL other rows ✅

-- 2. FOR NO KEY UPDATE
SELECT * FROM accounts WHERE id = 1 FOR NO KEY UPDATE;
-- Like FOR UPDATE but allows foreign key checks
-- Blocks: FOR UPDATE, FOR NO KEY UPDATE, FOR SHARE, UPDATE, DELETE on row 1

-- 3. FOR SHARE
SELECT * FROM products WHERE id = 101 FOR SHARE;
-- Locks row 101 for shared access
-- Blocks: UPDATE, DELETE, FOR UPDATE on row 101
-- Allows: Other FOR SHARE on row 101, all operations on other rows

-- 4. FOR KEY SHARE (Weakest)
SELECT * FROM orders WHERE id = 1 FOR KEY SHARE;
-- Locks row for reading key
-- Blocks: DELETE, FOR UPDATE on row 1
-- Allows: UPDATE (non-key columns), all operations on other rows
```

#### **High Concurrency Example:**

```sql
-- Transaction A: Update row 1
BEGIN;
UPDATE accounts SET balance = 900 WHERE id = 1;
-- Locks only row 1

-- Transaction B: Update row 2 (CONCURRENT, no blocking!)
BEGIN;
UPDATE accounts SET balance = 1500 WHERE id = 2;
-- Locks only row 2, proceeds immediately ✅

-- Transaction C: Update row 3 (CONCURRENT!)
BEGIN;
UPDATE accounts SET balance = 2000 WHERE id = 3;
-- Locks only row 3, proceeds immediately ✅

-- Transaction D: Update row 1 (BLOCKS)
BEGIN;
UPDATE accounts SET balance = 950 WHERE id = 1;
-- Waits for Transaction A (same row)

-- All can commit independently (except D waits for A)
```

---

### Side-by-Side Comparison

#### Scenario 1: **Multiple Users Updating Different Rows**

**Table-Level Locks (Serialized):**
```sql
-- User 1: Update product 1
BEGIN;
UPDATE products SET stock = 50 WHERE id = 1;
-- Acquires ROW EXCLUSIVE table lock

-- User 2: Update product 2 (SAME TABLE)
BEGIN;
UPDATE products SET stock = 75 WHERE id = 2;
-- Can proceed! ROW EXCLUSIVE doesn't block other ROW EXCLUSIVE ✅

-- But if User 1 had used SHARE lock:
BEGIN;
LOCK TABLE products IN SHARE MODE;

-- User 2: Update product 2
UPDATE products SET stock = 75 WHERE id = 2;
-- BLOCKS! ❌ SHARE lock blocks all writes
```

**Row-Level Locks (Concurrent):**
```sql
-- User 1: Update product 1
BEGIN;
UPDATE products SET stock = 50 WHERE id = 1;
-- Locks only row id=1

-- User 2: Update product 2 (different row)
BEGIN;
UPDATE products SET stock = 75 WHERE id = 2;
-- Locks only row id=2, proceeds immediately ✅

-- User 3: Update product 3
BEGIN;
UPDATE products SET stock = 100 WHERE id = 3;
-- Locks only row id=3, proceeds immediately ✅

-- High concurrency! All three transactions run in parallel ✅
```

#### Scenario 2: **Bulk Operations**

**Row-Level Locks (Inefficient):**
```sql
-- Update 1 million rows
BEGIN;

UPDATE products SET price = price * 1.1;
-- Acquires row locks on 1 million rows!
-- Memory overhead: ~100 bytes × 1M = 100MB just for locks
-- Slower due to lock management overhead

COMMIT;
```

**Table-Level Lock (Efficient):**
```sql
-- Update 1 million rows
BEGIN;

-- Explicitly acquire table lock
LOCK TABLE products IN EXCLUSIVE MODE;

UPDATE products SET price = price * 1.1;
-- Single table lock, minimal overhead
-- Faster for bulk operations

COMMIT;
```

#### Scenario 3: **DDL Operations**

**Row-Level Locks (Not Applicable):**
```sql
-- DDL operations ALWAYS use table locks
ALTER TABLE products ADD COLUMN category VARCHAR(50);
-- Acquires ACCESS EXCLUSIVE table lock (blocks everything)
-- No choice, cannot use row locks for DDL
```

**Table-Level Lock (Required):**
```sql
BEGIN;

-- Must lock entire table for structure changes
ALTER TABLE products ADD COLUMN category VARCHAR(50);
-- ACCESS EXCLUSIVE lock

-- Blocks:
-- - All SELECT queries ❌
-- - All INSERT/UPDATE/DELETE ❌
-- - All other operations ❌

COMMIT;
```

---

### Performance Impact

#### **Benchmark: 1000 Concurrent Updates**

**Test Setup:**
```sql
CREATE TABLE test_accounts (
    id SERIAL PRIMARY KEY,
    balance NUMERIC(10,2)
);

INSERT INTO test_accounts (balance)
SELECT 1000 FROM generate_series(1, 10000);
```

**Test 1: Row-Level Locks (Different Rows)**
```sql
-- 1000 transactions, each updating different row
Transaction 1: UPDATE test_accounts SET balance = 900 WHERE id = 1;
Transaction 2: UPDATE test_accounts SET balance = 900 WHERE id = 2;
Transaction 3: UPDATE test_accounts SET balance = 900 WHERE id = 3;
...
Transaction 1000: UPDATE test_accounts SET balance = 900 WHERE id = 1000;

Results:
- Throughput: 5,200 TPS
- Average latency: 15ms
- Concurrent: All run in parallel ✅
```

**Test 2: Table-Level Lock (SHARE MODE)**
```sql
-- 1000 transactions, each wants SHARE lock
Transaction 1: LOCK TABLE test_accounts IN SHARE MODE; SELECT ...;
Transaction 2: LOCK TABLE test_accounts IN SHARE MODE; SELECT ...;
...

Results:
- Throughput: 4,800 TPS
- Average latency: 18ms
- Concurrent: Multiple SHARE locks OK ✅
```

**Test 3: Table-Level Lock (EXCLUSIVE MODE)**
```sql
-- 1000 transactions, each wants EXCLUSIVE lock
Transaction 1: LOCK TABLE test_accounts IN EXCLUSIVE MODE; UPDATE ...;
Transaction 2: LOCK TABLE test_accounts IN EXCLUSIVE MODE; UPDATE ...;
...

Results:
- Throughput: 450 TPS (90% slower!)
- Average latency: 2,200ms
- Serialized: All wait in queue ❌
```

---

### When to Use Each

#### **Use Row-Level Locks When:**

**✅ OLTP Applications (Most Common)**
```sql
-- Web app: User updates their profile
UPDATE users SET email = 'new@example.com' WHERE id = 123;
-- Only locks row 123, other users unaffected ✅
```

**✅ High Concurrency Required**
```sql
-- E-commerce: Many users buying different products
UPDATE products SET stock = stock - 1 WHERE id = 101;
UPDATE products SET stock = stock - 1 WHERE id = 202;
UPDATE products SET stock = stock - 1 WHERE id = 303;
-- All concurrent ✅
```

**✅ Small Updates**
```sql
-- Update 1-1000 rows
UPDATE orders SET status = 'shipped' WHERE id IN (1, 2, 3);
-- Low overhead ✅
```

**✅ Mixed Read/Write Workloads**
```sql
-- Transaction A: Update row 1
UPDATE accounts SET balance = 900 WHERE id = 1;

-- Transaction B: Read row 2 (concurrent)
SELECT balance FROM accounts WHERE id = 2;
-- No blocking ✅
```

#### **Use Table-Level Locks When:**

**✅ DDL Operations (Automatic)**
```sql
-- Must use table lock (no choice)
ALTER TABLE products ADD COLUMN description TEXT;
CREATE INDEX idx_name ON products(name);
DROP TABLE old_products;
```

**✅ Bulk Updates (Entire Table)**
```sql
BEGIN;

-- Updating millions of rows
LOCK TABLE products IN EXCLUSIVE MODE;

UPDATE products SET price = price * 1.1;
-- 5 million rows updated

COMMIT;

-- More efficient than 5M row locks
```

**✅ Data Migrations**
```sql
BEGIN;

-- Ensure no changes during migration
LOCK TABLE old_products IN ACCESS EXCLUSIVE MODE;
LOCK TABLE new_products IN ACCESS EXCLUSIVE MODE;

-- Copy data
INSERT INTO new_products SELECT * FROM old_products;

-- Drop old table
DROP TABLE old_products;

COMMIT;
```

**✅ Ensure Consistency Across Entire Table**
```sql
BEGIN;

-- Prevent any changes during calculation
LOCK TABLE accounts IN SHARE MODE;

-- Calculate total balance (ensure no concurrent changes)
SELECT SUM(balance) FROM accounts;

-- Generate report...

COMMIT;
```

---

### Common Issues and Solutions

#### **Issue 1: Unnecessary Table Locks Blocking Operations**

**❌ Problem:**
```sql
-- Old code: Explicit table lock
BEGIN;

LOCK TABLE products IN EXCLUSIVE MODE;

UPDATE products SET price = 9.99 WHERE id = 101;
-- Only updating 1 row but locked entire table!

COMMIT;
-- Blocked all other operations for duration ❌
```

**✅ Solution:**
```sql
-- Use row-level locking (automatic)
BEGIN;

UPDATE products SET price = 9.99 WHERE id = 101;
-- Only locks row 101, high concurrency ✅

COMMIT;
```

#### **Issue 2: ALTER TABLE Blocking Production**

**❌ Problem:**
```sql
-- Adding column (ACCESS EXCLUSIVE lock)
ALTER TABLE products ADD COLUMN category VARCHAR(50);
-- Takes 30 seconds on large table
-- Blocks ALL queries for 30 seconds! ❌
```

**✅ Solution 1: Add with Default Later**
```sql
-- Step 1: Add column without default (fast, no rewrite)
ALTER TABLE products ADD COLUMN category VARCHAR(50);
-- ~1ms, minimal blocking ✅

-- Step 2: Set default for new rows only
ALTER TABLE products ALTER COLUMN category SET DEFAULT 'General';

-- Step 3: Backfill in batches (short transactions)
DO $$
BEGIN
    LOOP
        UPDATE products 
        SET category = 'General'
        WHERE category IS NULL
        AND id IN (
            SELECT id FROM products 
            WHERE category IS NULL 
            LIMIT 1000
        );
        
        EXIT WHEN NOT FOUND;
        
        COMMIT;
        -- Release locks after each batch ✅
    END LOOP;
END $$;
```

**✅ Solution 2: CREATE INDEX CONCURRENTLY**
```sql
-- ❌ Regular index creation (SHARE lock, blocks writes)
CREATE INDEX idx_category ON products(category);

-- ✅ Concurrent index creation (SHARE UPDATE EXCLUSIVE, allows writes)
CREATE INDEX CONCURRENTLY idx_category ON products(category);
-- Takes longer but doesn't block writes ✅
```

#### **Issue 3: Deadlock from Mixed Lock Levels**

**❌ Problem:**
```sql
-- Transaction A
BEGIN;
UPDATE products SET stock = 50 WHERE id = 1;  -- Row lock on row 1
LOCK TABLE categories IN SHARE MODE;  -- Table lock

-- Transaction B
BEGIN;
LOCK TABLE categories IN SHARE MODE;  -- Table lock (OK, SHARE is shareable)
UPDATE products SET stock = 75 WHERE id = 1;  -- Waits for row 1 lock

-- Transaction A
UPDATE categories SET name = 'New' WHERE id = 1;  -- Waits for table lock upgrade

-- DEADLOCK! ❌
```

**✅ Solution: Consistent Lock Ordering**
```sql
-- Always lock in same order: tables first, then rows
BEGIN;

-- Lock tables first
LOCK TABLE categories IN SHARE MODE;

-- Then lock rows
UPDATE products SET stock = 50 WHERE id = 1;

COMMIT;
```

---

### Lock Monitoring

#### **Check Lock Types:**

```sql
-- Show table locks
SELECT 
    l.locktype,
    l.relation::regclass AS table_name,
    l.mode,
    l.granted,
    a.pid,
    a.usename,
    a.query
FROM pg_locks l
JOIN pg_stat_activity a ON l.pid = a.pid
WHERE l.locktype = 'relation'  -- Table locks
ORDER BY l.relation;

-- Show row locks
SELECT 
    l.locktype,
    l.relation::regclass AS table_name,
    l.page,
    l.tuple AS row_num,
    l.mode,
    l.granted,
    a.query
FROM pg_locks l
JOIN pg_stat_activity a ON l.pid = a.pid
WHERE l.locktype = 'tuple'  -- Row locks
ORDER BY l.relation, l.page, l.tuple;
```

#### **Count Locks by Level:**

```sql
SELECT 
    locktype,
    COUNT(*) AS count,
    SUM(CASE WHEN granted THEN 1 ELSE 0 END) AS granted,
    SUM(CASE WHEN NOT granted THEN 1 ELSE 0 END) AS waiting
FROM pg_locks
GROUP BY locktype
ORDER BY count DESC;
```

**Example Output:**
```
  locktype   | count | granted | waiting
-------------+-------+---------+---------
 relation    |   523 |     523 |       0
 transactionid|  342 |     342 |       0
 tuple       |    15 |      12 |       3  ← 3 transactions waiting on row locks
 virtualxid  |   342 |     342 |       0
```

---

### Real-World Example: E-Commerce Platform

**Scenario:**
```
100,000 products in database
1,000 concurrent users
Mix of operations:
- 90% updates to individual products (price, stock)
- 5% bulk price updates (entire categories)
- 5% DDL operations (add columns, indexes)
```

**Strategy 1: All Row-Level Locks (Inefficient for Bulk)**
```sql
-- Individual updates (good)
UPDATE products SET stock = 45 WHERE id = 101;  -- Row lock ✅

-- Bulk update (inefficient)
UPDATE products SET price = price * 1.1 WHERE category = 'Electronics';
-- Locks 10,000 rows individually
-- High memory overhead (1MB for locks)
-- Slower due to lock management

Results:
- Individual updates: 5,200 TPS ✅
- Bulk updates: 450 TPS ❌ (slow)
- Memory: High lock overhead
```

**Strategy 2: Mixed Approach (Optimal)**
```sql
-- Individual updates: Row locks (automatic)
UPDATE products SET stock = 45 WHERE id = 101;  -- Row lock ✅

-- Bulk updates: Explicit table lock
BEGIN;
LOCK TABLE products IN EXCLUSIVE MODE;
UPDATE products SET price = price * 1.1 WHERE category = 'Electronics';
COMMIT;
-- Faster, lower memory ✅

-- DDL: Table lock (automatic)
CREATE INDEX CONCURRENTLY idx_category ON products(category);
-- SHARE UPDATE EXCLUSIVE (allows reads/writes) ✅

Results:
- Individual updates: 5,200 TPS ✅
- Bulk updates: 1,200 TPS ✅ (2.6x faster)
- Memory: Optimal
- Downtime: Minimal (CONCURRENTLY)
```

---

### Summary Table

| **Aspect** | **Row-Level** | **Table-Level** |
|------------|--------------|----------------|
| **Best for** | OLTP, individual updates | DDL, bulk operations |
| **Concurrency** | High (5,200 TPS) | Low (450 TPS) |
| **Overhead** | Higher (per row) | Lower (single lock) |
| **Blocking** | Only same row | Entire table |
| **Memory** | Higher for many rows | Minimal |
| **Default for** | INSERT, UPDATE, DELETE | ALTER, DROP, TRUNCATE |
| **When to use** | Most operations (95%) | Bulk updates, DDL (5%) |

---

### Interview Tips

- **Junior**: "Row-level locks lock individual rows, allowing high concurrency. Table-level locks lock entire table, used for DDL operations. Row locks are automatic for DML (INSERT/UPDATE/DELETE), table locks for DDL (ALTER/DROP). Row locks better for OLTP, table locks better for bulk operations."

- **Mid-Level**: "Used row-level locks (FOR UPDATE) for concurrent user updates achieving 5,200 TPS. Each transaction locks only specific rows, allowing other transactions to work on different rows simultaneously. For bulk updates, used explicit table lock (LOCK TABLE IN EXCLUSIVE MODE) reducing lock overhead from 1MB to single lock, improving throughput from 450 TPS to 1,200 TPS. Used CREATE INDEX CONCURRENTLY for adding indexes without blocking writes."

- **Senior**: "Implemented adaptive locking strategy: row-level locks for 95% of OLTP operations (5,200 TPS), table-level locks for 5% bulk operations (1,200 TPS). Identified unnecessary explicit table locks blocking production, replaced with automatic row locks improving concurrency 10x. Optimized DDL operations using CONCURRENTLY option (CREATE INDEX, REINDEX) preventing production downtime. Built monitoring showing lock distribution (523 table locks, 15 row locks) and wait times, alerting when waiting locks > 10."

- **Advanced**: "Designed hierarchical locking system analyzing query patterns to automatically select lock granularity: row-level for < 1000 rows (high concurrency), table-level for > 10K rows (lower overhead). Implemented dynamic lock escalation monitoring row lock count and upgrading to table lock when threshold exceeded (1000+ locks), reducing memory from 100MB to 1KB. Built lock contention analyzer identifying hot rows and suggesting sharding strategies. Created zero-downtime DDL framework using CREATE TABLE + atomic swap for large ALTER operations avoiding ACCESS EXCLUSIVE locks. Reduced lock wait time from 2200ms to 15ms average while maintaining 99.99% concurrency for 100K daily transactions."

</details>

62. What is a deadlock and how do you prevent it?

<details>
<summary><strong>Answer</strong></summary>

### What is a Deadlock?

A **deadlock** occurs when two or more transactions are waiting for each other to release locks, creating a circular dependency where none can proceed.

**Classic Example:**
```
Transaction 1:        Transaction 2:
─────────────        ─────────────
Locks Row A          Locks Row B
  ↓                    ↓
Waits for Row B      Waits for Row A
  ↓                    ↓
💥 DEADLOCK: Both wait forever, neither can proceed
```

**PostgreSQL's Response:**
- Automatically detects deadlocks after `deadlock_timeout` (default: 1 second)
- Aborts one transaction with error: `ERROR: deadlock detected`
- Other transaction proceeds normally

---

### Deadlock Example

#### **Simple Deadlock:**

```sql
-- Initial state: Two accounts with balances
accounts:
id | balance
---+--------
1  | 1000
2  | 500

-- Transaction 1 (User A transfers $100 from Account 1 to Account 2)
BEGIN;

-- Step 1: Lock and debit Account 1
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- ✅ Success, holds lock on row 1

-- Wait 2 seconds...

-- Transaction 2 (User B transfers $50 from Account 2 to Account 1)
BEGIN;

-- Step 1: Lock and debit Account 2
UPDATE accounts SET balance = balance - 50 WHERE id = 2;
-- ✅ Success, holds lock on row 2

-- Transaction 1 continues
-- Step 2: Credit Account 2
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
-- ⏳ WAITS for lock on row 2 (held by Transaction 2)

-- Transaction 2 continues
-- Step 2: Credit Account 1
UPDATE accounts SET balance = balance + 50 WHERE id = 1;
-- ⏳ WAITS for lock on row 1 (held by Transaction 1)

-- ⏱️ After 1 second (deadlock_timeout), PostgreSQL detects:
-- Transaction 1: Waiting for row 2, held by Transaction 2
-- Transaction 2: Waiting for row 1, held by Transaction 1
-- Circular dependency! 💥

-- PostgreSQL aborts Transaction 2:
-- ERROR: deadlock detected
-- DETAIL: Process 12345 waits for ShareLock on transaction 67890;
--         blocked by process 12346.
--         Process 12346 waits for ShareLock on transaction 67889;
--         blocked by process 12345.
-- HINT: See server log for query details.

-- Transaction 1 proceeds and commits ✅
-- Transaction 2 must retry ♻️
```

---

### Detecting Deadlocks

#### **1. Error Message**

```sql
BEGIN;
UPDATE accounts SET balance = 900 WHERE id = 1;
UPDATE accounts SET balance = 1500 WHERE id = 2;

-- If deadlock occurs:
ERROR: deadlock detected
DETAIL: Process 12345 waits for ShareLock on transaction 67890; blocked by process 12346.
        Process 12346 waits for ShareLock on transaction 67889; blocked by process 12345.
HINT: See server log for query details.
```

#### **2. PostgreSQL Logs**

```log
2025-12-26 10:30:15 UTC ERROR: deadlock detected
2025-12-26 10:30:15 UTC DETAIL: Process 12345 waits for ShareLock on transaction 67890; blocked by process 12346.
    Process 12346 waits for ShareLock on transaction 67889; blocked by process 12345.
2025-12-26 10:30:15 UTC CONTEXT: while updating tuple (0,1) in relation "accounts"
2025-12-26 10:30:15 UTC STATEMENT: UPDATE accounts SET balance = balance + 100 WHERE id = 2
```

#### **3. Monitoring Query**

```sql
-- Check deadlock count
SELECT 
    datname,
    deadlocks,
    deadlocks::float / NULLIF(xact_commit + xact_rollback, 0) * 100 AS deadlock_percentage
FROM pg_stat_database
WHERE datname = current_database();
```

**Output:**
```
  datname  | deadlocks | deadlock_percentage
-----------+-----------+---------------------
 ecommerce |       245 |               0.12%
```

#### **4. Check Current Locks and Waits**

```sql
-- Show blocking locks
SELECT 
    blocked_locks.pid AS blocked_pid,
    blocked_activity.usename AS blocked_user,
    blocking_locks.pid AS blocking_pid,
    blocking_activity.usename AS blocking_user,
    blocked_activity.query AS blocked_query,
    blocking_activity.query AS blocking_query,
    blocked_activity.application_name
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks 
    ON blocking_locks.locktype = blocked_locks.locktype
    AND blocking_locks.database IS NOT DISTINCT FROM blocked_locks.database
    AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
    AND blocking_locks.pid != blocked_locks.pid
JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted;
```

---

### Preventing Deadlocks

### **Solution 1: Consistent Lock Ordering** (Most Important!)

**Problem: Inconsistent Ordering**
```sql
-- Transaction 1: Locks A then B
UPDATE accounts SET balance = balance - 100 WHERE id = 5;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- Transaction 2: Locks B then A (DIFFERENT ORDER!)
UPDATE accounts SET balance = balance - 50 WHERE id = 2;
UPDATE accounts SET balance = balance + 50 WHERE id = 5;

-- Deadlock possible! ❌
```

**Solution: Sort IDs**
```sql
-- Always lock in ascending ID order
def transfer_money(from_id, to_id, amount):
    # Sort IDs to ensure consistent order
    ids = sorted([from_id, to_id])
    
    conn.execute("""
        SELECT * FROM accounts 
        WHERE id IN (%s, %s)
        ORDER BY id  -- Consistent order!
        FOR UPDATE
    """, (ids[0], ids[1]))
    
    # Now safe to update in any order
    conn.execute(
        "UPDATE accounts SET balance = balance - %s WHERE id = %s",
        (amount, from_id)
    )
    conn.execute(
        "UPDATE accounts SET balance = balance + %s WHERE id = %s",
        (amount, to_id)
    )
    
    conn.commit()

# Both transactions now lock in same order:
# Transaction 1: Lock 2, then 5
# Transaction 2: Lock 2, then 5
# No deadlock! ✅
```

**SQL Version:**
```sql
-- Lock rows in consistent order
BEGIN;

SELECT * FROM accounts 
WHERE id IN (5, 2)
ORDER BY id  -- Always: 2, 5
FOR UPDATE;

-- Update
UPDATE accounts SET balance = balance - 100 WHERE id = 5;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;
```

### **Solution 2: Lock Early (SELECT FOR UPDATE)**

**Problem: Late Locking**
```sql
-- Transaction 1
BEGIN;
SELECT balance FROM accounts WHERE id = 1;  -- No lock yet
-- Do some processing...
UPDATE accounts SET balance = 900 WHERE id = 1;  -- Lock acquired late

-- Transaction 2 can interleave, causing deadlock
```

**Solution: Lock Immediately**
```sql
-- Transaction 1
BEGIN;

-- Lock rows immediately
SELECT balance FROM accounts 
WHERE id IN (1, 2)
ORDER BY id
FOR UPDATE;  -- Lock acquired early

-- Now safe to update
UPDATE accounts SET balance = 900 WHERE id = 1;
UPDATE accounts SET balance = 1500 WHERE id = 2;

COMMIT;
```

### **Solution 3: Keep Transactions Short**

**Problem: Long Transaction**
```sql
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- Lock held...

-- Expensive operations while holding lock
send_email_confirmation();  -- 2 seconds
call_external_api();  -- 3 seconds
generate_pdf();  -- 5 seconds

UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;
-- Held locks for 10+ seconds! High deadlock risk ❌
```

**Solution: Short Transaction**
```sql
-- Do expensive work OUTSIDE transaction
email_data = prepare_email();
api_result = call_external_api();
pdf_data = generate_pdf();

-- Quick transaction
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;
-- Held locks for ~10ms! ✅
```

### **Solution 4: Lower Isolation Level**

**Problem: SERIALIZABLE**
```sql
-- SERIALIZABLE: Higher deadlock rate
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

SELECT * FROM accounts WHERE id = 1;
UPDATE accounts SET balance = 900 WHERE id = 1;

COMMIT;
-- More likely to have serialization conflicts
```

**Solution: READ COMMITTED**
```sql
-- READ COMMITTED: Lower deadlock rate
BEGIN;  -- Default: READ COMMITTED

UPDATE accounts SET balance = 900 WHERE id = 1;

COMMIT;
-- Fewer conflicts ✅
```

### **Solution 5: Retry Logic with Exponential Backoff**

**Python Implementation:**
```python
import psycopg2
import time
import random

def transfer_with_retry(from_id, to_id, amount, max_retries=3):
    """
    Transfer money with automatic deadlock retry
    """
    for attempt in range(max_retries):
        try:
            conn = get_connection()
            cursor = conn.cursor()
            
            cursor.execute("BEGIN")
            
            # Sort IDs for consistent locking
            ids = sorted([from_id, to_id])
            
            # Lock both accounts
            cursor.execute("""
                SELECT id, balance FROM accounts 
                WHERE id = ANY(%s)
                ORDER BY id
                FOR UPDATE
            """, (ids,))
            
            accounts = {row[0]: row[1] for row in cursor.fetchall()}
            
            # Validate
            if accounts[from_id] < amount:
                raise InsufficientFunds()
            
            # Transfer
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
                    # Exponential backoff with jitter
                    wait_time = (0.1 * (2 ** attempt)) + random.uniform(0, 0.1)
                    print(f"Deadlock detected, retry {attempt + 1}/{max_retries} after {wait_time:.3f}s")
                    time.sleep(wait_time)
                    continue
                else:
                    raise Exception("Max retries exceeded")
            else:
                raise
        
        except Exception as e:
            conn.rollback()
            raise
        
        finally:
            cursor.close()
            conn.close()
    
    return False

# Usage
success = transfer_with_retry(from_id=5, to_id=2, amount=100)
```

**Node.js Implementation:**
```javascript
async function transferWithRetry(fromId, toId, amount, maxRetries = 3) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    const client = await pool.connect();
    
    try {
      await client.query('BEGIN');
      
      // Sort IDs for consistent locking
      const ids = [fromId, toId].sort((a, b) => a - b);
      
      // Lock both accounts
      const result = await client.query(`
        SELECT id, balance FROM accounts 
        WHERE id = ANY($1::int[])
        ORDER BY id
        FOR UPDATE
      `, [ids]);
      
      const accounts = {};
      result.rows.forEach(row => {
        accounts[row.id] = row.balance;
      });
      
      // Validate
      if (accounts[fromId] < amount) {
        throw new Error('Insufficient funds');
      }
      
      // Transfer
      await client.query(
        'UPDATE accounts SET balance = balance - $1 WHERE id = $2',
        [amount, fromId]
      );
      await client.query(
        'UPDATE accounts SET balance = balance + $1 WHERE id = $2',
        [amount, toId]
      );
      
      await client.query('COMMIT');
      return true;  // Success!
      
    } catch (error) {
      await client.query('ROLLBACK');
      
      if (error.code === '40P01' && attempt < maxRetries - 1) {
        // Deadlock detected - retry
        const waitTime = (100 * Math.pow(2, attempt)) + Math.random() * 100;
        console.log(`Deadlock detected, retry ${attempt + 1}/${maxRetries} after ${waitTime}ms`);
        await new Promise(resolve => setTimeout(resolve, waitTime));
        continue;
      }
      
      throw error;
    } finally {
      client.release();
    }
  }
  
  return false;
}

// Usage
await transferWithRetry(5, 2, 100);
```

### **Solution 6: Use Advisory Locks**

**Problem: Complex Locking Across Tables**
```sql
-- Updating multiple related tables
BEGIN;

UPDATE users SET total_spent = total_spent + 100 WHERE id = 1;
UPDATE orders SET status = 'completed' WHERE user_id = 1;
UPDATE loyalty_points SET points = points + 10 WHERE user_id = 1;

-- Complex dependencies, deadlock risk
```

**Solution: Advisory Lock for Coordination**
```sql
BEGIN;

-- Acquire advisory lock on user_id
SELECT pg_advisory_xact_lock(1);

-- Now can safely update all related tables
UPDATE users SET total_spent = total_spent + 100 WHERE id = 1;
UPDATE orders SET status = 'completed' WHERE user_id = 1;
UPDATE loyalty_points SET points = points + 10 WHERE user_id = 1;

COMMIT;  -- Automatically releases advisory lock

-- All transactions for same user_id are serialized ✅
```

---

### Real-World Example: E-Commerce Order Processing

**Problem:**
```
Flash sale with 10,000 concurrent orders
Each order updates:
1. Product inventory (products table)
2. Customer total_spent (customers table)
3. Order record (orders table)

During peak: 460 deadlocks in 3 minutes
15% order failure rate
Customer complaints about failed payments
```

**Investigation:**
```sql
-- Check deadlock patterns
SELECT deadlocks FROM pg_stat_database WHERE datname = 'ecommerce';
-- Result: 460 deadlocks

-- Analyze queries
-- Found: Orders updating products in different sequences
-- Order A: Updates products [101, 205, 309]
-- Order B: Updates products [205, 101, 450]
-- Different order → deadlock!
```

**Original Code (Problematic):**
```python
def create_order(customer_id, items):
    """
    items = [(product_id, quantity), ...]
    """
    conn.execute("BEGIN")
    
    # Update products (random order based on cart)
    for product_id, quantity in items:
        conn.execute("""
            UPDATE products 
            SET stock = stock - %s 
            WHERE id = %s
        """, (quantity, product_id))
    
    # Update customer
    conn.execute("""
        UPDATE customers 
        SET total_spent = total_spent + %s
        WHERE id = %s
    """, (order_total, customer_id))
    
    # Create order
    conn.execute("""
        INSERT INTO orders (customer_id, total)
        VALUES (%s, %s)
    """, (customer_id, order_total))
    
    conn.commit()

# Problem: Products updated in cart order (inconsistent!)
# Cart A: [101, 205, 309] → locks in this order
# Cart B: [205, 101, 450] → locks in this order
# Deadlock when A holds 101 and B holds 205!
```

**Fixed Code:**
```python
def create_order(customer_id, items):
    """
    items = [(product_id, quantity), ...]
    """
    conn.execute("BEGIN")
    
    # Sort items by product_id for consistent locking
    sorted_items = sorted(items, key=lambda x: x[0])
    
    # Extract product IDs
    product_ids = [item[0] for item in sorted_items]
    
    # Lock all products in consistent order
    conn.execute("""
        SELECT id, stock FROM products 
        WHERE id = ANY(%s)
        ORDER BY id  -- Consistent order!
        FOR UPDATE
    """, (product_ids,))
    
    # Update products in sorted order
    for product_id, quantity in sorted_items:
        conn.execute("""
            UPDATE products 
            SET stock = stock - %s 
            WHERE id = %s AND stock >= %s
        """, (quantity, product_id, quantity))
        
        if cursor.rowcount == 0:
            raise InsufficientStock(product_id)
    
    # Update customer
    conn.execute("""
        UPDATE customers 
        SET total_spent = total_spent + %s
        WHERE id = %s
    """, (order_total, customer_id))
    
    # Create order
    conn.execute("""
        INSERT INTO orders (customer_id, total)
        VALUES (%s, %s)
    """, (customer_id, order_total))
    
    conn.commit()

# All carts now lock products in ascending ID order!
# Cart A: [101, 205, 309] → locks 101, 205, 309
# Cart B: [101, 205, 450] → locks 101, 205, 450
# B waits for A to finish → no deadlock! ✅
```

**Results:**
```
Before:
- Deadlocks: 460 in 3 minutes (peak)
- Order failure rate: 15%
- Average response time: 250ms (including retries)
- Customer satisfaction: Low

After:
- Deadlocks: 0 in 24 hours ✅
- Order failure rate: 0%
- Average response time: 50ms (5x faster, no retries)
- Customer satisfaction: 98%
```

---

### Configuration

```sql
-- Set deadlock detection timeout
ALTER SYSTEM SET deadlock_timeout = '1s';  -- Default

-- Enable lock wait logging
ALTER SYSTEM SET log_lock_waits = on;

-- Log deadlock details
ALTER SYSTEM SET log_min_messages = 'info';

-- Reload configuration
SELECT pg_reload_conf();
```

---

### Best Practices Summary

1. **✅ Always lock in consistent order** (sort by ID)
2. **✅ Lock early with SELECT FOR UPDATE**
3. **✅ Keep transactions short** (< 1 second)
4. **✅ Use READ COMMITTED** (default) unless SERIALIZABLE needed
5. **✅ Implement retry logic** (3-5 retries with exponential backoff)
6. **✅ Use advisory locks** for complex coordination
7. **✅ Monitor deadlock rate** (should be < 0.1%)
8. **✅ Log and analyze** deadlock patterns
9. **✅ Test under load** to find deadlock scenarios
10. **✅ Set lock_timeout** to prevent indefinite waits

---

### Summary

**Deadlock Causes:**
- Inconsistent lock ordering
- Late locking (lock during UPDATE instead of SELECT FOR UPDATE)
- Long transactions
- High isolation levels
- Complex multi-table updates

**Prevention:**
- **Primary**: Consistent lock ordering (sort IDs)
- Lock early (FOR UPDATE)
- Short transactions
- Retry logic
- Advisory locks for coordination

**Detection:**
- ERROR: deadlock detected
- PostgreSQL logs
- pg_stat_database (deadlock count)
- pg_locks (current waits)

---

### Interview Tips

- **Junior**: "Deadlock occurs when two transactions wait for each other's locks creating circular dependency. PostgreSQL detects after 1 second and aborts one transaction. Prevent by locking rows in consistent order using ORDER BY id in SELECT FOR UPDATE."

- **Mid-Level**: "Encountered deadlocks in money transfer system where Transaction A locked account 1 then 2, Transaction B locked 2 then 1. Fixed by sorting account IDs before locking ensuring consistent order. Implemented retry logic with exponential backoff handling deadlock errors (code 40P01). Reduced deadlock rate from 5% to 0.1%."

- **Senior**: "Eliminated 460 deadlocks in e-commerce flash sale by implementing consistent product locking order. Original code updated products in cart order causing inconsistent locking patterns. Solution: sorted product IDs before locking with SELECT FOR UPDATE ORDER BY id. Combined with early locking and short transactions (50ms vs 250ms). Implemented monitoring tracking deadlock percentage (0.12% → 0%), built retry framework with adaptive backoff. Zero deadlocks in 24-hour peak period after fix."

- **Advanced**: "Designed deadlock prevention framework analyzing transaction patterns and automatically detecting potential deadlock scenarios. Implemented predictive lock ordering using historical contention data and graph analysis identifying cyclic dependencies before execution. Built automatic lock hierarchy validator ensuring all application code follows consistent ordering rules. Used advisory locks for complex cross-table coordination reducing multi-table deadlocks 90%. Created real-time deadlock visualization showing transaction wait graphs and suggesting query reordering. Reduced deadlock rate from 0.12% to 0.001% while maintaining 99.99% transaction success rate for 100K daily transactions."

</details>

63. What is two-phase commit (2PC)?

<details>
<summary><strong>Answer</strong></summary>

**Two-Phase Commit (2PC)** is a distributed transaction protocol that ensures atomicity across multiple databases. It works in two phases:

**Phase 1 - Prepare:** The coordinator asks all participating databases if they're ready to commit. Each database locks resources and responds YES or NO.

**Phase 2 - Commit/Rollback:** If all databases vote YES, the coordinator sends COMMIT to all. If any votes NO, the coordinator sends ROLLBACK to all.

```sql
-- Enable 2PC
max_prepared_transactions = 100  -- in postgresql.conf

-- Database A
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
PREPARE TRANSACTION 'tx_001';  -- Phase 1: Vote YES

-- Database B
BEGIN;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
PREPARE TRANSACTION 'tx_001';  -- Phase 1: Vote YES

-- Coordinator: All voted YES, proceed with Phase 2
-- Database A
COMMIT PREPARED 'tx_001';  -- Phase 2: Commit

-- Database B
COMMIT PREPARED 'tx_001';  -- Phase 2: Commit
```

**Problems:**
- **Blocking:** Locks held during PREPARE phase
- **Coordinator failure:** If coordinator crashes, databases stuck in PREPARED state
- **Performance:** 5-8x slower than single-database transactions

**When to use:** Financial systems requiring strict consistency across databases, infrequent distributed transactions.

</details>

64. How does MVCC work in PostgreSQL?

<details>
<summary><strong>Answer</strong></summary>

**MVCC (Multi-Version Concurrency Control)** allows PostgreSQL to handle concurrent transactions without locking by creating multiple versions of each row.

**How it works:**
- Every row has hidden columns: `xmin` (transaction that created it) and `xmax` (transaction that deleted it)
- UPDATE creates a **new row version** instead of modifying in-place
- Each transaction sees a consistent **snapshot** based on transaction IDs
- Old versions are cleaned up by VACUUM

```sql
-- Initial state
SELECT ctid, xmin, xmax, * FROM accounts;
-- ctid  | xmin | xmax | id | balance
-- (0,1) | 1001 |    0 |  1 | 1000

-- Transaction A (ID: 1005)
BEGIN;
UPDATE accounts SET balance = 900 WHERE id = 1;

-- Physical storage now has TWO versions:
-- (0,1) | 1001 | 1005 |  1 | 1000  (old version, marked deleted by 1005)
-- (0,2) | 1005 |    0 |  1 |  900  (new version, created by 1005)

COMMIT;

-- Transaction B (started before 1005)
-- Still sees balance = 1000 (old version)

-- Transaction C (started after 1005)
-- Sees balance = 900 (new version)
```

**Benefits:**
- **Readers never block writers**
- **Writers never block readers**
- Each transaction sees consistent snapshot
- High concurrency

**Costs:**
- Multiple row versions consume storage
- Requires VACUUM to clean up dead tuples
- Transaction ID wraparound management

</details>

65. What is a savepoint in a transaction?

<details>
<summary><strong>Answer</strong></summary>

A **savepoint** is a named checkpoint within a transaction that allows you to rollback to that point without aborting the entire transaction.

**Use case:** When you want to keep some changes but undo others within the same transaction.

```sql
BEGIN;

-- Step 1: Debit account
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- balance: 900

SAVEPOINT sp1;  -- Create savepoint

-- Step 2: Try to credit account (might fail)
UPDATE accounts SET balance = balance + 100 WHERE id = 999;  
-- ERROR: Account 999 doesn't exist

-- Rollback to savepoint (keeps Step 1)
ROLLBACK TO SAVEPOINT sp1;

-- Try different account
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
-- Success!

COMMIT;  -- Both Step 1 and corrected Step 2 committed
```

**Commands:**
```sql
SAVEPOINT name;              -- Create savepoint
ROLLBACK TO SAVEPOINT name;  -- Rollback to savepoint
RELEASE SAVEPOINT name;      -- Delete savepoint (optional)
```

**Real-world example:**
```sql
BEGIN;

-- Insert order
INSERT INTO orders (user_id, total) VALUES (123, 99.99);
SAVEPOINT after_order;

-- Try to apply discount code
UPDATE discount_codes SET used = true WHERE code = 'SAVE20';
-- If discount invalid or expired:
ROLLBACK TO SAVEPOINT after_order;
-- Order still inserted, discount not applied

-- Proceed without discount
COMMIT;
```

**Benefits:**
- Partial rollback without losing entire transaction
- Error recovery within complex transactions
- Retry logic for specific steps

**Note:** Savepoints are transaction-scoped and released on COMMIT or full ROLLBACK.

</details>