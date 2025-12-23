
## **Basics**

<details>
<summary>1. What is TypeORM and what are its main features?</summary>

### Answer:

**TypeORM** is an Object-Relational Mapping (ORM) library for TypeScript and JavaScript that runs on Node.js, Browser, React Native, and other platforms. It helps developers work with databases using object-oriented programming paradigms.

### Main Features:

**1. Multiple Database Support**
- MySQL, PostgreSQL, MariaDB, SQLite, MS SQL Server, Oracle, MongoDB
- Can switch databases with minimal code changes

**2. Entity-Based Data Modeling**
```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column({ unique: true })
  email: string;
}
```

**3. Two Design Patterns**
- **Active Record**: Entities have methods to save/remove themselves
- **Data Mapper**: Repository handles all database operations (recommended for large apps)

**4. Powerful Query Builder**
```typescript
const users = await dataSource
  .getRepository(User)
  .createQueryBuilder('user')
  .where('user.age > :age', { age: 18 })
  .getMany();
```

**5. Relations & Eager/Lazy Loading**
- One-to-One, One-to-Many, Many-to-One, Many-to-Many
- Control when related data is loaded

**6. Migration System**
- Version control for database schema
- Automatic migration generation from entity changes
- Safe production deployments

**7. Transaction Support**
```typescript
await dataSource.transaction(async (manager) => {
  await manager.save(user);
  await manager.save(profile);
});
```

**8. Type Safety**
- Full TypeScript support
- Compile-time type checking
- IntelliSense/autocomplete

**9. Other Features**
- Connection pooling
- Query caching
- Subscribers and listeners for entity lifecycle events
- Soft deletes
- Tree entities (nested sets, closure table)
- Database replication support

### Production Benefits:
- **Maintainability**: Clean, readable code
- **Security**: SQL injection prevention with parameterized queries
- **Performance**: Built-in caching and query optimization
- **Scalability**: Connection pooling and replication support

**Interview Tip**: Emphasize that TypeORM bridges the gap between object-oriented code and relational databases, making it easier to work with complex data models while maintaining type safety.

</details>

<details>
<summary>2. How does TypeORM differ from other ORMs like Sequelize or Prisma?</summary>

### Answer:

### TypeORM vs Sequelize vs Prisma Comparison:

**1. Language & Type Safety**

**TypeORM:**
```typescript
// Full TypeScript support with decorators
@Entity()
class User {
  @PrimaryGeneratedColumn()
  id: number;
  
  @Column()
  name: string;
}
// Compile-time type checking
```

**Sequelize:**
```typescript
// JavaScript-first, TypeScript support via separate package
// Types not as tight, more runtime-focused
const User = sequelize.define('User', {
  id: { type: DataTypes.INTEGER, primaryKey: true },
  name: { type: DataTypes.STRING }
});
```

**Prisma:**
```prisma
// Schema-first approach with its own syntax
model User {
  id   Int    @id @default(autoincrement())
  name String
}
// Generates fully typed TypeScript client
```

**2. Pattern & Architecture**

**TypeORM:**
- Active Record OR Data Mapper pattern (your choice)
- Decorator-based entity definition
- More OOP-focused

**Sequelize:**
- Active Record pattern only
- Model-based approach
- More traditional ORM feel

**Prisma:**
- Data Mapper pattern only
- Schema-first, auto-generated client
- Modern, opinionated approach

**3. Query API**

**TypeORM:**
```typescript
// Multiple ways to query
// 1. Repository methods
await userRepo.find({ where: { age: 25 } });

// 2. Query Builder (powerful, flexible)
await userRepo
  .createQueryBuilder('user')
  .where('user.age > :age', { age: 18 })
  .orderBy('user.name')
  .getMany();

// 3. Active Record (if using that pattern)
await User.find({ where: { age: 25 } });
```

**Sequelize:**
```typescript
// Model-based queries
await User.findAll({
  where: { age: 25 },
  order: [['name', 'ASC']]
});
```

**Prisma:**
```typescript
// Clean, fluent API
await prisma.user.findMany({
  where: { age: 25 },
  orderBy: { name: 'asc' }
});
```

**4. Migrations**

**TypeORM:**
```bash
# Generate migration from entity changes
typeorm migration:generate -n CreateUser

# Run migrations
typeorm migration:run
```
- Can auto-generate from entity changes
- Manual migrations also supported

**Sequelize:**
```bash
# Manual migration creation
npx sequelize-cli migration:generate --name create-user
```
- Primarily manual migrations
- CLI-based workflow

**Prisma:**
```bash
# Schema changes tracked automatically
prisma migrate dev --name create-user
```
- Schema is single source of truth
- Migrations auto-generated from schema changes

**5. Relations Handling**

**TypeORM:**
```typescript
@Entity()
class User {
  @OneToMany(() => Post, post => post.user)
  posts: Post[];
}

// Explicit, decorator-based
// Full control over lazy/eager loading
```

**Sequelize:**
```typescript
User.hasMany(Post, { foreignKey: 'userId' });
// Association-based
// Defined separately from model
```

**Prisma:**
```prisma
model User {
  id    Int    @id
  posts Post[]
}
// Relation syntax in schema
// Auto-managed
```

**6. Performance & Maturity**

| Feature | TypeORM | Sequelize | Prisma |
|---------|---------|-----------|--------|
| Maturity | Mature (2016) | Most mature (2011) | Newer (2019) |
| Community | Large | Largest | Growing fast |
| Performance | Good | Good | Excellent |
| Bundle Size | Larger | Larger | Smaller |
| Raw SQL Support | Yes | Yes | Limited |

**7. When to Choose Each:**

**Choose TypeORM if:**
- You prefer decorator-based syntax
- You want flexibility (Active Record or Data Mapper)
- You need powerful QueryBuilder
- You're building large enterprise apps
- You need advanced features (tree entities, replication)

**Choose Sequelize if:**
- You're working with legacy JavaScript projects
- You want the most mature ORM
- You need extensive community resources
- You prefer traditional ORM patterns

**Choose Prisma if:**
- You want the best TypeScript experience
- You prefer schema-first development
- You want simpler, cleaner API
- You're starting a new project
- Performance is critical

**Interview Tip**: Mention that the choice depends on project requirements, team expertise, and architectural preferences. TypeORM offers more flexibility and control, Sequelize is battle-tested, and Prisma provides the most modern DX.

</details>

<details>
<summary>3. What databases are supported by TypeORM?</summary>

### Answer:

TypeORM supports a wide range of **SQL and NoSQL databases**, making it a versatile choice for different project needs.

### Supported Databases:

**1. MySQL / MariaDB**
```typescript
// Connection configuration
const dataSource = new DataSource({
  type: 'mysql', // or 'mariadb'
  host: 'localhost',
  port: 3306,
  username: 'root',
  password: 'password',
  database: 'mydb',
  entities: [User],
  synchronize: false // Always false in production
});
```
**Use Cases:**
- Web applications
- E-commerce platforms
- Content management systems
- Most common for general-purpose apps

**2. PostgreSQL**
```typescript
const dataSource = new DataSource({
  type: 'postgres',
  host: 'localhost',
  port: 5432,
  username: 'postgres',
  password: 'password',
  database: 'mydb',
  entities: [User],
  // PostgreSQL-specific features
  extra: {
    ssl: true, // For production
    connectionTimeoutMillis: 5000
  }
});
```
**Features:**
- Advanced data types (JSON, Arrays, UUID)
- Full-text search
- Transactions with ACID compliance
- Best for complex queries and data integrity

**3. SQLite**
```typescript
const dataSource = new DataSource({
  type: 'sqlite',
  database: './mydb.sqlite',
  entities: [User],
  synchronize: true // OK for development with SQLite
});
```
**Use Cases:**
- Mobile apps (React Native)
- Desktop applications (Electron)
- Testing and development
- Small embedded systems
- Prototyping

**4. Microsoft SQL Server**
```typescript
const dataSource = new DataSource({
  type: 'mssql',
  host: 'localhost',
  port: 1433,
  username: 'sa',
  password: 'password',
  database: 'mydb',
  entities: [User],
  options: {
    encrypt: true, // Use encryption
    trustServerCertificate: true // For development
  }
});
```
**Use Cases:**
- Enterprise applications
- Microsoft ecosystem integration
- Legacy system modernization

**5. Oracle**
```typescript
const dataSource = new DataSource({
  type: 'oracle',
  host: 'localhost',
  port: 1521,
  username: 'system',
  password: 'password',
  sid: 'xe', // or serviceName
  entities: [User]
});
```
**Use Cases:**
- Large enterprise systems
- Banking and financial applications
- High-performance requirements

**6. MongoDB (NoSQL)**
```typescript
import { Entity, ObjectIdColumn, ObjectId, Column } from 'typeorm';

@Entity()
class User {
  @ObjectIdColumn()
  id: ObjectId;

  @Column()
  name: string;

  @Column()
  email: string;
}

const dataSource = new DataSource({
  type: 'mongodb',
  host: 'localhost',
  port: 27017,
  database: 'mydb',
  entities: [User],
  useUnifiedTopology: true
});
```
**Note:** MongoDB support is more limited compared to SQL databases
**Use Cases:**
- Document-based storage
- Flexible schemas
- Real-time applications

**7. CockroachDB**
```typescript
const dataSource = new DataSource({
  type: 'cockroachdb',
  host: 'localhost',
  port: 26257,
  username: 'root',
  password: '',
  database: 'mydb',
  entities: [User],
  ssl: true
});
```
**Use Cases:**
- Distributed systems
- Cloud-native applications
- High availability requirements

**8. SAP Hana**
```typescript
const dataSource = new DataSource({
  type: 'sap',
  host: 'localhost',
  port: 30015,
  username: 'SYSTEM',
  password: 'password',
  database: 'mydb',
  entities: [User]
});
```

**9. Other Databases:**
- **Better-SQLite3**: Faster synchronous SQLite
- **Cordova/PhoneGap**: SQLite for mobile
- **SQL.js**: SQLite for browsers
- **React Native**: SQLite for mobile apps

### Database-Specific Features:

**PostgreSQL Only:**
```typescript
@Entity()
class User {
  // UUID type
  @Column('uuid')
  id: string;

  // Array column
  @Column('text', { array: true })
  tags: string[];

  // JSON column with better support
  @Column('jsonb')
  metadata: object;
}
```

**MySQL Only:**
```typescript
@Entity()
class User {
  // Spatial data types
  @Column('point')
  location: string;

  // Full-text index
  @Index({ fulltext: true })
  @Column('text')
  description: string;
}
```

### Production Considerations:

**1. Connection Pooling (All SQL Databases)**
```typescript
const dataSource = new DataSource({
  type: 'postgres',
  // ... other options
  poolSize: 10, // Maximum number of connections
  extra: {
    connectionTimeoutMillis: 5000,
    idleTimeoutMillis: 30000
  }
});
```

**2. SSL/TLS for Production**
```typescript
const dataSource = new DataSource({
  type: 'postgres',
  ssl: {
    rejectUnauthorized: true,
    ca: fs.readFileSync('./ca-certificate.crt').toString()
  }
});
```

**3. Replication Support**
```typescript
const dataSource = new DataSource({
  type: 'mysql',
  replication: {
    master: {
      host: 'master.example.com',
      port: 3306,
      username: 'root',
      password: 'password',
      database: 'mydb'
    },
    slaves: [
      {
        host: 'slave1.example.com',
        port: 3306,
        username: 'root',
        password: 'password',
        database: 'mydb'
      }
    ]
  }
});
```

**Interview Tip**: Mention that TypeORM's multi-database support allows for flexibility in choosing the right database for specific project needs. Emphasize that while the entity code remains mostly the same, each database has unique features that TypeORM can leverage.

</details>

<details>
<summary>4. What is the Active Record pattern vs Repository pattern in TypeORM?</summary>

### Answer:

TypeORM supports **two design patterns** for managing database operations: **Active Record** and **Data Mapper (Repository)** pattern. Understanding both is crucial for architecting scalable applications.

### Active Record Pattern

**Definition:** Entities contain methods to save, remove, and load themselves. The model instance is aware of the database.

**Implementation:**
```typescript
import { BaseEntity, Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

// Entity extends BaseEntity to use Active Record pattern
@Entity()
export class User extends BaseEntity {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column({ unique: true })
  email: string;

  @Column()
  age: number;

  // Custom instance method
  async isAdult(): Promise<boolean> {
    return this.age >= 18;
  }
}

// Usage - entities manage themselves
async function activeRecordExample() {
  // Create and save
  const user = new User();
  user.name = 'John Doe';
  user.email = 'john@example.com';
  user.age = 25;
  await user.save(); // Entity saves itself

  // Find
  const users = await User.find(); // Static method on entity
  const john = await User.findOne({ where: { email: 'john@example.com' } });

  // Update
  john.age = 26;
  await john.save(); // Entity updates itself

  // Delete
  await john.remove(); // Entity removes itself

  // Query builder on entity
  const adults = await User.createQueryBuilder('user')
    .where('user.age >= :age', { age: 18 })
    .getMany();
}
```

**Pros:**
- ✅ Simple and intuitive for small projects
- ✅ Less boilerplate code
- ✅ Quick to prototype
- ✅ Entity-centric thinking

**Cons:**
- ❌ Tight coupling between entity and database logic
- ❌ Hard to test (entities depend on database)
- ❌ Not suitable for complex business logic
- ❌ Difficult to maintain in large applications

### Data Mapper (Repository) Pattern

**Definition:** Entities are simple data containers. Repositories handle all database operations. Clear separation of concerns.

**Implementation:**
```typescript
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

// Entity is just a data container (no BaseEntity)
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column({ unique: true })
  email: string;

  @Column()
  age: number;

  // Only business logic, no database operations
  isAdult(): boolean {
    return this.age >= 18;
  }
}

// Repository handles all database operations
import { AppDataSource } from './data-source';

async function repositoryExample() {
  // Get repository
  const userRepository = AppDataSource.getRepository(User);

  // Create and save
  const user = new User();
  user.name = 'John Doe';
  user.email = 'john@example.com';
  user.age = 25;
  await userRepository.save(user); // Repository saves entity

  // Find
  const users = await userRepository.find();
  const john = await userRepository.findOne({ 
    where: { email: 'john@example.com' } 
  });

  // Update
  john.age = 26;
  await userRepository.save(john);

  // Delete
  await userRepository.remove(john);

  // Query builder through repository
  const adults = await userRepository
    .createQueryBuilder('user')
    .where('user.age >= :age', { age: 18 })
    .getMany();
}
```

**Custom Repository (Recommended for Production):**
```typescript
// user.repository.ts
import { Repository } from 'typeorm';
import { AppDataSource } from './data-source';
import { User } from './user.entity';

export class UserRepository extends Repository<User> {
  // Custom method for business logic
  async findAdults(): Promise<User[]> {
    return this.createQueryBuilder('user')
      .where('user.age >= :age', { age: 18 })
      .getMany();
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.findOne({ where: { email } });
  }

  async createUser(userData: Partial<User>): Promise<User> {
    const user = this.create(userData);
    return this.save(user);
  }

  async updateUserAge(id: number, age: number): Promise<void> {
    await this.update(id, { age });
  }
}

// Initialize custom repository
export const userRepository = AppDataSource.getRepository(User).extend({
  // Add custom methods
  async findAdults(): Promise<User[]> {
    return this.createQueryBuilder('user')
      .where('user.age >= :age', { age: 18 })
      .getMany();
  }
});

// Usage
const adults = await userRepository.findAdults();
```

**Pros:**
- ✅ Clean separation of concerns
- ✅ Easy to test (mock repositories)
- ✅ Scalable for large applications
- ✅ Better for complex business logic
- ✅ Entities are framework-independent
- ✅ Easier to maintain

**Cons:**
- ❌ More boilerplate code
- ❌ Steeper learning curve
- ❌ Requires more setup

### Side-by-Side Comparison:

| Aspect | Active Record | Repository (Data Mapper) |
|--------|--------------|-------------------------|
| **Coupling** | High (entity knows DB) | Low (separation) |
| **Testing** | Hard (DB dependency) | Easy (mock repos) |
| **Complexity** | Simple | More complex |
| **Scalability** | Small projects | Large projects |
| **Code** | Less boilerplate | More boilerplate |
| **Best For** | Prototypes, small apps | Enterprise apps |

### Production Best Practices:

**1. Service Layer with Repository Pattern:**
```typescript
// user.service.ts
import { injectable } from 'tsyringe';
import { UserRepository } from './user.repository';
import { User } from './user.entity';

@injectable()
export class UserService {
  constructor(private userRepository: UserRepository) {}

  async registerUser(email: string, name: string, age: number): Promise<User> {
    // Business logic
    const existingUser = await this.userRepository.findByEmail(email);
    if (existingUser) {
      throw new Error('Email already exists');
    }

    // Validation
    if (age < 0) {
      throw new Error('Invalid age');
    }

    // Create user
    return this.userRepository.createUser({ email, name, age });
  }

  async getAdultUsers(): Promise<User[]> {
    return this.userRepository.findAdults();
  }
}
```

**2. Testing with Repository Pattern:**
```typescript
// user.service.spec.ts
import { UserService } from './user.service';
import { UserRepository } from './user.repository';

describe('UserService', () => {
  it('should register a new user', async () => {
    // Mock repository
    const mockRepository = {
      findByEmail: jest.fn().mockResolvedValue(null),
      createUser: jest.fn().mockResolvedValue({ id: 1, email: 'test@example.com' })
    };

    const service = new UserService(mockRepository as any);
    const user = await service.registerUser('test@example.com', 'Test', 25);

    expect(user.id).toBe(1);
    expect(mockRepository.createUser).toHaveBeenCalled();
  });
});
```

### When to Use Each:

**Use Active Record when:**
- Building small applications or MVPs
- Rapid prototyping
- Simple CRUD operations
- Team is new to TypeORM
- Limited business logic

**Use Repository Pattern when:**
- Building enterprise applications
- Complex business logic
- Need testability
- Multiple developers
- Long-term maintenance
- Microservices architecture

**Interview Tip**: Emphasize that Repository pattern is generally preferred for production applications due to better testability, maintainability, and separation of concerns. Active Record is fine for learning or small projects. Most enterprise codebases use Repository pattern with a service layer.

</details>

<details>
<summary>5. How do you initialize a TypeORM connection?</summary>

### Answer:

TypeORM connection initialization has evolved from v0.2 to v0.3. The modern approach uses **DataSource** (v0.3+), while older versions used **createConnection**.

### Modern Approach: DataSource (TypeORM v0.3+)

**1. Basic DataSource Configuration:**
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
  
  // Entities
  entities: [User, Post],
  // Or glob pattern: entities: ['src/entities/**/*.ts']
  
  // Migrations
  migrations: ['src/migrations/**/*.ts'],
  
  // Subscribers (for entity lifecycle events)
  subscribers: ['src/subscribers/**/*.ts'],
  
  // Auto-create tables (NEVER use in production)
  synchronize: false,
  
  // Logging
  logging: ['error', 'warn'],
  
  // Connection pool
  poolSize: 10,
  
  // Additional options
  extra: {
    connectionTimeoutMillis: 5000,
  }
});

// Initialize connection
async function initialize() {
  try {
    await AppDataSource.initialize();
    console.log('Database connected successfully');
  } catch (error) {
    console.error('Database connection failed:', error);
    process.exit(1);
  }
}

initialize();
```

**2. Production-Grade Configuration with Environment Variables:**
```typescript
// data-source.ts
import { DataSource, DataSourceOptions } from 'typeorm';
import * as dotenv from 'dotenv';

dotenv.config();

const isProduction = process.env.NODE_ENV === 'production';

const config: DataSourceOptions = {
  type: 'postgres',
  host: process.env.DB_HOST || 'localhost',
  port: parseInt(process.env.DB_PORT || '5432'),
  username: process.env.DB_USERNAME || 'postgres',
  password: process.env.DB_PASSWORD || 'password',
  database: process.env.DB_NAME || 'mydb',
  
  // Dynamic entity loading
  entities: [__dirname + '/entities/**/*.{ts,js}'],
  migrations: [__dirname + '/migrations/**/*.{ts,js}'],
  
  // Production settings
  synchronize: false, // NEVER true in production
  logging: isProduction ? ['error'] : ['query', 'error'],
  
  // Connection pooling (important for production)
  poolSize: parseInt(process.env.DB_POOL_SIZE || '10'),
  
  // SSL for production
  ssl: isProduction ? {
    rejectUnauthorized: true,
    ca: process.env.DB_SSL_CA,
  } : false,
  
  // Timeouts
  extra: {
    connectionTimeoutMillis: 5000,
    idleTimeoutMillis: 30000,
    max: 20, // Maximum pool size
  },
  
  // Cache settings
  cache: isProduction ? {
    type: 'redis',
    options: {
      host: process.env.REDIS_HOST || 'localhost',
      port: parseInt(process.env.REDIS_PORT || '6379'),
    },
    duration: 30000, // 30 seconds
  } : false,
};

export const AppDataSource = new DataSource(config);

// Initialize with error handling
export async function initializeDatabase(): Promise<void> {
  try {
    if (!AppDataSource.isInitialized) {
      await AppDataSource.initialize();
      console.log('✅ Database connected successfully');
      
      // Run migrations in production
      if (isProduction) {
        await AppDataSource.runMigrations();
        console.log('✅ Migrations executed');
      }
    }
  } catch (error) {
    console.error('❌ Database connection failed:', error);
    throw error;
  }
}

// Graceful shutdown
export async function closeDatabase(): Promise<void> {
  try {
    if (AppDataSource.isInitialized) {
      await AppDataSource.destroy();
      console.log('Database connection closed');
    }
  } catch (error) {
    console.error('Error closing database:', error);
  }
}
```

**3. Express.js Application Integration:**
```typescript
// app.ts
import express from 'express';
import { AppDataSource, initializeDatabase, closeDatabase } from './data-source';
import userRoutes from './routes/user.routes';

const app = express();
const PORT = process.env.PORT || 3000;

app.use(express.json());

// Initialize database before starting server
async function startServer() {
  try {
    // Initialize database connection
    await initializeDatabase();
    
    // Setup routes
    app.use('/api/users', userRoutes);
    
    // Start server
    const server = app.listen(PORT, () => {
      console.log(`Server running on port ${PORT}`);
    });
    
    // Graceful shutdown
    process.on('SIGTERM', async () => {
      console.log('SIGTERM received, closing server...');
      server.close(async () => {
        await closeDatabase();
        process.exit(0);
      });
    });
    
    process.on('SIGINT', async () => {
      console.log('SIGINT received, closing server...');
      server.close(async () => {
        await closeDatabase();
        process.exit(0);
      });
    });
    
  } catch (error) {
    console.error('Failed to start server:', error);
    process.exit(1);
  }
}

startServer();
```

**4. NestJS Integration:**
```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { User } from './entities/User';

@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),
    TypeOrmModule.forRootAsync({
      inject: [ConfigService],
      useFactory: (configService: ConfigService) => ({
        type: 'postgres',
        host: configService.get('DB_HOST'),
        port: configService.get('DB_PORT'),
        username: configService.get('DB_USERNAME'),
        password: configService.get('DB_PASSWORD'),
        database: configService.get('DB_NAME'),
        entities: [User],
        synchronize: false,
        logging: configService.get('NODE_ENV') !== 'production',
      }),
    }),
    TypeOrmModule.forFeature([User]),
  ],
})
export class AppModule {}
```

**5. Multiple Database Connections:**
```typescript
// data-source.ts
import { DataSource } from 'typeorm';

// Primary database (PostgreSQL)
export const PrimaryDataSource = new DataSource({
  name: 'default',
  type: 'postgres',
  host: 'localhost',
  port: 5432,
  username: 'postgres',
  password: 'password',
  database: 'primary_db',
  entities: [__dirname + '/entities/primary/**/*.{ts,js}'],
  synchronize: false,
});

// Secondary database (MySQL for analytics)
export const AnalyticsDataSource = new DataSource({
  name: 'analytics',
  type: 'mysql',
  host: 'localhost',
  port: 3306,
  username: 'root',
  password: 'password',
  database: 'analytics_db',
  entities: [__dirname + '/entities/analytics/**/*.{ts,js}'],
  synchronize: false,
});

// Initialize both
export async function initializeAllDatabases() {
  await Promise.all([
    PrimaryDataSource.initialize(),
    AnalyticsDataSource.initialize(),
  ]);
  console.log('All databases connected');
}

// Usage
const userRepo = PrimaryDataSource.getRepository(User);
const analyticsRepo = AnalyticsDataSource.getRepository(AnalyticsEvent);
```

**6. Connection Health Check:**
```typescript
// health.service.ts
export class HealthService {
  async checkDatabaseHealth(): Promise<boolean> {
    try {
      if (!AppDataSource.isInitialized) {
        return false;
      }
      
      // Execute simple query to check connection
      await AppDataSource.query('SELECT 1');
      return true;
    } catch (error) {
      console.error('Database health check failed:', error);
      return false;
    }
  }
  
  async getDatabaseInfo() {
    return {
      isConnected: AppDataSource.isInitialized,
      driver: AppDataSource.driver.options.type,
      database: AppDataSource.driver.database,
      poolSize: AppDataSource.driver.options.poolSize,
    };
  }
}
```

### Environment Variables (.env file):
```env
# Database
DB_HOST=localhost
DB_PORT=5432
DB_USERNAME=postgres
DB_PASSWORD=your_secure_password
DB_NAME=mydb
DB_POOL_SIZE=10

# SSL Certificate (for production)
DB_SSL_CA=/path/to/ca-certificate.crt

# Redis (for caching)
REDIS_HOST=localhost
REDIS_PORT=6379

# Application
NODE_ENV=production
PORT=3000
```

### Best Practices for Production:

1. **Never use `synchronize: true` in production** - use migrations instead
2. **Use environment variables** for all sensitive data
3. **Enable SSL/TLS** for database connections in production
4. **Implement connection pooling** to handle concurrent requests
5. **Set appropriate timeouts** to prevent hanging connections
6. **Use logging selectively** (errors only in production)
7. **Implement graceful shutdown** to close connections properly
8. **Health checks** to monitor database connectivity
9. **Use caching (Redis)** for frequently accessed data
10. **Connection retry logic** for handling temporary failures

**Interview Tip**: Emphasize that proper database initialization is critical for application reliability. Mention the importance of connection pooling, graceful shutdown, environment-based configuration, and never using `synchronize: true` in production.

</details>

<details>
<summary>6. What is a DataSource in TypeORM and how do you configure it?</summary>

**DataSource** is the modern way (introduced in TypeORM 0.3+) to establish and manage database connections. It replaces the deprecated `createConnection()` method and provides better control over database connections.

### What is DataSource?

A `DataSource` holds your database connection configuration and establishes the actual connection. It serves as the central entry point for all database operations.

```typescript
import { DataSource } from 'typeorm';

// Create a DataSource instance
const AppDataSource = new DataSource({
  type: 'postgres',
  host: 'localhost',
  port: 5432,
  username: 'postgres',
  password: 'password',
  database: 'mydb',
  entities: [User, Post, Comment],
  synchronize: false,
  logging: true,
});

// Initialize the connection
AppDataSource.initialize()
  .then(() => {
    console.log('Data Source has been initialized!');
  })
  .catch((err) => {
    console.error('Error during Data Source initialization:', err);
  });
```

### Basic Configuration Options

```typescript
import { DataSource } from 'typeorm';
import { User } from './entities/User';
import { Post } from './entities/Post';

export const AppDataSource = new DataSource({
  // Database type
  type: 'postgres',
  
  // Connection details
  host: 'localhost',
  port: 5432,
  username: 'myuser',
  password: 'mypassword',
  database: 'mydatabase',
  
  // Entities
  entities: [User, Post],
  // Or load from directory
  // entities: ['src/entities/**/*.ts'],
  
  // Migrations
  migrations: ['src/migrations/**/*.ts'],
  
  // Subscribers (event listeners)
  subscribers: ['src/subscribers/**/*.ts'],
  
  // Synchronize schema (only for development)
  synchronize: false,
  
  // Logging
  logging: ['query', 'error', 'schema'],
  
  // Connection pool
  extra: {
    max: 10,
    min: 2,
  },
});
```

### Environment-Based Configuration

```typescript
import { DataSource } from 'typeorm';
import * as dotenv from 'dotenv';

dotenv.config();

export const AppDataSource = new DataSource({
  type: process.env.DB_TYPE as any,
  host: process.env.DB_HOST,
  port: parseInt(process.env.DB_PORT || '5432'),
  username: process.env.DB_USERNAME,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  
  entities: [__dirname + '/entities/**/*.{ts,js}'],
  migrations: [__dirname + '/migrations/**/*.{ts,js}'],
  
  synchronize: process.env.NODE_ENV === 'development',
  logging: process.env.NODE_ENV === 'development',
  
  ssl: process.env.NODE_ENV === 'production' ? {
    rejectUnauthorized: false,
  } : false,
  
  extra: {
    max: parseInt(process.env.DB_POOL_MAX || '10'),
    min: parseInt(process.env.DB_POOL_MIN || '2'),
    connectionTimeoutMillis: parseInt(process.env.DB_TIMEOUT || '10000'),
  },
});

// .env file example
/*
DB_TYPE=postgres
DB_HOST=localhost
DB_PORT=5432
DB_USERNAME=postgres
DB_PASSWORD=secretpassword
DB_NAME=mydb
DB_POOL_MAX=20
DB_POOL_MIN=5
DB_TIMEOUT=10000
NODE_ENV=development
*/
```

### Application Integration

```typescript
import express from 'express';
import { AppDataSource } from './data-source';

const app = express();

// Initialize database and start server
AppDataSource.initialize()
  .then(() => {
    console.log('✅ Database connected successfully');
    
    // Start server after successful connection
    app.listen(3000, () => {
      console.log('🚀 Server running on port 3000');
    });
  })
  .catch((error) => {
    console.error('❌ Database connection failed:', error);
    process.exit(1);
  });

// Graceful shutdown
process.on('SIGINT', async () => {
  await AppDataSource.destroy();
  console.log('Database connection closed');
  process.exit(0);
});
```

### Using DataSource in Services

```typescript
import { AppDataSource } from './data-source';
import { User } from './entities/User';

// Access repositories
const userRepository = AppDataSource.getRepository(User);

// Perform operations
async function createUser(name: string, email: string) {
  const user = userRepository.create({ name, email });
  await userRepository.save(user);
  return user;
}

// Using query runner for complex operations
async function complexOperation() {
  const queryRunner = AppDataSource.createQueryRunner();
  
  await queryRunner.connect();
  await queryRunner.startTransaction();
  
  try {
    await queryRunner.manager.save(User, { name: 'John' });
    await queryRunner.commitTransaction();
  } catch (err) {
    await queryRunner.rollbackTransaction();
    throw err;
  } finally {
    await queryRunner.release();
  }
}
```

### Multiple DataSources

```typescript
// User database
export const UserDataSource = new DataSource({
  name: 'userDb',
  type: 'postgres',
  host: 'localhost',
  port: 5432,
  database: 'users',
  entities: [User],
  synchronize: false,
});

// Products database
export const ProductDataSource = new DataSource({
  name: 'productDb',
  type: 'mysql',
  host: 'localhost',
  port: 3306,
  database: 'products',
  entities: [Product],
  synchronize: false,
});

// Initialize both
await Promise.all([
  UserDataSource.initialize(),
  ProductDataSource.initialize(),
]);

// Use them
const userRepo = UserDataSource.getRepository(User);
const productRepo = ProductDataSource.getRepository(Product);
```

### DataSource for Testing

```typescript
import { DataSource } from 'typeorm';

export const TestDataSource = new DataSource({
  type: 'sqlite',
  database: ':memory:',
  entities: [__dirname + '/../entities/**/*.ts'],
  synchronize: true,
  logging: false,
  dropSchema: true,
});

// In tests
beforeAll(async () => {
  await TestDataSource.initialize();
});

afterAll(async () => {
  await TestDataSource.destroy();
});
```

### Real-World Production Setup

```typescript
import { DataSource, DataSourceOptions } from 'typeorm';
import { SnakeNamingStrategy } from 'typeorm-naming-strategies';

const config: DataSourceOptions = {
  type: 'postgres',
  host: process.env.DB_HOST,
  port: parseInt(process.env.DB_PORT || '5432'),
  username: process.env.DB_USERNAME,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  
  entities: [__dirname + '/entities/**/*.{ts,js}'],
  migrations: [__dirname + '/migrations/**/*.{ts,js}'],
  
  // Never use synchronize in production
  synchronize: false,
  
  // Use migrations instead
  migrationsRun: true,
  
  // Naming strategy for consistent database naming
  namingStrategy: new SnakeNamingStrategy(),
  
  // Connection pooling
  extra: {
    max: 20,
    min: 5,
    idleTimeoutMillis: 30000,
    connectionTimeoutMillis: 10000,
  },
  
  // SSL for production
  ssl: {
    rejectUnauthorized: false,
  },
  
  // Logging only errors in production
  logging: ['error'],
  
  // Connection retry
  maxQueryExecutionTime: 5000,
};

export const AppDataSource = new DataSource(config);

// Initialize with retry logic
async function initializeDataSource(retries = 5, delay = 5000) {
  for (let i = 0; i < retries; i++) {
    try {
      await AppDataSource.initialize();
      console.log('✅ Database connected');
      return;
    } catch (error) {
      console.error(`❌ Connection attempt ${i + 1} failed:`, error.message);
      if (i < retries - 1) {
        console.log(`⏳ Retrying in ${delay / 1000} seconds...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
  }
  throw new Error('Failed to connect to database after multiple attempts');
}

initializeDataSource();
```

**Interview Tip**: Explain that DataSource is the modern approach replacing `createConnection()`. Emphasize the importance of never using `synchronize: true` in production, proper connection pooling, SSL configuration, and graceful shutdown handling. Mention environment-based configuration for different deployment environments.

</details>

<details>
<summary>7. What are the different connection options available in TypeORM?</summary>

TypeORM provides extensive connection options to configure database behavior, performance, security, and logging. Understanding these options is crucial for production deployments.

### Core Connection Options

```typescript
import { DataSource } from 'typeorm';

const AppDataSource = new DataSource({
  // ===== Basic Options =====
  
  // Database type (required)
  type: 'postgres', // 'mysql' | 'mariadb' | 'sqlite' | 'mssql' | 'oracle' | 'mongodb'
  
  // Connection details
  host: 'localhost',
  port: 5432,
  username: 'postgres',
  password: 'password',
  database: 'mydb',
  
  // Alternatively, use URL
  // url: 'postgres://user:password@localhost:5432/mydb',
  
  // ===== Entity Options =====
  
  // Entity files to load
  entities: [
    'src/entities/**/*.ts',
    // Or import directly
    // User, Post, Comment
  ],
  
  // ===== Migration Options =====
  
  migrations: ['src/migrations/**/*.ts'],
  migrationsTableName: 'migrations_history',
  migrationsRun: false, // Auto-run migrations on startup
  migrationsTransactionMode: 'all', // 'all' | 'none' | 'each'
  
  // ===== Subscriber Options =====
  
  subscribers: ['src/subscribers/**/*.ts'],
  
  // ===== Schema Options =====
  
  synchronize: false, // NEVER true in production
  dropSchema: false, // Drop schema on connection
  schema: 'public', // Default schema
  
  // ===== Logging Options =====
  
  logging: true, // boolean | 'all' | Array
  // logging: ['query', 'error', 'schema', 'warn', 'info', 'log']
  logger: 'advanced-console', // 'advanced-console' | 'simple-console' | 'file' | 'debug'
  maxQueryExecutionTime: 1000, // Log slow queries (ms)
  
  // ===== Caching Options =====
  
  cache: {
    type: 'redis',
    options: {
      host: 'localhost',
      port: 6379,
    },
    duration: 30000, // 30 seconds
  },
  
  // ===== Connection Pool Options =====
  
  extra: {
    // Connection pool size
    max: 10,
    min: 2,
    
    // Timeouts
    connectionTimeoutMillis: 10000,
    idleTimeoutMillis: 30000,
    
    // Keep alive
    keepAlive: true,
    keepAliveInitialDelayMillis: 0,
  },
});
```

### Database-Specific Options

```typescript
// ===== PostgreSQL Options =====
const PostgresDataSource = new DataSource({
  type: 'postgres',
  host: 'localhost',
  port: 5432,
  username: 'postgres',
  password: 'password',
  database: 'mydb',
  
  // PostgreSQL specific
  ssl: {
    rejectUnauthorized: false,
  },
  schema: 'public',
  applicationName: 'my-app',
  
  // Connection pool (pg driver options)
  extra: {
    max: 20,
    min: 5,
    idleTimeoutMillis: 30000,
    connectionTimeoutMillis: 10000,
    statement_timeout: 30000,
    query_timeout: 30000,
  },
  
  // Use native bindings (faster)
  // useUTC: true,
});

// ===== MySQL Options =====
const MySQLDataSource = new DataSource({
  type: 'mysql',
  host: 'localhost',
  port: 3306,
  username: 'root',
  password: 'password',
  database: 'mydb',
  
  // MySQL specific
  charset: 'utf8mb4',
  timezone: '+00:00',
  connectTimeout: 10000,
  
  // Connection pool
  extra: {
    connectionLimit: 10,
    waitForConnections: true,
    queueLimit: 0,
  },
  
  // BigInt handling
  bigNumberStrings: false,
  supportBigNumbers: true,
});

// ===== SQLite Options =====
const SQLiteDataSource = new DataSource({
  type: 'sqlite',
  database: 'database.sqlite',
  
  // Or in-memory
  // database: ':memory:',
});

// ===== Microsoft SQL Server Options =====
const MSSQLDataSource = new DataSource({
  type: 'mssql',
  host: 'localhost',
  port: 1433,
  username: 'sa',
  password: 'Password123',
  database: 'mydb',
  
  // MSSQL specific
  options: {
    encrypt: true,
    trustServerCertificate: true,
    enableArithAbort: true,
    instanceName: 'SQLEXPRESS',
  },
  
  extra: {
    max: 10,
    min: 0,
    idleTimeoutMillis: 30000,
  },
});
```

### Advanced Configuration

```typescript
import { DataSource } from 'typeorm';
import { SnakeNamingStrategy } from 'typeorm-naming-strategies';

const AdvancedDataSource = new DataSource({
  type: 'postgres',
  host: 'localhost',
  port: 5432,
  username: 'postgres',
  password: 'password',
  database: 'mydb',
  
  // ===== Naming Strategy =====
  // Convert camelCase to snake_case
  namingStrategy: new SnakeNamingStrategy(),
  
  // ===== Entity Metadata Caching =====
  // Cache entity metadata for better performance
  cache: {
    type: 'database',
    tableName: 'query_result_cache',
  },
  
  // ===== Replication Setup =====
  replication: {
    master: {
      host: 'master-db.example.com',
      port: 5432,
      username: 'postgres',
      password: 'password',
      database: 'mydb',
    },
    slaves: [
      {
        host: 'slave1-db.example.com',
        port: 5432,
        username: 'postgres',
        password: 'password',
        database: 'mydb',
      },
      {
        host: 'slave2-db.example.com',
        port: 5432,
        username: 'postgres',
        password: 'password',
        database: 'mydb',
      },
    ],
  },
  
  // ===== Driver-Specific Options =====
  driver: require('pg'),
  
  // ===== CLI Configuration =====
  cli: {
    migrationsDir: 'src/migrations',
    entitiesDir: 'src/entities',
    subscribersDir: 'src/subscribers',
  },
});
```

### Production Configuration

```typescript
import { DataSource, DataSourceOptions } from 'typeorm';

const productionConfig: DataSourceOptions = {
  type: 'postgres',
  
  // Use environment variables
  host: process.env.DB_HOST!,
  port: parseInt(process.env.DB_PORT || '5432'),
  username: process.env.DB_USERNAME!,
  password: process.env.DB_PASSWORD!,
  database: process.env.DB_NAME!,
  
  // Load entities
  entities: [__dirname + '/entities/**/*.js'],
  
  // Migrations
  migrations: [__dirname + '/migrations/**/*.js'],
  migrationsRun: true, // Auto-run migrations
  migrationsTableName: 'typeorm_migrations',
  
  // CRITICAL: Never synchronize in production
  synchronize: false,
  
  // Logging only errors
  logging: ['error'],
  logger: 'advanced-console',
  maxQueryExecutionTime: 5000, // Log slow queries
  
  // SSL Configuration
  ssl: {
    rejectUnauthorized: false,
    ca: process.env.DB_SSL_CA,
    key: process.env.DB_SSL_KEY,
    cert: process.env.DB_SSL_CERT,
  },
  
  // Connection Pool
  extra: {
    max: parseInt(process.env.DB_POOL_MAX || '20'),
    min: parseInt(process.env.DB_POOL_MIN || '5'),
    
    // Timeouts
    connectionTimeoutMillis: parseInt(process.env.DB_TIMEOUT || '10000'),
    idleTimeoutMillis: 30000,
    
    // Keep connections alive
    keepAlive: true,
    keepAliveInitialDelayMillis: 0,
    
    // Statement timeout
    statement_timeout: 60000,
  },
  
  // Query result caching
  cache: {
    type: 'redis',
    options: {
      host: process.env.REDIS_HOST || 'localhost',
      port: parseInt(process.env.REDIS_PORT || '6379'),
      password: process.env.REDIS_PASSWORD,
    },
    duration: 60000, // 1 minute default cache
    ignoreErrors: true, // Continue if cache fails
  },
};

export const AppDataSource = new DataSource(productionConfig);
```

### Connection Options by Environment

```typescript
import { DataSource, DataSourceOptions } from 'typeorm';

// Base configuration
const baseConfig: Partial<DataSourceOptions> = {
  type: 'postgres',
  entities: [__dirname + '/entities/**/*.{ts,js}'],
  migrations: [__dirname + '/migrations/**/*.{ts,js}'],
};

// Development configuration
const developmentConfig: DataSourceOptions = {
  ...baseConfig,
  host: 'localhost',
  port: 5432,
  username: 'dev_user',
  password: 'dev_password',
  database: 'dev_db',
  
  synchronize: true, // OK for development
  logging: true,
  dropSchema: false,
  
  extra: {
    max: 5,
    min: 1,
  },
} as DataSourceOptions;

// Staging configuration
const stagingConfig: DataSourceOptions = {
  ...baseConfig,
  host: process.env.STAGING_DB_HOST!,
  port: parseInt(process.env.STAGING_DB_PORT!),
  username: process.env.STAGING_DB_USER!,
  password: process.env.STAGING_DB_PASS!,
  database: process.env.STAGING_DB_NAME!,
  
  synchronize: false,
  migrationsRun: true,
  logging: ['error', 'warn'],
  
  ssl: { rejectUnauthorized: false },
  
  extra: {
    max: 10,
    min: 2,
  },
} as DataSourceOptions;

// Production configuration
const productionConfig: DataSourceOptions = {
  ...baseConfig,
  host: process.env.DB_HOST!,
  port: parseInt(process.env.DB_PORT!),
  username: process.env.DB_USER!,
  password: process.env.DB_PASS!,
  database: process.env.DB_NAME!,
  
  synchronize: false,
  migrationsRun: true,
  logging: ['error'],
  maxQueryExecutionTime: 5000,
  
  ssl: {
    rejectUnauthorized: true,
    ca: process.env.DB_SSL_CA,
  },
  
  extra: {
    max: 20,
    min: 5,
    connectionTimeoutMillis: 10000,
    idleTimeoutMillis: 30000,
  },
  
  cache: {
    type: 'redis',
    options: {
      host: process.env.REDIS_HOST,
      port: parseInt(process.env.REDIS_PORT || '6379'),
      password: process.env.REDIS_PASSWORD,
    },
  },
} as DataSourceOptions;

// Select config based on environment
const configs = {
  development: developmentConfig,
  staging: stagingConfig,
  production: productionConfig,
};

const env = process.env.NODE_ENV || 'development';
export const AppDataSource = new DataSource(configs[env]);
```

**Interview Tip**: Highlight the importance of environment-specific configurations. Emphasize that `synchronize: false` is critical for production, proper SSL setup is essential for security, connection pooling prevents resource exhaustion, and logging should be minimal in production (`['error']` only). Mention that understanding database-specific options (like `statement_timeout` for PostgreSQL) shows deep knowledge.

</details>

<details>
<summary>8. How do you handle multiple database connections in TypeORM?</summary>

TypeORM supports multiple database connections, which is essential for microservices, multi-tenant applications, or when working with separate databases for different concerns (e.g., users, analytics, logs).

### Basic Multiple Connections Setup

```typescript
import { DataSource } from 'typeorm';
import { User } from './entities/user/User';
import { Product } from './entities/product/Product';
import { Log } from './entities/log/Log';

// Primary database (Users)
export const UserDataSource = new DataSource({
  name: 'userConnection', // Optional name
  type: 'postgres',
  host: 'users-db.example.com',
  port: 5432,
  username: 'user_admin',
  password: 'password',
  database: 'users_db',
  entities: [User],
  synchronize: false,
  logging: ['error'],
});

// Secondary database (Products)
export const ProductDataSource = new DataSource({
  name: 'productConnection',
  type: 'mysql',
  host: 'products-db.example.com',
  port: 3306,
  username: 'product_admin',
  password: 'password',
  database: 'products_db',
  entities: [Product],
  synchronize: false,
  logging: ['error'],
});

// Tertiary database (Logs - Different type)
export const LogDataSource = new DataSource({
  name: 'logConnection',
  type: 'mongodb',
  host: 'logs-db.example.com',
  port: 27017,
  database: 'logs_db',
  entities: [Log],
  synchronize: false,
  useUnifiedTopology: true,
});

// Initialize all connections
export async function initializeConnections() {
  try {
    await Promise.all([
      UserDataSource.initialize(),
      ProductDataSource.initialize(),
      LogDataSource.initialize(),
    ]);
    console.log('✅ All database connections initialized');
  } catch (error) {
    console.error('❌ Error initializing connections:', error);
    throw error;
  }
}

// Cleanup on shutdown
export async function closeConnections() {
  await Promise.all([
    UserDataSource.destroy(),
    ProductDataSource.destroy(),
    LogDataSource.destroy(),
  ]);
  console.log('All connections closed');
}
```

### Using Multiple Connections in Services

```typescript
import { UserDataSource, ProductDataSource } from './data-sources';
import { User } from './entities/user/User';
import { Product } from './entities/product/Product';

class UserService {
  private userRepository = UserDataSource.getRepository(User);
  
  async createUser(name: string, email: string) {
    const user = this.userRepository.create({ name, email });
    return await this.userRepository.save(user);
  }
  
  async findUserById(id: number) {
    return await this.userRepository.findOneBy({ id });
  }
}

class ProductService {
  private productRepository = ProductDataSource.getRepository(Product);
  
  async createProduct(name: string, price: number) {
    const product = this.productRepository.create({ name, price });
    return await this.productRepository.save(product);
  }
  
  async findProductById(id: number) {
    return await this.productRepository.findOneBy({ id });
  }
}

// Usage
const userService = new UserService();
const productService = new ProductService();

async function example() {
  const user = await userService.createUser('John', 'john@example.com');
  const product = await productService.createProduct('Laptop', 999.99);
  
  console.log('User:', user);
  console.log('Product:', product);
}
```

### NestJS Integration with Multiple Databases

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { User } from './entities/user/User';
import { Product } from './entities/product/Product';
import { UserModule } from './modules/user/user.module';
import { ProductModule } from './modules/product/product.module';

@Module({
  imports: [
    // Primary connection (Users)
    TypeOrmModule.forRoot({
      name: 'userConnection',
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'postgres',
      password: 'password',
      database: 'users_db',
      entities: [User],
      synchronize: false,
    }),
    
    // Secondary connection (Products)
    TypeOrmModule.forRoot({
      name: 'productConnection',
      type: 'mysql',
      host: 'localhost',
      port: 3306,
      username: 'root',
      password: 'password',
      database: 'products_db',
      entities: [Product],
      synchronize: false,
    }),
    
    UserModule,
    ProductModule,
  ],
})
export class AppModule {}

// user.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { User } from './user.entity';
import { UserService } from './user.service';
import { UserController } from './user.controller';

@Module({
  imports: [
    // Specify connection name
    TypeOrmModule.forFeature([User], 'userConnection'),
  ],
  providers: [UserService],
  controllers: [UserController],
})
export class UserModule {}

// user.service.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './user.entity';

@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User, 'userConnection')
    private userRepository: Repository<User>,
  ) {}
  
  async findAll(): Promise<User[]> {
    return await this.userRepository.find();
  }
  
  async create(name: string, email: string): Promise<User> {
    const user = this.userRepository.create({ name, email });
    return await this.userRepository.save(user);
  }
}

// product.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { Product } from './product.entity';
import { ProductService } from './product.service';

@Module({
  imports: [
    TypeOrmModule.forFeature([Product], 'productConnection'),
  ],
  providers: [ProductService],
  exports: [ProductService],
})
export class ProductModule {}

// product.service.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Product } from './product.entity';

@Injectable()
export class ProductService {
  constructor(
    @InjectRepository(Product, 'productConnection')
    private productRepository: Repository<Product>,
  ) {}
  
  async findAll(): Promise<Product[]> {
    return await this.productRepository.find();
  }
}
```

### Transactions Across Multiple Databases

**Important**: TypeORM does NOT support distributed transactions across different databases. Each transaction is isolated to its own database connection.

```typescript
import { UserDataSource, ProductDataSource } from './data-sources';
import { User } from './entities/user/User';
import { Product } from './entities/product/Product';

// ❌ This will NOT work as a single distributed transaction
async function incorrectMultiDbTransaction() {
  const userQueryRunner = UserDataSource.createQueryRunner();
  const productQueryRunner = ProductDataSource.createQueryRunner();
  
  await userQueryRunner.startTransaction();
  await productQueryRunner.startTransaction();
  
  try {
    // These are two separate transactions, not one distributed transaction
    await userQueryRunner.manager.save(User, { name: 'John' });
    await productQueryRunner.manager.save(Product, { name: 'Laptop' });
    
    // If product save fails, user save won't be rolled back automatically
    await userQueryRunner.commitTransaction();
    await productQueryRunner.commitTransaction();
  } catch (error) {
    await userQueryRunner.rollbackTransaction();
    await productQueryRunner.rollbackTransaction();
    throw error;
  }
}

// ✅ Better approach: Saga pattern or compensation
async function sagaPattern() {
  let userCreated = false;
  let user: User | null = null;
  
  try {
    // Step 1: Create user
    const userRepository = UserDataSource.getRepository(User);
    user = await userRepository.save({ name: 'John', email: 'john@example.com' });
    userCreated = true;
    
    // Step 2: Create product
    const productRepository = ProductDataSource.getRepository(Product);
    await productRepository.save({ name: 'Laptop', price: 999 });
    
    return { success: true, user };
  } catch (error) {
    // Compensation: Rollback user creation if product creation fails
    if (userCreated && user) {
      try {
        const userRepository = UserDataSource.getRepository(User);
        await userRepository.remove(user);
        console.log('Compensated: Removed user after product creation failed');
      } catch (compensationError) {
        console.error('Compensation failed:', compensationError);
        // Log to dead letter queue or alert system
      }
    }
    throw error;
  }
}

// ✅ Event-driven approach
interface Event {
  type: string;
  data: any;
  timestamp: Date;
}

class EventBus {
  private events: Event[] = [];
  
  async publish(event: Event) {
    this.events.push(event);
    // Process event asynchronously
    await this.processEvent(event);
  }
  
  private async processEvent(event: Event) {
    try {
      if (event.type === 'USER_CREATED') {
        // Create related product
        const productRepository = ProductDataSource.getRepository(Product);
        await productRepository.save({
          name: `Product for ${event.data.userName}`,
          price: 100,
        });
      }
    } catch (error) {
      console.error('Event processing failed:', error);
      // Retry or send to dead letter queue
    }
  }
}

async function eventDrivenApproach() {
  const eventBus = new EventBus();
  
  // Create user
  const userRepository = UserDataSource.getRepository(User);
  const user = await userRepository.save({ name: 'John', email: 'john@example.com' });
  
  // Publish event (asynchronous)
  await eventBus.publish({
    type: 'USER_CREATED',
    data: { userId: user.id, userName: user.name },
    timestamp: new Date(),
  });
  
  return user;
}
```

### Multi-Tenant Architecture

```typescript
import { DataSource } from 'typeorm';
import { User } from './entities/User';

// Tenant connection manager
class TenantConnectionManager {
  private connections: Map<string, DataSource> = new Map();
  
  async getConnection(tenantId: string): Promise<DataSource> {
    // Return existing connection if available
    if (this.connections.has(tenantId)) {
      return this.connections.get(tenantId)!;
    }
    
    // Create new connection for tenant
    const dataSource = new DataSource({
      name: `tenant_${tenantId}`,
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'postgres',
      password: 'password',
      database: `tenant_${tenantId}_db`,
      entities: [User],
      synchronize: false,
    });
    
    await dataSource.initialize();
    this.connections.set(tenantId, dataSource);
    
    return dataSource;
  }
  
  async closeConnection(tenantId: string) {
    const connection = this.connections.get(tenantId);
    if (connection) {
      await connection.destroy();
      this.connections.delete(tenantId);
    }
  }
  
  async closeAllConnections() {
    for (const [tenantId, connection] of this.connections) {
      await connection.destroy();
    }
    this.connections.clear();
  }
}

// Usage in service
const connectionManager = new TenantConnectionManager();

async function getUsersForTenant(tenantId: string) {
  const connection = await connectionManager.getConnection(tenantId);
  const userRepository = connection.getRepository(User);
  return await userRepository.find();
}

// Middleware to set tenant context
import { Request, Response, NextFunction } from 'express';

interface TenantRequest extends Request {
  tenantId?: string;
}

async function tenantMiddleware(
  req: TenantRequest,
  res: Response,
  next: NextFunction
) {
  // Extract tenant from subdomain, header, or JWT
  const tenantId = req.headers['x-tenant-id'] as string;
  
  if (!tenantId) {
    return res.status(400).json({ error: 'Tenant ID required' });
  }
  
  req.tenantId = tenantId;
  next();
}

// Route handler
import express from 'express';

const app = express();
app.use(tenantMiddleware);

app.get('/users', async (req: TenantRequest, res: Response) => {
  try {
    const users = await getUsersForTenant(req.tenantId!);
    res.json(users);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

### Best Practices for Multiple Connections

```typescript
import { DataSource } from 'typeorm';

// 1. Connection pooling per database
const UserDataSource = new DataSource({
  name: 'userConnection',
  type: 'postgres',
  host: 'users-db.example.com',
  // ... other options
  extra: {
    max: 20, // Max pool size
    min: 5,  // Min pool size
  },
});

// 2. Health checks
async function checkDatabaseHealth() {
  const connections = [UserDataSource, ProductDataSource, LogDataSource];
  
  const results = await Promise.all(
    connections.map(async (ds) => {
      try {
        await ds.query('SELECT 1');
        return { name: ds.name, status: 'healthy' };
      } catch (error) {
        return { name: ds.name, status: 'unhealthy', error: error.message };
      }
    })
  );
  
  return results;
}

// 3. Graceful shutdown
process.on('SIGTERM', async () => {
  console.log('SIGTERM received, closing connections...');
  await closeConnections();
  process.exit(0);
});

// 4. Connection retry logic
async function initializeWithRetry(
  dataSource: DataSource,
  maxRetries = 5,
  delay = 5000
) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      await dataSource.initialize();
      console.log(`✅ ${dataSource.name} connected`);
      return;
    } catch (error) {
      console.error(`❌ ${dataSource.name} connection attempt ${i + 1} failed`);
      if (i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
  }
  throw new Error(`Failed to connect ${dataSource.name}`);
}

// 5. Monitoring
async function monitorConnections() {
  setInterval(async () => {
    const health = await checkDatabaseHealth();
    console.log('Database health:', health);
    
    // Alert if any connection is unhealthy
    const unhealthy = health.filter(h => h.status === 'unhealthy');
    if (unhealthy.length > 0) {
      console.error('Unhealthy connections detected:', unhealthy);
      // Send alert to monitoring system
    }
  }, 60000); // Check every minute
}
```

**Interview Tip**: Emphasize that TypeORM does NOT support distributed transactions across different databases. Explain alternative patterns like Saga, event-driven architecture, or eventual consistency for multi-database operations. Highlight the importance of proper connection pooling, health checks, and graceful shutdown. For NestJS, mention the `@InjectRepository` decorator with connection name parameter. Discuss multi-tenant architectures and connection management strategies.

</details>

<details>
<summary>9. What is the purpose of the synchronize option and should it be used in production?</summary>

The `synchronize` option in TypeORM automatically synchronizes your entity schema with the database schema. While convenient for development, it should **NEVER** be used in production environments.

### What Does synchronize Do?

When `synchronize: true`, TypeORM automatically:
- Creates tables for entities that don't exist
- Adds new columns for entity properties
- Updates column types if they change
- Creates indexes defined in entities
- Creates foreign key constraints

**Important**: It does NOT drop columns or tables that are removed from entities.

```typescript
import { DataSource } from 'typeorm';

const AppDataSource = new DataSource({
  type: 'postgres',
  host: 'localhost',
  port: 5432,
  username: 'postgres',
  password: 'password',
  database: 'mydb',
  entities: [User, Post],
  
  // Synchronize option
  synchronize: true, // Auto-sync schema with entities
});
```

### Development Usage

```typescript
// Development configuration
const DevDataSource = new DataSource({
  type: 'postgres',
  host: 'localhost',
  port: 5432,
  username: 'dev_user',
  password: 'dev_password',
  database: 'dev_db',
  entities: [__dirname + '/entities/**/*.ts'],
  
  // OK for development
  synchronize: true,
  
  // Also drop schema on start (clean slate)
  dropSchema: false, // Set to true only for testing
  
  logging: true,
});
```

### Why NOT Use in Production?

```typescript
// ❌ NEVER DO THIS IN PRODUCTION
const ProductionDataSource = new DataSource({
  type: 'postgres',
  host: 'production-db.example.com',
  database: 'production_db',
  entities: [User, Post],
  
  // ❌ DANGEROUS! Will modify production schema automatically
  synchronize: true,
});

/*
PROBLEMS with synchronize: true in production:
1. Data Loss: Can cause data loss if column types change
2. Downtime: Schema changes happen during app startup
3. No Rollback: No way to rollback if something goes wrong
4. No Review: Schema changes aren't reviewed or tested
5. Race Conditions: Multiple instances might conflict
6. Performance: Schema operations can lock tables
7. Incomplete: Doesn't handle complex migrations
*/
```

### Real-World Examples of Issues

```typescript
// Example 1: Column type change causes data loss
@Entity()
class User {
  @PrimaryGeneratedColumn()
  id: number;
  
  // Changed from string to number
  // If synchronize: true, TypeORM will change the column type
  // This could cause data loss or errors!
  @Column('int') // Was: @Column('varchar')
  age: number; // Was: age: string
}

// Example 2: Removing a property doesn't drop the column
@Entity()
class Post {
  @PrimaryGeneratedColumn()
  id: number;
  
  @Column()
  title: string;
  
  // Removed: @Column() body: string;
  // The 'body' column will remain in the database!
  // This can cause confusion and wasted storage
}

// Example 3: Renaming causes column duplication
@Entity()
class Product {
  @PrimaryGeneratedColumn()
  id: number;
  
  // Renamed from 'name' to 'productName'
  @Column()
  productName: string; // Was: name: string
  
  // TypeORM will CREATE 'productName' column
  // but NOT DROP 'name' column
  // Result: Both columns exist, data is lost!
}
```

### Correct Approach: Use Migrations

```typescript
// Production configuration
const ProductionDataSource = new DataSource({
  type: 'postgres',
  host: 'production-db.example.com',
  database: 'production_db',
  entities: [__dirname + '/entities/**/*.js'],
  
  // ✅ NEVER synchronize in production
  synchronize: false,
  
  // ✅ Use migrations instead
  migrations: [__dirname + '/migrations/**/*.js'],
  migrationsRun: true, // Auto-run pending migrations on startup
  migrationsTableName: 'typeorm_migrations',
});
```

### Creating Migrations Instead

```bash
# Generate migration from entity changes
npm run typeorm migration:generate -- -n AddUserAge

# Create empty migration
npm run typeorm migration:create -- -n AddIndexToEmail

# Run migrations
npm run typeorm migration:run

# Revert last migration
npm run typeorm migration:revert
```

```typescript
// Example migration file
import { MigrationInterface, QueryRunner, TableColumn } from 'typeorm';

export class AddUserAge1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    // Add new column with proper handling
    await queryRunner.addColumn('user', new TableColumn({
      name: 'age',
      type: 'int',
      isNullable: true, // Allow null for existing rows
      default: null,
    }));
    
    // Optionally set default values for existing rows
    await queryRunner.query(`
      UPDATE "user" SET age = 0 WHERE age IS NULL
    `);
    
    // Make it non-nullable if needed
    await queryRunner.changeColumn('user', 'age', new TableColumn({
      name: 'age',
      type: 'int',
      isNullable: false,
      default: 0,
    }));
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    // Rollback: remove the column
    await queryRunner.dropColumn('user', 'age');
  }
}
```

### Environment-Based Configuration

```typescript
import { DataSource } from 'typeorm';

const config = {
  development: {
    synchronize: true,
    dropSchema: false,
    logging: true,
    migrationsRun: false,
  },
  test: {
    synchronize: true,
    dropSchema: true, // Clean slate for each test run
    logging: false,
    migrationsRun: false,
  },
  staging: {
    synchronize: false,
    dropSchema: false,
    logging: ['error', 'warn'],
    migrationsRun: true,
  },
  production: {
    synchronize: false, // CRITICAL!
    dropSchema: false,
    logging: ['error'],
    migrationsRun: true,
  },
};

const env = process.env.NODE_ENV || 'development';

export const AppDataSource = new DataSource({
  type: 'postgres',
  host: process.env.DB_HOST,
  port: parseInt(process.env.DB_PORT || '5432'),
  username: process.env.DB_USERNAME,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  entities: [__dirname + '/entities/**/*.{ts,js}'],
  migrations: [__dirname + '/migrations/**/*.{ts,js}'],
  ...config[env],
});
```

### Testing with synchronize

```typescript
// test/setup.ts
import { DataSource } from 'typeorm';

export const TestDataSource = new DataSource({
  type: 'sqlite',
  database: ':memory:',
  entities: [__dirname + '/../src/entities/**/*.ts'],
  
  // OK for testing
  synchronize: true,
  dropSchema: true, // Clean database for each test
  
  logging: false,
});

// test/user.test.ts
import { TestDataSource } from './setup';

beforeAll(async () => {
  await TestDataSource.initialize();
});

afterAll(async () => {
  await TestDataSource.destroy();
});

describe('User', () => {
  it('should create a user', async () => {
    const userRepo = TestDataSource.getRepository(User);
    const user = userRepo.create({ name: 'John', email: 'john@example.com' });
    await userRepo.save(user);
    
    expect(user.id).toBeDefined();
  });
});
```

### Alternative: schema:sync Command

```typescript
// For development, manually sync schema when needed
// package.json
{
  "scripts": {
    "schema:sync": "typeorm schema:sync",
    "schema:drop": "typeorm schema:drop"
  }
}

// Run manually when you change entities
// npm run schema:sync
```

### Best Practices Summary

```typescript
// ✅ GOOD: Development
const devConfig = {
  synchronize: true,  // OK for local development
  dropSchema: false,  // Don't drop tables
  logging: true,      // See all queries
};

// ✅ GOOD: Testing
const testConfig = {
  synchronize: true,   // OK for tests
  dropSchema: true,    // Clean slate for each test run
  logging: false,      // Reduce noise
};

// ✅ GOOD: Staging
const stagingConfig = {
  synchronize: false,  // Use migrations
  migrationsRun: true, // Auto-run migrations
  logging: ['error'],  // Only errors
};

// ✅ GOOD: Production
const prodConfig = {
  synchronize: false,     // NEVER sync in production
  migrationsRun: true,    // Auto-run migrations (or manual)
  logging: ['error'],     // Only errors
  migrationsTableName: 'migrations', // Track migrations
};

// ❌ BAD: Any environment
const badConfig = {
  synchronize: true,   // Never use in staging/production
  dropSchema: true,    // Never use outside testing
};
```

### Migration Workflow (Recommended)

```bash
# 1. Modify your entities
# 2. Generate migration
npm run typeorm migration:generate -- -n DescriptiveNameHere

# 3. Review the generated migration file
# 4. Test migration in development
npm run typeorm migration:run

# 5. Test rollback
npm run typeorm migration:revert

# 6. Commit migration files to version control
git add src/migrations/*.ts
git commit -m "Add migration for user age field"

# 7. Deploy to production (migrations run automatically or manually)
```

**Interview Tip**: Strongly emphasize that `synchronize: true` should NEVER be used in production. Explain the risks: data loss, no rollback capability, race conditions with multiple instances, and lack of review process. Highlight that migrations provide version control, rollback capability, team review, and controlled deployment. Mention that `synchronize` is acceptable for development and testing environments, especially with in-memory databases. A good answer demonstrates understanding of both the feature and production best practices.

</details>

<details>
<summary>10. How do you close a database connection in TypeORM?</summary>

Properly closing database connections is crucial for preventing resource leaks, ensuring graceful shutdowns, and maintaining application stability. TypeORM provides several methods to manage connection lifecycle.

### Basic Connection Closing

```typescript
import { DataSource } from 'typeorm';

const AppDataSource = new DataSource({
  type: 'postgres',
  host: 'localhost',
  port: 5432,
  username: 'postgres',
  password: 'password',
  database: 'mydb',
  entities: [User],
});

// Initialize connection
await AppDataSource.initialize();

// Use connection...
// ...

// Close connection
await AppDataSource.destroy();
console.log('Connection closed');
```

### Graceful Shutdown in Applications

```typescript
import express from 'express';
import { AppDataSource } from './data-source';

const app = express();

// Initialize database
AppDataSource.initialize()
  .then(() => {
    console.log('✅ Database connected');
    
    // Start server
    const server = app.listen(3000, () => {
      console.log('🚀 Server running on port 3000');
    });
    
    // Graceful shutdown handler
    const shutdown = async () => {
      console.log('🛑 Shutdown signal received');
      
      // Stop accepting new requests
      server.close(() => {
        console.log('✅ HTTP server closed');
      });
      
      // Close database connection
      await AppDataSource.destroy();
      console.log('✅ Database connection closed');
      
      process.exit(0);
    };
    
    // Listen for shutdown signals
    process.on('SIGTERM', shutdown);
    process.on('SIGINT', shutdown);
  })
  .catch((error) => {
    console.error('❌ Database connection failed:', error);
    process.exit(1);
  });
```

### Multiple Connections Management

```typescript
import { DataSource } from 'typeorm';

// Define multiple data sources
const UserDataSource = new DataSource({
  name: 'userConnection',
  type: 'postgres',
  host: 'localhost',
  database: 'users_db',
  entities: [User],
});

const ProductDataSource = new DataSource({
  name: 'productConnection',
  type: 'mysql',
  host: 'localhost',
  database: 'products_db',
  entities: [Product],
});

// Initialize all connections
export async function initializeConnections() {
  await Promise.all([
    UserDataSource.initialize(),
    ProductDataSource.initialize(),
  ]);
  console.log('✅ All connections initialized');
}

// Close all connections
export async function closeAllConnections() {
  try {
    await Promise.all([
      UserDataSource.destroy(),
      ProductDataSource.destroy(),
    ]);
    console.log('✅ All connections closed');
  } catch (error) {
    console.error('❌ Error closing connections:', error);
    throw error;
  }
}

// Graceful shutdown
process.on('SIGTERM', async () => {
  console.log('SIGTERM received');
  await closeAllConnections();
  process.exit(0);
});

process.on('SIGINT', async () => {
  console.log('SIGINT received');
  await closeAllConnections();
  process.exit(0);
});
```

### NestJS Connection Management

```typescript
// app.module.ts
import { Module, OnModuleDestroy } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { DataSource } from 'typeorm';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'postgres',
      password: 'password',
      database: 'mydb',
      entities: [User],
      synchronize: false,
    }),
  ],
})
export class AppModule implements OnModuleDestroy {
  constructor(private dataSource: DataSource) {}
  
  // Called when application shuts down
  async onModuleDestroy() {
    await this.dataSource.destroy();
    console.log('✅ Database connection closed');
  }
}

// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Enable graceful shutdown
  app.enableShutdownHooks();
  
  await app.listen(3000);
  console.log('🚀 Application is running');
}

bootstrap();
```

### Connection Pool Management

```typescript
import { DataSource } from 'typeorm';

const AppDataSource = new DataSource({
  type: 'postgres',
  host: 'localhost',
  port: 5432,
  username: 'postgres',
  password: 'password',
  database: 'mydb',
  entities: [User],
  
  // Connection pool configuration
  extra: {
    max: 20, // Maximum pool size
    min: 5,  // Minimum pool size
    idleTimeoutMillis: 30000, // Close idle connections after 30s
  },
});

// Initialize
await AppDataSource.initialize();

// Get pool statistics (PostgreSQL)
const poolStats = await AppDataSource.query(`
  SELECT 
    count(*) as total,
    count(*) FILTER (WHERE state = 'active') as active,
    count(*) FILTER (WHERE state = 'idle') as idle
  FROM pg_stat_activity
  WHERE datname = current_database()
`);
console.log('Pool stats:', poolStats);

// Destroy connection (closes all pooled connections)
await AppDataSource.destroy();
```

### Handling Active Queries During Shutdown

```typescript
import { DataSource } from 'typeorm';

class DatabaseManager {
  private dataSource: DataSource;
  private activeQueries: Set<Promise<any>> = new Set();
  
  constructor(dataSource: DataSource) {
    this.dataSource = dataSource;
  }
  
  // Track active queries
  async query<T>(queryFn: () => Promise<T>): Promise<T> {
    const queryPromise = queryFn();
    this.activeQueries.add(queryPromise);
    
    try {
      const result = await queryPromise;
      return result;
    } finally {
      this.activeQueries.delete(queryPromise);
    }
  }
  
  // Graceful shutdown: wait for active queries
  async close(timeout: number = 30000): Promise<void> {
    console.log(`Waiting for ${this.activeQueries.size} active queries to complete...`);
    
    // Wait for active queries with timeout
    const timeoutPromise = new Promise((_, reject) => {
      setTimeout(() => reject(new Error('Shutdown timeout')), timeout);
    });
    
    try {
      await Promise.race([
        Promise.all(this.activeQueries),
        timeoutPromise,
      ]);
      console.log('✅ All queries completed');
    } catch (error) {
      console.warn('⚠️ Shutdown timeout, forcing close');
    }
    
    // Close connection
    await this.dataSource.destroy();
    console.log('✅ Connection closed');
  }
}

// Usage
const manager = new DatabaseManager(AppDataSource);

// Track query
const users = await manager.query(() => 
  AppDataSource.getRepository(User).find()
);

// Graceful shutdown
process.on('SIGTERM', async () => {
  await manager.close(30000); // 30 second timeout
  process.exit(0);
});
```

### Testing: Setup and Teardown

```typescript
import { DataSource } from 'typeorm';

describe('User Service', () => {
  let dataSource: DataSource;
  
  // Setup: Initialize connection before all tests
  beforeAll(async () => {
    dataSource = new DataSource({
      type: 'sqlite',
      database: ':memory:',
      entities: [User],
      synchronize: true,
      logging: false,
    });
    
    await dataSource.initialize();
    console.log('✅ Test database connected');
  });
  
  // Teardown: Close connection after all tests
  afterAll(async () => {
    await dataSource.destroy();
    console.log('✅ Test database closed');
  });
  
  // Clear data between tests
  afterEach(async () => {
    const entities = dataSource.entityMetadatas;
    for (const entity of entities) {
      const repository = dataSource.getRepository(entity.name);
      await repository.clear();
    }
  });
  
  test('should create user', async () => {
    const userRepo = dataSource.getRepository(User);
    const user = await userRepo.save({ name: 'John', email: 'john@example.com' });
    expect(user.id).toBeDefined();
  });
});
```

### Connection State Checking

```typescript
import { DataSource } from 'typeorm';

const AppDataSource = new DataSource({
  type: 'postgres',
  host: 'localhost',
  port: 5432,
  database: 'mydb',
  entities: [User],
});

// Check if connection is initialized
console.log('Is initialized:', AppDataSource.isInitialized);

// Initialize
await AppDataSource.initialize();
console.log('Is initialized:', AppDataSource.isInitialized); // true

// Close
await AppDataSource.destroy();
console.log('Is initialized:', AppDataSource.isInitialized); // false

// Prevent multiple initializations
async function ensureConnection(dataSource: DataSource) {
  if (!dataSource.isInitialized) {
    await dataSource.initialize();
    console.log('✅ Connection initialized');
  } else {
    console.log('ℹ️ Connection already active');
  }
}

// Prevent multiple destroys
async function ensureClosed(dataSource: DataSource) {
  if (dataSource.isInitialized) {
    await dataSource.destroy();
    console.log('✅ Connection closed');
  } else {
    console.log('ℹ️ Connection already closed');
  }
}
```

### Error Handling During Shutdown

```typescript
import { DataSource } from 'typeorm';

async function gracefulShutdown(dataSource: DataSource) {
  console.log('🛑 Initiating graceful shutdown...');
  
  try {
    // Set a timeout for closing
    const closePromise = dataSource.destroy();
    const timeoutPromise = new Promise((_, reject) => {
      setTimeout(() => reject(new Error('Shutdown timeout after 10 seconds')), 10000);
    });
    
    await Promise.race([closePromise, timeoutPromise]);
    console.log('✅ Database connection closed gracefully');
    process.exit(0);
  } catch (error) {
    console.error('❌ Error during shutdown:', error);
    
    // Force exit if graceful shutdown fails
    console.error('⚠️ Forcing shutdown...');
    process.exit(1);
  }
}

process.on('SIGTERM', () => gracefulShutdown(AppDataSource));
process.on('SIGINT', () => gracefulShutdown(AppDataSource));

// Handle uncaught errors
process.on('uncaughtException', async (error) => {
  console.error('❌ Uncaught exception:', error);
  await gracefulShutdown(AppDataSource);
});

process.on('unhandledRejection', async (reason) => {
  console.error('❌ Unhandled rejection:', reason);
  await gracefulShutdown(AppDataSource);
});
```

### Docker Container Shutdown

```typescript
// app.ts
import express from 'express';
import { AppDataSource } from './data-source';

const app = express();
let server;

async function startServer() {
  try {
    // Initialize database
    await AppDataSource.initialize();
    console.log('✅ Database connected');
    
    // Start server
    server = app.listen(3000, () => {
      console.log('🚀 Server listening on port 3000');
    });
  } catch (error) {
    console.error('❌ Failed to start server:', error);
    process.exit(1);
  }
}

async function stopServer() {
  console.log('🛑 Shutting down server...');
  
  return new Promise<void>((resolve) => {
    // Stop accepting new connections
    server.close(async () => {
      console.log('✅ Server closed');
      
      // Close database
      try {
        await AppDataSource.destroy();
        console.log('✅ Database closed');
        resolve();
      } catch (error) {
        console.error('❌ Error closing database:', error);
        resolve();
      }
    });
    
    // Force shutdown after 30 seconds
    setTimeout(() => {
      console.warn('⚠️ Forcing shutdown after timeout');
      resolve();
    }, 30000);
  });
}

// Graceful shutdown for Docker
process.on('SIGTERM', async () => {
  console.log('SIGTERM signal received');
  await stopServer();
  process.exit(0);
});

startServer();
```

### Health Check Endpoint

```typescript
import express from 'express';
import { AppDataSource } from './data-source';

const app = express();

// Health check endpoint
app.get('/health', async (req, res) => {
  try {
    // Check database connection
    if (!AppDataSource.isInitialized) {
      return res.status(503).json({
        status: 'unhealthy',
        database: 'not initialized',
      });
    }
    
    // Perform a simple query to verify connection
    await AppDataSource.query('SELECT 1');
    
    res.json({
      status: 'healthy',
      database: 'connected',
      uptime: process.uptime(),
    });
  } catch (error) {
    res.status(503).json({
      status: 'unhealthy',
      database: 'connection failed',
      error: error.message,
    });
  }
});

// Readiness check (for Kubernetes)
app.get('/ready', async (req, res) => {
  if (AppDataSource.isInitialized) {
    res.json({ ready: true });
  } else {
    res.status(503).json({ ready: false });
  }
});
```

### Best Practices Summary

```typescript
// ✅ GOOD PRACTICES

// 1. Always close connections in production
process.on('SIGTERM', async () => {
  await AppDataSource.destroy();
  process.exit(0);
});

// 2. Handle both SIGTERM and SIGINT
process.on('SIGINT', async () => {
  await AppDataSource.destroy();
  process.exit(0);
});

// 3. Set timeout for graceful shutdown
const shutdown = async () => {
  const timeout = setTimeout(() => {
    console.error('Shutdown timeout, forcing exit');
    process.exit(1);
  }, 30000);
  
  await AppDataSource.destroy();
  clearTimeout(timeout);
  process.exit(0);
};

// 4. Close connections in tests
afterAll(async () => {
  await TestDataSource.destroy();
});

// 5. Check connection state before operations
if (!AppDataSource.isInitialized) {
  await AppDataSource.initialize();
}

// ❌ BAD PRACTICES

// Don't forget to close connections
// Don't force exit without closing connections: process.exit(0)
// Don't ignore errors during shutdown
// Don't leave connections open in tests
```

**Interview Tip**: Emphasize the importance of graceful shutdown, especially in production and containerized environments. Explain that `destroy()` closes all connections in the pool and releases resources. Highlight handling both SIGTERM and SIGINT signals, setting timeouts to prevent hanging, and properly closing connections in tests. Mention that NestJS provides `enableShutdownHooks()` for automatic cleanup. A strong answer demonstrates understanding of resource management and production best practices.

</details>
