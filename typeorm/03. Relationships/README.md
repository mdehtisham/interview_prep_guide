## **Relationships**

<details>
<summary>26. What types of relationships does TypeORM support?</summary>

TypeORM supports four types of relationships that model how entities relate to each other in a relational database: **One-to-One**, **One-to-Many**, **Many-to-One**, and **Many-to-Many**.

### Overview of Relationship Types

```typescript
/*
1. ONE-TO-ONE (1:1)
   - One entity relates to exactly one instance of another entity
   - Example: User ←→ Profile (one user has one profile)
   
2. ONE-TO-MANY (1:N)
   - One entity relates to many instances of another entity
   - Example: User → Posts (one user has many posts)
   
3. MANY-TO-ONE (N:1)
   - Many entities relate to one instance of another entity
   - Example: Posts → User (many posts belong to one user)
   - Inverse of One-to-Many
   
4. MANY-TO-MANY (N:M)
   - Many entities relate to many instances of another entity
   - Example: Students ←→ Courses (students have many courses, courses have many students)
*/
```

### 1. One-to-One Relationship

```typescript
import { Entity, PrimaryGeneratedColumn, Column, OneToOne, JoinColumn } from 'typeorm';

// One user has one profile
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  email: string;

  // One-to-One relationship
  @OneToOne(() => Profile, profile => profile.user)
  @JoinColumn() // This side owns the relationship (has foreign key)
  profile: Profile;
}

@Entity()
export class Profile {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  bio: string;

  @Column()
  avatarUrl: string;

  // Inverse side of the relationship
  @OneToOne(() => User, user => user.profile)
  user: User;
}

// Database structure:
// users table: id, name, email, profileId (foreign key)
// profiles table: id, bio, avatarUrl
```

### 2. One-to-Many Relationship

```typescript
import { Entity, PrimaryGeneratedColumn, Column, OneToMany } from 'typeorm';

// One user has many posts
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // One-to-Many relationship
  @OneToMany(() => Post, post => post.author)
  posts: Post[];
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column('text')
  content: string;

  // This is automatically handled by @ManyToOne on the other side
  // No @JoinColumn needed here
}

// Database structure:
// users table: id, name
// posts table: id, title, content, authorId (foreign key to users)
```

### 3. Many-to-One Relationship

```typescript
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne } from 'typeorm';

// Many posts belong to one user
@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column('text')
  content: string;

  // Many-to-One relationship (inverse of One-to-Many)
  @ManyToOne(() => User, user => user.posts)
  author: User;
  // Foreign key (authorId) is automatically created
}

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Inverse side
  @OneToMany(() => Post, post => post.author)
  posts: Post[];
}

// Database structure:
// users table: id, name
// posts table: id, title, content, authorId (foreign key)
```

### 4. Many-to-Many Relationship

```typescript
import { Entity, PrimaryGeneratedColumn, Column, ManyToMany, JoinTable } from 'typeorm';

// Many students have many courses
@Entity()
export class Student {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Many-to-Many relationship
  @ManyToMany(() => Course, course => course.students)
  @JoinTable() // Owner side of the relationship
  courses: Course[];
}

@Entity()
export class Course {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  // Inverse side
  @ManyToMany(() => Student, student => student.courses)
  students: Student[];
}

// Database structure:
// students table: id, name
// courses table: id, title
// student_courses_course table: studentId, courseId (junction table)
```

### Real-World Example: E-commerce System

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
  CreateDateColumn,
} from 'typeorm';

// User entity
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column({ unique: true })
  email: string;

  // ONE-TO-ONE: User has one address
  @OneToOne(() => Address, address => address.user, { cascade: true })
  @JoinColumn()
  address: Address;

  // ONE-TO-MANY: User has many orders
  @OneToMany(() => Order, order => order.customer)
  orders: Order[];

  // ONE-TO-MANY: User has many reviews
  @OneToMany(() => Review, review => review.user)
  reviews: Review[];

  @CreateDateColumn()
  createdAt: Date;
}

// Address entity (One-to-One with User)
@Entity()
export class Address {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  street: string;

  @Column()
  city: string;

  @Column()
  zipCode: string;

  @Column()
  country: string;

  // Inverse side of One-to-One
  @OneToOne(() => User, user => user.address)
  user: User;
}

// Order entity
@Entity()
export class Order {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  orderNumber: string;

  @Column('decimal', { precision: 10, scale: 2 })
  total: number;

  // MANY-TO-ONE: Many orders belong to one customer
  @ManyToOne(() => User, user => user.orders)
  customer: User;

  // ONE-TO-MANY: Order has many order items
  @OneToMany(() => OrderItem, item => item.order, { cascade: true })
  items: OrderItem[];

  @CreateDateColumn()
  createdAt: Date;
}

// Product entity
@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column('decimal', { precision: 10, scale: 2 })
  price: number;

  // MANY-TO-ONE: Many products belong to one category
  @ManyToOne(() => Category, category => category.products)
  category: Category;

  // ONE-TO-MANY: Product has many order items
  @OneToMany(() => OrderItem, item => item.product)
  orderItems: OrderItem[];

  // ONE-TO-MANY: Product has many reviews
  @OneToMany(() => Review, review => review.product)
  reviews: Review[];

  // MANY-TO-MANY: Products have many tags
  @ManyToMany(() => Tag, tag => tag.products)
  @JoinTable()
  tags: Tag[];
}

// Category entity
@Entity()
export class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // ONE-TO-MANY: Category has many products
  @OneToMany(() => Product, product => product.category)
  products: Product[];
}

// OrderItem entity (junction with additional data)
@Entity()
export class OrderItem {
  @PrimaryGeneratedColumn()
  id: number;

  @Column('int')
  quantity: number;

  @Column('decimal', { precision: 10, scale: 2 })
  price: number; // Price at time of order

  // MANY-TO-ONE: Many items belong to one order
  @ManyToOne(() => Order, order => order.items)
  order: Order;

  // MANY-TO-ONE: Many items reference one product
  @ManyToOne(() => Product, product => product.orderItems)
  product: Product;
}

// Review entity
@Entity()
export class Review {
  @PrimaryGeneratedColumn()
  id: number;

  @Column('int')
  rating: number; // 1-5

  @Column('text')
  comment: string;

  // MANY-TO-ONE: Many reviews by one user
  @ManyToOne(() => User, user => user.reviews)
  user: User;

  // MANY-TO-ONE: Many reviews for one product
  @ManyToOne(() => Product, product => product.reviews)
  product: Product;

  @CreateDateColumn()
  createdAt: Date;
}

// Tag entity (Many-to-Many with Product)
@Entity()
export class Tag {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ unique: true })
  name: string;

  // MANY-TO-MANY: Tags belong to many products
  @ManyToMany(() => Product, product => product.tags)
  products: Product[];
}
```

### Usage Examples

```typescript
// Create entities with relationships
async function createUserWithOrder() {
  // Create user with address (One-to-One)
  const user = new User();
  user.name = 'John Doe';
  user.email = 'john@example.com';

  const address = new Address();
  address.street = '123 Main St';
  address.city = 'New York';
  address.zipCode = '10001';
  address.country = 'USA';

  user.address = address;

  await userRepository.save(user); // Cascade saves address too

  // Create order with items (One-to-Many)
  const order = new Order();
  order.orderNumber = 'ORD-2024-001';
  order.customer = user;

  const item1 = new OrderItem();
  item1.quantity = 2;
  item1.price = 29.99;
  // item1.product would be set to a Product entity

  const item2 = new OrderItem();
  item2.quantity = 1;
  item2.price = 49.99;

  order.items = [item1, item2];
  order.total = 109.97;

  await orderRepository.save(order); // Cascade saves items too
}

// Create Many-to-Many relationship
async function assignTagsToProduct() {
  const product = await productRepository.findOne({ where: { id: 1 } });
  const tags = await tagRepository.find({
    where: { name: In(['electronics', 'bestseller', 'sale']) },
  });

  product.tags = tags;
  await productRepository.save(product);
}

// Query with relationships
async function getUserWithOrders() {
  const user = await userRepository.findOne({
    where: { id: 1 },
    relations: ['address', 'orders', 'orders.items'],
  });

  console.log(user.name);
  console.log(user.address.city);
  console.log(user.orders.length);
  user.orders.forEach(order => {
    console.log(`Order ${order.orderNumber}: ${order.items.length} items`);
  });
}
```

### Relationship Comparison Table

```typescript
/*
┌─────────────────┬─────────────────┬──────────────────┬─────────────────────────────┐
│ Relationship    │ Decorator       │ Foreign Key      │ Example                     │
├─────────────────┼─────────────────┼──────────────────┼─────────────────────────────┤
│ One-to-One      │ @OneToOne       │ On owner side    │ User ←→ Profile             │
│                 │ @JoinColumn     │ (with @JoinCol)  │                             │
├─────────────────┼─────────────────┼──────────────────┼─────────────────────────────┤
│ One-to-Many     │ @OneToMany      │ On "many" side   │ User → Posts                │
│                 │                 │ (automatic)      │ (User has many Posts)       │
├─────────────────┼─────────────────┼──────────────────┼─────────────────────────────┤
│ Many-to-One     │ @ManyToOne      │ On "many" side   │ Posts → User                │
│                 │                 │ (automatic)      │ (Many Posts to one User)    │
├─────────────────┼─────────────────┼──────────────────┼─────────────────────────────┤
│ Many-to-Many    │ @ManyToMany     │ Junction table   │ Students ←→ Courses         │
│                 │ @JoinTable      │ (on owner side)  │                             │
└─────────────────┴─────────────────┴──────────────────┴─────────────────────────────┘

KEY POINTS:
- @JoinColumn: Required on ONE side of One-to-One (owner)
- @JoinTable: Required on ONE side of Many-to-Many (owner)
- Many-to-One automatically creates foreign key
- One-to-Many is inverse side (no foreign key)
- Owner side: Has @JoinColumn or @JoinTable
- Inverse side: References owner with second parameter
*/
```

### Choosing the Right Relationship

```typescript
// ✅ ONE-TO-ONE: When one entity exclusively belongs to another
// Examples: User ←→ Profile, User ←→ Settings, Employee ←→ Desk

// ✅ ONE-TO-MANY / MANY-TO-ONE: Most common relationship
// Examples: User → Posts, Category → Products, Order → Items

// ✅ MANY-TO-MANY: When entities can belong to multiple of each other
// Examples: Students ←→ Courses, Products ←→ Tags, Users ←→ Roles

// ❌ AVOID: Creating Many-to-Many when you need junction table data
// Instead: Create junction entity with Many-to-One relationships
// Example: Instead of Order ←→ Product, use Order → OrderItem ← Product
```

**Interview Tip**: Explain that TypeORM supports four relationship types: **One-to-One** (1:1), **One-to-Many** (1:N), **Many-to-One** (N:1), and **Many-to-Many** (N:M). Emphasize real-world examples: One-to-One like User-Profile, One-to-Many like User-Posts, Many-to-One is the inverse, Many-to-Many like Students-Courses. Highlight key decorators: `@OneToOne` needs `@JoinColumn` on owner side, `@ManyToMany` needs `@JoinTable` on owner side, `@ManyToOne` automatically creates foreign key. Mention that One-to-Many and Many-to-One are complementary (two sides of same relationship). Discuss when to use each: One-to-One for exclusive relationships, One-to-Many for hierarchical data, Many-to-Many for cross-referencing. Note that junction entities are better than Many-to-Many when you need additional data on the relationship. A strong answer demonstrates understanding of both technical implementation and proper database design patterns.

</details>

<details>
<summary>27. What is One-to-One relationship and How do you define a One-to-One relationship?</summary>

A **One-to-One relationship** means one entity instance relates to exactly one instance of another entity. Each row in table A corresponds to at most one row in table B, and vice versa.

### Basic One-to-One Relationship

```typescript
import { Entity, PrimaryGeneratedColumn, Column, OneToOne, JoinColumn } from 'typeorm';

// User entity (owner side - has foreign key)
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  email: string;

  // One-to-One relationship
  @OneToOne(() => Profile, profile => profile.user)
  @JoinColumn() // Marks this as the owner side (creates foreign key)
  profile: Profile;
}

// Profile entity (inverse side)
@Entity()
export class Profile {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  bio: string;

  @Column()
  website: string;

  @Column()
  avatarUrl: string;

  // Inverse side of the relationship
  @OneToOne(() => User, user => user.profile)
  user: User;
}

// Database structure:
// users table: id, name, email, profileId (foreign key)
// profiles table: id, bio, website, avatarUrl

// Usage
const user = new User();
user.name = 'John Doe';
user.email = 'john@example.com';

const profile = new Profile();
profile.bio = 'Software Developer';
profile.website = 'https://johndoe.com';
profile.avatarUrl = 'https://example.com/avatar.jpg';

user.profile = profile;

await userRepository.save(user); // Saves both user and profile
```

### Custom Foreign Key Column Name

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Custom foreign key column name
  @OneToOne(() => Profile, profile => profile.user)
  @JoinColumn({ name: 'user_profile_id' }) // Custom column name
  profile: Profile;
}

// Database: users table will have 'user_profile_id' instead of 'profileId'
```

### Bi-directional vs Uni-directional

```typescript
// Bi-directional (both sides know about each other)
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @OneToOne(() => Profile, profile => profile.user)
  @JoinColumn()
  profile: Profile;
}

@Entity()
export class Profile {
  @PrimaryGeneratedColumn()
  id: number;

  @OneToOne(() => User, user => user.profile)
  user: User; // Can access user from profile
}

// Uni-directional (only one side knows about the other)
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @OneToOne(() => Profile)
  @JoinColumn()
  profile: Profile;
}

@Entity()
export class Profile {
  @PrimaryGeneratedColumn()
  id: number;
  // No reference to User
}
```

### With Cascade Options

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Cascade: Automatically save/update/remove related entity
  @OneToOne(() => Profile, profile => profile.user, {
    cascade: true, // or ['insert', 'update']
    eager: false, // Don't auto-load with user
    nullable: true, // Profile is optional
  })
  @JoinColumn()
  profile: Profile;
}

// Usage
const user = new User();
user.name = 'John';

const profile = new Profile();
profile.bio = 'Developer';

user.profile = profile;

await userRepository.save(user); // Cascade saves profile automatically
```

### Real-World Example: User Settings

```typescript
import { Entity, PrimaryGeneratedColumn, Column, OneToOne, JoinColumn } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column({ unique: true })
  email: string;

  @Column()
  password: string;

  // One-to-One with Settings
  @OneToOne(() => UserSettings, settings => settings.user, {
    cascade: true,
    eager: true, // Auto-load settings with user
  })
  @JoinColumn()
  settings: UserSettings;

  @CreateDateColumn()
  createdAt: Date;
}

@Entity()
export class UserSettings {
  @PrimaryGeneratedColumn()
  id: number;

  // Notification settings
  @Column({ default: true })
  emailNotifications: boolean;

  @Column({ default: true })
  pushNotifications: boolean;

  @Column({ default: false })
  smsNotifications: boolean;

  // Display settings
  @Column({ default: 'light' })
  theme: string;

  @Column({ default: 'en' })
  language: string;

  @Column({ default: 'UTC' })
  timezone: string;

  // Privacy settings
  @Column({ default: true })
  profilePublic: boolean;

  @Column({ default: false })
  showEmail: boolean;

  @Column({ default: true })
  showActivity: boolean;

  // Inverse relationship
  @OneToOne(() => User, user => user.settings)
  user: User;

  @UpdateDateColumn()
  updatedAt: Date;
}

// Service
class UserService {
  async createUser(name: string, email: string, password: string): Promise<User> {
    const user = new User();
    user.name = name;
    user.email = email;
    user.password = password;

    // Create default settings
    const settings = new UserSettings();
    settings.emailNotifications = true;
    settings.theme = 'light';
    settings.language = 'en';

    user.settings = settings;

    // Cascade automatically saves settings
    return await userRepository.save(user);
  }

  async updateSettings(userId: number, newSettings: Partial<UserSettings>): Promise<User> {
    const user = await userRepository.findOne({
      where: { id: userId },
      relations: ['settings'],
    });

    if (!user) {
      throw new Error('User not found');
    }

    // Update settings
    Object.assign(user.settings, newSettings);

    return await userRepository.save(user);
  }
}
```

### Employee-Desk Assignment

```typescript
@Entity()
export class Employee {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column()
  employeeId: string;

  // One employee has one desk
  @OneToOne(() => Desk, desk => desk.employee, {
    cascade: true,
    nullable: true, // Desk is optional
  })
  @JoinColumn()
  desk: Desk | null;

  @Column()
  department: string;
}

@Entity()
export class Desk {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  deskNumber: string;

  @Column()
  floor: number;

  @Column()
  building: string;

  @Column({ default: false })
  hasMonitor: boolean;

  @Column({ default: false })
  hasKeyboard: boolean;

  // Inverse relationship
  @OneToOne(() => Employee, employee => employee.desk)
  employee: Employee;
}

// Usage
async function assignDeskToEmployee(employeeId: number, deskNumber: string) {
  const employee = await employeeRepository.findOne({
    where: { id: employeeId },
    relations: ['desk'],
  });

  const desk = await deskRepository.findOne({
    where: { deskNumber },
  });

  if (!desk) {
    throw new Error('Desk not found');
  }

  // Check if desk is already assigned
  const existingAssignment = await employeeRepository.findOne({
    where: { desk: { id: desk.id } },
  });

  if (existingAssignment) {
    throw new Error('Desk already assigned');
  }

  employee.desk = desk;
  await employeeRepository.save(employee);
}
```

### Querying One-to-One Relationships

```typescript
// Load with relations
const user = await userRepository.findOne({
  where: { id: 1 },
  relations: ['profile'],
});

console.log(user.profile.bio);

// Query builder with join
const users = await userRepository
  .createQueryBuilder('user')
  .leftJoinAndSelect('user.profile', 'profile')
  .where('user.email = :email', { email: 'john@example.com' })
  .getOne();

// Find users with profiles
const usersWithProfiles = await userRepository
  .createQueryBuilder('user')
  .innerJoinAndSelect('user.profile', 'profile')
  .getMany();

// Find users without profiles
const usersWithoutProfiles = await userRepository
  .createQueryBuilder('user')
  .leftJoinAndSelect('user.profile', 'profile')
  .where('profile.id IS NULL')
  .getMany();

// Update relationship
const user = await userRepository.findOne({
  where: { id: 1 },
  relations: ['profile'],
});

user.profile.bio = 'Updated bio';
await userRepository.save(user); // Updates profile

// Remove relationship
user.profile = null;
await userRepository.save(user); // Sets profileId to NULL
```

### Document-Metadata Example

```typescript
@Entity()
export class Document {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  title: string;

  @Column('text')
  content: string;

  @Column()
  authorId: number;

  // One-to-One with metadata
  @OneToOne(() => DocumentMetadata, metadata => metadata.document, {
    cascade: true,
    eager: true,
  })
  @JoinColumn()
  metadata: DocumentMetadata;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}

@Entity()
export class DocumentMetadata {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ type: 'int', default: 0 })
  viewCount: number;

  @Column({ type: 'int', default: 0 })
  downloadCount: number;

  @Column({ type: 'int', default: 0 })
  wordCount: number;

  @Column({ type: 'int', default: 0 })
  pageCount: number;

  @Column('simple-array', { nullable: true })
  tags: string[];

  @Column({ type: 'timestamp', nullable: true })
  lastViewedAt: Date | null;

  @Column({ type: 'timestamp', nullable: true })
  lastDownloadedAt: Date | null;

  @OneToOne(() => Document, document => document.metadata)
  document: Document;

  @UpdateDateColumn()
  updatedAt: Date;
}

// Service
class DocumentService {
  async createDocument(title: string, content: string, authorId: number): Promise<Document> {
    const document = new Document();
    document.title = title;
    document.content = content;
    document.authorId = authorId;

    // Create metadata
    const metadata = new DocumentMetadata();
    metadata.wordCount = content.split(/\s+/).length;
    metadata.pageCount = Math.ceil(metadata.wordCount / 500); // 500 words per page
    metadata.tags = [];

    document.metadata = metadata;

    return await documentRepository.save(document);
  }

  async trackView(documentId: string): Promise<void> {
    const document = await documentRepository.findOne({
      where: { id: documentId },
      relations: ['metadata'],
    });

    if (document) {
      document.metadata.viewCount++;
      document.metadata.lastViewedAt = new Date();
      await documentRepository.save(document);
    }
  }

  async trackDownload(documentId: string): Promise<void> {
    const document = await documentRepository.findOne({
      where: { id: documentId },
      relations: ['metadata'],
    });

    if (document) {
      document.metadata.downloadCount++;
      document.metadata.lastDownloadedAt = new Date();
      await documentRepository.save(document);
    }
  }
}
```

### Handling Orphaned Records

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // With onDelete: 'CASCADE'
  @OneToOne(() => Profile, profile => profile.user, {
    cascade: true,
    onDelete: 'CASCADE', // Delete profile when user is deleted
  })
  @JoinColumn()
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

// When user is deleted, profile is also deleted
await userRepository.remove(user);
// Profile associated with this user is automatically deleted
```

### Best Practices

```typescript
// ✅ GOOD: Use cascade for dependent entities
@OneToOne(() => Profile, profile => profile.user, {
  cascade: true, // Auto-save profile with user
})
@JoinColumn()
profile: Profile;

// ✅ GOOD: Use nullable for optional relationships
@OneToOne(() => Profile, profile => profile.user, {
  nullable: true, // Profile is optional
})
@JoinColumn()
profile: Profile | null;

// ✅ GOOD: Use eager loading for frequently accessed relations
@OneToOne(() => Profile, profile => profile.user, {
  eager: true, // Always load profile with user
})
@JoinColumn()
profile: Profile;

// ✅ GOOD: Always specify inverse side for bi-directional
@OneToOne(() => Profile, profile => profile.user) // Second parameter is inverse
@JoinColumn()
profile: Profile;

// ❌ BAD: Don't put @JoinColumn on both sides
// Only owner side should have @JoinColumn

// ❌ BAD: Don't forget @JoinColumn on owner side
@OneToOne(() => Profile)
profile: Profile; // Missing @JoinColumn - won't work!

// ✅ GOOD: Choose owner side logically
// Usually: parent entity owns child (User owns Profile)
// Not the other way around
```

**Interview Tip**: Explain that One-to-One means each entity instance relates to exactly one instance of another entity. Emphasize `@JoinColumn` marks the owner side (creates foreign key), inverse side doesn't need it. Discuss bi-directional vs uni-directional: bi-directional allows navigation both ways, uni-directional only one way. Mention cascade options: `cascade: true` auto-saves related entities, useful for dependent data like User-Settings. Highlight real-world examples: User-Profile, Employee-Desk, Document-Metadata - cases where one entity exclusively belongs to another. For production: use `nullable: true` for optional relationships, `eager: true` for frequently accessed relations, `onDelete: 'CASCADE'` to prevent orphaned records. A strong answer demonstrates understanding of both technical implementation and when One-to-One is the appropriate relationship type.

</details>

<details>
<summary>28. What is the difference between @OneToOne and @JoinColumn?</summary>

`@OneToOne` defines the **relationship** between entities, while `@JoinColumn` specifies which side **owns** the relationship and creates the foreign key column in the database.

### Key Differences

```typescript
/*
@OneToOne:
- Defines the TYPE of relationship (one-to-one)
- Required on BOTH sides (owner and inverse) for bi-directional
- Required only on owner side for uni-directional
- Specifies the target entity and inverse side

@JoinColumn:
- Specifies the OWNER side of the relationship
- Creates the foreign key column in the database
- Required only on ONE side (the owner)
- Optional: Can customize foreign key column name
*/
```

### Basic Example

```typescript
import { Entity, PrimaryGeneratedColumn, Column, OneToOne, JoinColumn } from 'typeorm';

// Owner side (HAS the foreign key)
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // @OneToOne: Defines relationship type
  // @JoinColumn: Marks this as owner (creates profileId column)
  @OneToOne(() => Profile, profile => profile.user)
  @JoinColumn() // This creates 'profileId' column in users table
  profile: Profile;
}

// Inverse side (NO foreign key)
@Entity()
export class Profile {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  bio: string;

  // @OneToOne without @JoinColumn: Inverse side
  @OneToOne(() => User, user => user.profile)
  user: User;
}

// Database structure:
// users table: id, name, profileId ← Foreign key created by @JoinColumn
// profiles table: id, bio ← No foreign key
```

### What Happens Without @JoinColumn

```typescript
// ❌ ERROR: Missing @JoinColumn
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @OneToOne(() => Profile)
  profile: Profile;
  // Error: Cannot create foreign key without @JoinColumn
}

// ✅ CORRECT: With @JoinColumn
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @OneToOne(() => Profile)
  @JoinColumn() // Now foreign key 'profileId' is created
  profile: Profile;
}
```

### Customizing Foreign Key with @JoinColumn

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToOne(() => Profile, profile => profile.user)
  @JoinColumn({
    name: 'user_profile_id', // Custom column name
    referencedColumnName: 'id', // Column in Profile table to reference
  })
  profile: Profile;
}

// Database: users table will have 'user_profile_id' instead of default 'profileId'
```

### Multiple Custom Options

```typescript
@Entity()
export class Employee {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToOne(() => Desk, desk => desk.employee)
  @JoinColumn({
    name: 'assigned_desk_id', // Custom foreign key column name
    referencedColumnName: 'id', // Column in Desk table
  })
  desk: Desk;
}

@Entity()
export class Desk {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  deskNumber: string;

  @OneToOne(() => Employee, employee => employee.desk)
  employee: Employee;
}

// Database:
// employees table: id, name, assigned_desk_id
// desks table: id, deskNumber
```

### Choosing Owner Side

```typescript
// Rule: Parent entity should own the relationship

// ✅ GOOD: User owns Profile
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @OneToOne(() => Profile, profile => profile.user)
  @JoinColumn() // User has profileId foreign key
  profile: Profile;
}

// ❌ BAD: Profile owns User (counter-intuitive)
@Entity()
export class Profile {
  @PrimaryGeneratedColumn()
  id: number;

  @OneToOne(() => User, user => user.profile)
  @JoinColumn() // Profile has userId foreign key - not ideal
  user: User;
}

/*
Guidelines for choosing owner side:
1. Parent entity owns child (User owns Profile, not Profile owns User)
2. Entity that "has" another entity is the owner
3. Entity that makes more sense to query is the owner
4. Entity that's more likely to exist independently is NOT the owner
*/
```

### Bi-directional with @JoinColumn

```typescript
// Both sides know about each other
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  // @OneToOne + @JoinColumn: Owner side
  @OneToOne(() => Profile, profile => profile.user, {
    cascade: true,
  })
  @JoinColumn()
  profile: Profile;
}

@Entity()
export class Profile {
  @PrimaryGeneratedColumn()
  id: number;

  // @OneToOne without @JoinColumn: Inverse side
  @OneToOne(() => User, user => user.profile)
  user: User;
}

// Can navigate both ways:
const user = await userRepository.findOne({
  where: { id: 1 },
  relations: ['profile'],
});
console.log(user.profile.bio); // User → Profile

const profile = await profileRepository.findOne({
  where: { id: 1 },
  relations: ['user'],
});
console.log(profile.user.name); // Profile → User
```

### Uni-directional (Only @OneToOne)

```typescript
// Only User knows about Profile
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @OneToOne(() => Profile)
  @JoinColumn()
  profile: Profile;
}

@Entity()
export class Profile {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  bio: string;
  // No reference to User
}

// Can only navigate from User to Profile
const user = await userRepository.findOne({
  where: { id: 1 },
  relations: ['profile'],
});
console.log(user.profile.bio); // ✅ Works

// Cannot navigate from Profile to User
const profile = await profileRepository.findOne({
  where: { id: 1 },
});
// profile.user doesn't exist ❌
```

### Real-World Example: Order Invoice

```typescript
@Entity()
export class Order {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  orderNumber: string;

  @Column('decimal', { precision: 10, scale: 2 })
  total: number;

  @Column()
  customerId: number;

  // @OneToOne: Defines relationship
  // @JoinColumn: Order owns Invoice (has invoiceId foreign key)
  @OneToOne(() => Invoice, invoice => invoice.order, {
    cascade: true, // Auto-save invoice when order is saved
    nullable: true, // Invoice might not exist immediately
  })
  @JoinColumn({ name: 'invoice_id' })
  invoice: Invoice | null;

  @Column()
  status: string;

  @CreateDateColumn()
  createdAt: Date;
}

@Entity()
export class Invoice {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  invoiceNumber: string;

  @Column('decimal', { precision: 10, scale: 2 })
  amount: number;

  @Column('decimal', { precision: 10, scale: 2 })
  tax: number;

  @Column({ type: 'date' })
  issueDate: Date;

  @Column({ type: 'date' })
  dueDate: Date;

  // Inverse side - no @JoinColumn
  @OneToOne(() => Order, order => order.invoice)
  order: Order;

  @Column({ default: 'pending' })
  status: string;
}

// Service
class OrderService {
  async createOrderWithInvoice(orderData: any): Promise<Order> {
    const order = new Order();
    order.orderNumber = 'ORD-2024-001';
    order.total = 299.99;
    order.customerId = 123;
    order.status = 'pending';

    // Create invoice
    const invoice = new Invoice();
    invoice.invoiceNumber = 'INV-2024-001';
    invoice.amount = 299.99;
    invoice.tax = 29.99;
    invoice.issueDate = new Date();
    invoice.dueDate = new Date(Date.now() + 30 * 24 * 60 * 60 * 1000); // 30 days
    invoice.status = 'pending';

    order.invoice = invoice;

    // Cascade automatically saves invoice
    return await orderRepository.save(order);
  }

  async getOrderWithInvoice(orderId: number): Promise<Order> {
    return await orderRepository.findOne({
      where: { id: orderId },
      relations: ['invoice'],
    });
  }
}
```

### Composite Foreign Keys with @JoinColumn

```typescript
@Entity()
export class User {
  @PrimaryColumn()
  tenantId: number;

  @PrimaryColumn()
  userId: number;

  @Column()
  name: string;

  @OneToOne(() => Profile, profile => profile.user)
  @JoinColumn([
    { name: 'profile_tenant_id', referencedColumnName: 'tenantId' },
    { name: 'profile_user_id', referencedColumnName: 'userId' },
  ])
  profile: Profile;
}

@Entity()
export class Profile {
  @PrimaryColumn()
  tenantId: number;

  @PrimaryColumn()
  userId: number;

  @Column()
  bio: string;

  @OneToOne(() => User, user => user.profile)
  user: User;
}

// Database:
// users table: tenantId, userId (composite PK), name, profile_tenant_id, profile_user_id
// profiles table: tenantId, userId (composite PK), bio
```

### Self-Referencing with @JoinColumn

```typescript
@Entity()
export class Employee {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Employee can have one mentor (another employee)
  @OneToOne(() => Employee, employee => employee.mentee, {
    nullable: true,
  })
  @JoinColumn({ name: 'mentor_id' })
  mentor: Employee | null;

  // Inverse: employee can be mentor to one mentee
  @OneToOne(() => Employee, employee => employee.mentor)
  mentee: Employee;
}

// Database:
// employees table: id, name, mentor_id (references employees.id)
```

### Comparison Table

```typescript
/*
┌─────────────────────────────────────────────────────────────────────────────┐
│                        @OneToOne vs @JoinColumn                             │
├──────────────────┬──────────────────────────┬────────────────────────────────┤
│ Aspect           │ @OneToOne                │ @JoinColumn                    │
├──────────────────┼──────────────────────────┼────────────────────────────────┤
│ Purpose          │ Define relationship type │ Specify ownership & FK         │
├──────────────────┼──────────────────────────┼────────────────────────────────┤
│ Usage            │ Required on both sides   │ Required only on owner side    │
│                  │ (bi-directional)         │                                │
├──────────────────┼──────────────────────────┼────────────────────────────────┤
│ Database Effect  │ No direct effect         │ Creates foreign key column     │
├──────────────────┼──────────────────────────┼────────────────────────────────┤
│ Parameters       │ Target entity,           │ Column name,                   │
│                  │ inverse side             │ referenced column              │
├──────────────────┼──────────────────────────┼────────────────────────────────┤
│ Can Customize    │ Cascade, eager, lazy     │ Foreign key column name        │
├──────────────────┼──────────────────────────┼────────────────────────────────┤
│ Example          │ @OneToOne(() => Profile) │ @JoinColumn({ name: 'fk_id' }) │
└──────────────────┴──────────────────────────┴────────────────────────────────┘

REMEMBER:
- @OneToOne DEFINES the relationship
- @JoinColumn OWNS the relationship (creates FK)
- Owner side: Has both @OneToOne + @JoinColumn
- Inverse side: Has only @OneToOne (no @JoinColumn)
- Without @JoinColumn: No foreign key created = Error
*/
```

### Best Practices

```typescript
// ✅ GOOD: Always use @JoinColumn on owner side
@OneToOne(() => Profile, profile => profile.user)
@JoinColumn()
profile: Profile;

// ✅ GOOD: Customize FK name for clarity
@OneToOne(() => Profile, profile => profile.user)
@JoinColumn({ name: 'user_profile_id' })
profile: Profile;

// ✅ GOOD: Choose logical owner (parent owns child)
// User owns Profile (not Profile owns User)

// ❌ BAD: Don't put @JoinColumn on both sides
@Entity()
export class User {
  @OneToOne(() => Profile)
  @JoinColumn() // ❌ Both sides have @JoinColumn
  profile: Profile;
}

@Entity()
export class Profile {
  @OneToOne(() => User)
  @JoinColumn() // ❌ Don't do this!
  user: User;
}

// ❌ BAD: Missing @JoinColumn entirely
@OneToOne(() => Profile)
profile: Profile; // ❌ No foreign key created

// ✅ GOOD: Always specify inverse side for bi-directional
@OneToOne(() => Profile, profile => profile.user) // Second param is inverse
@JoinColumn()
profile: Profile;
```

**Interview Tip**: Explain that `@OneToOne` defines the **relationship type**, while `@JoinColumn` specifies the **owner side** and creates the **foreign key**. Emphasize: owner side needs both decorators, inverse side only needs `@OneToOne`. The owner side has the foreign key column in the database. Discuss customization: `@JoinColumn({ name: 'custom_fk' })` changes column name. Mention choosing owner: parent entity should own child (User owns Profile, Order owns Invoice). For bi-directional relationships, specify inverse side in second parameter: `@OneToOne(() => Profile, profile => profile.user)`. Without `@JoinColumn`, no foreign key is created and the relationship won't work. A strong answer demonstrates understanding that `@OneToOne` is about the logical relationship, while `@JoinColumn` is about physical database structure.

</details>

<details>
<summary>29. What is One-to-Many relationship and How do you define a One-to-Many relationship?</summary>

A **One-to-Many relationship** means one entity instance relates to multiple instances of another entity. This is the most common relationship type in databases (e.g., one user has many posts, one category has many products).

### Basic One-to-Many Relationship

```typescript
import { Entity, PrimaryGeneratedColumn, Column, OneToMany } from 'typeorm';

// One user has many posts
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  email: string;

  // One-to-Many relationship
  @OneToMany(() => Post, post => post.author)
  posts: Post[];
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column('text')
  content: string;

  // Many-to-One side (foreign key is here)
  @ManyToOne(() => User, user => user.posts)
  author: User;
  // Foreign key 'authorId' is automatically created
}

// Database structure:
// users table: id, name, email
// posts table: id, title, content, authorId (foreign key to users.id)

// Usage
const user = new User();
user.name = 'John Doe';
user.email = 'john@example.com';
await userRepository.save(user);

const post1 = new Post();
post1.title = 'First Post';
post1.content = 'Hello World';
post1.author = user;

const post2 = new Post();
post2.title = 'Second Post';
post2.content = 'Another post';
post2.author = user;

await postRepository.save([post1, post2]);
```

### Key Points

```typescript
/*
One-to-Many Characteristics:
1. @OneToMany is always the INVERSE side (no foreign key)
2. Foreign key is on the "Many" side (@ManyToOne)
3. Returns an array on the "One" side
4. One-to-Many MUST be paired with @ManyToOne on the other side
5. Cannot exist without corresponding @ManyToOne
*/
```

### With Cascade Options

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Cascade: Save/remove posts when user is saved/removed
  @OneToMany(() => Post, post => post.author, {
    cascade: true, // Auto-save posts when saving user
    eager: false, // Don't auto-load posts (can be expensive)
  })
  posts: Post[];
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToOne(() => User, user => user.posts, {
    onDelete: 'CASCADE', // Delete posts when user is deleted
  })
  author: User;
}

// Usage with cascade
const user = new User();
user.name = 'John';

const post1 = new Post();
post1.title = 'Post 1';

const post2 = new Post();
post2.title = 'Post 2';

user.posts = [post1, post2];

await userRepository.save(user); // Cascade saves posts automatically
```

### Uni-directional One-to-Many

```typescript
// Only parent knows about children (not recommended)
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @OneToMany(() => Post, post => post.author)
  posts: Post[];
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  // Must have @ManyToOne even if uni-directional
  @ManyToOne(() => User)
  author: User;
}
```

### Real-World Example: Blog System

```typescript
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  OneToMany,
  ManyToOne,
  CreateDateColumn,
  UpdateDateColumn,
} from 'typeorm';

// User entity
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  username: string;

  @Column({ unique: true })
  email: string;

  // One user has many posts
  @OneToMany(() => Post, post => post.author, { cascade: true })
  posts: Post[];

  // One user has many comments
  @OneToMany(() => Comment, comment => comment.author)
  comments: Comment[];

  @CreateDateColumn()
  createdAt: Date;
}

// Post entity
@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column('text')
  content: string;

  @Column({ default: 'draft' })
  status: string;

  // Many posts belong to one author
  @ManyToOne(() => User, user => user.posts, {
    onDelete: 'CASCADE', // Delete posts when user is deleted
  })
  author: User;

  // One post has many comments
  @OneToMany(() => Comment, comment => comment.post, { cascade: true })
  comments: Comment[];

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}

// Comment entity
@Entity()
export class Comment {
  @PrimaryGeneratedColumn()
  id: number;

  @Column('text')
  content: string;

  // Many comments belong to one author
  @ManyToOne(() => User, user => user.comments, {
    onDelete: 'CASCADE',
  })
  author: User;

  // Many comments belong to one post
  @ManyToOne(() => Post, post => post.comments, {
    onDelete: 'CASCADE',
  })
  post: Post;

  @CreateDateColumn()
  createdAt: Date;
}

// Service
class BlogService {
  async createPostWithComments(userId: number): Promise<Post> {
    const user = await userRepository.findOneBy({ id: userId });

    const post = new Post();
    post.title = 'My Blog Post';
    post.content = 'This is the content';
    post.author = user;
    post.status = 'published';

    const comment1 = new Comment();
    comment1.content = 'Great post!';
    comment1.author = user;

    const comment2 = new Comment();
    comment2.content = 'Thanks for sharing';
    comment2.author = user;

    post.comments = [comment1, comment2];

    return await postRepository.save(post); // Cascade saves comments
  }

  async getUserPosts(userId: number): Promise<Post[]> {
    return await postRepository.find({
      where: { author: { id: userId } },
      relations: ['comments', 'comments.author'],
      order: { createdAt: 'DESC' },
    });
  }

  async getPostsByStatus(status: string): Promise<Post[]> {
    return await postRepository.find({
      where: { status },
      relations: ['author'],
    });
  }
}
```

### E-commerce Example: Category-Products

```typescript
@Entity()
export class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  slug: string;

  @Column('text', { nullable: true })
  description: string;

  // One category has many products
  @OneToMany(() => Product, product => product.category)
  products: Product[];

  @CreateDateColumn()
  createdAt: Date;
}

@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  sku: string;

  @Column('decimal', { precision: 10, scale: 2 })
  price: number;

  @Column({ type: 'int', default: 0 })
  stock: number;

  // Many products belong to one category
  @ManyToOne(() => Category, category => category.products, {
    nullable: false, // Product must have a category
  })
  category: Category;

  @CreateDateColumn()
  createdAt: Date;
}

// Service
class ProductService {
  async getProductsByCategory(categoryId: number): Promise<Product[]> {
    return await productRepository.find({
      where: { category: { id: categoryId } },
      order: { name: 'ASC' },
    });
  }

  async getCategoryWithProducts(categoryId: number): Promise<Category> {
    return await categoryRepository.findOne({
      where: { id: categoryId },
      relations: ['products'],
    });
  }

  async moveProductsToCategory(
    productIds: number[],
    targetCategoryId: number
  ): Promise<void> {
    const category = await categoryRepository.findOneBy({ id: targetCategoryId });

    if (!category) {
      throw new Error('Category not found');
    }

    await productRepository
      .createQueryBuilder()
      .update(Product)
      .set({ category })
      .where('id IN (:...ids)', { ids: productIds })
      .execute();
  }
}
```

### Querying One-to-Many Relationships

```typescript
// Load user with posts
const user = await userRepository.findOne({
  where: { id: 1 },
  relations: ['posts'],
});

console.log(`User ${user.name} has ${user.posts.length} posts`);

// Query builder with join
const users = await userRepository
  .createQueryBuilder('user')
  .leftJoinAndSelect('user.posts', 'post')
  .where('user.id = :id', { id: 1 })
  .getMany();

// Load posts with author
const posts = await postRepository.find({
  relations: ['author'],
  where: { author: { id: 1 } },
});

// Find users with at least one post
const activeUsers = await userRepository
  .createQueryBuilder('user')
  .innerJoinAndSelect('user.posts', 'post')
  .getMany();

// Find users without posts
const inactiveUsers = await userRepository
  .createQueryBuilder('user')
  .leftJoinAndSelect('user.posts', 'post')
  .where('post.id IS NULL')
  .getMany();

// Count posts per user
const userPostCounts = await userRepository
  .createQueryBuilder('user')
  .leftJoin('user.posts', 'post')
  .select('user.id', 'userId')
  .addSelect('user.name', 'userName')
  .addSelect('COUNT(post.id)', 'postCount')
  .groupBy('user.id')
  .getRawMany();
```

### Order Management Example

```typescript
@Entity()
export class Customer {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  email: string;

  // One customer has many orders
  @OneToMany(() => Order, order => order.customer)
  orders: Order[];

  @CreateDateColumn()
  createdAt: Date;
}

@Entity()
export class Order {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  orderNumber: string;

  @Column('decimal', { precision: 10, scale: 2 })
  total: number;

  @Column()
  status: string;

  // Many orders belong to one customer
  @ManyToOne(() => Customer, customer => customer.orders, {
    onDelete: 'RESTRICT', // Cannot delete customer with orders
  })
  customer: Customer;

  // One order has many items
  @OneToMany(() => OrderItem, item => item.order, { cascade: true })
  items: OrderItem[];

  @CreateDateColumn()
  createdAt: Date;
}

@Entity()
export class OrderItem {
  @PrimaryGeneratedColumn()
  id: number;

  @Column('int')
  quantity: number;

  @Column('decimal', { precision: 10, scale: 2 })
  price: number;

  // Many items belong to one order
  @ManyToOne(() => Order, order => order.items, {
    onDelete: 'CASCADE', // Delete items when order is deleted
  })
  order: Order;

  @Column()
  productId: number;
}

// Service
class OrderService {
  async createOrder(customerId: number, items: any[]): Promise<Order> {
    const customer = await customerRepository.findOneBy({ id: customerId });

    if (!customer) {
      throw new Error('Customer not found');
    }

    const order = new Order();
    order.orderNumber = `ORD-${Date.now()}`;
    order.customer = customer;
    order.status = 'pending';

    const orderItems = items.map(item => {
      const orderItem = new OrderItem();
      orderItem.quantity = item.quantity;
      orderItem.price = item.price;
      orderItem.productId = item.productId;
      return orderItem;
    });

    order.items = orderItems;
    order.total = orderItems.reduce(
      (sum, item) => sum + item.price * item.quantity,
      0
    );

    return await orderRepository.save(order);
  }

  async getCustomerOrders(customerId: number): Promise<Order[]> {
    return await orderRepository.find({
      where: { customer: { id: customerId } },
      relations: ['items'],
      order: { createdAt: 'DESC' },
    });
  }

  async getOrderStats(customerId: number): Promise<any> {
    return await orderRepository
      .createQueryBuilder('order')
      .select('COUNT(order.id)', 'totalOrders')
      .addSelect('SUM(order.total)', 'totalSpent')
      .addSelect('AVG(order.total)', 'averageOrderValue')
      .where('order.customerId = :customerId', { customerId })
      .getRawOne();
  }
}
```

### Pagination with One-to-Many

```typescript
// Paginate posts for a user
async function getUserPostsPaginated(
  userId: number,
  page: number = 1,
  limit: number = 10
): Promise<{ posts: Post[]; total: number; pages: number }> {
  const [posts, total] = await postRepository.findAndCount({
    where: { author: { id: userId } },
    relations: ['author'],
    order: { createdAt: 'DESC' },
    skip: (page - 1) * limit,
    take: limit,
  });

  return {
    posts,
    total,
    pages: Math.ceil(total / limit),
  };
}

// Query builder pagination
async function getPostsPaginated(
  page: number = 1,
  limit: number = 10
): Promise<any> {
  const query = postRepository
    .createQueryBuilder('post')
    .leftJoinAndSelect('post.author', 'author')
    .leftJoinAndSelect('post.comments', 'comment')
    .orderBy('post.createdAt', 'DESC')
    .skip((page - 1) * limit)
    .take(limit);

  const [posts, total] = await query.getManyAndCount();

  return {
    data: posts,
    pagination: {
      page,
      limit,
      total,
      pages: Math.ceil(total / limit),
    },
  };
}
```

### Best Practices

```typescript
// ✅ GOOD: Always pair @OneToMany with @ManyToOne
@Entity()
export class User {
  @OneToMany(() => Post, post => post.author)
  posts: Post[];
}

@Entity()
export class Post {
  @ManyToOne(() => User, user => user.posts)
  author: User;
}

// ✅ GOOD: Use cascade for dependent entities
@OneToMany(() => Post, post => post.author, {
  cascade: true, // Auto-save posts with user
})
posts: Post[];

// ✅ GOOD: Specify onDelete behavior
@ManyToOne(() => User, user => user.posts, {
  onDelete: 'CASCADE', // Delete posts when user is deleted
})
author: User;

// ✅ GOOD: Use nullable for optional relationships
@ManyToOne(() => Category, category => category.products, {
  nullable: true, // Product can exist without category
})
category: Category | null;

// ❌ BAD: Don't use eager loading for large collections
@OneToMany(() => Post, post => post.author, {
  eager: true, // Bad if user has thousands of posts
})
posts: Post[];

// ✅ GOOD: Load relations only when needed
const user = await userRepository.findOne({
  where: { id: 1 },
  relations: ['posts'], // Explicit loading
});

// ❌ BAD: Don't forget the inverse side
@OneToMany(() => Post, post => post.author)
posts: Post[];
// Must have corresponding @ManyToOne on Post entity

// ✅ GOOD: Initialize arrays to avoid null errors
@OneToMany(() => Post, post => post.author)
posts: Post[] = [];
```

**Interview Tip**: Explain that One-to-Many is the most common relationship where one entity has multiple instances of another (User has many Posts). Emphasize that `@OneToMany` is always the **inverse side** with no foreign key; the foreign key is on the `@ManyToOne` side. Highlight that `@OneToMany` must be paired with `@ManyToOne` on the other entity. Discuss cascade options: `cascade: true` auto-saves children, `onDelete: 'CASCADE'` deletes children when parent is deleted. Mention real-world examples: User-Posts, Category-Products, Order-Items. For production: avoid eager loading large collections, use pagination for large datasets, specify nullable for optional relationships, always load relations explicitly when needed. A strong answer demonstrates understanding of the bi-directional nature and proper cascade/delete behavior.

</details>

<details>
<summary>30. What is Many-to-One relationship and How do you define a Many-to-One relationship?</summary>

A **Many-to-One relationship** means multiple entity instances relate to one instance of another entity. It's the inverse of One-to-Many and is where the foreign key is actually stored in the database.

### Basic Many-to-One Relationship

```typescript
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne } from 'typeorm';

// Many posts belong to one author
@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column('text')
  content: string;

  // Many-to-One relationship (foreign key is here)
  @ManyToOne(() => User, user => user.posts)
  author: User;
  // Foreign key 'authorId' is automatically created
}

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  email: string;

  // Inverse side (One-to-Many)
  @OneToMany(() => Post, post => post.author)
  posts: Post[];
}

// Database structure:
// users table: id, name, email
// posts table: id, title, content, authorId (foreign key)

// Usage
const user = await userRepository.findOneBy({ id: 1 });

const post = new Post();
post.title = 'My Post';
post.content = 'Content here';
post.author = user; // Set the relationship

await postRepository.save(post);
```

### Key Characteristics

```typescript
/*
Many-to-One Characteristics:
1. Foreign key is stored on this side (in the "Many" table)
2. Automatically creates foreign key column (e.g., authorId)
3. Returns a single entity (not an array)
4. Most common side to query from (posts belong to author)
5. Can exist without corresponding @OneToMany on the other side
6. Always has @ManyToOne, while @OneToMany is optional (but recommended)
*/
```

### Custom Foreign Key Name

```typescript
@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  // Custom foreign key column name
  @ManyToOne(() => User, user => user.posts)
  @JoinColumn({ name: 'created_by_user_id' })
  author: User;
  // Foreign key will be 'created_by_user_id' instead of 'authorId'
}

// Database: posts table has 'created_by_user_id' column
```

### With Options

```typescript
@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToOne(() => User, user => user.posts, {
    eager: false, // Don't auto-load author (default)
    nullable: false, // Post must have an author (required)
    onDelete: 'CASCADE', // Delete posts when author is deleted
    onUpdate: 'CASCADE', // Update foreign key when author id changes
  })
  author: User;
}
```

### Nullable Relationships

```typescript
@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Optional category (nullable)
  @ManyToOne(() => Category, category => category.products, {
    nullable: true, // Product can exist without category
  })
  category: Category | null;
}

// Usage
const product = new Product();
product.name = 'Laptop';
product.category = null; // No category assigned
await productRepository.save(product);
```

### Multiple Many-to-One Relationships

```typescript
@Entity()
export class Task {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column('text')
  description: string;

  // Many tasks belong to one assignee
  @ManyToOne(() => User, user => user.assignedTasks, {
    nullable: true,
  })
  assignee: User | null;

  // Many tasks belong to one creator
  @ManyToOne(() => User, user => user.createdTasks, {
    nullable: false,
  })
  creator: User;

  // Many tasks belong to one project
  @ManyToOne(() => Project, project => project.tasks, {
    onDelete: 'CASCADE',
  })
  project: Project;

  @CreateDateColumn()
  createdAt: Date;
}

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Task, task => task.assignee)
  assignedTasks: Task[];

  @OneToMany(() => Task, task => task.creator)
  createdTasks: Task[];
}

@Entity()
export class Project {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Task, task => task.project)
  tasks: Task[];
}
```

### Real-World Example: Comment System

```typescript
@Entity()
export class Comment {
  @PrimaryGeneratedColumn()
  id: number;

  @Column('text')
  content: string;

  @Column({ type: 'int', default: 0 })
  likes: number;

  // Many comments by one user
  @ManyToOne(() => User, user => user.comments, {
    onDelete: 'SET NULL', // Keep comment, set author to null if user deleted
    nullable: true,
  })
  author: User | null;

  // Many comments on one post
  @ManyToOne(() => Post, post => post.comments, {
    onDelete: 'CASCADE', // Delete comment when post is deleted
  })
  post: Post;

  // Parent comment for replies (self-referencing)
  @ManyToOne(() => Comment, comment => comment.replies, {
    nullable: true,
    onDelete: 'CASCADE',
  })
  parentComment: Comment | null;

  // Child comments (replies)
  @OneToMany(() => Comment, comment => comment.parentComment)
  replies: Comment[];

  @CreateDateColumn()
  createdAt: Date;
}

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  username: string;

  @OneToMany(() => Comment, comment => comment.author)
  comments: Comment[];
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @OneToMany(() => Comment, comment => comment.post, { cascade: true })
  comments: Comment[];
}

// Service
class CommentService {
  async createComment(
    userId: number,
    postId: number,
    content: string,
    parentCommentId?: number
  ): Promise<Comment> {
    const user = await userRepository.findOneBy({ id: userId });
    const post = await postRepository.findOneBy({ id: postId });

    const comment = new Comment();
    comment.content = content;
    comment.author = user;
    comment.post = post;

    if (parentCommentId) {
      const parentComment = await commentRepository.findOneBy({
        id: parentCommentId,
      });
      comment.parentComment = parentComment;
    }

    return await commentRepository.save(comment);
  }

  async getPostComments(postId: number): Promise<Comment[]> {
    return await commentRepository.find({
      where: {
        post: { id: postId },
        parentComment: IsNull(), // Only top-level comments
      },
      relations: ['author', 'replies', 'replies.author'],
      order: { createdAt: 'DESC' },
    });
  }
}
```

### E-commerce Order Items

```typescript
@Entity()
export class OrderItem {
  @PrimaryGeneratedColumn()
  id: number;

  @Column('int')
  quantity: number;

  @Column('decimal', { precision: 10, scale: 2 })
  price: number; // Price at time of order

  // Many items belong to one order
  @ManyToOne(() => Order, order => order.items, {
    onDelete: 'CASCADE', // Delete items when order is deleted
  })
  order: Order;

  // Many items reference one product
  @ManyToOne(() => Product, product => product.orderItems, {
    eager: true, // Always load product info
  })
  product: Product;

  // Computed property
  getSubtotal(): number {
    return this.quantity * this.price;
  }
}

@Entity()
export class Order {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  orderNumber: string;

  @Column('decimal', { precision: 10, scale: 2 })
  total: number;

  @OneToMany(() => OrderItem, item => item.order, { cascade: true })
  items: OrderItem[];

  // Many orders belong to one customer
  @ManyToOne(() => Customer, customer => customer.orders)
  customer: Customer;

  @CreateDateColumn()
  createdAt: Date;
}

@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column('decimal', { precision: 10, scale: 2 })
  price: number;

  @OneToMany(() => OrderItem, item => item.product)
  orderItems: OrderItem[];
}

@Entity()
export class Customer {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Order, order => order.customer)
  orders: Order[];
}
```

### Querying Many-to-One Relationships

```typescript
// Find posts by author
const posts = await postRepository.find({
  where: { author: { id: 1 } },
  relations: ['author'],
});

// Query builder
const posts = await postRepository
  .createQueryBuilder('post')
  .leftJoinAndSelect('post.author', 'author')
  .where('author.id = :authorId', { authorId: 1 })
  .getMany();

// Find posts by author email
const posts = await postRepository
  .createQueryBuilder('post')
  .leftJoinAndSelect('post.author', 'author')
  .where('author.email = :email', { email: 'john@example.com' })
  .getMany();

// Count posts per author
const authorStats = await postRepository
  .createQueryBuilder('post')
  .leftJoin('post.author', 'author')
  .select('author.id', 'authorId')
  .addSelect('author.name', 'authorName')
  .addSelect('COUNT(post.id)', 'postCount')
  .groupBy('author.id')
  .getRawMany();

// Find posts without author (orphaned)
const orphanedPosts = await postRepository
  .createQueryBuilder('post')
  .where('post.authorId IS NULL')
  .getMany();
```

### Soft Delete with Many-to-One

```typescript
@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToOne(() => User, user => user.posts, {
    onDelete: 'SET NULL', // Set to null when user is soft-deleted
    nullable: true,
  })
  author: User | null;

  @DeleteDateColumn()
  deletedAt: Date | null;
}

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Post, post => post.author)
  posts: Post[];

  @DeleteDateColumn()
  deletedAt: Date | null;
}

// When user is soft-deleted, posts remain but author is set to null
```

### Audit Trail with Many-to-One

```typescript
@Entity()
export class AuditLog {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  action: string; // 'CREATE', 'UPDATE', 'DELETE'

  @Column()
  entityType: string; // 'Post', 'Comment', etc.

  @Column()
  entityId: number;

  @Column('json')
  changes: any;

  // Many logs by one user
  @ManyToOne(() => User, { eager: true })
  user: User;

  @CreateDateColumn()
  timestamp: Date;
}

// Service
class AuditService {
  async logAction(
    userId: number,
    action: string,
    entityType: string,
    entityId: number,
    changes: any
  ): Promise<void> {
    const user = await userRepository.findOneBy({ id: userId });

    const log = new AuditLog();
    log.user = user;
    log.action = action;
    log.entityType = entityType;
    log.entityId = entityId;
    log.changes = changes;

    await auditLogRepository.save(log);
  }

  async getUserAuditTrail(userId: number): Promise<AuditLog[]> {
    return await auditLogRepository.find({
      where: { user: { id: userId } },
      order: { timestamp: 'DESC' },
      take: 100,
    });
  }
}
```

### Best Practices

```typescript
// ✅ GOOD: Specify onDelete behavior
@ManyToOne(() => User, user => user.posts, {
  onDelete: 'CASCADE', // Delete posts when user is deleted
})
author: User;

// ✅ GOOD: Use nullable for optional relationships
@ManyToOne(() => Category, category => category.products, {
  nullable: true,
})
category: Category | null;

// ✅ GOOD: Use eager loading for frequently accessed relations
@ManyToOne(() => User, user => user.posts, {
  eager: true, // Always load author with post
})
author: User;

// ✅ GOOD: Custom foreign key names for clarity
@ManyToOne(() => User, user => user.createdPosts)
@JoinColumn({ name: 'created_by_user_id' })
creator: User;

// ❌ BAD: Don't set nullable: false without default
@ManyToOne(() => User, user => user.posts, {
  nullable: false, // Must provide author when creating post
})
author: User;

// ✅ GOOD: Use onDelete: 'SET NULL' for soft references
@ManyToOne(() => User, user => user.posts, {
  onDelete: 'SET NULL', // Keep post even if author deleted
  nullable: true,
})
author: User | null;

// ✅ GOOD: Index foreign keys for performance
@Index()
@ManyToOne(() => User, user => user.posts)
author: User;

// ❌ BAD: Don't forget to load relations when needed
const post = await postRepository.findOneBy({ id: 1 });
console.log(post.author.name); // Error: author not loaded
```

**Interview Tip**: Explain that Many-to-One is where the foreign key is actually stored (the "many" side owns the relationship). Emphasize it's the inverse of One-to-Many and most commonly used for queries (find posts by author). Highlight key features: automatically creates foreign key column, returns single entity (not array), can specify `nullable`, `onDelete`, and `onUpdate` options. Discuss onDelete behaviors: `CASCADE` deletes children with parent, `SET NULL` keeps children but nullifies reference, `RESTRICT` prevents deletion if children exist. Mention eager loading: useful for frequently accessed relations but can impact performance. For production: always specify onDelete behavior, index foreign keys for query performance, use nullable appropriately, consider soft delete implications. A strong answer demonstrates understanding of foreign key ownership and cascade behaviors.

</details>

<details>
<summary>31. What is Many-to-Many relationship and How do you define a Many-to-Many relationship?</summary>

A **Many-to-Many relationship** means multiple instances of one entity relate to multiple instances of another entity. This requires a **junction table** (also called join table) to store the relationships between the two entities.

### Basic Many-to-Many Relationship

```typescript
import { Entity, PrimaryGeneratedColumn, Column, ManyToMany, JoinTable } from 'typeorm';

// Many students enroll in many courses
@Entity()
export class Student {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  email: string;

  // Many-to-Many relationship (owner side)
  @ManyToMany(() => Course, course => course.students)
  @JoinTable() // Creates junction table
  courses: Course[];
}

@Entity()
export class Course {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column()
  code: string;

  // Inverse side (no @JoinTable)
  @ManyToMany(() => Student, student => student.courses)
  students: Student[];
}

// Database structure:
// students table: id, name, email
// courses table: id, title, code
// student_courses_course table: studentId, courseId (junction table)

// Usage
const student = new Student();
student.name = 'John Doe';
student.email = 'john@example.com';

const course1 = await courseRepository.findOneBy({ id: 1 });
const course2 = await courseRepository.findOneBy({ id: 2 });

student.courses = [course1, course2];

await studentRepository.save(student); // Creates junction table entries
```

### Key Points

```typescript
/*
Many-to-Many Characteristics:
1. Requires junction table (created automatically)
2. @JoinTable is required on ONE side (owner side)
3. Owner side can add/remove relationships
4. Inverse side can only read relationships
5. Both sides return arrays
6. No foreign keys in main tables, only in junction table
*/
```

### Custom Junction Table

```typescript
@Entity()
export class Student {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @ManyToMany(() => Course, course => course.students)
  @JoinTable({
    name: 'student_enrollments', // Custom junction table name
    joinColumn: {
      name: 'student_id', // Column in junction table for Student
      referencedColumnName: 'id',
    },
    inverseJoinColumn: {
      name: 'course_id', // Column in junction table for Course
      referencedColumnName: 'id',
    },
  })
  courses: Course[];
}

// Database:
// student_enrollments table: student_id, course_id
```

### With Cascade Options

```typescript
@Entity()
export class Student {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @ManyToMany(() => Course, course => course.students, {
    cascade: true, // Auto-save courses when student is saved
    eager: false, // Don't auto-load courses
  })
  @JoinTable()
  courses: Course[];
}

// Usage
const student = new Student();
student.name = 'John';

const course = new Course();
course.title = 'TypeScript 101';
course.code = 'TS101';

student.courses = [course];

await studentRepository.save(student); // Cascade saves course and junction entry
```

### Bi-directional vs Uni-directional

```typescript
// Bi-directional (both sides know about each other)
@Entity()
export class Student {
  @ManyToMany(() => Course, course => course.students)
  @JoinTable()
  courses: Course[];
}

@Entity()
export class Course {
  @ManyToMany(() => Student, student => student.courses)
  students: Student[];
}

// Uni-directional (only one side knows)
@Entity()
export class Student {
  @ManyToMany(() => Course)
  @JoinTable()
  courses: Course[];
}

@Entity()
export class Course {
  // No reference to students
}
```

### Real-World Example: Social Media Tags

```typescript
@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column('text')
  content: string;

  // Many posts have many tags
  @ManyToMany(() => Tag, tag => tag.posts, { cascade: true })
  @JoinTable({
    name: 'post_tags',
    joinColumn: { name: 'post_id' },
    inverseJoinColumn: { name: 'tag_id' },
  })
  tags: Tag[];

  @CreateDateColumn()
  createdAt: Date;
}

@Entity()
export class Tag {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ unique: true })
  name: string;

  @Column()
  slug: string;

  @Column({ default: 0 })
  usageCount: number;

  // Inverse side
  @ManyToMany(() => Post, post => post.tags)
  posts: Post[];
}

// Service
class PostService {
  async createPostWithTags(title: string, content: string, tagNames: string[]): Promise<Post> {
    const post = new Post();
    post.title = title;
    post.content = content;

    // Find or create tags
    const tags: Tag[] = [];
    for (const tagName of tagNames) {
      let tag = await tagRepository.findOne({ where: { name: tagName } });

      if (!tag) {
        tag = new Tag();
        tag.name = tagName;
        tag.slug = tagName.toLowerCase().replace(/\s+/g, '-');
        tag.usageCount = 0;
      }

      tag.usageCount++;
      tags.push(tag);
    }

    post.tags = tags;

    return await postRepository.save(post); // Cascade saves tags
  }

  async getPostsByTag(tagName: string): Promise<Post[]> {
    return await postRepository
      .createQueryBuilder('post')
      .innerJoinAndSelect('post.tags', 'tag')
      .where('tag.name = :tagName', { tagName })
      .getMany();
  }

  async getPopularTags(limit: number = 10): Promise<Tag[]> {
    return await tagRepository.find({
      order: { usageCount: 'DESC' },
      take: limit,
    });
  }
}
```

### User Roles Example

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  username: string;

  @Column()
  email: string;

  // Many users have many roles
  @ManyToMany(() => Role, role => role.users, { eager: true })
  @JoinTable({
    name: 'user_roles',
    joinColumn: { name: 'user_id' },
    inverseJoinColumn: { name: 'role_id' },
  })
  roles: Role[];

  hasRole(roleName: string): boolean {
    return this.roles.some(role => role.name === roleName);
  }

  hasAnyRole(roleNames: string[]): boolean {
    return this.roles.some(role => roleNames.includes(role.name));
  }

  hasAllRoles(roleNames: string[]): boolean {
    return roleNames.every(roleName =>
      this.roles.some(role => role.name === roleName)
    );
  }
}

@Entity()
export class Role {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ unique: true })
  name: string; // 'admin', 'user', 'moderator'

  @Column('simple-array')
  permissions: string[]; // ['read', 'write', 'delete']

  @ManyToMany(() => User, user => user.roles)
  users: User[];
}

// Service
class UserRoleService {
  async assignRole(userId: number, roleName: string): Promise<User> {
    const user = await userRepository.findOne({
      where: { id: userId },
      relations: ['roles'],
    });

    const role = await roleRepository.findOne({
      where: { name: roleName },
    });

    if (!user || !role) {
      throw new Error('User or Role not found');
    }

    // Check if user already has this role
    if (!user.roles.find(r => r.id === role.id)) {
      user.roles.push(role);
      await userRepository.save(user);
    }

    return user;
  }

  async removeRole(userId: number, roleName: string): Promise<User> {
    const user = await userRepository.findOne({
      where: { id: userId },
      relations: ['roles'],
    });

    if (!user) {
      throw new Error('User not found');
    }

    user.roles = user.roles.filter(role => role.name !== roleName);
    await userRepository.save(user);

    return user;
  }

  async getUsersByRole(roleName: string): Promise<User[]> {
    return await userRepository
      .createQueryBuilder('user')
      .innerJoinAndSelect('user.roles', 'role')
      .where('role.name = :roleName', { roleName })
      .getMany();
  }
}
```

### Product Categories (Hierarchical)

```typescript
@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column('decimal', { precision: 10, scale: 2 })
  price: number;

  // Product can belong to multiple categories
  @ManyToMany(() => Category, category => category.products)
  @JoinTable({
    name: 'product_categories',
    joinColumn: { name: 'product_id' },
    inverseJoinColumn: { name: 'category_id' },
  })
  categories: Category[];
}

@Entity()
export class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  slug: string;

  @ManyToMany(() => Product, product => product.categories)
  products: Product[];

  // Parent-child relationship (self-referencing)
  @ManyToOne(() => Category, category => category.children, {
    nullable: true,
  })
  parent: Category | null;

  @OneToMany(() => Category, category => category.parent)
  children: Category[];
}

// Service
class CategoryService {
  async getProductsByMultipleCategories(categoryIds: number[]): Promise<Product[]> {
    return await productRepository
      .createQueryBuilder('product')
      .innerJoinAndSelect('product.categories', 'category')
      .where('category.id IN (:...ids)', { ids: categoryIds })
      .getMany();
  }

  async getCategoryProducts(categoryId: number): Promise<Product[]> {
    const category = await categoryRepository.findOne({
      where: { id: categoryId },
      relations: ['products'],
    });

    return category?.products || [];
  }
}
```

### Querying Many-to-Many Relationships

```typescript
// Load student with courses
const student = await studentRepository.findOne({
  where: { id: 1 },
  relations: ['courses'],
});

console.log(`${student.name} enrolled in ${student.courses.length} courses`);

// Load course with students
const course = await courseRepository.findOne({
  where: { id: 1 },
  relations: ['students'],
});

// Query builder
const students = await studentRepository
  .createQueryBuilder('student')
  .leftJoinAndSelect('student.courses', 'course')
  .where('course.code = :code', { code: 'TS101' })
  .getMany();

// Find students in multiple courses
const students = await studentRepository
  .createQueryBuilder('student')
  .innerJoinAndSelect('student.courses', 'course')
  .where('course.id IN (:...ids)', { ids: [1, 2, 3] })
  .getMany();

// Count courses per student
const stats = await studentRepository
  .createQueryBuilder('student')
  .leftJoin('student.courses', 'course')
  .select('student.id', 'studentId')
  .addSelect('student.name', 'studentName')
  .addSelect('COUNT(course.id)', 'courseCount')
  .groupBy('student.id')
  .getRawMany();
```

### Adding/Removing Relationships

```typescript
// Add courses to student
const student = await studentRepository.findOne({
  where: { id: 1 },
  relations: ['courses'],
});

const newCourse = await courseRepository.findOneBy({ id: 5 });

student.courses.push(newCourse);
await studentRepository.save(student); // Updates junction table

// Remove course from student
student.courses = student.courses.filter(course => course.id !== 5);
await studentRepository.save(student);

// Replace all courses
student.courses = [course1, course2, course3];
await studentRepository.save(student);

// Query builder approach
await studentRepository
  .createQueryBuilder()
  .relation(Student, 'courses')
  .of(studentId)
  .add(courseId);

await studentRepository
  .createQueryBuilder()
  .relation(Student, 'courses')
  .of(studentId)
  .remove(courseId);
```

### Junction Table with Additional Data

```typescript
// ❌ BAD: Can't add extra fields to Many-to-Many
@ManyToMany(() => Course)
@JoinTable()
courses: Course[];
// Cannot add 'enrollmentDate' or 'grade' to junction table

// ✅ GOOD: Create explicit junction entity
@Entity()
export class Enrollment {
  @PrimaryGeneratedColumn()
  id: number;

  @ManyToOne(() => Student, student => student.enrollments)
  student: Student;

  @ManyToOne(() => Course, course => course.enrollments)
  course: Course;

  @Column({ type: 'date' })
  enrollmentDate: Date;

  @Column({ type: 'decimal', precision: 3, scale: 2, nullable: true })
  grade: number | null;

  @Column({ default: 'active' })
  status: string; // 'active', 'completed', 'dropped'
}

@Entity()
export class Student {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Enrollment, enrollment => enrollment.student)
  enrollments: Enrollment[];
}

@Entity()
export class Course {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @OneToMany(() => Enrollment, enrollment => enrollment.course)
  enrollments: Enrollment[];
}
```

### Best Practices

```typescript
// ✅ GOOD: Use @JoinTable on logical owner side
@Entity()
export class Student {
  @ManyToMany(() => Course, course => course.students)
  @JoinTable() // Student enrolls in courses (student is owner)
  courses: Course[];
}

// ✅ GOOD: Custom junction table name for clarity
@ManyToMany(() => Course, course => course.students)
@JoinTable({ name: 'student_course_enrollments' })
courses: Course[];

// ✅ GOOD: Use eager loading cautiously
@ManyToMany(() => Role, role => role.users, {
  eager: true, // Only if roles list is small
})
roles: Role[];

// ❌ BAD: Don't use eager on both sides
// This creates circular loading and performance issues

// ✅ GOOD: Create junction entity for additional data
// Instead of pure Many-to-Many, use junction entity with Many-to-One

// ✅ GOOD: Index junction table columns (in migration)
await queryRunner.query(`
  CREATE INDEX idx_student_courses_student 
  ON student_courses_course (studentId)
`);

await queryRunner.query(`
  CREATE INDEX idx_student_courses_course 
  ON student_courses_course (courseId)
`);

// ❌ BAD: Don't forget cascade implications
@ManyToMany(() => Tag, tag => tag.posts, {
  cascade: true, // Careful: saving post can create/update tags
})
tags: Tag[];
```

**Interview Tip**: Explain that Many-to-Many requires a junction table (join table) created automatically by `@JoinTable`. Emphasize that `@JoinTable` is required on ONE side (owner side) which can add/remove relationships. Discuss when to use: Users-Roles, Students-Courses, Products-Categories, Posts-Tags - scenarios where entities can have multiple of each other. Highlight customization: custom junction table name and column names. Mention important limitation: can't add extra fields to junction table; create explicit junction entity instead (Enrollment entity for Student-Course with grade and date). For production: index junction table columns, be careful with cascade and eager loading, choose owner side logically. A strong answer demonstrates understanding of junction tables and when to use explicit junction entities vs pure Many-to-Many.

</details>

<details>
<summary>32. What is @JoinTable and when should it be used?</summary>

`@JoinTable` is a decorator used exclusively with `@ManyToMany` relationships to specify which side **owns** the relationship and to create/customize the **junction table** (join table) that stores the Many-to-Many associations.

### Basic @JoinTable Usage

```typescript
import { Entity, PrimaryGeneratedColumn, Column, ManyToMany, JoinTable } from 'typeorm';

@Entity()
export class Student {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // @JoinTable marks this as the owner side
  @ManyToMany(() => Course, course => course.students)
  @JoinTable() // Creates junction table: student_courses_course
  courses: Course[];
}

@Entity()
export class Course {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  // Inverse side - NO @JoinTable
  @ManyToMany(() => Student, student => student.courses)
  students: Student[];
}

// Database structure:
// students table: id, name
// courses table: id, title
// student_courses_course table: studentId, courseId (junction table created by @JoinTable)
```

### Key Points About @JoinTable

```typescript
/*
@JoinTable Characteristics:
1. REQUIRED for Many-to-Many relationships
2. Used on ONLY ONE side (owner side)
3. Creates the junction/join table automatically
4. Owner side can add/remove relationships
5. Inverse side can only read relationships
6. Without @JoinTable, Many-to-Many won't work

When to Use @JoinTable:
- Only with @ManyToMany relationships
- Not used with @OneToOne, @OneToMany, or @ManyToOne
- Place on the side that logically "owns" the relationship
*/
```

### Custom Junction Table Name

```typescript
@Entity()
export class Student {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @ManyToMany(() => Course, course => course.students)
  @JoinTable({
    name: 'student_enrollments', // Custom junction table name
  })
  courses: Course[];
}

// Database:
// student_enrollments table: studentId, courseId
```

### Custom Column Names

```typescript
@Entity()
export class Student {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @ManyToMany(() => Course, course => course.students)
  @JoinTable({
    name: 'student_enrollments', // Junction table name
    joinColumn: {
      name: 'student_id', // Column for Student FK
      referencedColumnName: 'id',
    },
    inverseJoinColumn: {
      name: 'course_id', // Column for Course FK
      referencedColumnName: 'id',
    },
  })
  courses: Course[];
}

// Database:
// student_enrollments table: student_id, course_id
```

### Complete Customization

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  username: string;

  @ManyToMany(() => Role, role => role.users)
  @JoinTable({
    name: 'user_role_assignments', // Custom table name
    joinColumn: {
      name: 'user_uuid', // Custom column name
      referencedColumnName: 'id', // References User.id
    },
    inverseJoinColumn: {
      name: 'role_identifier', // Custom column name
      referencedColumnName: 'id', // References Role.id
    },
  })
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

// Database:
// user_role_assignments table: user_uuid (UUID), role_identifier (INT)
```

### Choosing the Owner Side

```typescript
// Rule: The entity that is more likely to initiate the relationship should be the owner

// ✅ GOOD: User enrolls in courses (User is owner)
@Entity()
export class User {
  @ManyToMany(() => Course, course => course.users)
  @JoinTable()
  courses: Course[];
}

// ✅ GOOD: Post has tags (Post is owner)
@Entity()
export class Post {
  @ManyToMany(() => Tag, tag => tag.posts)
  @JoinTable()
  tags: Tag[];
}

// ✅ GOOD: Product belongs to categories (Product is owner)
@Entity()
export class Product {
  @ManyToMany(() => Category, category => category.products)
  @JoinTable()
  categories: Category[];
}

// ❌ BAD: Don't put @JoinTable on both sides
@Entity()
export class Student {
  @ManyToMany(() => Course)
  @JoinTable() // ❌ Error: both sides have @JoinTable
  courses: Course[];
}

@Entity()
export class Course {
  @ManyToMany(() => Student)
  @JoinTable() // ❌ Don't do this!
  students: Student[];
}
```

### Real-World Example: User Permissions

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  username: string;

  @Column()
  email: string;

  // User has many roles
  @ManyToMany(() => Role, role => role.users)
  @JoinTable({
    name: 'user_roles',
    joinColumn: { name: 'user_id' },
    inverseJoinColumn: { name: 'role_id' },
  })
  roles: Role[];

  // User has many permissions (directly, not through roles)
  @ManyToMany(() => Permission, permission => permission.users)
  @JoinTable({
    name: 'user_permissions',
    joinColumn: { name: 'user_id' },
    inverseJoinColumn: { name: 'permission_id' },
  })
  permissions: Permission[];
}

@Entity()
export class Role {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string; // 'admin', 'editor', 'viewer'

  @Column('text', { nullable: true })
  description: string;

  @ManyToMany(() => User, user => user.roles)
  users: User[];

  // Role has many permissions
  @ManyToMany(() => Permission, permission => permission.roles)
  @JoinTable({
    name: 'role_permissions',
    joinColumn: { name: 'role_id' },
    inverseJoinColumn: { name: 'permission_id' },
  })
  permissions: Permission[];
}

@Entity()
export class Permission {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string; // 'posts:create', 'posts:read', 'posts:update', 'posts:delete'

  @Column()
  resource: string; // 'posts', 'users', 'comments'

  @Column()
  action: string; // 'create', 'read', 'update', 'delete'

  @ManyToMany(() => Role, role => role.permissions)
  roles: Role[];

  @ManyToMany(() => User, user => user.permissions)
  users: User[];
}

// Service
class PermissionService {
  async getUserPermissions(userId: number): Promise<string[]> {
    const user = await userRepository.findOne({
      where: { id: userId },
      relations: ['roles', 'roles.permissions', 'permissions'],
    });

    if (!user) {
      return [];
    }

    // Combine permissions from roles and direct permissions
    const rolePermissions = user.roles.flatMap(role => role.permissions);
    const allPermissions = [...rolePermissions, ...user.permissions];

    // Remove duplicates
    const uniquePermissions = Array.from(
      new Set(allPermissions.map(p => p.name))
    );

    return uniquePermissions;
  }

  async userHasPermission(userId: number, permissionName: string): Promise<boolean> {
    const permissions = await this.getUserPermissions(userId);
    return permissions.includes(permissionName);
  }
}
```

### E-commerce: Products and Categories

```typescript
@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  sku: string;

  @Column('decimal', { precision: 10, scale: 2 })
  price: number;

  // Product can belong to multiple categories
  @ManyToMany(() => Category, category => category.products)
  @JoinTable({
    name: 'product_category_mapping',
    joinColumn: {
      name: 'product_id',
      referencedColumnName: 'id',
    },
    inverseJoinColumn: {
      name: 'category_id',
      referencedColumnName: 'id',
    },
  })
  categories: Category[];

  // Product has many tags
  @ManyToMany(() => Tag, tag => tag.products)
  @JoinTable({
    name: 'product_tags',
    joinColumn: { name: 'product_id' },
    inverseJoinColumn: { name: 'tag_id' },
  })
  tags: Tag[];
}

@Entity()
export class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  slug: string;

  @ManyToMany(() => Product, product => product.categories)
  products: Product[];
}

@Entity()
export class Tag {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ unique: true })
  name: string;

  @ManyToMany(() => Product, product => product.tags)
  products: Product[];
}

// Service
class ProductService {
  async assignCategories(productId: number, categoryIds: number[]): Promise<Product> {
    const product = await productRepository.findOne({
      where: { id: productId },
      relations: ['categories'],
    });

    const categories = await categoryRepository.findBy({
      id: In(categoryIds),
    });

    product.categories = categories;
    return await productRepository.save(product);
  }

  async addTag(productId: number, tagName: string): Promise<Product> {
    const product = await productRepository.findOne({
      where: { id: productId },
      relations: ['tags'],
    });

    let tag = await tagRepository.findOne({ where: { name: tagName } });

    if (!tag) {
      tag = new Tag();
      tag.name = tagName;
      tag = await tagRepository.save(tag);
    }

    if (!product.tags.find(t => t.id === tag.id)) {
      product.tags.push(tag);
      await productRepository.save(product);
    }

    return product;
  }
}
```

### When @JoinTable is NOT Used

```typescript
// ❌ NOT for One-to-One (uses @JoinColumn instead)
@Entity()
export class User {
  @OneToOne(() => Profile, profile => profile.user)
  @JoinColumn() // ✅ Use @JoinColumn, not @JoinTable
  profile: Profile;
}

// ❌ NOT for One-to-Many (no decorator needed)
@Entity()
export class User {
  @OneToMany(() => Post, post => post.author)
  posts: Post[]; // No @JoinTable
}

// ❌ NOT for Many-to-One (foreign key is automatic)
@Entity()
export class Post {
  @ManyToOne(() => User, user => user.posts)
  author: User; // No @JoinTable
}

// ✅ ONLY for Many-to-Many
@Entity()
export class Student {
  @ManyToMany(() => Course, course => course.students)
  @JoinTable() // ✅ Required for Many-to-Many
  courses: Course[];
}
```

### Composite Primary Keys in Junction Table

```typescript
@Entity()
export class User {
  @PrimaryColumn()
  tenantId: number;

  @PrimaryColumn()
  userId: number;

  @Column()
  name: string;

  @ManyToMany(() => Role, role => role.users)
  @JoinTable({
    name: 'user_roles',
    joinColumns: [
      { name: 'user_tenant_id', referencedColumnName: 'tenantId' },
      { name: 'user_id', referencedColumnName: 'userId' },
    ],
    inverseJoinColumns: [
      { name: 'role_tenant_id', referencedColumnName: 'tenantId' },
      { name: 'role_id', referencedColumnName: 'roleId' },
    ],
  })
  roles: Role[];
}

@Entity()
export class Role {
  @PrimaryColumn()
  tenantId: number;

  @PrimaryColumn()
  roleId: number;

  @Column()
  name: string;

  @ManyToMany(() => User, user => user.roles)
  users: User[];
}

// Database:
// user_roles table: user_tenant_id, user_id, role_tenant_id, role_id
```

### Migration for Junction Table

```typescript
import { MigrationInterface, QueryRunner, Table, TableForeignKey } from 'typeorm';

export class CreateJunctionTables1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    // Create junction table manually
    await queryRunner.createTable(
      new Table({
        name: 'student_courses',
        columns: [
          {
            name: 'student_id',
            type: 'int',
            isPrimary: true,
          },
          {
            name: 'course_id',
            type: 'int',
            isPrimary: true,
          },
        ],
      })
    );

    // Add foreign keys
    await queryRunner.createForeignKey(
      'student_courses',
      new TableForeignKey({
        columnNames: ['student_id'],
        referencedColumnNames: ['id'],
        referencedTableName: 'students',
        onDelete: 'CASCADE',
      })
    );

    await queryRunner.createForeignKey(
      'student_courses',
      new TableForeignKey({
        columnNames: ['course_id'],
        referencedColumnNames: ['id'],
        referencedTableName: 'courses',
        onDelete: 'CASCADE',
      })
    );

    // Create indexes for performance
    await queryRunner.query(`
      CREATE INDEX idx_student_courses_student ON student_courses (student_id)
    `);
    await queryRunner.query(`
      CREATE INDEX idx_student_courses_course ON student_courses (course_id)
    `);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropTable('student_courses');
  }
}
```

### Best Practices

```typescript
// ✅ GOOD: Use @JoinTable on the owner side
@Entity()
export class User {
  @ManyToMany(() => Role, role => role.users)
  @JoinTable() // User owns the relationship
  roles: Role[];
}

// ✅ GOOD: Customize junction table name for clarity
@ManyToMany(() => Course, course => course.students)
@JoinTable({ name: 'student_enrollments' })
courses: Course[];

// ✅ GOOD: Custom column names for readability
@ManyToMany(() => Course)
@JoinTable({
  name: 'enrollments',
  joinColumn: { name: 'student_id' },
  inverseJoinColumn: { name: 'course_id' },
})
courses: Course[];

// ❌ BAD: Don't put @JoinTable on both sides
@Entity()
export class Student {
  @ManyToMany(() => Course)
  @JoinTable()
  courses: Course[];
}

@Entity()
export class Course {
  @ManyToMany(() => Student)
  @JoinTable() // ❌ Error!
  students: Student[];
}

// ❌ BAD: Don't forget @JoinTable entirely
@Entity()
export class Student {
  @ManyToMany(() => Course)
  courses: Course[]; // ❌ Missing @JoinTable - won't work!
}

// ✅ GOOD: Choose logical owner (who initiates relationship)
// User enrolls in courses → User is owner
// Post has tags → Post is owner
// Product belongs to categories → Product is owner

// ✅ GOOD: Index junction table columns (in migration)
CREATE INDEX idx_student_courses_student ON student_courses (student_id);
CREATE INDEX idx_student_courses_course ON student_courses (course_id);
```

### When to Use Explicit Junction Entity Instead

```typescript
// ❌ BAD: Can't add extra fields to @JoinTable
@ManyToMany(() => Course)
@JoinTable()
courses: Course[];
// Cannot add enrollmentDate, grade, status to junction table

// ✅ GOOD: Create explicit junction entity
@Entity()
export class Enrollment {
  @PrimaryGeneratedColumn()
  id: number;

  @ManyToOne(() => Student, student => student.enrollments)
  student: Student;

  @ManyToOne(() => Course, course => course.enrollments)
  course: Course;

  // Now you can add extra fields
  @Column({ type: 'date' })
  enrollmentDate: Date;

  @Column({ type: 'decimal', precision: 3, scale: 2, nullable: true })
  grade: number | null;

  @Column()
  status: string;
}

// Use explicit entity when you need:
// - Additional data in junction table (dates, grades, status)
// - Complex queries on junction table
// - Business logic related to the association
// - Timestamps or audit fields
```

**Interview Tip**: Explain that `@JoinTable` is **required for Many-to-Many** relationships to create the junction table. Emphasize it's used on **only ONE side** (the owner side) and the owner can add/remove relationships. Discuss customization: `@JoinTable({ name: 'custom_name' })` for table name, `joinColumn` and `inverseJoinColumn` for custom column names. Highlight choosing owner: place on the entity that logically initiates the relationship (User enrolls in Courses → User is owner). Mention key limitation: **cannot add extra fields** to junction table created by `@JoinTable`; must create explicit junction entity instead. For production: index junction table columns for query performance, choose meaningful junction table names, use explicit junction entities when you need additional data. A strong answer demonstrates understanding of junction tables and when to use `@JoinTable` vs explicit junction entities.

</details>

<details>
<summary>33. How do you define bi-directional relationships?</summary>

**Bi-directional relationships** allow navigation in both directions: from parent to child and from child to parent. Both entities know about each other and can access the related entity.

### Basic Bi-directional One-to-Many

```typescript
import { Entity, PrimaryGeneratedColumn, Column, OneToMany, ManyToOne } from 'typeorm';

// Parent entity
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // One-to-Many side (inverse/parent side)
  @OneToMany(() => Post, post => post.author) // Second parameter points to inverse side
  posts: Post[];
}

// Child entity
@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  // Many-to-One side (owner/child side)
  @ManyToOne(() => User, user => user.posts) // Second parameter points back to parent
  author: User;
}

// Can navigate both ways:
// User → Posts: user.posts
// Post → User: post.author
```

### Key Points

```typescript
/*
Bi-directional Relationship Characteristics:
1. Both entities reference each other
2. Second parameter in decorator specifies the inverse side
3. Can navigate from either entity to the other
4. One side is the owner (has foreign key), other is inverse
5. Requires consistent mapping on both sides

Syntax Pattern:
@Relation(() => TargetEntity, targetEntity => targetEntity.inverseProperty)
                                             ↑
                           Points to the property on the other side
*/
```

### Bi-directional One-to-One

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Owner side (has @JoinColumn)
  @OneToOne(() => Profile, profile => profile.user, { cascade: true })
  @JoinColumn()
  profile: Profile;
}

@Entity()
export class Profile {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  bio: string;

  // Inverse side (no @JoinColumn)
  @OneToOne(() => User, user => user.profile)
  user: User;
}

// Usage
const user = await userRepository.findOne({
  where: { id: 1 },
  relations: ['profile'],
});
console.log(user.profile.bio); // User → Profile

const profile = await profileRepository.findOne({
  where: { id: 1 },
  relations: ['user'],
});
console.log(profile.user.name); // Profile → User
```

### Bi-directional Many-to-Many

```typescript
@Entity()
export class Student {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Owner side (has @JoinTable)
  @ManyToMany(() => Course, course => course.students)
  @JoinTable()
  courses: Course[];
}

@Entity()
export class Course {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  // Inverse side (no @JoinTable)
  @ManyToMany(() => Student, student => student.courses)
  students: Student[];
}

// Can navigate both ways:
// Student → Courses: student.courses
// Course → Students: course.students
```

### Uni-directional vs Bi-directional

```typescript
// Uni-directional: Only parent knows about child
@Entity()
export class User {
  @OneToMany(() => Post, post => post.author)
  posts: Post[];
}

@Entity()
export class Post {
  @ManyToOne(() => User)
  author: User;
  // No reference back to User.posts
}
// Can access: user.posts ✅
// Cannot access: post.author.posts ❌

// Bi-directional: Both know about each other
@Entity()
export class User {
  @OneToMany(() => Post, post => post.author)
  posts: Post[];
}

@Entity()
export class Post {
  @ManyToOne(() => User, user => user.posts) // Points back!
  author: User;
}
// Can access: user.posts ✅
// Can access: post.author.posts ✅
```

### Real-World Example: Blog System

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  username: string;

  @Column()
  email: string;

  // Bi-directional: User has many posts
  @OneToMany(() => Post, post => post.author)
  posts: Post[];

  // Bi-directional: User has many comments
  @OneToMany(() => Comment, comment => comment.author)
  comments: Comment[];

  @CreateDateColumn()
  createdAt: Date;
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column('text')
  content: string;

  // Bi-directional: Post belongs to author
  @ManyToOne(() => User, user => user.posts, { eager: true })
  author: User;

  // Bi-directional: Post has many comments
  @OneToMany(() => Comment, comment => comment.post)
  comments: Comment[];

  // Bi-directional: Post has many tags
  @ManyToMany(() => Tag, tag => tag.posts)
  @JoinTable()
  tags: Tag[];

  @CreateDateColumn()
  createdAt: Date;
}

@Entity()
export class Comment {
  @PrimaryGeneratedColumn()
  id: number;

  @Column('text')
  content: string;

  // Bi-directional: Comment by author
  @ManyToOne(() => User, user => user.comments)
  author: User;

  // Bi-directional: Comment on post
  @ManyToOne(() => Post, post => post.comments, { onDelete: 'CASCADE' })
  post: Post;

  // Self-referencing bi-directional: Comment replies
  @ManyToOne(() => Comment, comment => comment.replies, { nullable: true })
  parentComment: Comment | null;

  @OneToMany(() => Comment, comment => comment.parentComment)
  replies: Comment[];

  @CreateDateColumn()
  createdAt: Date;
}

@Entity()
export class Tag {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Bi-directional: Tag belongs to many posts
  @ManyToMany(() => Post, post => post.tags)
  posts: Post[];
}

// Service demonstrating bi-directional navigation
class BlogService {
  async getPostWithAuthorAndComments(postId: number): Promise<Post> {
    const post = await postRepository.findOne({
      where: { id: postId },
      relations: ['author', 'comments', 'comments.author', 'tags'],
    });

    // Navigate bi-directionally
    console.log(`Post: ${post.title}`);
    console.log(`Author: ${post.author.username}`); // Post → Author
    console.log(`Author's posts: ${post.author.posts.length}`); // Author → Posts

    post.comments.forEach(comment => {
      console.log(`Comment by ${comment.author.username}`); // Comment → Author
    });

    return post;
  }

  async getUserActivity(userId: number): Promise<any> {
    const user = await userRepository.findOne({
      where: { id: userId },
      relations: ['posts', 'comments', 'comments.post'],
    });

    return {
      username: user.username,
      postsCount: user.posts.length,
      commentsCount: user.comments.length,
      posts: user.posts.map(post => ({
        title: post.title,
        commentsCount: post.comments?.length || 0, // Post → Comments
      })),
    };
  }
}
```

### E-commerce Order System

```typescript
@Entity()
export class Customer {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  email: string;

  // Bi-directional: Customer has many orders
  @OneToMany(() => Order, order => order.customer)
  orders: Order[];

  // Bi-directional: Customer has many addresses
  @OneToMany(() => Address, address => address.customer, { cascade: true })
  addresses: Address[];
}

@Entity()
export class Order {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  orderNumber: string;

  @Column('decimal', { precision: 10, scale: 2 })
  total: number;

  // Bi-directional: Order belongs to customer
  @ManyToOne(() => Customer, customer => customer.orders)
  customer: Customer;

  // Bi-directional: Order has many items
  @OneToMany(() => OrderItem, item => item.order, { cascade: true })
  items: OrderItem[];

  @CreateDateColumn()
  createdAt: Date;
}

@Entity()
export class OrderItem {
  @PrimaryGeneratedColumn()
  id: number;

  @Column('int')
  quantity: number;

  @Column('decimal', { precision: 10, scale: 2 })
  price: number;

  // Bi-directional: Item belongs to order
  @ManyToOne(() => Order, order => order.items)
  order: Order;

  // Bi-directional: Item references product
  @ManyToOne(() => Product, product => product.orderItems)
  product: Product;
}

@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column('decimal', { precision: 10, scale: 2 })
  price: number;

  // Bi-directional: Product appears in many order items
  @OneToMany(() => OrderItem, item => item.product)
  orderItems: OrderItem[];
}

@Entity()
export class Address {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  street: string;

  @Column()
  city: string;

  @Column()
  zipCode: string;

  // Bi-directional: Address belongs to customer
  @ManyToOne(() => Customer, customer => customer.addresses)
  customer: Customer;
}

// Service using bi-directional navigation
class OrderService {
  async getOrderDetails(orderId: number): Promise<any> {
    const order = await orderRepository.findOne({
      where: { id: orderId },
      relations: ['customer', 'items', 'items.product'],
    });

    return {
      orderNumber: order.orderNumber,
      customer: {
        name: order.customer.name, // Order → Customer
        email: order.customer.email,
        totalOrders: order.customer.orders?.length || 0, // Customer → Orders
      },
      items: order.items.map(item => ({
        product: item.product.name, // Item → Product
        quantity: item.quantity,
        price: item.price,
      })),
      total: order.total,
    };
  }
}
```

### Self-Referencing Bi-directional

```typescript
@Entity()
export class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Bi-directional: Parent category
  @ManyToOne(() => Category, category => category.children, { nullable: true })
  parent: Category | null;

  // Bi-directional: Child categories
  @OneToMany(() => Category, category => category.parent)
  children: Category[];
}

// Usage
const parentCategory = await categoryRepository.findOne({
  where: { id: 1 },
  relations: ['children', 'children.children'], // Load hierarchy
});

console.log(`Category: ${parentCategory.name}`);
parentCategory.children.forEach(child => {
  console.log(`- Subcategory: ${child.name}`);
  console.log(`  Parent: ${child.parent.name}`); // Child → Parent
});

// Employee hierarchy
@Entity()
export class Employee {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Bi-directional: Manager relationship
  @ManyToOne(() => Employee, employee => employee.subordinates, { nullable: true })
  manager: Employee | null;

  // Bi-directional: Subordinates
  @OneToMany(() => Employee, employee => employee.manager)
  subordinates: Employee[];
}
```

### Querying Bi-directional Relationships

```typescript
// Load with relations
const user = await userRepository.findOne({
  where: { id: 1 },
  relations: ['posts', 'posts.comments'],
});

// Navigate: User → Posts → Comments
user.posts.forEach(post => {
  console.log(`${post.title} has ${post.comments.length} comments`);
});

// Query builder with multiple levels
const posts = await postRepository
  .createQueryBuilder('post')
  .leftJoinAndSelect('post.author', 'author')
  .leftJoinAndSelect('post.comments', 'comment')
  .leftJoinAndSelect('comment.author', 'commentAuthor')
  .where('post.id = :id', { id: 1 })
  .getOne();

// Navigate: Post → Author → Posts (circular)
console.log(`Author ${posts.author.username} has ${posts.author.posts.length} posts`);

// Inverse navigation: Comment → Post → Comments
const comment = await commentRepository.findOne({
  where: { id: 1 },
  relations: ['post', 'post.comments'],
});

console.log(`This post has ${comment.post.comments.length} total comments`);
```

### Best Practices

```typescript
// ✅ GOOD: Always specify inverse side for bi-directional
@OneToMany(() => Post, post => post.author) // Second parameter: inverse
posts: Post[];

@ManyToOne(() => User, user => user.posts) // Second parameter: points back
author: User;

// ❌ BAD: Missing inverse side specification
@OneToMany(() => Post)
posts: Post[]; // ❌ Not truly bi-directional

// ✅ GOOD: Consistent mapping on both sides
@Entity()
export class User {
  @OneToMany(() => Post, post => post.author)
  posts: Post[];
}

@Entity()
export class Post {
  @ManyToOne(() => User, user => user.posts)
  author: User;
}

// ❌ BAD: Inconsistent mapping
@Entity()
export class User {
  @OneToMany(() => Post, post => post.writer) // ❌ 'writer' doesn't exist
  posts: Post[];
}

@Entity()
export class Post {
  @ManyToOne(() => User, user => user.posts)
  author: User; // Property name doesn't match
}

// ✅ GOOD: Use bi-directional for frequently navigated relationships
// User ↔ Posts: Need to access both user.posts and post.author

// ✅ GOOD: Use uni-directional for rarely navigated inverse
// Only need post.category, never category.posts

// ✅ GOOD: Be careful with circular loading
@OneToMany(() => Post, post => post.author, {
  eager: false, // Don't auto-load to avoid circular issues
})
posts: Post[];
```

**Interview Tip**: Explain that bi-directional relationships allow navigation in both directions by having both entities reference each other. Emphasize the **second parameter** in relationship decorators specifies the inverse side: `@OneToMany(() => Post, post => post.author)`. The second parameter points to the property name on the other entity. Highlight advantages: can navigate both ways (user.posts and post.author), better for complex queries with multiple levels. Mention that one side is still the owner (has FK or @JoinTable), the other is inverse. For production: ensure consistent mapping on both sides, avoid eager loading on both sides (circular loading issues), use bi-directional only when navigation is needed both ways. A strong answer demonstrates understanding of the inverse side specification and when bi-directional vs uni-directional is appropriate.

</details>

<details>
<summary>34. What is cascade in relationships and what are its options?</summary>

**Cascade** defines automatic operations that are performed on related entities when an operation is performed on the main entity. It allows you to automatically save, update, or remove related entities without explicit calls.

### Basic Cascade Usage

```typescript
import { Entity, PrimaryGeneratedColumn, Column, OneToMany, ManyToOne } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Cascade: true enables all cascade operations
  @OneToMany(() => Post, post => post.author, {
    cascade: true, // Auto-save, auto-update, auto-remove posts
  })
  posts: Post[];
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToOne(() => User, user => user.posts)
  author: User;
}

// Usage
const user = new User();
user.name = 'John';

const post = new Post();
post.title = 'My Post';

user.posts = [post];

await userRepository.save(user); // Cascade automatically saves post!
```

### Cascade Options

```typescript
/*
Cascade Options:
1. "insert" / "create" - Insert related entities when parent is inserted
2. "update" - Update related entities when parent is updated
3. "remove" - Remove related entities when parent is removed
4. "soft-remove" - Soft-remove related entities when parent is soft-removed
5. "recover" - Recover related entities when parent is recovered
6. true - Enable all cascade operations (shorthand for all above)
7. false - Disable all cascade operations (default)

Syntax:
cascade: true                           // All operations
cascade: ['insert', 'update']          // Specific operations
cascade: ['insert', 'update', 'remove'] // Multiple operations
*/
```

### Individual Cascade Operations

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Only cascade insert and update
  @OneToMany(() => Post, post => post.author, {
    cascade: ['insert', 'update'], // Not 'remove'
  })
  posts: Post[];
}

// Cascade insert
const user = new User();
user.name = 'John';

const post = new Post();
post.title = 'New Post';

user.posts = [post];
await userRepository.save(user); // ✅ Post is inserted (cascade: insert)

// Cascade update
user.posts[0].title = 'Updated Title';
await userRepository.save(user); // ✅ Post is updated (cascade: update)

// NO cascade remove
await userRepository.remove(user); // ❌ Posts are NOT removed (cascade: remove not enabled)
```

### Cascade Insert

```typescript
@Entity()
export class Order {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  orderNumber: string;

  // Cascade insert only
  @OneToMany(() => OrderItem, item => item.order, {
    cascade: ['insert'], // Auto-save items when order is created
  })
  items: OrderItem[];
}

@Entity()
export class OrderItem {
  @PrimaryGeneratedColumn()
  id: number;

  @Column('int')
  quantity: number;

  @ManyToOne(() => Order, order => order.items)
  order: Order;
}

// Usage
const order = new Order();
order.orderNumber = 'ORD-001';

const item1 = new OrderItem();
item1.quantity = 2;

const item2 = new OrderItem();
item2.quantity = 5;

order.items = [item1, item2];

await orderRepository.save(order); // Both items are automatically inserted
```

### Cascade Update

```typescript
@Entity()
export class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Cascade update
  @OneToMany(() => Product, product => product.category, {
    cascade: ['update'], // Auto-update products when category is updated
  })
  products: Product[];
}

@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @ManyToOne(() => Category, category => category.products)
  category: Category;
}

// Usage
const category = await categoryRepository.findOne({
  where: { id: 1 },
  relations: ['products'],
});

category.name = 'Updated Category';
category.products[0].name = 'Updated Product';

await categoryRepository.save(category); // Both category and product are updated
```

### Cascade Remove

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Cascade remove
  @OneToMany(() => Post, post => post.author, {
    cascade: ['remove'], // Delete posts when user is deleted
  })
  posts: Post[];
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToOne(() => User, user => user.posts)
  author: User;
}

// Usage
const user = await userRepository.findOne({
  where: { id: 1 },
  relations: ['posts'],
});

await userRepository.remove(user); // User AND all posts are deleted
```

### Cascade True (All Operations)

```typescript
@Entity()
export class Customer {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Cascade: true enables all operations
  @OneToMany(() => Address, address => address.customer, {
    cascade: true, // insert, update, remove, soft-remove, recover
  })
  addresses: Address[];
}

@Entity()
export class Address {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  street: string;

  @ManyToOne(() => Customer, customer => customer.addresses)
  customer: Customer;

  @DeleteDateColumn()
  deletedAt: Date | null;
}

// All operations cascade
const customer = new Customer();
customer.name = 'John';

const address = new Address();
address.street = '123 Main St';

customer.addresses = [address];

await customerRepository.save(customer); // ✅ Insert cascades

customer.addresses[0].street = '456 Oak Ave';
await customerRepository.save(customer); // ✅ Update cascades

await customerRepository.softRemove(customer); // ✅ Soft-remove cascades

await customerRepository.recover(customer); // ✅ Recover cascades

await customerRepository.remove(customer); // ✅ Remove cascades
```

### Real-World Example: Blog System

```typescript
@Entity()
export class Blog {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  // Full cascade for posts
  @OneToMany(() => Post, post => post.blog, {
    cascade: true, // All operations
  })
  posts: Post[];

  @CreateDateColumn()
  createdAt: Date;
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column('text')
  content: string;

  @ManyToOne(() => Blog, blog => blog.posts)
  blog: Blog;

  // Cascade insert and update for comments
  @OneToMany(() => Comment, comment => comment.post, {
    cascade: ['insert', 'update'], // Not remove - keep comments if post deleted
  })
  comments: Comment[];

  @DeleteDateColumn()
  deletedAt: Date | null;
}

@Entity()
export class Comment {
  @PrimaryGeneratedColumn()
  id: number;

  @Column('text')
  content: string;

  @ManyToOne(() => Post, post => post.comments)
  post: Post;

  @CreateDateColumn()
  createdAt: Date;
}

// Service
class BlogService {
  async createBlogWithPosts(): Promise<Blog> {
    const blog = new Blog();
    blog.title = 'My Tech Blog';

    const post1 = new Post();
    post1.title = 'First Post';
    post1.content = 'Hello World';

    const post2 = new Post();
    post2.title = 'Second Post';
    post2.content = 'More content';

    const comment1 = new Comment();
    comment1.content = 'Great post!';

    post1.comments = [comment1];
    blog.posts = [post1, post2];

    // All nested entities saved automatically due to cascade
    return await blogRepository.save(blog);
  }

  async deleteBlog(blogId: number): Promise<void> {
    const blog = await blogRepository.findOne({
      where: { id: blogId },
      relations: ['posts'],
    });

    // Deletes blog and all posts (cascade: true)
    // Comments remain (no cascade: remove on Post)
    await blogRepository.remove(blog);
  }
}
```

### E-commerce Order Example

```typescript
@Entity()
export class Order {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  orderNumber: string;

  @Column('decimal', { precision: 10, scale: 2 })
  total: number;

  // Full cascade for order items (lifecycle tied to order)
  @OneToMany(() => OrderItem, item => item.order, {
    cascade: true,
  })
  items: OrderItem[];

  // No cascade for customer (independent entity)
  @ManyToOne(() => Customer, customer => customer.orders)
  customer: Customer;

  @CreateDateColumn()
  createdAt: Date;
}

@Entity()
export class OrderItem {
  @PrimaryGeneratedColumn()
  id: number;

  @Column('int')
  quantity: number;

  @Column('decimal', { precision: 10, scale: 2 })
  price: number;

  @ManyToOne(() => Order, order => order.items)
  order: Order;

  // No cascade for product (independent entity)
  @ManyToOne(() => Product, product => product.orderItems, {
    eager: true,
  })
  product: Product;
}

@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column('decimal', { precision: 10, scale: 2 })
  price: number;

  @OneToMany(() => OrderItem, item => item.product)
  orderItems: OrderItem[];
}

@Entity()
export class Customer {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Order, order => order.customer)
  orders: Order[];
}

// Service
class OrderService {
  async createOrder(customerId: number, items: any[]): Promise<Order> {
    const customer = await customerRepository.findOneBy({ id: customerId });

    const order = new Order();
    order.orderNumber = `ORD-${Date.now()}`;
    order.customer = customer;

    const orderItems = await Promise.all(
      items.map(async item => {
        const product = await productRepository.findOneBy({ id: item.productId });

        const orderItem = new OrderItem();
        orderItem.product = product;
        orderItem.quantity = item.quantity;
        orderItem.price = product.price;

        return orderItem;
      })
    );

    order.items = orderItems;
    order.total = orderItems.reduce(
      (sum, item) => sum + item.price * item.quantity,
      0
    );

    // Cascade saves all order items automatically
    return await orderRepository.save(order);
  }

  async cancelOrder(orderId: number): Promise<void> {
    const order = await orderRepository.findOne({
      where: { id: orderId },
      relations: ['items'],
    });

    // Cascade removes all order items automatically
    await orderRepository.remove(order);
  }
}
```

### Cascade with Soft Delete

```typescript
@Entity()
export class Project {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Cascade soft-remove and recover
  @OneToMany(() => Task, task => task.project, {
    cascade: ['soft-remove', 'recover'],
  })
  tasks: Task[];

  @DeleteDateColumn()
  deletedAt: Date | null;
}

@Entity()
export class Task {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToOne(() => Project, project => project.tasks)
  project: Project;

  @DeleteDateColumn()
  deletedAt: Date | null;
}

// Usage
const project = await projectRepository.findOne({
  where: { id: 1 },
  relations: ['tasks'],
});

// Soft-remove cascades to tasks
await projectRepository.softRemove(project);
// project.deletedAt and all task.deletedAt are set

// Recover cascades to tasks
await projectRepository.recover(project);
// project.deletedAt and all task.deletedAt are set to null
```

### Cascade in Different Relationship Types

```typescript
// One-to-One with cascade
@Entity()
export class User {
  @OneToOne(() => Profile, profile => profile.user, {
    cascade: true,
  })
  @JoinColumn()
  profile: Profile;
}

// Many-to-Many with cascade
@Entity()
export class Student {
  @ManyToMany(() => Course, course => course.students, {
    cascade: ['insert'], // Only cascade insert, not remove
  })
  @JoinTable()
  courses: Course[];
}

// Many-to-One with cascade (rare, usually not needed)
@Entity()
export class Post {
  @ManyToOne(() => User, user => user.posts, {
    cascade: true, // Rarely used - would save user when saving post
  })
  author: User;
}
```

### Best Practices

```typescript
// ✅ GOOD: Use cascade for dependent entities
@OneToMany(() => OrderItem, item => item.order, {
  cascade: true, // Order items depend on order
})
items: OrderItem[];

// ✅ GOOD: Don't cascade to independent entities
@ManyToOne(() => User, user => user.posts)
author: User; // User is independent, no cascade

// ✅ GOOD: Use specific operations instead of true
@OneToMany(() => Post, post => post.author, {
  cascade: ['insert', 'update'], // Explicit control
})
posts: Post[];

// ❌ BAD: Cascading to independent entities
@ManyToOne(() => Category, category => category.products, {
  cascade: true, // ❌ Would save category when saving product
})
category: Category;

// ✅ GOOD: Cascade for composition relationships
// Order → OrderItems (composition: items don't exist without order)
@OneToMany(() => OrderItem, item => item.order, {
  cascade: true,
})
items: OrderItem[];

// ❌ BAD: Cascade for association relationships
// Post → Author (association: author exists independently)
@ManyToOne(() => User, user => user.posts, {
  cascade: true, // ❌ Bad: saves user when saving post
})
author: User;

// ✅ GOOD: Cascade remove for dependent data
@OneToMany(() => Comment, comment => comment.post, {
  cascade: ['remove'], // Delete comments when post is deleted
})
comments: Comment[];

// ✅ GOOD: No cascade remove for audit trail
@OneToMany(() => AuditLog, log => log.user, {
  cascade: false, // Keep audit logs even if user is deleted
})
auditLogs: AuditLog[];
```

**Interview Tip**: Explain that cascade defines automatic operations on related entities (insert, update, remove, soft-remove, recover). Emphasize `cascade: true` enables all operations, or use array for specific ones: `cascade: ['insert', 'update']`. Discuss when to use: cascade for **dependent/composed** entities (OrderItems belong to Order), **don't cascade** for independent/associated entities (Post's Author exists independently). Highlight cascade remove: use for data that should be deleted with parent, avoid for data that should persist (audit logs). Mention soft-delete cascade: `soft-remove` and `recover` cascade deletedAt column changes. For production: prefer explicit cascade array over true, cascade for composition relationships (parent-child), never cascade to shared/independent entities. A strong answer demonstrates understanding of entity lifecycle dependencies and composition vs association patterns.

</details>


<details>
<summary>35. What is eager loading vs lazy loading in TypeORM?</summary>

**Eager loading** automatically loads related entities whenever the main entity is loaded, while **lazy loading** loads related entities only when explicitly accessed. This affects query performance and data retrieval patterns.

### Eager Loading

```typescript
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne, OneToMany } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Eager loading: posts loaded automatically
  @OneToMany(() => Post, post => post.author, {
    eager: true, // Always load posts with user
  })
  posts: Post[];
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  // Eager loading: author loaded automatically
  @ManyToOne(() => User, user => user.posts, {
    eager: true, // Always load author with post
  })
  author: User;
}

// Usage: relations loaded automatically
const user = await userRepository.findOne({ where: { id: 1 } });
console.log(user.posts); // ✅ Posts already loaded (no additional query needed)

const post = await postRepository.findOne({ where: { id: 1 } });
console.log(post.author.name); // ✅ Author already loaded
```

### Lazy Loading (Default)

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Lazy loading (default): posts NOT loaded automatically
  @OneToMany(() => Post, post => post.author) // No eager: true
  posts: Post[];
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToOne(() => User, user => user.posts) // No eager: true
  author: User;
}

// Must explicitly load relations
const user = await userRepository.findOne({ where: { id: 1 } });
console.log(user.posts); // ❌ undefined - not loaded

// Option 1: Use relations
const userWithPosts = await userRepository.findOne({
  where: { id: 1 },
  relations: ['posts'], // Explicitly load
});
console.log(userWithPosts.posts); // ✅ Posts loaded

// Option 2: Use query builder
const userWithPosts2 = await userRepository
  .createQueryBuilder('user')
  .leftJoinAndSelect('user.posts', 'posts')
  .where('user.id = :id', { id: 1 })
  .getOne();
console.log(userWithPosts2.posts); // ✅ Posts loaded
```

### Key Differences

```typescript
/*
EAGER LOADING:
✅ Automatic: Relations loaded without specifying
✅ Convenient: No need to remember to load relations
✅ Consistent: Always have complete data
❌ Performance: Always loads relations (even when not needed)
❌ N+1 Problem: Can cause multiple queries
❌ Can't disable: Always loads (except with Query Builder)

LAZY LOADING:
✅ Performance: Only loads when needed
✅ Flexible: Choose when to load
✅ Control: Avoid unnecessary data
❌ Manual: Must remember to load relations
❌ Inconsistent: May forget to load needed data
❌ Extra queries: Need separate query to load

RULE OF THUMB:
- Use eager for: Small, frequently needed relations
- Use lazy for: Large collections, conditionally needed data
*/
```

### Eager Loading Examples

```typescript
@Entity()
export class Order {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  orderNumber: string;

  // Eager: Always need customer info
  @ManyToOne(() => Customer, customer => customer.orders, {
    eager: true,
  })
  customer: Customer;

  // Eager: Always need items for order
  @OneToMany(() => OrderItem, item => item.order, {
    eager: true,
  })
  items: OrderItem[];
}

@Entity()
export class OrderItem {
  @PrimaryGeneratedColumn()
  id: number;

  @Column('int')
  quantity: number;

  // Eager: Always need product info
  @ManyToOne(() => Product, product => product.orderItems, {
    eager: true,
  })
  product: Product;

  @ManyToOne(() => Order, order => order.items)
  order: Order;
}

@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column('decimal', { precision: 10, scale: 2 })
  price: number;

  @OneToMany(() => OrderItem, item => item.product)
  orderItems: OrderItem[];
}

@Entity()
export class Customer {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Order, order => order.customer)
  orders: Order[];
}

// Usage: everything loads automatically
const order = await orderRepository.findOne({ where: { id: 1 } });

console.log(order.customer.name); // ✅ Customer loaded
console.log(order.items.length); // ✅ Items loaded
console.log(order.items[0].product.name); // ✅ Products loaded
// All data available without specifying relations!
```

### Lazy Loading with Manual Loading

```typescript
@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column('text')
  content: string;

  // Lazy: Author loaded when needed
  @ManyToOne(() => User, user => user.posts)
  author: User;

  // Lazy: Comments loaded when needed
  @OneToMany(() => Comment, comment => comment.post)
  comments: Comment[];

  // Lazy: Tags loaded when needed
  @ManyToMany(() => Tag, tag => tag.posts)
  @JoinTable()
  tags: Tag[];
}

// Service with conditional loading
class PostService {
  async getPost(id: number, includeComments: boolean = false): Promise<Post> {
    const relations = ['author', 'tags']; // Always load author and tags

    if (includeComments) {
      relations.push('comments', 'comments.author'); // Conditionally load comments
    }

    return await postRepository.findOne({
      where: { id },
      relations,
    });
  }

  async getPostsForListing(): Promise<Post[]> {
    // List view: only basic info and author (no comments, no tags)
    return await postRepository.find({
      relations: ['author'],
      select: ['id', 'title', 'createdAt'],
    });
  }

  async getPostForDetailView(id: number): Promise<Post> {
    // Detail view: load everything
    return await postRepository.findOne({
      where: { id },
      relations: ['author', 'comments', 'comments.author', 'tags'],
    });
  }
}
```

### Performance Comparison

```typescript
// Eager loading performance impact
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Post, post => post.author, {
    eager: true, // Always loads posts
  })
  posts: Post[];
}

// List users: loads ALL posts for ALL users (expensive!)
const users = await userRepository.find();
// Query 1: SELECT * FROM users
// Query 2: SELECT * FROM posts WHERE authorId IN (1, 2, 3, ..., 100)
// Loads potentially thousands of posts even if not needed

// ✅ BETTER: Lazy loading with selective loading
@Entity()
export class User {
  @OneToMany(() => Post, post => post.author) // No eager
  posts: Post[];
}

// Only load users (fast)
const users = await userRepository.find();

// Load posts only when needed
const userWithPosts = await userRepository.findOne({
  where: { id: 1 },
  relations: ['posts'],
});
```

### Disabling Eager Loading

```typescript
@Entity()
export class Post {
  @ManyToOne(() => User, user => user.posts, {
    eager: true,
  })
  author: User;
}

// Find with eager loading disabled
const posts = await postRepository
  .createQueryBuilder('post')
  .getMany(); // ✅ Eager loading is disabled in Query Builder

// Eager loading still works with find()
const posts2 = await postRepository.find(); // Author loaded automatically
```

### Real-World Example: E-commerce Product

```typescript
@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column('text')
  description: string;

  @Column('decimal', { precision: 10, scale: 2 })
  price: number;

  // Eager: Always show category in product
  @ManyToOne(() => Category, category => category.products, {
    eager: true,
  })
  category: Category;

  // Lazy: Reviews only on detail page
  @OneToMany(() => Review, review => review.product)
  reviews: Review[];

  // Lazy: Order items for admin only
  @OneToMany(() => OrderItem, item => item.product)
  orderItems: OrderItem[];

  // Eager: Images always needed
  @OneToMany(() => ProductImage, image => image.product, {
    eager: true,
  })
  images: ProductImage[];
}

@Entity()
export class ProductImage {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  url: string;

  @Column()
  isPrimary: boolean;

  @ManyToOne(() => Product, product => product.images)
  product: Product;
}

@Entity()
export class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Product, product => product.category)
  products: Product[];
}

@Entity()
export class Review {
  @PrimaryGeneratedColumn()
  id: number;

  @Column('int')
  rating: number;

  @Column('text')
  comment: string;

  @ManyToOne(() => Product, product => product.reviews)
  product: Product;

  @ManyToOne(() => User, user => user.reviews, {
    eager: true, // Always show reviewer
  })
  user: User;
}

// Service
class ProductService {
  async getProductList(): Promise<Product[]> {
    // List: category and images loaded automatically (eager)
    const products = await productRepository.find({
      select: ['id', 'name', 'price'],
    });

    return products;
    // Each product has: category ✅, images ✅
    // Reviews NOT loaded (lazy)
  }

  async getProductDetail(id: number): Promise<Product> {
    // Detail: Load reviews explicitly
    const product = await productRepository.findOne({
      where: { id },
      relations: ['reviews', 'reviews.user'],
    });

    return product;
    // Product has: category ✅, images ✅, reviews ✅
  }

  async getProductForAdmin(id: number): Promise<Product> {
    // Admin: Load order history
    const product = await productRepository.findOne({
      where: { id },
      relations: ['reviews', 'orderItems', 'orderItems.order'],
    });

    return product;
  }
}
```

### N+1 Query Problem with Eager Loading

```typescript
@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToOne(() => User, user => user.posts, {
    eager: true, // Causes N+1 problem
  })
  author: User;
}

// N+1 Problem
const posts = await postRepository.find(); // 100 posts
// Query 1: SELECT * FROM posts (1 query)
// Query 2-101: SELECT * FROM users WHERE id = 1 (N queries for each post)
// Total: 101 queries!

// ✅ SOLUTION 1: Use Query Builder with join
const posts = await postRepository
  .createQueryBuilder('post')
  .leftJoinAndSelect('post.author', 'author') // Single JOIN query
  .getMany();
// Total: 1 query with JOIN

// ✅ SOLUTION 2: Use find() with relations
const posts2 = await postRepository.find({
  relations: ['author'], // Single JOIN query
});
// Total: 1 query with JOIN
```

### Mixing Eager and Lazy

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Eager: Profile always needed
  @OneToOne(() => Profile, profile => profile.user, {
    eager: true,
    cascade: true,
  })
  @JoinColumn()
  profile: Profile;

  // Lazy: Posts loaded conditionally
  @OneToMany(() => Post, post => post.author)
  posts: Post[];

  // Lazy: Comments loaded conditionally
  @OneToMany(() => Comment, comment => comment.author)
  comments: Comment[];
}

// Usage
const user = await userRepository.findOne({ where: { id: 1 } });
console.log(user.profile.bio); // ✅ Profile loaded automatically
console.log(user.posts); // ❌ undefined (lazy)

// Load posts when needed
const userWithPosts = await userRepository.findOne({
  where: { id: 1 },
  relations: ['posts'],
});
console.log(userWithPosts.profile.bio); // ✅ Profile loaded (eager)
console.log(userWithPosts.posts.length); // ✅ Posts loaded (explicit)
```

### Best Practices

```typescript
// ✅ GOOD: Eager for small, always-needed relations
@ManyToOne(() => Category, category => category.products, {
  eager: true, // Small lookup table
})
category: Category;

// ✅ GOOD: Lazy for large collections
@OneToMany(() => Post, post => post.author)
posts: Post[]; // Could be thousands

// ❌ BAD: Eager for large collections
@OneToMany(() => Post, post => post.author, {
  eager: true, // ❌ Loads all posts always
})
posts: Post[];

// ✅ GOOD: Eager for parent references
@ManyToOne(() => User, user => user.comments, {
  eager: true, // Always need user info
})
author: User;

// ❌ BAD: Eager on both sides of bi-directional
@Entity()
export class User {
  @OneToMany(() => Post, post => post.author, {
    eager: true, // ❌ Circular loading
  })
  posts: Post[];
}

@Entity()
export class Post {
  @ManyToOne(() => User, user => user.posts, {
    eager: true, // ❌ Circular loading
  })
  author: User;
}

// ✅ GOOD: Eager on one side only
@Entity()
export class Post {
  @ManyToOne(() => User, user => user.posts, {
    eager: true, // ✅ Only on Many side
  })
  author: User;
}

// ✅ GOOD: Conditional loading with lazy
async getUsers(includeStats: boolean) {
  const relations = [];
  if (includeStats) {
    relations.push('posts', 'comments');
  }
  return await userRepository.find({ relations });
}

// ✅ GOOD: Use Query Builder to bypass eager
const users = await userRepository
  .createQueryBuilder('user')
  .getMany(); // Eager loading disabled
```

**Interview Tip**: Explain that **eager loading** automatically loads relations with every query (convenient but can impact performance), while **lazy loading** only loads when explicitly requested (more control but requires manual specification). Emphasize eager is set with `eager: true` in relationship decorator, lazy is the default behavior. Discuss tradeoffs: eager is convenient for small, always-needed data (user profile, category), lazy is better for large collections or conditionally-needed data (user's 1000 posts). Highlight **N+1 problem**: eager loading can cause multiple queries (1 for main entity + N for each relation); solve with Query Builder joins. Mention that eager loading is bypassed in Query Builder but works with `find()`. For production: avoid eager on both sides of bi-directional relationships (circular loading), use eager sparingly for lookup tables only, prefer lazy with explicit loading for flexibility. A strong answer demonstrates understanding of performance implications and when to use each approach.

</details>

<details>
<summary>36. How do you implement lazy relationships?</summary>

**Lazy relationships** in TypeORM load related entities only when accessed, using **Promises** or **explicit loading**. TypeORM supports lazy loading through Promise-wrapped properties or manual loading with repository methods.

### Promise-based Lazy Loading

```typescript
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne, OneToMany } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Promise-based lazy loading
  @OneToMany(() => Post, post => post.author)
  posts: Promise<Post[]>; // Note: Promise<Post[]> instead of Post[]
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToOne(() => User, user => user.posts)
  author: Promise<User>; // Promise<User> instead of User
}

// Usage: await the property
const user = await userRepository.findOne({ where: { id: 1 } });

// Access lazy property (triggers database query)
const posts = await user.posts; // Returns Post[]
console.log(posts.length);

const post = await postRepository.findOne({ where: { id: 1 } });
const author = await post.author; // Returns User
console.log(author.name);
```

### Important Note: Promise Lazy Loading Limitations

```typescript
/*
WARNING: Promise-based lazy loading has limitations in TypeORM:
1. Not fully supported in newer TypeORM versions
2. Can cause TypeScript type issues
3. Doesn't work well with JSON serialization
4. Not recommended for production use

RECOMMENDED APPROACH: Use explicit loading instead
- Use find() with relations option
- Use Query Builder
- Use RelationQueryBuilder
*/
```

### Recommended: Explicit Loading with Relations

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Standard lazy relationship (no Promise)
  @OneToMany(() => Post, post => post.author)
  posts: Post[];

  @OneToMany(() => Comment, comment => comment.author)
  comments: Comment[];
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToOne(() => User, user => user.posts)
  author: User;

  @OneToMany(() => Comment, comment => comment.post)
  comments: Comment[];
}

// Explicit loading with find()
const user = await userRepository.findOne({
  where: { id: 1 },
  relations: ['posts'], // Explicitly load posts
});
console.log(user.posts); // ✅ Posts loaded

// Load multiple levels
const userWithData = await userRepository.findOne({
  where: { id: 1 },
  relations: ['posts', 'posts.comments', 'comments'],
});
```

### Explicit Loading with Query Builder

```typescript
// Query Builder: Most flexible approach
const user = await userRepository
  .createQueryBuilder('user')
  .leftJoinAndSelect('user.posts', 'posts')
  .leftJoinAndSelect('posts.comments', 'comments')
  .where('user.id = :id', { id: 1 })
  .getOne();

// Conditional loading
async function getUserWithRelations(userId: number, includePosts: boolean) {
  const qb = userRepository
    .createQueryBuilder('user')
    .where('user.id = :id', { id: userId });

  if (includePosts) {
    qb.leftJoinAndSelect('user.posts', 'posts');
  }

  return await qb.getOne();
}
```

### Explicit Loading with RelationQueryBuilder

```typescript
// Load relation after entity is loaded
const user = await userRepository.findOne({ where: { id: 1 } });

// Load posts separately
const posts = await userRepository
  .createQueryBuilder()
  .relation(User, 'posts')
  .of(user) // or .of(userId)
  .loadMany();

user.posts = posts;

// Load single relation
const post = await postRepository.findOne({ where: { id: 1 } });

const author = await postRepository
  .createQueryBuilder()
  .relation(Post, 'author')
  .of(post)
  .loadOne();

post.author = author;
```

### Real-World Example: Blog Service

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  username: string;

  @Column()
  email: string;

  @OneToMany(() => Post, post => post.author)
  posts: Post[];

  @OneToMany(() => Comment, comment => comment.author)
  comments: Comment[];

  @ManyToMany(() => Tag, tag => tag.followers)
  followedTags: Tag[];
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column('text')
  content: string;

  @ManyToOne(() => User, user => user.posts)
  author: User;

  @OneToMany(() => Comment, comment => comment.post)
  comments: Comment[];

  @ManyToMany(() => Tag, tag => tag.posts)
  @JoinTable()
  tags: Tag[];

  @CreateDateColumn()
  createdAt: Date;
}

@Entity()
export class Comment {
  @PrimaryGeneratedColumn()
  id: number;

  @Column('text')
  content: string;

  @ManyToOne(() => User, user => user.comments)
  author: User;

  @ManyToOne(() => Post, post => post.comments)
  post: Post;

  @CreateDateColumn()
  createdAt: Date;
}

@Entity()
export class Tag {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @ManyToMany(() => Post, post => post.tags)
  posts: Post[];

  @ManyToMany(() => User, user => user.followedTags)
  @JoinTable()
  followers: User[];
}

// Service with lazy loading patterns
class BlogService {
  // Load only what's needed for list view
  async getPostsList(): Promise<Post[]> {
    return await postRepository.find({
      relations: ['author'], // Only author, no comments
      order: { createdAt: 'DESC' },
      take: 20,
    });
  }

  // Load everything for detail view
  async getPostDetail(postId: number): Promise<Post> {
    return await postRepository.findOne({
      where: { id: postId },
      relations: ['author', 'comments', 'comments.author', 'tags'],
    });
  }

  // Conditionally load relations
  async getUser(userId: number, options: {
    includePosts?: boolean;
    includeComments?: boolean;
    includeFollowedTags?: boolean;
  }): Promise<User> {
    const relations: string[] = [];

    if (options.includePosts) {
      relations.push('posts');
    }
    if (options.includeComments) {
      relations.push('comments');
    }
    if (options.includeFollowedTags) {
      relations.push('followedTags');
    }

    return await userRepository.findOne({
      where: { id: userId },
      relations,
    });
  }

  // Load relations separately for performance
  async getUserProfile(userId: number): Promise<any> {
    // Load user first
    const user = await userRepository.findOne({
      where: { id: userId },
    });

    // Load statistics without loading all entities
    const postCount = await postRepository.count({
      where: { author: { id: userId } },
    });

    const commentCount = await commentRepository.count({
      where: { author: { id: userId } },
    });

    // Load recent posts only (not all)
    const recentPosts = await postRepository.find({
      where: { author: { id: userId } },
      order: { createdAt: 'DESC' },
      take: 5,
    });

    return {
      ...user,
      stats: {
        postCount,
        commentCount,
      },
      recentPosts,
    };
  }

  // Use Query Builder for complex lazy loading
  async getPostsWithFilters(authorId?: number, tagName?: string): Promise<Post[]> {
    const qb = postRepository
      .createQueryBuilder('post')
      .leftJoinAndSelect('post.author', 'author');

    // Conditionally join tags
    if (tagName) {
      qb.leftJoinAndSelect('post.tags', 'tag')
        .where('tag.name = :tagName', { tagName });
    }

    // Conditionally filter by author
    if (authorId) {
      qb.andWhere('author.id = :authorId', { authorId });
    }

    return await qb.getMany();
  }
}
```

### Lazy Loading with Pagination

```typescript
class PostService {
  async getPaginatedPosts(page: number = 1, limit: number = 10): Promise<{
    posts: Post[];
    total: number;
    page: number;
    totalPages: number;
  }> {
    const [posts, total] = await postRepository.findAndCount({
      relations: ['author'], // Lazy load only author
      skip: (page - 1) * limit,
      take: limit,
      order: { createdAt: 'DESC' },
    });

    // Load comment counts separately (avoid loading all comments)
    const postsWithCounts = await Promise.all(
      posts.map(async post => {
        const commentCount = await commentRepository.count({
          where: { post: { id: post.id } },
        });

        return {
          ...post,
          commentCount,
        };
      })
    );

    return {
      posts: postsWithCounts,
      total,
      page,
      totalPages: Math.ceil(total / limit),
    };
  }
}
```

### Lazy Loading with Caching

```typescript
class UserService {
  private userCache = new Map<number, User>();

  async getUserWithPosts(userId: number, forceReload: boolean = false): Promise<User> {
    // Check cache
    if (!forceReload && this.userCache.has(userId)) {
      return this.userCache.get(userId)!;
    }

    // Load from database (lazy)
    const user = await userRepository.findOne({
      where: { id: userId },
      relations: ['posts'],
    });

    // Cache the result
    this.userCache.set(userId, user);

    return user;
  }

  async getUserPosts(userId: number): Promise<Post[]> {
    // Load posts separately without loading full user
    return await postRepository.find({
      where: { author: { id: userId } },
      order: { createdAt: 'DESC' },
    });
  }
}
```

### DataLoader Pattern for Lazy Loading

```typescript
import DataLoader from 'dataloader';

// Batch load users to avoid N+1
const userLoader = new DataLoader(async (userIds: number[]) => {
  const users = await userRepository.findByIds(userIds);

  // Return in same order as requested
  return userIds.map(id => users.find(user => user.id === id));
});

class PostService {
  async getPostsWithAuthors(): Promise<any[]> {
    const posts = await postRepository.find(); // No relations

    // Load authors in batch (lazy but optimized)
    const postsWithAuthors = await Promise.all(
      posts.map(async post => ({
        ...post,
        author: await userLoader.load(post.authorId),
      }))
    );

    return postsWithAuthors;
  }
}
```

### E-commerce Lazy Loading Example

```typescript
@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column('decimal', { precision: 10, scale: 2 })
  price: number;

  @ManyToOne(() => Category, category => category.products)
  category: Category;

  @OneToMany(() => Review, review => review.product)
  reviews: Review[];

  @OneToMany(() => OrderItem, item => item.product)
  orderItems: OrderItem[];

  @ManyToMany(() => Tag, tag => tag.products)
  @JoinTable()
  tags: Tag[];
}

class ProductService {
  // List: minimal data
  async getProductList(): Promise<any[]> {
    const products = await productRepository.find({
      relations: ['category'], // Only category (small)
      select: ['id', 'name', 'price'],
    });

    // Load review counts separately (lazy)
    return await Promise.all(
      products.map(async product => {
        const reviewCount = await reviewRepository.count({
          where: { product: { id: product.id } },
        });

        const avgRating = await reviewRepository
          .createQueryBuilder('review')
          .select('AVG(review.rating)', 'avg')
          .where('review.productId = :id', { id: product.id })
          .getRawOne();

        return {
          ...product,
          reviewCount,
          avgRating: avgRating?.avg || 0,
        };
      })
    );
  }

  // Detail: load more data lazily
  async getProductDetail(productId: number): Promise<any> {
    const product = await productRepository.findOne({
      where: { id: productId },
      relations: ['category', 'tags'],
    });

    // Load recent reviews separately (not all)
    const reviews = await reviewRepository.find({
      where: { product: { id: productId } },
      relations: ['user'],
      order: { createdAt: 'DESC' },
      take: 10,
    });

    return {
      ...product,
      reviews,
    };
  }

  // Admin: load sales data (heavy query)
  async getProductAnalytics(productId: number): Promise<any> {
    const product = await productRepository.findOne({
      where: { id: productId },
    });

    // Lazy load order items for analytics
    const orderItems = await orderItemRepository.find({
      where: { product: { id: productId } },
      relations: ['order'],
    });

    const totalSales = orderItems.reduce(
      (sum, item) => sum + item.quantity * item.price,
      0
    );

    return {
      ...product,
      analytics: {
        totalOrders: orderItems.length,
        totalSales,
        averageQuantity:
          orderItems.reduce((sum, item) => sum + item.quantity, 0) /
          orderItems.length,
      },
    };
  }
}
```

### Best Practices

```typescript
// ✅ GOOD: Load only what you need
const users = await userRepository.find({
  relations: ['profile'], // Only profile
});

// ❌ BAD: Loading everything
const users = await userRepository.find({
  relations: ['profile', 'posts', 'comments', 'followers', 'following'],
});

// ✅ GOOD: Conditional lazy loading
async function getUser(id: number, includeDetails: boolean) {
  const relations = [];
  if (includeDetails) {
    relations.push('posts', 'comments');
  }
  return await userRepository.findOne({ where: { id }, relations });
}

// ✅ GOOD: Load counts instead of all entities
const postCount = await postRepository.count({
  where: { author: { id: userId } },
});
// Instead of loading all posts

// ✅ GOOD: Pagination for large collections
const posts = await postRepository.find({
  where: { author: { id: userId } },
  take: 20,
  skip: 0,
});

// ✅ GOOD: Use Query Builder for complex scenarios
const posts = await postRepository
  .createQueryBuilder('post')
  .leftJoinAndSelect('post.author', 'author')
  .where('post.published = :published', { published: true })
  .andWhere('author.active = :active', { active: true })
  .getMany();

// ❌ BAD: Promise-based lazy loading (deprecated)
@OneToMany(() => Post, post => post.author)
posts: Promise<Post[]>; // ❌ Not recommended

// ✅ GOOD: Standard lazy with explicit loading
@OneToMany(() => Post, post => post.author)
posts: Post[]; // Load with relations option
```

**Interview Tip**: Explain that lazy relationships load related entities only when explicitly requested, not automatically. Mention TypeORM originally supported **Promise-based lazy loading** (`posts: Promise<Post[]>`) but it's deprecated and not recommended. Emphasize the recommended approach: **explicit loading** using `find()` with relations option, Query Builder with joins, or RelationQueryBuilder. Discuss advantages: better performance (load only what's needed), flexibility (conditional loading), control over queries. Highlight use cases: pagination (load 20 posts, not all 10,000), list vs detail views (list loads minimal data, detail loads everything), statistics (load counts instead of all entities). For production: prefer Query Builder for complex scenarios, load counts/aggregates instead of full collections, use DataLoader pattern to avoid N+1 queries, implement caching for frequently accessed data. A strong answer demonstrates understanding that lazy loading is the default and requires manual specification of what to load.

</details>

<details>
<summary>37. What is the onDelete option in relationships?</summary>

The **onDelete** option defines the database-level behavior when a **parent entity is deleted**. It specifies what happens to **child records** that have a foreign key reference to the deleted parent.

### Basic onDelete Usage

```typescript
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne, OneToMany } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Post, post => post.author)
  posts: Post[];
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToOne(() => User, user => user.posts, {
    onDelete: 'CASCADE', // Delete posts when user is deleted
  })
  author: User;
}

// When user is deleted:
await userRepository.remove(user);
// All posts by this user are automatically deleted
```

### onDelete Options

```typescript
/*
onDelete Options:
1. "CASCADE" - Delete child records when parent is deleted
2. "SET NULL" - Set foreign key to NULL when parent is deleted
3. "RESTRICT" - Prevent deletion if child records exist
4. "NO ACTION" - Similar to RESTRICT (database default)
5. "SET DEFAULT" - Set foreign key to default value (rarely used)

Syntax:
@ManyToOne(() => Parent, parent => parent.children, {
  onDelete: 'CASCADE' | 'SET NULL' | 'RESTRICT' | 'NO ACTION' | 'SET DEFAULT'
})
*/
```

### CASCADE: Delete Children

```typescript
@Entity()
export class Blog {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @OneToMany(() => Post, post => post.blog)
  posts: Post[];
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  // CASCADE: Delete posts when blog is deleted
  @ManyToOne(() => Blog, blog => blog.posts, {
    onDelete: 'CASCADE',
  })
  blog: Blog;

  @OneToMany(() => Comment, comment => comment.post)
  comments: Comment[];
}

@Entity()
export class Comment {
  @PrimaryGeneratedColumn()
  id: number;

  @Column('text')
  content: string;

  // CASCADE: Delete comments when post is deleted
  @ManyToOne(() => Post, post => post.comments, {
    onDelete: 'CASCADE',
  })
  post: Post;
}

// Cascade chain:
// Delete Blog → All Posts deleted → All Comments deleted
await blogRepository.remove(blog);
```

### SET NULL: Keep Children

```typescript
@Entity()
export class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Product, product => product.category)
  products: Product[];
}

@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // SET NULL: Keep products, remove category reference
  @ManyToOne(() => Category, category => category.products, {
    onDelete: 'SET NULL',
    nullable: true, // Must be nullable for SET NULL
  })
  category: Category | null;
}

// When category is deleted:
await categoryRepository.remove(category);
// Products remain, but category is set to NULL
```

### RESTRICT: Prevent Deletion

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  username: string;

  @OneToMany(() => Order, order => order.customer)
  orders: Order[];
}

@Entity()
export class Order {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  orderNumber: string;

  // RESTRICT: Cannot delete user if orders exist
  @ManyToOne(() => User, user => user.orders, {
    onDelete: 'RESTRICT',
  })
  customer: User;
}

// Attempt to delete user with orders:
await userRepository.remove(user);
// ❌ Error: Cannot delete user because orders exist
// Must delete orders first, then user
```

### NO ACTION: Database Default

```typescript
@Entity()
export class Author {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Book, book => book.author)
  books: Book[];
}

@Entity()
export class Book {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  // NO ACTION: Use database default behavior
  @ManyToOne(() => Author, author => author.books, {
    onDelete: 'NO ACTION', // Similar to RESTRICT
  })
  author: Author;
}

/*
NO ACTION vs RESTRICT:
- RESTRICT: Checks immediately
- NO ACTION: Checks at end of transaction (can be deferred)
- In practice, often behaves the same
*/
```

### Real-World Example: Social Media

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  username: string;

  @OneToMany(() => Post, post => post.author)
  posts: Post[];

  @OneToMany(() => Comment, comment => comment.author)
  comments: Comment[];

  @OneToMany(() => Like, like => like.user)
  likes: Like[];
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column('text')
  content: string;

  // SET NULL: Keep posts if user deleted (show as [deleted])
  @ManyToOne(() => User, user => user.posts, {
    onDelete: 'SET NULL',
    nullable: true,
  })
  author: User | null;

  @OneToMany(() => Comment, comment => comment.post)
  comments: Comment[];

  @OneToMany(() => Like, like => like.post)
  likes: Like[];

  @CreateDateColumn()
  createdAt: Date;
}

@Entity()
export class Comment {
  @PrimaryGeneratedColumn()
  id: number;

  @Column('text')
  content: string;

  // SET NULL: Keep comments if user deleted
  @ManyToOne(() => User, user => user.comments, {
    onDelete: 'SET NULL',
    nullable: true,
  })
  author: User | null;

  // CASCADE: Delete comments if post deleted
  @ManyToOne(() => Post, post => post.comments, {
    onDelete: 'CASCADE',
  })
  post: Post;

  @CreateDateColumn()
  createdAt: Date;
}

@Entity()
export class Like {
  @PrimaryGeneratedColumn()
  id: number;

  // CASCADE: Delete likes if user deleted
  @ManyToOne(() => User, user => user.likes, {
    onDelete: 'CASCADE',
  })
  user: User;

  // CASCADE: Delete likes if post deleted
  @ManyToOne(() => Post, post => post.likes, {
    onDelete: 'CASCADE',
  })
  post: Post;
}

// Behavior:
// Delete User:
//   - Posts remain (SET NULL) - shown as "[deleted user]"
//   - Comments remain (SET NULL)
//   - Likes deleted (CASCADE)

// Delete Post:
//   - Comments deleted (CASCADE)
//   - Likes deleted (CASCADE)
```

### E-commerce Order System

```typescript
@Entity()
export class Customer {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Order, order => order.customer)
  orders: Order[];
}

@Entity()
export class Order {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  orderNumber: string;

  // RESTRICT: Cannot delete customer with orders (compliance)
  @ManyToOne(() => Customer, customer => customer.orders, {
    onDelete: 'RESTRICT',
  })
  customer: Customer;

  @OneToMany(() => OrderItem, item => item.order)
  items: OrderItem[];

  @Column()
  status: string;
}

@Entity()
export class OrderItem {
  @PrimaryGeneratedColumn()
  id: number;

  @Column('int')
  quantity: number;

  // CASCADE: Delete items when order is cancelled/deleted
  @ManyToOne(() => Order, order => order.items, {
    onDelete: 'CASCADE',
  })
  order: Order;

  // RESTRICT: Cannot delete product if in orders
  @ManyToOne(() => Product, product => product.orderItems, {
    onDelete: 'RESTRICT',
  })
  product: Product;
}

@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => OrderItem, item => item.product)
  orderItems: OrderItem[];
}

// Behavior:
// - Cannot delete Customer if they have orders (RESTRICT)
// - Delete Order → OrderItems deleted automatically (CASCADE)
// - Cannot delete Product if it's in any order (RESTRICT)
```

### onDelete with Soft Delete

```typescript
@Entity()
export class Team {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Project, project => project.team)
  projects: Project[];

  @DeleteDateColumn()
  deletedAt: Date | null;
}

@Entity()
export class Project {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // SET NULL: Keep projects if team soft-deleted
  @ManyToOne(() => Team, team => team.projects, {
    onDelete: 'SET NULL',
    nullable: true,
  })
  team: Team | null;

  @DeleteDateColumn()
  deletedAt: Date | null;
}

// Soft delete team
await teamRepository.softRemove(team);
// Projects remain, team reference stays (onDelete doesn't apply to soft delete)

// Hard delete team
await teamRepository.remove(team);
// Projects remain, team reference set to NULL (onDelete applies)
```

### Composite Foreign Keys

```typescript
@Entity()
export class Tenant {
  @PrimaryColumn()
  tenantId: number;

  @Column()
  name: string;

  @OneToMany(() => User, user => user.tenant)
  users: User[];
}

@Entity()
export class User {
  @PrimaryColumn()
  tenantId: number;

  @PrimaryColumn()
  userId: number;

  @Column()
  username: string;

  // CASCADE with composite key
  @ManyToOne(() => Tenant, tenant => tenant.users, {
    onDelete: 'CASCADE',
  })
  @JoinColumn({ name: 'tenantId', referencedColumnName: 'tenantId' })
  tenant: Tenant;

  @OneToMany(() => Post, post => post.author)
  posts: Post[];
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column()
  tenantId: number;

  @Column()
  authorId: number;

  // CASCADE: Delete posts when user deleted
  @ManyToOne(() => User, user => user.posts, {
    onDelete: 'CASCADE',
  })
  @JoinColumn([
    { name: 'tenantId', referencedColumnName: 'tenantId' },
    { name: 'authorId', referencedColumnName: 'userId' },
  ])
  author: User;
}

// Delete Tenant → Users deleted → Posts deleted (cascade chain)
```

### Migration with onDelete

```typescript
import { MigrationInterface, QueryRunner, Table, TableForeignKey } from 'typeorm';

export class CreatePostsTable1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.createTable(
      new Table({
        name: 'posts',
        columns: [
          {
            name: 'id',
            type: 'int',
            isPrimary: true,
            isGenerated: true,
            generationStrategy: 'increment',
          },
          {
            name: 'title',
            type: 'varchar',
          },
          {
            name: 'authorId',
            type: 'int',
            isNullable: true, // Nullable for SET NULL
          },
        ],
      })
    );

    // Add foreign key with onDelete
    await queryRunner.createForeignKey(
      'posts',
      new TableForeignKey({
        columnNames: ['authorId'],
        referencedColumnNames: ['id'],
        referencedTableName: 'users',
        onDelete: 'SET NULL', // Database-level constraint
      })
    );
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    const table = await queryRunner.getTable('posts');
    const foreignKey = table.foreignKeys.find(fk => fk.columnNames.indexOf('authorId') !== -1);
    await queryRunner.dropForeignKey('posts', foreignKey);
    await queryRunner.dropTable('posts');
  }
}
```

### Best Practices

```typescript
// ✅ GOOD: CASCADE for dependent data (owned entities)
@ManyToOne(() => Order, order => order.items, {
  onDelete: 'CASCADE', // OrderItems belong to Order
})
order: Order;

// ✅ GOOD: SET NULL for independent data
@ManyToOne(() => User, user => user.posts, {
  onDelete: 'SET NULL', // Keep posts if user deleted
  nullable: true, // Must be nullable
})
author: User | null;

// ✅ GOOD: RESTRICT for audit/compliance
@ManyToOne(() => Customer, customer => customer.orders, {
  onDelete: 'RESTRICT', // Cannot delete customer with orders
})
customer: Customer;

// ❌ BAD: CASCADE on independent entities
@ManyToOne(() => Category, category => category.products, {
  onDelete: 'CASCADE', // ❌ Deleting category shouldn't delete products
})
category: Category;

// ❌ BAD: SET NULL without nullable
@ManyToOne(() => User, user => user.posts, {
  onDelete: 'SET NULL',
  nullable: false, // ❌ Conflict: can't set NULL if not nullable
})
author: User;

// ✅ GOOD: CASCADE for composition (parent-child)
@ManyToOne(() => Post, post => post.comments, {
  onDelete: 'CASCADE', // Comments belong to post
})
post: Post;

// ✅ GOOD: RESTRICT for shared references
@ManyToOne(() => Product, product => product.orderItems, {
  onDelete: 'RESTRICT', // Product used by multiple orders
})
product: Product;

// ✅ GOOD: Combine with cascade option for application-level
@ManyToOne(() => Blog, blog => blog.posts, {
  onDelete: 'CASCADE', // Database-level
})
blog: Blog;

@OneToMany(() => Post, post => post.blog, {
  cascade: true, // Application-level
})
posts: Post[];
```

### Difference: onDelete vs Cascade

```typescript
/*
onDelete (Database-level):
- Enforced by the DATABASE
- Works even outside TypeORM
- Affects DELETE operations
- Options: CASCADE, SET NULL, RESTRICT, NO ACTION

cascade (Application-level):
- Enforced by TypeORM
- Only works through TypeORM operations
- Affects save/update/remove operations
- Options: insert, update, remove, soft-remove, recover
*/

// Database-level onDelete
@ManyToOne(() => User, user => user.posts, {
  onDelete: 'CASCADE', // Database deletes posts when user deleted
})
author: User;

// Application-level cascade
@OneToMany(() => Post, post => post.author, {
  cascade: ['remove'], // TypeORM removes posts when user removed
})
posts: Post[];

// Best practice: Use both for consistency
@ManyToOne(() => Order, order => order.items, {
  onDelete: 'CASCADE', // Database-level protection
})
order: Order;

@OneToMany(() => OrderItem, item => item.order, {
  cascade: true, // Application-level convenience
})
items: OrderItem[];
```

**Interview Tip**: Explain that `onDelete` is a **database-level** constraint that defines what happens to child records when parent is deleted, while `cascade` is application-level. Emphasize the four main options: **CASCADE** (delete children), **SET NULL** (keep children, null FK - requires nullable), **RESTRICT** (prevent deletion if children exist), **NO ACTION** (similar to RESTRICT). Discuss use cases: CASCADE for **composition** (OrderItems belong to Order), SET NULL for **association** (keep posts when user deleted), RESTRICT for **audit/compliance** (can't delete customer with orders). Highlight that onDelete is enforced by the database (works even outside TypeORM), while cascade is TypeORM-level. For production: use CASCADE for dependent/owned entities, SET NULL for independent entities you want to keep, RESTRICT for data integrity requirements, always combine with nullable: true when using SET NULL. A strong answer demonstrates understanding of composition vs association and database vs application-level enforcement.

</details>

**Interview Tip**: Explain that `onDelete` is a **database-level** constraint that defines what happens to child records when parent is deleted, while `cascade` is application-level. Emphasize the four main options: **CASCADE** (delete children), **SET NULL** (keep children, null FK - requires nullable), **RESTRICT** (prevent deletion if children exist), **NO ACTION** (similar to RESTRICT). Discuss use cases: CASCADE for **composition** (OrderItems belong to Order), SET NULL for **association** (keep posts when user deleted), RESTRICT for **audit/compliance** (can't delete customer with orders). Highlight that onDelete is enforced by the database (works even outside TypeORM), while cascade is TypeORM-level. For production: use CASCADE for dependent/owned entities, SET NULL for independent entities you want to keep, RESTRICT for data integrity requirements, always combine with nullable: true when using SET NULL. A strong answer demonstrates understanding of composition vs association and database vs application-level enforcement.

</details>

<details>
<summary>38. How do you define self-referencing relationships?</summary>

**Self-referencing relationships** occur when an entity has a relationship to itself, creating hierarchical or recursive structures. Common examples include organizational charts (employee-manager), threaded comments (comment replies), category trees, and folder structures.

### Basic Self-Referencing One-to-Many

```typescript
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne, OneToMany } from 'typeorm';

@Entity()
export class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Parent category (Many-to-One to self)
  @ManyToOne(() => Category, category => category.children, {
    nullable: true, // Root categories have no parent
  })
  parent: Category | null;

  // Child categories (One-to-Many to self)
  @OneToMany(() => Category, category => category.parent)
  children: Category[];
}

// Usage
const electronics = new Category();
electronics.name = 'Electronics';

const computers = new Category();
computers.name = 'Computers';
computers.parent = electronics;

const laptops = new Category();
laptops.name = 'Laptops';
laptops.parent = computers;

await categoryRepository.save([electronics, computers, laptops]);

// Query hierarchy
const category = await categoryRepository.findOne({
  where: { id: electronics.id },
  relations: ['children', 'children.children'], // Load multiple levels
});
```

### Employee-Manager Hierarchy

```typescript
@Entity()
export class Employee {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  position: string;

  @Column('decimal', { precision: 10, scale: 2 })
  salary: number;

  // Manager (self-reference)
  @ManyToOne(() => Employee, employee => employee.subordinates, {
    nullable: true, // CEO has no manager
    onDelete: 'SET NULL', // Keep employees if manager leaves
  })
  manager: Employee | null;

  // Subordinates (self-reference)
  @OneToMany(() => Employee, employee => employee.manager)
  subordinates: Employee[];
}

// Create hierarchy
const ceo = new Employee();
ceo.name = 'John CEO';
ceo.position = 'CEO';
ceo.manager = null; // No manager

const cto = new Employee();
cto.name = 'Jane CTO';
cto.position = 'CTO';
cto.manager = ceo;

const seniorDev = new Employee();
seniorDev.name = 'Bob Senior Dev';
seniorDev.position = 'Senior Developer';
seniorDev.manager = cto;

const juniorDev = new Employee();
juniorDev.name = 'Alice Junior Dev';
juniorDev.position = 'Junior Developer';
juniorDev.manager = seniorDev;

await employeeRepository.save([ceo, cto, seniorDev, juniorDev]);

// Query with manager chain
const employee = await employeeRepository.findOne({
  where: { id: juniorDev.id },
  relations: ['manager', 'manager.manager', 'manager.manager.manager'],
});

console.log(employee.name); // Alice Junior Dev
console.log(employee.manager.name); // Bob Senior Dev
console.log(employee.manager.manager.name); // Jane CTO
console.log(employee.manager.manager.manager.name); // John CEO
```

### Threaded Comments

```typescript
@Entity()
export class Comment {
  @PrimaryGeneratedColumn()
  id: number;

  @Column('text')
  content: string;

  @ManyToOne(() => User, user => user.comments)
  author: User;

  @ManyToOne(() => Post, post => post.comments)
  post: Post;

  // Self-referencing: Parent comment
  @ManyToOne(() => Comment, comment => comment.replies, {
    nullable: true, // Top-level comments have no parent
    onDelete: 'CASCADE', // Delete replies if parent deleted
  })
  parentComment: Comment | null;

  // Self-referencing: Child replies
  @OneToMany(() => Comment, comment => comment.parentComment)
  replies: Comment[];

  @CreateDateColumn()
  createdAt: Date;
}

// Service
class CommentService {
  async createComment(
    postId: number,
    authorId: number,
    content: string,
    parentCommentId?: number
  ): Promise<Comment> {
    const comment = new Comment();
    comment.content = content;
    comment.post = await postRepository.findOneBy({ id: postId });
    comment.author = await userRepository.findOneBy({ id: authorId });

    if (parentCommentId) {
      comment.parentComment = await commentRepository.findOneBy({
        id: parentCommentId,
      });
    }

    return await commentRepository.save(comment);
  }

  async getCommentThread(commentId: number): Promise<Comment> {
    return await commentRepository.findOne({
      where: { id: commentId },
      relations: [
        'author',
        'replies',
        'replies.author',
        'replies.replies',
        'replies.replies.author',
      ],
    });
  }

  async getRootComments(postId: number): Promise<Comment[]> {
    return await commentRepository.find({
      where: {
        post: { id: postId },
        parentComment: IsNull(), // Only top-level comments
      },
      relations: ['author', 'replies', 'replies.author'],
      order: { createdAt: 'DESC' },
    });
  }
}
```

### Tree Structure with TreeRepository

```typescript
import { Entity, PrimaryGeneratedColumn, Column, Tree, TreeChildren, TreeParent } from 'typeorm';

// Using @Tree decorator for hierarchical data
@Entity()
@Tree('closure-table') // closure-table, materialized-path, or nested-set
export class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column({ nullable: true })
  description: string;

  @TreeParent()
  parent: Category | null;

  @TreeChildren()
  children: Category[];
}

// Service with Tree operations
class CategoryService {
  private categoryTreeRepository = dataSource.getTreeRepository(Category);

  async createCategory(
    name: string,
    description: string,
    parentId?: number
  ): Promise<Category> {
    const category = new Category();
    category.name = name;
    category.description = description;

    if (parentId) {
      category.parent = await this.categoryTreeRepository.findOneBy({
        id: parentId,
      });
    }

    return await this.categoryTreeRepository.save(category);
  }

  // Get entire tree
  async getCategoryTree(): Promise<Category[]> {
    return await this.categoryTreeRepository.findTrees();
  }

  // Get all descendants (children, grandchildren, etc.)
  async getDescendants(categoryId: number): Promise<Category[]> {
    const category = await this.categoryTreeRepository.findOneBy({
      id: categoryId,
    });
    return await this.categoryTreeRepository.findDescendants(category);
  }

  // Get all ancestors (parent, grandparent, etc.)
  async getAncestors(categoryId: number): Promise<Category[]> {
    const category = await this.categoryTreeRepository.findOneBy({
      id: categoryId,
    });
    return await this.categoryTreeRepository.findAncestors(category);
  }

  // Get descendants tree (with hierarchy)
  async getDescendantsTree(categoryId: number): Promise<Category> {
    const category = await this.categoryTreeRepository.findOneBy({
      id: categoryId,
    });
    return await this.categoryTreeRepository.findDescendantsTree(category);
  }

  // Count descendants
  async countDescendants(categoryId: number): Promise<number> {
    const category = await this.categoryTreeRepository.findOneBy({
      id: categoryId,
    });
    return await this.categoryTreeRepository.countDescendants(category);
  }
}
```

### Folder Structure Example

```typescript
@Entity()
@Tree('materialized-path') // Efficient for read-heavy operations
export class Folder {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  path: string; // /documents/work/project1

  @TreeParent()
  parent: Folder | null;

  @TreeChildren()
  children: Folder[];

  @ManyToOne(() => User, user => user.folders)
  owner: User;

  @OneToMany(() => File, file => file.folder)
  files: File[];

  @CreateDateColumn()
  createdAt: Date;
}

@Entity()
export class File {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  size: number;

  @ManyToOne(() => Folder, folder => folder.files, {
    onDelete: 'CASCADE',
  })
  folder: Folder;

  @ManyToOne(() => User, user => user.files)
  owner: User;
}

// Service
class FolderService {
  private folderTreeRepository = dataSource.getTreeRepository(Folder);

  async createFolder(
    name: string,
    userId: number,
    parentFolderId?: number
  ): Promise<Folder> {
    const folder = new Folder();
    folder.name = name;
    folder.owner = await userRepository.findOneBy({ id: userId });

    if (parentFolderId) {
      const parent = await this.folderTreeRepository.findOneBy({
        id: parentFolderId,
      });
      folder.parent = parent;
      folder.path = `${parent.path}/${name}`;
    } else {
      folder.path = `/${name}`;
    }

    return await this.folderTreeRepository.save(folder);
  }

  async getFolderContents(folderId: number): Promise<any> {
    const folder = await this.folderTreeRepository.findOne({
      where: { id: folderId },
      relations: ['children', 'files'],
    });

    return {
      folder: folder.name,
      path: folder.path,
      subfolders: folder.children,
      files: folder.files,
    };
  }

  async moveFolder(folderId: number, newParentId: number): Promise<Folder> {
    const folder = await this.folderTreeRepository.findOneBy({ id: folderId });
    const newParent = await this.folderTreeRepository.findOneBy({
      id: newParentId,
    });

    folder.parent = newParent;
    folder.path = `${newParent.path}/${folder.name}`;

    return await this.folderTreeRepository.save(folder);
  }

  async getBreadcrumbs(folderId: number): Promise<Folder[]> {
    const folder = await this.folderTreeRepository.findOneBy({ id: folderId });
    return await this.folderTreeRepository.findAncestors(folder);
  }
}
```

### Recursive Query with Query Builder

```typescript
class EmployeeService {
  // Get all employees under a manager (recursive)
  async getTeamMembers(managerId: number): Promise<Employee[]> {
    // PostgreSQL recursive CTE
    const query = `
      WITH RECURSIVE team AS (
        SELECT * FROM employee WHERE id = $1
        UNION ALL
        SELECT e.* FROM employee e
        INNER JOIN team t ON e.managerId = t.id
      )
      SELECT * FROM team WHERE id != $1
    `;

    return await employeeRepository.query(query, [managerId]);
  }

  // Get manager chain (recursive up)
  async getManagerChain(employeeId: number): Promise<Employee[]> {
    const query = `
      WITH RECURSIVE managers AS (
        SELECT * FROM employee WHERE id = $1
        UNION ALL
        SELECT e.* FROM employee e
        INNER JOIN managers m ON e.id = m.managerId
      )
      SELECT * FROM managers WHERE id != $1
    `;

    return await employeeRepository.query(query, [employeeId]);
  }

  // Calculate organization depth
  async getOrganizationDepth(): Promise<number> {
    const query = `
      WITH RECURSIVE org_tree AS (
        SELECT id, managerId, 1 as depth FROM employee WHERE managerId IS NULL
        UNION ALL
        SELECT e.id, e.managerId, ot.depth + 1
        FROM employee e
        INNER JOIN org_tree ot ON e.managerId = ot.id
      )
      SELECT MAX(depth) as max_depth FROM org_tree
    `;

    const result = await employeeRepository.query(query);
    return result[0].max_depth;
  }
}
```

### Self-Referencing Many-to-Many

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  username: string;

  // Following (users this user follows)
  @ManyToMany(() => User, user => user.followers)
  @JoinTable({
    name: 'user_follows',
    joinColumn: { name: 'followerId' },
    inverseJoinColumn: { name: 'followingId' },
  })
  following: User[];

  // Followers (users who follow this user)
  @ManyToMany(() => User, user => user.following)
  followers: User[];
}

// Service
class UserService {
  async followUser(followerId: number, followingId: number): Promise<void> {
    const follower = await userRepository.findOne({
      where: { id: followerId },
      relations: ['following'],
    });

    const following = await userRepository.findOneBy({ id: followingId });

    if (!follower.following.find(u => u.id === following.id)) {
      follower.following.push(following);
      await userRepository.save(follower);
    }
  }

  async unfollowUser(followerId: number, followingId: number): Promise<void> {
    const follower = await userRepository.findOne({
      where: { id: followerId },
      relations: ['following'],
    });

    follower.following = follower.following.filter(u => u.id !== followingId);
    await userRepository.save(follower);
  }

  async getFollowers(userId: number): Promise<User[]> {
    const user = await userRepository.findOne({
      where: { id: userId },
      relations: ['followers'],
    });
    return user.followers;
  }

  async getFollowing(userId: number): Promise<User[]> {
    const user = await userRepository.findOne({
      where: { id: userId },
      relations: ['following'],
    });
    return user.following;
  }

  async getMutualFollows(userId: number): Promise<User[]> {
    const user = await userRepository.findOne({
      where: { id: userId },
      relations: ['followers', 'following'],
    });

    // Users who follow each other
    return user.followers.filter(follower =>
      user.following.some(following => following.id === follower.id)
    );
  }
}
```

### Best Practices

```typescript
// ✅ GOOD: Make parent nullable for root nodes
@ManyToOne(() => Category, category => category.children, {
  nullable: true, // Root categories
})
parent: Category | null;

// ✅ GOOD: Use onDelete: 'CASCADE' for comments/threads
@ManyToOne(() => Comment, comment => comment.replies, {
  nullable: true,
  onDelete: 'CASCADE', // Delete replies when parent deleted
})
parentComment: Comment | null;

// ✅ GOOD: Use onDelete: 'SET NULL' for organizational structures
@ManyToOne(() => Employee, employee => employee.subordinates, {
  nullable: true,
  onDelete: 'SET NULL', // Keep employees if manager leaves
})
manager: Employee | null;

// ✅ GOOD: Use Tree decorators for complex hierarchies
@Entity()
@Tree('closure-table')
export class Category {
  @TreeParent()
  parent: Category | null;

  @TreeChildren()
  children: Category[];
}

// ✅ GOOD: Limit depth when loading relations
relations: ['children', 'children.children'] // Max 2 levels
// Avoid: loading entire tree with find()

// ✅ GOOD: Use recursive CTEs for deep queries
const query = `
  WITH RECURSIVE tree AS (
    SELECT * FROM category WHERE id = $1
    UNION ALL
    SELECT c.* FROM category c
    INNER JOIN tree t ON c.parentId = t.id
  )
  SELECT * FROM tree
`;

// ❌ BAD: Loading entire tree with eager loading
@OneToMany(() => Category, category => category.parent, {
  eager: true, // ❌ Loads all descendants always
})
children: Category[];

// ❌ BAD: Circular references without nullable
@ManyToOne(() => Category, category => category.children)
parent: Category; // ❌ Should be nullable

// ✅ GOOD: Index parent foreign key
@ManyToOne(() => Category, category => category.children)
@Index() // Index for performance
parent: Category | null;
```

### Tree Pattern Comparison

```typescript
/*
CLOSURE TABLE (@Tree('closure-table')):
✅ Fast reads (ancestors/descendants)
✅ Simple to understand
❌ Extra table for relationships
❌ More storage

MATERIALIZED PATH (@Tree('materialized-path')):
✅ Fast path queries
✅ Simple queries (LIKE path%)
❌ Path updates on move
❌ Limited path length

NESTED SET (@Tree('nested-set')):
✅ Very fast reads
✅ Efficient range queries
❌ Complex updates
❌ Difficult to maintain

ADJACENCY LIST (Standard self-reference):
✅ Simple structure
✅ Easy to update
❌ Slow recursive queries
❌ Manual recursion needed

RECOMMENDATION:
- Use closure-table for general hierarchies
- Use materialized-path for file systems
- Use adjacency list for simple 2-3 level trees
*/
```

**Interview Tip**: Explain that self-referencing relationships occur when an entity relates to itself, creating hierarchical structures. Show basic pattern: `@ManyToOne(() => Entity, entity => entity.children)` for parent and `@OneToMany(() => Entity, entity => entity.parent)` for children. Emphasize parent should be **nullable** for root nodes. Discuss common use cases: employee-manager hierarchy (nullable parent with onDelete: 'SET NULL'), threaded comments (nullable parent with onDelete: 'CASCADE'), category trees (use @Tree decorators). Highlight TypeORM's **Tree decorators** (`@Tree('closure-table')`, `@TreeParent`, `@TreeChildren`) with TreeRepository methods (findTrees, findDescendants, findAncestors). Mention self-referencing Many-to-Many for social graphs (followers/following). For production: use Tree decorators for complex hierarchies (better performance), limit relation depth when loading (avoid loading entire tree), use recursive CTEs for deep queries, index parent foreign key. A strong answer demonstrates understanding of different tree patterns and when to use Tree decorators vs manual self-references.

</details>

<details>
<summary>39. What is orphanedRowAction and when to use it?</summary>

**orphanedRowAction** defines what happens to child entities when they are **removed from their parent's collection** but not explicitly deleted. It handles the scenario where a relationship is broken but the child entity still exists in the database.

### Basic orphanedRowAction Usage

```typescript
import { Entity, PrimaryGeneratedColumn, Column, OneToMany, ManyToOne } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Photo, photo => photo.user, {
    cascade: true,
    orphanedRowAction: 'delete', // Delete photos removed from collection
  })
  photos: Photo[];
}

@Entity()
export class Photo {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  url: string;

  @ManyToOne(() => User, user => user.photos)
  user: User;
}

// Usage
const user = await userRepository.findOne({
  where: { id: 1 },
  relations: ['photos'],
});

// Remove photo from collection (orphan it)
user.photos = user.photos.filter(photo => photo.id !== 5);

await userRepository.save(user);
// Photo with id=5 is automatically DELETED (orphanedRowAction: 'delete')
```

### orphanedRowAction Options

```typescript
/*
orphanedRowAction Options:
1. "delete" - Delete orphaned entities from database
2. "nullify" - Set foreign key to NULL (default)
3. "disable" - Do nothing (keep orphaned entity as-is)

Syntax:
@OneToMany(() => Child, child => child.parent, {
  cascade: true, // Required for orphanedRowAction to work
  orphanedRowAction: 'delete' | 'nullify' | 'disable'
})
children: Child[];

IMPORTANT: orphanedRowAction only works with cascade enabled!
*/
```

### orphanedRowAction: 'delete'

```typescript
@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @OneToMany(() => Comment, comment => comment.post, {
    cascade: true,
    orphanedRowAction: 'delete', // Delete orphaned comments
  })
  comments: Comment[];
}

@Entity()
export class Comment {
  @PrimaryGeneratedColumn()
  id: number;

  @Column('text')
  content: string;

  @ManyToOne(() => Post, post => post.comments)
  post: Post;
}

// Service
class PostService {
  async removeComment(postId: number, commentId: number): Promise<void> {
    const post = await postRepository.findOne({
      where: { id: postId },
      relations: ['comments'],
    });

    // Remove comment from collection
    post.comments = post.comments.filter(c => c.id !== commentId);

    await postRepository.save(post);
    // Comment is DELETED from database (orphanedRowAction: 'delete')
  }
}
```

### orphanedRowAction: 'nullify' (Default)

```typescript
@Entity()
export class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Product, product => product.category, {
    cascade: true,
    orphanedRowAction: 'nullify', // Set FK to NULL (default)
  })
  products: Product[];
}

@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @ManyToOne(() => Category, category => category.products, {
    nullable: true, // Must be nullable for 'nullify'
  })
  category: Category | null;
}

// Service
class CategoryService {
  async removeProductFromCategory(
    categoryId: number,
    productId: number
  ): Promise<void> {
    const category = await categoryRepository.findOne({
      where: { id: categoryId },
      relations: ['products'],
    });

    // Remove product from category
    category.products = category.products.filter(p => p.id !== productId);

    await categoryRepository.save(category);
    // Product remains in database but category FK is NULL
  }
}
```

### orphanedRowAction: 'disable'

```typescript
@Entity()
export class Team {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Player, player => player.team, {
    cascade: true,
    orphanedRowAction: 'disable', // Keep orphaned players unchanged
  })
  players: Player[];
}

@Entity()
export class Player {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @ManyToOne(() => Team, team => team.players)
  team: Team;
}

// Service
class TeamService {
  async removePlayerFromTeam(teamId: number, playerId: number): Promise<void> {
    const team = await teamRepository.findOne({
      where: { id: teamId },
      relations: ['players'],
    });

    // Remove player from collection
    team.players = team.players.filter(p => p.id !== playerId);

    await teamRepository.save(team);
    // Player still references team (orphanedRowAction: 'disable')
    // Must manually update player.team = null if needed
  }
}
```

### Real-World Example: Shopping Cart

```typescript
@Entity()
export class ShoppingCart {
  @PrimaryGeneratedColumn()
  id: number;

  @ManyToOne(() => User, user => user.carts)
  user: User;

  @OneToMany(() => CartItem, item => item.cart, {
    cascade: true,
    orphanedRowAction: 'delete', // Delete items removed from cart
  })
  items: CartItem[];

  @CreateDateColumn()
  createdAt: Date;
}

@Entity()
export class CartItem {
  @PrimaryGeneratedColumn()
  id: number;

  @ManyToOne(() => ShoppingCart, cart => cart.items)
  cart: ShoppingCart;

  @ManyToOne(() => Product, product => product.cartItems)
  product: Product;

  @Column('int')
  quantity: number;

  @Column('decimal', { precision: 10, scale: 2 })
  price: number;
}

@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column('decimal', { precision: 10, scale: 2 })
  price: number;

  @OneToMany(() => CartItem, item => item.product)
  cartItems: CartItem[];
}

// Service
class CartService {
  async removeItemFromCart(cartId: number, itemId: number): Promise<void> {
    const cart = await cartRepository.findOne({
      where: { id: cartId },
      relations: ['items'],
    });

    // Remove item from cart
    cart.items = cart.items.filter(item => item.id !== itemId);

    await cartRepository.save(cart);
    // CartItem is DELETED (orphanedRowAction: 'delete')
  }

  async updateCartItems(cartId: number, newItems: CartItem[]): Promise<void> {
    const cart = await cartRepository.findOne({
      where: { id: cartId },
      relations: ['items'],
    });

    // Replace all items
    cart.items = newItems;

    await cartRepository.save(cart);
    // Old items not in newItems are DELETED (orphanedRowAction: 'delete')
  }

  async clearCart(cartId: number): Promise<void> {
    const cart = await cartRepository.findOne({
      where: { id: cartId },
      relations: ['items'],
    });

    // Remove all items
    cart.items = [];

    await cartRepository.save(cart);
    // All CartItems DELETED (orphanedRowAction: 'delete')
  }
}
```

### Photo Gallery Example

```typescript
@Entity()
export class Album {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToOne(() => User, user => user.albums)
  owner: User;

  @OneToMany(() => Photo, photo => photo.album, {
    cascade: true,
    orphanedRowAction: 'delete', // Delete photos removed from album
  })
  photos: Photo[];
}

@Entity()
export class Photo {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  url: string;

  @Column()
  fileName: string;

  @ManyToOne(() => Album, album => album.photos)
  album: Album;

  @ManyToOne(() => User, user => user.photos)
  uploader: User;
}

// Service
class AlbumService {
  async removePhotos(albumId: number, photoIds: number[]): Promise<void> {
    const album = await albumRepository.findOne({
      where: { id: albumId },
      relations: ['photos'],
    });

    // Remove photos from album
    album.photos = album.photos.filter(photo => !photoIds.includes(photo.id));

    await albumRepository.save(album);
    // Removed photos are DELETED (orphanedRowAction: 'delete')
  }

  async reorderPhotos(albumId: number, orderedPhotoIds: number[]): Promise<void> {
    const album = await albumRepository.findOne({
      where: { id: albumId },
      relations: ['photos'],
    });

    // Keep only photos in new order
    const orderedPhotos = orderedPhotoIds
      .map(id => album.photos.find(p => p.id === id))
      .filter(p => p !== undefined);

    album.photos = orderedPhotos;

    await albumRepository.save(album);
    // Photos not in orderedPhotoIds are DELETED
  }
}
```

### Playlist Management Example

```typescript
@Entity()
export class Playlist {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @ManyToOne(() => User, user => user.playlists)
  owner: User;

  @OneToMany(() => PlaylistItem, item => item.playlist, {
    cascade: true,
    orphanedRowAction: 'delete', // Delete removed playlist items
  })
  items: PlaylistItem[];
}

@Entity()
export class PlaylistItem {
  @PrimaryGeneratedColumn()
  id: number;

  @ManyToOne(() => Playlist, playlist => playlist.items)
  playlist: Playlist;

  @ManyToOne(() => Song, song => song.playlistItems)
  song: Song;

  @Column('int')
  position: number;

  @CreateDateColumn()
  addedAt: Date;
}

@Entity()
export class Song {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column()
  artist: string;

  @OneToMany(() => PlaylistItem, item => item.song)
  playlistItems: PlaylistItem[];
}

// Service
class PlaylistService {
  async removeSongFromPlaylist(
    playlistId: number,
    songId: number
  ): Promise<void> {
    const playlist = await playlistRepository.findOne({
      where: { id: playlistId },
      relations: ['items', 'items.song'],
    });

    // Remove song from playlist
    playlist.items = playlist.items.filter(item => item.song.id !== songId);

    await playlistRepository.save(playlist);
    // PlaylistItem is DELETED (orphanedRowAction: 'delete')
    // Song remains in database
  }

  async reorderPlaylist(
    playlistId: number,
    newOrder: number[]
  ): Promise<void> {
    const playlist = await playlistRepository.findOne({
      where: { id: playlistId },
      relations: ['items'],
    });

    // Update positions
    playlist.items = newOrder.map((itemId, index) => {
      const item = playlist.items.find(i => i.id === itemId);
      item.position = index;
      return item;
    });

    await playlistRepository.save(playlist);
  }
}
```

### Important: Cascade Required

```typescript
// ❌ WRONG: orphanedRowAction without cascade
@OneToMany(() => Comment, comment => comment.post, {
  orphanedRowAction: 'delete', // ❌ Won't work without cascade!
})
comments: Comment[];

// ✅ CORRECT: orphanedRowAction with cascade
@OneToMany(() => Comment, comment => comment.post, {
  cascade: true, // Required!
  orphanedRowAction: 'delete',
})
comments: Comment[];

/*
orphanedRowAction ONLY works when:
1. cascade is enabled (cascade: true or specific operations)
2. You save the parent entity (not the child)
3. Child is removed from parent's collection
*/
```

### Comparison with Manual Deletion

```typescript
// WITHOUT orphanedRowAction
@OneToMany(() => Photo, photo => photo.user, {
  cascade: true,
})
photos: Photo[];

// Must manually delete
async removePhoto(userId: number, photoId: number) {
  const user = await userRepository.findOne({
    where: { id: userId },
    relations: ['photos'],
  });

  const photo = user.photos.find(p => p.id === photoId);
  user.photos = user.photos.filter(p => p.id !== photoId);

  await userRepository.save(user); // Only removes relationship
  await photoRepository.remove(photo); // Must manually delete
}

// WITH orphanedRowAction
@OneToMany(() => Photo, photo => photo.user, {
  cascade: true,
  orphanedRowAction: 'delete', // Automatic deletion
})
photos: Photo[];

// Automatic deletion
async removePhoto(userId: number, photoId: number) {
  const user = await userRepository.findOne({
    where: { id: userId },
    relations: ['photos'],
  });

  user.photos = user.photos.filter(p => p.id !== photoId);

  await userRepository.save(user); // Automatically deletes photo
}
```

### Best Practices

```typescript
// ✅ GOOD: Use 'delete' for owned/dependent entities
@OneToMany(() => CartItem, item => item.cart, {
  cascade: true,
  orphanedRowAction: 'delete', // Cart items belong to cart
})
items: CartItem[];

// ✅ GOOD: Use 'nullify' for independent entities
@OneToMany(() => Product, product => product.category, {
  cascade: true,
  orphanedRowAction: 'nullify', // Products exist independently
})
products: Product[];

// ✅ GOOD: Ensure nullable when using 'nullify'
@ManyToOne(() => Category, category => category.products, {
  nullable: true, // Required for 'nullify'
})
category: Category | null;

// ❌ BAD: orphanedRowAction without cascade
@OneToMany(() => Comment, comment => comment.post, {
  orphanedRowAction: 'delete', // ❌ Won't work!
})
comments: Comment[];

// ❌ BAD: Using 'nullify' with non-nullable FK
@OneToMany(() => Product, product => product.category, {
  cascade: true,
  orphanedRowAction: 'nullify',
})
products: Product[];

@ManyToOne(() => Category, category => category.products)
category: Category; // ❌ Should be nullable

// ✅ GOOD: Use for composition relationships
// Order → OrderItems (composition)
@OneToMany(() => OrderItem, item => item.order, {
  cascade: true,
  orphanedRowAction: 'delete',
})
items: OrderItem[];

// ✅ GOOD: Don't use for association relationships
// Post → Author (association - author exists independently)
@ManyToOne(() => User, user => user.posts)
author: User; // No orphanedRowAction on Many side
```

**Interview Tip**: Explain that `orphanedRowAction` handles child entities **removed from parent's collection** but not explicitly deleted. Emphasize three options: **'delete'** (remove from database - for owned entities), **'nullify'** (set FK to NULL - default, for independent entities), **'disable'** (do nothing). Highlight critical requirement: **cascade must be enabled** for orphanedRowAction to work. Discuss use cases: 'delete' for composition relationships (CartItems in Cart, Photos in Album), 'nullify' for association (Products in Category - products exist independently). Mention it only triggers when parent is saved (not child) and child is removed from collection. For production: use 'delete' for dependent/owned entities, 'nullify' for independent entities (ensure nullable FK), consider manual deletion for more control. A strong answer demonstrates understanding of composition vs association and automatic cleanup of orphaned records.

</details>

<details>
<summary>40. How do you handle circular dependencies in relationships?</summary>

**Circular dependencies** occur when entities have mutual relationships that create import cycles. TypeORM resolves this using **forward references** with arrow functions `() => Entity` and the **forwardRef** pattern.

### Problem: Circular Dependency

```typescript
// ❌ CIRCULAR IMPORT ERROR
// user.entity.ts
import { Post } from './post.entity';

@Entity()
export class User {
  @OneToMany(() => Post, post => post.author) // ❌ Post not defined yet
  posts: Post[];
}

// post.entity.ts
import { User } from './user.entity';

@Entity()
export class Post {
  @ManyToOne(() => User, user => user.posts) // ❌ User not defined yet
  author: User;
}

/*
Error: Cannot access 'User' before initialization
OR
Error: Cannot access 'Post' before initialization
*/
```

### Solution 1: Arrow Function (Recommended)

```typescript
// user.entity.ts
import { Entity, PrimaryGeneratedColumn, Column, OneToMany } from 'typeorm';
import { Post } from './post.entity';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // ✅ Arrow function delays evaluation
  @OneToMany(() => Post, post => post.author)
  posts: Post[];
}

// post.entity.ts
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne } from 'typeorm';
import { User } from './user.entity';

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  // ✅ Arrow function delays evaluation
  @ManyToOne(() => User, user => user.posts)
  author: User;
}

/*
Arrow functions delay entity reference evaluation until runtime,
avoiding circular dependency during module loading.
*/
```

### Solution 2: String-based Type Reference

```typescript
// user.entity.ts
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // ✅ String reference (less type-safe)
  @OneToMany('Post', 'author')
  posts: Post[];
}

// post.entity.ts
@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  // ✅ String reference
  @ManyToOne('User', 'posts')
  author: User;
}

/*
String-based references work but lose TypeScript type safety.
Arrow functions are preferred.
*/
```

### Real-World Example: Blog System

```typescript
// user.entity.ts
import { Entity, PrimaryGeneratedColumn, Column, OneToMany } from 'typeorm';
import { Post } from './post.entity';
import { Comment } from './comment.entity';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  username: string;

  @Column()
  email: string;

  @OneToMany(() => Post, post => post.author)
  posts: Post[];

  @OneToMany(() => Comment, comment => comment.author)
  comments: Comment[];
}

// post.entity.ts
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne, OneToMany } from 'typeorm';
import { User } from './user.entity';
import { Comment } from './comment.entity';
import { Tag } from './tag.entity';

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column('text')
  content: string;

  @ManyToOne(() => User, user => user.posts)
  author: User;

  @OneToMany(() => Comment, comment => comment.post)
  comments: Comment[];

  @ManyToMany(() => Tag, tag => tag.posts)
  @JoinTable()
  tags: Tag[];
}

// comment.entity.ts
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne } from 'typeorm';
import { User } from './user.entity';
import { Post } from './post.entity';

@Entity()
export class Comment {
  @PrimaryGeneratedColumn()
  id: number;

  @Column('text')
  content: string;

  @ManyToOne(() => User, user => user.comments)
  author: User;

  @ManyToOne(() => Post, post => post.comments)
  post: Post;
}

// tag.entity.ts
import { Entity, PrimaryGeneratedColumn, Column, ManyToMany } from 'typeorm';
import { Post } from './post.entity';

@Entity()
export class Tag {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @ManyToMany(() => Post, post => post.tags)
  posts: Post[];
}
```

### Module Organization for Circular Dependencies

```typescript
// entities/index.ts - Barrel export
export { User } from './user.entity';
export { Post } from './post.entity';
export { Comment } from './comment.entity';
export { Tag } from './tag.entity';

// user.entity.ts
import { Post, Comment } from './index'; // ✅ Import from barrel

@Entity()
export class User {
  @OneToMany(() => Post, post => post.author)
  posts: Post[];

  @OneToMany(() => Comment, comment => comment.author)
  comments: Comment[];
}

// post.entity.ts
import { User, Comment, Tag } from './index';

@Entity()
export class Post {
  @ManyToOne(() => User, user => user.posts)
  author: User;

  @OneToMany(() => Comment, comment => comment.post)
  comments: Comment[];

  @ManyToMany(() => Tag, tag => tag.posts)
  @JoinTable()
  tags: Tag[];
}
```

### Complex Circular: Self-Referencing + Other Entities

```typescript
// employee.entity.ts
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne, OneToMany } from 'typeorm';
import { Department } from './department.entity';

@Entity()
export class Employee {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Self-referencing
  @ManyToOne(() => Employee, employee => employee.subordinates, {
    nullable: true,
  })
  manager: Employee | null;

  @OneToMany(() => Employee, employee => employee.manager)
  subordinates: Employee[];

  // Circular with Department
  @ManyToOne(() => Department, department => department.employees)
  department: Department;
}

// department.entity.ts
import { Entity, PrimaryGeneratedColumn, Column, OneToMany, ManyToOne } from 'typeorm';
import { Employee } from './employee.entity';

@Entity()
export class Department {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Circular with Employee
  @OneToMany(() => Employee, employee => employee.department)
  employees: Employee[];

  // Self-referencing manager
  @ManyToOne(() => Employee, { nullable: true })
  manager: Employee | null;
}
```

### Service Layer with Circular Dependencies

```typescript
// user.service.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './entities/user.entity';
import { PostService } from './post.service'; // ❌ Potential circular

@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>,
    private postService: PostService // ❌ Circular dependency
  ) {}

  async getUserWithPosts(userId: number) {
    return await this.userRepository.findOne({
      where: { id: userId },
      relations: ['posts'],
    });
  }
}

// post.service.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Post } from './entities/post.entity';
import { UserService } from './user.service'; // ❌ Circular dependency

@Injectable()
export class PostService {
  constructor(
    @InjectRepository(Post)
    private postRepository: Repository<Post>,
    private userService: UserService // ❌ Circular dependency
  ) {}

  async getPostWithAuthor(postId: number) {
    return await this.postRepository.findOne({
      where: { id: postId },
      relations: ['author'],
    });
  }
}
```

### Solution: forwardRef in NestJS

```typescript
// user.service.ts
import { Injectable, Inject, forwardRef } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './entities/user.entity';
import { PostService } from './post.service';

@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>,
    @Inject(forwardRef(() => PostService)) // ✅ Forward reference
    private postService: PostService
  ) {}

  async getUserStats(userId: number) {
    const user = await this.userRepository.findOneBy({ id: userId });
    const postCount = await this.postService.countByUser(userId);
    return { ...user, postCount };
  }
}

// post.service.ts
import { Injectable, Inject, forwardRef } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Post } from './entities/post.entity';
import { UserService } from './user.service';

@Injectable()
export class PostService {
  constructor(
    @InjectRepository(Post)
    private postRepository: Repository<Post>,
    @Inject(forwardRef(() => UserService)) // ✅ Forward reference
    private userService: UserService
  ) {}

  async countByUser(userId: number): Promise<number> {
    return await this.postRepository.count({
      where: { author: { id: userId } },
    });
  }

  async getPostWithAuthor(postId: number) {
    return await this.postRepository.findOne({
      where: { id: postId },
      relations: ['author'],
    });
  }
}
```

### Better Solution: Avoid Service Circular Dependencies

```typescript
// user.service.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './entities/user.entity';
import { Post } from './entities/post.entity';

@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>,
    @InjectRepository(Post) // ✅ Inject repository, not service
    private postRepository: Repository<Post>
  ) {}

  async getUserWithStats(userId: number) {
    const user = await this.userRepository.findOneBy({ id: userId });

    // Query directly without other service
    const postCount = await this.postRepository.count({
      where: { author: { id: userId } },
    });

    return { ...user, postCount };
  }
}

// post.service.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Post } from './entities/post.entity';
import { User } from './entities/user.entity';

@Injectable()
export class PostService {
  constructor(
    @InjectRepository(Post)
    private postRepository: Repository<Post>,
    @InjectRepository(User) // ✅ Inject repository, not service
    private userRepository: Repository<User>
  ) {}

  async getPostWithAuthor(postId: number) {
    return await this.postRepository.findOne({
      where: { id: postId },
      relations: ['author'],
    });
  }
}
```

### Extract Common Logic to Shared Service

```typescript
// blog.service.ts - Shared service
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './entities/user.entity';
import { Post } from './entities/post.entity';
import { Comment } from './entities/comment.entity';

@Injectable()
export class BlogService {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>,
    @InjectRepository(Post)
    private postRepository: Repository<Post>,
    @InjectRepository(Comment)
    private commentRepository: Repository<Comment>
  ) {}

  async getUserStats(userId: number) {
    const [postCount, commentCount] = await Promise.all([
      this.postRepository.count({ where: { author: { id: userId } } }),
      this.commentRepository.count({ where: { author: { id: userId } } }),
    ]);

    return { postCount, commentCount };
  }

  async getPostStats(postId: number) {
    const commentCount = await this.commentRepository.count({
      where: { post: { id: postId } },
    });

    return { commentCount };
  }
}

// user.service.ts - Uses shared service
import { Injectable } from '@nestjs/common';
import { BlogService } from './blog.service';

@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>,
    private blogService: BlogService // ✅ No circular dependency
  ) {}

  async getUserProfile(userId: number) {
    const user = await this.userRepository.findOneBy({ id: userId });
    const stats = await this.blogService.getUserStats(userId);
    return { ...user, ...stats };
  }
}
```

### Best Practices

```typescript
// ✅ GOOD: Always use arrow functions in entity relations
@OneToMany(() => Post, post => post.author)
posts: Post[];

// ❌ BAD: Direct class reference (may cause circular dependency)
@OneToMany(Post, post => post.author) // ❌ Avoid
posts: Post[];

// ✅ GOOD: Barrel exports for entity modules
// entities/index.ts
export * from './user.entity';
export * from './post.entity';
export * from './comment.entity';

// ✅ GOOD: Import from barrel
import { User, Post, Comment } from './entities';

// ✅ GOOD: Use forwardRef for service circular dependencies
@Inject(forwardRef(() => PostService))
private postService: PostService;

// ✅ BETTER: Avoid service circular dependencies
// Inject repositories instead of services
@InjectRepository(Post)
private postRepository: Repository<Post>;

// ✅ BEST: Extract common logic to shared service
private sharedService: SharedService;

// ✅ GOOD: Organize by feature modules
/*
src/
  modules/
    user/
      user.entity.ts
      user.service.ts
      user.module.ts
    post/
      post.entity.ts
      post.service.ts
      post.module.ts
    shared/
      shared.service.ts // Common logic
*/
```

### TypeScript Configuration

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2021",
    "module": "commonjs",
    "moduleResolution": "node",
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
    "strictPropertyInitialization": false
  }
}
```

**Interview Tip**: Explain that circular dependencies occur when entities import each other, creating module load cycles. Emphasize the solution: **arrow functions** `() => Entity` in relationship decorators delay evaluation until runtime. Show syntax: `@ManyToOne(() => User, user => user.posts)` instead of direct class reference. Discuss alternative: string-based references ('User', 'posts') but arrow functions are preferred (type-safe). Highlight service layer circulars: use **forwardRef** in NestJS (`@Inject(forwardRef(() => Service))`) but better to avoid by injecting repositories instead of services, or extract common logic to shared service. Mention module organization: barrel exports (index.ts) help manage imports. For production: always use arrow functions in entity relations, avoid service circular dependencies (inject repositories), use forwardRef as last resort, organize by feature modules. A strong answer demonstrates understanding that entity circular dependencies are solved by TypeORM's arrow function pattern, while service circulars need architectural solutions.

</details>