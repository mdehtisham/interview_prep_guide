# NestJS Testing - Top Interview Questions

## Testing Fundamentals

### 1. What testing frameworks does NestJS support?

<details>
<summary>Answer</summary>

**NestJS supports multiple testing frameworks**, but **Jest** is the default and most popular choice.

**Supported Testing Frameworks:**

| Framework | Type | Best For | NestJS Default |
|-----------|------|----------|----------------|
| **Jest** | Full-featured | Unit, Integration, Mocking | ✅ Yes |
| **Mocha** | Test runner | Flexible testing | ❌ No |
| **Jasmine** | BDD framework | Behavior-driven tests | ❌ No |
| **Vitest** | Fast Vite-based | Modern projects | ❌ No |

**Why Jest is Default:**

```typescript
// package.json (NestJS CLI creates this automatically)
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:cov": "jest --coverage",
    "test:debug": "node --inspect-brk -r tsconfig-paths/register -r ts-node/register node_modules/.bin/jest --runInTest",
    "test:e2e": "jest --config ./test/jest-e2e.json"
  },
  "jest": {
    "moduleFileExtensions": ["js", "json", "ts"],
    "rootDir": "src",
    "testRegex": ".*\\.spec\\.ts$",
    "transform": {
      "^.+\\.(t|j)s$": "ts-jest"
    },
    "collectCoverageFrom": [
      "**/*.(t|j)s"
    ],
    "coverageDirectory": "../coverage",
    "testEnvironment": "node"
  }
}
```

**Using Other Frameworks (Example with Mocha):**

```bash
npm uninstall @nestjs/testing jest
npm install --save-dev mocha chai @types/mocha @types/chai
```

```typescript
// test/users.service.spec.ts (Mocha syntax)
import { expect } from 'chai';
import { UsersService } from '../src/users/users.service';

describe('UsersService', () => {
  let service: UsersService;

  beforeEach(() => {
    service = new UsersService();
  });

  it('should create a user', () => {
    const user = service.create({ name: 'John' });
    expect(user).to.have.property('id');
    expect(user.name).to.equal('John');
  });
});
```

**Interview Tip**: NestJS officially supports Jest (default), but you can use Mocha, Jasmine, or Vitest. Jest is recommended because it includes test runner, assertion library, mocking, and coverage tools all-in-one. The NestJS CLI automatically sets up Jest with TypeScript support via `ts-jest`.

</details>

### 2. What is Jest and why is it the default in NestJS?

<details>
<summary>Answer</summary>

**Jest** is a **zero-configuration JavaScript testing framework** developed by Facebook. It's the default in NestJS because of its comprehensive features and simplicity.

**Why Jest is Default:**

| Feature | Benefit |
|---------|---------|
| **Zero Config** | Works out-of-the-box |
| **All-in-One** | Test runner + assertions + mocking |
| **Fast** | Parallel execution, smart test detection |
| **Snapshot Testing** | UI component testing |
| **Code Coverage** | Built-in coverage reports |
| **TypeScript Support** | Works with ts-jest |
| **Mocking** | Powerful mocking capabilities |
| **Watch Mode** | Auto-rerun on file changes |

**Jest Features in NestJS:**

```typescript
// users.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { UsersService } from './users.service';
import { getRepositoryToken } from '@nestjs/typeorm';
import { User } from './entities/user.entity';

describe('UsersService', () => {
  let service: UsersService;
  let mockRepository: any;

  beforeEach(async () => {
    // 1. Setup before each test
    mockRepository = {
      find: jest.fn(),
      findOne: jest.fn(),
      save: jest.fn(),
      remove: jest.fn(),
    };

    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User),
          useValue: mockRepository,
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
  });

  // 2. Test cases with describe/it blocks
  describe('findAll', () => {
    it('should return an array of users', async () => {
      // Arrange
      const users = [{ id: 1, name: 'John' }, { id: 2, name: 'Jane' }];
      mockRepository.find.mockResolvedValue(users);

      // Act
      const result = await service.findAll();

      // Assert
      expect(result).toEqual(users);
      expect(mockRepository.find).toHaveBeenCalledTimes(1);
    });
  });

  // 3. Testing error scenarios
  describe('findOne', () => {
    it('should throw NotFoundException when user not found', async () => {
      mockRepository.findOne.mockResolvedValue(null);

      await expect(service.findOne(999)).rejects.toThrow('User not found');
    });
  });

  // 4. Snapshot testing
  it('should match snapshot', () => {
    const user = { id: 1, name: 'John', email: 'john@example.com' };
    expect(user).toMatchSnapshot();
  });
});
```

**Jest CLI Commands:**

```bash
# Run all tests
npm test

# Watch mode (re-run on changes)
npm run test:watch

# Coverage report
npm run test:cov

# Debug mode
npm run test:debug

# Run specific test file
npm test users.service.spec.ts

# Run tests matching pattern
npm test -- --testNamePattern="findAll"

# Update snapshots
npm test -- -u
```

**Jest Configuration (jest.config.js):**

```javascript
module.exports = {
  moduleFileExtensions: ['js', 'json', 'ts'],
  rootDir: 'src',
  testRegex: '.*\\.spec\\.ts$',
  transform: {
    '^.+\\.(t|j)s$': 'ts-jest',
  },
  collectCoverageFrom: ['**/*.(t|j)s'],
  coverageDirectory: '../coverage',
  testEnvironment: 'node',
  // Advanced options
  setupFilesAfterEnv: ['<rootDir>/../test/setup.ts'],
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/$1',
  },
  testTimeout: 10000,
  verbose: true,
};
```

**Interview Tip**: Jest is the default because it's **all-in-one** (no need for separate assertion/mocking libraries), has **zero configuration**, supports **TypeScript** via ts-jest, and includes **built-in coverage**. NestJS provides `@nestjs/testing` package that works seamlessly with Jest for dependency injection in tests.

</details>

### 3. What are the types of tests (unit, integration, e2e)?

<details>
<summary>Answer</summary>

There are **three main types of tests**: **Unit**, **Integration**, and **E2E (End-to-End)**.

**Comparison:**

| Type | Scope | Speed | Cost | Confidence | Example |
|------|-------|-------|------|------------|---------|
| **Unit** | Single function/class | ⚡ Very Fast | 💰 Low | ⭐⭐ | Test a service method |
| **Integration** | Multiple components | 🏃 Fast | 💰💰 Medium | ⭐⭐⭐ | Test service + repository |
| **E2E** | Full application | 🐌 Slow | 💰💰💰 High | ⭐⭐⭐⭐ | Test API endpoint |

**Test Pyramid:**

```
      /\
     /E2E\      ← Few (10%) - Slow but high confidence
    /------\
   /  INT   \   ← Some (30%) - Medium speed & cost
  /----------\
 /    UNIT    \ ← Many (60%) - Fast & cheap
/--------------\
```

**1. Unit Tests** - Test individual components in isolation:

```typescript
// users.service.spec.ts
describe('UsersService', () => {
  let service: UsersService;
  let mockRepository: jest.Mocked<Repository<User>>;

  beforeEach(async () => {
    mockRepository = {
      findOne: jest.fn(),
      save: jest.fn(),
    } as any;

    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        { provide: getRepositoryToken(User), useValue: mockRepository },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
  });

  // Testing service method in isolation
  it('should create a user', async () => {
    const createDto = { email: 'test@example.com', password: 'pass123' };
    const savedUser = { id: 1, ...createDto };
    
    mockRepository.save.mockResolvedValue(savedUser as User);

    const result = await service.create(createDto);

    expect(result).toEqual(savedUser);
    expect(mockRepository.save).toHaveBeenCalledWith(createDto);
  });
});
```

**2. Integration Tests** - Test multiple components working together:

```typescript
// users.integration.spec.ts
describe('UsersService Integration', () => {
  let service: UsersService;
  let module: TestingModule;
  let repository: Repository<User>;

  beforeAll(async () => {
    // Use real database (test DB or in-memory)
    module = await Test.createTestingModule({
      imports: [
        TypeOrmModule.forRoot({
          type: 'sqlite',
          database: ':memory:',
          entities: [User],
          synchronize: true,
        }),
        TypeOrmModule.forFeature([User]),
      ],
      providers: [UsersService],
    }).compile();

    service = module.get<UsersService>(UsersService);
    repository = module.get<Repository<User>>(getRepositoryToken(User));
  });

  afterAll(async () => {
    await module.close();
  });

  // Testing service with real database
  it('should save user to database and retrieve it', async () => {
    const createDto = { email: 'test@example.com', password: 'pass123' };
    
    const created = await service.create(createDto);
    const found = await service.findOne(created.id);

    expect(found.email).toBe(createDto.email);
  });
});
```

**3. E2E Tests** - Test the entire application through HTTP:

```typescript
// test/users.e2e-spec.ts
describe('Users (e2e)', () => {
  let app: INestApplication;

  beforeAll(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();
  });

  afterAll(async () => {
    await app.close();
  });

  // Testing full HTTP request/response cycle
  it('/users (POST)', () => {
    return request(app.getHttpServer())
      .post('/users')
      .send({ email: 'test@example.com', password: 'pass123' })
      .expect(201)
      .expect((res) => {
        expect(res.body).toHaveProperty('id');
        expect(res.body.email).toBe('test@example.com');
      });
  });

  it('/users/:id (GET)', async () => {
    // Create user first
    const createResponse = await request(app.getHttpServer())
      .post('/users')
      .send({ email: 'test@example.com', password: 'pass123' });

    // Then retrieve it
    return request(app.getHttpServer())
      .get(`/users/${createResponse.body.id}`)
      .expect(200)
      .expect((res) => {
        expect(res.body.email).toBe('test@example.com');
      });
  });
});
```

**When to Use Each Type:**

| Scenario | Test Type |
|----------|-----------|
| Testing business logic | Unit |
| Testing data validation | Unit |
| Testing service + database | Integration |
| Testing module interactions | Integration |
| Testing API endpoints | E2E |
| Testing authentication flow | E2E |
| Testing user workflows | E2E |

**File Naming Conventions:**

```
src/
├── users/
│   ├── users.service.ts
│   ├── users.service.spec.ts       ← Unit test
│   ├── users.controller.ts
│   └── users.controller.spec.ts    ← Unit test
test/
├── users.integration.spec.ts       ← Integration test
└── users.e2e-spec.ts                ← E2E test
```

**Interview Tip**: Follow the **test pyramid** - write mostly **unit tests** (60%), some **integration tests** (30%), and few **E2E tests** (10%). Unit tests are fast and cheap but less confident. E2E tests are slow and expensive but give high confidence. Balance all three types for comprehensive coverage.

</details>
### 4. What is the difference between unit tests and integration tests?

<details>
<summary>Answer</summary>

**Unit tests** test individual components in **isolation** with **mocked dependencies**. **Integration tests** test **multiple components** working together with **real dependencies**.

**Key Differences:**

| Aspect | Unit Tests | Integration Tests |
|--------|-----------|-------------------|
| **Scope** | Single class/function | Multiple components |
| **Dependencies** | Mocked | Real or test doubles |
| **Database** | Mocked repository | Real test DB or in-memory |
| **Speed** | Very fast (<1ms) | Slower (10-100ms) |
| **Isolation** | Complete | Partial |
| **Confidence** | Low-Medium | Medium-High |
| **Debugging** | Easy | Harder |
| **Quantity** | Many (60%) | Some (30%) |

**Unit Test Example** - Service in isolation:

```typescript
// users.service.spec.ts (Unit Test)
describe('UsersService - Unit', () => {
  let service: UsersService;
  let mockRepository: jest.Mocked<Repository<User>>;
  let mockHashService: jest.Mocked<HashService>;

  beforeEach(async () => {
    // Mock all dependencies
    mockRepository = {
      findOne: jest.fn(),
      save: jest.fn(),
      find: jest.fn(),
    } as any;

    mockHashService = {
      hash: jest.fn(),
      compare: jest.fn(),
    } as any;

    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        { provide: getRepositoryToken(User), useValue: mockRepository },
        { provide: HashService, useValue: mockHashService },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
  });

  it('should create a user with hashed password', async () => {
    // Arrange
    const createDto = { email: 'test@example.com', password: 'plain123' };
    const hashedPassword = 'hashed123';
    mockHashService.hash.mockResolvedValue(hashedPassword);
    mockRepository.save.mockResolvedValue({ id: 1, ...createDto, password: hashedPassword } as User);

    // Act
    const result = await service.create(createDto);

    // Assert
    expect(mockHashService.hash).toHaveBeenCalledWith('plain123');
    expect(mockRepository.save).toHaveBeenCalledWith({
      ...createDto,
      password: hashedPassword,
    });
    expect(result.password).toBe(hashedPassword);
  });

  it('should throw error if email exists', async () => {
    const createDto = { email: 'existing@example.com', password: 'pass123' };
    mockRepository.findOne.mockResolvedValue({ id: 1 } as User);

    await expect(service.create(createDto)).rejects.toThrow('Email already exists');
  });
});
```

**Integration Test Example** - Service with real database:

```typescript
// users.integration.spec.ts (Integration Test)
describe('UsersService - Integration', () => {
  let app: INestApplication;
  let service: UsersService;
  let repository: Repository<User>;
  let connection: DataSource;

  beforeAll(async () => {
    // Use real database (SQLite in-memory for testing)
    const module = await Test.createTestingModule({
      imports: [
        TypeOrmModule.forRoot({
          type: 'sqlite',
          database: ':memory:',
          entities: [User],
          synchronize: true,
        }),
        TypeOrmModule.forFeature([User]),
      ],
      providers: [UsersService, HashService],
    }).compile();

    app = module.createNestApplication();
    await app.init();

    service = module.get<UsersService>(UsersService);
    repository = module.get<Repository<User>>(getRepositoryToken(User));
    connection = module.get<DataSource>(DataSource);
  });

  afterEach(async () => {
    // Clean database after each test
    await repository.clear();
  });

  afterAll(async () => {
    await connection.destroy();
    await app.close();
  });

  it('should save user to database and retrieve it', async () => {
    // Act - Real database operations
    const createDto = { email: 'test@example.com', password: 'pass123' };
    const created = await service.create(createDto);
    
    // Verify in database
    const found = await repository.findOne({ where: { id: created.id } });

    // Assert
    expect(found).toBeDefined();
    expect(found.email).toBe(createDto.email);
    expect(found.password).not.toBe(createDto.password); // Should be hashed
  });

  it('should not allow duplicate emails (database constraint)', async () => {
    const createDto = { email: 'test@example.com', password: 'pass123' };
    
    // Create first user
    await service.create(createDto);
    
    // Try to create duplicate
    await expect(service.create(createDto)).rejects.toThrow();
  });

  it('should update user profile', async () => {
    // Create user
    const user = await service.create({ email: 'test@example.com', password: 'pass123' });
    
    // Update user
    await service.update(user.id, { name: 'John Doe' });
    
    // Verify update persisted
    const updated = await repository.findOne({ where: { id: user.id } });
    expect(updated.name).toBe('John Doe');
  });
});
```

**When to Use Unit vs Integration Tests:**

```typescript
// ✅ Unit Test - Testing business logic
it('should calculate discount correctly', () => {
  const price = 100;
  const discount = service.calculateDiscount(price, 0.1);
  expect(discount).toBe(10);
});

// ✅ Integration Test - Testing database operations
it('should save order with items', async () => {
  const order = await orderService.create({
    userId: 1,
    items: [{ productId: 1, quantity: 2 }],
  });
  
  const found = await orderRepository.findOne({
    where: { id: order.id },
    relations: ['items'],
  });
  
  expect(found.items).toHaveLength(1);
});

// ✅ Unit Test - Testing validation
it('should throw error for invalid email', () => {
  expect(() => service.validateEmail('invalid')).toThrow();
});

// ✅ Integration Test - Testing transactions
it('should rollback on error', async () => {
  await expect(
    service.transferFunds({ from: 1, to: 2, amount: 1000 })
  ).rejects.toThrow();
  
  const account = await accountRepository.findOne({ where: { id: 1 } });
  expect(account.balance).toBe(500); // Original balance unchanged
});
```

**Interview Tip**: **Unit tests** mock dependencies and test logic in isolation - fast and easy to debug. **Integration tests** use real components (database, services) - slower but more realistic. Use **unit tests** for business logic/validation, **integration tests** for database operations/transactions. Follow 60% unit, 30% integration, 10% E2E.

</details>

## Unit Testing

### 5. How do you set up unit tests for services?

<details>
<summary>Answer</summary>

**Set up unit tests** using `Test.createTestingModule()` to create an isolated testing module with **mocked dependencies**.

**Complete Setup Pattern:**

```typescript
// users.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { getRepositoryToken } from '@nestjs/typeorm';
import { UsersService } from './users.service';
import { User } from './entities/user.entity';
import { Repository } from 'typeorm';
import { NotFoundException } from '@nestjs/common';

describe('UsersService', () => {
  let service: UsersService;
  let repository: jest.Mocked<Repository<User>>;

  // Mock repository methods
  const mockRepository = {
    find: jest.fn(),
    findOne: jest.fn(),
    save: jest.fn(),
    remove: jest.fn(),
    create: jest.fn(),
  };

  beforeEach(async () => {
    // Create testing module
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          // Provide mock instead of real repository
          provide: getRepositoryToken(User),
          useValue: mockRepository,
        },
      ],
    }).compile();

    // Get service instance
    service = module.get<UsersService>(UsersService);
    repository = module.get(getRepositoryToken(User));

    // Clear mock calls before each test
    jest.clearAllMocks();
  });

  afterEach(() => {
    jest.clearAllMocks();
  });

  // Test suite organized by method
  describe('findAll', () => {
    it('should return an array of users', async () => {
      // Arrange
      const users = [
        { id: 1, email: 'user1@example.com' },
        { id: 2, email: 'user2@example.com' },
      ];
      mockRepository.find.mockResolvedValue(users as User[]);

      // Act
      const result = await service.findAll();

      // Assert
      expect(result).toEqual(users);
      expect(mockRepository.find).toHaveBeenCalledTimes(1);
    });

    it('should return empty array when no users', async () => {
      mockRepository.find.mockResolvedValue([]);

      const result = await service.findAll();

      expect(result).toEqual([]);
    });
  });

  describe('findOne', () => {
    it('should return a user by id', async () => {
      const user = { id: 1, email: 'test@example.com' };
      mockRepository.findOne.mockResolvedValue(user as User);

      const result = await service.findOne(1);

      expect(result).toEqual(user);
      expect(mockRepository.findOne).toHaveBeenCalledWith({ where: { id: 1 } });
    });

    it('should throw NotFoundException when user not found', async () => {
      mockRepository.findOne.mockResolvedValue(null);

      await expect(service.findOne(999)).rejects.toThrow(NotFoundException);
      await expect(service.findOne(999)).rejects.toThrow('User with ID 999 not found');
    });
  });

  describe('create', () => {
    it('should create and save a new user', async () => {
      const createDto = { email: 'new@example.com', password: 'pass123' };
      const savedUser = { id: 1, ...createDto };
      
      mockRepository.create.mockReturnValue(savedUser as User);
      mockRepository.save.mockResolvedValue(savedUser as User);

      const result = await service.create(createDto);

      expect(mockRepository.create).toHaveBeenCalledWith(createDto);
      expect(mockRepository.save).toHaveBeenCalledWith(savedUser);
      expect(result).toEqual(savedUser);
    });
  });

  describe('remove', () => {
    it('should remove a user', async () => {
      const user = { id: 1, email: 'test@example.com' };
      mockRepository.findOne.mockResolvedValue(user as User);
      mockRepository.remove.mockResolvedValue(user as User);

      await service.remove(1);

      expect(mockRepository.findOne).toHaveBeenCalledWith({ where: { id: 1 } });
      expect(mockRepository.remove).toHaveBeenCalledWith(user);
    });

    it('should throw NotFoundException when removing non-existent user', async () => {
      mockRepository.findOne.mockResolvedValue(null);

      await expect(service.remove(999)).rejects.toThrow(NotFoundException);
    });
  });
});
```

**Service with Multiple Dependencies:**

```typescript
// orders.service.spec.ts
describe('OrdersService', () => {
  let service: OrdersService;
  let orderRepository: jest.Mocked<Repository<Order>>;
  let userService: jest.Mocked<UsersService>;
  let paymentService: jest.Mocked<PaymentService>;
  let emailService: jest.Mocked<EmailService>;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        OrdersService,
        {
          provide: getRepositoryToken(Order),
          useValue: {
            find: jest.fn(),
            save: jest.fn(),
          },
        },
        {
          provide: UsersService,
          useValue: {
            findOne: jest.fn(),
          },
        },
        {
          provide: PaymentService,
          useValue: {
            charge: jest.fn(),
          },
        },
        {
          provide: EmailService,
          useValue: {
            sendOrderConfirmation: jest.fn(),
          },
        },
      ],
    }).compile();

    service = module.get<OrdersService>(OrdersService);
    orderRepository = module.get(getRepositoryToken(Order));
    userService = module.get(UsersService);
    paymentService = module.get(PaymentService);
    emailService = module.get(EmailService);
  });

  it('should create order with payment and email', async () => {
    // Arrange
    const user = { id: 1, email: 'user@example.com' };
    const orderDto = { userId: 1, items: [], total: 100 };
    const savedOrder = { id: 1, ...orderDto };
    
    userService.findOne.mockResolvedValue(user as User);
    paymentService.charge.mockResolvedValue({ success: true });
    orderRepository.save.mockResolvedValue(savedOrder as Order);
    emailService.sendOrderConfirmation.mockResolvedValue(undefined);

    // Act
    const result = await service.create(orderDto);

    // Assert
    expect(userService.findOne).toHaveBeenCalledWith(1);
    expect(paymentService.charge).toHaveBeenCalledWith(user, 100);
    expect(orderRepository.save).toHaveBeenCalled();
    expect(emailService.sendOrderConfirmation).toHaveBeenCalledWith(user.email, savedOrder);
    expect(result).toEqual(savedOrder);
  });
});
```

**Best Practices:**

```typescript
// ✅ Good - Clear test names
it('should return 404 when user not found', () => {});

// ❌ Bad - Vague test names
it('should work', () => {});

// ✅ Good - One assertion per test
it('should save user', async () => {
  await service.create(dto);
  expect(mockRepository.save).toHaveBeenCalledWith(dto);
});

// ❌ Bad - Multiple unrelated assertions
it('should do everything', async () => {
  expect(service.findAll()).toBeDefined();
  expect(service.create(dto)).toBeDefined();
  expect(service.remove(1)).toBeDefined();
});

// ✅ Good - Test error scenarios
it('should throw BadRequestException for invalid input', async () => {
  await expect(service.create({ email: 'invalid' })).rejects.toThrow(BadRequestException);
});
```

**Interview Tip**: Use `Test.createTestingModule()` to create isolated testing modules. Mock all dependencies using `useValue` with jest mock objects. Organize tests by method using nested `describe()` blocks. Follow **AAA pattern** (Arrange, Act, Assert). Clear mocks in `beforeEach()` to prevent test interference.

</details>

### 6. What is `Test.createTestingModule()` used for?

<details>
<summary>Answer</summary>

**`Test.createTestingModule()`** creates an **isolated testing module** that mimics NestJS's dependency injection system for tests.

**Purpose:**

| Feature | Description |
|---------|-------------|
| **DI Container** | Creates isolated dependency injection container |
| **Module Mocking** | Allows replacing providers with mocks |
| **Isolation** | Each test gets fresh module instance |
| **Compilation** | Compiles module like real NestJS app |
| **Access** | Get service instances via `module.get()` |

**Basic Usage:**

```typescript
import { Test, TestingModule } from '@nestjs/testing';

describe('UsersService', () => {
  let service: UsersService;
  let module: TestingModule;

  beforeEach(async () => {
    // 1. Create testing module
    module = await Test.createTestingModule({
      providers: [UsersService],
    }).compile();

    // 2. Get service instance
    service = module.get<UsersService>(UsersService);
  });

  afterEach(async () => {
    // 3. Clean up
    await module.close();
  });

  it('should be defined', () => {
    expect(service).toBeDefined();
  });
});
```

**With Mocked Dependencies:**

```typescript
describe('UsersService with mocks', () => {
  let service: UsersService;
  let mockRepository: any;

  beforeEach(async () => {
    // Create mock
    mockRepository = {
      find: jest.fn(),
      findOne: jest.fn(),
      save: jest.fn(),
    };

    // Create module with mocked provider
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User), // Token for TypeORM repository
          useValue: mockRepository,          // Mock implementation
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
  });

  it('should call repository.find', async () => {
    mockRepository.find.mockResolvedValue([]);
    
    await service.findAll();
    
    expect(mockRepository.find).toHaveBeenCalled();
  });
});
```

**Importing Modules:**

```typescript
describe('UsersController', () => {
  let controller: UsersController;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      // Import entire modules
      imports: [
        TypeOrmModule.forRoot({
          type: 'sqlite',
          database: ':memory:',
          entities: [User],
          synchronize: true,
        }),
        UsersModule,
      ],
    }).compile();

    controller = module.get<UsersController>(UsersController);
  });

  it('should be defined', () => {
    expect(controller).toBeDefined();
  });
});
```

**Overriding Providers:**

```typescript
describe('UsersService with overrides', () => {
  let service: UsersService;
  let mockMailService: any;

  beforeEach(async () => {
    mockMailService = { send: jest.fn() };

    const module = await Test.createTestingModule({
      imports: [UsersModule],
    })
      // Override specific provider
      .overrideProvider(MailService)
      .useValue(mockMailService)
      // Override guard
      .overrideGuard(JwtAuthGuard)
      .useValue({ canActivate: () => true })
      .compile();

    service = module.get<UsersService>(UsersService);
  });

  it('should send email on user creation', async () => {
    await service.create({ email: 'test@example.com' });
    
    expect(mockMailService.send).toHaveBeenCalled();
  });
});
```

**Custom Providers:**

```typescript
beforeEach(async () => {
  const module = await Test.createTestingModule({
    providers: [
      UsersService,
      // useClass - provide different class
      {
        provide: CacheService,
        useClass: MockCacheService,
      },
      // useValue - provide object/mock
      {
        provide: ConfigService,
        useValue: { get: jest.fn((key) => 'test-value') },
      },
      // useFactory - dynamic creation
      {
        provide: DatabaseConnection,
        useFactory: () => ({
          connect: jest.fn(),
          disconnect: jest.fn(),
        }),
      },
    ],
  }).compile();

  service = module.get<UsersService>(UsersService);
});
```

**Getting Instances:**

```typescript
const module = await Test.createTestingModule({
  providers: [UsersService, AuthService],
}).compile();

// Get single instance
const userService = module.get<UsersService>(UsersService);

// Get using token
const repository = module.get<Repository<User>>(getRepositoryToken(User));

// Get optional (returns undefined if not found)
const optionalService = module.get<OptionalService>(OptionalService, { strict: false });
```

**Creating NestJS Application:**

```typescript
describe('E2E Test', () => {
  let app: INestApplication;

  beforeAll(async () => {
    const module = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    // Create full NestJS application
    app = module.createNestApplication();
    
    // Apply middleware, pipes, etc.
    app.useGlobalPipes(new ValidationPipe());
    
    await app.init();
  });

  afterAll(async () => {
    await app.close();
  });

  it('should handle HTTP request', () => {
    return request(app.getHttpServer())
      .get('/users')
      .expect(200);
  });
});
```

**Interview Tip**: `Test.createTestingModule()` creates an **isolated DI container** for testing. It mimics NestJS module system but allows **mocking providers** using `useValue`, `useClass`, or `useFactory`. Use `compile()` to finalize module, then `module.get()` to retrieve instances. Use `overrideProvider()` to replace specific dependencies without recreating entire module.

</details>
### 7. How do you mock dependencies in unit tests?

<details>
<summary>Answer</summary>

**Mock dependencies** by providing **mock objects** in the testing module using `useValue`, `useClass`, or `useFactory`.

**Common Mocking Patterns:**

**1. Mock TypeORM Repository:**

```typescript
describe('UsersService', () => {
  let service: UsersService;
  let mockRepository: jest.Mocked<Repository<User>>;

  beforeEach(async () => {
    // Create mock repository
    const mockRepository = {
      find: jest.fn(),
      findOne: jest.fn(),
      save: jest.fn(),
      create: jest.fn(),
      remove: jest.fn(),
      update: jest.fn(),
      delete: jest.fn(),
    };

    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          // Use getRepositoryToken to get the injection token
          provide: getRepositoryToken(User),
          useValue: mockRepository,
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
    mockRepository = module.get(getRepositoryToken(User));
  });

  it('should find all users', async () => {
    const users = [{ id: 1, email: 'test@example.com' }];
    mockRepository.find.mockResolvedValue(users as User[]);

    const result = await service.findAll();

    expect(result).toEqual(users);
    expect(mockRepository.find).toHaveBeenCalledTimes(1);
  });
});
```

**2. Mock Custom Services:**

```typescript
describe('OrdersService', () => {
  let service: OrdersService;
  let mockUserService: jest.Mocked<UsersService>;
  let mockPaymentService: jest.Mocked<PaymentService>;

  beforeEach(async () => {
    // Create service mocks
    mockUserService = {
      findOne: jest.fn(),
      create: jest.fn(),
    } as any;

    mockPaymentService = {
      charge: jest.fn(),
      refund: jest.fn(),
    } as any;

    const module = await Test.createTestingModule({
      providers: [
        OrdersService,
        { provide: UsersService, useValue: mockUserService },
        { provide: PaymentService, useValue: mockPaymentService },
      ],
    }).compile();

    service = module.get<OrdersService>(OrdersService);
  });

  it('should create order and charge payment', async () => {
    const user = { id: 1, email: 'user@example.com' };
    const orderDto = { userId: 1, total: 100 };
    
    mockUserService.findOne.mockResolvedValue(user as User);
    mockPaymentService.charge.mockResolvedValue({ success: true, transactionId: '123' });

    const result = await service.createOrder(orderDto);

    expect(mockUserService.findOne).toHaveBeenCalledWith(1);
    expect(mockPaymentService.charge).toHaveBeenCalledWith(user, 100);
    expect(result).toHaveProperty('transactionId');
  });
});
```

**3. Mock External HTTP Services:**

```typescript
describe('GithubService', () => {
  let service: GithubService;
  let mockHttpService: jest.Mocked<HttpService>;

  beforeEach(async () => {
    // Mock HttpService from @nestjs/axios
    mockHttpService = {
      get: jest.fn(),
      post: jest.fn(),
    } as any;

    const module = await Test.createTestingModule({
      providers: [
        GithubService,
        { provide: HttpService, useValue: mockHttpService },
      ],
    }).compile();

    service = module.get<GithubService>(GithubService);
  });

  it('should fetch user repos from GitHub API', async () => {
    const repos = [{ id: 1, name: 'repo1' }];
    mockHttpService.get.mockReturnValue(
      of({ data: repos }) as any, // Return Observable
    );

    const result = await service.getUserRepos('username');

    expect(mockHttpService.get).toHaveBeenCalledWith('https://api.github.com/users/username/repos');
    expect(result).toEqual(repos);
  });
});
```

**4. Mock ConfigService:**

```typescript
describe('AppService', () => {
  let service: AppService;
  let mockConfigService: jest.Mocked<ConfigService>;

  beforeEach(async () => {
    mockConfigService = {
      get: jest.fn((key: string) => {
        const config = {
          'DATABASE_URL': 'test-db-url',
          'JWT_SECRET': 'test-secret',
          'PORT': 3000,
        };
        return config[key];
      }),
    } as any;

    const module = await Test.createTestingModule({
      providers: [
        AppService,
        { provide: ConfigService, useValue: mockConfigService },
      ],
    }).compile();

    service = module.get<AppService>(AppService);
  });

  it('should get database URL from config', () => {
    const dbUrl = service.getDatabaseUrl();

    expect(mockConfigService.get).toHaveBeenCalledWith('DATABASE_URL');
    expect(dbUrl).toBe('test-db-url');
  });
});
```

**5. Mock with useClass:**

```typescript
// Create mock class
class MockCacheService {
  get = jest.fn();
  set = jest.fn();
  del = jest.fn();
}

describe('UsersService with useClass', () => {
  let service: UsersService;
  let mockCacheService: MockCacheService;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: CacheService,
          useClass: MockCacheService, // Use mock class
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
    mockCacheService = module.get<CacheService>(CacheService) as any;
  });

  it('should cache user data', async () => {
    mockCacheService.set.mockResolvedValue(true);

    await service.cacheUser({ id: 1, email: 'test@example.com' });

    expect(mockCacheService.set).toHaveBeenCalled();
  });
});
```

**6. Mock with useFactory:**

```typescript
describe('UsersService with useFactory', () => {
  let service: UsersService;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: 'DATABASE_CONNECTION',
          useFactory: () => ({
            query: jest.fn(),
            connect: jest.fn(),
            disconnect: jest.fn(),
          }),
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
  });
});
```

**7. Partial Mocking:**

```typescript
describe('UsersService with partial mock', () => {
  let service: UsersService;
  let mockRepository: any;

  beforeEach(async () => {
    // Only mock methods you need
    mockRepository = {
      findOne: jest.fn(),
      save: jest.fn(),
      // Other methods will be undefined
    };

    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        { provide: getRepositoryToken(User), useValue: mockRepository },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
  });
});
```

**Interview Tip**: Mock dependencies using **`useValue`** (most common), **`useClass`**, or **`useFactory`**. For TypeORM repositories, use `getRepositoryToken(Entity)` as the provide token. Create mock objects with `jest.fn()` for each method. Use `mockResolvedValue()` for promises and `mockReturnValue()` for synchronous returns. Always type your mocks with `jest.Mocked<T>` for better TypeScript support.

</details>

### 8. How do you mock repositories and external services?

<details>
<summary>Answer</summary>

**Mock repositories** using `getRepositoryToken()` and **external services** using `useValue` with jest mocks.

**Mocking TypeORM Repository:**

```typescript
import { getRepositoryToken } from '@nestjs/typeorm';
import { Repository } from 'typeorm';

describe('UsersService', () => {
  let service: UsersService;
  let repository: jest.Mocked<Repository<User>>;

  // Create comprehensive repository mock
  const mockRepository = () => ({
    // Query methods
    find: jest.fn(),
    findOne: jest.fn(),
    findOneBy: jest.fn(),
    findAndCount: jest.fn(),
    findBy: jest.fn(),
    
    // Create/Save methods
    create: jest.fn(),
    save: jest.fn(),
    insert: jest.fn(),
    
    // Update methods
    update: jest.fn(),
    
    // Delete methods
    remove: jest.fn(),
    delete: jest.fn(),
    softDelete: jest.fn(),
    
    // Transaction methods
    manager: {
      transaction: jest.fn(),
    },
    
    // Query builder
    createQueryBuilder: jest.fn(() => ({
      where: jest.fn().mockReturnThis(),
      andWhere: jest.fn().mockReturnThis(),
      leftJoinAndSelect: jest.fn().mockReturnThis(),
      getOne: jest.fn(),
      getMany: jest.fn(),
    })),
  });

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User),
          useValue: mockRepository(),
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
    repository = module.get(getRepositoryToken(User));
  });

  it('should find user with relations', async () => {
    const user = { id: 1, email: 'test@example.com', posts: [] };
    repository.findOne.mockResolvedValue(user as User);

    const result = await service.findOneWithPosts(1);

    expect(repository.findOne).toHaveBeenCalledWith({
      where: { id: 1 },
      relations: ['posts'],
    });
    expect(result).toEqual(user);
  });

  it('should use query builder', async () => {
    const users = [{ id: 1, email: 'test@example.com' }];
    const queryBuilder = repository.createQueryBuilder();
    (queryBuilder.getMany as jest.Mock).mockResolvedValue(users);

    const result = await service.searchUsers('test');

    expect(repository.createQueryBuilder).toHaveBeenCalled();
    expect(queryBuilder.where).toHaveBeenCalled();
    expect(result).toEqual(users);
  });
});
```

**Mocking Mongoose Model:**

```typescript
import { getModelToken } from '@nestjs/mongoose';
import { Model } from 'mongoose';

describe('UsersService (Mongoose)', () => {
  let service: UsersService;
  let model: jest.Mocked<Model<User>>;

  const mockModel = () => ({
    find: jest.fn(),
    findById: jest.fn(),
    findOne: jest.fn(),
    create: jest.fn(),
    findByIdAndUpdate: jest.fn(),
    findByIdAndDelete: jest.fn(),
    exec: jest.fn(),
    // Constructor for new Model()
    constructor: jest.fn(),
    save: jest.fn(),
  });

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getModelToken(User.name),
          useValue: mockModel(),
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
    model = module.get(getModelToken(User.name));
  });

  it('should find all users', async () => {
    const users = [{ _id: '1', email: 'test@example.com' }];
    (model.find as jest.Mock).mockReturnValue({
      exec: jest.fn().mockResolvedValue(users),
    });

    const result = await service.findAll();

    expect(model.find).toHaveBeenCalled();
    expect(result).toEqual(users);
  });
});
```

**Mocking External HTTP Service (@nestjs/axios):**

```typescript
import { HttpService } from '@nestjs/axios';
import { of, throwError } from 'rxjs';

describe('GithubService', () => {
  let service: GithubService;
  let httpService: jest.Mocked<HttpService>;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        GithubService,
        {
          provide: HttpService,
          useValue: {
            get: jest.fn(),
            post: jest.fn(),
            put: jest.fn(),
            delete: jest.fn(),
          },
        },
      ],
    }).compile();

    service = module.get<GithubService>(GithubService);
    httpService = module.get(HttpService);
  });

  it('should fetch user data from GitHub', async () => {
    const userData = { login: 'testuser', id: 123 };
    
    // Mock returns Observable
    httpService.get.mockReturnValue(
      of({ data: userData, status: 200, statusText: 'OK', headers: {}, config: {} } as any)
    );

    const result = await service.getUser('testuser');

    expect(httpService.get).toHaveBeenCalledWith('https://api.github.com/users/testuser');
    expect(result).toEqual(userData);
  });

  it('should handle HTTP errors', async () => {
    httpService.get.mockReturnValue(
      throwError(() => new Error('Network error'))
    );

    await expect(service.getUser('invalid')).rejects.toThrow('Network error');
  });
});
```

**Mocking Third-Party Services (Stripe, AWS, etc.):**

```typescript
describe('PaymentService', () => {
  let service: PaymentService;
  let mockStripe: any;

  beforeEach(async () => {
    // Mock Stripe SDK
    mockStripe = {
      charges: {
        create: jest.fn(),
        retrieve: jest.fn(),
      },
      customers: {
        create: jest.fn(),
        retrieve: jest.fn(),
      },
      paymentIntents: {
        create: jest.fn(),
        confirm: jest.fn(),
      },
    };

    const module = await Test.createTestingModule({
      providers: [
        PaymentService,
        {
          provide: 'STRIPE_CLIENT',
          useValue: mockStripe,
        },
      ],
    }).compile();

    service = module.get<PaymentService>(PaymentService);
  });

  it('should create a charge', async () => {
    const charge = { id: 'ch_123', amount: 1000, status: 'succeeded' };
    mockStripe.charges.create.mockResolvedValue(charge);

    const result = await service.charge({ amount: 1000, currency: 'usd' });

    expect(mockStripe.charges.create).toHaveBeenCalledWith({
      amount: 1000,
      currency: 'usd',
    });
    expect(result).toEqual(charge);
  });
});
```

**Mocking AWS S3:**

```typescript
describe('StorageService', () => {
  let service: StorageService;
  let mockS3: any;

  beforeEach(async () => {
    // Mock AWS S3 SDK
    mockS3 = {
      upload: jest.fn().mockReturnValue({
        promise: jest.fn(),
      }),
      getObject: jest.fn().mockReturnValue({
        promise: jest.fn(),
      }),
      deleteObject: jest.fn().mockReturnValue({
        promise: jest.fn(),
      }),
    };

    const module = await Test.createTestingModule({
      providers: [
        StorageService,
        {
          provide: 'S3_CLIENT',
          useValue: mockS3,
        },
      ],
    }).compile();

    service = module.get<StorageService>(StorageService);
  });

  it('should upload file to S3', async () => {
    const uploadResult = { Location: 'https://s3.amazonaws.com/bucket/file.jpg' };
    mockS3.upload().promise.mockResolvedValue(uploadResult);

    const result = await service.uploadFile(Buffer.from('data'), 'file.jpg');

    expect(mockS3.upload).toHaveBeenCalled();
    expect(result).toEqual(uploadResult.Location);
  });
});
```

**Mocking Email Service (Nodemailer):**

```typescript
describe('MailService', () => {
  let service: MailService;
  let mockTransporter: any;

  beforeEach(async () => {
    mockTransporter = {
      sendMail: jest.fn(),
      verify: jest.fn(),
    };

    const module = await Test.createTestingModule({
      providers: [
        MailService,
        {
          provide: 'MAIL_TRANSPORTER',
          useValue: mockTransporter,
        },
      ],
    }).compile();

    service = module.get<MailService>(MailService);
  });

  it('should send email', async () => {
    const info = { messageId: '123', response: '250 Message accepted' };
    mockTransporter.sendMail.mockResolvedValue(info);

    const result = await service.sendEmail({
      to: 'user@example.com',
      subject: 'Test',
      html: '<p>Test</p>',
    });

    expect(mockTransporter.sendMail).toHaveBeenCalledWith({
      to: 'user@example.com',
      subject: 'Test',
      html: '<p>Test</p>',
    });
    expect(result).toEqual(info);
  });
});
```

**Interview Tip**: For **TypeORM repositories**, use `getRepositoryToken(Entity)`. For **Mongoose models**, use `getModelToken(Model.name)`. For **external services** (HTTP, Stripe, AWS), inject with custom token and mock all methods. Use `mockResolvedValue()` for async methods and `mockReturnValue()` for sync. Mock HttpService returns **Observables** using `of()` from RxJS.

</details>

### 9. How do you use `jest.fn()` to create mock functions?

<details>
<summary>Answer</summary>

**`jest.fn()`** creates a **mock function** that tracks calls, arguments, and return values for testing.

**Basic Usage:**

```typescript
describe('jest.fn() basics', () => {
  it('should create a mock function', () => {
    // Create mock
    const mockFn = jest.fn();

    // Call it
    mockFn('arg1', 'arg2');

    // Assert it was called
    expect(mockFn).toHaveBeenCalled();
    expect(mockFn).toHaveBeenCalledTimes(1);
    expect(mockFn).toHaveBeenCalledWith('arg1', 'arg2');
  });

  it('should return a value', () => {
    const mockFn = jest.fn().mockReturnValue('mocked value');

    const result = mockFn();

    expect(result).toBe('mocked value');
  });

  it('should return async value', async () => {
    const mockFn = jest.fn().mockResolvedValue('async value');

    const result = await mockFn();

    expect(result).toBe('async value');
  });

  it('should throw error', () => {
    const mockFn = jest.fn().mockRejectedValue(new Error('Mock error'));

    expect(mockFn()).rejects.toThrow('Mock error');
  });
});
```

**Mock Return Values:**

```typescript
describe('UsersService with jest.fn()', () => {
  let service: UsersService;
  let mockRepository: any;

  beforeEach(async () => {
    mockRepository = {
      // Simple return value
      find: jest.fn().mockReturnValue([]),
      
      // Promise return value
      findOne: jest.fn().mockResolvedValue({ id: 1, email: 'test@example.com' }),
      
      // Multiple different returns
      save: jest.fn()
        .mockResolvedValueOnce({ id: 1, email: 'user1@example.com' })
        .mockResolvedValueOnce({ id: 2, email: 'user2@example.com' }),
      
      // Implementation function
      create: jest.fn((dto) => ({ id: 1, ...dto })),
      
      // Throw error
      remove: jest.fn().mockRejectedValue(new Error('Delete failed')),
    };

    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        { provide: getRepositoryToken(User), useValue: mockRepository },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
  });

  it('should use mockReturnValue for sync', () => {
    const users = mockRepository.find();
    expect(users).toEqual([]);
  });

  it('should use mockResolvedValue for async', async () => {
    const user = await mockRepository.findOne({ where: { id: 1 } });
    expect(user.email).toBe('test@example.com');
  });

  it('should return different values on successive calls', async () => {
    const user1 = await mockRepository.save({});
    const user2 = await mockRepository.save({});
    
    expect(user1.id).toBe(1);
    expect(user2.id).toBe(2);
  });

  it('should use custom implementation', () => {
    const user = mockRepository.create({ email: 'new@example.com' });
    expect(user).toEqual({ id: 1, email: 'new@example.com' });
  });
});
```

**Assertion Methods:**

```typescript
describe('Mock function assertions', () => {
  const mockFn = jest.fn();

  beforeEach(() => {
    mockFn.mockClear(); // Clear call history
  });

  it('should track calls', () => {
    mockFn('arg1');
    mockFn('arg2');

    // Called at least once
    expect(mockFn).toHaveBeenCalled();
    
    // Called exactly N times
    expect(mockFn).toHaveBeenCalledTimes(2);
    
    // Called with specific arguments
    expect(mockFn).toHaveBeenCalledWith('arg1');
    expect(mockFn).toHaveBeenLastCalledWith('arg2');
    
    // First call arguments
    expect(mockFn).toHaveBeenNthCalledWith(1, 'arg1');
    
    // Not called with
    expect(mockFn).not.toHaveBeenCalledWith('arg3');
  });

  it('should access call history', () => {
    mockFn('arg1', 'arg2');
    mockFn('arg3', 'arg4');

    // All calls
    expect(mockFn.mock.calls).toEqual([
      ['arg1', 'arg2'],
      ['arg3', 'arg4'],
    ]);

    // First call
    expect(mockFn.mock.calls[0]).toEqual(['arg1', 'arg2']);

    // Return values
    mockFn.mockReturnValue('result');
    mockFn();
    expect(mockFn.mock.results[2].value).toBe('result');
  });
});
```

**Mock Implementation:**

```typescript
describe('UsersService with mock implementations', () => {
  it('should mock with custom logic', async () => {
    const mockRepository = {
      // Simple implementation
      findOne: jest.fn(async (options) => {
        if (options.where.id === 1) {
          return { id: 1, email: 'test@example.com' };
        }
        return null;
      }),

      // Conditional logic
      save: jest.fn(async (user) => {
        if (!user.email) {
          throw new Error('Email required');
        }
        return { id: 1, ...user };
      }),
    };

    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        { provide: getRepositoryToken(User), useValue: mockRepository },
      ],
    }).compile();

    const service = module.get<UsersService>(UsersService);

    // Test found user
    const user1 = await service.findOne(1);
    expect(user1).toBeDefined();

    // Test not found
    const user2 = await service.findOne(999);
    expect(user2).toBeNull();

    // Test validation error
    await expect(service.create({ email: '' })).rejects.toThrow('Email required');
  });
});
```

**Mocking Class Methods:**

```typescript
class UserService {
  async findOne(id: number) {
    // Real implementation
  }
  
  async create(dto: CreateUserDto) {
    // Real implementation
  }
}

describe('OrdersService', () => {
  it('should mock specific methods', async () => {
    const mockUserService = {
      findOne: jest.fn().mockResolvedValue({ id: 1, email: 'test@example.com' }),
      create: jest.fn().mockResolvedValue({ id: 2, email: 'new@example.com' }),
    };

    const module = await Test.createTestingModule({
      providers: [
        OrdersService,
        { provide: UserService, useValue: mockUserService },
      ],
    }).compile();

    const orderService = module.get<OrdersService>(OrdersService);
    
    await orderService.createOrder({ userId: 1, items: [] });

    expect(mockUserService.findOne).toHaveBeenCalledWith(1);
  });
});
```

**Advanced Patterns:**

```typescript
describe('Advanced jest.fn() patterns', () => {
  it('should mock different returns based on arguments', () => {
    const mockFn = jest.fn((id: number) => {
      if (id === 1) return { id: 1, name: 'John' };
      if (id === 2) return { id: 2, name: 'Jane' };
      return null;
    });

    expect(mockFn(1)).toEqual({ id: 1, name: 'John' });
    expect(mockFn(2)).toEqual({ id: 2, name: 'Jane' });
    expect(mockFn(3)).toBeNull();
  });

  it('should chain mock behaviors', () => {
    const mockFn = jest.fn()
      .mockReturnValueOnce('first')
      .mockReturnValueOnce('second')
      .mockReturnValue('default');

    expect(mockFn()).toBe('first');
    expect(mockFn()).toBe('second');
    expect(mockFn()).toBe('default');
    expect(mockFn()).toBe('default');
  });

  it('should reset mock state', () => {
    const mockFn = jest.fn().mockReturnValue('value');
    
    mockFn();
    expect(mockFn).toHaveBeenCalledTimes(1);
    
    // Reset calls but keep implementation
    mockFn.mockClear();
    expect(mockFn).toHaveBeenCalledTimes(0);
    expect(mockFn()).toBe('value');
    
    // Reset implementation
    mockFn.mockReset();
    expect(mockFn()).toBeUndefined();
    
    // Restore original (only works with jest.spyOn)
    // mockFn.mockRestore();
  });
});
```

**Partial Argument Matching:**

```typescript
describe('Partial argument matching', () => {
  it('should match with expect.any()', () => {
    const mockFn = jest.fn();
    
    mockFn({ id: 1, createdAt: new Date() });

    expect(mockFn).toHaveBeenCalledWith({
      id: expect.any(Number),
      createdAt: expect.any(Date),
    });
  });

  it('should match with expect.objectContaining()', () => {
    const mockFn = jest.fn();
    
    mockFn({ id: 1, email: 'test@example.com', name: 'John', age: 30 });

    expect(mockFn).toHaveBeenCalledWith(
      expect.objectContaining({
        email: 'test@example.com',
      })
    );
  });

  it('should match with expect.arrayContaining()', () => {
    const mockFn = jest.fn();
    
    mockFn([1, 2, 3, 4, 5]);

    expect(mockFn).toHaveBeenCalledWith(
      expect.arrayContaining([2, 4])
    );
  });
});
```

**Interview Tip**: Use **`jest.fn()`** to create mock functions. Use **`mockReturnValue()`** for sync, **`mockResolvedValue()`** for promises, **`mockRejectedValue()`** for errors. Track calls with **`toHaveBeenCalled()`**, **`toHaveBeenCalledWith()`**, **`toHaveBeenCalledTimes()`**. Access call history via **`mockFn.mock.calls`**. Use **`mockClear()`** to reset call count, **`mockReset()`** to reset implementation.

</details>
### 10. How do you use `jest.spyOn()` for mocking methods?

<details>
<summary>Answer</summary>

**`jest.spyOn()`** creates a **spy** on an existing method, allowing you to **mock and track** method calls while preserving the original implementation.

**Difference: jest.fn() vs jest.spyOn():**

| Feature | jest.fn() | jest.spyOn() |
|---------|-----------|-------------|
| **Creates** | New mock function | Spy on existing method |
| **Original** | No original implementation | Preserves original |
| **Use Case** | Create new mocks | Mock existing methods |
| **Restore** | Cannot restore | Can restore with mockRestore() |

**Basic Usage:**

```typescript
describe('jest.spyOn() basics', () => {
  class Calculator {
    add(a: number, b: number): number {
      return a + b;
    }
    
    multiply(a: number, b: number): number {
      return a * b;
    }
  }

  it('should spy on method and track calls', () => {
    const calculator = new Calculator();
    const spy = jest.spyOn(calculator, 'add');

    calculator.add(2, 3);

    expect(spy).toHaveBeenCalledWith(2, 3);
    expect(spy).toHaveBeenCalledTimes(1);
    expect(spy).toHaveReturnedWith(5); // Original implementation runs
    
    spy.mockRestore(); // Restore original
  });

  it('should mock return value', () => {
    const calculator = new Calculator();
    const spy = jest.spyOn(calculator, 'add').mockReturnValue(100);

    const result = calculator.add(2, 3);

    expect(result).toBe(100); // Mocked value, not 5
    expect(spy).toHaveBeenCalledWith(2, 3);
  });

  it('should mock implementation', () => {
    const calculator = new Calculator();
    const spy = jest.spyOn(calculator, 'multiply').mockImplementation((a, b) => {
      return a + b; // Changed behavior
    });

    const result = calculator.multiply(2, 3);

    expect(result).toBe(5); // Addition instead of multiplication
  });
});
```

**Spying on Service Methods:**

```typescript
describe('UsersService with spyOn', () => {
  let service: UsersService;
  let repository: Repository<User>;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      imports: [
        TypeOrmModule.forRoot({
          type: 'sqlite',
          database: ':memory:',
          entities: [User],
          synchronize: true,
        }),
        TypeOrmModule.forFeature([User]),
      ],
      providers: [UsersService],
    }).compile();

    service = module.get<UsersService>(UsersService);
    repository = module.get<Repository<User>>(getRepositoryToken(User));
  });

  it('should spy on repository method', async () => {
    const spy = jest.spyOn(repository, 'find').mockResolvedValue([
      { id: 1, email: 'test@example.com' } as User,
    ]);

    const users = await service.findAll();

    expect(spy).toHaveBeenCalled();
    expect(users).toHaveLength(1);
    
    spy.mockRestore();
  });

  it('should spy on private method (using any)', async () => {
    const spy = jest.spyOn(service as any, 'validateEmail').mockReturnValue(true);

    await service.create({ email: 'test@example.com', password: 'pass' });

    expect(spy).toHaveBeenCalledWith('test@example.com');
  });
});
```

**Spying on External Services:**

```typescript
describe('OrdersService with spyOn', () => {
  let ordersService: OrdersService;
  let paymentService: PaymentService;
  let emailService: EmailService;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [OrdersService, PaymentService, EmailService],
    }).compile();

    ordersService = module.get<OrdersService>(OrdersService);
    paymentService = module.get<PaymentService>(PaymentService);
    emailService = module.get<EmailService>(EmailService);
  });

  it('should call payment and email services', async () => {
    // Spy on real service methods
    const paymentSpy = jest.spyOn(paymentService, 'charge').mockResolvedValue({
      success: true,
      transactionId: 'tx_123',
    });
    
    const emailSpy = jest.spyOn(emailService, 'sendOrderConfirmation').mockResolvedValue(undefined);

    await ordersService.createOrder({
      userId: 1,
      items: [{ productId: 1, quantity: 2 }],
      total: 100,
    });

    expect(paymentSpy).toHaveBeenCalledWith(expect.objectContaining({ total: 100 }));
    expect(emailSpy).toHaveBeenCalled();
  });
});
```

**Spying on Global Objects:**

```typescript
describe('Spying on global objects', () => {
  it('should spy on console.log', () => {
    const spy = jest.spyOn(console, 'log').mockImplementation();

    console.log('test message');

    expect(spy).toHaveBeenCalledWith('test message');
    
    spy.mockRestore();
  });

  it('should spy on Date.now', () => {
    const mockDate = new Date('2024-01-01').getTime();
    const spy = jest.spyOn(Date, 'now').mockReturnValue(mockDate);

    const timestamp = Date.now();

    expect(timestamp).toBe(mockDate);
    expect(spy).toHaveBeenCalled();
    
    spy.mockRestore();
  });

  it('should spy on Math.random', () => {
    const spy = jest.spyOn(Math, 'random').mockReturnValue(0.5);

    const random = Math.random();

    expect(random).toBe(0.5);
    
    spy.mockRestore();
  });
});
```

**Partial Mocking:**

```typescript
describe('Partial mocking with spyOn', () => {
  it('should mock only specific methods', async () => {
    const module = await Test.createTestingModule({
      providers: [UsersService, Repository],
    }).compile();

    const service = module.get<UsersService>(UsersService);
    
    // Only mock findOne, other methods use real implementation
    const findOneSpy = jest.spyOn(service, 'findOne').mockResolvedValue({
      id: 1,
      email: 'test@example.com',
    } as User);

    const user = await service.findOne(1);
    expect(user.id).toBe(1);
    
    // Other methods work normally
    const users = await service.findAll(); // Real implementation
    
    findOneSpy.mockRestore();
  });
});
```

**Spy Cleanup:**

```typescript
describe('Spy cleanup patterns', () => {
  let spy: jest.SpyInstance;

  beforeEach(() => {
    spy = jest.spyOn(console, 'log').mockImplementation();
  });

  afterEach(() => {
    // Always restore spies
    spy.mockRestore();
    // Or use jest.restoreAllMocks() to restore all spies
    jest.restoreAllMocks();
  });

  it('test 1', () => {
    console.log('message 1');
    expect(spy).toHaveBeenCalledWith('message 1');
  });

  it('test 2', () => {
    // Spy is restored and re-created for each test
    console.log('message 2');
    expect(spy).toHaveBeenCalledTimes(1); // Only called once in this test
  });
});
```

**Advanced Patterns:**

```typescript
describe('Advanced spyOn patterns', () => {
  it('should spy on getter', () => {
    const user = {
      _email: 'test@example.com',
      get email() {
        return this._email;
      },
    };

    const spy = jest.spyOn(user, 'email', 'get').mockReturnValue('mocked@example.com');

    expect(user.email).toBe('mocked@example.com');
    
    spy.mockRestore();
  });

  it('should spy on setter', () => {
    const user = {
      _email: '',
      set email(value: string) {
        this._email = value;
      },
    };

    const spy = jest.spyOn(user, 'email', 'set');

    user.email = 'test@example.com';

    expect(spy).toHaveBeenCalledWith('test@example.com');
    
    spy.mockRestore();
  });

  it('should call through original implementation', () => {
    const calculator = {
      add: (a: number, b: number) => a + b,
    };

    const spy = jest.spyOn(calculator, 'add');
    // No mock, calls original

    const result = calculator.add(2, 3);

    expect(result).toBe(5); // Original implementation
    expect(spy).toHaveBeenCalledWith(2, 3);
  });
});
```

**Interview Tip**: Use **`jest.spyOn()`** to spy on existing methods while preserving original behavior. Use **`mockReturnValue()`** or **`mockImplementation()`** to override behavior. Always **`mockRestore()`** in `afterEach()` to clean up. Use for **partial mocking** when you want to mock specific methods while keeping others real. **`jest.fn()`** creates new mocks, **`jest.spyOn()`** spies on existing methods.

</details>

## Testing Controllers

### 11. How do you unit test controllers?

<details>
<summary>Answer</summary>

**Unit test controllers** by mocking all service dependencies and testing request/response handling.

**Basic Controller Test Setup:**

```typescript
// users.controller.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';
import { CreateUserDto } from './dto/create-user.dto';
import { NotFoundException } from '@nestjs/common';

describe('UsersController', () => {
  let controller: UsersController;
  let service: jest.Mocked<UsersService>;

  // Mock service
  const mockUsersService = {
    findAll: jest.fn(),
    findOne: jest.fn(),
    create: jest.fn(),
    update: jest.fn(),
    remove: jest.fn(),
  };

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      controllers: [UsersController],
      providers: [
        {
          provide: UsersService,
          useValue: mockUsersService,
        },
      ],
    }).compile();

    controller = module.get<UsersController>(UsersController);
    service = module.get(UsersService);

    // Clear mocks before each test
    jest.clearAllMocks();
  });

  it('should be defined', () => {
    expect(controller).toBeDefined();
  });

  describe('findAll', () => {
    it('should return an array of users', async () => {
      // Arrange
      const users = [
        { id: 1, email: 'user1@example.com', name: 'User 1' },
        { id: 2, email: 'user2@example.com', name: 'User 2' },
      ];
      mockUsersService.findAll.mockResolvedValue(users);

      // Act
      const result = await controller.findAll();

      // Assert
      expect(result).toEqual(users);
      expect(mockUsersService.findAll).toHaveBeenCalledTimes(1);
    });

    it('should return empty array when no users', async () => {
      mockUsersService.findAll.mockResolvedValue([]);

      const result = await controller.findAll();

      expect(result).toEqual([]);
    });
  });

  describe('findOne', () => {
    it('should return a user by id', async () => {
      const user = { id: 1, email: 'test@example.com', name: 'Test User' };
      mockUsersService.findOne.mockResolvedValue(user);

      const result = await controller.findOne('1');

      expect(result).toEqual(user);
      expect(mockUsersService.findOne).toHaveBeenCalledWith(1);
    });

    it('should throw NotFoundException when user not found', async () => {
      mockUsersService.findOne.mockRejectedValue(
        new NotFoundException('User not found'),
      );

      await expect(controller.findOne('999')).rejects.toThrow(NotFoundException);
    });
  });

  describe('create', () => {
    it('should create a new user', async () => {
      const createDto: CreateUserDto = {
        email: 'new@example.com',
        name: 'New User',
        password: 'password123',
      };
      const createdUser = { id: 1, ...createDto };
      mockUsersService.create.mockResolvedValue(createdUser);

      const result = await controller.create(createDto);

      expect(result).toEqual(createdUser);
      expect(mockUsersService.create).toHaveBeenCalledWith(createDto);
    });

    it('should handle validation errors', async () => {
      const invalidDto = { email: 'invalid' } as CreateUserDto;
      mockUsersService.create.mockRejectedValue(
        new Error('Validation failed'),
      );

      await expect(controller.create(invalidDto)).rejects.toThrow();
    });
  });

  describe('update', () => {
    it('should update a user', async () => {
      const updateDto = { name: 'Updated Name' };
      const updatedUser = { id: 1, email: 'test@example.com', name: 'Updated Name' };
      mockUsersService.update.mockResolvedValue(updatedUser);

      const result = await controller.update('1', updateDto);

      expect(result).toEqual(updatedUser);
      expect(mockUsersService.update).toHaveBeenCalledWith(1, updateDto);
    });
  });

  describe('remove', () => {
    it('should remove a user', async () => {
      mockUsersService.remove.mockResolvedValue({ message: 'User deleted' });

      const result = await controller.remove('1');

      expect(result).toEqual({ message: 'User deleted' });
      expect(mockUsersService.remove).toHaveBeenCalledWith(1);
    });
  });
});
```

**Testing with Query Parameters:**

```typescript
// users.controller.ts
@Get()
findAll(@Query('role') role?: string, @Query('limit') limit?: number) {
  return this.usersService.findAll({ role, limit });
}

// users.controller.spec.ts
describe('findAll with query params', () => {
  it('should filter by role', async () => {
    const users = [{ id: 1, email: 'admin@example.com', role: 'admin' }];
    mockUsersService.findAll.mockResolvedValue(users);

    const result = await controller.findAll('admin', undefined);

    expect(mockUsersService.findAll).toHaveBeenCalledWith({
      role: 'admin',
      limit: undefined,
    });
    expect(result).toEqual(users);
  });

  it('should limit results', async () => {
    const users = [{ id: 1, email: 'test@example.com' }];
    mockUsersService.findAll.mockResolvedValue(users);

    await controller.findAll(undefined, 10);

    expect(mockUsersService.findAll).toHaveBeenCalledWith({
      role: undefined,
      limit: 10,
    });
  });
});
```

**Testing with Request/Response Objects:**

```typescript
// auth.controller.ts
@Post('login')
async login(
  @Body() loginDto: LoginDto,
  @Res({ passthrough: true }) response: Response,
) {
  const tokens = await this.authService.login(loginDto);
  response.cookie('refresh_token', tokens.refresh_token, { httpOnly: true });
  return { access_token: tokens.access_token };
}

// auth.controller.spec.ts
describe('login', () => {
  it('should set refresh token cookie', async () => {
    const loginDto = { email: 'test@example.com', password: 'pass123' };
    const tokens = { access_token: 'access', refresh_token: 'refresh' };
    mockAuthService.login.mockResolvedValue(tokens);

    // Create mock response
    const mockResponse = {
      cookie: jest.fn(),
    } as any;

    const result = await controller.login(loginDto, mockResponse);

    expect(mockResponse.cookie).toHaveBeenCalledWith(
      'refresh_token',
      'refresh',
      expect.objectContaining({ httpOnly: true }),
    );
    expect(result).toEqual({ access_token: 'access' });
  });
});
```

**Testing with Custom Decorators:**

```typescript
// users.controller.ts
@Get('profile')
getProfile(@User() user: UserEntity) {
  return this.usersService.getProfile(user.id);
}

// users.controller.spec.ts
describe('getProfile', () => {
  it('should return user profile', async () => {
    const user = { id: 1, email: 'test@example.com' };
    const profile = { ...user, bio: 'Test bio' };
    mockUsersService.getProfile.mockResolvedValue(profile);

    const result = await controller.getProfile(user as UserEntity);

    expect(mockUsersService.getProfile).toHaveBeenCalledWith(1);
    expect(result).toEqual(profile);
  });
});
```

**Testing Exception Handling:**

```typescript
describe('exception handling', () => {
  it('should throw BadRequestException for invalid data', async () => {
    const invalidDto = { email: 'invalid-email' };
    mockUsersService.create.mockRejectedValue(
      new BadRequestException('Invalid email format'),
    );

    await expect(controller.create(invalidDto as any)).rejects.toThrow(
      BadRequestException,
    );
  });

  it('should throw UnauthorizedException', async () => {
    mockAuthService.login.mockRejectedValue(
      new UnauthorizedException('Invalid credentials'),
    );

    await expect(
      controller.login({ email: 'test@example.com', password: 'wrong' }),
    ).rejects.toThrow(UnauthorizedException);
  });
});
```

**Interview Tip**: Mock all **service dependencies** with `jest.fn()`. Test controller **logic only** - verify correct service methods are called with correct arguments. Don't test business logic (that's in service tests). Test **query params**, **request/response**, **decorators**, and **exception handling**. Use **AAA pattern** (Arrange, Act, Assert).

</details>

### 12. How do you mock services in controller tests?

<details>
<summary>Answer</summary>

**Mock services** using `useValue` with jest mock objects containing all service methods.

**Complete Service Mocking Pattern:**

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';
import { AuthService } from '../auth/auth.service';
import { MailService } from '../mail/mail.service';

describe('UsersController', () => {
  let controller: UsersController;
  let usersService: jest.Mocked<UsersService>;
  let authService: jest.Mocked<AuthService>;
  let mailService: jest.Mocked<MailService>;

  beforeEach(async () => {
    // Create mock objects for all services
    const mockUsersService = {
      findAll: jest.fn(),
      findOne: jest.fn(),
      findByEmail: jest.fn(),
      create: jest.fn(),
      update: jest.fn(),
      remove: jest.fn(),
      activate: jest.fn(),
    };

    const mockAuthService = {
      validateUser: jest.fn(),
      generateToken: jest.fn(),
      verifyToken: jest.fn(),
    };

    const mockMailService = {
      sendWelcomeEmail: jest.fn(),
      sendPasswordResetEmail: jest.fn(),
    };

    const module: TestingModule = await Test.createTestingModule({
      controllers: [UsersController],
      providers: [
        { provide: UsersService, useValue: mockUsersService },
        { provide: AuthService, useValue: mockAuthService },
        { provide: MailService, useValue: mockMailService },
      ],
    }).compile();

    controller = module.get<UsersController>(UsersController);
    usersService = module.get(UsersService);
    authService = module.get(AuthService);
    mailService = module.get(MailService);
  });

  afterEach(() => {
    jest.clearAllMocks();
  });

  describe('create', () => {
    it('should create user and send welcome email', async () => {
      // Arrange
      const createDto = { email: 'new@example.com', password: 'pass123' };
      const createdUser = { id: 1, ...createDto };
      
      usersService.create.mockResolvedValue(createdUser);
      mailService.sendWelcomeEmail.mockResolvedValue(undefined);

      // Act
      const result = await controller.create(createDto);

      // Assert
      expect(usersService.create).toHaveBeenCalledWith(createDto);
      expect(mailService.sendWelcomeEmail).toHaveBeenCalledWith(createdUser.email);
      expect(result).toEqual(createdUser);
    });

    it('should handle email service failure gracefully', async () => {
      const createDto = { email: 'new@example.com', password: 'pass123' };
      const createdUser = { id: 1, ...createDto };
      
      usersService.create.mockResolvedValue(createdUser);
      mailService.sendWelcomeEmail.mockRejectedValue(new Error('Mail service down'));

      // Should still return user even if email fails
      const result = await controller.create(createDto);

      expect(result).toEqual(createdUser);
    });
  });
});
```

**Mocking Service with Different Return Values:**

```typescript
describe('findOne with different scenarios', () => {
  it('should return user when found', async () => {
    const user = { id: 1, email: 'test@example.com' };
    usersService.findOne.mockResolvedValue(user);

    const result = await controller.findOne('1');

    expect(result).toEqual(user);
  });

  it('should handle not found', async () => {
    usersService.findOne.mockResolvedValue(null);

    const result = await controller.findOne('999');

    expect(result).toBeNull();
  });

  it('should handle database errors', async () => {
    usersService.findOne.mockRejectedValue(new Error('Database connection failed'));

    await expect(controller.findOne('1')).rejects.toThrow('Database connection failed');
  });
});
```

**Mocking Multiple Service Calls:**

```typescript
describe('complex operations with multiple services', () => {
  it('should register user with authentication', async () => {
    const registerDto = { email: 'test@example.com', password: 'pass123' };
    const user = { id: 1, ...registerDto };
    const token = 'jwt-token';

    // Mock multiple service calls
    usersService.findByEmail.mockResolvedValue(null); // Email not taken
    usersService.create.mockResolvedValue(user);
    authService.generateToken.mockResolvedValue(token);
    mailService.sendWelcomeEmail.mockResolvedValue(undefined);

    const result = await controller.register(registerDto);

    // Verify call order and arguments
    expect(usersService.findByEmail).toHaveBeenCalledWith(registerDto.email);
    expect(usersService.create).toHaveBeenCalledWith(registerDto);
    expect(authService.generateToken).toHaveBeenCalledWith(user);
    expect(mailService.sendWelcomeEmail).toHaveBeenCalledWith(user.email);
    expect(result).toEqual({ user, token });
  });

  it('should throw error if email already exists', async () => {
    const registerDto = { email: 'existing@example.com', password: 'pass123' };
    usersService.findByEmail.mockResolvedValue({ id: 1, email: registerDto.email });

    await expect(controller.register(registerDto)).rejects.toThrow('Email already exists');
    
    // Verify subsequent services were not called
    expect(usersService.create).not.toHaveBeenCalled();
    expect(authService.generateToken).not.toHaveBeenCalled();
  });
});
```

**Using jest.Mock Type:**

```typescript
describe('UsersController with typed mocks', () => {
  let controller: UsersController;
  let mockUsersService: {
    [K in keyof UsersService]: jest.Mock;
  };

  beforeEach(async () => {
    mockUsersService = {
      findAll: jest.fn(),
      findOne: jest.fn(),
      create: jest.fn(),
      update: jest.fn(),
      remove: jest.fn(),
    } as any;

    const module = await Test.createTestingModule({
      controllers: [UsersController],
      providers: [{ provide: UsersService, useValue: mockUsersService }],
    }).compile();

    controller = module.get<UsersController>(UsersController);
  });

  it('should have type-safe mocks', async () => {
    mockUsersService.findAll.mockResolvedValue([]);
    
    const result = await controller.findAll();
    
    expect(result).toEqual([]);
  });
});
```

**Creating Reusable Mock Factory:**

```typescript
// test/mocks/services.mock.ts
export const createMockUsersService = (): jest.Mocked<UsersService> => ({
  findAll: jest.fn(),
  findOne: jest.fn(),
  create: jest.fn(),
  update: jest.fn(),
  remove: jest.fn(),
} as any);

export const createMockAuthService = (): jest.Mocked<AuthService> => ({
  validateUser: jest.fn(),
  generateToken: jest.fn(),
  verifyToken: jest.fn(),
} as any);

// users.controller.spec.ts
import { createMockUsersService, createMockAuthService } from '../test/mocks/services.mock';

describe('UsersController with factories', () => {
  let controller: UsersController;
  let usersService: jest.Mocked<UsersService>;
  let authService: jest.Mocked<AuthService>;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      controllers: [UsersController],
      providers: [
        { provide: UsersService, useValue: createMockUsersService() },
        { provide: AuthService, useValue: createMockAuthService() },
      ],
    }).compile();

    controller = module.get<UsersController>(UsersController);
    usersService = module.get(UsersService);
    authService = module.get(AuthService);
  });
});
```

**Mocking with Conditional Logic:**

```typescript
describe('conditional mocking', () => {
  it('should handle different user roles', async () => {
    // Mock returns different values based on input
    usersService.findOne.mockImplementation(async (id: number) => {
      if (id === 1) return { id: 1, role: 'admin' };
      if (id === 2) return { id: 2, role: 'user' };
      return null;
    });

    const admin = await controller.findOne('1');
    const user = await controller.findOne('2');
    const notFound = await controller.findOne('999');

    expect(admin.role).toBe('admin');
    expect(user.role).toBe('user');
    expect(notFound).toBeNull();
  });
});
```

**Interview Tip**: Create **mock objects** with all service methods using `jest.fn()`. Provide mocks using **`useValue`** in testing module. Type mocks with **`jest.Mocked<ServiceType>`** for TypeScript support. Use **`mockResolvedValue()`** for promises, **`mockImplementation()`** for conditional logic. Always **`clearAllMocks()`** in `afterEach()`. Create **reusable mock factories** to avoid duplication.

</details>
### 13. How do you test request and response objects?

<details>
<summary>Answer</summary>

**Test request/response objects** by creating mock objects with the properties and methods your controller uses.

**Testing Request Object:**

```typescript
import { Request } from 'express';

describe('UsersController with Request', () => {
  let controller: UsersController;
  let service: jest.Mocked<UsersService>;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      controllers: [UsersController],
      providers: [{ provide: UsersService, useValue: { findAll: jest.fn() } }],
    }).compile();

    controller = module.get<UsersController>(UsersController);
    service = module.get(UsersService);
  });

  // Testing with @Req() decorator
  it('should access request headers', async () => {
    // Create mock request
    const mockRequest = {
      headers: {
        'user-agent': 'Jest Test',
        'authorization': 'Bearer token123',
      },
      ip: '127.0.0.1',
      method: 'GET',
      url: '/users',
      user: { id: 1, email: 'test@example.com' }, // From auth guard
    } as Request;

    // @Get()
    // getUsers(@Req() req: Request) {
    //   return this.usersService.findAll(req.user.id);
    // }
    const result = await controller.getUsers(mockRequest);

    expect(service.findAll).toHaveBeenCalledWith(1);
  });

  it('should access query parameters', () => {
    const mockRequest = {
      query: {
        page: '1',
        limit: '10',
        role: 'admin',
      },
    } as any;

    controller.findAll(mockRequest);

    expect(service.findAll).toHaveBeenCalledWith({
      page: 1,
      limit: 10,
      role: 'admin',
    });
  });

  it('should access request body', async () => {
    const mockRequest = {
      body: {
        email: 'test@example.com',
        password: 'pass123',
      },
    } as Request;

    await controller.login(mockRequest);

    expect(service.login).toHaveBeenCalledWith(mockRequest.body);
  });
});
```

**Testing Response Object:**

```typescript
import { Response } from 'express';

describe('AuthController with Response', () => {
  let controller: AuthController;
  let service: jest.Mocked<AuthService>;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      controllers: [AuthController],
      providers: [{ provide: AuthService, useValue: { login: jest.fn() } }],
    }).compile();

    controller = module.get<AuthController>(AuthController);
    service = module.get(AuthService);
  });

  // Testing @Res() decorator with cookies
  it('should set cookie in response', async () => {
    const mockResponse = {
      cookie: jest.fn(),
      status: jest.fn().mockReturnThis(),
      json: jest.fn(),
    } as unknown as Response;

    const loginDto = { email: 'test@example.com', password: 'pass123' };
    const tokens = { access_token: 'access', refresh_token: 'refresh' };
    service.login.mockResolvedValue(tokens);

    // @Post('login')
    // async login(@Body() dto: LoginDto, @Res({ passthrough: true }) res: Response) {
    //   const tokens = await this.authService.login(dto);
    //   res.cookie('refresh_token', tokens.refresh_token, { httpOnly: true });
    //   return { access_token: tokens.access_token };
    // }
    await controller.login(loginDto, mockResponse);

    expect(mockResponse.cookie).toHaveBeenCalledWith(
      'refresh_token',
      'refresh',
      expect.objectContaining({ httpOnly: true }),
    );
  });

  it('should set custom status code', async () => {
    const mockResponse = {
      status: jest.fn().mockReturnThis(),
      json: jest.fn(),
      send: jest.fn(),
    } as unknown as Response;

    // @Delete(':id')
    // async remove(@Param('id') id: string, @Res() res: Response) {
    //   await this.service.remove(+id);
    //   return res.status(204).send();
    // }
    await controller.remove('1', mockResponse);

    expect(mockResponse.status).toHaveBeenCalledWith(204);
    expect(mockResponse.send).toHaveBeenCalled();
  });

  it('should clear cookie', () => {
    const mockResponse = {
      clearCookie: jest.fn(),
    } as unknown as Response;

    controller.logout(mockResponse);

    expect(mockResponse.clearCookie).toHaveBeenCalledWith('refresh_token');
  });

  it('should set custom headers', () => {
    const mockResponse = {
      setHeader: jest.fn(),
      send: jest.fn(),
    } as unknown as Response;

    controller.download(mockResponse);

    expect(mockResponse.setHeader).toHaveBeenCalledWith(
      'Content-Disposition',
      'attachment; filename="file.pdf"',
    );
  });
});
```

**Testing File Upload (Request with files):**

```typescript
describe('FilesController', () => {
  it('should handle file upload', async () => {
    const mockFile = {
      fieldname: 'file',
      originalname: 'test.jpg',
      encoding: '7bit',
      mimetype: 'image/jpeg',
      buffer: Buffer.from('fake image'),
      size: 1024,
    } as Express.Multer.File;

    const mockRequest = {
      file: mockFile,
      user: { id: 1 },
    } as any;

    // @Post('upload')
    // @UseInterceptors(FileInterceptor('file'))
    // uploadFile(@UploadedFile() file: Express.Multer.File, @Req() req: Request) {
    //   return this.filesService.upload(file, req.user.id);
    // }
    await controller.uploadFile(mockFile, mockRequest);

    expect(service.upload).toHaveBeenCalledWith(mockFile, 1);
  });

  it('should handle multiple files', async () => {
    const mockFiles = [
      { originalname: 'file1.jpg', buffer: Buffer.from('1') },
      { originalname: 'file2.jpg', buffer: Buffer.from('2') },
    ] as Express.Multer.File[];

    await controller.uploadMultiple(mockFiles);

    expect(service.uploadMultiple).toHaveBeenCalledWith(mockFiles);
  });
});
```

**Testing Streaming Responses:**

```typescript
import { StreamableFile } from '@nestjs/common';
import { createReadStream } from 'fs';

describe('FilesController streaming', () => {
  it('should return streamable file', async () => {
    const mockResponse = {
      setHeader: jest.fn(),
    } as unknown as Response;

    jest.spyOn(controller as any, 'getFileStream').mockReturnValue(
      createReadStream('test.txt'),
    );

    const result = await controller.download('test.txt', mockResponse);

    expect(result).toBeInstanceOf(StreamableFile);
    expect(mockResponse.setHeader).toHaveBeenCalled();
  });
});
```

**Testing with Session:**

```typescript
describe('SessionController', () => {
  it('should save data to session', () => {
    const mockRequest = {
      session: {
        user: undefined,
        save: jest.fn((cb) => cb()),
      },
    } as any;

    controller.login({ email: 'test@example.com' }, mockRequest);

    expect(mockRequest.session.user).toBeDefined();
    expect(mockRequest.session.save).toHaveBeenCalled();
  });

  it('should destroy session on logout', () => {
    const mockRequest = {
      session: {
        destroy: jest.fn((cb) => cb()),
      },
    } as any;

    controller.logout(mockRequest);

    expect(mockRequest.session.destroy).toHaveBeenCalled();
  });
});
```

**Interview Tip**: Create **mock Request/Response** objects with only the properties your code uses. Use **`as Request`** or **`as any`** for type casting. Test **cookies** with `res.cookie()` mock, **headers** with `req.headers`, **file uploads** with `Express.Multer.File`, and **sessions** with `req.session`. Use **`mockReturnThis()`** for chainable methods like `res.status().json()`.

</details>

## Testing Services

### 14. How do you unit test services?

<details>
<summary>Answer</summary>

**Unit test services** by mocking all dependencies (repositories, other services) and testing business logic in isolation.

**Complete Service Test Example:**

```typescript
// users.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { getRepositoryToken } from '@nestjs/typeorm';
import { UsersService } from './users.service';
import { User } from './entities/user.entity';
import { Repository } from 'typeorm';
import { NotFoundException, BadRequestException } from '@nestjs/common';
import { HashService } from '../common/hash.service';

describe('UsersService', () => {
  let service: UsersService;
  let repository: jest.Mocked<Repository<User>>;
  let hashService: jest.Mocked<HashService>;

  // Mock repository
  const mockRepository = {
    find: jest.fn(),
    findOne: jest.fn(),
    findOneBy: jest.fn(),
    create: jest.fn(),
    save: jest.fn(),
    update: jest.fn(),
    delete: jest.fn(),
    remove: jest.fn(),
  };

  // Mock hash service
  const mockHashService = {
    hash: jest.fn(),
    compare: jest.fn(),
  };

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User),
          useValue: mockRepository,
        },
        {
          provide: HashService,
          useValue: mockHashService,
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
    repository = module.get(getRepositoryToken(User));
    hashService = module.get(HashService);

    // Clear all mocks before each test
    jest.clearAllMocks();
  });

  describe('findAll', () => {
    it('should return array of users', async () => {
      const users = [
        { id: 1, email: 'user1@example.com', name: 'User 1' },
        { id: 2, email: 'user2@example.com', name: 'User 2' },
      ];
      mockRepository.find.mockResolvedValue(users as User[]);

      const result = await service.findAll();

      expect(result).toEqual(users);
      expect(mockRepository.find).toHaveBeenCalledTimes(1);
    });

    it('should return empty array when no users', async () => {
      mockRepository.find.mockResolvedValue([]);

      const result = await service.findAll();

      expect(result).toEqual([]);
    });

    it('should apply filters', async () => {
      const users = [{ id: 1, email: 'admin@example.com', role: 'admin' }];
      mockRepository.find.mockResolvedValue(users as User[]);

      const result = await service.findAll({ role: 'admin' });

      expect(mockRepository.find).toHaveBeenCalledWith({
        where: { role: 'admin' },
      });
      expect(result).toEqual(users);
    });
  });

  describe('findOne', () => {
    it('should return a user by id', async () => {
      const user = { id: 1, email: 'test@example.com' };
      mockRepository.findOne.mockResolvedValue(user as User);

      const result = await service.findOne(1);

      expect(result).toEqual(user);
      expect(mockRepository.findOne).toHaveBeenCalledWith({ where: { id: 1 } });
    });

    it('should throw NotFoundException when user not found', async () => {
      mockRepository.findOne.mockResolvedValue(null);

      await expect(service.findOne(999)).rejects.toThrow(NotFoundException);
      await expect(service.findOne(999)).rejects.toThrow('User with ID 999 not found');
    });
  });

  describe('create', () => {
    it('should create a new user with hashed password', async () => {
      const createDto = { email: 'new@example.com', password: 'plain123' };
      const hashedPassword = '$2b$10$hashedpassword';
      const user = { id: 1, ...createDto, password: hashedPassword };

      mockRepository.findOne.mockResolvedValue(null); // Email not taken
      mockHashService.hash.mockResolvedValue(hashedPassword);
      mockRepository.create.mockReturnValue(user as User);
      mockRepository.save.mockResolvedValue(user as User);

      const result = await service.create(createDto);

      expect(mockHashService.hash).toHaveBeenCalledWith('plain123');
      expect(mockRepository.create).toHaveBeenCalledWith({
        ...createDto,
        password: hashedPassword,
      });
      expect(mockRepository.save).toHaveBeenCalled();
      expect(result.password).toBe(hashedPassword);
    });

    it('should throw BadRequestException if email exists', async () => {
      const createDto = { email: 'existing@example.com', password: 'pass123' };
      mockRepository.findOne.mockResolvedValue({ id: 1 } as User);

      await expect(service.create(createDto)).rejects.toThrow(BadRequestException);
      await expect(service.create(createDto)).rejects.toThrow('Email already exists');
      
      expect(mockHashService.hash).not.toHaveBeenCalled();
      expect(mockRepository.save).not.toHaveBeenCalled();
    });
  });

  describe('update', () => {
    it('should update user', async () => {
      const updateDto = { name: 'Updated Name' };
      const existingUser = { id: 1, email: 'test@example.com', name: 'Old Name' };
      const updatedUser = { ...existingUser, ...updateDto };

      mockRepository.findOne.mockResolvedValue(existingUser as User);
      mockRepository.save.mockResolvedValue(updatedUser as User);

      const result = await service.update(1, updateDto);

      expect(mockRepository.findOne).toHaveBeenCalledWith({ where: { id: 1 } });
      expect(mockRepository.save).toHaveBeenCalledWith(updatedUser);
      expect(result.name).toBe('Updated Name');
    });

    it('should throw NotFoundException when updating non-existent user', async () => {
      mockRepository.findOne.mockResolvedValue(null);

      await expect(service.update(999, { name: 'Test' })).rejects.toThrow(NotFoundException);
    });
  });

  describe('remove', () => {
    it('should remove user', async () => {
      const user = { id: 1, email: 'test@example.com' };
      mockRepository.findOne.mockResolvedValue(user as User);
      mockRepository.remove.mockResolvedValue(user as User);

      await service.remove(1);

      expect(mockRepository.findOne).toHaveBeenCalledWith({ where: { id: 1 } });
      expect(mockRepository.remove).toHaveBeenCalledWith(user);
    });

    it('should throw NotFoundException when removing non-existent user', async () => {
      mockRepository.findOne.mockResolvedValue(null);

      await expect(service.remove(999)).rejects.toThrow(NotFoundException);
    });
  });

  describe('validateUser', () => {
    it('should return user when credentials are valid', async () => {
      const user = { id: 1, email: 'test@example.com', password: 'hashed' };
      mockRepository.findOne.mockResolvedValue(user as User);
      mockHashService.compare.mockResolvedValue(true);

      const result = await service.validateUser('test@example.com', 'plain');

      expect(mockHashService.compare).toHaveBeenCalledWith('plain', 'hashed');
      expect(result).toEqual(user);
    });

    it('should return null when password is invalid', async () => {
      const user = { id: 1, email: 'test@example.com', password: 'hashed' };
      mockRepository.findOne.mockResolvedValue(user as User);
      mockHashService.compare.mockResolvedValue(false);

      const result = await service.validateUser('test@example.com', 'wrong');

      expect(result).toBeNull();
    });

    it('should return null when user not found', async () => {
      mockRepository.findOne.mockResolvedValue(null);

      const result = await service.validateUser('notfound@example.com', 'pass');

      expect(result).toBeNull();
      expect(mockHashService.compare).not.toHaveBeenCalled();
    });
  });
});
```

**Interview Tip**: Mock **all dependencies** (repositories, services). Test **business logic**, **validation**, **error handling**, and **edge cases**. Organize tests by method using nested `describe()`. Use **AAA pattern** (Arrange, Act, Assert). Test both **success** and **failure** scenarios. Verify mocks were called with correct arguments using **`toHaveBeenCalledWith()`**.

</details>

### 15. How do you inject mocked dependencies into services?

<details>
<summary>Answer</summary>

**Inject mocked dependencies** using `Test.createTestingModule()` with `providers` array and `useValue`/`useClass`/`useFactory`.

**Basic Dependency Injection:**

```typescript
describe('UsersService with injected mocks', () => {
  let service: UsersService;
  let repository: jest.Mocked<Repository<User>>;
  let mailService: jest.Mocked<MailService>;
  let cacheService: jest.Mocked<CacheService>;

  beforeEach(async () => {
    // Create mocks
    const mockRepository = {
      find: jest.fn(),
      findOne: jest.fn(),
      save: jest.fn(),
    };

    const mockMailService = {
      send: jest.fn(),
    };

    const mockCacheService = {
      get: jest.fn(),
      set: jest.fn(),
      del: jest.fn(),
    };

    // Create testing module with mocked providers
    const module = await Test.createTestingModule({
      providers: [
        UsersService, // Service under test
        {
          provide: getRepositoryToken(User), // TypeORM repository
          useValue: mockRepository,
        },
        {
          provide: MailService, // Custom service
          useValue: mockMailService,
        },
        {
          provide: CacheService, // Another service
          useValue: mockCacheService,
        },
      ],
    }).compile();

    // Get instances
    service = module.get<UsersService>(UsersService);
    repository = module.get(getRepositoryToken(User));
    mailService = module.get(MailService);
    cacheService = module.get(CacheService);
  });

  it('should inject all dependencies', () => {
    expect(service).toBeDefined();
    expect(repository).toBeDefined();
    expect(mailService).toBeDefined();
    expect(cacheService).toBeDefined();
  });

  it('should use injected dependencies', async () => {
    const user = { id: 1, email: 'test@example.com' };
    repository.findOne.mockResolvedValue(user as User);
    cacheService.get.mockResolvedValue(null);
    cacheService.set.mockResolvedValue(undefined);

    await service.findOne(1);

    expect(cacheService.get).toHaveBeenCalledWith('user:1');
    expect(repository.findOne).toHaveBeenCalled();
    expect(cacheService.set).toHaveBeenCalledWith('user:1', user);
  });
});
```

**Injecting with useClass:**

```typescript
// Create mock class
class MockMailService {
  send = jest.fn().mockResolvedValue(true);
}

describe('UsersService with useClass', () => {
  let service: UsersService;
  let mailService: MockMailService;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: MailService,
          useClass: MockMailService, // Use mock class
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
    mailService = module.get<MailService>(MailService) as any;
  });

  it('should use mock class methods', async () => {
    await service.sendWelcomeEmail('test@example.com');

    expect(mailService.send).toHaveBeenCalled();
  });
});
```

**Injecting with useFactory:**

```typescript
describe('UsersService with useFactory', () => {
  let service: UsersService;
  let mockConfigValue: string;

  beforeEach(async () => {
    mockConfigValue = 'test-value';

    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: ConfigService,
          useFactory: () => ({
            get: jest.fn((key: string) => {
              const config = {
                DATABASE_URL: 'test-db',
                JWT_SECRET: mockConfigValue,
              };
              return config[key];
            }),
          }),
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
  });

  it('should use factory-created mock', () => {
    const secret = service.getJwtSecret();
    expect(secret).toBe('test-value');
  });
});
```

**Injecting Custom Tokens:**

```typescript
// users.service.ts
export class UsersService {
  constructor(
    @InjectRepository(User)
    private repository: Repository<User>,
    @Inject('MAIL_SERVICE')
    private mailService: MailService,
    @Inject('CONFIG_OPTIONS')
    private config: ConfigOptions,
  ) {}
}

// users.service.spec.ts
describe('UsersService with custom tokens', () => {
  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User),
          useValue: mockRepository,
        },
        {
          provide: 'MAIL_SERVICE', // Custom token
          useValue: mockMailService,
        },
        {
          provide: 'CONFIG_OPTIONS', // Custom token
          useValue: { apiKey: 'test-key' },
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
  });
});
```

**Overriding Module Providers:**

```typescript
describe('UsersService with overrides', () => {
  let service: UsersService;
  let mockMailService: any;

  beforeEach(async () => {
    mockMailService = { send: jest.fn() };

    const module = await Test.createTestingModule({
      imports: [UsersModule], // Import real module
    })
      .overrideProvider(MailService) // Override specific provider
      .useValue(mockMailService)
      .compile();

    service = module.get<UsersService>(UsersService);
  });

  it('should use overridden provider', async () => {
    await service.sendEmail('test@example.com');
    expect(mockMailService.send).toHaveBeenCalled();
  });
});
```

**Interview Tip**: Use **`useValue`** for mock objects (most common), **`useClass`** for mock classes, **`useFactory`** for dynamic mocks. For **TypeORM repositories**, use `getRepositoryToken(Entity)`. For **custom injection tokens**, provide the same token string. Use **`overrideProvider()`** to replace providers from imported modules.

</details>

### 16. How do you test async methods?

<details>
<summary>Answer</summary>

**Test async methods** using `async/await` in test functions and `mockResolvedValue()`/`mockRejectedValue()` for promises.

**Basic Async Testing:**

```typescript
describe('Async methods', () => {
  let service: UsersService;
  let repository: jest.Mocked<Repository<User>>;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User),
          useValue: {
            findOne: jest.fn(),
            save: jest.fn(),
          },
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
    repository = module.get(getRepositoryToken(User));
  });

  // ✅ Using async/await
  it('should handle async operations with await', async () => {
    const user = { id: 1, email: 'test@example.com' };
    repository.findOne.mockResolvedValue(user as User);

    const result = await service.findOne(1);

    expect(result).toEqual(user);
  });

  // ✅ Using .resolves matcher
  it('should handle async operations with resolves', () => {
    const user = { id: 1, email: 'test@example.com' };
    repository.findOne.mockResolvedValue(user as User);

    return expect(service.findOne(1)).resolves.toEqual(user);
  });

  // ✅ Using done callback (older style)
  it('should handle async operations with done', (done) => {
    const user = { id: 1, email: 'test@example.com' };
    repository.findOne.mockResolvedValue(user as User);

    service.findOne(1).then((result) => {
      expect(result).toEqual(user);
      done();
    });
  });
});
```

**Testing Multiple Async Calls:**

```typescript
describe('Multiple async operations', () => {
  it('should handle sequential async calls', async () => {
    repository.findOne.mockResolvedValue({ id: 1, email: 'test@example.com' } as User);
    repository.save.mockResolvedValue({ id: 1, email: 'updated@example.com' } as User);

    const user = await service.findOne(1);
    const updated = await service.update(1, { email: 'updated@example.com' });

    expect(user.email).toBe('test@example.com');
    expect(updated.email).toBe('updated@example.com');
  });

  it('should handle parallel async calls', async () => {
    repository.findOne
      .mockResolvedValueOnce({ id: 1 } as User)
      .mockResolvedValueOnce({ id: 2 } as User);

    const [user1, user2] = await Promise.all([
      service.findOne(1),
      service.findOne(2),
    ]);

    expect(user1.id).toBe(1);
    expect(user2.id).toBe(2);
  });
});
```

**Testing Async Errors:**

```typescript
describe('Async error handling', () => {
  // ✅ Using await + try/catch
  it('should catch errors with try/catch', async () => {
    repository.findOne.mockRejectedValue(new Error('Database error'));

    try {
      await service.findOne(1);
      fail('Should have thrown error');
    } catch (error) {
      expect(error.message).toBe('Database error');
    }
  });

  // ✅ Using expect().rejects
  it('should catch errors with rejects', async () => {
    repository.findOne.mockRejectedValue(new NotFoundException('User not found'));

    await expect(service.findOne(999)).rejects.toThrow(NotFoundException);
    await expect(service.findOne(999)).rejects.toThrow('User not found');
  });

  // ✅ Testing specific error types
  it('should throw specific error type', async () => {
    repository.save.mockRejectedValue(new BadRequestException('Invalid data'));

    await expect(service.create({ email: 'invalid' })).rejects.toBeInstanceOf(
      BadRequestException,
    );
  });
});
```

**Testing Timeouts:**

```typescript
describe('Async timeouts', () => {
  it('should timeout after 5 seconds', async () => {
    jest.setTimeout(6000); // Increase Jest timeout

    repository.findOne.mockImplementation(
      () => new Promise((resolve) => setTimeout(() => resolve({} as User), 5000)),
    );

    const start = Date.now();
    await service.findOne(1);
    const duration = Date.now() - start;

    expect(duration).toBeGreaterThanOrEqual(5000);
  });

  it('should handle timeout errors', async () => {
    repository.findOne.mockImplementation(
      () =>
        new Promise((_, reject) =>
          setTimeout(() => reject(new Error('Timeout')), 1000),
        ),
    );

    await expect(service.findOne(1)).rejects.toThrow('Timeout');
  });
});
```

**Testing with Fake Timers:**

```typescript
describe('Async with fake timers', () => {
  beforeEach(() => {
    jest.useFakeTimers();
  });

  afterEach(() => {
    jest.useRealTimers();
  });

  it('should resolve after delay', async () => {
    const promise = service.scheduleTask();

    // Fast-forward time
    jest.advanceTimersByTime(5000);

    await expect(promise).resolves.toBe('completed');
  });

  it('should handle setInterval', async () => {
    const callback = jest.fn();
    service.startPolling(callback);

    // Run timers
    jest.advanceTimersByTime(10000);

    expect(callback).toHaveBeenCalledTimes(10); // Called every 1 second
  });
});
```

**Testing Observables (RxJS):**

```typescript
import { of, throwError } from 'rxjs';
import { HttpService } from '@nestjs/axios';

describe('Async observables', () => {
  let service: GithubService;
  let httpService: jest.Mocked<HttpService>;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        GithubService,
        {
          provide: HttpService,
          useValue: { get: jest.fn() },
        },
      ],
    }).compile();

    service = module.get<GithubService>(GithubService);
    httpService = module.get(HttpService);
  });

  it('should handle observable success', async () => {
    const data = { login: 'testuser' };
    httpService.get.mockReturnValue(of({ data } as any));

    const result = await service.getUser('testuser');

    expect(result).toEqual(data);
  });

  it('should handle observable error', async () => {
    httpService.get.mockReturnValue(
      throwError(() => new Error('API error')),
    );

    await expect(service.getUser('invalid')).rejects.toThrow('API error');
  });
});
```

**Interview Tip**: Use **`async/await`** in test functions for async operations. Use **`mockResolvedValue()`** for successful promises, **`mockRejectedValue()`** for errors. Test errors with **`expect().rejects.toThrow()`**. Use **`jest.useFakeTimers()`** for testing delays/intervals. For **Observables**, use `of()` for success and `throwError()` for errors.

</details>

### 17. How do you test error handling in services?

<details>
<summary>Answer</summary>

**Test error handling** by mocking failures and verifying correct exceptions are thrown with proper messages.

**Testing NestJS Exceptions:**

```typescript
import {
  NotFoundException,
  BadRequestException,
  UnauthorizedException,
  ForbiddenException,
  ConflictException,
  InternalServerErrorException,
} from '@nestjs/common';

describe('Error handling in UsersService', () => {
  let service: UsersService;
  let repository: jest.Mocked<Repository<User>>;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User),
          useValue: {
            findOne: jest.fn(),
            save: jest.fn(),
            delete: jest.fn(),
          },
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
    repository = module.get(getRepositoryToken(User));
  });

  // Test NotFoundException
  describe('findOne error handling', () => {
    it('should throw NotFoundException when user not found', async () => {
      repository.findOne.mockResolvedValue(null);

      await expect(service.findOne(999)).rejects.toThrow(NotFoundException);
      await expect(service.findOne(999)).rejects.toThrow('User with ID 999 not found');
    });

    it('should check exception status and message', async () => {
      repository.findOne.mockResolvedValue(null);

      try {
        await service.findOne(999);
      } catch (error) {
        expect(error).toBeInstanceOf(NotFoundException);
        expect(error.message).toBe('User with ID 999 not found');
        expect(error.getStatus()).toBe(404);
      }
    });
  });

  // Test BadRequestException
  describe('create error handling', () => {
    it('should throw BadRequestException for invalid email', async () => {
      const invalidDto = { email: 'invalid-email', password: 'pass123' };

      await expect(service.create(invalidDto)).rejects.toThrow(BadRequestException);
      await expect(service.create(invalidDto)).rejects.toThrow('Invalid email format');
    });

    it('should throw ConflictException for duplicate email', async () => {
      const createDto = { email: 'existing@example.com', password: 'pass123' };
      repository.findOne.mockResolvedValue({ id: 1 } as User);

      await expect(service.create(createDto)).rejects.toThrow(ConflictException);
      await expect(service.create(createDto)).rejects.toThrow('Email already exists');
    });
  });

  // Test UnauthorizedException
  describe('validateUser error handling', () => {
    it('should throw UnauthorizedException for invalid credentials', async () => {
      repository.findOne.mockResolvedValue(null);

      await expect(
        service.validateUser('test@example.com', 'wrong'),
      ).rejects.toThrow(UnauthorizedException);
    });
  });

  // Test database errors
  describe('database error handling', () => {
    it('should handle database connection errors', async () => {
      repository.findOne.mockRejectedValue(new Error('Connection timeout'));

      await expect(service.findOne(1)).rejects.toThrow('Connection timeout');
    });

    it('should wrap database errors in InternalServerErrorException', async () => {
      repository.save.mockRejectedValue(new Error('Database constraint violation'));

      await expect(service.create({ email: 'test@example.com' })).rejects.toThrow(
        InternalServerErrorException,
      );
    });
  });
});
```

**Testing Custom Exceptions:**

```typescript
// custom-exceptions.ts
export class UserNotFoundException extends NotFoundException {
  constructor(userId: number) {
    super(`User with ID ${userId} not found`);
  }
}

export class InvalidPasswordException extends BadRequestException {
  constructor() {
    super('Password must be at least 8 characters');
  }
}

// users.service.spec.ts
describe('Custom exceptions', () => {
  it('should throw custom NotFoundException', async () => {
    repository.findOne.mockResolvedValue(null);

    await expect(service.findOne(999)).rejects.toThrow(UserNotFoundException);
    await expect(service.findOne(999)).rejects.toThrow('User with ID 999 not found');
  });

  it('should throw custom BadRequestException', async () => {
    await expect(
      service.create({ email: 'test@example.com', password: '123' }),
    ).rejects.toThrow(InvalidPasswordException);
  });
});
```

**Testing Error Recovery:**

```typescript
describe('Error recovery', () => {
  it('should retry on transient errors', async () => {
    repository.findOne
      .mockRejectedValueOnce(new Error('Timeout'))
      .mockRejectedValueOnce(new Error('Timeout'))
      .mockResolvedValueOnce({ id: 1 } as User);

    const result = await service.findOneWithRetry(1, 3);

    expect(repository.findOne).toHaveBeenCalledTimes(3);
    expect(result).toEqual({ id: 1 });
  });

  it('should fail after max retries', async () => {
    repository.findOne.mockRejectedValue(new Error('Permanent failure'));

    await expect(service.findOneWithRetry(1, 3)).rejects.toThrow('Permanent failure');
    expect(repository.findOne).toHaveBeenCalledTimes(3);
  });

  it('should fallback to cache on database error', async () => {
    repository.findOne.mockRejectedValue(new Error('Database down'));
    cacheService.get.mockResolvedValue({ id: 1, email: 'cached@example.com' });

    const result = await service.findOneWithCache(1);

    expect(result.email).toBe('cached@example.com');
    expect(cacheService.get).toHaveBeenCalledWith('user:1');
  });
});
```

**Testing Validation Errors:**

```typescript
describe('Validation errors', () => {
  it('should throw error for missing required fields', async () => {
    await expect(service.create({} as any)).rejects.toThrow(
      'Email and password are required',
    );
  });

  it('should throw error for invalid data types', async () => {
    await expect(
      service.update(1, { age: 'invalid' } as any),
    ).rejects.toThrow('Age must be a number');
  });

  it('should validate multiple fields', async () => {
    const invalidDto = {
      email: 'invalid',
      password: '123',
      age: -5,
    };

    try {
      await service.create(invalidDto);
    } catch (error) {
      expect(error).toBeInstanceOf(BadRequestException);
      expect(error.message).toContain('email');
      expect(error.message).toContain('password');
      expect(error.message).toContain('age');
    }
  });
});
```

**Testing Error Context:**

```typescript
describe('Error context', () => {
  it('should include context in error', async () => {
    repository.findOne.mockResolvedValue(null);

    try {
      await service.findOne(999);
    } catch (error) {
      expect(error.getResponse()).toEqual({
        statusCode: 404,
        message: 'User with ID 999 not found',
        error: 'Not Found',
      });
    }
  });

  it('should log errors', async () => {
    const loggerSpy = jest.spyOn(service['logger'], 'error');
    repository.save.mockRejectedValue(new Error('Database error'));

    try {
      await service.create({ email: 'test@example.com' });
    } catch (error) {
      expect(loggerSpy).toHaveBeenCalledWith(
        'Failed to create user',
        error.stack,
      );
    }
  });
});
```

**Interview Tip**: Use **`expect().rejects.toThrow()`** to test exceptions. Verify **exception type** with `toBeInstanceOf()` and **message** with `toThrow('message')`. Test **all error scenarios**: not found, validation, conflicts, authorization. Test **error recovery**: retries, fallbacks, graceful degradation. Use **try/catch** to inspect exception details like `getStatus()` and `getResponse()`.

</details>

## Mocking

### 18. What is `@nestjs/testing` package?

<details>
<summary>Answer</summary>

**`@nestjs/testing`** is NestJS's official testing package that provides utilities for creating isolated testing modules and mocking dependencies.

**Key Features:**

| Feature | Description |
|---------|-------------|
| **Test.createTestingModule()** | Creates isolated DI container |
| **TestingModule** | Testing module interface |
| **overrideProvider()** | Replace providers |
| **overrideGuard()** | Replace guards |
| **overrideInterceptor()** | Replace interceptors |
| **overridePipe()** | Replace pipes |
| **compile()** | Finalize module |
| **module.get()** | Retrieve instances |
| **module.close()** | Clean up resources |

**Installation:**

```bash
npm install --save-dev @nestjs/testing

# Usually installed with
npm install --save-dev @nestjs/testing jest @types/jest ts-jest
```

**Basic Usage:**

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';

describe('UsersController', () => {
  let controller: UsersController;
  let service: UsersService;
  let module: TestingModule;

  beforeEach(async () => {
    // Create testing module
    module = await Test.createTestingModule({
      controllers: [UsersController],
      providers: [UsersService],
    }).compile();

    // Get instances
    controller = module.get<UsersController>(UsersController);
    service = module.get<UsersService>(UsersService);
  });

  afterEach(async () => {
    // Clean up
    await module.close();
  });

  it('should be defined', () => {
    expect(controller).toBeDefined();
    expect(service).toBeDefined();
  });
});
```

**Creating Testing Module with Imports:**

```typescript
import { TypeOrmModule } from '@nestjs/typeorm';
import { User } from './entities/user.entity';

describe('Integration test', () => {
  let module: TestingModule;

  beforeAll(async () => {
    module = await Test.createTestingModule({
      imports: [
        // Import modules
        TypeOrmModule.forRoot({
          type: 'sqlite',
          database: ':memory:',
          entities: [User],
          synchronize: true,
        }),
        TypeOrmModule.forFeature([User]),
      ],
      controllers: [UsersController],
      providers: [UsersService],
    }).compile();
  });

  afterAll(async () => {
    await module.close();
  });
});
```

**Using overrideProvider():**

```typescript
describe('UsersService with overrides', () => {
  let service: UsersService;
  let mockRepository: any;

  beforeEach(async () => {
    mockRepository = {
      find: jest.fn(),
      findOne: jest.fn(),
    };

    const module = await Test.createTestingModule({
      imports: [UsersModule], // Import real module
    })
      // Override specific provider
      .overrideProvider(getRepositoryToken(User))
      .useValue(mockRepository)
      .compile();

    service = module.get<UsersService>(UsersService);
  });

  it('should use mocked repository', async () => {
    mockRepository.find.mockResolvedValue([]);
    const result = await service.findAll();
    expect(result).toEqual([]);
  });
});
```

**Using overrideGuard():**

```typescript
import { JwtAuthGuard } from './guards/jwt-auth.guard';

describe('Protected endpoint', () => {
  beforeEach(async () => {
    const module = await Test.createTestingModule({
      controllers: [UsersController],
      providers: [UsersService],
    })
      // Override guard to always allow
      .overrideGuard(JwtAuthGuard)
      .useValue({ canActivate: () => true })
      .compile();

    controller = module.get<UsersController>(UsersController);
  });

  it('should bypass authentication', async () => {
    const result = await controller.getProfile();
    expect(result).toBeDefined();
  });
});
```

**Using overrideInterceptor():**

```typescript
import { CacheInterceptor } from '@nestjs/cache-manager';

describe('UsersController without caching', () => {
  beforeEach(async () => {
    const module = await Test.createTestingModule({
      controllers: [UsersController],
      providers: [UsersService],
    })
      .overrideInterceptor(CacheInterceptor)
      .useValue({ intercept: (context, next) => next.handle() })
      .compile();

    controller = module.get<UsersController>(UsersController);
  });
});
```

**Using overridePipe():**

```typescript
import { ValidationPipe } from '@nestjs/common';

describe('UsersController without validation', () => {
  beforeEach(async () => {
    const module = await Test.createTestingModule({
      controllers: [UsersController],
      providers: [UsersService],
    })
      .overridePipe(ValidationPipe)
      .useValue({ transform: (value) => value })
      .compile();

    controller = module.get<UsersController>(UsersController);
  });
});
```

**Creating Full Application:**

```typescript
import { INestApplication } from '@nestjs/common';
import * as request from 'supertest';

describe('E2E test', () => {
  let app: INestApplication;

  beforeAll(async () => {
    const module = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    // Create application instance
    app = module.createNestApplication();
    
    // Apply global middleware, pipes, etc.
    app.useGlobalPipes(new ValidationPipe());
    
    await app.init();
  });

  afterAll(async () => {
    await app.close();
  });

  it('/users (GET)', () => {
    return request(app.getHttpServer())
      .get('/users')
      .expect(200);
  });
});
```

**Using module.select():**

```typescript
describe('Accessing scoped providers', () => {
  it('should access provider from imported module', async () => {
    const module = await Test.createTestingModule({
      imports: [UsersModule, AuthModule],
    }).compile();

    // Access from specific module
    const authService = module.select(AuthModule).get(AuthService);
    
    expect(authService).toBeDefined();
  });
});
```

**Interview Tip**: **`@nestjs/testing`** provides **`Test.createTestingModule()`** to create isolated testing modules with DI. Use **`overrideProvider()`** to mock dependencies, **`overrideGuard()`** for guards, **`overrideInterceptor()`** for interceptors. Call **`compile()`** to finalize, **`module.get()`** to retrieve instances, **`module.close()`** to clean up. For E2E tests, use **`createNestApplication()`** to create full app instance.

</details>
### 19. How do you create mock providers?

<details>
<summary>Answer</summary>

**Create mock providers** using objects with jest.fn() methods or mock classes, then provide them using `useValue`, `useClass`, or `useFactory`.

**Pattern 1: Mock Object with jest.fn():**

```typescript
describe('Mock providers with useValue', () => {
  let service: UsersService;
  let mockRepository: any;
  let mockMailService: any;

  beforeEach(async () => {
    // Create mock objects
    mockRepository = {
      find: jest.fn(),
      findOne: jest.fn(),
      save: jest.fn(),
      create: jest.fn(),
      update: jest.fn(),
      remove: jest.fn(),
    };

    mockMailService = {
      sendEmail: jest.fn(),
      sendWelcomeEmail: jest.fn(),
    };

    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User),
          useValue: mockRepository,
        },
        {
          provide: MailService,
          useValue: mockMailService,
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
  });

  it('should use mock repository', async () => {
    mockRepository.find.mockResolvedValue([]);
    const result = await service.findAll();
    expect(result).toEqual([]);
  });
});
```

**Pattern 2: Mock Class:**

```typescript
// Create reusable mock class
class MockUsersRepository {
  find = jest.fn();
  findOne = jest.fn();
  save = jest.fn();
  create = jest.fn((dto) => ({ id: 1, ...dto }));
  update = jest.fn();
  remove = jest.fn();
}

class MockMailService {
  sendEmail = jest.fn().mockResolvedValue(true);
  sendWelcomeEmail = jest.fn().mockResolvedValue(true);
}

describe('Mock providers with useClass', () => {
  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User),
          useClass: MockUsersRepository,
        },
        {
          provide: MailService,
          useClass: MockMailService,
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
  });
});
```

**Pattern 3: Factory Function:**

```typescript
// Create mock factory
function createMockRepository() {
  return {
    find: jest.fn(),
    findOne: jest.fn(),
    save: jest.fn(),
  };
}

function createMockMailService() {
  return {
    sendEmail: jest.fn().mockResolvedValue(true),
  };
}

describe('Mock providers with factories', () => {
  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User),
          useValue: createMockRepository(),
        },
        {
          provide: MailService,
          useValue: createMockMailService(),
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
  });
});
```

**Pattern 4: TypeScript Typed Mocks:**

```typescript
type MockType<T> = {
  [P in keyof T]?: jest.Mock<any>;
};

const repositoryMockFactory: () => MockType<Repository<User>> = jest.fn(() => ({
  find: jest.fn(),
  findOne: jest.fn(),
  save: jest.fn(),
  create: jest.fn(),
}));

describe('Typed mock providers', () => {
  let repository: MockType<Repository<User>>;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User),
          useFactory: repositoryMockFactory,
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
    repository = module.get(getRepositoryToken(User));
  });

  it('should have type-safe mocks', async () => {
    repository.find.mockResolvedValue([]);
    const result = await service.findAll();
    expect(result).toEqual([]);
  });
});
```

**Pattern 5: Partial Mocks:**

```typescript
describe('Partial mock providers', () => {
  beforeEach(async () => {
    // Only mock methods you need
    const partialMock = {
      findOne: jest.fn(),
      save: jest.fn(),
      // Other methods undefined
    };

    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User),
          useValue: partialMock,
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
  });
});
```

**Interview Tip**: Create mock providers using **mock objects** (most common), **mock classes**, or **factory functions**. Use **`useValue`** for object mocks, **`useClass`** for class mocks, **`useFactory`** for dynamic creation. Type mocks with **`jest.Mocked<T>`** or **`MockType<T>`** for TypeScript support. Store mock factories in separate files for reusability.

</details>

### 20. How do you override providers in tests using `overrideProvider()`?

<details>
<summary>Answer</summary>

**`overrideProvider()`** replaces providers from imported modules without recreating the entire testing module.

**Basic Usage:**

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { UsersModule } from './users.module';
import { UsersService } from './users.service';
import { MailService } from '../mail/mail.service';

describe('Override provider', () => {
  let service: UsersService;
  let mockMailService: any;

  beforeEach(async () => {
    mockMailService = {
      sendEmail: jest.fn(),
      sendWelcomeEmail: jest.fn(),
    };

    const module = await Test.createTestingModule({
      imports: [UsersModule], // Import real module with all dependencies
    })
      .overrideProvider(MailService) // Override specific provider
      .useValue(mockMailService)
      .compile();

    service = module.get<UsersService>(UsersService);
  });

  it('should use overridden mail service', async () => {
    mockMailService.sendWelcomeEmail.mockResolvedValue(true);
    
    await service.sendWelcome('test@example.com');
    
    expect(mockMailService.sendWelcomeEmail).toHaveBeenCalled();
  });
});
```

**Overriding Multiple Providers:**

```typescript
describe('Override multiple providers', () => {
  let mockMailService: any;
  let mockCacheService: any;
  let mockLoggerService: any;

  beforeEach(async () => {
    mockMailService = { send: jest.fn() };
    mockCacheService = { get: jest.fn(), set: jest.fn() };
    mockLoggerService = { log: jest.fn(), error: jest.fn() };

    const module = await Test.createTestingModule({
      imports: [UsersModule],
    })
      .overrideProvider(MailService)
      .useValue(mockMailService)
      .overrideProvider(CacheService)
      .useValue(mockCacheService)
      .overrideProvider(Logger)
      .useValue(mockLoggerService)
      .compile();

    service = module.get<UsersService>(UsersService);
  });
});
```

**Override with useClass:**

```typescript
class MockMailService {
  sendEmail = jest.fn().mockResolvedValue(true);
}

describe('Override with useClass', () => {
  beforeEach(async () => {
    const module = await Test.createTestingModule({
      imports: [UsersModule],
    })
      .overrideProvider(MailService)
      .useClass(MockMailService)
      .compile();

    service = module.get<UsersService>(UsersService);
  });
});
```

**Override with useFactory:**

```typescript
describe('Override with useFactory', () => {
  beforeEach(async () => {
    const module = await Test.createTestingModule({
      imports: [UsersModule],
    })
      .overrideProvider(ConfigService)
      .useFactory({
        factory: () => ({
          get: jest.fn((key: string) => {
            const config = {
              DATABASE_URL: 'test-db',
              JWT_SECRET: 'test-secret',
            };
            return config[key];
          }),
        }),
      })
      .compile();

    service = module.get<UsersService>(UsersService);
  });
});
```

**Override TypeORM Repository:**

```typescript
import { getRepositoryToken } from '@nestjs/typeorm';

describe('Override TypeORM repository', () => {
  let mockRepository: any;

  beforeEach(async () => {
    mockRepository = {
      find: jest.fn(),
      findOne: jest.fn(),
      save: jest.fn(),
    };

    const module = await Test.createTestingModule({
      imports: [UsersModule],
    })
      .overrideProvider(getRepositoryToken(User))
      .useValue(mockRepository)
      .compile();

    service = module.get<UsersService>(UsersService);
  });
});
```

**Override Custom Injection Token:**

```typescript
describe('Override custom token', () => {
  beforeEach(async () => {
    const module = await Test.createTestingModule({
      imports: [UsersModule],
    })
      .overrideProvider('DATABASE_CONNECTION')
      .useValue({ query: jest.fn() })
      .overrideProvider('CONFIG_OPTIONS')
      .useValue({ apiKey: 'test-key' })
      .compile();

    service = module.get<UsersService>(UsersService);
  });
});
```

**Override Guards, Interceptors, Pipes:**

```typescript
import { JwtAuthGuard } from './guards/jwt-auth.guard';
import { CacheInterceptor } from '@nestjs/cache-manager';
import { ValidationPipe } from '@nestjs/common';

describe('Override guards, interceptors, pipes', () => {
  beforeEach(async () => {
    const module = await Test.createTestingModule({
      imports: [UsersModule],
    })
      // Override guard
      .overrideGuard(JwtAuthGuard)
      .useValue({ canActivate: () => true })
      // Override interceptor
      .overrideInterceptor(CacheInterceptor)
      .useValue({ intercept: (context, next) => next.handle() })
      // Override pipe
      .overridePipe(ValidationPipe)
      .useValue({ transform: (value) => value })
      .compile();

    controller = module.get<UsersController>(UsersController);
  });
});
```

**Conditional Override:**

```typescript
describe('Conditional override', () => {
  it('should use real service in production mode', async () => {
    const module = await Test.createTestingModule({
      imports: [UsersModule],
    });

    // Only override if condition is met
    if (process.env.USE_MOCK_MAIL === 'true') {
      module.overrideProvider(MailService).useValue(mockMailService);
    }

    await module.compile();
  });
});
```

**Interview Tip**: Use **`overrideProvider()`** to replace specific providers from imported modules. Chain multiple overrides for multiple providers. Use **`useValue`** for mock objects, **`useClass`** for mock classes, **`useFactory`** for dynamic mocks. Also supports **`overrideGuard()`**, **`overrideInterceptor()`**, and **`overridePipe()`**. More flexible than manually providing all dependencies.

</details>

### 21. How do you mock database repositories?

<details>
<summary>Answer</summary>

**Mock database repositories** by creating mock objects with repository methods and providing them using `getRepositoryToken()` or `getModelToken()`.

**Mocking TypeORM Repository:**

```typescript
import { getRepositoryToken } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './entities/user.entity';

describe('Mock TypeORM Repository', () => {
  let service: UsersService;
  let repository: jest.Mocked<Repository<User>>;

  // Comprehensive repository mock
  const mockRepository = {
    // Find methods
    find: jest.fn(),
    findOne: jest.fn(),
    findOneBy: jest.fn(),
    findBy: jest.fn(),
    findAndCount: jest.fn(),
    findOneOrFail: jest.fn(),
    
    // Create/Save methods
    create: jest.fn(),
    save: jest.fn(),
    insert: jest.fn(),
    
    // Update methods
    update: jest.fn(),
    
    // Delete methods
    delete: jest.fn(),
    remove: jest.fn(),
    softDelete: jest.fn(),
    restore: jest.fn(),
    
    // Count methods
    count: jest.fn(),
    countBy: jest.fn(),
    
    // Query builder
    createQueryBuilder: jest.fn(() => ({
      where: jest.fn().mockReturnThis(),
      andWhere: jest.fn().mockReturnThis(),
      orWhere: jest.fn().mockReturnThis(),
      leftJoin: jest.fn().mockReturnThis(),
      leftJoinAndSelect: jest.fn().mockReturnThis(),
      innerJoin: jest.fn().mockReturnThis(),
      orderBy: jest.fn().mockReturnThis(),
      skip: jest.fn().mockReturnThis(),
      take: jest.fn().mockReturnThis(),
      getOne: jest.fn(),
      getMany: jest.fn(),
      getManyAndCount: jest.fn(),
      getRawOne: jest.fn(),
      getRawMany: jest.fn(),
    })),
    
    // Manager and metadata
    manager: {
      transaction: jest.fn(),
      save: jest.fn(),
    },
    metadata: {
      columns: [],
      relations: [],
    },
  };

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User),
          useValue: mockRepository,
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
    repository = module.get(getRepositoryToken(User));

    jest.clearAllMocks();
  });

  describe('CRUD operations', () => {
    it('should find all users', async () => {
      const users = [{ id: 1, email: 'test@example.com' }];
      mockRepository.find.mockResolvedValue(users as User[]);

      const result = await service.findAll();

      expect(result).toEqual(users);
      expect(mockRepository.find).toHaveBeenCalled();
    });

    it('should find user by id', async () => {
      const user = { id: 1, email: 'test@example.com' };
      mockRepository.findOne.mockResolvedValue(user as User);

      const result = await service.findOne(1);

      expect(mockRepository.findOne).toHaveBeenCalledWith({ where: { id: 1 } });
      expect(result).toEqual(user);
    });

    it('should create user', async () => {
      const createDto = { email: 'new@example.com', password: 'pass123' };
      const user = { id: 1, ...createDto };
      
      mockRepository.create.mockReturnValue(user as User);
      mockRepository.save.mockResolvedValue(user as User);

      const result = await service.create(createDto);

      expect(mockRepository.create).toHaveBeenCalledWith(createDto);
      expect(mockRepository.save).toHaveBeenCalledWith(user);
      expect(result).toEqual(user);
    });

    it('should update user', async () => {
      mockRepository.update.mockResolvedValue({ affected: 1 } as any);

      await service.update(1, { name: 'Updated' });

      expect(mockRepository.update).toHaveBeenCalledWith(1, { name: 'Updated' });
    });

    it('should delete user', async () => {
      mockRepository.delete.mockResolvedValue({ affected: 1 } as any);

      await service.remove(1);

      expect(mockRepository.delete).toHaveBeenCalledWith(1);
    });
  });

  describe('Query builder', () => {
    it('should use query builder', async () => {
      const users = [{ id: 1, email: 'test@example.com' }];
      const queryBuilder = mockRepository.createQueryBuilder();
      (queryBuilder.getMany as jest.Mock).mockResolvedValue(users);

      const result = await service.searchUsers('test');

      expect(mockRepository.createQueryBuilder).toHaveBeenCalled();
      expect(queryBuilder.where).toHaveBeenCalled();
      expect(result).toEqual(users);
    });
  });
});
```

**Mocking Mongoose Model:**

```typescript
import { getModelToken } from '@nestjs/mongoose';
import { Model } from 'mongoose';
import { User } from './schemas/user.schema';

describe('Mock Mongoose Model', () => {
  let service: UsersService;
  let model: jest.Mocked<Model<User>>;

  const mockModel = {
    find: jest.fn(),
    findById: jest.fn(),
    findOne: jest.fn(),
    findByIdAndUpdate: jest.fn(),
    findByIdAndDelete: jest.fn(),
    create: jest.fn(),
    deleteMany: jest.fn(),
    updateMany: jest.fn(),
    countDocuments: jest.fn(),
    // Constructor for new Model()
    constructor: jest.fn(),
  };

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getModelToken(User.name),
          useValue: mockModel,
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
    model = module.get(getModelToken(User.name));
  });

  it('should find all users', async () => {
    const users = [{ _id: '1', email: 'test@example.com' }];
    (mockModel.find as jest.Mock).mockReturnValue({
      exec: jest.fn().mockResolvedValue(users),
    });

    const result = await service.findAll();

    expect(mockModel.find).toHaveBeenCalled();
    expect(result).toEqual(users);
  });

  it('should create user', async () => {
    const createDto = { email: 'new@example.com' };
    const user = { _id: '1', ...createDto };
    (mockModel.create as jest.Mock).mockResolvedValue(user);

    const result = await service.create(createDto);

    expect(mockModel.create).toHaveBeenCalledWith(createDto);
    expect(result).toEqual(user);
  });
});
```

**Mocking with Relations:**

```typescript
describe('Mock repository with relations', () => {
  it('should find user with posts', async () => {
    const user = {
      id: 1,
      email: 'test@example.com',
      posts: [{ id: 1, title: 'Post 1' }],
    };
    mockRepository.findOne.mockResolvedValue(user as User);

    const result = await service.findOneWithPosts(1);

    expect(mockRepository.findOne).toHaveBeenCalledWith({
      where: { id: 1 },
      relations: ['posts'],
    });
    expect(result.posts).toHaveLength(1);
  });
});
```

**Interview Tip**: For **TypeORM**, use `getRepositoryToken(Entity)` to get injection token. For **Mongoose**, use `getModelToken(Model.name)`. Mock **all methods** your code uses. Mock **query builder** with chainable methods using `mockReturnThis()`. For **async operations**, use `mockResolvedValue()`. Create **reusable mock factories** to avoid duplication.

</details>

### 22. How do you use `useValue`, `useClass`, `useFactory` in mocks?

<details>
<summary>Answer</summary>

**`useValue`, `useClass`, `useFactory`** are three ways to provide mock implementations in testing modules.

**1. useValue - Most Common:**

```typescript
// Provide mock object directly
describe('useValue - mock objects', () => {
  beforeEach(async () => {
    const mockRepository = {
      find: jest.fn().mockResolvedValue([]),
      findOne: jest.fn().mockResolvedValue({ id: 1 }),
      save: jest.fn(),
    };

    const mockConfigService = {
      get: jest.fn((key: string) => {
        const config = { DB_HOST: 'localhost', PORT: 3000 };
        return config[key];
      }),
    };

    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User),
          useValue: mockRepository, // Provide object
        },
        {
          provide: ConfigService,
          useValue: mockConfigService, // Provide object
        },
      ],
    }).compile();
  });

  // Best for: Simple mocks, mock objects with jest.fn()
  // Pros: Simple, direct, full control
  // Cons: Verbose for complex mocks
});
```

**2. useClass - Mock Class:**

```typescript
// Create mock class
class MockUsersRepository {
  find = jest.fn().mockResolvedValue([]);
  findOne = jest.fn().mockResolvedValue({ id: 1 });
  save = jest.fn();
}

class MockMailService {
  constructor() {
    console.log('MockMailService instantiated');
  }
  
  sendEmail = jest.fn().mockResolvedValue(true);
  sendWelcomeEmail = jest.fn().mockResolvedValue(true);
}

describe('useClass - mock classes', () => {
  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User),
          useClass: MockUsersRepository, // Provide class
        },
        {
          provide: MailService,
          useClass: MockMailService, // Provide class
        },
      ],
    }).compile();
  });

  // Best for: Reusable mocks, complex mock logic
  // Pros: Reusable, organized, can have constructor logic
  // Cons: More boilerplate
});
```

**3. useFactory - Dynamic Creation:**

```typescript
describe('useFactory - dynamic mocks', () => {
  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User),
          useFactory: () => ({
            find: jest.fn().mockResolvedValue([]),
            findOne: jest.fn().mockResolvedValue({ id: 1 }),
          }),
        },
        {
          provide: ConfigService,
          useFactory: () => {
            // Dynamic logic
            const env = process.env.NODE_ENV || 'test';
            return {
              get: jest.fn((key: string) => `${env}-${key}`),
            };
          },
        },
      ],
    }).compile();
  });

  // Best for: Dynamic mocks, conditional logic
  // Pros: Flexible, can inject dependencies
  // Cons: Can be complex
});
```

**useFactory with Injected Dependencies:**

```typescript
describe('useFactory with injection', () => {
  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: 'LOGGER',
          useValue: { log: jest.fn() },
        },
        {
          provide: CacheService,
          useFactory: (logger) => {
            // Inject logger into factory
            return {
              get: jest.fn(),
              set: jest.fn((key, value) => {
                logger.log(`Caching ${key}`);
                return Promise.resolve();
              }),
            };
          },
          inject: ['LOGGER'], // Inject dependencies
        },
      ],
    }).compile();
  });
});
```

**Comparison Table:**

| Method | Use Case | Pros | Cons |
|--------|----------|------|------|
| **useValue** | Simple mocks | Direct, simple | Verbose for complex mocks |
| **useClass** | Reusable mocks | Organized, reusable | More boilerplate |
| **useFactory** | Dynamic mocks | Flexible, can inject deps | Can be complex |

**Real-World Example:**

```typescript
describe('Real-world mock providers', () => {
  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        OrdersService,
        
        // useValue for simple repository mock
        {
          provide: getRepositoryToken(Order),
          useValue: {
            find: jest.fn(),
            save: jest.fn(),
          },
        },
        
        // useClass for complex service mock
        {
          provide: PaymentService,
          useClass: MockPaymentService,
        },
        
        // useFactory for config-dependent mock
        {
          provide: EmailService,
          useFactory: () => {
            if (process.env.MOCK_EMAIL === 'true') {
              return { send: jest.fn() };
            }
            return new RealEmailService();
          },
        },
      ],
    }).compile();
  });
});
```

**When to Use Each:**

```typescript
// ✅ Use useValue for: Simple mocks
const mockRepo = { find: jest.fn(), save: jest.fn() };
{ provide: Repository, useValue: mockRepo }

// ✅ Use useClass for: Reusable, complex mocks
class MockMailService { /* ... */ }
{ provide: MailService, useClass: MockMailService }

// ✅ Use useFactory for: Dynamic, conditional mocks
{
  provide: ConfigService,
  useFactory: () => ({
    get: jest.fn((key) => process.env[key]),
  }),
}
```

**Interview Tip**: **`useValue`** is most common for simple mock objects. **`useClass`** for reusable mock classes with methods. **`useFactory`** for dynamic mocks that need conditional logic or dependency injection. Use **`useValue`** for 90% of cases, **`useClass`** for complex reusable mocks, **`useFactory`** when you need runtime logic.

</details>

## Integration Testing

### 23. What are integration tests?

<details>
<summary>Answer</summary>

**Integration tests** verify that **multiple components work together** correctly, testing interactions between services, databases, and modules.

**Unit vs Integration Tests:**

| Aspect | Unit Tests | Integration Tests |
|--------|-----------|------------------|
| **Scope** | Single component | Multiple components |
| **Dependencies** | All mocked | Real or minimal mocks |
| **Database** | Mocked | Real test DB |
| **Speed** | Very fast (1-10ms) | Slower (100-500ms) |
| **Purpose** | Test logic in isolation | Test component interaction |
| **Example** | Service with mock repo | Service + real database |

**Basic Integration Test:**

```typescript
// users.integration.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UsersService } from './users.service';
import { User } from './entities/user.entity';
import { Repository, DataSource } from 'typeorm';
import { getRepositoryToken } from '@nestjs/typeorm';

describe('UsersService Integration Tests', () => {
  let service: UsersService;
  let repository: Repository<User>;
  let dataSource: DataSource;
  let module: TestingModule;

  beforeAll(async () => {
    // Create testing module with REAL database
    module = await Test.createTestingModule({
      imports: [
        TypeOrmModule.forRoot({
          type: 'sqlite',
          database: ':memory:', // In-memory SQLite for testing
          entities: [User],
          synchronize: true, // Auto-create tables
          dropSchema: true, // Clean start
        }),
        TypeOrmModule.forFeature([User]),
      ],
      providers: [UsersService],
    }).compile();

    service = module.get<UsersService>(UsersService);
    repository = module.get<Repository<User>>(getRepositoryToken(User));
    dataSource = module.get<DataSource>(DataSource);
  });

  afterEach(async () => {
    // Clean up database after each test
    await repository.clear();
  });

  afterAll(async () => {
    // Close connections
    await dataSource.destroy();
    await module.close();
  });

  describe('CRUD operations', () => {
    it('should create and save user to database', async () => {
      const createDto = { email: 'test@example.com', password: 'pass123' };
      
      const created = await service.create(createDto);
      
      // Verify in database
      const found = await repository.findOne({ where: { id: created.id } });
      expect(found).toBeDefined();
      expect(found.email).toBe(createDto.email);
      expect(found.id).toBe(created.id);
    });

    it('should find all users from database', async () => {
      // Seed database
      await repository.save([{ email: 'user1@example.com' }, { email: 'user2@example.com' }]);
      
      const users = await service.findAll();
      
      expect(users).toHaveLength(2);
    });

    it('should update user in database', async () => {
      const user = await repository.save({ email: 'test@example.com' });
      
      await service.update(user.id, { name: 'Updated Name' });
      
      const updated = await repository.findOne({ where: { id: user.id } });
      expect(updated.name).toBe('Updated Name');
    });

    it('should delete user from database', async () => {
      const user = await repository.save({ email: 'test@example.com' });
      
      await service.remove(user.id);
      
      const found = await repository.findOne({ where: { id: user.id } });
      expect(found).toBeNull();
    });
  });

  describe('Database constraints', () => {
    it('should enforce unique email constraint', async () => {
      await repository.save({ email: 'duplicate@example.com' });
      
      await expect(
        service.create({ email: 'duplicate@example.com', password: 'pass' }),
      ).rejects.toThrow();
    });

    it('should enforce not null constraints', async () => {
      await expect(
        repository.save({ email: null } as any),
      ).rejects.toThrow();
    });
  });

  describe('Transactions', () => {
    it('should rollback on error', async () => {
      const initialCount = await repository.count();
      
      try {
        await service.createMultipleUsersInTransaction([
          { email: 'user1@example.com' },
          { email: 'invalid' }, // This will fail
        ]);
      } catch (error) {
        // Transaction rolled back
      }
      
      const finalCount = await repository.count();
      expect(finalCount).toBe(initialCount); // No users created
    });
  });

  describe('Relations', () => {
    it('should load user with posts', async () => {
      // Assuming User has posts relation
      const user = await repository.save({ email: 'test@example.com' });
      // Create related posts
      
      const userWithPosts = await service.findOneWithPosts(user.id);
      
      expect(userWithPosts.posts).toBeDefined();
    });
  });
});
```

**Testing Multiple Modules:**

```typescript
describe('Orders + Users Integration', () => {
  let ordersService: OrdersService;
  let usersService: UsersService;
  let module: TestingModule;

  beforeAll(async () => {
    module = await Test.createTestingModule({
      imports: [
        TypeOrmModule.forRoot({
          type: 'sqlite',
          database: ':memory:',
          entities: [User, Order],
          synchronize: true,
        }),
        UsersModule,
        OrdersModule,
      ],
    }).compile();

    ordersService = module.get<OrdersService>(OrdersService);
    usersService = module.get<UsersService>(UsersService);
  });

  it('should create order for existing user', async () => {
    const user = await usersService.create({ email: 'test@example.com' });
    
    const order = await ordersService.create({
      userId: user.id,
      items: [{ productId: 1, quantity: 2 }],
    });
    
    expect(order.userId).toBe(user.id);
  });
});
```

**Interview Tip**: Integration tests use **real databases** (in-memory or test DB), test **multiple components together**, and verify **actual interactions**. Slower than unit tests but provide more confidence. Use **SQLite in-memory** for fast, isolated tests. Clean database in **`afterEach()`**, close connections in **`afterAll()`**. Balance: 60% unit, 30% integration, 10% E2E.

</details>

### 24. How do you set up a test database for integration tests?

<details>
<summary>Answer</summary>

**Set up test database** using **SQLite in-memory** (fastest), **Docker containers**, or **separate test database**.

**Option 1: SQLite In-Memory (Recommended for Speed):**

```typescript
// users.integration.spec.ts
import { Test } from '@nestjs/testing';
import { TypeOrmModule } from '@nestjs/typeorm';
import { User } from './entities/user.entity';
import { Post } from '../posts/entities/post.entity';

describe('Integration with SQLite', () => {
  let module: TestingModule;
  let dataSource: DataSource;

  beforeAll(async () => {
    module = await Test.createTestingModule({
      imports: [
        TypeOrmModule.forRoot({
          type: 'sqlite',
          database: ':memory:', // In-memory database
          entities: [User, Post],
          synchronize: true, // Auto-create schema
          dropSchema: true, // Drop schema on start
          logging: false, // Disable logging for cleaner output
        }),
        TypeOrmModule.forFeature([User, Post]),
      ],
      providers: [UsersService],
    }).compile();

    dataSource = module.get<DataSource>(DataSource);
  });

  afterEach(async () => {
    // Clear all tables after each test
    const entities = dataSource.entityMetadatas;
    for (const entity of entities) {
      const repository = dataSource.getRepository(entity.name);
      await repository.clear();
    }
  });

  afterAll(async () => {
    await dataSource.destroy();
    await module.close();
  });
});
```

**Option 2: Separate Test Database:**

```typescript
// test/database.config.ts
export const testDatabaseConfig = {
  type: 'postgres' as const,
  host: 'localhost',
  port: 5433, // Different port from production
  username: 'test',
  password: 'test',
  database: 'test_db',
  entities: ['src/**/*.entity.ts'],
  synchronize: true,
  dropSchema: true, // WARNING: Drops all data on start
};

// users.integration.spec.ts
import { testDatabaseConfig } from '../test/database.config';

describe('Integration with Postgres', () => {
  beforeAll(async () => {
    module = await Test.createTestingModule({
      imports: [
        TypeOrmModule.forRoot(testDatabaseConfig),
        TypeOrmModule.forFeature([User]),
      ],
      providers: [UsersService],
    }).compile();
  });
});
```

**Option 3: Docker Test Container:**

```typescript
// test/test-database.setup.ts
import { GenericContainer, StartedTestContainer } from 'testcontainers';

let postgresContainer: StartedTestContainer;

export async function setupTestDatabase() {
  // Start PostgreSQL container
  postgresContainer = await new GenericContainer('postgres:15')
    .withEnvironment({
      POSTGRES_USER: 'test',
      POSTGRES_PASSWORD: 'test',
      POSTGRES_DB: 'test_db',
    })
    .withExposedPorts(5432)
    .start();

  const config = {
    type: 'postgres' as const,
    host: postgresContainer.getHost(),
    port: postgresContainer.getMappedPort(5432),
    username: 'test',
    password: 'test',
    database: 'test_db',
    synchronize: true,
  };

  return config;
}

export async function teardownTestDatabase() {
  await postgresContainer?.stop();
}

// users.integration.spec.ts
import { setupTestDatabase, teardownTestDatabase } from '../test/test-database.setup';

describe('Integration with Docker', () => {
  let dbConfig: any;

  beforeAll(async () => {
    dbConfig = await setupTestDatabase();
    
    module = await Test.createTestingModule({
      imports: [
        TypeOrmModule.forRoot(dbConfig),
        TypeOrmModule.forFeature([User]),
      ],
      providers: [UsersService],
    }).compile();
  });

  afterAll(async () => {
    await module.close();
    await teardownTestDatabase();
  });
});
```

**Option 4: Environment-Based Configuration:**

```typescript
// config/database.config.ts
export const getDatabaseConfig = () => {
  if (process.env.NODE_ENV === 'test') {
    return {
      type: 'sqlite' as const,
      database: ':memory:',
      entities: [__dirname + '/../**/*.entity{.ts,.js}'],
      synchronize: true,
    };
  }
  
  return {
    type: 'postgres' as const,
    host: process.env.DB_HOST,
    port: parseInt(process.env.DB_PORT),
    username: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_NAME,
  };
};

// users.integration.spec.ts
import { getDatabaseConfig } from '../config/database.config';

describe('Integration with env config', () => {
  beforeAll(async () => {
    module = await Test.createTestingModule({
      imports: [
        TypeOrmModule.forRoot(getDatabaseConfig()),
        TypeOrmModule.forFeature([User]),
      ],
      providers: [UsersService],
    }).compile();
  });
});
```

**Database Cleanup Strategies:**

```typescript
// Strategy 1: Clear all tables
aftereEach(async () => {
  const entities = dataSource.entityMetadatas;
  for (const entity of entities) {
    const repository = dataSource.getRepository(entity.name);
    await repository.clear();
  }
});

// Strategy 2: Drop and recreate schema
afterEach(async () => {
  await dataSource.dropDatabase();
  await dataSource.synchronize();
});

// Strategy 3: Transactions (rollback after each test)
beforeEach(async () => {
  await dataSource.transaction(async (manager) => {
    // Test runs in transaction
    // Automatically rolled back
  });
});

// Strategy 4: Truncate tables (faster than clear)
afterEach(async () => {
  const tables = dataSource.entityMetadatas.map((e) => e.tableName);
  for (const table of tables) {
    await dataSource.query(`TRUNCATE TABLE "${table}" CASCADE`);
  }
});
```

**Seeding Test Data:**

```typescript
// test/helpers/seed.ts
export async function seedDatabase(dataSource: DataSource) {
  const userRepository = dataSource.getRepository(User);
  
  await userRepository.save([
    { email: 'user1@example.com', role: 'admin' },
    { email: 'user2@example.com', role: 'user' },
  ]);
}

// users.integration.spec.ts
beforeEach(async () => {
  await seedDatabase(dataSource);
});
```

**Interview Tip**: Use **SQLite in-memory** for fastest tests (`:memory:`). Use **separate test database** for production-like testing. Use **Docker containers** for isolated, reproducible tests. Always **clean up** database in `afterEach()` and **close connections** in `afterAll()`. Set **`synchronize: true`** and **`dropSchema: true`** for clean test environments. **Never** run tests on production database.

</details>

### 25. How do you test multiple modules together?

<details>
<summary>Answer</summary>

**Test multiple modules** by importing them into the testing module and verifying their interactions with real or minimal mocks.

**Testing Two Modules (Users + Orders):**

```typescript
// users-orders.integration.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UsersModule } from '../users/users.module';
import { OrdersModule } from '../orders/orders.module';
import { User } from '../users/entities/user.entity';
import { Order } from '../orders/entities/order.entity';
import { UsersService } from '../users/users.service';
import { OrdersService } from '../orders/orders.service';
import { DataSource } from 'typeorm';

describe('Users + Orders Integration', () => {
  let module: TestingModule;
  let usersService: UsersService;
  let ordersService: OrdersService;
  let dataSource: DataSource;

  beforeAll(async () => {
    module = await Test.createTestingModule({
      imports: [
        // Database setup
        TypeOrmModule.forRoot({
          type: 'sqlite',
          database: ':memory:',
          entities: [User, Order],
          synchronize: true,
        }),
        // Import both modules
        UsersModule,
        OrdersModule,
      ],
    }).compile();

    usersService = module.get<UsersService>(UsersService);
    ordersService = module.get<OrdersService>(OrdersService);
    dataSource = module.get<DataSource>(DataSource);
  });

  afterEach(async () => {
    // Clean all tables
    const entities = dataSource.entityMetadatas;
    for (const entity of entities) {
      await dataSource.getRepository(entity.name).clear();
    }
  });

  afterAll(async () => {
    await dataSource.destroy();
    await module.close();
  });

  describe('User-Order relationship', () => {
    it('should create order for existing user', async () => {
      // Create user
      const user = await usersService.create({
        email: 'test@example.com',
        password: 'pass123',
      });

      // Create order for that user
      const order = await ordersService.create({
        userId: user.id,
        items: [{ productId: 1, quantity: 2 }],
        total: 100,
      });

      expect(order.userId).toBe(user.id);
      expect(order.items).toHaveLength(1);
    });

    it('should throw error when creating order for non-existent user', async () => {
      await expect(
        ordersService.create({
          userId: 999,
          items: [],
          total: 0,
        }),
      ).rejects.toThrow('User not found');
    });

    it('should load user orders', async () => {
      const user = await usersService.create({
        email: 'test@example.com',
        password: 'pass123',
      });

      await ordersService.create({ userId: user.id, items: [], total: 50 });
      await ordersService.create({ userId: user.id, items: [], total: 100 });

      const orders = await ordersService.findByUserId(user.id);

      expect(orders).toHaveLength(2);
    });
  });
});
```

**Testing Three Modules (Users + Auth + Mail):**

```typescript
// auth-flow.integration.spec.ts
describe('Auth Flow Integration', () => {
  let module: TestingModule;
  let usersService: UsersService;
  let authService: AuthService;
  let mockMailService: any;

  beforeAll(async () => {
    // Mock mail service
    mockMailService = {
      sendWelcomeEmail: jest.fn().mockResolvedValue(true),
      sendPasswordResetEmail: jest.fn().mockResolvedValue(true),
    };

    module = await Test.createTestingModule({
      imports: [
        TypeOrmModule.forRoot({
          type: 'sqlite',
          database: ':memory:',
          entities: [User],
          synchronize: true,
        }),
        UsersModule,
        AuthModule,
      ],
    })
      .overrideProvider(MailService)
      .useValue(mockMailService)
      .compile();

    usersService = module.get<UsersService>(UsersService);
    authService = module.get<AuthService>(AuthService);
  });

  it('should complete full registration flow', async () => {
    // Register user
    const registerDto = {
      email: 'new@example.com',
      password: 'pass123',
    };
    const user = await authService.register(registerDto);

    // Verify user created
    expect(user.email).toBe(registerDto.email);
    expect(mockMailService.sendWelcomeEmail).toHaveBeenCalledWith(user.email);

    // Login
    const tokens = await authService.login(registerDto);
    expect(tokens.access_token).toBeDefined();
    expect(tokens.refresh_token).toBeDefined();
  });

  it('should complete password reset flow', async () => {
    // Create user
    const user = await usersService.create({
      email: 'test@example.com',
      password: 'oldpass',
    });

    // Request password reset
    await authService.requestPasswordReset(user.email);
    expect(mockMailService.sendPasswordResetEmail).toHaveBeenCalled();

    // Reset password (in real scenario, token would come from email)
    const token = 'mock-reset-token';
    await authService.resetPassword(token, 'newpass123');

    // Login with new password
    const tokens = await authService.login({
      email: user.email,
      password: 'newpass123',
    });
    expect(tokens.access_token).toBeDefined();
  });
});
```

**Testing Complex Module Dependencies:**

```typescript
// e-commerce.integration.spec.ts
describe('E-commerce Integration', () => {
  let module: TestingModule;
  let usersService: UsersService;
  let productsService: ProductsService;
  let ordersService: OrdersService;
  let paymentService: PaymentService;
  let inventoryService: InventoryService;

  beforeAll(async () => {
    const mockPaymentService = {
      charge: jest.fn().mockResolvedValue({ success: true, transactionId: '123' }),
      refund: jest.fn().mockResolvedValue({ success: true }),
    };

    module = await Test.createTestingModule({
      imports: [
        TypeOrmModule.forRoot({
          type: 'sqlite',
          database: ':memory:',
          entities: [User, Product, Order, Inventory],
          synchronize: true,
        }),
        UsersModule,
        ProductsModule,
        OrdersModule,
        InventoryModule,
      ],
    })
      .overrideProvider(PaymentService)
      .useValue(mockPaymentService)
      .compile();

    usersService = module.get<UsersService>(UsersService);
    productsService = module.get<ProductsService>(ProductsService);
    ordersService = module.get<OrdersService>(OrdersService);
    paymentService = module.get(PaymentService);
    inventoryService = module.get<InventoryService>(InventoryService);
  });

  it('should complete full purchase flow', async () => {
    // 1. Create user
    const user = await usersService.create({
      email: 'buyer@example.com',
      password: 'pass123',
    });

    // 2. Create product with inventory
    const product = await productsService.create({
      name: 'Test Product',
      price: 50,
    });
    await inventoryService.setStock(product.id, 10);

    // 3. Create order
    const order = await ordersService.create({
      userId: user.id,
      items: [{ productId: product.id, quantity: 2 }],
      total: 100,
    });

    // 4. Verify payment called
    expect(paymentService.charge).toHaveBeenCalledWith(
      expect.objectContaining({ amount: 100 }),
    );

    // 5. Verify inventory reduced
    const updatedStock = await inventoryService.getStock(product.id);
    expect(updatedStock).toBe(8); // 10 - 2

    // 6. Verify order status
    expect(order.status).toBe('confirmed');
  });

  it('should rollback on payment failure', async () => {
    // Mock payment failure
    paymentService.charge.mockRejectedValueOnce(new Error('Payment failed'));

    const user = await usersService.create({ email: 'test@example.com' });
    const product = await productsService.create({ name: 'Product', price: 50 });
    await inventoryService.setStock(product.id, 10);

    const initialStock = await inventoryService.getStock(product.id);

    // Try to create order
    await expect(
      ordersService.create({
        userId: user.id,
        items: [{ productId: product.id, quantity: 2 }],
        total: 100,
      }),
    ).rejects.toThrow('Payment failed');

    // Verify stock not changed
    const finalStock = await inventoryService.getStock(product.id);
    expect(finalStock).toBe(initialStock);
  });
});
```

**Interview Tip**: Test multiple modules by **importing them together** in testing module. Use **real database** for interactions. Mock **external dependencies** (payment, email) but keep core modules real. Test **complete workflows** and **error scenarios**. Verify **transactions** roll back correctly. Clean database between tests.

</details>

## E2E (End-to-End) Testing

### 26. What is E2E testing and why is it important?

<details>
<summary>Answer</summary>

**E2E (End-to-End) testing** tests the **entire application** through the **HTTP layer**, simulating real user interactions from request to response.

**Why E2E Testing is Important:**

| Benefit | Description |
|---------|-------------|
| **High Confidence** | Tests full application stack |
| **Real Scenarios** | Tests as users would interact |
| **Integration Verification** | Ensures all parts work together |
| **Catches Edge Cases** | Finds issues unit tests miss |
| **API Contract** | Verifies API behaves as expected |
| **Regression Prevention** | Catches breaking changes |

**Test Pyramid with E2E:**

```
      /\
     /E2E\      ← 10% - Full application, slow, high confidence
    /------\
   /  INT   \   ← 30% - Multiple components, medium speed
  /----------\
 /    UNIT    \ ← 60% - Single components, fast, low confidence
/--------------\
```

**Unit vs Integration vs E2E:**

| Aspect | Unit | Integration | E2E |
|--------|------|-------------|-----|
| **Scope** | Single function | Multiple components | Full app |
| **Network** | No | No | Yes (HTTP) |
| **Database** | Mocked | Real test DB | Real test DB |
| **Speed** | 1-10ms | 100-500ms | 500-2000ms |
| **Confidence** | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Debugging** | Easy | Medium | Hard |
| **Cost** | Low | Medium | High |

**Basic E2E Test Example:**

```typescript
// test/users.e2e-spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { INestApplication } from '@nestjs/common';
import * as request from 'supertest';
import { AppModule } from '../src/app.module';

describe('Users E2E', () => {
  let app: INestApplication;

  beforeAll(async () => {
    // Create full application
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    
    // Apply same setup as main.ts
    app.useGlobalPipes(new ValidationPipe());
    
    await app.init();
  });

  afterAll(async () => {
    await app.close();
  });

  describe('/users (GET)', () => {
    it('should return array of users', () => {
      return request(app.getHttpServer())
        .get('/users')
        .expect(200)
        .expect((res) => {
          expect(Array.isArray(res.body)).toBe(true);
        });
    });
  });

  describe('/users (POST)', () => {
    it('should create a new user', () => {
      return request(app.getHttpServer())
        .post('/users')
        .send({
          email: 'test@example.com',
          password: 'pass123',
          name: 'Test User',
        })
        .expect(201)
        .expect((res) => {
          expect(res.body).toHaveProperty('id');
          expect(res.body.email).toBe('test@example.com');
        });
    });

    it('should return 400 for invalid email', () => {
      return request(app.getHttpServer())
        .post('/users')
        .send({
          email: 'invalid-email',
          password: 'pass123',
        })
        .expect(400)
        .expect((res) => {
          expect(res.body.message).toContain('email');
        });
    });
  });

  describe('/users/:id (GET)', () => {
    it('should return user by id', async () => {
      // Create user first
      const createRes = await request(app.getHttpServer())
        .post('/users')
        .send({ email: 'test@example.com', password: 'pass123' });

      const userId = createRes.body.id;

      // Get user
      return request(app.getHttpServer())
        .get(`/users/${userId}`)
        .expect(200)
        .expect((res) => {
          expect(res.body.id).toBe(userId);
          expect(res.body.email).toBe('test@example.com');
        });
    });

    it('should return 404 for non-existent user', () => {
      return request(app.getHttpServer())
        .get('/users/999')
        .expect(404);
    });
  });
});
```

**What E2E Tests Should Cover:**

```typescript
describe('E2E Test Coverage', () => {
  // ✅ Happy paths
  it('should complete successful user registration', () => {});
  it('should complete successful login', () => {});
  it('should complete successful purchase', () => {});

  // ✅ Error scenarios
  it('should handle invalid input', () => {});
  it('should handle unauthorized access', () => {});
  it('should handle duplicate email', () => {});

  // ✅ Edge cases
  it('should handle concurrent requests', () => {});
  it('should handle large payloads', () => {});
  it('should handle missing required fields', () => {});

  // ✅ User workflows
  it('should complete full checkout flow', () => {});
  it('should complete password reset flow', () => {});
  it('should complete profile update flow', () => {});

  // ✅ Security
  it('should reject requests without auth token', () => {});
  it('should reject expired tokens', () => {});
  it('should prevent SQL injection', () => {});
});
```

**E2E vs Integration Testing:**

```typescript
// Integration Test - Direct service calls
it('should create order', async () => {
  const user = await usersService.create({ email: 'test@example.com' });
  const order = await ordersService.create({ userId: user.id });
  expect(order).toBeDefined();
});

// E2E Test - HTTP requests
it('should create order via API', async () => {
  // Register user
  const registerRes = await request(app.getHttpServer())
    .post('/auth/register')
    .send({ email: 'test@example.com', password: 'pass123' });
  
  const token = registerRes.body.access_token;
  
  // Create order
  return request(app.getHttpServer())
    .post('/orders')
    .set('Authorization', `Bearer ${token}`)
    .send({ items: [{ productId: 1, quantity: 2 }] })
    .expect(201);
});
```

**Interview Tip**: E2E tests **test full application via HTTP**, providing **highest confidence** but are **slowest**. Use for **critical user flows**, **API contracts**, and **end-to-end scenarios**. Keep E2E tests to **10%** of total tests. Focus on **happy paths** and **critical error scenarios**. Run E2E tests before deployment to catch integration issues.

</details>

### 27. How do you set up E2E tests using `supertest`?

<details>
<summary>Answer</summary>

**Set up E2E tests** using `@nestjs/testing` to create the app and `supertest` to make HTTP requests.

**Installation:**

```bash
npm install --save-dev supertest @types/supertest
```

**Basic E2E Test Setup:**

```typescript
// test/app.e2e-spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { INestApplication, ValidationPipe } from '@nestjs/common';
import * as request from 'supertest';
import { AppModule } from '../src/app.module';

describe('AppController (e2e)', () => {
  let app: INestApplication;

  beforeAll(async () => {
    // 1. Create testing module
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule], // Import main app module
    }).compile();

    // 2. Create Nest application
    app = moduleFixture.createNestApplication();
    
    // 3. Apply same configuration as main.ts
    app.useGlobalPipes(
      new ValidationPipe({
        whitelist: true,
        transform: true,
      }),
    );
    
    // 4. Initialize app
    await app.init();
  });

  afterAll(async () => {
    // Clean up
    await app.close();
  });

  // 5. Write tests
  it('/ (GET)', () => {
    return request(app.getHttpServer())
      .get('/')
      .expect(200)
      .expect('Hello World!');
  });
});
```

**Jest E2E Configuration:**

```javascript
// test/jest-e2e.json
{
  "moduleFileExtensions": ["js", "json", "ts"],
  "rootDir": ".",
  "testEnvironment": "node",
  "testRegex": ".e2e-spec.ts$",
  "transform": {
    "^.+\\.(t|j)s$": "ts-jest"
  },
  "moduleNameMapper": {
    "^@/(.*)$": "<rootDir>/../src/$1"
  }
}
```

```json
// package.json
{
  "scripts": {
    "test:e2e": "jest --config ./test/jest-e2e.json",
    "test:e2e:watch": "jest --config ./test/jest-e2e.json --watch",
    "test:e2e:cov": "jest --config ./test/jest-e2e.json --coverage"
  }
}
```

**Complete E2E Test Setup with Database:**

```typescript
// test/users.e2e-spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { INestApplication } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import * as request from 'supertest';
import { UsersModule } from '../src/users/users.module';
import { User } from '../src/users/entities/user.entity';
import { DataSource } from 'typeorm';

describe('Users E2E', () => {
  let app: INestApplication;
  let dataSource: DataSource;

  beforeAll(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [
        // Test database configuration
        TypeOrmModule.forRoot({
          type: 'sqlite',
          database: ':memory:',
          entities: [User],
          synchronize: true,
        }),
        UsersModule,
      ],
    }).compile();

    app = moduleFixture.createNestApplication();
    dataSource = moduleFixture.get<DataSource>(DataSource);
    
    await app.init();
  });

  afterEach(async () => {
    // Clean database after each test
    const entities = dataSource.entityMetadatas;
    for (const entity of entities) {
      const repository = dataSource.getRepository(entity.name);
      await repository.clear();
    }
  });

  afterAll(async () => {
    await dataSource.destroy();
    await app.close();
  });

  describe('/users (POST)', () => {
    it('should create a user', () => {
      return request(app.getHttpServer())
        .post('/users')
        .send({
          email: 'test@example.com',
          password: 'pass123',
        })
        .expect(201)
        .expect((res) => {
          expect(res.body).toHaveProperty('id');
          expect(res.body.email).toBe('test@example.com');
        });
    });
  });
});
```

**Supertest API Methods:**

```typescript
describe('Supertest methods', () => {
  it('should test GET request', () => {
    return request(app.getHttpServer())
      .get('/users')
      .expect(200);
  });

  it('should test POST request', () => {
    return request(app.getHttpServer())
      .post('/users')
      .send({ email: 'test@example.com' })
      .expect(201);
  });

  it('should test PUT request', () => {
    return request(app.getHttpServer())
      .put('/users/1')
      .send({ name: 'Updated' })
      .expect(200);
  });

  it('should test DELETE request', () => {
    return request(app.getHttpServer())
      .delete('/users/1')
      .expect(204);
  });

  it('should test with headers', () => {
    return request(app.getHttpServer())
      .get('/users')
      .set('Authorization', 'Bearer token123')
      .set('Accept', 'application/json')
      .expect(200);
  });

  it('should test with query parameters', () => {
    return request(app.getHttpServer())
      .get('/users')
      .query({ role: 'admin', limit: 10 })
      .expect(200);
  });

  it('should test response body', () => {
    return request(app.getHttpServer())
      .get('/users/1')
      .expect(200)
      .expect((res) => {
        expect(res.body.email).toBe('test@example.com');
        expect(res.body).toHaveProperty('id');
      });
  });

  it('should test response headers', () => {
    return request(app.getHttpServer())
      .get('/users')
      .expect(200)
      .expect('Content-Type', /json/);
  });
});
```

**Using async/await with Supertest:**

```typescript
describe('Async supertest', () => {
  it('should create and retrieve user', async () => {
    // Create user
    const createRes = await request(app.getHttpServer())
      .post('/users')
      .send({ email: 'test@example.com', password: 'pass123' })
      .expect(201);

    const userId = createRes.body.id;

    // Retrieve user
    const getRes = await request(app.getHttpServer())
      .get(`/users/${userId}`)
      .expect(200);

    expect(getRes.body.email).toBe('test@example.com');
  });
});
```

**Interview Tip**: Set up E2E tests by creating **full Nest application** with `createNestApplication()`, applying **same config as main.ts**, using **`supertest`** for HTTP requests, and **test database** (SQLite in-memory). Clean database in `afterEach()`, close app in `afterAll()`. Use **`.expect()`** for status codes and response validation.

</details>

### 28. How do you test HTTP endpoints in E2E tests?

<details>
<summary>Answer</summary>

**Test HTTP endpoints** using `supertest` to make requests and verify responses, status codes, headers, and body.

**Testing Different HTTP Methods:**

```typescript
import * as request from 'supertest';

describe('HTTP Endpoints E2E', () => {
  let app: INestApplication;

  beforeAll(async () => {
    const module = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();
    app = module.createNestApplication();
    await app.init();
  });

  afterAll(async () => {
    await app.close();
  });

  // GET requests
  describe('GET /users', () => {
    it('should return all users', () => {
      return request(app.getHttpServer())
        .get('/users')
        .expect(200)
        .expect('Content-Type', /json/)
        .expect((res) => {
          expect(Array.isArray(res.body)).toBe(true);
        });
    });

    it('should filter users by role', () => {
      return request(app.getHttpServer())
        .get('/users')
        .query({ role: 'admin' }) // Query parameters
        .expect(200)
        .expect((res) => {
          res.body.forEach((user) => {
            expect(user.role).toBe('admin');
          });
        });
    });
  });

  // POST requests
  describe('POST /users', () => {
    it('should create user with valid data', () => {
      return request(app.getHttpServer())
        .post('/users')
        .send({
          email: 'new@example.com',
          password: 'pass123',
          name: 'New User',
        })
        .expect(201)
        .expect((res) => {
          expect(res.body).toHaveProperty('id');
          expect(res.body.email).toBe('new@example.com');
          expect(res.body.password).toBeUndefined(); // Password should not be returned
        });
    });

    it('should validate required fields', () => {
      return request(app.getHttpServer())
        .post('/users')
        .send({ email: 'test@example.com' }) // Missing password
        .expect(400)
        .expect((res) => {
          expect(res.body.message).toContain('password');
        });
    });

    it('should validate email format', () => {
      return request(app.getHttpServer())
        .post('/users')
        .send({
          email: 'invalid-email',
          password: 'pass123',
        })
        .expect(400);
    });
  });

  // PUT requests
  describe('PUT /users/:id', () => {
    it('should update user', async () => {
      // Create user first
      const createRes = await request(app.getHttpServer())
        .post('/users')
        .send({ email: 'test@example.com', password: 'pass123' });

      const userId = createRes.body.id;

      // Update user
      return request(app.getHttpServer())
        .put(`/users/${userId}`)
        .send({ name: 'Updated Name' })
        .expect(200)
        .expect((res) => {
          expect(res.body.name).toBe('Updated Name');
        });
    });

    it('should return 404 for non-existent user', () => {
      return request(app.getHttpServer())
        .put('/users/999')
        .send({ name: 'Updated' })
        .expect(404);
    });
  });

  // PATCH requests
  describe('PATCH /users/:id', () => {
    it('should partially update user', async () => {
      const createRes = await request(app.getHttpServer())
        .post('/users')
        .send({ email: 'test@example.com', password: 'pass123', name: 'Original' });

      return request(app.getHttpServer())
        .patch(`/users/${createRes.body.id}`)
        .send({ name: 'Patched' }) // Only update name
        .expect(200)
        .expect((res) => {
          expect(res.body.name).toBe('Patched');
          expect(res.body.email).toBe('test@example.com'); // Email unchanged
        });
    });
  });

  // DELETE requests
  describe('DELETE /users/:id', () => {
    it('should delete user', async () => {
      const createRes = await request(app.getHttpServer())
        .post('/users')
        .send({ email: 'test@example.com', password: 'pass123' });

      const userId = createRes.body.id;

      // Delete
      await request(app.getHttpServer())
        .delete(`/users/${userId}`)
        .expect(204);

      // Verify deleted
      await request(app.getHttpServer())
        .get(`/users/${userId}`)
        .expect(404);
    });
  });
});
```

**Testing Headers:**

```typescript
describe('Headers testing', () => {
  it('should set custom headers in request', () => {
    return request(app.getHttpServer())
      .get('/users')
      .set('Authorization', 'Bearer token123')
      .set('Accept', 'application/json')
      .set('X-Custom-Header', 'value')
      .expect(200);
  });

  it('should verify response headers', () => {
    return request(app.getHttpServer())
      .get('/users')
      .expect(200)
      .expect('Content-Type', /json/)
      .expect((res) => {
        expect(res.headers['x-total-count']).toBeDefined();
      });
  });
});
```

**Testing Query Parameters:**

```typescript
describe('Query parameters', () => {
  it('should handle single query param', () => {
    return request(app.getHttpServer())
      .get('/users')
      .query({ role: 'admin' })
      .expect(200);
  });

  it('should handle multiple query params', () => {
    return request(app.getHttpServer())
      .get('/users')
      .query({ role: 'admin', status: 'active', limit: 10 })
      .expect(200);
  });

  it('should handle array query params', () => {
    return request(app.getHttpServer())
      .get('/users')
      .query({ ids: [1, 2, 3] })
      .expect(200);
  });
});
```

**Testing File Uploads:**

```typescript
import * as path from 'path';

describe('File uploads', () => {
  it('should upload file', () => {
    return request(app.getHttpServer())
      .post('/upload')
      .attach('file', path.join(__dirname, 'fixtures/test-file.jpg'))
      .expect(201)
      .expect((res) => {
        expect(res.body.filename).toBeDefined();
        expect(res.body.mimetype).toBe('image/jpeg');
      });
  });

  it('should upload multiple files', () => {
    return request(app.getHttpServer())
      .post('/upload/multiple')
      .attach('files', path.join(__dirname, 'fixtures/file1.jpg'))
      .attach('files', path.join(__dirname, 'fixtures/file2.jpg'))
      .expect(201);
  });
});
```

**Testing Complex Response Bodies:**

```typescript
describe('Response body validation', () => {
  it('should match exact response', () => {
    return request(app.getHttpServer())
      .get('/users/1')
      .expect(200)
      .expect({
        id: 1,
        email: 'test@example.com',
        name: 'Test User',
      });
  });

  it('should match partial response', () => {
    return request(app.getHttpServer())
      .get('/users/1')
      .expect(200)
      .expect((res) => {
        expect(res.body).toMatchObject({
          email: 'test@example.com',
        });
      });
  });

  it('should validate nested objects', () => {
    return request(app.getHttpServer())
      .get('/users/1')
      .expect(200)
      .expect((res) => {
        expect(res.body.profile).toMatchObject({
          age: expect.any(Number),
          address: expect.objectContaining({
            city: expect.any(String),
          }),
        });
      });
  });
});
```

**Testing Error Responses:**

```typescript
describe('Error responses', () => {
  it('should return 400 for bad request', () => {
    return request(app.getHttpServer())
      .post('/users')
      .send({ invalid: 'data' })
      .expect(400)
      .expect((res) => {
        expect(res.body).toHaveProperty('statusCode', 400);
        expect(res.body).toHaveProperty('message');
        expect(res.body).toHaveProperty('error', 'Bad Request');
      });
  });

  it('should return 404 for not found', () => {
    return request(app.getHttpServer())
      .get('/users/999')
      .expect(404)
      .expect((res) => {
        expect(res.body.message).toContain('not found');
      });
  });

  it('should return 409 for conflict', async () => {
    await request(app.getHttpServer())
      .post('/users')
      .send({ email: 'duplicate@example.com', password: 'pass' });

    return request(app.getHttpServer())
      .post('/users')
      .send({ email: 'duplicate@example.com', password: 'pass' })
      .expect(409);
  });
});
```

**Interview Tip**: Use **`request(app.getHttpServer())`** to make HTTP requests. Chain **`.expect()`** for status codes, headers, and body validation. Use **`.send()`** for request body, **`.query()`** for query params, **`.set()`** for headers. Test **all HTTP methods** (GET, POST, PUT, PATCH, DELETE), **validation errors**, **edge cases**, and **error responses**.

</details>

### 29. How do you test authentication in E2E tests?

<details>
<summary>Answer</summary>

**Test authentication** by registering/logging in users, obtaining tokens, and using them in subsequent requests.

**Complete Authentication Flow E2E:**

```typescript
// test/auth.e2e-spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { INestApplication } from '@nestjs/common';
import * as request from 'supertest';
import { AppModule } from '../src/app.module';

describe('Authentication E2E', () => {
  let app: INestApplication;
  let accessToken: string;
  let refreshToken: string;

  beforeAll(async () => {
    const moduleFixture = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();
  });

  afterAll(async () => {
    await app.close();
  });

  describe('POST /auth/register', () => {
    it('should register a new user', () => {
      return request(app.getHttpServer())
        .post('/auth/register')
        .send({
          email: 'newuser@example.com',
          password: 'password123',
          name: 'New User',
        })
        .expect(201)
        .expect((res) => {
          expect(res.body).toHaveProperty('id');
          expect(res.body).toHaveProperty('email', 'newuser@example.com');
          expect(res.body).not.toHaveProperty('password'); // Password should not be returned
        });
    });

    it('should reject duplicate email', async () => {
      const userData = {
        email: 'duplicate@example.com',
        password: 'pass123',
      };

      // Register first time
      await request(app.getHttpServer())
        .post('/auth/register')
        .send(userData)
        .expect(201);

      // Try to register again
      return request(app.getHttpServer())
        .post('/auth/register')
        .send(userData)
        .expect(409)
        .expect((res) => {
          expect(res.body.message).toContain('already exists');
        });
    });

    it('should validate password strength', () => {
      return request(app.getHttpServer())
        .post('/auth/register')
        .send({
          email: 'test@example.com',
          password: '123', // Weak password
        })
        .expect(400)
        .expect((res) => {
          expect(res.body.message).toContain('password');
        });
    });
  });

  describe('POST /auth/login', () => {
    beforeEach(async () => {
      // Register user for login tests
      await request(app.getHttpServer())
        .post('/auth/register')
        .send({
          email: 'loginuser@example.com',
          password: 'password123',
        });
    });

    it('should login with correct credentials', () => {
      return request(app.getHttpServer())
        .post('/auth/login')
        .send({
          email: 'loginuser@example.com',
          password: 'password123',
        })
        .expect(200)
        .expect((res) => {
          expect(res.body).toHaveProperty('access_token');
          expect(res.body).toHaveProperty('refresh_token');
          expect(typeof res.body.access_token).toBe('string');
          
          // Save tokens for later tests
          accessToken = res.body.access_token;
          refreshToken = res.body.refresh_token;
        });
    });

    it('should reject wrong password', () => {
      return request(app.getHttpServer())
        .post('/auth/login')
        .send({
          email: 'loginuser@example.com',
          password: 'wrongpassword',
        })
        .expect(401)
        .expect((res) => {
          expect(res.body.message).toContain('Invalid credentials');
        });
    });

    it('should reject non-existent user', () => {
      return request(app.getHttpServer())
        .post('/auth/login')
        .send({
          email: 'nonexistent@example.com',
          password: 'password123',
        })
        .expect(401);
    });
  });

  describe('GET /auth/profile', () => {
    let token: string;

    beforeEach(async () => {
      // Register and login to get token
      await request(app.getHttpServer())
        .post('/auth/register')
        .send({ email: 'profile@example.com', password: 'pass123' });

      const loginRes = await request(app.getHttpServer())
        .post('/auth/login')
        .send({ email: 'profile@example.com', password: 'pass123' });

      token = loginRes.body.access_token;
    });

    it('should get profile with valid token', () => {
      return request(app.getHttpServer())
        .get('/auth/profile')
        .set('Authorization', `Bearer ${token}`)
        .expect(200)
        .expect((res) => {
          expect(res.body).toHaveProperty('email', 'profile@example.com');
          expect(res.body).not.toHaveProperty('password');
        });
    });

    it('should reject request without token', () => {
      return request(app.getHttpServer())
        .get('/auth/profile')
        .expect(401);
    });

    it('should reject request with invalid token', () => {
      return request(app.getHttpServer())
        .get('/auth/profile')
        .set('Authorization', 'Bearer invalid_token')
        .expect(401);
    });

    it('should reject request with expired token', () => {
      const expiredToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...';
      return request(app.getHttpServer())
        .get('/auth/profile')
        .set('Authorization', `Bearer ${expiredToken}`)
        .expect(401)
        .expect((res) => {
          expect(res.body.message).toContain('expired');
        });
    });
  });

  describe('POST /auth/refresh', () => {
    it('should refresh tokens', async () => {
      // Register and login
      await request(app.getHttpServer())
        .post('/auth/register')
        .send({ email: 'refresh@example.com', password: 'pass123' });

      const loginRes = await request(app.getHttpServer())
        .post('/auth/login')
        .send({ email: 'refresh@example.com', password: 'pass123' });

      const refreshToken = loginRes.body.refresh_token;

      // Refresh tokens
      return request(app.getHttpServer())
        .post('/auth/refresh')
        .send({ refresh_token: refreshToken })
        .expect(200)
        .expect((res) => {
          expect(res.body).toHaveProperty('access_token');
          expect(res.body.access_token).not.toBe(loginRes.body.access_token);
        });
    });

    it('should reject invalid refresh token', () => {
      return request(app.getHttpServer())
        .post('/auth/refresh')
        .send({ refresh_token: 'invalid_token' })
        .expect(401);
    });
  });

  describe('POST /auth/logout', () => {
    it('should logout and invalidate tokens', async () => {
      // Register and login
      await request(app.getHttpServer())
        .post('/auth/register')
        .send({ email: 'logout@example.com', password: 'pass123' });

      const loginRes = await request(app.getHttpServer())
        .post('/auth/login')
        .send({ email: 'logout@example.com', password: 'pass123' });

      const token = loginRes.body.access_token;

      // Logout
      await request(app.getHttpServer())
        .post('/auth/logout')
        .set('Authorization', `Bearer ${token}`)
        .expect(200);

      // Try to use token after logout
      return request(app.getHttpServer())
        .get('/auth/profile')
        .set('Authorization', `Bearer ${token}`)
        .expect(401);
    });
  });
});
```

**Helper Function for Authentication:**

```typescript
// test/helpers/auth.helper.ts
export async function registerAndLogin(
  app: INestApplication,
  email: string,
  password: string,
) {
  // Register
  await request(app.getHttpServer())
    .post('/auth/register')
    .send({ email, password });

  // Login
  const loginRes = await request(app.getHttpServer())
    .post('/auth/login')
    .send({ email, password });

  return {
    accessToken: loginRes.body.access_token,
    refreshToken: loginRes.body.refresh_token,
    user: loginRes.body.user,
  };
}

// Usage in tests
it('should access protected route', async () => {
  const { accessToken } = await registerAndLogin(
    app,
    'test@example.com',
    'pass123',
  );

  return request(app.getHttpServer())
    .get('/protected')
    .set('Authorization', `Bearer ${accessToken}`)
    .expect(200);
});
```

**Interview Tip**: Test **complete auth flow**: register → login → get token → use token → refresh → logout. Verify **token presence**, **format**, and **expiration**. Test **unauthorized access**, **invalid tokens**, **expired tokens**. Use **helper functions** to register/login for reusable authentication. Test **token refresh** and **logout invalidation**.

</details>

### 30. How do you test protected routes?

<details>
<summary>Answer</summary>

**Test protected routes** by authenticating first, obtaining tokens, then using them in Authorization headers.

**Testing Protected Routes:**

```typescript
// test/protected-routes.e2e-spec.ts
import { Test } from '@nestjs/testing';
import { INestApplication } from '@nestjs/common';
import * as request from 'supertest';
import { AppModule } from '../src/app.module';

describe('Protected Routes E2E', () => {
  let app: INestApplication;
  let adminToken: string;
  let userToken: string;

  beforeAll(async () => {
    const module = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = module.createNestApplication();
    await app.init();

    // Setup: Create users with different roles
    await request(app.getHttpServer())
      .post('/auth/register')
      .send({ email: 'admin@example.com', password: 'pass123', role: 'admin' });

    await request(app.getHttpServer())
      .post('/auth/register')
      .send({ email: 'user@example.com', password: 'pass123', role: 'user' });

    // Get admin token
    const adminLogin = await request(app.getHttpServer())
      .post('/auth/login')
      .send({ email: 'admin@example.com', password: 'pass123' });
    adminToken = adminLogin.body.access_token;

    // Get user token
    const userLogin = await request(app.getHttpServer())
      .post('/auth/login')
      .send({ email: 'user@example.com', password: 'pass123' });
    userToken = userLogin.body.access_token;
  });

  afterAll(async () => {
    await app.close();
  });

  describe('Authentication Required', () => {
    it('should reject request without token', () => {
      return request(app.getHttpServer())
        .get('/users/profile')
        .expect(401)
        .expect((res) => {
          expect(res.body.message).toContain('Unauthorized');
        });
    });

    it('should reject request with invalid token', () => {
      return request(app.getHttpServer())
        .get('/users/profile')
        .set('Authorization', 'Bearer invalid_token_here')
        .expect(401);
    });

    it('should accept request with valid token', () => {
      return request(app.getHttpServer())
        .get('/users/profile')
        .set('Authorization', `Bearer ${userToken}`)
        .expect(200);
    });

    it('should reject malformed Authorization header', () => {
      return request(app.getHttpServer())
        .get('/users/profile')
        .set('Authorization', userToken) // Missing 'Bearer '
        .expect(401);
    });
  });

  describe('Role-Based Access Control', () => {
    it('should allow admin to access admin route', () => {
      return request(app.getHttpServer())
        .get('/admin/users')
        .set('Authorization', `Bearer ${adminToken}`)
        .expect(200);
    });

    it('should deny regular user access to admin route', () => {
      return request(app.getHttpServer())
        .get('/admin/users')
        .set('Authorization', `Bearer ${userToken}`)
        .expect(403)
        .expect((res) => {
          expect(res.body.message).toContain('Forbidden');
        });
    });

    it('should allow user to access own resource', async () => {
      const { body: user } = await request(app.getHttpServer())
        .get('/users/profile')
        .set('Authorization', `Bearer ${userToken}`);

      return request(app.getHttpServer())
        .get(`/users/${user.id}`)
        .set('Authorization', `Bearer ${userToken}`)
        .expect(200);
    });

    it('should deny user access to other user resource', () => {
      return request(app.getHttpServer())
        .get('/users/999') // Different user ID
        .set('Authorization', `Bearer ${userToken}`)
        .expect(403);
    });
  });

  describe('Protected CRUD Operations', () => {
    it('should allow authenticated POST', () => {
      return request(app.getHttpServer())
        .post('/posts')
        .set('Authorization', `Bearer ${userToken}`)
        .send({
          title: 'New Post',
          content: 'Post content',
        })
        .expect(201)
        .expect((res) => {
          expect(res.body).toHaveProperty('id');
          expect(res.body.title).toBe('New Post');
        });
    });

    it('should reject unauthenticated POST', () => {
      return request(app.getHttpServer())
        .post('/posts')
        .send({ title: 'New Post' })
        .expect(401);
    });

    it('should allow user to update own post', async () => {
      // Create post
      const createRes = await request(app.getHttpServer())
        .post('/posts')
        .set('Authorization', `Bearer ${userToken}`)
        .send({ title: 'My Post' });

      const postId = createRes.body.id;

      // Update post
      return request(app.getHttpServer())
        .put(`/posts/${postId}`)
        .set('Authorization', `Bearer ${userToken}`)
        .send({ title: 'Updated Title' })
        .expect(200);
    });

    it('should deny user update of other user post', async () => {
      // Create post as admin
      const createRes = await request(app.getHttpServer())
        .post('/posts')
        .set('Authorization', `Bearer ${adminToken}`)
        .send({ title: 'Admin Post' });

      const postId = createRes.body.id;

      // Try to update as regular user
      return request(app.getHttpServer())
        .put(`/posts/${postId}`)
        .set('Authorization', `Bearer ${userToken}`)
        .send({ title: 'Hacked' })
        .expect(403);
    });

    it('should allow user to delete own post', async () => {
      // Create post
      const createRes = await request(app.getHttpServer())
        .post('/posts')
        .set('Authorization', `Bearer ${userToken}`)
        .send({ title: 'My Post' });

      // Delete post
      return request(app.getHttpServer())
        .delete(`/posts/${createRes.body.id}`)
        .set('Authorization', `Bearer ${userToken}`)
        .expect(204);
    });
  });

  describe('Token Expiration', () => {
    it('should reject expired token', () => {
      const expiredToken = 'expired.jwt.token';
      return request(app.getHttpServer())
        .get('/users/profile')
        .set('Authorization', `Bearer ${expiredToken}`)
        .expect(401)
        .expect((res) => {
          expect(res.body.message).toMatch(/expired|invalid/i);
        });
    });
  });

  describe('Multiple Endpoints', () => {
    it('should use same token for multiple requests', async () => {
      // Request 1
      await request(app.getHttpServer())
        .get('/users/profile')
        .set('Authorization', `Bearer ${userToken}`)
        .expect(200);

      // Request 2
      await request(app.getHttpServer())
        .get('/posts')
        .set('Authorization', `Bearer ${userToken}`)
        .expect(200);

      // Request 3
      await request(app.getHttpServer())
        .post('/comments')
        .set('Authorization', `Bearer ${userToken}`)
        .send({ text: 'Comment' })
        .expect(201);
    });
  });
});
```

**Reusable Auth Helper:**

```typescript
// test/helpers/auth.helper.ts
export class AuthHelper {
  private tokens: Map<string, string> = new Map();

  constructor(private app: INestApplication) {}

  async registerAndLogin(email: string, password: string, role?: string) {
    // Register
    await request(this.app.getHttpServer())
      .post('/auth/register')
      .send({ email, password, role });

    // Login
    const res = await request(this.app.getHttpServer())
      .post('/auth/login')
      .send({ email, password });

    const token = res.body.access_token;
    this.tokens.set(email, token);
    return token;
  }

  getToken(email: string): string {
    return this.tokens.get(email);
  }

  makeAuthRequest(method: 'get' | 'post' | 'put' | 'delete', path: string, token: string) {
    return request(this.app.getHttpServer())
      [method](path)
      .set('Authorization', `Bearer ${token}`);
  }
}

// Usage
describe('With AuthHelper', () => {
  let authHelper: AuthHelper;

  beforeAll(async () => {
    authHelper = new AuthHelper(app);
    await authHelper.registerAndLogin('test@example.com', 'pass123');
  });

  it('should access protected route', () => {
    const token = authHelper.getToken('test@example.com');
    return authHelper
      .makeAuthRequest('get', '/protected', token)
      .expect(200);
  });
});
```

**Interview Tip**: Test **authentication required** (401 without token), **authorization** (403 wrong role), **own vs others' resources**. Use **`set('Authorization', `Bearer ${token}`)`** for authenticated requests. Test **CRUD operations** with auth, **expired tokens**, **invalid tokens**. Create **helper functions** for register/login to avoid duplication. Test **role-based access control** with different user roles.

</details>

## Testing Guards, Pipes, Interceptors

### 31. How do you test Guards?

<details>
<summary>Answer</summary>

**Test Guards** by calling `canActivate()` directly with a mocked `ExecutionContext` and verifying the return value.

**Testing Basic Auth Guard:**

```typescript
// auth.guard.spec.ts
import { AuthGuard } from './auth.guard';
import { ExecutionContext, UnauthorizedException } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';

describe('AuthGuard', () => {
  let guard: AuthGuard;
  let jwtService: JwtService;

  beforeEach(() => {
    jwtService = {
      verify: jest.fn(),
    } as any;

    guard = new AuthGuard(jwtService);
  });

  describe('canActivate', () => {
    it('should return true for valid token', () => {
      const mockContext = createMockExecutionContext({
        headers: { authorization: 'Bearer valid-token' },
      });

      jwtService.verify = jest.fn().mockReturnValue({ userId: 1, email: 'test@example.com' });

      const result = guard.canActivate(mockContext);

      expect(result).toBe(true);
      expect(jwtService.verify).toHaveBeenCalledWith('valid-token');
    });

    it('should throw UnauthorizedException when token is missing', () => {
      const mockContext = createMockExecutionContext({
        headers: {},
      });

      expect(() => guard.canActivate(mockContext)).toThrow(UnauthorizedException);
    });

    it('should throw UnauthorizedException for invalid token', () => {
      const mockContext = createMockExecutionContext({
        headers: { authorization: 'Bearer invalid-token' },
      });

      jwtService.verify = jest.fn().mockImplementation(() => {
        throw new Error('Invalid token');
      });

      expect(() => guard.canActivate(mockContext)).toThrow(UnauthorizedException);
    });

    it('should throw UnauthorizedException for malformed authorization header', () => {
      const mockContext = createMockExecutionContext({
        headers: { authorization: 'InvalidFormat' },
      });

      expect(() => guard.canActivate(mockContext)).toThrow(UnauthorizedException);
    });

    it('should attach user to request object', () => {
      const mockRequest = { headers: { authorization: 'Bearer valid-token' } };
      const mockContext = createMockExecutionContext(mockRequest);

      const userData = { userId: 1, email: 'test@example.com' };
      jwtService.verify = jest.fn().mockReturnValue(userData);

      guard.canActivate(mockContext);

      expect(mockRequest['user']).toEqual(userData);
    });
  });
});

// Helper function to create mock ExecutionContext
function createMockExecutionContext(request: any): ExecutionContext {
  return {
    switchToHttp: () => ({
      getRequest: () => request,
      getResponse: () => ({}),
    }),
    getHandler: () => ({}),
    getClass: () => ({}),
  } as ExecutionContext;
}
```

**Testing Roles Guard:**

```typescript
// roles.guard.spec.ts
import { RolesGuard } from './roles.guard';
import { Reflector } from '@nestjs/core';
import { ExecutionContext } from '@nestjs/common';

describe('RolesGuard', () => {
  let guard: RolesGuard;
  let reflector: Reflector;

  beforeEach(() => {
    reflector = new Reflector();
    guard = new RolesGuard(reflector);
  });

  it('should allow access when no roles are required', () => {
    jest.spyOn(reflector, 'getAllAndOverride').mockReturnValue(undefined);

    const mockContext = createMockExecutionContext({
      user: { id: 1, role: 'user' },
    });

    const result = guard.canActivate(mockContext);

    expect(result).toBe(true);
  });

  it('should allow access when user has required role', () => {
    jest.spyOn(reflector, 'getAllAndOverride').mockReturnValue(['admin']);

    const mockContext = createMockExecutionContext({
      user: { id: 1, role: 'admin' },
    });

    const result = guard.canActivate(mockContext);

    expect(result).toBe(true);
  });

  it('should deny access when user does not have required role', () => {
    jest.spyOn(reflector, 'getAllAndOverride').mockReturnValue(['admin']);

    const mockContext = createMockExecutionContext({
      user: { id: 1, role: 'user' },
    });

    const result = guard.canActivate(mockContext);

    expect(result).toBe(false);
  });

  it('should allow access when user has one of multiple required roles', () => {
    jest.spyOn(reflector, 'getAllAndOverride').mockReturnValue(['admin', 'moderator']);

    const mockContext = createMockExecutionContext({
      user: { id: 1, role: 'moderator' },
    });

    const result = guard.canActivate(mockContext);

    expect(result).toBe(true);
  });

  it('should deny access when user is not authenticated', () => {
    jest.spyOn(reflector, 'getAllAndOverride').mockReturnValue(['admin']);

    const mockContext = createMockExecutionContext({
      user: null,
    });

    const result = guard.canActivate(mockContext);

    expect(result).toBe(false);
  });
});

function createMockExecutionContext(request: any): ExecutionContext {
  return {
    switchToHttp: () => ({
      getRequest: () => request,
    }),
    getHandler: () => ({}),
    getClass: () => ({}),
  } as ExecutionContext;
}
```

**Testing Guard with Async canActivate:**

```typescript
// api-key.guard.spec.ts
import { ApiKeyGuard } from './api-key.guard';
import { ConfigService } from '@nestjs/config';
import { ExecutionContext } from '@nestjs/common';

describe('ApiKeyGuard', () => {
  let guard: ApiKeyGuard;
  let configService: ConfigService;

  beforeEach(() => {
    configService = {
      get: jest.fn().mockReturnValue('secret-api-key'),
    } as any;

    guard = new ApiKeyGuard(configService);
  });

  it('should allow access with valid API key', async () => {
    const mockContext = createMockExecutionContext({
      headers: { 'x-api-key': 'secret-api-key' },
    });

    const result = await guard.canActivate(mockContext);

    expect(result).toBe(true);
  });

  it('should deny access with invalid API key', async () => {
    const mockContext = createMockExecutionContext({
      headers: { 'x-api-key': 'wrong-key' },
    });

    const result = await guard.canActivate(mockContext);

    expect(result).toBe(false);
  });

  it('should deny access when API key is missing', async () => {
    const mockContext = createMockExecutionContext({
      headers: {},
    });

    const result = await guard.canActivate(mockContext);

    expect(result).toBe(false);
  });
});
```

**Testing Guard in Controller Context:**

```typescript
// users.controller.spec.ts
import { Test } from '@nestjs/testing';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';
import { AuthGuard } from '../guards/auth.guard';
import { RolesGuard } from '../guards/roles.guard';

describe('UsersController with Guards', () => {
  let controller: UsersController;
  let service: UsersService;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      controllers: [UsersController],
      providers: [
        {
          provide: UsersService,
          useValue: {
            findAll: jest.fn().mockResolvedValue([]),
          },
        },
      ],
    })
      .overrideGuard(AuthGuard)
      .useValue({ canActivate: () => true })
      .overrideGuard(RolesGuard)
      .useValue({ canActivate: () => true })
      .compile();

    controller = module.get<UsersController>(UsersController);
    service = module.get<UsersService>(UsersService);
  });

  it('should be defined', () => {
    expect(controller).toBeDefined();
  });

  it('should call service.findAll', async () => {
    await controller.findAll();
    expect(service.findAll).toHaveBeenCalled();
  });
});
```

**Interview Tip**: Test Guards by calling **`canActivate()`** directly with **mocked ExecutionContext**. Test **authorization success**, **missing tokens**, **invalid tokens**, **role checks**. Mock **dependencies** like JwtService, ConfigService. Test **async guards** with async/await. Use **`overrideGuard()`** in controller tests to bypass guards.

</details>

### 32. How do you test Pipes?

<details>
<summary>Answer</summary>

**Test Pipes** by calling `transform()` directly with test values and verifying the transformed output or exceptions.

**Testing Validation Pipe:**

```typescript
// validation.pipe.spec.ts
import { ValidationPipe, BadRequestException } from '@nestjs/common';
import { IsString, IsInt, Min, Max } from 'class-validator';
import { plainToInstance } from 'class-transformer';

class CreateUserDto {
  @IsString()
  email: string;

  @IsString()
  @Min(6)
  password: string;

  @IsInt()
  @Min(18)
  @Max(100)
  age: number;
}

describe('ValidationPipe', () => {
  let pipe: ValidationPipe;

  beforeEach(() => {
    pipe = new ValidationPipe({
      whitelist: true,
      forbidNonWhitelisted: true,
      transform: true,
    });
  });

  it('should pass valid data', async () => {
    const dto = {
      email: 'test@example.com',
      password: 'password123',
      age: 25,
    };

    const result = await pipe.transform(dto, {
      type: 'body',
      metatype: CreateUserDto,
    });

    expect(result).toEqual(dto);
  });

  it('should throw error for invalid email', async () => {
    const dto = {
      email: 'invalid-email',
      password: 'password123',
      age: 25,
    };

    await expect(
      pipe.transform(dto, {
        type: 'body',
        metatype: CreateUserDto,
      }),
    ).rejects.toThrow(BadRequestException);
  });

  it('should throw error for missing required field', async () => {
    const dto = {
      email: 'test@example.com',
      // Missing password
      age: 25,
    };

    await expect(
      pipe.transform(dto, {
        type: 'body',
        metatype: CreateUserDto,
      }),
    ).rejects.toThrow(BadRequestException);
  });

  it('should strip non-whitelisted properties', async () => {
    const dto = {
      email: 'test@example.com',
      password: 'password123',
      age: 25,
      extraField: 'should be removed',
    };

    const result = await pipe.transform(dto, {
      type: 'body',
      metatype: CreateUserDto,
    });

    expect(result).not.toHaveProperty('extraField');
  });
});
```

**Testing Custom Transformation Pipe:**

```typescript
// parse-int.pipe.spec.ts
import { ParseIntPipe, BadRequestException } from '@nestjs/common';

describe('ParseIntPipe', () => {
  let pipe: ParseIntPipe;

  beforeEach(() => {
    pipe = new ParseIntPipe();
  });

  it('should transform string to integer', async () => {
    const result = await pipe.transform('42', {
      type: 'param',
      metatype: Number,
      data: 'id',
    });

    expect(result).toBe(42);
    expect(typeof result).toBe('number');
  });

  it('should throw BadRequestException for invalid number', async () => {
    await expect(
      pipe.transform('abc', {
        type: 'param',
        metatype: Number,
        data: 'id',
      }),
    ).rejects.toThrow(BadRequestException);
  });

  it('should throw BadRequestException for empty string', async () => {
    await expect(
      pipe.transform('', {
        type: 'param',
        metatype: Number,
        data: 'id',
      }),
    ).rejects.toThrow(BadRequestException);
  });

  it('should handle negative numbers', async () => {
    const result = await pipe.transform('-10', {
      type: 'param',
      metatype: Number,
      data: 'id',
    });

    expect(result).toBe(-10);
  });
});
```

**Testing Custom Business Logic Pipe:**

```typescript
// uppercase.pipe.spec.ts
import { UppercasePipe } from './uppercase.pipe';
import { ArgumentMetadata } from '@nestjs/common';

describe('UppercasePipe', () => {
  let pipe: UppercasePipe;

  beforeEach(() => {
    pipe = new UppercasePipe();
  });

  it('should transform string to uppercase', () => {
    const result = pipe.transform('hello world', {} as ArgumentMetadata);
    expect(result).toBe('HELLO WORLD');
  });

  it('should handle empty string', () => {
    const result = pipe.transform('', {} as ArgumentMetadata);
    expect(result).toBe('');
  });

  it('should handle already uppercase string', () => {
    const result = pipe.transform('ALREADY UPPER', {} as ArgumentMetadata);
    expect(result).toBe('ALREADY UPPER');
  });

  it('should return non-string values unchanged', () => {
    const result = pipe.transform(123, {} as ArgumentMetadata);
    expect(result).toBe(123);
  });
});
```

**Testing Pipe with Dependencies:**

```typescript
// sanitize.pipe.spec.ts
import { SanitizePipe } from './sanitize.pipe';
import { ConfigService } from '@nestjs/config';

describe('SanitizePipe', () => {
  let pipe: SanitizePipe;
  let configService: ConfigService;

  beforeEach(() => {
    configService = {
      get: jest.fn().mockReturnValue(true),
    } as any;

    pipe = new SanitizePipe(configService);
  });

  it('should sanitize HTML when enabled', () => {
    configService.get = jest.fn().mockReturnValue(true);
    
    const input = '<script>alert("xss")</script><p>Safe content</p>';
    const result = pipe.transform(input, {} as any);

    expect(result).not.toContain('<script>');
    expect(result).toContain('Safe content');
  });

  it('should not sanitize when disabled', () => {
    configService.get = jest.fn().mockReturnValue(false);
    
    const input = '<script>alert("xss")</script>';
    const result = pipe.transform(input, {} as any);

    expect(result).toBe(input);
  });
});
```

**Testing Pipe in Controller:**

```typescript
// users.controller.spec.ts
import { Test } from '@nestjs/testing';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';
import { ParseIntPipe } from '@nestjs/common';

describe('UsersController with Pipes', () => {
  let controller: UsersController;
  let service: UsersService;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      controllers: [UsersController],
      providers: [
        {
          provide: UsersService,
          useValue: {
            findOne: jest.fn().mockResolvedValue({ id: 1, name: 'Test' }),
          },
        },
      ],
    }).compile();

    controller = module.get<UsersController>(UsersController);
    service = module.get<UsersService>(UsersService);
  });

  it('should transform string id to number', async () => {
    const result = await controller.findOne('42');
    
    expect(service.findOne).toHaveBeenCalledWith(42);
    expect(result).toEqual({ id: 1, name: 'Test' });
  });
});
```

**Interview Tip**: Test Pipes by calling **`transform()`** directly with **test values** and **ArgumentMetadata**. Test **valid transformations**, **validation errors**, **edge cases** (empty, null, invalid types). Test **custom business logic**, **dependencies** (ConfigService), and **error throwing**. Pipes should be **pure functions** - easy to test!

</details>

### 33. How do you test Interceptors?

<details>
<summary>Answer</summary>

**Test Interceptors** by calling `intercept()` with mocked `ExecutionContext` and `CallHandler`, then verifying the transformed observable.

**Testing Logging Interceptor:**

```typescript
// logging.interceptor.spec.ts
import { LoggingInterceptor } from './logging.interceptor';
import { ExecutionContext, CallHandler } from '@nestjs/common';
import { of } from 'rxjs';

describe('LoggingInterceptor', () => {
  let interceptor: LoggingInterceptor;
  let mockExecutionContext: ExecutionContext;
  let mockCallHandler: CallHandler;

  beforeEach(() => {
    interceptor = new LoggingInterceptor();

    mockExecutionContext = {
      getClass: jest.fn().mockReturnValue({ name: 'TestController' }),
      getHandler: jest.fn().mockReturnValue({ name: 'testMethod' }),
      switchToHttp: jest.fn().mockReturnValue({
        getRequest: jest.fn().mockReturnValue({
          method: 'GET',
          url: '/test',
        }),
      }),
    } as any;

    mockCallHandler = {
      handle: jest.fn().mockReturnValue(of({ data: 'test' })),
    };
  });

  it('should log before and after request', (done) => {
    const consoleSpy = jest.spyOn(console, 'log').mockImplementation();

    interceptor.intercept(mockExecutionContext, mockCallHandler).subscribe({
      next: (value) => {
        expect(value).toEqual({ data: 'test' });
        expect(consoleSpy).toHaveBeenCalledWith(
          expect.stringContaining('Before'),
        );
        expect(consoleSpy).toHaveBeenCalledWith(
          expect.stringContaining('After'),
        );
        consoleSpy.mockRestore();
        done();
      },
    });
  });

  it('should call next handler', (done) => {
    interceptor.intercept(mockExecutionContext, mockCallHandler).subscribe({
      next: () => {
        expect(mockCallHandler.handle).toHaveBeenCalled();
        done();
      },
    });
  });

  it('should measure execution time', (done) => {
    const consoleSpy = jest.spyOn(console, 'log').mockImplementation();

    interceptor.intercept(mockExecutionContext, mockCallHandler).subscribe({
      next: () => {
        const logCalls = consoleSpy.mock.calls;
        const afterLog = logCalls.find((call) =>
          call[0].includes('After'),
        );
        expect(afterLog).toBeDefined();
        expect(afterLog[0]).toMatch(/\d+ms/);
        consoleSpy.mockRestore();
        done();
      },
    });
  });
});
```

**Testing Transform Interceptor:**

```typescript
// transform.interceptor.spec.ts
import { TransformInterceptor } from './transform.interceptor';
import { ExecutionContext, CallHandler } from '@nestjs/common';
import { of } from 'rxjs';

describe('TransformInterceptor', () => {
  let interceptor: TransformInterceptor;
  let mockExecutionContext: ExecutionContext;
  let mockCallHandler: CallHandler;

  beforeEach(() => {
    interceptor = new TransformInterceptor();
    mockExecutionContext = {} as ExecutionContext;
  });

  it('should wrap response in data property', (done) => {
    mockCallHandler = {
      handle: jest.fn().mockReturnValue(of({ id: 1, name: 'Test' })),
    };

    interceptor.intercept(mockExecutionContext, mockCallHandler).subscribe({
      next: (value) => {
        expect(value).toEqual({
          data: { id: 1, name: 'Test' },
          statusCode: 200,
          message: 'Success',
        });
        done();
      },
    });
  });

  it('should handle array responses', (done) => {
    mockCallHandler = {
      handle: jest.fn().mockReturnValue(of([1, 2, 3])),
    };

    interceptor.intercept(mockExecutionContext, mockCallHandler).subscribe({
      next: (value) => {
        expect(value.data).toEqual([1, 2, 3]);
        expect(Array.isArray(value.data)).toBe(true);
        done();
      },
    });
  });

  it('should handle null responses', (done) => {
    mockCallHandler = {
      handle: jest.fn().mockReturnValue(of(null)),
    };

    interceptor.intercept(mockExecutionContext, mockCallHandler).subscribe({
      next: (value) => {
        expect(value.data).toBeNull();
        done();
      },
    });
  });
});
```

**Testing Cache Interceptor:**

```typescript
// cache.interceptor.spec.ts
import { CacheInterceptor } from './cache.interceptor';
import { ExecutionContext, CallHandler } from '@nestjs/common';
import { of } from 'rxjs';

describe('CacheInterceptor', () => {
  let interceptor: CacheInterceptor;
  let mockCache: Map<string, any>;

  beforeEach(() => {
    mockCache = new Map();
    interceptor = new CacheInterceptor(mockCache as any);
  });

  it('should return cached value if exists', (done) => {
    const cacheKey = 'test-key';
    const cachedValue = { cached: true };
    mockCache.set(cacheKey, cachedValue);

    const mockContext = createMockContext('/test');
    const mockHandler: CallHandler = {
      handle: jest.fn().mockReturnValue(of({ fresh: true })),
    };

    interceptor.intercept(mockContext, mockHandler).subscribe({
      next: (value) => {
        expect(value).toEqual(cachedValue);
        expect(mockHandler.handle).not.toHaveBeenCalled();
        done();
      },
    });
  });

  it('should fetch and cache if not in cache', (done) => {
    const mockContext = createMockContext('/test');
    const freshValue = { fresh: true };
    const mockHandler: CallHandler = {
      handle: jest.fn().mockReturnValue(of(freshValue)),
    };

    interceptor.intercept(mockContext, mockHandler).subscribe({
      next: (value) => {
        expect(value).toEqual(freshValue);
        expect(mockHandler.handle).toHaveBeenCalled();
        expect(mockCache.get('/test')).toEqual(freshValue);
        done();
      },
    });
  });
});

function createMockContext(url: string): ExecutionContext {
  return {
    switchToHttp: () => ({
      getRequest: () => ({ url }),
    }),
  } as any;
}
```

**Testing Error Handling Interceptor:**

```typescript
// error-handling.interceptor.spec.ts
import { ErrorHandlingInterceptor } from './error-handling.interceptor';
import { ExecutionContext, CallHandler } from '@nestjs/common';
import { of, throwError } from 'rxjs';

describe('ErrorHandlingInterceptor', () => {
  let interceptor: ErrorHandlingInterceptor;

  beforeEach(() => {
    interceptor = new ErrorHandlingInterceptor();
  });

  it('should pass through successful responses', (done) => {
    const mockContext = {} as ExecutionContext;
    const mockHandler: CallHandler = {
      handle: () => of({ success: true }),
    };

    interceptor.intercept(mockContext, mockHandler).subscribe({
      next: (value) => {
        expect(value).toEqual({ success: true });
        done();
      },
    });
  });

  it('should catch and transform errors', (done) => {
    const mockContext = {} as ExecutionContext;
    const error = new Error('Test error');
    const mockHandler: CallHandler = {
      handle: () => throwError(() => error),
    };

    interceptor.intercept(mockContext, mockHandler).subscribe({
      next: (value) => {
        expect(value).toEqual({
          error: 'Test error',
          statusCode: 500,
        });
        done();
      },
      error: () => {
        // Should not reach here
        done.fail('Should not throw error');
      },
    });
  });

  it('should log errors', (done) => {
    const consoleSpy = jest.spyOn(console, 'error').mockImplementation();
    const mockContext = {} as ExecutionContext;
    const error = new Error('Test error');
    const mockHandler: CallHandler = {
      handle: () => throwError(() => error),
    };

    interceptor.intercept(mockContext, mockHandler).subscribe({
      next: () => {
        expect(consoleSpy).toHaveBeenCalledWith(
          expect.stringContaining('Test error'),
        );
        consoleSpy.mockRestore();
        done();
      },
    });
  });
});
```

**Interview Tip**: Test Interceptors by calling **`intercept()`** with **mocked ExecutionContext and CallHandler**. Use **RxJS operators** (`of()`, `throwError()`) to simulate responses. Test **transformation logic**, **caching**, **error handling**, **logging**. Subscribe to the returned observable and verify results in **`next` callback**. Use **`done()`** for async tests.

</details>

### 34. How do you mock `ExecutionContext` for Guard tests?

<details>
<summary>Answer</summary>

**Mock `ExecutionContext`** by creating an object that implements the ExecutionContext interface with jest mocked methods.

**Basic ExecutionContext Mock:**

```typescript
// test-utils/execution-context.mock.ts
import { ExecutionContext } from '@nestjs/common';

export function createMockExecutionContext(
  request: any = {},
  response: any = {},
): ExecutionContext {
  return {
    switchToHttp: jest.fn().mockReturnValue({
      getRequest: jest.fn().mockReturnValue(request),
      getResponse: jest.fn().mockReturnValue(response),
      getNext: jest.fn(),
    }),
    switchToRpc: jest.fn().mockReturnValue({
      getContext: jest.fn(),
      getData: jest.fn(),
    }),
    switchToWs: jest.fn().mockReturnValue({
      getClient: jest.fn(),
      getData: jest.fn(),
    }),
    getType: jest.fn().mockReturnValue('http'),
    getClass: jest.fn(),
    getHandler: jest.fn(),
    getArgs: jest.fn(),
    getArgByIndex: jest.fn(),
  } as ExecutionContext;
}
```

**Using Mock in Guard Tests:**

```typescript
// auth.guard.spec.ts
import { AuthGuard } from './auth.guard';
import { UnauthorizedException } from '@nestjs/common';
import { createMockExecutionContext } from '../test-utils/execution-context.mock';

describe('AuthGuard with Mock ExecutionContext', () => {
  let guard: AuthGuard;

  beforeEach(() => {
    guard = new AuthGuard();
  });

  it('should extract token from Authorization header', () => {
    const mockRequest = {
      headers: {
        authorization: 'Bearer test-token-123',
      },
    };

    const context = createMockExecutionContext(mockRequest);

    const result = guard.canActivate(context);

    expect(context.switchToHttp).toHaveBeenCalled();
    expect(result).toBe(true);
  });

  it('should throw UnauthorizedException when no authorization header', () => {
    const mockRequest = {
      headers: {},
    };

    const context = createMockExecutionContext(mockRequest);

    expect(() => guard.canActivate(context)).toThrow(UnauthorizedException);
  });
});
```

**Advanced ExecutionContext Mock with Request/Response:**

```typescript
// advanced-execution-context.mock.ts
import { ExecutionContext, HttpArgumentsHost } from '@nestjs/common';

export interface MockExecutionContextOptions {
  request?: Partial<Request>;
  response?: Partial<Response>;
  user?: any;
  headers?: Record<string, string>;
  body?: any;
  params?: any;
  query?: any;
}

export function createAdvancedMockContext(
  options: MockExecutionContextOptions = {},
): ExecutionContext {
  const mockRequest: any = {
    headers: options.headers || {},
    body: options.body || {},
    params: options.params || {},
    query: options.query || {},
    user: options.user,
    ...options.request,
  };

  const mockResponse: any = {
    status: jest.fn().mockReturnThis(),
    json: jest.fn().mockReturnThis(),
    send: jest.fn().mockReturnThis(),
    ...options.response,
  };

  const httpArgumentsHost: HttpArgumentsHost = {
    getRequest: jest.fn().mockReturnValue(mockRequest),
    getResponse: jest.fn().mockReturnValue(mockResponse),
    getNext: jest.fn(),
  };

  return {
    switchToHttp: jest.fn().mockReturnValue(httpArgumentsHost),
    switchToRpc: jest.fn(),
    switchToWs: jest.fn(),
    getType: jest.fn().mockReturnValue('http'),
    getClass: jest.fn().mockReturnValue({ name: 'TestController' }),
    getHandler: jest.fn().mockReturnValue({ name: 'testMethod' }),
    getArgs: jest.fn().mockReturnValue([mockRequest, mockResponse]),
    getArgByIndex: jest.fn((index: number) => [mockRequest, mockResponse][index]),
  } as ExecutionContext;
}
```

**Using Advanced Mock:**

```typescript
// roles.guard.spec.ts
import { RolesGuard } from './roles.guard';
import { Reflector } from '@nestjs/core';
import { createAdvancedMockContext } from '../test-utils/execution-context.mock';

describe('RolesGuard with Advanced Mock', () => {
  let guard: RolesGuard;
  let reflector: Reflector;

  beforeEach(() => {
    reflector = new Reflector();
    guard = new RolesGuard(reflector);
  });

  it('should allow admin user to access admin route', () => {
    jest.spyOn(reflector, 'getAllAndOverride').mockReturnValue(['admin']);

    const context = createAdvancedMockContext({
      user: { id: 1, email: 'admin@example.com', role: 'admin' },
      headers: { authorization: 'Bearer admin-token' },
    });

    const result = guard.canActivate(context);

    expect(result).toBe(true);
  });

  it('should deny regular user access to admin route', () => {
    jest.spyOn(reflector, 'getAllAndOverride').mockReturnValue(['admin']);

    const context = createAdvancedMockContext({
      user: { id: 2, email: 'user@example.com', role: 'user' },
    });

    const result = guard.canActivate(context);

    expect(result).toBe(false);
  });
});
```

**Mock for WebSocket Context:**

```typescript
// ws-execution-context.mock.ts
import { ExecutionContext } from '@nestjs/common';

export function createWsExecutionContext(client: any, data: any): ExecutionContext {
  return {
    switchToWs: jest.fn().mockReturnValue({
      getClient: jest.fn().mockReturnValue(client),
      getData: jest.fn().mockReturnValue(data),
    }),
    switchToHttp: jest.fn(),
    switchToRpc: jest.fn(),
    getType: jest.fn().mockReturnValue('ws'),
    getClass: jest.fn(),
    getHandler: jest.fn(),
    getArgs: jest.fn(),
    getArgByIndex: jest.fn(),
  } as ExecutionContext;
}

// Usage
it('should validate WebSocket connection', () => {
  const mockClient = { handshake: { headers: { authorization: 'Bearer token' } } };
  const context = createWsExecutionContext(mockClient, { message: 'test' });

  const result = guard.canActivate(context);
  expect(result).toBe(true);
});
```

**Mock for GraphQL Context:**

```typescript
// graphql-execution-context.mock.ts
import { ExecutionContext } from '@nestjs/common';
import { GqlExecutionContext } from '@nestjs/graphql';

export function createGqlExecutionContext(context: any): ExecutionContext {
  const mockContext = {
    getContext: jest.fn().mockReturnValue(context),
    getArgs: jest.fn().mockReturnValue([{}, context.variables, context, {}]),
  };

  return {
    switchToHttp: jest.fn(),
    switchToRpc: jest.fn(),
    switchToWs: jest.fn(),
    getType: jest.fn().mockReturnValue('graphql'),
    getClass: jest.fn(),
    getHandler: jest.fn(),
    getArgs: jest.fn().mockReturnValue([{}, context.variables, context, {}]),
    getArgByIndex: jest.fn(),
  } as ExecutionContext;
}

// Usage
it('should validate GraphQL request', () => {
  const context = createGqlExecutionContext({
    req: {
      headers: { authorization: 'Bearer token' },
      user: { id: 1 },
    },
    variables: { id: 1 },
  });

  const gqlContext = GqlExecutionContext.create(context);
  const request = gqlContext.getContext().req;

  expect(request.user.id).toBe(1);
});
```

**Interview Tip**: Mock ExecutionContext by creating **helper factory functions** that return objects implementing the interface. Mock **`switchToHttp()`**, **`getRequest()`**, **`getResponse()`** for HTTP. Mock **`switchToWs()`** for WebSockets. Create **reusable mock utilities** with options for headers, user, body, params. Test different **context types** (http, ws, rpc, graphql).

</details>

## Test Coverage

### 35. How do you measure test coverage using Jest?

<details>
<summary>Answer</summary>

**Measure test coverage** using Jest's built-in `--coverage` flag, which generates detailed coverage reports.

**Running Coverage:**

```bash
# Run all tests with coverage
npm test -- --coverage

# Run specific test file with coverage
npm test users.service.spec.ts -- --coverage

# Run coverage with watch mode
npm test -- --coverage --watch

# Generate coverage without running tests (if previous data exists)
jest --coverage --collectCoverageFrom='src/**/*.ts'
```

**Jest Configuration for Coverage:**

```javascript
// jest.config.js
module.exports = {
  moduleFileExtensions: ['js', 'json', 'ts'],
  rootDir: 'src',
  testRegex: '.*\\.spec\\.ts$',
  transform: {
    '^.+\\.(t|j)s$': 'ts-jest',
  },
  collectCoverageFrom: [
    '**/*.(t|j)s',
    '!**/*.spec.ts',
    '!**/*.module.ts',
    '!**/node_modules/**',
    '!**/dist/**',
    '!**/main.ts',
  ],
  coverageDirectory: '../coverage',
  testEnvironment: 'node',
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
  coverageReporters: ['text', 'lcov', 'html', 'json'],
};
```

**Package.json Scripts:**

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:cov": "jest --coverage",
    "test:cov:watch": "jest --coverage --watch",
    "test:debug": "node --inspect-brk -r tsconfig-paths/register -r ts-node/register node_modules/.bin/jest --runInBand"
  }
}
```

**Understanding Coverage Metrics:**

```
Coverage Metrics:
┌─────────────┬────────┬──────────┬─────────┬──────────┐
│   Type      │ % Stmts│ % Branch │ % Funcs │ % Lines  │
├─────────────┼────────┼──────────┼─────────┼──────────┤
│ Statements  │  85.5  │    -     │    -    │    -     │
│ Branches    │   -    │   78.2   │    -    │    -     │
│ Functions   │   -    │    -     │  90.1   │    -     │
│ Lines       │   -    │    -     │    -    │  86.3    │
└─────────────┴────────┴──────────┴─────────┴──────────┘

**Statements**: % of code statements executed
**Branches**: % of if/else, switch branches covered
**Functions**: % of functions called
**Lines**: % of executable lines run
```

**Coverage Report Example:**

```typescript
// users.service.ts - Before testing
export class UsersService {
  async findAll(): Promise<User[]> {
    return this.userRepository.find();
  }

  async findOne(id: number): Promise<User> {
    const user = await this.userRepository.findOne({ where: { id } });
    
    if (!user) {
      throw new NotFoundException('User not found');
    }
    
    return user;
  }

  async create(createUserDto: CreateUserDto): Promise<User> {
    const user = this.userRepository.create(createUserDto);
    return this.userRepository.save(user);
  }
}

// Coverage report shows:
// ✓ findAll() - 100% covered
// ✓ findOne() - 100% covered (both branches: found and not found)
// ✗ create() - 0% covered (not tested)
```

**Viewing Coverage in Different Formats:**

```bash
# Text output in terminal
npm test -- --coverage

# HTML report (open coverage/index.html in browser)
npm test -- --coverage --coverageReporters=html

# LCOV format (for CI/CD tools)
npm test -- --coverage --coverageReporters=lcov

# JSON format (for programmatic access)
npm test -- --coverage --coverageReporters=json
```

**Coverage Thresholds:**

```javascript
// jest.config.js
module.exports = {
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
    './src/users/**/*.ts': {
      branches: 90,
      functions: 90,
      lines: 90,
      statements: 90,
    },
    './src/core/**/*.ts': {
      branches: 100,
      functions: 100,
      lines: 100,
      statements: 100,
    },
  },
};

// If coverage falls below thresholds, tests fail
```

**Ignoring Code from Coverage:**

```typescript
// Using Istanbul comments
export class UsersService {
  async findOne(id: number): Promise<User> {
    /* istanbul ignore next */
    if (process.env.NODE_ENV === 'development') {
      console.log('Debug: Finding user', id);
    }

    return this.userRepository.findOne({ where: { id } });
  }

  /* istanbul ignore next */
  private debugLog(message: string): void {
    // This entire function ignored from coverage
    console.log(message);
  }
}
```

**CI/CD Integration:**

```yaml
# .github/workflows/test.yml
name: Test with Coverage

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests with coverage
        run: npm run test:cov
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info
          fail_ci_if_error: true
```

**Interview Tip**: Run coverage with **`jest --coverage`**. Configure **`collectCoverageFrom`** to include source files and exclude specs. Set **`coverageThreshold`** to enforce minimum coverage. Use **`coverageReporters`** for different formats (text, html, lcov). Coverage measures: **statements** (code executed), **branches** (if/else paths), **functions** (called functions), **lines** (executed lines). View HTML report for detailed visualization.

</details>

### 36. What is a good test coverage percentage?

<details>
<summary>Answer</summary>

**Good test coverage** depends on the project, but generally aim for **80-90%** overall with **critical paths at 100%**.

**Coverage Targets by Component:**

| Component Type | Target Coverage | Rationale |
|----------------|----------------|-----------|
| **Business Logic** | 90-100% | Critical functionality |
| **Services** | 85-95% | Core application logic |
| **Controllers** | 70-85% | Routing and validation |
| **Guards** | 90-100% | Security critical |
| **Pipes** | 90-100% | Data validation |
| **Interceptors** | 85-95% | Cross-cutting concerns |
| **Utilities** | 90-100% | Reusable functions |
| **Entities** | 50-70% | Simple DTOs |
| **Modules** | 60-70% | Configuration only |

**Recommended Thresholds:**

```javascript
// jest.config.js - Recommended configuration
module.exports = {
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
    // Critical business logic - higher threshold
    './src/services/**/*.ts': {
      branches: 90,
      functions: 90,
      lines: 90,
      statements: 90,
    },
    // Security components - highest threshold
    './src/guards/**/*.ts': {
      branches: 95,
      functions: 95,
      lines: 95,
      statements: 95,
    },
    './src/auth/**/*.ts': {
      branches: 95,
      functions: 95,
      lines: 95,
      statements: 95,
    },
    // Controllers - moderate threshold
    './src/**/*.controller.ts': {
      branches: 70,
      functions: 75,
      lines: 75,
      statements: 75,
    },
  },
};
```

**Coverage Quality > Coverage Quantity:**

```typescript
// ❌ BAD: 100% coverage but poor tests
describe('UsersService', () => {
  it('should have findAll method', () => {
    expect(service.findAll).toBeDefined(); // Useless test
  });

  it('should call repository', async () => {
    await service.findAll();
    expect(mockRepository.find).toHaveBeenCalled(); // Only tests mock was called
  });
});

// ✅ GOOD: Lower coverage but meaningful tests
describe('UsersService', () => {
  it('should return all users from repository', async () => {
    const expectedUsers = [{ id: 1, email: 'test@example.com' }];
    mockRepository.find.mockResolvedValue(expectedUsers);

    const result = await service.findAll();

    expect(result).toEqual(expectedUsers);
    expect(result).toHaveLength(1);
    expect(result[0]).toHaveProperty('email');
  });

  it('should throw error when repository fails', async () => {
    mockRepository.find.mockRejectedValue(new Error('DB error'));

    await expect(service.findAll()).rejects.toThrow('DB error');
  });
});
```

**What to Focus On:**

```typescript
// ✅ MUST have 100% coverage:
// 1. Payment processing
class PaymentService {
  async processPayment(amount: number, cardToken: string) {
    if (amount <= 0) throw new Error('Invalid amount');
    if (!cardToken) throw new Error('Card required');
    
    // Must test: valid payment, invalid amount, missing card, API failure
    return this.stripeService.charge({ amount, cardToken });
  }
}

// 2. Authentication & Authorization
class AuthService {
  async validateUser(email: string, password: string) {
    // Must test: valid credentials, wrong password, user not found, account locked
  }
}

// 3. Data validation
class ValidationPipe {
  transform(value: any) {
    // Must test: valid data, invalid data, edge cases, type coercion
  }
}

// ✅ OK to have lower coverage:
// 1. Simple DTOs
export class CreateUserDto {
  email: string; // No logic to test
  password: string;
}

// 2. Configuration files
@Module({
  imports: [/* ... */],
  controllers: [/* ... */],
  providers: [/* ... */],
})
export class AppModule {} // No logic to test

// 3. Main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
```

**Coverage Anti-Patterns:**

```typescript
// ❌ DON'T: Chase 100% coverage by testing trivial code
it('should set property', () => {
  const dto = new CreateUserDto();
  dto.email = 'test@example.com';
  expect(dto.email).toBe('test@example.com'); // Waste of time
});

// ❌ DON'T: Write tests just to increase coverage
it('should have constructor', () => {
  expect(new UsersService(mockRepository)).toBeDefined(); // Useless
});

// ❌ DON'T: Ignore branch coverage
it('should find user', async () => {
  const user = await service.findOne(1);
  expect(user).toBeDefined();
  // Missing: test when user not found (if statement not covered)
});

// ✅ DO: Test meaningful behavior
it('should throw NotFoundException when user not found', async () => {
  mockRepository.findOne.mockResolvedValue(null);
  await expect(service.findOne(999)).rejects.toThrow(NotFoundException);
});
```

**Coverage by Project Type:**

```typescript
// Startup/MVP: 60-70%
// - Focus on critical paths
// - Skip DTOs, simple getters/setters
// - Test happy paths only

// Production Application: 80-90%
// - Comprehensive test suite
// - Test happy paths + error scenarios
// - Cover edge cases

// Financial/Healthcare: 95-100%
// - Regulatory requirements
// - Every branch tested
// - Extensive edge case coverage
// - Mutation testing
```

**Interview Tip**: **80-90% overall coverage** is good, but **quality > quantity**. **Critical code (auth, payment, validation)** should have **90-100%** coverage. **Simple code (DTOs, configs)** can have **50-70%**. Focus on **branch coverage** (testing all if/else paths). Don't write tests just to increase numbers - write **meaningful tests** that catch real bugs. Use coverage as a **guide**, not a **goal**.

</details>

### 37. How do you generate coverage reports?

<details>
<summary>Answer</summary>

**Generate coverage reports** using Jest's `--coverage` flag with different reporter formats for various use cases.

**Basic Coverage Reports:**

```bash
# Text report in terminal
npm test -- --coverage

# HTML report (interactive, browser-based)
npm test -- --coverage --coverageReporters=html

# Multiple reporters at once
npm test -- --coverage --coverageReporters=text --coverageReporters=html --coverageReporters=lcov
```

**Jest Configuration for Reports:**

```javascript
// jest.config.js
module.exports = {
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.spec.ts',
    '!src/**/*.module.ts',
    '!src/main.ts',
    '!src/**/*.interface.ts',
    '!src/**/*.dto.ts',
  ],
  coverageDirectory: 'coverage',
  coverageReporters: [
    'text',          // Console output
    'text-summary',  // Brief console summary
    'html',          // Interactive HTML
    'lcov',          // For CI/CD tools
    'json',          // Programmatic access
    'cobertura',     // For Jenkins/GitLab
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
};
```

**Package.json Scripts:**

```json
{
  "scripts": {
    "test": "jest",
    "test:cov": "jest --coverage",
    "test:cov:watch": "jest --coverage --watch",
    "test:cov:html": "jest --coverage --coverageReporters=html && open coverage/index.html",
    "test:cov:lcov": "jest --coverage --coverageReporters=lcov",
    "test:cov:json": "jest --coverage --coverageReporters=json",
    "test:cov:text": "jest --coverage --coverageReporters=text-summary"
  }
}
```

**Coverage Report Types:**

**1. Text Report (Console):**
```bash
npm test -- --coverage --coverageReporters=text

# Output:
--------------------------|---------|----------|---------|---------|
File                      | % Stmts | % Branch | % Funcs | % Lines |
--------------------------|---------|----------|---------|---------|
All files                 |   85.5  |   78.2   |  90.1   |  86.3   |
 src/users                |   92.3  |   87.5   |  95.0   |  93.1   |
  users.controller.ts     |   88.9  |   80.0   |  90.0   |  89.5   |
  users.service.ts        |   95.2  |   92.3   |  100    |  96.0   |
 src/auth                 |   78.5  |   65.0   |  85.0   |  79.8   |
  auth.controller.ts      |   75.0  |   60.0   |  80.0   |  76.5   |
  auth.service.ts         |   82.0  |   70.0   |  90.0   |  83.1   |
--------------------------|---------|----------|---------|---------|
```

**2. HTML Report (Interactive):**
```bash
npm test -- --coverage --coverageReporters=html

# Opens coverage/index.html in browser
# Shows:
# - File-by-file breakdown
# - Line-by-line highlighting (green = covered, red = not covered)
# - Branch coverage visualization
# - Click through to see uncovered code
```

**3. LCOV Report (CI/CD):**
```bash
npm test -- --coverage --coverageReporters=lcov

# Generates coverage/lcov.info
# Used by: Codecov, Coveralls, SonarQube, GitHub Actions
```

**4. JSON Report (Programmatic):**
```bash
npm test -- --coverage --coverageReporters=json

# Generates coverage/coverage-final.json
# Can be parsed by custom scripts
```

**HTML Report Structure:**

```
coverage/
├── index.html              # Main page with summary
├── lcov-report/
│   ├── index.html         # Alternative HTML format
│   ├── users/
│   │   ├── index.html
│   │   ├── users.service.ts.html
│   │   └── users.controller.ts.html
│   └── auth/
│       ├── index.html
│       └── auth.service.ts.html
├── coverage-final.json    # JSON data
└── lcov.info             # LCOV format
```

**CI/CD Integration Examples:**

**GitHub Actions:**
```yaml
# .github/workflows/test.yml
name: Tests with Coverage

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests with coverage
        run: npm run test:cov
      
      - name: Upload to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info
          flags: unittests
          name: codecov-umbrella
      
      - name: Upload coverage artifact
        uses: actions/upload-artifact@v3
        with:
          name: coverage-report
          path: coverage/
```

**GitLab CI:**
```yaml
# .gitlab-ci.yml
test:
  stage: test
  script:
    - npm ci
    - npm run test:cov
  coverage: '/All files[^|]*\|[^|]*\s+([\d\.]+)/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
    paths:
      - coverage/
    expire_in: 30 days
```

**Custom Coverage Report Script:**

```typescript
// scripts/coverage-report.ts
import * as fs from 'fs';
import * as path from 'path';

const coverageFile = path.join(__dirname, '../coverage/coverage-final.json');
const coverage = JSON.parse(fs.readFileSync(coverageFile, 'utf8'));

let totalStatements = 0;
let coveredStatements = 0;

for (const file in coverage) {
  const fileCoverage = coverage[file];
  const statements = fileCoverage.s;
  
  totalStatements += Object.keys(statements).length;
  coveredStatements += Object.values(statements).filter((count: any) => count > 0).length;
}

const percentage = (coveredStatements / totalStatements) * 100;

console.log(`Coverage: ${percentage.toFixed(2)}%`);
console.log(`Statements: ${coveredStatements}/${totalStatements}`);

if (percentage < 80) {
  console.error('Coverage below 80%!');
  process.exit(1);
}
```

**Viewing Reports:**

```bash
# Open HTML report in browser (macOS)
open coverage/index.html

# Open HTML report (Windows)
start coverage/index.html

# Open HTML report (Linux)
xdg-open coverage/index.html

# Serve HTML report with live server
npx http-server coverage -o
```

**Coverage Badges:**

```markdown
<!-- README.md with Codecov badge -->
# My NestJS Project

[![codecov](https://codecov.io/gh/username/repo/branch/main/graph/badge.svg)](https://codecov.io/gh/username/repo)

## Coverage Report

![Coverage](https://img.shields.io/badge/coverage-85%25-green)
```

**Interview Tip**: Generate reports with **`jest --coverage --coverageReporters=<type>`**. Use **`text`** for terminal output, **`html`** for interactive browsing, **`lcov`** for CI/CD tools, **`json`** for custom scripts. Configure **`coverageReporters`** in jest.config.js for multiple formats. HTML reports show **line-by-line** uncovered code. LCOV integrates with **Codecov, Coveralls, SonarQube**. Upload coverage as **CI artifacts** for historical tracking.

</details>

## Best Practices

### 38. Should you test private methods?

<details>
<summary>Answer</summary>

**No, don't test private methods directly.** Test them **indirectly** through public methods that use them.

**Why Not Test Private Methods:**

| Reason | Explanation |
|--------|-------------|
| **Implementation Detail** | Private methods are internal, can change anytime |
| **Breaks Encapsulation** | Tests shouldn't know about internals |
| **Brittle Tests** | Tests break when refactoring private code |
| **Public API Focus** | Users only care about public behavior |
| **Test What Matters** | Test outcomes, not implementation |

**❌ BAD: Testing Private Methods Directly:**

```typescript
// users.service.ts
export class UsersService {
  private hashPassword(password: string): string {
    return bcrypt.hashSync(password, 10);
  }

  async create(createUserDto: CreateUserDto): Promise<User> {
    const hashedPassword = this.hashPassword(createUserDto.password);
    const user = this.userRepository.create({
      ...createUserDto,
      password: hashedPassword,
    });
    return this.userRepository.save(user);
  }
}

// users.service.spec.ts - ❌ BAD APPROACH
describe('UsersService', () => {
  it('should hash password correctly', () => {
    // Accessing private method - WRONG!
    const result = service['hashPassword']('password123');
    expect(result).toMatch(/^\$2[ayb]\$.{56}$/);
  });
});
```

**✅ GOOD: Testing Private Methods Indirectly:**

```typescript
// users.service.spec.ts - ✅ GOOD APPROACH
describe('UsersService', () => {
  describe('create', () => {
    it('should hash password when creating user', async () => {
      const createUserDto = {
        email: 'test@example.com',
        password: 'plainPassword123',
      };

      const user = await service.create(createUserDto);

      // Test the outcome: password is hashed
      expect(user.password).not.toBe('plainPassword123');
      expect(user.password).toMatch(/^\$2[ayb]\$.{56}$/);
      
      // Verify hashed password works
      const isValid = await bcrypt.compare('plainPassword123', user.password);
      expect(isValid).toBe(true);
    });
  });
});
```

**When Private Methods Are Complex:**

```typescript
// ❌ If private method is very complex, you might be tempted to test it
export class OrdersService {
  // Complex private method
  private calculateTotalWithDiscounts(
    items: OrderItem[],
    couponCode?: string,
    loyaltyPoints?: number,
  ): number {
    // 50 lines of complex calculation logic...
    let total = items.reduce((sum, item) => sum + item.price * item.quantity, 0);
    
    if (couponCode) {
      total -= this.applyCoupon(couponCode, total);
    }
    
    if (loyaltyPoints) {
      total -= this.applyLoyaltyPoints(loyaltyPoints);
    }
    
    return total;
  }

  async createOrder(createOrderDto: CreateOrderDto): Promise<Order> {
    const total = this.calculateTotalWithDiscounts(
      createOrderDto.items,
      createOrderDto.couponCode,
      createOrderDto.loyaltyPoints,
    );
    // ... rest of order creation
  }
}

// ✅ SOLUTION 1: Test through public API
describe('OrdersService', () => {
  it('should apply coupon discount when creating order', async () => {
    const order = await service.createOrder({
      items: [{ productId: 1, quantity: 2, price: 50 }],
      couponCode: 'SAVE20',
    });

    expect(order.total).toBe(80); // 100 - 20% discount
  });

  it('should apply loyalty points discount', async () => {
    const order = await service.createOrder({
      items: [{ productId: 1, quantity: 2, price: 50 }],
      loyaltyPoints: 100,
    });

    expect(order.total).toBe(90); // 100 - 10 points
  });

  it('should combine coupon and loyalty discounts', async () => {
    const order = await service.createOrder({
      items: [{ productId: 1, quantity: 2, price: 50 }],
      couponCode: 'SAVE20',
      loyaltyPoints: 100,
    });

    expect(order.total).toBe(70); // 100 - 20% - 10 points
  });
});

// ✅ SOLUTION 2: Extract to separate public utility/service
export class DiscountCalculator {
  calculateTotal(items: OrderItem[], couponCode?: string, loyaltyPoints?: number): number {
    // Now it's public and testable!
  }
}

// Easy to test separately
describe('DiscountCalculator', () => {
  it('should calculate total with discounts', () => {
    const calculator = new DiscountCalculator();
    const total = calculator.calculateTotal(
      [{ productId: 1, quantity: 2, price: 50 }],
      'SAVE20',
      100,
    );
    expect(total).toBe(70);
  });
});
```

**Exceptions (Rare Cases):**

```typescript
// Sometimes testing private methods is acceptable:

// 1. Legacy code with no public interface to test
// 2. Temporary testing during refactoring
// 3. When making private method public would harm API design

// Use TypeScript type casting with caution:
it('should hash password (testing private method as exception)', () => {
  const result = (service as any).hashPassword('test');
  expect(result).toBeDefined();
});

// Or change to protected for testing (suboptimal):
export class UsersService {
  protected hashPassword(password: string): string {
    return bcrypt.hashSync(password, 10);
  }
}

// Create test subclass (also suboptimal):
class TestableUsersService extends UsersService {
  public testHashPassword(password: string): string {
    return this.hashPassword(password);
  }
}
```

**Interview Tip**: **Don't test private methods directly**. Test them **indirectly through public API** that calls them. If private method is too complex to test indirectly, **extract it to a separate public class/utility**. Focus tests on **public behavior and outcomes**, not implementation details. Testing private methods makes tests **brittle** and breaks **encapsulation**.

</details>

### 39. What is AAA pattern (Arrange, Act, Assert)?

<details>
<summary>Answer</summary>

**AAA pattern** (Arrange, Act, Assert) is a **testing structure** that organizes tests into three clear sections for readability and maintainability.

**AAA Pattern Structure:**

```typescript
it('should do something', () => {
  // ARRANGE: Set up test data and dependencies
  const input = 'test';
  const expected = 'expected result';
  
  // ACT: Execute the code being tested
  const result = functionUnderTest(input);
  
  // ASSERT: Verify the result
  expect(result).toBe(expected);
});
```

**Detailed Example:**

```typescript
// users.service.spec.ts
describe('UsersService', () => {
  describe('create', () => {
    it('should create a new user with hashed password', async () => {
      // ============ ARRANGE ============
      // Setup: Create test data
      const createUserDto = {
        email: 'test@example.com',
        password: 'plainPassword123',
        name: 'Test User',
      };

      // Setup: Define expected behavior
      const expectedUser = {
        id: 1,
        email: 'test@example.com',
        password: 'hashedPassword',
        name: 'Test User',
        createdAt: new Date(),
      };

      // Setup: Configure mocks
      mockRepository.create.mockReturnValue(expectedUser);
      mockRepository.save.mockResolvedValue(expectedUser);

      // ============ ACT ============
      // Execute: Call the method being tested
      const result = await service.create(createUserDto);

      // ============ ASSERT ============
      // Verify: Check the results
      expect(result).toEqual(expectedUser);
      expect(result.password).not.toBe(createUserDto.password);
      expect(mockRepository.create).toHaveBeenCalledWith(
        expect.objectContaining({
          email: createUserDto.email,
          name: createUserDto.name,
        }),
      );
      expect(mockRepository.save).toHaveBeenCalledWith(expectedUser);
    });
  });
});
```

**Visual Separation:**

```typescript
describe('OrdersService', () => {
  it('should calculate order total correctly', async () => {
    // --- ARRANGE ---
    const items = [
      { productId: 1, quantity: 2, price: 50 },
      { productId: 2, quantity: 1, price: 30 },
    ];
    const expectedTotal = 130; // (2 * 50) + (1 * 30)

    // --- ACT ---
    const order = await service.createOrder({ items });

    // --- ASSERT ---
    expect(order.total).toBe(expectedTotal);
  });
});
```

**With Comments:**

```typescript
it('should validate and create user', async () => {
  // Arrange: Set up test data and mocks
  const userData = { email: 'test@example.com', password: 'pass123' };
  mockValidator.validate.mockResolvedValue(true);
  mockRepository.save.mockResolvedValue({ id: 1, ...userData });

  // Act: Execute the method
  const result = await service.create(userData);

  // Assert: Verify outcomes
  expect(result.id).toBe(1);
  expect(mockValidator.validate).toHaveBeenCalled();
  expect(mockRepository.save).toHaveBeenCalled();
});
```

**Complex Arrange Section:**

```typescript
describe('PaymentService', () => {
  it('should process payment with discount', async () => {
    // ===== ARRANGE =====
    // Create user
    const user = {
      id: 1,
      email: 'test@example.com',
      loyaltyPoints: 100,
    };

    // Create order
    const order = {
      id: 1,
      userId: user.id,
      total: 200,
    };

    // Create coupon
    const coupon = {
      code: 'SAVE20',
      discountPercent: 20,
    };

    // Mock payment gateway
    mockStripe.charges.create.mockResolvedValue({
      id: 'charge_123',
      status: 'succeeded',
    });

    // Mock coupon service
    mockCouponService.validate.mockResolvedValue(true);
    mockCouponService.calculateDiscount.mockReturnValue(40);

    // Expected result
    const expectedCharge = 160; // 200 - 40 (20% discount)

    // ===== ACT =====
    const result = await paymentService.processPayment(
      order,
      user,
      coupon.code,
    );

    // ===== ASSERT =====
    expect(result.success).toBe(true);
    expect(result.chargedAmount).toBe(expectedCharge);
    expect(mockStripe.charges.create).toHaveBeenCalledWith({
      amount: expectedCharge * 100, // Stripe uses cents
      currency: 'usd',
      customer: user.id,
    });
  });
});
```

**Multiple Assertions:**

```typescript
it('should update user profile', async () => {
  // Arrange
  const userId = 1;
  const updateData = { name: 'New Name', age: 30 };
  const existingUser = { id: 1, name: 'Old Name', age: 25, email: 'test@example.com' };
  const updatedUser = { ...existingUser, ...updateData };
  
  mockRepository.findOne.mockResolvedValue(existingUser);
  mockRepository.save.mockResolvedValue(updatedUser);

  // Act
  const result = await service.update(userId, updateData);

  // Assert
  // Verify result
  expect(result.name).toBe('New Name');
  expect(result.age).toBe(30);
  expect(result.email).toBe('test@example.com'); // Unchanged
  
  // Verify methods called
  expect(mockRepository.findOne).toHaveBeenCalledWith({ where: { id: userId } });
  expect(mockRepository.save).toHaveBeenCalledWith(
    expect.objectContaining(updateData),
  );
});
```

**Given-When-Then Variation (BDD style):**

```typescript
describe('UsersService', () => {
  it('should throw error when user not found', async () => {
    // GIVEN a non-existent user ID
    const userId = 999;
    mockRepository.findOne.mockResolvedValue(null);

    // WHEN we try to find the user
    const findUser = () => service.findOne(userId);

    // THEN it should throw NotFoundException
    await expect(findUser()).rejects.toThrow(NotFoundException);
    await expect(findUser()).rejects.toThrow('User not found');
  });
});
```

**Avoiding Anti-Patterns:**

```typescript
// ❌ BAD: Mixed arrange and act
it('should create user', async () => {
  const dto = { email: 'test@example.com' };
  mockRepository.save.mockResolvedValue({ id: 1 });
  const result = await service.create(dto); // Act mixed with arrange
  mockRepository.create.mockReturnValue({ id: 1 }); // More arrange after act
  expect(result.id).toBe(1);
});

// ✅ GOOD: Clear separation
it('should create user', async () => {
  // Arrange
  const dto = { email: 'test@example.com' };
  mockRepository.create.mockReturnValue({ id: 1 });
  mockRepository.save.mockResolvedValue({ id: 1 });

  // Act
  const result = await service.create(dto);

  // Assert
  expect(result.id).toBe(1);
});

// ❌ BAD: Act in multiple places
it('should update twice', async () => {
  // Arrange
  const dto = { name: 'Name' };
  
  // Act
  const first = await service.update(1, dto);
  
  // Assert (premature)
  expect(first.name).toBe('Name');
  
  // Act again? (confusing)
  const second = await service.update(2, dto);
  
  // Assert
  expect(second.name).toBe('Name');
});

// ✅ GOOD: Split into separate tests
it('should update user 1', async () => {
  // Arrange
  const dto = { name: 'Name' };
  
  // Act
  const result = await service.update(1, dto);
  
  // Assert
  expect(result.name).toBe('Name');
});

it('should update user 2', async () => {
  // Arrange
  const dto = { name: 'Name' };
  
  // Act
  const result = await service.update(2, dto);
  
  // Assert
  expect(result.name).toBe('Name');
});
```

**Interview Tip**: AAA pattern organizes tests into **Arrange** (setup), **Act** (execute), **Assert** (verify). Makes tests **readable**, **maintainable**, and **consistent**. **Arrange**: Create test data, configure mocks. **Act**: Call the method once. **Assert**: Verify results and interactions. Use **blank lines** or **comments** to separate sections. Alternative: **Given-When-Then** (BDD style). Keep tests **focused** - one Act per test.

</details>

### 40. How do you organize test files?

<details>
<summary>Answer</summary>

**Organize test files** by keeping them **next to source files** with `.spec.ts` extension, mirroring the source structure.

**Standard File Organization:**

```
src/
├── users/
│   ├── users.controller.ts
│   ├── users.controller.spec.ts      ← Controller tests
│   ├── users.service.ts
│   ├── users.service.spec.ts         ← Service tests
│   ├── users.module.ts
│   ├── entities/
│   │   └── user.entity.ts
│   ├── dto/
│   │   ├── create-user.dto.ts
│   │   └── update-user.dto.ts
│   └── guards/
│       ├── user-owner.guard.ts
│       └── user-owner.guard.spec.ts  ← Guard tests
├── auth/
│   ├── auth.controller.ts
│   ├── auth.controller.spec.ts
│   ├── auth.service.ts
│   ├── auth.service.spec.ts
│   ├── auth.module.ts
│   └── strategies/
│       ├── jwt.strategy.ts
│       └── jwt.strategy.spec.ts
└── common/
    ├── filters/
    │   ├── http-exception.filter.ts
    │   └── http-exception.filter.spec.ts
    ├── interceptors/
    │   ├── logging.interceptor.ts
    │   └── logging.interceptor.spec.ts
    └── pipes/
        ├── validation.pipe.ts
        └── validation.pipe.spec.ts

test/                                  ← E2E tests separate
├── app.e2e-spec.ts
├── users.e2e-spec.ts
├── auth.e2e-spec.ts
└── jest-e2e.json
```

**Naming Conventions:**

```typescript
// Source file: users.service.ts
// Test file: users.service.spec.ts

// Source file: auth.controller.ts
// Test file: auth.controller.spec.ts

// E2E test: users.e2e-spec.ts
// Integration test: users.integration.spec.ts (if separate)
```

**Test File Structure:**

```typescript
// users.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { UsersService } from './users.service';
import { getRepositoryToken } from '@nestjs/typeorm';
import { User } from './entities/user.entity';

describe('UsersService', () => {
  let service: UsersService;
  let mockRepository: any;

  // Setup
  beforeEach(async () => {
    mockRepository = {
      find: jest.fn(),
      findOne: jest.fn(),
      create: jest.fn(),
      save: jest.fn(),
      remove: jest.fn(),
    };

    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User),
          useValue: mockRepository,
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
  });

  // Group tests by method
  describe('findAll', () => {
    it('should return array of users', async () => {
      // Test implementation
    });

    it('should return empty array when no users', async () => {
      // Test implementation
    });
  });

  describe('findOne', () => {
    it('should return user by id', async () => {
      // Test implementation
    });

    it('should throw NotFoundException when user not found', async () => {
      // Test implementation
    });
  });

  describe('create', () => {
    it('should create a new user', async () => {
      // Test implementation
    });

    it('should hash password before saving', async () => {
      // Test implementation
    });
  });

  describe('update', () => {
    it('should update existing user', async () => {
      // Test implementation
    });
  });

  describe('remove', () => {
    it('should delete user', async () => {
      // Test implementation
    });
  });
});
```

**Alternative: Test Folder Structure (Less Common):**

```
src/
├── users/
│   ├── users.controller.ts
│   ├── users.service.ts
│   ├── users.module.ts
│   └── __tests__/                    ← All tests in subfolder
│       ├── users.controller.spec.ts
│       ├── users.service.spec.ts
│       └── users.integration.spec.ts
```

**Shared Test Utilities:**

```
src/
├── test-utils/                       ← Shared test helpers
│   ├── mock-repository.factory.ts
│   ├── execution-context.mock.ts
│   ├── test-data.builder.ts
│   └── database-test.module.ts
├── users/
│   ├── users.service.ts
│   └── users.service.spec.ts
└── auth/
    ├── auth.service.ts
    └── auth.service.spec.ts

// Usage in test files
import { createMockRepository } from '../test-utils/mock-repository.factory';
import { createMockExecutionContext } from '../test-utils/execution-context.mock';
```

**Test Utilities Example:**

```typescript
// test-utils/mock-repository.factory.ts
export function createMockRepository() {
  return {
    find: jest.fn(),
    findOne: jest.fn(),
    findOneBy: jest.fn(),
    create: jest.fn(),
    save: jest.fn(),
    update: jest.fn(),
    delete: jest.fn(),
    remove: jest.fn(),
    count: jest.fn(),
    createQueryBuilder: jest.fn(() => ({
      where: jest.fn().mockReturnThis(),
      andWhere: jest.fn().mockReturnThis(),
      orWhere: jest.fn().mockReturnThis(),
      getMany: jest.fn(),
      getOne: jest.fn(),
    })),
  };
}

// Usage
import { createMockRepository } from '../test-utils/mock-repository.factory';

describe('UsersService', () => {
  let mockRepository: any;

  beforeEach(() => {
    mockRepository = createMockRepository();
    // ...
  });
});
```

**E2E Test Organization:**

```
test/
├── jest-e2e.json                     ← E2E Jest config
├── fixtures/                         ← Test data
│   ├── users.fixture.ts
│   └── products.fixture.ts
├── helpers/                          ← E2E helpers
│   ├── auth.helper.ts
│   └── database.helper.ts
├── app.e2e-spec.ts                   ← Root tests
├── users/                            ← Feature-based E2E
│   ├── users-create.e2e-spec.ts
│   ├── users-update.e2e-spec.ts
│   └── users-delete.e2e-spec.ts
└── auth/
    ├── auth-login.e2e-spec.ts
    └── auth-register.e2e-spec.ts
```

**Jest Configuration:**

```javascript
// jest.config.js (Unit tests)
module.exports = {
  moduleFileExtensions: ['js', 'json', 'ts'],
  rootDir: 'src',
  testRegex: '.*\\.spec\\.ts$',
  transform: {
    '^.+\\.(t|j)s$': 'ts-jest',
  },
  collectCoverageFrom: ['**/*.(t|j)s'],
  coverageDirectory: '../coverage',
  testEnvironment: 'node',
};

// test/jest-e2e.json (E2E tests)
{
  "moduleFileExtensions": ["js", "json", "ts"],
  "rootDir": ".",
  "testEnvironment": "node",
  "testRegex": ".e2e-spec.ts$",
  "transform": {
    "^.+\\.(t|j)s$": "ts-jest"
  }
}
```

**Package.json Scripts:**

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:cov": "jest --coverage",
    "test:e2e": "jest --config ./test/jest-e2e.json",
    "test:e2e:watch": "jest --config ./test/jest-e2e.json --watch"
  }
}
```

**Interview Tip**: **Co-locate** test files with source files using **`.spec.ts`** extension. Keep **E2E tests** in separate `test/` directory with **`.e2e-spec.ts`** extension. Use **`describe()`** to group tests by method/feature. Create **shared test utilities** in `test-utils/` for reusable mocks and helpers. Mirror **source structure** in test organization. Use **clear naming**: `users.service.spec.ts`, `users.controller.spec.ts`.

</details>

### 41. What should you test and what should you not test?

<details>
<summary>Answer</summary>

**Test business logic, critical paths, and edge cases. Don't test framework code, third-party libraries, or trivial getters/setters.**

**✅ WHAT TO TEST:**

**1. Business Logic:**
```typescript
// ✅ TEST: Complex business rules
class OrdersService {
  calculateTotal(items: OrderItem[], coupon?: Coupon, loyaltyPoints?: number): number {
    // Complex calculation with multiple rules
    let total = items.reduce((sum, item) => sum + item.price * item.quantity, 0);
    
    if (coupon && this.isCouponValid(coupon)) {
      total -= this.applyCouponDiscount(coupon, total);
    }
    
    if (loyaltyPoints && loyaltyPoints >= 100) {
      total -= this.applyLoyaltyDiscount(loyaltyPoints);
    }
    
    if (total < 0) total = 0;
    
    return total;
  }
}

// Test all scenarios
it('should calculate total without discounts', () => {});
it('should apply coupon discount', () => {});
it('should apply loyalty points discount', () => {});
it('should combine multiple discounts', () => {});
it('should not allow negative total', () => {});
```

**2. Edge Cases and Error Handling:**
```typescript
// ✅ TEST: Boundary conditions
describe('validateAge', () => {
  it('should accept age 18 (minimum)', () => {});
  it('should accept age 100 (maximum)', () => {});
  it('should reject age 17 (below minimum)', () => {});
  it('should reject age 101 (above maximum)', () => {});
  it('should reject negative age', () => {});
  it('should reject non-numeric age', () => {});
  it('should handle null age', () => {});
  it('should handle undefined age', () => {});
});
```

**3. Critical Paths:**
```typescript
// ✅ TEST: Authentication, payment, security
class AuthService {
  async login(email: string, password: string) {
    // MUST TEST: Security critical
  }
  
  async validateToken(token: string) {
    // MUST TEST: Security critical
  }
}

class PaymentService {
  async processPayment(amount: number, cardToken: string) {
    // MUST TEST: Money involved
  }
}
```

**4. Public API:**
```typescript
// ✅ TEST: All public methods
class UsersService {
  async findAll() { /* TEST */ }
  async findOne(id: number) { /* TEST */ }
  async create(dto: CreateUserDto) { /* TEST */ }
  async update(id: number, dto: UpdateUserDto) { /* TEST */ }
  async remove(id: number) { /* TEST */ }
}
```

**5. Data Transformations:**
```typescript
// ✅ TEST: Data manipulation
class DataTransformer {
  transform(input: RawData): ProcessedData {
    // TEST: Transformation logic
  }
}

class ValidationPipe {
  transform(value: any, metadata: ArgumentMetadata) {
    // TEST: Validation and transformation
  }
}
```

**❌ WHAT NOT TO TEST:**

**1. Framework Code:**
```typescript
// ❌ DON'T TEST: NestJS decorators
@Controller('users')
export class UsersController {
  // Don't test that @Controller works - that's NestJS's job
}

// ❌ DON'T TEST: Dependency injection
@Injectable()
export class UsersService {
  constructor(private userRepository: Repository<User>) {}
  // Don't test that DI works - framework handles this
}
```

**2. Third-Party Libraries:**
```typescript
// ❌ DON'T TEST: TypeORM, bcrypt, etc.
async create(dto: CreateUserDto) {
  const hashedPassword = await bcrypt.hash(dto.password, 10);
  // Don't test bcrypt.hash - library's responsibility
  
  const user = this.userRepository.create(dto);
  return this.userRepository.save(user);
  // Don't test TypeORM's save - library's responsibility
}

// ✅ DO TEST: Your logic using these libraries
it('should hash password before saving user', async () => {
  const dto = { email: 'test@example.com', password: 'plain' };
  const user = await service.create(dto);
  
  expect(user.password).not.toBe('plain');
  expect(user.password).toMatch(/^\$2[ayb]\$.{56}$/);
});
```

**3. Simple Getters/Setters:**
```typescript
// ❌ DON'T TEST: Trivial code
export class User {
  private _email: string;
  
  get email(): string {
    return this._email; // No logic to test
  }
  
  set email(value: string) {
    this._email = value; // No logic to test
  }
}

// ❌ DON'T TEST: Simple assignments
export class CreateUserDto {
  email: string;
  password: string;
  // No logic, no tests needed
}
```

**4. Configuration/Module Files:**
```typescript
// ❌ DON'T TEST: Module configuration
@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}
// No logic, no tests needed
```

**5. Constants and Enums:**
```typescript
// ❌ DON'T TEST: Static values
export enum UserRole {
  ADMIN = 'admin',
  USER = 'user',
  MODERATOR = 'moderator',
}

export const API_VERSION = 'v1';
export const MAX_FILE_SIZE = 5 * 1024 * 1024;
// No logic, no tests needed
```

**Practical Examples:**

**✅ Test This:**
```typescript
class UserValidator {
  validatePassword(password: string): ValidationResult {
    const errors = [];
    
    if (password.length < 8) {
      errors.push('Password must be at least 8 characters');
    }
    
    if (!/[A-Z]/.test(password)) {
      errors.push('Password must contain uppercase letter');
    }
    
    if (!/[0-9]/.test(password)) {
      errors.push('Password must contain number');
    }
    
    return {
      isValid: errors.length === 0,
      errors,
    };
  }
}

// Test all rules:
it('should validate password length', () => {});
it('should validate uppercase requirement', () => {});
it('should validate number requirement', () => {});
it('should return all errors', () => {});
it('should pass valid password', () => {});
```

**❌ Don't Test This:**
```typescript
// Simple DTO - no logic
export class CreateUserDto {
  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  password: string;
}
// class-validator tests its own decorators

// Simple entity - no logic
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  email: string;

  @Column()
  password: string;
}
// TypeORM tests its own decorators
```

**Decision Tree:**

```
Is it your code?
├─ No → Don't test (framework/library code)
└─ Yes
   ├─ Does it have logic?
   │  ├─ No → Don't test (getters/setters/DTOs)
   │  └─ Yes
   │     ├─ Is it critical? (auth/payment/security)
   │     │  └─ Yes → Test extensively (100% coverage)
   │     └─ Is it public API?
   │        └─ Yes → Test all paths
   └─ Is it just configuration?
      └─ Yes → Don't test (modules/constants)
```

**Interview Tip**: **Test**: Business logic, edge cases, error handling, critical paths (auth, payment), public APIs, data transformations. **Don't test**: Framework code (NestJS, TypeORM), third-party libraries (bcrypt, axios), simple getters/setters, DTOs without logic, configuration files, constants/enums. Focus on **value-adding tests** that catch real bugs, not tests that just exercise code for coverage percentage.

</details>

### 42. How do you handle database cleanup in tests?

<details>
<summary>Answer</summary>

**Handle database cleanup** by clearing data after each test or using transactions that rollback automatically.

**Strategy 1: Clear Database After Each Test:**

```typescript
// users.service.spec.ts
import { Test } from '@nestjs/testing';
import { TypeOrmModule } from '@nestjs/typeorm';
import { DataSource } from 'typeorm';
import { User } from './entities/user.entity';

describe('UsersService Integration', () => {
  let service: UsersService;
  let dataSource: DataSource;

  beforeAll(async () => {
    const module = await Test.createTestingModule({
      imports: [
        TypeOrmModule.forRoot({
          type: 'sqlite',
          database: ':memory:',
          entities: [User],
          synchronize: true,
        }),
        TypeOrmModule.forFeature([User]),
      ],
      providers: [UsersService],
    }).compile();

    service = module.get<UsersService>(UsersService);
    dataSource = module.get<DataSource>(DataSource);
  });

  afterEach(async () => {
    // Clear all tables after each test
    const entities = dataSource.entityMetadatas;
    for (const entity of entities) {
      const repository = dataSource.getRepository(entity.name);
      await repository.clear();
    }
  });

  afterAll(async () => {
    await dataSource.destroy();
  });

  it('should create user', async () => {
    const user = await service.create({ email: 'test@example.com' });
    expect(user.id).toBeDefined();
  });

  it('should start with empty database', async () => {
    // Database cleaned from previous test
    const users = await service.findAll();
    expect(users).toHaveLength(0);
  });
});
```

**Strategy 2: Database Helper Utility:**

```typescript
// test-utils/database-cleanup.helper.ts
import { DataSource } from 'typeorm';

export class DatabaseCleanup {
  static async cleanDatabase(dataSource: DataSource): Promise<void> {
    const entities = dataSource.entityMetadatas;
    
    for (const entity of entities) {
      const repository = dataSource.getRepository(entity.name);
      await repository.query(`DELETE FROM ${entity.tableName};`);
    }
  }

  static async resetAutoIncrement(dataSource: DataSource): Promise<void> {
    const entities = dataSource.entityMetadatas;
    
    for (const entity of entities) {
      if (dataSource.options.type === 'postgres') {
        await dataSource.query(
          `ALTER SEQUENCE ${entity.tableName}_id_seq RESTART WITH 1;`,
        );
      } else if (dataSource.options.type === 'mysql') {
        await dataSource.query(
          `ALTER TABLE ${entity.tableName} AUTO_INCREMENT = 1;`,
        );
      } else if (dataSource.options.type === 'sqlite') {
        await dataSource.query(
          `DELETE FROM sqlite_sequence WHERE name='${entity.tableName}';`,
        );
      }
    }
  }
}

// Usage in tests
import { DatabaseCleanup } from '../test-utils/database-cleanup.helper';

describe('UsersService', () => {
  afterEach(async () => {
    await DatabaseCleanup.cleanDatabase(dataSource);
  });
});
```

**Strategy 3: Transaction Rollback (Best for Speed):**

```typescript
// users.service.spec.ts
import { DataSource, QueryRunner } from 'typeorm';

describe('UsersService with Transactions', () => {
  let service: UsersService;
  let dataSource: DataSource;
  let queryRunner: QueryRunner;

  beforeAll(async () => {
    const module = await Test.createTestingModule({
      imports: [
        TypeOrmModule.forRoot({
          type: 'postgres',
          host: 'localhost',
          port: 5432,
          database: 'test_db',
          entities: [User],
          synchronize: true,
        }),
        TypeOrmModule.forFeature([User]),
      ],
      providers: [UsersService],
    }).compile();

    service = module.get<UsersService>(UsersService);
    dataSource = module.get<DataSource>(DataSource);
  });

  beforeEach(async () => {
    // Start transaction before each test
    queryRunner = dataSource.createQueryRunner();
    await queryRunner.connect();
    await queryRunner.startTransaction();
  });

  afterEach(async () => {
    // Rollback transaction after each test
    await queryRunner.rollbackTransaction();
    await queryRunner.release();
  });

  afterAll(async () => {
    await dataSource.destroy();
  });

  it('should create user', async () => {
    const user = await service.create({ email: 'test@example.com' });
    expect(user.id).toBeDefined();
    // Transaction will rollback, user won't persist
  });
});
```

**Strategy 4: Truncate Tables (Fast):**

```typescript
// test-utils/database-truncate.helper.ts
export class DatabaseTruncate {
  static async truncateAll(dataSource: DataSource): Promise<void> {
    const entities = dataSource.entityMetadatas;
    
    // Disable foreign key constraints
    if (dataSource.options.type === 'postgres') {
      await dataSource.query('SET session_replication_role = replica;');
    } else if (dataSource.options.type === 'mysql') {
      await dataSource.query('SET FOREIGN_KEY_CHECKS = 0;');
    }
    
    // Truncate all tables
    for (const entity of entities) {
      await dataSource.query(`TRUNCATE TABLE "${entity.tableName}" CASCADE;`);
    }
    
    // Re-enable foreign key constraints
    if (dataSource.options.type === 'postgres') {
      await dataSource.query('SET session_replication_role = DEFAULT;');
    } else if (dataSource.options.type === 'mysql') {
      await dataSource.query('SET FOREIGN_KEY_CHECKS = 1;');
    }
  }
}

// Usage
afterEach(async () => {
  await DatabaseTruncate.truncateAll(dataSource);
});
```

**Strategy 5: Separate Test Database:**

```typescript
// test/database-test.config.ts
export const testDatabaseConfig = {
  type: 'postgres' as const,
  host: process.env.TEST_DB_HOST || 'localhost',
  port: parseInt(process.env.TEST_DB_PORT) || 5433,
  username: process.env.TEST_DB_USER || 'test',
  password: process.env.TEST_DB_PASS || 'test',
  database: process.env.TEST_DB_NAME || 'nest_test',
  entities: ['src/**/*.entity.ts'],
  synchronize: true,
  dropSchema: true, // Drop and recreate schema before tests
};

// Usage in tests
beforeAll(async () => {
  const module = await Test.createTestingModule({
    imports: [TypeOrmModule.forRoot(testDatabaseConfig)],
  }).compile();
});
```

**Strategy 6: Docker Container for Tests:**

```yaml
# docker-compose.test.yml
version: '3.8'
services:
  test-db:
    image: postgres:15
    environment:
      POSTGRES_USER: test
      POSTGRES_PASSWORD: test
      POSTGRES_DB: nest_test
    ports:
      - '5433:5432'
    tmpfs:
      - /var/lib/postgresql/data  # In-memory for speed
```

```bash
# package.json
{
  "scripts": {
    "test:db:up": "docker-compose -f docker-compose.test.yml up -d",
    "test:db:down": "docker-compose -f docker-compose.test.yml down -v",
    "test:integration": "npm run test:db:up && jest --runInBand && npm run test:db:down"
  }
}
```

**Strategy 7: Global Cleanup Setup:**

```typescript
// test/setup.ts
import { DataSource } from 'typeorm';

let dataSource: DataSource;

global.beforeAll(async () => {
  // Setup test database connection
  dataSource = new DataSource({
    type: 'sqlite',
    database: ':memory:',
    entities: ['src/**/*.entity.ts'],
    synchronize: true,
  });
  await dataSource.initialize();
});

global.afterEach(async () => {
  // Clean all tables after each test
  const entities = dataSource.entityMetadatas;
  for (const entity of entities) {
    await dataSource.getRepository(entity.name).clear();
  }
});

global.afterAll(async () => {
  await dataSource.destroy();
});

// jest.config.js
module.exports = {
  setupFilesAfterEnv: ['<rootDir>/test/setup.ts'],
};
```

**E2E Test Cleanup:**

```typescript
// test/users.e2e-spec.ts
describe('Users E2E', () => {
  let app: INestApplication;
  let dataSource: DataSource;

  beforeAll(async () => {
    const module = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = module.createNestApplication();
    await app.init();
    
    dataSource = module.get<DataSource>(DataSource);
  });

  afterEach(async () => {
    // Clean database between E2E tests
    const entities = dataSource.entityMetadatas;
    for (const entity of entities) {
      await dataSource.getRepository(entity.name).delete({});
    }
  });

  afterAll(async () => {
    await dataSource.destroy();
    await app.close();
  });
});
```

**Interview Tip**: Clean database using **`afterEach()`** hook to clear all tables. Use **`repository.clear()`** or **`repository.delete({})`** to remove data. For **speed**, use **transactions with rollback** or **SQLite in-memory**. Create **helper utilities** for reusable cleanup logic. Use **separate test database** to avoid affecting development data. For **E2E tests**, use **Docker containers** with **tmpfs** for fast isolated databases. Always **close connections** in `afterAll()`.

</details>

### 43. How do you test error scenarios?

<details>
<summary>Answer</summary>

**Test error scenarios** by asserting that methods throw expected exceptions with correct messages and status codes.

**Testing Thrown Exceptions:**

```typescript
// users.service.spec.ts
import { NotFoundException, BadRequestException, ConflictException } from '@nestjs/common';

describe('UsersService Error Scenarios', () => {
  describe('findOne', () => {
    it('should throw NotFoundException when user not found', async () => {
      mockRepository.findOne.mockResolvedValue(null);

      await expect(service.findOne(999)).rejects.toThrow(NotFoundException);
    });

    it('should throw NotFoundException with correct message', async () => {
      mockRepository.findOne.mockResolvedValue(null);

      await expect(service.findOne(999)).rejects.toThrow('User not found');
    });

    it('should throw specific NotFoundException', async () => {
      mockRepository.findOne.mockResolvedValue(null);

      await expect(service.findOne(999)).rejects.toThrow(
        new NotFoundException('User with ID 999 not found'),
      );
    });
  });

  describe('create', () => {
    it('should throw BadRequestException for invalid email', async () => {
      const dto = { email: 'invalid-email', password: 'pass123' };

      await expect(service.create(dto)).rejects.toThrow(BadRequestException);
    });

    it('should throw ConflictException for duplicate email', async () => {
      const dto = { email: 'existing@example.com', password: 'pass123' };
      mockRepository.save.mockRejectedValue({ code: '23505' }); // Postgres unique violation

      await expect(service.create(dto)).rejects.toThrow(ConflictException);
      await expect(service.create(dto)).rejects.toThrow('Email already exists');
    });
  });

  describe('update', () => {
    it('should throw NotFoundException when updating non-existent user', async () => {
      mockRepository.findOne.mockResolvedValue(null);

      await expect(service.update(999, { name: 'New Name' })).rejects.toThrow(
        NotFoundException,
      );
    });
  });

  describe('remove', () => {
    it('should throw NotFoundException when deleting non-existent user', async () => {
      mockRepository.findOne.mockResolvedValue(null);

      await expect(service.remove(999)).rejects.toThrow(NotFoundException);
    });
  });
});
```

**Testing Database Errors:**

```typescript
describe('Database Error Handling', () => {
  it('should handle database connection error', async () => {
    mockRepository.find.mockRejectedValue(
      new Error('Connection timeout'),
    );

    await expect(service.findAll()).rejects.toThrow('Connection timeout');
  });

  it('should handle query error', async () => {
    mockRepository.findOne.mockRejectedValue(
      new Error('Column does not exist'),
    );

    await expect(service.findOne(1)).rejects.toThrow('Column does not exist');
  });

  it('should transform database unique constraint error', async () => {
    const dbError = { code: '23505', detail: 'Key (email)=(test@example.com) already exists' };
    mockRepository.save.mockRejectedValue(dbError);

    await expect(service.create({ email: 'test@example.com' }))
      .rejects
      .toThrow(ConflictException);
  });

  it('should transform database foreign key error', async () => {
    const dbError = { code: '23503', detail: 'Foreign key violation' };
    mockRepository.remove.mockRejectedValue(dbError);

    await expect(service.remove(1))
      .rejects
      .toThrow('Cannot delete user with existing relationships');
  });
});
```

**Testing External Service Errors:**

```typescript
// payment.service.spec.ts
describe('PaymentService Error Scenarios', () => {
  it('should handle Stripe API error', async () => {
    const stripeError = {
      type: 'StripeCardError',
      code: 'card_declined',
      message: 'Your card was declined',
    };
    mockStripe.charges.create.mockRejectedValue(stripeError);

    await expect(
      service.processPayment({ amount: 100, cardToken: 'tok_123' }),
    ).rejects.toThrow('Your card was declined');
  });

  it('should handle network timeout', async () => {
    mockHttpService.post.mockRejectedValue(
      new Error('ETIMEDOUT'),
    );

    await expect(service.callExternalAPI()).rejects.toThrow('ETIMEDOUT');
  });

  it('should handle 500 server error', async () => {
    mockHttpService.get.mockRejectedValue({
      response: { status: 500, data: { error: 'Internal Server Error' } },
    });

    await expect(service.fetchData()).rejects.toThrow('External service unavailable');
  });
});
```

**Testing Validation Errors:**

```typescript
// validation.pipe.spec.ts
describe('ValidationPipe Error Scenarios', () => {
  it('should throw BadRequestException for invalid email', async () => {
    const dto = { email: 'not-an-email', password: 'pass123' };

    await expect(
      pipe.transform(dto, { type: 'body', metatype: CreateUserDto }),
    ).rejects.toThrow(BadRequestException);
  });

  it('should provide detailed validation errors', async () => {
    const dto = { email: 'invalid', password: '123' }; // Multiple errors

    try {
      await pipe.transform(dto, { type: 'body', metatype: CreateUserDto });
    } catch (error) {
      expect(error).toBeInstanceOf(BadRequestException);
      expect(error.response.message).toContain('email must be an email');
      expect(error.response.message).toContain('password must be longer');
    }
  });
});
```

**Testing Guard Errors:**

```typescript
// auth.guard.spec.ts
describe('AuthGuard Error Scenarios', () => {
  it('should throw UnauthorizedException when token missing', () => {
    const context = createMockContext({ headers: {} });

    expect(() => guard.canActivate(context)).toThrow(UnauthorizedException);
  });

  it('should throw UnauthorizedException for invalid token', () => {
    const context = createMockContext({
      headers: { authorization: 'Bearer invalid_token' },
    });
    mockJwtService.verify.mockImplementation(() => {
      throw new Error('Invalid token');
    });

    expect(() => guard.canActivate(context)).toThrow(UnauthorizedException);
  });

  it('should throw UnauthorizedException for expired token', () => {
    const context = createMockContext({
      headers: { authorization: 'Bearer expired_token' },
    });
    mockJwtService.verify.mockImplementation(() => {
      throw new Error('Token expired');
    });

    expect(() => guard.canActivate(context)).toThrow('Token expired');
  });
});
```

**Testing Multiple Error Types:**

```typescript
describe('Error Handling Comprehensive', () => {
  it('should throw UnauthorizedException (401)', async () => {
    await expect(service.login('wrong@example.com', 'wrong'))
      .rejects
      .toThrow(UnauthorizedException);
  });

  it('should throw ForbiddenException (403)', async () => {
    await expect(service.accessAdminResource(regularUserId))
      .rejects
      .toThrow(ForbiddenException);
  });

  it('should throw NotFoundException (404)', async () => {
    await expect(service.findOne(999))
      .rejects
      .toThrow(NotFoundException);
  });

  it('should throw ConflictException (409)', async () => {
    await expect(service.create({ email: 'duplicate@example.com' }))
      .rejects
      .toThrow(ConflictException);
  });

  it('should throw BadRequestException (400)', async () => {
    await expect(service.create({ email: 'invalid' }))
      .rejects
      .toThrow(BadRequestException);
  });

  it('should throw InternalServerErrorException (500)', async () => {
    mockRepository.find.mockRejectedValue(new Error('Database crashed'));
    
    await expect(service.findAll())
      .rejects
      .toThrow(InternalServerErrorException);
  });
});
```

**Testing Error Messages:**

```typescript
describe('Error Messages', () => {
  it('should provide helpful error message for user not found', async () => {
    mockRepository.findOne.mockResolvedValue(null);

    try {
      await service.findOne(123);
      fail('Should have thrown');
    } catch (error) {
      expect(error.message).toBe('User with ID 123 not found');
      expect(error.status).toBe(404);
    }
  });

  it('should include context in error message', async () => {
    try {
      await service.transferMoney(fromId, toId, -100);
      fail('Should have thrown');
    } catch (error) {
      expect(error.message).toContain('Amount must be positive');
      expect(error.message).toContain('-100');
    }
  });
});
```

**Testing Error Recovery:**

```typescript
describe('Error Recovery', () => {
  it('should retry on transient error', async () => {
    mockHttpService.get
      .mockRejectedValueOnce(new Error('ETIMEDOUT'))
      .mockRejectedValueOnce(new Error('ETIMEDOUT'))
      .mockResolvedValueOnce({ data: { success: true } });

    const result = await service.fetchWithRetry();

    expect(result.success).toBe(true);
    expect(mockHttpService.get).toHaveBeenCalledTimes(3);
  });

  it('should fallback to default value on error', async () => {
    mockConfigService.get.mockImplementation(() => {
      throw new Error('Config not found');
    });

    const result = service.getConfigWithDefault('key', 'default');

    expect(result).toBe('default');
  });
});
```

**E2E Error Testing:**

```typescript
// users.e2e-spec.ts
describe('Users E2E Error Scenarios', () => {
  it('should return 404 for non-existent user', () => {
    return request(app.getHttpServer())
      .get('/users/999')
      .expect(404)
      .expect((res) => {
        expect(res.body.message).toContain('not found');
        expect(res.body.statusCode).toBe(404);
        expect(res.body.error).toBe('Not Found');
      });
  });

  it('should return 400 for invalid input', () => {
    return request(app.getHttpServer())
      .post('/users')
      .send({ email: 'invalid' })
      .expect(400)
      .expect((res) => {
        expect(res.body.message).toContain('validation');
        expect(Array.isArray(res.body.message)).toBe(true);
      });
  });

  it('should return 401 for missing auth token', () => {
    return request(app.getHttpServer())
      .get('/users/profile')
      .expect(401)
      .expect((res) => {
        expect(res.body.message).toBe('Unauthorized');
      });
  });
});
```

**Interview Tip**: Test error scenarios using **`expect().rejects.toThrow()`** for async code. Test **all exception types** (NotFoundException, BadRequestException, etc.). Verify **error messages** are helpful and specific. Test **database errors** (unique constraints, foreign keys). Test **external service errors** (API failures, timeouts). Test **validation errors** with detailed messages. Test **error recovery** (retries, fallbacks). In **E2E tests**, verify **HTTP status codes** (400, 401, 404, 409, 500) and error response format.

</details>

## Common Testing Patterns

### 44. How do you test database transactions?

<details>
<summary>Answer</summary>

**Test database transactions** by verifying that operations within a transaction either all succeed or all rollback on error.

**Testing Transaction Success:**

```typescript
// orders.service.spec.ts
import { DataSource, QueryRunner } from 'typeorm';

describe('OrdersService Transactions', () => {
  let service: OrdersService;
  let dataSource: DataSource;
  let mockQueryRunner: QueryRunner;

  beforeEach(() => {
    mockQueryRunner = {
      connect: jest.fn(),
      startTransaction: jest.fn(),
      commitTransaction: jest.fn(),
      rollbackTransaction: jest.fn(),
      release: jest.fn(),
      manager: {
        save: jest.fn(),
        update: jest.fn(),
        findOne: jest.fn(),
      },
    } as any;

    dataSource = {
      createQueryRunner: jest.fn().mockReturnValue(mockQueryRunner),
    } as any;

    service = new OrdersService(dataSource);
  });

  describe('createOrder', () => {
    it('should commit transaction on successful order creation', async () => {
      const orderData = {
        userId: 1,
        items: [{ productId: 1, quantity: 2, price: 50 }],
        total: 100,
      };

      mockQueryRunner.manager.save.mockResolvedValue({ id: 1, ...orderData });

      await service.createOrder(orderData);

      expect(mockQueryRunner.connect).toHaveBeenCalled();
      expect(mockQueryRunner.startTransaction).toHaveBeenCalled();
      expect(mockQueryRunner.commitTransaction).toHaveBeenCalled();
      expect(mockQueryRunner.release).toHaveBeenCalled();
      expect(mockQueryRunner.rollbackTransaction).not.toHaveBeenCalled();
    });

    it('should rollback transaction on error', async () => {
      const orderData = {
        userId: 1,
        items: [{ productId: 1, quantity: 2, price: 50 }],
        total: 100,
      };

      mockQueryRunner.manager.save.mockRejectedValue(new Error('Database error'));

      await expect(service.createOrder(orderData)).rejects.toThrow('Database error');

      expect(mockQueryRunner.startTransaction).toHaveBeenCalled();
      expect(mockQueryRunner.rollbackTransaction).toHaveBeenCalled();
      expect(mockQueryRunner.commitTransaction).not.toHaveBeenCalled();
      expect(mockQueryRunner.release).toHaveBeenCalled();
    });
  });
});
```

**Service Implementation Example:**

```typescript
// orders.service.ts
@Injectable()
export class OrdersService {
  constructor(private dataSource: DataSource) {}

  async createOrder(orderData: CreateOrderDto): Promise<Order> {
    const queryRunner = this.dataSource.createQueryRunner();
    await queryRunner.connect();
    await queryRunner.startTransaction();

    try {
      // Create order
      const order = await queryRunner.manager.save(Order, {
        userId: orderData.userId,
        total: orderData.total,
      });

      // Create order items
      for (const item of orderData.items) {
        await queryRunner.manager.save(OrderItem, {
          orderId: order.id,
          productId: item.productId,
          quantity: item.quantity,
          price: item.price,
        });

        // Update inventory
        await queryRunner.manager.decrement(
          Inventory,
          { productId: item.productId },
          'quantity',
          item.quantity,
        );
      }

      // Commit transaction
      await queryRunner.commitTransaction();
      return order;
    } catch (error) {
      // Rollback on error
      await queryRunner.rollbackTransaction();
      throw error;
    } finally {
      // Release query runner
      await queryRunner.release();
    }
  }
}
```

**Testing Complex Transaction:**

```typescript
describe('transferMoney', () => {
  it('should transfer money between accounts in transaction', async () => {
    const fromAccountId = 1;
    const toAccountId = 2;
    const amount = 100;

    const fromAccount = { id: 1, balance: 500 };
    const toAccount = { id: 2, balance: 200 };

    mockQueryRunner.manager.findOne
      .mockResolvedValueOnce(fromAccount)
      .mockResolvedValueOnce(toAccount);

    mockQueryRunner.manager.save.mockResolvedValue(true);

    await service.transferMoney(fromAccountId, toAccountId, amount);

    expect(mockQueryRunner.startTransaction).toHaveBeenCalled();
    expect(mockQueryRunner.manager.save).toHaveBeenCalledTimes(2);
    expect(mockQueryRunner.commitTransaction).toHaveBeenCalled();
  });

  it('should rollback if sender has insufficient funds', async () => {
    const fromAccount = { id: 1, balance: 50 };
    const toAccount = { id: 2, balance: 200 };

    mockQueryRunner.manager.findOne
      .mockResolvedValueOnce(fromAccount)
      .mockResolvedValueOnce(toAccount);

    await expect(
      service.transferMoney(1, 2, 100),
    ).rejects.toThrow('Insufficient funds');

    expect(mockQueryRunner.startTransaction).toHaveBeenCalled();
    expect(mockQueryRunner.rollbackTransaction).toHaveBeenCalled();
    expect(mockQueryRunner.commitTransaction).not.toHaveBeenCalled();
  });

  it('should rollback if recipient account not found', async () => {
    const fromAccount = { id: 1, balance: 500 };

    mockQueryRunner.manager.findOne
      .mockResolvedValueOnce(fromAccount)
      .mockResolvedValueOnce(null); // Recipient not found

    await expect(
      service.transferMoney(1, 2, 100),
    ).rejects.toThrow('Account not found');

    expect(mockQueryRunner.rollbackTransaction).toHaveBeenCalled();
  });
});
```

**Integration Test with Real Transactions:**

```typescript
// orders.service.integration.spec.ts
describe('OrdersService Integration (Real Transactions)', () => {
  let service: OrdersService;
  let dataSource: DataSource;

  beforeAll(async () => {
    const module = await Test.createTestingModule({
      imports: [
        TypeOrmModule.forRoot({
          type: 'postgres',
          host: 'localhost',
          database: 'test_db',
          entities: [Order, OrderItem, Inventory],
          synchronize: true,
        }),
      ],
      providers: [OrdersService],
    }).compile();

    service = module.get<OrdersService>(OrdersService);
    dataSource = module.get<DataSource>(DataSource);
  });

  afterEach(async () => {
    // Clean database
    await dataSource.query('TRUNCATE TABLE orders, order_items, inventory CASCADE');
  });

  it('should create order and update inventory in transaction', async () => {
    // Setup: Create product with inventory
    await dataSource.getRepository(Inventory).save({
      productId: 1,
      quantity: 10,
    });

    // Act: Create order
    const order = await service.createOrder({
      userId: 1,
      items: [{ productId: 1, quantity: 2, price: 50 }],
      total: 100,
    });

    // Assert: Order created
    expect(order.id).toBeDefined();

    // Assert: Inventory updated
    const inventory = await dataSource.getRepository(Inventory).findOne({
      where: { productId: 1 },
    });
    expect(inventory.quantity).toBe(8); // 10 - 2
  });

  it('should rollback entire transaction on inventory error', async () => {
    // Setup: Create product with insufficient inventory
    await dataSource.getRepository(Inventory).save({
      productId: 1,
      quantity: 1,
    });

    // Act & Assert: Should fail
    await expect(
      service.createOrder({
        userId: 1,
        items: [{ productId: 1, quantity: 5, price: 50 }],
        total: 250,
      }),
    ).rejects.toThrow('Insufficient inventory');

    // Assert: No order created (transaction rolled back)
    const orders = await dataSource.getRepository(Order).find();
    expect(orders).toHaveLength(0);

    // Assert: Inventory unchanged
    const inventory = await dataSource.getRepository(Inventory).findOne({
      where: { productId: 1 },
    });
    expect(inventory.quantity).toBe(1); // Still 1, not decremented
  });
});
```

**Testing Nested Transactions:**

```typescript
describe('nestedTransactions', () => {
  it('should handle nested transaction with savepoint', async () => {
    mockQueryRunner.manager.save
      .mockResolvedValueOnce({ id: 1 }) // Parent save
      .mockResolvedValueOnce({ id: 2 }); // Child save

    await service.createOrderWithItems(orderData);

    expect(mockQueryRunner.startTransaction).toHaveBeenCalled();
    expect(mockQueryRunner.manager.save).toHaveBeenCalledTimes(2);
    expect(mockQueryRunner.commitTransaction).toHaveBeenCalled();
  });

  it('should rollback nested transaction independently', async () => {
    mockQueryRunner.manager.save
      .mockResolvedValueOnce({ id: 1 }) // Parent succeeds
      .mockRejectedValueOnce(new Error('Child failed')); // Child fails

    await expect(service.createOrderWithItems(orderData))
      .rejects.toThrow('Child failed');

    expect(mockQueryRunner.rollbackTransaction).toHaveBeenCalled();
  });
});
```

**Testing Transaction Isolation:**

```typescript
describe('transaction isolation', () => {
  it('should not see uncommitted changes from other transaction', async () => {
    // Simulate concurrent transactions
    const queryRunner1 = dataSource.createQueryRunner();
    const queryRunner2 = dataSource.createQueryRunner();

    await queryRunner1.connect();
    await queryRunner2.connect();

    await queryRunner1.startTransaction('READ COMMITTED');
    await queryRunner2.startTransaction('READ COMMITTED');

    try {
      // Transaction 1: Update balance
      await queryRunner1.manager.update(Account, { id: 1 }, { balance: 500 });

      // Transaction 2: Try to read (should not see uncommitted change)
      const account = await queryRunner2.manager.findOne(Account, {
        where: { id: 1 },
      });
      expect(account.balance).toBe(1000); // Original value

      // Commit transaction 1
      await queryRunner1.commitTransaction();

      // Transaction 2: Now should see committed change
      const updatedAccount = await queryRunner2.manager.findOne(Account, {
        where: { id: 1 },
      });
      expect(updatedAccount.balance).toBe(500);

      await queryRunner2.commitTransaction();
    } finally {
      await queryRunner1.release();
      await queryRunner2.release();
    }
  });
});
```

**Interview Tip**: Test transactions by **mocking QueryRunner** with `startTransaction()`, `commitTransaction()`, `rollbackTransaction()`. Verify **commit on success**, **rollback on error**. Test **multiple operations** within transaction. For **integration tests**, use **real database** to verify atomicity. Test **error scenarios** that trigger rollback. Verify **data consistency** after rollback. Test **transaction isolation levels** if needed.

</details>

### 45. How do you test file uploads?

<details>
<summary>Answer</summary>

**Test file uploads** by mocking the file object and verifying file processing logic and storage operations.

**Testing File Upload Controller:**

```typescript
// files.controller.spec.ts
import { FilesController } from './files.controller';
import { FilesService } from './files.service';

describe('FilesController', () => {
  let controller: FilesController;
  let service: FilesService;

  beforeEach(() => {
    service = {
      uploadFile: jest.fn(),
      uploadMultipleFiles: jest.fn(),
    } as any;

    controller = new FilesController(service);
  });

  describe('uploadFile', () => {
    it('should upload single file', async () => {
      const mockFile: Express.Multer.File = {
        fieldname: 'file',
        originalname: 'test.jpg',
        encoding: '7bit',
        mimetype: 'image/jpeg',
        size: 12345,
        buffer: Buffer.from('fake-image-data'),
        destination: '/uploads',
        filename: 'test-123.jpg',
        path: '/uploads/test-123.jpg',
        stream: null,
      } as Express.Multer.File;

      const expectedResult = {
        filename: 'test-123.jpg',
        originalName: 'test.jpg',
        size: 12345,
        url: 'http://localhost:3000/uploads/test-123.jpg',
      };

      (service.uploadFile as jest.Mock).mockResolvedValue(expectedResult);

      const result = await controller.uploadFile(mockFile);

      expect(result).toEqual(expectedResult);
      expect(service.uploadFile).toHaveBeenCalledWith(mockFile);
    });

    it('should throw error if no file provided', async () => {
      await expect(controller.uploadFile(null)).rejects.toThrow(
        'No file uploaded',
      );
    });

    it('should validate file type', async () => {
      const mockFile: Express.Multer.File = {
        originalname: 'test.exe',
        mimetype: 'application/x-msdownload',
        size: 12345,
        buffer: Buffer.from('fake-data'),
      } as Express.Multer.File;

      await expect(controller.uploadFile(mockFile)).rejects.toThrow(
        'Invalid file type',
      );
    });

    it('should validate file size', async () => {
      const mockFile: Express.Multer.File = {
        originalname: 'test.jpg',
        mimetype: 'image/jpeg',
        size: 10 * 1024 * 1024, // 10MB
        buffer: Buffer.from('fake-data'),
      } as Express.Multer.File;

      await expect(controller.uploadFile(mockFile)).rejects.toThrow(
        'File too large',
      );
    });
  });

  describe('uploadMultipleFiles', () => {
    it('should upload multiple files', async () => {
      const mockFiles: Express.Multer.File[] = [
        {
          originalname: 'file1.jpg',
          mimetype: 'image/jpeg',
          size: 1000,
          buffer: Buffer.from('file1'),
        } as Express.Multer.File,
        {
          originalname: 'file2.jpg',
          mimetype: 'image/jpeg',
          size: 2000,
          buffer: Buffer.from('file2'),
        } as Express.Multer.File,
      ];

      const expectedResult = [
        { filename: 'file1.jpg', url: '/uploads/file1.jpg' },
        { filename: 'file2.jpg', url: '/uploads/file2.jpg' },
      ];

      (service.uploadMultipleFiles as jest.Mock).mockResolvedValue(expectedResult);

      const result = await controller.uploadMultipleFiles(mockFiles);

      expect(result).toEqual(expectedResult);
      expect(result).toHaveLength(2);
    });
  });
});
```

**Testing File Upload Service:**

```typescript
// files.service.spec.ts
import { FilesService } from './files.service';
import * as fs from 'fs';
import * as path from 'path';

jest.mock('fs');
jest.mock('path');

describe('FilesService', () => {
  let service: FilesService;

  beforeEach(() => {
    service = new FilesService();
    jest.clearAllMocks();
  });

  describe('uploadFile', () => {
    it('should save file to disk', async () => {
      const mockFile: Express.Multer.File = {
        originalname: 'test.jpg',
        mimetype: 'image/jpeg',
        size: 12345,
        buffer: Buffer.from('fake-image-data'),
      } as Express.Multer.File;

      (fs.writeFile as jest.Mock).mockImplementation((path, data, callback) => {
        callback(null);
      });

      (path.join as jest.Mock).mockReturnValue('/uploads/test-123.jpg');

      const result = await service.uploadFile(mockFile);

      expect(fs.writeFile).toHaveBeenCalled();
      expect(result).toHaveProperty('filename');
      expect(result).toHaveProperty('url');
    });

    it('should handle file write error', async () => {
      const mockFile: Express.Multer.File = {
        originalname: 'test.jpg',
        buffer: Buffer.from('data'),
      } as Express.Multer.File;

      (fs.writeFile as jest.Mock).mockImplementation((path, data, callback) => {
        callback(new Error('Disk full'));
      });

      await expect(service.uploadFile(mockFile)).rejects.toThrow('Disk full');
    });

    it('should generate unique filename', async () => {
      const mockFile: Express.Multer.File = {
        originalname: 'test.jpg',
        buffer: Buffer.from('data'),
      } as Express.Multer.File;

      (fs.writeFile as jest.Mock).mockImplementation((path, data, callback) => {
        callback(null);
      });

      const result1 = await service.uploadFile(mockFile);
      const result2 = await service.uploadFile(mockFile);

      expect(result1.filename).not.toBe(result2.filename);
    });
  });

  describe('validateFile', () => {
    it('should accept valid image types', () => {
      const validTypes = ['image/jpeg', 'image/png', 'image/gif'];

      validTypes.forEach((mimetype) => {
        const file = { mimetype } as Express.Multer.File;
        expect(() => service.validateFile(file)).not.toThrow();
      });
    });

    it('should reject invalid file types', () => {
      const invalidTypes = ['application/exe', 'text/javascript', 'video/mp4'];

      invalidTypes.forEach((mimetype) => {
        const file = { mimetype } as Express.Multer.File;
        expect(() => service.validateFile(file)).toThrow('Invalid file type');
      });
    });

    it('should validate file size', () => {
      const maxSize = 5 * 1024 * 1024; // 5MB

      const validFile = { size: maxSize - 1 } as Express.Multer.File;
      expect(() => service.validateFile(validFile)).not.toThrow();

      const invalidFile = { size: maxSize + 1 } as Express.Multer.File;
      expect(() => service.validateFile(invalidFile)).toThrow('File too large');
    });
  });
});
```

**Testing with Cloud Storage (S3):**

```typescript
// s3-upload.service.spec.ts
import { S3 } from 'aws-sdk';

describe('S3UploadService', () => {
  let service: S3UploadService;
  let mockS3: jest.Mocked<S3>;

  beforeEach(() => {
    mockS3 = {
      upload: jest.fn().mockReturnThis(),
      promise: jest.fn(),
    } as any;

    service = new S3UploadService(mockS3);
  });

  it('should upload file to S3', async () => {
    const mockFile: Express.Multer.File = {
      originalname: 'test.jpg',
      mimetype: 'image/jpeg',
      buffer: Buffer.from('fake-data'),
      size: 12345,
    } as Express.Multer.File;

    const mockS3Response = {
      Location: 'https://bucket.s3.amazonaws.com/test.jpg',
      Key: 'uploads/test-123.jpg',
      Bucket: 'my-bucket',
    };

    mockS3.upload().promise.mockResolvedValue(mockS3Response as any);

    const result = await service.uploadToS3(mockFile);

    expect(result.url).toBe(mockS3Response.Location);
    expect(mockS3.upload).toHaveBeenCalledWith({
      Bucket: 'my-bucket',
      Key: expect.stringContaining('test-123.jpg'),
      Body: mockFile.buffer,
      ContentType: mockFile.mimetype,
    });
  });

  it('should handle S3 upload error', async () => {
    const mockFile: Express.Multer.File = {
      buffer: Buffer.from('data'),
    } as Express.Multer.File;

    mockS3.upload().promise.mockRejectedValue(new Error('S3 error'));

    await expect(service.uploadToS3(mockFile)).rejects.toThrow('S3 error');
  });
});
```

**E2E Testing File Uploads:**

```typescript
// files.e2e-spec.ts
import * as request from 'supertest';
import * as path from 'path';

describe('Files E2E', () => {
  let app: INestApplication;

  beforeAll(async () => {
    const module = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = module.createNestApplication();
    await app.init();
  });

  afterAll(async () => {
    await app.close();
  });

  describe('POST /files/upload', () => {
    it('should upload single file', () => {
      return request(app.getHttpServer())
        .post('/files/upload')
        .attach('file', path.join(__dirname, 'fixtures/test-image.jpg'))
        .expect(201)
        .expect((res) => {
          expect(res.body).toHaveProperty('filename');
          expect(res.body).toHaveProperty('url');
          expect(res.body.originalName).toBe('test-image.jpg');
        });
    });

    it('should reject file without multipart/form-data', () => {
      return request(app.getHttpServer())
        .post('/files/upload')
        .send({ file: 'not-a-file' })
        .expect(400);
    });

    it('should reject oversized file', () => {
      return request(app.getHttpServer())
        .post('/files/upload')
        .attach('file', path.join(__dirname, 'fixtures/large-file.jpg'))
        .expect(413)
        .expect((res) => {
          expect(res.body.message).toContain('File too large');
        });
    });

    it('should reject invalid file type', () => {
      return request(app.getHttpServer())
        .post('/files/upload')
        .attach('file', path.join(__dirname, 'fixtures/test.exe'))
        .expect(400)
        .expect((res) => {
          expect(res.body.message).toContain('Invalid file type');
        });
    });
  });

  describe('POST /files/upload/multiple', () => {
    it('should upload multiple files', () => {
      return request(app.getHttpServer())
        .post('/files/upload/multiple')
        .attach('files', path.join(__dirname, 'fixtures/image1.jpg'))
        .attach('files', path.join(__dirname, 'fixtures/image2.jpg'))
        .attach('files', path.join(__dirname, 'fixtures/image3.jpg'))
        .expect(201)
        .expect((res) => {
          expect(Array.isArray(res.body)).toBe(true);
          expect(res.body).toHaveLength(3);
        });
    });

    it('should limit number of files', () => {
      const files = Array(11).fill(path.join(__dirname, 'fixtures/image.jpg'));
      
      let req = request(app.getHttpServer()).post('/files/upload/multiple');
      
      files.forEach((file) => {
        req = req.attach('files', file);
      });

      return req.expect(400).expect((res) => {
        expect(res.body.message).toContain('Too many files');
      });
    });
  });
});
```

**Testing File Processing:**

```typescript
describe('processImage', () => {
  it('should resize uploaded image', async () => {
    const mockFile: Express.Multer.File = {
      buffer: Buffer.from('fake-image-data'),
      mimetype: 'image/jpeg',
    } as Express.Multer.File;

    const mockSharp = {
      resize: jest.fn().mockReturnThis(),
      toBuffer: jest.fn().mockResolvedValue(Buffer.from('resized-data')),
    };

    (sharp as any).mockReturnValue(mockSharp);

    const result = await service.processImage(mockFile, { width: 800, height: 600 });

    expect(mockSharp.resize).toHaveBeenCalledWith(800, 600);
    expect(result).toBeInstanceOf(Buffer);
  });

  it('should generate thumbnail', async () => {
    const mockFile: Express.Multer.File = {
      buffer: Buffer.from('fake-image-data'),
    } as Express.Multer.File;

    const result = await service.generateThumbnail(mockFile);

    expect(result).toHaveProperty('thumbnail');
    expect(result.thumbnail).toHaveProperty('buffer');
    expect(result.thumbnail).toHaveProperty('size');
  });
});
```

**Interview Tip**: Test file uploads by creating **mock Express.Multer.File objects** with buffer, originalname, mimetype, size. Test **file validation** (type, size, extension). Mock **file system operations** (fs.writeFile) or **cloud storage** (S3.upload). For **E2E tests**, use **`supertest`'s `.attach()`** method with real fixture files. Test **multiple file uploads**, **error scenarios** (disk full, invalid type), and **file processing** (resize, thumbnail generation).

</details>

### 46. How do you test WebSockets?

<details>
<summary>Answer</summary>

**Test WebSockets** by mocking the socket connection and verifying event emission/handling logic.

**Testing WebSocket Gateway:**

```typescript
// chat.gateway.spec.ts
import { Test } from '@nestjs/testing';
import { ChatGateway } from './chat.gateway';
import { ChatService } from './chat.service';
import { Socket } from 'socket.io';

describe('ChatGateway', () => {
  let gateway: ChatGateway;
  let service: ChatService;
  let mockSocket: Socket;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        ChatGateway,
        {
          provide: ChatService,
          useValue: {
            addMessage: jest.fn(),
            getMessages: jest.fn(),
            addUserToRoom: jest.fn(),
          },
        },
      ],
    }).compile();

    gateway = module.get<ChatGateway>(ChatGateway);
    service = module.get<ChatService>(ChatService);

    // Mock Socket object
    mockSocket = {
      id: 'test-socket-id',
      rooms: new Set(['room1']),
      join: jest.fn(),
      leave: jest.fn(),
      emit: jest.fn(),
      broadcast: {
        emit: jest.fn(),
        to: jest.fn().mockReturnThis(),
      },
      to: jest.fn().mockReturnThis(),
      handshake: {
        auth: { token: 'test-token' },
        headers: {},
      },
    } as any;
  });

  describe('handleConnection', () => {
    it('should handle client connection', async () => {
      await gateway.handleConnection(mockSocket);

      expect(mockSocket.emit).toHaveBeenCalledWith('connected', {
        message: 'Connected to chat server',
        socketId: 'test-socket-id',
      });
    });

    it('should authenticate client on connection', async () => {
      const authenticatedSocket = {
        ...mockSocket,
        handshake: {
          auth: { token: 'valid-token' },
        },
      } as any;

      await gateway.handleConnection(authenticatedSocket);

      expect(authenticatedSocket.emit).toHaveBeenCalled();
    });

    it('should reject unauthenticated connection', async () => {
      const unauthenticatedSocket = {
        ...mockSocket,
        handshake: { auth: {} },
        disconnect: jest.fn(),
      } as any;

      await gateway.handleConnection(unauthenticatedSocket);

      expect(unauthenticatedSocket.disconnect).toHaveBeenCalled();
    });
  });

  describe('handleDisconnect', () => {
    it('should handle client disconnection', () => {
      gateway.handleDisconnect(mockSocket);

      // Verify cleanup logic
      expect(service.removeUser).toHaveBeenCalledWith('test-socket-id');
    });

    it('should broadcast user left message', () => {
      gateway.handleDisconnect(mockSocket);

      expect(mockSocket.broadcast.emit).toHaveBeenCalledWith('userLeft', {
        socketId: 'test-socket-id',
      });
    });
  });

  describe('handleMessage', () => {
    it('should handle incoming message', async () => {
      const messageData = {
        room: 'room1',
        message: 'Hello World',
        user: 'John',
      };

      (service.addMessage as jest.Mock).mockResolvedValue({
        id: 1,
        ...messageData,
        timestamp: new Date(),
      });

      await gateway.handleMessage(mockSocket, messageData);

      expect(service.addMessage).toHaveBeenCalledWith(messageData);
      expect(mockSocket.to).toHaveBeenCalledWith('room1');
      expect(mockSocket.broadcast.emit).toHaveBeenCalledWith(
        'newMessage',
        expect.objectContaining({
          message: 'Hello World',
        }),
      );
    });

    it('should validate message data', async () => {
      const invalidData = { message: '' }; // Missing required fields

      await expect(gateway.handleMessage(mockSocket, invalidData)).rejects.toThrow(
        'Invalid message data',
      );
    });

    it('should handle message errors', async () => {
      const messageData = { room: 'room1', message: 'Hello' };

      (service.addMessage as jest.Mock).mockRejectedValue(
        new Error('Database error'),
      );

      await expect(gateway.handleMessage(mockSocket, messageData)).rejects.toThrow(
        'Database error',
      );

      expect(mockSocket.emit).toHaveBeenCalledWith('error', {
        message: 'Failed to send message',
      });
    });
  });

  describe('handleJoinRoom', () => {
    it('should add user to room', async () => {
      const roomData = { room: 'room1', username: 'John' };

      await gateway.handleJoinRoom(mockSocket, roomData);

      expect(mockSocket.join).toHaveBeenCalledWith('room1');
      expect(service.addUserToRoom).toHaveBeenCalledWith('room1', 'John');
      expect(mockSocket.to).toHaveBeenCalledWith('room1');
      expect(mockSocket.broadcast.emit).toHaveBeenCalledWith('userJoined', {
        username: 'John',
        room: 'room1',
      });
    });

    it('should handle room join errors', async () => {
      const roomData = { room: 'room1' };

      (service.addUserToRoom as jest.Mock).mockRejectedValue(
        new Error('Room full'),
      );

      await expect(gateway.handleJoinRoom(mockSocket, roomData)).rejects.toThrow(
        'Room full',
      );

      expect(mockSocket.emit).toHaveBeenCalledWith('error', {
        message: 'Failed to join room',
      });
    });
  });

  describe('handleLeaveRoom', () => {
    it('should remove user from room', async () => {
      const roomData = { room: 'room1' };

      await gateway.handleLeaveRoom(mockSocket, roomData);

      expect(mockSocket.leave).toHaveBeenCalledWith('room1');
      expect(mockSocket.to).toHaveBeenCalledWith('room1');
      expect(mockSocket.broadcast.emit).toHaveBeenCalledWith('userLeft', {
        socketId: 'test-socket-id',
        room: 'room1',
      });
    });
  });
});
```

**E2E Testing WebSockets:**

```typescript
// chat.e2e-spec.ts
import { Test } from '@nestjs/testing';
import { INestApplication } from '@nestjs/common';
import { io, Socket as ClientSocket } from 'socket.io-client';
import { AppModule } from '../src/app.module';

describe('ChatGateway (E2E)', () => {
  let app: INestApplication;
  let client1: ClientSocket;
  let client2: ClientSocket;
  const port = 3001;

  beforeAll(async () => {
    const module = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = module.createNestApplication();
    await app.listen(port);
  });

  afterAll(async () => {
    await app.close();
  });

  beforeEach((done) => {
    // Connect first client
    client1 = io(`http://localhost:${port}`, {
      auth: { token: 'test-token-1' },
    });

    client1.on('connect', () => {
      // Connect second client
      client2 = io(`http://localhost:${port}`, {
        auth: { token: 'test-token-2' },
      });

      client2.on('connect', done);
    });
  });

  afterEach(() => {
    client1.disconnect();
    client2.disconnect();
  });

  it('should connect to WebSocket server', (done) => {
    expect(client1.connected).toBe(true);
    expect(client2.connected).toBe(true);
    done();
  });

  it('should broadcast message to room', (done) => {
    const room = 'test-room';
    const message = 'Hello World';

    // Both clients join room
    client1.emit('joinRoom', { room, username: 'User1' });
    client2.emit('joinRoom', { room, username: 'User2' });

    // Client2 listens for messages
    client2.on('newMessage', (data) => {
      expect(data.message).toBe(message);
      expect(data.room).toBe(room);
      done();
    });

    // Client1 sends message
    setTimeout(() => {
      client1.emit('message', { room, message, user: 'User1' });
    }, 100);
  });
});
```

**Interview Tip**: Test WebSockets by **mocking Socket objects** with emit(), join(), leave(), broadcast properties. Test **connection/disconnection** handlers. Mock **authentication** via handshake.auth. For **E2E tests**, use **socket.io-client** to create real connections. Test **room management**, **message broadcasting**, **event emission**. Use **setTimeout** for async event testing.

</details>

### 47. How do you test scheduled jobs/cron tasks?

<details>
<summary>Answer</summary>

**Test scheduled jobs** by mocking the scheduler and manually triggering cron methods rather than waiting for scheduled execution.

**Testing Cron Jobs:**

```typescript
// tasks.service.spec.ts
import { Test } from '@nestjs/testing';
import { TasksService } from './tasks.service';
import { EmailService } from './email.service';
import { SchedulerRegistry } from '@nestjs/schedule';

describe('TasksService', () => {
  let service: TasksService;
  let emailService: EmailService;
  let schedulerRegistry: SchedulerRegistry;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        TasksService,
        {
          provide: EmailService,
          useValue: {
            sendDailyReport: jest.fn(),
            sendWeeklyDigest: jest.fn(),
          },
        },
        {
          provide: SchedulerRegistry,
          useValue: {
            getCronJob: jest.fn(),
            addCronJob: jest.fn(),
            deleteCronJob: jest.fn(),
            getCronJobs: jest.fn(),
          },
        },
      ],
    }).compile();

    service = module.get<TasksService>(TasksService);
    emailService = module.get<EmailService>(EmailService);
    schedulerRegistry = module.get<SchedulerRegistry>(SchedulerRegistry);
  });

  describe('handleDailyReport', () => {
    it('should send daily report', async () => {
      (emailService.sendDailyReport as jest.Mock).mockResolvedValue(true);

      await service.handleDailyReport();

      expect(emailService.sendDailyReport).toHaveBeenCalled();
    });

    it('should handle errors in daily report', async () => {
      (emailService.sendDailyReport as jest.Mock).mockRejectedValue(
        new Error('Email service down'),
      );

      await expect(service.handleDailyReport()).rejects.toThrow(
        'Email service down',
      );
    });
  });

  describe('handleWeeklyDigest', () => {
    it('should send weekly digest on Sunday', async () => {
      // Mock date to Sunday
      jest.useFakeTimers();
      jest.setSystemTime(new Date('2024-01-07')); // Sunday

      (emailService.sendWeeklyDigest as jest.Mock).mockResolvedValue(true);

      await service.handleWeeklyDigest();

      expect(emailService.sendWeeklyDigest).toHaveBeenCalled();

      jest.useRealTimers();
    });
  });
});
```

**Testing with jest.useFakeTimers:**

```typescript
describe('Interval Tasks', () => {
  beforeEach(() => {
    jest.useFakeTimers();
  });

  afterEach(() => {
    jest.useRealTimers();
  });

  it('should run task every 10 seconds', async () => {
    const taskSpy = jest.spyOn(service, 'handleHealthCheck');

    // Fast-forward time
    jest.advanceTimersByTime(10000);

    expect(taskSpy).toHaveBeenCalledTimes(1);

    jest.advanceTimersByTime(10000);
    expect(taskSpy).toHaveBeenCalledTimes(2);
  });
});
```

**Testing Dynamic Cron Jobs:**

```typescript
describe('Dynamic Cron Jobs', () => {
  it('should add dynamic cron job', () => {
    const cronTime = '*/5 * * * *'; // Every 5 minutes
    const jobName = 'dynamicJob';

    service.addDynamicCronJob(jobName, cronTime);

    expect(schedulerRegistry.addCronJob).toHaveBeenCalledWith(
      jobName,
      expect.any(Object),
    );
  });

  it('should manually trigger cron job', async () => {
    const mockJob = {
      lastDate: jest.fn(),
      nextDate: jest.fn(),
      fireOnTick: jest.fn(),
    };

    (schedulerRegistry.getCronJob as jest.Mock).mockReturnValue(mockJob);

    await service.triggerCronJob('dailyReport');

    expect(schedulerRegistry.getCronJob).toHaveBeenCalledWith('dailyReport');
    expect(mockJob.fireOnTick).toHaveBeenCalled();
  });
});
```

**Interview Tip**: Test scheduled jobs by **calling cron methods directly** instead of waiting for schedule. Use **jest.useFakeTimers()** and **jest.advanceTimersByTime()** for Interval/Timeout testing. Mock **SchedulerRegistry** to test dynamic job management. Test **business logic** within cron handlers. For integration tests, use **ScheduleModule.forRoot()** and manually trigger jobs via **fireOnTick()**.

</details>

### 48. How do you test event emitters?

<details>
<summary>Answer</summary>

**Test event emitters** by verifying event emission, listener registration, and event payload handling.

**Testing Event Emission:**

```typescript
// orders.service.spec.ts
import { Test } from '@nestjs/testing';
import { OrdersService } from './orders.service';
import { EventEmitter2 } from '@nestjs/event-emitter';

describe('OrdersService', () => {
  let service: OrdersService;
  let eventEmitter: EventEmitter2;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        OrdersService,
        {
          provide: EventEmitter2,
          useValue: {
            emit: jest.fn(),
            emitAsync: jest.fn(),
            on: jest.fn(),
            once: jest.fn(),
            removeListener: jest.fn(),
            removeAllListeners: jest.fn(),
          },
        },
      ],
    }).compile();

    service = module.get<OrdersService>(OrdersService);
    eventEmitter = module.get<EventEmitter2>(EventEmitter2);
  });

  describe('createOrder', () => {
    it('should emit order.created event', async () => {
      const orderData = {
        userId: 1,
        items: [{ productId: 1, quantity: 2 }],
        total: 100,
      };

      const createdOrder = { id: 1, ...orderData };

      await service.createOrder(orderData);

      expect(eventEmitter.emit).toHaveBeenCalledWith('order.created', createdOrder);
    });

    it('should emit order.created event with correct payload', async () => {
      const orderData = { userId: 1, total: 100 };

      await service.createOrder(orderData);

      expect(eventEmitter.emit).toHaveBeenCalledWith(
        'order.created',
        expect.objectContaining({
          id: expect.any(Number),
          userId: 1,
          total: 100,
          createdAt: expect.any(Date),
        }),
      );
    });

    it('should not emit event if order creation fails', async () => {
      const invalidData = { userId: null, total: -1 };

      await expect(service.createOrder(invalidData)).rejects.toThrow();

      expect(eventEmitter.emit).not.toHaveBeenCalled();
    });
  });

  describe('cancelOrder', () => {
    it('should emit order.cancelled event', async () => {
      const orderId = 1;

      await service.cancelOrder(orderId);

      expect(eventEmitter.emit).toHaveBeenCalledWith(
        'order.cancelled',
        expect.objectContaining({ orderId }),
      );
    });

    it('should emit async event for notifications', async () => {
      const orderId = 1;

      (eventEmitter.emitAsync as jest.Mock).mockResolvedValue([true, true]);

      await service.cancelOrder(orderId);

      expect(eventEmitter.emitAsync).toHaveBeenCalledWith(
        'order.cancelled',
        expect.any(Object),
      );
    });
  });
});
```

**Service Implementation with Events:**

```typescript
// orders.service.ts
import { Injectable } from '@nestjs/common';
import { EventEmitter2 } from '@nestjs/event-emitter';

@Injectable()
export class OrdersService {
  constructor(private eventEmitter: EventEmitter2) {}

  async createOrder(orderData: CreateOrderDto) {
    // Create order
    const order = await this.ordersRepository.save(orderData);

    // Emit event
    this.eventEmitter.emit('order.created', order);

    return order;
  }

  async cancelOrder(orderId: number) {
    const order = await this.ordersRepository.findOne({ where: { id: orderId } });
    
    if (!order) {
      throw new NotFoundException('Order not found');
    }

    order.status = 'cancelled';
    await this.ordersRepository.save(order);

    // Emit async event (waits for all listeners)
    await this.eventEmitter.emitAsync('order.cancelled', {
      orderId: order.id,
      cancelledAt: new Date(),
    });

    return order;
  }
}
```

**Testing Event Listeners:**

```typescript
// notifications.listener.spec.ts
import { Test } from '@nestjs/testing';
import { NotificationsListener } from './notifications.listener';
import { EmailService } from './email.service';

describe('NotificationsListener', () => {
  let listener: NotificationsListener;
  let emailService: EmailService;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        NotificationsListener,
        {
          provide: EmailService,
          useValue: {
            sendOrderConfirmation: jest.fn(),
            sendCancellationEmail: jest.fn(),
          },
        },
      ],
    }).compile();

    listener = module.get<NotificationsListener>(NotificationsListener);
    emailService = module.get<EmailService>(EmailService);
  });

  describe('handleOrderCreated', () => {
    it('should send order confirmation email', async () => {
      const orderEvent = {
        id: 1,
        userId: 1,
        total: 100,
        createdAt: new Date(),
      };

      (emailService.sendOrderConfirmation as jest.Mock).mockResolvedValue(true);

      await listener.handleOrderCreated(orderEvent);

      expect(emailService.sendOrderConfirmation).toHaveBeenCalledWith(orderEvent);
    });

    it('should handle email service errors', async () => {
      const orderEvent = { id: 1, userId: 1 };

      (emailService.sendOrderConfirmation as jest.Mock).mockRejectedValue(
        new Error('Email service down'),
      );

      // Listener should not throw
      await expect(listener.handleOrderCreated(orderEvent)).resolves.not.toThrow();
    });

    it('should log email sent successfully', async () => {
      const logSpy = jest.spyOn(listener['logger'], 'log');
      const orderEvent = { id: 1, userId: 1 };

      await listener.handleOrderCreated(orderEvent);

      expect(logSpy).toHaveBeenCalledWith('Order confirmation sent for order #1');
    });
  });

  describe('handleOrderCancelled', () => {
    it('should send cancellation email', async () => {
      const cancelEvent = {
        orderId: 1,
        cancelledAt: new Date(),
      };

      await listener.handleOrderCancelled(cancelEvent);

      expect(emailService.sendCancellationEmail).toHaveBeenCalledWith(cancelEvent);
    });
  });
});
```

**Listener Implementation:**

```typescript
// notifications.listener.ts
import { Injectable, Logger } from '@nestjs/common';
import { OnEvent } from '@nestjs/event-emitter';

@Injectable()
export class NotificationsListener {
  private readonly logger = new Logger(NotificationsListener.name);

  constructor(private emailService: EmailService) {}

  @OnEvent('order.created')
  async handleOrderCreated(payload: any) {
    try {
      await this.emailService.sendOrderConfirmation(payload);
      this.logger.log(`Order confirmation sent for order #${payload.id}`);
    } catch (error) {
      this.logger.error('Failed to send order confirmation', error);
    }
  }

  @OnEvent('order.cancelled')
  async handleOrderCancelled(payload: any) {
    await this.emailService.sendCancellationEmail(payload);
  }

  @OnEvent('order.*', { async: true })
  async handleAllOrderEvents(payload: any) {
    this.logger.log('Order event received', payload);
  }
}
```

**Testing Multiple Listeners:**

```typescript
describe('Multiple Event Listeners', () => {
  let notificationsListener: NotificationsListener;
  let analyticsListener: AnalyticsListener;
  let eventEmitter: EventEmitter2;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        NotificationsListener,
        AnalyticsListener,
        {
          provide: EventEmitter2,
          useValue: {
            emit: jest.fn(),
          },
        },
      ],
    }).compile();

    notificationsListener = module.get<NotificationsListener>(NotificationsListener);
    analyticsListener = module.get<AnalyticsListener>(AnalyticsListener);
  });

  it('should call both listeners for same event', async () => {
    const notificationsSpy = jest.spyOn(notificationsListener, 'handleOrderCreated');
    const analyticsSpy = jest.spyOn(analyticsListener, 'trackOrderCreated');

    const orderEvent = { id: 1, userId: 1 };

    await notificationsListener.handleOrderCreated(orderEvent);
    await analyticsListener.trackOrderCreated(orderEvent);

    expect(notificationsSpy).toHaveBeenCalledWith(orderEvent);
    expect(analyticsSpy).toHaveBeenCalledWith(orderEvent);
  });
});
```

**Integration Test with Real Event Emitter:**

```typescript
// events.integration.spec.ts
describe('Events Integration', () => {
  let app: INestApplication;
  let ordersService: OrdersService;
  let notificationsListener: NotificationsListener;
  let emailService: EmailService;

  beforeAll(async () => {
    const module = await Test.createTestingModule({
      imports: [
        EventEmitterModule.forRoot(),
        OrdersModule,
        NotificationsModule,
      ],
    }).compile();

    app = module.createNestApplication();
    await app.init();

    ordersService = module.get<OrdersService>(OrdersService);
    notificationsListener = module.get<NotificationsListener>(NotificationsListener);
    emailService = module.get<EmailService>(EmailService);
  });

  afterAll(async () => {
    await app.close();
  });

  it('should trigger listener when event is emitted', async (done) => {
    const emailSpy = jest.spyOn(emailService, 'sendOrderConfirmation');

    const orderData = { userId: 1, total: 100 };

    // Create order (emits event)
    await ordersService.createOrder(orderData);

    // Give event loop time to process
    setTimeout(() => {
      expect(emailSpy).toHaveBeenCalled();
      done();
    }, 100);
  });

  it('should handle async events', async () => {
    const emailSpy = jest.spyOn(emailService, 'sendCancellationEmail');

    await ordersService.cancelOrder(1);

    // Async events wait for listeners
    expect(emailSpy).toHaveBeenCalled();
  });
});
```

**Testing Wildcard Event Listeners:**

```typescript
describe('Wildcard Listeners', () => {
  it('should listen to all order events', async () => {
    const wildcardSpy = jest.spyOn(listener, 'handleAllOrderEvents');

    await listener.handleOrderCreated({ id: 1 });
    await listener.handleOrderCancelled({ orderId: 1 });

    expect(wildcardSpy).toHaveBeenCalledTimes(2);
  });

  it('should filter events by pattern', async () => {
    const orderSpy = jest.spyOn(listener, 'handleAllOrderEvents');
    const userSpy = jest.spyOn(listener, 'handleUserEvents');

    // Only order.* events should trigger
    await listener.handleAllOrderEvents({ type: 'order.created' });

    expect(orderSpy).toHaveBeenCalled();
    expect(userSpy).not.toHaveBeenCalled();
  });
});
```

**Testing Event Payload Transformation:**

```typescript
describe('Event Payload', () => {
  it('should transform payload before emission', async () => {
    const orderData = { userId: 1, items: [], total: 100 };

    await service.createOrder(orderData);

    expect(eventEmitter.emit).toHaveBeenCalledWith(
      'order.created',
      expect.objectContaining({
        id: expect.any(Number),
        userId: 1,
        total: 100,
        items: [],
        // Transformed fields
        totalFormatted: '$100.00',
        itemCount: 0,
        createdAt: expect.any(Date),
      }),
    );
  });

  it('should validate payload before emission', async () => {
    const invalidData = { userId: null };

    await expect(service.createOrder(invalidData)).rejects.toThrow(
      'Invalid order data',
    );

    expect(eventEmitter.emit).not.toHaveBeenCalled();
  });
});
```

**Interview Tip**: Test event emitters by **mocking EventEmitter2** with emit(), emitAsync(), on() methods. Verify **emit() calls** with correct event names and payloads. Test **listeners separately** from emitters. Use **jest.spyOn()** to verify listener execution. For **integration tests**, use **EventEmitterModule.forRoot()** with real event flow. Test **wildcard listeners** (order.*), **async events**, **multiple listeners**, and **error handling** in listeners. Use **setTimeout** in integration tests to allow event processing.

</details>

## Test Performance

### 49. How do you speed up test execution?

<details>
<summary>Answer</summary>

**Speed up test execution** by running tests in parallel, reducing setup overhead, and optimizing database operations.

**1. Run Tests in Parallel:**

```json
// package.json
{
  "scripts": {
    "test": "jest --maxWorkers=4",
    "test:parallel": "jest --maxWorkers=50%",
    "test:serial": "jest --runInBand"
  }
}
```

**Jest Configuration:**

```javascript
// jest.config.js
module.exports = {
  maxWorkers: '50%', // Use 50% of available CPU cores
  maxConcurrency: 10, // Max concurrent tests per worker
  testTimeout: 10000, // 10 seconds
  bail: 1, // Stop after first failure
  cache: true, // Enable caching
  cacheDirectory: '.jest-cache',
};
```

**2. Optimize Module Imports:**

```typescript
// ❌ Slow: Import entire module
import { Test, TestingModule } from '@nestjs/testing';
import { TypeOrmModule } from '@nestjs/typeorm';
import { ConfigModule } from '@nestjs/config';

// ✅ Fast: Use module caching
let cachedModule: TestingModule;

beforeAll(async () => {
  if (!cachedModule) {
    cachedModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();
  }
});

// Reuse compiled module
beforeEach(() => {
  service = cachedModule.get<UsersService>(UsersService);
});
```

**3. Use Global Test Setup:**

```typescript
// jest-global-setup.ts
module.exports = async () => {
  // Set up once for all tests
  process.env.NODE_ENV = 'test';
  process.env.DATABASE_URL = 'postgresql://test:test@localhost:5432/test_db';
  
  // Start Docker containers
  await startTestDatabase();
};
```

```typescript
// jest-global-teardown.ts
module.exports = async () => {
  // Clean up after all tests
  await stopTestDatabase();
};
```

```javascript
// jest.config.js
module.exports = {
  globalSetup: './jest-global-setup.ts',
  globalTeardown: './jest-global-teardown.ts',
};
```

**4. Mock Heavy Dependencies:**

```typescript
// ❌ Slow: Real database connection
const module = await Test.createTestingModule({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost',
      database: 'test_db',
    }),
  ],
}).compile();

// ✅ Fast: Mock database
const module = await Test.createTestingModule({
  providers: [
    UsersService,
    {
      provide: getRepositoryToken(User),
      useValue: {
        find: jest.fn(),
        findOne: jest.fn(),
        save: jest.fn(),
      },
    },
  ],
}).compile();
```

**5. Use In-Memory Database:**

```typescript
// Fast SQLite in-memory for unit tests
const module = await Test.createTestingModule({
  imports: [
    TypeOrmModule.forRoot({
      type: 'sqlite',
      database: ':memory:',
      entities: [User, Order],
      synchronize: true,
    }),
  ],
}).compile();
```

**6. Reduce beforeEach/afterEach Overhead:**

```typescript
// ❌ Slow: Recreate module for each test
describe('UsersService', () => {
  let service: UsersService;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [UsersService],
    }).compile();
    service = module.get<UsersService>(UsersService);
  });

  // 100 tests = 100 module compilations
});

// ✅ Fast: Create once, reset mocks
describe('UsersService', () => {
  let service: UsersService;
  let repository: Repository<User>;

  beforeAll(async () => {
    const module = await Test.createTestingModule({
      providers: [UsersService, mockRepository],
    }).compile();
    service = module.get<UsersService>(UsersService);
    repository = module.get(getRepositoryToken(User));
  });

  afterEach(() => {
    jest.clearAllMocks(); // Only reset mocks
  });

  // 100 tests = 1 module compilation
});
```

**7. Skip Unnecessary Tests:**

```typescript
// Skip integration tests in watch mode
describe.skip('Integration Tests', () => {
  // Skipped
});

// Run only specific tests
describe.only('Critical Tests', () => {
  // Only these run
});

// Conditional tests
const runIntegration = process.env.RUN_INTEGRATION === 'true';
(runIntegration ? describe : describe.skip)('Integration', () => {
  // Runs conditionally
});
```

**8. Optimize Database Cleanup:**

```typescript
// ❌ Slow: Drop and recreate for each test
afterEach(async () => {
  await connection.dropDatabase();
  await connection.synchronize();
});

// ✅ Fast: Use transactions and rollback
beforeEach(async () => {
  await connection.query('BEGIN');
});

afterEach(async () => {
  await connection.query('ROLLBACK');
});

// ✅ Faster: Truncate instead of drop
afterEach(async () => {
  const entities = connection.entityMetadatas;
  for (const entity of entities) {
    await connection.query(`TRUNCATE TABLE "${entity.tableName}" CASCADE`);
  }
});
```

**9. Use Test Sharding:**

```json
// package.json - Run tests in CI with sharding
{
  "scripts": {
    "test:shard1": "jest --shard=1/4",
    "test:shard2": "jest --shard=2/4",
    "test:shard3": "jest --shard=3/4",
    "test:shard4": "jest --shard=4/4"
  }
}
```

**10. Profile Slow Tests:**

```bash
# Find slow tests
jest --listTests
jest --showConfig
jest --verbose

# Profile test execution
NODE_OPTIONS="--max-old-space-size=4096" jest --logHeapUsage

# Detect slow tests
jest --detectOpenHandles --forceExit
```

**Jest Performance Configuration:**

```javascript
// jest.config.js
module.exports = {
  // Performance optimizations
  maxWorkers: '50%',
  cache: true,
  bail: 1,
  testTimeout: 5000,
  
  // Skip coverage for faster runs
  collectCoverage: false,
  
  // Ignore node_modules
  testPathIgnorePatterns: ['/node_modules/', '/dist/'],
  
  // Transform only necessary files
  transformIgnorePatterns: [
    'node_modules/(?!(some-es6-module)/)',
  ],
  
  // Use faster transformer
  transform: {
    '^.+\\.(t|j)s$': ['@swc/jest'], // Faster than ts-jest
  },
};
```

**Comparison: Test Execution Times:**

| Optimization | Time (100 tests) | Improvement |
|--------------|------------------|-------------|
| Sequential (--runInBand) | 60s | Baseline |
| Parallel (--maxWorkers=4) | 20s | 3x faster |
| In-memory DB | 15s | 4x faster |
| Mock dependencies | 8s | 7.5x faster |
| beforeAll + cached module | 5s | 12x faster |

**Interview Tip**: Speed up tests by **running in parallel** (--maxWorkers), using **beforeAll** instead of beforeEach, **mocking heavy dependencies**, using **in-memory databases**, enabling **Jest cache**, and using **transactions** for database cleanup. Optimize with **@swc/jest** instead of ts-jest. Use **--bail** to stop on first failure. Profile with **--logHeapUsage**. For CI, use **test sharding** (--shard=1/4).

</details>

### 50. Should you use in-memory databases for tests?

<details>
<summary>Answer</summary>

**Use in-memory databases for unit and integration tests** when speed is critical, but use real databases for E2E tests to catch production issues.

**In-Memory Database Pros & Cons:**

| Aspect | In-Memory (SQLite) | Real Database (PostgreSQL) |
|--------|-------------------|---------------------------|
| **Speed** | ✅ Very fast (10-50x) | ❌ Slower |
| **Setup** | ✅ No external dependencies | ❌ Requires Docker/service |
| **Isolation** | ✅ Perfect isolation | ⚠️ Needs cleanup |
| **Accuracy** | ❌ Different SQL dialect | ✅ Production-like |
| **Features** | ❌ Missing features | ✅ Full features |
| **CI/CD** | ✅ No setup needed | ❌ Docker required |

**1. SQLite In-Memory Setup:**

```typescript
// test-db.config.ts
import { TypeOrmModuleOptions } from '@nestjs/typeorm';

export const testDbConfig: TypeOrmModuleOptions = {
  type: 'sqlite',
  database: ':memory:',
  entities: [__dirname + '/../**/*.entity{.ts,.js}'],
  synchronize: true,
  dropSchema: true,
  logging: false,
};
```

```typescript
// users.service.integration.spec.ts
describe('UsersService (In-Memory)', () => {
  let app: INestApplication;
  let service: UsersService;

  beforeAll(async () => {
    const module = await Test.createTestingModule({
      imports: [
        TypeOrmModule.forRoot({
          type: 'sqlite',
          database: ':memory:',
          entities: [User, Order],
          synchronize: true,
        }),
        TypeOrmModule.forFeature([User, Order]),
        UsersModule,
      ],
    }).compile();

    app = module.createNestApplication();
    await app.init();

    service = module.get<UsersService>(UsersService);
  });

  afterAll(async () => {
    await app.close();
  });

  it('should create user', async () => {
    const user = await service.create({
      name: 'John',
      email: 'john@example.com',
    });

    expect(user.id).toBeDefined();
    expect(user.name).toBe('John');
  });

  it('should find user by email', async () => {
    await service.create({ name: 'John', email: 'john@example.com' });

    const user = await service.findByEmail('john@example.com');

    expect(user).toBeDefined();
    expect(user.name).toBe('John');
  });
});
```

**2. Real Database Setup (Docker):**

```typescript
// test-db-real.config.ts
export const realTestDbConfig: TypeOrmModuleOptions = {
  type: 'postgres',
  host: 'localhost',
  port: 5433, // Different from production port
  username: 'test',
  password: 'test',
  database: 'test_db',
  entities: [__dirname + '/../**/*.entity{.ts,.js}'],
  synchronize: true,
  dropSchema: true,
};
```

```yaml
# docker-compose.test.yml
version: '3.8'
services:
  postgres-test:
    image: postgres:15
    environment:
      POSTGRES_USER: test
      POSTGRES_PASSWORD: test
      POSTGRES_DB: test_db
    ports:
      - "5433:5432"
    tmpfs:
      - /var/lib/postgresql/data # In-memory storage
```

```typescript
// users.service.real-db.spec.ts
describe('UsersService (Real Database)', () => {
  let app: INestApplication;
  let service: UsersService;
  let dataSource: DataSource;

  beforeAll(async () => {
    const module = await Test.createTestingModule({
      imports: [
        TypeOrmModule.forRoot(realTestDbConfig),
        UsersModule,
      ],
    }).compile();

    app = module.createNestApplication();
    await app.init();

    service = module.get<UsersService>(UsersService);
    dataSource = module.get<DataSource>(DataSource);
  });

  afterEach(async () => {
    // Clean database after each test
    await dataSource.query('TRUNCATE TABLE users CASCADE');
  });

  afterAll(async () => {
    await dataSource.destroy();
    await app.close();
  });

  it('should use PostgreSQL-specific features', async () => {
    // Test JSONB queries, full-text search, etc.
    const result = await dataSource.query(`
      SELECT * FROM users 
      WHERE metadata @> '{"role": "admin"}'::jsonb
    `);

    expect(result).toBeDefined();
  });
});
```

**3. When to Use Each Approach:**

**Use In-Memory (SQLite) for:**
```typescript
// ✅ Unit tests with database
describe('UsersRepository Unit Tests', () => {
  // Fast, isolated, no external dependencies
});

// ✅ Simple CRUD operations
it('should create and find user', async () => {
  const user = await repo.save({ name: 'John' });
  const found = await repo.findOne({ where: { id: user.id } });
  expect(found).toBeDefined();
});

// ✅ Basic relationships
it('should load user with orders', async () => {
  const user = await repo.findOne({
    where: { id: 1 },
    relations: ['orders'],
  });
  expect(user.orders).toHaveLength(2);
});
```

**Use Real Database for:**
```typescript
// ✅ Database-specific features
it('should use PostgreSQL full-text search', async () => {
  const results = await repo.query(`
    SELECT * FROM users 
    WHERE to_tsvector('english', name) @@ to_tsquery('John')
  `);
  expect(results).toHaveLength(1);
});

// ✅ Complex queries with specific SQL
it('should use window functions', async () => {
  const results = await repo.query(`
    SELECT *, ROW_NUMBER() OVER (PARTITION BY department_id) 
    FROM users
  `);
  expect(results).toBeDefined();
});

// ✅ JSON/JSONB operations
it('should query JSONB fields', async () => {
  const user = await repo.findOne({
    where: {
      metadata: () => "metadata @> '{\"role\": \"admin\"}'",
    },
  });
  expect(user).toBeDefined();
});

// ✅ Transaction isolation levels
it('should use SERIALIZABLE isolation', async () => {
  await dataSource.transaction('SERIALIZABLE', async (em) => {
    // Test concurrent transaction behavior
  });
});

// ✅ E2E tests
describe('API E2E Tests', () => {
  // Use production-like database
});
```

**4. Hybrid Approach:**

```typescript
// jest.config.js
module.exports = {
  projects: [
    {
      displayName: 'unit',
      testMatch: ['**/*.spec.ts'],
      // Fast: SQLite in-memory
      globals: {
        DATABASE_TYPE: 'sqlite',
      },
    },
    {
      displayName: 'integration',
      testMatch: ['**/*.integration.spec.ts'],
      // Accurate: Real PostgreSQL
      globals: {
        DATABASE_TYPE: 'postgres',
      },
    },
  ],
};
```

**5. SQLite Limitations to Watch:**

```typescript
// ❌ SQLite doesn't support these
describe('Database-Specific Features', () => {
  // PostgreSQL ENUM types
  it('should use enum', async () => {
    // Works in PostgreSQL, fails in SQLite
    await repo.save({ status: 'active' }); // ENUM type
  });

  // Array columns
  it('should query array column', async () => {
    // PostgreSQL: tags TEXT[]
    // SQLite: No native array support
    const users = await repo.find({
      where: { tags: () => "'admin' = ANY(tags)" },
    });
  });

  // JSONB operators
  it('should use JSONB containment', async () => {
    // PostgreSQL: metadata @> '{}'
    // SQLite: Limited JSON support
  });

  // Full-text search
  it('should use full-text search', async () => {
    // PostgreSQL: to_tsvector, to_tsquery
    // SQLite: Different FTS implementation
  });
});
```

**6. Performance Comparison:**

```typescript
// Benchmark test execution
describe('Performance Comparison', () => {
  it('SQLite in-memory: 100 inserts', async () => {
    const start = Date.now();
    
    for (let i = 0; i < 100; i++) {
      await repo.save({ name: `User ${i}` });
    }
    
    const elapsed = Date.now() - start;
    console.log(`SQLite: ${elapsed}ms`); // ~50ms
  });

  it('PostgreSQL: 100 inserts', async () => {
    const start = Date.now();
    
    for (let i = 0; i < 100; i++) {
      await repo.save({ name: `User ${i}` });
    }
    
    const elapsed = Date.now() - start;
    console.log(`PostgreSQL: ${elapsed}ms`); // ~500ms
  });
});
```

**Recommendation:**

```typescript
// ✅ Best Practice: Use both strategically
{
  "scripts": {
    "test": "jest",                    // Fast: SQLite for unit tests
    "test:integration": "jest --config jest-integration.config.js", // Real DB
    "test:e2e": "jest --config jest-e2e.config.js"                  // Real DB
  }
}
```

**Interview Tip**: Use **SQLite :memory:** for **unit and simple integration tests** (fast, no setup). Use **real database** (PostgreSQL/MySQL) for **complex queries**, **database-specific features**, **E2E tests**, and **production-like scenarios**. Hybrid approach: **SQLite for speed**, **real DB for accuracy**. Watch for **SQL dialect differences** (ENUM, arrays, JSONB, full-text search). For CI/CD, SQLite is faster but consider **Docker with tmpfs** for real DB speed.

</details>
