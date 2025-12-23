## **Entities**

<details>
<summary>11. What is an Entity in TypeORM?</summary>

An **Entity** in TypeORM is a class that maps to a database table. Each instance of the entity class represents a row in that table. Entities are the core building blocks of TypeORM and define the structure, relationships, and behavior of your data.

### Basic Entity Definition

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

// @Entity() decorator marks this class as a database entity
@Entity()
export class User {
  // Primary key with auto-increment
  @PrimaryGeneratedColumn()
  id: number;

  // Simple column
  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column({ unique: true })
  email: string;

  @Column({ default: true })
  isActive: boolean;
}
```

### Entity with Custom Table Name

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

// Specify custom table name (default would be 'user')
@Entity('users') // Table name in database will be 'users'
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;
}

// With schema specification (PostgreSQL)
@Entity({ name: 'users', schema: 'public' })
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;
}
```

### Complete Entity Example

```typescript
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  UpdateDateColumn,
  DeleteDateColumn,
} from 'typeorm';

@Entity('users')
export class User {
  // Primary key
  @PrimaryGeneratedColumn('uuid')
  id: string;

  // Basic columns
  @Column({ length: 100 })
  firstName: string;

  @Column({ length: 100 })
  lastName: string;

  @Column({ unique: true })
  email: string;

  @Column({ select: false }) // Not selected by default in queries
  password: string;

  @Column({ nullable: true })
  phoneNumber: string | null;

  @Column({ type: 'int', default: 0 })
  age: number;

  @Column({ default: true })
  isActive: boolean;

  // Timestamp columns
  @CreateDateColumn() // Automatically set on insert
  createdAt: Date;

  @UpdateDateColumn() // Automatically updated on every update
  updatedAt: Date;

  @DeleteDateColumn() // For soft deletes
  deletedAt: Date | null;
}
```

### Entity with Methods

```typescript
import { Entity, PrimaryGeneratedColumn, Column, BeforeInsert } from 'typeorm';
import * as bcrypt from 'bcrypt';

@Entity('users')
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column({ unique: true })
  email: string;

  @Column({ select: false })
  password: string;

  // Custom getter method
  get fullName(): string {
    return `${this.firstName} ${this.lastName}`;
  }

  // Instance method
  async validatePassword(plainPassword: string): Promise<boolean> {
    return bcrypt.compare(plainPassword, this.password);
  }

  // Lifecycle hook - runs before entity is inserted
  @BeforeInsert()
  async hashPassword() {
    if (this.password) {
      this.password = await bcrypt.hash(this.password, 10);
    }
  }

  // Transform email to lowercase before insert
  @BeforeInsert()
  normalizeEmail() {
    this.email = this.email.toLowerCase().trim();
  }
}

// Usage
const user = new User();
user.firstName = 'John';
user.lastName = 'Doe';
user.email = 'JOHN@EXAMPLE.COM'; // Will be normalized to john@example.com
user.password = 'mypassword'; // Will be hashed before saving

await userRepository.save(user);
console.log(user.fullName); // "John Doe"

const isValid = await user.validatePassword('mypassword'); // true
```

### Entity with Enums

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

// Define enum
export enum UserRole {
  ADMIN = 'admin',
  USER = 'user',
  MODERATOR = 'moderator',
}

export enum UserStatus {
  ACTIVE = 'active',
  INACTIVE = 'inactive',
  SUSPENDED = 'suspended',
}

@Entity('users')
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Enum column
  @Column({
    type: 'enum',
    enum: UserRole,
    default: UserRole.USER,
  })
  role: UserRole;

  @Column({
    type: 'enum',
    enum: UserStatus,
    default: UserStatus.ACTIVE,
  })
  status: UserStatus;
}

// Usage
const user = new User();
user.name = 'John';
user.role = UserRole.ADMIN;
user.status = UserStatus.ACTIVE;
```

### Entity Registration

```typescript
// data-source.ts
import { DataSource } from 'typeorm';
import { User } from './entities/User';
import { Post } from './entities/Post';

export const AppDataSource = new DataSource({
  type: 'postgres',
  host: 'localhost',
  port: 5432,
  username: 'postgres',
  password: 'password',
  database: 'mydb',
  
  // Register entities
  entities: [User, Post],
  
  // Or load from directory
  // entities: ['src/entities/**/*.ts'],
  // For production (compiled JS)
  // entities: ['dist/entities/**/*.js'],
  
  synchronize: false,
});
```

### Active Record vs Data Mapper Pattern

```typescript
// Active Record Pattern
import { Entity, PrimaryGeneratedColumn, Column, BaseEntity } from 'typeorm';

@Entity()
export class User extends BaseEntity {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  email: string;
}

// Usage with Active Record
const user = new User();
user.name = 'John';
user.email = 'john@example.com';
await user.save(); // Method on entity

const users = await User.find(); // Static method on entity
const john = await User.findOneBy({ name: 'John' });
await john.remove();

// Data Mapper Pattern (Recommended for larger applications)
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  email: string;
}

// Usage with Data Mapper (Repository)
import { AppDataSource } from './data-source';

const userRepository = AppDataSource.getRepository(User);

const user = new User();
user.name = 'John';
user.email = 'john@example.com';
await userRepository.save(user); // Method on repository

const users = await userRepository.find();
const john = await userRepository.findOneBy({ name: 'John' });
await userRepository.remove(john);
```

### Real-World Production Entity

```typescript
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  UpdateDateColumn,
  DeleteDateColumn,
  Index,
  BeforeInsert,
  BeforeUpdate,
} from 'typeorm';
import * as bcrypt from 'bcrypt';

@Entity('users')
@Index(['email']) // Index for faster email lookups
@Index(['lastName', 'firstName']) // Composite index
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ length: 50 })
  firstName: string;

  @Column({ length: 50 })
  lastName: string;

  @Column({ unique: true, length: 255 })
  @Index() // Additional index decorator
  email: string;

  @Column({ select: false }) // Never select by default (security)
  password: string;

  @Column({ nullable: true, length: 20 })
  phoneNumber: string | null;

  @Column({
    type: 'enum',
    enum: UserRole,
    default: UserRole.USER,
  })
  role: UserRole;

  @Column({ default: true })
  isActive: boolean;

  @Column({ default: false })
  isEmailVerified: boolean;

  @Column({ nullable: true })
  emailVerifiedAt: Date | null;

  @Column({ nullable: true })
  lastLoginAt: Date | null;

  @Column({ type: 'int', default: 0 })
  loginAttempts: number;

  @Column({ nullable: true })
  lockedUntil: Date | null;

  // Timestamps
  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  @DeleteDateColumn() // Soft delete support
  deletedAt: Date | null;

  // Computed property
  get fullName(): string {
    return `${this.firstName} ${this.lastName}`;
  }

  get isLocked(): boolean {
    return this.lockedUntil !== null && this.lockedUntil > new Date();
  }

  // Lifecycle hooks
  @BeforeInsert()
  @BeforeUpdate()
  async validate() {
    // Normalize email
    if (this.email) {
      this.email = this.email.toLowerCase().trim();
    }

    // Validate email format
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(this.email)) {
      throw new Error('Invalid email format');
    }
  }

  @BeforeInsert()
  async hashPasswordBeforeInsert() {
    if (this.password) {
      this.password = await bcrypt.hash(this.password, 12);
    }
  }

  // Instance methods
  async comparePassword(plainPassword: string): Promise<boolean> {
    return bcrypt.compare(plainPassword, this.password);
  }

  recordLogin() {
    this.lastLoginAt = new Date();
    this.loginAttempts = 0;
    this.lockedUntil = null;
  }

  incrementLoginAttempts() {
    this.loginAttempts++;
    // Lock account after 5 failed attempts for 15 minutes
    if (this.loginAttempts >= 5) {
      this.lockedUntil = new Date(Date.now() + 15 * 60 * 1000);
    }
  }
}
```

**Interview Tip**: Explain that entities are TypeScript classes decorated with `@Entity()` that map to database tables. Emphasize the difference between Active Record (entity extends BaseEntity) and Data Mapper (repository pattern) approaches. Mention lifecycle hooks like `@BeforeInsert()`, `@BeforeUpdate()` for validation and data transformation. Highlight production considerations: proper indexing, soft deletes, timestamp tracking, security (select: false for passwords), and validation. A strong answer demonstrates understanding of both basic concepts and production best practices.

</details>

<details>
<summary>12. What are the different decorators used to define entities?</summary>

TypeORM provides a comprehensive set of decorators to define entities, columns, relationships, indexes, and lifecycle hooks. Understanding these decorators is essential for building robust database schemas.

### Entity Decorators

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

// Basic entity decorator
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;
}

// Entity with custom table name
@Entity('users')
export class User {
  @PrimaryGeneratedColumn()
  id: number;
}

// Entity with schema (PostgreSQL)
@Entity({ name: 'users', schema: 'public' })
export class User {
  @PrimaryGeneratedColumn()
  id: number;
}

// Entity with database (for multiple database connections)
@Entity({ name: 'users', database: 'secondary_db' })
export class User {
  @PrimaryGeneratedColumn()
  id: number;
}
```

### Primary Key Decorators

```typescript
import {
  Entity,
  PrimaryColumn,
  PrimaryGeneratedColumn,
  Column,
} from 'typeorm';

// Auto-increment primary key (default)
@Entity()
export class User {
  @PrimaryGeneratedColumn() // Auto-increment integer
  id: number;
}

// UUID primary key
@Entity()
export class User {
  @PrimaryGeneratedColumn('uuid') // Auto-generated UUID
  id: string;
}

// Custom primary column (you provide the value)
@Entity()
export class User {
  @PrimaryColumn()
  id: number;

  @Column()
  name: string;
}

// Composite primary key
@Entity()
export class UserRole {
  @PrimaryColumn()
  userId: number;

  @PrimaryColumn()
  roleId: number;

  @Column()
  assignedAt: Date;
}
```

### Column Decorators

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  // Basic column
  @Column()
  name: string;

  // Column with type
  @Column('varchar', { length: 200 })
  email: string;

  // Or using object notation
  @Column({ type: 'varchar', length: 200 })
  email2: string;

  // Integer column
  @Column('int')
  age: number;

  // Nullable column
  @Column({ nullable: true })
  middleName: string | null;

  // Column with default value
  @Column({ default: true })
  isActive: boolean;

  // Unique column
  @Column({ unique: true })
  username: string;

  // Column not selected by default
  @Column({ select: false })
  password: string;

  // Column with custom name in database
  @Column({ name: 'first_name' })
  firstName: string;

  // Precision for decimal
  @Column('decimal', { precision: 10, scale: 2 })
  price: number;

  // Text column
  @Column('text')
  description: string;

  // JSON column
  @Column('json')
  metadata: any;

  // Array column (PostgreSQL)
  @Column('simple-array')
  tags: string[]; // Stored as comma-separated string

  @Column('simple-json')
  settings: { theme: string; language: string };

  // Enum column
  @Column({
    type: 'enum',
    enum: ['admin', 'user', 'moderator'],
    default: 'user',
  })
  role: string;
}
```

### Timestamp Decorators

```typescript
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  UpdateDateColumn,
  DeleteDateColumn,
  VersionColumn,
} from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Automatically set to current date on insert
  @CreateDateColumn()
  createdAt: Date;

  // Automatically updated to current date on every update
  @UpdateDateColumn()
  updatedAt: Date;

  // For soft deletes (set when entity is soft-deleted)
  @DeleteDateColumn()
  deletedAt: Date | null;

  // Optimistic locking (auto-incremented on each update)
  @VersionColumn()
  version: number;
}

// Usage of soft delete
const userRepo = AppDataSource.getRepository(User);

// Soft delete
await userRepo.softDelete(1); // Sets deletedAt timestamp

// Restore soft-deleted entity
await userRepo.restore(1); // Clears deletedAt timestamp

// Find only non-deleted entities (default)
const users = await userRepo.find();

// Include soft-deleted entities
const allUsers = await userRepo.find({ withDeleted: true });

// Find only soft-deleted entities
const deletedUsers = await userRepo
  .createQueryBuilder('user')
  .withDeleted()
  .where('user.deletedAt IS NOT NULL')
  .getMany();
```

### Generated Column Decorators

```typescript
import { Entity, PrimaryGeneratedColumn, Column, Generated } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  // UUID column with auto-generation
  @Column()
  @Generated('uuid')
  uuid: string;

  // Custom increment column
  @Column()
  @Generated('increment')
  orderNumber: number;

  @Column()
  name: string;
}
```

### Index Decorators

```typescript
import { Entity, PrimaryGeneratedColumn, Column, Index } from 'typeorm';

// Class-level indexes
@Entity()
@Index(['firstName', 'lastName']) // Composite index
@Index(['email'], { unique: true }) // Unique index
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  // Column-level index
  @Column()
  @Index() // Simple index on email column
  email: string;

  @Column()
  @Index('user_username_idx') // Named index
  username: string;

  // Unique index
  @Column()
  @Index({ unique: true })
  ssn: string;

  // Full-text index (MySQL)
  @Column()
  @Index({ fulltext: true })
  biography: string;

  // Spatial index (MySQL, PostgreSQL with PostGIS)
  @Column('geometry')
  @Index({ spatial: true })
  location: string;
}
```

### Relationship Decorators

```typescript
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  OneToOne,
  OneToMany,
  ManyToOne,
  ManyToMany,
  JoinColumn,
  JoinTable,
} from 'typeorm';

// One-to-One relationship
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToOne(() => Profile, profile => profile.user, { cascade: true })
  @JoinColumn() // Only on one side of the relationship
  profile: Profile;
}

@Entity()
export class Profile {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  bio: string;

  @OneToOne(() => User, user => user.profile)
  user: User;
}

// One-to-Many / Many-to-One relationship
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Post, post => post.user)
  posts: Post[];
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToOne(() => User, user => user.posts, { onDelete: 'CASCADE' })
  user: User;
}

// Many-to-Many relationship
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @ManyToMany(() => Role, role => role.users)
  @JoinTable() // Only on one side (owner side)
  roles: Role[];
}

@Entity()
export class Role {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @ManyToMany(() => User, user => user.roles)
  users: User[];
}
```

### Lifecycle Hook Decorators

```typescript
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  BeforeInsert,
  BeforeUpdate,
  BeforeRemove,
  AfterInsert,
  AfterUpdate,
  AfterRemove,
  AfterLoad,
} from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  email: string;

  @Column({ select: false })
  password: string;

  @Column()
  createdAt: Date;

  // Before insert hook
  @BeforeInsert()
  async hashPassword() {
    console.log('Before inserting user');
    if (this.password) {
      const bcrypt = require('bcrypt');
      this.password = await bcrypt.hash(this.password, 10);
    }
  }

  @BeforeInsert()
  setCreatedAt() {
    this.createdAt = new Date();
  }

  // Before update hook
  @BeforeUpdate()
  normalizeEmail() {
    console.log('Before updating user');
    this.email = this.email.toLowerCase();
  }

  // Before remove hook
  @BeforeRemove()
  logRemoval() {
    console.log(`User ${this.id} is about to be removed`);
  }

  // After insert hook
  @AfterInsert()
  logInsert() {
    console.log(`User ${this.id} has been inserted`);
  }

  // After update hook
  @AfterUpdate()
  logUpdate() {
    console.log(`User ${this.id} has been updated`);
  }

  // After remove hook
  @AfterRemove()
  logRemove() {
    console.log('User has been removed');
  }

  // After entity is loaded from database
  @AfterLoad()
  computeFullName() {
    // Perform computations after loading
    console.log('User entity loaded');
  }
}
```

### Unique Constraint Decorators

```typescript
import { Entity, PrimaryGeneratedColumn, Column, Unique } from 'typeorm';

// Class-level unique constraint
@Entity()
@Unique(['email']) // Single column unique
@Unique(['firstName', 'lastName']) // Composite unique
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column()
  email: string;

  // Column-level unique (simpler syntax)
  @Column({ unique: true })
  username: string;
}
```

### Check Constraint Decorators

```typescript
import { Entity, PrimaryGeneratedColumn, Column, Check } from 'typeorm';

// Class-level check constraint
@Entity()
@Check(`"age" >= 18`) // SQL check constraint
@Check(`"price" > 0`)
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column('int')
  age: number;

  @Column('decimal')
  price: number;
}
```

### Exclusion Constraint Decorator (PostgreSQL)

```typescript
import { Entity, PrimaryGeneratedColumn, Column, Exclusion } from 'typeorm';

@Entity()
@Exclusion(`USING gist (room WITH =, daterange(start_date, end_date) WITH &&)`)
export class Booking {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  room: string;

  @Column('date')
  startDate: Date;

  @Column('date')
  endDate: Date;
}
```

### Tree Entity Decorators

```typescript
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  Tree,
  TreeParent,
  TreeChildren,
} from 'typeorm';

// Closure table pattern
@Entity()
@Tree('closure-table')
export class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @TreeParent()
  parent: Category;

  @TreeChildren()
  children: Category[];
}

// Nested set pattern
@Entity()
@Tree('nested-set')
export class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @TreeParent()
  parent: Category;

  @TreeChildren()
  children: Category[];
}

// Materialized path pattern
@Entity()
@Tree('materialized-path')
export class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @TreeParent()
  parent: Category;

  @TreeChildren()
  children: Category[];
}
```

### Complete Example with Multiple Decorators

```typescript
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  UpdateDateColumn,
  DeleteDateColumn,
  Index,
  Unique,
  Check,
  BeforeInsert,
  BeforeUpdate,
  Generated,
  VersionColumn,
} from 'typeorm';

@Entity('users')
@Unique(['email'])
@Unique(['username'])
@Index(['lastName', 'firstName'])
@Check(`"age" >= 0 AND "age" <= 150`)
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  @Generated('increment')
  userNumber: number;

  @Column({ length: 50 })
  @Index()
  firstName: string;

  @Column({ length: 50 })
  lastName: string;

  @Column({ unique: true, length: 100 })
  email: string;

  @Column({ unique: true, length: 50 })
  username: string;

  @Column({ select: false })
  password: string;

  @Column({ type: 'int', default: 0 })
  age: number;

  @Column({ default: true })
  isActive: boolean;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  @DeleteDateColumn()
  deletedAt: Date | null;

  @VersionColumn()
  version: number;

  @BeforeInsert()
  @BeforeUpdate()
  normalizeEmail() {
    this.email = this.email.toLowerCase().trim();
  }

  @BeforeInsert()
  async hashPassword() {
    if (this.password) {
      const bcrypt = require('bcrypt');
      this.password = await bcrypt.hash(this.password, 10);
    }
  }
}
```

**Interview Tip**: Categorize decorators into groups: Entity decorators (`@Entity`), Primary Key decorators (`@PrimaryGeneratedColumn`, `@PrimaryColumn`), Column decorators (`@Column`, `@CreateDateColumn`, `@UpdateDateColumn`), Relationship decorators (`@OneToOne`, `@OneToMany`, `@ManyToOne`, `@ManyToMany`), Index decorators (`@Index`, `@Unique`), and Lifecycle hooks (`@BeforeInsert`, `@AfterLoad`). Mention that decorators provide metadata for TypeORM to understand how to map classes to database tables. Emphasize production use cases like soft deletes (`@DeleteDateColumn`), optimistic locking (`@VersionColumn`), and proper indexing for performance.

</details>

<details>
<summary>13. What is the difference between @PrimaryColumn and @PrimaryGeneratedColumn?</summary>

Both `@PrimaryColumn` and `@PrimaryGeneratedColumn` define primary keys, but they differ in how the primary key value is generated and managed.

### @PrimaryGeneratedColumn

`@PrimaryGeneratedColumn` creates a primary key that is **automatically generated** by the database. You don't need to provide the value manually.

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

@Entity()
export class User {
  // Auto-increment integer (default)
  @PrimaryGeneratedColumn()
  id: number; // Database generates: 1, 2, 3, 4...

  @Column()
  name: string;
}

// Usage
const user = new User();
user.name = 'John';
// No need to set user.id - it will be auto-generated

await userRepository.save(user);
console.log(user.id); // e.g., 1 (auto-generated by database)
```

### @PrimaryGeneratedColumn with UUID

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

@Entity()
export class User {
  // UUID primary key (auto-generated)
  @PrimaryGeneratedColumn('uuid')
  id: string; // e.g., "a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11"

  @Column()
  name: string;
}

// Usage
const user = new User();
user.name = 'John';
// user.id is auto-generated as UUID

await userRepository.save(user);
console.log(user.id); // e.g., "550e8400-e29b-41d4-a716-446655440000"
```

### @PrimaryGeneratedColumn with ROWID (CockroachDB)

```typescript
@Entity()
export class User {
  // CockroachDB-specific primary key
  @PrimaryGeneratedColumn('rowid')
  id: number;

  @Column()
  name: string;
}
```

### @PrimaryColumn

`@PrimaryColumn` creates a primary key where **you must provide the value manually**. The database does NOT auto-generate it.

```typescript
import { Entity, PrimaryColumn, Column } from 'typeorm';

@Entity()
export class User {
  // Manual primary key - you must set the value
  @PrimaryColumn()
  id: number;

  @Column()
  name: string;
}

// Usage
const user = new User();
user.id = 100; // MUST set manually
user.name = 'John';

await userRepository.save(user);

// ❌ This will fail if you don't set id
const user2 = new User();
user2.name = 'Jane';
// await userRepository.save(user2); // Error: id is required!
```

### @PrimaryColumn with Custom Type

```typescript
import { Entity, PrimaryColumn, Column } from 'typeorm';

@Entity()
export class User {
  // String primary key
  @PrimaryColumn('varchar', { length: 50 })
  username: string; // Use username as primary key

  @Column()
  email: string;
}

// Usage
const user = new User();
user.username = 'john_doe'; // Must provide value
user.email = 'john@example.com';

await userRepository.save(user);
```

### Composite Primary Key

You can only use `@PrimaryColumn` for composite primary keys (multiple columns as primary key).

```typescript
import { Entity, PrimaryColumn, Column } from 'typeorm';

@Entity()
export class UserRole {
  // Composite primary key (userId + roleId)
  @PrimaryColumn()
  userId: number;

  @PrimaryColumn()
  roleId: number;

  @Column()
  assignedAt: Date;
}

// Usage
const userRole = new UserRole();
userRole.userId = 1;    // Must set manually
userRole.roleId = 2;    // Must set manually
userRole.assignedAt = new Date();

await userRoleRepository.save(userRole);

// Find by composite key
const found = await userRoleRepository.findOneBy({
  userId: 1,
  roleId: 2,
});
```

### Comparison Table

| Feature | @PrimaryGeneratedColumn | @PrimaryColumn |
|---------|------------------------|----------------|
| **Value Generation** | Automatic (database) | Manual (you provide) |
| **Use Case** | Auto-increment IDs, UUIDs | Custom IDs, natural keys |
| **Composite Keys** | ❌ Not supported | ✅ Supported |
| **Types** | `number` (increment), `string` (UUID) | Any type (number, string, date, etc.) |
| **Must Set Value** | ❌ No | ✅ Yes |
| **Database Support** | All databases | All databases |

### When to Use @PrimaryGeneratedColumn

```typescript
// 1. Standard auto-increment IDs
@Entity()
export class Product {
  @PrimaryGeneratedColumn() // Use for most entities
  id: number;
  
  @Column()
  name: string;
}

// 2. Distributed systems (UUID for uniqueness across systems)
@Entity()
export class Order {
  @PrimaryGeneratedColumn('uuid') // No conflicts in distributed systems
  id: string;
  
  @Column()
  orderNumber: string;
}

// 3. When you don't want to manage ID assignment
@Entity()
export class Log {
  @PrimaryGeneratedColumn()
  id: number; // Database handles it
  
  @Column()
  message: string;
}
```

### When to Use @PrimaryColumn

```typescript
// 1. Natural primary keys (unique business identifiers)
@Entity()
export class Country {
  @PrimaryColumn('varchar', { length: 2 })
  code: string; // 'US', 'UK', 'FR' - natural key
  
  @Column()
  name: string;
}

// Usage
const usa = new Country();
usa.code = 'US';
usa.name = 'United States';

// 2. Composite keys (junction tables, many-to-many)
@Entity()
export class UserCourse {
  @PrimaryColumn()
  userId: number;
  
  @PrimaryColumn()
  courseId: number;
  
  @Column()
  enrolledAt: Date;
}

// 3. External system IDs (syncing with external systems)
@Entity()
export class ExternalUser {
  @PrimaryColumn('varchar')
  externalId: string; // ID from external API
  
  @Column()
  name: string;
  
  @Column()
  syncedAt: Date;
}

// 4. Date-based primary keys
@Entity()
export class DailyReport {
  @PrimaryColumn('date')
  reportDate: Date; // Date as primary key
  
  @Column('decimal')
  revenue: number;
}
```

### Mixed Approach (Both in Same Table)

You cannot have both `@PrimaryGeneratedColumn` and `@PrimaryColumn` on the same entity, but you can have a generated column that's NOT the primary key:

```typescript
import { Entity, PrimaryGeneratedColumn, Column, Generated } from 'typeorm';

@Entity()
export class User {
  // Auto-generated primary key
  @PrimaryGeneratedColumn()
  id: number;

  // Another auto-generated column (NOT primary)
  @Column()
  @Generated('uuid')
  uuid: string;

  @Column()
  name: string;
}

// Usage
const user = new User();
user.name = 'John';
// Both id and uuid are auto-generated

await userRepository.save(user);
console.log(user.id);   // 1
console.log(user.uuid); // "550e8400-..."
```

### Real-World Example: E-commerce System

```typescript
// Product - auto-generated ID (most entities)
@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number; // Auto-increment

  @Column({ unique: true })
  sku: string; // Also unique, but not primary key

  @Column()
  name: string;
}

// Order - UUID for distributed systems
@Entity()
export class Order {
  @PrimaryGeneratedColumn('uuid')
  id: string; // UUID for microservices

  @Column()
  orderNumber: string;

  @Column('decimal')
  total: number;
}

// ProductCategory - composite key (many-to-many junction)
@Entity()
export class ProductCategory {
  @PrimaryColumn()
  productId: number;

  @PrimaryColumn()
  categoryId: number;

  @Column()
  assignedAt: Date;
}

// Country - natural key
@Entity()
export class Country {
  @PrimaryColumn('varchar', { length: 2 })
  code: string; // 'US', 'GB', etc.

  @Column()
  name: string;
}

// Usage example
async function example() {
  const productRepo = AppDataSource.getRepository(Product);
  const orderRepo = AppDataSource.getRepository(Order);
  const productCategoryRepo = AppDataSource.getRepository(ProductCategory);
  const countryRepo = AppDataSource.getRepository(Country);

  // Product - auto ID
  const product = new Product();
  product.sku = 'LAPTOP-001';
  product.name = 'Gaming Laptop';
  await productRepo.save(product);
  console.log('Product ID:', product.id); // Auto-generated

  // Order - UUID
  const order = new Order();
  order.orderNumber = 'ORD-2024-001';
  order.total = 1299.99;
  await orderRepo.save(order);
  console.log('Order ID:', order.id); // UUID

  // ProductCategory - composite key
  const pc = new ProductCategory();
  pc.productId = product.id;
  pc.categoryId = 1;
  pc.assignedAt = new Date();
  await productCategoryRepo.save(pc);

  // Country - natural key
  const usa = new Country();
  usa.code = 'US';
  usa.name = 'United States';
  await countryRepo.save(usa);
}
```

### Performance Considerations

```typescript
// Auto-increment (best for most cases)
@Entity()
export class User {
  @PrimaryGeneratedColumn() // Fast, efficient, sequential
  id: number;
}

// UUID (better for distributed systems, slightly slower)
@Entity()
export class User {
  @PrimaryGeneratedColumn('uuid') // Unique across systems
  id: string; // Uses more storage (36 chars vs 4-8 bytes for int)
}

// Composite keys (can impact join performance)
@Entity()
export class UserRole {
  @PrimaryColumn()
  userId: number;
  
  @PrimaryColumn()
  roleId: number;
  
  // Consider adding a surrogate key for better performance
  // @PrimaryGeneratedColumn()
  // id: number;
}
```

**Interview Tip**: Clearly explain that `@PrimaryGeneratedColumn` is for auto-generated keys (database handles it), while `@PrimaryColumn` requires manual value assignment. Emphasize that `@PrimaryGeneratedColumn` is used for most entities, `@PrimaryGeneratedColumn('uuid')` for distributed systems, and `@PrimaryColumn` for natural keys and composite keys. Mention performance: auto-increment integers are fastest, UUIDs use more storage but provide global uniqueness, and composite keys can impact join performance. A good answer shows understanding of both technical differences and practical use cases.

</details>

<details>
<summary>14. How do you define a composite primary key in TypeORM?</summary>

A **composite primary key** (also called compound key) consists of multiple columns that together uniquely identify a row. In TypeORM, you define composite keys using multiple `@PrimaryColumn()` decorators.

### Basic Composite Primary Key

```typescript
import { Entity, PrimaryColumn, Column } from 'typeorm';

@Entity()
export class UserRole {
  // Composite primary key: userId + roleId
  @PrimaryColumn()
  userId: number;

  @PrimaryColumn()
  roleId: number;

  @Column()
  assignedAt: Date;

  @Column({ nullable: true })
  assignedBy: string;
}

// Usage
const userRole = new UserRole();
userRole.userId = 1;
userRole.roleId = 2;
userRole.assignedAt = new Date();

await userRoleRepository.save(userRole);

// Find by composite key
const found = await userRoleRepository.findOneBy({
  userId: 1,
  roleId: 2,
});

// Update
found.assignedBy = 'admin';
await userRoleRepository.save(found);

// Delete by composite key
await userRoleRepository.delete({
  userId: 1,
  roleId: 2,
});
```

### Many-to-Many Junction Table with Composite Key

```typescript
import { Entity, PrimaryColumn, Column, ManyToOne, JoinColumn } from 'typeorm';
import { User } from './User';
import { Course } from './Course';

@Entity('user_courses')
export class UserCourse {
  // Composite primary key
  @PrimaryColumn()
  userId: number;

  @PrimaryColumn()
  courseId: number;

  // Relationships
  @ManyToOne(() => User, user => user.userCourses)
  @JoinColumn({ name: 'userId' })
  user: User;

  @ManyToOne(() => Course, course => course.userCourses)
  @JoinColumn({ name: 'courseId' })
  course: Course;

  // Additional columns
  @Column()
  enrolledAt: Date;

  @Column({ nullable: true })
  completedAt: Date | null;

  @Column({ type: 'int', default: 0 })
  progress: number;

  @Column({ type: 'decimal', precision: 5, scale: 2, nullable: true })
  grade: number | null;
}

// User entity
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => UserCourse, userCourse => userCourse.user)
  userCourses: UserCourse[];
}

// Course entity
@Entity()
export class Course {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @OneToMany(() => UserCourse, userCourse => userCourse.course)
  userCourses: UserCourse[];
}

// Usage
const userCourse = new UserCourse();
userCourse.userId = 1;
userCourse.courseId = 5;
userCourse.enrolledAt = new Date();
userCourse.progress = 0;

await userCourseRepository.save(userCourse);
```

### Composite Key with Different Types

```typescript
import { Entity, PrimaryColumn, Column } from 'typeorm';

@Entity('product_locations')
export class ProductLocation {
  // Composite key with mixed types
  @PrimaryColumn('int')
  productId: number;

  @PrimaryColumn('varchar', { length: 10 })
  warehouseCode: string; // e.g., 'WH-001'

  @PrimaryColumn('date')
  inventoryDate: Date;

  @Column('int')
  quantity: number;

  @Column({ nullable: true })
  notes: string;
}

// Usage
const location = new ProductLocation();
location.productId = 100;
location.warehouseCode = 'WH-001';
location.inventoryDate = new Date('2024-01-15');
location.quantity = 50;

await productLocationRepository.save(location);

// Query with composite key
const found = await productLocationRepository.findOneBy({
  productId: 100,
  warehouseCode: 'WH-001',
  inventoryDate: new Date('2024-01-15'),
});
```

### Time-Series Data with Composite Key

```typescript
import { Entity, PrimaryColumn, Column, Index } from 'typeorm';

@Entity('sensor_readings')
@Index(['sensorId', 'timestamp']) // Index for faster queries
export class SensorReading {
  // Composite primary key
  @PrimaryColumn()
  sensorId: number;

  @PrimaryColumn('timestamp')
  timestamp: Date;

  @Column('decimal', { precision: 10, scale: 2 })
  temperature: number;

  @Column('decimal', { precision: 10, scale: 2 })
  humidity: number;

  @Column({ nullable: true })
  location: string;
}

// Usage
const reading = new SensorReading();
reading.sensorId = 101;
reading.timestamp = new Date();
reading.temperature = 22.5;
reading.humidity = 45.8;

await sensorReadingRepository.save(reading);

// Query range with composite key
const readings = await sensorReadingRepository
  .createQueryBuilder('reading')
  .where('reading.sensorId = :sensorId', { sensorId: 101 })
  .andWhere('reading.timestamp BETWEEN :start AND :end', {
    start: new Date('2024-01-01'),
    end: new Date('2024-01-31'),
  })
  .orderBy('reading.timestamp', 'ASC')
  .getMany();
```

### Composite Key with Relationships

```typescript
import { Entity, PrimaryColumn, Column, ManyToOne, JoinColumn } from 'typeorm';

@Entity('order_items')
export class OrderItem {
  // Composite primary key
  @PrimaryColumn()
  orderId: number;

  @PrimaryColumn()
  productId: number;

  // Relationships using the composite key parts
  @ManyToOne(() => Order, order => order.items, { onDelete: 'CASCADE' })
  @JoinColumn({ name: 'orderId' })
  order: Order;

  @ManyToOne(() => Product, product => product.orderItems)
  @JoinColumn({ name: 'productId' })
  product: Product;

  // Additional fields
  @Column('int')
  quantity: number;

  @Column('decimal', { precision: 10, scale: 2 })
  unitPrice: number;

  @Column('decimal', { precision: 10, scale: 2 })
  discount: number;

  // Computed field
  get totalPrice(): number {
    return this.quantity * this.unitPrice * (1 - this.discount / 100);
  }
}

@Entity()
export class Order {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  orderNumber: string;

  @Column()
  customerId: number;

  @OneToMany(() => OrderItem, item => item.order, { cascade: true })
  items: OrderItem[];
}

@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column('decimal')
  price: number;

  @OneToMany(() => OrderItem, item => item.product)
  orderItems: OrderItem[];
}

// Usage
const order = new Order();
order.orderNumber = 'ORD-2024-001';
order.customerId = 1;

const item1 = new OrderItem();
item1.orderId = order.id;
item1.productId = 10;
item1.quantity = 2;
item1.unitPrice = 29.99;
item1.discount = 10;

order.items = [item1];
await orderRepository.save(order); // Cascade saves items
```

### Querying with Composite Keys

```typescript
import { AppDataSource } from './data-source';
import { UserRole } from './entities/UserRole';

const userRoleRepository = AppDataSource.getRepository(UserRole);

// 1. Find by exact composite key
const userRole = await userRoleRepository.findOneBy({
  userId: 1,
  roleId: 2,
});

// 2. Find all for one part of composite key
const allUserRoles = await userRoleRepository.findBy({
  userId: 1, // All roles for user 1
});

// 3. Query builder for complex queries
const roles = await userRoleRepository
  .createQueryBuilder('ur')
  .where('ur.userId = :userId', { userId: 1 })
  .andWhere('ur.assignedAt > :date', { date: new Date('2024-01-01') })
  .getMany();

// 4. Join with related entities
const userRolesWithDetails = await userRoleRepository
  .createQueryBuilder('ur')
  .leftJoinAndSelect('ur.user', 'user')
  .leftJoinAndSelect('ur.role', 'role')
  .where('ur.userId = :userId', { userId: 1 })
  .getMany();

// 5. Delete by composite key
await userRoleRepository.delete({
  userId: 1,
  roleId: 2,
});

// 6. Delete by partial key
await userRoleRepository.delete({
  userId: 1, // Deletes all roles for user 1
});

// 7. Update by composite key
await userRoleRepository.update(
  { userId: 1, roleId: 2 },
  { assignedBy: 'admin' }
);

// 8. Check existence
const exists = await userRoleRepository.existsBy({
  userId: 1,
  roleId: 2,
});

// 9. Count
const count = await userRoleRepository.countBy({
  userId: 1,
});
```

### Composite Key vs Surrogate Key

Sometimes it's better to use a surrogate key (single auto-generated ID) instead of a composite key:

```typescript
// Option 1: Composite primary key (no surrogate key)
@Entity()
export class UserRole {
  @PrimaryColumn()
  userId: number;

  @PrimaryColumn()
  roleId: number;

  @Column()
  assignedAt: Date;
}

// Option 2: Surrogate key + unique constraint (often better)
@Entity()
@Unique(['userId', 'roleId']) // Enforce uniqueness of combination
export class UserRole {
  @PrimaryGeneratedColumn() // Surrogate key
  id: number;

  @Column()
  userId: number;

  @Column()
  roleId: number;

  @Column()
  assignedAt: Date;

  @ManyToOne(() => User)
  @JoinColumn({ name: 'userId' })
  user: User;

  @ManyToOne(() => Role)
  @JoinColumn({ name: 'roleId' })
  role: Role;
}

// Pros of surrogate key approach:
// - Simpler foreign key references
// - Easier to reference from other tables
// - Better query performance in some cases
// - Easier to work with in ORMs

// Pros of composite key approach:
// - Enforces uniqueness at database level
// - More compact (no extra ID column)
// - Natural representation of the relationship
```

### Real-World Example: Multi-Tenant Application

```typescript
import { Entity, PrimaryColumn, Column, ManyToOne, JoinColumn } from 'typeorm';

// Multi-tenant data with composite key
@Entity('tenant_users')
export class TenantUser {
  // Composite key: tenantId + userId
  @PrimaryColumn('uuid')
  tenantId: string;

  @PrimaryColumn('uuid')
  userId: string;

  @Column()
  email: string;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column({
    type: 'enum',
    enum: ['admin', 'user', 'readonly'],
    default: 'user',
  })
  role: string;

  @Column({ default: true })
  isActive: boolean;

  @Column({ type: 'timestamp' })
  createdAt: Date;

  @Column({ type: 'timestamp' })
  lastLoginAt: Date;

  // Relationships
  @ManyToOne(() => Tenant)
  @JoinColumn({ name: 'tenantId' })
  tenant: Tenant;
}

@Entity()
export class Tenant {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  name: string;

  @Column({ unique: true })
  subdomain: string;

  @OneToMany(() => TenantUser, user => user.tenant)
  users: TenantUser[];
}

// Middleware to filter by tenant
class TenantUserService {
  constructor(private repository: Repository<TenantUser>) {}

  async findUserInTenant(tenantId: string, userId: string) {
    return await this.repository.findOneBy({
      tenantId,
      userId,
    });
  }

  async getAllUsersInTenant(tenantId: string) {
    return await this.repository.findBy({ tenantId });
  }

  async createUser(tenantId: string, userData: Partial<TenantUser>) {
    const user = this.repository.create({
      ...userData,
      tenantId,
      userId: generateUUID(),
      createdAt: new Date(),
      lastLoginAt: new Date(),
    });
    return await this.repository.save(user);
  }
}
```

### Performance Considerations

```typescript
import { Entity, PrimaryColumn, Column, Index } from 'typeorm';

// Add indexes for better query performance
@Entity('audit_logs')
@Index(['tenantId', 'userId', 'timestamp']) // Composite index
@Index(['timestamp']) // Separate index for time-based queries
export class AuditLog {
  // Composite primary key
  @PrimaryColumn()
  tenantId: number;

  @PrimaryColumn()
  userId: number;

  @PrimaryColumn('timestamp')
  timestamp: Date;

  @Column()
  action: string;

  @Column('text')
  details: string;

  @Column()
  ipAddress: string;
}

// Efficient queries with composite key
const logs = await auditLogRepository
  .createQueryBuilder('log')
  .where('log.tenantId = :tenantId', { tenantId: 1 })
  .andWhere('log.userId = :userId', { userId: 100 })
  .andWhere('log.timestamp >= :startDate', { startDate: new Date('2024-01-01') })
  .orderBy('log.timestamp', 'DESC')
  .take(100)
  .getMany();
```

### Migration for Composite Primary Key

```typescript
import { MigrationInterface, QueryRunner, Table, TableIndex } from 'typeorm';

export class CreateUserRolesTable1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.createTable(
      new Table({
        name: 'user_roles',
        columns: [
          {
            name: 'userId',
            type: 'int',
            isPrimary: true, // Part of composite key
          },
          {
            name: 'roleId',
            type: 'int',
            isPrimary: true, // Part of composite key
          },
          {
            name: 'assignedAt',
            type: 'timestamp',
            default: 'CURRENT_TIMESTAMP',
          },
          {
            name: 'assignedBy',
            type: 'varchar',
            length: '100',
            isNullable: true,
          },
        ],
        foreignKeys: [
          {
            columnNames: ['userId'],
            referencedTableName: 'users',
            referencedColumnNames: ['id'],
            onDelete: 'CASCADE',
          },
          {
            columnNames: ['roleId'],
            referencedTableName: 'roles',
            referencedColumnNames: ['id'],
            onDelete: 'CASCADE',
          },
        ],
      }),
      true
    );

    // Add index for better query performance
    await queryRunner.createIndex(
      'user_roles',
      new TableIndex({
        name: 'IDX_USER_ROLES_USER_ID',
        columnNames: ['userId'],
      })
    );
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropTable('user_roles');
  }
}
```

**Interview Tip**: Explain that composite primary keys use multiple `@PrimaryColumn()` decorators and are commonly used for junction tables in many-to-many relationships. Emphasize that you cannot use `@PrimaryGeneratedColumn()` for composite keys - all parts must be `@PrimaryColumn()` with manually assigned values. Discuss when to use composite keys (natural relationships, junction tables) vs surrogate keys (simpler foreign key references, easier to work with). Mention performance: add indexes on composite keys for better query performance, especially when querying by partial keys. A strong answer demonstrates understanding of both the technical implementation and design trade-offs.

</details>

<details>
<summary>15. What is the @Column decorator and what options does it support?</summary>

The `@Column()` decorator marks a class property as a database table column. It supports extensive options for defining column behavior, constraints, and database-specific features.

### Basic Column Usage

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  // Simple column (type inferred from TypeScript)
  @Column()
  name: string; // VARCHAR(255)

  @Column()
  age: number; // INT

  @Column()
  isActive: boolean; // BOOLEAN
}
```

### Column Type Options

```typescript
@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  // Explicit type specification
  @Column('varchar', { length: 100 })
  name: string;

  // Alternative object notation
  @Column({ type: 'varchar', length: 100 })
  name2: string;

  // Integer types
  @Column('int')
  stock: number;

  @Column('bigint')
  bigNumber: number;

  @Column('smallint')
  smallNumber: number;

  // Decimal/Float types
  @Column('decimal', { precision: 10, scale: 2 })
  price: number; // e.g., 12345678.90

  @Column('float')
  weight: number;

  @Column('double')
  distance: number;

  // Text types
  @Column('text')
  description: string; // Unlimited length

  @Column('varchar', { length: 50 })
  sku: string; // Limited length

  @Column('char', { length: 2 })
  countryCode: string; // Fixed length

  // Date/Time types
  @Column('date')
  manufactureDate: Date;

  @Column('time')
  openTime: Date;

  @Column('datetime')
  createdAt: Date;

  @Column('timestamp')
  updatedAt: Date;

  // Boolean
  @Column('boolean')
  isAvailable: boolean;

  // JSON
  @Column('json')
  specifications: any;

  @Column('jsonb') // PostgreSQL optimized JSON
  metadata: any;

  // Binary
  @Column('blob')
  image: Buffer;
}
```

### Common Column Options

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  // Nullable column (allows NULL values)
  @Column({ nullable: true })
  middleName: string | null;

  // Default value
  @Column({ default: true })
  isActive: boolean;

  @Column({ default: 0 })
  loginCount: number;

  @Column({ default: () => 'CURRENT_TIMESTAMP' })
  registeredAt: Date;

  // Unique constraint
  @Column({ unique: true })
  email: string;

  // Column not selected by default (good for sensitive data)
  @Column({ select: false })
  password: string;

  // Custom column name in database
  @Column({ name: 'first_name' })
  firstName: string; // Property: firstName, DB column: first_name

  // Length specification
  @Column({ length: 100 })
  username: string;

  // Comment (appears in database schema)
  @Column({ comment: 'User email address for login' })
  userEmail: string;

  // Unsigned (positive numbers only - MySQL)
  @Column({ type: 'int', unsigned: true })
  age: number;

  // Zerofill (MySQL)
  @Column({ type: 'int', width: 5, zerofill: true })
  zipCode: number; // Stored as 00123

  // Charset and collation (MySQL)
  @Column({ type: 'varchar', length: 255, charset: 'utf8mb4', collation: 'utf8mb4_unicode_ci' })
  description: string;

  // Readonly (cannot be updated after insert)
  @Column({ update: false })
  createdById: number;

  // Enum column
  @Column({
    type: 'enum',
    enum: ['admin', 'user', 'guest'],
    default: 'user',
  })
  role: string;

  // Set column (MySQL - multiple values)
  @Column({
    type: 'set',
    enum: ['red', 'blue', 'green', 'yellow'],
    default: ['red'],
  })
  favoriteColors: string[];

  // Array column (PostgreSQL)
  @Column('text', { array: true })
  tags: string[];

  @Column('int', { array: true })
  scores: number[];
}
```

### Precision and Scale for Decimals

```typescript
@Entity()
export class FinancialTransaction {
  @PrimaryGeneratedColumn()
  id: number;

  // Decimal with precision (total digits) and scale (decimal places)
  // precision: 10 = total 10 digits
  // scale: 2 = 2 digits after decimal
  // Max value: 99999999.99
  @Column('decimal', { precision: 10, scale: 2 })
  amount: number;

  // For currencies
  @Column('decimal', { precision: 19, scale: 4 })
  price: number; // Supports up to 999,999,999,999,999.9999

  // For percentages
  @Column('decimal', { precision: 5, scale: 2 })
  interestRate: number; // e.g., 100.00 for 100%

  // Without decimal places
  @Column('decimal', { precision: 10, scale: 0 })
  wholeDollars: number;
}
```

### Width and Display Options

```typescript
@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  // Width (display width, not storage size)
  @Column({ type: 'int', width: 11 })
  stockLevel: number;

  // Unsigned (no negative values)
  @Column({ type: 'int', unsigned: true })
  viewCount: number;

  // Zerofill (pads with zeros)
  @Column({ type: 'int', width: 6, zerofill: true })
  productCode: number; // Displayed as 000123
}
```

### Array and JSON Columns

```typescript
@Entity()
export class Article {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  // Simple array (PostgreSQL)
  @Column('text', { array: true })
  tags: string[];

  @Column('int', { array: true })
  relatedIds: number[];

  // JSON column
  @Column('json')
  metadata: {
    author: string;
    publishDate: string;
    category: string;
  };

  // JSONB (PostgreSQL - better performance)
  @Column('jsonb')
  settings: any;

  // Simple array stored as comma-separated string
  @Column('simple-array')
  keywords: string[]; // Stored as: "keyword1,keyword2,keyword3"

  // Simple JSON stored as string
  @Column('simple-json')
  preferences: { theme: string; language: string }; // Stored as JSON string
}

// Usage
const article = new Article();
article.title = 'TypeORM Guide';
article.tags = ['typescript', 'orm', 'database'];
article.metadata = {
  author: 'John Doe',
  publishDate: '2024-01-15',
  category: 'Tutorial',
};
article.keywords = ['typeorm', 'guide'];

await articleRepository.save(article);
```

### Generated Columns (Database Level)

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  // Stored generated column (computed and stored)
  @Column({
    type: 'varchar',
    generatedType: 'STORED',
    asExpression: `CONCAT(first_name, ' ', last_name)`,
  })
  fullName: string;

  // Virtual generated column (computed on-the-fly)
  @Column({
    type: 'int',
    generatedType: 'VIRTUAL',
    asExpression: `YEAR(CURRENT_DATE) - birth_year`,
  })
  age: number;

  @Column('int')
  birthYear: number;
}
```

### Spatial Columns (Geographic Data)

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

@Entity()
export class Location {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Point (longitude, latitude)
  @Column({
    type: 'geometry',
    spatialFeatureType: 'Point',
    srid: 4326, // WGS 84 coordinate system
  })
  coordinates: string; // Stored as WKT: 'POINT(longitude latitude)'

  // Polygon
  @Column({
    type: 'geometry',
    spatialFeatureType: 'Polygon',
    srid: 4326,
  })
  boundary: string;

  // LineString
  @Column({
    type: 'geometry',
    spatialFeatureType: 'LineString',
    srid: 4326,
  })
  route: string;
}

// Usage
const location = new Location();
location.name = 'Central Park';
location.coordinates = 'POINT(-73.965355 40.782865)';
```

### Full Column Options Reference

```typescript
@Entity()
export class CompleteExample {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({
    // Basic options
    type: 'varchar',              // Column type
    name: 'user_name',            // Custom column name in DB
    length: 100,                  // Length for string types
    width: 100,                   // Display width
    nullable: false,              // Allow NULL values
    readonly: false,              // Cannot be updated
    select: true,                 // Include in SELECT queries by default
    insert: true,                 // Can be inserted
    update: true,                 // Can be updated
    primary: false,               // Is primary key
    unique: false,                // Unique constraint
    comment: 'Username field',    // Database comment

    // Default values
    default: 'guest',             // Static default
    // default: () => 'CURRENT_TIMESTAMP', // Function default

    // Numeric options
    precision: 10,                // Total digits for decimal
    scale: 2,                     // Decimal places
    unsigned: false,              // No negative values (MySQL)
    zerofill: false,              // Pad with zeros (MySQL)

    // String options
    charset: 'utf8mb4',           // Character set (MySQL)
    collation: 'utf8mb4_unicode_ci', // Collation (MySQL)

    // Enum/Set
    enum: ['admin', 'user'],      // Enum values
    enumName: 'user_role_enum',   // Enum type name (PostgreSQL)

    // Array (PostgreSQL)
    array: false,                 // Is array type

    // Spatial (MySQL, PostgreSQL)
    spatialFeatureType: 'Point',  // Spatial type
    srid: 4326,                   // Spatial reference ID

    // Generated columns
    generatedType: 'STORED',      // STORED or VIRTUAL
    asExpression: 'UPPER(name)',  // Generation expression

    // Transformer (custom serialization)
    transformer: {
      to: (value: string) => value.toLowerCase(),
      from: (value: string) => value.toUpperCase(),
    },
  })
  username: string;
}
```

### Column Transformers

```typescript
import { Entity, PrimaryGeneratedColumn, Column, ValueTransformer } from 'typeorm';

// Custom transformer for encryption
const encryptionTransformer: ValueTransformer = {
  // Transform when writing to database
  to: (value: string): string => {
    if (!value) return value;
    // Encrypt value (pseudo-code)
    return encrypt(value);
  },
  // Transform when reading from database
  from: (value: string): string => {
    if (!value) return value;
    // Decrypt value (pseudo-code)
    return decrypt(value);
  },
};

// Lowercase transformer
const lowercaseTransformer: ValueTransformer = {
  to: (value: string) => value?.toLowerCase(),
  from: (value: string) => value,
};

// JSON transformer
const jsonTransformer: ValueTransformer = {
  to: (value: any) => JSON.stringify(value),
  from: (value: string) => JSON.parse(value),
};

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ transformer: lowercaseTransformer })
  email: string; // Always stored as lowercase

  @Column({ transformer: encryptionTransformer, select: false })
  ssn: string; // Encrypted in database

  @Column('text', { transformer: jsonTransformer })
  preferences: { theme: string; notifications: boolean };
}

// Usage
const user = new User();
user.email = 'JOHN@EXAMPLE.COM'; // Stored as: john@example.com
user.ssn = '123-45-6789';         // Stored as: encrypted value
user.preferences = { theme: 'dark', notifications: true }; // Stored as: JSON string
```

### Real-World Production Example

```typescript
import { Entity, PrimaryGeneratedColumn, Column, Index, CreateDateColumn, UpdateDateColumn } from 'typeorm';

@Entity('users')
@Index(['email'])
@Index(['lastName', 'firstName'])
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ length: 50, nullable: false })
  firstName: string;

  @Column({ length: 50, nullable: false })
  lastName: string;

  @Column({ unique: true, length: 255 })
  email: string;

  @Column({ select: false, nullable: false })
  password: string;

  @Column({ length: 20, nullable: true })
  phoneNumber: string | null;

  @Column({ type: 'date', nullable: true })
  dateOfBirth: Date | null;

  @Column({ type: 'enum', enum: ['admin', 'user', 'moderator'], default: 'user' })
  role: string;

  @Column({ default: true })
  isActive: boolean;

  @Column({ default: false })
  isEmailVerified: boolean;

  @Column({ type: 'timestamp', nullable: true })
  emailVerifiedAt: Date | null;

  @Column({ type: 'timestamp', nullable: true })
  lastLoginAt: Date | null;

  @Column({ type: 'int', default: 0, unsigned: true })
  loginAttempts: number;

  @Column({ type: 'json', nullable: true })
  settings: {
    theme: string;
    language: string;
    notifications: boolean;
  } | null;

  @Column('simple-array', { nullable: true })
  preferredCategories: string[] | null;

  @Column({ type: 'decimal', precision: 10, scale: 2, default: 0 })
  accountBalance: number;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  @Column({ type: 'timestamp', nullable: true })
  deletedAt: Date | null;

  @Column({ update: false, nullable: true, comment: 'ID of user who created this record' })
  createdBy: number | null;
}
```

**Interview Tip**: Explain that `@Column()` is the core decorator for defining table columns and supports extensive options for type specification, constraints, defaults, and database-specific features. Highlight important options: `nullable` for NULL values, `unique` for uniqueness, `select: false` for sensitive data, `default` for default values, `length` for strings, `precision`/`scale` for decimals. Mention transformers for custom serialization/deserialization. Discuss production considerations: proper indexing, appropriate data types for storage optimization, use of `select: false` for passwords, and `update: false` for audit fields. A strong answer demonstrates both breadth of knowledge and practical production experience.

</details>

<details>
<summary>16. How do you define column types in TypeORM?</summary>

TypeORM supports a wide variety of column types that map to database-specific types. Understanding how to choose and define the right column type is crucial for data integrity, storage efficiency, and query performance.

### Type Inference from TypeScript

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  // TypeORM infers types from TypeScript types
  @Column()
  name: string; // → VARCHAR(255)

  @Column()
  age: number; // → INT

  @Column()
  isActive: boolean; // → BOOLEAN

  @Column()
  createdAt: Date; // → DATETIME
}
```

### Explicit Type Specification

```typescript
@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  // Method 1: String notation
  @Column('varchar', { length: 100 })
  name: string;

  // Method 2: Object notation
  @Column({ type: 'varchar', length: 100 })
  title: string;

  // Both methods are equivalent
}
```

### String Column Types

```typescript
@Entity()
export class Article {
  @PrimaryGeneratedColumn()
  id: number;

  // VARCHAR - variable length string (specify length)
  @Column('varchar', { length: 255 })
  title: string;

  // CHAR - fixed length string
  @Column('char', { length: 2 })
  countryCode: string; // Always 2 characters

  // TEXT - unlimited length text
  @Column('text')
  content: string;

  // LONGTEXT - very large text (MySQL)
  @Column('longtext')
  largeContent: string;

  // MEDIUMTEXT - medium text (MySQL)
  @Column('mediumtext')
  mediumContent: string;

  // TINYTEXT - small text (MySQL)
  @Column('tinytext')
  smallText: string;
}
```

### Numeric Column Types

```typescript
@Entity()
export class Statistics {
  @PrimaryGeneratedColumn()
  id: number;

  // INTEGER types
  @Column('int')
  viewCount: number; // 4 bytes, -2147483648 to 2147483647

  @Column('tinyint')
  rating: number; // 1 byte, -128 to 127 (or 0 to 255 if unsigned)

  @Column('smallint')
  likes: number; // 2 bytes, -32768 to 32767

  @Column('mediumint')
  shares: number; // 3 bytes, -8388608 to 8388607 (MySQL)

  @Column('bigint')
  bigNumber: number; // 8 bytes, very large numbers

  // Unsigned integers (positive only)
  @Column({ type: 'int', unsigned: true })
  positiveCount: number; // 0 to 4294967295

  // DECIMAL - exact numeric values (for money, prices)
  @Column('decimal', { precision: 10, scale: 2 })
  price: number; // Max: 99999999.99

  @Column('decimal', { precision: 19, scale: 4 })
  exchangeRate: number; // High precision

  // FLOAT - approximate floating point (4 bytes)
  @Column('float')
  temperature: number;

  // DOUBLE - double precision floating point (8 bytes)
  @Column('double')
  distance: number;

  // REAL (alias for DOUBLE in some databases)
  @Column('real')
  measurement: number;
}
```

### Date and Time Column Types

```typescript
@Entity()
export class Event {
  @PrimaryGeneratedColumn()
  id: number;

  // DATE - date only (YYYY-MM-DD)
  @Column('date')
  eventDate: Date; // e.g., 2024-01-15

  // TIME - time only (HH:MM:SS)
  @Column('time')
  startTime: Date; // e.g., 14:30:00

  // DATETIME - date and time
  @Column('datetime')
  scheduledAt: Date; // e.g., 2024-01-15 14:30:00

  // TIMESTAMP - timestamp with timezone awareness
  @Column('timestamp')
  createdAt: Date;

  // TIMESTAMP with default current timestamp
  @Column({
    type: 'timestamp',
    default: () => 'CURRENT_TIMESTAMP',
  })
  registeredAt: Date;

  // TIMESTAMP with auto-update on change
  @Column({
    type: 'timestamp',
    default: () => 'CURRENT_TIMESTAMP',
    onUpdate: 'CURRENT_TIMESTAMP',
  })
  updatedAt: Date;

  // YEAR (MySQL) - year only
  @Column('year')
  eventYear: number; // e.g., 2024
}
```

### Boolean Column Types

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  // BOOLEAN (stored differently across databases)
  // PostgreSQL: true boolean type
  // MySQL: TINYINT(1) - 0 or 1
  // SQLite: INTEGER - 0 or 1
  @Column('boolean')
  isActive: boolean;

  @Column({ type: 'boolean', default: false })
  isVerified: boolean;

  @Column('bool') // Alias for boolean
  isAdmin: boolean;
}
```

### Binary Column Types

```typescript
@Entity()
export class Document {
  @PrimaryGeneratedColumn()
  id: number;

  // BLOB - binary large object (MySQL)
  @Column('blob')
  thumbnail: Buffer;

  // TINYBLOB (MySQL) - up to 255 bytes
  @Column('tinyblob')
  smallImage: Buffer;

  // MEDIUMBLOB (MySQL) - up to 16MB
  @Column('mediumblob')
  mediumFile: Buffer;

  // LONGBLOB (MySQL) - up to 4GB
  @Column('longblob')
  largeFile: Buffer;

  // BYTEA (PostgreSQL) - binary data
  @Column('bytea')
  fileData: Buffer;

  // VARBINARY (MySQL, SQL Server) - variable length binary
  @Column('varbinary', { length: 255 })
  hash: Buffer;
}
```

### JSON Column Types

```typescript
@Entity()
export class Configuration {
  @PrimaryGeneratedColumn()
  id: number;

  // JSON - JSON data type
  @Column('json')
  settings: any;

  // JSONB - Binary JSON (PostgreSQL) - better performance
  @Column('jsonb')
  metadata: {
    author: string;
    tags: string[];
    published: boolean;
  };

  // Simple JSON (stored as string, not native JSON)
  @Column('simple-json')
  preferences: { theme: string; language: string };
}

// Usage
const config = new Configuration();
config.settings = { theme: 'dark', notifications: true };
config.metadata = {
  author: 'John Doe',
  tags: ['typescript', 'database'],
  published: true,
};
```

### Array Column Types (PostgreSQL)

```typescript
@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  // Text array
  @Column('text', { array: true })
  tags: string[];

  // Integer array
  @Column('int', { array: true })
  relatedPostIds: number[];

  // Simple array (stored as comma-separated string, works on all databases)
  @Column('simple-array')
  categories: string[]; // Stored as: "cat1,cat2,cat3"
}

// Usage
const post = new Post();
post.title = 'TypeORM Guide';
post.tags = ['orm', 'typescript', 'database'];
post.relatedPostIds = [1, 5, 12];
post.categories = ['tutorial', 'backend'];
```

### Enum Column Types

```typescript
// Define enum
enum UserRole {
  ADMIN = 'admin',
  USER = 'user',
  MODERATOR = 'moderator',
}

enum OrderStatus {
  PENDING = 'pending',
  PROCESSING = 'processing',
  SHIPPED = 'shipped',
  DELIVERED = 'delivered',
  CANCELLED = 'cancelled',
}

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  // ENUM column
  @Column({
    type: 'enum',
    enum: UserRole,
    default: UserRole.USER,
  })
  role: UserRole;

  // With PostgreSQL enum name
  @Column({
    type: 'enum',
    enum: OrderStatus,
    enumName: 'order_status_enum', // Custom enum type name in PostgreSQL
    default: OrderStatus.PENDING,
  })
  orderStatus: OrderStatus;

  // SET column (MySQL only) - multiple values allowed
  @Column({
    type: 'set',
    enum: ['red', 'blue', 'green', 'yellow'],
    default: ['red'],
  })
  favoriteColors: string[];
}
```

### Spatial/Geographic Column Types

```typescript
@Entity()
export class Location {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // GEOMETRY - general spatial type
  @Column('geometry')
  area: string;

  // POINT - specific point location
  @Column({
    type: 'geometry',
    spatialFeatureType: 'Point',
    srid: 4326, // WGS 84 coordinate system
  })
  coordinates: string; // 'POINT(-73.935242 40.730610)'

  // POLYGON - area boundary
  @Column({
    type: 'geometry',
    spatialFeatureType: 'Polygon',
    srid: 4326,
  })
  boundary: string;

  // LINESTRING - route/path
  @Column({
    type: 'geometry',
    spatialFeatureType: 'LineString',
    srid: 4326,
  })
  route: string;

  // GEOGRAPHY (PostgreSQL with PostGIS)
  @Column('geography')
  location: string;
}
```

### UUID Column Type

```typescript
@Entity()
export class User {
  // UUID as primary key
  @PrimaryGeneratedColumn('uuid')
  id: string; // Auto-generated UUID

  // Regular UUID column
  @Column('uuid')
  externalId: string;

  // UUID with default
  @Column({
    type: 'uuid',
    default: () => 'gen_random_uuid()', // PostgreSQL
  })
  referenceId: string;
}
```

### Database-Specific Types

```typescript
// PostgreSQL specific
@Entity()
export class PostgreSQLTypes {
  @PrimaryGeneratedColumn()
  id: number;

  // CIDR - IP network
  @Column('cidr')
  ipNetwork: string; // e.g., '192.168.1.0/24'

  // INET - IP address
  @Column('inet')
  ipAddress: string; // e.g., '192.168.1.1'

  // MACADDR - MAC address
  @Column('macaddr')
  macAddress: string; // e.g., '08:00:2b:01:02:03'

  // INTERVAL - time interval
  @Column('interval')
  duration: string; // e.g., '1 day 2 hours'

  // UUID
  @Column('uuid')
  uniqueId: string;
}

// MySQL specific
@Entity()
export class MySQLTypes {
  @PrimaryGeneratedColumn()
  id: number;

  // BIT
  @Column('bit', { length: 8 })
  flags: Buffer;

  // YEAR
  @Column('year')
  yearValue: number;

  // GEOMETRY types
  @Column('point')
  location: string;

  @Column('polygon')
  area: string;
}

// SQL Server specific
@Entity()
export class SQLServerTypes {
  @PrimaryGeneratedColumn()
  id: number;

  // UNIQUEIDENTIFIER
  @Column('uniqueidentifier')
  guid: string;

  // MONEY
  @Column('money')
  salary: number;

  // XML
  @Column('xml')
  xmlData: string;
}
```

### Complete Type Reference by Database

```typescript
// Common types across all databases
@Entity()
export class AllTypes {
  @PrimaryGeneratedColumn()
  id: number;

  // Strings
  @Column('varchar') name: string;
  @Column('text') description: string;
  @Column('char') code: string;

  // Numbers
  @Column('int') count: number;
  @Column('bigint') bigCount: number;
  @Column('decimal') price: number;
  @Column('float') rating: number;
  @Column('double') distance: number;

  // Date/Time
  @Column('date') dateOnly: Date;
  @Column('time') timeOnly: Date;
  @Column('datetime') dateTime: Date;
  @Column('timestamp') timestamp: Date;

  // Boolean
  @Column('boolean') isActive: boolean;

  // Binary
  @Column('blob') fileData: Buffer;

  // JSON
  @Column('json') metadata: any;
}
```

### Choosing the Right Type

```typescript
@Entity()
export class BestPractices {
  @PrimaryGeneratedColumn()
  id: number;

  // ✅ Use VARCHAR with appropriate length for strings
  @Column('varchar', { length: 100 })
  username: string; // Not too long, not TEXT

  // ✅ Use TEXT for unlimited text (articles, comments)
  @Column('text')
  articleContent: string;

  // ✅ Use DECIMAL for money (never FLOAT!)
  @Column('decimal', { precision: 10, scale: 2 })
  price: number; // Exact values for currency

  // ❌ Don't use FLOAT for money (precision issues)
  // @Column('float')
  // price: number; // Bad: 19.99 might become 19.989999...

  // ✅ Use INT for counts
  @Column('int', { unsigned: true })
  viewCount: number;

  // ✅ Use BIGINT for very large numbers
  @Column('bigint')
  totalRevenue: number;

  // ✅ Use TIMESTAMP for audit timestamps
  @Column('timestamp', { default: () => 'CURRENT_TIMESTAMP' })
  createdAt: Date;

  // ✅ Use BOOLEAN for flags
  @Column('boolean', { default: false })
  isVerified: boolean;

  // ✅ Use ENUM for fixed set of values
  @Column({ type: 'enum', enum: ['draft', 'published', 'archived'] })
  status: string;

  // ✅ Use JSON/JSONB for structured data that doesn't need indexing
  @Column('jsonb')
  settings: { theme: string; notifications: boolean };

  // ✅ Use UUID for distributed systems
  @Column('uuid')
  externalId: string;
}
```

### Performance Considerations

```typescript
@Entity()
export class PerformanceOptimized {
  @PrimaryGeneratedColumn()
  id: number;

  // Small types for better performance
  @Column('tinyint', { unsigned: true }) // 1 byte (0-255)
  age: number;

  @Column('smallint', { unsigned: true }) // 2 bytes (0-65535)
  quantity: number;

  // Fixed length is faster than variable length
  @Column('char', { length: 36 }) // Fixed 36 chars for UUID
  uuid: string;

  // DECIMAL vs FLOAT
  @Column('decimal', { precision: 10, scale: 2 }) // Exact, slower
  exactPrice: number;

  @Column('float') // Approximate, faster, but imprecise
  approximateWeight: number;

  // TEXT is slower for sorting/indexing
  @Column('varchar', { length: 500 }) // Better for indexed fields
  indexedField: string;

  @Column('text') // Use for large, non-indexed content
  nonIndexedContent: string;
}
```

**Interview Tip**: Explain that TypeORM supports all common SQL types and database-specific types. Emphasize choosing the right type for data integrity and performance: `DECIMAL` for money (never FLOAT), appropriate string lengths for VARCHAR, `INT` with `unsigned` for counts, `TIMESTAMP` for audit fields, `ENUM` for fixed value sets. Mention type inference from TypeScript but recommend explicit types for clarity. Discuss performance: smaller types use less storage, fixed-length is faster than variable-length, proper type selection improves query performance. Highlight database-specific types when targeting a specific database. A strong answer demonstrates understanding of both the types available and when to use each one.

</details>

<details>
<summary>17. What is the difference between nullable and default in column options?</summary>

The `nullable` and `default` column options serve different purposes in database schema design. Understanding their differences is crucial for proper data modeling and application logic.

### nullable Option

`nullable` determines whether a column can store `NULL` values. It controls the database constraint.

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  // NOT NULL constraint (nullable: false is default)
  @Column()
  firstName: string; // Database: firstName VARCHAR(255) NOT NULL

  // Allows NULL values
  @Column({ nullable: true })
  middleName: string | null; // Database: middleName VARCHAR(255) NULL

  @Column()
  lastName: string;

  // Explicitly NOT NULL
  @Column({ nullable: false })
  email: string; // Same as @Column() - NOT NULL by default
}

// Usage
const user = new User();
user.firstName = 'John';
user.lastName = 'Doe';
user.middleName = null; // ✅ OK - nullable is true

await userRepository.save(user);

// ❌ This will fail - firstName cannot be null
const user2 = new User();
user2.lastName = 'Smith';
// user2.firstName is undefined
// await userRepository.save(user2); // Error: firstName cannot be NULL
```

### default Option

`default` specifies a value to use when no value is provided during INSERT. It sets the database default value.

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Default value: true
  @Column({ default: true })
  isActive: boolean; // Database: isActive BOOLEAN DEFAULT true

  // Default value: 0
  @Column({ type: 'int', default: 0 })
  loginCount: number; // Database: loginCount INT DEFAULT 0

  // Default value: 'user'
  @Column({ default: 'user' })
  role: string; // Database: role VARCHAR(255) DEFAULT 'user'

  // Function default (current timestamp)
  @Column({ default: () => 'CURRENT_TIMESTAMP' })
  registeredAt: Date; // Database: registeredAt DATETIME DEFAULT CURRENT_TIMESTAMP
}

// Usage
const user = new User();
user.name = 'John';
// isActive, loginCount, role, registeredAt will use default values

await userRepository.save(user);
console.log(user.isActive); // true (from default)
console.log(user.loginCount); // 0 (from default)
console.log(user.role); // 'user' (from default)
```

### Key Differences

```typescript
@Entity()
export class Example {
  @PrimaryGeneratedColumn()
  id: number;

  // Case 1: nullable: true, no default
  // Can be NULL, no default value
  @Column({ nullable: true })
  field1: string | null;
  // Database: field1 VARCHAR(255) NULL
  // Behavior: If not provided, stores NULL

  // Case 2: default value, nullable: false (implicit)
  // Cannot be NULL, has default value
  @Column({ default: 'default_value' })
  field2: string;
  // Database: field2 VARCHAR(255) NOT NULL DEFAULT 'default_value'
  // Behavior: If not provided, stores 'default_value'

  // Case 3: nullable: true, with default
  // Can be NULL, but has default if not provided
  @Column({ nullable: true, default: 'default_value' })
  field3: string | null;
  // Database: field3 VARCHAR(255) NULL DEFAULT 'default_value'
  // Behavior: If not provided, stores 'default_value'
  //           Can explicitly set to NULL

  // Case 4: nullable: false, no default
  // Cannot be NULL, no default - MUST provide value
  @Column({ nullable: false })
  field4: string;
  // Database: field4 VARCHAR(255) NOT NULL
  // Behavior: MUST provide value or error
}

// Usage examples
const example = new Example();

// field1: nullable, no default
// example.field1 is undefined → stored as NULL
console.log(example.field1); // undefined → NULL in DB

// field2: not nullable, has default
// example.field2 is undefined → stored as 'default_value'
console.log(example.field2); // undefined → 'default_value' in DB

// field3: nullable, has default
// example.field3 is undefined → stored as 'default_value'
console.log(example.field3); // undefined → 'default_value' in DB
example.field3 = null; // Can explicitly set to NULL

// field4: not nullable, no default
// example.field4 is undefined → ERROR (must provide value)
example.field4 = 'required_value'; // Must set

await exampleRepository.save(example);
```

### Practical Examples

```typescript
@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Price is required (not nullable, no default)
  @Column('decimal', { precision: 10, scale: 2, nullable: false })
  price: number;
  // MUST provide price when creating product

  // Discount is optional (nullable, no default)
  @Column({ type: 'decimal', precision: 5, scale: 2, nullable: true })
  discount: number | null;
  // Can be NULL (no discount), or a specific value

  // Stock defaults to 0 (not nullable, with default)
  @Column({ type: 'int', default: 0 })
  stock: number;
  // If not provided, automatically set to 0
  // Cannot be NULL

  // Description is optional with default (nullable with default)
  @Column({ nullable: true, default: 'No description available' })
  description: string | null;
  // If not provided, uses default text
  // Can be explicitly set to NULL
}

// Usage
const product = new Product();
product.name = 'Laptop';
product.price = 999.99; // REQUIRED - nullable: false, no default
// product.discount - not set, will be NULL (nullable: true)
// product.stock - not set, will be 0 (default: 0)
// product.description - not set, will be 'No description available' (default)

await productRepository.save(product);

console.log(product.discount); // null
console.log(product.stock); // 0
console.log(product.description); // 'No description available'

// Can explicitly set null for nullable fields
const product2 = new Product();
product2.name = 'Mouse';
product2.price = 29.99;
product2.description = null; // Explicitly NULL
product2.stock = 100; // Override default

await productRepository.save(product2);
```

### Timestamp Examples

```typescript
@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  // Created at with default timestamp (not nullable)
  @Column({
    type: 'timestamp',
    default: () => 'CURRENT_TIMESTAMP',
    nullable: false,
  })
  createdAt: Date;
  // Always has a value (defaults to current time)

  // Published at is optional (nullable, no default)
  @Column({ type: 'timestamp', nullable: true })
  publishedAt: Date | null;
  // NULL until published

  // Updated at with default and auto-update
  @Column({
    type: 'timestamp',
    default: () => 'CURRENT_TIMESTAMP',
    onUpdate: 'CURRENT_TIMESTAMP',
  })
  updatedAt: Date;
  // Defaults to current time, updates automatically
}

// Usage
const post = new Post();
post.title = 'My First Post';
// createdAt will be set to current timestamp
// publishedAt will be NULL
// updatedAt will be set to current timestamp

await postRepository.save(post);

// Publish later
post.publishedAt = new Date();
await postRepository.save(post);
// updatedAt automatically updated
```

### Boolean Fields

```typescript
@Entity()
export class UserAccount {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  username: string;

  // Active status defaults to true (not nullable)
  @Column({ type: 'boolean', default: true })
  isActive: boolean;
  // New users are active by default
  // Cannot be NULL

  // Email verified is initially false (not nullable, with default)
  @Column({ type: 'boolean', default: false })
  isEmailVerified: boolean;
  // Must explicitly verify email
  // Cannot be NULL

  // Premium status is optional (nullable, no default)
  @Column({ type: 'boolean', nullable: true })
  isPremium: boolean | null;
  // NULL = not set, false = explicitly not premium, true = premium

  // Two-factor auth defaults to false
  @Column({ type: 'boolean', default: false, nullable: false })
  twoFactorEnabled: boolean;
  // Defaults to disabled, but always has a value
}
```

### Combining with TypeScript Types

```typescript
@Entity()
export class Employee {
  @PrimaryGeneratedColumn()
  id: number;

  // Required field (not nullable, no default)
  @Column()
  firstName: string; // TypeScript: string (not null)

  // Optional field (nullable)
  @Column({ nullable: true })
  middleName: string | null; // TypeScript: string | null

  // Field with default (not nullable)
  @Column({ default: 'full-time' })
  employmentType: string; // TypeScript: string (will have value)

  // Optional with default (nullable)
  @Column({ nullable: true, default: 'USD' })
  currency: string | null; // TypeScript: string | null (but defaults to 'USD')

  // Required number (not nullable, no default)
  @Column('decimal', { precision: 10, scale: 2 })
  salary: number; // Must provide

  // Optional number (nullable)
  @Column({ type: 'decimal', precision: 10, scale: 2, nullable: true })
  bonus: number | null;

  // Number with default
  @Column({ type: 'int', default: 0 })
  yearsOfService: number; // Defaults to 0
}
```

### Real-World Best Practices

```typescript
@Entity()
export class Order {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  // Required fields (not nullable, no default) - business critical
  @Column()
  customerId: number;

  @Column('decimal', { precision: 10, scale: 2 })
  totalAmount: number;

  // Status with default (not nullable) - always has a state
  @Column({
    type: 'enum',
    enum: ['pending', 'processing', 'completed', 'cancelled'],
    default: 'pending',
  })
  status: string;

  // Optional fields (nullable) - may not be set
  @Column({ nullable: true })
  trackingNumber: string | null;

  @Column({ type: 'timestamp', nullable: true })
  shippedAt: Date | null;

  @Column({ type: 'text', nullable: true })
  customerNotes: string | null;

  // Audit fields with defaults (not nullable)
  @Column({ type: 'timestamp', default: () => 'CURRENT_TIMESTAMP' })
  createdAt: Date;

  @Column({
    type: 'timestamp',
    default: () => 'CURRENT_TIMESTAMP',
    onUpdate: 'CURRENT_TIMESTAMP',
  })
  updatedAt: Date;

  // Counter with default (not nullable)
  @Column({ type: 'int', default: 0 })
  attemptCount: number;

  // Optional with meaningful default
  @Column({ nullable: true, default: 'USD' })
  currency: string | null;
}
```

### Validation and Error Handling

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ nullable: false })
  email: string;

  @Column({ nullable: true })
  phoneNumber: string | null;

  @Column({ default: true })
  isActive: boolean;
}

// Service with validation
class UserService {
  async createUser(data: Partial<User>): Promise<User> {
    const userRepository = AppDataSource.getRepository(User);

    // Validate required fields (not nullable, no default)
    if (!data.email) {
      throw new Error('Email is required');
    }

    const user = userRepository.create({
      email: data.email,
      phoneNumber: data.phoneNumber || null, // Explicitly set NULL if not provided
      // isActive will use default value (true)
    });

    try {
      return await userRepository.save(user);
    } catch (error) {
      if (error.code === '23502') { // NOT NULL violation
        throw new Error('Required field is missing');
      }
      throw error;
    }
  }
}
```

### Migration Examples

```typescript
import { MigrationInterface, QueryRunner, TableColumn } from 'typeorm';

export class UpdateUserColumns1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    // Add nullable column
    await queryRunner.addColumn(
      'users',
      new TableColumn({
        name: 'middle_name',
        type: 'varchar',
        length: '50',
        isNullable: true, // nullable: true
      })
    );

    // Add column with default
    await queryRunner.addColumn(
      'users',
      new TableColumn({
        name: 'is_active',
        type: 'boolean',
        default: true, // default: true
        isNullable: false, // not nullable
      })
    );

    // Add nullable column with default
    await queryRunner.addColumn(
      'users',
      new TableColumn({
        name: 'preferred_language',
        type: 'varchar',
        length: '10',
        default: "'en'", // default: 'en'
        isNullable: true, // nullable: true
      })
    );
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropColumn('users', 'middle_name');
    await queryRunner.dropColumn('users', 'is_active');
    await queryRunner.dropColumn('users', 'preferred_language');
  }
}
```

### Summary Table

| Configuration | nullable | default | Database Constraint | Behavior When Not Set | Use Case |
|--------------|----------|---------|---------------------|----------------------|----------|
| `{}` | false (implicit) | none | NOT NULL | **Error** - must provide | Required fields |
| `{ nullable: true }` | true | none | NULL | Stores **NULL** | Optional fields |
| `{ default: X }` | false (implicit) | X | NOT NULL DEFAULT X | Stores **X** | Fields with sensible defaults |
| `{ nullable: true, default: X }` | true | X | NULL DEFAULT X | Stores **X**, can set NULL | Optional with default |

**Interview Tip**: Clearly explain that `nullable` controls whether a column accepts NULL values (database constraint), while `default` provides a value when none is specified (database default). Emphasize the four combinations: required fields (no nullable, no default), optional fields (nullable: true), fields with defaults (default value), and optional with defaults (both). Highlight TypeScript type alignment: use `| null` for nullable fields. Mention that `nullable: false` is the default behavior. Discuss real-world scenarios: required business data, optional metadata, status fields with defaults, and audit fields with timestamp defaults. A strong answer demonstrates understanding of both database constraints and practical application design.

</details>

<details>
<summary>18. How do you create enum columns in TypeORM?</summary>

Enum columns in TypeORM restrict column values to a predefined set of options. They provide type safety, data validation, and clearer code semantics compared to plain strings or numbers.

### Basic Enum Column

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

// Define TypeScript enum
export enum UserRole {
  ADMIN = 'admin',
  USER = 'user',
  MODERATOR = 'moderator',
}

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Enum column
  @Column({
    type: 'enum',
    enum: UserRole,
    default: UserRole.USER,
  })
  role: UserRole;
}

// Usage
const user = new User();
user.name = 'John';
user.role = UserRole.ADMIN; // Type-safe

await userRepository.save(user);

// Query with enum
const admins = await userRepository.findBy({
  role: UserRole.ADMIN,
});
```

### String vs Numeric Enums

```typescript
// String enum (recommended for database)
export enum OrderStatus {
  PENDING = 'pending',
  PROCESSING = 'processing',
  SHIPPED = 'shipped',
  DELIVERED = 'delivered',
  CANCELLED = 'cancelled',
}

// Numeric enum (not recommended for database)
export enum Priority {
  LOW = 1,
  MEDIUM = 2,
  HIGH = 3,
  URGENT = 4,
}

@Entity()
export class Order {
  @PrimaryGeneratedColumn()
  id: number;

  // String enum (readable in database)
  @Column({
    type: 'enum',
    enum: OrderStatus,
    default: OrderStatus.PENDING,
  })
  status: OrderStatus;
  // Database stores: 'pending', 'processing', etc.

  // Numeric enum (stores numbers)
  @Column({
    type: 'enum',
    enum: Priority,
    default: Priority.MEDIUM,
  })
  priority: Priority;
  // Database stores: 1, 2, 3, 4
}
```

### PostgreSQL Named Enum Type

```typescript
export enum UserRole {
  ADMIN = 'admin',
  USER = 'user',
  MODERATOR = 'moderator',
}

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // PostgreSQL: Creates a named enum type
  @Column({
    type: 'enum',
    enum: UserRole,
    enumName: 'user_role_enum', // Custom enum type name in PostgreSQL
    default: UserRole.USER,
  })
  role: UserRole;
}

// Generated SQL (PostgreSQL):
// CREATE TYPE user_role_enum AS ENUM ('admin', 'user', 'moderator');
// CREATE TABLE users (
//   id SERIAL PRIMARY KEY,
//   name VARCHAR NOT NULL,
//   role user_role_enum DEFAULT 'user'
// );
```

### Multiple Enum Columns

```typescript
export enum UserRole {
  ADMIN = 'admin',
  USER = 'user',
  GUEST = 'guest',
}

export enum AccountStatus {
  ACTIVE = 'active',
  INACTIVE = 'inactive',
  SUSPENDED = 'suspended',
  DELETED = 'deleted',
}

export enum SubscriptionTier {
  FREE = 'free',
  BASIC = 'basic',
  PREMIUM = 'premium',
  ENTERPRISE = 'enterprise',
}

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  email: string;

  @Column({
    type: 'enum',
    enum: UserRole,
    default: UserRole.USER,
  })
  role: UserRole;

  @Column({
    type: 'enum',
    enum: AccountStatus,
    default: AccountStatus.ACTIVE,
  })
  status: AccountStatus;

  @Column({
    type: 'enum',
    enum: SubscriptionTier,
    default: SubscriptionTier.FREE,
  })
  tier: SubscriptionTier;
}

// Usage with type safety
const user = new User();
user.email = 'john@example.com';
user.role = UserRole.ADMIN;
user.status = AccountStatus.ACTIVE;
user.tier = SubscriptionTier.PREMIUM;
```

### Enum Column with Nullable

```typescript
export enum PaymentMethod {
  CREDIT_CARD = 'credit_card',
  DEBIT_CARD = 'debit_card',
  PAYPAL = 'paypal',
  BANK_TRANSFER = 'bank_transfer',
}

@Entity()
export class Order {
  @PrimaryGeneratedColumn()
  id: number;

  @Column('decimal', { precision: 10, scale: 2 })
  total: number;

  // Optional enum (nullable)
  @Column({
    type: 'enum',
    enum: PaymentMethod,
    nullable: true,
  })
  paymentMethod: PaymentMethod | null;
  // NULL until payment method is selected
}

// Usage
const order = new Order();
order.total = 99.99;
order.paymentMethod = null; // Not yet selected

await orderRepository.save(order);

// Later, set payment method
order.paymentMethod = PaymentMethod.CREDIT_CARD;
await orderRepository.save(order);
```

### Querying with Enums

```typescript
import { In, Not } from 'typeorm';

export enum UserRole {
  ADMIN = 'admin',
  USER = 'user',
  MODERATOR = 'moderator',
}

// Find by enum value
const admins = await userRepository.findBy({
  role: UserRole.ADMIN,
});

// Find with multiple enum values
const privilegedUsers = await userRepository.findBy({
  role: In([UserRole.ADMIN, UserRole.MODERATOR]),
});

// Exclude enum value
const nonAdmins = await userRepository.findBy({
  role: Not(UserRole.ADMIN),
});

// Query builder
const activeAdmins = await userRepository
  .createQueryBuilder('user')
  .where('user.role = :role', { role: UserRole.ADMIN })
  .andWhere('user.status = :status', { status: AccountStatus.ACTIVE })
  .getMany();

// Raw query with enum
const results = await userRepository.query(
  `SELECT * FROM users WHERE role = $1`,
  [UserRole.ADMIN]
);
```

### MySQL SET Column (Multiple Values)

```typescript
// SET allows multiple enum values (MySQL only)
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // SET column - can have multiple values
  @Column({
    type: 'set',
    enum: ['notifications', 'newsletter', 'promotions', 'updates'],
    default: ['notifications'],
  })
  emailPreferences: string[];
  // Can be: ['notifications'], ['notifications', 'newsletter'], etc.
}

// Usage
const user = new User();
user.name = 'John';
user.emailPreferences = ['notifications', 'newsletter', 'promotions'];

await userRepository.save(user);

// Query users who want notifications
const users = await userRepository
  .createQueryBuilder('user')
  .where("FIND_IN_SET('notifications', user.emailPreferences) > 0")
  .getMany();
```

### Enum Column with Validation

```typescript
import { Entity, PrimaryGeneratedColumn, Column, BeforeInsert, BeforeUpdate } from 'typeorm';

export enum UserRole {
  ADMIN = 'admin',
  USER = 'user',
  MODERATOR = 'moderator',
}

export enum AccountStatus {
  ACTIVE = 'active',
  INACTIVE = 'inactive',
  SUSPENDED = 'suspended',
}

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column({
    type: 'enum',
    enum: UserRole,
    default: UserRole.USER,
  })
  role: UserRole;

  @Column({
    type: 'enum',
    enum: AccountStatus,
    default: AccountStatus.ACTIVE,
  })
  status: AccountStatus;

  // Validation hook
  @BeforeInsert()
  @BeforeUpdate()
  validateEnums() {
    // Ensure role is valid
    if (!Object.values(UserRole).includes(this.role)) {
      throw new Error(`Invalid role: ${this.role}`);
    }

    // Business logic validation
    if (this.role === UserRole.ADMIN && this.status === AccountStatus.SUSPENDED) {
      throw new Error('Admin accounts cannot be suspended');
    }
  }
}
```

### Enum Utilities and Helpers

```typescript
export enum OrderStatus {
  PENDING = 'pending',
  PROCESSING = 'processing',
  SHIPPED = 'shipped',
  DELIVERED = 'delivered',
  CANCELLED = 'cancelled',
  REFUNDED = 'refunded',
}

// Helper functions
export class OrderStatusHelper {
  // Get all enum values
  static getAllStatuses(): OrderStatus[] {
    return Object.values(OrderStatus);
  }

  // Check if status is valid
  static isValidStatus(status: string): boolean {
    return Object.values(OrderStatus).includes(status as OrderStatus);
  }

  // Get display name
  static getDisplayName(status: OrderStatus): string {
    const names = {
      [OrderStatus.PENDING]: 'Pending',
      [OrderStatus.PROCESSING]: 'Processing',
      [OrderStatus.SHIPPED]: 'Shipped',
      [OrderStatus.DELIVERED]: 'Delivered',
      [OrderStatus.CANCELLED]: 'Cancelled',
      [OrderStatus.REFUNDED]: 'Refunded',
    };
    return names[status];
  }

  // Check if status can transition
  static canTransitionTo(from: OrderStatus, to: OrderStatus): boolean {
    const validTransitions = {
      [OrderStatus.PENDING]: [OrderStatus.PROCESSING, OrderStatus.CANCELLED],
      [OrderStatus.PROCESSING]: [OrderStatus.SHIPPED, OrderStatus.CANCELLED],
      [OrderStatus.SHIPPED]: [OrderStatus.DELIVERED],
      [OrderStatus.DELIVERED]: [OrderStatus.REFUNDED],
      [OrderStatus.CANCELLED]: [],
      [OrderStatus.REFUNDED]: [],
    };
    return validTransitions[from].includes(to);
  }

  // Get active statuses (not terminal)
  static getActiveStatuses(): OrderStatus[] {
    return [
      OrderStatus.PENDING,
      OrderStatus.PROCESSING,
      OrderStatus.SHIPPED,
    ];
  }

  // Get terminal statuses (final states)
  static getTerminalStatuses(): OrderStatus[] {
    return [
      OrderStatus.DELIVERED,
      OrderStatus.CANCELLED,
      OrderStatus.REFUNDED,
    ];
  }
}

// Usage in service
class OrderService {
  async updateOrderStatus(orderId: number, newStatus: OrderStatus) {
    const order = await orderRepository.findOneBy({ id: orderId });
    
    if (!order) {
      throw new Error('Order not found');
    }

    // Validate transition
    if (!OrderStatusHelper.canTransitionTo(order.status, newStatus)) {
      throw new Error(
        `Cannot transition from ${order.status} to ${newStatus}`
      );
    }

    order.status = newStatus;
    return await orderRepository.save(order);
  }
}
```

### Enum with Descriptions

```typescript
export enum TaskPriority {
  LOW = 'low',
  MEDIUM = 'medium',
  HIGH = 'high',
  URGENT = 'urgent',
}

// Enum metadata
export const TaskPriorityMetadata = {
  [TaskPriority.LOW]: {
    label: 'Low Priority',
    color: 'green',
    level: 1,
    description: 'Can be done when time permits',
  },
  [TaskPriority.MEDIUM]: {
    label: 'Medium Priority',
    color: 'yellow',
    level: 2,
    description: 'Should be completed soon',
  },
  [TaskPriority.HIGH]: {
    label: 'High Priority',
    color: 'orange',
    level: 3,
    description: 'Needs attention soon',
  },
  [TaskPriority.URGENT]: {
    label: 'Urgent',
    color: 'red',
    level: 4,
    description: 'Requires immediate attention',
  },
};

@Entity()
export class Task {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column({
    type: 'enum',
    enum: TaskPriority,
    default: TaskPriority.MEDIUM,
  })
  priority: TaskPriority;

  // Get priority metadata
  getPriorityInfo() {
    return TaskPriorityMetadata[this.priority];
  }
}

// Usage
const task = await taskRepository.findOneBy({ id: 1 });
const info = task.getPriorityInfo();
console.log(info.label); // 'High Priority'
console.log(info.color); // 'orange'
console.log(info.level); // 3
```

### Migration for Enum Columns

```typescript
import { MigrationInterface, QueryRunner, TableColumn } from 'typeorm';

export class AddUserRoleColumn1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    // PostgreSQL: Create enum type first
    await queryRunner.query(`
      CREATE TYPE user_role_enum AS ENUM ('admin', 'user', 'moderator')
    `);

    // Add column using enum type
    await queryRunner.addColumn(
      'users',
      new TableColumn({
        name: 'role',
        type: 'enum',
        enum: ['admin', 'user', 'moderator'],
        default: "'user'",
        isNullable: false,
      })
    );
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropColumn('users', 'role');
    await queryRunner.query(`DROP TYPE user_role_enum`);
  }
}

// Adding a new enum value (PostgreSQL)
export class AddModeratorRole1234567891 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    // PostgreSQL: Add new enum value
    await queryRunner.query(`
      ALTER TYPE user_role_enum ADD VALUE 'moderator'
    `);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    // Note: PostgreSQL doesn't support removing enum values easily
    // You would need to recreate the enum type
  }
}
```

### Real-World Example: E-commerce Order

```typescript
export enum OrderStatus {
  DRAFT = 'draft',
  PENDING_PAYMENT = 'pending_payment',
  PAYMENT_CONFIRMED = 'payment_confirmed',
  PROCESSING = 'processing',
  READY_TO_SHIP = 'ready_to_ship',
  SHIPPED = 'shipped',
  IN_TRANSIT = 'in_transit',
  OUT_FOR_DELIVERY = 'out_for_delivery',
  DELIVERED = 'delivered',
  CANCELLED = 'cancelled',
  REFUND_REQUESTED = 'refund_requested',
  REFUNDED = 'refunded',
}

export enum PaymentStatus {
  PENDING = 'pending',
  AUTHORIZED = 'authorized',
  CAPTURED = 'captured',
  FAILED = 'failed',
  REFUNDED = 'refunded',
}

export enum ShippingMethod {
  STANDARD = 'standard',
  EXPRESS = 'express',
  OVERNIGHT = 'overnight',
  PICKUP = 'pickup',
}

@Entity()
export class Order {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  orderNumber: string;

  @Column()
  customerId: number;

  @Column({
    type: 'enum',
    enum: OrderStatus,
    default: OrderStatus.DRAFT,
  })
  status: OrderStatus;

  @Column({
    type: 'enum',
    enum: PaymentStatus,
    default: PaymentStatus.PENDING,
  })
  paymentStatus: PaymentStatus;

  @Column({
    type: 'enum',
    enum: ShippingMethod,
    default: ShippingMethod.STANDARD,
  })
  shippingMethod: ShippingMethod;

  @Column('decimal', { precision: 10, scale: 2 })
  total: number;

  @Column({ type: 'timestamp', default: () => 'CURRENT_TIMESTAMP' })
  createdAt: Date;

  // Business logic methods
  canBeCancelled(): boolean {
    const cancellableStatuses = [
      OrderStatus.DRAFT,
      OrderStatus.PENDING_PAYMENT,
      OrderStatus.PAYMENT_CONFIRMED,
    ];
    return cancellableStatuses.includes(this.status);
  }

  isInFinalState(): boolean {
    const finalStatuses = [
      OrderStatus.DELIVERED,
      OrderStatus.CANCELLED,
      OrderStatus.REFUNDED,
    ];
    return finalStatuses.includes(this.status);
  }
}
```

**Interview Tip**: Explain that enum columns provide type safety and constrain values to a predefined set. Emphasize using string enums over numeric enums for database readability. Mention PostgreSQL's named enum types with `enumName` option. Discuss practical uses: user roles, order statuses, payment methods, priority levels. Highlight benefits: prevents invalid data, self-documenting code, easier queries, better IDE support. Mention enum utilities for validation and state transitions. For production, discuss migration strategies for adding enum values and handling enum changes. A strong answer demonstrates both technical implementation and practical application design patterns.

</details>

<details>
<summary>19. What is @CreateDateColumn and @UpdateDateColumn?</summary>

`@CreateDateColumn` and `@UpdateDateColumn` are special decorators that automatically manage timestamp columns for tracking entity creation and modification times. They're essential for audit trails and data tracking.

### Basic Usage

```typescript
import { Entity, PrimaryGeneratedColumn, Column, CreateDateColumn, UpdateDateColumn } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  email: string;

  // Automatically set to current timestamp on insert
  @CreateDateColumn()
  createdAt: Date;

  // Automatically updated to current timestamp on every update
  @UpdateDateColumn()
  updatedAt: Date;
}

// Usage
const user = new User();
user.name = 'John';
user.email = 'john@example.com';
// No need to set createdAt or updatedAt

await userRepository.save(user);
console.log(user.createdAt); // e.g., 2024-01-15 10:30:00
console.log(user.updatedAt); // e.g., 2024-01-15 10:30:00 (same as createdAt initially)

// Update the user
user.name = 'John Doe';
await userRepository.save(user);
console.log(user.createdAt); // Still 2024-01-15 10:30:00 (unchanged)
console.log(user.updatedAt); // e.g., 2024-01-15 14:45:00 (updated automatically)
```

### How They Work

```typescript
@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // @CreateDateColumn behavior:
  // - Set once during INSERT
  // - Never changed on UPDATE
  // - Type: timestamp/datetime
  // - Cannot be manually set (ignored if provided)
  @CreateDateColumn()
  createdAt: Date;

  // @UpdateDateColumn behavior:
  // - Set during INSERT (same as createdAt initially)
  // - Automatically updated on every UPDATE
  // - Type: timestamp/datetime
  // - Cannot be manually set (ignored if provided)
  @UpdateDateColumn()
  updatedAt: Date;
}

// Generated SQL (PostgreSQL):
// CREATE TABLE product (
//   id SERIAL PRIMARY KEY,
//   name VARCHAR NOT NULL,
//   createdAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
//   updatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP
// );

// On UPDATE, TypeORM automatically sets updatedAt to current timestamp
```

### With Custom Options

```typescript
@Entity()
export class Article {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  // Custom column name
  @CreateDateColumn({ name: 'created_at' })
  createdAt: Date;
  // Database column name: created_at

  @UpdateDateColumn({ name: 'updated_at' })
  updatedAt: Date;
  // Database column name: updated_at

  // With type specification
  @CreateDateColumn({ type: 'timestamp' })
  createdAtTimestamp: Date;

  @CreateDateColumn({ type: 'datetime' })
  createdAtDatetime: Date;

  // With timezone (PostgreSQL)
  @CreateDateColumn({ type: 'timestamp with time zone' })
  createdAtWithTz: Date;

  // With precision (MySQL)
  @CreateDateColumn({ type: 'datetime', precision: 6 })
  createdAtPrecise: Date; // Microsecond precision
}
```

### Complete Audit Trail

```typescript
@Entity()
export class AuditedEntity {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  // When created
  @CreateDateColumn()
  createdAt: Date;

  // When last updated
  @UpdateDateColumn()
  updatedAt: Date;

  // Who created (set manually)
  @Column({ nullable: true })
  createdBy: number;

  // Who last updated (set manually)
  @Column({ nullable: true })
  updatedBy: number;
}

// Usage with audit info
class AuditService {
  async createEntity(userId: number, data: Partial<AuditedEntity>) {
    const repository = AppDataSource.getRepository(AuditedEntity);
    
    const entity = repository.create({
      ...data,
      createdBy: userId,
      // createdAt set automatically
      // updatedAt set automatically
    });

    return await repository.save(entity);
  }

  async updateEntity(userId: number, id: number, data: Partial<AuditedEntity>) {
    const repository = AppDataSource.getRepository(AuditedEntity);
    const entity = await repository.findOneBy({ id });

    if (!entity) {
      throw new Error('Entity not found');
    }

    Object.assign(entity, data);
    entity.updatedBy = userId;
    // updatedAt set automatically

    return await repository.save(entity);
  }
}
```

### Soft Delete with Timestamps

```typescript
import { Entity, PrimaryGeneratedColumn, Column, CreateDateColumn, UpdateDateColumn, DeleteDateColumn } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Created timestamp
  @CreateDateColumn()
  createdAt: Date;

  // Updated timestamp
  @UpdateDateColumn()
  updatedAt: Date;

  // Soft delete timestamp (null = not deleted)
  @DeleteDateColumn()
  deletedAt: Date | null;
}

// Usage
const user = new User();
user.name = 'John';
await userRepository.save(user);
// createdAt: 2024-01-15 10:00:00
// updatedAt: 2024-01-15 10:00:00
// deletedAt: null

// Update user
user.name = 'John Doe';
await userRepository.save(user);
// createdAt: 2024-01-15 10:00:00 (unchanged)
// updatedAt: 2024-01-15 11:00:00 (updated)
// deletedAt: null

// Soft delete
await userRepository.softDelete(user.id);
// createdAt: 2024-01-15 10:00:00 (unchanged)
// updatedAt: 2024-01-15 11:00:00 (unchanged)
// deletedAt: 2024-01-15 12:00:00 (set to current time)

// Restore
await userRepository.restore(user.id);
// deletedAt: null (cleared)
```

### Querying with Timestamps

```typescript
import { LessThan, MoreThan, Between } from 'typeorm';

// Find recently created entities
const recentUsers = await userRepository.find({
  where: {
    createdAt: MoreThan(new Date(Date.now() - 24 * 60 * 60 * 1000)), // Last 24 hours
  },
});

// Find recently updated entities
const recentlyUpdated = await userRepository.find({
  where: {
    updatedAt: MoreThan(new Date(Date.now() - 1 * 60 * 60 * 1000)), // Last hour
  },
  order: {
    updatedAt: 'DESC',
  },
});

// Find by date range
const usersInRange = await userRepository.find({
  where: {
    createdAt: Between(
      new Date('2024-01-01'),
      new Date('2024-01-31')
    ),
  },
});

// Query builder
const oldUsers = await userRepository
  .createQueryBuilder('user')
  .where('user.createdAt < :date', {
    date: new Date('2023-01-01'),
  })
  .andWhere('user.updatedAt < :updateDate', {
    updateDate: new Date('2024-01-01'),
  })
  .getMany();

// Find stale records (not updated in 30 days)
const staleRecords = await userRepository
  .createQueryBuilder('user')
  .where('user.updatedAt < NOW() - INTERVAL 30 DAY')
  .getMany();
```

### Timezone Considerations

```typescript
@Entity()
export class Event {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  // Without timezone (local time)
  @CreateDateColumn({ type: 'datetime' })
  createdAt: Date;

  // With timezone (PostgreSQL, recommended for distributed systems)
  @CreateDateColumn({ type: 'timestamp with time zone' })
  createdAtUtc: Date;

  @UpdateDateColumn({ type: 'timestamp with time zone' })
  updatedAtUtc: Date;
}

// Usage: Always stored in UTC, converted to local time when retrieved
const event = new Event();
event.title = 'Conference';
await eventRepository.save(event);

// Timestamps are in UTC
console.log(event.createdAtUtc.toISOString()); // 2024-01-15T10:30:00.000Z
console.log(event.createdAtUtc.toLocaleString('en-US', { timeZone: 'America/New_York' }));
```

### Custom Update Behavior

```typescript
@Entity()
export class Document {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column({ type: 'text' })
  content: string;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  // Last published timestamp (manual)
  @Column({ type: 'timestamp', nullable: true })
  publishedAt: Date | null;

  // Prevent auto-update in specific cases
  @Column({ default: false, select: false })
  skipAutoUpdate: boolean;
}

// Service with custom update logic
class DocumentService {
  async updateContent(id: number, content: string) {
    const doc = await documentRepository.findOneBy({ id });
    doc.content = content;
    // updatedAt will be automatically updated
    await documentRepository.save(doc);
  }

  async publish(id: number) {
    const doc = await documentRepository.findOneBy({ id });
    doc.publishedAt = new Date();
    // updatedAt will also be updated
    await documentRepository.save(doc);
  }

  // Update without changing updatedAt
  async updateViewCount(id: number, views: number) {
    // Use query builder to bypass entity update
    await documentRepository
      .createQueryBuilder()
      .update(Document)
      .set({ viewCount: views })
      .where('id = :id', { id })
      .execute();
    // updatedAt is NOT changed when using query builder
  }
}
```

### Real-World Production Example

```typescript
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  UpdateDateColumn,
  DeleteDateColumn,
  VersionColumn,
  Index,
} from 'typeorm';

@Entity('products')
@Index(['createdAt'])
@Index(['updatedAt'])
export class Product {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ length: 200 })
  name: string;

  @Column('text')
  description: string;

  @Column('decimal', { precision: 10, scale: 2 })
  price: number;

  @Column({ type: 'int', default: 0 })
  stock: number;

  // Audit timestamps
  @CreateDateColumn({ type: 'timestamp' })
  createdAt: Date;

  @UpdateDateColumn({ type: 'timestamp' })
  updatedAt: Date;

  @DeleteDateColumn({ type: 'timestamp', nullable: true })
  deletedAt: Date | null;

  // Optimistic locking
  @VersionColumn()
  version: number;

  // Audit user IDs
  @Column({ nullable: true })
  createdBy: number;

  @Column({ nullable: true })
  updatedBy: number;

  @Column({ nullable: true })
  deletedBy: number;

  // Published timestamp (manual control)
  @Column({ type: 'timestamp', nullable: true })
  publishedAt: Date | null;

  // Last price change tracking
  @Column({ type: 'timestamp', nullable: true })
  priceChangedAt: Date | null;

  @Column({ type: 'decimal', precision: 10, scale: 2, nullable: true })
  previousPrice: number | null;

  // Helper methods
  isPublished(): boolean {
    return this.publishedAt !== null && this.publishedAt <= new Date();
  }

  isDeleted(): boolean {
    return this.deletedAt !== null;
  }

  getAge(): number {
    return Date.now() - this.createdAt.getTime();
  }

  getDaysSinceUpdate(): number {
    const diff = Date.now() - this.updatedAt.getTime();
    return Math.floor(diff / (1000 * 60 * 60 * 24));
  }
}

// Service with comprehensive audit tracking
class ProductService {
  async create(userId: number, data: Partial<Product>): Promise<Product> {
    const product = productRepository.create({
      ...data,
      createdBy: userId,
      // createdAt and updatedAt set automatically
    });
    return await productRepository.save(product);
  }

  async update(userId: number, id: string, data: Partial<Product>): Promise<Product> {
    const product = await productRepository.findOneBy({ id });
    
    if (!product) {
      throw new Error('Product not found');
    }

    // Track price changes
    if (data.price && data.price !== product.price) {
      product.previousPrice = product.price;
      product.priceChangedAt = new Date();
    }

    Object.assign(product, data);
    product.updatedBy = userId;
    // updatedAt set automatically

    return await productRepository.save(product);
  }

  async publish(userId: number, id: string): Promise<Product> {
    const product = await productRepository.findOneBy({ id });
    
    if (!product) {
      throw new Error('Product not found');
    }

    product.publishedAt = new Date();
    product.updatedBy = userId;
    // updatedAt set automatically

    return await productRepository.save(product);
  }

  async softDelete(userId: number, id: string): Promise<void> {
    const product = await productRepository.findOneBy({ id });
    
    if (!product) {
      throw new Error('Product not found');
    }

    product.deletedBy = userId;
    await productRepository.save(product);

    // Soft delete sets deletedAt
    await productRepository.softDelete(id);
  }

  async getRecentlyUpdated(hours: number = 24): Promise<Product[]> {
    const cutoff = new Date(Date.now() - hours * 60 * 60 * 1000);
    
    return await productRepository.find({
      where: {
        updatedAt: MoreThan(cutoff),
      },
      order: {
        updatedAt: 'DESC',
      },
    });
  }

  async getStaleProducts(days: number = 30): Promise<Product[]> {
    return await productRepository
      .createQueryBuilder('product')
      .where('product.updatedAt < NOW() - INTERVAL :days DAY', { days })
      .getMany();
  }
}
```

### Migration for Timestamp Columns

```typescript
import { MigrationInterface, QueryRunner, TableColumn } from 'typeorm';

export class AddTimestampColumns1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.addColumn(
      'users',
      new TableColumn({
        name: 'created_at',
        type: 'timestamp',
        default: 'CURRENT_TIMESTAMP',
        isNullable: false,
      })
    );

    await queryRunner.addColumn(
      'users',
      new TableColumn({
        name: 'updated_at',
        type: 'timestamp',
        default: 'CURRENT_TIMESTAMP',
        onUpdate: 'CURRENT_TIMESTAMP',
        isNullable: false,
      })
    );
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropColumn('users', 'created_at');
    await queryRunner.dropColumn('users', 'updated_at');
  }
}
```

**Interview Tip**: Explain that `@CreateDateColumn` automatically sets the timestamp on entity creation and never changes, while `@UpdateDateColumn` sets the timestamp on creation and automatically updates on every modification. Emphasize their use for audit trails and data tracking. Highlight that these are managed by TypeORM, not database triggers. Mention combining with `@DeleteDateColumn` for soft deletes and manual audit fields (createdBy, updatedBy) for complete audit trails. Discuss timezone considerations: recommend `timestamp with time zone` for distributed systems. Note that query builder updates bypass automatic timestamp updates. A strong answer demonstrates understanding of both the automatic behavior and practical audit trail implementation patterns.

</details>

<details>
<summary>20. How do you define JSON columns in TypeORM?</summary>

JSON columns allow you to store complex structured data as JSON in the database. TypeORM provides several ways to define JSON columns with varying levels of performance and compatibility.

### Basic JSON Column

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // JSON column
  @Column('json')
  settings: {
    theme: string;
    language: string;
    notifications: boolean;
  };

  // JSON column with any type
  @Column('json')
  metadata: any;
}

// Usage
const user = new User();
user.name = 'John';
user.settings = {
  theme: 'dark',
  language: 'en',
  notifications: true,
};
user.metadata = {
  lastLogin: '2024-01-15',
  preferences: {
    fontSize: 14,
    autoSave: true,
  },
};

await userRepository.save(user);

// Retrieve and use
const foundUser = await userRepository.findOneBy({ id: 1 });
console.log(foundUser.settings.theme); // 'dark'
console.log(foundUser.metadata.preferences.fontSize); // 14
```

### JSON vs JSONB (PostgreSQL)

```typescript
@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // JSON - stored as text, no indexing
  @Column('json')
  basicData: any;
  // Slower for queries, but faster for write operations

  // JSONB - binary JSON, can be indexed (PostgreSQL only)
  @Column('jsonb')
  specifications: {
    weight: number;
    dimensions: { width: number; height: number; depth: number };
    features: string[];
  };
  // Faster for queries, supports indexing, slightly slower writes
}

// Querying JSONB (PostgreSQL)
const products = await productRepository
  .createQueryBuilder('product')
  .where("product.specifications->>'weight' > :weight", { weight: 100 })
  .getMany();

// JSONB array contains
const productsWithFeature = await productRepository
  .createQueryBuilder('product')
  .where("product.specifications->'features' @> :feature", {
    feature: JSON.stringify(['waterproof']),
  })
  .getMany();
```

### Simple JSON (Cross-Database Compatibility)

```typescript
@Entity()
export class Configuration {
  @PrimaryGeneratedColumn()
  id: number;

  // simple-json: stored as string, works on all databases
  @Column('simple-json')
  settings: {
    theme: string;
    notifications: boolean;
    language: string;
  };
  // Serialized as JSON string in database
  // Less efficient for queries, but compatible with all databases
}

// Usage is the same as regular JSON
const config = new Configuration();
config.settings = {
  theme: 'light',
  notifications: true,
  language: 'en',
};
```

### Typed JSON Columns

```typescript
// Define interfaces for type safety
interface UserSettings {
  theme: 'light' | 'dark';
  language: string;
  notifications: boolean;
  privacy: {
    showEmail: boolean;
    showProfile: boolean;
  };
}

interface UserMetadata {
  lastLogin?: Date;
  loginCount: number;
  devices: Array<{
    type: string;
    name: string;
    lastUsed: Date;
  }>;
}

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  email: string;

  // Strongly typed JSON
  @Column('jsonb')
  settings: UserSettings;

  @Column('jsonb')
  metadata: UserMetadata;
}

// Usage with type safety
const user = new User();
user.email = 'john@example.com';
user.settings = {
  theme: 'dark', // Type-checked: only 'light' or 'dark' allowed
  language: 'en',
  notifications: true,
  privacy: {
    showEmail: false,
    showProfile: true,
  },
};
user.metadata = {
  loginCount: 5,
  devices: [
    {
      type: 'desktop',
      name: 'Chrome',
      lastUsed: new Date(),
    },
  ],
};

await userRepository.save(user);
```

### JSON with Default Values

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // JSON with default value
  @Column({
    type: 'json',
    default: {
      theme: 'light',
      language: 'en',
      notifications: true,
    },
  })
  settings: any;

  // JSONB with default (PostgreSQL)
  @Column({
    type: 'jsonb',
    default: () => "'{}'", // Empty JSON object
  })
  metadata: any;

  // Nullable JSON
  @Column({ type: 'json', nullable: true })
  optionalData: any | null;
}
```

### Querying JSON Columns

```typescript
import { AppDataSource } from './data-source';
import { User } from './entities/User';

const userRepository = AppDataSource.getRepository(User);

// PostgreSQL JSONB queries
// 1. Query by JSON field
const darkThemeUsers = await userRepository
  .createQueryBuilder('user')
  .where("user.settings->>'theme' = :theme", { theme: 'dark' })
  .getMany();

// 2. Query nested JSON
const users = await userRepository
  .createQueryBuilder('user')
  .where("user.settings->'privacy'->>'showEmail' = :show", { show: 'true' })
  .getMany();

// 3. Check if JSON contains key
const usersWithNotifications = await userRepository
  .createQueryBuilder('user')
  .where("user.settings ? 'notifications'")
  .getMany();

// 4. JSON array contains
const usersWithDevice = await userRepository
  .createQueryBuilder('user')
  .where("user.metadata->'devices' @> :device", {
    device: JSON.stringify([{ type: 'mobile' }]),
  })
  .getMany();

// 5. JSON equality
const specificSettings = await userRepository
  .createQueryBuilder('user')
  .where('user.settings @> :settings', {
    settings: JSON.stringify({ theme: 'dark', language: 'en' }),
  })
  .getMany();

// MySQL JSON queries
// 1. Extract JSON field (MySQL)
const mobileUsers = await userRepository
  .createQueryBuilder('user')
  .where("JSON_EXTRACT(user.settings, '$.theme') = :theme", { theme: 'dark' })
  .getMany();

// 2. JSON contains (MySQL)
const usersWithFeature = await userRepository
  .createQueryBuilder('user')
  .where("JSON_CONTAINS(user.settings, :value, '$.notifications')", {
    value: 'true',
  })
  .getMany();
```

### Updating JSON Fields

```typescript
// Update entire JSON object
const user = await userRepository.findOneBy({ id: 1 });
user.settings = {
  theme: 'light',
  language: 'fr',
  notifications: false,
};
await userRepository.save(user);

// Update specific JSON field (PostgreSQL)
await userRepository
  .createQueryBuilder()
  .update(User)
  .set({
    settings: () => "jsonb_set(settings, '{theme}', '\"dark\"')",
  })
  .where('id = :id', { id: 1 })
  .execute();

// Update nested JSON field (PostgreSQL)
await userRepository
  .createQueryBuilder()
  .update(User)
  .set({
    settings: () => "jsonb_set(settings, '{privacy,showEmail}', 'true')",
  })
  .where('id = :id', { id: 1 })
  .execute();

// Update JSON field in application code
const user = await userRepository.findOneBy({ id: 1 });
user.settings.theme = 'dark'; // Modify JSON field
await userRepository.save(user); // Save entire object
```

### JSON Arrays

```typescript
interface Tag {
  name: string;
  color: string;
}

interface Comment {
  author: string;
  text: string;
  timestamp: Date;
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  // JSON array
  @Column('jsonb')
  tags: Tag[];

  @Column('jsonb')
  comments: Comment[];

  // Simple array (comma-separated)
  @Column('simple-array')
  categories: string[]; // Stored as: "cat1,cat2,cat3"
}

// Usage
const post = new Post();
post.title = 'My Post';
post.tags = [
  { name: 'typescript', color: 'blue' },
  { name: 'database', color: 'green' },
];
post.comments = [
  {
    author: 'John',
    text: 'Great post!',
    timestamp: new Date(),
  },
];
post.categories = ['tutorial', 'backend'];

await postRepository.save(post);

// Query posts with specific tag
const postsWithTag = await postRepository
  .createQueryBuilder('post')
  .where("post.tags @> :tag", {
    tag: JSON.stringify([{ name: 'typescript' }]),
  })
  .getMany();
```

### Complex Nested JSON Structures

```typescript
interface Address {
  street: string;
  city: string;
  state: string;
  zipCode: string;
  coordinates?: {
    latitude: number;
    longitude: number;
  };
}

interface ContactInfo {
  primaryEmail: string;
  secondaryEmail?: string;
  phones: Array<{
    type: 'mobile' | 'home' | 'work';
    number: string;
    primary: boolean;
  }>;
  addresses: Address[];
}

@Entity()
export class Customer {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Complex nested JSON
  @Column('jsonb')
  contactInfo: ContactInfo;

  // Preferences with nested structure
  @Column('jsonb')
  preferences: {
    communication: {
      email: boolean;
      sms: boolean;
      push: boolean;
    };
    marketing: {
      newsletter: boolean;
      promotions: boolean;
      surveys: boolean;
    };
    display: {
      theme: string;
      language: string;
      timezone: string;
    };
  };
}

// Usage
const customer = new Customer();
customer.name = 'John Doe';
customer.contactInfo = {
  primaryEmail: 'john@example.com',
  secondaryEmail: 'john.doe@work.com',
  phones: [
    { type: 'mobile', number: '+1234567890', primary: true },
    { type: 'work', number: '+0987654321', primary: false },
  ],
  addresses: [
    {
      street: '123 Main St',
      city: 'New York',
      state: 'NY',
      zipCode: '10001',
      coordinates: {
        latitude: 40.7128,
        longitude: -74.0060,
      },
    },
  ],
};
customer.preferences = {
  communication: { email: true, sms: false, push: true },
  marketing: { newsletter: true, promotions: false, surveys: false },
  display: { theme: 'dark', language: 'en', timezone: 'America/New_York' },
};

await customerRepository.save(customer);
```

### JSON Column with Validation

```typescript
import { Entity, PrimaryGeneratedColumn, Column, BeforeInsert, BeforeUpdate } from 'typeorm';

interface ProductSpec {
  weight: number;
  dimensions: {
    width: number;
    height: number;
    depth: number;
  };
  features: string[];
}

@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column('jsonb')
  specifications: ProductSpec;

  // Validation hook
  @BeforeInsert()
  @BeforeUpdate()
  validateSpecifications() {
    if (!this.specifications) {
      throw new Error('Specifications are required');
    }

    // Validate weight
    if (this.specifications.weight <= 0) {
      throw new Error('Weight must be positive');
    }

    // Validate dimensions
    const { width, height, depth } = this.specifications.dimensions;
    if (width <= 0 || height <= 0 || depth <= 0) {
      throw new Error('All dimensions must be positive');
    }

    // Validate features array
    if (!Array.isArray(this.specifications.features)) {
      throw new Error('Features must be an array');
    }
  }
}
```

### Indexing JSON Columns (PostgreSQL)

```typescript
import { MigrationInterface, QueryRunner } from 'typeorm';

export class AddJsonIndexes1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    // Create GIN index on entire JSONB column (general purpose)
    await queryRunner.query(`
      CREATE INDEX idx_user_settings 
      ON "user" USING gin (settings)
    `);

    // Create index on specific JSON field
    await queryRunner.query(`
      CREATE INDEX idx_user_settings_theme 
      ON "user" ((settings->>'theme'))
    `);

    // Create index on nested JSON field
    await queryRunner.query(`
      CREATE INDEX idx_user_privacy_show_email 
      ON "user" ((settings->'privacy'->>'showEmail'))
    `);

    // Create partial index
    await queryRunner.query(`
      CREATE INDEX idx_dark_theme_users 
      ON "user" ((settings->>'theme'))
      WHERE settings->>'theme' = 'dark'
    `);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP INDEX idx_user_settings`);
    await queryRunner.query(`DROP INDEX idx_user_settings_theme`);
    await queryRunner.query(`DROP INDEX idx_user_privacy_show_email`);
    await queryRunner.query(`DROP INDEX idx_dark_theme_users`);
  }
}
```

### Real-World E-commerce Example

```typescript
interface CartItem {
  productId: number;
  productName: string;
  quantity: number;
  price: number;
  options?: {
    size?: string;
    color?: string;
    customization?: string;
  };
}

interface ShippingAddress {
  recipientName: string;
  street: string;
  city: string;
  state: string;
  zipCode: string;
  country: string;
  phone: string;
}

interface OrderMetadata {
  source: 'web' | 'mobile' | 'api';
  device: {
    type: string;
    os: string;
    browser?: string;
  };
  campaign?: {
    name: string;
    source: string;
    medium: string;
  };
  couponCodes?: string[];
  giftMessage?: string;
  specialInstructions?: string;
}

@Entity()
export class Order {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  orderNumber: string;

  @Column()
  customerId: number;

  // Cart items as JSON
  @Column('jsonb')
  items: CartItem[];

  // Shipping address
  @Column('jsonb')
  shippingAddress: ShippingAddress;

  // Billing address (can be different from shipping)
  @Column('jsonb')
  billingAddress: ShippingAddress;

  // Order metadata
  @Column('jsonb')
  metadata: OrderMetadata;

  // Payment details (sensitive, encrypted separately)
  @Column({ type: 'jsonb', select: false })
  paymentInfo: {
    method: string;
    last4?: string;
    expiryMonth?: number;
    expiryYear?: number;
  };

  @Column('decimal', { precision: 10, scale: 2 })
  subtotal: number;

  @Column('decimal', { precision: 10, scale: 2 })
  tax: number;

  @Column('decimal', { precision: 10, scale: 2 })
  shipping: number;

  @Column('decimal', { precision: 10, scale: 2 })
  total: number;

  @Column({
    type: 'enum',
    enum: ['pending', 'paid', 'shipped', 'delivered', 'cancelled'],
    default: 'pending',
  })
  status: string;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  // Helper methods
  getTotalItems(): number {
    return this.items.reduce((sum, item) => sum + item.quantity, 0);
  }

  getItemByProductId(productId: number): CartItem | undefined {
    return this.items.find(item => item.productId === productId);
  }

  calculateSubtotal(): number {
    return this.items.reduce(
      (sum, item) => sum + item.price * item.quantity,
      0
    );
  }
}

// Service with JSON operations
class OrderService {
  async createOrder(customerId: number, orderData: Partial<Order>): Promise<Order> {
    const order = orderRepository.create({
      customerId,
      orderNumber: generateOrderNumber(),
      items: orderData.items || [],
      shippingAddress: orderData.shippingAddress,
      billingAddress: orderData.billingAddress || orderData.shippingAddress,
      metadata: {
        source: 'web',
        device: {
          type: 'desktop',
          os: 'Windows',
          browser: 'Chrome',
        },
        ...orderData.metadata,
      },
    });

    // Calculate totals
    order.subtotal = order.calculateSubtotal();
    order.tax = order.subtotal * 0.1; // 10% tax
    order.shipping = 9.99;
    order.total = order.subtotal + order.tax + order.shipping;

    return await orderRepository.save(order);
  }

  async updateOrderItem(orderId: string, productId: number, quantity: number): Promise<Order> {
    const order = await orderRepository.findOneBy({ id: orderId });
    
    if (!order) {
      throw new Error('Order not found');
    }

    const item = order.getItemByProductId(productId);
    if (item) {
      item.quantity = quantity;
      order.subtotal = order.calculateSubtotal();
      order.total = order.subtotal + order.tax + order.shipping;
      return await orderRepository.save(order);
    }

    throw new Error('Item not found in order');
  }

  async findOrdersByDevice(deviceType: string): Promise<Order[]> {
    return await orderRepository
      .createQueryBuilder('order')
      .where("order.metadata->'device'->>'type' = :deviceType", { deviceType })
      .getMany();
  }

  async findOrdersWithCoupon(couponCode: string): Promise<Order[]> {
    return await orderRepository
      .createQueryBuilder('order')
      .where("order.metadata->'couponCodes' @> :coupon", {
        coupon: JSON.stringify([couponCode]),
      })
      .getMany();
  }
}
```

### Best Practices

```typescript
// ✅ GOOD: Use typed interfaces
interface UserSettings {
  theme: 'light' | 'dark';
  language: string;
}

@Column('jsonb')
settings: UserSettings; // Type-safe

// ❌ BAD: Using any without structure
@Column('json')
settings: any; // No type safety

// ✅ GOOD: Use JSONB for PostgreSQL (better performance)
@Column('jsonb')
data: any;

// ✅ GOOD: Provide default values
@Column({ type: 'jsonb', default: {} })
metadata: any;

// ✅ GOOD: Use nullable for optional JSON
@Column({ type: 'json', nullable: true })
optionalData: any | null;

// ✅ GOOD: Index frequently queried JSON fields (PostgreSQL)
// CREATE INDEX idx_theme ON users ((settings->>'theme'))

// ❌ BAD: Don't store large binary data in JSON
// Use BLOB/BYTEA columns instead

// ✅ GOOD: Validate JSON structure in hooks
@BeforeInsert()
@BeforeUpdate()
validateJson() {
  // Validate structure
}
```

**Interview Tip**: Explain that TypeORM supports `json`, `jsonb` (PostgreSQL), and `simple-json` types. Emphasize `jsonb` advantages for PostgreSQL: indexable, faster queries, supports operators. Discuss when to use JSON: flexible schemas, configuration data, metadata, audit logs. Mention querying: PostgreSQL has rich JSON operators (`->`, `->>`, `@>`), MySQL uses `JSON_EXTRACT`. Highlight type safety: use TypeScript interfaces for JSON structures. For production: index frequently queried fields, validate JSON structure, prefer `jsonb` over `json` in PostgreSQL, avoid storing large binary data in JSON. A strong answer demonstrates understanding of both database-specific features and practical JSON usage patterns.

</details>

<details>
<summary>21. What are embedded entities and how do you use them?</summary>

**Embedded entities** allow you to group related columns into reusable classes that can be embedded in multiple entities. They help organize code, reduce duplication, and improve maintainability without creating separate database tables.

### Basic Embedded Entity

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

// Embedded entity (not a table)
class Name {
  @Column()
  first: string;

  @Column()
  last: string;

  // Computed property
  get full(): string {
    return `${this.first} ${this.last}`;
  }
}

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  // Embed Name class
  @Column(() => Name)
  name: Name;

  @Column()
  email: string;
}

// Database table structure:
// users table has columns: id, first, last, email
// No separate 'name' table is created

// Usage
const user = new User();
user.name = new Name();
user.name.first = 'John';
user.name.last = 'Doe';
user.email = 'john@example.com';

await userRepository.save(user);

console.log(user.name.full); // "John Doe"
```

### Embedded Entity with Prefix

```typescript
class Address {
  @Column()
  street: string;

  @Column()
  city: string;

  @Column()
  state: string;

  @Column()
  zipCode: string;
}

@Entity()
export class Company {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Embedded with prefix
  @Column(() => Address, { prefix: 'billing_' })
  billingAddress: Address;

  @Column(() => Address, { prefix: 'shipping_' })
  shippingAddress: Address;
}

// Database columns:
// id, name, 
// billing_street, billing_city, billing_state, billing_zipCode,
// shipping_street, shipping_city, shipping_state, shipping_zipCode

// Usage
const company = new Company();
company.name = 'Acme Corp';

company.billingAddress = new Address();
company.billingAddress.street = '123 Business St';
company.billingAddress.city = 'New York';
company.billingAddress.state = 'NY';
company.billingAddress.zipCode = '10001';

company.shippingAddress = new Address();
company.shippingAddress.street = '456 Warehouse Ave';
company.shippingAddress.city = 'Los Angeles';
company.shippingAddress.state = 'CA';
company.shippingAddress.zipCode = '90001';

await companyRepository.save(company);
```

### Embedded Entity with Custom Column Names

```typescript
class ContactInfo {
  @Column({ name: 'email_address' })
  email: string;

  @Column({ name: 'phone_number' })
  phone: string;

  @Column({ name: 'mobile_number', nullable: true })
  mobile: string | null;
}

@Entity()
export class Customer {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column(() => ContactInfo)
  contact: ContactInfo;
}

// Database columns:
// id, name, email_address, phone_number, mobile_number
```

### Nested Embedded Entities

```typescript
class Coordinates {
  @Column('decimal', { precision: 10, scale: 7 })
  latitude: number;

  @Column('decimal', { precision: 10, scale: 7 })
  longitude: number;
}

class Address {
  @Column()
  street: string;

  @Column()
  city: string;

  @Column()
  zipCode: string;

  // Nested embedded entity
  @Column(() => Coordinates)
  coordinates: Coordinates;
}

@Entity()
export class Location {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Embedded entity with nested embedded entity
  @Column(() => Address)
  address: Address;
}

// Database columns:
// id, name, street, city, zipCode, latitude, longitude

// Usage
const location = new Location();
location.name = 'Main Office';

location.address = new Address();
location.address.street = '123 Main St';
location.address.city = 'New York';
location.address.zipCode = '10001';

location.address.coordinates = new Coordinates();
location.address.coordinates.latitude = 40.7128;
location.address.coordinates.longitude = -74.0060;

await locationRepository.save(location);
```

### Multiple Embedded Entities

```typescript
class PersonalInfo {
  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column({ type: 'date' })
  dateOfBirth: Date;

  @Column({ nullable: true })
  ssn: string | null;
}

class ContactInfo {
  @Column()
  email: string;

  @Column()
  phone: string;

  @Column({ nullable: true })
  alternatePhone: string | null;
}

class Address {
  @Column()
  street: string;

  @Column()
  city: string;

  @Column()
  state: string;

  @Column()
  zipCode: string;
}

@Entity()
export class Employee {
  @PrimaryGeneratedColumn()
  id: number;

  @Column(() => PersonalInfo)
  personal: PersonalInfo;

  @Column(() => ContactInfo)
  contact: ContactInfo;

  @Column(() => Address, { prefix: 'home_' })
  homeAddress: Address;

  @Column(() => Address, { prefix: 'work_' })
  workAddress: Address;

  @Column('decimal', { precision: 10, scale: 2 })
  salary: number;

  @Column({ type: 'date' })
  hireDate: Date;
}

// Database columns include all embedded fields with appropriate prefixes
```

### Embedded Entity with Default Values

```typescript
class Preferences {
  @Column({ default: 'light' })
  theme: string;

  @Column({ default: 'en' })
  language: string;

  @Column({ default: true })
  notifications: boolean;

  @Column({ default: false })
  newsletter: boolean;
}

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  email: string;

  @Column(() => Preferences)
  preferences: Preferences;
}

// Usage - preferences get default values if not set
const user = new User();
user.email = 'john@example.com';
user.preferences = new Preferences(); // Gets default values

await userRepository.save(user);
```

### Embedded Entity with Methods

```typescript
class Money {
  @Column('decimal', { precision: 10, scale: 2 })
  amount: number;

  @Column({ length: 3 })
  currency: string;

  // Methods in embedded entity
  format(): string {
    return `${this.currency} ${this.amount.toFixed(2)}`;
  }

  convertTo(rate: number, targetCurrency: string): Money {
    const converted = new Money();
    converted.amount = this.amount * rate;
    converted.currency = targetCurrency;
    return converted;
  }

  add(other: Money): Money {
    if (this.currency !== other.currency) {
      throw new Error('Cannot add different currencies');
    }
    const result = new Money();
    result.amount = this.amount + other.amount;
    result.currency = this.currency;
    return result;
  }
}

@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column(() => Money)
  price: Money;

  @Column(() => Money, { prefix: 'cost_' })
  cost: Money;

  // Calculate profit
  getProfit(): Money {
    if (this.price.currency !== this.cost.currency) {
      throw new Error('Price and cost must be in same currency');
    }
    const profit = new Money();
    profit.amount = this.price.amount - this.cost.amount;
    profit.currency = this.price.currency;
    return profit;
  }
}

// Usage
const product = new Product();
product.name = 'Laptop';

product.price = new Money();
product.price.amount = 1200;
product.price.currency = 'USD';

product.cost = new Money();
product.cost.amount = 800;
product.cost.currency = 'USD';

await productRepository.save(product);

console.log(product.price.format()); // "USD 1200.00"
const profit = product.getProfit();
console.log(profit.format()); // "USD 400.00"
```

### Querying with Embedded Entities

```typescript
class Address {
  @Column()
  city: string;

  @Column()
  state: string;

  @Column()
  zipCode: string;
}

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column(() => Address)
  address: Address;
}

// Query by embedded entity fields
const usersInNY = await userRepository.find({
  where: {
    address: {
      state: 'NY',
    },
  },
});

// Query builder
const usersInCity = await userRepository
  .createQueryBuilder('user')
  .where('user.city = :city', { city: 'New York' })
  .andWhere('user.state = :state', { state: 'NY' })
  .getMany();

// Complex query
const users = await userRepository
  .createQueryBuilder('user')
  .where('user.city IN (:...cities)', { cities: ['New York', 'Los Angeles'] })
  .andWhere('user.zipCode LIKE :zip', { zip: '100%' })
  .getMany();
```

### Embedded Entity with Validation

```typescript
import { Column } from 'typeorm';

class Email {
  @Column()
  address: string;

  @Column({ default: false })
  verified: boolean;

  @Column({ type: 'timestamp', nullable: true })
  verifiedAt: Date | null;

  // Validation method
  isValid(): boolean {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(this.address);
  }

  // Domain extraction
  getDomain(): string {
    return this.address.split('@')[1];
  }
}

class PhoneNumber {
  @Column()
  number: string;

  @Column({ length: 2 })
  countryCode: string;

  @Column({ default: false })
  verified: boolean;

  // Format method
  format(): string {
    return `+${this.countryCode} ${this.number}`;
  }

  // Validation
  isValid(): boolean {
    // Simple validation - only digits
    return /^\d+$/.test(this.number);
  }
}

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column(() => Email)
  email: Email;

  @Column(() => PhoneNumber, { prefix: 'phone_' })
  phone: PhoneNumber;

  @BeforeInsert()
  @BeforeUpdate()
  validate() {
    if (!this.email.isValid()) {
      throw new Error('Invalid email address');
    }
    if (!this.phone.isValid()) {
      throw new Error('Invalid phone number');
    }
  }
}
```

### Real-World Example: E-commerce

```typescript
import { Entity, PrimaryGeneratedColumn, Column, CreateDateColumn } from 'typeorm';

// Reusable embedded entities
class Money {
  @Column('decimal', { precision: 10, scale: 2 })
  amount: number;

  @Column({ length: 3, default: 'USD' })
  currency: string;

  format(): string {
    return new Intl.NumberFormat('en-US', {
      style: 'currency',
      currency: this.currency,
    }).format(this.amount);
  }
}

class Address {
  @Column()
  recipientName: string;

  @Column()
  street: string;

  @Column({ nullable: true })
  apartment: string | null;

  @Column()
  city: string;

  @Column()
  state: string;

  @Column()
  zipCode: string;

  @Column({ default: 'US' })
  country: string;

  @Column()
  phone: string;

  format(): string {
    return `${this.recipientName}\n${this.street}${
      this.apartment ? ` ${this.apartment}` : ''
    }\n${this.city}, ${this.state} ${this.zipCode}\n${this.country}`;
  }
}

class Dimensions {
  @Column('decimal', { precision: 10, scale: 2 })
  length: number;

  @Column('decimal', { precision: 10, scale: 2 })
  width: number;

  @Column('decimal', { precision: 10, scale: 2 })
  height: number;

  @Column({ length: 10, default: 'cm' })
  unit: string;

  getVolume(): number {
    return this.length * this.width * this.height;
  }

  format(): string {
    return `${this.length}x${this.width}x${this.height} ${this.unit}`;
  }
}

class Weight {
  @Column('decimal', { precision: 10, scale: 3 })
  value: number;

  @Column({ length: 10, default: 'kg' })
  unit: string;

  format(): string {
    return `${this.value} ${this.unit}`;
  }
}

// Product entity using embedded entities
@Entity()
export class Product {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  name: string;

  @Column('text')
  description: string;

  @Column(() => Money, { prefix: 'price_' })
  price: Money;

  @Column(() => Money, { prefix: 'cost_' })
  cost: Money;

  @Column(() => Dimensions)
  dimensions: Dimensions;

  @Column(() => Weight)
  weight: Weight;

  @Column({ type: 'int', default: 0 })
  stock: number;

  @CreateDateColumn()
  createdAt: Date;

  // Business logic
  getProfit(): Money {
    const profit = new Money();
    profit.amount = this.price.amount - this.cost.amount;
    profit.currency = this.price.currency;
    return profit;
  }

  getProfitMargin(): number {
    if (this.price.amount === 0) return 0;
    return ((this.price.amount - this.cost.amount) / this.price.amount) * 100;
  }

  isInStock(): boolean {
    return this.stock > 0;
  }
}

// Order entity using embedded entities
@Entity()
export class Order {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  orderNumber: string;

  @Column()
  customerId: number;

  @Column(() => Address, { prefix: 'shipping_' })
  shippingAddress: Address;

  @Column(() => Address, { prefix: 'billing_' })
  billingAddress: Address;

  @Column(() => Money, { prefix: 'subtotal_' })
  subtotal: Money;

  @Column(() => Money, { prefix: 'tax_' })
  tax: Money;

  @Column(() => Money, { prefix: 'shipping_' })
  shippingCost: Money;

  @Column(() => Money, { prefix: 'total_' })
  total: Money;

  @CreateDateColumn()
  createdAt: Date;

  // Calculate total from components
  calculateTotal(): Money {
    const total = new Money();
    total.amount =
      this.subtotal.amount + this.tax.amount + this.shippingCost.amount;
    total.currency = this.subtotal.currency;
    return total;
  }
}

// Usage
const product = new Product();
product.name = 'Laptop';
product.description = 'High-performance laptop';

product.price = new Money();
product.price.amount = 1299.99;
product.price.currency = 'USD';

product.cost = new Money();
product.cost.amount = 799.99;
product.cost.currency = 'USD';

product.dimensions = new Dimensions();
product.dimensions.length = 35;
product.dimensions.width = 25;
product.dimensions.height = 2;
product.dimensions.unit = 'cm';

product.weight = new Weight();
product.weight.value = 1.5;
product.weight.unit = 'kg';

await productRepository.save(product);

console.log(product.price.format()); // "$1,299.99"
console.log(product.dimensions.format()); // "35x25x2 cm"
console.log(product.weight.format()); // "1.5 kg"
console.log(product.getProfit().format()); // "$500.00"
console.log(product.getProfitMargin()); // 38.46
```

### Benefits and Use Cases

```typescript
// ✅ GOOD: Use embedded entities for:
// 1. Reusable value objects
class Money { /* ... */ }
class Address { /* ... */ }

// 2. Grouping related fields
class ContactInfo {
  @Column() email: string;
  @Column() phone: string;
}

// 3. Domain-driven design value objects
class DateRange {
  @Column() startDate: Date;
  @Column() endDate: Date;
  getDuration(): number { /* ... */ }
}

// 4. Avoiding duplication
// Use same Address class for shipping and billing

// ❌ BAD: Don't use embedded entities for:
// - Entities with their own identity (use relations instead)
// - Data that should be in separate table (use @OneToOne, @ManyToOne)
// - Collections of items (use @OneToMany, @ManyToMany)
```

**Interview Tip**: Explain that embedded entities are classes that group related columns without creating separate tables. Emphasize key benefits: code reuse (same Address for shipping/billing), better organization, encapsulation of business logic, and domain-driven design support. Highlight the `prefix` option for embedding the same class multiple times with different column prefixes. Mention that embedded entities are value objects - they don't have their own identity and lifecycle. Contrast with relations: use embedded entities for value objects that are part of the parent entity, use relations for entities with independent identity. Discuss when to use: reusable value objects (Money, Address), grouping related fields (ContactInfo), domain models. A strong answer demonstrates understanding of both technical implementation and proper domain modeling.

</details>

<details>
<summary>22. How do you use @Generated decorator for auto-increment columns?</summary>

The `@Generated` decorator creates columns with auto-generated values. Unlike `@PrimaryGeneratedColumn`, it can be used on non-primary columns and supports different generation strategies.

### Basic @Generated Usage

```typescript
import { Entity, PrimaryGeneratedColumn, Column, Generated } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number; // Primary key auto-increment

  @Column()
  @Generated('uuid')
  uuid: string; // Auto-generated UUID

  @Column()
  name: string;

  @Column()
  email: string;
}

// Usage
const user = new User();
user.name = 'John';
user.email = 'john@example.com';
// No need to set uuid - it's auto-generated

await userRepository.save(user);
console.log(user.uuid); // e.g., "550e8400-e29b-41d4-a716-446655440000"
```

### Generation Strategies

```typescript
@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Strategy 1: UUID generation
  @Column()
  @Generated('uuid')
  externalId: string; // Unique identifier for external systems

  // Strategy 2: Increment (auto-incrementing number)
  @Column()
  @Generated('increment')
  orderNumber: number; // Sequential order number

  // Strategy 3: Row ID (CockroachDB specific)
  @Column()
  @Generated('rowid')
  rowId: number; // CockroachDB row identifier
}

// Usage
const product = new Product();
product.name = 'Laptop';
// externalId and orderNumber are auto-generated

await productRepository.save(product);
console.log(product.externalId); // "a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11"
console.log(product.orderNumber); // 1, 2, 3, etc.
```

### UUID for External References

```typescript
@Entity()
export class Order {
  // Internal primary key (integer)
  @PrimaryGeneratedColumn()
  id: number;

  // External UUID for API/public references
  @Column()
  @Generated('uuid')
  publicId: string;

  @Column()
  orderNumber: string;

  @Column()
  customerId: number;

  @Column('decimal', { precision: 10, scale: 2 })
  total: number;
}

// Usage
const order = new Order();
order.orderNumber = 'ORD-2024-001';
order.customerId = 123;
order.total = 299.99;

await orderRepository.save(order);

// Use publicId for external APIs
console.log(order.id); // 1 (internal)
console.log(order.publicId); // "c4ca4238-a0b9-2382-0dcc-509a6f75849b" (external)

// Find by public ID
const found = await orderRepository.findOne({
  where: { publicId: order.publicId },
});
```

### Sequential Numbers for Different Purposes

```typescript
@Entity()
export class Invoice {
  @PrimaryGeneratedColumn()
  id: number;

  // Invoice number (auto-incremented)
  @Column()
  @Generated('increment')
  invoiceNumber: number;

  // External reference UUID
  @Column()
  @Generated('uuid')
  externalRef: string;

  @Column()
  customerId: number;

  @Column('decimal', { precision: 10, scale: 2 })
  amount: number;

  @Column({ type: 'date' })
  issueDate: Date;
}

// Usage
const invoice = new Invoice();
invoice.customerId = 100;
invoice.amount = 1500.00;
invoice.issueDate = new Date();

await invoiceRepository.save(invoice);
console.log(invoice.invoiceNumber); // 1, 2, 3... (sequential)
console.log(invoice.externalRef); // UUID
```

### Multiple Generated Columns

```typescript
@Entity()
export class Document {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  // Document number (sequential)
  @Column()
  @Generated('increment')
  documentNumber: number;

  // Version UUID (for version tracking)
  @Column()
  @Generated('uuid')
  versionId: string;

  // External API reference
  @Column()
  @Generated('uuid')
  apiKey: string;

  @Column('text')
  content: string;

  @CreateDateColumn()
  createdAt: Date;
}

// Usage
const doc = new Document();
doc.title = 'Contract Agreement';
doc.content = 'Terms and conditions...';

await documentRepository.save(doc);
console.log(doc.documentNumber); // 1
console.log(doc.versionId); // UUID for this version
console.log(doc.apiKey); // UUID for API access
```

### Custom Generated Values (Database Level)

```typescript
// For database-level generated columns
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  // Generated column (computed by database)
  @Column({
    type: 'varchar',
    generatedType: 'STORED',
    asExpression: "CONCAT(first_name, ' ', last_name)",
  })
  fullName: string;

  @Column()
  @Generated('uuid')
  uuid: string;

  // Generated timestamp
  @Column({
    type: 'timestamp',
    default: () => 'CURRENT_TIMESTAMP',
  })
  registeredAt: Date;
}
```

### UUID as Primary Key vs Generated Column

```typescript
// Option 1: UUID as primary key
@Entity()
export class UserV1 {
  @PrimaryGeneratedColumn('uuid')
  id: string; // UUID primary key

  @Column()
  email: string;
}

// Option 2: Integer primary key + UUID column
@Entity()
export class UserV2 {
  @PrimaryGeneratedColumn()
  id: number; // Integer primary key (faster joins)

  @Column()
  @Generated('uuid')
  uuid: string; // UUID for external references

  @Column()
  email: string;
}

// Option 2 is often better for:
// - Faster database performance (integer primary keys)
// - Better indexing
// - UUID for public/external references
```

### Tracking and Audit with Generated Columns

```typescript
@Entity()
export class AuditLog {
  @PrimaryGeneratedColumn()
  id: number;

  // Sequential log number
  @Column()
  @Generated('increment')
  logNumber: number;

  // Unique identifier for external systems
  @Column()
  @Generated('uuid')
  correlationId: string;

  @Column()
  userId: number;

  @Column()
  action: string;

  @Column('json')
  changes: any;

  @CreateDateColumn()
  timestamp: Date;
}

// Usage
const auditLog = new AuditLog();
auditLog.userId = 123;
auditLog.action = 'UPDATE_PROFILE';
auditLog.changes = {
  field: 'email',
  oldValue: 'old@example.com',
  newValue: 'new@example.com',
};

await auditLogRepository.save(auditLog);
console.log(auditLog.logNumber); // Sequential number for ordering
console.log(auditLog.correlationId); // UUID for tracking across systems
```

### Querying Generated Columns

```typescript
// Find by generated UUID
const user = await userRepository.findOne({
  where: { uuid: 'a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11' },
});

// Find by generated increment number
const invoice = await invoiceRepository.findOne({
  where: { invoiceNumber: 12345 },
});

// Query builder
const orders = await orderRepository
  .createQueryBuilder('order')
  .where('order.publicId = :publicId', { publicId: 'uuid-here' })
  .getOne();

// Range query on generated increment
const recentInvoices = await invoiceRepository
  .createQueryBuilder('invoice')
  .where('invoice.invoiceNumber BETWEEN :start AND :end', {
    start: 1000,
    end: 2000,
  })
  .getMany();
```

### Real-World Example: Order Management

```typescript
@Entity()
export class Order {
  // Internal ID (fast joins, relationships)
  @PrimaryGeneratedColumn()
  id: number;

  // Public-facing ID (customer-friendly, API)
  @Column()
  @Generated('uuid')
  publicId: string;

  // Human-readable order number (sequential)
  @Column()
  @Generated('increment')
  orderNumber: number;

  // Tracking number (UUID for logistics)
  @Column({ nullable: true })
  @Generated('uuid')
  trackingId: string | null;

  @Column()
  customerId: number;

  @Column()
  status: string;

  @Column('decimal', { precision: 10, scale: 2 })
  total: number;

  @CreateDateColumn()
  createdAt: Date;

  // Format order number for display
  getFormattedOrderNumber(): string {
    const year = this.createdAt.getFullYear();
    const paddedNumber = String(this.orderNumber).padStart(6, '0');
    return `ORD-${year}-${paddedNumber}`;
  }
}

// Service
class OrderService {
  async createOrder(data: Partial<Order>): Promise<Order> {
    const order = orderRepository.create(data);
    // id, publicId, orderNumber, trackingId auto-generated
    await orderRepository.save(order);

    // Send email with public ID
    await this.sendOrderConfirmation(order.publicId);

    return order;
  }

  async findByPublicId(publicId: string): Promise<Order | null> {
    return await orderRepository.findOne({
      where: { publicId },
    });
  }

  async findByOrderNumber(orderNumber: number): Promise<Order | null> {
    return await orderRepository.findOne({
      where: { orderNumber },
    });
  }

  async generateTrackingNumber(orderId: number): Promise<string> {
    const order = await orderRepository.findOneBy({ id: orderId });
    if (!order) {
      throw new Error('Order not found');
    }

    // trackingId already generated, just return it
    return order.trackingId;
  }
}
```

### Migration for Generated Columns

```typescript
import { MigrationInterface, QueryRunner, TableColumn } from 'typeorm';

export class AddGeneratedColumns1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    // Add UUID column (PostgreSQL)
    await queryRunner.addColumn(
      'users',
      new TableColumn({
        name: 'uuid',
        type: 'uuid',
        isGenerated: true,
        generationStrategy: 'uuid',
        default: 'uuid_generate_v4()',
      })
    );

    // Add auto-increment column
    await queryRunner.addColumn(
      'orders',
      new TableColumn({
        name: 'order_number',
        type: 'int',
        isGenerated: true,
        generationStrategy: 'increment',
      })
    );
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropColumn('users', 'uuid');
    await queryRunner.dropColumn('orders', 'order_number');
  }
}
```

### Best Practices

```typescript
// ✅ GOOD: Use integer primary key + UUID for external refs
@Entity()
export class User {
  @PrimaryGeneratedColumn() // Fast joins
  id: number;

  @Column()
  @Generated('uuid') // External API references
  publicId: string;

  @Column()
  email: string;
}

// ✅ GOOD: Sequential numbers for user-friendly identifiers
@Entity()
export class Invoice {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  @Generated('increment') // Human-readable: 1, 2, 3...
  invoiceNumber: number;
}

// ✅ GOOD: Multiple UUIDs for different purposes
@Entity()
export class Document {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  @Generated('uuid')
  publicId: string; // For sharing

  @Column()
  @Generated('uuid')
  editToken: string; // For editing

  @Column()
  @Generated('uuid')
  viewToken: string; // For viewing
}

// ❌ BAD: Don't manually set generated values
const user = new User();
user.uuid = 'manual-uuid'; // Will be overwritten by database

// ✅ GOOD: Let database generate values
const user = new User();
user.email = 'john@example.com';
// uuid is auto-generated
```

### Performance Considerations

```typescript
// Integer primary keys are faster than UUIDs
@Entity()
export class OptimizedEntity {
  @PrimaryGeneratedColumn() // Integer: 4 bytes, fast joins
  id: number;

  @Column()
  @Generated('uuid') // UUID: 16 bytes, for external use
  publicId: string;

  @Column()
  @Generated('increment') // Sequential, good for ordering
  sequenceNumber: number;
}

// Index generated columns for faster lookups
@Entity()
@Index(['publicId'])
@Index(['sequenceNumber'])
export class IndexedEntity {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  @Generated('uuid')
  publicId: string;

  @Column()
  @Generated('increment')
  sequenceNumber: number;
}
```

**Interview Tip**: Explain that `@Generated` creates auto-generated columns with strategies: `uuid` (universally unique identifier), `increment` (sequential numbers), `rowid` (CockroachDB). Emphasize the difference: `@PrimaryGeneratedColumn` is for primary keys, `@Generated` is for any column. Discuss use cases: UUID for external/public references (API endpoints, sharing), increment for human-readable sequential numbers (order numbers, invoice numbers). Highlight best practice: use integer primary keys for performance + UUID columns for external references. Mention that generated values are set by the database after save. For production: index generated columns used in queries, use UUIDs for distributed systems to avoid collisions. A strong answer demonstrates understanding of generation strategies and when to use each approach.

</details>

<details>
<summary>23. What is the @VersionColumn decorator used for?</summary>

The `@VersionColumn` decorator implements **optimistic locking** by automatically incrementing a version number on every update. It prevents lost updates when multiple users modify the same entity concurrently.

### Basic @VersionColumn Usage

```typescript
import { Entity, PrimaryGeneratedColumn, Column, VersionColumn } from 'typeorm';

@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column('decimal', { precision: 10, scale: 2 })
  price: number;

  @Column({ type: 'int', default: 0 })
  stock: number;

  // Version column for optimistic locking
  @VersionColumn()
  version: number;
}

// Usage
const product = new Product();
product.name = 'Laptop';
product.price = 999.99;
product.stock = 10;

await productRepository.save(product);
console.log(product.version); // 1 (initial version)

// Update
product.price = 899.99;
await productRepository.save(product);
console.log(product.version); // 2 (automatically incremented)
```

### How Optimistic Locking Works

```typescript
// Scenario: Two users editing the same product concurrently

// User 1 loads product
const product1 = await productRepository.findOneBy({ id: 1 });
console.log(product1.version); // 1

// User 2 loads product (same version)
const product2 = await productRepository.findOneBy({ id: 1 });
console.log(product2.version); // 1

// User 1 updates and saves
product1.price = 899.99;
await productRepository.save(product1);
console.log(product1.version); // 2 (incremented)

// User 2 tries to update (with old version)
product2.stock = 5;
try {
  await productRepository.save(product2);
  // This will fail because version is now 2, but product2 has version 1
} catch (error) {
  console.error('Optimistic lock failed: Entity was modified by another user');
  // User 2 must reload and reapply changes
}

// User 2 reloads and applies changes
const freshProduct = await productRepository.findOneBy({ id: 1 });
console.log(freshProduct.version); // 2
freshProduct.stock = 5;
await productRepository.save(freshProduct);
console.log(freshProduct.version); // 3 (success)
```

### Version Column with Updates

```typescript
@Entity()
export class Document {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column('text')
  content: string;

  @Column()
  authorId: number;

  @VersionColumn()
  version: number;

  @UpdateDateColumn()
  updatedAt: Date;
}

// Update operation
async function updateDocument(
  id: number,
  expectedVersion: number,
  newContent: string
): Promise<Document> {
  const document = await documentRepository.findOneBy({ id });

  if (!document) {
    throw new Error('Document not found');
  }

  // Check version before updating
  if (document.version !== expectedVersion) {
    throw new Error(
      `Document was modified by another user. Expected version ${expectedVersion}, but current version is ${document.version}`
    );
  }

  document.content = newContent;
  await documentRepository.save(document); // Version auto-incremented

  return document;
}

// Usage
const doc = await documentRepository.findOneBy({ id: 1 });
const originalVersion = doc.version;

// Simulate work/editing
await new Promise(resolve => setTimeout(resolve, 1000));

// Try to update with version check
try {
  await updateDocument(doc.id, originalVersion, 'Updated content');
  console.log('Update successful');
} catch (error) {
  console.error(error.message);
  // Handle conflict: reload and retry
}
```

### Query Builder with Version

```typescript
// Update with version check using query builder
const result = await documentRepository
  .createQueryBuilder()
  .update(Document)
  .set({ content: 'New content' })
  .where('id = :id', { id: 1 })
  .andWhere('version = :version', { version: expectedVersion })
  .execute();

if (result.affected === 0) {
  throw new Error('Document was modified by another user or not found');
}

// The version is automatically incremented by TypeORM
```

### Version Column Types

```typescript
@Entity()
export class Example {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Default: number (integer)
  @VersionColumn()
  version: number;

  // Or use timestamp for version tracking
  @VersionColumn({ type: 'timestamp' })
  versionTimestamp: Date;
}

// Number version: 1, 2, 3, 4...
// Timestamp version: tracks when each version was created
```

### Handling Version Conflicts

```typescript
class ConcurrencyError extends Error {
  constructor(
    public entityName: string,
    public entityId: number,
    public expectedVersion: number,
    public actualVersion: number
  ) {
    super(
      `${entityName} with id ${entityId} was modified. Expected version ${expectedVersion}, but current version is ${actualVersion}`
    );
  }
}

async function safeUpdate<T>(
  repository: Repository<T>,
  id: number,
  expectedVersion: number,
  updates: Partial<T>
): Promise<T> {
  const entity = await repository.findOneBy({ id } as any);

  if (!entity) {
    throw new Error('Entity not found');
  }

  // Check version
  const currentVersion = (entity as any).version;
  if (currentVersion !== expectedVersion) {
    throw new ConcurrencyError(
      repository.metadata.name,
      id,
      expectedVersion,
      currentVersion
    );
  }

  // Apply updates
  Object.assign(entity, updates);

  // Save (version auto-incremented)
  return await repository.save(entity);
}

// Usage with retry logic
async function updateWithRetry<T>(
  repository: Repository<T>,
  id: number,
  updateFn: (entity: T) => void,
  maxRetries = 3
): Promise<T> {
  let attempts = 0;

  while (attempts < maxRetries) {
    try {
      const entity = await repository.findOneBy({ id } as any);
      if (!entity) {
        throw new Error('Entity not found');
      }

      const version = (entity as any).version;

      // Apply updates
      updateFn(entity);

      // Try to save
      return await safeUpdate(repository, id, version, entity);
    } catch (error) {
      if (error instanceof ConcurrencyError) {
        attempts++;
        if (attempts >= maxRetries) {
          throw new Error(
            `Failed to update after ${maxRetries} attempts due to concurrent modifications`
          );
        }
        // Wait before retrying (exponential backoff)
        await new Promise(resolve =>
          setTimeout(resolve, Math.pow(2, attempts) * 100)
        );
      } else {
        throw error;
      }
    }
  }

  throw new Error('Update failed');
}

// Usage
await updateWithRetry(productRepository, 1, product => {
  product.price = 799.99;
  product.stock -= 1;
});
```

### Real-World Example: Inventory Management

```typescript
@Entity()
export class InventoryItem {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  productId: number;

  @Column()
  warehouseId: number;

  @Column({ type: 'int' })
  quantity: number;

  @Column({ type: 'int', default: 0 })
  reserved: number; // Quantity reserved for pending orders

  @VersionColumn()
  version: number;

  @UpdateDateColumn()
  lastUpdated: Date;

  // Available quantity
  getAvailable(): number {
    return this.quantity - this.reserved;
  }
}

class InventoryService {
  // Reserve stock for an order
  async reserveStock(
    productId: number,
    warehouseId: number,
    quantity: number
  ): Promise<void> {
    const maxRetries = 5;
    let attempts = 0;

    while (attempts < maxRetries) {
      try {
        const inventory = await inventoryRepository.findOne({
          where: { productId, warehouseId },
        });

        if (!inventory) {
          throw new Error('Inventory item not found');
        }

        // Check available stock
        if (inventory.getAvailable() < quantity) {
          throw new Error('Insufficient stock');
        }

        const currentVersion = inventory.version;

        // Reserve stock
        inventory.reserved += quantity;

        // Save with version check
        await inventoryRepository
          .createQueryBuilder()
          .update(InventoryItem)
          .set({ reserved: inventory.reserved })
          .where('id = :id', { id: inventory.id })
          .andWhere('version = :version', { version: currentVersion })
          .execute();

        // Verify update succeeded
        const updated = await inventoryRepository.findOneBy({
          id: inventory.id,
        });

        if (updated && updated.version === currentVersion + 1) {
          return; // Success
        }

        // Version mismatch, retry
        throw new ConcurrencyError(
          'InventoryItem',
          inventory.id,
          currentVersion,
          updated?.version || -1
        );
      } catch (error) {
        if (error instanceof ConcurrencyError) {
          attempts++;
          if (attempts >= maxRetries) {
            throw new Error(
              `Failed to reserve stock after ${maxRetries} attempts. Please try again.`
            );
          }
          // Exponential backoff
          await new Promise(resolve =>
            setTimeout(resolve, Math.pow(2, attempts) * 50)
          );
        } else {
          throw error;
        }
      }
    }
  }

  // Release reserved stock
  async releaseStock(
    productId: number,
    warehouseId: number,
    quantity: number
  ): Promise<void> {
    const inventory = await inventoryRepository.findOne({
      where: { productId, warehouseId },
    });

    if (!inventory) {
      throw new Error('Inventory item not found');
    }

    inventory.reserved -= quantity;

    // Ensure reserved doesn't go negative
    if (inventory.reserved < 0) {
      inventory.reserved = 0;
    }

    await inventoryRepository.save(inventory);
  }

  // Confirm order and decrease quantity
  async confirmOrder(
    productId: number,
    warehouseId: number,
    quantity: number
  ): Promise<void> {
    const inventory = await inventoryRepository.findOne({
      where: { productId, warehouseId },
    });

    if (!inventory) {
      throw new Error('Inventory item not found');
    }

    inventory.quantity -= quantity;
    inventory.reserved -= quantity;

    await inventoryRepository.save(inventory);
  }
}
```

### Version Column with Transactions

```typescript
@Entity()
export class BankAccount {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  accountNumber: string;

  @Column('decimal', { precision: 15, scale: 2 })
  balance: number;

  @VersionColumn()
  version: number;

  @UpdateDateColumn()
  lastTransaction: Date;
}

async function transferMoney(
  fromAccountId: number,
  toAccountId: number,
  amount: number
): Promise<void> {
  const queryRunner = AppDataSource.createQueryRunner();
  await queryRunner.connect();
  await queryRunner.startTransaction();

  try {
    // Load accounts with version
    const fromAccount = await queryRunner.manager.findOneBy(BankAccount, {
      id: fromAccountId,
    });
    const toAccount = await queryRunner.manager.findOneBy(BankAccount, {
      id: toAccountId,
    });

    if (!fromAccount || !toAccount) {
      throw new Error('Account not found');
    }

    if (fromAccount.balance < amount) {
      throw new Error('Insufficient funds');
    }

    const fromVersion = fromAccount.version;
    const toVersion = toAccount.version;

    // Deduct from sender
    fromAccount.balance -= amount;

    // Add to receiver
    toAccount.balance += amount;

    // Save both accounts (versions will be checked automatically)
    await queryRunner.manager.save(fromAccount);
    await queryRunner.manager.save(toAccount);

    // Commit transaction
    await queryRunner.commitTransaction();
  } catch (error) {
    // Rollback on error
    await queryRunner.rollbackTransaction();
    throw error;
  } finally {
    await queryRunner.release();
  }
}
```

### Combining Version with Soft Delete

```typescript
@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column('text')
  content: string;

  @Column()
  authorId: number;

  @VersionColumn()
  version: number;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  @DeleteDateColumn()
  deletedAt: Date | null;
}

// Update preserves version even with soft delete
const post = await postRepository.findOneBy({ id: 1 });
console.log(post.version); // e.g., 5

// Soft delete
await postRepository.softDelete(1);

// Restore
await postRepository.restore(1);

const restored = await postRepository.findOneBy({ id: 1 });
console.log(restored.version); // Still 5 (soft delete doesn't increment)

// But regular update does increment
restored.title = 'Updated Title';
await postRepository.save(restored);
console.log(restored.version); // 6
```

### Audit Trail with Version

```typescript
@Entity()
export class Contract {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column('text')
  terms: string;

  @Column()
  partyAId: number;

  @Column()
  partyBId: number;

  @Column({ type: 'enum', enum: ['draft', 'active', 'expired', 'terminated'] })
  status: string;

  @VersionColumn()
  version: number;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  lastModified: Date;

  @Column({ nullable: true })
  lastModifiedBy: number | null;
}

@Entity()
export class ContractVersion {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  contractId: number;

  @Column()
  versionNumber: number;

  @Column('text')
  terms: string;

  @Column()
  status: string;

  @Column()
  modifiedBy: number;

  @CreateDateColumn()
  modifiedAt: Date;
}

// Service to track version history
class ContractService {
  async updateContract(
    contractId: number,
    userId: number,
    updates: Partial<Contract>
  ): Promise<Contract> {
    const contract = await contractRepository.findOneBy({ id: contractId });

    if (!contract) {
      throw new Error('Contract not found');
    }

    // Save current version to history
    const versionSnapshot = new ContractVersion();
    versionSnapshot.contractId = contract.id;
    versionSnapshot.versionNumber = contract.version;
    versionSnapshot.terms = contract.terms;
    versionSnapshot.status = contract.status;
    versionSnapshot.modifiedBy = userId;

    await contractVersionRepository.save(versionSnapshot);

    // Apply updates
    Object.assign(contract, updates);
    contract.lastModifiedBy = userId;

    // Save (version auto-incremented)
    await contractRepository.save(contract);

    return contract;
  }

  async getVersionHistory(contractId: number): Promise<ContractVersion[]> {
    return await contractVersionRepository.find({
      where: { contractId },
      order: { versionNumber: 'DESC' },
    });
  }

  async revertToVersion(
    contractId: number,
    versionNumber: number,
    userId: number
  ): Promise<Contract> {
    const versionSnapshot = await contractVersionRepository.findOne({
      where: { contractId, versionNumber },
    });

    if (!versionSnapshot) {
      throw new Error('Version not found');
    }

    return await this.updateContract(contractId, userId, {
      terms: versionSnapshot.terms,
      status: versionSnapshot.status,
    });
  }
}
```

### Migration for Version Column

```typescript
import { MigrationInterface, QueryRunner, TableColumn } from 'typeorm';

export class AddVersionColumn1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.addColumn(
      'products',
      new TableColumn({
        name: 'version',
        type: 'int',
        default: 1,
        isNullable: false,
      })
    );

    // Set initial version for existing records
    await queryRunner.query(`
      UPDATE products SET version = 1 WHERE version IS NULL
    `);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropColumn('products', 'version');
  }
}
```

### Best Practices

```typescript
// ✅ GOOD: Use @VersionColumn for concurrent updates
@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column('int')
  stock: number;

  @VersionColumn() // Prevents lost updates
  version: number;
}

// ✅ GOOD: Implement retry logic for version conflicts
async function updateWithRetry(entity: any): Promise<any> {
  const maxRetries = 3;
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await repository.save(entity);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      // Reload and retry
      const fresh = await repository.findOneBy({ id: entity.id });
      Object.assign(entity, fresh);
    }
  }
}

// ✅ GOOD: Use version column for critical data (inventory, balances, etc.)
@Entity()
export class Inventory {
  @PrimaryGeneratedColumn()
  id: number;

  @Column('int')
  quantity: number;

  @VersionColumn() // Critical for accurate inventory
  version: number;
}

// ❌ BAD: Don't manually modify version column
const product = await repository.findOneBy({ id: 1 });
product.version = 999; // Don't do this! Let TypeORM manage it

// ❌ BAD: Don't ignore version conflicts
try {
  await repository.save(entity);
} catch (error) {
  // Ignoring the error loses data integrity
}

// ✅ GOOD: Handle version conflicts gracefully
try {
  await repository.save(entity);
} catch (error) {
  if (error.message.includes('version')) {
    // Reload entity and ask user to resolve conflict
    const fresh = await repository.findOneBy({ id: entity.id });
    // Present both versions to user
  }
}
```

**Interview Tip**: Explain that `@VersionColumn` implements optimistic locking by automatically incrementing a version number on every update. Emphasize the problem it solves: preventing lost updates when multiple users/processes modify the same entity concurrently. Describe the mechanism: when saving, TypeORM checks if the version matches the database version; if not, the update fails. Discuss use cases: critical data like inventory, financial balances, documents, contracts - anywhere concurrent modifications could cause data loss. Mention handling conflicts: implement retry logic with exponential backoff, or present conflict to user for manual resolution. Contrast with pessimistic locking: optimistic locking doesn't hold database locks, better for web applications with high concurrency. For production: always use on entities with concurrent access, implement proper error handling, consider keeping version history for audit trails. A strong answer demonstrates understanding of concurrency problems and practical conflict resolution strategies.

</details>

<details>
<summary>24. How do you define unique constraints in TypeORM?</summary>

Unique constraints ensure that column values are unique across all rows in a table. TypeORM provides multiple ways to define unique constraints at the column level and entity level.

### Column-Level Unique Constraint

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Simple unique constraint on single column
  @Column({ unique: true })
  email: string;

  // Another unique column
  @Column({ unique: true })
  username: string;

  @Column()
  password: string;
}

// Database creates unique indexes on email and username columns
// INSERT will fail if duplicate email or username is provided
```

### Entity-Level Unique Constraint

```typescript
import { Entity, PrimaryGeneratedColumn, Column, Unique } from 'typeorm';

// Single unique constraint on one column
@Entity()
@Unique(['email'])
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  email: string;

  @Column()
  username: string;
}

// Multiple separate unique constraints
@Entity()
@Unique(['email'])
@Unique(['username'])
export class UserV2 {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  email: string;

  @Column()
  username: string;
}
```

### Composite Unique Constraint

```typescript
// Unique constraint on multiple columns together
@Entity()
@Unique(['email', 'provider']) // Combination must be unique
export class SocialAccount {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  userId: number;

  @Column()
  email: string;

  @Column()
  provider: string; // 'google', 'facebook', 'github'

  @Column()
  providerId: string;
}

// Same email can exist for different providers
// john@example.com + google: OK
// john@example.com + facebook: OK
// john@example.com + google: DUPLICATE (not allowed)

// Usage
const account1 = new SocialAccount();
account1.email = 'john@example.com';
account1.provider = 'google';
await repository.save(account1); // Success

const account2 = new SocialAccount();
account2.email = 'john@example.com';
account2.provider = 'facebook';
await repository.save(account2); // Success (different provider)

const account3 = new SocialAccount();
account3.email = 'john@example.com';
account3.provider = 'google';
await repository.save(account3); // Error: duplicate key (email + provider)
```

### Named Unique Constraints

```typescript
import { Entity, PrimaryGeneratedColumn, Column, Unique } from 'typeorm';

// Named unique constraint (useful for error handling)
@Entity()
@Unique('UQ_USER_EMAIL', ['email'])
@Unique('UQ_USER_USERNAME', ['username'])
@Unique('UQ_USER_PHONE', ['phone', 'countryCode'])
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  email: string;

  @Column()
  username: string;

  @Column()
  phone: string;

  @Column()
  countryCode: string;
}

// Named constraints help identify which constraint was violated
try {
  await userRepository.save(user);
} catch (error) {
  if (error.constraint === 'UQ_USER_EMAIL') {
    throw new Error('Email already exists');
  } else if (error.constraint === 'UQ_USER_USERNAME') {
    throw new Error('Username already taken');
  } else if (error.constraint === 'UQ_USER_PHONE') {
    throw new Error('Phone number already registered');
  }
}
```

### Multiple Composite Unique Constraints

```typescript
@Entity()
@Unique(['tenantId', 'email']) // Email unique per tenant
@Unique(['tenantId', 'username']) // Username unique per tenant
@Unique(['tenantId', 'employeeId']) // Employee ID unique per tenant
export class Employee {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  tenantId: number; // Multi-tenant application

  @Column()
  email: string;

  @Column()
  username: string;

  @Column()
  employeeId: string;

  @Column()
  name: string;

  @Column()
  department: string;
}

// Each tenant can have users with same email/username/employeeId
// But within a tenant, these must be unique

// Tenant 1:
// - john@example.com (OK)
// - jane@example.com (OK)

// Tenant 2:
// - john@example.com (OK - different tenant)
// - jane@example.com (OK - different tenant)
```

### Unique Constraint with Nullable Columns

```typescript
@Entity()
@Unique(['phoneNumber']) // NULL values are allowed and don't violate uniqueness
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  email: string;

  @Column({ nullable: true })
  phoneNumber: string | null;

  @Column({ nullable: true })
  socialSecurityNumber: string | null;
}

// Multiple rows can have NULL phone numbers
// But non-NULL phone numbers must be unique

const user1 = new User();
user1.email = 'john@example.com';
user1.phoneNumber = null;
await repository.save(user1); // OK

const user2 = new User();
user2.email = 'jane@example.com';
user2.phoneNumber = null;
await repository.save(user2); // OK (NULL doesn't violate uniqueness)

const user3 = new User();
user3.email = 'bob@example.com';
user3.phoneNumber = '+1234567890';
await repository.save(user3); // OK

const user4 = new User();
user4.email = 'alice@example.com';
user4.phoneNumber = '+1234567890';
await repository.save(user4); // Error: duplicate phone number
```

### Partial Unique Constraint (PostgreSQL)

```typescript
// For complex unique constraints, use raw SQL in migrations
@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  sku: string;

  @Column({ default: false })
  deleted: boolean;

  @DeleteDateColumn()
  deletedAt: Date | null;
}

// Migration: Unique only for non-deleted products
export class AddPartialUniqueConstraint1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    // PostgreSQL: Unique where deleted = false
    await queryRunner.query(`
      CREATE UNIQUE INDEX idx_product_sku_active 
      ON product (sku) 
      WHERE deleted = false
    `);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP INDEX idx_product_sku_active`);
  }
}

// Now: SKU must be unique among active products
// Deleted products can have duplicate SKUs
```

### Case-Insensitive Unique Constraint

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Column-level unique (case-sensitive by default)
  @Column({ unique: true })
  email: string;

  @Column()
  username: string;
}

// For case-insensitive uniqueness, use migration
export class AddCaseInsensitiveUnique1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    // Remove default unique constraint
    await queryRunner.query(`
      ALTER TABLE "user" DROP CONSTRAINT IF EXISTS "UQ_user_email"
    `);

    // Create case-insensitive unique index (PostgreSQL)
    await queryRunner.query(`
      CREATE UNIQUE INDEX idx_user_email_ci 
      ON "user" (LOWER(email))
    `);

    // MySQL: use UNIQUE with case-insensitive collation
    // CREATE UNIQUE INDEX idx_user_email_ci 
    // ON user (email) 
    // COLLATE utf8mb4_unicode_ci
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP INDEX idx_user_email_ci`);
  }
}
```

### Handling Unique Constraint Violations

```typescript
import { QueryFailedError } from 'typeorm';

class UserService {
  async createUser(email: string, username: string): Promise<User> {
    const user = new User();
    user.email = email;
    user.username = username;

    try {
      return await userRepository.save(user);
    } catch (error) {
      if (error instanceof QueryFailedError) {
        // PostgreSQL error code for unique violation
        if (error.driverError.code === '23505') {
          // Parse which constraint failed
          const detail = error.driverError.detail || '';

          if (detail.includes('email')) {
            throw new Error('Email already exists');
          } else if (detail.includes('username')) {
            throw new Error('Username already taken');
          }

          throw new Error('Duplicate value');
        }
      }

      throw error;
    }
  }

  // Check existence before insert
  async registerUser(email: string, username: string): Promise<User> {
    // Check if email exists
    const existingEmail = await userRepository.findOne({
      where: { email },
    });

    if (existingEmail) {
      throw new Error('Email already registered');
    }

    // Check if username exists
    const existingUsername = await userRepository.findOne({
      where: { username },
    });

    if (existingUsername) {
      throw new Error('Username already taken');
    }

    // Create user
    const user = new User();
    user.email = email;
    user.username = username;

    return await userRepository.save(user);
  }
}
```

### Upsert with Unique Constraints

```typescript
@Entity()
@Unique(['email'])
export class Subscriber {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  email: string;

  @Column({ default: true })
  active: boolean;

  @Column({ type: 'int', default: 0 })
  subscriptionCount: number;

  @UpdateDateColumn()
  lastUpdated: Date;
}

// Upsert: Insert or update if exists
async function subscribeEmail(email: string): Promise<Subscriber> {
  // Try to find existing
  let subscriber = await subscriberRepository.findOne({
    where: { email },
  });

  if (subscriber) {
    // Update existing
    subscriber.active = true;
    subscriber.subscriptionCount += 1;
  } else {
    // Create new
    subscriber = new Subscriber();
    subscriber.email = email;
    subscriber.active = true;
    subscriber.subscriptionCount = 1;
  }

  return await subscriberRepository.save(subscriber);
}

// Using query builder upsert (PostgreSQL)
async function upsertSubscriber(email: string): Promise<void> {
  await subscriberRepository
    .createQueryBuilder()
    .insert()
    .into(Subscriber)
    .values({ email, active: true, subscriptionCount: 1 })
    .orUpdate(['active', 'subscriptionCount'], ['email'])
    .execute();
}
```

### Real-World Example: Multi-Tenant Application

```typescript
@Entity()
@Unique('UQ_TENANT_USER_EMAIL', ['tenantId', 'email'])
@Unique('UQ_TENANT_USER_USERNAME', ['tenantId', 'username'])
@Unique('UQ_TENANT_USER_EMPLOYEE_ID', ['tenantId', 'employeeId'])
export class TenantUser {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  tenantId: number;

  @Column()
  email: string;

  @Column()
  username: string;

  @Column({ nullable: true })
  employeeId: string | null;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column()
  role: string;

  @Column({ default: true })
  active: boolean;

  @CreateDateColumn()
  createdAt: Date;
}

@Entity()
@Unique(['domain']) // Domain must be globally unique
@Unique(['slug']) // Slug must be globally unique
export class Tenant {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  domain: string; // e.g., "acme.myapp.com"

  @Column()
  slug: string; // e.g., "acme"

  @Column({ default: true })
  active: boolean;

  @CreateDateColumn()
  createdAt: Date;
}

// Service
class TenantUserService {
  async createUser(
    tenantId: number,
    email: string,
    username: string,
    employeeId: string | null
  ): Promise<TenantUser> {
    const user = new TenantUser();
    user.tenantId = tenantId;
    user.email = email;
    user.username = username;
    user.employeeId = employeeId;

    try {
      return await tenantUserRepository.save(user);
    } catch (error) {
      if (error instanceof QueryFailedError) {
        if (error.driverError.code === '23505') {
          const constraint = error.driverError.constraint;

          switch (constraint) {
            case 'UQ_TENANT_USER_EMAIL':
              throw new Error('Email already exists in this organization');
            case 'UQ_TENANT_USER_USERNAME':
              throw new Error('Username already taken in this organization');
            case 'UQ_TENANT_USER_EMPLOYEE_ID':
              throw new Error('Employee ID already exists in this organization');
            default:
              throw new Error('Duplicate value');
          }
        }
      }

      throw error;
    }
  }

  async isEmailAvailable(tenantId: number, email: string): Promise<boolean> {
    const count = await tenantUserRepository.count({
      where: { tenantId, email },
    });
    return count === 0;
  }

  async isUsernameAvailable(
    tenantId: number,
    username: string
  ): Promise<boolean> {
    const count = await tenantUserRepository.count({
      where: { tenantId, username },
    });
    return count === 0;
  }
}
```

### Soft Delete with Unique Constraints

```typescript
@Entity()
@Unique(['email']) // Problem: deleted users block email reuse
export class UserProblem {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  email: string;

  @DeleteDateColumn()
  deletedAt: Date | null;
}

// Solution 1: Composite unique with deletedAt NULL
@Entity()
export class UserSolution1 {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  email: string;

  @DeleteDateColumn()
  deletedAt: Date | null;
}

// Migration: Partial unique index (PostgreSQL)
export class AddPartialEmailUnique1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
      CREATE UNIQUE INDEX idx_user_email_active 
      ON user_solution1 (email) 
      WHERE deleted_at IS NULL
    `);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP INDEX idx_user_email_active`);
  }
}

// Solution 2: Append ID to email on soft delete
@Entity()
export class UserSolution2 {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ unique: true })
  email: string;

  @Column()
  originalEmail: string;

  @DeleteDateColumn()
  deletedAt: Date | null;

  @BeforeRemove()
  appendIdToEmail() {
    this.originalEmail = this.email;
    this.email = `${this.email}_deleted_${this.id}`;
  }
}
```

### Migration for Unique Constraints

```typescript
import {
  MigrationInterface,
  QueryRunner,
  Table,
  TableIndex,
  TableUnique,
} from 'typeorm';

export class CreateUserTable1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    // Create table with unique constraints
    await queryRunner.createTable(
      new Table({
        name: 'user',
        columns: [
          {
            name: 'id',
            type: 'int',
            isPrimary: true,
            isGenerated: true,
            generationStrategy: 'increment',
          },
          {
            name: 'email',
            type: 'varchar',
          },
          {
            name: 'username',
            type: 'varchar',
          },
          {
            name: 'phone',
            type: 'varchar',
            isNullable: true,
          },
        ],
        uniques: [
          new TableUnique({
            name: 'UQ_USER_EMAIL',
            columnNames: ['email'],
          }),
          new TableUnique({
            name: 'UQ_USER_USERNAME',
            columnNames: ['username'],
          }),
        ],
      })
    );
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropTable('user');
  }
}

// Add unique constraint to existing table
export class AddEmailUnique1234567891 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.createUniqueConstraint(
      'user',
      new TableUnique({
        name: 'UQ_USER_EMAIL',
        columnNames: ['email'],
      })
    );
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropUniqueConstraint('user', 'UQ_USER_EMAIL');
  }
}
```

### Best Practices

```typescript
// ✅ GOOD: Use column-level unique for simple cases
@Column({ unique: true })
email: string;

// ✅ GOOD: Use entity-level unique for composite constraints
@Entity()
@Unique(['tenantId', 'email'])
export class User { }

// ✅ GOOD: Name your constraints for better error handling
@Entity()
@Unique('UQ_USER_EMAIL', ['email'])
@Unique('UQ_USER_USERNAME', ['username'])
export class User { }

// ✅ GOOD: Handle unique violations gracefully
try {
  await repository.save(user);
} catch (error) {
  if (error.code === '23505') { // PostgreSQL unique violation
    throw new Error('Email already exists');
  }
}

// ✅ GOOD: Check existence before insert (better UX)
const exists = await repository.findOne({ where: { email } });
if (exists) {
  throw new Error('Email already registered');
}

// ✅ GOOD: Use partial indexes for soft deletes (PostgreSQL)
// CREATE UNIQUE INDEX idx_email ON users (email) WHERE deleted_at IS NULL

// ❌ BAD: Don't rely only on database errors
// Check before insert for better user experience

// ❌ BAD: Don't use unique on columns that will be updated frequently
// Consider the use case before adding unique constraint

// ✅ GOOD: Use case-insensitive unique for emails
// CREATE UNIQUE INDEX idx_email ON users (LOWER(email))

// ✅ GOOD: Multiple nulls are allowed in unique columns
@Column({ unique: true, nullable: true })
phoneNumber: string | null; // Multiple NULL values OK
```

**Interview Tip**: Explain that TypeORM supports unique constraints at column level (`@Column({ unique: true })`) and entity level (`@Unique(['column'])`). Emphasize composite unique constraints: `@Unique(['col1', 'col2'])` ensures the combination is unique, not individual columns. Discuss naming constraints for better error handling: `@Unique('UQ_USER_EMAIL', ['email'])` helps identify which constraint failed. Mention NULL handling: multiple NULL values don't violate uniqueness in most databases. Highlight practical considerations: check existence before insert for better UX, use partial indexes for soft deletes (PostgreSQL), implement case-insensitive uniqueness for emails. For multi-tenant apps: use composite constraints like `@Unique(['tenantId', 'email'])` to scope uniqueness per tenant. A strong answer demonstrates understanding of both technical implementation and practical database design patterns.

</details>

<details>
<summary>25. What is the purpose of the @Index decorator?</summary>

The `@Index` decorator creates database indexes to improve query performance. Indexes speed up data retrieval operations but add overhead to insert/update/delete operations and consume additional storage space.

### Basic Index on Single Column

```typescript
import { Entity, PrimaryGeneratedColumn, Column, Index } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Create index on email column
  @Index()
  @Column()
  email: string;

  // Create index on username column
  @Index()
  @Column()
  username: string;

  @Column()
  password: string;
}

// Database creates indexes:
// - idx_user_email on email column
// - idx_user_username on username column

// Queries on indexed columns are faster:
const user = await userRepository.findOne({ where: { email: 'john@example.com' } });
// Uses index for fast lookup instead of full table scan
```

### Named Indexes

```typescript
@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Named index
  @Index('IDX_PRODUCT_SKU')
  @Column()
  sku: string;

  @Index('IDX_PRODUCT_CATEGORY')
  @Column()
  category: string;

  @Column('decimal', { precision: 10, scale: 2 })
  price: number;
}

// Named indexes are easier to identify and manage
// ALTER TABLE product ADD INDEX IDX_PRODUCT_SKU (sku)
```

### Entity-Level Indexes

```typescript
import { Entity, PrimaryGeneratedColumn, Column, Index } from 'typeorm';

// Single column index at entity level
@Entity()
@Index(['email'])
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  email: string;
}

// Multiple indexes at entity level
@Entity()
@Index(['email'])
@Index(['username'])
@Index(['createdAt'])
export class UserV2 {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  email: string;

  @Column()
  username: string;

  @CreateDateColumn()
  createdAt: Date;
}
```

### Composite Indexes

```typescript
// Composite index on multiple columns
@Entity()
@Index(['lastName', 'firstName']) // Index on last name + first name combination
export class Employee {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column()
  email: string;

  @Column()
  department: string;
}

// Speeds up queries like:
const employees = await employeeRepository.find({
  where: { lastName: 'Smith', firstName: 'John' },
  // Uses composite index for fast lookup
});

// Also speeds up queries on leftmost column:
const smiths = await employeeRepository.find({
  where: { lastName: 'Smith' },
  // Can use the composite index (leftmost prefix)
});

// But won't help queries only on firstName:
const johns = await employeeRepository.find({
  where: { firstName: 'John' },
  // Cannot use composite index (not leftmost)
});
```

### Named Composite Indexes

```typescript
@Entity()
@Index('IDX_USER_TENANT_EMAIL', ['tenantId', 'email'])
@Index('IDX_USER_TENANT_STATUS', ['tenantId', 'status'])
@Index('IDX_USER_CREATED', ['createdAt'])
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  tenantId: number;

  @Column()
  email: string;

  @Column()
  status: string;

  @CreateDateColumn()
  createdAt: Date;
}

// Optimizes queries:
// 1. Find user by tenant and email (uses IDX_USER_TENANT_EMAIL)
const user = await userRepository.findOne({
  where: { tenantId: 1, email: 'john@example.com' },
});

// 2. Find active users in tenant (uses IDX_USER_TENANT_STATUS)
const activeUsers = await userRepository.find({
  where: { tenantId: 1, status: 'active' },
});

// 3. Find recent users (uses IDX_USER_CREATED)
const recentUsers = await userRepository
  .createQueryBuilder('user')
  .where('user.createdAt > :date', { date: new Date('2024-01-01') })
  .getMany();
```

### Index Options

```typescript
@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Unique index (combines index with unique constraint)
  @Index({ unique: true })
  @Column()
  sku: string;

  // Named unique index
  @Index('IDX_PRODUCT_BARCODE', { unique: true })
  @Column()
  barcode: string;

  // Spatial index (for geographic data)
  @Index({ spatial: true })
  @Column('point')
  location: string;

  // Full-text index (MySQL)
  @Index({ fulltext: true })
  @Column('text')
  description: string;
}
```

### Composite Index with Ordering

```typescript
@Entity()
@Index('IDX_ORDER_CUSTOMER_DATE', ['customerId', 'createdAt'])
export class Order {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  customerId: number;

  @Column('decimal', { precision: 10, scale: 2 })
  total: number;

  @CreateDateColumn()
  createdAt: Date;
}

// Optimizes queries with ORDER BY:
const customerOrders = await orderRepository
  .createQueryBuilder('order')
  .where('order.customerId = :customerId', { customerId: 123 })
  .orderBy('order.createdAt', 'DESC')
  .getMany();
// Uses index for both WHERE and ORDER BY
```

### Partial Indexes (PostgreSQL)

```typescript
@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  status: string;

  @Column('decimal', { precision: 10, scale: 2 })
  price: number;

  @DeleteDateColumn()
  deletedAt: Date | null;
}

// Partial index in migration (PostgreSQL)
export class AddPartialIndexes1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    // Index only active products
    await queryRunner.query(`
      CREATE INDEX idx_product_active 
      ON product (status) 
      WHERE deleted_at IS NULL AND status = 'active'
    `);

    // Index only expensive products
    await queryRunner.query(`
      CREATE INDEX idx_product_expensive 
      ON product (price) 
      WHERE price > 1000
    `);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP INDEX idx_product_active`);
    await queryRunner.query(`DROP INDEX idx_product_expensive`);
  }
}
```

### Expression Indexes (PostgreSQL)

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  email: string;

  @Column()
  firstName: string;

  @Column()
  lastName: string;
}

// Expression index in migration
export class AddExpressionIndexes1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    // Case-insensitive email index
    await queryRunner.query(`
      CREATE INDEX idx_user_email_lower 
      ON "user" (LOWER(email))
    `);

    // Full name index
    await queryRunner.query(`
      CREATE INDEX idx_user_full_name 
      ON "user" ((first_name || ' ' || last_name))
    `);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP INDEX idx_user_email_lower`);
    await queryRunner.query(`DROP INDEX idx_user_full_name`);
  }
}

// Queries using expression indexes:
const users = await userRepository
  .createQueryBuilder('user')
  .where('LOWER(user.email) = LOWER(:email)', {
    email: 'JOHN@EXAMPLE.COM',
  })
  .getMany();
```

### Full-Text Indexes

```typescript
@Entity()
export class Article {
  @PrimaryGeneratedColumn()
  id: number;

  // Full-text index on title
  @Index({ fulltext: true })
  @Column()
  title: string;

  // Full-text index on content
  @Index({ fulltext: true })
  @Column('text')
  content: string;

  @Column()
  authorId: number;

  @CreateDateColumn()
  publishedAt: Date;
}

// Full-text search query (MySQL)
const articles = await articleRepository
  .createQueryBuilder('article')
  .where('MATCH(article.title, article.content) AGAINST(:search IN BOOLEAN MODE)', {
    search: '+typescript +database',
  })
  .getMany();

// PostgreSQL full-text search
@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column('text')
  content: string;

  // tsvector column for full-text search
  @Index()
  @Column({
    type: 'tsvector',
    generatedType: 'STORED',
    asExpression: "to_tsvector('english', title || ' ' || content)",
  })
  searchVector: string;
}

// PostgreSQL full-text query
const posts = await postRepository
  .createQueryBuilder('post')
  .where('post.searchVector @@ to_tsquery(:query)', {
    query: 'typescript & database',
  })
  .getMany();
```

### Covering Indexes

```typescript
@Entity()
@Index('IDX_ORDER_CUSTOMER_TOTAL', ['customerId', 'total', 'status'])
export class Order {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  customerId: number;

  @Column('decimal', { precision: 10, scale: 2 })
  total: number;

  @Column()
  status: string;

  @CreateDateColumn()
  createdAt: Date;
}

// Covering index includes all columns needed for the query
// Database can satisfy query entirely from the index without accessing the table
const totalSpent = await orderRepository
  .createQueryBuilder('order')
  .select('SUM(order.total)', 'total')
  .where('order.customerId = :customerId', { customerId: 123 })
  .andWhere('order.status = :status', { status: 'completed' })
  .getRawOne();
// All needed columns (customerId, total, status) are in the index
```

### Real-World Example: E-commerce Application

```typescript
@Entity()
@Index('IDX_PRODUCT_CATEGORY_PRICE', ['category', 'price'])
@Index('IDX_PRODUCT_BRAND_PRICE', ['brand', 'price'])
@Index('IDX_PRODUCT_SEARCH', ['name', 'status'])
@Index('IDX_PRODUCT_CREATED', ['createdAt'])
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Index()
  @Column()
  sku: string;

  @Column()
  category: string;

  @Column()
  brand: string;

  @Column('decimal', { precision: 10, scale: 2 })
  price: number;

  @Column({ type: 'int', default: 0 })
  stock: number;

  @Column({
    type: 'enum',
    enum: ['draft', 'active', 'discontinued'],
    default: 'active',
  })
  status: string;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}

@Entity()
@Index('IDX_ORDER_CUSTOMER_STATUS', ['customerId', 'status'])
@Index('IDX_ORDER_CUSTOMER_DATE', ['customerId', 'createdAt'])
@Index('IDX_ORDER_STATUS_DATE', ['status', 'createdAt'])
export class Order {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  orderNumber: string;

  @Column()
  customerId: number;

  @Column('decimal', { precision: 10, scale: 2 })
  total: number;

  @Column({
    type: 'enum',
    enum: ['pending', 'paid', 'shipped', 'delivered', 'cancelled'],
  })
  status: string;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}

// Optimized queries using indexes:

// 1. Find products by category, sorted by price
const products = await productRepository
  .createQueryBuilder('product')
  .where('product.category = :category', { category: 'Electronics' })
  .andWhere('product.status = :status', { status: 'active' })
  .orderBy('product.price', 'ASC')
  .getMany();
// Uses IDX_PRODUCT_CATEGORY_PRICE

// 2. Find customer orders by status
const customerOrders = await orderRepository
  .createQueryBuilder('order')
  .where('order.customerId = :customerId', { customerId: 123 })
  .andWhere('order.status = :status', { status: 'pending' })
  .getMany();
// Uses IDX_ORDER_CUSTOMER_STATUS

// 3. Find recent customer orders
const recentOrders = await orderRepository
  .createQueryBuilder('order')
  .where('order.customerId = :customerId', { customerId: 123 })
  .orderBy('order.createdAt', 'DESC')
  .limit(10)
  .getMany();
// Uses IDX_ORDER_CUSTOMER_DATE

// 4. Find product by SKU
const product = await productRepository.findOne({
  where: { sku: 'PROD-12345' },
});
// Uses IDX on sku column
```

### Index Performance Monitoring

```typescript
// Service to analyze query performance
class QueryPerformanceService {
  async analyzeQuery(sql: string): Promise<any> {
    const queryRunner = AppDataSource.createQueryRunner();

    try {
      // PostgreSQL: EXPLAIN ANALYZE
      const result = await queryRunner.query(`EXPLAIN ANALYZE ${sql}`);
      return result;

      // MySQL: EXPLAIN
      // const result = await queryRunner.query(`EXPLAIN ${sql}`);
      // return result;
    } finally {
      await queryRunner.release();
    }
  }

  async checkIndexUsage(tableName: string): Promise<any> {
    const queryRunner = AppDataSource.createQueryRunner();

    try {
      // PostgreSQL: Check index usage
      const result = await queryRunner.query(`
        SELECT 
          schemaname,
          tablename,
          indexname,
          idx_scan as index_scans,
          idx_tup_read as tuples_read,
          idx_tup_fetch as tuples_fetched
        FROM pg_stat_user_indexes
        WHERE tablename = $1
        ORDER BY idx_scan DESC
      `, [tableName]);

      return result;
    } finally {
      await queryRunner.release();
    }
  }

  async findUnusedIndexes(): Promise<any> {
    const queryRunner = AppDataSource.createQueryRunner();

    try {
      // PostgreSQL: Find indexes that are never used
      const result = await queryRunner.query(`
        SELECT 
          schemaname,
          tablename,
          indexname,
          idx_scan as scans
        FROM pg_stat_user_indexes
        WHERE idx_scan = 0
        AND indexrelname NOT LIKE '%_pkey'
        ORDER BY tablename, indexname
      `);

      return result;
    } finally {
      await queryRunner.release();
    }
  }
}
```

### Index Maintenance

```typescript
// Migration to add indexes
export class AddProductIndexes1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    // Create single column index
    await queryRunner.query(`
      CREATE INDEX idx_product_sku ON product (sku)
    `);

    // Create composite index
    await queryRunner.query(`
      CREATE INDEX idx_product_category_price 
      ON product (category, price)
    `);

    // Create unique index
    await queryRunner.query(`
      CREATE UNIQUE INDEX idx_product_sku_unique 
      ON product (sku)
    `);

    // Create partial index (PostgreSQL)
    await queryRunner.query(`
      CREATE INDEX idx_product_active 
      ON product (status) 
      WHERE status = 'active'
    `);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP INDEX idx_product_sku`);
    await queryRunner.query(`DROP INDEX idx_product_category_price`);
    await queryRunner.query(`DROP INDEX idx_product_sku_unique`);
    await queryRunner.query(`DROP INDEX idx_product_active`);
  }
}

// Concurrent index creation (PostgreSQL - doesn't lock table)
export class AddIndexConcurrently1234567891 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
      CREATE INDEX CONCURRENTLY idx_order_customer 
      ON "order" (customer_id)
    `);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP INDEX idx_order_customer`);
  }
}
```

### Best Practices

```typescript
// ✅ GOOD: Index foreign keys
@Entity()
export class Order {
  @PrimaryGeneratedColumn()
  id: number;

  @Index()
  @Column()
  customerId: number; // Foreign key - should be indexed

  @Index()
  @Column()
  productId: number; // Foreign key - should be indexed
}

// ✅ GOOD: Index columns used in WHERE clauses
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Index()
  @Column()
  email: string; // Frequently used in WHERE

  @Index()
  @Column()
  status: string; // Frequently filtered
}

// ✅ GOOD: Composite index for common query patterns
@Entity()
@Index(['tenantId', 'userId', 'createdAt'])
export class AuditLog {
  // Query pattern: WHERE tenantId = ? AND userId = ? ORDER BY createdAt DESC
}

// ✅ GOOD: Index columns used in ORDER BY
@Entity()
@Index(['createdAt'])
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @CreateDateColumn()
  createdAt: Date; // Frequently sorted
}

// ❌ BAD: Don't over-index
@Entity()
export class User {
  @Index() @Column() field1: string;
  @Index() @Column() field2: string;
  @Index() @Column() field3: string;
  @Index() @Column() field4: string;
  @Index() @Column() field5: string;
  // Too many indexes slow down writes
}

// ❌ BAD: Don't index low-cardinality columns (few distinct values)
@Index()
@Column()
isActive: boolean; // Only 2 values - poor index selectivity

// ✅ GOOD: Use partial index instead for low-cardinality
// CREATE INDEX idx_active_users ON users (id) WHERE is_active = true

// ❌ BAD: Don't index small tables
// Tables with < 1000 rows usually don't benefit from indexes

// ✅ GOOD: Index columns used in JOIN conditions
@Entity()
export class OrderItem {
  @Index()
  @Column()
  orderId: number; // Used in JOIN with Order table

  @Index()
  @Column()
  productId: number; // Used in JOIN with Product table
}

// ✅ GOOD: Use covering indexes for frequently accessed columns
@Index(['customerId', 'status', 'total']) // All columns in SELECT
@Entity()
export class Order {
  @Column()
  customerId: number;

  @Column()
  status: string;

  @Column('decimal')
  total: number;
}

// ✅ GOOD: Monitor and remove unused indexes
// Run queries to check index usage statistics periodically

// ✅ GOOD: Consider index size
// Large indexes consume memory and disk space
// Balance query performance with storage costs
```

### Index Trade-offs

```typescript
/*
BENEFITS of Indexes:
- Faster SELECT queries (especially with WHERE, ORDER BY, JOIN)
- Enforces uniqueness (unique indexes)
- Speeds up MIN/MAX operations
- Improves JOIN performance

COSTS of Indexes:
- Slower INSERT/UPDATE/DELETE operations
- Additional disk space
- Memory usage for index cache
- Maintenance overhead

WHEN TO INDEX:
✅ Foreign key columns
✅ Columns frequently used in WHERE clauses
✅ Columns frequently used in ORDER BY
✅ Columns frequently used in JOIN conditions
✅ Columns with high cardinality (many distinct values)
✅ Large tables (> 1000 rows)

WHEN NOT TO INDEX:
❌ Small tables (< 1000 rows)
❌ Columns with low cardinality (few distinct values)
❌ Columns rarely used in queries
❌ Columns frequently updated
❌ Too many indexes on one table (diminishing returns)

COMPOSITE INDEX CONSIDERATIONS:
- Order matters: [A, B] helps queries on A or A+B, but not B alone
- Leftmost prefix rule: Can use index for leftmost columns
- More specific columns first if used independently
- Less specific but more frequently queried columns first
*/
```

**Interview Tip**: Explain that `@Index` creates database indexes to speed up queries, especially those with WHERE, ORDER BY, and JOIN clauses. Emphasize the trade-off: indexes improve read performance but slow down writes and consume storage. Discuss composite indexes: `@Index(['col1', 'col2'])` helps queries on col1 or col1+col2, but not col2 alone (leftmost prefix rule). Mention when to index: foreign keys, frequently filtered/sorted columns, large tables with high-cardinality columns. Highlight PostgreSQL features: partial indexes (index subset of rows), expression indexes (index computed values), full-text indexes. For production: monitor index usage, remove unused indexes, use `EXPLAIN` to analyze query plans, consider concurrent index creation to avoid locking. A strong answer demonstrates understanding of both performance benefits and costs, with practical knowledge of when and how to index effectively.

</details>