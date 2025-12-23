## **Query Builder**

<details>
<summary>61. What is QueryBuilder in TypeORM?</summary>

**QueryBuilder** is TypeORM's powerful API for building complex SQL queries programmatically using TypeScript. It provides a fluent, chainable interface for constructing queries with type safety, supporting SELECT, INSERT, UPDATE, DELETE operations, JOINs, subqueries, and raw SQL expressions.

### Why QueryBuilder?

```typescript
/*
QUERYBUILDER vs REPOSITORY METHODS:

Repository Methods (find, findOne, etc.):
✅ Simple queries
✅ Quick CRUD operations
✅ Basic filtering
❌ Limited for complex queries
❌ Can't do complex JOINs
❌ No subqueries

QueryBuilder:
✅ Complex queries
✅ Multiple JOINs
✅ Subqueries
✅ Custom SELECT columns
✅ Aggregations (COUNT, SUM, AVG)
✅ Raw SQL when needed
✅ Full SQL control with type safety
*/
```

### Basic QueryBuilder Example

```typescript
import { DataSource } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column()
  email: string;

  @Column({ default: true })
  isActive: boolean;

  @Column()
  age: number;

  @OneToMany(() => Post, post => post.author)
  posts: Post[];
}

const dataSource = new DataSource(/* config */);
await dataSource.initialize();

// Simple query with QueryBuilder
const users = await dataSource
  .createQueryBuilder()
  .select('user')
  .from(User, 'user')
  .where('user.isActive = :active', { active: true })
  .andWhere('user.age >= :minAge', { minAge: 18 })
  .orderBy('user.firstName', 'ASC')
  .getMany();

// Equivalent to:
// SELECT * FROM users user
// WHERE user.isActive = true AND user.age >= 18
// ORDER BY user.firstName ASC
```

### QueryBuilder vs Raw SQL

```typescript
// ❌ Raw SQL (no type safety, SQL injection risk)
const users = await dataSource.query(
  `SELECT * FROM users WHERE email = '${email}'` // Dangerous!
);

// ✅ QueryBuilder (type-safe, SQL injection protected)
const users = await dataSource
  .createQueryBuilder()
  .select('user')
  .from(User, 'user')
  .where('user.email = :email', { email })
  .getMany();
```

### Key QueryBuilder Methods

```typescript
// SELECT queries
const users = await dataSource
  .createQueryBuilder()
  .select('user')
  .from(User, 'user')
  .where('user.isActive = :active', { active: true })
  .getMany();

// INSERT
await dataSource
  .createQueryBuilder()
  .insert()
  .into(User)
  .values({ firstName: 'John', email: 'john@example.com' })
  .execute();

// UPDATE
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ isActive: false })
  .where('id = :id', { id: 1 })
  .execute();

// DELETE
await dataSource
  .createQueryBuilder()
  .delete()
  .from(User)
  .where('id = :id', { id: 1 })
  .execute();
```

### Complex Query Example

```typescript
// Find active users with more than 5 posts, including post data
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoinAndSelect('user.posts', 'post')
  .where('user.isActive = :active', { active: true })
  .andWhere((qb) => {
    const subQuery = qb
      .subQuery()
      .select('COUNT(*)')
      .from(Post, 'p')
      .where('p.authorId = user.id')
      .getQuery();
    return `${subQuery} > :minPosts`;
  })
  .setParameter('minPosts', 5)
  .orderBy('user.createdAt', 'DESC')
  .take(10)
  .getMany();

// SQL: SELECT user.*, post.*
//      FROM users user
//      LEFT JOIN posts post ON post.authorId = user.id
//      WHERE user.isActive = true
//        AND (SELECT COUNT(*) FROM posts p WHERE p.authorId = user.id) > 5
//      ORDER BY user.createdAt DESC
//      LIMIT 10
```

### Benefits of QueryBuilder

```typescript
// 1. Type Safety
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.email = :email', { email })
  .getMany(); // Returns User[]

// 2. SQL Injection Prevention
const email = "'; DROP TABLE users; --";
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.email = :email', { email }) // Safe! Parameters are escaped
  .getMany();

// 3. Dynamic Query Building
let query = dataSource
  .createQueryBuilder(User, 'user')
  .where('user.isActive = :active', { active: true });

if (minAge) {
  query = query.andWhere('user.age >= :minAge', { minAge });
}

if (city) {
  query = query.andWhere('user.city = :city', { city });
}

const users = await query.getMany();

// 4. Complex JOINs
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoinAndSelect('user.posts', 'post')
  .leftJoinAndSelect('post.comments', 'comment')
  .leftJoinAndSelect('comment.author', 'commentAuthor')
  .where('user.id = :id', { id: 1 })
  .getOne();

// 5. Aggregations
const result = await dataSource
  .createQueryBuilder(User, 'user')
  .select('COUNT(*)', 'total')
  .addSelect('AVG(user.age)', 'averageAge')
  .addSelect('user.city', 'city')
  .groupBy('user.city')
  .getRawMany();
```

### Real-World Use Cases

```typescript
// Dashboard statistics
const stats = await dataSource
  .createQueryBuilder(User, 'user')
  .select('COUNT(DISTINCT user.id)', 'totalUsers')
  .addSelect('COUNT(DISTINCT post.id)', 'totalPosts')
  .addSelect('AVG(user.age)', 'averageAge')
  .leftJoin('user.posts', 'post')
  .where('user.isActive = :active', { active: true })
  .getRawOne();

// Advanced search with multiple filters
const searchUsers = async (filters: {
  query?: string;
  minAge?: number;
  maxAge?: number;
  hasAvatar?: boolean;
  city?: string[];
}) => {
  let qb = dataSource
    .createQueryBuilder(User, 'user')
    .where('user.isActive = :active', { active: true });

  if (filters.query) {
    qb = qb.andWhere(
      '(user.firstName ILIKE :query OR user.lastName ILIKE :query)',
      { query: `%${filters.query}%` }
    );
  }

  if (filters.minAge || filters.maxAge) {
    qb = qb.andWhere('user.age BETWEEN :minAge AND :maxAge', {
      minAge: filters.minAge || 0,
      maxAge: filters.maxAge || 150,
    });
  }

  if (filters.hasAvatar !== undefined) {
    qb = qb.andWhere(
      filters.hasAvatar
        ? 'user.avatarUrl IS NOT NULL'
        : 'user.avatarUrl IS NULL'
    );
  }

  if (filters.city && filters.city.length > 0) {
    qb = qb.andWhere('user.city IN (:...cities)', { cities: filters.city });
  }

  return qb.orderBy('user.createdAt', 'DESC').getMany();
};
```

**Interview Tip**: Explain that **QueryBuilder** is TypeORM's API for building complex SQL queries programmatically with **type safety** and **SQL injection protection**. It provides a **fluent, chainable interface** (select, where, join, orderBy) that compiles to SQL. Emphasize when to use: Repository methods (find, save) for simple CRUD, **QueryBuilder for complex queries** (multiple JOINs, subqueries, aggregations, dynamic filtering). Highlight benefits: type-safe (returns typed entities), prevents SQL injection (parameterized queries), supports dynamic query building (conditional where clauses), full SQL power (JOINs, subqueries, raw expressions). For production: use QueryBuilder for reporting, dashboards, complex searches, bulk operations; always use parameters (:paramName), build queries dynamically based on filters, optimize with proper indexes.

</details>

<details>
<summary>62. How do you create a QueryBuilder instance?</summary>

There are **three main ways** to create a QueryBuilder instance in TypeORM: from **DataSource**, from **Repository**, or from **EntityManager**. Each approach has different use cases and syntax.

### Method 1: DataSource.createQueryBuilder()

```typescript
import { DataSource } from 'typeorm';

const dataSource = new DataSource({
  type: 'postgres',
  // ... config
});

await dataSource.initialize();

// Create QueryBuilder from DataSource
const users = await dataSource
  .createQueryBuilder()
  .select('user')
  .from(User, 'user')
  .where('user.isActive = :active', { active: true })
  .getMany();

// With entity and alias in one call
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.isActive = :active', { active: true })
  .getMany();
```

### Method 2: Repository.createQueryBuilder()

```typescript
// Get repository first
const userRepository = dataSource.getRepository(User);

// Create QueryBuilder from repository
const users = await userRepository
  .createQueryBuilder('user') // Alias required
  .where('user.isActive = :active', { active: true })
  .orderBy('user.firstName', 'ASC')
  .getMany();

// The entity is already known from repository
// No need to specify from()
```

### Method 3: EntityManager.createQueryBuilder()

```typescript
// Get EntityManager
const manager = dataSource.manager;

// Create QueryBuilder from EntityManager
const users = await manager
  .createQueryBuilder(User, 'user')
  .where('user.isActive = :active', { active: true })
  .getMany();

// Useful in transactions
await dataSource.transaction(async (transactionalManager) => {
  const users = await transactionalManager
    .createQueryBuilder(User, 'user')
    .where('user.id IN (:...ids)', { ids: [1, 2, 3] })
    .getMany();

  // More operations...
});
```

### Comparison of Methods

```typescript
/*
╔═══════════════════════════╤═══════════════════════════╤═══════════════════════════╗
║ METHOD                    │ SYNTAX                    │ USE CASE                  ║
╠═══════════════════════════╪═══════════════════════════╪═══════════════════════════╣
║ DataSource                │ dataSource                │ Global queries            ║
║                           │   .createQueryBuilder()   │ Multiple entities         ║
║                           │   .select('user')         │ No repository needed      ║
║                           │   .from(User, 'user')     │                           ║
╠═══════════════════════════╪═══════════════════════════╪═══════════════════════════╣
║ Repository                │ repository                │ Single entity queries     ║
║                           │   .createQueryBuilder()   │ Entity already known      ║
║                           │   .where(...)             │ Cleaner syntax            ║
╠═══════════════════════════╪═══════════════════════════╪═══════════════════════════╣
║ EntityManager             │ manager                   │ Transactions              ║
║                           │   .createQueryBuilder()   │ Multiple entities         ║
║                           │   .select('user')         │ Manager-based operations  ║
║                           │   .from(User, 'user')     │                           ║
╚═══════════════════════════╧═══════════════════════════╧═══════════════════════════╝
*/
```

### Detailed Examples

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  email: string;

  @Column({ default: true })
  isActive: boolean;
}

// ===== 1. DataSource - Without Entity =====
const users = await dataSource
  .createQueryBuilder()
  .select('user')
  .from(User, 'user')
  .where('user.isActive = :active', { active: true })
  .getMany();

// ===== 2. DataSource - With Entity =====
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.isActive = :active', { active: true })
  .getMany();

// ===== 3. Repository =====
const userRepository = dataSource.getRepository(User);
const users = await userRepository
  .createQueryBuilder('user')
  .where('user.isActive = :active', { active: true })
  .getMany();

// ===== 4. EntityManager =====
const users = await dataSource.manager
  .createQueryBuilder(User, 'user')
  .where('user.isActive = :active', { active: true })
  .getMany();
```

### In NestJS Services

```typescript
@Injectable()
export class UserService {
  constructor(
    private dataSource: DataSource,
    @InjectRepository(User)
    private userRepository: Repository<User>
  ) {}

  // Using DataSource
  async getUsersWithDataSource() {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .where('user.isActive = :active', { active: true })
      .getMany();
  }

  // Using Repository (cleaner)
  async getUsersWithRepository() {
    return this.userRepository
      .createQueryBuilder('user')
      .where('user.isActive = :active', { active: true })
      .getMany();
  }

  // Using EntityManager in transaction
  async transferData(fromId: number, toId: number) {
    await this.dataSource.transaction(async (manager) => {
      const from = await manager
        .createQueryBuilder(User, 'user')
        .where('user.id = :id', { id: fromId })
        .getOne();

      const to = await manager
        .createQueryBuilder(User, 'user')
        .where('user.id = :id', { id: toId })
        .getOne();

      // Transfer logic...
    });
  }
}
```

### Multiple Entities Query

```typescript
// DataSource - Query multiple entities
const results = await dataSource
  .createQueryBuilder()
  .select('user.firstName', 'name')
  .addSelect('post.title', 'postTitle')
  .from(User, 'user')
  .leftJoin(Post, 'post', 'post.authorId = user.id')
  .where('user.isActive = :active', { active: true })
  .getRawMany();

// Repository - Limited to single entity
const userRepository = dataSource.getRepository(User);
const users = await userRepository
  .createQueryBuilder('user')
  .leftJoinAndSelect('user.posts', 'post')
  .where('user.isActive = :active', { active: true })
  .getMany();
```

### Transaction Usage

```typescript
// EntityManager in transaction
await dataSource.transaction(async (transactionalManager) => {
  // All queries in this transaction use the same manager
  const user = await transactionalManager
    .createQueryBuilder(User, 'user')
    .where('user.id = :id', { id: 1 })
    .getOne();

  user.balance += 100;
  await transactionalManager.save(user);

  await transactionalManager
    .createQueryBuilder()
    .insert()
    .into(Transaction)
    .values({ userId: user.id, amount: 100 })
    .execute();

  // If any error occurs, entire transaction rolls back
});
```

### Best Practices

```typescript
// ✅ GOOD: Use Repository.createQueryBuilder for single entity
const userRepository = dataSource.getRepository(User);
const users = await userRepository
  .createQueryBuilder('user')
  .where('user.isActive = :active', { active: true })
  .getMany();

// ✅ GOOD: Use DataSource for multiple entities
const results = await dataSource
  .createQueryBuilder()
  .select('user')
  .addSelect('post')
  .from(User, 'user')
  .leftJoin(Post, 'post', 'post.authorId = user.id')
  .getMany();

// ✅ GOOD: Use EntityManager in transactions
await dataSource.transaction(async (manager) => {
  const qb = manager.createQueryBuilder(User, 'user');
  // ...
});

// ❌ BAD: Mixing DataSource and Repository unnecessarily
const users = await dataSource
  .getRepository(User)
  .createQueryBuilder('user')
  .getMany();
// Just use: userRepository.createQueryBuilder('user')

// ✅ GOOD: Store DataSource/Repository in service
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>
  ) {}

  async findActive() {
    return this.userRepository
      .createQueryBuilder('user')
      .where('user.isActive = :active', { active: true })
      .getMany();
  }
}

// ✅ GOOD: Always provide alias
const users = await userRepository
  .createQueryBuilder('user') // Alias required
  .getMany();

// ❌ BAD: Missing alias
const users = await userRepository
  .createQueryBuilder() // Error! Alias required
  .getMany();
```

**Interview Tip**: Explain **three ways to create QueryBuilder**: **DataSource.createQueryBuilder()** (global queries, multiple entities), **Repository.createQueryBuilder(alias)** (single entity, cleaner), **EntityManager.createQueryBuilder()** (transactions). Emphasize **Repository approach is preferred** for single entity queries (entity already known, no from() needed). Mention DataSource for complex multi-entity queries, EntityManager for transactional queries. Highlight syntax differences: Repository requires alias only, DataSource needs from(Entity, alias), both are chainable. For production: use Repository.createQueryBuilder in services, DataSource for complex JOINs across entities, EntityManager within transactions, always provide alias, inject Repository via DI in NestJS.

</details>

<details>
<summary>63. What is the difference between createQueryBuilder and getRepository().createQueryBuilder()?</summary>

The key difference is **DataSource.createQueryBuilder()** creates a query builder for any entity (requires **from()** clause), while **Repository.createQueryBuilder()** is pre-bound to a specific entity (entity already known, no **from()** needed). Repository approach is more concise for single-entity queries.

### Syntax Comparison

```typescript
import { DataSource } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  email: string;
}

const dataSource = new DataSource(/* config */);
await dataSource.initialize();

// ===== DataSource.createQueryBuilder() =====
// Must specify entity and alias
const users1 = await dataSource
  .createQueryBuilder()
  .select('user')
  .from(User, 'user') // from() is REQUIRED
  .where('user.isActive = :active', { active: true })
  .getMany();

// Or with entity in createQueryBuilder
const users2 = await dataSource
  .createQueryBuilder(User, 'user') // Entity specified here
  .where('user.isActive = :active', { active: true })
  .getMany();

// ===== Repository.createQueryBuilder() =====
// Entity already known from repository
const userRepository = dataSource.getRepository(User);
const users3 = await userRepository
  .createQueryBuilder('user') // Only alias needed
  .where('user.isActive = :active', { active: true })
  .getMany(); // from() not needed!
```

### Detailed Differences

```typescript
/*
╔═══════════════════════════╤═══════════════════════════╤═══════════════════════════╗
║ FEATURE                   │ DataSource                │ Repository                ║
╠═══════════════════════════╪═══════════════════════════╪═══════════════════════════╣
║ Entity Binding            │ ❌ Not pre-bound          │ ✅ Pre-bound to entity    ║
║ from() Required           │ ✅ Yes                    │ ❌ No                     ║
║ Syntax                    │ Verbose                   │ Concise                   ║
║ Use Case                  │ Multiple entities         │ Single entity             ║
║                           │ Complex cross-table       │ Entity-specific queries   ║
║ Flexibility               │ ✅ High (any entity)      │ ⚠️ Limited (one entity)   ║
║ Common Usage              │ Complex reports           │ Standard CRUD with filters║
╚═══════════════════════════╧═══════════════════════════╧═══════════════════════════╝
*/

// DataSource: Flexible but verbose
const result = await dataSource
  .createQueryBuilder()
  .select('user.firstName')
  .addSelect('post.title')
  .from(User, 'user') // Must specify entity
  .leftJoin(Post, 'post', 'post.authorId = user.id')
  .where('user.id = :id', { id: 1 })
  .getRawMany();

// Repository: Concise for single entity
const userRepository = dataSource.getRepository(User);
const users = await userRepository
  .createQueryBuilder('user') // Entity already known
  .leftJoinAndSelect('user.posts', 'post')
  .where('user.id = :id', { id: 1 })
  .getMany();
```

### When to Use Each

```typescript
// ✅ Use Repository.createQueryBuilder - Single Entity Operations
export class UserService {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>
  ) {}

  // Simple entity queries
  async findActiveUsers(): Promise<User[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .where('user.isActive = :active', { active: true })
      .orderBy('user.createdAt', 'DESC')
      .getMany();
  }

  // With relations (same entity)
  async getUserWithPosts(userId: number): Promise<User> {
    return this.userRepository
      .createQueryBuilder('user')
      .leftJoinAndSelect('user.posts', 'post')
      .where('user.id = :id', { id: userId })
      .getOne();
  }
}

// ✅ Use DataSource.createQueryBuilder - Multiple Entities
export class ReportService {
  constructor(private dataSource: DataSource) {}

  // Complex cross-table queries
  async getUserPostStats() {
    return this.dataSource
      .createQueryBuilder()
      .select('user.id', 'userId')
      .addSelect('user.firstName', 'name')
      .addSelect('COUNT(post.id)', 'postCount')
      .addSelect('COUNT(comment.id)', 'commentCount')
      .from(User, 'user')
      .leftJoin(Post, 'post', 'post.authorId = user.id')
      .leftJoin(Comment, 'comment', 'comment.postId = post.id')
      .groupBy('user.id')
      .getRawMany();
  }

  // Multiple unrelated entities
  async getDashboardData() {
    return this.dataSource
      .createQueryBuilder()
      .select('COUNT(DISTINCT user.id)', 'userCount')
      .addSelect('COUNT(DISTINCT post.id)', 'postCount')
      .addSelect('COUNT(DISTINCT comment.id)', 'commentCount')
      .from(User, 'user')
      .leftJoin(Post, 'post', 'post.authorId = user.id')
      .leftJoin(Comment, 'comment', 'comment.authorId = user.id')
      .getRawOne();
  }
}
```

### Real-World Examples

```typescript
// Repository - User-specific operations
export class UserService {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>
  ) {}

  async searchUsers(query: string): Promise<User[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .where('user.firstName ILIKE :query', { query: `%${query}%` })
      .orWhere('user.lastName ILIKE :query', { query: `%${query}%` })
      .orWhere('user.email ILIKE :query', { query: `%${query}%` })
      .orderBy('user.firstName', 'ASC')
      .take(20)
      .getMany();
  }

  async findUsersWithManyPosts(minPosts: number): Promise<User[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .leftJoin('user.posts', 'post')
      .groupBy('user.id')
      .having('COUNT(post.id) >= :minPosts', { minPosts })
      .getMany();
  }
}

// DataSource - Cross-entity analytics
export class AnalyticsService {
  constructor(private dataSource: DataSource) {}

  async getActivityReport(startDate: Date, endDate: Date) {
    return this.dataSource
      .createQueryBuilder()
      .select('DATE(user.createdAt)', 'date')
      .addSelect('COUNT(DISTINCT user.id)', 'newUsers')
      .addSelect('COUNT(DISTINCT post.id)', 'newPosts')
      .addSelect('COUNT(DISTINCT comment.id)', 'newComments')
      .from(User, 'user')
      .leftJoin(
        Post,
        'post',
        'post.authorId = user.id AND post.createdAt BETWEEN :start AND :end'
      )
      .leftJoin(
        Comment,
        'comment',
        'comment.authorId = user.id AND comment.createdAt BETWEEN :start AND :end'
      )
      .where('user.createdAt BETWEEN :start AND :end', {
        start: startDate,
        end: endDate,
      })
      .groupBy('DATE(user.createdAt)')
      .orderBy('date', 'ASC')
      .getRawMany();
  }
}
```

### Migration Pattern

```typescript
// ❌ BAD: Using DataSource when Repository is cleaner
const users = await dataSource
  .createQueryBuilder()
  .select('user')
  .from(User, 'user')
  .where('user.isActive = :active', { active: true })
  .getMany();

// ✅ GOOD: Use Repository for single entity
const userRepository = dataSource.getRepository(User);
const users = await userRepository
  .createQueryBuilder('user')
  .where('user.isActive = :active', { active: true })
  .getMany();

// ✅ GOOD: Use DataSource for multiple entities
const stats = await dataSource
  .createQueryBuilder()
  .select('COUNT(DISTINCT user.id)', 'users')
  .addSelect('COUNT(DISTINCT post.id)', 'posts')
  .from(User, 'user')
  .leftJoin(Post, 'post', 'post.authorId = user.id')
  .getRawOne();
```

### Best Practices

```typescript
// ✅ GOOD: Repository for entity-bound queries
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>
  ) {}

  async findActive() {
    return this.userRepository
      .createQueryBuilder('user')
      .where('user.isActive = :active', { active: true })
      .getMany();
  }
}

// ✅ GOOD: DataSource for cross-entity reports
@Injectable()
export class ReportService {
  constructor(private dataSource: DataSource) {}

  async getStats() {
    return this.dataSource
      .createQueryBuilder()
      .select('user.city', 'city')
      .addSelect('COUNT(post.id)', 'posts')
      .from(User, 'user')
      .leftJoin(Post, 'post', 'post.authorId = user.id')
      .groupBy('user.city')
      .getRawMany();
  }
}

// ✅ GOOD: Inject both when needed
@Injectable()
export class UserService {
  constructor(
    private dataSource: DataSource,
    @InjectRepository(User)
    private userRepository: Repository<User>
  ) {}

  // Use repository for simple queries
  async findById(id: number) {
    return this.userRepository
      .createQueryBuilder('user')
      .where('user.id = :id', { id })
      .getOne();
  }

  // Use dataSource for complex analytics
  async getStatsById(id: number) {
    return this.dataSource
      .createQueryBuilder()
      .select('COUNT(post.id)', 'posts')
      .addSelect('COUNT(comment.id)', 'comments')
      .from(Post, 'post')
      .leftJoin(Comment, 'comment', 'comment.postId = post.id')
      .where('post.authorId = :id', { id })
      .getRawOne();
  }
}
```

**Interview Tip**: Explain **DataSource.createQueryBuilder()** is for queries involving **multiple entities** or complex cross-table operations (requires from() clause), while **Repository.createQueryBuilder(alias)** is for **single entity queries** (entity pre-bound, no from() needed). Emphasize **Repository is preferred** for standard entity operations (cleaner syntax, less verbose). Mention use cases: Repository for CRUD with filters, entity-specific searches, relations on same entity; DataSource for reports, dashboards, cross-entity analytics, complex JOINs. For production: inject Repository via @InjectRepository in services for entity operations, use DataSource for reporting/analytics services, choose based on query complexity.

</details>

<details>
<summary>64. How do you use SELECT with QueryBuilder?</summary>

The **SELECT** clause in QueryBuilder defines which columns to retrieve. You can select entire entities (`.select('alias')`), specific columns (`.select('alias.column')`), or use **addSelect()** to add more columns. QueryBuilder supports column aliases, raw SQL expressions, and computed fields.

### Basic SELECT Operations

```typescript
import { DataSource } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column()
  email: string;

  @Column({ default: true })
  isActive: boolean;

  @Column()
  age: number;
}

const dataSource = new DataSource(/* config */);
await dataSource.initialize();

// 1. Select entire entity (all columns)
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user') // SELECT user.* FROM users user
  .getMany();

// 2. Select specific column
const emails = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.email') // SELECT user.email FROM users user
  .getMany();

// 3. Select multiple columns
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .select(['user.id', 'user.firstName', 'user.email'])
  .getMany();

// 4. Using addSelect() to add more columns
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.id')
  .addSelect('user.firstName')
  .addSelect('user.email')
  .getMany();
```

### SELECT with Column Aliases

```typescript
// Use getRawMany() to get raw results with custom column names
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.id', 'userId')
  .addSelect('user.firstName', 'name')
  .addSelect('user.email', 'emailAddress')
  .getRawMany();

// Result: [{ userId: 1, name: 'John', emailAddress: 'john@example.com' }, ...]

// Compare with getMany() - returns entity instances
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .select(['user.id', 'user.firstName', 'user.email'])
  .getMany();
// Result: [User { id: 1, firstName: 'John', email: 'john@example.com' }, ...]
```

### SELECT with Raw SQL Expressions

```typescript
// Computed columns
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.id', 'id')
  .addSelect('user.firstName', 'firstName')
  .addSelect('user.lastName', 'lastName')
  .addSelect(
    "CONCAT(user.firstName, ' ', user.lastName)",
    'fullName'
  )
  .getRawMany();
// Result: [{ id: 1, firstName: 'John', lastName: 'Doe', fullName: 'John Doe' }]

// Date formatting
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.id', 'id')
  .addSelect("TO_CHAR(user.createdAt, 'YYYY-MM-DD')", 'createdDate')
  .getRawMany();

// Mathematical operations
const products = await dataSource
  .createQueryBuilder(Product, 'product')
  .select('product.name', 'name')
  .addSelect('product.price', 'price')
  .addSelect('product.price * 0.9', 'discountedPrice')
  .getRawMany();
```

### SELECT with Relations

```typescript
@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToOne(() => User, user => user.posts)
  author: User;
}

// Select entity with relations using leftJoinAndSelect
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user') // Main entity
  .leftJoinAndSelect('user.posts', 'post') // Load posts relation
  .getMany();
// Result: User instances with posts array populated

// Select specific columns from relation
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.id', 'userId')
  .addSelect('user.firstName', 'userName')
  .addSelect('post.title', 'postTitle')
  .leftJoin('user.posts', 'post')
  .getRawMany();
// Result: [{ userId: 1, userName: 'John', postTitle: 'First Post' }, ...]
```

### SELECT Distinct

```typescript
// Get distinct values
const cities = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city')
  .distinct(true)
  .getRawMany();

// Distinct with multiple columns
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city')
  .addSelect('user.country')
  .distinct(true)
  .getRawMany();
```

### SELECT with Aggregations

```typescript
// Count users
const count = await dataSource
  .createQueryBuilder(User, 'user')
  .select('COUNT(*)', 'total')
  .getRawOne();
// Result: { total: '150' }

// Multiple aggregations
const stats = await dataSource
  .createQueryBuilder(User, 'user')
  .select('COUNT(*)', 'totalUsers')
  .addSelect('AVG(user.age)', 'averageAge')
  .addSelect('MIN(user.age)', 'minAge')
  .addSelect('MAX(user.age)', 'maxAge')
  .getRawOne();
// Result: { totalUsers: '150', averageAge: '32.5', minAge: '18', maxAge: '75' }

// Group by with aggregations
const cityStats = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city', 'city')
  .addSelect('COUNT(*)', 'userCount')
  .addSelect('AVG(user.age)', 'avgAge')
  .groupBy('user.city')
  .getRawMany();
```

### Real-World Examples

```typescript
// User search with specific fields
export class UserService {
  async searchUsers(query: string) {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .select([
        'user.id',
        'user.firstName',
        'user.lastName',
        'user.email',
        'user.avatarUrl',
      ])
      .where('user.firstName ILIKE :query', { query: `%${query}%` })
      .orWhere('user.lastName ILIKE :query', { query: `%${query}%` })
      .orderBy('user.firstName', 'ASC')
      .take(20)
      .getMany();
  }

  // Dashboard statistics
  async getDashboardStats() {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .select('COUNT(*)', 'totalUsers')
      .addSelect(
        'COUNT(CASE WHEN user.isActive = true THEN 1 END)',
        'activeUsers'
      )
      .addSelect(
        'COUNT(CASE WHEN user.createdAt > NOW() - INTERVAL \'7 days\' THEN 1 END)',
        'newUsersThisWeek'
      )
      .addSelect('AVG(user.age)', 'averageAge')
      .getRawOne();
  }

  // User with computed full name
  async getUsersWithFullName() {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .select('user.id', 'id')
      .addSelect('user.firstName', 'firstName')
      .addSelect('user.lastName', 'lastName')
      .addSelect(
        "CONCAT(user.firstName, ' ', user.lastName)",
        'fullName'
      )
      .addSelect('user.email', 'email')
      .where('user.isActive = :active', { active: true })
      .getRawMany();
  }
}
```

### SELECT Best Practices

```typescript
// ✅ GOOD: Select only needed columns for performance
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .select(['user.id', 'user.firstName', 'user.email'])
  .getMany();

// ❌ BAD: Selecting all columns when only few are needed
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user') // Loads all columns
  .getMany();

// ✅ GOOD: Use getMany() for entities, getRawMany() for custom columns
const entities = await dataSource
  .createQueryBuilder(User, 'user')
  .select(['user.id', 'user.firstName'])
  .getMany(); // Returns User instances

const rawData = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.id', 'userId')
  .addSelect('UPPER(user.firstName)', 'upperName')
  .getRawMany(); // Returns plain objects

// ✅ GOOD: Use addSelect for conditional columns
let query = dataSource
  .createQueryBuilder(User, 'user')
  .select(['user.id', 'user.firstName']);

if (includeEmail) {
  query = query.addSelect('user.email');
}

if (includeAvatar) {
  query = query.addSelect('user.avatarUrl');
}

const users = await query.getMany();

// ✅ GOOD: Use leftJoinAndSelect for relations
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user')
  .leftJoinAndSelect('user.posts', 'post')
  .leftJoinAndSelect('user.profile', 'profile')
  .getMany();

// ❌ BAD: Don't mix select() array syntax with string syntax
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .select(['user.id', 'user.firstName'])
  .select('user.email') // This REPLACES previous select!
  .getMany();
// Use addSelect instead:
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .select(['user.id', 'user.firstName'])
  .addSelect('user.email') // ✅ Adds to selection
  .getMany();
```

**Interview Tip**: Explain **SELECT** in QueryBuilder defines which columns to retrieve. Use `.select('alias')` for entire entity, `.select(['alias.col1', 'alias.col2'])` for specific columns, `.addSelect('alias.col')` to add more columns. Mention **getMany()** returns entity instances (for selected columns), **getRawMany()** returns plain objects (for custom queries with aliases). Emphasize use cases: select specific columns for **performance** (reduce data transfer), use **raw SQL expressions** for computed fields (CONCAT, date formatting), use **aggregations** (COUNT, AVG, SUM) with getRawOne/getRawMany(). For production: select only needed columns, use leftJoinAndSelect for relations, use getRawMany() for custom column aliases, build dynamic selects with addSelect(), always consider performance implications.

</details>

<details>
<summary>65. How do you add WHERE clauses in QueryBuilder?</summary>

The **WHERE** clause in QueryBuilder filters query results. Use `.where()` to set the initial condition, `.andWhere()` to add AND conditions, `.orWhere()` for OR conditions. Always use **parameterized queries** (`:paramName`) to prevent SQL injection.

### Basic WHERE Operations

```typescript
import { DataSource } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  email: string;

  @Column({ default: true })
  isActive: boolean;

  @Column()
  age: number;

  @Column()
  city: string;
}

const dataSource = new DataSource(/* config */);
await dataSource.initialize();

// 1. Simple WHERE with parameter
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.isActive = :active', { active: true })
  .getMany();
// SQL: WHERE user.isActive = true

// 2. Multiple parameters
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.age >= :minAge AND user.age <= :maxAge', {
    minAge: 18,
    maxAge: 65,
  })
  .getMany();
// SQL: WHERE user.age >= 18 AND user.age <= 65

// 3. Object-style WHERE (simple equality)
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where({ isActive: true, city: 'New York' })
  .getMany();
// SQL: WHERE user.isActive = true AND user.city = 'New York'
```

### WHERE with Comparison Operators

```typescript
// Equality
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.city = :city', { city: 'New York' })
  .getMany();

// Inequality
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.city != :city', { city: 'New York' })
  .getMany();

// Greater than / Less than
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.age > :age', { age: 18 })
  .getMany();

const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.age >= :minAge', { minAge: 18 })
  .getMany();

// BETWEEN
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.age BETWEEN :min AND :max', { min: 18, max: 65 })
  .getMany();

// IN clause
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.city IN (:...cities)', {
    cities: ['New York', 'Los Angeles', 'Chicago'],
  })
  .getMany();
// Note: ...cities spreads the array

// NOT IN
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.city NOT IN (:...cities)', {
    cities: ['Banned City 1', 'Banned City 2'],
  })
  .getMany();
```

### WHERE with String Operations

```typescript
// LIKE (case-sensitive)
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.firstName LIKE :name', { name: 'John%' })
  .getMany();
// Matches: John, Johnny, Johannes

// ILIKE (case-insensitive, PostgreSQL)
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.email ILIKE :query', { query: '%@gmail.com' })
  .getMany();

// Contains pattern
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.email LIKE :pattern', { pattern: '%john%' })
  .getMany();

// Starts with
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.firstName LIKE :pattern', { pattern: 'A%' })
  .getMany();

// Ends with
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.email LIKE :pattern', { pattern: '%@gmail.com' })
  .getMany();
```

### WHERE with NULL Checks

```typescript
// IS NULL
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.deletedAt IS NULL')
  .getMany();

// IS NOT NULL
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.avatarUrl IS NOT NULL')
  .getMany();

// Conditional NULL check
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where(
    includeDeleted
      ? '1=1' // Always true (no filter)
      : 'user.deletedAt IS NULL'
  )
  .getMany();
```

### WHERE with Brackets (Complex Conditions)

```typescript
// Using Brackets for grouping
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.isActive = :active', { active: true })
  .andWhere(
    new Brackets((qb) => {
      qb.where('user.age >= :minAge', { minAge: 18 })
        .orWhere('user.role = :role', { role: 'admin' });
    })
  )
  .getMany();
// SQL: WHERE user.isActive = true AND (user.age >= 18 OR user.role = 'admin')

// Nested brackets
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where(
    new Brackets((qb) => {
      qb.where('user.city = :city1', { city1: 'New York' })
        .orWhere('user.city = :city2', { city2: 'Los Angeles' });
    })
  )
  .andWhere(
    new Brackets((qb) => {
      qb.where('user.age >= :minAge', { minAge: 18 })
        .andWhere('user.isActive = :active', { active: true });
    })
  )
  .getMany();
// SQL: WHERE (user.city = 'New York' OR user.city = 'Los Angeles')
//      AND (user.age >= 18 AND user.isActive = true)
```

### Dynamic WHERE Clauses

```typescript
// Building WHERE dynamically based on filters
export class UserService {
  async searchUsers(filters: {
    query?: string;
    minAge?: number;
    maxAge?: number;
    cities?: string[];
    isActive?: boolean;
  }) {
    let qb = this.dataSource.createQueryBuilder(User, 'user');

    // Always start with a base condition or use where('1=1')
    qb = qb.where('1=1');

    if (filters.query) {
      qb = qb.andWhere(
        '(user.firstName ILIKE :query OR user.lastName ILIKE :query OR user.email ILIKE :query)',
        { query: `%${filters.query}%` }
      );
    }

    if (filters.minAge !== undefined) {
      qb = qb.andWhere('user.age >= :minAge', { minAge: filters.minAge });
    }

    if (filters.maxAge !== undefined) {
      qb = qb.andWhere('user.age <= :maxAge', { maxAge: filters.maxAge });
    }

    if (filters.cities && filters.cities.length > 0) {
      qb = qb.andWhere('user.city IN (:...cities)', {
        cities: filters.cities,
      });
    }

    if (filters.isActive !== undefined) {
      qb = qb.andWhere('user.isActive = :active', {
        active: filters.isActive,
      });
    }

    return qb.orderBy('user.createdAt', 'DESC').getMany();
  }
}
```

### WHERE with Subqueries

```typescript
// Users who have posts
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where((qb) => {
    const subQuery = qb
      .subQuery()
      .select('post.authorId')
      .from(Post, 'post')
      .getQuery();
    return `user.id IN ${subQuery}`;
  })
  .getMany();

// Users with more than 5 posts
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where((qb) => {
    const subQuery = qb
      .subQuery()
      .select('COUNT(*)')
      .from(Post, 'post')
      .where('post.authorId = user.id')
      .getQuery();
    return `${subQuery} > :minPosts`;
  })
  .setParameter('minPosts', 5)
  .getMany();
```

### Real-World Examples

```typescript
// Advanced search with multiple filters
async advancedUserSearch({
  query,
  minAge,
  maxAge,
  hasAvatar,
  isVerified,
  registeredAfter,
  cities,
}: SearchFilters) {
  let qb = this.dataSource
    .createQueryBuilder(User, 'user')
    .where('user.isActive = :active', { active: true });

  // Text search
  if (query) {
    qb = qb.andWhere(
      new Brackets((subQb) => {
        subQb
          .where('user.firstName ILIKE :query', { query: `%${query}%` })
          .orWhere('user.lastName ILIKE :query', { query: `%${query}%` })
          .orWhere('user.email ILIKE :query', { query: `%${query}%` });
      })
    );
  }

  // Age range
  if (minAge || maxAge) {
    qb = qb.andWhere('user.age BETWEEN :min AND :max', {
      min: minAge || 0,
      max: maxAge || 150,
    });
  }

  // Avatar filter
  if (hasAvatar !== undefined) {
    qb = qb.andWhere(
      hasAvatar ? 'user.avatarUrl IS NOT NULL' : 'user.avatarUrl IS NULL'
    );
  }

  // Verification status
  if (isVerified !== undefined) {
    qb = qb.andWhere('user.isVerified = :verified', { verified: isVerified });
  }

  // Registration date
  if (registeredAfter) {
    qb = qb.andWhere('user.createdAt >= :date', { date: registeredAfter });
  }

  // Cities filter
  if (cities && cities.length > 0) {
    qb = qb.andWhere('user.city IN (:...cities)', { cities });
  }

  return qb.orderBy('user.createdAt', 'DESC').take(50).getMany();
}
```

### WHERE Best Practices

```typescript
// ✅ GOOD: Always use parameters to prevent SQL injection
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.email = :email', { email })
  .getMany();

// ❌ BAD: String concatenation (SQL injection risk!)
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where(`user.email = '${email}'`) // DANGEROUS!
  .getMany();

// ✅ GOOD: Use Brackets for complex OR conditions
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.isActive = :active', { active: true })
  .andWhere(
    new Brackets((qb) => {
      qb.where('user.role = :role1', { role1: 'admin' })
        .orWhere('user.role = :role2', { role2: 'moderator' });
    })
  )
  .getMany();

// ✅ GOOD: Use ...spread for IN arrays
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.id IN (:...ids)', { ids: [1, 2, 3, 4, 5] })
  .getMany();

// ✅ GOOD: Start with '1=1' for dynamic queries
let qb = dataSource
  .createQueryBuilder(User, 'user')
  .where('1=1'); // Always true base

if (filter1) qb = qb.andWhere('user.col1 = :val', { val: filter1 });
if (filter2) qb = qb.andWhere('user.col2 = :val', { val: filter2 });
```

**Interview Tip**: Explain **WHERE** clause filters query results using `.where()` for initial condition, `.andWhere()` for AND, `.orWhere()` for OR. Emphasize **always use parameterized queries** (`:paramName`) to prevent SQL injection. Mention operators: equality (=, !=), comparison (>, <, >=, <=, BETWEEN), string matching (LIKE, ILIKE), set operations (IN, NOT IN), NULL checks (IS NULL, IS NOT NULL). Highlight **Brackets** for complex conditions (grouping OR within AND). For production: use parameters for all user input, use Brackets for complex logic, build dynamic WHERE with conditional andWhere(), start with '1=1' base for fully dynamic queries, use ...spread for IN arrays, prefer andWhere() over multiple conditions in one where() for readability.

</details>

<details>
<summary>66. What is the difference between where() and andWhere()?</summary>

**where()** **replaces** any previous WHERE conditions, while **andWhere()** **adds** a new condition with AND logic. Use `.where()` for the first condition or to reset the WHERE clause, and `.andWhere()` to append additional conditions.

### Key Difference

```typescript
import { DataSource } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column({ default: true })
  isActive: boolean;

  @Column()
  age: number;
}

const dataSource = new DataSource(/* config */);

// ❌ PROBLEM: where() replaces previous condition
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.isActive = :active', { active: true })
  .where('user.age >= :age', { age: 18 }) // REPLACES previous where!
  .getMany();
// SQL: WHERE user.age >= 18
// ❌ Lost the isActive condition!

// ✅ CORRECT: andWhere() adds to existing condition
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.isActive = :active', { active: true })
  .andWhere('user.age >= :age', { age: 18 }) // ADDS to where
  .getMany();
// SQL: WHERE user.isActive = true AND user.age >= 18
// ✅ Both conditions applied!
```

### Detailed Behavior

```typescript
// where() - Sets/Replaces WHERE clause
const qb1 = dataSource
  .createQueryBuilder(User, 'user')
  .where('user.firstName = :name', { name: 'John' });
// SQL: WHERE user.firstName = 'John'

qb1.where('user.age > :age', { age: 18 });
// SQL: WHERE user.age > 18
// Previous condition REPLACED!

// andWhere() - Adds AND condition
const qb2 = dataSource
  .createQueryBuilder(User, 'user')
  .where('user.firstName = :name', { name: 'John' })
  .andWhere('user.age > :age', { age: 18 })
  .andWhere('user.isActive = :active', { active: true });
// SQL: WHERE user.firstName = 'John' AND user.age > 18 AND user.isActive = true
// All conditions combined!
```

### When to Use Each

```typescript
// Use where() for:

// 1. First condition
const qb = dataSource
  .createQueryBuilder(User, 'user')
  .where('user.isActive = :active', { active: true }); // First condition

// 2. Resetting WHERE clause
let query = dataSource
  .createQueryBuilder(User, 'user')
  .where('user.city = :city', { city: 'New York' });

// Later, completely replace the condition
if (needDifferentFilter) {
  query = query.where('user.country = :country', { country: 'USA' });
  // Previous city condition is GONE
}

// 3. Setting base WHERE in reusable query builder
function createBaseQuery() {
  return dataSource
    .createQueryBuilder(User, 'user')
    .where('user.deletedAt IS NULL'); // Base filter
}

const query1 = createBaseQuery().andWhere('user.age > :age', { age: 18 });
const query2 = createBaseQuery().andWhere('user.city = :city', {
  city: 'LA',
});
```

```typescript
// Use andWhere() for:

// 1. Adding conditions to existing WHERE
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.isActive = :active', { active: true })
  .andWhere('user.age >= :minAge', { minAge: 18 })
  .andWhere('user.isVerified = :verified', { verified: true })
  .getMany();
// SQL: WHERE user.isActive = true AND user.age >= 18 AND user.isVerified = true

// 2. Dynamic query building
let qb = dataSource
  .createQueryBuilder(User, 'user')
  .where('user.isActive = :active', { active: true });

if (minAge) {
  qb = qb.andWhere('user.age >= :minAge', { minAge });
}

if (city) {
  qb = qb.andWhere('user.city = :city', { city });
}

if (hasAvatar) {
  qb = qb.andWhere('user.avatarUrl IS NOT NULL');
}

const users = await qb.getMany();

// 3. Chaining multiple filters
const query = dataSource
  .createQueryBuilder(User, 'user')
  .where('user.country = :country', { country: 'USA' })
  .andWhere('user.state = :state', { state: 'CA' })
  .andWhere('user.city = :city', { city: 'Los Angeles' })
  .andWhere('user.isActive = :active', { active: true });
```

### Common Mistakes

```typescript
// ❌ MISTAKE 1: Using where() multiple times
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.isActive = :active', { active: true })
  .where('user.age >= :age', { age: 18 }) // REPLACES previous!
  .where('user.city = :city', { city: 'NYC' }) // REPLACES again!
  .getMany();
// Result SQL: WHERE user.city = 'NYC'
// Lost isActive and age conditions!

// ✅ CORRECT: Use where() once, then andWhere()
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.isActive = :active', { active: true })
  .andWhere('user.age >= :age', { age: 18 })
  .andWhere('user.city = :city', { city: 'NYC' })
  .getMany();
// SQL: WHERE user.isActive = true AND user.age >= 18 AND user.city = 'NYC'

// ❌ MISTAKE 2: Forgetting where() replaces
let qb = dataSource.createQueryBuilder(User, 'user');

if (filter1) {
  qb = qb.where('user.col1 = :val1', { val1: filter1 });
}

if (filter2) {
  qb = qb.where('user.col2 = :val2', { val2: filter2 }); // REPLACES!
}
// If both filters are set, only filter2 is applied

// ✅ CORRECT: Use andWhere() for additional filters
let qb = dataSource
  .createQueryBuilder(User, 'user')
  .where('1=1'); // Base condition

if (filter1) {
  qb = qb.andWhere('user.col1 = :val1', { val1: filter1 });
}

if (filter2) {
  qb = qb.andWhere('user.col2 = :val2', { val2: filter2 });
}
// Both filters applied correctly
```

### Real-World Patterns

```typescript
// Pattern 1: Dynamic filters with base WHERE
export class UserService {
  async searchUsers(filters: SearchFilters) {
    // Start with base where()
    let qb = this.dataSource
      .createQueryBuilder(User, 'user')
      .where('user.isActive = :active', { active: true });

    // Add filters with andWhere()
    if (filters.query) {
      qb = qb.andWhere(
        '(user.firstName ILIKE :q OR user.lastName ILIKE :q)',
        { q: `%${filters.query}%` }
      );
    }

    if (filters.minAge) {
      qb = qb.andWhere('user.age >= :minAge', { minAge: filters.minAge });
    }

    if (filters.cities?.length > 0) {
      qb = qb.andWhere('user.city IN (:...cities)', {
        cities: filters.cities,
      });
    }

    return qb.getMany();
  }
}

// Pattern 2: Completely dynamic WHERE
export class ProductService {
  async searchProducts(filters: ProductFilters) {
    let qb = this.dataSource
      .createQueryBuilder(Product, 'product')
      .where('1=1'); // Neutral base (always true)

    // All conditions use andWhere()
    if (filters.category) {
      qb = qb.andWhere('product.category = :cat', { cat: filters.category });
    }

    if (filters.minPrice !== undefined) {
      qb = qb.andWhere('product.price >= :min', { min: filters.minPrice });
    }

    if (filters.maxPrice !== undefined) {
      qb = qb.andWhere('product.price <= :max', { max: filters.maxPrice });
    }

    if (filters.inStock !== undefined) {
      qb = qb.andWhere('product.stock > 0');
    }

    return qb.getMany();
  }
}

// Pattern 3: Reset WHERE based on condition
async getUsers(useNewFilter: boolean, filters: any) {
  let qb = this.dataSource
    .createQueryBuilder(User, 'user')
    .where('user.isActive = :active', { active: true });

  if (useNewFilter) {
    // Completely replace WHERE
    qb = qb.where('user.status = :status', { status: 'premium' });
  } else {
    // Add to existing WHERE
    qb = qb.andWhere('user.age >= :age', { age: 18 });
  }

  return qb.getMany();
}
```

### Best Practices

```typescript
// ✅ GOOD: Use where() for first condition
const qb = dataSource
  .createQueryBuilder(User, 'user')
  .where('user.isActive = :active', { active: true });

// ✅ GOOD: Use andWhere() for all subsequent conditions
qb.andWhere('user.age >= :age', { age: 18 })
  .andWhere('user.city = :city', { city: 'NYC' });

// ✅ GOOD: Start with '1=1' for fully dynamic queries
let qb = dataSource
  .createQueryBuilder(User, 'user')
  .where('1=1');

if (filter1) qb = qb.andWhere('condition1');
if (filter2) qb = qb.andWhere('condition2');

// ✅ GOOD: Use where() to reset when needed
let qb = dataSource.createQueryBuilder(User, 'user');

if (scenario === 'A') {
  qb = qb.where('user.type = :type', { type: 'premium' });
} else {
  qb = qb.where('user.type = :type', { type: 'standard' });
}

// ❌ BAD: Don't chain multiple where() calls
const qb = dataSource
  .createQueryBuilder(User, 'user')
  .where('condition1')
  .where('condition2') // REPLACES condition1!
  .where('condition3'); // REPLACES condition2!

// ✅ GOOD: Use where() once, then andWhere()
const qb = dataSource
  .createQueryBuilder(User, 'user')
  .where('condition1')
  .andWhere('condition2')
  .andWhere('condition3');
```

**Interview Tip**: Explain **where()** **replaces** previous WHERE conditions while **andWhere()** **adds** conditions with AND logic. Use `.where()` for **first condition** or to **reset** WHERE clause, `.andWhere()` to **append** additional filters. Emphasize common mistake: chaining multiple where() calls (each replaces previous). Mention patterns: start with where('1=1') for fully dynamic queries, use where() for base filter + andWhere() for optional filters, use where() to conditionally reset entire WHERE clause. For production: use where() once at start, use andWhere() for all additional conditions, start with '1=1' if all conditions are optional, document when intentionally replacing WHERE with where(), prefer andWhere() for dynamic query building to avoid accidentally losing conditions.

</details>

<details>
<summary>67. How do you use OR conditions in QueryBuilder?</summary>

Use **orWhere()** to add OR conditions to your query. For complex OR logic within AND conditions, use **Brackets** to group conditions properly. TypeORM also provides **orWhere()** for simple cases and nested **Brackets** for advanced scenarios.

### Basic OR Conditions

```typescript
import { DataSource } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  email: string;

  @Column()
  role: string;

  @Column({ default: true })
  isActive: boolean;

  @Column()
  age: number;
}

const dataSource = new DataSource(/* config */);

// Simple OR with orWhere()
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.firstName = :name', { name: 'John' })
  .orWhere('user.firstName = :name2', { name2: 'Jane' })
  .getMany();
// SQL: WHERE user.firstName = 'John' OR user.firstName = 'Jane'

// Multiple OR conditions
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.role = :role1', { role1: 'admin' })
  .orWhere('user.role = :role2', { role2: 'moderator' })
  .orWhere('user.role = :role3', { role3: 'editor' })
  .getMany();
// SQL: WHERE user.role = 'admin' OR user.role = 'moderator' OR user.role = 'editor'
```

### OR with AND - Using Brackets

```typescript
import { Brackets } from 'typeorm';

// Problem: Mixing AND and OR without Brackets
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.isActive = :active', { active: true })
  .andWhere('user.age >= :age', { age: 18 })
  .orWhere('user.role = :role', { role: 'admin' })
  .getMany();
// SQL: WHERE user.isActive = true AND user.age >= 18 OR user.role = 'admin'
// ⚠️ This might not work as expected due to operator precedence!

// ✅ CORRECT: Use Brackets to group OR conditions
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.isActive = :active', { active: true })
  .andWhere(
    new Brackets((qb) => {
      qb.where('user.age >= :age', { age: 18 })
        .orWhere('user.role = :role', { role: 'admin' });
    })
  )
  .getMany();
// SQL: WHERE user.isActive = true AND (user.age >= 18 OR user.role = 'admin')
```

### Complex OR Scenarios

```typescript
// Multiple OR groups with AND
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where(
    new Brackets((qb) => {
      qb.where('user.firstName = :name1', { name1: 'John' })
        .orWhere('user.firstName = :name2', { name2: 'Jane' });
    })
  )
  .andWhere(
    new Brackets((qb) => {
      qb.where('user.role = :role1', { role1: 'admin' })
        .orWhere('user.role = :role2', { role2: 'moderator' });
    })
  )
  .getMany();
// SQL: WHERE (user.firstName = 'John' OR user.firstName = 'Jane')
//      AND (user.role = 'admin' OR user.role = 'moderator')

// Nested OR conditions
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.isActive = :active', { active: true })
  .andWhere(
    new Brackets((qb) => {
      qb.where(
        new Brackets((subQb) => {
          subQb
            .where('user.age >= :minAge', { minAge: 18 })
            .andWhere('user.age <= :maxAge', { maxAge: 30 });
        })
      ).orWhere(
        new Brackets((subQb) => {
          subQb
            .where('user.role = :role1', { role1: 'admin' })
            .orWhere('user.role = :role2', { role2: 'moderator' });
        })
      );
    })
  )
  .getMany();
// SQL: WHERE user.isActive = true
//      AND ((user.age >= 18 AND user.age <= 30) OR (user.role = 'admin' OR user.role = 'moderator'))
```

### OR with Search Queries

```typescript
// Search across multiple fields
export class UserService {
  async searchUsers(query: string) {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .where('user.firstName ILIKE :query', { query: `%${query}%` })
      .orWhere('user.lastName ILIKE :query', { query: `%${query}%` })
      .orWhere('user.email ILIKE :query', { query: `%${query}%` })
      .getMany();
  }

  // Better: Use Brackets when combining with other filters
  async searchActiveUsers(query: string) {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .where('user.isActive = :active', { active: true })
      .andWhere(
        new Brackets((qb) => {
          qb.where('user.firstName ILIKE :query', { query: `%${query}%` })
            .orWhere('user.lastName ILIKE :query', { query: `%${query}%` })
            .orWhere('user.email ILIKE :query', { query: `%${query}%` });
        })
      )
      .getMany();
  }
  // SQL: WHERE user.isActive = true
  //      AND (user.firstName ILIKE '%query%' OR user.lastName ILIKE '%query%' OR user.email ILIKE '%query%')
}
```

### Dynamic OR Conditions

```typescript
// Building dynamic OR queries
export class UserService {
  async findUsersByRoles(roles: string[]) {
    if (!roles || roles.length === 0) {
      return [];
    }

    let qb = this.dataSource
      .createQueryBuilder(User, 'user')
      .where('1=0'); // Start with false condition

    for (const role of roles) {
      qb = qb.orWhere('user.role = :role', { role });
    }

    return qb.getMany();
  }

  // Better approach: Use IN operator
  async findUsersByRolesBetter(roles: string[]) {
    if (!roles || roles.length === 0) {
      return [];
    }

    return this.dataSource
      .createQueryBuilder(User, 'user')
      .where('user.role IN (:...roles)', { roles })
      .getMany();
  }

  // Dynamic search with optional filters
  async advancedSearch(filters: {
    query?: string;
    roles?: string[];
    minAge?: number;
  }) {
    let qb = this.dataSource
      .createQueryBuilder(User, 'user')
      .where('user.isActive = :active', { active: true });

    // Add text search with OR
    if (filters.query) {
      qb = qb.andWhere(
        new Brackets((subQb) => {
          subQb
            .where('user.firstName ILIKE :q', { q: `%${filters.query}%` })
            .orWhere('user.lastName ILIKE :q', { q: `%${filters.query}%` })
            .orWhere('user.email ILIKE :q', { q: `%${filters.query}%` });
        })
      );
    }

    // Add role filter with OR
    if (filters.roles && filters.roles.length > 0) {
      qb = qb.andWhere(
        new Brackets((subQb) => {
          filters.roles.forEach((role, index) => {
            if (index === 0) {
              subQb.where('user.role = :role' + index, {
                ['role' + index]: role,
              });
            } else {
              subQb.orWhere('user.role = :role' + index, {
                ['role' + index]: role,
              });
            }
          });
        })
      );
    }

    // Add age filter
    if (filters.minAge) {
      qb = qb.andWhere('user.age >= :minAge', { minAge: filters.minAge });
    }

    return qb.getMany();
  }
}
```

### Real-World Examples

```typescript
// Multi-criteria user search
export class UserService {
  async flexibleSearch(criteria: {
    nameOrEmail?: string;
    isPremiumOrAdmin?: boolean;
    youngOrSenior?: boolean;
  }) {
    let qb = this.dataSource
      .createQueryBuilder(User, 'user')
      .where('user.isActive = :active', { active: true });

    // Search by name OR email
    if (criteria.nameOrEmail) {
      qb = qb.andWhere(
        new Brackets((subQb) => {
          subQb
            .where('user.firstName ILIKE :search', {
              search: `%${criteria.nameOrEmail}%`,
            })
            .orWhere('user.lastName ILIKE :search', {
              search: `%${criteria.nameOrEmail}%`,
            })
            .orWhere('user.email ILIKE :search', {
              search: `%${criteria.nameOrEmail}%`,
            });
        })
      );
    }

    // Filter by premium OR admin
    if (criteria.isPremiumOrAdmin) {
      qb = qb.andWhere(
        new Brackets((subQb) => {
          subQb
            .where('user.isPremium = :premium', { premium: true })
            .orWhere('user.role = :role', { role: 'admin' });
        })
      );
    }

    // Filter by age groups (young: 18-30 OR senior: 60+)
    if (criteria.youngOrSenior) {
      qb = qb.andWhere(
        new Brackets((subQb) => {
          subQb
            .where('user.age BETWEEN :young1 AND :young2', {
              young1: 18,
              young2: 30,
            })
            .orWhere('user.age >= :senior', { senior: 60 });
        })
      );
    }

    return qb.getMany();
  }

  // Permission-based access
  async getUsersByPermission(userId: number) {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .where(
        new Brackets((qb) => {
          qb.where('user.id = :userId', { userId }) // Own profile
            .orWhere('user.isPublic = :public', { public: true }) // Public profiles
            .orWhere('user.teamId IN (SELECT teamId FROM users WHERE id = :uid)', {
              uid: userId,
            }); // Same team
        })
      )
      .getMany();
  }
}
```

### Common Patterns

```typescript
// Pattern 1: OR within AND groups
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.country = :country', { country: 'USA' })
  .andWhere(
    new Brackets((qb) => {
      qb.where('user.state = :state1', { state1: 'CA' })
        .orWhere('user.state = :state2', { state2: 'NY' })
        .orWhere('user.state = :state3', { state3: 'TX' });
    })
  )
  .andWhere('user.isActive = :active', { active: true })
  .getMany();
// SQL: WHERE user.country = 'USA'
//      AND (user.state = 'CA' OR user.state = 'NY' OR user.state = 'TX')
//      AND user.isActive = true

// Pattern 2: Multiple OR groups
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where(
    new Brackets((qb1) => {
      qb1
        .where('user.role = :admin', { admin: 'admin' })
        .orWhere('user.role = :mod', { mod: 'moderator' });
    })
  )
  .andWhere(
    new Brackets((qb2) => {
      qb2
        .where('user.verified = :verified', { verified: true })
        .orWhere('user.trustScore > :score', { score: 100 });
    })
  )
  .getMany();
// SQL: WHERE (user.role = 'admin' OR user.role = 'moderator')
//      AND (user.verified = true OR user.trustScore > 100)
```

### Best Practices

```typescript
// ✅ GOOD: Use Brackets for complex OR conditions
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.isActive = :active', { active: true })
  .andWhere(
    new Brackets((qb) => {
      qb.where('user.age >= :age', { age: 18 }).orWhere('user.role = :role', {
        role: 'admin',
      });
    })
  )
  .getMany();

// ❌ BAD: Mixing AND/OR without Brackets (unpredictable)
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.isActive = :active', { active: true })
  .andWhere('user.age >= :age', { age: 18 })
  .orWhere('user.role = :role', { role: 'admin' })
  .getMany();

// ✅ GOOD: Use IN instead of multiple OR for equality checks
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.role IN (:...roles)', { roles: ['admin', 'moderator', 'editor'] })
  .getMany();

// ❌ BAD: Multiple OR for the same column
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.role = :r1', { r1: 'admin' })
  .orWhere('user.role = :r2', { r2: 'moderator' })
  .orWhere('user.role = :r3', { r3: 'editor' })
  .getMany();

// ✅ GOOD: Always use Brackets when combining search with other filters
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.isActive = :active', { active: true })
  .andWhere(
    new Brackets((qb) => {
      qb.where('user.name ILIKE :q', { q: `%${query}%` })
        .orWhere('user.email ILIKE :q', { q: `%${query}%` });
    })
  )
  .getMany();

// ✅ GOOD: Document complex OR logic
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.isActive = :active', { active: true })
  .andWhere(
    // Users who are either: verified premium users OR admins with high trust score
    new Brackets((qb) => {
      qb.where(
        new Brackets((premiumQb) => {
          premiumQb
            .where('user.isPremium = :premium', { premium: true })
            .andWhere('user.verified = :verified', { verified: true });
        })
      ).orWhere(
        new Brackets((adminQb) => {
          adminQb
            .where('user.role = :role', { role: 'admin' })
            .andWhere('user.trustScore >= :score', { score: 90 });
        })
      );
    })
  )
  .getMany();
```

**Interview Tip**: Explain **orWhere()** adds OR conditions to queries. Emphasize **Brackets are critical** for complex logic—use `new Brackets((qb) => { qb.where(...).orWhere(...) })` to group OR conditions within AND logic, preventing operator precedence issues. Mention alternatives: use **IN operator** instead of multiple OR for same-column equality checks (cleaner, faster). Highlight real-world use cases: multi-field search (firstName OR lastName OR email), role-based access (admin OR moderator), permission checks. For production: always wrap OR groups in Brackets when combined with AND, use IN for lists of values, document complex OR logic, test SQL output to verify grouping, prefer readable Brackets structure over flat OR chains, use dynamic Brackets for conditional OR filters.

</details>

<details>
<summary>68. How do you perform JOIN operations with QueryBuilder?</summary>

QueryBuilder supports **innerJoin**, **leftJoin**, **innerJoinAndSelect**, and **leftJoinAndSelect** for JOIN operations. Use **Join** to include relations, **JoinAndSelect** to load related entities. TypeORM handles both relation-based joins (using entity relations) and raw SQL joins.

### Basic JOIN Operations

```typescript
import { DataSource } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @OneToMany(() => Post, (post) => post.author)
  posts: Post[];

  @OneToOne(() => Profile, (profile) => profile.user)
  profile: Profile;
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column()
  content: string;

  @ManyToOne(() => User, (user) => user.posts)
  author: User;

  @OneToMany(() => Comment, (comment) => comment.post)
  comments: Comment[];
}

@Entity()
export class Comment {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  text: string;

  @ManyToOne(() => Post, (post) => post.comments)
  post: Post;

  @ManyToOne(() => User)
  author: User;
}

const dataSource = new DataSource(/* config */);

// 1. leftJoin - Join without loading relation data
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoin('user.posts', 'post')
  .where('post.title ILIKE :title', { title: '%TypeORM%' })
  .getMany();
// Returns User entities, but posts are NOT loaded
// SQL: SELECT user.* FROM users user
//      LEFT JOIN posts post ON post.authorId = user.id
//      WHERE post.title ILIKE '%TypeORM%'

// 2. leftJoinAndSelect - Join and load relation data
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoinAndSelect('user.posts', 'post')
  .getMany();
// Returns User entities WITH posts array populated
// SQL: SELECT user.*, post.* FROM users user
//      LEFT JOIN posts post ON post.authorId = user.id
```

### JOIN Types Comparison

```typescript
// leftJoin - Includes all users, even without posts
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoin('user.posts', 'post')
  .select(['user.id', 'user.firstName'])
  .addSelect('COUNT(post.id)', 'postCount')
  .groupBy('user.id')
  .getRawMany();
// Returns all users with post count (0 if no posts)

// innerJoin - Only users who HAVE posts
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .innerJoin('user.posts', 'post')
  .select(['user.id', 'user.firstName'])
  .addSelect('COUNT(post.id)', 'postCount')
  .groupBy('user.id')
  .getRawMany();
// Returns only users with at least one post

// leftJoinAndSelect - Load all users WITH their posts
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoinAndSelect('user.posts', 'post')
  .getMany();
// User entities with posts array (empty array if no posts)

// innerJoinAndSelect - Load users who have posts, WITH posts loaded
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .innerJoinAndSelect('user.posts', 'post')
  .getMany();
// Only users with posts, posts array populated
```

### Multiple JOINs

```typescript
// Join multiple relations
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoinAndSelect('user.posts', 'post')
  .leftJoinAndSelect('user.profile', 'profile')
  .leftJoinAndSelect('post.comments', 'comment')
  .where('user.id = :id', { id: 1 })
  .getOne();
// Returns user with posts, profile, and comments on posts

// Nested joins
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoinAndSelect('user.posts', 'post')
  .leftJoinAndSelect('post.comments', 'comment')
  .leftJoinAndSelect('comment.author', 'commentAuthor')
  .where('user.id = :id', { id: 1 })
  .getOne();
// Returns user → posts → comments → comment authors
```

### JOIN with WHERE Conditions

```typescript
// Filter on joined table
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoinAndSelect('user.posts', 'post')
  .where('post.isPublished = :published', { published: true })
  .getMany();
// Users with published posts loaded

// Multiple conditions on joins
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoinAndSelect('user.posts', 'post')
  .where('user.isActive = :active', { active: true })
  .andWhere('post.isPublished = :published', { published: true })
  .andWhere('post.createdAt > :date', { date: new Date('2024-01-01') })
  .getMany();

// Filter parent but load all children
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoinAndSelect('user.posts', 'post')
  .where('user.isActive = :active', { active: true })
  .getMany();
// Returns active users with ALL their posts (published or not)
```

### JOIN with ON Conditions

```typescript
// Custom JOIN condition
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoinAndSelect(
    'user.posts',
    'post',
    'post.isPublished = :published',
    { published: true }
  )
  .getMany();
// Returns all users, but only their published posts

// Complex ON condition
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoinAndSelect(
    'user.posts',
    'post',
    'post.isPublished = :published AND post.createdAt > :date',
    { published: true, date: new Date('2024-01-01') }
  )
  .getMany();
// Returns all users with posts published after 2024-01-01
```

### Manual (Non-Relation) JOINs

```typescript
// Join tables without explicit relation
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoin(Post, 'post', 'post.authorId = user.id')
  .select(['user.id', 'user.firstName'])
  .addSelect('COUNT(post.id)', 'postCount')
  .groupBy('user.id')
  .getRawMany();

// Join with complex condition
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoin(
    Post,
    'post',
    'post.authorId = user.id AND post.isPublished = :published',
    { published: true }
  )
  .select('user.id', 'userId')
  .addSelect('COUNT(post.id)', 'publishedPostCount')
  .groupBy('user.id')
  .getRawMany();
```

### Real-World Examples

```typescript
// Get users with post statistics
export class UserService {
  async getUsersWithStats() {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .leftJoin('user.posts', 'post')
      .select(['user.id', 'user.firstName', 'user.email'])
      .addSelect('COUNT(post.id)', 'postCount')
      .addSelect('MAX(post.createdAt)', 'lastPostDate')
      .groupBy('user.id')
      .getRawMany();
  }

  // Get users with filtered relations
  async getUsersWithRecentPosts(days: number = 7) {
    const cutoffDate = new Date();
    cutoffDate.setDate(cutoffDate.getDate() - days);

    return this.dataSource
      .createQueryBuilder(User, 'user')
      .leftJoinAndSelect(
        'user.posts',
        'post',
        'post.createdAt > :cutoff',
        { cutoff: cutoffDate }
      )
      .where('user.isActive = :active', { active: true })
      .getMany();
  }

  // Complex nested query
  async getUsersWithEngagement() {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .leftJoinAndSelect('user.posts', 'post', 'post.isPublished = :pub', {
        pub: true,
      })
      .leftJoinAndSelect('post.comments', 'comment')
      .leftJoinAndSelect('comment.author', 'commentAuthor')
      .where('user.isActive = :active', { active: true })
      .orderBy('user.createdAt', 'DESC')
      .addOrderBy('post.createdAt', 'DESC')
      .addOrderBy('comment.createdAt', 'DESC')
      .take(10)
      .getMany();
  }
}

// Dashboard with aggregations
export class DashboardService {
  async getUserActivityStats() {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .leftJoin('user.posts', 'post')
      .leftJoin('post.comments', 'comment')
      .select('user.id', 'userId')
      .addSelect('user.firstName', 'name')
      .addSelect('COUNT(DISTINCT post.id)', 'totalPosts')
      .addSelect('COUNT(DISTINCT comment.id)', 'totalComments')
      .addSelect(
        'COUNT(DISTINCT CASE WHEN post.createdAt > NOW() - INTERVAL \'7 days\' THEN post.id END)',
        'recentPosts'
      )
      .groupBy('user.id')
      .having('COUNT(post.id) > :minPosts', { minPosts: 0 })
      .orderBy('totalPosts', 'DESC')
      .getRawMany();
  }
}
```

### JOIN Best Practices

```typescript
// ✅ GOOD: Use leftJoinAndSelect to load relations
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoinAndSelect('user.posts', 'post')
  .getMany();
// Returns User[] with posts populated

// ✅ GOOD: Use leftJoin for filtering without loading
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoin('user.posts', 'post')
  .where('post.title ILIKE :search', { search: '%query%' })
  .getMany();
// Returns User[] (posts not loaded, but filtered by posts)

// ❌ BAD: Using innerJoin when you want all parent records
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .innerJoin('user.posts', 'post')
  .getMany();
// Only returns users who have posts

// ✅ GOOD: Use leftJoin to include users without posts
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoin('user.posts', 'post')
  .getMany();
// Returns all users (even without posts)

// ✅ GOOD: Use ON condition to filter joined records
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoinAndSelect(
    'user.posts',
    'post',
    'post.isPublished = :pub',
    { pub: true }
  )
  .getMany();
// All users, but only published posts loaded

// ✅ GOOD: Order by joined table columns
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoinAndSelect('user.posts', 'post')
  .orderBy('user.createdAt', 'DESC')
  .addOrderBy('post.createdAt', 'DESC')
  .getMany();

// ✅ GOOD: Use aliases consistently
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoinAndSelect('user.posts', 'post')
  .leftJoinAndSelect('post.comments', 'comment')
  .where('user.isActive = :active', { active: true })
  .andWhere('post.isPublished = :pub', { pub: true })
  .getMany();
```

**Interview Tip**: Explain **JOIN** operations load related entities using **leftJoin**, **innerJoin**, **leftJoinAndSelect**, **innerJoinAndSelect**. Emphasize difference: **Join** (filter by relation, don't load), **JoinAndSelect** (load relation data). Mention **leftJoin** includes all parent records (even without children), **innerJoin** only parents with children. Highlight **ON conditions** for filtering joined records (third parameter). For production: use leftJoinAndSelect for loading relations, leftJoin for filtering without loading data, innerJoin when parent requires child, add ON conditions to filter joined records at JOIN level (not WHERE), order multiple joins with addOrderBy(), be aware of N+1 queries (use joins to eager load), test performance with EXPLAIN on complex joins.

</details>

<details>
<summary>69. What is the difference between innerJoin and leftJoin?</summary>

**innerJoin** returns only parent records that **have matching** child records, while **leftJoin** returns **all parent records** regardless of whether they have children. Use **innerJoin** when child is required, **leftJoin** when child is optional.

### Key Difference

```typescript
import { DataSource } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @OneToMany(() => Post, (post) => post.author)
  posts: Post[];
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToOne(() => User, (user) => user.posts)
  author: User;
}

const dataSource = new DataSource(/* config */);

// Assume: User 1 has 2 posts, User 2 has 0 posts, User 3 has 1 post

// leftJoin - Returns ALL users (3 users)
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoin('user.posts', 'post')
  .getMany();
// Result: [User 1, User 2, User 3]
// SQL: SELECT * FROM users user LEFT JOIN posts post ON post.authorId = user.id

// innerJoin - Returns ONLY users with posts (2 users)
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .innerJoin('user.posts', 'post')
  .getMany();
// Result: [User 1, User 3]
// SQL: SELECT * FROM users user INNER JOIN posts post ON post.authorId = user.id
```

### Visual Comparison

```typescript
/*
Database State:
Users: [User 1, User 2, User 3]
Posts: [Post A (author: User 1), Post B (author: User 1), Post C (author: User 3)]

┌─────────────┬──────────────────┬──────────────────┐
│ JOIN Type   │ Result           │ Count            │
├─────────────┼──────────────────┼──────────────────┤
│ leftJoin    │ User 1, User 2,  │ 3 users          │
│             │ User 3           │ (includes User 2 │
│             │                  │  with no posts)  │
├─────────────┼──────────────────┼──────────────────┤
│ innerJoin   │ User 1, User 3   │ 2 users          │
│             │                  │ (excludes User 2)│
└─────────────┴──────────────────┴──────────────────┘
*/
```

### With Aggregations

```typescript
// leftJoin - All users with post count (0 if no posts)
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoin('user.posts', 'post')
  .select('user.id', 'userId')
  .addSelect('user.firstName', 'name')
  .addSelect('COUNT(post.id)', 'postCount')
  .groupBy('user.id')
  .getRawMany();
// Result: [
//   { userId: 1, name: 'John', postCount: '2' },
//   { userId: 2, name: 'Jane', postCount: '0' },  ← Included with 0
//   { userId: 3, name: 'Bob', postCount: '1' }
// ]

// innerJoin - Only users with posts
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .innerJoin('user.posts', 'post')
  .select('user.id', 'userId')
  .addSelect('user.firstName', 'name')
  .addSelect('COUNT(post.id)', 'postCount')
  .groupBy('user.id')
  .getRawMany();
// Result: [
//   { userId: 1, name: 'John', postCount: '2' },
//   { userId: 3, name: 'Bob', postCount: '1' }
// ]  ← User 2 excluded
```

### With JoinAndSelect

```typescript
// leftJoinAndSelect - All users, with posts array (empty if no posts)
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoinAndSelect('user.posts', 'post')
  .getMany();
// Result: [
//   User { id: 1, firstName: 'John', posts: [Post, Post] },
//   User { id: 2, firstName: 'Jane', posts: [] },  ← Empty array
//   User { id: 3, firstName: 'Bob', posts: [Post] }
// ]

// innerJoinAndSelect - Only users with posts, posts array populated
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .innerJoinAndSelect('user.posts', 'post')
  .getMany();
// Result: [
//   User { id: 1, firstName: 'John', posts: [Post, Post] },
//   User { id: 3, firstName: 'Bob', posts: [Post] }
// ]  ← User 2 not included
```

### Use Cases

```typescript
// Use leftJoin when:

// 1. You want all parent records (optional child)
const allUsers = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoinAndSelect('user.posts', 'post')
  .getMany();
// Get all users, with or without posts

// 2. Counting children (including zero counts)
const userStats = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoin('user.posts', 'post')
  .select('user.id', 'id')
  .addSelect('COUNT(post.id)', 'postCount')
  .groupBy('user.id')
  .getRawMany();
// All users with their post counts (0 for users without posts)

// 3. Dashboard with optional data
const dashboard = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoin('user.posts', 'post')
  .leftJoin('user.profile', 'profile')
  .select('user.id', 'id')
  .addSelect('COUNT(post.id)', 'posts')
  .addSelect('profile.bio', 'bio')
  .groupBy('user.id')
  .addGroupBy('profile.bio')
  .getRawMany();
// All users even if they don't have posts or profile
```

```typescript
// Use innerJoin when:

// 1. Child record is required
const usersWithPosts = await dataSource
  .createQueryBuilder(User, 'user')
  .innerJoinAndSelect('user.posts', 'post')
  .getMany();
// Only users who have written posts

// 2. Filtering by child properties
const activeAuthors = await dataSource
  .createQueryBuilder(User, 'user')
  .innerJoin('user.posts', 'post')
  .where('post.isPublished = :pub', { pub: true })
  .distinct(true)
  .getMany();
// Only users with published posts

// 3. Performance optimization (reduce result set)
const contributingUsers = await dataSource
  .createQueryBuilder(User, 'user')
  .innerJoin('user.posts', 'post')
  .innerJoin('post.comments', 'comment')
  .where('comment.createdAt > :date', { date: recentDate })
  .distinct(true)
  .getMany();
// Only users whose posts have recent comments
```

### Common Mistakes

```typescript
// ❌ MISTAKE: Using innerJoin when you want all parents
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .innerJoinAndSelect('user.posts', 'post')
  .getMany();
// This excludes users without posts!

// ✅ CORRECT: Use leftJoin for optional relations
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoinAndSelect('user.posts', 'post')
  .getMany();
// Includes all users

// ❌ MISTAKE: Using leftJoin for required relations
const postsWithAuthors = await dataSource
  .createQueryBuilder(Post, 'post')
  .leftJoinAndSelect('post.author', 'author')
  .getMany();
// This would include posts with null authors (if possible)

// ✅ CORRECT: Use innerJoin when relation is required
const postsWithAuthors = await dataSource
  .createQueryBuilder(Post, 'post')
  .innerJoinAndSelect('post.author', 'author')
  .getMany();
// Only posts with valid authors
```

### Real-World Examples

```typescript
// User listing with optional profile
export class UserService {
  async listUsers() {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .leftJoinAndSelect('user.profile', 'profile')
      .leftJoinAndSelect('user.posts', 'post')
      .orderBy('user.createdAt', 'DESC')
      .getMany();
  }
  // Returns all users, even those without profile or posts

  // Active authors only
  async getActiveAuthors() {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .innerJoin('user.posts', 'post')
      .where('post.isPublished = :pub', { pub: true })
      .andWhere('user.isActive = :active', { active: true })
      .distinct(true)
      .getMany();
  }
  // Returns only users with published posts

  // User statistics (all users)
  async getUserStatistics() {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .leftJoin('user.posts', 'post')
      .leftJoin('post.comments', 'comment')
      .select('user.id', 'userId')
      .addSelect('user.firstName', 'name')
      .addSelect('COUNT(DISTINCT post.id)', 'totalPosts')
      .addSelect('COUNT(DISTINCT comment.id)', 'totalComments')
      .groupBy('user.id')
      .getRawMany();
  }
  // All users with their stats (0 if no posts/comments)

  // Contributors only (users with activity)
  async getContributors() {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .innerJoin('user.posts', 'post')
      .select('user.id', 'userId')
      .addSelect('user.firstName', 'name')
      .addSelect('COUNT(post.id)', 'contributions')
      .groupBy('user.id')
      .having('COUNT(post.id) >= :min', { min: 5 })
      .orderBy('contributions', 'DESC')
      .getRawMany();
  }
  // Only users with at least 5 posts
}
```

### Performance Considerations

```typescript
// leftJoin - Larger result set
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoinAndSelect('user.posts', 'post')
  .getMany();
// Loads all 1000 users (even 900 without posts)

// innerJoin - Smaller result set (better performance)
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .innerJoinAndSelect('user.posts', 'post')
  .getMany();
// Loads only 100 users who have posts

// Use innerJoin for filtering, then leftJoin for loading
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .innerJoin('user.posts', 'post') // Filter: users with posts
  .leftJoinAndSelect('user.profile', 'profile') // Load: optional profile
  .where('post.isPublished = :pub', { pub: true })
  .distinct(true)
  .getMany();
// Users with published posts, with optional profiles loaded
```

### Best Practices

```typescript
// ✅ GOOD: Use leftJoin for optional relations
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoinAndSelect('user.profile', 'profile')
  .leftJoinAndSelect('user.posts', 'post')
  .getMany();

// ✅ GOOD: Use innerJoin when child is required
const posts = await dataSource
  .createQueryBuilder(Post, 'post')
  .innerJoinAndSelect('post.author', 'author')
  .getMany();

// ✅ GOOD: Mix joins based on requirements
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .innerJoinAndSelect('user.posts', 'post') // Must have posts
  .leftJoinAndSelect('user.profile', 'profile') // Profile optional
  .leftJoinAndSelect('post.comments', 'comment') // Comments optional
  .getMany();

// ✅ GOOD: Use innerJoin for filtering, leftJoin for loading
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .innerJoin('user.posts', 'post') // Filter by posts
  .leftJoinAndSelect('user.profile', 'profile') // Load profile
  .where('post.isPublished = :pub', { pub: true })
  .distinct(true)
  .getMany();

// ✅ GOOD: Document why you chose specific JOIN type
// leftJoin: We want all users, even those without posts
const allUsers = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoinAndSelect('user.posts', 'post')
  .getMany();

// innerJoin: Only users who have contributed content
const contributors = await dataSource
  .createQueryBuilder(User, 'user')
  .innerJoinAndSelect('user.posts', 'post')
  .getMany();
```

**Interview Tip**: Explain **innerJoin** returns only parents with **matching children** (required relation), **leftJoin** returns **all parents** regardless of children (optional relation). Emphasize SQL behavior: innerJoin excludes parents without children, leftJoin includes all parents (child fields NULL if no match). Mention use cases: leftJoin for dashboards/stats (include zero counts), optional relations (profile, avatar), all-users lists; innerJoin for filtering by child properties, required relations (post must have author), performance optimization (smaller result set). For production: use leftJoin by default unless child is required, use innerJoin to filter by child existence, mix both (innerJoin to filter, leftJoin to load other optional relations), add distinct(true) with innerJoin to avoid duplicates, test result set size for performance.

</details>

<details>
<summary>70. How do you use parameters to prevent SQL injection?</summary>

Always use **parameterized queries** with `:paramName` syntax to prevent SQL injection. TypeORM automatically escapes parameters, making queries safe from malicious input. **Never** concatenate user input directly into SQL strings.

### Basic Parameter Usage

```typescript
import { DataSource } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  email: string;

  @Column()
  role: string;
}

const dataSource = new DataSource(/* config */);

// ✅ SAFE: Using parameters
const email = "user@example.com'; DROP TABLE users; --"; // Malicious input
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.email = :email', { email })
  .getMany();
// SQL: WHERE user.email = 'user@example.com''; DROP TABLE users; --'
// Parameter is properly escaped - safe!

// ❌ DANGEROUS: String concatenation
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where(`user.email = '${email}'`) // SQL INJECTION RISK!
  .getMany();
// SQL: WHERE user.email = 'user@example.com'; DROP TABLE users; --'
// This would DROP your users table!
```

### Parameter Syntax Options

```typescript
// 1. Inline parameters (recommended)
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.age >= :minAge', { minAge: 18 })
  .andWhere('user.city = :city', { city: 'New York' })
  .getMany();

// 2. Using setParameter() method
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.age >= :minAge')
  .setParameter('minAge', 18)
  .andWhere('user.city = :city')
  .setParameter('city', 'New York')
  .getMany();

// 3. Using setParameters() for multiple parameters
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.age >= :minAge AND user.city = :city')
  .setParameters({ minAge: 18, city: 'New York' })
  .getMany();

// 4. Object-style WHERE (auto-parameterized)
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where({ email: 'user@example.com', isActive: true })
  .getMany();
// TypeORM automatically parameterizes these values
```

### Array Parameters (IN Clause)

```typescript
// Use ...spread syntax for arrays
const roles = ['admin', 'moderator', 'editor'];
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.role IN (:...roles)', { roles })
  .getMany();
// SQL: WHERE user.role IN ('admin', 'moderator', 'editor')

// Multiple array parameters
const cities = ['New York', 'Los Angeles', 'Chicago'];
const statuses = ['active', 'pending'];
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.city IN (:...cities)', { cities })
  .andWhere('user.status IN (:...statuses)', { statuses })
  .getMany();

// ❌ WRONG: Without spread operator
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.role IN (:roles)', { roles }) // ❌ Won't work!
  .getMany();

// ✅ CORRECT: With spread operator
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.role IN (:...roles)', { roles }) // ✅ Works!
  .getMany();
```

### Common SQL Injection Scenarios

```typescript
// ❌ VULNERABLE: Direct string concatenation
const searchTerm = "'; DROP TABLE users; --";
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where(`user.firstName LIKE '%${searchTerm}%'`) // DANGEROUS!
  .getMany();

// ✅ SAFE: Using parameters
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.firstName LIKE :search', { search: `%${searchTerm}%` })
  .getMany();

// ❌ VULNERABLE: Dynamic column names (avoid!)
const sortColumn = "firstName; DROP TABLE users; --";
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .orderBy(`user.${sortColumn}`, 'ASC') // DANGEROUS!
  .getMany();

// ✅ SAFE: Whitelist column names
const allowedColumns = ['firstName', 'lastName', 'email', 'createdAt'];
const sortColumn = 'firstName';
if (allowedColumns.includes(sortColumn)) {
  const users = await dataSource
    .createQueryBuilder(User, 'user')
    .orderBy(`user.${sortColumn}`, 'ASC')
    .getMany();
}
```

### Real-World Examples

```typescript
// User authentication (safe from SQL injection)
export class AuthService {
  async validateUser(email: string, password: string) {
    // ✅ SAFE: Parameters used for user input
    const user = await this.dataSource
      .createQueryBuilder(User, 'user')
      .where('user.email = :email', { email })
      .andWhere('user.isActive = :active', { active: true })
      .getOne();

    if (user && (await bcrypt.compare(password, user.passwordHash))) {
      return user;
    }
    return null;
  }
}

// Search functionality
export class UserService {
  async searchUsers(query: string, filters: {
    roles?: string[];
    minAge?: number;
    cities?: string[];
  }) {
    let qb = this.dataSource
      .createQueryBuilder(User, 'user')
      .where('user.isActive = :active', { active: true });

    // ✅ SAFE: Search term parameterized
    if (query) {
      qb = qb.andWhere(
        new Brackets((subQb) => {
          subQb
            .where('user.firstName ILIKE :query', { query: `%${query}%` })
            .orWhere('user.lastName ILIKE :query', { query: `%${query}%` })
            .orWhere('user.email ILIKE :query', { query: `%${query}%` });
        })
      );
    }

    // ✅ SAFE: Array parameter with spread
    if (filters.roles && filters.roles.length > 0) {
      qb = qb.andWhere('user.role IN (:...roles)', { roles: filters.roles });
    }

    // ✅ SAFE: Numeric parameter
    if (filters.minAge) {
      qb = qb.andWhere('user.age >= :minAge', { minAge: filters.minAge });
    }

    // ✅ SAFE: Array parameter
    if (filters.cities && filters.cities.length > 0) {
      qb = qb.andWhere('user.city IN (:...cities)', { cities: filters.cities });
    }

    return qb.getMany();
  }
}

// Dynamic sorting with whitelist
export class UserService {
  private readonly ALLOWED_SORT_COLUMNS = [
    'firstName',
    'lastName',
    'email',
    'createdAt',
    'age',
  ] as const;

  async listUsers(
    sortBy?: string,
    sortOrder?: 'ASC' | 'DESC'
  ) {
    const qb = this.dataSource.createQueryBuilder(User, 'user');

    // ✅ SAFE: Whitelist validation for column names
    if (sortBy && this.ALLOWED_SORT_COLUMNS.includes(sortBy as any)) {
      const order = sortOrder === 'DESC' ? 'DESC' : 'ASC';
      qb.orderBy(`user.${sortBy}`, order);
    } else {
      qb.orderBy('user.createdAt', 'DESC'); // Default sort
    }

    return qb.getMany();
  }
}
```

### Parameter Reuse and Naming

```typescript
// Different parameter names for same column
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.age >= :minAge', { minAge: 18 })
  .andWhere('user.age <= :maxAge', { maxAge: 65 })
  .getMany();

// Reusing parameter name (same value)
const searchTerm = 'john';
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.firstName ILIKE :search', { search: `%${searchTerm}%` })
  .orWhere('user.lastName ILIKE :search', { search: `%${searchTerm}%` })
  .orWhere('user.email ILIKE :search', { search: `%${searchTerm}%` })
  .getMany();
// Same :search parameter used multiple times

// Dynamic parameter names for loops
const filters = [
  { field: 'city', value: 'New York' },
  { field: 'country', value: 'USA' },
];

let qb = dataSource
  .createQueryBuilder(User, 'user')
  .where('1=1');

filters.forEach((filter, index) => {
  qb = qb.andWhere(`user.${filter.field} = :value${index}`, {
    [`value${index}`]: filter.value,
  });
});
// Generates unique parameter names: :value0, :value1, etc.
```

### Advanced Parameter Usage

```typescript
// LIKE patterns with parameters
const searchTerm = 'john';
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.email LIKE :pattern', { pattern: `%${searchTerm}%` })
  .getMany();
// Contains

const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.email LIKE :pattern', { pattern: `${searchTerm}%` })
  .getMany();
// Starts with

const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.email LIKE :pattern', { pattern: `%${searchTerm}` })
  .getMany();
// Ends with

// BETWEEN with parameters
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.age BETWEEN :min AND :max', { min: 18, max: 65 })
  .getMany();

// IS NULL (no parameter needed)
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.deletedAt IS NULL')
  .getMany();

// Date parameters
const startDate = new Date('2024-01-01');
const endDate = new Date('2024-12-31');
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.createdAt BETWEEN :start AND :end', {
    start: startDate,
    end: endDate,
  })
  .getMany();
```

### Best Practices

```typescript
// ✅ GOOD: Always use parameters for user input
const userInput = req.body.email;
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.email = :email', { email: userInput })
  .getMany();

// ❌ BAD: Never concatenate user input
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where(`user.email = '${req.body.email}'`)
  .getMany();

// ✅ GOOD: Use spread operator for array parameters
const roles = ['admin', 'moderator'];
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.role IN (:...roles)', { roles })
  .getMany();

// ✅ GOOD: Whitelist dynamic column names
const ALLOWED_COLUMNS = ['firstName', 'email', 'createdAt'];
const sortBy = req.query.sortBy;
if (ALLOWED_COLUMNS.includes(sortBy)) {
  qb.orderBy(`user.${sortBy}`, 'ASC');
}

// ✅ GOOD: Validate and sanitize input before using
const email = req.body.email.trim().toLowerCase();
if (validator.isEmail(email)) {
  const users = await dataSource
    .createQueryBuilder(User, 'user')
    .where('user.email = :email', { email })
    .getMany();
}

// ✅ GOOD: Use TypeScript types for parameter validation
interface SearchParams {
  email?: string;
  minAge?: number;
  roles?: string[];
}

async function searchUsers(params: SearchParams) {
  return dataSource
    .createQueryBuilder(User, 'user')
    .where('user.email = :email', { email: params.email })
    .andWhere('user.age >= :minAge', { minAge: params.minAge })
    .andWhere('user.role IN (:...roles)', { roles: params.roles })
    .getMany();
}

// ✅ GOOD: Log SQL for debugging (in development)
const qb = dataSource
  .createQueryBuilder(User, 'user')
  .where('user.email = :email', { email: userInput });

if (process.env.NODE_ENV === 'development') {
  console.log('SQL:', qb.getSql());
  console.log('Parameters:', qb.getParameters());
}

const users = await qb.getMany();
```

### What NOT to do

```typescript
// ❌ NEVER: String interpolation/concatenation
const email = req.body.email;
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where(`user.email = '${email}'`) // VULNERABLE!
  .getMany();

// ❌ NEVER: Template literals with user input
const query = `user.firstName LIKE '%${req.body.search}%'`;
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where(query) // VULNERABLE!
  .getMany();

// ❌ NEVER: Dynamic column names without whitelist
const column = req.query.sortBy; // Could be: "id; DROP TABLE users; --"
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .orderBy(`user.${column}`, 'ASC') // VULNERABLE!
  .getMany();

// ❌ NEVER: Raw SQL with user input
const users = await dataSource.query(
  `SELECT * FROM users WHERE email = '${req.body.email}'` // VULNERABLE!
);

// ❌ NEVER: Disabling parameter escaping
// TypeORM doesn't have this, but be aware in other ORMs
```

**Interview Tip**: Explain **parameterized queries** prevent SQL injection by separating SQL code from user data. Use **:paramName** syntax (`:email`, `:minAge`) and pass values in parameter object `{ email, minAge }`. TypeORM **automatically escapes** parameters, making queries safe. Emphasize **never concatenate user input** into SQL strings (template literals, string concatenation). Mention special cases: use **...spread** for arrays (`:...roles`), **whitelist** dynamic column names (ORDER BY, GROUP BY), validate input before querying. For production: always use parameters for user input, whitelist column names for dynamic queries, validate/sanitize input first, log SQL in development to verify safety, use TypeScript types for parameter validation, prefer object-style where() for simple equality checks (auto-parameterized), never trust user input directly in SQL.

</details>

<details>
<summary>71. How do you implement pagination with QueryBuilder?</summary>

Use **skip()** and **take()** (or **offset()** and **limit()**) methods for pagination. Combine with **getManyAndCount()** to get total count for calculating pages. TypeORM provides both offset-based pagination and cursor-based pagination options.

### Basic Pagination

```typescript
import { DataSource } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  email: string;

  @Column()
  createdAt: Date;
}

const dataSource = new DataSource(/* config */);

// Basic pagination with skip/take
const page = 2; // Current page (1-indexed)
const pageSize = 10; // Items per page

const users = await dataSource
  .createQueryBuilder(User, 'user')
  .orderBy('user.createdAt', 'DESC')
  .skip((page - 1) * pageSize) // Skip first 10 items
  .take(pageSize) // Take 10 items
  .getMany();

// Alternative: offset/limit (same as skip/take)
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .orderBy('user.createdAt', 'DESC')
  .offset((page - 1) * pageSize)
  .limit(pageSize)
  .getMany();
```

### Pagination with Total Count

```typescript
// Get items and total count in one query
const page = 1;
const pageSize = 10;

const [users, total] = await dataSource
  .createQueryBuilder(User, 'user')
  .orderBy('user.createdAt', 'DESC')
  .skip((page - 1) * pageSize)
  .take(pageSize)
  .getManyAndCount();

// Calculate pagination metadata
const totalPages = Math.ceil(total / pageSize);
const hasNextPage = page < totalPages;
const hasPreviousPage = page > 1;

console.log({
  data: users,
  page,
  pageSize,
  total,
  totalPages,
  hasNextPage,
  hasPreviousPage,
});
```

### Complete Pagination Service

```typescript
interface PaginationOptions {
  page: number;
  pageSize: number;
}

interface PaginatedResult<T> {
  data: T[];
  page: number;
  pageSize: number;
  total: number;
  totalPages: number;
  hasNextPage: boolean;
  hasPreviousPage: boolean;
}

export class UserService {
  async getUsers(options: PaginationOptions): Promise<PaginatedResult<User>> {
    const { page = 1, pageSize = 10 } = options;

    // Validate pagination parameters
    const validPage = Math.max(1, page);
    const validPageSize = Math.min(Math.max(1, pageSize), 100); // Max 100 items

    const [data, total] = await this.dataSource
      .createQueryBuilder(User, 'user')
      .where('user.isActive = :active', { active: true })
      .orderBy('user.createdAt', 'DESC')
      .skip((validPage - 1) * validPageSize)
      .take(validPageSize)
      .getManyAndCount();

    const totalPages = Math.ceil(total / validPageSize);

    return {
      data,
      page: validPage,
      pageSize: validPageSize,
      total,
      totalPages,
      hasNextPage: validPage < totalPages,
      hasPreviousPage: validPage > 1,
    };
  }
}
```

### Pagination with Filters and Search

```typescript
export class UserService {
  async searchUsers(
    query: string,
    filters: { role?: string; minAge?: number },
    pagination: { page: number; pageSize: number }
  ): Promise<PaginatedResult<User>> {
    const { page = 1, pageSize = 10 } = pagination;

    let qb = this.dataSource
      .createQueryBuilder(User, 'user')
      .where('user.isActive = :active', { active: true });

    // Apply search
    if (query) {
      qb = qb.andWhere(
        new Brackets((subQb) => {
          subQb
            .where('user.firstName ILIKE :query', { query: `%${query}%` })
            .orWhere('user.lastName ILIKE :query', { query: `%${query}%` })
            .orWhere('user.email ILIKE :query', { query: `%${query}%` });
        })
      );
    }

    // Apply filters
    if (filters.role) {
      qb = qb.andWhere('user.role = :role', { role: filters.role });
    }

    if (filters.minAge) {
      qb = qb.andWhere('user.age >= :minAge', { minAge: filters.minAge });
    }

    // Apply pagination
    const [data, total] = await qb
      .orderBy('user.createdAt', 'DESC')
      .skip((page - 1) * pageSize)
      .take(pageSize)
      .getManyAndCount();

    return {
      data,
      page,
      pageSize,
      total,
      totalPages: Math.ceil(total / pageSize),
      hasNextPage: page < Math.ceil(total / pageSize),
      hasPreviousPage: page > 1,
    };
  }
}
```

### Cursor-Based Pagination

```typescript
// Better for large datasets, no skip overhead
interface CursorPaginationOptions {
  limit: number;
  cursor?: number; // Last seen ID
}

interface CursorPaginatedResult<T> {
  data: T[];
  nextCursor: number | null;
  hasMore: boolean;
}

export class UserService {
  async getUsersCursor(
    options: CursorPaginationOptions
  ): Promise<CursorPaginatedResult<User>> {
    const { limit = 10, cursor } = options;
    const validLimit = Math.min(Math.max(1, limit), 100);

    let qb = this.dataSource
      .createQueryBuilder(User, 'user')
      .where('user.isActive = :active', { active: true })
      .orderBy('user.id', 'DESC')
      .take(validLimit + 1); // Fetch one extra to check if more exists

    // Apply cursor
    if (cursor) {
      qb = qb.andWhere('user.id < :cursor', { cursor });
    }

    const users = await qb.getMany();
    const hasMore = users.length > validLimit;
    const data = hasMore ? users.slice(0, validLimit) : users;
    const nextCursor = hasMore ? data[data.length - 1].id : null;

    return {
      data,
      nextCursor,
      hasMore,
    };
  }
}
```

### REST API Pagination Example

```typescript
import { Controller, Get, Query } from '@nestjs/common';

@Controller('users')
export class UserController {
  constructor(private userService: UserService) {}

  @Get()
  async getUsers(
    @Query('page') page: number = 1,
    @Query('pageSize') pageSize: number = 10,
    @Query('search') search?: string,
    @Query('role') role?: string
  ) {
    const result = await this.userService.searchUsers(
      search,
      { role },
      { page: Number(page), pageSize: Number(pageSize) }
    );

    return {
      ...result,
      links: {
        self: `/users?page=${result.page}&pageSize=${result.pageSize}`,
        first: `/users?page=1&pageSize=${result.pageSize}`,
        last: `/users?page=${result.totalPages}&pageSize=${result.pageSize}`,
        ...(result.hasNextPage && {
          next: `/users?page=${result.page + 1}&pageSize=${result.pageSize}`,
        }),
        ...(result.hasPreviousPage && {
          prev: `/users?page=${result.page - 1}&pageSize=${result.pageSize}`,
        }),
      },
    };
  }

  // Cursor-based endpoint
  @Get('feed')
  async getUserFeed(
    @Query('limit') limit: number = 20,
    @Query('cursor') cursor?: number
  ) {
    return this.userService.getUsersCursor({
      limit: Number(limit),
      cursor: cursor ? Number(cursor) : undefined,
    });
  }
}
```

### Pagination with Joins

```typescript
// Pagination with relations (be careful with duplicates)
export class UserService {
  async getUsersWithPosts(
    page: number = 1,
    pageSize: number = 10
  ): Promise<PaginatedResult<User>> {
    // Step 1: Get paginated user IDs first
    const [userIds, total] = await this.dataSource
      .createQueryBuilder(User, 'user')
      .select('user.id')
      .orderBy('user.createdAt', 'DESC')
      .skip((page - 1) * pageSize)
      .take(pageSize)
      .getManyAndCount();

    if (userIds.length === 0) {
      return {
        data: [],
        page,
        pageSize,
        total,
        totalPages: 0,
        hasNextPage: false,
        hasPreviousPage: false,
      };
    }

    // Step 2: Load users with relations
    const ids = userIds.map((u) => u.id);
    const data = await this.dataSource
      .createQueryBuilder(User, 'user')
      .leftJoinAndSelect('user.posts', 'post')
      .leftJoinAndSelect('user.profile', 'profile')
      .where('user.id IN (:...ids)', { ids })
      .orderBy('user.createdAt', 'DESC')
      .getMany();

    return {
      data,
      page,
      pageSize,
      total,
      totalPages: Math.ceil(total / pageSize),
      hasNextPage: page < Math.ceil(total / pageSize),
      hasPreviousPage: page > 1,
    };
  }
}
```

### Performance Optimization

```typescript
// Cache total count for better performance
export class UserService {
  private countCache: Map<string, { count: number; timestamp: number }> = new Map();
  private CACHE_TTL = 60000; // 1 minute

  async getUsersOptimized(
    page: number = 1,
    pageSize: number = 10
  ): Promise<PaginatedResult<User>> {
    const cacheKey = 'users:total';
    const cached = this.countCache.get(cacheKey);
    const now = Date.now();

    let total: number;

    if (cached && now - cached.timestamp < this.CACHE_TTL) {
      // Use cached count
      total = cached.count;
    } else {
      // Get fresh count
      total = await this.dataSource
        .createQueryBuilder(User, 'user')
        .where('user.isActive = :active', { active: true })
        .getCount();
      
      this.countCache.set(cacheKey, { count: total, timestamp: now });
    }

    // Get data without count (faster)
    const data = await this.dataSource
      .createQueryBuilder(User, 'user')
      .where('user.isActive = :active', { active: true })
      .orderBy('user.createdAt', 'DESC')
      .skip((page - 1) * pageSize)
      .take(pageSize)
      .getMany();

    return {
      data,
      page,
      pageSize,
      total,
      totalPages: Math.ceil(total / pageSize),
      hasNextPage: page < Math.ceil(total / pageSize),
      hasPreviousPage: page > 1,
    };
  }
}
```

### Best Practices

```typescript
// ✅ GOOD: Validate pagination parameters
function validatePagination(page: number, pageSize: number) {
  return {
    page: Math.max(1, page || 1),
    pageSize: Math.min(Math.max(1, pageSize || 10), 100), // Max 100
  };
}

// ✅ GOOD: Always include ORDER BY for consistent results
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .orderBy('user.id', 'ASC') // Consistent ordering
  .skip(offset)
  .take(limit)
  .getMany();

// ❌ BAD: Pagination without ORDER BY (inconsistent results)
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .skip(offset)
  .take(limit)
  .getMany();

// ✅ GOOD: Use cursor pagination for large datasets
const cursorResult = await userService.getUsersCursor({ limit: 20, cursor: lastId });

// ✅ GOOD: Provide pagination metadata in response
return {
  data: users,
  pagination: {
    page,
    pageSize,
    total,
    totalPages,
    hasNextPage,
    hasPreviousPage,
  },
};

// ✅ GOOD: Paginate IDs first when using JOINs
const ids = await qb.select('user.id').skip(offset).take(limit).getMany();
const data = await qb.whereInIds(ids.map((u) => u.id)).getMany();

// ✅ GOOD: Set reasonable max page size
const MAX_PAGE_SIZE = 100;
const pageSize = Math.min(requestedPageSize, MAX_PAGE_SIZE);
```

**Interview Tip**: Explain **pagination** uses **skip()** and **take()** (or **offset()**/**limit()**) to retrieve subset of results. Use **getManyAndCount()** to get both data and total count for calculating pages. Mention **offset-based** (skip/take) for simple pagination, **cursor-based** (WHERE id < cursor) for large datasets (no skip overhead). Emphasize best practices: always include **ORDER BY** for consistent results, validate page parameters (max page size), paginate IDs first when using JOINs (avoid duplicate rows), cache total count for performance. For production: return pagination metadata (page, total, hasNext), use cursor pagination for infinite scroll, validate/sanitize pagination params, use two-step loading for complex joins, consider caching count queries, provide HATEOAS links in REST APIs, set reasonable max page size (100).

</details>

<details>
<summary>72. How do you use ORDER BY with QueryBuilder?</summary>

Use **orderBy()** to set the primary sort order and **addOrderBy()** to add additional sort columns. TypeORM supports ascending (`ASC`), descending (`DESC`), and `NULLS FIRST`/`NULLS LAST` options. You can sort by entity columns, computed expressions, or joined table columns.

### Basic ORDER BY

```typescript
import { DataSource } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column()
  email: string;

  @Column()
  age: number;

  @Column()
  createdAt: Date;
}

const dataSource = new DataSource(/* config */);

// Single column, ascending
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .orderBy('user.firstName', 'ASC')
  .getMany();
// SQL: ORDER BY user.firstName ASC

// Single column, descending
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .orderBy('user.createdAt', 'DESC')
  .getMany();
// SQL: ORDER BY user.createdAt DESC

// Default order (ASC if not specified)
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .orderBy('user.firstName')
  .getMany();
// SQL: ORDER BY user.firstName ASC
```

### Multiple ORDER BY Columns

```typescript
// Using addOrderBy for multiple columns
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .orderBy('user.lastName', 'ASC')
  .addOrderBy('user.firstName', 'ASC')
  .addOrderBy('user.id', 'ASC')
  .getMany();
// SQL: ORDER BY user.lastName ASC, user.firstName ASC, user.id ASC

// ⚠️ WARNING: orderBy() replaces previous order
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .orderBy('user.lastName', 'ASC')
  .orderBy('user.firstName', 'ASC') // REPLACES previous orderBy!
  .getMany();
// SQL: ORDER BY user.firstName ASC (lastName is lost!)

// ✅ CORRECT: Use orderBy once, then addOrderBy
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .orderBy('user.lastName', 'ASC')
  .addOrderBy('user.firstName', 'ASC')
  .addOrderBy('user.createdAt', 'DESC')
  .getMany();
```

### ORDER BY with NULLS Handling

```typescript
// NULLS FIRST (PostgreSQL)
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .orderBy('user.deletedAt', 'ASC', 'NULLS FIRST')
  .getMany();
// SQL: ORDER BY user.deletedAt ASC NULLS FIRST

// NULLS LAST
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .orderBy('user.deletedAt', 'ASC', 'NULLS LAST')
  .getMany();
// SQL: ORDER BY user.deletedAt ASC NULLS LAST

// With multiple columns
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .orderBy('user.lastName', 'ASC', 'NULLS LAST')
  .addOrderBy('user.firstName', 'ASC', 'NULLS LAST')
  .getMany();
```

### ORDER BY Computed Expressions

```typescript
// Order by calculated field
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .orderBy("CONCAT(user.firstName, ' ', user.lastName)", 'ASC')
  .getMany();

// Order by expression with alias
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user')
  .addSelect("CONCAT(user.firstName, ' ', user.lastName)", 'fullName')
  .orderBy('fullName', 'ASC')
  .getMany();

// Order by CASE statement
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .orderBy(
    `CASE 
      WHEN user.role = 'admin' THEN 1 
      WHEN user.role = 'moderator' THEN 2 
      ELSE 3 
    END`,
    'ASC'
  )
  .addOrderBy('user.firstName', 'ASC')
  .getMany();
// Admins first, then moderators, then others
```

### ORDER BY with JOINs

```typescript
@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToOne(() => User, (user) => user.posts)
  author: User;

  @Column()
  viewCount: number;

  @Column()
  createdAt: Date;
}

// Order by joined table column
const posts = await dataSource
  .createQueryBuilder(Post, 'post')
  .leftJoinAndSelect('post.author', 'author')
  .orderBy('author.firstName', 'ASC')
  .addOrderBy('post.createdAt', 'DESC')
  .getMany();
// SQL: ORDER BY author.firstName ASC, post.createdAt DESC

// Order parent and children separately
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoinAndSelect('user.posts', 'post')
  .orderBy('user.createdAt', 'DESC') // Order users
  .addOrderBy('post.viewCount', 'DESC') // Order posts within each user
  .getMany();
```

### Dynamic ORDER BY

```typescript
export class UserService {
  private readonly ALLOWED_SORT_COLUMNS = [
    'firstName',
    'lastName',
    'email',
    'age',
    'createdAt',
  ] as const;

  async getUsers(
    sortBy?: string,
    sortOrder?: 'ASC' | 'DESC'
  ): Promise<User[]> {
    const qb = this.dataSource.createQueryBuilder(User, 'user');

    // Validate sort column (whitelist)
    if (sortBy && this.ALLOWED_SORT_COLUMNS.includes(sortBy as any)) {
      const order = sortOrder === 'DESC' ? 'DESC' : 'ASC';
      qb.orderBy(`user.${sortBy}`, order);
    } else {
      // Default sort
      qb.orderBy('user.createdAt', 'DESC');
    }

    return qb.getMany();
  }

  // Multiple dynamic sorts
  async getUsersAdvanced(sorts: Array<{ column: string; order: 'ASC' | 'DESC' }>) {
    let qb = this.dataSource.createQueryBuilder(User, 'user');
    let isFirst = true;

    for (const sort of sorts) {
      if (this.ALLOWED_SORT_COLUMNS.includes(sort.column as any)) {
        if (isFirst) {
          qb = qb.orderBy(`user.${sort.column}`, sort.order);
          isFirst = false;
        } else {
          qb = qb.addOrderBy(`user.${sort.column}`, sort.order);
        }
      }
    }

    // Fallback if no valid sorts
    if (isFirst) {
      qb = qb.orderBy('user.id', 'ASC');
    }

    return qb.getMany();
  }
}
```

### Real-World Examples

```typescript
// User listing with default sort
export class UserService {
  async listUsers(): Promise<User[]> {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .where('user.isActive = :active', { active: true })
      .orderBy('user.lastName', 'ASC')
      .addOrderBy('user.firstName', 'ASC')
      .getMany();
  }

  // Recent users first
  async getRecentUsers(limit: number = 10): Promise<User[]> {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .orderBy('user.createdAt', 'DESC')
      .take(limit)
      .getMany();
  }

  // Top users by activity
  async getTopUsers(limit: number = 10): Promise<User[]> {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .leftJoin('user.posts', 'post')
      .select('user')
      .addSelect('COUNT(post.id)', 'postCount')
      .groupBy('user.id')
      .orderBy('postCount', 'DESC')
      .addOrderBy('user.createdAt', 'DESC')
      .take(limit)
      .getMany();
  }

  // Priority-based sorting
  async getUsersByPriority(): Promise<User[]> {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .orderBy(
        `CASE 
          WHEN user.role = 'admin' THEN 1
          WHEN user.role = 'moderator' THEN 2
          WHEN user.isPremium = true THEN 3
          ELSE 4
        END`,
        'ASC'
      )
      .addOrderBy('user.lastName', 'ASC')
      .addOrderBy('user.firstName', 'ASC')
      .getMany();
  }
}
```

### REST API with Dynamic Sorting

```typescript
import { Controller, Get, Query } from '@nestjs/common';

@Controller('users')
export class UserController {
  constructor(private userService: UserService) {}

  @Get()
  async getUsers(
    @Query('sortBy') sortBy?: string,
    @Query('order') order?: 'ASC' | 'DESC'
  ) {
    return this.userService.getUsers(sortBy, order);
  }

  // Multiple sorts: ?sort=lastName:asc&sort=firstName:asc
  @Get('advanced')
  async getUsersAdvanced(@Query('sort') sortParams?: string | string[]) {
    const sorts = [];
    
    if (sortParams) {
      const sortArray = Array.isArray(sortParams) ? sortParams : [sortParams];
      
      for (const param of sortArray) {
        const [column, order] = param.split(':');
        sorts.push({
          column,
          order: (order?.toUpperCase() === 'DESC' ? 'DESC' : 'ASC') as 'ASC' | 'DESC',
        });
      }
    }

    return this.userService.getUsersAdvanced(sorts);
  }
}
```

### Best Practices

```typescript
// ✅ GOOD: Always specify sort direction explicitly
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .orderBy('user.createdAt', 'DESC')
  .getMany();

// ✅ GOOD: Use orderBy once, then addOrderBy
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .orderBy('user.lastName', 'ASC')
  .addOrderBy('user.firstName', 'ASC')
  .getMany();

// ❌ BAD: Multiple orderBy calls (replaces previous)
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .orderBy('user.lastName', 'ASC')
  .orderBy('user.firstName', 'ASC') // REPLACES lastName!
  .getMany();

// ✅ GOOD: Whitelist columns for dynamic sorting
const ALLOWED_COLUMNS = ['firstName', 'email', 'createdAt'];
if (ALLOWED_COLUMNS.includes(sortBy)) {
  qb.orderBy(`user.${sortBy}`, sortOrder);
}

// ❌ BAD: Unvalidated dynamic column (SQL injection risk)
qb.orderBy(`user.${req.query.sortBy}`, 'ASC'); // Dangerous!

// ✅ GOOD: Include stable sort column (id) as last resort
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .orderBy('user.lastName', 'ASC')
  .addOrderBy('user.firstName', 'ASC')
  .addOrderBy('user.id', 'ASC') // Ensures consistent order
  .getMany();

// ✅ GOOD: Use NULLS handling for nullable columns
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .orderBy('user.deletedAt', 'DESC', 'NULLS LAST')
  .getMany();

// ✅ GOOD: Order joined data appropriately
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoinAndSelect('user.posts', 'post')
  .orderBy('user.createdAt', 'DESC') // Parent order
  .addOrderBy('post.createdAt', 'DESC') // Child order
  .getMany();
```

**Interview Tip**: Explain **orderBy()** sets primary sort, **addOrderBy()** adds secondary sorts. Emphasize **orderBy() replaces previous order** (common mistake), use orderBy once then addOrderBy. Mention sort directions: ASC (ascending), DESC (descending), NULLS FIRST/LAST (PostgreSQL). Highlight dynamic sorting: **whitelist columns** to prevent SQL injection, validate sort direction, provide default sort. For production: always specify sort direction explicitly, use addOrderBy for multiple columns, include stable sort column (id) for pagination consistency, whitelist dynamic column names, handle NULL values with NULLS FIRST/LAST, order both parent and child in joins, provide sensible defaults, validate user input for sort params, use CASE expressions for priority sorting.

</details>

<details>
<summary>73. How do you use GROUP BY and HAVING clauses?</summary>

Use **groupBy()** to group rows by columns and **having()** to filter grouped results. **GROUP BY** aggregates data, **HAVING** filters aggregated results (unlike WHERE which filters before aggregation). Use **addGroupBy()** for multiple columns, similar to addOrderBy().

### Basic GROUP BY

```typescript
import { DataSource } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  city: string;

  @Column()
  country: string;

  @Column()
  age: number;

  @OneToMany(() => Post, (post) => post.author)
  posts: Post[];
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToOne(() => User, (user) => user.posts)
  author: User;
}

const dataSource = new DataSource(/* config */);

// Count users by city
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city', 'city')
  .addSelect('COUNT(*)', 'userCount')
  .groupBy('user.city')
  .getRawMany();
// Result: [{ city: 'New York', userCount: '150' }, { city: 'LA', userCount: '80' }, ...]
// SQL: SELECT user.city, COUNT(*) FROM users user GROUP BY user.city

// Average age by city
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city', 'city')
  .addSelect('AVG(user.age)', 'averageAge')
  .addSelect('COUNT(*)', 'userCount')
  .groupBy('user.city')
  .getRawMany();
```

### Multiple GROUP BY Columns

```typescript
// Group by multiple columns using addGroupBy
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city', 'city')
  .addSelect('user.country', 'country')
  .addSelect('COUNT(*)', 'userCount')
  .groupBy('user.city')
  .addGroupBy('user.country')
  .getRawMany();
// SQL: GROUP BY user.city, user.country

// ⚠️ WARNING: groupBy() replaces previous GROUP BY
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city', 'city')
  .addSelect('COUNT(*)', 'count')
  .groupBy('user.city')
  .groupBy('user.country') // REPLACES city!
  .getRawMany();
// SQL: GROUP BY user.country (city is lost!)

// ✅ CORRECT: Use groupBy once, then addGroupBy
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city', 'city')
  .addSelect('user.country', 'country')
  .addSelect('COUNT(*)', 'count')
  .groupBy('user.city')
  .addGroupBy('user.country')
  .getRawMany();
```

### HAVING Clause

```typescript
// HAVING filters aggregated results
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city', 'city')
  .addSelect('COUNT(*)', 'userCount')
  .groupBy('user.city')
  .having('COUNT(*) > :minCount', { minCount: 50 })
  .getRawMany();
// SQL: SELECT user.city, COUNT(*) FROM users GROUP BY user.city HAVING COUNT(*) > 50
// Only cities with more than 50 users

// Multiple HAVING conditions with andHaving
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city', 'city')
  .addSelect('COUNT(*)', 'userCount')
  .addSelect('AVG(user.age)', 'averageAge')
  .groupBy('user.city')
  .having('COUNT(*) > :minCount', { minCount: 50 })
  .andHaving('AVG(user.age) >= :minAge', { minAge: 25 })
  .getRawMany();
// Cities with 50+ users AND average age >= 25

// HAVING with OR conditions
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city', 'city')
  .addSelect('COUNT(*)', 'userCount')
  .groupBy('user.city')
  .having('COUNT(*) > :highCount', { highCount: 100 })
  .orHaving('COUNT(*) < :lowCount', { lowCount: 5 })
  .getRawMany();
// Very large or very small cities
```

### WHERE vs HAVING

```typescript
// WHERE filters BEFORE grouping, HAVING filters AFTER
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city', 'city')
  .addSelect('COUNT(*)', 'activeUserCount')
  .where('user.isActive = :active', { active: true }) // Filter before grouping
  .groupBy('user.city')
  .having('COUNT(*) >= :minCount', { minCount: 10 }) // Filter after grouping
  .getRawMany();
// SQL: SELECT user.city, COUNT(*)
//      FROM users user
//      WHERE user.isActive = true
//      GROUP BY user.city
//      HAVING COUNT(*) >= 10
// Process: Filter active users → Group by city → Filter cities with 10+ users

/*
Key Difference:
- WHERE: Filters individual rows BEFORE aggregation
- HAVING: Filters aggregated results AFTER GROUP BY
*/
```

### GROUP BY with JOINs

```typescript
// Count posts per user
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoin('user.posts', 'post')
  .select('user.id', 'userId')
  .addSelect('user.firstName', 'name')
  .addSelect('COUNT(post.id)', 'postCount')
  .groupBy('user.id')
  .getRawMany();
// SQL: SELECT user.id, user.firstName, COUNT(post.id)
//      FROM users user LEFT JOIN posts post ON post.authorId = user.id
//      GROUP BY user.id

// With HAVING to filter users
const activeAuthors = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoin('user.posts', 'post')
  .select('user.id', 'userId')
  .addSelect('user.firstName', 'name')
  .addSelect('COUNT(post.id)', 'postCount')
  .groupBy('user.id')
  .having('COUNT(post.id) >= :minPosts', { minPosts: 5 })
  .orderBy('postCount', 'DESC')
  .getRawMany();
// Only users with 5+ posts
```

### Real-World Examples

```typescript
// Dashboard statistics by category
export class AnalyticsService {
  async getUserStatsByCity() {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .select('user.city', 'city')
      .addSelect('COUNT(*)', 'totalUsers')
      .addSelect('COUNT(CASE WHEN user.isActive = true THEN 1 END)', 'activeUsers')
      .addSelect('AVG(user.age)', 'averageAge')
      .addSelect('MIN(user.createdAt)', 'firstUserDate')
      .addSelect('MAX(user.createdAt)', 'lastUserDate')
      .groupBy('user.city')
      .having('COUNT(*) >= :minUsers', { minUsers: 10 })
      .orderBy('totalUsers', 'DESC')
      .getRawMany();
  }

  // Top contributors
  async getTopContributors(minPosts: number = 10) {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .leftJoin('user.posts', 'post')
      .leftJoin('post.comments', 'comment')
      .select('user.id', 'userId')
      .addSelect('user.firstName', 'name')
      .addSelect('user.email', 'email')
      .addSelect('COUNT(DISTINCT post.id)', 'totalPosts')
      .addSelect('COUNT(DISTINCT comment.id)', 'totalComments')
      .addSelect('MAX(post.createdAt)', 'lastPostDate')
      .groupBy('user.id')
      .having('COUNT(DISTINCT post.id) >= :minPosts', { minPosts })
      .orderBy('totalPosts', 'DESC')
      .addOrderBy('totalComments', 'DESC')
      .take(50)
      .getRawMany();
  }

  // Monthly user registration stats
  async getMonthlySignups(year: number) {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .select("TO_CHAR(user.createdAt, 'YYYY-MM')", 'month')
      .addSelect('COUNT(*)', 'signups')
      .addSelect('COUNT(CASE WHEN user.isPremium = true THEN 1 END)', 'premiumSignups')
      .where("EXTRACT(YEAR FROM user.createdAt) = :year", { year })
      .groupBy('month')
      .orderBy('month', 'ASC')
      .getRawMany();
  }

  // Age group analysis
  async getUsersByAgeGroup() {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .select(
        `CASE 
          WHEN user.age < 18 THEN 'Under 18'
          WHEN user.age BETWEEN 18 AND 24 THEN '18-24'
          WHEN user.age BETWEEN 25 AND 34 THEN '25-34'
          WHEN user.age BETWEEN 35 AND 44 THEN '35-44'
          WHEN user.age BETWEEN 45 AND 54 THEN '45-54'
          WHEN user.age >= 55 THEN '55+'
        END`,
        'ageGroup'
      )
      .addSelect('COUNT(*)', 'userCount')
      .addSelect('AVG(user.age)', 'avgAge')
      .groupBy('ageGroup')
      .orderBy('MIN(user.age)', 'ASC')
      .getRawMany();
  }
}
```

### Advanced GROUP BY Scenarios

```typescript
// Multiple aggregations with complex HAVING
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoin('user.posts', 'post')
  .select('user.city', 'city')
  .addSelect('COUNT(DISTINCT user.id)', 'userCount')
  .addSelect('COUNT(post.id)', 'totalPosts')
  .addSelect('AVG(user.age)', 'avgAge')
  .groupBy('user.city')
  .having('COUNT(DISTINCT user.id) >= :minUsers', { minUsers: 50 })
  .andHaving('COUNT(post.id) >= :minPosts', { minPosts: 100 })
  .andHaving('AVG(user.age) BETWEEN :minAge AND :maxAge', { 
    minAge: 25, 
    maxAge: 45 
  })
  .orderBy('totalPosts', 'DESC')
  .getRawMany();

// GROUP BY with subquery in HAVING
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoin('user.posts', 'post')
  .select('user.id', 'userId')
  .addSelect('user.firstName', 'name')
  .addSelect('COUNT(post.id)', 'postCount')
  .groupBy('user.id')
  .having((qb) => {
    const subQuery = qb
      .subQuery()
      .select('AVG(postCount)')
      .from((subQb) => {
        return subQb
          .select('COUNT(p.id)', 'postCount')
          .from(Post, 'p')
          .groupBy('p.authorId');
      }, 'avgCounts')
      .getQuery();
    return `COUNT(post.id) > ${subQuery}`;
  })
  .getRawMany();
// Users with above-average post counts
```

### GROUP BY with Date Functions

```typescript
// Daily statistics
const dailyStats = await dataSource
  .createQueryBuilder(User, 'user')
  .select("DATE(user.createdAt)", 'date')
  .addSelect('COUNT(*)', 'signups')
  .groupBy('date')
  .orderBy('date', 'DESC')
  .take(30)
  .getRawMany();

// Weekly statistics (PostgreSQL)
const weeklyStats = await dataSource
  .createQueryBuilder(User, 'user')
  .select("DATE_TRUNC('week', user.createdAt)", 'week')
  .addSelect('COUNT(*)', 'signups')
  .where('user.createdAt > :date', { date: thirtyDaysAgo })
  .groupBy('week')
  .orderBy('week', 'DESC')
  .getRawMany();

// Monthly statistics
const monthlyStats = await dataSource
  .createQueryBuilder(User, 'user')
  .select("TO_CHAR(user.createdAt, 'YYYY-MM')", 'month')
  .addSelect('COUNT(*)', 'signups')
  .addSelect('AVG(user.age)', 'avgAge')
  .groupBy('month')
  .having('COUNT(*) > :minSignups', { minSignups: 10 })
  .orderBy('month', 'DESC')
  .getRawMany();
```

### Best Practices

```typescript
// ✅ GOOD: Use groupBy once, then addGroupBy
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city', 'city')
  .addSelect('user.country', 'country')
  .addSelect('COUNT(*)', 'count')
  .groupBy('user.city')
  .addGroupBy('user.country')
  .getRawMany();

// ❌ BAD: Multiple groupBy calls (replaces previous)
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .groupBy('user.city')
  .groupBy('user.country') // REPLACES city!
  .getRawMany();

// ✅ GOOD: Include all non-aggregated columns in GROUP BY
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city', 'city')
  .addSelect('user.country', 'country')
  .addSelect('COUNT(*)', 'count')
  .groupBy('user.city')
  .addGroupBy('user.country') // Both selected columns in GROUP BY
  .getRawMany();

// ❌ BAD: Selected column not in GROUP BY
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city', 'city')
  .addSelect('user.country', 'country') // Not in GROUP BY!
  .addSelect('COUNT(*)', 'count')
  .groupBy('user.city')
  .getRawMany();
// This will cause SQL error in most databases

// ✅ GOOD: Use HAVING for aggregated filters
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city', 'city')
  .addSelect('COUNT(*)', 'count')
  .groupBy('user.city')
  .having('COUNT(*) >= :min', { min: 10 })
  .getRawMany();

// ✅ GOOD: Use WHERE for non-aggregated filters
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city', 'city')
  .addSelect('COUNT(*)', 'count')
  .where('user.isActive = :active', { active: true })
  .groupBy('user.city')
  .getRawMany();

// ✅ GOOD: Order by aggregated columns
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city', 'city')
  .addSelect('COUNT(*)', 'count')
  .groupBy('user.city')
  .orderBy('count', 'DESC') // Order by alias
  .getRawMany();

// ✅ GOOD: Use getRawMany() with GROUP BY
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city', 'city')
  .addSelect('COUNT(*)', 'count')
  .groupBy('user.city')
  .getRawMany(); // Returns plain objects with selected columns
```

**Interview Tip**: Explain **GROUP BY** aggregates rows by columns using **groupBy()**, **HAVING** filters aggregated results using **having()**. Emphasize difference: **WHERE filters before grouping** (individual rows), **HAVING filters after grouping** (aggregated results). Mention **groupBy() replaces previous GROUP BY** (use addGroupBy for multiple columns). Highlight rules: all non-aggregated SELECT columns must be in GROUP BY, use getRawMany() for results, order by aggregated columns with aliases. For production: use groupBy once then addGroupBy, include all selected columns in GROUP BY, use WHERE for row filters (before grouping), use HAVING for aggregate filters (after grouping), always use parameters in HAVING conditions, order by aggregated columns for meaningful results, consider performance on large datasets (indexes on GROUP BY columns).

</details>

<details>
<summary>74. How do you perform aggregations (COUNT, SUM, AVG) with QueryBuilder?</summary>

Use SQL aggregate functions in **select()** or **addSelect()**: **COUNT()**, **SUM()**, **AVG()**, **MIN()**, **MAX()**. Combine with **groupBy()** for grouped aggregations. Use **getRawOne()** for single results, **getRawMany()** for grouped results.

### Basic Aggregations

```typescript
import { DataSource } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  age: number;

  @Column()
  salary: number;

  @Column({ default: true })
  isActive: boolean;

  @OneToMany(() => Post, (post) => post.author)
  posts: Post[];
}

const dataSource = new DataSource(/* config */);

// COUNT - Total users
const result = await dataSource
  .createQueryBuilder(User, 'user')
  .select('COUNT(*)', 'total')
  .getRawOne();
// Result: { total: '150' }

// COUNT with condition
const result = await dataSource
  .createQueryBuilder(User, 'user')
  .select('COUNT(*)', 'activeUsers')
  .where('user.isActive = :active', { active: true })
  .getRawOne();

// COUNT DISTINCT
const result = await dataSource
  .createQueryBuilder(User, 'user')
  .select('COUNT(DISTINCT user.city)', 'uniqueCities')
  .getRawOne();

// SUM - Total salary
const result = await dataSource
  .createQueryBuilder(User, 'user')
  .select('SUM(user.salary)', 'totalSalary')
  .getRawOne();
// Result: { totalSalary: '5000000' }

// AVG - Average age
const result = await dataSource
  .createQueryBuilder(User, 'user')
  .select('AVG(user.age)', 'averageAge')
  .getRawOne();
// Result: { averageAge: '32.5' }

// MIN and MAX
const result = await dataSource
  .createQueryBuilder(User, 'user')
  .select('MIN(user.age)', 'minAge')
  .addSelect('MAX(user.age)', 'maxAge')
  .getRawOne();
// Result: { minAge: '18', maxAge: '75' }
```

### Multiple Aggregations

```typescript
// Combine multiple aggregate functions
const stats = await dataSource
  .createQueryBuilder(User, 'user')
  .select('COUNT(*)', 'totalUsers')
  .addSelect('COUNT(CASE WHEN user.isActive = true THEN 1 END)', 'activeUsers')
  .addSelect('AVG(user.age)', 'averageAge')
  .addSelect('MIN(user.age)', 'minAge')
  .addSelect('MAX(user.age)', 'maxAge')
  .addSelect('SUM(user.salary)', 'totalSalary')
  .addSelect('AVG(user.salary)', 'averageSalary')
  .getRawOne();
// Result: {
//   totalUsers: '150',
//   activeUsers: '120',
//   averageAge: '32.5',
//   minAge: '18',
//   maxAge: '75',
//   totalSalary: '5000000',
//   averageSalary: '33333.33'
// }
```

### Aggregations with GROUP BY

```typescript
// Count users by city
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city', 'city')
  .addSelect('COUNT(*)', 'userCount')
  .addSelect('AVG(user.age)', 'averageAge')
  .addSelect('SUM(user.salary)', 'totalSalary')
  .groupBy('user.city')
  .orderBy('userCount', 'DESC')
  .getRawMany();
// Result: [
//   { city: 'New York', userCount: '50', averageAge: '35', totalSalary: '2000000' },
//   { city: 'LA', userCount: '40', averageAge: '30', totalSalary: '1500000' },
//   ...
// ]

// Multiple grouping levels
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.country', 'country')
  .addSelect('user.city', 'city')
  .addSelect('COUNT(*)', 'count')
  .addSelect('AVG(user.salary)', 'avgSalary')
  .groupBy('user.country')
  .addGroupBy('user.city')
  .orderBy('country', 'ASC')
  .addOrderBy('count', 'DESC')
  .getRawMany();
```

### Conditional Aggregations (CASE)

```typescript
// Count with conditions
const stats = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city', 'city')
  .addSelect('COUNT(*)', 'total')
  .addSelect(
    'COUNT(CASE WHEN user.age < 18 THEN 1 END)',
    'minors'
  )
  .addSelect(
    'COUNT(CASE WHEN user.age BETWEEN 18 AND 65 THEN 1 END)',
    'adults'
  )
  .addSelect(
    'COUNT(CASE WHEN user.age > 65 THEN 1 END)',
    'seniors'
  )
  .addSelect(
    'COUNT(CASE WHEN user.isActive = true THEN 1 END)',
    'active'
  )
  .groupBy('user.city')
  .getRawMany();

// Conditional SUM
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city', 'city')
  .addSelect(
    'SUM(CASE WHEN user.isActive = true THEN user.salary ELSE 0 END)',
    'activeSalary'
  )
  .addSelect(
    'SUM(CASE WHEN user.isActive = false THEN user.salary ELSE 0 END)',
    'inactiveSalary'
  )
  .groupBy('user.city')
  .getRawMany();
```

### Aggregations with JOINs

```typescript
@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column()
  viewCount: number;

  @Column()
  likeCount: number;

  @ManyToOne(() => User, (user) => user.posts)
  author: User;
}

// Count posts per user
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoin('user.posts', 'post')
  .select('user.id', 'userId')
  .addSelect('user.firstName', 'name')
  .addSelect('COUNT(post.id)', 'postCount')
  .addSelect('SUM(post.viewCount)', 'totalViews')
  .addSelect('AVG(post.likeCount)', 'avgLikes')
  .groupBy('user.id')
  .orderBy('postCount', 'DESC')
  .getRawMany();

// Multiple JOIN aggregations
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoin('user.posts', 'post')
  .leftJoin('post.comments', 'comment')
  .select('user.id', 'userId')
  .addSelect('user.firstName', 'name')
  .addSelect('COUNT(DISTINCT post.id)', 'totalPosts')
  .addSelect('COUNT(DISTINCT comment.id)', 'totalComments')
  .addSelect('SUM(post.viewCount)', 'totalViews')
  .addSelect('MAX(post.createdAt)', 'lastPostDate')
  .groupBy('user.id')
  .having('COUNT(DISTINCT post.id) > :min', { min: 0 })
  .orderBy('totalPosts', 'DESC')
  .getRawMany();
```

### Real-World Examples

```typescript
// Dashboard overview
export class DashboardService {
  async getOverviewStats() {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .select('COUNT(*)', 'totalUsers')
      .addSelect(
        'COUNT(CASE WHEN user.isActive = true THEN 1 END)',
        'activeUsers'
      )
      .addSelect(
        'COUNT(CASE WHEN user.isPremium = true THEN 1 END)',
        'premiumUsers'
      )
      .addSelect(
        "COUNT(CASE WHEN user.createdAt > NOW() - INTERVAL '7 days' THEN 1 END)",
        'newUsersThisWeek'
      )
      .addSelect(
        "COUNT(CASE WHEN user.createdAt > NOW() - INTERVAL '30 days' THEN 1 END)",
        'newUsersThisMonth'
      )
      .addSelect('AVG(user.age)', 'averageAge')
      .addSelect('SUM(user.salary)', 'totalSalary')
      .getRawOne();
  }

  // User engagement metrics
  async getUserEngagementMetrics() {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .leftJoin('user.posts', 'post')
      .leftJoin('post.comments', 'comment')
      .leftJoin('user.likes', 'like')
      .select('user.id', 'userId')
      .addSelect('user.firstName', 'name')
      .addSelect('user.email', 'email')
      .addSelect('COUNT(DISTINCT post.id)', 'postsCreated')
      .addSelect('COUNT(DISTINCT comment.id)', 'commentsReceived')
      .addSelect('COUNT(DISTINCT like.id)', 'likesReceived')
      .addSelect('SUM(post.viewCount)', 'totalViews')
      .addSelect(
        'AVG(EXTRACT(EPOCH FROM (NOW() - user.lastLoginAt)))',
        'avgDaysSinceLogin'
      )
      .groupBy('user.id')
      .orderBy('totalViews', 'DESC')
      .take(100)
      .getRawMany();
  }

  // Revenue analytics
  async getRevenueByCity() {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .leftJoin('user.subscriptions', 'sub')
      .select('user.city', 'city')
      .addSelect('COUNT(DISTINCT user.id)', 'totalUsers')
      .addSelect('COUNT(DISTINCT sub.id)', 'totalSubscriptions')
      .addSelect('SUM(sub.amount)', 'totalRevenue')
      .addSelect('AVG(sub.amount)', 'avgSubscriptionValue')
      .addSelect('MAX(sub.amount)', 'maxSubscriptionValue')
      .where('sub.status = :status', { status: 'active' })
      .groupBy('user.city')
      .having('SUM(sub.amount) > :minRevenue', { minRevenue: 1000 })
      .orderBy('totalRevenue', 'DESC')
      .getRawMany();
  }

  // Time-series aggregations
  async getDailySignups(days: number = 30) {
    const startDate = new Date();
    startDate.setDate(startDate.getDate() - days);

    return this.dataSource
      .createQueryBuilder(User, 'user')
      .select("DATE(user.createdAt)", 'date')
      .addSelect('COUNT(*)', 'signups')
      .addSelect(
        'COUNT(CASE WHEN user.isPremium = true THEN 1 END)',
        'premiumSignups'
      )
      .addSelect('AVG(user.age)', 'avgAge')
      .where('user.createdAt >= :startDate', { startDate })
      .groupBy('date')
      .orderBy('date', 'ASC')
      .getRawMany();
  }
}
```

### Advanced Aggregations

```typescript
// Percentile calculations (PostgreSQL)
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city', 'city')
  .addSelect('PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY user.age)', 'medianAge')
  .addSelect('PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY user.salary)', 'salary25th')
  .addSelect('PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY user.salary)', 'salary75th')
  .groupBy('user.city')
  .getRawMany();

// Standard deviation and variance
const stats = await dataSource
  .createQueryBuilder(User, 'user')
  .select('STDDEV(user.age)', 'ageStdDev')
  .addSelect('VARIANCE(user.age)', 'ageVariance')
  .addSelect('STDDEV(user.salary)', 'salaryStdDev')
  .getRawOne();

// String aggregation (PostgreSQL)
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city', 'city')
  .addSelect("STRING_AGG(user.firstName, ', ')", 'userNames')
  .addSelect('COUNT(*)', 'userCount')
  .groupBy('user.city')
  .getRawMany();
// Result: { city: 'NYC', userNames: 'John, Jane, Bob', userCount: '3' }
```

### Aggregations with Subqueries

```typescript
// Compare user stats to average
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoin('user.posts', 'post')
  .select('user.id', 'userId')
  .addSelect('user.firstName', 'name')
  .addSelect('COUNT(post.id)', 'postCount')
  .addSelect(
    (subQuery) => {
      return subQuery
        .select('AVG(postCount)')
        .from((qb) => {
          return qb
            .select('COUNT(p.id)', 'postCount')
            .from(Post, 'p')
            .groupBy('p.authorId');
        }, 'counts')
        .getQuery();
    },
    'avgPostCount'
  )
  .groupBy('user.id')
  .getRawMany();
```

### Best Practices

```typescript
// ✅ GOOD: Use getRawOne() for single aggregate result
const total = await dataSource
  .createQueryBuilder(User, 'user')
  .select('COUNT(*)', 'total')
  .getRawOne();

// ✅ GOOD: Use getRawMany() for grouped aggregations
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city', 'city')
  .addSelect('COUNT(*)', 'count')
  .groupBy('user.city')
  .getRawMany();

// ✅ GOOD: Use DISTINCT for accurate counts with JOINs
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoin('user.posts', 'post')
  .select('COUNT(DISTINCT user.id)', 'userCount')
  .addSelect('COUNT(post.id)', 'postCount')
  .getRawOne();

// ❌ BAD: Using COUNT(*) with JOINs (inflated count)
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoin('user.posts', 'post')
  .select('COUNT(*)', 'userCount') // Wrong if users have multiple posts!
  .getRawOne();

// ✅ GOOD: Use CASE for conditional aggregations
const stats = await dataSource
  .createQueryBuilder(User, 'user')
  .select('COUNT(CASE WHEN user.isActive THEN 1 END)', 'activeCount')
  .addSelect('COUNT(CASE WHEN NOT user.isActive THEN 1 END)', 'inactiveCount')
  .getRawOne();

// ✅ GOOD: Always provide column aliases
const result = await dataSource
  .createQueryBuilder(User, 'user')
  .select('COUNT(*)', 'totalUsers') // Alias for clarity
  .addSelect('AVG(user.age)', 'averageAge')
  .getRawOne();

// ✅ GOOD: Filter with WHERE before aggregating
const result = await dataSource
  .createQueryBuilder(User, 'user')
  .select('COUNT(*)', 'activeUsers')
  .where('user.isActive = :active', { active: true })
  .getRawOne();

// ✅ GOOD: Use HAVING for filtered aggregations
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city', 'city')
  .addSelect('COUNT(*)', 'count')
  .groupBy('user.city')
  .having('COUNT(*) >= :min', { min: 10 })
  .getRawMany();
```

**Interview Tip**: Explain aggregate functions: **COUNT()** (count rows), **SUM()** (total), **AVG()** (average), **MIN()**/**MAX()** (extremes). Use in **select()** or **addSelect()**, get results with **getRawOne()** (single result) or **getRawMany()** (grouped results). Emphasize **COUNT(DISTINCT col)** for accurate counts with JOINs, **CASE expressions** for conditional aggregations (COUNT with filters). Mention **GROUP BY** for grouped aggregations (per city, per month). For production: use getRawOne/getRawMany (not getMany), use DISTINCT with COUNT when JOINs involved, filter with WHERE before aggregating (performance), use HAVING for aggregate filters, always provide column aliases for clarity, use CASE for conditional counts, consider database-specific functions (PERCENTILE_CONT, STRING_AGG), cache expensive aggregate queries, use indexes on aggregated columns.

</details>

<details>
<summary>75. How do you use subqueries in QueryBuilder?</summary>

Use **subQuery()** method to create subqueries within WHERE, SELECT, FROM, or JOIN clauses. TypeORM supports correlated and non-correlated subqueries. Subqueries can be created inline with callback functions or as separate query builders.

### Basic Subquery in WHERE

```typescript
import { DataSource } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  age: number;

  @OneToMany(() => Post, (post) => post.author)
  posts: Post[];
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToOne(() => User, (user) => user.posts)
  author: User;
}

const dataSource = new DataSource(/* config */);

// Users who have posts
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where((qb) => {
    const subQuery = qb
      .subQuery()
      .select('post.authorId')
      .from(Post, 'post')
      .getQuery();
    return `user.id IN ${subQuery}`;
  })
  .getMany();
// SQL: SELECT * FROM users user
//      WHERE user.id IN (SELECT post.authorId FROM posts post)

// Users older than average age
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where((qb) => {
    const subQuery = qb
      .subQuery()
      .select('AVG(u.age)')
      .from(User, 'u')
      .getQuery();
    return `user.age > ${subQuery}`;
  })
  .getMany();
// SQL: WHERE user.age > (SELECT AVG(u.age) FROM users u)
```

### Subquery with Parameters

```typescript
// Users with more than X posts
const minPosts = 5;
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where((qb) => {
    const subQuery = qb
      .subQuery()
      .select('COUNT(*)')
      .from(Post, 'post')
      .where('post.authorId = user.id')
      .getQuery();
    return `${subQuery} > :minPosts`;
  })
  .setParameter('minPosts', minPosts)
  .getMany();
// SQL: WHERE (SELECT COUNT(*) FROM posts WHERE authorId = user.id) > 5

// Users in specific cities with posts
const cities = ['New York', 'Los Angeles'];
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.city IN (:...cities)', { cities })
  .andWhere((qb) => {
    const subQuery = qb
      .subQuery()
      .select('post.authorId')
      .from(Post, 'post')
      .where('post.isPublished = :pub', { pub: true })
      .getQuery();
    return `user.id IN ${subQuery}`;
  })
  .getMany();
```

### Subquery in SELECT

```typescript
// Select user with post count from subquery
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .select(['user.id', 'user.firstName'])
  .addSelect((subQuery) => {
    return subQuery
      .select('COUNT(post.id)')
      .from(Post, 'post')
      .where('post.authorId = user.id');
  }, 'postCount')
  .getRawMany();
// SQL: SELECT user.id, user.firstName,
//      (SELECT COUNT(post.id) FROM posts post WHERE post.authorId = user.id) as postCount
// Result: [{ user_id: 1, user_firstName: 'John', postCount: '5' }, ...]

// Multiple subqueries in SELECT
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .select(['user.id', 'user.firstName'])
  .addSelect((subQuery) => {
    return subQuery
      .select('COUNT(*)')
      .from(Post, 'post')
      .where('post.authorId = user.id');
  }, 'totalPosts')
  .addSelect((subQuery) => {
    return subQuery
      .select('COUNT(*)')
      .from(Post, 'post')
      .where('post.authorId = user.id')
      .andWhere('post.isPublished = :pub', { pub: true });
  }, 'publishedPosts')
  .getRawMany();
```

### Subquery in FROM

```typescript
// Query from subquery result
const results = await dataSource
  .createQueryBuilder()
  .select('subquery.city', 'city')
  .addSelect('subquery.userCount', 'count')
  .from((subQuery) => {
    return subQuery
      .select('user.city', 'city')
      .addSelect('COUNT(*)', 'userCount')
      .from(User, 'user')
      .groupBy('user.city');
  }, 'subquery')
  .where('subquery.userCount > :min', { min: 10 })
  .orderBy('subquery.userCount', 'DESC')
  .getRawMany();
// SQL: SELECT subquery.city, subquery.userCount
//      FROM (SELECT user.city, COUNT(*) as userCount
//            FROM users user
//            GROUP BY user.city) subquery
//      WHERE subquery.userCount > 10
//      ORDER BY subquery.userCount DESC
```

### Correlated Subqueries

```typescript
// Correlated: Subquery references outer query
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where((qb) => {
    const subQuery = qb
      .subQuery()
      .select('COUNT(*)')
      .from(Post, 'post')
      .where('post.authorId = user.id') // References outer 'user'
      .andWhere('post.createdAt > :date', { date: lastMonth })
      .getQuery();
    return `${subQuery} > :minPosts`;
  })
  .setParameter('minPosts', 5)
  .getMany();
// Users with more than 5 posts in the last month

// Multiple correlated subqueries
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.id', 'userId')
  .addSelect('user.firstName', 'name')
  .addSelect((subQuery) => {
    return subQuery
      .select('COUNT(*)')
      .from(Post, 'post')
      .where('post.authorId = user.id');
  }, 'postCount')
  .addSelect((subQuery) => {
    return subQuery
      .select('MAX(post.createdAt)')
      .from(Post, 'post')
      .where('post.authorId = user.id');
  }, 'lastPostDate')
  .getRawMany();
```

### EXISTS Subqueries

```typescript
// Users who have at least one post
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where((qb) => {
    const subQuery = qb
      .subQuery()
      .select('1')
      .from(Post, 'post')
      .where('post.authorId = user.id')
      .getQuery();
    return `EXISTS ${subQuery}`;
  })
  .getMany();
// SQL: WHERE EXISTS (SELECT 1 FROM posts post WHERE post.authorId = user.id)

// NOT EXISTS - Users without posts
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where((qb) => {
    const subQuery = qb
      .subQuery()
      .select('1')
      .from(Post, 'post')
      .where('post.authorId = user.id')
      .getQuery();
    return `NOT EXISTS ${subQuery}`;
  })
  .getMany();
```

### Real-World Examples

```typescript
// Find users more active than average
export class UserService {
  async getAboveAverageUsers() {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .leftJoin('user.posts', 'post')
      .select('user.id', 'userId')
      .addSelect('user.firstName', 'name')
      .addSelect('COUNT(post.id)', 'postCount')
      .groupBy('user.id')
      .having((qb) => {
        const subQuery = qb
          .subQuery()
          .select('AVG(postCount)')
          .from((subQb) => {
            return subQb
              .select('COUNT(p.id)', 'postCount')
              .from(Post, 'p')
              .groupBy('p.authorId');
          }, 'counts')
          .getQuery();
        return `COUNT(post.id) > ${subQuery}`;
      })
      .getRawMany();
  }

  // Users with recent activity
  async getUsersWithRecentActivity(days: number = 7) {
    const cutoffDate = new Date();
    cutoffDate.setDate(cutoffDate.getDate() - days);

    return this.dataSource
      .createQueryBuilder(User, 'user')
      .where((qb) => {
        const subQuery = qb
          .subQuery()
          .select('post.authorId')
          .from(Post, 'post')
          .where('post.createdAt > :cutoff', { cutoff: cutoffDate })
          .getQuery();
        return `user.id IN ${subQuery}`;
      })
      .getMany();
  }

  // Top contributors by percentile
  async getTopContributors(percentile: number = 90) {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .leftJoin('user.posts', 'post')
      .select('user.id', 'userId')
      .addSelect('user.firstName', 'name')
      .addSelect('COUNT(post.id)', 'postCount')
      .groupBy('user.id')
      .having((qb) => {
        const subQuery = qb
          .subQuery()
          .select(`PERCENTILE_CONT(${percentile / 100}) WITHIN GROUP (ORDER BY COUNT(p.id))`)
          .from(Post, 'p')
          .groupBy('p.authorId')
          .getQuery();
        return `COUNT(post.id) >= ${subQuery}`;
      })
      .orderBy('postCount', 'DESC')
      .getRawMany();
  }
}
```

### Complex Nested Subqueries

```typescript
// Users in cities with above-average user counts
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where((qb) => {
    const subQuery = qb
      .subQuery()
      .select('subCity.city')
      .from((innerQb) => {
        return innerQb
          .select('u.city', 'city')
          .addSelect('COUNT(*)', 'userCount')
          .from(User, 'u')
          .groupBy('u.city');
      }, 'subCity')
      .where((innerQb2) => {
        const avgQuery = innerQb2
          .subQuery()
          .select('AVG(cityCount.userCount)')
          .from((avgQb) => {
            return avgQb
              .select('COUNT(*)', 'userCount')
              .from(User, 'u2')
              .groupBy('u2.city');
          }, 'cityCount')
          .getQuery();
        return `subCity.userCount > ${avgQuery}`;
      })
      .getQuery();
    return `user.city IN ${subQuery}`;
  })
  .getMany();
```

### Subquery Performance Optimization

```typescript
// ❌ SLOW: Correlated subquery in SELECT (N+1 issue)
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user')
  .addSelect((subQuery) => {
    return subQuery
      .select('COUNT(*)')
      .from(Post, 'post')
      .where('post.authorId = user.id'); // Executed for each user!
  }, 'postCount')
  .getRawMany();

// ✅ BETTER: Use JOIN instead
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoin('user.posts', 'post')
  .select('user.id', 'userId')
  .addSelect('COUNT(post.id)', 'postCount')
  .groupBy('user.id')
  .getRawMany();

// ✅ GOOD: Non-correlated subquery for filtering
const avgAge = await dataSource
  .createQueryBuilder(User, 'u')
  .select('AVG(u.age)')
  .getRawOne();

const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.age > :avg', { avg: avgAge })
  .getMany();
```

### Best Practices

```typescript
// ✅ GOOD: Use EXISTS for existence checks
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where((qb) => {
    const subQuery = qb
      .subQuery()
      .select('1')
      .from(Post, 'post')
      .where('post.authorId = user.id')
      .getQuery();
    return `EXISTS ${subQuery}`;
  })
  .getMany();

// ✅ GOOD: Use IN for list matching
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where((qb) => {
    const subQuery = qb
      .subQuery()
      .select('post.authorId')
      .from(Post, 'post')
      .where('post.isPublished = :pub', { pub: true })
      .getQuery();
    return `user.id IN ${subQuery}`;
  })
  .getMany();

// ✅ GOOD: Use JOINs when possible (often faster)
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .innerJoin('user.posts', 'post')
  .distinct(true)
  .getMany();
// Usually faster than EXISTS subquery

// ✅ GOOD: Cache static subquery results
const avgAge = await this.cacheManager.get('avgAge');
if (!avgAge) {
  const result = await dataSource
    .createQueryBuilder(User, 'user')
    .select('AVG(user.age)')
    .getRawOne();
  await this.cacheManager.set('avgAge', result, 3600);
}

// ✅ GOOD: Use proper indentation for readability
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where((qb) => {
    const subQuery = qb
      .subQuery()
      .select('post.authorId')
      .from(Post, 'post')
      .where('post.isPublished = :pub', { pub: true })
      .getQuery();
    return `user.id IN ${subQuery}`;
  })
  .getMany();

// ❌ AVOID: Deeply nested subqueries (hard to maintain)
// Consider breaking into multiple queries or using JOINs

// ❌ AVOID: Correlated subqueries in SELECT (performance)
// Use JOINs + GROUP BY instead
```

**Interview Tip**: Explain **subqueries** are queries nested within another query using **subQuery()** method. Use in WHERE (filtering), SELECT (computed columns), FROM (derived tables). Mention types: **correlated** (references outer query, slower), **non-correlated** (independent, faster). Emphasize **EXISTS** for existence checks (efficient), **IN** for list matching, prefer **JOINs over subqueries** when possible (performance). For production: use EXISTS instead of IN when checking existence, avoid correlated subqueries in SELECT (N+1 issue), prefer JOINs for better performance, cache static subquery results, use non-correlated subqueries when possible, consider query execution plans, break deeply nested subqueries into CTEs or multiple queries, always test subquery performance with EXPLAIN.

</details>

<details>
<summary>76. What is the difference between getMany() and getRawMany()?</summary>

**getMany()** returns entity instances with relations and transformations applied, while **getRawMany()** returns raw database rows as plain objects with column names as keys. Use **getMany()** for working with entities and relations, **getRawMany()** for aggregations, custom SELECTs, or performance-critical queries.

### Basic Difference

```typescript
import { DataSource } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column()
  email: string;

  @OneToMany(() => Post, (post) => post.author)
  posts: Post[];

  // Virtual/computed property
  get fullName(): string {
    return `${this.firstName} ${this.lastName}`;
  }
}

const dataSource = new DataSource(/* config */);

// getMany() - Returns User entity instances
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.isActive = :active', { active: true })
  .getMany();
// Result: [User { id: 1, firstName: 'John', lastName: 'Doe', ... }, ...]
// Type: User[]
// Instances of User class with methods, getters, etc.
console.log(users[0].fullName); // 'John Doe' - getter works!
console.log(users[0] instanceof User); // true

// getRawMany() - Returns plain objects
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.isActive = :active', { active: true })
  .getRawMany();
// Result: [{ user_id: 1, user_firstName: 'John', user_lastName: 'Doe', ... }, ...]
// Type: any[]
// Plain objects with column names prefixed by alias
console.log(users[0].user_firstName); // 'John' - note the prefix!
console.log(users[0].fullName); // undefined - no getter
console.log(users[0] instanceof User); // false
```

### Column Naming Differences

```typescript
// getMany() - Property names match entity
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .getMany();
// Result: [{ id: 1, firstName: 'John', lastName: 'Doe', email: '...' }, ...]
// Access: users[0].firstName

// getRawMany() - Column names prefixed by alias
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .getRawMany();
// Result: [{ user_id: 1, user_firstName: 'John', user_lastName: 'Doe', ... }, ...]
// Access: users[0].user_firstName (alias_columnName format)

// getRawMany() with custom aliases
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.id', 'userId')
  .addSelect('user.firstName', 'name')
  .addSelect('user.email', 'emailAddress')
  .getRawMany();
// Result: [{ userId: 1, name: 'John', emailAddress: 'john@...' }, ...]
// Access: users[0].name (uses your custom aliases)
```

### With Relations

```typescript
@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToOne(() => User, (user) => user.posts)
  author: User;
}

// getMany() with relations - Nested objects
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoinAndSelect('user.posts', 'post')
  .getMany();
// Result: [
//   User {
//     id: 1,
//     firstName: 'John',
//     posts: [
//       Post { id: 1, title: 'Post 1', ... },
//       Post { id: 2, title: 'Post 2', ... }
//     ]
//   },
//   ...
// ]
// Access: users[0].posts[0].title

// getRawMany() with relations - Flat structure
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoin('user.posts', 'post')
  .select('user.id', 'userId')
  .addSelect('user.firstName', 'userName')
  .addSelect('post.id', 'postId')
  .addSelect('post.title', 'postTitle')
  .getRawMany();
// Result: [
//   { userId: 1, userName: 'John', postId: 1, postTitle: 'Post 1' },
//   { userId: 1, userName: 'John', postId: 2, postTitle: 'Post 2' },
//   { userId: 2, userName: 'Jane', postId: 3, postTitle: 'Post 3' }
// ]
// Note: Flat structure, one row per post (duplicates user data)
```

### With Aggregations

```typescript
// getMany() - Cannot use aggregations directly
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoin('user.posts', 'post')
  .select(['user.id', 'user.firstName'])
  .addSelect('COUNT(post.id)', 'postCount') // Won't work with getMany()!
  .groupBy('user.id')
  .getMany();
// Result: User entities WITHOUT the postCount field
// getMany() ignores aggregate columns

// getRawMany() - Works with aggregations
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoin('user.posts', 'post')
  .select('user.id', 'userId')
  .addSelect('user.firstName', 'name')
  .addSelect('COUNT(post.id)', 'postCount')
  .groupBy('user.id')
  .getRawMany();
// Result: [
//   { userId: 1, name: 'John', postCount: '5' },
//   { userId: 2, name: 'Jane', postCount: '3' }
// ]
// getRawMany() includes all selected columns including aggregates
```

### Performance Comparison

```typescript
// getMany() - More overhead (entity instantiation, transformations)
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.city = :city', { city: 'New York' })
  .getMany();
// - Instantiates User class for each row
// - Applies transformers (@Transform decorators)
// - Executes getters/setters
// - Slower but provides full entity features

// getRawMany() - Less overhead (plain objects)
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.city = :city', { city: 'New York' })
  .getRawMany();
// - Returns plain JavaScript objects
// - No class instantiation
// - No transformations
// - Faster for large datasets or reporting
```

### When to Use Each

```typescript
// ✅ Use getMany() when:

// 1. Working with entity methods
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .getMany();
users.forEach(user => user.sendWelcomeEmail()); // Method available

// 2. Need computed properties/getters
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .getMany();
console.log(users[0].fullName); // Works with getters

// 3. Loading relations as nested objects
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoinAndSelect('user.posts', 'post')
  .leftJoinAndSelect('user.profile', 'profile')
  .getMany();
// Clean nested structure

// 4. Need TypeScript type safety
const users: User[] = await dataSource
  .createQueryBuilder(User, 'user')
  .getMany();
// users[0] is strongly typed as User

// ✅ Use getRawMany() when:

// 1. Performing aggregations
const stats = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city', 'city')
  .addSelect('COUNT(*)', 'userCount')
  .groupBy('user.city')
  .getRawMany();

// 2. Selecting custom columns or calculations
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.firstName || \' \' || user.lastName', 'fullName')
  .addSelect('EXTRACT(YEAR FROM user.createdAt)', 'joinYear')
  .getRawMany();

// 3. Need flat structure with JOINs
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoin('user.posts', 'post')
  .select('user.firstName', 'author')
  .addSelect('post.title', 'postTitle')
  .getRawMany();

// 4. Performance-critical queries (large datasets)
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select(['user.id', 'user.email'])
  .where('user.isActive = :active', { active: true })
  .getRawMany();
// Faster for millions of rows

// 5. Exporting/reporting data
const exportData = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.firstName', 'firstName')
  .addSelect('user.email', 'email')
  .addSelect('user.createdAt', 'signupDate')
  .getRawMany();
// Ready for CSV export
```

### Real-World Examples

```typescript
export class UserService {
  // Use getMany() for business logic
  async getActiveUsers(): Promise<User[]> {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .leftJoinAndSelect('user.profile', 'profile')
      .leftJoinAndSelect('user.posts', 'posts')
      .where('user.isActive = :active', { active: true })
      .getMany();
    // Returns User entities with relations, can call methods
  }

  // Use getRawMany() for statistics
  async getUserStatsByCity() {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .select('user.city', 'city')
      .addSelect('COUNT(*)', 'totalUsers')
      .addSelect('COUNT(CASE WHEN user.isActive THEN 1 END)', 'activeUsers')
      .addSelect('AVG(user.age)', 'averageAge')
      .groupBy('user.city')
      .orderBy('totalUsers', 'DESC')
      .getRawMany();
    // Returns plain objects with aggregated data
  }

  // Use getRawMany() for reports
  async generateUserReport() {
    const results = await this.dataSource
      .createQueryBuilder(User, 'user')
      .leftJoin('user.posts', 'post')
      .select('user.id', 'userId')
      .addSelect('user.firstName', 'firstName')
      .addSelect('user.email', 'email')
      .addSelect('COUNT(post.id)', 'totalPosts')
      .addSelect('MAX(post.createdAt)', 'lastPostDate')
      .groupBy('user.id')
      .getRawMany();

    // Convert to CSV or Excel
    return this.convertToCSV(results);
  }

  // Use getMany() for entity operations
  async deactivateInactiveUsers() {
    const inactiveUsers = await this.dataSource
      .createQueryBuilder(User, 'user')
      .where('user.lastLoginAt < :date', { 
        date: new Date(Date.now() - 90 * 24 * 60 * 60 * 1000) 
      })
      .getMany();

    // Can use entity methods
    for (const user of inactiveUsers) {
      user.isActive = false;
      await user.sendDeactivationEmail(); // Entity method
    }

    return this.userRepository.save(inactiveUsers);
  }
}
```

### getRawOne() vs getOne()

```typescript
// getOne() - Returns single entity or null
const user = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.id = :id', { id: 1 })
  .getOne();
// Result: User { id: 1, firstName: 'John', ... } or null
// Type: User | null

// getRawOne() - Returns single plain object or undefined
const user = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.id = :id', { id: 1 })
  .getRawOne();
// Result: { user_id: 1, user_firstName: 'John', ... } or undefined
// Type: any | undefined

// getRawOne() with aggregation
const stats = await dataSource
  .createQueryBuilder(User, 'user')
  .select('COUNT(*)', 'total')
  .addSelect('AVG(user.age)', 'avgAge')
  .getRawOne();
// Result: { total: '150', avgAge: '32.5' }
```

### Combining getMany() and getRawMany()

```typescript
// Sometimes you need both!

// Step 1: Get IDs with aggregations (getRawMany)
const topUserIds = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoin('user.posts', 'post')
  .select('user.id', 'userId')
  .addSelect('COUNT(post.id)', 'postCount')
  .groupBy('user.id')
  .having('COUNT(post.id) >= :min', { min: 10 })
  .orderBy('postCount', 'DESC')
  .take(10)
  .getRawMany();

const ids = topUserIds.map(u => u.userId);

// Step 2: Load full entities with relations (getMany)
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoinAndSelect('user.posts', 'posts')
  .leftJoinAndSelect('user.profile', 'profile')
  .whereInIds(ids)
  .getMany();
// Now you have full User entities with relations
```

### Best Practices

```typescript
// ✅ GOOD: Use getMany() for entities with relations
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .leftJoinAndSelect('user.posts', 'posts')
  .getMany();

// ✅ GOOD: Use getRawMany() for aggregations
const stats = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city', 'city')
  .addSelect('COUNT(*)', 'count')
  .groupBy('user.city')
  .getRawMany();

// ✅ GOOD: Provide aliases with getRawMany()
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.id', 'userId')
  .addSelect('user.firstName', 'name')
  .getRawMany();
// Clean property names without alias prefix

// ❌ BAD: Using getMany() with aggregations (won't work)
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .select(['user.id', 'user.firstName'])
  .addSelect('COUNT(post.id)', 'postCount')
  .leftJoin('user.posts', 'post')
  .groupBy('user.id')
  .getMany();
// postCount will be missing!

// ❌ BAD: Using getRawMany() without aliases
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .getRawMany();
// Properties will be prefixed: user_id, user_firstName (annoying)

// ✅ GOOD: Type the result of getRawMany()
interface UserStats {
  city: string;
  userCount: number;
  avgAge: number;
}

const stats: UserStats[] = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city', 'city')
  .addSelect('COUNT(*)', 'userCount')
  .addSelect('AVG(user.age)', 'avgAge')
  .groupBy('user.city')
  .getRawMany();

// ✅ GOOD: Use getRawMany() for performance-critical queries
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.id', 'id')
  .addSelect('user.email', 'email')
  .where('user.createdAt > :date', { date: lastMonth })
  .getRawMany();
// Faster than getMany() for large results
```

**Interview Tip**: Explain **getMany()** returns entity instances (class objects with methods, getters, transformations), **getRawMany()** returns plain objects (raw database rows). Key differences: **getMany()** has property names matching entity, supports relations as nested objects, cannot include aggregations; **getRawMany()** has column names (alias_column format or custom aliases), returns flat structure, includes aggregations. Use **getMany()** for business logic with entities and relations, **getRawMany()** for aggregations, custom queries, reporting, or performance-critical queries. Mention: **getRawOne()** vs **getOne()** for single results, always provide custom aliases with getRawMany() for clean property names, type the results of getRawMany() for type safety, getMany() has overhead (instantiation, transformers) while getRawMany() is faster for large datasets.

</details>

<details>
<summary>77. How do you perform UPDATE operations with QueryBuilder?</summary>

Use **.update(Entity)** to start an UPDATE query builder, **.set()** to specify column values, **.where()** to filter rows, and **.execute()** to run the query. UPDATE operations bypass entity lifecycle hooks and validations but are much faster than loading entities and saving them.

### Basic UPDATE

```typescript
import { DataSource } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column()
  email: string;

  @Column({ default: true })
  isActive: boolean;

  @UpdateDateColumn()
  updatedAt: Date;
}

const dataSource = new DataSource(/* config */);

// Update single column
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ isActive: false })
  .where('id = :id', { id: 1 })
  .execute();
// SQL: UPDATE users SET isActive = false WHERE id = 1

// Update multiple columns
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ 
    firstName: 'Jane',
    lastName: 'Smith',
    email: 'jane.smith@example.com'
  })
  .where('id = :id', { id: 1 })
  .execute();

// Update with parameters
const userId = 1;
const newEmail = 'newemail@example.com';

await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ email: newEmail })
  .where('id = :id', { id: userId })
  .execute();
```

### UPDATE with Expressions

```typescript
// Increment a value
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ 
    loginCount: () => 'loginCount + 1' 
  })
  .where('id = :id', { id: 1 })
  .execute();
// SQL: UPDATE users SET loginCount = loginCount + 1 WHERE id = 1

// Update with calculation
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ 
    salary: () => 'salary * 1.1' // 10% raise
  })
  .where('department = :dept', { dept: 'Engineering' })
  .execute();

// Update with CASE expression
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({
    tier: () => `CASE 
      WHEN points > 1000 THEN 'gold'
      WHEN points > 500 THEN 'silver'
      ELSE 'bronze'
    END`
  })
  .execute();

// Concatenate strings
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ 
    fullName: () => "firstName || ' ' || lastName"
  })
  .execute();
```

### UPDATE with Multiple Conditions

```typescript
// AND conditions
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ isActive: false })
  .where('lastLoginAt < :date', { date: sixMonthsAgo })
  .andWhere('isActive = :active', { active: true })
  .execute();

// OR conditions
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ isPremium: false })
  .where('subscriptionEndDate < :now', { now: new Date() })
  .orWhere('paymentFailed = :failed', { failed: true })
  .execute();

// IN clause
const userIds = [1, 2, 3, 4, 5];
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ notificationEnabled: true })
  .where('id IN (:...ids)', { ids: userIds })
  .execute();

// Complex conditions with Brackets
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ status: 'inactive' })
  .where('isActive = :active', { active: true })
  .andWhere(new Brackets(qb => {
    qb.where('lastLoginAt < :date', { date: ninetyDaysAgo })
      .orWhere('emailVerified = :verified', { verified: false });
  }))
  .execute();
```

### Conditional UPDATE (UPDATE only if condition matches)

```typescript
// Update with subquery
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ hasOrders: true })
  .where((qb) => {
    const subQuery = qb
      .subQuery()
      .select('order.userId')
      .from('orders', 'order')
      .where('order.status = :status', { status: 'completed' })
      .getQuery();
    return `id IN ${subQuery}`;
  })
  .execute();

// Update based on joined data
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ isTopContributor: true })
  .where((qb) => {
    const subQuery = qb
      .subQuery()
      .select('post.authorId')
      .from('posts', 'post')
      .groupBy('post.authorId')
      .having('COUNT(*) >= :minPosts', { minPosts: 50 })
      .getQuery();
    return `id IN ${subQuery}`;
  })
  .execute();
```

### Bulk UPDATE

```typescript
// Update all rows
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ tier: 'bronze' })
  .execute();
// Updates ALL users (no WHERE clause)

// Update multiple rows with same values
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ lastNotificationSent: new Date() })
  .where('notificationEnabled = :enabled', { enabled: true })
  .execute();

// Bulk update specific IDs
const idsToUpdate = [1, 5, 10, 15, 20];
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ isVerified: true })
  .where('id IN (:...ids)', { ids: idsToUpdate })
  .execute();
```

### UPDATE with Return Values

```typescript
// Get affected rows count
const result = await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ isActive: false })
  .where('lastLoginAt < :date', { date: sixMonthsAgo })
  .execute();

console.log(result.affected); // Number of rows updated
// result.affected: 45 (updated 45 users)

// Check if any rows were updated
const result = await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ email: newEmail })
  .where('id = :id', { id: userId })
  .execute();

if (result.affected === 0) {
  throw new Error('User not found');
}
```

### Real-World Examples

```typescript
export class UserService {
  // Deactivate inactive users
  async deactivateInactiveUsers(days: number = 90) {
    const cutoffDate = new Date();
    cutoffDate.setDate(cutoffDate.getDate() - days);

    const result = await this.dataSource
      .createQueryBuilder()
      .update(User)
      .set({ 
        isActive: false,
        deactivationReason: 'Inactivity',
        deactivatedAt: new Date()
      })
      .where('lastLoginAt < :cutoff', { cutoff: cutoffDate })
      .andWhere('isActive = :active', { active: true })
      .execute();

    return { deactivatedCount: result.affected };
  }

  // Bulk status update
  async updateUserStatus(userIds: number[], status: string) {
    if (userIds.length === 0) return;

    const result = await this.dataSource
      .createQueryBuilder()
      .update(User)
      .set({ 
        status,
        statusUpdatedAt: new Date()
      })
      .where('id IN (:...ids)', { ids: userIds })
      .execute();

    return { updated: result.affected };
  }

  // Increment login count
  async recordLogin(userId: number) {
    await this.dataSource
      .createQueryBuilder()
      .update(User)
      .set({ 
        loginCount: () => 'loginCount + 1',
        lastLoginAt: new Date()
      })
      .where('id = :id', { id: userId })
      .execute();
  }

  // Apply discount based on tier
  async applyTierDiscounts() {
    await this.dataSource
      .createQueryBuilder()
      .update(User)
      .set({
        discount: () => `CASE 
          WHEN tier = 'gold' THEN 20
          WHEN tier = 'silver' THEN 10
          WHEN tier = 'bronze' THEN 5
          ELSE 0
        END`
      })
      .execute();
  }

  // Update email verification
  async verifyEmail(token: string) {
    const result = await this.dataSource
      .createQueryBuilder()
      .update(User)
      .set({ 
        emailVerified: true,
        emailVerifiedAt: new Date(),
        verificationToken: null
      })
      .where('verificationToken = :token', { token })
      .andWhere('emailVerified = :verified', { verified: false })
      .execute();

    if (result.affected === 0) {
      throw new Error('Invalid or expired token');
    }

    return { success: true };
  }

  // Batch update with limits
  async batchUpdateInChunks(updateData: { id: number; value: any }[]) {
    const CHUNK_SIZE = 1000;
    let totalUpdated = 0;

    for (let i = 0; i < updateData.length; i += CHUNK_SIZE) {
      const chunk = updateData.slice(i, i + CHUNK_SIZE);
      const ids = chunk.map(item => item.id);

      const result = await this.dataSource
        .createQueryBuilder()
        .update(User)
        .set({ someField: chunk[0].value }) // Simplified
        .where('id IN (:...ids)', { ids })
        .execute();

      totalUpdated += result.affected || 0;
    }

    return { totalUpdated };
  }
}
```

### UPDATE vs Repository.save()

```typescript
// ❌ SLOW: Load entity and save (triggers lifecycle hooks)
const user = await userRepository.findOne({ where: { id: 1 } });
user.isActive = false;
await userRepository.save(user);
// - SELECT query to load entity
// - UPDATE query to save
// - Triggers @BeforeUpdate, @AfterUpdate hooks
// - Validates entity
// - Updates @UpdateDateColumn automatically

// ✅ FAST: Direct UPDATE with QueryBuilder
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ isActive: false })
  .where('id = :id', { id: 1 })
  .execute();
// - Single UPDATE query
// - No entity loading
// - No lifecycle hooks
// - No validation
// - @UpdateDateColumn NOT updated automatically
// - Much faster for bulk updates

// Trade-offs:
// QueryBuilder UPDATE:
// ✅ Faster (single query, no overhead)
// ✅ Bulk operations efficient
// ❌ Bypasses lifecycle hooks (@BeforeUpdate, @AfterUpdate)
// ❌ No entity validation
// ❌ @UpdateDateColumn not updated
// ❌ No TypeScript entity methods

// Repository.save():
// ✅ Triggers lifecycle hooks
// ✅ Validates entity
// ✅ Updates @UpdateDateColumn
// ✅ Can use entity methods
// ❌ Slower (2 queries: SELECT + UPDATE)
// ❌ Inefficient for bulk operations
```

### Best Practices

```typescript
// ✅ GOOD: Always use WHERE clause (prevent accidental mass updates)
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ isActive: false })
  .where('id = :id', { id: userId })
  .execute();

// ❌ DANGEROUS: No WHERE clause (updates ALL rows!)
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ isActive: false })
  .execute();
// Updates every single user!

// ✅ GOOD: Check affected count
const result = await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ email: newEmail })
  .where('id = :id', { id: userId })
  .execute();

if (result.affected === 0) {
  throw new NotFoundException('User not found');
}

// ✅ GOOD: Use parameters to prevent SQL injection
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ firstName: userInput })
  .where('id = :id', { id: userId })
  .execute();

// ❌ BAD: String interpolation (SQL injection risk)
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ firstName: `'${userInput}'` }) // NEVER DO THIS!
  .where(`id = ${userId}`) // NEVER DO THIS!
  .execute();

// ✅ GOOD: Use functions for expressions
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ 
    count: () => 'count + 1' // Function syntax
  })
  .where('id = :id', { id: 1 })
  .execute();

// ✅ GOOD: Transaction for multiple updates
await dataSource.transaction(async (manager) => {
  await manager
    .createQueryBuilder()
    .update(User)
    .set({ balance: () => 'balance - :amount' })
    .where('id = :senderId', { senderId, amount })
    .execute();

  await manager
    .createQueryBuilder()
    .update(User)
    .set({ balance: () => 'balance + :amount' })
    .where('id = :receiverId', { receiverId, amount })
    .execute();
});

// ✅ GOOD: Manually update @UpdateDateColumn if needed
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ 
    isActive: false,
    updatedAt: new Date() // Manually set
  })
  .where('id = :id', { id: userId })
  .execute();

// ✅ GOOD: Limit bulk updates with WHERE
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ isPremium: true })
  .where('id IN (:...ids)', { ids: userIds.slice(0, 1000) }) // Limit batch size
  .execute();
```

**Interview Tip**: Explain **UPDATE** with QueryBuilder uses **.update(Entity)** to start, **.set()** for values, **.where()** to filter, **.execute()** to run. Emphasize **bypasses lifecycle hooks** (@BeforeUpdate, @AfterUpdate), no validation, @UpdateDateColumn NOT updated automatically, but **much faster** than repository.save() (single query, no entity loading). Mention use cases: **bulk updates** (deactivate inactive users), **increment counters** (loginCount + 1), **conditional updates** (CASE expressions). Best practices: always use WHERE clause (prevent mass updates), check result.affected (verify rows updated), use parameters (SQL injection prevention), use transactions for multiple related updates, manually set @UpdateDateColumn if needed, prefer QueryBuilder for performance-critical bulk updates, use repository.save() when need hooks/validation.

</details>

<details>
<summary>78. How do you perform DELETE operations with QueryBuilder?</summary>

Use **.delete()** to start a DELETE query builder, **.from(Entity)** to specify the table, **.where()** to filter rows, and **.execute()** to run the query. Like UPDATE, DELETE operations bypass entity lifecycle hooks but are much faster than loading and removing entities individually.

### Basic DELETE

```typescript
import { DataSource } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  isActive: boolean;

  @Column()
  lastLoginAt: Date;
}

const dataSource = new DataSource(/* config */);

// Delete by ID
await dataSource
  .createQueryBuilder()
  .delete()
  .from(User)
  .where('id = :id', { id: 1 })
  .execute();
// SQL: DELETE FROM users WHERE id = 1

// Delete with condition
await dataSource
  .createQueryBuilder()
  .delete()
  .from(User)
  .where('isActive = :active', { active: false })
  .execute();

// Delete with multiple conditions
await dataSource
  .createQueryBuilder()
  .delete()
  .from(User)
  .where('isActive = :active', { active: false })
  .andWhere('lastLoginAt < :date', { date: oneYearAgo })
  .execute();
```

### DELETE with Multiple Conditions

```typescript
// AND conditions
await dataSource
  .createQueryBuilder()
  .delete()
  .from(User)
  .where('emailVerified = :verified', { verified: false })
  .andWhere('createdAt < :date', { date: thirtyDaysAgo })
  .execute();

// OR conditions
await dataSource
  .createQueryBuilder()
  .delete()
  .from(User)
  .where('isActive = :active', { active: false })
  .orWhere('isDeleted = :deleted', { deleted: true })
  .execute();

// IN clause
const userIds = [1, 2, 3, 4, 5];
await dataSource
  .createQueryBuilder()
  .delete()
  .from(User)
  .where('id IN (:...ids)', { ids: userIds })
  .execute();

// Complex conditions with Brackets
await dataSource
  .createQueryBuilder()
  .delete()
  .from(User)
  .where('isActive = :active', { active: false })
  .andWhere(new Brackets(qb => {
    qb.where('lastLoginAt < :date', { date: sixMonthsAgo })
      .orWhere('emailBounced = :bounced', { bounced: true });
  }))
  .execute();
// SQL: DELETE FROM users
//      WHERE isActive = false
//      AND (lastLoginAt < '2024-06-23' OR emailBounced = true)
```

### DELETE with Subqueries

```typescript
// Delete users without posts
await dataSource
  .createQueryBuilder()
  .delete()
  .from(User)
  .where((qb) => {
    const subQuery = qb
      .subQuery()
      .select('post.authorId')
      .from('posts', 'post')
      .getQuery();
    return `id NOT IN ${subQuery}`;
  })
  .execute();

// Delete inactive users
await dataSource
  .createQueryBuilder()
  .delete()
  .from(User)
  .where((qb) => {
    const subQuery = qb
      .subQuery()
      .select('login.userId')
      .from('user_logins', 'login')
      .where('login.createdAt > :date', { date: ninetyDaysAgo })
      .getQuery();
    return `id NOT IN ${subQuery}`;
  })
  .execute();
// Users who haven't logged in for 90 days

// Delete based on aggregate condition
await dataSource
  .createQueryBuilder()
  .delete()
  .from(User)
  .where((qb) => {
    const subQuery = qb
      .subQuery()
      .select('u.id')
      .from(User, 'u')
      .leftJoin('u.posts', 'p')
      .groupBy('u.id')
      .having('COUNT(p.id) = 0')
      .getQuery();
    return `id IN ${subQuery}`;
  })
  .execute();
```

### Bulk DELETE

```typescript
// Delete all inactive users
await dataSource
  .createQueryBuilder()
  .delete()
  .from(User)
  .where('isActive = :active', { active: false })
  .execute();

// Delete multiple IDs
const idsToDelete = [1, 5, 10, 15, 20];
await dataSource
  .createQueryBuilder()
  .delete()
  .from(User)
  .where('id IN (:...ids)', { ids: idsToDelete })
  .execute();

// Delete old records
await dataSource
  .createQueryBuilder()
  .delete()
  .from(User)
  .where('createdAt < :date', { date: twoYearsAgo })
  .execute();

// ⚠️ WARNING: Delete ALL rows (no WHERE clause)
await dataSource
  .createQueryBuilder()
  .delete()
  .from(User)
  .execute();
// Deletes EVERY user! Dangerous!
```

### DELETE with Return Values

```typescript
// Get affected rows count
const result = await dataSource
  .createQueryBuilder()
  .delete()
  .from(User)
  .where('isActive = :active', { active: false })
  .andWhere('lastLoginAt < :date', { date: oneYearAgo })
  .execute();

console.log(result.affected); // Number of rows deleted
// result.affected: 23 (deleted 23 users)

// Check if any rows were deleted
const result = await dataSource
  .createQueryBuilder()
  .delete()
  .from(User)
  .where('id = :id', { id: userId })
  .execute();

if (result.affected === 0) {
  throw new NotFoundException('User not found');
}

return { deleted: true };
```

### Real-World Examples

```typescript
export class UserService {
  // Cleanup old unverified accounts
  async cleanupUnverifiedAccounts(days: number = 30) {
    const cutoffDate = new Date();
    cutoffDate.setDate(cutoffDate.getDate() - days);

    const result = await this.dataSource
      .createQueryBuilder()
      .delete()
      .from(User)
      .where('emailVerified = :verified', { verified: false })
      .andWhere('createdAt < :cutoff', { cutoff: cutoffDate })
      .execute();

    return { deletedCount: result.affected };
  }

  // Delete specific users
  async deleteUsers(userIds: number[]) {
    if (userIds.length === 0) {
      return { deletedCount: 0 };
    }

    const result = await this.dataSource
      .createQueryBuilder()
      .delete()
      .from(User)
      .where('id IN (:...ids)', { ids: userIds })
      .execute();

    return { deletedCount: result.affected };
  }

  // Delete inactive users
  async deleteInactiveUsers(months: number = 12) {
    const cutoffDate = new Date();
    cutoffDate.setMonth(cutoffDate.getMonth() - months);

    const result = await this.dataSource
      .createQueryBuilder()
      .delete()
      .from(User)
      .where('lastLoginAt < :cutoff', { cutoff: cutoffDate })
      .andWhere('isActive = :active', { active: false })
      .execute();

    return { deletedCount: result.affected };
  }

  // Delete users without any posts
  async deleteUsersWithoutContent() {
    const result = await this.dataSource
      .createQueryBuilder()
      .delete()
      .from(User)
      .where((qb) => {
        const subQuery = qb
          .subQuery()
          .select('post.authorId')
          .from('posts', 'post')
          .getQuery();
        return `id NOT IN ${subQuery}`;
      })
      .andWhere('createdAt < :date', { date: sixMonthsAgo })
      .execute();

    return { deletedCount: result.affected };
  }

  // Batch delete in chunks (avoid locking)
  async batchDeleteInChunks(userIds: number[]) {
    const CHUNK_SIZE = 1000;
    let totalDeleted = 0;

    for (let i = 0; i < userIds.length; i += CHUNK_SIZE) {
      const chunk = userIds.slice(i, i + CHUNK_SIZE);

      const result = await this.dataSource
        .createQueryBuilder()
        .delete()
        .from(User)
        .where('id IN (:...ids)', { ids: chunk })
        .execute();

      totalDeleted += result.affected || 0;

      // Optional: Add delay to reduce database load
      if (i + CHUNK_SIZE < userIds.length) {
        await new Promise(resolve => setTimeout(resolve, 100));
      }
    }

    return { totalDeleted };
  }

  // Cascade delete (manual implementation)
  async deleteUserAndRelatedData(userId: number) {
    await this.dataSource.transaction(async (manager) => {
      // Delete related records first
      await manager
        .createQueryBuilder()
        .delete()
        .from('user_posts')
        .where('userId = :userId', { userId })
        .execute();

      await manager
        .createQueryBuilder()
        .delete()
        .from('user_comments')
        .where('userId = :userId', { userId })
        .execute();

      // Delete user last
      const result = await manager
        .createQueryBuilder()
        .delete()
        .from(User)
        .where('id = :id', { id: userId })
        .execute();

      if (result.affected === 0) {
        throw new NotFoundException('User not found');
      }
    });

    return { deleted: true };
  }
}
```

### Soft Delete vs Hard Delete

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @DeleteDateColumn()
  deletedAt: Date; // For soft deletes
}

// Hard DELETE (permanent removal)
await dataSource
  .createQueryBuilder()
  .delete()
  .from(User)
  .where('id = :id', { id: userId })
  .execute();
// Row is permanently deleted from database

// Soft DELETE (set deletedAt column)
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ deletedAt: new Date() })
  .where('id = :id', { id: userId })
  .execute();
// Row remains in database but marked as deleted

// Restore soft-deleted user
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ deletedAt: null })
  .where('id = :id', { id: userId })
  .execute();

// Query excluding soft-deleted
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.deletedAt IS NULL')
  .getMany();
```

### DELETE vs Repository.remove()

```typescript
// ❌ SLOW: Load entity and remove (triggers lifecycle hooks)
const user = await userRepository.findOne({ where: { id: 1 } });
await userRepository.remove(user);
// - SELECT query to load entity
// - DELETE query to remove
// - Triggers @BeforeRemove, @AfterRemove hooks
// - Slower for bulk deletes

// ✅ FAST: Direct DELETE with QueryBuilder
await dataSource
  .createQueryBuilder()
  .delete()
  .from(User)
  .where('id = :id', { id: 1 })
  .execute();
// - Single DELETE query
// - No entity loading
// - No lifecycle hooks
// - Much faster for bulk deletes

// Trade-offs:
// QueryBuilder DELETE:
// ✅ Faster (single query)
// ✅ Efficient for bulk operations
// ❌ Bypasses lifecycle hooks (@BeforeRemove, @AfterRemove)
// ❌ No cascade delete handling (must handle manually)
// ❌ No soft delete support (unless manual UPDATE)

// Repository.remove():
// ✅ Triggers lifecycle hooks
// ✅ Handles cascade deletes automatically
// ✅ Supports soft deletes with @DeleteDateColumn
// ❌ Slower (2 queries: SELECT + DELETE)
// ❌ Inefficient for bulk operations
```

### Best Practices

```typescript
// ✅ GOOD: Always use WHERE clause (prevent mass deletes)
await dataSource
  .createQueryBuilder()
  .delete()
  .from(User)
  .where('id = :id', { id: userId })
  .execute();

// ❌ DANGEROUS: No WHERE clause (deletes ALL rows!)
await dataSource
  .createQueryBuilder()
  .delete()
  .from(User)
  .execute();
// Deletes every single user!

// ✅ GOOD: Check affected count
const result = await dataSource
  .createQueryBuilder()
  .delete()
  .from(User)
  .where('id = :id', { id: userId })
  .execute();

if (result.affected === 0) {
  throw new NotFoundException('User not found');
}

// ✅ GOOD: Use transactions for related deletes
await dataSource.transaction(async (manager) => {
  // Delete child records first
  await manager
    .createQueryBuilder()
    .delete()
    .from('posts')
    .where('authorId = :userId', { userId })
    .execute();

  // Delete parent
  await manager
    .createQueryBuilder()
    .delete()
    .from(User)
    .where('id = :id', { id: userId })
    .execute();
});

// ✅ GOOD: Consider soft delete for important data
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({ deletedAt: new Date() })
  .where('id = :id', { id: userId })
  .execute();
// Can be restored if needed

// ✅ GOOD: Batch delete in chunks
const CHUNK_SIZE = 1000;
for (let i = 0; i < largeIdList.length; i += CHUNK_SIZE) {
  const chunk = largeIdList.slice(i, i + CHUNK_SIZE);
  await dataSource
    .createQueryBuilder()
    .delete()
    .from(User)
    .where('id IN (:...ids)', { ids: chunk })
    .execute();
}

// ✅ GOOD: Use parameters to prevent SQL injection
await dataSource
  .createQueryBuilder()
  .delete()
  .from(User)
  .where('email = :email', { email: userInput })
  .execute();

// ❌ BAD: String interpolation (SQL injection risk)
await dataSource
  .createQueryBuilder()
  .delete()
  .from(User)
  .where(`email = '${userInput}'`) // NEVER DO THIS!
  .execute();

// ✅ GOOD: Add safety checks for bulk deletes
const MAX_DELETE_COUNT = 10000;
const result = await dataSource
  .createQueryBuilder()
  .delete()
  .from(User)
  .where('isActive = :active', { active: false })
  .execute();

if (result.affected > MAX_DELETE_COUNT) {
  // Log warning or alert
  console.warn(`Large delete operation: ${result.affected} rows`);
}

// ✅ GOOD: Archive before delete
await dataSource.transaction(async (manager) => {
  // Copy to archive table
  await manager.query(`
    INSERT INTO users_archive 
    SELECT * FROM users WHERE id IN (:...ids)
  `, [userIds]);

  // Then delete
  await manager
    .createQueryBuilder()
    .delete()
    .from(User)
    .where('id IN (:...ids)', { ids: userIds })
    .execute();
});
```

**Interview Tip**: Explain **DELETE** with QueryBuilder uses **.delete()** to start, **.from(Entity)** for table, **.where()** to filter, **.execute()** to run. Emphasize **bypasses lifecycle hooks** (@BeforeRemove, @AfterRemove), no cascade handling, but **much faster** than repository.remove() (single query, no entity loading). Mention use cases: **cleanup old records** (unverified accounts), **bulk deletes** (inactive users), **conditional deletes** (users without content). Highlight **soft delete** (UPDATE deletedAt) vs **hard delete** (permanent removal). Best practices: always use WHERE clause (prevent mass deletes), check result.affected (verify deletion), use transactions for related deletes, consider soft deletes for important data, batch large deletes in chunks, archive before deleting important data, use parameters for SQL injection prevention, add safety checks for bulk operations.

</details>

<details>
<summary>79. How do you perform INSERT operations with QueryBuilder?</summary>

Use **.insert()** to start an INSERT query builder, **.into(Entity)** to specify the table, **.values()** to provide data, and **.execute()** to run the query. You can insert single rows or bulk insert multiple rows. INSERT operations bypass entity lifecycle hooks but are faster than using repository.save().

### Basic INSERT

```typescript
import { DataSource } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column()
  email: string;

  @Column({ default: true })
  isActive: boolean;

  @CreateDateColumn()
  createdAt: Date;
}

const dataSource = new DataSource(/* config */);

// Insert single row
await dataSource
  .createQueryBuilder()
  .insert()
  .into(User)
  .values({
    firstName: 'John',
    lastName: 'Doe',
    email: 'john@example.com'
  })
  .execute();
// SQL: INSERT INTO users (firstName, lastName, email) VALUES ('John', 'Doe', 'john@example.com')

// Insert with all columns
await dataSource
  .createQueryBuilder()
  .insert()
  .into(User)
  .values({
    firstName: 'Jane',
    lastName: 'Smith',
    email: 'jane@example.com',
    isActive: true
  })
  .execute();

// Insert with parameters
const userData = {
  firstName: 'Bob',
  lastName: 'Johnson',
  email: 'bob@example.com'
};

await dataSource
  .createQueryBuilder()
  .insert()
  .into(User)
  .values(userData)
  .execute();
```

### Bulk INSERT (Multiple Rows)

```typescript
// Insert multiple rows
const users = [
  { firstName: 'John', lastName: 'Doe', email: 'john@example.com' },
  { firstName: 'Jane', lastName: 'Smith', email: 'jane@example.com' },
  { firstName: 'Bob', lastName: 'Johnson', email: 'bob@example.com' }
];

await dataSource
  .createQueryBuilder()
  .insert()
  .into(User)
  .values(users)
  .execute();
// SQL: INSERT INTO users (firstName, lastName, email)
//      VALUES ('John', 'Doe', 'john@example.com'),
//             ('Jane', 'Smith', 'jane@example.com'),
//             ('Bob', 'Johnson', 'bob@example.com')

// Bulk insert with dynamic data
const userDataArray = await fetchUsersFromAPI();

await dataSource
  .createQueryBuilder()
  .insert()
  .into(User)
  .values(userDataArray)
  .execute();
```

### INSERT with Return Values

```typescript
// Get inserted IDs
const result = await dataSource
  .createQueryBuilder()
  .insert()
  .into(User)
  .values({
    firstName: 'John',
    lastName: 'Doe',
    email: 'john@example.com'
  })
  .execute();

console.log(result.identifiers); // [{ id: 1 }]
console.log(result.generatedMaps); // [{ id: 1, createdAt: Date }]
console.log(result.raw); // Database-specific raw result

// Access the generated ID
const newUserId = result.identifiers[0].id;
console.log(`Created user with ID: ${newUserId}`);

// Bulk insert - get all IDs
const result = await dataSource
  .createQueryBuilder()
  .insert()
  .into(User)
  .values([
    { firstName: 'John', lastName: 'Doe', email: 'john@example.com' },
    { firstName: 'Jane', lastName: 'Smith', email: 'jane@example.com' }
  ])
  .execute();

console.log(result.identifiers); // [{ id: 1 }, { id: 2 }]
// Access all generated IDs
const insertedIds = result.identifiers.map(item => item.id);
console.log(insertedIds); // [1, 2]
```

### INSERT with SQL Functions

```typescript
// Insert with database functions
await dataSource
  .createQueryBuilder()
  .insert()
  .into(User)
  .values({
    firstName: 'John',
    lastName: 'Doe',
    email: 'john@example.com',
    createdAt: () => 'NOW()' // Use SQL function
  })
  .execute();

// Insert with expressions
await dataSource
  .createQueryBuilder()
  .insert()
  .into(User)
  .values({
    firstName: 'JOHN',
    lastName: 'DOE',
    email: () => "LOWER('JOHN@EXAMPLE.COM')", // Convert to lowercase
    registrationCode: () => "MD5(RANDOM()::text)" // Generate random hash
  })
  .execute();

// Insert with calculated values
await dataSource
  .createQueryBuilder()
  .insert()
  .into(User)
  .values({
    firstName: 'John',
    lastName: 'Doe',
    email: 'john@example.com',
    displayName: () => "firstName || ' ' || lastName", // Concatenate
    age: () => "EXTRACT(YEAR FROM CURRENT_DATE) - birthYear"
  })
  .execute();
```

### INSERT with Subquery

```typescript
// Insert data from another table
await dataSource
  .createQueryBuilder()
  .insert()
  .into('user_archive')
  .values([
    {
      userId: () => 'id',
      firstName: () => 'firstName',
      archivedAt: () => 'NOW()'
    }
  ])
  .from((qb) => {
    return qb
      .select('user.id', 'id')
      .addSelect('user.firstName', 'firstName')
      .from(User, 'user')
      .where('user.isActive = :active', { active: false });
  })
  .execute();
// Copies data from users table to user_archive

// Insert aggregated data
await dataSource
  .createQueryBuilder()
  .insert()
  .into('daily_stats')
  .values([
    {
      date: () => 'CURRENT_DATE',
      userCount: () => 'COUNT(*)',
      newUsers: () => "COUNT(CASE WHEN DATE(createdAt) = CURRENT_DATE THEN 1 END)"
    }
  ])
  .from(User, 'user')
  .execute();
```

### INSERT ... ON CONFLICT (Upsert)

```typescript
// PostgreSQL: INSERT with ON CONFLICT (upsert)
await dataSource
  .createQueryBuilder()
  .insert()
  .into(User)
  .values({
    email: 'john@example.com',
    firstName: 'John',
    lastName: 'Doe'
  })
  .orUpdate(
    ['firstName', 'lastName'], // Columns to update on conflict
    ['email'] // Conflict target (unique column)
  )
  .execute();
// SQL: INSERT INTO users (email, firstName, lastName)
//      VALUES ('john@example.com', 'John', 'Doe')
//      ON CONFLICT (email) DO UPDATE SET firstName = EXCLUDED.firstName, lastName = EXCLUDED.lastName

// MySQL: INSERT with ON DUPLICATE KEY UPDATE
await dataSource
  .createQueryBuilder()
  .insert()
  .into(User)
  .values({
    email: 'john@example.com',
    firstName: 'John',
    lastName: 'Doe'
  })
  .orUpdate(['firstName', 'lastName'], ['email'])
  .execute();

// Upsert with additional update expressions
await dataSource
  .createQueryBuilder()
  .insert()
  .into(User)
  .values({
    email: 'john@example.com',
    firstName: 'John',
    lastName: 'Doe',
    loginCount: 1
  })
  .orUpdate(
    ['firstName', 'lastName', 'loginCount'],
    ['email'],
    {
      skipUpdateIfNoValuesChanged: true, // Skip if values are the same
      upsertType: 'on-conflict-do-update' // PostgreSQL syntax
    }
  )
  .execute();
```

### INSERT ... IGNORE (Skip on Duplicate)

```typescript
// MySQL: INSERT IGNORE (skip duplicates silently)
await dataSource
  .createQueryBuilder()
  .insert()
  .into(User)
  .values([
    { email: 'john@example.com', firstName: 'John', lastName: 'Doe' },
    { email: 'jane@example.com', firstName: 'Jane', lastName: 'Smith' }
  ])
  .orIgnore()
  .execute();
// SQL: INSERT IGNORE INTO users (email, firstName, lastName) VALUES (...)
// Skips rows that would cause duplicate key errors

// PostgreSQL: ON CONFLICT DO NOTHING
await dataSource
  .createQueryBuilder()
  .insert()
  .into(User)
  .values({
    email: 'john@example.com',
    firstName: 'John',
    lastName: 'Doe'
  })
  .orIgnore()
  .execute();
// SQL: INSERT INTO users (email, firstName, lastName)
//      VALUES ('john@example.com', 'John', 'Doe')
//      ON CONFLICT DO NOTHING
```

### Real-World Examples

```typescript
export class UserService {
  // Create single user
  async createUser(userData: CreateUserDto) {
    const result = await this.dataSource
      .createQueryBuilder()
      .insert()
      .into(User)
      .values({
        firstName: userData.firstName,
        lastName: userData.lastName,
        email: userData.email.toLowerCase(),
        passwordHash: await this.hashPassword(userData.password),
        createdAt: new Date()
      })
      .execute();

    return { userId: result.identifiers[0].id };
  }

  // Bulk import users
  async bulkImportUsers(users: CreateUserDto[]) {
    if (users.length === 0) return { imported: 0 };

    const BATCH_SIZE = 1000;
    let totalImported = 0;

    for (let i = 0; i < users.length; i += BATCH_SIZE) {
      const batch = users.slice(i, i + BATCH_SIZE);
      
      const values = batch.map(user => ({
        firstName: user.firstName,
        lastName: user.lastName,
        email: user.email.toLowerCase(),
        createdAt: new Date()
      }));

      const result = await this.dataSource
        .createQueryBuilder()
        .insert()
        .into(User)
        .values(values)
        .orIgnore() // Skip duplicates
        .execute();

      totalImported += result.identifiers.length;
    }

    return { imported: totalImported };
  }

  // Upsert user (insert or update)
  async upsertUser(email: string, userData: Partial<User>) {
    const result = await this.dataSource
      .createQueryBuilder()
      .insert()
      .into(User)
      .values({
        email: email.toLowerCase(),
        firstName: userData.firstName,
        lastName: userData.lastName,
        isActive: userData.isActive ?? true
      })
      .orUpdate(
        ['firstName', 'lastName', 'isActive'],
        ['email'],
        { skipUpdateIfNoValuesChanged: true }
      )
      .execute();

    return { 
      userId: result.identifiers[0]?.id,
      isNew: result.raw?.affectedRows === 1
    };
  }

  // Copy users to archive
  async archiveInactiveUsers(days: number = 365) {
    const cutoffDate = new Date();
    cutoffDate.setDate(cutoffDate.getDate() - days);

    const result = await this.dataSource
      .createQueryBuilder()
      .insert()
      .into('user_archive')
      .values([
        {
          userId: () => 'id',
          firstName: () => 'firstName',
          lastName: () => 'lastName',
          email: () => 'email',
          archivedAt: () => 'NOW()',
          archivedReason: () => "'Inactivity'"
        }
      ])
      .from((qb) => {
        return qb
          .select([
            'user.id',
            'user.firstName',
            'user.lastName',
            'user.email'
          ])
          .from(User, 'user')
          .where('user.lastLoginAt < :cutoff', { cutoff: cutoffDate })
          .andWhere('user.isActive = :active', { active: false });
      })
      .execute();

    return { archivedCount: result.identifiers.length };
  }

  // Insert with auto-generated fields
  async createUserWithDefaults(email: string, firstName: string) {
    const result = await this.dataSource
      .createQueryBuilder()
      .insert()
      .into(User)
      .values({
        email: email.toLowerCase(),
        firstName,
        lastName: '',
        username: () => `CONCAT('user_', FLOOR(RANDOM() * 1000000))`,
        apiKey: () => "MD5(RANDOM()::text)",
        registrationIp: () => "INET_CLIENT_ADDR()",
        createdAt: () => 'NOW()'
      })
      .execute();

    return { userId: result.identifiers[0].id };
  }

  // Batch insert with error handling
  async batchInsertWithErrorHandling(users: CreateUserDto[]) {
    const results = {
      success: [] as number[],
      failed: [] as { email: string; error: string }[]
    };

    for (const user of users) {
      try {
        const result = await this.dataSource
          .createQueryBuilder()
          .insert()
          .into(User)
          .values({
            firstName: user.firstName,
            lastName: user.lastName,
            email: user.email.toLowerCase()
          })
          .execute();

        results.success.push(result.identifiers[0].id);
      } catch (error) {
        results.failed.push({
          email: user.email,
          error: error.message
        });
      }
    }

    return results;
  }

  // Insert with relationship IDs
  async createPost(authorId: number, postData: CreatePostDto) {
    const result = await this.dataSource
      .createQueryBuilder()
      .insert()
      .into('posts')
      .values({
        title: postData.title,
        content: postData.content,
        authorId: authorId, // Foreign key
        categoryId: postData.categoryId,
        isPublished: false,
        createdAt: new Date()
      })
      .execute();

    return { postId: result.identifiers[0].id };
  }
}
```

### INSERT vs Repository.save()

```typescript
// ❌ SLOWER: Using repository.save()
const user = userRepository.create({
  firstName: 'John',
  lastName: 'Doe',
  email: 'john@example.com'
});
await userRepository.save(user);
// - Creates entity instance
// - Validates entity
// - Triggers @BeforeInsert, @AfterInsert hooks
// - Automatically sets @CreateDateColumn
// - Slower but provides full entity features

// ✅ FASTER: Direct INSERT with QueryBuilder
await dataSource
  .createQueryBuilder()
  .insert()
  .into(User)
  .values({
    firstName: 'John',
    lastName: 'Doe',
    email: 'john@example.com'
  })
  .execute();
// - Single INSERT query
// - No entity instantiation
// - No lifecycle hooks
// - @CreateDateColumn NOT set automatically
// - Much faster for bulk inserts

// Trade-offs:
// QueryBuilder INSERT:
// ✅ Faster (single query, no overhead)
// ✅ Efficient for bulk inserts
// ✅ Can use SQL functions and expressions
// ❌ Bypasses lifecycle hooks (@BeforeInsert, @AfterInsert)
// ❌ No entity validation
// ❌ @CreateDateColumn, @UpdateDateColumn not set automatically
// ❌ Must manually handle default values

// Repository.save():
// ✅ Triggers lifecycle hooks
// ✅ Validates entity
// ✅ Automatically sets timestamps
// ✅ Handles default values
// ✅ Type-safe entity instance
// ❌ Slower (entity instantiation + validation)
// ❌ Inefficient for bulk inserts
```

### Best Practices

```typescript
// ✅ GOOD: Validate data before inserting
const validateEmail = (email: string) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);

if (!validateEmail(userData.email)) {
  throw new BadRequestException('Invalid email');
}

await dataSource
  .createQueryBuilder()
  .insert()
  .into(User)
  .values(userData)
  .execute();

// ✅ GOOD: Use transactions for related inserts
await dataSource.transaction(async (manager) => {
  const userResult = await manager
    .createQueryBuilder()
    .insert()
    .into(User)
    .values({ firstName: 'John', lastName: 'Doe', email: 'john@example.com' })
    .execute();

  const userId = userResult.identifiers[0].id;

  await manager
    .createQueryBuilder()
    .insert()
    .into('user_profile')
    .values({ userId, bio: 'Hello!', avatar: 'default.png' })
    .execute();
});

// ✅ GOOD: Batch large inserts
const BATCH_SIZE = 1000;
for (let i = 0; i < largeDataset.length; i += BATCH_SIZE) {
  const batch = largeDataset.slice(i, i + BATCH_SIZE);
  await dataSource
    .createQueryBuilder()
    .insert()
    .into(User)
    .values(batch)
    .execute();
}

// ✅ GOOD: Use orIgnore() to skip duplicates
await dataSource
  .createQueryBuilder()
  .insert()
  .into(User)
  .values(users)
  .orIgnore()
  .execute();

// ✅ GOOD: Use orUpdate() for upsert
await dataSource
  .createQueryBuilder()
  .insert()
  .into(User)
  .values({ email: 'john@example.com', firstName: 'John' })
  .orUpdate(['firstName'], ['email'])
  .execute();

// ✅ GOOD: Handle errors properly
try {
  const result = await dataSource
    .createQueryBuilder()
    .insert()
    .into(User)
    .values(userData)
    .execute();
  return { userId: result.identifiers[0].id };
} catch (error) {
  if (error.code === '23505') { // PostgreSQL unique violation
    throw new ConflictException('Email already exists');
  }
  throw error;
}

// ❌ BAD: Not using parameters (SQL injection risk)
await dataSource
  .createQueryBuilder()
  .insert()
  .into(User)
  .values({
    firstName: userInput // OK if userInput is a variable
  })
  .execute();

// ✅ GOOD: Set timestamps manually if needed
await dataSource
  .createQueryBuilder()
  .insert()
  .into(User)
  .values({
    firstName: 'John',
    lastName: 'Doe',
    email: 'john@example.com',
    createdAt: new Date(), // Manual timestamp
    updatedAt: new Date()
  })
  .execute();

// ✅ GOOD: Check insert result
const result = await dataSource
  .createQueryBuilder()
  .insert()
  .into(User)
  .values(userData)
  .execute();

if (result.identifiers.length === 0) {
  throw new Error('Insert failed');
}

// ✅ GOOD: Use returning() for PostgreSQL (get inserted data)
const result = await dataSource
  .createQueryBuilder()
  .insert()
  .into(User)
  .values({ firstName: 'John', lastName: 'Doe', email: 'john@example.com' })
  .returning(['id', 'createdAt'])
  .execute();

console.log(result.generatedMaps); // [{ id: 1, createdAt: Date }]
```

**Interview Tip**: Explain **INSERT** with QueryBuilder uses **.insert()** to start, **.into(Entity)** for table, **.values()** for data, **.execute()** to run. Supports **single row** (one object) or **bulk insert** (array of objects). Emphasize **bypasses lifecycle hooks** (@BeforeInsert, @AfterInsert), no validation, @CreateDateColumn NOT set automatically, but **much faster** than repository.save() especially for bulk inserts. Mention **result.identifiers** to get generated IDs, **orUpdate()** for upsert (INSERT ... ON CONFLICT), **orIgnore()** to skip duplicates. Highlight use cases: bulk data import, copy data between tables, high-performance inserts. Best practices: batch large inserts (1000 rows per batch), use transactions for related inserts, validate data manually, handle duplicate key errors, set timestamps manually if needed, use orUpdate for upsert patterns, prefer QueryBuilder for bulk operations and repository.save() for single inserts with validation.

</details>

<details>
<summary>80. How do you use raw SQL expressions in QueryBuilder?</summary>

Use **raw SQL expressions** within QueryBuilder methods for complex calculations, database-specific functions, or operations not directly supported by TypeORM. Use **() => 'SQL'** syntax in set/values, **addSelect()** with raw SQL, or **setParameter()** for parameterized raw SQL to prevent injection.

### Raw SQL in SELECT

```typescript
import { DataSource } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column()
  age: number;

  @Column()
  salary: number;

  @Column()
  createdAt: Date;
}

const dataSource = new DataSource(/* config */);

// Concatenate strings
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.id', 'id')
  .addSelect("user.firstName || ' ' || user.lastName", 'fullName')
  .getRawMany();
// SQL: SELECT user.id, user.firstName || ' ' || user.lastName as fullName

// Math expressions
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.id', 'id')
  .addSelect('user.salary * 12', 'annualSalary')
  .addSelect('user.salary * 0.3', 'tax')
  .addSelect('user.salary * 0.7', 'netSalary')
  .getRawMany();

// CASE expressions
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.id', 'id')
  .addSelect('user.firstName', 'name')
  .addSelect(
    `CASE 
      WHEN user.age < 18 THEN 'Minor'
      WHEN user.age BETWEEN 18 AND 65 THEN 'Adult'
      ELSE 'Senior'
    END`,
    'ageGroup'
  )
  .getRawMany();

// Date functions
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.id', 'id')
  .addSelect('EXTRACT(YEAR FROM user.createdAt)', 'joinYear')
  .addSelect('EXTRACT(MONTH FROM user.createdAt)', 'joinMonth')
  .addSelect('AGE(CURRENT_DATE, user.createdAt)', 'accountAge')
  .addSelect("TO_CHAR(user.createdAt, 'YYYY-MM-DD')", 'formattedDate')
  .getRawMany();

// String functions
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.id', 'id')
  .addSelect('UPPER(user.firstName)', 'firstNameUpper')
  .addSelect('LOWER(user.email)', 'emailLower')
  .addSelect('LENGTH(user.firstName)', 'nameLength')
  .addSelect('SUBSTRING(user.email, 1, 3)', 'emailPrefix')
  .getRawMany();
```

### Raw SQL in WHERE

```typescript
// Complex WHERE conditions
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('EXTRACT(YEAR FROM user.createdAt) = :year', { year: 2024 })
  .getMany();

// Date comparisons
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where("DATE(user.createdAt) = CURRENT_DATE")
  .getMany();

// String operations
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where("LOWER(user.email) LIKE LOWER(:pattern)", { pattern: '%@gmail.com' })
  .getMany();

// Math operations
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.salary * 12 > :annualThreshold', { annualThreshold: 100000 })
  .getMany();

// JSON operations (PostgreSQL)
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where("user.preferences->>'theme' = :theme", { theme: 'dark' })
  .getMany();

// Array operations (PostgreSQL)
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where(':tag = ANY(user.tags)', { tag: 'admin' })
  .getMany();

// Full-text search (PostgreSQL)
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where(
    "to_tsvector('english', user.bio) @@ to_tsquery('english', :query)",
    { query: 'developer & typescript' }
  )
  .getMany();
```

### Raw SQL in UPDATE

```typescript
// Update with expressions
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({
    salary: () => 'salary * 1.1', // 10% raise
    lastRaiseDate: () => 'CURRENT_DATE'
  })
  .where('department = :dept', { dept: 'Engineering' })
  .execute();

// Update with CASE
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({
    tier: () => `CASE 
      WHEN points >= 1000 THEN 'gold'
      WHEN points >= 500 THEN 'silver'
      ELSE 'bronze'
    END`
  })
  .execute();

// Update with concatenation
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({
    fullName: () => "firstName || ' ' || lastName",
    displayName: () => "UPPER(SUBSTRING(firstName, 1, 1)) || LOWER(SUBSTRING(firstName, 2))"
  })
  .execute();

// Update with math
await dataSource
  .createQueryBuilder()
  .update(User)
  .set({
    balance: () => 'balance + :amount',
    totalDeposits: () => 'totalDeposits + 1'
  })
  .where('id = :id', { id: userId, amount: 100 })
  .execute();
```

### Raw SQL in INSERT

```typescript
// Insert with functions
await dataSource
  .createQueryBuilder()
  .insert()
  .into(User)
  .values({
    firstName: 'John',
    lastName: 'Doe',
    email: () => "LOWER('JOHN@EXAMPLE.COM')",
    createdAt: () => 'NOW()',
    uuid: () => 'UUID_GENERATE_V4()'
  })
  .execute();

// Insert with calculations
await dataSource
  .createQueryBuilder()
  .insert()
  .into(User)
  .values({
    firstName: 'John',
    lastName: 'Doe',
    displayName: () => "firstName || ' ' || lastName",
    age: () => 'EXTRACT(YEAR FROM CURRENT_DATE) - birthYear'
  })
  .execute();

// Insert with CASE
await dataSource
  .createQueryBuilder()
  .insert()
  .into(User)
  .values({
    firstName: 'John',
    lastName: 'Doe',
    status: () => `CASE 
      WHEN age < 18 THEN 'minor'
      ELSE 'adult'
    END`
  })
  .execute();
```

### Raw SQL in ORDER BY

```typescript
// Custom sort order
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .orderBy('CASE WHEN user.isPremium THEN 0 ELSE 1 END', 'ASC')
  .addOrderBy('user.createdAt', 'DESC')
  .getMany();
// Premium users first, then by creation date

// Sort by computed field
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .orderBy('user.salary * 12', 'DESC') // Annual salary
  .getMany();

// Custom priority sort
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .orderBy(
    `CASE user.status
      WHEN 'urgent' THEN 1
      WHEN 'high' THEN 2
      WHEN 'medium' THEN 3
      ELSE 4
    END`,
    'ASC'
  )
  .getMany();

// Sort by expression
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .orderBy('LENGTH(user.firstName)', 'ASC') // Shortest names first
  .getMany();
```

### Raw SQL in GROUP BY and HAVING

```typescript
// Group by date parts
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('EXTRACT(YEAR FROM user.createdAt)', 'year')
  .addSelect('EXTRACT(MONTH FROM user.createdAt)', 'month')
  .addSelect('COUNT(*)', 'userCount')
  .groupBy('year')
  .addGroupBy('month')
  .orderBy('year', 'DESC')
  .addOrderBy('month', 'DESC')
  .getRawMany();

// Group by expression
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select(
    `CASE 
      WHEN user.age < 18 THEN 'Under 18'
      WHEN user.age BETWEEN 18 AND 30 THEN '18-30'
      WHEN user.age BETWEEN 31 AND 50 THEN '31-50'
      ELSE 'Over 50'
    END`,
    'ageGroup'
  )
  .addSelect('COUNT(*)', 'count')
  .groupBy('ageGroup')
  .getRawMany();

// HAVING with raw SQL
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city', 'city')
  .addSelect('AVG(user.salary)', 'avgSalary')
  .groupBy('user.city')
  .having('AVG(user.salary) > :threshold', { threshold: 50000 })
  .getRawMany();
```

### Complex Raw SQL Examples

```typescript
// Window functions (PostgreSQL)
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.id', 'id')
  .addSelect('user.firstName', 'name')
  .addSelect('user.salary', 'salary')
  .addSelect(
    'ROW_NUMBER() OVER (PARTITION BY user.department ORDER BY user.salary DESC)',
    'rankInDepartment'
  )
  .addSelect(
    'AVG(user.salary) OVER (PARTITION BY user.department)',
    'departmentAvgSalary'
  )
  .getRawMany();

// CTE (Common Table Expression) simulation
const results = await dataSource
  .createQueryBuilder()
  .select('subquery.*')
  .from((qb) => {
    return qb
      .select('user.city', 'city')
      .addSelect('COUNT(*)', 'userCount')
      .addSelect('AVG(user.salary)', 'avgSalary')
      .from(User, 'user')
      .groupBy('user.city');
  }, 'subquery')
  .where('subquery.userCount > :min', { min: 10 })
  .orderBy('subquery.avgSalary', 'DESC')
  .getRawMany();

// JSON aggregation (PostgreSQL)
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.city', 'city')
  .addSelect(
    "JSON_AGG(JSON_BUILD_OBJECT('id', user.id, 'name', user.firstName))",
    'users'
  )
  .groupBy('user.city')
  .getRawMany();
// Groups users as JSON array per city

// Geospatial queries (PostGIS)
const nearbyUsers = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.id', 'id')
  .addSelect('user.firstName', 'name')
  .addSelect(
    'ST_Distance(user.location, ST_MakePoint(:lng, :lat)::geography)',
    'distance'
  )
  .where(
    'ST_DWithin(user.location, ST_MakePoint(:lng, :lat)::geography, :radius)',
    { lng: -73.9857, lat: 40.7484, radius: 5000 } // 5km radius
  )
  .orderBy('distance', 'ASC')
  .getRawMany();
```

### Real-World Examples

```typescript
export class AnalyticsService {
  // User growth statistics
  async getUserGrowthStats(months: number = 12) {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .select("TO_CHAR(user.createdAt, 'YYYY-MM')", 'month')
      .addSelect('COUNT(*)', 'newUsers')
      .addSelect(
        'SUM(COUNT(*)) OVER (ORDER BY TO_CHAR(user.createdAt, \'YYYY-MM\'))',
        'cumulativeUsers'
      )
      .addSelect(
        `COUNT(CASE WHEN user.isPremium THEN 1 END)`,
        'premiumUsers'
      )
      .where('user.createdAt >= NOW() - INTERVAL :months MONTH', { months })
      .groupBy('month')
      .orderBy('month', 'ASC')
      .getRawMany();
  }

  // Revenue by cohort
  async getRevenueByCohort() {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .select("TO_CHAR(user.createdAt, 'YYYY-MM')", 'cohort')
      .addSelect(
        'COUNT(DISTINCT user.id)',
        'usersInCohort'
      )
      .addSelect(
        'SUM(CASE WHEN sub.status = \'active\' THEN sub.amount ELSE 0 END)',
        'totalRevenue'
      )
      .addSelect(
        'AVG(CASE WHEN sub.status = \'active\' THEN sub.amount ELSE 0 END)',
        'avgRevenuePerUser'
      )
      .leftJoin('user.subscriptions', 'sub')
      .groupBy('cohort')
      .orderBy('cohort', 'DESC')
      .getRawMany();
  }

  // User engagement score
  async getUserEngagementScores() {
    return this.dataSource
      .createQueryBuilder(User, 'user')
      .select('user.id', 'userId')
      .addSelect('user.firstName', 'name')
      .addSelect(
        `(
          COALESCE(postCount, 0) * 3 +
          COALESCE(commentCount, 0) * 2 +
          COALESCE(likeCount, 0) * 1 +
          CASE WHEN user.lastLoginAt > NOW() - INTERVAL '7 days' THEN 10 ELSE 0 END
        )`,
        'engagementScore'
      )
      .leftJoin(
        (qb) =>
          qb
            .select('authorId', 'authorId')
            .addSelect('COUNT(*)', 'postCount')
            .from('posts', 'p')
            .groupBy('authorId'),
        'posts',
        'posts.authorId = user.id'
      )
      .leftJoin(
        (qb) =>
          qb
            .select('userId', 'userId')
            .addSelect('COUNT(*)', 'commentCount')
            .from('comments', 'c')
            .groupBy('userId'),
        'comments',
        'comments.userId = user.id'
      )
      .leftJoin(
        (qb) =>
          qb
            .select('userId', 'userId')
            .addSelect('COUNT(*)', 'likeCount')
            .from('likes', 'l')
            .groupBy('userId'),
        'likes',
        'likes.userId = user.id'
      )
      .orderBy('engagementScore', 'DESC')
      .take(100)
      .getRawMany();
  }

  // Time-based patterns
  async getUserActivityByHourOfDay() {
    return this.dataSource
      .createQueryBuilder()
      .select('EXTRACT(HOUR FROM activity.createdAt)', 'hour')
      .addSelect('COUNT(*)', 'activityCount')
      .addSelect('COUNT(DISTINCT activity.userId)', 'uniqueUsers')
      .from('user_activities', 'activity')
      .where('activity.createdAt >= NOW() - INTERVAL \'30 days\'')
      .groupBy('hour')
      .orderBy('hour', 'ASC')
      .getRawMany();
  }
}
```

### Best Practices

```typescript
// ✅ GOOD: Use parameters with raw SQL
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('LOWER(user.email) LIKE LOWER(:pattern)', { pattern: `%${searchTerm}%` })
  .getMany();

// ❌ BAD: String interpolation (SQL injection risk!)
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where(`LOWER(user.email) LIKE '%${searchTerm}%'`) // NEVER DO THIS!
  .getMany();

// ✅ GOOD: Use database-agnostic SQL when possible
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.isActive = :active', { active: true })
  .getMany();

// ✅ GOOD: Document database-specific SQL
// PostgreSQL-specific full-text search
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where(
    "to_tsvector('english', user.bio) @@ to_tsquery('english', :query)",
    { query: searchQuery }
  )
  .getMany();
// Note: Requires PostgreSQL with tsvector support

// ✅ GOOD: Use TypeORM methods when available
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.age > :age', { age: 18 })
  .orderBy('user.createdAt', 'DESC')
  .getMany();

// ❌ AVOID: Raw SQL when TypeORM has built-in support
const users = await dataSource
  .createQueryBuilder(User, 'user')
  .where('user.age > :age AND user.createdAt > :date', { age: 18, date: lastWeek })
  .getMany();
// Better: Use .andWhere() for clarity

// ✅ GOOD: Type the results of raw queries
interface UserStats {
  userId: number;
  name: string;
  totalPosts: number;
  avgLikes: number;
}

const stats: UserStats[] = await dataSource
  .createQueryBuilder(User, 'user')
  .select('user.id', 'userId')
  .addSelect('user.firstName', 'name')
  .addSelect('COUNT(post.id)', 'totalPosts')
  .addSelect('AVG(post.likeCount)', 'avgLikes')
  .leftJoin('user.posts', 'post')
  .groupBy('user.id')
  .getRawMany();

// ✅ GOOD: Test raw SQL in different databases
// MySQL
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select("DATE_FORMAT(user.createdAt, '%Y-%m')", 'month')
  .getRawMany();

// PostgreSQL
const results = await dataSource
  .createQueryBuilder(User, 'user')
  .select("TO_CHAR(user.createdAt, 'YYYY-MM')", 'month')
  .getRawMany();
```

**Interview Tip**: Explain **raw SQL expressions** allow using database-specific functions, complex calculations, or operations not directly supported by TypeORM. Use **() => 'SQL'** syntax in set/values for UPDATE/INSERT, **addSelect()** with raw SQL strings for SELECT, always **use parameters** (:paramName) to prevent SQL injection. Mention use cases: **string operations** (CONCAT, UPPER, LOWER), **date functions** (EXTRACT, TO_CHAR), **CASE expressions** (conditional logic), **window functions** (ROW_NUMBER, AVG OVER), **aggregations** (GROUP BY expressions), **JSON operations** (PostgreSQL). Best practices: always use parameterized queries, document database-specific SQL, prefer TypeORM methods when available (better portability), type the results of getRawMany(), test queries across target databases, use raw SQL only when necessary (TypeORM methods are database-agnostic), be aware of SQL injection risks with string concatenation.

</details>