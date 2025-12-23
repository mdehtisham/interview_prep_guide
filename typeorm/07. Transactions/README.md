## **Transactions**

<details>
<summary>96. What are transactions in TypeORM?</summary>

### Answer:

**Transactions** in TypeORM ensure that multiple database operations execute as a single atomic unit - either all operations succeed together, or all fail together. This prevents data inconsistencies when performing related operations.

#### **ACID Properties:**
- **Atomicity**: All operations complete or none do
- **Consistency**: Database remains in valid state
- **Isolation**: Concurrent transactions don't interfere
- **Durability**: Committed changes persist permanently

#### **When to Use Transactions:**
```typescript
// ❌ Without transaction - risky
async transferMoney(fromId: number, toId: number, amount: number) {
  await accountRepo.decrement({ id: fromId }, 'balance', amount);
  // If this fails, money is lost!
  await accountRepo.increment({ id: toId }, 'balance', amount);
}

// ✅ With transaction - safe
async transferMoney(fromId: number, toId: number, amount: number) {
  await dataSource.transaction(async (manager) => {
    await manager.decrement(Account, { id: fromId }, 'balance', amount);
    await manager.increment(Account, { id: toId }, 'balance', amount);
    // If any operation fails, both are rolled back
  });
}
```

#### **Real-World Use Cases:**
1. **E-commerce Order Processing:**
   - Deduct inventory
   - Process payment
   - Create order record
   - Send confirmation email log

2. **Banking Operations:**
   - Transfer funds between accounts
   - Record transaction history
   - Update balance sheets

3. **User Registration:**
   - Create user account
   - Create profile
   - Assign default roles
   - Send welcome email log

#### **Key Benefits:**
- **Data Integrity**: Prevents partial updates
- **Consistency**: All related data stays in sync
- **Error Recovery**: Automatic rollback on failures
- **Concurrent Safety**: Handles multiple users safely

#### **Interview Tips:**
- Explain ACID properties with examples
- Mention the difference between database-level and application-level transactions
- Discuss when NOT to use transactions (read-only operations, single operations)
- Highlight the performance consideration - keep transactions short

</details>

<details>
<summary>97. How do you create a transaction?</summary>

### Answer:

TypeORM provides **two main approaches** to create transactions, each suited for different scenarios.

---

### **Method 1: Using dataSource.transaction() - Recommended**

This is the **high-level, automatic** approach that handles connection management and rollback automatically.

```typescript
import { DataSource } from 'typeorm';
import { User } from './entities/User';
import { Profile } from './entities/Profile';

class UserService {
  constructor(private dataSource: DataSource) {}

  async createUserWithProfile(userData: any, profileData: any) {
    // The transaction method automatically manages everything
    return await this.dataSource.transaction(async (transactionalEntityManager) => {
      // Create user
      const user = transactionalEntityManager.create(User, userData);
      await transactionalEntityManager.save(user);
      
      // Create profile linked to user
      const profile = transactionalEntityManager.create(Profile, {
        ...profileData,
        userId: user.id
      });
      await transactionalEntityManager.save(profile);
      
      // If anything fails here, both operations are rolled back
      return { user, profile };
    });
  }
}
```

**Advantages:**
- ✅ Automatic connection management
- ✅ Automatic rollback on errors
- ✅ Cleaner, less boilerplate code
- ✅ Try-catch not needed (handles internally)

---

### **Method 2: Using QueryRunner - Advanced Control**

Use this when you need **manual control** over the transaction lifecycle.

```typescript
import { DataSource } from 'typeorm';
import { User } from './entities/User';

class UserService {
  constructor(private dataSource: DataSource) {}

  async createUserWithManualControl(userData: any) {
    // Step 1: Create a QueryRunner
    const queryRunner = this.dataSource.createQueryRunner();
    
    // Step 2: Connect to database
    await queryRunner.connect();
    
    // Step 3: Start transaction
    await queryRunner.startTransaction();
    
    try {
      // Perform operations using queryRunner.manager
      const user = queryRunner.manager.create(User, userData);
      await queryRunner.manager.save(user);
      
      // You can also execute raw SQL
      await queryRunner.query(
        'INSERT INTO user_logs (user_id, action) VALUES ($1, $2)',
        [user.id, 'USER_CREATED']
      );
      
      // Step 4: Commit if all succeeded
      await queryRunner.commitTransaction();
      return user;
      
    } catch (error) {
      // Step 5: Rollback on error
      await queryRunner.rollbackTransaction();
      throw error;
      
    } finally {
      // Step 6: Always release the connection
      await queryRunner.release();
    }
  }
}
```

**Advantages:**
- ✅ Full control over transaction lifecycle
- ✅ Can execute raw SQL within transaction
- ✅ Can use the same connection for multiple operations
- ✅ Useful for complex scenarios and migrations

---

### **Comparison Table:**

| Feature | dataSource.transaction() | QueryRunner |
|---------|-------------------------|-------------|
| **Complexity** | Simple | More code |
| **Rollback** | Automatic | Manual |
| **Connection** | Auto-managed | Manual |
| **Raw SQL** | Limited | Full support |
| **Use Case** | Business logic | Advanced scenarios |

---

### **Production Example: Order Processing**

```typescript
async processOrder(orderId: number) {
  await this.dataSource.transaction(async (manager) => {
    // 1. Get order details
    const order = await manager.findOne(Order, { where: { id: orderId } });
    
    // 2. Update inventory
    for (const item of order.items) {
      await manager.decrement(
        Product, 
        { id: item.productId }, 
        'stock', 
        item.quantity
      );
    }
    
    // 3. Update order status
    order.status = 'PROCESSING';
    await manager.save(order);
    
    // 4. Create payment record
    await manager.save(Payment, {
      orderId: order.id,
      amount: order.total,
      status: 'PENDING'
    });
    
    // All operations succeed together or fail together
  });
}
```

---

### **Interview Tips:**
- Start with the simple `transaction()` method example
- Explain when you'd use QueryRunner (raw SQL, migrations, complex logic)
- Mention automatic vs manual rollback handling
- Discuss the importance of releasing connections in QueryRunner

</details>

<details>
<summary>98. What is QueryRunner and how is it used for transactions?</summary>

### Answer:

**QueryRunner** is a low-level API in TypeORM that provides **direct control** over database connections and transactions. It's essentially a wrapper around a single database connection that allows you to manage the transaction lifecycle manually.

---

### **Key Features of QueryRunner:**

1. **Single Connection Management**: Maintains one dedicated database connection
2. **Transaction Control**: Start, commit, and rollback manually
3. **Raw SQL Execution**: Execute any SQL query directly
4. **Entity Manager Access**: Get a transaction-scoped EntityManager
5. **Table Operations**: Create, alter, drop tables programmatically

---

### **QueryRunner Lifecycle:**

```typescript
// 1. Create QueryRunner from DataSource
const queryRunner = dataSource.createQueryRunner();

// 2. Establish connection
await queryRunner.connect();

// 3. Start transaction
await queryRunner.startTransaction();

// 4. Perform operations
// ...

// 5. Commit or Rollback
await queryRunner.commitTransaction();
// OR
await queryRunner.rollbackTransaction();

// 6. Always release connection
await queryRunner.release();
```

---

### **Complete Transaction Example:**

```typescript
import { DataSource } from 'typeorm';
import { User } from './entities/User';
import { Order } from './entities/Order';

class OrderService {
  constructor(private dataSource: DataSource) {}

  async createOrderWithTransaction(userId: number, orderData: any) {
    // Create a QueryRunner instance
    const queryRunner = this.dataSource.createQueryRunner();
    
    try {
      // Establish connection to database
      await queryRunner.connect();
      
      // Begin transaction
      await queryRunner.startTransaction();
      
      // Use queryRunner.manager for all operations
      // This ensures they use the same connection
      
      // 1. Verify user exists
      const user = await queryRunner.manager.findOne(User, {
        where: { id: userId }
      });
      
      if (!user) {
        throw new Error('User not found');
      }
      
      // 2. Create order
      const order = queryRunner.manager.create(Order, {
        userId: user.id,
        ...orderData
      });
      await queryRunner.manager.save(order);
      
      // 3. Execute raw SQL (QueryRunner advantage)
      await queryRunner.query(
        `UPDATE users SET total_orders = total_orders + 1 WHERE id = $1`,
        [userId]
      );
      
      // 4. Update user's last order date
      user.lastOrderDate = new Date();
      await queryRunner.manager.save(user);
      
      // Commit if everything succeeded
      await queryRunner.commitTransaction();
      
      return order;
      
    } catch (error) {
      // Rollback transaction on any error
      await queryRunner.rollbackTransaction();
      throw error;
      
    } finally {
      // Release the connection back to pool
      await queryRunner.release();
    }
  }
}
```

---

### **Advanced Use Case: Batch Operations with Savepoints**

```typescript
async processBatchOrders(orders: any[]) {
  const queryRunner = this.dataSource.createQueryRunner();
  await queryRunner.connect();
  await queryRunner.startTransaction();
  
  try {
    for (const [index, orderData] of orders.entries()) {
      // Create a savepoint before each order
      await queryRunner.query(`SAVEPOINT order_${index}`);
      
      try {
        // Process individual order
        await queryRunner.manager.save(Order, orderData);
        
      } catch (error) {
        // Rollback only this order, not entire batch
        await queryRunner.query(`ROLLBACK TO SAVEPOINT order_${index}`);
        console.log(`Order ${index} failed, continuing...`);
      }
    }
    
    // Commit all successful orders
    await queryRunner.commitTransaction();
    
  } catch (error) {
    await queryRunner.rollbackTransaction();
    throw error;
  } finally {
    await queryRunner.release();
  }
}
```

---

### **When to Use QueryRunner:**

✅ **Use QueryRunner when you need:**
- Execute raw SQL within a transaction
- Multiple operations requiring the same connection
- Fine-grained control over commit/rollback timing
- Database migrations
- Complex operations with savepoints
- Streaming large result sets

❌ **Don't use QueryRunner when:**
- Simple transaction operations (use `transaction()` instead)
- You don't need raw SQL
- You want simpler, cleaner code

---

### **Common Methods:**

```typescript
// Transaction control
await queryRunner.startTransaction();
await queryRunner.commitTransaction();
await queryRunner.rollbackTransaction();

// Execute queries
await queryRunner.query('SELECT * FROM users');
const result = await queryRunner.manager.find(User);

// Table operations (useful in migrations)
await queryRunner.createTable(/* table definition */);
await queryRunner.dropTable('table_name');
await queryRunner.addColumn('table_name', /* column */);

// Connection management
await queryRunner.connect();
await queryRunner.release();
```

---

### **Production Example: Financial Transfer**

```typescript
async transferFunds(fromAccountId: number, toAccountId: number, amount: number) {
  const queryRunner = this.dataSource.createQueryRunner();
  await queryRunner.connect();
  await queryRunner.startTransaction('SERIALIZABLE'); // Highest isolation level
  
  try {
    // Lock accounts for update
    const fromAccount = await queryRunner.query(
      'SELECT * FROM accounts WHERE id = $1 FOR UPDATE',
      [fromAccountId]
    );
    
    const toAccount = await queryRunner.query(
      'SELECT * FROM accounts WHERE id = $1 FOR UPDATE',
      [toAccountId]
    );
    
    // Check sufficient balance
    if (fromAccount[0].balance < amount) {
      throw new Error('Insufficient funds');
    }
    
    // Perform transfer
    await queryRunner.query(
      'UPDATE accounts SET balance = balance - $1 WHERE id = $2',
      [amount, fromAccountId]
    );
    
    await queryRunner.query(
      'UPDATE accounts SET balance = balance + $1 WHERE id = $2',
      [amount, toAccountId]
    );
    
    // Log transaction
    await queryRunner.query(
      'INSERT INTO transactions (from_id, to_id, amount) VALUES ($1, $2, $3)',
      [fromAccountId, toAccountId, amount]
    );
    
    await queryRunner.commitTransaction();
    
  } catch (error) {
    await queryRunner.rollbackTransaction();
    throw error;
  } finally {
    await queryRunner.release();
  }
}
```

---

### **Interview Tips:**
- Explain QueryRunner as "manual transmission" vs transaction() as "automatic"
- Emphasize the importance of `finally` block to release connections
- Discuss the trade-off: more control vs more responsibility
- Mention real use cases: migrations, raw SQL, savepoints
- Show understanding of connection pooling

</details>

<details>
<summary>99. How do you use the transaction decorator?</summary>

### Answer:

TypeORM provides **decorators** for transaction management: `@Transaction()` and `@TransactionManager()`. These decorators inject a transaction-scoped EntityManager into your methods, making transaction handling more elegant in class-based services.

---

### **Basic Decorator Usage:**

```typescript
import { Transaction, TransactionManager, EntityManager } from 'typeorm';
import { User } from './entities/User';
import { Profile } from './entities/Profile';

class UserService {
  
  // Mark method as transactional
  @Transaction()
  async createUserWithProfile(
    userData: any,
    profileData: any,
    // Inject transaction manager
    @TransactionManager() manager: EntityManager
  ) {
    // All operations using this manager are transactional
    const user = manager.create(User, userData);
    await manager.save(user);
    
    const profile = manager.create(Profile, {
      ...profileData,
      userId: user.id
    });
    await manager.save(profile);
    
    // If any error occurs, all operations are rolled back automatically
    return { user, profile };
  }
}
```

---

### **How It Works:**

1. `@Transaction()` decorator wraps the method in a transaction
2. `@TransactionManager()` injects an EntityManager scoped to that transaction
3. All database operations using the injected manager are part of the transaction
4. Automatic rollback on any thrown error
5. Automatic commit when method completes successfully

---

### **Multiple Repository Parameters:**

```typescript
import { Transaction, TransactionRepository } from 'typeorm';
import { Repository } from 'typeorm';
import { User } from './entities/User';
import { Order } from './entities/Order';

class OrderService {
  
  @Transaction()
  async createOrderForUser(
    userId: number,
    orderData: any,
    // Inject transaction-scoped repositories
    @TransactionRepository(User) userRepo: Repository<User>,
    @TransactionRepository(Order) orderRepo: Repository<Order>
  ) {
    // Find user with transactional repository
    const user = await userRepo.findOne({ where: { id: userId } });
    
    if (!user) {
      throw new Error('User not found');
    }
    
    // Create order with transactional repository
    const order = orderRepo.create({
      ...orderData,
      userId: user.id
    });
    await orderRepo.save(order);
    
    // Update user's order count
    user.totalOrders += 1;
    await userRepo.save(user);
    
    return order;
  }
}
```

---

### **Calling Transactional Methods:**

```typescript
// Instantiate service
const userService = new UserService();

// Call the method normally - no transaction setup needed
try {
  const result = await userService.createUserWithProfile(
    { name: 'John', email: 'john@example.com' },
    { bio: 'Software developer' }
  );
  console.log('User created:', result);
} catch (error) {
  console.error('Transaction failed:', error);
  // All operations were rolled back automatically
}
```

---

### **Nested Transactional Methods:**

```typescript
class UserService {
  
  @Transaction()
  async createUser(
    userData: any,
    @TransactionManager() manager: EntityManager
  ) {
    const user = manager.create(User, userData);
    return await manager.save(user);
  }
  
  @Transaction()
  async createUserWithProfile(
    userData: any,
    profileData: any,
    @TransactionManager() manager: EntityManager
  ) {
    // Call another transactional method
    // Both will share the same transaction
    const user = await this.createUser(userData, manager);
    
    const profile = manager.create(Profile, {
      ...profileData,
      userId: user.id
    });
    await manager.save(profile);
    
    return { user, profile };
  }
}
```

---

### **Isolation Level with Decorator:**

```typescript
import { Transaction, TransactionManager, EntityManager } from 'typeorm';

class AccountService {
  
  // Specify isolation level
  @Transaction({ isolation: 'SERIALIZABLE' })
  async transferMoney(
    fromId: number,
    toId: number,
    amount: number,
    @TransactionManager() manager: EntityManager
  ) {
    // High isolation for financial operations
    const fromAccount = await manager.findOne(Account, { where: { id: fromId } });
    const toAccount = await manager.findOne(Account, { where: { id: toId } });
    
    fromAccount.balance -= amount;
    toAccount.balance += amount;
    
    await manager.save([fromAccount, toAccount]);
  }
}
```

---

### **Modern Alternative (Recommended):**

**Note:** Transaction decorators are less commonly used in modern TypeORM development. The explicit `dataSource.transaction()` approach is now preferred because:

✅ Better testability  
✅ More explicit control  
✅ Easier to understand  
✅ No decorator magic  

```typescript
// Modern approach - more explicit
class UserService {
  constructor(private dataSource: DataSource) {}
  
  async createUserWithProfile(userData: any, profileData: any) {
    return await this.dataSource.transaction(async (manager) => {
      const user = manager.create(User, userData);
      await manager.save(user);
      
      const profile = manager.create(Profile, {
        ...profileData,
        userId: user.id
      });
      await manager.save(profile);
      
      return { user, profile };
    });
  }
}
```

---

### **Pros and Cons:**

| Aspect | Decorator Approach | Explicit transaction() |
|--------|-------------------|----------------------|
| **Code Style** | Declarative | Imperative |
| **Verbosity** | Less code | More explicit |
| **Testing** | Harder to mock | Easier to test |
| **Clarity** | "Magic" behavior | Clear transaction scope |
| **Modern Use** | Less common | Recommended |

---

### **Interview Tips:**
- Show both decorator and modern approaches
- Explain that decorators are syntactic sugar over transaction()
- Mention the trend toward explicit transaction blocks
- Discuss testing challenges with decorators
- Note that decorators require experimental decorator support in TypeScript

</details>

<details>
<summary>100. How do you handle transaction rollbacks?</summary>

### Answer:

**Transaction rollbacks** undo all database changes made within a transaction when an error occurs or when explicitly triggered. TypeORM provides both automatic and manual rollback mechanisms.

---

### **Automatic Rollback (Recommended):**

With `dataSource.transaction()`, any thrown error automatically triggers a rollback.

```typescript
import { DataSource } from 'typeorm';
import { User } from './entities/User';
import { Account } from './entities/Account';

class UserService {
  constructor(private dataSource: DataSource) {}
  
  async createUserWithAccount(userData: any, initialBalance: number) {
    try {
      await this.dataSource.transaction(async (manager) => {
        // Create user
        const user = manager.create(User, userData);
        await manager.save(user);
        
        // Validate email format
        if (!userData.email.includes('@')) {
          // Throwing error triggers automatic rollback
          throw new Error('Invalid email format');
        }
        
        // Create account
        const account = manager.create(Account, {
          userId: user.id,
          balance: initialBalance
        });
        await manager.save(account);
        
        // If we reach here, transaction commits
      });
      // Success!
      
    } catch (error) {
      // Transaction already rolled back automatically
      console.error('Transaction failed and rolled back:', error.message);
      throw error;
    }
  }
}
```

**How it works:**
- Any error thrown inside the transaction block triggers rollback
- All changes are undone
- The error is re-thrown for handling
- No manual cleanup needed

---

### **Manual Rollback with QueryRunner:**

For fine-grained control, use QueryRunner to manually trigger rollbacks.

```typescript
async transferFunds(fromId: number, toId: number, amount: number) {
  const queryRunner = this.dataSource.createQueryRunner();
  await queryRunner.connect();
  await queryRunner.startTransaction();
  
  try {
    // Get accounts
    const fromAccount = await queryRunner.manager.findOne(Account, {
      where: { id: fromId }
    });
    const toAccount = await queryRunner.manager.findOne(Account, {
      where: { id: toId }
    });
    
    // Business validation
    if (fromAccount.balance < amount) {
      // Explicitly rollback
      await queryRunner.rollbackTransaction();
      throw new Error('Insufficient funds');
    }
    
    // Perform transfer
    fromAccount.balance -= amount;
    toAccount.balance += amount;
    
    await queryRunner.manager.save([fromAccount, toAccount]);
    
    // Commit if all is good
    await queryRunner.commitTransaction();
    
  } catch (error) {
    // Manual rollback in catch block
    if (queryRunner.isTransactionActive) {
      await queryRunner.rollbackTransaction();
    }
    throw error;
    
  } finally {
    // Always release connection
    await queryRunner.release();
  }
}
```

---

### **Conditional Rollback:**

Sometimes you need to rollback based on business logic, not just errors.

```typescript
async processBulkOrders(orders: any[]) {
  return await this.dataSource.transaction(async (manager) => {
    let successCount = 0;
    const errors = [];
    
    for (const orderData of orders) {
      try {
        // Process each order
        const order = manager.create(Order, orderData);
        await manager.save(order);
        successCount++;
        
      } catch (error) {
        errors.push({ order: orderData, error: error.message });
      }
    }
    
    // Conditional rollback based on success rate
    if (successCount < orders.length * 0.8) {
      // Less than 80% success - rollback everything
      throw new Error(`Only ${successCount}/${orders.length} orders succeeded`);
    }
    
    // Otherwise, commit
    return { successCount, errors };
  });
}
```

---

### **Partial Rollback with Savepoints:**

Use savepoints to rollback only part of a transaction.

```typescript
async processOrdersWithSavepoints(orders: any[]) {
  const queryRunner = this.dataSource.createQueryRunner();
  await queryRunner.connect();
  await queryRunner.startTransaction();
  
  const results = [];
  
  try {
    for (const [index, orderData] of orders.entries()) {
      // Create savepoint before each order
      await queryRunner.query(`SAVEPOINT sp_order_${index}`);
      
      try {
        // Try to process order
        const order = await queryRunner.manager.save(Order, orderData);
        results.push({ success: true, order });
        
      } catch (error) {
        // Rollback only to savepoint (not entire transaction)
        await queryRunner.query(`ROLLBACK TO SAVEPOINT sp_order_${index}`);
        results.push({ success: false, error: error.message });
      }
    }
    
    // Commit all successful orders
    await queryRunner.commitTransaction();
    return results;
    
  } catch (error) {
    // Rollback everything
    await queryRunner.rollbackTransaction();
    throw error;
    
  } finally {
    await queryRunner.release();
  }
}
```

---

### **Logging Rollbacks for Debugging:**

```typescript
async createUserWithLogging(userData: any) {
  const queryRunner = this.dataSource.createQueryRunner();
  await queryRunner.connect();
  await queryRunner.startTransaction();
  
  try {
    const user = await queryRunner.manager.save(User, userData);
    
    // Simulate error
    if (user.age < 18) {
      throw new Error('User must be 18 or older');
    }
    
    await queryRunner.commitTransaction();
    console.log(`✅ Transaction committed for user ${user.id}`);
    return user;
    
  } catch (error) {
    await queryRunner.rollbackTransaction();
    
    // Log rollback for monitoring
    console.error('🔄 Transaction rolled back:', {
      reason: error.message,
      userData,
      timestamp: new Date().toISOString()
    });
    
    // Could also log to monitoring service
    // await this.logger.logRollback({ error, userData });
    
    throw error;
    
  } finally {
    await queryRunner.release();
  }
}
```

---

### **Best Practices for Rollbacks:**

1. **Let automatic rollback handle errors** - Don't manually rollback unless needed
2. **Always check transaction state** - Use `queryRunner.isTransactionActive`
3. **Log rollback events** - For debugging and monitoring
4. **Clean up resources** - Release connections in `finally` blocks
5. **Test rollback scenarios** - Verify data integrity after rollbacks
6. **Use savepoints** - For partial rollbacks in batch operations
7. **Document reasons** - Make it clear why a rollback happened

---

### **Testing Rollbacks:**

```typescript
describe('Transaction Rollbacks', () => {
  it('should rollback on validation error', async () => {
    const initialCount = await userRepo.count();
    
    try {
      await userService.createUser({ email: 'invalid' });
    } catch (error) {
      // Expected error
    }
    
    // Verify no user was created (rollback worked)
    const finalCount = await userRepo.count();
    expect(finalCount).toBe(initialCount);
  });
});
```

---

### **Interview Tips:**
- Explain automatic vs manual rollback
- Show examples of both approaches
- Discuss savepoints for partial rollbacks
- Mention importance of logging rollbacks in production
- Emphasize testing rollback scenarios

</details>

<details>
<summary>101. What is the difference between transaction() method and QueryRunner?</summary>

### Answer:

Both `transaction()` method and `QueryRunner` enable transaction management in TypeORM, but they differ significantly in abstraction level, control, and use cases.

---

### **Quick Comparison:**

| Feature | transaction() Method | QueryRunner |
|---------|---------------------|-------------|
| **Abstraction** | High-level, automatic | Low-level, manual |
| **Code Complexity** | Simple, minimal code | More boilerplate |
| **Connection** | Auto-managed | Manual connect/release |
| **Rollback** | Automatic on error | Manual in catch block |
| **Raw SQL** | Limited support | Full raw SQL support |
| **Isolation Level** | Easy to set | Requires manual setup |
| **Use Case** | Business logic | Advanced scenarios |
| **Learning Curve** | Easy | Moderate |

---

### **1. transaction() Method - High-Level Approach**

The `transaction()` method is the **recommended approach** for most use cases. It abstracts away connection management and provides automatic rollback.

```typescript
import { DataSource } from 'typeorm';

class OrderService {
  constructor(private dataSource: DataSource) {}
  
  async createOrder(orderData: any) {
    // Simple and clean
    return await this.dataSource.transaction(async (manager) => {
      // Create order
      const order = manager.create(Order, orderData);
      await manager.save(order);
      
      // Update inventory
      await manager.decrement(
        Product,
        { id: orderData.productId },
        'stock',
        orderData.quantity
      );
      
      return order;
      // Automatic commit if successful
      // Automatic rollback if error thrown
    });
  }
}
```

**Advantages:**
- ✅ Clean, readable code
- ✅ Automatic connection management
- ✅ Automatic rollback on errors
- ✅ No need for try-catch-finally
- ✅ Perfect for business logic

---

### **2. QueryRunner - Low-Level Approach**

QueryRunner provides **manual control** over the entire transaction lifecycle. Use it when you need advanced features or raw SQL.

```typescript
import { DataSource } from 'typeorm';

class OrderService {
  constructor(private dataSource: DataSource) {}
  
  async createOrderWithRunner(orderData: any) {
    // Create QueryRunner
    const queryRunner = this.dataSource.createQueryRunner();
    
    // Manual connection
    await queryRunner.connect();
    
    // Manual transaction start
    await queryRunner.startTransaction();
    
    try {
      // Create order
      const order = queryRunner.manager.create(Order, orderData);
      await queryRunner.manager.save(order);
      
      // Execute raw SQL (QueryRunner advantage)
      await queryRunner.query(
        'UPDATE products SET stock = stock - $1 WHERE id = $2',
        [orderData.quantity, orderData.productId]
      );
      
      // Manual commit
      await queryRunner.commitTransaction();
      return order;
      
    } catch (error) {
      // Manual rollback
      await queryRunner.rollbackTransaction();
      throw error;
      
    } finally {
      // Manual connection release
      await queryRunner.release();
    }
  }
}
```

**Advantages:**
- ✅ Full control over transaction lifecycle
- ✅ Execute raw SQL easily
- ✅ Share same connection across operations
- ✅ Advanced features (savepoints, custom isolation)
- ✅ Essential for migrations

---

### **Side-by-Side Example:**

**Scenario:** Transfer money between accounts

```typescript
// ============================================
// Using transaction() - Simpler
// ============================================
async transferWithTransaction(fromId: number, toId: number, amount: number) {
  await this.dataSource.transaction(async (manager) => {
    await manager.decrement(Account, { id: fromId }, 'balance', amount);
    await manager.increment(Account, { id: toId }, 'balance', amount);
  });
}

// ============================================
// Using QueryRunner - More Control
// ============================================
async transferWithQueryRunner(fromId: number, toId: number, amount: number) {
  const queryRunner = this.dataSource.createQueryRunner();
  await queryRunner.connect();
  await queryRunner.startTransaction('SERIALIZABLE'); // Custom isolation
  
  try {
    // Lock rows with raw SQL
    await queryRunner.query(
      'SELECT * FROM accounts WHERE id IN ($1, $2) FOR UPDATE',
      [fromId, toId]
    );
    
    await queryRunner.manager.decrement(Account, { id: fromId }, 'balance', amount);
    await queryRunner.manager.increment(Account, { id: toId }, 'balance', amount);
    
    await queryRunner.commitTransaction();
  } catch (error) {
    await queryRunner.rollbackTransaction();
    throw error;
  } finally {
    await queryRunner.release();
  }
}
```

---

### **When to Use Each:**

**Use transaction() when:**
- ✅ Writing standard business logic
- ✅ You want clean, maintainable code
- ✅ You don't need raw SQL
- ✅ Automatic rollback is sufficient
- ✅ You're new to TypeORM transactions

**Use QueryRunner when:**
- ✅ Writing database migrations
- ✅ Need to execute raw SQL within transaction
- ✅ Require fine-grained control
- ✅ Working with savepoints
- ✅ Need to share connection across multiple operations
- ✅ Implementing custom transaction logic

---

### **Real Production Example:**

```typescript
class PaymentService {
  constructor(private dataSource: DataSource) {}
  
  // Simple payment - use transaction()
  async processPayment(orderId: number, amount: number) {
    return await this.dataSource.transaction(async (manager) => {
      const order = await manager.findOne(Order, { where: { id: orderId } });
      order.status = 'PAID';
      await manager.save(order);
      
      await manager.save(Payment, {
        orderId,
        amount,
        timestamp: new Date()
      });
    });
  }
  
  // Complex with raw SQL - use QueryRunner
  async generateMonthlyReport(month: number, year: number) {
    const queryRunner = this.dataSource.createQueryRunner();
    await queryRunner.connect();
    await queryRunner.startTransaction();
    
    try {
      // Complex aggregation with raw SQL
      const revenue = await queryRunner.query(`
        SELECT 
          SUM(amount) as total,
          COUNT(*) as count
        FROM payments
        WHERE EXTRACT(MONTH FROM timestamp) = $1
          AND EXTRACT(YEAR FROM timestamp) = $2
      `, [month, year]);
      
      // Insert report
      await queryRunner.query(`
        INSERT INTO monthly_reports (month, year, revenue, payment_count)
        VALUES ($1, $2, $3, $4)
      `, [month, year, revenue[0].total, revenue[0].count]);
      
      await queryRunner.commitTransaction();
      return revenue[0];
      
    } catch (error) {
      await queryRunner.rollbackTransaction();
      throw error;
    } finally {
      await queryRunner.release();
    }
  }
}
```

---

### **Common Mistake:**

```typescript
// ❌ Wrong: Creating QueryRunner when transaction() is enough
async simpleUpdate(userId: number) {
  const queryRunner = this.dataSource.createQueryRunner();
  await queryRunner.connect();
  await queryRunner.startTransaction();
  try {
    await queryRunner.manager.update(User, userId, { lastLogin: new Date() });
    await queryRunner.commitTransaction();
  } catch (error) {
    await queryRunner.rollbackTransaction();
  } finally {
    await queryRunner.release();
  }
}

// ✅ Right: Use transaction() for simple operations
async simpleUpdate(userId: number) {
  await this.dataSource.transaction(async (manager) => {
    await manager.update(User, userId, { lastLogin: new Date() });
  });
}
```

---

### **Interview Tips:**
- Start by explaining both are for transactions but differ in abstraction
- Use the "automatic vs manual transmission" analogy
- Show code examples demonstrating the differences
- Explain when you'd choose one over the other
- Mention that transaction() is preferred for most cases
- Discuss QueryRunner's necessity in migrations

</details>

<details>
<summary>102. How do you implement nested transactions?</summary>

### Answer:

**Nested transactions** allow you to create transaction-like behavior within an existing transaction. In TypeORM, true nested transactions aren't supported, but you can achieve similar functionality using **savepoints** - markers within a transaction that you can rollback to without affecting the entire transaction.

---

### **Understanding Savepoints:**

Savepoints act like "checkpoints" within a transaction:
- Create a savepoint to mark a point in the transaction
- If something fails, rollback to the savepoint (not the entire transaction)
- The outer transaction can still commit successfully
- Useful for batch operations where partial failures are acceptable

---

### **Basic Savepoint Implementation:**

```typescript
import { DataSource } from 'typeorm';
import { User } from './entities/User';
import { Order } from './entities/Order';

class OrderService {
  constructor(private dataSource: DataSource) {}
  
  async processOrdersWithSavepoints(userId: number, orders: any[]) {
    const queryRunner = this.dataSource.createQueryRunner();
    await queryRunner.connect();
    await queryRunner.startTransaction();
    
    const results = [];
    
    try {
      // Verify user exists (outer transaction)
      const user = await queryRunner.manager.findOne(User, {
        where: { id: userId }
      });
      
      if (!user) {
        throw new Error('User not found');
      }
      
      // Process each order with savepoints
      for (const [index, orderData] of orders.entries()) {
        // Create a savepoint before processing each order
        const savepointName = `order_${index}`;
        await queryRunner.query(`SAVEPOINT ${savepointName}`);
        
        try {
          // Attempt to create order (nested operation)
          const order = queryRunner.manager.create(Order, {
            ...orderData,
            userId: user.id
          });
          await queryRunner.manager.save(order);
          
          // Validate order
          if (order.total < 0) {
            throw new Error('Invalid order total');
          }
          
          results.push({ success: true, orderId: order.id });
          
        } catch (error) {
          // Rollback only to the savepoint (this order)
          await queryRunner.query(`ROLLBACK TO SAVEPOINT ${savepointName}`);
          results.push({ 
            success: false, 
            error: error.message,
            orderData 
          });
          
          // Continue processing other orders
          console.log(`Order ${index} failed, continuing with others...`);
        }
      }
      
      // Commit the entire transaction (successful orders persist)
      await queryRunner.commitTransaction();
      return results;
      
    } catch (error) {
      // Rollback entire transaction
      await queryRunner.rollbackTransaction();
      throw error;
      
    } finally {
      await queryRunner.release();
    }
  }
}
```

---

### **Production Use Case: Bulk User Import**

```typescript
async importUsers(userDataArray: any[]) {
  const queryRunner = this.dataSource.createQueryRunner();
  await queryRunner.connect();
  await queryRunner.startTransaction();
  
  const importResults = {
    successful: [],
    failed: []
  };
  
  try {
    for (const [index, userData] of userDataArray.entries()) {
      // Create savepoint for each user
      await queryRunner.query(`SAVEPOINT user_import_${index}`);
      
      try {
        // Import user with profile
        const user = await queryRunner.manager.save(User, {
          name: userData.name,
          email: userData.email
        });
        
        // Create associated profile
        await queryRunner.manager.save(Profile, {
          userId: user.id,
          bio: userData.bio || 'No bio provided'
        });
        
        importResults.successful.push({
          email: userData.email,
          userId: user.id
        });
        
      } catch (error) {
        // Rollback this user only
        await queryRunner.query(`ROLLBACK TO SAVEPOINT user_import_${index}`);
        
        importResults.failed.push({
          email: userData.email,
          reason: error.message
        });
      }
    }
    
    // Commit all successful imports
    await queryRunner.commitTransaction();
    
    return {
      total: userDataArray.length,
      succeeded: importResults.successful.length,
      failed: importResults.failed.length,
      details: importResults
    };
    
  } catch (error) {
    await queryRunner.rollbackTransaction();
    throw error;
  } finally {
    await queryRunner.release();
  }
}
```

---

### **Named Savepoints with Retry Logic:**

```typescript
async processWithRetry(operations: any[]) {
  const queryRunner = this.dataSource.createQueryRunner();
  await queryRunner.connect();
  await queryRunner.startTransaction();
  
  try {
    for (const operation of operations) {
      const savepointName = `sp_${operation.id}`;
      let retries = 3;
      let success = false;
      
      while (retries > 0 && !success) {
        // Create savepoint before attempt
        await queryRunner.query(`SAVEPOINT ${savepointName}`);
        
        try {
          // Execute operation
          await this.executeOperation(queryRunner.manager, operation);
          success = true;
          
        } catch (error) {
          retries--;
          
          // Rollback to savepoint
          await queryRunner.query(`ROLLBACK TO SAVEPOINT ${savepointName}`);
          
          if (retries === 0) {
            console.error(`Operation ${operation.id} failed after 3 attempts`);
            throw error;
          }
          
          // Wait before retry
          await new Promise(resolve => setTimeout(resolve, 100));
        }
      }
    }
    
    await queryRunner.commitTransaction();
    
  } catch (error) {
    await queryRunner.rollbackTransaction();
    throw error;
  } finally {
    await queryRunner.release();
  }
}

private async executeOperation(manager: any, operation: any) {
  // Your operation logic here
  await manager.save(SomeEntity, operation.data);
}
```

---

### **Simulating Nested Transactions:**

```typescript
async transferMoneyWithAudit(fromId: number, toId: number, amount: number) {
  const queryRunner = this.dataSource.createQueryRunner();
  await queryRunner.connect();
  await queryRunner.startTransaction();
  
  try {
    // Main transaction: Transfer money
    const fromAccount = await queryRunner.manager.findOne(Account, {
      where: { id: fromId }
    });
    const toAccount = await queryRunner.manager.findOne(Account, {
      where: { id: toId }
    });
    
    fromAccount.balance -= amount;
    toAccount.balance += amount;
    
    await queryRunner.manager.save([fromAccount, toAccount]);
    
    // Nested operation: Create audit log (with savepoint)
    await queryRunner.query('SAVEPOINT audit_log');
    
    try {
      // Try to create audit entry
      await queryRunner.manager.save(AuditLog, {
        action: 'TRANSFER',
        fromAccountId: fromId,
        toAccountId: toId,
        amount,
        timestamp: new Date()
      });
      
    } catch (error) {
      // If audit fails, rollback only the audit (not the transfer)
      await queryRunner.query('ROLLBACK TO SAVEPOINT audit_log');
      console.warn('Audit log failed but transfer succeeded');
    }
    
    // Commit entire transaction
    await queryRunner.commitTransaction();
    
  } catch (error) {
    await queryRunner.rollbackTransaction();
    throw error;
  } finally {
    await queryRunner.release();
  }
}
```

---

### **Database Support:**

Not all databases support savepoints equally:

| Database | Savepoint Support | Syntax |
|----------|------------------|---------|
| PostgreSQL | ✅ Full | `SAVEPOINT name` |
| MySQL/MariaDB | ✅ Full | `SAVEPOINT name` |
| SQLite | ✅ Full | `SAVEPOINT name` |
| MS SQL Server | ✅ Full | `SAVE TRANSACTION name` |
| Oracle | ✅ Full | `SAVEPOINT name` |

---

### **Best Practices:**

1. **Use descriptive savepoint names**: `sp_user_${id}` instead of `sp1`
2. **Always handle errors**: Wrap savepoint operations in try-catch
3. **Clean up savepoints**: Release unnecessary savepoints with `RELEASE SAVEPOINT`
4. **Limit nesting depth**: Too many savepoints can be confusing
5. **Log failures**: Track which operations failed for debugging
6. **Test thoroughly**: Ensure partial rollbacks work as expected

---

### **Common Pitfall:**

```typescript
// ❌ Wrong: Not using savepoints in batch operations
async processBatch(items: any[]) {
  await this.dataSource.transaction(async (manager) => {
    for (const item of items) {
      await manager.save(Item, item); // If one fails, all rollback!
    }
  });
}

// ✅ Right: Use savepoints for independent operations
async processBatch(items: any[]) {
  const queryRunner = this.dataSource.createQueryRunner();
  await queryRunner.startTransaction();
  
  try {
    for (const [i, item] of items.entries()) {
      await queryRunner.query(`SAVEPOINT item_${i}`);
      try {
        await queryRunner.manager.save(Item, item);
      } catch (error) {
        await queryRunner.query(`ROLLBACK TO SAVEPOINT item_${i}`);
      }
    }
    await queryRunner.commitTransaction();
  } finally {
    await queryRunner.release();
  }
}
```

---

### **Interview Tips:**
- Explain that TypeORM doesn't have true nested transactions
- Describe savepoints as "checkpoints" within a transaction
- Show practical examples: batch imports, partial failures
- Mention database compatibility
- Discuss when to use savepoints vs separate transactions

</details>

<details>
<summary>103. What are isolation levels and how do you set them?</summary>

### Answer:

**Isolation levels** control how transaction integrity is maintained when multiple transactions run concurrently. They define what data a transaction can see when other transactions are modifying the same data.

---

### **The Four Standard Isolation Levels:**

| Level | Dirty Read | Non-Repeatable Read | Phantom Read | Performance |
|-------|-----------|-------------------|-------------|-------------|
| **READ UNCOMMITTED** | ✅ Yes | ✅ Yes | ✅ Yes | Fastest |
| **READ COMMITTED** | ❌ No | ✅ Yes | ✅ Yes | Fast |
| **REPEATABLE READ** | ❌ No | ❌ No | ✅ Yes | Slower |
| **SERIALIZABLE** | ❌ No | ❌ No | ❌ No | Slowest |

---

### **Understanding Read Phenomena:**

**1. Dirty Read:** Reading uncommitted changes from another transaction
```typescript
// Transaction A: Updates balance but hasn't committed
UPDATE accounts SET balance = 500 WHERE id = 1;

// Transaction B: Reads uncommitted value (500)
// If Transaction A rolls back, Transaction B read invalid data!
```

**2. Non-Repeatable Read:** Same query returns different results within a transaction
```typescript
// Transaction A reads twice
SELECT balance FROM accounts WHERE id = 1; // Returns 1000
// Transaction B commits update to 500
SELECT balance FROM accounts WHERE id = 1; // Returns 500 (changed!)
```

**3. Phantom Read:** New rows appear in subsequent queries
```typescript
// Transaction A counts rows
SELECT COUNT(*) FROM accounts WHERE balance > 1000; // Returns 5
// Transaction B inserts new account
SELECT COUNT(*) FROM accounts WHERE balance > 1000; // Returns 6 (phantom!)
```

---

### **Setting Isolation Levels in TypeORM:**

**Method 1: Using transaction() method**

```typescript
import { DataSource } from 'typeorm';

class AccountService {
  constructor(private dataSource: DataSource) {}
  
  async transferMoney(fromId: number, toId: number, amount: number) {
    // Set isolation level as first parameter
    await this.dataSource.transaction('SERIALIZABLE', async (manager) => {
      const fromAccount = await manager.findOne(Account, {
        where: { id: fromId }
      });
      const toAccount = await manager.findOne(Account, {
        where: { id: toId }
      });
      
      fromAccount.balance -= amount;
      toAccount.balance += amount;
      
      await manager.save([fromAccount, toAccount]);
    });
  }
}
```

**Method 2: Using QueryRunner**

```typescript
async processOrder(orderId: number) {
  const queryRunner = this.dataSource.createQueryRunner();
  await queryRunner.connect();
  
  // Set isolation level when starting transaction
  await queryRunner.startTransaction('REPEATABLE READ');
  
  try {
    const order = await queryRunner.manager.findOne(Order, {
      where: { id: orderId }
    });
    
    // Process order...
    await queryRunner.manager.save(order);
    
    await queryRunner.commitTransaction();
    
  } catch (error) {
    await queryRunner.rollbackTransaction();
    throw error;
  } finally {
    await queryRunner.release();
  }
}
```

---

### **Isolation Level Examples by Use Case:**

**1. READ UNCOMMITTED - Rarely Used**
```typescript
// Use only for non-critical analytics where speed matters more than accuracy
async getDashboardStats() {
  await this.dataSource.transaction('READ UNCOMMITTED', async (manager) => {
    // Fast but may read uncommitted data
    const stats = await manager
      .createQueryBuilder(Order, 'order')
      .select('COUNT(*)', 'total')
      .getRawOne();
    
    return stats;
  });
}
```

**2. READ COMMITTED - Default for Most Databases**
```typescript
// Good for general-purpose operations
async getProducts() {
  await this.dataSource.transaction('READ COMMITTED', async (manager) => {
    // Reads only committed data
    // But same query might return different results
    const products = await manager.find(Product, {
      where: { inStock: true }
    });
    
    return products;
  });
}
```

**3. REPEATABLE READ - For Consistent Reads**
```typescript
// Use when you need consistent data throughout transaction
async generateReport(accountId: number) {
  await this.dataSource.transaction('REPEATABLE READ', async (manager) => {
    // First read
    const account = await manager.findOne(Account, {
      where: { id: accountId }
    });
    
    // Some processing...
    await this.doComplexCalculation(account);
    
    // Second read - will see same data as first read
    const accountAgain = await manager.findOne(Account, {
      where: { id: accountId }
    });
    
    // accountAgain.balance === account.balance (guaranteed)
    return this.finalizeReport(account, accountAgain);
  });
}
```

**4. SERIALIZABLE - Strictest Isolation**
```typescript
// Use for critical financial operations
async processPayment(orderId: number, paymentData: any) {
  await this.dataSource.transaction('SERIALIZABLE', async (manager) => {
    // Highest isolation - transactions execute as if serialized
    const order = await manager.findOne(Order, {
      where: { id: orderId }
    });
    
    if (order.status !== 'PENDING') {
      throw new Error('Order already processed');
    }
    
    // No other transaction can modify this order
    order.status = 'PAID';
    await manager.save(order);
    
    await manager.save(Payment, paymentData);
  });
}
```

---

### **Production Example: Banking System**

```typescript
class BankingService {
  constructor(private dataSource: DataSource) {}
  
  // High isolation for money transfers
  async transferFunds(fromId: number, toId: number, amount: number) {
    return await this.dataSource.transaction('SERIALIZABLE', async (manager) => {
      // Lock accounts to prevent concurrent modifications
      const fromAccount = await manager.findOne(Account, {
        where: { id: fromId },
        lock: { mode: 'pessimistic_write' }
      });
      
      const toAccount = await manager.findOne(Account, {
        where: { id: toId },
        lock: { mode: 'pessimistic_write' }
      });
      
      // Validate
      if (fromAccount.balance < amount) {
        throw new Error('Insufficient funds');
      }
      
      // Transfer
      fromAccount.balance -= amount;
      toAccount.balance += amount;
      
      await manager.save([fromAccount, toAccount]);
      
      // Create transaction log
      await manager.save(TransactionLog, {
        fromAccountId: fromId,
        toAccountId: toId,
        amount,
        timestamp: new Date()
      });
      
      return { success: true, newBalance: fromAccount.balance };
    });
  }
  
  // Lower isolation for read-heavy operations
  async getAccountHistory(accountId: number) {
    return await this.dataSource.transaction('READ COMMITTED', async (manager) => {
      return await manager.find(TransactionLog, {
        where: [
          { fromAccountId: accountId },
          { toAccountId: accountId }
        ],
        order: { timestamp: 'DESC' },
        take: 50
      });
    });
  }
}
```

---

### **Database-Specific Default Levels:**

| Database | Default Level |
|----------|--------------|
| PostgreSQL | READ COMMITTED |
| MySQL | REPEATABLE READ |
| SQL Server | READ COMMITTED |
| Oracle | READ COMMITTED |
| SQLite | SERIALIZABLE |

---

### **Trade-offs:**

**Higher Isolation (SERIALIZABLE):**
- ✅ More data consistency
- ✅ Prevents concurrency issues
- ❌ Lower performance
- ❌ More locks and contention
- ❌ Higher chance of deadlocks

**Lower Isolation (READ UNCOMMITTED):**
- ✅ Better performance
- ✅ Less locking
- ❌ Risk of dirty reads
- ❌ Data inconsistencies
- ❌ Not suitable for critical operations

---

### **Best Practices:**

1. **Use appropriate level**: Don't use SERIALIZABLE everywhere
2. **READ COMMITTED for most cases**: Good balance of consistency and performance
3. **SERIALIZABLE for financial operations**: When accuracy is critical
4. **Test with concurrent users**: Verify behavior under load
5. **Monitor deadlocks**: Higher isolation = more deadlock potential
6. **Document your choice**: Explain why you chose a specific level

---

### **Interview Tips:**
- Explain the four levels and what they prevent
- Use real examples: banking (SERIALIZABLE), analytics (READ UNCOMMITTED)
- Discuss the trade-off between consistency and performance
- Show how to set isolation levels in code
- Mention default levels vary by database

</details>

<details>
<summary>104. How do you handle transaction deadlocks?</summary>

### Answer:

**Deadlocks** occur when two or more transactions wait indefinitely for each other to release locks. Transaction A holds Lock 1 and waits for Lock 2, while Transaction B holds Lock 2 and waits for Lock 1 - neither can proceed.

---

### **Understanding Deadlocks:**

```typescript
// Transaction A
BEGIN TRANSACTION;
UPDATE accounts SET balance = 1000 WHERE id = 1; -- Locks row 1
UPDATE accounts SET balance = 2000 WHERE id = 2; -- Waits for row 2

// Transaction B (running simultaneously)
BEGIN TRANSACTION;
UPDATE accounts SET balance = 2000 WHERE id = 2; -- Locks row 2
UPDATE accounts SET balance = 1000 WHERE id = 1; -- Waits for row 1

// Result: DEADLOCK! Both transactions waiting for each other
```

---

### **Strategy 1: Automatic Retry with Exponential Backoff**

```typescript
class TransactionService {
  constructor(private dataSource: DataSource) {}
  
  /**
   * Retry a transaction operation with exponential backoff
   */
  async withRetry<T>(
    operation: () => Promise<T>,
    maxRetries: number = 3,
    baseDelay: number = 100
  ): Promise<T> {
    let lastError: Error;
    
    for (let attempt = 0; attempt < maxRetries; attempt++) {
      try {
        return await operation();
        
      } catch (error) {
        lastError = error;
        
        // Check if it's a deadlock error
        const isDeadlock = this.isDeadlockError(error);
        
        if (!isDeadlock || attempt === maxRetries - 1) {
          // Not a deadlock or last attempt - throw error
          throw error;
        }
        
        // Calculate backoff delay (exponential: 100ms, 200ms, 400ms)
        const delay = baseDelay * Math.pow(2, attempt);
        
        console.warn(`Deadlock detected, retrying in ${delay}ms (attempt ${attempt + 1}/${maxRetries})`);
        
        // Wait before retrying
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
    
    throw lastError!;
  }
  
  /**
   * Check if error is a deadlock
   */
  private isDeadlockError(error: any): boolean {
    // PostgreSQL deadlock code: 40P01
    // MySQL deadlock code: 1213
    // SQL Server deadlock code: 1205
    
    const deadlockCodes = ['40P01', '1213', '1205', 'SQLITE_BUSY'];
    const errorCode = error.code || error.errno?.toString();
    
    return deadlockCodes.includes(errorCode) || 
           error.message?.includes('deadlock');
  }
}
```

**Usage:**
```typescript
async transferMoney(fromId: number, toId: number, amount: number) {
  return await this.withRetry(async () => {
    return await this.dataSource.transaction(async (manager) => {
      await manager.decrement(Account, { id: fromId }, 'balance', amount);
      await manager.increment(Account, { id: toId }, 'balance', amount);
    });
  });
}
```

---

### **Strategy 2: Consistent Resource Ordering**

The most effective deadlock prevention: always access resources in the same order.

```typescript
class AccountService {
  constructor(private dataSource: DataSource) {}
  
  async transferMoney(fromId: number, toId: number, amount: number) {
    // Always lock accounts in ascending ID order
    const [firstId, secondId] = fromId < toId ? [fromId, toId] : [toId, fromId];
    
    await this.dataSource.transaction('SERIALIZABLE', async (manager) => {
      // Lock in consistent order
      const firstAccount = await manager.findOne(Account, {
        where: { id: firstId },
        lock: { mode: 'pessimistic_write' }
      });
      
      const secondAccount = await manager.findOne(Account, {
        where: { id: secondId },
        lock: { mode: 'pessimistic_write' }
      });
      
      // Determine which is from/to
      const fromAccount = firstAccount.id === fromId ? firstAccount : secondAccount;
      const toAccount = firstAccount.id === toId ? firstAccount : secondAccount;
      
      // Perform transfer
      fromAccount.balance -= amount;
      toAccount.balance += amount;
      
      await manager.save([fromAccount, toAccount]);
    });
  }
}
```

---

### **Strategy 3: Keep Transactions Short**

```typescript
// ❌ Bad: Long-running transaction with user interaction
async processOrderBad(orderId: number) {
  await this.dataSource.transaction(async (manager) => {
    const order = await manager.findOne(Order, { where: { id: orderId } });
    
    // Holding locks while doing external work!
    await this.sendEmailToCustomer(order); // Takes 2 seconds
    await this.callPaymentAPI(order); // Takes 3 seconds
    
    order.status = 'PROCESSED';
    await manager.save(order);
  });
}

// ✅ Good: Short transaction, external work outside
async processOrderGood(orderId: number) {
  // Get data first
  const order = await this.orderRepo.findOne({ where: { id: orderId } });
  
  // Do external work outside transaction
  await this.sendEmailToCustomer(order);
  const paymentResult = await this.callPaymentAPI(order);
  
  // Quick transaction to update status
  await this.dataSource.transaction(async (manager) => {
    order.status = 'PROCESSED';
    order.paymentId = paymentResult.id;
    await manager.save(order);
  });
}
```

---

### **Strategy 4: Use Lower Isolation Levels**

```typescript
// For read-heavy operations, use lower isolation
async generateReport() {
  // READ COMMITTED is less prone to deadlocks than SERIALIZABLE
  await this.dataSource.transaction('READ COMMITTED', async (manager) => {
    const orders = await manager.find(Order, {
      where: { status: 'COMPLETED' }
    });
    
    return this.buildReport(orders);
  });
}

// Use SERIALIZABLE only when absolutely necessary
async criticalFinancialOperation() {
  await this.dataSource.transaction('SERIALIZABLE', async (manager) => {
    // Critical operation requiring highest isolation
  });
}
```

---

### **Strategy 5: Implement Timeout**

```typescript
class DeadlockSafeService {
  async executeWithTimeout<T>(
    operation: () => Promise<T>,
    timeoutMs: number = 5000
  ): Promise<T> {
    return Promise.race([
      operation(),
      new Promise<T>((_, reject) => 
        setTimeout(() => reject(new Error('Transaction timeout')), timeoutMs)
      )
    ]);
  }
  
  async transferWithTimeout(fromId: number, toId: number, amount: number) {
    return await this.executeWithTimeout(async () => {
      return await this.dataSource.transaction(async (manager) => {
        // Transaction logic...
        await manager.decrement(Account, { id: fromId }, 'balance', amount);
        await manager.increment(Account, { id: toId }, 'balance', amount);
      });
    }, 3000); // 3 second timeout
  }
}
```

---

### **Production-Grade Deadlock Handler:**

```typescript
class RobustTransactionService {
  constructor(private dataSource: DataSource) {}
  
  async executeWithDeadlockHandling<T>(
    operation: () => Promise<T>,
    options: {
      maxRetries?: number;
      baseDelay?: number;
      timeout?: number;
      onDeadlock?: (attempt: number, error: Error) => void;
    } = {}
  ): Promise<T> {
    const {
      maxRetries = 3,
      baseDelay = 100,
      timeout = 10000,
      onDeadlock
    } = options;
    
    for (let attempt = 0; attempt < maxRetries; attempt++) {
      try {
        // Execute with timeout
        return await Promise.race([
          operation(),
          new Promise<T>((_, reject) =>
            setTimeout(() => reject(new Error('Transaction timeout')), timeout)
          )
        ]);
        
      } catch (error) {
        const isDeadlock = this.isDeadlockError(error);
        const isLastAttempt = attempt === maxRetries - 1;
        
        if (!isDeadlock || isLastAttempt) {
          throw error;
        }
        
        // Callback for monitoring
        if (onDeadlock) {
          onDeadlock(attempt + 1, error);
        }
        
        // Exponential backoff with jitter
        const jitter = Math.random() * 50; // Add randomness
        const delay = baseDelay * Math.pow(2, attempt) + jitter;
        
        console.warn(
          `[Deadlock] Retry ${attempt + 1}/${maxRetries} after ${delay.toFixed(0)}ms`,
          { error: error.message }
        );
        
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
    
    throw new Error('Transaction failed after maximum retries');
  }
  
  private isDeadlockError(error: any): boolean {
    const deadlockIndicators = [
      '40P01',  // PostgreSQL
      '1213',   // MySQL
      '1205',   // SQL Server
      'deadlock detected',
      'deadlock found',
      'SQLITE_BUSY'
    ];
    
    const errorString = JSON.stringify(error).toLowerCase();
    return deadlockIndicators.some(indicator => 
      errorString.includes(indicator.toLowerCase())
    );
  }
}

// Usage
const service = new RobustTransactionService(dataSource);

await service.executeWithDeadlockHandling(
  () => this.dataSource.transaction(async (manager) => {
    // Your transaction logic
  }),
  {
    maxRetries: 5,
    baseDelay: 200,
    timeout: 5000,
    onDeadlock: (attempt, error) => {
      // Log to monitoring service
      logger.warn('Deadlock occurred', { attempt, error });
    }
  }
);
```

---

### **Monitoring and Logging:**

```typescript
// Track deadlock metrics for monitoring
interface DeadlockMetrics {
  totalDeadlocks: number;
  successfulRetries: number;
  failedTransactions: number;
}

class DeadlockMonitor {
  private metrics: DeadlockMetrics = {
    totalDeadlocks: 0,
    successfulRetries: 0,
    failedTransactions: 0
  };
  
  recordDeadlock(resolved: boolean) {
    this.metrics.totalDeadlocks++;
    if (resolved) {
      this.metrics.successfulRetries++;
    } else {
      this.metrics.failedTransactions++;
    }
  }
  
  getMetrics() {
    return { ...this.metrics };
  }
}
```

---

### **Best Practices:**

1. ✅ **Always access resources in the same order**
2. ✅ **Keep transactions short** - minimize lock holding time
3. ✅ **Implement retry logic** - with exponential backoff
4. ✅ **Use appropriate isolation levels** - don't use SERIALIZABLE everywhere
5. ✅ **Set transaction timeouts** - prevent infinite waiting
6. ✅ **Monitor deadlock frequency** - track and alert
7. ✅ **Log deadlock occurrences** - for debugging and analysis
8. ✅ **Test under load** - simulate concurrent users

---

### **Interview Tips:**
- Explain what deadlocks are with a clear example
- Discuss prevention (resource ordering) vs recovery (retry)
- Show retry logic with exponential backoff code
- Mention importance of monitoring deadlocks in production
- Emphasize keeping transactions short

</details>

<details>
<summary>105. What are the best practices for using transactions?</summary>

### Answer:

Following best practices for transactions ensures data integrity, optimal performance, and maintainable code in production applications.

---

### **1. Keep Transactions Short and Focused**

Minimize the time locks are held to reduce contention and deadlock risk.

```typescript
// ❌ Bad: Long transaction with external calls
async processOrderBad(orderId: number) {
  await this.dataSource.transaction(async (manager) => {
    const order = await manager.findOne(Order, { where: { id: orderId } });
    
    // Holding database locks while making external API calls!
    await this.emailService.sendConfirmation(order); // 2 seconds
    await this.paymentGateway.charge(order); // 3 seconds
    await this.inventoryAPI.reserve(order); // 1 second
    
    order.status = 'PROCESSED';
    await manager.save(order);
  });
}

// ✅ Good: Quick transaction, external work outside
async processOrderGood(orderId: number) {
  // Fetch data
  const order = await this.orderRepo.findOne({ where: { id: orderId } });
  
  // External calls outside transaction
  await this.emailService.sendConfirmation(order);
  const payment = await this.paymentGateway.charge(order);
  const reservation = await this.inventoryAPI.reserve(order);
  
  // Quick transaction to update
  await this.dataSource.transaction(async (manager) => {
    order.status = 'PROCESSED';
    order.paymentId = payment.id;
    order.reservationId = reservation.id;
    await manager.save(order);
  });
}
```

**Rule:** Only database operations inside transactions, external work outside.

---

### **2. Use Appropriate Isolation Levels**

Don't default to SERIALIZABLE for everything - choose based on requirements.

```typescript
class TransactionService {
  // Financial operations - high isolation
  async transferMoney(fromId: number, toId: number, amount: number) {
    await this.dataSource.transaction('SERIALIZABLE', async (manager) => {
      // Critical operation requiring strictest isolation
      await manager.decrement(Account, { id: fromId }, 'balance', amount);
      await manager.increment(Account, { id: toId }, 'balance', amount);
    });
  }
  
  // Read operations - lower isolation for better performance
  async getOrders(userId: number) {
    await this.dataSource.transaction('READ COMMITTED', async (manager) => {
      return await manager.find(Order, { where: { userId } });
    });
  }
  
  // Analytics - even lower isolation
  async getDashboardStats() {
    await this.dataSource.transaction('READ UNCOMMITTED', async (manager) => {
      // Fast, approximate data is acceptable
      return await manager.query('SELECT COUNT(*) FROM orders');
    });
  }
}
```

---

### **3. Always Handle Errors Properly**

Ensure proper error handling and logging for debugging.

```typescript
async createUser(userData: any) {
  try {
    return await this.dataSource.transaction(async (manager) => {
      const user = manager.create(User, userData);
      await manager.save(user);
      
      const profile = manager.create(Profile, { userId: user.id });
      await manager.save(profile);
      
      return user;
    });
    
  } catch (error) {
    // Log with context
    console.error('Transaction failed:', {
      operation: 'createUser',
      userData: { email: userData.email }, // Don't log sensitive data
      error: error.message,
      stack: error.stack
    });
    
    // Send to monitoring service
    await this.errorTracker.captureException(error, {
      context: 'user-creation-transaction'
    });
    
    // Re-throw or return appropriate error
    throw new Error('Failed to create user account');
  }
}
```

---

### **4. Implement Retry Logic for Deadlocks**

Handle deadlocks gracefully with automatic retry.

```typescript
class RetryableTransactionService {
  async withRetry<T>(
    operation: () => Promise<T>,
    maxRetries: number = 3
  ): Promise<T> {
    let lastError: Error;
    
    for (let attempt = 0; attempt < maxRetries; attempt++) {
      try {
        return await operation();
      } catch (error) {
        lastError = error;
        
        // Check if deadlock
        const isDeadlock = error.code === '40P01' || 
                          error.code === '1213' ||
                          error.message?.includes('deadlock');
        
        if (!isDeadlock || attempt === maxRetries - 1) {
          throw error;
        }
        
        // Exponential backoff
        const delay = 100 * Math.pow(2, attempt);
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
    
    throw lastError!;
  }
}
```

---

### **5. Access Resources in Consistent Order**

Prevent deadlocks by always locking resources in the same order.

```typescript
// ✅ Good: Always lock in ascending ID order
async transferBetweenAccounts(fromId: number, toId: number, amount: number) {
  // Sort IDs to ensure consistent order
  const [firstId, secondId] = fromId < toId ? [fromId, toId] : [toId, fromId];
  
  await this.dataSource.transaction(async (manager) => {
    // Lock in order
    const first = await manager.findOne(Account, {
      where: { id: firstId },
      lock: { mode: 'pessimistic_write' }
    });
    
    const second = await manager.findOne(Account, {
      where: { id: secondId },
      lock: { mode: 'pessimistic_write' }
    });
    
    // Perform operations
    const from = first.id === fromId ? first : second;
    const to = first.id === toId ? first : second;
    
    from.balance -= amount;
    to.balance += amount;
    
    await manager.save([from, to]);
  });
}
```

---

### **6. Prefer transaction() Over QueryRunner**

Use the high-level API unless you need advanced features.

```typescript
// ✅ Preferred for most cases
async createOrder(orderData: any) {
  return await this.dataSource.transaction(async (manager) => {
    const order = manager.create(Order, orderData);
    await manager.save(order);
    return order;
  });
}

// Only use QueryRunner when you need:
// - Raw SQL execution
// - Savepoints
// - Custom transaction lifecycle
async complexOperation() {
  const queryRunner = this.dataSource.createQueryRunner();
  await queryRunner.connect();
  await queryRunner.startTransaction();
  
  try {
    await queryRunner.query('CUSTOM SQL HERE');
    await queryRunner.commitTransaction();
  } catch (error) {
    await queryRunner.rollbackTransaction();
    throw error;
  } finally {
    await queryRunner.release(); // Always release!
  }
}
```

---

### **7. Test Transaction Scenarios**

Write comprehensive tests including failure scenarios.

```typescript
describe('User Registration Transaction', () => {
  it('should rollback on profile creation failure', async () => {
    const initialUserCount = await userRepo.count();
    const initialProfileCount = await profileRepo.count();
    
    // Mock profile save to fail
    jest.spyOn(profileRepo, 'save').mockRejectedValue(new Error('DB error'));
    
    try {
      await userService.registerUser({
        email: 'test@example.com',
        name: 'Test User'
      });
    } catch (error) {
      // Expected to fail
    }
    
    // Verify rollback worked
    const finalUserCount = await userRepo.count();
    const finalProfileCount = await profileRepo.count();
    
    expect(finalUserCount).toBe(initialUserCount);
    expect(finalProfileCount).toBe(initialProfileCount);
  });
  
  it('should handle concurrent transactions', async () => {
    // Test with multiple concurrent operations
    const promises = Array(10).fill(null).map((_, i) =>
      userService.registerUser({
        email: `user${i}@example.com`,
        name: `User ${i}`
      })
    );
    
    const results = await Promise.all(promises);
    expect(results).toHaveLength(10);
  });
});
```

---

### **8. Set Transaction Timeouts**

Prevent long-running transactions from blocking resources.

```typescript
async executeWithTimeout<T>(
  operation: () => Promise<T>,
  timeoutMs: number = 10000
): Promise<T> {
  return Promise.race([
    operation(),
    new Promise<T>((_, reject) =>
      setTimeout(() => reject(new Error('Transaction timeout')), timeoutMs)
    )
  ]);
}

// Usage
await this.executeWithTimeout(
  () => this.dataSource.transaction(async (manager) => {
    // Transaction logic
  }),
  5000 // 5 second timeout
);
```

---

### **9. Document Transaction Boundaries**

Make transaction scopes clear in your code.

```typescript
class OrderService {
  /**
   * Creates an order with inventory reservation
   * @transactional - Wraps all operations in a transaction
   * @throws InsufficientStockError if product out of stock
   */
  async createOrder(userId: number, orderData: any): Promise<Order> {
    return await this.dataSource.transaction(async (manager) => {
      // Transaction boundary is clear
      const user = await manager.findOne(User, { where: { id: userId } });
      
      const order = manager.create(Order, {
        ...orderData,
        userId: user.id
      });
      await manager.save(order);
      
      await manager.decrement(
        Product,
        { id: orderData.productId },
        'stock',
        orderData.quantity
      );
      
      return order;
    });
  }
}
```

---

### **10. Monitor Transaction Performance**

Track metrics in production.

```typescript
class MonitoredTransactionService {
  async executeWithMetrics<T>(
    name: string,
    operation: () => Promise<T>
  ): Promise<T> {
    const startTime = Date.now();
    
    try {
      const result = await operation();
      
      const duration = Date.now() - startTime;
      
      // Log metrics
      console.log(`Transaction "${name}" completed`, {
        duration,
        status: 'success'
      });
      
      // Send to monitoring service
      this.metrics.recordTransaction(name, duration, 'success');
      
      // Alert if slow
      if (duration > 1000) {
        this.alerts.slowTransaction(name, duration);
      }
      
      return result;
      
    } catch (error) {
      const duration = Date.now() - startTime;
      
      console.error(`Transaction "${name}" failed`, {
        duration,
        status: 'failed',
        error: error.message
      });
      
      this.metrics.recordTransaction(name, duration, 'failed');
      
      throw error;
    }
  }
}

// Usage
await service.executeWithMetrics(
  'user-registration',
  () => this.dataSource.transaction(async (manager) => {
    // Transaction logic
  })
);
```

---

### **Quick Reference Checklist:**

✅ Keep transactions short (minimize lock time)  
✅ Use appropriate isolation levels (not always SERIALIZABLE)  
✅ Handle errors with proper logging  
✅ Implement retry logic for deadlocks  
✅ Access resources in consistent order  
✅ Prefer `transaction()` over QueryRunner  
✅ Test rollback scenarios thoroughly  
✅ Set transaction timeouts  
✅ Document transaction boundaries clearly  
✅ Monitor transaction performance  
✅ Avoid user input/interaction inside transactions  
✅ Release QueryRunner connections in finally blocks  
✅ Don't nest transactions unnecessarily (use savepoints instead)  
✅ Use pessimistic locking for critical operations  
✅ Keep related operations together in one transaction  

---

### **Common Anti-Patterns to Avoid:**

```typescript
// ❌ Don't: Nested transaction() calls
await this.dataSource.transaction(async (manager1) => {
  await this.dataSource.transaction(async (manager2) => {
    // This creates separate transactions!
  });
});

// ❌ Don't: Forget to release QueryRunner
const qr = dataSource.createQueryRunner();
await qr.startTransaction();
await qr.commitTransaction();
// Missing: await qr.release();

// ❌ Don't: Mix transaction managers
await this.dataSource.transaction(async (manager) => {
  await this.userRepo.save(user); // Wrong! Use manager instead
  await manager.save(user); // Correct!
});

// ❌ Don't: Long-running operations in transactions
await this.dataSource.transaction(async (manager) => {
  await this.sendEmail(); // External API call!
  await this.processImage(); // CPU intensive!
  await manager.save(user);
});
```

---

### **Interview Tips:**
- Emphasize "keep it short" as the #1 rule
- Mention the importance of proper error handling
- Discuss deadlock prevention and retry strategies
- Show awareness of testing transaction scenarios
- Talk about monitoring and metrics in production
- Demonstrate understanding of isolation level trade-offs

</details>