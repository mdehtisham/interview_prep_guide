# NestJS Database Integration - Top Interview Questions

## Database Fundamentals

### 1. What are the main database integration options in NestJS (TypeORM, Mongoose, Prisma, Sequelize)?

<details>
<summary>Answer</summary>

NestJS supports multiple ORM/ODM libraries for database integration.

**Main Options:**

| Library | Type | Best For | Databases |
|---------|------|----------|-----------|
| **TypeORM** | ORM | SQL databases | PostgreSQL, MySQL, SQLite, MSSQL |
| **Mongoose** | ODM | MongoDB | MongoDB only |
| **Prisma** | ORM | Modern SQL | PostgreSQL, MySQL, SQLite, MongoDB |
| **Sequelize** | ORM | Legacy SQL projects | PostgreSQL, MySQL, SQLite, MSSQL |

**TypeORM:**
```typescript
// Installation
npm install @nestjs/typeorm typeorm pg

// Setup
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'user',
      password: 'password',
      database: 'mydb',
      entities: [User],
      synchronize: true,
    }),
  ],
})
export class AppModule {}

// Entity
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;
  
  @Column()
  name: string;
}
```

**Mongoose (MongoDB):**
```typescript
// Installation
npm install @nestjs/mongoose mongoose

// Setup
import { MongooseModule } from '@nestjs/mongoose';

@Module({
  imports: [
    MongooseModule.forRoot('mongodb://localhost/mydb'),
    MongooseModule.forFeature([{ name: User.name, schema: UserSchema }]),
  ],
})
export class AppModule {}

// Schema
@Schema()
export class User {
  @Prop()
  name: string;
  
  @Prop()
  email: string;
}

export const UserSchema = SchemaFactory.createForClass(User);
```

**Prisma:**
```typescript
// Installation
npm install @prisma/client
npm install -D prisma

// Setup prisma/schema.prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id    Int    @id @default(autoincrement())
  name  String
  email String @unique
}

// Service
@Injectable()
export class UsersService {
  constructor(private prisma: PrismaService) {}
  
  async findAll() {
    return this.prisma.user.findMany();
  }
}
```

**Sequelize:**
```typescript
// Installation
npm install @nestjs/sequelize sequelize sequelize-typescript pg

// Setup
import { SequelizeModule } from '@nestjs/sequelize';

@Module({
  imports: [
    SequelizeModule.forRoot({
      dialect: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'user',
      password: 'password',
      database: 'mydb',
      models: [User],
    }),
  ],
})
export class AppModule {}

// Model
@Table
export class User extends Model {
  @Column
  name: string;
  
  @Column
  email: string;
}
```

**Comparison:**

| Feature | TypeORM | Mongoose | Prisma | Sequelize |
|---------|---------|----------|--------|-----------|
| Learning Curve | Medium | Easy | Easy | Medium |
| TypeScript Support | ✅ Excellent | ✅ Good | ✅ Excellent | ⚠️ Fair |
| Active Development | ✅ Yes | ✅ Yes | ✅ Yes | ⚠️ Declining |
| Migrations | ✅ Built-in | ❌ Manual | ✅ Built-in | ✅ Built-in |
| Query Builder | ✅ Yes | ❌ No | ✅ Yes | ✅ Yes |
| NestJS Integration | ✅ Official | ✅ Official | ⚠️ Community | ✅ Official |

**When to Use Each:**
```typescript
// TypeORM - Most versatile, SQL databases
// ✅ Use when: Working with relational databases
// ✅ Use when: Need complex queries and relationships
// ✅ Use when: Want decorator-based approach

// Mongoose - MongoDB ODM
// ✅ Use when: Working with MongoDB
// ✅ Use when: Need flexible schema
// ✅ Use when: Working with document-based data

// Prisma - Modern ORM
// ✅ Use when: Starting new project
// ✅ Use when: Want type-safe queries
// ✅ Use when: Need excellent TypeScript support

// Sequelize - Legacy ORM
// ⚠️ Use when: Maintaining legacy project
// ⚠️ Use when: Already familiar with Sequelize
```

**Interview Tip**: NestJS officially supports TypeORM, Mongoose, and Sequelize. TypeORM most popular for SQL, Mongoose for MongoDB. Prisma is modern alternative with excellent TypeScript support. Choose based on database type and project requirements.

</details>

### 2. What is TypeORM and when do you use it?

<details>
<summary>Answer</summary>

**TypeORM** is an Object-Relational Mapping (ORM) library for TypeScript/JavaScript that works with SQL databases.

**Key Features:**
1. Decorator-based entities
2. Repository pattern
3. Query builder
4. Migrations
5. Relationships
6. Transactions
7. TypeScript support

**Basic Setup:**
```typescript
// Installation
npm install @nestjs/typeorm typeorm pg

// app.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'postgres',
      password: 'password',
      database: 'mydb',
      entities: [__dirname + '/**/*.entity{.ts,.js}'],
      synchronize: true, // ⚠️ Don't use in production
    }),
  ],
})
export class AppModule {}
```

**Entity Definition:**
```typescript
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

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
  
  @Column({ default: true })
  isActive: boolean;
  
  @Column({ type: 'timestamp', default: () => 'CURRENT_TIMESTAMP' })
  createdAt: Date;
}
```

**Repository Usage:**
```typescript
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './user.entity';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>,
  ) {}
  
  async findAll(): Promise<User[]> {
    return this.usersRepository.find();
  }
  
  async findOne(id: number): Promise<User> {
    return this.usersRepository.findOne({ where: { id } });
  }
  
  async create(userData: Partial<User>): Promise<User> {
    const user = this.usersRepository.create(userData);
    return this.usersRepository.save(user);
  }
  
  async update(id: number, userData: Partial<User>): Promise<void> {
    await this.usersRepository.update(id, userData);
  }
  
  async remove(id: number): Promise<void> {
    await this.usersRepository.delete(id);
  }
}
```

**Relationships:**
```typescript
// One-to-Many
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
  
  @ManyToOne(() => User, user => user.posts)
  author: User;
}

// Usage with relations
const user = await this.usersRepository.findOne({
  where: { id: 1 },
  relations: ['posts'],
});
```

**Query Builder:**
```typescript
async findActiveUsers(): Promise<User[]> {
  return this.usersRepository
    .createQueryBuilder('user')
    .where('user.isActive = :isActive', { isActive: true })
    .andWhere('user.email LIKE :email', { email: '%@example.com' })
    .orderBy('user.createdAt', 'DESC')
    .getMany();
}
```

**Transactions:**
```typescript
async transferMoney(fromId: number, toId: number, amount: number): Promise<void> {
  const queryRunner = this.dataSource.createQueryRunner();
  await queryRunner.connect();
  await queryRunner.startTransaction();
  
  try {
    const fromUser = await queryRunner.manager.findOne(User, { where: { id: fromId } });
    const toUser = await queryRunner.manager.findOne(User, { where: { id: toId } });
    
    fromUser.balance -= amount;
    toUser.balance += amount;
    
    await queryRunner.manager.save(fromUser);
    await queryRunner.manager.save(toUser);
    
    await queryRunner.commitTransaction();
  } catch (err) {
    await queryRunner.rollbackTransaction();
    throw err;
  } finally {
    await queryRunner.release();
  }
}
```

**When to Use TypeORM:**

**✅ Use TypeORM when:**
- Working with SQL databases (PostgreSQL, MySQL, SQLite, MSSQL)
- Need complex relationships (one-to-many, many-to-many)
- Want decorator-based approach
- Need migrations support
- Require query builder for complex queries
- Want repository pattern

**❌ Don't use TypeORM when:**
- Working with MongoDB (use Mongoose instead)
- Need maximum performance (consider raw queries)
- Want simple key-value storage (use Redis)
- Project requires NoSQL flexibility

**Interview Tip**: TypeORM is most popular ORM for NestJS with SQL databases. Provides decorator-based entities, repository pattern, query builder, migrations. Best for relational databases with complex relationships. Use `@Entity()` for models, `@InjectRepository()` for injection. Supports transactions, eager/lazy loading, cascades.

</details>

### 3. What is Mongoose and when do you use it?

<details>
<summary>Answer</summary>

**Mongoose** is an Object Document Mapper (ODM) for MongoDB and Node.js. It provides schema-based modeling for MongoDB documents.

**Key Features:**
1. Schema definitions
2. Validation
3. Middleware hooks
4. Population (joins)
5. Virtual properties
6. TypeScript support

**Installation:**
```bash
npm install @nestjs/mongoose mongoose
```

**Setup:**
```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { MongooseModule } from '@nestjs/mongoose';

@Module({
  imports: [
    MongooseModule.forRoot('mongodb://localhost:27017/mydb'),
  ],
})
export class AppModule {}

// With options
@Module({
  imports: [
    MongooseModule.forRoot('mongodb://localhost:27017/mydb', {
      useNewUrlParser: true,
      useUnifiedTopology: true,
    }),
  ],
})
export class AppModule {}
```

**Schema Definition:**
```typescript
import { Prop, Schema, SchemaFactory } from '@nestjs/mongoose';
import { Document } from 'mongoose';

@Schema({ timestamps: true })
export class User extends Document {
  @Prop({ required: true })
  name: string;
  
  @Prop({ required: true, unique: true })
  email: string;
  
  @Prop()
  age: number;
  
  @Prop({ default: true })
  isActive: boolean;
  
  @Prop({ type: [String] })
  tags: string[];
}

export const UserSchema = SchemaFactory.createForClass(User);
```

**Module Registration:**
```typescript
import { Module } from '@nestjs/common';
import { MongooseModule } from '@nestjs/mongoose';
import { UsersService } from './users.service';
import { UsersController } from './users.controller';
import { User, UserSchema } from './user.schema';

@Module({
  imports: [
    MongooseModule.forFeature([{ name: User.name, schema: UserSchema }]),
  ],
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}
```

**Service with CRUD Operations:**
```typescript
import { Injectable } from '@nestjs/common';
import { InjectModel } from '@nestjs/mongoose';
import { Model } from 'mongoose';
import { User } from './user.schema';

@Injectable()
export class UsersService {
  constructor(
    @InjectModel(User.name)
    private userModel: Model<User>,
  ) {}
  
  async create(createUserDto: any): Promise<User> {
    const user = new this.userModel(createUserDto);
    return user.save();
  }
  
  async findAll(): Promise<User[]> {
    return this.userModel.find().exec();
  }
  
  async findOne(id: string): Promise<User> {
    return this.userModel.findById(id).exec();
  }
  
  async update(id: string, updateUserDto: any): Promise<User> {
    return this.userModel
      .findByIdAndUpdate(id, updateUserDto, { new: true })
      .exec();
  }
  
  async remove(id: string): Promise<User> {
    return this.userModel.findByIdAndDelete(id).exec();
  }
}
```

**Relationships (References):**
```typescript
// User schema with posts reference
@Schema()
export class User extends Document {
  @Prop()
  name: string;
  
  @Prop({ type: [{ type: mongoose.Schema.Types.ObjectId, ref: 'Post' }] })
  posts: Post[];
}

// Post schema with author reference
@Schema()
export class Post extends Document {
  @Prop()
  title: string;
  
  @Prop({ type: mongoose.Schema.Types.ObjectId, ref: 'User' })
  author: User;
}

export const UserSchema = SchemaFactory.createForClass(User);
export const PostSchema = SchemaFactory.createForClass(Post);
```

**Population (Similar to SQL Joins):**
```typescript
@Injectable()
export class UsersService {
  constructor(
    @InjectModel(User.name) private userModel: Model<User>,
  ) {}
  
  async findUserWithPosts(id: string): Promise<User> {
    return this.userModel
      .findById(id)
      .populate('posts') // Load referenced posts
      .exec();
  }
  
  async findPostWithAuthor(postId: string): Promise<Post> {
    return this.postModel
      .findById(postId)
      .populate('author') // Load referenced author
      .exec();
  }
  
  // Nested population
  async findUserWithPostsAndComments(id: string): Promise<User> {
    return this.userModel
      .findById(id)
      .populate({
        path: 'posts',
        populate: { path: 'comments' },
      })
      .exec();
  }
}
```

**Validation:**
```typescript
@Schema()
export class User extends Document {
  @Prop({
    required: [true, 'Name is required'],
    minlength: [3, 'Name must be at least 3 characters'],
    maxlength: [50, 'Name cannot exceed 50 characters'],
  })
  name: string;
  
  @Prop({
    required: true,
    unique: true,
    lowercase: true,
    match: [/^\S+@\S+\.\S+$/, 'Please enter a valid email'],
  })
  email: string;
  
  @Prop({
    min: [18, 'Must be at least 18 years old'],
    max: [120, 'Age cannot exceed 120'],
  })
  age: number;
}
```

**Middleware Hooks:**
```typescript
@Schema()
export class User extends Document {
  @Prop()
  password: string;
}

export const UserSchema = SchemaFactory.createForClass(User);

// Pre-save hook
UserSchema.pre('save', async function(next) {
  if (this.isModified('password')) {
    this.password = await bcrypt.hash(this.password, 10);
  }
  next();
});

// Post-find hook
UserSchema.post('find', function(docs) {
  console.log('Found users:', docs.length);
});
```

**When to Use Mongoose:**

**✅ Use Mongoose when:**
- Working with MongoDB
- Need flexible schema
- Want document-based data model
- Require schema validation
- Need middleware hooks
- Working with nested documents
- Want easy population (joins)

**❌ Don't use Mongoose when:**
- Working with SQL databases (use TypeORM)
- Need strong ACID guarantees
- Require complex transactions
- Working with relational data

**Interview Tip**: Mongoose is ODM for MongoDB in NestJS. Use `@Schema()` for models, `@Prop()` for fields, `@InjectModel()` for injection. Supports validation, middleware hooks, population (similar to joins). Best for document-based data with flexible schema. Use `populate()` for loading references.

</details>

### 4. What databases does NestJS support?

<details>
<summary>Answer</summary>

NestJS supports a wide range of SQL and NoSQL databases through various ORMs/ODMs.

**SQL Databases (via TypeORM):**

| Database | Driver | Use Case |
|----------|--------|----------|
| **PostgreSQL** | `pg` | Production apps, complex queries |
| **MySQL** | `mysql2` | Web applications, WordPress-like |
| **MariaDB** | `mysql2` | MySQL alternative |
| **SQLite** | `sqlite3` | Development, testing, mobile |
| **Microsoft SQL Server** | `mssql` | Enterprise, Windows environments |
| **Oracle** | `oracledb` | Enterprise applications |
| **CockroachDB** | `pg` | Distributed SQL |

**NoSQL Databases:**

| Database | Library | Use Case |
|----------|---------|----------|
| **MongoDB** | Mongoose | Document storage, flexible schema |
| **Redis** | `ioredis` | Caching, sessions, real-time |
| **Cassandra** | `cassandra-driver` | Time-series, large-scale |
| **DynamoDB** | AWS SDK | Serverless applications |

**PostgreSQL Setup:**
```typescript
// Installation
npm install @nestjs/typeorm typeorm pg

// Configuration
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
      synchronize: true,
    }),
  ],
})
export class AppModule {}
```

**MySQL Setup:**
```typescript
// Installation
npm install @nestjs/typeorm typeorm mysql2

// Configuration
@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'mysql',
      host: 'localhost',
      port: 3306,
      username: 'root',
      password: 'password',
      database: 'mydb',
      entities: [User],
      synchronize: true,
    }),
  ],
})
export class AppModule {}
```

**SQLite Setup:**
```typescript
// Installation
npm install @nestjs/typeorm typeorm sqlite3

// Configuration
@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'sqlite',
      database: 'mydb.sqlite',
      entities: [User],
      synchronize: true,
    }),
  ],
})
export class AppModule {}
```

**MongoDB Setup:**
```typescript
// Installation
npm install @nestjs/mongoose mongoose

// Configuration
@Module({
  imports: [
    MongooseModule.forRoot('mongodb://localhost:27017/mydb'),
  ],
})
export class AppModule {}
```

**Redis Setup:**
```typescript
// Installation
npm install ioredis

// Configuration
import { Module } from '@nestjs/common';
import { RedisModule } from '@nestjs-modules/ioredis';

@Module({
  imports: [
    RedisModule.forRoot({
      config: {
        host: 'localhost',
        port: 6379,
      },
    }),
  ],
})
export class AppModule {}

// Usage
@Injectable()
export class CacheService {
  constructor(
    @InjectRedis() private readonly redis: Redis,
  ) {}
  
  async set(key: string, value: string): Promise<void> {
    await this.redis.set(key, value);
  }
  
  async get(key: string): Promise<string> {
    return this.redis.get(key);
  }
}
```

**Multiple Databases:**
```typescript
@Module({
  imports: [
    // PostgreSQL
    TypeOrmModule.forRoot({
      name: 'default',
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      database: 'maindb',
      entities: [User],
    }),
    // MongoDB
    MongooseModule.forRoot('mongodb://localhost:27017/logsdb'),
    // Redis
    RedisModule.forRoot({
      config: { host: 'localhost', port: 6379 },
    }),
  ],
})
export class AppModule {}
```

**Database Comparison:**

| Database | Type | Best For | Scalability | ACID |
|----------|------|----------|-------------|------|
| PostgreSQL | SQL | Complex queries, JSON | Vertical | ✅ Yes |
| MySQL | SQL | Web apps, read-heavy | Vertical | ✅ Yes |
| MongoDB | NoSQL | Flexible schema, documents | Horizontal | ⚠️ Limited |
| Redis | Key-Value | Caching, sessions | Horizontal | ❌ No |
| SQLite | SQL | Development, testing | Single-file | ✅ Yes |

**Choosing the Right Database:**

```typescript
// PostgreSQL - Complex apps, strong consistency
// ✅ Use for: Financial apps, complex relationships
@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      // ... config
    }),
  ],
})

// MongoDB - Flexible schema, document storage
// ✅ Use for: Content management, catalogs
@Module({
  imports: [
    MongooseModule.forRoot('mongodb://...'),
  ],
})

// Redis - High-speed caching
// ✅ Use for: Sessions, caching, real-time
@Module({
  imports: [
    RedisModule.forRoot({ ... }),
  ],
})

// SQLite - Development and testing
// ✅ Use for: Local development, mobile apps
@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'sqlite',
      database: ':memory:',
    }),
  ],
})
```

**Interview Tip**: NestJS supports SQL (PostgreSQL, MySQL, SQLite, MSSQL) via TypeORM and NoSQL (MongoDB) via Mongoose. Also supports Redis for caching. PostgreSQL most popular for production. Use TypeORM for SQL, Mongoose for MongoDB. Can configure multiple databases in same app.

</details>

## TypeORM Setup

### 5. How do you install and configure TypeORM in NestJS?

<details>
<summary>Answer</summary>

**Installation Steps:**

**1. Install Dependencies:**
```bash
# For PostgreSQL
npm install @nestjs/typeorm typeorm pg

# For MySQL
npm install @nestjs/typeorm typeorm mysql2

# For SQLite
npm install @nestjs/typeorm typeorm sqlite3

# For MSSQL
npm install @nestjs/typeorm typeorm mssql
```

**2. Basic Configuration:**
```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'postgres',
      password: 'password',
      database: 'mydb',
      entities: [__dirname + '/**/*.entity{.ts,.js}'],
      synchronize: true, // ⚠️ Don't use in production!
    }),
  ],
})
export class AppModule {}
```

**3. Environment-Based Configuration:**
```typescript
// Installation
npm install @nestjs/config

// .env file
DB_HOST=localhost
DB_PORT=5432
DB_USERNAME=postgres
DB_PASSWORD=password
DB_DATABASE=mydb

// app.module.ts
import { ConfigModule, ConfigService } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
    }),
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: (configService: ConfigService) => ({
        type: 'postgres',
        host: configService.get('DB_HOST'),
        port: +configService.get('DB_PORT'),
        username: configService.get('DB_USERNAME'),
        password: configService.get('DB_PASSWORD'),
        database: configService.get('DB_DATABASE'),
        entities: [__dirname + '/**/*.entity{.ts,.js}'],
        synchronize: configService.get('NODE_ENV') === 'development',
      }),
      inject: [ConfigService],
    }),
  ],
})
export class AppModule {}
```

**4. Separate Configuration File:**
```typescript
// ormconfig.ts
import { TypeOrmModuleOptions } from '@nestjs/typeorm';

export const typeOrmConfig: TypeOrmModuleOptions = {
  type: 'postgres',
  host: process.env.DB_HOST || 'localhost',
  port: parseInt(process.env.DB_PORT) || 5432,
  username: process.env.DB_USERNAME || 'postgres',
  password: process.env.DB_PASSWORD || 'password',
  database: process.env.DB_DATABASE || 'mydb',
  entities: [__dirname + '/../**/*.entity{.ts,.js}'],
  synchronize: false,
  logging: process.env.NODE_ENV === 'development',
  migrations: [__dirname + '/../migrations/*{.ts,.js}'],
  migrationsRun: true,
};

// app.module.ts
import { typeOrmConfig } from './config/ormconfig';

@Module({
  imports: [
    TypeOrmModule.forRoot(typeOrmConfig),
  ],
})
export class AppModule {}
```

**5. Multiple Database Connections:**
```typescript
@Module({
  imports: [
    // Default connection
    TypeOrmModule.forRoot({
      name: 'default',
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'postgres',
      password: 'password',
      database: 'maindb',
      entities: [User, Post],
    }),
    // Secondary connection
    TypeOrmModule.forRoot({
      name: 'analytics',
      type: 'mysql',
      host: 'localhost',
      port: 3306,
      username: 'root',
      password: 'password',
      database: 'analyticsdb',
      entities: [Log, Event],
    }),
  ],
})
export class AppModule {}
```

**6. Configuration Options:**
```typescript
TypeOrmModule.forRoot({
  // Connection
  type: 'postgres',
  host: 'localhost',
  port: 5432,
  username: 'postgres',
  password: 'password',
  database: 'mydb',
  
  // Entities
  entities: [User, Post, Comment],
  // Or auto-load
  entities: [__dirname + '/**/*.entity{.ts,.js}'],
  
  // Schema management
  synchronize: false, // ⚠️ NEVER true in production
  migrationsRun: true,
  migrations: ['src/migrations/*{.ts,.js}'],
  
  // Logging
  logging: true,
  logger: 'advanced-console',
  
  // Connection pool
  extra: {
    max: 10, // Maximum connections
    min: 2,  // Minimum connections
  },
  
  // SSL (for production)
  ssl: {
    rejectUnauthorized: false,
  },
  
  // Timezone
  timezone: 'Z',
  
  // Cache
  cache: {
    duration: 30000, // 30 seconds
  },
})
```

**7. Production Configuration:**
```typescript
@Module({
  imports: [
    TypeOrmModule.forRootAsync({
      useFactory: () => ({
        type: 'postgres',
        host: process.env.DB_HOST,
        port: +process.env.DB_PORT,
        username: process.env.DB_USERNAME,
        password: process.env.DB_PASSWORD,
        database: process.env.DB_DATABASE,
        entities: [__dirname + '/**/*.entity{.ts,.js}'],
        
        // ⚠️ Production settings
        synchronize: false, // NEVER true in production
        migrationsRun: true,
        logging: ['error', 'warn'],
        
        // Connection pool
        extra: {
          max: 20,
          min: 5,
          idleTimeoutMillis: 30000,
        },
        
        // SSL
        ssl: {
          rejectUnauthorized: true,
          ca: process.env.DB_SSL_CA,
        },
      }),
    }),
  ],
})
export class AppModule {}
```

**Interview Tip**: Install `@nestjs/typeorm`, `typeorm`, and database driver. Use `TypeOrmModule.forRoot()` for configuration. Use `TypeOrmModule.forRootAsync()` with `ConfigService` for environment variables. Set `synchronize: false` in production. Use migrations instead. Can configure multiple databases with `name` property.

</details>

### 6. What is `TypeOrmModule` and how do you use it?

<details>
<summary>Answer</summary>

`TypeOrmModule` is NestJS's integration package for TypeORM. It provides decorators and utilities to configure and use TypeORM in NestJS.

**Main Methods:**

| Method | Purpose | Scope |
|--------|---------|-------|
| `forRoot()` | Configure database connection | AppModule (global) |
| `forRootAsync()` | Configure with async providers | AppModule (global) |
| `forFeature()` | Register entities/repositories | Feature modules |

**1. TypeOrmModule.forRoot() - Global Configuration:**
```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'postgres',
      password: 'password',
      database: 'mydb',
      entities: [User, Post],
      synchronize: true,
    }),
    UsersModule,
    PostsModule,
  ],
})
export class AppModule {}
```

**2. TypeOrmModule.forRootAsync() - Async Configuration:**
```typescript
import { ConfigModule, ConfigService } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot(),
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: (configService: ConfigService) => ({
        type: 'postgres',
        host: configService.get('DB_HOST'),
        port: configService.get('DB_PORT'),
        username: configService.get('DB_USERNAME'),
        password: configService.get('DB_PASSWORD'),
        database: configService.get('DB_DATABASE'),
        entities: [__dirname + '/**/*.entity{.ts,.js}'],
        synchronize: false,
      }),
      inject: [ConfigService],
    }),
  ],
})
export class AppModule {}
```

**3. TypeOrmModule.forFeature() - Register Entities:**
```typescript
// users.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { User } from './user.entity';
import { UsersService } from './users.service';
import { UsersController } from './users.controller';

@Module({
  imports: [
    TypeOrmModule.forFeature([User]), // Register User entity
  ],
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}

// Multiple entities
@Module({
  imports: [
    TypeOrmModule.forFeature([User, Post, Comment]),
  ],
  // ...
})
export class BlogModule {}
```

**4. Using Repository in Service:**
```typescript
// users.service.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './user.entity';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>,
  ) {}
  
  async findAll(): Promise<User[]> {
    return this.usersRepository.find();
  }
  
  async create(userData: Partial<User>): Promise<User> {
    const user = this.usersRepository.create(userData);
    return this.usersRepository.save(user);
  }
}
```

**5. Multiple Database Connections:**
```typescript
// app.module.ts
@Module({
  imports: [
    TypeOrmModule.forRoot({
      name: 'default',
      type: 'postgres',
      database: 'maindb',
      entities: [User],
    }),
    TypeOrmModule.forRoot({
      name: 'analytics',
      type: 'mysql',
      database: 'analyticsdb',
      entities: [Log],
    }),
  ],
})
export class AppModule {}

// users.module.ts - Uses default connection
@Module({
  imports: [
    TypeOrmModule.forFeature([User]),
  ],
})
export class UsersModule {}

// logs.module.ts - Uses analytics connection
@Module({
  imports: [
    TypeOrmModule.forFeature([Log], 'analytics'),
  ],
})
export class LogsModule {}

// logs.service.ts
@Injectable()
export class LogsService {
  constructor(
    @InjectRepository(Log, 'analytics')
    private logsRepository: Repository<Log>,
  ) {}
}
```

**6. Custom Repository:**
```typescript
// user.repository.ts
import { EntityRepository, Repository } from 'typeorm';
import { User } from './user.entity';

@EntityRepository(User)
export class UserRepository extends Repository<User> {
  async findByEmail(email: string): Promise<User> {
    return this.findOne({ where: { email } });
  }
  
  async findActiveUsers(): Promise<User[]> {
    return this.find({ where: { isActive: true } });
  }
}

// users.module.ts
@Module({
  imports: [
    TypeOrmModule.forFeature([User, UserRepository]),
  ],
})
export class UsersModule {}

// users.service.ts
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(UserRepository)
    private userRepository: UserRepository,
  ) {}
  
  async findByEmail(email: string): Promise<User> {
    return this.userRepository.findByEmail(email);
  }
}
```

**7. Export and Share Repositories:**
```typescript
// users.module.ts
@Module({
  imports: [
    TypeOrmModule.forFeature([User]),
  ],
  providers: [UsersService],
  exports: [
    TypeOrmModule, // Export to make User repository available
    UsersService,
  ],
})
export class UsersModule {}

// posts.module.ts - Can now use User repository
@Module({
  imports: [
    TypeOrmModule.forFeature([Post]),
    UsersModule, // Import UsersModule
  ],
  providers: [PostsService],
})
export class PostsModule {}

// posts.service.ts
@Injectable()
export class PostsService {
  constructor(
    @InjectRepository(Post)
    private postsRepository: Repository<Post>,
    @InjectRepository(User) // Can inject User repository
    private usersRepository: Repository<User>,
  ) {}
}
```

**Interview Tip**: `TypeOrmModule` has three main methods: `forRoot()` for global DB config, `forRootAsync()` for async config with DI, `forFeature()` to register entities in feature modules. Use `@InjectRepository()` to inject repositories. Export `TypeOrmModule` from feature modules to share repositories. Support multiple databases with `name` parameter.

</details>

### 7. How do you configure database connection using `TypeOrmModule.forRoot()`?

<details>
<summary>Answer</summary>

`TypeOrmModule.forRoot()` configures the database connection with various options.

**Basic Configuration:**
```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'postgres',
      password: 'password',
      database: 'mydb',
      entities: [User, Post],
      synchronize: true,
    }),
  ],
})
export class AppModule {}
```

**All Configuration Options:**
```typescript
TypeOrmModule.forRoot({
  // Database type
  type: 'postgres', // 'mysql' | 'mariadb' | 'sqlite' | 'mssql' | etc.
  
  // Connection details
  host: 'localhost',
  port: 5432,
  username: 'postgres',
  password: 'password',
  database: 'mydb',
  
  // Entities
  entities: [User, Post, Comment],
  // Or auto-discover
  entities: ['dist/**/*.entity{.ts,.js}'],
  
  // Schema synchronization
  synchronize: false, // ⚠️ NEVER true in production!
  
  // Migrations
  migrations: ['dist/migrations/*{.ts,.js}'],
  migrationsTableName: 'migrations',
  migrationsRun: true,
  
  // Logging
  logging: true, // or ['query', 'error', 'schema', 'warn', 'info', 'log']
  logger: 'advanced-console', // or 'simple-console' | 'file' | 'debug'
  
  // Connection pool
  extra: {
    max: 10, // max connections
    min: 2,  // min connections
    idleTimeoutMillis: 30000,
    connectionTimeoutMillis: 2000,
  },
  
  // Connection options
  connectTimeoutMS: 10000,
  
  // Timezone
  timezone: 'Z', // UTC
  
  // SSL
  ssl: {
    rejectUnauthorized: false,
  },
  
  // Cache
  cache: {
    duration: 30000, // cache duration in milliseconds
    type: 'database',
    tableName: 'query_cache',
  },
  
  // Name (for multiple connections)
  name: 'default',
  
  // Retry attempts
  retryAttempts: 10,
  retryDelay: 3000,
  
  // Auto load entities
  autoLoadEntities: true,
  
  // Keep connection alive
  keepConnectionAlive: true,
})
```

**PostgreSQL Configuration:**
```typescript
TypeOrmModule.forRoot({
  type: 'postgres',
  host: 'localhost',
  port: 5432,
  username: 'postgres',
  password: 'password',
  database: 'mydb',
  entities: [__dirname + '/**/*.entity{.ts,.js}'],
  synchronize: false,
  logging: true,
  ssl: process.env.NODE_ENV === 'production' ? {
    rejectUnauthorized: false,
  } : false,
})
```

**MySQL Configuration:**
```typescript
TypeOrmModule.forRoot({
  type: 'mysql',
  host: 'localhost',
  port: 3306,
  username: 'root',
  password: 'password',
  database: 'mydb',
  entities: [User, Post],
  charset: 'utf8mb4',
  timezone: '+00:00',
  extra: {
    connectionLimit: 10,
  },
})
```

**SQLite Configuration:**
```typescript
TypeOrmModule.forRoot({
  type: 'sqlite',
  database: 'mydb.sqlite',
  entities: [User, Post],
  synchronize: true, // OK for SQLite in dev
})

// In-memory database (for testing)
TypeOrmModule.forRoot({
  type: 'sqlite',
  database: ':memory:',
  entities: [User, Post],
  synchronize: true,
  dropSchema: true,
})
```

**Development vs Production:**
```typescript
// Development
TypeOrmModule.forRoot({
  type: 'postgres',
  host: 'localhost',
  port: 5432,
  username: 'postgres',
  password: 'password',
  database: 'mydb_dev',
  entities: [__dirname + '/**/*.entity{.ts,.js}'],
  synchronize: true, // Auto-sync schema
  logging: true, // Log all queries
  dropSchema: false,
})

// Production
TypeOrmModule.forRoot({
  type: 'postgres',
  host: process.env.DB_HOST,
  port: +process.env.DB_PORT,
  username: process.env.DB_USERNAME,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_DATABASE,
  entities: [__dirname + '/**/*.entity{.ts,.js}'],
  synchronize: false, // ⚠️ NEVER true in production!
  migrationsRun: true,
  logging: ['error'], // Only log errors
  ssl: {
    rejectUnauthorized: true,
    ca: process.env.DB_SSL_CA,
  },
  extra: {
    max: 20,
    min: 5,
  },
})
```

**With Connection URI:**
```typescript
TypeOrmModule.forRoot({
  type: 'postgres',
  url: 'postgresql://user:password@localhost:5432/mydb',
  entities: [User, Post],
  synchronize: false,
})

// Or from environment
TypeOrmModule.forRoot({
  type: 'postgres',
  url: process.env.DATABASE_URL,
  entities: [__dirname + '/**/*.entity{.ts,.js}'],
  ssl: {
    rejectUnauthorized: false,
  },
})
```

**Auto Load Entities:**
```typescript
// Instead of listing all entities
entities: [User, Post, Comment, Like, Follow, ...]

// Use autoLoadEntities
TypeOrmModule.forRoot({
  type: 'postgres',
  host: 'localhost',
  port: 5432,
  username: 'postgres',
  password: 'password',
  database: 'mydb',
  autoLoadEntities: true, // Automatically discover entities from forFeature()
  synchronize: false,
})

// Then in feature modules
@Module({
  imports: [
    TypeOrmModule.forFeature([User]), // Auto-loaded
  ],
})
export class UsersModule {}
```

**Interview Tip**: `TypeOrmModule.forRoot()` configures DB connection. Required options: `type`, `host`, `port`, `username`, `password`, `database`, `entities`. Set `synchronize: false` in production. Use `logging: true` in development. Configure SSL for production. Use `autoLoadEntities: true` to auto-discover entities. Use connection pool with `extra.max` and `extra.min`.

</details>

### 8. How do you use environment variables for database configuration?

<details>
<summary>Answer</summary>

Use `@nestjs/config` to load environment variables and configure TypeORM dynamically.

**1. Install ConfigModule:**
```bash
npm install @nestjs/config
```

**2. Create .env File:**
```env
# .env
NODE_ENV=development

DB_TYPE=postgres
DB_HOST=localhost
DB_PORT=5432
DB_USERNAME=postgres
DB_PASSWORD=password
DB_DATABASE=mydb
DB_SYNCHRONIZE=false
DB_LOGGING=true

# For production
# DATABASE_URL=postgresql://user:pass@host:5432/db
```

**3. Basic Setup with ConfigModule:**
```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    // Load .env file
    ConfigModule.forRoot({
      isGlobal: true, // Make ConfigService available globally
      envFilePath: '.env',
    }),
    
    // Configure TypeORM with environment variables
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: (configService: ConfigService) => ({
        type: configService.get('DB_TYPE') as any,
        host: configService.get('DB_HOST'),
        port: +configService.get<number>('DB_PORT'),
        username: configService.get('DB_USERNAME'),
        password: configService.get('DB_PASSWORD'),
        database: configService.get('DB_DATABASE'),
        entities: [__dirname + '/**/*.entity{.ts,.js}'],
        synchronize: configService.get('DB_SYNCHRONIZE') === 'true',
        logging: configService.get('DB_LOGGING') === 'true',
      }),
      inject: [ConfigService],
    }),
  ],
})
export class AppModule {}
```

**4. Multiple Environment Files:**
```typescript
// .env.development
DB_HOST=localhost
DB_PORT=5432
DB_DATABASE=mydb_dev
DB_SYNCHRONIZE=true

// .env.production
DB_HOST=prod.database.com
DB_PORT=5432
DB_DATABASE=mydb_prod
DB_SYNCHRONIZE=false

// .env.test
DB_HOST=localhost
DB_PORT=5432
DB_DATABASE=mydb_test
DB_SYNCHRONIZE=true

// app.module.ts
ConfigModule.forRoot({
  isGlobal: true,
  envFilePath: `.env.${process.env.NODE_ENV || 'development'}`,
})
```

**5. Separate Database Config File:**
```typescript
// config/database.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('database', () => ({
  type: process.env.DB_TYPE || 'postgres',
  host: process.env.DB_HOST || 'localhost',
  port: parseInt(process.env.DB_PORT, 10) || 5432,
  username: process.env.DB_USERNAME || 'postgres',
  password: process.env.DB_PASSWORD || 'password',
  database: process.env.DB_DATABASE || 'mydb',
  synchronize: process.env.DB_SYNCHRONIZE === 'true',
  logging: process.env.DB_LOGGING === 'true',
  entities: ['dist/**/*.entity{.ts,.js}'],
  migrations: ['dist/migrations/*{.ts,.js}'],
  ssl: process.env.DB_SSL === 'true' ? {
    rejectUnauthorized: false,
  } : false,
}));

// app.module.ts
import databaseConfig from './config/database.config';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      load: [databaseConfig],
    }),
    TypeOrmModule.forRootAsync({
      useFactory: (configService: ConfigService) => ({
        ...configService.get('database'),
      }),
      inject: [ConfigService],
    }),
  ],
})
export class AppModule {}
```

**6. Validation with Class Validator:**
```bash
npm install class-validator class-transformer
```

```typescript
// config/env.validation.ts
import { plainToClass } from 'class-transformer';
import { IsString, IsNumber, IsBoolean, validateSync } from 'class-validator';

class EnvironmentVariables {
  @IsString()
  DB_HOST: string;
  
  @IsNumber()
  DB_PORT: number;
  
  @IsString()
  DB_USERNAME: string;
  
  @IsString()
  DB_PASSWORD: string;
  
  @IsString()
  DB_DATABASE: string;
  
  @IsBoolean()
  DB_SYNCHRONIZE: boolean;
}

export function validate(config: Record<string, unknown>) {
  const validatedConfig = plainToClass(EnvironmentVariables, config, {
    enableImplicitConversion: true,
  });
  
  const errors = validateSync(validatedConfig, {
    skipMissingProperties: false,
  });
  
  if (errors.length > 0) {
    throw new Error(errors.toString());
  }
  
  return validatedConfig;
}

// app.module.ts
ConfigModule.forRoot({
  isGlobal: true,
  validate,
})
```

**7. Using DATABASE_URL (Common in Production):**
```env
# .env
DATABASE_URL=postgresql://user:password@localhost:5432/mydb?sslmode=require
```

```typescript
TypeOrmModule.forRootAsync({
  useFactory: (configService: ConfigService) => {
    const databaseUrl = configService.get('DATABASE_URL');
    
    if (databaseUrl) {
      // Parse DATABASE_URL
      return {
        type: 'postgres',
        url: databaseUrl,
        entities: [__dirname + '/**/*.entity{.ts,.js}'],
        synchronize: false,
        ssl: {
          rejectUnauthorized: false,
        },
      };
    }
    
    // Fallback to individual variables
    return {
      type: 'postgres',
      host: configService.get('DB_HOST'),
      port: +configService.get('DB_PORT'),
      username: configService.get('DB_USERNAME'),
      password: configService.get('DB_PASSWORD'),
      database: configService.get('DB_DATABASE'),
      entities: [__dirname + '/**/*.entity{.ts,.js}'],
      synchronize: false,
    };
  },
  inject: [ConfigService],
})
```

**8. Complete Production Example:**
```typescript
// .env.production
NODE_ENV=production
DATABASE_URL=postgresql://user:pass@prod-db.com:5432/mydb
DB_SSL=true
DB_POOL_MAX=20
DB_POOL_MIN=5
DB_LOGGING=false

// app.module.ts
@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: `.env.${process.env.NODE_ENV}`,
    }),
    TypeOrmModule.forRootAsync({
      useFactory: (configService: ConfigService) => ({
        type: 'postgres',
        url: configService.get('DATABASE_URL'),
        entities: [__dirname + '/**/*.entity{.ts,.js}'],
        synchronize: false,
        migrationsRun: true,
        logging: configService.get('DB_LOGGING') === 'true',
        ssl: configService.get('DB_SSL') === 'true' ? {
          rejectUnauthorized: false,
        } : false,
        extra: {
          max: +configService.get('DB_POOL_MAX') || 10,
          min: +configService.get('DB_POOL_MIN') || 2,
        },
      }),
      inject: [ConfigService],
    }),
  ],
})
export class AppModule {}
```

**Interview Tip**: Use `@nestjs/config` and `TypeOrmModule.forRootAsync()` for environment variables. Load `.env` with `ConfigModule.forRoot()`. Inject `ConfigService` in `useFactory`. Use `DATABASE_URL` for production (Heroku, Vercel). Validate env variables with `class-validator`. Never commit `.env` to git. Use different env files per environment.

</details>

## Entities

### 9. What is an Entity in TypeORM?

<details>
<summary>Answer</summary>

An **Entity** is a class that maps to a database table. Each instance represents a row in the table.

**Basic Entity:**
```typescript
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity('users') // Table name (optional, defaults to class name)
export class User {
  @PrimaryGeneratedColumn()
  id: number;
  
  @Column()
  firstName: string;
  
  @Column()
  lastName: string;
  
  @Column({ unique: true })
  email: string;
}
```

**Key Concepts:**

| Decorator | Purpose | Example |
|-----------|---------|---------|
| `@Entity()` | Mark class as entity | `@Entity('users')` |
| `@PrimaryGeneratedColumn()` | Auto-increment primary key | `id: number` |
| `@Column()` | Table column | `@Column()` |
| `@CreateDateColumn()` | Auto-created timestamp | `createdAt: Date` |
| `@UpdateDateColumn()` | Auto-updated timestamp | `updatedAt: Date` |

**Complete Entity Example:**
```typescript
import {
  Entity,
  Column,
  PrimaryGeneratedColumn,
  CreateDateColumn,
  UpdateDateColumn,
  DeleteDateColumn,
} from 'typeorm';

@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;
  
  @Column({ length: 100 })
  firstName: string;
  
  @Column({ length: 100 })
  lastName: string;
  
  @Column({ unique: true })
  email: string;
  
  @Column({ select: false }) // Hidden by default
  password: string;
  
  @Column({ default: true })
  isActive: boolean;
  
  @Column({ type: 'int', nullable: true })
  age: number;
  
  @Column('simple-array', { nullable: true })
  roles: string[];
  
  @CreateDateColumn()
  createdAt: Date;
  
  @UpdateDateColumn()
  updatedAt: Date;
  
  @DeleteDateColumn()
  deletedAt: Date; // For soft deletes
}
```

**Column Types:**
```typescript
@Entity()
export class Product {
  @PrimaryGeneratedColumn()
  id: number;
  
  // String types
  @Column({ type: 'varchar', length: 255 })
  name: string;
  
  @Column({ type: 'text' })
  description: string;
  
  // Number types
  @Column({ type: 'int' })
  quantity: number;
  
  @Column({ type: 'decimal', precision: 10, scale: 2 })
  price: number;
  
  @Column({ type: 'float' })
  rating: number;
  
  // Boolean
  @Column({ type: 'boolean', default: true })
  inStock: boolean;
  
  // Date
  @Column({ type: 'date' })
  manufactureDate: Date;
  
  @Column({ type: 'timestamp' })
  lastChecked: Date;
  
  // JSON
  @Column({ type: 'json' })
  metadata: Record<string, any>;
  
  // Array
  @Column('simple-array')
  tags: string[];
  
  // Enum
  @Column({
    type: 'enum',
    enum: ['active', 'inactive', 'pending'],
    default: 'active',
  })
  status: string;
}
```

**Column Options:**
```typescript
@Entity()
export class User {
  @Column({
    type: 'varchar',
    length: 100,
    unique: true,
    nullable: false,
    default: 'John',
    comment: 'User first name',
  })
  firstName: string;
  
  @Column({
    name: 'user_email', // Database column name
    select: false, // Don't include in queries by default
  })
  email: string;
  
  @Column({
    transformer: {
      to: (value: string) => value.toLowerCase(),
      from: (value: string) => value.toUpperCase(),
    },
  })
  username: string;
}
```

**Primary Keys:**
```typescript
// Auto-increment integer
@PrimaryGeneratedColumn()
id: number;

// UUID
@PrimaryGeneratedColumn('uuid')
id: string;

// Custom primary key
@PrimaryColumn()
customId: string;

// Composite primary key
@Entity()
export class UserRole {
  @PrimaryColumn()
  userId: number;
  
  @PrimaryColumn()
  roleId: number;
}
```

**Timestamps:**
```typescript
@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;
  
  @Column()
  title: string;
  
  @CreateDateColumn() // Auto-set on insert
  createdAt: Date;
  
  @UpdateDateColumn() // Auto-updated on save
  updatedAt: Date;
  
  @DeleteDateColumn() // Set on soft delete
  deletedAt: Date;
}
```

**Soft Deletes:**
```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;
  
  @Column()
  name: string;
  
  @DeleteDateColumn()
  deletedAt: Date; // Automatically set when soft-deleted
}

// Usage
await userRepository.softDelete(id); // Sets deletedAt
await userRepository.restore(id); // Clears deletedAt
await userRepository.find(); // Excludes soft-deleted
await userRepository.find({ withDeleted: true }); // Includes soft-deleted
```

**Interview Tip**: Entity is a class decorated with `@Entity()` that maps to database table. Use `@PrimaryGeneratedColumn()` for ID, `@Column()` for fields. Supports many column types: varchar, int, decimal, json, array, enum. Use `@CreateDateColumn()` and `@UpdateDateColumn()` for timestamps. Use `@DeleteDateColumn()` for soft deletes. Column options: `nullable`, `unique`, `default`, `length`.

</details>

### 10. What are the essential decorators: `@Entity()`, `@PrimaryGeneratedColumn()`, `@Column()`?

<details>
<summary>Answer</summary>

These three decorators are fundamental for creating TypeORM entities.

**1. @Entity() - Define Entity:**

Marks a class as a database table.

```typescript
import { Entity } from 'typeorm';

// Table name is 'user' (class name in lowercase)
@Entity()
export class User {
  // ...
}

// Custom table name
@Entity('users')
export class User {
  // ...
}

// With schema (PostgreSQL)
@Entity({ name: 'users', schema: 'public' })
export class User {
  // ...
}

// With database (MySQL)
@Entity({ name: 'users', database: 'mydb' })
export class User {
  // ...
}
```

**2. @PrimaryGeneratedColumn() - Primary Key:**

Creates an auto-incrementing primary key.

```typescript
import { PrimaryGeneratedColumn } from 'typeorm';

// Auto-increment integer
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;
}

// UUID
@Entity()
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;
}

// Custom increment strategy
@Entity()
export class User {
  @PrimaryGeneratedColumn('increment')
  id: number;
}

// ROW ID (PostgreSQL)
@Entity()
export class User {
  @PrimaryGeneratedColumn('rowid')
  id: number;
}
```

**3. @Column() - Table Column:**

Defines a table column.

```typescript
import { Column } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;
  
  // Simple column
  @Column()
  name: string;
  
  // With type
  @Column('varchar')
  email: string;
  
  // With length
  @Column({ length: 100 })
  firstName: string;
  
  // Nullable
  @Column({ nullable: true })
  middleName: string;
  
  // Unique
  @Column({ unique: true })
  username: string;
  
  // Default value
  @Column({ default: true })
  isActive: boolean;
  
  // Multiple options
  @Column({
    type: 'varchar',
    length: 255,
    unique: true,
    nullable: false,
    default: 'user@example.com',
    comment: 'User email address',
  })
  email: string;
}
```

**Complete Example:**
```typescript
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  UpdateDateColumn,
} from 'typeorm';

@Entity('users')
export class User {
  // Primary key
  @PrimaryGeneratedColumn('uuid')
  id: string;
  
  // Required columns
  @Column({ length: 50 })
  firstName: string;
  
  @Column({ length: 50 })
  lastName: string;
  
  @Column({ unique: true })
  email: string;
  
  // Optional column
  @Column({ nullable: true })
  phoneNumber: string;
  
  // Column with default
  @Column({ default: true })
  isActive: boolean;
  
  // Number column
  @Column({ type: 'int', default: 0 })
  loginCount: number;
  
  // Decimal column
  @Column({ type: 'decimal', precision: 10, scale: 2, default: 0 })
  balance: number;
  
  // Enum column
  @Column({
    type: 'enum',
    enum: ['user', 'admin', 'moderator'],
    default: 'user',
  })
  role: string;
  
  // Array column
  @Column('simple-array', { nullable: true })
  interests: string[];
  
  // JSON column
  @Column({ type: 'json', nullable: true })
  settings: Record<string, any>;
  
  // Timestamps
  @CreateDateColumn()
  createdAt: Date;
  
  @UpdateDateColumn()
  updatedAt: Date;
}
```

**Column Types Comparison:**

| Type | TypeScript | Database | Example |
|------|------------|----------|---------|
| varchar | string | VARCHAR | `@Column('varchar')` |
| text | string | TEXT | `@Column('text')` |
| int | number | INTEGER | `@Column('int')` |
| decimal | number | DECIMAL | `@Column('decimal')` |
| boolean | boolean | BOOLEAN | `@Column('boolean')` |
| date | Date | DATE | `@Column('date')` |
| timestamp | Date | TIMESTAMP | `@Column('timestamp')` |
| json | object | JSON | `@Column('json')` |
| simple-array | string[] | TEXT | `@Column('simple-array')` |

**Special Columns:**
```typescript
@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;
  
  @Column()
  title: string;
  
  // Auto-generated columns
  @CreateDateColumn()
  createdAt: Date; // Automatically set on creation
  
  @UpdateDateColumn()
  updatedAt: Date; // Automatically updated on save
  
  @DeleteDateColumn()
  deletedAt: Date; // Set on soft delete
  
  @VersionColumn()
  version: number; // Incremented on each update
  
  // Generated column
  @Generated('uuid')
  uuid: string;
  
  // Virtual column (not in database)
  @Column()
  firstName: string;
  
  @Column()
  lastName: string;
  
  // Computed property (not decorated)
  get fullName(): string {
    return `${this.firstName} ${this.lastName}`;
  }
}
```

**Custom Column Names:**
```typescript
@Entity('users')
export class User {
  @PrimaryGeneratedColumn()
  id: number;
  
  // TypeScript property: firstName
  // Database column: first_name
  @Column({ name: 'first_name' })
  firstName: string;
  
  @Column({ name: 'last_name' })
  lastName: string;
  
  @Column({ name: 'email_address' })
  email: string;
}
```

**Interview Tip**: `@Entity()` marks class as database table, optional table name parameter. `@PrimaryGeneratedColumn()` creates auto-increment primary key, use `'uuid'` for UUID. `@Column()` defines table column with options: `type`, `length`, `nullable`, `unique`, `default`. Also use `@CreateDateColumn()`, `@UpdateDateColumn()` for timestamps. Can specify custom column names with `name` option.

</details>

### 11. How do you define relationships: `@OneToMany()`, `@ManyToOne()`, `@ManyToMany()`, `@OneToOne()`?

<details>
<summary>Answer</summary>

TypeORM provides decorators to define relationships between entities.

**1. @OneToMany and @ManyToOne (One-to-Many):**

```typescript
// User has many posts
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;
  
  @Column()
  name: string;
  
  @OneToMany(() => Post, post => post.author)
  posts: Post[];
}

// Post belongs to one user
@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;
  
  @Column()
  title: string;
  
  @ManyToOne(() => User, user => user.posts)
  author: User;
  
  @Column()
  authorId: number; // Foreign key column
}

// Usage
const user = await userRepository.findOne({
  where: { id: 1 },
  relations: ['posts'],
});
console.log(user.posts); // Array of posts
```

**2. @ManyToMany (Many-to-Many):**

```typescript
// User can have many roles, Role can have many users
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;
  
  @Column()
  name: string;
  
  @ManyToMany(() => Role, role => role.users)
  @JoinTable() // Only on one side (owning side)
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

// Creates join table: user_roles_role
// Columns: userId, roleId

// Usage
const user = await userRepository.findOne({
  where: { id: 1 },
  relations: ['roles'],
});
console.log(user.roles); // Array of roles
```

**3. @OneToOne (One-to-One):**

```typescript
// User has one profile
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;
  
  @Column()
  name: string;
  
  @OneToOne(() => Profile, profile => profile.user)
  @JoinColumn() // Foreign key on this side
  profile: Profile;
}

// Profile belongs to one user
@Entity()
export class Profile {
  @PrimaryGeneratedColumn()
  id: number;
  
  @Column()
  bio: string;
  
  @OneToOne(() => User, user => user.profile)
  user: User;
}

// Usage
const user = await userRepository.findOne({
  where: { id: 1 },
  relations: ['profile'],
});
console.log(user.profile); // Profile object
```

**Complete Example with All Relationships:**

```typescript
// User entity
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;
  
  @Column()
  name: string;
  
  // One-to-One: User has one profile
  @OneToOne(() => Profile, profile => profile.user, {
    cascade: true,
    onDelete: 'CASCADE',
  })
  @JoinColumn()
  profile: Profile;
  
  // One-to-Many: User has many posts
  @OneToMany(() => Post, post => post.author, {
    cascade: true,
  })
  posts: Post[];
  
  // Many-to-Many: User has many roles
  @ManyToMany(() => Role, role => role.users)
  @JoinTable({
    name: 'user_roles',
    joinColumn: { name: 'user_id', referencedColumnName: 'id' },
    inverseJoinColumn: { name: 'role_id', referencedColumnName: 'id' },
  })
  roles: Role[];
}

// Profile entity
@Entity()
export class Profile {
  @PrimaryGeneratedColumn()
  id: number;
  
  @Column()
  bio: string;
  
  @OneToOne(() => User, user => user.profile)
  user: User;
}

// Post entity
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
  
  @Column()
  authorId: number;
}

// Role entity
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

**Bi-directional vs Uni-directional:**

```typescript
// Bi-directional (both sides know about each other)
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

// Uni-directional (only one side knows)
@Entity()
export class Post {
  @ManyToOne(() => User)
  @JoinColumn({ name: 'authorId' })
  author: User;
  
  @Column()
  authorId: number;
}
```

**Self-referencing Relationships:**

```typescript
// User can follow other users
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;
  
  @Column()
  name: string;
  
  @ManyToMany(() => User, user => user.followers)
  @JoinTable()
  following: User[];
  
  @ManyToMany(() => User, user => user.following)
  followers: User[];
}

// Category tree
@Entity()
export class Category {
  @PrimaryGeneratedColumn()
  id: number;
  
  @Column()
  name: string;
  
  @ManyToOne(() => Category, category => category.children)
  parent: Category;
  
  @OneToMany(() => Category, category => category.parent)
  children: Category[];
}
```

**Custom Join Table:**

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;
  
  @ManyToMany(() => Group, group => group.users)
  @JoinTable({
    name: 'user_groups',
    joinColumn: {
      name: 'user_id',
      referencedColumnName: 'id',
    },
    inverseJoinColumn: {
      name: 'group_id',
      referencedColumnName: 'id',
    },
  })
  groups: Group[];
}
```

**Interview Tip**: `@OneToMany()` and `@ManyToOne()` for one-to-many (parent-child). `@ManyToMany()` with `@JoinTable()` for many-to-many. `@OneToOne()` with `@JoinColumn()` for one-to-one. First parameter is entity type function, second is inverse side. Use `relations` option in find to load relationships. Can be bi-directional or uni-directional.

</details>

### 12. What is cascade and eager loading in relationships?

<details>
<summary>Answer</summary>

**Cascade** operations automatically perform actions on related entities. **Eager loading** automatically loads related entities.

**1. Cascade Operations:**

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;
  
  @Column()
  name: string;
  
  @OneToMany(() => Post, post => post.author, {
    cascade: true, // Enable all cascade operations
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

// With cascade: true
const user = new User();
user.name = 'John';
user.posts = [
  { title: 'Post 1' },
  { title: 'Post 2' },
];

// Saves user AND posts in one operation
await userRepository.save(user);
```

**Cascade Options:**

```typescript
@OneToMany(() => Post, post => post.author, {
  cascade: ['insert'], // Only cascade inserts
})
posts: Post[];

@OneToMany(() => Post, post => post.author, {
  cascade: ['update'], // Only cascade updates
})
posts: Post[];

@OneToMany(() => Post, post => post.author, {
  cascade: ['remove'], // Only cascade deletes
})
posts: Post[];

@OneToMany(() => Post, post => post.author, {
  cascade: ['insert', 'update'], // Multiple operations
})
posts: Post[];

@OneToMany(() => Post, post => post.author, {
  cascade: true, // All operations: insert, update, remove, soft-remove, recover
})
posts: Post[];
```

**Cascade Example:**

```typescript
// Without cascade
const user = new User();
user.name = 'John';
await userRepository.save(user);

const post1 = new Post();
post1.title = 'First Post';
post1.author = user;
await postRepository.save(post1);

const post2 = new Post();
post2.title = 'Second Post';
post2.author = user;
await postRepository.save(post2);

// With cascade: true
@Entity()
export class User {
  @OneToMany(() => Post, post => post.author, {
    cascade: true,
  })
  posts: Post[];
}

const user = new User();
user.name = 'John';
user.posts = [
  { title: 'First Post' },
  { title: 'Second Post' },
];

// Saves user AND all posts automatically
await userRepository.save(user);
```

**2. Eager Loading:**

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;
  
  @Column()
  name: string;
  
  @OneToMany(() => Post, post => post.author, {
    eager: true, // Always load posts with user
  })
  posts: Post[];
}

// No need to specify relations
const user = await userRepository.findOne({ where: { id: 1 } });
console.log(user.posts); // Posts are automatically loaded
```

**Lazy Loading (Default):**

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;
  
  @Column()
  name: string;
  
  @OneToMany(() => Post, post => post.author)
  posts: Post[]; // NOT eager
}

// Must explicitly load relations
const user = await userRepository.findOne({
  where: { id: 1 },
  relations: ['posts'], // Must specify
});
console.log(user.posts); // Now available
```

**Eager vs Lazy:**

| Loading | When Loaded | Query Count | Use Case |
|---------|-------------|-------------|----------|
| **Eager** | Always | 1 (with JOIN) | Frequently needed relations |
| **Lazy** | When requested | 1 + N queries | Rarely needed relations |

**Eager Loading Example:**

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;
  
  @Column()
  name: string;
  
  // Always load profile
  @OneToOne(() => Profile, profile => profile.user, {
    eager: true,
  })
  @JoinColumn()
  profile: Profile;
  
  // Don't automatically load posts
  @OneToMany(() => Post, post => post.author)
  posts: Post[];
}

const user = await userRepository.findOne({ where: { id: 1 } });
console.log(user.profile); // ✅ Loaded automatically
console.log(user.posts); // ❌ undefined (need to specify relations)

// To load posts
const userWithPosts = await userRepository.findOne({
  where: { id: 1 },
  relations: ['posts'],
});
console.log(userWithPosts.posts); // ✅ Now loaded
```

**onDelete Cascade:**

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;
  
  @OneToMany(() => Post, post => post.author, {
    cascade: true,
    onDelete: 'CASCADE', // Database-level cascade delete
  })
  posts: Post[];
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;
  
  @ManyToOne(() => User, user => user.posts, {
    onDelete: 'CASCADE', // When user is deleted, delete posts
  })
  author: User;
}

// Deleting user will automatically delete all their posts
await userRepository.delete(userId);
```

**onDelete Options:**

```typescript
@ManyToOne(() => User, user => user.posts, {
  onDelete: 'CASCADE', // Delete related records
})
author: User;

@ManyToOne(() => User, user => user.posts, {
  onDelete: 'SET NULL', // Set foreign key to NULL
})
author: User;

@ManyToOne(() => User, user => user.posts, {
  onDelete: 'RESTRICT', // Prevent deletion if related records exist
})
author: User;

@ManyToOne(() => User, user => user.posts, {
  onDelete: 'NO ACTION', // Database decides (usually RESTRICT)
})
author: User;
```

**Complete Example:**

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;
  
  @Column()
  name: string;
  
  // Eager + Cascade: Always loaded, saves/updates automatically
  @OneToOne(() => Profile, profile => profile.user, {
    eager: true,
    cascade: true,
    onDelete: 'CASCADE',
  })
  @JoinColumn()
  profile: Profile;
  
  // Cascade only: Not auto-loaded, but saves/updates/deletes cascade
  @OneToMany(() => Post, post => post.author, {
    cascade: ['insert', 'update'],
    onDelete: 'CASCADE',
  })
  posts: Post[];
  
  // Neither: Must load manually, must save manually
  @ManyToMany(() => Role, role => role.users)
  @JoinTable()
  roles: Role[];
}

// Usage
const user = new User();
user.name = 'John';
user.profile = { bio: 'Hello' }; // Will be saved automatically
user.posts = [{ title: 'Post 1' }]; // Will be saved automatically

await userRepository.save(user);

// Retrieve
const savedUser = await userRepository.findOne({ where: { id: user.id } });
console.log(savedUser.profile); // ✅ Loaded (eager: true)
console.log(savedUser.posts); // ❌ undefined (not eager)
console.log(savedUser.roles); // ❌ undefined (not eager)
```

**Interview Tip**: **Cascade** automatically saves/updates/deletes related entities. Options: `insert`, `update`, `remove`, or `true` for all. **Eager loading** automatically loads relations, use sparingly (can cause performance issues). Default is **lazy loading** (must specify `relations`). Use `onDelete` for database-level cascade. Eager loads with JOIN, lazy requires separate query.

</details>

## Repository Pattern

### 13. What is the Repository Pattern in TypeORM?

<details>
<summary>Answer</summary>

The **Repository Pattern** abstracts database operations, providing a clean API for data access without exposing implementation details.

**Basic Repository:**

```typescript
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './user.entity';

@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private userRepository: Repository<User>,
  ) {}
  
  async findAll(): Promise<User[]> {
    return this.userRepository.find();
  }
  
  async findOne(id: number): Promise<User> {
    return this.userRepository.findOne({ where: { id } });
  }
  
  async create(user: User): Promise<User> {
    return this.userRepository.save(user);
  }
  
  async update(id: number, user: Partial<User>): Promise<User> {
    await this.userRepository.update(id, user);
    return this.findOne(id);
  }
  
  async remove(id: number): Promise<void> {
    await this.userRepository.delete(id);
  }
}
```

**Repository Methods:**

| Method | Purpose | Example |
|--------|---------|---------|
| `find()` | Find multiple | `repository.find()` |
| `findOne()` | Find single | `repository.findOne({ where: { id: 1 } })` |
| `save()` | Insert or update | `repository.save(user)` |
| `insert()` | Insert only | `repository.insert(user)` |
| `update()` | Update by ID | `repository.update(1, { name: 'John' })` |
| `delete()` | Delete by ID | `repository.delete(1)` |
| `remove()` | Delete entities | `repository.remove(users)` |
| `count()` | Count records | `repository.count()` |
| `createQueryBuilder()` | Complex queries | `repository.createQueryBuilder('user')` |

**Complete CRUD Service:**

```typescript
import { Injectable, NotFoundException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './user.entity';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';

@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // Create
  async create(createUserDto: CreateUserDto): Promise<User> {
    const user = this.userRepository.create(createUserDto);
    return this.userRepository.save(user);
  }
  
  // Read all
  async findAll(): Promise<User[]> {
    return this.userRepository.find({
      relations: ['profile', 'posts'],
      order: { createdAt: 'DESC' },
    });
  }
  
  // Read one
  async findOne(id: number): Promise<User> {
    const user = await this.userRepository.findOne({
      where: { id },
      relations: ['profile', 'posts'],
    });
    
    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    
    return user;
  }
  
  // Update
  async update(id: number, updateUserDto: UpdateUserDto): Promise<User> {
    await this.userRepository.update(id, updateUserDto);
    return this.findOne(id);
  }
  
  // Delete
  async remove(id: number): Promise<void> {
    const result = await this.userRepository.delete(id);
    
    if (result.affected === 0) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
  }
  
  // Soft delete
  async softRemove(id: number): Promise<void> {
    const user = await this.findOne(id);
    await this.userRepository.softRemove(user);
  }
  
  // Restore soft deleted
  async restore(id: number): Promise<User> {
    await this.userRepository.restore(id);
    return this.findOne(id);
  }
  
  // Find by email
  async findByEmail(email: string): Promise<User | null> {
    return this.userRepository.findOne({ where: { email } });
  }
  
  // Check if exists
  async exists(id: number): Promise<boolean> {
    const count = await this.userRepository.count({ where: { id } });
    return count > 0;
  }
  
  // Count
  async count(): Promise<number> {
    return this.userRepository.count();
  }
}
```

**Custom Repository:**

```typescript
import { EntityRepository, Repository } from 'typeorm';
import { User } from './user.entity';

@EntityRepository(User)
export class UserRepository extends Repository<User> {
  // Custom method
  async findByEmail(email: string): Promise<User | null> {
    return this.findOne({ where: { email } });
  }
  
  // Custom method with QueryBuilder
  async findActiveUsers(): Promise<User[]> {
    return this.createQueryBuilder('user')
      .where('user.isActive = :isActive', { isActive: true })
      .orderBy('user.createdAt', 'DESC')
      .getMany();
  }
  
  // Custom method with relations
  async findWithPosts(id: number): Promise<User> {
    return this.createQueryBuilder('user')
      .leftJoinAndSelect('user.posts', 'posts')
      .where('user.id = :id', { id })
      .getOne();
  }
  
  // Bulk operations
  async createMany(users: User[]): Promise<User[]> {
    return this.save(users);
  }
  
  // Search
  async search(term: string): Promise<User[]> {
    return this.createQueryBuilder('user')
      .where('user.firstName LIKE :term OR user.lastName LIKE :term', {
        term: `%${term}%`,
      })
      .getMany();
  }
}

// Usage in service
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(UserRepository)
    private readonly userRepository: UserRepository,
  ) {}
  
  async findByEmail(email: string): Promise<User> {
    return this.userRepository.findByEmail(email);
  }
  
  async findActiveUsers(): Promise<User[]> {
    return this.userRepository.findActiveUsers();
  }
}
```

**TreeRepository (for hierarchical data):**

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

@Injectable()
export class CategoryService {
  constructor(
    @InjectRepository(Category)
    private readonly categoryRepository: TreeRepository<Category>,
  ) {}
  
  async findTrees(): Promise<Category[]> {
    return this.categoryRepository.findTrees();
  }
  
  async findDescendants(category: Category): Promise<Category[]> {
    return this.categoryRepository.findDescendants(category);
  }
  
  async findAncestors(category: Category): Promise<Category[]> {
    return this.categoryRepository.findAncestors(category);
  }
}
```

**Repository with Transactions:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
    private readonly dataSource: DataSource,
  ) {}
  
  async transferBalance(fromId: number, toId: number, amount: number): Promise<void> {
    const queryRunner = this.dataSource.createQueryRunner();
    await queryRunner.connect();
    await queryRunner.startTransaction();
    
    try {
      const userRepo = queryRunner.manager.getRepository(User);
      
      const fromUser = await userRepo.findOne({ where: { id: fromId } });
      const toUser = await userRepo.findOne({ where: { id: toId } });
      
      fromUser.balance -= amount;
      toUser.balance += amount;
      
      await userRepo.save(fromUser);
      await userRepo.save(toUser);
      
      await queryRunner.commitTransaction();
    } catch (error) {
      await queryRunner.rollbackTransaction();
      throw error;
    } finally {
      await queryRunner.release();
    }
  }
}
```

**Interview Tip**: Repository Pattern provides clean API for database operations: `find()`, `findOne()`, `save()`, `update()`, `delete()`. Inject with `@InjectRepository(Entity)`. Can create custom repositories extending `Repository<T>` with custom methods. Use `createQueryBuilder()` for complex queries. Supports transactions with QueryRunner.

</details>

### 14. What is `TypeOrmModule.forFeature()` used for?

<details>
<summary>Answer</summary>

`forFeature()` registers entities for a specific module, making their repositories available for injection.

**Basic Usage:**

```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { User } from './user.entity';
import { UserService } from './user.service';
import { UserController } from './user.controller';

@Module({
  imports: [
    TypeOrmModule.forFeature([User]), // Register User entity
  ],
  providers: [UserService],
  controllers: [UserController],
})
export class UserModule {}
```

**Multiple Entities:**

```typescript
@Module({
  imports: [
    TypeOrmModule.forFeature([
      User,
      Post,
      Comment,
      Profile,
    ]),
  ],
  providers: [UserService, PostService, CommentService],
  controllers: [UserController, PostController],
})
export class UserModule {}
```

**Named Connection:**

```typescript
// app.module.ts - Multiple database connections
@Module({
  imports: [
    TypeOrmModule.forRoot({
      name: 'default',
      type: 'postgres',
      // ... config
    }),
    TypeOrmModule.forRoot({
      name: 'secondary',
      type: 'mysql',
      // ... config
    }),
  ],
})
export class AppModule {}

// user.module.ts - Use specific connection
@Module({
  imports: [
    TypeOrmModule.forFeature([User], 'default'), // Default connection
    TypeOrmModule.forFeature([Product], 'secondary'), // Secondary connection
  ],
})
export class UserModule {}
```

**Injecting Repository:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User) // Inject User repository
    private readonly userRepository: Repository<User>,
    
    @InjectRepository(Post) // Inject Post repository
    private readonly postRepository: Repository<Post>,
  ) {}
  
  async findAll(): Promise<User[]> {
    return this.userRepository.find();
  }
  
  async createUserWithPost(userData: any, postData: any): Promise<User> {
    const user = await this.userRepository.save(userData);
    const post = this.postRepository.create({ ...postData, authorId: user.id });
    await this.postRepository.save(post);
    return user;
  }
}
```

**Exporting Repositories:**

```typescript
// user.module.ts - Export repositories for other modules
@Module({
  imports: [
    TypeOrmModule.forFeature([User, Profile]),
  ],
  providers: [UserService],
  controllers: [UserController],
  exports: [
    TypeOrmModule, // Export TypeOrmModule to share repositories
    UserService,
  ],
})
export class UserModule {}

// post.module.ts - Use exported repositories
@Module({
  imports: [
    TypeOrmModule.forFeature([Post]),
    UserModule, // Import UserModule to access User repository
  ],
  providers: [PostService],
  controllers: [PostController],
})
export class PostModule {}

// post.service.ts - Use User repository from UserModule
@Injectable()
export class PostService {
  constructor(
    @InjectRepository(Post)
    private readonly postRepository: Repository<Post>,
    
    @InjectRepository(User) // Available because UserModule exports TypeOrmModule
    private readonly userRepository: Repository<User>,
  ) {}
  
  async createPost(authorId: number, postData: any): Promise<Post> {
    const author = await this.userRepository.findOne({ where: { id: authorId } });
    const post = this.postRepository.create({ ...postData, author });
    return this.postRepository.save(post);
  }
}
```

**Custom Repository:**

```typescript
// user.repository.ts
@EntityRepository(User)
export class UserRepository extends Repository<User> {
  async findByEmail(email: string): Promise<User | null> {
    return this.findOne({ where: { email } });
  }
}

// user.module.ts
@Module({
  imports: [
    TypeOrmModule.forFeature([UserRepository]), // Register custom repository
  ],
  providers: [UserService],
  controllers: [UserController],
})
export class UserModule {}

// user.service.ts
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(UserRepository)
    private readonly userRepository: UserRepository,
  ) {}
  
  async findByEmail(email: string): Promise<User> {
    return this.userRepository.findByEmail(email);
  }
}
```

**Multiple Connections Example:**

```typescript
// app.module.ts
@Module({
  imports: [
    // PostgreSQL connection
    TypeOrmModule.forRoot({
      name: 'postgres',
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      database: 'main_db',
      entities: [User, Post],
      synchronize: true,
    }),
    
    // MySQL connection
    TypeOrmModule.forRoot({
      name: 'mysql',
      type: 'mysql',
      host: 'localhost',
      port: 3306,
      database: 'analytics_db',
      entities: [Analytics, Report],
      synchronize: true,
    }),
  ],
})
export class AppModule {}

// user.module.ts
@Module({
  imports: [
    TypeOrmModule.forFeature([User, Post], 'postgres'), // Postgres entities
    TypeOrmModule.forFeature([Analytics], 'mysql'), // MySQL entities
  ],
  providers: [UserService],
})
export class UserModule {}

// user.service.ts
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User, 'postgres')
    private readonly userRepository: Repository<User>,
    
    @InjectRepository(Analytics, 'mysql')
    private readonly analyticsRepository: Repository<Analytics>,
  ) {}
  
  async createUser(userData: any): Promise<User> {
    const user = await this.userRepository.save(userData);
    
    // Log to analytics database
    await this.analyticsRepository.save({
      event: 'user_created',
      userId: user.id,
    });
    
    return user;
  }
}
```

**Testing with forFeature():**

```typescript
// user.service.spec.ts
describe('UserService', () => {
  let service: UserService;
  let repository: Repository<User>;
  
  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      imports: [
        TypeOrmModule.forFeature([User]), // Include in test module
      ],
      providers: [
        UserService,
        {
          provide: getRepositoryToken(User),
          useValue: {
            find: jest.fn(),
            findOne: jest.fn(),
            save: jest.fn(),
            delete: jest.fn(),
          },
        },
      ],
    }).compile();
    
    service = module.get<UserService>(UserService);
    repository = module.get<Repository<User>>(getRepositoryToken(User));
  });
  
  it('should find all users', async () => {
    const users = [{ id: 1, name: 'John' }];
    jest.spyOn(repository, 'find').mockResolvedValue(users as any);
    
    expect(await service.findAll()).toEqual(users);
  });
});
```

**Interview Tip**: `forFeature()` registers entities for a module, making repositories injectable. Syntax: `TypeOrmModule.forFeature([Entity])`. For multiple connections: `forFeature([Entity], 'connectionName')`. Export `TypeOrmModule` to share repositories between modules. Inject with `@InjectRepository(Entity)` or `@InjectRepository(Entity, 'connectionName')`.

</details>

### 15. How do you inject a repository using `@InjectRepository()`?

<details>
<summary>Answer</summary>

`@InjectRepository()` injects a TypeORM repository into a provider, enabling database operations.

**Basic Injection:**

```typescript
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './user.entity';

@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  async findAll(): Promise<User[]> {
    return this.userRepository.find();
  }
}
```

**Multiple Repositories:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
    
    @InjectRepository(Post)
    private readonly postRepository: Repository<Post>,
    
    @InjectRepository(Comment)
    private readonly commentRepository: Repository<Comment>,
  ) {}
  
  async getUserWithContent(userId: number) {
    const user = await this.userRepository.findOne({ where: { id: userId } });
    const posts = await this.postRepository.find({ where: { authorId: userId } });
    const comments = await this.commentRepository.find({ where: { authorId: userId } });
    
    return { user, posts, comments };
  }
}
```

**Named Connection:**

```typescript
// Inject from specific database connection
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User, 'default')
    private readonly userRepository: Repository<User>,
    
    @InjectRepository(Product, 'secondary')
    private readonly productRepository: Repository<Product>,
  ) {}
  
  async createUserAndProduct(userData: any, productData: any) {
    // Save to default database
    const user = await this.userRepository.save(userData);
    
    // Save to secondary database
    const product = await this.productRepository.save(productData);
    
    return { user, product };
  }
}
```

**Repository Types:**

```typescript
import { Repository, TreeRepository, MongoRepository } from 'typeorm';

@Injectable()
export class DataService {
  constructor(
    // Standard repository
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
    
    // Tree repository for hierarchical data
    @InjectRepository(Category)
    private readonly categoryRepository: TreeRepository<Category>,
    
    // MongoDB repository
    @InjectRepository(Document)
    private readonly documentRepository: MongoRepository<Document>,
  ) {}
  
  async getHierarchy() {
    return this.categoryRepository.findTrees();
  }
}
```

**Custom Repository:**

```typescript
// user.repository.ts
import { EntityRepository, Repository } from 'typeorm';

@EntityRepository(User)
export class UserRepository extends Repository<User> {
  async findByEmail(email: string): Promise<User | null> {
    return this.findOne({ where: { email } });
  }
  
  async findActiveUsers(): Promise<User[]> {
    return this.find({ where: { isActive: true } });
  }
}

// user.module.ts
@Module({
  imports: [TypeOrmModule.forFeature([UserRepository])],
  providers: [UserService],
})
export class UserModule {}

// user.service.ts
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(UserRepository)
    private readonly userRepository: UserRepository,
  ) {}
  
  // Use custom methods
  async findByEmail(email: string): Promise<User> {
    return this.userRepository.findByEmail(email);
  }
  
  async getActiveUsers(): Promise<User[]> {
    return this.userRepository.findActiveUsers();
  }
}
```

**Complete Service Example:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
    
    @InjectRepository(Profile)
    private readonly profileRepository: Repository<Profile>,
  ) {}
  
  // CRUD operations
  async create(createUserDto: CreateUserDto): Promise<User> {
    const user = this.userRepository.create(createUserDto);
    return this.userRepository.save(user);
  }
  
  async findAll(): Promise<User[]> {
    return this.userRepository.find({
      relations: ['profile'],
    });
  }
  
  async findOne(id: number): Promise<User> {
    return this.userRepository.findOne({
      where: { id },
      relations: ['profile', 'posts'],
    });
  }
  
  async update(id: number, updateUserDto: UpdateUserDto): Promise<User> {
    await this.userRepository.update(id, updateUserDto);
    return this.findOne(id);
  }
  
  async remove(id: number): Promise<void> {
    await this.userRepository.delete(id);
  }
  
  // Custom operations
  async findByEmail(email: string): Promise<User | null> {
    return this.userRepository.findOne({ where: { email } });
  }
  
  async createWithProfile(userData: any, profileData: any): Promise<User> {
    const profile = this.profileRepository.create(profileData);
    await this.profileRepository.save(profile);
    
    const user = this.userRepository.create({
      ...userData,
      profile,
    });
    
    return this.userRepository.save(user);
  }
  
  async count(): Promise<number> {
    return this.userRepository.count();
  }
  
  async searchUsers(term: string): Promise<User[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .where('user.firstName LIKE :term OR user.lastName LIKE :term', {
        term: `%${term}%`,
      })
      .getMany();
  }
}
```

**Testing with Mock Repository:**

```typescript
describe('UserService', () => {
  let service: UserService;
  let repository: Repository<User>;
  
  const mockRepository = {
    find: jest.fn(),
    findOne: jest.fn(),
    create: jest.fn(),
    save: jest.fn(),
    update: jest.fn(),
    delete: jest.fn(),
  };
  
  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UserService,
        {
          provide: getRepositoryToken(User),
          useValue: mockRepository,
        },
      ],
    }).compile();
    
    service = module.get<UserService>(UserService);
    repository = module.get<Repository<User>>(getRepositoryToken(User));
  });
  
  it('should find all users', async () => {
    const users = [{ id: 1, name: 'John' }];
    mockRepository.find.mockResolvedValue(users);
    
    const result = await service.findAll();
    
    expect(result).toEqual(users);
    expect(mockRepository.find).toHaveBeenCalled();
  });
  
  it('should create a user', async () => {
    const createUserDto = { name: 'John', email: 'john@example.com' };
    const savedUser = { id: 1, ...createUserDto };
    
    mockRepository.create.mockReturnValue(createUserDto);
    mockRepository.save.mockResolvedValue(savedUser);
    
    const result = await service.create(createUserDto);
    
    expect(result).toEqual(savedUser);
    expect(mockRepository.create).toHaveBeenCalledWith(createUserDto);
    expect(mockRepository.save).toHaveBeenCalledWith(createUserDto);
  });
});
```

**Using with DataSource:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
    
    @InjectDataSource()
    private readonly dataSource: DataSource,
  ) {}
  
  async complexOperation() {
    // Use repository for simple operations
    const users = await this.userRepository.find();
    
    // Use dataSource for advanced operations
    const queryRunner = this.dataSource.createQueryRunner();
    await queryRunner.connect();
    await queryRunner.startTransaction();
    
    try {
      await queryRunner.manager.save(User, { name: 'John' });
      await queryRunner.commitTransaction();
    } catch (error) {
      await queryRunner.rollbackTransaction();
      throw error;
    } finally {
      await queryRunner.release();
    }
  }
}
```

**Interview Tip**: `@InjectRepository(Entity)` injects TypeORM repository for database operations. Must register entity in module with `TypeOrmModule.forFeature([Entity])`. For named connections: `@InjectRepository(Entity, 'connectionName')`. Returns `Repository<Entity>` type with methods: `find()`, `findOne()`, `save()`, `update()`, `delete()`. Test with `getRepositoryToken(Entity)` to provide mock.

</details>

## CRUD Operations

### 16. How do you create records using `save()` vs `insert()`?

<details>
<summary>Answer</summary>

Both `save()` and `insert()` create records, but they have different behaviors.

**save() Method:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // save() - Insert or Update
  async createUser(createUserDto: CreateUserDto): Promise<User> {
    const user = this.userRepository.create(createUserDto);
    return this.userRepository.save(user);
  }
  
  // save() with existing ID - Updates
  async updateOrCreate(id: number, userData: any): Promise<User> {
    const user = await this.userRepository.findOne({ where: { id } });
    if (user) {
      Object.assign(user, userData);
    } else {
      const newUser = this.userRepository.create({ id, ...userData });
      return this.userRepository.save(newUser);
    }
    return this.userRepository.save(user);
  }
  
  // save() multiple records
  async createMany(users: CreateUserDto[]): Promise<User[]> {
    const entities = this.userRepository.create(users);
    return this.userRepository.save(entities);
  }
}
```

**insert() Method:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // insert() - Only Insert (fails if exists)
  async insertUser(createUserDto: CreateUserDto): Promise<void> {
    await this.userRepository.insert(createUserDto);
  }
  
  // insert() multiple records (more efficient)
  async insertMany(users: CreateUserDto[]): Promise<void> {
    await this.userRepository.insert(users);
  }
  
  // insert() returns InsertResult
  async insertWithResult(createUserDto: CreateUserDto) {
    const result = await this.userRepository.insert(createUserDto);
    return {
      identifiers: result.identifiers, // Array of inserted IDs
      generatedMaps: result.generatedMaps, // Generated values
      raw: result.raw, // Raw database response
    };
  }
}
```

**Comparison Table:**

| Feature | `save()` | `insert()` |
|---------|----------|------------|
| **Operation** | Insert or Update | Insert only |
| **Behavior** | Upsert (checks if exists) | Always insert |
| **Performance** | Slower (checks existence) | Faster (direct insert) |
| **Returns** | Entity instance(s) | InsertResult |
| **Hooks** | Triggers @BeforeInsert/@AfterInsert | Triggers @BeforeInsert/@AfterInsert |
| **Validation** | Full entity validation | Partial validation |
| **Relations** | Handles cascades | Doesn't handle cascades |
| **Use Case** | General purpose | Bulk inserts |

**Complete Examples:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // 1. save() - Basic create
  async createWithSave(createUserDto: CreateUserDto): Promise<User> {
    const user = this.userRepository.create(createUserDto);
    return this.userRepository.save(user);
  }
  
  // 2. save() - With relations (cascade)
  async createUserWithProfile(userData: any, profileData: any): Promise<User> {
    const user = this.userRepository.create({
      ...userData,
      profile: profileData, // Cascade save
    });
    return this.userRepository.save(user);
  }
  
  // 3. insert() - Fast insert without return
  async fastInsert(createUserDto: CreateUserDto): Promise<void> {
    await this.userRepository.insert(createUserDto);
  }
  
  // 4. insert() - Bulk insert (efficient)
  async bulkInsert(users: CreateUserDto[]): Promise<number> {
    const result = await this.userRepository.insert(users);
    return result.identifiers.length;
  }
  
  // 5. save() - Update existing
  async updateUser(id: number, updateUserDto: UpdateUserDto): Promise<User> {
    const user = await this.userRepository.findOne({ where: { id } });
    Object.assign(user, updateUserDto);
    return this.userRepository.save(user); // Updates because ID exists
  }
  
  // 6. insert() with transaction
  async insertWithTransaction(users: CreateUserDto[]): Promise<void> {
    await this.userRepository.manager.transaction(async (manager) => {
      await manager.insert(User, users);
    });
  }
}
```

**Entity Hooks Example:**

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
  
  // Called before insert with both save() and insert()
  @BeforeInsert()
  async hashPassword() {
    if (this.password) {
      this.password = await bcrypt.hash(this.password, 10);
    }
  }
  
  // Called after insert with both save() and insert()
  @AfterInsert()
  logInsert() {
    console.log('User inserted:', this.id);
  }
}
```

**Performance Comparison:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // Slow: save() checks if each record exists
  async importUsers_Slow(users: CreateUserDto[]): Promise<User[]> {
    const savedUsers = [];
    for (const userData of users) {
      const user = await this.userRepository.save(userData);
      savedUsers.push(user);
    }
    return savedUsers; // 1000 queries for 1000 records
  }
  
  // Fast: insert() bulk insert
  async importUsers_Fast(users: CreateUserDto[]): Promise<void> {
    await this.userRepository.insert(users); // 1 query for 1000 records
  }
  
  // Balanced: save() bulk (still checks existence)
  async importUsers_Balanced(users: CreateUserDto[]): Promise<User[]> {
    return this.userRepository.save(users); // Fewer queries, returns entities
  }
}
```

**Error Handling:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // save() - Handles duplicates gracefully
  async createOrUpdateUser(email: string, data: any): Promise<User> {
    let user = await this.userRepository.findOne({ where: { email } });
    if (user) {
      Object.assign(user, data);
    } else {
      user = this.userRepository.create({ email, ...data });
    }
    return this.userRepository.save(user);
  }
  
  // insert() - Throws error on duplicate
  async insertUser(createUserDto: CreateUserDto): Promise<void> {
    try {
      await this.userRepository.insert(createUserDto);
    } catch (error) {
      if (error.code === '23505') { // PostgreSQL duplicate key
        throw new ConflictException('User already exists');
      }
      throw error;
    }
  }
}
```

**Interview Tip**: `save()` performs insert or update (upsert), checks if entity exists by ID, triggers hooks, handles relations. Returns entity instance. `insert()` only inserts, faster for bulk operations, returns `InsertResult` with IDs. Use `save()` for general CRUD with relations, `insert()` for bulk imports. Both trigger `@BeforeInsert` and `@AfterInsert` hooks.

</details>

### 17. How do you read records using `find()`, `findOne()`, `findOneBy()`?

<details>
<summary>Answer</summary>

TypeORM provides multiple methods to query records with different options.

**Basic Find Methods:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // find() - Get all records
  async findAll(): Promise<User[]> {
    return this.userRepository.find();
  }
  
  // findOne() - Get single record with options
  async findOne(id: number): Promise<User | null> {
    return this.userRepository.findOne({ where: { id } });
  }
  
  // findOneBy() - Simplified findOne (TypeORM 0.3+)
  async findOneById(id: number): Promise<User | null> {
    return this.userRepository.findOneBy({ id });
  }
}
```

**find() with Options:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // 1. Basic find with where
  async findActiveUsers(): Promise<User[]> {
    return this.userRepository.find({
      where: { isActive: true },
    });
  }
  
  // 2. Find with multiple conditions (AND)
  async findByEmailAndActive(email: string): Promise<User | null> {
    return this.userRepository.findOne({
      where: {
        email,
        isActive: true,
      },
    });
  }
  
  // 3. Find with OR conditions
  async findByEmailOrUsername(email: string, username: string): Promise<User[]> {
    return this.userRepository.find({
      where: [
        { email },
        { username },
      ],
    });
  }
  
  // 4. Find with relations
  async findWithRelations(id: number): Promise<User | null> {
    return this.userRepository.findOne({
      where: { id },
      relations: ['profile', 'posts', 'posts.comments'],
    });
  }
  
  // 5. Find with select (specific columns)
  async findWithSelect(): Promise<User[]> {
    return this.userRepository.find({
      select: ['id', 'email', 'firstName', 'lastName'],
    });
  }
  
  // 6. Find with order
  async findOrderedUsers(): Promise<User[]> {
    return this.userRepository.find({
      order: {
        createdAt: 'DESC',
        firstName: 'ASC',
      },
    });
  }
  
  // 7. Find with pagination
  async findPaginated(page: number, limit: number): Promise<User[]> {
    return this.userRepository.find({
      skip: (page - 1) * limit,
      take: limit,
      order: { createdAt: 'DESC' },
    });
  }
  
  // 8. Complex find
  async findComplex(): Promise<User[]> {
    return this.userRepository.find({
      where: { isActive: true },
      relations: ['profile', 'posts'],
      select: ['id', 'email', 'firstName'],
      order: { createdAt: 'DESC' },
      skip: 0,
      take: 10,
    });
  }
}
```

**findOne() Examples:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // findOne by ID
  async findById(id: number): Promise<User | null> {
    return this.userRepository.findOne({ where: { id } });
  }
  
  // findOne by email
  async findByEmail(email: string): Promise<User | null> {
    return this.userRepository.findOne({ where: { email } });
  }
  
  // findOne with relations
  async findWithPosts(id: number): Promise<User | null> {
    return this.userRepository.findOne({
      where: { id },
      relations: ['posts'],
    });
  }
  
  // findOne or fail
  async findOneOrFail(id: number): Promise<User> {
    const user = await this.userRepository.findOne({ where: { id } });
    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    return user;
  }
}
```

**findOneBy() Examples (TypeORM 0.3+):**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // Simple syntax
  async findByEmail(email: string): Promise<User | null> {
    return this.userRepository.findOneBy({ email });
  }
  
  // Multiple conditions
  async findByEmailAndActive(email: string): Promise<User | null> {
    return this.userRepository.findOneBy({
      email,
      isActive: true,
    });
  }
}
```

**Advanced Find Options:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // 1. Find with cache
  async findWithCache(): Promise<User[]> {
    return this.userRepository.find({
      where: { isActive: true },
      cache: {
        id: 'active_users',
        milliseconds: 60000, // 1 minute
      },
    });
  }
  
  // 2. Find with lock (pessimistic)
  async findWithLock(id: number): Promise<User | null> {
    return this.userRepository.findOne({
      where: { id },
      lock: { mode: 'pessimistic_write' },
    });
  }
  
  // 3. Find with transaction
  async findInTransaction(manager: EntityManager, id: number): Promise<User | null> {
    return manager.findOne(User, {
      where: { id },
    });
  }
  
  // 4. Find with soft deleted records
  async findWithDeleted(): Promise<User[]> {
    return this.userRepository.find({
      withDeleted: true,
    });
  }
  
  // 5. Find only deleted records
  async findOnlyDeleted(): Promise<User[]> {
    return this.userRepository.find({
      withDeleted: true,
      where: {
        deletedAt: Not(IsNull()),
      },
    });
  }
}
```

**Comparison Operators:**

```typescript
import { Like, Between, In, MoreThan, LessThan, Not, IsNull } from 'typeorm';

@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // LIKE operator
  async searchByName(name: string): Promise<User[]> {
    return this.userRepository.find({
      where: { firstName: Like(`%${name}%`) },
    });
  }
  
  // IN operator
  async findByIds(ids: number[]): Promise<User[]> {
    return this.userRepository.find({
      where: { id: In(ids) },
    });
  }
  
  // BETWEEN operator
  async findByAgeRange(min: number, max: number): Promise<User[]> {
    return this.userRepository.find({
      where: { age: Between(min, max) },
    });
  }
  
  // Greater than / Less than
  async findUsersOlderThan(age: number): Promise<User[]> {
    return this.userRepository.find({
      where: { age: MoreThan(age) },
    });
  }
  
  async findUsersYoungerThan(age: number): Promise<User[]> {
    return this.userRepository.find({
      where: { age: LessThan(age) },
    });
  }
  
  // NOT operator
  async findInactiveUsers(): Promise<User[]> {
    return this.userRepository.find({
      where: { isActive: Not(true) },
    });
  }
  
  // IS NULL
  async findUsersWithoutProfile(): Promise<User[]> {
    return this.userRepository.find({
      where: { profileId: IsNull() },
    });
  }
}
```

**Complete Service Example:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // Get all users
  async findAll(): Promise<User[]> {
    return this.userRepository.find({
      relations: ['profile'],
      order: { createdAt: 'DESC' },
    });
  }
  
  // Get user by ID
  async findOne(id: number): Promise<User> {
    const user = await this.userRepository.findOne({
      where: { id },
      relations: ['profile', 'posts'],
    });
    
    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    
    return user;
  }
  
  // Get user by email
  async findByEmail(email: string): Promise<User | null> {
    return this.userRepository.findOneBy({ email });
  }
  
  // Search users
  async search(term: string): Promise<User[]> {
    return this.userRepository.find({
      where: [
        { firstName: Like(`%${term}%`) },
        { lastName: Like(`%${term}%`) },
        { email: Like(`%${term}%`) },
      ],
    });
  }
  
  // Get paginated users
  async findPaginated(page: number, limit: number) {
    const [users, total] = await this.userRepository.findAndCount({
      skip: (page - 1) * limit,
      take: limit,
      order: { createdAt: 'DESC' },
    });
    
    return {
      data: users,
      total,
      page,
      lastPage: Math.ceil(total / limit),
    };
  }
  
  // Get active users
  async findActive(): Promise<User[]> {
    return this.userRepository.findBy({ isActive: true });
  }
  
  // Check if user exists
  async exists(id: number): Promise<boolean> {
    const user = await this.userRepository.findOneBy({ id });
    return !!user;
  }
}
```

**Interview Tip**: `find()` returns array of entities, `findOne()` returns single entity or null. `findOneBy()` is simplified `findOne()` (TypeORM 0.3+). Options: `where` (conditions), `relations` (join), `select` (columns), `order` (sorting), `skip`/`take` (pagination). Use operators: `Like`, `In`, `Between`, `MoreThan`, `LessThan`, `Not`, `IsNull`. Always handle null returns from `findOne()`.

</details>

### 18. How do you update records using `update()`?

<details>
<summary>Answer</summary>

TypeORM provides multiple methods to update records: `update()`, `save()`, and QueryBuilder.

**update() Method:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // Basic update by ID
  async updateUser(id: number, updateUserDto: UpdateUserDto): Promise<void> {
    await this.userRepository.update(id, updateUserDto);
  }
  
  // Update with criteria
  async updateByEmail(email: string, data: Partial<User>): Promise<void> {
    await this.userRepository.update({ email }, data);
  }
  
  // Update multiple records
  async deactivateUsers(ids: number[]): Promise<void> {
    await this.userRepository.update(
      { id: In(ids) },
      { isActive: false },
    );
  }
  
  // Update and return result
  async updateWithResult(id: number, data: Partial<User>) {
    const result = await this.userRepository.update(id, data);
    return {
      affected: result.affected, // Number of rows updated
      raw: result.raw, // Raw database response
    };
  }
}
```

**save() Method (Update):**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // Update using save (loads entity first)
  async updateWithSave(id: number, updateUserDto: UpdateUserDto): Promise<User> {
    const user = await this.userRepository.findOne({ where: { id } });
    
    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    
    Object.assign(user, updateUserDto);
    return this.userRepository.save(user);
  }
  
  // Partial update with save
  async partialUpdate(id: number, data: Partial<User>): Promise<User> {
    const user = await this.userRepository.findOneBy({ id });
    
    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    
    const updated = this.userRepository.merge(user, data);
    return this.userRepository.save(updated);
  }
}
```

**Comparison: update() vs save():**

| Feature | `update()` | `save()` |
|---------|------------|----------|
| **Performance** | Fast (direct SQL) | Slower (loads entity) |
| **Queries** | 1 UPDATE query | 1 SELECT + 1 UPDATE |
| **Returns** | UpdateResult | Updated entity |
| **Hooks** | No hooks | Triggers @BeforeUpdate/@AfterUpdate |
| **Validation** | No validation | Full validation |
| **Relations** | No cascade | Handles cascades |
| **Use Case** | Simple updates | Complex updates with validation |

**Complete Update Examples:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // 1. Simple update (fast, no hooks)
  async updateEmail(id: number, email: string): Promise<void> {
    await this.userRepository.update(id, { email });
  }
  
  // 2. Update with validation (slow, with hooks)
  async updateWithValidation(id: number, updateUserDto: UpdateUserDto): Promise<User> {
    const user = await this.userRepository.findOneBy({ id });
    
    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    
    Object.assign(user, updateUserDto);
    return this.userRepository.save(user);
  }
  
  // 3. Bulk update
  async bulkUpdate(criteria: any, data: Partial<User>): Promise<number> {
    const result = await this.userRepository.update(criteria, data);
    return result.affected || 0;
  }
  
  // 4. Conditional update
  async incrementLoginCount(id: number): Promise<void> {
    await this.userRepository
      .createQueryBuilder()
      .update(User)
      .set({ loginCount: () => 'loginCount + 1' })
      .where('id = :id', { id })
      .execute();
  }
  
  // 5. Update with relations
  async updateWithProfile(id: number, userData: any, profileData: any): Promise<User> {
    const user = await this.userRepository.findOne({
      where: { id },
      relations: ['profile'],
    });
    
    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    
    Object.assign(user, userData);
    if (user.profile) {
      Object.assign(user.profile, profileData);
    }
    
    return this.userRepository.save(user);
  }
  
  // 6. Upsert (update or insert)
  async upsert(email: string, userData: Partial<User>): Promise<User> {
    let user = await this.userRepository.findOneBy({ email });
    
    if (user) {
      Object.assign(user, userData);
    } else {
      user = this.userRepository.create({ email, ...userData });
    }
    
    return this.userRepository.save(user);
  }
}
```

**QueryBuilder Updates:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // Update with QueryBuilder
  async updateWithQueryBuilder(id: number, data: Partial<User>): Promise<void> {
    await this.userRepository
      .createQueryBuilder()
      .update(User)
      .set(data)
      .where('id = :id', { id })
      .execute();
  }
  
  // Conditional update
  async updateActiveUsers(data: Partial<User>): Promise<number> {
    const result = await this.userRepository
      .createQueryBuilder()
      .update(User)
      .set(data)
      .where('isActive = :isActive', { isActive: true })
      .execute();
    
    return result.affected || 0;
  }
  
  // SQL expressions
  async incrementBalance(id: number, amount: number): Promise<void> {
    await this.userRepository
      .createQueryBuilder()
      .update(User)
      .set({ balance: () => `balance + ${amount}` })
      .where('id = :id', { id })
      .execute();
  }
  
  // Update with subquery
  async updateFromSubquery(): Promise<void> {
    await this.userRepository
      .createQueryBuilder()
      .update(User)
      .set({
        postCount: () => `(SELECT COUNT(*) FROM post WHERE post.authorId = user.id)`,
      })
      .execute();
  }
}
```

**Entity Hooks for Updates:**

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
  
  // Called before update (only with save())
  @BeforeUpdate()
  async hashPasswordOnUpdate() {
    if (this.password) {
      this.password = await bcrypt.hash(this.password, 10);
    }
  }
  
  // Called after update (only with save())
  @AfterUpdate()
  logUpdate() {
    console.log('User updated:', this.id);
  }
}
```

**Transaction Updates:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
    private readonly dataSource: DataSource,
  ) {}
  
  async updateInTransaction(id: number, data: Partial<User>): Promise<User> {
    const queryRunner = this.dataSource.createQueryRunner();
    await queryRunner.connect();
    await queryRunner.startTransaction();
    
    try {
      const user = await queryRunner.manager.findOneBy(User, { id });
      
      if (!user) {
        throw new NotFoundException(`User with ID ${id} not found`);
      }
      
      Object.assign(user, data);
      const updated = await queryRunner.manager.save(user);
      
      await queryRunner.commitTransaction();
      return updated;
    } catch (error) {
      await queryRunner.rollbackTransaction();
      throw error;
    } finally {
      await queryRunner.release();
    }
  }
}
```

**Best Practices:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // ✅ Good: Fast update for simple fields
  async updateEmail(id: number, email: string): Promise<void> {
    await this.userRepository.update(id, { email });
  }
  
  // ✅ Good: Use save() when you need hooks/validation
  async updatePassword(id: number, password: string): Promise<User> {
    const user = await this.userRepository.findOneBy({ id });
    user.password = password; // Will trigger @BeforeUpdate hook
    return this.userRepository.save(user);
  }
  
  // ✅ Good: Return updated entity
  async updateAndReturn(id: number, data: Partial<User>): Promise<User> {
    await this.userRepository.update(id, data);
    return this.userRepository.findOneBy({ id });
  }
  
  // ✅ Good: Check if update was successful
  async updateWithCheck(id: number, data: Partial<User>): Promise<void> {
    const result = await this.userRepository.update(id, data);
    
    if (result.affected === 0) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
  }
  
  // ❌ Bad: Loading entity when not needed
  async updateEmailBad(id: number, email: string): Promise<void> {
    const user = await this.userRepository.findOneBy({ id }); // Unnecessary query
    user.email = email;
    await this.userRepository.save(user);
  }
  
  // ✅ Better: Direct update
  async updateEmailGood(id: number, email: string): Promise<void> {
    await this.userRepository.update(id, { email });
  }
}
```

**Interview Tip**: `update()` performs direct SQL UPDATE, fast but no hooks. `save()` loads entity first, triggers hooks, handles validation and relations. Use `update()` for simple field updates, `save()` for complex updates with validation. Can use QueryBuilder for conditional updates. `@UpdateDateColumn()` auto-updates timestamp. Check `result.affected` to verify update success.

</details>

### 19. How do you delete records using `delete()` vs `remove()`?

<details>
<summary>Answer</summary>

Both `delete()` and `remove()` delete records, but they have different behaviors.

**delete() Method:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // Delete by ID
  async deleteUser(id: number): Promise<void> {
    await this.userRepository.delete(id);
  }
  
  // Delete by criteria
  async deleteByEmail(email: string): Promise<void> {
    await this.userRepository.delete({ email });
  }
  
  // Delete multiple records
  async deleteMany(ids: number[]): Promise<void> {
    await this.userRepository.delete(ids);
  }
  
  // Delete with result
  async deleteWithResult(id: number) {
    const result = await this.userRepository.delete(id);
    return {
      affected: result.affected, // Number of rows deleted
      raw: result.raw, // Raw database response
    };
  }
}
```

**remove() Method:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // Remove single entity (must load first)
  async removeUser(id: number): Promise<void> {
    const user = await this.userRepository.findOne({ where: { id } });
    
    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    
    await this.userRepository.remove(user);
  }
  
  // Remove multiple entities
  async removeMany(ids: number[]): Promise<void> {
    const users = await this.userRepository.findBy({ id: In(ids) });
    await this.userRepository.remove(users);
  }
  
  // Remove returns the removed entity
  async removeAndReturn(id: number): Promise<User> {
    const user = await this.userRepository.findOneBy({ id });
    
    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    
    return this.userRepository.remove(user);
  }
}
```

**Comparison Table:**

| Feature | `delete()` | `remove()` |
|---------|------------|------------|
| **Operation** | Direct SQL DELETE | Loads then deletes |
| **Performance** | Fast (1 query) | Slower (SELECT + DELETE) |
| **Input** | ID or criteria | Entity instance(s) |
| **Returns** | DeleteResult | Removed entity |
| **Hooks** | No hooks | Triggers @BeforeRemove/@AfterRemove |
| **Cascades** | Database-level only | Application-level cascades |
| **Use Case** | Simple deletes | Complex deletes with hooks |

**Soft Delete:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // Soft delete by ID
  async softDelete(id: number): Promise<void> {
    await this.userRepository.softDelete(id);
  }
  
  // Soft delete by criteria
  async softDeleteInactive(): Promise<void> {
    await this.userRepository.softDelete({ isActive: false });
  }
  
  // Soft remove (with entity)
  async softRemove(id: number): Promise<User> {
    const user = await this.userRepository.findOneBy({ id });
    
    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    
    return this.userRepository.softRemove(user);
  }
  
  // Restore soft-deleted
  async restore(id: number): Promise<void> {
    await this.userRepository.restore(id);
  }
  
  // Find soft-deleted records
  async findDeleted(): Promise<User[]> {
    return this.userRepository.find({ withDeleted: true });
  }
  
  // Recover soft-deleted entity
  async recover(id: number): Promise<User> {
    const user = await this.userRepository.findOne({
      where: { id },
      withDeleted: true,
    });
    
    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    
    return this.userRepository.recover(user);
  }
}
```

**Interview Tip**: `delete()` performs direct SQL DELETE, fast but no hooks. `remove()` loads entity first, triggers hooks. Use `softDelete()` to mark as deleted without removing (sets `deletedAt`). Can restore with `restore()`. Use `delete()` for simple deletes, `remove()` when hooks needed. Check `result.affected` to verify deletion. Soft delete is recommended for user data.

</details>

## Querying

### 20. How do you use `find()` with options (where, select, relations, order)?

<details>
<summary>Answer</summary>

The `find()` method accepts various options to filter, select, join, and sort data. This is covered in detail in Q17, but here's additional focus on combining all options.

**Complete Example:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  async findAdvanced(filters: any): Promise<User[]> {
    return this.userRepository.find({
      // WHERE: Filter conditions
      where: {
        isActive: true,
        role: In(['admin', 'moderator']),
        age: Between(18, 65),
      },
      
      // SELECT: Specific columns
      select: ['id', 'email', 'firstName', 'lastName', 'createdAt'],
      
      // RELATIONS: Join tables
      relations: ['profile', 'posts', 'roles'],
      
      // ORDER: Sort results
      order: {
        createdAt: 'DESC',
        lastName: 'ASC',
      },
      
      // PAGINATION
      skip: 0,
      take: 10,
    });
  }
}
```

**Interview Tip**: Combine `where`, `select`, `relations`, `order`, `skip`, `take` in `find()`. See Q17 for comprehensive examples of each option.

</details>

### 21. How do you implement pagination using `skip` and `take`?

<details>
<summary>Answer</summary>

Pagination uses `skip` (offset) and `take` (limit) with `findAndCount()` for total count.

**Complete Pagination:**

```typescript
export interface PaginationResult<T> {
  data: T[];
  meta: {
    total: number;
    page: number;
    limit: number;
    totalPages: number;
    hasNextPage: boolean;
    hasPreviousPage: boolean;
  };
}

@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  async findPaginated(page: number, limit: number): Promise<PaginationResult<User>> {
    const [data, total] = await this.userRepository.findAndCount({
      skip: (page - 1) * limit,
      take: limit,
      order: { createdAt: 'DESC' },
      relations: ['profile'],
    });
    
    const totalPages = Math.ceil(total / limit);
    
    return {
      data,
      meta: {
        total,
        page,
        limit,
        totalPages,
        hasNextPage: page < totalPages,
        hasPreviousPage: page > 1,
      },
    };
  }
}
```

**Interview Tip**: Use `skip = (page - 1) * limit` and `take = limit`. Use `findAndCount()` to get data and total in one query. Return metadata for UI pagination controls.

</details>

### 22. What is `findAndCount()` used for?

<details>
<summary>Answer</summary>

`findAndCount()` returns both entities and total count in a single operation, perfect for pagination.

**Basic Usage:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  async findWithCount(): Promise<[User[], number]> {
    const [users, total] = await this.userRepository.findAndCount({
      where: { isActive: true },
      order: { createdAt: 'DESC' },
      skip: 0,
      take: 10,
    });
    
    console.log(`Found ${total} users, showing ${users.length}`);
    return [users, total];
  }
}
```

**Pagination Example:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  async paginate(page: number, limit: number) {
    const [data, total] = await this.userRepository.findAndCount({
      skip: (page - 1) * limit,
      take: limit,
      order: { createdAt: 'DESC' },
    });
    
    return {
      data,
      pagination: {
        total,
        page,
        limit,
        pages: Math.ceil(total / limit),
      },
    };
  }
}
```

**With Filters:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  async searchWithCount(term: string, page: number, limit: number) {
    const [data, total] = await this.userRepository.findAndCount({
      where: [
        { firstName: Like(`%${term}%`) },
        { lastName: Like(`%${term}%`) },
        { email: Like(`%${term}%`) },
      ],
      skip: (page - 1) * limit,
      take: limit,
      order: { createdAt: 'DESC' },
    });
    
    return { data, total };
  }
}
```

**QueryBuilder Alternative:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  async findAndCountWithQueryBuilder(page: number, limit: number) {
    const [data, total] = await this.userRepository
      .createQueryBuilder('user')
      .where('user.isActive = :isActive', { isActive: true })
      .skip((page - 1) * limit)
      .take(limit)
      .orderBy('user.createdAt', 'DESC')
      .getManyAndCount();
    
    return { data, total };
  }
}
```

**Interview Tip**: `findAndCount()` executes two queries (SELECT + COUNT) but returns both results. Returns tuple `[entities, total]`. More efficient than separate `find()` and `count()` calls. Essential for pagination. QueryBuilder has `getManyAndCount()` equivalent.

</details>

## Query Builder

### 23. What is QueryBuilder and when should you use it?

<details>
<summary>Answer</summary>

**QueryBuilder** is TypeORM's powerful API for building complex SQL queries programmatically.

**Basic QueryBuilder:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // Simple QueryBuilder
  async findActiveUsers(): Promise<User[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .where('user.isActive = :isActive', { isActive: true })
      .getMany();
  }
}
```

**When to Use QueryBuilder:**

| Use Case | Use QueryBuilder | Use find() |
|----------|------------------|------------|
| Simple queries | ❌ No | ✅ Yes |
| Complex WHERE with OR/AND | ✅ Yes | ❌ Limited |
| Subqueries | ✅ Yes | ❌ No |
| Raw SQL expressions | ✅ Yes | ❌ No |
| Complex joins | ✅ Yes | ⚠️ Limited |
| Aggregations (COUNT, SUM) | ✅ Yes | ❌ No |
| UNION queries | ✅ Yes | ❌ No |

**Complete QueryBuilder Examples:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // 1. Basic WHERE
  async findByRole(role: string): Promise<User[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .where('user.role = :role', { role })
      .getMany();
  }
  
  // 2. Multiple WHERE conditions (AND)
  async findActiveAdmins(): Promise<User[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .where('user.isActive = :isActive', { isActive: true })
      .andWhere('user.role = :role', { role: 'admin' })
      .getMany();
  }
  
  // 3. OR conditions
  async findByEmailOrUsername(email: string, username: string): Promise<User[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .where('user.email = :email', { email })
      .orWhere('user.username = :username', { username })
      .getMany();
  }
  
  // 4. Complex WHERE with brackets
  async findComplexConditions(): Promise<User[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .where('user.isActive = :isActive', { isActive: true })
      .andWhere(
        new Brackets((qb) => {
          qb.where('user.role = :admin', { admin: 'admin' })
            .orWhere('user.role = :moderator', { moderator: 'moderator' });
        }),
      )
      .getMany();
    // SQL: WHERE isActive = true AND (role = 'admin' OR role = 'moderator')
  }
  
  // 5. LIKE operator
  async search(term: string): Promise<User[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .where('user.firstName LIKE :term', { term: `%${term}%` })
      .orWhere('user.lastName LIKE :term', { term: `%${term}%` })
      .orWhere('user.email LIKE :term', { term: `%${term}%` })
      .getMany();
  }
  
  // 6. IN operator
  async findByIds(ids: number[]): Promise<User[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .where('user.id IN (:...ids)', { ids })
      .getMany();
  }
  
  // 7. BETWEEN operator
  async findByAgeRange(minAge: number, maxAge: number): Promise<User[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .where('user.age BETWEEN :minAge AND :maxAge', { minAge, maxAge })
      .getMany();
  }
  
  // 8. ORDER BY
  async findOrdered(): Promise<User[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .orderBy('user.createdAt', 'DESC')
      .addOrderBy('user.lastName', 'ASC')
      .getMany();
  }
  
  // 9. LIMIT and OFFSET
  async findPaginated(page: number, limit: number): Promise<User[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .skip((page - 1) * limit)
      .take(limit)
      .orderBy('user.createdAt', 'DESC')
      .getMany();
  }
  
  // 10. SELECT specific columns
  async findWithSelect(): Promise<User[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .select(['user.id', 'user.email', 'user.firstName'])
      .getMany();
  }
}
```

**Aggregation Functions:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // COUNT
  async countActiveUsers(): Promise<number> {
    return this.userRepository
      .createQueryBuilder('user')
      .where('user.isActive = :isActive', { isActive: true })
      .getCount();
  }
  
  // SUM
  async getTotalBalance(): Promise<number> {
    const result = await this.userRepository
      .createQueryBuilder('user')
      .select('SUM(user.balance)', 'total')
      .getRawOne();
    
    return result.total || 0;
  }
  
  // AVG, MIN, MAX
  async getStatistics() {
    const result = await this.userRepository
      .createQueryBuilder('user')
      .select('AVG(user.age)', 'avgAge')
      .addSelect('MIN(user.age)', 'minAge')
      .addSelect('MAX(user.age)', 'maxAge')
      .addSelect('COUNT(user.id)', 'totalUsers')
      .getRawOne();
    
    return result;
  }
  
  // GROUP BY
  async countByRole() {
    return this.userRepository
      .createQueryBuilder('user')
      .select('user.role', 'role')
      .addSelect('COUNT(user.id)', 'count')
      .groupBy('user.role')
      .getRawMany();
  }
  
  // HAVING
  async findRolesWithMultipleUsers() {
    return this.userRepository
      .createQueryBuilder('user')
      .select('user.role', 'role')
      .addSelect('COUNT(user.id)', 'count')
      .groupBy('user.role')
      .having('COUNT(user.id) > :min', { min: 5 })
      .getRawMany();
  }
}
```

**Subqueries:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
    @InjectRepository(Post)
    private readonly postRepository: Repository<Post>,
  ) {}
  
  // Subquery in WHERE
  async findUsersWithPosts(): Promise<User[]> {
    const subQuery = this.postRepository
      .createQueryBuilder('post')
      .select('post.authorId')
      .where('post.isPublished = :isPublished', { isPublished: true })
      .getQuery();
    
    return this.userRepository
      .createQueryBuilder('user')
      .where(`user.id IN (${subQuery})`)
      .setParameters({ isPublished: true })
      .getMany();
  }
  
  // Subquery in SELECT
  async findUsersWithPostCount(): Promise<any[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .select('user.id', 'id')
      .addSelect('user.email', 'email')
      .addSelect(
        (subQuery) => {
          return subQuery
            .select('COUNT(post.id)')
            .from(Post, 'post')
            .where('post.authorId = user.id');
        },
        'postCount',
      )
      .getRawMany();
  }
}
```

**Raw SQL:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // Raw WHERE
  async findWithRawSQL(): Promise<User[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .where('LOWER(user.email) = LOWER(:email)', { email: 'john@example.com' })
      .getMany();
  }
  
  // Raw SELECT
  async findWithRawSelect(): Promise<any[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .select('user.id', 'id')
      .addSelect('CONCAT(user.firstName, \' \', user.lastName)', 'fullName')
      .addSelect('YEAR(user.createdAt)', 'year')
      .getRawMany();
  }
}
```

**Interview Tip**: Use QueryBuilder for complex queries: multiple OR/AND conditions with brackets, subqueries, aggregations (COUNT/SUM/AVG), raw SQL expressions, UNION queries. Use `find()` for simple queries. Methods: `where()`, `andWhere()`, `orWhere()`, `orderBy()`, `skip()`, `take()`. Return options: `getMany()`, `getOne()`, `getCount()`, `getRawMany()`, `getRawOne()`.

</details>

### 24. How do you use QueryBuilder for complex queries with joins?

<details>
<summary>Answer</summary>

QueryBuilder provides powerful join capabilities for complex relational queries.

**Join Types:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // INNER JOIN
  async findUsersWithPosts(): Promise<User[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .innerJoinAndSelect('user.posts', 'posts')
      .getMany();
  }
  
  // LEFT JOIN
  async findAllUsersWithOptionalPosts(): Promise<User[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .leftJoinAndSelect('user.posts', 'posts')
      .getMany();
  }
  
  // Multiple joins
  async findWithMultipleJoins(): Promise<User[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .leftJoinAndSelect('user.profile', 'profile')
      .leftJoinAndSelect('user.posts', 'posts')
      .leftJoinAndSelect('posts.comments', 'comments')
      .getMany();
  }
}
```

**Join Methods:**

| Method | Description | Returns |
|--------|-------------|---------|
| `innerJoin()` | INNER JOIN without data | Alias only |
| `innerJoinAndSelect()` | INNER JOIN with data | Joined entities |
| `leftJoin()` | LEFT JOIN without data | Alias only |
| `leftJoinAndSelect()` | LEFT JOIN with data | Joined entities |
| `innerJoinAndMapOne()` | Map to single property | Custom mapping |
| `innerJoinAndMapMany()` | Map to array property | Custom mapping |

**Complete Join Examples:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // 1. Basic join with conditions
  async findUsersWithPublishedPosts(): Promise<User[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .leftJoinAndSelect('user.posts', 'posts', 'posts.isPublished = :isPublished', {
        isPublished: true,
      })
      .getMany();
  }
  
  // 2. Join with WHERE on joined table
  async findUsersHavingPostsWithKeyword(keyword: string): Promise<User[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .leftJoinAndSelect('user.posts', 'posts')
      .where('posts.title LIKE :keyword', { keyword: `%${keyword}%` })
      .getMany();
  }
  
  // 3. Nested joins
  async findUsersWithPostsAndComments(): Promise<User[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .leftJoinAndSelect('user.posts', 'posts')
      .leftJoinAndSelect('posts.comments', 'comments')
      .leftJoinAndSelect('comments.author', 'commentAuthor')
      .orderBy('user.createdAt', 'DESC')
      .addOrderBy('posts.createdAt', 'DESC')
      .addOrderBy('comments.createdAt', 'DESC')
      .getMany();
  }
  
  // 4. Join without selecting (for filtering only)
  async findUsersWithPostCount(): Promise<User[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .leftJoin('user.posts', 'posts')
      .where('posts.id IS NOT NULL')
      .getMany();
  }
  
  // 5. Join with grouping
  async findUsersWithPostCountGrouped(): Promise<any[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .leftJoin('user.posts', 'posts')
      .select('user.id', 'userId')
      .addSelect('user.email', 'email')
      .addSelect('COUNT(posts.id)', 'postCount')
      .groupBy('user.id')
      .getRawMany();
  }
  
  // 6. Self-join (followers example)
  async findUsersWithFollowers(): Promise<User[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .leftJoinAndSelect('user.followers', 'followers')
      .where('followers.isActive = :isActive', { isActive: true })
      .getMany();
  }
}
```

**Join with Custom Mapping:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // Map to custom property
  async findUsersWithLatestPost(): Promise<User[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .innerJoinAndMapOne(
        'user.latestPost',
        Post,
        'post',
        'post.authorId = user.id AND post.id = (SELECT MAX(p.id) FROM post p WHERE p.authorId = user.id)',
      )
      .getMany();
  }
  
  // Map to array
  async findUsersWithRecentPosts(): Promise<User[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .innerJoinAndMapMany(
        'user.recentPosts',
        Post,
        'post',
        'post.authorId = user.id AND post.createdAt > :date',
        { date: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000) },
      )
      .getMany();
  }
}
```

**Complex Query Examples:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // Find users with posts and comments count
  async findUsersWithStats(): Promise<any[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .leftJoin('user.posts', 'posts')
      .leftJoin('posts.comments', 'comments')
      .select('user.id', 'id')
      .addSelect('user.email', 'email')
      .addSelect('user.firstName', 'firstName')
      .addSelect('COUNT(DISTINCT posts.id)', 'postCount')
      .addSelect('COUNT(comments.id)', 'commentCount')
      .groupBy('user.id')
      .getRawMany();
  }
  
  // Find users who have commented on specific post
  async findUsersWhoCommentedOnPost(postId: number): Promise<User[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .innerJoin('user.comments', 'comments')
      .innerJoin('comments.post', 'post')
      .where('post.id = :postId', { postId })
      .distinct(true)
      .getMany();
  }
  
  // Find users with recent activity
  async findActiveUsers(days: number): Promise<User[]> {
    const date = new Date();
    date.setDate(date.getDate() - days);
    
    return this.userRepository
      .createQueryBuilder('user')
      .leftJoinAndSelect('user.posts', 'posts')
      .leftJoinAndSelect('user.comments', 'comments')
      .where(
        new Brackets((qb) => {
          qb.where('posts.createdAt > :date', { date })
            .orWhere('comments.createdAt > :date', { date });
        }),
      )
      .getMany();
  }
  
  // Find users with pagination and filters
  async searchUsersWithFilters(
    search: string,
    role: string,
    page: number,
    limit: number,
  ): Promise<[User[], number]> {
    const query = this.userRepository
      .createQueryBuilder('user')
      .leftJoinAndSelect('user.profile', 'profile')
      .leftJoinAndSelect('user.posts', 'posts');
    
    if (search) {
      query.andWhere(
        new Brackets((qb) => {
          qb.where('user.firstName LIKE :search', { search: `%${search}%` })
            .orWhere('user.lastName LIKE :search', { search: `%${search}%` })
            .orWhere('user.email LIKE :search', { search: `%${search}%` });
        }),
      );
    }
    
    if (role) {
      query.andWhere('user.role = :role', { role });
    }
    
    query
      .skip((page - 1) * limit)
      .take(limit)
      .orderBy('user.createdAt', 'DESC');
    
    return query.getManyAndCount();
  }
}
```

**Performance Optimization:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}
  
  // Use join instead of leftJoinAndSelect for better performance
  async findUserIdsWithPosts(): Promise<number[]> {
    const result = await this.userRepository
      .createQueryBuilder('user')
      .innerJoin('user.posts', 'posts')
      .select('DISTINCT user.id', 'id')
      .getRawMany();
    
    return result.map((r) => r.id);
  }
  
  // Limit joined data
  async findUsersWithRecentPosts(): Promise<User[]> {
    return this.userRepository
      .createQueryBuilder('user')
      .leftJoinAndSelect(
        'user.posts',
        'posts',
        'posts.createdAt > :date',
        { date: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000) },
      )
      .orderBy('posts.createdAt', 'DESC')
      .getMany();
  }
}
```

**Interview Tip**: Join methods: `innerJoinAndSelect()` (only matching), `leftJoinAndSelect()` (all + matching). Can add conditions to joins. Use nested joins for deep relations. Use `innerJoin()` without Select for filtering only. For aggregations, use `leftJoin()` + `groupBy()`. Methods return: `getMany()` (entities), `getRawMany()` (raw objects), `getManyAndCount()` (entities + count).

</details>

## Transactions

### 25. What are transactions and why are they important?

<details>
<summary>Answer</summary>

**Transactions** ensure that a series of database operations either all succeed or all fail together (ACID properties).

**ACID Properties:**

| Property | Description | Example |
|----------|-------------|---------|
| **Atomicity** | All or nothing | Transfer money: debit AND credit both succeed or both fail |
| **Consistency** | Valid state always | Account balance never negative |
| **Isolation** | Independent execution | Concurrent transfers don't interfere |
| **Durability** | Permanent once committed | Data survives system crashes |

**Why Transactions Are Important:**

```typescript
// WITHOUT Transaction - DANGEROUS ❌
@Injectable()
export class BankService {
  async transferMoney(fromId: number, toId: number, amount: number) {
    const fromAccount = await this.accountRepository.findOneBy({ id: fromId });
    fromAccount.balance -= amount;
    await this.accountRepository.save(fromAccount); // ✅ Saved
    
    // 💥 Server crashes here!
    
    const toAccount = await this.accountRepository.findOneBy({ id: toId });
    toAccount.balance += amount;
    await this.accountRepository.save(toAccount); // ❌ Never executed
    
    // Result: Money disappeared! 💸
  }
}

// WITH Transaction - SAFE ✅
@Injectable()
export class BankService {
  async transferMoney(fromId: number, toId: number, amount: number) {
    await this.dataSource.transaction(async (manager) => {
      const fromAccount = await manager.findOneBy(Account, { id: fromId });
      fromAccount.balance -= amount;
      await manager.save(fromAccount);
      
      // 💥 If crash happens here, BOTH operations rollback
      
      const toAccount = await manager.findOneBy(Account, { id: toId });
      toAccount.balance += amount;
      await manager.save(toAccount);
      
      // Both succeed or both fail together ✅
    });
  }
}
```

**Common Use Cases:**

```typescript
@Injectable()
export class OrderService {
  // 1. E-commerce Order
  async createOrder(userId: number, items: any[]) {
    await this.dataSource.transaction(async (manager) => {
      // Create order
      const order = await manager.save(Order, { userId });
      
      // Add order items
      for (const item of items) {
        await manager.save(OrderItem, { orderId: order.id, ...item });
        
        // Reduce stock
        await manager.decrement(Product, { id: item.productId }, 'stock', item.quantity);
      }
      
      // Create payment
      await manager.save(Payment, { orderId: order.id, amount: totalAmount });
    });
  }
  
  // 2. User Registration with Profile
  async registerUser(userData: any, profileData: any) {
    await this.dataSource.transaction(async (manager) => {
      const user = await manager.save(User, userData);
      await manager.save(Profile, { ...profileData, userId: user.id });
      await manager.save(UserSettings, { userId: user.id });
    });
  }
  
  // 3. Batch Update
  async promoteUsers(userIds: number[]) {
    await this.dataSource.transaction(async (manager) => {
      for (const id of userIds) {
        await manager.update(User, id, { role: 'admin' });
        await manager.save(AuditLog, { action: 'promotion', userId: id });
      }
    });
  }
}
```

**Transaction Benefits:**

1. **Data Integrity**: Prevents partial updates
2. **Consistency**: Maintains valid database state
3. **Error Recovery**: Auto-rollback on failure
4. **Concurrent Safety**: Isolates operations
5. **Business Logic**: Enforces multi-step rules

**Interview Tip**: Transactions ensure ACID (Atomicity, Consistency, Isolation, Durability). Use for multi-step operations: money transfers, order processing, batch updates. All operations succeed together or all rollback. Prevents data corruption from partial failures. TypeORM supports transactions via QueryRunner or `@Transaction()` decorator.

</details>

### 26. How do you implement transactions in TypeORM using `QueryRunner`?

<details>
<summary>Answer</summary>

**QueryRunner** provides low-level control over database transactions.

**Basic Transaction Pattern:**

```typescript
@Injectable()
export class UserService {
  constructor(
    private readonly dataSource: DataSource,
  ) {}
  
  async createUserWithProfile(userData: any, profileData: any): Promise<User> {
    // 1. Create query runner
    const queryRunner = this.dataSource.createQueryRunner();
    
    // 2. Connect
    await queryRunner.connect();
    
    // 3. Start transaction
    await queryRunner.startTransaction();
    
    try {
      // 4. Execute operations
      const user = await queryRunner.manager.save(User, userData);
      await queryRunner.manager.save(Profile, { ...profileData, userId: user.id });
      
      // 5. Commit if all succeed
      await queryRunner.commitTransaction();
      
      return user;
    } catch (error) {
      // 6. Rollback on error
      await queryRunner.rollbackTransaction();
      throw error;
    } finally {
      // 7. Release connection
      await queryRunner.release();
    }
  }
}
```

**Complete Examples:**

```typescript
@Injectable()
export class BankService {
  constructor(
    private readonly dataSource: DataSource,
  ) {}
  
  // Money Transfer
  async transferMoney(
    fromAccountId: number,
    toAccountId: number,
    amount: number,
  ): Promise<void> {
    const queryRunner = this.dataSource.createQueryRunner();
    await queryRunner.connect();
    await queryRunner.startTransaction();
    
    try {
      // Get accounts
      const fromAccount = await queryRunner.manager.findOneBy(Account, {
        id: fromAccountId,
      });
      const toAccount = await queryRunner.manager.findOneBy(Account, {
        id: toAccountId,
      });
      
      // Validate
      if (!fromAccount || !toAccount) {
        throw new NotFoundException('Account not found');
      }
      
      if (fromAccount.balance < amount) {
        throw new BadRequestException('Insufficient funds');
      }
      
      // Update balances
      fromAccount.balance -= amount;
      toAccount.balance += amount;
      
      await queryRunner.manager.save(fromAccount);
      await queryRunner.manager.save(toAccount);
      
      // Log transaction
      await queryRunner.manager.save(Transaction, {
        fromAccountId,
        toAccountId,
        amount,
        type: 'transfer',
      });
      
      await queryRunner.commitTransaction();
    } catch (error) {
      await queryRunner.rollbackTransaction();
      throw error;
    } finally {
      await queryRunner.release();
    }
  }
}
```

**Order Processing:**

```typescript
@Injectable()
export class OrderService {
  constructor(
    private readonly dataSource: DataSource,
  ) {}
  
  async createOrder(userId: number, items: OrderItemDto[]): Promise<Order> {
    const queryRunner = this.dataSource.createQueryRunner();
    await queryRunner.connect();
    await queryRunner.startTransaction();
    
    try {
      // 1. Create order
      const order = await queryRunner.manager.save(Order, {
        userId,
        status: 'pending',
        total: 0,
      });
      
      let total = 0;
      
      // 2. Process each item
      for (const item of items) {
        // Get product
        const product = await queryRunner.manager.findOneBy(Product, {
          id: item.productId,
        });
        
        if (!product) {
          throw new NotFoundException(`Product ${item.productId} not found`);
        }
        
        if (product.stock < item.quantity) {
          throw new BadRequestException(`Insufficient stock for ${product.name}`);
        }
        
        // Create order item
        await queryRunner.manager.save(OrderItem, {
          orderId: order.id,
          productId: product.id,
          quantity: item.quantity,
          price: product.price,
        });
        
        // Reduce stock
        product.stock -= item.quantity;
        await queryRunner.manager.save(product);
        
        total += product.price * item.quantity;
      }
      
      // 3. Update order total
      order.total = total;
      await queryRunner.manager.save(order);
      
      // 4. Create payment record
      await queryRunner.manager.save(Payment, {
        orderId: order.id,
        amount: total,
        status: 'pending',
      });
      
      await queryRunner.commitTransaction();
      
      return order;
    } catch (error) {
      await queryRunner.rollbackTransaction();
      throw error;
    } finally {
      await queryRunner.release();
    }
  }
}
```

**Nested Transactions (Savepoints):**

```typescript
@Injectable()
export class UserService {
  constructor(
    private readonly dataSource: DataSource,
  ) {}
  
  async complexOperation(): Promise<void> {
    const queryRunner = this.dataSource.createQueryRunner();
    await queryRunner.connect();
    await queryRunner.startTransaction();
    
    try {
      // Main transaction operations
      await queryRunner.manager.save(User, { name: 'John' });
      
      // Create savepoint
      await queryRunner.startTransaction('savepoint1');
      
      try {
        // Nested operations
        await queryRunner.manager.save(Profile, { bio: 'Test' });
        
        // Commit savepoint
        await queryRunner.commitTransaction();
      } catch (error) {
        // Rollback to savepoint
        await queryRunner.rollbackTransaction();
        // Main transaction continues
      }
      
      // More main transaction operations
      await queryRunner.manager.save(Settings, {});
      
      await queryRunner.commitTransaction();
    } catch (error) {
      await queryRunner.rollbackTransaction();
      throw error;
    } finally {
      await queryRunner.release();
    }
  }
}
```

**Transaction Isolation Levels:**

```typescript
@Injectable()
export class UserService {
  constructor(
    private readonly dataSource: DataSource,
  ) {}
  
  async readWithIsolation(id: number): Promise<User> {
    const queryRunner = this.dataSource.createQueryRunner();
    await queryRunner.connect();
    
    // Set isolation level
    await queryRunner.startTransaction('READ COMMITTED');
    // Options: READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE
    
    try {
      const user = await queryRunner.manager.findOneBy(User, { id });
      await queryRunner.commitTransaction();
      return user;
    } catch (error) {
      await queryRunner.rollbackTransaction();
      throw error;
    } finally {
      await queryRunner.release();
    }
  }
}
```

**Reusable Transaction Helper:**

```typescript
export class TransactionHelper {
  static async runInTransaction<T>(
    dataSource: DataSource,
    work: (manager: EntityManager) => Promise<T>,
  ): Promise<T> {
    const queryRunner = dataSource.createQueryRunner();
    await queryRunner.connect();
    await queryRunner.startTransaction();
    
    try {
      const result = await work(queryRunner.manager);
      await queryRunner.commitTransaction();
      return result;
    } catch (error) {
      await queryRunner.rollbackTransaction();
      throw error;
    } finally {
      await queryRunner.release();
    }
  }
}

// Usage
@Injectable()
export class UserService {
  constructor(private readonly dataSource: DataSource) {}
  
  async createUser(userData: any, profileData: any): Promise<User> {
    return TransactionHelper.runInTransaction(this.dataSource, async (manager) => {
      const user = await manager.save(User, userData);
      await manager.save(Profile, { ...profileData, userId: user.id });
      return user;
    });
  }
}
```

**Interview Tip**: QueryRunner provides transaction control: `createQueryRunner()`, `connect()`, `startTransaction()`, `commitTransaction()`, `rollbackTransaction()`, `release()`. Use try-catch-finally pattern. Always release connection in finally. Use `queryRunner.manager` for database operations. Supports savepoints for nested transactions. Can set isolation levels.

</details>

### 27. How do you use `@Transaction()` decorator?

<details>
<summary>Answer</summary>

The `@Transaction()` decorator provides a simpler way to handle transactions (deprecated in newer TypeORM versions, use QueryRunner instead).

**Basic Usage (Legacy):**

```typescript
import { Transaction, TransactionManager, EntityManager } from 'typeorm';

@Injectable()
export class UserService {
  // Decorator creates transaction automatically
  @Transaction()
  async createUserWithProfile(
    userData: any,
    profileData: any,
    @TransactionManager() manager: EntityManager,
  ): Promise<User> {
    const user = await manager.save(User, userData);
    await manager.save(Profile, { ...profileData, userId: user.id });
    return user;
  }
}
```

**Modern Approach (Recommended):**

```typescript
@Injectable()
export class UserService {
  constructor(
    private readonly dataSource: DataSource,
  ) {}
  
  // Use dataSource.transaction() method
  async createUserWithProfile(userData: any, profileData: any): Promise<User> {
    return this.dataSource.transaction(async (manager) => {
      const user = await manager.save(User, userData);
      await manager.save(Profile, { ...profileData, userId: user.id });
      return user;
    });
  }
}
```

**Comparison:**

```typescript
@Injectable()
export class UserService {
  constructor(
    private readonly dataSource: DataSource,
  ) {}
  
  // ❌ OLD WAY (Deprecated)
  @Transaction()
  async oldWay(
    userData: any,
    @TransactionManager() manager: EntityManager,
  ): Promise<User> {
    return manager.save(User, userData);
  }
  
  // ✅ NEW WAY (Recommended)
  async newWay(userData: any): Promise<User> {
    return this.dataSource.transaction(async (manager) => {
      return manager.save(User, userData);
    });
  }
  
  // ✅ EXPLICIT WAY (Full control)
  async explicitWay(userData: any): Promise<User> {
    const queryRunner = this.dataSource.createQueryRunner();
    await queryRunner.connect();
    await queryRunner.startTransaction();
    
    try {
      const user = await queryRunner.manager.save(User, userData);
      await queryRunner.commitTransaction();
      return user;
    } catch (error) {
      await queryRunner.rollbackTransaction();
      throw error;
    } finally {
      await queryRunner.release();
    }
  }
}
```

**Complete Modern Examples:**

```typescript
@Injectable()
export class OrderService {
  constructor(
    private readonly dataSource: DataSource,
  ) {}
  
  // Simple transaction
  async createOrder(orderData: any, items: any[]): Promise<Order> {
    return this.dataSource.transaction(async (manager) => {
      const order = await manager.save(Order, orderData);
      
      for (const item of items) {
        await manager.save(OrderItem, { ...item, orderId: order.id });
      }
      
      return order;
    });
  }
  
  // Transaction with isolation level
  async criticalOperation(data: any): Promise<void> {
    return this.dataSource.transaction('SERIALIZABLE', async (manager) => {
      // Highest isolation level
      await manager.save(CriticalData, data);
    });
  }
}
```

**Interview Tip**: `@Transaction()` decorator is deprecated. Use modern approaches: `dataSource.transaction()` for simple cases, QueryRunner for complex scenarios. Modern method: `dataSource.transaction(async (manager) => { ... })`. Can specify isolation level as first parameter. Auto-commits on success, auto-rolls back on error.

</details>

## Migrations

### 28. What are migrations and why do you need them?

<details>
<summary>Answer</summary>

**Migrations** are version-controlled database schema changes that can be applied and reverted systematically.

**Why Migrations Are Needed:**

1. **Version Control**: Track database changes like code
2. **Team Collaboration**: Share schema changes across team
3. **Deployment**: Apply changes consistently across environments
4. **Rollback**: Revert changes if needed
5. **Production Safety**: Avoid `synchronize: true` in production

**Problem Without Migrations:**

```typescript
// ❌ DANGEROUS in production
TypeOrmModule.forRoot({
  synchronize: true, // Auto-creates/modifies tables
  // Can cause data loss!
})
```

**Solution With Migrations:**

```typescript
// ✅ SAFE for production
TypeOrmModule.forRoot({
  synchronize: false, // Disabled
  migrations: ['dist/migrations/*{.ts,.js}'],
  migrationsRun: true, // Auto-run on startup
})
```

**Migration Workflow:**

```bash
# 1. Create entity changes
# 2. Generate migration
npm run migration:generate -- -n AddUserRole

# 3. Review generated migration file
# 4. Run migration
npm run migration:run

# 5. If needed, revert
npm run migration:revert
```

**Example Migration:**

```typescript
import { MigrationInterface, QueryRunner } from 'typeorm';

export class AddUserRole1234567890 implements MigrationInterface {
  // Apply changes
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
      ALTER TABLE "user"
      ADD COLUMN "role" varchar(50) DEFAULT 'user'
    `);
  }
  
  // Revert changes
  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
      ALTER TABLE "user"
      DROP COLUMN "role"
    `);
  }
}
```

**Interview Tip**: Migrations are version-controlled database changes. Essential for production (never use `synchronize: true`). Workflow: create entity → generate migration → review → run. Each migration has `up()` (apply) and `down()` (revert) methods. Use TypeORM CLI or npm scripts to manage migrations.

</details>

### 29. How do you generate and run migrations in TypeORM?

<details>
<summary>Answer</summary>

TypeORM provides CLI commands to generate and manage migrations.

**Setup Migration Configuration:**

```typescript
// data-source.ts
import { DataSource } from 'typeorm';

export const AppDataSource = new DataSource({
  type: 'postgres',
  host: 'localhost',
  port: 5432,
  username: 'user',
  password: 'password',
  database: 'mydb',
  entities: ['src/**/*.entity{.ts,.js}'],
  migrations: ['src/migrations/*{.ts,.js}'],
  synchronize: false,
});
```

**Package.json Scripts:**

```json
{
  "scripts": {
    "migration:generate": "typeorm-ts-node-commonjs migration:generate",
    "migration:create": "typeorm-ts-node-commonjs migration:create",
    "migration:run": "typeorm-ts-node-commonjs migration:run",
    "migration:revert": "typeorm-ts-node-commonjs migration:revert",
    "migration:show": "typeorm-ts-node-commonjs migration:show"
  }
}
```

**Generate Migration (Auto-detect changes):**

```bash
# Generate migration from entity changes
npm run migration:generate -- src/migrations/AddUserRole

# TypeORM compares entities with database and generates SQL
```

**Generated Migration Example:**

```typescript
import { MigrationInterface, QueryRunner } from 'typeorm';

export class AddUserRole1703808000000 implements MigrationInterface {
  name = 'AddUserRole1703808000000';
  
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
      ALTER TABLE "user"
      ADD "role" character varying NOT NULL DEFAULT 'user'
    `);
    
    await queryRunner.query(`
      CREATE INDEX "IDX_user_role" ON "user" ("role")
    `);
  }
  
  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP INDEX "IDX_user_role"`);
    await queryRunner.query(`ALTER TABLE "user" DROP COLUMN "role"`);
  }
}
```

**Create Empty Migration (Manual):**

```bash
# Create empty migration file
npm run migration:create -- src/migrations/AddCustomIndex
```

**Manual Migration Example:**

```typescript
import { MigrationInterface, QueryRunner, Table, TableIndex } from 'typeorm';

export class AddCustomIndex1703808000000 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    // Add column
    await queryRunner.addColumn('user', {
      name: 'lastLoginAt',
      type: 'timestamp',
      isNullable: true,
    });
    
    // Create index
    await queryRunner.createIndex(
      'user',
      new TableIndex({
        name: 'IDX_USER_LAST_LOGIN',
        columnNames: ['lastLoginAt'],
      }),
    );
  }
  
  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropIndex('user', 'IDX_USER_LAST_LOGIN');
    await queryRunner.dropColumn('user', 'lastLoginAt');
  }
}
```

**Run Migrations:**

```bash
# Run all pending migrations
npm run migration:run

# Show migration status
npm run migration:show

# Revert last migration
npm run migration:revert
```

**Complex Migration Examples:**

```typescript
export class ComplexMigration1703808000000 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    // Create new table
    await queryRunner.createTable(
      new Table({
        name: 'audit_log',
        columns: [
          {
            name: 'id',
            type: 'int',
            isPrimary: true,
            isGenerated: true,
            generationStrategy: 'increment',
          },
          {
            name: 'action',
            type: 'varchar',
            length: '100',
          },
          {
            name: 'userId',
            type: 'int',
          },
          {
            name: 'createdAt',
            type: 'timestamp',
            default: 'CURRENT_TIMESTAMP',
          },
        ],
      }),
      true,
    );
    
    // Add foreign key
    await queryRunner.createForeignKey('audit_log', {
      columnNames: ['userId'],
      referencedColumnNames: ['id'],
      referencedTableName: 'user',
      onDelete: 'CASCADE',
    });
    
    // Migrate data
    await queryRunner.query(`
      INSERT INTO audit_log (action, userId)
      SELECT 'migration', id FROM user
    `);
  }
  
  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropTable('audit_log');
  }
}
```

**Data Migration:**

```typescript
export class MigrateUserData1703808000000 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    // Add new column
    await queryRunner.addColumn('user', {
      name: 'fullName',
      type: 'varchar',
      isNullable: true,
    });
    
    // Migrate data
    await queryRunner.query(`
      UPDATE "user"
      SET "fullName" = CONCAT("firstName", ' ', "lastName")
      WHERE "fullName" IS NULL
    `);
    
    // Make column required
    await queryRunner.query(`
      ALTER TABLE "user"
      ALTER COLUMN "fullName" SET NOT NULL
    `);
  }
  
  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropColumn('user', 'fullName');
  }
}
```

**Run Migrations in NestJS:**

```typescript
// main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Run migrations on startup
  const dataSource = app.get(DataSource);
  await dataSource.runMigrations();
  
  await app.listen(3000);
}
```

**Interview Tip**: Generate migrations: `typeorm migration:generate -n MigrationName` (auto-detects changes) or `typeorm migration:create -n MigrationName` (empty). Run: `typeorm migration:run`. Revert: `typeorm migration:revert`. Each migration has `up()` and `down()`. Use QueryRunner methods: `createTable()`, `addColumn()`, `createIndex()`, `createForeignKey()`. Can include data migrations. Configure in data-source.ts.

</details>

## Mongoose Integration

### 30. How do you integrate Mongoose with NestJS?

<details>
<summary>Answer</summary>

Mongoose is an ODM (Object Document Mapper) for MongoDB. Integration requires `@nestjs/mongoose` and `mongoose` packages.

**Installation:**

```bash
npm install @nestjs/mongoose mongoose
```

**Basic Integration:**

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { MongooseModule } from '@nestjs/mongoose';

@Module({
  imports: [
    MongooseModule.forRoot('mongodb://localhost/nest'),
  ],
})
export class AppModule {}
```

**With Options:**

```typescript
@Module({
  imports: [
    MongooseModule.forRoot('mongodb://localhost/nest', {
      useNewUrlParser: true,
      useUnifiedTopology: true,
    }),
  ],
})
export class AppModule {}
```

**With Environment Variables:**

```typescript
import { ConfigModule, ConfigService } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot(),
    MongooseModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: async (configService: ConfigService) => ({
        uri: configService.get<string>('MONGODB_URI'),
      }),
      inject: [ConfigService],
    }),
  ],
})
export class AppModule {}
```

**.env File:**

```env
MONGODB_URI=mongodb://localhost:27017/nest
```

**Interview Tip**: Install `@nestjs/mongoose` and `mongoose`. Use `MongooseModule.forRoot()` with connection string. For async config with environment variables, use `forRootAsync()` with ConfigService.

</details>

### 31. What is `MongooseModule` and how do you configure it?

<details>
<summary>Answer</summary>

`MongooseModule` provides Mongoose integration for NestJS with `forRoot()` and `forFeature()` methods.

**forRoot() - Global Connection:**

```typescript
MongooseModule.forRoot('mongodb://localhost/nest', {
  connectionName: 'default',
  useNewUrlParser: true,
  useUnifiedTopology: true,
  maxPoolSize: 10,
  serverSelectionTimeoutMS: 5000,
  socketTimeoutMS: 45000,
})
```

**forFeature() - Register Schemas:**

```typescript
// user.module.ts
import { MongooseModule } from '@nestjs/mongoose';
import { User, UserSchema } from './user.schema';

@Module({
  imports: [
    MongooseModule.forFeature([
      { name: User.name, schema: UserSchema },
    ]),
  ],
  providers: [UserService],
  controllers: [UserController],
})
export class UserModule {}
```

**Multiple Schemas:**

```typescript
@Module({
  imports: [
    MongooseModule.forFeature([
      { name: User.name, schema: UserSchema },
      { name: Post.name, schema: PostSchema },
      { name: Comment.name, schema: CommentSchema },
    ]),
  ],
})
export class UserModule {}
```

**Interview Tip**: `MongooseModule.forRoot()` establishes connection. `forFeature()` registers schemas for a module. Each schema needs `{ name, schema }` object.

</details>

### 32. What is `@Schema()` and `@Prop()` decorator?

<details>
<summary>Answer</summary>

`@Schema()` defines a Mongoose schema class. `@Prop()` defines schema properties.

**Basic Schema:**

```typescript
import { Prop, Schema, SchemaFactory } from '@nestjs/mongoose';
import { Document } from 'mongoose';

@Schema()
export class User extends Document {
  @Prop({ required: true })
  name: string;
  
  @Prop({ required: true, unique: true })
  email: string;
  
  @Prop()
  age: number;
}

export const UserSchema = SchemaFactory.createForClass(User);
```

**Property Options:**

```typescript
@Schema()
export class Product extends Document {
  @Prop({ required: true, minlength: 3, maxlength: 100 })
  name: string;
  
  @Prop({ type: Number, min: 0, default: 0 })
  price: number;
  
  @Prop({ type: [String], default: [] })
  tags: string[];
  
  @Prop({ type: Date, default: Date.now })
  createdAt: Date;
  
  @Prop({ enum: ['active', 'inactive'], default: 'active' })
  status: string;
}

export const ProductSchema = SchemaFactory.createForClass(Product);
```

**Interview Tip**: `@Schema()` decorator creates Mongoose schema from class. `@Prop()` defines fields with options: `required`, `unique`, `default`, `enum`, `min`, `max`. Use `SchemaFactory.createForClass()` to generate schema.

</details>

### 33. How do you inject a Mongoose model using `@InjectModel()`?

<details>
<summary>Answer</summary>

`@InjectModel()` injects a Mongoose model into a service.

**Basic Injection:**

```typescript
import { Injectable } from '@nestjs/common';
import { InjectModel } from '@nestjs/mongoose';
import { Model } from 'mongoose';
import { User } from './user.schema';

@Injectable()
export class UserService {
  constructor(
    @InjectModel(User.name)
    private userModel: Model<User>,
  ) {}
  
  async findAll(): Promise<User[]> {
    return this.userModel.find().exec();
  }
}
```

**Multiple Models:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectModel(User.name)
    private userModel: Model<User>,
    
    @InjectModel(Post.name)
    private postModel: Model<Post>,
  ) {}
}
```

**Interview Tip**: Use `@InjectModel(SchemaName.name)` with type `Model<SchemaClass>`. Must register schema in module with `forFeature()`.

</details>

### 34. How do you perform CRUD operations in Mongoose?

<details>
<summary>Answer</summary>

Mongoose provides methods for Create, Read, Update, Delete operations.

**Complete CRUD Service:**

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectModel(User.name)
    private userModel: Model<User>,
  ) {}
  
  // CREATE
  async create(createUserDto: CreateUserDto): Promise<User> {
    const user = new this.userModel(createUserDto);
    return user.save();
  }
  
  // READ
  async findAll(): Promise<User[]> {
    return this.userModel.find().exec();
  }
  
  async findOne(id: string): Promise<User> {
    return this.userModel.findById(id).exec();
  }
  
  async findByEmail(email: string): Promise<User> {
    return this.userModel.findOne({ email }).exec();
  }
  
  // UPDATE
  async update(id: string, updateUserDto: UpdateUserDto): Promise<User> {
    return this.userModel.findByIdAndUpdate(id, updateUserDto, { new: true }).exec();
  }
  
  // DELETE
  async remove(id: string): Promise<User> {
    return this.userModel.findByIdAndDelete(id).exec();
  }
}
```

**Interview Tip**: CREATE: `new Model()` + `save()` or `Model.create()`. READ: `find()`, `findById()`, `findOne()`. UPDATE: `findByIdAndUpdate()` with `{ new: true }`. DELETE: `findByIdAndDelete()` or `deleteOne()`.

</details>

### 35. What is population in Mongoose?

<details>
<summary>Answer</summary>

**Population** loads referenced documents automatically (like SQL joins).

**Schema with References:**

```typescript
@Schema()
export class Post extends Document {
  @Prop({ required: true })
  title: string;
  
  @Prop({ type: mongoose.Schema.Types.ObjectId, ref: 'User' })
  author: User;
}

export const PostSchema = SchemaFactory.createForClass(Post);
```

**Using Population:**

```typescript
@Injectable()
export class PostService {
  constructor(
    @InjectModel(Post.name)
    private postModel: Model<Post>,
  ) {}
  
  // Populate author
  async findWithAuthor(id: string): Promise<Post> {
    return this.postModel
      .findById(id)
      .populate('author')
      .exec();
  }
  
  // Populate multiple fields
  async findAllWithRelations(): Promise<Post[]> {
    return this.postModel
      .find()
      .populate('author')
      .populate('comments')
      .exec();
  }
  
  // Nested population
  async findWithNestedPopulation(): Promise<Post[]> {
    return this.postModel
      .find()
      .populate({
        path: 'comments',
        populate: { path: 'author' },
      })
      .exec();
  }
}
```

**Interview Tip**: Population loads referenced documents using `ref` in schema. Use `.populate('fieldName')` on queries. Can populate multiple fields and nested references. Similar to SQL joins but happens at application level.

</details>

## Multiple Databases

### 36. How do you configure multiple database connections?

<details>
<summary>Answer</summary>

TypeORM and Mongoose support multiple database connections using named connections.

**TypeORM Multiple Connections:**

```typescript
@Module({
  imports: [
    // PostgreSQL connection
    TypeOrmModule.forRoot({
      name: 'postgres',
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      database: 'main_db',
      entities: [User, Post],
      synchronize: false,
    }),
    
    // MySQL connection
    TypeOrmModule.forRoot({
      name: 'mysql',
      type: 'mysql',
      host: 'localhost',
      port: 3306,
      database: 'analytics_db',
      entities: [Analytics],
      synchronize: false,
    }),
  ],
})
export class AppModule {}
```

**Using Named Connections:**

```typescript
// user.module.ts
@Module({
  imports: [
    TypeOrmModule.forFeature([User, Post], 'postgres'),
    TypeOrmModule.forFeature([Analytics], 'mysql'),
  ],
})
export class UserModule {}

// user.service.ts
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User, 'postgres')
    private userRepository: Repository<User>,
    
    @InjectRepository(Analytics, 'mysql')
    private analyticsRepository: Repository<Analytics>,
  ) {}
}
```

**Mongoose Multiple Connections:**

```typescript
@Module({
  imports: [
    MongooseModule.forRoot('mongodb://localhost/app', {
      connectionName: 'app',
    }),
    MongooseModule.forRoot('mongodb://localhost/logs', {
      connectionName: 'logs',
    }),
  ],
})
export class AppModule {}
```

**Interview Tip**: Use named connections with `name` property. Register entities/schemas with connection name: `forFeature([Entity], 'connectionName')`. Inject with connection name: `@InjectRepository(Entity, 'connectionName')` or `@InjectModel(Schema.name, 'connectionName')`.

</details>

## Performance

### 37. What are N+1 query problems and how do you prevent them?

<details>
<summary>Answer</summary>

**N+1 Problem**: Executing 1 query to fetch parent records, then N queries to fetch related records.

**Problem Example:**

```typescript
// BAD: N+1 queries ❌
async getAllUsersWithPosts(): Promise<User[]> {
  const users = await this.userRepository.find(); // 1 query
  
  for (const user of users) {
    user.posts = await this.postRepository.find({ where: { authorId: user.id } }); // N queries
  }
  
  return users; // Total: 1 + N queries
}
```

**Solution: Use Relations/Joins:**

```typescript
// GOOD: 1 query with JOIN ✅
async getAllUsersWithPosts(): Promise<User[]> {
  return this.userRepository.find({
    relations: ['posts'],
  }); // 1 query with LEFT JOIN
}

// Or with QueryBuilder
async getAllUsersWithPostsQB(): Promise<User[]> {
  return this.userRepository
    .createQueryBuilder('user')
    .leftJoinAndSelect('user.posts', 'posts')
    .getMany(); // 1 query with LEFT JOIN
}
```

**Interview Tip**: N+1 occurs when fetching relations in loops. Solution: Use `relations` option or `leftJoinAndSelect()` in QueryBuilder. Loads all data in single query with JOIN. Use eager loading for frequently accessed relations.

</details>

### 38. What is the difference between eager and lazy loading?

<details>
<summary>Answer</summary>

**Eager Loading**: Automatically loads relations. **Lazy Loading**: Loads relations only when accessed.

**Eager Loading:**

```typescript
@Entity()
export class User {
  @OneToMany(() => Post, post => post.author, {
    eager: true, // Always load posts
  })
  posts: Post[];
}

// Posts automatically loaded
const user = await userRepository.findOneBy({ id: 1 });
console.log(user.posts); // Available immediately
```

**Lazy Loading:**

```typescript
@Entity()
export class User {
  @OneToMany(() => Post, post => post.author)
  posts: Promise<Post[]>; // Note: Promise type for lazy
}

// Posts NOT loaded
const user = await userRepository.findOneBy({ id: 1 });
const posts = await user.posts; // Loads now
```

**Comparison:**

| Feature | Eager Loading | Lazy Loading |
|---------|---------------|--------------|
| **Load Time** | With parent | When accessed |
| **Queries** | 1 (with JOIN) | 1 + on-demand |
| **Performance** | Better if always needed | Better if rarely needed |
| **Type** | Actual type | Promise type |
| **Use Case** | Frequently needed | Rarely needed |

**Interview Tip**: Eager: set `eager: true`, always loads with JOIN. Lazy: default behavior, loads on access (Promise type). Use eager for frequently needed relations, lazy for optional data. Careful with eager - can cause performance issues.

</details>

### 39. How do you optimize database queries?

<details>
<summary>Answer</summary>

**Optimization Techniques:**

1. **Use Indexes**: Speed up WHERE, JOIN, ORDER BY
2. **Select Specific Columns**: Reduce data transfer
3. **Pagination**: Limit results
4. **Avoid N+1**: Use joins
5. **Query Caching**: Cache frequent queries
6. **Connection Pooling**: Reuse connections

**Examples:**

```typescript
// 1. Select specific columns
const users = await this.userRepository.find({
  select: ['id', 'email', 'firstName'],
});

// 2. Use pagination
const users = await this.userRepository.find({
  skip: 0,
  take: 20,
});

// 3. Use indexes (in entity)
@Entity()
@Index(['email'])
@Index(['lastName', 'firstName'])
export class User {
  @Column({ unique: true })
  @Index()
  email: string;
}

// 4. Load only needed relations
const users = await this.userRepository.find({
  relations: ['profile'], // Only profile, not posts
});

// 5. Use caching
const users = await this.userRepository.find({
  where: { isActive: true },
  cache: {
    id: 'active_users',
    milliseconds: 60000,
  },
});

// 6. Use QueryBuilder for complex queries
const users = await this.userRepository
  .createQueryBuilder('user')
  .select(['user.id', 'user.email'])
  .where('user.isActive = :active', { active: true })
  .take(20)
  .getMany();
```

**Interview Tip**: Use indexes on frequently queried columns. Select only needed columns. Use pagination. Avoid N+1 with joins. Enable query caching. Use connection pooling. Monitor slow queries with logging.

</details>

## Testing

### 40. How do you mock repositories in unit tests?

<details>
<summary>Answer</summary>

Use `getRepositoryToken()` to provide mock repositories in tests.

**Basic Test Setup:**

```typescript
describe('UserService', () => {
  let service: UserService;
  let repository: Repository<User>;
  
  const mockRepository = {
    find: jest.fn(),
    findOne: jest.fn(),
    findOneBy: jest.fn(),
    save: jest.fn(),
    create: jest.fn(),
    update: jest.fn(),
    delete: jest.fn(),
  };
  
  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UserService,
        {
          provide: getRepositoryToken(User),
          useValue: mockRepository,
        },
      ],
    }).compile();
    
    service = module.get<UserService>(UserService);
    repository = module.get<Repository<User>>(getRepositoryToken(User));
  });
  
  it('should find all users', async () => {
    const users = [{ id: 1, name: 'John' }];
    mockRepository.find.mockResolvedValue(users);
    
    const result = await service.findAll();
    
    expect(result).toEqual(users);
    expect(mockRepository.find).toHaveBeenCalled();
  });
});
```

**Interview Tip**: Use `getRepositoryToken(Entity)` to mock repositories. Create mock object with jest.fn() for each method. Use `mockResolvedValue()` to mock async returns. Test calls with `toHaveBeenCalled()`.

</details>

### 41. What is `TypeOrmModule.forRootAsync()` used for?

<details>
<summary>Answer</summary>

`forRootAsync()` allows async configuration with dependency injection (e.g., ConfigService).

**Basic Usage:**

```typescript
@Module({
  imports: [
    TypeOrmModule.forRootAsync({
      useFactory: () => ({
        type: 'postgres',
        host: 'localhost',
        port: 5432,
        username: 'user',
        password: 'password',
        database: 'mydb',
        entities: [User],
        synchronize: false,
      }),
    }),
  ],
})
export class AppModule {}
```

**With ConfigService:**

```typescript
@Module({
  imports: [
    ConfigModule.forRoot(),
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: (configService: ConfigService) => ({
        type: 'postgres',
        host: configService.get('DB_HOST'),
        port: configService.get('DB_PORT'),
        username: configService.get('DB_USERNAME'),
        password: configService.get('DB_PASSWORD'),
        database: configService.get('DB_DATABASE'),
        entities: [User],
        synchronize: false,
      }),
      inject: [ConfigService],
    }),
  ],
})
export class AppModule {}
```

**Interview Tip**: Use `forRootAsync()` for async configuration. Allows dependency injection with `inject` array. Use with ConfigService to load environment variables. Provides `useFactory`, `useClass`, or `useExisting` options.

</details>

## Best Practices

### 42. Should business logic be in services or repositories?

<details>
<summary>Answer</summary>

**Business logic belongs in Services**. Repositories should only handle data access.

**Correct Pattern:**

```typescript
// ✅ Repository: Data access only
@Injectable()
export class UserRepository {
  constructor(
    @InjectRepository(User)
    private repository: Repository<User>,
  ) {}
  
  async findById(id: number): Promise<User> {
    return this.repository.findOneBy({ id });
  }
  
  async save(user: User): Promise<User> {
    return this.repository.save(user);
  }
}

// ✅ Service: Business logic
@Injectable()
export class UserService {
  constructor(private userRepository: UserRepository) {}
  
  async registerUser(createUserDto: CreateUserDto): Promise<User> {
    // Validation
    if (!this.isValidEmail(createUserDto.email)) {
      throw new BadRequestException('Invalid email');
    }
    
    // Business rule
    const existingUser = await this.userRepository.findByEmail(createUserDto.email);
    if (existingUser) {
      throw new ConflictException('Email already exists');
    }
    
    // Hash password (business logic)
    const hashedPassword = await bcrypt.hash(createUserDto.password, 10);
    
    // Save
    return this.userRepository.save({
      ...createUserDto,
      password: hashedPassword,
    });
  }
  
  private isValidEmail(email: string): boolean {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }
}
```

**Interview Tip**: Services contain business logic, validation, orchestration. Repositories only do database operations (CRUD). This separation ensures testability, reusability, and follows single responsibility principle.

</details>

### 43. How do you handle database errors?

<details>
<summary>Answer</summary>

Use try-catch blocks and NestJS exception filters to handle database errors gracefully.

**Basic Error Handling:**

```typescript
@Injectable()
export class UserService {
  async create(createUserDto: CreateUserDto): Promise<User> {
    try {
      return await this.userRepository.save(createUserDto);
    } catch (error) {
      if (error.code === '23505') { // PostgreSQL duplicate key
        throw new ConflictException('Email already exists');
      }
      throw new InternalServerException('Database error');
    }
  }
}
```

**Global Exception Filter:**

```typescript
@Catch()
export class DatabaseExceptionFilter implements ExceptionFilter {
  catch(exception: any, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    
    let status = 500;
    let message = 'Internal server error';
    
    if (exception.code === '23505') {
      status = 409;
      message = 'Duplicate entry';
    } else if (exception.code === '23503') {
      status = 400;
      message = 'Foreign key violation';
    }
    
    response.status(status).json({
      statusCode: status,
      message,
      timestamp: new Date().toISOString(),
    });
  }
}
```

**Interview Tip**: Use try-catch in services. Check error codes (23505 = duplicate, 23503 = foreign key). Throw appropriate NestJS exceptions. Use exception filters for global handling. Log errors for debugging.

</details>

### 44. How do you secure database credentials using environment variables?

<details>
<summary>Answer</summary>

Never hardcode credentials. Use environment variables with ConfigModule.

**Setup:**

```bash
npm install @nestjs/config
```

**.env File:**

```env
DB_TYPE=postgres
DB_HOST=localhost
DB_PORT=5432
DB_USERNAME=user
DB_PASSWORD=secretpassword
DB_DATABASE=mydb
DATABASE_URL=postgresql://user:pass@localhost:5432/mydb
```

**Configuration:**

```typescript
@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: '.env',
    }),
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: (configService: ConfigService) => ({
        type: configService.get('DB_TYPE'),
        host: configService.get('DB_HOST'),
        port: configService.get('DB_PORT'),
        username: configService.get('DB_USERNAME'),
        password: configService.get('DB_PASSWORD'),
        database: configService.get('DB_DATABASE'),
        autoLoadEntities: true,
        synchronize: false,
      }),
      inject: [ConfigService],
    }),
  ],
})
export class AppModule {}
```

**.gitignore:**

```
.env
.env.local
.env.production
```

**Interview Tip**: Use `@nestjs/config` with .env files. Never commit .env to version control. Use `ConfigService` to access variables. Different .env files for dev/prod. Use `forRootAsync()` for dynamic configuration.

</details>

### 45. How do you enable query logging in TypeORM?

<details>
<summary>Answer</summary>

Enable logging to see SQL queries executed by TypeORM.

**Basic Logging:**

```typescript
TypeOrmModule.forRoot({
  type: 'postgres',
  host: 'localhost',
  port: 5432,
  database: 'mydb',
  logging: true, // Log all queries
})
```

**Selective Logging:**

```typescript
TypeOrmModule.forRoot({
  logging: ['query', 'error', 'schema', 'warn', 'info', 'log'],
})
```

**Log Only Slow Queries:**

```typescript
TypeOrmModule.forRoot({
  logging: true,
  maxQueryExecutionTime: 1000, // Log queries taking > 1s
})
```

**Custom Logger:**

```typescript
import { Logger } from 'typeorm';

export class CustomLogger implements Logger {
  logQuery(query: string, parameters?: any[]) {
    console.log(`Query: ${query}`);
    console.log(`Parameters: ${JSON.stringify(parameters)}`);
  }
  
  logQueryError(error: string, query: string, parameters?: any[]) {
    console.error(`Query Failed: ${query}`);
    console.error(`Error: ${error}`);
  }
  
  logQuerySlow(time: number, query: string, parameters?: any[]) {
    console.warn(`Slow Query (${time}ms): ${query}`);
  }
  
  logSchemaBuild(message: string) {
    console.log(`Schema: ${message}`);
  }
  
  logMigration(message: string) {
    console.log(`Migration: ${message}`);
  }
  
  log(level: 'log' | 'info' | 'warn', message: any) {
    console[level](message);
  }
}

// Use custom logger
TypeOrmModule.forRoot({
  logger: new CustomLogger(),
  logging: true,
})
```

**Interview Tip**: Enable with `logging: true`. Options: `['query', 'error', 'schema', 'warn', 'info', 'log']`. Use `maxQueryExecutionTime` to log slow queries. Create custom logger for advanced logging. Disable in production for performance.

</details>

---

## Summary

This guide covered 45 comprehensive questions on Database Integration with NestJS, TypeORM, and Mongoose. Key topics include:

- **Database Fundamentals** (Q1-4): Integration options, TypeORM, Mongoose, supported databases
- **TypeORM Setup** (Q5-8): Installation, configuration, TypeOrmModule, environment variables
- **Entities** (Q9-12): Entity decorators, relationships, cascade, eager loading
- **Repository Pattern** (Q13-15): Repository usage, forFeature(), InjectRepository()
- **CRUD Operations** (Q16-19): save/insert, find/findOne, update, delete/remove
- **Querying** (Q20-22): find() options, pagination, findAndCount()
- **Query Builder** (Q23-24): Complex queries, joins, subqueries
- **Transactions** (Q25-27): ACID properties, QueryRunner, transaction patterns
- **Migrations** (Q28-29): Version control, generate/run migrations
- **Mongoose** (Q30-35): MongoDB integration, schemas, models, population
- **Advanced Topics** (Q36-41): Multiple databases, N+1 problems, performance, testing
- **Best Practices** (Q42-45): Architecture, error handling, security, logging

Each answer includes production-ready code examples, comparison tables, and interview tips for effective learning and interview preparation.
