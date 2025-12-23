
## **Repositories**

<details>
<summary>41. What is a Repository in TypeORM?</summary>

A **Repository** in TypeORM is a design pattern that provides a collection-like interface for accessing and managing entities. It encapsulates data access logic and provides methods for CRUD operations on a specific entity type.

### Basic Repository Concept

```typescript
import { DataSource, Repository } from 'typeorm';
import { User } from './entities/user.entity';

// Repository is a wrapper around EntityManager for a specific entity
// It provides type-safe methods for working with that entity

const dataSource = new DataSource({
  type: 'postgres',
  // ... connection options
  entities: [User],
});

await dataSource.initialize();

// Get repository for User entity
const userRepository: Repository<User> = dataSource.getRepository(User);

// Now you can use repository methods
const user = await userRepository.findOne({ where: { id: 1 } });
const allUsers = await userRepository.find();
await userRepository.save(user);
```

### What is the Repository Pattern?

```typescript
/*
REPOSITORY PATTERN:
- Mediates between domain and data mapping layers
- Acts like an in-memory collection of entities
- Provides centralized, consistent access to data
- Abstracts database operations from business logic

BENEFITS:
✅ Type Safety: Strongly typed for specific entity
✅ Convenience: Pre-built methods (find, save, remove, etc.)
✅ Consistency: Standardized data access across app
✅ Testability: Easy to mock for unit tests
✅ Separation: Business logic separated from data access

COMMON METHODS:
- find(), findOne(), findOneBy(), findAndCount()
- save(), insert(), update()
- remove(), delete(), softDelete()
- count(), increment(), decrement()
- createQueryBuilder()
*/
```

### Repository vs Direct Database Access

```typescript
// ❌ WITHOUT Repository (Direct SQL)
import { DataSource } from 'typeorm';

const dataSource = new DataSource(/* config */);
await dataSource.initialize();

// Manual SQL queries
const users = await dataSource.query('SELECT * FROM users WHERE age > $1', [18]);
const user = await dataSource.query('SELECT * FROM users WHERE id = $1', [1]);

// Problems:
// - No type safety
// - Manual SQL writing
// - Error-prone
// - Hard to maintain

// ✅ WITH Repository (TypeORM Pattern)
import { User } from './entities/user.entity';

const userRepository = dataSource.getRepository(User);

// Type-safe, clean API
const users = await userRepository.find({
  where: { age: MoreThan(18) },
});

const user = await userRepository.findOne({
  where: { id: 1 },
});

// Benefits:
// ✅ Type-safe
// ✅ Clean, readable API
// ✅ Auto-generates SQL
// ✅ Easy to maintain
```

### Repository Structure

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

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

  @Column({ default: true })
  isActive: boolean;
}

// Repository provides methods for User entity
class UserService {
  constructor(
    private userRepository: Repository<User>
  ) {}

  // CRUD operations using repository
  async createUser(data: Partial<User>): Promise<User> {
    const user = this.userRepository.create(data);
    return await this.userRepository.save(user);
  }

  async getUserById(id: number): Promise<User | null> {
    return await this.userRepository.findOne({
      where: { id },
    });
  }

  async getAllUsers(): Promise<User[]> {
    return await this.userRepository.find();
  }

  async updateUser(id: number, data: Partial<User>): Promise<User> {
    await this.userRepository.update(id, data);
    return await this.getUserById(id);
  }

  async deleteUser(id: number): Promise<void> {
    await this.userRepository.delete(id);
  }

  async getActiveUsers(): Promise<User[]> {
    return await this.userRepository.find({
      where: { isActive: true },
    });
  }

  async getUserCount(): Promise<number> {
    return await this.userRepository.count();
  }
}
```

### Real-World Example: Blog System

```typescript
// entities/post.entity.ts
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

  @Column({ default: false })
  published: boolean;

  @Column({ default: 0 })
  viewCount: number;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}

// services/post.service.ts
import { Repository } from 'typeorm';
import { Post } from '../entities/post.entity';

export class PostService {
  constructor(
    private postRepository: Repository<Post>
  ) {}

  // Create new post
  async createPost(title: string, content: string, authorId: number): Promise<Post> {
    const post = this.postRepository.create({
      title,
      content,
      author: { id: authorId } as User,
    });

    return await this.postRepository.save(post);
  }

  // Get all published posts
  async getPublishedPosts(): Promise<Post[]> {
    return await this.postRepository.find({
      where: { published: true },
      relations: ['author'],
      order: { createdAt: 'DESC' },
    });
  }

  // Get post by ID with author
  async getPostById(id: number): Promise<Post | null> {
    return await this.postRepository.findOne({
      where: { id },
      relations: ['author'],
    });
  }

  // Publish post
  async publishPost(id: number): Promise<void> {
    await this.postRepository.update(id, { published: true });
  }

  // Increment view count
  async incrementViewCount(id: number): Promise<void> {
    await this.postRepository.increment({ id }, 'viewCount', 1);
  }

  // Get posts by author
  async getPostsByAuthor(authorId: number): Promise<Post[]> {
    return await this.postRepository.find({
      where: { author: { id: authorId } },
      order: { createdAt: 'DESC' },
    });
  }

  // Search posts
  async searchPosts(query: string): Promise<Post[]> {
    return await this.postRepository
      .createQueryBuilder('post')
      .where('post.title ILIKE :query', { query: `%${query}%` })
      .orWhere('post.content ILIKE :query', { query: `%${query}%` })
      .andWhere('post.published = :published', { published: true })
      .leftJoinAndSelect('post.author', 'author')
      .orderBy('post.createdAt', 'DESC')
      .getMany();
  }

  // Get popular posts
  async getPopularPosts(limit: number = 10): Promise<Post[]> {
    return await this.postRepository.find({
      where: { published: true },
      order: { viewCount: 'DESC' },
      take: limit,
      relations: ['author'],
    });
  }

  // Delete post
  async deletePost(id: number): Promise<void> {
    await this.postRepository.delete(id);
  }
}
```

### E-commerce Example: Product Repository

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

  @Column('int')
  stock: number;

  @ManyToOne(() => Category, category => category.products)
  category: Category;

  @Column({ default: true })
  isActive: boolean;

  @CreateDateColumn()
  createdAt: Date;
}

export class ProductService {
  constructor(
    private productRepository: Repository<Product>
  ) {}

  async createProduct(data: {
    name: string;
    description: string;
    price: number;
    stock: number;
    categoryId: number;
  }): Promise<Product> {
    const product = this.productRepository.create({
      ...data,
      category: { id: data.categoryId } as Category,
    });

    return await this.productRepository.save(product);
  }

  async getAvailableProducts(): Promise<Product[]> {
    return await this.productRepository.find({
      where: {
        isActive: true,
        stock: MoreThan(0),
      },
      relations: ['category'],
      order: { createdAt: 'DESC' },
    });
  }

  async getProductsByCategory(categoryId: number): Promise<Product[]> {
    return await this.productRepository.find({
      where: {
        category: { id: categoryId },
        isActive: true,
      },
      order: { name: 'ASC' },
    });
  }

  async getProductsInPriceRange(minPrice: number, maxPrice: number): Promise<Product[]> {
    return await this.productRepository.find({
      where: {
        price: Between(minPrice, maxPrice),
        isActive: true,
      },
      order: { price: 'ASC' },
    });
  }

  async updateStock(productId: number, quantity: number): Promise<void> {
    await this.productRepository.increment(
      { id: productId },
      'stock',
      quantity
    );
  }

  async checkStock(productId: number): Promise<number> {
    const product = await this.productRepository.findOne({
      where: { id: productId },
      select: ['stock'],
    });

    return product?.stock || 0;
  }

  async getLowStockProducts(threshold: number = 10): Promise<Product[]> {
    return await this.productRepository.find({
      where: {
        stock: LessThanOrEqual(threshold),
        isActive: true,
      },
      order: { stock: 'ASC' },
    });
  }

  async deactivateProduct(productId: number): Promise<void> {
    await this.productRepository.update(productId, { isActive: false });
  }
}
```

### Repository Benefits in Testing

```typescript
// Easy to mock for unit tests
describe('PostService', () => {
  let postService: PostService;
  let mockPostRepository: jest.Mocked<Repository<Post>>;

  beforeEach(() => {
    // Mock repository
    mockPostRepository = {
      find: jest.fn(),
      findOne: jest.fn(),
      save: jest.fn(),
      create: jest.fn(),
      update: jest.fn(),
      delete: jest.fn(),
    } as any;

    postService = new PostService(mockPostRepository);
  });

  it('should get published posts', async () => {
    const mockPosts = [
      { id: 1, title: 'Post 1', published: true },
      { id: 2, title: 'Post 2', published: true },
    ] as Post[];

    mockPostRepository.find.mockResolvedValue(mockPosts);

    const result = await postService.getPublishedPosts();

    expect(result).toEqual(mockPosts);
    expect(mockPostRepository.find).toHaveBeenCalledWith({
      where: { published: true },
      relations: ['author'],
      order: { createdAt: 'DESC' },
    });
  });

  it('should create post', async () => {
    const mockPost = {
      id: 1,
      title: 'New Post',
      content: 'Content',
    } as Post;

    mockPostRepository.create.mockReturnValue(mockPost);
    mockPostRepository.save.mockResolvedValue(mockPost);

    const result = await postService.createPost('New Post', 'Content', 1);

    expect(result).toEqual(mockPost);
    expect(mockPostRepository.save).toHaveBeenCalled();
  });
});
```

### Repository with Transactions

```typescript
import { DataSource } from 'typeorm';

export class OrderService {
  constructor(
    private dataSource: DataSource,
    private orderRepository: Repository<Order>,
    private productRepository: Repository<Product>
  ) {}

  async createOrder(userId: number, items: { productId: number; quantity: number }[]): Promise<Order> {
    // Use transaction with repository
    return await this.dataSource.transaction(async (manager) => {
      // Get repositories within transaction
      const orderRepo = manager.getRepository(Order);
      const productRepo = manager.getRepository(Product);

      // Create order
      const order = orderRepo.create({
        user: { id: userId } as User,
        status: 'pending',
      });

      await orderRepo.save(order);

      // Process items and update stock
      for (const item of items) {
        const product = await productRepo.findOne({
          where: { id: item.productId },
        });

        if (!product || product.stock < item.quantity) {
          throw new Error(`Insufficient stock for product ${item.productId}`);
        }

        // Decrease stock
        await productRepo.decrement(
          { id: item.productId },
          'stock',
          item.quantity
        );

        // Create order item
        const orderItem = manager.getRepository(OrderItem).create({
          order,
          product,
          quantity: item.quantity,
          price: product.price,
        });

        await manager.save(orderItem);
      }

      return order;
    });
  }
}
```

### Best Practices

```typescript
// ✅ GOOD: Inject repository in service
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>
  ) {}
}

// ✅ GOOD: Use repository methods for common operations
await userRepository.find({ where: { isActive: true } });

// ✅ GOOD: Use QueryBuilder for complex queries
await userRepository
  .createQueryBuilder('user')
  .where('user.age > :age', { age: 18 })
  .andWhere('user.isActive = :active', { active: true })
  .getMany();

// ✅ GOOD: Repository per entity
const userRepository = dataSource.getRepository(User);
const postRepository = dataSource.getRepository(Post);

// ❌ BAD: Don't use EntityManager for everything
const user = await entityManager.query('SELECT * FROM users WHERE id = $1', [1]);

// ✅ GOOD: Use repository with type safety
const user = await userRepository.findOne({ where: { id: 1 } });

// ✅ GOOD: Create custom repositories for complex logic
export class UserRepository extends Repository<User> {
  async findActiveUsers(): Promise<User[]> {
    return this.find({ where: { isActive: true } });
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.findOne({ where: { email } });
  }
}

// ✅ GOOD: Use transactions for multi-step operations
await dataSource.transaction(async (manager) => {
  const userRepo = manager.getRepository(User);
  const profileRepo = manager.getRepository(Profile);
  
  await userRepo.save(user);
  await profileRepo.save(profile);
});
```

**Interview Tip**: Explain that a **Repository** is a design pattern providing a collection-like interface for entity access, acting as a layer between business logic and data access. Emphasize it provides **type-safe, pre-built methods** (find, save, remove, etc.) for CRUD operations on specific entities. Highlight benefits: type safety (strongly typed for entity), convenience (no manual SQL), testability (easy to mock), separation of concerns (business logic isolated from data access). Mention it's obtained via `dataSource.getRepository(Entity)` and provides methods like find(), save(), remove(), createQueryBuilder(). Discuss real-world usage: inject repositories in services, use for common operations, extend for custom methods. For production: one repository per entity, use QueryBuilder for complex queries, wrap multi-step operations in transactions, mock repositories for unit tests. A strong answer demonstrates understanding of the Repository pattern's role in clean architecture.

</details>

<details>
<summary>42. How do you get a repository instance?</summary>

There are multiple ways to get a **Repository** instance in TypeORM, depending on your setup: using **DataSource**, **EntityManager**, or **dependency injection** (in frameworks like NestJS).

### Method 1: Using DataSource (Recommended)

```typescript
import { DataSource } from 'typeorm';
import { User } from './entities/user.entity';

// Initialize DataSource
const dataSource = new DataSource({
  type: 'postgres',
  host: 'localhost',
  port: 5432,
  username: 'postgres',
  password: 'password',
  database: 'mydb',
  entities: [User],
  synchronize: true,
});

await dataSource.initialize();

// Get repository from DataSource
const userRepository = dataSource.getRepository(User);

// Use repository
const users = await userRepository.find();
const user = await userRepository.findOne({ where: { id: 1 } });
await userRepository.save(user);
```

### Method 2: Using EntityManager

```typescript
import { DataSource } from 'typeorm';
import { User } from './entities/user.entity';

const dataSource = new DataSource(/* config */);
await dataSource.initialize();

// Get EntityManager from DataSource
const entityManager = dataSource.manager;

// Get repository from EntityManager
const userRepository = entityManager.getRepository(User);

// Use repository
const users = await userRepository.find();
```

### Method 3: NestJS Dependency Injection (Most Common in NestJS)

```typescript
// user.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { User } from './entities/user.entity';
import { UserService } from './user.service';
import { UserController } from './user.controller';

@Module({
  imports: [
    TypeOrmModule.forFeature([User]), // Register entity
  ],
  providers: [UserService],
  controllers: [UserController],
})
export class UserModule {}

// user.service.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './entities/user.entity';

@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User) // Inject repository
    private userRepository: Repository<User>
  ) {}

  async findAll(): Promise<User[]> {
    return await this.userRepository.find();
  }

  async findOne(id: number): Promise<User> {
    return await this.userRepository.findOne({ where: { id } });
  }

  async create(userData: Partial<User>): Promise<User> {
    const user = this.userRepository.create(userData);
    return await this.userRepository.save(user);
  }
}

// user.controller.ts
import { Controller, Get, Post, Body, Param } from '@nestjs/common';
import { UserService } from './user.service';
import { User } from './entities/user.entity';

@Controller('users')
export class UserController {
  constructor(private userService: UserService) {}

  @Get()
  async findAll(): Promise<User[]> {
    return await this.userService.findAll();
  }

  @Get(':id')
  async findOne(@Param('id') id: number): Promise<User> {
    return await this.userService.findOne(id);
  }

  @Post()
  async create(@Body() userData: Partial<User>): Promise<User> {
    return await this.userService.create(userData);
  }
}
```

### Method 4: Using getRepository() Helper (Deprecated)

```typescript
import { getRepository } from 'typeorm';
import { User } from './entities/user.entity';

// ⚠️ DEPRECATED in TypeORM 0.3+
// This method is no longer recommended
const userRepository = getRepository(User);

// ✅ RECOMMENDED: Use DataSource instead
const dataSource = new DataSource(/* config */);
await dataSource.initialize();
const userRepository = dataSource.getRepository(User);
```

### Real-World Example: Express.js Application

```typescript
// src/data-source.ts
import { DataSource } from 'typeorm';
import { User } from './entities/user.entity';
import { Post } from './entities/post.entity';

export const AppDataSource = new DataSource({
  type: 'postgres',
  host: process.env.DB_HOST || 'localhost',
  port: parseInt(process.env.DB_PORT) || 5432,
  username: process.env.DB_USER || 'postgres',
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME || 'myapp',
  entities: [User, Post],
  synchronize: process.env.NODE_ENV === 'development',
  logging: process.env.NODE_ENV === 'development',
});

// src/services/user.service.ts
import { Repository } from 'typeorm';
import { User } from '../entities/user.entity';
import { AppDataSource } from '../data-source';

export class UserService {
  private userRepository: Repository<User>;

  constructor() {
    // Get repository from DataSource
    this.userRepository = AppDataSource.getRepository(User);
  }

  async getAllUsers(): Promise<User[]> {
    return await this.userRepository.find();
  }

  async getUserById(id: number): Promise<User | null> {
    return await this.userRepository.findOne({ where: { id } });
  }

  async createUser(data: Partial<User>): Promise<User> {
    const user = this.userRepository.create(data);
    return await this.userRepository.save(user);
  }

  async updateUser(id: number, data: Partial<User>): Promise<User | null> {
    await this.userRepository.update(id, data);
    return await this.getUserById(id);
  }

  async deleteUser(id: number): Promise<void> {
    await this.userRepository.delete(id);
  }
}

// src/controllers/user.controller.ts
import { Request, Response } from 'express';
import { UserService } from '../services/user.service';

export class UserController {
  private userService: UserService;

  constructor() {
    this.userService = new UserService();
  }

  async getAll(req: Request, res: Response): Promise<void> {
    try {
      const users = await this.userService.getAllUsers();
      res.json(users);
    } catch (error) {
      res.status(500).json({ error: error.message });
    }
  }

  async getOne(req: Request, res: Response): Promise<void> {
    try {
      const id = parseInt(req.params.id);
      const user = await this.userService.getUserById(id);
      
      if (!user) {
        res.status(404).json({ error: 'User not found' });
        return;
      }
      
      res.json(user);
    } catch (error) {
      res.status(500).json({ error: error.message });
    }
  }

  async create(req: Request, res: Response): Promise<void> {
    try {
      const user = await this.userService.createUser(req.body);
      res.status(201).json(user);
    } catch (error) {
      res.status(400).json({ error: error.message });
    }
  }
}

// src/app.ts
import express from 'express';
import { AppDataSource } from './data-source';
import { UserController } from './controllers/user.controller';

const app = express();
app.use(express.json());

const userController = new UserController();

// Routes
app.get('/users', (req, res) => userController.getAll(req, res));
app.get('/users/:id', (req, res) => userController.getOne(req, res));
app.post('/users', (req, res) => userController.create(req, res));

// Initialize DataSource and start server
AppDataSource.initialize()
  .then(() => {
    console.log('Database connected');
    app.listen(3000, () => {
      console.log('Server running on port 3000');
    });
  })
  .catch((error) => {
    console.error('Database connection error:', error);
  });
```

### Getting Multiple Repositories

```typescript
import { DataSource } from 'typeorm';
import { User } from './entities/user.entity';
import { Post } from './entities/post.entity';
import { Comment } from './entities/comment.entity';

const dataSource = new DataSource(/* config */);
await dataSource.initialize();

// Get multiple repositories
const userRepository = dataSource.getRepository(User);
const postRepository = dataSource.getRepository(Post);
const commentRepository = dataSource.getRepository(Comment);

// Use in service
export class BlogService {
  constructor(
    private dataSource: DataSource
  ) {}

  async createPostWithComments(
    userId: number,
    postData: Partial<Post>,
    comments: string[]
  ): Promise<Post> {
    const userRepo = this.dataSource.getRepository(User);
    const postRepo = this.dataSource.getRepository(Post);
    const commentRepo = this.dataSource.getRepository(Comment);

    // Get user
    const user = await userRepo.findOne({ where: { id: userId } });

    if (!user) {
      throw new Error('User not found');
    }

    // Create post
    const post = postRepo.create({
      ...postData,
      author: user,
    });

    await postRepo.save(post);

    // Create comments
    for (const commentText of comments) {
      const comment = commentRepo.create({
        content: commentText,
        post,
        author: user,
      });

      await commentRepo.save(comment);
    }

    // Return post with comments
    return await postRepo.findOne({
      where: { id: post.id },
      relations: ['comments', 'author'],
    });
  }
}
```

### Repository in Transaction

```typescript
import { DataSource } from 'typeorm';
import { User } from './entities/user.entity';
import { Account } from './entities/account.entity';

export class TransferService {
  constructor(private dataSource: DataSource) {}

  async transferMoney(
    fromUserId: number,
    toUserId: number,
    amount: number
  ): Promise<void> {
    await this.dataSource.transaction(async (manager) => {
      // Get repositories within transaction
      const accountRepo = manager.getRepository(Account);

      // Get accounts
      const fromAccount = await accountRepo.findOne({
        where: { user: { id: fromUserId } },
      });

      const toAccount = await accountRepo.findOne({
        where: { user: { id: toUserId } },
      });

      if (!fromAccount || !toAccount) {
        throw new Error('Account not found');
      }

      if (fromAccount.balance < amount) {
        throw new Error('Insufficient funds');
      }

      // Deduct from sender
      fromAccount.balance -= amount;
      await accountRepo.save(fromAccount);

      // Add to receiver
      toAccount.balance += amount;
      await accountRepo.save(toAccount);
    });
  }
}
```

### Singleton Pattern for DataSource

```typescript
// src/database/data-source.ts
import { DataSource } from 'typeorm';
import { User } from '../entities/user.entity';

class DatabaseConnection {
  private static instance: DataSource;

  private constructor() {}

  public static async getInstance(): Promise<DataSource> {
    if (!DatabaseConnection.instance) {
      DatabaseConnection.instance = new DataSource({
        type: 'postgres',
        host: 'localhost',
        port: 5432,
        username: 'postgres',
        password: 'password',
        database: 'mydb',
        entities: [User],
        synchronize: true,
      });

      await DatabaseConnection.instance.initialize();
    }

    return DatabaseConnection.instance;
  }
}

// Usage in services
export class UserService {
  private userRepository: Repository<User>;

  async initialize() {
    const dataSource = await DatabaseConnection.getInstance();
    this.userRepository = dataSource.getRepository(User);
  }

  async getUsers(): Promise<User[]> {
    return await this.userRepository.find();
  }
}
```

### Comparison of Methods

```typescript
/*
METHOD 1: DataSource.getRepository() - Vanilla TypeORM
✅ Direct control
✅ Works in any Node.js app
✅ Good for Express, Koa, Fastify
❌ Manual dependency management
Use: Standalone apps, Express apps

METHOD 2: EntityManager.getRepository()
✅ Works within transactions
✅ Access via manager
❌ Extra indirection
Use: Transaction contexts

METHOD 3: @InjectRepository() - NestJS
✅ Automatic dependency injection
✅ Testable (easy mocking)
✅ Clean, declarative
❌ Framework-specific
Use: NestJS applications (RECOMMENDED for NestJS)

METHOD 4: getRepository() - DEPRECATED
❌ No longer supported in TypeORM 0.3+
❌ Used global connection
❌ Migration required
Use: AVOID - migrate to DataSource
*/
```

### Best Practices

```typescript
// ✅ GOOD: Initialize DataSource once
const AppDataSource = new DataSource(/* config */);
await AppDataSource.initialize();

// ✅ GOOD: Reuse DataSource across app
const userRepository = AppDataSource.getRepository(User);
const postRepository = AppDataSource.getRepository(Post);

// ✅ GOOD: NestJS - Use @InjectRepository
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>
  ) {}
}

// ✅ GOOD: Get repository in transaction
await dataSource.transaction(async (manager) => {
  const repo = manager.getRepository(User);
  await repo.save(user);
});

// ❌ BAD: Creating multiple DataSource instances
const ds1 = new DataSource(config);
const ds2 = new DataSource(config); // Unnecessary

// ❌ BAD: Using deprecated getRepository()
import { getRepository } from 'typeorm';
const repo = getRepository(User); // DEPRECATED

// ✅ GOOD: Use DataSource
const repo = dataSource.getRepository(User);

// ✅ GOOD: Export DataSource for reuse
// data-source.ts
export const AppDataSource = new DataSource(config);

// user.service.ts
import { AppDataSource } from './data-source';
const userRepository = AppDataSource.getRepository(User);
```

**Interview Tip**: Explain there are three main ways to get a Repository: **DataSource.getRepository()** (vanilla TypeORM, most flexible), **EntityManager.getRepository()** (within transactions), and **@InjectRepository()** (NestJS dependency injection). Emphasize DataSource approach is recommended for TypeORM 0.3+ replacing deprecated `getRepository()` helper. Highlight NestJS pattern: register entity with `TypeOrmModule.forFeature([Entity])`, then inject with `@InjectRepository(Entity)` - this enables automatic DI and easy mocking for tests. Mention transaction context: use `manager.getRepository()` inside `dataSource.transaction()` to ensure operations use same transaction. For production: initialize DataSource once as singleton, reuse across app, use DI in frameworks like NestJS, get repository from manager in transactions. A strong answer demonstrates understanding of different contexts (standalone vs framework) and when to use each method.

</details>

<details>
<summary>43. What is the difference between Repository and EntityManager?</summary>

**Repository** and **EntityManager** are both used for database operations in TypeORM, but they differ in scope, usage patterns, and type safety. Repository is **entity-specific** while EntityManager is **generic** for all entities.

### Basic Differences

```typescript
import { DataSource, Repository, EntityManager } from 'typeorm';
import { User } from './entities/user.entity';
import { Post } from './entities/post.entity';

const dataSource = new DataSource(/* config */);
await dataSource.initialize();

// REPOSITORY: Specific to ONE entity
const userRepository: Repository<User> = dataSource.getRepository(User);
const postRepository: Repository<Post> = dataSource.getRepository(Post);

// Only works with User entity
await userRepository.find(); // Returns User[]
await userRepository.save(user); // Saves User

// ENTITY MANAGER: Works with ALL entities
const entityManager: EntityManager = dataSource.manager;

// Works with any entity (pass entity class)
await entityManager.find(User); // Returns User[]
await entityManager.find(Post); // Returns Post[]
await entityManager.save(user); // Saves User
await entityManager.save(post); // Saves Post
```

### Key Differences Table

```typescript
/*
╔════════════════════════╤════════════════════════╤════════════════════════╗
║ FEATURE                │ REPOSITORY             │ ENTITY MANAGER         ║
╠════════════════════════╪════════════════════════╪════════════════════════╣
║ Scope                  │ Single entity type     │ All entities           ║
║ Type Safety            │ ✅ Strong (typed)      │ ⚠️ Weaker (generic)   ║
║ API Style              │ Concise (no entity)    │ Verbose (pass entity)  ║
║ Usage Pattern          │ Specific operations    │ General operations     ║
║ Transaction Context    │ Per entity             │ Global context         ║
║ Extends                │ Can extend             │ Cannot extend          ║
║ Custom Methods         │ ✅ Easy to add         │ ❌ Not recommended     ║
║ Recommended For        │ Most cases             │ Transactions, generic  ║
╚════════════════════════╧════════════════════════╧════════════════════════╝
*/
```

### Type Safety Comparison

```typescript
// REPOSITORY: Strong type safety
const userRepository = dataSource.getRepository(User);

// ✅ Type-safe: knows return type is User | null
const user: User | null = await userRepository.findOne({
  where: { id: 1 },
});

// ✅ TypeScript knows this returns User[]
const users: User[] = await userRepository.find();

// ✅ TypeScript enforces User type
const newUser = userRepository.create({
  firstName: 'John',
  lastName: 'Doe',
});

// ENTITY MANAGER: Less type safety
const entityManager = dataSource.manager;

// ⚠️ Must specify entity class each time
const user = await entityManager.findOne(User, {
  where: { id: 1 },
});

// ⚠️ More verbose, pass entity class
const users = await entityManager.find(User);

// ⚠️ Must specify entity
const newUser = entityManager.create(User, {
  firstName: 'John',
  lastName: 'Doe',
});
```

### API Comparison

```typescript
import { User } from './entities/user.entity';

const userRepository = dataSource.getRepository(User);
const entityManager = dataSource.manager;

// FIND OPERATIONS

// Repository: Concise
await userRepository.find();
await userRepository.findOne({ where: { id: 1 } });
await userRepository.findOneBy({ email: 'test@example.com' });
await userRepository.findAndCount();

// EntityManager: Requires entity class
await entityManager.find(User);
await entityManager.findOne(User, { where: { id: 1 } });
await entityManager.findOneBy(User, { email: 'test@example.com' });
await entityManager.findAndCount(User);

// SAVE OPERATIONS

// Repository
const user = userRepository.create({ firstName: 'John' });
await userRepository.save(user);

// EntityManager
const user = entityManager.create(User, { firstName: 'John' });
await entityManager.save(user);

// UPDATE OPERATIONS

// Repository
await userRepository.update(1, { firstName: 'Jane' });

// EntityManager
await entityManager.update(User, 1, { firstName: 'Jane' });

// DELETE OPERATIONS

// Repository
await userRepository.delete(1);
await userRepository.remove(user);

// EntityManager
await entityManager.delete(User, 1);
await entityManager.remove(user);

// COUNT

// Repository
await userRepository.count();
await userRepository.countBy({ isActive: true });

// EntityManager
await entityManager.count(User);
await entityManager.countBy(User, { isActive: true });
```

### When to Use Each

```typescript
// ✅ USE REPOSITORY: Most common operations
export class UserService {
  constructor(
    private userRepository: Repository<User>
  ) {}

  async getUsers(): Promise<User[]> {
    // Clean, type-safe
    return await this.userRepository.find();
  }

  async getUserById(id: number): Promise<User | null> {
    return await this.userRepository.findOne({ where: { id } });
  }

  async createUser(data: Partial<User>): Promise<User> {
    const user = this.userRepository.create(data);
    return await this.userRepository.save(user);
  }
}

// ✅ USE ENTITY MANAGER: Transactions with multiple entities
export class OrderService {
  constructor(
    private dataSource: DataSource
  ) {}

  async createOrder(orderData: any): Promise<Order> {
    return await this.dataSource.transaction(async (manager) => {
      // EntityManager handles transaction context
      // Works with multiple entities in same transaction

      const user = await manager.findOne(User, {
        where: { id: orderData.userId },
      });

      const order = manager.create(Order, {
        user,
        status: 'pending',
      });

      await manager.save(order);

      for (const item of orderData.items) {
        const product = await manager.findOne(Product, {
          where: { id: item.productId },
        });

        const orderItem = manager.create(OrderItem, {
          order,
          product,
          quantity: item.quantity,
        });

        await manager.save(orderItem);

        // Update product stock
        await manager.update(Product, product.id, {
          stock: product.stock - item.quantity,
        });
      }

      return order;
    });
  }
}
```

### Repository with Custom Methods

```typescript
// ✅ Repository: Easy to extend with custom methods
import { DataSource, Repository } from 'typeorm';
import { User } from './entities/user.entity';

export class UserRepository extends Repository<User> {
  constructor(private dataSource: DataSource) {
    super(User, dataSource.createEntityManager());
  }

  // Custom method: Find active users
  async findActiveUsers(): Promise<User[]> {
    return this.find({ where: { isActive: true } });
  }

  // Custom method: Find by email
  async findByEmail(email: string): Promise<User | null> {
    return this.findOne({ where: { email } });
  }

  // Custom method: Find users with posts
  async findUsersWithPosts(): Promise<User[]> {
    return this.createQueryBuilder('user')
      .leftJoinAndSelect('user.posts', 'posts')
      .where('posts.id IS NOT NULL')
      .getMany();
  }

  // Custom method: Get user statistics
  async getUserStats(userId: number): Promise<any> {
    return this.createQueryBuilder('user')
      .select('user.id', 'userId')
      .addSelect('COUNT(DISTINCT posts.id)', 'postCount')
      .addSelect('COUNT(DISTINCT comments.id)', 'commentCount')
      .leftJoin('user.posts', 'posts')
      .leftJoin('user.comments', 'comments')
      .where('user.id = :userId', { userId })
      .groupBy('user.id')
      .getRawOne();
  }
}

// Usage
const userRepository = new UserRepository(dataSource);
const activeUsers = await userRepository.findActiveUsers();
const user = await userRepository.findByEmail('test@example.com');
```

### EntityManager for Generic Operations

```typescript
// EntityManager: Generic operations across multiple entities
export class GenericDataService {
  constructor(private entityManager: EntityManager) {}

  // Generic save for any entity
  async saveEntity<T>(entity: T): Promise<T> {
    return await this.entityManager.save(entity);
  }

  // Generic find for any entity type
  async findById<T>(entityClass: new () => T, id: number): Promise<T | null> {
    return await this.entityManager.findOne(entityClass as any, {
      where: { id } as any,
    });
  }

  // Bulk operations on multiple entity types
  async bulkSave(entities: any[]): Promise<void> {
    await this.entityManager.save(entities);
  }

  // Clear data from multiple tables
  async clearDatabase(): Promise<void> {
    await this.entityManager.clear(User);
    await this.entityManager.clear(Post);
    await this.entityManager.clear(Comment);
  }

  // Count all records across entity
  async countAll(entityClass: any): Promise<number> {
    return await this.entityManager.count(entityClass);
  }
}
```

### Transaction Comparison

```typescript
// REPOSITORY: Get fresh repository per transaction
await dataSource.transaction(async (manager) => {
  // Must get repository from transaction manager
  const userRepo = manager.getRepository(User);
  const postRepo = manager.getRepository(Post);

  const user = await userRepo.save(newUser);
  const post = await postRepo.save(newPost);
});

// ENTITY MANAGER: Already have manager in transaction
await dataSource.transaction(async (manager) => {
  // manager IS the EntityManager
  const user = await manager.save(newUser);
  const post = await manager.save(newPost);
  
  // Can work with any entity
  await manager.update(User, userId, { lastLogin: new Date() });
  await manager.increment(Post, { id: postId }, 'viewCount', 1);
});
```

### Real-World Example: Blog Service

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  username: string;

  @OneToMany(() => Post, post => post.author)
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

// Using REPOSITORY for specific operations
export class PostService {
  constructor(
    private postRepository: Repository<Post>,
    private userRepository: Repository<User>
  ) {}

  // Clean, type-safe repository operations
  async getPublishedPosts(): Promise<Post[]> {
    return await this.postRepository.find({
      where: { published: true },
      relations: ['author'],
      order: { createdAt: 'DESC' },
    });
  }

  async getPostsByAuthor(authorId: number): Promise<Post[]> {
    return await this.postRepository.find({
      where: { author: { id: authorId } },
    });
  }
}

// Using ENTITY MANAGER for transactions
export class BlogTransactionService {
  constructor(private dataSource: DataSource) {}

  async createUserWithPost(
    userData: Partial<User>,
    postData: Partial<Post>
  ): Promise<{ user: User; post: Post }> {
    return await this.dataSource.transaction(async (manager) => {
      // EntityManager for multiple entities in transaction

      // Create user
      const user = manager.create(User, userData);
      await manager.save(user);

      // Create post
      const post = manager.create(Post, {
        ...postData,
        author: user,
      });
      await manager.save(post);

      return { user, post };
    });
  }

  async transferPostOwnership(
    postId: number,
    newAuthorId: number
  ): Promise<void> {
    await this.dataSource.transaction(async (manager) => {
      const post = await manager.findOne(Post, {
        where: { id: postId },
      });

      const newAuthor = await manager.findOne(User, {
        where: { id: newAuthorId },
      });

      if (!post || !newAuthor) {
        throw new Error('Post or User not found');
      }

      post.author = newAuthor;
      await manager.save(post);

      // Update user stats
      await manager.increment(
        User,
        { id: newAuthorId },
        'postCount',
        1
      );
    });
  }
}
```

### Performance and Use Cases

```typescript
/*
REPOSITORY - Best for:
✅ Single entity operations
✅ Type-safe operations
✅ Custom business logic
✅ Service layer (one service per entity)
✅ Extending with custom methods
✅ Clear, readable code

ENTITY MANAGER - Best for:
✅ Transactions with multiple entities
✅ Generic/dynamic operations
✅ Bulk operations across entities
✅ Migration scripts
✅ Admin/utility functions
✅ Complex multi-entity workflows
*/
```

### Best Practices

```typescript
// ✅ GOOD: Use Repository for most operations
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>
  ) {}

  async getUsers() {
    return await this.userRepository.find();
  }
}

// ✅ GOOD: Use EntityManager for transactions
@Injectable()
export class OrderService {
  constructor(private dataSource: DataSource) {}

  async createOrder(data: any) {
    return await this.dataSource.transaction(async (manager) => {
      // Work with multiple entities
      await manager.save(order);
      await manager.save(items);
      await manager.update(Product, productId, { stock: newStock });
    });
  }
}

// ✅ GOOD: Extend Repository for custom methods
export class UserRepository extends Repository<User> {
  async findActiveUsers() {
    return this.find({ where: { isActive: true } });
  }
}

// ❌ BAD: Using EntityManager for simple operations
async getUser(id: number) {
  // Verbose, less type-safe
  return await this.entityManager.findOne(User, { where: { id } });
}

// ✅ GOOD: Using Repository for simple operations
async getUser(id: number) {
  // Clean, type-safe
  return await this.userRepository.findOne({ where: { id } });
}

// ✅ GOOD: Repository per entity in service
export class BlogService {
  constructor(
    private userRepository: Repository<User>,
    private postRepository: Repository<Post>,
    private commentRepository: Repository<Comment>
  ) {}
}

// ✅ GOOD: EntityManager for multi-entity transactions
export class BlogService {
  constructor(private dataSource: DataSource) {}

  async complexOperation() {
    return await this.dataSource.transaction(async (manager) => {
      // Multiple entities in single transaction
    });
  }
}
```

**Interview Tip**: Explain that **Repository** is entity-specific with strong type safety (concise API, no need to pass entity class), while **EntityManager** is generic for all entities (must pass entity class each time, more verbose). Emphasize Repository is recommended for most cases: single entity operations, custom methods, service layer - provides cleaner, type-safe code. Mention EntityManager is better for: transactions with multiple entities, generic/dynamic operations, bulk operations across different entity types. Highlight key difference: Repository methods like `find()` automatically know the entity type, EntityManager requires `find(Entity, ...)` for every call. Discuss that Repository can be extended with custom business logic methods, EntityManager cannot. For production: use Repository in services (one repository per entity), use EntityManager in transactions, prefer Repository for type safety and maintainability. A strong answer demonstrates understanding of when each tool is appropriate based on scope and type safety needs.

</details>

<details>
<summary>44. What methods are available in the Repository API?</summary>

The **Repository API** provides a comprehensive set of methods for CRUD operations, queries, and entity management. These methods cover finding, saving, updating, deleting, counting, and advanced operations.

### Core Repository Methods Overview

```typescript
import { Repository, DataSource } from 'typeorm';
import { User } from './entities/user.entity';

const dataSource = new DataSource(/* config */);
await dataSource.initialize();

const userRepository: Repository<User> = dataSource.getRepository(User);

/*
REPOSITORY API METHODS:

1. QUERY METHODS (Read):
   - find()                    - Find multiple entities
   - findOne()                 - Find single entity
   - findOneBy()               - Find one by simple conditions
   - findOneByOrFail()         - Find one or throw error
   - findBy()                  - Find by simple conditions
   - findAndCount()            - Find entities and count
   - findAndCountBy()          - Find and count by simple conditions
   - count()                   - Count entities
   - countBy()                 - Count by conditions
   - sum()                     - Sum column values
   - average()                 - Average of column
   - minimum()                 - Minimum value
   - maximum()                 - Maximum value
   - exist()                   - Check if entity exists
   - existBy()                 - Check existence by conditions

2. MUTATION METHODS (Create/Update):
   - save()                    - Insert or update entity
   - insert()                  - Insert new records
   - update()                  - Update existing records
   - upsert()                  - Insert or update (conflict handling)

3. DELETION METHODS:
   - remove()                  - Remove entities (cascades)
   - delete()                  - Delete by criteria
   - softRemove()              - Soft remove entities
   - softDelete()              - Soft delete by criteria
   - restore()                 - Restore soft-deleted
   - recover()                 - Recover soft-removed entities

4. UTILITY METHODS:
   - create()                  - Create entity instance
   - preload()                 - Create entity from plain object
   - merge()                   - Merge multiple entities
   - increment()               - Increment column value
   - decrement()               - Decrement column value

5. QUERY BUILDER:
   - createQueryBuilder()      - Create query builder

6. METADATA:
   - getId()                   - Get entity ID
   - hasId()                   - Check if entity has ID
   - metadata                  - Entity metadata
   - target                    - Entity class
   - manager                   - EntityManager instance
*/
```

### 1. Query Methods (Read Operations)

```typescript
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

  @Column({ default: true })
  isActive: boolean;

  @Column('decimal', { precision: 10, scale: 2, default: 0 })
  balance: number;
}

const userRepository = dataSource.getRepository(User);

// find() - Find all matching entities
const allUsers = await userRepository.find();

const activeUsers = await userRepository.find({
  where: { isActive: true },
  order: { lastName: 'ASC' },
  take: 10,
  skip: 0,
});

// findOne() - Find single entity
const user = await userRepository.findOne({
  where: { id: 1 },
  relations: ['posts'],
});

// findOneBy() - Simplified findOne
const userByEmail = await userRepository.findOneBy({
  email: 'test@example.com',
});

// findOneByOrFail() - Throws error if not found
try {
  const user = await userRepository.findOneByOrFail({ id: 999 });
} catch (error) {
  console.error('User not found');
}

// findBy() - Simplified find
const activeUsersSimple = await userRepository.findBy({
  isActive: true,
});

// findAndCount() - Get entities and total count
const [users, totalCount] = await userRepository.findAndCount({
  where: { isActive: true },
  take: 10,
  skip: 0,
});

console.log(`Found ${users.length} users out of ${totalCount} total`);

// findAndCountBy() - Simplified findAndCount
const [usersSimple, count] = await userRepository.findAndCountBy({
  isActive: true,
});

// count() - Count all entities
const totalUsers = await userRepository.count();

// countBy() - Count with conditions
const activeUserCount = await userRepository.countBy({
  isActive: true,
});

// sum() - Sum column values
const totalBalance = await userRepository.sum('balance', {
  isActive: true,
});

// average() - Calculate average
const avgAge = await userRepository.average('age', {
  isActive: true,
});

// minimum() - Find minimum value
const minAge = await userRepository.minimum('age');

// maximum() - Find maximum value
const maxBalance = await userRepository.maximum('balance');

// exist() - Check if any entity exists
const hasUsers = await userRepository.exist();

// existBy() - Check existence with conditions
const hasActiveUsers = await userRepository.existBy({
  isActive: true,
});
```

### 2. Mutation Methods (Create/Update)

```typescript
// create() - Create entity instance (not saved)
const user = userRepository.create({
  firstName: 'John',
  lastName: 'Doe',
  email: 'john@example.com',
  age: 30,
});

// Create multiple instances
const users = userRepository.create([
  { firstName: 'John', lastName: 'Doe', email: 'john@example.com', age: 30 },
  { firstName: 'Jane', lastName: 'Smith', email: 'jane@example.com', age: 25 },
]);

// save() - Insert or update entity
const savedUser = await userRepository.save(user);

// Save multiple entities
const savedUsers = await userRepository.save([user1, user2, user3]);

// insert() - Insert new records (faster than save)
await userRepository.insert({
  firstName: 'John',
  lastName: 'Doe',
  email: 'john@example.com',
  age: 30,
});

// Insert multiple records
await userRepository.insert([
  { firstName: 'John', lastName: 'Doe', email: 'john@example.com', age: 30 },
  { firstName: 'Jane', lastName: 'Smith', email: 'jane@example.com', age: 25 },
]);

// update() - Update existing records
await userRepository.update(1, {
  firstName: 'Johnny',
});

// Update by criteria
await userRepository.update(
  { isActive: false },
  { isActive: true }
);

// upsert() - Insert or update on conflict
await userRepository.upsert(
  {
    email: 'john@example.com',
    firstName: 'John',
    lastName: 'Doe',
    age: 30,
  },
  ['email'] // Conflict target (unique column)
);

// preload() - Create entity from partial data
const partialUser = {
  id: 1,
  firstName: 'Updated Name',
};

const preloadedUser = await userRepository.preload(partialUser);
if (preloadedUser) {
  await userRepository.save(preloadedUser);
}

// merge() - Merge multiple entities
const user1 = await userRepository.findOne({ where: { id: 1 } });
const updates = { firstName: 'Updated' };

const mergedUser = userRepository.merge(user1, updates, { age: 35 });
await userRepository.save(mergedUser);

// increment() - Increment column value
await userRepository.increment(
  { id: 1 },
  'balance',
  100
);

// Increment by condition
await userRepository.increment(
  { isActive: true },
  'balance',
  50
);

// decrement() - Decrement column value
await userRepository.decrement(
  { id: 1 },
  'balance',
  50
);
```

### 3. Deletion Methods

```typescript
// remove() - Remove entities (requires loading first, respects cascades)
const user = await userRepository.findOne({ where: { id: 1 } });
await userRepository.remove(user);

// Remove multiple entities
const users = await userRepository.find({ where: { isActive: false } });
await userRepository.remove(users);

// delete() - Delete by criteria (doesn't load entities, faster)
await userRepository.delete(1);

// Delete by criteria
await userRepository.delete({ isActive: false });

// Delete multiple IDs
await userRepository.delete([1, 2, 3]);

// softRemove() - Soft remove loaded entities
const user = await userRepository.findOne({ where: { id: 1 } });
await userRepository.softRemove(user);

// softDelete() - Soft delete by criteria
await userRepository.softDelete(1);

await userRepository.softDelete({ isActive: false });

// restore() - Restore soft-deleted entities
await userRepository.restore(1);

await userRepository.restore({ email: 'john@example.com' });

// recover() - Recover soft-removed entities
const user = await userRepository.findOne({
  where: { id: 1 },
  withDeleted: true,
});

await userRepository.recover(user);
```

### 4. Query Builder

```typescript
// createQueryBuilder() - Create advanced queries
const users = await userRepository
  .createQueryBuilder('user')
  .where('user.age > :age', { age: 18 })
  .andWhere('user.isActive = :active', { active: true })
  .orderBy('user.lastName', 'ASC')
  .take(10)
  .getMany();

// With joins
const usersWithPosts = await userRepository
  .createQueryBuilder('user')
  .leftJoinAndSelect('user.posts', 'posts')
  .where('posts.published = :published', { published: true })
  .getMany();

// Aggregations
const result = await userRepository
  .createQueryBuilder('user')
  .select('AVG(user.age)', 'averageAge')
  .addSelect('COUNT(user.id)', 'totalUsers')
  .where('user.isActive = :active', { active: true })
  .getRawOne();
```

### 5. Utility Methods

```typescript
// getId() - Get entity ID
const user = await userRepository.findOne({ where: { email: 'john@example.com' } });
const userId = userRepository.getId(user);

// hasId() - Check if entity has ID
const newUser = userRepository.create({ firstName: 'John' });
const hasId = userRepository.hasId(newUser); // false

const savedUser = await userRepository.save(newUser);
const hasIdNow = userRepository.hasId(savedUser); // true

// metadata - Access entity metadata
const metadata = userRepository.metadata;
console.log(metadata.tableName); // 'user'
console.log(metadata.columns); // Column metadata

// target - Get entity class
const entityClass = userRepository.target; // User class

// manager - Get EntityManager
const manager = userRepository.manager;
```

### Real-World Example: Complete User Service

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column({ unique: true })
  email: string;

  @Column()
  age: number;

  @Column({ default: true })
  isActive: boolean;

  @Column('decimal', { precision: 10, scale: 2, default: 0 })
  balance: number;

  @Column({ default: 0 })
  loginCount: number;

  @DeleteDateColumn()
  deletedAt: Date;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  @OneToMany(() => Post, post => post.author)
  posts: Post[];
}

export class UserService {
  constructor(
    private userRepository: Repository<User>
  ) {}

  // CREATE operations
  async createUser(data: Partial<User>): Promise<User> {
    const user = this.userRepository.create(data);
    return await this.userRepository.save(user);
  }

  async createMultipleUsers(usersData: Partial<User>[]): Promise<User[]> {
    const users = this.userRepository.create(usersData);
    return await this.userRepository.save(users);
  }

  async bulkInsert(usersData: Partial<User>[]): Promise<void> {
    await this.userRepository.insert(usersData);
  }

  // READ operations
  async getAllUsers(): Promise<User[]> {
    return await this.userRepository.find({
      order: { createdAt: 'DESC' },
    });
  }

  async getUserById(id: number): Promise<User | null> {
    return await this.userRepository.findOne({
      where: { id },
      relations: ['posts'],
    });
  }

  async getUserByEmail(email: string): Promise<User | null> {
    return await this.userRepository.findOneBy({ email });
  }

  async getActiveUsers(): Promise<User[]> {
    return await this.userRepository.findBy({ isActive: true });
  }

  async getUsersWithPagination(
    page: number = 1,
    limit: number = 10
  ): Promise<{ users: User[]; total: number; page: number; totalPages: number }> {
    const [users, total] = await this.userRepository.findAndCount({
      where: { isActive: true },
      order: { createdAt: 'DESC' },
      take: limit,
      skip: (page - 1) * limit,
    });

    return {
      users,
      total,
      page,
      totalPages: Math.ceil(total / limit),
    };
  }

  async getUserCount(): Promise<number> {
    return await this.userRepository.count();
  }

  async getActiveUserCount(): Promise<number> {
    return await this.userRepository.countBy({ isActive: true });
  }

  async getUserStats(): Promise<any> {
    const totalUsers = await this.userRepository.count();
    const activeUsers = await this.userRepository.countBy({ isActive: true });
    const totalBalance = await this.userRepository.sum('balance', { isActive: true });
    const avgAge = await this.userRepository.average('age', { isActive: true });

    return {
      totalUsers,
      activeUsers,
      totalBalance,
      averageAge: avgAge,
    };
  }

  async searchUsers(query: string): Promise<User[]> {
    return await this.userRepository
      .createQueryBuilder('user')
      .where('user.firstName ILIKE :query', { query: `%${query}%` })
      .orWhere('user.lastName ILIKE :query', { query: `%${query}%` })
      .orWhere('user.email ILIKE :query', { query: `%${query}%` })
      .orderBy('user.lastName', 'ASC')
      .getMany();
  }

  // UPDATE operations
  async updateUser(id: number, data: Partial<User>): Promise<void> {
    await this.userRepository.update(id, data);
  }

  async updateUserBalance(id: number, amount: number): Promise<void> {
    if (amount > 0) {
      await this.userRepository.increment({ id }, 'balance', amount);
    } else {
      await this.userRepository.decrement({ id }, 'balance', Math.abs(amount));
    }
  }

  async incrementLoginCount(userId: number): Promise<void> {
    await this.userRepository.increment({ id: userId }, 'loginCount', 1);
  }

  async activateUser(id: number): Promise<void> {
    await this.userRepository.update(id, { isActive: true });
  }

  async deactivateUser(id: number): Promise<void> {
    await this.userRepository.update(id, { isActive: false });
  }

  async activateInactiveUsers(): Promise<void> {
    await this.userRepository.update({ isActive: false }, { isActive: true });
  }

  // DELETE operations
  async deleteUser(id: number): Promise<void> {
    await this.userRepository.delete(id);
  }

  async deleteInactiveUsers(): Promise<void> {
    await this.userRepository.delete({ isActive: false });
  }

  async softDeleteUser(id: number): Promise<void> {
    await this.userRepository.softDelete(id);
  }

  async restoreUser(id: number): Promise<void> {
    await this.userRepository.restore(id);
  }

  async permanentlyDeleteUser(id: number): Promise<void> {
    const user = await this.userRepository.findOne({
      where: { id },
      withDeleted: true,
    });

    if (user) {
      await this.userRepository.remove(user);
    }
  }

  // UTILITY operations
  async userExists(email: string): Promise<boolean> {
    return await this.userRepository.existBy({ email });
  }

  async hasUsers(): Promise<boolean> {
    return await this.userRepository.exist();
  }

  async getUserWithHighestBalance(): Promise<User | null> {
    return await this.userRepository
      .createQueryBuilder('user')
      .orderBy('user.balance', 'DESC')
      .limit(1)
      .getOne();
  }

  async getYoungestUser(): Promise<User | null> {
    return await this.userRepository
      .createQueryBuilder('user')
      .orderBy('user.age', 'ASC')
      .limit(1)
      .getOne();
  }
}
```

### Method Categories Summary

```typescript
/*
QUERY METHODS (Non-mutating):
✅ find() - Most flexible, supports complex options
✅ findOne() - Single entity with options
✅ findBy() - Simple conditions
✅ findAndCount() - Pagination support
✅ count(), countBy() - Quick counts
✅ sum(), average(), min(), max() - Aggregations
✅ exist(), existBy() - Existence checks

MUTATION METHODS:
✅ save() - Insert or update (smart, slower)
✅ insert() - Insert only (faster, no hooks)
✅ update() - Update by criteria (fast)
✅ upsert() - Handle conflicts
✅ increment(), decrement() - Atomic operations

DELETION METHODS:
✅ remove() - Respects cascades, triggers hooks
✅ delete() - Fast, bypasses hooks
✅ softDelete() - Soft delete support
✅ restore() - Undo soft deletes

UTILITY METHODS:
✅ create() - Create instances
✅ merge() - Combine entities
✅ preload() - Load and merge
✅ createQueryBuilder() - Advanced queries

CHOOSE BASED ON:
- save() for entities with relations/hooks
- insert() for bulk inserts (performance)
- update() for simple updates (fast)
- remove() when cascades matter
- delete() for performance
- QueryBuilder for complex queries
*/
```

### Best Practices

```typescript
// ✅ GOOD: Use appropriate method for task
// For single insert with relations
await userRepository.save(user);

// For bulk inserts without relations
await userRepository.insert(users);

// For simple updates
await userRepository.update(id, data);

// ✅ GOOD: Use exist() instead of count() for checks
const hasUsers = await userRepository.exist();
// Instead of: const count = await userRepository.count(); if (count > 0)

// ✅ GOOD: Use findBy() for simple queries
const activeUsers = await userRepository.findBy({ isActive: true });
// Instead of: find({ where: { isActive: true } })

// ✅ GOOD: Use increment/decrement for atomic operations
await userRepository.increment({ id: 1 }, 'balance', 100);
// Instead of: load, modify, save

// ✅ GOOD: Use Query Builder for complex queries
const result = await userRepository
  .createQueryBuilder('user')
  .where('user.age BETWEEN :min AND :max', { min: 18, max: 65 })
  .andWhere('user.isActive = :active', { active: true })
  .getMany();

// ❌ BAD: Using find() when exist() suffices
const users = await userRepository.find();
if (users.length > 0) { /* ... */ }

// ✅ GOOD: Using exist()
if (await userRepository.exist()) { /* ... */ }
```

**Interview Tip**: Explain that Repository provides **comprehensive methods** grouped into categories: **Query** (find, count, aggregations), **Mutation** (save, insert, update), **Deletion** (remove, delete, soft deletes), and **Utility** (create, merge, QueryBuilder). Emphasize choosing the right method: `save()` for entities with relations/hooks, `insert()` for bulk operations (faster), `update()` for simple updates, `remove()` when cascades matter, `delete()` for performance. Highlight useful methods: `findAndCount()` for pagination, `increment()/decrement()` for atomic operations, `exist()` for checks (faster than count), `upsert()` for conflict handling. Mention Query Builder for complex queries. For production: use `findBy()` for simple conditions (cleaner), `exist()` instead of `count() > 0`, atomic operations for counters, appropriate method based on performance needs. A strong answer demonstrates understanding of when to use each method based on requirements and performance considerations.

</details>

<details>
<summary>45. How do you use the save() method?</summary>

The **save()** method is used to persist entities to the database. It's **smart** - it can both **insert new entities** and **update existing ones** based on whether the entity has an ID. It also triggers lifecycle hooks and handles relations.

### Basic save() Usage

```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';
import { Repository } from 'typeorm';

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
}

const userRepository = dataSource.getRepository(User);

// INSERT: Save new entity (no ID)
const newUser = userRepository.create({
  firstName: 'John',
  lastName: 'Doe',
  email: 'john@example.com',
});

const savedUser = await userRepository.save(newUser);
console.log(savedUser.id); // Generated ID (e.g., 1)

// UPDATE: Save existing entity (has ID)
savedUser.firstName = 'Johnny';
const updatedUser = await userRepository.save(savedUser);
console.log(updatedUser.firstName); // 'Johnny'
```

### How save() Works

```typescript
/*
save() BEHAVIOR:

1. IF entity has NO ID (or ID is undefined):
   → INSERT new record
   → Generate ID
   → Return entity with ID

2. IF entity has ID:
   → UPDATE existing record
   → Keep same ID
   → Return updated entity

3. ALWAYS:
   → Triggers lifecycle hooks (@BeforeInsert, @AfterInsert, etc.)
   → Handles relations (if cascade enabled)
   → Returns the saved entity
   → Works with transactions

KEY CHARACTERISTICS:
✅ Smart: Determines insert vs update automatically
✅ Returns entity: Get back saved entity with ID
✅ Lifecycle hooks: Triggers decorators
✅ Relations: Saves related entities (with cascade)
✅ Transactions: Participates in transactions
❌ Slower: Loads data first (for updates)
❌ Not bulk-friendly: Use insert() for bulk operations
*/
```

### save() vs Direct Creation

```typescript
// Option 1: Create first, then save
const user = userRepository.create({
  firstName: 'John',
  lastName: 'Doe',
  email: 'john@example.com',
});

await userRepository.save(user);

// Option 2: Save directly
await userRepository.save({
  firstName: 'John',
  lastName: 'Doe',
  email: 'john@example.com',
});

// Both work, but create() is recommended:
// - Type safety
// - Apply default values
// - Initialize entity properly
```

### Saving Multiple Entities

```typescript
// Save array of entities
const users = [
  userRepository.create({ firstName: 'John', lastName: 'Doe', email: 'john@example.com' }),
  userRepository.create({ firstName: 'Jane', lastName: 'Smith', email: 'jane@example.com' }),
  userRepository.create({ firstName: 'Bob', lastName: 'Johnson', email: 'bob@example.com' }),
];

// Save all at once (more efficient)
const savedUsers = await userRepository.save(users);

console.log(savedUsers.length); // 3
console.log(savedUsers[0].id); // Generated ID

// Each user now has an ID
savedUsers.forEach(user => {
  console.log(`${user.firstName} - ID: ${user.id}`);
});
```

### save() with Relations

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToOne(() => Profile, profile => profile.user, {
    cascade: true, // Enable cascade
  })
  @JoinColumn()
  profile: Profile;

  @OneToMany(() => Post, post => post.author, {
    cascade: true,
  })
  posts: Post[];
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

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToOne(() => User, user => user.posts)
  author: User;
}

// Save user with profile (cascade enabled)
const user = userRepository.create({
  name: 'John Doe',
  profile: {
    bio: 'Software Developer',
  },
});

await userRepository.save(user);
// Both user AND profile are saved (cascade)

// Save user with posts
const userWithPosts = userRepository.create({
  name: 'Jane Smith',
  posts: [
    { title: 'First Post' },
    { title: 'Second Post' },
  ],
});

await userRepository.save(userWithPosts);
// User and all posts are saved (cascade)
```

### save() with Lifecycle Hooks

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  email: string;

  @Column()
  password: string;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  @BeforeInsert()
  async hashPassword() {
    if (this.password) {
      // Hash password before insert
      this.password = await bcrypt.hash(this.password, 10);
    }
  }

  @BeforeUpdate()
  async updatePassword() {
    if (this.password && !this.password.startsWith('$2')) {
      // Hash password if it was changed
      this.password = await bcrypt.hash(this.password, 10);
    }
  }

  @AfterInsert()
  logInsert() {
    console.log(`User ${this.email} created with ID ${this.id}`);
  }

  @AfterUpdate()
  logUpdate() {
    console.log(`User ${this.email} updated`);
  }
}

// INSERT: Triggers @BeforeInsert and @AfterInsert
const user = userRepository.create({
  email: 'john@example.com',
  password: 'plaintext123',
});

await userRepository.save(user);
// Password is hashed before saving
// Log: "User john@example.com created with ID 1"

// UPDATE: Triggers @BeforeUpdate and @AfterUpdate
user.email = 'newemail@example.com';
await userRepository.save(user);
// Log: "User newemail@example.com updated"
```

### save() in Transactions

```typescript
import { DataSource } from 'typeorm';

const dataSource = new DataSource(/* config */);

// save() within transaction
await dataSource.transaction(async (manager) => {
  const userRepo = manager.getRepository(User);
  const postRepo = manager.getRepository(Post);

  // Create user
  const user = userRepo.create({
    firstName: 'John',
    lastName: 'Doe',
    email: 'john@example.com',
  });

  await userRepo.save(user);

  // Create posts for user
  const post1 = postRepo.create({
    title: 'First Post',
    author: user,
  });

  const post2 = postRepo.create({
    title: 'Second Post',
    author: user,
  });

  await postRepo.save([post1, post2]);

  // All operations committed together
  // If any fails, all rollback
});
```

### Real-World Example: User Registration

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ unique: true })
  email: string;

  @Column()
  password: string;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column({ default: false })
  isEmailVerified: boolean;

  @OneToOne(() => Profile, profile => profile.user, {
    cascade: true,
  })
  @JoinColumn()
  profile: Profile;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  @BeforeInsert()
  @BeforeUpdate()
  async hashPassword() {
    if (this.password && !this.password.startsWith('$2')) {
      this.password = await bcrypt.hash(this.password, 10);
    }
  }

  @BeforeInsert()
  normalizeEmail() {
    this.email = this.email.toLowerCase().trim();
  }
}

@Entity()
export class Profile {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ nullable: true })
  bio: string;

  @Column({ nullable: true })
  avatarUrl: string;

  @Column({ nullable: true })
  phone: string;

  @OneToOne(() => User, user => user.profile)
  user: User;
}

// Service
export class AuthService {
  constructor(
    private userRepository: Repository<User>
  ) {}

  async register(data: {
    email: string;
    password: string;
    firstName: string;
    lastName: string;
    bio?: string;
  }): Promise<User> {
    // Check if user exists
    const existingUser = await this.userRepository.findOneBy({
      email: data.email,
    });

    if (existingUser) {
      throw new Error('User already exists');
    }

    // Create user with profile
    const user = this.userRepository.create({
      email: data.email,
      password: data.password,
      firstName: data.firstName,
      lastName: data.lastName,
      profile: {
        bio: data.bio || '',
      },
    });

    // Save user (and profile due to cascade)
    // Triggers @BeforeInsert hooks (hash password, normalize email)
    const savedUser = await this.userRepository.save(user);

    // Remove password from response
    delete savedUser.password;

    return savedUser;
  }

  async updateProfile(userId: number, data: {
    firstName?: string;
    lastName?: string;
    bio?: string;
    phone?: string;
  }): Promise<User> {
    // Load user with profile
    const user = await this.userRepository.findOne({
      where: { id: userId },
      relations: ['profile'],
    });

    if (!user) {
      throw new Error('User not found');
    }

    // Update user fields
    if (data.firstName) user.firstName = data.firstName;
    if (data.lastName) user.lastName = data.lastName;

    // Update profile fields
    if (data.bio !== undefined) user.profile.bio = data.bio;
    if (data.phone !== undefined) user.profile.phone = data.phone;

    // Save updates (triggers @BeforeUpdate)
    const updatedUser = await this.userRepository.save(user);

    delete updatedUser.password;
    return updatedUser;
  }

  async changePassword(userId: number, newPassword: string): Promise<void> {
    const user = await this.userRepository.findOneBy({ id: userId });

    if (!user) {
      throw new Error('User not found');
    }

    // Update password
    user.password = newPassword;

    // Save (password will be hashed in @BeforeUpdate)
    await this.userRepository.save(user);
  }
}
```

### save() with Partial Updates

```typescript
// Load entity first, then update specific fields
const user = await userRepository.findOneBy({ id: 1 });

if (user) {
  user.firstName = 'Updated Name';
  user.email = 'newemail@example.com';
  
  await userRepository.save(user);
}

// Only changed fields are updated
// updatedAt timestamp automatically updated
```

### save() Options

```typescript
// save() with options
await userRepository.save(user, {
  // Chunk size for bulk saves
  chunk: 1000,
  
  // Reload entity after save
  reload: true,
  
  // Additional transaction options
  transaction: true,
  
  // Listen for save events
  listeners: true,
});

// Example with chunk (for large datasets)
const thousandUsers = []; // 10000 users
for (let i = 0; i < 10000; i++) {
  thousandUsers.push(userRepository.create({
    firstName: `User${i}`,
    lastName: `Last${i}`,
    email: `user${i}@example.com`,
  }));
}

// Save in chunks of 1000
await userRepository.save(thousandUsers, { chunk: 1000 });
```

### Performance Considerations

```typescript
// ❌ SLOW: Save one at a time
for (const userData of usersData) {
  const user = userRepository.create(userData);
  await userRepository.save(user); // Multiple DB trips
}

// ✅ FAST: Save all at once
const users = userRepository.create(usersData);
await userRepository.save(users); // Single DB trip

// ✅ FASTER: Use insert() for bulk inserts (no hooks/relations)
await userRepository.insert(usersData);

// When to use what:
// - save() single entity: Relations, hooks, need returned entity
// - save() multiple: Moderate bulk with hooks
// - insert(): Large bulk, no hooks needed, best performance
```

### Common save() Patterns

```typescript
// Pattern 1: Create and save
const user = userRepository.create(userData);
await userRepository.save(user);

// Pattern 2: Load, modify, save
const user = await userRepository.findOneBy({ id: 1 });
user.firstName = 'Updated';
await userRepository.save(user);

// Pattern 3: Upsert pattern
async function upsertUser(email: string, data: Partial<User>): Promise<User> {
  let user = await userRepository.findOneBy({ email });
  
  if (!user) {
    // Create new
    user = userRepository.create({ email, ...data });
  } else {
    // Update existing
    Object.assign(user, data);
  }
  
  return await userRepository.save(user);
}

// Pattern 4: Save with error handling
try {
  const user = userRepository.create(userData);
  const savedUser = await userRepository.save(user);
  return savedUser;
} catch (error) {
  if (error.code === '23505') { // Unique violation
    throw new Error('User already exists');
  }
  throw error;
}
```

### Best Practices

```typescript
// ✅ GOOD: Use create() before save() for type safety
const user = userRepository.create(userData);
await userRepository.save(user);

// ✅ GOOD: Save arrays for bulk operations
await userRepository.save([user1, user2, user3]);

// ✅ GOOD: Use save() when you need lifecycle hooks
// Password hashing, validation, etc. happen in hooks
await userRepository.save(user);

// ✅ GOOD: Use save() for entities with relations (cascade)
const user = userRepository.create({
  name: 'John',
  profile: { bio: 'Developer' }, // Related entity
});
await userRepository.save(user); // Saves both

// ❌ BAD: Using save() in loops
for (const data of largeDataset) {
  await userRepository.save(data); // Slow!
}

// ✅ GOOD: Batch save or use insert()
await userRepository.save(largeDataset); // Or insert()

// ✅ GOOD: Handle unique constraint violations
try {
  await userRepository.save(user);
} catch (error) {
  if (error.code === '23505') {
    // Handle duplicate
  }
}

// ✅ GOOD: Remove sensitive data after save
const user = await userRepository.save(newUser);
delete user.password;
return user;
```

**Interview Tip**: Explain that `save()` is a **smart method** that handles both insert (no ID) and update (has ID) automatically. Emphasize it **returns the saved entity**, **triggers lifecycle hooks** (@BeforeInsert, @AfterInsert, @BeforeUpdate, @AfterUpdate), and **handles relations** when cascade is enabled. Highlight when to use: single entity saves, when you need hooks (password hashing, validation), when working with relations (cascade saves), when you need the returned entity with generated ID. Mention it can save arrays efficiently but for large bulk inserts without hooks, `insert()` is faster. Discuss real-world pattern: create entity with `create()`, apply business logic, save with `save()`, hooks trigger automatically. For production: use `create()` before `save()` for type safety, batch operations with arrays, handle unique violations, remove sensitive data from response. A strong answer demonstrates understanding of save()'s intelligence and when its features (hooks, cascades) justify its use over faster alternatives.

</details>

<details>
<summary>46. What is the difference between save() and insert()?</summary>

**save()** and **insert()** both add entities to the database, but they differ significantly in behavior, performance, and use cases. **save()** is smart and feature-rich (handles updates, hooks, relations), while **insert()** is fast and simple (insert only, no hooks).

### Key Differences Overview

```typescript
/*
╔═══════════════════════╤═══════════════════════╤═══════════════════════╗
║ FEATURE               │ save()                │ insert()              ║
╠═══════════════════════╪═══════════════════════╪═══════════════════════╣
║ Operation             │ Insert OR Update      │ Insert ONLY           ║
║ Determines Action     │ Checks if ID exists   │ Always inserts        ║
║ Returns               │ Full entity with ID   │ InsertResult (raw)    ║
║ Lifecycle Hooks       │ ✅ Triggers all hooks │ ❌ No hooks           ║
║ Cascade               │ ✅ Saves relations    │ ❌ No cascade         ║
║ Performance           │ ⚠️ Slower             │ ✅ Faster             ║
║ Transactions          │ ✅ Supported          │ ✅ Supported          ║
║ Bulk Operations       │ ⚠️ OK                 │ ✅ Optimized          ║
║ Updates Existing      │ ✅ Yes (if ID exists) │ ❌ Fails/Error        ║
║ Use Case              │ Single entities       │ Bulk inserts          ║
║                       │ With relations        │ Raw data              ║
║                       │ Need hooks            │ Performance critical  ║
╚═══════════════════════╧═══════════════════════╧═══════════════════════╝
*/
```

### Basic Comparison

```typescript
import { Repository } from 'typeorm';

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

  @BeforeInsert()
  logInsert() {
    console.log('Before insert hook triggered');
  }

  @AfterInsert()
  logAfterInsert() {
    console.log('After insert hook triggered');
  }
}

const userRepository = dataSource.getRepository(User);

// ===== save() =====
const user1 = userRepository.create({
  firstName: 'John',
  lastName: 'Doe',
  email: 'john@example.com',
});

const savedUser = await userRepository.save(user1);
// Logs: "Before insert hook triggered"
// Logs: "After insert hook triggered"
console.log(savedUser.id); // Generated ID (e.g., 1)
console.log(savedUser instanceof User); // true

// ===== insert() =====
const result = await userRepository.insert({
  firstName: 'Jane',
  lastName: 'Smith',
  email: 'jane@example.com',
});
// NO hooks triggered
console.log(result.identifiers); // [{ id: 2 }]
console.log(result.raw); // Raw database result
// Note: Doesn't return User entity
```

### save() Behavior

```typescript
// save(): Insert OR Update

// INSERT (no ID)
const newUser = userRepository.create({
  firstName: 'John',
  lastName: 'Doe',
  email: 'john@example.com',
});

const saved = await userRepository.save(newUser);
console.log(saved.id); // 1 (generated)

// UPDATE (has ID)
saved.firstName = 'Johnny';
const updated = await userRepository.save(saved);
console.log(updated.id); // Still 1 (same entity updated)
console.log(updated.firstName); // 'Johnny'

// Key Point: save() is smart - detects insert vs update
```

### insert() Behavior

```typescript
// insert(): Insert ONLY

// INSERT new record
await userRepository.insert({
  firstName: 'Jane',
  lastName: 'Smith',
  email: 'jane@example.com',
});

// Trying to insert with existing ID causes error
await userRepository.insert({
  id: 1, // Existing ID
  firstName: 'Bob',
  lastName: 'Johnson',
  email: 'bob@example.com',
});
// ❌ Error: Duplicate key violation

// Key Point: insert() always tries to INSERT, never updates
```

### Return Values

```typescript
// save() returns the entity
const user = userRepository.create({
  firstName: 'John',
  lastName: 'Doe',
  email: 'john@example.com',
});

const savedUser = await userRepository.save(user);

console.log(savedUser.id); // Generated ID
console.log(savedUser.firstName); // 'John'
console.log(savedUser instanceof User); // true
// Can immediately use the returned entity

// insert() returns InsertResult
const result = await userRepository.insert({
  firstName: 'Jane',
  lastName: 'Smith',
  email: 'jane@example.com',
});

console.log(result.identifiers); // [{ id: 2 }]
console.log(result.generatedMaps); // [{ id: 2, ... }]
console.log(result.raw); // Raw DB response
// Need to manually fetch entity if needed

// Get the inserted ID
const insertedId = result.identifiers[0].id;

// Fetch entity if needed
const insertedUser = await userRepository.findOneBy({ id: insertedId });
```

### Bulk Operations

```typescript
const usersData = [
  { firstName: 'User1', lastName: 'Last1', email: 'user1@example.com' },
  { firstName: 'User2', lastName: 'Last2', email: 'user2@example.com' },
  { firstName: 'User3', lastName: 'Last3', email: 'user3@example.com' },
  // ... 1000 users
];

// ===== save() - Good for moderate bulk =====
const users = userRepository.create(usersData);
const savedUsers = await userRepository.save(users);
// - Returns full entities with IDs
// - Triggers hooks for each entity
// - Handles relations
// - Slower for large datasets

// ===== insert() - Best for large bulk =====
await userRepository.insert(usersData);
// - Single optimized query
// - No hooks triggered
// - Returns only IDs
// - Much faster for large datasets

/*
PERFORMANCE COMPARISON (1000 records):
- save(): ~5-10 seconds (with hooks, multiple queries)
- insert(): ~0.5-1 second (single optimized query)

RECOMMENDATION:
- save(): < 100 records, need hooks/relations
- insert(): > 100 records, pure data insert
*/
```

### Lifecycle Hooks

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  email: string;

  @Column()
  password: string;

  @Column({ default: true })
  isActive: boolean;

  @CreateDateColumn()
  createdAt: Date;

  @BeforeInsert()
  async hashPassword() {
    this.password = await bcrypt.hash(this.password, 10);
    console.log('Password hashed');
  }

  @BeforeInsert()
  normalizeEmail() {
    this.email = this.email.toLowerCase();
    console.log('Email normalized');
  }

  @AfterInsert()
  sendWelcomeEmail() {
    console.log(`Welcome email sent to ${this.email}`);
  }
}

// ===== save() - Triggers hooks =====
const user = userRepository.create({
  email: 'JOHN@EXAMPLE.COM',
  password: 'plaintext123',
});

await userRepository.save(user);
// Logs: "Password hashed"
// Logs: "Email normalized"
// Logs: "Welcome email sent to john@example.com"
// Password is hashed ✅
// Email is lowercase ✅

// ===== insert() - NO hooks =====
await userRepository.insert({
  email: 'JANE@EXAMPLE.COM',
  password: 'plaintext456',
});
// NO logs
// Password is NOT hashed ❌
// Email stays uppercase ❌
// No welcome email sent ❌
```

### Relations and Cascade

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToOne(() => Profile, profile => profile.user, {
    cascade: true,
  })
  @JoinColumn()
  profile: Profile;

  @OneToMany(() => Post, post => post.author, {
    cascade: true,
  })
  posts: Post[];
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

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToOne(() => User, user => user.posts)
  author: User;
}

// ===== save() - Handles relations with cascade =====
const user = userRepository.create({
  name: 'John Doe',
  profile: {
    bio: 'Software Developer',
  },
  posts: [
    { title: 'First Post' },
    { title: 'Second Post' },
  ],
});

await userRepository.save(user);
// ✅ User saved
// ✅ Profile saved (cascade)
// ✅ Posts saved (cascade)
// All in correct order with FKs

// ===== insert() - NO cascade, relations ignored =====
await userRepository.insert({
  name: 'Jane Smith',
  profile: { bio: 'Designer' }, // ❌ Ignored
  posts: [{ title: 'Post' }], // ❌ Ignored
});
// ✅ Only user inserted
// ❌ Profile NOT inserted
// ❌ Posts NOT inserted
// Must manually insert related entities
```

### Performance Comparison

```typescript
// Scenario: Insert 1000 users

// ===== Method 1: save() one by one (SLOWEST) =====
console.time('save-loop');
for (const userData of usersData) {
  const user = userRepository.create(userData);
  await userRepository.save(user);
}
console.timeEnd('save-loop');
// Time: ~10-15 seconds (1000 DB trips, hooks for each)

// ===== Method 2: save() array (MODERATE) =====
console.time('save-batch');
const users = userRepository.create(usersData);
await userRepository.save(users);
console.timeEnd('save-batch');
// Time: ~3-5 seconds (optimized, but still hooks)

// ===== Method 3: insert() bulk (FASTEST) =====
console.time('insert-bulk');
await userRepository.insert(usersData);
console.timeEnd('insert-bulk');
// Time: ~0.5-1 second (single query, no hooks)

/*
PERFORMANCE RESULTS (1000 records):
1. save() loop:   ~15 seconds  ❌
2. save() array:  ~4 seconds   ⚠️
3. insert() bulk: ~0.8 seconds ✅

USE CASES:
- save() loop: NEVER (always use array)
- save() array: When hooks/relations needed
- insert() bulk: Pure data import, best performance
*/
```

### When to Use Each

```typescript
// ===== USE save() WHEN: =====

// 1. Need lifecycle hooks
const user = userRepository.create({
  email: 'john@example.com',
  password: 'plaintext', // Will be hashed in @BeforeInsert
});
await userRepository.save(user);

// 2. Working with relations (cascade)
const user = userRepository.create({
  name: 'John',
  profile: { bio: 'Developer' },
});
await userRepository.save(user); // Saves both

// 3. Need to update if exists
const user = await userRepository.findOneBy({ email });
user.firstName = 'Updated';
await userRepository.save(user); // Updates existing

// 4. Need the saved entity immediately
const savedUser = await userRepository.save(user);
console.log(savedUser.id); // Use immediately

// 5. Single or few entities
await userRepository.save(user);

// ===== USE insert() WHEN: =====

// 1. Bulk insert (performance critical)
await userRepository.insert(thousandUsers); // Fast

// 2. Raw data without business logic
await userRepository.insert({
  firstName: 'John',
  lastName: 'Doe',
  email: 'john@example.com',
});

// 3. No hooks needed
// Password already hashed, email already normalized
await userRepository.insert(processedUsers);

// 4. No relations to save
await userRepository.insert(flatDataArray);

// 5. Migration/seeding scripts
await userRepository.insert(seedData);
```

### Real-World Example: User Registration vs Bulk Import

```typescript
// ===== User Registration (USE save()) =====
export class AuthService {
  async register(data: {
    email: string;
    password: string;
    firstName: string;
    lastName: string;
  }): Promise<User> {
    const user = this.userRepository.create({
      email: data.email,
      password: data.password, // Plain text
      firstName: data.firstName,
      lastName: data.lastName,
      profile: {
        bio: '',
      },
    });

    // Use save():
    // - Password hashed in @BeforeInsert
    // - Email normalized in @BeforeInsert
    // - Welcome email sent in @AfterInsert
    // - Profile saved via cascade
    const savedUser = await this.userRepository.save(user);

    delete savedUser.password;
    return savedUser;
  }
}

// ===== Bulk Import (USE insert()) =====
export class ImportService {
  async importUsersFromCSV(csvPath: string): Promise<void> {
    const usersData = await this.parseCSV(csvPath);

    // Pre-process data (hash passwords, normalize emails)
    const processedUsers = await Promise.all(
      usersData.map(async (userData) => ({
        firstName: userData.firstName,
        lastName: userData.lastName,
        email: userData.email.toLowerCase(),
        password: await bcrypt.hash(userData.password, 10),
        isActive: true,
      }))
    );

    // Use insert() for performance
    // Already processed, no hooks needed
    // No relations to save
    await this.userRepository.insert(processedUsers);

    console.log(`Imported ${processedUsers.length} users`);
  }
}
```

### Combining Both Methods

```typescript
export class UserService {
  // Registration: use save() (hooks, relations)
  async createUser(data: Partial<User>): Promise<User> {
    const user = this.userRepository.create(data);
    return await this.userRepository.save(user);
  }

  // Bulk seeding: use insert() (performance)
  async seedUsers(count: number): Promise<void> {
    const users = Array.from({ length: count }, (_, i) => ({
      firstName: `User${i}`,
      lastName: `Last${i}`,
      email: `user${i}@example.com`,
      password: 'hashed_password_here', // Pre-hashed
      isActive: true,
    }));

    await this.userRepository.insert(users);
  }

  // Update: use save() (smart update)
  async updateUser(id: number, data: Partial<User>): Promise<User> {
    const user = await this.userRepository.findOneBy({ id });
    Object.assign(user, data);
    return await this.userRepository.save(user);
  }
}
```

### Best Practices

```typescript
// ✅ GOOD: Use save() for single entities with logic
const user = userRepository.create({
  email: 'john@example.com',
  password: 'plaintext',
});
await userRepository.save(user); // Hooks trigger

// ✅ GOOD: Use insert() for bulk operations
await userRepository.insert(largeDataset); // Fast

// ✅ GOOD: Use save() when working with relations
const user = userRepository.create({
  name: 'John',
  profile: { bio: 'Dev' },
});
await userRepository.save(user); // Cascade

// ❌ BAD: Using save() in loop for bulk insert
for (const data of largeDataset) {
  await userRepository.save(data); // SLOW!
}

// ✅ GOOD: Batch with insert()
await userRepository.insert(largeDataset);

// ✅ GOOD: Pre-process before insert()
const processedData = data.map(item => ({
  ...item,
  email: item.email.toLowerCase(),
  password: hashSync(item.password),
}));
await userRepository.insert(processedData);

// ❌ BAD: Using insert() when hooks are needed
await userRepository.insert({
  email: 'JOHN@EXAMPLE.COM', // Won't be normalized
  password: 'plaintext', // Won't be hashed
});

// ✅ GOOD: Use save() when hooks are needed
const user = userRepository.create({
  email: 'JOHN@EXAMPLE.COM',
  password: 'plaintext',
});
await userRepository.save(user); // Hooks handle it
```

### Summary Table

```typescript
/*
CHOOSE save() FOR:
✅ Single entity insert
✅ Update existing entity
✅ Need lifecycle hooks (@BeforeInsert, @AfterInsert)
✅ Working with relations (cascade)
✅ Need full entity returned
✅ Business logic in hooks
✅ < 100 records

CHOOSE insert() FOR:
✅ Bulk inserts (> 100 records)
✅ Performance-critical operations
✅ No hooks needed
✅ Pre-processed data
✅ No relations to save
✅ Migration/seeding scripts
✅ Only need IDs returned

DECISION FLOW:
1. Need hooks? → save()
2. Need cascade? → save()
3. Bulk operation (>100)? → insert()
4. Need entity back? → save()
5. Update existing? → save()
6. Pure data insert? → insert()
*/
```

**Interview Tip**: Explain that **save()** is smart (insert or update based on ID presence), triggers lifecycle hooks, handles cascading relations, and returns the full entity - ideal for single entities with business logic. **insert()** is fast (insert only), bypasses hooks, ignores relations, returns only IDs - perfect for bulk operations. Emphasize performance difference: insert() is 5-10x faster for bulk (single optimized query vs multiple queries with hooks). Highlight when to use each: save() when you need hooks (password hashing), relations (cascade), or entity updates; insert() for bulk imports, migrations, pre-processed data. Mention real-world pattern: use save() for user registration (hooks, relations), insert() for CSV imports (performance). For production: never use save() in loops for bulk data, pre-process data before insert(), use save() for < 100 records with logic, insert() for > 100 records without hooks. A strong answer demonstrates understanding of the tradeoff between features (hooks, relations) and performance.

</details>

<details>
<summary>47. How do you use the find() method?</summary>

The **find()** method retrieves multiple entities from the database based on specified conditions. It's the most flexible query method, supporting filtering, sorting, pagination, relations loading, and field selection through **FindOptions**.

### Basic find() Usage

```typescript
import { Repository } from 'typeorm';

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

  @Column({ default: true })
  isActive: boolean;

  @CreateDateColumn()
  createdAt: Date;

  @OneToMany(() => Post, post => post.author)
  posts: Post[];
}

const userRepository = dataSource.getRepository(User);

// Find all users
const allUsers = await userRepository.find();

// Find with simple conditions
const activeUsers = await userRepository.find({
  where: { isActive: true },
});

// Find with multiple conditions
const specificUsers = await userRepository.find({
  where: {
    isActive: true,
    age: 25,
  },
});
```

### FindOptions Structure

```typescript
/*
FindOptions Interface:
{
  where?: FindConditions,        // Filter conditions
  select?: string[],             // Columns to select
  relations?: string[],          // Relations to load
  order?: { [key]: 'ASC' | 'DESC' },  // Sorting
  skip?: number,                 // Offset for pagination
  take?: number,                 // Limit for pagination
  cache?: boolean | number,      // Query caching
  withDeleted?: boolean,         // Include soft-deleted
  loadEagerRelations?: boolean,  // Load eager relations
  lock?: { mode: 'optimistic' | 'pessimistic' },
}
*/
```

### where: Filtering Conditions

```typescript
import { Equal, Not, LessThan, LessThanOrEqual, MoreThan, MoreThanOrEqual, Like, ILike, Between, In, IsNull, Not } from 'typeorm';

// Simple equality
const users = await userRepository.find({
  where: { isActive: true },
});

// Multiple conditions (AND logic)
const users = await userRepository.find({
  where: {
    isActive: true,
    age: 25,
  },
});

// OR conditions (array of conditions)
const users = await userRepository.find({
  where: [
    { firstName: 'John' },
    { firstName: 'Jane' },
  ],
});
// SQL: WHERE firstName = 'John' OR firstName = 'Jane'

// Advanced operators
const users = await userRepository.find({
  where: {
    age: MoreThan(18),
  },
});

// Greater than or equal
const users = await userRepository.find({
  where: {
    age: MoreThanOrEqual(21),
  },
});

// Less than
const users = await userRepository.find({
  where: {
    age: LessThan(65),
  },
});

// Less than or equal
const users = await userRepository.find({
  where: {
    age: LessThanOrEqual(30),
  },
});

// Between range
const users = await userRepository.find({
  where: {
    age: Between(18, 65),
  },
});

// IN clause
const users = await userRepository.find({
  where: {
    firstName: In(['John', 'Jane', 'Bob']),
  },
});

// NOT IN
const users = await userRepository.find({
  where: {
    firstName: Not(In(['Admin', 'System'])),
  },
});

// LIKE pattern matching (case-sensitive)
const users = await userRepository.find({
  where: {
    email: Like('%@gmail.com'),
  },
});

// ILIKE pattern matching (case-insensitive, PostgreSQL)
const users = await userRepository.find({
  where: {
    firstName: ILike('%john%'),
  },
});

// IS NULL
const users = await userRepository.find({
  where: {
    deletedAt: IsNull(),
  },
});

// NOT NULL
const users = await userRepository.find({
  where: {
    deletedAt: Not(IsNull()),
  },
});

// Complex combinations
const users = await userRepository.find({
  where: {
    age: Between(18, 65),
    isActive: true,
    email: Like('%@company.com'),
  },
});
```

### select: Choose Specific Fields

```typescript
// Select specific columns only
const users = await userRepository.find({
  select: ['id', 'firstName', 'lastName', 'email'],
});
// Only these fields are returned, other fields are undefined

// Exclude password field
const users = await userRepository.find({
  select: {
    id: true,
    firstName: true,
    lastName: true,
    email: true,
    password: false, // Explicitly exclude
  },
});

// Select with conditions
const users = await userRepository.find({
  where: { isActive: true },
  select: ['id', 'email'],
});
```

### relations: Load Related Entities

```typescript
// Load single relation
const users = await userRepository.find({
  relations: ['posts'],
});
// Each user has posts array loaded

// Load multiple relations
const users = await userRepository.find({
  relations: ['posts', 'profile', 'comments'],
});

// Load nested relations
const users = await userRepository.find({
  relations: ['posts', 'posts.comments', 'posts.comments.author'],
});
// Users with posts, each post with comments, each comment with author

// Load relations with conditions
const users = await userRepository.find({
  where: { isActive: true },
  relations: ['posts'],
});

// RelationIds only (performance optimization)
const users = await userRepository.find({
  relations: ['posts'],
  select: ['id', 'firstName'],
});
```

### order: Sorting Results

```typescript
// Sort by single field
const users = await userRepository.find({
  order: { lastName: 'ASC' },
});

// Sort by multiple fields
const users = await userRepository.find({
  order: {
    lastName: 'ASC',
    firstName: 'ASC',
  },
});

// Descending order
const users = await userRepository.find({
  order: { createdAt: 'DESC' },
});

// Sort with conditions
const users = await userRepository.find({
  where: { isActive: true },
  order: { createdAt: 'DESC' },
});

// Sort by relation field
const users = await userRepository.find({
  relations: ['profile'],
  order: {
    'profile.bio': 'ASC',
  },
});
```

### skip & take: Pagination

```typescript
// Get first 10 users
const users = await userRepository.find({
  take: 10,
});

// Skip first 20, get next 10 (page 3)
const users = await userRepository.find({
  skip: 20,
  take: 10,
});

// Pagination helper function
async function getPaginatedUsers(page: number, limit: number) {
  const users = await userRepository.find({
    skip: (page - 1) * limit,
    take: limit,
    order: { createdAt: 'DESC' },
  });

  return users;
}

// Usage
const page1 = await getPaginatedUsers(1, 10); // First 10
const page2 = await getPaginatedUsers(2, 10); // Next 10
const page3 = await getPaginatedUsers(3, 10); // Next 10

// Better: Use findAndCount for pagination
const [users, total] = await userRepository.findAndCount({
  skip: (page - 1) * limit,
  take: limit,
  order: { createdAt: 'DESC' },
});

const totalPages = Math.ceil(total / limit);
```

### cache: Query Caching

```typescript
// Enable caching (default cache duration)
const users = await userRepository.find({
  cache: true,
});

// Cache for specific duration (milliseconds)
const users = await userRepository.find({
  cache: 60000, // Cache for 1 minute
});

// Cache with custom key
const users = await userRepository.find({
  cache: {
    id: 'active_users_list',
    milliseconds: 60000,
  },
  where: { isActive: true },
});
```

### withDeleted: Include Soft-Deleted Entities

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @DeleteDateColumn()
  deletedAt: Date;
}

// Normal find (excludes soft-deleted)
const activeUsers = await userRepository.find();

// Include soft-deleted entities
const allUsers = await userRepository.find({
  withDeleted: true,
});

// Find only soft-deleted
const deletedUsers = await userRepository.find({
  where: {
    deletedAt: Not(IsNull()),
  },
  withDeleted: true,
});
```

### Real-World Example: User Search Service

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column({ unique: true })
  email: string;

  @Column()
  age: number;

  @Column({ default: true })
  isActive: boolean;

  @Column({ nullable: true })
  city: string;

  @Column('decimal', { precision: 10, scale: 2, default: 0 })
  balance: number;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  @DeleteDateColumn()
  deletedAt: Date;

  @OneToMany(() => Post, post => post.author)
  posts: Post[];

  @OneToOne(() => Profile, profile => profile.user)
  @JoinColumn()
  profile: Profile;
}

export class UserService {
  constructor(
    private userRepository: Repository<User>
  ) {}

  // Get all active users
  async getAllActiveUsers(): Promise<User[]> {
    return await this.userRepository.find({
      where: { isActive: true },
      order: { lastName: 'ASC', firstName: 'ASC' },
    });
  }

  // Get users with pagination
  async getUsersPaginated(
    page: number = 1,
    limit: number = 10
  ): Promise<{
    users: User[];
    total: number;
    page: number;
    totalPages: number;
  }> {
    const [users, total] = await this.userRepository.findAndCount({
      where: { isActive: true },
      order: { createdAt: 'DESC' },
      skip: (page - 1) * limit,
      take: limit,
      select: ['id', 'firstName', 'lastName', 'email', 'createdAt'],
    });

    return {
      users,
      total,
      page,
      totalPages: Math.ceil(total / limit),
    };
  }

  // Search users by name or email
  async searchUsers(query: string): Promise<User[]> {
    return await this.userRepository.find({
      where: [
        { firstName: ILike(`%${query}%`) },
        { lastName: ILike(`%${query}%`) },
        { email: ILike(`%${query}%`) },
      ],
      select: ['id', 'firstName', 'lastName', 'email'],
      order: { lastName: 'ASC' },
      take: 50,
    });
  }

  // Get users by age range
  async getUsersByAgeRange(minAge: number, maxAge: number): Promise<User[]> {
    return await this.userRepository.find({
      where: {
        age: Between(minAge, maxAge),
        isActive: true,
      },
      order: { age: 'ASC' },
    });
  }

  // Get users in specific cities
  async getUsersByCities(cities: string[]): Promise<User[]> {
    return await this.userRepository.find({
      where: {
        city: In(cities),
        isActive: true,
      },
      order: { city: 'ASC', lastName: 'ASC' },
    });
  }

  // Get users with high balance
  async getHighBalanceUsers(minBalance: number = 1000): Promise<User[]> {
    return await this.userRepository.find({
      where: {
        balance: MoreThanOrEqual(minBalance),
        isActive: true,
      },
      order: { balance: 'DESC' },
      take: 100,
    });
  }

  // Get recent users
  async getRecentUsers(days: number = 7): Promise<User[]> {
    const date = new Date();
    date.setDate(date.getDate() - days);

    return await this.userRepository.find({
      where: {
        createdAt: MoreThan(date),
      },
      order: { createdAt: 'DESC' },
    });
  }

  // Get users with posts
  async getUsersWithPosts(): Promise<User[]> {
    return await this.userRepository.find({
      relations: ['posts'],
      where: { isActive: true },
      order: { lastName: 'ASC' },
    });
  }

  // Get user details with all relations
  async getUserDetails(userId: number): Promise<User | null> {
    const users = await this.userRepository.find({
      where: { id: userId },
      relations: ['posts', 'profile', 'posts.comments'],
    });

    return users[0] || null;
  }

  // Get deleted users (admin only)
  async getDeletedUsers(): Promise<User[]> {
    return await this.userRepository.find({
      where: {
        deletedAt: Not(IsNull()),
      },
      withDeleted: true,
      order: { deletedAt: 'DESC' },
    });
  }

  // Advanced filtering
  async advancedSearch(filters: {
    minAge?: number;
    maxAge?: number;
    cities?: string[];
    minBalance?: number;
    isActive?: boolean;
    query?: string;
  }): Promise<User[]> {
    const where: any = {};

    if (filters.minAge !== undefined || filters.maxAge !== undefined) {
      where.age = Between(filters.minAge || 0, filters.maxAge || 150);
    }

    if (filters.cities && filters.cities.length > 0) {
      where.city = In(filters.cities);
    }

    if (filters.minBalance !== undefined) {
      where.balance = MoreThanOrEqual(filters.minBalance);
    }

    if (filters.isActive !== undefined) {
      where.isActive = filters.isActive;
    }

    if (filters.query) {
      return await this.userRepository.find({
        where: [
          { ...where, firstName: ILike(`%${filters.query}%`) },
          { ...where, lastName: ILike(`%${filters.query}%`) },
          { ...where, email: ILike(`%${filters.query}%`) },
        ],
        order: { lastName: 'ASC' },
      });
    }

    return await this.userRepository.find({
      where,
      order: { lastName: 'ASC' },
    });
  }
}
```

### E-commerce Product Listing Example

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

  @Column('int')
  stock: number;

  @Column({ default: true })
  isActive: boolean;

  @ManyToOne(() => Category, category => category.products)
  category: Category;

  @Column({ nullable: true })
  brand: string;

  @Column('decimal', { precision: 3, scale: 2, default: 0 })
  rating: number;

  @CreateDateColumn()
  createdAt: Date;
}

export class ProductService {
  constructor(
    private productRepository: Repository<Product>
  ) {}

  // Get available products
  async getAvailableProducts(): Promise<Product[]> {
    return await this.productRepository.find({
      where: {
        isActive: true,
        stock: MoreThan(0),
      },
      order: { name: 'ASC' },
    });
  }

  // Get products by category
  async getProductsByCategory(categoryId: number): Promise<Product[]> {
    return await this.productRepository.find({
      where: {
        category: { id: categoryId },
        isActive: true,
        stock: MoreThan(0),
      },
      relations: ['category'],
      order: { name: 'ASC' },
    });
  }

  // Get products by price range
  async getProductsByPriceRange(
    minPrice: number,
    maxPrice: number
  ): Promise<Product[]> {
    return await this.productRepository.find({
      where: {
        price: Between(minPrice, maxPrice),
        isActive: true,
        stock: MoreThan(0),
      },
      order: { price: 'ASC' },
    });
  }

  // Get products by brands
  async getProductsByBrands(brands: string[]): Promise<Product[]> {
    return await this.productRepository.find({
      where: {
        brand: In(brands),
        isActive: true,
        stock: MoreThan(0),
      },
      order: { brand: 'ASC', name: 'ASC' },
    });
  }

  // Get top-rated products
  async getTopRatedProducts(limit: number = 10): Promise<Product[]> {
    return await this.productRepository.find({
      where: {
        isActive: true,
        stock: MoreThan(0),
        rating: MoreThanOrEqual(4.0),
      },
      order: { rating: 'DESC', name: 'ASC' },
      take: limit,
    });
  }

  // Get new arrivals
  async getNewArrivals(days: number = 30): Promise<Product[]> {
    const date = new Date();
    date.setDate(date.getDate() - days);

    return await this.productRepository.find({
      where: {
        createdAt: MoreThan(date),
        isActive: true,
      },
      order: { createdAt: 'DESC' },
      take: 50,
    });
  }

  // Search products
  async searchProducts(query: string): Promise<Product[]> {
    return await this.productRepository.find({
      where: [
        { name: ILike(`%${query}%`), isActive: true },
        { description: ILike(`%${query}%`), isActive: true },
        { brand: ILike(`%${query}%`), isActive: true },
      ],
      order: { name: 'ASC' },
      take: 100,
    });
  }

  // Get products with pagination and filters
  async getProductsFiltered(filters: {
    page?: number;
    limit?: number;
    categoryId?: number;
    minPrice?: number;
    maxPrice?: number;
    brands?: string[];
    minRating?: number;
    inStock?: boolean;
  }): Promise<{ products: Product[]; total: number; page: number; totalPages: number }> {
    const page = filters.page || 1;
    const limit = filters.limit || 20;

    const where: any = {
      isActive: true,
    };

    if (filters.categoryId) {
      where.category = { id: filters.categoryId };
    }

    if (filters.minPrice !== undefined || filters.maxPrice !== undefined) {
      where.price = Between(
        filters.minPrice || 0,
        filters.maxPrice || 999999
      );
    }

    if (filters.brands && filters.brands.length > 0) {
      where.brand = In(filters.brands);
    }

    if (filters.minRating !== undefined) {
      where.rating = MoreThanOrEqual(filters.minRating);
    }

    if (filters.inStock) {
      where.stock = MoreThan(0);
    }

    const [products, total] = await this.productRepository.findAndCount({
      where,
      relations: ['category'],
      order: { name: 'ASC' },
      skip: (page - 1) * limit,
      take: limit,
    });

    return {
      products,
      total,
      page,
      totalPages: Math.ceil(total / limit),
    };
  }
}
```

### Best Practices

```typescript
// ✅ GOOD: Use findAndCount for pagination
const [users, total] = await userRepository.findAndCount({
  skip: (page - 1) * limit,
  take: limit,
});

// ✅ GOOD: Select only needed fields
const users = await userRepository.find({
  select: ['id', 'firstName', 'lastName', 'email'],
});

// ✅ GOOD: Use proper operators
const users = await userRepository.find({
  where: {
    age: Between(18, 65),
    email: Like('%@company.com'),
  },
});

// ✅ GOOD: Load relations when needed
const users = await userRepository.find({
  relations: ['posts', 'profile'],
});

// ❌ BAD: Loading unnecessary relations
const users = await userRepository.find({
  relations: ['posts', 'comments', 'likes', 'followers'], // Too many!
});

// ✅ GOOD: Limit large result sets
const users = await userRepository.find({
  take: 1000, // Limit to prevent memory issues
});

// ❌ BAD: No limit on potentially large datasets
const allUsers = await userRepository.find(); // Could be millions!

// ✅ GOOD: Use cache for frequently accessed data
const users = await userRepository.find({
  cache: 60000, // 1 minute
});

// ✅ GOOD: Combine conditions properly
const users = await userRepository.find({
  where: {
    isActive: true,
    age: MoreThan(18),
    city: In(['NYC', 'LA', 'Chicago']),
  },
  order: { lastName: 'ASC' },
  take: 100,
});
```

**Interview Tip**: Explain that `find()` is the **most flexible query method** for retrieving multiple entities, accepting **FindOptions** object with properties: `where` (filter conditions with operators like MoreThan, Between, Like), `select` (specific fields), `relations` (load related entities), `order` (sorting), `skip`/`take` (pagination), and options like `cache`, `withDeleted`. Emphasize operator usage: use Between for ranges, In for multiple values, Like/ILike for pattern matching, comparison operators (MoreThan, LessThan) for numbers/dates. Highlight pagination: use `skip`/`take` for offset-limit, prefer `findAndCount()` to get total count. Mention performance: select only needed fields, limit large result sets, use relations sparingly, enable caching for frequently accessed data. For production: always limit queries on large tables, use findAndCount for pagination, select specific fields to reduce payload, combine conditions properly for optimal queries. A strong answer demonstrates understanding of FindOptions structure and appropriate operator selection for different query needs.

</details>

<details>
<summary>48. What is the difference between find() and findOne()?</summary>

**find()** and **findOne()** are both query methods, but they differ in return type, use case, and behavior. **find()** returns an **array** of entities (can be empty), while **findOne()** returns a **single entity or null**.

### Key Differences Overview

```typescript
/*
╔════════════════════════╤═════════════════════════╤════════════════════════╗
║ FEATURE                │ find()                  │ findOne()              ║
╠════════════════════════╪═════════════════════════╪════════════════════════╣
║ Returns                │ Array (Entity[])        │ Single entity or null  ║
║ Empty Result           │ [] (empty array)        │ null                   ║
║ Use Case               │ Multiple records        │ Single record          ║
║ Query Result           │ All matching records    │ First matching record  ║
║ Performance            │ Can return many         │ Stops after first      ║
║ Type                   │ User[]                  │ User | null            ║
║ Null Check Required    │ .length === 0           │ if (!user)             ║
║ Order Matters          │ Returns all ordered     │ Returns first ordered  ║
║ With Limit             │ Can use take            │ Implicit LIMIT 1       ║
╚════════════════════════╧═════════════════════════╧════════════════════════╝
*/
```

### Basic Comparison

```typescript
import { Repository } from 'typeorm';

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

const userRepository = dataSource.getRepository(User);

// ===== find() - Returns array =====
const users = await userRepository.find({
  where: { isActive: true },
});

console.log(users); // User[] - array (could be empty)
console.log(users.length); // Number of users found
console.log(Array.isArray(users)); // true

if (users.length === 0) {
  console.log('No users found');
} else {
  console.log(`Found ${users.length} users`);
  users.forEach(user => console.log(user.email));
}

// ===== findOne() - Returns single entity or null =====
const user = await userRepository.findOne({
  where: { id: 1 },
});

console.log(user); // User | null - single entity or null
console.log(Array.isArray(user)); // false

if (!user) {
  console.log('User not found');
} else {
  console.log(`Found user: ${user.email}`);
}
```

### Return Type Differences

```typescript
// find() returns Array<Entity>
const users: User[] = await userRepository.find();

// Always an array (empty if no results)
if (users.length === 0) {
  console.log('No users found');
}

// Safe to iterate
users.forEach(user => {
  console.log(user.firstName);
});

// findOne() returns Entity | null
const user: User | null = await userRepository.findOne({
  where: { id: 1 },
});

// Returns null if not found
if (user === null) {
  console.log('User not found');
}

// Must check before accessing properties
if (user) {
  console.log(user.firstName); // Safe
}

// TypeScript enforces null check
// console.log(user.firstName); // Error: Object is possibly 'null'
```

### Query Behavior

```typescript
// find() - Returns ALL matching records
const users = await userRepository.find({
  where: { isActive: true },
});
// Returns: [user1, user2, user3, ...] - ALL active users

// findOne() - Returns FIRST matching record
const user = await userRepository.findOne({
  where: { isActive: true },
});
// Returns: user1 - ONLY the first active user

// find() with limit - Returns specified number
const users = await userRepository.find({
  where: { isActive: true },
  take: 5,
});
// Returns: [user1, user2, user3, user4, user5] - First 5

// findOne() - Implicit LIMIT 1
const user = await userRepository.findOne({
  where: { isActive: true },
  order: { createdAt: 'DESC' },
});
// Returns: Most recent active user (LIMIT 1)
```

### Order Importance

```typescript
// find() - Order affects array sequence
const users = await userRepository.find({
  where: { isActive: true },
  order: { createdAt: 'DESC' },
});
// Returns: [newest, ..., oldest] - All in order

// findOne() - Order determines WHICH one is returned
const newestUser = await userRepository.findOne({
  where: { isActive: true },
  order: { createdAt: 'DESC' },
});
// Returns: Newest user only

const oldestUser = await userRepository.findOne({
  where: { isActive: true },
  order: { createdAt: 'ASC' },
});
// Returns: Oldest user only
```

### Finding by ID

```typescript
// findOne() by ID - Common pattern
const user = await userRepository.findOne({
  where: { id: 1 },
});

if (!user) {
  throw new Error('User not found');
}

// Shorthand: findOneBy()
const user = await userRepository.findOneBy({ id: 1 });

// find() by ID - Returns array with one element
const users = await userRepository.find({
  where: { id: 1 },
});

if (users.length === 0) {
  throw new Error('User not found');
}

const user = users[0]; // Extract first element

// ❌ Awkward pattern - use findOne() instead
```

### Finding by Unique Field

```typescript
// Email is unique - use findOne()
const user = await userRepository.findOne({
  where: { email: 'john@example.com' },
});

if (!user) {
  console.log('User with this email not found');
} else {
  console.log('User found:', user.firstName);
}

// ❌ Using find() for unique field (inefficient)
const users = await userRepository.find({
  where: { email: 'john@example.com' },
});

const user = users[0]; // Awkward
```

### Finding Multiple vs Single

```typescript
// Get ALL users in a city - use find()
const usersInNYC = await userRepository.find({
  where: { city: 'New York' },
});

console.log(`Found ${usersInNYC.length} users in NYC`);

// Get ONE user in a city - use findOne()
const oneUserInNYC = await userRepository.findOne({
  where: { city: 'New York' },
});

if (oneUserInNYC) {
  console.log('One NYC user:', oneUserInNYC.firstName);
}
```

### Performance Considerations

```typescript
// findOne() - More efficient for single record
// Adds LIMIT 1 to query
const user = await userRepository.findOne({
  where: { email: 'john@example.com' },
});
// SQL: SELECT ... FROM users WHERE email = 'john@example.com' LIMIT 1

// find() - Can return many records
// No automatic limit
const users = await userRepository.find({
  where: { isActive: true },
});
// SQL: SELECT ... FROM users WHERE isActive = true
// Could return millions of records!

// ✅ GOOD: Use findOne() when you only need one
const latestUser = await userRepository.findOne({
  order: { createdAt: 'DESC' },
});

// ❌ BAD: Using find() and taking first element
const users = await userRepository.find({
  order: { createdAt: 'DESC' },
});
const latestUser = users[0]; // Inefficient
```

### Real-World Examples

```typescript
export class UserService {
  constructor(
    private userRepository: Repository<User>
  ) {}

  // Get user by ID - use findOne()
  async getUserById(id: number): Promise<User | null> {
    return await this.userRepository.findOne({
      where: { id },
      relations: ['profile', 'posts'],
    });
  }

  // Get user by email - use findOne()
  async getUserByEmail(email: string): Promise<User | null> {
    return await this.userRepository.findOne({
      where: { email },
    });
  }

  // Get all active users - use find()
  async getActiveUsers(): Promise<User[]> {
    return await this.userRepository.find({
      where: { isActive: true },
      order: { lastName: 'ASC' },
    });
  }

  // Get users by city - use find()
  async getUsersByCity(city: string): Promise<User[]> {
    return await this.userRepository.find({
      where: { city },
      order: { lastName: 'ASC' },
    });
  }

  // Get latest user - use findOne()
  async getLatestUser(): Promise<User | null> {
    return await this.userRepository.findOne({
      order: { createdAt: 'DESC' },
    });
  }

  // Get oldest user - use findOne()
  async getOldestUser(): Promise<User | null> {
    return await this.userRepository.findOne({
      order: { createdAt: 'ASC' },
    });
  }

  // Check if email exists - use findOne()
  async emailExists(email: string): Promise<boolean> {
    const user = await this.userRepository.findOne({
      where: { email },
      select: ['id'], // Only need ID
    });

    return user !== null;
  }

  // Get users paginated - use find()
  async getUsersPaginated(page: number, limit: number): Promise<User[]> {
    return await this.userRepository.find({
      skip: (page - 1) * limit,
      take: limit,
      order: { createdAt: 'DESC' },
    });
  }
}
```

### Error Handling Patterns

```typescript
// findOne() - Handle null case
async function getUserById(id: number): Promise<User> {
  const user = await userRepository.findOne({ where: { id } });

  if (!user) {
    throw new Error(`User with ID ${id} not found`);
  }

  return user;
}

// Or use findOneOrFail()
async function getUserById(id: number): Promise<User> {
  return await userRepository.findOneOrFail({ where: { id } });
  // Throws EntityNotFoundError if not found
}

// find() - Handle empty array
async function getUsersByCity(city: string): Promise<User[]> {
  const users = await userRepository.find({ where: { city } });

  if (users.length === 0) {
    console.log(`No users found in ${city}`);
  }

  return users; // Return empty array (valid response)
}
```

### Null Safety

```typescript
// findOne() requires null checks
const user = await userRepository.findOne({ where: { id: 1 } });

// ❌ BAD: No null check (TypeScript error)
// console.log(user.firstName); // Error: Object is possibly 'null'

// ✅ GOOD: Null check
if (user) {
  console.log(user.firstName);
}

// ✅ GOOD: Optional chaining
console.log(user?.firstName);

// ✅ GOOD: Nullish coalescing
const name = user?.firstName ?? 'Unknown';

// find() doesn't require null checks (always returns array)
const users = await userRepository.find({ where: { isActive: true } });

// ✅ Safe: Always an array
console.log(users.length);
users.forEach(user => console.log(user.firstName));
```

### When to Use Each

```typescript
/*
USE find() WHEN:
✅ Expecting multiple results
✅ Need all matching records
✅ Building lists/collections
✅ Implementing search results
✅ Pagination
✅ Batch processing
✅ Generating reports

USE findOne() WHEN:
✅ Looking for specific record by ID
✅ Unique field lookup (email, username)
✅ Need only first/latest/oldest record
✅ Checking existence (with select)
✅ Getting single related entity
✅ Performance matters (LIMIT 1)
✅ Expecting exactly one result
*/
```

### Best Practices

```typescript
// ✅ GOOD: Use findOne() for single record
const user = await userRepository.findOne({ where: { id: 1 } });

// ❌ BAD: Using find() and extracting first element
const users = await userRepository.find({ where: { id: 1 } });
const user = users[0];

// ✅ GOOD: Use find() for multiple records
const activeUsers = await userRepository.find({
  where: { isActive: true },
});

// ✅ GOOD: Handle null from findOne()
const user = await userRepository.findOne({ where: { id: 1 } });
if (!user) {
  throw new NotFoundException('User not found');
}

// ✅ GOOD: Handle empty array from find()
const users = await userRepository.find({ where: { city: 'NYC' } });
if (users.length === 0) {
  return []; // Valid empty result
}

// ✅ GOOD: Use findOne() with order for "latest"
const latestPost = await postRepository.findOne({
  order: { createdAt: 'DESC' },
});

// ✅ GOOD: Use findOneBy() shorthand
const user = await userRepository.findOneBy({ email: 'john@example.com' });

// ✅ GOOD: Limit find() results
const users = await userRepository.find({
  take: 1000, // Prevent loading millions
});
```

**Interview Tip**: Explain that **find()** returns an **array** of entities (empty array if none found), while **findOne()** returns a **single entity or null** (first matching record). Emphasize key difference: find() gets ALL matching records, findOne() stops at first match (implicit LIMIT 1) - better for performance when you only need one. Highlight return type handling: find() always returns array (check `.length === 0`), findOne() can return null (check `if (!entity)`). Mention use cases: findOne() for lookups by ID/unique fields, finding latest/oldest record, existence checks; find() for lists, search results, pagination, multiple records. Discuss performance: findOne() is more efficient for single record (LIMIT 1), find() without limits can return millions. For production: use findOne() for single record lookups, always handle null from findOne(), limit find() queries on large tables, prefer findOneBy() shorthand for simple conditions. A strong answer demonstrates understanding of when to use each based on expected result count and performance implications.

</details>

<details>
<summary>49. How do you use findOneBy() and findOneByOrFail()?</summary>

**findOneBy()** and **findOneByOrFail()** are simplified versions of **findOne()** for common single-entity lookups. **findOneBy()** returns the entity or null, while **findOneByOrFail()** throws an error if not found.

### Basic Usage

```typescript
import { Repository } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column({ unique: true })
  email: string;

  @Column({ default: true })
  isActive: boolean;
}

const userRepository = dataSource.getRepository(User);

// ===== findOneBy() - Returns entity or null =====
const user = await userRepository.findOneBy({
  id: 1,
});

if (!user) {
  console.log('User not found');
} else {
  console.log('User found:', user.firstName);
}

// ===== findOneByOrFail() - Throws error if not found =====
try {
  const user = await userRepository.findOneByOrFail({
    id: 999, // Doesn't exist
  });
  console.log('User found:', user.firstName);
} catch (error) {
  console.error('Error: User not found'); // EntityNotFoundError
}
```

### Comparison with findOne()

```typescript
// findOne() - Full options object
const user = await userRepository.findOne({
  where: { id: 1 },
  relations: ['posts'],
  select: ['id', 'firstName', 'email'],
  order: { createdAt: 'DESC' },
});

// findOneBy() - Simple conditions only
const user = await userRepository.findOneBy({
  id: 1,
});
// Cannot specify relations, select, order, etc.

// findOneBy() is shorthand for:
const user = await userRepository.findOne({
  where: { id: 1 },
});
```

### findOneBy() - Returns null

```typescript
// Find by ID
const user = await userRepository.findOneBy({ id: 1 });

if (!user) {
  // Handle not found case
  return null; // or throw custom error
}

// Safe to use
console.log(user.firstName);

// Find by email
const user = await userRepository.findOneBy({
  email: 'john@example.com',
});

// Find by multiple conditions
const user = await userRepository.findOneBy({
  email: 'john@example.com',
  isActive: true,
});

// All conditions are AND (not OR)
// SQL: WHERE email = 'john@example.com' AND isActive = true
```

### findOneByOrFail() - Throws Error

```typescript
import { EntityNotFoundError } from 'typeorm';

// Throws EntityNotFoundError if not found
try {
  const user = await userRepository.findOneByOrFail({ id: 999 });
  console.log(user.firstName); // Won't execute if not found
} catch (error) {
  if (error instanceof EntityNotFoundError) {
    console.error('User not found');
  }
}

// No need for null check
const user = await userRepository.findOneByOrFail({ id: 1 });
console.log(user.firstName); // Safe (or exception already thrown)

// In async function (error propagates)
async function getUser(id: number): Promise<User> {
  return await userRepository.findOneByOrFail({ id });
  // Caller must handle potential EntityNotFoundError
}
```

### Error Handling Patterns

```typescript
// Pattern 1: findOneBy() with custom error
async function getUserByEmail(email: string): Promise<User> {
  const user = await userRepository.findOneBy({ email });

  if (!user) {
    throw new Error(`User with email ${email} not found`);
  }

  return user;
}

// Pattern 2: findOneByOrFail() with try-catch
async function getUserById(id: number): Promise<User | null> {
  try {
    return await userRepository.findOneByOrFail({ id });
  } catch (error) {
    if (error instanceof EntityNotFoundError) {
      return null; // Convert to null instead of throwing
    }
    throw error; // Re-throw other errors
  }
}

// Pattern 3: findOneByOrFail() let error propagate
async function getUserById(id: number): Promise<User> {
  return await userRepository.findOneByOrFail({ id });
  // Let EntityNotFoundError propagate to caller
}

// Pattern 4: NestJS exception filter
async function getUserById(id: number): Promise<User> {
  try {
    return await userRepository.findOneByOrFail({ id });
  } catch (error) {
    if (error instanceof EntityNotFoundError) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    throw error;
  }
}
```

### Use Cases

```typescript
export class UserService {
  constructor(
    private userRepository: Repository<User>
  ) {}

  // Use findOneBy() when null is acceptable
  async findUserByEmail(email: string): Promise<User | null> {
    return await this.userRepository.findOneBy({ email });
  }

  // Use findOneByOrFail() when entity must exist
  async getUserById(id: number): Promise<User> {
    return await this.userRepository.findOneByOrFail({ id });
  }

  // Use findOneBy() for optional lookup
  async checkEmailExists(email: string): Promise<boolean> {
    const user = await this.userRepository.findOneBy({ email });
    return user !== null;
  }

  // Use findOneByOrFail() for required operations
  async updateUser(id: number, data: Partial<User>): Promise<User> {
    const user = await this.userRepository.findOneByOrFail({ id });
    
    Object.assign(user, data);
    return await this.userRepository.save(user);
  }

  // Use findOneBy() with custom error handling
  async deleteUser(id: number): Promise<void> {
    const user = await this.userRepository.findOneBy({ id });

    if (!user) {
      throw new Error('Cannot delete: User not found');
    }

    await this.userRepository.remove(user);
  }
}
```

### Multiple Conditions

```typescript
// All conditions are AND
const user = await userRepository.findOneBy({
  email: 'john@example.com',
  isActive: true,
  age: 25,
});
// SQL: WHERE email = '...' AND isActive = true AND age = 25

// Find first matching (returns first if multiple match)
const user = await userRepository.findOneBy({
  isActive: true,
});
// Returns first active user found

// With unique constraint
const user = await userRepository.findOneBy({
  email: 'john@example.com', // Unique field
});
// Only one user has this email
```

### Real-World Example: User Authentication

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ unique: true })
  email: string;

  @Column()
  password: string;

  @Column({ unique: true })
  username: string;

  @Column({ default: true })
  isActive: boolean;

  @Column({ default: false })
  isEmailVerified: boolean;
}

export class AuthService {
  constructor(
    private userRepository: Repository<User>
  ) {}

  // Login - user must exist
  async login(email: string, password: string): Promise<User> {
    // Find user or throw error
    const user = await this.userRepository.findOneByOrFail({
      email,
    });

    // Verify account is active
    if (!user.isActive) {
      throw new Error('Account is deactivated');
    }

    // Verify password
    const isPasswordValid = await bcrypt.compare(password, user.password);

    if (!isPasswordValid) {
      throw new Error('Invalid credentials');
    }

    return user;
  }

  // Register - check if email exists
  async register(data: {
    email: string;
    password: string;
    username: string;
  }): Promise<User> {
    // Check if email already exists
    const existingUser = await this.userRepository.findOneBy({
      email: data.email,
    });

    if (existingUser) {
      throw new Error('Email already registered');
    }

    // Check if username already exists
    const existingUsername = await this.userRepository.findOneBy({
      username: data.username,
    });

    if (existingUsername) {
      throw new Error('Username already taken');
    }

    // Create new user
    const user = this.userRepository.create({
      email: data.email,
      password: await bcrypt.hash(data.password, 10),
      username: data.username,
    });

    return await this.userRepository.save(user);
  }

  // Get user profile - must exist
  async getUserProfile(userId: number): Promise<User> {
    return await this.userRepository.findOneByOrFail({ id: userId });
  }

  // Verify email token
  async verifyEmail(email: string): Promise<void> {
    const user = await this.userRepository.findOneByOrFail({ email });

    user.isEmailVerified = true;
    await this.userRepository.save(user);
  }

  // Check username availability
  async isUsernameAvailable(username: string): Promise<boolean> {
    const user = await this.userRepository.findOneBy({ username });
    return user === null;
  }
}
```

### NestJS Integration

```typescript
import { Injectable, NotFoundException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';

@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>
  ) {}

  // Returns User or throws NotFoundException
  async findOne(id: number): Promise<User> {
    try {
      return await this.userRepository.findOneByOrFail({ id });
    } catch (error) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
  }

  // Returns User or null
  async findByEmail(email: string): Promise<User | null> {
    return await this.userRepository.findOneBy({ email });
  }

  // Update - user must exist
  async update(id: number, updateUserDto: UpdateUserDto): Promise<User> {
    const user = await this.findOne(id); // Uses findOneByOrFail
    
    Object.assign(user, updateUserDto);
    return await this.userRepository.save(user);
  }

  // Delete - user must exist
  async remove(id: number): Promise<void> {
    const user = await this.findOne(id);
    await this.userRepository.remove(user);
  }
}
```

### Performance Considerations

```typescript
// ✅ FAST: Simple lookup by ID
const user = await userRepository.findOneBy({ id: 1 });

// ✅ FAST: Lookup by indexed unique field
const user = await userRepository.findOneBy({ email: 'john@example.com' });

// ⚠️ SLOWER: Lookup by non-indexed field
const user = await userRepository.findOneBy({ firstName: 'John' });

// ⚠️ SLOWER: Multiple non-indexed conditions
const user = await userRepository.findOneBy({
  firstName: 'John',
  city: 'New York',
});

// TIP: Ensure database indexes on frequently queried fields
@Entity()
export class User {
  @Column({ unique: true })
  @Index() // Indexed
  email: string;

  @Column()
  @Index() // Indexed for faster lookups
  username: string;

  @Column()
  firstName: string; // Not indexed
}
```

### Limitations

```typescript
// ❌ CANNOT: Load relations with findOneBy()
// const user = await userRepository.findOneBy({ 
//   id: 1,
//   relations: ['posts'], // ERROR: Not supported
// });

// ✅ USE: findOne() for relations
const user = await userRepository.findOne({
  where: { id: 1 },
  relations: ['posts'],
});

// ❌ CANNOT: Select specific fields with findOneBy()
// const user = await userRepository.findOneBy({
//   id: 1,
//   select: ['firstName', 'email'], // ERROR: Not supported
// });

// ✅ USE: findOne() for field selection
const user = await userRepository.findOne({
  where: { id: 1 },
  select: ['id', 'firstName', 'email'],
});

// ❌ CANNOT: Use operators with findOneBy()
// const user = await userRepository.findOneBy({
//   age: MoreThan(18), // ERROR: Not supported
// });

// ✅ USE: findOne() for operators
const user = await userRepository.findOne({
  where: { age: MoreThan(18) },
});

// ❌ CANNOT: Order with findOneBy()
// const user = await userRepository.findOneBy({
//   isActive: true,
//   order: { createdAt: 'DESC' }, // ERROR: Not supported
// });

// ✅ USE: findOne() for ordering
const user = await userRepository.findOne({
  where: { isActive: true },
  order: { createdAt: 'DESC' },
});
```

### When to Use Each

```typescript
/*
USE findOneBy() WHEN:
✅ Simple equality conditions only
✅ Looking up by ID or unique field
✅ Don't need relations loaded
✅ Don't need specific fields selected
✅ Null result is acceptable
✅ Cleaner, more concise code desired

USE findOneByOrFail() WHEN:
✅ Entity MUST exist
✅ Want automatic error throwing
✅ Simplify error handling
✅ In services where missing entity is error
✅ Update/delete operations (require entity)

USE findOne() WHEN:
✅ Need relations loaded
✅ Need specific field selection
✅ Need complex conditions (operators)
✅ Need ordering
✅ Need OR conditions
✅ More control over query
*/
```

### Best Practices

```typescript
// ✅ GOOD: Use findOneBy() for simple lookups
const user = await userRepository.findOneBy({ id: 1 });

// ✅ GOOD: Use findOneByOrFail() when entity must exist
const user = await userRepository.findOneByOrFail({ id: 1 });

// ✅ GOOD: Handle null from findOneBy()
const user = await userRepository.findOneBy({ email });
if (!user) {
  throw new NotFoundException('User not found');
}

// ✅ GOOD: Convert EntityNotFoundError to custom error
try {
  const user = await userRepository.findOneByOrFail({ id });
} catch (error) {
  if (error instanceof EntityNotFoundError) {
    throw new NotFoundException(`User ${id} not found`);
  }
  throw error;
}

// ✅ GOOD: Use for existence checks
const exists = (await userRepository.findOneBy({ email })) !== null;

// ❌ BAD: Using findOneBy() with unsupported options
// const user = await userRepository.findOneBy({
//   id: 1,
//   relations: ['posts'], // ERROR
// });

// ✅ GOOD: Use findOne() for complex queries
const user = await userRepository.findOne({
  where: { id: 1 },
  relations: ['posts'],
  select: ['id', 'firstName'],
});
```

**Interview Tip**: Explain that **findOneBy()** is a **simplified version of findOne()** for basic lookups with equality conditions only - returns entity or null, cleaner syntax for simple cases. **findOneByOrFail()** is same but **throws EntityNotFoundError** if not found, useful when entity must exist (updates, deletes). Emphasize key differences: findOneBy() requires null handling, findOneByOrFail() throws error automatically. Highlight limitations: both only support simple equality conditions (no operators, no relations, no select, no ordering) - use full findOne() for complex queries. Mention use cases: findOneBy() for optional lookups, existence checks; findOneByOrFail() for operations requiring entity (update, delete), simplifying error handling. For production: use findOneBy() for ID/unique field lookups, findOneByOrFail() in services where missing entity is error, convert EntityNotFoundError to framework-specific exceptions (NestJS NotFoundException), ensure database indexes on queried fields. A strong answer demonstrates understanding of when simplified methods suffice vs when full findOne() is needed.

</details>

<details>
<summary>50. What are FindOptions in TypeORM?</summary>

**FindOptions** is a configuration object that defines how TypeORM queries entities. It provides a rich set of options for filtering, selecting fields, loading relations, sorting, pagination, caching, and more - giving you fine-grained control over database queries.

### FindOptions Interface Structure

```typescript
interface FindOptions<Entity> {
  // Filtering
  where?: FindConditions<Entity> | FindConditions<Entity>[];
  
  // Field Selection
  select?: (keyof Entity)[] | FindOptionsSelect<Entity>;
  
  // Relations
  relations?: string[] | FindOptionsRelations<Entity>;
  
  // Ordering
  order?: FindOptionsOrder<Entity>;
  
  // Pagination
  skip?: number;
  take?: number;
  
  // Caching
  cache?: boolean | number | {
    id?: string;
    milliseconds?: number;
  };
  
  // Soft Delete
  withDeleted?: boolean;
  
  // Locking
  lock?: {
    mode: 'optimistic_read' | 'optimistic_write' | 'pessimistic_read' | 'pessimistic_write';
    version?: number | Date;
  };
  
  // Advanced
  loadEagerRelations?: boolean;
  loadRelationIds?: boolean | {
    relations?: string[];
    disableMixedMap?: boolean;
  };
  relationLoadStrategy?: 'join' | 'query';
  comment?: string;
}
```

### 1. where: Filtering Options

```typescript
import { Equal, Not, LessThan, LessThanOrEqual, MoreThan, MoreThanOrEqual, Like, ILike, Between, In, Any, IsNull, Not, Raw } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  age: number;

  @Column()
  city: string;

  @Column({ default: true })
  isActive: boolean;
}

const userRepository = dataSource.getRepository(User);

// Simple equality
await userRepository.find({
  where: { isActive: true },
});

// Multiple conditions (AND)
await userRepository.find({
  where: {
    isActive: true,
    age: 25,
  },
});
// SQL: WHERE isActive = true AND age = 25

// OR conditions (array of where objects)
await userRepository.find({
  where: [
    { firstName: 'John' },
    { firstName: 'Jane' },
  ],
});
// SQL: WHERE firstName = 'John' OR firstName = 'Jane'

// Comparison operators
await userRepository.find({
  where: {
    age: MoreThan(18), // age > 18
  },
});

await userRepository.find({
  where: {
    age: MoreThanOrEqual(21), // age >= 21
  },
});

await userRepository.find({
  where: {
    age: LessThan(65), // age < 65
  },
});

await userRepository.find({
  where: {
    age: LessThanOrEqual(30), // age <= 30
  },
});

// Between range
await userRepository.find({
  where: {
    age: Between(18, 65), // age BETWEEN 18 AND 65
  },
});

// IN clause
await userRepository.find({
  where: {
    city: In(['New York', 'Los Angeles', 'Chicago']),
  },
});
// SQL: WHERE city IN ('New York', 'Los Angeles', 'Chicago')

// NOT
await userRepository.find({
  where: {
    city: Not('New York'), // city != 'New York'
  },
});

// NOT IN
await userRepository.find({
  where: {
    city: Not(In(['NYC', 'LA'])),
  },
});

// LIKE pattern
await userRepository.find({
  where: {
    firstName: Like('%john%'), // LIKE '%john%' (case-sensitive)
  },
});

// ILIKE pattern (case-insensitive, PostgreSQL)
await userRepository.find({
  where: {
    firstName: ILike('%john%'), // ILIKE '%john%'
  },
});

// IS NULL
await userRepository.find({
  where: {
    deletedAt: IsNull(), // deletedAt IS NULL
  },
});

// IS NOT NULL
await userRepository.find({
  where: {
    deletedAt: Not(IsNull()), // deletedAt IS NOT NULL
  },
});

// Raw SQL
await userRepository.find({
  where: {
    age: Raw(alias => `${alias} > 18 AND ${alias} < 65`),
  },
});

// Complex combinations
await userRepository.find({
  where: {
    age: Between(18, 65),
    isActive: true,
    city: In(['NYC', 'LA']),
    firstName: ILike('%john%'),
  },
});
```

### 2. select: Field Selection

```typescript
// Array of field names
await userRepository.find({
  select: ['id', 'firstName', 'lastName', 'email'],
});
// Only these fields returned

// Object notation (TypeORM 0.3+)
await userRepository.find({
  select: {
    id: true,
    firstName: true,
    lastName: true,
    email: true,
    password: false, // Explicitly exclude
  },
});

// Select with conditions
await userRepository.find({
  where: { isActive: true },
  select: ['id', 'firstName', 'email'],
});

// Select specific fields from relations
await userRepository.find({
  select: {
    id: true,
    firstName: true,
    profile: {
      bio: true,
      avatarUrl: true,
    },
  },
  relations: ['profile'],
});
```

### 3. relations: Loading Related Entities

```typescript
// Single relation
await userRepository.find({
  relations: ['posts'],
});

// Multiple relations
await userRepository.find({
  relations: ['posts', 'profile', 'comments'],
});

// Nested relations
await userRepository.find({
  relations: ['posts', 'posts.comments', 'posts.comments.author'],
});

// Object notation (TypeORM 0.3+)
await userRepository.find({
  relations: {
    posts: true,
    profile: true,
    comments: {
      author: true,
    },
  },
});

// Selective nested relations
await userRepository.find({
  relations: {
    posts: {
      comments: true,
      tags: true,
    },
    profile: true,
  },
});
```

### 4. order: Sorting

```typescript
// Single field
await userRepository.find({
  order: { lastName: 'ASC' },
});

// Multiple fields
await userRepository.find({
  order: {
    lastName: 'ASC',
    firstName: 'ASC',
  },
});

// Descending
await userRepository.find({
  order: { createdAt: 'DESC' },
});

// Order by relation field
await userRepository.find({
  relations: ['profile'],
  order: {
    'profile.createdAt': 'DESC',
  },
});

// Complex ordering
await userRepository.find({
  order: {
    isActive: 'DESC', // Active users first
    createdAt: 'DESC', // Then by creation date
    lastName: 'ASC', // Then alphabetically
  },
});

// Nulls handling (PostgreSQL)
await userRepository.find({
  order: {
    deletedAt: {
      direction: 'ASC',
      nulls: 'NULLS FIRST',
    },
  },
});
```

### 5. skip & take: Pagination

```typescript
// First 10 records
await userRepository.find({
  take: 10,
});

// Skip 20, get next 10 (page 3)
await userRepository.find({
  skip: 20,
  take: 10,
});

// Pagination formula
function paginate(page: number, pageSize: number) {
  return {
    skip: (page - 1) * pageSize,
    take: pageSize,
  };
}

// Get page 5, 20 items per page
await userRepository.find({
  ...paginate(5, 20),
  order: { createdAt: 'DESC' },
});
```

### 6. cache: Query Caching

```typescript
// Enable caching (default duration)
await userRepository.find({
  cache: true,
});

// Cache for specific duration (milliseconds)
await userRepository.find({
  cache: 60000, // 1 minute
});

// Cache with custom ID
await userRepository.find({
  cache: {
    id: 'active_users',
    milliseconds: 300000, // 5 minutes
  },
  where: { isActive: true },
});
```

### 7. withDeleted: Soft Deletes

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @DeleteDateColumn()
  deletedAt: Date;
}

// Exclude soft-deleted (default)
await userRepository.find();

// Include soft-deleted
await userRepository.find({
  withDeleted: true,
});

// Only soft-deleted
await userRepository.find({
  where: {
    deletedAt: Not(IsNull()),
  },
  withDeleted: true,
});
```

### 8. lock: Pessimistic/Optimistic Locking

```typescript
// Pessimistic read lock
await userRepository.find({
  where: { id: 1 },
  lock: { mode: 'pessimistic_read' },
});
// SQL: SELECT ... FOR SHARE

// Pessimistic write lock
await userRepository.find({
  where: { id: 1 },
  lock: { mode: 'pessimistic_write' },
});
// SQL: SELECT ... FOR UPDATE

// Optimistic locking
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @VersionColumn()
  version: number;
}

await userRepository.find({
  where: { id: 1 },
  lock: { mode: 'optimistic_read', version: 1 },
});
```

### 9. loadEagerRelations

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Post, post => post.author, {
    eager: true, // Eager by default
  })
  posts: Post[];
}

// Disable eager loading
await userRepository.find({
  loadEagerRelations: false,
});
// Posts won't be loaded despite eager: true
```

### 10. loadRelationIds

```typescript
// Load only IDs of related entities (not full entities)
await userRepository.find({
  loadRelationIds: true,
});
// Returns: { id: 1, name: 'John', postIds: [1, 2, 3] }

// Load IDs for specific relations
await userRepository.find({
  loadRelationIds: {
    relations: ['posts', 'comments'],
  },
});
```

### 11. relationLoadStrategy

```typescript
// Join strategy (single query with JOINs)
await userRepository.find({
  relations: ['posts', 'profile'],
  relationLoadStrategy: 'join',
});
// SQL: SELECT ... FROM users LEFT JOIN posts LEFT JOIN profile

// Query strategy (separate queries)
await userRepository.find({
  relations: ['posts', 'profile'],
  relationLoadStrategy: 'query',
});
// SQL: SELECT ... FROM users
//      SELECT ... FROM posts WHERE userId IN (...)
//      SELECT ... FROM profile WHERE userId IN (...)
```

### 12. comment: SQL Comments

```typescript
// Add comment to generated SQL (debugging)
await userRepository.find({
  where: { isActive: true },
  comment: 'Fetch active users for dashboard',
});
// SQL: SELECT ... /* Fetch active users for dashboard */
```

### Real-World Example: Advanced Search

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

  @Column('int')
  stock: number;

  @Column({ default: true })
  isActive: boolean;

  @ManyToOne(() => Category, category => category.products)
  category: Category;

  @ManyToMany(() => Tag, tag => tag.products)
  @JoinTable()
  tags: Tag[];

  @Column('decimal', { precision: 3, scale: 2, default: 0 })
  rating: number;

  @CreateDateColumn()
  createdAt: Date;

  @DeleteDateColumn()
  deletedAt: Date;
}

export class ProductService {
  constructor(
    private productRepository: Repository<Product>
  ) {}

  async searchProducts(filters: {
    query?: string;
    categoryId?: number;
    minPrice?: number;
    maxPrice?: number;
    minRating?: number;
    inStock?: boolean;
    tags?: string[];
    page?: number;
    limit?: number;
    sortBy?: 'price' | 'rating' | 'name' | 'createdAt';
    sortOrder?: 'ASC' | 'DESC';
    includeDeleted?: boolean;
  }): Promise<{
    products: Product[];
    total: number;
    page: number;
    totalPages: number;
  }> {
    // Build where conditions
    const where: any = {
      isActive: true,
    };

    if (filters.categoryId) {
      where.category = { id: filters.categoryId };
    }

    if (filters.minPrice !== undefined || filters.maxPrice !== undefined) {
      where.price = Between(
        filters.minPrice ?? 0,
        filters.maxPrice ?? 999999
      );
    }

    if (filters.minRating !== undefined) {
      where.rating = MoreThanOrEqual(filters.minRating);
    }

    if (filters.inStock) {
      where.stock = MoreThan(0);
    }

    // Build order
    const sortBy = filters.sortBy || 'createdAt';
    const sortOrder = filters.sortOrder || 'DESC';
    const order = { [sortBy]: sortOrder };

    // Pagination
    const page = filters.page || 1;
    const limit = filters.limit || 20;

    // Build FindOptions
    const findOptions: FindOptions<Product> = {
      where,
      select: {
        id: true,
        name: true,
        description: true,
        price: true,
        stock: true,
        rating: true,
        createdAt: true,
      },
      relations: {
        category: true,
        tags: true,
      },
      order,
      skip: (page - 1) * limit,
      take: limit,
      cache: {
        id: `products_search_${JSON.stringify(filters)}`,
        milliseconds: 60000, // 1 minute
      },
      withDeleted: filters.includeDeleted || false,
    };

    // Execute query
    const [products, total] = await this.productRepository.findAndCount(
      findOptions
    );

    return {
      products,
      total,
      page,
      totalPages: Math.ceil(total / limit),
    };
  }
}
```

### Complete FindOptions Example

```typescript
const findOptions: FindOptions<User> = {
  // WHERE clause
  where: {
    isActive: true,
    age: Between(18, 65),
    city: In(['NYC', 'LA', 'Chicago']),
    email: Like('%@company.com'),
  },
  
  // SELECT specific fields
  select: {
    id: true,
    firstName: true,
    lastName: true,
    email: true,
    profile: {
      bio: true,
      avatarUrl: true,
    },
  },
  
  // JOIN relations
  relations: {
    profile: true,
    posts: {
      comments: true,
    },
  },
  
  // ORDER BY
  order: {
    lastName: 'ASC',
    firstName: 'ASC',
  },
  
  // LIMIT and OFFSET
  skip: 20,
  take: 10,
  
  // Caching
  cache: {
    id: 'active_users_list',
    milliseconds: 60000,
  },
  
  // Include soft-deleted
  withDeleted: false,
  
  // Locking
  lock: {
    mode: 'pessimistic_read',
  },
  
  // Relation loading strategy
  relationLoadStrategy: 'join',
  
  // SQL comment
  comment: 'Fetch active users for admin dashboard',
};

const users = await userRepository.find(findOptions);
```

### Best Practices

```typescript
// ✅ GOOD: Use type-safe FindOptions
const options: FindManyOptions<User> = {
  where: { isActive: true },
  select: ['id', 'firstName', 'email'],
};

// ✅ GOOD: Combine conditions properly
const options: FindManyOptions<User> = {
  where: {
    age: Between(18, 65),
    isActive: true,
    city: In(['NYC', 'LA']),
  },
  order: { lastName: 'ASC' },
  take: 100,
};

// ✅ GOOD: Use operators for comparisons
where: {
  age: MoreThan(18), // Not: age: >18 (invalid)
  price: Between(10, 100), // Not: price: '10-100' (invalid)
}

// ✅ GOOD: Limit large result sets
const options: FindManyOptions<User> = {
  take: 1000, // Prevent memory issues
};

// ❌ BAD: No limit on large tables
const users = await userRepository.find(); // Could be millions!

// ✅ GOOD: Select only needed fields
select: ['id', 'firstName', 'email'], // Reduce payload

// ❌ BAD: Loading everything unnecessarily
select: undefined, // Loads all fields

// ✅ GOOD: Cache frequently accessed queries
cache: {
  id: 'active_users',
  milliseconds: 60000,
}

// ✅ GOOD: Use proper pagination
skip: (page - 1) * limit,
take: limit,

// ✅ GOOD: Load relations selectively
relations: ['profile'], // Only what's needed

// ❌ BAD: Loading too many relations
relations: ['posts', 'comments', 'likes', 'followers', 'following'], // Slow!
```

**Interview Tip**: Explain that **FindOptions** is a comprehensive configuration object for TypeORM queries with properties: **where** (filtering with operators like MoreThan, Between, Like, In), **select** (field selection), **relations** (load related entities), **order** (sorting), **skip/take** (pagination), **cache** (query caching), **withDeleted** (include soft-deleted), **lock** (pessimistic/optimistic locking), and advanced options like **relationLoadStrategy**, **loadRelationIds**, **comment**. Emphasize operator usage: use Between for ranges, In for multiple values, Like/ILike for patterns, comparison operators for numbers. Highlight that FindOptions provides type safety and prevents SQL injection. Mention real-world patterns: combine multiple options for complex queries, use cache for frequent reads, select specific fields to reduce payload, limit results on large tables. For production: always use operators (not raw strings), limit queries with take, cache frequently accessed data, load relations selectively, use proper pagination with skip/take. A strong answer demonstrates understanding of how FindOptions enables powerful, type-safe queries without raw SQL.

</details>

<details>
<summary>51. How do you implement pagination with find() method?</summary>

**Pagination** in TypeORM is implemented using **skip** (offset) and **take** (limit) options with the **find()** or **findAndCount()** methods. The **findAndCount()** method is preferred as it returns both results and total count in a single database query.

### Basic Pagination with skip & take

```typescript
import { Repository } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  email: string;

  @CreateDateColumn()
  createdAt: Date;
}

const userRepository = dataSource.getRepository(User);

// Page 1 (first 10 records)
const users = await userRepository.find({
  skip: 0,
  take: 10,
  order: { createdAt: 'DESC' },
});

// Page 2 (records 11-20)
const users = await userRepository.find({
  skip: 10,
  take: 10,
  order: { createdAt: 'DESC' },
});

// Page 3 (records 21-30)
const users = await userRepository.find({
  skip: 20,
  take: 10,
  order: { createdAt: 'DESC' },
});
```

### Pagination Formula

```typescript
/*
PAGINATION FORMULA:
skip = (page - 1) * pageSize
take = pageSize

Example:
- Page 1, 10 items: skip = 0, take = 10
- Page 2, 10 items: skip = 10, take = 10
- Page 3, 10 items: skip = 20, take = 10
- Page 5, 20 items: skip = 80, take = 20
*/

function calculatePagination(page: number, pageSize: number) {
  return {
    skip: (page - 1) * pageSize,
    take: pageSize,
  };
}

// Usage
const { skip, take } = calculatePagination(3, 10);
const users = await userRepository.find({
  skip,
  take,
  order: { createdAt: 'DESC' },
});
```

### Using findAndCount() for Total Count

```typescript
// findAndCount() returns [results, total]
const [users, total] = await userRepository.findAndCount({
  skip: 0,
  take: 10,
  order: { createdAt: 'DESC' },
});

console.log(`Found ${users.length} users out of ${total} total`);

// Calculate total pages
const pageSize = 10;
const totalPages = Math.ceil(total / pageSize);

console.log(`Total pages: ${totalPages}`);
```

### Complete Pagination Function

```typescript
interface PaginationParams {
  page?: number;
  limit?: number;
}

interface PaginatedResult<T> {
  data: T[];
  total: number;
  page: number;
  limit: number;
  totalPages: number;
  hasNext: boolean;
  hasPrevious: boolean;
}

async function paginate<T>(
  repository: Repository<T>,
  options: {
    page?: number;
    limit?: number;
    where?: any;
    order?: any;
    relations?: string[];
  } = {}
): Promise<PaginatedResult<T>> {
  const page = options.page || 1;
  const limit = options.limit || 10;

  // Validate inputs
  if (page < 1) {
    throw new Error('Page must be >= 1');
  }

  if (limit < 1 || limit > 100) {
    throw new Error('Limit must be between 1 and 100');
  }

  const skip = (page - 1) * limit;

  const [data, total] = await repository.findAndCount({
    where: options.where,
    order: options.order,
    relations: options.relations,
    skip,
    take: limit,
  });

  const totalPages = Math.ceil(total / limit);

  return {
    data,
    total,
    page,
    limit,
    totalPages,
    hasNext: page < totalPages,
    hasPrevious: page > 1,
  };
}

// Usage
const result = await paginate(userRepository, {
  page: 2,
  limit: 20,
  where: { isActive: true },
  order: { createdAt: 'DESC' },
});

console.log(result);
/*
{
  data: [...], // 20 users
  total: 157,
  page: 2,
  limit: 20,
  totalPages: 8,
  hasNext: true,
  hasPrevious: true
}
*/
```

### Service Layer Pagination

```typescript
export class UserService {
  constructor(
    private userRepository: Repository<User>
  ) {}

  async getUsers(
    page: number = 1,
    limit: number = 10
  ): Promise<PaginatedResult<User>> {
    // Validate inputs
    if (page < 1) page = 1;
    if (limit < 1) limit = 10;
    if (limit > 100) limit = 100; // Max limit

    const skip = (page - 1) * limit;

    const [users, total] = await this.userRepository.findAndCount({
      where: { isActive: true },
      order: { createdAt: 'DESC' },
      skip,
      take: limit,
      select: ['id', 'firstName', 'lastName', 'email', 'createdAt'],
    });

    return {
      data: users,
      total,
      page,
      limit,
      totalPages: Math.ceil(total / limit),
      hasNext: page < Math.ceil(total / limit),
      hasPrevious: page > 1,
    };
  }

  async searchUsers(
    query: string,
    page: number = 1,
    limit: number = 10
  ): Promise<PaginatedResult<User>> {
    const skip = (page - 1) * limit;

    const [users, total] = await this.userRepository.findAndCount({
      where: [
        { firstName: ILike(`%${query}%`) },
        { lastName: ILike(`%${query}%`) },
        { email: ILike(`%${query}%`) },
      ],
      order: { lastName: 'ASC' },
      skip,
      take: limit,
    });

    return {
      data: users,
      total,
      page,
      limit,
      totalPages: Math.ceil(total / limit),
      hasNext: page < Math.ceil(total / limit),
      hasPrevious: page > 1,
    };
  }
}
```

### REST API Controller Example (NestJS)

```typescript
import { Controller, Get, Query } from '@nestjs/common';

@Controller('users')
export class UserController {
  constructor(private userService: UserService) {}

  @Get()
  async getUsers(
    @Query('page') page: number = 1,
    @Query('limit') limit: number = 10
  ) {
    return await this.userService.getUsers(page, limit);
  }

  @Get('search')
  async searchUsers(
    @Query('q') query: string,
    @Query('page') page: number = 1,
    @Query('limit') limit: number = 10
  ) {
    return await this.userService.searchUsers(query, page, limit);
  }
}

// Request: GET /users?page=2&limit=20
// Response:
/*
{
  "data": [...], // 20 users
  "total": 157,
  "page": 2,
  "limit": 20,
  "totalPages": 8,
  "hasNext": true,
  "hasPrevious": true
}
*/
```

### Express.js Example

```typescript
import express from 'express';

const app = express();

app.get('/users', async (req, res) => {
  try {
    const page = parseInt(req.query.page as string) || 1;
    const limit = parseInt(req.query.limit as string) || 10;

    // Validate
    if (page < 1) {
      return res.status(400).json({ error: 'Page must be >= 1' });
    }

    if (limit < 1 || limit > 100) {
      return res.status(400).json({ error: 'Limit must be between 1 and 100' });
    }

    const skip = (page - 1) * limit;

    const [users, total] = await userRepository.findAndCount({
      where: { isActive: true },
      order: { createdAt: 'DESC' },
      skip,
      take: limit,
      select: ['id', 'firstName', 'lastName', 'email'],
    });

    const totalPages = Math.ceil(total / limit);

    res.json({
      data: users,
      pagination: {
        total,
        page,
        limit,
        totalPages,
        hasNext: page < totalPages,
        hasPrevious: page > 1,
      },
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

### Cursor-based Pagination (Alternative)

```typescript
// Cursor-based pagination (for real-time data, infinite scroll)
interface CursorPaginationParams {
  limit?: number;
  cursor?: number; // Last seen ID
}

async function paginateWithCursor(
  repository: Repository<User>,
  params: CursorPaginationParams
): Promise<{
  data: User[];
  nextCursor: number | null;
  hasMore: boolean;
}> {
  const limit = params.limit || 10;

  const queryBuilder = repository
    .createQueryBuilder('user')
    .orderBy('user.id', 'DESC')
    .take(limit + 1); // Fetch one extra to check if there's more

  if (params.cursor) {
    queryBuilder.where('user.id < :cursor', { cursor: params.cursor });
  }

  const users = await queryBuilder.getMany();

  const hasMore = users.length > limit;
  const data = hasMore ? users.slice(0, limit) : users;
  const nextCursor = hasMore && data.length > 0 
    ? data[data.length - 1].id 
    : null;

  return {
    data,
    nextCursor,
    hasMore,
  };
}

// Usage
const result = await paginateWithCursor(userRepository, {
  limit: 20,
  cursor: 100, // Last seen ID
});

// Request: GET /users?limit=20&cursor=100
// Response:
/*
{
  "data": [...], // Next 20 users after ID 100
  "nextCursor": 80,
  "hasMore": true
}
*/
```

### Pagination with Filters

```typescript
interface UserFilterParams {
  page?: number;
  limit?: number;
  isActive?: boolean;
  minAge?: number;
  maxAge?: number;
  city?: string;
  sortBy?: 'firstName' | 'lastName' | 'createdAt';
  sortOrder?: 'ASC' | 'DESC';
}

async function getPaginatedUsers(
  params: UserFilterParams
): Promise<PaginatedResult<User>> {
  const page = params.page || 1;
  const limit = Math.min(params.limit || 10, 100);
  const skip = (page - 1) * limit;

  // Build where conditions
  const where: any = {};

  if (params.isActive !== undefined) {
    where.isActive = params.isActive;
  }

  if (params.minAge || params.maxAge) {
    where.age = Between(params.minAge || 0, params.maxAge || 150);
  }

  if (params.city) {
    where.city = params.city;
  }

  // Build order
  const sortBy = params.sortBy || 'createdAt';
  const sortOrder = params.sortOrder || 'DESC';
  const order = { [sortBy]: sortOrder };

  const [users, total] = await userRepository.findAndCount({
    where,
    order,
    skip,
    take: limit,
  });

  return {
    data: users,
    total,
    page,
    limit,
    totalPages: Math.ceil(total / limit),
    hasNext: page < Math.ceil(total / limit),
    hasPrevious: page > 1,
  };
}

// Usage
const result = await getPaginatedUsers({
  page: 2,
  limit: 20,
  isActive: true,
  minAge: 18,
  maxAge: 65,
  city: 'New York',
  sortBy: 'lastName',
  sortOrder: 'ASC',
});
```

### Performance Optimization

```typescript
// ✅ GOOD: Use indexes on sort columns
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  @Index() // Index for sorting
  createdAt: Date;

  @Column()
  @Index() // Index for sorting
  lastName: string;
}

// ✅ GOOD: Select only needed fields
const [users, total] = await userRepository.findAndCount({
  select: ['id', 'firstName', 'lastName', 'email'], // Reduce payload
  skip,
  take: limit,
});

// ✅ GOOD: Cache total count for large datasets
let cachedTotal = await cache.get('users:total');

if (!cachedTotal) {
  cachedTotal = await userRepository.count();
  await cache.set('users:total', cachedTotal, 300); // Cache 5 minutes
}

const users = await userRepository.find({
  skip,
  take: limit,
});

// Return with cached total
return {
  data: users,
  total: cachedTotal,
  // ...
};

// ✅ GOOD: Use cursor pagination for real-time data
// Avoid skip/take for very large offsets (slow)

// ❌ BAD: Large offset (slow on big datasets)
const users = await userRepository.find({
  skip: 1000000, // Very slow!
  take: 10,
});

// ✅ GOOD: Cursor-based (faster)
const users = await userRepository
  .createQueryBuilder('user')
  .where('user.id < :cursor', { cursor: lastSeenId })
  .orderBy('user.id', 'DESC')
  .take(10)
  .getMany();
```

### Reusable Pagination Class

```typescript
export class Paginator<T> {
  constructor(private repository: Repository<T>) {}

  async paginate(
    options: {
      page?: number;
      limit?: number;
      where?: any;
      order?: any;
      relations?: string[];
      select?: any;
    } = {}
  ): Promise<PaginatedResult<T>> {
    const page = Math.max(1, options.page || 1);
    const limit = Math.min(Math.max(1, options.limit || 10), 100);
    const skip = (page - 1) * limit;

    const [data, total] = await this.repository.findAndCount({
      where: options.where,
      order: options.order || { id: 'DESC' },
      relations: options.relations,
      select: options.select,
      skip,
      take: limit,
    });

    const totalPages = Math.ceil(total / limit);

    return {
      data,
      total,
      page,
      limit,
      totalPages,
      hasNext: page < totalPages,
      hasPrevious: page > 1,
    };
  }
}

// Usage
const userPaginator = new Paginator(userRepository);

const result = await userPaginator.paginate({
  page: 2,
  limit: 20,
  where: { isActive: true },
  order: { createdAt: 'DESC' },
});
```

### Best Practices

```typescript
// ✅ GOOD: Validate page and limit
const page = Math.max(1, requestedPage);
const limit = Math.min(Math.max(1, requestedLimit), 100);

// ✅ GOOD: Set maximum limit
const MAX_LIMIT = 100;
const limit = Math.min(requestedLimit, MAX_LIMIT);

// ✅ GOOD: Use findAndCount for total
const [data, total] = await repository.findAndCount({
  skip,
  take: limit,
});

// ❌ BAD: Separate count query (2 queries instead of 1)
const data = await repository.find({ skip, take: limit });
const total = await repository.count(); // Extra query

// ✅ GOOD: Index sort columns
@Index()
@Column()
createdAt: Date;

// ✅ GOOD: Cache total for frequently accessed pages
cache.set('users:total', total, 300);

// ✅ GOOD: Use cursor pagination for large datasets
where: { id: LessThan(cursor) }

// ✅ GOOD: Return pagination metadata
return {
  data,
  pagination: {
    total,
    page,
    limit,
    totalPages,
    hasNext,
    hasPrevious,
  },
};
```

**Interview Tip**: Explain that pagination in TypeORM uses **skip** (offset) and **take** (limit) with formula: `skip = (page - 1) * pageSize`. Emphasize **findAndCount()** is preferred over separate find/count calls - returns `[results, total]` in single query. Highlight complete pagination response includes: data, total count, current page, page size, total pages, hasNext/hasPrevious flags. Mention **cursor-based pagination** as alternative for real-time data/infinite scroll - uses ID-based cursors instead of offsets, better performance for large datasets. Discuss validation: enforce max limit (e.g., 100), validate page >= 1, handle edge cases. For production: index sort columns, select only needed fields, cache total counts for large datasets, use cursor pagination for very large tables (skip becomes slow with large offsets), return comprehensive metadata for client-side pagination UI. A strong answer demonstrates understanding of offset vs cursor pagination and when to use each based on data size and use case.

</details>

<details>
<summary>52. What is the update() method and how does it differ from save()?</summary>

The **update()** method directly updates database records without loading entities first, while **save()** loads the entity, modifies it, and saves it back. **update()** is faster for simple updates but doesn't trigger hooks or handle relations, whereas **save()** is feature-rich but slower.

### Key Differences Overview

```typescript
/*
╔═══════════════════════╤═══════════════════════╤═══════════════════════╗
║ FEATURE               │ update()              │ save()                ║
╠═══════════════════════╪═══════════════════════╪═══════════════════════╣
║ Loads Entity First    │ ❌ No                 │ ✅ Yes                ║
║ Returns               │ UpdateResult (raw)    │ Updated entity        ║
║ Lifecycle Hooks       │ ❌ No hooks           │ ✅ Triggers hooks     ║
║ Cascades              │ ❌ No cascade         │ ✅ Handles cascade    ║
║ Relations             │ ❌ Doesn't handle     │ ✅ Handles relations  ║
║ Performance           │ ✅ Fast (direct SQL)  │ ⚠️ Slower (load+save) ║
║ Use Case              │ Simple field updates  │ Complex updates       ║
║                       │ Bulk updates          │ With relations        ║
║                       │ No hooks needed       │ Need hooks/validation ║
║ Insert Support        │ ❌ Update only        │ ✅ Insert or update   ║
║ Partial Updates       │ ✅ Yes                │ ✅ Yes                ║
╚═══════════════════════╧═══════════════════════╧═══════════════════════╝
*/
```

### Basic Comparison

```typescript
import { Repository } from 'typeorm';

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

  @UpdateDateColumn()
  updatedAt: Date;

  @BeforeUpdate()
  logUpdate() {
    console.log('Before update hook triggered');
  }
}

const userRepository = dataSource.getRepository(User);

// ===== update() - Direct SQL update =====
const result = await userRepository.update(1, {
  firstName: 'John Updated',
});

console.log(result.affected); // Number of rows affected
console.log(result.raw); // Raw database response
// NO hooks triggered
// NO entity returned

// ===== save() - Load, modify, save =====
const user = await userRepository.findOneBy({ id: 1 });
user.firstName = 'John Updated';
const savedUser = await userRepository.save(user);

console.log(savedUser.firstName); // 'John Updated'
// Hooks triggered
// Full entity returned
```

### update() Method

```typescript
// Update by ID
await userRepository.update(1, {
  firstName: 'John',
  isActive: true,
});

// Update by criteria
await userRepository.update(
  { isActive: false },
  { isActive: true }
);

// Update multiple IDs
await userRepository.update([1, 2, 3], {
  isActive: false,
});

// Returns UpdateResult
const result = await userRepository.update(1, { firstName: 'John' });

console.log(result.affected); // 1 (number of rows updated)
console.log(result.raw); // Raw database result
console.log(result.generatedMaps); // Generated values (if any)

// Update with complex conditions
await userRepository.update(
  {
    age: MoreThan(65),
    isActive: true,
  },
  {
    isActive: false,
  }
);
```

### save() Method

```typescript
// Load entity first
const user = await userRepository.findOneBy({ id: 1 });

if (!user) {
  throw new Error('User not found');
}

// Modify entity
user.firstName = 'John Updated';
user.email = 'newemail@example.com';

// Save (triggers hooks, handles relations)
const savedUser = await userRepository.save(user);

console.log(savedUser); // Full User entity with all fields
console.log(savedUser.updatedAt); // Automatically updated

// save() can also insert if no ID
const newUser = userRepository.create({
  firstName: 'Jane',
  email: 'jane@example.com',
});

await userRepository.save(newUser); // INSERT
```

### Performance Comparison

```typescript
// ===== update() - Fast (single query) =====
console.time('update');
await userRepository.update(1, { firstName: 'John' });
console.timeEnd('update');
// Time: ~5-10ms
// SQL: UPDATE users SET firstName = 'John' WHERE id = 1

// ===== save() - Slower (two queries) =====
console.time('save');
const user = await userRepository.findOneBy({ id: 1 }); // Query 1
user.firstName = 'John';
await userRepository.save(user); // Query 2
console.timeEnd('save');
// Time: ~15-30ms
// SQL: SELECT ... WHERE id = 1
//      UPDATE users SET firstName = 'John', updatedAt = NOW() WHERE id = 1

/*
PERFORMANCE RESULTS:
update(): ~5-10ms (1 query)
save(): ~15-30ms (2 queries + hooks)

UPDATE IS 2-3X FASTER
*/
```

### Lifecycle Hooks Difference

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  email: string;

  @Column()
  password: string;

  @UpdateDateColumn()
  updatedAt: Date;

  @BeforeUpdate()
  async hashPassword() {
    if (this.password && !this.password.startsWith('$2')) {
      this.password = await bcrypt.hash(this.password, 10);
      console.log('Password hashed');
    }
  }

  @AfterUpdate()
  logUpdate() {
    console.log(`User ${this.email} updated`);
  }
}

// ===== update() - NO hooks =====
await userRepository.update(1, {
  password: 'newpassword123',
});
// NO hooks triggered
// Password is NOT hashed ❌
// No log message ❌

// ===== save() - Triggers hooks =====
const user = await userRepository.findOneBy({ id: 1 });
user.password = 'newpassword123';
await userRepository.save(user);
// Logs: "Password hashed"
// Logs: "User user@example.com updated"
// Password IS hashed ✅
```

### Bulk Updates

```typescript
// update() - Efficient for bulk updates
await userRepository.update(
  { isActive: false },
  { isActive: true }
);
// Updates all inactive users in one query

// save() - Inefficient for bulk (loads all entities)
const inactiveUsers = await userRepository.find({
  where: { isActive: false },
});

inactiveUsers.forEach(user => {
  user.isActive = true;
});

await userRepository.save(inactiveUsers);
// Multiple queries, triggers hooks for each

/*
BULK UPDATE COMPARISON (1000 records):
update(): ~50-100ms (1 query)
save(): ~5-10 seconds (1000+ queries with hooks)

UPDATE IS 50-100X FASTER FOR BULK
*/
```

### Relations Handling

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

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

  @Column()
  bio: string;

  @OneToOne(() => User, user => user.profile)
  user: User;
}

// ===== update() - Doesn't handle relations =====
await userRepository.update(1, {
  name: 'John Updated',
  profile: { bio: 'New bio' }, // ❌ Ignored
});
// Only name updated, profile ignored

// ===== save() - Handles relations with cascade =====
const user = await userRepository.findOne({
  where: { id: 1 },
  relations: ['profile'],
});

user.name = 'John Updated';
user.profile.bio = 'New bio';

await userRepository.save(user);
// Both user AND profile updated ✅
```

### When to Use Each

```typescript
export class UserService {
  constructor(
    private userRepository: Repository<User>
  ) {}

  // USE update() - Simple field updates, no hooks needed
  async deactivateUser(userId: number): Promise<void> {
    await this.userRepository.update(userId, {
      isActive: false,
    });
  }

  // USE update() - Bulk updates
  async deactivateOldUsers(): Promise<void> {
    const sixMonthsAgo = new Date();
    sixMonthsAgo.setMonth(sixMonthsAgo.getMonth() - 6);

    await this.userRepository.update(
      {
        lastLoginAt: LessThan(sixMonthsAgo),
      },
      {
        isActive: false,
      }
    );
  }

  // USE save() - Password change (needs hashing hook)
  async changePassword(userId: number, newPassword: string): Promise<void> {
    const user = await this.userRepository.findOneByOrFail({ id: userId });
    
    user.password = newPassword;
    
    await this.userRepository.save(user);
    // Password hashed in @BeforeUpdate hook
  }

  // USE save() - Updating relations
  async updateUserProfile(
    userId: number,
    data: {
      firstName?: string;
      bio?: string;
    }
  ): Promise<User> {
    const user = await this.userRepository.findOne({
      where: { id: userId },
      relations: ['profile'],
    });

    if (!user) {
      throw new Error('User not found');
    }

    if (data.firstName) {
      user.firstName = data.firstName;
    }

    if (data.bio) {
      user.profile.bio = data.bio;
    }

    return await this.userRepository.save(user);
  }

  // USE update() - Simple status change
  async activateUser(userId: number): Promise<void> {
    await this.userRepository.update(userId, {
      isActive: true,
      activatedAt: new Date(),
    });
  }

  // USE save() - Complex business logic
  async promoteToAdmin(userId: number): Promise<User> {
    const user = await this.userRepository.findOneByOrFail({ id: userId });

    // Business logic
    user.role = 'admin';
    user.permissions = ['read', 'write', 'delete'];
    user.promotedAt = new Date();

    // Triggers @BeforeUpdate, @AfterUpdate hooks
    return await this.userRepository.save(user);
  }
}
```

### Return Value Differences

```typescript
// update() returns UpdateResult
const result = await userRepository.update(1, { firstName: 'John' });

console.log(result.affected); // 1
console.log(result.raw); // Raw DB response
console.log(result.generatedMaps); // []

// Need to fetch entity separately if needed
const user = await userRepository.findOneBy({ id: 1 });

// save() returns updated entity
const user = await userRepository.findOneBy({ id: 1 });
user.firstName = 'John';
const savedUser = await userRepository.save(user);

console.log(savedUser); // Full User entity
console.log(savedUser.firstName); // 'John'
console.log(savedUser.updatedAt); // Updated timestamp

// Can use immediately
sendEmail(savedUser.email);
```

### Increment/Decrement Operations

```typescript
// update() can't do increment
// ❌ WRONG: This sets value to 1, not increments by 1
await userRepository.update(1, { loginCount: 1 });

// ✅ CORRECT: Use increment() method
await userRepository.increment({ id: 1 }, 'loginCount', 1);

// Or raw SQL with update()
await userRepository
  .createQueryBuilder()
  .update(User)
  .set({ loginCount: () => 'loginCount + 1' })
  .where('id = :id', { id: 1 })
  .execute();

// save() can increment
const user = await userRepository.findOneBy({ id: 1 });
user.loginCount += 1;
await userRepository.save(user);
// But this has race condition!
```

### Transaction Differences

```typescript
await dataSource.transaction(async (manager) => {
  const userRepo = manager.getRepository(User);

  // update() in transaction
  await userRepo.update(1, { firstName: 'John' });

  // save() in transaction
  const user = await userRepo.findOneBy({ id: 1 });
  user.firstName = 'John';
  await userRepo.save(user);

  // Both support transactions
  // Both rollback on error
});
```

### Best Practices

```typescript
// ✅ GOOD: Use update() for simple updates
await userRepository.update(userId, {
  lastLoginAt: new Date(),
});

// ✅ GOOD: Use save() when hooks are needed
const user = await userRepository.findOneBy({ id: userId });
user.password = newPassword; // Will be hashed in hook
await userRepository.save(user);

// ✅ GOOD: Use update() for bulk operations
await userRepository.update(
  { isActive: false },
  { isActive: true }
);

// ❌ BAD: Using save() for bulk (slow)
const users = await userRepository.find({ where: { isActive: false } });
users.forEach(u => u.isActive = true);
await userRepository.save(users);

// ✅ GOOD: Use save() for relations
const user = await userRepository.findOne({
  where: { id: userId },
  relations: ['profile'],
});
user.profile.bio = 'Updated bio';
await userRepository.save(user);

// ❌ BAD: update() doesn't handle relations
await userRepository.update(userId, {
  profile: { bio: 'Updated bio' }, // Ignored!
});

// ✅ GOOD: Check affected rows with update()
const result = await userRepository.update(userId, { isActive: false });
if (result.affected === 0) {
  throw new Error('User not found');
}

// ✅ GOOD: Use increment() for atomic counters
await userRepository.increment({ id: userId }, 'viewCount', 1);

// ❌ BAD: Using save() for counters (race condition)
const user = await userRepository.findOneBy({ id: userId });
user.viewCount += 1;
await userRepository.save(user);
```

### Decision Flow

```typescript
/*
CHOOSE update() WHEN:
✅ Simple field updates (no relations)
✅ No hooks/validation needed
✅ Bulk updates (multiple records)
✅ Performance is critical
✅ Don't need returned entity
✅ Updating counters/timestamps

CHOOSE save() WHEN:
✅ Need lifecycle hooks (@BeforeUpdate, @AfterUpdate)
✅ Updating relations
✅ Need validation
✅ Complex business logic
✅ Need updated entity returned
✅ Inserting new records (auto-detects)

DECISION FLOWCHART:
1. Need hooks? → save()
2. Updating relations? → save()
3. Bulk update? → update()
4. Simple field update? → update()
5. Need entity back? → save()
6. Performance critical? → update()
*/
```

**Interview Tip**: Explain that **update()** executes direct SQL UPDATE without loading entities - fast (single query), but no hooks, no relations, returns UpdateResult with affected count. **save()** loads entity first, modifies, saves - slower (two queries), but triggers hooks, handles relations, returns updated entity. Emphasize performance: update() is 2-3x faster for single updates, 50-100x faster for bulk. Highlight when to use: update() for simple field changes, bulk updates, no hooks needed; save() when hooks required (password hashing, validation), updating relations with cascade, need returned entity, complex business logic. Mention update() limitations: can't handle relations, doesn't trigger hooks, can't auto-detect insert vs update. For production: use update() for simple/bulk updates, save() when hooks/relations matter, increment() for atomic counters (not update or save), check result.affected with update() to verify update happened. A strong answer demonstrates understanding of the performance vs features tradeoff.

</details>

<details>
<summary>53. How do you use the delete() and remove() methods?</summary>

The **delete()** method removes records directly by criteria without loading entities (fast, no hooks), while **remove()** requires loaded entities first (slower, triggers hooks). **delete()** is for simple deletions, **remove()** is for complex deletions with hooks or cascades.

### Key Differences Overview

```typescript
/*
╔═══════════════════════╤═══════════════════════╤═══════════════════════╗
║ FEATURE               │ delete()              │ remove()              ║
╠═══════════════════════╪═══════════════════════╪═══════════════════════╣
║ Loads Entity First    │ ❌ No                 │ ✅ Yes (required)     ║
║ Returns               │ DeleteResult (raw)    │ Removed entity        ║
║ Lifecycle Hooks       │ ❌ No hooks           │ ✅ Triggers hooks     ║
║ Cascades              │ ⚠️ DB-level only      │ ✅ Handles cascade    ║
║ Relations             │ ⚠️ Manual handling    │ ✅ Automatic cascade  ║
║ Performance           │ ✅ Fast (direct SQL)  │ ⚠️ Slower (load+del)  ║
║ Use Case              │ Simple deletions      │ Complex deletions     ║
║                       │ Bulk deletions        │ With hooks            ║
║                       │ No hooks needed       │ Need validation       ║
║ Soft Delete Support   │ ❌ No                 │ ❌ No (use softDelete)║
║ Accepts               │ ID or criteria        │ Entity or entities    ║
╚═══════════════════════╧═══════════════════════╧═══════════════════════╝
*/
```

### delete() Method

```typescript
import { Repository } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  email: string;

  @BeforeRemove()
  logRemoval() {
    console.log('Before remove hook triggered');
  }
}

const userRepository = dataSource.getRepository(User);

// ===== Delete by ID =====
const result = await userRepository.delete(1);

console.log(result.affected); // 1 (number of rows deleted)
console.log(result.raw); // Raw database response

// ===== Delete by criteria =====
await userRepository.delete({ email: 'john@example.com' });

// ===== Delete multiple IDs =====
await userRepository.delete([1, 2, 3]);

// ===== Delete with conditions =====
await userRepository.delete({
  isActive: false,
  createdAt: LessThan(new Date('2020-01-01')),
});

// ===== Returns DeleteResult =====
const result = await userRepository.delete(1);

console.log(result.affected); // Number of rows deleted (0 if not found)
console.log(result.raw); // Raw database result

if (result.affected === 0) {
  throw new Error('User not found');
}

// NO hooks triggered
// NO entity returned
```

### remove() Method

```typescript
// ===== Must load entity first =====
const user = await userRepository.findOneBy({ id: 1 });

if (!user) {
  throw new Error('User not found');
}

const removedUser = await userRepository.remove(user);

console.log(removedUser); // Entity with id removed (undefined)
console.log(removedUser.firstName); // Still accessible
// Hooks triggered
// Full entity returned

// ===== Remove multiple entities =====
const inactiveUsers = await userRepository.find({
  where: { isActive: false },
});

const removed = await userRepository.remove(inactiveUsers);
// Triggers hooks for each entity

// ===== Single entity =====
const user = await userRepository.findOneByOrFail({ id: 1 });
await userRepository.remove(user);

// ===== Array of entities =====
const users = await userRepository.findBy({ isActive: false });
await userRepository.remove(users);
```

### Performance Comparison

```typescript
// ===== delete() - Fast (single query) =====
console.time('delete');
await userRepository.delete(1);
console.timeEnd('delete');
// Time: ~5-10ms
// SQL: DELETE FROM users WHERE id = 1

// ===== remove() - Slower (two queries + hooks) =====
console.time('remove');
const user = await userRepository.findOneBy({ id: 1 }); // Query 1
await userRepository.remove(user); // Query 2
console.timeEnd('remove');
// Time: ~15-30ms
// SQL: SELECT ... WHERE id = 1
//      DELETE FROM users WHERE id = 1

/*
PERFORMANCE RESULTS:
delete(): ~5-10ms (1 query)
remove(): ~15-30ms (2+ queries with hooks)

DELETE IS 2-3X FASTER
*/
```

### Lifecycle Hooks Difference

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  email: string;

  @OneToMany(() => Post, post => post.author)
  posts: Post[];

  @BeforeRemove()
  async cleanupUserData() {
    console.log(`Cleaning up data for user: ${this.email}`);
    // Custom cleanup logic
    await this.archiveUserData();
  }

  @AfterRemove()
  sendFarewellEmail() {
    console.log(`Sending farewell email to: ${this.email}`);
  }

  private async archiveUserData() {
    // Archive logic
  }
}

// ===== delete() - NO hooks =====
await userRepository.delete(1);
// NO hooks triggered ❌
// No cleanup ❌
// No email sent ❌

// ===== remove() - Triggers hooks =====
const user = await userRepository.findOneBy({ id: 1 });
await userRepository.remove(user);
// Logs: "Cleaning up data for user: user@example.com"
// Logs: "Sending farewell email to: user@example.com"
// Hooks triggered ✅
```

### Bulk Deletions

```typescript
// ===== delete() - Efficient for bulk =====
const result = await userRepository.delete({
  isActive: false,
  createdAt: LessThan(new Date('2020-01-01')),
});

console.log(`Deleted ${result.affected} users`);
// Single query, very fast

// ===== remove() - Inefficient for bulk =====
const oldUsers = await userRepository.find({
  where: {
    isActive: false,
    createdAt: LessThan(new Date('2020-01-01')),
  },
});

await userRepository.remove(oldUsers);
// Multiple queries, triggers hooks for each

/*
BULK DELETE COMPARISON (1000 records):
delete(): ~50-100ms (1 query)
remove(): ~5-10 seconds (1000+ queries with hooks)

DELETE IS 50-100X FASTER FOR BULK
*/
```

### Cascade Deletions

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Post, post => post.author, {
    cascade: true,
    onDelete: 'CASCADE', // Database-level cascade
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
    onDelete: 'CASCADE',
  })
  author: User;
}

// ===== delete() - Uses database-level cascade =====
await userRepository.delete(1);
// SQL: DELETE FROM users WHERE id = 1
// Database deletes related posts automatically (if onDelete: CASCADE)
// No TypeORM cascade, no hooks on related entities

// ===== remove() - Uses TypeORM cascade =====
const user = await userRepository.findOne({
  where: { id: 1 },
  relations: ['posts'],
});

await userRepository.remove(user);
// TypeORM removes posts first (if cascade: true)
// Triggers hooks on both User and Post entities
// Then removes user
```

### Real-World Service Examples

```typescript
export class UserService {
  constructor(
    private userRepository: Repository<User>
  ) {}

  // USE delete() - Simple deletion, no hooks needed
  async deleteUser(userId: number): Promise<void> {
    const result = await this.userRepository.delete(userId);

    if (result.affected === 0) {
      throw new Error('User not found');
    }
  }

  // USE delete() - Bulk deletion
  async deleteInactiveUsers(): Promise<number> {
    const result = await this.userRepository.delete({
      isActive: false,
      lastLoginAt: LessThan(new Date(Date.now() - 365 * 24 * 60 * 60 * 1000)),
    });

    return result.affected;
  }

  // USE remove() - Need cleanup hooks
  async deleteUserWithCleanup(userId: number): Promise<void> {
    const user = await this.userRepository.findOneBy({ id: userId });

    if (!user) {
      throw new NotFoundException('User not found');
    }

    // Triggers @BeforeRemove, @AfterRemove hooks
    await this.userRepository.remove(user);
  }

  // USE remove() - Cascade deletions with hooks
  async deleteUserAndPosts(userId: number): Promise<void> {
    const user = await this.userRepository.findOne({
      where: { id: userId },
      relations: ['posts', 'comments'],
    });

    if (!user) {
      throw new NotFoundException('User not found');
    }

    // Removes user and cascades to posts/comments
    // Triggers hooks on all entities
    await this.userRepository.remove(user);
  }

  // USE delete() - Conditional deletion
  async deleteUsersByEmail(email: string): Promise<number> {
    const result = await this.userRepository.delete({ email });
    return result.affected;
  }

  // USE remove() - Need entity for business logic
  async deleteUserWithValidation(userId: number): Promise<void> {
    const user = await this.userRepository.findOneByOrFail({ id: userId });

    // Business logic validation
    if (user.role === 'admin') {
      throw new Error('Cannot delete admin users');
    }

    if (user.hasActiveSubscription) {
      throw new Error('Cancel subscription first');
    }

    // Triggers cleanup hooks
    await this.userRepository.remove(user);
  }
}
```

### Checking Deletion Success

```typescript
// delete() - Check affected rows
const result = await userRepository.delete(userId);

if (result.affected === 0) {
  throw new NotFoundException('User not found');
}

console.log(`Deleted ${result.affected} user(s)`);

// remove() - Entity must exist or throw
const user = await userRepository.findOneBy({ id: userId });

if (!user) {
  throw new NotFoundException('User not found');
}

await userRepository.remove(user);
console.log('User removed successfully');

// Or use findOneByOrFail
const user = await userRepository.findOneByOrFail({ id: userId });
await userRepository.remove(user);
```

### Transaction Support

```typescript
await dataSource.transaction(async (manager) => {
  const userRepo = manager.getRepository(User);
  const postRepo = manager.getRepository(Post);

  // delete() in transaction
  await userRepo.delete({ isActive: false });

  // remove() in transaction
  const users = await userRepo.find({ where: { role: 'guest' } });
  await userRepo.remove(users);

  // Both support transactions
  // Both rollback on error
});

// Example: Delete user and their posts atomically
await dataSource.transaction(async (manager) => {
  const userRepo = manager.getRepository(User);
  const postRepo = manager.getRepository(Post);

  // Delete posts first
  await postRepo.delete({ authorId: userId });

  // Then delete user
  const result = await userRepo.delete(userId);

  if (result.affected === 0) {
    throw new Error('User not found');
  }
});
```

### Error Handling

```typescript
// delete() - Check affected rows
try {
  const result = await userRepository.delete(userId);

  if (result.affected === 0) {
    throw new NotFoundException(`User with ID ${userId} not found`);
  }

  return { message: 'User deleted successfully' };
} catch (error) {
  if (error.code === '23503') {
    // Foreign key constraint violation (PostgreSQL)
    throw new BadRequestException('Cannot delete user with existing references');
  }
  throw error;
}

// remove() - Entity not found
try {
  const user = await userRepository.findOneByOrFail({ id: userId });
  await userRepository.remove(user);
  return { message: 'User removed successfully' };
} catch (error) {
  if (error.name === 'EntityNotFoundError') {
    throw new NotFoundException(`User with ID ${userId} not found`);
  }
  throw error;
}
```

### Soft Delete (Different Methods)

```typescript
// delete() and remove() are HARD deletes
// For soft deletes, use softDelete() or softRemove()

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @DeleteDateColumn()
  deletedAt: Date;
}

// Soft delete (covered in next question)
await userRepository.softDelete(1);
// Sets deletedAt timestamp, doesn't delete

// Hard delete
await userRepository.delete(1);
// Actually removes from database
```

### REST API Examples

```typescript
// NestJS Controller
@Controller('users')
export class UserController {
  constructor(private userService: UserService) {}

  // DELETE /users/:id
  @Delete(':id')
  async deleteUser(@Param('id') id: number) {
    await this.userService.deleteUser(id);
    return { message: 'User deleted successfully' };
  }

  // DELETE /users/bulk
  @Delete('bulk')
  async bulkDelete(@Body() ids: number[]) {
    const result = await this.userRepository.delete(ids);
    return {
      message: `Deleted ${result.affected} users`,
      count: result.affected,
    };
  }
}

// Express.js
app.delete('/users/:id', async (req, res) => {
  try {
    const userId = parseInt(req.params.id);
    const result = await userRepository.delete(userId);

    if (result.affected === 0) {
      return res.status(404).json({ error: 'User not found' });
    }

    res.json({ message: 'User deleted successfully' });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

### Best Practices

```typescript
// ✅ GOOD: Use delete() for simple deletions
await userRepository.delete(userId);

// ✅ GOOD: Use remove() when hooks are needed
const user = await userRepository.findOneBy({ id: userId });
await userRepository.remove(user); // Triggers cleanup hooks

// ✅ GOOD: Check affected rows with delete()
const result = await userRepository.delete(userId);
if (result.affected === 0) {
  throw new NotFoundException('User not found');
}

// ✅ GOOD: Use delete() for bulk operations
await userRepository.delete({ isActive: false });

// ❌ BAD: Using remove() for bulk (slow)
const users = await userRepository.find({ where: { isActive: false } });
await userRepository.remove(users); // Very slow!

// ✅ GOOD: Load relations for cascading with remove()
const user = await userRepository.findOne({
  where: { id: userId },
  relations: ['posts', 'comments'],
});
await userRepository.remove(user);

// ✅ GOOD: Handle foreign key constraints
try {
  await userRepository.delete(userId);
} catch (error) {
  if (error.code === '23503') {
    throw new Error('Cannot delete user with existing posts');
  }
}

// ✅ GOOD: Use transactions for complex deletions
await dataSource.transaction(async (manager) => {
  await manager.getRepository(Post).delete({ authorId: userId });
  await manager.getRepository(User).delete(userId);
});

// ❌ BAD: Not checking if entity exists with remove()
const user = await userRepository.findOneBy({ id: userId }); // Could be null
await userRepository.remove(user); // Error!

// ✅ GOOD: Check existence
const user = await userRepository.findOneByOrFail({ id: userId });
await userRepository.remove(user);

// ✅ GOOD: Consider soft delete instead of hard delete
await userRepository.softDelete(userId); // Reversible

// ❌ BAD: Hard delete user data (irreversible)
await userRepository.delete(userId); // Gone forever!
```

### Decision Flow

```typescript
/*
CHOOSE delete() WHEN:
✅ Simple deletions (no hooks needed)
✅ Bulk deletions (multiple records)
✅ Performance is critical
✅ No cascade to TypeORM entities
✅ Database-level cascades are sufficient
✅ No business logic validation needed

CHOOSE remove() WHEN:
✅ Need lifecycle hooks (@BeforeRemove, @AfterRemove)
✅ Complex cascade deletions (TypeORM-managed)
✅ Need business logic validation before deletion
✅ Need to audit/log deleted entity details
✅ Custom cleanup logic required
✅ Working with relations that need TypeORM cascade

DECISION FLOWCHART:
1. Need hooks? → remove()
2. Need cascade with hooks? → remove()
3. Bulk deletion? → delete()
4. Simple deletion? → delete()
5. Need validation logic? → remove()
6. Performance critical? → delete()

CONSIDER soft delete (softDelete/softRemove) for:
✅ User data (legal requirements)
✅ Audit trails
✅ Undo functionality
✅ Data recovery needs
*/
```

### Common Patterns

```typescript
// Pattern 1: Safe deletion with validation
async safeDelete(userId: number): Promise<void> {
  const user = await this.userRepository.findOneByOrFail({ id: userId });

  // Validation
  if (user.role === 'admin') {
    throw new Error('Cannot delete admin');
  }

  await this.userRepository.remove(user);
}

// Pattern 2: Bulk delete with count
async bulkDelete(criteria: any): Promise<number> {
  const result = await this.userRepository.delete(criteria);
  return result.affected;
}

// Pattern 3: Cascade deletion
async deleteWithRelations(userId: number): Promise<void> {
  const user = await this.userRepository.findOne({
    where: { id: userId },
    relations: ['posts', 'comments', 'profile'],
  });

  if (!user) {
    throw new NotFoundException('User not found');
  }

  await this.userRepository.remove(user);
}

// Pattern 4: Conditional bulk delete
async deleteOldInactive(): Promise<number> {
  const result = await this.userRepository.delete({
    isActive: false,
    lastLoginAt: LessThan(new Date(Date.now() - 365 * 24 * 60 * 60 * 1000)),
  });

  return result.affected;
}
```

**Interview Tip**: Explain that **delete()** executes direct SQL DELETE by ID or criteria - fast (single query), but no hooks, returns DeleteResult with affected count. **remove()** requires loaded entity first - slower (two queries), but triggers hooks, handles TypeORM cascades, returns removed entity. Emphasize performance: delete() is 2-3x faster for single deletions, 50-100x faster for bulk. Highlight when to use: delete() for simple deletions, bulk operations, no hooks needed, database-level cascades sufficient; remove() when hooks required (cleanup, logging, notifications), TypeORM cascade with relations, business logic validation, need deleted entity details. Mention both are hard deletes (permanent) - consider softDelete()/softRemove() for reversible deletions. For production: use delete() for simple/bulk deletions, check result.affected to verify deletion, use remove() when hooks/cascades matter, handle foreign key constraint errors, use transactions for complex deletions, prefer soft deletes for user data. A strong answer demonstrates understanding of the performance vs features tradeoff and when hard vs soft delete is appropriate.

</details>

<details>
<summary>54. What is softDelete() and how do you implement soft deletes?</summary>

**Soft delete** marks records as deleted without actually removing them from the database by setting a **deletedAt** timestamp. TypeORM provides **@DeleteDateColumn()** decorator and **softDelete()** / **softRemove()** methods to implement this pattern, allowing data recovery and maintaining audit trails.

### Why Soft Deletes?

```typescript
/*
BENEFITS OF SOFT DELETE:
✅ Data recovery (undo delete)
✅ Audit trail (who deleted what when)
✅ Legal compliance (GDPR, data retention)
✅ Referential integrity (no broken foreign keys)
✅ Historical analysis (include deleted data)
✅ Graceful failures (restore accidentally deleted)

USE CASES:
- User accounts (reactivation)
- E-commerce orders (history)
- Blog posts (unpublish, not delete)
- Comments (moderation)
- Documents (version control)
- Invoices (legal requirements)
*/
```

### Basic Implementation

```typescript
import { Entity, PrimaryGeneratedColumn, Column, DeleteDateColumn } from 'typeorm';

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

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  // This column enables soft delete
  @DeleteDateColumn()
  deletedAt: Date; // NULL = not deleted, timestamp = deleted
}

const userRepository = dataSource.getRepository(User);

// Create user
const user = await userRepository.save({
  firstName: 'John',
  email: 'john@example.com',
});

// Soft delete (sets deletedAt timestamp)
await userRepository.softDelete(1);

// Query automatically excludes soft-deleted
const activeUsers = await userRepository.find();
// Does NOT include soft-deleted users

// Query including soft-deleted
const allUsers = await userRepository.find({
  withDeleted: true,
});
// Includes soft-deleted users
```

### @DeleteDateColumn() Decorator

```typescript
@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column('text')
  content: string;

  // Soft delete column
  @DeleteDateColumn()
  deletedAt: Date;
  /*
  - Type: Date | null
  - NULL = not deleted
  - Timestamp = soft deleted at this time
  - Automatically excluded from queries
  - Can query with withDeleted: true
  */
}
```

### softDelete() vs softRemove()

```typescript
/*
╔═══════════════════════╤═══════════════════════╤═══════════════════════╗
║ FEATURE               │ softDelete()          │ softRemove()          ║
╠═══════════════════════╪═══════════════════════╪═══════════════════════╣
║ Loads Entity First    │ ❌ No                 │ ✅ Yes (required)     ║
║ Returns               │ UpdateResult          │ Entity with deletedAt ║
║ Lifecycle Hooks       │ ❌ No hooks           │ ✅ Triggers hooks     ║
║ Performance           │ ✅ Fast (direct SQL)  │ ⚠️ Slower (load+soft) ║
║ Accepts               │ ID or criteria        │ Entity or entities    ║
║ Use Case              │ Simple soft delete    │ With hooks/validation ║
╚═══════════════════════╧═══════════════════════╧═══════════════════════╝
*/

// softDelete() - Direct UPDATE
const result = await userRepository.softDelete(1);
console.log(result.affected); // 1
// SQL: UPDATE users SET deletedAt = NOW() WHERE id = 1

// softRemove() - Load first, then update
const user = await userRepository.findOneBy({ id: 1 });
const removed = await userRepository.softRemove(user);
console.log(removed.deletedAt); // 2024-12-22T10:30:00.000Z
// SQL: SELECT ... WHERE id = 1
//      UPDATE users SET deletedAt = NOW() WHERE id = 1
```

### softDelete() Method

```typescript
// Soft delete by ID
await userRepository.softDelete(1);

// Soft delete by criteria
await userRepository.softDelete({ email: 'john@example.com' });

// Soft delete multiple IDs
await userRepository.softDelete([1, 2, 3]);

// Soft delete with conditions
await userRepository.softDelete({
  isActive: false,
  lastLoginAt: LessThan(new Date('2020-01-01')),
});

// Returns UpdateResult
const result = await userRepository.softDelete(1);
console.log(result.affected); // Number of rows soft deleted
console.log(result.raw); // Raw database response

// Check if soft deleted
if (result.affected === 0) {
  throw new Error('User not found or already deleted');
}
```

### softRemove() Method

```typescript
// Must load entity first
const user = await userRepository.findOneBy({ id: 1 });

if (!user) {
  throw new Error('User not found');
}

const removed = await userRepository.softRemove(user);
console.log(removed.deletedAt); // Timestamp
console.log(removed.id); // Still has ID

// Soft remove multiple entities
const users = await userRepository.findBy({ isActive: false });
await userRepository.softRemove(users);

// With hooks
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  email: string;

  @DeleteDateColumn()
  deletedAt: Date;

  @BeforeRemove()
  logSoftDelete() {
    console.log(`Soft deleting user: ${this.email}`);
  }
}

// Triggers @BeforeRemove, @AfterRemove hooks
const user = await userRepository.findOneBy({ id: 1 });
await userRepository.softRemove(user);
// Logs: "Soft deleting user: john@example.com"
```

### Querying Soft-Deleted Entities

```typescript
// Default: Excludes soft-deleted
const activeUsers = await userRepository.find();
// WHERE deletedAt IS NULL

// Include soft-deleted
const allUsers = await userRepository.find({
  withDeleted: true,
});
// No WHERE deletedAt filter

// Only soft-deleted
const deletedUsers = await userRepository.find({
  where: {
    deletedAt: Not(IsNull()),
  },
  withDeleted: true,
});
// WHERE deletedAt IS NOT NULL

// Soft-deleted in last 30 days
const recentlyDeleted = await userRepository.find({
  where: {
    deletedAt: MoreThan(new Date(Date.now() - 30 * 24 * 60 * 60 * 1000)),
  },
  withDeleted: true,
});

// Count excluding soft-deleted
const activeCount = await userRepository.count();

// Count including soft-deleted
const totalCount = await userRepository.count({
  withDeleted: true,
});

// Count only soft-deleted
const deletedCount = await userRepository.count({
  where: {
    deletedAt: Not(IsNull()),
  },
  withDeleted: true,
});
```

### Restoring Soft-Deleted Entities

```typescript
// restore() - Restore by ID or criteria
await userRepository.restore(1);
// SQL: UPDATE users SET deletedAt = NULL WHERE id = 1

// Restore by criteria
await userRepository.restore({ email: 'john@example.com' });

// Restore multiple IDs
await userRepository.restore([1, 2, 3]);

// Returns UpdateResult
const result = await userRepository.restore(1);
console.log(result.affected); // 1

if (result.affected === 0) {
  throw new Error('User not found or not deleted');
}

// recover() - Restore loaded entity
const user = await userRepository.findOne({
  where: { id: 1 },
  withDeleted: true, // Must include deleted
});

if (!user) {
  throw new Error('User not found');
}

if (!user.deletedAt) {
  throw new Error('User is not deleted');
}

const restored = await userRepository.recover(user);
console.log(restored.deletedAt); // null
// Triggers @BeforeRecover, @AfterRecover hooks
```

### Complete Soft Delete Service

```typescript
export class UserService {
  constructor(
    private userRepository: Repository<User>
  ) {}

  // Soft delete user
  async softDeleteUser(userId: number): Promise<void> {
    const result = await this.userRepository.softDelete(userId);

    if (result.affected === 0) {
      throw new NotFoundException('User not found');
    }
  }

  // Soft delete with hooks
  async softDeleteWithHooks(userId: number): Promise<User> {
    const user = await this.userRepository.findOneByOrFail({ id: userId });
    return await this.userRepository.softRemove(user);
  }

  // Restore user
  async restoreUser(userId: number): Promise<void> {
    const result = await this.userRepository.restore(userId);

    if (result.affected === 0) {
      throw new NotFoundException('User not found or not deleted');
    }
  }

  // Get active users (excludes soft-deleted)
  async getActiveUsers(): Promise<User[]> {
    return await this.userRepository.find();
  }

  // Get all users (includes soft-deleted)
  async getAllUsers(): Promise<User[]> {
    return await this.userRepository.find({
      withDeleted: true,
    });
  }

  // Get only soft-deleted users
  async getDeletedUsers(): Promise<User[]> {
    return await this.userRepository.find({
      where: {
        deletedAt: Not(IsNull()),
      },
      withDeleted: true,
      order: {
        deletedAt: 'DESC',
      },
    });
  }

  // Check if user is soft-deleted
  async isUserDeleted(userId: number): Promise<boolean> {
    const user = await this.userRepository.findOne({
      where: { id: userId },
      withDeleted: true,
    });

    return user !== null && user.deletedAt !== null;
  }

  // Permanently delete (hard delete)
  async permanentlyDelete(userId: number): Promise<void> {
    // First check if it exists
    const user = await this.userRepository.findOne({
      where: { id: userId },
      withDeleted: true,
    });

    if (!user) {
      throw new NotFoundException('User not found');
    }

    // Hard delete
    await this.userRepository.remove(user);
  }

  // Auto-cleanup old soft-deleted records
  async cleanupOldDeleted(daysOld: number = 90): Promise<number> {
    const cutoffDate = new Date();
    cutoffDate.setDate(cutoffDate.getDate() - daysOld);

    // Find old soft-deleted records
    const oldDeleted = await this.userRepository.find({
      where: {
        deletedAt: LessThan(cutoffDate),
      },
      withDeleted: true,
    });

    // Permanently delete them
    await this.userRepository.remove(oldDeleted);

    return oldDeleted.length;
  }
}
```

### REST API Implementation

```typescript
// NestJS Controller
@Controller('users')
export class UserController {
  constructor(private userService: UserService) {}

  // Soft delete user
  @Delete(':id')
  async softDeleteUser(@Param('id') id: number) {
    await this.userService.softDeleteUser(id);
    return { message: 'User soft deleted successfully' };
  }

  // Restore user
  @Post(':id/restore')
  async restoreUser(@Param('id') id: number) {
    await this.userService.restoreUser(id);
    return { message: 'User restored successfully' };
  }

  // Get active users
  @Get()
  async getUsers() {
    return await this.userService.getActiveUsers();
  }

  // Get deleted users (admin only)
  @Get('deleted')
  @UseGuards(AdminGuard)
  async getDeletedUsers() {
    return await this.userService.getDeletedUsers();
  }

  // Permanently delete (admin only)
  @Delete(':id/permanent')
  @UseGuards(AdminGuard)
  async permanentlyDelete(@Param('id') id: number) {
    await this.userService.permanentlyDelete(id);
    return { message: 'User permanently deleted' };
  }
}
```

### Soft Delete with Relations

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @DeleteDateColumn()
  deletedAt: Date;

  @OneToMany(() => Post, post => post.author)
  posts: Post[];
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @DeleteDateColumn()
  deletedAt: Date;

  @ManyToOne(() => User, user => user.posts)
  author: User;
}

// Soft delete user (posts remain)
await userRepository.softDelete(1);
// Only user is soft-deleted, posts are NOT

// Manually soft delete related posts
const user = await userRepository.findOne({
  where: { id: 1 },
  relations: ['posts'],
});

await postRepository.softDelete(
  user.posts.map(p => p.id)
);

await userRepository.softDelete(user.id);

// Or use cascade (custom implementation)
@Entity()
export class User {
  @OneToMany(() => Post, post => post.author, {
    cascade: true,
  })
  posts: Post[];

  @BeforeRemove()
  async cascadeSoftDelete() {
    // Manually soft delete posts
    if (this.posts) {
      for (const post of this.posts) {
        post.deletedAt = new Date();
      }
    }
  }
}
```

### Query Builder with Soft Deletes

```typescript
// Query builder respects soft deletes
const users = await userRepository
  .createQueryBuilder('user')
  .where('user.isActive = :active', { active: true })
  .getMany();
// Automatically excludes soft-deleted

// Include soft-deleted in query builder
const allUsers = await userRepository
  .createQueryBuilder('user')
  .withDeleted() // Include soft-deleted
  .getMany();

// Only soft-deleted
const deletedUsers = await userRepository
  .createQueryBuilder('user')
  .withDeleted()
  .where('user.deletedAt IS NOT NULL')
  .getMany();

// Soft delete with query builder
await userRepository
  .createQueryBuilder()
  .softDelete()
  .where('id = :id', { id: 1 })
  .execute();

// Restore with query builder
await userRepository
  .createQueryBuilder()
  .restore()
  .where('id = :id', { id: 1 })
  .execute();
```

### Audit Trail Pattern

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  email: string;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  @DeleteDateColumn()
  deletedAt: Date;

  @Column({ nullable: true })
  deletedBy: number; // User ID who deleted

  @Column({ nullable: true })
  deleteReason: string;
}

export class UserService {
  async softDeleteWithAudit(
    userId: number,
    deletedBy: number,
    reason: string
  ): Promise<void> {
    // Load entity
    const user = await this.userRepository.findOneByOrFail({ id: userId });

    // Set audit fields
    user.deletedBy = deletedBy;
    user.deleteReason = reason;

    // Soft delete
    await this.userRepository.softRemove(user);
  }

  async getDeletedUsersWithAudit(): Promise<User[]> {
    return await this.userRepository.find({
      where: {
        deletedAt: Not(IsNull()),
      },
      withDeleted: true,
      order: {
        deletedAt: 'DESC',
      },
      select: ['id', 'email', 'deletedAt', 'deletedBy', 'deleteReason'],
    });
  }
}
```

### Best Practices

```typescript
// ✅ GOOD: Always use @DeleteDateColumn for soft deletes
@DeleteDateColumn()
deletedAt: Date;

// ✅ GOOD: Use softDelete() for simple operations
await userRepository.softDelete(userId);

// ✅ GOOD: Use softRemove() when hooks needed
const user = await userRepository.findOneBy({ id: userId });
await userRepository.softRemove(user);

// ✅ GOOD: Check affected rows
const result = await userRepository.softDelete(userId);
if (result.affected === 0) {
  throw new NotFoundException('User not found');
}

// ✅ GOOD: Use withDeleted when querying deleted entities
const user = await userRepository.findOne({
  where: { id: userId },
  withDeleted: true,
});

// ❌ BAD: Forgetting withDeleted
const user = await userRepository.findOneBy({ id: userId });
// Won't find soft-deleted users!

// ✅ GOOD: Restore soft-deleted entities
await userRepository.restore(userId);

// ✅ GOOD: Cleanup old soft-deleted records periodically
async cleanupOldDeleted() {
  const cutoffDate = new Date();
  cutoffDate.setDate(cutoffDate.getDate() - 90);

  const old = await userRepository.find({
    where: { deletedAt: LessThan(cutoffDate) },
    withDeleted: true,
  });

  await userRepository.remove(old); // Hard delete
}

// ✅ GOOD: Add audit fields
@Column({ nullable: true })
deletedBy: number;

@Column({ nullable: true })
deleteReason: string;

// ✅ GOOD: Document soft delete behavior
/**
 * Soft deletes a user (sets deletedAt timestamp).
 * User can be restored using restore() method.
 */
async softDeleteUser(userId: number): Promise<void> {
  // ...
}

// ❌ BAD: Mixing hard and soft deletes
await userRepository.delete(userId); // Hard delete
await userRepository.softDelete(userId); // Soft delete
// Be consistent!

// ✅ GOOD: Provide both soft and hard delete
async softDelete(userId: number): Promise<void> {
  await this.userRepository.softDelete(userId);
}

async hardDelete(userId: number): Promise<void> {
  await this.userRepository.delete(userId);
}
```

### Migration for Existing Tables

```typescript
import { MigrationInterface, QueryRunner, TableColumn } from 'typeorm';

export class AddSoftDelete1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.addColumn(
      'users',
      new TableColumn({
        name: 'deletedAt',
        type: 'timestamp',
        isNullable: true,
        default: null,
      })
    );

    // Add index for better query performance
    await queryRunner.query(`
      CREATE INDEX "IDX_users_deletedAt" ON "users" ("deletedAt")
    `);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropColumn('users', 'deletedAt');
  }
}
```

**Interview Tip**: Explain that **soft delete** marks records as deleted using **@DeleteDateColumn()** (sets deletedAt timestamp) instead of actually removing them - enables data recovery, audit trails, legal compliance. TypeORM provides **softDelete()** (fast, direct SQL, no hooks) and **softRemove()** (loads first, triggers hooks). Emphasize automatic query filtering: find() excludes soft-deleted by default, use **withDeleted: true** to include them. Mention **restore()** method to undelete records (sets deletedAt to null). Highlight use cases: user accounts (reactivation), e-commerce orders (history), legal requirements (GDPR retention), audit trails (who deleted what when). For production: use softDelete() for simple cases, softRemove() when hooks needed, restore() for undelete, add audit fields (deletedBy, deleteReason), periodically cleanup old soft-deleted records (hard delete after 90+ days), index deletedAt column for performance, be consistent (don't mix hard/soft deletes). A strong answer demonstrates understanding of when soft delete is appropriate vs hard delete and the tradeoffs (disk space, query complexity vs data recovery).

</details>

<details>
<summary>55. How do you restore soft-deleted entities?</summary>

TypeORM provides **restore()** and **recover()** methods to restore soft-deleted entities. **restore()** sets `deletedAt` to NULL by ID or criteria without loading entities (fast), while **recover()** requires loaded entities first (slower, triggers hooks). Both methods reverse soft deletes, making entities visible in queries again.

### Key Differences: restore() vs recover()

```typescript
/*
╔═══════════════════════╤═══════════════════════╤═══════════════════════╗
║ FEATURE               │ restore()             │ recover()             ║
╠═══════════════════════╪═══════════════════════╪═══════════════════════╣
║ Loads Entity First    │ ❌ No                 │ ✅ Yes (required)     ║
║ Returns               │ UpdateResult          │ Recovered entity      ║
║ Lifecycle Hooks       │ ❌ No hooks           │ ✅ Triggers hooks     ║
║ Performance           │ ✅ Fast (direct SQL)  │ ⚠️ Slower (load+save) ║
║ Accepts               │ ID or criteria        │ Entity or entities    ║
║ Use Case              │ Simple restore        │ With hooks/validation ║
╚═══════════════════════╧═══════════════════════╧═══════════════════════╝
*/
```

### Basic Setup

```typescript
import { Entity, PrimaryGeneratedColumn, Column, DeleteDateColumn } from 'typeorm';

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

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  @DeleteDateColumn()
  deletedAt: Date; // NULL = active, timestamp = soft deleted
}

const userRepository = dataSource.getRepository(User);

// Soft delete a user
await userRepository.softDelete(1);

// User is now soft-deleted
const user = await userRepository.findOneBy({ id: 1 });
console.log(user); // null (excluded from queries)

// Restore the user
await userRepository.restore(1);

// User is now active again
const restored = await userRepository.findOneBy({ id: 1 });
console.log(restored); // User entity (visible again)
```

### restore() Method

```typescript
// Restore by ID
await userRepository.restore(1);
// SQL: UPDATE users SET deletedAt = NULL WHERE id = 1

// Restore by criteria
await userRepository.restore({ email: 'john@example.com' });

// Restore multiple IDs
await userRepository.restore([1, 2, 3]);

// Restore with conditions
await userRepository.restore({
  deletedAt: MoreThan(new Date('2024-01-01')),
});

// Returns UpdateResult
const result = await userRepository.restore(1);
console.log(result.affected); // 1 (number of rows restored)
console.log(result.raw); // Raw database response

// Check if restore was successful
if (result.affected === 0) {
  throw new Error('User not found or not deleted');
}

// NO hooks triggered
// NO entity returned (just affected count)
```

### recover() Method

```typescript
// Must load entity first (with withDeleted: true)
const user = await userRepository.findOne({
  where: { id: 1 },
  withDeleted: true, // IMPORTANT: Include deleted entities
});

if (!user) {
  throw new Error('User not found');
}

if (!user.deletedAt) {
  throw new Error('User is not deleted');
}

// Recover the entity
const recovered = await userRepository.recover(user);

console.log(recovered.deletedAt); // null
console.log(recovered.email); // 'john@example.com'
// Hooks triggered
// Full entity returned

// Recover multiple entities
const deletedUsers = await userRepository.find({
  where: {
    deletedAt: Not(IsNull()),
  },
  withDeleted: true,
});

const recovered = await userRepository.recover(deletedUsers);
console.log(recovered.length); // Number of recovered entities
```

### Lifecycle Hooks for Recovery

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  email: string;

  @DeleteDateColumn()
  deletedAt: Date;

  @Column({ nullable: true })
  restoredAt: Date;

  @Column({ nullable: true })
  restoredBy: number;

  @BeforeRecover()
  async beforeRecover() {
    console.log(`About to recover user: ${this.email}`);
    this.restoredAt = new Date();
  }

  @AfterRecover()
  async afterRecover() {
    console.log(`User recovered: ${this.email}`);
    // Send re-activation email
    await this.sendReactivationEmail();
  }

  private async sendReactivationEmail() {
    // Email logic
  }
}

// restore() - NO hooks
await userRepository.restore(1);
// No logs, no hooks ❌

// recover() - Triggers hooks
const user = await userRepository.findOne({
  where: { id: 1 },
  withDeleted: true,
});

await userRepository.recover(user);
// Logs: "About to recover user: john@example.com"
// Logs: "User recovered: john@example.com"
// Email sent ✅
```

### Complete Restore Service

```typescript
export class UserService {
  constructor(
    private userRepository: Repository<User>
  ) {}

  // Simple restore
  async restoreUser(userId: number): Promise<void> {
    const result = await this.userRepository.restore(userId);

    if (result.affected === 0) {
      throw new NotFoundException('User not found or not deleted');
    }
  }

  // Restore with hooks
  async restoreUserWithHooks(userId: number): Promise<User> {
    const user = await this.userRepository.findOne({
      where: { id: userId },
      withDeleted: true,
    });

    if (!user) {
      throw new NotFoundException('User not found');
    }

    if (!user.deletedAt) {
      throw new BadRequestException('User is not deleted');
    }

    return await this.userRepository.recover(user);
  }

  // Restore with audit trail
  async restoreWithAudit(
    userId: number,
    restoredBy: number,
    reason: string
  ): Promise<User> {
    const user = await this.userRepository.findOne({
      where: { id: userId },
      withDeleted: true,
    });

    if (!user) {
      throw new NotFoundException('User not found');
    }

    if (!user.deletedAt) {
      throw new BadRequestException('User is not deleted');
    }

    // Set audit fields
    user.restoredAt = new Date();
    user.restoredBy = restoredBy;
    user.restoreReason = reason;

    return await this.userRepository.recover(user);
  }

  // Bulk restore
  async restoreMultiple(userIds: number[]): Promise<number> {
    const result = await this.userRepository.restore(userIds);
    return result.affected;
  }

  // Restore by criteria
  async restoreRecentlyDeleted(days: number = 30): Promise<number> {
    const cutoffDate = new Date();
    cutoffDate.setDate(cutoffDate.getDate() - days);

    const result = await this.userRepository.restore({
      deletedAt: MoreThan(cutoffDate),
    });

    return result.affected;
  }

  // Check if user can be restored
  async canRestore(userId: number): Promise<boolean> {
    const user = await this.userRepository.findOne({
      where: { id: userId },
      withDeleted: true,
    });

    return user !== null && user.deletedAt !== null;
  }

  // Get restorable users (soft-deleted)
  async getRestorableUsers(): Promise<User[]> {
    return await this.userRepository.find({
      where: {
        deletedAt: Not(IsNull()),
      },
      withDeleted: true,
      order: {
        deletedAt: 'DESC',
      },
      select: ['id', 'firstName', 'lastName', 'email', 'deletedAt'],
    });
  }
}
```

### REST API Implementation

```typescript
// NestJS Controller
@Controller('users')
export class UserController {
  constructor(private userService: UserService) {}

  // Restore user
  @Post(':id/restore')
  async restoreUser(@Param('id') id: number) {
    await this.userService.restoreUser(id);
    return { message: 'User restored successfully' };
  }

  // Restore with audit (admin only)
  @Post(':id/restore-with-audit')
  @UseGuards(AdminGuard)
  async restoreWithAudit(
    @Param('id') id: number,
    @Body() body: { restoredBy: number; reason: string },
    @CurrentUser() currentUser: User
  ) {
    const user = await this.userService.restoreWithAudit(
      id,
      currentUser.id,
      body.reason
    );

    return {
      message: 'User restored successfully',
      user,
    };
  }

  // Bulk restore (admin only)
  @Post('restore/bulk')
  @UseGuards(AdminGuard)
  async bulkRestore(@Body() body: { userIds: number[] }) {
    const count = await this.userService.restoreMultiple(body.userIds);
    return {
      message: `Restored ${count} users`,
      count,
    };
  }

  // Get restorable users (admin only)
  @Get('deleted')
  @UseGuards(AdminGuard)
  async getRestorableUsers() {
    return await this.userService.getRestorableUsers();
  }
}

// Express.js
app.post('/users/:id/restore', async (req, res) => {
  try {
    const userId = parseInt(req.params.id);
    const result = await userRepository.restore(userId);

    if (result.affected === 0) {
      return res.status(404).json({ error: 'User not found or not deleted' });
    }

    res.json({ message: 'User restored successfully' });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

### Query Builder with Restore

```typescript
// Restore with query builder
await userRepository
  .createQueryBuilder()
  .restore()
  .where('id = :id', { id: 1 })
  .execute();

// Restore by criteria
await userRepository
  .createQueryBuilder()
  .restore()
  .where('email = :email', { email: 'john@example.com' })
  .execute();

// Restore multiple with conditions
await userRepository
  .createQueryBuilder()
  .restore()
  .where('deletedAt > :date', { date: new Date('2024-01-01') })
  .execute();

// Get count of restorable entities
const count = await userRepository
  .createQueryBuilder('user')
  .withDeleted()
  .where('user.deletedAt IS NOT NULL')
  .getCount();
```

### Restore with Validation

```typescript
export class UserService {
  async restoreUserWithValidation(userId: number): Promise<User> {
    // Find user (including deleted)
    const user = await this.userRepository.findOne({
      where: { id: userId },
      withDeleted: true,
    });

    if (!user) {
      throw new NotFoundException('User not found');
    }

    // Check if user is actually deleted
    if (!user.deletedAt) {
      throw new BadRequestException('User is not deleted');
    }

    // Business logic validation
    const emailExists = await this.userRepository.findOne({
      where: { email: user.email },
    });

    if (emailExists) {
      throw new ConflictException('A user with this email already exists');
    }

    // Check if deleted too long ago
    const daysSinceDeleted = Math.floor(
      (Date.now() - user.deletedAt.getTime()) / (1000 * 60 * 60 * 24)
    );

    if (daysSinceDeleted > 90) {
      throw new BadRequestException(
        'User deleted more than 90 days ago cannot be restored'
      );
    }

    // Restore with hooks
    return await this.userRepository.recover(user);
  }
}
```

### Transaction Support

```typescript
await dataSource.transaction(async (manager) => {
  const userRepo = manager.getRepository(User);
  const postRepo = manager.getRepository(Post);

  // Restore user
  await userRepo.restore(userId);

  // Restore user's posts
  await postRepo.restore({ authorId: userId });

  // Both restored atomically
});

// Example: Restore with related entities
await dataSource.transaction(async (manager) => {
  const userRepo = manager.getRepository(User);
  const profileRepo = manager.getRepository(Profile);

  // Find user
  const user = await userRepo.findOne({
    where: { id: userId },
    withDeleted: true,
  });

  if (!user || !user.deletedAt) {
    throw new Error('User not found or not deleted');
  }

  // Restore user
  await userRepo.recover(user);

  // Restore profile
  const result = await profileRepo.restore({ userId: user.id });

  if (result.affected === 0) {
    console.warn('No profile to restore');
  }
});
```

### Batch Restore Operations

```typescript
export class UserService {
  // Restore all users deleted in last N days
  async restoreRecentDeletions(days: number = 7): Promise<number> {
    const cutoffDate = new Date();
    cutoffDate.setDate(cutoffDate.getDate() - days);

    const result = await this.userRepository.restore({
      deletedAt: MoreThan(cutoffDate),
    });

    return result.affected;
  }

  // Restore by email pattern
  async restoreByEmailPattern(pattern: string): Promise<number> {
    // Find matching deleted users
    const users = await this.userRepository.find({
      where: {
        email: Like(pattern),
        deletedAt: Not(IsNull()),
      },
      withDeleted: true,
    });

    if (users.length === 0) {
      return 0;
    }

    const userIds = users.map(u => u.id);
    const result = await this.userRepository.restore(userIds);

    return result.affected;
  }

  // Restore with hooks for multiple entities
  async restoreMultipleWithHooks(userIds: number[]): Promise<User[]> {
    const users = await this.userRepository.find({
      where: {
        id: In(userIds),
        deletedAt: Not(IsNull()),
      },
      withDeleted: true,
    });

    if (users.length === 0) {
      throw new NotFoundException('No deleted users found');
    }

    // Recover each (triggers hooks)
    return await this.userRepository.recover(users);
  }
}
```

### Audit Trail for Restores

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  email: string;

  @DeleteDateColumn()
  deletedAt: Date;

  @Column({ nullable: true })
  deletedBy: number;

  @Column({ nullable: true })
  deleteReason: string;

  @Column({ nullable: true })
  restoredAt: Date;

  @Column({ nullable: true })
  restoredBy: number;

  @Column({ nullable: true })
  restoreReason: string;

  @BeforeRecover()
  logRestore() {
    console.log(`Restoring user: ${this.email}`);
  }

  @AfterRecover()
  async notifyRestore() {
    console.log(`User ${this.email} restored`);
    // Send notification
    await this.sendRestoreNotification();
  }

  private async sendRestoreNotification() {
    // Email notification logic
  }
}

export class UserService {
  async restoreWithFullAudit(
    userId: number,
    restoredBy: number,
    reason: string
  ): Promise<User> {
    const user = await this.userRepository.findOne({
      where: { id: userId },
      withDeleted: true,
    });

    if (!user) {
      throw new NotFoundException('User not found');
    }

    if (!user.deletedAt) {
      throw new BadRequestException('User is not deleted');
    }

    // Set audit fields before restore
    user.restoredAt = new Date();
    user.restoredBy = restoredBy;
    user.restoreReason = reason;

    // Recover with hooks
    const restored = await this.userRepository.recover(user);

    // Log audit event
    await this.auditLogService.log({
      action: 'USER_RESTORED',
      entityType: 'User',
      entityId: userId,
      userId: restoredBy,
      metadata: {
        email: user.email,
        deletedAt: user.deletedAt,
        restoredAt: restored.restoredAt,
        reason,
      },
    });

    return restored;
  }

  // Get restore history
  async getRestoreHistory(): Promise<User[]> {
    return await this.userRepository.find({
      where: {
        restoredAt: Not(IsNull()),
      },
      order: {
        restoredAt: 'DESC',
      },
      select: [
        'id',
        'email',
        'deletedAt',
        'deletedBy',
        'deleteReason',
        'restoredAt',
        'restoredBy',
        'restoreReason',
      ],
    });
  }
}
```

### Best Practices

```typescript
// ✅ GOOD: Use restore() for simple operations
await userRepository.restore(userId);

// ✅ GOOD: Use recover() when hooks needed
const user = await userRepository.findOne({
  where: { id: userId },
  withDeleted: true,
});
await userRepository.recover(user);

// ✅ GOOD: Always check if entity exists and is deleted
const user = await userRepository.findOne({
  where: { id: userId },
  withDeleted: true,
});

if (!user) {
  throw new NotFoundException('User not found');
}

if (!user.deletedAt) {
  throw new BadRequestException('User is not deleted');
}

// ✅ GOOD: Check affected rows with restore()
const result = await userRepository.restore(userId);
if (result.affected === 0) {
  throw new NotFoundException('User not found or not deleted');
}

// ❌ BAD: Forgetting withDeleted when loading
const user = await userRepository.findOneBy({ id: userId });
// Returns null for soft-deleted users!

// ✅ GOOD: Always use withDeleted
const user = await userRepository.findOne({
  where: { id: userId },
  withDeleted: true,
});

// ✅ GOOD: Add audit fields for restore tracking
@Column({ nullable: true })
restoredAt: Date;

@Column({ nullable: true })
restoredBy: number;

// ✅ GOOD: Validate before restore
if (daysSinceDeleted > 90) {
  throw new BadRequestException('Cannot restore after 90 days');
}

// ✅ GOOD: Use transactions for complex restores
await dataSource.transaction(async (manager) => {
  await manager.getRepository(User).restore(userId);
  await manager.getRepository(Post).restore({ authorId: userId });
});

// ✅ GOOD: Document restore behavior
/**
 * Restores a soft-deleted user.
 * @throws NotFoundException if user not found
 * @throws BadRequestException if user is not deleted
 */
async restoreUser(userId: number): Promise<void> {
  // ...
}

// ✅ GOOD: Provide admin-only restore endpoints
@UseGuards(AdminGuard)
@Post(':id/restore')
async restoreUser(@Param('id') id: number) {
  // ...
}
```

### Testing Soft Delete and Restore

```typescript
describe('User Restore', () => {
  let userRepository: Repository<User>;

  beforeEach(async () => {
    userRepository = dataSource.getRepository(User);
  });

  it('should restore soft-deleted user', async () => {
    // Create user
    const user = await userRepository.save({
      email: 'test@example.com',
      firstName: 'Test',
    });

    // Soft delete
    await userRepository.softDelete(user.id);

    // Verify soft-deleted
    const deleted = await userRepository.findOneBy({ id: user.id });
    expect(deleted).toBeNull();

    // Restore
    await userRepository.restore(user.id);

    // Verify restored
    const restored = await userRepository.findOneBy({ id: user.id });
    expect(restored).toBeDefined();
    expect(restored.email).toBe('test@example.com');
    expect(restored.deletedAt).toBeNull();
  });

  it('should trigger hooks on recover', async () => {
    const user = await userRepository.save({
      email: 'test@example.com',
    });

    await userRepository.softDelete(user.id);

    const deleted = await userRepository.findOne({
      where: { id: user.id },
      withDeleted: true,
    });

    const spy = jest.spyOn(deleted, 'afterRecover');
    await userRepository.recover(deleted);

    expect(spy).toHaveBeenCalled();
  });
});
```

**Interview Tip**: Explain that TypeORM provides two restore methods: **restore()** (direct SQL, fast, no hooks, accepts ID/criteria) and **recover()** (loads entity first, triggers hooks, returns entity). Emphasize **restore()** sets deletedAt to NULL by criteria - use for simple restores. **recover()** requires loading entity with **withDeleted: true** first - use when hooks needed. Highlight validation: check if entity exists AND is deleted before restore, verify result.affected with restore(), handle edge cases (already active, deleted too long ago). Mention audit trail: add restoredAt, restoredBy, restoreReason fields for tracking. For production: use restore() for simple cases, recover() when hooks needed (notifications, logging), validate before restore, use transactions for complex restores with relations, add admin-only restore endpoints, set time limits (e.g., can't restore after 90 days), document restore behavior. A strong answer demonstrates understanding of when restore() vs recover() is appropriate and proper validation patterns.

</details>

<details>
<summary>56. What is the count() method and how do you use it?</summary>

The **count()** method returns the number of entities matching given conditions without loading the actual entities. It's a lightweight operation that executes `SELECT COUNT(*)` SQL query, useful for pagination, analytics, and conditional logic without fetching full entity data.

### Basic Usage

```typescript
import { Repository } from 'typeorm';

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

  @CreateDateColumn()
  createdAt: Date;

  @DeleteDateColumn()
  deletedAt: Date;
}

const userRepository = dataSource.getRepository(User);

// Count all users (excludes soft-deleted)
const total = await userRepository.count();
console.log(total); // 150

// Count with conditions
const activeCount = await userRepository.count({
  where: { isActive: true },
});
console.log(activeCount); // 120

// Count with multiple conditions
const adultCount = await userRepository.count({
  where: {
    age: MoreThanOrEqual(18),
    isActive: true,
  },
});
console.log(adultCount); // 95
```

### count() with FindOptions

```typescript
// Count with where conditions
const count = await userRepository.count({
  where: { isActive: true },
});

// Count with OR conditions
const count = await userRepository.count({
  where: [
    { firstName: 'John' },
    { firstName: 'Jane' },
  ],
});

// Count with comparison operators
const count = await userRepository.count({
  where: {
    age: Between(18, 65),
    isActive: true,
  },
});

// Count with IN operator
const count = await userRepository.count({
  where: {
    city: In(['NYC', 'LA', 'Chicago']),
  },
});

// Count with LIKE pattern
const count = await userRepository.count({
  where: {
    email: Like('%@company.com'),
  },
});

// Count including soft-deleted
const totalWithDeleted = await userRepository.count({
  withDeleted: true,
});

// Count only soft-deleted
const deletedCount = await userRepository.count({
  where: {
    deletedAt: Not(IsNull()),
  },
  withDeleted: true,
});
```

### count() vs countBy()

```typescript
// count() - Uses FindOptions (more flexible)
const count1 = await userRepository.count({
  where: { isActive: true },
  withDeleted: true,
});

// countBy() - Simplified syntax (TypeORM 0.3+)
const count2 = await userRepository.countBy({ isActive: true });
// Equivalent to count({ where: { isActive: true } })

// countBy only supports simple equality
const count3 = await userRepository.countBy({
  isActive: true,
  age: 25,
});

// For complex queries, use count()
const count4 = await userRepository.count({
  where: {
    age: MoreThan(18),
    isActive: true,
  },
});
```

### count() vs find().length

```typescript
/*
╔═══════════════════════╤═══════════════════════╤═══════════════════════╗
║ FEATURE               │ count()               │ find().length         ║
╠═══════════════════════╪═══════════════════════╪═══════════════════════╣
║ SQL Query             │ SELECT COUNT(*)       │ SELECT *              ║
║ Data Transfer         │ ✅ Single number      │ ❌ All entity data    ║
║ Memory Usage          │ ✅ Minimal            │ ❌ High               ║
║ Performance           │ ✅ Fast               │ ⚠️ Slow (large data)  ║
║ Use Case              │ Count only needed     │ Need entities too     ║
╚═══════════════════════╧═══════════════════════╧═══════════════════════╝
*/

// ✅ GOOD: Use count() when only count is needed
const totalUsers = await userRepository.count();
console.log(`Total users: ${totalUsers}`);

// ❌ BAD: Loading all entities just to count
const users = await userRepository.find();
console.log(`Total users: ${users.length}`);
// Loads ALL user data into memory! Very slow for large tables.

// Example with 1 million users:
// count(): ~50ms, transfers 4 bytes
// find().length: ~30 seconds, transfers ~500MB
```

### findAndCount() for Pagination

```typescript
// findAndCount() - Get entities AND count in one call
const [users, total] = await userRepository.findAndCount({
  where: { isActive: true },
  skip: 20,
  take: 10,
  order: { createdAt: 'DESC' },
});

console.log(users.length); // 10 (current page)
console.log(total); // 157 (total matching count)

// Equivalent to two separate queries:
const users = await userRepository.find({
  where: { isActive: true },
  skip: 20,
  take: 10,
});

const total = await userRepository.count({
  where: { isActive: true },
});

// But findAndCount() is optimized (single DB round trip in some cases)
```

### Query Builder with getCount()

```typescript
// Simple count with query builder
const count = await userRepository
  .createQueryBuilder('user')
  .where('user.isActive = :active', { active: true })
  .getCount();

// Count with joins
const count = await userRepository
  .createQueryBuilder('user')
  .leftJoin('user.posts', 'post')
  .where('post.publishedAt IS NOT NULL')
  .getCount();

// Count distinct users
const count = await userRepository
  .createQueryBuilder('user')
  .select('COUNT(DISTINCT user.id)', 'count')
  .getRawOne();

console.log(count.count);

// Count with grouping (use getRawMany)
const counts = await userRepository
  .createQueryBuilder('user')
  .select('user.city')
  .addSelect('COUNT(*)', 'count')
  .groupBy('user.city')
  .getRawMany();

console.log(counts);
/*
[
  { user_city: 'NYC', count: '50' },
  { user_city: 'LA', count: '30' },
  { user_city: 'Chicago', count: '20' }
]
*/

// getManyAndCount() - Query builder equivalent of findAndCount()
const [users, total] = await userRepository
  .createQueryBuilder('user')
  .where('user.isActive = :active', { active: true })
  .skip(20)
  .take(10)
  .orderBy('user.createdAt', 'DESC')
  .getManyAndCount();
```

### Practical Examples

```typescript
export class UserService {
  constructor(
    private userRepository: Repository<User>
  ) {}

  // Get total user count
  async getTotalUsers(): Promise<number> {
    return await this.userRepository.count();
  }

  // Get active user count
  async getActiveUserCount(): Promise<number> {
    return await this.userRepository.count({
      where: { isActive: true },
    });
  }

  // Get user statistics
  async getUserStats(): Promise<{
    total: number;
    active: number;
    inactive: number;
    deleted: number;
  }> {
    const total = await this.userRepository.count();
    const active = await this.userRepository.count({
      where: { isActive: true },
    });
    const inactive = await this.userRepository.count({
      where: { isActive: false },
    });
    const deleted = await this.userRepository.count({
      where: { deletedAt: Not(IsNull()) },
      withDeleted: true,
    });

    return { total, active, inactive, deleted };
  }

  // Check if user exists
  async userExists(email: string): Promise<boolean> {
    const count = await this.userRepository.count({
      where: { email },
    });
    return count > 0;
  }

  // Count by age groups
  async countByAgeGroups(): Promise<Record<string, number>> {
    const children = await this.userRepository.count({
      where: { age: LessThan(18) },
    });

    const adults = await this.userRepository.count({
      where: { age: Between(18, 65) },
    });

    const seniors = await this.userRepository.count({
      where: { age: MoreThan(65) },
    });

    return {
      children,
      adults,
      seniors,
      total: children + adults + seniors,
    };
  }

  // Count new users in last N days
  async countNewUsers(days: number = 30): Promise<number> {
    const cutoffDate = new Date();
    cutoffDate.setDate(cutoffDate.getDate() - days);

    return await this.userRepository.count({
      where: {
        createdAt: MoreThan(cutoffDate),
      },
    });
  }

  // Pagination with count
  async getPaginatedUsers(
    page: number = 1,
    limit: number = 10
  ): Promise<{
    data: User[];
    total: number;
    page: number;
    totalPages: number;
  }> {
    const skip = (page - 1) * limit;

    const [data, total] = await this.userRepository.findAndCount({
      skip,
      take: limit,
      order: { createdAt: 'DESC' },
    });

    return {
      data,
      total,
      page,
      totalPages: Math.ceil(total / limit),
    };
  }

  // Conditional logic based on count
  async hasManyUsers(): Promise<boolean> {
    const count = await this.userRepository.count();
    return count > 1000;
  }

  // Count by multiple criteria
  async countByCriteria(criteria: {
    isActive?: boolean;
    minAge?: number;
    maxAge?: number;
    city?: string;
  }): Promise<number> {
    const where: any = {};

    if (criteria.isActive !== undefined) {
      where.isActive = criteria.isActive;
    }

    if (criteria.minAge !== undefined || criteria.maxAge !== undefined) {
      where.age = Between(
        criteria.minAge ?? 0,
        criteria.maxAge ?? 150
      );
    }

    if (criteria.city) {
      where.city = criteria.city;
    }

    return await this.userRepository.count({ where });
  }
}
```

### Dashboard Statistics

```typescript
export class DashboardService {
  constructor(
    private userRepository: Repository<User>,
    private postRepository: Repository<Post>,
    private orderRepository: Repository<Order>
  ) {}

  async getDashboardStats(): Promise<{
    users: {
      total: number;
      active: number;
      newToday: number;
      newThisWeek: number;
      newThisMonth: number;
    };
    posts: {
      total: number;
      published: number;
      draft: number;
    };
    orders: {
      total: number;
      pending: number;
      completed: number;
      cancelled: number;
    };
  }> {
    const today = new Date();
    today.setHours(0, 0, 0, 0);

    const weekAgo = new Date();
    weekAgo.setDate(weekAgo.getDate() - 7);

    const monthAgo = new Date();
    monthAgo.setMonth(monthAgo.getMonth() - 1);

    // User counts
    const [
      totalUsers,
      activeUsers,
      newUsersToday,
      newUsersWeek,
      newUsersMonth,
    ] = await Promise.all([
      this.userRepository.count(),
      this.userRepository.count({ where: { isActive: true } }),
      this.userRepository.count({ where: { createdAt: MoreThan(today) } }),
      this.userRepository.count({ where: { createdAt: MoreThan(weekAgo) } }),
      this.userRepository.count({ where: { createdAt: MoreThan(monthAgo) } }),
    ]);

    // Post counts
    const [totalPosts, publishedPosts, draftPosts] = await Promise.all([
      this.postRepository.count(),
      this.postRepository.count({ where: { status: 'published' } }),
      this.postRepository.count({ where: { status: 'draft' } }),
    ]);

    // Order counts
    const [totalOrders, pendingOrders, completedOrders, cancelledOrders] =
      await Promise.all([
        this.orderRepository.count(),
        this.orderRepository.count({ where: { status: 'pending' } }),
        this.orderRepository.count({ where: { status: 'completed' } }),
        this.orderRepository.count({ where: { status: 'cancelled' } }),
      ]);

    return {
      users: {
        total: totalUsers,
        active: activeUsers,
        newToday: newUsersToday,
        newThisWeek: newUsersWeek,
        newThisMonth: newUsersMonth,
      },
      posts: {
        total: totalPosts,
        published: publishedPosts,
        draft: draftPosts,
      },
      orders: {
        total: totalOrders,
        pending: pendingOrders,
        completed: completedOrders,
        cancelled: cancelledOrders,
      },
    };
  }
}
```

### REST API Examples

```typescript
// NestJS Controller
@Controller('users')
export class UserController {
  constructor(private userService: UserService) {}

  // Get total user count
  @Get('count')
  async getUserCount() {
    const count = await this.userService.getTotalUsers();
    return { count };
  }

  // Get user statistics
  @Get('stats')
  async getUserStats() {
    return await this.userService.getUserStats();
  }

  // Check if user exists
  @Head(':email')
  async checkUserExists(@Param('email') email: string) {
    const exists = await this.userService.userExists(email);
    return exists ? 200 : 404;
  }
}

// Express.js
app.get('/users/count', async (req, res) => {
  try {
    const count = await userRepository.count();
    res.json({ count });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.get('/users/stats', async (req, res) => {
  try {
    const total = await userRepository.count();
    const active = await userRepository.count({ where: { isActive: true } });
    const inactive = total - active;

    res.json({ total, active, inactive });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

### Performance Optimization

```typescript
// ✅ GOOD: Use count() for checking existence
const exists = (await userRepository.count({ where: { email } })) > 0;

// ❌ BAD: Loading entity just to check existence
const user = await userRepository.findOneBy({ email });
const exists = user !== null;

// ✅ GOOD: Parallel count queries
const [total, active, inactive] = await Promise.all([
  userRepository.count(),
  userRepository.count({ where: { isActive: true } }),
  userRepository.count({ where: { isActive: false } }),
]);

// ❌ BAD: Sequential count queries
const total = await userRepository.count();
const active = await userRepository.count({ where: { isActive: true } });
const inactive = await userRepository.count({ where: { isActive: false } });

// ✅ GOOD: Cache frequent counts
const cacheKey = 'users:total';
let total = await cache.get(cacheKey);

if (!total) {
  total = await userRepository.count();
  await cache.set(cacheKey, total, 300); // Cache 5 minutes
}

// ✅ GOOD: Use findAndCount for pagination
const [users, total] = await userRepository.findAndCount({
  skip,
  take: limit,
});

// ❌ BAD: Separate queries
const users = await userRepository.find({ skip, take: limit });
const total = await userRepository.count(); // Extra query
```

### Best Practices

```typescript
// ✅ GOOD: Use count() when only count needed
const total = await userRepository.count();

// ❌ BAD: Loading all data just to count
const users = await userRepository.find();
const total = users.length;

// ✅ GOOD: Use countBy() for simple equality
const activeCount = await userRepository.countBy({ isActive: true });

// ✅ GOOD: Use count() for complex conditions
const adultCount = await userRepository.count({
  where: { age: MoreThanOrEqual(18) },
});

// ✅ GOOD: Use findAndCount for pagination
const [data, total] = await userRepository.findAndCount({
  skip,
  take: limit,
});

// ✅ GOOD: Include withDeleted for accurate totals
const totalIncludingDeleted = await userRepository.count({
  withDeleted: true,
});

// ✅ GOOD: Cache frequently accessed counts
const total = await cache.getOrSet('users:count', async () => {
  return await userRepository.count();
}, 300);

// ✅ GOOD: Use Promise.all for parallel counts
const [total, active, deleted] = await Promise.all([
  userRepository.count(),
  userRepository.count({ where: { isActive: true } }),
  userRepository.count({
    where: { deletedAt: Not(IsNull()) },
    withDeleted: true,
  }),
]);

// ✅ GOOD: Add indexes for count queries on filtered columns
@Entity()
export class User {
  @Column()
  @Index() // Index for count() with isActive filter
  isActive: boolean;

  @Column()
  @Index() // Index for count() with age filter
  age: number;
}
```

**Interview Tip**: Explain that **count()** returns number of entities matching conditions using `SELECT COUNT(*)` SQL - lightweight, fast, doesn't load entity data. Emphasize **count()** accepts FindOptions (where, withDeleted) for filtering. Highlight **countBy()** as simplified version for equality conditions only (TypeORM 0.3+). Mention **findAndCount()** returns both entities and count in optimized single call - ideal for pagination. Discuss performance: count() is 100-1000x faster than find().length for large datasets (transfers single number vs all entity data). For production: use count() when only count needed, findAndCount() for pagination, cache frequent counts (5-10 min TTL), run parallel counts with Promise.all, index columns used in count filters, use countBy() for simple equality checks. A strong answer demonstrates understanding of when count() vs findAndCount() is appropriate and the massive performance difference vs loading all entities.

</details>

<details>
<summary>57. How do you create custom repository methods?</summary>

**Custom repository methods** encapsulate complex queries and business logic in reusable methods. In TypeORM 0.3+, create custom repositories by **extending Repository class** and using **dataSource.getRepository().extend()** or by calling **dataSource.withRepository()** to register custom repositories.

### Modern Approach (TypeORM 0.3+)

```typescript
import { Repository, DataSource } from 'typeorm';

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

  @CreateDateColumn()
  createdAt: Date;
}

// ===== Method 1: Extend repository with custom methods =====
export const UserRepository = dataSource.getRepository(User).extend({
  // Custom method: Find by full name
  findByFullName(firstName: string, lastName: string): Promise<User[]> {
    return this.createQueryBuilder('user')
      .where('user.firstName = :firstName', { firstName })
      .andWhere('user.lastName = :lastName', { lastName })
      .getMany();
  },

  // Custom method: Find active users
  findActiveUsers(): Promise<User[]> {
    return this.find({
      where: { isActive: true },
      order: { createdAt: 'DESC' },
    });
  },

  // Custom method: Find by email (case-insensitive)
  async findByEmailIgnoreCase(email: string): Promise<User | null> {
    return this.createQueryBuilder('user')
      .where('LOWER(user.email) = LOWER(:email)', { email })
      .getOne();
  },

  // Custom method: Count active users
  async countActiveUsers(): Promise<number> {
    return this.count({
      where: { isActive: true },
    });
  },

  // Custom method: Find adults (age >= 18)
  findAdults(): Promise<User[]> {
    return this.createQueryBuilder('user')
      .where('user.age >= :minAge', { minAge: 18 })
      .orderBy('user.age', 'ASC')
      .getMany();
  },

  // Custom method: Search users
  async searchUsers(query: string): Promise<User[]> {
    return this.createQueryBuilder('user')
      .where('user.firstName ILIKE :query', { query: `%${query}%` })
      .orWhere('user.lastName ILIKE :query', { query: `%${query}%` })
      .orWhere('user.email ILIKE :query', { query: `%${query}%` })
      .getMany();
  },
});

// Usage
const users = await UserRepository.findByFullName('John', 'Doe');
const activeUsers = await UserRepository.findActiveUsers();
const user = await UserRepository.findByEmailIgnoreCase('JOHN@EXAMPLE.COM');
```

### Method 2: Class-based Custom Repository

```typescript
import { Repository, DataSource } from 'typeorm';

export class UserRepository extends Repository<User> {
  constructor(private dataSource: DataSource) {
    super(User, dataSource.createEntityManager());
  }

  // Custom method: Find by full name
  async findByFullName(
    firstName: string,
    lastName: string
  ): Promise<User[]> {
    return this.createQueryBuilder('user')
      .where('user.firstName = :firstName', { firstName })
      .andWhere('user.lastName = :lastName', { lastName })
      .getMany();
  }

  // Custom method: Find or create
  async findOrCreate(data: Partial<User>): Promise<User> {
    let user = await this.findOneBy({ email: data.email });

    if (!user) {
      user = this.create(data);
      await this.save(user);
    }

    return user;
  }

  // Custom method: Update or create (upsert)
  async upsert(email: string, data: Partial<User>): Promise<User> {
    let user = await this.findOneBy({ email });

    if (user) {
      Object.assign(user, data);
    } else {
      user = this.create({ ...data, email });
    }

    return await this.save(user);
  }

  // Custom method: Deactivate old users
  async deactivateOldUsers(days: number = 90): Promise<number> {
    const cutoffDate = new Date();
    cutoffDate.setDate(cutoffDate.getDate() - days);

    const result = await this.update(
      {
        lastLoginAt: LessThan(cutoffDate),
        isActive: true,
      },
      {
        isActive: false,
      }
    );

    return result.affected || 0;
  }

  // Custom method: Get user statistics
  async getStatistics(): Promise<{
    total: number;
    active: number;
    inactive: number;
    averageAge: number;
  }> {
    const [total, active] = await Promise.all([
      this.count(),
      this.count({ where: { isActive: true } }),
    ]);

    const result = await this.createQueryBuilder('user')
      .select('AVG(user.age)', 'averageAge')
      .getRawOne();

    return {
      total,
      active,
      inactive: total - active,
      averageAge: parseFloat(result.averageAge) || 0,
    };
  }

  // Custom method: Pagination with search
  async searchWithPagination(
    query: string,
    page: number = 1,
    limit: number = 10
  ): Promise<{
    data: User[];
    total: number;
    page: number;
    totalPages: number;
  }> {
    const skip = (page - 1) * limit;

    const [data, total] = await this.createQueryBuilder('user')
      .where('user.firstName ILIKE :query', { query: `%${query}%` })
      .orWhere('user.lastName ILIKE :query', { query: `%${query}%` })
      .orWhere('user.email ILIKE :query', { query: `%${query}%` })
      .skip(skip)
      .take(limit)
      .orderBy('user.createdAt', 'DESC')
      .getManyAndCount();

    return {
      data,
      total,
      page,
      totalPages: Math.ceil(total / limit),
    };
  }
}

// Register custom repository
export const UserRepositoryInstance = (dataSource: DataSource) =>
  dataSource.getRepository(User).extend(UserRepository.prototype);
```

### Method 3: Repository Factory Pattern

```typescript
export function createUserRepository(dataSource: DataSource) {
  const baseRepository = dataSource.getRepository(User);

  return baseRepository.extend({
    // Custom method: Find by email domain
    async findByEmailDomain(domain: string): Promise<User[]> {
      return this.createQueryBuilder('user')
        .where('user.email LIKE :domain', { domain: `%@${domain}` })
        .getMany();
    },

    // Custom method: Find users by age range
    async findByAgeRange(minAge: number, maxAge: number): Promise<User[]> {
      return this.find({
        where: {
          age: Between(minAge, maxAge),
        },
        order: {
          age: 'ASC',
        },
      });
    },

    // Custom method: Bulk update status
    async bulkUpdateStatus(
      userIds: number[],
      isActive: boolean
    ): Promise<number> {
      const result = await this.update(userIds, { isActive });
      return result.affected || 0;
    },

    // Custom method: Get recent signups
    async getRecentSignups(days: number = 7): Promise<User[]> {
      const cutoffDate = new Date();
      cutoffDate.setDate(cutoffDate.getDate() - days);

      return this.find({
        where: {
          createdAt: MoreThan(cutoffDate),
        },
        order: {
          createdAt: 'DESC',
        },
      });
    },

    // Custom method: Check email availability
    async isEmailAvailable(email: string): Promise<boolean> {
      const count = await this.count({
        where: { email },
      });
      return count === 0;
    },

    // Custom method: Find with related data
    async findWithProfile(userId: number): Promise<User | null> {
      return this.findOne({
        where: { id: userId },
        relations: ['profile', 'posts'],
        select: {
          id: true,
          firstName: true,
          lastName: true,
          email: true,
          profile: {
            bio: true,
            avatarUrl: true,
          },
        },
      });
    },
  });
}

// Usage
const userRepository = createUserRepository(dataSource);
const users = await userRepository.findByEmailDomain('company.com');
```

### NestJS Integration

```typescript
// user.repository.ts
import { Injectable } from '@nestjs/common';
import { DataSource, Repository } from 'typeorm';

@Injectable()
export class UserRepository extends Repository<User> {
  constructor(private dataSource: DataSource) {
    super(User, dataSource.createEntityManager());
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.findOne({
      where: { email },
    });
  }

  async findActiveUsers(): Promise<User[]> {
    return this.find({
      where: { isActive: true },
      order: { createdAt: 'DESC' },
    });
  }

  async searchUsers(query: string): Promise<User[]> {
    return this.createQueryBuilder('user')
      .where('user.firstName ILIKE :query', { query: `%${query}%` })
      .orWhere('user.lastName ILIKE :query', { query: `%${query}%` })
      .orWhere('user.email ILIKE :query', { query: `%${query}%` })
      .getMany();
  }

  async getUserWithPosts(userId: number): Promise<User | null> {
    return this.findOne({
      where: { id: userId },
      relations: ['posts'],
    });
  }

  async deactivateUser(userId: number): Promise<void> {
    await this.update(userId, { isActive: false });
  }

  async countByStatus(isActive: boolean): Promise<number> {
    return this.count({
      where: { isActive },
    });
  }
}

// user.module.ts
@Module({
  providers: [UserRepository, UserService],
  exports: [UserRepository],
})
export class UserModule {}

// user.service.ts
@Injectable()
export class UserService {
  constructor(private userRepository: UserRepository) {}

  async findByEmail(email: string): Promise<User> {
    const user = await this.userRepository.findByEmail(email);
    
    if (!user) {
      throw new NotFoundException('User not found');
    }

    return user;
  }

  async getActiveUsers(): Promise<User[]> {
    return this.userRepository.findActiveUsers();
  }

  async searchUsers(query: string): Promise<User[]> {
    return this.userRepository.searchUsers(query);
  }
}
```

### Complex Custom Methods

```typescript
export const UserRepository = dataSource.getRepository(User).extend({
  // Advanced search with multiple filters
  async advancedSearch(filters: {
    query?: string;
    isActive?: boolean;
    minAge?: number;
    maxAge?: number;
    city?: string;
    sortBy?: 'firstName' | 'lastName' | 'createdAt';
    sortOrder?: 'ASC' | 'DESC';
    page?: number;
    limit?: number;
  }): Promise<{
    data: User[];
    total: number;
    page: number;
    totalPages: number;
  }> {
    const page = filters.page || 1;
    const limit = filters.limit || 10;
    const skip = (page - 1) * limit;

    let queryBuilder = this.createQueryBuilder('user');

    // Text search
    if (filters.query) {
      queryBuilder = queryBuilder.where(
        '(user.firstName ILIKE :query OR user.lastName ILIKE :query OR user.email ILIKE :query)',
        { query: `%${filters.query}%` }
      );
    }

    // Status filter
    if (filters.isActive !== undefined) {
      queryBuilder = queryBuilder.andWhere('user.isActive = :isActive', {
        isActive: filters.isActive,
      });
    }

    // Age range filter
    if (filters.minAge !== undefined || filters.maxAge !== undefined) {
      const minAge = filters.minAge || 0;
      const maxAge = filters.maxAge || 150;
      queryBuilder = queryBuilder.andWhere(
        'user.age BETWEEN :minAge AND :maxAge',
        { minAge, maxAge }
      );
    }

    // City filter
    if (filters.city) {
      queryBuilder = queryBuilder.andWhere('user.city = :city', {
        city: filters.city,
      });
    }

    // Sorting
    const sortBy = filters.sortBy || 'createdAt';
    const sortOrder = filters.sortOrder || 'DESC';
    queryBuilder = queryBuilder.orderBy(`user.${sortBy}`, sortOrder);

    // Pagination
    const [data, total] = await queryBuilder
      .skip(skip)
      .take(limit)
      .getManyAndCount();

    return {
      data,
      total,
      page,
      totalPages: Math.ceil(total / limit),
    };
  },

  // Find users with specific criteria and count posts
  async findUsersWithPostCount(): Promise<
    Array<User & { postCount: number }>
  > {
    const result = await this.createQueryBuilder('user')
      .leftJoin('user.posts', 'post')
      .select([
        'user.id',
        'user.firstName',
        'user.lastName',
        'user.email',
      ])
      .addSelect('COUNT(post.id)', 'postCount')
      .groupBy('user.id')
      .orderBy('postCount', 'DESC')
      .getRawAndEntities();

    return result.entities.map((user, index) => ({
      ...user,
      postCount: parseInt(result.raw[index].postCount) || 0,
    }));
  },

  // Batch operation: Create or update multiple users
  async batchUpsert(users: Partial<User>[]): Promise<User[]> {
    const results: User[] = [];

    for (const userData of users) {
      let user = await this.findOneBy({ email: userData.email });

      if (user) {
        Object.assign(user, userData);
      } else {
        user = this.create(userData);
      }

      results.push(await this.save(user));
    }

    return results;
  },

  // Find users by multiple IDs with specific relations
  async findByIdsWithRelations(
    ids: number[],
    relations: string[] = []
  ): Promise<User[]> {
    return this.find({
      where: {
        id: In(ids),
      },
      relations,
    });
  },

  // Get user analytics
  async getAnalytics(): Promise<{
    totalUsers: number;
    activeUsers: number;
    inactiveUsers: number;
    averageAge: number;
    usersPerCity: Array<{ city: string; count: number }>;
    recentSignups: number;
  }> {
    const [totalUsers, activeUsers] = await Promise.all([
      this.count(),
      this.count({ where: { isActive: true } }),
    ]);

    const ageResult = await this.createQueryBuilder('user')
      .select('AVG(user.age)', 'averageAge')
      .getRawOne();

    const cityResults = await this.createQueryBuilder('user')
      .select('user.city', 'city')
      .addSelect('COUNT(*)', 'count')
      .groupBy('user.city')
      .orderBy('count', 'DESC')
      .getRawMany();

    const sevenDaysAgo = new Date();
    sevenDaysAgo.setDate(sevenDaysAgo.getDate() - 7);

    const recentSignups = await this.count({
      where: {
        createdAt: MoreThan(sevenDaysAgo),
      },
    });

    return {
      totalUsers,
      activeUsers,
      inactiveUsers: totalUsers - activeUsers,
      averageAge: parseFloat(ageResult.averageAge) || 0,
      usersPerCity: cityResults.map(r => ({
        city: r.city,
        count: parseInt(r.count),
      })),
      recentSignups,
    };
  },
});
```

### Legacy Approach (@EntityRepository - Deprecated)

```typescript
// ⚠️ DEPRECATED: This approach is no longer recommended in TypeORM 0.3+
import { EntityRepository, Repository } from 'typeorm';

@EntityRepository(User)
export class UserRepository extends Repository<User> {
  findByEmail(email: string): Promise<User | null> {
    return this.findOne({ where: { email } });
  }

  findActiveUsers(): Promise<User[]> {
    return this.find({ where: { isActive: true } });
  }
}

// Usage (deprecated)
// const userRepository = connection.getCustomRepository(UserRepository);
```

### Best Practices

```typescript
// ✅ GOOD: Use extend() for custom methods (TypeORM 0.3+)
export const UserRepository = dataSource.getRepository(User).extend({
  findByEmail(email: string): Promise<User | null> {
    return this.findOne({ where: { email } });
  },
});

// ✅ GOOD: Create class-based repository for complex logic
export class UserRepository extends Repository<User> {
  constructor(dataSource: DataSource) {
    super(User, dataSource.createEntityManager());
  }

  // Custom methods here
}

// ✅ GOOD: Use query builder for complex queries
async findActiveUsersWithPosts(): Promise<User[]> {
  return this.createQueryBuilder('user')
    .leftJoinAndSelect('user.posts', 'post')
    .where('user.isActive = :active', { active: true })
    .getMany();
}

// ✅ GOOD: Encapsulate business logic
async deactivateInactiveUsers(days: number): Promise<number> {
  const cutoffDate = new Date();
  cutoffDate.setDate(cutoffDate.getDate() - days);

  const result = await this.update(
    {
      lastLoginAt: LessThan(cutoffDate),
      isActive: true,
    },
    {
      isActive: false,
    }
  );

  return result.affected || 0;
}

// ✅ GOOD: Return domain-specific types
async getUserWithFullProfile(userId: number): Promise<UserWithProfile | null> {
  const user = await this.findOne({
    where: { id: userId },
    relations: ['profile', 'posts'],
  });

  if (!user) return null;

  return {
    ...user,
    fullName: `${user.firstName} ${user.lastName}`,
    postCount: user.posts.length,
  };
}

// ❌ BAD: Don't use deprecated @EntityRepository
@EntityRepository(User) // Removed in TypeORM 0.3+
export class UserRepository extends Repository<User> {}

// ❌ BAD: Don't mix business logic in repository
async createUserAndSendEmail(data: CreateUserDto): Promise<User> {
  const user = await this.save(data);
  await emailService.send(user.email); // Business logic!
  return user;
}

// ✅ GOOD: Keep repository focused on data access
async createUser(data: CreateUserDto): Promise<User> {
  return this.save(data);
}

// ❌ BAD: Don't expose raw entity manager
getManager() {
  return this.manager; // Breaks encapsulation
}

// ✅ GOOD: Document custom methods
/**
 * Finds users by email domain (case-insensitive).
 * @param domain - Email domain (e.g., 'company.com')
 * @returns Array of users with matching email domain
 */
async findByEmailDomain(domain: string): Promise<User[]> {
  return this.createQueryBuilder('user')
    .where('user.email ILIKE :domain', { domain: `%@${domain}` })
    .getMany();
}
```

### Testing Custom Repositories

```typescript
describe('UserRepository', () => {
  let dataSource: DataSource;
  let userRepository: ReturnType<typeof createUserRepository>;

  beforeAll(async () => {
    dataSource = await createTestDataSource();
    userRepository = createUserRepository(dataSource);
  });

  afterAll(async () => {
    await dataSource.destroy();
  });

  describe('findByEmail', () => {
    it('should find user by email', async () => {
      const user = await userRepository.save({
        firstName: 'John',
        lastName: 'Doe',
        email: 'john@example.com',
      });

      const found = await userRepository.findByEmail('john@example.com');

      expect(found).toBeDefined();
      expect(found.id).toBe(user.id);
      expect(found.email).toBe('john@example.com');
    });

    it('should return null if user not found', async () => {
      const found = await userRepository.findByEmail('notfound@example.com');
      expect(found).toBeNull();
    });
  });

  describe('findActiveUsers', () => {
    it('should return only active users', async () => {
      await userRepository.save([
        { firstName: 'Active', email: 'active@test.com', isActive: true },
        { firstName: 'Inactive', email: 'inactive@test.com', isActive: false },
      ]);

      const active = await userRepository.findActiveUsers();

      expect(active.length).toBe(1);
      expect(active[0].isActive).toBe(true);
    });
  });
});
```

**Interview Tip**: Explain that **custom repository methods** encapsulate complex queries and business logic in reusable methods. In **TypeORM 0.3+**, use **dataSource.getRepository().extend()** to add custom methods - returns extended repository with both standard and custom methods. Mention **class-based approach**: extend Repository class for complex logic, inject DataSource in constructor. Highlight benefits: code reusability, testability, encapsulation, type safety, maintainability. Emphasize **@EntityRepository decorator is deprecated** in 0.3+ - use extend() or class-based approach instead. For production: keep repositories focused on data access (no business logic), use query builder for complex queries, document custom methods, return domain-specific types, write unit tests for custom methods, use NestJS @Injectable() for dependency injection. A strong answer demonstrates understanding of modern TypeORM patterns and separation of concerns between repositories and services.

</details>

<details>
<summary>58. What is @EntityRepository decorator (deprecated) and its replacement?</summary>

The **@EntityRepository** decorator was used in TypeORM 0.2.x to create custom repositories but was **deprecated and removed in TypeORM 0.3+**. The modern replacement is **extending Repository** with the **extend()** method or creating class-based repositories with dependency injection.

### Old Approach (@EntityRepository - Deprecated)

```typescript
// ⚠️ DEPRECATED: TypeORM 0.2.x only
import { EntityRepository, Repository } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  email: string;

  @Column()
  firstName: string;

  @Column({ default: true })
  isActive: boolean;
}

// Old way: @EntityRepository decorator
@EntityRepository(User)
export class UserRepository extends Repository<User> {
  findByEmail(email: string): Promise<User | null> {
    return this.findOne({ where: { email } });
  }

  findActiveUsers(): Promise<User[]> {
    return this.find({ where: { isActive: true } });
  }

  async getUserStats(): Promise<{ total: number; active: number }> {
    const total = await this.count();
    const active = await this.count({ where: { isActive: true } });
    return { total, active };
  }
}

// Old usage (TypeORM 0.2.x)
const connection = await createConnection();
const userRepository = connection.getCustomRepository(UserRepository);

const user = await userRepository.findByEmail('john@example.com');
const stats = await userRepository.getUserStats();
```

### Why @EntityRepository Was Deprecated

```typescript
/*
REASONS FOR DEPRECATION:

1. ❌ Tight Coupling with Decorators
   - Required experimental decorator support
   - Caused issues with TypeScript configurations
   - Not compatible with ECMAScript standard decorators

2. ❌ Dependency Injection Problems
   - Difficult to inject dependencies into custom repositories
   - Hard to test (couldn't easily mock dependencies)
   - Poor integration with DI frameworks (NestJS, etc.)

3. ❌ Connection/DataSource Confusion
   - Used connection.getCustomRepository() which was confusing
   - Hard to manage multiple connections
   - Not compatible with new DataSource API

4. ❌ Maintenance Burden
   - Required special handling in TypeORM internals
   - Conflicted with standard TypeScript/JavaScript patterns
   - Increased complexity for framework maintainers

5. ✅ Better Alternatives Available
   - extend() method is simpler and more flexible
   - Class-based approach works better with DI
   - More testable and maintainable
*/
```

### Modern Replacement 1: extend() Method

```typescript
// ✅ MODERN: TypeORM 0.3+ with extend()
import { DataSource, Repository } from 'typeorm';

export const UserRepository = (dataSource: DataSource) => {
  return dataSource.getRepository(User).extend({
    // Custom method: Find by email
    findByEmail(email: string): Promise<User | null> {
      return this.findOne({ where: { email } });
    },

    // Custom method: Find active users
    findActiveUsers(): Promise<User[]> {
      return this.find({
        where: { isActive: true },
        order: { createdAt: 'DESC' },
      });
    },

    // Custom method: Get statistics
    async getUserStats(): Promise<{ total: number; active: number }> {
      const total = await this.count();
      const active = await this.count({ where: { isActive: true } });
      return { total, active };
    },

    // Custom method: Search users
    async searchUsers(query: string): Promise<User[]> {
      return this.createQueryBuilder('user')
        .where('user.firstName ILIKE :query', { query: `%${query}%` })
        .orWhere('user.lastName ILIKE :query', { query: `%${query}%` })
        .orWhere('user.email ILIKE :query', { query: `%${query}%` })
        .getMany();
    },

    // Custom method: Deactivate old users
    async deactivateOldUsers(days: number = 90): Promise<number> {
      const cutoffDate = new Date();
      cutoffDate.setDate(cutoffDate.getDate() - days);

      const result = await this.update(
        {
          lastLoginAt: LessThan(cutoffDate),
          isActive: true,
        },
        {
          isActive: false,
        }
      );

      return result.affected || 0;
    },
  });
};

// Usage
const dataSource = await initializeDataSource();
const userRepository = UserRepository(dataSource);

const user = await userRepository.findByEmail('john@example.com');
const stats = await userRepository.getUserStats();
const activeUsers = await userRepository.findActiveUsers();
```

### Modern Replacement 2: Class-Based Repository

```typescript
// ✅ MODERN: Class-based approach with DataSource injection
import { DataSource, Repository } from 'typeorm';

export class UserRepository extends Repository<User> {
  constructor(private dataSource: DataSource) {
    super(User, dataSource.createEntityManager());
  }

  // Custom method: Find by email
  async findByEmail(email: string): Promise<User | null> {
    return this.findOne({ where: { email } });
  }

  // Custom method: Find active users
  async findActiveUsers(): Promise<User[]> {
    return this.find({
      where: { isActive: true },
      order: { createdAt: 'DESC' },
    });
  }

  // Custom method: Get statistics
  async getUserStats(): Promise<{ total: number; active: number }> {
    const [total, active] = await Promise.all([
      this.count(),
      this.count({ where: { isActive: true } }),
    ]);

    return { total, active };
  }

  // Custom method: Search users
  async searchUsers(query: string): Promise<User[]> {
    return this.createQueryBuilder('user')
      .where('user.firstName ILIKE :query', { query: `%${query}%` })
      .orWhere('user.lastName ILIKE :query', { query: `%${query}%` })
      .orWhere('user.email ILIKE :query', { query: `%${query}%` })
      .getMany();
  }

  // Custom method: Find or create
  async findOrCreate(data: Partial<User>): Promise<User> {
    let user = await this.findByEmail(data.email);

    if (!user) {
      user = this.create(data);
      await this.save(user);
    }

    return user;
  }

  // Custom method: Batch operations
  async bulkUpdateStatus(userIds: number[], isActive: boolean): Promise<number> {
    const result = await this.update(userIds, { isActive });
    return result.affected || 0;
  }
}

// Usage
const dataSource = await initializeDataSource();
const userRepository = new UserRepository(dataSource);

const user = await userRepository.findByEmail('john@example.com');
const stats = await userRepository.getUserStats();
```

### Modern Replacement 3: NestJS Integration

```typescript
// ✅ MODERN: NestJS with @Injectable()
import { Injectable } from '@nestjs/common';
import { DataSource, Repository } from 'typeorm';

@Injectable()
export class UserRepository extends Repository<User> {
  constructor(private dataSource: DataSource) {
    super(User, dataSource.createEntityManager());
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.findOne({ where: { email } });
  }

  async findActiveUsers(): Promise<User[]> {
    return this.find({
      where: { isActive: true },
      order: { createdAt: 'DESC' },
    });
  }

  async searchUsers(query: string): Promise<User[]> {
    return this.createQueryBuilder('user')
      .where('user.firstName ILIKE :query', { query: `%${query}%` })
      .orWhere('user.lastName ILIKE :query', { query: `%${query}%` })
      .orWhere('user.email ILIKE :query', { query: `%${query}%` })
      .getMany();
  }

  async getUserWithPosts(userId: number): Promise<User | null> {
    return this.findOne({
      where: { id: userId },
      relations: ['posts'],
    });
  }

  async deactivateUser(userId: number): Promise<void> {
    await this.update(userId, { isActive: false });
  }

  async countByStatus(isActive: boolean): Promise<number> {
    return this.count({ where: { isActive } });
  }
}

// user.module.ts
@Module({
  providers: [UserRepository, UserService],
  exports: [UserRepository],
})
export class UserModule {}

// user.service.ts
@Injectable()
export class UserService {
  constructor(
    private userRepository: UserRepository,
    private emailService: EmailService // Can inject other dependencies!
  ) {}

  async registerUser(data: CreateUserDto): Promise<User> {
    const user = await this.userRepository.save(data);
    await this.emailService.sendWelcomeEmail(user.email);
    return user;
  }

  async findByEmail(email: string): Promise<User> {
    const user = await this.userRepository.findByEmail(email);

    if (!user) {
      throw new NotFoundException('User not found');
    }

    return user;
  }

  async searchUsers(query: string): Promise<User[]> {
    return this.userRepository.searchUsers(query);
  }
}
```

### Migration Guide: From @EntityRepository to Modern Approach

```typescript
// ===== BEFORE (TypeORM 0.2.x) =====
import { EntityRepository, Repository } from 'typeorm';

@EntityRepository(User)
export class UserRepository extends Repository<User> {
  findByEmail(email: string): Promise<User | null> {
    return this.findOne({ where: { email } });
  }
}

// Usage
const connection = await createConnection();
const userRepository = connection.getCustomRepository(UserRepository);
await userRepository.findByEmail('john@example.com');

// ===== AFTER (TypeORM 0.3+) - Option 1: extend() =====
export const UserRepository = (dataSource: DataSource) => {
  return dataSource.getRepository(User).extend({
    findByEmail(email: string): Promise<User | null> {
      return this.findOne({ where: { email } });
    },
  });
};

// Usage
const dataSource = await initializeDataSource();
const userRepository = UserRepository(dataSource);
await userRepository.findByEmail('john@example.com');

// ===== AFTER (TypeORM 0.3+) - Option 2: Class-based =====
export class UserRepository extends Repository<User> {
  constructor(private dataSource: DataSource) {
    super(User, dataSource.createEntityManager());
  }

  findByEmail(email: string): Promise<User | null> {
    return this.findOne({ where: { email } });
  }
}

// Usage
const dataSource = await initializeDataSource();
const userRepository = new UserRepository(dataSource);
await userRepository.findByEmail('john@example.com');

// ===== AFTER (TypeORM 0.3+) - Option 3: NestJS =====
@Injectable()
export class UserRepository extends Repository<User> {
  constructor(private dataSource: DataSource) {
    super(User, dataSource.createEntityManager());
  }

  findByEmail(email: string): Promise<User | null> {
    return this.findOne({ where: { email } });
  }
}

@Module({
  providers: [UserRepository],
  exports: [UserRepository],
})
export class UserModule {}

// Usage via dependency injection
@Injectable()
export class UserService {
  constructor(private userRepository: UserRepository) {}

  async getUser(email: string): Promise<User> {
    return this.userRepository.findByEmail(email);
  }
}
```

### Comparison Table

```typescript
/*
╔═══════════════════════════╤═══════════════════════════╤═══════════════════════════╗
║ FEATURE                   │ @EntityRepository (OLD)   │ extend() / Class (NEW)    ║
╠═══════════════════════════╪═══════════════════════════╪═══════════════════════════╣
║ TypeORM Version           │ 0.2.x only                │ 0.3.x+                    ║
║ Status                    │ ❌ Deprecated/Removed     │ ✅ Recommended            ║
║ Decorator Required        │ ✅ Yes (@EntityRepository)│ ❌ No                     ║
║ Dependency Injection      │ ⚠️ Limited                │ ✅ Full support           ║
║ Testing                   │ ⚠️ Harder to mock         │ ✅ Easy to mock           ║
║ NestJS Integration        │ ⚠️ Complex workarounds    │ ✅ Native support         ║
║ Multiple DataSources      │ ⚠️ Confusing              │ ✅ Clear                  ║
║ Custom Dependencies       │ ❌ Not supported          │ ✅ Easy injection         ║
║ Type Safety               │ ✅ Good                   │ ✅ Excellent              ║
║ Code Complexity           │ ⚠️ Medium                 │ ✅ Low                    ║
║ Maintainability           │ ⚠️ Poor (deprecated)      │ ✅ Excellent              ║
╚═══════════════════════════╧═══════════════════════════╧═══════════════════════════╝
*/
```

### Benefits of Modern Approach

```typescript
// 1. ✅ Better Dependency Injection
@Injectable()
export class UserRepository extends Repository<User> {
  constructor(
    private dataSource: DataSource,
    private cacheService: CacheService, // Can inject other services!
    private logger: LoggerService
  ) {
    super(User, dataSource.createEntityManager());
  }

  async findByEmail(email: string): Promise<User | null> {
    const cacheKey = `user:${email}`;
    const cached = await this.cacheService.get(cacheKey);

    if (cached) {
      this.logger.log(`Cache hit for ${email}`);
      return cached;
    }

    const user = await this.findOne({ where: { email } });
    await this.cacheService.set(cacheKey, user, 300);

    return user;
  }
}

// 2. ✅ Easier Testing
describe('UserRepository', () => {
  let userRepository: UserRepository;
  let mockDataSource: jest.Mocked<DataSource>;
  let mockCacheService: jest.Mocked<CacheService>;

  beforeEach(() => {
    mockDataSource = createMockDataSource();
    mockCacheService = createMockCacheService();

    userRepository = new UserRepository(
      mockDataSource,
      mockCacheService,
      mockLogger
    );
  });

  it('should find user by email', async () => {
    // Easy to test with mocked dependencies!
  });
});

// 3. ✅ Better Type Safety
export const UserRepository = (dataSource: DataSource) => {
  return dataSource.getRepository(User).extend({
    findByEmail(email: string): Promise<User | null> {
      // 'this' is correctly typed as Repository<User> & custom methods
      return this.findOne({ where: { email } });
    },
  });
};

// 4. ✅ Multiple DataSources Support
export class UserRepository extends Repository<User> {
  constructor(
    @Inject('PRIMARY_DATA_SOURCE') private primaryDs: DataSource
  ) {
    super(User, primaryDs.createEntityManager());
  }
}

export class UserArchiveRepository extends Repository<User> {
  constructor(
    @Inject('ARCHIVE_DATA_SOURCE') private archiveDs: DataSource
  ) {
    super(User, archiveDs.createEntityManager());
  }
}
```

### Best Practices for Migration

```typescript
// ✅ GOOD: Use extend() for simple custom methods
export const UserRepository = (dataSource: DataSource) =>
  dataSource.getRepository(User).extend({
    findByEmail(email: string) {
      return this.findOne({ where: { email } });
    },
  });

// ✅ GOOD: Use class-based for complex logic with dependencies
@Injectable()
export class UserRepository extends Repository<User> {
  constructor(
    private dataSource: DataSource,
    private cacheService: CacheService
  ) {
    super(User, dataSource.createEntityManager());
  }

  async findByEmail(email: string): Promise<User | null> {
    // Can use injected dependencies
    const cached = await this.cacheService.get(`user:${email}`);
    if (cached) return cached;

    const user = await this.findOne({ where: { email } });
    await this.cacheService.set(`user:${email}`, user);
    return user;
  }
}

// ❌ BAD: Don't try to use @EntityRepository in TypeORM 0.3+
@EntityRepository(User) // Will cause error!
export class UserRepository extends Repository<User> {}

// ❌ BAD: Don't use getCustomRepository (removed)
const repo = connection.getCustomRepository(UserRepository); // Doesn't exist!

// ✅ GOOD: Document migration path for team
/**
 * UserRepository - Custom repository for User entity
 * 
 * MIGRATION NOTE: Previously used @EntityRepository decorator.
 * Now uses class-based approach with @Injectable() for NestJS compatibility.
 * 
 * @see https://typeorm.io/custom-repository
 */
@Injectable()
export class UserRepository extends Repository<User> {
  // ...
}
```

**Interview Tip**: Explain that **@EntityRepository** was deprecated in TypeORM 0.3+ due to tight coupling with decorators, poor DI support, and maintenance burden. **Modern replacements**: **extend()** method (simple, functional approach for custom methods) or **class-based repositories** (better for complex logic, DI, testing). Emphasize benefits of new approach: better dependency injection (can inject services), easier testing (mockable dependencies), cleaner code (no decorator magic), better NestJS integration (@Injectable()), support for multiple DataSources. Mention migration: replace @EntityRepository with extend() or class extending Repository, replace connection.getCustomRepository() with direct instantiation or DI, inject DataSource instead of relying on decorators. For production: use extend() for simple custom methods, class-based for complex logic with dependencies, @Injectable() in NestJS, document migration for legacy codebases. A strong answer demonstrates understanding of why the change was made and the architectural improvements in TypeORM 0.3+.

</details>

<details>
<summary>59. How do you extend the Repository class?</summary>

**Extending Repository** allows you to add custom methods and business logic to entity repositories. In TypeORM 0.3+, there are two primary approaches: **extend()** method for simple extensions, and **class inheritance** for complex logic with dependency injection.

### Method 1: Using extend()

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

// Extend repository with custom methods
export const UserRepository = dataSource.getRepository(User).extend({
  // All standard Repository methods available (find, save, update, etc.)
  // Plus custom methods below:

  findByEmail(email: string): Promise<User | null> {
    return this.findOne({ where: { email } });
  },

  findActiveUsers(): Promise<User[]> {
    return this.find({
      where: { isActive: true },
      order: { createdAt: 'DESC' },
    });
  },

  async searchUsers(query: string): Promise<User[]> {
    return this.createQueryBuilder('user')
      .where('user.firstName ILIKE :query', { query: `%${query}%` })
      .orWhere('user.lastName ILIKE :query', { query: `%${query}%` })
      .orWhere('user.email ILIKE :query', { query: `%${query}%` })
      .getMany();
  },

  async countActiveUsers(): Promise<number> {
    return this.count({ where: { isActive: true } });
  },
});

// Usage - Has both standard and custom methods
const users = await UserRepository.find(); // Standard method
const activeUsers = await UserRepository.findActiveUsers(); // Custom method
const user = await UserRepository.findByEmail('john@example.com'); // Custom method
```

### Method 2: Class Inheritance

```typescript
import { DataSource, Repository } from 'typeorm';

export class UserRepository extends Repository<User> {
  constructor(private dataSource: DataSource) {
    super(User, dataSource.createEntityManager());
  }

  // Custom method: Find by email
  async findByEmail(email: string): Promise<User | null> {
    return this.findOne({ where: { email } });
  }

  // Custom method: Find active users
  async findActiveUsers(): Promise<User[]> {
    return this.find({
      where: { isActive: true },
      order: { createdAt: 'DESC' },
    });
  }

  // Custom method: Search users
  async searchUsers(query: string): Promise<User[]> {
    return this.createQueryBuilder('user')
      .where('user.firstName ILIKE :query', { query: `%${query}%` })
      .orWhere('user.lastName ILIKE :query', { query: `%${query}%` })
      .orWhere('user.email ILIKE :query', { query: `%${query}%` })
      .getMany();
  }

  // Custom method: Find or create
  async findOrCreate(data: Partial<User>): Promise<User> {
    let user = await this.findByEmail(data.email);

    if (!user) {
      user = this.create(data);
      await this.save(user);
    }

    return user;
  }

  // Custom method: Deactivate old users
  async deactivateOldUsers(days: number = 90): Promise<number> {
    const cutoffDate = new Date();
    cutoffDate.setDate(cutoffDate.getDate() - days);

    const result = await this.update(
      {
        lastLoginAt: LessThan(cutoffDate),
        isActive: true,
      },
      {
        isActive: false,
      }
    );

    return result.affected || 0;
  }
}

// Usage
const userRepository = new UserRepository(dataSource);

const users = await userRepository.find(); // Inherited method
const activeUsers = await userRepository.findActiveUsers(); // Custom method
const user = await userRepository.findOrCreate({ email: 'john@example.com' }); // Custom method
```

### Method 3: NestJS with Dependency Injection

```typescript
import { Injectable } from '@nestjs/common';
import { DataSource, Repository } from 'typeorm';

@Injectable()
export class UserRepository extends Repository<User> {
  constructor(private dataSource: DataSource) {
    super(User, dataSource.createEntityManager());
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.findOne({ where: { email } });
  }

  async findActiveUsers(): Promise<User[]> {
    return this.find({
      where: { isActive: true },
      order: { createdAt: 'DESC' },
    });
  }

  async searchUsers(query: string): Promise<User[]> {
    return this.createQueryBuilder('user')
      .where('user.firstName ILIKE :query', { query: `%${query}%` })
      .orWhere('user.lastName ILIKE :query', { query: `%${query}%` })
      .orWhere('user.email ILIKE :query', { query: `%${query}%` })
      .getMany();
  }

  async getUserWithPosts(userId: number): Promise<User | null> {
    return this.findOne({
      where: { id: userId },
      relations: ['posts'],
    });
  }
}

// user.module.ts
@Module({
  providers: [UserRepository, UserService],
  exports: [UserRepository],
})
export class UserModule {}

// user.service.ts
@Injectable()
export class UserService {
  constructor(private userRepository: UserRepository) {}

  async findByEmail(email: string): Promise<User> {
    const user = await this.userRepository.findByEmail(email);

    if (!user) {
      throw new NotFoundException('User not found');
    }

    return user;
  }

  async getActiveUsers(): Promise<User[]> {
    return this.userRepository.findActiveUsers();
  }

  async searchUsers(query: string): Promise<User[]> {
    return this.userRepository.searchUsers(query);
  }
}
```

### Advanced Extension with Dependencies

```typescript
import { Injectable } from '@nestjs/common';
import { DataSource, Repository } from 'typeorm';

@Injectable()
export class UserRepository extends Repository<User> {
  constructor(
    private dataSource: DataSource,
    private cacheService: CacheService,
    private logger: Logger
  ) {
    super(User, dataSource.createEntityManager());
  }

  // Custom method with caching
  async findByEmailWithCache(email: string): Promise<User | null> {
    const cacheKey = `user:email:${email}`;

    // Check cache first
    const cached = await this.cacheService.get<User>(cacheKey);
    if (cached) {
      this.logger.log(`Cache hit for user email: ${email}`);
      return cached;
    }

    // Query database
    const user = await this.findOne({ where: { email } });

    if (user) {
      // Cache result for 5 minutes
      await this.cacheService.set(cacheKey, user, 300);
      this.logger.log(`Cached user: ${user.id}`);
    }

    return user;
  }

  // Custom method with logging
  async createUserWithLog(data: Partial<User>): Promise<User> {
    this.logger.log(`Creating user: ${data.email}`);

    const user = this.create(data);
    const saved = await this.save(user);

    this.logger.log(`User created successfully: ${saved.id}`);

    return saved;
  }

  // Custom method with transaction
  async transferOwnership(
    fromUserId: number,
    toUserId: number
  ): Promise<void> {
    await this.dataSource.transaction(async (manager) => {
      const userRepo = manager.getRepository(User);
      const postRepo = manager.getRepository(Post);

      const fromUser = await userRepo.findOneByOrFail({ id: fromUserId });
      const toUser = await userRepo.findOneByOrFail({ id: toUserId });

      // Transfer all posts
      await postRepo.update({ authorId: fromUserId }, { authorId: toUserId });

      this.logger.log(
        `Transferred ownership from ${fromUser.email} to ${toUser.email}`
      );
    });
  }
}
```

### Factory Pattern for Multiple Entities

```typescript
// Generic repository factory
export function createCustomRepository<T>(
  entity: new () => T,
  dataSource: DataSource
) {
  return dataSource.getRepository(entity).extend({
    async findByIds(ids: number[]): Promise<T[]> {
      return this.findBy({ id: In(ids) } as any);
    },

    async findActive(): Promise<T[]> {
      return this.find({
        where: { isActive: true } as any,
      });
    },

    async softDeleteMany(ids: number[]): Promise<number> {
      const result = await this.softDelete(ids);
      return result.affected || 0;
    },

    async countActive(): Promise<number> {
      return this.count({
        where: { isActive: true } as any,
      });
    },
  });
}

// Usage with multiple entities
export const UserRepository = createCustomRepository(User, dataSource);
export const PostRepository = createCustomRepository(Post, dataSource);
export const CommentRepository = createCustomRepository(Comment, dataSource);

// All have the same custom methods
const activeUsers = await UserRepository.findActive();
const activePosts = await PostRepository.findActive();
const activeComments = await CommentRepository.findActive();
```

### Extending with Query Builders

```typescript
export class UserRepository extends Repository<User> {
  constructor(private dataSource: DataSource) {
    super(User, dataSource.createEntityManager());
  }

  // Complex query with joins
  async findUsersWithPostCount(): Promise<
    Array<User & { postCount: number }>
  > {
    const result = await this.createQueryBuilder('user')
      .leftJoin('user.posts', 'post')
      .select([
        'user.id',
        'user.firstName',
        'user.lastName',
        'user.email',
      ])
      .addSelect('COUNT(post.id)', 'postCount')
      .groupBy('user.id')
      .orderBy('postCount', 'DESC')
      .getRawAndEntities();

    return result.entities.map((user, index) => ({
      ...user,
      postCount: parseInt(result.raw[index].postCount) || 0,
    }));
  }

  // Complex search with multiple conditions
  async advancedSearch(filters: {
    query?: string;
    isActive?: boolean;
    minAge?: number;
    maxAge?: number;
    city?: string;
    hasProfile?: boolean;
  }): Promise<User[]> {
    let queryBuilder = this.createQueryBuilder('user');

    // Text search
    if (filters.query) {
      queryBuilder = queryBuilder.where(
        '(user.firstName ILIKE :query OR user.lastName ILIKE :query OR user.email ILIKE :query)',
        { query: `%${filters.query}%` }
      );
    }

    // Status filter
    if (filters.isActive !== undefined) {
      queryBuilder = queryBuilder.andWhere('user.isActive = :isActive', {
        isActive: filters.isActive,
      });
    }

    // Age range
    if (filters.minAge !== undefined || filters.maxAge !== undefined) {
      queryBuilder = queryBuilder.andWhere(
        'user.age BETWEEN :minAge AND :maxAge',
        {
          minAge: filters.minAge || 0,
          maxAge: filters.maxAge || 150,
        }
      );
    }

    // City filter
    if (filters.city) {
      queryBuilder = queryBuilder.andWhere('user.city = :city', {
        city: filters.city,
      });
    }

    // Profile filter
    if (filters.hasProfile !== undefined) {
      if (filters.hasProfile) {
        queryBuilder = queryBuilder
          .innerJoin('user.profile', 'profile')
          .andWhere('profile.id IS NOT NULL');
      } else {
        queryBuilder = queryBuilder
          .leftJoin('user.profile', 'profile')
          .andWhere('profile.id IS NULL');
      }
    }

    return queryBuilder.getMany();
  }

  // Pagination with advanced options
  async paginateWithFilters(options: {
    page?: number;
    limit?: number;
    where?: any;
    order?: any;
    relations?: string[];
  }): Promise<{
    data: User[];
    total: number;
    page: number;
    totalPages: number;
  }> {
    const page = options.page || 1;
    const limit = options.limit || 10;
    const skip = (page - 1) * limit;

    const [data, total] = await this.findAndCount({
      where: options.where,
      order: options.order || { createdAt: 'DESC' },
      relations: options.relations,
      skip,
      take: limit,
    });

    return {
      data,
      total,
      page,
      totalPages: Math.ceil(total / limit),
    };
  }
}
```

### Comparison: extend() vs Class Inheritance

```typescript
/*
╔═══════════════════════════╤═══════════════════════════╤═══════════════════════════╗
║ FEATURE                   │ extend()                  │ Class Inheritance         ║
╠═══════════════════════════╪═══════════════════════════╪═══════════════════════════╣
║ Syntax                    │ ✅ Simpler (functional)   │ ⚠️ More verbose (OOP)     ║
║ Dependency Injection      │ ❌ Limited                │ ✅ Full support           ║
║ Custom Dependencies       │ ❌ Not easy               │ ✅ Easy (constructor)     ║
║ Testing                   │ ⚠️ Harder to mock deps    │ ✅ Easy to mock           ║
║ NestJS Integration        │ ⚠️ Requires workarounds   │ ✅ Native @Injectable()   ║
║ Type Safety               │ ✅ Good                   │ ✅ Excellent              ║
║ Use Case                  │ Simple custom methods     │ Complex logic with DI     ║
║ Reusability               │ ✅ Easy (export function) │ ✅ Easy (inheritance)     ║
║ Multiple DataSources      │ ⚠️ Create new instance    │ ✅ Inject specific DS     ║
║ Learning Curve            │ ✅ Low                    │ ⚠️ Medium                 ║
╚═══════════════════════════╧═══════════════════════════╧═══════════════════════════╝

RECOMMENDATION:
- Use extend() for simple custom methods without dependencies
- Use class inheritance for complex logic with DI, caching, logging, etc.
- Use class with @Injectable() in NestJS applications
*/
```

### Best Practices

```typescript
// ✅ GOOD: Use extend() for simple custom methods
export const UserRepository = dataSource.getRepository(User).extend({
  findByEmail(email: string) {
    return this.findOne({ where: { email } });
  },
});

// ✅ GOOD: Use class inheritance for complex logic
export class UserRepository extends Repository<User> {
  constructor(
    private dataSource: DataSource,
    private cacheService: CacheService
  ) {
    super(User, dataSource.createEntityManager());
  }

  async findByEmailWithCache(email: string): Promise<User | null> {
    // Can use injected dependencies
  }
}

// ✅ GOOD: Keep repository methods focused on data access
async findActiveUsers(): Promise<User[]> {
  return this.find({ where: { isActive: true } });
}

// ❌ BAD: Don't mix business logic in repository
async registerUserAndSendEmail(data: CreateUserDto): Promise<User> {
  const user = await this.save(data);
  await emailService.sendWelcome(user.email); // Business logic!
  return user;
}

// ✅ GOOD: Return domain-specific types
async findUserWithStats(userId: number): Promise<UserWithStats | null> {
  const user = await this.findOne({
    where: { id: userId },
    relations: ['posts', 'comments'],
  });

  if (!user) return null;

  return {
    ...user,
    postCount: user.posts.length,
    commentCount: user.comments.length,
  };
}

// ✅ GOOD: Document custom methods
/**
 * Finds users by email domain (case-insensitive).
 * @param domain Email domain (e.g., 'company.com')
 * @returns Array of users with matching domain
 * @example
 * const users = await repository.findByEmailDomain('gmail.com');
 */
async findByEmailDomain(domain: string): Promise<User[]> {
  return this.createQueryBuilder('user')
    .where('user.email ILIKE :domain', { domain: `%@${domain}` })
    .getMany();
}

// ✅ GOOD: Use transactions for complex operations
async bulkOperationWithTransaction(
  userIds: number[],
  action: 'activate' | 'deactivate'
): Promise<void> {
  await this.dataSource.transaction(async (manager) => {
    const repo = manager.getRepository(User);
    await repo.update(userIds, {
      isActive: action === 'activate',
    });
  });
}

// ✅ GOOD: Provide both sync and async versions when appropriate
findActiveUsersSync(): FindOptionsWhere<User> {
  return { isActive: true };
}

async findActiveUsers(): Promise<User[]> {
  return this.find({ where: this.findActiveUsersSync() });
}
```

### Testing Extended Repositories

```typescript
describe('UserRepository', () => {
  let dataSource: DataSource;
  let userRepository: UserRepository;

  beforeAll(async () => {
    dataSource = await createTestDataSource();
    userRepository = new UserRepository(dataSource);
  });

  afterAll(async () => {
    await dataSource.destroy();
  });

  describe('findByEmail', () => {
    it('should find user by email', async () => {
      // Arrange
      const user = await userRepository.save({
        firstName: 'John',
        lastName: 'Doe',
        email: 'john@example.com',
      });

      // Act
      const found = await userRepository.findByEmail('john@example.com');

      // Assert
      expect(found).toBeDefined();
      expect(found.id).toBe(user.id);
      expect(found.email).toBe('john@example.com');
    });

    it('should return null if not found', async () => {
      const found = await userRepository.findByEmail('notfound@example.com');
      expect(found).toBeNull();
    });
  });

  describe('findActiveUsers', () => {
    it('should return only active users', async () => {
      await userRepository.save([
        { firstName: 'Active', email: 'active@test.com', isActive: true },
        { firstName: 'Inactive', email: 'inactive@test.com', isActive: false },
      ]);

      const active = await userRepository.findActiveUsers();

      expect(active.length).toBeGreaterThan(0);
      expect(active.every(u => u.isActive)).toBe(true);
    });
  });
});
```

**Interview Tip**: Explain that **extending Repository** is done via **extend()** method (simple, functional approach) or **class inheritance** (complex logic with DI). With **extend()**, call `dataSource.getRepository(Entity).extend({ customMethods })` - gets standard methods plus custom ones. With **class inheritance**, extend `Repository<Entity>`, pass DataSource to super() constructor, add custom methods - better for dependency injection, caching, logging. Emphasize **extend()** is simpler for basic custom queries, while **class-based** is better for complex logic, NestJS integration (@Injectable()), testing (mockable dependencies). Mention best practices: keep repositories focused on data access (no business logic), document custom methods, use transactions for complex operations, return domain-specific types. For production: use extend() for simple cases, class-based for DI needs, @Injectable() in NestJS, write unit tests for custom methods, inject services (cache, logger) when needed. A strong answer demonstrates understanding of when to use each approach and proper separation of concerns.

</details>

<details>
<summary>60. What is TreeRepository and when to use it?</summary>

**TreeRepository** is a specialized repository for managing **hierarchical tree structures** (parent-child relationships) in TypeORM. It provides methods for working with tree entities like **categories, comments, organizational charts, file systems**, and supports multiple tree patterns: **Adjacency List**, **Nested Set**, **Closure Table**, and **Materialized Path**.

### Tree Patterns Supported

```typescript
/*
╔═══════════════════════╤═══════════════════════╤═══════════════════════╤═══════════════════════╗
║ PATTERN               │ Storage               │ Query Performance     │ Update Performance    ║
╠═══════════════════════╪═══════════════════════╪═══════════════════════╪═══════════════════════╣
║ Adjacency List        │ ✅ Simple (parent_id) │ ⚠️ Slow (recursion)   │ ✅ Fast               ║
║ Nested Set            │ ⚠️ Complex (lft, rgt) │ ✅ Very fast          │ ❌ Slow (reindex)     ║
║ Closure Table         │ ❌ Extra table        │ ✅ Fast               │ ✅ Fast               ║
║ Materialized Path     │ ⚠️ Path string        │ ✅ Fast               │ ✅ Fast               ║
╚═══════════════════════╧═══════════════════════╧═══════════════════════╧═══════════════════════╝

RECOMMENDATION:
- Adjacency List: Simple trees, frequent updates, small depth
- Nested Set: Read-heavy, deep trees, rare updates
- Closure Table: Balanced read/write, complex queries
- Materialized Path: Good all-around, path-based queries
*/
```

### 1. Adjacency List (Most Common)

```typescript
import { Entity, PrimaryGeneratedColumn, Column, Tree, TreeChildren, TreeParent } from 'typeorm';

@Entity()
@Tree('adjacency-list')
export class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column({ nullable: true })
  description: string;

  // Parent reference
  @TreeParent()
  parent: Category;

  // Children collection
  @TreeChildren()
  children: Category[];
}

// Get TreeRepository
const categoryRepository = dataSource.getTreeRepository(Category);

// Create tree structure
const electronics = categoryRepository.create({
  name: 'Electronics',
});
await categoryRepository.save(electronics);

const computers = categoryRepository.create({
  name: 'Computers',
  parent: electronics,
});
await categoryRepository.save(computers);

const laptops = categoryRepository.create({
  name: 'Laptops',
  parent: computers,
});
await categoryRepository.save(laptops);

const desktops = categoryRepository.create({
  name: 'Desktops',
  parent: computers,
});
await categoryRepository.save(desktops);

/*
Tree Structure:
Electronics (1)
└── Computers (2)
    ├── Laptops (3)
    └── Desktops (4)
*/
```

### TreeRepository Methods

```typescript
// ===== findTrees() - Get entire tree structure =====
const trees = await categoryRepository.findTrees();
console.log(trees);
/*
[
  {
    id: 1,
    name: 'Electronics',
    children: [
      {
        id: 2,
        name: 'Computers',
        children: [
          { id: 3, name: 'Laptops', children: [] },
          { id: 4, name: 'Desktops', children: [] }
        ]
      }
    ]
  }
]
*/

// ===== findRoots() - Get root nodes only =====
const roots = await categoryRepository.findRoots();
// Returns: [{ id: 1, name: 'Electronics' }]

// ===== findDescendantsTree() - Get descendants as tree =====
const computers = await categoryRepository.findOneBy({ name: 'Computers' });
const descendantsTree = await categoryRepository.findDescendantsTree(computers);
console.log(descendantsTree);
/*
{
  id: 2,
  name: 'Computers',
  children: [
    { id: 3, name: 'Laptops', children: [] },
    { id: 4, name: 'Desktops', children: [] }
  ]
}
*/

// ===== findDescendants() - Get descendants as flat array =====
const descendants = await categoryRepository.findDescendants(computers);
// Returns: [Computers, Laptops, Desktops]

// ===== countDescendants() - Count all descendants =====
const count = await categoryRepository.countDescendants(computers);
// Returns: 3 (including itself)

// ===== findAncestorsTree() - Get ancestors as tree =====
const laptops = await categoryRepository.findOneBy({ name: 'Laptops' });
const ancestorsTree = await categoryRepository.findAncestorsTree(laptops);
console.log(ancestorsTree);
/*
{
  id: 1,
  name: 'Electronics',
  children: [
    {
      id: 2,
      name: 'Computers',
      children: [
        { id: 3, name: 'Laptops', children: [] }
      ]
    }
  ]
}
*/

// ===== findAncestors() - Get ancestors as flat array =====
const ancestors = await categoryRepository.findAncestors(laptops);
// Returns: [Laptops, Computers, Electronics]

// ===== countAncestors() - Count all ancestors =====
const ancestorCount = await categoryRepository.countAncestors(laptops);
// Returns: 3 (including itself)
```

### 2. Closure Table Pattern

```typescript
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

/*
Creates two tables:
1. category - Main entity table
2. category_closure - Stores all ancestor-descendant relationships

Advantages:
✅ Fast queries (JOIN on closure table)
✅ Fast updates (no recursive updates)
✅ Can query "all descendants" efficiently

Disadvantages:
❌ Extra storage (closure table)
❌ More complex schema
*/
```

### 3. Nested Set Pattern

```typescript
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

  // Nested set specific fields (auto-managed by TypeORM)
  @TreeLevelColumn()
  level: number;
}

/*
Uses left and right boundaries (lft, rgt) to represent hierarchy.

Example:
Electronics (lft: 1, rgt: 8)
└── Computers (lft: 2, rgt: 7)
    ├── Laptops (lft: 3, rgt: 4)
    └── Desktops (lft: 5, rgt: 6)

Advantages:
✅ Very fast reads (single query with WHERE lft BETWEEN)
✅ Easy to get subtrees

Disadvantages:
❌ Slow updates (must reindex lft/rgt values)
❌ Complex to understand
*/
```

### 4. Materialized Path Pattern

```typescript
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

/*
Stores full path as string: "1.2.3" (great-grandparent.grandparent.parent)

Advantages:
✅ Fast queries (path LIKE '1.2.%')
✅ Easy to understand
✅ Can query by path depth

Disadvantages:
⚠️ Path string storage overhead
⚠️ Limited path length
*/
```

### Real-World Example: Category Management

```typescript
@Entity()
@Tree('adjacency-list')
export class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column({ nullable: true })
  description: string;

  @Column({ nullable: true })
  slug: string;

  @Column({ default: true })
  isActive: boolean;

  @TreeParent()
  parent: Category;

  @TreeChildren()
  children: Category[];

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}

export class CategoryService {
  constructor(
    private categoryRepository: TreeRepository<Category>
  ) {}

  // Get full category tree
  async getCategoryTree(): Promise<Category[]> {
    return this.categoryRepository.findTrees();
  }

  // Get root categories
  async getRootCategories(): Promise<Category[]> {
    return this.categoryRepository.findRoots();
  }

  // Get category with all descendants
  async getCategoryWithDescendants(categoryId: number): Promise<Category | null> {
    const category = await this.categoryRepository.findOneBy({ id: categoryId });

    if (!category) return null;

    return this.categoryRepository.findDescendantsTree(category);
  }

  // Get breadcrumb for category
  async getBreadcrumb(categoryId: number): Promise<Category[]> {
    const category = await this.categoryRepository.findOneBy({ id: categoryId });

    if (!category) return [];

    const ancestors = await this.categoryRepository.findAncestors(category);

    // Reverse to get root -> leaf order
    return ancestors.reverse();
  }

  // Move category to new parent
  async moveCategory(
    categoryId: number,
    newParentId: number | null
  ): Promise<Category> {
    const category = await this.categoryRepository.findOneByOrFail({ id: categoryId });

    if (newParentId) {
      const newParent = await this.categoryRepository.findOneByOrFail({
        id: newParentId,
      });

      // Check for circular reference
      const descendants = await this.categoryRepository.findDescendants(category);
      if (descendants.some(d => d.id === newParentId)) {
        throw new Error('Cannot move category under its own descendant');
      }

      category.parent = newParent;
    } else {
      category.parent = null; // Move to root
    }

    return this.categoryRepository.save(category);
  }

  // Delete category and all descendants
  async deleteCategoryWithDescendants(categoryId: number): Promise<void> {
    const category = await this.categoryRepository.findOneByOrFail({ id: categoryId });

    // Get all descendants (including self)
    const descendants = await this.categoryRepository.findDescendants(category);

    // Delete all (in reverse order to respect foreign keys)
    await this.categoryRepository.remove(descendants.reverse());
  }

  // Check if category has children
  async hasChildren(categoryId: number): Promise<boolean> {
    const category = await this.categoryRepository.findOneByOrFail({ id: categoryId });
    const count = await this.categoryRepository.countDescendants(category);
    return count > 1; // > 1 because it includes itself
  }

  // Get category depth level
  async getCategoryDepth(categoryId: number): Promise<number> {
    const category = await this.categoryRepository.findOneByOrFail({ id: categoryId });
    const ancestors = await this.categoryRepository.findAncestors(category);
    return ancestors.length - 1; // -1 to exclude itself
  }

  // Get siblings
  async getSiblings(categoryId: number): Promise<Category[]> {
    const category = await this.categoryRepository.findOne({
      where: { id: categoryId },
      relations: ['parent'],
    });

    if (!category) return [];

    if (!category.parent) {
      // Root level - get all roots except this one
      const roots = await this.categoryRepository.findRoots();
      return roots.filter(c => c.id !== categoryId);
    }

    // Get parent's children except this one
    const parent = await this.categoryRepository.findDescendantsTree(
      category.parent
    );
    return parent.children.filter(c => c.id !== categoryId);
  }
}
```

### REST API Example (NestJS)

```typescript
@Controller('categories')
export class CategoryController {
  constructor(private categoryService: CategoryService) {}

  // GET /categories/tree
  @Get('tree')
  async getCategoryTree() {
    return this.categoryService.getCategoryTree();
  }

  // GET /categories/:id/descendants
  @Get(':id/descendants')
  async getDescendants(@Param('id') id: number) {
    return this.categoryService.getCategoryWithDescendants(id);
  }

  // GET /categories/:id/breadcrumb
  @Get(':id/breadcrumb')
  async getBreadcrumb(@Param('id') id: number) {
    return this.categoryService.getBreadcrumb(id);
  }

  // POST /categories/:id/move
  @Post(':id/move')
  async moveCategory(
    @Param('id') id: number,
    @Body() body: { newParentId: number | null }
  ) {
    return this.categoryService.moveCategory(id, body.newParentId);
  }

  // DELETE /categories/:id/with-descendants
  @Delete(':id/with-descendants')
  async deleteCategoryWithDescendants(@Param('id') id: number) {
    await this.categoryService.deleteCategoryWithDescendants(id);
    return { message: 'Category and descendants deleted' };
  }

  // GET /categories/:id/siblings
  @Get(':id/siblings')
  async getSiblings(@Param('id') id: number) {
    return this.categoryService.getSiblings(id);
  }
}
```

### Common Use Cases

```typescript
// 1. ✅ Category Trees (e-commerce)
@Tree('adjacency-list')
export class ProductCategory {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @TreeParent()
  parent: ProductCategory;

  @TreeChildren()
  children: ProductCategory[];

  @OneToMany(() => Product, product => product.category)
  products: Product[];
}

// 2. ✅ Threaded Comments
@Tree('closure-table')
export class Comment {
  @PrimaryGeneratedColumn()
  id: number;

  @Column('text')
  content: string;

  @ManyToOne(() => User)
  author: User;

  @TreeParent()
  parentComment: Comment;

  @TreeChildren()
  replies: Comment[];
}

// 3. ✅ Organizational Chart
@Tree('nested-set')
export class Employee {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  position: string;

  @TreeParent()
  manager: Employee;

  @TreeChildren()
  subordinates: Employee[];

  @TreeLevelColumn()
  level: number;
}

// 4. ✅ File System
@Tree('materialized-path')
export class FileSystemNode {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column({ type: 'enum', enum: ['file', 'folder'] })
  type: 'file' | 'folder';

  @TreeParent()
  parentFolder: FileSystemNode;

  @TreeChildren()
  children: FileSystemNode[];
}

// 5. ✅ Menu/Navigation
@Tree('adjacency-list')
export class MenuItem {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column({ nullable: true })
  url: string;

  @Column({ nullable: true })
  icon: string;

  @Column({ default: 0 })
  order: number;

  @TreeParent()
  parent: MenuItem;

  @TreeChildren({ cascade: true })
  children: MenuItem[];
}
```

### Best Practices

```typescript
// ✅ GOOD: Use appropriate tree pattern for your needs
// Adjacency List: Most common, frequent updates
@Tree('adjacency-list')

// Closure Table: Complex queries, balanced read/write
@Tree('closure-table')

// Nested Set: Read-heavy, rare updates
@Tree('nested-set')

// Materialized Path: Path-based queries
@Tree('materialized-path')

// ✅ GOOD: Add indexes for parent references
@Entity()
@Tree('adjacency-list')
export class Category {
  @TreeParent()
  @Index() // Index for faster parent lookups
  parent: Category;
}

// ✅ GOOD: Prevent circular references
async moveCategory(categoryId: number, newParentId: number): Promise<void> {
  const category = await this.categoryRepository.findOneByOrFail({ id: categoryId });
  const descendants = await this.categoryRepository.findDescendants(category);

  if (descendants.some(d => d.id === newParentId)) {
    throw new Error('Cannot create circular reference');
  }

  // Safe to move
}

// ✅ GOOD: Use transactions for complex tree operations
await dataSource.transaction(async (manager) => {
  const repo = manager.getTreeRepository(Category);

  const category = await repo.findOneByOrFail({ id: categoryId });
  const descendants = await repo.findDescendants(category);

  // Move all descendants
  for (const desc of descendants) {
    desc.parent = newParent;
    await repo.save(desc);
  }
});

// ✅ GOOD: Cache tree structures
async getCategoryTree(): Promise<Category[]> {
  const cached = await this.cacheService.get('category:tree');
  if (cached) return cached;

  const tree = await this.categoryRepository.findTrees();
  await this.cacheService.set('category:tree', tree, 3600); // 1 hour

  return tree;
}

// ❌ BAD: Loading entire tree without filtering
const tree = await categoryRepository.findTrees(); // Can be huge!

// ✅ GOOD: Filter inactive categories
const activeTrees = await categoryRepository.findTrees({
  where: { isActive: true },
});

// ❌ BAD: N+1 query problem
for (const category of categories) {
  const children = await categoryRepository.findDescendants(category);
}

// ✅ GOOD: Load tree structure once
const tree = await categoryRepository.findTrees();
// Navigate tree in memory

// ✅ GOOD: Add ordering to children
@TreeChildren({ cascade: true })
@OrderBy({ order: 'ASC', name: 'ASC' })
children: Category[];
```

**Interview Tip**: Explain that **TreeRepository** manages hierarchical parent-child relationships in databases, providing methods like **findTrees()** (full tree), **findDescendants()** (all children), **findAncestors()** (parent chain), **findRoots()** (top-level nodes). Mention **4 tree patterns**: **Adjacency List** (simple, parent_id column, frequent updates), **Closure Table** (extra table, fast queries), **Nested Set** (left/right boundaries, fast reads, slow updates), **Materialized Path** (path string, good all-around). Emphasize use cases: categories (e-commerce), threaded comments, org charts, file systems, navigation menus. Highlight methods: findDescendantsTree vs findDescendants (tree vs flat), countDescendants/countAncestors for counts, use @TreeParent and @TreeChildren decorators. For production: choose pattern based on read/write ratio, index parent columns, prevent circular references, cache tree structures, use transactions for complex operations, filter inactive nodes. A strong answer demonstrates understanding of when TreeRepository is appropriate vs standard Repository and tradeoffs between tree patterns.

</details>
